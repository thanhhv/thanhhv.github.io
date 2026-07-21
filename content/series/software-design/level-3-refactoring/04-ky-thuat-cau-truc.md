+++
title = "3.4 — Kỹ thuật refactoring cấu trúc: Replace Conditional with Polymorphism, Replace Inheritance with Composition, Replace Loop with Strategy"
date = "2026-07-17T09:50:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

Ba kỹ thuật này khác nhóm 3.3 về tầm vóc: chúng không dọn dẹp *bên trong* một hàm/class mà **đổi cấu trúc phân nhánh và quan hệ giữa các type** — và điểm đến của chúng thường *chính là* một design pattern. Đây là cây cầu sang Level 4: pattern không phải thứ áp vào từ đầu, mà là **hình dạng code sau khi các refactoring này hoàn thành**.

---

## A. Replace Conditional with Polymorphism

**Bài toán nó giải**: cùng một cấu trúc switch/if-else trên cùng một "loại" bị lặp ở nhiều hàm — mỗi lần thêm loại phải đi săn đủ các bản sao (đã phân tích tiêu chí đầy đủ ở OCP, chương 2.2).

### Điểm khởi hành

```go
// ❌ switch trên shipMethod xuất hiện ở 4 hàm khác nhau trong package
func fee(o Order) int64 {
    switch o.ShipMethod {
    case "standard": return 20_000
    case "express":  return 45_000
    case "same_day": return o.Weight() > 5 ? /* ... logic riêng ... */
    }
}
func eta(o Order) time.Duration { switch o.ShipMethod { /* ... lặp lại cấu trúc ... */ } }
func carrier(o Order) string    { switch o.ShipMethod { /* ... lặp lại ... */ } }
func label(o Order) string      { switch o.ShipMethod { /* ... lặp lại ... */ } }
```

Nhìn theo **ma trận loại × thao tác**: 3 loại × 4 thao tác = 12 ô logic. Code hiện tại tổ chức theo *hàng* (mỗi thao tác một hàm chứa mọi loại). Polymorphism tổ chức lại theo *cột* (mỗi loại một type chứa mọi thao tác của nó).

### Mechanics — từng bước giữ hệ thống chạy

```
1. Khai interface với method đầu tiên (Fee)
2. Mỗi loại một struct implement Fee — copy logic từ case tương ứng
3. Hàm fee() cũ: thay switch bằng dispatch qua interface — nhưng GIỮ hàm cũ làm vỏ
4. Test xanh → lặp bước 1-3 cho eta, carrier, label — mỗi method một commit
5. Vỏ hàm cũ hết việc → inline, xóa. Nơi tạo object từ string ("standard" → StandardShipping{})
   trở thành ĐIỂM SWITCH DUY NHẤT còn lại — và nó có tên: Factory (Level 4)
```

### Điểm đến

```go
// ✅ mỗi loại một type — thêm loại mới = thêm MỘT file, không sửa file nào
type ShipMethod interface {
    Fee(o Order) int64
    ETA(o Order) time.Duration
    Carrier() string
    Label() string
}

type StandardShipping struct{}
func (StandardShipping) Fee(o Order) int64        { return 20_000 }
func (StandardShipping) ETA(o Order) time.Duration { return 72 * time.Hour }
/* ... */

type SameDayShipping struct{ cutoff time.Hour }   // loại phức tạp được QUYỀN có state riêng
/* ... */

// Điểm phân loại cuối cùng — một nơi duy nhất, có kiểm soát:
func ParseShipMethod(s string) (ShipMethod, error) { /* map lookup */ }
```

Đây chính là **Strategy** (mỗi variant một hành vi) hoặc **State** (nếu variant đổi theo vòng đời object) — Level 4 phân biệt hai anh em này. Điều đáng ghi nhớ: bạn đến được đây bằng chuỗi bước cơ giới từ code switch có thật, không phải bằng cách vẽ UML trước.

### Trade-off — nhắc lại có chủ đích

Chuyển từ hàng sang cột là **đánh đổi hai chiều mở rộng** (expression problem, 2.2): thêm *loại* giờ rẻ (một file mới), thêm *thao tác chung* giờ đắt (sửa interface + mọi struct). Switch ba case ở đúng một nơi, năm không đổi một lần → **giữ nguyên switch là quyết định đúng**. Kỹ thuật này chỉ đáng giá khi ma trận đủ lớn và chiều "thêm loại" nóng hơn chiều "thêm thao tác".

---

## B. Replace Inheritance with Composition

**Bài toán nó giải**: cây kế thừa bắt đầu gãy — con override để *tắt* hành vi cha, cha sửa là con vỡ, tổ hợp tính năng bùng nổ lớp (toàn bộ bệnh án ở chương 1.4).

### Mechanics — quan trọng vì thao tác này hay bị làm kiểu big-bang

Trình bày bằng TypeScript vì đây là refactoring của thế giới có `extends` (dev Go gặp nó khi... port code hoặc review codebase TS):

```
1. Trong lớp con, tạo field trỏ tới MỘT INSTANCE của lớp cha (composition bắt đầu)
2. Từng method kế thừa mà con thực sự dùng → viết method delegate tường minh sang field đó
3. Cắt quan hệ extends. Compiler chỉ ra mọi chỗ còn dựa vào kế thừa ngầm — xử từng cái
4. Interface hẹp hóa: con giờ chỉ expose method NÓ CHỦ ĐÍCH cung cấp
   (không còn thừa hưởng bừa 30 method public của tổ tiên)
5. Lớp "cha" mất vai trò cha → thường tách tiếp thành các mảnh chức năng (Extract Class)
   mà các "con cũ" compose theo nhu cầu thật
```

### Ví dụ rút gọn

```typescript
// Trước: Stack extends Array — thừa hưởng cả push/pop LẪN splice, sort, reverse...
class Stack<T> extends Array<T> {}       // ai đó gọi stack.splice() → invariant LIFO vỡ

// Sau: Stack CHỨA Array — expose đúng 3 method, invariant được niêm phong
class Stack<T> {
  private items: T[] = [];
  push(x: T)  { this.items.push(x); }
  pop(): T | undefined { return this.items.pop(); }
  peek(): T | undefined { return this.items.at(-1); }
}
```

Ví dụ `Stack extends Array` nhỏ nhưng là nguyên mẫu của mọi ca lớn: **kế thừa để lấy code = nhận cả interface công khai mình không muốn** — và interface thừa đó là cửa phá invariant. Composition + delegate tường minh = trả giá vài dòng boilerplate để mua quyền kiểm soát trọn vẹn bề mặt public.

Trong Go, kỹ thuật này xuất hiện dưới dạng phòng ngừa: chọn **embed struct** (nâng cả method set lên — tiện nhưng lộ) hay **field thường + delegate tay** (kín nhưng dài dòng). Quy tắc: embed khi bạn *muốn* toàn bộ method set là một phần hợp đồng công khai của type mới (`bufio.ReadWriter` embed cả Reader lẫn Writer — chủ đích); field thường khi chỉ mượn năng lực bên trong (`Stack` chứa slice — không ai được `splice`).

### Điểm đến

Các mảnh chức năng độc lập + type lắp ráp chúng — tức là hình dạng **Decorator/Strategy/plain composition** như chuỗi Notifier ở 1.4. Một lần nữa: pattern là *điểm đến của refactoring*, không phải điểm xuất phát của thiết kế.

---

## C. Replace Loop with Strategy (Pipeline)

**Bài toán nó giải**: một vòng lặp "làm tất cả" — lọc + biến đổi + tính toán + side effect trộn trong một thân loop — nơi mọi yêu cầu mới đều chen thêm một nhánh if vào giữa.

### Điểm khởi hành

```go
// ❌ vòng lặp tính hoa hồng: 4 mối quan tâm bện vào nhau
func Commissions(orders []Order) map[string]int64 {
    result := map[string]int64{}
    for _, o := range orders {
        if o.Status != "completed" { continue }                    // lọc 1
        if o.Total < 100_000 { continue }                          // lọc 2
        rate := int64(5)
        if o.Seller.Tier == "gold" { rate = 8 }                    // rule hoa hồng
        if time.Since(o.CreatedAt) < 30*24*time.Hour { rate += 2 } // rule khuyến khích
        c := o.Total * rate / 100
        if c > 500_000 { c = 500_000 }                             // trần
        result[o.Seller.ID] += c
        log.Printf("commission %s: %d", o.ID, c)                   // side effect chen giữa
    }
    return result
}
```

Mỗi rule mới (loại trừ đơn hoàn, hệ số theo ngành hàng, trần theo tháng...) = chen if vào giữa thân loop → thứ tự if dần mang ngữ nghĩa ngầm, test phải chạy cả loop cho mọi rule — đúng quỹ đạo xuống dốc của `ShippingFee` chương 1.1.

### Mechanics & điểm đến — tách VÒNG LẶP khỏi VIỆC-TRONG-LẶP

```go
// ✅ mỗi mối quan tâm một mảnh, đặt tên được, test được, tổ hợp được
type OrderFilter func(Order) bool
type RateRule func(Order, int64) int64   // nhận rate hiện tại, trả rate mới

func Commissions(orders []Order, filters []OrderFilter, rules []RateRule, cap int64) map[string]int64 {
    result := map[string]int64{}
    for _, o := range orders {
        if !passAll(o, filters) { continue }
        rate := int64(0)
        for _, r := range rules { rate = r(o, rate) }
        result[o.Seller.ID] += min(o.Total*rate/100, cap)
    }
    return result
}

// Các rule giờ là hàm thuần 2-3 dòng, khai báo ở nơi cấu hình nghiệp vụ:
var completedOnly OrderFilter = func(o Order) bool { return o.Status == "completed" }
var goldBonus RateRule = func(o Order, rate int64) int64 {
    if o.Seller.Tier == "gold" { return rate + 3 }
    return rate
}
```

Bộ khung loop viết một lần và đóng lại; nghiệp vụ mở qua danh sách rule — **OCP đạt được bằng function value**, phiên bản Strategy nhẹ nhất (1.4 đã gọi tên: đa hình không cần type). Side effect (log) đẩy hẳn ra ngoài hoặc thành một rule quan sát riêng — thân loop giờ thuần túy, test bằng bảng.

### Biến thể pipeline & lời cảnh tỉnh hiệu năng

Cùng họ với chuỗi `filter/map/reduce` (TS: `orders.filter(...).map(...).reduce(...)`; Go 1.23+: iterator `iter.Seq` cho phép chuỗi lazy không cấp phát slice trung gian). Nhưng trung thực về benchmark: trong Go, **vòng for thô luôn nhanh nhất** — chuỗi filter/map trên slice cấp phát bản sao trung gian ở mỗi bước; function value ngăn inline. Chênh lệch chỉ đáng bàn ở hot path triệu phần tử — code nghiệp vụ thường thì độ rõ thắng — nhưng đó là lý do *đừng* mang phong cách "mọi loop đều phải functional" từ JS sang Go một cách tôn giáo. Đo, rồi chọn.

### Khi nào KHÔNG dùng

Loop 5 dòng, một việc, một chỗ — để nguyên; đây là kỹ thuật cho loop-nhiều-rule-hay-đổi, không phải chiến dịch xóa sổ vòng for. Và nếu các "rule" không độc lập (rule sau đọc kết quả trung gian phức tạp của rule trước), ép chúng vào danh sách phẳng sẽ tạo coupling ngầm qua tham số tích lũy — khi đó cấu trúc đúng có thể là pipeline nhiều giai đoạn có kiểu trung gian tường minh, hoặc... vẫn là cái loop cũ.

---

## Tổng kết Level 3 — và cây cầu sang Level 4

Ba chương smell + hai chương kỹ thuật gộp thành một bảng hành quân duy nhất:

| Con đường refactor | Pattern chờ ở cuối đường |
|---|---|
| Switch lặp nhiều nơi → Replace Conditional with Polymorphism | **Strategy / State**, điểm tạo object thành **Factory** |
| Tính năng chung của cây kế thừa → tách wrapper compose | **Decorator**, đôi khi **Template Method** đảo thành function value |
| Loop đa rule → tách khung khỏi rule | **Strategy (function value) / Pipeline / Chain of Responsibility** |
| God Object → Extract Class theo trách nhiệm | **Facade** (vỏ cũ ủy quyền) trên đường teo |
| Legacy không seam → tách lõi thuần khỏi vỏ I/O | **Adapter/Ports** — cửa ngõ Hexagonal (Level 5) |

Đó là luận điểm trung tâm của cả bộ tài liệu, giờ đã được chứng minh bằng đường đi cụ thể: **design pattern là hóa thạch của các refactoring thành công** — cấu trúc lặp lại đủ nhiều để được đặt tên. Level 4 sẽ đi qua các pattern với đúng tinh thần đó: mỗi pattern khai quật từ một bài toán, soi bằng nguyên lý Level 1-2, và luôn kèm câu hỏi "cách đơn giản hơn là gì?".

---

*Tiếp theo: Level 4 — Design Patterns (phần tiếp theo của bộ tài liệu).*
