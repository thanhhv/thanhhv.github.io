+++
title = "Chương 06 — CPU Scheduling: chia thời gian CPU như thế nào cho công bằng"
date = "2026-02-21T13:00:00+07:00"
draft = false
tags = ["backend", "linux", "operating-system"]
series = ["Operating Systems cho Backend Engineer"]
+++

## 1. Problem Statement

Máy có 32 core; tại một thời điểm có 80 task muốn chạy (runnable). Ai chạy? Chạy bao lâu? Trên core nào? Sai câu trả lời nào cũng có hậu quả: task quan trọng chờ lâu (latency), CPU nhảy task liên tục (mất throughput vì context switch + cache nguội), task nào đó đói vĩnh viễn (starvation), core này 100% core kia rảnh (mất tiền).

Nếu không có scheduler: quay lại chương 01 — `while(1)` chiếm CPU mãi mãi. Scheduler + timer interrupt là bộ đôi biến "một CPU" thành "ảo ảnh nhiều CPU".

## 2. Tại sao nó tồn tại — và vì sao khó

Bài toán scheduling không có lời giải tối ưu tuyệt đối vì các mục tiêu **mâu thuẫn trực tiếp**:

- **Throughput** muốn time slice dài (ít switch, cache ấm).
- **Latency** muốn time slice ngắn (task mới đến được chạy sớm).
- **Fairness** muốn chia đều — nhưng đều theo cái gì? Theo task? Theo user? Theo container?
- **Interactivity**: task ngủ nhiều chờ event (UI, network) cần được ưu ái khi thức dậy — nhưng làm sao phân biệt nó với task lười?

Mọi scheduler là một điểm trên mặt trade-off này. Lịch sử scheduler Linux là lịch sử dịch chuyển điểm đó: O(n) → O(1) (2002, nhanh nhưng đầy heuristic phân loại "interactive" dễ bị lừa) → **CFS** (2007, vứt heuristic, dùng một mô hình toán) → **EEVDF** (6.6, 2023, vá điểm yếu latency của CFS).

## 3. Internal Architecture

### 3.1. Scheduling class — nhiều chính sách xếp tầng

```
Ưu tiên tuyệt đối giảm dần (kernel/sched/):
┌────────────────────────────────────────────────────────────┐
│ stop_sched_class   — việc khẩn của kernel (migration...)   │
│ dl_sched_class     — SCHED_DEADLINE (EDF thật sự)          │
│ rt_sched_class     — SCHED_FIFO / SCHED_RR (real-time)     │
│ fair_sched_class   — SCHED_NORMAL/BATCH (CFS/EEVDF) ← 99%  │
│ idle_sched_class   — task idle                             │
└────────────────────────────────────────────────────────────┘
Quy tắc sắt: còn task RT runnable thì task fair KHÔNG BAO GIỜ chạy.
```

- **SCHED_FIFO**: chạy đến khi tự nhường hoặc bị RT ưu tiên cao hơn đè. Không có time slice. Một FIFO bug loop = core đó chết với thế giới fair (kernel chừa 5% qua `sched_rt_runtime_us` để cứu). Chỉ dùng cho audio, điều khiển, PTP... và phải hiểu rõ Priority Inversion (case chương 16).
- **SCHED_RR**: FIFO + time slice xoay vòng trong cùng priority.
- Backend service: gần như luôn là **SCHED_NORMAL**. Đừng đưa service thường lên RT để "cho nhanh" — đó là anti-pattern nguy hiểm hàng đầu.

### 3.2. CFS — Completely Fair Scheduler

Ý tưởng trung tâm đẹp đến mức một dòng: **mô phỏng một CPU lý tưởng chia đều cho N task; luôn chạy task nào đang bị thiệt nhất.**

```
Mỗi task có vruntime = tổng thời gian CPU đã dùng × (trọng số chuẩn / trọng số của nó)

Run queue mỗi core = CÂY ĐỎ-ĐEN sắp theo vruntime
        ┌── pick_next: task TRÁI NHẤT (vruntime nhỏ nhất = "đói nhất")
        ▼
   [t3: 100ms] ── [t1: 102ms] ── [t7: 105ms] ── ...
   
Task chạy → vruntime tăng → tự trôi sang phải → task khác thành trái nhất
Task ngủ lâu → vruntime đứng yên → khi dậy được kẹp (clamp) quanh min_vruntime
               (không được "để dành" vô hạn — nhưng vẫn dậy là chạy gần như ngay
                → tự nhiên ưu ái task IO-bound, KHÔNG CẦN heuristic!)
```

- **Nice level** (−20..+19) ánh xạ thành trọng số nhân 1.25 mỗi bậc: nice thấp → vruntime tăng chậm → được chạy nhiều hơn. Nice −20 nhận cỡ ~90 lần trọng số nice +19.
- **Không có time slice cố định**: mỗi task nhận phần của một chu kỳ mục tiêu (`sched_latency` ~6-24ms, scale theo số task). Nhiều task → slice ngắn → switch nhiều — lý do máy quá tải càng thêm tải.
- Chọn cây đỏ-đen: thao tác O(log n), và min luôn ở lá trái (cache con trỏ) → pick_next gần O(1).

### 3.3. EEVDF — vì sao CFS bị thay sau 16 năm

Điểm yếu của CFS: fairness chỉ đảm bảo **về lượng** (ai cũng nhận đủ phần CPU về lâu dài) nhưng không đảm bảo **về thời điểm** (nhận phần đó *kịp lúc* hay không). Task latency-sensitive nhận đủ 5% CPU của nó nhưng nhận thành cục trễ 20ms — với audio hay trading là hỏng. CFS vá bằng đống heuristic (`wakeup_preempt`, buddy...) ngày càng rối.

**EEVDF** (Earliest Eligible Virtual Deadline First — mặc định từ kernel 6.6): mỗi task ngoài phần CPU còn có **virtual deadline** = khi nào nó *nên* nhận xong phần của mình; chọn task đủ điều kiện (eligible — không vượt phần) có deadline sớm nhất. Task xin slice ngắn (yêu cầu latency) có deadline gần → được chạy sớm hơn nhưng **không được chạy nhiều hơn** — tách bạch latency khỏi share, có nền tảng lý thuyết thay vì heuristic. Với backend engineer: hành vi mặc định tốt hơn cho tail latency khi máy đông task; các tham số `sched_latency` cũ biến mất; ít lý do tuning tay hơn.

### 3.4. Load Balancing và CPU Affinity

Mỗi core một run queue riêng (tránh một lock toàn cục — bài học scale kinh điển). Hệ quả: phải **cân bằng giữa các queue**:

- Định kỳ + khi core rảnh: kéo task từ core bận, theo tầng topology (**sched_domain**: SMT siblings → cùng L3 → cùng NUMA node → khác node) — càng xa càng ngại kéo, vì kéo là mất cache/locality.
- **Wake-up placement**: task thức dậy đặt ở đâu? Gần core cũ (cache ấm) hay gần task đánh thức nó (dữ liệu chung)? Đây là nguồn của nhiều "bí ẩn latency" — hai service ping-pong nhau có thể nhảy core liên tục.
- **CPU affinity** (`taskset`, `sched_setaffinity`, cpuset cgroup): ghim task vào core. Được: cache ấm, cách ly noisy neighbor, latency ổn định (kết hợp `isolcpus`/`nohz_full` cho hệ siêu nhạy). Mất: kernel mất quyền cân bằng — ghim sai gây core quá tải giữa máy rảnh. Chỉ ghim khi đo được lợi ích.

## 4. Cách hoạt động — dòng đời một quyết định schedule

```
Timer interrupt (tick) trên core 3
  ↓
scheduler_tick(): cập nhật vruntime của task đang chạy
  ↓ (nếu hết phần hoặc có task xứng đáng hơn)
đặt cờ TIF_NEED_RESCHED
  ↓ (tại điểm quay về user space, hoặc điểm preempt trong kernel)
__schedule():
  1. pick_next_task() — hỏi từng sched class từ cao xuống thấp
  2. context_switch() — như chương 05
  ↓
task mới chạy trên core 3

Đường thứ hai — wake-up preemption:
packet đến → softirq xử lý → wake_up(task đang chờ socket)
  → task dậy, EEVDF thấy deadline nó sớm hơn task đang chạy
  → preempt NGAY (không đợi tick) → độ trễ đánh thức ~µs
```

Điểm hay bị hiểu sai: **preemption không xảy ra "bất kỳ lúc nào"** mà tại các điểm xác định (quay về user space, preemption point trong kernel). Kernel build `PREEMPT_NONE/VOLUNTARY/FULL` khác nhau ở mật độ điểm này — server distro thường dùng voluntary: throughput tốt hơn, đổi lấy latency kém hơn desktop/RT.

## 5. Trade-off

| Quyết định | Được | Mất |
|---|---|---|
| Preemptive (timer tick) | Không ai chiếm CPU mãi | Overhead tick; task bị cắt giữa chừng lúc cache đang ấm |
| Fairness theo vruntime | Không starvation, không heuristic gian lận được | Fairness ≠ latency (lý do sinh EEVDF) |
| Run queue per-core | Scale với số core | Cần load balancing; quyết định placement có thể sai |
| Ưu ái task hay ngủ (tự nhiên qua vruntime) | Interactive/IO-bound phản hồi nhanh | Task batch bị chèn ép khi máy đông |
| RT class tuyệt đối | Đảm bảo cho hard real-time | RT bug = chiếm core; user thường bị cấm dùng (cần CAP_SYS_NICE) |

## 6. Production

- **Run queue depth**: `vmstat 1` cột `r` — số task runnable. Bền vững > số core = CPU đang là bottleneck thật sự. Đây, chứ không phải %CPU, là chỉ số "CPU đủ hay thiếu" đúng nhất.
- **Scheduling delay của từng process** — vàng ròng cho chẩn đoán "service chậm mà CPU không full": `cat /proc/<pid>/schedstat` (trường 2 = tổng ns chờ trong run queue), hoặc `perf sched record` + `perf sched latency`, hoặc eBPF `runqlat` (histogram độ trễ run queue toàn máy).
- **PSI (Pressure Stall Information)** — `/proc/pressure/cpu`: "some avg10=12.5" nghĩa là 12.5% thời gian có task phải chờ CPU. Metric hiện đại, đáng đưa lên dashboard hơn load average.
- Trong container: nhìn thêm **cpu.stat của cgroup** (`nr_throttled`, `throttled_time`) — service "chậm bí ẩn" mỗi 100ms một nhịp gần như chắc chắn là CFS bandwidth throttling (chương 13 & 17).

## 7. Anti-pattern

- Đặt service lên SCHED_FIFO để "ưu tiên" → bug loop chiếm nguyên core, ssh không vào được.
- Nice −20 cho app chính rồi quên các daemon phụ trợ (log shipper, health check) → chúng đói → hệ quả dây chuyền kỳ quái.
- Ghim affinity theo "trực giác" rồi quên khi scale số worker — 8 worker ghim vào 4 core đầu, 60 core còn lại ngồi chơi.
- Đọc %CPU 60% và kết luận "CPU dư" — trong khi run queue sâu do burst: %CPU là trung bình, latency sống ở tail.

## 8. Failure Analysis — case mẫu: p99 tăng mỗi khi cron backup chạy

- **Triệu chứng**: 02:00 hàng đêm p99 API 40ms → 800ms trong 20 phút; %CPU tổng chỉ 70%.
- **Điều tra**: `runqlat` cho thấy độ trễ run queue p99 nhảy từ 60µs lên 15ms lúc 02:00. `pidstat` → tiến trình nén backup dùng 16 thread CPU-bound.
- **Root cause**: 16 thread nén + 30 thread service > 32 core → run queue có hàng; EEVDF chia công bằng nhưng service nhạy latency phải xếp hàng sau các slice nén.
- **Fix**: chạy backup với `nice 19` + `ionice`, hoặc cgroup `cpu.weight` thấp / `cpu.max` giới hạn cứng. Sau fix: p99 45ms trong lúc backup.
- **Prevention**: mọi batch job trên máy chung phải khai báo cgroup với weight thấp; alert trên PSI cpu some > 10%.

## 9. Khi nào không nên tối ưu

Nếu run queue hiếm khi vượt số core và PSI cpu ~0, mọi tinh chỉnh scheduler (nice, affinity, sched policy, tham số debugfs) đều là placebo — scheduler chỉ quyết định *thứ tự chờ*, không làm code chạy nhanh hơn khi không ai phải chờ. Tối ưu đúng lúc đó nằm ở code (giảm CPU cần dùng) hoặc mua thêm core. Tuning scheduler có ý nghĩa khi: máy chạy trộn workload nhạy latency + batch, hoặc hệ real-time thật sự.

---

**Chương tiếp theo**: CPU đã chia xong — đến tài nguyên tranh chấp thứ hai, phức tạp hơn nhiều: [Chương 07: Memory Management](/series/linux-os-for-backend/07-memory-management/).
