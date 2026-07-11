+++
title = "Chương 3 — Physical Storage: Page, Tuple, TOAST, FSM, Visibility Map"
date = "2026-07-11T10:00:00+07:00"
draft = false
tags = ["backend", "database", "postgresql"]
series = ["PostgreSQL Internals"]
+++

> Level 2. Đây là chương nền tảng nhất của bộ tài liệu. Hiểu từng byte của heap page
> là hiểu được một nửa PostgreSQL: MVCC, VACUUM, HOT, bloat, index — tất cả đều quy về đây.

---

## 1. Problem Statement

Bảng `orders` 200GB phải nằm trên disk sao cho:

- Tìm một order theo con trỏ mất đúng 1 lần đọc disk.
- Thêm order mới không phải dịch chuyển dữ liệu cũ.
- Nhiều version của một row (MVCC) cùng tồn tại được.
- Crash giữa chừng không phá cấu trúc.
- Cột `note` dài 1MB không làm hỏng hiệu quả của các cột 8 byte bên cạnh.

Nếu tổ chức sai tầng này: mọi thao tác đọc thành O(n), UPDATE phải rewrite file, crash để lại file rác. Toàn bộ chương này là câu trả lời của PostgreSQL cho bài toán đó.

## 2. Từ Relation đến Block — chuỗi ánh xạ

```
Relation (bảng/index)  "orders"
   │  ánh xạ qua catalog: pg_class.relfilenode = 16385
   ▼
File vật lý           base/16384/16385, 16385.1, 16385.2 ...
   │  mỗi SEGMENT tối đa 1GB
   ▼
Block / Page          mỗi file = dãy page 8KB đánh số 0,1,2,...
   │  BlockNumber (uint32) → tối đa 2^32 × 8KB = 32TB / relation
   ▼
Tuple                 mỗi page chứa nhiều tuple (row version)
   │  định danh bằng ItemPointer = (BlockNumber, OffsetNumber)
   ▼                  ← chính là ctid mà bạn thấy trong SQL
Byte trên disk
```

**Tại sao segment 1GB?** Di sản từ thời filesystem không hỗ trợ file >2GB, giữ lại vì: copy/backup từng phần dễ, và một số filesystem vẫn xử lý file khổng lồ kém. Trade-off: bảng 1TB = 1000 file descriptor tiềm năng — PostgreSQL phải có LRU cache cho fd (`max_files_per_process`).

**Tại sao page 8KB?** Cân bằng giữa: (a) đơn vị I/O đủ lớn để amortize chi phí syscall; (b) đủ nhỏ để không phí RAM khi chỉ cần 1 tuple; (c) khớp bội số của block filesystem 4KB. Đổi được lúc compile (`--with-blocksize`) nhưng gần như không ai đổi — toàn bộ tuning ecosystem giả định 8KB. Lưu ý quan trọng: **8KB > 4KB atomic của disk → một page có thể bị ghi rách đôi (torn page)** → sinh ra cơ chế full page write ở Chương 5.

### Fork — một relation, nhiều file

Mỗi relation có tới 4 "fork" (dòng file song song):

| Fork | File | Nội dung | Nếu bỏ đi thì sao? |
|---|---|---|---|
| main | `16385` | Dữ liệu thật (heap/index page) | — |
| FSM | `16385_fsm` | Free Space Map: page nào còn bao nhiêu chỗ trống | INSERT phải quét tuần tự tìm chỗ trống → O(n) mỗi lần chèn |
| VM | `16385_vm` | Visibility Map: 2 bit/page — all-visible, all-frozen | Index-only scan bất khả thi; VACUUM phải đọc lại toàn bộ bảng mỗi lần |
| init | `16385_init` | Cho UNLOGGED table: ảnh trạng thái rỗng để reset sau crash | Unlogged table sau crash chứa rác thay vì rỗng sạch |

**FSM** là cây max-heap 3 tầng lưu lượng chỗ trống (độ phân giải 1/256 page) — tìm "page nào chứa vừa tuple X byte" trong O(log n) mà không cần lock toàn bảng. Nó chỉ là **gợi ý** (hint): không được WAL-log đầy đủ, sau crash có thể lạc hậu, tự sửa dần khi dùng.

**VM** quan trọng vượt cỡ của nó (2 bit/page — bảng 1TB chỉ cần VM 32MB): bit `all-visible` cho phép Index-Only Scan bỏ qua bước đọc heap (Chương 9), và cho VACUUM bỏ qua page không có gì để dọn (Chương 10). Bit `all-frozen` (PG9.6+) cho phép anti-wraparound vacuum bỏ qua page đã đóng băng — trước đó, vacuum freeze bảng 1TB phải đọc đủ 1TB.

## 3. Heap Page — giải phẫu 8192 byte

PostgreSQL dùng bố cục "slotted page" kinh điển, dữ liệu mọc từ hai đầu vào giữa:

```
 0                                                          8192
 ┌──────────┬────┬────┬────┬─────▶            ◀─────┬───────────┐
 │PageHeader│ lp1│ lp2│ lp3│  ...trống (hole)...    │  tuple3   │
 │ (24 byte)│(4B)│(4B)│(4B)│                        ├───────────┤
 └──────────┴────┴────┴────┘                        │  tuple2   │
      │        │                                    ├───────────┤
      │        └── line pointer: (offset,flags,len) │  tuple1   │
      │            trỏ tới tuple ở cuối page        └───────────┘
      │                                             ▲ (special space
      ▼                                               — chỉ index page)
 PageHeaderData:
   pd_lsn      (8B)  ← LSN của WAL record cuối sửa page này  ★WAL rule
   pd_checksum (2B)  ← checksum (nếu bật khi initdb)
   pd_flags    (2B)
   pd_lower    (2B)  ← ranh giới cuối mảng line pointer
   pd_upper    (2B)  ← ranh giới đầu vùng tuple
   pd_special  (2B)  ← đầu vùng special (B-tree dùng cho sibling links)
   pd_pagesize_version (2B)
   pd_prune_xid (4B) ← gợi ý: XID cũ nhất có thể prune được
```

Ba câu hỏi thiết kế:

**Tại sao cần line pointer (gián tiếp), không trỏ thẳng vào tuple?**
Vì tuple **di chuyển trong page** khi defragment (prune/vacuum gom hố trống), và tuple **bị thay thế** trong HOT update. Line pointer là địa chỉ ổn định: index bên ngoài trỏ vào `(block, lp số 3)` — tuple thật nằm đâu trong page là chuyện nội bộ. Bỏ nó đi → mỗi lần dọn page phải cập nhật mọi index trỏ vào page đó. Đây là quyết định mua sự tự do dọn dẹp nội-page với giá 4 byte/tuple.

**Tại sao pd_lsn nằm ngay đầu page?**
Đây là chốt của quy tắc WAL (Chương 5): *page chỉ được ghi xuống disk khi WAL record có LSN ≤ pd_lsn đã được flush*. Buffer manager kiểm tra field này trước mỗi lần ghi page.

**pd_checksum:** phát hiện hư hỏng do disk/RAM (bit rot). Tính khi ghi page ra disk, kiểm khi đọc lên. Chi phí ~vài % CPU. Không bật mặc định trước PG18 (initdb `--data-checksums`); production nên luôn bật — phát hiện corruption sớm rẻ hơn vô hạn lần so với phát hiện muộn.

## 4. Tuple — giải phẫu một row version

Đây là cấu trúc quan trọng nhất PostgreSQL. Mỗi tuple = header 23 byte (+ padding + null bitmap) + dữ liệu:

```
 ┌─────────────────────────── HeapTupleHeaderData ──────────────────────────┐
 │ t_xmin      (4B)  XID của transaction TẠO tuple này                      │
 │ t_xmax      (4B)  XID của transaction XÓA/UPDATE tuple này (0 = còn sống)│
 │ t_cid       (4B)  CommandId trong transaction (INSERT thứ mấy)           │
 │                   (dùng chung chỗ với t_xvac — vacuum full cũ)           │
 │ t_ctid      (6B)  ItemPointer: trỏ tới CHÍNH NÓ, hoặc version MỚI HƠN    │
 │                   → tạo thành "update chain"                             │
 │ t_infomask2 (2B)  số cột + cờ HOT (HEAP_HOT_UPDATED, HEAP_ONLY_TUPLE)    │
 │ t_infomask  (2B)  ★HINT BITS: XMIN_COMMITTED/ABORTED,                    │
 │                   XMAX_COMMITTED/ABORTED, HAS_NULL, ...                  │
 │ t_hoff      (1B)  offset tới dữ liệu thật (sau null bitmap + pad)        │
 │ [null bitmap]     1 bit/cột, chỉ có mặt nếu có NULL                      │
 │ [padding tới bội số MAXALIGN 8B]                                         │
 ├───────────────────────────── user data ──────────────────────────────────┤
 │ cột 1 │ cột 2 │ ...  (mỗi cột align theo typalign của kiểu)              │
 └───────────────────────────────────────────────────────────────────────────┘
```

### Những hệ quả trực tiếp lên đời engineer

**23 byte overhead mỗi row version.** Bảng 1 tỷ row "chỉ có 2 cột int" (8 byte data) thực tế tốn ≥ (23 header + 1 pad + 8 data + 4 line pointer) ≈ 36 byte/row ≈ 36GB chưa kể page overhead. So sánh: cùng dữ liệu trong file nhị phân phẳng: 8GB. Khoảng chênh đó mua: MVCC, crash safety, khả năng truy vấn.

**Alignment (padding) là tiền thật.** Cột được xếp đúng thứ tự khai báo, mỗi kiểu có yêu cầu align (int8/timestamp: 8B; int4: 4B; int2: 2B; text/varlena: 1B nhưng thường 4B):

```sql
-- Bảng A: (id int4, created timestamptz, flag bool, amount int8)
--   int4(4) + PAD(4) + ts(8) + bool(1) + PAD(7) + int8(8) = 32 byte
-- Bảng B: (created timestamptz, amount int8, id int4, flag bool)
--   ts(8) + int8(8) + int4(4) + bool(1) + PAD(3)          = 24 byte
-- → Cùng dữ liệu, tiết kiệm 25% chỉ bằng thứ tự cột.
```

Quy tắc thực dụng: **khai báo cột từ kiểu to align xuống nhỏ** (int8/timestamptz → int4 → int2 → bool → text/varlena).

**t_ctid và update chain.** Khi UPDATE, version cũ có `t_ctid` trỏ tới version mới:

```
UPDATE orders SET status='paid' WHERE id=7;

 page 42                                   page 42 (sau update, còn chỗ)
 ┌──────────────────────────┐              ┌──────────────────────────────┐
 │ lp1 → tuple v1           │              │ lp1 → tuple v1               │
 │   xmin=100 xmax=0        │    ──▶       │   xmin=100 xmax=205 ← bị "xóa"│
 │   ctid=(42,1)            │              │   ctid=(42,2) ───┐  bởi XID 205│
 │                          │              │ lp2 → tuple v2 ◀─┘           │
 │                          │              │   xmin=205 xmax=0            │
 └──────────────────────────┘              └──────────────────────────────┘
```

Cả hai version cùng tồn tại. Ai nhìn thấy version nào do MVCC quyết (Chương 6). Version cũ trở thành **dead tuple** khi không snapshot nào cần → mồi cho VACUUM (Chương 10). **Đây chính là nguồn gốc vật lý của table bloat.**

**Hint bits (t_infomask).** Để biết XID 100 đã commit chưa, phải tra CLOG — một lần đọc SLRU có thể miss cache. Làm cho *mỗi tuple, mỗi lần đọc* thì quá đắt. Giải pháp: lần đầu tra xong, **ghi kết quả vào chính tuple** (bit XMIN_COMMITTED). Lần sau đọc bit là xong.

Hệ quả kinh điển làm sửng sốt người mới: **SELECT có thể tạo ra write I/O**. Đọc một bảng vừa bulk-insert xong → backend set hint bit hàng loạt → page dirty hàng loạt → SELECT đầu tiên sau import chậm và gây bão ghi. Xử lý: chạy `VACUUM` (không FULL) sau bulk load để set hint bit + freeze chủ động.

## 5. TOAST — The Oversized-Attribute Storage Technique

**Problem:** tuple không được vượt ~2KB (để một page 8KB chứa được ≥4 tuple — con số chọn để B-tree index hoạt động tốt). Nhưng user cần lưu text/jsonb/bytea hàng MB.

**Giải pháp:** giá trị lớn bị (theo thứ tự ưu tiên):
1. **Nén** tại chỗ (pglz, hoặc lz4 từ PG14 — nhanh hơn đáng kể, nên dùng).
2. **Cắt lát** đưa sang bảng phụ `pg_toast.pg_toast_<oid>`, mỗi lát ~2000 byte; tuple gốc chỉ giữ con trỏ 18 byte (varatt_external: toast oid + value oid + size).

```
 heap tuple:  │ id │ status │ [TOAST pointer 18B] │
                                    │
                                    ▼
 pg_toast_16385 (bảng heap thường + index riêng):
   (chunk_id=9001, seq=0, data[~2000B])
   (chunk_id=9001, seq=1, data[~2000B])
   ...
```

4 chiến lược mỗi cột: `PLAIN` (không toast — kiểu fixed), `EXTENDED` (nén + external, mặc định), `EXTERNAL` (external không nén — tốt cho dữ liệu đã nén như ảnh), `MAIN` (ưu tiên nén, hạn chế external).

**Điểm hay bị bỏ qua trong production:**
- Đọc cột TOAST = thêm index scan + N lần đọc bảng toast → `SELECT *` trên bảng có jsonb to đắt hơn `SELECT id, status` **hàng chục lần**.
- UPDATE một cột thường trên row có TOAST **không** rewrite phần toast (con trỏ được copy) — nhưng UPDATE chính cột toast thì ghi lại toàn bộ giá trị (không có sửa-một-phần).
- Bảng toast cũng bloat, cũng cần vacuum, và **ẩn**: `pg_total_relation_size` mới thấy nó. Failure case #17 Chương 12.

## 6. Cách hoạt động: INSERT một row, từng bước, tới từng byte

```
INSERT INTO orders(id, amount) VALUES (7, 100);

1. Executor tạo tuple trong memory (palloc trong TupleTableSlot).
2. heap_insert() (src/backend/access/heap/heapam.c):
   a. Gán t_xmin = XID hiện tại, t_xmax = 0, t_cid = command id.
   b. Hỏi FSM: page nào còn ≥ len(tuple) byte trống?  → page 42
   c. Buffer Manager: pin + đọc page 42 vào shared buffers (Chương 4).
   d. LWLock exclusive lên buffer.
   e. PageAddItem(): 
      - cấp line pointer mới (pd_lower += 4)
      - copy tuple vào cuối vùng trống (pd_upper -= len)
   f. Ghi WAL record XLOG_HEAP_INSERT vào WAL buffer,
      nhận LSN, set pd_lsn của page = LSN đó.
   g. Đánh dấu buffer dirty, nhả LWLock, unpin.
3. COMMIT: flush WAL ≤ LSN đó + fsync; ghi bit "committed" vào CLOG.
4. (sau này) checkpointer/bgwriter ghi page 42 xuống disk.
5. (sau này) SELECT đầu tiên đọc tuple → tra CLOG thấy committed
   → set hint bit XMIN_COMMITTED → page dirty lần nữa.
```

Pseudo code Go cho bước 2e — nhìn rõ tính đơn giản của slotted page:

```go
func PageAddItem(p *Page, tuple []byte) (OffsetNumber, error) {
    need := align8(len(tuple))
    if int(p.pdUpper)-int(p.pdLower) < need+4 {
        return 0, ErrNoSpace // caller sẽ hỏi FSM page khác
    }
    p.pdUpper -= uint16(need)
    copy(p.data[p.pdUpper:], tuple)
    lp := LinePointer{Off: p.pdUpper, Len: uint16(len(tuple)), Flags: LP_NORMAL}
    p.writeLinePointerAt(p.pdLower, lp)
    p.pdLower += 4
    return p.lineCount(), nil
}
```

## 7. Trade-off: Heap vs Clustered Storage

Đây là khác biệt kiến trúc lớn nhất giữa PostgreSQL và MySQL/InnoDB:

| | PostgreSQL (Heap) | InnoDB (Clustered B-tree theo PK) |
|---|---|---|
| Bảng là gì | Đống page không thứ tự | Chính là B-tree, leaf chứa full row |
| Secondary index trỏ vào | ctid (vị trí vật lý) | Giá trị PK (phải đi 2 B-tree khi tra) |
| Đọc range theo PK | Cần index scan + heap fetch tứ tán | Tuyệt vời — dữ liệu liền kề vật lý |
| UPDATE non-key | Version mới có thể cùng page (HOT) — index không cần sửa | Sửa in-place trong B-tree + undo log |
| UPDATE làm row di chuyển | ctid đổi → nếu không HOT, MỌI index phải thêm entry | Secondary index không đổi (trỏ PK) |
| Page split của bảng | Không tồn tại (heap không split) | Có — insert giữa chừng gây split |
| Bloat | Ở heap, index — cần VACUUM | Undo log dài do transaction dài |

Không bên nào thắng tuyệt đối. PostgreSQL trả giá "index phải update khi row đổi chỗ" và giảm nhẹ nó bằng **HOT update** (Chương 10). InnoDB trả giá "mọi secondary index lookup đi qua 2 cây".

**"Nếu muốn clustered trong PostgreSQL?"** Lệnh `CLUSTER` sắp xếp lại bảng theo một index **một lần** (không duy trì), khóa bảng khi chạy. Đủ dùng cho bảng gần-như-chỉ-đọc; không phải clustered storage thật.

## 8. Production

- **Đo kích thước đúng cách:** `pg_relation_size` (main fork) < `pg_table_size` (+toast, fsm, vm) < `pg_total_relation_size` (+indexes). Bloat ẩn thường ở toast và index.
- **Fillfactor:** mặc định heap 100 (nhồi kín). Bảng UPDATE nhiều → hạ xuống 70–90 để chừa chỗ cho HOT update cùng page (Chương 10). Trả giá: bảng to hơn 10–30%, seq scan chậm hơn tương ứng.
- **Thứ tự cột:** như mục 4 — sắp theo align giảm dần khi tạo bảng mới. Với bảng tồn tại, chỉ sửa được bằng rewrite.
- **Checksum:** luôn bật `--data-checksums` từ đầu (PG18 mặc định bật); bật sau bằng `pg_checksums` yêu cầu downtime.
- **Theo dõi:** extension `pgstattuple` cho tỷ lệ dead tuple/free space thật (đọc cả bảng — cẩn thận với bảng lớn); `pg_freespacemap` xem FSM.

## 9. Failure case của chương: "bảng 10GB nhưng dữ liệu thật chỉ 1GB"

**Triệu chứng:** Seq scan chậm dần theo tuần dù số row không tăng. `pg_relation_size` = 10GB; `SELECT count(*)` như cũ.

**Root cause:** Table bloat — UPDATE/DELETE để lại dead tuple; VACUUM có dọn nhưng **heap không bao giờ tự co lại** (chỉ trả chỗ trống cho FSM; page cuối file chỉ được cắt nếu hoàn toàn rỗng). Pattern "UPDATE toàn bảng mỗi đêm" là thủ phạm kinh điển.

**Bên trong:** mỗi UPDATE chèn version mới có thể vào page mới cuối bảng → high-water mark tăng; version cũ thành dead → vacuum biến thành hố trống rải rác giữa file — seq scan vẫn phải đọc đủ 10GB page.

**Điều tra:** `pgstattuple('orders')` → xem `dead_tuple_percent` + `free_percent`; `pg_stat_user_tables.n_dead_tup`, `last_autovacuum`.

**Khắc phục:** `VACUUM FULL` (rewrite, khóa exclusive — cần cửa bảo trì) hoặc `pg_repack` (online). **Phòng tránh:** sửa pattern update-toàn-bảng; fillfactor; autovacuum đủ hung hãn (Chương 10).

## 10. Tóm tắt

- Relation → segment 1GB → page 8KB → tuple, định danh bằng ctid=(page, line pointer).
- Line pointer mua sự tự do dọn dẹp nội page; pd_lsn là chốt của WAL rule; hint bit khiến cả SELECT cũng ghi.
- Tuple header 23 byte là giá của MVCC; alignment là tiền thật; TOAST giấu chi phí ở bảng phụ.
- Heap không clustered: không page split cho bảng, nhưng UPDATE đổi chỗ row bắt mọi index trả giá — thứ HOT sẽ cứu.
- Heap không bao giờ tự co: bloat chỉ xử lý được bằng rewrite.

**Tiếp theo:** [Chương 4 — Buffer Manager](/series/postgres-internal/04-buffer-manager/): các page 8KB này sống trong RAM như thế nào.
