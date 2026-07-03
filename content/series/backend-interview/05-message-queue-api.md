+++
title = "Bài 5 — Message Queue & API"
date = "2026-07-03T12:00:00+07:00"
draft = false
tags = ["backend", "kafka", "interview"]
series = ["Backend Interview"]
+++

# Message Queue & API

---

# PHẦN A — MESSAGE QUEUE

## Câu 1 — [Intermediate → Senior] Kafka hoạt động thế nào? Tại sao nó đạt throughput hàng triệu message/giây?

### 1. Câu hỏi
"Kiến trúc Kafka: topic, partition, consumer group, offset. Điều gì cho phép Kafka nhanh như vậy trên phần cứng thường?"

### 2. Interviewer muốn kiểm tra điều gì?
- Hiểu Kafka là **distributed log**, không phải message queue truyền thống — khác biệt định hình mọi tính chất.
- Giải thích được nguồn hiệu năng bằng cơ chế cụ thể (sequential I/O, zero-copy, batching), không phải "Kafka nhanh vì được thiết kế tốt".
- Nắm ordering, rebalance, lag — các khái niệm vận hành hằng ngày.

### 3. Câu trả lời ngắn gọn (30 giây)
"Kafka là log phân tán: topic chia thành partition, mỗi partition là file log append-only có thứ tự; message không bị xóa khi đọc mà giữ theo retention; consumer tự quản offset của mình. Consumer group chia partition cho các consumer — scale đọc bằng cách thêm consumer tới tối đa = số partition. Nhanh nhờ: ghi/đọc **tuần tự** trên đĩa (nhanh gần bằng RAM, tận dụng page cache OS), **zero-copy** (sendfile — dữ liệu từ page cache thẳng ra socket không qua user space), **batching + nén** cả lô, và broker cực kỳ 'ngu' — không track trạng thái từng message như RabbitMQ, chỉ là cái log có index."

### 4. Câu trả lời Senior Level (3–5 phút)
**Mô hình:**
- Partition là đơn vị của mọi thứ: ordering (chỉ đảm bảo **trong** partition), parallelism (1 partition chỉ 1 consumer trong group xử lý), replication (leader/follower, ISR).
- Producer chọn partition theo key (hash) → cùng key cùng partition → thứ tự per-key. Chọn key là quyết định thiết kế ngang tầm chọn shard key.
- Consumer **pull** (không phải broker push): consumer tự điều tốc — backpressure tự nhiên; đổi lại latency thêm chút và phải quản lý offset commit.
- **Offset commit** là ranh giới ngữ nghĩa delivery: commit trước khi xử lý → at-most-once (mất khi crash); xử lý xong mới commit → at-least-once (trùng khi crash) → consumer phải **idempotent**. Đây là câu then chốt: mặc định thực tế của mọi hệ Kafka là at-least-once + idempotent consumer.
- **Replication:** leader nhận ghi; ISR = tập follower bám kịp; `acks=all` + `min.insync.replicas=2` → message bền qua mất 1 broker. `acks=1` nhanh hơn, mất dữ liệu khi leader chết trước khi replicate.
- **Rebalance:** thêm/bớt consumer → chia lại partition; rebalance storm (deploy rolling làm rebalance liên hoàn) từng là nỗi đau lớn — cooperative sticky assignor + static membership giảm đáng kể.

**Nguồn hiệu năng:** (1) mọi I/O tuần tự — HDD cũng đạt hàng trăm MB/s tuần tự; (2) dựa hẳn vào page cache của OS, không cache riêng trong JVM heap (né GC); (3) zero-copy sendfile cho consumer đọc dữ liệu đã nén sẵn; (4) batch từ producer tới đĩa tới consumer — amortize mọi chi phí; (5) không per-message state ở broker: "ai đọc gì" là việc của consumer (offset), broker chỉ append và serve theo offset — O(1) bất kể số consumer.

### 5. Giải thích bản chất
Kafka thắng nhờ **đảo ngược trách nhiệm**: queue truyền thống (RabbitMQ) coi broker là bên thông minh — track từng message, ai đã ack, xóa khi xong; chi phí trạng thái tỷ lệ với số message × số consumer. Kafka nhận ra: nếu dữ liệu là **log bất biến có thứ tự**, thì trạng thái đọc của mỗi consumer nén được thành **một con số** (offset). Toàn bộ độ phức tạp per-message biến mất; broker thành máy append + serve tuần tự — thứ mà đĩa và OS làm giỏi nhất. Đây cũng là lý do Kafka replay được (dữ liệu còn nguyên) và nhiều consumer group đọc độc lập không tốn thêm gì. Cùng insight với WAL của database: **log là cấu trúc dữ liệu nền tảng của hệ phân tán** ("The Log" — Jay Kreps).

### 6. Trade-off
- **Partition ordering:** thứ tự per-key rẻ ↔ thứ tự toàn cục không có (muốn có = 1 partition = mất parallelism).
- **Pull model:** backpressure tự nhiên, replay dễ ↔ latency nhỉnh hơn push với tải thấp.
- **Nhiều partition:** parallelism cao ↔ nhiều file handle, recovery lâu, end-to-end latency tăng (batch nhỏ đi); con số cần quy hoạch, đổi sau này đau (key bị hash lại → mất per-key ordering tại thời điểm đổi).
- **`acks=all` + min ISR:** bền ↔ latency ghi tăng và mất availability ghi khi ISR co lại (chọn C hơn A đúng nghĩa CAP).
- **Retention theo thời gian/dung lượng:** replay và audit ↔ chi phí đĩa; tiered storage (KIP-405) làm dịu bài toán này.

### 7. Ví dụ Production
Sự cố mẫu: hệ thanh toán consume topic `payment-events`, xử lý xong mới commit offset (at-least-once, đúng chuẩn). Deploy giữa giờ cao điểm → rebalance → một batch đang xử lý dở bị giao lại cho consumer khác → **xử lý trùng 1.200 event** → may mà bảng ghi có unique constraint theo `event_id` nên chỉ 1.200 lỗi insert vô hại trong log. Team khác cùng công ty không có idempotency — trừ tiền trùng. Cùng một hành vi Kafka hoàn toàn đúng spec, một team coi là non-event, một team thành sự cố cấp cao. Bài học tôi luôn nhấn: **đừng hỏi 'Kafka có exactly-once không', hãy làm consumer idempotent — rẻ và bulletproof.**

### 8. Những câu trả lời chưa đủ tốt
- "Kafka nhanh vì phân tán." → RabbitMQ cũng phân tán được. Cơ chế cụ thể: sequential I/O? zero-copy? batching? phải nêu được ít nhất hai.
- "Kafka đảm bảo thứ tự message." → Chỉ trong partition, chỉ per-key, và chỉ khi producer cấu hình đúng (`max.in.flight` + idempotent producer; retry không idempotent có thể đảo thứ tự).

### 9. Sai lầm phổ biến của ứng viên
- Không biết ordering chỉ tồn tại trong partition — thiết kế hệ ngầm giả định thứ tự toàn topic.
- Nhầm consumer group (chia việc) với nhiều group (mỗi group một bản sao đầy đủ luồng dữ liệu).
- Không phân biệt commit offset trước/sau xử lý và hệ quả delivery semantics.
- Chọn số partition tùy hứng, hoặc key null (round-robin) rồi ngạc nhiên vì mất thứ tự per-entity.
- Nghĩ lag = hỏng. Lag là chỉ số **tốc độ tương đối**; lag ổn định có thể ổn, lag tăng đơn điệu mới là báo động.

### 10. Follow-up Questions
- Exactly-once của Kafka (idempotent producer, transactions) thực sự đảm bảo gì, phạm vi nào? (exactly-once **trong** hệ sinh thái Kafka read-process-write; ra ngoài DB vẫn cần idempotency.)
- Consumer chậm làm lag tăng — chẩn đoán và xử lý theo thứ tự nào? (đo per-partition, tăng parallelism trong consumer trước khi tăng partition, batch xử lý, tách slow path.)
- Poison message trong Kafka xử lý sao khi không có DLQ built-in? (retry topic + DLQ topic pattern, skip-and-log.)
- KRaft thay ZooKeeper — thay đổi gì về vận hành?
- Thiết kế topic cho hệ 500 loại event: topic per event type hay chung? Tiêu chí? (schema, consumer nào cần gì, ordering cần theo key nào, quota vận hành.)

### 11. Liên hệ với Production
LinkedIn (nơi sinh Kafka) chạy hàng nghìn tỷ message/ngày; các ngân hàng số Việt Nam đều xây backbone event trên Kafka. Vấn đề nghiêm trọng khi: consumer lag tăng không hồi phục (dấu hiệu under-provisioned hoặc poison message loop), rebalance storm mỗi lần deploy, hoặc topic bùng nổ không governance (nghìn topic không ai own, schema thay đổi làm vỡ consumer im lặng — cần schema registry + compatibility rule từ sớm). Dấu hiệu cần hành động: lag theo partition lệch (hot key), ISR shrink thường xuyên (mạng/đĩa broker), thời gian rebalance vượt phút.

---

## Câu 2 — [Senior] RabbitMQ vs Kafka vs SQS vs NATS — chọn thế nào cho đúng bài toán?

### 1. Câu hỏi
"Bốn hệ messaging: RabbitMQ, Kafka, SQS, NATS — mô hình khác nhau thế nào và anh chọn theo tiêu chí gì?"

### 2. Interviewer muốn kiểm tra điều gì?
- Tư duy chọn công nghệ theo bài toán, không theo trend — dấu hiệu Senior thật.
- Hiểu mô hình ngữ nghĩa khác nhau: smart broker vs log vs managed queue vs lightweight pub/sub.
- Biết chi phí vận hành là một chiều của quyết định.

### 3. Câu trả lời ngắn gọn (30 giây)
"Khác nhau ở mô hình: **RabbitMQ** — smart broker, routing phong phú (exchange/binding), per-message ack, ưu tiên, TTL, DLQ built-in → task queue và routing phức tạp; **Kafka** — distributed log, replay, retention, throughput khủng, ordering per-key → event streaming, event sourcing, pipeline dữ liệu; **SQS** — queue managed hoàn toàn, gần zero vận hành, scale vô hạn, nhưng ngữ nghĩa tối giản (at-least-once, không ordering trừ FIFO queue có giới hạn throughput) → glue giữa các service trên AWS; **NATS** — pub/sub siêu nhẹ, latency µs, core không bền vững (JetStream thêm persistence) → control plane, service mesh nội bộ, IoT. Tiêu chí chọn: cần replay không? routing phức tạp không? ai vận hành? throughput/latency bao nhiêu? đội đã có gì rồi?"

### 4. Câu trả lời Senior Level (3–5 phút)
**Khung phân tích theo 5 câu hỏi:**
1. **Message là lệnh (command) hay sự kiện (event)?** Command — một người xử lý, cần ack/retry/DLQ per-message → RabbitMQ/SQS. Event — nhiều bên quan tâm, có thể cần replay → Kafka.
2. **Có cần đọc lại quá khứ?** Consumer mới cần lịch sử, audit, rebuild projection → chỉ Kafka (log + retention) làm tự nhiên. Queue xóa message sau ack — quá khứ biến mất.
3. **Routing phức tạp đến đâu?** Topic exchange, header routing, priority, delay — RabbitMQ giàu nhất. Kafka chỉ có topic/partition — routing là việc của consumer.
4. **Ai vận hành?** SQS/SNS: không ai — đổi lấy lock-in và ngữ nghĩa giới hạn. Kafka self-hosted là một hệ thống cần chuyên môn thật (hoặc trả tiền MSK/Confluent). RabbitMQ ở giữa. NATS nhẹ nhất.
5. **Thang đo:** RabbitMQ thoải mái tới ~vài chục nghìn msg/s per node, đau khi queue dài (message ứ đọng làm chậm cả broker — trạng thái per-message); Kafka hàng triệu msg/s, và **queue dài không ảnh hưởng gì** (chỉ là log chưa đọc); SQS scale trong suốt; NATS core hàng triệu msg/s latency thấp nhất nhưng fire-and-forget.

**Các cặp nhầm lẫn kinh điển:** dùng Kafka làm task queue (thiếu per-message ack/retry/priority — consumer chậm 1 message chặn cả partition sau nó — head-of-line blocking); dùng RabbitMQ làm event store (không replay được); dùng SQS khi cần fan-out (phải ghép SNS→SQS); dùng NATS core cho dữ liệu không được mất.

**Thực tế tổ chức:** nếu công ty đã vận hành Kafka tốt, dùng Kafka cho cả task queue "tạm đủ" đôi khi hợp lý hơn dựng thêm RabbitMQ — chi phí một hệ thống mới lớn hơn sự không hoàn hảo của hệ hiện có. Quyết định công nghệ là quyết định danh mục vận hành (operational portfolio), không phải cuộc thi tính năng.

### 5. Giải thích bản chất
Sự khác biệt sâu nhất nằm ở **nơi giữ trạng thái tiêu thụ**: RabbitMQ/SQS giữ ở broker (per-message: delivered/acked → broker thông minh, đắt trạng thái, không replay); Kafka giữ ở consumer (một offset → broker rẻ, replay tự nhiên, consumer tự lo); NATS core **không giữ** (ai nghe thì nhận, không nghe thì thôi — ngữ nghĩa radio). Mọi tính chất khác (throughput, DLQ, fan-out, backpressure) đều suy ra từ lựa chọn này. Không có mô hình đúng — có mô hình khớp bài toán: bạn cần bưu điện (đảm bảo từng lá thư), nhật ký (đọc lại từ trang bất kỳ), hay đài phát thanh (nói cho ai đang nghe)?

### 6. Trade-off
- **Smart broker (Rabbit):** ngữ nghĩa giàu ↔ trạng thái là gánh nặng; queue sâu = hiệu năng sập.
- **Dumb broker (Kafka):** scale + replay ↔ mọi sự thông minh (retry, DLQ, priority, delay) tự xây ở tầng consumer.
- **Managed (SQS):** zero ops ↔ lock-in, chi phí theo message ở scale lớn, ngữ nghĩa không đàm phán được (visibility timeout thay cho ack truyền thống).
- **Lightweight (NATS):** latency, đơn giản ↔ độ bền phải mua thêm bằng JetStream, hệ sinh thái mỏng hơn.

### 7. Ví dụ Production
Case tôi tư vấn: team dùng Kafka làm queue gửi email. Một email tới địa chỉ chết làm SMTP call treo 30s; vì partition là hàng nối tiếp, **hàng nghìn email sau nó trong cùng partition chờ** — head-of-line blocking. Trong RabbitMQ, message đó đơn giản bị nack sang DLQ, những cái sau chạy tiếp. Fix trên Kafka phải tự xây: timeout + retry topic + DLQ topic + skip. Ngược lại, team analytics từng dùng RabbitMQ phát event: sau này muốn dựng lại toàn bộ dashboard từ lịch sử → không còn gì để đọc. Hai sự cố đối xứng — cùng một nguyên nhân: chọn công cụ theo cái đội biết, không theo hình dạng bài toán.

### 8. Những câu trả lời chưa đủ tốt
- "Kafka tốt nhất vì throughput cao nhất." → Throughput là một trục. Task queue 100 msg/s cần DLQ và priority thì Kafka là lựa chọn tệ.
- "Tùy use case." (rồi dừng) → Đúng nhưng rỗng. Senior phải nêu được **tiêu chí cụ thể** và ánh xạ tiêu chí → công cụ.

### 9. Sai lầm phổ biến của ứng viên
- Không phân biệt command vs event — gốc của hầu hết lựa chọn sai.
- Không biết head-of-line blocking khi dùng Kafka làm task queue.
- Không biết RabbitMQ sập hiệu năng khi queue tích tụ hàng triệu message (nơi Kafka bình thản).
- Bỏ qua chiều chi phí vận hành/con người trong so sánh.
- Không biết SQS FIFO có giới hạn throughput (per message group) — "cần ordering" trên SQS không miễn phí.

### 10. Follow-up Questions
- Delay queue (gửi sau 15 phút) làm thế nào trên từng hệ? (Rabbit: TTL+DLX hoặc delayed plugin; SQS: DelaySeconds ≤15m; Kafka: tự xây timer topic — đau.)
- Priority queue trên Kafka có làm được không? (không tự nhiên — topic riêng per priority + consumer ưu tiên.)
- Backpressure ở mỗi hệ hoạt động thế nào? (Kafka: pull tự nhiên; Rabbit: prefetch/credit; SQS: consumer tự điều; NATS: slow consumer bị ngắt!)
- Nếu phải chuẩn hóa công ty về MỘT hệ messaging, chọn gì và hy sinh gì? (câu Principal — không có đáp án đúng, chấm cách lập luận.)
- Outbox pattern cần gì từ hệ messaging? Hệ nào ghép với CDC tự nhiên nhất?

### 11. Liên hệ với Production
Netflix chạy cả Kafka (data pipeline) lẫn SQS (glue) — chuẩn hóa một công cụ cho mọi việc là ảo tưởng ở quy mô lớn, nhưng startup nên gom về 1–2. Vấn đề nghiêm trọng khi: số hệ messaging = số team (mỗi team một sở thích — chi phí vận hành nhân bản), hoặc một hệ bị dùng ngược sở trường quá xa. Dấu hiệu cần rà soát: RabbitMQ queue depth hàng triệu, Kafka consumer viết lại retry/DLQ logic lần thứ ba ở ba team khác nhau (lúc đó nên có platform library hoặc đổi công cụ).

---

# PHẦN B — API

## Câu 3 — [Intermediate → Senior] REST vs gRPC vs GraphQL — bản chất, hiệu năng, và tiêu chí lựa chọn

### 1. Câu hỏi
"So sánh REST, gRPC, GraphQL: khác nhau bản chất ở đâu, hiệu năng thế nào, và anh chọn theo tiêu chí gì cho từng loại API?"

### 2. Interviewer muốn kiểm tra điều gì?
- Vượt qua so sánh bề mặt (JSON vs binary) để chạm tới mô hình: resource vs procedure vs query.
- Hiểu chi phí thật của từng lựa chọn ở quy mô: N+1, over-fetching, HTTP/2, caching.
- Kinh nghiệm tổ chức: contract, versioning, ai là consumer.

### 3. Câu trả lời ngắn gọn (30 giây)
"Ba mô hình tư duy khác nhau: REST — **resource** và động từ chuẩn HTTP, tận dụng toàn bộ hạ tầng web (cache, CDN, proxy), contract lỏng; gRPC — **procedure call** với contract chặt (protobuf), HTTP/2 multiplexing, binary nhỏ và nhanh, streaming hai chiều, codegen đa ngôn ngữ → chuẩn cho service-to-service nội bộ; GraphQL — **ngôn ngữ truy vấn**, client tự khai báo hình dạng dữ liệu cần → giải over/under-fetching cho client đa dạng (mobile/web), trả giá bằng độ phức tạp server (N+1, complexity limit, caching khó). Công thức phổ biến: gRPC nội bộ, REST cho public API, GraphQL cho BFF phục vụ nhiều loại client."

### 4. Câu trả lời Senior Level (3–5 phút)
**REST:** sức mạnh thật không nằm ở "JSON dễ đọc" mà ở việc **đi cùng chiều với hạ tầng web**: GET cacheable ở mọi tầng (browser, CDN, reverse proxy) không cần code; statelessness cho scale ngang; semantics chuẩn (status code, idempotency của GET/PUT/DELETE) mà mọi công cụ hiểu sẵn. Điểm yếu: contract lỏng (OpenAPI là thuốc, uống hay không tùy kỷ luật), over-fetching/under-fetching, chatty với client cần dữ liệu ghép.

**gRPC:** protobuf contract-first — codegen client/server, type safety xuyên ngôn ngữ, backward compatibility có quy tắc rõ (field number, không reuse). HTTP/2: multiplexing (nhiều call trên 1 connection — hết head-of-line blocking ở tầng HTTP), header compression, streaming 4 kiểu. Binary + schema sẵn → serialize/deserialize nhanh hơn JSON nhiều lần, payload nhỏ hơn. Deadline propagation và interceptor chuẩn hóa cross-cutting concerns. Điểm yếu: browser không gọi trực tiếp (cần grpc-web/gateway), khó debug bằng mắt (cần grpcurl), load balancing L7 cần hiểu HTTP/2 (connection dài — LB per-request cần proxy hiểu gRPC hoặc client-side LB).

**GraphQL:** giải đúng một bài toán — **nhiều client với nhu cầu dữ liệu khác nhau trên cùng đồ thị dữ liệu**. Mobile cần 5 field, web cần 50 — REST hoặc trả thừa hoặc đẻ endpoint per màn hình. GraphQL để client khai báo. Chi phí server nặng: resolver N+1 (bắt buộc DataLoader batch), query độc hại sâu vô hạn (cần depth/complexity limit), caching HTTP mất tác dụng (mọi thứ là POST /graphql — cần persisted query + cache tầng app), authorization theo field. GraphQL là chuyển độ phức tạp từ client về server — hợp lý khi một server team phục vụ nhiều client team.

**Chiều tổ chức (điểm Staff):** chọn API style là chọn **giao thức giữa các team**. gRPC nội bộ = ép contract chặt giữa các team backend. GraphQL = trao quyền cho client team, gánh nặng lên platform team. REST public = tối ưu cho người tích hợp lạ (curl được, đọc được, Google được).

### 5. Giải thích bản chất
Ba mô hình là ba câu trả lời cho câu hỏi "**ai quyết định hình dạng dữ liệu trả về?**": REST — server quyết (resource cố định); gRPC — hai bên ký hợp đồng trước (schema); GraphQL — client quyết lúc runtime. Quyền càng về client, server càng phải phòng thủ (limit, cost analysis) và cache càng khó (không gian response bùng nổ); quyền càng về server, client càng phải chắp vá (nhiều call, thừa dữ liệu). Không có điểm tối ưu tuyệt đối — chỉ có điểm khớp với **cấu trúc team và độ đa dạng client** của bạn. Đây là định lý Conway hiện hình trong thiết kế API.

### 6. Trade-off
- **REST:** hạ tầng web miễn phí, phổ cập ↔ contract lỏng, chatty, versioning lộn xộn (URL? header?).
- **gRPC:** hiệu năng + contract + streaming ↔ độ mờ với công cụ web, browser cần cầu nối, vận hành LB phức tạp hơn.
- **GraphQL:** linh hoạt client tối đa ↔ server phức tạp gấp bội, hiệu năng khó dự đoán (mỗi query một hình dạng), monitoring per-query khó hơn per-endpoint.
- **Cả ba đồng thời:** mỗi mô hình một chỗ ↔ ba hệ công cụ, ba loại chuyên môn — đắt cho công ty nhỏ.

### 7. Ví dụ Production
Migration thật: hệ nội bộ 40 service REST/JSON, p99 tệ do serialize + nhiều call ghép. Chuyển core path sang gRPC: latency nội bộ giảm ~35%, nhưng bài học nằm ở chỗ khác — **protobuf contract + codegen chấm dứt loại bug 'field đổi tên mà consumer không biết'** vốn gây ra ~1 sự cố/tháng. Trong khi đó, phía GraphQL gateway cho mobile: một query lồng 6 cấp từ client kéo sập DB vì N+1 nhân 10.000 — phải thêm DataLoader, depth limit 5, complexity budget, và persisted queries (chỉ cho phép query đã đăng ký) — GraphQL public không rào chắn là mở cửa cho DoS hợp pháp.

### 8. Những câu trả lời chưa đủ tốt
- "gRPC nhanh hơn REST vì binary." → Một phần nhỏ. HTTP/2 multiplexing, schema-driven parsing, connection reuse — và nhanh hơn **bao nhiêu, ở đâu** (serialize? network? cả hai?).
- "GraphQL hiện đại hơn REST." → GraphQL giải một bài toán cụ thể với chi phí cụ thể. 'Hiện đại' không phải tiêu chí kỹ thuật.

### 9. Sai lầm phổ biến của ứng viên
- So sánh cả ba như hàng thay thế 1-1 trong khi chúng trả lời câu hỏi khác nhau.
- Không biết N+1 trong GraphQL và DataLoader.
- Không biết gRPC load balancing cần xử lý đặc biệt (long-lived HTTP/2 connection).
- REST versioning: chỉ biết /v1, /v2 mà không nói được additive change + tolerant reader — cách tránh version ngay từ đầu.
- Quên idempotency: POST không idempotent — retry tạo đơn trùng; thiếu idempotency key trong thiết kế API có ghi.

### 10. Follow-up Questions
- Thiết kế backward-compatible change trong protobuf: quy tắc nào? Điều gì cấm kỵ? (không đổi field number, không đổi kiểu không tương thích, reserved khi xóa.)
- REST API public: chiến lược rate limit, pagination (offset vs cursor), idempotency key cho POST?
- gRPC deadline propagation phối hợp với context của Go thế nào?
- GraphQL federation là gì, giải bài toán tổ chức nào?
- Nếu client cần realtime updates thì mỗi mô hình mở rộng thế nào? (REST: SSE/polling; gRPC: server streaming; GraphQL: subscription — và chi phí vận hành từng cái.)

### 11. Liên hệ với Production
Google (gRPC nội bộ toàn bộ), GitHub (REST + GraphQL song song), Shopify/Facebook (GraphQL cho client đa dạng) — mỗi lựa chọn phản chiếu cấu trúc tổ chức của họ. Vấn đề nghiêm trọng khi: public API đổi breaking change không version (đối tác vỡ hàng loạt), GraphQL không giới hạn complexity (sự cố do một query), gRPC connection reuse làm tải lệch hẳn về vài pod (thiếu client-side LB). Dấu hiệu cần rà soát: tỷ lệ payload bị vứt (over-fetching) cao, số round-trip per màn hình lớn, sự cố do contract drift giữa các team lặp lại.

---

## Câu 4 — [Senior → Staff] WebSocket ở quy mô lớn: thiết kế hệ realtime 1 triệu kết nối

### 1. Câu hỏi
"WebSocket khác gì HTTP thường? Thiết kế hệ notification/chat realtime cho 1 triệu kết nối đồng thời — những vấn đề nào sẽ xuất hiện?"

### 2. Interviewer muốn kiểm tra điều gì?
- Hiểu bản chất stateful của WebSocket — thứ phá vỡ mọi giả định stateless của scale web.
- Bài toán hệ thống thật: sticky routing, fan-out, reconnect storm, phát hiện kết nối chết.
- Biết cân nhắc SSE/long-polling — không phải cái gì realtime cũng cần WebSocket.

### 3. Câu trả lời ngắn gọn (30 giây)
"WebSocket là kết nối TCP hai chiều bền vững, mở bằng HTTP Upgrade — sau đó thoát khỏi mô hình request/response. Cái giá: **connection là state**. Ba bài toán lớn ở 1M kết nối: (1) **routing message tới đúng connection** — user A ở pod nào? Cần registry (Redis) hoặc pub/sub broadcast tới mọi pod; (2) **vòng đời kết nối** — heartbeat phát hiện chết, reconnect storm khi deploy/sự cố mạng (1M client cùng reconnect = tự DDoS — cần jitter backoff), resume/missed messages; (3) **deploy** — rolling restart cắt mọi kết nối. Và câu hỏi đầu tiên phải hỏi: có thật cần hai chiều không? Chỉ server đẩy xuống thì SSE đơn giản hơn đáng kể."

### 4. Câu trả lời Senior Level (3–5 phút)
**Bản chất:** HTTP request/response — server không thể chủ động nói. Các giải pháp đẩy dữ liệu: short polling (trễ + phí), long polling (giữ request chờ — trễ thấp hơn, vẫn overhead per message), SSE (một chiều server→client trên HTTP thường — cache/proxy thân thiện, tự reconnect built-in), WebSocket (hai chiều thật, sau handshake Upgrade là frame protocol trên TCP). Chọn theo **chiều dữ liệu**: chat/game/collab cần client→server tần suất cao → WebSocket; notification/feed/giá cả → SSE thường đủ và rẻ hơn nhiều về vận hành.

**Thiết kế 1M kết nối:**
- **Tầng connection gateway:** tách riêng service chỉ giữ kết nối (Go/Node đều giữ được 100k–1M+ mỗi node nếu tuned: file descriptor limit, buffer size per socket — RAM là giới hạn thật, ~vài chục KB/connection). Gateway "ngu": auth lúc handshake, giữ socket, forward hai chiều. Business logic ở service phía sau — stateless, scale bình thường.
- **Routing/fan-out:** map `userId → gateway node` trong Redis, backend publish message vào Redis Pub/Sub hoặc Kafka; mỗi gateway subscribe và đẩy xuống socket nó giữ. Fan-out lớn (1 message → 100k người trong một kênh) tính theo **outbound bandwidth** — nhân bản message là việc của gateway, không phải backend.
- **Heartbeat:** TCP không báo chết kịp thời (nửa mở) → ping/pong tầng ứng dụng 15–30s; timeout thì dọn state.
- **Reconnect storm:** deploy 1 gateway → 100k connection rơi → cùng reconnect. Bắt buộc: exponential backoff + jitter phía client, connection draining phía server (ngừng nhận mới, chờ hoặc chủ động đẩy client sang node khác dần dần), và rate limit handshake.
- **Missed messages:** client rớt 30 giây có nhận lại tin nhắn không? Nếu có: cần message log per channel (Kafka/Redis Stream) + client gửi last-seen-id khi reconnect → về bản chất bạn đang xây một consumer offset system. Nếu không cần (giá cổ phiếu — chỉ giá mới nhất có nghĩa): đơn giản hơn hẳn. **Câu hỏi này định hình 50% kiến trúc.**
- **Load balancer:** L4 hoặc L7 hỗ trợ WS; idle timeout của LB/proxy phải > heartbeat interval (sự cố kinh điển: ELB cắt kết nối im lặng 60s).

### 5. Giải thích bản chất
HTTP stateless scale đẹp vì **mọi request là hạt độc lập** — LB ném đâu cũng được, server chết request sau đi chỗ khác. WebSocket kéo ta về thế giới **stateful**: mỗi kết nối là một hợp đồng dài hạn với một máy cụ thể — đúng thứ mà kiến trúc web 20 năm qua cố loại bỏ. Vì thế mọi kỹ thuật trong bài (gateway tách riêng, registry, draining) đều là một mục tiêu: **cô lập state vào một tầng mỏng nhất có thể, giữ phần còn lại stateless**. Đây là nguyên lý tổng quát của thiết kế hệ thống: không xóa được state thì dồn nó vào một chỗ và làm chỗ đó thật giỏi một việc.

### 6. Trade-off
- **WebSocket vs SSE:** hai chiều, latency thấp nhất ↔ vận hành đắt hơn hẳn (LB, proxy, reconnect, không HTTP cache); SSE một chiều nhưng "chỉ là HTTP" — mọi hạ tầng hiện có dùng được.
- **Gateway to (1M conn/node) vs nhiều node nhỏ:** ít node, ít routing ↔ blast radius khủng khi một node chết (1M reconnect).
- **Broadcast pub/sub tới mọi gateway vs routing chính xác:** đơn giản ↔ băng thông nội bộ lãng phí; routing chính xác ↔ registry là thêm một hệ phải đúng.
- **Đảm bảo delivery qua reconnect:** UX tốt ↔ tự xây nửa cái Kafka; cân nhắc "client tự fetch lại qua REST khi reconnect" — thường đủ và đơn giản hơn nhiều.

### 7. Ví dụ Production
Sự cố tôi thích kể: hệ notification 300k kết nối chạy êm nhiều tháng. Một chiều thứ sáu, mạng của một ISP lớn chập 90 giây → 120k client rớt → client code retry **ngay lập tức, không backoff** → 120k handshake/giây dội vào auth service → auth chết → toàn bộ client (kể cả 180k chưa rớt bắt đầu bị timeout do auth nghẽn) vào vòng retry → hệ thống tự nướng mình 40 phút dù mạng ISP đã hồi sau 90 giây. Fix: backoff + jitter + trạng thái "degraded" phía client (dùng polling tạm), circuit breaker trước auth, và priority lane cho reconnect có token còn hạn. Bài học: **hệ realtime chết vì đàn client đồng loạt, không phải vì tải đều** — thiết kế cho stampede là thiết kế chính, không phải edge case.

### 8. Những câu trả lời chưa đủ tốt
- "WebSocket cho realtime, xong." → Realtime chiều nào? Tần suất? Missed message? SSE đủ không? Không phân tích là chưa thiết kế.
- "Scale bằng cách thêm node sau LB." → State thì sao? Message tới user ở node khác đi đường nào? Đây mới là bài toán.

### 9. Sai lầm phổ biến của ứng viên
- Quên hoàn toàn reconnect storm — thiết kế cho steady state, chết ở transient.
- Không có heartbeat tầng app, tin vào TCP keepalive (mặc định 2 giờ!).
- Không nghĩ tới deploy: rolling restart hệ stateful cần draining, không phải kill -9.
- Đặt business logic trong connection gateway → mỗi lần sửa logic là một lần cắt triệu kết nối.
- Không hỏi về missed messages — bỏ qua quyết định kiến trúc lớn nhất của bài.

### 10. Follow-up Questions
- Scale lên 10M kết nối, 100k message/s fan-out — thay đổi gì? (shard theo channel, tree fan-out, tính bandwidth trước tiên.)
- Client mobile: WebSocket giữ qua background thế nào? Push notification (FCM/APNs) thay thế khi nào?
- Đo và giám sát gì cho hệ WS? (concurrent conns, handshake rate, heartbeat miss rate, delivery latency e2e, fan-out lag.)
- Multi-region: user Việt Nam và user Mỹ chung một phòng chat — kiến trúc? (gateway region-local + message bus liên region + thứ tự tin nhắn: vector clock hay một region làm leader per room?)
- So sánh với managed (Pusher, Ably, API Gateway WebSocket): khi nào tự xây? (chi phí theo connection-phút của managed ở 1M conn thường đắt hơn tự vận hành — nhưng dưới 50k thì ngược lại.)

### 11. Liên hệ với Production
Slack, Discord công bố kiến trúc gateway/guild shard đúng mô hình trên (Discord: mỗi guild một process Elixir, gateway tách riêng). Vấn đề nghiêm trọng khi: số kết nối vượt file descriptor/RAM planning, deploy trở thành sự kiện gây sự cố (thiếu draining), hoặc một kênh "celebrity" fan-out vượt bandwidth một node. Dấu hiệu cần hành động: reconnect rate spike theo deploy, phân phối kết nối lệch giữa các node (LB không cân connection dài), heartbeat miss tăng trước khi user complain — đây là early warning quý nhất của hệ realtime.
