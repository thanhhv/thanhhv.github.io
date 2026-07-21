+++
title = "1.3 — Coupling, Cohesion & Dependency: thước đo của mọi quyết định thiết kế"
date = "2026-07-17T07:30:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## 1. Problem Statement

Đây là chương quan trọng nhất của Level 1. Mọi nguyên lý (SOLID), mọi pattern (Strategy, Observer...), mọi kiến trúc (Clean, Hexagonal) — khi bóc hết vỏ — đều quy về đúng hai đại lượng:

> **Coupling: hai module dính nhau chặt đến đâu — sửa cái này có phải sửa cái kia không?**
> **Cohesion: những thứ trong cùng một module có thực sự thuộc về nhau không?**

Mục tiêu bất biến của thiết kế: **low coupling giữa các module, high cohesion trong mỗi module**. Nếu bạn chỉ nhớ một câu từ toàn bộ tài liệu này, hãy nhớ câu đó. Khi đánh giá bất kỳ pattern nào, câu hỏi luôn là: "nó giảm coupling ở đâu, tăng cohesion ở đâu, và đổi lại tốn gì?"

Bài toán thực tế: team bạn có codebase mà mỗi khi thêm một field vào response API, phải sửa 7 file ở 4 package khác nhau. Deploy một thay đổi nhỏ cần regression test cả hệ thống. Tại sao? Trả lời được câu này cần ngôn ngữ chính xác về coupling.

## 2. Các mức độ Coupling — từ tệ nhất đến tốt nhất

Coupling không phải nhị phân có/không. Nó là một phổ. Xếp từ tệ đến tốt:

### (1) Content Coupling — module này thò tay sửa ruột module kia

```go
// ❌ package payment sửa thẳng trạng thái nội bộ của order
func ProcessPayment(o *order.Order) {
    // ...
    o.Status = "paid"          // payment biết và sửa ruột của order
    o.PaidAt = time.Now()      // invariant của order bị quyết định ở payment
}
```

Tệ nhất vì: invariant của `Order` giờ được "đồng sở hữu" bởi mọi package. Đổi cách `Order` quản lý trạng thái = sửa khắp nơi. Đây là lý do encapsulation tồn tại (chương 1.2).

### (2) Common/Global Coupling — dính nhau qua biến toàn cục

```go
// ❌ mọi module đọc/ghi chung config toàn cục mutable
var GlobalConfig *Config

func init() { GlobalConfig = loadConfig() }
```

Vì sao tệ: (a) thứ tự khởi tạo trở thành bug tiềm ẩn — ai chạy `init` trước?; (b) test không cách ly được — test A sửa config làm test B fail, chạy song song (`t.Parallel()`) là ăn data race; (c) đọc một hàm không thể biết nó phụ thuộc gì — dependency tàng hình. Đây là lý do "Singleton Everywhere" là anti-pattern nổi tiếng (Level 4).

### (3) Control Coupling — truyền cờ để điều khiển hành vi bên trong

```go
// ❌ caller phải biết logic BÊN TRONG hàm để truyền cờ đúng
func SaveOrder(o *Order, skipValidation bool, isMigration bool) error
```

Cờ boolean là lời thú nhận: "hàm này làm nhiều việc và caller phải chọn". Mỗi cờ nhân đôi số đường đi. Refactor: tách hàm theo mục đích (`SaveOrder`, `ImportLegacyOrder`) — mỗi hàm một việc, không cờ.

### (4) Stamp Coupling — truyền cả struct to trong khi chỉ cần 2 field

```go
// ❌ hàm tính phí ship nhận cả *Order (40 field) nhưng chỉ đọc Total và Weight
func ShippingFee(o *Order) int64

// ✅ nhận đúng thứ nó cần — chữ ký hàm trở thành tài liệu
func ShippingFee(total int64, weightKg float64) int64
```

Vấn đề của bản đầu: (a) muốn test phải dựng cả `Order` 40 field; (b) đọc chữ ký không biết hàm dùng gì — phải đọc thân hàm; (c) hàm giờ coupled với type `Order` — package khác muốn tái sử dụng phải import cả `order`. Nhưng lưu ý trade-off: nếu hàm cần 6-7 field thì truyền struct lại tốt hơn (tránh parameter list dài) — khi đó cân nhắc một struct tham số nhỏ riêng (`ShippingInput`).

### (5) Data Coupling — chỉ truyền đúng dữ liệu cần thiết

Trạng thái lý tưởng cho hàm thuần: input đơn giản → output. Dễ test, dễ hiểu, dễ tái sử dụng.

### Ngoài ra: Temporal Coupling — dính nhau qua thứ tự thời gian

```go
// ❌ phải gọi đúng thứ tự, compiler không giúp gì
p := NewProcessor()
p.SetConfig(cfg)   // quên gọi? panic lúc runtime, ở production, lúc 3h sáng
p.Init()           // gọi Init trước SetConfig? cũng panic
p.Process(data)
```

```go
// ✅ làm trạng thái không hợp lệ trở thành KHÔNG THỂ BIỂU DIỄN
p, err := NewProcessor(cfg) // không có đường nào tạo Processor thiếu config
if err != nil { ... }
p.Process(data)
```

Nguyên tắc vàng: **"Make invalid states unrepresentable"** — dùng constructor và type system để thứ tự sai không compile được, thay vì cầu nguyện dev đọc doc. Đây cũng là nền của Builder pattern và Functional Options (Level 4).

## 3. Cohesion — những thứ trong module có thuộc về nhau không?

Cohesion cao nhất khi mọi phần tử của module cùng phục vụ **một mục đích, một lý do thay đổi**. Các mức từ tệ đến tốt:

- **Coincidental** (tệ nhất): `utils.go` chứa `FormatDate`, `RetryHTTP`, `ParseCSV`, `HashPassword` — chỉ chung nhau chỗ ở. Dấu hiệu: tên package không nói được nó làm gì (`utils`, `common`, `helpers`, `misc`).
- **Logical**: nhóm theo "loại kỹ thuật" — package `validators` chứa validator của order, user, payment. Nghe hợp lý nhưng: sửa nghiệp vụ order phải đụng 4 package (`models`, `validators`, `services`, `handlers`). Đây chính là nguồn gốc smell *Shotgun Surgery*.
- **Functional** (tốt nhất): mọi thứ về một khái niệm nghiệp vụ ở cùng nhau — package `order` chứa model, validation, service của order.

Đây là gốc rễ của tranh luận kinh điển về cấu trúc project:

```
❌ Package theo tầng kỹ thuật          ✅ Package theo tính năng/domain
   (logical cohesion)                     (functional cohesion)

/models                                /order
   order.go, user.go, payment.go          order.go, service.go, repo.go
/services                              /user
   order.go, user.go, payment.go          user.go, service.go, repo.go
/repositories                          /payment
   order.go, user.go, payment.go          payment.go, service.go, repo.go

Thêm 1 field cho order:                Thêm 1 field cho order:
sửa 3-4 package rải rác                sửa 1 package
```

Go còn thêm một lý do kỹ thuật: package theo tầng kiểu `models` import lẫn nhau rất dễ tạo **import cycle** — Go cấm tuyệt đối, compiler báo lỗi ngay. Đây là một trong những ràng buộc tốt nhất của Go: nó **ép** bạn suy nghĩ về chiều phụ thuộc.

## 4. Dependency — hướng của mũi tên là quyết định kiến trúc

Coupling nói về *độ chặt*, dependency nói về *chiều*. Nguyên tắc nền tảng:

> **Mũi tên phụ thuộc phải trỏ từ thứ hay thay đổi → thứ ổn định.**

```
✅ Ổn định ← hay đổi                    ❌ Hay đổi ← ổn định

   handler → service → domain            domain → databaseDriver
   (HTTP đổi thoải mái,                  (đổi driver = domain rung chuyển,
    domain không rung động)               mà domain là thứ đắt nhất để sửa)
```

Domain (nghiệp vụ) là phần ổn định và giá trị nhất — nó không nên biết gì về HTTP, SQL, Kafka. Khi phát hiện domain đang import driver hạ tầng, đó là lúc cần **đảo chiều phụ thuộc** bằng interface (như V3 chương 1.2): domain định nghĩa interface nó cần, hạ tầng implement. Chi tiết đầy đủ ở chương 2.5 (DIP) — ở đây chỉ cần nắm trực giác: **interface là công cụ bẻ chiều mũi tên**.

Cách "nhìn thấy" dependency trong Go: đọc khối `import` của mỗi package chính là đọc sơ đồ kiến trúc. Package domain lý tưởng chỉ import stdlib. Lệnh hữu ích: `go mod graph`, hoặc test kiến trúc tự động bằng cách assert package `domain` không import `infra` (có thể viết test dùng `go/build` hoặc dùng linter như `depguard`).

## 5. Refactoring Journey — gỡ một ca coupling thực tế

Hiện trạng thường gặp trong codebase Node.js lẫn Go:

```go
// V1 — ❌ service import thẳng 4 hạ tầng, tự khởi tạo tất cả
package report

import (
    "github.com/segmentio/kafka-go"
    "github.com/redis/go-redis/v9"
    "gorm.io/gorm"
)

type Service struct{}

func (s *Service) DailyReport(ctx context.Context) (*Report, error) {
    db, _ := gorm.Open(/* DSN hardcode */)          // tự tạo dependency
    rdb := redis.NewClient(&redis.Options{ /*...*/ })

    if cached, err := rdb.Get(ctx, "report:today").Result(); err == nil {
        /* parse & return */
    }
    var rows []OrderRow
    db.Raw("SELECT ...").Scan(&rows)
    r := aggregate(rows)
    rdb.Set(ctx, "report:today", r.JSON(), time.Hour)
    kafka.NewWriter(kafka.WriterConfig{ /*...*/ }).WriteMessages(ctx, /*...*/)
    return r, nil
}
```

Vấn đề: unit test cần Postgres + Redis + Kafka thật; mỗi lần gọi mở connection mới (bug hiệu năng thật, rất phổ biến); service biết DSN, biết địa chỉ broker — config rò rỉ khắp nơi.

**Bước 1 — tách "tạo" khỏi "dùng".** Dependency được tạo một lần ở `main`, truyền vào qua constructor:

```go
// V2 — dependency injection thủ công, không cần framework
type Service struct {
    db    *gorm.DB
    redis *redis.Client
    kafka *kafka.Writer
}

func NewService(db *gorm.DB, r *redis.Client, k *kafka.Writer) *Service {
    return &Service{db: db, redis: r, kafka: k}
}
```

Chỉ riêng bước này đã sửa bug connection, gom config về `main`, và làm dependency **tường minh** — đọc chữ ký `NewService` là thấy hết service cần gì. Nhưng test vẫn cần hạ tầng thật (dù đã có thể trỏ vào testcontainer).

**Bước 2 — trừu tượng hóa theo nhu cầu, KHÔNG phải theo hạ tầng:**

```go
// V3 — interface nói ngôn ngữ của report, không nói "redis" hay "kafka"
type OrderStats interface {
    OrdersInRange(ctx context.Context, from, to time.Time) ([]OrderRow, error)
}
type ReportCache interface {
    Get(ctx context.Context, key string) (*Report, bool)
    Set(ctx context.Context, key string, r *Report, ttl time.Duration)
}
type ReportPublisher interface {
    Published(ctx context.Context, r *Report) error
}

type Service struct {
    stats OrderStats
    cache ReportCache
    pub   ReportPublisher
}
```

Giờ unit test dùng fake in-memory chạy trong micro giây; hạ tầng đổi (Redis → Memcached, Kafka → NATS) không đụng service. Lưu ý ta **không** tạo interface `Database` hay `Cache` tổng quát — interface theo *nhu cầu cụ thể của consumer* (3 method nhỏ) chứ không mô phỏng lại toàn bộ API Redis (mầm của ISP, chương 2.4).

Mỗi bước refactor đều diễn giải được bằng ngôn ngữ chương này: V1→V2 chuyển từ *hidden coupling* sang *explicit coupling* (chưa giảm nhưng lộ ra để quản lý); V2→V3 giảm cấp coupling từ "dính vào concrete type của hạ tầng" xuống "dính vào hợp đồng nhỏ do chính mình định nghĩa".

## 6. Trade-off

- **Coupling bằng 0 là ảo tưởng.** Hệ thống hữu ích phải có các phần nói chuyện với nhau. Mục tiêu là coupling *có chủ đích, lỏng, và trỏ đúng chiều* — không phải loại bỏ coupling.
- **Giảm coupling luôn có giá**: thêm interface, thêm file, thêm một bước gián tiếp khi đọc code. Với module nhỏ ổn định, coupling trực tiếp rẻ hơn tổng chi phí của lớp trừu tượng.
- **Cohesion có thể "quá cao" theo nghĩa sai**: gom mọi thứ liên quan đến order vào một struct 3.000 dòng là cohesion giả — đó là God Object. Cohesion đúng đo bằng *lý do thay đổi*, không phải bằng *chủ đề*.
- **Microservices là bài học coupling đắt nhất thập kỷ**: tách service mà không tách đúng ranh giới nghiệp vụ → distributed monolith — coupling giữ nguyên nhưng giờ cộng thêm network latency, partial failure và versioning. Coupling không biến mất khi bạn cắm HTTP vào giữa; nó chỉ đắt hơn.

## 7. Production Examples

- **`context.Context` (Go)**: giải bài toán coupling kinh điển — làm sao truyền deadline/cancellation/trace-id xuyên qua nhiều tầng mà không thêm N tham số vào mọi hàm? Go chọn quy ước "tham số đầu tiên là `ctx`" — tường minh nhưng lan tỏa (mọi hàm phải khai báo). Node.js chọn `AsyncLocalStorage` — ngầm, không đổi chữ ký hàm, nhưng dependency tàng hình (đọc hàm không biết nó dùng context). Hai lời giải, hai trade-off — không có lời giải miễn phí.
- **`net/http`**: `http.Handler` chỉ phụ thuộc `ResponseWriter` (interface) và `*Request` — không phụ thuộc TCP, TLS, HTTP/2. Nhờ đó `httptest` fake được toàn bộ mà không mở socket nào — thiết kế coupling tốt *tự động sinh ra* khả năng test.
- **Kubernetes**: controller không gọi nhau trực tiếp — mọi controller chỉ coupling với API Server (đọc/ghi desired state). Thêm một controller mới không sửa controller nào. Đây là coupling qua dữ liệu trung tâm thay vì qua lời gọi trực tiếp — mầm tư duy event-driven (Level 5).

## 8. Anti-pattern & Khi nào KHÔNG cần tối ưu

- **Package `utils`/`common` phình to** — coincidental cohesion; mọi package import nó, nó import nhiều thứ khác → nam châm hút import cycle. Thuốc: giải tán theo chức năng (`timeutil`, `moneyfmt`) hoặc trả code về package dùng nó.
- **Distributed monolith** — như trên, coupling cấp mạng.
- **Event tàng hình**: lạm dụng event để "giảm coupling" đến mức không ai trace được luồng nghiệp vụ — coupling hành vi vẫn còn, chỉ là không nhìn thấy nữa. Giảm coupling *cú pháp* nhưng tăng chi phí *hiểu*.
- **Khi nào kệ nó**: script nhỏ, tool một lần, prototype — coupling chặt hoàn toàn ổn. Và trong mọi codebase, hai module *thay đổi cùng nhau một cách tự nhiên* (ví dụ struct và validator của chính nó) thì **nên** ở gần nhau và coupling chặt — đó là cohesion, không phải lỗi.

---

*Chương tiếp theo: [1.4 — Composition vs Inheritance & Polymorphism](/series/software-design/level-1-fundamentals/04-composition-inheritance-polymorphism/) — vì sao Go từ chối kế thừa, và đó là quyết định thiết kế sáng suốt.*
