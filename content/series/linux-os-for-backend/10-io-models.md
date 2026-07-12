+++
title = "Chương 10 — IO Models: từ blocking đến io_uring, tiến hóa của câu hỏi \"chờ thế nào\""
date = "2026-02-21T17:00:00+07:00"
draft = false
tags = ["backend", "linux", "operating-system"]
series = ["Operating Systems cho Backend Engineer"]
+++

## 1. Problem Statement

Server có 10.000 connection. Tại mỗi thời điểm chỉ vài trăm connection có dữ liệu; còn lại im lặng. Câu hỏi định hình cả một thế hệ kiến trúc backend: **làm sao biết connection nào sẵn sàng mà không tốn một thread cho mỗi connection và không đốt CPU quay vòng hỏi?** Đây là "bài toán C10K" (1999) — và mỗi IO model là một câu trả lời ở một thời kỳ.

Chú ý phạm vi: chương này chủ yếu nói về **network IO**. File IO trên disk luôn "sẵn sàng" theo nghĩa của select/poll/epoll (chúng không giúp gì cho file) — đó là lỗ hổng lịch sử mà mãi tới io_uring mới lấp tử tế.

## 2. Tại sao tồn tại nhiều model — bốn câu hỏi độc lập

1. Syscall có **block** không khi chưa sẵn sàng? (blocking vs non-blocking)
2. Làm sao **biết** fd sẵn sàng? (hỏi từng cái vs multiplexing: select/poll/epoll)
3. Ai **thực hiện** IO: app tự read/write khi sẵn sàng (readiness) hay kernel làm hộ rồi báo xong (completion — io_uring)?
4. Chi phí **mỗi event** là bao nhiêu syscall, bao nhiêu copy?

## 3. Internal Architecture — tiến hóa từng bước

### 3.1. Blocking IO — mô hình gốc

```c
n = read(fd, buf, len);   // chưa có data → process ngủ (TASK_INTERRUPTIBLE)
                          // data đến → softirq đánh thức → copy → return
```

Đơn giản, đúng, **không tốn CPU khi chờ** (chương 03). Vấn đề duy nhất: một thread chỉ chờ được MỘT fd. 10k connection = 10k thread = memory + context switch (chương 05). Nhưng đừng khinh nó: với vài trăm connection, thread-per-connection + blocking IO vẫn là kiến trúc dễ đúng nhất.

### 3.2. Non-blocking IO — tách "thử" khỏi "chờ"

```c
fcntl(fd, F_SETFL, O_NONBLOCK);
n = read(fd, buf, len);   // chưa có data → trả -1, errno=EAGAIN NGAY
```

Tự nó vô dụng (busy-loop 100% CPU). Giá trị của nó là làm **nguyên liệu** cho multiplexing: chờ tập trung một chỗ, đọc/ghi không bao giờ kẹt.

### 3.3. select / poll — multiplexing thế hệ 1

```c
// select: bitmap 1024 fd cứng; poll: mảng pollfd không giới hạn
poll(fds, nfds, timeout);   // block đến khi ÍT NHẤT MỘT fd sẵn sàng
```

Khuyết tật chung, mang tính cấu trúc: **stateless** — mỗi lần gọi phải (1) copy toàn bộ danh sách fd vào kernel, (2) kernel duyệt và gắn wait-callback lên *từng* fd, (3) trả về rồi vứt hết, lần sau làm lại từ đầu. Chi phí O(n) **mỗi lần gọi** theo tổng số fd theo dõi, kể cả khi chỉ 1 fd có event. 10k connection × nghìn lần gọi/giây = nghìn tỷ thao tác vô ích.

### 3.4. epoll — chuyển state vào kernel, O(1) theo số fd

Sửa đúng khuyết tật cấu trúc: **đăng ký một lần, kernel nhớ**:

```c
epfd = epoll_create1(0);
epoll_ctl(epfd, EPOLL_CTL_ADD, fd, &ev);   // 1 LẦN mỗi fd

// vòng đời:
n = epoll_wait(epfd, events, max, timeout); // trả về CHỈ các fd sẵn sàng
```

Bên trong kernel (`fs/eventpoll.c`):

```
epoll instance:
  ├── cây đỏ-đen: mọi fd đã đăng ký (để ctl nhanh, chống trùng)
  └── ready list: danh sách fd ĐANG sẵn sàng
Cơ chế đẩy (không phải quét!):
  packet đến → TCP xử lý → socket có data → callback ep_poll_callback
  → thêm socket vào ready list → đánh thức epoll_wait
→ chi phí tỷ lệ với SỐ EVENT, không phải số fd theo dõi. 10k im lặng: 0 chi phí.
```

**Level-triggered (LT, mặc định)** vs **Edge-triggered (ET)**: LT — "còn data là còn báo" (dễ đúng, có thể thừa wakeup); ET — "chỉ báo khi *chuyển trạng thái* rỗng→có" (ít wakeup, nhưng **bắt buộc** đọc cạn tới EAGAIN mỗi lần, quên là treo connection vĩnh viễn — bug kinh điển, xem case Epoll Bug chương 16). Nginx, Go netpoller dùng ET. Multi-thread cùng chờ một listen fd: vấn đề thundering herd lịch sử → `EPOLLEXCLUSIVE`, hoặc `SO_REUSEPORT` (mỗi worker một queue riêng — cách hiện đại).

Đây chính là trái tim của Nginx, Redis, HAProxy, Node.js (libuv), Go netpoller, Java NIO. **Một thread epoll phục vụ hàng trăm nghìn connection.**

### 3.5. io_uring — thế hệ completion: kernel làm hộ, báo khi xong

epoll vẫn còn ba khoản phí: (1) mỗi vòng vẫn ≥1 syscall `epoll_wait` + n syscall read/write; (2) readiness không áp dụng cho file IO; (3) mỗi read vẫn copy kernel→user lúc gọi. **io_uring** (5.1, 2019) đổi hẳn mô hình:

```
             USER SPACE                      KERNEL
  ┌─────────────────────────────┐
  │ SQ ring (submission queue)  │──── kernel đọc lệnh: "đọc fd 7 vào buf X",
  │  đặt lệnh vào ring buffer   │     "ghi fd 9", "accept", "fsync", chuỗi lệnh
  │  CHIA SẺ MEMORY với kernel  │     link nhau...
  │                             │
  │ CQ ring (completion queue)  │◄─── kernel đặt kết quả: "lệnh #42 xong, 8192B"
  └─────────────────────────────┘
  Syscall io_uring_enter chỉ cần khi: nộp batch + chờ. Với SQPOLL:
  kernel thread tự poll SQ → ứng dụng có thể đạt GẦN 0 SYSCALL.
```

- **Completion-based**: không hỏi "sẵn sàng chưa" — nộp *việc*, nhận *kết quả*. Thống nhất network + file + fsync + accept + timeout... trong một interface.
- Batch tự nhiên: 1 lần enter nộp 100 lệnh, gặt 100 kết quả — chia phí syscall cho 100.
- Registered buffers/files: đăng ký trước, bỏ phí map/check mỗi lần.

Benchmark đại diện (echo server, 1 thread): epoll ~200-400k req/s; io_uring ~1.5-2x epoll ở IO nhỏ dồn dập; với file IO ngẫu nhiên NVMe, io_uring + registered buffer tiệm cận năng lực thiết bị mà thread-pool read() thường không chạm tới. Đổi lại: API phức tạp hơn hẳn, lifecycle buffer khó (buffer phải sống tới khi completion!), từng có nhiều CVE những năm đầu (nhiều môi trường container/seccomp chặn mặc định — kiểm tra trước khi thiết kế quanh nó).

### 3.6. Bảng tổng kết

| Model | Biết sẵn sàng | Chi phí/vòng | Trần thực tế | Ai dùng |
|---|---|---|---|---|
| Blocking + thread | không cần | context switch | ~nghìn conn | app đơn giản, JDBC... |
| select/poll | quét O(n) mỗi lần | copy + quét toàn bộ | ~nghìn | code cũ |
| epoll | kernel đẩy, O(events) | 1 syscall/batch event + n read | trăm nghìn conn | Nginx, Redis, Go, Node |
| io_uring | completion | ~0-1 syscall/batch việc | triệu IO/s | DB engine, proxy mới, cần file async |

## 4. Cách hoạt động — Go netpoller: epoll giấu sau blocking API

Go cho bạn viết code *như blocking* nhưng chạy *trên epoll* — đáng mổ xẻ vì nó là mô hình lai tinh tế nhất:

```go
conn, _ := listener.Accept()        // trông như blocking
go func() {
    n, _ := conn.Read(buf)          // trông như blocking
}()
```

```
conn.Read khi chưa có data:
  1. fd đã là non-blocking từ lúc Accept → read trả EAGAIN
  2. runtime: gopark goroutine, đăng ký fd vào epoll instance của runtime
     (thread M KHÔNG block — đi chạy goroutine khác)
  3. netpoller (chạy nhờ trong sysmon/scheduler): epoll_wait gom event
  4. event đến → goroutine về run queue → chạy lại → read lần nữa → có data
→ 100k goroutine "blocking read" = vài thread + 1 epoll. Ergonomics của
  blocking, chi phí của event loop — đúng lời hứa ở chương 05.
```

## 5. Trade-off

| Trục | Readiness (epoll) | Completion (io_uring) |
|---|---|---|
| Mô hình tư duy | "báo tôi khi đọc được" — app giữ quyền | "làm hộ tôi" — kernel giữ buffer tới khi xong |
| File IO | vô dụng (luôn "sẵn sàng") | hạng nhất |
| Độ phức tạp | vừa (ET có bẫy) | cao (lifecycle, ổn định API) |
| Syscall/công việc | đã tốt | gần 0 |
| Hệ sinh thái | 20 năm, mọi nơi | đang trưởng thành; bị chặn ở nhiều sandbox |

Và một trade-off tổng: **mọi bước tiến về throughput đều trả bằng độ phức tạp lập trình** — blocking (dễ nhất) → epoll callback/state machine (khó nhất) → runtime giấu epoll (dễ lại, trả bằng phức tạp runtime) → io_uring (khó, chưa có "Go của io_uring" phổ cập).

## 6. Production

- Đo mật độ syscall: `perf trace -s` hoặc `syscount-bpfcc` — service event-loop khỏe mạnh có tỷ lệ (event xử lý)/(syscall) cao. Nghìn `epoll_wait` trả 1-2 event mỗi lần = vòng loop bị đánh thức vụn (xem: timeout quá ngắn, timer dày, healthcheck spam).
- Latency trong loop: một callback chậm chặn **mọi** connection sau nó — Node: event-loop lag metric; Go: `runtime/metrics` scheduler latency; nguyên tắc chung: **không CPU nặng, không blocking call trong thread event loop**.
- Backlog của readiness: socket sẵn sàng nhưng app chậm gặt → data dồn ở socket buffer — `ss -tmi` xem Recv-Q (case backlog chương 16 là ở tầng accept, đây là tầng đọc).
- Kiểm tra io_uring có bị chặn: `io_uring_setup` trả EPERM trong container → seccomp profile; quyết định kiến trúc phải biết trước điều này.

## 7. Anti-pattern

- Trộn CPU-bound vào event loop (nén ảnh trong Node/handler epoll) — mọi connection cùng chịu.
- ET mà không đọc cạn EAGAIN; hoặc LT mà không bao giờ đọc → wakeup bão hòa CPU.
- Poll với timeout 0 trong vòng lặp ("cho realtime") — 100% CPU vô nghĩa; dùng timeout/timerfd.
- Mỗi request một `epoll_create` (đã thấy thật!) — một instance sống cả đời process là đủ.
- Chọn io_uring vì hype khi service của bạn làm 3k req/s — chi phí phức tạp không mua lại được gì (mục 9).

## 8. Failure Analysis — case mẫu: CPU 100% mà throughput bằng 0 tăng trưởng

- **Triệu chứng**: sau khi thêm tính năng "đọc nhanh", một worker Nginx-like tự viết ăn 100% CPU cả khi tải thấp.
- **Điều tra**: `strace -c` (bản staging): 2 triệu `epoll_wait`/phút, đa số trả về ngay với cùng 1 fd; fd đó là một pipe nội bộ.
- **Root cause**: pipe được thêm vào epoll ở chế độ LT, phía đọc chỉ đọc khi "rảnh" — pipe luôn có data → epoll_wait trả về ngay lập tức mãi mãi → busy loop hoàn hảo.
- **Kernel view**: ep_poll thấy ready list không rỗng → không ngủ — kernel làm đúng hợp đồng LT.
- **Fix**: đọc cạn pipe trong handler, hoặc chuyển fd đó sang ET. **Prevention**: mọi fd trong epoll phải có handler đọc-cạn tương ứng; metric "event/wakeup ratio" trên dashboard.

## 9. Khi nào không nên tối ưu

Dưới ~1k concurrent connection và CPU không nghẽn vì syscall: blocking + pool (hoặc để Go/Java Virtual Threads lo) là lựa chọn đúng — đọc được, debug được, ít bug. epoll thủ công chỉ khi viết infrastructure (proxy, gateway, runtime). io_uring chỉ khi đo được: (a) syscall overhead chiếm % CPU đáng kể ở tải đích, hoặc (b) cần file IO async thật sự. "Chuyển sang io_uring" không phải kế hoạch tăng trưởng — "giảm số IO cần làm" (cache, batch, pipeline) hầu như luôn thắng trước.

---

**Chương tiếp theo**: IO model là cách chờ — giờ xem chính dữ liệu network đi qua kernel thế nào: [Chương 11: Network Stack](/series/linux-os-for-backend/11-network-stack/).
