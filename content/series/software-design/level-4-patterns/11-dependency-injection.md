+++
title = "4.11 — Dependency Injection & DI Container: từ nguyên tắc đến công cụ, và ranh giới giữa hai thứ"
date = "2026-07-17T11:50:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## 1. Đặt lại nền — ba khái niệm hay bị trộn làm một

Chương 2.5 đã cắm mốc, giờ dựng thành khung hoàn chỉnh vì toàn bộ chương này đứng trên đó:

```
DIP (nguyên tắc)      : chiều mũi tên — nghiệp vụ định nghĩa interface, hạ tầng implement
DI (kỹ thuật)         : cách trao dependency — tạo ở ngoài, đưa vào qua constructor/tham số
DI Container (công cụ): máy tự động hóa việc trao — tự phân giải graph, tự gọi constructor
```

Ba tầng độc lập: dùng container cả ngày vẫn vi phạm DIP được (inject thẳng concrete hạ tầng vào domain); tuân DIP hoàn hảo không cần container nào (wiring tay ở `main`). Chương này trả lời câu hỏi thực dụng còn nợ: **khi nào kỹ thuật cần leo lên công cụ — và cái giá của cú leo đó.**

## 2. Code evolution — hành trình của một hàm `main`

**V1 — wiring tay, 20 dòng.** Đã thấy ở 2.5: service Go điển hình, `main` đọc từ trên xuống như một bản khai kiến trúc. **Đây là baseline và là điểm dừng đúng của đa số service.**

**V2 — app lớn lên: 60 component.** `main` giờ 400 dòng wiring; các cụm khởi tạo phụ thuộc nhau chằng chịt; thêm một dependency vào constructor của service tầng sâu → sửa dây chuyền các hàm khởi tạo phía trên; thứ tự đóng (shutdown ngược thứ tự mở) tự quản bằng tay dễ sai. Ba nỗi đau có thật — và có hai lối đi:

**Lối A — kỷ luật hóa wiring tay:**

```go
// Gom theo tầng, mỗi tầng một hàm dựng — main vẫn tường minh, giờ có mục lục
func buildInfra(cfg Config) (*Infra, error)      // db, redis, kafka, s3...
func buildDomain(inf *Infra) *Domain             // các service nghiệp vụ
func buildTransport(d *Domain) http.Handler      // handler, middleware

func main() {
    inf := must(buildInfra(cfg))
    defer inf.Close()                            // đóng có trật tự — tay nhưng rõ
    h := buildTransport(buildDomain(inf))
    /* ... */
}
```

Cộng thêm máy phát code: **Wire (google/wire)** — khai provider, Wire *sinh ra* file wiring tay lúc build. Được cả hai đầu: hết viết boilerplate, nhưng sản phẩm là **code Go thường đọc được, debug được, sai thì fail lúc compile**. Wire là lựa chọn "container không runtime" — rất Go về khí chất.

**Lối B — runtime container: Uber Fx (Go), NestJS (TS):**

```go
// Fx: khai BẢN CHẤT (ai cung cấp gì), container tự suy ra THỨ TỰ
fx.New(
    fx.Provide(newConfig, newDB, newRedis, newOrderService, newHandler),
    fx.Invoke(startHTTP),
).Run()
// Fx phân giải graph bằng reflection lúc RUNTIME, quản luôn lifecycle OnStart/OnStop
```

## 3. Trade-off thật của runtime container — nói đủ hai chiều

**Cái được** (lý do Uber xây Fx cho nghìn service):

- **Chi phí thêm node vào graph ~ O(1)**: thêm dependency thứ 61 = một `fx.Provide`, không sửa dây chuyền hàm dựng — đây là điểm gãy quy mô thật sự mà wiring tay đau.
- **Lifecycle chuẩn hóa**: OnStart/OnStop theo đúng thứ tự graph, timeout, graceful shutdown — bài toán mà mọi codebase tay đều tự viết một bản (thường thiếu).
- **Chuẩn hóa tổ chức**: nghìn service cùng khung module (`fx.Module("payment", ...)`) — người mới đọc service lạ vẫn biết tìm gì ở đâu. Với tổ chức lớn, tính đồng phục này *là* sản phẩm.

**Cái mất** (lý do phần lớn cộng đồng Go ngoài Uber-scale không dùng):

- **Lỗi wiring dời từ compile-time sang runtime**: quên provider → app nổ lúc khởi động với stack trace của framework, không phải lỗi biên dịch chỉ đúng dòng. Go đánh đổi cả thiết kế ngôn ngữ để có lỗi sớm — container trả lại lỗi muộn.
- **Luồng tàng hình**: "constructor này ai gọi, khi nào?" — trả lời bằng cách hiểu reflection của framework thay vì đọc code. Grep `newOrderService` không còn thấy call site.
- **Hút vào hệ tư tưởng**: Fx kéo theo fx.Module, fx.Annotate, group, tag... — một ngôn ngữ cấu hình graph phải học, và mọi thư viện nội bộ dần viết "cho Fx".

**NestJS** — đối chiếu hệ TS: container là *bản sắc* framework (decorator `@Injectable`, token, module system). Trong khuôn khổ đó nó vận hành tốt vì **cả hệ sinh thái đồng thuận** — bài học giống Gin/Echo middleware (4.5): công cụ áp đặt khuôn thắng khi bạn mua trọn khuôn. Điểm yếu riêng đáng biết: token theo class + `emitDecoratorMetadata` — inject theo *interface* TS không tự nhiên (interface biến mất lúc runtime), phải dùng token thủ công → nhiều codebase NestJS bỏ luôn interface, inject thẳng concrete — **container xịn mà DIP rỗng**, minh họa sống cho mục 1.

**Ma trận quyết định:**

```
Service < ~30 component            → wiring tay (V1/V2-A). Không có gì phải bàn thêm
App lớn, muốn giữ khí chất Go      → hàm dựng theo tầng + Wire (compile-time)
Tổ chức nghìn service cần đồng phục → Fx/container — mua chuẩn hóa, trả bằng magic
TS/NestJS                          → container của framework; kỷ luật DIP tự giữ
```

## 4. So sánh khách quan: Dependency Injection vs Service Locator

Cặp so sánh cuối cùng còn nợ từ đầu tài liệu — và quan trọng vì Service Locator trông *giống hệt* DI ở khoảng cách 10 mét: cũng container, cũng "tách khởi tạo khỏi sử dụng":

```go
// Service Locator: component TỰ ĐI LẤY từ registry toàn cục
func (s *OrderService) Confirm(ctx context.Context, id string) error {
    repo := locator.Get[OrderRepo]()        // ← hỏi xin lúc CHẠY, từ TRONG ruột
    mailer := locator.Get[Mailer]()
    /* ... */
}

// DI: component ĐƯỢC TRAO qua constructor — từ NGOÀI, lúc DỰNG
func NewOrderService(repo OrderRepo, mailer Mailer) *OrderService
```

| | DI (constructor injection) | Service Locator |
|---|---|---|
| Dependency lộ ở đâu | **Chữ ký constructor** — hợp đồng public, đọc là biết cần gì | **Rải trong thân method** — muốn biết cần gì phải đọc hết ruột |
| Chiều phụ thuộc | Component không biết gì về cơ chế wiring | Mọi component **import locator** — coupling toàn codebase vào một hub |
| Lỗi thiếu dependency | Lúc dựng graph (compile với Wire, startup-fail-fast với Fx) | **Lúc method chạy** — có thể là 3h sáng, nhánh code hiếm |
| Test | Truyền fake qua constructor — cách ly tự nhiên, song song vô tư | Phải mutate locator toàn cục — test dính nhau, không song song (đúng bệnh án Singleton 4.3) |
| Cám dỗ | — | Quá tiện: khỏi sửa chữ ký, cứ `Get` là có → dependency mọc không kiểm soát |

Nhận diện quan trọng: **Service Locator = Singleton `GetInstance()` tổng quát hóa** — mọi bệnh án ở 4.3 áp nguyên. Và ranh giới tinh tế đáng nhớ: bên trong composition root, container *được phép* hoạt động như locator (nó là máy wiring — đó là việc của nó); bệnh chỉ phát khi tham chiếu container **rò vào code nghiệp vụ** (`@Inject(REQUEST)` rải khắp NestJS service, `ctx.Get("db")` trong handler Gin — context framework làm locator trá hình là dạng phổ biến nhất trong thực tế Go/TS, đã cảnh báo ở 4.5 mục context).

Trường hợp Service Locator còn biện hộ được — nói cho công bằng: plugin system nơi tập dependency **không biết trước lúc compile** (plugin tự hỏi host "có gì cho tôi?"), và legacy migration (locator làm cầu tạm khi chưa gỡ được wiring cũ — nợ có sổ, 3.0). Ngoài hai ngách đó: constructor injection thắng tuyệt đối.

## 5. Anti-pattern tổng hợp của chủ đề DI

- **Constructor 12 tham số**: không phải lỗi của DI — là SRP gào lên (3.1 God Object); container chỉ làm bạn *hết đau* mà không hết bệnh: thêm dep thứ 13 dễ quá nên chẳng ai tách class nữa. Container giỏi che smell — kỷ luật review phải bù lại.
- **Inject những thứ không cần inject**: `Clock`, `UUIDGenerator`, `RandomSource` interface cho mọi service "để test được" — trong khi 90% test không cần điều khiển thời gian. Inject khi có nhu cầu test thật (2.5: interface `Clock` chỉ khi thật sự test thời gian).
- **Scope nhầm**: đối tượng per-request (transaction, user context) đăng ký singleton-scope trong container — chia sẻ transaction giữa các request là thảm họa dữ liệu. Hiểu scope của container trước khi dùng nó.
- **Hai container trong một app** (thừa kế sau merge/migration): hai nguồn sự thật về graph — mọi bug wiring thành trò chơi "container nào tạo cái này?".

---

*Tiếp theo: [4.12 — Repository, Unit of Work & Specification](/series/software-design/level-4-patterns/12-repository-uow-specification/)*
