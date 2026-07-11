+++
title = "Chương 8 — Query Processing: từ SQL text đến thao tác trên Page"
date = "2026-07-11T15:00:00+07:00"
draft = false
tags = ["backend", "database", "postgresql"]
series = ["PostgreSQL Internals"]
+++

> Level 4. SQL là ngôn ngữ khai báo: bạn nói *cần gì*, không nói *làm thế nào*.
> Toàn bộ chương này là câu chuyện PostgreSQL biến "cần gì" thành "làm thế nào" —
> và tại sao đôi khi nó chọn sai.

---

## 1. Problem Statement

```sql
SELECT c.name, sum(o.amount)
FROM customers c JOIN orders o ON o.customer_id = c.id
WHERE c.region = 'APAC' AND o.created_at > now() - interval '30 days'
GROUP BY c.name;
```

Câu này có thể thực thi theo hàng nghìn cách: quét bảng nào trước, dùng index nào, join bằng thuật toán nào, group bằng hash hay sort. Chênh lệch giữa plan tốt nhất và tệ nhất trên dữ liệu lớn: **10⁴–10⁶ lần**. Không tầng nào khác của database tạo ra chênh lệch hiệu năng lớn như planner.

**Nếu bỏ planner, thực thi theo thứ tự viết?** Chính là thế giới trước System R (1979): programmer phải tự viết access path. SQL tồn tại được là nhờ tin rằng optimizer chọn giỏi hơn người trong đa số trường hợp — và chương này cũng chỉ ra các trường hợp niềm tin đó gãy.

## 2. Pipeline tổng thể

```
 SQL text
   │  Parser (gram.y — văn phạm LALR): text → parse tree thô
   │      lỗi cú pháp bắn ra ở đây
   ▼
 Analyzer: tra catalog (bảng? cột? kiểu?) → Query tree
   ▼
 Rewriter: áp dụng RULE — quan trọng nhất: VIEW được "inline"
   │      (view = macro, không phải bảng tạm!)
   ▼
 Planner/Optimizer: Query tree → Plan tree (CHỌN CÁCH THỰC THI)
   ▼
 Executor: chạy plan tree, kéo tuple qua Buffer Manager/heap/index
   │      (mọi chương trước phục vụ tầng này)
   ▼
 kết quả trả client
```

Prepared statement cắt pipeline: parse+analyze một lần, plan có thể cache (generic plan sau 5 lần chạy nếu không thiệt — nguồn của failure case "query nhanh 5 lần đầu, chậm từ lần 6", xem mục 7).

## 3. Planner — trái tim

### 3.1. Cost model: đơn vị đo là "trang seq scan"

Planner định giá mỗi cách thực thi bằng công thức tuyến tính trên các hằng:

```
seq_page_cost    = 1.0    ← đơn vị gốc: đọc 1 page tuần tự
random_page_cost = 4.0    ← đọc 1 page ngẫu nhiên (số của thời HDD!)
cpu_tuple_cost   = 0.01   ← xử lý 1 tuple
cpu_index_tuple_cost = 0.005
cpu_operator_cost    = 0.0025

cost(SeqScan)   = số_page × 1.0 + số_tuple × 0.01
cost(IndexScan) = (page index + page heap dự kiến) × random_page_cost × tương_quan
                + tuple × (cpu costs)
```

**Hằng số quan trọng nhất để tune: `random_page_cost`.** Mặc định 4.0 mô hình hóa HDD (seek đắt). Trên SSD/NVMe thực tế ~1.1–1.5. Để 4.0 trên SSD → planner **sợ index scan một cách vô lý** → chọn seq scan cho query lẽ ra dùng index. Đây là tinh chỉnh một-dòng có tác động lớn nhất trong postgresql.conf với hạ tầng hiện đại (kèm `effective_cache_size` = ~70% RAM để planner tin rằng index đang ấm).

### 3.2. Selectivity — planner "nhìn" dữ liệu qua thống kê

Planner không đọc dữ liệu lúc plan. Nó dùng thống kê do `ANALYZE` thu (mẫu ~30.000×`statistics_target/100` row), lưu trong `pg_statistic` (xem qua view `pg_stats`):

- `n_distinct`: số giá trị khác nhau (ước lượng!)
- MCV: danh sách giá trị phổ biến nhất + tần suất
- histogram: phân bố phần còn lại
- `correlation`: thứ tự vật lý tương quan thứ tự logic (quyết định index scan rẻ hay đắt!)

```
WHERE region='APAC'         → tra MCV: 'APAC' chiếm 7% → selectivity 0.07
WHERE created > X           → tra histogram: ~12% row lớn hơn X
WHERE region='APAC' AND created > X
                            → 0.07 × 0.12 = 0.0084  ★ GIẢ ĐỊNH ĐỘC LẬP!
```

**Giả định độc lập giữa các cột là gót chân Achilles.** Cột tương quan (city='Hanoi' AND country='VN') → planner nhân xác suất → ước lượng thấp hơn thực tế hàng trăm lần → chọn nested loop cho 1 triệu row → thảm họa. Thuốc: `CREATE STATISTICS` (extended statistics: dependencies, ndistinct, mcv trên nhóm cột) — PG10+, ít người dùng, hiệu quả cao.

### 3.3. Join ordering — bài toán tổ hợp

N bảng join → số thứ tự khả dĩ tăng kiểu giai thừa/Catalan. Planner dùng dynamic programming (System R style) đủ N ≤ `geqo_threshold` (12); vượt ngưỡng chuyển sang **GEQO** (genetic algorithm — nhanh nhưng plan ngẫu nhiên hơn). Query ORM join 15 bảng = plan không ổn định by design.

### 3.4. Ba thuật toán join — và khi nào planner chọn sai

| | Nested Loop | Hash Join | Merge Join |
|---|---|---|---|
| Cách chạy | Với mỗi row ngoài, dò bảng trong (qua index) | Build hash table từ bảng nhỏ, probe bằng bảng lớn | Sort hai phía, trộn |
| Tốt khi | Bảng ngoài **ít row**, bảng trong có index | Equi-join, bảng vừa memory (work_mem) | Hai phía đã có thứ tự (index) |
| Thảm họa khi | Ước lượng bảng ngoài sai: 10 row hóa 1 triệu → 1 triệu lần index probe | Hash tràn work_mem → spill disk (batch) | Phải sort hai bảng khổng lồ |
| Chi phí | O(N×logM) | O(N+M) | O(NlogN+MlogM) hoặc O(N+M) nếu sẵn thứ tự |

90% các "query đột nhiên chậm" là một trong hai chữ ký: (a) **nested loop trên ước lượng sai** (rows=1 thực tế =500k); (b) **hash/sort spill disk** vì work_mem thiếu. `EXPLAIN (ANALYZE, BUFFERS)` phân biệt được ngay: so sánh `rows=` ước lượng vs `actual rows=`; tìm `Sort Method: external merge Disk:` hoặc `Batches: >1`.

## 4. Executor — Volcano model

Plan tree được thực thi theo mô hình **Volcano/iterator**: mỗi node có một hàm "cho tôi tuple tiếp theo", kéo (pull) từ con của nó:

```go
// Mọi node executor tuân một interface
type PlanNode interface{ Next() (Tuple, bool) }

// Ví dụ: HashJoin
func (j *HashJoin) Next() (Tuple, bool) {
    if !j.built {                      // pha build: hút cạn bên trong
        for t, ok := j.inner.Next(); ok; t, ok = j.inner.Next() {
            j.table.Insert(hash(t.key), t)
        }
        j.built = true
    }
    for {                              // pha probe: kéo từng tuple ngoài
        t, ok := j.outer.Next()
        if !ok { return Tuple{}, false }
        if m := j.table.Lookup(hash(t.key)); m != nil {
            return join(t, m), true    // trả NGAY — pipeline!
        }
    }
}
```

Tính chất pipeline nghĩa là: `LIMIT 10` có thể dừng cả cây sau 10 tuple (rẻ) — trừ khi trong cây có node **chặn** (Sort, HashAgg, phase build của HashJoin) phải hút cạn con trước khi nhả tuple đầu tiên. Nhìn plan, phân biệt node chảy/node chặn là kỹ năng đọc plan quan trọng nhất.

Chi tiết đáng biết: executor xử lý **một tuple mỗi lần gọi** — tối ưu tổng quát nhưng đắt CPU (function call, cache miss mỗi tuple). PG12+ JIT (LLVM) biên dịch biểu thức nóng; các hệ OLAP (DuckDB, ClickHouse) chọn vectorized execution (batch 1000+ tuple) — nhanh hơn hẳn cho analytics, và là một lý do PostgreSQL không phải database phân tích (Chương 13).

### Các node scan — nối về Chương 3, 4, 9

- **Seq Scan:** đọc heap tuần tự (ring buffer — Chương 4), lọc từng tuple. Thắng khi lấy >5–10% bảng.
- **Index Scan:** dò B-tree → ctid → **nhảy vào heap kiểm tra visibility** (MVCC không sống trong index!). Mỗi tuple = 1 lần random access heap.
- **Index Only Scan:** như trên nhưng nếu page all-visible (VM — Chương 3) thì khỏi chạm heap. VM lạc hậu (vacuum chưa chạy) → "index only" vẫn đọc heap (`Heap Fetches:` cao trong EXPLAIN — dấu hiệu cần vacuum).
- **Bitmap Index/Heap Scan:** dò index, gom ctid vào bitmap, sort theo vị trí vật lý, đọc heap **tuần tự hóa**. Trung dung giữa hai thái cực; tràn `work_mem` → bitmap chuyển "lossy" (nhớ theo page, phải recheck từng tuple).

## 5. Cách hoạt động — một query đi hết pipeline

```
EXPLAIN (ANALYZE, BUFFERS)
SELECT c.name, sum(o.amount) FROM customers c JOIN orders o ...

 HashAggregate  (actual time=891..902 rows=1204)      ← node chặn
   Group Key: c.name
   Buffers: shared hit=48211 read=1503                ← 1503 page từ ngoài PG cache
   ->  Hash Join  (actual rows=182_000)
         Hash Cond: (o.customer_id = c.id)
         ->  Index Scan using idx_orders_created on orders o
               Index Cond: (created_at > ...)         ← chọn nhờ histogram
               (actual rows=190_500)  ← so với ước lượng rows=185_000: TỐT
         ->  Hash (actual rows=890)
               ->  Index Scan using idx_cust_region on customers c
                     Index Cond: (region = 'APAC')
 Planning Time: 0.4 ms
 Execution Time: 903 ms
```

Trình tự đọc một plan khi debug: (1) tìm node có `actual time` lớn nhất; (2) so `rows` ước lượng vs thực tế ở node đó và các con — lệch >10× là gốc rễ; (3) nhìn `Buffers` tìm I/O bất thường; (4) tìm `Disk:`/`Batches:`/`Heap Fetches:`.

## 6. Trade-off

| Quyết định | Được | Mất |
|---|---|---|
| Cost-based thay rule-based | Thích nghi dữ liệu thật | Phụ thuộc thống kê — thống kê sai là plan sai; plan **không ổn định** theo thời gian |
| Không hint (triết lý core team) | Ép sửa gốc rễ (thống kê, schema) thay che triệu chứng | Lúc cháy nhà không có bình cứu hỏa — thực tế nhiều nơi cài `pg_hint_plan` |
| Volcano 1-tuple/lần | Tổng quát, LIMIT/pipeline tự nhiên | CPU overhead cao cho analytics (đối thủ: vectorized) |
| Generic plan cho prepared stmt | Tiết kiệm chi phí plan | Plan mù giá trị tham số — bẫy phân bố lệch (mục 7) |
| GEQO cho join lớn | Plan được query 15+ bảng | Plan ngẫu nhiên, khó tái lập |

## 7. Production

- **`pg_stat_statements` là extension bắt buộc số 1:** tổng thời gian, số lần chạy, I/O từng query chuẩn hóa. Sự cố hiệu năng bắt đầu từ `ORDER BY total_exec_time DESC LIMIT 20`, không phải từ đoán.
- **auto_explain:** log plan của query vượt ngưỡng (`auto_explain.log_min_duration=1s, log_analyze=on` cẩn thận overhead) — cách duy nhất bắt được plan xấu "lúc 3 giờ sáng".
- **ANALYZE đúng nhịp:** autovacuum analyze theo `autovacuum_analyze_scale_factor` (10% row đổi). Bảng lớn: 10% = quá thưa → hạ per-table. Sau bulk load: ANALYZE tay ngay.
- **Bẫy generic plan:** prepared statement chạy 5 lần custom plan, từ lần 6 có thể chuyển generic. Cột phân bố lệch (90% status='done') → generic plan chọn đường cho giá trị "trung bình" → chậm 100× với giá trị hiếm. Nhận diện: query nhanh trong psql, chậm qua ứng dụng. Sửa: `SET plan_cache_mode = force_custom_plan` (per role/db nếu cần).
- **JIT:** lợi cho analytics chạy giây/phút; hại cho OLTP (chi phí compile > lợi ích). PG17 đã bớt hăng mặc định; nếu p99 OLTP có spike lạ vài chục ms — thử `jit=off`.

## 8. Failure case của chương: plan đổi đột ngột sau nửa đêm

**Triệu chứng:** 02:15, query dashboard từ 40ms nhảy lên 35s. Không deploy gì. Sáng hôm sau tự... vẫn chậm.

**Diễn biến:** autovacuum ANALYZE chạy 02:10 trên bảng orders vừa vượt ngưỡng 10% row mới. Mẫu mới làm ước lượng `created_at > now()-'7 days'` đổi từ 80k row thành 350 row (dữ liệu 7 ngày cuối chưa vào histogram đủ dày — bài toán kinh điển **cột tăng đơn điệu**: thống kê luôn "hụt" phần mới nhất). Planner đổi hash join → nested loop; thực tế 80k row → 80k lần index probe vào bảng lớn → 35s.

**Điều tra:** auto_explain bắt plan lúc chậm; so `rows` vs `actual`; `pg_stats.histogram_bounds` cho thấy biên cuối < now()-7d.

**Khắc phục:** ANALYZE lại ngay (mẫu mới hơn); tăng `statistics_target` cho cột; lên lịch ANALYZE thường xuyên cho bảng append nặng; cân nhắc extended statistics hoặc partial index cho khoảng nóng.

**Bài học:** plan là hàm của thống kê; thống kê là mẫu của quá khứ. Cột thời gian tăng đơn điệu là kẻ thù tự nhiên của cả hai.

## 9. Tóm tắt

- Pipeline: Parser → Analyzer → Rewriter (view = macro) → Planner → Executor (Volcano pull).
- Cost model = số học trên thống kê mẫu; `random_page_cost` và `effective_cache_size` là hai nút chỉnh lớn nhất trên SSD.
- Giả định độc lập giữa cột và cột-thời-gian-tăng-đơn-điệu là hai nguồn ước lượng sai kinh điển; extended statistics là thuốc ít dùng.
- Đọc plan: tìm node đắt nhất → so ước lượng/thực tế → nhìn Buffers → tìm spill.
- pg_stat_statements + auto_explain = đôi mắt bắt buộc phải có trong production.

**Tiếp theo:** [Chương 9 — Index Internals](/series/postgres-internal/09-index-internals/): cấu trúc bên trong những access path mà planner vừa lựa chọn.
