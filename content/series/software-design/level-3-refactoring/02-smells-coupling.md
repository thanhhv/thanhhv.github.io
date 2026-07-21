+++
title = "3.2 — Code Smells nhóm coupling: Duplicate Code, Shotgun Surgery, Feature Envy, Message Chains, Middle Man"
date = "2026-07-17T09:30:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

Nhóm smell này chung bệnh sinh: **tri thức đặt sai chỗ** — hoặc một tri thức bị nhân bản nhiều nơi, hoặc hành vi sống xa dữ liệu của nó, hoặc cấu trúc nội bộ bị phơi ra cho kẻ khác dựa vào.

---

## A. Duplicate Code

### Ví dụ — ba cấp độ, cấp sau khó thấy hơn cấp trước

```go
// Cấp 1 — trùng nguyên văn (dễ thấy, tool bắt được):
// hai handler cùng 12 dòng parse-validate-load-user y hệt nhau

// Cấp 2 — trùng cấu trúc, khác chi tiết:
func exportOrdersCSV(orders []Order) []byte { /* mở buffer, ghi header, loop, format, flush */ }
func exportUsersCSV(users []User) []byte    { /* mở buffer, ghi header, loop, format, flush */ }

// Cấp 3 — trùng TRI THỨC, code trông khác hẳn nhau (nguy hiểm nhất):
// Rule "đơn >= 500k miễn ship" xuất hiện ở:
//   - Go backend:   if total >= 500_000 { fee = 0 }
//   - TS frontend:  total >= 500_000 ? "Miễn phí" : "20.000đ"
//   - SQL report:   CASE WHEN total >= 500000 THEN 0 ...
//   - Chatbot config: "đơn từ 500 nghìn được freeship nha!"
```

### Nguyên nhân

Copy-paste là đường đi ngắn nhất lúc viết; hai người viết cùng logic không biết nhau; và ranh giới hệ thống (frontend/backend/report) khiến trùng lặp tri thức gần như *bắt buộc* về mặt kỹ thuật — nhưng không ai ghi sổ để đồng bộ.

### Hậu quả

Sửa một quên một → hai nơi cùng trả lời một câu hỏi nghiệp vụ *khác nhau* — với cấp 3, đó là khách thấy "miễn phí" trên web nhưng bị tính 20k lúc thanh toán: bug nghiệp vụ nhìn thấy bằng mắt thường, xói mòn niềm tin. Chi phí sửa nhân theo số bản sao, và tệ hơn: *không ai biết có bao nhiêu bản sao*.

### Refactoring phù hợp — theo đúng cấp độ

- Cấp 1: **Extract Method/Function** — gom về một nơi, xong.
- Cấp 2: tách phần *khung* khỏi phần *khác biệt* — khung thành hàm nhận phần khác biệt qua tham số (function value hoặc generics): mầm của **Template Method/Strategy** (Level 4).
- Cấp 3: không giải được bằng extract — giải bằng **single source of truth có chủ đích**: backend trả `shipping_fee` + `free_ship_threshold` trong API để frontend *hiển thị* chứ không *tính lại*; report đọc từ cột đã tính thay vì tính lại bằng SQL. Khi bắt buộc trùng (validation client-side cho UX), ghi sổ rõ: *"rule này có 2 bản sao, đây là danh sách"*.

### Khi nào KHÔNG phải smell

Nhắc lại từ 2.6 vì đây là chỗ hay lạm sát nhất: **trùng hình thức ≠ trùng tri thức**. Hai đoạn giống nhau nhưng sẽ tiến hóa theo hai lý do khác nhau → gộp là tạo wrong abstraction. Test code trùng lặp cho dễ đọc — chấp nhận được. Giữa hai microservice — copy thường lành mạnh hơn shared library nghiệp vụ (coupling deploy). Rule of Three luôn áp dụng.

---

## B. Shotgun Surgery & Divergent Change — cặp smell soi bằng git

Hai smell đối ngẫu, định nghĩa bằng ma trận *thay đổi × module*:

```
Shotgun Surgery:   MỘT lý do thay đổi  → phải sửa NHIỀU module   (tri thức bị rải)
Divergent Change:  NHIỀU lý do thay đổi → cùng sửa MỘT module    (nhiều tri thức bị nhồi — chính là vi phạm SRP, đã phân tích ở 2.1)
```

### Ví dụ (Shotgun Surgery)

"Thêm phương thức thanh toán ZaloPay" đụng: `payment/processor.go` (switch), `payment/fee.go` (switch khác), `api/response.go` (label hiển thị), `report/reconcile.go` (đối soát), `webhook/router.go`, `admin/filter.go` — 6 file, 4 package. Sót một chỗ = bug im lặng. Đây chính là hiện trường mở đầu chương OCP (2.2).

### Nguyên nhân & cách nhận diện

Một khái niệm nghiệp vụ (phương thức thanh toán) không có *nhà* — mỗi khía cạnh của nó (phí, tên hiển thị, cách đối soát) sống nhờ nhà người khác. Nhận diện bằng dữ liệu, không bằng cảm giác: các file **luôn xuất hiện cùng nhau trong một commit** là các mảnh của một khái niệm bị rải — git log cho câu trả lời:

```bash
# những cặp file hay được sửa chung (co-change) — bằng chứng khách quan của Shotgun Surgery
git log --format=%h --name-only | awk '...'   # hoặc dùng code-maat / git-truck
```

### Hậu quả & Refactoring

Chi phí mỗi thay đổi = chi phí *tìm đủ* mọi nơi cần sửa — tri thức về "danh sách nơi cần sửa" sống trong đầu vài người cũ; họ nghỉ việc là chi phí thành rủi ro. Thuốc: **Move Method/Move Field** gom các mảnh về một nhà — với ví dụ trên là gom về `Gateway` interface + registry như chương 2.2 đã làm trọn vẹn. Lưu ý chiều ngược: chữa Shotgun Surgery quá tay (nhồi tất cả về một chỗ) tạo ra Divergent Change — hai smell này là hai bờ vực của cùng con đường, đích đến là *một lý do thay đổi ↔ một module*.

---

## C. Feature Envy

### Ví dụ

```go
// ❌ method của InvoiceService nhưng "ghen tị" với dữ liệu của Order
func (s *InvoiceService) CalcInvoiceTotal(o *order.Order) int64 {
    var sum int64
    for _, item := range o.Items() {                    // dữ liệu của Order
        sum += item.Price * int64(item.Qty)             // dữ liệu của Order
        if item.Category == "digital" {                 // dữ liệu của Order
            sum -= item.Price * int64(item.Qty) / 10    // rule của... ai?
        }
    }
    if o.Customer().Tier() == "gold" {                  // dữ liệu của Order
        sum = sum * 95 / 100
    }
    return sum
}
```

Đếm truy cập: method đọc dữ liệu của `Order` bảy lần, dữ liệu của chính `InvoiceService` **không lần nào**. Nó là method của Order đang sống lưu vong.

### Nguyên nhân

Thường là hệ quả của kiến trúc "service làm hết, model chỉ chứa data" (Anemic Domain Model — 2.1): mọi logic bị hút về tầng service theo quán tính, bất kể logic đó thuộc về dữ liệu nào. Cũng có khi vì model nằm ở package mà dev ngại sửa (owned bởi team khác, hoặc sinh tự động từ ORM).

### Hậu quả

Logic của Order rải ở N service → sửa cách tính tiền phải đi săn khắp các service (Shotgun Surgery); `Order` buộc phơi hết ruột ra (getter cho mọi field — encapsulation chết, chương 1.2); và rule "digital giảm 10%" không có chủ — hai service tính hai kiểu khi một bên sửa.

### Refactoring phù hợp

**Move Method** (3.3) — chuyển hành vi về sống với dữ liệu:

```go
// ✅ Order tự trả lời câu hỏi về tiền của nó — Tell, Don't Ask (2.6)
func (o *Order) Total() int64 { /* toàn bộ logic trên, giờ ở NHÀ của nó */ }

// InvoiceService chỉ còn việc của invoice thật: định dạng, xuất, lưu
func (s *InvoiceService) Generate(o *order.Order) (*Invoice, error) {
    total := o.Total()
    /* ... */
}
```

### Khi nào KHÔNG phải smell

Hàm *điều phối* nhiều object ngang hàng (use case, orchestrator) tất nhiên đọc dữ liệu của nhiều bên — đó là việc của nó, miễn nó không *tính toán hộ* bên nào. Và có ngoại lệ chiến lược: khi logic cần dữ liệu của **hai** domain (tính điểm loyalty cần Order + Campaign), đặt ở một trong hai đều khiên cưỡng → một domain service đứng giữa là lựa chọn đúng (DDD, Level 5). Feature Envy rõ án nhất khi ghen tị với **một** object duy nhất.

---

## D. Message Chains

### Ví dụ

```go
rate := order.GetCustomer().GetMembership().GetTier().GetDiscountRate()
```

Đã phân tích kỹ ở Law of Demeter (2.6) — ở đây đặt vào khung smell cho đủ danh mục.

### Nguyên nhân — Hậu quả — Thuốc

Nguyên nhân: object graph phơi cấu trúc, caller tự đi bộ qua graph lấy thứ nó cần — vì "nhanh hơn là xin thêm method". Hậu quả: caller coupling với **hình dạng** của cả chuỗi — đổi một khớp giữa chuỗi vỡ mọi caller (đặc biệt đau: chuỗi xuất hiện trong template/handler rải khắp nơi). Thuốc: **Hide Delegate** — mỗi mắt xích cung cấp câu trả lời thay vì cung cấp hàng xóm (`order.DiscountRate()`); hoặc khi chuỗi chỉ để *lấy dữ liệu hiển thị*, cắt hẳn bằng một DTO phẳng dựng ở tầng query.

Nhắc lại ranh giới quan trọng nhất từ 2.6: **chấm qua data (DTO, config, JSON) không phải smell** — cấu trúc của data chính là hợp đồng. Smell chỉ tính khi chấm xuyên qua *object có hành vi và invariant*. Và cẩn thận mặt trái: Hide Delegate quá tay sinh ra smell kế tiếp.

---

## E. Middle Man

### Ví dụ

```go
// ❌ OrderService: 9/10 method chỉ forward — một trạm trung chuyển không tạo giá trị
func (s *OrderService) GetOrder(ctx context.Context, id string) (*Order, error) {
    return s.repo.GetOrder(ctx, id)
}
func (s *OrderService) ListOrders(ctx context.Context, f Filter) ([]*Order, error) {
    return s.repo.ListOrders(ctx, f)
}
/* ... 7 method forward nữa ... */
```

### Nguyên nhân

Hai nguồn chính: (1) kiến trúc tầng bậc **áp dụng như nghi lễ** — "mọi handler phải gọi qua service, mọi service phải gọi qua repo" kể cả khi tầng giữa không có gì để làm; (2) hậu quả của refactor dở dang — Hide Delegate/Extract Class xong, class cũ chỉ còn vỏ ủy quyền nhưng không ai dọn.

### Hậu quả

Mỗi thay đổi chữ ký xuyên qua N tầng photocopy (thêm một tham số = sửa 3 interface + 3 struct); người đọc nhảy qua các tầng rỗng để tìm nơi logic thật sự sống; và tầng rỗng *che khuất* câu hỏi đáng giá: "service này tồn tại để làm gì?"

### Refactoring phù hợp

**Remove Middle Man / Inline**: cho handler gọi thẳng repo với các đường đọc đơn giản. Trong Go, không có luật nào cấm handler gọi repository trực tiếp khi giữa chúng không có nghiệp vụ — tầng service chỉ dựng ở những use case *có* logic (transaction nhiều bước, gọi nhiều domain). Kiến trúc tốt cho phép **số tầng khác nhau theo từng luồng**, thay vì đồng phục N tầng cho mọi luồng.

### Khi nào KHÔNG phải smell

Delegation có *chủ đích* không phải Middle Man: facade đơn giản hóa một subsystem phức tạp (Facade, Level 4); adapter đổi interface; decorator thêm hành vi (retry/log); anti-corruption layer chặn model bên ngoài (DDD). Phép thử: **lớp giữa có *biến đổi* gì không — đổi interface, thêm hành vi, che phức tạp?** Có → thiết kế. Không, chỉ photocopy chữ ký → Middle Man.

---

## Tổng kết chương — bảng tra nhanh

| Smell | Câu hỏi chẩn đoán | Thuốc chính | Liên hệ nguyên lý |
|---|---|---|---|
| Duplicate Code | Trùng *tri thức* hay chỉ trùng *hình thức*? | Extract Function; single source of truth; ghi sổ bản sao bắt buộc | DRY (2.6) |
| Shotgun Surgery | Một thay đổi nghiệp vụ đụng bao nhiêu file? (soi git co-change) | Move Method/Field gom về một nhà | OCP (2.2), cohesion (1.3) |
| Divergent Change | Module này bị sửa vì bao nhiêu lý do khác nhau? | Extract Class tách theo lý do | SRP (2.1) |
| Feature Envy | Method này đọc dữ liệu của ai nhiều nhất? | Move Method về nhà của dữ liệu | Tell Don't Ask (2.6) |
| Message Chains | Chuỗi chấm xuyên qua object hay data? | Hide Delegate; DTO phẳng cho đường đọc | Law of Demeter (2.6) |
| Middle Man | Tầng giữa có biến đổi gì hay chỉ forward? | Remove Middle Man; tầng theo nhu cầu từng luồng | KISS (2.6) |

Quan sát xâu chuỗi: sáu smell này không độc lập — chúng là **các pha của cùng một dòng chảy**. Anemic model sinh Feature Envy; Feature Envy rải logic sinh Duplicate Code và Shotgun Surgery; chữa bằng Hide Delegate quá tay sinh Middle Man. Kim chỉ nam duy nhất xuyên suốt: **tri thức về một khái niệm sống ở một nơi, hành vi sống cạnh dữ liệu, cấu trúc nội bộ không phải hợp đồng.**

---

*Tiếp theo: [3.3 — Kỹ thuật refactoring cơ bản](/series/software-design/level-3-refactoring/03-ky-thuat-co-ban/)*
