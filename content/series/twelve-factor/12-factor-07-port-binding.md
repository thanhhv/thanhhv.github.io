+++
title = "Chương 12 — Factor 7: Port Binding"
date = "2026-07-10T13:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> *"Export services via port binding"* — Xuất khẩu dịch vụ qua việc bind port.
>
> **Phần 2 – The Twelve Factors** | Nhóm: **Disposable & Scalable** | Chương trước: [Processes](/series/twelve-factor/11-factor-06-processes/) | Chương sau: [Concurrency](/series/twelve-factor/13-factor-08-concurrency/)

---

## 1. Problem Statement

Factor này khó cảm nhận với dev Go — vì Go **mặc định đã đúng**. Để hiểu nó giải quyết gì, phải nhìn về mô hình mà nó phủ định: **ứng dụng sống ký sinh trong một app server**.

Thế giới cũ (Java/PHP điển hình):

```
   MÔ HÌNH KÝ SINH (cũ)                MÔ HÌNH TỰ CHỦ (12-Factor)
  ─────────────────────────           ─────────────────────────
   Tomcat/JBoss/Apache+mod_php         Process app CỦA BẠN
   ├── cài đặt, cấu hình riêng         └── nhúng HTTP server
   ├── app.war deploy VÀO trong            (net/http của Go)
   │   container đó                        bind vào PORT từ env
   └── app KHÔNG tự chạy được —            → app là process tự chủ,
       nó là "khách trọ" của server          tự phục vụ qua port
```

Vấn đề của mô hình ký sinh:

- **App không tự mô tả được cách chạy mình.** Chạy app = cài + cấu hình app server trước (một dependency ngầm khổng lồ, vi phạm F2). "App của tôi cần Tomcat 8.5.x, config JNDI thế này, JVM flags thế kia" — tri thức nằm ngoài codebase.
- **Runtime dùng chung.** Nhiều app trong một Tomcat: chung heap, chung GC, chung vòng đời — một app rò rỉ memory giết tất cả; restart server là restart mọi app; không scale riêng từng app được.
- **Không đồng nhất giữa các loại app.** App HTTP chạy trong Tomcat, app khác chạy kiểu khác → nền tảng tự động hóa phải biết N cách chạy N loại app.

Port binding chuẩn hóa tất cả về một hợp đồng tối giản: **app là một process tự chứa, lắng nghe trên một port được cấp qua config; ai muốn nói chuyện với app, gửi request tới port đó.** Nền tảng không cần biết gì thêm.

## 2. Tại sao nguyên lý này tồn tại

- **Deployment**: nền tảng tự động (Heroku dyno, K8s kubelet) chỉ cần làm đúng một việc cho *mọi* app: chạy process, cấp `PORT`, đợi app nghe. Hợp đồng càng nhỏ, tự động hóa càng bền.
- **Operational**: mỗi app một process độc lập → cô lập sự cố, restart riêng, đo lường riêng, giới hạn tài nguyên riêng (cgroups/container làm việc này ở tầng process).
- **Scalability**: app xuất khẩu dịch vụ qua port → load balancer chỉ cần danh sách `địa chỉ:port` để phân phối; nhân bản app = thêm entry vào danh sách. Port binding là *mặt tiếp xúc* để F8 (Concurrency) hoạt động.
- **Composability**: app này bind port → app khác trỏ tới nó qua URL trong config → app này trở thành **backing service** (F4) của app kia. Toàn bộ kiến trúc service-to-service xây trên vòng lặp đó.

## 3. Bản chất

Ba mệnh đề cốt lõi:

1. **Self-contained**: app nhúng server (HTTP, gRPC...) như một *library* (`net/http`), không sống *trong* server như một guest. Đảo ngược quan hệ sở hữu: app sở hữu server, không phải server sở hữu app.
2. **Hợp đồng tối giản với nền tảng**: "cho tôi biết `PORT`, tôi sẽ nghe ở đó". Không đăng ký, không file descriptor đặc biệt, không giao thức quản lý riêng.
3. **Routing là việc của tầng ngoài**: app nghe trên port *nội bộ*; việc ánh xạ ra thế giới (domain, TLS, load balancing, path routing) thuộc về router/ingress của nền tảng. App không biết và không cần biết nó được expose thế nào.

**Điều gì xảy ra nếu vi phạm?** Hardcode port → hai app đụng port trên cùng máy, nền tảng không điều phối được. Phụ thuộc app server ngoài → toàn bộ vấn đề mô hình ký sinh quay lại. App tự làm TLS/routing/virtual host → trùng trách nhiệm với ingress, cấu hình rải hai nơi, đổi domain phải sửa app.

## 4. Cách áp dụng với Go

### 4.1. HTTP server chuẩn production

Go là ngôn ngữ minh họa lý tưởng: `net/http` biến port binding thành mặc định tự nhiên. Điểm cần chú ý là các chi tiết production quanh nó:

```go
// cmd/server/main.go
package main

import (
	"context"
	"errors"
	"fmt"
	"log/slog"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"
)

func main() {
	logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))

	port := os.Getenv("PORT") // hợp đồng với nền tảng: PORT từ env (F3)
	if port == "" {
		port = "8080"
	}

	mux := http.NewServeMux()
	mux.HandleFunc("GET /healthz", func(w http.ResponseWriter, r *http.Request) {
		w.WriteHeader(http.StatusOK) // liveness — chương 18 bàn kỹ
	})
	mux.HandleFunc("GET /api/products/{id}", getProduct)

	srv := &http.Server{
		Addr:    ":" + port, // bind 0.0.0.0 — trong container PHẢI thế,
		Handler: mux,        // bind 127.0.0.1 là lỗi kinh điển: ngoài container không gọi vào được

		// Timeout server-side: thiếu chúng, client chậm/độc chiếm connection
		// làm cạn tài nguyên — bắt buộc cho production
		ReadHeaderTimeout: 5 * time.Second,
		ReadTimeout:       10 * time.Second,
		WriteTimeout:      30 * time.Second,
		IdleTimeout:       120 * time.Second,
	}

	// Chạy server trong goroutine để main còn đợi signal (F9 — graceful shutdown)
	errCh := make(chan error, 1)
	go func() {
		logger.Info("listening", "port", port)
		if err := srv.ListenAndServe(); err != nil && !errors.Is(err, http.ErrServerClosed) {
			errCh <- err
		}
	}()

	ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
	defer stop()

	select {
	case err := <-errCh:
		logger.Error("server error", "err", err)
		os.Exit(1)
	case <-ctx.Done():
		shutdownCtx, cancel := context.WithTimeout(context.Background(), 15*time.Second)
		defer cancel()
		if err := srv.Shutdown(shutdownCtx); err != nil {
			logger.Error("shutdown", "err", err)
		}
		logger.Info("stopped cleanly")
	}
}

func getProduct(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, `{"id": %q}`, r.PathValue("id"))
}
```

Port binding không chỉ cho HTTP — gRPC theo đúng cùng một mẫu:

```go
lis, err := net.Listen("tcp", ":"+port)
if err != nil { log.Fatal(err) }
grpcServer := grpc.NewServer()
pb.RegisterOrderServiceServer(grpcServer, svc)
grpcServer.Serve(lis) // graceful: grpcServer.GracefulStop()
```

### 4.2. Docker — port là bề mặt công khai của container

```dockerfile
# Dockerfile (trích)
ENV PORT=8080
EXPOSE 8080          # tài liệu hóa hợp đồng port (không tự mở gì cả)
ENTRYPOINT ["/app"]
```

```bash
docker run -p 80:8080 myapp   # nền tảng ánh xạ port ngoài ↔ port app — app không biết, không quan tâm
```

### 4.3. Kubernetes — chuỗi routing hoàn chỉnh

```yaml
# Chuỗi: Ingress (domain/TLS) → Service (load balance) → Pod:port (app)
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      containers:
        - name: server
          image: registry.example.com/myapp@sha256:...
          env:
            - name: PORT
              value: "8080"
          ports:
            - name: http          # đặt TÊN cho port — nơi khác tham chiếu theo tên
              containerPort: 8080
---
apiVersion: v1
kind: Service
metadata: { name: myapp }
spec:
  selector: { app: myapp }
  ports:
    - port: 80
      targetPort: http            # theo tên: đổi port app không sửa Service
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp
spec:
  tls: [{ hosts: [api.example.com], secretName: api-tls }]  # TLS terminate Ở ĐÂY, không trong app
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend: { service: { name: myapp, port: { number: 80 } } }
```

Phân giao trách nhiệm rõ: **app** nghe `:8080` và nói HTTP thuần; **Service** load-balance giữa các pod; **Ingress** lo domain, TLS, path. Đổi domain, thêm chứng chỉ, chuyển CDN — không chạm vào app.

## 5. Trade-off

**TLS trong app vs terminate ở ingress.** Mặc định đúng: terminate ở ingress/mesh — quản lý cert tập trung (cert-manager), app đơn giản. Ngoại lệ: yêu cầu mã hóa end-to-end tới tận pod (zero-trust, compliance) → mTLS, và cách sạch là **service mesh sidecar/ambient** (Istio, Linkerd) để app *vẫn* nói plaintext còn mã hóa là việc của nền tảng — giữ đúng tinh thần factor. Tự làm TLS trong app là lựa chọn cuối.

**Một port vs nhiều port.** App thực tế thường nghe 2 port: port chính (traffic) và port quản trị (metrics `/metrics`, health, pprof). Tách port quản trị khỏi port công khai là best practice bảo mật (ingress không route tới port quản trị) — không vi phạm factor, miễn cả hai đều từ config.

**Unix socket vs TCP port.** Giao tiếp cùng-máy qua unix socket nhanh hơn chút và kiểm soát quyền qua filesystem — nhưng phá tính đồng nhất (không route qua network được, không load-balance). Chỉ dùng cho mẫu sidecar rất cụ thể; mặc định là TCP port.

## 6. Best Practices

- `PORT` từ env, default hợp lý; bind `0.0.0.0` trong container; port < 1024 không cần thiết (đòi quyền root/capability — dùng 8080 và để nền tảng map).
- Đủ bộ timeout trên `http.Server` (ReadHeader/Read/Write/Idle) — Go **không** có default, quên là lỗ hổng DoS.
- Đặt tên port trong K8s manifest, tham chiếu theo tên.
- Port quản trị (metrics, pprof, health) tách khỏi port công khai; chặn ở ingress/NetworkPolicy.
- App không tự quản lý virtual host/domain/TLS — đó là việc của ingress; app chỉ biết "tôi nghe ở PORT".

## 7. Anti-patterns

- **Hardcode port trong code** — đụng độ khi nền tảng cần xếp nhiều process; phá hợp đồng với PaaS (Heroku/Cloud Run *cấp* PORT, app phải nghe đúng đó).
- **Bind `127.0.0.1` trong container** — health check pass từ trong container, mọi traffic ngoài fail; một trong những lỗi "container chạy mà không ai gọi được" phổ biến nhất.
- **Kiến trúc đòi app server ngoài** — quay lại mô hình ký sinh với mọi hệ lụy của nó.
- **Nhét routing/TLS/domain logic vào app** (`if host == "api.example.com"`) — trùng vai với ingress, config rải hai nơi.
- **Expose port quản trị ra công khai** — pprof/metrics/debug endpoint mở ra internet là sự cố bảo mật đợi ngày phát nổ.
- **Thiếu timeout trên server** — không phải anti-pattern của port binding thuần túy, nhưng là lỗi đi kèm phổ biến nhất khi "tự nhúng server".

## 8. Khi nào KHÔNG áp dụng

- **Worker/consumer thuần** (đọc queue, không nhận request): không cần bind port cho *traffic* — nhưng vẫn nên bind port *quản trị* cho health/metrics; K8s probe cũng có thể dùng exec thay port.
- **CLI tool, batch job**: chạy xong thoát, không có khái niệm "phục vụ".
- **Serverless function** (Lambda): nền tảng thay hợp đồng port bằng hợp đồng hàm — cùng tinh thần (hợp đồng tối giản, nền tảng lo routing), khác cơ chế. Lambda Web Adapter thậm chí chuyển hợp đồng hàm về... hợp đồng port, cho thấy port binding phổ quát đến mức nào.
- **FastCGI/PHP-FPM và các mô hình legacy** vẫn vận hành tốt ở hệ thống hiện hữu — đừng đập đi chỉ vì "không đúng factor"; áp dụng cho phần mới trước.

---

## Tóm tắt

- Port binding = app là **process tự chứa**, nhúng server như library, nghe trên **PORT cấp từ env** — phủ định mô hình "app sống trong app server".
- Hợp đồng tối giản app–nền tảng: nền tảng cấp port và route traffic; app không biết gì về domain/TLS/load balancer.
- App bind port → app trở thành backing service của app khác — vòng lặp nền tảng của kiến trúc service-to-service.
- Với Go: `net/http` + PORT từ env + **đủ bộ timeout** + bind `0.0.0.0` + tách port quản trị. Routing/TLS thuộc Ingress.
