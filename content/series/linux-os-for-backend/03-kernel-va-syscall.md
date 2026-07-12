+++
title = "Chương 03 — Kernel và Syscall: cánh cổng duy nhất vào thế giới đặc quyền"
date = "2026-02-21T10:00:00+07:00"
draft = false
tags = ["backend", "linux", "operating-system"]
series = ["Operating Systems cho Backend Engineer"]
+++

## 1. Problem Statement

Chương 01 kết luận: cần một trọng tài không thể bị qua mặt. Chương 02 cho thấy phần cứng cung cấp nguyên liệu: privilege ring, trap, interrupt. Chương này trả lời: **kernel dựng ranh giới bảo vệ bằng nguyên liệu đó như thế nào, và một chương trình "xin" kernel làm việc qua cơ chế gì.**

Nếu không có ranh giới user/kernel: mọi process đều đọc được memory của nhau và của kernel (mật khẩu, key, dữ liệu khách hàng); mọi process đều ra lệnh trực tiếp cho disk/NIC (phá nát filesystem, giả mạo packet); không thể có multi-tenant, không thể có cloud. Nếu không có syscall: có ranh giới nhưng không có cửa — chương trình không làm được gì hữu ích.

## 2. Tại sao nó tồn tại

- **Security problem**: cô lập là yêu cầu số một. Ranh giới phải do **phần cứng** cưỡng chế — mọi ranh giới thuần phần mềm đều bị code độc qua mặt.
- **Business problem**: cloud và multi-tenancy chỉ tồn tại được vì kernel đảm bảo code của khách hàng A không đụng được khách hàng B.
- **Hardware limitation**: thiết bị vật lý không biết khái niệm "quyền" — ai ghi lệnh vào thanh ghi của NVMe controller thì NVMe làm theo. Phải có độc quyền truy cập.
- **Abstraction problem**: 500 loại NIC khác nhau cần hiện ra với ứng dụng như một interface duy nhất (socket). Kernel là nơi đặt lớp trừu tượng đó.

## 3. Internal Architecture

### 3.1. User space vs Kernel space — chia bằng địa chỉ

Trên x86-64, mỗi process có không gian địa chỉ ảo 48/57-bit, chia đôi:

```
0xFFFFFFFFFFFFFFFF ┌────────────────────────────┐
                   │       KERNEL SPACE         │  Chỉ truy cập được ở Ring 0.
                   │  - kernel code & data      │  GIỐNG NHAU trong mọi process
0xFFFF800000000000 │  - direct map toàn bộ RAM  │  (map sẵn, đổi CR3 không đổi phần này)
                   ├────── vùng không dùng ─────┤
0x00007FFFFFFFFFFF │        USER SPACE          │  Ring 3 truy cập được.
                   │  - stack (lớn xuống)       │  KHÁC NHAU giữa các process.
                   │  - mmap region, libs       │
                   │  - heap (lớn lên)          │
                   │  - .data, .bss             │
0x0000000000000000 │  - .text (code)            │
                   └────────────────────────────┘
```

Kernel được map vào *mọi* process nhưng đánh dấu supervisor-only trong page table. Vì vậy syscall **không phải context switch** — không đổi CR3, chỉ đổi ring. Đây là lựa chọn thiết kế vì tốc độ; cái giá lộ ra năm 2018: Meltdown cho phép user code đọc lén kernel memory qua speculative execution, buộc Linux thêm KPTI (tách page table kernel/user) — làm syscall đắt thêm đáng kể trên CPU cũ. Bài học: **map chung = nhanh, tách riêng = an toàn; Linux đã phải trả giá để đổi bên.**

### 3.2. Kernel là gì về mặt vật lý?

Kernel không phải một process. Nó là **một khối code + data nằm sẵn trong memory**, được thực thi trong ba ngữ cảnh:

1. **Process context**: khi process gọi syscall — kernel code chạy "thay mặt" process đó (vẫn là process đó, chỉ đổi ring và đổi sang kernel stack riêng — mỗi thread có một kernel stack ~16KB).
2. **Interrupt context**: khi hardware ngắt — không thuộc process nào, không được ngủ (sleep).
3. **Kernel thread**: các luồng riêng của kernel (`ksoftirqd`, `kswapd`, `kworker`...) — bạn thấy chúng trong `ps` với tên trong ngoặc vuông.

### 3.3. Monolithic vs Microkernel — trade-off nền tảng

- **Microkernel** (Minix, QNX, seL4): kernel tối thiểu (IPC + scheduling + memory thô); driver, filesystem, network chạy như *process user space*. Được: cô lập lỗi (driver sập không kéo kernel sập), dễ verify. Mất: mỗi thao tác IO thành nhiều lần IPC qua lại — chậm hơn (dù các microkernel hiện đại đã thu hẹp khoảng cách).
- **Monolithic** (Linux): toàn bộ driver, FS, network stack chạy trong kernel space, gọi nhau bằng function call thường. Được: nhanh. Mất: một driver lỗi = kernel panic; bề mặt tấn công lớn.

Linux chọn monolithic vì **performance là yếu tố sống còn năm 1991 và vẫn vậy với server ngày nay**, rồi vá nhược điểm bằng: **kernel module** (nạp/gỡ driver động — `.ko`, không phải cô lập, chỉ là tiện lợi), và gần đây **eBPF** — cho phép nạp code user đã verify vào kernel một cách an toàn (chương 12). Có thể nói eBPF là cách Linux "mượn" ý tưởng mở rộng an toàn của microkernel mà không trả giá IPC.

## 4. Cách hoạt động — giải phẫu một syscall

Theo dấu lệnh Go `os.ReadFile` → cuối cùng là syscall `read(fd, buf, 4096)`:

```
Application (Go)                     ┃ Ring 3
  file.Read(buf)                     ┃
    ↓                                ┃
Go runtime / glibc                   ┃ Chuẩn bị: RAX=0 (số hiệu syscall read)
  syscall wrapper                    ┃ RDI=fd, RSI=&buf, RDX=4096
    ↓                                ┃
  lệnh SYSCALL ━━━━━━━━━━━━━━━━━━━━━╋━━━━━ CPU chuyển Ring 3 → Ring 0 ━━━━
    ↓                                ┃ - RIP cũ lưu vào RCX
Kernel entry (entry_64.S)            ┃ - Nhảy tới địa chỉ trong MSR_LSTAR
  - swapgs, đổi sang kernel stack    ┃ - Lưu registers (pt_regs)
    ↓                                ┃
sys_read() → VFS → filesystem        ┃ Ring 0, process context
  - Page Cache hit?                  ┃
    ├─ Hit: copy data sang user buf  ┃ → quay về gần như ngay
    └─ Miss: phát lệnh đọc NVMe,     ┃
       đánh dấu process TASK_        ┃
       UNINTERRUPTIBLE, gọi          ┃
       schedule() → CHẠY PROCESS     ┃ ← điểm mấu chốt: "blocking" nghĩa là
       KHÁC trong lúc chờ            ┃   process rời CPU, không phải CPU chờ
    ↓ (interrupt: disk xong)         ┃
  đánh thức process, scheduler       ┃
  chọn nó chạy lại                   ┃
    ↓                                ┃
  return; lệnh SYSRET ━━━━━━━━━━━━━━╋━━━━━ Ring 0 → Ring 3 ━━━━━━━━━━━━━━
    ↓                                ┃
Application nhận n bytes             ┃ Ring 3, chạy tiếp
```

Những điểm phải khắc sâu:

1. **User code không bao giờ "gọi hàm" kernel.** Nó thực thi lệnh `SYSCALL` — một trap có kiểm soát. Kernel quyết định làm gì dựa trên số hiệu trong RAX (bảng `sys_call_table`, xem `arch/x86/entry/syscalls/syscall_64.tbl` trong source). Điểm vào là **duy nhất và do kernel kiểm soát** — user không chọn được nhảy vào đâu.
2. **Kernel kiểm tra mọi tham số.** Con trỏ user được xác minh trước khi dùng (`copy_from_user`/`copy_to_user`) — vì user có thể đưa địa chỉ rác hoặc địa chỉ kernel hòng lừa kernel tự đọc chính mình.
3. **"Blocking" là cơ chế nhường CPU**, không phải chờ bận. Một service có 200 thread block trên IO tiêu tốn 0% CPU. Nhầm lẫn "blocking = tốn CPU" là hiểu lầm phổ biến nhất về IO (chương 10 phân tích kỹ).

### 4.1. Chi phí thật của syscall

Chi phí trực tiếp (chuyển ring, save/restore) chỉ ~50-150ns. Chi phí **gián tiếp** mới đáng kể: đuổi một phần cache/TLB/branch predictor state → code user chạy chậm đi *sau khi* syscall trả về. Nghiên cứu kinh điển (FlexSC) đo được ảnh hưởng gián tiếp có thể lớn hơn trực tiếp nhiều lần. Vì vậy chiến lược xuyên suốt của Linux hiện đại là **giảm số lần vượt biên**:

- **vDSO**: các syscall chỉ-đọc siêu phổ biến (`gettimeofday`, `clock_gettime`) được kernel map một trang code+data vào user space — gọi như hàm thường, **không trap**. Đo: `clock_gettime` qua vDSO ~20-25ns, ép trap thật ~200-300ns.
- **futex** (chương 08): lock không tranh chấp → 0 syscall.
- **epoll** (chương 10): 1 syscall lấy N event.
- **io_uring** (chương 10): submit/complete qua ring buffer chia sẻ, gần 0 syscall.

Minh họa bằng Go — đếm syscall thật sự của một chương trình:

```bash
strace -c -f ./myserver   # bảng tổng hợp: syscall nào, bao nhiêu lần, tốn bao lâu
# Với service Go điển hình sẽ thấy: epoll_pwait, read, write, futex chiếm đa số
```

### 4.2. Trap, Exception, Interrupt — phân loại các đường vào kernel

Ba đường vào kernel, khác nhau ở *nguồn* và *tính chủ động*:

| Loại | Nguồn | Ví dụ | Đồng bộ với lệnh? |
|---|---|---|---|
| **Syscall (trap chủ động)** | User code cố ý | `read`, `write`, `mmap` | Có |
| **Exception (trap bị động)** | Lệnh hiện tại gây lỗi | Page fault, chia 0, lệnh cấm | Có |
| **Interrupt** | Thiết bị ngoài | Timer, NIC, disk | Không (bất kỳ lúc nào) |

Cả ba đều đi qua cùng bộ máy: lưu trạng thái → chạy handler Ring 0 → khôi phục. **Page fault là exception quan trọng nhất** — nó không phải lỗi mà là cơ chế vận hành bình thường của virtual memory (chương 07). SIGSEGV nổi tiếng chỉ là trường hợp page fault mà kernel kết luận "truy cập này không hợp lệ".

## 5. Trade-off

| Quyết định | Được | Mất |
|---|---|---|
| Ranh giới ring cưỡng chế phần cứng | An toàn tuyệt đối trước code thường | Mọi dịch vụ kernel đều tốn phí vượt biên |
| Map kernel vào mọi process | Syscall nhanh (không đổi CR3) | Meltdown; KPTI làm chậm lại để vá |
| Monolithic kernel | Nhanh, đơn giản về IPC | Driver lỗi = sập cả máy; bề mặt tấn công lớn |
| Syscall interface ổn định tuyệt đối ("không bao giờ phá userspace" — quy tắc số 1 của Linus) | App cũ chạy mãi trên kernel mới | Kernel phải gánh mọi quyết định API sai lầm trong quá khứ vĩnh viễn |
| vDSO / io_uring / futex né syscall | Giảm mạnh overhead | Tăng độ phức tạp; bề mặt bug mới (xem case Epoll Bug) |

## 6. Production

- **Đo tỷ lệ thời gian kernel vs user**: `top` cột `%sy` vs `%us`. Service tính toán mà `%sy` > 20-30% → nghi vấn: quá nhiều syscall nhỏ (write từng byte?), lock contention (futex), hoặc network stack quá tải.
- **strace** để xem syscall nhưng **cấm dùng thẳng vào production process chịu tải** — strace dùng ptrace, làm chậm target 10-100 lần. Thay bằng `perf trace` hoặc eBPF (`syscount`, `tracepoint:raw_syscalls:sys_enter`) — overhead ~vài %.
- **Đếm syscall nóng nhất**: `perf top` thấy nhiều thời gian trong `entry_SYSCALL_64`, `copy_user_generic` → app đang trả quá nhiều "thuế vượt biên"; nghĩ đến batching (buffered writer, vector IO `writev`, io_uring).
- Kiểm tra vDSO hoạt động: nếu profiling thấy `clock_gettime` là syscall thật (nhiều sample kernel-side) — thường do chạy trong VM với clocksource không ổn định (xen, một số cấu hình kvm) → chỉnh clocksource. Đây là bug hiệu năng thật, hay gặp ở service log timestamp dày đặc.

## 7. Anti-pattern

- Ghi log/write không buffer — mỗi dòng log một syscall `write`. Dùng buffered writer, phí syscall giảm hàng trăm lần.
- Gọi `time.Now()`/`gettimeofday` trong vòng lặp hot mà môi trường không có vDSO hiệu lực.
- Bật `strace` vào process production đang chịu tải (đã thấy sự cố thật: latency tăng 50 lần, cascade timeout).
- Tin rằng "chạy trong kernel = nhanh hơn" — viết kernel module cho logic ứng dụng. Sai cả về hiệu năng lẫn vận hành; nếu thật sự cần, cân nhắc eBPF.

## 8. Failure Analysis — case mẫu: `%sy` cao bất thường

- **Triệu chứng**: CPU 90% nhưng `%us` chỉ 30%, `%sy` 60%. Throughput giảm.
- **Điều tra**: `perf top` → 40% thời gian trong `native_queued_spin_lock_slowpath` (spinlock kernel tranh chấp) gọi từ đường `futex`. → `perf record -g` → hàng chục nghìn futex call/giây từ một mutex nóng trong app.
- **Root cause**: app tăng số thread từ 16 lên 256 để "tăng tốc"; tất cả tranh nhau một lock → chi phí chuyển sang kernel-side (futex slowpath) bùng nổ.
- **Fix**: giảm thread về sát số core; shard dữ liệu để tách lock.
- **Prevention**: dashboard có context-switch rate và tỷ lệ sys/user; load test khi thay đổi mô hình thread.

## 9. Khi nào không nên tối ưu

Đừng săn syscall khi tổng thời gian kernel < 5-10%. Một service làm 2.000 syscall/giây (mức rất bình thường) tốn cỡ 1ms CPU/giây cho việc vượt biên — 0.1%. Chuyển kiến trúc sang io_uring để tiết kiệm 0.1% CPU là premature optimization kinh điển; io_uring đáng giá khi bạn ở mức trăm nghìn IO/giây trở lên.

---

**Chương tiếp theo**: đã có kernel và cổng vào — giờ dựng khái niệm trung tâm: [Chương 04: Process](/series/linux-os-for-backend/04-process/) — đơn vị của ảo ảnh "máy tính riêng".
