+++
title = "Bài 4 — Container"
date = "2026-02-01T10:00:00+07:00"
draft = false
tags = ["backend", "infrastructure", "networking"]
series = ["Infrastructure & Networking"]
+++

# Module 04 — Container

> Điều kiện tiên quyết: Module 03 (namespace, cgroup, OOM). Container không phải VM nhẹ — nó là process Linux được cách ly. Toàn bộ module này triển khai từ câu đó.

---

## 1. Tại sao container tồn tại

### Problem Statement
Trước container, deploy có 2 lựa chọn tệ:
1. **Cài trực tiếp lên máy**: "works on my machine" — app phụ thuộc phiên bản libc, Python, thư viện hệ thống của máy đó. Hai app cần 2 phiên bản khác nhau của cùng thư viện = xung đột. Môi trường dev/staging/prod lệch nhau = lớp bug riêng.
2. **VM**: cách ly tốt nhưng mỗi VM chở nguyên một kernel + OS: boot hàng phút, tốn GB RAM cho phần không phải app, mật độ thấp.

Container giải cả hai: **đóng gói app + toàn bộ userspace dependency thành một artifact bất biến** (giải quyết "works on my machine"), chạy như **process thường trên kernel chung** (giải quyết chi phí VM — khởi động ms, overhead ~0).

Trade-off nền tảng đổi lại: **chung kernel = ranh giới bảo mật yếu hơn VM**. Kernel exploit từ container = chiếm host. Vì thế multi-tenant thù địch (chạy code của khách hàng) cần lớp thêm: gVisor, Kata (microVM), Firecracker — đó là lý do Lambda chạy Firecracker chứ không chạy container trần.

---

## 2. Docker Architecture và Container Runtime

### Cách hoạt động bên trong
`docker run` không phải một khối — nó là chuỗi ủy quyền, và hiểu chuỗi này giúp debug + hiểu vì sao Kubernetes bỏ Docker:

```
docker CLI → dockerd (API, build, volume, network)
           → containerd (quản lý lifecycle container, image)
             → containerd-shim (mỗi container 1 shim — giữ stdio, cho phép
               restart dockerd/containerd mà container KHÔNG chết)
               → runc (binary nhỏ: tạo namespace + cgroup, exec process, thoát)
                 → process của bạn
```

- **runc** = hiện thực chuẩn OCI runtime: nó chỉ làm đúng việc "tạo cách ly rồi exec" rồi biến mất. Container sau đó là process con của shim.
- **Kubernetes nói chuyện thẳng với containerd** (qua CRI) — bỏ dockerd vì không cần lớp đó. "Kubernetes removed Docker" chỉ là bỏ lớp CLI/daemon, image vẫn là OCI image, không có gì phải build lại.
- Vì container chỉ là process: `ps aux` trên host **thấy** process trong container; `kill` từ host giết được nó; `nsenter -t <pid> -a` chui vào mọi namespace của nó để debug — vũ khí mạnh khi container không có shell.

```bash
ctr -n k8s.io containers list        # nói chuyện trực tiếp với containerd
crictl ps; crictl logs <id>          # debug node Kubernetes không có docker CLI
nsenter -t <pid> -n ss -tnp          # xem socket BÊN TRONG container từ host
```

---

## 3. Image, Layer và Overlay Filesystem

### Problem Statement
Nếu mỗi image là một khối tarball nguyên khối: 100 service dùng chung Ubuntu base = lưu và kéo Ubuntu 100 lần; sửa 1 dòng code = đẩy lại toàn bộ GB. Cần cơ chế **chia sẻ và tái sử dụng phần chung**.

### Cách hoạt động
Image = chồng **layer bất biến**, mỗi layer là diff filesystem, định danh bằng content hash (nội dung giống nhau = layer dùng chung, cache được, kéo 1 lần). Khi chạy, **overlayfs** ghép các layer read-only (lowerdir) + 1 layer ghi (upperdir) thành một cây thống nhất:

```
Container thấy:   [merged view]
                   ├─ upperdir (RW — mọi thứ container ghi)
                   ├─ layer N: COPY app
                   ├─ layer 2: RUN pip install
                   └─ layer 1: ubuntu base (RO, chia sẻ giữa mọi container)
```

- Ghi vào file thuộc layer dưới → **copy-up toàn bộ file** lên upperdir rồi mới sửa. Sửa 1 byte của file 1GB = copy 1GB. Workload ghi nặng không được ghi vào overlay — dùng **volume** (bind mount, bypass overlay hoàn toàn).
- Upperdir chết cùng container. Tính bất biến này là **feature**: container là cattle; mọi state quan trọng phải ở volume hoặc dịch vụ ngoài.

### Kỹ thuật build image — nơi lý thuyết thành tiền và thành bảo mật
Layer cache bị vô hiệu từ **lệnh đầu tiên thay đổi trở xuống**, do đó thứ tự Dockerfile quyết định tốc độ build:

```dockerfile
# ĐÚNG: thứ ít đổi lên trên
COPY go.mod go.sum ./
RUN go mod download          # cache tồn tại chừng nào dependency không đổi
COPY . .                     # code đổi hàng ngày — chỉ từ đây trở xuống rebuild
RUN go build -o /app

# Multi-stage: build image ≠ runtime image
FROM golang:1.22 AS build
# ... build ...
FROM gcr.io/distroless/static
COPY --from=build /app /app
ENTRYPOINT ["/app"]          # exec form — nhớ bài học PID 1 module 03
```

Multi-stage + base tối thiểu (distroless/alpine) cho: image 20MB thay vì 1GB → kéo nhanh (autoscale nhanh hơn), **bề mặt tấn công nhỏ** (không shell, không package manager = kẻ xâm nhập không có công cụ), ít CVE phải vá.

**Anti-patterns build**:
- Tag `latest` trong production → không biết đang chạy gì, không rollback được về "phiên bản cũ" vì latest đã bị ghi đè. Dùng tag immutable (git SHA) hoặc digest.
- Secret trong layer (`COPY key.pem` rồi `RUN rm key.pem` — **file vẫn nằm trong layer trước**, `docker history` + extract là lộ). Dùng BuildKit `--mount=type=secret`.
- `RUN apt-get upgrade` trong Dockerfile → image không tái lập được (build hôm nay khác hôm qua).

---

## 4. Container Networking

### Cách hoạt động
Mỗi container có net namespace riêng — tức một network stack trống. Nối ra ngoài bằng **veth pair** (ống ảo 2 đầu): một đầu trong container (thành `eth0`), một đầu cắm vào bridge trên host (`docker0`). Ra Internet: iptables MASQUERADE (SNAT). Vào container: DNAT (`-p 8080:80`).

```
[container eth0]──veth──[docker0 bridge]──iptables NAT──[host eth0]──Internet
```

Hệ quả cần biết:
- Container-to-container cùng bridge: đi thẳng qua bridge, nhanh. Khác host: cần overlay (VXLAN — encapsulate, tốn ~5–10% throughput + CPU) hoặc routing thật (Calico BGP, cloud CNI cấp IP thật cho pod — nhanh hơn, đây là hướng của EKS/GKE).
- `--network=host`: bỏ cách ly mạng, hiệu năng bằng host — dùng cho workload network-intensive (proxy, monitoring agent), đổi lại mất cách ly port.
- Docker port mapping là iptables: nhiều rule = chậm dần; conntrack áp dụng (bài học module 01 về conntrack full vẫn nguyên giá trị).

---

## 5. Storage

Ba loại, chọn đúng loại là hết 80% vấn đề:

| Loại | Bản chất | Dùng cho |
|---|---|---|
| Overlay (mặc định) | COW, chết cùng container | Chỉ filesystem của app, KHÔNG ghi dữ liệu |
| Volume / bind mount | Bypass overlay, I/O = native | Dữ liệu cần bền, ghi nhiều (DB, log) |
| tmpfs | RAM | Scratch nhạy cảm (secret), nhanh, mất khi dừng |

Chạy database trong container hoàn toàn ổn **miễn là data ở volume** — hiệu năng I/O qua bind mount ≈ native. Điều làm "DB trong container" nguy hiểm không phải container, mà là orchestrator di chuyển/giết container thiếu suy nghĩ (xem StatefulSet, module 05).

---

## 6. Container Security

### Mô hình đe dọa: ranh giới là kernel
Mặc định Docker: root trong container = **root thật** (UID 0) bị giới hạn capability + seccomp. Thoát ra ngoài (container escape) qua: kernel exploit, misconfiguration (`--privileged`, mount docker.sock), hoặc capability thừa.

Phòng thủ theo lớp — thứ tự hiệu quả trên chi phí:
1. **Chạy non-root** (`USER app` trong Dockerfile). Một dòng, chặn cả lớp tấn công. Không có lý do gì app backend chạy UID 0.
2. **Read-only root filesystem** (`readOnlyRootFilesystem: true` trong k8s): kẻ xâm nhập không ghi được binary.
3. **Drop capabilities**: mặc định drop ALL, thêm lại đúng cái cần (thường là không cần gì; `NET_BIND_SERVICE` nếu bind port <1024).
4. **Không bao giờ**: `--privileged` (tắt toàn bộ cách ly), mount `/var/run/docker.sock` vào container (= trao quyền root host; CI runner hay phạm — dùng rootless buildkit/kaniko thay thế).
5. **Scan image + ký image** trong CI (trivy/grype; cosign): chặn CVE và bảo đảm image chạy đúng là image đã build.
6. Multi-tenant thù địch → nâng cấp ranh giới: gVisor/Kata/Firecracker như đã nói.

### Checklist image production
- [ ] Base tối thiểu (distroless/chainguard/alpine), multi-stage build
- [ ] USER non-root, exec-form ENTRYPOINT, xử lý SIGTERM
- [ ] Tag immutable, ký image, scan trong CI, không secret trong layer
- [ ] .dockerignore (không COPY .git, .env)
- [ ] HEALTHCHECK hoặc probe khai báo ở orchestrator
