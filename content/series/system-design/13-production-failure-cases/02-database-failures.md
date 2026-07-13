+++
title = "13.2. Database Failures"
date = "2026-07-13T16:30:00+07:00"
draft = false
tags = ["backend", "system-design"]
series = ["System Design — Tư Duy Thiết Kế Hệ Thống"]
+++

> Sáu tình huống: Database Hotspot, N+1 Query, Deadlock, Replica Lag, Connection Pool Exhaustion, Hot Partition. Database là nơi bottleneck ghé thăm thường xuyên nhất ([chương 1.5](/series/system-design/01-foundations/05-bottleneck-analysis/)) — và cũng là nơi sự cố khó scale-out thoát thân nhất.

---

## Case 4 — Database Hotspot

### Triệu chứng

DB tổng thể "khỏe" (CPU trung bình 40%) nhưng một nhóm thao tác cụ thể chậm bất thường; lock wait tăng; các query chạm vào *một hàng/một trang* cụ thể xếp hàng dài.

### Root cause & tại sao xảy ra

Tải không phân bố đều — dồn vào một điểm dữ liệu: **một hàng** được update liên tục (số dư ví của shop lớn, counter tổng, tồn kho của 1 sản phẩm flash sale), hoặc **một trang index** nhận mọi insert (primary key tự tăng/timestamp → mọi insert chen vào trang cuối cùng của B-tree). Ghi vào một hàng phải tuần tự hóa qua row lock — đây là "phần tuần tự" trong Amdahl's Law: thêm CPU, thêm replica đều vô ích, vì trần là tốc độ xử lý lock của **một** hàng.

### Kiến trúc bị ảnh hưởng

Mọi RDBMS; các schema có counter tập trung, ví/số dư tập trung, bảng ghi sự kiện với key đơn điệu tăng.

### Metric / Dashboard / Alert / Điều tra

- Metric: lock wait time (`pg_locks` + `pg_stat_activity` với `wait_event_type=Lock`), thời gian giữ transaction, QPS phân theo giá trị khóa (nếu log được), buffer contention.
- Dashboard: top hàng/bảng theo lock wait; phân bố update theo key.
- Alert: lock wait p99 vượt ngưỡng; số session chờ lock > N.
- Điều tra: từ `pg_stat_activity` lấy các query đang chờ → xem chúng chờ lock của hàng nào → truy nghiệp vụ tương ứng. Nếu mọi mũi tên chỉ về một `id` — hotspot xác nhận.

### Khắc phục & phòng tránh

- **Cầm máu:** giảm tần suất ghi vào điểm nóng (bật queue phía trước, gom batch), hoặc tạm chuyển counter sang Redis.
- **Chữa gốc — tản điểm nóng:**
  - **Sharded counter:** 1 counter → N hàng con (`counter_0..counter_15`, update ngẫu nhiên một hàng, đọc thì SUM) — trần ghi ×N.
  - **Gom ghi (write batching):** cộng dồn trong memory/Redis, flush xuống DB mỗi X ms — đổi độ tươi lấy throughput.
  - **Đổi mô hình:** thay "update số dư" bằng "insert bút toán" (append-only ledger) — insert không tranh lock với nhau; số dư là tổng suy diễn. Đây là lý do sâu xa ngành tài chính dùng ledger.
  - Với index hotspot: key ngẫu nhiên (UUIDv7 cân bằng giữa ngẫu nhiên và locality), hoặc partition bảng.
- **Phòng:** design review bắt buộc hỏi "thực thể nào sẽ nhận ghi tập trung?" — hotspot nhìn thấy được từ trên giấy, trước khi viết dòng code nào.

---

## Case 5 — N+1 Query

### Triệu chứng

Endpoint chậm tuyến tính theo kích thước danh sách trả về; DB QPS cao bất thường so với lượng request; slow log *không có* query nào chậm — chỉ có **rất nhiều** query rất nhanh, giống nhau, khác mỗi tham số id.

### Root cause & tại sao xảy ra

Code lấy danh sách N bản ghi (1 query) rồi lặp từng bản ghi lấy dữ liệu liên quan (N query) — thường do ORM lazy-loading che mất ranh giới giữa "truy cập thuộc tính" và "round-trip database". Trang 50 đơn hàng × (khách + sản phẩm + vận chuyển) = 151 query. Mỗi query nhanh (1ms) nhưng 151 round-trip = 151ms chỉ riêng network, cộng chi phí parse/plan ×151. Độc ở chỗ: dev không *thấy* mình viết 151 query — ORM viết hộ.

### Kiến trúc bị ảnh hưởng

Mọi app dùng ORM (ActiveRecord, Hibernate, Django ORM, Prisma...); GraphQL server naive (mỗi resolver một query — N+1 dạng phân tán); microservices gọi API trong vòng lặp (N+1 qua network — đắt gấp 10).

### Metric / Dashboard / Alert / Điều tra

- Metric: **số query trên mỗi request** (middleware đếm — metric rẻ mà giá trị nhất để bắt N+1), DB QPS / app RPS ratio.
- Dashboard: top endpoint theo queries-per-request.
- Alert: endpoint có queries-per-request > 20 (ngưỡng tùy app).
- Điều tra: bật query log cho một request mẫu → đếm và nhóm theo fingerprint → thấy ngay chuỗi `SELECT ... WHERE id = ?` lặp N lần.

### Khắc phục & phòng tránh

- **Chữa:** eager loading (`includes`/`select_related`/`JOIN FETCH`), batch loading (`WHERE id IN (...)`), GraphQL dùng DataLoader (gom các lookup trong một tick thành một batch query). Với N+1 qua network: bulk API endpoint.
- **Phòng:** middleware cảnh báo queries-per-request ngay ở môi trường dev (`bullet` cho Rails, `nplusone` cho Python); test hiệu năng với **dữ liệu cỡ thật** — N+1 vô hình với bảng 10 hàng ở máy dev, đó là lý do nó sống sót đến production.

---

## Case 6 — Deadlock

### Triệu chứng

Lỗi rải rác `deadlock detected` / `Lock wait timeout exceeded`; một tỷ lệ nhỏ transaction fail và (nếu có retry) tự thành công lần hai; tần suất tăng theo tải và theo *độ trùng lặp* của dữ liệu được chạm (cùng bán một mã hàng hot).

### Root cause & tại sao xảy ra

Hai transaction giữ lock và chờ lock của nhau theo vòng tròn: T1 khóa hàng A chờ B; T2 khóa B chờ A. DB phát hiện chu trình và **giết một bên** (đây là hành vi đúng, không phải bug của DB). Xảy ra vì hai luồng nghiệp vụ khóa *cùng tập hàng theo thứ tự khác nhau* — ví dụ kinh điển: chuyển tiền A→B song song với B→A; hoặc bulk update sắp theo thứ tự khác nhau. Biến thể tinh vi: lock ngầm từ foreign key, unique index, hoặc `SELECT ... FOR UPDATE` phạm vi rộng.

### Kiến trúc bị ảnh hưởng

Mọi RDBMS dùng transaction đa câu lệnh. Nặng hơn khi transaction dài (giữ lock lâu → cửa sổ giao nhau rộng) và khi có "hàng nóng" nhiều luồng cùng chạm.

### Metric / Dashboard / Alert / Điều tra

- Metric: deadlocks/phút (`pg_stat_database.deadlocks`), lock wait time, transaction duration p99.
- Alert: deadlock rate tăng đột biến (nền vài cái/ngày có thể chấp nhận; hàng chục/phút là sự cố).
- Điều tra: log của DB in ra **cả hai query** dính chu trình — đọc nó (đừng đoán). Truy hai luồng nghiệp vụ tương ứng, vẽ thứ tự khóa của từng luồng → tìm điểm ngược chiều.

### Khắc phục & phòng tránh

- **Cầm máu:** retry tự động cho transaction bị chọn làm nạn nhân (deadlock là lỗi *tạm thời* chuẩn — luôn đáng retry, với backoff + giới hạn).
- **Chữa gốc:**
  - **Khóa theo thứ tự toàn cục nhất quán:** mọi luồng chạm nhiều hàng phải khóa theo cùng thứ tự (sort theo id trước khi update; chuyển tiền: luôn khóa account id nhỏ trước). Đây là thuốc đặc trị — deadlock không thể tồn tại nếu không có chu trình.
  - **Transaction ngắn:** không gọi API ngoài / không chờ user trong transaction; khóa muộn nhất có thể.
  - Giảm phạm vi lock: update có điều kiện một câu (`UPDATE ... WHERE stock >= 1`) thay vì SELECT FOR UPDATE + kiểm tra + UPDATE.
- **Phòng:** quy ước thứ tự khóa thành văn trong team; test tải song song trên các luồng chạm chung dữ liệu.

---

## Case 7 — Replica Lag

### Triệu chứng

User sửa xong không thấy thay đổi (rồi F5 lại thấy); dữ liệu "nhảy qua nhảy lại" giữa các lần đọc; job đọc replica xử lý dữ liệu cũ. Về phía hệ thống: `replication lag` tăng — đôi khi tăng *đơn điệu không hồi*.

### Root cause & tại sao xảy ra

Replication async: replica áp dụng log của primary trễ một khoảng ([chương 4.2](/series/system-design/04-distributed-systems/02-replication-consistency/)). Lag phình khi: (1) ghi ở primary bùng nổ (migration, batch import) vượt tốc độ apply của replica; (2) replica yếu hơn primary hoặc bận phục vụ query nặng (query dài chặn apply — PostgreSQL còn có xung đột vacuum/query làm dừng apply); (3) network nghẽn. Lag tăng không hồi = tốc độ apply < tốc độ ghi kéo dài — replica sẽ không bao giờ đuổi kịp nếu không can thiệp.

### Kiến trúc bị ảnh hưởng

Mọi kiến trúc read-replica ([giai đoạn 2+, Phần 12](/series/system-design/12-evolution/02-them-redis/)); CQRS projection lag là dạng tổng quát của cùng bệnh ([giai đoạn 8](/series/system-design/12-evolution/08-cqrs/)).

### Metric / Dashboard / Alert / Điều tra

- Metric: lag theo **giây** (`pg_last_xact_replay_timestamp`) *và* theo **byte** (WAL chưa apply); write throughput primary; long-running query trên replica.
- Alert: hai tầng — lag > ngưỡng ứng dụng chịu được (vd 10s): cảnh báo; lag tăng đơn điệu 15 phút: khẩn.
- Điều tra: lag tăng từ lúc nào → trùng sự kiện gì (deploy, migration, báo cáo nặng chạy trên replica)? Byte-lag tăng nhưng giây-lag đứng im = replica đang nghẽn apply (tìm query dài trên replica).

### Khắc phục & phòng tránh

- **Cầm máu:** giết query dài trên replica; tạm route đọc quan trọng về primary; hoãn batch ghi.
- **Chữa gốc:** tách replica theo mục đích (replica phục vụ app ≠ replica analytics); throttle batch ghi ở primary; nâng cấp replica ngang primary; với PostgreSQL — cấu hình `hot_standby_feedback`/`max_standby_streaming_delay` có ý thức về trade-off của chúng.
- **Phòng (quan trọng nhất — ở tầng ứng dụng):** chấp nhận lag *là thuộc tính vĩnh viễn* và thiết kế quanh nó: read-your-writes routing (đọc primary trong X giây sau ghi của chính user đó), lag-aware routing (loại replica lag cao khỏi pool), và **cấm đọc replica cho mọi quyết định ghi**.

---

## Case 8 — Connection Pool Exhaustion

### Triệu chứng

Latency tăng vọt hoặc lỗi `timeout acquiring connection` / `too many connections` — trong khi **CPU của cả app lẫn DB đều thấp**. Đây là chữ ký đặc trưng: hệ thống "rảnh" mà request xếp hàng. Thường lan nhanh: một endpoint chậm kéo mọi endpoint khác cùng chết.

### Root cause & tại sao xảy ra

Connection là tài nguyên đếm được. Theo Little's Law: connections cần = QPS × thời gian giữ connection. Một query đột nhiên chậm (thiếu index sau khi dữ liệu lớn, lock, replica down làm dồn tải) → thời gian giữ ×10 → nhu cầu connection ×10 > pool size → mọi request khác (kể cả nhanh) chờ connection → thread web bị giam theo → **cạn lan tầng**. Nguyên nhân phụ trợ kinh điển: transaction ôm gọi API ngoài (giữ connection suốt 5s timeout của bên thứ ba); connection leak (thiếu `finally`/`defer` trả connection); quá nhiều instance app × pool mỗi instance > `max_connections` của DB.

### Kiến trúc bị ảnh hưởng

Mọi app có pool (tức là mọi app). Microservices tệ hơn: chuỗi pool nối tiếp (HTTP pool → service → DB pool) — cạn một tầng giam tầng trước, khuếch đại thành [cascading failure](/series/system-design/13-production-failure-cases/04-distributed-failures/).

### Metric / Dashboard / Alert / Điều tra

- Metric: **pool wait time** (quý giá nhất), active/idle/max của pool, connection acquire timeout count, thời gian giữ connection p99, DB `max_connections` headroom.
- Dashboard: pool utilization từng service + wait time, cạnh biểu đồ latency query.
- Alert: pool wait p99 > vài chục ms; active = max kéo dài > 1 phút.
- Điều tra: pool nào cạn trước (nhìn theo chuỗi phụ thuộc)? → tại sao thời gian giữ tăng (slow query mới? lock? API ngoài chậm?) → "cái gì vừa thay đổi?"

### Khắc phục & phòng tránh

- **Cầm máu:** giết các query/transaction treo lâu ở DB; restart instance bị leak; **đừng phản xạ tăng pool size** — nếu gốc là DB chậm, pool to hơn = đổ thêm tải vào DB đang ngộp.
- **Chữa gốc:** fix query chậm; đưa API ngoài ra khỏi transaction; timeout ở *mọi* tầng (acquire timeout, query timeout, transaction timeout — nguyên tắc: tầng ngoài dài hơn tầng trong); pool size tính từ Little's Law chứ không copy số của người khác; PostgreSQL nhiều instance → PgBouncer.
- **Phòng:** bulkhead — pool riêng cho endpoint nặng/nhẹ để export báo cáo không giam checkout; leak detection của pool (HikariCP `leakDetectionThreshold`); load test có kịch bản "một dependency chậm đi 10×".

---

## Case 9 — Hot Partition

### Triệu chứng

Trong hệ đã shard/partition (Kafka, Cassandra, DynamoDB, Citus...): **một** shard/partition/node quá tải trong khi anh em nó rảnh; throttling error (`ProvisionedThroughputExceeded`) dù tổng capacity dư; consumer của một partition Kafka lag trong khi các partition khác sạch.

### Root cause & tại sao xảy ra

Sharding đứng trên giả định **tải phân bố đều theo partition key** — và thực tế phá giả định đó theo hai cách: (1) *key skew tự nhiên*: dữ liệu theo luật lũy thừa — shop lớn nhất bằng 10.000 shop nhỏ, celebrity có 10M follower, ngày hôm nay nóng hơn mọi ngày (time-based key là hot partition có lịch hẹn); (2) *key skew thiết kế*: chọn key ít cardinality (status, country) hoặc key đơn điệu (timestamp). Hệ quả giống Database Hotspot nhưng ở tầng hạ tầng phân tán: trần của hệ = trần của partition nóng nhất, không phải tổng các partition — tiền mua N node, dùng được 1.

### Kiến trúc bị ảnh hưởng

Mọi hệ partition theo key: Kafka (partition by key), Cassandra/DynamoDB (partition key), sharded SQL, Elasticsearch (routing), Redis Cluster (hash slot).

### Metric / Dashboard / Alert / Điều tra

- Metric: **phân bố tải theo partition** (QPS/bytes/lag per partition — không chỉ tổng!), heat map partition, độ lệch max/median.
- Dashboard: heat map partition theo thời gian — hot partition hiện thành một vệt sáng.
- Alert: partition max > 3–5× median kéo dài.
- Điều tra: partition nào nóng → key nào chiếm tải trong partition đó (sampling) → key đó là ai (thường ra ngay: một tenant lớn, một sản phẩm viral, một bot).

### Khắc phục & phòng tránh

- **Cầm máu:** rate limit riêng key nóng; với Kafka — tạm tách key nóng ra topic riêng có nhiều consumer; với DynamoDB — bật adaptive capacity (có sẵn) + cache trước key nóng.
- **Chữa gốc:**
  - **Key composite / salting:** `celebrity_id` → `celebrity_id#random(0..9)` — tản 1 key ra 10 partition, đọc thì gom 10 (đổi chi phí đọc lấy trần ghi).
  - **Xử lý riêng cho "VIP key":** chấp nhận rằng phân bố lũy thừa nghĩa là top 0.1% key xứng đáng một đường đi riêng (cache riêng, shard riêng, pipeline riêng) — đừng cố nhét voi và kiến vào cùng một khuôn.
  - Chọn lại partition key: cardinality cao + phân bố đều + khớp access pattern ([Phần 8](/series/system-design/08-data-partitioning/00-tong-quan/)).
- **Phòng:** phân tích phân bố key **trước khi** chọn key (top-K key chiếm bao nhiêu % tải?); mô phỏng với dữ liệu thật; nhớ quy luật: *mọi hệ thống thành công rồi sẽ có key nóng* — thiết kế sẵn đường thoát.

---

## Tổng kết nhóm database

| Case | Chữ ký nhận diện nhanh | Thuốc đặc trị |
|---|---|---|
| Hotspot | DB khỏe tổng thể, một hàng xếp hàng lock | Tản điểm ghi: sharded counter, ledger, batch |
| N+1 | Nhiều query nhanh giống nhau, chậm theo cỡ danh sách | Eager/batch loading; đếm queries-per-request |
| Deadlock | Lỗi `deadlock detected` rải rác, tăng theo tải | Khóa theo thứ tự nhất quán; transaction ngắn; retry |
| Replica Lag | User "không thấy" thay đổi của chính mình | Read-your-writes routing; tách replica theo mục đích |
| Pool Exhaustion | Chậm/lỗi trong khi CPU mọi nơi đều thấp | Timeout mọi tầng; fix thời-gian-giữ; bulkhead |
| Hot Partition | Một shard cháy, anh em rảnh; max ≫ median | Salting, đường riêng cho VIP key, chọn lại key |

Bài học chung: cả sáu case đều là **giả định bị thực tế phá vỡ** — giả định tải đều, giả định ORM vô hại, giả định lock tự do, giả định replica tức thời, giả định connection vô hạn, giả định key phân bố đều. Thiết kế trưởng thành là thiết kế viết rõ giả định của mình ra và đặt metric canh chừng ngày chúng gãy.
