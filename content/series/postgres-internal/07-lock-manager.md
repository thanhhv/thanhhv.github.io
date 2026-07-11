+++
title = "Chương 7 — Lock Manager"
date = "2026-07-11T14:00:00+07:00"
draft = false
tags = ["backend", "database", "postgresql"]
series = ["PostgreSQL Internals"]
+++

> Level 3. MVCC loại bỏ xung đột reader–writer. Phần còn lại — writer–writer,
> DDL, và bảo vệ cấu trúc trong bộ nhớ — thuộc về ba tầng lock của PostgreSQL.
> Đa số sự cố "database treo" trong production là sự cố lock, không phải sự cố I/O.

---

## 1. Problem Statement

Ba lớp bài toán tranh chấp khác nhau về bản chất và thang thời gian:

1. **Micro (nanô giây):** hai CPU cùng sửa một biến trong shared memory (refcount của buffer). Cần loại trừ trong vài chục chu kỳ CPU.
2. **Meso (micro giây):** một backend sửa nội dung page trong khi backend khác đọc. Cần read/write lock ngắn.
3. **Macro (mili giây → giờ):** hai transaction cùng UPDATE một row; ALTER TABLE trong khi query đang chạy. Cần lock theo ngữ nghĩa transaction: có hàng đợi, có phát hiện deadlock, tự nhả khi commit.

Một cơ chế duy nhất không phục vụ nổi cả ba (lock có hàng đợi + deadlock detection quá đắt cho việc tăng một biến đếm). PostgreSQL xây ba tầng:

```
 Tầng        Cơ chế                Giữ trong      Có deadlock detection?
 ─────────────────────────────────────────────────────────────────────
 Micro       Spinlock (+atomic)    ~ns            Không (không được phép chờ lâu)
 Meso        LWLock                ~µs            Không (thứ tự cấp phát kỷ luật)
 Macro       Heavyweight Lock      đến hết txn    CÓ (deadlock_timeout)
             + Row Lock (trên tuple)
```

## 2. Spinlock & Atomic — tầng đáy

Spinlock = một biến + vòng lặp test-and-set (compare-and-swap của CPU). Không nhường CPU ngay; xoay tại chỗ vì kỳ vọng chủ lock chỉ giữ vài chục ns. Dùng bảo vệ: con trỏ head của freelist, vài counter... Ngày nay nhiều chỗ đã thay bằng **atomic operation thuần** (như state của buffer descriptor — Chương 4).

```go
// bản chất spinlock
for !atomic.CompareAndSwapInt32(&s.lock, 0, 1) {
    spinDelay() // pause; sau ~1000 lần xoay hụt → sleep tăng dần
}
// ... vùng critical vài chục ns ...
atomic.StoreInt32(&s.lock, 0)
```

Điều đáng nhớ với người vận hành: spinlock tranh chấp nặng biểu hiện thành **CPU cao mà throughput không tăng** (mọi core bận xoay). Hồ sơ perf thấy `s_lock` chiếm cao → tìm điểm nóng cấu trúc (thường là SLRU/ProcArray trên hệ connection khổng lồ).

## 3. LWLock — bảo vệ cấu trúc shared memory

LWLock (lightweight lock) = read/write lock với hai chế độ SHARED/EXCLUSIVE, có hàng đợi ngủ (không xoay mãi), **không có deadlock detection** — an toàn nhờ kỷ luật code: giữ cực ngắn, cấp phát theo thứ tự cố định, không chờ I/O tùy tiện khi đang giữ (trừ vài chỗ có chủ đích như WALWriteLock).

Các LWLock bạn sẽ gặp tên trong `pg_stat_activity.wait_event`:

| LWLock | Bảo vệ | Khi thấy nhiều nghĩa là |
|---|---|---|
| `BufferContent` | Nội dung một page | Page nóng (counter row, index root split) — hot spot dữ liệu |
| `BufferMapping` | Hash table buffer | Cache churn dữ dội |
| `WALInsert` (×8) | Giành chỗ trong WAL buffer | Ghi WAL quá nhiều/quá dồn |
| `WALWrite` | Flush WAL xuống disk | fsync chậm, disk WAL nghẽn |
| `ProcArray` | Danh sách backend/XID | Quá nhiều connection, snapshot dồn dập |
| `LockManager` (×16) | Bảng heavyweight lock | Bão lock (nhiều partition lock/giây) |
| `SubtransSLRU`... | Cache SLRU | Subtransaction overflow (Chương 6) |

**Tại sao không dùng luôn heavyweight lock cho các cấu trúc này?** Chi phí: heavyweight lock cần ghi vào bảng lock trong shared memory + hỗ trợ hàng đợi phức tạp + deadlock detection định kỳ — đắt hơn LWLock hàng chục lần. LWLock đổi tính năng lấy tốc độ, và trả bằng kỷ luật lập trình.

## 4. Heavyweight Lock — lock theo ngữ nghĩa transaction

### 4.1. Ma trận 8 mức trên relation

Mọi lệnh SQL lấy lock **cấp bảng** ở một trong 8 mức. Điều quan trọng duy nhất cần thuộc: **ai xung đột với ai**:

```
                                  ┌─ tự tương thích với chính nó?
 1 AccessShare      SELECT                        ✔  chỉ đụng #8
 2 RowShare         SELECT FOR UPDATE/SHARE       ✔  đụng #7,#8
 3 RowExclusive     INSERT/UPDATE/DELETE          ✔  đụng #5..#8
 4 ShareUpdateExcl  VACUUM, ANALYZE, CREATE INDEX CONCURRENTLY,
                    ALTER TABLE (một số dạng)     ✘  đụng #4..#8
 5 Share            CREATE INDEX (không concurrent)✔* đụng #3,#4,#6..#8
 6 ShareRowExcl     một số ALTER, triggers        ✘  đụng #3..#8
 7 Exclusive        REFRESH MAT VIEW CONCURRENTLY ✘  đụng #2..#8
 8 AccessExclusive  DROP, TRUNCATE, VACUUM FULL,
                    đa số ALTER TABLE, LOCK TABLE ✘  đụng TẤT CẢ, kể cả SELECT
```

Hai định luật vận hành rút ra:

**Định luật 1 — DML thường không cản nhau ở cấp bảng.** SELECT/INSERT/UPDATE/DELETE (mức 1–3) tương thích hết với nhau. Tranh chấp thật xảy ra ở cấp **row**.

**Định luật 2 — hàng đợi lock là FIFO, và đây là cái bẫy chết người của DDL.**

```
 10:00:00  Query báo cáo (AccessShare) chạy, còn 10 phút
 10:00:05  ALTER TABLE ... (AccessExclusive) đến → XẾP HÀNG chờ báo cáo
 10:00:06  MỌI câu SELECT/INSERT mới đến → xếp hàng SAU ALTER
           (dù bản thân chúng tương thích với báo cáo đang chạy!)
 → Toàn bộ traffic vào bảng đứng hình 10 phút. "Database treo" mà
   CPU ~0%, I/O ~0%. Nguyên nhân: MỘT câu ALTER không có lock_timeout.
```

Phòng tránh: luôn `SET lock_timeout = '3s'` (và retry) trước mọi DDL; dùng các biến thể không cần AccessExclusive lâu (`CREATE INDEX CONCURRENTLY`, `ALTER TABLE ... ADD COLUMN` không rewrite từ PG11, `ATTACH PARTITION CONCURRENTLY`...).

### 4.2. Row lock — nằm trong tuple, không nằm trong RAM

Điểm thiết kế xuất sắc ít người để ý: PostgreSQL **không có bảng lock trong RAM cho từng row**. Lock một row = ghi XID của mình vào `t_xmax` của tuple (+ cờ trong infomask nói "đây là lock, không phải delete"). Nhiều kẻ cùng lock share một row → MultiXact (Chương 6).

**Nếu làm kiểu khác (bảng row-lock trong RAM như SQL Server):** UPDATE 10 triệu row = 10 triệu entry lock → tràn bộ nhớ lock → **lock escalation** (leo thang thành lock cả bảng) → nghẽn kiểu khác. PostgreSQL không bao giờ escalate row lock; giá phải trả: chờ row lock được thực hiện gián tiếp — kẻ đến sau chờ trên... **heavyweight lock của XID kẻ đến trước** (mỗi transaction giữ ExclusiveLock trên chính XID của nó; muốn chờ ai thì lock XID của kẻ đó). Tinh tế nhưng scale vô hạn theo số row.

4 mức row lock: `FOR KEY SHARE` < `FOR SHARE` < `FOR NO KEY UPDATE` (UPDATE thường) < `FOR UPDATE` (UPDATE cột key/DELETE). FK constraint dùng KEY SHARE — lý do update bảng cha bị chậm bởi insert ồ ạt vào bảng con.

### 4.3. Deadlock detection

PostgreSQL không ngăn deadlock (quá đắt) — nó **phát hiện và xử tử**. Backend chờ lock quá `deadlock_timeout` (mặc định 1s) sẽ chạy thuật toán tìm chu trình trên đồ thị chờ (wait-for graph); tìm thấy → abort **chính transaction phát hiện** (không phải "nạn nhân trẻ nhất" như SQL Server):

```
  T1: UPDATE accounts WHERE id=1;  ... UPDATE accounts WHERE id=2;
  T2: UPDATE accounts WHERE id=2;  ... UPDATE accounts WHERE id=1;

        T1 ──chờ row 2 (do T2 giữ)──▶ T2
        ▲                              │
        └────chờ row 1 (do T1 giữ)─────┘   → chu trình → ERROR 40P01
```

Phòng deadlock ở tầng ứng dụng chỉ có một quy tắc: **mọi code path chạm nhiều row/bảng phải chạm theo cùng một thứ tự toàn cục** (vd luôn ORDER BY id khi SELECT FOR UPDATE nhiều row). Và luôn có retry cho 40P01.

### 4.4. Predicate lock (SIRead) — riêng cho SERIALIZABLE

SERIALIZABLE cần biết "transaction A đã đọc vùng dữ liệu mà B sau đó ghi vào". Lock SIRead ghi nhận *vùng đã đọc* (tuple/page/relation, tự leo thang để tiết kiệm), **không chặn ai** — chỉ để dò chu trình rw-antidependency. Chi phí RAM: `max_pred_locks_per_transaction`. Query đọc rộng dưới SERIALIZABLE → SIRead leo thang lên relation → tỉ lệ false-positive abort tăng. Đó là giá của serializability thật.

## 5. Cách hoạt động — một UPDATE giành lock từng bước

```
UPDATE accounts SET balance=... WHERE id=7;

 1. Heavyweight: RowExclusiveLock trên bảng + index (fastpath — xem dưới)
 2. Tìm tuple qua index → pin buffer, LWLock BufferContent (SHARE) để đọc
 3. Tuple có xmax=0? 
    CÓ  → LWLock EXCLUSIVE, set xmax=myXID, ghi WAL, xong (row đã "khóa")
    KHÔNG, xmax=XID 205 đang chạy →
       a. nhả LWLock (không bao giờ ngủ khi đang giữ LWLock!)
       b. XactLockTableWait: xin ShareLock trên "transaction 205"
          → ngủ trong hàng đợi heavyweight lock
       c. sau deadlock_timeout=1s chưa được → chạy deadlock check
       d. T205 commit → lock của nó nhả → ta dậy, đọc lại tuple
          (EvalPlanQual nếu READ COMMITTED — Chương 6)
 4. Commit: nhả toàn bộ heavyweight lock. (Row lock "tự nhả" vì
    xmax=XID đã kết thúc — không cần dọn gì trong tuple!)
```

**Fast-path lock:** lấy RowExclusive/AccessShare trên bảng qua bảng lock chung tốn LWLock partition → nghẽn khi QPS cao. Tối ưu: mỗi backend có 16 slot fastpath riêng (PG18 mở rộng) cho lock "yếu" không xung đột — không đụng bảng chung. Query join nhiều bảng + nhiều index (>16 relation) tràn fastpath → thấy `LockManager` wait event tăng. Sửa: giảm số relation/partition chạm mỗi query, hoặc nâng PG.

## 6. Trade-off

| Quyết định | Được | Mất |
|---|---|---|
| Row lock trong tuple, không trong RAM | Không giới hạn số row lock, không escalation | Chờ row lock vòng qua XID lock (khó đọc khi debug); mỗi lần lock row = ghi page (dirty + WAL) |
| Deadlock detect chậm (1s) thay vì ngăn ngừa | Không tốn chi phí lúc bình thường | Deadlock tồn tại 1s mới bị phá; workload nhiều deadlock chịu penalty |
| 8 mức table lock hạt mịn | DML, vacuum, index build đồng thời tối đa | Ma trận phức tạp; DDL xếp hàng gây bão như 4.2 |
| LWLock không deadlock detection | Nhanh | Bug thứ tự lock trong core = treo cứng (hiếm, nhưng đã từng có CVE/fix) |

## 7. Production

- **Ba câu tra cứu thuộc lòng:**

```sql
-- Ai chặn ai (cây blocking):
SELECT pid, pg_blocking_pids(pid) AS blocked_by, wait_event_type, wait_event,
       state, query FROM pg_stat_activity WHERE cardinality(pg_blocking_pids(pid)) > 0;
-- Lock đang giữ/chờ:
SELECT locktype, relation::regclass, mode, granted, pid FROM pg_locks;
-- Bật log để hậu kiểm:
SET log_lock_waits = on;  -- log mọi lần chờ > deadlock_timeout
```

- **wait event là la bàn:** `pg_stat_activity.wait_event_type` = `Lock` (heavyweight — nhìn thấy tên đối tượng), `LWLock` (tranh chấp cấu trúc — nhìn tên lock đoán hệ nghẽn), `IO`, `Client`... Dashboard tốt = biểu đồ diện tích wait event theo thời gian (kiểu ASH của Oracle).
- **Chuẩn DDL an toàn:** `lock_timeout=3s` + retry; tách `ADD COLUMN` khỏi `SET DEFAULT` backfill; `CREATE INDEX CONCURRENTLY` (nhớ: fail để lại index INVALID phải drop tay); không bao giờ chạy migration lúc peak.
- **`NOWAIT` / `SKIP LOCKED`:** `SELECT ... FOR UPDATE SKIP LOCKED` là công cụ chuẩn cho job queue trên PostgreSQL — worker không xếp hàng chờ nhau.

## 8. Failure case của chương: bão lock từ một migration

**Triệu chứng:** 14:02, toàn bộ API timeout trong 40 giây. CPU database 5%. Sau đó tự hết. Log ứng dụng: connection pool cạn.

**Diễn biến bên trong:** migration `ALTER TABLE orders ADD CONSTRAINT ... FOREIGN KEY` (cần lock mạnh trên cả hai bảng, kèm full scan validate) đến lúc 14:02, xếp sau một SELECT analytics 40 giây; mọi query mới vào `orders` xếp sau nó (định luật 2); pool ứng dụng cạn vì mọi connection đều đang "active" chờ lock; wait event 100% `Lock:relation`.

**Điều tra sau sự cố:** log có `process ... still waiting for AccessExclusiveLock` (nhờ log_lock_waits); đối chiếu thời điểm migration deploy.

**Khắc phục lâu dài:** mọi migration qua khuôn: `SET lock_timeout='3s'` + retry n lần; FK thêm theo hai bước `NOT VALID` rồi `VALIDATE CONSTRAINT` (validate chỉ cần ShareUpdateExclusive — không chặn DML).

## 9. Tóm tắt

- Ba tầng: spinlock/atomic (ns, cấu trúc micro) → LWLock (µs, shared memory) → heavyweight (transaction, có deadlock detection).
- DML không cản nhau cấp bảng; kẻ giết hệ thống là DDL xếp hàng — luôn dùng lock_timeout.
- Row lock sống trong tuple (xmax) — scale vô hạn, không escalation, nhưng lock row = ghi disk.
- Deadlock: phát hiện sau 1s, abort kẻ phát hiện; phòng bằng thứ tự truy cập toàn cục + retry.
- wait_event trong pg_stat_activity là công cụ chẩn đoán số một của chương này.

**Tiếp theo:** [Chương 8 — Query Processing](/series/postgres-internal/08-query-processing/): từ chuỗi SQL đến các thao tác trên page mà ta đã hiểu.
