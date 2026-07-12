+++
title = "Chương 13 — Container: không phải máy ảo, mà là ảo ảnh do kernel dựng"
date = "2026-02-21T20:00:00+07:00"
draft = false
tags = ["backend", "linux", "operating-system"]
series = ["Operating Systems cho Backend Engineer"]
+++

## 1. Problem Statement

Bài toán kinh doanh: đóng gói app + dependency thành một đơn vị chạy được ở mọi nơi, khởi động trong milli-giây, nhét hàng chục cái vào một máy, cô lập vừa đủ để không giẫm chân nhau. VM giải được cô lập nhưng trả giá: mỗi VM một kernel + OS riêng (GB memory, khởi động chục giây, density thấp).

Câu trả lời của Linux: **không cần máy ảo — chỉ cần nói dối process một cách có hệ thống.** Container = process bình thường + ba lớp cơ chế kernel: **namespace** (thấy gì), **cgroup** (dùng bao nhiêu), **capability/seccomp/LSM** (làm được gì — chương 14). Không có "Container Kernel Object" nào cả — `docker` chỉ là trình lắp ráp các cơ chế này.

Nếu không có container: quay lại thời "chạy được trên máy tôi", mỗi service một VM lãng phí, deploy tính bằng phút.

## 2. Tại sao nó tồn tại — và vì sao KHÔNG phải VM

```
        VM                                    CONTAINER
┌─────────────────────┐              ┌─────────────────────┐
│ App                 │              │ App  ← process Linux │
│ Guest OS + KERNEL   │              │ (không có OS con!)   │
├─────────────────────┤              ├─────────────────────┤
│ Hypervisor ảo hóa   │              │ MỘT kernel chung,    │
│ PHẦN CỨNG           │              │ ảo hóa GÓC NHÌN      │
└─────────────────────┘              └─────────────────────┘
Ranh giới: CPU (VT-x), như 2 máy     Ranh giới: syscall interface,
Cô lập: mạnh (khác kernel)           như 2 process. Cô lập: yếu hơn —
Giá: GB RAM/VM, boot ~10s            chung kernel = chung bug kernel.
                                     Giá: ~MB, boot ~ms, density gấp 10x
```

Hệ quả nhìn thấy ngay: `uname -r` trong mọi container trên một máy trả về **cùng một kernel**; container "Ubuntu" trên host "Fedora" chỉ là *userland* Ubuntu (glibc, apt, file layout) chạy trên kernel Fedora. Suy ra hai điều production: (1) không thể chạy container đòi kernel khác (module, sysctl riêng — có loại là per-namespace, có loại là toàn máy); (2) **một lỗ hổng kernel = thoát mọi container trên máy** → multi-tenant thật sự cần thêm lớp (gVisor, Kata, Firecracker — microVM: lấy lại ranh giới VM nhưng tối giản để boot ~100ms).

## 3. Internal Architecture

### 3.1. Namespace — ảo hóa "cái nhìn thấy"

8 loại, mỗi loại nói dối về một thứ (tạo bằng flags của `clone()`/`unshare()` — đúng syscall ở chương 04/05):

| Namespace | Nói dối về | Ví dụ hệ quả |
|---|---|---|
| **PID** | Cây process | App là PID 1 *bên trong* (nhưng là PID 34567 nhìn từ host — hai số cùng một task_struct!). PID 1 có trách nhiệm reap zombie + không có default signal handler → hai bẫy ở chương 04 |
| **Mount** | Cây filesystem | Root `/` của container là image; không thấy filesystem host |
| **Network** | Toàn bộ stack mạng | Interface, IP, routing table, iptables, port RIÊNG — 2 container cùng listen :8080 vô tư; nối ra ngoài bằng veth pair + bridge/NAT (đây là phần Docker networking) |
| **UTS** | hostname | mỗi container một hostname |
| **IPC** | System V IPC, shm | không thấy shared memory của nhau |
| **User** | uid/gid mapping | root(0) trong container = uid 100000 không quyền trên host — nền của rootless container |
| **cgroup** | gốc cây cgroup | che cấu trúc cgroup của host |
| **Time** (5.6+) | clock offset | ít dùng |

### 3.2. cgroup v2 — ảo hóa "cái được dùng"

Cây phân cấp tại `/sys/fs/cgroup`; process thuộc một node; controller áp giới hạn theo node và phân phối xuống con:

```
cpu:     cpu.weight (chia sẻ tương đối khi tranh chấp — mềm)
         cpu.max = "50000 100000" (quota/period: 50ms CPU mỗi 100ms — CỨNG)
           ← nguồn của THROTTLING: dùng hết quota giữa period → MỌI thread
             của container ĐỨNG HÌNH đến period sau, kể cả đang giữa request!
             Số liệu: cpu.stat → nr_throttled, throttled_usec — PHẢI có trên
             dashboard của mọi service có CPU limit
memory:  memory.max (cứng → vượt là OOM kill CỤC BỘ trong cgroup)
         memory.low (bảo vệ mềm), memory.current
           ← memory.current GỒM CẢ page cache của container → "memory tăng
             mãi" khi ghi log/file nhiều là cache, không phải leak (chương 07)
io:      io.max (IOPS/bandwidth), io.latency, io.weight
pids:    pids.max (chống fork bomb — case chương 16)
```

Kubernetes requests/limits chỉ là cách viết khác: requests.cpu → cpu.weight (+ điểm cho scheduler đặt pod); limits.cpu → cpu.max; limits.memory → memory.max. **CPU limit = throttling, memory limit = OOM kill** — hai cơ chế phạt hoàn toàn khác nhau, hiểu sai là đọc sai mọi sự cố K8s.

### 3.3. OverlayFS — ảo hóa filesystem của image

```
Image = chồng layer CHỈ ĐỌC (mỗi lệnh Dockerfile một layer, chia sẻ giữa
        mọi container cùng base → 50 container Ubuntu = 1 bản base trên disk
        + page cache CHUNG cho file giống nhau — density là đây)
Container chạy = các layer đó + MỘT layer ghi (upperdir) qua overlayfs:
  đọc: tìm từ trên xuống — thấy ở layer nào lấy layer đó
  ghi file thuộc layer dưới: COPY-UP toàn bộ file lên upper rồi mới sửa
    ← ghi 1 byte vào file 2GB thuộc image = copy 2GB (lần đầu)!
  xóa file layer dưới: đặt "whiteout" che đi (không xóa thật)
```

Quy tắc rút ra: **dữ liệu ghi nhiều (database, log, WAL) phải nằm trên volume** (bind mount/volume — filesystem thật, bypass overlay), không nằm trong container layer. Container layer mất khi container bị xóa — vốn dĩ được thiết kế để vứt.

### 3.4. Runtime và OCI — ai lắp ráp tất cả

```
docker CLI → dockerd → containerd (quản lý image, lifecycle)
                          → runc (binary nhỏ: đọc config.json OCI →
                            gọi clone() với các CLONE_NEW* + viết cgroup
                            + pivot_root vào overlay + drop capability
                            + nạp seccomp → exec app → THOÁT)
```

Chuẩn **OCI** (image spec + runtime spec) là lý do hệ sinh thái không vỡ: image build bằng Docker chạy được trên containerd/CRI-O/podman; runc thay được bằng gVisor (runsc), Kata. Lưu ý vận hành: sau khi runc thoát, **container = process con của containerd-shim** — không có "daemon ảo hóa" nào đứng giữa app và kernel lúc runtime. Chi phí CPU/syscall của container ≈ **0** (vẫn là process gọi thẳng syscall); chi phí thật nằm ở: overlay (IO metadata), NAT/veth (network hop), và các *giới hạn* bạn tự đặt.

## 4. Cách hoạt động — `docker run` dưới kính hiển vi

```
docker run --cpus 2 --memory 4g -p 8080:80 myapp
1. containerd: kéo/mở image → chuẩn bị snapshot overlayfs (lower=image layers)
2. Tạo network namespace mới; veth pair: một đầu vào ns, một đầu cắm bridge
   docker0; iptables DNAT: host:8080 → container_ip:80
3. runc: clone(CLONE_NEWPID|NEWNS|NEWNET|NEWUTS|NEWIPC|...)
4. Trong con: mount /proc mới (để ps trong container chỉ thấy mình),
   pivot_root vào overlay merged dir, mount volume
5. Ghi cgroup: cpu.max="200000 100000", memory.max=4G, gắn PID vào
6. Drop capabilities (giữ ~14 cái mặc định), nạp seccomp profile mặc định
   (chặn ~40 syscall nguy hiểm), set no_new_privs
7. execve("/app/server") → TỪ ĐÂY: chỉ là một process Linux bình thường
   với cái nhìn bị bó hẹp — mọi chương trước của tài liệu áp dụng nguyên vẹn
```

## 5. Trade-off

| Quyết định | Được | Mất |
|---|---|---|
| Chung kernel | Density, boot ms, chi phí runtime ~0 | Cô lập yếu hơn VM; kernel bug là single point |
| Namespace | Cô lập cái-nhìn rẻ tiền | Ảo ảnh không hoàn hảo: /proc/meminfo, nproc... hiện số HOST → app đọc sai (bẫy lớn, xem §6) |
| cgroup CPU quota | Multi-tenant công bằng, tính tiền được | Throttling làm tail latency xấu theo cách rất khó thấy |
| cgroup memory.max | Chặn leak lan máy | OOM kill cục bộ đột ngột; tính cả page cache gây hiểu lầm |
| OverlayFS | Layer chia sẻ, build cache, ship nhanh | Copy-up file lớn; hiệu năng metadata kém FS thật; không dành cho data |
| Image immutable | Reproducible, rollback = đổi tag | Không patch tại chỗ; kích thước image thành vấn đề vận hành |

## 6. Production — những cái bẫy đặc thù container

- **App đọc tài nguyên của HOST**: `nproc`, `/proc/meminfo`, `runtime.NumCPU()` trả số của máy thật, không phải limit của cgroup. Hậu quả kinh điển: JVM heap default = 1/4 RAM *host* → OOMKilled; Go tạo GOMAXPROCS=64 P trên quota 2 CPU → throttling nặng (chương 05). Fix: JVM hiện đại đã container-aware (`UseContainerSupport`); Go 1.25+ tự nhận cgroup, trước đó dùng `automaxprocs`; mọi ngôn ngữ khác: kiểm tra thủ công.
- **Chẩn đoán throttling**: trong container đọc `/sys/fs/cgroup/cpu.stat` — `nr_throttled` tăng đều là bằng chứng. Triệu chứng bên ngoài: p99 có "bậc thang" ~quantum 100ms. Lối thoát: tăng limit, bỏ CPU limit (giữ requests — nhiều nơi lớn chạy K8s không CPU limit), hoặc static CPU policy cho pod nhạy.
- **OOMKilled (exit 137)**: xem `kubectl describe` / dmesg host. Nhớ: cache tính vào memory.current; kiểm tra app có ghi file lớn trong container FS không trước khi kết luận leak.
- Debug container "không có gì bên trong" (distroless): `kubectl debug` / `nsenter -t <pid> -n -m` từ host — vào *namespace* của container với *tool* của host. Đây là ứng dụng trực tiếp của việc hiểu container = namespace.
- PID 1: dùng `--init`/tini nếu app fork con (zombie — chương 04); handle SIGTERM (K8s grace period).

## 7. Anti-pattern

- Chạy database ghi mạnh trên overlay layer (thay vì volume) — chậm + mất data khi xóa container.
- Đặt CPU limit = requests một cách máy móc cho service latency-sensitive → tự tay bật throttling; hoặc memory limit sát RSS đỉnh → OOMKilled định kỳ như đồng hồ.
- Image chứa cả toolchain build (2GB, bề mặt tấn công) — multi-stage build còn 50MB.
- Chạy privileged / mount docker.sock vào container vì "cho tiện" — tương đương root máy host, vô hiệu toàn bộ chương 14.
- Coi container như VM: SSH vào sửa tay, cài package tại chỗ — mất reproducibility, "config drift" quay lại thời tiền container.

## 8. Failure Analysis — case mẫu: p99 gấp 10 lần sau khi chuyển lên Kubernetes

- **Triệu chứng**: service Go, bare metal p99=8ms; cùng code trên K8s (requests 500m, limits 2 CPU) p99=80ms từng đợt, p50 không đổi.
- **Điều tra**: `cpu.stat`: `nr_throttled` tăng vài chục lần mỗi phút; trace thấy các quãng đứng đúng bội số ~100ms (period). GOMAXPROCS=32 (số core node!).
- **Root cause**: 32 P của Go tiêu 200ms quota (2 CPU × 100ms) trong ~vài chục ms đầu period khi GC + burst request → cả process bị đóng băng phần period còn lại. Throttling là hành vi *đúng* của CFS bandwidth với cấu hình đã cho.
- **Fix**: GOMAXPROCS=2 (khớp limit) → burst được dàn phẳng; nâng limit lên 4; kết quả p99=11ms. **Prevention**: alert `container_cpu_cfs_throttled_periods_total` ratio > 1%; chuẩn hóa automaxprocs/Go mới toàn công ty; load test TRÊN K8s chứ không chỉ trên VM dev.

## 9. Khi nào không nên tối ưu

Đừng đổi runtime (containerd→CRI-O), FS driver, hay CNI vì hiệu năng *trước khi đo* — với đa số service, overhead container thực ~0 và bottleneck nằm ở limit tự cấu hình sai. Đừng dựng gVisor/Kata khi mọi workload là code nội bộ tin cậy — trả giá syscall overhead (gVisor) cho mối đe dọa không có thật với bạn. Và đừng "tối ưu density" ép memory limit sát sạt để nhét thêm pod — chuỗi OOMKilled + restart storm đắt hơn nhiều so với 20% RAM tiết kiệm được.

---

**Chương tiếp theo**: lớp thứ ba của container — và của mọi process: [Chương 14: Security](/series/linux-os-for-backend/14-security/) — capability, seccomp, LSM.
