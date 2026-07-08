+++
title = "Chương 22 — Observability: Logs, Metrics, Traces và OpenTelemetry"
date = "2026-07-10T23:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> **Phần 3 – Engineering** | Chương trước: [CI/CD](/series/twelve-factor/21-cicd/) | Chương sau: [Chủ đề Principal](/series/twelve-factor/23-chu-de-principal/)

Factor 11 chỉ nói về log — vì năm 2011 hệ thống điển hình là một app monolith vài instance. Hệ phân tán hiện đại cần nhiều hơn: **observability** — khả năng đặt câu hỏi *chưa nghĩ trước* về trạng thái hệ thống từ dữ liệu nó phát ra. Đây là "factor 13" không chính thức mà mọi bản cập nhật 12-Factor đều bổ sung.

---

## 1. Problem Statement

Monitoring truyền thống trả lời câu hỏi **đã biết trước**: "CPU > 80%?", "service có sống không?". Nó đủ khi hệ thống là một khối: sự cố ít dạng, đoán trước được.

Hệ phân tán chết theo cách không đoán trước: một request đi qua 6 service, chậm ở đâu đó, chỉ với user Việt Nam, chỉ khi giỏ hàng > 10 món. Không dashboard dựng sẵn nào trả lời được — bạn cần đặt câu hỏi *mới* trên dữ liệu *đã thu*: đó là observability. Không có nó, quy trình sự cố là "nhìn log 6 service bằng mắt và đoán" — MTTR tính bằng giờ, và nhiều sự cố không bao giờ có root cause.

## 2. Ba trụ — mỗi trụ một câu hỏi, một cấu trúc chi phí

| Trụ | Trả lời | Đặc tính | Chi phí |
|---|---|---|---|
| **Metrics** | *Có* vấn đề không? Ở mức nào? | Số tổng hợp theo thời gian; rẻ, giữ lâu, alert được | Thấp (cardinality là kẻ thù duy nhất) |
| **Traces** | Vấn đề *ở đâu* trong chuỗi gọi? | Cây span theo request xuyên service | Trung bình (sampling để kiểm soát) |
| **Logs** | *Chi tiết* chuyện gì đã xảy ra? | Sự kiện rời rạc, ngữ cảnh giàu (chương 16) | Cao nhất per-byte |

Quy trình điều tra chuẩn nối ba trụ: **alert từ metrics** ("p99 checkout > 2s") → **mở trace** của các request chậm (thấy span `payment-svc → bank API` chiếm 1.8s) → **nhảy đến logs** đúng service, đúng `trace_id` (thấy lỗi cụ thể). Mỗi bước là một cú click **nếu** ba loại dữ liệu chia sẻ chung định danh — `trace_id` — và đó chính là việc của OpenTelemetry.

## 3. OpenTelemetry — vì sao nó là câu trả lời

Trước OTel: mỗi vendor một SDK (Datadog, New Relic, Jaeger...), đổi backend = sửa code toàn bộ — chính là environment coupling ở tầng telemetry, vi phạm tinh thần F4. OpenTelemetry (CNCF, hợp nhất OpenTracing + OpenCensus) chuẩn hóa: **một API/SDK trung lập trong code + giao thức OTLP + Collector đứng giữa**. App phát telemetry một kiểu duy nhất; backend là quyết định của môi trường — **đúng nguyên tắc Factor 11 mở rộng cho cả ba trụ**.

```
  app (OTel SDK) ──OTLP──▶ OTel Collector ──▶ Prometheus/Mimir (metrics)
                            (agent/gateway,  ──▶ Tempo/Jaeger    (traces)
                             batch, filter,  ──▶ Loki/Elastic    (logs)
                             sample, route)  ──▶ hoặc Datadog/Grafana Cloud — đổi ở ĐÂY,
                                                 không đổi trong app
```

## 4. Áp dụng với Go

### 4.1. Khởi tạo OTel SDK

```go
// internal/telemetry/otel.go
package telemetry

import (
	"context"

	"go.opentelemetry.io/otel"
	"go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracegrpc"
	"go.opentelemetry.io/otel/propagation"
	sdkresource "go.opentelemetry.io/otel/sdk/resource"
	sdktrace "go.opentelemetry.io/otel/sdk/trace"
	semconv "go.opentelemetry.io/otel/semconv/v1.26.0"
)

// Setup trả về hàm shutdown — gọi trong giai đoạn cleanup (F9)
func Setup(ctx context.Context, serviceName, version string) (func(context.Context) error, error) {
	// Endpoint collector từ env chuẩn OTEL_EXPORTER_OTLP_ENDPOINT (F3 —
	// SDK tự đọc; không hardcode)
	exp, err := otlptracegrpc.New(ctx)
	if err != nil {
		return nil, err
	}

	res, _ := sdkresource.Merge(sdkresource.Default(), sdkresource.NewWithAttributes(
		semconv.SchemaURL,
		semconv.ServiceName(serviceName),
		semconv.ServiceVersion(version), // nối telemetry về release (F5!)
	))

	tp := sdktrace.NewTracerProvider(
		sdktrace.WithBatcher(exp),
		sdktrace.WithResource(res),
		// Sampling: cha quyết định, gốc lấy 10% — chỉnh bằng env
		// OTEL_TRACES_SAMPLER=parentbased_traceidratio OTEL_TRACES_SAMPLER_ARG=0.1
	)
	otel.SetTracerProvider(tp)
	// Propagator W3C: trace context đi xuyên service qua header `traceparent`
	otel.SetTextMapPropagator(propagation.NewCompositeTextMapPropagator(
		propagation.TraceContext{}, propagation.Baggage{}))
	return tp.Shutdown, nil
}
```

### 4.2. Instrument ba điểm chạm — HTTP vào, HTTP/DB ra, logic trong

```go
// (1) HTTP server: otelhttp bọc router — mỗi request một span gốc/nối tiếp upstream
handler := otelhttp.NewHandler(router, "http.server")

// (2) HTTP client & DB: dùng wrapper có sẵn — context mang trace đi tiếp
client := &http.Client{Transport: otelhttp.NewTransport(http.DefaultTransport)}
// pgx: github.com/exaring/otelpgx — mỗi query một span con, thấy SQL chậm ngay trên trace

// (3) Span thủ công cho bước nghiệp vụ quan trọng:
func (s *OrderService) Create(ctx context.Context, o *Order) error {
	ctx, span := otel.Tracer("order").Start(ctx, "OrderService.Create")
	defer span.End()
	span.SetAttributes(attribute.Int64("order.amount", o.Amount))
	// ... — mọi IO bên dưới nhận ctx này → span con tự nối vào cây
}
```

```go
// Log nối vào trace: handler slog tự gắn trace_id từ context
// (slog-otel hoặc tự viết ~20 dòng) → từ trace click sang log và ngược lại
l := logger.With("trace_id", span.SpanContext().TraceID().String())
```

Chú ý mẫu số chung: **`context.Context` là mạch máu** — kỷ luật context của chương 18 giờ trả cổ tức: trace tự lan truyền qua đúng đường ống đã xây.

### 4.3. Metrics — RED cho service, đừng đợi nghĩ ra metric mới

Bộ khởi đầu chuẩn cho mọi service: **RED** — Rate (RPS), Errors (tỷ lệ lỗi), Duration (histogram latency — p50/p95/p99, *không bao giờ* chỉ trung bình). Thêm bộ **USE** cho tài nguyên (runtime Go: goroutines, heap, GC — có sẵn qua instrumentation runtime), và metric nghiệp vụ chọn lọc (orders_created_total).

```go
// otelhttp đã phát sẵn http.server.request.duration...
// Metric nghiệp vụ:
meter := otel.Meter("order")
ordersCreated, _ := meter.Int64Counter("orders.created")
ordersCreated.Add(ctx, 1, metric.WithAttributes(attribute.String("payment", "vnpay")))
// CẢNH BÁO cardinality: KHÔNG bỏ user_id/order_id vào label metric —
// mỗi giá trị label là một time series; ID làm nổ tung Prometheus.
// ID thuộc về traces/logs; label metric chỉ dành cho chiều CÓ HẠN (status, method, plan).
```

### 4.4. Kubernetes — Collector là phần của môi trường

```yaml
# App chỉ cần env (F3):
env:
  - name: OTEL_EXPORTER_OTLP_ENDPOINT
    value: http://otel-collector.observability:4317
  - name: OTEL_SERVICE_NAME
    value: myapp
# Collector (Helm chart) do platform team vận hành: nhận OTLP từ mọi app,
# route đến backend — đổi Datadog→Grafana là đổi config Collector, 0 app phải sửa
```

## 5. SLO — observability có mục đích

Dữ liệu không tự có ý nghĩa. Khung chưng cất chuẩn: **SLI** (chỉ số đo được — "tỷ lệ request checkout < 500ms") → **SLO** (mục tiêu — "99.9% trong 30 ngày") → **error budget** (0.1% được phép hỏng — ngân sách chi cho deploy táo bạo; cháy nhanh thì đóng băng release). Giá trị kép: alert theo *burn rate* của SLO thay vì theo "CPU cao" — chỉ báo động khi user thật sự đau, chấm dứt alert fatigue; và error budget biến "ổn định vs tốc độ ship" từ tranh cãi cảm tính thành con số thương lượng được giữa engineering và product.

## 6. Trade-off

**Chi phí telemetry là thật.** Hệ thống lớn có hóa đơn observability chiếm 10–30% hóa đơn hạ tầng. Đòn bẩy: sampling trace (head 10% + tail sampling giữ 100% lỗi/chậm — làm ở Collector); metric cardinality kiểm soát nghiêm; log level + retention phân tầng (chương 16). Nguyên tắc phân bổ: metrics phủ 100% (rẻ), traces sample có chủ đích, logs chi tiết nhưng ngắn hạn.

**Tự vận hành (Grafana stack: Prometheus/Loki/Tempo) vs SaaS (Datadog...).** SaaS: nhanh có giá trị, đắt theo scale, lock-in dashboard/alert. Tự vận hành: rẻ hơn ở scale lớn nhưng là một hệ thống production nữa phải nuôi. Nhờ OTel, quyết định này **đảo chiều được** — bắt đầu SaaS, xuống tự vận hành khi hóa đơn vượt lương một kỹ sư, không sửa app.

**Instrument đến đâu?** Auto-instrumentation (HTTP, DB) cho 80% giá trị với 5% công sức — làm ngay. Span thủ công cho từng hàm nghiệp vụ: rơi vào noise. Ngưỡng: span cho bước *có thể chậm hoặc hỏng độc lập* (gọi ngoài, transaction phức tạp); attribute cho chiều *bạn sẽ muốn lọc khi điều tra*.

## 7. Anti-patterns

- **Đợi sự cố mới lắp observability** — dữ liệu không tồn tại ngược thời gian; sự cố đầu tiên của hệ thống mù là sự cố không bao giờ có root cause.
- **Alert trên nguyên nhân thay vì triệu chứng** ("CPU > 80%" thay vì "p99 vượt SLO") — CPU cao mà user không đau là noise; alert fatigue giết on-call.
- **ID vô hạn trong label metric** — nổ cardinality; bài học đắt nhất của người mới dùng Prometheus.
- **Log thay metrics** (đếm dashboard bằng query log) — đắt gấp trăm lần, chậm, và gãy khi đổi format log.
- **Mỗi service một chuẩn telemetry** (service A dùng Datadog SDK, B tự chế header) — trace đứt gãy ở ranh giới service, đúng nơi cần nó nhất; chuẩn hóa OTel + W3C traceparent toàn tổ chức.
- **Thu 100% trace** ở tải lớn "cho chắc" — trả tiền cho hàng triệu trace giống hệt nhau; tail sampling giữ đúng thứ đáng xem.
- **Dashboard vạn năng 60 panel** không ai hiểu — mỗi service một dashboard RED + SLO; sâu hơn thì đi bằng exploratory query, không phải dashboard tĩnh.

## 8. Khi nào KHÔNG cần đầy đủ

- **Monolith một service**: chưa cần distributed tracing (không có "distributed") — metrics RED + structured logs + request_id là đủ; lắp OTel sẵn vẫn rẻ nếu định tách service sau.
- **Internal tool ít người dùng**: log tốt + uptime check là đủ; SLO cho tool 5 user là nghi lễ rỗng.
- **Batch job**: metrics theo run (thành công/thời lượng/số bản ghi) quan trọng hơn trace per-request.
- Nhưng ở **mọi** quy mô: structured logs với request_id (chương 16) và bộ RED cơ bản — chi phí một buổi chiều, giá trị từ sự cố đầu tiên.

---

## Tóm tắt

- Observability = trả lời được câu hỏi **chưa nghĩ trước**; ba trụ chia vai: metrics (*có* vấn đề?), traces (*ở đâu*?), logs (*chi tiết gì*?) — nối bằng `trace_id`.
- OpenTelemetry là "F4/F11 cho telemetry": app phát một chuẩn (OTLP qua env config), backend là quyết định của môi trường qua Collector — đổi vendor không sửa app.
- Go: otelhttp/otelpgx cho 80% giá trị; context là mạch máu; RED metrics mặc định; cardinality là kẻ thù số một.
- SLO + error budget biến dữ liệu thành quyết định: alert khi user đau, thương lượng tốc độ ship bằng ngân sách lỗi.
