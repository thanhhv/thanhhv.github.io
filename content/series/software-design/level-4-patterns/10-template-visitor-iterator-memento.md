+++
title = "4.10 — Template Method, Visitor, Iterator & Memento: bốn số phận thời hậu GoF"
date = "2026-07-17T11:40:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

Bốn pattern behavioral còn lại, điểm chung: **ngôn ngữ hiện đại đã thay đổi số phận của chúng mạnh nhất** — hai cái tan gần hết vào ngôn ngữ (Template Method, Iterator), một cái thu hẹp về ngách chuyên môn (Visitor), một cái sống dưới các tên khác (Memento). Chương này ngắn có chủ đích, ưu tiên "nhận ra và chọn đúng thay thế" hơn là "cài đặt nguyên bản".

---

## A. Template Method — khung cố định, chừa bước biến thiên

### Bài toán & dạng GoF

Nhiều quy trình giống nhau 80%: import file (mở → parse *tùy định dạng* → validate → ghi DB → báo cáo), xử lý consumer (nhận → decode *tùy topic* → xử lý → ack). GoF giải bằng inheritance: lớp cha sở hữu khung `Process()`, chừa các bước abstract cho con override — chính là `ReportJob` ở 4.1, và là xương sống của framework Java/C# một thời (`AbstractController`, `HttpServlet.doGet`...).

Bệnh án của dạng inheritance đã mổ đủ ở 1.4: con-cha coupling mạnh nhất, fragile base class, con override để *tắt* bước — không nhắc lại. Điều cần nói là **Go không thể viết dạng này kể cả muốn** (không có override ảo — method của struct cha không bao giờ gọi được "phiên bản của con", 1.4 phần embedding), nên câu hỏi thực tế là: thay bằng gì?

### Hai thay thế, chọn theo số bước biến thiên

```go
// (1) MỘT bước biến thiên → truyền function: Template Method = hàm nhận hàm
func RunImport(ctx context.Context, src io.Reader, parse func([]byte) ([]Row, error)) (Report, error) {
    /* khung: đọc → gọi parse → validate → ghi → báo cáo */
}
// chính là mẫu sort.Slice, http.HandlerFunc — "chừa một bước" = "nhận một hàm" (bài học 4.1)

// (2) NHIỀU bước biến thiên liên quan nhau → nhận interface (nhỏ!):
type ImportFormat interface {
    Parse([]byte) ([]Row, error)
    Validate(Row) error
}
func RunImport(ctx context.Context, src io.Reader, f ImportFormat) (Report, error)
```

Cả hai đảo quan hệ so với GoF: biến thiên được **tiêm vào** khung (composition) thay vì khung bị **kế thừa bởi** biến thiên — mọi lợi ích của 1.4 theo về. Nhận diện nhanh trong code Go/TS hiện đại: hễ thấy "hàm khung nhận callback/interface cho phần khác biệt" — đó là Template Method đang sống dưới dạng đã tiến hóa, gồm cả `testing.T.Run(name, func)` và table-driven test bạn viết mỗi ngày.

---

## B. Iterator — pattern thắng cuộc tuyệt đối: nó thành cú pháp

### Bài toán GoF và số phận

Duyệt collection mà không phơi cấu trúc trong (mảng? cây? trang từ API?) — GoF cần interface `Iterator{next, hasNext}`. Số phận: **thắng đến mức biến mất** — `for range` (Go), `for...of` + generator (TS), mọi ngôn ngữ chính thống đều nuốt nó vào cú pháp.

Phần đáng học còn lại cho backend engineer nằm ở hai chỗ:

**(1) Go 1.23 range-over-func — Iterator chính thức có hợp đồng chuẩn:**

```go
// iter.Seq2: MỌI nguồn dữ liệu giờ duyệt được bằng for range — kể cả nguồn phân trang, lazy
func (r *Repo) AllOrders(ctx context.Context) iter.Seq2[Order, error] {
    return func(yield func(Order, error) bool) {
        for page := 0; ; page++ {
            batch, err := r.fetchPage(ctx, page)         // phân trang GIẤU TRONG iterator
            if err != nil { yield(Order{}, err); return }
            for _, o := range batch {
                if !yield(o, nil) { return }             // caller break → dừng fetch sớm
            }
            if len(batch) < pageSize { return }
        }
    }
}

// Caller: mọi phức tạp phân trang vô hình — và KHÔNG load hết vào RAM
for o, err := range repo.AllOrders(ctx) { /* ... */ }
```

Đây là điều Iterator luôn hứa: **tách cách-duyệt khỏi việc-làm-gì-mỗi-phần-tử** — giờ có cú pháp chính chủ. Trước 1.23, cùng nhu cầu giải bằng channel (tốn goroutine, khó dọn khi break sớm) hoặc mẫu `rows.Next()` thủ công.

**(2) `sql.Rows`/`bufio.Scanner` — dạng iterator thủ công vẫn phải hiểu**: `for rows.Next() { rows.Scan(...) }; rows.Err()` — hợp đồng ba bước (tiến → đọc → *kiểm lỗi sau vòng lặp*) mà quên bước ba là bug nuốt lỗi kinh điển. Iterator che cấu trúc nhưng **không che được lỗi I/O** — mọi thiết kế iterator trên nguồn bất định phải trả lời "lỗi chảy ra đường nào?" (đó là lý do `iter.Seq2[T, error]` tồn tại).

TS đối chiếu: generator `function*` + `for await...of` cho async iterator — Node stream hiện đại (`for await (const chunk of stream)`) chính là chỗ dev backend TS dùng nó thật.

---

## C. Visitor — pattern gây tranh cãi nhất, thu về đúng ngách của nó

### Bài toán thật — và cái giá GoF

Cấu trúc dữ liệu **ổn định** (AST với 30 loại node), thao tác trên nó **mở** (type-check, format, optimize, lint... — mỗi quý thêm vài cái). Đây là mặt B của expression problem (2.2, 3.4): interface thường mở theo *loại* đóng theo *thao tác*; Visitor **đảo lại** — thêm thao tác = một visitor mới, không đụng 30 node; đổi lại thêm *loại node* = sửa mọi visitor.

Cơ chế GoF (double dispatch — node `Accept(v)` gọi ngược `v.VisitX(node)`) tồn tại vì C++/Java thiếu cách rẽ nhánh theo runtime type tử tế. Go có **type switch**, nên dạng nhẹ trước, nghi thức sau:

```go
// Dạng nhẹ — type switch: đủ cho đa số nhu cầu "một thao tác mới trên cây type"
func fmtNode(n ast.Node) string {
    switch n := n.(type) {
    case *ast.BinaryExpr: /* ... */
    case *ast.CallExpr:   /* ... */
    /* ... */
    }
}
```

Yếu điểm thật của type switch — **quên case im lặng** (thêm node mới, 12 switch rải rác, sót 3 cái không ai báo) — chính là thứ Visitor nghi thức sửa: thêm `VisitNewNode` vào interface → compiler chỉ mặt mọi visitor chưa implement. Vậy ngưỡng chọn: **ít thao tác, nguồn switch tập trung → type switch; nhiều thao tác sống dài trên cấu trúc lớn ổn định → Visitor** (hoặc TS: discriminated union + exhaustiveness check — compiler bắt sót case ngay trong switch, ăn cả hai đầu; Go không có tương đương, chỉ có linter).

Production đúng ngách: `go/ast.Walk` + `astutil.Apply` (toàn bộ tooling Go — linter, gofmt, gopls — sống trên đó), TS Compiler API visitor, ANTLR, terraform plan walker. Ngoài ngách compiler/AST/tài-liệu-có-cấu-trúc — gặp Visitor trong code nghiệp vụ thường là over-engineering: CRUD không có 30 loại node và 10 thao tác mở.

---

## D. Memento — sống khỏe dưới những cái tên khác

### Bài toán GoF và bản dịch hiện đại

Chụp trạng thái object để khôi phục sau này, **không phá encapsulation** (bên giữ snapshot không đọc được ruột). Nghe xa lạ cho đến khi điểm danh nơi nó đang sống:

- **Transaction savepoint / `defer`-rollback**: chụp trước khi thử, hỏng thì về — Memento của DB.
- **Undo qua snapshot** (thay vì inverse command — 4.9): editor giữ stack các bản chụp immutable.
- **Event sourcing snapshot** (Level 5): stream sự kiện dài → chụp aggregate mỗi N event để replay nhanh — Memento ở quy mô kiến trúc.
- **React/Redux time-travel debug**: mỗi action một state snapshot — immutable state (1.5) làm Memento thành *miễn phí*: chỉ cần giữ con trỏ tới bản cũ.

Dòng cuối là bài học lớn nhất: **chi phí của Memento tỉ lệ nghịch với độ immutable của thiết kế**. State mutable → chụp = deep copy đắt và dễ sai (bẫy nông/sâu, 4.3 Prototype — hai pattern này là anh em: Prototype chụp để *tạo mới*, Memento chụp để *quay về*). State immutable → chụp = giữ một tham chiếu. Nếu bạn thấy mình viết code snapshot/restore phức tạp, câu hỏi đầu tiên không phải "cài Memento sao cho đúng" mà là *"thiết kế này immutable hơn được không?"* (1.5).

Trong Go backend hằng ngày, dạng gặp nhiều nhất khiêm tốn hơn nhiều: copy struct trước khi thử thay đổi có-thể-fail, hoặc `t.Setenv`/`t.Cleanup` trong test — chụp và tự động khôi phục môi trường. Không ai gọi đó là Memento; không cần — nhận ra bài toán "chụp-để-quay-về" và chọn snapshot vs inverse vs transaction là toàn bộ giá trị còn lại của pattern.

---

## Tổng kết nhóm Behavioral

| Pattern | Số phận 2026 | Dạng nên viết trong Go/TS |
|---|---|---|
| Strategy | Tan vào ngôn ngữ | function value → interface nhỏ → registry (thang 4.7) |
| State | Nguyên giá trị, đúng bài | bảng chuyển trạng thái → State type khi hành vi phân hóa (4.7) |
| Observer | Nguyên giá trị, đổi hình | slice observer → event bus → broker khi cần bền (4.8) |
| Command | Nguyên giá trị, đổi hình | closure trong process; struct + registry khi cần bền/xa (4.9) |
| Chain of Resp. | Sống qua biến thể | middleware (4.5); dạng thuần cho fallback/escalation (4.9) |
| Mediator | Nhãn chẩn đoán | use case/orchestrator; cảnh giác God Object (4.9) |
| Template Method | Tan vào ngôn ngữ | hàm-nhận-hàm hoặc khung-nhận-interface-nhỏ (4.10) |
| Iterator | Thành cú pháp | `for range` / `iter.Seq` / generator; nhớ hợp đồng lỗi (4.10) |
| Visitor | Thu về ngách | type switch trước; Visitor cho AST/tooling (4.10) |
| Memento | Sống dưới tên khác | snapshot + immutability; savepoint; event sourcing (4.10) |

Nhóm Behavioral xác nhận trọn vẹn luận điểm 4.0: pattern không chết — **bài toán bất tử, nghi thức tiến hóa theo ngôn ngữ**. Ai học thuộc cấu trúc 1994 mà không học bài toán sẽ viết code Java bằng cú pháp Go; ai học bài toán sẽ nhận ra mình đã dùng cả mười pattern này từ trước khi biết tên chúng.

---

*Tiếp theo: [4.11 — Dependency Injection & DI Container](/series/software-design/level-4-patterns/11-dependency-injection/) — mở màn phần Modern Design.*
