+++
title = "Chương 9 — Factor 4: Backing Services"
date = "2026-07-10T10:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> *"Treat backing services as attached resources"* — Coi mọi backing service là tài nguyên gắn kèm.
>
> **Phần 2 – The Twelve Factors** | Nhóm: **Configurable** | Chương trước: [Config](/series/twelve-factor/08-factor-03-config/) | Chương sau: [Build, Release, Run](/series/twelve-factor/10-factor-05-build-release-run/)

---

## 1. Problem Statement

**Backing service** = bất kỳ dịch vụ nào app tiêu thụ qua mạng để hoạt động: database (PostgreSQL, MySQL), cache (Redis, Memcached), message broker (Kafka, RabbitMQ), object storage (S3, MinIO), SMTP, dịch vụ bên thứ ba (payment gateway, Maps API), và cả **các service khác trong chính hệ thống của bạn**.

Factor này yêu cầu: app **không phân biệt** service do mình tự vận hành với service của bên thứ ba, và không phân biệt "con Postgres trên docker-compose ở local" với "RDS Multi-AZ trên production". Tất cả chỉ là **tài nguyên (resource) gắn vào app qua một địa chỉ + credential nằm trong config** — gắn vào, tháo ra, thay thế được mà **không đổi một dòng code**.

Nếu không có nguyên tắc này:

- Muốn nâng cấp Postgres 15 → 17: dựng cụm mới, nhưng app trỏ cứng vào cụm cũ trong code → phải sửa code + build + deploy để... đổi một địa chỉ.
- DB production gặp sự cố phần cứng: thao tác đúng là "tháo resource hỏng, gắn resource thay thế từ backup" trong vài phút; với app coupling chặt, đó là một chiến dịch sửa code lúc nửa đêm.
- Test tự động phải chạy trên DB "thật" dùng chung → test phá dữ liệu của nhau, không chạy song song được.
- Muốn chuyển từ RabbitMQ tự vận hành sang managed service → dự án refactor nhiều tháng thay vì một thay đổi config.

## 2. Tại sao nguyên lý này tồn tại

- **Operational**: tình huống vận hành phổ biến nhất với backing service là *swap* — thay node hỏng, failover sang replica, nâng version, chuyển vùng. Swap phải là thao tác config (giây/phút), không phải thao tác code (giờ/ngày).
- **Deployment**: mỗi deploy (dev/staging/prod/preview-per-PR) cần bộ resource riêng của nó. Chỉ khả thi khi "resource nào" là tham số.
- **Business**: đàm phán lại với vendor (đổi nhà cung cấp email, payment, cloud) không nên bị chặn bởi chi phí kỹ thuật của việc đổi — loose coupling giữ quyền mặc cả cho business.
- **Scalability**: đây chính là *nơi state sống* trong kiến trúc stateless (F6). Process vứt bỏ được **vì** state đã nằm an toàn trong các resource gắn kèm. F4 và F6 là hai mặt của một đồng xu.

## 3. Bản chất

Bản chất của factor này là một nguyên lý thiết kế phần mềm kinh điển được áp vào tầng kiến trúc hệ thống: **phụ thuộc vào abstraction (giao thức + địa chỉ), không phụ thuộc vào implementation (con server cụ thể)**.

```
   App ──[giao thức Postgres]──▶ postgres://user:pw@HOST/db  ← config quyết định
                                        │
              ┌─────────────────────────┼──────────────────────────┐
              ▼                         ▼                          ▼
      docker-compose local        RDS staging              Cụm HA on-prem prod
      (dev của Hùng)              (nhỏ, rẻ)                (Multi-AZ, PITR)

   Cùng code. Cùng image. Khác duy nhất: giá trị DATABASE_URL.
```

Ba mức độ "coi là attached resource", từ bắt buộc đến nâng cao:

1. **Mức bắt buộc — địa chỉ qua config** (giao nhau với F3): không hostname/IP/credential nào của resource xuất hiện trong code.
2. **Mức nên có — che sau interface trong code**: tầng nghiệp vụ nói chuyện với `OrderRepository`, không nói chuyện với `*pgxpool.Pool`. Cho phép swap *loại* resource (Postgres → CockroachDB) và test bằng implementation giả.
3. **Mức cảnh giác — kỷ luật với tính di động**: dùng tính năng độc quyền của một resource (S3 event notification, Postgres advisory lock, Kafka exactly-once) là quyết định có ý thức về lock-in, được ghi lại, không phải trượt vào vô thức.

**Điều gì xảy ra nếu vi phạm?** App và resource trở thành một khối: không swap được lúc sự cố (kéo dài downtime), không tạo được môi trường mới nhanh (chặn preview env, chặn onboarding), không test được cô lập (chặn CI), và mọi quyết định hạ tầng của team khác (đổi DB, dời vùng) đều thành việc sửa code của team bạn.

## 4. Cách áp dụng với Go

### 4.1. Kết nối qua URL từ config + che sau interface

```go
// internal/repository/order.go — tầng nghiệp vụ chỉ biết interface
package repository

import "context"

type Order struct {
	ID     string
	UserID string
	Amount int64
}

type OrderRepository interface {
	Create(ctx context.Context, o *Order) error
	GetByID(ctx context.Context, id string) (*Order, error)
}
```

```go
// internal/repository/postgres/order.go — MỘT implementation, thay được
package postgres

import (
	"context"
	"fmt"

	"github.com/jackc/pgx/v5/pgxpool"
	"myapp/internal/repository"
)

type OrderRepo struct{ pool *pgxpool.Pool }

// NewPool: địa chỉ resource đến từ config — KHÔNG có hostname nào trong code
func NewPool(ctx context.Context, databaseURL string) (*pgxpool.Pool, error) {
	cfg, err := pgxpool.ParseConfig(databaseURL)
	if err != nil {
		return nil, fmt.Errorf("parse DATABASE_URL: %w", err)
	}
	cfg.MaxConns = 20 // pool tuning cũng nên đọc từ config nếu khác theo môi trường
	pool, err := pgxpool.NewWithConfig(ctx, cfg)
	if err != nil {
		return nil, err
	}
	// Fail fast: resource không gắn được thì app không nên nhận traffic
	if err := pool.Ping(ctx); err != nil {
		return nil, fmt.Errorf("ping database: %w", err)
	}
	return pool, nil
}

func NewOrderRepo(pool *pgxpool.Pool) *OrderRepo { return &OrderRepo{pool: pool} }

func (r *OrderRepo) Create(ctx context.Context, o *repository.Order) error {
	_, err := r.pool.Exec(ctx,
		`INSERT INTO orders (id, user_id, amount) VALUES ($1, $2, $3)`,
		o.ID, o.UserID, o.Amount)
	return err
}

func (r *OrderRepo) GetByID(ctx context.Context, id string) (*repository.Order, error) {
	var o repository.Order
	err := r.pool.QueryRow(ctx,
		`SELECT id, user_id, amount FROM orders WHERE id = $1`, id).
		Scan(&o.ID, &o.UserID, &o.Amount)
	if err != nil {
		return nil, err
	}
	return &o, nil
}
```

```go
// cmd/server/main.go (trích) — wiring: gắn resource vào app lúc khởi động
pool, err := postgres.NewPool(ctx, cfg.DatabaseURL)
if err != nil {
	log.Fatalf("attach database: %v", err)
}
defer pool.Close()

orderRepo := postgres.NewOrderRepo(pool)       // implementation thật
orderSvc := service.NewOrderService(orderRepo) // nghiệp vụ chỉ thấy interface
```

### 4.2. Resilience — vì backing service là network call

Coi resource là "gắn qua mạng" nghĩa là chấp nhận: nó **sẽ** chậm, chớp nhờn, đứt. Code production cần tối thiểu:

```go
// Timeout cho MỌI lời gọi resource — không có ngoại lệ
ctx, cancel := context.WithTimeout(ctx, 2*time.Second)
defer cancel()
order, err := repo.GetByID(ctx, id)

// Retry có backoff + jitter cho lỗi tạm thời (chỉ với thao tác idempotent!)
// Circuit breaker cho resource hay hỏng (sony/gobreaker, failsafe-go)
// Health check của app phản ánh trạng thái resource (chương 18):
//   - liveness KHÔNG phụ thuộc DB (DB chết mà restart app thì vô ích)
//   - readiness CÓ THỂ phụ thuộc DB (DB chết → ngừng nhận traffic mới)
```

### 4.3. docker-compose — local dùng resource "cùng loài, khác cỡ"

```yaml
# docker-compose.yml: dev gắn Postgres/Redis/MinIO local —
# cùng giao thức với RDS/ElastiCache/S3 trên prod (phục vụ luôn F10)
services:
  app:
    build: .
    environment:
      DATABASE_URL: postgres://app:dev@db:5432/app?sslmode=disable
      REDIS_URL: redis://redis:6379/0
      S3_ENDPOINT: http://minio:9000        # prod: để trống → SDK dùng AWS thật
    depends_on:
      db: { condition: service_healthy }
  db:
    image: postgres:17-alpine
    environment: { POSTGRES_USER: app, POSTGRES_PASSWORD: dev, POSTGRES_DB: app }
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app"]
      interval: 2s
      retries: 15
  redis:
    image: redis:7-alpine
  minio:
    image: minio/minio
    command: server /data
```

### 4.4. Kubernetes — resource là địa chỉ trong Secret, thường nằm ngoài cluster

```yaml
# Backing service stateful (DB) thường KHÔNG chạy trong cùng cluster với app
# (managed service hoặc cụm riêng). App chỉ cần biết địa chỉ:
apiVersion: v1
kind: Secret
metadata: { name: myapp-secrets }
stringData:
  DATABASE_URL: postgres://app:xxx@myapp-prod.abc123.ap-southeast-1.rds.amazonaws.com:5432/app
---
# Với service nội bộ trong cluster, "địa chỉ" là DNS của K8s Service:
#   REDIS_URL: redis://redis-master.data.svc.cluster.local:6379/0
# → service discovery của K8s chính là hiện thực hóa Factor 4 ở tầng nền tảng.
```

## 5. Trade-off

**Interface hóa mọi thứ vs pragmatism.** Mức 2 (che sau interface) có chi phí: thêm tầng gián tiếp, thêm code. Đáng giá với resource *có khả năng bị swap loại* hoặc *cần mock trong test* (DB, queue, external API). Không đáng với thứ chắc chắn không đổi (đừng viết `LoggerInterface` bọc `slog`). Kinh nghiệm: interface hẹp, đặt ở phía **người dùng** interface (consumer-side, đúng idiom Go), khai báo đúng method cần dùng.

**Tính di động vs sức mạnh của resource cụ thể.** Viết SQL "chuẩn ANSI để dễ đổi DB" thường là tối ưu cho một tương lai không đến, đổi bằng việc từ chối sức mạnh thật (JSONB, window function, LISTEN/NOTIFY của Postgres). Khuyến nghị thực dụng: **di động ở tầng địa chỉ (bắt buộc, miễn phí), thực dụng ở tầng tính năng (dùng hết sức mạnh của resource đã chọn, ghi nhận lock-in một cách có ý thức)**.

**Managed service vs tự vận hành.** RDS/ElastiCache đắt hơn EC2 tự cài 1.5–3 lần trên giá niêm yết, nhưng rẻ hơn rất nhiều khi tính chi phí kỹ sư vận hành (backup, failover, vá lỗi, on-call). Với đa số team, managed là mặc định đúng; tự vận hành hợp lý khi ở quy mô rất lớn (chi phí license/markup vượt lương đội chuyên trách) hoặc có yêu cầu đặc thù. Chú ý: chính vì app coi resource là attached, quyết định này **đổi chiều được** — đó là giá trị của factor.

## 6. Best Practices

- Mọi resource định danh bằng URL/DSN trong config; một resource một biến (`DATABASE_URL`, `REDIS_URL`, `KAFKA_BROKERS`).
- Fail fast lúc start (ping resource bắt buộc), nhưng cân nhắc *lazy + readiness* cho resource không thiết yếu (app vẫn nên phục vụ được khi email service chết).
- Timeout mọi lời gọi; retry có backoff chỉ với thao tác idempotent; circuit breaker cho dependency hay lỗi.
- Connection pool có giới hạn và được tính toán (`MaxConns × số instance` không được vượt `max_connections` của DB — lỗi tràn connection khi scale out là kinh điển).
- Test: unit test dùng fake/mock qua interface; integration test dùng resource thật ephemeral (testcontainers-go) — không dùng chung DB test.
- Phân biệt resource *thiết yếu* (DB — readiness fail khi mất) và *không thiết yếu* (email — degrade, ghi queue chờ) ngay trong thiết kế.

## 7. Anti-patterns

- **Hardcode địa chỉ resource trong code** — dạng thô nhất; swap resource đòi hỏi build lại.
- **App "ôm" database**: gọi thẳng `*pgxpool.Pool` từ handler HTTP, SQL rải khắp codebase — không mock được, không swap được, không nhìn thấy ranh giới resource.
- **Hai app dùng chung một database** (integration qua DB) — schema thành API công khai không version; app A đổi cột làm sập app B; không ai dám migrate. Giao tiếp giữa các app phải qua API/queue của nhau — khi đó app kia trở thành backing service đúng nghĩa.
- **Dev/staging/prod dùng chung một resource** ("đỡ tốn") — test ghi đè dữ liệu thật; một câu `DELETE` nhầm môi trường thành sự cố production. Mỗi deploy một bộ resource.
- **Không timeout** — một resource treo kéo sập toàn bộ worker pool của app (thread starvation), biến sự cố cục bộ thành sự cố toàn hệ thống.
- **Chôn logic vào resource** (stored procedure phức tạp, trigger nghiệp vụ) khi không có lý do hiệu năng đo được — logic thoát khỏi codebase (F1), thoát khỏi CI (F5), và trói chặt vào implementation cụ thể.

## 8. Khi nào KHÔNG cần áp dụng đầy đủ

- **SQLite nhúng cho internal tool / desktop app / edge**: database *trong* process, không phải backing service — hợp lệ và thường là lựa chọn đúng. Điều kiện đảo chiều: cần instance thứ hai → chuyển client-server DB.
- **Hệ thống một máy có chủ đích** (đã phân tích chương 1): file local thay object storage chấp nhận được — miễn ghi nhận giới hạn.
- **Mức 2 (interface hóa)** bỏ qua được ở script nhỏ, prototype — nhưng mức 1 (địa chỉ qua config) thì gần như luôn đáng giữ vì miễn phí.

---

## Tóm tắt

- Backing service = mọi thứ app tiêu thụ qua mạng; tất cả là **tài nguyên gắn kèm qua URL trong config** — gắn, tháo, swap không đổi code.
- Ba mức: địa chỉ qua config (bắt buộc) → che sau interface (nên) → kỷ luật với lock-in tính năng (ý thức).
- Vì resource là network call: timeout mọi nơi, retry idempotent, circuit breaker, phân biệt resource thiết yếu/không thiết yếu trong readiness.
- Anti-pattern chí mạng: hai app chung một DB, và ba môi trường chung một resource.
- F4 + F6 là cặp bài trùng: process stateless được **vì** state sống trong attached resources.
