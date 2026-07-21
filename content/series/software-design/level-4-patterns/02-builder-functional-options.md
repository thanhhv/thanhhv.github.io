+++
title = "4.2 — Builder & Functional Options: khởi tạo phức tạp mà không phát điên"
date = "2026-07-17T10:20:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## 1. Problem Statement

Factory (4.1) giải bài toán *chọn loại nào*. Chương này giải bài toán trực giao: **một loại duy nhất nhưng khởi tạo phức tạp** — nhiều tham số, đa số tùy chọn, có ràng buộc chéo giữa các tham số, có giá trị mặc định thông minh. Nạn nhân điển hình: HTTP client, server, DB connection, message consumer — thứ nào cũng 15+ nút vặn.

## 2. Code Evolution — con đường đau khổ quen thuộc

**V1 — constructor phình dần** (Massive Constructor, đã điểm mặt trong danh sách anti-pattern từ đầu tài liệu):

```go
// Tháng 1: đẹp
func NewClient(baseURL string) *Client

// Tháng 9: mỗi lần thêm tùy chọn là một BREAKING CHANGE cho mọi caller
func NewClient(baseURL string, timeout time.Duration, retries int,
    retryBackoff time.Duration, logger *slog.Logger, tracer trace.Tracer,
    maxIdleConns int, userAgent string, apiKey string) *Client

// Call site — đố ai biết 30*time.Second là timeout hay backoff:
c := NewClient("https://api.x.vn", 30*time.Second, 3, time.Second, nil, nil, 10, "", key)
```

**V2 — config struct.** Bước tiến tự nhiên và *đủ tốt cho rất nhiều trường hợp*:

```go
type ClientConfig struct {
    BaseURL string
    Timeout time.Duration   // zero value → dùng default? hay thật sự muốn 0?
    Retries int
    Logger  *slog.Logger
}
func NewClient(cfg ClientConfig) (*Client, error)
```

Được: named field, thêm field không vỡ caller cũ, struct tài liệu hóa toàn bộ bề mặt config. Ba điểm yếu còn lại: (1) **zero value nhập nhằng** — `Timeout: 0` là "chưa set, cho tôi default 30s" hay "tôi thật sự muốn không timeout"?; (2) **không chặn được mutate sau khi tạo** — caller giữ struct, sửa field lúc client đang chạy; (3) **default phải xử lý rải rác** trong constructor (`if cfg.Timeout == 0 { cfg.Timeout = 30s }` — nhân 15 field).

Hai pattern của chương này là hai lời giải cho phần còn lại đó — một của thế giới OOP kinh điển, một là idiom đặc sản Go.

## 3. Builder — dạng GoF/fluent kinh điển

```typescript
// TS — fluent builder, thói quen chuẩn của hệ Java/TS
const client = new ClientBuilder("https://api.x.vn")
  .timeout(30_000)
  .retries(3, { backoff: 1_000 })
  .logger(myLogger)
  .build();                       // ← validate ràng buộc chéo TẠI ĐÂY, một lần
```

Cấu trúc: object trung gian (builder) **mutable, thu thập lựa chọn từng bước**; `build()` validate tổng thể rồi sinh ra object đích **immutable, hợp lệ trọn vẹn** (1.5: invalid state không thể biểu diễn — trạng thái nửa vời chỉ tồn tại trong builder, không bao giờ trong object thật).

Giá trị thật của Builder nằm ở hai chỗ mà config struct không với tới: **validate ràng buộc chéo tại một điểm chốt** (`retries > 0` thì `backoff` phải có; `apiKey` và `mTLS` không được cùng lúc), và **quy trình dựng nhiều bước có thứ tự/điều kiện** (dựng SQL query, dựng request phức tạp). GoF còn nhấn khía cạnh ít người nhớ: *cùng một quy trình dựng, nhiều representation* — một `ReportBuilder` interface, hai implementation dựng ra PDF và HTML từ cùng chuỗi lời gọi `addSection/addTable` — gặp trong thực tế ở các thư viện render đa đầu ra.

Trong Go, fluent builder tồn tại nhưng **không phải mặc định văn hóa** vì hai lẽ: method chaining vướng convention trả error (chain thì error phải tích lũy trong builder, check dồn ở `Build()` — trễ và dễ quên); và Go đã có idiom riêng chiếm chỗ này:

## 4. Functional Options — Builder phiên bản Go

Idiom do Rob Pike khởi xướng và Dave Cheney phổ biến — trở thành chuẩn de-facto cho public API của thư viện Go:

```go
type Client struct {
    baseURL string
    timeout time.Duration
    retries int
    logger  *slog.Logger
}

// Option = function biến đổi config — MỖI TÙY CHỌN LÀ MỘT GIÁ TRỊ
type Option func(*Client) error

func WithTimeout(d time.Duration) Option {
    return func(c *Client) error {
        if d <= 0 {
            return fmt.Errorf("timeout must be positive, got %v", d)  // validate TẠI option
        }
        c.timeout = d
        return nil
    }
}

func WithRetries(n int, backoff time.Duration) Option { /* ... */ }
func WithLogger(l *slog.Logger) Option                { /* ... */ }

func NewClient(baseURL string, opts ...Option) (*Client, error) {
    c := &Client{
        baseURL: baseURL,
        timeout: 30 * time.Second,   // DEFAULT tường minh, một chỗ, đọc được
        retries: 3,
    }
    for _, opt := range opts {
        if err := opt(c); err != nil {
            return nil, fmt.Errorf("client: %w", err)
        }
    }
    if err := c.validate(); err != nil {   // ràng buộc chéo — vẫn có điểm chốt
        return nil, err
    }
    return c, nil
}
```

```go
// Call site — đọc như văn xuôi, chỉ nói thứ mình cần khác default:
c, err := NewClient("https://api.x.vn",
    WithTimeout(10*time.Second),
    WithLogger(logger),
)
```

Vì sao nó thắng config struct đúng ở ba điểm yếu đã nêu: **default tường minh** tại một chỗ, không đoán qua zero value — "không truyền option" ≠ "truyền giá trị zero", nhập nhằng biến mất; **object đóng sau khi tạo** — field private, không mutate được từ ngoài; và tham số bắt buộc (baseURL) tách hẳn khỏi tùy chọn ngay trên chữ ký.

Cộng thêm sức mạnh chỉ có ở "option là giá trị": option **compose được** — gom bộ, đặt tên, tái sử dụng:

```go
// Preset cho môi trường — chính là tri thức vận hành được đóng gói
func ProductionDefaults() []Option {
    return []Option{WithTimeout(5 * time.Second), WithRetries(3, time.Second)}
}
c, _ := NewClient(url, append(ProductionDefaults(), WithLogger(l))...)
```

**Chi phí thật, nói thẳng**: mỗi tùy chọn ~5 dòng boilerplate (so với 1 dòng field của config struct); người mới đọc `Option func(*Client) error` lần đầu phải dừng lại nghĩ; và bề mặt API là *danh sách hàm With\** rải trong godoc thay vì một struct nhìn phát thấy hết. Vậy nên đường ranh thực dụng của cộng đồng Go:

```
Config struct      → app nội bộ, config load từ file/env (viper/envconfig map thẳng vào struct),
                     ít ràng buộc chéo, người đọc cần "nhìn một phát thấy cả bề mặt"
Functional Options → THƯ VIỆN public: API phải tiến hóa nhiều năm không breaking change,
                     default thông minh, validate per-option, tham số bắt buộc tách bạch
Fluent Builder     → quy trình dựng NHIỀU BƯỚC có thứ tự (query builder, request builder) —
                     lúc này "chuỗi bước" chính là ngữ nghĩa, chaining là cú pháp tự nhiên
```

## 5. So sánh khách quan: Factory vs Builder

Cặp so sánh hay bị lẫn vì cùng đứng ở "chỗ tạo object":

| | Factory | Builder / Functional Options |
|---|---|---|
| Câu hỏi trả lời | Tạo **loại nào**? (chọn concrete type sau interface) | Tạo **như thế nào**? (cấu hình một type phức tạp) |
| Input đặc trưng | Một discriminator (tên, config env) | Nhiều lựa chọn tích lũy |
| Output | Interface (che loại) | Concrete type (đầy đủ, hợp lệ) |
| Gốc nguyên lý | OCP — mở theo chiều *loại* (2.2) | Encapsulation invariant lúc khởi tạo (1.2, 1.5) |
| Kết hợp | Hoàn toàn tự nhiên: factory chọn loại, bên trong dùng options để dựng — `sql.Open(driverName, dsn)` chính là factory nhận "builder input" dạng chuỗi DSN | |

## 6. Prototype-of-production sightings

- **gRPC-Go**: `grpc.NewClient(target, grpc.WithTransportCredentials(...), grpc.WithUnaryInterceptor(...))` — functional options ở quy mô API công khai lớn nhất hệ Go; hàng chục option tiến hóa 10 năm không phá caller cũ. Đây là lý do tồn tại số một của pattern: **tiến hóa API không breaking change**.
- **Uber Fx / zap**: zap dùng cả hai tầng đúng sách — `zap.Config` struct (map từ file config) *và* `zap.Option` (điều chỉnh programmatic) — minh chứng hai pattern không loại trừ nhau mà phục vụ hai loại người dùng.
- **`http.Server` (stdlib)**: đối chứng ngược — config struct trần, public field, thậm chí mutate được. Trả giá: tài liệu phải ghi "không sửa field sau khi Serve", zero value từng gây bug timeout thật (`ReadTimeout: 0` = không timeout — nhiều server production bị slowloris vì default này). Stdlib giữ nó vì đơn giản + lịch sử; bài học zero-value của nó được cả cộng đồng học thuộc.
- **`strings.Builder` / `bytes.Buffer`**: builder theo nghĩa nguyên thủy nhất — tích lũy từng phần, chốt bằng `String()`. Tồn tại vì lý do hiệu năng (1.5: immutability của string có giá; builder là van xả).
- **Squirrel / GORM clause builder, Prisma/Knex (TS)**: query builder — đất diễn đúng nhất của fluent chaining: câu query *là* một chuỗi mệnh đề có thứ tự.

## 7. Anti-pattern

- **Builder cho struct 3 field**: nghi lễ. `&Config{A, B, C}` xong việc.
- **Option làm việc nặng**: `WithDatabase(dsn)` mà bên trong *mở luôn connection* — option phải là phép gán + validate; side effect nặng thuộc về constructor hoặc `Start()`. Option có side effect = thứ tự truyền option bắt đầu mang ngữ nghĩa ngầm — đúng loại bug temporal coupling (1.3).
- **Builder rò rỉ object nửa chín**: cho phép dùng object trước `build()` xong (hoặc `Build()` không validate) — mất chính cái lý do pattern tồn tại.
- **Hai đường config song song không rõ luật**: vừa public field vừa options cho cùng knob — ai thắng ai khi cả hai cùng set? Chọn một cửa duy nhất.

## 8. Khi nào KHÔNG dùng

Struct nội bộ giữa các package của cùng app: literal `&Client{...}` với named field của Go đã là "builder miễn phí 80%" — thêm nghi thức chỉ đáng khi có invariant thật sự cần gác. Ba câu hỏi trước khi viết options/builder: *(1) có default không tầm thường không? (2) có validate/ràng buộc chéo không? (3) API này có sống lâu và public không?* — dưới hai câu "có", dùng struct trần.

---

*Tiếp theo: [4.3 — Singleton & Prototype](/series/software-design/level-4-patterns/03-singleton-prototype/)*
