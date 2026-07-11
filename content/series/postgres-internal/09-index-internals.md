+++
title = "Chương 9 — Index Internals: B-tree, Hash, GIN, GiST, BRIN"
date = "2026-07-11T16:00:00+07:00"
draft = false
tags = ["backend", "database", "postgresql"]
series = ["PostgreSQL Internals"]
+++

> Level 4. Index là cấu trúc dữ liệu trên disk — nghĩa là mọi thiết kế của nó
> bị chi phối bởi page 8KB, WAL, MVCC và concurrency. Chương này mở nắp từng loại.

---

## 1. Problem Statement

Heap không có thứ tự (Chương 3): tìm `WHERE email='x@y.vn'` = đọc toàn bộ bảng, O(N) page. Bảng 100GB → ~13 giây chỉ riêng I/O tuần tự, mỗi query. Cần cấu trúc phụ ánh xạ `giá trị → ctid` với chi phí tra cứu ~O(log N) **tính bằng số page** (không phải số phần tử — mỗi page là một lần I/O tiềm năng).

Nhưng index không miễn phí, và đây là phần người ta hay quên định lượng:

- Mỗi INSERT phải chèn vào **mọi** index của bảng (n index = n+1 lần ghi + WAL).
- UPDATE làm row đổi chỗ (non-HOT) cũng vậy.
- Index cũng bloat, cũng cần vacuum, cũng chiếm cache.
- Index sai còn tệ hơn không có: planner có thể chọn nó và chậm hơn seq scan.

## 2. Nguyên lý chung cho mọi index PostgreSQL

1. **Index không chứa thông tin visibility.** Entry trong index trỏ ctid; tuple sống hay chết phải nhảy vào heap mới biết (hoặc VM nói cả page all-visible — index-only scan). Hệ quả: index không thể trả lời query một mình khi bảng nhiều dead tuple. *Tại sao không nhét xmin/xmax vào index?* — mỗi UPDATE sẽ phải sửa mọi index kể cả HOT, nhân đôi write amplification. Trade-off nghiêng về giữ index "ngu".
2. **Mọi index đều qua Buffer Manager + WAL** như heap page — crash-safe, replica có đủ.
3. **Access method là plugin** (`pg_am`): B-tree, hash, GIN... cùng một interface (build, insert, scan, vacuum) — kiến trúc mở giúp bristlecone/bloom/rum tồn tại được ngoài core.

## 3. B-tree — 90% số index trên đời

### 3.1. Cấu trúc trên page 8KB

```
                        ┌────────── meta page (block 0) ──────────┐
                        │ trỏ tới root, thông tin version         │
                        └──────────────────┬──────────────────────┘
                                     ┌─────▼─────┐
                 internal            │   root    │ [k<100 | k<500 | ...]
                 pages               └─┬───┬───┬─┘
                        ┌──────────────┘   │   └──────────────┐
                  ┌─────▼─────┐      ┌─────▼─────┐      ┌─────▼─────┐
                  │ internal  │      │ internal  │      │ internal  │
                  └─┬───────┬─┘      └───────────┘      └───────────┘
              ┌─────▼─┐   ┌─▼─────┐
    leaf      │ leaf  │◀─▶│ leaf  │◀─▶ ... (doubly-linked qua pd_special!)
    level     │(k,ctid)│  │(k,ctid)│
              └───────┘   └───────┘
```

- Fan-out lớn: page 8KB chứa hàng trăm key → bảng 1 tỷ row: **cây cao 3–4 tầng**. Root + internal gần như luôn trong cache → 1 lần tra = thực tế ~1 lần I/O (leaf).
- **Leaf liên kết đôi** (qua vùng `pd_special` — Chương 3) → range scan = lướt ngang leaf, không phải quay lại cây.
- PG12+ (Lehman-Yao có cải tiến): key trùng lặp lưu dạng **deduplicated posting list** (PG13) — index trên cột ít giá trị (status) nhỏ đi 3–5 lần.

### 3.2. Page split — và tại sao nó không bao giờ co lại

Leaf đầy → **split**: cấp page mới, chia key 50/50 (hoặc 90/10 nếu chèn vào cực phải — tối ưu cho cột tăng đơn điệu như id/created_at), đẩy separator key lên cha; cha đầy → split lan lên. Split được WAL-log cẩn thận để crash giữa chừng không làm gãy cây (right-link của Lehman-Yao cho phép reader "đi vòng" qua split dở).

**B-tree không merge page khi xóa.** Page rỗng hoàn toàn được vacuum thu hồi (recycle), nhưng page còn 1 key vẫn đứng đó. Hệ quả: pattern "index trên cột thời gian + DELETE dữ liệu cũ theo batch nhưng giữ lại rải rác" → index thưa như tổ ong → **index bloat không tự hết** (REINDEX là câu trả lời — mà từ PG12 có `REINDEX CONCURRENTLY`).

*Tại sao không merge?* — merge yêu cầu lock nhiều page + cha, phức tạp concurrency tăng vọt, trong khi thống kê cho thấy đa số workload key mới sẽ lấp lại chỗ cũ. Trade-off: đơn giản + nhanh cho 95%, trả bằng REINDEX định kỳ cho 5%.

### 3.3. Concurrency: Lehman-Yao right-link

Vấn đề: reader đang lần từ cha xuống con thì con bị split, key nó tìm chạy sang page mới. Giải của Lehman-Yao: mọi page có **right-link** + **high key** (chặn trên của page). Reader tới page, thấy key tìm kiếm > high key → "à, đã có split, đi theo right-link". Kết quả: **tra cứu không cần khóa toàn đường đi** — chỉ giữ lock 1 page mỗi lúc. Đây là lý do B-tree PostgreSQL chịu được hàng triệu lookup/giây.

## 4. Các index còn lại — mỗi loại giải một bài toán mà B-tree bó tay

### Hash
- Ánh xạ hash(key) → bucket page. Chỉ hỗ trợ `=`.
- Từng vô dụng (pre-PG10 không WAL-log!). Nay: hợp lệ, nhỉnh hơn B-tree chút cho key rất dài chỉ tra `=` (URL, UUID dài) vì lưu hash 4 byte thay full key.
- Thực dụng: hiếm khi đáng dùng — B-tree deduplicated đủ tốt và đa năng hơn.

### GIN — Generalized Inverted Index
- Bài toán: giá trị **kép hợp** (mảng, jsonb, tsvector) — một row chứa nhiều "phần tử", query hỏi "row nào chứa phần tử X".
- Cấu trúc: B-tree của **phần tử** → mỗi phần tử trỏ posting list/tree các ctid. Đúng nghĩa "inverted index" như search engine.
- Giá phải trả: **INSERT cực đắt** (1 row jsonb 20 key = 20 entry). Giảm đau bằng `fastupdate`: dồn vào pending list, merge sau — nhưng pending list làm SELECT chậm dần và được dọn bởi... autovacuum. Ghi nặng + GIN = theo dõi `gin_pending_list_limit`.
- Không hỗ trợ index-only scan, thường vô dụng cho ORDER BY.

### GiST — Generalized Search Tree
- Framework cây cân bằng cho dữ liệu **không có thứ tự tuyến tính**: hình học (R-tree behavior), range type, full-text gần đúng, KNN (`ORDER BY point <-> ?`).
- Internal node chứa **predicate bao trùm** con (bounding box) — tra cứu có thể phải xuống **nhiều nhánh** (khác B-tree). Kém chính xác hơn GIN cho membership, nhưng hỗ trợ KNN và kiểu dữ liệu lạ.
- PostGIS đứng trên GiST — lý do PostgreSQL thống trị mảng geospatial.

### BRIN — Block Range Index
- Ý tưởng ngược đời: **không index từng row** — lưu min/max cho mỗi dải 128 page. Query range → loại các dải chắc chắn không chứa → seq scan phần còn lại.
- Kích thước lố bịch: bảng 1TB → BRIN ~vài MB (so với B-tree ~50GB).
- Chỉ hiệu quả khi **thứ tự vật lý tương quan giá trị** (correlation ~1): bảng append-only theo thời gian. Một UPDATE làm row "lạc dải" là min/max nở ra → index mất dần tác dụng, âm thầm.

### Bảng chọn nhanh

| Bài toán | Index |
|---|---|
| So sánh/range/sort thông thường | B-tree (mặc định đúng 90%) |
| jsonb `@>`, mảng, full-text chính xác | GIN |
| Geospatial, range type, KNN | GiST (PostGIS) |
| Bảng log khổng lồ append-only, query theo khoảng thời gian | BRIN |
| `=` trên key rất dài | Hash (cân nhắc) |

## 5. Bên dưới: index và MVCC — nơi mọi thứ giao nhau

Chuỗi sự kiện khi UPDATE **không** HOT (row đổi page hoặc cột index đổi):

```
UPDATE user SET email='new@x.vn' WHERE id=7;   (email có index)

 heap:   tuple v1 (42,1) xmax=205        ── v2 chèn vào (91,3)
 btree email: entry 'old@x.vn' → (42,1)  GIỮ NGUYÊN (ai dọn? vacuum!)
              entry 'new@x.vn' → (91,3)  THÊM MỚI
 btree id:    entry 7 → (42,1)           GIỮ NGUYÊN
              entry 7 → (91,3)           THÊM MỚI  ← dù id KHÔNG đổi!
```

Ba hệ quả:

1. **Mọi index đều phình theo update**, kể cả index trên cột không đổi (trừ khi HOT — Chương 10).
2. Index scan có thể lội qua hàng loạt entry trỏ tuple chết → "index đúng mà vẫn chậm". `EXPLAIN (ANALYZE, BUFFERS)` thấy Buffers cao bất thường so số row trả.
3. PostgreSQL vá bằng hai cơ chế vi mô: **killed tuple hint** (index scan phát hiện tuple chết → đánh dấu entry LP_DEAD, lần sau khỏi vào heap) và **bottom-up deletion** (PG14: trước khi split, thử dọn entry chết trong page — cứu được rất nhiều split "oan" do update).

## 6. Trade-off tổng

| Quyết định | Được | Mất |
|---|---|---|
| Index trỏ ctid (vị trí vật lý) | Lookup 1 bước; không phụ thuộc PK | Row di chuyển = sửa mọi index (nguồn của HOT) |
| Index "không biết" visibility | Nhẹ, ghi ít | Luôn cần heap/VM xác minh; count(*) không rẻ như tưởng |
| Split không merge | Concurrency đơn giản, nhanh | Index bloat một chiều — cần REINDEX |
| Access method mở | Hệ sinh thái (PostGIS, pgvector) | Chất lượng không đều giữa các AM |

## 7. Production

- **Tìm index thừa:** `pg_stat_user_indexes.idx_scan = 0` (sau thời gian đủ dài, kiểm tra cả replica!) → mỗi cái đang đánh thuế mọi INSERT/UPDATE. Nhưng đừng xóa index enforce constraint (unique) và index phục vụ query hiếm-mà-sống-còn (báo cáo cuối tháng).
- **Tìm index thiếu:** `pg_stat_user_tables.seq_scan` cao trên bảng lớn + `pg_stat_statements` lọc query đọc nhiều buffer.
- **Index bloat:** so `pgstatindex('idx').avg_leaf_density` (lành mạnh ~70–90%; <50% là bloat); REINDEX CONCURRENTLY định kỳ cho index churn cao.
- **Partial + covering index** — hai vũ khí ít dùng: `CREATE INDEX ... WHERE status='active'` (index 5% bảng, phục vụ 95% query); `INCLUDE (col)` cho index-only scan không phình key.
- **Cột tăng đơn điệu (id, created_at):** mọi insert dồn vào leaf cực phải → tranh chấp BufferContent trên đúng 1 page ở QPS rất cao. Nhận diện qua wait event; giải bằng hash-sharded key hoặc UUID v7 (còn UUID v4 thì ngược lại: random chèn khắp cây → cache miss + split khắp nơi + WAL FPI nhiều — chọn bên đau chịu được).

## 8. Failure case của chương: index vẫn còn, query vẫn đúng — chậm gấp 40 lần sau một năm

**Triệu chứng:** lookup theo `idx_events_ref` từ 0.3ms lên 12ms sau một năm vận hành. Số row trả về không đổi.

**Root cause:** bảng events UPDATE trạng thái 3–5 lần mỗi row đời sống + DELETE theo tháng. Index bloat: leaf density 31%, cây cao thêm 1 tầng, entry chết chi chít — mỗi lookup đọc 15 page thay 4, một nửa dẫn vào tuple chết.

**Điều tra:** `pgstatindex` (density), `pg_stat_user_indexes.idx_tup_read` >> `idx_tup_fetch` (đọc entry nhiều hơn số tuple sống lấy được), EXPLAIN BUFFERS.

**Khắc phục:** `REINDEX INDEX CONCURRENTLY idx_events_ref` (PG12+; cần disk tạm ~bằng index, chạy ngoài giờ cao điểm). **Phòng:** autovacuum đủ nhanh trên bảng này (để killed-hint/bottom-up deletion còn tác dụng), partition theo tháng để DELETE thành DROP PARTITION (không sinh rác index).

## 9. Tóm tắt

- B-tree: fan-out lớn → cây 3–4 tầng cho tỷ row; leaf linked-list cho range; Lehman-Yao cho concurrency không khóa đường đi; split không merge → bloat một chiều.
- GIN cho phần tử-trong-tập, GiST cho không gian/KNN, BRIN cho append-only khổng lồ, Hash gần như để đó.
- Index mù visibility và trỏ vị trí vật lý — hai quyết định nền tạo ra cả HOT lẫn nhu cầu vacuum index.
- Mỗi index là thuế trên mọi lệnh ghi — kiểm toán idx_scan định kỳ.

**Tiếp theo:** [Chương 10 — VACUUM](/series/postgres-internal/10-vacuum-hot-freeze/): trả món nợ mà MVCC + index đã vay suốt sáu chương.
