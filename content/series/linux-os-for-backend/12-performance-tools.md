+++
title = "Chương 12 — Linux Performance Tools: công cụ hoạt động thế nào, không chỉ dùng thế nào"
date = "2026-02-21T19:00:00+07:00"
draft = false
tags = ["backend", "linux", "operating-system"]
series = ["Operating Systems cho Backend Engineer"]
+++

## 1. Problem Statement

Production chậm. Bạn có 30 phút và một máy 64 core đang chạy 200 process. Câu hỏi không phải "gõ lệnh gì" mà là: **mỗi công cụ nhìn hệ thống qua cửa sổ nào, trả giá bao nhiêu để nhìn, và nói dối ở đâu.** Không hiểu cơ chế thì: dùng strace làm sập production (overhead 100x), đọc %util của NVMe như HDD (sai), tin flame graph thiếu frame (vì thiếu frame pointer), hay đo latency bằng trung bình (che mất tail).

## 2. Bản đồ: bốn nguồn dữ liệu của mọi công cụ

Mọi công cụ quan sát Linux đều uống từ bốn nguồn:

```
1. /proc, /sys        — kernel XUẤT BẢN counter/state ra file ảo
                        (top, ps, vmstat, iostat, sar, ss... đọc từ đây)
2. Hardware PMU       — counter trong CPU: cycles, instructions, cache miss
                        (perf stat/record ở chế độ hardware event)
3. Static tracepoints — điểm hook ĐẶT SẴN trong source kernel (~2000 điểm:
   + kprobes/uprobes    sched:*, syscalls:*, block:*...) và hook ĐỘNG vào
                        bất kỳ hàm kernel (kprobe) / hàm user (uprobe)
                        (perf, ftrace, eBPF đều cắm vào tầng này)
4. ptrace             — quyền điều khiển/chặn process khác (strace, gdb)
                        ← nguồn DUY NHẤT có overhead thảm họa
```

Nắm bảng này là nắm được cả chương: mỗi công cụ = cách đọc + tổng hợp một hoặc vài nguồn trên.

## 3. Bộ đồ nghề cổ điển — đọc counter

### 3.1. top/htop — cơ chế và những chỗ nói dối

Cơ chế: mỗi 1-3s đọc `/proc/<pid>/stat` của mọi process, lấy delta CPU tick chia khoảng thời gian. Hệ quả: (1) %CPU là **trung bình của interval** — burst 50ms không hiện; (2) process sống ngắn **không bao giờ xuất hiện** (sinh và chết giữa hai lần đọc — dùng `perf sched`/eBPF `execsnoop` để bắt); (3) đọc cột: `%us` user, `%sy` kernel, `%wa` iowait (nghĩa thật: *CPU rảnh trong khi có IO đang chờ* — không phải "CPU bận vì IO"; %wa cao + %us thấp = máy rảnh chờ disk, không phải máy quá tải), `%si` softirq (network!), `%st` bị hypervisor cướp (VM hàng xóm ồn).
Load average: trung bình mũ 1/5/15 phút của số task **runnable + D-state** — vì có D nên load 40 với CPU idle 90% = bão IO/NFS, không phải bão CPU (case chương 15).

### 3.2. vmstat / iostat / sar — delta của counter toàn cục

- `vmstat 1`: mỗi giây đọc `/proc/stat`, `/proc/vmstat`. Cột quan trọng: `r` (run queue — so với số core), `b` (D-state), `si/so` (swap in/out — phải là 0), `cs` (context switch), `wa`.
- `iostat -x 1`: đọc `/proc/diskstats`. Nhớ: `%util` = tỷ lệ thời gian thiết bị có ≥1 IO — với NVMe xử lý 64k lệnh song song, 100% util có thể vẫn còn 90% năng lực; nhìn `await` (latency) và `aqu-sz` (độ sâu hàng đợi) mới đúng.
- `sar`: chính là bộ counter đó nhưng **ghi lịch sử** (10 phút/lần qua cron) — trả lời "đêm qua 3h có gì" khi mọi công cụ khác chỉ thấy hiện tại. Bật sẵn `sysstat` trên mọi máy — miễn phí, cứu mạng khi điều tra hồi cứu.

### 3.3. strace / ltrace — mạnh, chậm, nguy hiểm

Cơ chế strace: `ptrace(PTRACE_SYSCALL)` — **mỗi syscall của target dừng 2 lần** (vào + ra), kernel schedule strace chạy để đọc register, rồi đánh thức lại target. Mỗi syscall thành ~4 context switch → chậm 10-100 lần. Quy tắc: production chịu tải → **không strace**; thay bằng `perf trace` (tracepoint, overhead thấp) hoặc eBPF. strace dành cho: process đã hỏng sẵn, môi trường dev, hoặc câu hỏi "nó chết ở syscall nào" (`strace -f -e trace=%file` xem mở file gì là use case kinh điển vô hại với process ngắn). ltrace tương tự cho library call, còn chậm hơn.

## 4. perf — kính hiển vi thống kê

### 4.1. Cơ chế sampling

`perf record` KHÔNG đo mọi sự kiện — nó **lấy mẫu**: PMU đếm (ví dụ) cycles, tràn ngưỡng → NMI interrupt → perf ghi lại (PC + stack) → reset. 99Hz nghĩa là ~99 ảnh chụp/giây/CPU. Hệ quả: overhead thấp (1-5%), kết quả là *phân bố thống kê* — hàm chiếm 30% sample ≈ 30% CPU time; hàm 0.1% có thể không hiện. Sampling trả lời "CPU **đang ở đâu**", không trả lời "hàm này chạy bao nhiêu lần" (cái đó cần tracing).

```bash
perf top                               # "top của hàm" — nhìn nhanh 30 giây đầu
perf record -F 99 -g -p <pid> -- sleep 30
perf report                            # hoặc:
perf script | stackcollapse-perf.pl | flamegraph.pl > cpu.svg
```

**Flame graph**: trục ngang = tỷ lệ sample (KHÔNG phải thời gian tuần tự), trục dọc = stack. Cao nguyên rộng trên đỉnh = hotspot. Bẫy số một: **stack cụt/vô nghĩa** vì binary build không có frame pointer — Go mặc định OK; C/C++ cần `-fno-omit-frame-pointer` (nhiều distro đã bật lại mặc định từ 2023-2024) hoặc dùng DWARF (`--call-graph dwarf`, nặng hơn); JIT (Java/Node) cần perf-map-agent/`--perf-basic-prof`.

`perf stat` (đếm chứ không sample) cho bức tranh vi kiến trúc: IPC < 1 → nghẽn memory; cache-misses, branch-misses (chương 02). `perf sched` ghi sự kiện scheduler — độ trễ run queue per-task. `perf c2c` — bắt false sharing.

## 5. ftrace — máy ghi âm của kernel

Cơ chế: kernel được build với khả năng bật hook ở **đầu mọi hàm kernel** (nop được vá thành lệnh gọi khi bật — zero cost khi tắt); điều khiển qua `/sys/kernel/tracing` (tracefs). Không cần cài gì — có mặt trên **mọi** máy Linux, kể cả máy không cài nổi package lúc 3h sáng:

```bash
cd /sys/kernel/tracing
echo function_graph > current_tracer      # vẽ cây gọi hàm kernel + thời gian
echo 1 > events/sched/sched_switch/enable # hoặc bật từng tracepoint
cat trace_pipe
# tiện hơn: trace-cmd record -e block; kernelshark để xem GUI
```

Dùng khi: cần biết *chính xác* kernel làm gì theo thứ tự thời gian (ai gọi `vfs_fsync`, vì sao ngủ ở đâu), độ trễ wakeup, hành vi IRQ. ftrace là công cụ "sự kiện tuần tự"; perf là "thống kê"; hai góc nhìn bổ nhau.

## 6. eBPF — cách quan sát hiện đại: chạy code trong kernel một cách an toàn

Bước nhảy khái niệm: thay vì kéo *dữ liệu thô* ra user space rồi tổng hợp (đắt), **đưa chương trình tổng hợp vào kernel**:

```
Bạn viết chương trình nhỏ (C bị giới hạn / bpftrace DSL)
  → biên dịch ra bytecode eBPF
  → VERIFIER kiểm chứng: không loop vô hạn, không đọc memory bừa,
    có giới hạn lệnh → TỪ CHỐI nếu không chứng minh được an toàn
  → JIT thành mã máy, cắm vào hook: kprobe/uprobe/tracepoint/perf event
  → chạy MỖI LẦN sự kiện xảy ra, ghi kết quả vào BPF MAP (hash/histogram
    trong kernel) → user space đọc map định kỳ
→ đo hàng triệu sự kiện/giây với overhead vài %, không sửa kernel,
  không restart app, an toàn theo CONSTRUCT chứ không theo niềm tin
```

Đây là điều strace/perf không làm được: ví dụ `biolatency` cắm vào block layer, cộng dồn **histogram latency trong kernel**, mỗi giây chỉ chuyển vài trăm byte (cái histogram) ra ngoài. Bộ công cụ dùng ngay (bcc-tools / bpftrace):

| Câu hỏi production | Công cụ |
|---|---|
| Thời gian chờ run queue? | `runqlat` |
| Ai đang exec process mới liên tục? | `execsnoop` |
| Phân bố latency block IO? IO nào chậm? | `biolatency`, `biosnoop` |
| Ai mở file gì? | `opensnoop` |
| TCP retransmit/drop từng sự kiện? | `tcpretrans`, `tcpdrop` |
| Thread block ở đâu (off-CPU)? | `offcputime` |
| Syscall nào nhiều nhất? | `syscount` |
| Page cache hit ratio? | `cachestat` |

Một dòng bpftrace trả lời câu hỏi tùy biến: 

```bash
bpftrace -e 'kprobe:vfs_fsync { @[comm] = count(); }'   # ai đang gọi fsync?
bpftrace -e 'tracepoint:sched:sched_switch { @[args->prev_comm] = count(); }'
```

eBPF còn vượt ra ngoài observability: XDP (xử lý packet tại driver — DDoS filter của Cloudflare), thay iptables (Cilium trong Kubernetes), seccomp-style policy — trở thành cơ chế mở rộng kernel chính thống như đã nhắc ở chương 03.

## 7. Phương pháp — công cụ vô dụng nếu không có trình tự

**USE method** (Brendan Gregg) cho mỗi tài nguyên (CPU, memory, disk, network, và cả *lock, pool, queue của app*): kiểm tra **U**tilization – **S**aturation – **E**rrors:

```
CPU:    util=%CPU | sat=run queue/PSI cpu | err=hiếm
Memory: util=available | sat=swap in-out, direct reclaim, PSI mem | err=OOM kill
Disk:   util=await so baseline | sat=aqu-sz, PSI io | err=I/O error dmesg
Net:    util=băng thông | sat=drop, overflow, retrans | err=nstat errors
```

60 giây đầu tiên trên máy có sự cố (checklist kinh điển, thứ tự có chủ đích):
`uptime` (load trend) → `dmesg | tail` (OOM? lỗi disk? bug kernel?) → `vmstat 1` (r, si/so, us/sy/wa) → `mpstat -P ALL 1` (một core gánh hết? %si?) → `pidstat 1` (ai ăn) → `iostat -x 1` → `free -m` → `sar -n DEV 1` → `sar -n TCP,ETCP 1` (retrans!) → rồi mới đến perf/eBPF theo giả thuyết. Kỷ luật: **đi từ rẻ đến đắt, từ rộng đến hẹp, mỗi bước loại trừ một lớp giả thuyết.**

## 8. Anti-pattern

- strace vào process production chịu tải (đã nhấn mạnh, vẫn là tai nạn phổ biến nhất).
- Đo trung bình, bỏ tail — mọi công cụ histogram (biolatency, runqlat, HDR trong app) tồn tại vì p99 mới là trải nghiệm người dùng khi fan-out.
- Flame graph một lần lúc *bình thường* rồi kết luận — luôn so sánh baseline vs sự cố (differential flame graph).
- Benchmark microservice bằng `ab` từ chính máy đó (đo cả contention của tool với app).
- Sửa trước đo sau ("chắc là do GC") — mọi thay đổi phải có số trước/sau cùng điều kiện.
- Quên rằng quan sát có giá: perf 99Hz ~vài %, nhưng uprobe vào hàm gọi 10 triệu lần/giây thì chính nó thành bottleneck.

## 9. Khi nào không nên tối ưu (việc đo)

Đừng xây "observability tối đa" trước khi có sự cố đầu tiên: bật mọi tracer, ghi mọi event = trả phí thường trực cho dữ liệu không ai xem. Nền tảng đúng cho 90% team: metrics chuẩn (USE cho host + RED cho service) + sar + PSI + khả năng *bật* eBPF/perf khi cần trong vài phút. Tracing sâu là công cụ điều tra, không phải trang sức dashboard.

---

**Chương tiếp theo**: gói mọi cơ chế đã học (namespace hóa tài nguyên, giới hạn bằng cgroup) thành sản phẩm định hình một thập kỷ hạ tầng: [Chương 13: Container](/series/linux-os-for-backend/13-container/).
