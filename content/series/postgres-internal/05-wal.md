+++
title = "Chương 5 — WAL: Write-Ahead Log"
date = "2026-07-11T12:00:00+07:00"
draft = false
tags = ["backend", "database", "postgresql"]
series = ["PostgreSQL Internals"]
+++

> Level 3. WAL là phát minh quan trọng nhất của ngành database (ARIES, 1992).
> Nó là lý do PostgreSQL dám giữ dirty page trong RAM, là nền của recovery,
> replication, PITR — và là nguồn của một nửa số sự cố production về disk.

---

## 1. Problem Statement

Chương 1 xác lập hai sự thật vật lý mâu thuẫn nhau:

- Durability đòi hỏi fsync trước khi báo commit — mà fsync đắt (~0.1–1ms NVMe, ~10ms HDD).
- Thay đổi của một transaction rải rác trên nhiều page (heap, nhiều index, toast) — fsync tất cả các page đó mỗi commit = nhiều random write + nhiều fsync → vài chục ms mỗi commit → throughput chết.

Thêm bài toán thứ ba: page 8KB > sector atomic 4KB → crash giữa lúc ghi page tạo **torn page** — nửa mới nửa cũ, checksum sai, không dùng được.

**Ý tưởng WAL:** đừng ghi thay đổi vào chỗ thật (random). Ghi **mô tả thay đổi** vào một file log **append-only, tuần tự** rồi fsync file đó — một lần, sequential, rẻ. Data file thật ghi sau, lúc nào tiện. Crash? Đọc log, làm lại (redo).

```
Không WAL:  commit = fsync(heap page) + fsync(index1) + fsync(index2) + ...
            = N random write + N fsync                     ≈ 5-50 ms

Có WAL:     commit = append(log) + 1 fsync (sequential)   ≈ 0.1-1 ms
            data file ghi sau, gom hàng nghìn thay đổi một lần
```

**Nếu bỏ WAL đi?** Một trong hai: hoặc mất durability (crash mất dữ liệu đã commit), hoặc phải fsync mọi page mỗi commit (chậm 10–100 lần). Không có lựa chọn thứ ba. (UNLOGGED table chính là "bỏ WAL cục bộ": nhanh hơn ~2 lần, đổi lấy việc bảng bị **truncate về rỗng** sau crash.)

## 2. The WAL Rule — một câu, hai vế

> **Trước khi một dirty page được ghi xuống disk, mọi WAL record mô tả thay đổi trên page đó phải đã nằm an toàn trên disk.**
> **Trước khi báo COMMIT thành công, WAL record commit của transaction phải đã fsync.**

Vế 1 thi hành ở buffer manager (Chương 4: `XLogFlush(page.pd_lsn)` trước khi evict/checkpoint ghi page). Vế 2 thi hành ở `CommitTransaction()`. Chỉ cần hai vế này, mọi trạng thái sau crash đều cứu được:

| Crash xảy ra khi | Data file | WAL | Recovery làm gì |
|---|---|---|---|
| Trước commit | Có thể lẫn thay đổi dở | Không có commit record | Thay đổi vô hình vĩnh viễn (MVCC + CLOG không bao giờ đánh dấu committed) |
| Sau fsync WAL, trước trả lời client | Chưa có gì | Đủ | Redo dựng lại; transaction TÍNH là committed (client nên viết code idempotent!) |
| Sau commit, trước checkpoint | Lạc hậu | Đủ | Redo dựng lại toàn bộ |

## 3. Internal Architecture

### 3.1. LSN — trục thời gian của cả hệ thống

LSN (Log Sequence Number) = **offset byte trong dòng WAL vô tận**, hiển thị dạng `A/B` (vd `5C/7A123456` — 64 bit). LSN xuất hiện khắp nơi:

- `pd_lsn` đầu mỗi page (Chương 3) — "page này đã chứa thay đổi tới LSN nào".
- Vị trí replay của replica — `SELECT pg_last_wal_replay_lsn()`.
- Replication lag = LSN primary − LSN replica (đơn vị: **byte**, không phải giây).
- Checkpoint = một LSN mốc.

Toàn bộ recovery và replication của PostgreSQL là các phép so sánh trên trục LSN. Ví dụ cốt lõi của redo:

```go
// Khi replay một WAL record cho page P:
if page.LSN >= record.LSN {
    skip() // page trên disk ĐÃ chứa thay đổi này (được ghi trước crash)
} else {
    applyRedo(page, record)
    page.SetLSN(record.LSN)
}
// → replay idempotent: chạy lại bao nhiêu lần cũng ra cùng kết quả
```

### 3.2. WAL record

```
XLogRecord header (24B):
  xl_tot_len   tổng độ dài
  xl_xid       transaction tạo record
  xl_prev      LSN của record TRƯỚC ★ tạo backward chain — phát hiện
               điểm đứt của log (điểm dừng replay) mà không cần "EOF marker"
  xl_info      loại record (HEAP_INSERT, BTREE_SPLIT, COMMIT...)
  xl_rmid      resource manager (heap, btree, xact, ...)
  xl_crc       CRC-32C của record ★ record hỏng CRC = điểm kết thúc log hợp lệ
+ block references: page nào bị ảnh hưởng (relfilenode, block)
+ payload: dữ liệu redo (tuple mới, offset, ...) hoặc FULL PAGE IMAGE
```

Hai chi tiết `xl_prev` và `xl_crc` trả lời câu hỏi hóc búa: *làm sao biết WAL kết thúc ở đâu sau crash?* — đọc tới record đầu tiên có CRC sai hoặc xl_prev không khớp: đó là ranh giới của những gì đã kịp ghi.

### 3.3. Full Page Write (FPW) — thuốc giải torn page

Vấn đề: redo dạng "sửa offset X trong page P" chỉ đúng nếu page P trên disk **toàn vẹn**. Nhưng crash có thể để lại P rách đôi (8KB ghi được 4KB). Redo trên page rách = rác.

Giải pháp: **lần đầu tiên** một page bị sửa sau mỗi checkpoint, WAL record chứa **toàn bộ ảnh 8KB của page** thay vì chỉ delta. Khi replay, gặp full page image thì ghi đè nguyên page — không cần page trên disk toàn vẹn.

```
checkpoint ──┬─ UPDATE page 42 → WAL: [FULL IMAGE 8KB của page 42]
             ├─ UPDATE page 42 → WAL: [delta 100B]
             ├─ UPDATE page 42 → WAL: [delta 80B]
 checkpoint ─┼─ UPDATE page 42 → WAL: [FULL IMAGE 8KB]  ← reset sau checkpoint
             └─ ...
```

**Hệ quả production lớn:** ngay sau mỗi checkpoint, lượng WAL sinh ra **tăng vọt** (mọi page nóng đều phải ghi full image lần đầu). Checkpoint càng dày → FPW càng nhiều → WAL càng phình → đây là lý do chính để **giãn checkpoint** (max_wal_size lớn, timeout 15–30 phút) trên hệ ghi nặng. Đo được: `pg_waldump --stats` xem tỉ lệ FPI (thường 40–70% dung lượng WAL!), `wal_compression = on/lz4` nén FPI đổi CPU lấy dung lượng.

### 3.4. WAL buffer → segment file

```
 backends ──ghi record──▶ WAL buffers (vòng, wal_buffers ~16MB, trong shared memory)
                              │
              ┌───────────────┴─ ai flush?
              │ • backend lúc COMMIT (synchronous_commit=on)
              │ • walwriter mỗi 200ms (cho async commit)
              │ • buffer đầy
              ▼
 pg_wal/000000010000000A0000002F   (segment 16MB, tên = timeline+LSN)
 pg_wal/000000010000000A00000030
              │ segment đầy → archiver copy đi (nếu bật archive_mode)
              │ segment cũ → tái sử dụng (rename) hoặc xóa
              ▼
 giữ lại tối thiểu để: crash recovery (từ checkpoint cuối), replication slot,
 wal_keep_size → NGUỒN CỦA "pg_wal ĐẦY DISK" (Chương 12 #5, #21)
```

Ghi vào WAL buffer có tối ưu đồng thời tinh vi: backend **giành chỗ** (reserve range byte) bằng một spinlock cực ngắn, rồi **copy dữ liệu song song** vào vùng của mình (`WALInsertLock` ×8 partition). Trước PG9.4 đây là single lock — từng là nghẽn ghi lớn nhất.

### 3.5. Group Commit

10 transaction cùng commit trong 1ms — không lẽ 10 lần fsync? Không: backend đến sau thấy có fsync đang chạy sẽ **chờ kết quả của lần fsync đó** (nếu LSN của nó đã được bao phủ) hoặc gom vào đợt sau. Kết quả: throughput commit vượt xa IOPS fsync của disk.

```
Benchmark đại diện (NVMe, fsync ~0.3ms, pgbench đơn giản):
  1 client :   ~2.500 tps   (mỗi commit một fsync — bị chặn bởi độ trễ fsync)
  64 clients: ~90.000 tps   (group commit: ~1.400 fsync/s phục vụ 90k commit)
→ Throughput tăng 36 lần trong khi số fsync/s gần như không đổi.
```

Tinh chỉnh thêm: `commit_delay/commit_siblings` (chờ chủ động để gom nhiều hơn — hiếm khi cần), `synchronous_commit`:

| Giá trị | Chờ gì lúc commit | Mất gì khi crash |
|---|---|---|
| `on` (mặc định) | WAL fsync local | Không mất gì |
| `remote_apply/remote_write/on`* | + replica nhận/ghi/apply | Không mất gì kể cả mất node (tùy mức) |
| `off` | Không chờ gì (WAL flush trong ≤3×wal_writer_delay) | Tối đa ~600ms transaction **cuối** — nhưng KHÔNG BAO GIỜ corrupt (WAL vẫn tuần tự) |

`synchronous_commit=off` là nút tăng tốc hợp pháp cho dữ liệu chịu được mất vài trăm ms (event log, metrics) — khác hoàn toàn với `fsync=off` (tắt fsync thật sự → crash = corrupt = **không bao giờ dùng production**).

## 4. Cách hoạt động — COMMIT từng bước

```
COMMIT;
 1. Backend ghi WAL record XLOG_XACT_COMMIT vào WAL buffer → nhận LSN L.
 2. XLogFlush(L):
    a. Đã có ai flush qua L chưa? Rồi → xong (group commit hưởng ké).
    b. Chưa → giành WALWriteLock, write() các trang WAL buffer tới L,
       fsync segment (fdatasync mặc định trên Linux).
 3. Ghi trạng thái COMMITTED vào CLOG (trong SLRU buffer — chưa cần fsync,
    vì WAL record commit đã durable, CLOG dựng lại được từ WAL).
 4. Nếu synchronous_standby_names: chờ ACK từ replica theo mức đã chọn.
 5. Nhả snapshot, nhả lock, trả "COMMIT" cho client.
```

Để ý bước 3: **CLOG cũng được WAL bảo vệ**. Mọi cấu trúc bền của PostgreSQL (heap, index, CLOG, VM, FSM một phần) đều sống dưới ô của WAL — "single source of truth" khi crash là WAL, mọi thứ khác chỉ là cache/materialization.

## 5. Bên dưới lớp vật lý

- **Segment 16MB** (đổi được lúc initdb `--wal-segsize`): đủ nhỏ để archive/recycle từng phần, đủ lớn để không tạo quá nhiều file. Tên file mã hóa (timeline, LSN cao, LSN thấp) — đọc được vị trí trong dòng log từ tên.
- **Recycle thay vì tạo mới:** segment cũ được rename thành segment tương lai — tránh chi phí cấp phát block mới của filesystem và giữ file đã "warm" về metadata. Đây là lý do `pg_wal` có sẵn nhiều file cả khi idle.
- **fdatasync vs fsync vs open_datasync:** `wal_sync_method` — mặc định Linux là fdatasync (bỏ qua flush metadata mtime). `pg_test_fsync` đo phương pháp nhanh nhất cho disk của bạn.
- **Disk riêng cho WAL** (symlink pg_wal sang volume khác): tách sequential write của WAL khỏi random I/O của data — vẫn đáng làm trên cloud (IOPS budget riêng), từng là bắt buộc thời HDD.
- **wal_level:** `minimal` (chỉ đủ crash recovery — không stream được) / `replica` (mặc định — đủ cho replication + PITR) / `logical` (+ thông tin giải mã row cho logical replication). Mức cao hơn = WAL nhiều hơn.

## 6. Trade-off

| Quyết định | Được | Mất |
|---|---|---|
| Redo-only, không undo log (khác ARIES đầy đủ, khác InnoDB) | Recovery đơn giản: chỉ replay tiến; abort miễn phí (MVCC) | Dead tuple + VACUUM (Chương 10); data file không bao giờ "sạch" tức thời |
| FPW mặc định bật | An toàn tuyệt đối với torn page trên mọi filesystem | WAL phình 2–5 lần quanh checkpoint; (ZFS/btrfs CoW có thể tắt — tự chịu trách nhiệm) |
| WAL vật lý (byte-level) thay logical | Replay nhanh, đúng tuyệt đối, replication vật lý rẻ | Replica vật lý phải giống hệt kiến trúc/version; logical decoding phải làm thêm |
| Một dòng WAL duy nhất cho cả cluster | Thứ tự toàn cục rõ ràng, đơn giản | Trần throughput ghi = tốc độ tuần tự hóa WAL (dù đã partition insert lock); các DB shared-nothing chọn nhiều log để scale write |

## 7. Production

- **Định cỡ `max_wal_size`:** đủ lớn để checkpoint xảy ra do `checkpoint_timeout` chứ không do đầy WAL. Kiểm tra: log có `checkpoints occurring too frequently` hoặc `pg_stat_bgwriter.checkpoints_req` áp đảo `checkpoints_timed` → tăng.
- **Theo dõi sản lượng WAL:** `pg_stat_wal` (PG14+): wal_bytes/s, wal_fpi. Tăng đột biến thường do: checkpoint dày, UPDATE rộng, REINDEX, hoặc bật wal_level=logical.
- **Archive:** `archive_command` fail → segment không được dọn → **pg_wal đầy → PANIC, database sập**. Giám sát `pg_stat_archiver.failed_count` là bắt buộc. Tương tự với **replication slot bỏ rơi** (`pg_replication_slots.restart_lsn` đứng yên).
- **`pg_waldump`**: công cụ đọc WAL — trả lời "cái gì đang sinh nhiều WAL vậy?" (`--stats=record` nhóm theo loại).
- **Đừng bao giờ xóa file trong pg_wal bằng tay.** Mọi trường hợp "pg_wal đầy" đều có nguyên nhân dọn được đúng cách (Chương 12 #5).

## 8. Failure case của chương: commit latency nhấp nhô

**Triệu chứng:** p50 commit 1ms, nhưng p99 thỉnh thoảng 200–800ms, theo cụm.

**Root cause phổ biến:** (a) checkpoint sync phase — fsync dồn dập làm nghẽn cả fsync của WAL nếu chung disk; (b) FPW ngay sau checkpoint làm mỗi UPDATE ghi 8KB WAL thay vì 100B, WAL flush to hơn; (c) ext4 journal contention (writeback của data file chặn fsync WAL — hiện tượng "stable page writes").

**Điều tra:** bật `log_checkpoints=on`, đối chiếu timestamp p99 spike với pha `sync` của checkpoint; `pg_stat_wal.wal_sync_time`; iostat xem `w_await` của volume WAL lúc spike.

**Khắc phục:** tách WAL ra volume riêng; giãn checkpoint + `checkpoint_completion_target=0.9`; `wal_compression=lz4`; cân nhắc `backend_flush_after`/`checkpoint_flush_after` để kernel writeback đều tay hơn.

## 9. Tóm tắt

- WAL biến "N random write + N fsync mỗi commit" thành "1 sequential append + 1 fsync (chia sẻ được)".
- LSN là trục thời gian của toàn hệ thống; replay idempotent nhờ so sánh pd_lsn.
- FPW giải quyết torn page, đổi bằng WAL phình quanh checkpoint — lý do số một để giãn checkpoint.
- Group commit cho throughput vượt IOPS fsync; `synchronous_commit=off` là trade-off hợp pháp, `fsync=off` là tự sát.
- WAL là single source of truth; heap/index/CLOG chỉ là "cache" dựng lại được.

**Tiếp theo:** [Chương 6 — MVCC](/series/postgres-internal/06-mvcc-transactions/): dùng xmin/xmax/CLOG của các chương trước để trả lời "ai thấy gì".
