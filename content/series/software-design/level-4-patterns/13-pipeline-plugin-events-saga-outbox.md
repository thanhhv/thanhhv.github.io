+++
title = "4.13 — Pipeline, Plugin, Domain Events, Saga & Outbox — và bản đồ tổng kết Level 4"
date = "2026-07-17T12:10:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

Chương khép Level 4: hai idiom kiến trúc nhỏ (Pipeline, Plugin — phần lớn đã xây ở các chương trước, giờ chốt sổ), rồi bộ ba pattern của **thế giới phân tán** (Domain Events → Saga → Outbox — ba nấc của cùng một câu chuyện), và bảng phân loại cuối cùng: đâu là GoF, đâu là Enterprise, đâu là idiom Go.

---

## A. Pipeline — chốt sổ

Đã xây ở 3.4 (Replace Loop with Strategy → tách khung khỏi rule) và 4.5 (middleware = pipeline của handler). Định nghĩa chốt: **chuỗi giai đoạn xử lý, output giai đoạn này là input giai đoạn sau, mỗi giai đoạn không biết gì về nhau ngoài hợp đồng dữ liệu**. Ba hình dạng trong Go, chọn theo tải:

```go
// (1) Chuỗi hàm — pipeline đồng bộ, đủ cho 90% nhu cầu xử lý nghiệp vụ
func Process(r RawOrder) (Order, error)  →  Validate → Normalize → Enrich → Price

// (2) iter.Seq — pipeline lazy trên stream dữ liệu lớn (4.10): không load hết vào RAM

// (3) Channel + goroutine — pipeline CONCURRENT khi các giai đoạn tốn thời gian khác nhau:
//     stage1 → chan → stage2(×4 worker) → chan → stage3
//     Đặc sản Go; trả giá: dọn dẹp khi lỗi giữa chừng (context cancel + drain) khó hơn
//     tuần tự RẤT nhiều — chỉ leo khi đo thấy cần song song thật
```

Bẫy duy nhất đáng ghi: **nghiện channel** — dựng concurrent pipeline cho thứ chạy 2ms tuần tự. Nhắc lại kim chỉ nam 3.4: đo, rồi chọn.

## B. Plugin — chốt sổ

Đã xây đủ ở 2.2/4.1 (registry — plugin *trong process, cùng binary*: driver `database/sql`, format exporter). Nấc còn lại là **plugin ngoài binary** khi bên thứ ba không thể/không được recompile vào binary của bạn:

- **Go `plugin` package (.so)**: hầu như chết trong thực tế — đòi cùng version Go, cùng dependency graph, không Windows. Tránh.
- **Process riêng + RPC**: cách Terraform provider, gRPC plugin của protoc, LSP server hoạt động — plugin là binary độc lập nói chuyện qua stdin/gRPC. Đắt hơn registry một bậc lớn (versioning hợp đồng, lifecycle process) nhưng cách ly hoàn toàn: plugin crash không sập host, viết bằng ngôn ngữ khác cũng được. Đây là "OCP đẩy tới giới hạn" đã nhắc ở 2.2 — giờ có tên đầy đủ.
- **WASM**: đường mới cho plugin sandbox (Envoy filter, Istio) — đáng theo dõi hơn là đáng dùng mặc định.

Thang plugin: **registry trong binary → process ngoài qua RPC → WASM** — leo một nấc khi ranh giới tin cậy/deploy thật sự đòi hỏi.

---

## C. Domain Events → Saga → Outbox: ba nấc của một câu chuyện phân tán

### Nấc 1 — Domain Events: sự kiện thành công dân của domain

4.8 dừng ở "event là dữ liệu bất biến, quá khứ". Domain Events (DDD) đi thêm một bước: sự kiện là **một phần của mô hình nghiệp vụ** — aggregate *ghi nhận* điều quan trọng vừa xảy ra, như một phần hành vi của nó:

```go
// Aggregate tự ghi sự kiện khi hành vi nghiệp vụ xảy ra — không publish gì cả
func (o *Order) Confirm() error {
    /* ... check invariant, đổi trạng thái (2.6 Tell Don't Ask) ... */
    o.record(OrderConfirmed{OrderID: o.id, Total: o.Total(), At: o.now()})
    return nil
}
func (o *Order) Events() []DomainEvent { return o.events }   // use case lấy ra sau khi Save
```

Khác biệt tinh tế mà quan trọng so với observer 4.8: aggregate **không gọi ai** — nó chỉ *ghi sổ*. Use case, sau khi `Save` thành công (trong UoW — 4.12), mới lấy events ra phát. Hai lợi ích: sự kiện sinh ra **cùng transaction với thay đổi dữ liệu** (không còn "confirm rồi mà chưa chắc đã phát event"), và danh sách sự kiện của domain trở thành **tài liệu nghiệp vụ đọc được** (`OrderConfirmed`, `PaymentCaptured`, `ShipmentDelayed` — ngôn ngữ chung với PM, mầm của event storming Level 5).

### Nấc 2 — Saga: transaction không sống qua network

Bài toán mà UoW (4.12) tuyên bố đầu hàng: quy trình xuyên **nhiều service** — đặt đơn = trừ kho (service A) + thu tiền (service B) + tạo vận đơn (service C). Không có transaction phân tán thực dụng (2PC chết vì blocking + coordinator là single point) → thay bằng **Saga**: chuỗi transaction cục bộ, mỗi bước có **hành động bù (compensation)** — thu tiền fail thì *hoàn kho* (không phải "rollback" — kho đã commit; là một transaction *mới* làm nghiệp vụ ngược, chính là ý "undo bằng hành động bù" đã hẹn ở 4.9 Command).

Hai kiểu điều phối — và đây là Mediator vs Observer tái hiện ở tầng phân tán (4.9/4.8):

| | **Orchestration** (có nhạc trưởng) | **Choreography** (domino sự kiện) |
|---|---|---|
| Cơ chế | Một orchestrator gọi từng bước, nhận kết quả, quyết bước sau/bù | Mỗi service nghe event của service trước, tự làm phần mình, phát event tiếp |
| Luồng đọc ở đâu | **Một chỗ** — code/state machine của orchestrator | **Không ở đâu cả** — tổng gộp ngầm của các subscription |
| Coupling | Services coupling vào orchestrator | Services coupling vào *schema event* của nhau — lỏng hơn về deploy, khó thấy hơn |
| Hợp với | Quy trình có thứ tự, có bù, cần theo dõi trạng thái ("đơn này đang kẹt bước nào?") | Phản ứng phụ đơn giản, ít bước, không cần bù chéo |
| Bệnh đặc trưng | Orchestrator phình thành God Object phân tán | Chuỗi domino tàng hình — đúng anti-pattern 4.8, phiên bản đắt nhất |

Lời khuyên thẳng: quy trình nghiệp vụ **có tiền và có bù** → orchestration (Temporal/Cadence tồn tại chính vì viết orchestrator tay — state, retry, timeout, versioning — là cả một hệ thống; các workflow engine này *là* Saga orchestration đóng gói). Choreography để cho các phản ứng phụ thật sự phụ.

### Nấc 3 — Outbox: bịt khe hở giữa DB và broker

Khe hở đã chỉ ở bảng 4.8: `Save(order)` commit vào Postgres, rồi `Publish(OrderConfirmed)` sang Kafka — **hai hệ, không cùng transaction**. Crash giữa hai lệnh: đơn đã confirm mà thế giới không bao giờ biết (hoặc ngược lại nếu publish trước). Với Saga/event-driven, khe hở này không phải góc cạnh — nó là lỗ thủng đắm thuyền.

**Outbox pattern** — bịt bằng chính transaction DB:

```
1. Trong CÙNG transaction với thay đổi nghiệp vụ:
     INSERT INTO orders ...;
     INSERT INTO outbox (event_type, payload, created_at) VALUES ('OrderConfirmed', {...});
   → commit nguyên tử: dữ liệu đổi ⟺ event được ghi. Không khe hở.

2. Một relay riêng đọc bảng outbox → publish sang broker → đánh dấu đã gửi
   (polling đơn giản, hoặc CDC/Debezium đọc WAL cho tải lớn)
```

Hệ quả phải chấp nhận và thiết kế theo: relay có thể gửi trùng (crash sau publish trước khi đánh dấu) → **at-least-once, consumer phải idempotent** — đúng nghĩa vụ đã ghi ở 4.9 Command-đi-xa; thứ tự chỉ tương đối → event mang `sequence`/version của aggregate. Outbox + idempotent consumer là **cặp nền móng** của mọi kiến trúc event-driven nghiêm túc — Level 5 đứng trên nó.

Chuỗi ba nấc gói lại: **Domain Events** cho sự kiện sinh đúng chỗ (trong domain, cùng transaction), **Outbox** cho sự kiện rời process an toàn, **Saga** cho quy trình nhiều bước dùng sự kiện đó mà vẫn có đường lùi. Ba mảnh của một máy — thiếu mảnh giữa (Outbox) thì hai mảnh kia là máy đẹp chạy trên cát.

---

## D. Bản đồ tổng kết Level 4 — GoF / Enterprise / Go idiom

Bảng phân loại đã hứa từ đầu tài liệu — mọi pattern đã đi qua, xếp theo xuất xứ và tình trạng:

| Xuất xứ | Pattern | Tình trạng trong Go/TS backend 2026 |
|---|---|---|
| **GoF (1994)** | Factory, Builder, Adapter, Facade, Decorator, Proxy, Composite, Strategy, State, Observer, Command, CoR | Sống khỏe — hình dạng đã idiom hóa theo ngôn ngữ (4.1–4.9) |
| | Singleton, Prototype, Bridge, Mediator, Template Method, Iterator, Visitor, Memento, Flyweight | Tan vào ngôn ngữ / thu hẹp ngách / thành nhãn chẩn đoán (4.3, 4.6, 4.9, 4.10) |
| **Enterprise (Fowler 2002, Evans 2003)** | Repository, Unit of Work, Specification, Domain Events, Service Layer (= use case/Facade) | Nền của kiến trúc backend có domain dày; cargo-cult khi domain mỏng (4.12) |
| **Phân tán (2010s+)** | Saga, Outbox, Idempotent Consumer, Circuit Breaker, CQRS | Bắt buộc *khi và chỉ khi* bước qua ranh giới process/service (4.13, Level 5) |
| **Go idiom** | Functional Options, `func(Handler) Handler` middleware, interface consumer-side nhỏ, `iter.Seq`, channel pipeline, errgroup | Không phải "pattern GoF viết bằng Go" — là sản phẩm của chính các ràng buộc và năng lực của Go (4.2, 4.5, 4.10) |

Và **bản đồ quyết định cuối cùng của Level 4** — mọi pattern quy về một trong bốn câu hỏi gốc:

```
"Tạo object này phức tạp/biến thiên?"        → Creational  (thang 4.3 tổng kết)
"Đặt type nào cạnh type nào, để làm gì?"     → Structural  (bảng 4 pattern "bọc", 4.5)
"Ai quyết định việc gì, lúc chạy?"           → Behavioral  (bảng số phận, 4.10)
"Dữ liệu/sự kiện vượt ranh giới nào?"        → Enterprise & phân tán (4.12–4.13)
```

Nếu một pattern không trả lời câu hỏi nào bạn đang thực sự có — nó không phải công cụ, nó là trang sức. Đó là câu tổng kết của toàn Level 4, và là tấm vé vào Level 5: kiến trúc chính là các câu hỏi trên, hỏi ở quy mô toàn hệ thống.

---

*Tiếp theo: Level 5 — Production Architecture: DDD, Clean/Hexagonal, Event-driven, CQRS và các case study (phần cuối của bộ tài liệu).*
