+++
title = "Phần 8 — Data Partitioning"
date = "2026-07-13T11:40:00+07:00"
draft = false
tags = ["backend", "system-design"]
series = ["System Design — Tư Duy Thiết Kế Hệ Thống"]
+++

> Chủ đề: Partitioning, Sharding, Consistent Hashing, phối hợp với Replication — và bài toán khó nhất: resharding hệ đang chạy.

## Luận điểm trung tâm của phần này

Partitioning là câu trả lời cho câu hỏi cuối cùng của scale: **khi một node không chứa nổi / phục vụ nổi toàn bộ dữ liệu thì sao?** Chia dữ liệu ra N node — nhưng cái giá là mất những gì "một node" cho không: transaction toàn cục, join tùy tiện, thứ tự toàn cục, và sự đơn giản. Vì thế nguyên tắc số một, nhắc lại từ [1.5](/series/system-design/01-foundations/05-bottleneck-analysis/): **sharding là phương án cuối, sau khi index/cache/replica/scale-up đã hết bài** — và là phương án gần như không có đường lùi.

## Mục lục

- [8.1. Partitioning & Sharding — chia dữ liệu và cái giá của shard key](/series/system-design/08-data-partitioning/01-partitioning-sharding/)
- [8.2. Consistent Hashing — thêm bớt node mà không xáo cả thế giới](/series/system-design/08-data-partitioning/02-consistent-hashing/)
- [8.3. Resharding & vận hành hệ đã shard](/series/system-design/08-data-partitioning/03-resharding-van-hanh/)

## Ba khái niệm hay lẫn — phân định một lần

| | Chia gì | Giải bài toán gì | Ví dụ |
|---|---|---|---|
| **Partitioning** (trong node) | Một bảng → nhiều phần *trên cùng node* | Quản trị + quét: DROP partition cũ, prune khi query | PostgreSQL partition theo tháng ([5.1 §7](/series/system-design/05-data-layer/01-postgresql/)) |
| **Sharding** (ra nhiều node) | Dữ liệu → nhiều node, mỗi node một phần | Vượt trần ghi/dung lượng một máy | Citus, Vitess, MongoDB sharded ([5.3](/series/system-design/05-data-layer/03-mongodb/)) |
| **Replication** | Nhân bản *toàn bộ* ra nhiều node | Availability + scale đọc | [4.2](/series/system-design/04-distributed-systems/02-replication-consistency/) |

Hệ lớn dùng cả ba chồng nhau: bảng partition theo thời gian, nằm trong shard theo tenant, mỗi shard có replica — ba lớp, ba mục đích, đừng để một từ "chia dữ liệu" che mất ba quyết định riêng.

## Sợi chỉ đỏ của cả phần

Mọi chương đều xoay quanh một sự thật: **sharding đứng trên giả định tải phân bố đều theo key, và thực tế phá giả định đó bằng luật lũy thừa.** Hot partition ([13.2](/series/system-design/13-production-failure-cases/02-database-failures/)), celebrity problem, tenant khổng lồ — thiết kế partition tốt không phải thiết kế cho phân bố đều, mà là thiết kế **biết trước phân bố sẽ lệch** và có sẵn đường xử lý.
