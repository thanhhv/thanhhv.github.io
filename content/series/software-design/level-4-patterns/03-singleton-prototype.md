+++
title = "4.3 — Singleton & Prototype: hai pattern thu hẹp đất sống"
date = "2026-07-17T10:30:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

Hai pattern Creational còn lại thuộc nhóm số phận thứ ba của bản đồ 4.0: bài toán gốc của chúng hoặc đã có lời giải rẻ hơn, hoặc hiếm hơn xưa nhiều. Chương này ngắn hơn các chương khác **một cách chủ đích** — độ dài tỉ lệ với tần suất bạn nên dùng chúng.

---

## Phần A — Singleton

### 1. Bài toán gốc — và cú trượt dài nửa thế kỷ

Bài toán chính đáng: một tài nguyên **về bản chất chỉ nên tồn tại một** trong process — connection pool, logger gốc, registry metric, config đã load. Tạo nhiều bản là lãng phí (N connection pool) hoặc sai (hai registry metric đếm lệch nhau).

GoF giải bằng: constructor private + method static `getInstance()` trả về instance duy nhất, khởi tạo lười. Và trong 20 năm sau đó, nó trở thành **pattern bị lạm dụng nhất lịch sử** — vì nó quá *tiện*: gọi được từ bất cứ đâu, khỏi truyền tham số, khỏi nghĩ wiring. Cái tiện đó chính là cái bẫy.

### 2. Vì sao Singleton cổ điển là global variable đội mũ

Nhìn bằng ngôn ngữ Level 1, `getInstance()` gọi từ mọi nơi = **common coupling** (1.3) nguyên chất, kèm đủ hồ sơ bệnh án:

```go
// ❌ Singleton kiểu cổ điển trong Go
var instance *Database
var once sync.Once

func GetDB() *Database {
    once.Do(func() {
        instance = connect(os.Getenv("DSN"))   // đọc env sâu trong ruột — config tàng hình
    })
    return instance
}

// và ở 47 file khác nhau:
func ChargeOrder(o *Order) error {
    db := GetDB()          // dependency tàng hình: chữ ký hàm không hé lộ nó cần DB
    /* ... */
}
```

- **Dependency tàng hình**: đọc chữ ký `ChargeOrder(o *Order) error` không thể biết nó chạm DB — phải đọc thân hàm, đệ quy xuống mọi hàm nó gọi. Chi phí hiểu (1.1) tăng khắp codebase.
- **Test địa ngục**: muốn test `ChargeOrder` với DB giả? Phải chen vào `instance` — thêm hàm `SetDBForTest()` (cửa hậu), và mọi test *dùng chung* trạng thái đó → test không chạy song song được, thứ tự test ảnh hưởng kết quả — flaky test có địa chỉ nhà.
- **Thứ tự khởi tạo ngầm**: ai gọi `GetDB()` đầu tiên quyết định *lúc nào* connection mở, với env var *lúc đó* — bug chỉ hiện khi đổi thứ tự chạy.
- **Giấu một quyết định thiết kế**: "chỉ có một DB" hôm nay; ngày mai cần read-replica, multi-tenant, hai DSN — mà cả codebase đã đóng đinh vào `GetDB()` không tham số. Singleton biến giả định nhất thời thành hợp đồng vĩnh viễn.

### 3. Lời giải hiện đại: tính "một" là việc của composition root

Điều thật sự cần là **một instance** — không phải **truy cập toàn cục**. Hai thứ đó tách được, và tách chính là lời giải:

```go
// ✅ main tạo MỘT lần — "singleton by construction", không cần cơ chế nào
func main() {
    cfg := loadConfig()
    db := mustConnect(cfg.DSN)          // một instance, vòng đời rõ, đóng có trật tự
    defer db.Close()
    logger := newLogger(cfg.Log)

    orders := order.NewService(db, logger)   // ai cần thì ĐƯỢC TRAO, tường minh
    api := httpapi.New(orders)
    /* ... */
}
```

Tính duy nhất được bảo đảm bởi *cấu trúc chương trình* (chỉ một chỗ gọi `mustConnect`) thay vì bởi *cơ chế chống tạo* (`once`, private constructor). Mọi bệnh án ở mục 2 tan: dependency hiện trên chữ ký, test truyền fake tự do và song song, thứ tự khởi tạo đọc được từ trên xuống trong `main`, và ngày cần hai DB — thêm một dòng, không đập gì.

Đây là lý do các DI framework (Fx, NestJS) gọi scope mặc định là "singleton" mà không mang tiếng xấu: **singleton-vòng-đời** (container tạo một lần, tiêm khắp nơi) lành mạnh; **singleton-truy-cập-toàn-cục** (`getInstance()` từ mọi nơi) mới là bệnh. Chữ dùng chung, hai thứ khác hẳn nhau.

### 4. Phần còn hợp lệ của `sync.Once` — và các ngoại lệ trung thực

`sync.Once`/`sync.OnceValue` vẫn là công cụ tốt cho việc **lazy-init một giá trị đắt bên trong một struct** — chú ý: trong struct được inject, không phải ở biến package:

```go
// ✅ hợp lệ: memoize nội bộ — bên ngoài không ai biết, không ai phụ thuộc
type TemplateRenderer struct {
    dir  string
    tmpl func() (*template.Template, error)   // sync.OnceValues bọc parse đắt đỏ
}
```

Ngoại lệ thực dụng, nói thẳng thay vì giả vờ thuần khiết: `slog.Default()`, `prometheus.DefaultRegisterer`, `http.DefaultClient` — stdlib và hệ sinh thái *có* dùng global có chủ đích cho **cross-cutting concern mà truyền tay qua mọi chữ ký là cực hình**. Chấp nhận được khi: chỉ đọc/ghi qua interface ổn định, có đường override cho test, và app của bạn vẫn *cho phép* inject bản riêng ở chỗ cần. Logger là ca ranh giới nổi tiếng — nhiều codebase Go chọn global logger và sống ổn; chỉ cần biết mình đang trả giá gì (test log khó cách ly) và đừng để tiền lệ đó mở cửa cho `GetDB()`.

### 5. Kết luận nhanh

Singleton GoF trả lời đúng câu hỏi ("một instance") bằng sai công cụ ("global access"). Trong Go/TS hiện đại: **tạo một lần ở composition root + inject** là mặc định; `sync.Once` cho lazy nội bộ; global chỉ cho cross-cutting có đường thoát test. Nếu bạn đang viết `GetXxx()` toàn cục cho một dependency nghiệp vụ — quay lại chương 2.5.

---

## Phần B — Prototype

### 1. Bài toán gốc — đọc trong ngữ cảnh 1994

GoF: *"tạo object mới bằng cách clone một object mẫu, thay vì gọi constructor"*. Bài toán thật của thời đó: (a) khởi tạo **đắt** (parse file, dựng cấu trúc lớn) — clone rẻ hơn dựng lại; (b) hệ thống cần tạo object mà **không biết concrete class lúc compile** (C++ không có reflection tử tế — đăng ký object mẫu rồi clone là cách "new theo tên" duy nhất); (c) object có **hàng chục biến thể cấu hình** — mỗi biến thể một prototype mẫu, clone rồi chỉnh.

Nhìn kỹ sẽ thấy: bài toán (b) ngày nay giải bằng registry factory (4.1); bài toán (c) giải bằng Functional Options/preset (4.2). Còn lại bài toán (a) và một bài toán mới mà GoF không nhấn: **snapshot/copy an toàn** — và đây là phần đáng nói trong Go.

### 2. Hình dạng Go — và cái bẫy nông/sâu

Go không cần interface `Cloneable` cho struct thuần giá trị — phép gán *là* copy:

```go
base := RequestConfig{Timeout: 30 * time.Second, Retries: 3}
perCall := base            // copy — prototype "miễn phí"
perCall.Timeout = 5 * time.Second   // chỉnh bản sao, mẫu nguyên vẹn
```

Nhưng đây là **shallow copy** — và mọi giá trị của pattern sụp đổ nếu quên điều đó:

```go
type Policy struct {
    Name  string
    Rules []Rule            // ❌ slice = con trỏ + len + cap
}
custom := standardPolicy            // "clone"
custom.Rules[0].Limit = 999         // vừa sửa luôn RULE CỦA MẪU — mọi "clone" sau đều nhiễm
```

Struct chứa slice/map/pointer thì phải viết `Clone()` tường minh, deep-copy đúng các field tham chiếu:

```go
func (p Policy) Clone() Policy {
    p.Rules = slices.Clone(p.Rules)   // đủ khi Rule là value; Rule chứa pointer thì đệ quy tiếp
    return p
}
```

Đây chính là địa hạt của chương 1.5: nếu `Policy` **immutable** thì clone thành thừa — chia sẻ mẫu thoải mái vì không ai sửa được. **Prototype là thuốc giảm đau cho mutable state; immutability là vắc-xin.** Có vắc-xin thì ít cần thuốc.

TypeScript: cùng bài — spread `{...obj}` là shallow, bẫy y hệt; `structuredClone()` (ES2022) cho deep copy chuẩn — sự tồn tại của API này trong ngôn ngữ chính là "Prototype tan vào ngôn ngữ".

### 3. Chỗ còn dùng thật trong production

- **`http.Request.Clone(ctx)`** (stdlib): middleware cần sửa request (thêm header, đổi context) mà không phá request của tầng ngoài — clone rồi sửa bản sao. Prototype đúng nghĩa, tồn tại vì Request là mutable struct lớn chứa nhiều tham chiếu — và chính vì viết deep-copy đúng *khó* nên stdlib làm hộ.
- **Config theo tenant/môi trường**: base config load một lần (đắt — parse, validate), mỗi tenant clone + override vài field. Kết hợp đẹp với preset options (4.2) — hai cách biểu diễn cùng một ý.
- **Test fixtures**: `baseOrder()` trả object mẫu hợp lệ, mỗi test clone rồi chỉnh đúng field liên quan — giữ test ngắn và độc lập. Đây có lẽ là nơi dev backend "dùng Prototype" nhiều nhất mỗi ngày mà không gọi tên.
- **Kubernetes `DeepCopy()`**: mọi API object có `DeepCopyObject()` sinh tự động (code-gen) — vì controller chia sẻ object từ cache dùng chung, sửa mà không copy là hỏng cache của mọi controller khác. Bài học kép: (1) Prototype sống khỏe ở nơi shared mutable state là bắt buộc vì hiệu năng; (2) deep-copy đúng khó đến mức Kubernetes phải *sinh code tự động* thay vì tin con người viết tay.

### 4. Kết luận nhanh

Prototype trong Go/TS hiện đại = **biết rõ khi nào cần deep copy và làm nó đúng** — không phải interface `Cloneable` trong sơ đồ lớp. Dùng khi: chỉnh-từ-mẫu (config, fixture), sửa-không-phá-của-người-khác (Request, cache). Trước khi viết `Clone()`, hỏi câu 1.5: *type này có thể immutable không?* — nếu có, bạn vừa tiết kiệm được cả pattern lẫn cả lớp bug.

---

## Tổng kết Creational — chọn công cụ trong 30 giây

```
Cần MỘT instance dùng chung        → tạo ở main, inject (đừng GetInstance)
Khởi tạo có invariant              → constructor NewX(...) (1.2)
Chọn concrete type theo tên/config → factory function; cần mở cho bên ngoài → registry
Họ object phải khớp bộ             → Abstract Factory
Nhiều tùy chọn, API sống lâu       → Functional Options (Go) / options object (TS)
Quy trình dựng nhiều bước          → Builder
Tạo-từ-mẫu, sửa-không-phá-mẫu      → copy/Clone có ý thức nông sâu; cân nhắc immutable trước
```

---

*Tiếp theo: [4.4 — Adapter & Facade](/series/software-design/level-4-patterns/04-adapter-facade/)*
