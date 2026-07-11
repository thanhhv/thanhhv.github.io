+++
title = "Bài 2 — Node.js Internals"
date = "2026-07-03T09:00:00+07:00"
draft = false
tags = ["backend", "nodejs", "interview"]
series = ["Backend Interview"]
+++

---

## Câu 1 — [Fundamental → Senior] Event Loop hoạt động thế nào? Tại sao Node single-thread mà vẫn xử lý được hàng chục nghìn kết nối?

### 1. Câu hỏi
"Trình bày các phase của event loop. Node.js 'single-threaded' — điều đó đúng đến mức nào?"

### 2. Interviewer muốn kiểm tra điều gì?
- Hiểu mô hình thực thi nền tảng — mọi bug hiệu năng Node đều quy về đây.
- Phân biệt được microtask vs macrotask — nguồn của các bug thứ tự thực thi.
- Biết giới hạn của mô hình: CPU-bound giết Node như thế nào.

### 3. Câu trả lời ngắn gọn (30 giây)
"Node chạy JavaScript trên một thread duy nhất, nhưng I/O được libuv ủy quyền cho kernel (epoll/kqueue) hoặc thread pool (mặc định 4 thread cho file I/O, DNS, crypto). Event loop quay qua các phase: timers → pending callbacks → poll (chờ I/O) → check (setImmediate) → close. Giữa mỗi callback, toàn bộ microtask queue (promise, `process.nextTick`) được xả sạch. Node xử lý được C10K vì thread không bao giờ chờ I/O — nó chỉ chạy callback. Điểm chết: một callback tính toán nặng chặn toàn bộ loop, mọi request khác đứng hình."

### 4. Câu trả lời Senior Level (3–5 phút)
**Problem:** mô hình thread-per-connection tốn RAM và context switch. Node chọn hướng ngược với Go: thay vì làm thread rẻ đi, loại bỏ luôn nhu cầu nhiều thread bằng non-blocking I/O + callback.

**How:**
- **libuv** trừu tượng hóa epoll (Linux), kqueue (macOS), IOCP (Windows). Network I/O hoàn toàn non-blocking trên thread chính. File I/O, DNS lookup (`getaddrinfo`), một số crypto — không có API non-blocking tốt ở kernel — đi qua **thread pool** (`UV_THREADPOOL_SIZE`, mặc định 4).
- **Các phase mỗi vòng lặp:** (1) `timers`: chạy callback của `setTimeout/setInterval` đã đến hạn; (2) `pending callbacks`; (3) `poll`: lấy I/O event mới, chạy callback của chúng, và **block chờ** ở đây nếu không có gì làm; (4) `check`: `setImmediate`; (5) `close callbacks`.
- **Microtask:** sau **mỗi callback** (không phải mỗi phase), V8 xả toàn bộ promise queue; `process.nextTick` còn ưu tiên cao hơn promise. Hệ quả: chuỗi promise/nextTick đệ quy có thể **bỏ đói** I/O — loop không bao giờ tới phase poll.
- "Single-threaded" chỉ đúng cho JS execution; process Node thực tế có nhiều thread: V8 GC threads, libuv pool, worker threads.

**Trade-off:** không data race trên state JS (không cần mutex) — đổi lại không tận dụng được multi-core cho JS, và một callback chậm là "head-of-line blocking" cho toàn process.

**Production:** hai chỉ số tôi luôn theo dõi: **event loop lag** (độ trễ giữa thời điểm hẹn và thời điểm chạy của một timer — đo bằng `monitorEventLoopDelay`) và **event loop utilization (ELU)**. Lag p99 > 50–100ms nghĩa là có code đồng bộ chặn loop.

### 5. Giải thích bản chất
First principles: một CPU core chỉ làm một việc một lúc, câu hỏi là **ai quyết định làm việc gì tiếp theo**. Kernel scheduling (thread) quyết định hộ bạn — trả giá context switch. Event loop để chính ứng dụng quyết định — rẻ, nhưng "hợp tác": mỗi callback phải tự giác trả quyền điều khiển nhanh. Đây chính là cooperative scheduling — mô hình mà các OS đã từ bỏ từ thập niên 90 vì một chương trình tồi treo cả máy... nhưng hợp lý cho server I/O-bound nơi ta kiểm soát toàn bộ code. **Nếu thiết kế khác:** nếu Node preemptive được JS, nó cần nhiều thread + lock cho mọi object → thành Java. Sự "yếu" của Node và sự "đơn giản" của nó là cùng một quyết định thiết kế.

### 6. Trade-off
- **Đơn giản về concurrency** (không lock, không race trên JS state) vs **mong manh về latency** (một `JSON.parse` payload 50MB chặn mọi request).
- **Tối ưu I/O-bound** vs **tệ cho CPU-bound** (image processing, nén, ML inference).
- **Thread pool 4** là nút cổ chai ẩn: 5 thao tác `fs` đồng thời → cái thứ 5 xếp hàng; bcrypt hash đồng loạt lúc đăng nhập cao điểm làm chậm cả DNS lookup (chung pool!).

### 7. Ví dụ Production
Sự cố thật: API bình thường p99 = 30ms, thỉnh thoảng vọt 8s toàn bộ endpoint. Nguyên nhân: một endpoint export CSV dùng `JSON.stringify` object 200MB — chặn loop 8s, mọi request khác (kể cả health check) chết theo → load balancer đánh dấu instance unhealthy → traffic dồn sang instance khác → domino. Fix: chuyển export sang worker thread + streaming. Bài học Senior: trong Node, **một endpoint tồi là vấn đề của mọi endpoint** — khác hẳn Go/Java.

### 8. Những câu trả lời chưa đủ tốt
- "Node non-blocking nên nhanh." → Non-blocking không làm một request nhanh hơn; nó làm **nhiều request chờ nhau ít hơn**. Và chỉ đúng khi code của bạn không chặn loop.
- "Node single-thread." → Thiếu: JS single-thread, còn libuv pool, GC thread, worker thread thì sao?

### 9. Sai lầm phổ biến của ứng viên
- Không biết `process.nextTick` chạy trước promise, và cả hai chạy trước macrotask kế tiếp.
- Nghĩ `setTimeout(fn, 0)` chạy ngay — nó chờ tới phase timers vòng sau, sau mọi microtask.
- Không biết DNS lookup mặc định dùng thread pool (nguồn nghẽn kinh điển khi gọi API ngoài nhiều).
- Dùng `sync` API (`fs.readFileSync`, `crypto.pbkdf2Sync`) trong request handler.
- Không biết `setImmediate` vs `setTimeout(0)` khác nhau thế nào tùy ngữ cảnh gọi (trong I/O callback, setImmediate luôn chạy trước).

### 10. Follow-up Questions
- Thứ tự output của đoạn code trộn `nextTick`, promise, `setTimeout`, `setImmediate`?
- Làm sao phát hiện cái gì đang chặn event loop trên production? (ELU, `--prof`, clinic.js, async hooks overhead?)
- Tăng `UV_THREADPOOL_SIZE` lên 128 có ổn không? Trade-off?
- Nếu phải xử lý CPU-bound trong Node, có những lựa chọn nào? (worker threads, tách service, native addon, WASM.)
- Event loop lag cao nhưng CPU thấp — nghĩa là gì? (GC? swap? thread pool nghẽn không gây lag loop — vậy còn gì?)

### 11. Liên hệ với Production
Netflix, PayPal dùng Node cho BFF/API layer — I/O-bound thuần túy, đúng sở trường. Vấn đề nghiêm trọng khi ứng dụng bắt đầu làm việc nặng CPU trong process web (báo cáo, export, nén ảnh). Dấu hiệu cần hành động: event loop lag p99 tăng, ELU > 0.7–0.8 kéo dài, health check timeout ngẫu nhiên dù hạ tầng khỏe — gần như chắc chắn có synchronous blocking.

---

## Câu 2 — [Intermediate → Senior] Promise và async/await hoạt động thế nào bên dưới? Các sai lầm nguy hiểm nhất?

### 1. Câu hỏi
"`async/await` thực chất là gì? Kể những lỗi async phổ biến gây sự cố production."

### 2. Interviewer muốn kiểm tra điều gì?
- Hiểu await là syntax trên promise + microtask, không phải "chờ" theo nghĩa blocking.
- Kinh nghiệm thực chiến với các lỗi: unhandled rejection, sequential-thay-vì-parallel, floating promise.
- Hiểu error handling trong async context.

### 3. Câu trả lời ngắn gọn (30 giây)
"`async function` trả về promise; `await` tách hàm thành các đoạn tiếp nối (continuation) được schedule vào microtask queue khi promise resolve — thread không hề bị chặn. Ba lỗi tôi gặp nhiều nhất trên production: await tuần tự những việc độc lập (nhân latency lên N lần); floating promise — quên await nên lỗi biến mất và thao tác đua với response; và unhandled rejection làm crash process từ Node 15. Nguyên tắc: mọi promise phải có người await hoặc catch, và việc độc lập phải `Promise.all`."

### 4. Câu trả lời Senior Level (3–5 phút)
**How bên dưới:**
- Promise là state machine: pending → fulfilled/rejected, kèm danh sách callback đăng ký qua `.then`. Callback **luôn** chạy async qua microtask — kể cả khi promise đã resolve sẵn (đảm bảo thứ tự nhất quán — quyết định thiết kế học từ "Zalgo problem" của callback era).
- `async/await` được V8 biến thành coroutine ngầm: mỗi `await` là một điểm suspend; phần còn lại của hàm thành continuation nối vào promise. Về ngữ nghĩa ~ generator + máy chạy promise.
- Từ đó suy ra: **giữa hai `await` là atomic** đối với JS khác (không bị chen ngang) — nhưng **qua một `await` thì mọi state chung có thể đã bị thay đổi** bởi request khác. Đây là "race condition kiểu Node": không có data race cấp bộ nhớ, nhưng đầy logic race (check-then-act qua await).

**Các lỗi production:**
1. **Tuần tự hóa vô ý:** `const a = await getA(); const b = await getB();` khi A, B độc lập → latency = A + B thay vì max(A, B). Fix: `Promise.all`. Chú ý bẫy `Promise.all` fail-fast: một cái reject là toàn bộ reject, các promise còn lại **vẫn chạy** nhưng kết quả bị vứt và lỗi của chúng thành unhandled → cân nhắc `allSettled`.
2. **Floating promise:** gọi hàm async không await/không catch. Lỗi im lặng, hoặc process crash vì unhandled rejection, hoặc thao tác ghi DB hoàn tất **sau khi** đã trả response (đau nhất khi có transaction).
3. **Await trong vòng lặp** `for` với hàng nghìn item → tuần tự; nhưng `Promise.all` cả nghìn cái → dội database. Cần concurrency limit (p-limit).
4. **try/catch không bọc đúng chỗ:** lỗi ném từ callback bên trong (ví dụ `setTimeout`) không bị catch bởi try/catch bao ngoài.

**Production:** bật `--unhandled-rejections=strict` từ sớm, lint rule `no-floating-promises` (TypeScript), và wrap mọi background job với error boundary + logging.

### 5. Giải thích bản chất
Await là **CPS transformation (continuation-passing style) do compiler làm hộ**. Trước đây ta viết continuation bằng tay (callback hell); promise chuẩn hóa giao thức "gọi lại tôi khi xong"; async/await để compiler viết callback hộ. Hiểu tầng này giải thích mọi hành vi "kỳ lạ": tại sao lỗi không bị catch (continuation chạy ở stack khác — stack trace cũng vì thế mà cụt), tại sao code sau await thấy state đã đổi (loop đã chạy việc khác giữa chừng). **Nếu thiết kế khác:** nếu await chặn thread thật thì Node chết ngay lập tức — toàn bộ giá trị của Node nằm ở chỗ thread luôn rảnh.

### 6. Trade-off
- **Ergonomics vs Transparency:** async/await che giấu điểm suspend — code trông tuần tự nhưng không tuần tự, dev quên rằng thế giới thay đổi qua mỗi await.
- **`Promise.all` vs tuần tự:** nhanh hơn nhưng dội tải đồng thời xuống downstream — latency đổi lấy áp lực tài nguyên.
- **Microtask ưu tiên tuyệt đối:** promise chain dài làm đói I/O — fairness đổi lấy tính đơn giản của mô hình.

### 7. Ví dụ Production
Sự cố: endpoint thanh toán thỉnh thoảng ghi log "success" nhưng DB không có bản ghi. Nguyên nhân: `saveAudit(order)` thiếu `await`, nằm sau `res.json()`; khi pod bị scale down ngay sau response, thao tác đang bay bị giết. Không lỗi, không log, mất dữ liệu âm thầm suốt 2 tháng. Bài học: floating promise nguy hiểm nhất khi nó **thường thành công** — chỉ fail lúc deploy/scale/crash, đúng lúc khó điều tra nhất.

### 8. Những câu trả lời chưa đủ tốt
- "Await làm code chạy đồng bộ." → Sai bản chất. Nó làm code **trông** tuần tự; thread vẫn chạy việc khác.
- "Dùng try/catch là bắt hết lỗi async." → Chỉ bắt lỗi trong chuỗi await của chính nó.

### 9. Sai lầm phổ biến của ứng viên
- Không biết `Promise.all` fail-fast và hệ quả với các promise còn lại.
- Tưởng `forEach(async ...)` chờ các callback — `forEach` không await gì cả, đây là floating promise hàng loạt.
- Không phân biệt `allSettled` / `race` / `any` và use case của từng cái.
- Nghĩ Node không có race condition — logic race qua await là nguồn bug lớn (double-spend khi hai request cùng check số dư rồi cùng trừ).
- Tạo promise thủ công (`new Promise`) bọc thứ đã là promise — anti-pattern "promise constructor".

### 10. Follow-up Questions
- Làm sao giới hạn concurrency khi phải gọi 10.000 request? Tự viết p-limit thế nào?
- Hai request cùng đọc số dư rồi cùng ghi — Node single-thread sao vẫn sai? Giải quyết ở tầng nào (app lock, DB transaction, unique constraint)?
- Unhandled rejection nên xử lý ở process level thế nào — log rồi tiếp tục hay crash rồi restart? Lập luận?
- AsyncLocalStorage hoạt động thế nào, dùng làm gì? (request context, trace ID — tương đương context của Go.)
- Stack trace trong async code bị cụt — tại sao, và async stack traces của V8 cứu được đến đâu?

### 11. Liên hệ với Production
Mọi codebase Node lớn đều có lint bắt buộc cho floating promise — bài học trả bằng máu. Vấn đề nghiêm trọng khi: hệ thống có ghi dữ liệu quan trọng (payment, order) mà thiếu kỷ luật await, hoặc migration callback→promise nửa vời tạo vùng lỗi rơi tự do. Dấu hiệu cần rà soát: log "unhandledRejection" xuất hiện, dữ liệu thiếu nhất quán không giải thích được, latency một endpoint = tổng latency các downstream (dấu hiệu tuần tự hóa vô ý).

---

## Câu 3 — [Senior] Stream và Backpressure — tại sao là kỹ năng bắt buộc với dữ liệu lớn?

### 1. Câu hỏi
"Stream trong Node giải quyết vấn đề gì? Backpressure là gì và điều gì xảy ra nếu bỏ qua nó?"

### 2. Interviewer muốn kiểm tra điều gì?
- Có từng xử lý dữ liệu lớn hơn RAM chưa, hay chỉ toàn `readFile` cả file vào bộ nhớ.
- Hiểu backpressure — khái niệm nền tảng xuất hiện lại ở message queue, TCP, reactive systems.
- Biết pipeline và error handling trong stream (nơi rất nhiều code production sai).

### 3. Câu trả lời ngắn gọn (30 giây)
"Stream xử lý dữ liệu theo chunk thay vì tải toàn bộ vào RAM — cho phép xử lý file 50GB với vài chục MB bộ nhớ, và giảm time-to-first-byte. Backpressure là cơ chế phản hồi khi consumer chậm hơn producer: `write()` trả về false nghĩa là buffer nội bộ vượt highWaterMark, producer phải dừng và chờ event `drain`. Bỏ qua tín hiệu này, dữ liệu chất đống trong buffer của process → memory phình → GC quằn quại → OOM. Luôn dùng `pipeline()` thay vì `.pipe()` để backpressure và error/cleanup được xử lý đúng."

### 4. Câu trả lời Senior Level (3–5 phút)
**Problem:** đọc file 10GB bằng `readFile` = 10GB RAM + user chờ toàn bộ trước byte đầu tiên. Tổng quát hơn: bất kỳ đâu producer và consumer lệch tốc độ, phải có cơ chế điều hòa.

**How:**
- 4 loại: Readable, Writable, Duplex, Transform. Dữ liệu chảy theo chunk (Buffer hoặc object mode).
- **highWaterMark** (mặc định 64KB cho byte stream) là ngưỡng buffer nội bộ. Writable: `write()` trả `false` khi vượt → producer nên dừng → nghe `'drain'` để tiếp tục. Readable ở flowing mode tự pause nguồn khi đích báo đầy (qua `.pipe`/`pipeline`).
- **`stream.pipeline()`** (hoặc `stream/promises`): nối chuỗi stream, propagate error đúng, destroy tất cả khi một mắt xích lỗi. `.pipe()` trần **không** làm điều này — lỗi ở đích không destroy nguồn → leak file descriptor.
- Backpressure xuyên hệ thống: socket TCP chậm → Writable của response đầy → pipeline pause → đọc file chậm lại. Toàn chuỗi tự điều tốc theo mắt xích chậm nhất — đây là vẻ đẹp của nó: **flow control end-to-end không cần code thủ công**.
- Web Streams (WHATWG) đang hợp nhất dần với Node streams — đáng nhắc để thể hiện cập nhật.

**Trade-off:** code stream khó viết/debug hơn buffer-toàn-bộ; object mode có overhead per-chunk; transform chậm là nút cổ chai toàn tuyến.

**Production:** use case điển hình: proxy file lớn, ETL CSV hàng trăm triệu dòng, nén gzip on-the-fly, đổ dữ liệu từ DB cursor ra S3. Nguyên tắc: dữ liệu có thể lớn không kiểm soát → bắt buộc stream; kèm limit kích thước và timeout để chống abuse.

### 5. Giải thích bản chất
Backpressure là hiện thân của một định luật hệ thống: **trong pipeline, throughput bền vững = throughput của mắt xích chậm nhất**; mọi cố gắng đẩy nhanh hơn chỉ tích tụ ở buffer nào đó — RAM của bạn, queue, hay swap — và buffer nào cũng hữu hạn. Ba lựa chọn khi producer nhanh hơn consumer: (1) buffer (tạm thời, sẽ tràn), (2) drop (mất dữ liệu), (3) **báo ngược cho producer chậm lại** — backpressure là lựa chọn 3. TCP flow control (receive window) là backpressure ở tầng 4; Kafka consumer lag là "thiếu backpressure" ở tầng ứng dụng; Reactive Streams spec sinh ra cũng vì đúng vấn đề này. Trả lời được sự tương đồng này là điểm cộng lớn ở mức Staff.

### 6. Trade-off
- **Memory vs Complexity:** stream tiết kiệm RAM tuyệt đối nhưng code phức tạp hơn hẳn (state machine, partial chunk, encoding cắt đôi ký tự multi-byte).
- **highWaterMark:** lớn → throughput mượt hơn, ít pause/resume — nhưng tốn RAM và che tín hiệu chậm; nhỏ → phản ứng nhanh nhưng ping-pong pause/resume tốn CPU.
- **Latency vs Throughput:** chunk nhỏ giảm time-to-first-byte nhưng tăng overhead per-chunk.

### 7. Ví dụ Production
Sự cố kinh điển: service export báo cáo, code `rows.forEach(r => res.write(csv(r)))` không kiểm tra giá trị trả về của `write`. Client mạng chậm tải file 2GB → toàn bộ 2GB tích trong buffer của response object → heap 3GB → OOMKill, giết luôn 200 request khác trên cùng pod. Fix: dùng `pipeline(dbCursorStream, csvTransform, res)` — RAM giữ ổn định ~50MB bất kể tốc độ client. Đây là câu chuyện tôi hay dùng để kiểm tra: nhiều ứng viên biết "stream tiết kiệm RAM" nhưng không biết chính `res.write` bỏ qua backpressure mới là quả bom.

### 8. Những câu trả lời chưa đủ tốt
- "Stream để xử lý file lớn." → Đúng 1/3. Còn time-to-first-byte và flow control?
- "Dùng pipe là xong." → `.pipe()` không xử lý lỗi đúng cách. Ai destroy stream nguồn khi đích lỗi?

### 9. Sai lầm phổ biến của ứng viên
- Không kiểm tra `write()` trả về false — bỏ qua backpressure phía Writable.
- Dùng `.pipe()` mà không xử lý error trên **từng** stream, hoặc không biết `pipeline` tồn tại.
- Buffer toàn bộ upload của user vào RAM (multer memory storage) rồi mới xử lý.
- Không biết object mode và cho rằng stream chỉ dành cho Buffer.
- Quên rằng lỗi giữa chừng stream đã gửi headers 200 → không thể đổi status code; cần chiến lược (trailer, định dạng có footer checksum, hoặc buffer phần đầu).

### 10. Follow-up Questions
- Nếu transform ở giữa chậm, điều gì xảy ra với toàn pipeline? Đo nút cổ chai bằng cách nào?
- Xử lý lỗi giữa stream khi đã gửi HTTP 200 và một nửa body?
- So sánh backpressure của Node stream với TCP flow control và Kafka consumer pull model?
- Thiết kế ETL đọc 500GB từ S3, transform, ghi vào Postgres — stream ở đâu, batch ở đâu, checkpoint/resume thế nào?
- Web Streams vs Node Streams — khác biệt và tương lai?

### 11. Liên hệ với Production
Mọi hệ thống proxy/gateway bằng Node (kể cả chính `http` module) sống nhờ backpressure — không có nó, một client chậm là một quả bom RAM. Vấn đề nghiêm trọng khi: kích thước dữ liệu do user quyết định (upload, export, webhook payload). Dấu hiệu cần rà soát: heap tăng tương quan với số client mạng chậm, OOM xảy ra khi có người export dữ liệu lớn, RSS tăng khi traffic tăng dù request count bình thường.

---

## Câu 4 — [Senior → Staff] Worker Threads vs Cluster; V8 GC và cách điều tra Memory Leak trên production

### 1. Câu hỏi
"Khi nào dùng Worker Threads, khi nào dùng Cluster? V8 quản lý bộ nhớ thế nào, và quy trình của anh khi một service Node bị nghi memory leak trên production?"

### 2. Interviewer muốn kiểm tra điều gì?
- Mức Staff: chọn đúng công cụ scale cho đúng loại tải, hiểu chi phí từng lựa chọn.
- Hiểu V8 heap đủ sâu để lý giải triệu chứng (RSS tăng, GC pause, OOM ở ~4GB).
- Có quy trình điều tra sự cố thực thụ, không phải "restart cho xong".

### 3. Câu trả lời ngắn gọn (30 giây)
"Cluster fork nhiều **process** chia nhau port — scale I/O-bound ra nhiều core, cách ly lỗi tốt, không chia sẻ bộ nhớ. Worker Threads là **thread thật trong cùng process**, mỗi worker có event loop + V8 isolate riêng, chia sẻ được bộ nhớ qua SharedArrayBuffer — dành cho CPU-bound task cần offload khỏi main loop. V8 heap chia thế hệ: new space (scavenge, nhanh, thường xuyên) và old space (mark-sweep-compact, đắt hơn). Điều tra leak: xác nhận bằng metrics trước (heapUsed tăng đơn điệu qua các full GC), rồi chụp heap snapshot ở 2–3 thời điểm, so sánh delta object và lần theo retainer path tới GC root."

### 4. Câu trả lời Senior Level (3–5 phút)
**Cluster vs Worker Threads:**
- Cluster (hoặc tốt hơn: nhiều pod K8s + LB — cluster module ngày càng ít lý do tồn tại khi đã có container orchestration): mỗi worker là process độc lập → crash một cái không kéo cả nhóm; nhưng N bản copy heap, IPC qua serialize, không share cache in-memory.
- Worker Threads: dành cho tính toán nặng (nén, parse lớn, crypto, resize ảnh). Truyền dữ liệu qua `postMessage` (structured clone — copy, có giá!) hoặc **transfer** ArrayBuffer (zero-copy, mất quyền sở hữu) hoặc SharedArrayBuffer + Atomics (share thật — kèm mọi rủi ro của shared memory). Nên dùng worker **pool** vì tạo worker tốn ~vài chục ms và vài MB.
- Quy tắc: I/O-bound → nhiều process/pod; CPU-bound → worker pool; đừng dùng worker threads để "scale" I/O — vô nghĩa, event loop đã làm việc đó.

**V8 memory:**
- Heap phân thế hệ dựa trên giả thuyết thế hệ ("hầu hết object chết trẻ"): **new space** (nhỏ, 1–16MB×2, thu bằng Scavenger — copy semispace, chỉ tốn theo số object **sống**), object sống sót 2 lần được thăng cấp lên **old space** (thu bằng Mark-Sweep-Compact, có phần concurrent/incremental nhưng vẫn đắt).
- Ngoài heap V8 còn: Buffer (off-heap), native addon memory, ArrayBuffer — vì thế **RSS tăng không đồng nghĩa V8 heap leak**.
- Giới hạn heap mặc định ~2–4GB (`--max-old-space-size` để chỉnh — và nhớ chừa phần off-heap khi đặt container limit).

**Quy trình điều tra leak:**
1. **Xác nhận:** `heapUsed` sau mỗi full GC có tăng đơn điệu không? (Sawtooth với đáy đi lên = leak thật; đáy phẳng = chỉ là GC lười.) Phân biệt heap leak vs RSS tăng (Buffer/native).
2. **Tái hiện có kiểm soát:** nếu được, đánh tải giả lên staging.
3. **Heap snapshot** (`v8.writeHeapSnapshot()` hoặc inspector) tại 2–3 mốc; so sánh trong Chrome DevTools: object nào tăng số lượng; xem **retainer path** — chuỗi tham chiếu từ GC root giữ object sống.
4. Thủ phạm quen mặt: listener đăng ký không remove (`EventEmitter` leak warning), map/cache module-level không bound (thiếu LRU/TTL), closure trong setInterval giữ cả request context, promise treo vĩnh viễn giữ toàn bộ scope, AsyncLocalStorage giữ context của request không kết thúc.
5. Vá tạm bằng restart theo lịch/memory threshold **trong khi** điều tra — nhưng phải nói rõ đó là giảm đau, không phải chữa bệnh.

### 5. Giải thích bản chất
"Leak" trong ngôn ngữ có GC không phải là quên `free` — mà là **quên cắt tham chiếu**: object vẫn reachable từ GC root nên GC bất lực. Vậy điều tra leak = truy vết đồ thị reachability, và câu hỏi đúng luôn là "**ai đang giữ nó?**" chứ không phải "tại sao GC không dọn?". GC hoạt động hoàn hảo — nó chỉ trung thành với định nghĩa "còn với tới được thì còn sống". Generational GC là một phép cá cược thống kê: trả chi phí thấp cho đa số (object trẻ) và chấp nhận chi phí cao hiếm khi (old space) — hiểu điều này lý giải vì sao leak (object cứ thăng cấp lên old space) làm full GC ngày càng dài và pause ngày càng tệ **trước khi** OOM xảy ra.

### 6. Trade-off
- **Cluster:** cách ly tốt, đơn giản — tốn RAM ×N, không share state, cần sticky session cho WebSocket.
- **Worker Threads:** tận dụng core trong 1 process — nhưng structured clone copy dữ liệu (truyền 100MB = nghẽn), SharedArrayBuffer mở cửa cho data race thật sự trong JS.
- **Heap limit lớn:** đỡ OOM — nhưng full GC trên heap lớn pause dài hơn; đôi khi 4 process × 1GB tốt hơn 1 process × 4GB.
- **Heap snapshot trên production:** thông tin quý — nhưng chụp snapshot **pause process** và tốn RAM tương đương heap; trên pod đang ngắc ngoải có thể là phát súng ân huệ.

### 7. Ví dụ Production
Leak thật tôi từng xử lý: service WebSocket, RAM tăng 100MB/giờ. Snapshot delta chỉ ra hàng trăm nghìn closure giữ `socket` object. Nguyên nhân: mỗi connection đăng ký handler vào một EventEmitter **toàn cục** (`bus.on('priceUpdate', handler)`) nhưng cleanup chỉ chạy trên đường `close` "sạch" — kết nối rớt đột ngột đi đường `error` không có cleanup. Mỗi socket chết để lại listener giữ toàn bộ context. Fix 3 dòng; tìm ra nó mất 2 ngày. Bài học: leak hầu như luôn nằm ở **đường lifecycle không hạnh phúc** (error path, timeout path).

### 8. Những câu trả lời chưa đủ tốt
- "Bị leak thì restart định kỳ." → Là mitigation, không phải diagnosis. Interviewer muốn nghe quy trình tìm gốc rễ.
- "Dùng worker threads cho nhanh." → Nhanh cái gì? I/O-bound thì worker threads không giúp gì, còn thêm chi phí copy message.

### 9. Sai lầm phổ biến của ứng viên
- Lẫn lộn cluster (process) và worker threads (thread) — câu sàng lọc nhanh nhất của chủ đề này.
- Nghĩ RSS tăng = heap leak (bỏ qua Buffer, native, fragmentation).
- Không biết `global.gc()` cần flag `--expose-gc` và không biết heapUsed phải đo **sau full GC** mới có ý nghĩa.
- Truyền object lớn qua postMessage rồi thắc mắc worker "chậm hơn chạy trực tiếp".
- Đặt `--max-old-space-size` = 100% container memory limit → OOMKill bởi phần off-heap.

### 10. Follow-up Questions
- SharedArrayBuffer + Atomics: khi nào đáng dùng bất chấp độ phức tạp? Race condition trong JS trông thế nào?
- Chụp heap snapshot làm pause service — làm sao điều tra leak với rủi ro thấp nhất? (allocation sampling profiler, canary pod nhận % traffic.)
- Leak nằm ở native addon thì heap snapshot vô dụng — điều tra tiếp thế nào? (RSS vs heap, valgrind/ASAN, `process.memoryUsage` breakdown.)
- Tại sao full GC pause tăng dần theo tuổi service là tín hiệu leak sớm hơn cả đường RAM?
- Thiết kế worker pool: kích thước bao nhiêu, queue thế nào, timeout và kill worker treo ra sao?

### 11. Liên hệ với Production
Các công ty chạy Node quy mô lớn đều có memory monitoring per-pod với alert trên **xu hướng** (slope của heapUsed sau GC) chứ không chỉ ngưỡng tuyệt đối — phát hiện leak trước OOM hàng ngày. Vấn đề nghiêm trọng khi: leak chậm chỉ lộ sau 3–7 ngày (đúng chu kỳ deploy nên bị che — "leak được chữa bởi release"), hoặc khi giảm tần suất deploy đột nhiên pod chết hàng loạt. Dấu hiệu cần điều tra: GC pause tăng dần theo uptime, sawtooth memory với đáy dâng lên, OOMKill tương quan với uptime chứ không với traffic.
