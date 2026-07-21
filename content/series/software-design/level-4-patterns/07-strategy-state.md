+++
title = "4.7 — Strategy & State: hai anh em cùng cấu trúc, khác linh hồn"
date = "2026-07-17T11:10:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## 1. Strategy — pattern bạn đã dùng suốt từ đầu tài liệu

Không cần dựng lại code evolution — bạn đã đi trọn con đường của nó ba lần: rule tính phí ship thành `[]FeeRule` (1.1 → 1.4), gateway thanh toán thành interface + registry (2.2), rule hoa hồng thành `[]RateRule` (3.4). Chỉ cần chính danh hóa:

**Intent (GoF)**: đóng gói một *họ thuật toán* thay thế được cho nhau, để thuật toán biến thiên độc lập với client dùng nó. **Cấu trúc GoF**: interface `Strategy` + N concrete class + context giữ tham chiếu strategy.

**Hình dạng Go hiện đại — thang ba bậc, chọn thấp nhất đủ dùng** (đúc kết từ 1.4):

```go
// Bậc 1 — function value: strategy KHÔNG STATE, một thao tác
type FeeRule func(Order) int64
// → 90% nhu cầu strategy trong code nghiệp vụ dừng ở đây

// Bậc 2 — interface 1-2 method: strategy CÓ STATE/CONFIG hoặc cần định danh
type Compressor interface {
    Compress(dst io.Writer, src io.Reader) error
    Ext() string                    // strategy tự mô tả — function value không làm được
}

// Bậc 3 — interface + registry: danh sách strategy MỞ cho bên ngoài (2.2, 4.1)
```

Câu hỏi phân bậc: *strategy có cần state/config riêng không? có cần nhiều method không? ai cung cấp strategy mới?* — leo bậc khi câu trả lời đòi hỏi, không leo trước.

**TS đối chiếu**: y hệt — callback/closure là bậc 1 mặc định (`array.sort(comparator)` chính là Strategy); class chỉ khi có state đáng kể.

## 2. State — cấu trúc của Strategy, bài toán của vòng đời

### Code evolution — nơi State thật sự sinh ra

Bài toán chạy xuyên tài liệu, giờ nhìn vào khía cạnh chưa mổ: **vòng đời đơn hàng**. `pending → confirmed → shipped → delivered`, cộng `cancelled` và `returned`. V1 mà mọi codebase đều có:

```go
// ❌ V1 — mỗi hành động một hàm, mỗi hàm một rừng check trạng thái
func (o *Order) Cancel() error {
    if o.Status == "shipped" || o.Status == "delivered" {
        return errors.New("cannot cancel shipped order")
    }
    if o.Status == "cancelled" {
        return errors.New("already cancelled")
    }
    if o.Status == "confirmed" && o.PaymentCaptured {
        o.refund()                        // hủy sau khi thu tiền → phải hoàn
    }
    o.Status = "cancelled"
    return nil
}

func (o *Order) Ship() error   { /* rừng if khác, trùng một phần */ }
func (o *Order) Return() error { /* rừng if khác nữa */ }
```

Quỹ đạo xuống cấp có mùi riêng: mỗi **trạng thái mới** (thêm `on_hold`) phải rà *mọi* hàm hành động xem có nhánh nào bị ảnh hưởng — tri thức về một trạng thái bị rải theo chiều ngang khắp các hàm (Shotgun Surgery theo trục trạng thái); bảng chuyển trạng thái hợp lệ — tài sản nghiệp vụ quan trọng nhất của module — **không tồn tại ở đâu cả**, nó là tổng gộp ngầm của các câu if.

### Hai lời giải — và lời khuyên trung thực về thứ tự thử

**Lời giải 1 — bảng chuyển trạng thái tường minh.** Chưa cần pattern:

```go
// ✅ V2 — bảng chuyển hợp lệ: tài sản nghiệp vụ thành DATA đọc được, review được
var transitions = map[Status][]Status{
    Pending:   {Confirmed, Cancelled},
    Confirmed: {Shipped, Cancelled, OnHold},
    Shipped:   {Delivered, Returned},
    /* ... */
}

func (o *Order) transitionTo(next Status) error {
    if !slices.Contains(transitions[o.status], next) {
        return fmt.Errorf("invalid transition %s → %s", o.status, next)
    }
    o.status = next
    return nil
}

func (o *Order) Cancel() error {
    if err := o.transitionTo(Cancelled); err != nil {
        return err
    }
    if o.paymentCaptured { o.refund() }   // side effect vẫn if — còn ít thì còn ổn
    return nil
}
```

Với đa số module backend, **V2 là điểm dừng đúng**: bảng chuyển thành single source of truth, thêm trạng thái = thêm dòng vào bảng, PM đọc được bảng để đối chiếu nghiệp vụ. V2 chỉ hụt hơi khi *hành vi trong mỗi trạng thái* phân hóa mạnh — không chỉ "được chuyển đi đâu" mà cả "Cancel lúc này làm gì" khác hẳn nhau theo trạng thái.

**Lời giải 2 — State pattern.** Khi mỗi trạng thái là một cụm hành vi thật sự:

```go
// ✅ V3 — mỗi trạng thái MỘT TYPE, gom trọn hành vi của nó về một nhà
type orderState interface {
    Cancel(o *Order) error
    Ship(o *Order) error
    Return(o *Order) error
}

type confirmedState struct{}
func (confirmedState) Cancel(o *Order) error {
    if o.paymentCaptured { o.refund() }
    o.setState(cancelledState{})          // ← trạng thái TỰ quyết định chuyển đi đâu
    return nil
}
func (confirmedState) Ship(o *Order) error   { o.setState(shippedState{}); /*...*/ return nil }
func (confirmedState) Return(o *Order) error { return ErrInvalidAction }

type shippedState struct{}
func (shippedState) Cancel(o *Order) error { return errors.New("cannot cancel shipped order") }
/* ... */

// Order ủy quyền mọi hành động cho state hiện tại — không còn MỘT câu if trạng thái nào
func (o *Order) Cancel() error { return o.state.Cancel(o) }
```

Đọc `confirmedState` là thấy **toàn bộ** những gì có thể xảy ra ở trạng thái confirmed — tri thức gom theo chiều dọc, thêm trạng thái mới = thêm một type (OCP theo trục trạng thái). Giá phải trả: N trạng thái × M hành động method (đa số trả `ErrInvalidAction` — boilerplate); bảng chuyển tổng thể lại *tan vào* các method (V2 có bảng, V3 mất bảng — muốn cả hai phải kỷ luật tài liệu); và serialize state object ↔ cột `status` trong DB cần mapping thêm.

**Ma trận quyết định:**

```
Hành vi theo trạng thái đơn giản, bảng chuyển là tài sản chính  → V2 (bảng + if)
Hành vi phân hóa mạnh, mỗi trạng thái một thế giới con          → V3 (State pattern)
Vòng đời phức tạp + phân tán + cần audit từng bước chuyển       → event sourcing / workflow
                                                                   engine (Level 5, Saga)
```

## 3. So sánh khách quan: Strategy vs State

Cấu trúc UML **giống hệt nhau** (context ủy quyền cho interface, N implementation) — phân biệt nằm trọn trong ba câu hỏi runtime:

| | Strategy | State |
|---|---|---|
| **Ai chọn implementation?** | **Client/caller** — chọn lúc lắp ráp, từ bên ngoài | **Chính các state** — mỗi state quyết định chuyển tiếp đi đâu, từ bên trong |
| **Đổi implementation giữa chừng?** | Hiếm — chọn xong thường giữ nguyên vòng đời | **Liên tục** — chuyển trạng thái là bản chất của pattern |
| **Các implementation có biết nhau?** | Không — các strategy độc lập, không quan tâm nhau | **Có** — state này nắm luật chuyển sang state kia |
| **Câu nghiệp vụ tương ứng** | "Tính phí *bằng cách nào*?" | "Đơn hàng *đang ở đâu* trong vòng đời?" |
| **Production** | sort comparator, retry policy, compression codec | TCP connection state, order lifecycle, circuit breaker (closed→open→half-open) |

Circuit breaker là ví dụ chốt đáng nhớ: ba trạng thái, hành vi `Call()` khác hẳn nhau (closed: gọi thật + đếm lỗi; open: fail ngay; half-open: cho một request thăm dò), và **tự chuyển** theo kết quả — State pattern giáo khoa sống trong mọi thư viện resilience (gobreaker, resilience4j, opossum).

## 4. So sánh khách quan: Interface vs Generics — chọn cơ chế đa hình nào

Chương 1.4 đã cho quy tắc nhanh; giờ đủ ngữ cảnh để mổ đầy đủ — vì Strategy chính là nơi câu hỏi này nổi lên nhiều nhất từ Go 1.18:

```go
// Cùng một bài "xử lý theo kiểu khác nhau" — hai cơ chế:

// Interface — dispatch RUNTIME: các kiểu KHÁC NHAU sống chung một danh sách
func Render(shapes []Shape) { for _, s := range shapes { s.Draw() } }
// []Shape chứa Circle lẫn Square — quyết định Draw nào chạy: lúc chạy, từng phần tử

// Generics — đơn hình hóa COMPILE-TIME: mỗi lần dùng chốt MỘT kiểu
func Sum[T Number](xs []T) T { var s T; for _, x := range xs { s += x } return s }
// Sum[int] và Sum[float64] là hai hàm riêng sau compile — []T thuần nhất một kiểu
```

| Tiêu chí | Interface | Generics |
|---|---|---|
| Quyết định kiểu lúc | Runtime (dynamic dispatch) | Compile-time (monomorphization) |
| Hỗn hợp kiểu trong một collection | ✅ bản chất | ❌ một instance một kiểu |
| Hành vi *khác nhau* theo kiểu | ✅ mỗi type tự implement | ❌ generics đòi hành vi *giống nhau*, chỉ kiểu dữ liệu khác |
| Hiệu năng | Gián tiếp qua itable; giá trị có thể escape lên heap | Code chuyên biệt hóa, inline được — nhanh hơn ở hot path |
| Chi phí trừu tượng | Thấp, quen thuộc | Chữ ký phức tạp nhanh; constraint lan truyền qua call chain |
| Bài toán mẫu | Strategy, plugin, DI, mock — **hành vi biến thiên** | Container, slice/map helper, thuật toán số — **kiểu biến thiên, hành vi bất biến** |

Câu thần chú của cộng đồng Go: **"generics cho kiểu, interface cho hành vi"** — viết Strategy bằng generics (`Processor[T PaymentMethod]`) là dùng sai công cụ: bạn mất khả năng chứa hỗn hợp và mất dispatch runtime, đổi lấy tối ưu không cần thiết. Ngược lại, viết `Map/Filter` bằng interface + type assertion (kiểu Go cũ) là trả giá runtime cho thứ compile-time làm được sạch hơn. Hai công cụ **trực giao, không cạnh tranh** — và kết hợp được: `func SortBy[T any](xs []T, less func(a, b T) bool)` — generics giữ kiểu, function value (strategy) giữ hành vi: chính là chữ ký của `slices.SortFunc`.

TS đối chiếu: ranh giới mờ hơn (generics TS chỉ là type-level, không monomorphize, không khác biệt hiệu năng) — nên bên TS câu hỏi này nhẹ; bên Go nó là quyết định thiết kế thật với hệ quả đo được.

## 5. Anti-pattern & khi nào KHÔNG dùng

- **Strategy cho thứ không biến thiên**: interface `TaxCalculator` + đúng một `VNTaxCalculator` ba năm — bậc thang leo trước bằng chứng (đã điểm mặt ở 1.5, 4.1; nhắc vì Strategy là nơi phạm nhiều nhất).
- **State pattern cho hai trạng thái**: `active/inactive` với một câu if — V3 cho bài toán V1 chưa có.
- **Strategy chọn bằng switch bên trong context**: context nhận tên rồi `switch name { case "a": s = &A{} }` ngay trong method nghiệp vụ — chọn strategy là việc của composition root/factory; trộn vào context là mất luôn lợi ích tách biệt.
- **Máy trạng thái ngầm bằng boolean**: `isPaid, isShipped, isCancelled, isReturned` — 4 boolean = 16 tổ hợp trong khi chỉ 6 hợp lệ; các tổ hợp vô nghĩa (`isCancelled && isShipped`) tồn tại được trong bộ nhớ và DB. Một enum trạng thái + bảng chuyển (V2) là thuốc — "make invalid states unrepresentable" (1.3/1.5) áp cho vòng đời.

---

*Tiếp theo: [4.8 — Observer & pub/sub](/series/software-design/level-4-patterns/08-observer-pubsub/)*
