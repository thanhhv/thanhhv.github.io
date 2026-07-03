+++
title = "Bài 5 — So Sánh PostgreSQL, MongoDB & ClickHouse"
date = "2026-07-02T13:00:00+07:00"
draft = false
tags = ["backend", "database"]
series = ["Database Thực Chiến"]
+++

# So sánh PostgreSQL, MongoDB và ClickHouse

> **Nguyên tắc đọc chương này:** ba hệ không cùng hạng cân trên cùng một trục. So sánh đúng nghĩa là *ánh xạ workload → điểm mạnh cấu trúc*, không phải tìm "database tốt nhất". Mỗi ô dưới đây là hệ quả của kiến trúc đã phân tích ở Chương 2–4; khi nghi ngờ, quay lại chương tương ứng để xem "vì sao".

---

## 5.1. Bảng so sánh tổng hợp

| Tiêu chí | PostgreSQL | MongoDB | ClickHouse |
|---|---|---|---|
| **Data Model** | Quan hệ (bảng, JOIN, constraint) + JSONB làm van linh hoạt | Document (BSON), embed/reference, schema-on-read | Bảng phẳng columnar, denormalize có chủ đích, không quan hệ |
| **Transaction** | ACID đầy đủ, Serializable thật, transactional DDL — chuẩn vàng | ACID trong 1 document (luôn); đa document từ 4.0 — dùng như ngoại lệ, không hàng ngày | Gần như không (atomic theo block insert; không đa statement) — theo thiết kế |
| **Query Capability** | Giàu nhất: SQL đầy đủ, CTE, window, lateral, geo, full-text, vector | Giàu theo trục document: CRUD + aggregation pipeline; JOIN yếu | SQL analytics rất mạnh (aggregate, window, hàm xấp xỉ); JOIN lớn hạn chế; không tối ưu cho OLTP query |
| **Read Performance** | Point lookup/OLTP xuất sắc (B+Tree + buffer pool); analytics lớn kém | Point/one-trục xuất sắc khi có index + đủ RAM; analytics lớn kém | Point lookup kém (cấu trúc); scan/aggregate hàng tỷ row: nhanh hơn 2 hệ kia 10–1000× |
| **Write Performance** | Tốt (chục nghìn TPS đơn giản/máy tốt); trần = 1 máy; update-heavy chịu thuế vacuum | Tốt + scale-out ghi qua sharding; per-document atomic rẻ | Ingest batch cực cao (triệu row/s/node); ghi lẻ tẻ = sự cố; update/delete đắt |
| **Analytics** | Đủ cho báo cáo vừa (parallel query, partition); đuối từ trăm triệu row | Aggregation operational tốt; analytical toàn dataset yếu | Sinh ra để làm việc này |
| **Horizontal Scaling** | Read replica dễ; sharding không tích hợp (Citus/app-level) | Tích hợp, trưởng thành (mongos, balancer, reshard 5.0+) | Tích hợp (Distributed table) nhưng thủ công hơn Mongo; Cloud tách compute/storage |
| **Compression** | Khiêm tốn (TOAST cho giá trị lớn; không nén page mặc định) | Tốt (block compression WiredTiger, prefix index) | Xuất sắc (5–20×, codec chuyên dụng theo cột) |
| **Storage Efficiency** | Trung bình; bloat nếu vacuum không theo kịp | Khá; thuế tên field lặp, bù bằng nén | Tốt nhất cho analytics; part immutable + TTL/tiered |
| **Consistency** | Mạnh mặc định; single-leader rõ ràng | Tunable per-operation (w/readConcern) — mạnh khi cấu hình đúng | Eventual giữa replica; đủ cho analytics, không cho nghiệp vụ |
| **Availability** | Cao nhưng failover cần tầng ngoài (Patroni/managed) | Cao, failover tự động tích hợp (~10–30s) | Cao cho đọc (multi-replica); ghi cần thiết kế ingest chịu lỗi |
| **Operational Complexity** | Thấp–vừa (1 node/RS); vacuum + pooler là hai môn bắt buộc | Vừa (replica set dễ); sharded cluster = bậc phức tạp mới | Vừa–cao tự quản (Keeper, merge, ingest pipeline); thấp nếu Cloud |
| **Cost** | Rẻ nhất ở quy mô nhỏ–vừa; đắt khi ép làm analytics lớn (máy khủng) | Hạ tầng RAM-heavy; Atlas tiện nhưng đắt ở scale | Chi phí/TB analytics thấp nhất (nén + máy thường); tốn đầu tư ingest |
| **Learning Curve** | SQL phổ cập; nội tạng (vacuum, planner) cần thời gian | Bắt đầu dễ nhất; modeling ĐÚNG khó hơn vẻ ngoài nhiều | SQL quen; tư duy OLAP (batch, ORDER BY key, denormalize) phải học lại |
| **Best Use Cases** | Hệ giao dịch, tiền, tồn kho, SaaS core, dữ liệu quan hệ bất kỳ | Catalog/profile/content đa dạng schema, IoT theo thiết bị, scale-out một-trục | Event/log/metric/clickstream, dashboard real-time, time-series lớn |
| **Worst Use Cases** | Analytics tỷ row; write scale-out; queue tần suất cực cao | Quan hệ chặt + bất biến phức tạp; BI toàn dataset | OLTP bất kỳ; dữ liệu nhỏ; update thường xuyên |

---

## 5.2. Ba trục phân tích sâu

### Trục 1 — Mô hình dữ liệu quyết định 80% quyết định

Ba câu hỏi theo thứ tự:

1. **Dữ liệu có bất biến nghiệp vụ cần hệ thống thực thi không?** (số dư không âm, ghế không bán 2 lần, tổng nợ = tổng có) → Có: PostgreSQL, gần như không phải bàn. Constraint + transaction + serializable là thứ hai hệ kia không có ở mức tương đương.
2. **Access pattern có đi theo một trục tự nhiên không?** (mọi query theo user/device/order) và schema đa dạng? → MongoDB khớp: document = đơn vị truy cập, shard key = trục đó.
3. **Câu hỏi đặt lên dữ liệu là "đếm/tổng/phân bố trên rất nhiều bản ghi"?** → ClickHouse, và dữ liệu nên *chảy vào* từ hai hệ trên chứ không sinh ra tại đó.

### Trục 2 — Ngưỡng scale thực tế (số để định cỡ, không phải giới hạn cứng)

| Quy mô | PostgreSQL | MongoDB | ClickHouse |
|---|---|---|---|
| ≤ trăm GB, ≤ vài nghìn TPS | Thoải mái, 1 node + replica | Thoải mái, 1 RS | Thường là thừa |
| TB, chục nghìn TPS ghi | Vẫn được với NVMe + tuning; bắt đầu cân nhắc trần | Vùng thoải mái; cân nhắc shard | Vùng thoải mái (analytics) |
| Chục TB+, ghi vượt 1 máy | Citus/app-shard — phức tạp tăng vọt | Sharded cluster — đúng sân | 1 cluster nhỏ vẫn cân tốt (nén!) |
| Trăm TB–PB analytics | Không thực tế | Không thực tế | Đúng sân (cluster + tiered storage) |

Lưu ý hai chiều: (a) phần cứng hiện đại đẩy trần 1 máy xa hơn trực giác cũ rất nhiều; (b) nén 10× của ClickHouse nghĩa là "100TB dữ liệu" có thể chỉ là 10TB disk — định cỡ theo dữ liệu sau nén.

### Trục 3 — Chi phí vận hành toàn cục (TCO thật)

Chi phí một hệ database = hạ tầng + **người hiểu nó lúc 3 giờ sáng** + chi phí cơ hội của độ phức tạp. Hệ quả thực dụng:

- Số loại database trong công ty nên tăng chậm hơn số microservice rất nhiều. Mỗi hệ mới phải "trả lương" bằng năng lực mà các hệ hiện có kém về *cấu trúc* (không phải kém do chưa tuning).
- PostgreSQL là mặc định tốt vì phủ rộng nhất trên một hệ. MongoDB mua thêm scale-out + schema linh hoạt. ClickHouse mua thêm analytics nhanh 100×. Cả hai khoản "mua thêm" chỉ đáng khi nhu cầu là thật và đo được.

---

## 5.3. Cùng một tính năng, ba cách nhìn — bảng "dịch thuật" cho architect

| Khái niệm | PostgreSQL | MongoDB | ClickHouse |
|---|---|---|---|
| Đơn vị dữ liệu | Row trong heap page | BSON document | Giá trị trong cột, gom theo granule/part |
| "WAL" | WAL | Journal (WiredTiger) + oplog (logic, replication) | Không WAL kiểu OLTP; part atomic + replication log trên Keeper |
| MVCC | Version trong heap + VACUUM | Update chain trong WT cache | Không cần — part immutable, đọc trên snapshot part |
| Index chính | B+Tree trỏ từng row | B+Tree trỏ từng document | Sparse index trỏ granule 8192 row |
| Thay đổi dữ liệu | UPDATE in-MVCC | Update document (atomic) | Insert mới + xử lý lúc merge (Replacing/Summing) |
| Replication | WAL shipping (physical) / logical | Oplog (logical, idempotent) | Đồng bộ part qua Keeper |
| Failover | Tầng ngoài (Patroni) | Tích hợp (bầu cử) | Đọc: LB nhiều replica; ghi: ingest chịu lỗi |
| CDC ra ngoài | Logical decoding (Debezium) | Change streams | Ít cần (thường là đích, không phải nguồn) |
| Nỗi đau đặc trưng | VACUUM/bloat, connection | Shard key sai, unbounded array, RAM | Too many parts, insert lẻ, JOIN to |

Bảng này hữu ích khi review thiết kế: nghe ai đó nói "ta sẽ update record trong ClickHouse mỗi khi user sửa hồ sơ" hay "ta sẽ join 12 collection bằng $lookup" — đối chiếu hàng tương ứng sẽ thấy ngay thiết kế đang chống lại cấu trúc của engine.

---

## 5.4. Kết luận: mô hình phối hợp thay vì lựa chọn duy nhất

Ở quy mô vừa trở lên, câu trả lời trưởng thành hiếm khi là *một* trong ba, mà là **phân công**:

```
                    ┌──────────────┐   CDC / event stream    ┌────────────┐
  Nghiệp vụ chính → │ PostgreSQL   │ ──────────────────────► │ ClickHouse │ → dashboard,
  (tiền, đơn hàng)  │ (source of   │  (Debezium/Kafka)       │ (analytics)│   BI, alerting
                    │  truth)      │                         └────────────┘
                    └──────────────┘                               ▲
                    ┌──────────────┐   change streams              │
  Dữ liệu document →│ MongoDB      │ ──────────────────────────────┘
  (catalog, profile)│ (nếu cần)    │
                    └──────────────┘
```

Nguyên tắc phân công: **source of truth thuộc về engine có bảo đảm mạnh nhất mà workload ghi chịu được; analytics thuộc về engine đọc nhanh nhất; dữ liệu chảy một chiều qua CDC/event.** Chương 6 áp dụng khung này vào 12 kiến trúc thực tế.
