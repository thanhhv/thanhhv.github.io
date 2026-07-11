+++
title = "Bài 0 — Giới Thiệu Series"
date = "2026-07-02T08:00:00+07:00"
draft = false
tags = ["backend", "database"]
series = ["Database Thực Chiến"]
+++

**Database Engineering — từ First Principles đến quyết định kiến trúc production**

Dành cho Software Engineer, Backend Engineer, Senior Engineer, Tech Lead, Solution Architect và Software Architect. Viết theo tinh thần Principal Engineer hướng dẫn Senior Engineer: mọi kết luận đều trả lời *tại sao*, *đánh đổi điều gì*, và *điều gì xảy ra nếu làm ngược lại*.

---

## Mục lục

### [Chương 1 — Database Fundamentals](/series/database/01-database-fundamentals/)
Nền tảng bắt buộc trước khi đọc các chương sau: Disk vs Memory, Page, Buffer Pool, WAL, Transaction & ACID, Isolation Levels, MVCC, B+Tree vs LSM Tree, Query Execution & Optimizer, Replication, Partitioning vs Sharding, CAP & PACELC, Distributed Systems Fundamentals. Kết chương bằng bản đồ tư duy đối chiếu ba database.

### [Chương 2 — PostgreSQL](/series/database/02-postgresql/)
Process architecture và hệ quả (connection pooling), MVCC kiểu version-in-heap và cái giá VACUUM/bloat, WAL & checkpoint, Query Planner & statistics, hệ index (B-Tree/GIN/GiST/BRIN, partial/covering), JSONB, Partitioning, Streaming & Logical Replication, HA với Patroni. Đầy đủ 11 mục: điểm mạnh/yếu, trade-off, production, best practices, anti-patterns, khi nào không dùng, decision framework.

### [Chương 3 — MongoDB](/series/database/03-mongodb/)
Document model & BSON, nguyên tắc "query together, store together", WiredTiger (B+Tree, MVCC, cache, nén), Indexing (ESR, multikey, TTL), Aggregation Pipeline, Replica Set & oplog, tunable consistency (writeConcern/readConcern), Sharding & nghệ thuật chọn shard key, multi-document transactions và vì sao chúng nên là ngoại lệ. Đầy đủ 11 mục như trên.

### [Chương 4 — ClickHouse](/series/database/04-clickhouse/)
Vì sao RDBMS thất bại về cấu trúc với OLAP lớn; Columnar storage & codec nén; MergeTree, part, quy tắc vàng batch insert; Sparse index & granule; Partitioning & TTL; các biến thể Replacing/Summing/AggregatingMergeTree; Materialized View; vectorized execution; Replication qua Keeper & Distributed table. Đầy đủ 11 mục như trên.

### [Chương 5 — So sánh PostgreSQL, MongoDB, ClickHouse](/series/database/05-so-sanh/)
Bảng so sánh 16 tiêu chí; ba trục phân tích (mô hình dữ liệu, ngưỡng scale, TCO); bảng "dịch thuật" khái niệm giữa ba engine; mô hình phối hợp source-of-truth → CDC → analytics.

### [Chương 6 — Kiến trúc thực tế & Decision Framework](/series/database/06-kien-truc-thuc-te/)
12 loại hệ thống: E-commerce, FinTech, Social Network, SaaS, Messaging, Ride Sharing, Blockchain, AI Platform, Real-time Analytics, Event Tracking, Logging, Data Warehouse — mỗi loại: chọn gì, vì sao, vì sao không chọn phương án khác, scale, chi phí, rủi ro. Kết bằng cây quyết định, bảng điều kiện, checklist 7 bước cho architect.

---

## Lộ trình đọc gợi ý

- **Engineer muốn hiểu sâu một database đang dùng:** Chương 1 → chương database tương ứng.
- **Tech Lead/Architect đang chọn công nghệ:** Chương 5 → Chương 6 → quay lại chương chi tiết của ứng viên (đặc biệt mục 5, 9, 10 — điểm yếu, anti-pattern, khi nào không dùng).
- **Chuẩn bị phỏng vấn senior/staff:** Chương 1 kỹ nhất, sau đó mục 3 (cách hoạt động bên trong) và 6 (trade-off) của từng chương.

## Ba nguyên tắc xuyên suốt bộ tài liệu

1. **Mọi thiết kế database là hệ quả của vật lý phần cứng** — sequential I/O rẻ, random I/O đắt; RAM nhanh, disk bền. Hiểu điều này thì VACUUM, oplog hay MergeTree đều trở thành "tất nhiên phải thế".
2. **Không có database tốt nhất, chỉ có điểm đánh đổi khớp workload.** Câu hỏi đúng không phải "hệ nào mạnh hơn" mà là "hệ nào chọn đánh đổi giống bài toán của ta".
3. **Chi phí thật của một database là chi phí vận hành nó lúc 3 giờ sáng.** TCO gồm cả con người, không chỉ hạ tầng.

---

*Các chi tiết tính năng theo phiên bản major gần đây của từng hệ (PostgreSQL 16+, MongoDB 7+, ClickHouse 24+); nguyên lý kiến trúc không phụ thuộc phiên bản.*
