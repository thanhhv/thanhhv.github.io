+++
title = "Chương 4 — Buffer Manager"
date = "2026-07-11T11:00:00+07:00"
draft = false
tags = ["backend", "database", "postgresql"]
series = ["PostgreSQL Internals"]
+++

> Level 2. Disk chậm hơn RAM ~1000 lần. Buffer Manager là tầng quyết định
> PostgreSQL chạy ở tốc độ RAM hay tốc độ disk — tức là quyết định 90% cảm nhận hiệu năng.

---

## 1. Problem Statement

Mỗi thao tác đọc/ghi tuple đều cần page 8KB tương ứng. Nếu mỗi lần đều `read()`/`write()` xuống disk: một query đọc 1 triệu tuple rải trên 100k page × ~100µs/lần đọc NVMe = 10 giây thuần I/O — trong khi từ RAM mất ~50ms.

Bài toán con mà Buffer Manager phải giải cùng lúc:

1. **Tra cứu**: page (relation R, block B) đang ở đâu trong RAM? — phải O(1).
2. **Thay thế**: RAM đầy, đuổi (evict) page nào? — đuổi sai page nóng là thảm họa.
3. **Đồng thời**: hàng trăm backend cùng đọc/sửa page — ai chờ ai, chờ bao lâu?
4. **An toàn**: page bẩn (dirty) chỉ được ghi disk sau WAL của nó (WAL rule).
5. **Không bị quét rác**: một seq scan bảng 500GB không được đuổi sạch cache nóng.

**Nếu bỏ Buffer Manager, dựa hoàn toàn vào OS page cache?** Mất: kiểm soát WAL-before-data (kernel không biết LSN), pin page trong lúc sửa (kernel có thể evict bất cứ lúc nào), thứ tự flush cho checkpoint, và thông tin usage theo ngữ nghĩa database. Một số hệ (như SQLite ở mức nào đó) chấp nhận; mọi database server nghiêm túc đều tự quản cache.

## 2. Internal Architecture

```
                 BufTable (hash table trong shared memory)
        key: BufferTag{spcOid, dbOid, relNumber, forkNum, blockNum}
        value: buffer_id ──────────────────┐
                                           ▼
 ┌──────────────────────────────────────────────────────────────┐
 │ BufferDescriptors[N]  (mảng descriptor, mỗi cái 64B = 1 cache line)│
 │  ┌──────────────────────────────────────────────┐            │
 │  │ tag          : page nào đang ở đây           │            │
 │  │ state (atomic u32) đóng gói:                 │            │
 │  │   refcount   : bao nhiêu backend đang PIN    │            │
 │  │   usage_count: 0..5 — độ "nóng" (clock sweep)│            │
 │  │   flags      : DIRTY | VALID | IO_IN_PROGRESS│            │
 │  │ content_lock : LWLock đọc/sửa nội dung page  │            │
 │  └──────────────────────────────────────────────┘            │
 └──────────────────────────────────────────────────────────────┘
                                           │ buffer_id × 8192
                                           ▼
 ┌──────────────────────────────────────────────────────────────┐
 │ BufferBlocks: N × 8KB — dữ liệu page thật (shared_buffers)   │
 └──────────────────────────────────────────────────────────────┘
```

Ba tầng bảo vệ đồng thời, từ nhẹ tới nặng:

| Cơ chế | Bảo vệ gì | Thời gian giữ |
|---|---|---|
| **Pin** (refcount, atomic) | Page không bị evict khi đang dùng | Suốt lúc xử lý tuple trong page |
| **Content LWLock** (share/exclusive) | Nội dung page nhất quán khi đọc/sửa | Vài µs — chỉ lúc thao tác byte |
| **Buffer mapping lock** (partition ×128) | Hash table khi thêm/xóa entry | Rất ngắn |

Chi tiết LWLock/spinlock ở Chương 7. Điều cần nhớ: **pin rẻ (một atomic op) nhưng giữ lâu; lock đắt hơn nhưng giữ cực ngắn**.

## 3. Cách hoạt động — ReadBuffer từng bước

```go
// Pseudo code: backend cần page (rel, blk)
func ReadBuffer(rel Relation, blk BlockNumber) Buffer {
    tag := BufferTag{rel.spc, rel.db, rel.filenode, MAIN_FORK, blk}

    // 1. HIT PATH — mong đợi 99%
    if id, ok := bufTable.Lookup(tag); ok {
        PinBuffer(id)                    // atomic refcount++
        bumpUsageCount(id)               // tối đa 5
        return id                        // KHÔNG có syscall nào
    }

    // 2. MISS PATH
    victim := ClockSweep()               // tìm buffer đuổi được (xem dưới)
    if victim.dirty {
        // ★ WAL RULE: flush WAL tới victim.page.pd_lsn trước
        XLogFlush(victim.page.LSN())
        smgrwrite(victim.tag, victim.data)   // write() → OS cache
        // (không fsync ở đây — checkpoint mới fsync)
    }
    bufTable.Delete(victim.tag)
    bufTable.Insert(tag, victim.id)
    smgrread(tag, victim.data)           // read() 8KB — có thể hit OS cache!
    victim.state = VALID | PINNED
    return victim.id
}
```

### Clock Sweep — thuật toán thay thế

PostgreSQL không dùng LRU đúng nghĩa. LRU cần cập nhật danh sách liên kết **mỗi lần chạm page** → một điểm ghi chung, tranh chấp khủng khiếp với hàng trăm backend. Thay vào đó: **clock sweep (LRU xấp xỉ)**:

```
        buffers xếp thành vòng tròn, một "kim đồng hồ" đi quanh:

              ┌───┐ usage=3
        ┌───┐ │ B2│ ┌───┐          Kim chỉ B4: usage=0 & unpinned → VICTIM!
   u=5  │ B1│ └───┘ │ B3│ u=1      Nếu usage>0: usage-- rồi đi tiếp.
        └───┘   ▲   └───┘          Nếu pinned: bỏ qua.
              ┌─┴─┐
              │ B4│ u=0  ← kim
              └───┘
```

```go
func ClockSweep() *BufferDesc {
    for {
        buf := buffers[clockHand]
        clockHand = (clockHand + 1) % N
        if buf.refcount == 0 {           // không ai pin
            if buf.usageCount == 0 {
                return buf               // victim
            }
            buf.usageCount--             // "tha lần này"
        }
    }
}
```

Mỗi lần page được dùng: usage_count++ (trần 5). Page phải bị kim quét qua 5 lần không ai chạm mới bị đuổi. Trade-off so với LRU thật: kém chính xác một chút, nhưng **hit path hoàn toàn không có điểm tranh chấp chung** (chỉ atomic trên descriptor của chính page đó).

### Ring buffer — chống seq scan phá cache

Seq scan bảng lớn hơn 1/4 shared_buffers **không** dùng chung cơ chế trên: nó được cấp một **ring buffer riêng ~256KB** (32 buffer) và tự tái sử dụng vòng tròn trong đó. Tương tự: VACUUM (ring 2MB), bulk write như COPY (16MB).

**Nếu bỏ cơ chế này:** một analyst chạy `SELECT count(*) FROM events` (bảng 500GB) → 64 triệu page đi qua cache → usage_count của mọi page nóng bị kim quét về 0 → toàn bộ cache OLTP bị đuổi → p99 latency của cả hệ thống tăng vọt trong nhiều phút. Ring buffer đổi "seq scan nhanh hơn một chút nhờ cache" lấy "phần còn lại của hệ thống không bị thảm sát" — một trade-off dễ.

Hệ quả ngược cần biết: vì seq scan lớn dùng ring 256KB, **chạy lại seq scan lần hai vẫn đọc I/O như lần đầu** (nó không được cache trong shared buffers — nhưng có thể hit OS cache).

## 4. Bên dưới lớp vật lý — Double Buffering với OS cache

PostgreSQL đọc/ghi data file bằng buffered I/O (không dùng O_DIRECT cho data). Nghĩa là một page nóng tồn tại **hai bản**: một trong shared buffers, một trong OS page cache.

```
   shared_buffers (8GB)        ┌────────── RAM 64GB ──────────┐
   ┌────────────┐              │                              │
   │ page A ●   │   write()    │   OS page cache (~50GB)      │
   │ page B ●   │ ──────────▶  │   page A ● (bản sao!)        │
   └────────────┘              │   page C ● (chỉ ở đây)       │
        ▲                      └──────────┬───────────────────┘
        │ read() page C: "miss" của PG    │ fsync khi checkpoint
        │ nhưng hit OS cache → ~vài µs    ▼
        │ (nhanh hơn disk 20-50 lần)    DISK
```

**Tại sao chấp nhận lãng phí này?** (a) OS cache miễn phí về code — kernel làm readahead, writeback scheduling giỏi; (b) shared buffers quá lớn từng có scalability issue với clock sweep và checkpoint ghi dồn; (c) linh hoạt — RAM không bị "khóa cứng" cho PostgreSQL. Đây là lý do khuyến nghị kinh điển `shared_buffers = 25% RAM`: phần còn lại làm OS cache, và "miss" của shared buffers thường chỉ rơi xuống OS cache chứ không xuống disk.

**Hướng tương lai:** PG16+ xây dựng lại tầng I/O (AIO trong PG18, direct I/O thử nghiệm) để dần thoát double buffering — theo dõi vì khuyến nghị 25% sẽ thay đổi.

Lưu ý đo lường quan trọng: `pg_stat_database.blks_read` đếm "miss của shared buffers", **không phải** "đọc từ disk". Hit ratio 90% không có nghĩa 10% còn lại chạm disk — đa phần hit OS cache. Muốn biết disk thật: `pg_stat_io` (PG16+) kết hợp iostat.

## 5. Dirty page và hai người ghi nền

Page bị sửa chỉ đánh dấu DIRTY trong RAM. Ai ghi nó xuống disk? Ba ứng viên, theo thứ tự mong muốn **giảm dần**:

1. **Checkpointer** (Chương 11): mỗi 5 phút ghi *tất cả* dirty page, rải đều theo `checkpoint_completion_target`. Ghi tuần tự theo file → hiệu quả I/O tốt nhất.
2. **Background writer**: chạy trước kim clock sweep, ghi sẵn các page sắp-bị-evict để backend khỏi phải ghi. Nhịp `bgwriter_delay=200ms`, mỗi nhịp tối đa `bgwriter_lru_maxpages=100` page (mặc định — thường phải tăng).
3. **Backend tự ghi** (tệ nhất): clock sweep chọn victim dirty → backend phải flush WAL + write() page **trong critical path của query** → latency spike cho user.

Giám sát tỉ lệ ba nguồn này (PG17: `pg_stat_io`, cột `writes` theo `backend_type`; trước đó: `pg_stat_bgwriter.buffers_backend` vs `buffers_clean` vs `buffers_checkpoint`). **Backend chiếm tỉ lệ cao = shared_buffers thiếu hoặc bgwriter yếu.**

## 6. Trade-off

| Quyết định | Được | Mất |
|---|---|---|
| Clock sweep thay LRU thật | Hit path không tranh chấp | Xấp xỉ — page nóng vẫn có thể bị đuổi oan |
| Buffered I/O thay O_DIRECT | Tận dụng kernel readahead/writeback, code đơn giản | Double buffering tốn RAM; kernel writeback khó dự đoán (checkpoint storm — Chương 12 #5) |
| shared_buffers 25% thay 80% | OS cache đỡ miss; tránh nghẽn checkpoint | Hai lần copy cho page nóng |
| Ring buffer cho scan lớn | Cache OLTP an toàn | Scan lặp lại không nhanh lên |
| Pin/unpin bằng atomic | Rẻ | Buffer bị pin lâu (cursor bỏ quên) chặn evict và vacuum |

## 7. Production

- **Định cỡ:** bắt đầu 25% RAM. Tăng lên nếu: working set xác định được và vừa RAM, `pg_stat_io` cho thấy miss thật xuống disk nhiều. Giảm nếu: checkpoint viết quá dồn, hoặc máy chạy chung nhiều dịch vụ. Workload ghi nặng đôi khi chạy tốt hơn với shared_buffers **nhỏ hơn** (ít dirty page dồn cục cho checkpoint).
- **Hit ratio:** `blks_hit/(blks_hit+blks_read)` — OLTP lành mạnh ≥ 99%. Nhưng nhớ mục 4: đây là hit của tầng PG, không phải của RAM tổng.
- **Đo từng bảng:** `pg_statio_user_tables` — bảng nào gây miss nhiều nhất; extension `pg_buffercache` — soi chính xác cái gì đang chiếm cache (`SELECT relname, count(*) FROM pg_buffercache b JOIN pg_class c ON b.relfilenode=c.relfilenode GROUP BY 1 ORDER BY 2 DESC`).
- **prewarm:** sau restart, cache nguội → p99 xấu trong nhiều phút. `pg_prewarm` (kèm `pg_prewarm.autoprewarm=on`) lưu danh sách buffer và nạp lại sau restart.
- **Benchmark đại diện** (pgbench scale 1000, ~15GB data, máy 16GB RAM, NVMe):

```
shared_buffers=1GB : tps ≈ 8.500   (miss nhiều nhưng OS cache đỡ)
shared_buffers=4GB : tps ≈ 12.000  (working set index vừa vào)
shared_buffers=12GB: tps ≈ 9.000   (OS cache còn quá ít, checkpoint nặng)
→ Đường cong hình chuông — "to hơn" không phải lúc nào cũng "nhanh hơn".
```

## 8. Failure Cases

### Case A: Buffer cache thrashing
**Triệu chứng:** hit ratio tụt từ 99% xuống 85%, mọi query chậm đều, disk read tăng vọt, CPU iowait cao.
**Root cause điển hình:** working set phình vượt RAM (dữ liệu tăng trưởng tự nhiên/bloat làm working set "loãng"), hoặc một query pattern mới quét index khổng lồ.
**Bên trong:** clock sweep quay điên cuồng (theo dõi được qua `pg_stat_bgwriter.buffers_alloc` tăng vọt), page nóng bị đuổi rồi nạp lại liên tục.
**Điều tra:** `pg_buffercache` xem cache đang chứa gì; `pg_stat_statements` sort theo `shared_blks_read` tìm query gây miss; kiểm tra bloat (working set loãng vì dead tuple chen giữa).
**Khắc phục:** diệt bloat trước (rẻ nhất), thêm index cho query quét rộng, rồi mới tính thêm RAM.

### Case B: "IO chậm nhưng chỉ thỉnh thoảng" — backend tự ghi page
**Triệu chứng:** p99 nhấp nhô theo chu kỳ, trùng nhịp không rõ ràng.
**Root cause:** bgwriter không kịp → backend tự evict dirty page → mỗi lần dính thêm 1 write + có thể 1 XLogFlush.
**Điều tra:** `buffers_backend` (hoặc `pg_stat_io` writes của backend_type=client backend) chiếm >20–30% tổng write.
**Khắc phục:** tăng `bgwriter_lru_maxpages` (vd 1000), giảm `bgwriter_delay`, xem lại shared_buffers và nhịp checkpoint.

## 9. Tóm tắt

- Buffer manager = hash table + descriptor + clock sweep + 3 tầng bảo vệ (pin, content lock, mapping lock).
- WAL rule được thi hành tại điểm evict và checkpoint: không dirty page nào chạm disk trước WAL của nó.
- Ring buffer cách ly scan lớn khỏi cache OLTP.
- Double buffering với OS cache là lựa chọn có chủ đích — nguồn gốc quy tắc 25% RAM, đang thay đổi dần với direct I/O/AIO.
- Ba người ghi dirty page: checkpointer (tốt) > bgwriter (ổn) > backend (xấu — theo dõi tỉ lệ này).

**Tiếp theo:** [Chương 5 — WAL](/series/postgres-internal/05-wal/): thứ cho phép mọi dirty page trong chương này được phép "lười".
