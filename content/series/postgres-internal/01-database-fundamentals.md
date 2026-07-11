+++
title = "Chương 1 — Database Fundamentals: từ First Principles"
date = "2026-07-11T08:00:00+07:00"
draft = false
tags = ["backend", "database", "postgresql"]
series = ["PostgreSQL Internals"]
+++

> Level 1. Trước khi nói về PostgreSQL, phải trả lời câu hỏi gốc:
> **tại sao phần mềm quản lý dữ liệu lại khó đến mức cần một engine hàng triệu dòng code?**

---

## 1. Problem Statement

Mọi hệ thống phần mềm đều quy về một bài toán: **nhận dữ liệu, lưu dữ liệu, trả lại dữ liệu đúng**.

Một hệ thống thanh toán nhận lệnh chuyển 500.000đ từ tài khoản A sang B. Yêu cầu nghe đơn giản:

- Tiền trừ ở A **phải** cộng ở B — không được mất giữa chừng.
- Server chết giữa chừng (mất điện, kernel panic, OOM kill) — dữ liệu không được sai.
- 10.000 giao dịch đồng thời — không giao dịch nào đọc thấy trạng thái dở dang của giao dịch khác.
- Dữ liệu đã báo "thành công" cho khách — không bao giờ được biến mất.

Nếu không có database engine, bạn phải tự giải cả 4 bài toán trên bằng file. Chương này chứng minh rằng **tự giải bằng file là bất khả thi ở quy mô production**, và từng ràng buộc vật lý của disk/OS đã ép các database engine — trong đó có PostgreSQL — hội tụ về cùng một tập thiết kế.

## 2. Tại sao không thể "chỉ dùng file"?

### 2.1. Thí nghiệm tư duy: ngân hàng bằng file JSON

Giả sử ta lưu số dư bằng một file JSON và code Go "ngây thơ":

```go
// NAIVE — mọi dòng dưới đây đều chứa ít nhất một lỗi chết người
func Transfer(from, to string, amount int64) error {
    data, _ := os.ReadFile("balances.json")        // (1)
    var balances map[string]int64
    json.Unmarshal(data, &balances)

    balances[from] -= amount                        // (2)
    balances[to]   += amount

    out, _ := json.Marshal(balances)
    return os.WriteFile("balances.json", out, 0644) // (3)
}
```

Từng dòng thất bại như sau:

**(1) Race condition — Concurrency Problem.** Hai goroutine (hoặc hai process) cùng đọc `balances.json`, cùng thấy A có 1.000.000đ, cùng trừ 500.000đ, cùng ghi lại. Kết quả: chỉ trừ một lần. Đây là **lost update**. Muốn tránh phải lock — nhưng lock cả file thì mọi giao dịch xếp hàng tuần tự, throughput sập.

**(2) Không có atomicity — Transaction Problem.** Nếu process bị kill giữa hai dòng gán (SIGKILL không chờ ai), trạng thái trong memory mất, nhưng nếu đã ghi một nửa ra file thì tiền "bốc hơi". Cần khái niệm **transaction**: hoặc cả hai thay đổi cùng xảy ra, hoặc không thay đổi nào xảy ra.

**(3) `os.WriteFile` không atomic và không durable — Storage/Recovery Problem.** Đây là chỗ hầu hết engineer đánh giá thấp. Ba sự thật vật lý:

- `write()` chỉ copy dữ liệu vào **OS page cache** (RAM của kernel). Mất điện lúc này → dữ liệu mất, dù syscall đã trả về thành công.
- Muốn dữ liệu chạm platter/NAND thật, phải gọi `fsync()`. Nhưng `fsync` trên một file lớn có thể mất hàng chục ms — gọi mỗi giao dịch thì chậm, không gọi thì mất dữ liệu.
- Kernel ghi file ra disk theo đơn vị block, **không theo thứ tự bạn ghi**, và một lần ghi file lớn có thể bị đứt giữa chừng (torn write). Sau crash, file JSON có thể là nửa cũ nửa mới — **không parse nổi**.

Bài học: hệ thống lưu trữ đáng tin cậy không thể xây trên "ghi đè file". Nó phải được xây trên các **nguyên thủy (primitive) mà phần cứng thật sự đảm bảo** — và các đảm bảo đó nhỏ hơn nhiều so với trực giác.

### 2.2. Phần cứng thật sự đảm bảo gì?

```
┌───────────────────────────────────────────────────────────────┐
│ TẦNG                  ĐƠN VỊ ATOMIC       ĐỘ TRỄ ĐẠI DIỆN     │
├───────────────────────────────────────────────────────────────┤
│ CPU cache line        64 byte             ~1 ns               │
│ RAM                   8 byte (aligned)    ~100 ns             │
│ NVMe SSD (write)      4 KB page*          ~20–100 µs          │
│ SATA SSD (write)      4 KB page*          ~50–500 µs          │
│ HDD (random write)    512 B/4 KB sector   ~5–10 ms            │
│ fsync (NVMe)          —                   ~0.1–1 ms           │
│ fsync (HDD)           —                   ~10–30 ms           │
│ Network RTT (cùng DC) —                   ~0.5 ms             │
└───────────────────────────────────────────────────────────────┘
* SSD chỉ đảm bảo atomic ở mức sector; ghi 8KB có thể rách đôi (torn page).
```

Ba hệ quả chi phối toàn bộ thiết kế database:

1. **Random I/O đắt hơn sequential I/O.** Trên HDD chênh ~100 lần (seek 5–10ms so với đọc tuần tự 200MB/s). Trên SSD chênh ít hơn nhưng vẫn tồn tại (đặc biệt với write, do erase block của NAND). → Database sẽ luôn tìm cách **biến random write thành sequential write**. Đây chính là lý do WAL tồn tại (Chương 5).

2. **Không có atomic write lớn hơn một sector.** Ghi một page 8KB có thể rách giữa chừng khi mất điện. → Database phải có cơ chế phát hiện và sửa torn page (full page write — Chương 5).

3. **Độ trễ phân tầng ~10³ mỗi bậc** (RAM nhanh hơn SSD ~1000 lần). → Cache là bắt buộc, không phải tối ưu. Database phải tự quản lý cache của mình (Buffer Manager — Chương 4) vì OS cache không hiểu ngữ nghĩa transaction.

### 2.3. Filesystem giúp gì — và không giúp gì

Filesystem (ext4, xfs...) cung cấp: không gian tên (file, thư mục), cấp phát block, journaling **cho metadata của chính nó**. Điều nhiều người nhầm: journal của ext4 (chế độ mặc định `data=ordered`) chỉ bảo vệ metadata (inode, bitmap) — **không bảo vệ nội dung dữ liệu của bạn**. Sau crash, file của bạn tồn tại, đúng kích thước, nhưng nội dung có thể là rác cũ.

Filesystem cũng không cung cấp: transaction đa file, lock theo dòng, kiểm soát thứ tự ghi ra disk (trừ khi bạn tự `fsync` đúng chỗ), hay khả năng truy vấn.

Kết luận: **database engine là tầng phần mềm biến các đảm bảo yếu của disk + filesystem thành các đảm bảo mạnh mà ứng dụng cần (ACID)**. Toàn bộ độ phức tạp của PostgreSQL nằm ở khoảng cách giữa hai tập đảm bảo này.

## 3. ACID — định nghĩa bằng ngôn ngữ của người xây engine

ACID thường được đọc như khẩu hiệu. Với người xây engine, mỗi chữ là một **bài toán kỹ thuật riêng với cơ chế riêng**:

| Chữ | Nghĩa | Bài toán kỹ thuật | Cơ chế trong PostgreSQL |
|-----|-------|-------------------|--------------------------|
| **A**tomicity | Tất cả hoặc không gì | Undo các thay đổi dở dang sau crash/abort | MVCC: version cũ còn nguyên; CLOG đánh dấu XID aborted → version mới tự động vô hình (Chương 6) |
| **C**onsistency | Ràng buộc luôn đúng | Kiểm tra constraint đúng thời điểm, đúng snapshot | Constraint checking trong Executor + trigger + serializable |
| **I**solation | Transaction không thấy trạng thái dở dang của nhau | Concurrency control | MVCC + Snapshot + Lock (Chương 6, 7) |
| **D**urability | Đã commit là không mất | Chống mất điện, torn write | WAL + fsync + full page write + checkpoint (Chương 5, 11) |

Điểm tinh tế đáng chú ý: **PostgreSQL không có undo log riêng** (khác Oracle, MySQL/InnoDB). Atomicity đạt được "miễn phí" nhờ MVCC: abort chỉ là một bit trong CLOG, các row version rác để lại cho VACUUM dọn. Đây là quyết định thiết kế nền tảng, và cái giá của nó (bloat, vacuum) chiếm nguyên Chương 10.

## 4. Hai họ Storage Engine — và vị trí của PostgreSQL

Từ ràng buộc "random I/O đắt", ngành database hội tụ về hai họ cấu trúc lưu trữ chính:

```
                    ┌────────────────────────────┐
                    │  Làm sao lưu và tìm record?│
                    └──────────┬─────────────────┘
          ┌────────────────────┴────────────────────┐
          ▼                                         ▼
┌───────────────────────┐               ┌───────────────────────────┐
│ B-tree / Heap family  │               │  LSM-tree family          │
│ (update-in-place*)    │               │  (append-only + compact)  │
├───────────────────────┤               ├───────────────────────────┤
│ PostgreSQL, MySQL,    │               │ RocksDB, Cassandra,       │
│ Oracle, SQL Server    │               │ LevelDB, ClickHouse(≈)    │
├───────────────────────┤               ├───────────────────────────┤
│ + Đọc điểm nhanh, ổn  │               │ + Write throughput rất cao│
│ + Range scan tốt      │               │ + Sequential write thuần  │
│ − Write khuếch đại    │               │ − Đọc phải merge nhiều    │
│   qua WAL + page      │               │   tầng (read amplification)│
│ − Cần vacuum/GC       │               │ − Compaction ngốn I/O nền │
└───────────────────────┘               └───────────────────────────┘
* PostgreSQL đặc biệt: heap là "append version mới + dọn rác sau",
  không ghi đè tuple cũ tại chỗ — hệ quả trực tiếp của MVCC.
```

PostgreSQL chọn **Heap không clustered + B-tree index tách rời + MVCC append-version**:

- **Heap không clustered**: bảng là đống page chứa tuple không theo thứ tự nào. Index trỏ vào vị trí vật lý `(page, offset)`. Trade-off so với clustered (InnoDB lưu bảng ngay trong B-tree theo primary key): PostgreSQL đọc theo PK chậm hơn một chút (phải nhảy heap), nhưng secondary index không phải mang PK, không bị page split của bảng ảnh hưởng, và bảng không có "cột đặc quyền". Chi tiết ở Chương 3.
- **MVCC append-version**: UPDATE = INSERT version mới + đánh dấu version cũ. Reader không bao giờ block writer. Cái giá: dead tuple, VACUUM. Chi tiết Chương 6, 10.

## 5. Cách hoạt động — vòng đời một thay đổi dữ liệu (bức tranh sẽ vẽ chi tiết ở các chương sau)

```
Client: UPDATE accounts SET balance = balance - 500000 WHERE id = 'A';

  Backend Process
      │ 1. Parser: SQL text → parse tree            (Chương 8)
      │ 2. Planner: chọn Index Scan trên id         (Chương 8, 9)
      │ 3. Executor: chạy plan                      (Chương 8)
      ▼
  Buffer Manager                                    (Chương 4)
      │ 4. Đọc heap page chứa tuple vào Shared Buffers
      │    (nếu chưa có: read() từ disk qua OS cache)
      │ 5. Lock tuple, kiểm tra visibility (MVCC)   (Chương 6, 7)
      ▼
  Heap                                              (Chương 3)
      │ 6. Ghi tuple VERSION MỚI vào page (không ghi đè bản cũ)
      │ 7. Đánh dấu version cũ: xmax = XID hiện tại
      ▼
  WAL                                               (Chương 5)
      │ 8. Ghi WAL record mô tả thay đổi vào WAL buffer
      │ 9. COMMIT → flush WAL buffer + fsync xuống pg_wal/
      │    ★ ĐÂY là thời điểm durability được xác lập.
      │    Heap page bẩn vẫn nằm trong RAM, CHƯA cần ghi disk.
      ▼
  Sau đó, bất đồng bộ:
      │ 10. Checkpointer/BgWriter ghi dần dirty page về data file (Chương 4, 11)
      │ 11. VACUUM dọn version cũ khi không transaction nào cần   (Chương 10)
      ▼
  Nếu crash trước bước 10:
      │ Startup process replay WAL từ checkpoint gần nhất → dựng lại
      │ dirty page chưa kịp ghi. Không mất dữ liệu đã commit.      (Chương 11)
```

Hai nguyên lý rút ra, đáng khắc cốt:

1. **Commit ≠ ghi data file. Commit = ghi WAL.** Data file được phép lạc hậu tùy ý, vì WAL + recovery dựng lại được. Điều này biến mọi commit thành **sequential write nhỏ** — giải đúng bài toán "random I/O đắt" ở mục 2.2.
2. **Không gì bị xóa ngay lập tức.** UPDATE/DELETE chỉ đánh dấu. Sự trì hoãn này mua được concurrency (reader không block) và atomicity (abort miễn phí), trả giá bằng công việc dọn dẹp nền.

## 6. Trade-off nền tảng (sẽ gặp lại ở mọi chương)

| Trade-off | Hai phía | PostgreSQL chọn | Giá phải trả |
|-----------|----------|------------------|--------------|
| Concurrency | Lock (2PL) vs MVCC | MVCC | Dead tuple, VACUUM, XID wraparound |
| Storage | Clustered vs Heap | Heap | Đọc theo PK cần thêm bước heap fetch |
| Durability | fsync mỗi commit vs định kỳ | fsync WAL mỗi commit (group commit gộp) | Độ trễ commit ~ độ trễ fsync |
| Write path | In-place + undo log vs append version | Append version | Write amplification ở heap, bloat |
| Cache | Tự quản toàn bộ (O_DIRECT) vs chia với OS | Chia với OS (double buffering) | Tốn RAM gấp đôi cho page nóng, nhưng đơn giản, tận dụng readahead của kernel |
| Process model | Thread vs Process | Process per connection | Connection đắt, cần pooler; đổi lại cô lập lỗi tốt |

Không lựa chọn nào đúng tuyệt đối. Chuỗi chương sau giải thích từng lựa chọn đủ sâu để bạn dự đoán được **khi nào nó gãy trong production** — vì mọi thiết kế đều gãy, chỉ khác là gãy ở đâu.

## 7. Failure case dẫn nhập: "write thành công" mà vẫn mất dữ liệu

**Triệu chứng:** Một service ghi log giao dịch ra file, `write()` trả về OK, service báo khách thành công. Máy mất điện. Khởi động lại: 30 giây dữ liệu cuối biến mất.

**Root cause:** `write()` chỉ đưa dữ liệu vào OS page cache; kernel flush theo chu kỳ (`vm.dirty_expire_centisecs`, mặc định 30 giây). Không ai gọi `fsync`.

**Điều PostgreSQL làm khác:** mỗi COMMIT (với `synchronous_commit = on`) đều `fsync` WAL segment. PostgreSQL còn xử lý cả trường hợp thâm hiểm hơn: **fsyncgate** (2018) — khi `fsync` trả lỗi, một số kernel đánh dấu page sạch, lần fsync sau trả OK dù dữ liệu chưa ghi. Từ PostgreSQL 12+, phản ứng mặc định là **PANIC ngay khi fsync lỗi** (`data_sync_retry = off`) và phục hồi từ WAL, thay vì tin lời kernel.

**Bài học:** độ tin cậy không đến từ "gọi đúng API" mà từ **hiểu API đó thật sự hứa gì**. Đây là tinh thần của toàn bộ tài liệu.

## 8. Kiểm chứng nhanh trên máy của bạn

Đo chênh lệch chi phí durability — cùng ghi 10.000 record 100 byte:

```go
// bench_fsync.go — minh họa chi phí thật của durability
func benchWrite(withFsync bool) time.Duration {
    f, _ := os.OpenFile("bench.dat", os.O_CREATE|os.O_WRONLY, 0644)
    defer f.Close()
    buf := make([]byte, 100)
    start := time.Now()
    for i := 0; i < 10000; i++ {
        f.Write(buf)
        if withFsync {
            f.Sync() // fsync — điều PostgreSQL làm cho WAL mỗi commit
        }
    }
    return time.Since(start)
}
```

Kết quả đại diện (NVMe SSD):

```
withFsync=false : ~15 ms      (~660k writes/s)  — nhanh, nhưng KHÔNG durable
withFsync=true  : ~6.000 ms   (~1.6k writes/s)  — durable, chậm hơn ~400 lần
```

Khoảng cách ~400 lần này chính là "ngân sách" mà mọi database phải tối ưu. PostgreSQL thu hẹp nó bằng group commit (nhiều transaction chia nhau một fsync — Chương 5) và cho phép bạn đánh đổi có kiểm soát bằng `synchronous_commit = off` (chấp nhận mất tối đa ~600ms dữ liệu khi crash, **không bao giờ corrupt**).

## 9. Tóm tắt chương

- Disk và filesystem chỉ đảm bảo rất ít: atomic ở mức sector, durable chỉ sau fsync, thứ tự ghi không được tôn trọng.
- Database engine tồn tại để lấp khoảng cách giữa đảm bảo yếu đó và ACID mà ứng dụng cần.
- Chi phí vật lý (random vs sequential, RAM vs disk) ép mọi engine hội tụ về: WAL cho durability, cache tự quản cho tốc độ, và một cơ chế concurrency control.
- PostgreSQL chọn: heap không clustered, MVCC append-version, WAL + fsync mỗi commit, process-per-connection, chia cache với OS. Mỗi lựa chọn là một trade-off sẽ được mổ xẻ ở các chương sau.

**Tiếp theo:** [Chương 2 — Kiến trúc tổng thể: Process & Shared Memory](/series/postgres-internal/02-architecture-process-memory/)
