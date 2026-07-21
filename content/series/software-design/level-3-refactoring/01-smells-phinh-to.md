+++
title = "3.1 — Code Smells nhóm \"phình to\": Long Method, Large Class, God Object, Primitive Obsession, Data Clumps"
date = "2026-07-17T09:20:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

Nhóm smell này chung một bệnh sinh: **một đơn vị code (hàm, struct, kiểu dữ liệu) ôm nhiều hơn mức nó nên ôm** — cohesion giảm dần mà không ai để ý, vì mỗi lần chỉ thêm "một tí".

Mỗi smell trình bày theo khung: ví dụ → nguyên nhân → hậu quả → refactoring phù hợp → khi nào KHÔNG phải smell.

---

## A. Long Method

### Ví dụ

```go
// ❌ handler 150 dòng — rút gọn còn khung xương, bạn đã thấy nó ở mọi codebase
func (h *Handler) CreateOrder(w http.ResponseWriter, r *http.Request) {
    // ...15 dòng parse + validate JSON...
    // ...10 dòng check tồn kho, gọi inventory service...
    // ...20 dòng tính giá: giảm giá theo hạng thành viên, mã coupon, phí ship...
    // ...15 dòng ghi DB trong transaction, xử lý rollback...
    // ...10 dòng gửi event Kafka, log, metric...
    // ...10 dòng build response...
}
```

### Nguyên nhân

Không ai *viết* hàm 150 dòng — hàm 20 dòng *lớn lên* thành 150 qua hai năm, mỗi PR thêm 5-10 dòng "tiện thể nhét vào đây vì context sẵn rồi". Lực hấp dẫn tự nhiên: thêm code vào hàm có sẵn luôn *dễ hơn* tạo cấu trúc mới. Long Method là default của entropy, không phải lỗi cá nhân.

### Hậu quả

Đọc phải giữ toàn bộ 150 dòng trong đầu (biến khai báo dòng 12 được gán lại dòng 90, dùng dòng 140); test chỉ vào được từ cổng HTTP — muốn test riêng logic tính giá phải dựng cả request giả + DB + Kafka; hai dev sửa hai đoạn khác nhau của cùng hàm → conflict; và các đoạn logic không tái sử dụng được (muốn tính giá ở batch job → copy).

### Refactoring phù hợp

**Extract Method** (3.3) theo nguyên tắc *mỗi hàm một mức trừu tượng*:

```go
// ✅ hàm gốc trở thành MỤC LỤC — đọc 5 giây hiểu luồng
func (h *Handler) CreateOrder(w http.ResponseWriter, r *http.Request) {
    req, err := parseCreateOrderRequest(r)
    if err != nil { respondError(w, http.StatusBadRequest, err); return }

    if err := h.inventory.Reserve(r.Context(), req.Items); err != nil {
        respondError(w, http.StatusConflict, err); return
    }
    quote := h.pricer.Quote(req.Items, req.CustomerID, req.CouponCode)
    o, err := h.orders.Create(r.Context(), req, quote)
    if err != nil { respondError(w, http.StatusInternalServerError, err); return }

    h.events.OrderCreated(r.Context(), o)
    respondJSON(w, http.StatusCreated, toOrderResponse(o))
}
```

Tín hiệu chỉ điểm chỗ cắt: **comment "// bước 1: ..."** (mỗi comment phân đoạn là một tên hàm đang chờ được đặt), **dòng trắng chia cụm**, và **biến chỉ dùng trong một cụm**.

### Khi nào KHÔNG phải smell

Đếm dòng là thước đo hạng hai; thước đo hạng nhất là **số mức trừu tượng trộn lẫn**. Hàm 60 dòng làm *một việc ở một mức* (bảng switch ánh xạ dài, hàm dựng SQL query phức tạp nhưng tuyến tính, table-driven test) — hoàn toàn ổn. Ngược lại hàm 15 dòng trộn parse + business + I/O vẫn cần mổ. Và đừng chẻ quá tay: 30 hàm 3 dòng gọi dây chuyền còn khó đọc hơn một hàm 40 dòng mạch lạc — chẻ đến khi *mỗi tên hàm nói được một điều có nghĩa*, rồi dừng.

---

## B. Large Class & God Object

### Ví dụ

```go
// ❌ "class lớn" phiên bản Go: struct + 47 method trải trên 6 file
type OrderManager struct {
    db, redis, kafka, s3, mailer, sms, pdf, logger /* 8 dependency */
}
// order_manager.go:        CreateOrder, UpdateOrder, CancelOrder, ...
// order_manager_pricing.go: CalcDiscount, ApplyCoupon, EstimateShipping, ...
// order_manager_export.go:  ExportPDF, ExportExcel, GenerateInvoice, ...
// order_manager_notify.go:  SendConfirmation, SendReminder, ...
// order_manager_report.go:  DailyStats, RevenueByRegion, ...
```

God Object là thể nặng của Large Class: object mà **mọi luồng nghiệp vụ đều phải đi qua nó**, nó biết mọi bảng DB, giữ mọi dependency, và tên của nó thường kết thúc bằng `Manager`, `Service`, `Processor`, `Handler`, `Util` — những cái tên trống nghĩa vì nó làm quá nhiều thứ để đặt tên cụ thể.

### Nguyên nhân

Ba lực cộng hưởng: (1) chi phí kích hoạt — tạo package/struct mới đòi nghĩ tên, nghĩ ranh giới, còn thêm method vào chỗ cũ thì miễn phí; (2) tổ chức code theo *danh từ to* ("mọi thứ liên quan order") thay vì theo *lý do thay đổi* — cohesion chủ đề giả (chương 1.3); (3) DI thuận tay: struct đã có sẵn 8 dependency, thêm method mới dùng luôn, khỏi wiring.

### Hậu quả

Là điểm hội tụ của mọi thay đổi → conflict git hằng tuần, không ai nắm hết, mọi PR đều đụng nó nên mọi PR đều rủi ro; test dựng cả 8 dependency cho method dùng 1; không tách được team ("phần của ai?" — của tất cả, tức là của không ai); và nó **hút** thêm code mới theo quán tính — God Object chỉ to lên, không bao giờ tự nhỏ đi.

### Refactoring phù hợp

**Extract Class** (3.3), cắt theo **cụm dependency × cụm lý do thay đổi** — hai tín hiệu này thường trùng nhau và chỉ thẳng đường ranh:

```
OrderManager (47 methods, 8 deps)
    → order.Service      (Create/Update/Cancel — cần db, kafka)
    → pricing.Engine     (CalcDiscount/Coupon — cần db, redis)     ← đã gặp ở 2.5
    → invoice.Exporter   (PDF/Excel — cần s3, pdf)
    → notify.Service     (confirmation/reminder — cần mailer, sms)
    → orderreport.Query  (stats — cần db replica)
```

Chiến thuật an toàn: **tách dần từng cụm** — cụm export ít coupling nhất tách trước; `OrderManager` tạm giữ vai facade ủy quyền sang class mới (giữ caller cũ không vỡ), teo dần rồi xóa. Đúng nhịp "chuỗi bước nhỏ" của chương 3.0.

### Khi nào KHÔNG phải smell

Package `order` *gom nhiều file* quanh một domain là functional cohesion tốt — đừng nhầm "package nhiều code" với God Object; tiêu chí là *một struct/một điểm nghẽn* chứ không phải tổng số dòng. Và ở app nhỏ 2.000 dòng, một `Service` struct 15 method là hoàn toàn lành mạnh — God Object là bệnh của quy mô.

---

## C. Primitive Obsession

### Ví dụ

```go
// ❌ mọi thứ là string và int64
func Transfer(fromAccount string, toAccount string, amount int64, currency string) error
```

```go
// Và hóa đơn của nó, trả dần ở mọi call site:
Transfer(toAcc, fromAcc, amt, "VND")        // đảo ngược from/to — compile ngon, tiền đi nhầm chiều
Transfer(from, to, amtUSD, "VND")           // số tiền USD dán nhãn VND
if email != "" && strings.Contains(email, "@") { }   // validation rải rác, mỗi nơi một kiểu
```

### Nguyên nhân

Primitive luôn sẵn — `string` không cần khai báo type mới; và cảm giác "chỉ là cái email thôi, cần gì type riêng". Nhưng mỗi primitive trần là một **khái niệm nghiệp vụ chưa được đặt tên**: "email đã validate", "số tiền gắn liền currency", "ID của account (khác ID của order!)".

### Hậu quả

Compiler — công cụ kiểm lỗi mạnh nhất, miễn phí nhất — bị vô hiệu hóa: mọi string hoán đổi được cho nhau. Validation lặp và lệch nhau khắp nơi (DRY vi phạm ở mức tri thức). Nghiệp vụ ngầm không có chỗ sống: tiền VND không có phần thập phân, so sánh hai Money khác currency phải là lỗi — logic này rải trong if lẻ tẻ hoặc tệ hơn, trong đầu dev.

### Refactoring phù hợp

**Replace Primitive with Value Object** — đã gặp `Money` ở chương 1.5, giờ thành hệ thống:

```go
// ✅ mỗi khái niệm một type — compiler thành người gác nghiệp vụ
type AccountID string          // named type: AccountID và OrderID không gán lẫn được
type Email struct{ addr string }

func ParseEmail(s string) (Email, error) {  // validate MỘT LẦN ở biên
    if !emailRx.MatchString(s) {
        return Email{}, fmt.Errorf("invalid email %q", s)
    }
    return Email{addr: strings.ToLower(s)}, nil
}
// Email tồn tại ⟹ đã hợp lệ. Mọi hàm nhận Email khỏi validate lại.

func Transfer(from, to AccountID, amount Money) error   // currency sống trong Money
```

Nguyên tắc đi kèm: **Parse, don't validate** — biên hệ thống (HTTP handler, consumer) parse primitive thô thành value object; từ đó vào trong, type system bảo đảm tính hợp lệ, code lõi không còn phòng thủ lặt vặt. TypeScript không có named type thật (type alias là structural) — dùng branded type: `type AccountID = string & { readonly __brand: "AccountID" }` — cùng hiệu quả compile-time.

### Khi nào KHÔNG phải smell

Giá trị cục bộ, đi qua 1-2 hàm, không mang invariant: `limit int` của một query, `retries int` của một vòng lặp — bọc type là nghi lễ. Value object đáng giá khi khái niệm **xuyên nhiều tầng** hoặc **mang luật riêng**. Và trong Go, named type (`type AccountID string`) gần như miễn phí — với ID, ngưỡng dùng nên rất thấp.

---

## D. Data Clumps

### Ví dụ

```go
// ❌ bộ ba (street, district, city) đi tour khắp codebase
func ValidateAddress(street, district, city string) error
func ShippingFee(street, district, city string, weight float64) int64
func SaveCustomer(name, phone, street, district, city string) error
func FormatLabel(name, phone, street, district, city string) string
```

### Nguyên nhân & nhận diện

Vài mảnh dữ liệu **luôn xuất hiện cùng nhau** nhưng chưa ai đặt tên cho cụm — một khái niệm domain đang tồn tại *ẩn danh*. Phép thử của Fowler: *xóa một mảnh, các mảnh còn lại còn nghĩa không?* `district` đứng một mình vô nghĩa → chúng là một cụm. Họ hàng gần của Primitive Obsession (cụm primitive thay vì một primitive) và là nguyên nhân trực tiếp của Long Parameter List.

### Hậu quả

Chữ ký hàm dài và giòn — thêm `ward` (phường) vào địa chỉ = sửa 15 chữ ký + mọi call site (Shotgun Surgery); các tham số cùng type đứng cạnh nhau chờ bị truyền nhầm thứ tự; và logic của cụm (format địa chỉ, so sánh 2 địa chỉ) không có nhà, rải khắp nơi.

### Refactoring phù hợp

**Introduce Parameter Object** (3.3):

```go
// ✅ khái niệm ẩn danh được đặt tên — và lập tức thành nam châm hút hành vi về đúng nhà
type Address struct {
    Street, Ward, District, City string
}

func (a Address) Validate() error        { /* ... */ }
func (a Address) OneLine() string        { /* ... */ }
func (a Address) SameDistrict(b Address) bool { /* ... */ }

func ShippingFee(addr Address, weight float64) int64
func SaveCustomer(name, phone string, addr Address) error
```

Hiệu ứng dây chuyền đáng giá nhất: sau khi cụm có tên, các hàm thao tác trên cụm **tự tìm về** làm method — Feature Envy (3.2) quanh cụm đó tự tan. Thêm `ward` giờ là thay đổi một struct.

### Khi nào KHÔNG phải smell

Hai tham số đi cùng nhau đúng một lần — chưa đủ án; Rule of Three áp dụng ở đây. Và đừng gom những thứ *tình cờ* đi cùng trong một chữ ký nhưng không phải một khái niệm (`ctx` + `logger` không phải data clump — chúng là hai mối quan tâm độc lập).

---

## Tổng kết chương — bảng tra nhanh

| Smell | Câu hỏi chẩn đoán | Thuốc chính | Ngưỡng báo động |
|---|---|---|---|
| Long Method | Hàm này trộn mấy mức trừu tượng? | Extract Method | Comment "bước 1/2/3", biến sống cục bộ theo cụm |
| Large Class / God Object | Struct này có mấy *lý do thay đổi*, mấy cụm dependency? | Extract Class | Tên kết thúc bằng Manager/Util; mọi PR đều đụng nó |
| Primitive Obsession | String/int này mang khái niệm nghiệp vụ nào? | Replace Primitive with Value Object | Validation lặp; tham số cùng type kề nhau |
| Data Clumps | Cụm tham số này có phải một khái niệm ẩn danh? | Introduce Parameter Object | 3+ tham số luôn đi cùng nhau ở 3+ chữ ký |

---

*Tiếp theo: [3.2 — Code Smells nhóm coupling](/series/software-design/level-3-refactoring/02-smells-coupling/)*
