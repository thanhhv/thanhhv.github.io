+++
title = "5.3 — Production Case Studies: đọc pattern trong code thật đang chạy"
date = "2026-07-17T12:50:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

Chương này là bài tập tổng hợp của toàn bộ tài liệu: soi các hệ thống lớn — mỗi hệ vài đoạn, theo cùng khung ba câu hỏi: **họ chọn pattern gì, vì sao (đánh đổi cái gì), và nếu chọn khác thì hệ quả ra sao**. Nhiều case đã được nhắc rải rác ở các chương trước — ở đây gom lại thành chân dung trọn vẹn.

---

## A. Go Standard Library — bậc thầy của interface nhỏ

### `io.Reader` / `io.Writer`

**Chọn gì**: interface 1 method (ISP 2.4 cực hạn), hợp đồng ngữ nghĩa viết kỹ từng trường hợp biên (LSP 2.3), composition chồng vô hạn (Decorator 4.5: `bufio` bọc `gzip` bọc `net.Conn`).
**Vì sao**: số implementation là *vô hạn và không biết trước* — hợp đồng phải tối thiểu để dễ thỏa mãn nhất có thể; đổi lại caller chịu khó (loop đọc cẩn thận) và stdlib bù bằng helper (`io.ReadAll`, `io.Copy` — kèm tối ưu sniff optional interface `WriterTo/ReaderFrom`, kỹ thuật 2.4 bước 3).
**Nếu chọn khác**: interface "tiện caller" kiểu `ReadAll() []byte` → mất streaming (buộc load hết vào RAM), mất composition — cả hệ sinh thái streaming của Go không tồn tại. Một quyết định 1 method, hệ quả 15 năm.

### `net/http`

Một package, gần đủ bộ pattern của tài liệu: `Handler` 1 method (ISP) → `HandlerFunc` function adapter (4.4) → middleware decorator chain (4.5) → `httptest` fake được toàn bộ nhờ coupling qua interface (1.3). Server gọi handler của bạn — inversion of control (2.5). Phía ngược: `http.Get` facade trên `Client`/`Transport` (4.4), `Transport` là connection-pool proxy (4.5).
**Đánh đổi tỉnh táo đáng học**: `Request`/`ResponseWriter` là mutable struct + interface *không* hoàn hảo (bẫy optional interface 4.5b, phải vá bằng `ResponseController`) — tác giả ưu tiên đơn giản và hiệu năng hơn thuần khiết; vết sẹo được vá bằng API bổ sung thay vì breaking change. Bài học: **API public sống lâu vá bằng cách thêm, không bằng cách sửa** (OCP 2.2 với chính mình).

### `context.Context`

Immutable tree (1.5 — `WithTimeout` trả context mới bọc cha), observer qua channel `Done()` (4.8), lan truyền cross-cutting qua quy ước tham-số-đầu (1.3 — chọn tường minh, trả giá mọi chữ ký có `ctx`).
**Nếu chọn khác**: goroutine-local storage kiểu `AsyncLocalStorage` của Node → gọn chữ ký nhưng dependency tàng hình (1.3 đã so). Go chọn *nhìn thấy được* — nhất quán với toàn bộ khí chất ngôn ngữ. Vết gợn có thật: `ctx.Value` là túi không type-safe — nên mới có kỷ luật 4.5a.

---

## B. Frameworks & DI

### Gin / Echo

Middleware imperative `c.Next()` (4.5 đã so đủ với chuẩn `func(Handler) Handler`: thêm quyền chạy-hai-chiều và abort, trả giá lock-in). Điểm mổ thêm — `gin.Context` là **context-object gánh mọi thứ** (request, response, param, validate, render, key-value): tiện tối đa cho CRUD nhanh, nhưng chính là Service-Locator-trá-hình (4.11) khi handler móc dependency từ `c.MustGet("db")` — và handler test được phải dựng cả engine.
**Nếu chọn khác**: Echo tách `echo.Context` interface (mock được hơn); Chi trung thành stdlib (portable tối đa, ít "pin" nhất). Ba mức trên cùng một trục đánh đổi tiện-nay vs tự-do-mai.

### Uber Fx (đối chiếu NestJS)

Đã mổ trọn ở 4.11 (runtime DI container: mua chuẩn hóa nghìn-service + lifecycle, trả bằng lỗi-runtime + luồng tàng hình; NestJS: container là bản sắc, mạnh trong khuôn). Điểm case-study bổ sung: Fx ra đời *trong* Uber — nơi vấn đề thật là "nghìn service, mỗi service một kiểu bootstrap"; **pattern sinh ra từ quy mô tổ chức, không phải quy mô kỹ thuật**. Import Fx vào startup 5 service là import lời giải của bài toán mình không có.

---

## C. Data layer

### GORM vs Ent vs sqlc — ba triết lý cho một việc

- **GORM**: ActiveRecord-style, reflection + struct tag, chainable clause builder (4.2). Mua: khởi động nhanh nhất, CRUD 5 phút. Trả: hành vi ngầm nhiều (auto-preload, hook, zero-value-bỏ-qua-khi-update — nguồn bug kinh điển `Update` không set field false), lỗi phát hiện lúc runtime, dễ N+1. Đây là "easy nhưng không simple" (2.6 KISS) bằng xương thịt.
- **Ent**: schema-as-code → **codegen** ra API type-safe + query builder đồ thị. Mua: compile-time check, graph traversal đẹp, không reflection lúc chạy. Trả: buộc vào vòng đời codegen, schema DSL riêng phải học, code sinh khổng lồ. Cùng họ tư duy với Wire (4.11): **trả phức tạp lúc build để mua an toàn lúc chạy** — khí chất Go.
- **sqlc**: SQL thật → codegen hàm typed (4.12 đã khen). Mua: SQL là SQL (không học DSL, EXPLAIN được, DBA đọc được), type-safe, đơn giản nhất về khái niệm. Trả: query động khó, tự viết phần mapping quan hệ.
**Khung chọn**: đội mạnh SQL + nghiệp vụ dày → sqlc (+ repository 4.12 khi cần); đồ thị quan hệ phức tạp → Ent; prototype/CRUD tốc độ → GORM với kỷ luật. Không có "ORM tốt nhất" — có trade-off hợp đội.

### go-redis & database/sql

`database/sql` đã thành nhân vật chính bốn lần (registry 4.1, facade 4.4, bridge 4.6, LSP-tối-thiểu 2.3) — tự nó là một giáo trình. go-redis bổ sung một góc: client là **connection-pool proxy** (4.5) + options struct (4.2), và `Pipeline()`/`TxPipeline()` là **Command pattern hiện hình** (4.9): gom lệnh thành batch — lời gọi thành dữ liệu để gửi một chuyến. Kafka client (sarama vs franz-go) đã so ở 4.4: cùng domain, khác độ dày facade.

---

## D. Hạ tầng lớn — pattern ở quy mô hệ phân tán

### Kubernetes controller

Tổng hợp đỉnh cao của nửa cuối tài liệu: watch/informer = Observer bền (4.8, kèm bài học level-based vs edge-based — *thiết kế cho sự kiện bị mất* thay vì tin delivery); reconcile loop = mọi controller chỉ coupling với API server qua desired state (1.3 — coupling qua dữ liệu trung tâm); thêm controller không sửa controller nào (OCP 2.2 ở tầm hệ thống); CRD + controller = plugin qua data + process (4.13B); `DeepCopy` codegen = Prototype bắt buộc vì shared cache (4.3).
**Nếu chọn khác**: điều phối bằng lời gọi trực tiếp giữa các controller → lưới n² (4.9 Mediator đã vẽ), thêm một controller là sửa nhiều controller — không thể có hệ sinh thái operator như hôm nay. Kiến trúc "ai cũng chỉ nói chuyện với desired state" chính là lý do Kubernetes *mở rộng được bởi người lạ*.

### Docker & Terraform

- **Docker/containerd**: kiến trúc tách tầng qua hợp đồng chuẩn — CLI → daemon (REST) → containerd (gRPC) → runc (OCI spec). Mỗi ranh giới một facade + hợp đồng versioned; nhờ đó từng tầng thay được (Kubernetes bỏ dockershim dùng thẳng containerd — *vì* ranh giới sạch). Bài học DIP 2.5 ở tầm ngành: **hợp đồng chuẩn (OCI) là "interface do consumer-liên-minh định nghĩa"** — không vendor nào sở hữu.
- **Terraform provider**: plugin-qua-process-RPC (4.13B đã đặt tên) — provider là binary riêng, versioned protocol. Trả giá đã hứa ở 2.2: hợp đồng core↔plugin phải tương thích *vĩnh viễn*, đổi major protocol là chiến dịch di cư cả hệ sinh thái. Đổi lại: nghìn provider viết bởi người lạ, core không đổi một dòng — OCP đắt nhất và đáng nhất trong danh sách này.

---

## E. Tổng kết chương — ba quy luật rút từ mọi case

1. **Interface càng public, càng nhỏ và càng bất biến** — `io.Reader` 1 method sống 15 năm; hợp đồng Terraform versioned vĩnh viễn. Chi phí sửa hợp đồng public tỉ lệ với số người lạ đứng trên nó (2.2, 2.3).
2. **Tường minh thắng tiện lợi ở hệ sống lâu** — context tham-số-đầu, sqlc/Ent codegen, wiring tay/Wire: các lựa chọn "gõ nhiều hơn hôm nay" của hệ Go đều mua cùng một thứ: *đọc được, đoán được, fail sớm*. Các lựa chọn tiện (GORM ngầm, gin.Context, Fx reflection) đều trả bằng bất ngờ lúc chạy — hợp lý ở đời ngắn, đắt ở đời dài.
3. **Pattern lớn sinh từ bài toán tổ chức, không từ sách** — Fx từ nghìn service của Uber, informer từ scale của Google, OCI từ chiến tranh vendor. Trước khi mượn giải pháp, hỏi: *mình có bài toán của họ không?* — câu hỏi đã theo bạn từ chương 1.1, và là câu hỏi cuối cùng của mọi design review.

---

*Tiếp theo: [5.4 — Tổng kết toàn bộ tài liệu](/series/software-design/level-5-architecture/04-tong-ket/)*
