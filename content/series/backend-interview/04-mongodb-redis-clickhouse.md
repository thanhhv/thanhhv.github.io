+++
title = "Bài 4 — MongoDB, Redis & ClickHouse"
date = "2026-07-03T11:00:00+07:00"
draft = false
tags = ["backend", "database", "interview"]
series = ["Backend Interview"]
+++

---

# PHẦN A — MONGODB

## Câu 1 — [Intermediate → Senior] Replica Set hoạt động thế nào? Điều gì xảy ra khi failover?

### 1. Câu hỏi
"Trình bày cơ chế replica set của MongoDB: bầu chọn, oplog, và các mức write/read concern. Khi primary chết, dữ liệu có thể mất không?"

### 2. Interviewer muốn kiểm tra điều gì?
- Hiểu consensus/election thực tế (Raft-like) chứ không chỉ "có 3 node tự failover".
- Nắm write concern / read concern — nơi ẩn chứa các quyết định mất-hay-không-mất dữ liệu.
- Kinh nghiệm với rollback sau failover — thứ ít người biết cho tới khi gặp.

### 3. Câu trả lời ngắn gọn (30 giây)
"Replica set gồm 1 primary nhận ghi và các secondary kéo **oplog** (log thao tác idempotent) để replay. Election dựa trên giao thức giống Raft: cần đa số phiếu, node có oplog mới nhất được ưu tiên; vì cần majority nên phải có số node lẻ hoặc arbiter. Có mất dữ liệu không phụ thuộc **write concern**: `w:1` (ack từ primary) — ghi chưa kịp replicate sẽ bị **rollback** khi failover; `w:majority` — an toàn qua failover, đổi lấy latency. Read concern và read preference quyết định bạn đọc thấy gì và từ đâu."

### 4. Câu trả lời Senior Level (3–5 phút)
**How:**
- **Oplog:** capped collection ghi thao tác đã áp dụng, dạng idempotent (update trở thành set giá trị cuối) để replay an toàn. Secondary kéo oplog liên tục; oplog window (thời gian oplog phủ) là chỉ số vận hành quan trọng — secondary rớt lâu hơn window phải initial sync lại toàn bộ.
- **Election (từ pSv1, dựa trên Raft):** heartbeat 2s, primary mất liên lạc ~10s → secondary ứng cử; cần majority; term tăng dần chống split-brain: primary cũ bị cô lập tự giáng cấp vì không còn liên lạc được với đa số → không bao giờ có 2 primary **được majority công nhận** cùng lúc.
- **Rollback:** primary cũ có ghi chưa replicate mà cluster đã bầu primary mới → khi quay lại, các ghi đó bị gỡ và ném vào file rollback — dữ liệu **đã ack với client** (nếu w:1) biến mất khỏi DB. Đây là điểm nhấn Senior.
- **Write concern:** `w:1` nhanh, rủi ro rollback; `w:majority` sống sót failover; `j:true` chờ journal xuống đĩa. **Read concern:** `local` (có thể thấy dữ liệu sẽ bị rollback), `majority` (chỉ thấy dữ liệu bền), `linearizable` (đắt, single-document). **Read preference:** đọc secondary = chấp nhận stale.
- Causal consistency (session) giải quyết read-your-writes khi đọc secondary.

**Trade-off & Production:** `w:majority` + `readConcern:majority` là mặc định đúng cho dữ liệu quan trọng từ MongoDB 5. PSA (Primary-Secondary-Arbiter) tiết kiệm tiền nhưng khi một data node chết, `w:majority` treo — arbiter không giữ dữ liệu. Tôi luôn khuyên PSS thay vì PSA cho hệ nghiêm túc.

### 5. Giải thích bản chất
Bài toán gốc là **consensus**: nhiều node phải thống nhất một chuỗi thao tác duy nhất dù có node chết và mạng phân mảnh. Định lý cơ bản: quyết định an toàn cần **đa số** — vì hai đa số bất kỳ luôn giao nhau, đảm bảo thông tin không "quên". Mọi thiết kế (số node lẻ, arbiter, rollback) đều suy ra từ đây. Rollback là hệ quả tất yếu của việc cho phép ack thiểu số (`w:1`): hệ thống cho bạn *chọn* đứng ngoài vùng an toàn của consensus để đổi lấy latency — và nhiều team chọn mà không biết mình đã chọn. **Nếu thiết kế khác:** bắt buộc w:majority mọi lúc (như etcd) → không bao giờ rollback nhưng mọi ghi trả +1 RTT; cho phép multi-primary → phải merge conflict (hướng của Couchbase/Dynamo) — Mongo chọn single-primary để giữ ngữ nghĩa đơn giản.

### 6. Trade-off
- **w:1 vs w:majority:** latency thấp ↔ khả năng mất ghi đã ack khi failover. Chọn theo giá trị dữ liệu, per-operation.
- **Đọc secondary:** giảm tải primary ↔ stale read + phá read-your-writes; secondary còn dùng cho DR — dồn tải đọc lên nó là ăn vào margin an toàn.
- **PSA vs PSS:** rẻ hơn một node ↔ mất khả năng w:majority khi một data node chết.
- **Oplog lớn:** window dài, chịu được secondary rớt lâu ↔ tốn đĩa.

### 7. Ví dụ Production
Sự cố thật kiểu mẫu: hệ đơn hàng dùng `w:1` cho nhanh. Network partition 15 giây, failover xảy ra, 41 đơn hàng đã trả "success" cho khách nằm trong rollback file. Khách nhận email xác nhận, DB không có đơn. Xử lý hậu quả mất một tuần (đối soát từ log + rollback bson). Fix: `w:majority` cho collection orders (latency ghi tăng ~2ms — không ai nhận ra), `w:1` giữ cho analytics events. Bài học: write concern là quyết định **per-dữ-liệu**, không phải config toàn cục.

### 8. Những câu trả lời chưa đủ tốt
- "Replica set tự failover nên an toàn." → Failover ≠ không mất dữ liệu. Phải nói tới write concern và rollback.
- "Dùng 3 node là chuẩn." → Tại sao 3? (majority của 3 = 2, chịu được 1 node chết; 4 node majority = 3, vẫn chỉ chịu 1 — thêm node chẵn không thêm khả năng chịu lỗi.)

### 9. Sai lầm phổ biến của ứng viên
- Không biết rollback tồn tại — tin rằng ack là vĩnh viễn bất kể write concern.
- Nghĩ arbiter là node "dự phòng nhẹ" vô hại — không hiểu nó phá w:majority khi thiếu data node.
- Đặt số node chẵn; hoặc rải 2 node ở DC chính + 1 ở DC phụ mà không nghĩ đến việc DC chính sập là mất majority.
- Nhầm read preference (đọc từ đâu) với read concern (đọc dữ liệu ở mức bền nào).
- Không giám sát replication lag và oplog window.

### 10. Follow-up Questions
- Tại sao oplog phải idempotent? Điều gì xảy ra nếu replay 2 lần một thao tác `$inc`?
- Thiết kế replica set 2 region: 5 node đặt thế nào để region phụ sập không ảnh hưởng, region chính sập vẫn tự failover?
- `majority` commit point được tính thế nào? Read concern majority có chặn không nếu replication tắc?
- So sánh election của MongoDB với Raft nguyên bản — khác gì (priority, pre-vote)?
- Client driver làm gì trong lúc election ~10s? (retryable writes hoạt động ra sao, cần idempotency gì?)

### 11. Liên hệ với Production
Các hệ lớn dùng Mongo (eBay, Baidu) đều có chuẩn nội bộ: dữ liệu giao dịch bắt buộc w:majority + retryable writes. Vấn đề nghiêm trọng khi: mạng giữa các AZ chập chờn (election liên tục — mỗi lần ~10s không nhận ghi), hoặc bulk write đẩy replication lag vượt oplog window. Dấu hiệu cần hành động: election count tăng, `replSetGetStatus` lag tăng dần, xuất hiện rollback file trên node.

---

## Câu 2 — [Senior → Staff] Sharding trong MongoDB: chọn shard key thế nào và tại sao chọn sai là thảm họa?

### 1. Câu hỏi
"Kiến trúc sharded cluster của MongoDB? Nguyên tắc chọn shard key, và điều gì xảy ra khi chọn sai?"

### 2. Interviewer muốn kiểm tra điều gì?
- Tư duy thiết kế theo access pattern — kỹ năng cốt lõi của mọi hệ phân tán, không riêng Mongo.
- Hiểu scatter-gather, hot shard, jumbo chunk — hậu quả cụ thể của shard key tồi.
- Mức Staff: hiểu tính "gần như một chiều" của quyết định và cách giảm rủi ro.

### 3. Câu trả lời ngắn gọn (30 giây)
"Cluster gồm: shard (mỗi shard là một replica set), config server (metadata chunk), mongos (router). Dữ liệu chia thành chunk theo shard key; balancer di chuyển chunk giữa các shard. Shard key tốt phải đạt ba điều: **cardinality cao**, **phân bố ghi đều**, và **xuất hiện trong hầu hết query** để route thẳng tới một shard. Chọn sai gây hai thảm họa đối xứng: key đơn điệu (timestamp, ObjectId) dồn mọi ghi vào một shard (hot shard); key không có trong query biến mọi truy vấn thành scatter-gather ra toàn bộ shard — thêm shard không thêm throughput."

### 4. Câu trả lời Senior Level (3–5 phút)
**How:**
- Chunk (~128MB mặc định) là đơn vị phân phối; balancer di chuyển chunk khi lệch. Ranged sharding giữ locality cho range query; hashed sharding phân bố đều nhưng giết range query.
- **Targeted vs scatter-gather:** query chứa shard key → mongos route tới đúng shard (scale tuyến tính); không chứa → hỏi mọi shard rồi merge (fan-out — thêm shard làm p99 **tệ hơn** vì chờ shard chậm nhất; xem "tail at scale").
- **Hot shard:** key đơn điệu tăng → mọi insert rơi vào chunk cuối → một shard nhận 100% ghi, các shard khác ngồi chơi; balancer đuổi theo không kịp.
- **Jumbo chunk:** quá nhiều document cùng giá trị key (ví dụ shard theo `country`) → chunk không chia nhỏ được, không di chuyển được.
- **Giải pháp thực dụng:** compound key `{tenantId: 1, timestamp: 1}` — ghi phân tán theo tenant, query theo tenant vẫn targeted; hashed key cho pure key-value workload; Mongo 5+ cho phép reshard nhưng là thao tác nặng, không phải cứu cánh thường trực.

**Trade-off:** sharding đổi độ phức tạp lấy dung lượng — mất unique constraint toàn cục (trừ khi chứa shard key), transaction xuyên shard đắt (2PC), `count()` xấp xỉ, backup/restore phức tạp hơn nhiều.

**Production:** quyết định quan trọng nhất xảy ra **trước khi có dữ liệu**: liệt kê top query theo tần suất, kiểm tra từng ứng viên shard key trên cả ba tiêu chí với **dữ liệu tương lai mô phỏng**, không phải dữ liệu hiện tại.

### 5. Giải thích bản chất
Sharding là hash/range partitioning — bài toán tổng quát: **ánh xạ không gian khóa vào không gian máy sao cho tải đều và truy vấn cục bộ**. Hai mục tiêu này mâu thuẫn tự nhiên: phân bố đều nhất là hash ngẫu nhiên (mất mọi locality), locality tốt nhất là giữ nguyên thứ tự (dễ lệch tải). Mọi thiết kế shard key là tìm điểm giữa hai cực này **cho access pattern cụ thể của bạn** — vì thế không tồn tại shard key tốt phổ quát, và interviewer hỏi câu này để xem bạn có bắt đầu từ access pattern không. Cùng nguyên lý này lặp lại ở Kafka partition key, Cassandra partition key, DynamoDB partition key, Postgres partitioning — trả lời được một lần là trả lời được cả họ câu hỏi.

### 6. Trade-off
- **Ranged vs Hashed:** range query hiệu quả ↔ nguy cơ hot shard; hashed đều tải ↔ range query = scatter-gather.
- **Shard sớm:** tránh migration đau sau này ↔ trả chi phí vận hành trước khi cần, và chọn key khi hiểu biết về access pattern còn ít nhất — nghịch lý thời điểm.
- **Nhiều shard nhỏ vs ít shard to:** granularity cân bằng tải ↔ scatter-gather đắt hơn, nhiều máy hơn để hỏng.

### 7. Ví dụ Production
Case kinh điển tôi dùng khi dạy: hệ IoT shard theo `deviceId` hashed — ghi đều tuyệt đẹp. Sáu tháng sau, dashboard cần "mọi thiết bị của công ty X trong 24h qua" — query không có deviceId cụ thể → scatter-gather 64 shard, p99 5 giây. Reshard sang `{orgId: 1, deviceId: 1}` mất 3 tuần với live migration hai lần ghi song song. Nếu ngày đầu dành 2 giờ liệt kê query tương lai, đã tránh được. Bài học Staff: **shard key được chọn bởi câu hỏi bạn sẽ hỏi, không phải dữ liệu bạn sẽ chứa**.

### 8. Những câu trả lời chưa đủ tốt
- "Chọn key có cardinality cao." → Một trong ba tiêu chí. `ObjectId` cardinality cực cao và vẫn là thảm họa (đơn điệu).
- "Hashed cho đều là được." → Đều ghi, nhưng query của bạn có key để target không? Range query thì sao?

### 9. Sai lầm phổ biến của ứng viên
- Shard theo timestamp/ObjectId — lỗi số một, tạo hot shard ghi.
- Không phân biệt targeted query và scatter-gather; tưởng thêm shard luôn tăng throughput đọc.
- Quên rằng unique index toàn cục không còn (trừ khi prefix bằng shard key).
- Không biết jumbo chunk; shard theo trường low-cardinality.
- Coi resharding là việc dễ ("sau này đổi") — nó là dự án hàng tuần với rủi ro cao.

### 10. Follow-up Questions
- Transaction xuyên shard hoạt động thế nào? Chi phí so với single-shard?
- Nếu một shard chứa khách hàng chiếm 40% traffic (celebrity tenant) thì làm gì? (sub-shard tenant đó, tách riêng, thêm hậu tố ngẫu nhiên vào key + gather đọc.)
- Config server chết thì cluster còn hoạt động không?
- So sánh chiến lược của Mongo với consistent hashing của Cassandra/Dynamo — vì sao Mongo cần balancer còn Dynamo không?
- Zone sharding dùng cho bài toán gì? (data residency — dữ liệu EU nằm shard EU.)

### 11. Liên hệ với Production
Các SaaS multi-tenant lớn gần như luôn shard theo tenant trước tiên — vì mọi query đều trong phạm vi tenant. Vấn đề nghiêm trọng khi: một tenant phình to vượt một shard, hoặc tính năng mới đưa ra query xuyên tenant (analytics) đập vỡ giả định routing. Dấu hiệu cần hành động: phân phối ops/s giữa các shard lệch > 2–3 lần, tỷ lệ scatter-gather tăng trong profiler của mongos, balancer chạy liên tục không hội tụ.

---

## Câu 3 — [Senior] Data Modeling trong MongoDB: embed hay reference? Transaction đa document nói lên điều gì?

### 1. Câu hỏi
"Nguyên tắc quyết định embed vs reference trong MongoDB? Tại sao MongoDB khuyên rằng nếu bạn cần transaction đa document thường xuyên thì model của bạn có vấn đề?"

### 2. Interviewer muốn kiểm tra điều gì?
- Tư duy modeling theo access pattern — khác biệt căn bản với tư duy chuẩn hóa của RDBMS.
- Hiểu document = ranh giới atomic tự nhiên.
- Không cuồng cũng không bài xích NoSQL — đánh giá theo bài toán.

### 3. Câu trả lời ngắn gọn (30 giây)
"Nguyên tắc: **dữ liệu được đọc cùng nhau thì lưu cùng nhau**. Embed khi quan hệ là 'chứa', đọc kèm nhau, phía nhiều có giới hạn (địa chỉ trong user); reference khi phía nhiều không giới hạn (đơn hàng của user), truy cập độc lập, hoặc nhiều phía trỏ vào (many-to-many). Thao tác trên **một** document là atomic sẵn — model tốt đặt ranh giới nhất quán trùng ranh giới document, nên hiếm khi cần transaction đa document. Cần nó thường xuyên nghĩa là bạn đang model quan hệ kiểu SQL trong một document store — chọn sai công cụ hoặc sai model."

### 4. Câu trả lời Senior Level (3–5 phút)
**Why khác RDBMS:** chuẩn hóa của SQL tối ưu cho **ghi không dị thường** (mỗi sự thật một chỗ) và join lo phần đọc. Mongo không có join rẻ → tối ưu cho **đọc theo hình dạng dữ liệu**: câu hỏi đầu tiên không phải "các entity quan hệ gì" mà là "**query nào chạy nhiều nhất và nó cần hình dạng dữ liệu nào**".

**Bộ nguyên tắc:**
- Embed: one-to-few, đọc cùng nhau, vòng đời gắn với cha. Cẩn thận: document 16MB limit; mảng tăng vô hạn (unbounded array) là anti-pattern chết người — comment trong post viral phình mãi, mỗi update ghi lại document càng lúc càng to, move trên đĩa.
- Reference: one-to-many lớn, many-to-many, dữ liệu truy cập độc lập, dữ liệu thay đổi tần suất khác nhau.
- **Extended reference / duplication có chủ đích:** nhúng bản sao vài trường hay đọc (tên, avatar người bán trong order) — chấp nhận denormalization, trả giá bằng cập nhật lan truyền (và tự hỏi: trường đó có *cần* mới nhất không? Order lưu tên người bán **tại thời điểm mua** — duplication ở đây còn đúng hơn về nghiệp vụ!).
- Các pattern chuẩn: bucket (time-series gom theo giờ/ngày), subset (10 review mới nhất embed, còn lại collection riêng), computed (đếm sẵn), schema versioning.

**Transaction đa document:** có từ 4.0/4.2, dùng snapshot + WiredTiger, chi phí cao hơn hẳn single-doc (giữ snapshot, conflict retry, giới hạn thời gian 60s mặc định). Nó là **lưới an toàn cho trường hợp hiếm** (chuyển tiền giữa 2 ví), không phải công cụ hàng ngày. Nghiệp vụ toàn quan hệ + ràng buộc chéo → Postgres đúng hơn, và nói thẳng điều đó trong phỏng vấn là điểm cộng.

### 5. Giải thích bản chất
Câu hỏi sâu xa: **ranh giới nhất quán (consistency boundary) của nghiệp vụ nằm ở đâu?** RDBMS cho bạn ranh giới co giãn tùy ý (transaction bọc bao nhiêu bảng tùy thích) — trả giá bằng lock, phức tạp khi phân tán. Document store bắt bạn **tuyên bố trước** ranh giới đó bằng cấu trúc document — đổi lại atomic rẻ, shard tự nhiên. Đây chính là khái niệm **aggregate trong DDD** hiện thân vào storage: một document = một aggregate = một đơn vị nhất quán. Nếu bạn không vẽ được ranh giới aggregate ổn định cho nghiệp vụ, đó là tín hiệu nghiệp vụ mang tính quan hệ cao — và document store là lựa chọn sai từ gốc, không phải "dùng chưa đúng cách".

### 6. Trade-off
- **Embed:** đọc 1 lần có tất cả, atomic tự nhiên ↔ document phình, ghi lại nhiều, không truy cập phần con độc lập, 16MB trần.
- **Reference:** linh hoạt, không giới hạn ↔ N+1 query hoặc `$lookup` (đắt, không dùng được trên sharded collection tùy trường hợp).
- **Duplication:** đọc nhanh ↔ ghi lan truyền + nguy cơ lệch; chấp nhận được khi trường ít đổi hoặc cần giá trị tại-thời-điểm.
- **Schema linh hoạt:** tiến hóa nhanh ↔ validation đẩy về app; "schemaless" thực tế nghĩa là "schema ngầm rải trong code".

### 7. Ví dụ Production
Anti-pattern tôi gặp nhiều nhất: social app embed toàn bộ comment vào post. Post viral 80k comment → document 14MB sát trần 16MB → mỗi comment mới ghi lại 14MB → WiredTiger cache bốc cháy, replication lag vọt. Fix: subset pattern (20 comment mới nhất embed cho màn hình đầu) + collection comments riêng bucket theo postId+trang. Ngược lại, ví dụ dùng đúng: catalog sản phẩm với thuộc tính biến thiên theo ngành hàng (áo có size/màu, laptop có RAM/CPU) — document model đẹp hơn hẳn EAV table trong SQL.

### 8. Những câu trả lời chưa đủ tốt
- "Embed cho nhanh, reference khi dữ liệu lớn." → Thiếu tiêu chí quyết định thật: đọc cùng nhau? bounded? vòng đời? truy cập độc lập?
- "MongoDB giờ có transaction rồi nên như SQL." → Chi phí khác, giới hạn khác, và câu hỏi vẫn là: tại sao model của bạn cần nó thường xuyên?

### 9. Sai lầm phổ biến của ứng viên
- Model kiểu bảng chuẩn hóa trong Mongo (mỗi entity một collection, reference mọi nơi) → mất mọi ưu điểm, giữ mọi nhược điểm.
- Unbounded array trong document.
- Không biết 16MB limit và hệ quả của document lớn với cache/replication.
- Lạm dụng `$lookup` như join SQL rồi than Mongo chậm.
- Không version schema — sau 2 năm, 5 hình dạng document trong cùng collection và code đầy `if (doc.v === undefined)`.

### 10. Follow-up Questions
- Thiết kế schema cho chat app: message, conversation, participant — embed gì, reference gì, bucket không?
- Aggregation pipeline có những stage nào giết hiệu năng? (`$lookup` trên collection lớn, `$group` không dùng được index, `$unwind` nhân bản document, sort thiếu index vượt memory limit.)
- Change stream hoạt động dựa trên gì? Dùng cho pattern nào? (oplog; CDC, cache invalidation.)
- Nếu cần cả document linh hoạt lẫn transaction quan hệ mạnh — kiến trúc lai thế nào? (Postgres JSONB đủ không? Tách domain?)
- Time-series collection của Mongo 5+ khác gì tự làm bucket?

### 11. Liên hệ với Production
Các hệ dùng Mongo thành công (catalog thương mại điện tử, CMS, IoT telemetry) đều có điểm chung: access pattern đọc rõ ràng, aggregate boundary tự nhiên. Vấn đề nghiêm trọng khi: nghiệp vụ trưởng thành đẻ ra nhu cầu ad-hoc query và ràng buộc chéo (tài chính, đối soát) — lúc đó dữ liệu thường được CDC sang warehouse/Postgres. Dấu hiệu model có vấn đề: document size p99 tăng theo thời gian, tỷ lệ `$lookup` trong aggregation tăng, transaction đa document xuất hiện trong hot path.

---

# PHẦN B — REDIS

## Câu 4 — [Intermediate → Senior] Redis Persistence: RDB vs AOF — mất dữ liệu khi nào?

### 1. Câu hỏi
"Redis có những cơ chế persistence nào? Cấu hình thế nào cho từng loại use case, và kịch bản mất dữ liệu của mỗi cơ chế?"

### 2. Interviewer muốn kiểm tra điều gì?
- Hiểu Redis trước hết là in-memory — persistence là tính năng phụ có giới hạn rõ.
- Phân biệt được cache (mất được) và datastore (không mất được) — và Redis đang đóng vai nào trong hệ của bạn.
- Chi tiết vận hành: fork, copy-on-write, fsync policy.

### 3. Câu trả lời ngắn gọn (30 giây)
"RDB: snapshot định kỳ bằng cách fork process con ghi toàn bộ dataset — file nhỏ gọn, restore nhanh, nhưng crash mất mọi thứ từ snapshot cuối (phút tới giờ). AOF: append từng lệnh ghi, fsync theo policy — `everysec` (mặc định, mất tối đa ~1s), `always` (chậm, gần như không mất). AOF rewrite nén file định kỳ, cũng qua fork. Thực dụng: cache thuần → tắt persistence hoặc RDB thưa; dữ liệu cần giữ → AOF everysec + RDB làm backup; và **đừng bao giờ** coi Redis là source of truth duy nhất cho dữ liệu không thể mất — kể cả AOF always cũng không có ngữ nghĩa bền vững qua replication + failover như một DB thực thụ."

### 4. Câu trả lời Senior Level (3–5 phút)
**RDB:** `BGSAVE` fork; process con ghi snapshot nhờ **copy-on-write** — cha tiếp tục phục vụ, page bị sửa mới bị copy. Hệ quả vận hành quan trọng: workload ghi nhiều trong lúc BGSAVE → COW copy ồ ạt → **memory usage có thể tiến gần gấp đôi** → OOM killer. Đây là lý do quy hoạch RAM Redis chỉ ~50–60% máy khi có persistence. Fork trên heap lớn (100GB) cũng tốn thời gian đáng kể (copy page table) → latency spike định kỳ.

**AOF:** log lệnh, ba mức fsync. `everysec` là điểm cân bằng phổ biến. File phình → **AOF rewrite** (cũng fork, cũng COW). Từ Redis 7, Multi-Part AOF giảm đau rewrite. Restore = replay toàn bộ (chậm hơn RDB nhiều với dataset lớn) → thực tế dùng hybrid RDB+AOF (RDB preamble).

**Kịch bản mất dữ liệu ít người nói tới:** persistence chỉ cứu **node đó**. Failover sang replica **async** có thể mất nhiều hơn mọi cấu hình fsync — replica nhận dữ liệu trễ, được bầu làm master, các ghi chưa kịp sang bị vứt. Muốn chặt hơn có `WAIT` (chờ N replica ack) nhưng nó không phải sync replication đúng nghĩa và Redis không phải CP system. Kết luận Senior: **độ bền của Redis = min(persistence, replication) và cả hai đều best-effort**.

**Trade-off & Production:** cache thuần tắt AOF (tránh I/O và fork spike vô ích); session/leaderboard chấp nhận mất 1s → AOF everysec; hàng đợi/counter tiền bạc → đừng để một mình Redis giữ.

### 5. Giải thích bản chất
Toàn bộ giá trị của Redis đến từ một quyết định: **dataset nằm trọn trong RAM, xử lý single-thread trên cấu trúc dữ liệu tối ưu** → độ trễ µs và ngữ nghĩa atomic đơn giản. Persistence vì thế mãi mãi là "công dân hạng hai": mọi cơ chế bền vững hóa đều phải tránh chạm vào hot path (nên mới fork + COW thay vì ghi đồng bộ). So sánh với Postgres: WAL nằm **trước** ack trên hot path — đó là khác biệt bản chất giữa datastore-bền-vững và in-memory-store-có-persistence, không phải khác biệt về "cấu hình đúng". Hiểu tầng này thì mọi câu "Redis có mất dữ liệu không" tự trả lời được.

### 6. Trade-off
- **RDB:** restore nhanh, file gọn, backup đẹp ↔ RPO tính bằng phút; fork spike.
- **AOF always:** RPO ~0 trên node ↔ throughput giảm mạnh (fsync mỗi lệnh); vẫn không cứu failover async.
- **everysec:** cân bằng ↔ mất tới 1s — phải là quyết định có ý thức của nghiệp vụ, không phải mặc định vô thức.
- **Persistence bật trên replica, tắt trên master** là pattern giảm tải master — đổi lấy phức tạp vận hành.

### 7. Ví dụ Production
Sự cố đáng nhớ: Redis 60GB làm cache + counter khuyến mãi, RDB mỗi 5 phút. Black Friday: write rate ×10 → BGSAVE kích hoạt → COW copy gần như toàn bộ → RAM chạm trần, Linux OOM killer bắn chết Redis → toàn bộ counter khuyến mãi bay màu, restore từ snapshot 4 phút tuổi → khách claim trùng mã giảm giá, thiệt hại thật bằng tiền. Ba cái sai chồng nhau: quy hoạch RAM không tính COW, counter tiền bạc nằm một mình trong Redis, và không alert trên `rdb_last_bgsave_status`. Bài học: sự cố Redis hiếm khi do Redis — do người vận hành coi nó như DB.

### 8. Những câu trả lời chưa đủ tốt
- "Bật AOF là không mất dữ liệu." → fsync policy nào? Failover async thì sao? Câu này lộ ngay chưa nghĩ hết đường dữ liệu.
- "RDB cho backup, AOF cho durability." → Đúng sách vở; thiếu fork/COW và chi phí thật trên production.

### 9. Sai lầm phổ biến của ứng viên
- Không biết BGSAVE/AOF-rewrite fork và hệ quả COW memory.
- Nghĩ persistence cứu được failover (nhầm phạm vi: node vs cluster).
- Đặt maxmemory = 90–100% RAM máy.
- Không phân biệt Redis-làm-cache và Redis-làm-store trong thiết kế — một instance đóng cả hai vai với một cấu hình.
- Không biết eviction policy (`allkeys-lru`, `noeviction`...) tương tác với vai trò: store mà đặt allkeys-lru là tự xóa dữ liệu của mình.

### 10. Follow-up Questions
- `maxmemory-policy` nào cho cache, nào cho store? `volatile-lru` vs `allkeys-lru`?
- Latency spike mỗi 5 phút đúng chu kỳ — nghi ngờ gì, xác nhận thế nào? (fork; `latency history`, `INFO persistence`.)
- Redis 7 Function/Multi-part AOF thay đổi gì?
- Nếu cần một KV store bền vững thật sự với API kiểu Redis — có lựa chọn nào? (KeyDB/Dragonfly vẫn in-memory; hướng đĩa: Kvrocks trên RocksDB, hoặc chấp nhận dùng DB thường + cache.)
- `WAIT` command đảm bảo gì và KHÔNG đảm bảo gì?

### 11. Liên hệ với Production
GitHub, Twitter dùng Redis khổng lồ nhưng luôn ở vai cache/derived data — nguồn sự thật nằm nơi khác. Vấn đề nghiêm trọng khi: Redis "tạm thời" giữ dữ liệu nghiệp vụ rồi thành vĩnh viễn (rate limit → billing counter → không ai nhớ nó chỉ là cache). Dấu hiệu cần rà soát: `used_memory` > 60% RAM kèm persistence bật, latency spike chu kỳ, và câu hỏi kiểm toán đơn giản nhất: "nếu flush toàn bộ Redis ngay bây giờ, hệ thống mất gì vĩnh viễn?" — nếu câu trả lời khác "chỉ mất tốc độ", bạn có một quả bom.

---

## Câu 5 — [Senior → Staff] Distributed Lock bằng Redis: SETNX, Redlock và các cạm bẫy

### 1. Câu hỏi
"Implement distributed lock bằng Redis thế nào cho đúng? Redlock là gì và tranh cãi quanh nó? Khi nào KHÔNG nên dùng Redis lock?"

### 2. Interviewer muốn kiểm tra điều gì?
- Câu hỏi phân loại Staff kinh điển: đụng tới thời gian, mạng, và tính đúng đắn phân tán.
- Biết cả cách làm đúng lẫn giới hạn không thể vượt (fencing token, tranh luận Kleppmann–antirez).
- Phân biệt lock cho **efficiency** vs lock cho **correctness** — khung tư duy quyết định mọi thứ.

### 3. Câu trả lời ngắn gọn (30 giây)
"Lock đơn giản: `SET key uuid NX PX 30000` — atomic, có TTL chống deadlock khi holder chết, value ngẫu nhiên để chỉ chủ mới unlock (unlock bằng Lua script check-and-del). Vấn đề còn lại: TTL hết trong khi việc chưa xong (GC pause, network stall) → hai holder đồng thời. Redlock chạy trên 5 node độc lập lấy majority — nhưng Kleppmann chỉ ra nó vẫn dựa trên giả định timing, không an toàn tuyệt đối. Khung quyết định: lock để **tiết kiệm** (tránh làm trùng việc — thi thoảng trùng cũng không sao) → Redis lock đơn là đủ; lock để **đúng đắn** (không được phép có 2 holder) → cần fencing token và resource cuối phải kiểm tra token, hoặc dùng lock của hệ CP (ZooKeeper/etcd), hoặc tốt nhất: thiết kế lại để không cần lock."

### 4. Câu trả lời Senior Level (3–5 phút)
**Xây từng lớp:**
1. `SETNX` trần: holder chết → deadlock vĩnh viễn. Thêm TTL.
2. TTL nhưng unlock bằng `DEL` trần: process A bị chậm, TTL hết, B lấy lock, A tỉnh dậy DEL lock **của B**. → value ngẫu nhiên + Lua script `if GET==uuid then DEL` (atomic check-and-delete).
3. Vấn đề không thể vá bằng code trên client: **A cầm lock, bị GC pause/VM freeze/network stall 40s, TTL 30s hết, B lấy lock — giờ A và B cùng tin mình giữ lock.** A tỉnh dậy và ghi vào storage. Không cách nào trên A phát hiện kịp (kiểm tra rồi mới ghi vẫn có khe hở giữa kiểm tra và ghi).
4. **Fencing token:** lock service phát số đơn điệu tăng theo mỗi lần cấp lock; **storage cuối cùng** từ chối ghi có token nhỏ hơn token đã thấy. Đây là giải duy nhất kín kẽ — và điều kiện của nó: resource phải hỗ trợ kiểm tra token (DB thì được — so version trong WHERE; gọi API bên thứ ba thì chịu).
5. **Redlock:** lấy lock trên ≥3/5 Redis độc lập trong thời gian ngắn hơn nhiều so với TTL. Antirez cho rằng đủ an toàn thực dụng; Kleppmann phản biện: không có fencing token thì mọi lock dựa TTL đều unsafe trước GC pause — và nếu đã cần đúng tuyệt đối, dùng consensus system thật. Quan điểm thực dụng của tôi: Redlock phức tạp hơn nhiều mà không đóng được lỗ hổng bản chất → hoặc single Redis lock cho efficiency, hoặc etcd/ZooKeeper + fencing cho correctness; Redlock đứng ở giữa một cách khó xử.

**Không cần lock thì tốt hơn:** idempotency key, unique constraint trong DB, optimistic concurrency (version column), queue tuần tự hóa theo key (partition theo resource) — đa số bài toán "cần distributed lock" tan biến khi thiết kế lại.

### 5. Giải thích bản chất
Gốc rễ: trong hệ phân tán **không có "bây giờ" chung và không có phán đoán chết/sống đáng tin** (FLP, async network). Lock dựa TTL là dùng **thời gian** thay cho **consensus** — nhưng đồng hồ trôi, process ngưng bất kỳ lúc nào (GC, preemption, live migration), nên "TTL chưa hết" trên client không chứng minh được gì. Fencing token đổi cách tiếp cận: thay vì cố bảo đảm chỉ một holder (bất khả thi tuyệt đối), nó bảo đảm **hệ quả ghi của holder cũ bị vô hiệu** — chuyển tính đúng từ thời gian (không tin được) sang thứ tự (kiểm chứng được tại storage). Đây là một bài học thiết kế tổng quát: khi không thể ngăn điều xấu xảy ra, hãy làm cho hậu quả của nó vô hại.

### 6. Trade-off
- **Single Redis lock:** đơn giản, nhanh (µs), đủ cho efficiency ↔ SPOF, unsafe qua failover (lock "bốc hơi" khi replica lên master chưa nhận key).
- **Redlock:** bớt SPOF ↔ 5 node để vận hành, vẫn không kín GC-pause, độ phức tạp cao.
- **etcd/ZooKeeper:** an toàn consensus, có session/watch, sinh fencing token tự nhiên (revision/zxid) ↔ latency ms, thêm một hệ thống phải vận hành.
- **TTL ngắn:** giảm cửa sổ hai-holder ↔ việc dài phải gia hạn (watchdog) — watchdog chết là mất lock giữa chừng.

### 7. Ví dụ Production
Sự cố mẫu mực: cron job chạy trên 3 instance, lock Redis TTL 30s để chỉ một instance gửi email báo cáo. Một ngày, instance A bị stop-the-world GC 45 giây (heap khủng + full GC) đúng giữa lúc gửi → B lấy lock, gửi lại từ đầu → khách nhận email trùng. Vô hại vì đây là bài toán efficiency. Cùng pattern đó copy sang job **trừ tiền định kỳ** → trừ trùng tiền thật. Cùng một đoạn code, một bên chấp nhận được, một bên là sự cố nghiêm trọng — minh họa hoàn hảo cho khung efficiency vs correctness. Fix bên tiền: idempotency key theo (customer, billing_period) + unique constraint — hóa ra không cần lock ngay từ đầu.

### 8. Những câu trả lời chưa đủ tốt
- "Dùng SETNX với TTL là xong." → Unlock nhầm của người khác? TTL hết giữa chừng? Chưa chạm tới phần khó.
- "Dùng Redlock cho chuẩn." → Nói được tranh cãi Kleppmann–antirez chưa? Fencing token là gì? Trích tên thuật toán không thay được hiểu bản chất.

### 9. Sai lầm phổ biến của ứng viên
- Unlock bằng DEL không kiểm tra ownership.
- Không nghĩ tới GC pause/clock — tin rằng "TTL 30s là quá đủ" (chính niềm tin đó là lỗ hổng).
- Không phân biệt efficiency lock vs correctness lock — dùng một giải pháp cho cả hai.
- Không biết lock Redis biến mất khi failover async (master chết trước khi replicate key lock).
- Bỏ qua lựa chọn "thiết kế để không cần lock" — thường là câu trả lời đúng nhất.

### 10. Follow-up Questions
- Viết Lua script unlock an toàn? Tại sao phải Lua mà không phải GET rồi DEL?
- Watchdog gia hạn lock (kiểu Redisson) giải quyết gì, và lỗ hổng còn lại?
- Fencing token lấy từ đâu nếu dùng etcd? (revision). Nếu resource là API bên thứ ba không check được token thì sao?
- Thiết kế job scheduler chạy-đúng-một-lần trên cluster — cần lock không, hay có cách khác? (leader election, queue có visibility timeout, unique constraint.)
- So sánh lock của ZooKeeper (ephemeral znode + session) với Redis TTL — khác bản chất chỗ nào? (liveness gắn session heartbeat vs thời gian tuyệt đối — nhưng session cũng dựa timeout, nên fencing vẫn cần!)

### 11. Liên hệ với Production
Kafka, Kubernetes dùng lease + fencing (epoch/generation number) khắp nơi — controller epoch của Kafka chính là fencing token quy mô lớn. Vấn đề nghiêm trọng khi: hệ thống lớn lên làm GC pause dài ra và network kém ổn định — lock "chạy ổn 2 năm" bắt đầu double-fire. Dấu hiệu cần rà soát: bất kỳ Redis lock nào bảo vệ thao tác có tiền/hàng/dữ liệu không đảo ngược được, job có log "lock acquired" từ hai instance trong cùng cửa sổ thời gian, và code không có idempotency ở resource cuối.

---

## Câu 6 — [Senior] Cache Patterns: cache-aside, stampede, và tính nhất quán cache–DB

### 1. Câu hỏi
"Trình bày các cache pattern chính. Cache stampede là gì và chống thế nào? Làm sao giữ cache và DB nhất quán?"

### 2. Interviewer muốn kiểm tra điều gì?
- Caching là chủ đề system design xuất hiện trong 90% buổi phỏng vấn — đây là phần phải thuộc nằm lòng ở mức cơ chế.
- Hiểu các failure mode: stampede, race update, hot key.
- Trung thực về giới hạn: cache-aside không bao giờ nhất quán tuyệt đối.

### 3. Câu trả lời ngắn gọn (30 giây)
"Cache-aside phổ biến nhất: đọc cache → miss → đọc DB → set cache; ghi thì **update DB rồi delete cache** (không update cache — tránh race ghi đè giá trị cũ). Stampede: key nóng hết hạn, nghìn request cùng miss cùng dội DB — chống bằng: lock/single-flight (một request rebuild, số còn lại chờ), TTL jitter (tránh hết hạn đồng loạt), stale-while-revalidate, hoặc xác suất hết hạn sớm. Nhất quán: cache-aside chỉ đạt eventual — cửa sổ lệch tồn tại do thứ tự delete/set đan xen; thu hẹp bằng TTL hợp lý + delete sau ghi + (nếu cần chặt) CDC từ DB invalidate cache. Cần đúng tuyệt đối thì đừng đọc từ cache cho luồng đó."

### 4. Câu trả lời Senior Level (3–5 phút)
**Các pattern:** cache-aside (app tự quản — linh hoạt, phổ biến); read-through/write-through (cache layer đứng giữa — nhất quán dễ hơn, thêm hạ tầng); write-behind (ghi cache, flush DB async — nhanh nhất, rủi ro mất ghi cao nhất — hiếm khi đáng); refresh-ahead.

**Tại sao "update DB rồi DELETE cache" là chuẩn:** so các phương án — (a) update cache rồi update DB: DB fail là cache sai vĩnh viễn; (b) update DB rồi update cache: hai writer đan xen → cache giữ giá trị cũ **không TTL nào cứu nổi nếu set lại liên tục**; race này thực tế xảy ra; (c) delete cache rồi update DB: giữa delete và update, reader nạp lại giá trị cũ vào cache; (d) **update DB rồi delete cache**: vẫn còn một race hẹp (reader đọc DB cũ trước khi writer commit, set cache sau khi writer delete) nhưng cửa sổ nhỏ nhất và tự lành nhờ TTL. Trình bày được chuỗi loại trừ này là điểm Senior — không phải đáp án, mà là **lý do loại từng phương án**.

**Stampede ba biến thể:** (1) key nóng expire → single-flight per key (Go `singleflight`; phân tán thì mutex Redis ngắn) + serve stale trong lúc rebuild; (2) hàng loạt key expire cùng lúc (deploy warm cache cùng TTL) → jitter TTL ±10–20%; (3) **cache penetration** — key không tồn tại trong DB nên không bao giờ được cache → cache negative result TTL ngắn hoặc Bloom filter chặn trước.

**Hot key:** một key nhận trăm nghìn RPS vượt một node Redis → local cache tầng app (L1, TTL vài giây) + nhân bản key (`key#1..N` đọc ngẫu nhiên).

**Nhất quán mạnh hơn:** CDC (Debezium đọc WAL → invalidate) loại bỏ phụ thuộc "app nhớ delete" và đúng cả khi ghi ngoài app; hoặc version trong key. Nhưng câu trả lời trung thực nhất: **cache đọc là eventual consistency có chủ đích — luồng nào không chịu được thì bypass cache**, đọc DB (và đó là thiểu số luồng, nên cache vẫn gánh 99% tải).

### 5. Giải thích bản chất
Cache là **bản sao dữ liệu không có giao thức đồng bộ** — khác replica DB (có WAL stream), cache–DB không có gì ràng buộc ngoài kỷ luật của app. Vì vậy mọi pattern chỉ là các mức độ thu hẹp cửa sổ lệch, không bao giờ đóng kín — đây là hệ quả trực tiếp của việc hai hệ thống không chia sẻ transaction. (Muốn đóng kín phải đưa cache vào transaction — tức biến nó thành index/materialized view của DB, mất tính độc lập và tốc độ — quay lại điểm xuất phát.) Stampede là hiện tượng tổng quát hơn cache: **mọi hệ có "làm lại thứ đắt đỏ khi hết hạn" đều có thundering herd** — DNS hết TTL, token OAuth hết hạn đồng loạt, connection pool refill. Học một lần, nhận diện mọi nơi.

### 6. Trade-off
- **TTL dài:** hit ratio cao, DB nhàn ↔ dữ liệu cũ lâu; TTL là nút vặn trực tiếp giữa tải và độ tươi.
- **Single-flight:** DB được bảo vệ ↔ request chờ nhau (p99 tăng khi rebuild chậm); serve-stale đổi độ tươi lấy latency.
- **Write-through:** đọc luôn ấm, nhất quán hơn ↔ ghi chậm hơn, cache những thứ không ai đọc.
- **Local L1 + Redis L2:** cứu hot key, giảm RTT ↔ N bản sao lệch nhau ngắn hạn, invalidation khó hơn (cần pub/sub).

### 7. Ví dụ Production
Sự cố có thật ở nhiều công ty, phiên bản tôi chứng kiến: trang chủ cache config khuyến mãi TTL 60s đúng, không jitter, 400 pod. Đúng giây :00, 400 pod cùng miss → 400 query nặng cùng đập vào Postgres → connection pool cạn → **mọi** endpoint dùng chung pool nghẽn theo → site down 4 phút, tự hồi phục, rồi lặp lại đúng 60 giây sau — down theo nhịp tim. Fix ba lớp: singleflight trong pod, mutex Redis giữa các pod, TTL jitter. Nhận diện "outage tuần hoàn chu kỳ = TTL đồng loạt" là phản xạ chẩn đoán đáng giá.

### 8. Những câu trả lời chưa đủ tốt
- "Cache miss thì đọc DB rồi set lại." → Đó là happy path ai cũng biết. 1000 miss đồng thời thì sao? Key không tồn tại thì sao?
- "Update DB xong update cache cho nhất quán." → Chính là phương án có race tệ nhất. Không phân tích thứ tự là chưa đủ Senior.

### 9. Sai lầm phổ biến của ứng viên
- Update cache thay vì delete sau ghi (race hai writer).
- TTL đồng loạt không jitter.
- Không biết cache penetration (key không tồn tại) khác cache miss thường.
- Cache object to đùng serialize cả đồ thị — băng thông Redis thành nút cổ chai trước cả CPU.
- Không có metric hit ratio per key-pattern — vận hành mù.

### 10. Follow-up Questions
- Invalidate cache khi dữ liệu bị sửa bởi hệ thống khác (batch job, DBA) — làm sao? (CDC.)
- Thiết kế cache cho trang sản phẩm 1M RPS: tầng nào, TTL bao nhiêu, invalidation thế nào?
- Bloom filter chống penetration: false positive ảnh hưởng gì? Cập nhật filter khi thêm key mới ra sao?
- Nếu Redis cluster sập toàn bộ, hệ thống sống không? (câu hỏi thật sự là: DB chịu được cold traffic không — graceful degradation, request coalescing, load shedding.)
- So sánh eventual consistency của cache-aside với read-your-writes yêu cầu của UX — dung hòa thế nào cho luồng "user sửa profile rồi xem lại"?

### 11. Liên hệ với Production
Facebook công bố bài "Scaling Memcache" — bible của chủ đề này: lease chống stampede + race, invalidation qua CDC (McSqueal), regional pool. Vấn đề nghiêm trọng khi: hit ratio cao đến mức DB không còn chịu nổi tải miss (cache trở thành thành phần **bắt buộc sống** — mất cache = mất hệ thống, phải quy hoạch DR cho cache như cho DB). Dấu hiệu cần hành động: outage có chu kỳ, DB load spike khi deploy (cold cache), hit ratio giảm dần theo tăng trưởng key space (eviction vì thiếu RAM).

---

# PHẦN C — CLICKHOUSE

## Câu 7 — [Senior] Tại sao ClickHouse nhanh? Columnar storage và MergeTree hoạt động thế nào?

### 1. Câu hỏi
"Điều gì làm ClickHouse nhanh hơn Postgres hàng trăm lần cho analytics? Trình bày MergeTree, và tại sao ClickHouse tệ cho OLTP?"

### 2. Interviewer muốn kiểm tra điều gì?
- Hiểu sự khác biệt bản chất row-store vs column-store — không phải "CH nhanh vì được viết bằng C++".
- Hiểu MergeTree: sparse index, part, background merge — mô hình LSM-flavored.
- Biết giới hạn: điểm yếu của CH chính là mặt trái của điểm mạnh.

### 3. Câu trả lời ngắn gọn (30 giây)
"Ba nguồn tốc độ: (1) **columnar** — query analytics chỉ đọc vài cột trong hàng trăm, row-store phải đọc cả row, CH chỉ đọc đúng cột cần → I/O giảm hàng chục lần; (2) **nén** — dữ liệu cùng cột đồng nhất và thường gần nhau về giá trị, nén 10–100x với codec chuyên biệt (Delta, DoubleDelta, LZ4/ZSTD) → đọc từ đĩa ít hơn nữa; (3) **vectorized execution** — xử lý theo batch cột, tận dụng SIMD và cache CPU. MergeTree lưu dữ liệu thành các **part** bất biến sắp xếp theo ORDER BY key với **sparse index** (mỗi 8192 row một mốc), background merge gộp part nhỏ thành lớn. Tệ cho OLTP vì: không có point-read rẻ, UPDATE/DELETE là mutation bất đồng bộ ghi lại cả part, insert từng row tạo bão part — mọi thứ tối ưu cho scan triệu row, không cho tìm một row."

### 4. Câu trả lời Senior Level (3–5 phút)
**Why columnar:** bảng 100 cột, query `SELECT country, count() GROUP BY country WHERE date > X`. Row-store: đọc toàn bộ row (100 cột) dù cần 2 → 98% I/O vứt đi. Column-store: mỗi cột một dãy file riêng, đọc đúng 2 cột. Cộng hưởng với nén: cột `country` toàn chuỗi lặp → dictionary + LZ4 nén còn vài %. Tổng I/O có thể ít hơn 100–1000 lần — **tốc độ đến từ việc không đọc, không phải đọc nhanh**.

**MergeTree:**
- Insert tạo **part** mới (thư mục bất biến, dữ liệu sorted theo ORDER BY key, mỗi cột file riêng + nén theo block).
- **Sparse primary index:** không index từng row; mỗi granule 8192 row lưu một mốc key trong RAM → bảng nghìn tỷ row index chỉ vài trăm MB; tìm kiếm = nhị phân trên mốc rồi scan granule. Rẻ vì scan tuần tự granule là sở trường.
- **Background merge** gộp part (giống LSM compaction) — từ đây sinh ra họ engine: ReplacingMergeTree (dedupe khi merge — **eventually**!), SummingMergeTree, AggregatingMergeTree, CollapsingMergeTree.
- **Partition** (thường theo tháng/ngày) để prune và drop rẻ; **ORDER BY key là quyết định thiết kế quan trọng nhất** — quyết định locality: `(tenant_id, timestamp)` cho query per-tenant.
- Skip index (minmax, bloom) hỗ trợ lọc cột ngoài key.

**Why not OLTP:** mutation (`ALTER ... UPDATE/DELETE`) ghi lại part bất đồng bộ — đắt và không tức thời; không transaction đa câu lệnh đúng nghĩa; point read phải đọc cả granule nhiều cột; insert lắt nhắt tạo nghìn part nhỏ → "too many parts" — vì thế phải batch insert (hoặc async insert). CH là máy scan, không phải máy seek.

**Production:** pattern chuẩn — Kafka → batch insert CH (materialized view tính sẵn aggregate), dedupe bằng ReplacingMergeTree + `FINAL` chỉ khi bắt buộc (đắt), TTL tự xóa dữ liệu cũ, OLTP giữ ở Postgres và CDC sang.

### 5. Giải thích bản chất
Toàn bộ câu chuyện là **định lý về locality**: bạn chỉ đọc nhanh thứ nằm cạnh nhau theo đúng chiều bạn hỏi. OLTP hỏi theo chiều **row** ("mọi thứ về đơn hàng #123") → row-store đúng. Analytics hỏi theo chiều **cột** ("tổng amount của 1 tỷ đơn") → column-store đúng. Một cách sắp xếp vật lý không thể tối ưu cả hai chiều — đây không phải hạn chế kỹ thuật mà là hình học của dữ liệu. Mọi thứ khác (nén tốt — vì dữ liệu cùng kiểu; vectorization — vì cùng phép toán trên dãy dài; sparse index — vì sorted) đều là **hệ quả dây chuyền** của quyết định xếp theo cột + sort. Hiểu chuỗi nhân quả này giá trị hơn thuộc lòng benchmark.

### 6. Trade-off
- **Scan throughput ↔ point access:** đọc tỷ row/giây ↔ tìm một row đắt vô lý so với B-tree.
- **Part bất biến:** insert nhanh, nén sâu, backup dễ ↔ update/delete thành mutation nặng; dedupe là eventually.
- **Sparse index:** RAM tí hon cho bảng khổng lồ ↔ selectivity kém cho lookup kim-trong-đống-rơm.
- **Batch insert:** throughput khủng ↔ latency dữ liệu-tới-query (giây tới phút); real-time tuyệt đối không phải sở trường.
- **Không chuẩn SQL transaction:** đơn giản engine ↔ đẩy trách nhiệm exactly-once về pipeline ingest.

### 7. Ví dụ Production
Migration tôi từng chủ trì: bảng events 8 tỷ row trên Postgres (partition + BRIN đã vắt kiệt) — dashboard query 40–120s, chiếm 70% I/O cluster, ảnh hưởng OLTP. Chuyển sang CH: ORDER BY `(tenant_id, event_date, event_type)`, partition theo tháng, materialized view cho 5 dashboard nặng nhất → cùng query 200–800ms trên máy rẻ hơn, nén từ 4TB còn 180GB. Chi phí thật phải kể: pipeline Kafka batch + dedupe (at-least-once → ReplacingMergeTree), đội phải học khái niệm part/merge/mutation, và 2 tuần đau đầu vì "too many parts" do một service insert từng row. Bài học: CH không thay Postgres — nó **gỡ tải analytics ra khỏi** Postgres.

### 8. Những câu trả lời chưa đủ tốt
- "ClickHouse nhanh vì columnar." → Mới 1/3. Nén và vectorization đâu? Và **tại sao** columnar giúp — nói được chữ I/O chưa?
- "Dùng CH thay Postgres cho nhanh." → Cho workload nào? Point read và update thì sao? Câu này lộ chưa hiểu trade-off.

### 9. Sai lầm phổ biến của ứng viên
- Nghĩ CH là "Postgres nhanh hơn" thay vì một máy khác cho bài toán khác.
- Insert từng row như OLTP → too many parts.
- Không hiểu ReplacingMergeTree dedupe **khi merge** (không đảm bảo lúc query) — tưởng có unique constraint.
- Chọn ORDER BY key tùy tiện (hoặc để timestamp đứng đầu khi query luôn lọc tenant).
- Lạm dụng `FINAL` hoặc mutation thường xuyên — dùng CH ngược sở trường.

### 10. Follow-up Questions
- Thiết kế ORDER BY và partition cho hệ log 100TB, query chủ yếu theo (service, time range)?
- Materialized view của CH khác materialized view Postgres thế nào? (incremental, trigger theo insert — và các bẫy: chỉ áp cho dữ liệu mới.)
- Exactly-once từ Kafka vào CH làm thế nào? (block dedup, exactly-once của Kafka engine, hay idempotent theo khóa + Replacing.)
- Distributed table hoạt động ra sao? Query xuyên shard merge ở đâu, sharding key chọn thế nào?
- Khi nào chọn CH vs BigQuery/Snowflake vs Druid/Pinot? (tự vận hành vs serverless; latency vs chi phí; real-time ingest.)

### 11. Liên hệ với Production
Cloudflare (HTTP analytics), Uber (log), nhiều sàn TMĐT Việt Nam dùng CH cho analytics thời gian cận thực. Vấn đề nghiêm trọng khi: analytics query bắt đầu bóp chết OLTP DB (dấu hiệu cần tách), hoặc ngược lại — khi team dùng CH như OLTP (update nhiều, đọc point) và thất vọng. Dấu hiệu vận hành cần chú ý: số part per partition tăng (merge không kịp insert), mutation queue dài, query đọc quá nhiều granule do ORDER BY key không khớp filter phổ biến.
