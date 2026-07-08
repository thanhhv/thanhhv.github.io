+++
title = "Chương 7 — Integration: Kafka, RabbitMQ, Redis, External API, Event Publishing"
date = "2026-07-08T06:00:00+07:00"
draft = false
tags = ["backend", "golang", "clean-architecture"]
series = ["Clean Architecture với Golang"]
+++

> **Level 3 – Senior** · Ranh giới với thế giới bên ngoài — nơi lỗi, độ trễ và bất nhất là chuyện thường ngày.

---

## 1. Problem Statement

Hệ thống thực không sống một mình: nó gọi cổng thanh toán, phát event cho hệ thống khác, cache bằng Redis, nhận lệnh từ queue. Ba rủi ro kiến trúc:

1. **Kiến thức về hệ ngoài rò vào nghiệp vụ**: use case biết tên topic Kafka, format JSON của đối tác, TTL Redis → đổi đối tác/hạ tầng là mổ nghiệp vụ.
2. **Đặc tính mạng rò vào nghiệp vụ**: retry, timeout, circuit breaker viết lẫn trong use case → quy trình nghiệp vụ chìm trong code chống lỗi.
3. **Bất nhất dữ liệu**: ghi DB xong, publish event fail — hai hệ thống từ nay lệch nhau vĩnh viễn (bài toán dual-write).

Cấu trúc lời giải cho cả ba giống nhau — cũng là cấu trúc của cả tài liệu này: **vòng trong khai báo cổng theo ngôn ngữ nghiệp vụ; adapter vòng ngoài gánh mọi đặc tính của công nghệ cụ thể.**

## 2. Gọi External API — adapter + chống lỗi ở đúng tầng

```go
// usecase/ports.go — cổng nói ngôn ngữ nghiệp vụ, không nói "HTTP"
type PaymentGateway interface {
	// Charge trả ErrPaymentDeclined (nghiệp vụ) hoặc lỗi transient (hạ tầng).
	Charge(ctx context.Context, p Payment) (PaymentReceipt, error)
}
```

```go
// adapter/stripegw/gateway.go — mọi kiến thức về Stripe ở đây
type Gateway struct {
	client  *http.Client // timeout đã cấu hình
	apiKey  string
	baseURL string
}

func (g *Gateway) Charge(ctx context.Context, p usecase.Payment) (usecase.PaymentReceipt, error) {
	body, _ := json.Marshal(stripeChargeRequest{ // DTO của Stripe
		AmountVND: p.Amount, Source: p.Token,
		IdempotencyKey: p.IdempotencyKey, // chống double-charge khi retry
	})
	req, _ := http.NewRequestWithContext(ctx, http.MethodPost, g.baseURL+"/charges", bytes.NewReader(body))
	req.Header.Set("Authorization", "Bearer "+g.apiKey)

	resp, err := g.client.Do(req)
	if err != nil {
		return usecase.PaymentReceipt{}, fmt.Errorf("stripe call: %w", err) // transient
	}
	defer resp.Body.Close()

	switch {
	case resp.StatusCode == http.StatusPaymentRequired:
		return usecase.PaymentReceipt{}, usecase.ErrPaymentDeclined // dịch sang lỗi NGHIỆP VỤ
	case resp.StatusCode >= 500:
		return usecase.PaymentReceipt{}, fmt.Errorf("stripe 5xx: %w", ErrTransient)
	}
	// ... decode receipt
}
```

Ba quyết định đáng chú ý:

- **Phân loại lỗi tại adapter**: từ chối thanh toán là *sự kiện nghiệp vụ* (use case xử lý — báo khách); 503 là *sự cố hạ tầng* (tầng resilience xử lý — retry). Trộn hai loại này là nguồn bug retry-double-charge kinh điển.
- **Resilience là decorator**, không phải code trong use case:

```go
// adapter/resilience/retry.go — bọc quanh CỔNG, use case không biết
type RetryingGateway struct {
	next    usecase.PaymentGateway
	retries int
}

func (r *RetryingGateway) Charge(ctx context.Context, p usecase.Payment) (usecase.PaymentReceipt, error) {
	var lastErr error
	for i := 0; i <= r.retries; i++ {
		receipt, err := r.next.Charge(ctx, p)
		if err == nil || !errors.Is(err, ErrTransient) {
			return receipt, err // thành công, hoặc lỗi nghiệp vụ → KHÔNG retry
		}
		lastErr = err
		select {
		case <-ctx.Done(): return usecase.PaymentReceipt{}, ctx.Err()
		case <-time.After(backoff(i)): // exponential + jitter
		}
	}
	return usecase.PaymentReceipt{}, lastErr
}
// main.go: gw := resilience.WithRetry(resilience.WithBreaker(stripegw.New(cfg)), 3)
```

- **Idempotency key sinh từ use case** (nó thuộc ngữ nghĩa nghiệp vụ: "lần thanh toán này") và truyền xuống — để retry ở mọi tầng đều an toàn.

## 3. Event Publishing và bài toán dual-write

Use case phát event qua cổng `EventPublisher` (đã thấy ở chương 2.3). Nhưng viết thẳng "save DB rồi publish Kafka" là **dual-write bug**: crash giữa hai bước → hệ thống ngoài mất event vĩnh viễn. Lời giải chuẩn production — **Transactional Outbox**:

```
┌─ Transaction DB (atomic) ─────────────┐      ┌─ Relay (goroutine/worker) ─┐
│ 1. INSERT INTO wallets ...            │      │ 3. SELECT FROM outbox       │
│ 2. INSERT INTO outbox (event JSON)    │ ───▶ │ 4. Publish Kafka            │
└───────────────────────────────────────┘      │ 5. UPDATE outbox published  │
                                               └─────────────────────────────┘
```

```go
// adapter/outbox/publisher.go — implement usecase.EventPublisher bằng cách GHI DB
// trong CÙNG transaction với nghiệp vụ (dùng Transactor ctx của chương 5).
func (p *OutboxPublisher) PointsRedeemed(ctx context.Context, e usecase.PointsRedeemedEvent) error {
	payload, _ := json.Marshal(e)
	_, err := p.exec(ctx).ExecContext(ctx,
		`INSERT INTO outbox (id, topic, payload, created_at) VALUES ($1,$2,$3,$4)`,
		uuid.NewString(), "loyalty.points_redeemed", payload, time.Now())
	return err
}

// adapter/outbox/relay.go — vòng lặp riêng đẩy outbox → Kafka, at-least-once
func (r *Relay) Run(ctx context.Context) error {
	tick := time.NewTicker(200 * time.Millisecond)
	defer tick.Stop()
	for {
		select {
		case <-ctx.Done(): return ctx.Err()
		case <-tick.C:
			if err := r.publishBatch(ctx); err != nil {
				r.log.Error("outbox relay", "err", err) // sẽ thử lại tick sau
			}
		}
	}
}
```

Vẻ đẹp kiến trúc: use case gọi `events.PointsRedeemed(ctx, e)` — **không biết** đằng sau là Kafka trực tiếp, outbox, hay in-memory bus trong test. Quyết định consistency mức nào là quyết định của composition root. Hệ quả của at-least-once: consumer phải **idempotent** (xem chương 11).

Về **schema event**: event công bố ra ngoài là **API công khai** — version hóa (`v1` trong tên topic hoặc field version), có DTO riêng, không serialize domain entity trực tiếp. Event nội bộ module có thể thoáng hơn.

## 4. Kafka vs RabbitMQ — khác biệt kiến trúc, không chỉ công nghệ

| | Kafka | RabbitMQ |
|---|---|---|
| Mô hình | Log phân tán, consumer tự giữ offset | Message broker, queue + ack từng message |
| Message sau đọc | Còn nguyên (replay được) | Mất khỏi queue |
| Phù hợp | Event stream, event sourcing, nhiều consumer độc lập, thứ tự theo partition | Task queue, work distribution, routing phức tạp (exchange), RPC |
| Ordering | Trong một partition (chọn partition key = aggregate ID!) | Một queue, một consumer |
| Kiến trúc adapter | Consumer group + commit offset thủ công (chương 6 mục 5) | Ack/nack + DLX |

Điểm ăn tiền của kiến trúc sạch: cả hai đều nằm sau cổng `EventPublisher`/consumer adapter — **quyết định chọn broker đảo ngược được**. Chọn theo bài toán: "phân phối công việc" → RabbitMQ; "công bố sự kiện cho các hệ tiêu thụ độc lập + replay" → Kafka. Chi tiết vận hành (partition key, consumer group, DLQ) sống trọn trong adapter.

## 5. Redis — cache là decorator, không phải nghiệp vụ

Cache đúng chỗ trong Clean Architecture: **decorator quanh một cổng đã có**:

```go
// adapter/rediscache/wallet_repo.go — bọc WalletRepo, use case không biết cache tồn tại
type CachedWalletRepo struct {
	next usecase.WalletRepo
	rdb  *redis.Client
	ttl  time.Duration
}

func (c *CachedWalletRepo) ByCustomer(ctx context.Context, id string) (domain.Wallet, error) {
	if raw, err := c.rdb.Get(ctx, "wallet:"+id).Bytes(); err == nil {
		if w, err := decodeWallet(raw); err == nil { return w, nil }
	}
	w, err := c.next.ByCustomer(ctx, id) // miss hoặc lỗi Redis → xuống nguồn thật
	if err != nil { return domain.Wallet{}, err }
	if raw, err := encodeWallet(w); err == nil {
		c.rdb.Set(ctx, "wallet:"+id, raw, c.ttl) // best-effort, lỗi không chặn
	}
	return w, nil
}

func (c *CachedWalletRepo) Save(ctx context.Context, w domain.Wallet) error {
	if err := c.next.Save(ctx, w); err != nil { return err }
	c.rdb.Del(ctx, "wallet:"+w.CustomerID()) // invalidate-on-write
	return nil
}
```

Lợi ích của việc cache là decorator: bật/tắt ở composition root; đo lường được (bọc thêm metric decorator); nghiệp vụ test không cần Redis; và **chính sách cache đọc được ở một chỗ** thay vì rải `rdb.Get` khắp use case. Khi Redis dùng cho nghiệp vụ thật (rate limit, distributed lock, session) — đó không còn là cache mà là một cổng riêng với contract riêng (`RateLimiter`, `LockManager`), vẫn cùng nguyên tắc.

Cảnh báo trung thực về caching: invalidation sai là nguồn bug âm ỉ khó nhất — cache-aside như trên chỉ an toàn khi chấp nhận stale ngắn; dữ liệu đòi hỏi đúng tuyệt đối (số dư!) thì **đừng cache**, hoặc cache ở tầng đọc CQRS (chương 10).

## 6. Anti-patterns

- **Use case biết topic/queue/key**: `uc.kafka.Publish("loyalty.points.v2", ...)` — tên topic là chi tiết hạ tầng, thuộc adapter/config.
- **Retry mù**: retry cả lỗi nghiệp vụ (double-charge!), retry không backoff (dập chết downstream đang hồi), retry không idempotency key.
- **Cache rải trong use case**: `if cached := ...; else { ...; cache.Set }` lặp ở 20 use case — cross-cutting concern viết như logic chính.
- **Consumer gọi thẳng SQL** bỏ qua use case — nghiệp vụ hai đường, lệch nhau dần (chương 6).
- **Event = domain entity serialize**: schema công khai đóng băng domain; thêm field nội bộ = vỡ consumer đối tác.
- **Bỏ qua dual-write** "vì hiếm khi crash" — tần suất thấp × chi phí điều tra bất nhất cực cao = đáng đầu tư outbox ngay khi event có ý nghĩa nghiệp vụ.

## 7. Khi nào đơn giản hóa

- Event chỉ dùng nội bộ một process → channel/in-memory bus, khỏi broker.
- Gọi API ngoài trong tool nội bộ → `http.Get` thẳng, khỏi gateway interface.
- Hệ thống nhỏ chưa có consumer thứ hai → gọi hàm trực tiếp thay vì event ("đừng xây bưu điện khi hai phòng cạnh nhau").

## Tóm tắt

- Mọi tích hợp = cổng (ngôn ngữ nghiệp vụ) + adapter (toàn bộ kiến thức công nghệ) + decorator (retry, breaker, cache, metric).
- Phân loại lỗi nghiệp vụ/transient tại adapter là nền của mọi chính sách retry đúng.
- Dual-write giải bằng outbox; event công khai là API — version hóa.

**Chương tiếp theo:** [Testing Strategy](/series/clean-architect/08-testing/01-testing-strategy/)
