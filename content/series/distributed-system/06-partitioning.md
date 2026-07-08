+++
title = "Chương 06 – Partitioning / Sharding: Chia dữ liệu để scale write"
date = "2026-07-08T22:00:00+07:00"
draft = false
tags = ["backend", "distributed-systems"]
series = ["Distributed Systems"]
+++

## 1. Problem Statement

Replication (chương 05) nhân bản *toàn bộ* dữ liệu — giải quyết availability và scale read, nhưng **mỗi node vẫn phải chứa hết dữ liệu và nhận hết write**. Khi dataset vượt dung lượng một máy (chục TB) hoặc write throughput vượt khả năng một leader, con đường duy nhất: **chia dữ liệu thành các partition (shard), mỗi node giữ một phần.**

Nếu không partition: disk đầy (hard stop), write throughput chạm trần leader, index không vừa RAM → mọi query chậm dần, backup/restore lâu đến vô dụng. Giải pháp cũ — mua máy to hơn — chạm trần và đắt phi tuyến (chương 01).

Partition + Replication luôn đi cùng nhau trong hệ thật: dữ liệu chia thành P partition, mỗi partition có N replica (Kafka topic 100 partition × RF 3; CockroachDB chia range 512MB × 3 replica).

## 2. Hai chiến lược chia — và cái giá của mỗi bên

### 2.1. Range Partitioning

Chia theo khoảng giá trị key: `[a–f] → shard 1, [g–m] → shard 2...` (HBase, CockroachDB, Spanner, MongoDB range sharding).

- ✅ **Range query hiệu quả**: `WHERE time BETWEEN ...` chạm 1–2 shard.
- ✅ Chia lại linh hoạt: range to thì split, nhỏ thì merge.
- ❌ **Hot spot theo pattern ghi**: key tăng đơn điệu (timestamp, auto-increment ID) → mọi write dồn vào range cuối — **một shard nhận 100% write, các shard khác ngồi chơi**. Đây là lỗi thiết kế schema phổ biến nhất với range-partitioned store.

### 2.2. Hash Partitioning

`shard = hash(key) mod N` (hoặc theo hash range). Cassandra, DynamoDB, Redis Cluster.

- ✅ Phân bố đều gần như mọi pattern key.
- ❌ **Mất range query**: key liền kề nằm rải rác → `BETWEEN` phải scatter-gather toàn bộ shard.
- ❌ `mod N` ngây thơ: đổi N là gần như **toàn bộ** key đổi chỗ → rebalancing thảm họa. Dẫn đến §3.

Thiết kế lai phổ biến (DynamoDB, Cassandra): **compound key = partition key (hash) + sort key (range trong partition)** — phân bố đều giữa các partition, vẫn range query được *trong* một partition. Ví dụ: `(user_id, order_time)` — order của một user nằm cùng chỗ và có thứ tự.

### 2.3. So sánh

| | Range | Hash | Hash + Sort key |
|---|---|---|---|
| Range query xuyên key | ✅ | ❌ | Trong partition ✅ |
| Chống hot spot key đơn điệu | ❌ | ✅ | ✅ (nếu chọn partition key tốt) |
| Rebalancing | Split/merge tự nhiên | Cần consistent hashing / pre-split | Như hash |
| Đại diện | HBase, Spanner, CockroachDB | Cassandra, Redis Cluster | DynamoDB, Cassandra (chuẩn mực) |

## 3. Consistent Hashing — cơ chế bên trong

### 3.1. Bài toán

`hash(key) mod N`: thêm 1 node (N=4→5) làm ~80% key đổi node. Với cache cluster nghĩa là ~80% cache miss đồng loạt → DB nhận bão traffic (cache avalanche tự gây). Với storage nghĩa là di chuyển gần như toàn bộ dữ liệu.

### 3.2. Giải pháp: vòng hash

```
                    0/2³²
              node A ●─────● vnode B₁
             ╱                    ╲
    key "user42" → hash → điểm trên vòng
            │  đi THUẬN CHIỀU KIM ĐỒNG HỒ
            ▼  gặp node đầu tiên → node đó giữ key
    vnode C₂ ●                    ● vnode A₂
              ╲                  ╱
               ● vnode B₂ ────● node C
```

Cả node lẫn key được hash lên một vòng tròn; key thuộc về node đầu tiên theo chiều kim đồng hồ. **Thêm/bớt một node chỉ ảnh hưởng các key nằm giữa nó và node liền trước** — trung bình 1/N tổng số key, thay vì ~100%.

**Virtual nodes (vnodes)**: mỗi node vật lý = 100–256 điểm trên vòng. Giải quyết ba việc: phân bố đều hơn (luật số lớn), node mạnh gán nhiều vnode hơn (heterogeneous hardware), khi một node chết, tải của nó chia đều cho *tất cả* node còn lại thay vì dồn cả vào node kế tiếp.

Ứng dụng: Cassandra/ScyllaDB (token ring), DynamoDB, Riak, memcached client (Ketama), Envoy ring-hash LB. Ghi chú: hệ dùng **directory/metadata service** (đọc §4) không cần consistent hashing — họ tra bảng; consistent hashing tỏa sáng khi *không muốn* duy trì bảng tra tập trung.

### 3.3. Định tuyến request — ba mô hình

1. **Client biết topology** (smart client — Cassandra driver, Redis Cluster client): ít hop nhất; client phức tạp, phải theo dõi topology change.
2. **Routing tier** (Redis Cluster MOVED/proxy, mongos của MongoDB): client đơn giản; thêm hop + thành phần phải vận hành.
3. **Node nào cũng nhận rồi forward** (Dynamo coordinator): đơn giản triển khai; hop thừa.

Kèm theo là **metadata problem**: ai giữ bản đồ partition→node? (ZooKeeper/etcd, gossip protocol của Cassandra, config server của MongoDB). Bản đồ này chính là chỗ cần consensus (chương 07) — scale data thì partition, nhưng *metadata về partition* thì cần consistency mạnh.

## 4. Rebalancing — di chuyển dữ liệu mà không ai nhận ra

Khi nào cần: thêm/bớt node, shard quá to (split), tải lệch. Nguyên tắc thiết kế:

- **Di chuyển tối thiểu**: chỉ phần dữ liệu cần thiết (consistent hashing, hoặc fixed partitions — tạo sẵn 1024 partition từ đầu, rebalance = gán lại partition cho node, không chia lại key).
- **Throttle**: rebalancing ăn disk IO + network của traffic nghiệp vụ. Không throttle → rebalancing *gây ra* sự cố nó định phòng ngừa. Kafka có replication quota; Cassandra có stream throughput limit.
- **Không tự động quá nhạy**: node chậm 2 phút (GC, deploy) mà auto-rebalance kích hoạt → di chuyển TB dữ liệu vô ích, rồi lại chuyển về. Rebalance tự động cần hysteresis + xác nhận con người ở quy mô lớn.
- **Dual-read/dual-write trong chuyển tiếp**: giai đoạn migration, đọc cả chỗ cũ lẫn mới để không có cửa sổ mất dữ liệu.

## 5. Hot Partition / Hot Key — kẻ giết scale thầm lặng

### 5.1. Bản chất

Partitioning giả định tải chia đều theo key. Thực tế tuân theo power law: 1% key nhận 90% traffic. Celebrity trên Twitter, flash-sale SKU, tenant khổng lồ trong hệ multi-tenant — **một partition nóng chạm trần một node, và toàn hệ thống "đã scale" vẫn nghẽn như một máy** đối với key đó.

Phát hiện: metric per-partition (throughput, latency, queue depth) — trung bình toàn cluster đẹp long lanh trong khi partition 17 đang cháy. DynamoDB có CloudWatch ConsumedCapacity per-partition-key (top keys); Kafka có per-partition lag.

### 5.2. Kỹ thuật xử lý

| Kỹ thuật | Cơ chế | Giá phải trả |
|---|---|---|
| **Key salting / splitting** | `hot_key#1..#10` chia write ra 10 partition | Read phải scatter-gather 10 chỗ + merge |
| **Cache tầng trên** | Hot key thường là read-hot → cache local/CDN hấp thụ | Consistency + cache stampede (ch. 10) |
| **Tách riêng tenant/key nóng** | Celebrity đi hệ riêng (Twitter làm thật) | Hai code path, phức tạp vĩnh viễn |
| **Adaptive capacity** (DynamoDB) | Hạ tầng tự dồn capacity + isolate hot key sang partition riêng | Có trần; không cứu được key nóng *cực đoan* |
| **Batch/aggregate trước khi ghi** | Đếm view: gộp 1000 increment thành 1 | Mất real-time granularity |

## 6. Secondary Index & Query xuyên shard

Partition theo `user_id`, nhưng cần query theo `email`?

- **Local index** (document-partitioned): mỗi shard tự index dữ liệu của mình → write rẻ (một chỗ), read theo index phải **scatter-gather mọi shard** → p99 = shard chậm nhất, tail latency khuếch đại theo số shard.
- **Global index** (term-partitioned): index tự nó được partition theo giá trị index (email a–m ở node X...) → read một chỗ, write phải cập nhật index ở node khác → **write xuyên node, thường async** → index trễ (DynamoDB GSI là eventually consistent — đó là lý do, không phải lười).

Không có lựa chọn thứ ba không mất gì. Quy tắc thiết kế NoSQL: **mô hình hóa dữ liệu theo câu hỏi sẽ hỏi** (query-first design), chấp nhận denormalize — khác triết lý normalize của SQL một máy.

Cross-shard join/transaction: tránh bằng cách **co-locate dữ liệu hay join với nhau vào cùng shard** (cùng partition key — ví dụ order và order_items cùng key `order_id`; interleaved tables của Spanner). Không co-locate được → chương 08 (distributed transactions) với đầy đủ chi phí của nó.

## 7. Trade-off tổng hợp

- **Scale write vs độ phức tạp query**: partition mua write throughput bằng cách *phá vỡ tính toàn cục* — mọi thao tác xuyên shard (join, transaction, unique constraint, range scan) từ tầm thường thành đắt/khó.
- **Phân bố đều vs locality**: hash đều nhưng mất locality; range giữ locality nhưng rủi ro hot spot. Compound key là hòa giải, đổi bằng việc phải *nghĩ kỹ* khi thiết kế schema.
- **Số partition**: ít → trần scale thấp, partition to khó di chuyển; nhiều → overhead metadata/file handle/replication, recovery lâu. Kinh nghiệm: partition size mục tiêu 1–10GB (Cassandra <100MB/partition-key, Kafka partition <25GB, CockroachDB range 512MB tự split).
- **Auto-sharding vs manual**: auto (DynamoDB, CockroachDB, Spanner) giải phóng vận hành nhưng khi nó quyết định sai (split storm, hot range không tách) bạn debug hộp đen; manual (Vitess, Citus, application-level) kiểm soát hoàn toàn, trả bằng công sức kỹ sư liên tục.

## 8. Production Considerations & Best Practices

- **Chọn partition key là quyết định một-chiều quan trọng nhất**: đổi partition key = re-shard toàn bộ = dự án nhiều tháng. Tiêu chí: cardinality cao, phân bố tải đều, và là tiền tố của các query quan trọng nhất.
- **Giám sát phân bố**: top-N partition theo size và theo throughput; hệ số lệch (max/mean) — alert khi > 3–5x.
- **Load test với phân bố key THẬT**: test với key uniform là tự lừa mình — production luôn power-law.
- **Kế hoạch re-shard trước khi cần**: pre-split (tạo dư partition từ đầu — 4x số node dự kiến 3 năm), hoặc chọn hệ tự split. Đừng đợi disk 85%.
- **Capacity theo partition, không theo tổng**: cluster 100 node còn 40% capacity tổng vẫn chết nếu partition nóng nhất hết chỗ.

## 9. Anti-patterns & Khi nào KHÔNG shard

**Anti-patterns:**
- Partition key = timestamp/auto-increment trên range store → toàn bộ write vào một shard.
- Partition key cardinality thấp (`country` — 90% là VN) → hot partition bẩm sinh.
- Shard theo tenant khi có tenant chiếm 50% hệ thống.
- `mod N` không consistent hashing rồi "sẽ tính sau khi thêm node".
- Shard sớm: 50GB dữ liệu mà đã sharding — nhận toàn bộ độ phức tạp (mất join, mất transaction) để giải bài toán chưa có. **Một node PostgreSQL + NVMe chứa hàng TB và chục nghìn TPS thoải mái.**
- Cross-shard query trong hot path rồi ngạc nhiên vì p99.

**Không shard khi**: dataset < 1TB và write < ~10–50K TPS (tùy workload) — scale up + read replica + caching còn rất nhiều dư địa; hoặc khi 80% query là cross-entity analytics (→ data warehouse riêng, đừng bắt OLTP shard gánh).

## 10. Troubleshooting

| Triệu chứng | Chẩn đoán | Xử lý |
|---|---|---|
| p99 cao, p50 đẹp, CPU cluster thấp | Hot partition | Per-partition metrics → tìm key → salting/cache/tách riêng |
| Một node disk đầy trong khi cluster còn 60% | Phân bố key lệch hoặc partition to bất thường | Top partition theo size; split; xem lại key |
| Sau khi thêm node, cache miss tăng vọt | Hash mod N, không consistent hashing | Chuyển consistent hashing; warm cache trước khi cutover |
| Rebalancing làm sập cluster | Không throttle, chạy giờ cao điểm | Quota/throttle, chạy off-peak, tạm dừng được |
| Query theo trường phụ chậm dần theo số shard | Scatter-gather local index | Global index (chấp nhận trễ) hoặc denormalize theo query |
| Kafka: một consumer trong group luôn tụt lag | Partition hot (key skew) hoặc consumer đó yếu | Lag per-partition; sửa key hoặc sticky assignor |

---

## Tổng kết chương

### Những điều bắt buộc phải nhớ
1. Replication scale read và availability; **chỉ partitioning scale write và dung lượng**.
2. Chọn partition key là quyết định khó đảo ngược nhất trong thiết kế hệ dữ liệu — cardinality cao, tải đều, khớp query pattern.
3. Consistent hashing tồn tại để **thêm/bớt node chỉ di chuyển 1/N dữ liệu**; vnodes làm phân bố mượt.
4. Hot partition là failure mode số một của hệ đã shard — trung bình cluster đẹp không có nghĩa gì, nhìn per-partition.
5. Secondary index xuyên shard: local index = read đắt, global index = write đắt/trễ. Không có bữa trưa miễn phí.
6. Co-locate dữ liệu giao dịch cùng nhau vào cùng shard để né distributed transaction.

### Hiểu lầm phổ biến
- "Sharding làm mọi thứ nhanh hơn" — làm *write throughput tổng* cao hơn; từng query có thể *chậm đi* (thêm hop routing, scatter-gather).
- "Auto-sharding nghĩa là không cần nghĩ về key" — DynamoDB vẫn chết vì hot key như thường; auto chỉ chuyển *cách* bạn chết.
- "Thêm node là hết hot partition" — key nóng vẫn nằm trên MỘT node; thêm 100 node không đổi điều đó.

### Câu hỏi tự kiểm tra
1. Thiết kế partition key cho bảng message của app chat 50M user (query: lịch sử theo hội thoại, gần nhất trước). Vì sao `(conversation_id, message_time)` hơn `message_id`? Hội thoại nhóm 100K thành viên thì sao?
2. Hệ IoT ghi sensor data theo thời gian trên Cassandra. Vì sao `sensor_id` làm partition key sẽ chết dần theo năm tháng? (gợi ý: unbounded partition growth) — sửa thế nào? (`(sensor_id, day_bucket)`)
3. Chứng minh bằng số: 4 node cache, mod-4, thêm node thứ 5 — bao nhiêu % key đổi chỗ? Với consistent hashing?
4. Khi nào bạn khuyên dùng Vitess/Citus (shard PostgreSQL/MySQL) thay vì chuyển hẳn sang Cassandra?

### Tài liệu kinh điển nên đọc
- **"Consistent Hashing and Random Trees" (Karger et al., 1997)** — paper gốc, sinh ra cho web caching, giờ ở khắp nơi.
- **"Dynamo" (2007)** — consistent hashing + vnodes trong production đầu tiên được mô tả kỹ.
- **"Bigtable" (Google, 2006)** — range partitioning + tablet split: tổ tiên của HBase, nền tư duy của Spanner/CockroachDB.
- **"How Sharding Works" (Jeremy Cole / Vitess docs)** — sharding MySQL thực chiến ở quy mô YouTube.
- **DynamoDB adaptive capacity docs + re:Invent talks** — cách một hệ managed xử lý hot partition thực tế, rất đáng học cả khi không dùng AWS.
