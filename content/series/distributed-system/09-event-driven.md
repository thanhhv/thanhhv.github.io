+++
title = "Chương 09 – Event-driven Architecture: Sự kiện là hạng nhất"
date = "2026-07-09T01:00:00+07:00"
draft = false
tags = ["backend", "distributed-systems"]
series = ["Distributed Systems"]
+++

## 1. Problem Statement

Khi "đơn hàng được tạo" cần kích hoạt 8 việc (email, tồn kho, loyalty, analytics, fraud, shipping...), mô hình gọi-trực-tiếp buộc OrderService **biết và gọi** cả 8 — mỗi consumer mới là một lần sửa OrderService, availability của việc *tạo đơn* phụ thuộc cả 8 hệ phụ trợ, và latency cộng dồn. Nghiêm trọng hơn ở tầng tổ chức: team Order thành nút cổ chai của mọi team khác.

Event-driven đảo ngược sự phụ thuộc: OrderService chỉ tuyên bố **sự thật đã xảy ra** ("OrderCreated"), ai cần thì tự nghe. Thêm consumer thứ 9: không sửa gì ở producer. Cái giá: mất đi câu trả lời tức thì "8 việc kia xong chưa?", nợ eventual consistency lan ra toàn kiến trúc, và luồng nghiệp vụ trở nên vô hình nếu không đầu tư observability.

## 2. Event là gì — và không là gì

**Event = bản ghi bất biến về việc ĐÃ xảy ra**, thì quá khứ: `OrderCreated`, `PaymentCaptured`. So với hai người anh em:

| | Command | Event | Query |
|---|---|---|---|
| Ngữ nghĩa | "Hãy làm X" — mệnh lệnh, gửi tới MỘT nơi | "X đã xảy ra" — sự thật, ai nghe thì nghe | "Cho tôi biết X" |
| Từ chối được? | Có (validate, reject) | Không — quá khứ không thương lượng | — |
| Coupling | Sender biết receiver | Producer không biết consumer | Caller biết callee |

Nhầm command thành event là anti-pattern ngữ nghĩa phổ biến nhất: `SendEmailEvent` thực chất là command đội lốt — producer đang *ra lệnh* cho email service, chỉ là qua Kafka. Hệ quả: coupling vẫn nguyên (producer phải biết email service muốn gì) nhưng giờ thêm cả async phức tạp.

**Domain Event vs Integration Event**: domain event là chuyện nội bộ bounded context, schema thoải mái đổi, giàu chi tiết; integration event là **hợp đồng public giữa các team** — versioning nghiêm ngặt, schema registry, backward compatibility, tối giản. Phát thẳng domain event ra ngoài = mời cả công ty coupling vào internal model của bạn; đổi một field nội bộ = phá 5 team. Luôn có lớp dịch (anti-corruption layer) giữa hai loại.

**Event mang gì**: notification-only (chỉ ID → consumer gọi lại API: chatty, nhưng luôn tươi) vs event-carried state transfer (đủ dữ liệu → consumer tự trị, nhưng schema to và dữ liệu có thể cũ lúc xử lý). Thực dụng: mang dữ liệu mà *đa số* consumer cần, kèm ID để ai cần thì lấy thêm.

## 3. Event Bus vs Event Streaming — hai hạ tầng, hai triết lý

Đã phân tích kỹ ở chương 02 (queue vs log); nhắc lại điểm quyết định cho kiến trúc event-driven: **event streaming (Kafka) giữ event lại** → consumer mới đọc lại từ đầu (bootstrap read model, backfill analytics), consumer có bug sửa xong replay. Khả năng **replay** này là nền của CQRS/Event Sourcing bên dưới — event bus kiểu fire-and-forget (SNS, RabbitMQ fanout) không cho điều đó. Đổi lại là vận hành nặng hơn và retention là tiền.

## 4. CQRS — tách đường ghi khỏi đường đọc

### 4.1. Vì sao tồn tại

Một model dữ liệu phục vụ cả ghi lẫn đọc là một sự thỏa hiệp: ghi muốn normalize (toàn vẹn, không trùng lặp), đọc muốn denormalize (một trang cần dữ liệu từ 7 bảng — join đắt). Khi read:write = 100:1 (điển hình web), thỏa hiệp nghiêng hẳn về một phía vẫn không đủ.

CQRS: **write model** (validate, ràng buộc, normalize) tách khỏi **read model** (denormalized, materialized view, tối ưu cho từng màn hình), đồng bộ bằng event:

```
Command ──▶ [Write Model / DB ghi] ──event (qua outbox!)──▶ [Projector] ──▶ [Read Model(s)]
Query  ◀──────────────────────────────────────────────────────────────────── (Elasticsearch,
                                                                    Redis, bảng denorm...)
```

Bạn có thể đã dùng CQRS mà không gọi tên: **DB chính + Elasticsearch cho search + Redis cho trang chủ** chính là nó. Điểm mới của việc gọi tên: read model là *dẫn xuất có thể vứt đi và build lại từ event* — thay đổi cách nghĩ về migration và bug (sửa projector, replay, xong).

### 4.2. Cái giá

Read model **trễ hơn** write model (ms→s) → mọi vấn đề chương 03 (user vừa sửa xong chưa thấy — cần read-your-writes: đọc write model cho chính chủ, hoặc chờ version). Mỗi read model thêm là một projector phải vận hành + một nguồn drift phải đối soát. **CQRS theo từng ngữ cảnh, không toàn hệ thống** — màn hình admin CRUD đơn giản không cần.

## 5. Event Sourcing — event LÀ dữ liệu

### 5.1. Cơ chế

Thay vì lưu trạng thái hiện tại (UPDATE đè), lưu **chuỗi event bất biến**; trạng thái = fold(events):

```
AccountOpened(+0) → Deposited(+500) → Withdrew(-200) → Deposited(+100)
                                                  state = 400
```

Ghi mới = append event (kèm optimistic concurrency: expected version của aggregate — hai lệnh đồng thời trên cùng aggregate, một cái thua và retry). Đọc nhanh = **snapshot** mỗi N event + replay phần đuôi.

### 5.2. Được gì, mất gì — nói thẳng

**Được**: audit trail hoàn hảo *bẩm sinh* (ledger ngân hàng, hồ sơ y tế — nơi regulator hỏi "vì sao số dư là X" bạn trả lời bằng chứng cứ, không bằng niềm tin); truy vấn thời gian ("trạng thái lúc 23:59 ngày 31/12"); debug bằng replay đúng chuỗi event production; read model mới từ dữ liệu lịch sử đầy đủ.

**Mất**: (1) **Schema evolution của event là bài toán vĩnh viễn** — event 5 năm trước vẫn phải đọc được: upcaster, versioning, kỷ luật — chi phí trả mãi mãi. (2) Query ad-hoc "mọi account số dư > X" bất khả trên event store → **bắt buộc kéo theo CQRS** (read model cho mọi câu hỏi). (3) Xóa dữ liệu (GDPR) mâu thuẫn triết lý bất biến (giải: crypto-shredding — mã hóa PII per-user, xóa key). (4) Đường học dốc cho cả team, mãi mãi (mọi người mới đều phải học lại).

**Khuyến nghị mạnh**: Event Sourcing cho **một vài aggregate cốt lõi** có yêu cầu audit/temporal thật (ledger, order lifecycle) — không phải kiến trúc toàn hệ thống. "Event-source everything" là con đường đã làm nhiều team hối hận; các bài "we moved away from event sourcing" trên mạng đều cùng một cốt truyện: dùng nó như default thay vì như công cụ chuyên dụng.

Phân biệt với **event-carried integration** (phổ biến hơn nhiều): hệ của bạn lưu trạng thái bình thường trong PostgreSQL và *phát event qua outbox* — đó KHÔNG phải event sourcing (event là sản phẩm phụ, không phải source of truth), và với đa số hệ thống, đó chính là điểm dừng đúng.

## 6. Trade-off tổng hợp

- **Coupling vs khả năng nhìn thấu (visibility)**: event-driven đổi coupling compile-time lấy coupling ngầm runtime. Luồng "tạo đơn → 8 việc" không còn đọc được ở bất kỳ file code nào — nó *tồn tại* trong tổng hòa các subscription. Bắt buộc bù bằng: distributed tracing xuyên broker (correlation ID trong header event), event catalog (AsyncAPI), và sơ đồ luồng được duy trì như code.
- **Autonomy vs consistency**: mỗi service tự trị với read model riêng → toàn hệ thống là eventual. Câu hỏi "tổng tồn kho hiện tại chính xác?" không còn câu trả lời rẻ.
- **Throughput vs ordering**: ordering per-key (partition) là thứ Kafka cho; ordering toàn cục thì phải một partition — mất song song. Thiết kế consumer chịu được out-of-order giữa các key khác nhau.
- **Retention là tiền**: replay-được-từ-đầu nghĩa là trả tiền lưu trữ mãi (compacted topic giảm nhẹ cho state-oriented event).

## 7. Production Considerations

- **Consumer bắt buộc idempotent** (chương 08 — inbox pattern hoặc khóa tự nhiên). Không thương lượng, vì at-least-once là mặc định.
- **Consumer lag là SLI hạng nhất**: lag tăng đều = sự cố đang ủ. Alert theo đạo hàm và theo tuổi message già nhất.
- **DLQ + quy trình xử lý**: poison event không được chặn partition; nhưng DLQ không có người xem = hố đen nghiệp vụ. Dashboard tuổi + số lượng DLQ, quy trình redrive.
- **Schema Registry + compatibility mode** (backward cho consumer cũ đọc event mới): CI chặn breaking change trước khi nó ra production.
- **Backfill/replay có kế hoạch**: replay 100M event vào projector cũng ăn CPU/IO như traffic thật — throttle, chạy read model mới song song rồi cutover, đừng replay đè lên hệ đang phục vụ.
- **Event catalog**: ai phát gì, ai nghe gì, schema nào — không có nó, sau 2 năm không ai dám tắt topic nào nữa.

## 8. Anti-patterns

- **Command đội lốt event** (`SendEmailRequested` phát cho đúng 1 consumer đã biết trước) — coupling cũ, phức tạp mới.
- **Event spaghetti / vòng lặp**: A phát → B nghe phát → C nghe phát → A nghe... chu trình event vô tình = bão message tự khuếch đại. Vẽ đồ thị event, cấm chu trình.
- **Phát domain event nội bộ ra toàn công ty** — mọi team coupling vào internal schema; refactor nội bộ thành sự kiện liên-team.
- **Dual-write khi phát event** (quên outbox — chương 08).
- **Event sourcing toàn hệ thống theo trend** — xem §5.2.
- **Read model không build lại được** (projector có side effect, event thiếu dữ liệu) — mất đi lợi ích lớn nhất của toàn kiến trúc.
- **Giả định ordering toàn cục** — code chạy đúng ở dev (1 partition) sập ở production (12 partition).

## 9. Khi nào KHÔNG dùng event-driven

Luồng cần câu trả lời ngay để đi tiếp (validate thanh toán trước khi cho qua) — request/response là đúng, đừng gò thành event + chờ callback. Hệ nhỏ, một team, một DB — transaction cục bộ + gọi hàm nhanh gọn đúng đắn; event-driven ở đây là nghi lễ tốn kém. Nghiệp vụ đòi strong consistency giữa các bước — saga/event làm mọi thứ khó hơn, cân nhắc gom về một service. Team chưa có nền observability — event-driven mà không có tracing là hệ thống không ai hiểu nổi sau 18 tháng.

## 10. Troubleshooting

| Triệu chứng | Nguyên nhân khả dĩ | Xử lý |
|---|---|---|
| Read model lệch write model | Projector lỗi nuốt event; out-of-order giữa key (partition key sai); dual-write | Đối soát định kỳ; kiểm tra partition key; outbox |
| Event "biến mất" | Thực ra ở DLQ không ai xem; hoặc consumer group mới bắt đầu từ `latest` | Giám sát DLQ; hiểu `auto.offset.reset` |
| Xử lý trùng gây sai số liệu | Consumer không idempotent + rebalance/crash | Inbox pattern; khóa version |
| Bão message tự khuếch đại | Chu trình event; retry không backoff ở nhiều tầng | Đồ thị event; correlation ID để tìm chu trình; hoãn retry |
| Replay làm sập DB đích | Không throttle, projector ghi từng dòng | Batch write; build song song rồi cutover |
| "Luồng này đi qua đâu?" mất 2 ngày trả lời | Thiếu tracing xuyên broker + event catalog | Chuẩn hóa trace context trong header; AsyncAPI |

---

## Tổng kết chương

### Những điều bắt buộc phải nhớ
1. Event = sự thật quá khứ, không từ chối được; command = mệnh lệnh cho một nơi. Nhầm ngữ nghĩa là nhầm kiến trúc.
2. Domain event ≠ integration event: một cái là chuyện nhà, một cái là hợp đồng public — luôn có lớp dịch.
3. Event-driven đổi coupling lấy visibility — khoản bù bắt buộc: tracing, catalog, đối soát.
4. CQRS = tách model theo nhu cầu đọc/ghi, trả bằng lag và projector; bạn có thể đã đang dùng nó (DB + ES + Redis).
5. Event Sourcing là công cụ chuyên dụng cho aggregate cần audit/temporal — không phải default; và nó kéo theo CQRS bắt buộc.
6. Mọi consumer idempotent, mọi producer qua outbox — hai chân đứng của toàn kiến trúc.

### Hiểu lầm phổ biến
- "Dùng Kafka = event-driven architecture" — chở command qua Kafka vẫn là RPC, chỉ chậm hơn và khó debug hơn.
- "Event sourcing là bản xịn của audit log" — audit log là *một* lợi ích; cái giá là cả mô hình dữ liệu và query bị đảo lộn.
- "CQRS + ES phải đi cùng nhau" — CQRS đứng một mình rất tốt (và phổ biến hơn nhiều).
- "Eventual consistency là chi tiết kỹ thuật" — nó là *đặc tính sản phẩm* mà PM và UX phải cùng thiết kế (trạng thái "đang xử lý").

### Câu hỏi tự kiểm tra
1. Hệ e-commerce của bạn: liệt kê 5 event, phân loại domain/integration, quyết định payload cho từng cái (notification vs state transfer) và giải thích.
2. Thiết kế read model "trang đơn hàng của tôi" (order + payment + shipping status): projector nghe những event nào, partition key gì để ordering đúng, xử lý out-of-order giữa PaymentCaptured và OrderShipped thế nào?
3. Regulator yêu cầu chứng minh mọi thay đổi số dư trong 7 năm. So sánh: (a) audit table ghi bằng trigger, (b) event sourcing ledger. Chi phí và độ tin cậy mỗi bên?
4. Team bạn muốn "event-source toàn bộ hệ thống mới". Viết 5 câu hỏi phản biện bạn sẽ đặt ra trong design review.

### Tài liệu kinh điển nên đọc
- **"The Log" (Jay Kreps)** — nền tư tưởng: log như trục xương sống dữ liệu (nhắc lại từ chương 02 vì nó là móng của chương này).
- **Martin Fowler: "Event-Driven" / "CQRS" / "Event Sourcing"** — phân định khái niệm sạch nhất; đặc biệt bài "What do you mean by Event-Driven?" tách 4 pattern hay bị trộn.
- **Greg Young — "CQRS Documents" và các talk** — người đặt tên CQRS, kèm mọi cảnh báo mà người sau bỏ qua.
- **"Designing Event-Driven Systems" (Ben Stopford, miễn phí từ Confluent)** — event-driven trên Kafka, thực dụng từ người trong cuộc.
- **"Versioning in an Event Sourced System" (Greg Young)** — cuốn mỏng về bài toán đau nhất của ES: schema evolution.
