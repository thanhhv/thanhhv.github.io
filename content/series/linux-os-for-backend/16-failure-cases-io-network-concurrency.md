+++
title = "Chương 16 — Production Failure Cases (2): IO, Network & Concurrency"
date = "2026-02-21T23:00:00+07:00"
draft = false
tags = ["backend", "linux", "operating-system"]
series = ["Operating Systems cho Backend Engineer"]
+++

> Tiếp nối khung phân tích của chương 15.

---

## Case 10 — File Descriptor Exhausted

**Triệu chứng**: `accept: too many open files` / `EMFILE`; connection mới bị từ chối trong khi connection cũ vẫn chạy; có thể lan cả máy (`ENFILE` — trần toàn hệ thống).

**Bên trong kernel**: mỗi process có bảng FD (chương 04, `files_struct`) giới hạn bởi `RLIMIT_NOFILE` (mặc định soft 1024 lịch sử — quá thấp cho server!); toàn máy bởi `fs.file-max`. FD không chỉ là file: socket, pipe, epoll, timerfd, eventfd — **mọi thứ là FD**. Cạn FD thường là *leak*: quên close socket khi lỗi giữa chừng, response body không đóng (Go `resp.Body.Close()` — bug số một của HTTP client Go), connection pool cấu hình vô hạn.

**Điều tra**:
```
1. ls /proc/<pid>/fd | wc -l   so với  cat /proc/<pid>/limits | grep files
2. Loại gì đang chiếm: ls -l /proc/<pid>/fd | awk '{print $NF}' | sed 's/:.*//' | sort | uniq -c
   → socket đa số? → ss -p | grep <pid> → tới đích nào, state gì
   (đống CLOSE_WAIT = phía mình quên close sau khi bên kia đóng — leak rõ ràng)
3. Tăng theo thời gian? → leak; theo traffic? → limit thấp thật
```

**Khắc phục**: nâng limit (systemd `LimitNOFILE=65536`, không phải chỉ ulimit trong shell!); fix leak (defer close, timeout cho idle). **Phòng tránh**: metric FD count/limit mọi service (node_exporter có sẵn); lint/review pattern close; test lỗi (bên kia đóng đột ngột) trong CI.

---

## Case 11 — Disk IO 100% / IO chậm

**Triệu chứng**: `%wa` cao, load cao (D-state), mọi thứ chạm disk đều chậm; app timeout lác đác.

**Bên trong kernel** (chương 09): hàng đợi block layer dồn ứ — do (a) lượng IO vượt năng lực thiết bị (bão đọc do cache hụt, backup/compaction, thiếu index DB quét bảng); (b) thiết bị suy thoái (SSD hết over-provisioning, lỗi media, RAID rebuild, cloud volume hết burst credit — rất phổ biến trên EBS gp2/gp3!); (c) writeback storm (dirty dồn — chương 09 case mẫu).

**Điều tra**:
```
1. iostat -x 1: await tăng so baseline? aqu-sz sâu? r/s w/s bao nhiêu?
   (nhớ: %util vô nghĩa với NVMe — nhìn await + aqu-sz)
2. AI gây: iotop -o; biosnoop-bpfcc (từng IO: pid, size, latency, sector)
3. Kiểu IO: đọc hay ghi? tuần tự hay ngẫu nhiên (sector nhảy loạn trong biosnoop)?
4. Cloud: dashboard volume (IOPS/throughput/burst balance) — chạm trần thuê bao?
5. dmesg: I/O error, reset link NVMe? smartctl -a: media wearout?
```

**Khắc phục**: theo nguồn — thêm cache/index (giảm IO là vua), giới hạn batch job (cgroup io.max, ionice), nâng cấp volume/tách disk (WAL riêng!), fix burst credit. **Phòng tránh**: alert theo *latency* (await, biolatency p99) chứ không phải %util; capacity planning IOPS; tách ổ theo vai trò.

---

## Case 12 — fsync chậm

**Triệu chứng**: p99 của *ghi* (commit DB, produce Kafka acks=all, ghi WAL) tăng vọt trong khi đọc bình thường; DB log "checkpoint took too long" / "WAL flush slow".

**Bên trong kernel** (chương 09): fsync = flush dirty page của file + chờ journal commit + FLUSH cache thiết bị. Chậm khi: hàng đợi disk đang ứ IO khác (fsync xếp sau — ưu tiên kém), journal của FS thành điểm tuần tự hóa (nhiều fsync đồng thời chờ nhau commit transaction chung — ext4 một journal!), thiết bị không có write cache an toàn (consumer SSD: FLUSH thật sự đắt; enterprise có PLP nên FLUSH ~rẻ), hoặc cloud volume trần IOPS.

**Điều tra**:
```
1. Đo trực tiếp: bpftrace -e 'kprobe:vfs_fsync { @start[tid]=nsecs; }
   kretprobe:vfs_fsync /@start[tid]/ { @us=hist((nsecs-@start[tid])/1000); delete(@start[tid]); }'
   → histogram latency fsync thật, theo thời gian thực
2. ext4: cat /proc/fs/jbd2/<dev>-8/info → thời gian transaction journal
3. iostat -x: disk đang bận vì ai? (Case 11) — biosnoop lọc flush (F)
4. Phần cứng: loại SSD? PLP? cloud volume loại gì?
```

**Khắc phục**: tách WAL/journal sang thiết bị riêng ít tranh chấp; group commit (DB đã có — kiểm tra config `commit_delay`...); đánh giá lại mức bền cần thiết (`synchronous_commit=off` cho data chấp nhận mất 200ms? Kafka dựa replication?); phần cứng có PLP. **Phòng tránh**: benchmark fsync (`fio --fsync=1`) khi chọn phần cứng/volume — con số này quyết định trần commit-rate của mọi database đặt lên.

---

## Case 13 — inode đầy

**Triệu chứng**: "No space left on device" nhưng `df -h` còn 40% trống — dòng gây bối rối kinh điển.

**Bên trong kernel** (chương 09): ext4 cấp số inode cố định lúc mkfs (mặc định ~1 inode/16KB); hàng chục triệu file bé (session file, cache PHP, maildir, artifact CI, **container layer bỏ quên**) ăn hết inode trước khi hết byte. XFS cấp inode động — hiếm gặp hơn.

**Điều tra**:
```
1. df -i → IUse% 100%
2. Ổ đâu ra triệu file: du --inodes -xd3 / | sort -n | tail -20
3. Thủ phạm quen mặt: /var/lib/docker (layer/volume mồ côi), /tmp session,
   thư mục cache của framework, log chia file theo phút
```

**Khắc phục**: xóa có chủ đích (`docker system prune`, dọn cache theo tuổi — cẩn thận `rm` hàng triệu file cũng là bão IO, dùng `find -delete` từng phần giờ thấp điểm); dài hạn: mkfs lại với `-i` nhỏ hơn hoặc chuyển XFS. **Phòng tránh**: alert `df -i` song song `df -h` (nhiều nơi chỉ có cái sau!); thiết kế tránh triệu-file-nhỏ (bucket, SQLite, object store).

---

## Case 14 — TCP Backlog Full

**Triệu chứng**: client chập chờn "connection timeout/refused" khi tải cao; server "có vẻ khỏe" (CPU/RAM ổn).

**Bên trong kernel** (chương 11): accept queue đầy vì app gọi `accept()` chậm hơn tốc độ connection hoàn tất handshake (GC pause, thread accept kẹt, đơn giản là tải tăng). Kernel drop ACK cuối/SYN — client kẹt hoặc retry.

**Điều tra**:
```
1. nstat -az | grep -Ei 'ListenOverflows|ListenDrops' → tăng = chẩn đoán XONG
2. ss -lnt → cột Recv-Q (đang đầy) vs Send-Q (sức chứa) của listen socket
3. Vì sao accept chậm: GC log? thread accept đang làm gì (py-spy/jstack/pprof)?
4. somaxconn & backlog app: sysctl net.core.somaxconn; giá trị listen() trong code
```

**Khắc phục**: nâng `somaxconn` + backlog của app (mua thời gian sống qua burst/pause); fix nguồn accept chậm; nhiều acceptor + `SO_REUSEPORT`. **Phòng tránh**: alert ListenOverflows > 0 (phải luôn = 0); load test có burst hình răng cưa chứ không chỉ tải phẳng.

---

## Case 15 — SYN Flood

**Triệu chứng**: đột ngột không nhận connection mới; `dmesg`: "possible SYN flooding on port 443"; SYN queue đầy các entry half-open từ IP giả.

**Bên trong kernel** (chương 11): kẻ tấn công gửi bão SYN không hoàn tất handshake → SYN queue (chứa request state chờ ACK) đầy → SYN thật bị drop. Kernel phòng vệ bằng **syncookies**: khi queue đầy, thay vì lưu state, mã hóa state vào chính sequence number của SYN-ACK — ACK quay lại giải mã ra state → không tốn memory cho half-open. Trả giá: mất một số TCP option (window scale bị hạn chế...) trong lúc kích hoạt.

**Điều tra**:
```
1. nstat | grep -i syn → TcpExtTCPSynRetrans? SyncookiesSent tăng = đang bị + đang chống
2. ss -ntp state syn-recv | head; đếm: ss -n state syn-recv | wc -l
3. Nguồn: tcpdump -c 1000 'tcp[13]=2' → phân bố src IP (giả mạo thì vô nghĩa,
   nhưng biết được prefix/pattern)
```

**Khắc phục**: xác nhận `net.ipv4.tcp_syncookies=1` (mặc định); tăng `tcp_max_syn_backlog`; chặn upstream (network ACL, XDP/eBPF drop sớm — rẻ hơn nghìn lần so với để stack xử lý, Anycast/CDN/DDoS provider cho tấn công lớn). **Phòng tránh**: expose qua LB/CDN có chống DDoS thay vì IP trần; alert SyncookiesSent; diễn tập.

---

## Case 16 — Epoll Bug (edge-trigger bỏ đói connection)

**Triệu chứng**: một tỷ lệ nhỏ connection "đóng băng" — client chờ mãi trong khi server tưởng không có gì để làm; xuất hiện ngẫu nhiên, tải càng cao càng nhiều; restart hết ngay (rồi quay lại).

**Bên trong kernel** (chương 10): ET chỉ báo khi socket *chuyển* rỗng→có-data. Bug app: đọc một phần rồi quay lại chờ (không đọc tới EAGAIN) → data còn nhưng không bao giờ có event mới → connection kẹt vĩnh viễn. Biến thể: hai thread cùng xử lý một fd (event nuốt lẫn nhau), quên đăng ký lại EPOLLONESHOT, xử lý accept với ET mà không accept-cạn.

**Điều tra**:
```
1. Nhận diện connection kẹt: ss -tmi | grep -B1 -A1 'rcv_data' → socket có
   Recv-Q > 0 đứng yên lâu trong khi app "rảnh" = bằng chứng vàng
2. App đang chờ gì: pprof/jstack → thread event loop ngủ trong epoll_wait
3. Đọc code đường xử lý EPOLLIN: có vòng while đọc tới EAGAIN không?
   EPOLLONESHOT có re-arm không?
```

**Khắc phục**: sửa handler đọc-cạn; hoặc chuyển LT (đơn giản, chấp nhận thừa wakeup). **Phòng tránh**: dùng thư viện event loop đã kiểm chứng (libuv, netty, Go netpoller) thay vì tự viết trừ khi là nghề của bạn; test với data lớn hơn buffer đọc một lần (điều kiện kích hoạt bug); metric "socket Recv-Q>0 kéo dài" như canary.

---

## Case 17 — Thread Explosion

**Triệu chứng**: số thread của một process tăng từ trăm lên chục nghìn; memory tăng (kernel stack ~16KB + stack user); context switch bùng nổ (Case 3); có thể chạm `kernel.threads-max`/`pids.max` → không tạo nổi thread mới → crash dây chuyền.

**Bên trong kernel**: mỗi thread = task_struct + kernel stack (chương 05) — kernel không ngăn bạn tự sát bằng số lượng (trừ pids cgroup). Nguồn nổ điển hình: thread-per-request không giới hạn khi downstream chậm lại (mỗi request treo giữ một thread — chậm càng chậm); thread pool "cached/unbounded" (Java `newCachedThreadPool` dưới tải là quả bom); Go: blocking syscall/cgo dồn dập đẻ M (chương 05).

**Điều tra**:
```
1. cat /proc/<pid>/status | grep Threads — theo dõi theo thời gian
2. Thread đang làm gì: eu-stack -p <pid> / jstack / Go: pprof goroutine +
   go_threads metric → đa số thread block ở đâu? (thường: chờ downstream)
3. Đối chiếu timeline với latency downstream → thấy quan hệ nhân quả
```

**Khắc phục**: chặn nguồn (timeout cho mọi call ra ngoài! circuit breaker); giới hạn pool + hàng đợi có giới hạn + từ chối sớm (load shedding — fail fast hơn là chết chùm). **Phòng tránh**: nguyên tắc "mọi tài nguyên có giới hạn tường minh"; alert thread count; load test kịch bản *downstream chậm* (không chỉ downstream chết — chậm nguy hiểm hơn chết).

---

## Case 18 — Fork Bomb

**Triệu chứng**: máy tê liệt trong vài giây; không fork nổi lệnh mới để cứu (`ssh` được nhưng `ps` fail); load số nghìn.

**Bên trong kernel**: process đẻ theo cấp số nhân chiếm sạch PID + task_struct + run queue. Kinh điển `:(){ :|:& };:` — nhưng bản production thường *vô tình*: script retry-khi-fail tự gọi mình, supervisor restart-loop không backoff, CI runner đệ quy.

**Điều tra & Khắc phục**:
```
1. Nếu còn shell: exec ps (exec không fork!) / đọc /proc trực tiếp
2. Chặn: kill -STOP cả nhóm trước (SIGKILL từng con = whack-a-mole,
   chúng đẻ nhanh hơn bạn giết) → rồi kill -KILL cả session/cgroup:
   systemctl kill -s SIGKILL <unit> (giết theo CGROUP — đúng bài)
3. Không vào nổi → reboot (chấp nhận), rồi phân tích nguồn từ log/audit
```

**Phòng tránh**: `pids.max` cho MỌI unit/pod (systemd `TasksMax`, K8s pids limit, mặc định đã có nhưng kiểm tra); user limit `ulimit -u`; restart policy có backoff; review script retry.

---

## Case 19 — Zombie Process

(Đã phân tích cơ chế đầy đủ ở chương 04 §4.4 và case cạn PID §8.) Tóm tắt tác chiến: zombie = con chết chưa được `wait()`. Vài con vô hại; tích lũy = cha bug. `ps -eo stat,ppid | awk '$1~/Z/{print $2}' | sort | uniq -c` → tìm cha; fix cha hoặc restart cha (zombie về init được reap). Container: PID 1 phải biết reap (`--init`/tini). Alert số zombie > 100. **Không thể kill zombie — nó đã chết rồi.**

---

## Case 20 — Deadlock

**Triệu chứng**: một nhóm thread/goroutine đứng im vĩnh viễn; request liên quan treo tới timeout; CPU của nhóm đó = 0 (khác livelock!); phần còn lại của service có thể vẫn chạy → sự cố "một phần tính năng chết" khó thấy.

**Bên trong kernel**: kernel *không biết* app deadlock — các thread ngủ ngoan trong futex_wait như mọi thread chờ lock khác, không bao giờ được đánh thức. Bốn điều kiện Coffman hội đủ, thường là **lock ordering ngược**: T1 giữ A chờ B, T2 giữ B chờ A (chương 08); biến thể hiện đại: goroutine chờ channel không ai gửi, connection pool cạn + giữ connection đi xin connection nữa (self-deadlock qua pool!), lock + chờ chính worker pool đang bị mình chặn.

**Điều tra**:
```
1. Chụp stack TOÀN BỘ: Go: /debug/pprof/goroutine?debug=2 (goroutine "chan
   receive, 47 minutes" = ngay đó); Java: jstack (tự phát hiện "Found one
   Java-level deadlock"!); C: gdb -p → thread apply all bt
2. Tìm chu trình: ai giữ gì chờ gì — futex thì địa chỉ lock trong frame
3. Kernel-side hiếm hơn: task D-state mãi mãi + /proc/<pid>/stack (lock FS/NFS)
```

**Khắc phục**: restart (deadlock không tự khỏi); rồi fix: thứ tự lock toàn cục, `TryLock` + timeout, thu hẹp critical section, tách pool. **Phòng tránh**: quy ước lock ordering trong docs kiến trúc; Go race detector + `go vet`; timeout cho *mọi* thao tác chờ (không có "chờ vô hạn" trong production); test hỗn loạn concurrency.

---

## Case 21 — Livelock

**Triệu chứng**: như deadlock nhưng **CPU 100%**: các bên liên tục hoạt động (retry, back-off rồi lại đụng nhau) mà không bên nào tiến triển. Ví dụ thật: hai service retry lẫn nhau tức thời khi lỗi → bão retry tự duy trì sau một trục trặc thoáng qua (retry storm = livelock phân tán — kịch bản sập dây chuyền phổ biến nhất của microservice); CAS loop nóng dưới contention cực đại; hai transaction DB deadlock-detector giết rồi retry đồng thời mãi.

**Điều tra**: profile thấy vòng lặp retry/CAS chiếm CPU; log lỗi lặp chu kỳ; đồ thị traffic nội bộ gấp nhiều lần traffic vào (khuếch đại retry).

**Khắc phục & phòng tránh**: **exponential backoff + jitter** (jitter là phần hay bị quên — không có nó mọi client đồng bộ hóa nhịp retry!); retry budget (ví dụ tối đa 20% traffic là retry); circuit breaker; ở tầng lock: hàng đợi (ticket/futex) thay vì spin tự do. Nguyên lý: hệ có retry là hệ có vòng phản hồi dương tiềm ẩn — phải thiết kế phần tử cản.

---

## Case 22 — Priority Inversion

**Triệu chứng**: task ưu tiên cao (RT hoặc nice thấp) thỉnh thoảng trễ *rất* lâu một cách không thể hiểu — đúng bằng thời gian chạy của các task ưu tiên *trung bình* chẳng liên quan.

**Bên trong kernel** (chương 06, 08): H (cao) chờ lock mà L (thấp) đang giữ; M (giữa) preempt L → L không chạy được để nhả lock → **H chờ M** — đảo ngược ưu tiên. Kinh điển: Mars Pathfinder 1997. Linux giải bằng **priority inheritance** cho futex PI (`pthread_mutexattr_setprotocol(PTHREAD_PRIO_INHERIT)`): L tạm "mượn" ưu tiên của H khi giữ lock H cần; rt_mutex trong kernel có PI sẵn. Nhưng: mutex thường **không** có PI, và PI không cứu được nếu chuỗi chờ đi qua tài nguyên không phải lock (queue, IO).

**Điều tra**: chỉ hiện hình khi trace scheduling — `perf sched timehist` / ftrace `sched_switch` quanh thời điểm trễ: thấy H ở trạng thái chờ lock, L runnable-nhưng-không-được-chạy, M chiếm CPU.

**Khắc phục & phòng tránh**: nếu buộc phải trộn RT + lock chia sẻ → PI mutex, giữ critical section cực ngắn, cấp CPU riêng cho RT (isolcpus/cpuset); tốt hơn: **đừng chia sẻ lock giữa các mức ưu tiên khác nhau** — giao tiếp qua lock-free queue. Với backend thường (không RT): bài học áp dụng cho nice/cgroup weight chênh lệch lớn + lock chung — cùng một hình dạng vấn đề, mức độ nhẹ hơn.

---

## Phụ lục nhanh — Memory Fragmentation (bổ sung Case 5/7)

RAM còn nhiều nhưng cấp phát *liên tục vật lý* thất bại (huge page, DMA buffer, driver): `/proc/buddyinfo` thấy cạn block bậc cao. Triệu chứng: THP allocation stall (app khựng khi kernel compact), `page allocation failure` trong dmesg ở driver. Xử lý: `echo 1 > /proc/sys/vm/compact_memory` (tạm), giảm phụ thuộc huge page động (cấp hugetlbfs lúc boot), theo dõi `/proc/pagetypeinfo`. User-space fragmentation (heap allocator) → RSS cao dai dẳng: đổi allocator (jemalloc), restart định kỳ, thiết kế pool theo size-class.

---

*Tiếp: [Chương 17 — Kiến trúc thực tế](/series/linux-os-for-backend/17-kien-truc-thuc-te/): soi các cơ chế trên vào PostgreSQL, Redis, Kafka, Nginx, Go, Node.js, Docker, Kubernetes.*
