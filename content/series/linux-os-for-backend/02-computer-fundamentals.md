+++
title = "Chương 02 — Computer Fundamentals: phần cứng mà kernel phải quản lý"
date = "2026-02-21T09:00:00+07:00"
draft = false
tags = ["backend", "linux", "operating-system"]
series = ["Operating Systems cho Backend Engineer"]
+++

## 1. Problem Statement

Kernel không chạy trong chân không — nó chạy trên CPU thật, RAM thật, với những giới hạn vật lý thật. **Mọi quyết định thiết kế lớn của Linux đều là phản ứng trước một giới hạn phần cứng cụ thể.** Không hiểu phần cứng thì mọi giải thích về kernel đều thành học thuộc lòng.

Nếu bỏ qua chương này: bạn sẽ không hiểu vì sao context switch đắt (cache pollution), vì sao lock contention giết throughput (cache line bouncing), vì sao cùng một đoạn code chạy chậm gấp 3 trên máy 2 socket (NUMA), vì sao kernel *có thể* giật CPU từ một vòng lặp vô hạn (timer interrupt).

## 2. CPU — cỗ máy fetch-decode-execute

### 2.1. Register

Register là bộ nhớ nhanh nhất và nhỏ nhất: nằm ngay trong CPU, truy cập trong **0 cycle bổ sung** (so với ~4 cycle cho L1, ~200+ cycle cho RAM). x86-64 có 16 general-purpose register (RAX, RBX, RCX, RDX, RSI, RDI, RBP, RSP, R8–R15) cùng vài register đặc biệt quan trọng với OS:

- **RIP** (instruction pointer): địa chỉ lệnh đang thực thi. *Điều khiển RIP là điều khiển CPU* — context switch về bản chất là thay RIP (cùng các register khác).
- **RSP** (stack pointer): đỉnh stack hiện tại. Mỗi thread có stack riêng → đổi thread = đổi RSP.
- **CR3**: địa chỉ vật lý của page table gốc. *Đổi CR3 = đổi không gian địa chỉ = đổi process* (chương 07).
- **RFLAGS**: cờ trạng thái, gồm cả interrupt-enable flag.

**Hệ quả cho OS**: "trạng thái" của một luồng thực thi tại một thời điểm = giá trị của tập register. Vậy nên kernel có thể "đóng băng" một process bằng cách lưu hết register vào memory, và "hồi sinh" nó bằng cách nạp lại — đó chính là context switch. Không có gì ma thuật: chỉ là save/restore vài trăm byte.

### 2.2. Cache L1/L2/L3 — vấn đề trung tâm của performance hiện đại

Bài toán: từ ~2000 đến nay, tốc độ CPU tăng nhanh hơn tốc độ DRAM rất nhiều. Kết quả là **"memory wall"**: một lần truy cập RAM (~60-100ns) đủ để CPU thực thi 200-500 lệnh. CPU hiện đại về bản chất là cỗ máy chờ memory — cache tồn tại để giấu sự thật đó.

```
        ┌─────────────────────────── CPU die ──────────────────────────┐
        │  Core 0                Core 1                                │
        │ ┌─────────────┐      ┌─────────────┐                         │
        │ │ Registers   │      │ Registers   │   độ trễ    dung lượng  │
        │ │ L1i  L1d    │      │ L1i  L1d    │   ~1ns      32-64KB/core│
        │ │ L2          │      │ L2          │   ~4ns      0.5-2MB/core│
        │ └──────┬──────┘      └──────┬──────┘                         │
        │        └───────┬────────────┘                                │
        │            L3 (shared)                 ~15-40ns   16-128MB   │
        └────────────────┬──────────────────────────────────────────────┘
                         │ memory bus
                     DRAM (RAM)                  ~60-100ns  vài trăm GB
```

Ba sự thật về cache mà backend engineer phải nắm:

**a) Cache làm việc theo cache line 64 byte.** Đọc 1 byte → cả 64 byte quanh nó được kéo lên. Suy ra: duyệt array tuần tự nhanh hơn linked list hàng chục lần (số liệu đại diện: duyệt 1M phần tử int64 tuần tự ~0.5ms; đuổi theo con trỏ ngẫu nhiên ~50-100ms — chênh **100 lần** với cùng độ phức tạp O(n)).

**b) Cache coherence (MESI protocol).** Mỗi core có cache riêng, nhưng chúng phải nhất quán: khi core 0 ghi vào một cache line, mọi bản sao của line đó ở core khác bị vô hiệu (invalidate). Core khác muốn đọc lại phải lấy từ core 0 — giao dịch liên core tốn ~40-100ns. Đây là **lý do vật lý khiến lock contention và atomic contention chậm**: không phải lệnh atomic chậm, mà là cache line "nảy" (bouncing) giữa các core.

**c) False sharing.** Hai biến độc lập nằm chung một cache line, hai thread mỗi bên ghi một biến → line vẫn nảy qua lại như thể tranh chấp thật. Đây là bug hiệu năng kinh điển:

```go
// TỆ: hai counter chung cache line → 2 thread ghi song song CHẬM HƠN 1 thread
type Counters struct {
    a uint64 // byte 0-7
    b uint64 // byte 8-15  ← cùng cache line với a
}
// TỐT: padding để mỗi counter một line (Go: dùng cấu trúc 64-byte aligned)
type Counters struct {
    a uint64
    _ [56]byte // padding cho đủ 64 byte
    b uint64
}
```

Benchmark đại diện (2 thread, mỗi thread increment biến của mình 100M lần): có false sharing ~1.2s, sau padding ~0.15s — **8 lần**.

### 2.3. Pipeline và Branch Prediction

CPU không chạy từng lệnh một. Nó là dây chuyền lắp ráp (pipeline 14-20 tầng) và **superscalar** — mỗi cycle có thể phát nhiều lệnh, thực thi out-of-order. Để pipeline đầy, CPU phải biết *lệnh tiếp theo là gì* — nhưng khi gặp lệnh rẽ nhánh (`if`, vòng lặp), nó chưa biết. Giải pháp: **đoán** (branch prediction) và chạy trước theo nhánh đoán (speculative execution).

- Đoán đúng (98-99% với branch có quy luật): miễn phí.
- Đoán sai: vứt toàn bộ công việc chạy trước, nạp lại pipeline — mất **15-20 cycle**.

Hệ quả thực tế nổi tiếng: xử lý một mảng đã sort với điều kiện `if (x > 128)` nhanh hơn nhiều lần so với mảng chưa sort — cùng dữ liệu, cùng số phép so sánh, chỉ khác độ dự đoán được của branch. Với backend: hot path nên tránh branch khó đoán; đây cũng là lý do các hàm như `likely()/unlikely()` tồn tại trong kernel source (`include/linux/compiler.h`).

Speculative execution cũng chính là gốc của lỗ hổng **Meltdown/Spectre** (2018) — và các bản vá (KPTI) làm syscall đắt lên đáng kể, minh chứng cho việc hardware và OS gắn chặt đến mức nào.

### 2.4. Privilege Rings

x86 có 4 ring; Linux dùng 2: **Ring 0** (kernel — làm được mọi thứ: truy cập mọi memory, ra lệnh IO, đổi CR3, tắt interrupt) và **Ring 3** (user — bị cấm mọi lệnh đặc quyền). Thực thi lệnh đặc quyền ở Ring 3 → CPU tự động trap vào kernel. **Đây là nền tảng phần cứng của toàn bộ khái niệm "bảo vệ" trong OS** — không có ring thì kernel chỉ là một thư viện, không phải trọng tài (chi tiết ở chương 03).

## 3. Interrupt — trái tim của hệ điều hành

### 3.1. Problem statement

CPU chỉ chạy lệnh nối tiếp. Vậy khi packet đến NIC, khi disk đọc xong, khi user gõ phím — làm sao CPU biết? Hai cách:

- **Polling**: CPU lặp hỏi thiết bị "xong chưa?". Đơn giản, latency thấp — nhưng đốt 100% CPU để chờ.
- **Interrupt**: thiết bị chủ động **ngắt** CPU khi có việc. CPU rảnh làm việc khác.

### 3.2. Cách hoạt động

```
Thiết bị (NIC/disk/timer)                    CPU
        │                                     │ đang chạy process A, lệnh thứ N
        │──── kéo tín hiệu IRQ ──────────────►│
        │                                     │ 1. Chạy nốt lệnh hiện tại
        │                                     │ 2. Lưu RIP, RFLAGS vào stack
        │                                     │ 3. Chuyển sang Ring 0
        │                                     │ 4. Tra IDT (Interrupt Descriptor
        │                                     │    Table) → nhảy vào handler
        │                                     │ 5. Handler xử lý (ngắn nhất có thể)
        │                                     │ 6. iret: khôi phục, quay lại
        │                                     │    process A tại lệnh N+1
```

Process A **không hề biết** mình vừa bị ngắt. Đây là điểm mấu chốt: interrupt là cơ chế duy nhất cho phép kernel **giành lại quyền điều khiển mà không cần process hợp tác**.

**Timer interrupt** là interrupt quan trọng nhất với scheduler: một timer phần cứng (LAPIC timer) ngắt CPU định kỳ (hoặc theo lịch động — chế độ `NOHZ`/tickless của Linux hiện đại). Mỗi tick, kernel có cơ hội hỏi: "process hiện tại chạy đủ lâu chưa? Có ai xứng đáng chạy hơn không?" → **preemptive multitasking**. Không có timer interrupt, một `while(1)` sẽ chiếm CPU vĩnh viễn — đúng như thế giới không có OS ở chương 01.

### 3.3. Top half / Bottom half

Interrupt handler chạy với interrupt bị che (masked) — chạy lâu sẽ làm mất interrupt khác và tăng latency toàn hệ thống. Linux tách đôi:

- **Top half (hard IRQ)**: tối thiểu — ack thiết bị, ghi nhận công việc, lên lịch phần còn lại. Vài µs.
- **Bottom half (softirq/tasklet/workqueue)**: phần nặng — ví dụ toàn bộ xử lý TCP/IP của packet chạy trong softirq `NET_RX`. Đây là lý do bạn thấy `%si` (softirq) trong `top`, và process `ksoftirqd/N` ăn CPU khi network dồn dập.

**Production note**: server nhận nhiều traffic mà một core nghẽn 100% ở `si` → interrupt của NIC dồn về một core. Giải pháp: RSS (NIC nhiều queue), RPS/RFS, `irqbalance` — xem chương 11.

### 3.4. Trade-off polling vs interrupt

Interrupt không miễn phí (~1-2µs mỗi lần + cache pollution). Với NIC 10-100Gbps nhận hàng triệu packet/giây, interrupt cho từng packet sẽ nhấn chìm CPU (interrupt storm). Vì thế các hệ thống hiện đại **lai**: NAPI của Linux nhận interrupt đầu tiên rồi *chuyển sang polling* khi tải cao; DPDK/XDP đi xa hơn nữa. io_uring ở tầng ứng dụng cũng theo triết lý tương tự (chương 10). Quy luật chung: **tải thấp → interrupt (tiết kiệm CPU); tải cao → polling (giảm overhead mỗi đơn vị công việc)**.

## 4. DMA — giải phóng CPU khỏi việc bốc vác

Không có DMA (Direct Memory Access), muốn đọc 1MB từ disk, CPU phải tự copy từng word từ thiết bị vào RAM ("programmed IO") — hàng trăm nghìn cycle chỉ để bốc vác. Với DMA:

```
CPU: "Disk controller, đọc block 500-756, ghi vào RAM địa chỉ X, xong thì ngắt tôi"
CPU: → đi chạy process khác
Disk controller: ...tự chuyển dữ liệu vào RAM (không qua CPU)...
Disk controller: → interrupt "xong rồi"
```

CPU chỉ tốn vài trăm cycle setup + một interrupt. **Toàn bộ IO hiện đại (NVMe, NIC) đều là DMA.** Hệ quả kiến trúc quan trọng: dữ liệu network/disk đi vào RAM **không qua CPU cache** → kernel phải quản lý tính nhất quán, và "zero copy" (chương 11) mới có ý nghĩa: điểm tối ưu là loại bỏ những lần CPU copy dữ liệu *giữa các vùng RAM*, vì thiết bị đã tự DMA phần còn lại.

## 5. Memory Bus và NUMA

### 5.1. Vấn đề

Một memory controller không phục vụ nổi 64-128 core. Giải pháp phần cứng: chia máy thành các **NUMA node** — mỗi socket CPU có RAM "địa phương" gắn trực tiếp, truy cập RAM của socket khác phải đi qua interconnect (UPI/Infinity Fabric):

```
┌────────────── Node 0 ─────────────┐   ┌────────────── Node 1 ─────────────┐
│  CPU socket 0 (core 0-31)         │   │  CPU socket 1 (core 32-63)        │
│       │                           │   │       │                           │
│  RAM local:  ~80-100ns            │◄──┼──►RAM local:  ~80-100ns           │
└───────────────────────────────────┘UPI└───────────────────────────────────┘
     truy cập chéo node (remote): ~130-200ns (gấp 1.5-2x) + bandwidth thấp hơn
```

### 5.2. Hệ quả cho OS và backend

NUMA phá vỡ ảo ảnh "RAM là một khối đồng nhất". Kernel phải trả lời: cấp memory cho process ở node nào? Schedule thread ở node nào? Chính sách mặc định của Linux là **first-touch**: page được cấp ở node của CPU *chạm vào nó lần đầu*. Điều này tạo ra bẫy kinh điển:

> Thread khởi tạo (chạy ở node 0) cấp và ghi toàn bộ buffer → toàn bộ memory nằm ở node 0 → 32 worker thread ở node 1 truy cập chéo suốt vòng đời → throughput giảm 30-50% một cách "bí ẩn".

Đây là case **NUMA Imbalance** (chương 15). Các database lớn (PostgreSQL, Redis) đều có khuyến nghị riêng về NUMA; kernel có `numactl`, automatic NUMA balancing, và cgroup cpuset để kiểm soát. Với backend: máy 1 socket không cần quan tâm; máy 2+ socket chạy service latency-sensitive thì **bắt buộc** phải biết (`numactl --hardware`, `numastat`).

## 6. Trade-off tổng hợp của tầng phần cứng

| Cơ chế | Được | Mất |
|---|---|---|
| Cache nhiều tầng | Che memory wall, tăng tốc 10-100x | Coherence traffic, false sharing, hiệu năng khó dự đoán |
| Pipeline + branch prediction | IPC cao (4-6 lệnh/cycle) | Phạt 15-20 cycle khi đoán sai; lỗ hổng Spectre |
| Interrupt | CPU không phải chờ thiết bị | Overhead mỗi lần ngắt; interrupt storm khi tải cao |
| DMA | CPU không bốc vác dữ liệu | Phức tạp về coherence; cần IOMMU để an toàn |
| NUMA | Scale bandwidth theo số socket | Memory không còn đồng nhất; OS và app phải nhận thức topology |

## 7. Production — quan sát tầng phần cứng

Phần cứng có counter riêng (PMU — Performance Monitoring Unit) mà `perf` đọc được:

```bash
perf stat -e cycles,instructions,cache-misses,branch-misses ./app
# IPC (instructions per cycle): < 1 → app đang chờ memory (cache miss)
#                               > 2 → app đang tính toán hiệu quả
perf stat -e LLC-load-misses ./app        # miss L3 → phải xuống RAM
perf c2c record / report                   # tìm false sharing (cache-to-cache)
numastat -p <pid>                          # phân bố memory theo NUMA node
cat /proc/interrupts                       # interrupt phân bố theo core
```

Quy tắc đọc nhanh: **CPU 100% không có nghĩa là CPU đang "tính"** — nó có thể đang stall chờ RAM (vẫn tính là busy). IPC thấp + cache miss cao = bài toán memory layout, không phải bài toán thuật toán.

## 8. Anti-pattern

- Tối ưu thuật toán O(n log n) → O(n) nhưng dùng cấu trúc dữ liệu đuổi con trỏ — thua bản O(n log n) dùng array phẳng.
- Thêm thread để tăng tốc một workload bị giới hạn bởi memory bandwidth — thêm contention, chậm hơn.
- Bỏ qua topology: pin 2 thread giao tiếp nhiều vào 2 NUMA node khác nhau.
- Viết struct chia sẻ giữa các thread mà không nghĩ về cache line layout.

## 9. Khi nào không nên tối ưu

Tối ưu mức cache line, branch, NUMA chỉ đáng làm khi: (1) profiling chỉ đích danh (IPC thấp, cache miss cao ở hot path chiếm >10% thời gian), (2) hot path được gọi hàng triệu lần/giây, (3) đã hết bài ở tầng thuật toán và IO. Một service REST điển hình dành phần lớn thời gian chờ database và network — tối ưu false sharing ở đó là trang trí. Ngược lại, với database engine, message broker, hay proxy tầng L4/L7, đây là cơm áo gạo tiền.

---

**Chương tiếp theo**: phần cứng đã có ring, interrupt, MMU — giờ xem kernel dùng chúng để dựng ranh giới user/kernel và cánh cổng syscall như thế nào → [Chương 03: Kernel và Syscall](/series/linux-os-for-backend/03-kernel-va-syscall/).
