+++
title = "Bài 8 — Observability"
date = "2026-02-01T14:00:00+07:00"
draft = false
tags = ["backend", "infrastructure", "networking"]
series = ["Infrastructure & Networking"]
+++

# Module 08 — Observability

> Monitoring trả lời "hệ thống có đang hỏng theo cách tôi đã đoán trước không?". Observability trả lời "hệ thống đang làm cái quái gì vậy?" — kể cả với lỗi chưa ai từng đoán. Trong hệ phân tán, lỗi thú vị luôn là lỗi chưa ai đoán trước.

---

## 1. Problem Statement

Một request đi qua 15 service. User báo "chậm". Không có observability: 15 team nhìn 15 dashboard riêng, ai cũng "service tôi bình thường", 4 tiếng meeting để tìm ra một connection pool cạn ở service thứ 9. Chi phí thật của thiếu observability đo bằng **MTTR** (giờ thay vì phút) và bằng những sự cố âm ỉ không ai biết (lỗi 0.5% user suốt 3 tuần).

Ba trụ (log, metric, trace) không phải ba lựa chọn — chúng trả lời ba câu hỏi khác nhau và **giao nhau tại nhau**: metric nói CÓ vấn đề (rẻ, tổng hợp), trace nói vấn đề Ở ĐÂU trong chuỗi gọi, log nói CHÍNH XÁC CHUYỆN GÌ tại điểm đó. Thiết kế tốt = nhảy được giữa ba loại (exemplar: từ điểm bất thường trên biểu đồ metric → click ra đúng trace → từ span ra đúng dòng log qua trace_id).

---

## 2. Logging

### Nguyên tắc
- **Structured (JSON) hoặc là thua**: log để máy truy vấn, không phải để người đọc tuần tự. `{"level":"error","trace_id":"abc","user_id":123,"msg":"payment declined","code":"insufficient_funds"}` — filter, aggregate, join được. Log văn xuôi = grep và cầu nguyện.
- **Trace ID trong MỌI dòng log** — sợi chỉ khâu ba trụ với nhau. Không có nó, log của 15 service là 15 đống cát rời.
- Log là **sự kiện**, không phải trạng thái định kỳ (trạng thái định kỳ là việc của metric). Anti-pattern: log "health check OK" mỗi giây × 500 pod = trả tiền ingest cho rác.

### Bài toán chi phí — vấn đề thật của logging ở quy mô
Log ingest + retention là hạng mục observability phình nhanh nhất (đã nói ở module 07). Đòn bẩy theo thứ tự: (1) đừng log cái không bao giờ truy vấn (audit lại top volume — luôn có một logger debug bị quên); (2) **sampling log INFO của request thành công** (giữ 100% error +100% request chậm — cái bạn cần khi điều tra); (3) tier hóa retention: 7 ngày nóng truy vấn nhanh, 90 ngày lạnh trên object storage; (4) log mức request-summary (1 dòng/request kiểu access log giàu context — "canonical log line") thay vì 20 dòng vụn/request.

**Cẩn trọng**: log là nơi rò rỉ PII/secret phổ biến nhất (dump cả request body có password/token). Redaction ở thư viện logging, không dựa vào ý thức từng dev.

---

## 3. Metrics và Prometheus

### Bản chất của metric
Chuỗi số theo thời gian, **chi phí không phụ thuộc traffic** (đếm 1 req/s hay 1M req/s vẫn là một con số mỗi 15s) — vì thế metric là trụ rẻ nhất và là nơi đặt **alert**. Đổi lại: mất chi tiết từng sự kiện (đã tổng hợp), và chi phí đi theo **cardinality** chứ không theo volume.

### Cardinality — khái niệm quan trọng nhất, nơi mọi hệ metric chết
Mỗi tổ hợp label = một time series riêng trong RAM. `http_requests{path, status, pod}` với 100 path × 5 status × 200 pod = 100.000 series cho MỘT metric. Thêm label `user_id` (1M giá trị) = **nổ tung Prometheus** (OOM — cardinality explosion, tai nạn phổ biến nhất). Quy tắc: label chỉ chứa giá trị **hữu hạn và biết trước** (status, method, endpoint đã template hóa `/users/{id}` chứ không phải `/users/12345`). Định danh cá thể (user_id, request_id) thuộc về log/trace.

### Mô hình Prometheus
**Pull-based**: Prometheus chủ động scrape `/metrics` của từng target (tìm target qua service discovery — trong k8s là tự động). Vì sao pull: biết ngay target chết (scrape fail → alert `up == 0`), không sợ fleet đẩy sập monitoring, cấu hình tập trung. Trade-off: job ngắn hạn cần Pushgateway; scale ngang cần federation/Thanos/Mimir (một Prometheus giữ mọi series trong RAM — trần thực tế vài triệu series).

Bốn loại: Counter (chỉ tăng — đếm sự kiện; luôn dùng với `rate()`), Gauge (lên xuống — trạng thái), Histogram (phân phối vào bucket — **nguồn của p99**; bucket phải phủ dải latency thực tế), Summary (ít dùng, không aggregate được qua nhiều instance).

**Bẫy toán học của percentile**: KHÔNG BAO GIỜ lấy trung bình của p99 các instance ("avg of p99" vô nghĩa toán học). Aggregate histogram bucket qua fleet rồi mới tính `histogram_quantile`. Và p99 toàn fleet có thể đẹp trong khi 1 pod chết dí — nhìn cả max và per-instance khi điều tra.

### Bộ metric chuẩn cho mọi service
- **RED** (cho service): Rate, Errors, Duration (histogram).
- **USE** (cho tài nguyên): Utilization, Saturation (độ sâu hàng đợi — thường báo trước Utilization), Errors.
- Cộng: metric nghiệp vụ (đơn hàng/phút — thứ CEO hiểu, và là thứ phát hiện sự cố mà metric hạ tầng bỏ sót), và các metric "đã học bằng máu" từ module trước: pool wait time, CPU throttling, fd count, TIME_WAIT.

---

## 4. Tracing và OpenTelemetry

### Cách hoạt động
Trace = cây span; span = một đơn vị việc (service, DB call) có start/duration/attributes. **Context propagation** là phần lõi: `traceparent` header (W3C) đi theo request qua mọi hop — HTTP, gRPC metadata, và **cả message queue** (điểm hay bị đứt: producer phải nhét context vào message attribute, consumer phải nối lại; quên là trace đứt đôi ở queue).

Trace trả lời trực tiếp: thời gian đi đâu trong chuỗi 15 service (span dài nhất), gọi tuần tự thứ đáng lẽ song song (bậc thang trong waterfall), retry ẩn (span lặp), N+1 query (100 span DB giống nhau).

### Sampling — vì trace đắt
100% trace ở 100k req/s là bất khả thi về chi phí. **Head sampling** (quyết định từ đầu, vd 1%): rẻ, đơn giản, nhưng 99% khả năng **mất đúng trace lỗi hiếm** bạn cần. **Tail sampling** (thu hết, quyết định giữ sau khi biết kết cục — giữ 100% error/chậm, 0.1% thành công): giữ đúng cái quý, giá là buffer + collector cluster phức tạp hơn. Hệ trưởng thành dùng tail sampling.

### OpenTelemetry
Chuẩn hóa API + SDK + wire format (OTLP) cho cả ba trụ → instrument MỘT lần (phần đắt nhất, nằm trong code của bạn), đổi backend tùy ý (Jaeger/Tempo/Datadog/vendor nào cũng nhận OTLP) → thoát lock-in ở đúng tầng cần thoát. Kiến trúc chuẩn: app (OTel SDK, auto-instrumentation cho HTTP/gRPC/DB phổ biến) → **OTel Collector** (agent/gateway — buffer, batch, sample, redact, route) → backend. Collector là điểm điều khiển chi phí và là buffer khi backend chết — đừng cho app gửi thẳng vendor.

---

## 5. SLI / SLO / SLA và Alerting

### Vì sao SLO tồn tại
"Uptime 100%" là mục tiêu vô nghĩa (không thể đạt, và mỗi số 9 thêm đắt gấp ~10 lần). SLO biến reliability thành **quyết định kinh tế có ngân sách**: chọn mức đủ tốt cho user (99.9%? 99.95%?), phần còn lại là **error budget** — ngân sách để chi cho deploy nhanh, thí nghiệm, và sự cố không tránh khỏi.

- **SLI** = phép đo (tỷ lệ request thành công < 500ms, đo **tại nơi gần user nhất** — LB/client, không phải self-report của service).
- **SLO** = mục tiêu nội bộ (99.9% trong 30 ngày ⇒ budget 43 phút downtime/tháng).
- **SLA** = hợp đồng thương mại có đền bù — luôn lỏng hơn SLO (SLO 99.95 nội bộ đỡ SLA 99.9 ký với khách).

**Error budget là cơ chế ra quyết định**: còn budget → cứ deploy mạnh dạn; cháy budget → đóng băng release, dồn sức cho reliability. Nó chấm dứt cuộc cãi nhau muôn thuở dev-muốn-nhanh vs ops-muốn-ổn bằng một con số cả hai đã đồng ý trước.

### Alerting — nghệ thuật KHÔNG đánh thức người
Sự thật vận hành: **alert nhiễu nguy hiểm hơn thiếu alert** — alert fatigue làm người ta mute channel, và sự cố thật trôi qua trong tiếng ồn. Nguyên tắc:
1. **Page (đánh thức) chỉ khi SLO bị đe dọa và cần người NGAY**. Mọi thứ khác là ticket xử lý giờ hành chính. "CPU 80%" không bao giờ đáng page — user không cảm nhận CPU, user cảm nhận latency/error.
2. **Alert trên symptom (SLI), không trên cause** (disk, pod restart, một node chết). Cause-alert cho hệ tự chữa lành (k8s) đa phần là nhiễu — hệ đã tự xử.
3. **Burn-rate alert** (chuẩn SRE): page khi budget cháy nhanh (burn rate 14× trong 1h — sẽ hết budget tháng trong 2 ngày), ticket khi cháy chậm (2× trong 24h). Vừa nhanh với sự cố lớn, vừa không ồn với lỗi lắt nhắt.
4. Mọi alert PHẢI có runbook link và hành động rõ. Alert không ai biết phải làm gì khi nhận = xóa.

### Checklist
- [ ] JSON log + trace_id xuyên suốt; sampling và redaction có chủ đích
- [ ] RED + USE + business metric; label cardinality được kiểm soát
- [ ] OTel + collector; tail sampling giữ 100% error trace
- [ ] SLO cho user-facing service, đo tại edge; burn-rate alert; error budget được tôn trọng trong quy trình release
- [ ] Từ metric bất thường → trace → log trong < 3 cú click
