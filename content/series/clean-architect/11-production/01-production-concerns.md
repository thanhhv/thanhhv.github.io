+++
title = "Chương 11 — Production Concerns: Logging, Config, Observability, Shutdown, Retry, Idempotency"
date = "2026-07-08T10:00:00+07:00"
draft = false
tags = ["backend", "golang", "clean-architecture"]
series = ["Clean Architecture với Golang"]
+++

> **Level 3–4** · Kiến trúc đẹp mà không vận hành được là bài tập trên giấy. Chương này trả lời: các concern vận hành nằm ở vòng nào, và viết thế nào để chúng không ăn mòn ranh giới.

Nguyên tắc chung của cả chương: **concern vận hành là cross-cutting → sống ở adapter/middleware/decorator/composition root; vòng trong cùng lắm biết *khái niệm* (context, error), không bao giờ biết *công cụ* (Prometheus, OTel, Viper).**

---

## 1. Configuration

Config là input của **composition root**, không phải của component:

```go
// internal/platform/config/config.go
type Config struct {
	HTTPAddr     string        `env:"HTTP_ADDR" default:":8080"`
	DatabaseURL  string        `env:"DATABASE_URL,required"`
	KafkaBrokers []string      `env:"KAFKA_BROKERS,required"`
	RedeemLimit  int           `env:"REDEEM_DAILY_LIMIT" default:"3"`
	ShutdownWait time.Duration `env:"SHUTDOWN_WAIT" default:"10s"`
}

func MustLoad() Config { /* env → struct, fail-fast nếu thiếu */ }
```

Kỷ luật: (a) **đọc một lần lúc khởi động, fail-fast** — thiếu config chết ngay với thông báo rõ, không chết lúc 3h sáng khi request đầu chạm code path đó; (b) component nhận **giá trị đã bóc** (`NewService(limit int)`), không nhận cả `Config` — tránh mọi component coupling vào struct config toàn cục; (c) config nghiệp vụ (RedeemLimit) đi vào domain **qua constructor như mọi dependency** — domain không import package config; (d) secret từ secret manager, cũng tại composition root.

## 2. Logging

Go 1.21+ có `log/slog` — structured logging chuẩn, đủ cho production:

```go
// Middleware HTTP: log MỌI request một kiểu — không handler nào tự log request
func LogRequests(log *slog.Logger) Middleware {
	return func(next http.Handler) http.Handler {
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			start := time.Now()
			rw := &statusRecorder{ResponseWriter: w, status: 200}
			next.ServeHTTP(rw, r)
			log.InfoContext(r.Context(), "http_request",
				"method", r.Method, "path", r.URL.Path,
				"status", rw.status, "duration_ms", time.Since(start).Milliseconds())
		})
	}
}
```

Câu hỏi kiến trúc: **use case có được log không?** Quan điểm thực dụng: có, qua `*slog.Logger` inject (hoặc interface mỏng nếu muốn tuyệt đối) — vì log quyết định nghiệp vụ ("từ chối redeem vì limit") là log *có ngữ nghĩa* mà adapter không tái tạo được. Kỷ luật kèm theo: use case log **sự kiện nghiệp vụ**, adapter log **kỹ thuật** (SQL chậm, retry); không log trùng một sự kiện ở hai tầng; correlation/trace ID chảy qua `context` và handler của slog tự đính kèm — code nghiệp vụ không bao giờ tự nhét trace ID.

Đừng log rồi vẫn return error nguyên vẹn lên trên ("log and throw") — tầng trên lại log → một lỗi 4 dòng log. Quy tắc: **lỗi được log đúng một lần, ở nơi xử lý cuối cùng** (thường middleware/consumer loop); các tầng giữa chỉ wrap thêm ngữ cảnh (`fmt.Errorf("save wallet: %w", err)`).

## 3. Error Handling — hệ thần kinh của kiến trúc

Tổng hợp các mảnh đã rải qua các chương thành một chính sách:

```go
// 1. Domain định nghĩa lỗi NGHIỆP VỤ — sentinel hoặc typed
var ErrInsufficientPoints = errors.New("loyalty: insufficient points")

type ValidationError struct{ Field, Reason string }
func (e ValidationError) Error() string { return e.Field + ": " + e.Reason }

// 2. Adapter DỊCH lỗi hạ tầng → lỗi domain (sql.ErrNoRows → ErrWalletNotFound)
//    và phân loại transient để tầng resilience quyết định retry (chương 7)

// 3. Use case WRAP với ngữ cảnh, không nuốt, không dịch
//    return fmt.Errorf("redeem for %s: %w", id, err)

// 4. Delivery dịch lỗi domain → giao thức (HTTP status, gRPC code) — bảng tập trung
// 5. Log một lần ở biên; lỗi 5xx kèm stack/trace ID, lỗi 4xx không cần alert
```

Điều phải tránh tuyệt đối: use case `return errors.New("internal error")` (nuốt nguyên nhân), hoặc handler `errors.Is(err, sql.ErrNoRows)` (hạ tầng rò qua hai tầng lên delivery).

## 4. Observability — Metrics & Tracing

Cùng một pattern decorator/middleware:

- **Metrics (Prometheus)**: RED (Rate, Errors, Duration) ở middleware HTTP/gRPC + decorator quanh cổng (repo, gateway — chương 1.4 đã minh họa). Metric nghiệp vụ (`points_redeemed_total`) phát từ **event consumer** thay vì từ use case — tận dụng event đã có, giữ use case sạch.
- **Tracing (OpenTelemetry)**: span mở ở middleware, truyền qua `context.Context` — vốn đã chảy xuyên mọi tầng. Adapter mở span con (`otelsql` bọc driver DB, interceptor gRPC, instrumented http.Client). Use case *có thể* mở span nghiệp vụ nếu team muốn — qua interface mỏng `Tracer` — nhưng 80% giá trị nằm ở auto-instrument tầng adapter, miễn phí với vòng trong.

`context.Context` chính là "đường ống cross-cutting" hợp pháp duy nhất xuyên các vòng: cancellation, deadline, trace ID. Kỷ luật: **không nhét business data vào context** (userID cho authorization là tham số tường minh, không phải context value).

## 5. Health Check & Graceful Shutdown

```go
// /healthz — liveness: process sống. /readyz — readiness: sẵn sàng nhận traffic.
mux.HandleFunc("GET /healthz", func(w http.ResponseWriter, _ *http.Request) {
	w.WriteHeader(http.StatusOK)
})
mux.HandleFunc("GET /readyz", func(w http.ResponseWriter, r *http.Request) {
	ctx, cancel := context.WithTimeout(r.Context(), 2*time.Second)
	defer cancel()
	if err := db.PingContext(ctx); err != nil { // + kafka, redis...
		http.Error(w, "db: "+err.Error(), http.StatusServiceUnavailable)
		return
	}
	w.WriteHeader(http.StatusOK)
})
```

Shutdown đúng thứ tự — ngược với khởi động, và **ngừng nhận trước, xử nốt sau, đóng kết nối cuối**:

```go
func run(ctx context.Context, cfg Config) error {
	ctx, stop := signal.NotifyContext(ctx, syscall.SIGINT, syscall.SIGTERM)
	defer stop()

	// ... khởi tạo db, kafka, server, consumer (errgroup quản lý vòng đời)
	g, gctx := errgroup.WithContext(ctx)
	g.Go(func() error { return consumer.Run(gctx) })
	g.Go(func() error {
		<-gctx.Done() // nhận tín hiệu dừng
		shCtx, cancel := context.WithTimeout(context.Background(), cfg.ShutdownWait)
		defer cancel()
		return srv.Shutdown(shCtx) // 1. ngừng nhận request mới, đợi in-flight
	})
	g.Go(func() error {
		if err := srv.ListenAndServe(); err != http.ErrServerClosed { return err }
		return nil
	})
	err := g.Wait()  // 2. consumer thoát khi ctx hủy (commit offset nốt)
	db.Close()       // 3. đóng kết nối sau cùng
	return err
}
```

Toàn bộ nằm ở vòng 4 (`cmd/`) — use case không biết mình đang "được shutdown"; nó chỉ tôn trọng `ctx.Done()` như mọi khi. Kubernetes lưu ý: readiness fail *trước* khi SIGTERM xử xong giúp LB rút traffic sớm — thêm cờ `shuttingDown` cho `/readyz`.

## 6. Retry & Idempotency — cặp bài trùng

Retry đã dựng ở chương 7 (decorator, chỉ transient, backoff + jitter, tôn trọng ctx). Nửa còn lại: **mọi retry đều tạo nguy cơ thực hiện hai lần** — at-least-once là mặc định của thế giới phân tán. Idempotency là câu trả lời, và nó **thuộc thiết kế nghiệp vụ**, không chỉ kỹ thuật:

```go
// Consumer idempotent: bảng processed_events trong CÙNG transaction với side-effect
func (p *Projector) OnOrderPlaced(ctx context.Context, e OrderPlaced) error {
	return p.tx.WithinTx(ctx, func(ctx context.Context) error {
		inserted, err := p.markProcessed(ctx, e.EventID) // INSERT ... ON CONFLICT DO NOTHING
		if err != nil { return err }
		if !inserted { return nil } // đã xử lý — bỏ qua an toàn
		return p.applyProjection(ctx, e)
	})
}
```

```go
// API idempotent: client gửi Idempotency-Key; server lưu (key → kết quả)
// Request lặp với cùng key → trả kết quả cũ, KHÔNG chạy lại nghiệp vụ.
// Key sinh ở nơi hiểu ngữ nghĩa "một lần" — thường là client/use case (chương 7).
```

Checklist idempotency cho từng loại thao tác: ghi trạng thái tuyệt đối (`SET balance = x`) — tự idempotent; ghi tương đối (`balance += x`) — cần event ID dedupe; gọi API ngoài có tiền — bắt buộc idempotency key; gửi email — dedupe theo business key (đơn X chỉ 1 mail xác nhận).

## 7. Anti-patterns

- **`log.Fatal` trong thư viện/component** — giết process từ nơi không có quyền quyết định; chỉ `main` được fatal.
- **Component tự đọc env** (`os.Getenv` trong repo constructor) — config vương vãi, test phải set env, 12-factor giả.
- **Metric/trace code rải trong use case** — 5 dòng đo cho 3 dòng nghiệp vụ; dùng decorator.
- **Retry ở 3 tầng chồng nhau** (http.Client retry × gateway retry × consumer redelivery = 27 lần gọi) — chọn MỘT tầng sở hữu retry cho mỗi đường đi, thường tầng ngoài nhất hiểu ngữ nghĩa.
- **Health check "sâu" gọi cả downstream của downstream** — một service cảm cúm, cả cụm readiness fail dây chuyền. Readiness chỉ kiểm dependency *trực tiếp và thiết yếu*.
- **Nuốt ctx.Done()**: vòng lặp worker không check ctx — shutdown treo đến khi bị SIGKILL, mất message đang xử lý.

## Tóm tắt

- Mọi concern vận hành có một chỗ đúng: config → composition root; log kỹ thuật/metrics/tracing → middleware + decorator; log nghiệp vụ → use case với kỷ luật; shutdown → cmd/; retry → decorator quanh cổng; idempotency → thiết kế nghiệp vụ + dedupe tại consumer.
- `context.Context` là đường ống hợp pháp duy nhất xuyên tầng — cancellation và trace, không phải business data.
- Lỗi: domain định nghĩa, adapter dịch hai chiều, wrap ở giữa, log một lần ở biên.

**Chương tiếp theo:** [Ví dụ tổng hợp — Xây hệ thống E-commerce từng bước](/series/clean-architect/12-vi-du-ecommerce/01-giai-doan-1-monolith/)
