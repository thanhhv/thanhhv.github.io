+++
title = "Bài 1 — Database Fundamentals"
date = "2026-07-02T09:00:00+07:00"
draft = false
tags = ["backend", "database"]
series = ["Database Thực Chiến"]
+++

# Database Fundamentals

> **Đối tượng:** Software Engineer → Software Architect
> **Mục tiêu:** Hiểu bản chất vật lý và logic của mọi database engine, làm nền tảng để hiểu sâu PostgreSQL, MongoDB và ClickHouse ở các chương sau.

---

## 1.1. Problem Statement: Vì sao database là bài toán khó?

Về mặt bề ngoài, một database chỉ làm hai việc: **ghi dữ liệu xuống** và **đọc dữ liệu lên**. Vậy tại sao nhân loại cần hàng chục năm và hàng trăm engine khác nhau để giải bài toán này?

Câu trả lời nằm ở chỗ database phải thỏa mãn **đồng thời** nhiều ràng buộc mâu thuẫn nhau:

1. **Durability** — dữ liệu đã ghi không được mất, kể cả khi mất điện giữa chừng.
2. **Performance** — đọc/ghi phải nhanh, trong khi thiết bị lưu trữ bền (disk) lại chậm hơn RAM hàng trăm đến hàng nghìn lần.
3. **Concurrency** — hàng nghìn client đọc/ghi cùng lúc mà không dẫm lên nhau.
4. **Consistency** — dữ liệu phải đúng, không được ở trạng thái "nửa vời".
5. **Scale** — dữ liệu lớn hơn RAM, thậm chí lớn hơn một máy vật lý.

Mỗi ràng buộc đơn lẻ đều dễ. **Tổ hợp của chúng là bài toán trade-off**: tăng cái này phải trả giá bằng cái kia. Toàn bộ lịch sử database engineering là lịch sử của việc *chọn điểm đánh đổi khác nhau* cho các workload khác nhau. Đây là lý do PostgreSQL, MongoDB và ClickHouse cùng tồn tại — chúng không cạnh tranh nhau trên cùng một trục, chúng **chọn điểm đánh đổi khác nhau**.

---

## 1.2. Storage Fundamentals: Disk vs Memory

### Vấn đề gốc rễ: khoảng cách tốc độ

Mọi quyết định thiết kế trong database engine đều bắt nguồn từ một sự thật vật lý:

| Thao tác | Độ trễ (xấp xỉ) | Quy đổi trực quan |
|---|---|---|
| L1 cache reference | ~1 ns | 1 giây |
| RAM access | ~100 ns | ~2 phút |
| SSD random read (4KB) | ~16–100 µs | ~vài giờ |
| SSD sequential read (1MB) | ~200 µs–1 ms | — |
| HDD seek | ~2–10 ms | ~vài tháng |
| Network round-trip (cùng DC) | ~500 µs | ~vài ngày |

Hai hệ quả then chốt:

**Hệ quả 1 — Sequential I/O rẻ hơn random I/O rất nhiều.** Trên HDD, chênh lệch có thể lên tới 100–1000 lần. Trên SSD, chênh lệch nhỏ hơn nhưng vẫn đáng kể (do cơ chế đọc theo block, write amplification, và hàng đợi lệnh). Đây là lý do WAL, LSM Tree, và columnar storage đều được thiết kế xoay quanh **ghi tuần tự**.

**Hệ quả 2 — Không thể đọc/ghi từng byte.** Disk làm việc theo **block** (thường 4KB). Đọc 1 byte hay 4KB tốn chi phí gần như nhau. Database vì thế tổ chức dữ liệu theo **page**.

### Page — đơn vị làm việc của mọi database

**Page** (PostgreSQL gọi là page/block, mặc định 8KB; MySQL InnoDB 16KB; ClickHouse làm việc theo granule/block cột) là đơn vị nhỏ nhất mà engine đọc từ disk lên memory và ghi ngược xuống.

Cấu trúc điển hình của một page (row-oriented):

```
+--------------------------------------------------+
| Page Header (checksum, LSN, free space pointers) |
+--------------------------------------------------+
| Item Pointers (slot array) →                     |
|                                                  |
|            free space                            |
|                                                  |
|                      ← Tuples (dữ liệu thực)     |
+--------------------------------------------------+
```

Thiết kế "slot array mọc từ đầu, tuple mọc từ cuối" cho phép:

- Thêm/xóa tuple mà không phải dịch chuyển toàn bộ page.
- Tham chiếu ổn định đến tuple qua `(page_number, slot_index)` — chính là **TID/ctid** trong PostgreSQL.

**Điều gì xảy ra nếu làm ngược lại?** Nếu lưu tuple sát nhau không qua slot array, mỗi lần UPDATE làm tuple to ra sẽ phải dịch cả page và **mọi index trỏ vào page đó bị vô hiệu**. Trade-off ở đây: tốn vài byte cho slot array để đổi lấy tính ổn định của con trỏ.

### Buffer Pool — cầu nối giữa disk và memory

Vì disk chậm, mọi engine đều có **buffer pool** (PostgreSQL: `shared_buffers`; MongoDB/WiredTiger: internal cache; ClickHouse: dựa nhiều vào OS page cache): một vùng RAM cache các page đang dùng.

Luồng đọc:

```
Query → cần page P
  → P có trong buffer pool? → CÓ  → trả về (nhanh, ~100ns)
                            → KHÔNG → đọc từ disk (chậm, µs–ms)
                                     → nạp vào buffer pool
                                     → evict page khác nếu đầy (LRU/Clock)
```

Ba vấn đề kỹ thuật buffer pool phải giải:

1. **Eviction policy.** LRU thuần bị "cache pollution" khi một sequential scan lớn quét qua và đá hết page nóng ra ngoài. PostgreSQL dùng clock-sweep + ring buffer riêng cho scan lớn; MySQL dùng LRU hai vùng (young/old). Đây là ví dụ điển hình của việc *thuật toán sách giáo khoa không sống nổi trong production*.
2. **Dirty page.** Page bị sửa trong RAM nhưng chưa ghi xuống disk. Không thể ghi ngay từng page mỗi lần sửa (random I/O chết hiệu năng), nhưng để lâu quá thì recovery lâu và rủi ro cao. Giải pháp: **WAL + checkpoint** (xem 1.3).
3. **Double caching.** OS cũng có page cache. PostgreSQL dùng buffered I/O nên dữ liệu có thể nằm 2 nơi → khuyến nghị `shared_buffers` ~25% RAM thay vì 80%. ClickHouse chọn hướng ngược lại: gần như phó thác cho OS page cache. Không có lựa chọn "đúng tuyệt đối", chỉ có lựa chọn khớp với kiến trúc.

---

## 1.3. WAL — Write-Ahead Logging

### Problem

Mâu thuẫn cốt lõi: **muốn durability thì phải ghi disk trước khi báo thành công; nhưng ghi page xuống disk là random I/O đắt đỏ**, và một transaction có thể chạm hàng chục page rải rác.

Tệ hơn: ghi một page 8KB không atomic ở tầng phần cứng (sector thường 512B/4KB). Mất điện giữa chừng → **torn page** — page hỏng một nửa.

### Giải pháp: tách "ghi để bền" khỏi "ghi để đọc"

Ý tưởng của WAL:

> Trước khi sửa bất kỳ page nào, ghi **mô tả thay đổi** (log record) vào một file log **tuần tự**, và fsync file log đó. Page thật được ghi xuống disk *sau*, một cách lười biếng.

```
Client: COMMIT
   │
   ▼
1. Ghi log record vào WAL buffer
2. fsync WAL xuống disk  ←— điểm durability duy nhất
3. Báo client: thành công
   ...
4. (nền) Checkpointer/Background writer ghi dirty page xuống disk sau
```

**Vì sao nhanh?** WAL là **append-only, sequential write** — dạng I/O rẻ nhất. Nhiều transaction commit gần nhau còn được gộp chung một lần fsync (**group commit**).

**Recovery hoạt động thế nào?** Sau crash, engine replay WAL từ checkpoint gần nhất: những thay đổi đã log nhưng chưa xuống page sẽ được áp lại (REDO). Torn page được xử lý bằng full-page image trong WAL (PostgreSQL: `full_page_writes`).

### Trade-off của WAL

- **Write amplification:** một thay đổi được ghi ít nhất 2 lần (WAL + page). Đổi lại: durability + hiệu năng commit.
- **fsync latency là sàn của commit latency.** Trên SSD tốt ~vài trăm µs. Muốn nhanh hơn phải nới lỏng durability (`synchronous_commit = off` — chấp nhận mất vài trăm ms dữ liệu gần nhất khi crash).
- **WAL trở thành nền tảng của replication.** Vì WAL mô tả đầy đủ mọi thay đổi, chỉ cần stream WAL sang máy khác là có replica — đây chính là streaming replication của PostgreSQL, oplog của MongoDB về mặt ý tưởng, và replication log của ClickHouse (qua Keeper).

**Điều gì xảy ra nếu không có WAL?** Hoặc phải fsync từng page ngay khi sửa (chậm không chấp nhận được), hoặc chấp nhận mất/hỏng dữ liệu khi crash. Không có lựa chọn thứ ba — đó là lý do *mọi* database nghiêm túc đều có một biến thể của WAL (journal của WiredTiger, redo log của InnoDB...).

---

## 1.4. Transaction và ACID

### Problem

Chuyển 100đ từ tài khoản A sang B gồm 2 lệnh UPDATE. Nếu crash giữa chừng, hoặc một client khác đọc vào đúng khoảnh khắc giữa hai lệnh, hệ thống rơi vào trạng thái tiền "bốc hơi" hoặc "nhân đôi". Không thể yêu cầu lập trình viên ứng dụng tự xử lý mọi kịch bản này — cần một abstraction: **transaction**.

### ACID — bốn cam kết, bốn cơ chế khác nhau

| Tính chất | Cam kết | Cơ chế hiện thực điển hình |
|---|---|---|
| **Atomicity** | Tất cả hoặc không gì cả | WAL + undo (rollback) — PostgreSQL rollback gần như miễn phí nhờ MVCC |
| **Consistency** | Ràng buộc dữ liệu luôn đúng | Constraint, trigger, FK — được kiểm tra trong transaction |
| **Isolation** | Transaction song song không "nhìn trộm" trạng thái dở dang của nhau | MVCC, locking |
| **Durability** | Đã commit là không mất | WAL + fsync |

Lưu ý quan trọng: **ACID không phải là nhị phân có/không**. Isolation có nhiều mức; Durability có thể nới lỏng. Khi một database quảng cáo "hỗ trợ ACID transactions", câu hỏi của một Senior Engineer phải là: *isolation level nào, mặc định là gì, và trả giá gì?*

### Isolation Levels — thang trade-off giữa đúng đắn và hiệu năng

Các hiện tượng bất thường (anomaly) theo chuẩn SQL:

- **Dirty read:** đọc thấy dữ liệu chưa commit của transaction khác.
- **Non-repeatable read:** đọc lại cùng một row trong cùng transaction thấy giá trị khác (vì transaction khác đã commit UPDATE).
- **Phantom read:** chạy lại cùng một query thấy row *mới xuất hiện* (INSERT của transaction khác).
- **Serialization anomaly / write skew:** hai transaction cùng đọc một điều kiện rồi ghi dựa trên điều kiện đó, kết quả tổ hợp vi phạm bất biến (ví dụ kinh điển: hai bác sĩ cùng rút khỏi ca trực vì "vẫn còn người khác trực").

| Level | Dirty read | Non-repeatable | Phantom | Write skew | Chi phí |
|---|---|---|---|---|---|
| Read Uncommitted | Có thể | Có thể | Có thể | Có thể | Thấp nhất |
| Read Committed | Không | Có thể | Có thể | Có thể | Thấp |
| Repeatable Read / Snapshot | Không | Không | Không* | Có thể | Trung bình |
| Serializable | Không | Không | Không | Không | Cao nhất |

*PostgreSQL Repeatable Read (thực chất là Snapshot Isolation) chặn được phantom trong đa số trường hợp, nhưng vẫn hở write skew.

**Bài học production:** mặc định của PostgreSQL là Read Committed. Đa số hệ thống chạy tốt ở mức này, nhưng các bất biến kiểu "tổng số ghế đã đặt ≤ số ghế" **không được bảo vệ** — phải dùng Serializable, `SELECT ... FOR UPDATE`, hoặc unique/exclusion constraint. Rất nhiều sự cố double-booking trong thực tế đến từ việc engineer tưởng Read Committed lo hết cho mình.

---

## 1.5. MVCC — Multi-Version Concurrency Control

### Problem: reader và writer chặn nhau

Cách ngây thơ để đạt isolation là **lock**: đọc thì lấy shared lock, ghi thì lấy exclusive lock. Hậu quả: một báo cáo đọc 10 phút sẽ chặn mọi lệnh ghi trong 10 phút. Với workload OLTP có cả đọc dài lẫn ghi ngắn, đây là thảm họa.

### Ý tưởng MVCC

> Thay vì sửa dữ liệu tại chỗ, **tạo phiên bản mới**. Mỗi transaction nhìn thấy một **snapshot** — tập phiên bản nhất quán tại thời điểm nó bắt đầu.

Hệ quả vàng: **reader không bao giờ chặn writer, writer không bao giờ chặn reader**. Writer chỉ chặn writer khi ghi cùng một row.

Cơ chế (mô hình PostgreSQL):

```
Row "balance=100"  (xmin=50, xmax=∞)      ← tạo bởi txn 50
UPDATE balance=80  → không sửa tại chỗ, mà:
  - đánh dấu bản cũ: (xmin=50, xmax=90)   ← "chết" từ txn 90
  - tạo bản mới:     (xmin=90, xmax=∞)

Txn 85 (bắt đầu trước txn 90) đọc → thấy balance=100
Txn 95 (bắt đầu sau)          đọc → thấy balance=80
```

### Trade-off của MVCC — không có bữa trưa miễn phí

1. **Rác phiên bản cũ.** Các phiên bản "chết" phải được dọn. PostgreSQL dọn bằng **VACUUM** (chi tiết ở chương PostgreSQL — đây là nguồn đau khổ production số một). MySQL/InnoDB và WiredTiger cất bản cũ ở undo/history riêng, đổi lại rollback đắt hơn.
2. **Bloat.** Update-heavy workload sinh phiên bản nhanh hơn tốc độ dọn → bảng phình to, cache hit giảm, scan chậm dần.
3. **Snapshot quá lâu giữ rác sống mãi.** Một transaction/report chạy 2 giờ ngăn VACUUM dọn mọi thứ sinh ra trong 2 giờ đó — long-running transaction là kẻ thù thầm lặng của mọi hệ MVCC.

**Điều gì xảy ra nếu làm ngược lại (lock-based thuần)?** Nhìn SQL Server trước 2005: reader chặn writer, ứng dụng đầy `NOLOCK` hint (tự nguyện đọc bẩn) để sống sót. MVCC thắng thế trong hầu hết engine hiện đại vì workload thực tế luôn trộn đọc và ghi.

---

## 1.6. Index: B+Tree và LSM Tree — hai triết lý đối nghịch

### Problem

Không có index, tìm 1 row trong 100 triệu row = quét 100 triệu row. Index là cấu trúc phụ đổi **dung lượng + chi phí ghi** lấy **tốc độ tìm kiếm**. Câu hỏi thiết kế: tổ chức index thế nào trên thiết bị mà sequential I/O rẻ, random I/O đắt?

### B+Tree — tối ưu cho đọc

Cây cân bằng, node to bằng đúng một page, fan-out lớn (hàng trăm con/node):

```
                 [ 40 | 70 ]                ← root (luôn trong cache)
              /      |       \
      [10|25]     [50|60]    [80|95]        ← internal
      /  |  \     ...              \
 [leaf][leaf][leaf] ←→ [leaf] ←→ [leaf]     ← leaf, liên kết đôi
```

- **Tìm điểm:** O(log_fanout N) — với fan-out ~500, 1 tỷ key chỉ cần **4 mức** → thực tế 1–2 lần đọc disk (các mức trên nằm sẵn trong cache).
- **Range scan:** đến leaf đầu, lướt theo con trỏ ngang — tuần tự, rất nhanh.
- **Điểm yếu:** mỗi INSERT/UPDATE là một **random write** vào đúng vị trí trong cây; page split gây write amplification; ghi ngẫu nhiên rải khắp file.

B+Tree là lựa chọn của PostgreSQL, MySQL, và (dưới dạng biến thể) WiredTiger — vì workload OLTP đọc nhiều, đọc điểm, cần range scan.

### LSM Tree — tối ưu cho ghi

Đảo ngược triết lý: **không bao giờ ghi ngẫu nhiên**.

```
Write → MemTable (RAM, sorted)  → đầy → flush tuần tự thành SSTable L0
                                          ↓ (nền) compaction
        SSTable L1 (sorted, immutable)  → merge dần xuống L2, L3...
Read  → MemTable? → L0? → L1? ... (Bloom filter để bỏ qua file không chứa key)
```

- **Ghi:** append vào memtable + WAL → cực nhanh, hoàn toàn tuần tự.
- **Đọc điểm:** có thể phải kiểm tra nhiều tầng → chậm hơn B+Tree; cứu vãn bằng Bloom filter.
- **Chi phí ẩn: compaction.** Dữ liệu được ghi đi ghi lại nhiều lần khi merge giữa các tầng (write amplification 10–30x tùy cấu hình), ăn CPU và I/O nền — "compaction storm" là sự cố kinh điển của RocksDB/Cassandra.

### So sánh bản chất

| | B+Tree | LSM Tree |
|---|---|---|
| Ghi | Random, in-place (đắt) | Sequential, append-only (rẻ) |
| Đọc điểm | 1 đường đi duy nhất (rẻ) | Nhiều tầng (đắt hơn) |
| Range scan | Tự nhiên, tốt | Phải merge nhiều nguồn |
| Space | Fragmentation vừa phải | Cần chỗ trống cho compaction |
| Phù hợp | OLTP đọc-nhiều | Write-heavy, time-series, log |

**ClickHouse MergeTree** là họ hàng tư tưởng của LSM (ghi part immutable + merge nền) nhưng bỏ memtable, bỏ tối ưu đọc điểm — vì OLAP không cần đọc điểm. Cùng một nguyên lý vật lý (sequential I/O rẻ) sinh ra các thiết kế khác nhau tùy mục tiêu.

---

## 1.7. Query Execution và Query Optimizer

### Từ SQL đến kết quả

```
SQL text
  → Parser        → cây cú pháp
  → Rewriter      → áp dụng view, rule
  → Planner/Optimizer → chọn PHƯƠNG ÁN thực thi rẻ nhất
  → Executor      → chạy plan, trả kết quả
```

Điểm mấu chốt: SQL là ngôn ngữ **khai báo** — người dùng nói *cần gì*, không nói *làm thế nào*. Cùng một câu SQL có thể có hàng nghìn plan hợp lệ với chi phí chênh nhau **hàng triệu lần**. Optimizer là bộ não quyết định:

- **Access path:** seq scan hay index scan? (Đọc 90% bảng thì seq scan *nhanh hơn* index scan — vì tuần tự vs ngẫu nhiên.)
- **Join order:** join 5 bảng có 5! = 120 thứ tự; 10 bảng là 3,6 triệu.
- **Join algorithm:** Nested Loop (tốt khi một phía nhỏ + có index), Hash Join (tốt cho equi-join lớn), Merge Join (tốt khi hai phía đã sắp xếp).

### Cost-based optimization và gót chân Achilles: statistics

Optimizer ước lượng chi phí dựa trên **thống kê** (số row, phân bố giá trị, tỷ lệ distinct...). Từ đó suy ra **selectivity** — điều kiện WHERE này lọc còn bao nhiêu row.

**Hệ quả production quan trọng nhất:** khi statistics sai (bảng vừa nạp dữ liệu lớn chưa ANALYZE, phân bố lệch, các cột tương quan nhau), optimizer chọn sai plan — và **một query đang chạy 10ms đột nhiên chạy 10 phút mà không ai đổi một dòng code**. Kỹ năng đọc `EXPLAIN ANALYZE`, so sánh *estimated rows* với *actual rows*, là kỹ năng sống còn của Senior Engineer.

### Row-at-a-time vs Vectorized Execution

Executor truyền thống (Volcano model) xử lý **từng row một** qua từng operator — nhiều virtual function call, cache miss, không tận dụng SIMD. Tốt cho OLTP (mỗi query chạm ít row).

Executor phân tích hiện đại (ClickHouse, DuckDB) xử lý **theo batch hàng nghìn giá trị của một cột** — CPU chạy vòng lặp chặt trên mảng liên tục, tận dụng SIMD, cache locality. Nhanh hơn 10–100 lần cho aggregation. Đây là một nửa bí mật tốc độ của ClickHouse (nửa còn lại là columnar storage — chương 4).

---

## 1.8. Replication

### Problem

Một máy = một điểm chết (SPOF) + một trần đọc. Replication giải cả hai: **sao chép dữ liệu sang nhiều máy** để chịu lỗi và scale read.

### Các mô hình

**1. Single-leader (phổ biến nhất — PostgreSQL, MongoDB, MySQL):**

```
           ┌─→ Replica 1 (read-only)
Client → Leader (mọi ghi)
           └─→ Replica 2 (read-only)
```

Đơn giản, không xung đột ghi. Vấn đề trung tâm: **đồng bộ hay bất đồng bộ?**

- **Async:** leader báo thành công ngay sau khi ghi cục bộ → nhanh, nhưng leader chết trước khi replica kịp nhận → **mất dữ liệu đã báo thành công**.
- **Sync:** chờ ít nhất một replica xác nhận → không mất, nhưng latency tăng và replica chậm/kẹt kéo cả hệ thống.
- Thực tế production: **semi-sync / quorum** — chờ k trong n replica.

**2. Replication lag và hệ quả với ứng dụng.** Đọc từ replica async có thể thấy dữ liệu cũ vài ms đến vài phút. Anomaly kinh điển: user vừa đổi avatar, refresh trang (đọc trúng replica) thấy avatar cũ → nghĩ là bug. Các mức bảo đảm cần biết: **read-your-writes**, **monotonic reads**, **consistent prefix**. MongoDB cho chọn per-query qua read concern / read preference; với PostgreSQL, ứng dụng phải tự định tuyến (đọc-sau-ghi thì vào leader).

**3. Failover — phần khó nhất.** Phát hiện leader chết (bao lâu là "chết"? network chập chờn thì sao?), bầu leader mới (cần consensus — Raft/Paxos, hoặc công cụ ngoài như Patroni), tránh **split-brain** (hai leader cùng nhận ghi = hỏng dữ liệu gần như không cứu được). MongoDB tích hợp sẵn bầu cử trong Replica Set; PostgreSQL cần tầng ngoài — khác biệt vận hành đáng kể.

---

## 1.9. Partitioning và Sharding

### Phân biệt hai khái niệm hay bị lẫn

- **Partitioning:** chia một bảng lớn thành nhiều mảnh **trong cùng một máy/instance** — mục tiêu là quản trị (xóa dữ liệu cũ bằng DROP PARTITION thay vì DELETE) và hiệu năng (partition pruning — chỉ quét mảnh liên quan).
- **Sharding:** chia dữ liệu ra **nhiều máy** — mục tiêu là vượt trần một máy (ghi, dung lượng). Sharding = partitioning + distributed systems + tất cả nỗi đau của nó.

### Sharding: chọn shard key là quyết định quan trọng nhất

| Chiến lược | Ưu | Nhược |
|---|---|---|
| **Hash(key)** | Phân bố đều | Mất khả năng range query theo key |
| **Range(key)** | Range query tốt | Hotspot (key tăng dần → mọi ghi dồn 1 shard) |
| **Directory** | Linh hoạt | Thêm một service phải vận hành |

Ba vấn đề mà sharding *tạo ra*:

1. **Cross-shard query:** JOIN/aggregate xuyên shard phải scatter-gather → chậm, phức tạp.
2. **Cross-shard transaction:** cần 2PC hoặc từ bỏ atomicity xuyên shard. 2PC đắt và mong manh (coordinator chết giữa chừng = kẹt lock).
3. **Rebalancing:** thêm máy → di chuyển dữ liệu trong khi hệ thống đang chạy — bài toán vận hành rủi ro cao.

**Nguyên tắc Principal-level:** *sharding là giải pháp cuối cùng, không phải mặc định.* Một máy PostgreSQL hiện đại (64+ core, NVMe, RAM hàng trăm GB) phục vụ được lượng workload mà 10 năm trước cần cả cluster. Chi phí phức tạp của sharding là vĩnh viễn; hãy chắc chắn đã hết đường vertical scaling + read replica + caching trước khi trả cái giá đó.

---

## 1.10. CAP Theorem và PACELC

### CAP — hiểu đúng, đừng hiểu theo meme

Phát biểu chính xác: khi xảy ra **network Partition** (hai nhóm node mất liên lạc), hệ thống phải chọn:

- **C (Consistency — linearizability):** từ chối phục vụ ở phía thiểu số để không trả dữ liệu sai/cũ.
- **A (Availability):** mọi node vẫn trả lời → chấp nhận hai phía phân kỳ, dữ liệu không nhất quán.

Những hiểu lầm cần dập tắt:

1. **"Chọn 2 trong 3" là sai.** P không phải lựa chọn — network partition *sẽ* xảy ra. Câu hỏi thật là: *khi* partition xảy ra, chọn C hay A.
2. CAP nói về một tính chất C rất hẹp (linearizability) và A rất hẹp. Đa số hệ thống thực tế nằm ở vùng xám với các mức consistency trung gian.

### PACELC — mở rộng thực dụng hơn

> **P**artition → chọn **A** hay **C**; **E**lse (vận hành bình thường) → chọn **L**atency hay **C**onsistency.

Vế "Else" mới là vế quan trọng hàng ngày: **ngay cả khi không có sự cố**, muốn consistency mạnh hơn (chờ quorum xác nhận) thì latency cao hơn. Ví dụ:

- PostgreSQL + async replica: PC/EL — thường thì nhanh, đọc replica có thể cũ.
- PostgreSQL + sync replication: PC/EC — nhất quán, trả giá latency mỗi commit.
- MongoDB: tùy chỉnh per-operation qua `writeConcern`/`readConcern` — cùng một cluster có thể là EL cho log và EC cho thanh toán.
- ClickHouse replication: eventual consistency giữa các replica (PA/EL) — chấp nhận được vì analytics không cần đọc-ngay-sau-ghi tuyệt đối.

**Bài học:** đừng hỏi "database này CP hay AP?" — hãy hỏi "*thao tác này*, với *cấu hình này*, đánh đổi gì?".

---

## 1.11. Distributed Systems Fundamentals — những sự thật không thể né

Khi dữ liệu vượt ra ngoài một máy, các định luật sau chi phối mọi thiết kế:

1. **Network là bất định.** Gửi request không nhận được reply — không thể phân biệt: request lạc, server chết, server xử lý xong nhưng reply lạc. Hệ quả: **timeout + retry là bắt buộc**, và retry sinh ra yêu cầu **idempotency** (xử lý trùng không gây hại). Mọi API ghi tiền mà không có idempotency key là một sự cố đang chờ ngày xảy ra.

2. **Đồng hồ không đáng tin.** Clock skew giữa các máy là thực tế (NTP chỉ giảm, không triệt tiêu). "Lấy timestamp làm thứ tự sự kiện" giữa hai máy là sai về nguyên lý. Các hệ thống nghiêm túc dùng logical clock, hoặc TrueTime (Spanner) với phần cứng chuyên dụng.

3. **Consensus là đắt nhưng không tránh được.** Bầu leader, cấu hình cluster, membership — đều cần consensus (Raft/Paxos/ZAB). Mỗi quyết định consensus tốn ít nhất một round-trip đến đa số node. Đó là lý do các hệ thống chỉ dùng consensus cho **metadata/control plane** (MongoDB config servers, ClickHouse Keeper), còn data path thì dùng cơ chế nhẹ hơn.

4. **Failure là trạng thái thường trực, không phải ngoại lệ.** Cluster 1000 máy với MTBF mỗi máy 3 năm → trung bình gần 1 máy chết mỗi ngày. Thiết kế phải bắt đầu từ câu hỏi "khi X chết thì sao" chứ không phải "nếu".

---

## 1.12. Tổng kết: bản đồ tư duy cho ba chương tiếp theo

Ba database trong tài liệu này là ba lời giải khác nhau cho phương trình trade-off:

| | PostgreSQL | MongoDB | ClickHouse |
|---|---|---|---|
| Bài toán gốc | OLTP quan hệ, tính đúng đắn tối đa | Dev velocity + scale-out cho dữ liệu document | OLAP trên hàng tỷ row, tốc độ aggregation |
| Storage | Row-oriented, heap + B+Tree | Document (BSON), B+Tree (WiredTiger) | Columnar, MergeTree (họ LSM) |
| Concurrency | MVCC + VACUUM | MVCC (WiredTiger) + document-level lock | Gần như không cần (immutable parts, batch insert) |
| Consistency | Mạnh, mặc định | Tùy chỉnh per-operation | Eventual, đủ dùng cho analytics |
| Scale | Vertical + read replica (sharding qua extension) | Sharding tích hợp | Distributed table + shard tích hợp |
| Triết lý | "Đúng trước, nhanh sau" | "Linh hoạt và scale-out trước" | "Nhanh cho analytics trước, bỏ những gì OLAP không cần" |

Mọi chi tiết trong ba chương sau — từ VACUUM của PostgreSQL, oplog của MongoDB, đến compaction của ClickHouse — đều là hệ quả logic của các nguyên lý trong chương này: **sequential I/O rẻ, random I/O đắt; durability cần WAL; concurrency cần versioning; distributed cần trade-off C/A/L.** Nắm chắc chương này, các chương sau sẽ là "à, tất nhiên phải thế" thay vì "phải ghi nhớ".
