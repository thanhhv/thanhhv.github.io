+++
title = "13.1. Caching Failures"
date = "2026-07-13T16:20:00+07:00"
draft = false
tags = ["backend", "system-design"]
series = ["System Design — Tư Duy Thiết Kế Hệ Thống"]
+++

> Ba tình huống anh em: Cache Stampede, Cache Avalanche, Thundering Herd. Chung một mô hình gốc — **đồng bộ hóa ngẫu nhiên + khuếch đại** — khác nhau ở phạm vi và ngòi nổ.

---

## Case 1 — Cache Stampede (dogpile trên MỘT key nóng)

### Triệu chứng

Hệ thống êm ả, rồi đúng một khoảnh khắc: DB CPU dựng đứng trong vài giây, latency một nhóm endpoint tăng vọt, sau đó *có thể* tự hồi phục — lặp lại theo chu kỳ đúng bằng TTL của một key nào đó. Trong slow query log: hàng trăm bản sao *của cùng một query* trong cùng một giây.

### Root cause & tại sao xảy ra

Key nóng (trang chủ, sản phẩm hot) hết hạn TTL → trong khoảng khắc giữa "expire" và "ai đó set lại", **mọi** request tới key đó đều miss → tất cả cùng chạy query gốc → DB nhận N bản sao của một query đắt. Nếu query mất 2 giây và có 500 request/giây → 1000 query giống nhau chồng lên nhau, làm query càng chậm, càng nhiều request dồn vào — khuếch đại tự nuôi. Cache càng hiệu quả (hit rate càng cao), cú đấm khi miss càng nặng — vì DB đã lâu không thấy tải thật.

### Kiến trúc bị ảnh hưởng

Mọi kiến trúc cache-aside có key nóng + query gốc đắt ([giai đoạn 2, Phần 12](/series/system-design/12-evolution/02-them-redis/)).

### Metric / Dashboard / Alert

- Metric: cache hit rate theo key-pattern; DB QPS phân theo query fingerprint; số query trùng lặp đang chạy đồng thời (`pg_stat_activity` đếm theo query).
- Dashboard: overlay hit-rate và DB CPU trên cùng trục thời gian — stampede hiện thành răng cưa đối xứng nhau.
- Alert: DB QPS của một fingerprint tăng > 10× baseline trong < 1 phút.

### Điều tra

1. Xác định query bị nhân bản (slow log / `pg_stat_statements` sắp theo calls trong 5 phút gần nhất).
2. Truy ngược key cache tương ứng → xem TTL và tần suất truy cập.
3. Xác nhận chu kỳ sự cố ≈ TTL → chốt chẩn đoán.

### Khắc phục & phòng tránh

- **Cầm máu:** set lại key bằng tay (hoặc tăng TTL tạm) ngay khi phát hiện; nếu đang nghẽn, bật giới hạn concurrency ở connection pool cho query đó.
- **Chữa gốc — chọn một hoặc kết hợp:**
  - **Single-flight / mutex:** chỉ request đầu tiên đi tính lại, số còn lại chờ kết quả hoặc nhận giá trị cũ (Go `singleflight`, Redis `SET NX` lock ngắn).
  - **Stale-while-revalidate:** hết hạn "mềm" — tiếp tục trả giá trị cũ trong khi một tiến trình nền làm mới.
  - **Xác suất làm mới sớm (probabilistic early expiration):** mỗi request có xác suất nhỏ tự nguyện refresh trước hạn, xác suất tăng dần khi gần hết TTL — dàn mỏng thời điểm tính lại.
  - Key cực nóng, không được phép miss: **refresh chủ động bằng scheduler**, không dùng TTL làm cơ chế làm mới.

---

## Case 2 — Cache Avalanche (sụp đổ HÀNG LOẠT key, hoặc mất cả cache)

### Triệu chứng

DB quá tải trên **diện rộng** (không phải một query), error rate toàn hệ thống tăng, thường ngay sau một trong các sự kiện: deploy có flush cache, Redis restart/failover, hoặc một mốc thời gian tròn (00:00). Khác stampede: không tự hồi phục theo chu kỳ — hệ thống có thể xoáy xuống cho đến khi can thiệp.

### Root cause & tại sao xảy ra

Hai biến thể cùng bản chất "lá chắn biến mất đồng loạt":

1. **TTL đồng loạt:** hàng trăm nghìn key được set cùng đợt (warm-up, batch import) với cùng TTL → hết hạn cùng giây.
2. **Mất cache node:** Redis chết/restart → 100% miss tức thời. DB vốn chỉ quen chịu 5–10% tải thật (phần sau lá chắn cache) giờ nhận 100% → quá tải → chậm → app timeout → **retry nhân thêm tải** → DB gục hẳn. Điểm hiểm: cache hồi phục xong, DB vẫn đang gục, cache lại không có ai đổ dữ liệu vào → hệ thống kẹt ở trạng thái chết dù mọi thành phần đã "sống".

### Kiến trúc bị ảnh hưởng

Mọi hệ có cache hit-rate cao mà tầng sau **không chịu nổi tải khi mất cache** — tức là gần như mọi hệ trưởng thành. Hit rate 95% nghĩa là DB đang được bảo vệ 20× — và cũng nghĩa là mất cache = tải DB tăng 20× tức thời.

### Metric / Dashboard / Alert

- Metric: hit rate tổng (rơi thẳng đứng = avalanche), Redis uptime/failover event, DB connections + QPS + p99, tỷ lệ timeout app→DB.
- Alert: hit rate rơi > 20 điểm % trong 1 phút; Redis master thay đổi (failover event) — alert *thông tin* để người trực cảnh giác ngay cả khi chưa có triệu chứng.

### Điều tra

1. Hit rate có rơi thẳng đứng không? Rơi lúc nào, trùng sự kiện gì (deploy? failover? 00:00)?
2. Nếu hit rate bình thường mà DB vẫn quá tải diện rộng → không phải avalanche, quay về [chương 1.5](/series/system-design/01-foundations/05-bottleneck-analysis/).
3. Xem TTL distribution của key vừa hết hạn (nếu log được) hoặc lịch sử warm-up gần nhất.

### Khắc phục & phòng tránh

- **Cầm máu:** shed load — bật rate limit/queue ở edge để DB thở; warm lại cache **theo thứ tự key quan trọng** bằng script; tạm trả degraded response (trang chủ tĩnh) trong lúc warm.
- **Phòng:**
  - **TTL + jitter ngẫu nhiên** (±10–30%) — mọi nơi, mọi lúc, như một quy ước code review.
  - Redis HA (replica + auto-failover) để "mất cache" thành sự kiện hiếm.
  - **Load test kịch bản mất cache:** đây là điểm khác biệt giữa team đã trưởng thành và chưa — biết trước DB sống được bao lâu khi trần trụi, và có sẵn nút degraded mode.
  - Circuit breaker giữa app và DB: khi DB quá tải, fail nhanh + trả stale/default thay vì xếp hàng làm DB chết hẳn.

---

## Case 3 — Thundering Herd (bầy đàn dồn vào MỘT điểm sau MỘT tín hiệu)

### Triệu chứng

Sau một sự kiện đánh thức đồng loạt — service restart, connection bị cắt hàng loạt, lock được nhả, thông báo push gửi cho 1 triệu user — một tài nguyên duy nhất nhận cú đấm đồng thời từ hàng nghìn client: CPU spike, connection storm, accept queue tràn. Đặc trưng: **có một "tiếng còi" xác định được** ngay trước sự cố.

### Root cause & tại sao xảy ra

Nhiều tác nhân chờ cùng một điều kiện; điều kiện xảy ra **một lần, cho tất cả** → tất cả hành động cùng khoảnh khắc. Gốc rễ nằm ở thiết kế "đánh thức tất cả rồi để chúng tranh nhau" thay vì "đánh thức đúng số lượng cần thiết, giãn theo thời gian". (Tên gọi đến từ bài toán `accept()` của OS — nghìn process cùng dậy tranh một connection — nhưng dạng tổng quát xuất hiện ở mọi tầng: reconnect sau khi LB restart, cron 0 0 * * * của nghìn tenant, push notification kéo triệu app cùng mở, client polling đồng nhịp.)

### Kiến trúc bị ảnh hưởng

Hệ có nhiều client đồng nhất (mobile app, IoT fleet, worker pool, cron farm) + một điểm hội tụ (API, DB, auth service). Càng nhiều client và càng "kỷ luật" (cùng config, cùng nhịp), herd càng gọn và càng đau.

### Metric / Dashboard / Alert

- Metric: connection rate (new conn/s — quan trọng hơn tổng conn), request rate theo giây (độ phân giải 1s mới thấy spike; đồ thị trung bình theo phút *che mất* herd), listen/accept queue overflow (`netstat -s`), auth request/s (herd thường đập vào auth trước tiên).
- Alert: new connections/s > N× baseline trong cửa sổ 10 giây.

### Điều tra

1. Tìm "tiếng còi": sự kiện gì xảy ra đúng trước spike? (deploy, restart, push campaign, mốc giờ tròn, mạng chớp).
2. Nhìn phân bố nguồn: hàng nghìn client khác nhau cùng hành vi = herd; một client điên = bài khác (bug/abuse).
3. Kiểm tra nhịp lặp: herd retry sẽ dội lại theo sóng — khoảng cách sóng tiết lộ cấu hình retry của client.

### Khắc phục & phòng tránh

- **Cầm máu:** rate limit/queue tại điểm hội tụ (chịu hy sinh một phần client để cứu tổng thể); nếu là reconnect storm — tăng accept backlog + scale tạm điểm nhận; nếu là push campaign — dừng campaign.
- **Phòng — nguyên tắc chung: đừng bao giờ để N client cùng nhịp:**
  - **Jitter cho mọi thứ lặp lại:** retry (exponential backoff **+ full jitter** — không có jitter, backoff chỉ biến một sóng thành nhiều sóng đều tăm tắp), polling interval, cron (hash tenant-id ra phút), TTL reconnect.
  - Push campaign lớn: **rải theo thời gian** (1 triệu notification trong 30 phút, không phải 30 giây) — phối hợp với team marketing, vì tiếng còi hay được bóp từ ngoài engineering.
  - Client staggered reconnect: sau mất kết nối, chờ `random(0, backoff)` trước khi reconnect.
  - Điểm hội tụ tự vệ: rate limit theo nguồn + accept queue đủ + graceful shedding (trả 429 + Retry-After có jitter) — dạy client cư xử ngay trong response.

---

## Tổng kết nhóm caching

| | Stampede | Avalanche | Thundering Herd |
|---|---|---|---|
| Phạm vi | 1 key nóng | Hàng loạt key / cả cache | 1 điểm hội tụ, N client |
| Ngòi nổ | TTL của key đó | TTL đồng loạt / cache chết | Một tín hiệu đánh thức đồng loạt |
| Chữ ký nhận diện | Chu kỳ = TTL, 1 query nhân bản | Hit rate rơi thẳng đứng | Có "tiếng còi" ngay trước spike |
| Thuốc đặc trị | Single-flight, stale-while-revalidate | Jitter TTL, HA cache, degraded mode | Jitter mọi nhịp, rải sự kiện |

Cả ba chia sẻ một bài học thiết kế: **hệ thống có cache tốt là hệ thống đã quên mất tải thật của mình.** Định kỳ hỏi: "nếu lá chắn này biến mất 60 giây, chuyện gì xảy ra?" — và trả lời bằng load test, không phải bằng niềm tin.
