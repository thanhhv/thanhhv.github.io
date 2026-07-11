+++
title = "Chương 6 — Transaction Engine: MVCC, Snapshot, Visibility"
date = "2026-07-11T13:00:00+07:00"
draft = false
tags = ["backend", "database", "postgresql"]
series = ["PostgreSQL Internals"]
+++

> Level 3. MVCC là quyết định thiết kế trung tâm của PostgreSQL. Hiểu nó là hiểu được:
> tại sao reader không block writer, tại sao có dead tuple, tại sao có VACUUM,
> tại sao "idle in transaction" là kẻ giết hệ thống thầm lặng.

---

## 1. Problem Statement

1.000 transaction đồng thời trên cùng bảng. Yêu cầu Isolation: mỗi transaction thấy database như một ảnh chụp nhất quán, không thấy thay đổi dở dang của người khác.

Cách tiếp cận cổ điển — **Two-Phase Locking (2PL)**: đọc lấy shared lock, ghi lấy exclusive lock, giữ tới hết transaction. Đúng đắn, nhưng: **reader block writer, writer block reader**. Một báo cáo chạy 10 phút sẽ chặn mọi UPDATE vào các row nó đọc. OLTP + báo cáo không sống chung được.

**MVCC (Multi-Version Concurrency Control)** giải khác: đừng ghi đè — **giữ nhiều version của mỗi row**. Writer tạo version mới; reader đọc version cũ phù hợp với "thời điểm" của nó. Reader không bao giờ chờ writer, writer không bao giờ chờ reader (writer vẫn chờ writer trên cùng row).

**Nếu bỏ MVCC dùng lock thuần?** Nhìn các hệ đã từng: DB2, SQL Server trước 2005 — lock escalation, deadlock giữa reader và writer, và cả một nghề "hint NOLOCK" chấp nhận đọc dữ liệu rác để sống. MVCC mua sự trong sạch đó bằng: **dữ liệu rác tích lũy (dead tuple) và bộ máy dọn rác (VACUUM)**.

## 2. Nguyên liệu — đã gặp ở Chương 3, giờ ghép lại

Mỗi tuple mang: `xmin` (XID tạo), `xmax` (XID xóa/thay thế, 0 nếu chưa), `ctid` (trỏ version kế). Mỗi transaction ghi có một **XID** — số nguyên 32 bit cấp tăng dần. Trạng thái mỗi XID (in-progress / committed / aborted) nằm trong **CLOG** (`pg_xact`, 2 bit/XID, cache qua SLRU).

Chú ý hai tinh tế:

- **Transaction chỉ-đọc không tốn XID** (PG lười cấp XID tới khi ghi lần đầu) — nhờ vậy workload đọc nặng không đốt không gian XID.
- **XID 32 bit và so sánh vòng tròn (modulo 2³¹)** — "quá khứ" và "tương lai" là tương đối. Đây là mầm của bài toán **wraparound** (Chương 10): nếu để một XID già hơn ~2 tỷ transaction so với hiện tại, phép so sánh lật ngược — quá khứ biến thành tương lai, dữ liệu committed "biến mất". PostgreSQL ngăn bằng FREEZE. Ghi nhớ từ bây giờ.

## 3. Snapshot — "thời điểm" của transaction là gì

Snapshot không phải bản chụp dữ liệu. Nó chỉ là **một bộ ba đủ để phân loại mọi XID**:

```
Snapshot {
    xmin    : XID nhỏ nhất còn đang chạy lúc chụp
              → mọi XID < xmin: đã KẾT THÚC (tra CLOG biết commit/abort)
    xmax    : XID tiếp theo sẽ cấp lúc chụp
              → mọi XID ≥ xmax: TƯƠNG LAI (vô hình, kể cả nếu nó commit ngay sau đó)
    xip[]   : danh sách XID ĐANG CHẠY lúc chụp
              → vô hình với snapshot này dù sau đó nó commit
}
```

Chụp snapshot = quét ProcArray (Chương 2) — chi phí O(số connection), lý do pooler tồn tại.

```
        trục XID ─────────────────────────────────────────▶
        ... 95   96   97   98   99   100  101  102  103 ...
                  ▲              ▲              ▲
   snapshot{xmin=97, xip=[97,100], xmax=102}
             
   XID 95,96      : quá khứ đã kết thúc → thấy nếu committed
   XID 97,100     : đang chạy lúc chụp  → KHÔNG thấy (kể cả commit sau đó)
   XID 98,99,101  : kết thúc trước khi chụp → thấy nếu committed
   XID ≥ 102      : tương lai → KHÔNG thấy
```

### Isolation level = tần suất chụp snapshot

| Level | Chụp snapshot khi nào | Hiện tượng còn lại |
|---|---|---|
| READ COMMITTED (mặc định) | **Mỗi statement** | Non-repeatable read, phantom; hai SELECT trong cùng transaction có thể thấy khác nhau |
| REPEATABLE READ | **Một lần đầu transaction** | Snapshot đóng băng toàn transaction; UPDATE đụng row đã bị kẻ khác sửa → lỗi serialization, phải retry |
| SERIALIZABLE | Như RR + **SSI** (Serializable Snapshot Isolation): theo dõi dependency đọc-ghi bằng predicate lock (SIRead), abort giao dịch tạo chu trình nguy hiểm | Không còn anomaly nào; giá: CPU/memory theo dõi + tỉ lệ retry |

Điểm hay bị hiểu sai ở READ COMMITTED: khi UPDATE gặp row đang bị transaction khác sửa, nó **chờ**; nếu kẻ kia commit, nó **đọc lại version mới nhất** (EvalPlanQual) và kiểm tra lại WHERE — không phải "atomic snapshot" như nhiều người tưởng. Hệ quả kinh điển: hai lệnh `UPDATE counter SET n = n + 1` đồng thời vẫn đúng (ra +2), nhưng logic phức tạp hơn dạng read-modify-write bằng hai câu SQL riêng thì **không** — phải dùng `SELECT ... FOR UPDATE` hoặc RR/SERIALIZABLE.

## 4. Visibility check — thuật toán trung tâm

Mọi lần đọc tuple đều chạy hàm này (rút gọn từ `HeapTupleSatisfiesMVCC`, src/backend/access/heap/heapam_visibility.c):

```go
func Visible(t Tuple, s Snapshot, myXID XID) bool {
    // ── Phần 1: người TẠO tuple (xmin) ──────────────────────
    switch {
    case t.xmin == myXID:
        // chính mình tạo → thấy, nếu tạo ở command trước (kiểm tra t_cid
        // với CommandId hiện tại — để "UPDATE không thấy row chính nó vừa chèn")
        if t.cid >= s.curcid { return false }
    case s.InProgress(t.xmin):        // đang chạy lúc chụp snapshot
        return false
    case clog.State(t.xmin) != COMMITTED:  // aborted / crashed
        return false                  // ← đây là cách ROLLBACK "hoạt động":
                                      //   không cần undo, tuple tự vô hình
    case t.xmin >= s.xmax:            // commit sau khi chụp
        return false
    }
    // xmin hợp lệ → tuple ĐÃ TỒN TẠI với snapshot này. Nó còn sống không?

    // ── Phần 2: người XÓA tuple (xmax) ──────────────────────
    if t.xmax == 0 { return true }    // chưa ai xóa
    if t.xmax == myXID {              // chính mình xóa
        return t.cidXmax >= s.curcid  // command sau mới coi là đã xóa
    }
    if s.InProgress(t.xmax) { return true }  // kẻ xóa chưa xong → vẫn thấy
    if clog.State(t.xmax) != COMMITTED { return true } // kẻ xóa rollback
    if t.xmax >= s.xmax { return true }      // xóa "trong tương lai" của snapshot
    return false                       // đã bị xóa thật sự trước snapshot
}
```

(Bản thật còn tầng hint bit: trước khi tra CLOG, kiểm tra `t_infomask`; tra xong thì set hint bit — Chương 3 mục 4.)

Đọc kỹ hàm này, bạn nhận ra ba điều lớn:

1. **ROLLBACK là O(1).** Không undo gì cả — chỉ ghi 2 bit ABORTED vào CLOG. Mọi tuple của transaction đó tự động vô hình từ đây về sau. So với InnoDB phải chạy undo log (rollback transaction lớn có thể mất hàng giờ), đây là ưu thế thật của PostgreSQL.
2. **DELETE không xóa gì, UPDATE không sửa gì.** Chỉ set xmax / chèn version mới. Dữ liệu "bị xóa" nằm nguyên trên disk tới khi VACUUM dọn — hệ quả bảo mật (xóa dữ liệu nhạy cảm ≠ dữ liệu biến mất khỏi disk) và hệ quả dung lượng (bloat).
3. **Một tuple là "dead" hay không phụ thuộc người hỏi.** Tuple bị xóa bởi XID 100 vẫn "sống" với snapshot chụp trước khi 100 commit. → Định nghĩa chính xác: *dead tuple = tuple không còn visible với BẤT KỲ snapshot hiện tại và tương lai nào*. Ngưỡng đó gọi là **xmin horizon** — min(xmin của mọi snapshot đang tồn tại trên hệ thống, kể cả của replica nếu bật hot_standby_feedback).

## 5. xmin horizon — khái niệm production quan trọng nhất chương này

```
   horizon = min(snapshot.xmin của MỌI backend + replication slot + prepared xact)

   ──────────────────────────── trục XID ───────────────────────────▶
        │ tuple chết trước đây      │ tuple chết sau đây
        │ → VACUUM DỌN ĐƯỢC         │ → VACUUM PHẢI GIỮ LẠI
        └───────────── horizon ─────┘
```

**Một transaction (hoặc cursor, hoặc replication slot) giữ snapshot già sẽ ghim horizon lại** → mọi dead tuple sinh sau đó trên **toàn cluster** (không chỉ bảng nó đọc!) không dọn được → bloat tăng, HOT chain dài, index phình — dù autovacuum chạy hùng hục "0 dead tuples removed".

Đây là root cause của cả một họ sự cố production với cùng một chữ ký:

```sql
-- Ba nghi phạm kinh điển khi bloat tăng mà vacuum "chạy mà không dọn được":
SELECT pid, state, xact_start, backend_xmin FROM pg_stat_activity
 WHERE backend_xmin IS NOT NULL ORDER BY age(backend_xmin) DESC;  -- (1) txn/idle-in-txn già
SELECT slot_name, xmin, catalog_xmin FROM pg_replication_slots;    -- (2) slot bỏ rơi
SELECT gid, prepared FROM pg_prepared_xacts;                       -- (3) 2PC bị quên
```

Phòng tránh hàng đầu: `idle_in_transaction_session_timeout`, `statement_timeout`, và (PG14+) `old_snapshot_threshold`/`transaction_timeout` (PG17) tùy phiên bản.

## 6. Bên dưới lớp vật lý — CLOG, subtransaction, MultiXact

- **CLOG (pg_xact):** mảng 2-bit vô tận đánh số theo XID, chia file 256KB, cache SLRU. 1 triệu transaction ≈ 256KB — rẻ. Được WAL bảo vệ, truncate phần già hơn `datfrozenxid` sau freeze.
- **Subtransaction (SAVEPOINT, EXCEPTION trong PL/pgSQL):** mỗi savepoint cấp sub-XID, ánh xạ sub→parent nằm trong `pg_subtrans` (SLRU). Quá **64 sub-XID/transaction** → tràn cache trong ProcArray (`suboverflowed`) → mọi visibility check phải tra pg_subtrans → chậm **toàn hệ thống**, đặc biệt trên replica. Đây là sự cố nổi tiếng ở nhiều công ty lớn (thường do ORM bọc mỗi statement trong SAVEPOINT). Nhận diện: PG16+ `pg_stat_slru`, wait event `SubtransSLRU`.
- **MultiXact:** khi NHIỀU transaction cùng giữ lock trên một row (FOR SHARE, hoặc FK check), xmax không chứa nổi nhiều XID → trỏ tới một **MultiXactId** trong `pg_multixact`. MultiXact cũng 32 bit, cũng cần freeze, cũng wraparound được — và ít người giám sát (`mxid_age(relminmxid)`). FK nhiều + update nóng = MultiXact tăng nhanh.

## 7. Trade-off: MVCC kiểu PostgreSQL vs kiểu Oracle/InnoDB

|  | PostgreSQL: version trong heap | Oracle/InnoDB: undo segment riêng |
|---|---|---|
| UPDATE | Chèn version mới cạnh version cũ | Ghi đè in-place, version cũ chép vào undo |
| ROLLBACK | O(1) — flip bit CLOG | Chạy ngược undo — đắt với transaction lớn |
| Reader cần version cũ | Đọc ngay trong heap | Dựng lại từ undo chain — "ORA-01555 snapshot too old" khi undo bị recycle |
| Rác nằm ở | Heap + index (bloat, cần VACUUM) | Undo segment (tự recycle, gọn hơn) |
| Transaction dài làm hại | Ghim horizon → bloat toàn cluster | Ngốn undo → lỗi query đọc, nhưng bảng không phình |
| Index khi row đổi version | Phải thêm entry (trừ HOT) | Không đổi (trỏ PK) |

Không ai thắng: PostgreSQL đau mạn tính (bloat/vacuum, quản được bằng kỷ luật vận hành), Oracle/InnoDB đau cấp tính (rollback lâu, snapshot too old). Chọn PostgreSQL nghĩa là **chấp nhận nghề vacuum** — Chương 10.

## 8. Production

- **Ba timeout bắt buộc có:** `idle_in_transaction_session_timeout` (vd 60s), `statement_timeout` (theo SLA từng service), `lock_timeout` (cho DDL). Thiếu bộ ba này, một client treo có thể ghim horizon cả ngày.
- **Giám sát horizon:** `max(age(backend_xmin))` từ pg_stat_activity, `age(datfrozenxid)` từng database (cảnh báo ở 200M, hành động ở 500M — mặc định autovacuum ép freeze ở `autovacuum_freeze_max_age`=200M).
- **Retry cho RR/SERIALIZABLE:** mã lỗi 40001/40P01 là "hãy thử lại", không phải bug — application phải có retry loop.
- **Đừng đếm bằng `SELECT count(*)` nóng:** count phải quét visibility từng tuple (không đọc được từ index thuần trước index-only scan, và vẫn phụ thuộc VM). Bảng lớn → dùng ước lượng `pg_class.reltuples` hoặc đếm tăng dần.

## 9. Failure case của chương: "database chậm dần mỗi chiều thứ Hai"

**Triệu chứng:** mỗi sáng thứ Hai chạy job export dữ liệu (transaction đọc 4 giờ). Từ trưa, UPDATE chậm dần, bloat các bảng nóng tăng, autovacuum log "removed 0 dead tuples".

**Bên trong:** transaction export giữ snapshot với xmin cũ → horizon ghim 4 giờ → hàng triệu dead tuple của workload OLTP sáng thứ Hai không dọn được → HOT chain dài ra, mỗi index lookup lội qua nhiều version chết → CPU tăng, latency tăng. Sau khi export xong, vacuum dọn được nhưng bảng đã phình — và heap không tự co (Chương 3).

**Điều tra:** truy vấn (1)(2)(3) ở mục 5; `pg_stat_user_tables.n_dead_tup` tăng trong khi `last_autovacuum` gần đây → chữ ký của horizon bị ghim.

**Khắc phục & phòng tránh:** chạy export trên replica (không bật `hot_standby_feedback`, chấp nhận query bị cancel do conflict — hoặc dùng snapshot export + chia nhỏ); hoặc chia export thành nhiều transaction ngắn; đặt `idle_in_transaction_session_timeout` cho user thường, whitelist riêng cho job.

## 10. Tóm tắt

- MVCC: writer tạo version mới, reader đọc version cũ theo snapshot — không ai block ai (giữa reader và writer).
- Snapshot = (xmin, xmax, xip); visibility = hàm thuần trên (tuple.xmin/xmax, snapshot, CLOG).
- ROLLBACK là flip-bit; DELETE/UPDATE không xóa gì — rác chờ VACUUM.
- xmin horizon là khái niệm vận hành số một: một snapshot già ghim rác của toàn cluster.
- XID 32-bit so sánh vòng tròn → freeze/wraparound là món nợ phải trả định kỳ (Chương 10).
- Subtransaction >64 và MultiXact là hai bãi mìn ít người biết.

**Tiếp theo:** [Chương 7 — Lock Manager](/series/postgres-internal/07-lock-manager/): phần "writer chờ writer" mà MVCC không xử lý.
