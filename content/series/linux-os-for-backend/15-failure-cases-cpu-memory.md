+++
title = "Chương 15 — Production Failure Cases (1): CPU & Memory"
date = "2026-02-21T22:00:00+07:00"
draft = false
tags = ["backend", "linux", "operating-system"]
series = ["Operating Systems cho Backend Engineer"]
+++

> Mỗi case theo khung: **Triệu chứng → Root cause phổ biến → Bên trong kernel → Metric & Dashboard → Điều tra từng bước → Khắc phục → Phòng tránh.** Các lệnh mặc định chạy được trên mọi máy Linux; công cụ eBPF (bcc-tools) khi có càng tốt. Kiến thức nền: chương tương ứng ghi trong ngoặc.

---

## Case 1 — CPU 100%

**Triệu chứng**: %CPU chạm trần; latency tăng; có thể kèm autoscale bùng nổ chi phí.

**Bên trong kernel**: CPU 100% chỉ có nghĩa "luôn có task runnable". Bốn kịch bản khác hẳn nhau: (a) user code tính toán (`%us` cao) — loop bug, regex độc, GC storm, crypto mining(!); (b) kernel làm việc hộ (`%sy` cao) — syscall storm, lock contention futex, network stack; (c) softirq (`%si`) — bão packet; (d) steal (`%st`) — hàng xóm trên hypervisor. Ngoài ra CPU "bận" có thể đang **stall chờ RAM** (IPC thấp — chương 02).

**Điều tra từng bước**:
```
1. top → đọc cơ cấu %us/%sy/%si/%wa/%st  ← quyết định hướng đi
2. pidstat 1 → process nào; top -H -p <pid> → THREAD nào
3. %us cao → perf top -p <pid> / flame graph / profiler ngôn ngữ (pprof...)
4. %sy cao → perf top (hàm kernel nào); syscount-bpfcc -p <pid>
5. %si cao → mpstat -P ALL 1 (dồn 1 core?); case network chương 11
6. Nghi stall: perf stat -p <pid> → IPC < 1? cache-misses cao?
```

**Khắc phục & phòng tránh**: theo nhánh (fix code, batching syscall, shard lock, RSS/RPS, đổi node). Phòng: CPU profiling liên tục (continuous profiling), alert theo *cơ cấu* CPU chứ không chỉ tổng %, giới hạn regex/input từ user.

---

## Case 2 — Load Average tăng cao

**Triệu chứng**: load 1-5-15 phút tăng (ví dụ 85 trên máy 32 core) nhưng có khi %CPU lại thấp — gây hoảng loạn sai hướng.

**Bên trong kernel**: load = trung bình mũ của số task **TASK_RUNNING (đang/đợi chạy) + TASK_UNINTERRUPTIBLE (D — chờ disk/NFS/lock kernel)** (chương 04, 06). Vế thứ hai là đặc sản Linux: load cao ≠ CPU thiếu.

**Điều tra**:
```
1. vmstat 1: cột r (runnable) vs cột b (blocked/D)
   r >> số core → thật sự thiếu CPU → sang Case 1/3
   b lớn        → bão D-state → đi bước 2
2. ps -eo state,pid,wchan:30,comm | awk '$1=="D"' → kẹt ở hàm kernel nào
   (wchan: io_schedule = disk; rpc_wait = NFS; ...)
3. iostat -x 1 (await? %util?), PSI: cat /proc/pressure/{cpu,io,memory}
4. Nếu NFS: nfsstat, mount option (hard mount treo vô hạn khi server mất!)
```

**Khắc phục**: nguồn IO chậm (Case 11), NFS chết (remount/failover), hoặc thiếu CPU thật (scale). **Phòng tránh**: dashboard tách `r` và `b` (hoặc dùng PSI thay load); hiểu rằng alert "load > N" thô sơ gây cả false positive lẫn false negative — PSI cpu/io/memory là bản nâng cấp đúng.

---

## Case 3 — Context Switch quá nhiều

**Triệu chứng**: `vmstat` cột `cs` gấp 5-10 lần baseline; CPU tăng mà công việc hoàn thành không tăng; p99 xấu.

**Bên trong kernel**: mỗi switch ~1-5µs trực tiếp + đuổi cache/TLB (chương 05). 500k cs/s × 3µs ≈ 1.5 core bốc hơi *chưa kể* cache nguội. Nguyên nhân điển hình: quá nhiều thread runnable (thread explosion — Case 15), lock contention (ngủ-dậy liên hồi quanh futex), IO nhỏ li ti, hoặc timer/polling dày đặc.

**Điều tra**:
```
1. vmstat 1 → cs baseline? (ghi baseline từ thời bình!)
2. pidstat -w 1 → process nào; cstop/voluntary vs involuntary:
   voluntary cao → block-wake liên tục: lock? IO nhỏ? channel ping-pong?
   involuntary cao → tranh CPU: quá nhiều thread / cgroup throttle
3. perf sched record -- sleep 10; perf sched latency → ai switch nhiều, chờ bao lâu
4. Lock? → mutex profile của app / futex count qua bpftrace (chương 08)
```

**Khắc phục**: giảm thread về ~số core, gộp IO (buffer), sửa lock nóng, nới cgroup quota. **Phòng tránh**: load test đo cs khi thay đổi concurrency; quy tắc "thread pool có giới hạn, giới hạn có lý do".

---

## Case 4 — OOM Killer

**Triệu chứng**: process biến mất không log; `dmesg`: `Out of memory: Killed process ... anon-rss:...`; K8s: exit 137/OOMKilled.

**Bên trong kernel** (chương 07): cấp phát thất bại sau mọi nỗ lực reclaim → `out_of_memory()` chấm điểm `oom_badness` (≈ RSS + swap, hiệu chỉnh `oom_score_adj`) → SIGKILL nạn nhân điểm cao nhất. Trong cgroup: vượt `memory.max` → OOM *cục bộ*. Nhớ: overcommit nghĩa là malloc hầu như không bao giờ fail — nợ chỉ đòi khi *chạm* page.

**Điều tra**:
```
1. dmesg -T | grep -i -A 20 'killed process' → nạn nhân, RSS, bảng điểm mọi task
   → THỦ PHẠM thường không phải NẠN NHÂN — xem task nào RSS lớn bất thường
2. Toàn máy hay cgroup? (log có "memory cgroup out of memory")
3. sar -r / dashboard: available giảm từ khi nào? đột ngột (job mới) hay
   từ từ (leak — Case 5)?
4. K8s: kubectl describe pod → limit bao nhiêu; app có ghi file lớn
   (page cache tính vào!) không?
```

**Khắc phục**: khôi phục service; tách/giới hạn thủ phạm bằng cgroup; chỉnh `oom_score_adj` bảo vệ process chính. **Phòng tránh**: quy hoạch memory bằng cgroup cho *mọi* workload trên máy chung; alert available < 15% + PSI memory; với K8s đặt limit dựa trên đo đạc + headroom, không đoán.

---

## Case 5 — Memory Leak

**Triệu chứng**: RSS tăng đơn điệu qua ngày/tuần; kết cục là Case 4. Dạng khó: tăng chậm, chỉ thấy khi nhìn đồ thị 30 ngày.

**Bên trong kernel**: kernel không biết "leak" — nó chỉ thấy process chạm ngày càng nhiều page. Phân biệt 3 dạng: (a) leak thật trong app (mất tham chiếu — ngôn ngữ GC vẫn leak được: map/cache/goroutine giữ tham chiếu, listener quên gỡ); (b) **allocator giữ lại** (free rồi nhưng chưa trả kernel — RSS cao nhưng ổn định ở cao nguyên, không phải leak); (c) fragmentation (Case 8 phụ lục). Còn một dạng nữa: **kernel memory leak** qua slab (FD/socket không đóng → `slabtop` thấy `sock_inode_cache`, `dentry` phình).

**Điều tra**:
```
1. Đồ thị RSS dài hạn: tuyến tính theo thời gian? theo traffic? cao nguyên?
2. Go: pprof heap diff 2 thời điểm; Java: heap dump + MAT; C/C++: jemalloc
   profiling / valgrind (staging), bpftrace uprobe malloc (production thận trọng)
3. Goroutine leak: /debug/pprof/goroutine — số goroutine tăng mãi = leak
   phổ biến nhất trong Go (goroutine kẹt chờ channel không ai đóng)
4. Không phải heap app? → cat /proc/<pid>/smaps_rollup (vùng nào phình);
   slabtop (kernel object); lsof đếm FD
```

**Khắc phục**: fix code; tạm thời restart định kỳ có kiểm soát (rolling) — "chữa triệu chứng" nhưng mua thời gian. **Phòng tránh**: đồ thị RSS + goroutine/FD count trên dashboard mặc định; leak test trong CI (chạy tải 1h, RSS phải phẳng); code review pattern giữ-tham-chiếu.

---

## Case 6 — Swap Thrashing

**Triệu chứng**: hệ thống chậm *toàn diện* kỳ lạ; disk IO cao trên swap partition; `vmstat` cột `si`/`so` khác 0 liên tục; load tăng do D-state.

**Bên trong kernel** (chương 07): working set các process > RAM → reclaim đẩy page anonymous ra swap; process chạm lại → major fault đọc từ disk → để có chỗ lại đẩy page khác ra — **vòng xoáy**: CPU rảnh nhưng ai cũng chờ swap-in. Độ trễ memory access nhảy từ 100ns lên 100µs-10ms (× 1.000-100.000 lần).

**Điều tra**:
```
1. vmstat 1: si/so — vài trang lẻ tẻ = vô hại; hàng nghìn KB/s liên tục = thrashing
2. Ai bị swap: for p in /proc/[0-9]*/status; do grep -H VmSwap $p; done | sort -k2 -n
3. PSI: /proc/pressure/memory — full > 0 kéo dài = cả hệ đứng vì memory
4. Vì sao thiếu RAM: process mới? leak (Case 5)? cấu hình cache DB quá tay?
```

**Khắc phục**: giết/di dời workload ít quan trọng (đừng `swapoff` giữa cơn — nó ép đọc *toàn bộ* swap về RAM đang thiếu → tê liệt thêm); thêm RAM/scale out; sửa cấu hình ăn quá RAM. **Phòng tránh**: quy hoạch working set; swappiness thấp cho máy DB; alert `so > 0` bền vững + PSI memory some > 10%; cgroup memory.low bảo vệ service chính.

---

## Case 7 — Page Fault Storm

**Triệu chứng**: CPU %sy cao, service khựng từng đợt dù RAM "đủ"; `sar -B` minor/major fault tăng vọt.

**Bên trong kernel** (chương 07): ba nguồn bão — (a) **major fault storm**: mmap file lớn đọc ngẫu nhiên lạnh cache / swap-in (Case 6); (b) **minor fault storm**: app cấp phát và giải phóng vùng lớn liên tục (madvise DONTNEED rồi chạm lại — vòng lặp map/unmap; hay gặp ở runtime trả memory quá hăng: GOGC/GC tuning), mỗi lần chạm lại là fault + zero page + có thể TLB shootdown đa core; (c) **COW storm sau fork** (Redis BGSAVE — chương 04/07).

**Điều tra**:
```
1. sar -B 1: fault/s vs majflt/s → major hay minor storm?
2. Per-process: ps -o min_flt,maj_flt -p <pid> (delta theo thời gian)
3. Major → file nào: bpftrace tracepoint:exceptions:page_fault_user +
   filemap fault; hoặc perf record -e major-faults -g
4. Minor + đa thread → nghi TLB shootdown: grep TLB /proc/interrupts (tăng nhanh?)
5. Mới fork/BGSAVE xong? → COW storm, xem timeline với redis INFO persistence
```

**Khắc phục**: (a) preload/pretouch (mmap + MAP_POPULATE, `vmtouch` warm cache), chuyển đọc ngẫu nhiên lạnh sang read() có kiểm soát; (b) tune runtime bớt trả-rồi-đòi memory, dùng pool; (c) lịch BGSAVE giờ thấp điểm / replica riêng cho persistence. **Phòng tránh**: warm-up sau deploy trước khi nhận traffic; theo dõi fault rate như metric hạng nhất của service mmap nhiều (database!).

---

## Case 8 — NUMA Imbalance

**Triệu chứng**: máy 2+ socket: throughput thấp hơn kỳ vọng 30-50%; latency dao động không giải thích được; một nửa số core "chậm hơn" nửa kia với cùng workload.

**Bên trong kernel** (chương 02): first-touch đặt page ở node của thread khởi tạo; scheduler sau đó di cư thread sang node khác → truy cập chéo QPI/UPI (~1.5-2× chậm, bandwidth hẹp); hoặc một node cạn memory cục bộ → reclaim/swap cục bộ *trong khi node kia còn RAM* (zone reclaim). Automatic NUMA balancing của kernel di cư page/thread nhưng chính việc di cư cũng tốn (fault + copy).

**Điều tra**:
```
1. numactl --hardware → topology; numastat → numa_miss/numa_foreign cao?
2. numastat -p <pid> → memory của process nằm node nào; taskset -pc <pid> → chạy node nào
3. perf stat -e node-load-misses,node-loads → tỷ lệ remote access
4. /proc/zoneinfo → node nào cạn; sự cố "swap khi RAM còn" → vm.zone_reclaim_mode?
```

**Khắc phục**: bind cả CPU + memory (`numactl --cpunodebind=0 --membind=0` — hoặc chia service thành N instance, mỗi instance một node — pattern chuẩn cho DB/proxy lớn); interleave cho workload truy cập đều (`--interleave=all` — khuyến nghị cũ cho một số DB); đảm bảo `vm.zone_reclaim_mode=0` (mặc định hiện nay). **Phòng tránh**: benchmark trên đúng topology production; cân nhắc tắt automatic NUMA balancing cho DB đã bind tay; K8s: topology manager + static CPU policy cho pod nhạy.

---

## Case 9 — Cache Miss tăng cao

**Triệu chứng**: cùng lượng request, CPU cao hơn hẳn sau một thay đổi "vô hại"; IPC tụt (ví dụ 1.8 → 0.6); không có gì bất thường ở mọi metric hệ thống khác — dạng sự cố "tàng hình" nhất.

**Bên trong kernel/phần cứng** (chương 02): các thủ phạm — cấu trúc dữ liệu đổi layout (thêm field làm struct vượt cache line, array-of-struct → struct-of-array ngược), false sharing sau khi thêm counter atomic, working set vượt L3 (thêm feature cache nội bộ), context switch tăng đuổi cache (Case 3 kéo theo), hoặc *hàng xóm cùng máy* chiếm L3 (noisy neighbor thật sự — L3 chia sẻ toàn socket!).

**Điều tra**:
```
1. perf stat -p <pid>: IPC, cache-misses, LLC-load-misses — so BASELINE
   (không có baseline thì con số vô nghĩa → đo định kỳ từ thời bình)
2. perf record -e LLC-load-misses -g → miss từ code nào
3. perf c2c record/report → false sharing (cột HITM cao, hai offset
   cùng cache line từ hai CPU)
4. Nghi hàng xóm: đo cache miss khi neighbor bật/tắt; resctrl (Intel CAT)
   để chia L3 nếu phần cứng hỗ trợ
```

**Khắc phục**: sửa layout (padding, tách read-only khỏi read-write, gom field nóng), giảm working set hot path, cách ly CPU/L3 cho service nhạy. **Phòng tránh**: perf stat trong benchmark CI cho hot path quan trọng; review "thêm một atomic counter toàn cục" như review một thay đổi kiến trúc.

---

*Tiếp: [Chương 16 — Failure Cases: IO, Network & Concurrency](/series/linux-os-for-backend/16-failure-cases-io-network-concurrency/).*
