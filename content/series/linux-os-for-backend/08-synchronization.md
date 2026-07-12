+++
title = "Chương 08 — Synchronization: cái giá của việc chia sẻ"
date = "2026-02-21T15:00:00+07:00"
draft = false
tags = ["backend", "linux", "operating-system"]
series = ["Operating Systems cho Backend Engineer"]
+++

## 1. Problem Statement

Hai thread cùng chạy `counter++`. Lệnh này thực chất là ba bước: load → add → store. Xen kẽ giữa hai thread → mất update. Đây là **data race** — nguồn bug khó tái hiện bậc nhất, và với CPU hiện đại còn tệ hơn trực giác: compiler đảo lệnh, CPU thực thi out-of-order, mỗi core có store buffer riêng — **không có "thứ tự toàn cục" tự nhiên nào giữa các core cả**. Muốn có, phải *trả tiền để mua* nó, bằng các primitive của chương này.

Nếu không có synchronization primitive: mọi cấu trúc dữ liệu chia sẻ đều hỏng ngầm, hoặc phải quay về single-thread/share-nothing.

## 2. Tại sao nó tồn tại — phần cứng cho gì, kernel thêm gì

- Phần cứng cho **lệnh atomic** (x86: `LOCK CMPXCHG`, `LOCK XADD`...) — thao tác đơn trên một cache line, không thể xen kẽ; và **memory barrier** — ép thứ tự.
- Nhưng atomic chỉ bảo vệ *một word*. Bảo vệ *một đoạn code* (critical section) cần lock — và khi không lấy được lock thì làm gì? **Chờ thế nào** chính là toàn bộ nội dung thiết kế: quay tại chỗ (đốt CPU, latency thấp) hay ngủ (nhường CPU, trả phí đánh thức ~µs). Mọi primitive là một câu trả lời khác nhau cho câu hỏi này.

## 3. Internal Architecture — bậc thang các primitive

### 3.1. Atomic — nền móng

```go
atomic.AddInt64(&counter, 1)   // biên dịch thành LOCK XADD — một lệnh, không lock
```

Chi phí: uncontended ~5-20ns; **contended** (nhiều core cùng cache line): 50-200ns+ vì cache line bouncing (chương 02) — atomic *không miễn nhiễm* với contention, nó chỉ miễn nhiễm với xen kẽ. Kỹ thuật chuẩn khi counter siêu nóng: per-CPU/per-thread counter, đọc thì cộng gộp (kernel dùng percpu counter khắp nơi; Go: sharded counter).

Trên atomic xây được **lock-free structure** (queue, stack — CAS loop). Nhanh khi contention thấp-vừa, nhưng độ phức tạp trí tuệ rất cao (ABA problem, memory ordering) — dùng thư viện đã kiểm chứng, đừng tự viết.

### 3.2. Spinlock — chờ bằng cách quay

```c
while (atomic_cmpxchg(&lock, 0, 1) != 0)
    cpu_relax();   // PAUSE — quay đến khi lấy được
```

Ưu: lấy/nhả ~10-30ns, latency đánh thức = 0. Nhược: **đốt CPU trong khi chờ**; nếu kẻ giữ lock bị preempt thì kẻ chờ quay vô ích cả time slice. Vì vậy: spinlock là công cụ **của kernel** (nơi có thể tắt preemption khi giữ lock, và critical section được kỷ luật giữ cực ngắn — vài trăm ns). Kernel dùng qspinlock (hàng đợi MCS — mỗi CPU quay trên biến riêng, tránh bouncing). **Trong user space, spinlock thuần gần như luôn là sai lầm** — bạn không tắt được preemption; mutex hiện đại đã spin nhẹ sẵn trước khi ngủ (adaptive).

### 3.3. Mutex và Futex — phát minh quan trọng nhất của đồng bộ hóa Linux

Vấn đề: mutex ngủ cần kernel (xếp hàng, đánh thức) — nhưng gọi syscall cho *mọi* lock/unlock thì lock uncontended (trường hợp áp đảo) phải trả ~200ns+ vô ích. **Futex (Fast Userspace muTEX)** tách đôi:

```
lock:   CAS user-space trên một biến int32
        ├─ thành công (không tranh chấp) → XONG. 0 syscall, ~15-25ns ← 95-99% trường hợp
        └─ thất bại → syscall futex(FUTEX_WAIT, &word, giá_trị_đã_thấy)
             kernel: kiểm tra lại word (chống lost wakeup — atomic với hàng đợi)
                     → xếp vào hash bucket theo địa chỉ, ngủ
unlock: đặt word = 0 (user space)
        └─ nếu có người chờ (bit đánh dấu) → futex(FUTEX_WAKE, &word, 1)
```

**Fast path hoàn toàn trong user space; kernel chỉ xuất hiện khi thật sự có tranh chấp.** Đây là hiện thân đẹp nhất của nguyên lý "né syscall" (chương 03). Mọi thứ đứng trên futex: pthread mutex/condvar/rwlock/semaphore/barrier, `sync.Mutex` của Go (semaphore nội bộ → futex), `java.util.concurrent`, và cả `WaitGroup`, `channel` (phần blocking). Khi `strace` thấy bão futex call = lock của bạn *đang tranh chấp thật*, vì uncontended không hiện gì.

- **Semaphore** = đếm N suất (mutex ≈ semaphore(1) + quyền sở hữu) — dùng làm giới hạn concurrency (connection pool chính là semaphore).
- **RWLock**: nhiều reader hoặc một writer. Bẫy: phí lấy lock đọc *cao hơn mutex* (state phức tạp hơn, cache line vẫn bouncing giữa các reader!) — chỉ thắng khi critical section đọc *dài* và writer *hiếm*. Với section ngắn, mutex thường nhanh hơn — kết quả benchmark gây ngạc nhiên số một của chương này. (Kernel giải bài đọc-nhiều bằng **RCU** — reader không lock, gần như 0 chi phí; writer copy rồi публish — nền của route table, dentry cache...)
- **Condition variable**: "ngủ đến khi điều kiện X" — luôn dùng trong vòng `while (!condition) wait(cv, mutex)` vì **spurious wakeup** là hành vi được phép.

## 4. Cách hoạt động — giải phẫu một lần tranh chấp `sync.Mutex`

```
Goroutine G1 giữ lock, G2 gọi Lock():
1. G2: CAS thất bại
2. G2: spin nhẹ vài vòng (adaptive — hy vọng G1 nhả ngay; chỉ spin khi
   có core rảnh và ít kẻ chờ)
3. Vẫn thất bại → Go runtime park G2 (không phải thread block! — semaphore
   của runtime; thread M đi chạy goroutine khác)
   [nếu là pthread/C: đây là lúc futex(FUTEX_WAIT) — thread ngủ trong kernel]
4. G1 Unlock() → thấy có kẻ chờ → hand-off/wake G2
5. Go mutex có "starvation mode": kẻ chờ > 1ms được trao lock trực tiếp
   (FIFO) thay vì cho kẻ mới đến cướp (barging) — đổi throughput lấy fairness,
   đúng bài toán latency tail
```

Chi phí thực nghiệm đại diện (1 triệu lock/unlock):

```
uncontended mutex        : ~15-25 ns/op
2 thread tranh chấp vừa  : ~100-300 ns/op   (cache bouncing + spin)
8 thread tranh chấp nặng : ~1-3 µs/op       (futex wait/wake, context switch)
→ tranh chấp làm lock ĐẮT LÊN 100 LẦN — vấn đề không phải lock, là TRANH CHẤP
```

## 5. Trade-off — bảng chọn công cụ

| Primitive | Chờ bằng | Uncontended | Contended | Dùng khi |
|---|---|---|---|---|
| Atomic | không chờ | ~5-20ns | bouncing | counter, flag, con trỏ swap |
| Spinlock | quay | ~10-30ns | đốt CPU | chỉ kernel / section vài trăm ns |
| Mutex (futex) | spin rồi ngủ | ~15-25ns | ~µs | mặc định cho mọi thứ |
| RWLock | như mutex | đắt hơn mutex | phức tạp | đọc dài, ghi hiếm — phải đo |
| Semaphore | ngủ | ~như mutex | ~µs | giới hạn N suất |
| Channel/queue | ngủ | đắt hơn mutex | có backpressure | truyền quyền sở hữu, pipeline |

Nguyên tắc kiến trúc quan trọng hơn chọn primitive: **giảm chia sẻ thắng mọi tối ưu lock** — shard dữ liệu theo core/goroutine, share-nothing, truyền message thay vì chia sẻ state. Lock granularity: lock thô (đơn giản, nghẽn) vs lock mịn (scale, dễ deadlock, phí nhiều lock) — đi từ thô, mịn hóa *khi đo thấy* contention.

## 6. Production

- **Phát hiện contention**: Go — `runtime.SetMutexProfileFraction(100)` + pprof mutex profile (thấy đích danh lock nào, stack nào); `perf top` thấy `native_queued_spin_lock_slowpath`/`futex_wait` cao; eBPF: `bpftrace -e 'tracepoint:syscalls:sys_enter_futex { @[comm] = count(); }'`; off-cpu profiling thấy stack ngủ trong lock.
- **Race detection**: Go `-race` (chạy trong CI + staging với traffic thật — bắt race theo *đường chạy thực*), C/C++ TSAN. Race là bug "hoạt động 3 năm rồi nổ" — công cụ này rẻ hơn hậu quả hàng nghìn lần.
- Triệu chứng contention trên dashboard: CPU tăng phi tuyến khi thêm load, `%sy` tăng, context switch tăng vọt, throughput *giảm* khi thêm thread (đường cong lật ngược — dấu vân tay của contention).

## 7. Anti-pattern

- Giữ lock qua IO/RPC: một request chậm → mọi request xếp hàng sau lock (thảm họa phổ biến nhất). Lock chỉ ôm thao tác memory.
- Lock ordering không nhất quán → deadlock (case chương 16). Quy ước: luôn lấy theo một thứ tự toàn cục.
- Double-checked locking không có atomic/barrier — sai về memory model dù "chạy được".
- Spinlock user space "cho nhanh" — chết vì preemption.
- Một map + một mutex toàn cục cho toàn service, rồi tăng thread để chữa chậm.
- Dùng channel cho mọi thứ kể cả bảo vệ một counter (chi phí gấp nhiều lần atomic) — hoặc ngược lại, dựng lock tinh vi cho thứ channel diễn đạt tự nhiên.

## 8. Failure Analysis — case mẫu: throughput giảm khi scale từ 16 lên 64 core

- **Triệu chứng**: nâng máy 16→64 core, throughput tăng 20% rồi *giảm* dưới mức máy cũ khi tải cao. CPU 100% nhưng công việc hoàn thành ít hơn.
- **Điều tra**: perf: 45% cycle trong `osq_lock`+futex path; mutex profile chỉ vào lock của LRU cache toàn cục — mỗi GET đều `Lock(); moveToFront(); Unlock()`.
- **Root cause**: 64 core × hàng triệu op/giây trên **một cache line** — thời gian thật đi vào cache bouncing và hàng đợi futex, không phải "công việc". Định luật Amdahl bằng xương thịt: phần tuần tự (critical section) chặn trần scale.
- **Fix**: shard LRU 64 mảnh theo hash key (contention chia 64), đổi "move to front" thành đánh dấu lười. Kết quả: scale gần tuyến tính lại.
- **Prevention**: benchmark ở concurrency dự kiến ×3; mutex profile bật thường trực (chi phí thấp); review mọi cấu trúc "toàn cục + lock".

## 9. Khi nào không nên tối ưu

Lock uncontended rẻ (~20ns) — đổi mutex lấy lock-free ở nơi không có số liệu contention là đổi code dễ hiểu lấy bug memory-ordering. Bậc thang tối ưu đúng: (1) đo thấy contention → (2) thu nhỏ critical section → (3) shard → (4) đổi cấu trúc dữ liệu (RCU-style, immutable) → (5) lock-free — và tuyệt đại đa số dự án nên dừng ở bậc 3.

---

**Chương tiếp theo**: rời CPU và memory, xuống thế giới chậm hơn nghìn lần nhưng bền vững: [Chương 09: Filesystem](/series/linux-os-for-backend/09-filesystem/).
