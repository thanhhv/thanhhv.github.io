+++
title = "Operating Systems cho Backend Engineer — Linux Internals từ First Principles"
date = "2026-02-21T07:00:00+07:00"
draft = false
tags = ["backend", "linux", "operating-system"]
series = ["Operating Systems cho Backend Engineer"]
+++

> Bộ tài liệu chuyên sâu về hệ điều hành, tập trung vào Linux, viết cho Backend Engineer, Senior Backend Engineer, Tech Lead và Architect. Mục tiêu không phải là học lệnh Linux — mà là hiểu **tại sao** hệ thống hoạt động như vậy, **điều gì xảy ra bên trong kernel** khi backend của bạn chạy trong production, và **trade-off** đằng sau mỗi quyết định thiết kế.

---

## Cách đọc bộ tài liệu này

Mỗi chương đi theo một template thống nhất:

1. **Problem Statement** — thành phần này giải quyết bài toán gì, nếu không có thì sao.
2. **Tại sao nó tồn tại** — business problem, hardware limitation, performance problem, security problem.
3. **Internal Architecture** — memory layout, kernel space / user space, control flow, data flow.
4. **Cách hoạt động** — đi từng bước, từ application xuống hardware và quay lại.
5. **Trade-off** — latency, throughput, fairness, complexity, CPU, memory, cache.
6. **Production** — monitoring, tracing, profiling, debugging, tuning.
7. **Anti-pattern** — sai lầm phổ biến.
8. **Failure Analysis** — root cause, metric, tracing, fix, prevention.
9. **Khi nào không nên tối ưu** — nhận diện premature optimization.

Diagram trong tài liệu dùng ASCII để đọc được ở mọi nơi. Code minh họa dùng **C** khi cần chạm vào syscall trực tiếp, và **Go** khi minh họa từ góc nhìn backend. Số liệu benchmark là **số đại diện (order of magnitude)** trên phần cứng x86-64 hiện đại — con số tuyệt đối thay đổi theo máy, nhưng tỷ lệ giữa chúng thì ổn định và đó mới là thứ cần nhớ.

---

## Bản đồ tư duy của toàn bộ tài liệu

Mọi chương đều bám theo chuỗi suy luận này:

```
Business Problem: một dịch vụ backend cần phục vụ hàng nghìn request/giây,
ổn định, an toàn, tận dụng hết phần cứng
        │
        ▼
Một chương trình muốn chạy ──► nhưng CPU chỉ biết fetch–decode–execute,
        │                      không biết "process" là gì
        ▼
Memory không an toàn ──► mọi chương trình đều thấy toàn bộ RAM,
        │                một con trỏ lỗi giết cả hệ thống
        ▼
Các chương trình tranh chấp tài nguyên ──► CPU, RAM, disk, network
        │                                   đều hữu hạn
        ▼
Operating System ra đời ──► một lớp trung gian được TIN CẬY,
        │                    độc quyền quản lý phần cứng
        ▼
Kernel ──► Memory ──► Scheduler ──► Filesystem ──► Network
        │
        ▼
Production ──► monitoring, debugging, tuning
        │
        ▼
Trade-off ──► không có thiết kế nào miễn phí
```

---

## Mục lục

### Level 1 — Computer Fundamentals

| Chương | Nội dung |
|---|---|
| [01 — Từ bài toán kinh doanh đến hệ điều hành](/series/linux-os-for-backend/01-tu-bai-toan-den-os/) | Vì sao OS tồn tại, dựng lại OS từ con số 0 |
| [02 — Computer Fundamentals](/series/linux-os-for-backend/02-computer-fundamentals/) | CPU, Register, Cache L1/L2/L3, Pipeline, Branch Prediction, Interrupt, DMA, Memory Bus, NUMA |

### Level 2–3 — Operating System & Linux Kernel

| Chương | Nội dung |
|---|---|
| [03 — Kernel và Syscall](/series/linux-os-for-backend/03-kernel-va-syscall/) | User Space vs Kernel Space, Syscall, Trap, Interrupt, Exception, Kernel Module, Monolithic vs Microkernel |
| [04 — Process](/series/linux-os-for-backend/04-process/) | task_struct (PCB), PID, Address Space, Lifecycle, fork, exec, wait, Zombie, Orphan, Daemon |
| [05 — Thread](/series/linux-os-for-backend/05-thread/) | Kernel Thread, User Thread, Pthread, Goroutine, Context Switching |
| [06 — CPU Scheduling](/series/linux-os-for-backend/06-cpu-scheduling/) | CFS, EEVDF, Round Robin, Priority, Real-time, CPU Affinity, Load Balancing |
| [07 — Memory Management](/series/linux-os-for-backend/07-memory-management/) | Virtual Memory, Page Table, MMU, TLB, Page Fault, Copy-on-Write, Huge Page, Swap, OOM Killer, Fragmentation |
| [08 — Synchronization](/series/linux-os-for-backend/08-synchronization/) | Atomic, Spinlock, Mutex, Futex, Semaphore, RWLock, Condition Variable |
| [09 — Filesystem](/series/linux-os-for-backend/09-filesystem/) | VFS, ext4, XFS, Inode, Journal, Page Cache, fsync |
| [10 — IO Models](/series/linux-os-for-backend/10-io-models/) | Blocking IO, Non-blocking IO, select, poll, epoll, io_uring |
| [11 — Network Stack](/series/linux-os-for-backend/11-network-stack/) | Socket, TCP/UDP trong kernel, Buffer, Backlog, NIC, Zero Copy, sendfile, splice |

### Level 4 — Performance Engineering

| Chương | Nội dung |
|---|---|
| [12 — Linux Performance Tools](/series/linux-os-for-backend/12-performance-tools/) | perf, eBPF, ftrace, strace, ltrace, vmstat, iostat, sar, top/htop — **cơ chế hoạt động**, không chỉ cách dùng |
| [13 — Container](/series/linux-os-for-backend/13-container/) | Namespace, cgroup, OverlayFS, Docker Runtime, OCI — vì sao container không phải VM |
| [14 — Security](/series/linux-os-for-backend/14-security/) | Capability, seccomp, SELinux, AppArmor |

### Level 5 — Production Debugging

| Chương | Nội dung |
|---|---|
| [15 — Failure Cases: CPU & Memory](/series/linux-os-for-backend/15-failure-cases-cpu-memory/) | CPU 100%, Load Average tăng, Context Switch quá nhiều, OOM Killer, Memory Leak, Swap Thrashing, Page Fault Storm, NUMA Imbalance, Cache Miss |
| [16 — Failure Cases: IO, Network & Concurrency](/series/linux-os-for-backend/16-failure-cases-io-network-concurrency/) | FD Exhausted, TCP Backlog Full, SYN Flood, Disk IO 100%, fsync chậm, inode đầy, Epoll Bug, Thread Explosion, Fork Bomb, Zombie, Deadlock, Livelock, Priority Inversion |
| [17 — Kiến trúc thực tế](/series/linux-os-for-backend/17-kien-truc-thuc-te/) | Linux ảnh hưởng thế nào tới PostgreSQL, Redis, Kafka, Nginx, Go Runtime, Node.js, Docker, Kubernetes — sysctl quan trọng và trade-off khi tuning |

---

## Các con số mọi Backend Engineer nên thuộc

Đây là "latency numbers" — nền tảng cho mọi phân tích performance trong tài liệu:

```
Thao tác                              Thời gian       So sánh (nếu 1 CPU cycle = 1 giây)
─────────────────────────────────────────────────────────────────────────────
1 CPU cycle (~3-5 GHz)                ~0.3 ns         1 giây
L1 cache hit                          ~1 ns           ~3 giây
L2 cache hit                          ~4 ns           ~13 giây
L3 cache hit                          ~15-40 ns       1-2 phút
Main memory (DRAM) access             ~60-100 ns      4-5 phút
Atomic CAS (contended)                ~20-100 ns      1-5 phút
Syscall (getpid, đo round-trip)       ~50-300 ns      3-15 phút
Context switch (process)              ~1-10 µs        1-10 giờ   (+ cache pollution)
Mutex lock/unlock (uncontended)       ~15-25 ns       ~1-2 phút  (futex, không syscall)
Mutex (contended, phải vào kernel)    ~1-5 µs         1-5 giờ
Đọc 4KB từ NVMe SSD                   ~20-100 µs      1-5 ngày
Đọc 4KB từ Page Cache                 ~1 µs           1 giờ
fsync lên SSD                         ~0.1-5 ms       tuần → tháng
Round-trip trong cùng datacenter      ~100-500 µs     tuần
Đọc 1MB tuần tự từ RAM                ~20-50 µs       1-2 ngày
Đọc 1MB tuần tự từ NVMe               ~200-500 µs     1-2 tuần
Round-trip liên lục địa (VN → US)     ~150-200 ms     ~20 năm
```

Ba kết luận rút ra ngay từ bảng này — và sẽ lặp lại xuyên suốt tài liệu:

1. **RAM chậm hơn CPU ~100 lần** → toàn bộ thiết kế cache, TLB, NUMA, prefetch tồn tại để che khoảng cách này.
2. **Kernel entry đắt hơn function call ~50-100 lần** → toàn bộ thiết kế futex, epoll, io_uring, vDSO tồn tại để né syscall.
3. **Disk/network chậm hơn RAM hàng nghìn lần** → toàn bộ thiết kế Page Cache, buffer, batching, zero-copy tồn tại để giấu độ trễ IO.

Hiểu ba điều này là hiểu 80% lý do Linux được thiết kế như hiện tại.
