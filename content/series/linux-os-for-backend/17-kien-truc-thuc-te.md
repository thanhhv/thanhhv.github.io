+++
title = "Chương 17 — Kiến trúc thực tế: Linux dưới các hệ thống bạn chạy hằng ngày"
date = "2026-02-22T00:00:00+07:00"
draft = false
tags = ["backend", "linux", "operating-system"]
series = ["Operating Systems cho Backend Engineer"]
+++

> Chương tổng kết: mỗi hệ thống lớn là một **tập lựa chọn** trên các trade-off đã học. Đọc mỗi mục và tự kiểm tra: bạn có chỉ ra được chương nào của tài liệu đứng sau từng quyết định không?

---

## 17.1. PostgreSQL — đặt cược vào Page Cache của kernel

**Kiến trúc trên OS**: process-per-connection (fork — chương 04; lý do connection đắt và PgBouncer gần như bắt buộc ở connection count cao); shared_buffers là shared memory giữa các process; **double buffering có chủ đích**: dữ liệu nằm cả ở shared_buffers *và* Page Cache (Postgres đọc ghi qua buffered IO — tin kernel làm cache tầng hai, khác Oracle/MySQL-InnoDB dùng O_DIRECT tự quản 100%).

**Hệ quả & tuning**:

- shared_buffers ~25% RAM (không phải 80% như DB tự quản cache!) — phần còn lại *chính là* cache của Postgres, dưới tên Page Cache.
- WAL fsync mỗi commit (chương 09/12): trần commit-rate = năng lực fsync của thiết bị → WAL nên nằm disk riêng; `synchronous_commit=off` đổi độ bền 200-600ms lấy throughput khi nghiệp vụ cho phép.
- Checkpoint = bão ghi định kỳ → dàn phẳng bằng `checkpoint_completion_target`, và sysctl `vm.dirty_background_bytes` thấp để writeback chảy đều (case chương 09).
- Huge pages (`huge_pages=on` + `vm.nr_hugepages`) cho shared_buffers lớn: giảm TLB miss + page table (chương 07 — page table của trăm process cùng map 64GB shared buffer là con số khổng lồ nếu dùng page 4KB).
- THP: khuyến nghị tắt/madvise (khựng compaction). NUMA: máy 2 socket cân nhắc `numactl --interleave` cho shared_buffers hoặc instance-per-node (case NUMA chương 15).
- OOM: `vm.overcommit_memory=2` thường được khuyên cho máy DB riêng + `oom_score_adj` bảo vệ postmaster (chương 07).
- Sự cố kinh điển: fsyncgate (chương 09), OOM giết backend (case 4), connection storm (fork đắt), IO ứ do batch (case 11).

## 17.2. Redis — single thread, và mọi hệ quả của nó

**Kiến trúc trên OS**: một event loop epoll (chương 10) xử lý mọi command → không lock, latency ổn — nhưng **một command chậm chặn tất cả** (KEYS, SMEMBERS to, script Lua dài), và chỉ dùng 1 core (I/O threads từ Redis 6 chỉ đỡ phần đọc/ghi socket).

**Ba điểm chạm kernel định mệnh**:

- **fork + COW cho BGSAVE/AOF-rewrite** (chương 04/07): ghi dồn dập trong lúc snapshot → memory phình tới ~2×, latency gợn. Vì thế: THP bắt buộc tắt (COW theo 2MB!), `vm.overcommit_memory=1` (để fork không bị từ chối khi RSS lớn), maxmemory chừa headroom cho COW, persistence nặng đẩy sang replica.
- **Latency ns nhạy với mọi thứ**: một lần swap-in hay direct reclaim là thảm họa cho dịch vụ hứa hẹn sub-ms → swappiness thấp/không swap, không bao giờ để máy Redis căng memory.
- Expire/eviction chạy trong cùng thread → maxmemory-policy và bão expire cũng là "command chậm".

## 17.3. Kafka — thiết kế xoay quanh Page Cache và sendfile

Kafka là ví dụ đẹp nhất của "thuận theo kernel thay vì chống lại":

- Log segment **append-only tuần tự** — đúng sở trường của disk + Page Cache + readahead (chương 09).
- **Không tự quản cache**: đọc/ghi qua Page Cache; "Kafka dùng ít heap nhưng cần nhiều RAM" — RAM đó là Page Cache cho tail của log (consumer đọc gần cuối = 100% cache hit, không chạm disk).
- **sendfile** (chương 11): consumer fetch = page cache → socket, zero copy, CPU gần như không chạm dữ liệu. (Lưu ý: bật TLS là mất sendfile — phải mã hóa trong user space; trade-off băng thông/bảo mật cụ thể, đo được.)
- Độ bền: mặc định **không fsync mỗi message** — tin vào replication (an toàn bằng nhiều máy — so sánh triết lý với Postgres ở 17.1!); `flush.messages=1` sẽ giết throughput đúng bằng Case 12.
- Tuning hay gặp: FD limit lớn (mỗi segment+socket một FD — case 10), `vm.max_map_count` (mmap index), dirty ratio để writeback đều, XFS phổ biến; JVM heap nhỏ vừa phải — RAM để dành cho kernel.

## 17.4. Nginx — worker-per-core, epoll thuần chủng

Master fork N worker (= số core), mỗi worker một epoll loop non-blocking (chương 10), `SO_REUSEPORT` chia connection tại kernel (chương 11), `sendfile on` cho static, không thread cho request path (thread pool chỉ cho file IO chậm). Vì thế một worker làm nghẽn = 1/N năng lực — module Lua/njs chạy CPU dài hay gọi blocking là tự bắn chân (đúng anti-pattern chương 10). Tuning: `worker_connections` × N ≤ FD limit; `worker_cpu_affinity` (cache ấm — chương 02); somaxconn + backlog khi làm LB tuyến đầu (case 14); nếu TLS nặng — đó là CPU thật (`%us`), scale bằng core hoặc offload.

## 17.5. Go Runtime — hệ điều hành thu nhỏ trong process

Như chương 05/10 đã mổ xẻ: GMP scheduler (user-space, work-stealing), netpoller trên epoll, blocking syscall tách M. Các điểm chạm OS phải nhớ khi vận hành:

- **GOMAXPROCS vs cgroup quota** — case throttling chương 13 (Go 1.25+ tự xử; trước đó: automaxprocs).
- GC: STW ngắn nhưng assist + mark ăn CPU — trong container quota chặt, GC burst chính là thứ kích hoạt throttling; `GOGC`/`GOMEMLIMIT` (đặt GOMEMLIMIT ~90% memory.max là phòng OOMKilled hiệu quả).
- Metric cặp đôi: `go_goroutines` (logic) và `go_threads` (M — kernel thread thật); threads tăng bền = blocking syscall/cgo dồn (case 17 kiểu Go).
- Profiling xuyên hai tầng: perf thấy M và hàm runtime; pprof thấy goroutine — sự cố OS-level (throttle, swap) phải nhìn từ *cả hai phía* mới ghép được bức tranh.

## 17.6. Node.js — một event loop, và kỷ luật đi kèm

libuv: epoll cho network; **thread pool (mặc định 4!) cho file IO + DNS + crypto** — `UV_THREADPOOL_SIZE` là tuning bị bỏ quên nhiều nhất (service gọi DNS/fs nhiều bị nghẽn ở 4 thread này trong khi CPU rảnh). Một process = một core cho JS → `cluster`/nhiều instance + LB. Kỷ luật: không sync API trên request path (`fs.readFileSync`, `JSON.parse` payload chục MB cũng là "sync"); đo event-loop lag như metric hạng nhất; CPU nặng → worker_threads. Về bản chất OS: Node chọn "đơn giản hóa concurrency bằng cách chỉ có một" — mọi sự cố đặc trưng của nó đều là hệ quả trực tiếp của lựa chọn này.

## 17.7. Docker & Kubernetes — cấp phát trade-off cho người khác

Docker/K8s không thêm cơ chế mới — chúng **phân phối** cơ chế chương 13 ở quy mô cụm, và các sự cố đặc trưng đều là cơ chế kernel mặc quần áo mới:

- CPU limits → CFS bandwidth → **throttling** (case chương 13) | memory limits → cgroup OOM (exit 137) | pids limit → chống fork bomb (case 18).
- QoS class (Guaranteed/Burstable/BestEffort) → thực chất là `oom_score_adj` + cgroup weight — nghĩa là **thứ tự hy sinh khi node căng memory**.
- Node pressure eviction (kubelet) chạy *trước* OOM killer — hai tầng giết chóc với log ở hai chỗ khác nhau (describe pod vs dmesg).
- Networking: kube-proxy iptables/IPVS hay Cilium eBPF (chương 12) — độ trễ và khả năng debug khác nhau; conntrack đầy là case kinh điển của node làm NAT nhiều (`nf_conntrack_max`).
- Bài học tổng: **đọc sự cố K8s bằng ngôn ngữ kernel** — "pod bị evict", "OOMKilled", "throttled", "CrashLoopBackOff sau SIGTERM" đều dịch được 1-1 sang các chương của tài liệu này.

## 17.8. Bảng sysctl đáng biết (đo trước, chỉnh sau — từng dòng một)

| sysctl | Mặc định | Khi nào chỉnh | Trade-off |
|---|---|---|---|
| `net.core.somaxconn` | 4096 (cũ: 128) | server nhận connection rate cao | queue dài = latency ẩn khi app chậm accept |
| `net.ipv4.tcp_max_syn_backlog` | ~1024+ | kèm somaxconn, chống burst/SYN | memory cho half-open |
| `net.ipv4.ip_local_port_range` | 32768-60999 | client/proxy tạo nhiều outbound | — |
| `net.ipv4.tcp_tw_reuse` | 0/2 | client cạn port vì TIME_WAIT | an toàn với timestamp; KHÔNG dùng tcp_tw_recycle (đã xóa) |
| `net.ipv4.tcp_rmem/wmem` (max) | ~6MB | đường dài băng rộng (BDP lớn) | memory per-socket × số connection |
| `net.core.netdev_max_backlog` | 1000 | bão packet, %si cao | trễ trong backlog |
| `vm.swappiness` | 60 | máy DB/latency: 1-10 | mất đệm đàn hồi (chương 07) |
| `vm.dirty_background_bytes` | 0 (dùng ratio 10%) | RAM lớn + ghi nhiều: 256MB-1GB | ghi sớm hơn = IO đều hơn nhưng nhiều lần hơn |
| `vm.overcommit_memory` | 0 | Redis: 1; DB riêng: cân nhắc 2 | 1: OOM muộn hơn; 2: malloc fail sớm |
| `vm.max_map_count` | 65530 | Kafka/ES (mmap nhiều) | — |
| `fs.file-max` + `LimitNOFILE` | tùy | mọi server thật sự | — (nhưng đặt vô hạn che mất leak) |
| `kernel.pid_max`, `pids.max` (cgroup) | 32768/4M | máy nhiều thread; chống fork bomb | — |
| THP (`/sys/.../transparent_hugepage/enabled`) | always/madvise | Redis/Mongo/DB: madvise hoặc never | mất lợi ích THP cho workload phù hợp nó |

Quy tắc cuối cùng, cũng là quy tắc đầu tiên của tài liệu: **mỗi dòng tuning phải đi kèm (a) số liệu trước, (b) lý do bằng cơ chế — chương nào giải thích nó, (c) số liệu sau, (d) ghi chú lại cho người sau.** Sysctl không có (b) là mê tín; không có (d) là bẫy hẹn giờ cho đồng nghiệp tương lai.

---

## Lời kết

Quay lại câu hỏi mở đầu: *hệ điều hành thực chất là gì?* Sau 17 chương, câu trả lời không còn là định nghĩa — mà là phản xạ:

- Khi thấy latency, bạn hỏi: hàng đợi nào? (run queue, accept queue, disk queue, lock queue — mọi latency là một hàng đợi ở đâu đó.)
- Khi thấy "đã ghi/đã gửi/đã cấp", bạn hỏi: *thật* chưa, hay mới vào buffer/cache/lời hứa overcommit?
- Khi thấy con số đẹp (CPU 60%, ping tốt, %util thấp), bạn hỏi: trung bình đang che cái tail nào, counter này *đo cái gì thật sự*?
- Khi tuning, bạn hỏi: mình đang dịch chuyển trade-off nào, và trả giá ở đâu?

Kernel bán ảo ảnh — và giờ bạn biết giá niêm yết của từng món.
