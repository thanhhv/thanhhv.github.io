+++
title = "Chương 01 – Foundations: Vì sao Distributed Systems tồn tại"
date = "2026-07-08T17:00:00+07:00"
draft = false
tags = ["backend", "distributed-systems"]
series = ["Distributed Systems"]
+++

## 1. Problem Statement

Không ai *muốn* xây hệ thống phân tán. Hệ phân tán khó debug hơn, đắt hơn, nhiều chế độ lỗi hơn, và đòi hỏi đội ngũ giỏi hơn. Chúng ta xây hệ phân tán vì **bị ép buộc** bởi ba lực:

1. **Giới hạn vật lý của một máy** — CPU, RAM, Disk IO, Network bandwidth đều có trần. Một máy 128 core, 4TB RAM vẫn là *một* máy.
2. **Single Point of Failure (SPOF)** — một máy có xác suất hỏng khác 0. Khi nó hỏng, toàn bộ nghiệp vụ dừng. Không có cấu hình phần cứng nào loại bỏ được rủi ro này.
3. **Địa lý** — người dùng ở Việt Nam gọi server ở Virginia chịu tối thiểu ~220ms round-trip. Tốc độ ánh sáng là giới hạn cứng, không mua được bằng tiền.

Nếu không giải quyết: hệ thống sập khi traffic tăng, mất doanh thu khi máy hỏng, và trải nghiệm người dùng tệ ở xa datacenter. Giải pháp cũ (mua máy to hơn — Scale Up) chỉ trì hoãn vấn đề: giá tăng phi tuyến theo cấu hình, và SPOF vẫn nguyên vẹn.

## 2. Tại sao giải pháp này tồn tại

**Business Problem.** Một e-commerce Việt Nam khởi đầu với 1.000 đơn/ngày: một server MySQL + một app server là đủ, thậm chí là *tối ưu*. Đến 1.000.000 đơn/ngày và các đợt flash sale 12.12 với 50.000 request/giây, cùng SLA "sập 1 giờ mất 2 tỷ doanh thu" — bài toán đổi bản chất: từ "xử lý request" thành "xử lý request **khi một phần hệ thống đang hỏng**".

**Technical Problem.** Trên một máy, bạn có ba thứ quý giá mà bạn không nhận ra mình đang có:

- **Shared memory**: hai thread nhìn thấy cùng một biến, ngay lập tức.
- **Đồng hồ duy nhất**: `now()` chỉ có một giá trị.
- **Failure model đơn giản**: hoặc process sống, hoặc chết. Không có trạng thái "có thể sống, có thể chết, không ai biết".

Bước sang nhiều máy, cả ba biến mất. Đây là điểm mấu chốt của toàn bộ lĩnh vực này.

**Scale Problem.** Định luật Moore về tốc độ single-core đã dừng từ ~2005. Tăng trưởng hiệu năng đến từ *nhiều* core, *nhiều* máy. Phần cứng ép kiến trúc phần mềm đi theo hướng phân tán.

## 3. Bản chất — First Principles

### 3.1. Định nghĩa thực dụng

> "A distributed system is one in which the failure of a computer you didn't even know existed can render your own computer unusable." — Leslie Lamport

Hệ phân tán là hệ thống mà nhiều node phối hợp qua **mạng không đáng tin cậy** để hoàn thành một nhiệm vụ chung. Hai từ khóa: *phối hợp* và *không đáng tin cậy*.

### 3.2. Ba sự thật nền tảng (mọi thứ khác suy ra từ đây)

**Sự thật 1: Mạng không đáng tin cậy và độ trễ không xác định.**
Khi node A gửi message cho node B và không nhận được reply, A **không thể phân biệt** giữa các trường hợp:

```
A ──request──▶ ?          (1) request bị mất trên đường đi
A ──request──▶ B (chết)   (2) B đã chết trước khi nhận
A ──request──▶ B (chậm)   (3) B nhận nhưng đang GC pause 30 giây
A ──request──▶ B ──reply──▶ ?   (4) B xử lý XONG, reply bị mất
```

Trường hợp (4) nguy hiểm nhất: hành động **đã xảy ra** nhưng A tưởng là chưa. Đây là gốc rễ của yêu cầu **Idempotency** (chương 08) và toàn bộ nghệ thuật Retry (chương 11). Nếu bạn chỉ nhớ một điều từ chương này, hãy nhớ: **timeout không cho bạn biết điều gì đã xảy ra, chỉ cho biết bạn đã chờ bao lâu.**

**Sự thật 2: Không có đồng hồ toàn cục.**
Mỗi máy có quartz oscillator riêng, trôi (drift) vài chục ppm — tức vài ms đến vài trăm ms mỗi ngày dù có NTP. Hệ quả: không thể dùng timestamp để xác định thứ tự sự kiện giữa hai máy. "Sự kiện nào xảy ra trước" — câu hỏi tưởng tầm thường — cần cả một chương (chương 12) để trả lời.

**Sự thật 3: Partial Failure.**
Trên một máy: chết là chết hết, trạng thái nhất quán. Trên 100 máy: tại mọi thời điểm, *một vài* máy đang hỏng, *một vài* đường mạng đang đứt, và hệ thống phải tiếp tục chạy đúng. Partial failure không phải trường hợp ngoại lệ — ở quy mô lớn, nó là **trạng thái thường trực**. Google từng công bố: trong năm đầu của một cluster 1.800 máy, kỳ vọng ~1.000 lần hỏng máy đơn lẻ và hàng nghìn lần hỏng đĩa.

### 3.3. 8 ngộ nhận kinh điển (Fallacies of Distributed Computing)

Peter Deutsch và các kỹ sư Sun liệt kê 8 giả định sai mà mọi người mới đều mắc:

| # | Ngộ nhận | Thực tế production |
|---|---|---|
| 1 | Mạng đáng tin cậy | Switch reboot, cáp bị cắt, packet loss 0.01% vẫn là hàng nghìn lỗi/ngày ở 10K rps |
| 2 | Latency = 0 | Cùng DC: 0.5ms. Liên vùng: 50–300ms. Mọi remote call đắt hơn local call ~10⁶ lần |
| 3 | Bandwidth vô hạn | Replication traffic có thể bóp nghẹt traffic nghiệp vụ |
| 4 | Mạng an toàn | Mọi liên kết nội bộ đều cần được coi là có thể bị nghe lén (zero-trust) |
| 5 | Topology không đổi | Autoscaling, deploy, node chết — topology đổi *liên tục* |
| 6 | Có một admin duy nhất | Nhiều team, nhiều vendor, nhiều cloud |
| 7 | Chi phí vận chuyển = 0 | Serialize/deserialize tốn CPU; cross-AZ traffic tốn tiền thật |
| 8 | Mạng đồng nhất | Latency giữa các cặp node khác nhau hàng chục lần |

## 4. Các khái niệm nền tảng

### 4.1. Scalability — khả năng mở rộng

Scalability là quan hệ giữa **tài nguyên thêm vào** và **tải xử lý được thêm**. Hệ scale tốt: gấp đôi máy → gần gấp đôi throughput. Hệ scale kém: gấp đôi máy → +10% throughput vì tất cả tranh nhau một lock hoặc một database.

**Amdahl's Law / Universal Scalability Law (USL):** nếu hệ thống có phần tuần tự hóa (serialization — ví dụ mọi write đi qua một leader) chiếm σ và chi phí phối hợp (crosstalk — ví dụ cache coherence, consensus) chiếm κ, throughput theo số node N:

```
Throughput(N) = N / (1 + σ(N−1) + κN(N−1))
```

Điểm quan trọng: với κ > 0, throughput **đạt đỉnh rồi đi xuống** khi thêm node. Thêm máy có thể làm hệ thống *chậm đi* — điều phản trực giác nhưng thấy thường xuyên trong production (thêm replica cho DB đang nghẽn write không giúp gì, vì write vẫn qua một leader; thêm consumer vượt số partition của Kafka topic không tăng gì).

**Scale Up vs Scale Out:**

| Tiêu chí | Scale Up (Vertical) | Scale Out (Horizontal) |
|---|---|---|
| Cách làm | Mua máy to hơn | Thêm nhiều máy |
| Độ phức tạp phần mềm | Không đổi | Tăng mạnh (partition, replication, consensus) |
| Trần | Cứng (~vài trăm core, vài TB RAM) | Mềm (nghìn node), nhưng USL giới hạn |
| Chi phí | Phi tuyến (máy top-tier đắt gấp 5–10 lần/đơn vị hiệu năng) | Tuyến tính về phần cứng, nhưng chi phí *vận hành và con người* tăng |
| SPOF | Còn nguyên | Loại bỏ được (nếu thiết kế đúng) |
| Downtime khi nâng cấp | Thường có | Rolling upgrade được |
| Khi nào chọn | Tải < trần một máy, team nhỏ, latency nội bộ quan trọng | Tải vượt một máy, cần HA, tải tăng khó dự đoán |

**Kinh nghiệm production:** một máy PostgreSQL hiện đại (64 core, NVMe) xử lý được 50–100K TPS đơn giản. Stack Overflow phục vụ hàng trăm triệu pageview/tháng nhiều năm liền với **một** SQL Server cluster nhỏ và web tier khiêm tốn. **Scale Up trước, Scale Out khi bị ép** — làm ngược lại là anti-pattern phổ biến nhất của thập kỷ microservices.

### 4.2. Availability — khả năng sẵn sàng

Availability = tỷ lệ thời gian hệ thống phục vụ được request.

| SLA | Downtime/năm | Downtime/tháng | Ý nghĩa thực tế |
|---|---|---|---|
| 99% | 3.65 ngày | 7.3 giờ | Chấp nhận được cho internal tool |
| 99.9% | 8.77 giờ | 43.8 phút | Chuẩn SaaS thông thường |
| 99.99% | 52.6 phút | 4.38 phút | Cần automation hoàn toàn — con người không kịp phản ứng |
| 99.999% | 5.26 phút | 26.3 giây | Cần multi-region, chi phí tăng ~10x mỗi số 9 |

Hai công thức phải nhớ:

- **Serial (phụ thuộc chuỗi):** A gọi B gọi C, mỗi cái 99.9% → tổng = 0.999³ ≈ **99.7%**. Đây là lý do "chatty microservices" giết availability: 30 dependency 99.9% → tổng chỉ còn ~97%.
- **Parallel (redundancy):** 2 bản độc lập, mỗi bản 99% → 1 − (0.01)² = **99.99%**. Redundancy là công cụ duy nhất mua thêm số 9 — và nó chỉ hoạt động khi các bản hỏng *độc lập* (khác rack, khác AZ, khác region; cùng bug phần mềm thì hỏng cùng nhau, redundancy vô nghĩa).

### 4.3. Reliability, Fault Tolerance và mối quan hệ với Availability

- **Reliability**: xác suất hệ thống làm *đúng* chức năng trong một khoảng thời gian. Đo bằng MTBF (Mean Time Between Failures).
- **Availability**: tỷ lệ thời gian hệ thống *sẵn sàng*. Availability = MTBF / (MTBF + MTTR).
- **Fault Tolerance**: khả năng hệ thống *tiếp tục đúng* khi một số thành phần hỏng (fault ≠ failure: fault là thành phần hỏng, failure là toàn hệ thống sai).

Nhận xét quan trọng từ công thức Availability: có **hai** con đường tăng availability — tăng MTBF (khó, phần cứng là phần cứng) hoặc **giảm MTTR** (dễ hơn nhiều: tự động failover trong 10 giây thay vì on-call engineer thức dậy trong 30 phút). Các hệ thống hiện đại chọn triết lý "embrace failure": không cố ngăn hỏng, mà làm cho việc hỏng **rẻ và nhanh phục hồi**. Đây là lý do Netflix chạy Chaos Monkey — chủ động giết instance trong giờ hành chính để chắc chắn hệ thống chịu được.

### 4.4. Những vấn đề CHỈ xuất hiện khi có nhiều node

Đây là "cái giá vào cửa" của distributed systems — danh sách các bài toán mà hệ một máy **không hề có**:

| Vấn đề | Bản chất | Chương xử lý |
|---|---|---|
| Data không nhất quán giữa các bản sao | Replication lag, mạng chậm | 03, 05 |
| Không biết node khác sống hay chết | Failure detection chỉ là phỏng đoán qua timeout | 07 |
| Split Brain | Hai node cùng tưởng mình là leader | 07 |
| Không có thứ tự sự kiện toàn cục | Không có đồng hồ chung | 12 |
| Message trùng lặp | Retry sau timeout khi thao tác đã thành công | 08, 11 |
| Transaction trải trên nhiều node | Không có shared memory để lock | 08 |
| Hot partition | Dữ liệu/tải phân bố lệch | 06 |
| Cascading failure | Một node chậm kéo sập cả chuỗi | 11 |

## 5. Trade-off

**Đây là trade-off gốc, mọi trade-off khác là hệ quả:**

```
        MỘT MÁY                          NHIỀU MÁY
┌─────────────────────┐        ┌──────────────────────────┐
│ + Đơn giản          │        │ + Không SPOF             │
│ + Consistency free  │  ───▶  │ + Scale gần tuyến tính   │
│ + Debug dễ          │        │ + Chịu lỗi từng phần     │
│ − SPOF              │        │ − Consistency phải TRẢ GIÁ│
│ − Trần hiệu năng    │        │ − Phức tạp vận hành 10x  │
│ − Trần availability │        │ − Failure mode mới       │
└─────────────────────┘        └──────────────────────────┘
```

- **Simplicity vs Scalability**: mỗi lớp thêm vào (cache, queue, shard) tăng khả năng chịu tải nhưng tăng số failure mode theo cấp số nhân. Chi phí thật không nằm ở lúc xây mà ở 5 năm vận hành sau đó.
- **Consistency vs Availability**: khi mạng phân mảnh (partition), phải chọn: từ chối phục vụ (giữ consistency) hoặc phục vụ dữ liệu có thể cũ (giữ availability). Nguyên nhân kỹ thuật: khi hai nhóm node không liên lạc được với nhau, không tồn tại thuật toán nào cho phép cả hai nhóm cùng nhận write mà không phân kỳ dữ liệu. Chi tiết ở chương 04.
- **Cost vs Reliability**: mỗi số 9 thêm vào tốn ~10x chi phí (thêm region, thêm replica, thêm on-call, thêm kiểm thử DR). Câu hỏi đúng không phải "làm sao đạt 99.999%" mà là "nghiệp vụ này *cần* bao nhiêu số 9, và ai trả tiền cho số 9 đó".
- **Latency vs Throughput**: batching tăng throughput (gộp 100 message một lần ghi đĩa) nhưng message đầu tiên trong batch phải *chờ* — tăng latency. Kafka là ví dụ kinh điển: `linger.ms` chính là núm vặn trade-off này.

## 6. Production Considerations

- **Capacity Planning**: đo baseline (p50/p95/p99 latency, throughput đỉnh), thiết kế cho 2–3x đỉnh dự kiến, load test trước sự kiện lớn. Đừng thiết kế cho 100x — YAGNI áp dụng cả với scale.
- **Monitoring**: 4 golden signals (Google SRE): **Latency, Traffic, Errors, Saturation**. Luôn đo percentile, không đo trung bình — trung bình che giấu tail latency, mà tail latency là thứ người dùng cảm nhận (1% request chậm ở hệ có 100 dependency nghĩa là ~63% *người dùng* gặp ít nhất một request chậm).
- **Alerting**: alert trên **triệu chứng người dùng thấy** (error rate, p99), không alert trên nguyên nhân (CPU cao). CPU 90% mà latency tốt thì là tối ưu chi phí, không phải sự cố.
- **Observability**: logs (chuyện gì xảy ra) + metrics (bao nhiêu) + traces (ở đâu trong chuỗi gọi). Distributed tracing (OpenTelemetry) không phải "nice to have" — thiếu nó, debug một request chậm qua 15 service là bất khả thi.
- **Security**: bề mặt tấn công tăng theo số liên kết mạng. mTLS giữa service, secret rotation, principle of least privilege.

## 7. Best Practices

1. **Bắt đầu bằng monolith có kỷ luật** (modular monolith). Tách service khi có *bằng chứng* (đo được) về nhu cầu scale/tổ chức, không tách theo trend.
2. **Thiết kế cho failure từ ngày đầu**: mọi remote call phải có timeout; mọi retry phải có giới hạn và jitter; mọi write qua mạng phải idempotent.
3. **Đo trước khi tối ưu**: bottleneck thật hầu như luôn khác bottleneck đoán.
4. **Giảm MTTR quan trọng hơn tăng MTBF**: đầu tư vào automation failover, runbook, game day.
5. **Redundancy phải độc lập**: 2 replica cùng rack là 1 replica với extra steps.

## 8. Anti-patterns

- **Distributed Monolith**: tách monolith thành 20 service nhưng service nào cũng gọi đồng bộ service khác và deploy phải theo lockstep. Nhận đủ độ phức tạp của phân tán, không nhận được lợi ích nào. Dấu hiệu: không thể deploy service A mà không deploy B; một request đi qua >5 hop đồng bộ.
- **Premature Distribution**: microservices cho startup 5 người, 100 users. Chi phí phối hợp giết tốc độ phát triển.
- **Shared Database giữa các service**: hai service ghi chung một bảng → coupling qua schema, không thể evolve độc lập — tệ hơn monolith vì thêm network mà không thêm isolation.
- **Bỏ qua tail latency**: tối ưu p50 trong khi p99 = 10 giây.

## 9. Khi nào KHÔNG nên phân tán

- Dữ liệu < vài trăm GB, tải < 10K rps: **một máy PostgreSQL + replica standby** thắng gần như mọi kiến trúc phân tán về mọi mặt trừ "độ ngầu".
- Team < 10 engineer: chi phí vận hành hệ phân tán ăn hết năng lực phát triển tính năng.
- Nghiệp vụ cần transaction mạnh trên toàn bộ dữ liệu (core banking ledger nhỏ): một node + sync replica đơn giản và *đúng* hơn.
- Latency nội bộ là yếu tố sống còn (trading engine): mọi network hop là kẻ thù — các sàn giao dịch chạy matching engine trên **một** máy, single-threaded, có bản sao hot-standby.

Quy tắc: **phân tán là phương tiện bất đắc dĩ, không phải mục tiêu.**

## 10. Troubleshooting nhập môn

Khi hệ phân tán "có vấn đề", quy trình tư duy:

1. **Xác định blast radius**: một node, một AZ, một service, hay toàn bộ? (dashboard theo dimension)
2. **Cái gì thay đổi?**: 80% sự cố đến từ deploy/config change trong 24h gần nhất. Kiểm tra change log trước khi kiểm tra bất cứ thứ gì khác.
3. **Phân biệt nguyên nhân và triệu chứng lan truyền**: service A lỗi có thể vì B chậm, B chậm vì C hết connection pool, C hết pool vì D bị hot partition. Trace từ *dưới* lên.
4. **Nghi ngờ mạng cuối cùng nhưng đừng loại trừ**: "It's not DNS. There's no way it's DNS. It was DNS."

Các failure mode kinh điển (chi tiết ở các chương sau): Network Partition (ch. 04, 07), Split Brain (ch. 07), Clock Drift (ch. 12), Message Duplication (ch. 08), Thundering Herd & Cache Stampede (ch. 10, 11), Hot Partition (ch. 06).

---

## Tổng kết chương

### Những điều bắt buộc phải nhớ
1. Phân tán là **bị ép buộc** (giới hạn vật lý, SPOF, địa lý), không phải lựa chọn phong cách.
2. Timeout không cho biết chuyện gì đã xảy ra — chỉ cho biết bạn đã chờ bao lâu. Mọi thiết kế đúng đều xuất phát từ sự thật này.
3. Không có đồng hồ toàn cục, không có shared memory, partial failure là thường trực.
4. Availability = MTBF/(MTBF+MTTR); giảm MTTR thường rẻ hơn tăng MTBF.
5. Chuỗi phụ thuộc đồng bộ nhân availability lại với nhau; redundancy độc lập mới mua được số 9.
6. Scale Up trước, Scale Out khi có bằng chứng.

### Hiểu lầm phổ biến
- "Microservices = scalable." Sai — USL cho thấy phối hợp có thể làm thêm node giảm throughput.
- "Cloud tự lo availability." Cloud lo *hạ tầng*; kiến trúc ứng dụng sai vẫn sập như thường (một region outage của AWS kéo sập hàng nghìn công ty đặt hết trứng vào us-east-1).
- "Mạng nội bộ DC thì đáng tin." Packet loss, GC pause, VM migration xảy ra hằng ngày ở quy mô lớn.

### Câu hỏi tự kiểm tra
1. Node A gửi lệnh trừ tiền cho node B, timeout. Liệt kê 4 khả năng đã xảy ra và thiết kế của bạn xử lý ra sao?
2. Hệ của bạn gồm 25 service, mỗi service 99.9%, một request đi qua 8 service. Availability tổng? Làm sao cải thiện mà không tăng SLA từng service?
3. Vì sao thêm read replica không giải quyết nghẽn write? Cái gì giải quyết?
4. Khi nào bạn khuyên một công ty *quay lại* monolith?

### Tài liệu kinh điển nên đọc
- **"Fallacies of Distributed Computing Explained" (Rotem-Gal-Oz)** — 8 ngộ nhận, nền của mọi thiết kế đúng.
- **"The Google File System" (2003)** — lần đầu một công ty nói to: "phần cứng hỏng là chuyện thường, thiết kế quanh nó" — thay đổi cả ngành.
- **"Site Reliability Engineering" (Google, sách miễn phí)** — chương SLO/Error Budget: cách nghĩ đúng về availability như một ngân sách chi tiêu.
- **"Notes on Distributed Systems for Young Bloods" (Jeff Hodges)** — bài blog ngắn, đậm đặc kinh nghiệm production.
- **Universal Scalability Law (Neil Gunther)** — mô hình toán giải thích vì sao thêm máy không luôn nhanh hơn.
