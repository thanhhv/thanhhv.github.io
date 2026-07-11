+++
title = "Chương 2 — Kiến trúc tổng thể: Process & Shared Memory"
date = "2026-07-11T09:00:00+07:00"
draft = false
tags = ["backend", "database", "postgresql"]
series = ["PostgreSQL Internals"]
+++

> Level 1–2. PostgreSQL là một "hệ điều hành thu nhỏ": có process quản lý, có bộ nhớ chia sẻ,
> có scheduler nền, có cơ chế phục hồi. Chương này vẽ bản đồ toàn bộ hệ thống đó.

---

## 1. Problem Statement

Một database server phải đồng thời: phục vụ hàng trăm connection, ghi dữ liệu nền, dọn rác, đồng bộ replica, thu thập thống kê — mà một thành phần chết không được kéo sập dữ liệu. Bài toán: **tổ chức các đơn vị thực thi (process/thread) và bộ nhớ chung như thế nào để vừa cô lập lỗi, vừa chia sẻ trạng thái hiệu quả?**

Nếu thiết kế sai tầng này, mọi thứ bên trên đều sai: một backend crash làm hỏng bộ nhớ chung → toàn bộ cluster phải giả định dữ liệu trong RAM đã hỏng.

## 2. Tại sao PostgreSQL chọn Process thay vì Thread

PostgreSQL dùng **một process cho mỗi connection** (process-per-connection), không phải thread. Quyết định này có từ thập niên 1980 (Berkeley POSTGRES) nhưng được giữ lại có chủ đích:

| Tiêu chí | Process model (PostgreSQL) | Thread model (MySQL, SQL Server) |
|---|---|---|
| Cô lập lỗi | Backend crash → chỉ process đó chết; postmaster phát hiện, reset shared memory một cách có kiểm soát | Thread crash có thể làm hỏng toàn bộ address space |
| Memory bug | Con trỏ lạc chỉ phá được process riêng + vùng shared memory (được checksum/kiểm soát) | Con trỏ lạc phá được mọi thứ |
| Chi phí tạo | Đắt (~vài ms fork + init) | Rẻ (~µs) |
| Chi phí mỗi connection | ~1–2 MB (page table, stack) + catalog cache riêng, có thể lên hàng chục MB | Nhẹ hơn đáng kể |
| Context switch | Đắt hơn (TLB flush) | Rẻ hơn |

**Nếu bỏ process model đi thì sao?** Cộng đồng PostgreSQL đang thảo luận nghiêm túc việc chuyển sang thread (thread mailing list 2023+), vì chi phí connection là điểm đau lớn nhất. Nhưng cái giá chuyển đổi khổng lồ: hàng nghìn biến global trong source code C phải biến thành thread-local.

**Hệ quả production quan trọng nhất:** connection PostgreSQL **đắt**. 500 connection idle vẫn tốn RAM và làm chậm snapshot computation (xem ProcArray bên dưới). Đây là lý do **connection pooler (PgBouncer) gần như bắt buộc** ở quy mô lớn — và là failure case #8 ở Chương 12.

## 3. Internal Architecture — cây process

```
postmaster (PID 1 của cluster)
│   • listen socket, fork mọi process con
│   • KHÔNG chạm shared memory data (để sống sót khi con chết)
│
├── backend process ×N        ← mỗi client connection một process
├── checkpointer              ← ghi toàn bộ dirty page theo chu kỳ
├── background writer         ← ghi dần dirty page, "dọn đường" cho backend
├── walwriter                 ← flush WAL buffer định kỳ
├── autovacuum launcher
│   └── autovacuum worker ×M  ← dọn dead tuple (Chương 10)
├── startup process           ← CHỈ khi khởi động: replay WAL (recovery)
├── archiver                  ← copy WAL segment đã đầy đi lưu trữ
├── walsender ×K              ← stream WAL cho replica
├── walreceiver               ← (trên replica) nhận WAL
├── logger                    ← thu log stderr của mọi process
└── (PG15+) không còn stats collector — thống kê chuyển vào shared memory
```

### Từng process: làm gì, khi nào chạy, chết thì sao

| Process | Làm gì | Khi nào chạy | Nếu chết | Restart? |
|---|---|---|---|---|
| **postmaster** | Nhận connection, fork con, giám sát | Luôn | Cluster ngừng nhận connection mới; các con được lệnh thoát | Do systemd/operator |
| **backend** | Thực thi query của 1 client | Khi có connection | Postmaster coi shared memory có thể hỏng → **kill toàn bộ backend, crash recovery** ("the database system is in recovery mode") | Tự động, mất vài giây–phút |
| **checkpointer** | Định kỳ ghi mọi dirty page xuống disk, cắt ngắn WAL cần replay | `checkpoint_timeout` (mặc định 5 phút) hoặc WAL vượt `max_wal_size` | Như backend chết: restart + recovery | Có |
| **background writer** | Ghi lai rai dirty page sắp bị evict để backend không phải tự ghi | Liên tục, nhịp `bgwriter_delay` (200ms) | Restart; hệ vẫn chạy nhưng backend tự ghi page nhiều hơn (latency tăng) | Có |
| **walwriter** | Flush WAL buffer mỗi `wal_writer_delay` (200ms); phục vụ `synchronous_commit=off` | Liên tục | Restart; commit async có thể trễ hơn | Có |
| **autovacuum launcher/worker** | Khởi động worker vacuum bảng vượt ngưỡng | Mỗi `autovacuum_naptime` (1 phút) | Restart. Nếu tắt hẳn lâu ngày → bloat, wraparound (Chương 12, case #2, #4) | Có |
| **startup** | Replay WAL từ checkpoint gần nhất | Chỉ lúc khởi động sau crash / trên replica (chạy mãi) | Trên replica: replica ngừng apply → lag | Có |
| **archiver** | Chạy `archive_command` cho từng WAL segment đầy | Khi có segment cần archive | Segment không được archive → **pg_wal đầy dần** (Chương 12, case #21) | Có |
| **logger** | Gom stderr ra file log | Luôn | Mất log tạm thời | Có |

Nguyên tắc thiết kế đáng nhớ: **postmaster tối giản đến mức gần như không thể chết vì bug dữ liệu** — nó không đọc/ghi shared memory data structures. Nhờ vậy nó luôn còn sống để làm "người dọn dẹp" khi con chết.

**Tại sao một backend chết lại phải restart tất cả?** Vì backend chết (segfault/OOM kill) có thể đã cầm LWLock hoặc đang sửa dở một cấu trúc trong shared memory. Không có cách nào rẻ để chứng minh shared memory còn toàn vẹn → PostgreSQL chọn an toàn tuyệt đối: vứt toàn bộ shared memory, crash recovery từ WAL. Trade-off: vài giây downtime đổi lấy đảm bảo không-bao-giờ-chạy-trên-bộ-nhớ-hỏng.

## 4. Shared Memory — giải phẫu

Toàn bộ shared memory được cấp phát **một lần lúc khởi động** (mmap, huge pages nếu bật), kích thước cố định. Không malloc/free động — loại bỏ cả fragmentation lẫn lỗi hết bộ nhớ giữa chừng.

```
┌───────────────────────────────────────────────────────────────┐
│                     SHARED MEMORY REGION                      │
│                                                               │
│  ┌─────────────────────────────────────────────┐              │
│  │ SHARED BUFFERS  (shared_buffers, vd 8GB)    │  ~85–95%     │
│  │ Mảng N buffer × 8KB + mảng descriptor       │  tổng size   │
│  └─────────────────────────────────────────────┘              │
│  ┌──────────────────────┐ ┌───────────────────┐               │
│  │ WAL BUFFERS          │ │ CLOG/SLRU BUFFERS │               │
│  │ (wal_buffers, ~16MB) │ │ commit status...  │               │
│  └──────────────────────┘ └───────────────────┘               │
│  ┌──────────────────────┐ ┌───────────────────┐               │
│  │ PROC ARRAY           │ │ LOCK TABLE        │               │
│  │ trạng thái mọi       │ │ heavyweight locks │               │
│  │ backend + XID đang   │ │ (max_locks_per_   │               │
│  │ chạy → tính snapshot │ │  transaction)     │               │
│  └──────────────────────┘ └───────────────────┘               │
│  ┌──────────────────────────────────────────┐                 │
│  │ Khác: bgworker slots, replication slots, │                 │
│  │ pg_stat shared memory (PG15+), semaphores│                 │
│  └──────────────────────────────────────────┘                 │
└───────────────────────────────────────────────────────────────┘
```

### 4.1. Shared Buffers
Cache page 8KB của data file — trái tim của hệ thống, chiếm nguyên Chương 4.

### 4.2. WAL Buffers
Vùng đệm cho WAL record trước khi flush xuống `pg_wal/`. Nhỏ (mặc định 1/32 shared_buffers, tối đa thường 16MB là đủ) vì WAL được flush liên tục. Chi tiết Chương 5.

### 4.3. ProcArray — cấu trúc bị đánh giá thấp nhất

Mảng trạng thái của mọi backend: PID, XID đang chạy, xmin của snapshot đang giữ. Mỗi lần một transaction cần snapshot (mỗi statement trong READ COMMITTED!), nó phải **quét ProcArray** để biết những XID nào đang chạy.

```go
// Pseudo code: chụp snapshot (Chương 6 dùng lại cấu trúc này)
func GetSnapshot(procArray []PGPROC) Snapshot {
    s := Snapshot{Xmin: nextXID, Xmax: nextXID}
    // Phải giữ LWLock ProcArrayLock (shared) trong lúc quét
    for _, proc := range procArray {         // O(số connection)!
        if proc.xid.Valid() {
            s.XipList = append(s.XipList, proc.xid)
            if proc.xid < s.Xmin { s.Xmin = proc.xid }
        }
    }
    return s
}
```

**Hệ quả:** chi phí lấy snapshot tỉ lệ thuận số connection. 2.000 connection (kể cả idle) → mọi query trên hệ thống chậm đi vì `GetSnapshotData()` dài ra và ProcArrayLock bị tranh chấp. PG14 đã tối ưu đáng kể (caching, tách mảng xid riêng dense hơn), nhưng nguyên tắc "connection không miễn phí" vẫn đúng. Đây là root cause kỹ thuật thật sự đằng sau lời khuyên "dùng PgBouncer".

### 4.4. SLRU — Simple LRU, cache cho metadata dạng log

Họ cache nhỏ cho dữ liệu "mảng vô hạn được chia file": CLOG (trạng thái commit của từng XID — 2 bit/XID), commit timestamp, MultiXact, subtransaction... Truy cập qua buffer LRU đơn giản, tách khỏi shared buffers.

**Tại sao tách riêng?** Pattern truy cập khác hẳn: CLOG đọc theo XID (gần như tuần tự, locality cao quanh XID mới), page nhỏ, số lượng ít. Nhét vào shared buffers sẽ vừa phí cơ chế clock-sweep vừa bị page dữ liệu đè evict. Trade-off: SLRU từng là điểm nghẽn (kích thước cố định, tìm kiếm tuyến tính); PG17 đại tu (`commit_timestamp_buffers`, banks) sau nhiều sự cố production nổi tiếng liên quan subtransaction (xem Chương 12, case #9 mở rộng).

### 4.5. Lock Table
Bảng băm heavyweight lock trong shared memory — Chương 7.

## 5. Bên dưới lớp vật lý — bố cục thư mục dữ liệu

```
$PGDATA/
├── base/                    ← mỗi database một thư mục (theo OID)
│   └── 16384/               ← database "shop"
│       ├── 16385            ← bảng orders, segment 0 (tối đa 1GB)
│       ├── 16385.1          ← segment 1 (khi bảng > 1GB)
│       ├── 16385_fsm        ← Free Space Map fork
│       ├── 16385_vm         ← Visibility Map fork
│       └── ...
├── global/                  ← catalog chung cluster (pg_database...)
├── pg_wal/                  ← WAL segments (16MB mỗi file)
│   ├── 000000010000000A0000002F
│   └── ...
├── pg_xact/                 ← CLOG: trạng thái commit của XID
├── pg_multixact/            ← lock nhiều transaction trên 1 row
├── pg_tblspc/               ← symlink tới tablespace ngoài
├── postmaster.pid
└── postgresql.conf, pg_hba.conf
```

Chi tiết từng byte trong data file: Chương 3. Chi tiết pg_wal: Chương 5.

## 6. Data flow tổng hợp: một UPDATE đi qua các process nào

```
 client ──SQL──▶ backend ──(1) đọc page──▶ Shared Buffers ◀──(nếu miss: đọc từ disk)
                   │
                   ├─(2) sửa tuple trong Shared Buffers (page thành dirty)
                   ├─(3) ghi WAL record vào WAL Buffers
                   │
              COMMIT│
                   ├─(4) flush + fsync WAL ──────────▶ pg_wal/  ★ durable tại đây
                   │        (walwriter phụ giúp cho async commit)
                   ▼
             trả "COMMIT" cho client
                   
      ... sau đó, không đồng bộ với client ...

 background writer ──(5) ghi lai rai dirty page ──▶ OS cache ──▶ data file
 checkpointer      ──(6) mỗi 5 phút: ghi TẤT CẢ dirty page + fsync
                          → ghi checkpoint record vào WAL
                          → WAL trước điểm này không cần cho recovery nữa
 autovacuum        ──(7) dọn version tuple cũ khi không còn ai cần
```

Ba câu hỏi kiểm tra hiểu bài:

1. *Client nhận "COMMIT OK" — data file đã có dữ liệu mới chưa?* Chưa chắc, thường là chưa. Chỉ WAL chắc chắn có.
2. *Checkpointer chết ngay trước khi kịp chạy — mất gì không?* Không. WAL còn nguyên, recovery replay lại. Chỉ tốn thời gian recovery dài hơn.
3. *Tắt background writer có mất dữ liệu không?* Không — nó thuần túy là tối ưu latency. Backend sẽ tự ghi page khi cần evict (chậm hơn).

## 7. Trade-off

- **Fixed-size shared memory**: đơn giản, không fragmentation, nhưng đổi config lớn (shared_buffers) phải restart. Đối thủ (Oracle ASMM) cho resize online với cái giá phức tạp hơn nhiều.
- **Nhiều background process tách rời** thay vì một "super daemon": cô lập lỗi và schedule độc lập, nhưng nhiều điểm cấu hình hơn và các process phải phối hợp qua shared memory + signal (phức tạp khó debug hơn).
- **Postmaster không dùng shared memory data**: sống dai, nhưng nghĩa là mọi việc "thông minh" phải ủy quyền cho con — kể cả việc nhận biết con chết chỉ qua exit code + SIGCHLD.
- **PG15 bỏ stats collector (UDP) chuyển sang shared memory**: trước đây thống kê gửi qua UDP socket, có thể **mất mẫu** khi tải cao và tốn CPU; giờ ghi thẳng shared memory — nhanh, chính xác, nhưng thêm một consumer của memory chung. Đây là ví dụ tốt về việc kiến trúc PostgreSQL vẫn tiến hóa mạnh.

## 8. Production

- **Định cỡ ban đầu (điểm xuất phát, không phải chân lý):** `shared_buffers` = 25% RAM (Chương 4 giải thích tại sao không phải 80%); `max_connections` thấp nhất có thể + PgBouncer; `max_wal_size` đủ lớn để checkpoint do timeout chứ không do WAL đầy (Chương 5, 11).
- **Giám sát process:** `pg_stat_activity` (backend nào đang làm gì, `state = 'idle in transaction'` là cờ đỏ), `pg_stat_bgwriter`/`pg_stat_checkpointer` (PG17) — tỉ lệ buffer do backend tự ghi (`buffers_backend`) cao nghĩa là bgwriter/checkpoint tụt hậu.
- **OOM killer là kẻ thù số một trên Linux:** backend bị SIGKILL → toàn cluster restart. Đặt `vm.overcommit_memory=2`, để ý `work_mem × số query song song` mới là mức tiêu thụ thật (work_mem là **per sort/hash node**, không phải per connection).
- **Huge pages** (`huge_pages = on`): với shared_buffers hàng chục GB, tiết kiệm TLB miss và page table (500 connection × page table của 64GB shared memory là con số khổng lồ).

## 9. Failure case tiêu biểu của chương

**Triệu chứng:** Ứng dụng thỉnh thoảng văng hàng loạt lỗi `FATAL: the database system is in recovery mode` trong ~10 giây rồi tự hết.

**Root cause phổ biến nhất:** một backend bị Linux OOM killer giết (query dùng quá nhiều work_mem). Postmaster thấy con chết bất thường → giết toàn bộ backend, chạy crash recovery.

**Điều xảy ra bên trong:** postmaster nhận SIGCHLD với exit status bất thường → log `server process (PID n) was terminated by signal 9` → gửi SIGQUIT cho mọi con → startup process replay WAL từ checkpoint cuối → mở cửa lại.

**Điều tra:** (1) PostgreSQL log: tìm "terminated by signal"; (2) `dmesg | grep -i oom`: xác nhận OOM kill, xem RSS của process bị giết; (3) đối chiếu query đang chạy lúc đó qua log `log_min_duration_statement`.

**Khắc phục & phòng tránh:** giảm `work_mem` hoặc số connection; `vm.overcommit_memory=2`; cgroup memory limit cho PostgreSQL đúng cách; canh `max_parallel_workers` (mỗi worker cũng ăn work_mem).

## 10. Tóm tắt

- Process-per-connection: cô lập lỗi mạnh, connection đắt → pooler gần như bắt buộc.
- Shared memory cấp phát tĩnh một lần: shared buffers + WAL buffers + ProcArray + SLRU + lock table.
- ProcArray khiến chi phí snapshot tăng theo số connection — lý do kỹ thuật sâu của "đừng mở 2000 connection".
- Một backend chết bất thường = toàn cluster restart + crash recovery, by design.
- Commit chỉ phụ thuộc WAL; mọi việc ghi data file là bất đồng bộ, có thể trễ tùy ý.

**Tiếp theo:** [Chương 3 — Physical Storage](/series/postgres-internal/03-physical-storage/): phóng to vào từng byte của data file.
