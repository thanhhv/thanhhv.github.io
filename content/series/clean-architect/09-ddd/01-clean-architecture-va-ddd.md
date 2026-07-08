+++
title = "Chương 9 — Clean Architecture & DDD: Lấp đầy vòng trong"
date = "2026-07-08T08:00:00+07:00"
draft = false
tags = ["backend", "golang", "clean-architecture"]
series = ["Clean Architecture với Golang"]
+++

> **Level 3 – Senior** · Clean Architecture cho bạn cái vỏ (ranh giới, hướng phụ thuộc); DDD cho bạn cái ruột (mô hình hóa nghiệp vụ bên trong vòng domain). Thiếu một trong hai: vỏ đẹp ruột rỗng, hoặc mô hình hay bị hạ tầng ăn mòn.

---

## 1. Problem Statement

Clean Architecture nói "đặt business rule vào vòng trong" nhưng **không nói business rule trông như thế nào**. Hệ quả phổ biến: team dựng đủ 4 tầng, rồi vòng domain chỉ chứa struct trần (anemic model), toàn bộ logic dồn vào "service" — tức Transaction Script khoác áo Clean Architecture, trả phí kiến trúc mà không nhận mô hình.

DDD trả lời đúng câu hỏi còn thiếu: mô hình hóa nghiệp vụ phức tạp thành code như thế nào. Nhưng DDD đầy đủ (Evans, 2003) là bộ công cụ nặng; áp nguyên xi vào mọi dự án là over-engineering kinh điển. Chương này: các khối DDD chiến thuật đáng dùng trong Go, cách chúng khớp vào các vòng, và **liều lượng**.

## 2. Ánh xạ DDD ↔ Clean Architecture

| Khối DDD | Vòng Clean Architecture | Ghi chú |
|---|---|---|
| Entity, Value Object, Aggregate | Vòng 1 — Entities | Chính là "Enterprise Business Rules" |
| Domain Service | Vòng 1 | Rule không thuộc riêng entity nào |
| Domain Event | Vòng 1 (định nghĩa), vòng 2 (phát) | |
| Application Service | Vòng 2 — Use Cases | Hai tên cho gần cùng một thứ |
| Repository (interface) | Vòng 2 sở hữu | Implementation ở vòng 3 |
| Bounded Context | Ranh giới **module/service** | Lớn hơn mọi vòng — mỗi context có bộ vòng riêng |
| Ubiquitous Language | Xuyên suốt | Tên package, struct, method = từ vựng nghiệp vụ |

Insight quan trọng nhất: **Bounded Context quyết định trước, các vòng quyết định sau.** Chia sai context (gộp "sản phẩm" của team catalog với "sản phẩm" của team kho vào một model) thì Clean Architecture bên trong không cứu được — model phình thành god-model phục vụ mọi ngữ nghĩa. Mỗi context một model riêng, trùng tên nhưng khác nghĩa là *bình thường và đúng*: `catalog.Product` có mô tả/giá/ảnh, `inventory.Product` có tồn kho/vị trí kệ — hai struct, hai package, không chung "base".

## 3. Value Object — vũ khí bị bỏ quên nhiều nhất

Value Object: bất biến, so sánh bằng giá trị, **tự bảo vệ tính hợp lệ**. Đây là khối lợi ích/chi phí tốt nhất của DDD — dùng được cả trong dự án không "làm DDD":

```go
// domain/money.go — kiểu tiền tệ không thể sai
package domain

import (
	"errors"
	"fmt"
)

type Currency string

const (
	VND Currency = "VND"
	USD Currency = "USD"
)

// Money là value object: unexported field, tạo qua constructor, immutable.
type Money struct {
	amount   int64 // đơn vị nhỏ nhất (đồng, cent)
	currency Currency
}

var ErrCurrencyMismatch = errors.New("money: currency mismatch")

func NewMoney(amount int64, c Currency) (Money, error) {
	if amount < 0 {
		return Money{}, fmt.Errorf("money: negative amount %d", amount)
	}
	return Money{amount: amount, currency: c}, nil
}

func (m Money) Add(other Money) (Money, error) {
	if m.currency != other.currency {
		return Money{}, fmt.Errorf("%w: %s + %s", ErrCurrencyMismatch, m.currency, other.currency)
	}
	return Money{amount: m.amount + other.amount, currency: m.currency}, nil
}

func (m Money) MultiplyPercent(p int64) Money {
	return Money{amount: m.amount * p / 100, currency: m.currency}
}
```

So với `totalVND int64` rải khắp code: cộng nhầm USD với VND giờ là **lỗi compile/lỗi tường minh** thay vì bug tiền bạc âm thầm. Các ứng viên VO tự nhiên: `Email`, `PhoneNumber`, `OrderID` (typed ID chống truyền nhầm `userID` vào chỗ `orderID`!), `Quantity`, `DateRange`, `Address`. Chi phí: constructor + method thay vì thao tác primitive — rẻ. Đây là thuốc giải cho **Primitive Obsession**.

```go
// Typed ID — một dòng, diệt cả họ bug
type OrderID string
type CustomerID string
// func Cancel(id OrderID) — compiler chặn Cancel(customerID) ngay
```

## 4. Aggregate — đơn vị nhất quán

Aggregate = cụm entity/VO thay đổi **nhất quán cùng nhau**, có một **root** làm cổng vào duy nhất. Ví dụ `Order` (root) + `OrderLine`s + rule "tổng tiền = tổng line, đơn confirmed không sửa line":

```go
// domain/order.go
type Order struct {
	id        OrderID
	status    Status
	lines     []OrderLine // KHÔNG expose slice ra ngoài
	total     Money
}

var ErrOrderLocked = errors.New("order: confirmed order cannot change")

// Mọi thay đổi đi qua root — root giữ bất biến
func (o *Order) AddLine(sku SKU, qty Quantity, unitPrice Money) error {
	if o.status != StatusDraft {
		return ErrOrderLocked
	}
	line := OrderLine{sku: sku, qty: qty, unitPrice: unitPrice}
	o.lines = append(o.lines, line)
	o.recalcTotal() // bất biến "total = sum(lines)" KHÔNG THỂ bị bỏ quên
	return nil
}

func (o *Order) Lines() []OrderLine { // trả bản copy — bảo vệ nội tạng
	out := make([]OrderLine, len(o.lines))
	copy(out, o.lines)
	return out
}
```

Ba quy tắc aggregate quan trọng trong thiết kế hệ thống:

1. **Ranh giới transaction = ranh giới aggregate** (chương 5): một tx ghi một aggregate. Cần đổi 2 aggregate → 2 tx + domain event + eventual consistency.
2. **Aggregate tham chiếu aggregate khác bằng ID**, không bằng con trỏ: `Order` giữ `CustomerID`, không giữ `*Customer` — nếu không, "lưu Order" kéo theo cả object graph, repository thành ORM và ranh giới nhất quán vỡ.
3. **Giữ aggregate NHỎ.** Aggregate to = lock rộng, contention cao, load nặng. "Order chứa mọi thứ liên quan đơn hàng" là cách hiểu sai; chỉ những gì phải nhất quán *ngay lập tức* mới vào cùng aggregate.

## 5. Domain Service vs Application Service — ai làm gì

```go
// DOMAIN service: rule nghiệp vụ liên quan NHIỀU aggregate, vẫn thuần túy
// "giá cuối = giá đơn - khuyến mãi tốt nhất khách đủ điều kiện"
func BestPrice(o Order, c Customer, promos []Promotion) Money {
	best := o.Total()
	for _, p := range promos {
		if p.EligibleFor(c) {
			if candidate := p.Apply(o.Total()); candidate.LessThan(best) {
				best = candidate
			}
		}
	}
	return best
}
```

Nhận diện: domain service **thuần** (không I/O, không cổng) — chỉ là hàm nhận domain object trả domain object. Application service (use case) thì **điều phối**: load qua repo, gọi entity/domain service, save, phát event. Kim chỉ nam khi phân vân code đặt đâu:

- Nói về *cái gì đúng/sai trong nghiệp vụ* → domain (entity nếu thuộc một aggregate, domain service nếu nhiều).
- Nói về *thứ tự làm việc và I/O* → use case.
- Nói về *format và giao thức* → adapter.

## 6. Domain Event trong module — decoupling nội bộ

Sự kiện nghiệp vụ ("OrderPlaced") cho phép module khác phản ứng mà module gốc không biết họ:

```go
// domain/events.go
type OrderPlaced struct {
	OrderID    OrderID
	CustomerID CustomerID
	Total      Money
	At         time.Time
}

// Entity GHI NHẬN event thay vì tự phát (entity không có I/O)
func (o *Order) Place(now time.Time) ([]DomainEvent, error) {
	if len(o.lines) == 0 { return nil, ErrEmptyOrder }
	o.status = StatusPlaced
	return []DomainEvent{OrderPlaced{o.id, o.customerID, o.total, now}}, nil
}

// Use case thu event từ entity và giao cho publisher (outbox — chương 7)
events, err := ord.Place(uc.now())
if err != nil { return err }
if err := uc.orders.Save(ctx, ord); err != nil { return err }
return uc.publisher.Publish(ctx, events...)
```

Pattern "entity trả event, use case phát" giữ entity thuần và điểm phát tập trung. Trong monolith, publisher có thể là in-process bus gọi handler của module khác — chuẩn bị sẵn đường tách service sau này (chương 12).

## 7. Liều lượng — dùng bao nhiêu DDD là đủ

DDD chiến thuật có thang liều:

| Mức | Dùng gì | Khi nào |
|---|---|---|
| 0 | Struct + hàm, transaction script | CRUD, ít rule |
| 1 | **Value Object + typed ID** | Hầu như luôn đáng — chi phí ~0 |
| 2 | Entity giữ bất biến + sentinel error domain | Có rule thật quanh trạng thái |
| 3 | Aggregate rõ ràng + repository theo aggregate | Nhiều object nhất quán cùng nhau, concurrent write |
| 4 | Domain event + domain service | Nhiều module phản ứng lẫn nhau, rule đa aggregate |
| 5 | Bounded context tách bạch + context mapping | Nhiều team, domain lớn, ngữ nghĩa xung đột |

Sai lầm phổ biến: nhảy thẳng mức 5 cho startup 2 người ("mình làm DDD mà"), hoặc kẹt mức 0 cho hệ thống tài chính 50 rule ("DDD phức tạp lắm"). Liều đúng đo bằng **độ phức tạp của rule nghiệp vụ**, không bằng độ ngầu của kiến trúc. Và các mức tăng dần được: bắt đầu mức 1–2, nâng khi rule dày lên.

## 8. Anti-patterns

- **Anemic Domain Model có nghi thức**: entity chỉ getter/setter, mọi rule trong service — mất encapsulation (rule quanh một dữ liệu rải ở N service), test khó hơn, chính xác thứ SRP/aggregate sinh ra để tránh. Nếu domain *thật sự* không có rule → thừa nhận và bỏ nghi thức (mức 0), đừng giữ vỏ DDD rỗng.
- **Thin Domain giả tạo**: entity có method nhưng method chỉ `SetStatus(s)` — gán không kiểm tra gì. Bất biến vẫn không được bảo vệ; tệ hơn anemic vì *trông như* được bảo vệ.
- **God Aggregate**: `Customer` chứa orders, wallet, tickets, addresses — mọi tx đụng Customer, contention và merge conflict cực đại. Tách theo ranh giới nhất quán thật.
- **DDD-cargo-cult vocabulary**: đổi tên `Service` thành `ApplicationService`, folder `valueobjects/` — từ vựng thay đổi, thiết kế y nguyên. DDD là mô hình hóa, không phải danh pháp.
- **Bounded context chia theo bảng DB** thay vì theo ngôn ngữ nghiệp vụ — context là ranh giới *ngữ nghĩa*; hai context có thể cùng lưu một DB (tách schema) trong monolith.

## Tóm tắt

- DDD lấp ruột cho vòng trong: VO/Entity/Aggregate = vòng 1, use case = application service, bounded context = ranh giới module.
- Đáng dùng gần như luôn: value object, typed ID, entity giữ bất biến. Đáng dùng khi phức tạp thật: aggregate, domain event, domain service. Bounded context: quyết định quan trọng nhất và phải quyết trước kiến trúc bên trong.
- Liều lượng theo độ dày của rule, không theo trend.

**Chương tiếp theo:** [Clean Architecture & CQRS](/series/clean-architect/10-cqrs/01-clean-architecture-va-cqrs/)
