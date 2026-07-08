+++
title = "Chương 24 — Case Study: Xây dựng REST API Golang chuẩn 12-Factor (kèm hành trình refactor)"
date = "2026-07-11T01:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> **Phần 4 – Thực chiến** | Chương trước: [Chủ đề Principal](/series/twelve-factor/23-chu-de-principal/) | Chương sau: [So sánh khách quan](/series/twelve-factor/25-so-sanh-khach-quan/)

Chương này ghép **mọi mảnh của 24 chương trước** thành một hệ thống hoàn chỉnh: dịch vụ đặt hàng `orderly` — REST API nhận đơn hàng, lưu PostgreSQL, cache Redis, phát sự kiện Kafka cho worker xử lý. Cấu trúc chương: hiện trạng "trước refactor" → kiến trúc đích → code đầy đủ → hạ tầng → pipeline → nhật ký refactor từng bước với lý do.

---

## 1. Điểm xuất phát: `orderly` phiên bản "tiền 12-Factor"

Hệ thống thật mà ta sẽ refactor — mọi vấn đề đều lấy từ các chương trước, giờ tập trung trong một chỗ:

```go
// main.go — PHIÊN BẢN CŨ (đừng bắt chước)
package main

const dbDSN = "postgres://app:Prod@Pass99@10.0.3.17:5432/orders" // ① F3: secret trong code

var orderCache = map[string]*Order{}   // ② F6: cache không TTL, không giới hạn — state + memory leak
var cacheMu sync.Mutex

func main() {
	db, _ := sql.Open("postgres", dbDSN)
	runMigrations(db)                   // ③ F5/F12: N instance đua migration lúc khởi động

	f, _ := os.OpenFile("/var/log/orderly.log", os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
	log.SetOutput(f)                    // ④ F11: log ra file local

	http.HandleFunc("/orders", func(w http.ResponseWriter, r *http.Request) {
		// ... tạo order, lưu DB ...
		go sendConfirmationEmail(order) // ⑤ F8/F9: job nền trong process web — deploy là mất email
		go exportToAccounting(order)    //    và không retry, không dấu vết
	})

	c := cron.New()
	c.AddFunc("0 3 * * *", cleanupExpired) // ⑥ F6/F12: in-process cron — scale là chạy N lần
	c.Start()

	http.ListenAndServe(":8080", nil)   // ⑦ F7/F9: port cứng; không graceful shutdown; không probe
}
```

Deploy hiện tại: build trên máy dev, `scp` binary lên một VM, chạy trong `screen`. Mỗi triệu chứng vận hành mà team đang chịu đều truy được về một con số ở trên: deploy nào cũng rơi vài request (⑦), muốn thêm máy thứ hai thì cron chạy đôi (⑥) và cache lệch (②), sự cố nửa đêm phải SSH đọc file log (④), bí mật lộ khi share code cho đối tác audit (①).

## 2. Kiến trúc đích

```
                        ┌─────────────── Kubernetes cluster ───────────────┐
  Internet ──▶ Ingress ─▶ Service ─▶ [web ×3..30 HPA]  ──▶ PostgreSQL (RDS)
                                        │        │     ──▶ Redis (ElastiCache)
                                        │ publish│
                                        ▼        │
                                     Kafka (MSK) │
                                        │        │
                                   [worker ×2..20 KEDA] ──▶ email, accounting...
                                                  │
              CronJob cleanup ────────────────────┘   Job migrate (release step)
              (mọi process type: CÙNG một image, khác args)
```

```
orderly/
├── go.mod / go.sum
├── cmd/app/main.go                    # server | worker | migrate | cleanup-expired
├── internal/
│   ├── config/config.go               # F3
│   ├── logging/logging.go             # F11 (slog JSON + trace_id)
│   ├── telemetry/otel.go              # ch22
│   ├── server/ (router, health, middleware)
│   ├── service/order.go               # business logic — chỉ interface
│   ├── repository/postgres/           # F4
│   ├── cache/redis.go                 # F4
│   ├── events/kafka.go                # F4 — producer + consumer
│   └── migrate/sql/                   # F12 — embed
├── deploy/
│   ├── Dockerfile
│   └── k8s/ (base + overlays)         # ch20, F10
├── docker-compose.yml                 # F10
└── .github/workflows/ci.yml           # ch21
```

## 3. Code — các mảnh ghép chính

*(Các pattern nền — khung `run()`, health, middleware, DI — đã có đầy đủ ở chương 18; ở đây chỉ hiện phần đặc thù của bài toán.)*

### 3.1. Config — bản đồ mọi backing service (F3, F4)

```go
// internal/config/config.go
type Config struct {
	Port            string        `env:"PORT" envDefault:"8080"`
	LogLevel        string        `env:"LOG_LEVEL" envDefault:"info"`
	DrainDelay      time.Duration `env:"DRAIN_DELAY" envDefault:"5s"`
	ShutdownTimeout time.Duration `env:"SHUTDOWN_TIMEOUT" envDefault:"15s"`

	DatabaseURL  string   `env:"DATABASE_URL,required"`
	RedisURL     string   `env:"REDIS_URL,required"`
	KafkaBrokers []string `env:"KAFKA_BROKERS,required"`

	CacheTTL          time.Duration `env:"CACHE_TTL" envDefault:"5m"`
	WorkerConcurrency int           `env:"WORKER_CONCURRENCY" envDefault:"8"`
}
```

### 3.2. Service — nghiệp vụ với transactional outbox

Điểm thiết kế đáng giá nhất của case study. Bài toán: lưu order vào Postgres **và** phát event Kafka — hai hệ thống, không có transaction chung. Ghi DB xong mới publish? Crash giữa chừng (F9 nói: *sẽ* xảy ra) → order có mà event mất, hệ thống downstream không bao giờ biết. Publish trước? Event có mà order fail. Giải pháp chuẩn ngành: **outbox pattern** — event ghi vào bảng `outbox` **trong cùng transaction** với order; một relay đọc outbox và đẩy sang Kafka, đánh dấu đã gửi. Crash ở bất kỳ điểm nào cũng không mất event (chỉ có thể gửi trùng → consumer idempotent, vốn đã là yêu cầu của F9).

```go
// internal/service/order.go (trích)
func (s *OrderService) Create(ctx context.Context, req CreateOrderRequest) (*Order, error) {
	if err := req.Validate(); err != nil {
		return nil, fmt.Errorf("%w: %v", ErrInvalid, err)
	}
	order := NewOrder(req)

	// Order + event: MỘT transaction — nguyên tử, chịu được crash bất kỳ lúc nào
	err := s.repo.CreateWithOutbox(ctx, order, OutboxEvent{
		Topic:   "orders.created",
		Key:     order.ID,
		Payload: mustJSON(OrderCreatedEvent{ID: order.ID, Amount: order.Amount}),
	})
	if err != nil {
		return nil, fmt.Errorf("create order: %w", err)
	}
	s.cache.Invalidate(ctx, order.UserID) // cache lỗi không fail request — chỉ log
	return order, nil
}
```

```go
// internal/repository/postgres/order.go (trích)
func (r *OrderRepo) CreateWithOutbox(ctx context.Context, o *service.Order, ev service.OutboxEvent) error {
	tx, err := r.pool.Begin(ctx)
	if err != nil {
		return err
	}
	defer tx.Rollback(ctx)

	if _, err := tx.Exec(ctx,
		`INSERT INTO orders (id, user_id, amount, status, created_at)
		 VALUES ($1,$2,$3,$4,$5)`,
		o.ID, o.UserID, o.Amount, o.Status, o.CreatedAt); err != nil {
		return err
	}
	if _, err := tx.Exec(ctx,
		`INSERT INTO outbox (topic, key, payload) VALUES ($1,$2,$3)`,
		ev.Topic, ev.Key, ev.Payload); err != nil {
		return err
	}
	return tx.Commit(ctx)
}
```

### 3.3. Cache Redis — cache-aside có kỷ luật (F4, F6)

```go
// internal/cache/redis.go (trích) — sửa lỗi ② của bản cũ:
// TTL bắt buộc, tập trung (mọi instance nhìn cùng cache), fail-open
func (c *Cache) GetOrder(ctx context.Context, id string) (*service.Order, bool) {
	b, err := c.rdb.Get(ctx, "order:"+id).Bytes()
	if err != nil {
		return nil, false // Redis chết → miss → đọc DB: cache là TỐI ƯU, không phải sự thật
	}
	var o service.Order
	if json.Unmarshal(b, &o) != nil {
		return nil, false
	}
	return &o, true
}

func (c *Cache) SetOrder(ctx context.Context, o *service.Order) {
	b, _ := json.Marshal(o)
	if err := c.rdb.Set(ctx, "order:"+o.ID, b, c.ttl).Err(); err != nil {
		c.log.Warn("cache set failed", "err", err) // không bao giờ fail request vì cache
	}
}
```

### 3.4. Worker — consumer Kafka idempotent (F8, F9)

```go
// cmd/app: case "worker" → events.RunConsumer (trích)
func (c *Consumer) handle(ctx context.Context, msg kafka.Message) error {
	var ev OrderCreatedEvent
	if err := json.Unmarshal(msg.Value, &ev); err != nil {
		c.log.Error("poison message", "offset", msg.Offset, "err", err)
		return nil // commit bỏ qua + đẩy DLQ — đừng chặn partition vì một message hỏng
	}
	// Idempotency: outbox có thể gửi trùng, Kafka giao at-least-once —
	// khóa duy nhất chặn xử lý lặp
	done, err := c.markProcessed(ctx, ev.ID) // INSERT ... ON CONFLICT DO NOTHING
	if err != nil {
		return err // lỗi hạ tầng → không commit → thử lại
	}
	if !done {
		return nil // đã xử lý rồi — bỏ qua êm
	}
	return c.sendConfirmationEmail(ctx, ev) // sửa lỗi ⑤: có retry, có dấu vết, sống sót qua deploy
}
```

### 3.5. Migration SQL

```sql
-- internal/migrate/sql/0001_init.up.sql
CREATE TABLE orders (
    id         TEXT PRIMARY KEY,
    user_id    TEXT NOT NULL,
    amount     BIGINT NOT NULL CHECK (amount > 0),
    status     TEXT NOT NULL DEFAULT 'pending',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_orders_user ON orders (user_id, created_at DESC);

CREATE TABLE outbox (
    id         BIGSERIAL PRIMARY KEY,
    topic      TEXT NOT NULL,
    key        TEXT NOT NULL,
    payload    JSONB NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    sent_at    TIMESTAMPTZ
);
CREATE INDEX idx_outbox_unsent ON outbox (id) WHERE sent_at IS NULL;

CREATE TABLE processed_events (          -- idempotency của worker
    event_id     TEXT PRIMARY KEY,
    processed_at TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

## 4. Hạ tầng

### 4.1. docker-compose — toàn hệ trên laptop trong một lệnh (F10)

```yaml
services:
  app:
    build: .
    command: ["server"]
    ports: ["8080:8080"]
    environment: &appenv
      DATABASE_URL: postgres://app:dev@db:5432/orderly?sslmode=disable
      REDIS_URL: redis://redis:6379/0
      KAFKA_BROKERS: kafka:9092
      LOG_LEVEL: debug
    depends_on:
      db: { condition: service_healthy }
      kafka: { condition: service_started }
  worker:
    build: .
    command: ["worker"]
    environment: *appenv
    depends_on: [app]
  migrate:
    build: .
    command: ["migrate"]
    environment: *appenv
    depends_on:
      db: { condition: service_healthy }
  db:
    image: postgres:17.4-alpine        # đúng version RDS (F10)
    environment: { POSTGRES_USER: app, POSTGRES_PASSWORD: dev, POSTGRES_DB: orderly }
    healthcheck: { test: ["CMD-SHELL", "pg_isready -U app"], interval: 2s, retries: 15 }
  redis:
    image: redis:7.2-alpine
  kafka:
    image: apache/kafka:3.9.0          # KRaft — không cần ZooKeeper
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: broker,controller
      # ... (cấu hình single-node chuẩn)
```

`docker compose up` → dev mới có toàn bộ hệ thống chạy trong 5 phút — chính là thước đo parity (F10) và onboarding tốt nhất.

### 4.2. Kubernetes & Dockerfile

Dockerfile: đúng bản chương 19 (multi-stage, distroless, exec form, `CMD ["server"]`). K8s: web Deployment + HPA như chương 20; worker Deployment + KEDA `ScaledObject` theo Kafka lag (chương 13); `migrate` là Job release-step (chương 10); `cleanup-expired` là CronJob `concurrencyPolicy: Forbid` (chương 17) — sửa dứt điểm lỗi ⑥. Tất cả trỏ **cùng một image digest**, khác `args`.

### 4.3. Pipeline

Đúng khung chương 21: verify (test với Postgres/Redis/Kafka service containers) → build/push/sign theo SHA → staging tự động → production sau phê duyệt, qua repo GitOps + Argo Rollouts canary (chương 23).

## 5. Nhật ký refactor — thứ tự và lý do

Bài học lớn nhất của case study không phải code đích, mà là **thứ tự đường đi** — refactor thật không được dừng hệ thống, và mỗi bước phải tự đứng vững, giao giá trị ngay:

| Bước | Việc | Sửa lỗi | Vì sao ở vị trí này |
|---|---|---|---|
| 1 | Config ra env + slog ra stdout | ①④ | Rẻ nhất, không đổi hành vi, mở khóa mọi bước sau; secret lộ là rủi ro đang chảy máu — cầm máu trước. (Kèm rotate toàn bộ credential đã từng vào Git!) |
| 2 | Dockerfile + compose + CI verify/build | F2/F5/F10 | Có artifact bất biến và môi trường tái tạo được rồi mới dám sửa tiếp — lưới an toàn cho các bước rủi ro hơn |
| 3 | Graceful shutdown + health probes | ⑦ | Nhỏ (100 dòng), chấm dứt rơi request mỗi deploy — quick win thấy được ngay, mua niềm tin cho dự án refactor |
| 4 | Migration tách thành subcommand + Job | ③ | Chuẩn bị điều kiện chạy nhiều instance |
| 5 | Cache local → Redis | ② | Điều kiện chạy nhiều instance (cùng bước 4 mở khóa scale out) |
| 6 | `go func` email/export → outbox + Kafka + worker | ⑤ | Nặng nhất, đụng nghiệp vụ — làm khi đã có CI, môi trường parity và kinh nghiệm từ 5 bước trước; bật *song song* với đường cũ, so sánh, rồi cắt |
| 7 | In-process cron → CronJob | ⑥ | Sau khi image đa vai đã tồn tại (bước 4) thì gần như miễn phí |
| 8 | Lên K8s + HPA/KEDA + GitOps + canary | — | Chỉ bây giờ — vì **app đã giữ hợp đồng**; lên K8s trước bước 3–5 là mang mọi bệnh cũ lên nền tảng mới với chi phí nhân đôi |

Điểm đáng suy ngẫm cuối: bước 1–5 chiếm ~20% công sức nhưng giao ~80% giá trị vận hành (không lộ secret, không rơi request, log tập trung, scale được). Nếu tổ chức chỉ cho bạn một quý — làm 5 bước đầu, phần còn lại để quý sau. **12-Factor là một cái thang, không phải một cánh cổng.**

---

## Tóm tắt

- Case study hội tụ mọi chương: config/env, attached resources, outbox pattern cho ranh giới DB–Kafka, consumer idempotent, cache fail-open, một image bốn vai (server/worker/migrate/cleanup), compose parity, pipeline ký + canary.
- Mẫu thiết kế đáng mang theo: **transactional outbox** (sự kiện không bao giờ mất, chỉ có thể trùng) + **idempotent consumer** (trùng thì vô hại) — cặp đôi chịu được mọi kiểu chết của F9.
- Refactor theo thang giá trị: cầm máu (secret, log) → lưới an toàn (artifact, CI, parity) → quick win (shutdown, probe) → mở khóa scale (Redis, migration) → nghiệp vụ (queue) → nền tảng (K8s). Không nhảy cóc.
