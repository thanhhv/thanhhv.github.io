+++
title = "System Design — Mục Lục & Cách Đọc Tài Liệu"
date = "2026-07-13T05:00:00+07:00"
draft = false
tags = ["backend", "system-design"]
series = ["System Design — Tư Duy Thiết Kế Hệ Thống"]
+++

> Tài liệu chuyên sâu về System Design dành cho Backend Engineer, Senior Backend Engineer, Tech Lead, Solution Architect và Software Architect.
>
> Mục tiêu: xây dựng **tư duy thiết kế hệ thống** — không phải bộ sưu tập lời giải phỏng vấn.

---

## Triết lý của tài liệu

Mọi quyết định kiến trúc trong tài liệu này đều đi theo chuỗi tư duy:

```
Business Requirement
        ↓
Functional Requirement
        ↓
Non-functional Requirement
        ↓
Scale Estimation
        ↓
Constraint
        ↓
Bottleneck
        ↓
Architecture Pattern
        ↓
Distributed Systems
        ↓
Trade-off
        ↓
Production
        ↓
Evolution
```

Mọi kết luận kỹ thuật phải trả lời được 5 câu hỏi:

1. **Tại sao?**
2. **Nếu không làm như vậy thì sao?**
3. **Trade-off là gì?**
4. **Có lựa chọn nào khác không?**
5. **Chi phí vận hành là gì?**

Không có "kiến trúc tối ưu". Chỉ có kiến trúc **phù hợp với bài toán, quy mô, ngân sách và đội ngũ tại một thời điểm**.

---

## Mục lục

### Chương mở đầu

- [00. Tư duy thiết kế hệ thống](/series/system-design/00-tu-duy-thiet-ke/) — Framework tư duy xuyên suốt toàn bộ tài liệu. **Đọc trước tiên.**

### Phần 1 — Foundations *(hoàn chỉnh)*

- [1.1. Functional & Non-functional Requirements](/series/system-design/01-foundations/01-requirements/)
- [1.2. SLA, SLO, SLI](/series/system-design/01-foundations/02-sla-slo-sli/)
- [1.3. Throughput & Latency](/series/system-design/01-foundations/03-throughput-latency/)
- [1.4. Scale Estimation & Capacity Planning](/series/system-design/01-foundations/04-scale-estimation-capacity-planning/)
- [1.5. Bottleneck Analysis](/series/system-design/01-foundations/05-bottleneck-analysis/)

### Phần 2 — Scalability *(hoàn chỉnh)*

- [Tổng quan & ba mệnh đề](/series/system-design/02-scalability/00-tong-quan/)
- [2.1. Vertical vs Horizontal Scaling — và bài toán state](/series/system-design/02-scalability/01-vertical-horizontal-scaling/)
- [2.2. Load Balancer — người gác cổng của scale-out](/series/system-design/02-scalability/02-load-balancer/)
- [2.3. Auto Scaling — co giãn theo tải](/series/system-design/02-scalability/03-auto-scaling/)

### Phần 3 — Availability & Reliability *(hoàn chỉnh)*

- [Tổng quan & ba mệnh đề nền](/series/system-design/03-availability-reliability/00-tong-quan/)
- [3.1. High Availability & Failover — giải phẫu quá trình chuyển đổi](/series/system-design/03-availability-reliability/01-ha-failover/)
- [3.2. Backup & Recovery — lớp phòng thủ cuối cùng](/series/system-design/03-availability-reliability/02-backup-recovery/)
- [3.3. Active-Active vs Active-Passive — hai triết lý redundancy](/series/system-design/03-availability-reliability/03-active-active-passive/)

### Phần 4 — Distributed Systems *(hoàn chỉnh)*

- [4.1. CAP Theorem & PACELC](/series/system-design/04-distributed-systems/01-cap-pacelc/)
- [4.2. Replication & Consistency Models](/series/system-design/04-distributed-systems/02-replication-consistency/)
- [4.3. Consensus, Quorum & Leader Election](/series/system-design/04-distributed-systems/03-consensus-quorum-leader-election/)
- [4.4. Clock Synchronization, Network Partition & Split Brain](/series/system-design/04-distributed-systems/04-clock-partition-split-brain/)

### Phần 5 — Data Layer *(hoàn chỉnh)*

- [Tổng quan & nguyên tắc polyglot](/series/system-design/05-data-layer/00-tong-quan/)
- [5.1. PostgreSQL — mặc định đúng cho dữ liệu nghiệp vụ](/series/system-design/05-data-layer/01-postgresql/)
- [5.2. MySQL — người anh em song sinh khác tính cách](/series/system-design/05-data-layer/02-mysql/)
- [5.3. MongoDB — khi dữ liệu thật sự là document](/series/system-design/05-data-layer/03-mongodb/)
- [5.4. Redis — cấu trúc dữ liệu trong RAM](/series/system-design/05-data-layer/04-redis/)
- [5.5. ClickHouse — cỗ máy quét tỷ hàng](/series/system-design/05-data-layer/05-clickhouse/)
- [5.6. Elasticsearch — index, không phải database](/series/system-design/05-data-layer/06-elasticsearch/)
- [5.7. So sánh & khung quyết định lựa chọn](/series/system-design/05-data-layer/07-so-sanh-lua-chon/)

### Phần 6 — Communication *(hoàn chỉnh)*

- [Tổng quan & bốn kỷ luật cho mọi cạnh giao tiếp](/series/system-design/06-communication/00-tong-quan/)
- [6.1. REST — hợp đồng chung của web](/series/system-design/06-communication/01-rest/)
- [6.2. GraphQL — client tự khai hình dữ liệu](/series/system-design/06-communication/02-graphql/)
- [6.3. gRPC — RPC có kỷ luật cho nội bộ](/series/system-design/06-communication/03-grpc/)
- [6.4. RabbitMQ — smart broker cho work queue](/series/system-design/06-communication/04-rabbitmq/)
- [6.5. Kafka — distributed log cho sự kiện](/series/system-design/06-communication/05-kafka/)
- [6.6. Event-driven Architecture — nghĩ bằng sự kiện](/series/system-design/06-communication/06-event-driven/)
- [6.7. Saga — transaction khi không còn transaction](/series/system-design/06-communication/07-saga/)
- [6.8. Outbox Pattern — móng của mọi event đáng tin](/series/system-design/06-communication/08-outbox/)

### Phần 7 — Caching *(hoàn chỉnh)*

- [Tổng quan & nguyên tắc bất di bất dịch](/series/system-design/07-caching/00-tong-quan/)
- [7.1. Bốn chiến lược cache — Cache Aside, Read Through, Write Through, Write Back](/series/system-design/07-caching/01-cache-strategies/)
- [7.2. Cache Invalidation — bài toán khó thứ nhất](/series/system-design/07-caching/02-cache-invalidation/)
- [7.3. Distributed Cache — cache khi một node không đủ](/series/system-design/07-caching/03-distributed-cache/)

### Phần 8 — Data Partitioning *(hoàn chỉnh)*

- [Tổng quan: ba khái niệm hay lẫn](/series/system-design/08-data-partitioning/00-tong-quan/)
- [8.1. Partitioning & Sharding — cái giá của shard key](/series/system-design/08-data-partitioning/01-partitioning-sharding/)
- [8.2. Consistent Hashing](/series/system-design/08-data-partitioning/02-consistent-hashing/)
- [8.3. Resharding & vận hành hệ đã shard](/series/system-design/08-data-partitioning/03-resharding-van-hanh/)

### Phần 9 — Search *(hoàn chỉnh)*

- [Tổng quan & phân vai với Phần 5](/series/system-design/09-search/00-tong-quan/)
- [9.1. Full-text Search — inverted index, analyzer, relevance](/series/system-design/09-search/01-full-text-search/)
- [9.2. Kiến trúc hệ search hoàn chỉnh — pipeline, query side, vận hành](/series/system-design/09-search/02-search-architecture/)
- [9.3. Lựa chọn công nghệ — PG FTS, Elasticsearch, OpenSearch, engine gọn](/series/system-design/09-search/03-lua-chon-cong-nghe/)

### Phần 10 — Observability *(hoàn chỉnh)*

- [Tổng quan & ba mệnh đề](/series/system-design/10-observability/00-tong-quan/)
- [10.1. Ba trụ — Logging, Metrics, Tracing](/series/system-design/10-observability/01-ba-tru/)
- [10.2. OpenTelemetry & pipeline tín hiệu](/series/system-design/10-observability/02-opentelemetry-pipeline/)
- [10.3. Dashboard, Alerting & On-call](/series/system-design/10-observability/03-dashboard-alerting-oncall/)

### Phần 11 — Security *(hoàn chỉnh)*

- [Tổng quan & ba nguyên tắc gốc](/series/system-design/11-security/00-tong-quan/)
- [11.1. Authentication & Authorization](/series/system-design/11-security/01-authn-authz/)
- [11.2. OAuth2, OIDC & JWT](/series/system-design/11-security/02-oauth2-jwt/)
- [11.3. Biên phòng thủ — API Gateway, Rate Limiting, WAF](/series/system-design/11-security/03-gateway-ratelimit-waf/)

### Phần 12 — System Design Evolution *(hoàn chỉnh — chương quan trọng nhất)*

Hành trình tiến hóa của một hệ thống E-commerce qua 10 giai đoạn, từ 0 đến hàng chục triệu người dùng:

- [Tổng quan hành trình tiến hóa](/series/system-design/12-evolution/00-tong-quan/)
- [Giai đoạn 1 — Monolith + PostgreSQL](/series/system-design/12-evolution/01-monolith-postgresql/)
- [Giai đoạn 2 — Thêm Redis](/series/system-design/12-evolution/02-them-redis/)
- [Giai đoạn 3 — Tách Background Worker](/series/system-design/12-evolution/03-background-worker/)
- [Giai đoạn 4 — Thêm Message Queue](/series/system-design/12-evolution/04-message-queue/)
- [Giai đoạn 5 — Modular Monolith](/series/system-design/12-evolution/05-modular-monolith/)
- [Giai đoạn 6 — Tách Microservices](/series/system-design/12-evolution/06-microservices/)
- [Giai đoạn 7 — Kafka & Event-driven](/series/system-design/12-evolution/07-kafka-event-driven/)
- [Giai đoạn 8 — CQRS](/series/system-design/12-evolution/08-cqrs/)
- [Giai đoạn 9 — Multi-region](/series/system-design/12-evolution/09-multi-region/)
- [Giai đoạn 10 — Disaster Recovery](/series/system-design/12-evolution/10-disaster-recovery/)

### Phần 13 — Production Failure Cases *(hoàn chỉnh)*

21 tình huống sự cố production, mỗi tình huống phân tích: triệu chứng → root cause → metric → dashboard → alert → điều tra → khắc phục → phòng tránh.

- [Tổng quan & phương pháp phân tích sự cố](/series/system-design/13-production-failure-cases/00-tong-quan/)
- [13.1. Caching Failures — Cache Stampede, Cache Avalanche, Thundering Herd](/series/system-design/13-production-failure-cases/01-caching-failures/)
- [13.2. Database Failures — Hotspot, N+1 Query, Deadlock, Replica Lag, Connection Pool Exhaustion, Hot Partition](/series/system-design/13-production-failure-cases/02-database-failures/)
- [13.3. Messaging Failures — Kafka Lag, Message Duplication, Queue Backlog, Retry Storm](/series/system-design/13-production-failure-cases/03-messaging-failures/)
- [13.4. Distributed Failures — Cascading Failure, Split Brain, Leader Election Failure](/series/system-design/13-production-failure-cases/04-distributed-failures/)
- [13.5. Infrastructure Failures — GC Pause, Out of Memory, DNS Failure, Region Outage, Third-party API Down](/series/system-design/13-production-failure-cases/05-infrastructure-failures/)

### Phần 14 — Case Studies *(hoàn chỉnh)*

- [Tổng quan & cách đọc](/series/system-design/14-case-studies/00-tong-quan/)
- [14.1. URL Shortener — bài tập khởi động hoàn hảo](/series/system-design/14-case-studies/01-url-shortener/)
- [14.2. Social Network — fan-out và celebrity problem](/series/system-design/14-case-studies/02-social-network/)
- [14.3. Chat Application — triệu kết nối sống](/series/system-design/14-case-studies/03-chat-application/)
- [14.4. Notification System — fan-out đa kênh](/series/system-design/14-case-studies/04-notification-system/)
- [14.5. Banking & FinTech — khi sai một đồng là sai tất cả](/series/system-design/14-case-studies/05-banking-fintech/)
- [14.6. Video Streaming — băng thông là kiến trúc](/series/system-design/14-case-studies/06-video-streaming/)
- [14.7. Ride Hailing — geo real-time và dữ liệu phù du](/series/system-design/14-case-studies/07-ride-hailing/)
- [14.8. SaaS Platform — multi-tenancy và noisy neighbor](/series/system-design/14-case-studies/08-saas-platform/)
- [14.9. AI Platform — GPU đắt và hai chế độ phục vụ](/series/system-design/14-case-studies/09-ai-platform/)
- [14.10. Search System — Phần 9 trong hành động](/series/system-design/14-case-studies/10-search-system/)

---

## Cách đọc tài liệu này

**Nếu bạn là Backend Engineer (2–4 năm kinh nghiệm):** đọc tuần tự 00 → Phần 1 → Phần 12. Phần 12 là nơi mọi khái niệm được đặt vào bối cảnh thực tế.

**Nếu bạn là Senior/Tech Lead:** đọc 00 để thống nhất framework tư duy, sau đó đi thẳng vào Phần 4 (Distributed Systems) và Phần 13 (Failure Cases) — đây là hai phần phân tách một Senior với một Architect.

**Nếu bạn là Architect:** dùng Phần 12 và 13 làm tài liệu training cho team, dùng template trong 00 làm chuẩn cho Architecture Decision Record (ADR) nội bộ.

**Nguyên tắc quan trọng nhất khi đọc:** đừng ghi nhớ giải pháp. Hãy ghi nhớ **câu hỏi** dẫn đến giải pháp. Giải pháp thay đổi theo thời gian; câu hỏi thì không.

---

## Template phân tích bắt buộc

Mỗi chủ đề trong tài liệu tuân theo template 9 phần:

| # | Phần | Câu hỏi trung tâm |
|---|------|-------------------|
| 1 | Problem Statement | Bài toán kinh doanh là gì? Ràng buộc là gì? |
| 2 | Tại sao giải pháp này tồn tại | Nó giải quyết vấn đề business/technical/scale/reliability nào? |
| 3 | First Principles | Bản chất là gì? Bỏ đi thì sao? Giả định nào đang được đặt ra? |
| 4 | Internal Architecture | Component, Data Flow, Control Flow, Deployment Flow, Failure Flow |
| 5 | Trade-off | Được gì, mất gì, chi phí, rủi ro |
| 6 | Production Considerations | Monitoring, Deployment, DR, Security, Cost |
| 7 | Best Practices | Cách dùng đúng trong production |
| 8 | Anti-patterns | Cách dùng sai và vì sao nguy hiểm |
| 9 | Khi nào KHÔNG nên dùng | Bối cảnh mà giải pháp này là lãng phí |

---

*Diagram trong tài liệu dùng [Mermaid](https://mermaid.js.org/) — render trực tiếp trên GitHub, VS Code (extension Markdown Preview Mermaid), Obsidian.*
