+++
title = "Distributed Systems – Từ First Principles đến Production"
date = "2026-07-08T16:00:00+07:00"
draft = false
tags = ["backend", "distributed-systems"]
series = ["Distributed Systems"]
+++

> Bộ tài liệu chuyên sâu về Hệ thống Phân tán, viết cho Backend Engineer, Senior Engineer, Tech Lead và Architect.
> Triết lý xuyên suốt: **Business Growth → Single Machine → Single Point of Failure → Scalability Problem → Distributed Systems → New Problems → Trade-off → Solutions → Production.**

---

## Cách đọc bộ tài liệu này

Distributed Systems **không phải** là tập hợp các công nghệ như Kafka, Redis hay Kubernetes. Đó là một lĩnh vực nghiên cứu về cách nhiều máy tính phối hợp giải quyết một bài toán chung, trong khi luôn phải đối mặt với: độ trễ mạng không xác định, phần cứng hỏng bất kỳ lúc nào, phần mềm có bug, và sự thật khó chịu nhất — **không node nào biết chắc trạng thái thật của node khác**.

Mọi cơ chế trong tài liệu này (Replication, Consensus, Sharding, Distributed Lock...) đều được giải thích từ góc nhìn **người thiết kế hệ thống**, không phải người dùng thư viện. Trước mỗi giải pháp, hãy tự hỏi 4 câu:

1. **Tại sao** giải pháp này tồn tại?
2. Nếu **không** giải quyết thì điều gì xảy ra?
3. Chúng ta **đánh đổi** điều gì?
4. Tại sao **không có** giải pháp hoàn hảo?

## Mục lục

### Level 1 – Foundation
| Chương | Nội dung |
|---|---|
| [01 – Foundations](/series/distributed-system/01-foundations/) | Vì sao Distributed Systems tồn tại. Scalability, Availability, Reliability, Fault Tolerance. Scale Up vs Scale Out. Các vấn đề chỉ xuất hiện khi có nhiều node. |

### Level 2 – Engineering
| Chương | Nội dung |
|---|---|
| [02 – Communication](/series/distributed-system/02-communication/) | RPC, REST, gRPC, Message Queue, Event Streaming, Service Discovery, API Gateway. |
| [03 – Consistency Models](/series/distributed-system/03-consistency/) | Strong / Eventual / Causal Consistency, Read Your Writes, Monotonic Reads, Quorum. |
| [04 – CAP & PACELC](/series/distributed-system/04-cap-pacelc/) | CAP Theorem, PACELC, những hiểu lầm phổ biến, cách Spanner/DynamoDB "lách" CAP. |
| [05 – Replication](/series/distributed-system/05-replication/) | Leader-Follower, Multi-Leader, Leaderless, Sync vs Async Replication. |
| [06 – Partitioning](/series/distributed-system/06-partitioning/) | Sharding, Consistent Hashing, Range vs Hash Partition, Hot Partition, Rebalancing. |
| [07 – Consensus](/series/distributed-system/07-consensus/) | Consensus Problem, Raft, Paxos, Leader Election, Split Brain, Quorum, Failure Detection. |

### Level 3 – Senior Level
| Chương | Nội dung |
|---|---|
| [08 – Distributed Transactions](/series/distributed-system/08-distributed-transactions/) | 2PC, Saga, Outbox/Inbox Pattern, Idempotency, Compensation. |
| [09 – Event-driven Architecture](/series/distributed-system/09-event-driven/) | Event Bus, Event Streaming, CQRS, Event Sourcing, Domain/Integration Event. |
| [10 – Distributed Cache](/series/distributed-system/10-distributed-cache/) | Cache Aside, Read/Write Through, Write Behind, Stampede, Penetration, Avalanche, Breakdown. |
| [11 – Reliability Patterns](/series/distributed-system/11-reliability/) | Retry, Timeout, Circuit Breaker, Bulkhead, Backpressure, Rate Limiting, DLQ, Graceful Degradation. |
| [12 – Time trong hệ phân tán](/series/distributed-system/12-time/) | Clock Drift, NTP, Lamport Clock, Vector Clock, TrueTime. |

### Level 4 – Principal Level
| Chương | Nội dung |
|---|---|
| [13 – Multi-region & Disaster Recovery](/series/distributed-system/13-multi-region/) | Geo Replication, Active-Active vs Active-Passive, Data Locality, DR, Cost Optimization, Evolutionary Architecture. |
| [14 – Case Studies](/series/distributed-system/14-case-studies/) | URL Shortener, E-commerce, Banking/Payment, Chat, Notification, Ride Hailing, Social Feed, Search, Video Streaming, AI Platform. |

## Quy ước

- Thuật ngữ chuyên ngành (Consensus, Quorum, Leader Election, Replication...) giữ nguyên tiếng Anh.
- Diagram dùng ASCII và Mermaid (render tốt trên GitHub/GitLab/VS Code).
- Mỗi chương kết thúc bằng: **Những điều bắt buộc phải nhớ**, **Hiểu lầm phổ biến**, **Câu hỏi tự kiểm tra**, **Paper kinh điển nên đọc**.

## Bản đồ tư duy tổng thể

```
Business Growth (users tăng 10x)
        │
        ▼
Single Machine ──── giới hạn vật lý: CPU, RAM, Disk IO, Network
        │
        ▼
Single Point of Failure ──── 1 máy chết = toàn bộ hệ thống chết
        │
        ▼
Scalability Problem ──── Scale Up chạm trần → buộc phải Scale Out
        │
        ▼
Distributed Systems ──── nhiều node phối hợp
        │
        ▼
New Problems ──── network partition, partial failure, clock drift,
        │          không có shared memory, không có global "now"
        ▼
Trade-off ──── CAP/PACELC, latency vs consistency, cost vs reliability
        │
        ▼
Solutions ──── Replication, Partitioning, Consensus, Event-driven, Cache...
        │
        ▼
Production ──── monitoring, capacity planning, DR, cost, con người vận hành
```

Không có bước nào trong chuỗi trên là "miễn phí". Mỗi mũi tên đi xuống là một lần bạn đổi **một vấn đề đã hiểu rõ** lấy **một tập vấn đề mới phức tạp hơn** — với hy vọng rằng tập vấn đề mới, dù khó, là *giải được*, còn vấn đề cũ (giới hạn vật lý của một máy) là *không giải được*.
