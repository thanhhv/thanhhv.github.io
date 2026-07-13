+++
title = "13.3. Messaging Failures"
date = "2026-07-13T16:40:00+07:00"
draft = false
tags = ["backend", "system-design"]
series = ["System Design — Tư Duy Thiết Kế Hệ Thống"]
+++

> Bốn tình huống: Kafka Lag, Message Duplication, Queue Backlog, Retry Storm. Hệ messaging đổi "fail ngay trước mặt" lấy "fail âm thầm phía sau" — sự cố của nó vì thế thường được phát hiện muộn, khi backlog đã thành núi.

---

## Case 10 — Kafka Lag (Consumer Lag)

### Triệu chứng

Dữ liệu downstream "cũ dần": dashboard analytics trễ 40 phút, projection CQRS lệch nguồn sự thật, email gửi sau sự kiện 2 giờ. Metric `consumer group lag` tăng — quan trọng nhất là **tăng đơn điệu không hồi**.

### Root cause & tại sao xảy ra

Lag = offset mới nhất − offset đã xử lý. Nó tăng khi **tốc độ tiêu thụ < tốc độ sản xuất**, do: consumer chậm đi (deploy mới có bug hiệu năng, dependency của consumer chậm — DB nó ghi vào, API nó gọi); producer tăng tốc (campaign, backfill); rebalance liên tục (consumer join/leave lặp → cả group dừng xử lý mỗi lần — thường do `max.poll.interval.ms` bị vượt vì xử lý batch quá lâu, tạo vòng luẩn quẩn: chậm → bị đá → rebalance → càng chậm); hoặc hot partition ([13.2](/series/system-design/13-production-failure-cases/02-database-failures/)) — lag chỉ ở 1 partition.

Điểm hiểm của lag: hệ thống "vẫn chạy" — không lỗi, không sập — chỉ *trễ dần*. Không ai thấy cho đến khi user hỏi "sao số liệu hôm qua chưa có?"

### Kiến trúc bị ảnh hưởng

Mọi kiến trúc event-driven ([giai đoạn 7–8, Phần 12](/series/system-design/12-evolution/07-kafka-event-driven/)); đặc biệt nguy hiểm với CQRS (projection lag = user thấy dữ liệu sai) và các consumer có side effect thời gian nhạy cảm (thông báo, fraud detection — phát hiện gian lận trễ 2 giờ là vô dụng).

### Metric / Dashboard / Alert

- Metric: lag theo group **và theo partition** (tổng che mất hot partition); *tốc độ đổi* của lag (đạo hàm — quan trọng hơn giá trị); consume rate vs produce rate; số rebalance/giờ; processing time per record.
- Dashboard: lag từng group theo thời gian; produce vs consume rate chồng lên nhau; rebalance events đánh dấu trên trục thời gian.
- Alert: (1) lag vượt SLA nghiệp vụ quy ra thời gian (lag_seconds = lag / consume_rate); (2) lag tăng đơn điệu 15–30 phút; (3) rebalance > N lần/giờ.

### Điều tra

1. Lag ở mọi partition hay một? (một → hot key; mọi → consumer chậm toàn cục).
2. Consume rate rơi hay produce rate tăng? — hai hướng điều tra khác hẳn nhau.
3. Consume rate rơi từ lúc nào → trùng deploy nào của consumer / sự cố nào của dependency consumer?
4. Log rebalance: group có ổn định không?

### Khắc phục & phòng tránh

- **Cầm máu:** scale consumer (chỉ hiệu quả đến số partition — trần song song của Kafka = số partition!); nếu nghẽn ở dependency → tăng batch ghi, bỏ bước không thiết yếu; nếu dữ liệu cũ hết giá trị → **chấp nhận skip** (reset offset về gần đuôi) — quyết định nghiệp vụ nên bàn trước.
- **Chữa gốc:** tối ưu consumer (batch processing, ghi DB theo batch, parallel trong một consumer có kiểm soát thứ tự); tăng số partition *trước khi cần* (giảm partition không làm được dễ dàng); tách consumer nhanh/chậm ra group riêng.
- **Phòng:** benchmark consumer khi viết (records/s trần là bao nhiêu?); capacity đệm ≥ 2× produce rate đỉnh; hợp đồng SLA độ tươi cho từng consumer, có alert theo đúng SLA đó.

---

## Case 11 — Message Duplication

### Triệu chứng

Hành động nghiệp vụ xảy ra ≥ 2 lần: hai email giống nhau, trừ kho hai lần, cộng điểm hai lần, hai bản ghi cùng nội dung khác id. Thường rộ lên **trong/sau sự cố khác** (restart, rebalance, network chớp) — lúc mọi người đang bận nhìn chỗ khác.

### Root cause & tại sao xảy ra

**At-least-once là hợp đồng mặc định của mọi hệ messaging thực tế**, và duplicate là mặt còn lại của đồng xu đó. Cơ chế sinh duplicate ở cả hai phía:

- *Producer:* gửi xong không nhận được ack (network chớp) → gửi lại → broker có 2 bản. (Kafka idempotent producer xử được trong một session, không xử được qua restart ứng dụng nếu logic app tự retry.)
- *Consumer:* xử lý xong nhưng **crash trước khi commit offset/ack** → message được giao lại cho consumer khác → xử lý lần hai. Cửa sổ "đã làm xong việc nhưng chưa kịp khai" này không thể thu về 0 — đây là hệ quả của việc "xử lý" và "ghi nhận đã xử lý" là hai thao tác trên hai hệ thống khác nhau, không có transaction chung (chính là bài toán two generals).

Vì sao không dùng exactly-once? Kafka có exactly-once *trong phạm vi Kafka→Kafka (transactions)*; nhưng end-to-end với side effect ngoài (gửi email, gọi API bank) thì **exactly-once là bất khả thi về nguyên lý** — chỉ có at-least-once + idempotency mô phỏng nó.

### Kiến trúc bị ảnh hưởng

Mọi hệ dùng queue/log ([giai đoạn 3+, Phần 12](/series/system-design/12-evolution/03-background-worker/)). Sát thương tỷ lệ với độ "không đảo ngược được" của side effect: gửi 2 email = xấu hổ; trừ tiền 2 lần = khủng hoảng.

### Metric / Dashboard / Alert

- Metric: **duplicate hit rate** — số lần idempotency check chặn được bản trùng (đo được nếu có idempotency — chính metric này chứng minh cơ chế hoạt động); redelivery count của broker; tỷ lệ unique constraint violation "hiền" trong DB.
- Alert: duplicate rate tăng đột biến (chỉ điểm sự cố ở producer/consumer đang diễn ra); *và* alert nghiệp vụ đối soát (tổng tiền trừ ≠ tổng đơn) làm lưới cuối.

### Điều tra

1. Bản trùng cách nhau bao lâu? (ms → producer retry; = thời gian visibility/rebalance → consumer redelivery).
2. Trùng đúng lúc có sự kiện gì? (deploy consumer, rebalance, network).
3. Idempotency có tồn tại ở consumer đó không, và tại sao nó không chặn? (thường: key idempotency chọn sai phạm vi — theo message-id thay vì theo business-operation-id, nên "cùng nghiệp vụ, hai message" lọt lưới).

### Khắc phục & phòng tránh

- **Khắc phục sự vụ:** xác định tập bản ghi bị nhân đôi (query theo cửa sổ thời gian + tiêu chí trùng) → bù trừ nghiệp vụ (hoàn điểm, hoàn kho) → thông báo minh bạch nếu chạm user.
- **Phòng — idempotency là câu trả lời duy nhất, làm ở tầng consumer:**
  - **Idempotency key theo nghiệp vụ** (`order_id + action`, không phải message-id) + bảng `processed_operations` với unique constraint; check-and-insert **trong cùng transaction** với side effect DB.
  - Side effect ngoài DB (email, API bên thứ ba): gửi kèm idempotency key để *bên kia* dedupe (Stripe-style) — nếu bên kia không hỗ trợ, ghi trạng thái "đang gửi" trước khi gọi + đối soát sau.
  - Thiết kế thao tác **tự nhiên idempotent** khi có thể: `SET status='paid'` thay vì `INCREMENT balance` — thao tác gán tuyệt đối chạy N lần vẫn đúng.
  - Văn hóa: mọi code review consumer hỏi đúng một câu — *"chạy hai lần thì sao?"*

---

## Case 12 — Queue Backlog

### Triệu chứng

Queue depth tăng liên tục; tuổi message già nhất tính bằng giờ; job "đặt hàng xong gửi email" chạy sau 4 tiếng; RabbitMQ bật flow control / memory alarm; disk broker đầy dần. Khác Kafka lag ở chỗ: queue truyền thống *giữ* message chưa xử lý trong RAM/disk của broker — backlog to làm **chính broker** ốm (Kafka thì retention theo thời gian, broker vô can, chỉ consumer trễ).

### Root cause & tại sao xảy ra

Cùng phương trình với Kafka lag — tiêu thụ < sản xuất — cộng thêm các ngòi riêng: **poison message** (message lỗi làm consumer crash → requeue → crash → consumer chết lặp còn queue thì phình — một message độc chặn cả dòng); consumer bị dependency ghìm (DB chậm → mọi consumer chậm đồng loạt); và **backlog tự nuôi**: queue càng dài, RAM broker càng căng, broker càng chậm, tiêu thụ càng chậm.

Điểm nghiệp vụ hiểm: job nằm queue 4 giờ chạy trên **thế giới đã thay đổi** — email "đơn hàng đã xác nhận" đến sau khi đơn đã hủy. Backlog không chỉ là trễ — nó là *sai dần theo độ trễ*.

### Kiến trúc bị ảnh hưởng

Mọi hệ dùng work queue ([giai đoạn 3–4, Phần 12](/series/system-design/12-evolution/04-message-queue/)).

### Metric / Dashboard / Alert

- Metric: queue depth từng queue; **message age (tuổi message già nhất)** — nghiệp vụ hơn depth; consumer utilisation; ack rate vs publish rate; redelivery count từng message (poison có count cao); RAM/disk broker.
- Dashboard: depth + age + publish/ack rate chồng trục thời gian, từng queue.
- Alert: age > SLA của loại job đó (email: 15 phút; analytics: 2 giờ); depth tăng đơn điệu 15 phút; broker memory > 70%.

### Điều tra

1. Publish tăng hay ack giảm? → hai nhánh.
2. Ack giảm: consumer sống không? Đang crash-loop vì một message (đọc log — poison)? Hay chậm vì dependency (trace một job mẫu)?
3. Redelivery count: message nào được giao đi giao lại nhiều nhất — thủ phạm poison lộ ngay.

### Khắc phục & phòng tránh

- **Cầm máu:** poison → móc message đó sang DLQ bằng tay, dòng chảy thông lại ngay; scale consumer; job đã mất giá trị theo thời gian → **purge có chọn lọc** (quyết định nghiệp vụ — email chào mừng trễ 2 ngày nên vứt, không nên gửi); tạm dừng producer không thiết yếu.
- **Chữa gốc:** max delivery attempts + DLQ **tự động** (không bao giờ để requeue vô hạn); TTL theo loại job — job hết đát tự rơi; consumer kiểm tra "điều kiện còn hiệu lực không" trước khi làm (đơn còn tồn tại? user còn active?) — vì thế giới đã trôi kể từ lúc enqueue.
- **Phòng:** load test consumer; capacity ≥ 2× peak; DLQ có dashboard + quy trình xử lý (DLQ không ai xem = hố đen nuốt nghiệp vụ); tách queue theo mức ưu tiên (email marketing không xếp chung hàng với xác nhận thanh toán).

---

## Case 13 — Retry Storm

### Triệu chứng

Một dependency chậm/lỗi *nhẹ* → traffic nội bộ **tăng vọt gấp nhiều lần** → dependency từ ốm thành chết hẳn → và **giữ nguyên trạng thái chết ngay cả khi nguyên nhân gốc đã hết** — vì chính lượng retry duy trì quá tải. Chữ ký: đồ thị request đến dependency cao gấp 3–10 lần request từ user thật; sự cố "không chịu tự hết".

### Root cause & tại sao xảy ra

Retry là khuếch đại: 1 request fail → 3 attempt. Xếp tầng là lũy thừa: client retry 3 × gateway retry 3 × service A retry 3 = **27 request** đập vào service B cho 1 ý định của user. Khi B chậm vì quá tải, retry *thêm tải cho hệ đang chết vì tải* — thuốc trở thành thuốc độc. Không có jitter, các retry còn đồng nhịp thành sóng ([thundering herd](/series/system-design/13-production-failure-cases/01-caching-failures/)). Đây là mô hình khuếch đại + vòng lặp dương thuần khiết nhất trong cả tài liệu.

### Kiến trúc bị ảnh hưởng

Microservices nhiều tầng gọi nhau là môi trường lý tưởng ([giai đoạn 6, Phần 12](/series/system-design/12-evolution/06-microservices/)) — mỗi tầng "tử tế" thêm retry của riêng mình, không ai nhìn thấy tích số. Cũng xảy ra client-mobile → API (triệu client cùng retry).

### Metric / Dashboard / Alert

- Metric: **retry rate tách khỏi request rate** (phải đo riêng attempt đầu vs retry — không tách thì mù); tỷ lệ retry/first-attempt; request rate nội bộ vs request rate từ edge (chênh lệch lớn = đang khuếch đại).
- Dashboard: cây gọi service với retry rate từng cạnh.
- Alert: retry ratio > 10–20%; internal/external request ratio tăng bất thường.

### Điều tra

1. Dependency nào là tâm bão (error rate cao nhất, sâu nhất trong chuỗi)?
2. Đếm tầng retry từ edge đến tâm: tích khuếch đại là bao nhiêu?
3. Nguyên nhân gốc ban đầu còn không, hay giờ chỉ còn cơn bão tự duy trì? (nhiều retry storm sống lâu hơn nguyên nhân gốc hàng giờ).

### Khắc phục & phòng tránh

- **Cầm máu:** **cắt khuếch đại trước, chữa gốc sau** — hạ retry về 0–1 ở các tầng giữa (config runtime — vì thế retry policy nên là config nóng, không phải hằng số compile); bật/hạ ngưỡng circuit breaker; shed load ở edge (429). Dependency thường tự hồi trong vài phút khi cơn mưa retry tạnh.
- **Chữa gốc — bốn kỷ luật retry:**
  1. **Retry ở một tầng duy nhất** (thường tầng gần user hoặc tầng hiểu nghiệp vụ nhất) — các tầng giữa fail-fast và truyền lỗi lên. Tích khuếch đại toàn hệ thống phải là con số được *thiết kế*, không phải tích ngẫu nhiên của các quyết định cục bộ.
  2. **Exponential backoff + full jitter** — luôn luôn.
  3. **Retry budget** (kiểu Google SRE: retry ≤ 10% tổng request tới một backend; vượt → ngừng retry) — chốt chặn tuyệt đối cho hệ số khuếch đại.
  4. **Circuit breaker** ở mọi client ra ngoài process: backend lỗi liên tục → mở mạch, fail-fast, thử lại nhỏ giọt (half-open). CB chính là "công tắc tự động cắt vòng lặp dương".
  - Chỉ retry lỗi *đáng retry* (timeout, 503, deadlock) — không retry 400/401/422 (retry cái chắc chắn fail = khuếch đại thuần túy).
- **Phòng:** chaos test "dependency chậm 10×" và xem tổng traffic nội bộ tăng bao nhiêu — con số đó là hệ số khuếch đại thật của kiến trúc; đưa retry policy vào service template chuẩn để không team nào tự sáng tác.

---

## Tổng kết nhóm messaging

| Case | Bản chất | Câu hỏi phòng ngừa |
|---|---|---|
| Kafka Lag | Tiêu thụ < sản xuất, âm thầm | "Consumer này trần bao nhiêu records/s? Ai canh độ tươi?" |
| Duplication | At-least-once là hợp đồng, duplicate là điều khoản | "Chạy hai lần thì sao?" |
| Queue Backlog | Trễ = sai dần; poison chặn dòng | "Job này hết đát sau bao lâu? DLQ ai xem?" |
| Retry Storm | Khuếch đại × vòng lặp dương | "Tích retry từ edge vào đây là bao nhiêu?" |

Bốn case chung một chân lý: **async không xóa lỗi — nó hoãn lỗi và đổi hình dạng lỗi.** Hệ sync fail to và ngay trước mặt; hệ async fail nhỏ, muộn, và ở góc khuất. Giám sát của hệ async vì thế phải chủ động gấp đôi: đo độ trễ, độ trùng, độ già — những thứ hệ sync không cần từ vựng để gọi tên.
