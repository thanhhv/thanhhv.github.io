+++
title = "Chương 20 — Kubernetes: nền tảng thực thi hợp đồng 12-Factor"
date = "2026-07-10T21:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> **Phần 3 – Engineering** | Chương trước: [Docker](/series/twelve-factor/19-docker/) | Chương sau: [CI/CD](/series/twelve-factor/21-cicd/)

Nếu 12-Factor là *hợp đồng* phía ứng dụng thì Kubernetes là *bên thi hành* phía nền tảng. Chương này đọc Kubernetes qua lăng kính đó: mỗi khái niệm K8s là câu trả lời cho một factor — và một app không theo 12-Factor sẽ vô hiệu hóa chính những khả năng bạn trả tiền để có.

---

## 1. Mô hình tư duy cốt lõi: declarative + reconciliation

Toàn bộ Kubernetes đứng trên một vòng lặp:

```
   Bạn KHAI BÁO trạng thái mong muốn          K8s LIÊN TỤC đối chiếu & điều chỉnh
   (YAML: "5 replica image X, mỗi cái   ──▶   controller: thực tế ≠ mong muốn?
    512Mi, probe thế này")                    → tạo/giết/dời pod cho đến khi khớp
                                              (reconciliation loop — mãi mãi)
```

Hệ quả quan trọng nhất cho người viết app: **không ai "chạy" pod của bạn cả — nó liên tục bị tạo, giết, thay, dời bởi một vòng lặp máy**. Đó chính là môi trường mà mọi factor (đặc biệt F6, F9) chuẩn bị cho app của bạn. App stateful/không-disposable trong môi trường này không "kém tối ưu" — nó **sai giả định nền tảng**.

Bảng ánh xạ hai chiều — đáng in ra dán tường:

| Kubernetes | Thi hành factor | App phải đáp lại bằng |
|---|---|---|
| Deployment + ReplicaSet | F8 (nhân bản), F5 (release) | Stateless (F6) |
| Service / EndpointSlice | F7 (routing tầng ngoài), F4 (discovery) | Bind PORT, không giữ địa chỉ peer |
| ConfigMap / Secret | F3 | Đọc env/file, không hardcode |
| Rolling update | F5 (release mới thay cũ) | F9 (SIGTERM êm, start nhanh) + probe đúng |
| HPA / KEDA | F8 (scale tự động) | Stateless + start nhanh |
| Liveness/Readiness probe | F9 | Endpoint health đúng ngữ nghĩa (chương 18) |
| Job / CronJob | F12 | Subcommand idempotent |
| DaemonSet log agent | F11 | stdout JSON |

## 2. Bộ manifest production chuẩn — có chú giải quyết định

Các mảnh đã xuất hiện rải rác từ chương 8–17; đây là bản hợp nhất với các quyết định *chưa* bàn kỹ:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-web
  labels: { app.kubernetes.io/name: myapp, app.kubernetes.io/component: web }
spec:
  replicas: 3                      # tối thiểu 2-3 vì HA — HPA sẽ điều chỉnh tiếp
  revisionHistoryLimit: 5
  selector:
    matchLabels: { app.kubernetes.io/name: myapp, app.kubernetes.io/component: web }
  strategy:
    type: RollingUpdate
    rollingUpdate: { maxUnavailable: 0, maxSurge: 25% }
  template:
    metadata:
      labels: { app.kubernetes.io/name: myapp, app.kubernetes.io/component: web }
    spec:
      terminationGracePeriodSeconds: 30
      # Trải pod ra nhiều node/zone — 3 replica cùng một node là HA giả:
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: topology.kubernetes.io/zone
          whenUnsatisfiable: ScheduleAnyway
          labelSelector:
            matchLabels: { app.kubernetes.io/name: myapp }
      securityContext:
        runAsNonRoot: true
        seccompProfile: { type: RuntimeDefault }
      containers:
        - name: web
          image: registry.example.com/myapp@sha256:4c2a...   # digest (F5)
          args: ["server"]
          ports: [{ name: http, containerPort: 8080 }]
          envFrom:
            - configMapRef: { name: myapp-config }
            - secretRef: { name: myapp-secrets }
          securityContext:
            readOnlyRootFilesystem: true
            allowPrivilegeEscalation: false
            capabilities: { drop: ["ALL"] }
          # ---- RESOURCES: quyết định quan trọng nhất và bị hiểu sai nhiều nhất ----
          resources:
            requests:              # dùng để XẾP LỊCH — cam kết tối thiểu
              cpu: 250m
              memory: 256Mi
            limits:                # trần cứng
              memory: 256Mi        # memory limit = request → QoS ổn định, OOM dự đoán được
              # CPU limit: cân nhắc BỎ — xem giải thích dưới
          readinessProbe:
            httpGet: { path: /readyz, port: http }
            periodSeconds: 5
            failureThreshold: 2
          livenessProbe:
            httpGet: { path: /healthz, port: http }
            initialDelaySeconds: 5
            periodSeconds: 10
            failureThreshold: 3
---
apiVersion: policy/v1
kind: PodDisruptionBudget           # node drain/upgrade không được giết quá nhiều pod
metadata: { name: myapp-web }
spec:
  minAvailable: 2
  selector:
    matchLabels: { app.kubernetes.io/name: myapp, app.kubernetes.io/component: web }
```

### Resources — CPU và memory hành xử khác hẳn nhau

- **Memory là tài nguyên không nén được**: vượt limit → **OOMKill** tức thì. Đặt `request = limit` cho memory để pod không bao giờ bị giết vì hàng xóm (QoS đoán được). Go: theo dõi heap thật (RSS) dưới tải, cộng ~30–50% dư; cân nhắc `GOMEMLIMIT` (Go ≥1.19) đặt ~90% limit để GC chủ động trước khi OOM.
- **CPU là tài nguyên nén được**: vượt limit → **throttle** (chạy chậm đi), không chết. CPU throttling gây tail latency kỳ quái khó debug; thực hành phổ biến hiện nay: **đặt CPU request đúng, bỏ CPU limit** (trên cluster của bạn; multi-tenant chặt chẽ thì khác). Nếu đặt limit, khớp `GOMAXPROCS` (chương 13).
- Không đặt requests = pod BestEffort: bị giết đầu tiên khi node căng, scheduler xếp mù — production không bao giờ để trống requests.

## 3. Service discovery — Factor 4 được nền tảng hóa

```
  App cần gọi service "orders" → chỉ cần biết tên DNS (từ config):
     http://orders.shop.svc.cluster.local

  Dưới nắp: Service (ClusterIP ổn định) → EndpointSlice (danh sách pod IP,
  cập nhật realtime khi pod đến/đi) → kube-proxy/eBPF route + load balance
```

Nghĩa là: hạ tầng lo *ai đang ở đâu* — app chỉ giữ *tên* trong config (F3+F4), không bao giờ giữ IP pod, không tự làm client-side discovery trong code (trừ khi có nhu cầu đặc biệt — gRPC client-side LB với headless service là ngoại lệ có chủ đích). Bổ sung `NetworkPolicy` để "ai được gọi ai" cũng là khai báo, không phải mặc định mở.

## 4. Autoscaling ba tầng

```
  HPA  — thêm/bớt POD theo metric (CPU, RPS, queue depth qua KEDA)   ← app-level
  VPA  — chỉnh request/limit của pod theo mức dùng thật              ← right-sizing
  CA/Karpenter — thêm/bớt NODE khi pod không còn chỗ xếp             ← hạ tầng
```

HPA đã có ở chương 13. Điểm phối hợp đáng nhớ: chuỗi phản ứng đầy đủ khi spike là *HPA tạo pod (giây) → hết chỗ → Karpenter thêm node (1–2 phút) → pod pending được xếp*. Thời gian đáp ứng tổng = thời gian dài nhất trong chuỗi — lại một lần nữa, app khởi động nhanh (F9) là mắt xích bạn kiểm soát được. VPA và HPA cùng chỉnh một tài nguyên (CPU) thì xung đột — thường dùng VPA ở chế độ khuyến nghị để right-size định kỳ.

## 5. Rolling update — giải phẫu một lần deploy zero-downtime

Ghép mọi mảnh đã học thành một dòng thời gian:

```
 t0  kubectl apply image mới (qua GitOps — chương 21)
 t1  K8s tạo pod mới (maxSurge) — pod cũ VẪN phục vụ
 t2  pod mới: start nhanh (F9) → readiness pass (ch18) → nhận traffic
 t3  K8s bắt đầu giết pod cũ: rút endpoint + SIGTERM (song song!)
 t4  pod cũ: fail readiness, drain 5s, xử lý nốt request, thoát (mẫu 3 giai đoạn ch14)
 t5  lặp cho đến khi 100% pod mới. User: không thấy gì. Đó là mục tiêu.
```

Mỗi mắt xích hỏng là một kiểu sự cố deploy: readiness hời hợt (pass khi chưa sẵn sàng) → 500 lúc t2; không bắt SIGTERM → 502 lúc t3–t4; start chậm → deploy kéo dài, autoscale tê liệt. **Zero-downtime không phải tính năng của K8s — nó là hợp đồng hai bên mà app phải giữ phần mình.**

## 6. Trade-off: có nên dùng Kubernetes không?

Câu hỏi principal thật sự không phải "K8s như thế nào" mà "có đáng không":

- **Chi phí thật của K8s**: độ dốc học tập cho cả team, vận hành control plane (managed đỡ nhiều nhưng không miễn phí kỹ năng), hệ sinh thái phụ trợ gần như bắt buộc (ingress, cert, observability, GitOps...) — "thuế nền tảng" dễ chiếm 1–2 kỹ sư full-time.
- **Khi nào xứng đáng**: nhiều service, nhiều team, nhu cầu chuẩn hóa deploy/scale/self-healing trên fleet lớn, hoặc multi-cloud/on-prem có thật.
- **Khi nào chưa**: một vài service, team nhỏ → PaaS (Cloud Run, App Runner, Fly.io) cho 80% lợi ích với 5% chi phí vận hành. **App viết đúng 12-Factor chạy tốt trên cả hai — đó chính là giá trị chiến lược của việc tách "cách viết app" khỏi "nền tảng chạy app"**: bạn đổi nền tảng sau này mà không viết lại app.

Trade-off nội bộ K8s đáng lưu ý: **managed (EKS/GKE/AKS) vs tự vận hành** — tự vận hành control plane chỉ hợp lý khi có ràng buộc on-prem/chủ quyền; **StatefulSet cho database trong cluster vs managed DB ngoài cluster** — mặc định đúng cho đa số team là DB managed ngoài cluster, để cluster chỉ chứa thứ disposable (đúng vùng an toàn của nó).

## 7. Anti-patterns

- **`kubectl edit/apply` tay lên production** — quay lại snowflake, lần này là snowflake YAML; mọi thay đổi qua Git (chương 21).
- **Không đặt resources** hoặc copy đại `cpu: "1"` cho mọi service — xếp lịch mù, OOM ngẫu nhiên, lãng phí cluster.
- **Liveness probe check DB** — tự sát tập thể (chương 18, nhắc lần ba vì gặp lần thứ n trên thực tế).
- **1 replica cho service production** — mọi rolling update/node drain là downtime; "có K8s" không tự nhiên có HA.
- **Chạy database quan trọng bằng Deployment + PVC "tạm thôi"** — sai công cụ (tối thiểu là StatefulSet + operator, hoặc managed).
- **Bỏ PodDisruptionBudget** — một lần node pool upgrade giết đủ số pod để thành sự cố.
- **Latest + imagePullPolicy: Always làm cơ chế deploy** — không ai biết đang chạy gì; digest hoặc về nhà.
- **Nhét mọi môi trường vào một namespace/cluster không ranh giới** — quyền, quota, network policy theo namespace là tối thiểu; prod tách cluster khi đủ lớn.

## 8. Khi nào KHÔNG cần Kubernetes

Đã trả lời ở mục 6 — chốt bằng thang quyết định thực dụng: 1–3 service → PaaS; ~3–10 service, team có kỹ năng → managed K8s *hoặc* PaaS tốt; nhiều team + fleet lớn + yêu cầu chuẩn hóa → K8s gần như là mặc định của ngành, và lúc đó đầu tư platform engineering (chương 23) để trả thuế nền tảng một lần cho mọi team.

---

## Tóm tắt

- K8s = vòng lặp **declarative reconciliation**: pod bị tạo/giết/dời liên tục — chính là môi trường mà 12-Factor giả định; app giữ phần hợp đồng của mình thì K8s mới thi hành được phần của nó.
- Manifest production tối thiểu: digest image, envFrom ConfigMap/Secret, đủ hai probe đúng ngữ nghĩa, resources có suy nghĩ (memory req=limit, cân nhắc bỏ CPU limit), PDB + topology spread, security context khóa chặt.
- Zero-downtime deploy là dây chuyền: start nhanh → readiness trung thực → SIGTERM êm — đứt mắt nào lộ mắt đó.
- K8s không phải đích đến; app 12-Factor chạy tốt từ PaaS đến K8s — chọn nền tảng theo quy mô tổ chức, không theo trend.
