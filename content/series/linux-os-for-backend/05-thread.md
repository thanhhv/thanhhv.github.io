+++
title = "Chương 05 — Thread: nhiều dòng thực thi, một không gian địa chỉ"
date = "2026-02-21T12:00:00+07:00"
draft = false
tags = ["backend", "linux", "operating-system"]
series = ["Operating Systems cho Backend Engineer"]
+++

## 1. Problem Statement

Process cô lập tốt nhưng trả giá: hai process muốn chia sẻ dữ liệu phải đi đường vòng (pipe, socket, shared memory — đều qua kernel hoặc setup phức tạp), và tạo/chuyển process tương đối đắt. Nhiều bài toán cần điều ngược lại: **nhiều dòng thực thi đồng thời trên cùng một khối dữ liệu** — web server xử lý 10.000 connection, database xử lý nhiều query cùng chạm một buffer pool.

Thread = dòng thực thi riêng (stack + registers riêng) nhưng **chia sẻ address space, FD, mọi thứ còn lại**. Nếu không có thread: hoặc chịu chi phí IPC giữa các process, hoặc tự viết cooperative scheduling trong một process (chính là hướng event loop / goroutine — và ta sẽ thấy chúng bổ sung chứ không thay thế thread).

## 2. Tại sao nó tồn tại

- **Hardware**: máy 64 core cần ≥64 dòng thực thi để dùng hết — trong cùng một app.
- **Performance**: chia sẻ qua memory chung nhanh hơn IPC hàng chục lần; tạo thread (~50-100µs) rẻ hơn fork.
- **Latency hiding**: khi một thread block chờ IO, thread khác chạy — CPU không rảnh rỗi.
- **Business**: mô hình lập trình "mỗi request một dòng thực thi tuần tự" dễ viết, dễ đọc, dễ thuê người — giá trị kỹ nghệ khổng lồ so với callback hell.

## 3. Internal Architecture

### 3.1. Trong Linux, thread là task_struct chia sẻ tài nguyên

Như chương 04 đã hé lộ: Linux không có "thread object" riêng. Syscall thống nhất là **clone()** — fork chỉ là clone với bộ flag "không chia sẻ gì", tạo thread là clone với bộ flag "chia sẻ gần hết":

```c
// pthread_create, bên dưới glibc gọi:
clone(CLONE_VM        // chung address space (mm_struct)
    | CLONE_FILES     // chung bảng FD
    | CLONE_FS        // chung cwd, root
    | CLONE_SIGHAND   // chung signal handler
    | CLONE_THREAD,   // chung thread-group → chung PID nhìn từ ngoài
    child_stack, ...);
```

```
Process (tgid 1000)                      nhìn từ kernel:
┌─────────────────────────────────┐      3 task_struct (pid 1000,1001,1002)
│  mm_struct (address space) ◄────┼──┐   cùng tgid 1000
│  files_struct (FDs)      ◄──────┼──┤
│                                 │  │   ps      → thấy 1 "process"
│  Thread 1000   Thread 1001  ... │  │   ps -T   → thấy 3 thread (SPID)
│  [stack+regs]  [stack+regs]     │  │   htop có thể hiện cả 3 (tùy config)
└─────────────────────────────────┘  └── mỗi thread vẫn có: stack riêng,
                                          kernel stack riêng, TID riêng,
                                          register set riêng, TLS riêng
```

Hệ quả thiết kế đẹp: **scheduler không cần biết thread hay process** — nó chỉ schedule task_struct. Một process 100 thread cạnh tranh CPU như 100 process (điểm này ảnh hưởng fairness — chương 06, autogroup/cgroup).

### 3.2. 1:1 vs M:N — cuộc chiến mô hình threading

- **1:1** (Linux/NPTL hiện tại): mỗi user thread = một kernel thread. Kernel thấy hết, schedule chuẩn, blocking syscall chỉ chặn đúng thread đó. Nhược: mỗi thread tốn kernel stack (~16KB) + task_struct; context switch luôn qua kernel (~1-5µs); 100.000 thread là không thực tế.
- **M:N** (user-level threads trên N kernel threads): chuyển ngữ cảnh trong user space (~100ns, rẻ hơn 10-50 lần), triệu "thread" nhẹ là khả thi. Nhược: kernel không thấy user thread → một blocking syscall chặn cả kernel thread và mọi user thread trên đó → runtime phải bọc/tránh mọi syscall blocking. Linux từng thử (NGPT) và **bỏ**, chọn 1:1 (NPTL, 2003) vì tính đúng đắn và đơn giản.
- **Goroutine = M:N làm đúng, ở tầng ngôn ngữ**: Go runtime sở hữu toàn bộ IO của chương trình nên giải quyết được nhược điểm chí mạng của M:N — xem 4.3.

## 4. Cách hoạt động

### 4.1. Context switch — giải phẫu chi phí

Khi kernel chuyển CPU từ task A sang task B (`__schedule()` → `context_switch()`, `kernel/sched/core.c`):

```
1. Lưu registers của A vào task_struct.thread
2. Đổi kernel stack → của B
3. Nếu B khác process (khác mm): đổi CR3 → page table của B
   → TLB flush (một phần, nhờ PCID) ← ĐẮT
   Nếu B cùng process: BỎ QUA bước này ← thread switch rẻ hơn đáng kể
4. Nạp registers của B → nhảy tiếp nơi B dừng
```

Chi phí trực tiếp: ~1-2µs. Chi phí gián tiếp (thứ thật sự đau): **cache và TLB của A bị B đuổi dần** — khi A quay lại, nó chạy trên cache nguội, chậm hơn trong hàng chục µs tiếp theo. Số đo kinh điển: chi phí thật của một context switch với working set lớn có thể tương đương 10-100µs công việc bị mất.

Benchmark đại diện (2 task ping-pong qua pipe, ép context switch):

```
context switch process↔process : ~3-5 µs/lần
context switch thread↔thread   : ~1-3 µs/lần  (không đổi CR3)
goroutine switch (Go, user mode): ~0.1-0.2 µs/lần
function call                   : ~0.001-0.002 µs
```

### 4.2. Voluntary vs involuntary switch — tín hiệu chẩn đoán quý

- **Voluntary** (tự nguyện): task tự block (chờ IO, lock) — dấu hiệu workload IO-bound. Bình thường.
- **Involuntary** (bị cưỡng chế): hết time slice hoặc bị preempt — dấu hiệu **tranh chấp CPU**: quá nhiều task muốn chạy so với số core.

Xem tại `/proc/<pid>/status` (`voluntary_ctxt_switches`, `nonvoluntary_ctxt_switches`) hoặc `pidstat -w`. Involuntary cao bất thường trong container → nghĩ ngay đến CPU throttling của cgroup (chương 13) hoặc máy quá tải. Đây là một trong những số liệu chẩn đoán nhanh giá trị nhất mà ít người nhìn.

### 4.3. Goroutine — M:N scheduler của Go (mô hình GMP)

```
G (goroutine)  = stack ~2-8KB (co giãn được!) + program counter — hàng triệu G là bình thường
M (machine)    = kernel thread thật
P (processor)  = "suất chạy", số lượng = GOMAXPROCS (mặc định = số core;
                 từ Go 1.25 nhận thức cả cgroup CPU limit)

  P0 ──run queue─► [G1 G2 G3]        M0 gắn P0, đang chạy G1
  P1 ──run queue─► [G4 G5]           M1 gắn P1, đang chạy G4
  global run queue ─► [G6 ...]       (work-stealing: P rảnh trộm G của P khác)
```

Cách Go né được nhược điểm M:N cổ điển:

- **Network IO**: mọi socket là non-blocking + netpoller (epoll). G đọc socket chưa có data → G park (không phải thread block), M chạy G khác. Khi epoll báo sẵn sàng → G về run queue. **Một service Go 10.000 connection chỉ cần ~số-core kernel thread.**
- **Blocking syscall thật** (file IO, cgo, syscall lạ): không né được — runtime *tách M khỏi P* trước khi gọi, P gắn sang M khác chạy tiếp. Syscall xong, M cũ tìm P; không có thì G về queue, M ngủ. → blocking syscall không chặn chương trình, nhưng **đẻ thêm kernel thread**: app Go đọc file dồn dập trên disk chậm có thể phình lên hàng trăm M — đây là "thread explosion kiểu Go" thấy được qua `ps -T`.
- **Preemption**: từ Go 1.14, runtime gửi SIGURG để cưỡng chế dừng G chạy vòng lặp tính toán dài — trước đó một vòng lặp không gọi hàm có thể chiếm P mãi (bug treo GC nổi tiếng).

Trade-off của goroutine so với thread: mất khả năng dùng TLS/thread-affinity trực tiếp, GC phải quét nhiều stack, scheduler user-space là black box khi debug bằng công cụ OS (perf thấy M, không thấy G — cần `go tool pprof`/`runtime/trace` bổ sung).

### 4.4. Minh họa: chi phí ba mô hình xử lý 10k connection

| Mô hình | Memory cho execution state | Context switch | Độ phức tạp code |
|---|---|---|---|
| 10k process (CGI cổ) | 10k × (vài MB) — không khả thi | kernel, đắt nhất | đơn giản |
| 10k thread | 10k × (8MB stack ảo, ~vài chục KB thật + 16KB kernel stack) ≈ 500MB-1GB thật | kernel, ~1-3µs | đơn giản, nhưng cần lock |
| epoll event loop (Nginx/Node) | ~vài MB tổng | không có (1 thread) | khó: callback/state machine |
| 10k goroutine | 10k × ~4KB ≈ 40MB | user, ~0.1µs | đơn giản như thread |

Goroutine chính là nỗ lực lấy *ergonomics của thread* với *chi phí của event loop*. Java Virtual Threads (Loom, JDK 21) đi đúng con đường này.

## 5. Trade-off

| Quyết định | Được | Mất |
|---|---|---|
| Thread chia sẻ address space | Chia sẻ dữ liệu tốc độ memory | Mọi bug đồng bộ hóa: race, deadlock (chương 08); một thread crash → cả process chết |
| Linux 1:1 (NPTL) | Đúng đắn, đơn giản, kernel thấy hết | Trần thực tế ~vài nghìn thread; switch luôn tốn kernel |
| Stack thread cố định 8MB (ảo) | Không lo tràn stack thông thường | Không co giãn; giới hạn số thread; goroutine giải quyết bằng stack co giãn |
| Go GMP | Triệu goroutine, code tuần tự | Runtime phức tạp; debug xuyên hai tầng scheduler; blocking syscall vẫn đẻ thread |

## 6. Production

- Đếm thread thật: `ps -eLf | wc -l` (toàn máy), `cat /proc/<pid>/status | grep Threads`. Với Go thêm: metric `go_threads` và `go_goroutines` (hai con số này kể hai câu chuyện khác nhau!).
- Context switch rate toàn máy: `vmstat 1` cột `cs`. Baseline tùy máy (10k-100k/s là thường); **đột biến so với baseline** mới là tín hiệu. Per-process: `pidstat -w -p <pid> 1`.
- Off-CPU analysis: CPU profiling chỉ thấy thread đang *chạy*; phần lớn thời gian request thường nằm ở thread đang *chờ*. Dùng eBPF `offcputime` để thấy stack ở những chỗ thread block — công cụ trả lời "thời gian đi đâu" mạnh nhất hiện nay.
- Quy tắc kích thước pool: CPU-bound → ~số core; IO-bound → số thread ≈ core × (1 + thời_gian_chờ/thời_gian_tính) — nhưng đo đạc thắng mọi công thức.

## 7. Anti-pattern

- **Thread-per-request không giới hạn**: traffic tăng → nghìn thread → context switch + memory ăn hết máy (case Thread Explosion, chương 16). Luôn dùng pool có giới hạn + queue có giới hạn + cơ chế từ chối (backpressure).
- Tăng size thread pool để chữa chậm do lock contention — thêm thread càng tranh nhau lock, càng chậm.
- Trộn blocking call vào event loop (Node.js: một `fs.readFileSync` trên hot path đóng băng toàn bộ server; Go đỡ hơn nhờ tách M nhưng vẫn trả giá).
- `GOMAXPROCS` mặc định trong container bị limit 2 CPU trên máy 64 core (Go < 1.25): 64 P tranh nhau 2 CPU-quota → throttling liên tục, tail latency tăng vọt. Fix: `automaxprocs` hoặc đặt tay (chương 13).

## 8. Failure Analysis — case mẫu: tail latency tăng sau khi "tối ưu" bằng thêm thread

- **Triệu chứng**: p50 không đổi, p99 tăng 5 lần sau deploy tăng worker pool 32→512.
- **Điều tra**: `vmstat` cs tăng 8 lần; `pidstat -w` involuntary switch tăng vọt; `perf sched latency` cho thấy run-queue delay trung bình 4ms (trước: 50µs).
- **Root cause**: 512 thread runnable trên 32 core → mỗi thread chờ trong run queue lâu hơn nhiều; cache liên tục bị đuổi.
- **Kernel view**: scheduler vẫn "công bằng" hoàn hảo — chia đều sự chậm chạp cho tất cả. Fairness ≠ latency.
- **Fix**: về 48 worker + queue giới hạn. **Prevention**: mọi thay đổi concurrency phải qua load test đo p99, không chỉ throughput.

## 9. Khi nào không nên tối ưu

Đừng chuyển kiến trúc thread→event-loop (hay ngược lại) vì niềm tin. Dưới ~1-2k concurrent connection và context switch < 5% CPU, mô hình thread pool đơn giản gần như luôn đủ và dễ bảo trì hơn. Chi phí thật của event loop là **chi phí kỹ sư** đọc code state machine lúc 3 giờ sáng. Ngôn ngữ đã có lightweight concurrency (Go, Java 21+) thì bài toán này phần lớn đã được giải hộ.

---

**Chương tiếp theo**: nhiều task muốn chạy, ít CPU — ai được chạy, chạy bao lâu, trên core nào? → [Chương 06: CPU Scheduling](/series/linux-os-for-backend/06-cpu-scheduling/).
