+++
title = "Bài 2 — PostgreSQL"
date = "2026-07-02T10:00:00+07:00"
draft = false
tags = ["backend", "database"]
series = ["Database Thực Chiến"]
+++

> **Tiền đề:** đã đọc Chương 1 (WAL, MVCC, B+Tree, buffer pool, replication).
> **Góc nhìn:** từ vị trí người thiết kế engine, không phải người dùng API.

---

## 1. Problem Statement

**PostgreSQL giải bài toán gì?** Lưu trữ dữ liệu quan hệ với **tính đúng đắn tuyệt đối** dưới concurrency cao: transaction ACID đầy đủ, constraint được thực thi nghiêm ngặt, SQL chuẩn và giàu tính năng, mở rộng được (extension), trong khi vẫn đủ nhanh cho đại đa số workload OLTP.

**Nếu không có nó thì sao?** Trước kỷ nguyên open-source database trưởng thành, lựa chọn là: trả phí rất đắt cho Oracle/DB2/SQL Server, hoặc dùng MySQL thời kỳ đầu (MyISAM — không transaction, không FK, silent data corruption). Khoảng trống: một RDBMS mã nguồn mở, đúng chuẩn, đáng tin cho dữ liệu nghiêm túc.

**Vì sao bài toán này khó?** Vì "đúng đắn dưới concurrency" là phần khó nhất của database engineering: hàng nghìn transaction đan xen, mỗi cái phải thấy một thế giới nhất quán, rollback được, sống sót qua crash — đồng thời không được chậm. PostgreSQL chọn điểm đánh đổi: **đúng trước, nhanh sau** — và rồi dành 30 năm làm cho nó nhanh.

---

## 2. Tại sao PostgreSQL tồn tại

**Bối cảnh lịch sử.** Xuất phát từ dự án POSTGRES của Michael Stonebraker tại UC Berkeley (1986), kế thừa Ingres. Ý tưởng khác biệt từ đầu: **object-relational** — hệ kiểu mở rộng được (user-defined types, operators, index methods). Đây không phải chi tiết lịch sử vô thưởng vô phạt: chính kiến trúc mở rộng này là lý do 30 năm sau PostgreSQL "mọc" ra được JSONB, PostGIS, full-text search, pgvector — những thứ database khác phải xây engine riêng để có.

**Áp lực kỹ thuật định hình thiết kế:**

- Cần MVCC để reader/writer không chặn nhau → chọn thiết kế **giữ mọi phiên bản ngay trong heap table** (khác Oracle/InnoDB dùng undo log riêng) → hệ quả trực tiếp: cần VACUUM.
- Cần độ tin cậy tuyệt đối → WAL chặt chẽ, fsync mặc định, full-page writes chống torn page.
- Cần mở rộng → mọi thứ là catalog: type, function, operator, index method đều là dữ liệu trong bảng hệ thống, extension chỉ việc thêm row.

**Vị trí hôm nay:** mặc định de-facto cho OLTP mã nguồn mở; nền tảng cho cả một hệ sinh thái (Timescale, Citus, Supabase, Neon, AlloyDB, Aurora PostgreSQL đều xây trên/tương thích nó).

---

## 3. Cách hoạt động bên trong

### 3.1. Process Architecture

PostgreSQL dùng mô hình **process-per-connection** (không phải thread):

```
                    ┌────────────────────────────┐
Client ──connect──► │ Postmaster (process chính) │
                    └──────────┬─────────────────┘
                        fork() │
        ┌──────────────┬──────┴────────┬──────────────────┐
        ▼              ▼               ▼                  ▼
   Backend #1     Backend #2      Background workers   Auxiliary:
   (1 client)     (1 client)      (parallel query,     - WAL writer
        │              │           autovacuum...)      - Checkpointer
        └──────┬───────┘                                - Background writer
               ▼                                        - Archiver
        Shared Memory                                   - Stats collector
        (shared_buffers, WAL buffers, lock table)
```

**Hệ quả kỹ thuật của process-per-connection:**

- Mỗi connection tốn vài MB + chi phí fork → **connection đắt**. 5000 connection trực tiếp là thảm họa (context switch, lock contention trên shared memory).
- Đây là lý do **connection pooling (PgBouncer/pgpool) gần như bắt buộc** trong production — không phải tùy chọn. Kiến trúc microservices với hàng trăm pod, mỗi pod một pool 20 connection, sẽ giết PostgreSQL nếu không có pooler tập trung.
- Đổi lại: cô lập lỗi tốt (một backend crash không sập cả server theo cách thread model có thể), code đơn giản và ổn định hơn.

**Điều gì xảy ra nếu làm ngược lại?** MySQL dùng thread-per-connection — chịu được nhiều connection hơn, nhưng một thread hỏng bộ nhớ có thể ảnh hưởng cả process. Không có lựa chọn đúng tuyệt đối; PostgreSQL chọn an toàn + đơn giản, và vá điểm yếu bằng pooler.

### 3.2. MVCC theo kiểu PostgreSQL — và cái giá của nó

Như Chương 1: mỗi row version (tuple) mang `xmin` (transaction tạo ra nó) và `xmax` (transaction xóa/thay nó). Điểm **đặc thù của PostgreSQL**: phiên bản cũ nằm **ngay trong heap table**, lẫn với phiên bản sống.

```
UPDATE users SET name='B' WHERE id=1;

Heap page trước:  [ (id=1,'A') xmin=100 xmax=∞ ]
Heap page sau:    [ (id=1,'A') xmin=100 xmax=205 ]   ← dead tuple, vẫn chiếm chỗ
                  [ (id=1,'B') xmin=205 xmax=∞  ]   ← tuple mới
```

So sánh thiết kế với Oracle/InnoDB (undo log riêng):

| | PostgreSQL (version trong heap) | Oracle/InnoDB (undo riêng) |
|---|---|---|
| UPDATE | Ghi tuple mới hoàn chỉnh | Sửa tại chỗ + ghi undo |
| Rollback | Gần như miễn phí (bỏ qua tuple mới) | Phải áp undo (đắt, tỷ lệ với lượng ghi) |
| Đọc bản cũ | Ngay trong heap | Phải lần theo undo chain |
| Dọn rác | **VACUUM quét heap** | Purge undo (gọn hơn) |
| Update-heavy | Bảng bloat | Undo phình (ORA-01555) |

**HOT update (Heap-Only Tuple)** — tối ưu quan trọng: nếu UPDATE không đổi cột nào có index và page còn chỗ, tuple mới được đặt cùng page và **index không cần cập nhật**. Hệ quả thiết kế thực dụng: bảng update nhiều nên để `fillfactor < 100` (chừa chỗ trong page) và **tránh index thừa** — mỗi index thêm vào làm giảm cơ hội HOT.

### 3.3. VACUUM và Autovacuum — nguồn sự cố production số một

VACUUM làm ba việc sống còn:

1. **Dọn dead tuple** — trả chỗ trống cho page (không trả OS trừ khi VACUUM FULL).
2. **Cập nhật visibility map** — bitmap đánh dấu page toàn tuple sống; là điều kiện để **index-only scan** hoạt động (không cần ghé heap kiểm tra visibility).
3. **Freeze** chống **transaction ID wraparound** — XID là số 32-bit quay vòng; tuple quá già phải được "đóng băng" (đánh dấu vĩnh viễn nhìn thấy). Nếu không freeze kịp: PostgreSQL **ngừng nhận ghi** để tự bảo vệ. Đây là sự cố nghiêm trọng nhất hệ PostgreSQL có thể gặp — và nó đã đánh sập những công ty lớn (sự cố Joyent/Manta, Sentry đều được mổ xẻ công khai).

**Ví dụ thất bại kinh điển (tổng hợp từ nhiều post-mortem thực tế):** bảng queue kiểu `UPDATE ... SET status='done'` hàng nghìn lần/giây. Dead tuple sinh nhanh hơn autovacuum mặc định dọn. Bảng 2GB dữ liệu sống phình lên 80GB. Index bloat theo. Query từ 1ms lên 500ms. Đội ứng phó bằng cách... thêm replica đọc, không giải quyết gốc rễ. Cuối cùng phải `pg_repack` giữa đêm.

**Kết luận bắt buộc nhớ:** autovacuum mặc định được cấu hình cho máy nhỏ những năm 2000. Trên máy hiện đại với workload ghi lớn, **phải chỉnh**: `autovacuum_vacuum_cost_limit` tăng mạnh, `autovacuum_vacuum_scale_factor` giảm cho bảng lớn (0.2 mặc định nghĩa là bảng 1 tỷ row phải chờ 200 triệu dead tuple mới được dọn — quá muộn), giám sát `n_dead_tup` và tuổi XID (`age(relfrozenxid)`) như giám sát disk full.

### 3.4. WAL trong PostgreSQL

Áp dụng nguyên lý Chương 1 với các chi tiết riêng:

- WAL chia thành segment 16MB, vị trí đo bằng **LSN** (Log Sequence Number).
- **Checkpoint:** thời điểm mọi dirty page trước LSN X đã xuống disk → recovery chỉ replay từ X. Checkpoint quá thưa = recovery lâu; quá dày = I/O spike + full-page writes nhiều (sau mỗi checkpoint, lần sửa đầu tiên của mỗi page phải ghi cả page vào WAL). Tuning thực tế: `max_wal_size` đủ lớn, `checkpoint_completion_target=0.9` để trải I/O.
- WAL là đầu vào cho: crash recovery, **streaming replication**, **PITR** (point-in-time recovery qua WAL archiving), và **logical decoding** (nền tảng CDC — Debezium đọc từ đây).

### 3.5. Query Planner / Optimizer

Cost-based đầy đủ (Chương 1, mục 1.7). Các đặc điểm riêng đáng chú ý:

- Thống kê thu bởi `ANALYZE` (mẫu ngẫu nhiên): histogram, most-common values, `n_distinct`, correlation. Cột tương quan nhau (city ↔ country) làm planner nhân selectivity sai → **`CREATE STATISTICS`** (extended statistics) là công cụ ít người biết nhưng cứu nhiều query.
- **Không có query hint chính thống** (triết lý dự án: sửa planner chứ không vá tay từng query) — gây tranh cãi; thực tế có extension `pg_hint_plan`.
- Plan cache của prepared statement: sau vài lần chạy có thể chuyển sang generic plan — thỉnh thoảng gây "query nhanh 5 lần đầu, chậm từ lần 6" — bug report kinh điển thực ra là feature.
- `EXPLAIN (ANALYZE, BUFFERS)` là công cụ số một: so `rows=` ước lượng với thực tế; lệch 100x nghĩa là statistics có vấn đề.

### 3.6. Hệ index — vũ khí mạnh nhất của PostgreSQL

| Index | Cấu trúc | Dùng cho | Ghi chú |
|---|---|---|---|
| **B-Tree** | B+Tree | `= < > BETWEEN ORDER BY` | Mặc định; 95% trường hợp |
| **GIN** | Inverted index | JSONB, mảng, full-text | Nhiều key/row; ghi đắt (fastupdate buffer), đọc rất nhanh |
| **GiST** | Cây cân bằng tổng quát | Hình học, range, khoảng cách | Nền của PostGIS; hỗ trợ exclusion constraint |
| **BRIN** | Min/max theo block range | Bảng khổng lồ, dữ liệu tương quan vật lý (time-series) | Nhỏ hơn B-Tree hàng nghìn lần; chỉ tốt khi dữ liệu ghi theo thứ tự |
| **Hash** | Hash | `=` thuần | Hiếm khi đáng dùng hơn B-Tree |
| **SP-GiST** | Cây không cân bằng | Dữ liệu phân cụm (IP, quadtree) | Ngách |

Các kỹ thuật cấp senior:

- **Partial index:** `CREATE INDEX ... WHERE status='active'` — index nhỏ hơn 100 lần nếu chỉ query dữ liệu active.
- **Covering index:** `INCLUDE (col)` → index-only scan, khỏi ghé heap (nhớ: cần visibility map sạch — lại là VACUUM).
- **Expression index:** `ON lower(email)` — bắt buộc nếu query dùng biểu thức.
- **Anti-pattern phổ biến nhất:** index mọi cột "cho chắc". Mỗi index làm chậm mọi INSERT/UPDATE, giảm HOT, phình VACUUM. Quan sát `pg_stat_user_indexes.idx_scan = 0` để tìm index chết.

### 3.7. JSONB — document store bên trong RDBMS

JSONB lưu JSON dạng nhị phân đã parse, index được bằng GIN (`@>`, `?`, jsonpath). Ý nghĩa kiến trúc: **rất nhiều trường hợp "cần MongoDB" thực ra chỉ cần một cột JSONB** — được luôn transaction, JOIN, constraint. Giới hạn phải biết: sửa 1 field = ghi lại toàn bộ giá trị JSONB (MVCC copy-on-write) → document to + update nhiều = bloat; không có thống kê bên trong JSONB → planner đoán mò selectivity; TOAST (nén/tách giá trị lớn ra bảng phụ) thêm chi phí đọc với document lớn.

### 3.8. Partitioning

Declarative partitioning (range/list/hash) từ PG10, trưởng thành từ PG12+:

- **Được:** partition pruning lúc plan/execute; DROP partition thay DELETE (giải pháp đúng cho data retention); autovacuum chạy song song theo partition.
- **Giới hạn phải biết:** unique constraint phải chứa partition key; quá nhiều partition (hàng chục nghìn) làm planning chậm; global index không tồn tại.
- Time-series thực dụng: partition theo tháng/ngày + BRIN trong mỗi partition, hoặc dùng TimescaleDB.

### 3.9. Replication & High Availability

**Streaming replication (physical):** ship WAL bytes → replica là bản sao bit-level, chỉ đọc. Sync/async/quorum per-transaction (`synchronous_commit`). Đơn giản, vững — nhưng replica cùng version, replicate *tất cả*.

**Logical replication:** decode WAL thành thay đổi logic (INSERT/UPDATE/DELETE) → chọn lọc bảng, khác version, làm nền cho **zero-downtime major upgrade** và CDC. Giới hạn: không replicate DDL, sequence cần xử lý riêng.

**HA thực tế:** PostgreSQL **không có failover tích hợp** — cần Patroni (+etcd/Consul làm consensus lưu leader lock) hoặc dịch vụ quản lý (RDS/Cloud SQL). Các quyết định sống còn khi tự vận hành: fencing node cũ (tránh split-brain), `pg_rewind` để tái sử dụng leader cũ, và **replication slot** (giữ WAL cho replica chậm — nhưng slot bỏ quên = WAL đầy disk = sập; giám sát bắt buộc).

---

## 4. Điểm mạnh

- **Tính đúng đắn hàng đầu:** transactional DDL (migration rollback được!), constraint đầy đủ (CHECK, FK, EXCLUSION), Serializable thật (SSI). Thiết kế "correctness-first" thấm vào mọi tầng.
- **Planner + hệ index giàu nhất trong các OSS RDBMS** → xử lý được query phức tạp (CTE, window function, lateral join) mà nơi khác phải kéo dữ liệu về ứng dụng.
- **Extension ecosystem** — hệ quả trực tiếp của kiến trúc catalog mở: PostGIS, pgvector, TimescaleDB, Citus... Một engine, nhiều "database".
- **MVCC rollback rẻ + reader/writer không chặn nhau** → chịu tốt workload hỗn hợp OLTP + báo cáo nhẹ.

## 5. Điểm yếu

- **VACUUM/bloat** — cái giá vĩnh viễn của thiết kế version-in-heap (mục 3.3). Update-heavy ở tần suất cực cao là điểm đau cấu trúc.
- **Connection đắt** — cái giá của process model; bắt buộc pooler.
- **Write scale bị trần một máy.** Không có sharding tích hợp — vượt trần phải dùng Citus hoặc shard ở tầng ứng dụng, độ phức tạp nhảy vọt.
- **Analytics trên dữ liệu rất lớn:** row storage + executor row-at-a-time → full scan 1 tỷ row cho aggregation chậm hơn ClickHouse 10–1000 lần. Parallel query giúp nhưng không đổi được bản chất row-oriented.
- **Major version upgrade** cần quy trình (pg_upgrade/logical replication), không tự động như dịch vụ quản lý.

## 6. Trade-off

| Trade-off | PostgreSQL chọn | Nguyên nhân kỹ thuật | Hệ quả production |
|---|---|---|---|
| Read vs Write | Cân bằng, nghiêng read-consistency | B+Tree + MVCC heap | Update-heavy cần tuning (fillfactor, autovacuum) |
| Consistency vs Availability | Consistency | Single-leader, sync tùy chọn | Failover cần tầng ngoài; RPO=0 phải trả bằng latency (sync) |
| Rollback vs Update cost | Rollback rẻ | Version trong heap | Bloat + VACUUM |
| Schema integrity vs flexibility | Integrity, nhưng có JSONB làm van xả | Catalog + constraint enforcement | Migration có kỷ luật; JSONB cho phần "mềm" |
| Đơn giản vận hành vs scale-out | Đơn giản (1 node mạnh) | Không sharding tích hợp | Trần ghi = trần một máy; vượt trần = trả phí phức tạp lớn |
| Latency vs Durability | Durability mặc định | fsync WAL mỗi commit | `synchronous_commit=off` là van xả có kiểm soát (mất ≤ vài trăm ms khi crash, không hỏng dữ liệu) |

## 7. Production Considerations

- **HA:** Patroni + etcd (tự vận hành) hoặc managed service. Kiểm tra failover định kỳ như diễn tập PCCC — HA chưa từng diễn tập là HA trên giấy.
- **Backup:** `pg_basebackup`/pgBackRest + **WAL archiving → PITR**. Nguyên tắc: *backup chưa từng restore thử = không có backup*. Replica KHÔNG phải backup (DROP TABLE replicate sang replica ngay lập tức).
- **Monitoring tối thiểu:** replication lag (bytes + giây), `n_dead_tup` & tuổi XID, connection count vs max, cache hit ratio, lock chờ lâu, `pg_stat_statements` (top query theo total_time), disk WAL, slot bị bỏ quên.
- **Capacity planning:** RAM đủ cho working set (cache hit > 99% với OLTP); NVMe cho WAL; connection = pool tập trung, không phải max_connections to.
- **Security:** TLS, `pg_hba.conf` tối thiểu quyền, row-level security cho multi-tenant, không bao giờ để superuser cho ứng dụng.
- **Upgrade:** minor = restart đơn giản, làm đều đặn (vá bảo mật). Major = pg_upgrade (downtime ngắn) hoặc logical replication (gần zero-downtime).
- **Multi-region:** PostgreSQL không có multi-master tin cậy — mô hình thực tế là leader một region + replica region khác (DR + đọc local), chấp nhận write latency xuyên region hoặc chấp nhận RPO nhỏ.

## 8. Best Practices

- Data modeling: chuẩn hóa trước, phi chuẩn hóa có chủ đích sau khi đo; JSONB cho thuộc tính động; kiểu dữ liệu đúng (`timestamptz`, không lưu tiền bằng float — dùng `numeric`).
- Index: theo query thực tế (`pg_stat_statements` dẫn đường), partial/covering khi phù hợp, xóa index không dùng.
- Query: tránh `SELECT *`; batch insert bằng COPY; phân trang bằng keyset (`WHERE id > $last LIMIT n`) thay vì `OFFSET` lớn (OFFSET 1 triệu = đọc và vứt 1 triệu row).
- Lock hygiene: migration dùng `lock_timeout`; `CREATE INDEX CONCURRENTLY`; thêm cột NOT NULL có default là rẻ từ PG11, nhưng đổi kiểu cột vẫn rewrite cả bảng.
- Transaction ngắn — long-running transaction chặn VACUUM toàn cluster.

## 9. Anti-patterns

1. **Không có connection pooler**, hoặc max_connections=5000 "cho thoải mái".
2. **Queue trong bảng bằng UPDATE liên tục** không tuning vacuum/fillfactor → bloat (nếu cần queue trong PG: `FOR UPDATE SKIP LOCKED` + xóa/partition, hoặc dùng công cụ chuyên dụng).
3. **Bỏ mặc autovacuum mặc định** trên bảng ghi lớn.
4. **Chạy analytics nặng trực tiếp trên primary OLTP** → snapshot lâu → giữ rác → bloat lan toàn hệ thống (đây là lý do kiến trúc "OLTP PG + CDC → ClickHouse" tồn tại — xem Chương 6).
5. **Replication slot mồ côi** → WAL ăn hết disk → primary sập.
6. **ORM sinh N+1 query** — không phải lỗi PostgreSQL nhưng PostgreSQL bị đổ tội.
7. **Tin rằng Read Committed bảo vệ bất biến nghiệp vụ** (double-booking; xem Chương 1, mục 1.4).

## 10. Khi nào KHÔNG nên dùng PostgreSQL

- **Analytics quét hàng tỷ row, aggregation nặng, dashboard real-time** → ClickHouse/BigQuery (nguyên nhân cấu trúc: row storage + executor OLTP).
- **Write throughput vượt xa một máy** và không muốn tự shard → hệ có sharding tích hợp (MongoDB, Cassandra, hoặc Citus nếu muốn ở lại hệ PG).
- **Cache/kv thuần với latency µs** → Redis. Dùng PG làm cache là dùng sai công cụ.
- **Log/metrics append-only khối lượng cực lớn** → ClickHouse/Loki; PG sẽ chết vì chi phí index + WAL + vacuum trên dữ liệu chỉ-ghi-thêm.
- **Vì sao nhiều đội vẫn chọn sai?** Vì "PostgreSQL làm được mọi thứ" *đúng ở quy mô nhỏ* — JSONB thay Mongo được, partition thay ClickHouse được... đến khi scale lên và các chi phí cấu trúc (vacuum, row storage) lộ ra. "Làm được" ≠ "là lựa chọn đúng ở quy mô mục tiêu".

## 11. Decision Framework

**Chọn PostgreSQL khi** (càng nhiều điều kiện đúng càng chắc chắn):

- Dữ liệu có quan hệ, cần JOIN, cần constraint bảo vệ bất biến nghiệp vụ.
- Cần transaction ACID mạnh (tiền, đơn hàng, tồn kho, booking).
- Working set vừa RAM một máy lớn; write throughput trong khả năng một máy (thực tế: hàng chục nghìn TPS đơn giản là bình thường trên phần cứng tốt).
- Muốn một engine phủ nhiều nhu cầu (quan hệ + JSONB + full-text + geo + vector) để giảm số hệ phải vận hành.

**Tránh hoặc bổ sung hệ khác khi:** analytics quy mô lớn (→ ClickHouse), write scale-out là yêu cầu ngày một (→ hệ sharded), cache nóng (→ Redis).

**Quy tắc thực dụng cho đa số công ty:** *bắt đầu bằng PostgreSQL cho dữ liệu giao dịch; chỉ thêm hệ khác khi có bằng chứng đo đạc rằng PostgreSQL không gánh nổi vai trò cụ thể đó.* Chi phí vận hành thêm một hệ database luôn lớn hơn ước tính.
