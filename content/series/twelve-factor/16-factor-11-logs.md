+++
title = "Chương 16 — Factor 11: Logs"
date = "2026-07-10T17:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> *"Treat logs as event streams"* — Coi log là dòng sự kiện.
>
> **Phần 2 – The Twelve Factors** | Nhóm: **Operable** | Chương trước: [Dev/Prod Parity](/series/twelve-factor/15-factor-10-dev-prod-parity/) | Chương sau: [Admin Processes](/series/twelve-factor/17-factor-12-admin-processes/)

---

## 1. Problem Statement

Mô hình cũ: app ghi log vào file (`/var/log/app/app.log`), tự lo rotate, nén, xóa. Trên hạ tầng hiện đại, mô hình đó gãy ở mọi khớp:

- **Filesystem là tạm bợ**: container chết là file log chết theo — đúng lúc bạn cần log nhất (điều tra vì sao nó chết) thì log đã bay màu cùng nó.
- **Nhiều instance**: 30 pod = 30 file log trên 30 node — "grep file log" đòi hỏi biết pod nào xử lý request nào, điều bạn không thể biết trước.
- **App phải biết quá nhiều**: đường dẫn, quyền ghi, rotate policy, dung lượng — toàn tri thức về *môi trường*, vi phạm nguyên tắc environment coupling ngay từ chương 1. Tệ hơn: ghi vào filesystem phá `readOnlyRootFilesystem`, và disk đầy vì log là sự cố "tự bắn vào chân" kinh điển.

Factor 11 tách đôi trách nhiệm bằng một hợp đồng tối giản:

> **App chỉ làm một việc: ghi dòng sự kiện ra `stdout`/`stderr`, không buffer, không quản lý file. Việc thu gom, định tuyến, lưu trữ, đánh index là của MÔI TRƯỜNG THỰC THI (execution environment), không phải của app.**

## 2. Tại sao nguyên lý này tồn tại

- **Operational**: không SSH được vào 30 pod tạm bợ — dòng log phải *chảy ra khỏi* nơi sinh ra nó đến một chỗ tập trung, sống lâu hơn process.
- **Deployment**: cùng một app, dev muốn xem log ở terminal, staging đẩy vào Loki, production đẩy vào Elasticsearch + lưu trữ lạnh S3. Nếu app tự quyết định đích đến, mỗi môi trường phải một bản build/config log riêng; app ghi stdout thì **đích đến là quyết định của môi trường** — đổi cả hệ thống log không đụng một dòng code.
- **Scalability**: log của N pod cần được *hợp dòng* (merge theo thời gian, lọc theo request) — chỉ làm được ở tầng thu gom tập trung.
- **Business/Compliance**: audit log phải bất biến, lưu đủ hạn định — không thể giao cho filesystem của process tạm bợ.

## 3. Bản chất

**Log là một event stream** — dòng sự kiện bất tận, có thứ tự thời gian, mỗi sự kiện mô tả "điều gì vừa xảy ra". Nhìn như stream (thay vì file) mở ra kiến trúc:

```
   30 pod                    TẦNG THU GOM (env lo)         TIÊU THỤ
  ┌────────┐                ┌───────────────────┐
  │app→stdout│──┐            │ node agent         │──▶ Loki / Elastic  (tra cứu)
  ├────────┤  ├─ container ─▶│ (Fluent Bit,      │──▶ S3 / cold store (audit, rẻ)
  │app→stdout│──┤  runtime    │  Vector, Promtail)│──▶ alerting (đếm ERROR)
  ├────────┤  │  ghi ra      │ đọc file đó, gắn  │──▶ stream processing
  │app→stdout│──┘  node       │ metadata pod/node │    (nếu cần realtime)
  └────────┘                └───────────────────┘
   App: 1 việc — println      Hoàn toàn trong suốt với app
```

Chuỗi trách nhiệm: app ghi stdout → container runtime bắt stream ghi ra node → agent (DaemonSet) đọc, gắn metadata (pod, namespace, node), đẩy về backend. App không biết và không cần biết gì từ mắt xích thứ hai trở đi.

**Structured logging — nâng cấp bắt buộc của thời hiện đại.** 12-Factor gốc chỉ nói "dòng text ra stdout". Ở quy mô hàng triệu dòng/phút, text tự do (`log.Printf("user %s failed to pay %d", ...)`) là ngõ cụt: muốn tìm phải viết regex, muốn thống kê phải parse. Chuẩn hiện đại: **mỗi dòng là một JSON object** — log trở thành dữ liệu truy vấn được (`level=error AND user_id=42 AND route=/pay`), không phải văn xuôi để grep.

**Điều gì xảy ra nếu vi phạm?** Ghi file local: mất log khi pod chết, disk đầy, và mù trên hệ nhiều instance. Tự đẩy log sang backend từ trong app (gọi Elasticsearch trực tiếp): app coupling với hạ tầng log (đổi backend = sửa app), log backend chậm làm *app* chậm theo, và log trong khoảnh khắc app crash — khoảnh khắc quý nhất — thất lạc vì buffer trong app chưa flush.

## 4. Cách áp dụng với Go

### 4.1. `log/slog` — structured logging chuẩn thư viện chuẩn

```go
// internal/logging/logging.go
package logging

import (
	"log/slog"
	"os"
)

func New(level string) *slog.Logger {
	var lv slog.Level
	_ = lv.UnmarshalText([]byte(level)) // "debug"|"info"|"warn"|"error" — từ config (F3)

	h := slog.NewJSONHandler(os.Stdout, &slog.HandlerOptions{ // stdout — hết. Không file.
		Level: lv,
	})
	return slog.New(h)
}
```

```go
// Sử dụng trong handler — log là DỮ LIỆU, không phải câu văn
logger.Info("order created",
	"order_id", order.ID,
	"user_id", order.UserID,
	"amount", order.Amount,
	"duration_ms", time.Since(start).Milliseconds(),
)
// → {"time":"2026-07-08T09:15:02Z","level":"INFO","msg":"order created",
//    "order_id":"ord_9x2","user_id":"u_42","amount":150000,"duration_ms":23}

logger.Error("payment failed",
	"err", err,                    // error luôn có key riêng
	"order_id", order.ID,
	"gateway", "vnpay",
)
```

### 4.2. Correlation: request ID xuyên suốt — log 30 pod đọc như một câu chuyện

Log tập trung chỉ hữu ích khi ghép được các dòng của *cùng một request*. Middleware gắn request ID (nhận từ header nếu upstream đã tạo — chuẩn W3C `traceparent` — hoặc tự sinh), nhét logger-có-context vào `context.Context`:

```go
// internal/middleware/requestid.go
func RequestLogger(base *slog.Logger) func(http.Handler) http.Handler {
	return func(next http.Handler) http.Handler {
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			reqID := r.Header.Get("X-Request-Id")
			if reqID == "" {
				reqID = uuid.NewString()
			}
			// logger con mang sẵn ngữ cảnh — mọi dòng log sau tự có request_id
			l := base.With("request_id", reqID, "method", r.Method, "path", r.URL.Path)
			ctx := logging.WithLogger(r.Context(), l)

			start := time.Now()
			sw := &statusWriter{ResponseWriter: w, status: 200}
			next.ServeHTTP(sw, r.WithContext(ctx))

			l.Info("request completed",
				"status", sw.status,
				"duration_ms", time.Since(start).Milliseconds(),
			)
		})
	}
}
```

```go
// Ở tầng sâu bất kỳ: lấy logger từ context — request_id đi theo tự động
func (s *OrderService) Create(ctx context.Context, o *Order) error {
	log := logging.FromContext(ctx)
	log.Info("creating order", "order_id", o.ID)
	// ...
}
```

Khi tích hợp OpenTelemetry (chương 22), thay `request_id` bằng `trace_id`/`span_id` — log, metrics, traces nối với nhau thành một mặt phẳng quan sát.

### 4.3. Docker & Kubernetes — phần của môi trường

```bash
# Dev: stream hiện ngay trên terminal — chính là lợi ích của hợp đồng stdout
docker compose logs -f app
kubectl logs -f deploy/myapp --tail=100
```

```yaml
# fluent-bit DaemonSet (trích ý tưởng cấu hình) — chạy 1 bản mỗi node,
# app KHÔNG biết sự tồn tại của nó
# INPUT:  tail /var/log/containers/*.log   (nơi runtime ghi stdout của mọi pod)
# FILTER: kubernetes (gắn pod name, namespace, labels vào mỗi dòng)
# OUTPUT: loki / elasticsearch / s3       (quyết định của platform team)
```

Trên cloud managed (EKS/GKE/Cloud Run), tầng này thường có sẵn (CloudWatch, Cloud Logging) — bạn ghi stdout là xong, đúng nghĩa "việc của môi trường".

## 5. Trade-off

**JSON vs text đẹp cho người đọc.** JSON tối ưu cho máy, khó đọc bằng mắt ở terminal dev. Giải pháp phổ biến: handler theo config — dev dùng text/tint (đẹp, màu), staging/prod dùng JSON. Lưu ý đây là khác biệt *trình bày* chấp nhận được, nhưng đừng để hai handler khác nhau về *nội dung* (field nào có ở prod phải có ở dev — không thì bug về log chỉ lộ trên prod, vi phạm F10).

**Log bao nhiêu là đủ?** Log là chi phí thật: CPU serialize, băng thông, và tiền lưu trữ/index (các hãng log SaaS tính tiền theo GB — hệ thống log bừa bãi trả hóa đơn hàng chục nghìn USD/tháng là chuyện thường). Chiến lược: INFO cho sự kiện nghiệp vụ (một request một dòng tổng kết), DEBUG tắt ở prod và bật được qua config khi điều tra, ERROR phải *actionable* (có người đọc và làm gì đó). Cân nhắc sampling cho log lặp lại nhiều. Đừng dùng log thay metrics: "đếm số request" là việc của counter (rẻ), không phải của việc parse triệu dòng log (đắt) — chương 22.

**Sync vs async ghi log.** stdout mặc định là blocking — dưới tải cực cao, ghi log tranh chấp. Thực tế: với JSON handler của slog, chi phí ~vài trăm ns–µs mỗi dòng, không đáng lo với đa số. Nếu profile chỉ ra log là nút cổ chai: giảm lượng log trước (thường là nguyên nhân thật), rồi mới nghĩ đến buffered writer (và chấp nhận rủi ro mất vài dòng cuối khi crash).

## 6. Best Practices

- `slog.NewJSONHandler(os.Stdout)` + level từ env; **không mở file, không rotate, không tự đẩy backend từ trong app**.
- Mỗi request: một request ID (tôn trọng header từ upstream), logger-trong-context, một dòng tổng kết access log.
- Field chuẩn hóa toàn công ty: `level`, `msg`, `time`, `request_id`/`trace_id`, `service`, `version` — dashboard và alert viết một lần dùng cho mọi service.
- **Không log secret/PII**: password, token, số thẻ, email thô — dùng masking helper; đây là quy định pháp lý (GDPR/PDPA), không chỉ là gu.
- `stderr` cho chẩn đoán khởi động sớm (trước khi logger dựng xong); panic mặc định đã ra stderr — đừng nuốt.
- Alert đếm trên log ERROR có chọn lọc; log kèm `version` (buildinfo chương 6) để so sánh trước/sau deploy.
- Retention phân tầng: hot (7–30 ngày, index đầy đủ) / cold (S3, nén, vài năm cho audit).

## 7. Anti-patterns

- **Ghi log vào file trong container** — mất khi pod chết, phá read-only filesystem, disk node đầy; mô hình cũ nguyên vẹn mọi nhược điểm.
- **App gọi thẳng Elasticsearch/Kafka để gửi log** — coupling, backend log chậm kéo app chậm, mất log lúc crash; nếu tổ chức muốn log qua Kafka, đó là việc của agent, không phải của app.
- **`fmt.Println` / `log.Printf` phi cấu trúc rải rác** — không level, không field, không tắt được debug; nghìn dòng "here 2" của quá khứ.
- **Log câu văn thay vì dữ liệu** (`"User Nam vừa mua 3 món hết 150k"`) — regex-hell cho người vận hành; chuyển thành field.
- **Nuốt error thay vì log** (`_ = doSomething()`) hoặc **log rồi ném tiếp cùng error ở mọi tầng** — một lỗi in 5 lần, alert đếm 5 lần; quy tắc: log tại nơi *xử lý* error, các tầng giữa chỉ wrap (`fmt.Errorf("...: %w", err)`).
- **Log PII/secret** — sự cố bảo mật qua đường log, nơi quyền đọc rộng nhất công ty.
- **Sidecar log cho từng pod khi node agent là đủ** — nhân chi phí thu gom lên số pod thay vì số node, không thêm giá trị.

## 8. Khi nào KHÔNG áp dụng đầy đủ

- **CLI tool**: stdout là *output của chương trình*, log chẩn đoán đi stderr — đúng chuẩn Unix, đừng trộn (JSON log lẫn vào output mà user đang pipe là phá UX).
- **Desktop/mobile app**: không có "môi trường thu gom" — ghi file local + cơ chế gửi báo cáo lỗi là đúng.
- **Embedded**: có thể chỉ có serial/flash — hoàn cảnh quyết định.
- **Hệ legacy ghi file đang chạy ổn**: không cần sửa app ngay — agent tail file cũng đạt 80% giá trị (log tập trung), sửa app ra stdout khi có dịp refactor.

---

## Tóm tắt

- Hợp đồng: **app ghi event ra stdout — mọi thứ còn lại (thu gom, định tuyến, lưu, index) là việc của môi trường.** App không biết log đi đâu là một *tính năng*.
- Chuẩn hiện đại: **structured JSON logging** (`log/slog`), level từ config, request_id/trace_id xuyên suốt qua context — log của 30 pod đọc như một câu chuyện có thể truy vấn.
- Log là chi phí: một dòng tổng kết mỗi request, DEBUG tắt ở prod, đừng dùng log làm metrics, retention phân tầng.
- Lỗi chí mạng cần nhớ: file trong container, app tự đẩy log đi backend, log secret/PII.
