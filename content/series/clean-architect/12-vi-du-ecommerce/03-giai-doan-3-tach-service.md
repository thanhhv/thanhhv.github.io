+++
title = "Chương 12.3 — E-commerce · Giai đoạn 3: Tách service khi tổ chức đòi hỏi"
date = "2026-07-08T13:00:00+07:00"
draft = false
tags = ["backend", "golang", "clean-architecture"]
series = ["Clean Architecture với Golang"]
+++

> Năm thứ 4: 40 engineer, 6 team, 500 nghìn đơn/ngày. Đây là chương về *lý do đúng* và *lý do sai* để tách, cùng kỹ thuật tách không ngừng hệ thống.

---

## 1. Khi nào monolith thật sự hết vai trò

Lý do **đúng** (xuất hiện ở công ty này):

- **Deploy coupling thành nút cổ chai tổ chức**: 6 team xếp hàng chung một release train; một bug của team Catalog rollback cả deploy chứa hotfix của team Payment. Chi phí phối hợp tăng theo bình phương số team.
- **Chênh lệch tải & tài nguyên**: đường search catalog cần 40 pod CPU cao vào giờ vàng; đường payment cần ổn định tuyệt đối và audit riêng (PCI-DSS) — chung binary nghĩa là scale cả cụm theo nhu cầu của phần nóng nhất, và mọi engineer đều "ở trong phạm vi audit".
- **Bán kính sự cố**: memory leak trong module notification OOM-kill cả tiến trình chứa payment.

Lý do **sai** (đã bị bác trong design review): "microservices hiện đại hơn", "để dùng ngôn ngữ mới", "CV-driven". Phép thử: *nếu tách xong mà số team, quy trình deploy và yêu cầu scale y nguyên — bạn vừa mua độ trễ mạng và partial failure mà không mua được gì.*

Quyết định: tách **hai** service đầu tiên — `payment` (vì ranh giới compliance + yêu cầu ổn định) và `catalog-search` (vì profile tải khác hẳn). Order, user, loyalty, notification **ở lại monolith**. Microservices không phải trạng thái tất-cả-hoặc-không; hệ thống lành mạnh thường là "monolith + vài service có lý do".

## 2. Vì sao giai đoạn 2 làm giai đoạn 3 rẻ

Nhờ modular monolith chuẩn bị sẵn:

- `payment.Facade` đã là contract duy nhất order dùng → thay implementation từ "gọi hàm" sang "gọi gRPC" mà order **không đổi một dòng use case**.
- Payment đã sở hữu riêng bảng của nó, không JOIN chéo → tách database = di chuyển schema, không phải gỡ query.
- Giao tiếp module đã qua event bus → chuyển bus in-process sang Kafka là đổi adapter.

Đây là luận điểm trung tâm của toàn tài liệu ở quy mô lớn: **Clean Architecture trong từng module là thứ làm cho kiến trúc hệ thống (monolith ↔ services) trở thành quyết định đảo ngược được.**

## 3. Kỹ thuật tách: Strangler Fig từng bước

```
Bước 0 (đang có):   order ──func call──▶ payment module ──▶ payment DB (schema riêng)
Bước 1: định nghĩa payment.proto ≈ payment.Facade; build payment-svc binary
        từ CHÍNH code module payment (cùng repo — monorepo giúp bước này gần miễn phí)
Bước 2: order đổi adapter: PaymentFacade → gRPC client (feature flag chọn đường)
        ┌─ flag OFF: gọi in-process (đường cũ)
        └─ flag ON : gọi payment-svc (đường mới)  — so sánh kết quả shadow-traffic
Bước 3: chuyển 1% → 10% → 100% traffic; theo dõi latency/error budget
Bước 4: di chuyển schema payment sang database riêng (đã không còn JOIN chéo)
Bước 5: xóa module payment khỏi binary monolith; payment-svc sở hữu trọn vòng đời
```

Code bước 2 — điểm khéo nhất của cả quá trình:

```go
// internal/order/adapter/gateways/payment_grpc.go
// Implement CÙNG interface order đã dùng 2 năm — use case không hề biết có cuộc di cư.
type GRPCPaymentGateway struct {
	client paymentpb.PaymentServiceClient
}

func (g *GRPCPaymentGateway) Charge(ctx context.Context, req order.ChargeRequest) (order.ChargeResult, error) {
	resp, err := g.client.Charge(ctx, &paymentpb.ChargeRequest{
		OrderId: req.OrderID, Method: toProtoMethod(req.Method), AmountVnd: req.AmountVND,
	})
	if err != nil {
		if status.Code(err) == codes.FailedPrecondition {
			return order.ChargeResult{}, order.ErrPaymentDeclined // dịch về lỗi domain như mọi adapter
		}
		return order.ChargeResult{}, fmt.Errorf("payment-svc: %w", err) // transient — decorator retry lo
	}
	return order.ChargeResult{TxnID: resp.TxnId}, nil
}
```

```go
// cmd/shop/main.go — feature flag tại composition root, đúng nơi của nó
var payGw order.PaymentFacade
if cfg.UsePaymentService {
	payGw = gateways.NewGRPCPayment(paymentConn)
} else {
	payGw = paymentMod.LocalFacade()
}
orderMod := order.NewModule(db, bus, payGw, ...)
```

## 4. Những thứ mới phải trả tiền khi ranh giới thành mạng

Tách xong, ba loại chi phí mới xuất hiện — liệt kê để quyết định tách là quyết định *có giá niêm yết*:

**a) Partial failure & consistency liên-service.** "Reserve stock (monolith) + charge (payment-svc)" giờ là giao dịch phân tán thật. Giải bằng **saga** dựa trên chính các event đã có:

```
PlaceOrder saga (choreography):
  order: Save(PENDING) + OrderPlaced ──▶ payment-svc: tạo payment intent
  payment-svc: PaymentCompleted ──▶ order: MarkPaid
  payment-svc: PaymentFailed/timeout ──▶ order: Cancel + ReleaseStock (compensation)
Mỗi bước idempotent (event ID dedupe — chương 11), mỗi compensation là use case tường minh.
```

**b) Contract versioning.** `payment.proto` giờ là API giữa các team — thay đổi phải backward-compatible (thêm field optional, không đổi nghĩa field cũ), có deprecation window. Contract test (CDC hoặc bộ test proto chung trong monorepo) chạy ở CI cả hai phía.

**c) Vận hành**: service discovery, mTLS, distributed tracing (may mắn: OTel đã cắm từ chương 11 — trace xuyên service gần như miễn phí), on-call theo service, dashboard SLO riêng. Đây là chi phí cố định hàng tháng, không phải một lần.

## 5. Kiến trúc kết cục

```
                    ┌────────────────────────────────┐
   Client ──HTTP──▶ │  shop monolith                 │
                    │  (order, user, loyalty,        │──┐
                    │   notification, catalog-write) │  │ events
                    └──────────┬─────────────────────┘  ▼
                          gRPC │                    ┌─────────┐
                    ┌──────────▼──────────┐         │  Kafka  │
                    │  payment-svc        │────────▶│         │
                    │  (DB riêng, PCI)    │         └────┬────┘
                    └─────────────────────┘              │
                    ┌─────────────────────┐              │
   Client ──search─▶│  catalog-search-svc │◀── consume ──┘
                    │  (Elasticsearch)    │   (index projector — CQRS mức 3)
                    └─────────────────────┘
```

Bên trong **mỗi** hộp: vẫn là các vòng của chương 2.3 — domain/usecase/adapter. Kiến trúc vi mô (Clean Architecture) và kiến trúc vĩ mô (service topology) là hai quyết định độc lập; cái trước làm cái sau rẻ để thay đổi.

## 6. Bài học tổng kết chuỗi ba giai đoạn

1. **Kiến trúc đúng là kiến trúc đúng-thời-điểm**: G1 tối giản có chọn lọc → G2 siết ranh giới khi team đông → G3 tách khi tổ chức/tải đòi hỏi. Nhảy cóc giai đoạn nào cũng trả giá: G3 ngay từ đầu = chết vì vận hành; kẹt ở G1 mãi = chết vì merge conflict.
2. **Thứ bất biến qua cả ba giai đoạn** là các nguyên tắc Level 1: interface theo consumer, phụ thuộc trỏ vào nghiệp vụ, lỗi domain tách lỗi hạ tầng, event cho phản ứng liên module. Code use case của `order` sống sót gần như nguyên vẹn từ G1 đến G3 — đó là ROI của Clean Architecture bằng con số.
3. **Ranh giới dữ liệu quyết định chi phí tách**: mọi thứ khó của G3 đã được trả trước ở G2 bằng "mỗi bảng một chủ, không JOIN chéo". Team nào bỏ qua kỷ luật này, phí tách service tăng một bậc độ lớn.

**Tiếp theo:** [So sánh các kiến trúc](/series/clean-architect/13-so-sanh/01-so-sanh-cac-kien-truc/)
