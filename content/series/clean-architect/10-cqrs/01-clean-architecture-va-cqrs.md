+++
title = "Chương 10 — Clean Architecture & CQRS: Đường ghi và đường đọc"
date = "2026-07-08T09:00:00+07:00"
draft = false
tags = ["backend", "golang", "clean-architecture"]
series = ["Clean Architecture với Golang"]
+++

> **Level 3 – Senior** · CQRS thường bị hiểu là "kiến trúc phức tạp với event sourcing và 2 database". Bản chất nó khiêm tốn và hữu ích hơn nhiều.

---

## 1. Problem Statement

Hai áp lực kéo model về hai hướng ngược nhau:

- **Đường ghi (command)** cần model giàu: aggregate, bất biến, rule — `Order.AddLine()` phải kiểm tra trạng thái, tính lại tổng (chương 9).
- **Đường đọc (query)** cần dữ liệu phẳng, join sẵn, đúng hình dạng màn hình: "danh sách đơn + tên khách + số item + trạng thái thanh toán, phân trang, lọc theo 5 tiêu chí".

Ép một model phục vụ cả hai tạo ra hai loại đau: (a) đường đọc **qua aggregate**: load N aggregate đầy đủ + N query phụ để render một cái bảng — chậm và code repository mọc `FindAllWithCustomerNameAndPaymentStatus(...)` vô tận; (b) aggregate **phình theo nhu cầu đọc**: thêm field chỉ để hiển thị, bất biến loãng dần.

CQRS (Command Query Responsibility Segregation — Greg Young) giải bằng cách thừa nhận: **ghi và đọc là hai bài toán khác nhau, cho phép chúng hai model khác nhau.**

## 2. Bản chất và các mức độ

CQRS có thang mức — và mức 1 đã giải quyết 90% vấn đề với 10% chi phí:

- **Mức 0**: một model, một repository cho cả hai. Đủ cho CRUD.
- **Mức 1 — tách code path, chung database**: command đi qua aggregate + repository; query đi **thẳng SQL → DTO**, bỏ qua domain model. Đây là mức khuyến nghị mặc định.
- **Mức 2 — read model vật lý riêng** (bảng chiếu/materialized view, cập nhật qua domain event): khi query phức tạp hoặc tải đọc lớn.
- **Mức 3 — datastore đọc riêng** (Elasticsearch cho search, Redis cho feed): khi công nghệ đọc chuyên dụng cần thiết. Eventual consistency thành hợp đồng công khai với UI.
- (Event Sourcing là quyết định **độc lập** — hay bị buộc chung với CQRS nhưng không bắt buộc; đa số hệ thống dùng CQRS mức 1–2 không cần ES.)

Vì sao tách query khỏi domain model là *hợp lệ* theo Clean Architecture: Dependency Rule bảo vệ **business rule**. Query hiển thị **không chứa rule** — nó là chiếu dữ liệu. Cho nó đi tắt không làm rò rule ra ngoài; ngược lại nó *bảo vệ* domain model khỏi bị nhu cầu hiển thị làm biến dạng.

## 3. Mức 1 trong Go — cấu trúc cụ thể

```
internal/order/
├── domain/            # aggregate Order — CHỈ đường ghi dùng
├── command/           # use case ghi: PlaceOrder, CancelOrder
│   └── place_order.go
├── query/             # đường đọc: SQL thẳng → DTO màn hình
│   └── order_list.go
└── adapter/...
```

```go
// command/place_order.go — đường ghi: đầy đủ nghi thức domain
func (h *PlaceOrderHandler) Handle(ctx context.Context, cmd PlaceOrder) (OrderID, error) {
	ord, err := domain.NewOrder(cmd.CustomerID, toLines(cmd.Items)) // rule chạy ở đây
	if err != nil { return "", err }
	if err := h.orders.Save(ctx, ord); err != nil { return "", err }
	return ord.ID(), nil
}
```

```go
// query/order_list.go — đường đọc: không domain model, không repository interface cầu kỳ
package query

type OrderListItem struct { // DTO đúng hình dạng màn hình
	OrderID      string    `json:"order_id"`
	CustomerName string    `json:"customer_name"`
	ItemCount    int       `json:"item_count"`
	TotalVND     int64     `json:"total_vnd"`
	Status       string    `json:"status"`
	PlacedAt     time.Time `json:"placed_at"`
}

type OrderList struct{ db *sql.DB } // que phụ thuộc thẳng DB — chấp nhận có chủ đích

func (q *OrderList) ByCustomer(ctx context.Context, customerID string, page Page) ([]OrderListItem, error) {
	rows, err := q.db.QueryContext(ctx, `
		SELECT o.id, c.full_name, count(l.id), o.total, o.status, o.placed_at
		FROM orders o
		JOIN customers c ON c.id = o.customer_id
		LEFT JOIN order_lines l ON l.order_id = o.id
		WHERE o.customer_id = $1
		GROUP BY o.id, c.full_name
		ORDER BY o.placed_at DESC
		LIMIT $2 OFFSET $3`,
		customerID, page.Size, page.Offset())
	// ... scan vào []OrderListItem
}
```

Điểm cần trung thực: `query` package phụ thuộc `*sql.DB` — **một ngoại lệ có chủ đích** của Dependency Rule, đổi lại: query nhanh (một câu SQL join thay vì N+1 qua aggregate), DTO tự do theo màn hình, domain model giữ được kỷ luật. Ngoại lệ được kiểm soát bằng hai điều kiện: query **chỉ SELECT** (không ghi — mọi ghi qua command), và query **không chứa rule** (chỉ lọc/chiếu/sắp). Nếu muốn test handler đọc bằng fake, thêm interface `OrderListQuery` ở phía handler — vẫn consumer-side như mọi khi.

## 4. Mức 2 — Read model cập nhật bằng event

Khi trang "dashboard người bán" join 7 bảng và chịu 100× tải của đường ghi:

```
Command ──▶ Aggregate ──▶ Save + OrderPlaced event (outbox)
                                    │
                                    ▼
                        Projector (consumer)
                                    │  UPDATE seller_dashboard
                                    ▼      (bảng phi chuẩn hóa)
Query  ◀──────────────  SELECT * FROM seller_dashboard WHERE ...
```

```go
// query/projector.go — consumer dựng bảng chiếu
func (p *DashboardProjector) OnOrderPlaced(ctx context.Context, e domain.OrderPlaced) error {
	_, err := p.db.ExecContext(ctx, `
		INSERT INTO seller_dashboard (seller_id, orders_today, revenue_today)
		VALUES ($1, 1, $2)
		ON CONFLICT (seller_id) DO UPDATE
		SET orders_today = seller_dashboard.orders_today + 1,
		    revenue_today = seller_dashboard.revenue_today + $2`,
		e.SellerID, e.TotalVND)
	return err // idempotency: xem chương 11 — event có ID, bảng processed_events
}
```

Cái giá phải trả và phải nói to với stakeholder: **đọc sẽ trễ so với ghi** (thường ms→giây). UI phải thiết kế cho điều đó (optimistic update, "đơn của bạn đang xử lý"). Nếu nghiệp vụ đòi read-your-own-write tuyệt đối ở mọi nơi — mức 2 sai chỗ.

## 5. Trade-off

| | Mức 0 | Mức 1 | Mức 2–3 |
|---|---|---|---|
| Độ phức tạp | Thấp nhất | +1 quy ước (2 thư mục) | +projector, +eventual consistency, +vận hành |
| Hiệu năng đọc | Kém khi màn hình phức tạp | Tốt (SQL tự do) | Tốt nhất (phi chuẩn hóa/công nghệ riêng) |
| Domain model | Bị nhu cầu đọc kéo | Sạch | Sạch |
| Consistency | Mạnh | Mạnh | Eventual — phải thiết kế UI/nghiệp vụ theo |
| Khi nào | CRUD | **Mặc định cho hệ có domain** | Có số đo chứng minh cần |

Quy tắc leo thang: **chỉ leo khi đau thật, đo được** — đừng dựng projector vì "sau này chắc cần". Mức 1 → 2 là refactor cục bộ (thêm bảng chiếu + projector, query đổi nguồn) chính nhờ mức 1 đã tách code path sẵn.

## 6. Anti-patterns

- **Query qua use case + repository + aggregate cho màn hình danh sách** — N+1, DTO mapping 3 tầng, `Repository.FindByXAndYAndZ` bùng nổ. Nguyên nhân: hiểu Dependency Rule máy móc.
- **Command trả dữ liệu màn hình** ("PlaceOrder trả luôn danh sách đơn cho tiện") — command lại gánh nhu cầu đọc, tách rồi như chưa tách.
- **Ghi qua đường query** — "chỉ UPDATE cái cờ thôi mà" — bất biến của aggregate bị vòng qua; mọi ghi phải qua command.
- **Read model chứa rule**: projector tính "khách VIP nếu chi > X" — rule trốn khỏi domain, không test được, hai nơi hai kết quả. Rule tính ở domain, event mang kết quả.
- **Mở CQRS mức 3 + event sourcing ngày đầu** vì bài blog — chi phí vận hành ăn cả team trước khi có user.

## Tóm tắt

- CQRS bản chất: cho ghi và đọc hai model, vì hai bài toán khác nhau.
- Mức 1 (tách code path, chung DB) là mặc định tốt: command qua aggregate, query SQL thẳng ra DTO — ngoại lệ Dependency Rule có chủ đích và có điều kiện.
- Leo mức 2–3 khi có số đo; mang theo eventual consistency như một quyết định sản phẩm, không chỉ kỹ thuật.

**Chương tiếp theo:** [Production Concerns](/series/clean-architect/11-production/01-production-concerns/)
