+++
title = "Chương 02 – Communication: Các node nói chuyện với nhau như thế nào"
date = "2026-07-08T18:00:00+07:00"
draft = false
tags = ["backend", "distributed-systems"]
series = ["Distributed Systems"]
+++

## 1. Problem Statement

Trên một máy, module A gọi module B bằng một function call: chi phí ~nanosecond, không bao giờ "mất", không bao giờ trả về hai lần, và nếu B crash thì A cũng crash cùng (trạng thái nhất quán). Khi B chuyển sang máy khác, function call trở thành **remote call**: chậm hơn ~10⁶ lần, có thể mất, có thể timeout khi đã thành công, và B chết không kéo A chết theo — nghe như ưu điểm, nhưng nghĩa là A phải *tự quyết định* làm gì khi không biết B sống hay chết.

Nếu không có lớp communication tử tế: mỗi service tự viết logic kết nối, serialize, retry — trùng lặp, không nhất quán, và sai ở những chỗ hiểm (retry non-idempotent call). Câu hỏi của chương này: **đồng bộ hay bất đồng bộ, gọi trực tiếp hay qua trung gian, và tìm nhau bằng cách nào.**

## 2. Tại sao các mô hình giao tiếp khác nhau tồn tại

Vì có hai loại quan hệ nghiệp vụ khác nhau về bản chất:

- **"Tôi cần câu trả lời NGAY để đi tiếp"** → Request/Response đồng bộ (REST, gRPC). Ví dụ: kiểm tra tồn kho trước khi cho đặt hàng.
- **"Tôi thông báo việc đã xảy ra, ai quan tâm thì xử lý"** → Messaging bất đồng bộ (queue, event streaming). Ví dụ: đơn hàng đã tạo → gửi email, cập nhật analytics, cộng điểm loyalty.

Dùng nhầm bên là nguồn gốc của phần lớn kiến trúc tồi: mọi thứ đồng bộ → chuỗi phụ thuộc dài, availability nhân nhau, latency cộng dồn; mọi thứ bất đồng bộ → không trả lời được câu hỏi cần trả lời ngay, độ phức tạp eventual consistency lan khắp nơi.

## 3. Request/Response: RPC, REST, gRPC

### 3.1. Bản chất RPC và bài toán "giả vờ là local call"

RPC (Remote Procedure Call) cố làm cho remote call *trông như* local call: `inventory.check(sku)` thay vì tự mở socket. Tiện, nhưng nguy hiểm nếu abstraction che mất sự thật: **remote call có thể fail theo cách local call không bao giờ fail**. Thế hệ RPC hiện đại (gRPC, Thrift) không che nữa — chúng expose deadline, status code, retry policy như first-class concept. Bài học lịch sử: CORBA/DCOM thất bại một phần vì giả vờ quá giỏi.

### 3.2. REST vs gRPC

**REST**: resource + HTTP verb + JSON, thường HTTP/1.1. **gRPC**: contract-first với Protobuf, HTTP/2 (multiplexing, header compression, streaming 2 chiều).

Vì sao gRPC nhanh hơn — nguyên nhân kỹ thuật, không phải marketing:

1. **Protobuf là binary, có schema**: field được đánh số, không lặp lại tên field trong từng message như JSON. Payload nhỏ hơn 3–10 lần, parse không cần tokenize chuỗi — ít CPU hơn hẳn.
2. **HTTP/2 multiplexing**: nhiều request song song trên một TCP connection, không head-of-line blocking ở tầng HTTP (vẫn còn ở tầng TCP), không tốn handshake mở connection mới.
3. **Streaming**: server-streaming, client-streaming, bidirectional — REST phải giả lập bằng polling/SSE/WebSocket.

| Tiêu chí | REST/JSON | gRPC/Protobuf |
|---|---|---|
| Human-readable, debug bằng curl | ✅ | ❌ (cần grpcurl, reflection) |
| Browser gọi trực tiếp | ✅ | ❌ (cần gRPC-Web/proxy) |
| Hiệu năng serialize + băng thông | Trung bình | Cao (3–10x nhỏ hơn) |
| Contract & codegen | OpenAPI (thường viết sau, lệch với code) | Proto là source of truth, codegen đa ngôn ngữ |
| Schema evolution | Tự do (nguy hiểm) | Quy tắc rõ ràng (field number không tái sử dụng) |
| Streaming | Hạn chế | Native, 2 chiều |
| Hệ sinh thái public API | Chuẩn de facto | Hiếm dùng public |
| Bối cảnh phù hợp | Public API, CRUD, team đa dạng trình độ | Internal service-to-service, latency-sensitive, polyglot |

**Khuyến nghị production phổ biến**: REST cho edge (public API), gRPC cho internal east-west traffic. Đừng migrate REST→gRPC chỉ để "hiện đại" — nếu bottleneck của bạn là DB query 50ms thì tiết kiệm 2ms serialize là vô nghĩa.

### 3.3. Failure semantics — phần quan trọng nhất mà tutorial bỏ qua

Mọi remote call có 3 kết cục: thành công, thất bại rõ ràng (connection refused), và **không biết** (timeout). Từ đó sinh ra 3 mức đảm bảo delivery:

- **At-most-once**: gửi 1 lần, không retry → có thể mất.
- **At-least-once**: retry đến khi có ack → có thể trùng.
- **Exactly-once**: không tồn tại trên mạng không đáng tin cậy ở tầng vận chuyển. Chỉ đạt được *hiệu ứng* exactly-once = at-least-once + **idempotent receiver** (chương 08).

Nếu API của bạn có thao tác ghi và caller có retry (mà chắc chắn có), API của bạn **phải** idempotent. Không thương lượng.

## 4. Messaging: Message Queue và Event Streaming

### 4.1. Vì sao cần trung gian (broker)

Gọi trực tiếp buộc hai bên **cùng sống, cùng lúc, cùng tốc độ**. Broker phá cả ba ràng buộc:

```
Producer ──▶ [ Broker (bền vững) ] ──▶ Consumer
   │                                      │
   └── không cần biết consumer là ai,     └── xử lý theo tốc độ của mình,
       đang sống hay chết                     chết thì message chờ sẵn
```

- **Decoupling thời gian**: consumer chết 10 phút, message không mất.
- **Decoupling tốc độ (load leveling)**: đỉnh 50K msg/s, consumer xử lý đều 10K msg/s, queue hấp thụ phần chênh — thay vì scale consumer cho đỉnh.
- **Decoupling không gian**: thêm consumer mới (analytics) không cần sửa producer.

Giá phải trả: thêm một hệ thống phải vận hành (và broker trở thành critical infrastructure), độ trễ end-to-end tăng, eventual consistency, và **bắt buộc xử lý message trùng lặp**.

### 4.2. Message Queue (RabbitMQ) vs Event Streaming (Kafka) — khác nhau ở bản chất, không ở tốc độ

**Queue (RabbitMQ, SQS)**: message là **lệnh việc cần làm**. Consumer nhận, xử lý, ack → message *bị xóa*. Smart broker (routing phức tạp, per-message ack, priority, delay), dumb consumer.

**Log (Kafka, Pulsar)**: message là **sự kiện đã xảy ra**, ghi vào log chỉ-thêm (append-only), *không xóa khi đọc* (xóa theo retention). Consumer tự giữ offset — dumb broker, smart consumer. Nhiều consumer group đọc độc lập cùng một log, mỗi group một tiến độ riêng.

```
Kafka topic (partition 0):  [e1][e2][e3][e4][e5][e6][e7] ──▶ append
                                      ▲           ▲
                              group A offset=3    group B offset=6
```

Vì sao Kafka đạt hàng triệu msg/s trên phần cứng thường — cơ chế bên trong:

1. **Sequential IO**: append vào cuối file. Disk tuần tự nhanh hơn random hàng trăm lần (kể cả SSD, do bỏ qua được nhiều tầng).
2. **Zero-copy** (`sendfile`): dữ liệu đi thẳng từ page cache ra socket, không copy qua user space.
3. **Batching + compression** ở cả producer và consumer.
4. **Không track trạng thái từng message**: broker chỉ cần biết offset của consumer group — O(1) metadata thay vì O(n) message state.

| Tiêu chí | RabbitMQ (queue) | Kafka (log) |
|---|---|---|
| Ngữ nghĩa | Task cần làm, xử lý xong biến mất | Sự kiện đã xảy ra, giữ lại đọc lại được |
| Replay | ❌ | ✅ (reset offset — cực quý khi consumer có bug) |
| Nhiều consumer độc lập cùng dữ liệu | Phải fanout exchange (copy) | Native (consumer group) |
| Routing phức tạp, per-message TTL/priority | ✅ | ❌ |
| Ordering | Per-queue, dễ vỡ khi có nhiều consumer | Chặt chẽ **trong một partition** |
| Throughput điển hình | Chục nghìn msg/s/node | Trăm nghìn – triệu msg/s/node |
| Độ phức tạp vận hành | Thấp–trung bình | Trung bình–cao (dù KRaft đã bỏ ZooKeeper) |
| Phù hợp | Task queue, RPC-over-queue, routing nghiệp vụ | Event backbone, stream processing, data pipeline, event sourcing |

Chọn sai hướng nào tệ hơn? Dùng Kafka làm task queue đơn giản = trả chi phí vận hành khổng lồ cho tính năng không dùng. Dùng RabbitMQ làm event backbone cho 20 team = nghẽn fanout và không replay được. **Ordering trong Kafka chỉ có trong một partition** — chọn partition key sai (không theo entity cần thứ tự, ví dụ `order_id`) là bug kinh điển: sự kiện `OrderPaid` đến trước `OrderCreated`.

## 5. Service Discovery

### 5.1. Vấn đề

Trong môi trường autoscaling/Kubernetes, IP của service instance đổi liên tục. Hardcode IP chết ngay; DNS truyền thống có TTL cache — trỏ vào instance đã chết hàng chục giây.

### 5.2. Hai mô hình

```
Client-side discovery:                Server-side discovery:
Client ──▶ Registry (Consul/etcd)     Client ──▶ Load Balancer/Proxy ──▶ instance
   │  danh sách instance                            │
   └──▶ tự chọn instance, tự LB                     └──▶ Registry
```

- **Client-side** (Netflix Eureka + Ribbon): client tự lấy danh sách, tự load-balance. Ít hop, kiểm soát tốt, nhưng logic discovery lặp lại trong mọi ngôn ngữ/framework.
- **Server-side** (Kubernetes Service, AWS ALB, service mesh sidecar): client chỉ biết một tên ổn định, hạ tầng lo phần còn lại. Đơn giản cho app, thêm một hop, LB thành điểm cần HA.

Xu hướng hiện tại: đẩy xuống hạ tầng (Kubernetes DNS + kube-proxy, hoặc service mesh như Istio/Linkerd — sidecar/ambient proxy lo discovery, retry, mTLS, circuit breaking). Trade-off của mesh: mua tính năng đồng nhất đa ngôn ngữ bằng độ phức tạp vận hành đáng kể và ~0.5–2ms latency mỗi hop.

**Health check quyết định chất lượng discovery**: liveness (process sống?) khác readiness (sẵn sàng nhận traffic?). Sai lầm kinh điển: readiness check gọi cả DB → DB chậm 1 chút → toàn bộ pod bị rút khỏi LB → outage toàn phần từ sự cố cục bộ.

## 6. API Gateway

Một cửa vào duy nhất cho client bên ngoài: routing, authentication/authorization, rate limiting, TLS termination, request transformation, đôi khi aggregation (BFF — Backend for Frontend).

Vì sao tồn tại: không có gateway, mỗi service tự làm auth + rate limit (trùng lặp, không nhất quán, lỗ hổng bảo mật khi một service quên), và client mobile phải gọi 8 service để vẽ một màn hình (8 round-trip trên mạng di động).

Anti-pattern nguy hiểm nhất: **gateway phình thành nơi chứa business logic** (kiểu ESB thời SOA) → thành monolith mới, mọi thay đổi nghiệp vụ phải sửa gateway, team gateway thành bottleneck tổ chức. Gateway chỉ nên chứa cross-cutting concern; logic nghiệp vụ thuộc về service.

## 7. Trade-off tổng hợp: Sync vs Async

| Khía cạnh | Synchronous (REST/gRPC) | Asynchronous (Queue/Stream) |
|---|---|---|
| Mô hình tư duy | Đơn giản, tuần tự | Khó hơn: eventual, out-of-order, duplicate |
| Coupling availability | Chuỗi nhân nhau: A cần B sống | A không cần B sống lúc gửi |
| Latency trả lời | Thấp nhất có thể | Cao hơn, không cam kết |
| Backpressure | Tự nhiên (caller chờ) | Phải thiết kế (queue depth, lag monitoring) |
| Debug/trace | Dễ | Khó (cần correlation ID, tracing xuyên broker) |
| Failure mode chính | Cascading failure, thundering herd | Queue tràn, consumer lag, poison message, duplicate |
| Nghiệp vụ phù hợp | Cần kết quả để đi tiếp (query, validate) | Thông báo sự kiện, side effect, batch, tích hợp nhiều consumer |

Nguyên nhân kỹ thuật của khác biệt cốt lõi: sync **truyền cả failure lẫn backpressure ngược lên caller ngay lập tức** (tốt: hệ thống tự điều tiết; xấu: lỗi lan truyền), async **cắt đường lan truyền đó bằng buffer** (tốt: cô lập lỗi; xấu: buffer che giấu vấn đề đến khi tràn — consumer lag là khoản nợ trả sau).

## 8. Production Considerations

- **Timeout mọi nơi, có ngân sách**: request từ edge có 800ms → chia xuống các hop (gateway 750ms → service A 500ms → DB 200ms). gRPC deadline propagation làm việc này tự động — hop sau biết còn bao nhiêu thời gian. Timeout hop sau **phải nhỏ hơn** hop trước, không thì hop trước bỏ đi rồi hop sau vẫn cày.
- **Connection pooling**: mở TCP+TLS mỗi request tốn 1–3 RTT. Pool phải có max size (chống cạn kiệt file descriptor) và health check.
- **Giám sát queue**: consumer lag (Kafka) / queue depth (RabbitMQ) là chỉ số sống còn — lag tăng đều nghĩa là consumer không theo kịp, sẽ thành sự cố trong X giờ nữa (tính được X!). Alert trên đạo hàm, đừng chờ tràn.
- **Poison message**: message làm consumer crash lặp đi lặp lại chặn cả queue → sau N lần thất bại chuyển vào **Dead Letter Queue** + alert (chi tiết chương 11).
- **Schema evolution**: dùng Schema Registry (Kafka) hoặc kỷ luật Protobuf (không đổi nghĩa field number, chỉ thêm optional field). Breaking change trên topic đang có consumer cũ = outage âm thầm.
- **Capacity**: Kafka partition count quyết định độ song song tối đa của consumer — tăng partition sau này làm vỡ ordering theo key, nên tính dư từ đầu (nhưng đừng 1000 partition cho topic 10 msg/s — mỗi partition tốn file handle và replication overhead).

## 9. Anti-patterns

- **Chatty services**: một request cha đẻ ra 50 remote call tuần tự. 50 × 2ms = 100ms chỉ cho network. Sửa: batch API, aggregation, xem lại ranh giới service (hai service nói chuyện nhiều thế thường nên là một).
- **Synchronous chain quá dài**: A→B→C→D→E đồng bộ. Availability nhân nhau, latency cộng nhau, retry ở mỗi tầng khuếch đại traffic (chương 11).
- **Dùng queue làm database**: lưu trạng thái nghiệp vụ dài hạn trong queue, query bằng cách đọc hết queue.
- **Request/Response giả trên queue** cho luồng cần trả lời ngay: độ trễ và phức tạp của async, ngữ nghĩa của sync — tệ cả hai đường.
- **Bỏ qua duplicate**: "message trùng hiếm lắm" — ở 1M msg/ngày, 0.01% là 100 lần trùng mỗi ngày. Email gửi 2 lần thì xấu hổ; trừ tiền 2 lần thì lên báo.
- **Event chứa cả thế giới**: event 2MB chứa toàn bộ object graph → mọi consumer coupling vào schema khổng lồ. Ngược lại, event chỉ có ID → mọi consumer quay lại gọi API producer (chatty). Cân bằng: event chứa dữ liệu mà đa số consumer cần (xem chương 09).

## 10. Troubleshooting

| Triệu chứng | Nghi phạm hàng đầu | Cách xác nhận |
|---|---|---|
| p99 tăng vọt, p50 bình thường | GC pause, connection pool cạn, một instance chậm, retry khuếch đại | Trace các request chậm; đo per-instance |
| Timeout hàng loạt một service | Service đó quá tải hoặc dependency của nó chậm (lan truyền) | Trace sâu xuống; xem saturation của dependency |
| Consumer lag tăng đều | Throughput consumer < producer; hoặc một partition hot | Lag per-partition; nếu 1 partition lệch hẳn → hot key |
| Message xử lý 2 lần | Consumer crash sau xử lý trước khi commit offset; retry của producer | Kiểm tra idempotency; log correlation ID |
| Sự kiện sai thứ tự | Partition key không theo entity; hoặc producer retry làm hoán vị | Xem partition key; bật `enable.idempotence` của Kafka producer |
| "Connection reset" rải rác | LB idle timeout < keepalive của app; pod bị kill khi đang phục vụ | So timeout các tầng; kiểm tra graceful shutdown + drain |

---

## Tổng kết chương

### Những điều bắt buộc phải nhớ
1. Remote call khác local call về **failure semantics**, không chỉ về tốc độ. Kết cục thứ ba — "không biết" — chi phối mọi thiết kế.
2. Exactly-once delivery không tồn tại ở tầng vận chuyển; chỉ có at-least-once + idempotency.
3. Queue ≠ Log: queue là việc-cần-làm (xóa khi xong), log là sự-đã-xảy-ra (giữ lại, replay được).
4. Kafka ordering chỉ trong một partition — partition key là quyết định thiết kế quan trọng nhất khi dùng Kafka.
5. Sync truyền lỗi và backpressure tức thì; async cắt lan truyền bằng buffer nhưng buffer che vấn đề đến khi tràn.
6. Gateway cho cross-cutting concern; business logic không bao giờ vào gateway.

### Hiểu lầm phổ biến
- "gRPC luôn tốt hơn REST" — chỉ khi serialize/network là bottleneck thật của bạn.
- "Kafka đảm bảo ordering" — chỉ per-partition, và chỉ khi producer cấu hình đúng.
- "Đưa queue vào là hệ thống resilient" — queue đổi failure mode (từ lỗi tức thì sang lag/tràn), không xóa failure.
- "Service mesh miễn phí" — trả bằng độ phức tạp vận hành và latency mỗi hop.

### Câu hỏi tự kiểm tra
1. Luồng thanh toán của bạn gọi service Fraud đồng bộ. Fraud chết thì sao? Chuyển sang async được không — điều gì trong nghiệp vụ quyết định?
2. Topic Kafka của bạn cần thứ tự theo đơn hàng nhưng có 100K đơn/giờ. Chọn partition key gì và bao nhiêu partition? Điều gì xảy ra nếu sau này tăng gấp đôi partition?
3. Thiết kế timeout budget cho chuỗi Gateway → OrderService → InventoryService → PostgreSQL với SLA 1 giây.
4. Vì sao readiness check không nên gọi xuống database? Khi nào thì nên?

### Tài liệu kinh điển nên đọc
- **"A Note on Distributed Computing" (Waldo et al., 1994)** — vì sao che giấu sự khác biệt local/remote là sai lầm; đọc để hiểu tử huyệt của RPC.
- **"Kafka: a Distributed Messaging System for Log Processing" (LinkedIn, 2011)** — bản thiết kế gốc: vì sao log, vì sao dumb broker.
- **"The Log: What every software engineer should know..." (Jay Kreps)** — bài essay biến log thành khái niệm trung tâm của kiến trúc dữ liệu; nền của chương 09.
- **"Enterprise Integration Patterns" (Hohpe & Woolf)** — từ điển pattern messaging, vẫn chuẩn sau 20 năm.
- **Google SRE Book, chương "Handling Overload"** — deadline propagation và retry budget trong production thật.
