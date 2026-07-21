+++
title = "1.5 — Interface & Immutability: hai vũ khí thiết kế của Go"
date = "2026-07-17T07:50:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## Phần A — Interface

### 1. Problem Statement

Interface đã xuất hiện ở mọi chương trước như "công cụ giảm coupling". Chương này trả lời các câu hỏi mà kỹ sư senior thực sự vướng: interface đặt ở **đâu**? To hay nhỏ? Khai báo **khi nào**? Vì sao interface Go khác Java/TS ở tầng triết lý — và điều đó đổi cách thiết kế ra sao?

### 2. Điều làm interface Go khác biệt: implicit satisfaction

```go
package storage

type Saver interface {
    Save(ctx context.Context, key string, data []byte) error
}
```

Bất kỳ type nào có method `Save` đúng chữ ký — tự động thỏa mãn `Saver`. Không `implements`, không import package chứa interface. Hệ quả thiết kế sâu sắc:

**(a) Interface có thể khai báo SAU khi implementation tồn tại.** Trong Java/C# truyền thống (nominal typing), muốn `S3Client` thỏa mãn interface của bạn thì phải sửa `S3Client` thêm `implements` — không thể nếu nó nằm trong thư viện của người khác. Trong Go, bạn định nghĩa interface *khớp với method có sẵn* của type bên thứ ba — nó thỏa mãn ngay, thư viện không cần biết bạn tồn tại. **Adapter pattern teo nhỏ lại đáng kể trong Go** vì phần lớn nhu cầu adapt biến mất.

**(b) Đảo ngược nơi đặt interface.** Vì không cần `implements`, interface không cần sống cạnh implementation. Quy ước Go chuẩn:

> **Interface được định nghĩa ở package TIÊU DÙNG (consumer), theo đúng nhu cầu của consumer — không phải ở package cung cấp (producer).**

```
❌ Kiểu Java cổ điển (producer-side):        ✅ Kiểu Go (consumer-side):

package storage                               package report  (người DÙNG)
  type Storage interface { 28 methods }         type StatsSource interface {
  type S3Storage struct{...}                        OrdersInRange(...) ([]Row, error)
                                                }   // đúng 1 method nó cần
package report
  import "storage" // dùng interface to đùng    package postgres (người CUNG CẤP)
                                                  // không biết report tồn tại,
                                                  // chỉ tình cờ có method khớp
```

Lợi ích đo được: interface consumer-side chỉ chứa method consumer thực sự dùng → fake trong test chỉ phải implement 1 method thay vì 28; đổi hạ tầng chỉ cần khớp 1 method. Đây là Interface Segregation (chương 2.4) được **ngôn ngữ khuyến khích một cách tự nhiên**.

TypeScript ở giữa: structural typing như Go (duck typing khi compile), nên về lý thuyết làm consumer-side interface được — nhưng văn hóa NestJS/Angular kéo về producer-side + DI token. Kỹ sư TS giỏi có thể "viết kiểu Go" trong TS và hưởng cùng lợi ích.

**(c) Interface nhỏ là chuẩn mực.** Go proverb: *"The bigger the interface, the weaker the abstraction."* Thống kê stdlib: phần lớn interface có 1–2 method (`io.Reader`, `fmt.Stringer`, `sort.Interface`, `http.Handler`). Interface 1 method tổ hợp được (`io.ReadWriteCloser` = 3 interface ghép), thỏa mãn dễ, fake dễ.

### 3. Refactoring Journey — "chờ interface xuất hiện"

Sai lầm phổ biến nhất của dev Java/C# chuyển sang Go — tạo interface ngày đầu tiên:

```go
// ❌ V1 kiểu "Java in Go": interface + đúng 1 implementation, cùng package
type UserService interface { /* 12 methods */ }
type userServiceImpl struct{ ... }
func NewUserService(...) UserService { return &userServiceImpl{...} }
```

Vấn đề: 12 method không ai cần đủ; mỗi lần thêm method sửa 2 chỗ + mọi mock; "flexibility" không ai dùng; và trả interface (thay vì concrete type) làm mất khả năng thêm method tiện ích, mất godoc rõ ràng. Quy ước Go: **"Accept interfaces, return structs"** — nhận interface (rộng lượng với input), trả concrete type (rõ ràng với output).

```go
// ✅ V2 — trình tự đúng trong Go:
// 1. Viết concrete type trước:
func NewService(db *sql.DB) *Service { ... }   // trả *Service, không giấu sau interface

// 2. Interface xuất hiện Ở PHÍA CONSUMER, khi consumer cần:
package handler
type OrderGetter interface {                    // khi handler cần test cách ly
    GetOrder(ctx context.Context, id string) (*order.Order, error)
}
func NewHandler(og OrderGetter) *Handler { ... } // *order.Service tự khớp
```

Interface trong Go là thứ **được chưng cất ra từ nhu cầu sử dụng thực tế**, không phải thứ thiết kế trước. Điều này đảo ngược thói quen "design by interface" của enterprise Java — và là lý do code Go tốt trông "ít kiến trúc" hơn nhưng dẻo hơn.

### 4. Trade-off & bẫy cần biết

- **Interface có chi phí runtime**: gọi qua interface = dynamic dispatch, mất cơ hội inline; đưa giá trị vào interface có thể gây heap allocation. Với hot path (hàm gọi hàng triệu lần/giây), đo trước khi trừu tượng hóa — `go test -bench` là bạn. Với code nghiệp vụ thường, chi phí này không đáng kể so với một câu SQL.
- **Bẫy typed-nil kinh điển**: interface Go = (type, value); con trỏ nil *có type* đặt vào interface → interface **không** nil. `if err != nil` bất ngờ đúng dù "không có lỗi". Quy tắc: hàm trả `error`, khi thành công hãy `return nil` tường minh, đừng trả biến con trỏ lỗi có thể nil.
- **Structural typing khớp "nhầm"**: hai interface 1-method cùng chữ ký nhưng ngữ nghĩa khác (Close của file vs Close của phiên giao dịch) — type thỏa mãn cả hai về cú pháp. Go đánh cược rằng trường hợp này hiếm; thực tế xác nhận, nhưng nên biết nó tồn tại.

---

## Phần B — Immutability

### 1. Problem Statement

Bug sản xuất có thật, gặp ở hầu hết codebase Go/Node đủ tuổi:

```go
// config được load một lần, truyền khắp nơi
cfg := LoadConfig()
go serviceA.Run(cfg)
go serviceB.Run(cfg)

// đâu đó sâu trong serviceA, một dev "tiện tay":
func (a *ServiceA) Run(cfg *Config) {
    cfg.Timeout = 5 * time.Second  // ❌ chỉ định sửa cho A, nhưng B cũng ăn đạn
}
```

Hoặc bản slice — bẫy Go đặc trưng:

```go
func TopThree(orders []Order) []Order {
    sort.Slice(orders, func(i, j int) bool {   // ❌ sort SỬA slice của caller!
        return orders[i].Total > orders[j].Total
    })
    return orders[:3]
}
// Caller: "tại sao danh sách đơn hàng của tôi tự nhiên đổi thứ tự?!"
```

Bản chất vấn đề bằng ngôn ngữ chương 1.3: **shared mutable state là coupling tàng hình mạnh nhất** — hai đoạn code không import nhau, không gọi nhau, vẫn phá nhau qua một vùng nhớ chung. Thêm goroutine vào là nâng cấp từ "bug logic khó lần" thành **data race** (undefined behavior).

### 2. First Principles

Chi phí của mutable state = **số nơi có thể ghi × số nơi đọc**. Mỗi tổ hợp là một tương tác tiềm năng phải suy xét khi debug. Immutability đưa số nơi ghi về 1 (lúc khởi tạo) → chi phí suy luận sụp về tuyến tính. Đó là lý do functional programming coi immutability là mặc định.

Go **không** có immutability ở mức ngôn ngữ (không `readonly`, `const` chỉ cho hằng số scalar; TS có `readonly`/`Readonly<T>` nhưng cũng chỉ compile-time). Vậy trong Go, immutability là **kỷ luật thiết kế**, thực thi bằng quy ước + API:

```go
// Chiến thuật 1 — value semantics: truyền/trả bằng GIÁ TRỊ khi struct nhỏ
type Money struct {
    amount   int64
    currency string
}
// Không có setter. "Sửa" = tạo mới:
func (m Money) Add(other Money) (Money, error) {
    if m.currency != other.currency {
        return Money{}, fmt.Errorf("currency mismatch: %s vs %s", m.currency, other.currency)
    }
    return Money{amount: m.amount + other.amount, currency: m.currency}, nil
}
// time.Time trong stdlib chính là mẫu này: Add trả Time mới, không sửa tại chỗ
```

```go
// Chiến thuật 2 — defensive copy tại biên giới
func TopThree(orders []Order) []Order {
    cp := slices.Clone(orders)   // copy trước, sort trên bản copy
    slices.SortFunc(cp, func(a, b Order) int { return int(b.Total - a.Total) })
    return cp[:3:3]              // giới hạn cap: append của caller không đè vùng nhớ gốc
}
```

```go
// Chiến thuật 3 — unexported field + không có setter = bất biến với bên ngoài package
type Config struct {
    timeout time.Duration
}
func (c Config) Timeout() time.Duration { return c.timeout }
```

### 3. Trade-off — vì sao Go không ép immutability

- **Chi phí copy là thật**: struct lớn/slice lớn copy mỗi lần "sửa" tốn allocation + GC pressure. Rust giải bằng ownership, Go chọn đơn giản: mutable mặc định, dev tự chọn điểm bất biến. Với hot path xử lý triệu record, mutate tại chỗ là quyết định đúng — **giới hạn trong phạm vi hàm** (mutate biến cục bộ thoải mái; đừng mutate thứ nhận từ ngoài hoặc chia sẻ ra ngoài).
- **Quy tắc thực dụng biên giới (boundary rule)**: bên trong một hàm — mutate tự do, rẻ và rõ. Xuyên qua ranh giới hàm/goroutine/package — bất biến hoặc copy. Bug shared-state hầu như luôn nằm ở *ranh giới*, không nằm trong thân hàm.
- **Value types đáng đầu tư ngay**: `Money`, `TimeRange`, `Email`, `OrderID` — struct nhỏ, so sánh được, không setter. Rẻ, chặn cả lớp bug (cộng nhầm currency, gán nhầm ID), và là viên gạch của DDD Value Object (Level 5).

### 4. Production Examples

- **`time.Time`, `netip.Addr` (Go stdlib)**: immutable value type chuẩn mực — mọi "phép sửa" trả giá trị mới; an toàn concurrent mặc nhiên, không cần lock.
- **`context.Context`**: `context.WithTimeout(parent, d)` không sửa parent — trả context *mới* bọc parent. Cây context bất biến là lý do context an toàn khi hàng nghìn goroutine chia sẻ.
- **`strings` package**: string trong Go bất biến ở mức ngôn ngữ — là lý do string dùng làm map key an toàn và chia sẻ giữa goroutine miễn phí. `strings.Builder` tồn tại chính vì immutability có chi phí (nối chuỗi trong vòng lặp) — minh họa hoàn hảo cặp trade-off.
- **Redux/React (Node.js ecosystem)**: cả kiến trúc đặt cược vào immutable state — so sánh reference rẻ (`===`) để biết state đổi. Đổi lấy: chi phí copy và kỷ luật spread operator; Immer sinh ra để giảm đau đúng chỗ này.
- **Kafka**: log bất biến append-only là *nguyên lý kiến trúc* — consumer đọc lại từ đầu, replay, audit được. Immutability không chỉ là chuyện struct — nó scale lên tới kiến trúc dữ liệu (Event Sourcing, Level 5).

### 5. Anti-pattern & Khi nào KHÔNG cần

- **Getter trả slice/map nội bộ**: `func (o *Order) Items() []Item { return o.items }` — caller sửa được ruột object, encapsulation thủng. Trả `slices.Clone` hoặc iterator.
- **"Immutable" nửa vời**: struct có 9 field readonly và 1 field mutable — mọi bảo đảm sụp; người dùng không thể tin type nữa. Bất biến phải trọn vẹn mới mua được sự an tâm.
- **Copy mù quáng trong hot path**: clone slice triệu phần tử mỗi request để "an toàn" — hãy đo (`go test -bench -benchmem`); đôi khi tài liệu hóa "hàm này mượn slice, không giữ lại sau khi return" là trade-off đúng (chính `io.Writer.Write` quy định caller không được giữ `p` — quy ước thay vì copy).
- **Không cần bận tâm khi**: biến cục bộ trong hàm ngắn; struct chỉ sống trong một goroutine với vòng đời rõ; DTO deserialize xong đọc-only theo quy ước tự nhiên.

---

## Tổng kết Level 1

Năm chương vừa rồi cho bạn bộ khung đánh giá **mọi** quyết định thiết kế:

1. Chi phí thay đổi gồm: hiểu, lan truyền, kiểm chứng, phối hợp. *(1.1)*
2. Abstraction nén chi tiết sau tên gọi đáng tin; encapsulation bảo vệ invariant bằng compiler. *(1.2)*
3. Thước đo vạn năng: **low coupling, high cohesion**, mũi tên phụ thuộc trỏ về phía ổn định. *(1.3)*
4. Composition + interface tách 3 nhu cầu mà inheritance trộn lẫn; function value là đa hình rẻ nhất. *(1.4)*
5. Interface nhỏ, đặt phía consumer, khai báo khi cần; bất biến tại ranh giới, mutate cục bộ. *(1.5)*

Level 2 sẽ cho thấy: **SOLID không phải 5 điều răn mới — mà là 5 hệ quả được đặt tên** của chính những nguyên lý gốc này.

---

*Tiếp theo: [2.0 — SOLID: lịch sử ra đời và vấn đề nó giải quyết](/series/software-design/level-2-principles/00-solid-tong-quan/)*
