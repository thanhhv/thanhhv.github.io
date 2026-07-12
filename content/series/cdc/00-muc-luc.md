+++
title = "Change Data Capture và Đồng bộ dữ liệu trong hệ thống Backend hiện đại"
date = "2026-02-20T07:00:00+07:00"
draft = false
tags = ["backend", "cdc", "kafka", "database"]
series = ["Change Data Capture"]
+++

Bộ tài liệu chuyên sâu về Change Data Capture (CDC) — viết từ góc nhìn của người thiết kế Data Platform và Distributed Systems, không phải từ góc nhìn người dùng công cụ.

**Đối tượng:** Backend Engineer, Data Engineer, Senior Backend Engineer, Tech Lead, Solution Architect, Software Architect.

**Triết lý trình bày:** Business Problem → Vì sao cần đồng bộ dữ liệu → Hạn chế của Polling → Trigger / Dual Write / Event Publishing → CDC → Log-based CDC → Internal Architecture → Trade-off → Production → Khi nào không nên dùng CDC.

Tài liệu không hướng dẫn cài đặt Debezium hay Kafka Connect. Mục tiêu là sau khi đọc xong, bạn có thể thiết kế pipeline đồng bộ dữ liệu, lựa chọn đúng phương pháp synchronization, phân tích bottleneck, debug sự cố production và đưa ra quyết định kiến trúc phù hợp.

---

## Cấu trúc tài liệu

### Level 1 — Foundation

| Chương | Nội dung chính |
|---|---|
| [Chương 1: Bài toán đồng bộ dữ liệu](/series/cdc/01-bai-toan-dong-bo-du-lieu/) | Vì sao một database không phục vụ được mọi workload; Polling vs Push; Event vs Data Synchronization; Transaction Log và write-ahead principle; Eventual Consistency |
| [Chương 2: Các phương pháp đồng bộ truyền thống và hạn chế](/series/cdc/02-cac-phuong-phap-dong-bo-truyen-thong/) | Tiến hóa Batch ETL → Polling → Trigger-based → Dual Write → Event Publishing; vì sao Dual Write là anti-pattern nguy hiểm nhất; bảng so sánh tổng hợp |
| [Chương 3: Bản chất của Change Data Capture](/series/cdc/03-ban-chat-cdc/) | Insight cốt lõi: khai thác chính transaction log; Query-based vs Log-based CDC; Snapshot, Offset, Delivery Guarantee, Ordering, Tombstone; kiến trúc pipeline chuẩn |

### Level 2 — Engineering (Database Internals)

| Chương | Nội dung chính |
|---|---|
| [Chương 4: PostgreSQL Internals — WAL và Logical Replication](/series/cdc/04-postgresql-internals/) | WAL, LSN, Logical Decoding, REPLICA IDENTITY, Replication Slot và failure "slot giữ WAL làm đầy disk", WAL Retention, failover slot |
| [Chương 5: MySQL Binlog và MongoDB Oplog](/series/cdc/05-mysql-mongodb-internals/) | Binlog vs Redo Log, ROW format, GTID; Oplog, Replica Set, Change Streams, resume token; so sánh ba mô hình log |
| [Chương 6: Cơ chế CDC — Snapshot, Offset, Ordering và Delivery Guarantee](/series/cdc/06-co-che-cdc/) | Initial snapshot và bài toán consistency; thuật toán Incremental Snapshot (watermark/DBLog); offset và at-least-once; per-key ordering; cấu trúc change event |

### Level 3 — Senior (Debezium, Kafka Connect, Pipeline)

| Chương | Nội dung chính |
|---|---|
| [Chương 7: Debezium Internal Architecture](/series/cdc/07-debezium-internals/) | Connector lifecycle, snapshot mode, offset storage, heartbeat, signal table, schema history, SMT — và hậu quả khi cấu hình sai |
| [Chương 8: Kafka Connect — Nền tảng vận hành Connector](/series/cdc/08-kafka-connect/) | Worker/Connector/Task, distributed mode, ba internal topic, rebalance, error handling và DLQ, capacity planning |
| [Chương 9: Xây dựng Event Pipeline hoàn chỉnh](/series/cdc/09-event-pipeline/) | Topic design, Schema Registry và compatibility mode, ClickHouse (ReplacingMergeTree), Elasticsearch, Data Lake; latency budget và backpressure toàn tuyến |

### Level 4 — Principal (Kiến trúc, Production, Quyết định)

| Chương | Nội dung chính |
|---|---|
| [Chương 10: CDC trong bức tranh kiến trúc](/series/cdc/10-cdc-va-kien-truc/) | Outbox Pattern, Event Sourcing vs CDC, CQRS, Materialized View, Cache Synchronization, Audit Log, read-your-own-writes |
| [Chương 11: Production Failure Cases](/series/cdc/11-production-failure-cases/) | **Chương quan trọng nhất** — 20 sự cố production điển hình: triệu chứng, root cause, cách điều tra, metric, alert, khắc phục, phòng tránh |
| [Chương 12: Kiến trúc CDC thực tế theo domain](/series/cdc/12-kien-truc-thuc-te/) | E-commerce, FinTech, Social Network, SaaS multi-tenant, Data Warehouse, Real-time Analytics, Search, Audit Log, Cache Sync |
| [Chương 13: Best Practices, Anti-patterns và khi nào KHÔNG nên dùng CDC](/series/cdc/13-best-practices-anti-patterns/) | Best practices kèm lý do; 12 anti-pattern; ma trận quyết định polling / batch / outbox / CDC; checklist trước khi adopt |

---

## Lộ trình đọc gợi ý

**Backend Engineer mới tiếp cận CDC:** đọc tuần tự 1 → 2 → 3, sau đó 6, 7, rồi 13.

**Senior Engineer chuẩn bị triển khai:** 3 → 4 (hoặc 5 tùy database) → 6 → 7 → 8 → 9 → 11 → 13.

**Tech Lead / Architect đánh giá phương án:** 2 → 3 → 10 → 12 → 13, tham chiếu 11 khi lập kế hoạch vận hành.

**Người đang chữa cháy production:** vào thẳng [Chương 11](/series/cdc/11-production-failure-cases/).

## Quy ước

Tài liệu viết bằng tiếng Việt; thuật ngữ chuyên ngành (CDC, WAL, Binlog, Replication Slot, Offset, Snapshot, Connector, Outbox Pattern...) giữ nguyên tiếng Anh. Diagram dùng Mermaid — render được trên GitHub, GitLab, Obsidian và các công cụ hỗ trợ Mermaid. Số liệu benchmark là con số minh họa điển hình từ kinh nghiệm production, không phải kết quả đo chính thức — luôn benchmark trên hệ thống của bạn trước khi quyết định.
