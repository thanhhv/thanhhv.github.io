+++
title = "2.3. Auto Scaling — co giãn theo tải mà không thức đêm"
date = "2026-07-13T06:40:00+07:00"
draft = false
tags = ["backend", "system-design"]
series = ["System Design — Tư Duy Thiết Kế Hệ Thống"]
+++

## 1. Problem Statement

Tải không phẳng: ngày gấp 3 đêm, tối gấp 2 trưa, flash sale gấp 10 ngày thường ([1.4](/series/system-design/01-foundations/04-scale-estimation-capacity-planning/)). Provision cho đỉnh = trả tiền công suất ngủ 80% thời gian; provision cho trung bình = gãy latency mỗi tối ([1.3 — hockey stick](/series/system-design/01-foundations/03-throughput-latency/)). Auto scaling hứa điều thứ ba: công suất bám theo tải. Lời hứa có điều kiện — và các điều kiện đó (stateless, khởi động nhanh, metric đúng, biết giới hạn tốc độ của chính nó) là nội dung thật của chương này.

## 2. First Principles — autoscaling là vòng điều khiển có trễ

Bản chất: một control loop — đo metric → so ngưỡng → thêm/bớt máy → đo lại. Mọi tính chất (và mọi tai nạn) của nó là tính chất của **vòng điều khiển có độ trễ**:

- **Trễ đo** (metric tổng hợp theo phút) + **trễ quyết định** (chờ vượt ngưỡng X phút cho chắc) + **trễ thực thi** (boot máy + warm-up: 1–5 phút VM, chậm hơn nếu image nặng) = **tổng trễ phản ứng cỡ vài phút**. Hệ quả không thương lượng được: *autoscaling xử lý được sóng (dao động theo giờ), không xử lý được vách đứng* (spike giây/phút — flash sale mở màn, push notification, [13.1 — thundering herd](/series/system-design/13-production-failure-cases/01-caching-failures/)). Vách đứng cần đệm sẵn: pre-scale theo lịch, warm pool, hoặc tầng hấp thụ (queue — [12.4 §3](/series/system-design/12-evolution/04-message-queue/), cache, CDN).
- **Vòng điều khiển có thể dao động (flapping):** scale-out → tải mỗi máy giảm → metric tụt dưới ngưỡng → scale-in → tải tăng lại → scale-out... Thuốc: cooldown giữa hai hành động, ngưỡng out/in cách xa nhau (ra ở 70%, vào ở 40% — hysteresis), và scale-in *chậm hơn* scale-out (thêm nhanh, bớt rụt rè — bất đối xứng có chủ đích vì giá của thiếu máy cao hơn giá của thừa máy vài phút).
- **Điều kiện tiên quyết kế thừa cả chuỗi:** instance stateless ([2.1](/series/system-design/02-scalability/01-vertical-horizontal-scaling/)) — scale-in giết máy bất kỳ không được làm mất gì; draining + slow-start ở LB ([2.2 §3.3](/series/system-design/02-scalability/02-load-balancer/)); và image/boot tối ưu — mỗi phút boot là một phút user chờ trong kịch bản xấu.

## 3. Scale theo metric nào — quyết định quan trọng nhất

| Metric | Đúng khi | Sai khi |
|---|---|---|
| CPU utilization | Workload CPU-bound (render, serialize nặng) | **Workload I/O-bound: CPU 25% mà thread pool cạn — không bao giờ scale** ([1.5 — nghẽn nhân tạo](/series/system-design/01-foundations/05-bottleneck-analysis/)); đây là lỗi cấu hình autoscaling phổ biến nhất |
| RPS per instance | Request đồng đều, đã đo capacity/instance bằng load test ([1.4 §3.1](/series/system-design/01-foundations/04-scale-estimation-capacity-planning/)) | Request nặng nhẹ lẫn lộn |
| Concurrency / in-flight requests | Đo "việc đang làm" trực tiếp — tốt cho đa số API (Little's Law — [1.3 §2](/series/system-design/01-foundations/03-throughput-latency/)) | Ít nền tảng hỗ trợ trực tiếp (Knative/một số mesh có) |
| Latency p99 | Nghe hợp lý — SLO trực tiếp | **Bẫy:** latency là *triệu chứng trễ* — khi nó tăng thì đã muộn; và tăng có thể do DB chậm — scale app vô ích, thậm chí hại ([2.README](/series/system-design/02-scalability/00-tong-quan/)) |
| **Queue depth / lag** | **Worker/consumer: chuẩn vàng** — tải đo trực tiếp bằng việc đang chờ ([13.3 — Kafka lag](/series/system-design/13-production-failure-cases/03-messaging-failures/)); KEDA scale theo lag là mẫu đúng | Đường sync không có queue |
| Lịch (scheduled) | Chu kỳ biết trước: 20h hằng ngày, ngày 12.12 — **đơn giản nhất và bị bỏ quên nhiều nhất** | Tải không có chu kỳ |

Nguyên tắc chọn: metric phải (1) **phản ánh nghẽn thật** của workload (đo bằng load test, không đoán), (2) **app instance mới giải được** — nếu nghẽn ở DB thì không metric nào cứu, autoscaling app chỉ chuyển sự cố xuống tầng dưới nhanh hơn.

## 4. Trade-off

| Được | Giá |
|---|---|
| Chi phí bám tải — tiết kiệm thật với tải dao động lớn | Độ phức tạp vận hành: một hệ *tự thay đổi chính nó* — sự cố giờ có thêm biến số "cụm lúc đó bao nhiêu máy?" |
| Hấp thụ tăng trưởng từ từ không cần người trực | Vô dụng với vách đứng — vẫn cần pre-scale/đệm; ảo tưởng "có autoscaling rồi" nguy hiểm hơn không có |
| Thay người trực đêm cho sóng thường nhật | Scale-in sai giết máy đang làm việc dở — đòi stateless + draining tuyệt đối |
| Kết hợp spot/preemptible tiết kiệm sâu | Máy bị thu hồi bất kỳ lúc nào — chỉ cho workload chịu được ([12.4 — worker sau queue là ứng viên đẹp](/series/system-design/12-evolution/04-message-queue/)) |

Và một trade-off tổng thể trung thực: với **tải phẳng**, autoscaling gần như không tiết kiệm gì — N máy cố định + headroom ([1.3 §3.3](/series/system-design/01-foundations/03-throughput-latency/)) đơn giản hơn và đủ; autoscaling tỏa sáng ở tải dao động ≥ 2–3×.

## 5. Production Considerations

- **Min/max là hàng rào an toàn, đặt bằng số đo:** min = đủ chịu tải nền + N+1 ([1.4 §3.2](/series/system-design/01-foundations/04-scale-estimation-capacity-planning/)) — không bao giờ để autoscaler kéo cụm về 1 máy lúc 4h sáng rồi 8h sáng leo không kịp; max = trần ngân sách *và* trần mà tầng dưới chịu được (100 app instance × pool 20 connection = 2000 connection đổ vào DB — autoscaling app có thể là súng bắn vào DB, [13.2 — pool exhaustion](/series/system-design/13-production-failure-cases/02-database-failures/)).
- **Giám sát chính autoscaler:** số sự kiện scale, lý do, thời gian từ trigger đến máy nhận traffic (đo cả chuỗi — con số này là "tốc độ phản xạ" thật); alert khi chạm max (chạm max nghĩa là autoscaling đã hết bài — con người vào cuộc).
- **Diễn tập kịch bản xấu:** tắt autoscaler thử vận hành tay (khi API cloud lỗi — [13.5 — region/API down](/series/system-design/13-production-failure-cases/05-infrastructure-failures/)); bơm tải vách đứng ở staging xem hệ gãy thế nào và đệm nào đỡ.
- Quota cloud là trần ẩn: autoscaler xin máy thứ 51 gặp quota 50 — lỗi chỉ xuất hiện đúng hôm cần nhất; kiểm kê quota theo capacity plan ([1.5 §7 — danh sách trần đã biết](/series/system-design/01-foundations/05-bottleneck-analysis/)).
- Pre-scale sự kiện biết trước là **quy trình có checklist** (nâng min trước giờ G, hạ sau khi ổn) — đừng phó mặc ngày doanh thu lớn nhất năm cho vòng điều khiển vài phút trễ ([1.4 §5 — kết hợp ba chiến lược](/series/system-design/01-foundations/04-scale-estimation-capacity-planning/)).

## 6. Anti-patterns

- **Scale theo CPU cho app I/O-bound** — không bao giờ trigger; hệ ngộp với CPU 25%.
- **Không cooldown/hysteresis** — flapping: cụm co giãn liên tục, cache lạnh liên tục, tệ hơn cụm tĩnh.
- **Min = 0 hoặc 1 cho dịch vụ user-facing** — đêm về 1 máy, sáng leo 5 phút, mỗi sáng một cơn nghẽn nhỏ (scale-to-zero là của batch/serverless, không phải của API chính).
- **Autoscaling như giải pháp cho mọi sự cố tải** — nghẽn DB, hot key, retry storm: thêm app máy đều vô ích hoặc phản tác dụng; chẩn đoán trước, scale sau ([1.5 — quy trình](/series/system-design/01-foundations/05-bottleneck-analysis/)).
- **Quên warm-up** — máy mới nhận full traffic với pool rỗng + JIT lạnh: p99 xấu đúng lúc đang cần thêm công suất; slow-start ở LB ([2.2](/series/system-design/02-scalability/02-load-balancer/)).
- **Không giới hạn max** — bug retry storm tự scale cụm lên 400 máy: hóa đơn là alert duy nhất.

## 7. Khi nào KHÔNG cần

Tải phẳng hoặc nhỏ (cụm 2–3 máy cố định + headroom là đủ và dễ hiểu hơn); hệ stateful chưa stateless hóa (autoscaling khi chưa đủ điều kiện là máy tạo sự cố); và giai đoạn đầu khi *biết* tải hình gì quan trọng hơn *phản ứng tự động* với nó — vận hành tay vài tháng, nhìn biểu đồ, rồi tự động hóa cái đã hiểu ([12 — thứ tự chi phí](/series/system-design/12-evolution/00-tong-quan/)). Tự động hóa một thứ chưa hiểu là đóng gói sự chưa hiểu vào một vòng lặp chạy 24/7.

---

*Hết Phần 2. Tiếp theo: [Phần 3 — Availability & Reliability](/series/system-design/03-availability-reliability/00-tong-quan/).*
