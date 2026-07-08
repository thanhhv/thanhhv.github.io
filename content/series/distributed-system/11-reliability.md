+++
title = "Chương 11 – Reliability Patterns: Sống sót giữa những dependency đang hỏng"
date = "2026-07-09T03:00:00+07:00"
draft = false
tags = ["backend", "distributed-systems"]
series = ["Distributed Systems"]
+++

## 1. Problem Statement

Chương 01 đã chứng minh: ở quy mô đủ lớn, **luôn có thứ gì đó đang hỏng**. Chương này trả lời câu hỏi tiếp theo: làm sao để một thành phần hỏng không kéo sập cả hệ — vì trong hệ phân tán, failure mode nguy hiểm nhất không phải là hỏng, mà là **hỏng lan truyền (cascading failure)**:

```
DB chậm (từ 5ms → 500ms)
 → connection pool của Service B cạn (request giữ connection lâu 100x)
  → thread của Service A chờ B, cạn thread pool
   → A timeout → client RETRY → tải TĂNG GẤP ĐÔI đúng lúc yếu nhất
    → toàn hệ sập, dù thứ hỏng ban đầu chỉ là MỘT query thiếu index
```

Chuỗi trên là kịch bản sự cố phổ biến nhất trong microservices — mọi pattern của chương này (timeout, retry có kỷ luật, circuit breaker, bulkhead, backpressure, rate limiting) tồn tại để **cắt đứt từng mắt xích** của nó. Điểm chung của cả chương: *hệ thống phải được thiết kế để từ chối bớt việc một cách có chủ đích, thay vì nhận hết rồi chết cùng nhau.*

## 2. Timeout — pattern nền, bị coi thường nhất

Không timeout = request treo giữ tài nguyên (thread, connection, memory) vô hạn → tài nguyên cạn là chết chắc, chỉ là bao giờ. Mặc định của nhiều thư viện là **vô hạn hoặc quá dài** (đi kiểm tra mặc định của HTTP client bạn đang dùng — ngay bây giờ).

Nguyên tắc thiết kế:
- **Timeout theo ngân sách end-to-end** (chương 02): SLA 1s ở edge → chia dần xuống; hop sau *ngắn hơn* hop trước, để tầng trên còn thời gian xử lý lỗi/degrade.
- **Đặt theo p99 thực tế + dư địa** (ví dụ p99=80ms → timeout 200–300ms), không theo "số đẹp" 30s. Timeout 30s cho call p99=80ms nghĩa là khi có sự cố, mỗi thread bị giam 30s — chính timeout trở thành cỗ máy cạn tài nguyên.
- **Phân biệt connect timeout (ngắn, 1–3s) và request timeout** (theo nghiệp vụ); thao tác ghi dài (upload, báo cáo) → chuyển async + polling thay vì timeout dài.
- Nhớ chương 01: timeout xảy ra **không cho biết** thao tác thất bại — có thể đã thành công. Mọi xử lý sau timeout phải giả định cả hai khả năng (idempotency — chương 08).

## 3. Retry — con dao hai lưỡi sắc nhất

### 3.1. Vì sao retry vừa bắt buộc vừa nguy hiểm

Phần lớn lỗi mạng là **transient** (packet loss, node vừa restart, LB đang cập nhật) — retry một lần sau 100ms giải quyết im lặng hàng nghìn lỗi/ngày. Nhưng retry là **bộ khuếch đại tải đúng lúc hệ yếu nhất**: service chết vì quá tải, mọi client retry 3 lần = tải x3–x4 → không bao giờ dậy nổi. Tệ hơn theo tầng: A retry 3 lần gọi B, B retry 3 lần gọi C → C nhận **9x** tải — *retry storm* lũy thừa theo độ sâu chuỗi gọi.

### 3.2. Kỷ luật retry — đủ bộ, thiếu một là hỏng

1. **Chỉ retry lỗi transient** (timeout, 503, connection reset). KHÔNG retry 4xx (lỗi của mình, thử lại vô ích), không retry lỗi nghiệp vụ (insufficient funds).
2. **Chỉ retry thao tác idempotent** — hoặc làm nó idempotent trước (idempotency key).
3. **Exponential backoff + FULL JITTER**: `sleep = random(0, min(cap, base × 2^attempt))`. Vì sao jitter bắt buộc: không jitter, mọi client fail cùng lúc sẽ retry *đồng loạt cùng lúc* — tạo sóng tải nhọn tuần hoàn (thundering herd tự chế). Bài phân tích của AWS cho thấy full jitter giảm tổng số call và thời gian hồi phục rõ rệt so với backoff thuần.
4. **Giới hạn số lần (2–3) + retry budget**: giới hạn tỷ lệ retry/tổng request toàn client (ví dụ 10–20%) — khi hệ dưới hỏng diện rộng, budget cạn → ngừng retry → cho nó thở. (Finagle, gRPC, Envoy hỗ trợ sẵn.)
5. **Chỉ retry ở MỘT tầng** (thường tầng gần client nhất hoặc mesh) — các tầng giữa "fail fast, propagate". Retry mọi tầng là công thức lũy thừa ở 3.1.

## 4. Circuit Breaker — ngắt mạch để cứu cả hai bên

### 4.1. Cơ chế

Retry xử lý lỗi *lẻ tẻ*; khi dependency hỏng *kéo dài*, tiếp tục gọi chỉ tốn thread + đè thêm lên kẻ đang hấp hối. Circuit breaker theo dõi tỷ lệ lỗi và **chủ động ngắt**:

```
           lỗi vượt ngưỡng (vd >50% trong 10s,
            tối thiểu N request)
 [CLOSED] ────────────────────────▶ [OPEN]
  gọi bình thường,                   fail NGAY không gọi
  đếm lỗi                            (fallback), chờ cooldown
     ▲                                   │ hết cooldown (vd 30s)
     │ thử nghiệm thành công             ▼
     └──────────────────────────── [HALF-OPEN]
            cho MỘT ÍT request thật đi qua để thăm dò
            (thất bại → OPEN lại, tăng cooldown)
```

Giá trị kép: (a) caller fail-fast trong ~µs thay vì giam thread chờ timeout — chặn cạn tài nguyên; (b) service hỏng được **giảm tải để hồi phục** — đây là điều retry không bao giờ làm được.

### 4.2. Chi tiết production hay sai

- Ngưỡng cần **volume tối thiểu** (2 lỗi/2 request = 100% nhưng vô nghĩa thống kê).
- Breaker theo **từng dependency/endpoint**, không toàn cục (search chết không được ngắt luôn checkout).
- HALF-OPEN thả *ít* request — thả toàn bộ là dội búa vào kẻ mới gượng dậy.
- Breaker OPEN phải có **metric + alert riêng**: breaker mở là sự kiện vận hành, không phải chi tiết ẩn.
- Quan trọng nhất: **fallback là quyết định nghiệp vụ** — trả cache cũ? giá trị mặc định? xếp hàng xử lý sau? hay lỗi tử tế? Không thiết kế fallback thì circuit breaker chỉ đổi "chết chậm" thành "chết nhanh" (đôi khi thế cũng đã tốt, nhưng hãy chọn có chủ đích).

## 5. Bulkhead — cô lập khoang tàu

Tên từ vách ngăn tàu thủy: thủng một khoang không chìm cả tàu. Kỹ thuật: **chia tài nguyên theo dependency/loại việc** — pool 100 thread chung cho mọi outbound call nghĩa là một dependency treo giam đủ 100 thread và mọi dependency khỏe khác cũng chết theo (đây chính là mắt xích 2 của chuỗi cascade §1). Tách: pool 20 cho Payment, 20 cho Search, 10 cho Recommendation — Recommendation treo thì chỉ 10 thread bị giam, 90% hệ vẫn thở.

Áp dụng ở mọi tầng: thread/connection pool per-dependency (Hystrix/Resilience4j), Kubernetes resource limits + PodDisruptionBudget, node pool riêng cho workload critical, **cell-based architecture** (chia toàn hệ thành N cell độc lập, mỗi cell phục vụ 1/N khách — sự cố tệ nhất có blast radius 1/N; AWS và Slack vận hành kiểu này), shuffle sharding (mỗi khách được gán một tổ hợp node gần-như-duy-nhất — khách "độc" chỉ đầu độc tổ hợp của nó, xác suất trùng hoàn toàn với khách khác cực nhỏ).

Giá: tài nguyên dùng kém hiệu quả hơn (pool tách nhỏ có lúc chỗ thừa chỗ thiếu) — đó chính là mua isolation bằng utilization.

## 6. Backpressure & Load Shedding — nói "không" đúng lúc

**Backpressure**: hệ dưới báo hệ trên "chậm lại" — lan ngược áp lực thay vì nhận vô hạn rồi chết. Cơ chế: bounded queue (queue không giới hạn là lời hứa dối — nhận mọi thứ, trả bằng OOM + latency vô hạn; **mọi queue trong hệ của bạn phải có giới hạn và hành vi-khi-đầy được định nghĩa**), TCP flow control, reactive streams (`request(n)`), Kafka consumer tự kéo theo sức mình (pull model — backpressure bẩm sinh, một lý do lớn để chọn pull).

**Load shedding**: khi bão đến, **chủ động bỏ bớt** để phần còn lại sống: từ chối sớm ở edge (rẻ nhất), ưu tiên theo loại (bỏ analytics, giữ checkout), degrade tính năng (tắt recommendation, trang chủ tĩnh). Kèm một quyết định tinh tế: request đã chờ trong queue lâu hơn timeout của client → **vứt luôn, đừng xử lý** — client đã bỏ đi, xử lý nữa là đốt CPU cho người không còn nghe (Google gọi là "serving the dead").

**Graceful Degradation** là mặt nghiệp vụ của load shedding — thiết kế *trước* các mức: đầy đủ → không personalize → đọc-only → trang tĩnh. Netflix: trang chủ cá nhân hóa hỏng → danh sách phổ biến tĩnh — người dùng thường không nhận ra.

## 7. Rate Limiting — hàng rào chủ động

Khác load shedding (phản ứng khi quá tải), rate limit là **hợp đồng đặt trước**: mỗi client X req/s — chống abuse, chống một tenant ăn hết tài nguyên (noisy neighbor), bảo vệ dependency hạ nguồn có hạn mức.

| Thuật toán | Cơ chế | Đặc tính |
|---|---|---|
| **Token bucket** | Bucket nạp token đều đặn (rate), chứa tối đa burst; request lấy 1 token | Cho phép burst có kiểm soát — mặc định đúng cho API |
| Leaky bucket | Xả đều đặn bất kể vào thế nào | Làm mượt output — tốt khi hạ nguồn cần nhịp đều |
| Fixed window | Đếm trong mỗi cửa sổ (1000/phút) | Rẻ; lỗ hổng biên cửa sổ (2000 req trong 2 giây quanh ranh giới phút) |
| Sliding window | Cửa sổ trượt (log hoặc xấp xỉ weighted) | Chính xác hơn, đắt hơn chút — dùng bản xấp xỉ |

Distributed rate limiting: đếm chính xác toàn cục cần Redis tập trung (thêm hop + SPOF — hãy Lua-script cho atomic) hoặc chấp nhận **đếm xấp xỉ per-node × N** (sai số đổi lấy độc lập — thường là lựa chọn đúng). Trả **429 + Retry-After**, tài liệu hóa hạn mức, có tầng ưu tiên (internal > paid > free).

## 8. Dead Letter Queue — nghĩa trang có quản trang

Consumer xử lý message thất bại N lần (poison message: schema lỗi, bug consumer, dữ liệu góc cạnh) → không được chặn queue (head-of-line blocking) và không được vứt → chuyển vào **DLQ** kèm metadata (lỗi gì, mấy lần, stack trace).

Kỷ luật vận hành — DLQ không có quy trình là hố đen nghiệp vụ: alert khi DLQ có message mới (mỗi message là một *nghiệp vụ chưa hoàn thành* — đơn hàng không giao, email không gửi); dashboard tuổi + số lượng; công cụ **redrive** (sửa bug → đưa message về queue chính xử lý lại — cần idempotency, chương 08); phân loại nguyên nhân định kỳ (bug consumer → sửa; schema producer → trả về team đó; dữ liệu rác → chặn từ nguồn).

## 9. Trade-off & phối hợp các pattern

Bức tranh ghép — một request đi qua đủ bộ:

```
Client → [Rate limiter] → [Bulkhead: pool riêng] → [Circuit breaker] 
       → [Timeout] → gọi dependency
                       │ lỗi transient → [Retry: budget + backoff + jitter]
                       │ lỗi kéo dài  → breaker OPEN → [Fallback/Degrade]
       (async path)  → [Bounded queue] → consumer → thất bại N lần → [DLQ]
```

- **Utilization vs Isolation** (bulkhead): tách pool là chấp nhận lãng phí để mua blast radius nhỏ.
- **Recovery nhanh vs An toàn** (retry/breaker): retry hăng hái phục hồi nhanh lỗi lẻ nhưng giết hệ đang quá tải; breaker bảo thủ an toàn nhưng false-positive làm mất traffic tốt. Không có số đúng tuyệt đối — chỉ có số *được đo và diễn tập*.
- **Đơn giản vs Đầy đủ**: đủ bộ pattern trên mọi call là over-engineering cho hệ 3 service. Thứ tự đầu tư theo giá trị/chi phí: **(1) timeout đúng mọi nơi, (2) retry có kỷ luật ở một tầng, (3) bounded queue mọi chỗ, (4) circuit breaker cho dependency hay hỏng, (5) bulkhead cho critical path, (6) load shedding ở edge.** Ba mục đầu gần như miễn phí và chặn 80% cascade.
- Vị trí đặt: **library (Resilience4j) vs service mesh (Envoy/Istio)** — mesh cho đồng nhất đa ngôn ngữ, config tập trung; library cho fallback gắn nghiệp vụ (mesh không biết "trả cache cũ" nghĩa là gì). Thực tế thường lai: mesh lo retry/timeout/outlier detection, app lo fallback.

## 10. Production, Anti-patterns & Troubleshooting

**Giám sát**: timeout rate, retry rate (và retry/total ratio), breaker state changes, queue depth + tuổi message già nhất, shed/reject count, DLQ size. **Diễn tập**: chaos engineering — tiêm latency (quan trọng hơn tiêm chết: *chậm nguy hiểm hơn chết* vì chết thì fail-fast, chậm thì giam tài nguyên), giết dependency, đầy queue — trong giờ hành chính, có kiểm soát.

**Anti-patterns**: retry vô hạn / không jitter / mọi tầng cùng retry; timeout mặc định vô hạn; unbounded queue; breaker toàn cục một cái cho mọi dependency; fallback chưa từng được test (fallback đọc cache — cache cũng chết cùng sự cố thì sao?); DLQ không ai xem; rate limit chỉ ở một node trong khi client đi qua LB round-robin; "chúng tôi có Hystrix nên yên tâm" (pattern không thay được capacity — thiếu máy thì mọi pattern chỉ chọn *ai* bị từ chối).

**Troubleshooting nhanh**:

| Triệu chứng | Nghi vấn | Kiểm tra |
|---|---|---|
| Sự cố nhỏ hóa sập diện rộng | Cascade: pool chung, retry storm, không breaker | Trace từ dependency hỏng ngược lên; đo retry ratio lúc sự cố |
| Service hồi phục xong lại sập ngay | Thundering herd: client dồn retry + queue tồn | Jitter, retry budget; xả queue có kiểm soát; warm-up |
| Latency tăng theo bậc thang mỗi 30s | Chuỗi timeout lồng nhau cộng dồn | Rà timeout budget các tầng |
| Queue depth tăng mãi không hồi | Consumer chậm hơn producer (bền vững) — không phải bug, là thiếu capacity | Tính toán lại: scale consumer hoặc shed ở nguồn |
| Breaker flapping (mở-đóng liên tục) | Ngưỡng quá nhạy / dependency hỏng chập chờn | Volume tối thiểu, cooldown lũy tiến; sửa gốc |
| Một tenant làm chậm mọi tenant | Thiếu bulkhead/rate limit per-tenant | Phân tích tải theo tenant; shuffle sharding |

---

## Tổng kết chương

### Những điều bắt buộc phải nhớ
1. Kẻ thù số một là **cascade**, và bộ khuếch đại của nó là **retry không kỷ luật** + **pool/queue không giới hạn**.
2. Retry đủ bộ: transient-only, idempotent-only, backoff + full jitter, giới hạn lần, retry budget, một tầng duy nhất.
3. Circuit breaker làm điều retry không làm được: **giảm tải cho kẻ đang hỏng**; fallback là quyết định nghiệp vụ.
4. Mọi queue phải bounded; mọi call phải có timeout theo ngân sách; chậm nguy hiểm hơn chết.
5. Ở quá tải, từ chối có chọn lọc (shed/degrade) là hành vi *đúng*, không phải thất bại.
6. Pattern không thay capacity — chúng chỉ quyết định hệ hỏng *có trật tự* hay hỏng *hỗn loạn*.

### Hiểu lầm phổ biến
- "Retry nhiều = resilient" — ngược lại ở đúng lúc quan trọng nhất.
- "Timeout dài an toàn hơn" — timeout dài giam tài nguyên, biến sự cố nhỏ thành cạn kiệt.
- "Circuit breaker tự biết fallback" — nó chỉ biết ngắt; phần "rồi sao nữa" là việc của bạn.
- "Queue càng to càng an toàn" — queue to = latency ẩn + OOM hẹn giờ + che giấu thiếu capacity.

### Câu hỏi tự kiểm tra
1. Vẽ lại chuỗi cascade ở §1 và chỉ ra chính xác pattern nào cắt mắt xích nào.
2. A→B→C, mỗi tầng retry 3 lần, C sập. Tính hệ số khuếch đại tải lên C. Sửa bằng chính sách gì ở tầng nào?
3. Thiết kế rate limiting cho API 3 hạng khách (free/paid/internal) chạy trên 20 node sau LB: thuật toán, nơi giữ đếm, xử lý sai số.
4. Fallback cho trang chi tiết sản phẩm khi service giá chết: liệt kê 3 phương án và rủi ro nghiệp vụ từng cái (hiện giá cache cũ → bán sai giá?).

### Tài liệu kinh điển nên đọc
- **"Release It!" (Michael Nygard)** — cuốn sách khai sinh vocabulary của chương này (circuit breaker, bulkhead) từ những sự cố có thật; chương nào cũng là một vết sẹo production.
- **AWS Architecture Blog: "Exponential Backoff and Jitter"** — phân tích định lượng vì sao full jitter thắng; ngắn, có số liệu.
- **Google SRE Book: "Handling Overload" + "Addressing Cascading Failures"** — load shedding, retry budget, serving-the-dead; kinh nghiệm ở quy mô hành tinh.
- **"The Tail at Scale" (Dean & Barroso)** — hedged requests và tư duy tail-tolerant.
- **AWS Builders' Library: "Shuffle Sharding" & "Avoiding insurmountable queue backlogs"** — hai bài thực chiến sâu, ít nơi dạy.
