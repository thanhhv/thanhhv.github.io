+++
title = "Phần 13 — Production Failure Cases"
date = "2026-07-13T16:10:00+07:00"
draft = false
tags = ["backend", "system-design"]
series = ["System Design — Tư Duy Thiết Kế Hệ Thống"]
+++

> 21 tình huống sự cố production kinh điển. Mục tiêu không phải ghi nhớ từng ca — mà là nhận ra **các mô hình lặp lại** để khi gặp biến thể thứ 22, bạn đã có phản xạ đúng.

## Cấu trúc phân tích mỗi tình huống

Mỗi failure case được phân tích theo khung 10 điểm:

1. **Triệu chứng** — những gì dashboard/user cho thấy
2. **Root cause** — nguyên nhân gốc
3. **Tại sao xảy ra** — cơ chế vật lý/logic đằng sau
4. **Kiến trúc nào bị ảnh hưởng**
5. **Metric cần theo dõi**
6. **Dashboard** — nhìn gì
7. **Alert** — ngưỡng nào
8. **Quy trình điều tra** — làm gì, theo thứ tự nào
9. **Cách khắc phục** — cầm máu ngay + chữa gốc
10. **Cách phòng tránh**

## Danh mục

| Nhóm | File | Các case |
|---|---|---|
| Caching | [13.1](/series/system-design/13-production-failure-cases/01-caching-failures/) | Cache Stampede · Cache Avalanche · Thundering Herd |
| Database | [13.2](/series/system-design/13-production-failure-cases/02-database-failures/) | Database Hotspot · N+1 Query · Deadlock · Replica Lag · Connection Pool Exhaustion · Hot Partition |
| Messaging | [13.3](/series/system-design/13-production-failure-cases/03-messaging-failures/) | Kafka Lag · Message Duplication · Queue Backlog · Retry Storm |
| Distributed | [13.4](/series/system-design/13-production-failure-cases/04-distributed-failures/) | Cascading Failure · Split Brain · Leader Election Failure |
| Infrastructure | [13.5](/series/system-design/13-production-failure-cases/05-infrastructure-failures/) | GC Pause · Out of Memory · DNS Failure · Region Outage · Third-party API Down |

## Bốn mô hình gốc đằng sau 21 tình huống

Đọc xong cả phần, bạn sẽ thấy hầu hết sự cố quy về bốn cơ chế:

**1. Đồng bộ hóa ngẫu nhiên (synchronized demand).** Nhiều tác nhân độc lập bỗng hành động cùng lúc — cache cùng hết hạn, retry cùng nhịp, cron cùng phút, connection cùng reconnect sau sự cố. Đám đông tự phát này tạo spike mà không tầng nào được thiết kế để chịu. *Thuốc chung: jitter (ngẫu nhiên hóa), phân tán thời điểm, single-flight.*

**2. Khuếch đại (amplification).** Một đơn vị lỗi/chậm sinh ra nhiều đơn vị tải: 1 miss → N query; 1 timeout → 3 retry; 1 service chậm → giam N thread của mọi upstream. Hệ số khuếch đại > 1 + vòng lặp = bùng nổ. *Thuốc chung: giới hạn hệ số (retry budget, coalescing), cắt vòng lặp (circuit breaker).*

**3. Cạn tài nguyên giới hạn (pool exhaustion).** Connection, thread, memory, file descriptor — tài nguyên có trần bị giữ lâu hơn dự kiến bởi một thứ chậm ở nơi khác. Triệu chứng hiện ra xa nơi nguyên nhân đứng. *Thuốc chung: timeout mọi nơi, bulkhead, đo saturation chứ không chỉ utilization.*

**4. Phát hiện lỗi sai trong hệ phân tán.** Timeout bị diễn giải thành "chết", trong khi thực tế là "chậm" — dẫn đến hai leader, failover oan, hành động trùng lặp. *Thuốc chung: quorum, lease, fencing token ([Phần 4](/series/system-design/04-distributed-systems/03-consensus-quorum-leader-election/)).*

## Nguyên tắc điều tra sự cố (áp dụng cho mọi case)

1. **Cầm máu trước, hiểu sau.** Mục tiêu đầu tiên là khôi phục dịch vụ (rollback, scale, shed load, bật degraded mode) — root cause phân tích sau khi user hết đau.
2. **"Cái gì vừa thay đổi?"** — deploy, config, migration, certificate, dữ liệu tăng qua ngưỡng, dependency bên ngoài. 80% sự cố có câu trả lời ở đây.
3. **Nhìn theo chuỗi phụ thuộc từ user vào trong** — tracing trước, đoán sau ([chương 1.5](/series/system-design/01-foundations/05-bottleneck-analysis/)).
4. **Ghi lại timeline trong lúc xử lý** (ai làm gì lúc nào) — postmortem viết từ trí nhớ là tiểu thuyết.
5. **Postmortem không đổ lỗi, có action item, có deadline.** Sự cố lặp lại y nguyên là lỗi của quy trình, không phải của người trực.
