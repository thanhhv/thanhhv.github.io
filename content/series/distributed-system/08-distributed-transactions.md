+++
title = "Chương 08 – Distributed Transactions: Nhiều node, một nghiệp vụ nguyên tử"
date = "2026-07-09T00:00:00+07:00"
draft = false
tags = ["backend", "distributed-systems"]
series = ["Distributed Systems"]
+++

## 1. Problem Statement

Nghiệp vụ "đặt hàng" chạm 3 hệ: trừ tồn kho (Inventory DB), tạo đơn (Order DB), trừ tiền (Payment service). Trên một database, `BEGIN...COMMIT` cho tất cả-hoặc-không. Khi dữ liệu đã tách (vì sharding — chương 06, vì microservices), **không còn transaction chung** — và mọi kịch bản dở dang trở nên khả thi: trừ tiền rồi mà không có đơn; giữ chỗ tồn kho rồi mà thanh toán fail và không ai nhả.

Nếu không giải quyết có hệ thống: dữ liệu các service trôi dạt khỏi nhau từng ngày, đối soát thủ công thành nghề toàn thời gian, khách hàng mất tiền. Câu hỏi của chương: **đạt "tất cả hoặc không" xuyên node bằng cách nào, và trả giá gì.** Có hai trường phái: ép các node commit cùng nhau (2PC) hoặc chấp nhận trạng thái trung gian rồi tiến/lùi dần (Saga) — và một bài toán con xuất hiện ở *mọi* giải pháp: ghi DB và phát message một cách nguyên tử (Outbox).

## 2. Two-Phase Commit (2PC)

### 2.1. Cơ chế

```
            PHASE 1: PREPARE                PHASE 2: COMMIT
Coordinator ──prepare?──▶ A: khóa tài nguyên, ghi WAL, "YES"
            ──prepare?──▶ B: khóa tài nguyên, ghi WAL, "YES"
    │ tất cả YES → ghi quyết định COMMIT vào log (điểm không quay đầu)
            ──commit!──▶ A: commit, nhả khóa, ack
            ──commit!──▶ B: commit, nhả khóa, ack
    (bất kỳ ai NO/timeout ở phase 1 → ABORT tất cả)
```

Điểm mấu chốt: sau khi trả lời "YES", participant **mất quyền tự quyết** — không được commit, không được abort, phải chờ lệnh, *giữ khóa trong lúc chờ*.

### 2.2. Vì sao 2PC mang tiếng xấu — nguyên nhân kỹ thuật

1. **Blocking**: coordinator chết sau prepare, trước khi phát quyết định → participant kẹt với khóa đang giữ, **vô hạn định**. Khóa bị giữ chặn các transaction khác → nghẽn lan rộng. (3PC cố sửa nhưng đổ vỡ khi network partition; sửa triệt để cần thay coordinator bằng consensus — chính là hướng của DB hiện đại, xem 2.3.)
2. **Latency**: 2 round-trip + 2 lần fsync mỗi participant, và transaction chạy theo tốc độ của **participant chậm nhất**.
3. **Availability nhân**: transaction cần *mọi* participant sống — thêm participant là giảm availability tổng (chương 01).
4. **Heuristic decisions**: admin sốt ruột can thiệp tay một phía → nửa commit nửa abort — chính cái điều 2PC sinh ra để chống.

### 2.3. 2PC vẫn sống — trong lòng các database hiện đại

Đừng kết luận "2PC đã chết". **Spanner, CockroachDB, TiDB, YugabyteDB đều chạy 2PC cho transaction xuyên shard** — nhưng với hai nâng cấp quyết định: (a) **coordinator state được replicate bằng Paxos/Raft** → coordinator "không chết" (failover trong giây), xóa gần hết vấn đề blocking; (b) participant cũng là nhóm consensus. Bài học tinh tế: 2PC-giữa-các-service-mong-manh là anti-pattern; 2PC-giữa-các-nhóm-Raft là kiến trúc chuẩn của NewSQL. Vấn đề của 2PC không phải protocol — là **độ bền của những kẻ tham gia**.

Còn 2PC ứng dụng (XA giữa app + MySQL + RabbitMQ...): tránh — mọi lý do ở 2.2, cộng thêm hỗ trợ XA nửa vời của các hệ và không hoạt động xuyên tổ chức (không ai cho bạn giữ khóa trên hệ của Stripe).

## 3. Saga — chấp nhận trung gian, cam kết hoàn tất

### 3.1. Tư tưởng

Chuỗi transaction **cục bộ** T1→T2→...→Tn, mỗi bước commit thật (không giữ khóa chờ ai). Bước Ti thất bại → chạy **compensation** C(i-1)...C1 theo chiều ngược để *hủy về mặt nghiệp vụ*. Đổi atomicity vật lý lấy atomicity nghiệp vụ; đổi isolation lấy availability.

```
Đặt hàng:  [Tạo đơn PENDING] → [Giữ tồn kho] → [Trừ tiền] → [Xác nhận đơn]
Thất bại ở Trừ tiền:                 ▼
           [Hủy đơn]      ◀── [Nhả tồn kho] ◀── ✗
```

Hệ quả phải nhìn thẳng: **mất Isolation**. Giữa T1 và Tn, thế giới *nhìn thấy* trạng thái trung gian (đơn PENDING, tiền đã trừ chưa có vé). Nghiệp vụ phải định nghĩa ngữ nghĩa cho trạng thái đó (pending, reserved, "đang xử lý") — đây không phải chi tiết kỹ thuật, là **thiết kế nghiệp vụ**. Và compensation không phải rollback: email đã gửi không rút lại được (compensation = gửi email đính chính); tiền đã trừ hoàn lại là một *giao dịch mới* (có phí, có độ trễ, có thể fail — cần retry đến cùng).

### 3.2. Orchestration vs Choreography

| | Orchestration | Choreography |
|---|---|---|
| Cơ chế | Một orchestrator (state machine) gọi từng bước, giữ trạng thái saga | Mỗi service nghe event của service trước, tự làm phần mình |
| Luồng nghiệp vụ nhìn thấy ở đâu | Một chỗ (code/định nghĩa của orchestrator) | Rải khắp các service — phải "khảo cổ" mới dựng lại được |
| Coupling | Service coupling vào orchestrator | Coupling ngầm qua event schema |
| Debug/vận hành | Dễ: hỏi orchestrator "saga #123 đang ở đâu" | Khó: lần theo event qua N service |
| Rủi ro | Orchestrator thành "god service" nếu nhét business logic của các bước vào | "Event spaghetti", vòng lặp event vô tình |
| Phù hợp | Saga ≥ 3 bước, có compensation, cần nhìn trạng thái | 2–3 bước đơn giản, fire-and-forget |
| Công cụ | Temporal, AWS Step Functions, Camunda | Kafka + kỷ luật |

Kinh nghiệm: saga có compensation → **orchestration**, gần như luôn luôn. Temporal-style (durable execution) đáng học: code saga như code tuần tự, framework lo persist state, retry, resume — giảm hẳn lớp lỗi tự chế state machine.

## 4. Outbox Pattern — bài toán dual-write

### 4.1. Vấn đề nền tảng nhất chương này

Service ghi DB **và** phát event lên Kafka. Hai hệ, không có transaction chung:

```
Ghi DB ✓ → publish Kafka ✗ (crash giữa chừng)  → hệ khác không bao giờ biết
Publish ✓ → ghi DB ✗                            → hệ khác biết điều chưa xảy ra
```

Mọi kiến trúc event-driven (chương 09) đứng trên miệng hố này. Đảo thứ tự không cứu được — chỉ đổi kiểu sai.

### 4.2. Giải pháp: mượn transaction cục bộ

```
BEGIN;
  INSERT INTO orders (...);
  INSERT INTO outbox (event_id, aggregate_id, type, payload); -- CÙNG transaction
COMMIT;                        ↑ nguyên tử vì cùng một DB

[Relay] đọc outbox → publish Kafka → đánh dấu đã gửi
  • Cách 1: polling (đơn giản, trễ ~giây)
  • Cách 2: CDC — Debezium đọc WAL/binlog → Kafka (trễ ~ms, không query app DB)
```

Event chỉ được "hứa" khi transaction nghiệp vụ commit — hai việc thành một việc. Relay có thể publish **trùng** (crash sau publish trước khi đánh dấu) → downstream phải idempotent. Nghĩa là outbox cho **at-least-once**, và đó là lý do §5 tồn tại.

**Inbox pattern** (phía consumer): bảng `inbox(message_id PK)` — xử lý message và INSERT message_id trong *cùng* transaction cục bộ; message trùng vi phạm PK → bỏ qua. Outbox + Inbox = exactly-once **effect** giữa hai service, xây hoàn toàn từ transaction cục bộ + retry. Đây là bộ khung đáng tin nhất trong chương.

## 5. Idempotency — nền móng của mọi thứ ở trên

Vì retry là bắt buộc (chương 01: timeout không cho biết đã xảy ra chưa; chương 11: retry là phản xạ mặc định), mọi thao tác ghi qua mạng **sẽ** được gọi ≥1 lần. Idempotency: gọi N lần, hiệu ứng như 1 lần.

Các kỹ thuật, từ rẻ đến đắt:

1. **Tự nhiên idempotent**: `SET status='PAID'` (thay vì `balance -= x`); ghi kiểu upsert theo khóa tự nhiên. Thiết kế được thế này là thắng lớn nhất.
2. **Idempotency key**: client sinh key (UUID) cho mỗi *ý định* nghiệp vụ; server lưu `(key → kết quả)` với unique constraint; gặp lại key → **trả lại kết quả cũ**, không thực hiện lại. Chuẩn ngành thanh toán (Stripe `Idempotency-Key` header). Chi tiết production: key phải theo ý-định (không phải theo request HTTP); lưu cả response để trả lại nhất quán; TTL đủ dài hơn cửa sổ retry (24h+); xử lý race hai request cùng key đồng thời (lock theo key hoặc để unique constraint quyết).
3. **Conditional write / version**: `UPDATE ... WHERE version = 5` — CAS, chống cả duplicate lẫn lost-update.
4. **Inbox** (§4.2) cho luồng message.

## 6. Trade-off tổng hợp

| | Local transaction | 2PC (app-level) | 2PC trên consensus (NewSQL) | Saga |
|---|---|---|---|---|
| Atomicity | ✅ | ✅ (khi coordinator sống) | ✅ | Nghiệp vụ (qua compensation) |
| Isolation | ✅ | ✅ (khóa giữ lâu) | ✅ (thường SSI/MVCC) | ❌ — trạng thái trung gian lộ ra |
| Availability | Cao | Thấp (nhân các participant) | Cao | Cao |
| Latency | Thấp nhất | Cao | Trung bình–cao | Mỗi bước thấp; end-to-end dài (eventual) |
| Coupling | — | Chặt, đồng bộ | Trong một DB | Lỏng, async |
| Độ phức tạp đổ về đâu | — | Hạ tầng transaction | Vendor DB lo | **Nghiệp vụ**: compensation, trạng thái trung gian, đối soát |
| Dùng khi | Luôn ưu tiên nếu có thể | Gần như không bao giờ (app mới) | Cần ACID xuyên shard, chọn NewSQL | Xuyên service/tổ chức, chuẩn microservices |

Thứ tự ưu tiên thiết kế (quan trọng nhất chương): **(1) Vẽ lại ranh giới để transaction thành cục bộ** (co-locate dữ liệu cùng shard/service — chương 06; nhiều "distributed transaction" là service boundary sai) → **(2) Local transaction + Outbox + Idempotent consumer** (đủ cho đa số luồng) → **(3) Saga orchestration** khi có nhiều bước và compensation thật → **(4) NewSQL** khi thật sự cần ACID xuyên shard → **(5) 2PC app-level**: gần như không bao giờ.

## 7. Production Considerations

- **Đối soát (reconciliation) là bắt buộc, không phải tùy chọn**: job định kỳ so trạng thái giữa các hệ (đơn PAID mà không có payment record?), tự sửa hoặc bắn alert. Mọi hệ thanh toán nghiêm túc chạy đối soát với ngân hàng/PSP hằng ngày — vì dù thiết kế đúng đến đâu, entropy thắng trên đường dài.
- **Giám sát saga như giám sát queue**: số saga đang chạy, tuổi p99, số saga kẹt ở mỗi bước, tỷ lệ compensation. Saga kẹt 3 ngày ở "chờ hoàn tiền" là tiền thật của khách.
- **Timeout cho mỗi bước saga + hành động khi hết hạn** (retry? compensation? con người?) — định nghĩa lúc thiết kế.
- **Bảng outbox phình**: dọn bản ghi đã gửi (partition theo ngày + drop); giám sát lag của relay/Debezium như giám sát consumer lag.
- **Kiểm thử compensation**: đội chỉ test happy path sẽ khám phá compensation có bug vào đúng lúc nó chạy lần đầu — trong sự cố thật. Chaos test: giết service giữa saga.

## 8. Anti-patterns

- **Lạm dụng distributed transaction**: ranh giới service sai → mọi thao tác cần 3 service → "sửa" bằng saga phức tạp. Sửa ranh giới trước.
- **Saga không compensation** ("bước sau chắc chắn thành công mà") — đến ngày nó fail, dữ liệu kẹt trung gian vĩnh viễn.
- **Compensation không idempotent / không retry đến cùng** — hoàn tiền fail một lần rồi bỏ = mất tiền khách.
- **Dual-write "cứ thế publish"** sau commit — nguồn bug "search không thấy sản phẩm mới" kinh điển; luôn qua outbox/CDC.
- **Idempotency key theo HTTP request thay vì theo ý định** — client retry sinh key mới → trùng như thường.
- **Retry non-idempotent call** — trước khi thêm retry policy, hỏi "endpoint này gọi 2 lần thì sao?".
- **Event "OrderCreated" phát trước khi commit** — consumer đọc DB không thấy đơn (đọc điều chưa tồn tại).

## 9. Khi nào KHÔNG cần các pattern này

Mọi dữ liệu của luồng nằm được trong một DB → local transaction, hết chuyện — và đây là lý lẽ mạnh cho modular monolith. Luồng chịu được inconsistency ngắn + có đối soát → outbox + eventual là đủ, đừng dựng Temporal cho việc gửi email chào mừng. Chỉ 2 bước và bước 2 retry-được-đến-thành-công (không bao giờ cần lùi) → không cần saga framework, chỉ cần outbox + retry bền bỉ.

## 10. Troubleshooting

| Triệu chứng | Nguyên nhân khả dĩ | Xử lý |
|---|---|---|
| Đơn PAID không có bản ghi tiền | Dual-write thiếu outbox; saga chết giữa chừng không resume | Đối soát tìm quy mô; thêm outbox; orchestrator có persistence |
| Trừ tiền 2 lần | Retry không idempotency key | Idempotency key + unique constraint; đối soát hoàn tiền |
| Saga kẹt hàng loạt ở một bước | Service bước đó chết/chậm; poison input | Dashboard saga theo bước; DLQ cho saga; can thiệp tay có công cụ |
| Bảng khóa DB đầy, transaction xếp hàng (XA) | 2PC coordinator chết, participant giữ khóa in-doubt | Giải quyết in-doubt txn theo log coordinator; kế hoạch thoát XA |
| Event ra Kafka mà DB không có bản ghi | Publish trước commit | Chuyển sang outbox; consumer defensive (retry đọc) |
| Outbox lag tăng | Relay chậm/chết; bảng outbox quá to | Giám sát relay như consumer; dọn bảng; CDC thay polling |

---

## Tổng kết chương

### Những điều bắt buộc phải nhớ
1. Thứ tự ưu tiên: **local transaction (vẽ lại ranh giới) → outbox + idempotency → saga orchestration → NewSQL → 2PC app-level (đừng)**.
2. 2PC blocking khi coordinator chết; NewSQL cứu 2PC bằng cách đặt nó lên consensus — vấn đề là độ bền participant, không phải protocol.
3. Saga đánh đổi **Isolation**: trạng thái trung gian là thiết kế nghiệp vụ, compensation là giao dịch mới (phải idempotent, phải retry đến cùng), không phải rollback.
4. Dual-write là hố tử thần; outbox đóng nó bằng transaction cục bộ; inbox đóng phía nhận.
5. Exactly-once delivery không tồn tại; exactly-once **effect** = at-least-once + idempotency. Idempotency không phải tính năng thêm — là móng.
6. Đối soát định kỳ là tuyến phòng thủ cuối, bắt buộc cho hệ có tiền.

### Hiểu lầm phổ biến
- "Saga = transaction phân tán xịn hơn" — saga *yếu hơn* về đảm bảo (mất isolation); nó thắng ở availability và coupling.
- "Kafka transactions cho tôi exactly-once mọi nơi" — chỉ trong phạm vi Kafka (read-process-write giữa các topic); ra ngoài (DB, API) vẫn cần outbox/idempotency.
- "2PC đã chết" — nó đang chạy trong Spanner/CockroachDB bạn dùng; chết là XA app-level.
- "Idempotency chỉ cần ở API công khai" — mọi consumer message đều cần, vì at-least-once là mặc định của thế giới.

### Câu hỏi tự kiểm tra
1. Thiết kế saga đặt vé máy bay: giữ chỗ → trừ tiền → xuất vé. Chỉ rõ compensation từng bước, trạng thái trung gian user thấy, và xử lý "trừ tiền timeout không rõ kết quả" (gợi ý: query trạng thái PSP + idempotency key, không phải đoán).
2. Vì sao outbox relay publish trùng là chấp nhận được còn publish sót là không? Điều đó quyết định thiết kế relay thế nào (at-least-once, đánh dấu sau publish)?
3. Hệ của bạn có luồng chạm 4 service. Trình bày 2 cách vẽ lại ranh giới để còn 1–2 service, và khi nào việc đó *không* nên làm.
4. Idempotency key lưu ở Redis TTL 1h có đủ không? Kịch bản nào xuyên thủng? (Redis mất dữ liệu, retry sau 1h, key theo request thay vì ý định.)

### Tài liệu kinh điển nên đọc
- **"Sagas" (Garcia-Molina & Salem, 1987)** — paper gốc, ngắn đến ngạc nhiên; long-lived transaction là bài toán từ trước microservices 30 năm.
- **"Life beyond Distributed Transactions" (Pat Helland, 2007)** — bản tuyên ngôn: entity, idempotency, at-least-once — nền tư duy của toàn chương; có lẽ paper đáng đọc nhất cho backend engineer hiện đại.
- **"Transactional Outbox" & "Saga" trên microservices.io (Chris Richardson)** — tài liệu tham chiếu pattern thực dụng.
- **Stripe engineering blog về Idempotency** — chuẩn công nghiệp của idempotency key, từ người vận hành tiền thật.
- **"Large-scale Incremental Processing" (Google Percolator, 2010)** — 2PC trên Bigtable: tổ tiên của TiDB; đọc để hiểu 2PC hiện đại hóa thế nào.
