+++
title = "Bài 9 — Reliability Engineering"
date = "2026-02-01T15:00:00+07:00"
draft = false
tags = ["backend", "infrastructure", "networking"]
series = ["Infrastructure & Networking"]
+++

# Module 09 — Reliability Engineering

> Tiên đề: **mọi thứ đều hỏng**. Disk hỏng, mạng phân mảnh, process bị OOM, con người gõ nhầm lệnh, và một ngày nào đó cả region sập. Reliability engineering không phải ngăn hỏng hóc — đó là thiết kế để **hệ thống vẫn phục vụ được user khi từng phần của nó đang hỏng**.

---

## 1. High Availability và Fault Tolerance

### Toán học nền tảng
Availability chuỗi nối tiếp NHÂN với nhau: 5 service 99.9% nối tiếp = 99.5% (43 phút → 3.6 giờ downtime/tháng). Hệ quả khắc nghiệt: **chuỗi phụ thuộc càng dài, availability càng tệ hơn mắt xích tệ nhất**, và mọi dependency đồng bộ trong critical path đều phải "trả thuế" này. Ngược lại, redundancy song song: 2 bản 99% độc lập = 99.99% — **nếu thật sự độc lập**.

"Nếu thật sự độc lập" là toàn bộ độ khó của nghề. Các failure mode tương quan phá vỡ phép nhân đẹp đẽ: 2 replica trên cùng node (module 05 — topologySpread tồn tại vì thế), cùng AZ, chung một bug deploy cùng lúc, chung một certificate hết hạn, chung một config service. **Phần lớn outage lớn trong ngành là correlated failure, không phải hỏng hóc đơn lẻ.**

### Redundancy có ích chỉ khi có 3 thứ đi kèm
1. **Phát hiện** nhanh (health check — với mọi cạm bẫy ở module 02).
2. **Failover tự động** (con người mất 15–30 phút; và failover thủ công lúc 3h sáng có xác suất lỗi thao tác cao).
3. **Capacity thật** ở bản dự phòng: chạy 2 node ở 80% tải = không có failover, chỉ có domino (node còn lại nhận 160%). N+1 nghĩa là N node ĐỦ tải, cộng 1.

Với stateful, thêm bài toán consensus: failover DB tự động cần phát hiện leader chết **chính xác** (phân biệt "chết" với "mạng chậm" là bất khả thi tuyệt đối — FLP), nên cần quorum + fencing chống **split-brain** (2 leader cùng ghi = hỏng dữ liệu, tệ hơn downtime). Đây là phần nên mua (managed DB, etcd, Patroni đã kiểm chứng) thay vì tự viết.

---

## 2. Disaster Recovery, RTO và RPO

### Hai con số quyết định mọi thứ
- **RPO** (Recovery Point Objective): mất tối đa bao nhiêu **dữ liệu** (tính bằng thời gian)? RPO 1 giờ = chấp nhận mất 1 giờ giao dịch cuối.
- **RTO** (Recovery Time Objective): mất tối đa bao lâu để **chạy lại**?

Cả hai là **quyết định kinh doanh** (mỗi bậc giảm là một bậc nhân chi phí — bảng ở module 07 mục multi-region), và khác nhau **theo từng hệ**: ledger thanh toán RPO~0, log phân tích RPO 24h là thừa. Đặt một RPO/RTO chung cho cả công ty = trả tiền mức cao nhất cho mọi thứ.

### Sự thật về backup — nơi ảo tưởng an toàn tập trung
1. **Backup chưa restore thử = không có backup.** Nỗi đau kinh điển: ngày cần restore mới phát hiện backup rỗng/hỏng/thiếu quyền/mất 26 giờ cho RTO 4 giờ. Diễn tập restore định kỳ, đo thời gian, tự động hóa việc verify.
2. Backup phải **cách ly khỏi blast radius của chính hệ thống**: khác account/region, immutable (object lock), offline copy cho kịch bản ransomware — kẻ mã hóa dữ liệu luôn tìm xóa backup trước. Quy tắc 3-2-1 vẫn đúng.
3. Replication KHÔNG phải backup: replica trung thành sao chép cả lệnh `DROP TABLE` trong mili giây. Replication chống hỏng phần cứng; backup (+ point-in-time recovery) chống lỗi con người và phần mềm — cần **cả hai**.
4. DR runbook mục nát theo tốc độ thay đổi hệ thống → game day (diễn tập DR thật) mỗi quý cho hệ quan trọng.

---

## 3. Các pattern cách ly và chịu lỗi

(Timeout, Retry + backoff + jitter + budget, Circuit Breaker — đã phân tích sâu ở module 02. Ở đây là các pattern cấp kiến trúc.)

### Bulkhead — cách ly để lỗi không lây
Đặt theo vách ngăn tàu thủy: **chia tài nguyên thành ngăn để một ngăn thủng không chìm cả tàu**. Hình thức: pool connection/thread riêng cho từng dependency (dependency chậm chỉ cạn pool CỦA NÓ — không kéo chết request không liên quan; thiếu bulkhead là cách một "service phụ đề xuất sản phẩm" giết trang thanh toán); node pool riêng cho workload ồn ào; cell-based architecture (chia user thành cell độc lập — blast radius = 1/N user; cách các hệ lớn như AWS tự tổ chức).

### Graceful Degradation — thiết kế "hỏng còn một nửa"
Câu hỏi thiết kế bắt buộc cho mọi dependency: "**nó chết 10 phút thì user thấy gì?**" Ba câu trả lời tốt theo thứ tự: (1) user không nhận ra (serve cache cũ — stale tốt hơn lỗi với đa số dữ liệu đọc); (2) mất tính năng phụ, lõi vẫn chạy (recommendation chết → hiện bestseller tĩnh; checkout vẫn chạy); (3) hàng đợi + xử lý sau (ghi nhận đơn, xử lý khi hồi phục). Câu trả lời tệ duy nhất: 500 toàn trang. Degradation phải được **code và test trước** — không ai viết được fallback tử tế giữa sự cố.

**Load shedding** cùng họ: khi quá tải, chủ động từ chối sớm một phần (429, ưu tiên request giá trị cao, drop việc rẻ) tốt hơn nhận tất cả rồi chậm đến chết tất cả — hệ quá tải mà không shed sẽ rơi vào **metastable state**: chậm → timeout phía client → retry → tải tăng → chậm hơn, và **không tự thoát được kể cả khi nguyên nhân gốc đã hết** (phải shed/tắt bớt traffic để thoát). Nhận diện trạng thái này tiết kiệm hàng giờ điều tra: "đã sửa nguyên nhân mà hệ vẫn cháy" = metastable, hãy giảm tải cưỡng bức.

### Backpressure
Hàng đợi không giới hạn = bom hẹn giờ: che quá tải cho đến khi OOM hoặc latency hàng đợi lên hàng phút (việc lấy ra đã vô nghĩa). Mọi hàng đợi cần: bound + hành vi khi đầy (reject/drop-oldest) + metric độ sâu (saturation — báo trước mọi thứ khác, module 08).

---

## 4. Chaos Engineering

### Vì sao tồn tại
Mọi cơ chế resilience (failover, fallback, retry, DR) đều là **code chưa chạy trong điều kiện thật cho đến ngày sự cố** — tức code chưa test. Chaos engineering = test chúng có chủ đích: tiêm lỗi (giết pod, chặn mạng, tăng latency, đánh sập AZ) trong điều kiện kiểm soát để tìm điểm gãy TRƯỚC khi production tìm hộ lúc 3h sáng.

### Làm cho đúng
Đây là **thí nghiệm khoa học, không phải phá phách**: (1) phát biểu giả thuyết ("giết 1 pod payment, error rate không đổi vì retry + LB"); (2) blast radius tối thiểu + **nút dừng khẩn**; (3) bắt đầu ở staging, chỉ lên production khi đã trưởng thành (và production mới là bài thật — staging không có traffic thật); (4) kết quả bất ngờ = kho báu (giả thuyết sai chính là bug tiềm ẩn được tìm thấy miễn phí); (5) sau đó tự động hóa thành continuous verification (kiểu Chaos Monkey — pod bị giết ngẫu nhiên hằng ngày buộc mọi team viết code chịu chết từ đầu).

Điều kiện tiên quyết: observability tốt (không đo được thì không phải thí nghiệm — là phá hoại thuần túy) và văn hóa blameless.

---

## 5. Văn hóa vận hành — phần quyết định

- **Blameless postmortem**: sự cố = tín hiệu hệ thống (kỹ thuật + quy trình) có lỗ hổng, không phải "ai ngu". Trừng phạt người gây sự cố → lần sau che giấu → tổ chức mù. Postmortem tốt: timeline khách quan, root cause đa tầng (5 whys thường ra: thiếu guard-rail chứ không phải thiếu cẩn thận), action item có owner + deadline và **được làm thật** (postmortem không action = nghi lễ).
- **Error budget** (module 08) là khớp nối giữa reliability và tốc độ release.
- **Giảm toil**: việc tay lặp lại tăng tuyến tính theo scale → tự động hóa hoặc chết chìm. SRE chuẩn Google: toil < 50% thời gian.
- On-call bền vững: rotation đủ người, alert sạch (module 08), quyền sửa tận gốc thứ đánh thức mình.

## Checklist reliability cho một service mới
- [ ] Trả lời "chết 10 phút thì user thấy gì?" cho TỪNG dependency; fallback đã code + test
- [ ] Replica rải qua node/AZ; capacity N+1 thật; PDB
- [ ] Timeout/retry/breaker theo chuẩn module 02; bulkhead cho dependency
- [ ] Queue có bound + shed; RPO/RTO khai báo và backup ĐÃ diễn tập restore
- [ ] SLO + burn-rate alert; runbook; đã qua ít nhất một game day
