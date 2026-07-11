+++
title = "Bài 9 — System Design Case Studies"
date = "2026-07-03T16:00:00+07:00"
draft = false
tags = ["backend", "interview"]
series = ["Backend Interview"]
+++

> Các chương trước dạy **nguyên liệu** (cache, queue, shard, replication...). Chương này dạy cách **nấu** — dẫn dắt một buổi system design 45 phút end-to-end. Khác biệt cốt lõi: system design không kiểm tra bạn biết bao nhiêu công nghệ, mà kiểm tra bạn **dẫn dắt sự mơ hồ thành một thiết kế có cơ sở** thế nào. Interviewer đóng vai khách hàng mơ hồ; bạn phải hỏi đúng, ước lượng, vẽ, rồi đào sâu.

---

## Phần 0 — Framework: cách dẫn dắt 45 phút (đọc trước mọi case)

Đây không phải một "câu hỏi" mà là bộ khung áp cho mọi bài. Interviewer chấm bạn theo **quá trình**, không theo đáp án cuối.

**Bước 1 — Làm rõ yêu cầu (5–8 phút, ĐỪNG bỏ qua).** Đây là bước phân biệt Senior với junior rõ nhất. Junior nhảy vào vẽ ngay; Senior hỏi trước:
- Functional: hệ thống làm chính xác những gì? Tính năng nào trong scope, ngoài scope?
- Non-functional: quy mô (bao nhiêu user, RPS)? Tỷ lệ đọc/ghi? Latency yêu cầu? Consistency cần mạnh hay eventual chấp nhận được? Availability mục tiêu?
- Chốt scope bằng lời: "Vậy em sẽ tập trung X, Y; bỏ qua Z, đúng không ạ?" — thể hiện kiểm soát phạm vi.

**Bước 2 — Ước lượng quy mô (3–5 phút).** Con số định hình kiến trúc. Ví dụ: 100M user, 10% active/ngày = 10M DAU; mỗi người 10 request = 100M req/ngày ≈ **~1160 RPS trung bình, peak ~3–5x ≈ 5000 RPS**. Storage: 100M record × 1KB = 100GB/năm. Băng thông, số kết nối. Đừng tính quá chi tiết — mục đích là biết "một máy đủ không, hay cần shard/cache" (thứ tự độ lớn quan trọng, không phải số lẻ).

**Bước 3 — API + Data model (3–5 phút).** Định nghĩa vài endpoint chính (chữ ký, không cần đầy đủ) và schema cốt lõi. Điều này neo thiết kế vào thực tế và lộ ra access pattern (quyết định index/shard key).

**Bước 4 — High-level design (5–8 phút).** Vẽ các thành phần chính: client → LB → service → DB/cache/queue. Nói luồng của một request tiêu biểu đi qua đâu. Giữ ở mức khối, chưa đào sâu.

**Bước 5 — Đào sâu (15–20 phút, phần chính).** Interviewer sẽ chỉ vào một điểm ("phần này scale thế nào?", "node này chết thì sao?"). Đây là lúc dùng mọi thứ từ 8 chương trước: cache pattern, shard key, replication, saga, backpressure. Chủ động nêu bottleneck và trade-off.

**Bước 6 — Bàn về bottleneck, failure, và mở rộng (5 phút).** "Nếu scale 100x thì vỡ ở đâu?", SPOF, và các đánh đổi bạn đã chọn.

**Nguyên tắc vàng xuyên suốt:** (1) **nghĩ thành tiếng** — interviewer chấm tư duy, im lặng vẽ là mất điểm; (2) **luôn nêu trade-off** — mọi lựa chọn phải kèm "được gì, mất gì, tại sao chọn"; (3) **bắt đầu đơn giản rồi mới scale** — đừng vẽ 20 microservice + Kafka + K8s cho một bài mà một Postgres + cache giải được; over-engineering bị trừ điểm ngang under-engineering; (4) **dẫn dắt, đừng chờ hỏi** — chủ động đề xuất bước tiếp theo.

---

## Case 1 — [Intermediate → Senior] Thiết kế Distributed Rate Limiter

### 1. Câu hỏi
"Thiết kế một rate limiter phân tán: giới hạn mỗi user 100 request/phút, hoạt động đúng qua nhiều server API. Trình bày thuật toán và cách nó chính xác khi có N instance."

### 2. Interviewer muốn kiểm tra điều gì?
- Biết các thuật toán (token bucket, sliding window) và trade-off, không chỉ một cách.
- Xử lý được phần "phân tán" — nơi bài toán trở nên khó (shared state, race condition, atomicity).
- Cân bằng chính xác vs hiệu năng vs độ phức tạp.

### 3. Câu trả lời ngắn gọn (30 giây)
"Thuật toán tôi chọn thường là **token bucket** (cho phép burst, mượt, ít bộ nhớ) hoặc **sliding window counter** (chính xác hơn về ranh giới thời gian). Với phân tán: state (số token/counter per user) phải chia sẻ giữa các instance → lưu ở **Redis**, và thao tác tăng-kiểm-tra phải **nguyên tử** (Lua script hoặc INCR + EXPIRE) để tránh race khi nhiều instance cùng cập nhật. Trade-off cốt lõi: chính xác tuyệt đối cần mỗi request round-trip tới Redis (latency + Redis là dependency nóng); muốn nhanh hơn thì mỗi instance giữ quota cục bộ (local token bucket) và đồng bộ lỏng với Redis — chấp nhận vượt giới hạn một chút. Chọn theo mức độ nghiêm ngặt của giới hạn."

### 4. Trình bày Senior Level (walkthrough)
**Làm rõ:** giới hạn theo gì (user? IP? API key?)? Vượt thì reject (429) hay queue? Cần chính xác tuyệt đối hay xấp xỉ chấp nhận được? Giới hạn cho chống abuse hay chống quá tải (hai mục tiêu khác nhau)?

**Thuật toán (nêu và so sánh):**
- **Fixed window counter:** đếm request trong cửa sổ cố định (mỗi phút reset). Đơn giản, ít bộ nhớ. Nhược: **burst ở ranh giới** — 100 req cuối phút 1 + 100 req đầu phút 2 = 200 req trong ~vài giây quanh ranh giới, vi phạm tinh thần giới hạn.
- **Sliding window log:** lưu timestamp mọi request, đếm trong cửa sổ trượt. Chính xác tuyệt đối. Nhược: tốn bộ nhớ (lưu mọi timestamp), đắt.
- **Sliding window counter:** xấp xỉ cửa sổ trượt bằng nội suy hai fixed window kề nhau. Cân bằng tốt — chính xác gần tuyệt đối, bộ nhớ nhỏ. Phổ biến nhất.
- **Token bucket:** bucket có N token, đổ đầy với tốc độ cố định; mỗi request tiêu một token, hết token thì reject. Cho phép **burst** (dùng token tích lũy) — thường mong muốn (user thật hay burst). Ít bộ nhớ (chỉ lưu token count + last refill time).
- **Leaky bucket:** như hàng đợi xả đều — làm mượt output, dùng khi cần rate ổn định tuyệt đối cho downstream.

**Phần phân tán (trọng tâm):**
- State chia sẻ ở Redis. Vấn đề race: instance A và B cùng đọc "còn 1 token", cùng cho qua → vượt giới hạn. Fix: **atomicity** — Lua script chạy đọc-tính-ghi nguyên tử trong Redis (single-threaded nên script chạy trọn vẹn), hoặc `INCR` + `EXPIRE` cho counter.
- **Trade-off latency:** mỗi request +1 RTT tới Redis. Với giới hạn lỏng, dùng **local rate limiting** per instance (chia quota: 10 instance thì mỗi cái 10 req/phút) — nhanh, không dependency, nhưng không chính xác (tải lệch giữa instance) và không xử lý được instance thêm/bớt động. Hoặc **hybrid**: local bucket + đồng bộ định kỳ với Redis (cho phép sai số nhỏ).
- **Redis là SPOF nóng:** nó chết thì rate limiter chết — fail-open (cho qua hết, mất bảo vệ) hay fail-closed (chặn hết, mất service)? Quyết định theo mục tiêu: chống abuse thì fail-open chấp nhận được; chống quá tải thì có thể cần fail-closed hoặc fallback local.

### 5. Giải thích bản chất
Rate limiting là bài toán **đo một tốc độ (đại lượng liên tục theo thời gian) bằng các quan sát rời rạc, dưới ràng buộc bộ nhớ**. Mọi thuật toán là một cách xấp xỉ "tốc độ trong cửa sổ trượt" với chi phí bộ nhớ khác nhau: log lưu tất cả (chính xác, tốn), fixed window nén thành một số (rẻ, sai ở ranh giới), sliding counter nội suy (cân bằng), token bucket mô hình hóa bằng vật lý dòng chảy (tự nhiên cho burst). Phần "phân tán" quy về một nguyên lý đã lặp khắp bộ tài liệu: **trạng thái chia sẻ + cập nhật đồng thời = cần atomicity**, và atomicity phân tán phải trả bằng round-trip tới một điểm tuần tự hóa (Redis single-thread) — lại là trade-off "tập trung để đúng vs phân tán để nhanh". Không có lời giải miễn phí; chỉ có chọn điểm trên trục chính xác–latency–phức tạp khớp với việc giới hạn này bảo vệ cái gì.

### 6. Trade-off
- **Token bucket:** cho burst, mượt, ít RAM ↔ cấu hình burst size cần cân nhắc (burst lớn = bảo vệ yếu).
- **Sliding window counter:** chính xác, ít RAM ↔ phức tạp hơn fixed window một chút.
- **Redis tập trung:** chính xác toàn cục ↔ +1 RTT/request, SPOF nóng.
- **Local limiting:** nhanh, không dependency ↔ không chính xác (tải lệch), khó với autoscale.
- **Fail-open vs fail-closed** khi Redis chết: giữ service vs giữ bảo vệ.

### 7. Ví dụ Production
Sự cố thật: rate limiter dùng fixed window 1 phút, chống spam OTP. Kẻ tấn công canh ranh giới phút: gửi 100 OTP lúc 10:00:59 và 100 lúc 10:01:00 → 200 SMS trong 1 giây → hóa đơn SMS tăng vọt và nhà mạng chặn. Fix: chuyển sang sliding window counter. Bài học: fixed window boundary burst không phải lỗ hổng lý thuyết — nó bị khai thác thật. Ví dụ hiệu năng: một API để rate limiter gọi Redis đồng bộ trên mọi request, Redis thành nút cổ chai p99; fix bằng hybrid local+Redis cho endpoint chịu tải cao (chấp nhận sai số ±5%) và giữ Redis-chính-xác cho endpoint nhạy cảm (thanh toán).

### 8. Những câu trả lời chưa đủ tốt
- "Đếm request trong Redis với INCR." → Fixed window — burst boundary? Race? Atomicity? Chưa đủ.
- "Dùng token bucket." (rồi dừng) → Còn phần phân tán? Race giữa instance? Redis chết thì sao? Thuật toán chỉ là một nửa bài.

### 9. Sai lầm phổ biến của ứng viên
- Bỏ qua race condition khi nhiều instance cập nhật — không dùng atomic operation.
- Không biết fixed window boundary burst.
- Không bàn Redis là SPOF và fail-open/closed.
- Quên EXPIRE → key rác tích tụ.
- Không hỏi giới hạn để làm gì (abuse vs overload) — quyết định fallback sai.

### 10. Follow-up Questions
- Scale lên 1M RPS — Redis một node đủ không? Shard rate limit theo user key thế nào?
- Giới hạn phân cấp (per user VÀ per API key VÀ global) — thiết kế ra sao?
- Client cần biết còn bao nhiêu quota — trả header gì? (X-RateLimit-Remaining, Retry-After.)
- Giới hạn khác nhau theo tier (free/paid) — cấu hình động không cần deploy?
- So sánh với cách API Gateway (Kong, Envoy) làm rate limiting sẵn — khi nào tự xây?

### 11. Liên hệ với Production
Mọi API public (Stripe, GitHub, Twitter) đều có rate limiter tinh vi với header quota chuẩn; thường đặt ở API gateway/edge. Vấn đề nghiêm trọng khi: chống abuse tốn tiền thật (SMS, email, compute đắt), hoặc bảo vệ downstream yếu khỏi quá tải. Dấu hiệu cần rà soát: chi phí do abuse tăng, downstream bị dội khi có traffic bất thường, và rate limiter tự nó thành bottleneck (Redis nóng).

---

## Case 2 — [Senior] Thiết kế URL Shortener (bit.ly)

### 1. Câu hỏi
"Thiết kế dịch vụ rút gọn URL như bit.ly: tạo short link, redirect, và chịu được đọc rất nhiều. Tập trung vào sinh short key và scale phần đọc."

### 2. Interviewer muốn kiểm tra điều gì?
- Bài read-heavy kinh điển — test tư duy cache và tỷ lệ đọc/ghi.
- Sinh unique ID phân tán — bài toán con quan trọng (collision, độ dài, đoán được).
- Ước lượng quy mô và chọn storage phù hợp.

### 3. Câu trả lời ngắn gọn (30 giây)
"Hai thao tác: tạo (ghi, ít) và redirect (đọc, cực nhiều — tỷ lệ đọc/ghi có thể 100:1 hoặc hơn). Sinh short key: base62 (a-zA-Z0-9) của một ID unique — 7 ký tự cho ~3500 tỷ URL. Sinh ID cách nào: counter tập trung (đơn giản, nhưng SPOF), hoặc pre-generate + phát theo range cho mỗi instance, hoặc hash URL rồi xử lý collision. Storage: key-value đơn giản (short→long) — chọn theo quy mô, có thể Postgres + index hoặc KV store. Scale đọc: đây là bài của **cache** — short key cực hợp cache (bất biến sau khi tạo, đọc lặp lại nhiều), CDN + Redis chặn phần lớn traffic trước khi chạm DB. Redirect trả 301/302 tùy có muốn đếm click không."

### 4. Trình bày Senior Level (walkthrough)
**Làm r洗:** custom alias không? Hết hạn link? Analytics (đếm click)? Quy mô? → giả sử 100M URL mới/tháng, đọc gấp 100x.

**Ước lượng:** ghi ~40/s (100M/tháng), đọc ~4000/s trung bình, peak cao hơn. Storage: 100M/tháng × 500 byte × giữ 5 năm ≈ vài TB — không khổng lồ. → Kết luận: ghi nhẹ, đọc nặng, dữ liệu vừa. **Bài toán là scale đọc, không phải ghi.**

**API + data model:** `POST /shorten {long_url} → {short_url}`; `GET /{key} → 301 redirect`. Schema: `{short_key (PK), long_url, created_at, expires_at, user_id}`.

**Sinh short key (bài con quan trọng — so sánh phương án):**
- **Hash URL (MD5/SHA) lấy vài ký tự:** cùng URL ra cùng key (dedup tự nhiên), nhưng collision phải xử lý (thêm salt, thử lại) và không dedup được nếu muốn mỗi lần một link.
- **Counter tập trung + base62:** ID tuần tự → base62. Đơn giản, không collision, key ngắn. Nhược: counter là SPOF/bottleneck, và key **đoán được** (tuần tự → lộ số lượng, crawl được) — thêm bước xáo trộn nếu cần.
- **Range allocation:** một service cấp range (mỗi instance nhận 1 triệu ID, tự phát trong range) → phân tán, không round-trip mỗi lần. Cân bằng tốt.
- **Pre-generated keys:** sinh sẵn hàng loạt key ngẫu nhiên vào bảng "chưa dùng", tạo link = lấy một cái → không collision runtime, không đoán được, nhưng cần job sinh trước.

**Scale đọc (trọng tâm):**
- Short key **bất biến sau tạo** → cache lý tưởng (TTL dài, hit ratio cực cao). Tầng: CDN (edge cache redirect) → Redis (hot keys) → DB. Phần lớn traffic dừng ở CDN/Redis.
- DB read replica cho phần miss cache.
- **301 vs 302:** 301 (permanent) được browser/CDN cache mạnh → giảm tải tối đa, nhưng **không đếm được click** (request không tới server nữa) và không đổi đích được. 302 (temporary) không cache → mọi click tới server (đếm được, đổi đích được) nhưng tải cao hơn. Chọn theo có cần analytics không — trade-off kinh điển của bài này.
- Analytics: nếu cần đếm click, đừng ghi DB đồng bộ mỗi redirect (giết hiệu năng) → bắn event vào queue (Kafka), xử lý async, tổng hợp (nối chương MQ + ClickHouse cho analytics).

### 5. Giải thích bản chất
URL shortener đẹp về mặt sư phạm vì nó cô lập một sự bất đối xứng tinh khiết: **ghi hiếm và bất biến, đọc thường và lặp lại** — đây là điều kiện lý tưởng cho caching, và toàn bộ thiết kế xoay quanh khai thác nó. Bài toán sinh key là một thể hiện của bài toán phân tán tổng quát: **sinh định danh unique mà không cần điểm phối hợp tập trung** (nối UUID, Snowflake ID, shard key) — mọi lời giải là điểm trên trục "phối hợp tập trung (đơn giản, có SPOF) ↔ phân tán (phức tạp, không SPOF)". Trade-off 301/302 dạy một bài sâu hơn: **cache mạnh nhất là cache bạn không kiểm soát** (browser/CDN cache 301) — nó cho hiệu năng tối đa nhưng lấy đi khả năng quan sát và thay đổi; đây là đánh đổi observability-vs-performance xuất hiện ở mọi tầng cache.

### 6. Trade-off
- **Counter tập trung:** key ngắn, không collision ↔ SPOF, đoán được.
- **Random/pre-generated:** không đoán được, phân tán ↔ key có thể dài hơn, cần job sinh/xử lý collision.
- **301:** cache tối đa, tải thấp nhất ↔ không đếm click, không đổi đích.
- **302:** đếm được, linh hoạt ↔ tải cao hơn (mọi click tới server).
- **Đếm click đồng bộ vs async:** chính xác tức thì ↔ giết hiệu năng redirect; async nhanh nhưng eventual.

### 7. Ví dụ Production
Bài học thật về key generation: một team dùng counter tuần tự + base62, không xáo trộn → competitor viết script tăng key tuần tự crawl **toàn bộ** URL đã tạo (bao gồm link nội bộ, tài liệu nhạy cảm user tưởng là private vì "link khó đoán"). Bài học: "security by obscurity" của short link chỉ đúng nếu key **thực sự không đoán được** — tuần tự là đoán được. Về scale: một dịch vụ redirect ghi analytics đồng bộ vào Postgres mỗi click → giờ cao điểm DB ghi bão hòa, redirect chậm 2s. Fix: 302 + bắn event async, redirect chỉ đọc cache → p99 về <10ms, analytics xử lý riêng.

### 8. Những câu trả lời chưa đủ tốt
- "Lưu short→long trong DB rồi query." → Đúng cơ bản nhưng bỏ qua phần chính: scale đọc bằng cache, sinh key thế nào, 301 vs 302.
- "Hash URL làm key." → Collision xử lý sao? Đoán được không? Dedup có mong muốn không?

### 9. Sai lầm phổ biến của ứng viên
- Không nhận ra đây là bài read-heavy → không nhấn cache.
- Bỏ qua trade-off 301/302 (rất đặc trưng của bài này).
- Key tuần tự đoán được mà không nhận ra rủi ro.
- Ghi analytics đồng bộ mỗi redirect.
- Không ước lượng để thấy dữ liệu không lớn (over-engineer với shard khi chưa cần).

### 10. Follow-up Questions
- Scale 100x đọc — CDN + cache đủ chưa? Cache miss storm khi key hot mới tạo (nối cache stampede)?
- Custom alias: xử lý va chạm với key tự sinh thế nào?
- Link hết hạn: xóa lazy hay job dọn? Redirect key đã hết hạn trả gì?
- Analytics realtime (đếm click theo thời gian, theo địa lý) — pipeline thế nào? (nối Kafka + ClickHouse.)
- Nếu cần global (user toàn cầu) — đặt cache/DB đa region thế nào? (nối multi-region.)

### 11. Liên hệ với Production
bit.ly, TinyURL và mọi link shortener trong sản phẩm (share link mạng xã hội) chạy mô hình này; phần đọc gần như hoàn toàn dựa CDN + cache. Vấn đề nghiêm trọng khi: một link viral (một key nhận triệu request/phút — hot key, nối chương cache), hoặc cần analytics realtime quy mô lớn. Dấu hiệu tối ưu: cache hit ratio, tải DB từ redirect (nên gần 0 nếu cache tốt), và độ trễ pipeline analytics.

---

## Case 3 — [Senior → Staff] Thiết kế Notification System (push/email/SMS đa kênh)

### 1. Câu hỏi
"Thiết kế hệ thống gửi thông báo đa kênh (push, email, SMS, in-app) cho hàng triệu user. Đảm bảo gửi đáng tin cậy, không spam trùng, và xử lý được fan-out lớn (một sự kiện → hàng triệu người nhận)."

### 2. Interviewer muốn kiểm tra điều gì?
- Kiến trúc event-driven và async ở quy mô — nối trực tiếp chương MQ, EDA.
- Xử lý reliability: retry, idempotency, dead letter, thứ tự, dedup.
- Fan-out lớn và tích hợp bên thứ ba không đáng tin (nhà mạng SMS, APNs/FCM).

### 3. Câu trả lời ngắn gọn (30 giây)
"Trục chính là **event-driven + queue**: service nghiệp vụ phát event ('order shipped'), notification service tiêu thụ, quyết định kênh + template + người nhận, rồi đẩy vào các queue per-kênh; worker mỗi kênh gọi provider tương ứng (FCM/APNs, SMTP/SendGrid, SMS gateway). Tách queue per kênh để cô lập: SMS gateway chậm không chặn push. Reliability: at-least-once + **idempotency** (dedup theo notification ID để không gửi trùng khi retry), retry với backoff, DLQ cho cái fail vĩnh viễn. Fan-out lớn (1 event → 1M người): không xử lý tuần tự — chia batch, nhiều worker, và tách 'quyết định gửi cho ai' (fan-out) khỏi 'gửi thật' (delivery). Thêm: user preference (opt-out, quiet hours), rate limit per user (chống spam)."

### 4. Trình bày Senior Level (walkthrough)
**Làm rõ:** kênh nào? Realtime (transactional: OTP, order) hay bulk (marketing)? Cần đảm bảo delivery tới đâu (OTP phải tới, marketing thì best-effort)? Quy mô fan-out? Cần dedup/user preference/quiet hours không?

**Kiến trúc luồng:**
1. **Trigger:** service nghiệp vụ phát event vào Kafka (dùng outbox pattern — nối chương EDA, đảm bảo không mất event khi DB commit nhưng publish fail).
2. **Notification service (consumer):** nhận event → resolve người nhận + kênh + template + preference. Đây là nơi fan-out xảy ra (1 event "flash sale" → query ra 1M user quan tâm).
3. **Fan-out:** với fan-out lớn, tách thành job: chia người nhận thành batch, đẩy từng batch vào queue → nhiều worker xử lý song song. Đừng để một consumer xử lý 1M người tuần tự (head-of-line blocking — nối Kafka).
4. **Queue per kênh:** push-queue, email-queue, sms-queue riêng — cô lập tốc độ và failure (SMS provider chậm không ảnh hưởng push).
5. **Delivery worker per kênh:** gọi provider (FCM/APNs, SendGrid, Twilio). Xử lý: retry + backoff khi provider lỗi tạm; DLQ khi fail vĩnh viễn (token chết, số không tồn tại); rate limit theo giới hạn của provider.

**Reliability (trọng tâm Staff):**
- **Idempotency:** mỗi notification có ID unique; delivery worker kiểm tra "đã gửi chưa" (Redis/DB) trước khi gửi → retry không gây gửi trùng. Đây là điều bắt buộc vì at-least-once là mặc định (nối Kafka).
- **Dedup logic:** cùng một sự kiện có thể trigger nhiều lần (bug upstream, retry) → dedup theo (user, event_type, thời gian) để không spam.
- **Xử lý provider không đáng tin:** APNs/FCM/SMS gateway thường xuyên lỗi tạm, rate limit, hoặc trả kết quả bất đồng bộ (SMS "đã nhận" ≠ "đã gửi tới máy"). Cần: timeout, retry, circuit breaker (provider chết thì ngừng dội — nối chương HA), và xử lý callback trạng thái async.
- **Ưu tiên:** OTP/transactional phải đi trước marketing — queue riêng hoặc priority, không để chiến dịch marketing 1M tin làm nghẽn OTP đăng nhập.

**User-facing concerns:** preference (opt-out per kênh, per loại), quiet hours (không gửi 2h sáng), rate limit per user (không quá N notification/ngày — chống spam làm user tắt thông báo).

### 5. Giải thích bản chất
Notification system là **bài toán tích hợp hệ thống không đáng tin ở quy mô lớn**, và bản chất nằm ở chỗ: bạn không kiểm soát điểm cuối (thiết bị user, nhà mạng, mail server) — chúng chậm, lỗi, và trả kết quả mơ hồ ("đã nhận để gửi" không phải "đã tới tay"). Vì thế mọi thiết kế đúng đều quy về hai nguyên lý đã lặp khắp bộ tài liệu: (1) **async + queue để tách sự đáng tin của bạn khỏi sự không đáng tin của bên ngoài** — event vào queue là "đã ghi nhận", việc gửi thật diễn ra sau với retry, thất bại một provider không mất event; (2) **at-least-once + idempotency** — vì không thể đảm bảo exactly-once qua ranh giới hệ thống không kiểm soát, ta chấp nhận gửi có thể lặp và làm cho lặp trở nên vô hại. Fan-out lớn là bài toán **tách quyết định khỏi thực thi**: "gửi cho ai" (một phép query lớn) và "gửi thật" (triệu thao tác I/O độc lập) có đặc tính scale khác nhau nên phải tách tầng — cùng tư duy tách read model/write model của CQRS.

### 6. Trade-off
- **Queue per kênh:** cô lập failure/tốc độ ↔ nhiều queue để quản, logic routing phức tạp hơn.
- **At-least-once + idempotency:** không mất notification ↔ phải xây dedup, lưu trạng thái đã-gửi (thêm state).
- **Priority queue (OTP trước marketing):** transactional không bị nghẽn ↔ phức tạp hơn queue phẳng.
- **Đảm bảo delivery mạnh:** OTP chắc tới ↔ chi phí retry/tracking cao — không phải kênh nào cũng cần (marketing best-effort là đủ, đừng trả giá cho nó).
- **Fan-out precompute vs on-demand:** tính sẵn danh sách người nhận (nhanh khi gửi) ↔ tốn storage và có thể lỗi thời.

### 7. Ví dụ Production
Sự cố kinh điển: chiến dịch marketing gửi 5M push, dùng chung queue với OTP đăng nhập. Marketing dội vào queue → OTP xếp sau 5M tin → user không nhận được OTP trong 10 phút → không đăng nhập được → support quá tải. Fix: tách queue + priority tuyệt đối cho transactional. Sự cố idempotency: một bug retry ở consumer gửi email "đơn hàng đã giao" **8 lần** cho 12.000 khách vì thiếu dedup → khách phàn nàn, tỷ lệ unsubscribe tăng vọt. Bài học kép: (1) **transactional và marketing phải cô lập tài nguyên** — trộn chung là để marketing làm hỏng thứ quan trọng; (2) **idempotency không phải chi tiết kỹ thuật, nó là trải nghiệm khách hàng** — gửi trùng 8 lần là mất niềm tin thật.

### 8. Những câu trả lời chưa đủ tốt
- "Gửi notification qua một service gọi FCM/SMTP." → Đồng bộ? Provider chậm/chết thì sao? Retry? Fan-out 1M? Chưa chạm phần khó.
- "Dùng queue để gửi async." → Đúng hướng nhưng: idempotency? per-kênh isolation? priority? fan-out? Cần cụ thể hơn.

### 9. Sai lầm phổ biến của ứng viên
- Gửi đồng bộ trong request nghiệp vụ (order service tự gọi SMTP) — coupling + chậm + mất khi provider lỗi.
- Không có idempotency → gửi trùng khi retry.
- Chung queue cho mọi kênh/mọi độ ưu tiên → marketing nghẽn OTP.
- Không xử lý provider không đáng tin (không retry/circuit breaker).
- Bỏ qua user preference/rate limit → spam → user tắt notification (mất kênh vĩnh viễn).
- Fan-out 1M xử lý tuần tự trong một consumer.

### 10. Follow-up Questions
- Fan-out cho "user có 10M follower đăng bài" (celebrity problem) — precompute hay on-demand? (nối news feed fan-out.)
- Đảm bảo OTP tới trong 30s — đo và alert delivery latency thế nào?
- Provider callback trạng thái async (delivered/failed) — cập nhật và xử lý thế nào?
- Dedup xuyên nhiều instance consumer — Redis? Cửa sổ dedup bao lâu?
- Nếu scale 100x volume marketing — kiến trúc vỡ ở đâu? (query fan-out, provider rate limit, queue depth.)

### 11. Liên hệ với Production
Mọi app có notification (thương mại điện tử, mạng xã hội, ngân hàng) chạy kiến trúc queue-based này; các nền tảng chuyên dụng (OneSignal, Firebase, AWS SNS/Pinpoint) đóng gói sẵn phần lớn. Vấn đề nghiêm trọng khi: volume lớn + đa kênh + yêu cầu reliability khác nhau (OTP vs marketing), hoặc fan-out celebrity. Dấu hiệu cần rà soát: OTP/transactional bị chậm khi có chiến dịch marketing, tỷ lệ gửi trùng/spam khiến unsubscribe tăng, và queue depth tăng không hồi phục ở kênh có provider chậm.

---

## Case 4 — [Staff → Principal] Thiết kế Payment System (đảm bảo tính đúng đắn về tiền)

### 1. Câu hỏi
"Thiết kế hệ thống thanh toán: user trả tiền cho đơn hàng qua cổng thanh toán bên thứ ba. Yêu cầu tuyệt đối: **không mất tiền, không trừ trùng, không cho qua đơn chưa trả**. Xử lý mọi trường hợp lỗi mạng/timeout."

### 2. Interviewer muốn kiểm tra điều gì?
- Đây là bài khó nhất — nơi mọi khái niệm hội tụ: idempotency, saga, isolation, exactly-once qua hệ không đáng tin, reconciliation.
- Tư duy "tiền là thiêng liêng" — chấp nhận chậm/phức tạp để đổi lấy đúng đắn tuyệt đối.
- Xử lý trường hợp mơ hồ (timeout = unknown, không phải failure) — dấu hiệu đã làm hệ thống tiền thật.

### 3. Câu trả lời ngắn gọn (30 giây)
"Nguyên tắc số một: **tiền không được phép sai, chậm cũng được**. Ba trụ cột: (1) **Idempotency tuyệt đối** — mọi thao tác thanh toán có idempotency key; gọi lại với cùng key trả cùng kết quả, không trừ tiền lần hai (bắt buộc vì retry/timeout là chắc chắn xảy ra). (2) **Ledger append-only (bút toán kép)** — không update số dư trực tiếp mà ghi các bút toán bất biến, số dư = tổng bút toán; đây là cách kế toán làm hàng trăm năm, cho audit và không bao giờ 'mất dấu' tiền. (3) **State machine + reconciliation** — mỗi payment là một state machine bền vững (pending→authorized→captured→settled/failed); và vì cổng thanh toán bên thứ ba không đáng tin, **timeout không phải failure mà là unknown** — phải có job đối soát định kỳ query lại trạng thái thật từ cổng và hòa giải. Dùng saga cho luồng đa bước, outbox cho phát event, và webhook + polling để nhận kết quả bất đồng bộ."

### 4. Trình bày Senior Level (walkthrough)
**Làm rõ:** đồng bộ hay có redirect (3DS)? Một cổng hay nhiều? Cần refund/partial? Đây là ví (giữ số dư) hay chỉ pass-through tới cổng? → giả sử: đơn hàng → charge qua cổng (Stripe-like), cần đúng tuyệt đối.

**Idempotency (nền tảng):**
- Client tạo idempotency key (UUID) cho mỗi ý định thanh toán, gửi kèm mọi retry. Server: key mới → xử lý + lưu kết quả gắn key; key đã thấy → trả kết quả cũ, **không** xử lý lại. Điều này biến "gọi nhiều lần" (do timeout/retry) thành vô hại — điều kiện sống còn khi nói chuyện với hệ thống qua mạng.
- Lưu key + trạng thái trong DB với unique constraint (chống race hai request cùng key đồng thời — nối isolation chương Postgres).

**Ledger (bút toán kép):**
- Không `UPDATE balance = balance - X`. Thay vào đó ghi bút toán bất biến: mỗi giao dịch là các dòng debit/credit cân bằng (tổng luôn = 0). Số dư = tổng các dòng của tài khoản. Ưu: audit hoàn hảo (mọi thay đổi có dấu vết), không bao giờ mất dấu tiền (bất biến kế toán), dễ đối soát. Đây là event sourcing áp cho tiền (nối chương EDA) — và với tiền thì nó **thực sự đáng** dù đắt.

**State machine + luồng bất đồng bộ:**
- Payment state: `pending → authorized → captured → settled` (hoặc `failed/cancelled/refunded`). Lưu bền vững, chuyển trạng thái nguyên tử.
- Cổng thanh toán trả kết quả qua: response đồng bộ (có thể timeout), **webhook** (async, đáng tin hơn nhưng có thể trễ/trùng/mất), và **polling API** (query trạng thái chủ động). Dùng cả ba: webhook là chính, polling là lưới an toàn.
- **Xử lý timeout (điểm Staff/Principal cốt tử):** khi charge bị timeout, **KHÔNG** được coi là thất bại và cho retry mù — tiền có thể đã bị trừ ở cổng (response lạc trên đường về). Phải: giữ trạng thái `unknown/pending`, rồi **query lại** cổng bằng idempotency key để biết kết quả thật trước khi quyết định. Đây là lý do idempotency key phải được gửi tới cả cổng thanh toán.

**Reconciliation (bắt buộc cho hệ tiền):**
- Job định kỳ đối soát: lấy danh sách giao dịch của mình vs báo cáo từ cổng → tìm lệch (mình nghĩ failed nhưng cổng đã charge, hoặc ngược lại) → hòa giải hoặc cảnh báo con người. Không hệ thống tiền nào chạy không có đối soát — nó là "compensation bằng con người" cho những gì code không lường (nối chương saga).

**Saga cho đa bước:** trừ kho + charge tiền + tạo đơn = saga với compensation (charge fail → nhả kho; charge thành công nhưng tạo đơn fail → refund). Sắp bước charge làm pivot, xử lý bước sau charge phải retriable (nối chương Saga).

### 5. Giải thích bản chất
Payment system là nơi ba sự thật khắc nghiệt của hệ phân tán hội tụ và không thể lảng tránh: (1) **mạng không đáng tin** → timeout là unknown, không phải failure — và với tiền, đối xử unknown như failure là công thức mất/trừ trùng tiền; (2) **exactly-once không tồn tại qua ranh giới hệ thống** → không thể ép cổng thanh toán "chỉ charge một lần" bằng ý chí; chỉ có thể làm cho việc gọi lặp trở nên an toàn (idempotency) và đối soát để bắt sai lệch; (3) **trạng thái phải phục hồi được và có dấu vết** → ledger append-only + state machine bền vững đảm bảo không mất thông tin dù crash bất kỳ lúc nào. Insight sâu nhất: ngành tài chính đã giải bài toán này **trước khi có máy tính** — bút toán kép, đối soát cuối ngày, số hiệu giao dịch (idempotency key bằng giấy) — vì bản chất bài toán là về **tính bất biến và khả năng kiểm toán của sự thật**, không phải về công nghệ. Kỹ sư giỏi nhất ở đây là người khiêm tốn học từ kế toán thay vì phát minh lại. Và triết lý bao trùm: với tiền, **đúng đắn > mọi thứ khác** — chậm hơn, phức tạp hơn, đắt hơn đều chấp nhận được; sai một xu thì không.

### 6. Trade-off
- **Idempotency + ledger + reconciliation:** đúng đắn tuyệt đối, audit hoàn hảo ↔ phức tạp cao, chậm hơn (nhiều bước, nhiều lưu trữ), tốn công xây.
- **Đồng bộ (chờ kết quả charge) vs async (webhook):** UX trả lời ngay ↔ async đáng tin hơn nhưng user phải chờ/poll trạng thái.
- **Strong consistency cho tiền:** không sai số ↔ latency cao (nối isolation Serializable, lock); nhưng với tiền đây là đánh đổi bắt buộc.
- **Ledger append-only:** không mất dấu, audit ↔ tốn storage, query số dư cần tổng hợp (snapshot định kỳ như event sourcing).
- **Reconciliation:** bắt mọi sai lệch ↔ cần đội/quy trình vận hành, không phải chỉ code.

### 7. Ví dụ Production
Sự cố (lặp lại từ chương Saga vì nó là bài học trung tâm): luồng charge bị timeout, code coi timeout = fail → nhả ghế/hủy đơn → nhưng cổng đã charge thành công (response lạc) → khách mất tiền không có hàng. Đây là lỗi "timeout = failure" — sai lầm chết người nhất của hệ tiền. Fix: timeout → trạng thái unknown → query cổng bằng idempotency key → biết sự thật rồi mới quyết định. Sự cố thứ hai: hệ update `balance` trực tiếp thay vì ledger; một bug race (nối isolation) làm hai giao dịch đồng thời ghi đè → số dư sai, và vì không có ledger, **không thể truy ra tiền đi đâu** — mất hàng tuần đối soát thủ công từ log. Sau đó chuyển sang ledger append-only: mọi đồng tiền có dấu vết, bug tương tự sau này phát hiện và sửa trong giờ. Bài học: **thiết kế cho khả năng đối soát quan trọng ngang thiết kế cho tính đúng** — vì bạn sẽ luôn cần biết "tiền đi đâu" khi có nghi ngờ.

### 8. Những câu trả lời chưa đủ tốt
- "Gọi API cổng thanh toán, thành công thì tạo đơn." → Timeout thì sao? Trừ trùng khi retry? Đối soát? Bỏ qua toàn bộ phần khó — với bài tiền là rớt.
- "Dùng transaction để đảm bảo đúng." → Transaction DB không bọc được cổng bên thứ ba (không có 2PC xuyên hệ ngoài); cần idempotency + saga + reconciliation. Hiểu sai phạm vi của transaction.

### 9. Sai lầm phổ biến của ứng viên
- Coi timeout là failure → retry mù → trừ trùng tiền (sai lầm số một).
- Không có idempotency key, hoặc không gửi nó tới cả cổng thanh toán.
- Update số dư trực tiếp thay vì ledger → mất khả năng đối soát.
- Không có reconciliation — tin rằng code đúng là đủ (không bao giờ đủ với tiền).
- Nghĩ có exactly-once qua cổng bên thứ ba.
- Dùng eventual consistency cho số dư mà không nhận ra rủi ro double-spend (nối isolation).

### 10. Follow-up Questions
- Hai request cùng idempotency key tới đồng thời (race) — xử lý thế nào? (unique constraint + lock — nối isolation.)
- Webhook từ cổng có thể trùng/mất/sai thứ tự — làm sao xử lý đáng tin? (idempotent webhook handler + polling fallback + verify signature.)
- Refund/partial refund tác động ledger thế nào? Giữ bất biến ra sao?
- Reconciliation phát hiện lệch: quy trình tự động hóa tới đâu, khi nào cần con người?
- Scale: 10.000 giao dịch/s — ledger append-only có thành bottleneck không? Shard theo gì? (theo account, giữ giao dịch của một account trong một shard — nối sharding.)
- Multi-currency, tỷ giá — mô hình hóa trong ledger thế nào?

### 11. Liên hệ với Production
Mọi ví điện tử, cổng thanh toán, ngân hàng số (Stripe, Grab, MoMo, các ngân hàng) chạy trên nền idempotency + ledger + reconciliation; Stripe công khai nhiều về thiết kế idempotency của họ. Vấn đề luôn nghiêm trọng vì đây là tiền — không có "sự cố nhỏ". Dấu hiệu bắt buộc phải có: idempotency trên mọi thao tác tiền, ledger append-only (không update số dư trực tiếp), đối soát định kỳ với mọi bên ngoài, và bộ phận/quy trình xử lý lệch. Câu hỏi kiểm toán tối hậu: "nếu hệ thống crash giữa charge và ghi đơn, làm sao khách không mất tiền và không nhận hàng miễn phí?" — nếu chưa có câu trả lời rõ ràng cho mọi điểm crash, hệ thống chưa sẵn sàng cho tiền thật.
