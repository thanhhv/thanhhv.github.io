+++
title = "4.6 — Composite, Bridge & Flyweight: ba pattern cấu trúc còn lại"
date = "2026-07-17T11:00:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

Ba pattern khép lại nhóm Structural, mỗi cái một số phận khác nhau trong backend hiện đại: Composite sống khỏe ở một ngách rõ ràng (cấu trúc cây); Bridge là *ý tưởng* quan trọng hơn *hình thức*; Flyweight chủ yếu đã chuyển hộ khẩu xuống tầng runtime và thư viện.

---

## Phần A — Composite

### 1. Bài toán — khi phần tử và nhóm phần tử phải cư xử như nhau

Bài toán thật: hệ thống phân quyền theo nhóm. Ban đầu chỉ có user:

```go
func CanAccess(u User, resource string) bool { /* ... */ }
```

Rồi yêu cầu: quyền cấp cho **nhóm** — và nhóm chứa user *lẫn nhóm con* (phòng Kỹ thuật chứa team Backend, team Backend chứa user). Code không có pattern sẽ mọc if phân loại ở mọi nơi duyệt:

```go
// ❌ mọi hàm đụng tới thành viên đều phải biết phân biệt user/group và tự đệ quy
func collectEmails(members []any) []string {
    for _, m := range members {
        switch v := m.(type) {
        case User:  /* lấy email */
        case Group: /* đệ quy... lại switch tiếp ở tầng dưới */
        }
    }
}
```

Cấu trúc dữ liệu là **cây**, nhưng code xử lý lại viết theo kiểu "danh sách phẳng + phân loại tại chỗ" — mỗi thao tác mới (đếm thành viên, kiểm tra quyền, gửi thông báo) lại copy khung switch-đệ-quy đó.

### 2. Pattern — lá và cành chung một interface

```go
// ✅ Composite: MỘT interface cho cả lá (User) lẫn cành (Group)
type Principal interface {
    Contains(userID string) bool
}

type User struct{ ID string }
func (u User) Contains(id string) bool { return u.ID == id }

type Group struct {
    Name    string
    Members []Principal          // ← đệ quy nằm TRONG CẤU TRÚC, không nằm trong caller
}
func (g Group) Contains(id string) bool {
    for _, m := range g.Members {
        if m.Contains(id) {      // lá hay cành — không cần biết, không cần switch
            return true
        }
    }
    return false
}
```

Caller giờ cầm một `Principal` — một user đơn lẻ hay cả cây tổ chức nghìn người — và gọi `Contains` y như nhau. **Tính chất định nghĩa của Composite: client không phân biệt một-phần-tử với cả-cây.** Mọi thao tác mới chỉ thêm method vào interface (với trade-off quen thuộc của chiều mở rộng đó — expression problem, 2.2/3.4).

### 3. Chỗ dùng thật — nhiều hơn bạn nghĩ, nhưng đều hình cây

- **`fs.FS` / cây thư mục**: file và directory — Composite nguyên bản của mọi hệ điều hành.
- **Cấu trúc rule nghiệp vụ**: điều kiện khuyến mãi dạng `AND(minTotal(500k), OR(isVIP, firstOrder))` — mỗi node lá là một predicate, node cành là AND/OR/NOT cũng *là* predicate. Specification pattern (phần Modern Design) chính là Composite áp vào rule — đây là chỗ dev backend gặp nó nhiều nhất.
- **`errors.Join` (Go 1.20)**: nhiều error gộp thành một error — thứ trả về vẫn là `error`, và `errors.Is/As` tự duyệt cây. Composite chui vào stdlib error handling mà ít ai gọi tên.
- **UI tree (React), AST, org chart, BOM (bill of materials)** — hễ dữ liệu là cây và thao tác cần "hỏi cả cây như hỏi một node".

Điều kiện dùng rõ ràng đến mức ít khi lạm dụng được: **dữ liệu phải thật sự đệ quy** (nhóm chứa nhóm). Danh sách phẳng một cấp thì slice + loop là đủ — đừng dựng cây cho thứ không có cành. Cạm bẫy đáng nhớ duy nhất: **chu trình** — nhóm A chứa nhóm B chứa nhóm A → `Contains` đệ quy vô hạn. Cây thật phải được *giữ* là cây (chặn chu trình lúc thêm member, hoặc đánh dấu visited lúc duyệt) — dữ liệu từ DB quan hệ rất giỏi lén tạo chu trình.

---

## Phần B — Bridge

### 1. Bài toán — hai chiều biến thiên độc lập nhân nhau

Bài toán kinh điển hóa bằng ví dụ thật: hệ thống thông báo có N *loại nội dung* (cảnh báo bảo mật, khuyến mãi, giao dịch) × M *kênh gửi* (email, SMS, push). Giải bằng kế thừa/type riêng cho từng tổ hợp:

```
SecurityAlertEmail, SecurityAlertSMS, SecurityAlertPush,
PromoEmail, PromoSMS, PromoPush, ...        ← N × M class, thêm 1 kênh = thêm N class
```

Bùng nổ tổ hợp — đúng căn bệnh của cây kế thừa ở 1.4, phiên bản hai chiều.

### 2. Pattern — tách hai chiều thành hai hierarchy nối bằng composition

```go
// Chiều 1: KÊNH GỬI (implementation side)
type Channel interface {
    Deliver(ctx context.Context, to Recipient, title, body string) error
}
// EmailChannel, SMSChannel, PushChannel — M type

// Chiều 2: LOẠI THÔNG BÁO (abstraction side) — CHỨA một Channel, không kế thừa gì
type SecurityAlert struct {
    channel Channel              // ← cây cầu
    /* dữ liệu riêng của alert */
}
func (a SecurityAlert) Notify(ctx context.Context, to Recipient) error {
    title, body := a.render()    // logic riêng chiều loại: format khẩn cấp, kèm hướng dẫn
    return a.channel.Deliver(ctx, to, title, body)
}
// PromoNotice, TxnReceipt — N type

// N + M type, tổ hợp lúc runtime: SecurityAlert{channel: SMSChannel{}}
```

N × M sụp xuống N + M. Và nhìn lại thì... **đây chính là composition + interface mà chương 1.4 đã dạy** — đúng vậy, và đó là điểm cần nói thẳng về Bridge: trong thế giới GoF/C++, nó là pattern "to" vì phải *chủ động phá* thói quen dồn mọi biến thiên vào một cây kế thừa. Trong Go — ngôn ngữ không có cây kế thừa — **Bridge không phải một kỹ thuật đặc biệt mà là hệ quả tự nhiên của "mỗi chiều biến thiên một interface, nối bằng field"**. Bạn đã viết Bridge cả trăm lần: struct nghiệp vụ chứa interface hạ tầng (2.5) chính là nó.

Giá trị còn lại của cái tên: **công cụ chẩn đoán**. Khi thấy tên type bắt đầu ghép đôi khái niệm (`PdfInvoiceExporter`, `CsvReportExporter`, `PdfReportExporter`...) — đó là tín hiệu hai chiều biến thiên đang bện vào nhau; câu hỏi "hai chiều này tách được không?" chính là câu hỏi Bridge. Trade-off khi tách: hai interface phải gặp nhau qua một hợp đồng trung gian (`Deliver(to, title, body)`) — hợp đồng đó là điểm nghẽn thiết kế: quá hẹp thì chiều loại bị bó (push cần deep-link, hợp đồng không chở được), quá rộng thì hai chiều lại dính nhau. Định hình hợp đồng giữa là 80% độ khó thật của Bridge.

### 3. Production sighting

`database/sql` lần thứ tư, và là góc nhìn đúng tên nhất: `sql.DB` (API cho app — chiều abstraction, phát triển thêm `QueryContext`, transaction, pool tuning) tách khỏi `driver.Driver` (chiều implementation — Postgres/MySQL tiến hóa độc lập), nối bằng hợp đồng `driver` nhỏ. Hai hierarchy, hai nhịp tiến hóa, một cây cầu — Bridge công nghiệp. Tương tự: `slog.Logger` (front-end API) ↔ `slog.Handler` (backend: JSON, text, OTLP...) — thiết kế logging của Go 1.21 là Bridge giáo khoa, và họ chọn nó *chính vì* hai chiều này đổi vì hai lý do khác nhau (API tiện dụng vs định dạng đầu ra).

---

## Phần C — Flyweight

### 1. Bài toán — triệu object giống nhau đến 90%

GoF: soạn thảo văn bản, mỗi ký tự một object (font, size, glyph) — triệu ký tự = triệu object trong khi chỉ có vài chục *tổ hợp thuộc tính* thật sự khác nhau. Giải: tách **intrinsic state** (phần dùng chung, bất biến — glyph, font metrics) đưa vào object chia sẻ, giữ **extrinsic state** (phần riêng từng vị trí — tọa độ) bên ngoài; một pool/factory bảo đảm mỗi tổ hợp intrinsic chỉ có một instance.

Điều kiện tiên quyết ghi thẳng trong GoF và hay bị quên: **phần chia sẻ phải immutable** (1.5) — chia sẻ mutable state giữa triệu chỗ dùng là công thức thảm họa, không phải pattern.

### 2. Số phận hiện đại — chuyển hộ khẩu xuống runtime và thư viện

Dev backend 2026 hầu như không *viết* Flyweight — nhưng *dùng* nó cả ngày:

- **String interning**: JVM/V8/CPython intern chuỗi; Go compiler chia sẻ backing array cho string literal trùng nhau. Go 1.23 thêm `unique.Make` — interning chính chủ cho type bất kỳ: canonical hóa giá trị trùng lặp (triệu label metric, tag, tenant ID trùng nhau) thành handle chia sẻ — so sánh rẻ, bộ nhớ giảm hàng chục lần. Đây là Flyweight chính danh trong stdlib hiện đại.
- **`sync.Pool`**: họ hàng gần (chia sẻ để giảm allocation — nhưng là *tái sử dụng luân phiên* object mutable, không phải chia sẻ đồng thời bản immutable; đừng đánh đồng).
- **Connection pool, prepared statement cache**: tinh thần flyweight — tài nguyên đắt, dùng chung có kiểm soát.
- **Ca tự viết còn lại** đúng nghĩa: dịch vụ multi-tenant load *policy/config object* lớn giống nhau cho nghìn tenant — cache canonical instance theo hash nội dung, tenant giữ tham chiếu + phần override riêng. Đo trước khi làm: pprof heap cho thấy trùng lặp thật sự đáng kể thì mới đáng — flyweight là **tối ưu hóa có đo đạc**, không phải cấu trúc mặc định.

### 3. Một câu kết luận

Flyweight là pattern duy nhất trong GoF mà mục đích là **bộ nhớ**, không phải **chi phí thay đổi** — vì thế quy trình dùng nó cũng khác mọi pattern trong tài liệu này: *profile trước, pattern sau*, và immutability là điều kiện vào cửa.

---

## Tổng kết nhóm Structural

Bảy pattern, xếp theo tần suất nên gặp trong backend Go/TS:

```
Hằng ngày:      Decorator/Middleware (4.5), Facade (4.4), Adapter (4.4)
Theo bài toán:  Composite — dữ liệu cây (4.6), Proxy — cache/authz/remote (4.5)
Là hệ quả:      Bridge — tự xuất hiện khi tách chiều biến thiên bằng interface (4.6)
Khi profiler bảo: Flyweight (4.6)
```

Sợi chỉ đỏ của cả nhóm: **tất cả đều là composition qua interface nhỏ** — bảy cách trả lời khác nhau cho câu hỏi "đặt một type đứng cạnh/trước/quanh type khác để làm gì?". Nhóm Behavioral tiếp theo đổi câu hỏi: không còn là *cấu trúc tĩnh* mà là *luồng quyết định lúc chạy* — ai quyết định việc gì, và làm sao đổi quyết định mà không đổi khung.

---

*Tiếp theo: Level 4 phần Behavioral — Strategy & State, Observer, Command & Chain of Responsibility... (phần tiếp theo của bộ tài liệu).*
