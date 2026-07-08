+++
title = "Chương 14 — Factor 9: Disposability"
date = "2026-07-10T15:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> *"Maximize robustness with fast startup and graceful shutdown"* — Tối đa hóa độ bền vững bằng khởi động nhanh và tắt êm.
>
> **Phần 2 – The Twelve Factors** | Nhóm: **Disposable & Scalable** | Chương trước: [Concurrency](/series/twelve-factor/13-factor-08-concurrency/) | Chương sau: [Dev/Prod Parity](/series/twelve-factor/15-factor-10-dev-prod-parity/)

---

## 1. Problem Statement

Chương 2 đã xác lập: trên hạ tầng hiện đại, **process của bạn sẽ bị giết — thường xuyên, và phần lớn là có chủ đích**. Hãy đếm các sự kiện giết process trong một tuần vận hành K8s bình thường:

- Rolling update: mỗi lần deploy, **mọi** pod cũ bị giết.
- Autoscale down: bớt tải là bớt pod.
- Node drain: nâng cấp node, spot instance bị thu hồi.
- Rebalance: scheduler dời pod sang node khác.
- OOM kill, liveness probe fail: giết để cứu.

Deploy 5 lần/ngày × 10 pod = process bị giết ~50 lần/ngày **chỉ riêng do deploy**. Câu hỏi của factor này: **"Việc giết và tạo process có phải là non-event không?"** Nếu không:

- Giết không êm → **rơi request đang xử lý** (user thấy lỗi 502 mỗi lần deploy), job dở dang, connection không đóng, dữ liệu nửa vời.
- Khởi động chậm → autoscale phản ứng trễ (spike đến, pod mới 2 phút sau mới nhận tải — quá muộn), recovery chậm, rolling update một fleet lớn kéo dài hàng giờ.

## 2. Tại sao nguyên lý này tồn tại

- **Deployment**: zero-downtime deploy về bản chất = "giết bản cũ êm + khởi động bản mới nhanh" lặp N lần. Disposability *là* zero-downtime deploy.
- **Scalability**: giá trị của autoscaling tỷ lệ nghịch với thời gian khởi động. Pod lên trong 5 giây → autoscale theo được spike; 3 phút → autoscale chỉ còn là trang trí.
- **Operational**: khi giết/tạo là non-event, mọi thao tác vận hành (drain node, dời pod, restart để xóa nghi vấn) trở nên **không cần xin phép, không cần cửa sổ bảo trì** — chi phí vận hành giảm cấp số nhân.
- **Reliability**: crash-only design — hệ thống chịu được cả cái chết *không* báo trước (kill -9, mất điện) thì cái chết *có* báo trước chỉ là trường hợp dễ.

## 3. Bản chất

### 3.1. Hợp đồng vòng đời với nền tảng

Disposability là mặt "vòng đời" của hợp đồng app–nền tảng. Trên Kubernetes, hợp đồng đó rất cụ thể — và đây là kiến thức bắt buộc của mọi backend engineer làm việc với K8s:

```
 KHỞI ĐỘNG                                  TẮT
 ─────────                                  ───
 1. Container start                         1. Pod bị đánh dấu Terminating
 2. App init: config, connect resources     2. ĐỒNG THỜI xảy ra:
 3. Bind port, sẵn sàng                        a. Pod bị RÚT khỏi Service endpoints
 4. readinessProbe pass                           (ngừng nhận request MỚI — nhưng
    → nền tảng bắt đầu đưa traffic vào             mất vài giây để lan ra mọi node!)
                                               b. Container nhận SIGTERM
 Yêu cầu: bước 2–4 tính bằng GIÂY           3. App: dừng nhận request mới, XỬ LÝ NỐT
                                               request dở, đóng connection, flush
 (Go: binary tĩnh, không JIT warm-up,       4. App thoát (exit 0)
  thường < 1–2s — lợi thế lớn                5. Nếu quá terminationGracePeriodSeconds
  của Go ở factor này)                          (mặc định 30s) → SIGKILL: chết cứng,
                                                không thương lượng
```

Hai điểm tinh tế gây ra 90% lỗi "deploy vẫn rơi request":

1. **SIGTERM đến trước khi traffic ngừng hẳn.** Việc rút endpoint (2a) và gửi SIGTERM (2b) chạy **song song**; kube-proxy/ingress cần vài giây để cập nhật. App nhận SIGTERM mà đóng listener *ngay lập tức* sẽ từ chối những request vẫn đang được route tới → 502. Giải pháp chuẩn: **nhận SIGTERM → chờ vài giây (hoặc fail readiness) → mới bắt đầu shutdown**.
2. **Graceful shutdown phải có deadline.** Chờ request dở mãi mãi = treo termination = ăn SIGKILL giữa chừng còn tệ hơn. Deadline app < `terminationGracePeriodSeconds`.

### 3.2. Crash-only thinking

Mặt còn lại của disposability: **thiết kế để chết đột tử cũng không hỏng dữ liệu**, vì SIGKILL, OOM, mất node là chuyện *sẽ* xảy ra. Nguyên tắc: graceful shutdown là *tối ưu trải nghiệm* (không rơi request), còn *tính đúng đắn* phải dựa vào những thứ chịu được chết cứng — transaction (atomic), queue với ACK sau khi xử lý xong (at-least-once + idempotent consumer), write-ahead log. Nếu tính đúng đắn của bạn *phụ thuộc* vào việc shutdown êm ái, thiết kế đang sai.

## 4. Cách áp dụng với Go — mẫu graceful shutdown production đầy đủ

```go
// cmd/server/main.go — mẫu hoàn chỉnh: HTTP server + worker con + K8s lifecycle
package main

import (
	"context"
	"errors"
	"log/slog"
	"net/http"
	"os"
	"os/signal"
	"sync/atomic"
	"syscall"
	"time"
)

func main() {
	logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))
	cfg := config.MustLoad()

	// --- Khởi động nhanh: chỉ làm việc THIẾT YẾU trước khi ready ---
	pool := mustConnectDB(cfg)      // thiết yếu → trước ready
	defer pool.Close()
	// Việc không thiết yếu (warm cache...) → goroutine sau khi ready

	// ready flag: readiness probe đọc cờ này
	var ready atomic.Bool

	mux := http.NewServeMux()
	mux.HandleFunc("GET /healthz", func(w http.ResponseWriter, r *http.Request) {
		w.WriteHeader(http.StatusOK) // liveness: "process còn sống" — luôn OK trừ khi kẹt chết
	})
	mux.HandleFunc("GET /readyz", func(w http.ResponseWriter, r *http.Request) {
		if !ready.Load() {
			http.Error(w, "not ready", http.StatusServiceUnavailable)
			return
		}
		w.WriteHeader(http.StatusOK)
	})
	mux.Handle("/api/", apiHandler(pool))

	srv := &http.Server{
		Addr:              ":" + cfg.Port,
		Handler:           mux,
		ReadHeaderTimeout: 5 * time.Second,
	}

	errCh := make(chan error, 1)
	go func() {
		if err := srv.ListenAndServe(); err != nil && !errors.Is(err, http.ErrServerClosed) {
			errCh <- err
		}
	}()

	ready.Store(true)
	logger.Info("started", "port", cfg.Port, "startup", "fast") // Go: thường <1s

	// --- Đợi tín hiệu chết ---
	ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGTERM, syscall.SIGINT)
	defer stop()

	select {
	case err := <-errCh:
		logger.Error("server failed", "err", err)
		os.Exit(1)
	case <-ctx.Done():
	}

	// --- Giai đoạn 1: ngừng NHẬN việc mới, nhưng CHƯA đóng gì ---
	logger.Info("SIGTERM received, draining")
	ready.Store(false) // readiness fail → nền tảng ngừng route (phòng bị kép)

	// Chờ endpoint removal lan tỏa: SIGTERM đến song song với việc rút khỏi
	// Service — đóng ngay là rơi các request vẫn đang bay tới
	time.Sleep(cfg.DrainDelay) // 3–5s là điểm cân bằng phổ biến

	// --- Giai đoạn 2: xử lý nốt request dở, có deadline ---
	shutdownCtx, cancel := context.WithTimeout(context.Background(),
		cfg.ShutdownTimeout) // PHẢI < terminationGracePeriodSeconds
	defer cancel()
	if err := srv.Shutdown(shutdownCtx); err != nil {
		logger.Warn("some requests were cut off", "err", err)
	}

	// --- Giai đoạn 3: dọn dẹp resource (defer pool.Close() chạy ở đây) ---
	logger.Info("stopped cleanly")
}
```

Với **worker** (queue consumer), mẫu tương tự nhưng "request dở" là message đang xử lý: nhận SIGTERM → ngừng lấy message mới → đợi message đang xử lý xong (có deadline) → không ACK những message chưa xong (chúng sẽ được giao lại — đây là lý do consumer phải idempotent).

### Kubernetes manifest khớp với code

```yaml
# deployment.yaml (trích) — các con số phải KHỚP với code
spec:
  template:
    spec:
      terminationGracePeriodSeconds: 30   # > DrainDelay(5s) + ShutdownTimeout(15s) + dư
      containers:
        - name: server
          image: registry.example.com/myapp@sha256:...
          readinessProbe:
            httpGet: { path: /readyz, port: 8080 }
            periodSeconds: 5
            failureThreshold: 2
          livenessProbe:
            httpGet: { path: /healthz, port: 8080 }
            initialDelaySeconds: 5
            periodSeconds: 10
            failureThreshold: 3
  strategy:
    rollingUpdate: { maxUnavailable: 0, maxSurge: 1 }  # chỉ giết pod cũ khi pod mới ready
```

Bộ ba số phải nhất quán: `DrainDelay + ShutdownTimeout + buffer < terminationGracePeriodSeconds`. Lệch là SIGKILL cắt ngang shutdown — và lỗi này chỉ lộ dưới tải thật.

## 5. Trade-off

**Drain delay: mất vài giây mỗi lần tắt vs rơi request.** `sleep 5s` trước shutdown làm mọi termination chậm đi 5 giây — với CronJob chạy 10k pod/ngày, đó là chi phí thật. Web-facing: luôn đáng. Worker nội bộ không nhận HTTP: bỏ drain delay, chỉ cần dừng-lấy-việc-mới.

**Graceful period dài vs ngắn.** Dài (5 phút): chiều được request/job dài, nhưng node drain chậm, rolling update ì ạch, spot instance không kịp (chỉ có 30–120s). Ngắn (15–30s): vận hành nhanh, nhưng ép kỷ luật "không có việc gì dài hơn 15s trong process này" — việc dài hơn phải thành job chia nhỏ + checkpoint. Khuyến nghị: giữ ngắn, coi nhu cầu period dài là **triệu chứng thiết kế** (việc dài đang ở sai chỗ).

**Khởi động nhanh vs warm-up đầy đủ.** Ready sớm khi cache nguội → latency cao ban đầu; ready muộn → autoscale trễ. Điểm cân bằng: ready khi *đúng đắn* (đủ điều kiện phục vụ chính xác), warm-up *hiệu năng* chạy nền sau đó. Go gần như miễn nhiễm vấn đề JIT warm-up của JVM — một lý do Go phổ biến trong hạ tầng cloud-native.

## 6. Best Practices

- Mẫu 3 giai đoạn: **drain (fail readiness + delay) → shutdown có deadline → cleanup**; số liệu khớp `terminationGracePeriodSeconds`.
- `maxUnavailable: 0` cho service quan trọng; PodDisruptionBudget để node drain không giết quá nhiều pod cùng lúc.
- Khởi động: chỉ việc thiết yếu trước ready; đo và alert thời gian startup (tăng dần là mùi hôi kiến trúc).
- Crash-safety là nền: transaction, ACK-sau-xử-lý, idempotency — graceful chỉ là tối ưu.
- Test cả hai đường chết: `kill -TERM` (êm) và `kill -9` (cứng) trong integration test; kiểm tra không rơi request bằng load test trong lúc rolling update (một bài test đáng giá hơn mọi lý thuyết).
- Đừng bắt `SIGKILL` (không bắt được) và đừng quên: trong container, app của bạn là PID 1 — đảm bảo binary tự xử lý signal (Go: ổn) hoặc dùng tini nếu entrypoint là shell script.

## 7. Anti-patterns

- **Không bắt SIGTERM** — mặc định Go process chết ngay lập tức: mọi request đang bay bị cắt. Anti-pattern phổ biến số một, vì local không bao giờ lộ.
- **Đóng listener ngay khi nhận SIGTERM** — quên khoảng trễ lan tỏa endpoint: vẫn 502 dù "đã có graceful shutdown". Phổ biến số hai.
- **Shutdown không deadline** — chờ vô hạn một request treo → SIGKILL → tệ hơn cả không graceful.
- **ENTRYPOINT dạng shell** (`sh -c "./app"`) — shell là PID 1 và **không chuyển tiếp SIGTERM**: app không bao giờ biết mình sắp chết. Dùng exec form `["/app"]`.
- **Startup làm việc nặng** (migration trong main — đã bàn ở F5, warm toàn bộ cache, retry connect 10 phút) — pod treo ở Pending-Ready, autoscale vô dụng, rolling update dài lê thê.
- **Tính đúng đắn dựa vào shutdown êm** (chỉ flush buffer khi tắt đẹp) — OOM kill sẽ dạy bạn bài học này bằng dữ liệu thật.
- **Liveness probe trỏ vào dependency** (check DB trong `/healthz`) — DB chết → K8s restart *app* hàng loạt vô ích, biến sự cố một tầng thành sự cố hai tầng (chi tiết chương 18).

## 8. Khi nào KHÔNG cần áp dụng đầy đủ

- **Batch job có checkpoint tốt**: chết giữa chừng và chạy lại từ checkpoint là mô hình đúng; SIGTERM chỉ cần "lưu checkpoint rồi thoát", thậm chí không cần nếu checkpoint theo nhịp.
- **CLI tool**: đời sống tính bằng giây, người dùng là kẻ duy nhất gửi signal.
- **Hệ thống một instance ít deploy**: giá trị của disposability tỷ lệ với tần suất giết/tạo process — deploy mỗi quý thì 5 giây downtime mỗi lần không đáng đầu tư phức tạp (nhưng bắt SIGTERM cơ bản vẫn nên: 10 dòng code).
- Lưu ý ngược: **hệ stateful** (DB) không "miễn" factor này — chúng cần disposability *hơn ai hết* (failover êm), chỉ là cơ chế phức tạp hơn nhiều (leader election, replication) và thường được đóng gói sẵn trong operator.

---

## Tóm tắt

- Process **sẽ** bị giết hàng chục lần mỗi ngày một cách bình thường — disposability biến điều đó thành non-event; nó chính là nền của zero-downtime deploy và autoscaling có ý nghĩa.
- Hợp đồng tắt trên K8s: SIGTERM đến **song song** với việc rút traffic → mẫu 3 giai đoạn: drain (fail readiness + chờ vài giây) → `srv.Shutdown` có deadline → cleanup; deadline khớp `terminationGracePeriodSeconds`.
- Khởi động nhanh = chỉ việc thiết yếu trước ready; Go có lợi thế tự nhiên (binary tĩnh, không warm-up JIT).
- Graceful là tối ưu; **crash-safety (transaction, ACK-sau-xử-lý, idempotency) mới là nền của tính đúng đắn** — thiết kế cho kill -9, hưởng thụ kill -TERM.
