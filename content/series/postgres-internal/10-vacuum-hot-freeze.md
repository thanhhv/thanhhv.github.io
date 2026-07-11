+++
title = "Chương 10 — VACUUM, HOT, Freeze, Wraparound"
date = "2026-07-11T17:00:00+07:00"
draft = false
tags = ["backend", "database", "postgresql"]
series = ["PostgreSQL Internals"]
+++

> Level 3–5. MVCC vay nợ: mỗi UPDATE/DELETE để lại rác, mỗi transaction đốt một XID hữu hạn.
> VACUUM là bộ máy trả nợ. Vận hành PostgreSQL nghiêm túc = vận hành VACUUM nghiêm túc.
> Chương này là chương "đáng tiền" nhất với người trực production.

---

## 1. Problem Statement — ba món nợ của MVCC

1. **Dead tuple:** version cũ không còn snapshot nào cần (Chương 6). Không dọn → bảng/index phình, cache loãng, scan chậm.
2. **Không gian XID hữu hạn:** XID 32 bit so sánh vòng tròn. Tuple mang xmin quá già (cách hiện tại >2³¹) sẽ bị phép so sánh hiểu ngược thành "tương lai" → **dữ liệu committed biến mất**. Phải "đóng băng" (freeze) tuple già trước khi điều đó xảy ra.
3. **Thống kê & Visibility Map lạc hậu:** planner cần ANALYZE; index-only scan cần VM được cập nhật.

VACUUM giải cả ba. **Nếu bỏ VACUUM?** — không phải "chậm dần" mà là **dừng hẳn**: đầy disk vì bloat, rồi trước cả đó, cluster tự khóa ghi để chống wraparound. Không có lựa chọn "không vacuum", chỉ có "vacuum có kiểm soát" hoặc "vacuum lúc 3h sáng theo lệnh của PostgreSQL".

## 2. HOT Update — tuyến phòng thủ đầu tiên (trước cả vacuum)

HOT (Heap-Only Tuple, PG8.3) là tối ưu quan trọng nhất của write path. Điều kiện: (a) version mới **vừa trong cùng page** với version cũ; (b) **không cột nào có index bị thay đổi**.

```
   UPDATE không đổi cột index, còn chỗ trong page:

   INDEX:  entry k → (42,1)          ← KHÔNG THÊM GÌ! vẫn trỏ lp1
   HEAP page 42:
     lp1 → v1 [xmax=205, HOT_UPDATED, ctid=(42,7)]
                    │  hot chain nội page
     lp7 → v2 [xmin=205, HEAP_ONLY_TUPLE]   ← không index nào trỏ tới

   Reader: index → lp1 → lần theo chain → v2 (kiểm visibility từng mắt xích)
```

Lợi kép: không ghi index nào (n index = tiết kiệm n lần chèn + WAL), và rác dọn được **không cần vacuum** — bằng **HOT pruning**: bất kỳ SELECT/UPDATE nào chạm page thấy chain có mắt xích chết vượt horizon sẽ tự cắt ngắn chain, gom xác (đổi line pointer thành REDIRECT, giải phóng chỗ) — chỉ với lock nội page, rẻ.

**Điều kiện (a) là lý do `fillfactor < 100` tồn tại:** page nhồi kín 100% thì version mới hết chỗ cùng page → non-HOT → mọi index bị đánh thuế. Bảng update nóng: fillfactor 70–90.

**Đo được:** `pg_stat_user_tables.n_tup_hot_upd / n_tup_upd` — tỷ lệ HOT. Bảng update nhiều mà HOT < 50% → soi: index thừa trên cột hay đổi? fillfactor? Chương 12 case #12.

## 3. VACUUM — bộ máy chính, mổ xẻ từng pha

`VACUUM` (thường, không FULL) chạy song song với DML — chỉ lấy ShareUpdateExclusiveLock (không chặn đọc/ghi, chỉ chặn DDL và vacuum khác):

```
Pha 1 — SCAN HEAP (theo VM: bỏ qua page all-visible)
   gom danh sách TID chết vào bộ nhớ
   (PG17+: TidStore nén, hết giới hạn cứng 1GB maintenance_work_mem cũ)
   đồng thời: prune HOT chain, defragment page, cập nhật FSM/VM

Pha 2 — VACUUM INDEXES  ★ pha đắt nhất
   VỚI TỪNG INDEX: quét TOÀN BỘ index, xóa entry trỏ tới TID chết
   (PG13+: index nhiều có thể vacuum song song — autovacuum thì không)
   → đây là lý do "nhiều index" làm vacuum chậm hơn tuyến tính

Pha 3 — VACUUM HEAP (quay lại)
   xóa hẳn line pointer chết (LP_DEAD → LP_UNUSED)
   — chỉ được làm SAU khi mọi index đã hết entry trỏ tới (nếu không:
   index trỏ vào slot được tái sử dụng = trả về row sai! đây là lý do
   thứ tự 3 pha là bất biến thiêng liêng)

Pha 4 — dọn dẹp: truncate đuôi file nếu các page cuối rỗng hoàn toàn
   (cần AccessExclusive thoáng qua — có thể bị bỏ nếu tranh chấp),
   cập nhật pg_class.reltuples/relpages, FSM/VM tổng, ANALYZE nếu là autovacuum analyze
```

Nếu danh sách TID chết không vừa bộ nhớ → **lặp lại nhiều vòng** pha 1–3 → vacuum lâu gấp bội. Trước PG17 đây là lý do số một tăng `maintenance_work_mem`/`autovacuum_work_mem`.

**VACUUM FULL là một lệnh hoàn toàn khác:** rewrite toàn bộ bảng + mọi index vào file mới, AccessExclusiveLock từ đầu đến cuối, cần disk gấp đôi. Nó là "cứu hộ bloat nặng", không phải "vacuum kỹ hơn". Online thay thế: `pg_repack`, `pg_squeeze`.

## 4. Freeze & Wraparound — món nợ thứ hai, nguy hiểm hơn

### 4.1. Cơ chế

XID so sánh vòng tròn: với XID hiện tại là C, các XID trong nửa vòng "sau lưng" (C−2³¹, C) là quá khứ, nửa vòng trước mặt là tương lai. Tuple mang xmin già hơn 2 tỷ transaction → lọt sang "trước mặt" → visibility check coi nó là chưa-được-tạo → **hàng loạt row committed biến mất khỏi kết quả**. Không corrupt file — corrupt **ngữ nghĩa**, tệ hơn nhiều vì backup cũng "mất" theo.

Giải pháp: trước khi xmin già tới mức đó, **freeze** tuple — đặt cờ `HEAP_XMIN_FROZEN` trong infomask (PG9.4+; trước kia ghi đè xmin=2). Tuple frozen = "già vô hạn, visible với mọi người, mãi mãi" — thoát khỏi vòng so sánh.

Sổ sách theo dõi: mỗi bảng có `pg_class.relfrozenxid` = "mọi xmin trong bảng này đều mới hơn hoặc đã frozen so với mốc này"; mỗi database có `pg_database.datfrozenxid` = min các bảng. `age(datfrozenxid)` là **đồng hồ đếm ngược wraparound của cluster**.

### 4.2. Nấc thang leo thang của PostgreSQL

```
age(relfrozenxid) vượt:
  vacuum_freeze_table_age (150M)   → vacuum thường chuyển chế độ aggressive
                                     (quét cả page all-visible chưa all-frozen)
  autovacuum_freeze_max_age (200M) → autovacuum ÉP chạy anti-wraparound vacuum
                                     kể cả khi bảng không có dead tuple,
                                     KỂ CẢ autovacuum=off  ← "vacuum lạ lúc 3h sáng"
  ~2^31 − 3M  (hết đường)          → cluster từ chối cấp XID mới: chỉ đọc,
                                     đòi vacuum trong single-user mode (PG14+ đỡ hơn)
```

Anti-wraparound vacuum **không thể hủy bằng cách thông thường** (nó tự trọng hơn vacuum thường: không nhường lock cho DDL). Trên bảng chục TB nó chạy hàng chục giờ, chiếm I/O — nhiều sự cố nổi tiếng (các bài post-mortem của Sentry, Mailchimp...) đều là biến thể "để dồn nợ freeze đến khi PostgreSQL tự đòi".

**Cách trả nợ đúng: trả sớm, trả nhỏ.** VM có bit all-frozen → vacuum aggressive bỏ qua page đã frozen → **vacuum freeze định kỳ trên bảng còn nhỏ là rẻ**. Chiến lược: giám sát `age(relfrozenxid)` per-table, chủ động `VACUUM (FREEZE)` các bảng lớn trong giờ thấp điểm khi age ~100–150M, đừng đợi 200M tự nổ. Nhớ cả **MultiXact** (`mxid_age(relminmxid)`) — cùng bài toán, ít người canh.

## 5. Autovacuum — scheduler và các nút chỉnh

### 5.1. Khi nào một bảng được chọn

```
ngưỡng dead  = autovacuum_vacuum_threshold(50) + scale_factor(0.2) × reltuples
ngưỡng insert= autovacuum_vacuum_insert_threshold(1000) + 0.2 × reltuples  (PG13+,
               để bảng append-only cũng được vacuum → set VM/freeze/hint bit)
ngưỡng analyze = 50 + 0.1 × reltuples
```

**Mặc định 20% là quá lười cho bảng lớn:** bảng 500M row phải gom 100M dead tuple mới được dọn. Chuẩn thực chiến: bảng lớn/nóng đặt per-table `autovacuum_vacuum_scale_factor=0.01` (thậm chí 0 + threshold tuyệt đối vài chục nghìn).

### 5.2. Cost-based throttling — cái phanh bị bỏ quên

Autovacuum tự phanh để không giết I/O production: mỗi page hit/miss/dirty cộng điểm; đủ `autovacuum_vacuum_cost_limit` (200) thì ngủ `autovacuum_vacuum_cost_delay` (2ms). Mặc định cho tốc độ dọn tối đa **chỉ ~vài chục MB/s** — quá chậm cho bảng ghi hàng GB/giờ: vacuum không bao giờ đuổi kịp, dead tuple tích lũy vô hạn trong khi "autovacuum vẫn đang chạy" (!). Hệ ghi nặng trên NVMe: tăng cost_limit lên 2000–10000 hoặc delay=0 cho các bảng nóng. **Vacuum chạy nhanh và xong còn hơn chạy dịu dàng và không bao giờ xong.**

`max_autovacuum_workers=3` mặc định cũng là nút nghẽn: 3 worker **chia nhau** cost_limit toàn cục, và một bảng chỉ được một worker (song song index chỉ có ở vacuum tay). Nhiều bảng lớn cùng đến hạn → xếp hàng.

## 6. Pseudo code — vòng đời dead tuple, nối mọi chương

```go
// Từ UPDATE đến khi byte được tái sử dụng:
func lifeOfDeadTuple() {
    // t0: UPDATE tạo v2, v1.xmax = 205                (Chương 3, 6)
    // t1: XID 205 commit                              (CLOG — Chương 6)
    // t2: các snapshot cũ hơn 205 lần lượt kết thúc
    //     → v1 vượt xmin horizon: CHÍNH THỨC DEAD     (Chương 6)
    // t3: SELECT chạm page → HOT prune cắt chain (nếu HOT)
    //     hoặc chờ vacuum
    // t4: autovacuum chọn bảng (vượt threshold)
    //     pha1 gom TID, pha2 quét MỌI index xóa entry,
    //     pha3 line pointer → LP_UNUSED, FSM ghi nhận chỗ trống
    // t5: INSERT sau đó hỏi FSM → tái sử dụng đúng chỗ đó
    //     (bảng KHÔNG co lại — chỉ ngừng phình)       (Chương 3)
}
```

## 7. Trade-off

| Quyết định | Được | Mất |
|---|---|---|
| Dọn rác nền (vacuum) thay vì inline (undo) | Write path nhanh, rollback O(1) | Cả một hệ thống scheduler/tuning; nợ có thể dồn |
| Freeze bằng cờ thay vì XID 64-bit | Tuple header giữ 23B (XID 64-bit = +8B/tuple mọi bảng) | Toàn bộ bộ máy freeze/wraparound; (cộng đồng vẫn tranh luận 64-bit XID nhiều năm) |
| Autovacuum throttle mặc định hiền | An toàn cho máy yếu | Trên máy khỏe: dọn không kịp — cấu hình mặc định là bẫy |
| VACUUM không trả disk cho OS | Không cần lock dài | Bloat chỉ "ngừng tệ đi", muốn co phải rewrite |

## 8. Production — checklist chưng cất

- **Giám sát 4 con số mỗi bảng lớn:** `n_dead_tup` (xu hướng), `last_autovacuum`, tỷ lệ HOT, `age(relfrozenxid)`. Cluster-wide: `max(age(datfrozenxid))` — alert 500M, trang 1 tỷ.
- **Vì sao vacuum "chạy mà không dọn":** horizon bị ghim (Chương 6 mục 5 — ba nghi phạm: txn dài, slot, prepared xact). Log vacuum có dòng `removable cutoff` / `dead but not yet removable` — đọc nó trước khi tune bất cứ gì.
- **`log_autovacuum_min_duration=0`** (hoặc 250ms): log mọi lần vacuum kèm chỉ số chi tiết — nguồn dữ liệu tune quý nhất, gần như miễn phí.
- **Bảng queue (ghi/xóa điên cuồng):** scale_factor=0, threshold thấp, cost_delay=0, fillfactor thấp; hoặc thiết kế lại (partition xoay vòng, `SKIP LOCKED`, hoặc đừng dùng bảng làm queue tần suất cực cao).
- **Partition lớn theo thời gian:** DELETE hàng loạt → DROP/DETACH PARTITION: không dead tuple, không vacuum, không bloat index. Đây là lý do vận hành số một để partition.

## 9. Failure case của chương: anti-wraparound trên bảng 8TB

**Triệu chứng:** 03:00, I/O tăng vọt, latency toàn hệ tăng; `pg_stat_activity` có `autovacuum: VACUUM public.events (to prevent wraparound)`. Kill → nó quay lại ngay. Đã tắt autovacuum trên bảng này từ lâu "vì nó làm chậm hệ thống" (!).

**Diễn biến:** chính vì tắt autovacuum, nợ freeze tích 200M+ age → PostgreSQL ép anti-wraparound (bỏ qua cờ off). Bảng 8TB chưa từng freeze → phải đọc gần như toàn bộ (VM ít bit all-frozen) với cost throttle mặc định → ước tính 30+ giờ. Trong lúc đó `datfrozenxid` không tiến → đồng hồ wraparound tiếp tục chạy → nguy cơ read-only thật sự.

**Điều tra:** `SELECT relname, age(relfrozenxid) FROM pg_class WHERE relkind='r' ORDER BY 2 DESC;` — thấy events ở 200M+. `pg_stat_progress_vacuum` xem pha và tiến độ.

**Xử lý khẩn:** KHÔNG kill vacuum vô ích; ngược lại **tháo phanh cho nó**: `ALTER TABLE events SET (autovacuum_vacuum_cost_delay=0);` (PG có thể cần restart worker), tăng autovacuum_work_mem, tạm hoãn job I/O khác. **Phòng:** không bao giờ tắt autovacuum trên bảng lớn; freeze chủ động theo lịch; partition; alert age hai nấc.

## 10. Tóm tắt

- HOT là tuyến đầu: update cùng page + không đổi cột index = không đánh thuế index, rác tự dọn bằng prune. Fillfactor và kỷ luật index quyết định tỷ lệ HOT.
- VACUUM 3 pha, thứ tự heap-scan → index → heap là bất biến an toàn; index nhiều = vacuum đắt.
- Freeze là nợ bắt buộc của XID 32-bit; trả sớm trả nhỏ, đừng để PostgreSQL tự xiết nợ bằng anti-wraparound trên bảng chục TB.
- Autovacuum mặc định được tune cho máy nhỏ 2007: scale_factor, cost_limit, workers đều cần chỉnh trên hệ hiện đại.
- Vacuum không co bảng — bloat đã thành hình chỉ xử bằng rewrite (pg_repack/VACUUM FULL).

**Tiếp theo:** [Chương 11 — Checkpoint, Recovery, Replication](/series/postgres-internal/11-recovery-replication/).
