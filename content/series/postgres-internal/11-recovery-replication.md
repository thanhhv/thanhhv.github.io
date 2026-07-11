+++
title = "Chương 11 — Checkpoint, Recovery, Replication"
date = "2026-07-11T18:00:00+07:00"
draft = false
tags = ["backend", "database", "postgresql"]
series = ["PostgreSQL Internals"]
+++

> Level 3–5. Chương 5 hứa: "crash không mất gì, WAL dựng lại được". Chương này
> trả nghĩa vụ chứng minh — và mở rộng cùng cơ chế đó thành PITR và replication:
> ba tính năng, một nền tảng duy nhất là WAL replay.

---

## 1. Problem Statement

- **Crash recovery:** sau mất điện, dựng lại trạng thái đã commit — tự động, không thao tác tay, thời gian đoán được.
- **Backup/PITR:** "khôi phục về 14:59, ngay trước câu DELETE nhầm lúc 15:00" — backup định kỳ thuần túy không làm nổi.
- **Replication:** bản sao nóng, trễ mili giây, đọc được — và failover khi primary chết.

Cả ba là cùng một bài toán: **tái tạo trạng thái từ (một ảnh chụp cũ) + (dòng WAL)**. Ai kiểm soát điểm dừng replay, người đó có tính năng tương ứng: dừng ở cuối WAL = crash recovery; dừng ở timestamp = PITR; không bao giờ dừng = replica.

## 2. Checkpoint — mua trần cho thời gian recovery

**Vấn đề:** WAL replay từ đầu lịch sử là vô hạn. Cần mốc: "mọi thay đổi trước LSN này ĐÃ nằm trong data file — replay chỉ cần bắt đầu từ đây".

**Checkpoint làm gì, từng bước:**

```
1. Ghi nhận REDO point = LSN hiện tại lúc bắt đầu
2. Ghi TOÀN BỘ dirty page trong shared buffers xuống OS
   — rải đều trong checkpoint_completion_target × chu kỳ (mặc định 0.9)
   — đây là "write phase", kéo dài hàng phút BY DESIGN
3. fsync mọi file đã ghi ("sync phase" — điểm đau I/O thật sự)
4. Ghi checkpoint record vào WAL + cập nhật pg_control
   (file 8KB, chứa REDO point — nơi ĐẦU TIÊN recovery đọc)
5. WAL trước REDO point của checkpoint TRƯỚC ĐÓ → recycle/xóa
```

**Trade-off trung tâm (đã hẹn từ Chương 5):**

```
Checkpoint DÀY (max_wal_size nhỏ, timeout ngắn)   Checkpoint THƯA
+ recovery nhanh (ít WAL replay)                  − recovery lâu hơn
− I/O nền nặng liên tục                           + I/O nền êm
− FPW nhiều → WAL phình, commit latency xấu       + FPW ít
− cùng page nóng bị ghi disk lặp đi lặp lại       + page nóng gom nhiều thay đổi/lần ghi
```

Chuẩn thực chiến hệ ghi nặng: `checkpoint_timeout=15–30min`, `max_wal_size` đủ để **không bao giờ** checkpoint vì lý do dung lượng (theo dõi `checkpoints_req` vs `checkpoints_timed`), chấp nhận recovery vài phút. "Checkpoint storm" — Chương 12 case #6.

## 3. Crash Recovery — REDO từng bước

```
start → đọc pg_control:
  "state: in production" + không có shutdown checkpoint → CRASH! bắt đầu recovery
  │
  ▼
Startup process:
  1. lấy REDO point từ checkpoint record cuối
  2. đọc WAL tuần tự từ REDO point:
       với mỗi record: đọc page liên quan
         page.LSN ≥ record.LSN ? bỏ qua (đã có) : áp dụng redo
       gặp full-page-image: ghi đè nguyên page (chữa torn page)
  3. dừng khi: CRC sai / xl_prev đứt = đuôi thật của WAL
  4. mở cửa (trước cả khi checkpoint đầu tiên hoàn tất — PG replay xong là chạy)
```

Tính chất đáng nhớ:

- **Idempotent nhờ so sánh LSN** — crash TRONG LÚC recovery cũng không sao, chạy lại từ đầu.
- **Transaction dở dang không cần undo:** không có commit record trong WAL → CLOG không bao giờ đánh dấu committed → MVCC làm chúng vô hình vĩnh viễn. (Trọn vẹn lời hứa "PostgreSQL không cần undo" — Chương 1, 6.)
- **Thời gian recovery ≈ lượng WAL từ checkpoint cuối ÷ tốc độ replay.** Replay đơn luồng về áp dụng record (có prefetch I/O từ PG15, `recovery_prefetch`) — máy ghi 2GB WAL/5 phút có thể cần vài phút recovery. Con số này là RTO thực tế của bạn khi chưa có replica: **đo nó định kỳ bằng cách restart có chủ đích trên staging.**

## 4. Backup & PITR

### 4.1. pg_basebackup — backup ONLINE không cần dừng ghi

Điều kỳ diệu ít ai nghĩ kỹ: copy data directory **trong khi nó đang bị sửa** — bản copy chắc chắn rách (page nửa cũ nửa mới, file lệch thời điểm). Nhưng không sao! Vì:

```
backup = (ảnh data dir RÁCH, chụp từ t0→t1) + (WAL từ t0→t1)
restore = đặt ảnh rách xuống + replay WAL từ t0
        → cơ chế FPW + LSN chữa mọi vết rách, hệt như crash recovery
        (backup online = "một cú crash được lên kế hoạch")
```

### 4.2. PITR

Nền: base backup + **WAL archive liên tục** (`archive_command`/`pg_receivewal`). Khôi phục về 14:59:

```
restore base backup → recovery.signal + restore_command
recovery_target_time = '2026-07-11 14:59:00'
→ startup replay WAL, DỪNG trước transaction commit sau 14:59
→ chọn timeline mới (history file) — nhánh lịch sử mới, tránh lẫn WAL cũ/mới
```

**Điểm sống còn:** chuỗi WAL archive phải **liên tục không thủng lỗ** — thiếu một segment là dừng tại đó. Vì thế giám sát archiver (Chương 12 #21) là giám sát backup. Và **backup chưa restore thử = chưa có backup** — quy trình diễn tập restore định kỳ không phải hình thức, nó là chỗ duy nhất phát hiện lỗ hổng chuỗi WAL.

## 5. Streaming Replication

### 5.1. Physical replication — replica = crash recovery không bao giờ kết thúc

```
PRIMARY                                    REPLICA
 backend ghi WAL ──▶ walsender ──TCP──▶ walreceiver ──▶ ghi pg_wal
                                              │
                                     startup process replay liên tục
                                              │
                                     backend chỉ-đọc phục vụ SELECT
                                     (hot standby)
```

- Replica **giống hệt từng byte** (cùng major version, cùng kiến trúc). Replay rẻ hơn thực thi lại SQL rất nhiều.
- **Replication slot:** primary giữ WAL đến khi replica xác nhận nhận. An toàn (replica rớt mạng lâu vẫn nối lại được) — và nguy hiểm (slot bỏ rơi giữ WAL vô hạn → đầy disk, Chương 12 #5). Đặt `max_slot_wal_keep_size` làm cầu chì.
- **Đồng bộ hay không:** `synchronous_standby_names` + mức chờ (Chương 5). Sync = RPO 0, giá là commit latency cộng RTT và **primary đứng hình nếu standby sync chết** (dùng quorum `ANY 1 (s1,s2)` để tránh).

### 5.2. Xung đột trên replica — thứ làm mọi người ngạc nhiên

Replay là byte-level và **không nhân nhượng**: nếu SELECT trên replica đang cần tuple mà primary đã vacuum mất → WAL record vacuum tới replica → chọn: chờ query (lag tăng) hay giết query. Mặc định chờ tối đa `max_standby_streaming_delay=30s` rồi giết: `ERROR: canceling statement due to conflict with recovery`.

Ba lối thoát, ba cái giá:
1. `hot_standby_feedback=on`: replica báo xmin của nó về primary → primary **hoãn vacuum** → bloat lan từ replica về primary (nối lại Chương 6 mục 5!).
2. Tăng `max_standby_streaming_delay`: báo cáo chạy được, lag phình theo.
3. Replica riêng cho analytics (delay cho phép lớn) tách khỏi replica HA (delay 0).

Không có lựa chọn thứ tư "miễn phí" — đây là ví dụ đẹp về trade-off lan xuyên hệ thống.

### 5.3. Logical replication

Giải mã WAL thành dòng thay đổi logic (INSERT/UPDATE/DELETE từng row) → gửi subscriber áp dụng bằng SQL. Cho phép: khác version/kiến trúc, replicate một phần, **zero-downtime major upgrade** (giá trị lớn nhất). Giá: chậm hơn physical, DDL không tự replicate, sequence không đồng bộ, UPDATE cần REPLICA IDENTITY, decoder giữ `catalog_xmin` (lại một kẻ ghim horizon tiềm năng).

### 5.4. Failover — phần khó không nằm trong PostgreSQL

PostgreSQL cung cấp `pg_promote()`, **không** cung cấp: phát hiện chết đúng (chết thật hay ngắt mạng?), chống split-brain (hai primary cùng nhận ghi = thảm họa dữ liệu thật sự), fencing node cũ, chuyển hướng client. Đó là việc của Patroni + DCS (etcd/consul) hoặc dịch vụ managed. Tự viết script failover bằng cron + ssh là con đường ngắn nhất tới split-brain — bài học đã được ngành trả giá đủ nhiều lần.

Sau failover, node cũ muốn quay lại làm replica: `pg_rewind` (tua ngược các block đã ghi khác timeline, cần `wal_log_hints=on` hoặc data checksums) — nhanh hơn nhiều so với clone lại từ đầu.

## 6. Trade-off tổng của chương

| Quyết định | Được | Mất |
|---|---|---|
| Recovery = REDO-only từ checkpoint | Đơn giản, idempotent, không undo | Phải checkpoint đều; recovery time tỉ lệ WAL tích lũy |
| Physical replication byte-level | Rẻ, đúng tuyệt đối, lag ms | Cùng version/arch; replica full-cluster (không chọn bảng) |
| Replication slot | Không bao giờ thiếu WAL cho replica | Slot rò rỉ = bom disk hẹn giờ |
| hot_standby_feedback | Query replica không bị giết | Bloat primary — trade-off xuyên node |
| Failover ngoài core | Core đơn giản, đáng tin | Vận hành HA = vận hành thêm Patroni/etcd |

## 7. Production checklist

- `checkpoints_req` ≈ 0; `log_checkpoints=on` và đọc nó mỗi khi p99 có sóng.
- Đo recovery time thật trên staging mỗi quý — đó là RTO khi chưa kịp failover.
- Backup: pgBackRest/WAL-G thay vì tự chế; restore-drill có lịch; giám sát khoảng trống chuỗi WAL.
- Replication: alert lag theo **byte** (`pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn)`) và theo giây (`pg_last_xact_replay_timestamp`); alert slot inactive + `max_slot_wal_keep_size`.
- Failover: Patroni với `synchronous_mode` nếu RPO 0; diễn tập switchover định kỳ — switchover chưa từng tập là failover sẽ thất bại.

## 8. Failure case của chương: replication lag 4 giờ không rõ lý do

**Triệu chứng:** replica analytics lag tăng dần từ sáng, `walreceiver` vẫn nhận bình thường (network OK, WAL đến đủ).

**Chẩn đoán phân tầng — lag ở đâu trong ba chặng?**
```sql
-- trên replica:
SELECT pg_wal_lsn_diff(pg_last_wal_receive_lsn(), pg_last_wal_replay_lsn());
-- nhận đủ mà replay tụt → nghẽn ở APPLY, không phải mạng
```
`pg_stat_activity` trên replica: startup process wait_event = `Lock:relation` — **replay đang chờ một AccessShareLock**: một query báo cáo 4 giờ đang chạy, còn WAL chứa `DROP INDEX` từ primary cần AccessExclusive trên relation đó → replay dừng **toàn bộ** (một dòng WAL, áp dụng tuần tự!) chờ đến `max_standby_streaming_delay` (đã bị chỉnh thành -1 = chờ vô hạn "cho báo cáo khỏi chết").

**Bài học bên trong:** physical replay là **một hàng đợi tuần tự duy nhất** — một record bị chặn thì mọi record sau chờ theo, lag của cả replica là con tin của một query. **Khắc phục:** giết query báo cáo; đặt lại `max_standby_streaming_delay=5min`; tách replica báo cáo khỏi replica HA. **Phòng:** không bao giờ đặt -1 trên replica có vai trò HA; DDL nặng trên primary xếp lịch giờ thấp điểm.

## 9. Tóm tắt

- Checkpoint đặt trần cho recovery time; giá là I/O nền + FPW — giãn có kiểm soát, đừng để checkpoint vì "hết chỗ".
- Crash recovery, PITR, replication là một cơ chế duy nhất (REDO từ ảnh cũ) với ba điểm dừng khác nhau.
- Backup online "rách" là hợp lệ nhờ FPW + LSN; chuỗi WAL liên tục là mạch máu — giám sát archiver = giám sát backup.
- Replica vật lý = crash recovery bất tận, replay tuần tự đơn dòng — hiểu điều này là hiểu mọi loại lag.
- Failover an toàn cần tooling ngoài core; split-brain là lỗi đắt nhất có thể tự gây ra.

**Tiếp theo:** [Chương 12 — Production Failure Cases](/series/postgres-internal/12-production-failure-cases/): 21 hồ sơ sự cố, tổng ôn toàn bộ tài liệu.
