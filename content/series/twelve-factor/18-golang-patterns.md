+++
title = "Chương 18 — Golang Patterns cho ứng dụng 12-Factor"
date = "2026-07-10T19:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> **Phần 3 – Engineering** | Chương trước: [Admin Processes](/series/twelve-factor/17-factor-12-admin-processes/) | Chương sau: [Docker](/series/twelve-factor/19-docker/)

Phần 2 đã rải các mảnh code Go theo từng factor. Chương này hệ thống hóa chúng thành **bộ pattern hoàn chỉnh** của một service Go production-ready: cấu trúc, dependency injection, health check, signal handling, và một bộ khung `main()` chuẩn có thể tái sử dụng cho mọi service.

---

## 1. Cấu trúc project — kiến trúc phản chiếu các factor

```
myapp/
├── go.mod / go.sum              # F2
├── cmd/app/main.go              # composition root — nơi DUY NHẤT wiring mọi thứ
├── internal/
│   ├── config/config.go         # F3 — nơi DUY NHẤT đọc env
│   ├── logging/logging.go       # F11 — slog JSON ra stdout
│   ├── server/                  # F7, F9 — HTTP server, health, graceful shutdown
│   │   ├── server.go
│   │   └── middleware/
│   ├── service/                 # business logic — KHÔNG import repository/postgres,
│   │   └── order.go             #   chỉ import interface nó tự khai báo
│   ├── repository/postgres/     # F4 — implementation của backing service
│   ├── queue/                   # F4, F8 — producer/consumer
│   └── migrate/sql/             # F12 — migration embed
└── deploy/                      # Dockerfile, k8s/, compose
```

Nguyên tắc phụ thuộc: `cmd → internal/{server,service,repository}`; `service` chỉ biết interface; `config` không import gì của app. Vòng phụ thuộc là mùi thiết kế — Go từ chối compile là đồng minh của bạn.

## 2. Dependency Injection — thủ công, tường minh, không magic

Go không cần DI framework. **Constructor injection + composition root** là đủ cho tuyệt đại đa số service, và quan trọng với 12-Factor vì: mọi backing service (F4) đi vào qua constructor → nhìn `main()` là thấy toàn bộ "app này gắn với gì"; test swap implementation không cần hack.

```go
// internal/service/order.go — service khai báo interface NÓ CẦN (consumer-side)
package service

type OrderRepo interface {
	Create(ctx context.Context, o *Order) error
	GetByID(ctx context.Context, id string) (*Order, error)
}

type EventPublisher interface {
	Publish(ctx context.Context, topic string, payload any) error
}

type OrderService struct {
	repo   OrderRepo
	events EventPublisher
	log    *slog.Logger
}

func NewOrderService(repo OrderRepo, events EventPublisher, log *slog.Logger) *OrderService {
	return &OrderService{repo: repo, events: events, log: log}
}

func (s *OrderService) Create(ctx context.Context, o *Order) error {
	if err := o.Validate(); err != nil {
		return fmt.Errorf("%w: %v", ErrInvalidOrder, err)
	}
	if err := s.repo.Create(ctx, o); err != nil {
		return fmt.Errorf("create order: %w", err)
	}
	// publish lỗi không làm fail order — quyết định nghiệp vụ tường minh
	if err := s.events.Publish(ctx, "orders.created", o); err != nil {
		s.log.Error("publish failed", "order_id", o.ID, "err", err)
	}
	return nil
}
```

```go
// cmd/app/main.go — composition root: đồ thị phụ thuộc dựng Ở MỘT NƠI
pool := postgres.MustNewPool(ctx, cfg.DatabaseURL)
rdb := redisclient.MustNew(cfg.RedisURL)
producer := kafkaqueue.MustNewProducer(cfg.KafkaBrokers)

orderSvc := service.NewOrderService(
	postgres.NewOrderRepo(pool),
	producer,
	logger,
)
srv := server.New(cfg, logger, orderSvc)
```

Khi đồ thị phụ thuộc thật sự lớn (hàng chục component), `google/wire` (codegen, vẫn tường minh) đáng cân nhắc; runtime container kiểu `dig/fx` đánh đổi tính tường minh — hiếm khi xứng đáng trong service Go điển hình.

## 3. Health checks — liveness, readiness, startup nói ba thứ tiếng khác nhau

Sai lầm phổ biến nhất: một endpoint `/health` cho mọi mục đích. Ba probe trả lời **ba câu hỏi khác nhau** và sai hệ quả khác nhau:

| Probe | Câu hỏi | Nếu fail, K8s sẽ | Nên check gì |
|---|---|---|---|
| **Liveness** | Process có kẹt chết không? | **Restart container** | Gần như không gì cả — process trả lời HTTP được là sống. **Không check dependency!** |
| **Readiness** | Có nên đưa traffic vào lúc này? | **Rút khỏi Service** (không restart) | Dependency thiết yếu (DB), trạng thái warm-up, cờ draining |
| **Startup** | Đã khởi động xong chưa? | Đợi (chặn liveness/readiness) | Cho app khởi động chậm; Go hiếm khi cần |

Vì sao liveness không được check DB — lý giải một lần cho mãi mãi: DB sập → mọi pod fail liveness → K8s **restart toàn bộ fleet** → app đang khỏe bị giết hàng loạt, connection storm dội vào DB đang yếu → sự cố tệ gấp đôi. DB sập thì việc đúng là *ngừng nhận traffic* (readiness) và *đợi* — không phải tự sát tập thể.

```go
// internal/server/health.go
package server

import (
	"context"
	"net/http"
	"sync/atomic"
	"time"
)

type Health struct {
	ready    atomic.Bool
	draining atomic.Bool
	checks   map[string]func(context.Context) error // dependency thiết yếu
}

func NewHealth() *Health {
	return &Health{checks: map[string]func(context.Context) error{}}
}

func (h *Health) AddReadinessCheck(name string, fn func(context.Context) error) {
	h.checks[name] = fn
}
func (h *Health) SetReady(v bool)  { h.ready.Store(v) }
func (h *Health) StartDraining()   { h.draining.Store(true) }

// Liveness: "tôi còn sống" — trả lời được là đủ
func (h *Health) LiveHandler(w http.ResponseWriter, r *http.Request) {
	w.WriteHeader(http.StatusOK)
}

// Readiness: "tôi phục vụ được ngay bây giờ"
func (h *Health) ReadyHandler(w http.ResponseWriter, r *http.Request) {
	if h.draining.Load() || !h.ready.Load() {
		http.Error(w, "not ready", http.StatusServiceUnavailable)
		return
	}
	ctx, cancel := context.WithTimeout(r.Context(), 2*time.Second)
	defer cancel()
	for name, check := range h.checks {
		if err := check(ctx); err != nil {
			http.Error(w, name+": "+err.Error(), http.StatusServiceUnavailable)
			return
		}
	}
	w.WriteHeader(http.StatusOK)
}
```

```go
// wiring: DB là dependency thiết yếu → vào readiness (KHÔNG vào liveness)
health := server.NewHealth()
health.AddReadinessCheck("postgres", func(ctx context.Context) error {
	return pool.Ping(ctx)
})
```

## 4. Bộ khung `main()` chuẩn — ghép tất cả pattern

Đây là khung tổng hợp mọi thứ đã học (F3 config, F4 attach resources, F7 port, F9 lifecycle, F11 logging), dùng `errgroup` quản lý nhiều thành phần dài hạn:

```go
// cmd/app/main.go — khung server production-ready hoàn chỉnh
package main

import (
	"context"
	"errors"
	"fmt"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"

	"golang.org/x/sync/errgroup"
)

func main() {
	if err := run(); err != nil {
		fmt.Fprintln(os.Stderr, "fatal:", err)
		os.Exit(1)
	}
}

// run tách khỏi main: trả error thay vì os.Exit rải rác → test được toàn bộ vòng đời
func run() error {
	// 1. Config trước tiên — fail fast (F3)
	cfg, err := config.Load()
	if err != nil {
		return fmt.Errorf("config: %w", err)
	}
	logger := logging.New(cfg.LogLevel) // F11
	slog.SetDefault(logger)

	// 2. Root context gắn với signal (F9)
	ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGTERM, syscall.SIGINT)
	defer stop()

	// 3. Attach backing services — fail fast với resource thiết yếu (F4)
	pool, err := postgres.NewPool(ctx, cfg.DatabaseURL)
	if err != nil {
		return fmt.Errorf("attach postgres: %w", err)
	}
	defer pool.Close()

	// 4. Composition root — DI thủ công
	health := server.NewHealth()
	health.AddReadinessCheck("postgres", pool.Ping)
	orderSvc := service.NewOrderService(postgres.NewOrderRepo(pool), /*...*/ logger)
	handler := server.NewRouter(logger, health, orderSvc) // middleware: recover, request-id, access log

	srv := &http.Server{
		Addr:              ":" + cfg.Port, // F7
		Handler:           handler,
		ReadHeaderTimeout: 5 * time.Second,
		ReadTimeout:       10 * time.Second,
		WriteTimeout:      30 * time.Second,
		IdleTimeout:       120 * time.Second,
	}

	// 5. errgroup: các thành phần dài hạn sống chết cùng nhau
	g, gctx := errgroup.WithContext(ctx)

	g.Go(func() error { // HTTP server
		logger.Info("listening", "port", cfg.Port, "version", buildinfo.Version)
		health.SetReady(true)
		if err := srv.ListenAndServe(); err != nil && !errors.Is(err, http.ErrServerClosed) {
			return fmt.Errorf("http server: %w", err)
		}
		return nil
	})

	g.Go(func() error { // vòng đời shutdown (F9 — mẫu 3 giai đoạn, chương 14)
		<-gctx.Done() // signal HOẶC thành phần khác chết
		logger.Info("shutting down")
		health.StartDraining()
		time.Sleep(cfg.DrainDelay) // đợi endpoint removal lan tỏa
		shCtx, cancel := context.WithTimeout(context.Background(), cfg.ShutdownTimeout)
		defer cancel()
		return srv.Shutdown(shCtx)
	})

	// (worker/consumer thêm g.Go tương tự — cùng vòng đời)

	if err := g.Wait(); err != nil {
		return err
	}
	logger.Info("stopped cleanly")
	return nil
}
```

Các tính chất của khung: **một đường thoát** (mọi lỗi chảy về `run() → error`); **thứ tự shutdown đúng** (drain → server → defer đóng pool sau cùng); **một thành phần chết kéo cả process xuống** có trật tự (errgroup hủy context chung) — để nền tảng thay pod nguyên vẹn, đúng triết lý crash-only thay vì sống dở chết dở.

## 5. Middleware tối thiểu cho production

Thứ tự bọc (ngoài → trong): **Recover → RequestID/Logger → (OTel) → Timeout → Auth → Handler**.

```go
// Recover: panic trong handler → 500 + log, KHÔNG chết process
// (panic ngoài handler thì cứ để chết — crash-only, nền tảng thay pod)
func Recover(log *slog.Logger) func(http.Handler) http.Handler {
	return func(next http.Handler) http.Handler {
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			defer func() {
				if rec := recover(); rec != nil {
					log.Error("panic", "recover", rec, "stack", string(debug.Stack()),
						"path", r.URL.Path)
					http.Error(w, "internal error", http.StatusInternalServerError)
				}
			}()
			next.ServeHTTP(w, r)
		})
	}
}
```

Cùng với RequestLogger (chương 16) và nguyên tắc **context xuyên suốt**: mọi hàm chạm IO nhận `ctx context.Context` làm tham số đầu — đó là đường ống mang deadline, cancellation (SIGTERM lan tới tận query đang chạy), request_id và trace. Code Go không kỷ luật context là code không tắt êm được.

## 6. Anti-patterns Go đặc thù

- **`init()` làm việc nặng / đọc env** — chạy trước `main`, không kiểm soát thứ tự, không trả error, không test được. `init()` chỉ dành cho đăng ký thuần túy.
- **Biến global cho dependency** (`var DB *sql.DB` package-level) — DI qua cửa sau: phụ thuộc vô hình, test phải mutate global, race trong test song song.
- **`log.Fatal` / `os.Exit` rải rác trong tầng sâu** — nhảy qua mọi defer (connection không đóng, graceful shutdown không chạy); chỉ tầng `main/run` được quyết định thoát.
- **Nuốt `ctx`**: `context.Background()` trong tầng sâu thay vì nhận từ trên — cắt đứt dây cancellation, request bị hủy mà query vẫn chạy.
- **Goroutine không có chủ** (`go doSomething()` không vào errgroup/WaitGroup) — rò rỉ, chết im lặng, hoặc sống sót qua shutdown; mọi goroutine dài hạn phải thuộc về một vòng đời.
- **Interface phình to phía provider** (`OrderRepository` 20 method) — ngược idiom Go; interface hẹp, khai báo phía consumer.

---

## Tóm tắt

- Kiến trúc phản chiếu factor: `config` một nơi (F3), backing service sau interface (F4), composition root ở `main` (DI thủ công), health tách ba loại probe, shutdown 3 giai đoạn (F9), slog ra stdout (F11).
- **Liveness không check dependency** — thuộc lòng điều này tránh được sự cố "tự sát tập thể" kinh điển.
- Khung `run() error` + `errgroup` + context xuyên suốt: một đường thoát, các thành phần sống chết cùng nhau có trật tự.
- Go không cần DI framework, không cần web framework nặng — kỷ luật với `net/http`, `slog`, `context` là 90% trận đấu.
