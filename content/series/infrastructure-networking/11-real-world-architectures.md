+++
title = "Bài 11 — Kiến Trúc Thực Tế"
date = "2026-02-01T17:00:00+07:00"
draft = false
tags = ["backend", "infrastructure", "networking"]
series = ["Infrastructure & Networking"]
+++

# Module 11 — Kiến trúc thực tế: các hệ thống lớn được xây thế nào và vì sao

> Mỗi mục dưới đây là một bài tập tổng hợp của 10 module trước. Cấu trúc chung: đặc trưng workload → nó ép ra kiến trúc gì → trade-off và failure case đã trả giá. Điều đáng học không phải sơ đồ — mà là **chuỗi suy luận từ workload ra kiến trúc**.

---

## 1. E-commerce Platform

**Đặc trưng workload**: đọc/ghi lệch cực đoan (duyệt:mua ~100:1); burst có thể 10–50× trong sale event (thứ giết hệ thống không phải tải trung bình mà là 10 phút đầu của flash sale); một chuỗi nghiệp vụ trộn lẫn phần chịu được sai lệch (catalog, review) và phần tuyệt đối không (thanh toán, tồn kho).

**Kiến trúc bị ép ra**:
- Tách đường đọc và đường ghi triệt để. Đường đọc (catalog/search): CDN + cache nhiều tầng + search engine (Elasticsearch) + DB replica — chấp nhận stale vài phút, đổi lấy hấp thụ 99% traffic bằng hạ tầng rẻ. Đường ghi (order/payment): DB transactional, không cache, bảo vệ bằng queue.
- **Checkout qua queue**: nhận đơn = ghi message (nhanh, hấp thụ burst) → xử lý payment/tồn kho theo tốc độ ổn định phía sau; user nhận "đang xử lý" thay vì timeout. Đây là backpressure (module 09) áp vào nghiệp vụ.
- Tồn kho là bài consistency kinh điển: khóa chặt (không oversell — mất doanh thu vì báo hết hàng oan trong burst) vs lạc quan (oversell rồi xin lỗi — chi phí xử lý ngoại lệ). Đa số chọn: reserve lạc quan + xác nhận khi thanh toán, oversell hiếm xử lý bằng quy trình. Trade-off nghiệp vụ quyết định, không phải kỹ thuật.
- Flash sale đúng nghĩa: thêm hàng chờ ảo (queue page) + load shedding có ưu tiên — thà phục vụ tốt 50k người trong hàng còn hơn 500k người cùng thấy 500.

**Failure case tiêu biểu**: sale event → autoscaling không kịp (bài học trễ 3–5 phút, module 05) → phải pre-scale theo lịch; cache stampede khi hot item hết TTL đúng giờ vàng (module 02); và "recommendation service chết kéo sập checkout" — thiếu bulkhead, lỗi kinh điển nhất của ngành này.

---

## 2. FinTech / Payment Platform

**Đặc trưng workload**: đúng > nhanh > rẻ, theo thứ tự tuyệt đối. Mỗi thao tác tiền phải: không mất, không nhân đôi, audit được, và tuân thủ pháp lý (PCI-DSS, data residency).

**Kiến trúc bị ép ra**:
- **Ledger append-only + double-entry** làm nguồn sự thật: không UPDATE số dư — ghi bút toán, số dư là tổng dẫn xuất. Sửa sai bằng bút toán đảo, không sửa lịch sử. Điều này cho audit trail miễn phí và loại bỏ cả lớp bug race condition trên số dư.
- **Idempotency key ở mọi API ghi** (module 02 — retry chỉ an toàn khi idempotent, và mạng LUÔN mơ hồ: timeout không cho biết lệnh đã chạy hay chưa). Client tạo key, server dedupe. Không có nó, retry + double-click = chuyển tiền 2 lần.
- Giao dịch xuyên hệ thống (với ngân hàng, đối tác): không có distributed transaction với bên thứ ba → **saga + reconciliation**: mỗi bước có bù trừ (compensation), và một hệ đối soát chạy định kỳ so khớp hai sổ — chấp nhận sự thật rằng hệ phân tán sẽ lệch, thiết kế quy trình phát hiện + sửa lệch thay vì giả vờ không lệch.
- DB: consistency chọn trước availability (CP) cho đường tiền — thà từ chối giao dịch còn hơn nhận mơ hồ. RPO ~0 bằng synchronous replication trong region (trả giá latency ghi), cross-region async + đối soát.
- Cô lập PCI scope: dữ liệu thẻ vào một vùng nhỏ tách biệt (tokenization — hệ chính chỉ giữ token), thu nhỏ phạm vi audit từ "cả công ty" xuống "một service".

**Failure case tiêu biểu**: double-spend qua retry không idempotent; lệch sổ âm thầm do bug saga không có reconciliation (phát hiện sau 3 tháng = ác mộng); và outage do chính cơ chế an toàn — synchronous replication treo khi replica chậm (trade-off consistency-availability hiện nguyên hình).

---

## 3. Social Network / Feed

**Đặc trưng workload**: đồ thị quan hệ + fan-out khổng lồ (1 post của người có 10M follower phải xuất hiện ở 10M feed); read nhiều áp đảo; chịu được eventual consistency gần như mọi nơi (thấy post trễ 30 giây không ai kiện).

**Kiến trúc bị ép ra**:
- Bài toán trung tâm: **fan-out on write** (đẩy post vào feed cache của từng follower lúc đăng — đọc rẻ, ghi đắt với celebrity) vs **fan-out on read** (gom lúc đọc — ghi rẻ, đọc đắt). Lời giải thực tế là **hybrid**: user thường = fan-out on write; celebrity = fan-out on read, merge lúc phục vụ. Bài học lớn: **phân phối power-law phá mọi thiết kế đồng nhất** — thiết kế cho trung bình chết vì đuôi.
- Cache là hạ tầng chính chứ không phải phụ trợ (cache-hit 95%+ là điều kiện sống): mọi bài học cache stampede, hot key (một celebrity = một hot key làm nóng một shard cache) áp dụng cực đại. Giải hot key: replicate key nóng ra nhiều node + local cache tầng app.
- Eventual consistency mặc định, trừ vài chỗ bắt buộc mạnh hơn (block user phải có hiệu lực NGAY — an toàn cá nhân; đây là ví dụ đẹp về consistency theo từng nghiệp vụ thay vì toàn hệ).

**Failure case**: thundering herd khi cache cluster restart (mất cache = DB nhận 20× tải = chết ngay — vì thế cache warm-up và restart từng phần là quy trình nghiêm ngặt); tin nổi bật toàn cầu tạo hot partition mà không shard scheme nào đỡ sẵn.

---

## 4. SaaS Platform (B2B, multi-tenant)

**Đặc trưng workload**: nhiều khách hàng chia sẻ hạ tầng; một tenant lớn có thể bằng 1000 tenant nhỏ; yêu cầu cách ly (dữ liệu + hiệu năng) và đôi khi residency/compliance theo từng khách.

**Kiến trúc bị ép ra** — trục quyết định là **mức độ cách ly tenant**:
| Mô hình | Cách ly | Chi phí/tenant | Vận hành |
|---|---|---|---|
| Pool (chung DB, cột tenant_id) | Logic (RLS) | Thấp nhất | 1 fleet, dễ; rủi ro bug rò dữ liệu chéo cao nhất |
| Bridge (chung app, DB/schema riêng) | Dữ liệu vật lý | Trung bình | Migration × N database |
| Silo (stack riêng từng tenant) | Hoàn toàn | Cao nhất | Phù hợp enterprise trả đậm/compliance |

Thực tế trưởng thành: **hỗn hợp theo tier** — long tail ở pool, enterprise ở bridge/silo. Kèm theo bắt buộc: (1) **tenant_id trong mọi truy vấn được enforce ở tầng nền** (RLS/middleware — không dựa vào kỷ luật dev; một `WHERE` thiếu = rò dữ liệu khách A cho khách B = sự cố tồn vong của công ty SaaS); (2) **noisy neighbor control**: rate limit + quota theo tenant, không để 1 tenant chạy report nặng làm chậm 999 tenant (bulkhead theo tenant); (3) mọi metric/log gắn tenant_id — "khách X kêu chậm" phải trả lời được trong phút.

**Failure case**: bug rò dữ liệu chéo tenant (kinh khủng nhất); migration schema chạy trên 5000 DB bridge kẹt giữa chừng ở DB thứ 3021 (tooling migration theo fleet là hạ tầng nghiêm túc); một tenant bật tính năng export và DDoS chính hệ thống từ bên trong.

---

## 5. Video Streaming

**Đặc trưng workload**: băng thông là chi phí và giới hạn số 1 (video = ~85% traffic Internet); latency giao hàng quan trọng hơn latency tính toán; nội dung bất biến sau khi encode → cache được gần tuyệt đối.

**Kiến trúc bị ép ra**:
- Pipeline offline: ingest → transcode thành **bậc thang bitrate** (240p→4K, mỗi bậc cắt segment 2–6s) → đẩy ra CDN. Player làm **ABR** (adaptive bitrate): đo throughput thực tế, tự chọn bậc cho segment kế — chất lượng thích nghi từng vài giây với mạng của từng user. Toàn bộ "streaming" thực chất là HTTP GET các file tĩnh nhỏ — thiết kế thiên tài ở chỗ biến bài toán streaming thành bài toán CDN đã giải.
- CDN không chỉ là mua dịch vụ: ở quy mô Netflix là **đặt appliance cache trong ISP** (Open Connect) — đưa nội dung đến sát user đến mức traffic không ra khỏi ISP; kèm **pre-position** nội dung dự đoán hot vào giờ thấp điểm. Bài học: ở quy mô đủ lớn, chi phí bandwidth biện minh cho việc xây hạ tầng vật lý riêng.
- Transcode là bài batch compute khổng lồ → spot instance + queue (chịu chết hoàn hảo — module 07); phần live streaming khó hơn VOD nhiều (không pre-position được, latency giây, spike đồng thời).

**Failure case**: sự kiện live lớn (chung kết bóng đá) = hàng chục triệu client cùng xin segment MỚI NHẤT mỗi 2s — cache hit cao nhưng origin vẫn nhận sóng đều đặn mỗi segment mới, và một nhịp origin chậm = hàng triệu player cùng downshift/rebuffer; đó là lý do live có kiến trúc origin shield + request coalescing riêng biệt và được diễn tập như chiến dịch.

---

## 6. Messaging Platform (chat)

**Đặc trưng workload**: hàng triệu connection dài đồng thời (WebSocket) nhưng mỗi connection thưa (idle 99%); message nhỏ, latency cảm nhận được (<1s); yêu cầu ordering theo hội thoại và delivery guarantee.

**Kiến trúc bị ép ra**:
- **Gateway tier giữ connection** (bài C10K/epoll module 03 ở dạng thuần khiết nhất — mỗi node giữ 500k–1M connection idle) tách khỏi **logic tier stateless**: connection đắt vì số lượng, logic đắt vì CPU — tách ra để scale độc lập. Cần bảng định tuyến user→gateway node (Redis/registry) để đẩy message tới đúng connection.
- Ordering: **tổng thứ tự toàn cục là bất khả thi và không cần** — chỉ cần thứ tự TRONG một hội thoại → gán sequence per-conversation (shard theo conversation_id, mỗi hội thoại một dòng ghi tuần tự). Chọn scope consistency nhỏ nhất mà nghiệp vụ cần là bài học tổng quát.
- Delivery: at-least-once + dedupe bằng message ID phía client (exactly-once end-to-end là ảo tưởng — module 02 idempotency một lần nữa); trạng thái sent/delivered/read là ACK ba tầng.
- Deploy gateway tier là nghi lễ (bài học WebSocket module 02): drain chậm, reconnect với jitter, và dimension hóa "connection churn rate" như một metric hạng nhất.

**Failure case**: mất một gateway node = trăm nghìn reconnect đồng loạt (thundering herd vào auth + registry — phải có jitter và rate limit reconnect); vòng lặp notification→mở app→connection→presence update→notification khuếch đại chính nó.

---

## 7. Blockchain Infrastructure

**Đặc trưng workload**: node phân tán không tin nhau; state machine replication qua consensus; yêu cầu về key management và tính bất biến cao nhất trong mọi ngành.

**Điều thú vị về hạ tầng** (bỏ qua tranh luận về ứng dụng): đây là hệ **thù địch by design** — mọi bài học security module 10 ở mức cực đại. Hạ tầng thực tế của một công ty vận hành trong ngành này: (1) **node fleet** (full/archive node) là StatefulSet ác mộng — state hàng TB, sync ban đầu hàng ngày/tuần, nâng cấp client phải theo lịch hard-fork toàn mạng (deadline không thể xin hoãn — hiếm có trong ngành hạ tầng); (2) **key management là sống còn tuyệt đối**: private key = tiền, mất là mất không kháng cáo → HSM, multi-sig/MPC, ceremony có nghi thức cho cold key — mô hình secret management (module 10) với hậu quả sai lầm không thể đảo ngược; (3) RPC tier cho ứng dụng đọc chain: cache + LB trước node fleet, vì node không được thiết kế để chịu query load — mọi bài học reverse proxy áp dụng.

**Failure case**: chain reorg làm dữ liệu "đã confirm" biến mất (phải thiết kế số confirmation chờ theo giá trị giao dịch); node client có bug consensus = fork ngoài ý muốn — chạy đa client (client diversity) chính là redundancy chống correlated failure (module 09) ở tầng phần mềm.

---

## 8. AI Platform

**Đặc trưng workload**: hai thế giới khác hẳn nhau chung một chữ AI — **training** (batch, chiếm GPU hàng tuần, checkpoint lớn, chịu restart) và **inference/serving** (online, latency-sensitive, autoscale theo traffic). Tài nguyên trung tâm là **GPU: đắt, khan hiếm, không chia sẻ mượt như CPU**.

**Kiến trúc bị ép ra**:
- GPU phá các giả định của scheduler thường: không "nén" được như CPU (module 03/05 — không có throttle, chỉ có được/không), bin packing theo card/MIG partition, và **utilization là KPI số 1** (GPU $30k+/card nằm im là đốt tiền) → queue + gang scheduling cho training (job 64 GPU cần đủ 64 cùng lúc — partial allocation là deadlock), ưu tiên + preemption giữa team.
- Training: checkpoint thường xuyên là bắt buộc (spot GPU + job dài ngày = chắc chắn có restart; bài học "thiết kế cho hỏng hóc" module 09 áp vào batch); data pipeline (đọc dataset TB) thường là bottleneck thật chứ không phải GPU — đo trước khi mua thêm card.
- Inference: model lớn = image/artifact hàng chục GB → cold start hàng phút phá autoscaling truyền thống (module 05 chuỗi trễ nhân 10) → giải bằng warm pool, model cache trên node, hoặc chấp nhận over-provision; batching động (gom request vài ms để GPU xử lý theo lô — đổi p50 latency lấy throughput 5–10×) là trade-off đặc trưng của serving.
- LLM serving thêm đặc thù: KV-cache làm mỗi request chiếm GPU memory theo độ dài context → capacity planning theo **token**, không theo request; streaming response (SSE) quay lại bài connection dài module 02.

**Failure case**: một team chiếm cả cụm GPU bằng job ưu tiên thấp không preempt được (thiếu quota + preemption từ đầu); cập nhật model mới có latency tệ hơn dưới tải thật mà offline benchmark không lộ (cần canary cho model như cho code — module 06, shadow traffic là kỹ thuật chuẩn); GPU node "chết một nửa" (ECC error, thermal throttle) làm job chậm bí ẩn — health check GPU chuyên biệt là hạng mục riêng.

---

## Tổng kết: các bất biến xuyên suốt 8 kiến trúc

1. **Workload quyết định kiến trúc** — không có kiến trúc tốt phổ quát, chỉ có kiến trúc khớp với hình dạng tải + yêu cầu consistency + cấu trúc chi phí của bài toán.
2. **Đường nóng được tách khỏi đường lạnh** ở mọi hệ: đọc/ghi, connection/logic, online/batch — tách để scale, bảo vệ và định giá độc lập.
3. **Queue xuất hiện ở mọi ranh giới chịu tải** — hấp thụ burst, cách ly nhịp độ, đổi latency lấy sống sót.
4. **Consistency được chọn theo từng mẩu nghiệp vụ**, không theo toàn hệ thống — và phần "chịu được sai lệch tạm" luôn lớn hơn trực giác ban đầu.
5. **Phân phối power-law (hot key, celebrity, tenant lớn, event lớn) phá thiết kế đồng nhất** — mọi hệ trưởng thành đều có đường xử lý riêng cho đuôi.
6. Cái giá của mọi cơ chế an toàn là có thật (latency, chi phí, độ phức tạp) — kiến trúc sư giỏi không phải người thêm nhiều lớp bảo vệ nhất, mà là người **biết chỗ nào không cần**.
