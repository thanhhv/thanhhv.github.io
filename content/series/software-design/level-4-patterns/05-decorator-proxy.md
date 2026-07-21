+++
title = "4.5 — Decorator & Proxy — và Middleware, đứa con lớn nhất của chúng"
date = "2026-07-17T10:50:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## 1. Vị trí xuất phát

Decorator không cần giới thiệu dài: chương 1.4 đã *phát minh lại* nó khi tháo cây kế thừa Notifier — retry/rate-limit/logging thành các wrapper cùng interface, tổ hợp tự do. Chương này làm ba việc: gọi tên chính thức và nêu đủ cấu trúc; phân định với người anh em song sinh Proxy; và mổ **Middleware** — hình thái Decorator quan trọng nhất của backend hiện đại, kèm những bài học production mà bản đồ chơi ở 1.4 chưa chạm tới.

## 2. Decorator — chính danh

**Intent (GoF)**: gắn thêm trách nhiệm cho một object **động, theo từng object** — thay thế mềm dẻo cho việc sinh lớp con. **Cấu trúc**: wrapper implement *cùng interface* với thứ nó bọc, giữ tham chiếu `next`, thêm hành vi trước/sau khi ủy quyền:

```
Client ──► [Logging ──► [Retry ──► [RateLimit ──► CoreNotifier]]]
            cùng một interface Notifier ở MỌI tầng — nên chồng vô hạn, hoán vị tự do
```

Điều kiện sống còn — cũng là điều 1.4 đã chỉ ra: **interface phải nhỏ** (1 method thì wrapper 5 dòng; 20 method thì mỗi decorator phải viết 19 method forward — ISP quyết định pattern này khả thi hay không) và **decorator trong suốt với hợp đồng** (LSP: `RetryNotifier` vẫn phải hành xử như một `Notifier` đúng nghĩa — caller không biết mình đang nói chuyện với chồng wrapper).

Điểm mới đáng bổ sung ở cấp production — **thứ tự bọc là ngữ nghĩa, không phải chi tiết**:

```go
// Retry bọc NGOÀI RateLimit: mỗi lần retry lại xếp hàng qua rate-limiter — đúng
retry(ratelimit(core))
// RateLimit bọc ngoài Retry: 3 lần retry BẮN LIÊN TIẾP không qua limiter — có thể sập đối tác
ratelimit(retry(core))
```

Cùng các mảnh, hai hành vi khác nhau. Hệ quả thực dụng: chuỗi wrapper dài phải được **lắp ở một nơi duy nhất** (composition root/factory) kèm comment giải thích thứ tự — đừng để mỗi nơi tự lắp một kiểu.

## 3. Proxy — song sinh khác tính cách

Cấu trúc **giống hệt** Decorator: cùng interface, ôm tham chiếu, đứng giữa client và object thật. Khác nhau nằm ở **ý định** — và ý định quyết định hành vi được phép:

- **Decorator**: *cộng thêm* hành vi; luôn gọi xuống `next`; client chủ động chọn bọc gì.
- **Proxy**: *kiểm soát quyền truy cập* tới object thật; **được quyền không gọi xuống** — trả từ cache, chặn vì thiếu quyền, trì hoãn tạo object thật; client thường *không biết và không chọn* — proxy được ấn vào tay như thể là object thật.

Bốn dạng proxy kinh điển, tất cả đều quen mặt trong backend:

```go
// (1) Caching proxy — có thể KHÔNG BAO GIỜ chạm object thật
type CachedRates struct {
    next  RateSource
    cache *ttlcache.Cache[string, Rate]
}
func (c *CachedRates) Rate(ctx context.Context, ccy string) (Rate, error) {
    if r, ok := c.cache.Get(ccy); ok {
        return r, nil                      // ← điểm phân biệt với decorator: dừng tại đây
    }
    r, err := c.next.Rate(ctx, ccy)
    if err == nil { c.cache.Set(ccy, r) }
    return r, err
}

// (2) Protection proxy — chặn trước cửa
func (p *AuthzRepo) Delete(ctx context.Context, id string) error {
    if !authz.From(ctx).Can("orders:delete") {
        return ErrForbidden                // không gọi xuống
    }
    return p.next.Delete(ctx, id)
}

// (3) Lazy/virtual proxy — object thật đắt, chỉ dựng khi đụng tới (sync.OnceValue, 4.3)
// (4) Remote proxy — object thật ở process khác: chính là gRPC client stub
```

**Remote proxy đáng một đoạn riêng** vì nó là dạng lớn nhất đời thực: client stub của gRPC *trông như* gọi method thường nhưng sau lưng là network — serialize, gửi, chờ, parse. Tiện lợi đó có mặt tối nổi tiếng ("fallacies of distributed computing"): che network *quá kỹ* khiến dev quên latency, partial failure, retry storm. Thiết kế gRPC hiện đại phản tỉnh điều này bằng cách *lộ có chủ đích*: bắt buộc `ctx` (deadline/cancel), error code phân biệt lỗi mạng — proxy tốt che **cơ chế** nhưng không che **bản chất** của thứ nó đại diện.

**So sánh chốt — cả bốn pattern "bọc"**:

| | Interface so với thứ được bọc | Được phép không gọi xuống? | Ai quyết định bọc | Ý định một chữ |
|---|---|---|---|---|
| Adapter | **Khác** (dịch sang ổ cắm khác) | Không | Người tích hợp | Dịch |
| Facade | **Mới, gọn hơn** (che N thành phần) | — (tự điều phối) | Người thiết kế API | Gom |
| Decorator | **Y hệt** | Không (luôn ủy quyền) | Client, lúc lắp ráp | Cộng |
| Proxy | **Y hệt** | **Có** | Hạ tầng/framework, thường ẩn | Gác |

## 4. Middleware — Decorator ăn cả thế giới backend

### Từ Decorator đến Middleware

Áp Decorator vào interface một-method quan trọng nhất của Go:

```go
// http.Handler chỉ có ServeHTTP(w, r) — mảnh đất hoàn hảo cho decorator
// Middleware = hàm biến một Handler thành Handler đã-được-bọc:
type Middleware func(http.Handler) http.Handler

func Logging(logger *slog.Logger) Middleware {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            start := time.Now()
            next.ServeHTTP(w, r)
            logger.Info("http", "method", r.Method, "path", r.URL.Path,
                "dur", time.Since(start))
        })
    }
}

func Recover() Middleware { /* defer/recover quanh next */ }
func Auth(verifier TokenVerifier) Middleware { /* parse token, bỏ vào context, hoặc 401 */ }

// Lắp chuỗi — nhìn rõ từng tầng hành tây:
handler = Recover()(Logging(logger)(Auth(v)(mux)))

// hoặc helper Chain cho dễ đọc (mọi router Go đều có bản riêng):
handler = Chain(mux, Recover(), Logging(logger), Auth(v))
```

Đây là Decorator thuần chủng — cùng interface, thêm hành vi, ủy quyền — cộng một nét của Chain of Responsibility: middleware **được quyền chặn** (Auth trả 401 không gọi next) — trên phổ Decorator↔Proxy, middleware đứng giữa, và chẳng ai buồn phân loại nữa: nó đã là pattern độc lập với cái tên riêng.

### Ba bài học production của middleware

**(a) Truyền dữ liệu xuống bằng context — và kỷ luật của nó.** Auth middleware xác thực xong, handler cần biết "user nào?". Kênh chuẩn: `context.WithValue` — kèm quy tắc chống bừa bãi: key là unexported type riêng (chống trùng), chỉ chở **dữ liệu phạm vi request** (user, trace ID, deadline) — không bao giờ chở dependency (logger, DB — thứ đó đi qua constructor, 2.5), và có accessor typed (`authz.From(ctx)`) thay vì rải type-assertion.

**(b) Bọc ResponseWriter — cái bẫy đã báo trước.** Logging middleware muốn ghi status code — nhưng `ResponseWriter` không có getter; phải bọc chính writer:

```go
type statusRecorder struct {
    http.ResponseWriter
    status int
}
func (r *statusRecorder) WriteHeader(code int) { r.status = code; r.ResponseWriter.WriteHeader(code) }
```

Bọc thế này **làm rụng optional interface** của writer gốc (`http.Flusher`, `http.Hijacker` — chương 2.4 đã cảnh báo): SSE/websocket đi qua middleware logging bỗng hỏng một cách bí ẩn. Đây là bug có thật ở nhiều thư viện; lối thoát hiện đại là `http.NewResponseController`. Bài học khái quát: **decorator trên interface có capability ẩn phải quyết định tường minh chuyện chuyển tiếp capability** — im lặng nuốt mất là vi phạm LSP dạng tinh vi.

**(c) Trong suốt là một ảo tưởng có ích — đo được là bắt buộc.** Chuỗi 12 middleware là 12 khung stack cho *mọi* request và là 12 nơi bug có thể ẩn ("sao request treo? — à, middleware số 7 buffer body"). Kỷ luật: mỗi middleware một việc nhỏ; chuỗi khai báo một nơi; và tracing span cho middleware nặng — để "cái hành tây" nhìn được bằng mắt trên APM thay vì bằng niềm tin.

### Cùng một pattern, ba hệ sinh thái

- **Gin/Echo**: biến thể *imperative* — middleware nhận context của framework và tự gọi `c.Next()` (thay vì nhận-trả handler). Cùng ngữ nghĩa, thêm quyền: code *sau* `c.Next()` chạy chiều đi ra (đo thời gian, sửa response) và `c.Abort()` chặn chuỗi. Đánh đổi: dính chặt vào type của framework — middleware Gin không chạy trên Echo, còn middleware `func(http.Handler) http.Handler` chạy trên mọi router chuẩn stdlib (chi phí lock-in của framework, hiện nguyên hình ở tầng middleware).
- **Express/NestJS (TS)**: `(req, res, next)` — ông tổ phổ biến hóa từ "middleware". Bài học đau của Express mà Go tránh được nhờ type system: quên gọi `next()` → request treo im lặng; lỗi async không `next(err)` → unhandled rejection. NestJS xếp tầng lại thành guard/interceptor/pipe/filter — bốn loại middleware chuyên môn hóa, đổi tự do lấy trật tự.
- **gRPC interceptor, Kafka consumer middleware, temporal interceptor**: cùng hình dạng cho RPC/message — bằng chứng pattern trưởng thành: *mọi* đường ống xử lý request đều mọc ra cùng cấu trúc.

## 5. Anti-pattern

- **Business logic trong middleware**: check "đơn hàng thuộc user" (cần query DB, hiểu domain) nhét vào middleware — nghiệp vụ trốn khỏi tầng use case, test phải dựng cả HTTP stack. Middleware cho **cross-cutting** (log, auth, trace, limit); nghiệp vụ ở trong handler/use case.
- **Context thành túi đồ nghề**: nhét service, logger, DB vào context — dependency tàng hình toàn tập (đã cấm ở mục 4a; nhắc vì đây là lỗi phổ biến nhất của dev mới với middleware).
- **Chuỗi bọc không ai nắm**: wrapper lắp rải rác ở nhiều tầng khởi tạo — đến lúc debug không ai trả lời được "rốt cuộc request đi qua những gì, theo thứ tự nào?".
- **Decorator cho interface béo**: 20 method forward để thêm hành vi cho 1 method — tín hiệu sai từ ISP, sửa interface trước khi viết decorator.

## 6. Khi nào KHÔNG dùng

Hành vi chỉ cần ở **một chỗ**: gọi thẳng trong hàm (log một dòng thì cứ log, đừng dựng LoggingDecorator). Hành vi **không cùng hợp đồng** với interface (biến đổi dữ liệu nghiệp vụ): đó là một bước pipeline, không phải decorator. Và ở app 3 endpoint: `mux` + vài dòng trong handler vẫn hợp lệ — middleware stack là khoản đầu tư trả lãi theo số endpoint và số concern lặp lại.

---

*Tiếp theo: [4.6 — Composite, Bridge & Flyweight](/series/software-design/level-4-patterns/06-composite-bridge-flyweight/)*
