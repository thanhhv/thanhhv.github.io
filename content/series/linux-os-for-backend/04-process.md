+++
title = "Chương 04 — Process: đơn vị của ảo ảnh \"máy tính riêng\""
date = "2026-02-21T11:00:00+07:00"
draft = false
tags = ["backend", "linux", "operating-system"]
series = ["Operating Systems cho Backend Engineer"]
+++

## 1. Problem Statement

CPU chỉ có một dòng lệnh; RAM là một khối phẳng. Nhưng ta muốn chạy 500 chương trình "đồng thời", mỗi cái tưởng mình sở hữu cả máy. **Process là câu trả lời: một gói đóng kín gồm (code + data + trạng thái CPU + tài nguyên) mà kernel có thể đóng băng, hồi sinh, cô lập, và thu hồi.**

Nếu không có process: không chạy được nhiều chương trình; không giết được chương trình treo; một chương trình lỗi kéo sập tất cả; không có khái niệm "quyền của ai".

## 2. Tại sao nó tồn tại

- **Business problem**: một máy chủ đắt tiền phải chạy được nhiều dịch vụ; thời gian CPU chờ IO của chương trình này phải dùng được cho chương trình khác (tận dụng tài nguyên = tiền).
- **Hardware limitation**: CPU không có khái niệm "nhiều chương trình" — kernel phải *tự dựng* khái niệm đó bằng cách save/restore trạng thái.
- **Failure isolation**: ranh giới process là ranh giới lỗi. Nginx worker segfault → chỉ worker đó chết, master fork worker mới.
- **Security**: ranh giới process là ranh giới quyền (UID, capability, namespace).

## 3. Internal Architecture

### 3.1. PCB — trong Linux là `task_struct`

Mọi thứ kernel biết về một process gói trong một struct khổng lồ (~10KB+, định nghĩa tại `include/linux/sched.h` — hàng trăm field). Các nhóm quan trọng:

```c
struct task_struct {                 // rút gọn, chú thích theo nhóm
    // ĐỊNH DANH
    pid_t pid;                       // định danh duy nhất
    pid_t tgid;                      // thread group id = "PID" mà user thấy
    struct task_struct *real_parent; // cha — nền của cây process

    // TRẠNG THÁI CPU (để context switch)
    struct thread_struct thread;     // registers được lưu khi rời CPU
    void *stack;                     // kernel stack riêng (~16KB)

    // SCHEDULING
    int prio;                        // ưu tiên
    struct sched_entity se;          // node trong cây đỏ-đen của CFS/EEVDF
    unsigned int policy;             // SCHED_NORMAL / FIFO / RR / ...

    // MEMORY
    struct mm_struct *mm;            // KHÔNG GIAN ĐỊA CHỈ — page table, các vùng VMA
                                     // threads cùng process TRỎ CHUNG một mm

    // FILES
    struct files_struct *files;      // bảng file descriptor

    // SIGNALS, CREDENTIALS, NAMESPACES, CGROUPS...
    struct signal_struct *signal;
    const struct cred *cred;         // uid, gid, capabilities
    struct nsproxy *nsproxy;         // namespaces (nền tảng container)
};
```

**Insight quan trọng nhất của chương**: trong Linux, *không có struct riêng cho thread*. Thread và process đều là `task_struct` — khác nhau duy nhất ở chỗ **chia sẻ hay sở hữu riêng** các tài nguyên (`mm`, `files`...). Điều này sẽ làm chương 05 trở nên rất ngắn gọn về mặt khái niệm.

### 3.2. Address space — `mm_struct` và VMA

Mỗi process có một `mm_struct`: con trỏ tới page table (giá trị nạp vào CR3) + danh sách **VMA** (Virtual Memory Area) — các "vùng hợp lệ" của không gian địa chỉ:

```
$ cat /proc/<pid>/maps        (mỗi dòng là một VMA)
00400000-00452000 r-xp  .../myserver     ← .text: code, đọc + thực thi
00651000-00652000 rw-p  .../myserver     ← .data: biến toàn cục
00e5a000-00e7b000 rw-p  [heap]           ← heap, lớn lên khi malloc/brk
7f3a...           r-xp  .../libc.so.6    ← thư viện dùng chung (map 1 lần trong RAM)
7ffd...           rw-p  [stack]          ← stack, lớn xuống
7ffd...           r-xp  [vdso]           ← vDSO (chương 03)
```

Truy cập địa chỉ ngoài mọi VMA → page fault → kernel không tìm thấy VMA → **SIGSEGV**. Truy cập trong VMA nhưng chưa có page vật lý → kernel lặng lẽ cấp page (minor fault) — đó là chuyện bình thường hàng nghìn lần mỗi giây (chương 07).

### 3.3. Trạng thái process

```
                 fork()
                   │
                   ▼
   ┌────────► TASK_RUNNING ◄────────────┐
   │        (đang chạy hoặc trong        │
   │         run queue chờ CPU)          │ wake_up
   │                │                    │
   │ được cấp CPU   │ chờ IO/lock/event  │
   │                ▼                    │
   │      TASK_INTERRUPTIBLE (S) ────────┤  ngủ, đánh thức được bằng signal
   │      TASK_UNINTERRUPTIBLE (D) ──────┘  ngủ, KHÔNG đánh thức được — thường
   │                │                       là đang chờ disk IO / NFS
   │                ▼ exit()
   └──────── EXIT_ZOMBIE (Z) ── cha gọi wait() ──► biến mất hoàn toàn
```

Hai trạng thái đáng chú ý trong production:

- **D (uninterruptible sleep)**: process kẹt trong kernel chờ IO — `kill -9` cũng không chết (signal chỉ xử lý được khi process quay về ranh giới user). Nhiều process D kéo dài = disk/NFS có vấn đề. **D cũng được tính vào load average** — lý do load cao mà CPU idle (case chương 15).
- **Z (zombie)**: đã chết nhưng chưa được "khai tử" — xem mục 4.4.

## 4. Cách hoạt động — vòng đời

### 4.1. fork() — nhân bản để sáng thế

Mô hình tạo process của Unix kỳ lạ mà thiên tài: không có syscall "tạo process mới từ đầu". Chỉ có **fork()** — nhân bản chính mình:

```c
pid_t pid = fork();
if (pid == 0)      { /* con: fork() trả 0 */ }
else if (pid > 0)  { /* cha: fork() trả PID con */ }
```

Một lần gọi, **trả về hai lần** — ở hai process khác nhau. Điều gì xảy ra trong kernel:

```
sys_fork → kernel_clone() (kernel/fork.c)
  1. Cấp task_struct mới, copy từ cha
  2. Cấp PID mới
  3. Copy/chia sẻ tài nguyên theo flags:
     - mm: COPY page table, nhưng KHÔNG copy memory!
       → mọi page đánh dấu read-only ở CẢ HAI bên = Copy-on-Write
     - files: copy bảng FD (hai bên trỏ chung file mở — chung offset!)
  4. Đặt con vào run queue → trả về
```

**Copy-on-Write (COW)** là điểm mấu chốt: fork một process 10GB chỉ tốn vài ms (copy page table, không copy 10GB). Khi bên nào *ghi* vào page nào, page fault → kernel copy đúng page đó (4KB) → hai bên tách ra dần. Trade-off của COW lộ ra ở production: **Redis fork để BGSAVE** — nếu trong lúc con đang snapshot mà cha ghi dữ liệu dồn dập, mỗi lần ghi trúng page chung là một lần copy 4KB → memory usage có thể tăng gần gấp đôi + latency spike. Đây là lý do Redis khuyên tắt Transparent Huge Pages (THP): page 2MB làm mỗi COW copy 2MB thay vì 4KB (chương 17).

### 4.2. exec() — thay linh hồn giữ thể xác

`execve(path, argv, envp)` **thay toàn bộ nội dung** process bằng chương trình mới: vứt address space cũ, nạp ELF mới, giữ nguyên PID, FD (trừ khi có `O_CLOEXEC`), quan hệ cha-con. Cặp fork+exec là cách shell chạy mọi lệnh:

```
shell: fork() ──► con: (tùy chọn: đổi stdin/stdout bằng dup2 — nền của pipe/redirect)
                  con: execve("/usr/bin/ls") ──► thành ls
shell: wait()  ◄── ls exit
```

Việc tách đôi fork/exec (thay vì một syscall "spawn") cho phép con **chỉnh môi trường của chính mình** giữa hai bước — pipe, redirect, drop privilege, setup namespace (container runtime làm chính xác điều này). Trade-off: fork ngày càng đắt với process lớn (copy page table của 100GB heap vẫn tốn hàng trăm ms) → tồn tại `posix_spawn` và `vfork`; Go runtime dùng `fork+exec` qua `syscall.ForkExec` với rất nhiều cẩn trọng, và các JVM lớn từng gặp sự cố OOM chỉ vì fork để chạy một script nhỏ (overcommit heuristics — chương 07).

### 4.3. wait() và exit code

Khi process exit, kernel giữ lại `task_struct` tối thiểu (PID, exit code, số liệu tài nguyên) để cha đọc qua `wait()/waitpid()`. Đây là cách shell biết `$?`, cách CI biết test pass/fail, cách Kubernetes biết container "Completed" hay "Error".

### 4.4. Zombie và Orphan

- **Zombie**: con đã exit nhưng **cha chưa gọi wait()**. Xác chết chiếm một PID. Vài zombie vô hại; hàng chục nghìn zombie → **cạn PID** (`fork: Resource temporarily unavailable`) — sự cố thật, thường do app tự spawn con mà quên `SIGCHLD` handler/wait. Không thể kill zombie (nó chết rồi); phải sửa cha hoặc kill cha.
- **Orphan**: cha chết trước con. Kernel cho con "tái định cư" về `init` (PID 1) hoặc subreaper gần nhất — init có nhiệm vụ wait() hộ. **Bẫy container kinh điển**: PID 1 trong container là app của bạn, không phải init thật → app không biết wait hộ orphan → zombie tích tụ. Fix: `docker run --init` / tini / dùng entrypoint biết reap.
- **Daemon**: process nền, tự cắt khỏi terminal (double-fork, setsid). Thời hiện đại: đừng tự daemon hóa — để systemd/container runtime quản lý foreground process, log ra stdout.

### 4.5. Minh họa Go — chạy process con đúng cách

```go
cmd := exec.Command("pg_dump", "mydb")
cmd.Stdout = outFile
// Dưới nắp: Go gọi fork (thực chất clone) + execve.
// cmd.Wait() BẮT BUỘC — chính là wait() tránh zombie.
if err := cmd.Start(); err != nil { ... }
err := cmd.Wait() // đọc exit code + giải phóng zombie
```

Quên `Wait()` trong service chạy lâu = leak zombie + leak pipe FD. (Go còn có finalizer đỡ phần nào, nhưng đừng dựa vào.)

## 5. Trade-off

| Quyết định | Được | Mất |
|---|---|---|
| Process = đơn vị cô lập nặng | An toàn, giết được, giới hạn được | Tạo/chuyển đắt hơn thread; IPC phải qua kernel |
| fork + COW | Tạo process nhanh bất kể size | Ghi sau fork gây fault storm; nguy cơ memory tăng gấp đôi (Redis) |
| fork/exec tách đôi | Linh hoạt tối đa (pipe, container setup) | fork process lớn vẫn đắt; race FD nếu quên O_CLOEXEC |
| Zombie giữ exit code | Cha luôn lấy được kết quả con | Cha vô trách nhiệm → rò PID |
| Mọi thứ là task_struct | Mô hình thống nhất process/thread | Ranh giới "process vs thread" mờ, dễ hiểu sai khi debug (ps vs ps -T) |

## 6. Production

- **Cây process là bản đồ dịch vụ**: `pstree -p`, `ps -ef --forest`. Hiểu ai fork ai = hiểu ai reap ai, signal truyền thế nào, systemd giết cả cgroup ra sao.
- Metric đáng có trên dashboard: **số process/thread toàn máy** (`/proc/loadavg` trường 4, hoặc `ps ax | wc`), **fork rate** (`vmstat` không có; dùng `perf stat -e sched:sched_process_fork -a sleep 10` hoặc node_exporter `node_forks_total`), **số zombie** (`ps -eo stat | grep -c Z`).
- Signal & shutdown: SIGTERM → app đóng graceful (drain connection, flush) → SIGKILL sau grace period. Kubernetes làm đúng trình tự này (mặc định 30s) — app phải bắt SIGTERM, và **PID 1 trong container không có handler mặc định cho SIGTERM** (khác process thường!) — app không bắt là không chết cho tới SIGKILL.
- `/proc/<pid>/` là cửa sổ nhìn vào task_struct: `status` (trạng thái, VmRSS, threads), `maps`, `fd/`, `stack` (kernel stack — quý giá khi process kẹt D-state), `wchan` (đang ngủ chờ gì).

## 7. Anti-pattern

- Fork từ process nhiều thread (không exec ngay): chỉ thread gọi fork được nhân bản, nhưng **lock của thread khác vẫn ở trạng thái đang giữ** trong bản sao → deadlock kinh điển. Chỉ được gọi async-signal-safe function giữa fork và exec.
- Spawn process con để xử lý request (CGI-style) trong hệ thống hiệu năng cao — chi phí fork+exec ~1ms/lần, 1000 req/s = 1 core chỉ để đẻ process.
- Quên wait() → zombie. Quên O_CLOEXEC → con thừa kế FD (đã thấy: process con giữ socket khiến port không giải phóng được sau khi cha chết).
- Tin `kill -9` là thuốc tiên: không giết được D-state; không cho app cơ hội flush → hỏng dữ liệu.

## 8. Failure Analysis — case mẫu: cạn PID vì zombie

- **Triệu chứng**: các dịch vụ trên máy lần lượt lỗi `cannot fork`; SSH được nhưng chạy lệnh chập chờn. CPU/memory bình thường.
- **Điều tra**: `ps -eo stat,ppid,comm | awk '$1~/Z/'` → 31.000 zombie, cùng PPID = một agent giám sát. `cat /proc/sys/kernel/pid_max` = 32768 → cạn.
- **Root cause**: agent spawn script mỗi 10 giây, bug ở handler SIGCHLD sau một bản update → không bao giờ wait().
- **Điều gì xảy ra trong kernel**: mỗi zombie giữ một `struct pid`; `alloc_pid()` hết số → `fork` trả `-EAGAIN` cho *mọi* process trên máy.
- **Fix khẩn cấp**: restart agent (cha chết → zombie về init → init reap sạch trong vài giây). **Prevention**: alert trên số zombie > 100; nâng `pid_max` chỉ là giảm đau, không phải thuốc chữa.

## 9. Khi nào không nên tối ưu

Chi phí tạo process (~0.5-2ms) chỉ thành vấn đề khi tần suất cao. Cron job mỗi phút fork chục process? Chẳng sao cả. Đừng viết lại pipeline shell thành một binary "để đỡ tốn fork" khi nó chạy 5 lần/ngày. Ngược lại, đừng dùng process-per-request khi ngưỡng vượt vài trăm request/giây — nhưng lúc đó thứ nên đổi là *kiến trúc* (worker pool, thread, event loop), không phải tuning fork.

---

**Chương tiếp theo**: process cô lập tốt nhưng nặng — khi cần nhiều dòng thực thi *chia sẻ* dữ liệu, ta cần [Chương 05: Thread](/series/linux-os-for-backend/05-thread/).
