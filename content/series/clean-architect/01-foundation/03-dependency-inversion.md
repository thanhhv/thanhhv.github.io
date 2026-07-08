+++
title = "Chương 1.3 — Dependency Inversion: Cỗ máy đảo chiều phụ thuộc"
date = "2026-07-07T21:00:00+07:00"
draft = false
tags = ["backend", "golang", "clean-architecture"]
series = ["Clean Architecture với Golang"]
+++

> **Level 1–2 Bridge** · Đây là chương kỹ thuật then chốt. Toàn bộ Clean Architecture đứng trên một cơ chế duy nhất được trình bày ở đây.

---

## 1. Problem Statement

### Bài toán

Business logic là tài sản quý nhất của hệ thống: nó mã hóa cách doanh nghiệp kiếm tiền, nó sống 10 năm trong khi framework HTTP sống 3 năm và fashion database đổi mỗi 5 năm. Vấn đề: **theo cách viết code tự nhiên, thứ quý nhất lại phụ thuộc vào thứ dễ thay đổi nhất.**

```go
package order // business logic

import (
	"myapp/internal/postgres"   // ← business biết Postgres
	"myapp/internal/smtpmail"   // ← business biết SMTP
)

func (s *Service) PlaceOrder(...) error {
	// logic quý giá... bị hàn chết vào hạ tầng
}
```

Hệ quả nếu không giải quyết:

- **Không thể unit test nghiệp vụ** — mọi test kéo theo DB, SMTP.
- **Không thể thay hạ tầng** — migrate Postgres → CockroachDB nghĩa là mổ vào code nghiệp vụ.
- **Không thể tái sử dụng nghiệp vụ** — muốn gọi PlaceOrder từ Kafka consumer thay vì HTTP? Logic dính đầy mảnh HTTP.
- **Thay đổi hạ tầng lan vào nghiệp vụ** — upgrade driver, đổi schema đều risk vỡ business rule.

### Cách tiếp cận cũ và hạn chế

**Layered Architecture cổ điển** (Presentation → Business → Data Access) đã nhận ra cần tách tầng, nhưng giữ nguyên hướng phụ thuộc chảy xuống: Business import Data Access. Tách tầng mà không đảo phụ thuộc chỉ *sắp xếp* coupling chứ không *cắt* nó — business vẫn biết hạ tầng, vẫn không test được độc lập, database vẫn là trung tâm hấp dẫn của toàn hệ thống.

DIP là mảnh ghép còn thiếu: giữ layer, nhưng **bẻ ngược mũi tên compile-time**.

## 2. Tại sao DIP tồn tại

- **Business Problem**: bảo vệ khoản đầu tư lớn nhất (business logic) khỏi vòng đời ngắn của công nghệ; giữ chi phí đổi công nghệ ở mức "viết một adapter" thay vì "viết lại hệ thống".
- **Technical Problem**: test nghiệp vụ phải chạy nhanh (mili-giây) và deterministic — bất khả thi khi nghiệp vụ import driver.
- **Design Problem**: cần một cơ chế để *chính sách* (policy — cái gì, khi nào) và *cơ chế* (mechanism — bằng cách nào) tiến hóa độc lập.

---

## 3. Giải thích bản chất

### 3.1 Tách hai khái niệm bị nhập nhằng: control flow vs source dependency

Đây là insight trung tâm, đáng đọc chậm:

- **Control flow (runtime)**: lúc chạy, hàm nào gọi hàm nào. `Service.PlaceOrder` *sẽ luôn* gọi xuống code lưu Postgres — điều này không thể và không cần thay đổi.
- **Source dependency (compile-time)**: package nào *biết tên* package nào qua `import`.

Lập trình viên thường mặc định hai luồng này trùng nhau: muốn gọi thì phải import. **Polymorphism (interface) phá vỡ sự mặc định đó**: bạn có thể gọi một hàm mà không import package chứa nó — chỉ cần gọi qua interface do chính bạn định nghĩa.

```
TRƯỚC (tự nhiên):                       SAU (đảo chiều):

 order ──import──▶ postgres              order ◀──import── postgres
   │                  ▲                    │ khai báo          │ implement
   └── gọi Save() ────┘                    │ OrderStore        │ OrderStore
                                           ▼                   │
 control flow  ──▶ trùng                 control flow runtime VẪN:
 source dep    ──▶ trùng                 order ──gọi qua interface──▶ postgres
```

Control flow không đổi. Chỉ có **kiến thức** đổi chỗ: trước đây `order` biết `postgres`; giờ `postgres` biết `order`, còn `order` không biết gì.

### 3.2 Vì sao hướng này đúng? — Stable Dependencies

Chương 1.1 đã thiết lập: phụ thuộc phải trỏ về phía ổn định. Xét độ ổn định thực nghiệm trong một hệ thống 5 năm tuổi:

| Thành phần | Tần suất thay đổi | Lý do đổi |
|---|---|---|
| Business rule cốt lõi ("đơn > 500k freeship") | Thấp nhất | Chỉ đổi khi doanh nghiệp đổi chính sách |
| Use case (quy trình đặt hàng) | Thấp | Đổi khi quy trình đổi |
| Schema DB, driver, ORM | Trung bình–cao | Tối ưu hóa, migrate, upgrade |
| Framework HTTP, format API | Cao | Fashion công nghệ, client mới |

Mũi tên phải trỏ từ hàng dưới lên hàng trên. DIP là cơ chế hiện thực hóa điều đó.

### 3.3 Abstraction thuộc về ai? — điểm bị hiểu sai nhiều nhất

DIP nói "cả hai phụ thuộc abstraction". Nhưng abstraction đặt ở **package nào**? Câu trả lời quyết định thành bại:

**Interface thuộc về phía cấp cao (consumer) — nó là *nhu cầu* của chính sách, không phải *mô tả* của cơ chế.**

`order.OrderStore` nghĩa là: *"nghiệp vụ order cần một chỗ lưu đơn"* — phát biểu từ góc nhìn nghiệp vụ, chỉ chứa những gì nghiệp vụ cần, dùng kiểu dữ liệu của nghiệp vụ. Nếu đặt interface ở package `postgres`, nó trở thành mô tả của Postgres repo, business phải import `postgres` để lấy nó → mũi tên trỏ sai, DIP thất bại dù "có interface". Có interface không đồng nghĩa với có inversion. **Vị trí của interface quyết định hướng của mũi tên.**

Go hỗ trợ điều này tốt hơn mọi ngôn ngữ mainstream: nhờ **implicit interface satisfaction**, package `postgres` thỏa mãn `order.OrderStore` mà không cần khai báo `implements`, thậm chí không cần import `order` nếu kiểu trùng — nhưng thực tế nó import `order` để dùng kiểu domain (`order.Order`), và đó chính xác là hướng mũi tên ta muốn.

### 3.4 DIP bảo vệ điều gì, giảm coupling thế nào

- **Bảo vệ**: tính thuần khiết (purity) của tầng chính sách — không kiến thức hạ tầng, không side-effect ngoài ý muốn, test được bằng bộ nhớ.
- **Giảm coupling**: từ coupling với *implementation cụ thể* (hàng nghìn dòng, thay đổi thường xuyên, kéo theo transitive dependencies) xuống coupling với *contract vài dòng* do chính mình sở hữu. Kiến thức mà tầng nghiệp vụ phải mang giảm từ "toàn bộ Postgres" xuống "một interface 2 method".

---

## 4. Cách hoạt động: xây dựng từng bước

Bài toán xuyên suốt: *đặt hàng — kiểm tra tồn kho, tạo đơn, gửi email xác nhận.*

### Bước 0 — Phiên bản chưa đảo (baseline)

```
placeorder-v0/
└── internal/
    ├── order/service.go        # import postgres, smtpmail  ✗
    ├── postgres/repo.go
    └── smtpmail/sender.go
```

```go
// internal/order/service.go — V0: nghiệp vụ ôm hạ tầng
package order

import (
	"context"
	"myapp/internal/postgres"  // ✗
	"myapp/internal/smtpmail"  // ✗
)

type Service struct {
	repo *postgres.OrderRepo   // kiểu cụ thể
	mail *smtpmail.Sender      // kiểu cụ thể
}

func (s *Service) PlaceOrder(ctx context.Context, customerID string, items []postgres.ItemRow) error {
	// ✗✗ nghiệp vụ dùng kiểu của tầng DB (ItemRow) — leaky đến tận chữ ký hàm
	...
}
```

Đếm kiến thức: `order` biết Postgres (schema, kiểu row), biết SMTP (địa chỉ server nằm trong Sender). Test `PlaceOrder` cần Postgres thật + SMTP thật hoặc build tag khéo léo. Không đạt.

### Bước 1 — Nghiệp vụ tuyên bố nhu cầu bằng interface của chính nó

```go
// internal/order/order.go — kiểu domain, thuộc về nghiệp vụ
package order

import (
	"context"
	"errors"
	"time"
)

type Order struct {
	ID         string
	CustomerID string
	Items      []Item
	Total      Money
	PlacedAt   time.Time
}

type Item struct {
	SKU      string
	Quantity int
	Price    Money
}

type Money int64

var (
	ErrEmptyOrder       = errors.New("order: empty order")
	ErrInsufficientStock = errors.New("order: insufficient stock")
)

// ==== NHU CẦU của nghiệp vụ — interface do nghiệp vụ sở hữu ====

// Repository: "tôi cần chỗ lưu đơn hàng"
type Repository interface {
	Save(ctx context.Context, o Order) error
}

// StockChecker: "tôi cần biết hàng còn không"
type StockChecker interface {
	Available(ctx context.Context, sku string) (int, error)
}

// Notifier: "tôi cần báo cho khách" — chú ý: KHÔNG phải EmailSender.
// Nghiệp vụ không quy định kênh; email chỉ là một cơ chế.
type Notifier interface {
	OrderPlaced(ctx context.Context, o Order) error
}
```

Ba interface này hẹp (1 method), dùng thuần kiểu domain, đặt tên theo **vai trò trong nghiệp vụ** chứ không theo công nghệ. Đây là những "cổng" (port) mà thế giới bên ngoài sẽ cắm vào.

### Bước 2 — Use case chỉ phụ thuộc các cổng

```go
// internal/order/service.go — V1: thuần chính sách
package order

import (
	"context"
	"fmt"
	"time"
)

type Service struct {
	repo    Repository
	stock   StockChecker
	notify  Notifier
	now     func() time.Time // cả thời gian cũng là dependency — test được
	newID   func() string
}

func NewService(repo Repository, stock StockChecker, notify Notifier,
	now func() time.Time, newID func() string) *Service {
	return &Service{repo: repo, stock: stock, notify: notify, now: now, newID: newID}
}

func (s *Service) PlaceOrder(ctx context.Context, customerID string, items []Item) (Order, error) {
	if len(items) == 0 {
		return Order{}, ErrEmptyOrder
	}
	for _, it := range items {
		avail, err := s.stock.Available(ctx, it.SKU)
		if err != nil {
			return Order{}, fmt.Errorf("check stock %s: %w", it.SKU, err)
		}
		if avail < it.Quantity {
			return Order{}, fmt.Errorf("%w: sku %s", ErrInsufficientStock, it.SKU)
		}
	}

	o := Order{
		ID:         s.newID(),
		CustomerID: customerID,
		Items:      items,
		Total:      total(items),
		PlacedAt:   s.now(),
	}
	if err := s.repo.Save(ctx, o); err != nil {
		return Order{}, fmt.Errorf("save order: %w", err)
	}
	// Quyết định thiết kế: gửi thông báo lỗi KHÔNG làm fail đơn —
	// đây là business rule, và giờ nó nằm đúng chỗ để đọc thấy được.
	if err := s.notify.OrderPlaced(ctx, o); err != nil {
		// log ở đây qua một logger được inject; đơn vẫn thành công
		_ = err
	}
	return o, nil
}

func total(items []Item) Money {
	var t Money
	for _, it := range items {
		t += it.Price * Money(it.Quantity)
	}
	return t
}
```

Kiểm tra: `go list -deps ./internal/order` → chỉ stdlib. Nghiệp vụ đạt độ thuần 100%.

### Bước 3 — Hạ tầng implement các cổng (mũi tên đã đảo)

```go
// internal/postgres/order_repo.go
package postgres

import (
	"context"
	"database/sql"
	"encoding/json"
	"fmt"

	"myapp/internal/order" // ← MŨI TÊN: hạ tầng import nghiệp vụ
)

type OrderRepo struct{ db *sql.DB }

func NewOrderRepo(db *sql.DB) *OrderRepo { return &OrderRepo{db} }

func (r *OrderRepo) Save(ctx context.Context, o order.Order) error {
	items, err := json.Marshal(o.Items)
	if err != nil {
		return fmt.Errorf("marshal items: %w", err)
	}
	_, err = r.db.ExecContext(ctx,
		`INSERT INTO orders (id, customer_id, items, total, placed_at)
		 VALUES ($1, $2, $3, $4, $5)`,
		o.ID, o.CustomerID, items, int64(o.Total), o.PlacedAt)
	if err != nil {
		return fmt.Errorf("insert order: %w", err)
	}
	return nil
}
```

```go
// internal/smtpmail/notifier.go
package smtpmail

import (
	"context"
	"fmt"
	"net/smtp"

	"myapp/internal/order"
)

type Notifier struct {
	addr, from string
	auth       smtp.Auth
	// map customerID → email; thực tế sẽ inject một CustomerDirectory
	emailOf func(ctx context.Context, customerID string) (string, error)
}

func (n *Notifier) OrderPlaced(ctx context.Context, o order.Order) error {
	to, err := n.emailOf(ctx, o.CustomerID)
	if err != nil {
		return fmt.Errorf("resolve email: %w", err)
	}
	body := fmt.Sprintf("Subject: Don hang %s\r\n\r\nTong tien: %d VND\r\n", o.ID, o.Total)
	return smtp.SendMail(n.addr, n.auth, n.from, []string{to}, []byte(body))
}
```

### Bước 4 — Composition root nối dây

```go
// cmd/api/main.go
package main

import (
	"crypto/rand"
	"database/sql"
	"encoding/hex"
	"log"
	"time"

	_ "github.com/lib/pq"

	"myapp/internal/httpapi"
	"myapp/internal/order"
	"myapp/internal/postgres"
	"myapp/internal/smtpmail"
)

func main() {
	db, err := sql.Open("postgres", mustEnv("DATABASE_URL"))
	if err != nil { log.Fatal(err) }

	svc := order.NewService(
		postgres.NewOrderRepo(db),
		postgres.NewStockRepo(db),
		smtpmail.New(mustEnv("SMTP_ADDR"), mustEnv("SMTP_FROM")),
		time.Now,
		newID,
	)
	log.Fatal(httpapi.New(svc).ListenAndServe(":8080"))
}

func newID() string {
	b := make([]byte, 16)
	rand.Read(b)
	return hex.EncodeToString(b)
}
```

### Bước 5 — Unit test nghiệp vụ bằng fake trong bộ nhớ

```go
// internal/order/service_test.go
package order_test

import (
	"context"
	"errors"
	"testing"
	"time"

	"myapp/internal/order"
)

// Fake — implementation thật nhưng chạy trong RAM. Không cần mockgen.
type fakeRepo struct{ saved []order.Order; failWith error }

func (f *fakeRepo) Save(_ context.Context, o order.Order) error {
	if f.failWith != nil { return f.failWith }
	f.saved = append(f.saved, o)
	return nil
}

type fakeStock map[string]int

func (f fakeStock) Available(_ context.Context, sku string) (int, error) { return f[sku], nil }

type fakeNotifier struct{ notified int; failWith error }

func (f *fakeNotifier) OrderPlaced(context.Context, order.Order) error {
	f.notified++
	return f.failWith
}

func newSvc(repo *fakeRepo, stock fakeStock, n *fakeNotifier) *order.Service {
	return order.NewService(repo, stock, n,
		func() time.Time { return time.Date(2026, 1, 1, 0, 0, 0, 0, time.UTC) },
		func() string { return "test-id-1" },
	)
}

func TestPlaceOrder_ThanhCong(t *testing.T) {
	repo, notif := &fakeRepo{}, &fakeNotifier{}
	svc := newSvc(repo, fakeStock{"SKU1": 10}, notif)

	o, err := svc.PlaceOrder(context.Background(), "cust-1",
		[]order.Item{{SKU: "SKU1", Quantity: 2, Price: 50_000}})

	if err != nil { t.Fatal(err) }
	if o.Total != 100_000 { t.Fatalf("total = %d", o.Total) }
	if len(repo.saved) != 1 { t.Fatal("order chua duoc luu") }
	if notif.notified != 1 { t.Fatal("khach chua duoc bao") }
}

func TestPlaceOrder_HetHang(t *testing.T) {
	svc := newSvc(&fakeRepo{}, fakeStock{"SKU1": 1}, &fakeNotifier{})
	_, err := svc.PlaceOrder(context.Background(), "cust-1",
		[]order.Item{{SKU: "SKU1", Quantity: 2, Price: 50_000}})
	if !errors.Is(err, order.ErrInsufficientStock) {
		t.Fatalf("want ErrInsufficientStock, got %v", err)
	}
}

func TestPlaceOrder_LoiGuiMail_DonVanThanhCong(t *testing.T) {
	repo := &fakeRepo{}
	svc := newSvc(repo, fakeStock{"SKU1": 10}, &fakeNotifier{failWith: errors.New("smtp down")})
	_, err := svc.PlaceOrder(context.Background(), "cust-1",
		[]order.Item{{SKU: "SKU1", Quantity: 1, Price: 50_000}})
	if err != nil { t.Fatalf("don phai thanh cong du mail loi, got %v", err) }
	if len(repo.saved) != 1 { t.Fatal("order chua duoc luu") }
}
```

Bộ test này chạy trong ~1ms, không network, không container, và **test đúng business rule** kể cả rule tinh tế "mail lỗi không fail đơn". Ở V0, rule đó không thể test mà không dựng SMTP giả ở tầng network.

### Sơ đồ tổng kết

```
                        ┌──────────────────────┐
                        │      cmd/api/main     │  (biết tất cả — chỉ để lắp ráp)
                        └──┬──────┬──────┬──────┘
                           │      │      │ import
              ┌────────────▼─┐  ┌─▼──────────┐  ┌─▼──────────┐
              │   httpapi     │  │  postgres  │  │  smtpmail  │
              └────────────┬─┘  └─┬──────────┘  └─┬──────────┘
                    import │      │ import         │ import
                           ▼      ▼                ▼
                        ┌──────────────────────────┐
                        │       order (domain)      │   ← MỌI mũi tên trỏ VÀO
                        │  Service + Repository +   │      import: chỉ stdlib
                        │  StockChecker + Notifier  │
                        └──────────────────────────┘
```

Đây, thu nhỏ, **chính là Clean Architecture**. Phần còn lại của tài liệu chỉ là mở rộng sơ đồ này ra nhiều tầng, nhiều module, nhiều pattern.

---

## 5. Trade-off

| Được | Mất |
|---|---|
| Nghiệp vụ test bằng RAM, mili-giây | Thêm ~30% số file (interface, adapter, wiring) |
| Đổi hạ tầng = viết adapter mới | Đọc code phải nhảy qua interface (IDE "go to implementation") |
| Nghiệp vụ tái dùng cho HTTP/gRPC/CLI/worker | Người mới cần hiểu quy ước trước khi đóng góp |
| Thay đổi hạ tầng không lan vào nghiệp vụ | Interface là contract phải bảo trì: thêm method = sửa mọi impl + fake |
| Compile-time enforcement bằng `internal/` + linter | Performance: gọi qua interface ngăn inline, thêm ~1–3ns/call |

Về performance: 1–3ns mỗi call qua interface so với 1–10ms mỗi query DB — nhiễu nền, **không bao giờ** là lý do từ chối DIP ở tầng use case. Nó chỉ đáng cân nhắc trong hot loop xử lý hàng triệu phần tử (lúc đó dùng kiểu cụ thể hoặc generics trong phạm vi hẹp).

Trade-off thật sự nằm ở **chi phí nhận thức**: mỗi interface là một khái niệm thêm vào đầu người đọc. Vì thế: đảo phụ thuộc **tại ranh giới kiến trúc** (nghiệp vụ ↔ hạ tầng), không đảo giữa hai package nghiệp vụ cùng tầng, càng không đảo bên trong một package.

---

## 6. Best Practices

1. **Interface ở phía consumer, đặt tên theo vai trò**: `order.Notifier`, không phải `smtp.IEmailService`.
2. **Interface hẹp** — bắt đầu 1 method; nhu cầu mới thì thêm interface mới hoặc compose.
3. **Chỉ dùng kiểu domain trong contract** — thấy `*sql.Row`, `*http.Request`, `pb.OrderProto` trong interface của package nghiệp vụ là báo động đỏ.
4. **Dịch lỗi tại adapter**: `sql.ErrNoRows` → `order.ErrNotFound`. Sentinel error của domain là một phần của contract.
5. **Inject cả non-determinism**: `time.Now`, random ID, UUID — như ví dụ trên — để test deterministic.
6. **Kiểm tra compile-time** implement đủ interface tại adapter:

```go
var _ order.Repository = (*OrderRepo)(nil) // vỡ build ngay nếu thiếu method
```

7. **Cưỡng chế hướng import bằng CI** (depguard/gomodguard) — kiến trúc chỉ tồn tại nếu được enforce.

---

## 7. Anti-patterns

- **Interface đặt cạnh implementation** (`postgres.Repository` mà business import) — inversion giả. Đã phân tích ở 3.3.
- **Interface "ăn theo" implementation**: 12 method vì repo có 12 method. Vi phạm ISP, mock nặng nề. → interface theo nhu cầu use case.
- **Đảo mọi thứ**: interface cho cả hàm tính tổng, struct config… Chi phí nhận thức tăng, giá trị bằng 0. → chỉ đảo tại ranh giới nghiệp vụ/hạ tầng.
- **Service Locator** (`container.Get("orderRepo")`): giấu dependency vào chuỗi string, mất kiểm tra compile-time, dependency graph trở nên vô hình. → constructor injection tường minh.
- **Adapter gọi ngược use case**: `postgres.OrderRepo` gọi `order.Service.Validate()` bên trong `Save` → vòng lặp logic, transaction lồng nhau khó lường. Adapter chỉ làm cơ chế, không làm chính sách.

---

## 8. Khi nào KHÔNG đảo phụ thuộc

- Giữa hai package **cùng tầng** nghiệp vụ ổn định: `order` gọi thẳng `pricing.Calculate()` — hàm thuần, cùng vòng đời, không cần interface ở giữa.
- Với **stdlib ổn định**: không ai cần `ITimeProvider` bọc `time.Now` *trừ khi* test cần điều khiển thời gian (như ví dụ trên — và khi đó inject `func() time.Time` là đủ, không cần interface).
- **Tool nhỏ, script**: toàn bộ chương trình là "tầng ngoài", không có chính sách đáng bảo vệ.
- Khi chỉ có một implementation **và** không cần fake để test **và** không nằm trên ranh giới kiến trúc — cả ba điều kiện cùng lúc.

---

## Tóm tắt chương

- DIP tách control flow (không đổi) khỏi source dependency (bị đảo): hạ tầng import nghiệp vụ.
- Vị trí interface quyết định tất cả: interface là *nhu cầu của consumer*, sống trong package consumer, nói ngôn ngữ domain.
- Go hỗ trợ DIP hạng nhất: implicit satisfaction + consumer-side interface + `internal/`.
- Sơ đồ "mọi mũi tên trỏ vào domain" ở mục 4 chính là Clean Architecture thu nhỏ.

**Chương tiếp theo:** [Separation of Concerns & Composition](/series/clean-architect/01-foundation/04-separation-of-concerns/) — mảnh nền cuối trước khi vào Clean Architecture Core.
