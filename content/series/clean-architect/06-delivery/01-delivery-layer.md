+++
title = "Chương 6 — Delivery Layer: HTTP, gRPC, GraphQL, CLI, Worker"
date = "2026-07-08T05:00:00+07:00"
draft = false
tags = ["backend", "golang", "clean-architecture"]
series = ["Clean Architecture với Golang"]
+++

> **Level 2–3** · Bài kiểm tra thật sự của Clean Architecture: cùng một use case phục vụ N cách vào mà không đổi một dòng nghiệp vụ.

---

## 1. Problem Statement

Hệ thống trưởng thành hiếm khi chỉ có một cửa vào. Cùng nghiệp vụ "đổi điểm": mobile app gọi REST, service nội bộ gọi gRPC, chiến dịch marketing chạy batch qua CLI, và một Kafka consumer đổi điểm tự động khi có event mua hàng. Nếu nghiệp vụ viết trong HTTP handler, ba cửa còn lại chỉ có hai lựa chọn tồi: copy-paste logic, hoặc **giả lập HTTP request nội bộ** (gọi chính API của mình) — cả hai đều là nợ.

Nguyên tắc: **delivery mechanism là chi tiết**. Use case nhận input struct thuần, trả output struct thuần; mỗi cửa vào là một adapter dịch giao thức của nó ↔ struct đó. Trách nhiệm chuẩn của một delivery adapter, không hơn: decode/parse → validate *hình thức* (đúng kiểu, đúng format — validate *nghiệp vụ* thuộc domain) → gọi use case → dịch kết quả/lỗi về giao thức → encode. Khoảng 30–60 dòng cho một endpoint. Dài hơn nghĩa là nghiệp vụ đang rò vào.

## 2. HTTP/REST — stdlib Go 1.22+

Go 1.22 nâng cấp `net/http` routing (method + path param), đủ cho đa số service không cần framework:

```go
// adapter/httpapi/router.go
func NewRouter(h *Handler, mw ...Middleware) http.Handler {
	mux := http.NewServeMux()
	mux.HandleFunc("POST /v1/customers/{customerID}/redeem", h.Redeem)
	mux.HandleFunc("GET  /v1/customers/{customerID}/wallet", h.GetWallet)
	var root http.Handler = mux
	for i := len(mw) - 1; i >= 0; i-- { root = mw[i](root) } // compose cross-cutting
	return root
}
```

Ba kỷ luật quan trọng hơn việc chọn Gin/Echo/chi hay stdlib:

**a) DTO riêng cho API đã version hóa** — request/response struct sống trong package httpapi; API v1 đóng băng được trong khi domain tiến hóa.

**b) Bảng dịch lỗi tập trung** — một hàm `toHTTPStatus(err)` duy nhất map sentinel/typed error của domain sang status code, thay vì mỗi handler tự switch (đã minh họa chương 2.3; khi số lỗi lớn, dùng bảng `map[error]int` hoặc interface `interface{ HTTPStatus() int }` implement ở adapter).

**c) Middleware cho cross-cutting** — auth, logging, recover, timeout là decorator quanh handler (chương 1.4), tuyệt đối không rải trong từng handler.

Chọn framework khi nào? Cần validation binding, OpenAPI codegen, middleware ecosystem → `chi` (gần stdlib nhất) hoặc `echo/gin` đều ổn — **miễn kiểu của framework dừng ở adapter**. `*gin.Context` xuất hiện trong chữ ký use case là vi phạm chấm hết.

## 3. gRPC — thêm một cửa, không thêm nghiệp vụ

```go
// adapter/grpcapi/server.go
type Server struct {
	loyaltypb.UnimplementedLoyaltyServiceServer
	redeem *usecase.Redeem
}

func (s *Server) Redeem(ctx context.Context, req *loyaltypb.RedeemRequest) (*loyaltypb.RedeemResponse, error) {
	out, err := s.redeem.Execute(ctx, usecase.RedeemInput{
		CustomerID: req.GetCustomerId(),
		Cost:       domain.Points(req.GetCost()),
	})
	if err != nil {
		return nil, toGRPCStatus(err) // dịch lỗi domain → codes.FailedPrecondition, v.v.
	}
	return &loyaltypb.RedeemResponse{
		Voucher: string(out.Voucher), NewBalance: int64(out.NewBalance),
	}, nil
}

func toGRPCStatus(err error) error {
	switch {
	case errors.Is(err, domain.ErrInsufficientPoints):
		return status.Error(codes.FailedPrecondition, err.Error())
	case errors.Is(err, domain.ErrWalletNotFound):
		return status.Error(codes.NotFound, "wallet not found")
	case errors.Is(err, domain.ErrRedeemLimitReached):
		return status.Error(codes.ResourceExhausted, err.Error())
	default:
		return status.Error(codes.Internal, "internal error")
	}
}
```

Điểm kiến trúc then chốt: **struct protobuf sinh ra là DTO của vòng 4**. Cám dỗ lớn nhất của gRPC là dùng `*loyaltypb.Wallet` làm domain model "cho đỡ mapping" — làm vậy là trao quyền định hình domain cho file `.proto` (vốn bị ràng buộc bởi backward-compat wire format, kiểu dữ liệu nghèo, không method). Mapping tay là phí bảo hiểm ranh giới.

Cross-cutting của gRPC = interceptor (tương đương middleware). Cùng một app compile hai binary hoặc một binary hai port: HTTP + gRPC cùng trỏ vào một bộ use case — đây là khoảnh khắc kiến trúc trả lãi nhìn thấy được.

## 4. GraphQL — resolver là controller

Dùng `gqlgen` (codegen, type-safe). Quy tắc như trên: resolver = adapter mỏng, model GraphQL là DTO. Hai bẫy riêng của GraphQL:

- **N+1 tự nhiên**: mỗi field resolver gọi use case riêng → dùng dataloader *ở tầng adapter* (batch + cache trong request); đừng sửa use case để "trả sẵn mọi thứ" theo hình dạng query.
- **Query tự do phá bảo vệ aggregate**: client xin sâu tùy ý → giới hạn depth/complexity ở adapter. GraphQL schema là **một view công khai**, không phải bản sao domain model.

## 5. CLI và Background Worker — delivery như mọi delivery khác

Hai cửa vào hay bị viết "ngoài kiến trúc" nhất, dẫn đến logic lệch nhau giữa API và batch. Chúng là delivery adapter đúng nghĩa:

```go
// cmd/loyalty-cli/main.go — CLI dùng CHUNG use case với server
func main() {
	redeemCmd := flag.NewFlagSet("redeem", flag.ExitOnError)
	customer := redeemCmd.String("customer", "", "customer ID")
	cost := redeemCmd.Int64("cost", 0, "points to redeem")

	// composition root như mọi binary khác
	app := buildApp(config.MustLoad())

	switch os.Args[1] {
	case "redeem":
		redeemCmd.Parse(os.Args[2:])
		out, err := app.Redeem.Execute(context.Background(), usecase.RedeemInput{
			CustomerID: *customer, Cost: domain.Points(*cost),
		})
		if err != nil { log.Fatal(err) }
		fmt.Printf("voucher: %s, balance: %d\n", out.Voucher, out.NewBalance)
	}
}
```

```go
// adapter/kafkaconsumer/consumer.go — worker: dịch message → use case
func (c *Consumer) Run(ctx context.Context) error {
	for {
		msg, err := c.reader.FetchMessage(ctx)
		if err != nil { return err }

		var evt purchaseCompletedEvent // DTO của message schema
		if err := json.Unmarshal(msg.Value, &evt); err != nil {
			c.log.Error("bad message", "err", err) // poison message → DLQ, đừng retry mãi
			c.reader.CommitMessages(ctx, msg)
			continue
		}
		_, err = c.earn.Execute(ctx, usecase.EarnInput{
			CustomerID: evt.CustomerID, AmountVND: evt.TotalVND,
		})
		if err != nil {
			// lỗi tạm thời (DB down) → KHÔNG commit offset, sẽ đọc lại
			// lỗi nghiệp vụ (ví không tồn tại) → commit + log/DLQ
			if isTransient(err) { continue }
			c.log.Error("business reject", "err", err)
		}
		c.reader.CommitMessages(ctx, msg)
	}
}
```

Worker minh họa rõ nhất vì sao phân loại lỗi domain/transient (chương 5, 11) là việc của kiến trúc: quyết định *retry hay bỏ qua* phụ thuộc **loại** lỗi — nếu use case trả lỗi mù (`errors.New("something failed")`), consumer không thể quyết định đúng.

## 6. Trade-off & tổng kết trách nhiệm

| Delivery | Codegen | DTO | Dịch lỗi sang | Cross-cutting |
|---|---|---|---|---|
| REST | tùy chọn (OpenAPI) | JSON struct trong adapter | HTTP status | middleware |
| gRPC | bắt buộc (proto) | protobuf message | status codes | interceptor |
| GraphQL | gqlgen | GraphQL model | GraphQL errors | field middleware + dataloader |
| CLI | — | flags/args | exit code + stderr | flag parsing, output format |
| Worker | tùy schema | message struct | commit/retry/DLQ | consumer group, poison handling |

Chi phí chung: mỗi cửa một lớp mapping. Lợi ích: N cửa × 1 bộ nghiệp vụ; thêm cửa mới là công việc tuyến tính, không phải mổ nghiệp vụ. Với hệ thống chỉ có **một** cửa mãi mãi (webhook receiver nhỏ), lớp DTO tách bạch có thể giảm nhẹ — nhưng giữ nguyên quy tắc "handler không chứa rule".

## 7. Anti-patterns

- **Fat controller**: validate nghiệp vụ, tính toán, gọi 3 repo trực tiếp trong handler — use case chỉ còn trên giấy. Dấu hiệu: handler > 80 dòng, import repo/domain sâu.
- **Use case nhận `*http.Request`/`*gin.Context`** — nghiệp vụ dính giao thức, các cửa khác giả lập HTTP để gọi.
- **Domain entity serialize thẳng ra JSON công khai** — đổi tên field nội bộ = breaking change API (chương 2.2).
- **Logic khác nhau giữa API và worker** cho cùng nghiệp vụ — vì worker viết "nhanh gọn" ngoài use case. Mọi cửa qua một use case.
- **Middleware chứa nghiệp vụ** ("check hạng GOLD trong middleware auth") — rule tàng hình khỏi domain, không test được cùng nghiệp vụ.

## Tóm tắt

- Delivery = phiên dịch giao thức ↔ use case input/output; mỏng, không rule, lỗi dịch tập trung.
- Protobuf/GraphQL/JSON model đều là DTO vòng 4; mapping tay là phí bảo hiểm ranh giới.
- CLI và worker là delivery hạng nhất — dùng chung use case, hưởng chung kiến trúc.

**Chương tiếp theo:** [Integration — Kafka, RabbitMQ, Redis, External API](/series/clean-architect/07-integration/01-integration/)
