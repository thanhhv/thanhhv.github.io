+++
title = "Bài 3 — Linux Internals"
date = "2026-02-01T09:00:00+07:00"
draft = false
tags = ["backend", "infrastructure", "networking"]
series = ["Infrastructure & Networking"]
+++

# Module 03 — Linux Internals

> Mọi thứ bạn deploy cuối cùng đều là process trên Linux. Container, Kubernetes, "serverless" — tất cả là abstraction trên các cơ chế trong module này. Hiểu chúng = debug được mọi tầng phía trên.

---

## 1. Process, Thread và Scheduler

### Problem Statement
Một CPU chỉ chạy được một luồng lệnh tại một thời điểm, nhưng hệ thống cần chạy hàng nghìn việc "đồng thời", cách ly lỗi giữa các chương trình, và phân phối CPU công bằng. Process/thread + scheduler là câu trả lời của kernel.

### Bản chất
- **Process** = đơn vị cách ly tài nguyên: không gian địa chỉ ảo riêng, bảng file descriptor riêng, credential riêng. Chết không kéo theo process khác.
- **Thread** = đơn vị thực thi: các thread trong 1 process **chia sẻ** address space và fd table, chỉ có stack + register riêng. Với kernel, thread và process đều là `task_struct` — khác nhau ở mức độ chia sẻ khi `clone()`.
- Trade-off: process cách ly tốt, giao tiếp đắt (IPC); thread giao tiếp rẻ (shared memory), đổi lấy toàn bộ lớp bug data race và một thread hỏng heap là cả process hỏng.

**Context switch có giá**: lưu/khôi phục register, đổi page table (với process khác nhau), và cái đắt nhất — **cache/TLB nguội đi**. Chi phí trực tiếp ~1–5µs; chi phí gián tiếp (cache miss sau đó) lớn hơn nhiều. Hệ thống có `csw` hàng trăm nghìn/giây (`vmstat 1`) đang đốt CPU vào việc đổi việc thay vì làm việc — thường do quá nhiều thread hoặc lock contention.

### CFS Scheduler và cái bẫy lớn nhất trong container
CFS (Completely Fair Scheduler) chọn task có `vruntime` nhỏ nhất (được chạy ít nhất so với trọng số) từ red-black tree. "Fair" nghĩa là chia CPU theo tỷ lệ weight (nice/`cpu.shares`).

**Bẫy production quan trọng nhất**: Kubernetes CPU **limit** = CFS bandwidth control: mỗi 100ms (period), cgroup được `quota` ms CPU. Limit 500m = 50ms mỗi 100ms. Ứng dụng 8 thread cùng chạy sẽ đốt hết 50ms trong ~6ms đầu → **bị throttle 94ms còn lại** → latency p99 tăng đột biến dù tổng CPU usage trông rất thấp (50%!).

```bash
cat /sys/fs/cgroup/cpu.stat        # nr_throttled, throttled_usec — metric bắt buộc theo dõi
```
Bài học: (1) theo dõi **throttling**, không chỉ CPU usage; (2) cân nhắc bỏ CPU limit, chỉ đặt request (CPU là tài nguyên nén được — tranh chấp chỉ làm chậm, không giết process); (3) runtime đa luồng (Go, JVM) phải được báo đúng số CPU khả dụng (`GOMAXPROCS`, `-XX:ActiveProcessorCount`) — mặc định chúng thấy tất cả core của node và tự bắn vào chân.

### Load average đọc cho đúng
Load average trên Linux đếm task **runnable + uninterruptible sleep (D state — thường là chờ disk I/O)**. Load 20 trên máy 4 core có thể là: CPU quá tải, HOẶC disk/NFS treo với CPU rảnh tênh. Phân biệt bằng: `vmstat 1` (cột `r` = chờ CPU, `b` = blocked I/O), `ps -eo state | grep D | wc -l`.

---

## 2. Memory Management và Virtual Memory

### Problem Statement
RAM hữu hạn, các process cần: không giẫm lên nhau, có ảo giác bộ nhớ liền mạch, và tổng bộ nhớ "hứa" có thể vượt RAM thật. Virtual memory là lớp indirection giải quyết cả ba.

### Cách hoạt động
Mỗi process thấy không gian địa chỉ ảo riêng; MMU + page table dịch địa chỉ ảo → vật lý theo **page 4KB**, TLB cache kết quả dịch. Các hệ quả kỹ sư backend phải biết:

- **Demand paging + overcommit**: `malloc()` chỉ ghi sổ; RAM thật chỉ cấp khi chạm vào page (page fault). Kernel mặc định **hứa nhiều hơn RAM có** (overcommit) — hóa đơn chỉ đến khi dùng. Khi hóa đơn vượt RAM: OOM Killer (phần 6).
- **VIRT vs RSS**: `VIRT` (đã hứa) thường vô nghĩa để đo — JVM có thể VIRT 20GB, RSS 2GB. Nhìn **RSS** (và với cgroup: RSS + page cache tính vào limit — nguồn gây OOM "bí ẩn" khi app ghi file nhiều).
- **Page cache**: RAM rảnh được kernel dùng hết làm cache disk — "free RAM là RAM lãng phí". `free -h` cột `available` mới là con số thật (buff/cache đòi lại được). Database đọc nhanh sau lần đầu chính là nhờ page cache; và node "hết RAM" trong khi buff/cache 30GB là node hoàn toàn khỏe.
- **Copy-on-write**: `fork()` không copy memory — 2 process chia sẻ page read-only, chỉ copy khi ghi. Redis BGSAVE dựa vào đây: fork để snapshot; nhưng nếu ghi nhiều trong lúc snapshot, memory usage có thể **tăng gần gấp đôi** — lý do quy hoạch RAM cho Redis phải chừa headroom.
- **Swap và container**: Kubernetes mặc định tắt swap — triết lý "chết nhanh và rõ (OOM kill) tốt hơn sống chậm và mờ (swap thrashing)". Trade-off chủ đích: predictability > tận dụng RAM.

```bash
free -h; vmstat 1                   # si/so > 0 liên tục = đang swap = báo động
cat /proc/<pid>/status | grep -E "VmRSS|VmSwap"
sar -B 1                            # page fault rate
```

---

## 3. File System, File Descriptor và "everything is a file"

### Bản chất
Linux thống nhất I/O qua một abstraction: **file descriptor** — số nguyên index vào bảng fd của process, trỏ tới file, socket, pipe, timer (timerfd), sự kiện (eventfd), thậm chí process (pidfd). Sức mạnh: một API (`read/write/close`) và một cơ chế chờ (`epoll`) cho mọi loại I/O.

### Những điều gây sự cố thật
- **fd là tài nguyên hữu hạn**: `ulimit -n` mặc định có thể chỉ 1024. Server bận rộn (mỗi connection = 1 fd, mỗi file mở = 1 fd, mỗi connection tới DB = 1 fd) chạm trần → `EMFILE: too many open files` → từ chối mọi connection mới. Luôn nâng limit cho service production (systemd: `LimitNOFILE=`) và **monitor số fd đang mở** (`ls /proc/<pid>/fd | wc -l`). fd leak (quên close response body — bug kinh điển trong Go) là lỗi cháy chậm: chạy tốt 3 ngày rồi sập.
- **Xóa file ≠ giải phóng dung lượng**: `rm` chỉ xóa tên (dentry); inode và data còn sống chừng nào còn process mở fd. Hiện tượng kinh điển: `df` báo đầy 100% nhưng `du` cộng lại chỉ 60% — có process giữ file log đã xóa. Tìm: `lsof +L1`. Giải pháp đúng cho log: `truncate -s 0` hoặc logrotate với `copytruncate`, không `rm`.
- **Buffered I/O và độ bền dữ liệu**: `write()` thành công = dữ liệu vào **page cache**, chưa xuống disk. Mất điện = mất dữ liệu, trừ khi `fsync()`. Mọi database đúng đắn đều fsync WAL trước khi báo commit — và đây là lý do fsync latency của disk quyết định TPS của database. Trên cloud: gp3/io2 (EBS) fsync ~1ms; local NVMe ~50µs — chênh 20× TPS ghi.

---

## 4. Socket, System Call và epoll — nền của mọi server hiệu năng cao

### Problem Statement — C10K
Mô hình cổ điển "1 connection = 1 thread" chết ở ~vài nghìn connection: mỗi thread tốn stack (mặc định 8MB virtual), context switch tăng theo số thread, và đa số thread chỉ đang... chờ dữ liệu. Câu hỏi C10K (1999): làm sao 1 máy phục vụ 10.000 connection? Câu trả lời của Linux: **epoll** — và nó định hình mọi runtime hiện đại (nginx, Node.js, Go netpoller, Netty, Envoy).

### Tiến hóa của cơ chế chờ I/O
1. `select`/`poll`: mỗi lần gọi, đưa **toàn bộ danh sách fd** cho kernel quét → O(n) mỗi lần, n lớn thì chết.
2. **epoll**: đăng ký fd **một lần** (`epoll_ctl`), kernel duy trì interest list; khi fd sẵn sàng, kernel bỏ vào ready list; `epoll_wait` chỉ trả về **những fd sẵn sàng** → O(số fd active), không phụ thuộc tổng số connection. 100k connection idle, 100 active → chỉ trả 100.
3. `io_uring` (hiện đại): 2 ring buffer chia sẻ user/kernel, đẩy syscall theo lô, đúng nghĩa async cả với file I/O (epoll không giúp gì cho regular file!). Đáng theo dõi nhưng epoll vẫn là chủ lực.

Kiến trúc chuẩn (nginx, Envoy): **N worker = N core**, mỗi worker một event loop epoll, non-blocking socket. Hệ quả cho người viết ứng dụng trên event loop (Node.js): **block event loop 10ms = block TẤT CẢ connection 10ms**. CPU-bound work phải ra worker thread.

**Edge-triggered vs level-triggered**: LT báo lại chừng nào còn dữ liệu (an toàn, dễ); ET báo đúng 1 lần khi trạng thái đổi (ít wakeup hơn, nhưng bắt buộc đọc đến `EAGAIN` — quên là treo connection vĩnh viễn). Bug ET là loại bug "connection thỉnh thoảng đơ" khó tìm bậc nhất.

### System call có giá
Mỗi syscall là một lần chuyển user→kernel mode (~100–300ns, tệ hơn sau các mitigation Spectre/Meltdown). Server 1M ops/s thì syscall overhead là số đo được. Vì thế mới có: buffered I/O (gom nhiều write), `sendfile`/zero-copy (file → socket không qua userspace — nginx serve static file), `writev` (nhiều buffer 1 syscall), io_uring (nhiều op 1 lần vào kernel).

```bash
strace -c -p <pid>       # đếm syscall — CẢNH BÁO: strace làm chậm process nặng, chỉ dùng ngoài production hoặc rất ngắn
perf trace -p <pid>      # nhẹ hơn strace nhiều
```

---

## 5. Signals

Signal = cơ chế thông báo bất đồng bộ của kernel tới process. Những gì kỹ sư backend cần thuộc lòng:

- `SIGTERM` (15): "hãy dọn dẹp rồi thoát" — catch được. `SIGKILL` (9): chết ngay, **không catch được**, không dọn dẹp. `docker stop` / Kubernetes: gửi SIGTERM, chờ grace period (mặc định 30s), rồi SIGKILL.
- **Graceful shutdown là hợp đồng bắt buộc**: nhận SIGTERM → ngừng nhận request mới (fail readiness) → chờ request đang chạy xong → đóng connection → thoát. Ứng dụng bỏ qua SIGTERM sẽ bị SIGKILL giữa chừng transaction → lỗi 502 mỗi lần deploy. "Deploy nào cũng có một nhúm 5xx" gần như luôn là bug graceful shutdown hoặc thiếu phối hợp readiness/LB.
- **Bẫy PID 1 trong container**: process PID 1 không có default signal handler — SIGTERM gửi tới bị **bỏ qua** nếu app không tự cài handler. Triệu chứng: container luôn mất đúng 30s (grace period) để dừng. Nguyên nhân phổ biến: `CMD` dạng shell (`sh -c "node app.js"`) — shell là PID 1 và không forward signal. Giải pháp: exec form `CMD ["node", "app.js"]`, hoặc `tini`/`dumb-init` làm PID 1 (kiêm luôn reaping zombie process).
- `SIGSEGV` → core dump (bug memory); `SIGPIPE` → ghi vào socket đã đóng, mặc định giết process — thư viện network phải ignore và xử lý `EPIPE`.

---

## 6. cgroups, namespaces và OOM Killer — nền móng của container

### Problem Statement
Nhiều workload chung một kernel cần: **thấy** thế giới riêng (cách ly nhìn) và **dùng** tài nguyên có hạn mức (cách ly dùng). Hai bài toán độc lập → hai cơ chế độc lập: namespace và cgroup. Container = namespace + cgroup + filesystem layered — **không có "công nghệ container" riêng biệt nào trong kernel cả**.

### Namespaces — cách ly cái nhìn thấy
| Namespace | Cách ly | Ý nghĩa thực tế |
|---|---|---|
| pid | Cây process | Trong container, app là PID 1 (xem bẫy signal ở trên) |
| net | Interface, routing, iptables | Mỗi container có network stack riêng; nối bằng veth |
| mnt | Mount points | Root filesystem riêng |
| uts | hostname | |
| ipc | Shared memory, semaphore | |
| user | Mapping UID | root trong container ≠ root trên host (nếu bật — nên bật!) |

### cgroups — hạn mức cái dùng được
cgroup v2 (chuẩn hiện tại): cây thống nhất, mỗi node giới hạn cpu/memory/io/pids cho nhóm process. Kubernetes requests/limits được dịch thẳng thành cgroup: CPU request → `cpu.weight`; CPU limit → `cpu.max` (quota/period — nguồn throttling ở phần 1); memory limit → `memory.max`.

### OOM Killer — hiểu để không bị bất ngờ
Khi cgroup chạm `memory.max` (hoặc toàn hệ thống hết RAM), kernel không có lựa chọn tốt: **giết một process** (chọn theo oom_score ~ lượng RAM đang dùng + điều chỉnh `oom_score_adj`).

Những điều hay bị hiểu sai:
1. Container OOM kill = **SIGKILL, không có warning, không có graceful**. Exit code 137. Không thể "xử lý" OOM từ bên trong.
2. **Page cache của container tính vào memory limit**. App ghi log/file nhiều có thể bị OOM dù heap nhỏ — kernel sẽ reclaim cache trước, nhưng dirty page chưa flush thì không reclaim được ngay. Triệu chứng: OOM khi I/O tăng đột biến.
3. Runtime có heap riêng (JVM) phải khớp với limit: heap 4GB + metaspace + thread stack + direct buffer trong limit 4GB = OOM chắc chắn. Quy tắc thô: heap ≤ 60–75% limit; hoặc dùng `-XX:MaxRAMPercentage`.
4. Node-level OOM (trước khi cgroup kịp chặn) nguy hiểm hơn: kernel có thể giết process "vô tội" — lý do Kubernetes có eviction và ưu tiên theo QoS class (BestEffort chết trước, Guaranteed chết cuối).

```bash
dmesg -T | grep -i "killed process"           # bằng chứng OOM
cat /sys/fs/cgroup/<path>/memory.events       # oom_kill counter
kubectl describe pod <pod> | grep -A3 "Last State"   # OOMKilled, exit 137
```

---

## Bảng lệnh điều tra nhanh (in ra dán cạnh màn hình)

| Triệu chứng | Lệnh đầu tiên | Nhìn gì |
|---|---|---|
| CPU cao | `top` → `pidstat 1 -p <pid> -t` | user vs sys; thread nào |
| Latency cao, CPU thấp | `cat /sys/fs/cgroup/cpu.stat`; `vmstat 1` | throttling; cột b (blocked) |
| Nghi hết RAM | `free -h`; `dmesg -T \| grep -i oom` | available; OOM kill |
| Disk đầy bí ẩn | `df -h` vs `du -sh /*`; `lsof +L1` | file đã xóa còn mở |
| Connection lỗi lạ | `ss -s`; `ss -tnp`; `dmesg \| grep conntrack` | TIME_WAIT, queue, conntrack full |
| Chậm không rõ | `vmstat 1`; `iostat -x 1`; `sar -n DEV 1` | CPU/disk/net cái nào bão hòa |
| Process treo | `cat /proc/<pid>/stack`; `ps -o state` | D state = kẹt I/O ở đâu |
| Đào sâu CPU | `perf top` / `perf record -g` + flamegraph | hàm nào đốt CPU |
