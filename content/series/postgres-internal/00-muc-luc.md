+++
title = "PostgreSQL Internals — Bên dưới lớp SQL"
date = "2026-07-11T07:00:00+07:00"
draft = false
tags = ["backend", "database", "postgresql"]
series = ["PostgreSQL Internals"]
+++

> Bộ tài liệu chuyên sâu về cách PostgreSQL hoạt động bên dưới lớp SQL và Storage,
> viết cho Backend Engineer, Database Engineer và Software Architect.

## Tài liệu này KHÔNG phải là gì

Đây không phải tài liệu dạy SQL. Không có `CREATE TABLE` ở chương mở đầu. Không giải thích cú pháp.

Đây là tài liệu giúp bạn hiểu PostgreSQL như một **hệ điều hành thu nhỏ dành cho dữ liệu**: cách nó tổ chức bộ nhớ, quản lý page, lưu tuple, điều phối transaction, ghi WAL, phục hồi sau sự cố, tối ưu truy vấn, và phối hợp với OS để đạt hiệu năng và độ tin cậy.

## Triết lý trình bày

Mọi chương đi theo dòng tư duy:

```
Business Problem
      ↓
Tại sao cần Database?
      ↓
Dữ liệu nằm ở đâu? (Disk, Filesystem)
      ↓
Database phải tổ chức dữ liệu thế nào? (Storage Engine)
      ↓
Memory Management → Transaction → Concurrency → Recovery
      ↓
Query Processing
      ↓
Production & Trade-off
```

Mỗi thành phần đều phải trả lời đủ 5 câu hỏi:

1. Nó tồn tại để giải quyết vấn đề gì?
2. Nếu loại bỏ nó, PostgreSQL gặp vấn đề gì?
3. Tại sao PostgreSQL chọn thiết kế này thay vì cách khác?
4. Trade-off của quyết định thiết kế là gì?
5. Tác động production (hiệu năng, độ tin cậy, khả năng mở rộng) ra sao?

## Mục lục

| # | Chương | Level | Nội dung chính |
|---|--------|-------|----------------|
| 01 | [Database Fundamentals — từ First Principles](/series/postgres-internal/01-database-fundamentals/) | 1 | Disk, filesystem, tại sao cần database engine, ACID, durability |
| 02 | [Kiến trúc tổng thể — Process & Shared Memory](/series/postgres-internal/02-architecture-process-memory/) | 1–2 | Postmaster, backend, checkpointer, bgwriter, shared buffers, SLRU |
| 03 | [Physical Storage — Page, Tuple, TOAST, FSM, VM](/series/postgres-internal/03-physical-storage/) | 2 | Relation file, segment, page layout, tuple header, line pointer, fork |
| 04 | [Buffer Manager](/series/postgres-internal/04-buffer-manager/) | 2 | Buffer pool, clock sweep, pin, dirty page, ring buffer, OS cache |
| 05 | [WAL — Write-Ahead Log](/series/postgres-internal/05-wal/) | 3 | WAL record, LSN, group commit, full page write, fsync |
| 06 | [Transaction Engine — MVCC, Snapshot, Visibility](/series/postgres-internal/06-mvcc-transactions/) | 3 | XID, snapshot, visibility rules, hint bit, CLOG, isolation |
| 07 | [Lock Manager](/series/postgres-internal/07-lock-manager/) | 3 | Row lock, table lock, LWLock, spinlock, deadlock |
| 08 | [Query Processing — Parser đến Executor](/series/postgres-internal/08-query-processing/) | 4 | Parser, rewriter, planner, cost model, executor |
| 09 | [Index Internals](/series/postgres-internal/09-index-internals/) | 4 | B-tree, Hash, GIN, GiST, BRIN — page layout, split, scan, cost |
| 10 | [VACUUM, HOT, Freeze, Wraparound](/series/postgres-internal/10-vacuum-hot-freeze/) | 3–5 | Dead tuple, vacuum phases, HOT chain, freeze, autovacuum |
| 11 | [Checkpoint, Recovery, Replication](/series/postgres-internal/11-recovery-replication/) | 3–5 | Crash recovery, WAL replay, PITR, streaming replication |
| 12 | [Production Failure Cases](/series/postgres-internal/12-production-failure-cases/) | 5 | 21 failure case: bloat, wraparound, checkpoint storm, lock contention... |
| 13 | [Anti-pattern & Khi nào không dùng PostgreSQL](/series/postgres-internal/13-antipatterns-limits/) | 5 | Anti-patterns, giới hạn kiến trúc, lựa chọn thay thế |

## Cách đọc

**Nếu bạn là Backend Engineer** muốn hiểu tại sao query chậm: đọc 01 → 03 → 04 → 06 → 08 → 09 → 12.

**Nếu bạn là Database Engineer / DBA**: đọc tuần tự toàn bộ. Chương 12 là tài liệu tra cứu khi trực production.

**Nếu bạn là Architect** đang chọn database: đọc 01, 02, 06, 13.

## Quy ước

- Thuật ngữ chuyên ngành (Page, Heap, Tuple, WAL, MVCC, Checkpoint...) giữ nguyên tiếng Anh.
- Diagram dùng ASCII để đọc được ở mọi nơi.
- Pseudo code mô tả thuật toán viết bằng **Go** để dễ đọc với backend engineer (source thật của PostgreSQL là C).
- Source code reference theo cây thư mục PostgreSQL: `src/backend/...` (nhánh master, các dòng lệnh có thể xê dịch giữa các version).
- Số liệu benchmark là số **đại diện** (representative) đo trên SSD NVMe hiện đại — dùng để hiểu bậc độ lớn (order of magnitude), không phải để so sánh tuyệt đối. Luôn tự benchmark trên hạ tầng của bạn.
- Nội dung bám theo PostgreSQL 16/17. Những điểm khác biệt lớn giữa các version được ghi chú rõ.

## Bức tranh tổng thể — giữ trong đầu suốt bộ tài liệu

```
                        ┌──────────────────────────────────────────┐
                        │                CLIENT                    │
                        └──────────────────┬───────────────────────┘
                                           │ libpq protocol
                        ┌──────────────────▼───────────────────────┐
                        │           BACKEND PROCESS (1/conn)       │
                        │  Parser → Rewriter → Planner → Executor  │
                        └──────┬─────────────────────────┬─────────┘
                               │                         │
              ┌────────────────▼──────────┐   ┌──────────▼──────────┐
              │      SHARED MEMORY        │   │     LOCK MANAGER    │
              │  ┌─────────────────────┐  │   │  heavyweight locks  │
              │  │   Shared Buffers    │  │   │  LWLocks, spinlocks │
              │  │  (page cache 8KB)   │  │   └─────────────────────┘
              │  └─────────────────────┘  │
              │  ┌──────────┐ ┌────────┐  │   ┌─────────────────────┐
              │  │WAL Buffer│ │  SLRU  │  │   │ BACKGROUND PROCESSES│
              │  └──────────┘ │ (CLOG) │  │   │ checkpointer        │
              │  ┌──────────┐ └────────┘  │   │ background writer   │
              │  │Proc Array│              │   │ wal writer          │
              │  └──────────┘              │   │ autovacuum workers  │
              └──────┬──────────────┬──────┘   └─────────┬───────────┘
                     │              │                    │
              ┌──────▼──────┐ ┌─────▼─────┐              │
              │  DATA FILES │ │  WAL FILES│◄─────────────┘
              │ base/…/16384│ │ pg_wal/…  │
              └──────┬──────┘ └─────┬─────┘
                     │              │
              ┌──────▼──────────────▼──────┐
              │   OS PAGE CACHE / FS       │
              └──────────────┬─────────────┘
                             │
              ┌──────────────▼─────────────┐
              │        DISK (SSD/NVMe)     │
              └────────────────────────────┘
```

Mọi chương sau đây là việc phóng to từng ô trong sơ đồ này.
