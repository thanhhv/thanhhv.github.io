+++
title = "3.3 — Kỹ thuật refactoring cơ bản: Extract Method, Move Method, Extract Class, Introduce Parameter Object"
date = "2026-07-17T09:40:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

Bốn thao tác "dao kéo hằng ngày" — 90% công việc refactoring thực tế là bốn thao tác này lặp đi lặp lại. Mỗi kỹ thuật trình bày: khi nào dùng → các bước cơ giới (mechanics) → bẫy đặc thù → ví dụ Go. Phần mechanics quan trọng hơn vẻ ngoài của nó: refactoring an toàn là *chuỗi bước nhỏ đến mức nhàm chán*, mỗi bước giữ test xanh — không phải "đập ra sắp lại theo trí nhớ".

> Lưu ý công cụ: Gopls (VS Code/GoLand) tự động hóa được Extract Function/Variable và Rename — hãy để máy làm phần cơ giới. Move Method và Extract Class trong Go phần lớn vẫn là thao tác tay + compiler dẫn đường.

---

## A. Extract Method (Extract Function)

**Bài toán nó giải**: Long Method (3.1); đoạn code cần comment giải thích; logic trùng lặp cấp 1 (3.2).

### Mechanics — thứ tự các bước ăn tiền ở chỗ nào cũng compile được

```
1. Tạo hàm mới, đặt tên theo Ý ĐỊNH (làm gì) — không theo cách làm
2. Copy đoạn code vào hàm mới
3. Biến cục bộ đoạn đó dùng → thành tham số; biến nó gán mà bên ngoài cần → thành giá trị trả về
4. Thay đoạn gốc bằng lời gọi. Compile. Test. Commit.
```

Bước 3 là nơi Go giúp đắc lực: compiler chỉ ra *chính xác* biến nào cần vào/ra — cứ copy trước rồi để lỗi compile dẫn đường.

### Ví dụ — kèm quyết định tinh tế nhất: đặt tên

```go
// Trước: 8 dòng + 1 comment
// check xem khách có được COD không
eligible := false
if c.Verified && c.FailedCODCount < 3 {
    if c.TotalSpent > 2_000_000 || c.OrderCount >= 5 {
        eligible = true
    }
}
```

```go
// Sau: comment CHẾT, tên hàm SỐNG thay nó
if c.EligibleForCOD() { ... }

func (c *Customer) EligibleForCOD() bool {
    trusted := c.Verified && c.FailedCODCount < 3
    loyal := c.TotalSpent > 2_000_000 || c.OrderCount >= 5
    return trusted && loyal
}
```

Để ý hai điều: tên hàm nói *ý định nghiệp vụ* (`EligibleForCOD`), không nói cơ chế (`checkVerifiedAndSpentAndCount`); và bên trong, hai biến trung gian `trusted`/`loyal` tiếp tục dán nhãn nghĩa cho từng cụm điều kiện (Extract Variable — em ruột của Extract Method).

### Bẫy

- Hàm cần 6 tham số vào + 3 giá trị ra → đoạn code đó **chưa cắt được ở chỗ đó** — cụm biến đang quấn nhau; cắt chỗ khác, hoặc gom cụm biến thành struct trước (Introduce Parameter Object), rồi cắt.
- Extract xong tên hàm không đặt nổi ("doStuffPart2") → đoạn cắt không phải một đơn vị ngữ nghĩa; hoàn tác, tìm ranh khác.

---

## B. Move Method (Move Function)

**Bài toán nó giải**: Feature Envy (3.2); chuẩn bị mặt bằng trước khi Extract Class; gom mảnh vỡ Shotgun Surgery về một nhà.

### Mechanics — kiểu "song song rồi cắt", an toàn tuyệt đối trên codebase lớn

```
1. Tạo method ở nhà MỚI (copy nguyên logic)
2. Nhà cũ: thân method thay bằng delegate sang nhà mới    ← hệ thống vẫn chạy, caller chưa biết gì
3. Đổi từng caller sang gọi thẳng nhà mới — mỗi nhóm caller một commit
4. Nhà cũ hết caller → xóa vỏ delegate. Commit.
```

Điểm quan trọng: giữa bước 2 và 4, code ở trạng thái "hai nhà cùng có method" **trong nhiều ngày cũng không sao** — refactor trên codebase lớn nhiều team là chuỗi trạng thái trung gian *đều chạy được*, không phải một PR 40 file.

### Ví dụ

```go
// Bước 1-2: CalcInvoiceTotal (Feature Envy từ 3.2) — logic dọn về Order, vỏ ở lại
func (o *Order) Total() int64 { /* toàn bộ logic */ }

// Deprecated: dùng order.Total(). Sẽ xóa sau khi hết caller (ticket ABC-123).
func (s *InvoiceService) CalcInvoiceTotal(o *order.Order) int64 {
    return o.Total()
}
```

Tag `// Deprecated:` được gopls/staticcheck hiểu — mọi caller còn lại hiện gạch ngang trong IDE: bộ đếm ngược tự nhiên cho bước 3.

### Bẫy

- Method dùng dữ liệu của *cả hai* nhà → chưa move được ngay; tách đôi method (phần của A, phần của B) rồi move từng nửa. Nếu không tách nổi — có thể nó đúng chỗ rồi, hoặc cần nhà thứ ba (domain service).
- Move theo "chủ đề" thay vì theo dữ liệu-nó-dùng: nghe hợp lý ("cái này liên quan invoice mà!") nhưng tạo Feature Envy mới theo chiều ngược lại. Kim chỉ nam duy nhất: **method về ở với dữ liệu nó đọc nhiều nhất.**

---

## C. Extract Class

**Bài toán nó giải**: Large Class / God Object, Divergent Change (một class nhiều lý do thay đổi).

### Mechanics

```
1. Xác định cụm trách nhiệm cần tách (soi: cụm field + cụm method dùng chúng + cụm dependency)
2. Tạo class/struct mới, đặt tên theo trách nhiệm
3. Move Field từng cái: field chuyển sang class mới, class cũ giữ tham chiếu tới class mới
4. Move Method từng cái (kỹ thuật B) — mỗi lần một method, test xanh, commit
5. Quyết định quan hệ cuối: class cũ CHỨA class mới (ủy quyền), hay hai class NGANG HÀNG
   được wiring ở composition root — thường bước này mới lộ ra thiết kế đúng
```

### Ví dụ — tách `OrderManager` (từ 3.1), khúc pricing

```go
// Bước 2-4: pricing.Engine ra ở riêng
type Engine struct {
    products ProductSource   // 2 dep, tách từ 8 dep của OrderManager
    cache    PriceCache
}
func (e *Engine) Quote(items []Item, customerID string, coupon string) Quote { /* ... */ }

// Bước 5 — hai lựa chọn kết thúc:
// (a) OrderManager giữ Engine bên trong, caller cũ không đổi:
type OrderManager struct {
    pricing *pricing.Engine
    /* ... */
}
func (m *OrderManager) CalcDiscount(...) ... { return m.pricing.Quote(...) } // vỏ, teo dần

// (b) Ngang hàng — handler nhận *pricing.Engine trực tiếp từ main:
//     đích cuối cùng; (a) là trạng thái trung gian hợp lệ trên đường đến (b)
```

### Bẫy

- **Tách theo layer kỹ thuật thay vì theo trách nhiệm**: bổ `OrderManager` thành `OrderManagerHelper` + `OrderManagerUtils` — vẫn một mớ, giờ hai file. Ranh giới đúng đo bằng *lý do thay đổi* và *cụm dependency*, đã có từ 3.1.
- **Hai class mới gọi nhau chằng chịt** sau khi tách → ranh giới đặt sai chỗ (cắt ngang một cohesion thật); hợp lại và tìm đường ranh khác. Extract Class thất bại có hoàn tác rẻ *nếu* đi bằng bước nhỏ — thêm một lý do để không big-bang.

---

## D. Introduce Parameter Object

**Bài toán nó giải**: Data Clumps, Long Parameter List (3.1); và mở đường cho hành vi tụ về (như đã thấy với `Address`).

### Mechanics

```
1. Khai báo struct mới cho cụm tham số
2. Thêm tham số struct vào CUỐI chữ ký hàm, giữ nguyên các tham số cũ  ← hàm nhận cả hai
3. Đổi từng caller: dựng struct, truyền vào (tham số cũ truyền zero value dần)
4. Hết caller cũ → xóa các tham số lẻ. Đổi thân hàm dùng struct.
5. (Phần lãi kép) Nhìn quanh: hành vi nào thuộc về struct mới? → Move Method về
```

### Ví dụ — cơn đau có thật của mọi hàm search/report

```go
// Trước — chữ ký chỉ có tác giả hiểu, mọi call site là câu đố
func SearchOrders(from, to time.Time, status string, minTotal, maxTotal int64,
    customerID string, limit, offset int, sortBy string, desc bool) ([]Order, error)

SearchOrders(f, t, "paid", 0, 0, "", 20, 0, "created_at", true) // 0 là "không lọc" hay "lọc = 0"?!
```

```go
// Sau — cụm tham số thành khái niệm có tên + zero value có nghĩa
type OrderQuery struct {
    Range      TimeRange     // value object từ 1.5, tái sử dụng
    Status     string        // "" = mọi status (zero value = không lọc, quy ước NHẤT QUÁN)
    Total      *MoneyRange   // nil = không lọc — phân biệt được với "lọc từ 0đ"
    CustomerID string
    Page       Pagination    // limit/offset/sort gom riêng — cụm trong cụm
}

func SearchOrders(ctx context.Context, q OrderQuery) ([]Order, error)

// Call site tự tài liệu hóa:
SearchOrders(ctx, OrderQuery{
    Range:  ThisMonth(),
    Status: "paid",
    Page:   Pagination{Limit: 20, Sort: "created_at", Desc: true},
})
```

Go có field name khi khởi tạo struct → parameter object còn đóng luôn vai **named & optional arguments** mà ngôn ngữ không có. TypeScript: cùng mẫu với options object + destructuring — `function search({ range, status, page }: OrderQuery)` — đến mức nó là *convention mặc định* của cả hệ sinh thái JS.

### Bẫy

- **Struct rác tổng hợp**: gom 10 tham số *không liên quan* vào `SearchParams` cho gọn chữ ký — cụm không phải khái niệm thì struct chỉ giấu bụi xuống thảm. Mỗi parameter object phải trả lời được "đây là cái gì?" bằng một danh từ nghiệp vụ.
- **Zero value mơ hồ**: `minTotal int64` — 0 là "không lọc" hay "từ 0 đồng"? Quy ước từng field một (con trỏ, kiểu Optional, hoặc field `HasX bool`) và nhất quán toàn struct.
- Với hàm khởi tạo có nhiều tùy chọn (không phải query), Go còn một lựa chọn khác cùng họ: Functional Options — so găng trực tiếp ở Level 4.

---

## Tổng kết chương

| Kỹ thuật | Chữa smell | Chốt an toàn |
|---|---|---|
| Extract Method | Long Method, Duplicate cấp 1 | Tên theo ý định; compiler dẫn tham số; không đặt nổi tên = cắt sai chỗ |
| Move Method | Feature Envy, Shotgun Surgery | Copy → delegate → dời caller dần → xóa vỏ; hai nhà cùng có method là trạng thái hợp lệ |
| Extract Class | God Object, Divergent Change | Ranh theo cụm field+method+dep; đi qua trạng thái ủy quyền trung gian |
| Introduce Parameter Object | Data Clumps, Long Parameter List | Cụm phải là khái niệm có tên; zero value phải có nghĩa nhất quán |

Cả bốn kỹ thuật cộng lại thành một vòng khép kín: Extract Method lộ ra các cụm logic → cụm logic + cụm data gợi ý Extract Class / Parameter Object → class mới thành nam châm cho Move Method → code sau khi gom lại lộ ra cấu trúc lặp *có hình thù* — và đó là lúc cần các kỹ thuật cấu trúc của chương sau, nơi pattern bắt đầu ló dạng.

---

*Tiếp theo: [3.4 — Kỹ thuật refactoring cấu trúc: cây cầu sang Design Patterns](/series/software-design/level-3-refactoring/04-ky-thuat-cau-truc/)*
