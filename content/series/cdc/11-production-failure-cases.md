+++
title = "Chương 11: Production Failure Cases"
date = "2026-02-20T18:00:00+07:00"
draft = false
tags = ["backend", "cdc", "kafka", "database"]
series = ["Change Data Capture"]
+++

Đây là chương quan trọng nhất của toàn bộ tài liệu. Lý do rất đơn giản: **CDC không khó ở lúc setup — CDC khó ở tháng thứ ba vận hành.** Demo Debezium chạy trong một buổi chiều; nhưng pipeline CDC production là một chuỗi hệ thống stateful nối tiếp nhau (database → connector → Kafka → consumer → sink), và mỗi mắt xích có những chế độ hỏng riêng, đôi khi âm thầm, đôi khi phá hủy cả database nguồn.

Tôi tổng hợp ở đây 20 failure case mà bản thân đã trực tiếp xử lý hoặc điều tra hậu quả trong hơn một thập kỷ vận hành CDC. Mỗi case theo cùng một cấu trúc: **Triệu chứng → Root cause → Cơ chế kỹ thuật → Cách điều tra → Metric cần theo dõi → Alert nên cấu hình → Cách khắc phục → Cách phòng tránh.** Các con số (threshold, dung lượng, thời gian) là số liệu minh họa điển hình từ kinh nghiệm — bạn phải hiệu chỉnh theo hệ thống của mình.

Một nguyên tắc đọc chương này: đừng hỏi "liệu sự cố này có xảy ra với mình không". Hãy hỏi "khi nó xảy ra, mình sẽ phát hiện sau bao lâu". Hầu hết thảm họa CDC không phải do sự cố hiếm, mà do sự cố *phổ biến* không được phát hiện trong nhiều ngày.

## Nhóm A: Sự cố phía database nguồn

### Case 1: Replication slot giữ WAL, disk database tăng hàng trăm GB — case kinh điển nhất

Nếu chỉ được dạy đội vận hành một case duy nhất, tôi chọn case này, vì nó là case duy nhất trong đó **hệ thống phụ trợ (CDC) giết chết hệ thống chính (OLTP database)**.

**Triệu chứng.** Disk usage của PostgreSQL primary tăng đều đặn không rõ lý do — vài GB mỗi giờ. Thư mục `pg_wal` chiếm hàng trăm GB thay vì vài GB như bình thường. Nếu không ai can thiệp: disk đầy 100%, PostgreSQL PANIC, **database ngừng nhận ghi hoàn toàn** — toàn bộ hệ thống nghiệp vụ sập, không chỉ pipeline CDC.

Kịch bản thực tế tôi từng điều tra: connector Debezium fail vào tối thứ Sáu (do một DDL không tương thích — xem Case 11), không ai cấu hình alert cho task status. Ba ngày cuối tuần, database vẫn ghi bình thường, WAL tích lũy ~600 GB. Sáng thứ Hai disk còn 4%, on-call nhận alert disk — và ngay lúc đó mới biết đến sự tồn tại của connector đã chết 3 ngày. May mắn kịp xử lý trước khi PANIC. Không phải team nào cũng may mắn như vậy.

**Root cause.** Replication slot là một *lời hứa* của PostgreSQL: "tôi sẽ giữ mọi WAL segment mà consumer của slot này chưa xác nhận đã xử lý". Khi Debezium chết (hoặc treo, hoặc mất mạng — xem Case 18), slot vẫn tồn tại, `confirmed_flush_lsn` đứng yên, và PostgreSQL trung thành giữ WAL mãi mãi. Slot không có timeout mặc định.

**Cơ chế kỹ thuật.** Checkpoint của PostgreSQL bình thường sẽ recycle/xóa WAL segment cũ. Nhưng thuật toán xóa lấy `min(restart_lsn của mọi slot, vị trí các standby, wal_keep_size)` làm ranh giới — segment nào còn cần cho bất kỳ slot nào đều được giữ. Một slot đứng yên kéo ranh giới này đứng yên theo, và tốc độ phình của `pg_wal` đúng bằng tốc độ ghi WAL của toàn database (mọi bảng, kể cả bảng không nằm trong publication — WAL là của cả instance).

**Cách điều tra.**

```sql
-- Slot nào đang tồn tại, active hay không, giữ lại bao nhiêu WAL?
SELECT slot_name, active, restart_lsn, confirmed_flush_lsn,
       pg_size_pretty(pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)) AS retained_wal,
       wal_status, safe_wal_size          -- PG 13+
FROM pg_replication_slots;

-- Dung lượng pg_wal hiện tại
SELECT pg_size_pretty(sum(size)) FROM pg_ls_waldir();
```

Đọc kết quả: `active = false` nghĩa là không có consumer nào đang kết nối slot — connector chết hoặc mất mạng. `retained_wal` là lượng WAL đang bị giữ. `wal_status = 'extended'` là cảnh báo; `'unreserved'`/`'lost'` (khi có `max_slot_wal_keep_size`) nghĩa là slot đã hoặc sắp bị vô hiệu. Tiếp theo kiểm tra phía connector: `GET /connectors/<name>/status` trên Kafka Connect REST API để xem task `FAILED` từ bao giờ và stack trace gì.

**Metric cần theo dõi.**
- `pg_wal_lsn_diff(pg_current_wal_lsn(), restart_lsn)` cho từng slot (postgres_exporter có sẵn) — đây là metric quan trọng nhất của toàn bộ pipeline CDC trên PostgreSQL.
- `active` flag của từng slot.
- Disk free của volume chứa `pg_wal`.

**Alert nên cấu hình.**
- WARNING: slot retained WAL > 10 GB trong 15 phút (minh họa — đặt theo tốc độ ghi và dung lượng disk của bạn).
- CRITICAL: slot retained WAL > 50 GB, hoặc slot `active = false` quá 10 phút, hoặc disk free của `pg_wal` < 20%.

**Cách khắc phục.** Theo thứ tự ưu tiên: (1) Sửa connector để nó chạy lại và bắt kịp — slot sẽ tự giải phóng WAL khi `confirmed_flush_lsn` tiến lên. (2) Nếu connector không thể cứu nhanh và disk sắp đầy: mở rộng disk (cloud cho phép online resize) để mua thời gian. (3) Phương án cuối cùng: `SELECT pg_drop_replication_slot('slot_name');` — WAL giải phóng ngay sau checkpoint, **nhưng bạn chấp nhận mất vị trí đọc: pipeline phải re-snapshot** (xem Case 19 về quy trình). Đừng bao giờ xóa file trong `pg_wal` bằng tay — đó là con đường ngắn nhất đến corrupt database.

**Cách phòng tránh.** (1) PostgreSQL 13+: đặt `max_slot_wal_keep_size = 50GB` (minh họa) — đây là cầu chì: khi vượt ngưỡng, PostgreSQL vô hiệu slot thay vì tự sát. Đánh đổi rõ ràng: bạn đổi "database chết" lấy "pipeline CDC phải re-snapshot" — với tôi đây là đánh đổi đúng trong 100% trường hợp. (2) Alert trên slot lag và slot inactive **từ ngày đầu tiên** deploy connector, trước cả khi có traffic thật. (3) Runbook rõ ràng: on-call nhìn thấy alert này phải biết làm gì trong 5 phút. (4) Quy trình: mọi connector bị pause/delete dài ngày phải kèm việc drop slot tương ứng.

Ghi chú cho MySQL: bài toán tương đương là binlog retention — MySQL *không* giữ binlog theo consumer, nên rủi ro đảo chiều: không phải disk đầy, mà là binlog bị purge khi connector chưa đọc (xem Case 8).

### Case 2: Slot lag tăng trên database ít ghi — thiếu heartbeat

**Triệu chứng.** Database (hoặc bảng được capture) gần như không có ghi, nhưng retained WAL của slot vẫn tăng dần; hoặc lag "ảo" hàng giờ trên dashboard dù pipeline khỏe.

**Root cause.** `confirmed_flush_lsn` chỉ tiến khi Debezium *nhận và xác nhận* event. Nếu các bảng trong `publication` không có thay đổi nào, Debezium không nhận gì để xác nhận — trong khi các bảng *khác* trên cùng instance (hoặc database khác trên cùng cluster) vẫn sinh WAL đều đặn. Slot đứng yên, WAL bị giữ. Đây là biến thể "âm thầm" của Case 1, đặc biệt nguy hiểm trên shared PostgreSQL cluster nơi một database busy và một database yên tĩnh dùng chung WAL.

**Cơ chế kỹ thuật.** WAL là một stream duy nhất cho cả instance. Slot trỏ vào một vị trí trong stream đó. Logical decoding chỉ gửi cho Debezium những thay đổi thuộc publication — những đoạn WAL "không liên quan" vẫn phải được slot đi qua và xác nhận, nhưng không có event nào làm mồi cho việc xác nhận.

**Cách điều tra.** Query `pg_replication_slots` như Case 1: slot `active = true` nhưng `confirmed_flush_lsn` không đổi qua nhiều lần query, trong khi `pg_current_wal_lsn()` tiến đều.

**Metric / Alert.** Chính là slot lag của Case 1 — case này là lý do alert của Case 1 sẽ kêu dù "không có gì hỏng".

**Cách khắc phục & phòng tránh.** Cấu hình heartbeat của Debezium: `heartbeat.interval.ms = 10000` và quan trọng hơn với PostgreSQL là `heartbeat.action.query = 'INSERT INTO debezium_heartbeat (id, ts) VALUES (1, now()) ON CONFLICT (id) DO UPDATE SET ts = now()'` — một bảng heartbeat nằm *trong publication*. Mỗi 10 giây có một thay đổi thật chảy qua slot, `confirmed_flush_lsn` tiến đều, WAL được giải phóng. Đây là cấu hình tôi coi là **bắt buộc** cho mọi Postgres connector, không phải tùy chọn.

### Case 3: Snapshot kéo dài nhiều giờ và database CPU tăng mạnh

**Triệu chứng.** Connector mới deploy (hoặc buộc re-snapshot) trên bảng vài trăm triệu tới vài tỷ rows: snapshot chạy 6–20 giờ (minh họa). Trong thời gian đó: CPU database tăng 30–60%, I/O đọc bão hòa, query nghiệp vụ chậm; đồng thời **thay đổi realtime bị dồn lại** — vì streaming chỉ bắt đầu sau khi snapshot xong — nên khi snapshot kết thúc, pipeline lại mất thêm nhiều giờ tiêu backlog.

**Root cause.** Snapshot mặc định là `SELECT *` tuần tự từng bảng, single-thread per table (với đa số connector), đọc toàn bộ dữ liệu qua một connection. Bảng 2 tỷ rows × 500 bytes = ~1 TB dữ liệu phải kéo qua network, decode, produce lên Kafka.

**Cơ chế kỹ thuật.** Hai chi phí chồng lên nhau: (a) full table scan đẩy buffer cache của database ra khỏi bộ nhớ, làm mọi query khác chậm theo (cache pollution); (b) với PostgreSQL, snapshot truyền thống giữ một transaction dài (repeatable read) — transaction dài ngăn vacuum dọn dead tuple, gây bloat trên các bảng busy khác. Kafka producer phía connector cũng có thể nghẽn, làm snapshot chậm hơn nữa.

**Cách điều tra.**

```sql
-- Transaction snapshot đang chạy bao lâu, đọc đến đâu?
SELECT pid, state, xact_start, query
FROM pg_stat_activity
WHERE usename = 'debezium' ORDER BY xact_start;
```

Phía connector: log Debezium in tiến độ snapshot (`Exported N of M rows for table ...`); JMX metric `RowsScanned`, `SnapshotRunning`, `SnapshotDurationInSeconds`.

**Metric.** `SnapshotRunning`, `RemainingTableCount`, CPU/IO của database, độ dài transaction lâu nhất (`max(now() - xact_start)`).

**Alert.** Snapshot chạy quá X giờ so với dự kiến; transaction từ user debezium mở quá 1 giờ (nếu bạn không chủ đích snapshot).

**Cách khắc phục & phòng tránh.**
1. **Incremental snapshot** (signal-based, thuật toán watermark DBLog): snapshot theo chunk (`incremental.snapshot.chunk.size`, ví dụ 10k rows), chạy *song song* với streaming, có thể pause/resume, không giữ transaction dài. Đây là lựa chọn mặc định của tôi cho mọi bảng lớn từ Debezium 1.6+. Kích hoạt bằng signal table hoặc Kafka signal topic:
```sql
INSERT INTO debezium_signal (id, type, data)
VALUES ('sig-1', 'execute-snapshot',
        '{"data-collections": ["public.orders"], "type": "incremental"}');
```
2. **Snapshot từ read replica** nếu topology cho phép, để primary không chịu tải đọc (lưu ý điểm chuyển tiếp snapshot→streaming cần cẩn thận về consistency).
3. **Chạy off-peak** và báo trước cho team DBA — snapshot là một batch job nặng, hãy đối xử với nó như vậy.
4. **Throttling**: giảm `max.batch.size`, `snapshot.fetch.size`; chấp nhận snapshot lâu hơn để nghiệp vụ không bị ảnh hưởng.
5. Lọc từ đầu: `table.include.list` chỉ những bảng thật sự cần — tôi từng thấy connector snapshot 400 bảng trong khi downstream chỉ dùng 12.

## Nhóm B: Sự cố phía connector / Kafka Connect

### Case 4: Connector restart liên tục do OOM

**Triệu chứng.** Task chuyển `RUNNING → FAILED → RUNNING` theo chu kỳ vài phút; log Connect worker có `java.lang.OutOfMemoryError` hoặc container bị OOMKilled (exit code 137 trên Kubernetes). Lag tăng dần vì connector chết trước khi kịp bắt kịp — vòng xoáy: càng lag, backlog càng lớn, batch càng nặng, càng dễ OOM.

**Root cause phổ biến.** (a) Transaction lớn phía nguồn — một batch UPDATE 5 triệu rows tạo ra lượng event khổng lồ phải buffer; (b) `max.queue.size` × kích thước event vượt heap; (c) snapshot bảng có cột TEXT/JSON/BLOB lớn; (d) heap worker chia cho quá nhiều connector cùng lúc.

**Cơ chế kỹ thuật.** Debezium giữ hàng đợi nội bộ giữa thread đọc log và thread produce (`max.queue.size`, mặc định 8192 event). Với event 100 KB (row nhiều cột JSON), riêng queue có thể chiếm ~800 MB. PostgreSQL logical decoding còn có phần buffer phía server (`logical_decoding_work_mem`) — transaction lớn hơn ngưỡng này bị spill ra disk phía server, nhưng phía connector vẫn phải nhận trọn transaction theo commit.

**Cách điều tra.** `kubectl describe pod` xem OOMKilled; heap dump nếu là OutOfMemoryError trong JVM; log Debezium tìm transaction lớn ("processing N events"); đối chiếu thời điểm crash với batch job phía nguồn (thường trùng giờ chạy job đêm).

**Metric.** JVM heap used của Connect worker, GC time, container memory working set; `QueueRemainingCapacity` của Debezium.

**Alert.** Heap > 85% kéo dài 10 phút; task FAILED (luôn luôn alert cái này — xem Case 1 vì sao); pod restart count tăng.

**Khắc phục & phòng tránh.** Tăng heap (`-Xmx`) có tính toán: ước lượng `max.queue.size × avg event size × số task`; hoặc dùng `max.queue.size.in.bytes` để chặn theo bytes thay vì số event. Giảm `max.batch.size`. Phía nguồn: yêu cầu batch job chia nhỏ transaction (tốt cho cả database lẫn CDC). Loại cột khổng lồ không cần thiết bằng `column.exclude.list`. Cô lập connector nặng sang worker riêng.

### Case 5: Connector crash vì mất schema history topic (MySQL/SQL Server)

**Triệu chứng.** Connector MySQL không start được sau một thời gian dừng, log: `The db history topic is missing... you may attempt to recover it by reconfiguring the connector to schema_only_recovery`.

**Root cause.** Topic `schema-changes.<name>` (database history) bị xóa hoặc — phổ biến hơn — bị **retention mặc định 7 ngày dọn mất**, vì người tạo topic quên rằng topic này phải giữ vĩnh viễn.

**Cơ chế kỹ thuật.** Binlog MySQL chứa row event *không kèm schema* — chỉ có column index và giá trị. Debezium phải tự duy trì "cuốn phim" schema theo thời gian: mỗi DDL được ghi vào schema history topic, và khi restart, connector replay topic này để dựng lại schema *đúng tại vị trí binlog đang đọc*. Mất topic = mất khả năng diễn giải binlog.

**Cách điều tra.** `kafka-topics --describe --topic schema-changes.inventory` — kiểm tra tồn tại và config `retention.ms`, `cleanup.policy`.

**Metric/Alert.** Không có metric trực tiếp — đây là lỗi cấu hình phát hiện khi đã muộn. Phòng tránh bằng audit: job định kỳ kiểm tra mọi schema history topic có `retention.ms = -1`, `cleanup.policy = delete` (không phải compact!), replication factor ≥ 3.

**Khắc phục.** Nếu schema không đổi từ thời điểm mất: `snapshot.mode = schema_only_recovery` (bản mới: `recovery`) — connector re-read schema hiện tại rồi tiếp tục từ offset cũ. Nếu schema *đã* đổi trong khoảng mất mát: không có cách an toàn tuyệt đối — re-snapshot là lựa chọn đúng.

**Phòng tránh.** Tạo schema history topic **thủ công trước** với config đúng, đừng để auto-create; đưa vào checklist deploy connector; backup định kỳ (topic này nhỏ, dump rẻ).

### Case 6: Connector restart liên tục do connection timeout / mạng chập chờn

**Triệu chứng.** Task FAILED với `Connection reset`, `EOFException`, `terminating connection due to administrator command`, hoặc treo không nhận event dù RUNNING. Thường xuất hiện sau khi thêm firewall, LB, hoặc chuyển database sang managed service.

**Root cause & cơ chế.** Replication connection là **kết nối dài, im lặng khi ít traffic** — đúng loại kết nối mà NAT gateway, LB, firewall thích cắt (idle timeout 5–15 phút). Có trường hợp tệ hơn: kết nối bị cắt "nửa vời" (half-open) — connector tưởng vẫn kết nối, không nhận gì, không lỗi — pipeline đứng im lặng, slot giữ WAL (quay lại Case 1).

**Cách điều tra.** So thời điểm lỗi với idle timeout của thiết bị mạng giữa đường; `SELECT * FROM pg_stat_replication;` phía DB xem kết nối còn không; bật TCP keepalive; kiểm tra `MilliSecondsSinceLastEvent` phía connector.

**Metric/Alert.** `MilliSecondsBehindSource` và `MilliSecondsSinceLastEvent` (kết hợp heartbeat Case 2 — có heartbeat thì "im lặng 5 phút" chắc chắn là bất thường); task status; số lần restart.

**Khắc phục & phòng tránh.** `database.tcpKeepAlive=true`, chỉnh keepalive OS (`net.ipv4.tcp_keepalive_time=60`); heartbeat interval nhỏ hơn idle timeout của mọi thiết bị trên đường đi; retry policy của Connect (`errors.retry.timeout`); với PostgreSQL đặt `wal_sender_timeout` hợp lý hai phía. Quan trọng nhất: alert dựa trên *sự im lặng* (không có event kể cả heartbeat trong N phút) chứ không chỉ dựa trên task status — half-open connection không làm task FAILED.

### Case 7: Kafka Connect mất offset — re-snapshot ngoài ý muốn hoặc mất vị trí

**Triệu chứng.** Connector restart và bắt đầu snapshot lại từ đầu (downstream đột nhiên nhận lại hàng trăm triệu event "READ"), hoặc bắt đầu đọc từ vị trí sai. Kịch bản kinh điển: sau sự cố Kafka cluster, hoặc sau khi ai đó "dọn dẹp topic".

**Root cause.** Topic `connect-offsets` — nơi Connect lưu vị trí đọc (LSN/GTID/binlog position) của mọi source connector — bị xóa, bị tạo lại với `cleanup.policy = delete` thay vì `compact`, hoặc retention sai làm mất record offset. Cũng gặp: đổi `connector name` hoặc `topic.prefix` — offset được key theo tên, đổi tên là "mất" offset một cách logic dù topic còn nguyên.

**Cơ chế kỹ thuật.** `connect-offsets` phải là compacted topic: mỗi connector ghi offset mới nhất theo key; compaction giữ bản cuối. Nếu `cleanup.policy = delete` với retention 7 ngày, một connector chạy ổn định *vẫn* ổn — cho đến lần restart đầu tiên sau khi record cũ bị dọn, lúc đó connector thấy "chưa có offset" và hành xử theo `snapshot.mode`: `initial` → re-snapshot toàn bộ (đau nhưng an toàn); `never`/`no_data` → đọc từ đầu binlog còn lại hoặc từ vị trí hiện tại — **có thể mất dữ liệu âm thầm** (xem Case 8).

**Cách điều tra.**

```bash
kafka-topics --describe --topic connect-offsets   # kiểm tra cleanup.policy=compact
kafka-console-consumer --topic connect-offsets --from-beginning \
  --property print.key=true    # xem offset hiện có của từng connector
```

**Metric/Alert.** Audit config topic hệ thống (offsets, configs, status) — cả ba phải compact, RF ≥ 3, `min.insync.replicas = 2`. Alert khi connector chuyển sang trạng thái snapshot ngoài kế hoạch (`SnapshotRunning = true` bất ngờ).

**Khắc phục.** Nếu có backup offset: dùng REST API (Kafka Connect 3.6+) `PATCH /connectors/<name>/offsets` hoặc produce trực tiếp record offset với đúng key/format vào topic, rồi restart connector. Nếu không có backup: chọn giữa re-snapshot (đúng nhưng đắt) và đặt offset thủ công theo LSN/GTID ước lượng (nhanh nhưng rủi ro mất/duplicate event — chỉ làm khi hiểu rõ).

**Phòng tránh.** **Backup offset định kỳ**: một cron consume topic `connect-offsets` và lưu bản mới nhất ra object storage (S3) mỗi giờ — chi phí gần bằng 0, cứu bạn cả tuần re-snapshot. Đưa 3 topic hệ thống của Connect vào diện "hạ tầng trọng yếu", cấm thao tác tay.

### Case 8: Mất event — snapshot mode sai khi log đã bị purge

**Triệu chứng.** Âm thầm, thường phát hiện sau nhiều tuần: đối soát số liệu giữa nguồn và đích lệch nhau; một dải thời gian không có event nào trong topic dù nguồn có ghi.

**Root cause.** Đây gần như luôn là **lỗi cấu hình**, gồm các biến thể: (a) MySQL: connector dừng lâu hơn binlog retention (`binlog_expire_logs_seconds` mặc định thấp, hoặc RDS purge nhanh), khi start lại với `snapshot.mode=when_needed` thì ổn, nhưng với cấu hình bỏ qua kiểm tra hoặc offset trỏ vào binlog đã purge kèm xử lý sai, một dải thay đổi biến mất; (b) PostgreSQL: slot bị drop (Case 1 khắc phục sai cách, hoặc `max_slot_wal_keep_size` vô hiệu slot) rồi connector được tạo lại với `snapshot.mode = never`/`no_data` — connector đọc *từ hiện tại*, toàn bộ thay đổi trong khoảng trống bị mất vĩnh viễn; (c) tạo connector mới với `never` mà tưởng dữ liệu lịch sử "tự có".

**Cơ chế kỹ thuật.** `snapshot.mode = never/no_data` nghĩa là "tôi cam kết rằng vị trí bắt đầu đọc log nối liền với dữ liệu đã có ở downstream". Cam kết đó vỡ ngay khi có khoảng trống giữa "log còn lại" và "điểm dừng trước đó". MySQL connector khi phát hiện offset trỏ vào binlog không tồn tại sẽ fail với `Cannot replay binlog... server has purged` — fail là *may mắn*; kịch bản tệ là người vận hành "sửa" bằng cách xóa offset và để `never`, biến lỗi ồn ào thành mất dữ liệu im lặng.

**Cách điều tra.**

```sql
-- MySQL: binlog còn giữ từ đâu?
SHOW BINARY LOGS;
SELECT @@binlog_expire_logs_seconds;
```
So sánh binlog position trong offset (đọc từ `connect-offsets`) với danh sách binlog còn tồn tại. Đối soát dữ liệu: count/checksum theo khung thời gian giữa nguồn và đích (xem reconciliation ở chương 12).

**Metric/Alert.** Không thể alert "đã mất event" trực tiếp — phải alert *điều kiện dẫn đến nó*: connector dừng quá X% của binlog retention; slot bị drop/invalidated (`wal_status='lost'`); và chạy **reconciliation job định kỳ** như lưới phát hiện cuối cùng.

**Khắc phục.** Khi khoảng trống đã xảy ra: chỉ có re-snapshot (toàn phần hoặc incremental snapshot cho các bảng nghi ngờ) để vá. Đánh dấu khoảng thời gian dữ liệu không tin cậy cho downstream analytics.

**Phòng tránh.** Binlog retention ≥ 3–7 ngày (nhiều hơn nhiều so với thời gian phát hiện + sửa connector lâu nhất bạn từng gặp); không bao giờ dùng `never` trừ khi có quy trình bootstrap dữ liệu độc lập và được đối soát; coi "drop slot / xóa offset" là thao tác phải đi kèm quyết định re-snapshot có chủ đích.

### Case 9: Duplicate event — at-least-once là mặc định, không phải bug

**Triệu chứng.** Downstream thấy cùng một thay đổi hai lần: row count đích cao hơn nguồn (nếu sink INSERT mù), số liệu analytics đếm trùng, hoặc consumer xử lý side-effect hai lần (gửi 2 email).

**Root cause.** Connector crash **giữa** thời điểm produce record lên Kafka và thời điểm commit offset (offset được flush định kỳ theo `offset.flush.interval.ms`, mặc định 60 giây). Khi restart, connector đọc lại từ offset cũ → phát lại các event đã produce. Đây là hành vi **thiết kế** của mô hình at-least-once: hệ thống chọn "thà lặp còn hơn mất". Restart connector, rebalance Connect cluster, kill pod — tất cả đều có thể sinh duplicate.

**Cơ chế kỹ thuật.** Hai bước "produce" và "ghi offset" không atomic (source connector không nằm trong Kafka transaction ở đa số phiên bản triển khai phổ biến). Cửa sổ duplicate = lượng event từ lần flush offset cuối đến thời điểm crash — với `offset.flush.interval.ms=60000` và 5k event/giây, tối đa ~300k event lặp lại mỗi lần crash (minh họa).

**Cách điều tra.** So `source.lsn`/`source.pos` trong payload event — duplicate có cùng position nguồn; log connector cho thấy restart tại thời điểm tương ứng.

**Metric/Alert.** Số lần task restart; phía consumer: tỷ lệ event bị bỏ qua bởi logic idempotency (nếu tăng vọt → vừa có replay).

**Khắc phục & phòng tránh.** Không "sửa" ở producer — **thiết kế consumer idempotent** là đường duy nhất bền vững: (a) sink kiểu upsert theo primary key (Elasticsearch index theo `_id`, JDBC sink `insert.mode=upsert`, ClickHouse ReplacingMergeTree); (b) version check theo LSN/`source.ts_ms` — bỏ qua event cũ hơn bản đã áp; (c) với side-effect không idempotent tự nhiên (gửi email, gọi API tiền), dùng dedup store (processed event id) có TTL. Chi tiết pattern ở chương 13. Có thể giảm tần suất duplicate bằng exactly-once support cho source connector (Kafka 3.3+, `exactly.once.source.support`) nhưng đừng lấy đó làm lý do bỏ idempotency ở sink — hàng phòng ngự cuối vẫn phải có.

### Case 10: Event sai thứ tự

**Triệu chứng.** Trạng thái cuối ở downstream thỉnh thoảng "cũ hơn" nguồn: order hiển thị `PAID` trong khi nguồn đã `SHIPPED`; sau đó có thể tự đúng lại (nếu có event mới) hoặc kẹt vĩnh viễn.

**Root cause.** Kafka chỉ bảo đảm thứ tự **trong một partition**. Sai thứ tự xảy ra khi: (a) message key sai — event của cùng một row rơi vào partition khác nhau (ví dụ ai đó đặt SMT đổi key, hoặc topic dùng key null → round-robin); (b) consumer đọc một partition nhưng **xử lý song song không theo key** (worker pool nội bộ lấy message ra xử lý đa luồng); (c) repartition ở tầng stream processing với key khác; (d) tăng số partition của topic đang chạy — cùng key bị hash sang partition mới, event cũ và mới nằm hai partition, thứ tự tương đối mất trong giai đoạn chuyển tiếp.

**Cơ chế kỹ thuật.** Debezium mặc định key event theo primary key của bảng → mọi thay đổi của một row vào đúng một partition → thứ tự per-row được giữ. Mọi thứ phá vỡ ánh xạ "một row → một dòng tuần tự" đều phá thứ tự. Lưu ý: thứ tự **giữa các row khác nhau** vốn dĩ không được bảo đảm giữa các partition — thiết kế downstream không được phụ thuộc vào nó (đây là lý do foreign key constraint ở sink là nguồn đau khổ — xem Case 15 và chương 12).

**Cách điều tra.** Với một key bị nghi: dump event của key đó từ mọi partition (`kafka-console-consumer` + grep), so `source.lsn`/`ts_ms` với partition/offset — nếu cùng key xuất hiện ở ≥ 2 partition là chuông báo đỏ. Xem lại lịch sử thay đổi partition count của topic.

**Metric/Alert.** Khó có metric trực tiếp; phía consumer nên đếm "event bị từ chối vì version cũ hơn" — tăng bất thường là dấu hiệu.

**Khắc phục & phòng tránh.** Giữ key = primary key; **không bao giờ tăng partition count** của topic CDC đang chạy (tạo topic mới + migrate có kiểm soát); consumer xử lý tuần tự theo key (parallelism theo partition, hoặc key-based worker dispatch); sink dùng version check để "sai thứ tự" chỉ còn là hiệu năng chứ không phải sai dữ liệu — event cũ đến muộn bị bỏ qua thay vì đè lên bản mới.

### Case 11: Schema change làm connector crash

**Triệu chứng.** Task FAILED ngay sau một đợt migration của team ứng dụng. Stack trace nhắc đến parse DDL (MySQL), hoặc lỗi từ Schema Registry: `Schema being registered is incompatible with an earlier schema`.

**Root cause.** Ba biến thể: (a) DDL mà connector không parse được hoặc chưa hỗ trợ (các lệnh `ALTER` phức tạp, syntax mới của database vượt trước parser của Debezium); (b) PostgreSQL `ALTER COLUMN TYPE` các kiểu không tương thích, hoặc thao tác với REPLICA IDENTITY sai làm event thiếu dữ liệu; (c) schema mới sinh ra Avro schema **vi phạm compatibility mode** của Schema Registry (ví dụ thêm cột NOT NULL không default với BACKWARD compatibility) — connector produce bị reject, task fail.

**Cơ chế kỹ thuật.** Chuỗi phụ thuộc: DDL → parser/schema tracking của connector → converter (Avro/JSON Schema) → Schema Registry compatibility check → consumer deserializer. Gãy ở bất kỳ khâu nào cũng dừng pipeline. Điểm nghẹt về mặt tổ chức: **người đổi schema (team ứng dụng) và người chịu hậu quả (team data platform) thường khác nhau và không nói chuyện với nhau.**

**Cách điều tra.** Log task (`GET /connectors/<name>/status` lấy trace); đối chiếu thời điểm fail với migration history (`SELECT * FROM flyway_schema_history ORDER BY installed_on DESC` hoặc tương đương); với Schema Registry: `GET /subjects/<topic>-value/versions` xem version mới nhất và test compatibility qua endpoint `/compatibility`.

**Metric/Alert.** Task status (lần thứ ba: luôn alert FAILED); đăng ký alert trên schema registry rejection trong log Connect.

**Khắc phục.** Với DDL parse error trên MySQL: nâng version Debezium (parser được vá liên tục), hoặc — thận trọng — dùng `database.history.skip.unparseable.ddl=true` chỉ khi DDL đó chắc chắn không liên quan bảng được capture. Với compatibility reject: hoặc sửa schema evolution cho tương thích (thêm default), hoặc có quyết định chủ đích nâng version không tương thích kèm kế hoạch cho consumer.

**Phòng tránh.** Đưa "ảnh hưởng CDC" vào quy trình review migration; quy ước schema evolution an toàn (expand-contract: thêm cột nullable → migrate code → xóa cột sau); test migration trên staging **có pipeline CDC chạy**; chọn compatibility mode có chủ đích (BACKWARD là điểm khởi đầu hợp lý — consumer cũ đọc được data mới).

### Case 12: Connector không theo kịp tốc độ ghi của database

**Triệu chứng.** `MilliSecondsBehindSource` tăng tuyến tính không hồi phục, kể cả ban đêm; retained WAL tăng theo (Case 1 dạng "mãn tính"). Không có lỗi nào trong log — mọi thứ RUNNING, chỉ là *chậm*.

**Root cause.** Đụng **throughput ceiling của single task**: logical decoding/binlog reading về bản chất là tuần tự cho một slot/một connector — không scale ngang bằng cách tăng `tasks.max` (với PostgreSQL/MySQL connector, source task là 1). Ceiling điển hình vài chục nghìn event/giây tùy kích thước event, CPU, network, serialization (minh họa — Avro nhẹ hơn JSON đáng kể).

**Cơ chế & điều tra.** Xác định nghẽn ở khâu nào: (a) decoding phía DB — CPU của walsender/`pgoutput`; (b) connector — CPU worker, `QueueRemainingCapacity` (queue luôn đầy → producer chậm; queue luôn rỗng → đọc chậm); (c) Kafka produce — `batch-size-avg`, `record-queue-time`, compression. JMX Debezium: `MilliSecondsBehindSource`, `TotalNumberOfEventsSeen` — chia theo thời gian ra events/giây thực tế, so với tốc độ ghi nguồn.

**Metric/Alert.** `MilliSecondsBehindSource` — WARNING > 5 phút trong 15 phút, CRITICAL > 30 phút (minh họa; sau khi trừ chu kỳ backlog tự nhiên như batch job đêm).

**Khắc phục & phòng tránh.** Theo thứ tự rẻ → đắt: (1) **Lọc bảng**: bỏ bảng log/staging/audit ra khỏi publication — tôi từng giảm 70% event chỉ bằng việc loại một bảng "click_events" không ai consume; (2) tune `max.batch.size` (2048→8192), `max.queue.size`, `poll.interval.ms`, producer `batch.size`, `linger.ms`, `compression.type=lz4`; (3) Avro thay JSON; (4) **tách connector**: nhóm bảng nóng một connector/slot riêng, bảng nguội một connector khác — mỗi slot một luồng tuần tự riêng (đánh đổi: thứ tự cross-table giữa hai nhóm không còn, và thêm slot = thêm rủi ro Case 1 × 2); (5) xem lại nguồn: batch UPDATE khổng lồ có cần chảy qua CDC không, hay nên xử lý bằng kênh khác.

### Case 13: Partition rebalancing của Connect/consumer gây pause ngắn lặp đi lặp lại

**Triệu chứng.** Lag kiểu răng cưa: cứ vài phút lại có gián đoạn 10–60 giây; log đầy `Rebalance started`, `(Re-)joining group`. Thường sau khi thêm worker, deploy rolling, hoặc có worker chập chờn (GC dài làm miss heartbeat).

**Root cause & cơ chế.** Connect cluster rebalance task giữa worker khi membership đổi; consumer group rebalance khi consumer vào/ra. Với eager rebalancing cổ điển, *mọi* task/consumer dừng trong lúc chia lại (stop-the-world). Worker có GC pause dài hơn `session.timeout.ms` bị coi là chết → rebalance → tải dồn → GC tiếp → rebalance tiếp: rebalance storm.

**Điều tra.** Log Connect (`Rebalance`), metric `rebalance-rate`, GC log của worker, event Kubernetes (pod bị restart/evict?).

**Metric/Alert.** `kafka.connect:type=connect-worker-rebalance-metrics` — `completed-rebalances-total` tăng bất thường; consumer group: `rebalance-latency-avg`. Alert khi > X rebalance/giờ.

**Khắc phục & phòng tránh.** Connect từ 2.3 đã có incremental cooperative rebalancing (mặc định) — kiểm tra không ai chỉnh về eager. Với consumer: `partition.assignment.strategy = CooperativeStickyAssignor`; **static membership** (`group.instance.id`) để restart pod không kích hoạt rebalance nếu quay lại trong `session.timeout.ms`; tune session timeout đủ lớn để sống qua GC/rolling deploy; giảm GC pause (heap hợp lý, G1/ZGC).

## Nhóm C: Sự cố phía sink và downstream

### Case 14: Consumer chậm gây backlog — Kafka lag tăng đột biến

**Triệu chứng.** Consumer lag từ vài nghìn nhảy lên hàng chục triệu; dashboard downstream trễ nhiều giờ; đôi khi kèm consumer bị kick khỏi group (`max.poll.interval.ms exceeded`).

**Root cause phổ biến.** (a) Sink chậm đi (Elasticsearch bulk reject, DB đích lock/IO); (b) GC pause dài phía consumer; (c) **hot partition** — key phân bố lệch (một tenant lớn chiếm 60% traffic), một partition gánh phần lớn dữ liệu trong khi các consumer khác ngồi chơi; (d) nguồn tăng đột biến (backfill, campaign); (e) một message độc (poison pill) làm consumer crash-restart loop — lag tăng trong khi throughput bằng 0.

**Điều tra.**

```bash
kafka-consumer-groups --bootstrap-server ... --describe --group es-sink
# Xem LAG theo TỪNG partition: lệch đều = sink chậm; lệch một partition = hot key
```
Poison pill: log consumer lặp cùng một offset. Sink: latency/reject metric của đích (ES `bulk.rejected`, xem Case 15).

**Metric/Alert.** Consumer lag (records và **thời gian** — lag 1 triệu record của topic 100k msg/s khác hẳn lag 1 triệu của topic 100 msg/s); consumption rate; alert khi lag-in-seconds > SLA của read model (ví dụ > 300 giây trong 10 phút).

**Khắc phục & phòng tránh.** Scale consumer chỉ có tác dụng đến số partition — hot partition thì scale vô ích, phải xử lý key (composite key, hoặc chấp nhận và cô lập tenant lớn). Tăng batch phía sink (Case 15, 16). Poison pill: DLQ (`errors.tolerance=all` + `errors.deadletterqueue.topic.name` cho sink connector; try-catch + DLQ topic cho consumer tự viết) — **đứng im vì một message là lựa chọn tồi nhất**. Capacity: giữ headroom ≥ 2× throughput trung bình để có khả năng *bắt kịp* sau sự cố — hệ chỉ vừa đủ nhanh sẽ không bao giờ tiêu hết backlog.

### Case 15: Elasticsearch indexing trễ / reject

**Triệu chứng.** Lag tăng ở sink ES; log connector: `429 es_rejected_execution_exception`; search kết quả cũ hàng chục phút.

**Root cause & cơ chế.** (a) Bulk size quá nhỏ hoặc quá lớn; (b) `refresh_interval=1s` mặc định — mỗi giây một segment mới, chi phí lớn khi ingest cao; (c) **mapping explosion**: dynamic mapping trên payload JSON tự do sinh hàng chục nghìn field, cluster state phình, mọi thao tác chậm; (d) write thread pool queue đầy → reject; (e) quá nhiều shard nhỏ.

**Điều tra.** `GET _cat/thread_pool/write?v` (rejected tăng?), `GET _cluster/stats` (số field trong mapping), `GET _cat/indices?v` (shard size), slow log của index.

**Metric/Alert.** ES: bulk rejection rate, indexing latency, thread pool queue; alert reject > 0 kéo dài; số field mapping vượt 1000 là dấu hiệu thiết kế sai.

**Khắc phục & phòng tránh.** Bulk 5–15 MB/request (tune bằng đo, không đoán); `refresh_interval=30s` cho index ingest nặng (đánh đổi: search thấy dữ liệu chậm hơn — nêu rõ với product); **strict/explicit mapping** cho index từ CDC — không thả dynamic mapping với dữ liệu tự do; index theo `_id = primary key` để idempotent (Case 9); backpressure: sink connector giảm `batch.size`/`max.in.flight` khi bị 429 thay vì fail.

### Case 16: ClickHouse ingest không kịp — "too many parts"

**Triệu chứng.** Lỗi `Too many parts (N). Merges are processing significantly slower than inserts`; insert bị reject/delay; CPU ClickHouse dồn cho merge.

**Root cause & cơ chế.** ClickHouse MergeTree tạo một *part* trên mỗi lần INSERT; nó được thiết kế cho **insert lớn, ít lần** (khuyến nghị kinh điển: mỗi insert hàng chục nghìn tới hàng triệu rows, ≤ vài insert/giây). Consumer CDC ngây thơ insert từng event hoặc batch trăm rows → hàng nghìn part nhỏ/phút → merge không kịp → ngưỡng `parts_to_throw_insert` (mặc định 300 part/partition) → lỗi.

**Điều tra.**

```sql
SELECT table, count() AS parts FROM system.parts
WHERE active GROUP BY table ORDER BY parts DESC;
SELECT * FROM system.merges;   -- merge đang chạy, backlog
```

**Metric/Alert.** `system.parts` active count per partition (alert > 150 — minh họa), insert rate, merge queue, `DelayedInserts`/`RejectedInserts`.

**Khắc phục & phòng tránh.** Batch phía consumer: gom 10k–100k rows hoặc flush mỗi 5–30 giây (đánh đổi độ trễ — với analytics gần như luôn chấp nhận được); hoặc dùng **async_insert** của ClickHouse; hoặc Kafka table engine/ClickPipes để ClickHouse tự kéo theo batch. Dedup/UPDATE semantics: `ReplacingMergeTree(version)` + `FINAL` khi query (hoặc argMax pattern) — đừng dùng `ALTER TABLE UPDATE` cho luồng CDC, mutation không phải để chạy mỗi giây. Partition key hợp lý (theo tháng/ngày, không theo cardinality cao).

## Nhóm D: Sự cố topology và thảm họa

### Case 17: Split-brain pipeline — hai "bộ não" cùng ghi một luồng

**Triệu chứng.** Biến thể 1 (PostgreSQL, 2 connector cùng slot): hai connector tranh nhau slot — bên nọ đá bên kia (`replication slot is active for PID ...`), pipeline giật cục, event có thể chia đôi ngẫu nhiên giữa hai bên nếu tranh chấp kéo dài. Biến thể 2 (2 Connect cluster cùng `group.id`/cùng bộ topic nội bộ): hai cluster tưởng mình là một — connector bị "bốc hơi" hoặc chạy đúp, config đè nhau, **duplicate event quy mô lớn** và offset bị ghi chéo. Kịch bản thực tế: team clone môi trường staging từ production copy nguyên `group.id` và `offset.storage.topic`, staging Connect kết nối nhầm Kafka production.

**Root cause.** Vi phạm nguyên tắc **một luồng đọc — một chủ sở hữu**: slot PostgreSQL là single-consumer; bộ ba topic nội bộ + group.id định danh duy nhất một Connect cluster.

**Điều tra.** `SELECT slot_name, active_pid FROM pg_replication_slots;` rồi truy PID → connection từ đâu; liệt kê consumer group `connect-<group.id>` xem member từ những host/IP nào — thấy IP lạ là tìm ra cluster thứ hai.

**Metric/Alert.** Số member của Connect group thay đổi bất thường; slot bị chiếm/nhả liên tục (log PostgreSQL); duplicate rate phía consumer tăng vọt.

**Khắc phục.** Tắt ngay bên "lậu"; kiểm tra khoảng thời gian split-brain và đối soát dữ liệu; nếu offset topic bị ghi chéo, khôi phục offset từ backup (Case 7).

**Phòng tránh.** Quy ước đặt tên có môi trường trong mọi định danh (`prod-cdc-connect`, `staging-cdc-connect`, slot `dbz_prod_orders`); network policy chặn staging → Kafka production; checklist clone môi trường phải đổi group.id + 3 topic nội bộ; mỗi connector một slot riêng, ghi rõ ownership.

### Case 18: Network partition giữa database và connector

**Triệu chứng.** Task có thể vẫn RUNNING (retry nội bộ) nhưng không có event; hoặc FAILED với connection error. Điểm chết người: **trong suốt thời gian mất kết nối, slot vẫn giữ WAL** — network partition 6 giờ trên database ghi 100 GB/ngày là ~25 GB WAL tích lũy (minh họa). Đây là dạng "Case 1 có hẹn giờ".

**Root cause & cơ chế.** Mạng giữa hai vùng (connector chạy ở Kubernetes cluster khác VPC/region với database) đứt hoặc suy hao; connector retry theo `errors.retry.timeout`, `retriable.restart.connector.wait.ms`. Slot phía database không biết gì về ý định của consumer — nó chỉ biết "chưa được xác nhận thì giữ".

**Điều tra.** Phân biệt "database chết" với "đường mạng chết": từ worker `psql`/`nc -vz db-host 5432`; từ DB xem `pg_stat_replication` có còn walsender không; traceroute/VPC flow logs.

**Metric/Alert.** Chính là bộ alert Case 1 (slot lag, inactive) + Case 6 (im lặng bất thường). Network partition không cần alert riêng — nó biểu hiện qua các alert đã có, miễn là chúng tồn tại.

**Khắc phục & phòng tránh.** Retry đủ kiên nhẫn (để sự cố mạng 10 phút tự lành không cần người); `max_slot_wal_keep_size` làm cầu chì cho sự cố dài; **đặt connector gần database về mặt network** (cùng region/VPC) — mỗi ranh giới mạng giữa DB và connector là một nguồn sự cố; định nghĩa trước ngưỡng thời gian mà sau đó bạn chủ động quyết định drop slot + re-snapshot thay vì chờ vô hạn.

### Case 19: Database failover — slot không tồn tại trên standby được promote

**Triệu chứng.** Sau failover (primary chết, standby promote), connector fail: `replication slot "dbz_orders" does not exist`. Người vận hành tạo lại slot trên primary mới, connector chạy tiếp — và **âm thầm mất mọi thay đổi từ điểm failover đến lúc slot mới tạo**, cộng thêm rủi ro mất các giao dịch chưa kịp replicate lúc failover.

**Root cause & cơ chế.** Ở PostgreSQL trước version 17, replication slot là trạng thái **cục bộ của primary**, không được replicate sang standby. Failover = slot biến mất = mất "con trỏ" và mất luôn phần WAL bảo đảm. Tạo slot mới chỉ bắt đầu từ *hiện tại* — khoảng trống ở giữa không ai capture.

**Cách điều tra sau failover.** Xác định LSN/thời điểm promote (log PostgreSQL, `pg_controldata`); so offset cuối của connector (từ `connect-offsets`) với điểm promote → độ rộng khoảng trống; đối soát dữ liệu trong khoảng đó.

**Metric/Alert.** Alert trên sự kiện failover (từ hệ thống HA — Patroni, RDS event) phải **tự động kèm cảnh báo CDC**: "failover xảy ra → kiểm tra slot + đánh giá re-snapshot". Đừng để hai hệ thống alert này tách rời.

**Khắc phục.** Trung thực với dữ liệu: nếu có khoảng trống, chạy incremental snapshot các bảng quan trọng (hoặc full re-snapshot) thay vì hy vọng "chắc không mất gì".

**Phòng tránh.** PostgreSQL 17+: **failover slot** (`sync_replication_slots`, slot được đồng bộ sang standby) — nếu bạn thiết kế hệ mới, đây là lý do mạnh để lên PG 17. Trước đó: Patroni có cơ chế tái tạo slot có kiểm soát, một số bản quản lý hỗ trợ đồng bộ slot — nhưng phải test failover *có pipeline CDC* trong drill, không tin chay tài liệu. **MySQL có lợi thế ở đây**: với GTID (`gtid_mode=ON`), vị trí đọc là tọa độ logic toàn cluster — connector failover sang replica mới và tiếp tục đúng chỗ theo GTID set, miễn binlog còn giữ (một lý do nhiều đội chọn MySQL cho luồng CDC trọng yếu). Dù nền nào: đưa "failover drill kèm CDC" vào lịch diễn tập ít nhất mỗi quý.

### Case 20: Disaster Recovery tổng thể cho pipeline CDC

Đây không hẳn một sự cố đơn lẻ mà là **năng lực tổng hợp**: khi mất Kafka cluster, mất Connect cluster, hoặc mất cả region — khôi phục thế nào, theo thứ tự nào, và mất gì?

**Backup những gì (theo thứ tự quan trọng):**
1. **Connector config** — export JSON qua REST API, lưu Git (thực ra: config phải *sống* trong Git từ đầu, deploy qua CI/CD, không tạo tay).
2. **Offset** (`connect-offsets`) — snapshot định kỳ ra object storage (Case 7). Đây là thứ đắt nhất nếu mất: mất offset = re-snapshot mọi thứ.
3. **Schema history topic** (MySQL) — dump định kỳ; topic nhỏ, chi phí không đáng kể.
4. **Schema Registry** (`_schemas` topic) — mất nó, consumer không đọc được Avro cũ.
5. Sink state nếu có (checkpoint Flink, v.v.).

**Thứ tự khôi phục** (kịch bản mất Kafka + Connect, database nguồn còn sống):
1. Dựng Kafka cluster, tạo lại topic hệ thống (offsets/configs/status — compact, RF3) và Schema Registry (restore `_schemas`).
2. Restore offset backup vào `connect-offsets` **trước khi** start bất kỳ connector nào.
3. Dựng Connect cluster (đúng group.id), deploy connector config từ Git — connector đọc offset đã restore, nối lại từ vị trí backup.
4. Đánh giá khoảng trống: offset backup cũ hơn hiện tại X giờ → connector sẽ đọc lại X giờ WAL/binlog — **điều kiện tiên quyết: WAL/binlog còn giữ đủ X giờ** (đây là mối liên hệ trực tiếp giữa RPO của backup offset và retention của log nguồn: retention log nguồn phải > chu kỳ backup offset + thời gian dựng lại hạ tầng).
5. Nếu không đủ log: re-snapshot có chọn lọc, ưu tiên bảng theo mức quan trọng nghiệp vụ; downstream idempotent (Case 9) làm cho việc "đọc lại X giờ" an toàn.

**RTO/RPO cho pipeline CDC** — điểm nhiều người hiểu sai: pipeline CDC *không mất dữ liệu* miễn log nguồn còn (dữ liệu gốc nằm ở database, pipeline chỉ là kênh vận chuyển) — **RPO thực chất của pipeline = 0 nếu và chỉ nếu log retention > RTO của pipeline.** Do đó biến số cần quản trị là RTO: dựng lại toàn bộ trong bao lâu (mục tiêu minh họa: < 4 giờ với hạ tầng as-code + backup offset mỗi giờ + binlog/WAL retention 7 ngày). Diễn tập DR thật — restore offset vào cluster mới, chạy đối soát — mỗi 6 tháng; DR chưa diễn tập là DR trên giấy.

## Bảng tổng hợp: failure → metric chính → alert threshold gợi ý

Threshold là **minh họa điển hình** — hiệu chỉnh theo hệ của bạn.

| # | Failure | Metric chính | Alert gợi ý |
|---|---|---|---|
| 1 | Slot giữ WAL, disk đầy | `pg_wal_lsn_diff(current, restart_lsn)`; slot active | >10GB warn, >50GB crit; inactive >10 phút |
| 2 | Thiếu heartbeat, lag ảo | confirmed_flush_lsn đứng yên | như case 1; bắt buộc heartbeat |
| 3 | Snapshot dài, DB quá tải | SnapshotRunning, DB CPU/IO, xact dài | snapshot > kế hoạch; xact >1h |
| 4 | OOM restart loop | JVM heap, pod restart, OOMKilled | heap >85%/10 phút; restart >3 lần/giờ |
| 5 | Mất schema history | audit retention topic | retention.ms ≠ -1 → fail audit |
| 6 | Connection timeout | MilliSecondsSinceLastEvent | im lặng > 3× heartbeat interval |
| 7 | Mất offset | SnapshotRunning bất ngờ; audit cleanup.policy | snapshot ngoài kế hoạch |
| 8 | Mất event do config | binlog/WAL retention vs downtime; reconciliation | downtime > 50% retention; đối soát lệch >0 |
| 9 | Duplicate event | task restart count; dedup-skip rate | restart tăng; skip rate đột biến |
| 10 | Sai thứ tự | version-reject rate ở consumer | tăng bất thường |
| 11 | Schema change crash | task status; registry reject | FAILED bất kỳ lúc nào |
| 12 | Không theo kịp nguồn | MilliSecondsBehindSource | >5 phút warn, >30 phút crit |
| 13 | Rebalance storm | rebalance rate, GC pause | > X rebalance/giờ |
| 14 | Consumer backlog | consumer lag theo giây, per-partition | lag-giây > SLA read model |
| 15 | ES trễ/reject | bulk rejected, thread pool queue | rejected > 0 kéo dài 5 phút |
| 16 | ClickHouse too many parts | active parts/partition, DelayedInserts | parts >150 warn, >250 crit |
| 17 | Split-brain | slot active_pid đổi liên tục; group member lạ | slot bị chiếm/nhả >3 lần/10 phút |
| 18 | Network partition | bộ alert case 1 + 6 | như case 1, 6 |
| 19 | Failover mất slot | HA failover event | failover → auto-ticket kiểm tra CDC |
| 20 | DR | tuổi backup offset; kết quả DR drill | backup >2 chu kỳ không chạy |

## Dashboard nên có

Bốn dashboard, theo đúng bốn tầng của pipeline — khi có sự cố, bạn nhìn từ trái sang phải để khoanh vùng:

1. **Database**: retained WAL per slot, slot active, `pg_wal` size, disk free; MySQL: binlog oldest timestamp, GTID executed; số transaction dài; CPU/IO khi có snapshot.
2. **Connect/Debezium**: task status theo connector (bảng màu xanh/đỏ — dashboard quan trọng nhất nhìn trong 2 giây), `MilliSecondsBehindSource`, events/giây, `QueueRemainingCapacity`, JVM heap/GC, rebalance rate, SnapshotRunning.
3. **Kafka**: consumer lag theo group (records + giây), produce/consume rate theo topic CDC, under-replicated partitions, dung lượng topic hệ thống.
4. **Sink**: ingest latency end-to-end (so `source.ts_ms` với thời điểm ghi đích — metric người dùng thực sự cảm nhận), ES bulk reject, ClickHouse parts/merges, DLQ depth (DLQ có message mới phải có người xem — DLQ không ai đọc là /dev/null có chi phí).

Và một **số duy nhất cho management**: end-to-end freshness — "dữ liệu ở read model cũ hơn nguồn bao nhiêu giây". Mọi thứ khác là chi tiết kỹ thuật; con số này là SLA.

## Tóm tắt chương

- Sự cố nguy hiểm nhất của CDC không nằm ở pipeline mà ở **tác động ngược lên database nguồn**: replication slot bị bỏ rơi giữ WAL đến khi disk đầy và database ngừng ghi. `max_slot_wal_keep_size` + alert slot lag từ ngày đầu là hai việc không được phép trì hoãn; heartbeat là cấu hình bắt buộc.
- Ba tài sản trạng thái phải bảo vệ như hạ tầng trọng yếu: **offset topic** (compact, backup định kỳ), **schema history topic** (retention vô hạn), **connector config** (sống trong Git). Mất offset = re-snapshot; mất schema history = connector MySQL không dịch được binlog.
- **At-least-once là mặc định**: duplicate xảy ra theo thiết kế, mất event xảy ra do cấu hình sai (snapshot mode `never` khi log đã purge, slot bị drop, retention quá ngắn). Consumer idempotent + version check + reconciliation job là ba lớp phòng ngự, không phải tùy chọn.
- Hiệu năng có trần: single task decoding là tuần tự — lọc bảng, tune batch, tách connector trước khi mơ scale ngang. Sink có đặc thù riêng: ClickHouse cần insert lớn ít lần, Elasticsearch cần bulk hợp lý + strict mapping.
- Topology phải tôn trọng nguyên tắc "một luồng — một chủ": slot single-consumer, Connect group định danh duy nhất. Failover database là sự kiện CDC (slot không theo primary trước PG 17; GTID cứu MySQL); DR pipeline xoay quanh một bất đẳng thức: **log retention nguồn > thời gian dựng lại pipeline**.
- Vận hành CDC = vận hành theo metric: 4 dashboard theo 4 tầng, bảng alert ở trên là bộ khung tối thiểu, và câu hỏi thường trực không phải "có hỏng không" mà là "hỏng thì bao lâu mình biết".

## Đọc tiếp

Chương 12 chuyển từ "chữa cháy" sang "thiết kế": các kiến trúc CDC thực tế theo từng domain — e-commerce, FinTech, social network, SaaS multi-tenant, data warehouse — với lý do chọn, sơ đồ, ước lượng scale/chi phí và bài học rút ra từ chính những failure case bạn vừa đọc.
