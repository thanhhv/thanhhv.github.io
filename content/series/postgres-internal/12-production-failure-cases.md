+++
title = "Chương 12 — Production Failure Cases: 21 hồ sơ sự cố"
date = "2026-07-11T19:00:00+07:00"
draft = false
tags = ["backend", "database", "postgresql"]
series = ["PostgreSQL Internals"]
+++

> Level 5. Mỗi case theo khuôn: Triệu chứng → Root cause → Bên trong PostgreSQL →
> Điều tra (metric/log/lệnh) → Khắc phục → Phòng tránh. Các case tham chiếu ngược
> về chương lý thuyết tương ứng — chương này đồng thời là bài tổng ôn.

**Dashboard nền khuyến nghị** (dùng chung cho mọi case): wait events theo thời gian (pg_stat_activity), TPS + p99 latency, connections theo state, dead tuples + last_(auto)vacuum các bảng top, WAL bytes/s + checkpoint timeline, replication lag byte, disk %util + fsync latency, cache hit ratio, `max(age(datfrozenxid))`.

---

## #1 Table Bloat

- **Triệu chứng:** bảng phình nhiều lần kích thước dữ liệu thật; seq scan/aggregate chậm dần theo tuần; cache hit giảm (page loãng chiếm chỗ).
- **Root cause:** dead tuple sinh nhanh hơn dọn — autovacuum lười (scale_factor 20%), bị throttle, hoặc horizon bị ghim (case #9).
- **Bên trong:** Chương 3 §9, Chương 10 — UPDATE/DELETE để lại version chết; vacuum trả chỗ cho FSM nhưng file không co; high-water mark chỉ tăng.
- **Điều tra:** `pgstattuple` (dead% + free%); `pg_stat_user_tables` (n_dead_tup, n_tup_upd vs n_tup_hot_upd, last_autovacuum); log autovacuum (`dead but not yet removable`?).
- **Khắc phục:** pg_repack (online) / VACUUM FULL (cần downtime); sau đó sửa nguyên nhân sinh rác.
- **Phòng:** per-table scale_factor 0.01 cho bảng nóng; cost_delay=0; fillfactor cho HOT; partition để DELETE→DROP.

## #2 Index Bloat

- **Triệu chứng:** query dùng index chậm dần dù data không đổi; index to hơn cả bảng.
- **Root cause:** split không merge (Chương 9 §3.2) + churn update/delete; vacuum không kịp cho killed-hint phát huy.
- **Điều tra:** `pgstatindex` → `avg_leaf_density` <50%; `idx_tup_read >> idx_tup_fetch`.
- **Khắc phục:** `REINDEX INDEX CONCURRENTLY` (đủ disk tạm; theo dõi lỗi để không để lại index `_ccnew`/INVALID).
- **Phòng:** vacuum đủ nhanh; bớt index trên cột hay update; partition.

## #3 Autovacuum không chạy (hoặc chạy mà vô dụng)

- **Triệu chứng:** n_dead_tup tăng vô hạn; last_autovacuum NULL hoặc rất cũ; hoặc vacuum chạy liên tục mà dead không giảm.
- **Root cause** — soi đúng nhánh:
  1. Không được kích hoạt: threshold quá cao với bảng lớn; `autovacuum=off` (kiểm tra cả per-table reloptions!).
  2. Chạy không tới lượt: 3 worker mặc định bận bảng khổng lồ khác.
  3. Chạy nhưng bị throttle: cost_limit mặc định — tốc độ vài MB/s (Chương 10 §5.2).
  4. Chạy nhưng không dọn được: **horizon ghim** — xem case #9.
  5. Bị hủy liên tục: DDL/lock ưu tiên hơn vacuum thường (log: `canceling autovacuum task`).
- **Điều tra:** log_autovacuum_min_duration=0 rồi ĐỌC LOG — mọi nhánh trên đều hiện nguyên hình trong log vacuum; `pg_stat_progress_vacuum` cho tiến trình đang chạy.
- **Khắc phục/Phòng:** theo đúng nhánh; nguyên tắc Chương 10: vacuum nhanh-và-xong hơn dịu-dàng-mãi-mãi.

## #4 Transaction ID Wraparound

- **Triệu chứng giai đoạn 1:** autovacuum "to prevent wraparound" tự chạy trên bảng lớn, I/O tăng.
  **Giai đoạn 2:** `WARNING: database must be vacuumed within N transactions`.
  **Giai đoạn 3:** `ERROR: database is not accepting commands...` — cluster tự khóa ghi.
- **Root cause:** nợ freeze không trả (autovacuum tắt/kẹt/quá chậm trên bảng TB; hoặc chuỗi case #9 kéo dài nhiều tuần).
- **Bên trong:** Chương 6 §2 + Chương 10 §4 — so sánh vòng tròn XID; datfrozenxid không tiến.
- **Điều tra:** `SELECT datname, age(datfrozenxid) FROM pg_database ORDER BY 2 DESC;` rồi drill-down pg_class theo `age(relfrozenxid)`; tìm thứ chặn horizon/vacuum.
- **Khắc phục:** ưu tiên tuyệt đối cho vacuum bảng già nhất (tháo throttle); PG14+ thường không cần single-user mode — vacuum được trong chế độ khóa ghi; gỡ mọi thứ ghim horizon.
- **Phòng:** alert age hai nấc (500M / 1B); freeze chủ động giờ thấp; canh cả `mxid_age`.

## #5 WAL đầy disk (pg_wal phình)

- **Triệu chứng:** dung lượng pg_wal tăng không ngừng; đầy → `PANIC: could not write to file "pg_wal/..."` → sập.
- **Root cause — đúng 4 nghi phạm, kiểm theo thứ tự:**
  1. **Replication slot bỏ rơi** (`pg_replication_slots`: active=f, restart_lsn đứng im) — replica đã chết/dỡ bỏ nhưng slot còn.
  2. **Archiver fail** (case #21) — segment chưa archive không được recycle.
  3. `wal_keep_size`/`max_wal_size` chỉnh quá tay.
  4. Sản lượng WAL đột biến (REINDEX, bulk update, checkpoint dày sinh FPW) vượt tốc độ dọn.
- **Khắc phục:** xử lý đúng nghi phạm — `pg_drop_replication_slot()`, sửa archive_command; **tuyệt đối không rm file trong pg_wal** (mất mạch recovery = corrupt). Hết chỗ thật sự: mở rộng volume/di chuyển tạm bằng symlink đúng cách.
- **Phòng:** `max_slot_wal_keep_size`; alert %disk pg_wal RIÊNG (tách khỏi data); alert `pg_stat_archiver.failed_count`.

## #6 Checkpoint Storm

- **Triệu chứng:** p99 latency dâng thành sóng đều đặn; iostat thấy write burst; log: `checkpoints are occurring too frequently`.
- **Root cause:** max_wal_size quá nhỏ so với tốc độ ghi → checkpoint by-size liên tục; mỗi checkpoint reset FPW → WAL càng phình → càng nhanh chạm ngưỡng — vòng xoáy (Chương 5 §3.3, Chương 11 §2).
- **Điều tra:** `pg_stat_bgwriter/pg_stat_checkpointer`: checkpoints_req >> checkpoints_timed; log_checkpoints cho write/sync time; `pg_stat_wal.wal_fpi` tỉ lệ cao.
- **Khắc phục:** tăng max_wal_size (thường ×4–10), checkpoint_timeout 15–30min, completion_target 0.9; `wal_compression=lz4`.
- **Phòng:** dashboard nhịp checkpoint; review lại sau mỗi lần workload ghi tăng cấp.

## #7 Shared Buffer hit ratio thấp

- **Triệu chứng:** hit ratio tụt (99%→<95%); query chậm đều toàn diện.
- **Root cause:** working set > shared_buffers — do dữ liệu lớn lên, do bloat làm working set "loãng" (case #1), hoặc query mới quét rộng.
- **Bên trong:** Chương 4 — clock sweep churn; chú ý "miss" có thể vẫn hit OS cache (đừng hoảng vì mỗi con số này — đối chiếu disk read thật qua `pg_stat_io`/iostat).
- **Điều tra:** `pg_buffercache` xem cache chứa gì; `pg_stat_statements` theo shared_blks_read; bloat-check.
- **Khắc phục:** diệt bloat → thêm index → rồi mới thêm RAM (đúng thứ tự chi phí).

## #8 Too Many Connections

- **Triệu chứng:** `FATAL: sorry, too many clients already`; hoặc chưa chạm trần nhưng mọi thứ chậm dần khi connection tăng.
- **Root cause:** process-per-connection (Chương 2 §2) — mỗi connection tốn RAM + làm GetSnapshotData/ProcArray đắt lên; app scale ngang mỗi pod một pool 20.
- **Bên trong:** Chương 2 §4.3 — snapshot O(n connection), ProcArrayLock contention; RAM: catalog cache mỗi backend.
- **Điều tra:** `pg_stat_activity` đếm theo state — thường 80% `idle`; wait event ProcArray/LWLock khi cao điểm.
- **Khắc phục/Phòng:** PgBouncer transaction-mode trước database (max_connections thật ~2–4× core); quota pool per-service; timeout bộ ba (idle_in_transaction, statement, lock).

## #9 Long Transaction / xmin horizon bị ghim

- **Triệu chứng "tam chứng":** bloat tăng ở NHIỀU bảng cùng lúc + autovacuum log "0 removed / N dead but not yet removable" + một backend/slot có backend_xmin rất già.
- **Root cause:** transaction dài (báo cáo, migration treo, `idle in transaction` do app quên commit), replication slot logical bỏ rơi, prepared transaction mồ côi, hoặc hot_standby_feedback từ replica chạy báo cáo dài.
- **Bên trong:** Chương 6 §5 — MỘT snapshot già ghim rác TOÀN cluster.
- **Điều tra:** ba câu SQL ở Chương 6 §5 (pg_stat_activity theo age(backend_xmin), pg_replication_slots, pg_prepared_xacts).
- **Khắc phục:** kết thúc thứ đang ghim (`pg_terminate_backend`, drop slot, commit/rollback prepared).
- **Phòng:** bộ ba timeout; giám sát `max age(backend_xmin)` như một SLI hạng nhất; job dài chạy replica riêng.

## #10 Lock Contention (không phải deadlock)

- **Triệu chứng:** throughput sụp khi tải tăng; CPU thấp; pg_stat_activity đầy wait_event_type=Lock.
- **Root cause phổ biến:** hot row (một row counter/wallet bị mọi transaction update — hàng đợi row lock tuần tự hóa toàn hệ); DDL xếp hàng (Chương 7 §4.1 định luật 2); FK check dồn vào bảng cha.
- **Điều tra:** `pg_blocking_pids`; log_lock_waits=on; đồ thị wait event.
- **Khắc phục/Phòng:** hot row → tách shard row/batch update/đưa counter ra ngoài (Redis) hoặc gom bằng advisory lock; DDL → lock_timeout+retry; FK → index cột FK, cân nhắc mức lock.

## #11 Deadlock thường xuyên

- **Triệu chứng:** `ERROR: deadlock detected` (40P01) rải rác, tăng theo tải.
- **Bên trong:** Chương 7 §4.3 — chu trình wait-for; PostgreSQL phá sau deadlock_timeout.
- **Điều tra:** log deadlock in đủ hai câu query + lock chi tiết — đọc kỹ là ra thứ tự chạm ngược nhau; thường là 2 code path UPDATE nhiều row theo thứ tự khác nhau, hoặc UPDATE cha + INSERT con chéo.
- **Khắc phục/Phòng:** chuẩn hóa thứ tự truy cập (ORDER BY id trong SELECT FOR UPDATE); gom multi-row update thành một câu (một câu tự sort không đảm bảo thứ tự lock! — dùng ORDER BY trong subquery FOR UPDATE); retry 40P01 ở app là BẮT BUỘC.

## #12 HOT Update mất hiệu lực

- **Triệu chứng:** bảng update-nặng bỗng bloat nhanh + WAL tăng, sau một lần "thêm mỗi cái index".
- **Root cause:** index mới trùm lên cột được update → mọi update thành non-HOT → n index đều nhận entry mới mỗi update (Chương 10 §2, Chương 9 §5).
- **Điều tra:** `n_tup_hot_upd/n_tup_upd` tụt (so trước/sau deploy); pg_stat_user_indexes index mới idx_scan thấp (thuế cao, dùng ít).
- **Khắc phục:** bỏ index nếu ROI âm; hoặc partial index tránh cột nóng; fillfactor hạ để cứu điều kiện cùng-page.
- **Phòng:** review index mới trên bảng nóng bằng câu hỏi bắt buộc: "cột này có bị UPDATE không?"

## #13 VACUUM chạy hàng giờ

- **Triệu chứng:** một bảng vacuum 6–30 giờ, chiếm I/O, và luôn có nguy cơ bị hủy giữa chừng rồi làm lại từ đầu (mất công index pass).
- **Root cause:** kết hợp: bảng quá lớn không partition + cost throttle mặc định + maintenance_work_mem nhỏ (nhiều vòng index pass — Chương 10 §3, đỡ nhiều từ PG17) + nhiều index.
- **Điều tra:** `pg_stat_progress_vacuum` (phase, heap_blks_scanned, index_vacuum_count — thấy >1 vòng là thiếu bộ nhớ TID).
- **Khắc phục:** tăng autovacuum_work_mem, delay=0 cho bảng này, chạy giờ thấp; PG13+ vacuum tay dùng `PARALLEL` cho index.
- **Phòng:** partition (giải pháp thật sự); giảm số index; nâng PG17+ (TidStore).

## #14 Query Plan thay đổi đột ngột

- **Triệu chứng:** một query ổn định nhiều tháng đột nhiên chậm 100–1000×, không deploy code.
- **Root cause hàng đầu:** (a) ANALYZE mới đổi thống kê (cột thời gian tăng đơn điệu — Chương 8 §8); (b) prepared statement chuyển generic plan lần thứ 6 (Chương 8 §7); (c) dữ liệu vượt ngưỡng làm planner đổi join; (d) autovacuum chưa chạy sau bulk load — thống kê rỗng.
- **Điều tra:** auto_explain so plan xấu vs `EXPLAIN ANALYZE` tay (nếu tay nhanh mà app chậm → nghi (b) ngay); pg_stats nhìn histogram biên.
- **Khắc phục:** ANALYZE lại; plan_cache_mode=force_custom_plan cho phiên/role đó; statistics_target cao hơn cho cột lệch; extended statistics.
- **Phòng:** auto_explain thường trực; ANALYZE sau mọi bulk operation nằm trong runbook.

## #15 Buffer Cache Thrashing

- **Triệu chứng:** disk read tăng vọt, latency mọi query tăng, sau khi một tính năng/báo cáo mới lên.
- **Root cause:** query pattern mới quét dữ liệu lớn KHÔNG qua đường seq-scan-ring (vd index scan rộng, bitmap heap khổng lồ) → đuổi page nóng (ring buffer chỉ cứu seq scan thuần — Chương 4 §3).
- **Điều tra:** pg_buffercache trước/sau; pg_stat_statements shared_blks_read theo query mới.
- **Khắc phục:** đưa báo cáo sang replica; sửa query (partial index, giới hạn range); tăng RAM nếu working set thật sự lớn lên.

## #16 Full Table Scan bất ngờ

- **Triệu chứng:** một endpoint chậm; EXPLAIN thấy Seq Scan trên bảng lớn dù "đã có index".
- **Root cause checklist:** kiểu không khớp (`text` vs `varchar` OK, nhưng `int = text` cast mất index); hàm bọc cột (`WHERE date(created)=...` — cần expression index); LIKE '%x'; OR trên hai cột (cần 2 index + BitmapOr); selectivity thật quá cao (lấy 40% bảng → seq scan LÀ đúng); random_page_cost=4 trên SSD (Chương 8 §3.1); index HOÀN TOÀN bị bloat/INVALID (kiểm `pg_index.indisvalid` — CIC fail để lại!).
- **Điều tra:** EXPLAIN (ANALYZE, BUFFERS); `\d` xem index có valid.
- **Phòng:** review EXPLAIN trong CI cho query nóng; lint schema (kiểu khớp FK).

## #17 TOAST phình quá lớn

- **Triệu chứng:** `pg_total_relation_size` gấp 5–20 lần `pg_relation_size`; SELECT cột to chậm; vacuum lâu một cách khó hiểu.
- **Root cause:** jsonb/text/bytea lớn, UPDATE thường xuyên (mỗi update ghi lại TOÀN BỘ giá trị toast — Chương 3 §5); hoặc lưu blob hàng chục MB mỗi row.
- **Điều tra:** `SELECT relname, pg_size_pretty(pg_relation_size(reltoastrelid)) FROM pg_class WHERE reltoastrelid<>0 ORDER BY pg_relation_size(reltoastrelid) DESC;` — soi thẳng bảng toast; vacuum log của pg_toast.*.
- **Khắc phục/Phòng:** tách cột to ra bảng riêng (join khi cần — tránh SELECT * kéo toast oan); lz4 compression; blob thật sự lớn → object storage, DB giữ con trỏ; jsonb hay-update-một-key → cân nhắc tách key nóng thành cột thường.

## #18 Disk I/O 100%

- **Triệu chứng:** %util 100%, mọi thứ chậm. Câu hỏi đúng không phải "sao IO cao" mà "**ai** đang IO".
- **Điều tra phân loại (theo thứ tự):** iostat tách read/write; PG16+ `pg_stat_io` theo backend_type — checkpointer? autovacuum? backend read (cache miss)? backend write (bgwriter tụt — Chương 4 §5)?; pg_stat_statements theo blk_read_time (bật track_io_timing!).
- **Root cause map:** write burst chu kỳ → case #6; read tăng dần → case #7/#15; write đều cao → WAL (đo pg_stat_wal) hoặc vacuum; read+write hỗn loạn → thiếu index đâu đó đang quét (case #16).
- **Phòng:** track_io_timing=on thường trực; tách volume WAL/data; biết trần IOPS đã mua (cloud!) — nhiều "sự cố database" là hết credit IOPS của volume.

## #19 CPU thấp nhưng query vẫn chậm

- **Triệu chứng:** latency cao, CPU idle, disk nhàn — hệ "chậm mà không bận".
- **Root cause = ĐANG CHỜ GÌ ĐÓ — wait event trả lời:** `Lock:*` → case #10/#11; `LWLock:*` → contention cấu trúc (SubtransSLRU! — Chương 6 §6; WALWrite → fsync chậm); `IO:` cụ thể loại nào; `Client:` → app đọc kết quả chậm/mạng; ngoài PG: connection pooler bão hòa (chờ ở PgBouncer không hiện trong PG!), DNS, disk latency spike (đo fsync trực tiếp).
- **Điều tra:** snapshot pg_stat_activity mỗi 5–10s ra time-series wait event (hoặc pg_wait_sampling/pgsentinel); đừng đoán — đo.
- **Bài học:** "CPU thấp" không phải là khỏe — nó thường nghĩa là mọi người đang xếp hàng.

## #20 Memory Pressure / OOM

- **Triệu chứng:** backend bị kill signal 9 → toàn cluster restart (Chương 2 §9); hoặc swap khiến mọi thứ lệt bệt.
- **Root cause:** hiểu sai mô hình bộ nhớ: work_mem là per-node-per-query (một query 5 sort × 100 connection × 64MB = 32GB "hợp lệ"); hash join ước lượng sai → cấp phát vượt dự kiến (PG13+ có hash_mem_multiplier); connection nhiều → catalog cache mỗi backend phình (schema nghìn bảng).
- **Điều tra:** dmesg OOM; `pg_log`: "terminated by signal 9"; EXPLAIN các query nặng nhìn Sort/Hash node count.
- **Khắc phục/Phòng:** work_mem đặt theo công thức ngân sách (RAM khả dụng ÷ (connections × node trung bình)), tăng cục bộ bằng SET cho job lớn; overcommit_memory=2; pooler giảm connection; huge_pages.

## #21 WAL Archiver bị lỗi

- **Triệu chứng:** thầm lặng chết người — hệ chạy bình thường, cho tới khi: pg_wal đầy (case #5) hoặc ngày cần PITR phát hiện chuỗi WAL thủng 3 tuần.
- **Root cause:** archive_command fail (S3 credential hết hạn, đầy đích, đổi network policy) — PostgreSQL **thử lại mãi mãi và giữ segment lại**, không tự bỏ cuộc.
- **Bên trong:** Chương 5 §5, Chương 11 §4 — segment chỉ được recycle sau khi archive OK; chuỗi WAL thủng = mọi backup sau lỗ thủng vô dụng cho PITR.
- **Điều tra:** `pg_stat_archiver` (failed_count tăng, last_failed_wal); log archiver in stderr của command.
- **Khắc phục:** sửa lệnh/credential; archiver tự bắt kịp (theo dõi backlog `ls pg_wal | wc -l` giảm dần).
- **Phòng:** alert failed_count > 0 (không phải >N — MỘT lần fail kéo dài là đủ chết); alert độ trễ archive (segment cũ nhất chưa archive); restore-drill định kỳ là lưới cuối cùng.

---

## Phụ lục: bản đồ chẩn đoán nhanh 60 giây

```
Hệ chậm toàn diện?
├─ CPU cao?        → pg_stat_statements top CPU; spinlock (perf s_lock)?; JIT?
├─ Disk 100%?      → #18: pg_stat_io phân loại ai đang IO
├─ CPU & disk thấp?→ #19: wait events — Lock? LWLock? Client? pooler?
└─ Theo sóng?      → #6 checkpoint (log_checkpoints); vacuum lịch; cron job

Một query chậm?
├─ Chậm từ từ theo tuần  → #1/#2 bloat; #7 working set
├─ Chậm đột ngột         → #14 plan đổi (auto_explain!); #16 mất index
└─ Chậm qua app, nhanh psql → #14b generic plan; pooler; mạng

Disk đầy?
├─ pg_wal    → #5 (slot? archiver? checkpoint config?)
├─ base/     → #1 bloat; #17 TOAST; log tạm (temp files — work_mem)
└─ Đừng rm gì cả. Chẩn đoán trước.

Lỗi bắn hàng loạt?
├─ "recovery mode"        → Chương 2 §9: OOM? segfault? (dmesg + pg log)
├─ "too many clients"     → #8
├─ "not accepting commands"→ #4 WRAPAROUND — all hands
└─ "conflict with recovery"→ Chương 11 §5.2
```
