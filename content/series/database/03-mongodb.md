+++
title = "Bài 3 — MongoDB"
date = "2026-07-02T11:00:00+07:00"
draft = false
tags = ["backend", "database"]
series = ["Database Thực Chiến"]
+++

# MongoDB

> **Tiền đề:** Chương 1 (LSM/B+Tree, replication, sharding, CAP/PACELC).
> **Góc nhìn:** từ người thiết kế engine — WiredTiger, oplog, Replica Set, sharding — không phải từ tutorial CRUD.

---

## 1. Problem Statement

**MongoDB giải bài toán gì?** Hai bài toán mà RDBMS đầu những năm 2000 giải kém:

1. **Impedance mismatch:** đối tượng trong ứng dụng (nested, mảng, đa hình) phải "băm" ra hàng chục bảng chuẩn hóa rồi JOIN lại. Mỗi thay đổi cấu trúc là một cuộc migration. Tốc độ phát triển sản phẩm bị schema ghì lại.
2. **Scale-out:** RDBMS thời đó scale ghi bằng cách mua máy to hơn; sharding phải tự chế ở tầng ứng dụng (như Facebook/YouTube từng làm với MySQL — tốn cả đội ngũ chỉ để vận hành sharding).

**Nếu không có nó thì sao?** Hoặc chịu ma sát schema + tự shard bằng tay, hoặc dùng key-value store thuần (mất khả năng query phong phú). MongoDB đặt cược vào điểm giữa: **document model giàu query + sharding tích hợp**.

**Vì sao khó?** Vì "schema linh hoạt" và "phân tán tự động" đều chuyển độ phức tạp từ database sang chỗ khác — câu hỏi là chuyển đi đâu và ai trả giá. Toàn bộ chương này xoay quanh câu trả lời.

---

## 2. Tại sao MongoDB tồn tại

**Bối cảnh 2007–2009:** web scale bùng nổ, phong trào NoSQL (BigTable/Dynamo papers), agile/startup cần ship nhanh. 10gen xây MongoDB với ưu tiên rõ rệt: **developer velocity trước, scale-out tích hợp, tính năng đúng đắn hoàn thiện dần sau**.

Lịch sử kỹ thuật của MongoDB là lịch sử **trả nợ dần các cam kết đúng đắn**:

- Thời kỳ đầu: MMAPv1 storage engine, per-database lock, write concern mặc định không chờ xác nhận → mang tiếng "mất dữ liệu". Các phân tích Jepsen thời kỳ này chỉ ra nhiều lỗ hổng thật.
- 2014: mua WiredTiger → storage engine hiện đại, document-level concurrency, nén.
- 3.2–4.0: raft-hóa bầu cử, majority read/write concern, multi-document transaction (4.0 trong replica set, 4.2 xuyên shard).
- Hiện nay: mặc định đã an toàn hơn nhiều (`w: majority` mặc định từ 5.0); các phiên bản gần đây qua được kiểm tra Jepsen với cấu hình đúng.

**Bài học meta cho architect:** đánh giá database phải theo *phiên bản và cấu hình cụ thể*, không theo danh tiếng cũ — cả chiều khen lẫn chiều chê.

---

## 3. Cách hoạt động bên trong

### 3.1. Document Model và BSON

Dữ liệu là **document** (tương tự JSON) trong **collection**, không có schema bắt buộc (từ 3.6 có schema validation tùy chọn qua JSON Schema).

**BSON** — Binary JSON — là quyết định thiết kế nền tảng:

- Nhị phân, có độ dài trường prefix → **traverse không cần parse toàn bộ** (nhảy qua field theo length), khác JSON text.
- Kiểu dữ liệu giàu hơn JSON: int32/int64, decimal128 (tiền!), date, ObjectId, binary.
- **ObjectId** 12 byte = timestamp(4) + random(5) + counter(3) → sinh phân tán không cần điều phối, sắp xếp thô theo thời gian. Đây là câu trả lời cho "làm sao có ID duy nhất trên 100 shard mà không có điểm điều phối trung tâm".

**Nguyên tắc modeling nền tảng — "what you query together, store together":**

```
RDBMS:  orders ──< order_items ──< ...   (JOIN lúc đọc)
MongoDB: {
  _id: ...,
  customer: {...},
  items: [ {sku, qty, price}, ... ]      ← embed: 1 lần đọc = cả đơn hàng
}
```

Embed đổi **chi phí JOIN lúc đọc** lấy **chi phí duplication + kích thước document**. Giới hạn cứng: document ≤ 16MB. Anti-pattern kinh điển: **unbounded array** (embed danh sách lớn dần vô hạn — comment của bài viral) → document phình, mỗi update ghi lại lớn dần, chạm trần 16MB = sự cố. Quy tắc: embed quan hệ "một-vài", reference quan hệ "một-vô-hạn".

### 3.2. WiredTiger — storage engine

- **Cấu trúc:** B+Tree (biến thể) cho collection và index; mỗi collection/index một file.
- **MVCC:** phiên bản giữ trong cache dưới dạng update chain; snapshot isolation cho đọc. **Concurrency ở mức document** — hai client sửa hai document khác nhau không chặn nhau (bước nhảy vọt so với per-database lock của MMAPv1).
- **Cache riêng** (mặc định ~50% RAM): trong cache, page ở dạng giải nén; xuống disk mới nén.
- **Nén mặc định:** snappy cho collection (zstd tùy chọn), **prefix compression cho index**. Đáng chú ý: dữ liệu document lặp tên field trong mọi document ("field name overhead") — nén block bù lại phần lớn chi phí này trên disk, nhưng **trong cache thì không** → RAM là tài nguyên quý nhất của MongoDB.
- **Durability:** journal (WAL của WiredTiger) + checkpoint (mặc định 60s). Commit chờ journal theo `j: true` hoặc bao hàm trong `w: majority`.

**Checkpoint + journal thay vì chỉ WAL:** WiredTiger phục hồi = checkpoint gần nhất + replay journal từ đó. Nguyên lý y hệt Chương 1 — chỉ khác tên gọi.

### 3.3. Indexing

B+Tree tương tự RDBMS, với các dạng đặc thù document model: compound (nhớ quy tắc **ESR — Equality, Sort, Range** khi xếp thứ tự field), multikey (index vào **từng phần tử mảng** — một document sinh nhiều entry; hạn chế: một compound index chỉ được một field mảng), TTL (tự xóa document hết hạn — cho session/cache), partial, text, geospatial (2dsphere), wildcard.

Khác biệt then chốt so với RDBMS: **không có index = COLLSCAN im lặng.** Không có DBA gác cổng, schema linh hoạt khiến dev quên index theo query mới. `explain()` với `totalDocsExamined >> nReturned` là chỉ số vàng cần giám sát.

### 3.4. Aggregation Pipeline

Mô hình xử lý dữ liệu dạng **pipeline các stage** — tư duy dòng chảy thay vì tư duy tập hợp của SQL:

```
db.orders.aggregate([
  { $match:  { status: "paid", ts: { $gte: ISODate("2026-01-01") } } },  // lọc sớm → dùng index
  { $group:  { _id: "$customerId", total: { $sum: "$amount" } } },
  { $sort:   { total: -1 } },
  { $limit:  10 }
])
```

Điểm kỹ thuật cần hiểu:

- **Thứ tự stage quyết định hiệu năng:** `$match`/`$sort` đứng đầu pipeline mới dùng được index; optimizer có tự đẩy `$match` lên nhưng không phải mọi trường hợp.
- `$group`/`$sort` không dùng được index phải giữ dữ liệu trong RAM (giới hạn 100MB/stage, tràn thì `allowDiskUse` — chậm).
- `$lookup` (left join) tồn tại nhưng là **tín hiệu cảnh báo modeling**: dùng thường xuyên trên đường nóng nghĩa là dữ liệu "query together" đã không được "store together" — và `$lookup` không hoạt động tự do xuyên shard (collection phải unsharded hoặc từ 5.1+ có hỗ trợ giới hạn với chi phí lớn).
- Aggregation trên volume lớn vẫn là **row-oriented, single-node-per-shard execution** — không phải đối thủ của columnar engine. MongoDB phù hợp aggregation *operational* (theo user, theo đơn hàng), không phải *analytical* (toàn bộ dataset).

### 3.5. Replica Set

Đơn vị HA của MongoDB — single-leader replication với **bầu cử tích hợp** (giao thức tinh thần Raft):

```
        ┌── Secondary (replicate oplog, có thể bầu làm primary)
Primary ┤
        └── Secondary
(ghi duy nhất; heartbeat 2s; mất liên lạc 10s → bầu cử; cần đa số phiếu)
```

- **Oplog:** capped collection ghi lại mọi thay đổi ở dạng **idempotent logic operation** (không phải bytes vật lý như WAL shipping của PG) → secondary áp lại được, dùng lại được cho **change streams** (CDC tích hợp — điểm mạnh thực dụng lớn).
- **Cần đa số để có primary** → cluster 2 node là vô nghĩa (mất 1 node = mất khả năng ghi); tối thiểu 3 (hoặc 2 + arbiter, nhưng arbiter làm yếu `w: majority` — tránh trong production nghiêm túc).
- **Tự động failover ~10–30s** không cần công cụ ngoài — khác biệt vận hành lớn nhất so với PostgreSQL.

**Tunable consistency — PACELC per-operation:**

| Cấu hình | Ý nghĩa | Trade-off |
|---|---|---|
| `w: 1` | Primary ack là xong | Nhanh; primary chết trước khi replicate → **rollback dữ liệu đã ack** |
| `w: majority` | Đa số ack | Sống sót failover; +latency một round-trip |
| `readConcern: local` | Đọc mới nhất tại node | Có thể đọc dữ liệu sẽ bị rollback |
| `readConcern: majority` | Chỉ đọc dữ liệu đã majority-commit | An toàn; có thể cũ hơn một chút |
| `readPreference: secondary` | Scale read | Replication lag → stale read |

Đây là ưu điểm (chọn theo từng nghiệp vụ: log dùng `w:1`, thanh toán dùng `w:majority` + `readConcern:majority`) và đồng thời là bẫy (mặc định lịch sử từng là `w:1` — nguồn gốc danh tiếng "mất dữ liệu").

### 3.6. Sharding

Kiến trúc ba thành phần:

```
Client → mongos (router, stateless, nhiều instance)
             │  (routing table cache)
   ┌─────────┼──────────┐
   ▼         ▼          ▼
 Shard 1   Shard 2    Shard 3      ← mỗi shard là một Replica Set đầy đủ
(RS 3 node)(RS 3 node)(RS 3 node)
             ▲
      Config Server RS (metadata: chunk → shard mapping)
```

- Dữ liệu chia theo **shard key** thành **chunk**; balancer di chuyển chunk giữa các shard.
- **Chọn shard key là quyết định một chiều quan trọng nhất** (reshard được từ 5.0 nhưng là chiến dịch lớn, đắt đỏ):
  - Cardinality cao, phân bố ghi đều, và **xuất hiện trong đa số query** (query có shard key → định tuyến 1 shard; không có → scatter-gather toàn bộ shard).
  - Anti-pattern chết người: shard key đơn điệu tăng (timestamp, ObjectId) → **mọi insert dồn vào chunk cuối của một shard** — cluster 10 shard, 1 shard gánh 100% ghi. Giải pháp: hashed shard key (mất range query) hoặc compound key có tiền tố phân tán.
- **Chi phí ẩn của sharding:** cluster tối thiểu nghiêm túc = 2 shard × 3 node + 3 config + 2 mongos ≈ **11 process** phải vận hành, giám sát, backup. So với 1 replica set 3 node — độ phức tạp nhảy một bậc. Nguyên tắc Chương 1 lặp lại: *shard khi có bằng chứng cần, không phải vì "để sẵn cho tương lai"*.

### 3.7. Multi-document Transactions

Có từ 4.0 (replica set) / 4.2 (sharded, dùng 2PC nội bộ). Snapshot isolation, giới hạn thời gian mặc định 60s. Vấn đề không phải "có hay không" mà là **chi phí**: giữ snapshot + cache pressure trên WiredTiger, xuyên shard thì 2PC latency. **Nguyên tắc thiết kế đúng:** transaction đa document trong MongoDB là **van an toàn cho ngoại lệ**, không phải công cụ hàng ngày. Nếu thiết kế cần multi-document transaction trên đường nóng, đó là tín hiệu hoặc modeling sai (đáng lẽ embed để atomic-trong-1-document — mọi thao tác trên MỘT document luôn atomic), hoặc chọn nhầm database (bài toán vốn quan hệ → PostgreSQL).

---

## 4. Điểm mạnh

- **Velocity:** model khớp object trong code, thêm field không cần migration — đúng chỗ nhất cho sản phẩm giai đoạn khám phá, schema biến động.
- **Scale-out tích hợp, HA tích hợp:** failover tự động không cần Patroni; sharding không cần tự chế — chi phí kỹ sư hạ tầng thấp hơn đáng kể ở quy mô lớn.
- **Atomic single-document + embed** phủ được tỷ lệ lớn nhu cầu transactional thực tế nếu modeling đúng.
- **Change streams** — CDC hạng nhất, xây event-driven dễ.
- **Tunable consistency per-operation** — một cluster phục vụ nhiều mức yêu cầu.

## 5. Điểm yếu

- **Đẩy trách nhiệm toàn vẹn về ứng dụng:** không FK, không CHECK constraint xuyên document, schema validation tùy chọn ít ai bật. "Schema linh hoạt" sau 3 năm và 5 thế hệ dev = **schema hỗn loạn ngầm** (field cùng tên khác kiểu, ngày lưu string lẫn Date). Chi phí này vô hình lúc đầu, lớn dần theo tuổi codebase.
- **JOIN yếu về cấu trúc** — `$lookup` hạn chế, đặc biệt xuyên shard. Bài toán vốn nhiều quan hệ nhiều chiều sẽ khổ sở.
- **RAM-hungry:** hiệu năng tụt vách đá khi working set tràn WiredTiger cache; field name lặp làm cache "loãng" hơn RDBMS tương đương.
- **Analytics lớn không phải sân của nó** (row-oriented — mục 3.4).
- **Chi phí vận hành sharded cluster** cao hơn vẻ ngoài (11+ process, chọn shard key một chiều).

## 6. Trade-off

| Trade-off | MongoDB chọn | Nguyên nhân kỹ thuật | Hệ quả |
|---|---|---|---|
| Flexibility vs Integrity | Flexibility | Schema-on-read, validation opt-in | Toàn vẹn là kỷ luật của đội, không phải bảo đảm của hệ thống |
| Read locality vs Duplication | Embed (locality) | Document = đơn vị I/O và atomicity | Đọc 1 lần lấy tất; sửa dữ liệu lặp ở N chỗ là N lần ghi |
| Consistency vs Latency | Tùy chỉnh per-op | writeConcern/readConcern | Mạnh khi dùng đúng; bẫy khi dùng mặc định mù |
| Scale-out vs Query tự do | Scale-out | Shard key routing | Query không có shard key = scatter-gather; thiết kế query bị shard key trói |
| Tích hợp HA vs kiểm soát tinh vi | Tích hợp | Bầu cử Raft-like nội bộ | Vận hành dễ hơn PG; ít điểm can thiệp hơn Patroni |

## 7. Production Considerations

- **Topology tối thiểu:** replica set 3 node, 3 AZ. `w: majority` cho dữ liệu quan trọng. Không dùng arbiter nếu tránh được.
- **Backup:** mongodump chỉ đủ cho dữ liệu nhỏ; production dùng snapshot filesystem nhất quán hoặc Ops Manager/Atlas continuous backup (PITR qua oplog).
- **Monitoring tối thiểu:** replication lag, oplog window (giờ oplog còn giữ — secondary chết lâu hơn window = phải initial sync lại toàn bộ), WiredTiger cache dirty/used %, connections, COLLSCAN (qua profiler/`totalDocsExamined`), disk.
- **Capacity:** RAM ≥ working set + index nóng; theo dõi cache eviction rate như tín hiệu sớm nhất của "sắp tụt vách đá".
- **Security:** authentication BẮT BUỘC bật (lịch sử Internet đầy cluster Mongo mở toang bị ransom — mặc định cũ bind 0.0.0.0 không auth), TLS, network isolation.
- **Upgrade:** rolling upgrade trong replica set — thay từng secondary, stepdown primary; đọc kỹ compatibility version (FCV).
- **Multi-region:** replica set trải region với priority điều khiển primary ở đâu; sharded + zone sharding cho data residency.
- **Atlas vs tự vận hành:** Atlas mua vận hành bằng tiền — thường rẻ hơn đội ngũ tự vận hành sharded cluster nghiêm túc; trade-off là chi phí ở scale lớn và lock-in.

## 8. Best Practices

- **Model theo access pattern, không theo "đối tượng nghiệp vụ":** liệt kê query trước, thiết kế document sau. Embed một-vài; reference một-nhiều-vô-hạn; các pattern chuẩn: bucket (time-series), subset (10 comment mới nhất embed, còn lại collection riêng), computed (tính trước lúc ghi), extended reference (nhúng bản sao rút gọn của đối tượng liên quan).
- **Bật schema validation** ở mức tối thiểu (kiểu của field cốt lõi) — chi phí gần 0, cứu chuộc tương lai.
- Index theo ESR; xóa index không dùng (`$indexStats`); dùng partial thay vì index cả collection thưa.
- Ghi có kỷ luật: `w: majority` mặc định ứng dụng; retryable writes bật (mặc định driver mới); idempotency ở tầng nghiệp vụ.
- Shard key: chọn như thể không thể đổi (dù 5.0+ reshard được); mô phỏng phân bố ghi trước khi chốt.

## 9. Anti-patterns

1. **Dùng MongoDB như RDBMS:** collection chuẩn hóa như bảng + `$lookup` khắp nơi → mọi nhược điểm của cả hai thế giới, ưu điểm của không thế giới nào.
2. **Unbounded array** (mục 3.1) — chạm trần 16MB giữa đêm.
3. **Shard key đơn điệu tăng** → một shard gánh cả cluster (mục 3.6).
4. **`w:1` cho dữ liệu tiền bạc** → rollback sau failover = "mất tiền đã báo thành công".
5. **Query không index trên collection lớn** — COLLSCAN âm thầm ăn hết I/O.
6. **Transaction đa document làm nếp hằng ngày** — tín hiệu modeling/chọn hệ sai (mục 3.7).
7. **Không giám sát oplog window** → secondary bảo trì lâu quay lại phải initial sync, kéo I/O primary đúng giờ cao điểm.

## 10. Khi nào KHÔNG nên dùng MongoDB

- **Dữ liệu quan hệ chặt, bất biến nghiệp vụ phức tạp, đối soát tài chính** → PostgreSQL. Cưỡng bức bài toán quan hệ vào document là nguồn khổ đau dài hạn phổ biến nhất.
- **Analytics quét lớn / BI** → ClickHouse hay warehouse columnar.
- **Ứng dụng CRUD nhỏ, schema ổn định, một máy đủ** → PostgreSQL đơn giản và "đủ" hơn; JSONB phủ được phần linh hoạt.
- **Cần multi-region active-active ghi mọi nơi với xung đột tự giải quyết** → hệ CRDT/Dynamo-style; MongoDB vẫn là single-primary per shard.
- **Vì sao nhiều đội chọn sai?** Vì demo 15 phút của MongoDB cực kỳ thuyết phục — insert JSON, không schema, chạy ngay. Chi phí thật (kỷ luật modeling, toàn vẹn ở app, RAM, shard key) chỉ lộ sau 12–24 tháng. Ngược lại cũng có đội né MongoDB vì danh tiếng 2012 dù ngày nay nó đã khác — cả hai đều là quyết định bằng meme thay vì phân tích.

## 11. Decision Framework

**Chọn MongoDB khi:**

- Dữ liệu tự nhiên là document (catalog sản phẩm đa dạng thuộc tính, hồ sơ người dùng, nội dung CMS, event/IoT theo thiết bị) và **access pattern chủ yếu theo một trục** (theo user, theo device, theo đơn).
- Schema biến động nhanh theo sản phẩm; đội chấp nhận kỷ luật validation.
- Lộ trình scale-out ghi là thật (đo đạc/dự báo được), muốn sharding+HA tích hợp thay vì tự xây trên RDBMS.
- Cần change streams / offline-sync (hệ sinh thái Realm/Atlas) làm xương sống event-driven.

**Chọn hệ khác khi:** bất biến quan hệ là cốt lõi (→ PostgreSQL), aggregation toàn dataset là cốt lõi (→ ClickHouse), key-value thuần tốc độ cao (→ Redis/DynamoDB).

**Câu hỏi quyết định nhanh:** *"Vẽ 5 query quan trọng nhất — chúng đi theo một trục (user/device/order) hay đan chéo nhiều thực thể?"* Một trục → MongoDB tỏa sáng. Đan chéo → RDBMS.
