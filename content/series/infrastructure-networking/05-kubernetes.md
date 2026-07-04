+++
title = "Bài 5 — Kubernetes"
date = "2026-02-01T11:00:00+07:00"
draft = false
tags = ["backend", "infrastructure", "networking"]
series = ["Infrastructure & Networking"]
+++

# Module 05 — Kubernetes

> Kubernetes không phải "công cụ chạy container". Nó là một **hệ điều khiển hội tụ trạng thái** (desired state reconciliation). Nắm được mô hình đó, mọi thứ còn lại là chi tiết.

---

## 1. Tại sao Kubernetes tồn tại và mô hình tư duy cốt lõi

### Problem Statement
Container giải bài toán đóng gói cho MỘT máy. Ở quy mô nhiều máy, các câu hỏi mới xuất hiện: đặt container nào lên máy nào (bin packing)? Máy chết thì container đi đâu? Traffic tìm container đang di chuyển thế nào? Rollout 200 instance không downtime bằng cách nào? Trước Kubernetes, mỗi công ty tự viết script/orchestrator riêng — Google đúc kết 15 năm chạy Borg thành Kubernetes.

### Mô hình cốt lõi: declarative + reconciliation
Bạn không ra lệnh ("chạy 5 container"). Bạn **khai báo trạng thái mong muốn** vào etcd ("tồn tại 5 replica"), và các **controller** chạy vòng lặp vô hạn:

```
loop {
  observed = trạng thái thật (qua API)
  desired  = trạng thái khai báo (etcd)
  diff → hành động để tiến gần desired
}
```

Vì sao mô hình này thắng: **tự chữa lành là hệ quả miễn phí** — node chết làm observed lệch desired, controller tự bù, không cần "code xử lý node chết" riêng. Lệnh imperative fail giữa chừng thì kẹt; reconciliation loop chỉ cần chạy lại. Đây cũng là điểm yếu của nó: hệ thống **hội tụ dần**, không tức thời — `kubectl apply` thành công không có nghĩa hệ thống đã ở trạng thái mới (nguồn hiểu lầm số 1 khi viết automation trên Kubernetes).

### Control plane
```
kubectl → API Server (cổng duy nhất, authn/authz, validation)
              │
            etcd (nguồn sự thật duy nhất — Raft consensus, cần quorum lẻ)
              │ (watch)
   ┌──────────┼────────────┐
Scheduler  Controller    kubelet (mỗi node — biến PodSpec thành container
(chọn node) Manager        thật qua CRI→containerd, chạy probe, báo status)
            (vòng lặp     kube-proxy (lập trình iptables/IPVS cho Service)
             reconcile)
```
Mọi thành phần chỉ nói chuyện với API server, phối hợp qua **watch** (long-poll trên etcd change stream) — không ai gọi trực tiếp ai. Kiến trúc này chịu được từng phần chết: control plane sập thì **workload đang chạy vẫn chạy** (chỉ mất khả năng thay đổi/tự chữa) — điểm thiết kế quan trọng ít người biết.

**etcd là trái tim và là giới hạn**: mọi object, mọi watch đi qua nó. Cluster lớn chết vì etcd trước tiên (quá nhiều object, quá nhiều watch từ operator viết ẩu, event spam). Bài học vận hành: giám sát etcd latency (`etcd_disk_wal_fsync_duration`), etcd cần disk nhanh (fsync! — bài học module 03), backup etcd = backup cả cluster.

---

## 2. Pod — đơn vị nhỏ nhất, và vì sao không phải là container

Pod = nhóm container **chia sẻ network namespace** (cùng IP, localhost với nhau) và có thể chia sẻ volume — được đặt lịch như một khối. Lý do tồn tại: có những cặp process cần sống chết cùng nhau và nói chuyện qua localhost (app + sidecar proxy trong service mesh, app + log shipper). Pod là **cattle tuyệt đối**: IP thay đổi mỗi lần tái sinh, không sửa tại chỗ, mọi thứ bền vững phải nằm ngoài Pod.

### Probe — hợp đồng giữa app và platform, nơi sự cố hay bắt nguồn
| Probe | Câu hỏi | Fail thì |
|---|---|---|
| liveness | Process còn cứu được không? | **Restart container** |
| readiness | Nhận traffic được không? | Rút khỏi Service endpoints (không restart) |
| startup | Khởi động xong chưa? | Trì hoãn 2 probe kia (app khởi động chậm) |

Sai lầm chết người: **liveness probe check dependency** (DB, service khác). DB chậm 1 phút → liveness fail toàn fleet → Kubernetes restart TẤT CẢ pod cùng lúc → sự cố nhỏ thành outage lớn, lặp vô hạn (CrashLoopBackOff hàng loạt). Quy tắc: liveness chỉ check "process tự nó còn tốt" (deadlock detection); dependency thuộc về readiness — và readiness fail đồng loạt cũng phải cân nhắc (mất hết endpoint = LB không còn chỗ gửi).

### Requests/Limits và QoS
`requests` = dùng để **đặt lịch** (scheduler) và tỷ lệ chia sẻ khi tranh chấp; `limits` = trần cứng (CPU: throttle — xem module 03; memory: OOM kill). QoS: Guaranteed (req=limit) > Burstable > BestEffort — quyết định thứ tự bị **evict** khi node cạn tài nguyên. Khuyến nghị thực dụng: memory req = limit (tránh eviction bất ngờ, vì memory không nén được); CPU đặt request, cân nhắc **không đặt limit** (tránh throttling; CPU nén được, tranh chấp chỉ làm chậm theo tỷ lệ request).

---

## 3. Workload Controllers

- **Deployment** (qua ReplicaSet): stateless, mọi pod thay thế được nhau. Rolling update = tạo RS mới, tăng dần/giảm dần theo `maxSurge`/`maxUnavailable`. 95% workload là đây.
- **StatefulSet**: khi pod CẦN danh tính bền (tên ổn định `db-0`, `db-1`; PVC riêng theo tên; thứ tự khởi động/dừng). Dùng cho database, Kafka, hệ có leader election. Đánh đổi: rollout chậm (tuần tự), vận hành phức tạp hơn — **đừng dùng StatefulSet khi Deployment + external state đủ**, và nghiêm túc cân nhắc managed database thay vì tự vận hành DB trong cluster (chi phí kỹ năng vận hành > chi phí managed service với đa số team).
- **DaemonSet**: mỗi node một pod — agent (log, monitoring, CNI).
- **Job/CronJob**: chạy xong thì thôi. Lưu ý CronJob: `concurrencyPolicy` (job trước chưa xong thì sao?) và idempotency — job có thể chạy lại.

---

## 4. Service, kube-proxy và Ingress — traffic tìm pod thế nào

### Problem Statement
Pod IP sinh diệt liên tục → cần một địa chỉ ổn định trỏ tới tập pod sống. Service = virtual IP (ClusterIP) + danh sách endpoint được cập nhật tự động theo label selector + readiness.

### Cách hoạt động bên trong — quan trọng để debug
ClusterIP **không tồn tại trên interface nào cả**. Nó là entry trong iptables/IPVS trên MỖI node, do kube-proxy lập trình: packet tới ClusterIP bị DNAT tại chỗ sang một pod IP ngẫu nhiên. Hệ quả:
- Không `ping` được ClusterIP (không phải máy thật) — đừng mất thời gian.
- Load balancing là **per-connection, L4, ngẫu nhiên** → bài toán gRPC/HTTP2 dồn 1 pod (module 01/02) áp dụng nguyên vẹn. Giải: headless service (`clusterIP: None` — DNS trả thẳng pod IPs, client tự LB) hoặc mesh/L7 proxy.
- Chuỗi debug service không tới nơi: (1) `kubectl get endpoints <svc>` — rỗng nghĩa là selector sai hoặc **readiness fail** (2 nguyên nhân của 90% ca "service không hoạt động"); (2) DNS: `nslookup <svc>` từ pod; (3) network policy chặn?; (4) kube-proxy/iptables trên node.

DNS nội bộ: `<svc>.<ns>.svc.cluster.local` (CoreDNS). Nhớ bẫy `ndots:5` từ module 01 — nó sống ở đây.

### Ingress / Gateway API
Service type LoadBalancer cấp 1 LB cloud **cho mỗi service** — đắt và loạn. Ingress = 1 điểm vào L7 dùng chung: 1 LB → ingress controller (nginx/Envoy chạy trong cluster) → route theo host/path về các Service, kiêm TLS termination (kết hợp cert-manager để tự động cert — bài học module 01). Gateway API là thế hệ kế nhiệm (chuẩn hơn, tách vai trò infra/app team) — dùng cho thiết kế mới.

---

## 5. Scheduler và Autoscaling

### Scheduler
Hai pha: **Filter** (loại node không đủ điều kiện: thiếu resource so với **requests**, taint không tolerate, nodeSelector/affinity không khớp, volume không attach được) → **Score** (chấm điểm node còn lại: rải đều, ưu tiên node ít tải, image có sẵn). Pod `Pending` = không node nào qua nổi Filter — `kubectl describe pod` đọc Events là ra đúng lý do, đây là kỹ năng debug cơ bản nhất.

Công cụ điều khiển đặt chỗ: nodeSelector/affinity (chọn loại node), **taint/toleration** (node đẩy pod ra — dành node GPU, spot), **topologySpreadConstraints / podAntiAffinity** (rải replica qua node/AZ — bắt buộc cho HA thật sự: 5 replica trên 1 node = HA giả), PodDisruptionBudget (giới hạn số pod bị rút cùng lúc khi drain node).

### Autoscaling — ba tầng độc lập phải phối hợp
1. **HPA** (số pod): scale theo metric (CPU trên **request**, hoặc custom metric — QPS, queue depth thường tốt hơn CPU). Bẫy: CPU limit throttling làm CPU% nhìn thấp → HPA không scale trong khi latency cháy. Có stabilization window chống flapping — nghĩa là scale **không tức thời**, phải chịu được burst trong lúc chờ.
2. **VPA** (kích cỡ pod): gợi ý/chỉnh request. Dùng chế độ recommend để right-size là giá trị nhất.
3. **Cluster Autoscaler / Karpenter** (số node): pod Pending vì thiếu chỗ → thêm node. Chuỗi trễ thực tế của một lần scale từ zero headroom: HPA quyết định (~15–30s) → pod Pending → node mới (cloud VM boot 1–3 phút) → image pull → app start. **Tổng có thể 3–5 phút** — traffic tăng nhanh hơn thế thì phải mua headroom (overprovisioning pause pods) hoặc scale chủ động trước theo lịch. Autoscaling không phải phép màu chống burst; nó là tối ưu chi phí cho sóng chậm.

---

## 6. Volume và dữ liệu

PersistentVolume (PV) = ổ đĩa thật; PersistentVolumeClaim (PVC) = yêu cầu của app; StorageClass = provisioner tự tạo PV (EBS, PD...). Điều quan trọng nhất phải hiểu: **block volume cloud bị buộc vào một AZ**. Pod dùng PVC ở AZ-a chỉ schedule được vào node AZ-a — node AZ-a hết chỗ là pod treo Pending mãi. Thiết kế HA cho stateful vì thế không phải "volume đa AZ" (không tồn tại với block storage) mà là **replication ở tầng ứng dụng** (Postgres streaming replication qua AZ, Kafka ISR...). `volumeBindingMode: WaitForFirstConsumer` để volume tạo đúng AZ nơi pod được xếp.

---

## 7. Vận hành production — những điều làm nên khác biệt

- **Resource hygiene**: mọi namespace có ResourceQuota + LimitRange; không pod nào thiếu requests (pod thiếu request = BestEffort = mồi eviction + phá bin packing).
- **Nâng cấp cluster**: Kubernetes ra bản mới ~3 lần/năm, chỉ hỗ trợ ~1 năm — **nâng cấp là việc thường xuyên, không phải dự án**. Đọc kỹ API deprecation (dùng `kubent`/pluto để quét), nâng control plane trước, node sau, một AZ/pool một lượt, PDB bảo vệ workload khi drain.
- **Chi phí**: right-size request (dữ liệu VPA), bin packing tốt, spot/preemptible cho stateless chịu chết (kết hợp taint + PDB + graceful shutdown), namespace-level showback. Lãng phí phổ biến nhất: request CPU gấp 5–10 lần thực dùng "cho chắc".
- **Đừng đưa vào cluster**: dữ liệu duy nhất một bản (etcd của bạn không phải chỗ đùa), workload cần kernel tuning đặc thù xung đột láng giềng, hoặc thứ mà managed service làm tốt hơn với chi phí kỹ sư thấp hơn.

### Bảng debug nhanh
| Triệu chứng | Lệnh | Nguyên nhân hay gặp |
|---|---|---|
| Pod Pending | `kubectl describe pod` (Events) | Thiếu resource, taint, PVC sai AZ |
| CrashLoopBackOff | `kubectl logs --previous` | App crash lúc boot, config/secret thiếu, liveness quá gắt |
| OOMKilled (exit 137) | `kubectl describe pod` Last State | Limit thấp / heap không khớp limit (module 03) |
| ImagePullBackOff | describe pod | Tag không tồn tại, thiếu imagePullSecret |
| Service không tới | `kubectl get endpoints` | Rỗng: selector sai hoặc readiness fail |
| Node NotReady | `kubectl describe node` | Kubelet chết, disk/memory pressure, mạng |
| Chậm sau deploy | `kubectl top pods`; cpu.stat trong pod | Throttling, thiếu warm-up, HPA chưa kịp |
