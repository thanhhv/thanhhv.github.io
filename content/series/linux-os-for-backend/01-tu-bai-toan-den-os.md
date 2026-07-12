+++
title = "Chương 01 — Từ bài toán kinh doanh đến hệ điều hành"
date = "2026-02-21T08:00:00+07:00"
draft = false
tags = ["backend", "linux", "operating-system"]
series = ["Operating Systems cho Backend Engineer"]
+++

## 1. Problem Statement

Bạn có một dịch vụ backend: nhận HTTP request, đọc database, trả JSON. Bài toán kinh doanh rất đơn giản: **phục vụ nhiều người dùng nhất có thể, nhanh nhất có thể, trên phần cứng rẻ nhất có thể, và không được sập.**

Bây giờ hãy thử làm điều đó **không có hệ điều hành**. Bài tập tư duy này là nền của toàn bộ tài liệu: mỗi lần bạn gặp một cơ chế kernel phức tạp (page table, futex, epoll...), hãy quay lại đây và hỏi — *nếu không có nó, mình phải tự làm gì?*

## 2. Dựng lại thế giới không có OS

### 2.1. CPU không biết "process" là gì

CPU là một cỗ máy trạng thái rất đơn giản. Vòng đời của nó chỉ có một việc:

```
loop forever:
    instruction = memory[RIP]      # fetch lệnh tại địa chỉ trong thanh ghi RIP
    decode(instruction)            # giải mã
    execute(instruction)           # thực thi: tính toán, đọc/ghi memory, nhảy
    RIP = next                     # sang lệnh kế tiếp
```

Không có khái niệm "chương trình A" hay "chương trình B". Không có khái niệm "file", "user", "quyền". Chỉ có **một dòng lệnh duy nhất chảy qua RIP**. Nếu bạn muốn chạy 2 chương trình, CPU không giúp gì cả — nó chỉ chạy lệnh tại nơi RIP đang trỏ tới.

Trên máy không có OS (bare metal — cách firmware, bootloader, hoặc microcontroller hoạt động), chương trình của bạn LÀ toàn bộ máy tính. Muốn đọc phím? Tự poll thanh ghi của keyboard controller. Muốn ghi disk? Tự lập trình NVMe controller qua MMIO. Muốn chạy 2 việc song song? Tự cắt nhỏ công việc và tự nhảy qua lại.

### 2.2. Memory không an toàn

Không có OS, mọi đoạn code đều thấy **toàn bộ physical memory**. Hệ quả:

```
Chương trình A                    Physical RAM               Chương trình B
─────────────                  ┌──────────────┐             ─────────────
int *p = (int*)0x4000;         │ 0x0000: ...  │
*p = 42;   ────────────────────► 0x4000: 42   ◄──────────── đây là biến số dư
                               │              │             tài khoản của B!
                               └──────────────┘
```

Một con trỏ lỗi trong service A ghi đè dữ liệu của service B. Một buffer overflow trong đoạn parse JSON ghi đè lên chính code đang chạy. Không có ranh giới nào cả — **lỗi của một chương trình là lỗi của toàn hệ thống**. Về mặt kinh doanh: bạn không bao giờ dám chạy code của hai team khác nhau (hay của khách hàng!) trên cùng một máy.

### 2.3. Tài nguyên tranh chấp

Giả sử bạn chấp nhận rủi ro và nhét 2 chương trình vào một máy. Ngay lập tức:

- **CPU**: ai chạy trước? Nếu A chạy `while(1)` thì B **không bao giờ** được chạy — không có cơ chế nào giật CPU lại từ A.
- **Disk**: A và B cùng ghi vào sector 100. Ai thắng? Dữ liệu nát.
- **Network**: packet đến NIC — của A hay của B? NIC không biết, nó chỉ là phần cứng.
- **RAM**: A cần thêm memory. Vùng nào đang trống? Ai theo dõi?

Mọi câu hỏi trên đều cần một **trọng tài** — một thực thể đứng trên tất cả chương trình, được quyền quyết định, và không thể bị chương trình thường qua mặt.

### 2.4. Suy ra các yêu cầu thiết kế

Từ ba vấn đề trên, ta *suy ra* (chứ không phải học thuộc) những gì một hệ điều hành phải có:

| Vấn đề | Yêu cầu | Cơ chế Linux tương ứng |
|---|---|---|
| CPU chỉ chạy 1 dòng lệnh | Ảo hóa CPU: mỗi chương trình tưởng mình có CPU riêng | **Process, Context Switch, Scheduler** (chương 04, 06) |
| `while(1)` chiếm CPU vĩnh viễn | Phải giật lại CPU được mà không cần chương trình hợp tác | **Timer Interrupt + Preemption** (chương 02, 06) |
| Memory ai cũng thấy hết | Ảo hóa memory: mỗi chương trình có không gian địa chỉ riêng, cô lập | **Virtual Memory, Page Table, MMU** (chương 07) |
| Chương trình có thể phá trọng tài | Phần cứng phải phân biệt "code trọng tài" và "code thường" | **CPU Privilege Ring, Kernel/User Mode** (chương 03) |
| Chương trình cần nhờ trọng tài làm việc | Một cổng giao tiếp có kiểm soát | **Syscall** (chương 03) |
| Disk là mảng sector thô | Trừu tượng hóa: file, thư mục, quyền | **VFS, Filesystem** (chương 09) |
| NIC chỉ nhận frame thô | Trừu tượng hóa: socket, connection | **Network Stack** (chương 11) |
| Nhiều luồng cùng sửa 1 dữ liệu | Cơ chế loại trừ lẫn nhau | **Atomic, Lock, Futex** (chương 08) |

**Đây chính là định nghĩa thực chất của hệ điều hành**: không phải "phần mềm quản lý máy tính" (định nghĩa sách giáo khoa, đúng nhưng vô dụng), mà là:

> Một lớp trung gian **được phần cứng bảo vệ**, độc quyền truy cập tài nguyên vật lý, và cung cấp cho mỗi chương trình một **ảo ảnh**: rằng nó có CPU riêng, memory riêng, liên tục và an toàn — trong khi thực tế mọi thứ đều được chia sẻ và tranh chấp.

Kernel bán ảo ảnh. Backend engineer giỏi là người biết ảo ảnh đó **rò rỉ ở đâu** — và đó chính là lúc production có sự cố.

## 3. Internal Architecture — bức tranh toàn cảnh

```
┌───────────────────────────────────────────────────────────────────┐
│                         USER SPACE (Ring 3)                       │
│                                                                   │
│  ┌──────────┐  ┌──────────┐  ┌─────────┐  ┌────────────────────┐ │
│  │ nginx    │  │ postgres │  │ your Go │  │ mọi process khác   │ │
│  │          │  │          │  │ service │  │                    │ │
│  └────┬─────┘  └────┬─────┘  └────┬────┘  └─────────┬──────────┘ │
│       │  glibc / Go runtime: wrapper cho syscall    │            │
├───────┼─────────────┼────────────┼──────────────────┼────────────┤
│       ▼             ▼            ▼                  ▼            │
│              SYSCALL INTERFACE (cửa khẩu duy nhất)               │
├───────────────────────────────────────────────────────────────────┤
│                        KERNEL SPACE (Ring 0)                      │
│                                                                   │
│  ┌──────────┐ ┌──────────┐ ┌───────────┐ ┌─────────┐ ┌────────┐ │
│  │ Process  │ │ Memory   │ │ VFS +     │ │ Network │ │ Device │ │
│  │ Scheduler│ │ Mgmt(MM) │ │ Filesystem│ │ Stack   │ │ Drivers│ │
│  └──────────┘ └──────────┘ └───────────┘ └─────────┘ └────────┘ │
├───────────────────────────────────────────────────────────────────┤
│                           HARDWARE                                │
│     CPU (+ MMU, cache)     RAM      NVMe/SSD      NIC      ...   │
└───────────────────────────────────────────────────────────────────┘
```

Hai điểm cần khắc sâu:

1. **Đường kẻ giữa user space và kernel space không phải quy ước phần mềm** — nó được CPU cưỡng chế bằng phần cứng (privilege ring, chương 03). Process không thể "quên" mà chạm vào kernel memory; CPU sẽ fault ngay lập tức.
2. **Mọi thứ backend của bạn làm** — accept connection, đọc file, cấp memory, tạo thread — **đều phải đi qua cửa khẩu syscall**. Kernel là dependency số một của mọi backend, trước cả database.

## 4. Cách hoạt động — một HTTP request đi qua những gì

Để thấy toàn bộ hệ thống phối hợp, hãy theo dấu một request `GET /users/42` vào service Go của bạn:

```
1.  Packet đến NIC ──► NIC ghi vào RAM qua DMA ──► bắn Interrupt
2.  CPU đang chạy process khác, bị ngắt ──► nhảy vào kernel (interrupt handler)
3.  Kernel network stack: xử lý IP ──► TCP ──► xếp data vào socket receive buffer
4.  Kernel thấy có process đang chờ socket này (epoll) ──► đánh thức nó
5.  Scheduler: chọn process của bạn chạy ──► context switch
6.  Go runtime: netpoller nhận event ──► goroutine xử lý request được schedule
7.  Goroutine gọi db.Query() ──► ghi socket tới Postgres ──► syscall write()
8.  Chờ Postgres trả lời ──► goroutine park, thread có thể chạy goroutine khác
9.  Postgres (process khác, có thể máy khác): đọc data ──► Page Cache hit? 
    Nếu miss ──► đọc NVMe ──► chờ IO ──► interrupt khi xong
10. Response quay về theo đúng đường cũ, chiều ngược lại
11. Goroutine của bạn serialize JSON ──► syscall write() vào socket client
12. Kernel TCP: cắt thành segment, NIC gửi đi qua DMA
```

Mười hai bước, **hàng chục lần chuyển giao giữa user và kernel**, ít nhất 2 process, nhiều interrupt, vài context switch. Mỗi bước là một chương trong tài liệu này, và mỗi bước đều có thể là bottleneck trong production:

- Bước 1-3 chậm → xem chương 11 (network stack) và case SYN flood, backlog full.
- Bước 5 chậm → run queue dài, xem chương 06 và case Load Average.
- Bước 8 xử lý sai → thread explosion, xem chương 05.
- Bước 9 miss cache nhiều → xem chương 09 và case Disk IO 100%.

## 5. Trade-off — cái giá của ảo ảnh

Mọi lớp trừu tượng đều có giá. OS không phải ngoại lệ:

| Lợi ích | Cái giá phải trả |
|---|---|
| Cô lập process (an toàn) | Mỗi lần cần kernel giúp phải trả phí syscall (~100-300ns + đuổi cache) |
| Ảo hóa memory | Mỗi lần truy cập memory phải dịch địa chỉ (che bằng TLB — nhưng TLB miss thì đắt) |
| Chia sẻ CPU công bằng | Context switch tốn 1-10µs + làm nguội cache; process không kiểm soát được khi nào bị tước CPU |
| Page Cache che độ trễ disk | Dữ liệu "đã ghi" chưa chắc đã nằm trên disk (xem case fsync — mất data khi mất điện) |
| Trừu tượng hóa socket | Copy dữ liệu user↔kernel, buffer bloat, backlog cần tuning |

Nguyên lý xuyên suốt: **Linux mặc định chọn throughput và tính tổng quát, trả giá bằng latency ở đuôi phân phối (tail latency) và tính dự đoán được.** Phần lớn công việc tuning của backend engineer là dịch chuyển điểm cân bằng này cho khớp workload cụ thể — và phần lớn sự cố production là do ảo ảnh rò rỉ ở điểm cân bằng mặc định.

## 6. Production — hệ quả thực tế của chương này

Ngay từ chương đầu, có thể rút ra vài quy tắc làm việc:

- Khi debug, **luôn hỏi "request của mình đang ở lớp nào?"** — user space (code của bạn, runtime), kernel (syscall, scheduler, network stack), hay hardware (disk, NIC, NUMA)? Công cụ khác nhau cho mỗi lớp: profiler ứng dụng → `perf`/eBPF → `iostat`/hardware counter.
- **Metric hệ thống là ngôn ngữ của kernel.** Load average, context switch rate, page fault rate, iowait — chúng là cách kernel kể cho bạn nghe chuyện gì đang xảy ra bên dưới ảo ảnh.
- **Đừng tin trực giác từ user space.** "Code mình chỉ đọc một biến" — nhưng biến đó có thể nằm ở page chưa map (page fault → disk IO). "Mình chỉ ghi ra file" — nhưng dữ liệu mới nằm ở Page Cache. Sự thật nằm ở kernel.

## 7. Anti-pattern

- **Coi OS như hộp đen "cứ thế mà chạy".** Hộp đen hoạt động tốt cho đến 2 giờ sáng khi OOM killer bắn chết database của bạn và bạn không biết bắt đầu từ đâu.
- **Học thuộc lệnh mà không hiểu cơ chế.** Biết gõ `top` nhưng không hiểu load average tính thế nào thì con số 8.5 không nói được điều gì.
- **Tuning theo blog post.** Copy 20 dòng sysctl từ Internet mà không hiểu từng dòng đổi trade-off nào — nhiều sự cố production bắt nguồn chính từ đây.

## 8. Failure Analysis — dạng tổng quát

Mọi failure case ở chương 15-16 đều quy về một trong ba dạng rò rỉ ảo ảnh:

1. **Ảo ảnh "tài nguyên vô hạn" vỡ**: memory hết (OOM), FD hết, PID hết, disk đầy, backlog đầy. Kernel buộc phải từ chối hoặc giết ai đó.
2. **Ảo ảnh "chỉ có mình mình" vỡ**: hàng xóm ồn ào (noisy neighbor) chiếm CPU/cache/disk bandwidth; lock contention; NUMA imbalance.
3. **Ảo ảnh "thao tác tức thời" vỡ**: ghi file tưởng xong nhưng chưa fsync; memory tưởng cấp rồi nhưng lazy allocation gây page fault storm; TCP tưởng gửi rồi nhưng còn nằm trong buffer.

Khi gặp sự cố lạ, hãy hỏi: *ảo ảnh nào vừa vỡ?* — câu hỏi này thu hẹp không gian điều tra nhanh hơn bất kỳ checklist nào.

## 9. Khi nào không nên tối ưu

Xuyên suốt tài liệu sẽ có mục này ở mỗi chương, vì tối ưu tầng OS là con dao hai lưỡi. Quy tắc chung:

- **Đo trước, tối ưu sau.** Nếu bạn chưa có số liệu chứng minh bottleneck nằm ở tầng OS, thì nó gần như chắc chắn nằm ở tầng ứng dụng: N+1 query, thiếu index, serialize thừa, gọi API tuần tự thay vì song song.
- Thứ tự chi phí/lợi ích thông thường: **thuật toán & IO của ứng dụng → kiến trúc (cache, batch) → cấu hình runtime (GC, pool) → cuối cùng mới đến kernel tuning.**
- Kernel tuning sai có chi phí ẩn rất cao: hệ thống chạy khác mặc định là hệ thống mà engineer mới không hiểu, tài liệu công cộng không áp dụng được, và upgrade kernel có thể thay đổi hành vi ngầm.

---

**Chương tiếp theo**: trước khi hiểu kernel, phải hiểu thứ kernel quản lý — [Chương 02: Computer Fundamentals](/series/linux-os-for-backend/02-computer-fundamentals/): CPU thực sự chạy lệnh thế nào, vì sao cache quyết định performance, và interrupt — trái tim của mọi hệ điều hành.
