+++
title = "Bài 7 — Software Architecture với Go"
date = "2026-06-15T08:00:00+07:00"
draft = false
tags = ["backend", "golang"]
series = ["NodeJS & Golang"]
+++

# Software Architecture với Go — DI, Clean/Hexagonal Architecture, DDD, Event-driven

---

## 1. Problem Statement

Code Go dễ viết — nhưng codebase Go 300.000 dòng sau 3 năm với 20 engineer thì không. Các vấn đề kiến trúc không xuất hiện ở tháng đầu, chúng xuất hiện khi:

- Đổi Postgres sang DynamoDB phải sửa 200 file — vì logic nghiệp vụ gọi thẳng SQL.
- Không unit test được vì mọi hàm đều chạm DB thật.
- Hai team sửa cùng một package "common" và block lẫn nhau hàng tuần.

Kiến trúc là câu trả lời cho câu hỏi: **thay đổi nào sẽ đến, và ta trả giá bao nhiêu khi nó đến?** Mọi pattern dưới đây đều là biến thể của một nguyên lý duy nhất: **tách cái thay đổi thường xuyên (chi tiết hạ tầng) khỏi cái ổn định (logic nghiệp vụ), qua ranh giới là interface.**

---

## 2. Dependency Injection trong Go

### 2.1. Vấn đề

```go
// KHÔNG DI: tự tạo dependency bên trong
func NewOrderService() *OrderService {
    db := postgres.Connect(os.Getenv("DB_URL")) // hard-wired
    return &OrderService{db: db}
}
```

Hệ quả: không test được nếu thiếu Postgres thật; không thay được implementation; thứ tự khởi tạo ẩn; config rải rác khắp nơi.

### 2.2. Cách của Go: interface ở phía người dùng + constructor injection

```go
// Interface khai báo Ở PACKAGE SỬ DỤNG (consumer), không phải package hiện thực
// — đặc sản Go: implicit interface satisfaction cho phép điều này
package order

type Repository interface {          // chỉ khai những method order CẦN
    Save(ctx context.Context, o Order) error
    FindByID(ctx context.Context, id string) (Order, error)
}

type Service struct {
    repo   Repository
    events EventPublisher
}

func NewService(repo Repository, events EventPublisher) *Service {
    return &Service{repo: repo, events: events}
}
```

Vì Go có **implicit interface** (không cần `implements`), package `postgres` không cần biết package `order` tồn tại — dependency chỉ chảy một chiều từ hạ tầng vào domain. Đây là Dependency Inversion đạt được bằng tính năng ngôn ngữ, không cần framework.

Quy tắc vàng Go: **"Accept interfaces, return structs"** và **interface nhỏ** (1-3 method). Interface 15 method là code smell — nó mô tả implementation chứ không mô tả nhu cầu.

### 2.3. DI framework — có cần không?

| Cách | Ưu | Nhược |
|---|---|---|
| Wire thủ công trong `main()` | Tường minh, compiler check, dễ debug | `main()` dài khi app lớn (nhưng chỉ dài, không phức tạp) |
| google/wire (codegen) | Tự sinh code wiring, vẫn compile-time | Thêm bước codegen, learning curve |
| uber/fx (runtime, reflection) | Mạnh, lifecycle hooks | Lỗi wiring lộ lúc runtime; magic — đi ngược triết lý Go |

Khuyến nghị thực dụng: **wire thủ công đến khi `main()` vượt ~200 dòng khởi tạo**, rồi cân nhắc `wire`. Cộng đồng Go nghiêng mạnh về tường minh — một `main()` dài 150 dòng đọc từ trên xuống dưới vẫn dễ hiểu hơn container ma thuật. Đây là khác biệt văn hóa lớn với Java/Node (NestJS).

---

## 3. Clean Architecture / Hexagonal (Ports & Adapters)

### 3.1. Hai tên gọi, một ý tưởng

Cả hai đều nói: **domain ở trung tâm, hạ tầng ở rìa, dependency chỉ hướng vào trong.**

```
            ┌──────────────────────────────────────┐
  HTTP ───► │ Adapter (handler)                    │
  gRPC ───► │   ┌──────────────────────────────┐   │
  Kafka ──► │   │ Application (use case)       │   │
            │   │   ┌──────────────────────┐   │   │
            │   │   │ Domain               │   │   │
            │   │   │ (entity, rule thuần) │   │   │
            │   │   └──────────────────────┘   │   │
            │   │  gọi ra ngoài qua PORT       │   │
            │   │  (interface do domain sở hữu)│   │
            │   └──────────────────────────────┘   │
            │ Adapter (Postgres, Redis, SMTP) ─────┼──► hạ tầng thật
            └──────────────────────────────────────┘
```

### 3.2. Cấu trúc thư mục Go thực dụng

```
/cmd/api/main.go          ← wiring, khởi động
/internal/
   order/                 ← theo DOMAIN, không theo tầng kỹ thuật!
      service.go          ← use case + port (interface)
      order.go            ← entity, rule
      postgres.go         ← adapter DB (hoặc /internal/order/postgres/)
      http.go             ← adapter HTTP
   payment/
      ...
```

Lưu ý quan trọng: tổ chức **theo domain** (`order/`, `payment/`) chứ không theo tầng (`controllers/`, `services/`, `repositories/`). Tổ chức theo tầng khiến một thay đổi nghiệp vụ chạm 4 thư mục, và mọi thứ import mọi thứ. Tổ chức theo domain + `internal/` cho ranh giới compiler-enforced — chuẩn bị sẵn cho việc tách microservice sau này nếu cần.

### 3.3. Liều lượng — lời khuyên principal-level

Clean Architecture có giá: thêm interface, thêm mapping struct giữa các tầng, thêm file. Với CRUD service 2.000 dòng, full Clean Architecture là **over-engineering** — handler gọi thẳng repository là đủ và ai cũng hiểu. Ngưỡng đáng đầu tư: logic nghiệp vụ thật sự tồn tại (không chỉ CRUD), nhiều adapter cho cùng port (test + thật, hoặc đa nhà cung cấp), đội > 5 người cần ranh giới rõ.

**Điều xảy ra nếu làm ngược lại (dùng full ceremony cho service bé):** 5 file để thêm một field; abstraction rỗng (interface có đúng 1 implementation vĩnh viễn và không bao giờ được mock); đội mới vào mất một tuần tìm chỗ code chạy. Kiến trúc tốt nhất là kiến trúc **tương xứng độ phức tạp của bài toán**.

---

## 4. Domain-Driven Design (DDD) — phần đáng giá nhất

DDD có hai nửa. Nửa **chiến thuật** (entity, value object, aggregate, repository) hay bị sùng bái quá mức. Nửa **chiến lược** mới là vàng:

- **Bounded Context:** cùng một từ "Order" mang nghĩa khác nhau ở team Bán hàng (giỏ hàng, giá) và team Vận chuyển (kiện hàng, địa chỉ). Đừng ép một model dùng chung — đó là nguồn gốc của god-object `Order` 80 field. Mỗi context một model riêng, dịch qua lại ở ranh giới (anti-corruption layer).
- **Ubiquitous Language:** code dùng đúng từ mà business dùng. Hàm tên `SubmitOrder` chứ không `ProcessData`. Nghe tầm thường; ở codebase lớn, đây là khác biệt giữa đọc-hiểu-ngay và khảo-cổ-học.
- **Aggregate = ranh giới transaction:** một aggregate (Order + OrderLines) được load và save **nguyên khối, trong một transaction**; giữa các aggregate chỉ có eventual consistency qua event. Quy tắc này trả lời câu hỏi muôn thuở "transaction nên bọc tới đâu".

Bounded context chính là đường kẻ tự nhiên để chia module/microservice — chia theo context thì service ít phải gọi nhau; chia sai (theo tầng kỹ thuật, theo bảng DB) thì mọi request nhảy qua 5 service.

---

## 5. Event-Driven Architecture

### 5.1. Vấn đề của gọi đồng bộ

Đặt hàng xong phải: trừ kho, gửi email, cộng điểm, báo analytics. Gọi đồng bộ 4 service: latency cộng dồn, đặt hàng fail vì... service email chết, và team Order phải biết mọi consumer tương lai (coupling ngược).

### 5.2. Đảo ngược: publish sự kiện

```
OrderService ──"OrderCreated"──► Kafka/NATS ──► InventoryService
                                            ──► EmailService
                                            ──► LoyaltyService
```

Producer không biết consumer. Thêm consumer mới = zero thay đổi phía Order. Email chết 10 phút → message chờ trong queue, xử lý bù sau — lỗi được **cô lập theo thời gian**.

### 5.3. Cái giá phải trả (phần người ta hay giấu)

1. **Dual-write problem:** ghi DB xong, publish Kafka fail (hoặc ngược lại) → hai nguồn sự thật lệch nhau. Giải chuẩn: **Transactional Outbox** — ghi event vào bảng `outbox` *cùng transaction* với nghiệp vụ; một relay (hoặc CDC/Debezium) đọc outbox đẩy sang broker. Không có outbox, hệ thống event-driven của bạn mất event một cách im lặng.
2. **At-least-once → consumer phải idempotent** (chương 6). Không thương lượng.
3. **Debug khó hơn hẳn:** flow xuyên 5 service qua broker — không có stack trace nào nối chúng. Bắt buộc có distributed tracing (trace ID truyền trong message header) từ ngày đầu.
4. **Eventual consistency lộ ra UI/nghiệp vụ:** đặt hàng xong, trang kho chưa trừ trong 2 giây. Phải thiết kế UX và nghiệp vụ chấp nhận điều đó — quyết định sản phẩm, không chỉ kỹ thuật.
5. **Schema của event là API công khai:** đổi field = breaking change cho consumer không biết mặt. Cần schema registry / versioning kỷ luật.

**Khi nào đừng dùng:** nghiệp vụ cần kết quả ngay trong request (thanh toán cần biết thành/bại — phần đó phải đồng bộ); hệ thống nhỏ một team (broker + outbox + tracing là thuế hạ tầng nặng); và đừng dùng event để **hỏi dữ liệu** (request/reply qua queue là dùng sai công cụ — cái đó là RPC).

---

## 6. Error handling như một phần của kiến trúc

Error trong Go là value → chiến lược lỗi là quyết định thiết kế, thống nhất toàn codebase:

```go
// Wrap có ngữ cảnh tại MỖI ranh giới tầng — một lần, không hơn
if err := s.repo.Save(ctx, order); err != nil {
    return fmt.Errorf("save order %s: %w", order.ID, err)
}
// Domain định nghĩa sentinel/type lỗi; adapter DỊCH lỗi hạ tầng sang lỗi domain
var ErrNotFound = errors.New("order not found")     // domain
// postgres adapter: if errors.Is(err, sql.ErrNoRows) { return ErrNotFound }
// handler map lỗi domain → HTTP status ở MỘT chỗ duy nhất
```

Ba quy tắc: **wrap một lần mỗi tầng** (wrap trùng lặp tạo lỗi dài dòng vô nghĩa: "failed to X: failed to X: ..."); **log hoặc return, không cả hai** (log-và-return tạo log trùng 4 lần cho một lỗi); **domain không leak lỗi hạ tầng** (handler mà `errors.Is(err, sql.ErrNoRows)` là kiến trúc đã thủng).

---

## 7. Best Practices tổng hợp chương

1. Interface nhỏ, khai báo phía consumer, chỉ khi có ≥2 implementation hoặc cần mock.
2. Tổ chức code theo domain, dùng `internal/` enforce ranh giới.
3. Wire dependency thủ công trong `main()` trước; framework sau, nếu cần.
4. Aggregate = ranh giới transaction; giữa aggregate dùng event + outbox.
5. Độ ceremony của kiến trúc tương xứng độ phức tạp nghiệp vụ — và được nâng cấp dần, không big-bang.
6. Chiến lược error thống nhất, viết thành convention trong repo.

## 8. Anti-patterns

1. **Interface mọi thứ ngay từ đầu** — "phòng khi cần" là YAGNI; interface 1-impl không mock là thuế đọc code thuần túy.
2. **Package `models` toàn cục** chứa struct của mọi domain → mọi thứ phụ thuộc mọi thứ.
3. **Repository generic `Save(interface{})`** — mất type safety để được gì không rõ.
4. **Event chứa cả object khổng lồ** thay vì fact gọn ("OrderCreated{ID, Total}") → coupling schema nặng nề.
5. **Microservice hóa theo bảng DB** (UserService, OrderService... mỗi cái một bảng CRUD) → distributed monolith: mọi tính năng chạm 4 service, chậm và mong manh hơn monolith gốc.

---

*Hết phần Golang. Chương tiếp theo: [08 — Node.js: Tại sao tồn tại, V8, Runtime Architecture](/series/nodejs-golang/08-node-nen-tang-v8/)*
