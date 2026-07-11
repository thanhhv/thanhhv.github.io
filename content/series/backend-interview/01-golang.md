+++
title = "Bài 1 — Golang Internals"
date = "2026-07-03T08:00:00+07:00"
draft = false
tags = ["backend", "golang", "interview"]
series = ["Backend Interview"]
+++

---

## Câu 1 — [Fundamental → Senior] Goroutine khác gì OS thread? Giải thích mô hình GPM.

### 1. Câu hỏi
"Goroutine là gì, khác gì OS thread? Trình bày cách Go scheduler hoạt động (mô hình GPM)."

### 2. Interviewer muốn kiểm tra điều gì?
- Hiểu cơ chế bên trong runtime, không chỉ dùng `go func()` như hộp đen.
- Phân biệt được user-space scheduling vs kernel scheduling.
- Biết hệ quả thực tế: khi nào goroutine "rẻ", khi nào nó gây ra sự cố.

### 3. Câu trả lời ngắn gọn (30 giây)
"Goroutine là đơn vị thực thi do Go runtime quản lý trong user space, stack khởi đầu ~2KB và co giãn được, chi phí tạo và context switch rẻ hơn OS thread hàng trăm lần. Runtime dùng mô hình GPM: G là goroutine, M là OS thread, P là logical processor giữ run queue. Scheduler multiplex hàng triệu G lên số ít M thông qua P (mặc định P = số CPU core). Nhờ đó Go xử lý được hàng trăm nghìn kết nối đồng thời mà không cần thread pool thủ công."

### 4. Câu trả lời Senior Level (3–5 phút)
**Problem:** OS thread đắt: stack mặc định ~1–8MB, tạo/hủy phải syscall, context switch qua kernel tốn ~1–10µs và làm bẩn cache. Với mô hình "1 connection = 1 thread", 100k kết nối là bất khả thi.

**Why:** Go muốn giữ mô hình lập trình đồng bộ (dễ đọc, dễ reason) nhưng đạt hiệu năng của mô hình event-driven. Cách duy nhất: đưa scheduling về user space, nơi runtime kiểm soát chi phí.

**How — GPM:**
- **G (Goroutine):** chứa stack, program counter, trạng thái. Stack khởi đầu 2KB, grow/shrink bằng cách copy sang stack mới (vì thế Go cần biết mọi pointer — liên quan tới GC precise).
- **M (Machine):** OS thread thật, nơi code thực sự chạy.
- **P (Processor):** tài nguyên logic, số lượng = `GOMAXPROCS`. Mỗi P có **local run queue** (~256 G). M phải cầm P mới chạy được Go code.
- **Work stealing:** P hết việc sẽ lấy từ global queue, rồi steal một nửa queue của P khác — cân bằng tải mà không cần lock toàn cục.
- **Blocking syscall:** khi G gọi syscall chặn, M bị chặn theo. Runtime tách P khỏi M đó, gắn P vào M khác (lấy từ pool hoặc tạo mới) để các G còn lại tiếp tục chạy. Đây là lý do một app Go có thể có hàng trăm OS thread dù GOMAXPROCS=8.
- **Network I/O:** không đi qua blocking syscall mà qua **netpoller** (epoll/kqueue). G chờ I/O bị park, M được giải phóng hoàn toàn. Đây là điểm ăn tiền: 1 triệu connection chờ I/O ≈ 1 triệu G bị park ≈ chỉ tốn RAM stack, không tốn thread.
- **Preemption:** từ Go 1.14, scheduler preempt bằng tín hiệu async (SIGURG), nên vòng lặp tính toán thuần không còn chiếm P mãi mãi.

**Trade-off:** stack co giãn gây chi phí copy; runtime phức tạp; mất kiểm soát chi tiết về thread affinity; blocking cgo/syscall vẫn tốn OS thread thật.

**Production:** tôi quan tâm 3 chỉ số: số goroutine (`runtime.NumGoroutine`), goroutine bị block ở đâu (pprof `goroutine` profile), và thread count (nếu tăng vọt → nhiều blocking syscall/cgo).

### 5. Giải thích bản chất
Từ first principles: scheduling là bài toán "ánh xạ N đơn vị công việc lên K CPU". Kernel làm việc này tổng quát cho mọi process nên phải trả giá: chuyển ring 3→ring 0, lưu toàn bộ register state, không biết gì về ngữ nghĩa ứng dụng. Go runtime biết chính xác khi nào G sẵn sàng nhường (gọi hàm, chờ channel, chờ I/O) nên chỉ cần lưu 3 register (SP, PC, BP) — context switch ~100–200ns. **Nếu thiết kế khác đi:** nếu Go dùng 1:1 threading như Java cổ điển, nó chết ở bài toán C10K/C100K; nếu dùng callback như Node, mất tính tuần tự của code. GPM là điểm giữa: M:N scheduling. Java sau này cũng thừa nhận hướng đi này bằng Virtual Threads (Project Loom).

### 6. Trade-off
- **Performance vs Complexity:** runtime scheduler cực phức tạp, khó debug khi có vấn đề (scheduler latency, P starvation).
- **Memory:** 1 triệu G × vài KB stack = vài GB — rẻ so với thread nhưng không miễn phí.
- **Latency:** work stealing và preemption 10ms có nghĩa là một G có thể chờ tới ~10ms trước khi được chạy — quan trọng với hệ thống low-latency.
- **cgo:** mỗi call cgo có thể ghim M, phá vỡ ưu thế M:N.

### 7. Ví dụ Production
Một API gateway viết bằng Go phục vụ 200k concurrent connection trên 8 core: 200k goroutine chờ ở netpoller, chỉ ~10–20 OS thread hoạt động. Sự cố kinh điển: một thư viện DNS resolver dùng cgo → mỗi lookup chặn 1 M → traffic tăng → thread count nổ lên hàng nghìn → OOM. Fix: `GODEBUG=netdns=go` để dùng pure-Go resolver.

### 8. Những câu trả lời chưa đủ tốt
- "Goroutine là lightweight thread." → Đúng nhưng vô nghĩa. Nhẹ **ở đâu**? Stack? Context switch? Tại sao nhẹ được?
- "Go nhanh vì có goroutine." → Goroutine không làm CPU nhanh hơn; nó làm **concurrency rẻ hơn**. Phải phân biệt concurrency và parallelism.

### 9. Sai lầm phổ biến của ứng viên
- Nghĩ GOMAXPROCS = số goroutine chạy được (thực ra là số G chạy **song song**, không giới hạn tổng số G).
- Không biết netpoller tồn tại — nghĩ mọi I/O đều chặn M.
- Nghĩ goroutine leak không sao vì "nó nhẹ" — leak kèm theo cả object mà G giữ reference.
- Không biết preemption trước Go 1.14 là cooperative (từng gây bug GC chờ mãi một vòng lặp tight loop).

### 10. Follow-up Questions
- Điều gì xảy ra khi 1 goroutine gọi blocking syscall? Khi gọi cgo?
- 1 triệu goroutine cùng chờ 1 channel — chi phí là gì? (đánh thức tuần tự, thundering herd?)
- GOMAXPROCS nên set thế nào khi chạy trong container bị limit 2 CPU? (Go <1.25 không đọc cgroup limit → cần `automaxprocs`; hệ quả: CFS throttling.)
- Làm sao phát hiện goroutine leak trên production?
- So sánh với Virtual Threads của Java — Go còn lợi thế gì?

### 11. Liên hệ với Production
Cloudflare, Uber, Twitch đều chọn Go cho edge/proxy layer chính vì chi phí concurrency. Vấn đề trở nên nghiêm trọng khi: goroutine count tăng đơn điệu (leak), thread count tăng đột biến (blocking syscall), hoặc p99 latency tăng dù CPU thấp (CFS throttling do GOMAXPROCS sai trong container — bug kinh điển Uber từng viết blog). Dấu hiệu cần tối ưu: pprof cho thấy thời gian lớn ở `runtime.schedule` hoặc `runtime.futex`.

---

## Câu 2 — [Intermediate → Senior] Channel hoạt động thế nào? Khi nào dùng channel, khi nào dùng mutex?

### 1. Câu hỏi
"Channel được cài đặt bên trong ra sao? Nguyên tắc chọn giữa channel và mutex?"

### 2. Interviewer muốn kiểm tra điều gì?
- Có hiểu channel không phải "phép màu" mà là mutex + queue + semaphore bên dưới.
- Kinh nghiệm thực tế chọn công cụ đồng bộ hóa — nơi rất nhiều ứng viên dùng sai.
- Hiểu các deadlock/leak pattern.

### 3. Câu trả lời ngắn gọn (30 giây)
"Channel bên trong là struct `hchan`: một circular buffer, hai hàng đợi goroutine chờ gửi/nhận, và một mutex bảo vệ tất cả. Gửi vào channel đầy hoặc nhận từ channel rỗng sẽ park goroutine vào hàng đợi. Nguyên tắc chọn: channel để **chuyển quyền sở hữu dữ liệu** và điều phối luồng (pipeline, fan-out, cancellation); mutex để **bảo vệ shared state** truy cập ngắn. Dùng channel làm mutex hay mutex làm cơ chế signaling đều là dùng sai công cụ."

### 4. Câu trả lời Senior Level (3–5 phút)
**Problem:** shared memory concurrency dễ sinh data race. Go đề xuất triết lý "Don't communicate by sharing memory; share memory by communicating".

**How channel hoạt động:**
- `hchan` gồm: `buf` (circular buffer), `sendx/recvx` (index), `sendq/recvq` (danh sách `sudog` — goroutine đang chờ), `lock` (mutex).
- **Fast path:** nếu có receiver đang chờ, sender copy dữ liệu **trực tiếp vào stack của receiver** (bỏ qua buffer) rồi đánh thức nó — một tối ưu quan trọng.
- **Slow path:** buffer đầy → sender bị `gopark`, gắn vào `sendq`. Khi có chỗ, runtime `goready` sender đầu tiên (FIFO — công bằng nhưng có chi phí).
- Unbuffered channel = điểm hẹn (rendezvous): gửi và nhận xảy ra đồng thời, tạo happens-before mạnh.

**Chọn channel vs mutex:**
- Channel: chuyển ownership (worker pool nhận job), signaling (done channel), pipeline, kết hợp `select` với cancellation/timeout.
- Mutex: đếm counter, cập nhật map, cache — critical section ngắn, không có "luồng dữ liệu". Mutex nhanh hơn đáng kể cho case này (không copy, không park/unpark khi uncontended — chỉ 1 CAS).
- Atomic: counter/flag đơn giản nhất — nhanh nhất nhưng dễ sai khi logic phức tạp hơn 1 biến.

**Trade-off & Production:** channel có chi phí ẩn: mỗi operation qua mutex nội bộ; channel bị đóng sai chỗ gây panic; goroutine chờ channel không ai gửi = leak vĩnh viễn. Trên production tôi thấy leak channel nhiều hơn deadlock mutex.

### 5. Giải thích bản chất
Tại sao channel không lock-free? Vì nó phải hỗ trợ `select` trên nhiều channel — một goroutine phải đăng ký chờ trên N channel nguyên tử, điều gần như bất khả thi với thiết kế lock-free mà vẫn đúng. Go chọn đúng đắn + đơn giản hơn là tối ưu tuyệt đối. **Nếu thiết kế khác:** nếu channel unbounded mặc định (như Erlang mailbox), backpressure biến mất — producer nhanh sẽ ăn hết RAM. Buffer bounded của Go là backpressure tự nhiên: producer bị chặn lại khi consumer chậm. Đây là một quyết định thiết kế hệ thống được nhúng vào ngôn ngữ.

### 6. Trade-off
- **Channel:** an toàn về ownership, tự có backpressure; đổi lại chậm hơn mutex ~2–5 lần cho shared state, dễ leak goroutine, API đóng channel khó dùng đúng (chỉ sender được close).
- **Mutex:** nhanh, quen thuộc; đổi lại dễ quên unlock, không compose được (lock 2 mutex theo thứ tự khác nhau = deadlock), không có cancellation.
- **Buffered size:** buffer lớn giấu vấn đề consumer chậm; buffer 0 làm hệ thống lock-step. Kích thước buffer là quyết định về latency vs throughput vs khả năng phát hiện sớm sự cố.

### 7. Ví dụ Production
Worker pool xử lý webhook: `jobs := make(chan Job, 1000)`. Khi downstream chậm, buffer đầy, HTTP handler chặn ở send → request timeout → alert sớm. Nếu dùng unbounded queue tự chế, hệ thống "có vẻ khỏe" cho tới khi OOM. Sự cố thật: một service leak 2 triệu goroutine trong 3 ngày vì `select` thiếu nhánh `ctx.Done()` khi channel kết quả không bao giờ được ghi (upstream đã lỗi) — mỗi leak giữ 1 buffer 64KB → 120GB RAM.

### 8. Những câu trả lời chưa đủ tốt
- "Channel dùng để giao tiếp giữa các goroutine." → Đúng nhưng mọi cơ chế đồng bộ đều thế. Phải nói được ownership transfer và backpressure.
- "Channel an toàn hơn mutex." → Không — channel sai vẫn deadlock, vẫn leak. An toàn hơn **cho pattern nào**?

### 9. Sai lầm phổ biến của ứng viên
- Close channel từ phía receiver, hoặc close 2 lần → panic.
- Không biết nhận từ channel đã đóng trả về zero value ngay lập tức → vòng lặp busy-loop.
- Dùng `sync.Mutex` copy theo value (struct chứa mutex bị copy) — race kinh điển.
- Nghĩ buffered channel là "async hoàn toàn" — quên rằng khi đầy nó vẫn chặn.
- Không biết `select` với nhiều case sẵn sàng sẽ chọn **ngẫu nhiên** (tránh starvation).

### 10. Follow-up Questions
- Làm sao implement timeout cho một thao tác gửi channel? (`select` + `time.After` — và tại sao `time.After` trong vòng lặp gây leak timer trước Go 1.23?)
- Nếu 10k goroutine cùng chờ 1 channel, close channel đó thì chuyện gì xảy ra? (tất cả được đánh thức — broadcast duy nhất của channel.)
- Thiết kế semaphore bằng channel? Khi nào dùng `golang.org/x/sync/semaphore` thay vì channel?
- `sync.RWMutex` khi nào **chậm hơn** Mutex thường? (write nhiều, hoặc cache-line contention của reader count.)
- Data race là gì ở mức memory model? Go memory model đảm bảo gì quanh channel?

### 11. Liên hệ với Production
Các codebase lớn (Kubernetes, etcd) dùng channel chủ yếu cho signaling và event loop, còn shared state hầu hết dùng mutex — đúng như nguyên tắc trên. Vấn đề nghiêm trọng khi: goroutine profile cho thấy hàng nghìn G kẹt ở `chan send/receive`, hoặc mutex profile (`pprof mutex`) cho thấy contention cao. Dấu hiệu cần tối ưu: p99 tăng theo concurrency trong khi CPU idle — thường là lock contention.

---

## Câu 3 — [Senior] Context hoạt động thế nào và tại sao nó quan trọng trong hệ thống phân tán?

### 1. Câu hỏi
"Trình bày cơ chế của `context.Context`. Tại sao Go bắt truyền context tường minh qua mọi tầng thay vì dùng thread-local như Java?"

### 2. Interviewer muốn kiểm tra điều gì?
- Hiểu cancellation propagation — kỹ năng sống còn khi viết service gọi service.
- Hiểu quyết định thiết kế API (tường minh vs ngầm định).
- Kinh nghiệm với timeout budget trong microservices.

### 3. Câu trả lời ngắn gọn (30 giây)
"Context là một cây: mỗi `WithCancel/WithTimeout` tạo node con; hủy node cha lan xuống toàn bộ con cháu qua việc đóng một channel `Done()`. Nó giải quyết bài toán: khi client bỏ cuộc hoặc hết timeout, mọi công việc phía sau (DB query, RPC, goroutine con) phải dừng để không lãng phí tài nguyên. Go chọn truyền tường minh làm tham số đầu tiên để cancellation là một phần của contract hàm, không phải trạng thái ẩn — code dễ reason hơn dù dài dòng hơn."

### 4. Câu trả lời Senior Level (3–5 phút)
**Problem:** trong hệ phân tán, một request đi qua 5 service. Client timeout sau 2s. Nếu không có cancellation, service cuối vẫn cày một query 30s cho một kết quả không ai nhận — tài nguyên chết dưới tải chính là nguyên nhân của cascading failure.

**How:**
- `Done()` trả về channel; hủy = close channel → mọi goroutine `select` trên nó được đánh thức đồng loạt (tận dụng tính broadcast của close).
- Cây context: cancel cha → duyệt và cancel mọi con. `WithTimeout` = `WithDeadline` = timer tự gọi cancel.
- `ctx.Err()` phân biệt `Canceled` vs `DeadlineExceeded` — quan trọng để quyết định retry hay không.
- **Deadline propagation:** gRPC tự truyền deadline qua metadata; service nhận được deadline còn lại, trừ hao rồi truyền tiếp — gọi là timeout budget.
- `context.Value`: chỉ dùng cho request-scoped metadata (trace ID, auth), không bao giờ dùng để truyền tham số nghiệp vụ — vì nó phá type safety và giấu dependency.

**Trade-off:** verbose (ctx ở mọi chữ ký hàm); dễ quên check `ctx.Done()` trong vòng lặp dài; `context.Value` là map lookup theo chuỗi cha — O(depth).

**Production:** quy tắc của tôi: mọi thao tác I/O nhận ctx; mọi vòng lặp dài check ctx định kỳ; đặt timeout **tại biên** (HTTP middleware) và để nó chảy xuống; log luôn kèm `ctx.Err()` khi hủy để phân biệt lỗi thật với lỗi do client bỏ cuộc.

### 5. Giải thích bản chất
Cancellation về bản chất là bài toán **broadcast một sự kiện xảy-ra-một-lần tới N listener không xác định trước**. Close channel là primitive duy nhất trong Go làm được điều này với chi phí O(1) cho người kiểm tra (`select` non-blocking). **Nếu thiết kế khác:** dùng thread-local (Java MDC/ThreadLocal) sẽ vỡ ngay trong Go vì goroutine không có identity ổn định và một request có thể nhảy qua nhiều goroutine. Truyền tường minh còn có hệ quả xã hội: reviewer nhìn chữ ký hàm là biết hàm này có thể bị hủy — contract rõ ràng hơn.

### 6. Trade-off
- **Explicitness vs Ergonomics:** an toàn, dễ đọc — nhưng boilerplate; so với Kotlin coroutine scope (structured concurrency ngầm) thì Go thô hơn.
- **Correctness vs Performance:** check ctx trong hot loop tốn một atomic load + branch — không đáng kể, nhưng `context.Value` trong hot path thì tốn thật.
- Hủy không mang tính bắt buộc: hàm phớt lờ ctx vẫn compile — kỷ luật đặt lên vai con người.

### 7. Ví dụ Production
Sự cố kinh điển: search service p99 = 5s trong khi client timeout 1s. 80% công việc backend làm là **cho những request đã chết**. Thêm propagation từ HTTP handler xuống Elasticsearch client → load giảm 40% ngay lập tức, p99 giảm theo. Ngược lại, sự cố do hủy quá tay: hủy ctx làm dở dang một thao tác ghi 2 hệ thống (đã ghi DB, chưa gửi Kafka) → phải tách "ctx của request" khỏi "ctx của công việc ghi" (`context.WithoutCancel` từ Go 1.21).

### 8. Những câu trả lời chưa đủ tốt
- "Context để truyền timeout và value." → Mô tả bề mặt. Phải nói được cơ chế cây + close channel + tại sao tường minh.
- "Luôn truyền context là best practice." → Tại sao? Best practice không có lý do là giáo điều.

### 9. Sai lầm phổ biến của ứng viên
- Lưu ctx vào struct field (ctx phải theo luồng gọi, không theo object lifetime).
- Quên `defer cancel()` → leak timer và node con trên cây.
- Dùng `context.Value` truyền DB connection, logger — dependency ẩn.
- Không phân biệt hủy do client vs hủy do deadline khi quyết định retry — retry cái đã DeadlineExceeded chỉ tăng tải.
- Nghĩ cancel giết được goroutine — không, cancellation trong Go là **hợp tác**, goroutine phải tự check.

### 10. Follow-up Questions
- Làm sao dừng một query PostgreSQL đang chạy khi ctx bị hủy? (driver gửi cancel request riêng — và hệ quả với connection pool?)
- Timeout budget: service A timeout 1s gọi B rồi C tuần tự — chia thời gian thế nào?
- `WithoutCancel` dùng khi nào? Rủi ro gì?
- Structured concurrency là gì? `errgroup` liên quan thế nào tới ctx?
- Nếu 1 triệu goroutine cùng select trên 1 ctx.Done() và ctx bị hủy — điều gì xảy ra với scheduler?

### 11. Liên hệ với Production
Google nội bộ coi deadline propagation là bắt buộc — mọi RPC có deadline, hết deadline là hủy toàn tuyến. Vấn đề nghiêm trọng khi hệ thống bắt đầu cascading failure: retry storm + không cancellation = load khuếch đại qua từng tầng. Dấu hiệu cần rà soát: tỷ lệ lớn log lỗi là `context deadline exceeded` ở tầng sâu, hoặc backend vẫn full load sau khi client đã giảm traffic.

---

## Câu 4 — [Senior → Staff] Go GC hoạt động thế nào? Escape analysis liên quan gì tới hiệu năng?

### 1. Câu hỏi
"Trình bày cơ chế GC của Go. Escape analysis là gì và tại sao một Senior phải quan tâm?"

### 2. Interviewer muốn kiểm tra điều gì?
- Hiểu sâu memory management — thứ phân biệt người "viết Go chạy được" và người "viết Go chạy nhanh".
- Khả năng chẩn đoán vấn đề GC trên production.
- Hiểu trade-off thiết kế GC của Go so với Java.

### 3. Câu trả lời ngắn gọn (30 giây)
"Go dùng concurrent, tri-color, mark-and-sweep GC, non-generational, non-compacting, tối ưu cho **latency thấp** — pause chỉ vài trăm microsecond nhờ chạy song song với chương trình, đổi lại throughput thấp hơn GC của Java và tốn ~25% CPU khi GC chạy. Escape analysis là bước compiler quyết định biến nằm ở stack hay heap: biến 'thoát' khỏi phạm vi hàm (trả về pointer, gửi vào channel, bị interface hóa) phải lên heap. Ít escape = ít rác = ít GC. Đây là đòn bẩy hiệu năng lớn nhất trong Go trước khi nghĩ tới thuật toán."

### 4. Câu trả lời Senior Level (3–5 phút)
**Problem:** GC phải chọn giữa ba thứ không thể có cùng lúc: pause ngắn, throughput cao, memory footprint thấp.

**How:**
- **Tri-color marking:** trắng (chưa thăm), xám (đã thăm, con chưa quét), đen (xong). Chạy **đồng thời** với mutator nhờ **write barrier**: khi chương trình ghi pointer trong lúc GC chạy, barrier ghi nhận để không "mất dấu" object — bảo toàn bất biến "đen không trỏ tới trắng".
- **Pacing:** `GOGC=100` nghĩa là GC chạy khi heap tăng gấp đôi so với live heap sau lần GC trước. `GOMEMLIMIT` (Go 1.19) đặt trần memory — cực quan trọng trong container.
- **Stop-the-world** chỉ còn 2 lần rất ngắn (bật/tắt write barrier), thường <1ms.
- **Không generational:** Go đặt cược rằng escape analysis + stack allocation đã lọc phần lớn "object trẻ chết sớm" — thế hệ trẻ nằm ở stack, không tới heap.
- **Không compacting:** tránh phải sửa pointer khi di chuyển object; đổi lại dựa vào allocator kiểu tcmalloc (size class) để chống phân mảnh.
- **Escape analysis:** compiler chứng minh được biến không sống lâu hơn hàm → cấp phát trên stack (miễn phí, tự thu hồi). Xem bằng `go build -gcflags="-m"`. Các nguyên nhân escape phổ biến: trả về pointer, đưa vào interface (boxing), closure capture, slice grow vượt kích thước biết trước, gửi qua channel.

**Trade-off & Production:** GC latency-oriented hợp với service online; nhưng batch job xử lý hàng trăm GB nên cân nhắc giảm allocation (sync.Pool, tái dùng buffer, tránh `[]byte` ↔ `string` chuyển đổi). Trên production tôi theo dõi: GC CPU fraction, pause p99, heap live vs goal, và allocation rate (bytes/s) — allocation rate mới là gốc rễ, GC chỉ là triệu chứng.

### 5. Giải thích bản chất
First principles: chi phí GC ≈ tần suất GC × chi phí mỗi lần. Tần suất tỉ lệ thuận với **allocation rate** và tỉ lệ nghịch với heap headroom. Vậy hai đòn bẩy: cấp phát ít hơn (escape analysis, pooling) hoặc cho heap nhiều headroom hơn (tăng GOGC — đổi RAM lấy CPU). **Nếu thiết kế khác:** Java chọn generational + compacting → throughput tốt, nhưng cần pause dài hơn hoặc cơ chế cực phức tạp (ZGC dùng colored pointer). Go chọn đơn giản + concurrent vì Go nhắm tới network service — nơi p99 latency quan trọng hơn throughput tính toán thuần. Thiết kế GC phản ánh use case mục tiêu của ngôn ngữ.

### 6. Trade-off
- **GOGC cao:** ít GC, ít CPU — nhưng RAM gấp nhiều lần live heap; nguy hiểm trong container limit chặt (OOMKill).
- **GOMEMLIMIT:** chặn OOM — nhưng khi live heap tiến sát limit, GC chạy liên tục ("GC death spiral"), CPU cháy vào GC.
- **sync.Pool:** giảm allocation — nhưng object trong pool bị clear mỗi GC cycle, và dùng sai (giữ reference sau khi Put) gây bug data corruption cực khó tìm.
- **Stack allocation:** miễn phí — nhưng stack grow (copy toàn bộ stack) cũng có giá nếu goroutine đệ quy sâu.

### 7. Ví dụ Production
Service ad-bidding p99 phải <10ms: pprof cho thấy 30% CPU vào GC, allocation rate 2GB/s. Nguyên nhân: mỗi request parse JSON tạo hàng trăm object nhỏ + log format `fmt.Sprintf` (escape). Fix: chuyển sang parser zero-allocation, tái dùng buffer qua `sync.Pool`, structured logging không reflection → allocation giảm 8 lần, GC CPU còn 4%, p99 giảm một nửa. Sự cố ngược: team set `GOGC=off` để "tắt GC cho nhanh" trong batch job → OOMKill giữa chừng sau 4 giờ chạy.

### 8. Những câu trả lời chưa đủ tốt
- "Go GC rất nhanh, pause dưới 1ms." → Nhanh **về latency**, còn tổng CPU cho GC thì sao? Nói một chiều là chưa hiểu trade-off.
- "Muốn nhanh thì tránh dùng pointer." → Ngược đời — đôi khi truyền value to còn tốn hơn (copy); vấn đề là **escape**, không phải pointer.

### 9. Sai lầm phổ biến của ứng viên
- Nghĩ `new()` lên heap còn `var` ở stack — vị trí do escape analysis quyết định, không do cú pháp.
- Không biết interface conversion gây allocation (boxing giá trị non-pointer).
- Tin rằng Go không bao giờ OOM vì có GC — GC không cứu được live heap thật sự lớn hoặc goroutine leak.
- Dùng finalizer để giải phóng tài nguyên — thứ tự và thời điểm không đảm bảo.
- Tối ưu allocation khi chưa profile — thủ phạm thật thường nằm ở 2–3 hàm.

### 10. Follow-up Questions
- Write barrier là gì, tại sao cần? Chi phí của nó?
- Tại sao Go không cần generational GC? Lập luận đó có điểm yếu nào? (workload tạo nhiều object sống-vừa.)
- GOMEMLIMIT hoạt động thế nào khi heap tiến sát limit? Death spiral là gì, phòng thế nào?
- Trong container 1GB, bạn set GOGC/GOMEMLIMIT ra sao? Tại sao?
- Làm sao biết một dòng code gây escape? Làm sao đo allocation của một handler cụ thể?

### 11. Liên hệ với Production
Twitch từng viết bài kinh điển giảm GC pause bằng "memory ballast" (trước khi có GOMEMLIMIT). Discord rời Go sang Rust cho một service chính vì spike latency định kỳ do GC trên heap lớn — bài học: GC latency-oriented vẫn có giới hạn khi live heap hàng chục GB. Dấu hiệu cần tối ưu: `go_gc_cpu_fraction` > 5–10%, pause p99 tăng theo heap, hoặc latency có "răng cưa" đều đặn trùng chu kỳ GC.

---

## Câu 5 — [Staff → Principal] Interface hoạt động bên trong ra sao? Khi nào dùng Generics thay vì interface? Chiến lược profiling một service Go chậm.

### 1. Câu hỏi
"Interface trong Go được biểu diễn thế nào ở runtime? Generics giải quyết vấn đề gì mà interface không làm được? Và nếu tôi đưa anh một service Go p99 đang tệ, anh tiếp cận thế nào?"

### 2. Interviewer muốn kiểm tra điều gì?
- Mức Staff: nối được ba mảng kiến thức (ngôn ngữ, runtime, vận hành) thành một câu chuyện.
- Có phương pháp luận tối ưu, không đoán mò.
- Hiểu chi phí của abstraction — tư duy "mọi lớp trừu tượng đều có giá".

### 3. Câu trả lời ngắn gọn (30 giây)
"Interface là fat pointer 2 word: con trỏ tới bảng phương thức (itab) và con trỏ tới dữ liệu — vì thế gọi qua interface là dynamic dispatch, chặn inline, và boxing giá trị gây heap allocation. Generics (Go 1.18) cho phép viết code tổng quát **không mất type safety và giảm chi phí boxing**, compile theo hướng GCShape stenciling — nhanh hơn interface cho code chứa logic, nhưng không thay thế interface trong vai trò contract giữa các module. Với service chậm: đo trước, USE/RED metrics để khoanh vùng, rồi pprof CPU/heap/block/mutex, rồi mới sửa — và sửa xong phải đo lại."

### 4. Câu trả lời Senior Level (3–5 phút)
**Interface internals:** `iface = (itab, data)`. `itab` cache cặp (interface type, concrete type) và bảng con trỏ hàm. Hệ quả: (1) gọi phương thức = 1 lần gián tiếp qua bảng → CPU khó dự đoán nhánh, compiler không inline được; (2) đưa giá trị non-pointer vào interface thường phải cấp phát bản sao trên heap; (3) `nil interface` ≠ interface chứa nil pointer — bug kinh điển khi trả về `error`.

**Generics:** trước 1.18, code tổng quát phải chọn: copy-paste (an toàn, nhanh, trùng lặp), `interface{}` + type assertion (mất an toàn, chậm), hay codegen (phức tạp). Generics dùng **GCShape stenciling**: các kiểu cùng "hình dạng bộ nhớ" (mọi pointer chung 1 shape) dùng chung một bản instantiate + dictionary runtime. Trade-off có chủ đích: tránh bùng nổ binary như C++ template, đổi lại kiểu pointer vẫn có một mức gián tiếp — generics không phải lúc nào cũng nhanh như code viết tay. Nguyên tắc của tôi: **generics cho cấu trúc dữ liệu và thuật toán; interface cho hành vi và ranh giới kiến trúc** (contract giữa layer, mock trong test).

**Phương pháp profiling:**
1. Định nghĩa "chậm": p50 hay p99? Mọi endpoint hay một endpoint? Từ bao giờ (deploy nào)?
2. Khoanh vùng bằng metrics/tracing: thời gian nằm ở service hay downstream (DB, RPC)? RED cho service, USE cho tài nguyên.
3. Nếu tại service: `pprof` CPU 30s dưới tải thật; heap profile cho allocation; **block/mutex profile** cho contention — p99 tệ trong khi CPU thấp thường là lock/IO chứ không phải tính toán.
4. Continuous profiling (Pyroscope/Parca) để so sánh trước–sau deploy.
5. Sửa một thứ một lần, benchmark bằng `benchstat` để tránh nhiễu.

### 5. Giải thích bản chất
Mọi abstraction là một lớp gián tiếp, và mọi lớp gián tiếp có giá ở một trong ba chỗ: compile time, binary size, hoặc runtime. Interface trả giá ở runtime (dynamic dispatch + allocation). C++ template trả giá ở compile time và binary size. Go generics cố ý đứng giữa. Hiểu điều này giúp trả lời mọi câu "nên dùng X hay Y": hỏi ngược lại — **giá trả ở đâu và workload của ta nhạy cảm với giá nào?** Với profiling, bản chất là khoa học thực nghiệm: hypothesis → measure → change → measure. Người thiếu kinh nghiệm sửa trước đo sau; hệ quả là tối ưu nhầm chỗ chiếm 1% thời gian.

### 6. Trade-off
- **Interface:** decoupling, testability — giá: dispatch, allocation, mất inline, và giá vô hình lớn nhất: đọc code không biết implementation nào chạy.
- **Generics:** type safety, hiệu năng — giá: phức tạp hóa API, GCShape không tối ưu tuyệt đối, khó đọc khi lạm dụng constraint.
- **Tối ưu hiệu năng:** mỗi lần tối ưu là một lần đổi tính dễ đọc/dễ sửa lấy tốc độ — chỉ trả giá này ở hot path đã được chứng minh bằng profile.

### 7. Ví dụ Production
Hot path serialize event (3 tỷ event/ngày) dùng `interface{}` + reflection: 40% CPU vào reflect + GC. Chuyển sang generics + code sinh sẵn: throughput ×3. Ngược lại, tầng repository giữ nguyên interface dù "chậm hơn" vài ns — vì giá trị mock/test và thay đổi DB engine lớn hơn nhiều. Bug production đáng nhớ: hàm trả về `*MyErr` nil được gán vào biến `error` → `err != nil` thành true → hệ thống retry vô hạn một thao tác đã thành công.

### 8. Những câu trả lời chưa đủ tốt
- "Generics tốt hơn interface vì nhanh hơn." → Nhanh hơn **cho việc gì**? Ranh giới kiến trúc vẫn cần interface.
- "Em sẽ dùng pprof." → Dùng profile nào, dưới tải nào, tìm gì, và nếu CPU profile phẳng thì làm gì tiếp? Công cụ không phải phương pháp.

### 9. Sai lầm phổ biến của ứng viên
- Nghĩ interface trong Go giống Java (nominal) — Go là structural typing, satisfy ngầm.
- Không biết bug nil-interface-chứa-nil-pointer.
- Generics hóa mọi thứ sau 1.18 — API khó đọc hơn mà không được gì.
- Profiling trên môi trường dev không tải — kết quả vô nghĩa.
- Chỉ nhìn CPU profile khi vấn đề là contention (cần block/mutex profile) hoặc off-CPU (chờ I/O).

### 10. Follow-up Questions
- Tại sao Go chọn structural typing? Hệ quả với việc tiến hóa API?
- GCShape stenciling khác monomorphization hoàn toàn (Rust) chỗ nào? Hệ quả hiệu năng?
- Nếu p99 tệ nhưng CPU 20%, anh nghi ngờ gì trước? (lock, connection pool cạn, GC assist, CFS throttling, downstream.)
- Làm sao profiling trên production mà không ảnh hưởng user? (sampling overhead của pprof, continuous profiling.)
- Có khi nào từ chối tối ưu dù profile chỉ rõ hot spot không? (khi chi phí bảo trì > tiền máy — quyết định Principal.)

### 11. Liên hệ với Production
Uber vận hành continuous profiling toàn công ty và từng công bố tiết kiệm hàng nghìn core nhờ sửa các hot spot allocation/reflection. Vấn đề trở nên nghiêm trọng khi: chi phí hạ tầng tăng tuyến tính theo traffic (không có economy of scale — dấu hiệu code kém hiệu quả), hoặc p99 vi phạm SLO khi traffic đạt đỉnh. Dấu hiệu cần hành động: flame graph có một hàm chiếm >10–20% CPU, allocation rate tăng sau một release, mutex contention tăng theo số core (service không scale theo chiều dọc).
