+++
title = "Chương 13 — Factor 8: Concurrency"
date = "2026-07-10T14:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> *"Scale out via the process model"* — Scale bằng cách nhân bản process.
>
> **Phần 2 – The Twelve Factors** | Nhóm: **Disposable & Scalable** | Chương trước: [Port Binding](/series/twelve-factor/12-factor-07-port-binding/) | Chương sau: [Disposability](/series/twelve-factor/14-factor-09-disposability/)

---

## 1. Problem Statement

Câu hỏi: **"Khi tải tăng, hệ thống lớn lên bằng cách nào?"**

Hai con đường:

```
   SCALE UP (dọc)                        SCALE OUT (ngang)
  ────────────────                      ─────────────────
   Máy to hơn: thêm CPU/RAM              Nhiều bản sao hơn của process
   + Không đổi kiến trúc                 + Không có trần (thêm máy là được)
   + Đơn giản                            + Chi tiết hóa: scale đúng phần cần
   − Có TRẦN vật lý                      + Kèm luôn HA (một bản chết còn bản khác)
   − Giá phi tuyến (máy 2x đắt >2x)      − ĐÒI HỎI app đúng kiểu: stateless (F6),
   − Downtime khi nâng cấp                 port binding (F7), disposable (F9)
   − Một máy chết = chết hết             − Phức tạp hơn: LB, orchestrator
```

Factor 8 chọn scale out, và chỉ rõ **đơn vị scale là process** (ngày nay: container/pod — một process trong một container). Không có gì tự nhiên ở lựa chọn này — nó là hệ quả: đã stateless (F6) thì nhân bản là an toàn; đã bind port (F7) thì nhân bản là *lắp thêm vào load balancer*; và ngược lại, chọn scale out thì *bắt buộc* F6, F7. Các factor khóa lẫn nhau.

Vế thứ hai của factor: **phân loại workload theo process type**. Web request, background job, scheduled task có đặc tính tải khác hẳn nhau → tách thành các loại process riêng (web, worker, scheduler), scale **độc lập** từng loại.

## 2. Tại sao nguyên lý này tồn tại

- **Scalability**: trần của scale up là trần vật lý của một máy; trần của scale out là ngân sách. Hệ thống Internet-scale chỉ có một đường.
- **Reliability**: N bản sao = redundancy miễn phí kèm theo. Scale up N lần vẫn là single point of failure.
- **Operational/Cost**: tải web tăng 10x lúc campaign chạy nhưng tải worker chỉ tăng 2x — tách process type cho phép trả tiền đúng chỗ cần. Gộp tất cả vào một process là scale cả cục vì một phần nóng.
- **Deployment**: rolling update (F5/F9) về bản chất *là* một thao tác scale out/in: thêm dần bản mới, bớt dần bản cũ. Không scale out được thì cũng không zero-downtime deploy được.

## 3. Bản chất — và bản cập nhật 2026 cho Go

### 3.1. Share-nothing horizontal scaling

Mô hình concurrency của 12-Factor là **process share-nothing xếp cạnh nhau**: các bản sao không biết nhau, không phối hợp trực tiếp, chỉ gặp nhau ở backing services. Điều phối (bản nào nhận request nào) là việc của load balancer; đồng bộ (khóa, đếm, hàng đợi) là việc của backing service (Redis, DB, queue). Chính vì không chia sẻ gì, việc thêm bản sao thứ N+1 có chi phí điều phối gần bằng 0 — đó là lý do mô hình này scale gần tuyến tính.

### 3.2. Hai tầng concurrency — trong process và giữa process

Tài liệu gốc 2011 viết trong bối cảnh Ruby/Python, nơi một process phục vụ kém hiệu quả nhiều việc song song → cần nhiều process ngay cả trên một máy. **Go thay đổi cục diện tầng trong-process**: goroutine + scheduler M:N cho phép *một* process Go khai thác toàn bộ core của máy và phục vụ hàng trăm nghìn kết nối đồng thời.

```
  Tầng 1 — TRONG process (Go lo):          Tầng 2 — GIỮA các process (nền tảng lo):
  goroutine per request (net/http           nhân bản pod theo tải
  làm sẵn), GOMAXPROCS dùng hết core        (HPA — xem 4.3)

  → Go: KHÔNG cần nhiều process             → VẪN CẦN nhiều process/pod, vì:
    trên một máy để tận dụng máy              • vượt giới hạn MỘT MÁY
                                              • high availability
                                              • rolling update
                                              • cô lập sự cố (1 pod OOM ≠ chết hết)
```

Kết luận cập nhật: với Go, ý nghĩa của Factor 8 **không phải** "chạy nhiều process để tận dụng CPU" (Go tự làm) mà là: **thiết kế để nhân bản pod là cách scale duy nhất cần nghĩ đến, và tách process type để scale độc lập**. Số replica tối thiểu cho production: 2–3 (vì HA, không phải vì tải).

**Điều gì xảy ra nếu vi phạm?** Dồn mọi workload vào một process → phần nóng nhất quyết định tài nguyên của tất cả; job nền nặng làm nghẹt web request cùng process (đói CPU, GC pressure); và mỗi lần deploy web là restart cả worker đang xử lý job dở. Không scale out được → quay về mọi giới hạn của scale up.

## 4. Cách áp dụng

### 4.1. Process type trong Go — nhiều entrypoint, một codebase

```
cmd/
├── server/main.go     # process type "web": nhận HTTP, trả lời nhanh, KHÔNG làm việc nặng
├── worker/main.go     # process type "worker": tiêu thụ queue, việc nặng/lâu
└── scheduler/         # (nếu cần) — thường thay bằng K8s CronJob (F12/chương 17)
```

```go
// cmd/worker/main.go — worker tiêu thụ queue, ví dụ với Redis Streams
package main

func main() {
	cfg := config.MustLoad()
	logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))
	rdb := redis.NewClient(mustParseURL(cfg.RedisURL))

	ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGTERM, syscall.SIGINT)
	defer stop()

	// Concurrency TRONG process: pool goroutine có giới hạn.
	// Giới hạn quan trọng hơn tốc độ: worker không được nuốt tài nguyên vô hạn.
	sem := make(chan struct{}, cfg.WorkerConcurrency)
	var wg sync.WaitGroup

	for {
		select {
		case <-ctx.Done():
			wg.Wait() // đợi job dở xong rồi mới thoát (F9)
			logger.Info("worker stopped cleanly")
			return
		default:
		}

		msgs, err := readBatch(ctx, rdb, "jobs", "workers", cfg.HostID) // XREADGROUP
		if err != nil {
			logger.Error("read queue", "err", err)
			time.Sleep(time.Second)
			continue
		}
		for _, m := range msgs {
			sem <- struct{}{}
			wg.Add(1)
			go func(m Message) {
				defer func() { <-sem; wg.Done() }()
				// Idempotent: message có thể được giao lại (at-least-once).
				// Consumer group đảm bảo mỗi message chỉ một worker nhận —
				// scale worker = tăng replicas, KHÔNG cần code thêm gì.
				if err := process(ctx, m); err != nil {
					logger.Error("process", "msg_id", m.ID, "err", err)
					return // không ACK → giao lại sau
				}
				ack(ctx, rdb, m)
			}(m)
		}
	}
}
```

Điểm kiến trúc then chốt: **queue với consumer group là cơ chế phân phối tải giữa các worker** — thêm worker replica là thêm throughput, không cần điều phối gì thêm. Với web, load balancer đóng vai trò đó. Cả hai đều là "điều phối ở tầng ngoài".

### 4.2. Kubernetes — mỗi process type một Deployment, scale độc lập

```yaml
# web: nhiều replica, tải theo request
apiVersion: apps/v1
kind: Deployment
metadata: { name: myapp-web }
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: web
          image: registry.example.com/myapp@sha256:...   # CÙNG image
          args: []                                        # entrypoint mặc định = server
          resources:
            requests: { cpu: 250m, memory: 128Mi }
            limits: { memory: 256Mi }   # limit CPU thường bỏ (tránh throttle) — chương 20
---
# worker: ít replica hơn, tải theo độ sâu queue
apiVersion: apps/v1
kind: Deployment
metadata: { name: myapp-worker }
spec:
  replicas: 2
  template:
    spec:
      containers:
        - name: worker
          image: registry.example.com/myapp@sha256:...   # cùng image, khác lệnh
          command: ["/worker"]
```

### 4.3. Autoscaling — nền tảng nhân bản process thay bạn

```yaml
# HPA cho web: scale theo CPU (đơn giản nhất; theo RPS/latency cần custom metrics)
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata: { name: myapp-web }
spec:
  scaleTargetRef: { apiVersion: apps/v1, kind: Deployment, name: myapp-web }
  minReplicas: 3
  maxReplicas: 30
  metrics:
    - type: Resource
      resource:
        name: cpu
        target: { type: Utilization, averageUtilization: 70 }
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300   # scale xuống từ tốn — tránh flapping
```

Worker nên scale theo **độ sâu queue** thay vì CPU (queue dài mà CPU thấp vẫn cần thêm worker) — công cụ chuẩn là **KEDA** (ScaledObject đọc lag của Kafka/Redis/RabbitMQ). Chi tiết chương 20.

Lưu ý Go-đặc-thù khi chạy trong container: nếu đặt CPU limit, hãy set `GOMAXPROCS` khớp với limit (Go ≥1.25 tự nhận biết cgroup; các bản trước dùng `automaxprocs` của Uber) — không thì Go tưởng có 32 core của node, tạo thừa thread và bị throttle.

## 5. Trade-off

**Scale out vs scale up — không phải tôn giáo.** Chi phí thật của scale out nằm ở backing service: 30 pod × 20 connection = 600 connection vào Postgres (cần PgBouncer); cache local nhân 30 bản; N pod cùng cold start dội vào DB. Với tải vừa phải, 2–3 instance cỡ vừa (vẫn HA) thắng 30 instance tí hon về mọi mặt trừ "độ mịn" của autoscaling. Chọn cỡ instance × số lượng là bài toán tối ưu hai biến, không phải "càng nhiều pod càng cloud-native".

**Tách process type vs vận hành đơn giản.** Mỗi process type thêm một Deployment, một dashboard, một chuỗi alert. App nhỏ có thể chạy web + worker trong một process (goroutine) — chấp nhận được ở quy mô nhỏ **nếu** job ngắn và bạn hiểu cái giá khi deploy/scale. Ngưỡng tách: job chạy > vài giây, tải hai loại lệch nhau rõ, hoặc job không được phép chết theo deploy của web.

**Autoscaling vs capacity cố định.** Autoscaling tiết kiệm khi tải dao động mạnh; với tải phẳng, nó chỉ thêm rủi ro (flapping, scale trễ trước spike đột ngột). Spike giây (flash sale mở màn) autoscaler không kịp phản ứng — cần pre-scale theo lịch hoặc overprovision buffer.

## 6. Best Practices

- Web trả lời nhanh, đẩy việc nặng sang worker qua queue; ranh giới ~100ms xử lý là gợi ý tách.
- Worker idempotent + at-least-once; concurrency trong worker có giới hạn (semaphore/pool).
- `minReplicas ≥ 2` cho mọi thứ production (HA); PodDisruptionBudget đi kèm.
- HPA cho web (CPU/RPS), KEDA cho worker (queue depth); scale-down có stabilization window.
- Tính toán tổng connection vào backing service khi scale: `maxReplicas × pool size` phải nằm trong giới hạn DB.
- `GOMAXPROCS` khớp CPU limit (Go ≥ 1.25 tự động; trước đó dùng automaxprocs).
- Cùng image cho mọi process type, khác `command` — một artifact, nhiều vai (F5 + F8).

## 7. Anti-patterns

- **Job nền nặng chạy goroutine trong process web** — nghẹt lẫn nhau, chết theo deploy; đã mổ xẻ ở F6.
- **Scale bằng tay** (`kubectl scale` mỗi khi chậm) — con người là autoscaler tồi: trễ, quên scale xuống, không có lúc 3h sáng.
- **Mọi process type chung một Deployment** ("container to chạy đủ thứ") — mất scale độc lập, mất cô lập sự cố; kèm anti-pattern nhiều process một container (chương 19).
- **Worker không giới hạn concurrency** — queue dồn 100k message, worker nuốt hết vào memory → OOM, message giao lại, OOM tiếp: crash loop bằng chính dữ liệu của mình.
- **Scale out app nhưng quên backing service** — 50 pod đè chết Postgres `max_connections=100`; sự cố "scale lên làm hệ thống chậm đi" nghe nghịch lý nhưng cực kỳ phổ biến.
- **Dựa vào số lượng pod để "đảm bảo" ordering hoặc singleton** — hai giả định bị rolling update phá vỡ ngay (lúc update luôn có N+1 pod). Singleton cần lease/lock; ordering cần partition key trong queue.

## 8. Khi nào KHÔNG áp dụng

- **Tải nhỏ và ổn định**: 2 replica cố định (vì HA) + không autoscaling là kiến trúc hoàn toàn trưởng thành. Đừng dựng KEDA cho 10 job/ngày.
- **Workload cần state cục bộ lớn** (ML model vài chục GB trong RAM): nhân bản đắt; scale up + ít replica lớn hợp lý hơn.
- **Hệ tuần tự bản chất** (xử lý ledger đòi hỏi thứ tự toàn cục): song song hóa phá tính đúng — scale bằng partition theo key, trong mỗi partition vẫn tuần tự.
- **Desktop/CLI/embedded**: không có khái niệm nhân bản.

---

## Tóm tắt

- Scale out qua process model: **nhân bản là cách lớn lên duy nhất cần nghĩ đến** — khả thi vì F6 (stateless) + F7 (port binding), và là điều kiện cho F9 (rolling, recovery).
- Với Go, tầng trong-process đã được goroutine giải quyết; giá trị của F8 nằm ở **tách process type (web/worker) và scale độc lập từng loại** — cùng image, khác command.
- Điều phối thuộc tầng ngoài: LB cho web, queue + consumer group cho worker, HPA/KEDA cho số lượng.
- Cạm bẫy lớn nhất: scale app mà quên sức chịu của backing service, và job nền sống chui trong process web.
