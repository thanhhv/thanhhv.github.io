+++
title = "Chương 13 — Anti-pattern & Khi nào PostgreSQL không phù hợp"
date = "2026-07-11T20:00:00+07:00"
draft = false
tags = ["backend", "database", "postgresql"]
series = ["PostgreSQL Internals"]
+++

> Level 5. Chương kết. Phần một: các cách dùng sai lặp đi lặp lại — mỗi cái là một
> chương lý thuyết bị vi phạm. Phần hai: ranh giới kiến trúc thật sự của PostgreSQL —
> nơi vấn đề không phải "dùng sai" mà là "chọn sai công cụ".

---

## Phần 1 — Anti-patterns

Mỗi anti-pattern dưới đây đều có chung cấu trúc: *tiện trước mắt → vi phạm một cơ chế nội tại → trả giá trễ và trả bằng lãi kép*.

### 1.1. Long transaction (và `idle in transaction`)
Vi phạm: xmin horizon (Ch.6 §5). Một transaction 4 giờ ghim rác toàn cluster; một connection `idle in transaction` do ORM quên commit còn tệ hơn — nó vô hạn.
Chữa: bộ ba timeout là cấu hình bắt buộc, không phải tùy chọn; job dài → replica/chia nhỏ.

### 1.2. Bảng làm queue tần suất cao (naive)
Vi phạm: MVCC — mỗi job = INSERT + UPDATE(s) + DELETE = 3+ version chết/job; polling `SELECT ... LIMIT 1 FOR UPDATE` không SKIP LOCKED = hàng đợi lock (Ch.7). Bảng queue 1000 job/s tạo ~10GB rác/ngày và autovacuum vĩnh viễn đuổi theo.
Chữa đúng cấp độ: SKIP LOCKED + fillfactor thấp + vacuum hung hãn (đủ cho vừa); partition xoay vòng; ngoài một ngưỡng — Kafka/SQS là công cụ đúng.

### 1.3. Over-indexing
Vi phạm: mỗi index = thuế trên mọi INSERT/UPDATE (Ch.9) + phá HOT (Ch.10 §2, case #12) + chậm vacuum (Ch.10 §3 pha 2). "Cứ chậm là thêm index" không có bước "trả nợ" là chiến lược thua chắc.
Chữa: kiểm toán idx_scan hàng quý; mỗi index mới trên bảng nóng phải qua câu hỏi "cột này có bị update?" và "partial được không?".

### 1.4. Thiếu ANALYZE sau thay đổi dữ liệu lớn
Vi phạm: planner mù (Ch.8 §3.2) — sau migration/bulk load, thống kê tả dữ liệu đã chết. Plan thảm họa đầu giờ sáng hôm sau.
Chữa: `ANALYZE` là dòng cuối của mọi script bulk. Không thương lượng.

### 1.5. Large tuple / lạm dụng jsonb
Vi phạm: TOAST (Ch.3 §5, case #17) — jsonb 500KB update mỗi phút = ghi lại 500KB + WAL tương ứng mỗi lần đổi 1 key; và mọi index GIN trên đó nhận bill (Ch.9 §4).
Chữa: jsonb cho dữ liệu *thật sự* schemaless và ít đổi; key nóng → cột thường; blob → object storage.

### 1.6. Connection bừa bãi
Vi phạm: process model + ProcArray (Ch.2). "Mỗi pod một pool 20, autoscale lên 80 pod" = 1600 connection = snapshot đắt cho tất cả.
Chữa: PgBouncer tầng giữa; ngân sách connection là tài nguyên có cấp phát, như RAM.

### 1.7. UPDATE toàn bảng định kỳ
Vi phạm: MVCC nhân đôi bảng mỗi lần chạy (Ch.3 §9). `UPDATE items SET score=recalc()` mỗi đêm trên 100M row = 100M dead tuple/đêm.
Chữa: chỉ update row thực sự đổi (`WHERE score IS DISTINCT FROM recalc()`); hoặc bảng score riêng append-only; hoặc tính lúc đọc.

### 1.8. DDL không lock_timeout
Vi phạm: hàng đợi FIFO của lock (Ch.7 §4.1 định luật 2) — một ALTER xếp sau một SELECT dài chặn toàn bộ traffic phía sau nó.
Chữa: khuôn migration chuẩn có SET lock_timeout + retry; những dạng ALTER an toàn/nguy hiểm phải nằm trong tri thức chung của team (không chỉ DBA).

### 1.9. Tắt hoặc bóp autovacuum "cho đỡ tốn IO"
Vi phạm: toàn bộ Chương 10. Nợ không biến mất — nó dồn thành anti-wraparound 30 giờ đúng mùa cao điểm (case #4, #13).
Chữa: chiều ngược lại mới đúng: autovacuum mặc định quá HIỀN — hãy tháo phanh, đừng siết.

### 1.10. Tin `count(*)`, `SELECT *` và OFFSET sâu
Ba thói quen nhỏ, chung một gốc: không hiểu chi phí vật lý. `count(*)` = quét visibility (Ch.6 §8); `SELECT *` = kéo TOAST oan (Ch.3 §5); `OFFSET 100000` = đọc và VỨT 100k row (executor không có cách nhảy cóc — Ch.8 §4). Chữa: ước lượng reltuples; liệt kê cột; keyset pagination (`WHERE id > $last ORDER BY id`).

---

## Phần 2 — Khi nào PostgreSQL không phù hợp

Nguyên tắc đánh giá trung thực: PostgreSQL là **generalist xuất sắc** — mặc định đúng cho 90% hệ thống, và ranh giới của nó lùi xa sau mỗi version. Nhưng có những bài toán mà *kiến trúc* (heap + MVCC + B-tree + single-node write + row store) là sai về căn bản, không tune nổi.

### 2.1. Time series khối lượng cực lớn
**Vướng kiến trúc nào:** heap + tuple header 23 byte/điểm dữ liệu (Ch.3) — điểm đo 16 byte mang overhead 150%; B-tree phình theo append (Ch.9); nén theo cột không có; retention bằng DELETE = bloat (Ch.10).
**Ngưỡng thực dụng:** đến ~vài chục nghìn điểm/giây và giữ vài tháng: PostgreSQL + partition + BRIN vẫn ổn. TimescaleDB (extension) đẩy xa hơn nhiều (nén cột, chunk tự động). Hàng triệu điểm/giây, retention năm: ClickHouse/InfluxDB/định dạng cột chuyên dụng.

### 2.2. Analytics / OLAP nặng
**Vướng:** row store — đọc 3 cột của bảng 100 cột vẫn nạp đủ page chứa row (Ch.3); executor Volcano 1-tuple/lần đắt CPU (Ch.8 §4); không vectorization, không nén cột; một query aggregate = ring buffer + đọc cả bảng (Ch.4).
**Chênh lệch đại diện:** aggregate trên 1 tỷ row: PostgreSQL hàng chục phút, ClickHouse/DuckDB giây — không phải tune kém, là kiến trúc khác.
**Mẫu đúng:** OLTP ở PostgreSQL, CDC (logical replication — Ch.11 §5.3) sang kho cột. Đừng bắt một engine làm cả hai ở quy mô lớn.

### 2.3. Document database thuần
jsonb rất tốt để *pha* document vào hệ quan hệ. Nhưng nếu 100% workload là document lớn, sửa từng phần, sharding tự nhiên → mọi write ghi lại toàn bộ document qua TOAST (Ch.3 §5), GIN đắt (Ch.9 §4). MongoDB/hệ document có delta-update và sharding gốc phù hợp hơn. (Ngược lại: nếu dữ liệu có quan hệ và cần transaction — jsonb trong PostgreSQL thường thắng Mongo. Ranh giới nằm ở tỷ trọng workload.)

### 2.4. Graph traversal sâu
Recursive CTE xử được đồ thị nông. Truy vấn "bạn của bạn của bạn, 6 bậc, có trọng số" = join lặp × N bậc, mỗi bậc một lần B-tree/hash — không có index-free adjacency. Neo4j/hệ graph lưu con trỏ kề trực tiếp. Ngưỡng: sâu >3–4 bậc và đồ thị lớn.

### 2.5. Write throughput vượt một node (high write log/ingest)
**Vướng cứng nhất:** một dòng WAL duy nhất (Ch.5 §6) + một primary nhận ghi. Physical replication scale ĐỌC, không scale GHI. Trần thực dụng đại diện: vài chục nghìn transaction ghi/giây có fsync trên phần cứng tốt.
Vượt trần: Citus (sharding trên PostgreSQL), CockroachDB/Spanner-class (trả giá bằng latency đồng thuận), Cassandra (trả bằng mô hình nhất quán), hoặc Kafka đứng trước lớp bền. Mỗi lựa chọn từ bỏ một phần đảm bảo mà PostgreSQL cho không: transaction đa row tùy ý, join tự do, snapshot nhất quán toàn cục.

### 2.6. Các trường hợp nhỏ nhưng hay gặp
- **Cache/session store TTL cao tần:** ghi-xóa liên tục = máy sản xuất dead tuple. Redis.
- **Full-text search ngôn ngữ phức tạp/tính năng search-engine:** tsvector+GIN tốt đến mức "đủ dùng"; relevance tuning, faceting, typo-tolerance → Elasticsearch/OpenSearch/Meilisearch.
- **Vector search:** pgvector đưa PostgreSQL vào cuộc chơi nghiêm túc (HNSW); ở hàng trăm triệu vector + QPS cao, hệ chuyên dụng vẫn dẫn.

### 2.7. Khung quyết định 5 câu hỏi

```
1. Write có vượt một node (sau khi đã pooling, batch, tune)? 
   → Chưa: PostgreSQL. Rồi: shard/hệ phân tán, chấp nhận mất gì?
2. Workload là scan-hàng-tỷ-row-vài-cột?
   → OLAP thật: kho cột. PostgreSQL chỉ giữ vai OLTP + CDC ra.
3. Dữ liệu có cấu trúc quan hệ và cần transaction đa đối tượng?
   → Có: điểm cộng lớn cho PostgreSQL, kể cả khi có document/vector lẫn vào.
4. Access pattern có "đặc chủng" không (graph sâu, time series tỷ điểm, TTL cao tần)?
   → Có: công cụ chuyên dụng, PostgreSQL làm hệ ghi chính (system of record).
5. Team đã vận hành giỏi bao nhiêu hệ? 
   → Mỗi database thêm vào stack là một nghề vận hành mới. "PostgreSQL cho tới khi
     có số liệu chứng minh điều ngược lại" là default đúng của đa số công ty.
```

---

## Lời kết của bộ tài liệu

Nhìn lại toàn bộ 13 chương, PostgreSQL hiện ra đúng như lời hứa ở README: một hệ điều hành thu nhỏ cho dữ liệu — có memory manager (buffer pool), có scheduler (background processes), có journaling (WAL), có garbage collector (VACUUM), có IPC (shared memory + lock), có compiler/optimizer (planner).

Và xuyên suốt là một số ít nguyên lý lặp đi lặp lại:

1. **Mọi thứ quy về page 8KB và trục LSN.** Hiểu hai thứ đó, các cơ chế còn lại là hệ quả.
2. **Trì hoãn là chiến lược trung tâm:** commit chỉ ghi WAL (data file sau), xóa chỉ đánh dấu (vacuum sau), hint bit ghi khi tiện. Trì hoãn mua hiệu năng và concurrency — và mọi khoản trì hoãn đều là nợ có ngày đáo hạn. Vận hành PostgreSQL = quản trị các khoản nợ đó.
3. **Không có thiết kế đúng, chỉ có trade-off được chọn có ý thức.** Heap vs clustered, MVCC vs lock, process vs thread, 25% vs 80% RAM — giá trị của việc hiểu internals không phải để khen chê lựa chọn, mà để **dự đoán được nó gãy ở đâu trước khi nó gãy**.

Tài liệu đọc tiếp được khuyến nghị: *The Internals of PostgreSQL* (Hironobu Suzuki, interdb.jp — miễn phí), *PostgreSQL 14 Internals* (Egor Rogov, PDF miễn phí của Postgres Professional), source code (`src/backend/` — README trong mỗi thư mục là tài liệu hạng nhất), và pgsql-hackers mailing list — nơi mọi trade-off trong tài liệu này được tranh luận công khai suốt 30 năm.
