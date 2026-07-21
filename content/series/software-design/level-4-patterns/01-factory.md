+++
title = "4.1 — Factory Method & Abstract Factory: kiểm soát điểm tạo object"
date = "2026-07-17T10:10:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## 1. Problem Statement

Xâu chuỗi những gì đã có: chương 2.2 (OCP) đưa các phương thức thanh toán về interface `Gateway` + registry; chương 3.4 kết thúc Replace Conditional with Polymorphism bằng câu: *"nơi tạo object từ string trở thành điểm switch duy nhất còn lại — và nó có tên: Factory"*. Chương này trả lời nốt: điểm tạo object đó thiết kế thế nào cho tử tế — vì **`new` (hay `&Struct{}`) chính là dạng coupling cứng nhất**: nơi nào gọi `&MomoGateway{...}` là nơi đó dính chặt vào concrete type, biết cấu trúc field, chịu trách nhiệm truyền config.

Bài toán chạy xuyên chương: hệ thống export báo cáo — người dùng chọn định dạng (CSV, XLSX, PDF), mỗi định dạng cần khởi tạo khác nhau (PDF cần font, XLSX cần template, CSV cần delimiter theo locale).

## 2. Code Evolution — từ if-else đến Factory

**V1 — tạo thẳng tại chỗ dùng.** Đúng cho ngày đầu:

```go
func (h *Handler) Export(w http.ResponseWriter, r *http.Request) {
    var exp Exporter
    switch r.URL.Query().Get("format") {
    case "csv":
        exp = &CSVExporter{Delimiter: ','}
    case "xlsx":
        exp = &XLSXExporter{Template: h.tmplPath}
    case "pdf":
        exp = &PDFExporter{FontDir: h.fontDir, Landscape: true}
    default:
        http.Error(w, "unsupported format", 400); return
    }
    exp.Write(w, h.loadReport(r.Context()))
}
```

**Yêu cầu đổi**: export còn được gọi từ cron job gửi mail hằng tuần và từ CLI admin — khối switch trên bị **copy ra ba nơi** (Duplicate Code cấp 2). Rồi PDF thêm tham số watermark: sửa 3 chỗ, quên 1 (Shotgun Surgery). Chi tiết khởi tạo (font, template, delimiter) rò vào handler, cron, CLI — ba nơi chẳng liên quan gì đến font chữ.

**V2 — Factory function: gom tri thức khởi tạo về một nhà.**

```go
// package export — MỘT nơi biết cách tạo mọi exporter
func New(format string, cfg Config) (Exporter, error) {
    switch format {
    case "csv":
        return &CSVExporter{Delimiter: cfg.CSVDelimiter}, nil
    case "xlsx":
        return &XLSXExporter{Template: cfg.TemplatePath}, nil
    case "pdf":
        return &PDFExporter{FontDir: cfg.FontDir, Landscape: cfg.Landscape}, nil
    default:
        return nil, fmt.Errorf("export: unsupported format %q", format)
    }
}
```

Đây là **factory function** — dạng factory phổ biến nhất và thường là *đủ*. Caller giờ chỉ biết interface `Exporter` và một cái tên; mọi tri thức khởi tạo có một nhà. Lưu ý một quyết định nhỏ mà quan trọng: factory trả `(Exporter, error)` — lỗi "format lạ" nổi lên tường minh tại một chỗ, thay vì rơi vào default ngẫu nhiên ở ba chỗ.

**V3 — Registry factory: khi danh sách loại phải mở.** Nếu (và chỉ nếu) yêu cầu "thêm format mới không sửa package export" là có thật — ví dụ format do plugin/team khác cung cấp:

```go
var registry = map[string]func(cfg Config) Exporter{}

func Register(name string, ctor func(cfg Config) Exporter) {
    if _, dup := registry[name]; dup {
        panic("export: duplicate format " + name)   // fail sớm, lúc khởi động
    }
    registry[name] = ctor
}

func New(format string, cfg Config) (Exporter, error) {
    ctor, ok := registry[format]
    if !ok {
        return nil, fmt.Errorf("export: unsupported format %q (registered: %v)", format, names())
    }
    return ctor(cfg), nil
}
```

Đây chính xác là kiến trúc `database/sql` + driver (2.2). Trade-off đã phân tích ở đó vẫn nguyên: danh sách format giờ là trạng thái runtime — trả lời "hệ thống hỗ trợ gì?" phải hỏi lúc chạy, không đọc được từ code tĩnh.

## 3. Factory Method — dạng GoF nguyên bản, và vì sao Go ít cần nó

Cẩn thận thuật ngữ: cái vừa xây là **factory function** (creation function). **Factory Method** theo GoF là thứ khác: *lớp cha định nghĩa khung thuật toán, chừa một method ảo "tạo object" cho lớp con quyết định concrete type* — tức là Template Method áp vào việc khởi tạo, sống bằng inheritance:

```typescript
// GoF Factory Method — hình dạng nguyên bản (TS, vì cần inheritance)
abstract class ReportJob {
  run(data: Report) {                    // khung chung: lớp cha sở hữu
    const exp = this.createExporter();   // ← factory method: lớp con quyết định
    const bytes = exp.write(data);
    this.upload(bytes);
  }
  protected abstract createExporter(): Exporter;
}
class PdfReportJob extends ReportJob {
  protected createExporter() { return new PdfExporter(this.fontDir); }
}
```

Go không có inheritance → dạng này dịch thành **truyền factory function như một dependency** — cấu trúc phẳng hơn hẳn mà cùng công dụng:

```go
// Go: "factory method" = một field kiểu function — không cần cây lớp nào
type ReportJob struct {
    newExporter func() Exporter      // điểm biến thiên, tiêm từ ngoài
    uploader    Uploader
}

func (j *ReportJob) Run(data Report) error {
    exp := j.newExporter()
    /* khung chung */
}

// Wiring: ReportJob{newExporter: func() Exporter { return NewPDF(fontDir) }, ...}
```

Bài học tổng quát đáng giá hơn bản thân pattern: **trong ngôn ngữ có first-class function, "chừa một bước cho kẻ khác quyết" = nhận một function**. Đây là số phận "tan vào ngôn ngữ" (4.0) — bài toán còn, nghi thức lớp lang biến mất. Khi nào cần *nhiều* bước tạo liên quan nhau thì function đơn lẻ hết tải — đó là lúc Abstract Factory vào sân.

## 4. Abstract Factory — họ object phải khớp bộ

**Bài toán riêng của nó** (đừng nhầm với factory thường): cần tạo **một họ object liên quan, bắt buộc tương thích nhau**. Ví dụ thật trong backend: hỗ trợ chạy trên hai stack hạ tầng — AWS (SQS + S3 + SES) và on-prem (RabbitMQ + MinIO + SMTP). Queue của AWS đi với storage của on-prem là cấu hình vô nghĩa; ba lựa chọn phải **khớp bộ**.

```go
// Abstract Factory = interface trả về CẢ HỌ — tính khớp bộ được thi hành bằng cấu trúc
type InfraFactory interface {
    Queue() (Queue, error)
    BlobStore() (BlobStore, error)
    Mailer() (Mailer, error)
}

type AWSFactory struct{ cfg AWSConfig }
func (f *AWSFactory) Queue() (Queue, error)         { return newSQS(f.cfg) }
func (f *AWSFactory) BlobStore() (BlobStore, error) { return newS3(f.cfg) }
func (f *AWSFactory) Mailer() (Mailer, error)       { return newSES(f.cfg) }

type OnPremFactory struct{ cfg OnPremConfig }
/* ... RabbitMQ + MinIO + SMTP ... */

// main: chọn HỌ một lần — từ đó không thể lệch bộ
var infra InfraFactory
switch cfg.Env {
case "aws":    infra = &AWSFactory{cfg: cfg.AWS}
case "onprem": infra = &OnPremFactory{cfg: cfg.OnPrem}
}
```

Giá trị cốt lõi: **ràng buộc "phải cùng họ" chuyển từ kỷ luật con người sang type system** — không tồn tại đường code nào lấy được SQS kèm MinIO. Nhận diện đúng bài toán bằng câu hỏi: *các object này có RÀNG BUỘC tương thích lẫn nhau không?* Không có ràng buộc → chỉ cần N factory function độc lập, đừng gom vào interface họ hàng cho có dáng pattern.

**Trade-off đặc thù**: thêm một loại object vào họ (thêm `Cache()`) = sửa interface + mọi factory — Abstract Factory *đóng* theo chiều thao tác y như mọi interface rộng (ISP đã cảnh báo). Họ càng đông, interface càng cứng. Trong thực tế backend, họ 3-5 thành viên là ngưỡng thoải mái; hơn nữa nên xét tách bộ hoặc quay về DI container wiring theo profile.

## 5. Trade-off tổng — thang mức độ, chọn thấp nhất đủ dùng

```
&Struct{...} tại chỗ      → mặc định. Một nơi dùng, concrete type, không có gì phải giấu
NewX(...) constructor     → khi khởi tạo có invariant cần bảo vệ (1.2) hoặc field private
Factory function (V2)     → khi ≥2 nơi cùng cần tạo-theo-tên, hoặc tri thức khởi tạo cần một nhà
Registry (V3)             → khi bên ngoài package phải thêm loại mới (plugin, driver)
Abstract Factory          → khi một HỌ object phải khớp bộ
```

Mỗi bậc thêm một tầng gián tiếp và một khái niệm phải học. Kỷ luật của kỹ sư giỏi là **leo đúng một bậc khi có bằng chứng, không leo trước hai bậc cho "chắc"**.

## 6. Production sightings

- **`database/sql`**: registry factory chuẩn công nghiệp (đã mổ ở 2.2). Chi tiết đáng học thêm: driver đăng ký trong `init()` qua blank import `_ "github.com/lib/pq"` — tiện, nhưng chính tác giả Go về sau xem `init()` + side-effect import là thiết kế đáng tiếc (magic, khó test, import là chạy code). Bài học: registry tường minh (gọi `Register` từ `main`) tốt hơn registry tự động khi bạn có quyền chọn.
- **`http.NewRequest` / `net.Dial`**: factory function trả interface hoặc concrete tùy mức trừu tượng cần thiết — minh họa "accept interfaces, return structs" (1.5) áp vào factory: `net.Dial` trả `net.Conn` (interface — nhiều transport), `http.NewRequest` trả `*http.Request` (concrete — chỉ có một).
- **AWS SDK v2 (Go)**: `config.LoadDefaultConfig` + `s3.NewFromConfig(cfg)` — một "config factory" trung tâm sinh client các service: họ object khớp bộ (credentials, region, retry policy dùng chung) — tinh thần Abstract Factory dù không mang tên đó.
- **NestJS `useFactory`**: DI container cho phép provider là factory function có inject dependency — factory trở thành *công dân của container*; cùng bài toán, giải trong khuôn khổ framework.
- **`testing` ecosystem**: hàm `newTestServer(t)`, `newFakeStore(t)` trong test — factory function khiêm tốn nhất và có ROI cao nhất: gom wiring test về một chỗ, mỗi test đọc được bằng một dòng.

## 7. Anti-pattern

- **God Factory**: một `factory.go` 800 dòng tạo *mọi thứ* trong app — factory cũng phải theo SRP; mỗi package tự lo factory của loại nó sở hữu. Nếu bạn có package tên `factories/` — đó là logical cohesion (1.3) nguyên chất.
- **Factory chồng factory**: `ExporterFactoryProvider.GetFactory().Create("pdf")` — mỗi tầng chỉ forward. Bệnh Java enterprise kinh điển; trong Go/TS một hàm `New` là đủ cho tuyệt đại đa số.
- **Interface một implementation + factory cho nó**: bộ đôi nghi lễ (đã điểm mặt ở 1.5) — factory chỉ đáng tồn tại khi *có gì đó để quyết định* (chọn loại, giấu khởi tạo phức tạp, bảo vệ invariant).
- **Factory giấu dependency**: `NewService()` mà bên trong tự mở DB, đọc env var, tạo HTTP client — trông gọn nhưng chôn coupling xuống đất (1.3 V1). Factory tử tế **nhận** dependency qua tham số và chỉ *lắp ráp*.

## 8. Khi nào KHÔNG dùng

Một nơi tạo, một loại, khởi tạo 1 dòng → `&Struct{}` trần là thiết kế đúng. App nhỏ config tĩnh → switch trong `main` đủ sạch. Và đừng quên lựa chọn rẻ nhất hay bị bỏ qua: **truyền sẵn object đã tạo** — nhiều chỗ "cần factory" thực ra chỉ cần caller tạo một lần rồi truyền xuống (DI thủ công, 2.5); factory chỉ cần thiết khi việc tạo phải xảy ra *muộn* (lazy, per-request, theo dữ liệu runtime).

---

*Tiếp theo: [4.2 — Builder & Functional Options](/series/software-design/level-4-patterns/02-builder-functional-options/)*
