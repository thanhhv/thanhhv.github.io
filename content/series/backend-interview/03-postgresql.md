+++
title = "Bài 3 — PostgreSQL"
date = "2026-07-03T10:00:00+07:00"
draft = false
tags = ["backend", "database", "interview"]
series = ["Backend Interview"]
+++

---

## Câu 1 — [Fundamental → Senior] Tại sao PostgreSQL cần MVCC? Nó hoạt động thế nào và cái giá phải trả là gì?

### 1. Câu hỏi
"Tại sao PostgreSQL cần MVCC? Trình bày cơ chế và hệ quả của nó."

### 2. Interviewer muốn kiểm tra điều gì?
- Hiểu bài toán gốc: concurrency control — reader và writer tranh chấp thế nào.
- Hiểu cơ chế bên trong (tuple version, xmin/xmax, snapshot) chứ không chỉ khẩu hiệu "reader không chặn writer".
- Kinh nghiệm production với hệ quả: bloat, vacuum, wraparound.

### 3. Câu trả lời ngắn gọn (30 giây)
"Không có MVCC, để đọc dữ liệu nhất quán, reader phải lock chặn writer và ngược lại — throughput sụp đổ khi mixed workload. MVCC giải quyết bằng cách: UPDATE/DELETE không ghi đè mà tạo **phiên bản tuple mới**; mỗi tuple mang xmin/xmax (transaction tạo/xóa nó); mỗi transaction cầm một snapshot và chỉ 'nhìn thấy' các version phù hợp snapshot của mình. Kết quả: reader không bao giờ chặn writer. Cái giá: các version chết tích tụ thành bloat, cần VACUUM dọn dẹp, và transaction ID 32-bit cần chống wraparound — đây là nguồn của phần lớn sự cố vận hành Postgres."

### 4. Câu trả lời Senior Level (3–5 phút)
**Problem:** một hệ CSDL phải cho nhiều transaction chạy đồng thời mà mỗi transaction thấy dữ liệu nhất quán. Cách ngây thơ: two-phase locking cho cả đọc — reader giữ shared lock, writer cần exclusive lock → báo cáo chạy 10 phút chặn mọi UPDATE trong 10 phút.

**How:**
- Mỗi row version (tuple) có header: `xmin` (XID của transaction tạo nó), `xmax` (XID của transaction xóa/update nó, nếu có).
- Transaction bắt đầu → lấy **snapshot**: danh sách transaction đang chạy tại thời điểm đó. Quy tắc nhìn thấy: tuple visible nếu xmin đã commit trước snapshot và xmax chưa commit/không tồn tại đối với snapshot.
- `UPDATE` = tạo tuple mới + đặt xmax lên tuple cũ. Cả hai cùng tồn tại vật lý trong heap cho đến khi VACUUM xác nhận không snapshot nào còn cần bản cũ.
- Trạng thái commit của XID tra qua CLOG; **hint bits** trên tuple cache kết quả tra cứu — lý do "lần đọc đầu sau bulk insert chậm và gây ghi" (một bất ngờ production kinh điển).
- Writer vs writer trên cùng row vẫn phải chờ nhau (row lock) — MVCC chỉ giải phóng reader.

**Trade-off:** không gian (nhiều version), chi phí dọn dẹp nền (VACUUM), index phình theo (mỗi version heap cần entry index, trừ khi HOT update), XID 32-bit phải freeze định kỳ chống wraparound.

**Production:** với bảng update-heavy, tôi theo dõi dead tuple ratio và tuning autovacuum riêng cho bảng đó; thiết kế schema để tận dụng **HOT update** (không index cột hay thay đổi); và canh `xact` age để không bao giờ tới gần wraparound.

### 5. Giải thích bản chất
First principles: mâu thuẫn gốc là **đọc nhất quán** đòi hỏi dữ liệu "đứng yên", còn **ghi** đòi hỏi dữ liệu thay đổi. Chỉ có 3 lối thoát: (1) bắt một bên chờ (locking), (2) cho reader xem bản cũ (multi-version), (3) cho reader xem dữ liệu không nhất quán (dirty read). MVCC là lựa chọn 2 — đổi **không gian** lấy **concurrency**, một biến thể của nguyên lý phổ quát "space–time trade-off". **Nếu thiết kế khác:** Oracle/MySQL InnoDB cũng multi-version nhưng giữ version cũ ở **undo log** riêng thay vì trong heap — ưu điểm: bảng chính không bloat, không cần vacuum; nhược: rollback lâu, long-running query có thể chết vì undo bị tái sử dụng ("snapshot too old"), read version cũ phải lần chuỗi undo. Postgres chọn version-in-heap: đọc version cũ rẻ, rollback tức thời (chỉ đánh dấu abort), nhưng trả giá bằng vacuum. Không có lựa chọn đúng — chỉ có phân bổ chi phí khác nhau.

### 6. Trade-off
- **Concurrency vs Space:** đọc không chặn ghi ↔ heap chứa cả xác chết.
- **Rollback rẻ vs Cleanup đắt:** abort transaction là O(1); dọn hậu quả là việc của vacuum sau này.
- **Update-heavy workload:** Postgres MVCC đau nhất ở đây (mỗi update = insert mới + dead tuple) — một lý do Uber từng chuyển sang MySQL (write amplification qua index).
- **Long transaction:** một transaction mở 6 giờ ghim snapshot → vacuum không dọn được gì mới hơn nó → bloat toàn cluster. MVCC biến "transaction dài" thành kẻ thù chung.

### 7. Ví dụ Production
Sự cố kinh điển ngành: một analytics query/stuck connection `idle in transaction` suốt cuối tuần → autovacuum chạy nhưng không thu hồi được gì → bảng orders phình từ 50GB lên 200GB, query plan xấu đi vì thống kê lệch, latency toàn hệ thống tăng. Xử lý: kill backend cũ, autovacuum dọn dần, nhưng **kích thước file không tự co lại** — phải `pg_repack` (online) hoặc chịu. Phòng ngừa: `idle_in_transaction_session_timeout`, alert trên `pg_stat_activity` xact age và dead tuple ratio.

### 8. Những câu trả lời chưa đủ tốt
- "MVCC giúp reader không chặn writer." → Đây là **kết quả**, chưa phải câu trả lời. Cơ chế nào tạo ra kết quả đó, và cái giá?
- "Postgres tốt hơn MySQL vì có MVCC." → InnoDB cũng MVCC. So sánh đúng phải là **version ở heap vs ở undo log** và hệ quả của mỗi hướng.

### 9. Sai lầm phổ biến của ứng viên
- Nghĩ DELETE giải phóng dung lượng ngay (thực ra chỉ đặt xmax; không gian chờ vacuum, file không co).
- Không biết UPDATE = INSERT mới + dead tuple, kể cả khi chỉ đổi 1 cột.
- Không biết long-running transaction chặn vacuum toàn cluster — kể cả transaction **chỉ đọc** (nó ghim snapshot).
- Nghĩ VACUUM FULL là giải pháp thường dùng (nó lấy exclusive lock, ghi lại cả bảng — gần như downtime).
- Chưa từng nghe transaction ID wraparound — với tôi đây là ranh giới giữa "đã vận hành Postgres thật" và "mới dùng Postgres".

### 10. Follow-up Questions
- Transaction ID wraparound là gì? Điều gì xảy ra khi tới ngưỡng? Freeze hoạt động thế nào?
- HOT update là gì, điều kiện nào để có HOT, thiết kế schema tận dụng nó ra sao?
- Tại sao Postgres index scan đôi khi vẫn phải vào heap kiểm tra visibility? Visibility map và index-only scan liên hệ gì?
- So sánh cách InnoDB làm MVCC. Workload nào Postgres lợi hơn, workload nào thiệt hơn?
- Nếu thiết kế lại từ đầu, có cách nào tránh vacuum không? (zheap từng thử; các fork như OrioleDB — hiểu vì sao khó.)

### 11. Liên hệ với Production
Mọi công ty chạy Postgres quy mô lớn đều có "văn hóa sợ long transaction" và monitoring wraparound (vụ Sentry và Mailchimp gần-downtime vì wraparound là case study công khai nổi tiếng). Vấn đề nghiêm trọng khi: workload update-heavy trên bảng lớn, ORM mở transaction rồi gọi API ngoài bên trong nó, hoặc connection pool giữ `idle in transaction`. Dấu hiệu cần hành động: `n_dead_tup` cao dai dẳng, `pg_stat_activity` có xact mở > vài phút, tuổi XID của database tăng gần ngưỡng, kích thước bảng tăng nhanh hơn dữ liệu thật.

---

## Câu 2 — [Intermediate → Senior] WAL là gì? Nó là nền tảng của những tính năng nào?

### 1. Câu hỏi
"Write-Ahead Log hoạt động thế nào? Tại sao gần như mọi database đều có một dạng WAL?"

### 2. Interviewer muốn kiểm tra điều gì?
- Hiểu nguyên lý durability nền tảng — WAL xuất hiện trong Postgres, MySQL, Kafka, thậm chí filesystem.
- Nối được WAL với replication, PITR, crash recovery — thấy được "một cơ chế, nhiều tính năng".
- Hiểu trade-off của các mức `synchronous_commit`.

### 3. Câu trả lời ngắn gọn (30 giây)
"Nguyên tắc WAL: **ghi log thay đổi và fsync log trước khi sửa data page**. Commit chỉ cần một lần ghi tuần tự vào log — nhanh hơn nhiều so với ghi ngẫu nhiên vào data files; data page sửa trong memory và được checkpoint ghi xuống sau. Crash thì replay WAL từ checkpoint gần nhất là khôi phục được. Vì WAL là dòng thay đổi tuyến tính hoàn chỉnh, nó 'miễn phí' sinh ra: streaming replication (ship WAL sang replica), PITR (backup + replay tới thời điểm bất kỳ), và logical decoding (CDC ra Kafka/Debezium)."

### 4. Câu trả lời Senior Level (3–5 phút)
**Problem:** durability đòi hỏi dữ liệu chạm đĩa trước khi báo commit thành công; nhưng một transaction có thể sửa nhiều page rải rác — ghi ngẫu nhiên + fsync từng page thì chậm không chấp nhận được, và crash giữa chừng để lại trạng thái nửa vời.

**How:**
- Mọi thay đổi được mô tả thành WAL record, append vào WAL buffer, và **fsync trước** thời điểm báo commit (write-ahead). Data page bẩn nằm ở shared_buffers, được flusher/checkpoint ghi xuống dần.
- **Checkpoint:** mốc mà mọi thay đổi trước đó đã nằm an toàn trong data file → WAL trước đó có thể tái chế. Recovery = từ checkpoint cuối, replay WAL. Checkpoint càng thưa → recovery càng lâu nhưng ít I/O; `checkpoint_timeout`, `max_wal_size` điều chỉnh cân bằng này.
- **Full page writes:** sau mỗi checkpoint, lần sửa đầu tiên của mỗi page ghi **cả page** vào WAL — chống "torn page" (page 8KB ghi dở khi mất điện vì đĩa chỉ atomic theo sector 512B/4KB). Đây là lý do WAL volume tăng vọt ngay sau checkpoint.
- **`synchronous_commit`:** `on` (chờ fsync — an toàn, chậm hơn), `off` (báo commit trước khi fsync — nhanh, có thể mất vài trăm ms giao dịch **đã báo thành công** khi crash, nhưng không bao giờ corrupt), các mức `remote_write/remote_apply` khi có synchronous replica.
- **Nền tảng của:** crash recovery, physical streaming replication, PITR, logical replication/CDC. Một cơ chế — bốn tính năng lớn.

**Production:** WAL volume là chỉ số phải theo dõi: đầy đĩa WAL là database dừng ghi. Replication slot bị bỏ rơi (consumer CDC chết) giữ WAL không cho tái chế → đầy đĩa — sự cố tôi gặp không dưới ba lần ở các công ty khác nhau.

### 5. Giải thích bản chất
First principles: đĩa (kể cả SSD) ghi **tuần tự** nhanh hơn ghi **ngẫu nhiên** nhiều lần, và fsync đắt. Vậy muốn durability rẻ: gom mọi thay đổi thành một dòng append tuần tự duy nhất, fsync một chỗ. Đây là ý tưởng sâu sắc đến mức xuất hiện khắp nơi: redo log của InnoDB, journal của ext4, memtable+WAL của RocksDB, commit log của Cassandra, và **chính Kafka về bản chất là một WAL được nâng cấp thành sản phẩm**. Event sourcing là WAL ở tầng ứng dụng. **Nếu thiết kế khác:** không WAL, mỗi commit phải fsync mọi data page nó chạm (chậm thảm) hoặc chấp nhận mất/hỏng dữ liệu khi crash. Nói được "log là nguồn sự thật, bảng chỉ là cache của log" là điểm nhận diện tư duy Staff.

### 6. Trade-off
- **Write amplification:** dữ liệu ghi ≥2 lần (WAL + data file), cộng full page writes. Đổi lấy: commit nhanh, recovery đúng.
- **`synchronous_commit off`:** throughput tăng mạnh — mất tối đa `wal_writer_delay`×3 dữ liệu đã ack khi crash. Hợp lý cho log/metrics, cấm kỵ cho payment.
- **Checkpoint dày vs thưa:** recovery nhanh + WAL nhỏ ↔ I/O spike và full-page-write storm mỗi checkpoint.
- **Synchronous replication:** không mất dữ liệu khi primary chết ↔ mỗi commit chờ network round-trip; replica chết làm primary treo ghi (phải có quorum/timeout).

### 7. Ví dụ Production
Sự cố: nửa đêm database ngừng nhận ghi, đĩa WAL đầy 100%. Nguyên nhân: pipeline Debezium CDC bị tắt hai tuần trước nhưng **replication slot không bị xóa** — Postgres trung thành giữ toàn bộ WAL chờ consumer quay lại. Xử lý khẩn: drop slot, WAL được tái chế. Phòng ngừa: alert trên `pg_replication_slots` (restart_lsn lag) và `max_slot_wal_keep_size` (Postgres 13+). Một ví dụ khác về tối ưu: hệ thống ingest nặng ghi, đổi `synchronous_commit=off` cho bảng event (chấp nhận mất <1s khi crash) → throughput ghi tăng ~3 lần, không đổi một dòng code.

### 8. Những câu trả lời chưa đủ tốt
- "WAL để khôi phục khi crash." → Đúng 1/4. Vai trò nền cho replication/PITR/CDC và **tại sao ghi log lại nhanh hơn ghi thẳng** mới là phần ăn điểm.
- "Bật fsync cho an toàn." → `fsync=off` và `synchronous_commit=off` khác nhau căn bản (một cái risk corruption, một cái chỉ risk mất giao dịch cuối) — nhầm lẫn này bị trừ điểm nặng.

### 9. Sai lầm phổ biến của ứng viên
- Nghĩ commit là lúc data file được cập nhật (thực ra chỉ WAL chạm đĩa; data page có thể vài phút sau).
- Không biết replication slot giữ WAL → đầy đĩa (lỗi vận hành phổ biến nhất liên quan WAL).
- Không giải thích được tại sao WAL tăng đột biến sau checkpoint (full page writes).
- Nhầm logical replication và physical replication — cái nào replay WAL byte-level, cái nào decode thành thay đổi logic.
- Nghĩ PITR chỉ cần WAL — cần base backup + chuỗi WAL liên tục.

### 10. Follow-up Questions
- Điều gì xảy ra nếu đĩa WAL và đĩa data tách riêng, một trong hai chết?
- Group commit là gì? `commit_delay` đánh đổi gì?
- Tại sao Kafka và WAL của database giống nhau về bản chất? Event sourcing liên hệ gì?
- Thiết kế PITR: RPO 5 phút, RTO 30 phút cho database 2TB — backup và archive WAL thế nào?
- Torn page là gì? Nếu filesystem/storage đảm bảo atomic write 8KB (một số cloud disk), tắt full_page_writes được không, lợi bao nhiêu?

### 11. Liên hệ với Production
Mọi setup Postgres nghiêm túc: WAL archiving lên object storage (wal-g/pgBackRest), giám sát WAL generation rate và slot lag. Vấn đề nghiêm trọng khi: ghi tăng đột biến (migration lớn, backfill) làm checkpoint dồn dập; đĩa WAL chung với data cạnh tranh I/O; CDC consumer không có ai own. Dấu hiệu cần hành động: `checkpoints_req` (checkpoint bị ép do max_wal_size) tăng thay vì `checkpoints_timed`, WAL disk usage tăng bất thường, latency ghi có spike chu kỳ trùng checkpoint.

---

## Câu 3 — [Senior] Isolation Levels trong Postgres: khác gì chuẩn SQL, và chọn thế nào cho đúng?

### 1. Câu hỏi
"Postgres có những isolation level nào, mỗi level chống được anomaly gì? Tại sao Read Committed là mặc định và khi nào nó **không đủ**?"

### 2. Interviewer muốn kiểm tra điều gì?
- Đây là câu tôi dùng để tách Senior thật khỏi Senior danh nghĩa: hầu hết dev chưa bao giờ đổi isolation level và không biết mình đang gặp anomaly gì.
- Hiểu anomaly cụ thể (lost update, write skew) bằng ví dụ nghiệp vụ, không phải học thuộc bảng.
- Biết Serializable của Postgres là SSI — optimistic, cần retry.

### 3. Câu trả lời ngắn gọn (30 giây)
"Postgres có 3 mức thực tế: Read Committed (mặc định — mỗi **câu lệnh** thấy snapshot mới), Repeatable Read (mỗi **transaction** một snapshot — thực chất là Snapshot Isolation), và Serializable (SSI — như tuần tự tuyệt đối, phát hiện xung đột và abort). Read Committed đủ cho đa số CRUD nhưng **không** chống lost update dạng read-modify-write và write skew. Với logic kiểu 'đọc rồi quyết định rồi ghi' — trừ tồn kho, kiểm tra hạn mức — phải chọn: khóa bi quan (`SELECT FOR UPDATE`), update nguyên tử (`SET x = x - 1` + điều kiện), hoặc Serializable kèm retry loop."

### 4. Câu trả lời Senior Level (3–5 phút)
**Problem:** isolation là mua sự đúng đắn bằng concurrency. Chuẩn SQL định nghĩa 4 mức qua 3 anomaly (dirty/non-repeatable/phantom read) — nhưng chuẩn này viết cho locking DB thập niên 90 và **bỏ sót các anomaly quan trọng nhất của MVCC**: lost update và write skew.

**Các mức trong Postgres:**
- **Read Uncommitted** = Read Committed (MVCC không bao giờ cho dirty read — chọn nó là lộ ngay chưa dùng Postgres).
- **Read Committed:** snapshot mới cho từng statement. Hai statement trong cùng transaction có thể thấy dữ liệu khác nhau. UPDATE gặp row bị transaction khác sửa → chờ, rồi **re-check điều kiện WHERE trên bản mới** và tiếp tục — hành vi tinh tế, đủ tốt cho single-statement update, nhưng vô dụng khi bạn đã SELECT ra biến ở statement trước.
- **Repeatable Read (Snapshot Isolation):** một snapshot cho cả transaction. Gặp write-write conflict → lỗi serialization, app phải retry. Chống lost update, **không chống write skew**.
- **Serializable (SSI):** theo dõi read/write dependency, phát hiện chu trình nguy hiểm → abort một bên. Optimistic: không thêm lock, trả giá bằng tỷ lệ abort và bắt buộc retry loop ở app.

**Write skew — ví dụ ăn điểm:** bệnh viện yêu cầu ≥1 bác sĩ trực. Hai bác sĩ cùng lúc: đọc thấy "2 người trực" → cùng rút tên. Hai transaction chạm **hai row khác nhau**, không write-write conflict → Snapshot Isolation cho qua cả hai → 0 người trực. Chỉ Serializable (hoặc lock/constraint vật lý hóa bất biến) bắt được, vì bất biến nằm trên **tập** dữ liệu, không trên một row.

**Chọn thế nào:** mặc định Read Committed + viết update dạng nguyên tử + unique/check constraint làm lưới an toàn; `FOR UPDATE` cho các luồng read-modify-write nóng; Serializable cho tiền/tồn kho nếu team chấp nhận kỷ luật retry. Điều tối kỵ: nghĩ rằng "transaction" tự động nghĩa là "an toàn".

### 5. Giải thích bản chất
Bản chất isolation: các transaction đan xen; kết quả "đúng" nghĩa là tương đương **một** thứ tự tuần tự nào đó (serializability). Mọi mức thấp hơn là bản chiết khấu: cho phép một số đan xen không tương đương tuần tự để đổi lấy concurrency. Snapshot Isolation nhìn có vẻ hoàn hảo — mỗi người một vũ trụ nhất quán — nhưng lỗ hổng nằm ở chỗ: **quyết định của bạn dựa trên những gì bạn đọc, mà vũ trụ bạn đọc đã cũ khi bạn ghi**. Ghi vào chỗ khác nhau nên hệ thống không thấy xung đột — write skew. SSI của Postgres (dựa trên nghiên cứu của Cahill 2008) phát hiện điều này bằng cách theo dõi dependency graph rw-antidependency và abort khi xuất hiện cấu trúc nguy hiểm — một trong số ít hiện thực serializable-không-cần-lock trong DB thương mại.

### 6. Trade-off
- **Mức cao hơn = đúng hơn, chậm hơn, abort nhiều hơn.** Serializable dưới contention cao có thể abort 10–30% transaction → retry storm nếu app xử lý kém.
- **Pessimistic (`FOR UPDATE`) vs Optimistic (SSI/version check):** contention cao → pessimistic thắng (chờ rẻ hơn làm-lại); contention thấp → optimistic thắng (không ai phải chờ).
- **Đẩy bất biến vào constraint** (unique, check, exclusion) là lựa chọn thứ ba thường bị quên: rẻ, đúng tuyệt đối, nhưng không phải bất biến nào cũng biểu diễn được.

### 7. Ví dụ Production
Bug thật ở một hệ ví điện tử: `SELECT balance` → kiểm tra đủ tiền ở app → `UPDATE balance = <giá trị tính ở app>`. Hai request rút tiền đồng thời cùng đọc 100k, cùng qua kiểm tra, ghi lần lượt 50k rồi 30k → mất dấu một lần rút (lost update) → số dư âm ngoài đời thật. Ba tầng fix: `UPDATE ... SET balance = balance - 50000 WHERE id=? AND balance >= 50000` (nguyên tử + điều kiện), thêm `CHECK (balance >= 0)` làm lưới cuối, và hàng nóng nhất chuyển sang `FOR UPDATE`. Bài học: bug isolation không bao giờ xuất hiện ở QA (cần đúng interleaving) — nó xuất hiện vào ngày khuyến mãi.

### 8. Những câu trả lời chưa đủ tốt
- Đọc thuộc bảng 4 level × 3 anomaly của chuẩn SQL. → Bảng đó không chứa lost update và write skew — chính là hai thứ gây mất tiền. Học thuộc bảng mà không biết điều này là red flag.
- "Cứ dùng Serializable cho an toàn." → Ai viết retry loop? Tỷ lệ abort dưới tải đỉnh? Câu này đúng lý thuyết, ngây thơ vận hành.

### 9. Sai lầm phổ biến của ứng viên
- Tin rằng bọc trong transaction là tự động chống race — sai hoàn toàn ở Read Committed.
- Không phân biệt lost update (hai ghi cùng chỗ) và write skew (hai ghi khác chỗ, vỡ bất biến chung).
- Không biết Repeatable Read của Postgres mạnh hơn chuẩn (là Snapshot Isolation, chống cả phantom trong thực tế).
- Dùng Serializable nhưng không retry — biến lỗi serialization thành 500 trả cho user.
- Không biết `FOR UPDATE` có thể deadlock khi hai transaction khóa nhiều row theo thứ tự khác nhau (fix: sort id trước khi khóa).

### 10. Follow-up Questions
- Hai transaction Serializable bị abort — app retry thế nào cho đúng? (idempotency, backoff, giới hạn lần.)
- `SELECT FOR UPDATE SKIP LOCKED` dùng làm gì? (job queue trên Postgres — pattern rất đáng biết.)
- Advisory lock là gì, khi nào dùng thay row lock?
- Isolation trên replica đọc thì sao? Read-your-writes khi đọc từ replica giải quyết thế nào?
- Nếu chuyển sang DB phân tán (Cockroach/Spanner), khái niệm isolation thay đổi gì? (external consistency, clock.)

### 11. Liên hệ với Production
Các hệ fintech nghiêm túc hoặc dùng Serializable + retry kỷ luật, hoặc thiết kế mọi thao tác tiền thành update nguyên tử/ledger append-only (né hẳn read-modify-write). Vấn đề trở nên nghiêm trọng khi: có khuyến mãi/flash sale (contention đột biến làm lộ race ngủ yên), hoặc khi thêm read replica (user thấy dữ liệu cũ ngay sau khi ghi). Dấu hiệu cần rà soát: số dư/tồn kho thỉnh thoảng "lệch không giải thích được", deadlock log tăng, `serialization_failure` không có handler.

---

## Câu 4 — [Senior] Index hoạt động thế nào và tại sao Query Planner đôi khi không dùng index của bạn?

### 1. Câu hỏi
"Các loại index trong Postgres và cách chọn? Tại sao planner đôi khi bỏ qua index? Quy trình xử lý một query chậm của anh?"

### 2. Interviewer muốn kiểm tra điều gì?
- Kỹ năng thực dụng số một của backend: 80% sự cố hiệu năng DB là query/index.
- Hiểu planner dựa trên cost và statistics — biết "tại sao nó nghĩ vậy" thay vì đổ lỗi cho DB.
- Đọc được EXPLAIN ANALYZE thành thạo.

### 3. Câu trả lời ngắn gọn (30 giây)
"B-tree cho so sánh/range (mặc định, 95% trường hợp); GIN cho JSONB/array/full-text; GiST cho không gian/khoảng; BRIN cho bảng khổng lồ có dữ liệu tương quan vật lý theo thứ tự (log theo thời gian); Hash ít khi đáng dùng. Planner chọn plan theo **cost ước lượng từ statistics**, không phải theo 'có index thì dùng': nó bỏ index khi ước lượng phải lấy phần lớn bảng (seq scan rẻ hơn do random I/O đắt), khi statistics sai, khi biểu thức không khớp index (hàm bọc cột, sai kiểu, sai collation), hoặc khi index không 'sargable'. Quy trình của tôi: `pg_stat_statements` tìm query tốn nhất → `EXPLAIN (ANALYZE, BUFFERS)` → so sánh **estimated vs actual rows** — lệch lớn là gốc rễ phổ biến nhất."

### 4. Câu trả lời Senior Level (3–5 phút)
**Why B-tree:** cây cân bằng fan-out lớn (hàng trăm key/page 8KB) → bảng tỷ row chỉ sâu 4–5 mức; leaf liên kết nhau nên range scan tuần tự. Index là bản đồ **có thứ tự** trỏ vào heap **không thứ tự** — từ đây suy ra mọi tính chất.

**Planner nghĩ gì:** với mỗi cách thực thi (seq scan, index scan, bitmap scan; nested loop/hash/merge join), tính cost từ: số row ước lượng (từ histogram/MCV trong `pg_statistic`, cập nhật bởi ANALYZE), chi phí trang (`random_page_cost` vs `seq_page_cost`), memory (`work_mem`). Chọn min cost. Suy ra các lý do "có index mà không dùng":
1. **Selectivity thấp:** lấy 30% bảng qua index = triệu lần random I/O; seq scan đọc tuần tự rẻ hơn. Planner **đúng** khi bỏ index — nhiều dev không tin điều này.
2. **Statistics cũ/sai:** sau bulk load, hoặc cột có phân phối lệch, hoặc **tương quan giữa các cột** mà planner mặc định coi là độc lập (multiply selectivity) → sai hàng nghìn lần → chọn nested loop cho 2 triệu row. Fix: `ANALYZE`, `CREATE STATISTICS` (extended stats).
3. **Không sargable:** `WHERE lower(email) = ...` (cần expression index), `WHERE created_at::date = ...`, so sánh khác kiểu, LIKE '%x' đầu wildcard.
4. **`random_page_cost` mặc định 4** hợp với HDD; trên SSD/NVMe nên ~1.1 — không chỉnh, planner sợ index scan một cách vô lý.

**Các kỹ thuật Senior:** composite index đúng thứ tự cột (equality trước, range sau; cột ORDER BY cuối); covering index (`INCLUDE`) cho index-only scan; partial index (`WHERE status='pending'`) nhỏ và nóng; nhớ chi phí phía ghi — mỗi index là một cây phải cập nhật mỗi lần ghi, và làm mất HOT update.

**Quy trình query chậm:** (1) `pg_stat_statements` xếp theo total_time — sửa query chạy 10ms×1M lần/ngày giá trị hơn query 10s×2 lần; (2) `EXPLAIN (ANALYZE, BUFFERS)`; (3) tìm node lệch estimate/actual lớn nhất và node tốn buffers nhất; (4) sửa (index/viết lại/statistics); (5) đo lại và canh regression.

### 5. Giải thích bản chất
Planner là một hệ **ra quyết định dưới thông tin không hoàn hảo**: nó không biết dữ liệu thật, chỉ biết bản tóm tắt thống kê. Mọi hành vi "khó hiểu" của nó đều quy về: (a) mô hình cost không khớp phần cứng thật, hoặc (b) thống kê không khớp dữ liệu thật. Hiểu điều này đổi cách debug: thay vì "ép nó dùng index" (hint — Postgres cố tình không có), hãy **sửa thông tin đầu vào của nó**. Nếu thiết kế khác — planner rule-based (Oracle cổ) thì dự đoán được nhưng ngu với dữ liệu lệch; planner học thích nghi (một số DB mới) thì thông minh hơn nhưng khó dự đoán/tái lập. Cost-based + statistics là điểm cân bằng đã thắng trong 30 năm.

### 6. Trade-off
- **Mỗi index:** đọc nhanh hơn ↔ ghi chậm hơn (write amplification), tốn đĩa, vacuum lâu hơn, phá HOT update. Bảng 15 index là red flag.
- **Index-only scan:** né heap ↔ phụ thuộc visibility map — bảng ghi nhiều thì hiệu quả giảm.
- **Partial index:** nhỏ, nhanh ↔ chỉ phục vụ query khớp điều kiện; điều kiện đổi là index thành vô dụng âm thầm.
- **Thời gian planning vs chất lượng plan:** join nhiều bảng, không gian plan bùng nổ — planner dùng heuristic (GEQO) và có thể ra plan tệ.

### 7. Ví dụ Production
Case thật: query dashboard chạy 45s, có đủ index. `EXPLAIN ANALYZE`: planner estimate 200 rows, actual 1.8 triệu → chọn nested loop → thảm họa. Nguyên nhân: hai cột filter (`country`, `city`) tương quan mạnh, planner nhân độc lập selectivity. Fix: `CREATE STATISTICS st (dependencies) ON country, city FROM users; ANALYZE users;` → plan chuyển hash join → 300ms. Không thêm index nào cả. Đây là ví dụ tôi thích vì nó chứng minh: hiểu **tại sao planner nghĩ sai** giá trị hơn phản xạ "thêm index".

### 8. Những câu trả lời chưa đủ tốt
- "Query chậm thì thêm index." → Thêm vào cột nào, loại gì, thứ tự nào, và bằng chứng đâu? Phản xạ này tạo ra những bảng 20 index ghi chậm như rùa.
- "Index làm query nhanh hơn." → Tại sao nhanh? Và khi nào nó làm **chậm** hơn (planner đúng khi chọn seq scan; chi phí ghi)?

### 9. Sai lầm phổ biến của ứng viên
- Không biết `EXPLAIN ANALYZE` khác `EXPLAIN` (chạy thật vs ước lượng) — và không bao giờ nhìn cột estimate vs actual.
- Composite index (a,b) nhưng query chỉ lọc b → không dùng được; không hiểu leftmost prefix.
- Không biết index không chứa visibility → index scan vẫn chạm heap; không biết index-only scan cần visibility map sạch.
- Tạo index trên production bằng `CREATE INDEX` thường (khóa ghi cả bảng) thay vì `CONCURRENTLY`.
- Không biết ORDER BY + LIMIT có thể khiến planner chọn index "sai" một cách hợp lý (đi theo thứ tự index mong dừng sớm — và thảm họa khi filter hiếm).

### 10. Follow-up Questions
- Bitmap index scan là gì, khi nào planner chọn nó thay index scan thường?
- `work_mem` ảnh hưởng gì tới plan? Sort spill ra đĩa nhận biết thế nào trong EXPLAIN?
- Pagination bằng OFFSET 1000000 tệ vì sao? Keyset pagination hoạt động thế nào?
- Query nhanh ở staging, chậm ở production cùng dữ liệu — những khả năng nào? (stats, cache nóng/lạnh, cấu hình, plan khác do bind parameter — generic vs custom plan.)
- Khi nào partitioning giải quyết được thứ index không giải quyết được? (bulk delete theo thời gian = drop partition; partition pruning.)

### 11. Liên hệ với Production
Các team giỏi coi `pg_stat_statements` là dashboard hàng tuần và có CI check EXPLAIN cho query mới trên bảng lớn. Vấn đề nghiêm trọng khi: dữ liệu vượt RAM (cache hit tụt, mọi giả định "đang nhanh" sụp), hoặc plan **flip** đột ngột sau ANALYZE tự động (hôm qua 10ms hôm nay 30s — plan instability). Dấu hiệu cần hành động: buffer cache hit ratio giảm, seq scan trên bảng lớn tăng trong `pg_stat_user_tables`, query time phân phối hai đỉnh (dấu hiệu hai plan khác nhau cho cùng query).

---

## Câu 5 — [Staff → Principal] Thiết kế chiến lược replication, partitioning và scale PostgreSQL cho hệ thống lớn

### 1. Câu hỏi
"Hệ thống của anh chạy trên một Postgres primary bắt đầu quá tải. Trình bày lộ trình scale: replication, partitioning, và xa hơn. Trade-off ở mỗi bước?"

### 2. Interviewer muốn kiểm tra điều gì?
- Tư duy Staff/Principal: lộ trình theo giai đoạn, chọn đúng cấp độ phức tạp cho đúng quy mô, biết chi phí tổ chức chứ không chỉ kỹ thuật.
- Hiểu sâu replication (async/sync, lag, failover) và hệ quả tính đúng đắn (read-your-writes).
- Biết giới hạn thật của single-node Postgres — và sự thật là nó xa hơn đa số người nghĩ.

### 3. Câu trả lời ngắn gọn (30 giây)
"Thứ tự của tôi: (1) vắt kiệt single node — index, query, pooling, phần cứng: một máy NVMe hiện đại chịu được hàng chục nghìn TPS, đừng phân tán sớm; (2) read replica cho tải đọc — kèm xử lý replication lag và read-your-writes; (3) partitioning cho bảng lớn — chủ yếu vì quản trị (drop partition, vacuum) hơn là tốc độ query; (4) tách theo domain (mỗi service một DB) trước khi nghĩ tới (5) sharding — bước cuối cùng, đắt nhất về độ phức tạp, mất transaction/join xuyên shard. Mỗi bước tăng một bậc độ phức tạp vận hành, nên chỉ bước tiếp khi có số liệu chứng minh bước hiện tại đã cạn."

### 4. Câu trả lời Senior Level (3–5 phút)
**Bước 1 — Single node trước:** phần lớn "Postgres quá tải" là query tồi + thiếu pooling. PgBouncer (transaction mode) vì mỗi connection Postgres là một process ~vài MB — 2000 connection trần trụi tự nó là sự cố. Nâng máy: đây là scale rẻ nhất tính theo **tổng chi phí gồm cả người**.

**Bước 2 — Read replica (streaming replication):**
- Async mặc định: replica replay WAL, lag từ ms tới phút (khi replica chạy query dài xung đột replay — `hot_standby_feedback` đổi lấy bloat trên primary).
- Hệ quả đúng đắn: user vừa ghi xong đọc lại không thấy — **read-your-writes**. Giải pháp: đọc-sau-ghi đi primary theo session (sticky trong N giây), hoặc so LSN (`pg_last_wal_replay_lsn` ≥ LSN lúc ghi), hoặc chấp nhận và thiết kế UI quanh nó (optimistic UI).
- Sync replication khi không được phép mất giao dịch lúc failover — trả giá latency ghi + cần ≥2 replica với quorum để replica chết không treo primary.
- **Failover** là phần khó nhất: dùng Patroni/cloud managed; cẩn thận split-brain (fencing); failover async = có thể mất các giao dịch cuối — phải nói được điều này với business bằng ngôn ngữ RPO.

**Bước 3 — Partitioning (declarative):**
- Range theo thời gian cho time-series/log: retention = `DROP PARTITION` (tức thời, không bloat) thay vì DELETE triệu row (bloat + vacuum khổng lồ). Đây là lý do số một.
- Partition pruning giúp query có điều kiện partition key; nhưng query **thiếu** partition key phải quét mọi partition — chậm hơn không partition. Chọn key theo access pattern, không theo trực giác.
- Giới hạn: unique constraint phải chứa partition key; số partition quá lớn (hàng chục nghìn) làm planning chậm.

**Bước 4 — Functional split:** tách bảng/domain ít liên quan ra DB riêng (orders vs analytics vs sessions). Rẻ hơn sharding vì ranh giới rõ, và thường đi cùng ranh giới team/service.

**Bước 5 — Sharding:** Citus, hoặc app-level. Chọn shard key là quyết định **một chiều** gần như không thể đảo; mất cross-shard transaction/join; rebalance là dự án lớn. Chỉ tới đây khi write throughput hoặc working set thật sự vượt một máy — và cân nhắc: có nên chuyển hẳn sang hệ phân tán natively (Spanner/Cockroach/Aurora limitless) thay vì tự vận hành sharding.

### 5. Giải thích bản chất
Nguyên lý nền: **một hệ chia sẻ trạng thái có thứ tự ghi toàn cục thì dễ đúng; phân tán trạng thái là đánh đổi tính đúng lấy dung lượng**. Replication không scale ghi — mọi ghi vẫn qua primary (nó scale **đọc** và tăng availability). Muốn scale ghi phải phá vỡ "một thứ tự toàn cục" — tức sharding — và ngay lập tức bạn mất những gì single-node cho không: transaction ACID xuyên dữ liệu, join tùy ý, unique constraint toàn cục. Vì thế lộ trình đúng luôn là trì hoãn sharding lâu nhất có thể. Định luật thứ hai: **độ phức tạp vận hành tăng theo bậc thang, không tuyến tính** — mỗi bước (replica → partition → shard) thêm một lớp chế độ lỗi mới mà on-call phải thuộc.

### 6. Trade-off
- **Async replica:** đọc scale + HA rẻ ↔ lag, read-your-writes, mất dữ liệu khi failover (RPO > 0).
- **Sync replica:** RPO = 0 ↔ +1 RTT mọi commit; cấu hình quorum sai thì một replica chết làm cả hệ ngừng ghi.
- **Partitioning:** quản trị dữ liệu lớn dễ hơn hẳn ↔ mọi query phải nghĩ về partition key; ràng buộc unique yếu đi.
- **Sharding:** dung lượng vô hạn lý thuyết ↔ mất ACID xuyên shard, hot shard (celebrity problem), chi phí kỹ sư vĩnh viễn. 
- **Managed (RDS/Cloud SQL) vs tự vận hành:** đắt tiền máy hơn ↔ rẻ tiền người hơn — ở đa số công ty, người đắt hơn máy.

### 7. Ví dụ Production
Câu chuyện tôi hay kể: startup nọ shard sớm ở 100GB dữ liệu "để chuẩn bị scale" — hai năm sau, 3 kỹ sư giỏi nhất làm full-time việc vận hành shard, rebalance, và viết lại join thành N+1 query xuyên shard; trong khi đối thủ chạy một Postgres 4TB trên NVMe với 2 replica, đội đó ship tính năng. Ngược lại, case chính đáng: hệ ad-tech ghi 500k event/s — không máy đơn nào chịu — shard theo `advertiser_id` từ ngày đầu, và **mọi** truy vấn nghiệp vụ đều nằm gọn trong một advertiser → sharding gần như miễn phí về logic. Bài học Principal: quyết định scale đúng/sai không nằm ở công nghệ mà ở **access pattern có shard được tự nhiên hay không**.

### 8. Những câu trả lời chưa đủ tốt
- "Quá tải thì thêm replica, rồi shard." → Lộ trình máy móc. Quá tải **cái gì** — CPU đọc? I/O ghi? connection? Mỗi cái một thuốc khác nhau.
- "Postgres không scale, nên dùng NoSQL." → Instagram phục vụ hàng trăm triệu user trên Postgres sharded. Câu này lộ việc chưa từng vắt kiệt một RDBMS.

### 9. Sai lầm phổ biến của ứng viên
- Nghĩ read replica giúp scale **ghi**.
- Quên replication lag khi thiết kế luồng đọc-sau-ghi (bug "tôi vừa đăng bài mà không thấy đâu").
- Partition theo cột "cho đều" (hash user_id) trong khi 90% query lọc theo thời gian — pruning vô dụng.
- Chọn shard key theo cấu trúc dữ liệu thay vì theo access pattern; không có kế hoạch resharding.
- Không phân biệt HA (failover khi chết) và scale (thêm dung lượng) — replica phục vụ cả hai nhưng cấu hình khác nhau.

### 10. Follow-up Questions
- Failover tự động: làm sao tránh split-brain? Fencing/STONITH là gì? Patroni dùng gì làm DCS?
- Nếu scale lên 100 lần traffic hiện tại, kiến trúc này vỡ ở đâu trước tiên?
- Logical replication dùng cho zero-downtime major upgrade thế nào? Bẫy nào (sequence, DDL không replicate)?
- Multi-region: Postgres primary một region, user toàn cầu — latency ghi xử lý sao? Active-active có khả thi với Postgres không?
- Connection pooling transaction mode phá vỡ tính năng nào (prepared statements, advisory lock, `SET`)? Xử lý sao?

### 11. Liên hệ với Production
Instagram, GitLab, Zalando đều công bố kiến trúc Postgres quy mô lớn — điểm chung: trì hoãn sharding tối đa, đầu tư mạnh vào pooling + replica + partitioning + archiving. Vấn đề trở nên nghiêm trọng khi: write TPS chạm giới hạn một máy (WAL fsync bão hòa), working set vượt RAM nhiều lần, hoặc vacuum không đuổi kịp tốc độ ghi (dead tuple tăng không hồi phục). Dấu hiệu phải chuẩn bị bước tiếp theo: p99 ghi tăng dù đã tối ưu, autovacuum chạy 24/7 vẫn tụt hậu, thời gian recovery/backup vượt RTO cam kết.
