+++
title = "Chương 13: Best Practices, Anti-patterns và Khi nào KHÔNG nên dùng CDC"
date = "2026-02-20T20:00:00+07:00"
draft = false
tags = ["backend", "cdc", "kafka", "database"]
series = ["Change Data Capture"]
+++

Chương cuối này là phần "chưng cất" của toàn bộ tài liệu. Sau mười hai chương về cơ chế, pattern, sự cố và kiến trúc, câu hỏi còn lại rất thực dụng: **làm gì, tránh gì, và — quan trọng không kém — khi nào đừng làm cả**. Tôi cố tình viết chương này theo kiểu có thể in ra dán cạnh bàn on-call: mỗi practice kèm *lý do*, mỗi anti-pattern kèm *hậu quả thật*, vì quy tắc không có lý do sẽ bị bỏ qua ngay lần deadline đầu tiên.

## 13.1. Best practices — kèm lý do

### Thiết kế topic và partition

**Topic naming có quy ước, có môi trường, có version.** Ví dụ: `prod.cdc.inventory.orders` cho CDC thuần, `prod.domain.order.v1` cho domain event từ outbox. Lý do: chương 11 Case 17 — split-brain giữa staging và production bắt đầu từ những cái tên không phân biệt môi trường; và version trong tên topic là van thoát khi buộc phải breaking change contract (tạo `v2`, chạy song song, migrate consumer dần) thay vì big-bang.

**Partition strategy: key theo primary key, chọn số partition dư trước một bậc, và không bao giờ tăng partition của topic đang chạy.** Key theo PK giữ thứ tự per-row (Case 10); số partition quyết định trần parallelism của consumer nên hãy chọn cho 3 năm tới (12 thay vì 4 — partition rỗng gần như miễn phí, còn re-key topic đang chạy thì rất đắt); tăng partition giữa chừng phá vỡ ánh xạ key→partition, event cùng key nằm hai partition, thứ tự tương đối mất.

### Snapshot và offset

**Snapshot strategy được quyết định trước, không phải mặc định.** Bảng lớn: incremental snapshot (Case 3) hoặc snapshot off-peak có báo DBA; định nghĩa sẵn quy trình re-snapshot từng bảng bằng signal — vì sau sự cố (Case 8, 19) bạn *sẽ* cần vá dữ liệu chọn lọc, và 2 giờ sáng không phải lúc thiết kế quy trình.

**Backup offset, schema history, connector config — theo lịch, có kiểm tra.** Config sống trong Git và deploy qua CI/CD; `connect-offsets` snapshot ra object storage mỗi giờ; schema history topic retention vô hạn và được dump định kỳ. Lý do: đây là toàn bộ "trạng thái" của pipeline — mất offset là re-snapshot mọi thứ (Case 7), và bất đẳng thức DR của Case 20: *log retention nguồn phải lớn hơn thời gian dựng lại pipeline*.

### Schema và contract

**Schema Registry với compatibility mode chọn có chủ đích.** BACKWARD là điểm khởi đầu hợp lý cho topic CDC (consumer cũ đọc được data mới — cho phép thêm cột nullable, cấm xóa/đổi kiểu bừa). Lý do: registry biến "schema change làm vỡ consumer lúc runtime" (khó debug, phát hiện muộn) thành "schema change bị từ chối lúc produce" (ồn ào, phát hiện sớm — Case 11). Kèm theo: quy ước expand-contract cho migration, và mục "ảnh hưởng CDC" trong review DDL.

### Consumer: idempotent, retry, DLQ, backpressure

**Idempotent consumer là điều kiện tiên quyết, không phải tối ưu.** Pipeline là at-least-once (Case 9) — duplicate là hành vi thiết kế. Ba kỹ thuật theo thứ tự ưu tiên: (1) *upsert theo PK* — Elasticsearch `_id`, JDBC `insert.mode=upsert`, ClickHouse ReplacingMergeTree: xử lý lại cùng event cho cùng kết quả; (2) *version check* — so LSN/`ts_ms`, bỏ qua event cũ hơn bản đã áp: đồng thời vô hiệu hóa cả tác hại của out-of-order; (3) *dedup store* cho side-effect không tự nhiên idempotent (gửi tiền, gửi email) — bảng processed_event_id ghi cùng transaction với kết quả.

**Retry có phân loại + DLQ có người đọc.** Lỗi transient (timeout, 429) retry với backoff; lỗi vĩnh viễn (parse, constraint) đưa vào DLQ ngay — retry mãi một poison pill làm consumer đứng im là lựa chọn tồi nhất (Case 14). Và DLQ phải có alert khi depth > 0: DLQ không ai đọc là /dev/null có chi phí.

**Backpressure: thiết kế cho lúc bắt kịp, không chỉ lúc bình thường.** Sink phải giảm tốc độ có kiểm soát khi đích quá tải (giảm batch khi ES trả 429) thay vì fail; và tổng công suất tiêu thụ nên ≥ 2× throughput trung bình — hệ vừa khít không bao giờ tiêu hết backlog sau sự cố.

### Vận hành phía nguồn

**Heartbeat cho slot — cấu hình bắt buộc với PostgreSQL.** `heartbeat.interval.ms` + `heartbeat.action.query` vào bảng heartbeat nằm trong publication. Lý do đầy đủ ở Case 2: không có nó, database ít ghi vẫn tích WAL và bạn nhận alert lag "ảo" không phân biệt được với sự cố thật.

**Monitor replication lag từ ngày đầu — trước cả khi có consumer thật.** Alert slot lag + slot inactive + task FAILED là bộ ba tối thiểu, dựng *cùng lúc* với connector đầu tiên. Lý do là toàn bộ Case 1: thảm họa CDC điển hình không phải sự cố hiếm, mà là sự cố thường không được phát hiện trong ba ngày. `max_slot_wal_keep_size` là cầu chì đi kèm.

**Capacity planning checklist** trước khi lên production:
- Tốc độ ghi nguồn: event/giây trung bình và peak? Batch job đêm sinh bao nhiêu event?
- Kích thước event trung bình/lớn nhất? Có cột TEXT/JSON khổng lồ cần loại không?
- Trần single task (Case 12) còn cách peak bao xa? Kế hoạch tách connector khi nào?
- Disk cho WAL/binlog retention mục tiêu (≥ 3–7 ngày)? Kafka retention và dung lượng?
- Thời gian snapshot bảng lớn nhất? Thời gian tiêu backlog 6 giờ nếu pipeline dừng?
- Ai on-call? Runbook cho 5 alert quan trọng nhất đã có chưa?

### Security và dữ liệu nhạy cảm

**Không stream cột PII thô.** Số thẻ, mật khẩu hash, CMND không được đi qua Kafka dưới dạng thô — Kafka retention + backup + mọi consumer đọc được là bề mặt rò rỉ nhân lên nhiều lần so với database gốc. Công cụ: `column.exclude.list` (loại hẳn), SMT masking/hashing tại connector (che trước khi rời nguồn), field-level encryption nếu downstream cần khớp nối. Kèm theo: ACL Kafka theo topic, TLS cho replication connection, và user replication chỉ có quyền tối thiểu. Lý do đặt masking *tại connector* chứ không tại consumer: một khi dữ liệu thô đã vào topic, bạn không thu hồi được — retention và mirror đã nhân bản nó.

## 13.2. Anti-patterns

Mỗi anti-pattern: mô tả → vì sao nguy hiểm → hậu quả thực tế (điển hình từ kinh nghiệm, số liệu minh họa) → cách sửa.

**1. Dual write.** Ghi DB rồi publish Kafka trong code ứng dụng. Nguy hiểm vì hai hệ thống không có transaction chung — mất event và event ma là chắc chắn xảy ra theo xác suất. Hậu quả thật: hệ thống loyalty lệch điểm với đơn hàng ~0,1% — nghe nhỏ, nhưng là hàng nghìn khiếu nại/tháng và không thể tự đối soát vì không biết lệch ở đâu. Sửa: Outbox + CDC (chương 10), hoặc CDC thuần nếu chỉ cần data change.

**2. Polling vài giây trên bảng lớn.** `SELECT ... WHERE updated_at > ?` mỗi 5 giây trên bảng trăm triệu rows. Nguy hiểm: tải đọc lặp vô hạn, bỏ sót DELETE, bỏ sót update trong cùng giây, và độ trễ tỷ lệ nghịch chi phí. Hậu quả thật: query polling chiếm 30% CPU database và index `updated_at` trở thành hot spot. Sửa: CDC nếu cần near real-time; nếu không cần — polling *thưa hơn* một cách trung thực (xem 13.3).

**3. Không monitor replication lag.** Deploy connector, thấy chạy, chuyển task khác. Hậu quả thật: toàn bộ Case 1 — connector chết thứ Sáu, thứ Hai disk còn 4%. Sửa: bộ ba alert tối thiểu dựng cùng ngày với connector.

**4. Không giới hạn slot / không có cầu chì WAL.** Tin rằng "connector sẽ luôn chạy". Hậu quả: database production ngừng ghi vì hết disk — sự cố phụ trợ giết hệ thống chính. Sửa: `max_slot_wal_keep_size`, binlog retention đủ dài, quy trình drop slot khi ngừng connector dài hạn.

**5. Không backup offset.** Hậu quả thật: sự cố Kafka làm mất `connect-offsets`, đội phải re-snapshot 14 bảng — 30 giờ pipeline degraded, hai đêm mất ngủ, trong khi một cron backup 20 dòng code là đủ tránh. Sửa: Case 7/20.

**6. Topic quá nhiều partition.** "Cho chắc" tạo 200 partition cho topic 50 event/giây, nhân với 300 topic CDC. Nguy hiểm: mỗi partition là chi phí metadata, file handle, replication traffic; cluster chậm chạp vì hàng chục nghìn partition rỗng — và thứ tự cross-row vốn đã không có thì thêm partition không thêm giá trị. Sửa: bắt đầu 6–12, scale bằng topic mới có kế hoạch khi thật sự cần.

**7. Consumer không idempotent.** INSERT mù vào bảng đích. Hậu quả thật: sau một lần connector restart, bảng analytics có 2,3 triệu rows duplicate; báo cáo doanh thu sai 1,8% trong hai tuần trước khi ai đó nghi ngờ — sai *âm thầm* là loại sai đắt nhất. Sửa: upsert/version check/dedup như 13.1.

**8. Event payload quá lớn.** Bảng có cột TEXT/JSON hàng MB (nội dung bài viết, response log API) chảy nguyên vào CDC. Nguy hiểm: vượt `message.max.bytes` làm connector fail; hoặc lọt qua thì Kafka thành nơi lưu blob — throughput sụt, OOM connector (Case 4). Hậu quả thật: một cột `raw_payload` JSON 5 MB làm pipeline chết mỗi khi có record lớn — lỗi xuất hiện *thỉnh thoảng*, khó nhất để debug. Sửa: `column.exclude.list` cột blob; nếu downstream cần, gửi reference (id + fetch lại) thay vì value; kiểm soát kích thước row từ thiết kế schema.

**9. Replay toàn bộ khi không cần.** Cứ có sự cố nhỏ là reset offset về đầu / re-snapshot tất cả "cho sạch". Nguy hiểm: replay hàng tỷ event tốn hàng chục giờ, đập vào mọi sink, và nếu consumer có side-effect thì tái phát side-effect. Hậu quả thật: một lần "reset cho sạch" gửi lại 40 nghìn email thông báo. Sửa: incremental snapshot *đúng bảng cần vá*; side-effect consumer phải có guard theo thời gian/id; coi replay là công cụ phẫu thuật, không phải nút "sửa mọi thứ".

**10. Dùng CDC event thô làm public API contract giữa các team.** Cho 5 team khác consume thẳng topic CDC của bảng nội bộ. Nguy hiểm: schema bảng trở thành contract công khai — bạn mất quyền refactor database của chính mình. Hậu quả thật (chương 10): đổi một cột `status` thành hai cột mất 2 tháng phối hợp 5 team. Sửa: Outbox cho domain event; hoặc một transform layer chuyển CDC thô thành contract ổn định do bạn kiểm soát; CDC thô chỉ tiêu thụ nội bộ trong cùng ownership.

**11. Chạy snapshot giờ cao điểm.** Deploy connector mới lúc 10 giờ sáng thứ Hai, full snapshot bảng 800 triệu rows. Hậu quả thật: Case 3 — CPU database +50%, checkout p99 tăng gấp ba, sự cố nghiệp vụ do một thao tác hạ tầng hoàn toàn tránh được. Sửa: snapshot off-peak, incremental snapshot, throttle, và báo trước cho team vận hành database.

**12. Một Connect cluster khổng lồ cho mọi team, không cô lập.** 40 connector của 8 team trên một cluster chung, ai cũng có quyền deploy. Nguy hiểm: một connector OOM (Case 4) kéo rebalance toàn cluster (Case 13), pipeline của mọi team giật theo; một team đổi config sai ảnh hưởng láng giềng; không ai chịu trách nhiệm tổng thể. Hậu quả thật: sự cố "thứ Ba hàng tuần" đúng lịch deploy của một team không liên quan. Sửa: chia cluster theo domain/mức độ trọng yếu (luồng tiền tách khỏi luồng analytics), quyền deploy qua CI/CD có review, ownership rõ từng connector.

## 13.3. Khi nào KHÔNG nên dùng CDC

Phần này quan trọng vì nó ít được viết ra: người viết tài liệu CDC thường là người yêu CDC. Sau 20 năm, điều tôi tin là: **CDC là công cụ mạnh với chi phí vận hành cao và kéo dài — nó chỉ xứng đáng khi giá trị của "near real-time + đầy đủ + không xâm lấn" vượt chi phí đó.** Các tình huống nên nói không:

**Dữ liệu nhỏ, thay đổi thưa.** Bảng cấu hình vài nghìn rows, vài update mỗi phút. Polling `updated_at` mỗi 30 giây (kèm soft delete để không sót DELETE) là 20 dòng code, không thêm hạ tầng, không on-call. Dựng Debezium + Kafka + Connect + monitoring cho bài toán này là dùng cần cẩu nhấc hộp bút — và cần cẩu cũng cần bảo trì (toàn bộ chương 11).

**Batch hàng ngày đáp ứng nghiệp vụ.** Báo cáo BI T+1, đối tác nhận file mỗi đêm. Scheduled export/batch ETL đơn giản hơn một bậc độ lớn về vận hành, và như 12.5 đã chỉ ra — đôi khi còn rẻ hơn về compute. Đừng bán "real-time" cho nghiệp vụ không cần nó; hãy hỏi product owner: *quyết định nào sẽ khác đi nếu dữ liệu tươi hơn?* Không có câu trả lời cụ thể = không có nhu cầu thật.

**Không có yêu cầu near real-time, chỉ có "nghe nói real-time tốt".** Resume-driven architecture là có thật. Độ trễ là một chiều của yêu cầu, không phải một đức tính tự thân.

**Team chưa có năng lực vận hành Kafka.** Pipeline CDC là hệ phân tán stateful; nếu tổ chức chưa từng vận hành Kafka, bạn đang nhận cùng lúc hai khoản nợ học tập. Lựa chọn trưởng thành: managed Kafka + managed connector, hoặc managed CDC service trọn gói (Datastream, DMS, Fivetran, Airbyte Cloud...), hoặc lùi lại dùng polling/batch cho đến khi năng lực sẵn sàng. Chi phí một sự cố Case 1 trên production thường lớn hơn một năm phí managed service.

**Chi phí hạ tầng vượt giá trị.** Kafka 3 broker + Connect + Schema Registry + monitoring + 0,5–1 FTE vận hành — nếu toàn bộ giá trị mang lại là "dashboard tươi hơn 4 giờ" cho một dashboard ít người xem, phép tính không khép được.

**Giải pháp thay thế theo tình huống:**

| Tình huống | Giải pháp thay thế hợp lý |
|---|---|
| Sync bảng nhỏ, trễ phút là ổn | Polling `updated_at` + soft delete |
| Báo cáo T+1 | Scheduled export / batch ETL |
| Giảm tải đọc báo cáo khỏi primary | Read replica + query trực tiếp |
| Cần domain event giữa service, team làm chủ code | Application event qua Outbox — vẫn cần CDC hoặc polling publisher cho outbox, nhưng phạm vi nhỏ và contract sạch |
| Cần CDC nhưng thiếu năng lực vận hành | Managed CDC service |

**Ma trận quyết định:**

| Tiêu chí | Polling | Batch ETL | Outbox + CDC | CDC thuần |
|---|---|---|---|---|
| Độ trễ chấp nhận | phút | giờ/ngày | giây | giây |
| Cần bắt DELETE | kém (cần soft delete) | kém | tốt | tốt |
| Cần business intent | không có | không có | **có** | không có |
| Sửa code ứng dụng | không | không | **phải sửa** | không |
| Tải lên nguồn | cao khi bảng lớn | dồn cục theo lịch | rất thấp | rất thấp |
| Độ phức tạp vận hành | thấp | thấp | cao | cao |
| Yêu cầu năng lực Kafka | không | không | có | có |
| Phù hợp khi | dữ liệu nhỏ, trễ phút OK | T+1 đủ | domain event giữa service | data integration near real-time |

Cách đọc ma trận: đi từ trái sang phải — **chọn giải pháp đơn giản nhất còn thỏa mãn yêu cầu**, không phải giải pháp mạnh nhất.

## 13.4. Checklist trước khi adopt CDC

Trả lời trung thực trước khi viết dòng config đầu tiên:

1. Nghiệp vụ nào *thay đổi quyết định* nhờ dữ liệu tươi hơn? SLA độ tươi cụ thể là bao nhiêu?
2. Downstream đã chấp nhận eventual consistency chưa? Read-your-own-writes có kế hoạch xử lý chưa? (chương 10)
3. Consumer có idempotent không? Ai chứng minh?
4. Ai on-call cho pipeline? Họ đã đọc chương 11 chưa — cụ thể: họ biết làm gì khi slot lag alert nổ lúc 2 giờ sáng?
5. Bộ ba alert tối thiểu (slot lag, slot inactive, task FAILED) và dashboard 4 tầng có trong định nghĩa done của dự án không?
6. WAL/binlog retention, `max_slot_wal_keep_size`, heartbeat đã cấu hình chưa?
7. Offset/schema history/config có kế hoạch backup chưa? DR drill lịch khi nào?
8. Schema change process có mục "ảnh hưởng CDC" chưa? Compatibility mode chọn gì, vì sao?
9. PII đã được liệt kê và masking tại connector chưa?
10. Đã tính phương án đơn giản hơn (polling/batch/managed) và có lý do *bằng số* để loại chúng chưa?

Thiếu câu trả lời cho quá ba mục — dự án chưa sẵn sàng, dù demo đã chạy đẹp.

## Tóm tắt chương

- Best practices cốt lõi: topic naming có môi trường và version; partition key theo PK và không tăng partition giữa chừng; incremental snapshot cho bảng lớn; backup offset/schema history/config như tài sản trọng yếu; Schema Registry với BACKWARD compatibility; consumer idempotent ba lớp (upsert, version check, dedup store); retry phân loại + DLQ có người đọc; heartbeat và monitor lag từ ngày đầu; masking PII tại connector.
- Anti-pattern nguy hiểm nhất không phải lỗi kỹ thuật phức tạp mà là lỗi kỷ luật: dual write, không monitor, không backup offset, consumer không idempotent, và dùng CDC event thô làm public contract — tất cả đều rẻ để phòng, rất đắt để chữa.
- CDC không phải mặc định đúng: dữ liệu nhỏ, nhu cầu T+1, team chưa vận hành được Kafka, hay giá trị không khép được với chi phí — polling, batch ETL, read replica, application event hoặc managed service là những câu trả lời trưởng thành hơn. Chọn giải pháp đơn giản nhất còn thỏa mãn yêu cầu.

## Lời kết toàn tài liệu

Nếu phải nén mười ba chương thành một đoạn, tôi sẽ viết thế này:

**CDC là phương pháp đồng bộ dữ liệu dựa trên transaction log** — nó đọc sự thật từ nơi trung thực nhất của database, nhờ đó đầy đủ (không sót DELETE, không sót thay đổi ngoài ứng dụng), gần thời gian thực, và gần như không tốn thêm gì của hệ thống nguồn. Đó là phần "mạnh". Phần "có giá" là: một pipeline stateful nhiều mắt xích với ngữ nghĩa at-least-once và eventual consistency, đòi hỏi kỷ luật vận hành thật — monitoring từ ngày đầu, consumer idempotent, backup trạng thái, quy trình schema change, và những người on-call hiểu vì sao một replication slot bị bỏ quên có thể làm database production ngừng ghi.

CDC không phải Event Sourcing, không phải message bus vạn năng, không phải đường tắt đến "real-time company". Nó là một công cụ hạ tầng xuất sắc cho đúng bài toán: data integration cần độ tươi tính bằng giây, ở tổ chức đủ năng lực trả chi phí vận hành của nó. Dùng đúng chỗ, nó âm thầm gánh cả hệ sinh thái dữ liệu của bạn trong nhiều năm; dùng sai chỗ, nó là hệ phân tán phức tạp nhất bạn từng phải nuôi để giải một bài toán mà cron job làm được.

Kỹ năng cao nhất của một architect không phải là dựng được pipeline CDC — chương 1 đến 9 dạy được điều đó. Kỹ năng cao nhất là nhìn một yêu cầu nghiệp vụ và trả lời đúng câu hỏi: *bài toán này xứng đáng với CDC, hay chỉ cần một câu SELECT chạy mỗi phút?* Tài liệu này hoàn thành nhiệm vụ nếu nó giúp bạn trả lời câu hỏi đó một cách trung thực — và khi câu trả lời là "CDC", giúp bạn vận hành nó qua đêm thứ Sáu mà vẫn ngủ ngon.

Chúc pipeline của bạn lag thấp và alert im lặng — vì những lý do đúng.
