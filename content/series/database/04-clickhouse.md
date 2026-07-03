+++
title = "Bài 4 — ClickHouse"
date = "2026-07-02T12:00:00+07:00"
draft = false
tags = ["backend", "database"]
series = ["Database Thực Chiến"]
+++

# ClickHouse

> **Tiền đề:** Chương 1 (columnar vs row, LSM, vectorized execution) và các chương PostgreSQL/MongoDB để đối chiếu.
> **Chủ đề trung tâm của chương:** ClickHouse nhanh không phải vì "code giỏi hơn", mà vì nó **từ bỏ một cách có chủ đích** những thứ OLAP không cần.

---

## 1. Problem Statement

**Bài toán:** trả lời câu hỏi phân tích (aggregation, group by, filter) trên **hàng tỷ đến hàng nghìn tỷ row** trong **dưới một giây**, với dữ liệu mới đến liên tục.

**Vì sao RDBMS thất bại ở bài toán này — về mặt cấu trúc, không phải tuning:**

Xét query: `SELECT avg(price) FROM orders WHERE date >= '2026-01-01'` trên bảng 1 tỷ row, 50 cột.

- **Row storage đọc thừa 49 cột.** PostgreSQL lưu cả row liền nhau; muốn đọc 2 cột vẫn phải kéo cả page chứa đủ 50 cột qua I/O và cache. Đọc thừa ~96% dữ liệu.
- **Executor row-at-a-time:** 1 tỷ lần lặp qua chuỗi operator, mỗi giá trị một lượt function call — CPU nghẹt vì overhead điều phối, không phải vì phép cộng.
- **B+Tree index không cứu được:** điều kiện chọn 30% của 1 tỷ row → 300 triệu random access, chậm hơn cả full scan; planner sẽ đúng đắn chọn full scan — và full scan row-oriented thì như trên.
- **Nén kém:** row chứa kiểu dữ liệu trộn lẫn → entropy cao → nén tỷ lệ thấp → I/O nhiều.

Kết luận: không có mức tuning nào đổi được cấu trúc. Cần engine thiết kế lại từ số 0 cho câu hỏi "aggregate vài cột trên rất nhiều row".

---

## 2. Tại sao ClickHouse tồn tại

**Bối cảnh:** Yandex.Metrica (đối thủ Google Analytics của Nga) cần đếm/phân tích hàng chục tỷ event mỗi ngày, dashboard tương tác cho khách — nghĩa là **truy vấn ad-hoc dưới giây trên dữ liệu thô**, không phải báo cáo tính trước. Các lựa chọn đương thời: pre-aggregate (mất tính ad-hoc), Hadoop/MapReduce (phút-giờ, không tương tác), warehouse thương mại (đắt). Họ xây engine riêng, open-source năm 2016.

**Ba nguyên tắc thiết kế xuyên suốt (hiểu ba điều này là hiểu ClickHouse):**

1. **Columnar tuyệt đối** — I/O tỷ lệ với *cột cần đọc*, không phải *độ rộng bảng*.
2. **Ăn hết phần cứng** — vectorized + SIMD + parallel mọi core; một query dùng 100% CPU của cả máy là *hành vi thiết kế*, không phải sự cố.
3. **Từ bỏ những gì OLAP không cần:** không transaction đa statement, không UPDATE/DELETE nhanh, không đọc điểm hiệu quả, không constraint — đổi tất cả lấy tốc độ scan/aggregate.

---

## 3. Cách hoạt động bên trong

### 3.1. Columnar Storage

Mỗi cột một (nhóm) file, giá trị cùng cột nằm liền nhau:

```
Row storage (PG):   [id,date,user,price,...cột 5..50][id,date,...] ...
Column storage (CH):
  id.bin:    1,2,3,4,...
  date.bin:  2026-01-01, 2026-01-01, ...
  price.bin: 10.5, 20.0, ...
```

Ba hệ quả vật lý:

- Query chạm k cột chỉ đọc k file — **I/O giảm theo tỷ lệ cột dùng/tổng cột** (bảng event 300 cột, query 3 cột → giảm ~100 lần trước khi làm gì khác).
- **Nén cực tốt:** dữ liệu cùng kiểu, cùng phân bố nằm liền nhau — entropy thấp. Tỷ lệ 5–20x là bình thường (so với 2–4x của row storage). Codec chuyên dụng theo bản chất dữ liệu: `Delta`/`DoubleDelta` (timestamp tăng dần → hiệu số gần bằng nhau → nén sâu), `Gorilla` (float dao động nhỏ), `LowCardinality` (dictionary encoding cho cột ít giá trị phân biệt — country, status).
- **Cache locality + SIMD:** mảng giá trị cùng kiểu liên tục → CPU xử lý 8–16 giá trị/lệnh.

Và một cái giá cấu trúc: **đọc/ghi một row hoàn chỉnh phải chạm N file cột** → đọc điểm và ghi lẻ tẻ là điều tệ nhất có thể làm với columnar engine.

### 3.2. MergeTree — trái tim của ClickHouse

Họ engine chủ lực. Nguyên lý giống LSM (Chương 1) nhưng tối giản cho OLAP:

```
INSERT (batch) ──► ghi ngay thành một PART mới trên disk
                   (immutable: dữ liệu sort theo ORDER BY key, nén, kèm index)

Background merge: part nhỏ + part nhỏ ──► part lớn hơn (sort-merge, tuần tự)

SELECT: đọc trên mọi part hiện có, hợp kết quả
```

Khác LSM kinh điển: **không có memtable** — insert ghi thẳng thành part. Hệ quả trực tiếp và quan trọng bậc nhất về vận hành:

> **Mỗi INSERT = ít nhất một part mới.** Insert từng row một, 1000 lần/giây = 1000 part/giây → merge không kịp → lỗi "too many parts" → sập ghi. **INSERT PHẢI THEO BATCH** (nghìn–triệu row/lần) hoặc qua async_insert/Kafka engine/Buffer table. Đây là quy tắc số một của ClickHouse.

**Part** = thư mục chứa: file `.bin` từng cột (nén), primary index, marks (ánh xạ granule → vị trí file), metadata partition. Immutable — mọi "thay đổi" đều là tạo part mới.

### 3.3. Sparse Primary Index — index thưa, không phải B+Tree

Dữ liệu trong part đã **sort theo `ORDER BY` key**. Index chỉ ghi giá trị key **mỗi 8192 row** (một **granule**):

```
Bảng 1 tỷ row → index chỉ ~122.000 entry → vài MB, nằm gọn trong RAM

SELECT ... WHERE user_id = 42   (ORDER BY (user_id, ts))
  → binary search trên index thưa → xác định dải granule có thể chứa user 42
  → chỉ đọc + giải nén các granule đó (mỗi granule 8192 row)
```

So sánh bản chất với B+Tree:

| | B+Tree (PG) | Sparse index (CH) |
|---|---|---|
| Trỏ đến | Từng row | Khối 8192 row |
| Kích thước | ~vài % bảng | ~0.001% bảng |
| Tìm 1 row | 1 row chính xác | Đọc cả granule 8192 row rồi lọc |
| Phù hợp | Point lookup OLTP | Range scan lớn OLAP |

Một lần nữa: thiết kế nói thẳng "tôi không phục vụ point lookup".

**`ORDER BY` key là quyết định thiết kế bảng quan trọng nhất** (tương đương shard key của MongoDB): quyết định query nào skip được dữ liệu, và nén sâu đến đâu (dữ liệu sort tốt nén tốt hơn). Quy tắc: cột lọc thường xuyên + cardinality thấp đứng trước (`(tenant_id, event_type, ts)`), cột cardinality cao (ts, user_id) đứng sau.

**Data skipping index** (secondary, tùy chọn): minmax, set, bloom_filter trên cột ngoài ORDER BY — không trỏ đến row, chỉ giúp *bỏ qua granule*. Tác dụng phụ thuộc tương quan dữ liệu với thứ tự vật lý; kỳ vọng đúng: "giảm scan", không phải "index như OLTP".

### 3.4. Partitioning

`PARTITION BY toYYYYMM(date)` — chia part theo tháng/ngày. Vai trò **chủ yếu là quản trị vòng đời**: `DROP PARTITION` xóa 1 tháng dữ liệu tức thì (xóa thư mục); TTL tự động (`TTL date + INTERVAL 90 DAY [DELETE | TO VOLUME 'cold']`) — chuyển dần xuống S3/disk rẻ rồi xóa. Pruning theo partition là phụ — sparse index đã lo phần lọc. **Anti-pattern:** partition quá mịn (theo giờ, theo user) → bùng nổ số part — partition là công cụ vòng đời, không phải công cụ tăng tốc query.

### 3.5. Biến thể MergeTree — "sửa/xóa" theo kiểu OLAP

UPDATE/DELETE tại chỗ đi ngược thiết kế immutable part (mutation `ALTER ... UPDATE/DELETE` = ghi lại toàn bộ part — chỉ dành cho GDPR/sửa lỗi hiếm hoi, không phải nghiệp vụ). Thay vào đó, ClickHouse giải quyết lúc **merge**:

- **ReplacingMergeTree:** giữ bản mới nhất theo key khi merge — dedup/upsert *eventual* (trước merge vẫn thấy trùng → query dùng `FINAL` hoặc `argMax`, có chi phí).
- **SummingMergeTree / AggregatingMergeTree:** các row cùng key được cộng dồn/gộp trạng thái aggregate khi merge — nền tảng của pre-aggregation rẻ.
- **CollapsingMergeTree:** "xóa" bằng row đối ứng sign=-1.

Tư duy chung: *đổi tính tức thời lấy throughput* — mọi thứ đúng dần theo merge nền.

### 3.6. Materialized View — pre-aggregation theo dòng chảy

MV trong ClickHouse là **insert trigger**, không phải view làm mới định kỳ: dữ liệu insert vào bảng nguồn được biến đổi và ghi *ngay* vào bảng đích (thường AggregatingMergeTree):

```
events (thô, TTL 30 ngày)
   │ MV: GROUP BY toStartOfHour(ts), page → count, uniqState(user)
   ▼
events_hourly (giữ 2 năm, nhỏ hơn hàng nghìn lần)
```

Đây là pattern trung tâm của mọi hệ ClickHouse production: **thô để ad-hoc ngắn hạn, MV nhiều tầng cho dashboard dài hạn**. Lưu ý vận hành: MV chỉ xử lý dữ liệu *từ lúc tạo* (backfill tay); lỗi trong MV làm fail INSERT nguồn; `uniqState/uniqMerge` (aggregate function state) là cơ chế cho phép gộp tiếp các aggregate không cộng được như count distinct.

### 3.7. Query Execution — vectorized, parallel, ăn hết máy

- **Vectorized:** operator xử lý block hàng nghìn giá trị/cột mỗi lượt (Chương 1, mục 1.7) — SIMD, ít branch, cache-friendly.
- **Parallel mặc định:** một query chia cho mọi core; cộng thêm parallel đọc nhiều part.
- **Aggregation engine:** hash table tối ưu theo kiểu key, two-level cho cardinality lớn, spill-to-disk khi tràn (`max_bytes_before_external_group_by`).
- **JOIN là điểm cần thận trọng:** mặc định hash join — **kéo toàn bộ vế phải vào RAM**. JOIN hai bảng tỷ row không phải sở trường (khác warehouse tối ưu join như Snowflake/BigQuery). Chiến lược đúng: phi chuẩn hóa từ lúc ingest (flatten trước khi vào CH), dùng dictionary (bảng dimension trong RAM, tra `dictGet` O(1)) — JOIN lớn chỉ khi bất khả kháng.

### 3.8. Distributed: Replication & Sharding

- **Replication theo bảng** (`ReplicatedMergeTree`): các replica đồng bộ **part** qua điều phối của **Keeper** (ZooKeeper protocol, bản CH tự viết) — Keeper giữ metadata + log part, dữ liệu truyền trực tiếp replica↔replica. Multi-write: insert vào replica nào cũng được, lan truyền eventual; `insert_quorum` nếu cần chắc hơn. Kèm dedup block insert (retry an toàn).
- **Sharding qua Distributed table:** bảng "ảo" định tuyến — query fan-out ra mọi shard, mỗi shard aggregate cục bộ (partial aggregate), node nhận gộp kết quả (**scatter-gather 2 pha** — hiệu quả vì partial aggregate nhỏ hơn dữ liệu thô hàng nghìn lần).
- Không có balancer tự động như MongoDB — resharding là việc tay chân. ClickHouse Cloud giải khác hẳn: **tách compute/storage trên S3 (SharedMergeTree)** — mở rộng compute không phải di chuyển dữ liệu; đây là hướng kiến trúc của mọi warehouse hiện đại.

---

## 4. Điểm mạnh

- **Tốc độ aggregation ở scale** — hệ quả cộng hưởng: columnar (I/O ↓ chục lần) × nén (↓ nữa) × skip theo sparse index × vectorized+SIMD × parallel. Full scan tỷ row trong trăm ms–vài giây trên một máy tốt là bình thường. Benchmark tự công bố và cộng đồng (ClickBench) đặt CH ở nhóm đầu OLAP OSS — như mọi benchmark, hãy tự chạy trên workload của bạn trước khi kết luận.
- **Chi phí/hiệu năng:** nén 10x + máy thường + OSS → chi phí thấp hơn warehouse thương mại nhiều lần cho cùng khối lượng (đổi bằng chi phí vận hành — xem mục 7).
- **Ingest khủng khiếp** khi batch đúng: hàng triệu row/giây/node (ghi tuần tự part + nén).
- **SQL quen thuộc** + hàm phân tích chuyên dụng (funnel, retention, quantile xấp xỉ, uniqCombined...).
- **Vòng đời dữ liệu tích hợp:** TTL, tiered storage nóng/lạnh, DROP PARTITION.

## 5. Điểm yếu

- **Không phải OLTP — về cấu trúc:** không transaction đa statement, đọc điểm đắt (granule 8192 row), UPDATE/DELETE là mutation nặng, không FK/unique constraint. Không thể "tiện thể" dùng CH làm database nghiệp vụ.
- **Nhạy cảm insert pattern:** ghi lẻ tẻ = "too many parts" = sự cố (mục 3.2).
- **JOIN lớn hạn chế** (mục 3.7) — thiết kế schema phải denormalize từ đầu.
- **Eventual consistency nội tại:** replica lệch nhau khoảng ngắn; ReplacingMergeTree trùng đến khi merge — ứng dụng phải hiểu và chấp nhận.
- **Concurrency mục tiêu thấp:** thiết kế cho chục–trăm query phân tích đồng thời, không phải chục nghìn query/giây kiểu OLTP; một query ad-hoc tồi có thể chiếm cả máy (cần quota/settings profile).
- **Vận hành cluster tự quản có chiều sâu:** Keeper, schema thay đổi trên cluster, resharding tay, tuning merge.

## 6. Trade-off

| Trade-off | ClickHouse chọn | Nguyên nhân kỹ thuật | Hệ quả |
|---|---|---|---|
| Scan throughput vs Point access | Scan | Columnar + granule + sparse index | Lookup 1 row đọc 8192 row × N cột |
| Ingest throughput vs Ghi lẻ | Batch ingest | Part-per-insert, không memtable | Bắt buộc batching ở kiến trúc ingest |
| Nén sâu vs Update rẻ | Nén (immutable part) | Sửa 1 giá trị = ghi lại part | Update là mutation nặng; sửa-lúc-merge là lối đi |
| Tốc độ vs Consistency | Tốc độ | Replication eventual, không 2PC | Không dùng cho dữ liệu cần đọc-ngay-đúng-tuyệt-đối |
| Tốc độ single query vs Cách ly đa tenant | Single query ăn cả máy | Parallel mặc định toàn core | Cần quota/queue khi mở cho nhiều người dùng ad-hoc |
| Denormalize vs JOIN linh hoạt | Denormalize | Hash join in-memory | Chi phí chuyển sang tầng ingest (ETL/stream) |

## 7. Production Considerations

- **Kiến trúc ingest là 50% thành công:** Kafka → (Kafka engine/consumer batch) → MergeTree; hoặc async_insert; không bao giờ cho ứng dụng insert lẻ trực tiếp.
- **Monitoring tối thiểu:** số part/partition (`system.parts`), merge backlog, lỗi too-many-parts, replication queue (`system.replication_queue`), Keeper health, disk theo tier, query chậm (`system.query_log`), RAM đỉnh theo query.
- **HA:** ≥2 replica/shard, 3 node Keeper; load balancer phía đọc. Backup: `BACKUP ... TO S3` (bản mới) hoặc clickhouse-backup; part immutable → backup incremental tự nhiên.
- **Capacity:** ước lượng theo *dữ liệu sau nén* (đo tỷ lệ nén thật của chính bạn — 5–20x tùy dữ liệu); RAM cho aggregate + dictionary; CPU quyết định latency query.
- **Security & multi-tenant:** user/quota/settings profile (max_memory_usage, max_execution_time per user) — bắt buộc khi mở ad-hoc cho analyst; row policy cho multi-tenant.
- **Upgrade:** rolling theo replica; CH phát hành nhanh — dùng bản LTS cho production.
- **Cloud vs tự quản:** ClickHouse Cloud/Altinity bán vận hành + tách compute-storage; tự quản rẻ hơn ở tải ổn định lớn, đắt hơn ở chi phí người.

## 8. Best Practices

- **ORDER BY key theo query thực** (lọc thường xuyên trước, cardinality thấp trước); PARTITION BY theo đơn vị vòng đời (tháng/ngày), không mịn hơn.
- **Kiểu dữ liệu chặt:** UInt32 thay Int64 khi đủ, LowCardinality(String) cho cột enum-like, DateTime thay String — kiểu đúng = nén tốt = nhanh.
- **Batch insert** ≥ nghìn row; dedup nhờ block-hash khi retry.
- **MV nhiều tầng** cho dashboard; giữ thô ngắn hạn với TTL.
- **Denormalize lúc ingest;** dictionary cho dimension; tránh JOIN tỷ×tỷ.
- **Đặt guard-rail mặc định:** max_memory_usage, max_execution_time, readonly cho BI user — trước khi có sự cố đầu tiên, không phải sau.

## 9. Anti-patterns

1. **Insert từng row từ ứng dụng** → too many parts (lỗi phổ biến số một của người mới).
2. **Dùng CH làm OLTP** — "SQL mà, tiện thể lưu đơn hàng luôn" → phát hiện không có transaction/unique khi đã muộn.
3. **`SELECT * FROM events`** trên bảng 300 cột — vứt bỏ chính lợi thế columnar.
4. **ORDER BY (timestamp)** cho bảng multi-tenant — mọi query theo tenant phải quét toàn thời gian; đúng phải là `(tenant_id, timestamp)`.
5. **Partition theo user_id/giờ** — bùng nổ part.
6. **Lạm dụng `FINAL`** trên ReplacingMergeTree ở mọi query — trả chi phí merge-lúc-đọc thường trực; xem lại nhu cầu dedup từ gốc ingest.
7. **JOIN thay cho denormalize** vì "làm ETL lười" — RAM nổ theo vế phải join.

## 10. Khi nào KHÔNG nên dùng ClickHouse

- **Dữ liệu giao dịch nghiệp vụ** (đơn hàng, số dư, booking) → PostgreSQL. Tuyệt đối — thiếu transaction và constraint không phải thứ bù được bằng code ứng dụng.
- **Point lookup/CRUD theo thực thể** → PostgreSQL/MongoDB/Redis.
- **Dữ liệu nhỏ** (vài chục GB, vài triệu row) → PostgreSQL cân được cả analytics ở cỡ này; thêm CH chỉ thêm hệ để nuôi.
- **Update/delete thường xuyên theo nghiệp vụ** → sai mô hình dữ liệu ngay từ đề bài.
- **Cần join hình sao nặng nề kiểu enterprise BI cổ điển với đội không kiểm soát được ETL** → warehouse tối ưu join (Snowflake/BigQuery) có thể khớp hơn dù đắt.
- **Vì sao đội chọn sai?** Hai chiều: (a) bị con số benchmark quyến rũ, đem CH gánh việc OLTP; (b) ngược lại — cố nuôi analytics tỷ row trên PostgreSQL vì "thêm hệ mới phức tạp", trả giá bằng primary OLTP bị scan đè chết. Cả hai đều né được nếu phân tách rõ workload.

## 11. Decision Framework

**Chọn ClickHouse khi hội đủ:**

- Workload là **append-mostly + aggregate/filter trên nhiều row, ít cột** (event, log, metric, clickstream, telemetry, time-series).
- Khối lượng khiến PostgreSQL đuối về cấu trúc: trăm triệu row trở lên, dashboard cần dưới giây.
- Chấp nhận: batch ingest, eventual consistency, denormalize, không transaction.
- Có (hoặc xây được) đường ingest tử tế: Kafka/buffer/CDC.

**Cấu trúc điển hình đúng bài:** OLTP ở PostgreSQL/MongoDB → CDC/event stream → ClickHouse phục vụ analytics — mỗi hệ đúng sở trường (chi tiết Chương 6).

**Câu hỏi quyết định nhanh:** *"Query chủ đạo trả về bao nhiêu row so với lượng row nó phải xem xét?"* Xem hàng triệu, trả về hàng chục (aggregate) → ClickHouse. Xem một, trả một (lookup) → OLTP engine.
