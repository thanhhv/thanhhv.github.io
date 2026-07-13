+++
title = "2.1. Vertical vs Horizontal Scaling — và bài toán state"
date = "2026-07-13T06:20:00+07:00"
draft = false
tags = ["backend", "system-design"]
series = ["System Design — Tư Duy Thiết Kế Hệ Thống"]
+++

## 1. Problem Statement

Hệ thống chạm trần một máy: CPU bão hòa giờ peak, latency gãy ([1.3 — hockey stick](/series/system-design/01-foundations/03-throughput-latency/)). Hai con đường: **máy to hơn** (vertical/scale-up) hay **nhiều máy hơn** (horizontal/scale-out)? Câu hỏi nghe như so sánh giá tiền — thực chất là so sánh hai triết lý kiến trúc, và con đường thứ hai có một điều kiện tiên quyết bị đánh giá thấp một cách hệ thống: **state phải dọn nhà trước**.

## 2. Vertical Scaling — lá bài rẻ bị coi thường

Thêm CPU/RAM/NVMe cho máy hiện có. Không đổi một dòng code, không đổi kiến trúc, không thêm failure mode. Bị coi thường vì "không cool", trong khi:

- Trần hiện đại rất cao: máy cloud 128+ vCPU, vài TB RAM là hàng đặt được trong ngày; một PostgreSQL trên máy như vậy phục vụ được lượng tải mà 90% công ty không bao giờ đạt tới ([5.1 §4](/series/system-design/05-data-layer/01-postgresql/)).
- Với hệ **stateful** (DB), scale-up là lựa chọn *mặc định đúng* trong thời gian dài — vì scale-out storage là sharding, món đắt nhất thực đơn ([8.1](/series/system-design/08-data-partitioning/01-partitioning-sharding/)).

Giới hạn thật của scale-up, theo thứ tự chạm phải: (1) **giá phi tuyến ở cận trần** — máy gấp đôi thường đắt hơn gấp đôi từ một cỡ nào đó; (2) **vẫn là SPOF** — máy to nhất thế giới vẫn chết được, và downtime khi nâng cấp/thay máy to là downtime thật; (3) **trần tuyệt đối tồn tại** — hết cỡ là hết, không thương lượng. Vậy scale-up mua *thời gian*, không mua *tương lai vô hạn* — và mua thời gian là món hời khi thời gian đó dùng để hoãn độ phức tạp ([12 bài học 2](/series/system-design/12-evolution/00-tong-quan/)).

## 3. Horizontal Scaling — và cái giá tên là state

Nhiều máy giống nhau sau một LB. Trần lý thuyết gần vô hạn, chết một máy còn N−1 ([availability đi kèm miễn phí — Phần 3](/series/system-design/03-availability-reliability/00-tong-quan/)), chi phí tuyến tính theo tải. Nhưng có điều kiện tiên quyết:

**Một request bất kỳ phải phục vụ được bởi một instance bất kỳ.** Điều này chỉ đúng khi instance **stateless** — không giữ thứ gì mà request sau cần tìm lại. Kiểm kê những thứ hay "bám" vào instance và chỗ dọn đến:

| State bám instance | Triệu chứng khi scale-out | Dọn về đâu |
|---|---|---|
| Session trong memory | User bị logout ngẫu nhiên theo instance nhận request | Redis ([12.2](/series/system-design/12-evolution/02-them-redis/)) hoặc token tự chứa (JWT — [Phần 11](/series/system-design/11-security/00-tong-quan/)) |
| File upload ghi disk local | File "biến mất" tùy máy | Object storage |
| Cache in-process có thẩm quyền | Mỗi máy một sự thật | Distributed cache; local cache chỉ giữ bản sao có TTL ([7.3](/series/system-design/07-caching/03-distributed-cache/)) |
| Cron/scheduler trong app | Chạy N lần khi có N instance | Worker riêng + lock/lease ([4.3](/series/system-design/04-distributed-systems/03-consensus-quorum-leader-election/)), hoặc queue ([12.3](/series/system-design/12-evolution/03-background-worker/)) |
| WebSocket connection | Bản chất stateful — không dọn được | Chấp nhận stateful tier riêng có registry/pub-sub (bài của Chat case study — [Phần 14](/series/system-design/14-case-studies/00-tong-quan/)) |
| Trạng thái xử lý dở (multi-step trong RAM) | Mất khi instance chết/deploy | DB/queue — mỗi bước bền hóa ([6.7 — saga state](/series/system-design/06-communication/07-saga/)) |

**Sticky session — cách né việc dọn nhà, và vì sao chỉ nên là nạng tạm:** LB găm user vào một instance (cookie/IP hash) để state trong memory "vẫn chạy". Giá phải trả: phân tải lệch (user nặng dồn máy xui — chính là [hot partition](/series/system-design/13-production-failure-cases/02-database-failures/) tầng app); instance chết = mất session của cả nhóm user; deploy/scale-in phức tạp (phải chờ session cạn — connection draining kéo dài); autoscaling kém hiệu quả (máy mới không nhận user cũ). Dùng nó có ý thức như **giải pháp chuyển tiếp có ngày hết hạn** trong lúc di dời state — không phải như kiến trúc.

## 4. First Principles

**Vì sao stateless scale được còn stateful thì không?** Vì stateless nghĩa là *mọi instance thay thế được nhau hoàn hảo* — thêm máy = thêm công suất, thuần túy số học. State phá tính thay thế đó: request phải *tìm về* nơi giữ state → routing có trí nhớ → phân tải lệch → mất luôn cả tính đơn giản của failover. Scale ngang không "giải" bài toán state — nó **đẩy** bài toán state xuống các hệ chuyên trách (DB, Redis, Kafka) là những nơi đã trả chi phí kỹ nghệ khổng lồ để vừa giữ state vừa scale ([4.2](/series/system-design/04-distributed-systems/02-replication-consistency/), [Phần 8](/series/system-design/08-data-partitioning/00-tong-quan/)). "App stateless" thực chất là tuyên bố phân công: *tầng này không giữ state, để các tầng giỏi việc đó giữ.*

**Nếu bỏ qua và scale-out app còn state thì sao?** Hệ vẫn chạy — đến khi có 2 instance trở lên, và các bug xuất hiện *theo xác suất routing*: user A lúc thấy giỏ hàng lúc không, file lúc có lúc mất — loại bug tồi nhất để debug vì không tái hiện được theo ý muốn ([12.2 §3 — điều kiện stateless khi thêm LB](/series/system-design/12-evolution/02-them-redis/)).

**Amdahl nhắc nhở:** scale-out chỉ nhân phần *song song hóa được*; phần tuần tự (single DB, global lock, leader) là trần thật ([1.5 §4](/series/system-design/01-foundations/05-bottleneck-analysis/)) — thêm 100 app instance không đẩy trần đó lên một milimét.

## 5. Trade-off

| | Vertical | Horizontal |
|---|---|---|
| Thay đổi kiến trúc | Không | Phải stateless hóa + LB + (nếu chưa có) CI/CD cho N máy |
| Trần | Có, tuyệt đối | Gần vô hạn cho tầng stateless |
| Availability | Vẫn SPOF; nâng cấp = downtime | N−1 sống tiếp; rolling deploy không downtime |
| Chi phí | Phi tuyến ở cận trần | Tuyến tính; nhưng cộng chi phí LB + vận hành N máy |
| Failure mode mới | Không | Phân tải lệch, partial failure, config drift giữa các máy |
| Phù hợp | Stateful (DB), hệ nhỏ-vừa, mua thời gian | Tầng app/web, tải biến động, cần HA |

Thực tế mọi hệ trưởng thành là **lai**: app scale-out (rẻ, sạch) + DB scale-up hết cỡ rồi mới replica/shard — đúng trình tự VietShop đã đi ([Phần 12](/series/system-design/12-evolution/00-tong-quan/)).

## 6. Production Considerations

- Scale-out chỉ thật khi **đã diễn tập mất một máy**: giết một instance giờ cao điểm ở staging — traffic có dồn mượt không, capacity còn lại chịu nổi không (N+1 — [1.4 §3.2](/series/system-design/01-foundations/04-scale-estimation-capacity-planning/))?
- **Config/secret đồng nhất qua công cụ** (không SSH sửa tay từng máy) — config drift giữa instance là nguồn bug "chỉ xảy ra trên máy 3".
- Instance phải **khởi động nhanh và tự đăng ký** (health check pass là nhận traffic) — thời gian khởi động là biến số sống còn của autoscaling ([2.3](/series/system-design/02-scalability/03-auto-scaling/)).
- Đo **phân bố tải giữa các instance** (max/median RPS per instance) — lệch kéo dài chỉ điểm sticky ngầm (keep-alive dồn connection, LB thuật toán kém — [2.2](/series/system-design/02-scalability/02-load-balancer/)).

## 7. Anti-patterns

- **Scale-out app khi nghẽn DB** — thêm connection đè DB, tệ hơn điểm xuất phát ([1.5 §8](/series/system-design/01-foundations/05-bottleneck-analysis/)).
- **Sticky session như kiến trúc vĩnh viễn** — mọi giá đã kê ở §3, cộng lãi kép theo thời gian.
- **"Stateless" nhưng cron trong app** — job chạy N lần, thường phát hiện khi khách nhận 4 email giống nhau.
- **Scale-up mãi vì ngại LB** — đến trần tuyệt đối mới bắt đầu học scale-out là học trong khủng hoảng.
- **Scale-out sớm vì "chuẩn bị viral"** — N máy cho 100 user: trả chi phí vận hành N máy cho tải của nửa máy ([1.4 §8](/series/system-design/01-foundations/04-scale-estimation-capacity-planning/)).

## 8. Khi nào KHÔNG cần quan tâm

Một máy + backup tốt phục vụ ổn hàng chục nghìn user cho app điển hình ([12.1](/series/system-design/12-evolution/01-monolith-postgresql/)). Khi utilization peak < 40–50% và tăng trưởng < 2×/năm — câu hỏi scaling chưa đến lượt; đầu tư vào [1.5 — biết trần của mình ở đâu](/series/system-design/01-foundations/05-bottleneck-analysis/) và để dành độ phức tạp cho ngày có số đo đòi hỏi.

---

*Tiếp theo: [2.2. Load Balancer](/series/system-design/02-scalability/02-load-balancer/)*
