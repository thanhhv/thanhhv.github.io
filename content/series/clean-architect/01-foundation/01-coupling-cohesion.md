+++
title = "Chương 1.1 — Coupling & Cohesion: Gốc rễ của mọi vấn đề kiến trúc"
date = "2026-07-07T19:00:00+07:00"
draft = false
tags = ["backend", "golang", "clean-architecture"]
series = ["Clean Architecture với Golang"]
+++

> **Level 1 – Foundation** · Đối tượng: Backend Engineer trở lên
> Đây là chương quan trọng nhất của toàn bộ tài liệu. Nếu bạn hiểu sâu coupling và cohesion, mọi pattern phía sau — SOLID, Dependency Rule, Clean Architecture, DDD — chỉ là hệ quả logic.

---

## 1. Problem Statement

### Bài toán kinh doanh

Hãy bắt đầu từ một tình huống có thật, không phải từ lý thuyết.

Một công ty e-commerce có hệ thống đặt hàng viết bằng Go, chạy tốt trong 2 năm đầu. Đến năm thứ 3:

- Thêm một phương thức thanh toán mới mất **3 tuần** thay vì 3 ngày.
- Sửa logic tính phí vận chuyển làm **hỏng luôn** tính năng khuyến mãi, dù hai tính năng này "không liên quan".
- Không ai dám refactor vì "đụng vào là vỡ", test suite chạy 40 phút và fail ngẫu nhiên.
- Onboard một engineer mới mất 2 tháng mới dám merge PR đầu tiên.

Chi phí kinh doanh: thời gian ra thị trường (time-to-market) tăng, chi phí engineering tăng phi tuyến theo kích thước codebase, và tệ nhất — **công ty mất khả năng phản ứng với thị trường**.

Câu hỏi cốt lõi: *Tại sao code càng lớn càng khó sửa?* Câu trả lời không nằm ở ngôn ngữ, framework hay database. Nó nằm ở hai đại lượng: **coupling** (mức độ phụ thuộc giữa các phần) và **cohesion** (mức độ gắn kết bên trong một phần).

### Nếu không kiểm soát coupling/cohesion thì điều gì xảy ra?

- **Change amplification**: một thay đổi nghiệp vụ nhỏ lan ra hàng chục file. Sửa một dòng trong module A buộc bạn sửa B, C, D.
- **Cognitive load**: để hiểu một hàm, bạn phải hiểu 5 module khác mà nó chạm vào.
- **Unknown unknowns**: bạn không biết thay đổi của mình phá vỡ cái gì cho đến khi production cháy.

Ba triệu chứng này chính là định nghĩa của "legacy code" — và chúng đều là hệ quả trực tiếp của coupling cao + cohesion thấp.

### Cách tiếp cận cũ và hạn chế

Trước khi có tư duy kiến trúc hiện đại, ngành phần mềm đã thử:

| Cách tiếp cận | Ý tưởng | Vì sao thất bại |
|---|---|---|
| "Cứ viết cho chạy" | Tối ưu tốc độ ra tính năng | Nợ kỹ thuật lãi kép; sau 18–24 tháng velocity về gần 0 |
| Chia theo file/folder | Tách code thành nhiều file cho "gọn" | Chia vật lý nhưng không chia logic — các file vẫn gọi chằng chịt lẫn nhau |
| Comment & tài liệu | Bù đắp bằng docs | Docs lỗi thời ngay khi code đổi; không ngăn được phụ thuộc sai |
| Quy ước miệng ("đừng gọi thẳng DB từ handler nhé") | Kỷ luật tự giác | Không có cơ chế cưỡng chế (enforcement) → vi phạm tích lũy dần |

Bài học: **kỷ luật không thể thay thế cấu trúc**. Kiến trúc tốt là kiến trúc khiến việc làm đúng dễ hơn việc làm sai.

---

## 2. Tại sao khái niệm này tồn tại

- **Business Problem**: doanh nghiệp cần thay đổi phần mềm liên tục với chi phí dự đoán được. Chi phí thay đổi tỉ lệ thuận với số điểm phải chạm vào — tức tỉ lệ thuận với coupling.
- **Technical Problem**: con người chỉ giữ được ~7 khái niệm trong đầu cùng lúc. Module có coupling cao buộc lập trình viên nạp toàn bộ hệ thống vào đầu để sửa một chỗ.
- **Design Problem**: chúng ta cần một đơn vị đo để trả lời "thiết kế này tốt hay xấu?" một cách khách quan, thay vì cảm tính. Coupling và cohesion (do Larry Constantine đưa ra từ thập niên 1960 trong Structured Design) chính là đơn vị đo đó — và chúng sống lâu hơn mọi framework.

---

## 3. Giải thích bản chất

### 3.1 Coupling — phụ thuộc là gì, thực sự?

Định nghĩa hình thức: *Module A coupled với module B nếu thay đổi B có thể buộc A phải thay đổi.*

Chú ý từ "**có thể**". Coupling không phải là "A đang gọi B", mà là "A **biết** gì về B". Kiến thức (knowledge) mới là thứ tạo ra phụ thuộc. A càng biết nhiều chi tiết của B — struct nội bộ, thứ tự gọi hàm, format lỗi, schema DB mà B dùng — thì xác suất A vỡ khi B đổi càng cao.

Đây là insight quan trọng nhất: **giảm coupling = giảm lượng kiến thức mà một module cần biết về module khác**, không phải "không gọi nhau". Hai module gọi nhau qua một interface hẹp, ổn định có coupling thấp hơn hai module không gọi nhau nhưng cùng đọc chung một bảng database.

### 3.2 Các mức độ coupling (từ tệ nhất đến tốt nhất)

1. **Content coupling** — A sửa trực tiếp dữ liệu nội bộ của B (truy cập field, biến global của B). Tệ nhất.
2. **Common coupling** — A và B dùng chung state toàn cục (biến global, bảng DB chung, singleton mutable).
3. **Control coupling** — A truyền flag để điều khiển luồng bên trong B (`process(order, isRetry, skipValidation)`).
4. **Stamp coupling** — A truyền cho B cả struct lớn trong khi B chỉ cần 2 field.
5. **Data coupling** — A truyền cho B đúng những gì B cần, qua tham số rõ ràng. Tốt.
6. **Message coupling** — A và B chỉ biết nhau qua contract/message, không biết implementation. Tốt nhất.

Trong Go, các mức này ánh xạ trực tiếp:

```go
// (1) Content coupling — package order thò tay vào nội tạng package user
user.Cache["u123"].Profile.Tier = "GOLD" // sửa map nội bộ của package khác

// (2) Common coupling — hai package cùng đọc/ghi biến global
var GlobalConfig map[string]string // ai cũng đọc, ai cũng ghi

// (3) Control coupling — flag điều khiển hành vi nội bộ
func ProcessOrder(o Order, skipStockCheck bool, isAdminOverride bool) error

// (5) Data coupling — chỉ truyền cái cần
func CalculateShippingFee(weightKg float64, destZone Zone) (Money, error)

// (6) Message coupling — chỉ biết contract
type PaymentGateway interface {
    Charge(ctx context.Context, req ChargeRequest) (ChargeResult, error)
}
```

### 3.3 Coupling hướng (afferent/efferent) — thứ Clean Architecture thực sự quản lý

Với hệ thống lớn, câu hỏi không còn là "coupling nhiều hay ít" mà là "**coupling trỏ về hướng nào**":

- **Efferent coupling (Ce)**: module này phụ thuộc bao nhiêu module khác (đi ra).
- **Afferent coupling (Ca)**: bao nhiêu module khác phụ thuộc module này (đi vào).
- **Instability** `I = Ce / (Ca + Ce)`: module có I ≈ 0 rất khó đổi (ai cũng phụ thuộc nó), module có I ≈ 1 dễ đổi (không ai phụ thuộc nó).

Nguyên tắc rút ra (Stable Dependencies Principle): **phụ thuộc phải trỏ về phía ổn định hơn**. Business rule của bạn thay đổi ít hơn framework HTTP hay driver DB — vậy driver phải phụ thuộc business rule, không phải ngược lại. *Đây chính là tiền đề của Dependency Rule trong Clean Architecture* — chúng ta sẽ chứng minh điều này ở chương 2.

### 3.4 Cohesion — gắn kết là gì?

Định nghĩa: *các thành phần trong một module có cùng lý do tồn tại và cùng lý do thay đổi hay không.*

Module cohesion cao: mọi hàm, struct trong đó phục vụ **một** trách nhiệm nghiệp vụ. Module cohesion thấp: cái túi tạp hóa — `utils`, `common`, `helpers`, `shared`.

Các mức cohesion (tốt dần):

1. **Coincidental** — gom ngẫu nhiên (`utils.go` chứa `FormatDate`, `RetryHTTP`, `HashPassword`).
2. **Logical** — gom theo thể loại kỹ thuật (`all handlers`, `all repositories`) chứ không theo nghiệp vụ.
3. **Temporal** — gom vì chạy cùng lúc (`init.go` khởi tạo mọi thứ).
4. **Communicational** — cùng thao tác trên một dữ liệu.
5. **Functional** — mọi thứ phục vụ đúng một nhiệm vụ nghiệp vụ. Tốt nhất.

Điểm tinh tế: **layer-based packaging (`handlers/`, `services/`, `repos/`) chỉ đạt logical cohesion** — mức thứ 2 từ dưới lên. Feature-based packaging (`order/`, `payment/`) mới đạt functional cohesion. Đây là lý do chương 3 (Go Project Structure) sẽ khuyến nghị tổ chức theo feature.

### 3.5 Quan hệ nghịch đảo và điều nó bảo vệ

Coupling và cohesion là hai mặt của cùng một quyết định: **đặt ranh giới (boundary) ở đâu**.

- Đặt ranh giới đúng → những thứ hay đổi cùng nhau nằm cạnh nhau (cohesion cao), những thứ đổi độc lập bị ngăn cách (coupling thấp).
- Đặt ranh giới sai → một thay đổi nghiệp vụ phải xuyên qua nhiều ranh giới (shotgun surgery), hoặc một module ôm nhiều lý do thay đổi (god object).

Kiến trúc, xét cho cùng, là **nghệ thuật vẽ ranh giới**. Clean Architecture chỉ là một bộ quy tắc vẽ ranh giới đã được kiểm chứng.

---

## 4. Cách hoạt động: đo và nhìn thấy coupling trong Go

### 4.1 Dependency flow trong Go là gì?

Trong Go, đơn vị coupling là **package**. Câu lệnh `import` là khai báo phụ thuộc tường minh, và compiler **cấm import vòng** — đây là món quà kiến trúc lớn nhất của Go: bạn *bắt buộc* phải nghĩ về hướng phụ thuộc từ ngày đầu.

```
┌────────────┐     import      ┌────────────┐     import      ┌────────────┐
│  handler    │ ───────────────▶│  service    │ ───────────────▶│  repository │
│ (HTTP)      │                 │ (business)  │                 │ (Postgres)  │
└────────────┘                 └────────────┘                 └────────────┘
     Control flow ─────▶            Control flow ─────▶
     Dependency flow ───▶           Dependency flow ───▶   ← vấn đề: business phụ thuộc hạ tầng!
```

Ở sơ đồ trên, **control flow** (thứ tự gọi hàm lúc runtime) và **dependency flow** (hướng import lúc compile-time) trùng nhau. Chương 1.3 (Dependency Inversion) sẽ chỉ ra cách tách hai luồng này — đó là toàn bộ "phép màu" của Clean Architecture.

### 4.2 Đo coupling thực tế

```bash
# Liệt kê mọi package mà package order phụ thuộc (efferent)
go list -deps ./internal/order | grep myapp

# Ai đang phụ thuộc vào package postgres? (afferent)
grep -r "myapp/internal/postgres" --include="*.go" -l | sort -u

# Vẽ đồ thị phụ thuộc
go install github.com/loov/goda@latest
goda graph ./... | dot -Tsvg -o deps.svg
```

Chỉ số đáng theo dõi trong review kiến trúc định kỳ:

- Package nghiệp vụ import bao nhiêu package hạ tầng? (mục tiêu: **0**)
- Package `utils`/`common` có bao nhiêu afferent coupling? (càng cao càng nguy hiểm — nó là điểm nghẽn thay đổi)
- Độ sâu chuỗi import dài nhất? (chuỗi dài = change amplification lớn)

---

## 5. Code minh họa: cùng một nghiệp vụ, hai mức coupling

Bài toán: *tính phí vận chuyển cho đơn hàng, cộng phụ phí nếu khách không phải hạng GOLD.*

### 5.1 Phiên bản coupling cao (rất phổ biến trong thực tế)

Cấu trúc thư mục:

```
shipping-bad/
├── go.mod
├── main.go
└── shipping/
    └── shipping.go
```

```go
// shipping-bad/shipping/shipping.go
package shipping

import (
	"database/sql"
	"encoding/json"
	"fmt"
	"net/http"
	"strconv"
)

// DB là biến global — common coupling: mọi hàm trong mọi package đều có thể
// đọc/ghi, không ai kiểm soát được vòng đời và transaction.
var DB *sql.DB

// HandleShippingFee ôm trọn 4 trách nhiệm: HTTP, truy vấn DB,
// business rule, và serialization. Cohesion: coincidental.
func HandleShippingFee(w http.ResponseWriter, r *http.Request) {
	orderID, err := strconv.ParseInt(r.URL.Query().Get("order_id"), 10, 64)
	if err != nil {
		http.Error(w, "bad order_id", http.StatusBadRequest)
		return
	}

	// Coupling với schema DB: handler biết tên bảng, tên cột.
	// Đổi schema → sửa handler.
	var weightKg float64
	var customerID int64
	err = DB.QueryRow(
		`SELECT weight_kg, customer_id FROM orders WHERE id = $1`, orderID,
	).Scan(&weightKg, &customerID)
	if err != nil {
		http.Error(w, "order not found", http.StatusNotFound)
		return
	}

	var tier string
	err = DB.QueryRow(
		`SELECT tier FROM customers WHERE id = $1`, customerID,
	).Scan(&tier)
	if err != nil {
		http.Error(w, "customer not found", http.StatusNotFound)
		return
	}

	// Business rule bị chôn giữa code hạ tầng — muốn unit test phải
	// dựng cả HTTP server lẫn database.
	fee := weightKg * 12000
	if tier != "GOLD" {
		fee += 15000
	}
	if fee < 20000 {
		fee = 20000
	}

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(map[string]any{
		"order_id": orderID,
		"fee_vnd":  fmt.Sprintf("%.0f", fee),
	})
}
```

**Vì sao đây là quả bom hẹn giờ:**

- Muốn tính phí từ một background worker hoặc gRPC? Copy-paste — vì logic bị hàn chết vào `http.ResponseWriter`.
- Muốn test rule "phí tối thiểu 20.000"? Phải dựng Postgres + HTTP request. Test chậm → ít test → sợ sửa.
- Đổi cột `weight_kg` thành `weight_grams`? Sửa handler — nghĩa là **thay đổi hạ tầng lan vào nơi chứa nghiệp vụ**.
- 2 năm sau, 30 handler như thế này cùng đọc bảng `customers` → đổi schema là "đại phẫu".

### 5.2 Phiên bản coupling thấp, cohesion cao

Cấu trúc thư mục:

```
shipping-good/
├── go.mod                    # module example.com/shipping
├── main.go                   # composition root: nối mọi thứ lại
└── internal/
    ├── shipping/             # NGHIỆP VỤ — không import gì ngoài stdlib
    │   ├── shipping.go
    │   └── shipping_test.go
    ├── postgres/             # hạ tầng — phụ thuộc VÀO nghiệp vụ
    │   └── order_repo.go
    └── httpapi/              # delivery — phụ thuộc VÀO nghiệp vụ
        └── handler.go
```

```go
// internal/shipping/shipping.go
package shipping

import (
	"context"
	"errors"
	"fmt"
)

// ===== Domain types: ngôn ngữ của nghiệp vụ, không biết gì về HTTP/SQL =====

type Tier string

const (
	TierStandard Tier = "STANDARD"
	TierGold     Tier = "GOLD"
)

type Money int64 // VND, tránh float cho tiền tệ

type Order struct {
	ID       int64
	WeightKg float64
	Customer Customer
}

type Customer struct {
	ID   int64
	Tier Tier
}

var ErrOrderNotFound = errors.New("shipping: order not found")

// OrderReader là NHU CẦU của nghiệp vụ, khai báo tại nơi tiêu thụ.
// Nghiệp vụ nói: "tôi cần ai đó đưa tôi Order" — không quan tâm từ đâu.
// Đây là message coupling: hẹp nhất có thể (1 method, đúng dữ liệu cần).
type OrderReader interface {
	OrderByID(ctx context.Context, id int64) (Order, error)
}

// ===== Business rules: thuần túy, deterministic, test bằng bảng =====

const (
	ratePerKg   Money = 12_000
	nonGoldSurcharge  Money = 15_000
	minimumFee  Money = 20_000
)

// Fee chứa TOÀN BỘ rule tính phí. Functional cohesion:
// mọi dòng ở đây tồn tại vì đúng một lý do — "cách tính phí vận chuyển".
func Fee(o Order) Money {
	fee := Money(o.WeightKg * float64(ratePerKg))
	if o.Customer.Tier != TierGold {
		fee += nonGoldSurcharge
	}
	if fee < minimumFee {
		fee = minimumFee
	}
	return fee
}

// ===== Use case: điều phối, vẫn không biết HTTP/SQL là gì =====

type Service struct {
	orders OrderReader
}

func NewService(orders OrderReader) *Service {
	return &Service{orders: orders}
}

func (s *Service) QuoteFee(ctx context.Context, orderID int64) (Money, error) {
	o, err := s.orders.OrderByID(ctx, orderID)
	if err != nil {
		return 0, fmt.Errorf("quote fee for order %d: %w", orderID, err)
	}
	return Fee(o), nil
}
```

```go
// internal/shipping/shipping_test.go
package shipping

import "testing"

// Unit test business rule: không DB, không HTTP, chạy trong micro-giây.
func TestFee(t *testing.T) {
	cases := []struct {
		name string
		o    Order
		want Money
	}{
		{"gold khong phu phi", Order{WeightKg: 2, Customer: Customer{Tier: TierGold}}, 24_000},
		{"standard co phu phi", Order{WeightKg: 2, Customer: Customer{Tier: TierStandard}}, 39_000},
		{"ap dung phi toi thieu", Order{WeightKg: 0.1, Customer: Customer{Tier: TierGold}}, 20_000},
	}
	for _, tc := range cases {
		t.Run(tc.name, func(t *testing.T) {
			if got := Fee(tc.o); got != tc.want {
				t.Fatalf("Fee() = %d, want %d", got, tc.want)
			}
		})
	}
}
```

```go
// internal/postgres/order_repo.go
package postgres

import (
	"context"
	"database/sql"
	"errors"
	"fmt"

	"example.com/shipping/internal/shipping" // hạ tầng import nghiệp vụ — KHÔNG ngược lại
)

type OrderRepo struct{ db *sql.DB }

func NewOrderRepo(db *sql.DB) *OrderRepo { return &OrderRepo{db: db} }

// OrderByID thỏa mãn shipping.OrderReader một cách NGẦM ĐỊNH (structural typing).
// Package shipping không hề biết file này tồn tại.
func (r *OrderRepo) OrderByID(ctx context.Context, id int64) (shipping.Order, error) {
	const q = `
		SELECT o.id, o.weight_kg, c.id, c.tier
		FROM orders o JOIN customers c ON c.id = o.customer_id
		WHERE o.id = $1`
	var o shipping.Order
	err := r.db.QueryRowContext(ctx, q, id).
		Scan(&o.ID, &o.WeightKg, &o.Customer.ID, &o.Customer.Tier)
	if errors.Is(err, sql.ErrNoRows) {
		return shipping.Order{}, shipping.ErrOrderNotFound
	}
	if err != nil {
		return shipping.Order{}, fmt.Errorf("query order %d: %w", id, err)
	}
	return o, nil
}
```

```go
// internal/httpapi/handler.go
package httpapi

import (
	"encoding/json"
	"errors"
	"net/http"
	"strconv"

	"example.com/shipping/internal/shipping"
)

type Handler struct{ svc *shipping.Service }

func NewHandler(svc *shipping.Service) *Handler { return &Handler{svc: svc} }

// Handler chỉ làm việc của HTTP: parse, gọi use case, render.
// Không một dòng business rule nào ở đây.
func (h *Handler) QuoteFee(w http.ResponseWriter, r *http.Request) {
	orderID, err := strconv.ParseInt(r.URL.Query().Get("order_id"), 10, 64)
	if err != nil {
		http.Error(w, "bad order_id", http.StatusBadRequest)
		return
	}
	fee, err := h.svc.QuoteFee(r.Context(), orderID)
	switch {
	case errors.Is(err, shipping.ErrOrderNotFound):
		http.Error(w, "order not found", http.StatusNotFound)
		return
	case err != nil:
		http.Error(w, "internal error", http.StatusInternalServerError)
		return
	}
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(map[string]any{"order_id": orderID, "fee_vnd": fee})
}
```

```go
// main.go — composition root: NƠI DUY NHẤT biết mọi lớp
package main

import (
	"database/sql"
	"log"
	"net/http"

	_ "github.com/lib/pq"

	"example.com/shipping/internal/httpapi"
	"example.com/shipping/internal/postgres"
	"example.com/shipping/internal/shipping"
)

func main() {
	db, err := sql.Open("postgres", "postgres://localhost/shop?sslmode=disable")
	if err != nil {
		log.Fatal(err)
	}
	defer db.Close()

	// Dependency Injection thủ công — tường minh, dễ đọc, không cần framework.
	repo := postgres.NewOrderRepo(db)
	svc := shipping.NewService(repo)
	h := httpapi.NewHandler(svc)

	mux := http.NewServeMux()
	mux.HandleFunc("GET /shipping/fee", h.QuoteFee)
	log.Fatal(http.ListenAndServe(":8080", mux))
}
```

### 5.3 Luồng request và luồng phụ thuộc

```
Control flow (runtime):
  HTTP request ──▶ httpapi.Handler ──▶ shipping.Service ──▶ postgres.OrderRepo ──▶ DB

Dependency flow (compile-time):
  httpapi  ────────▶ shipping ◀──────── postgres
                        ▲
                        │
                     main (import tất cả, chỉ để lắp ráp)
```

Hai luồng **không còn trùng nhau**: control flow đi từ HTTP xuống DB, nhưng dependency flow đều chụm về `shipping`. Package nghiệp vụ có instability I ≈ 0 (ổn định nhất, được mọi thứ phụ thuộc) và **import zero package hạ tầng**. Đây chính là Dependency Inversion — sẽ mổ xẻ ở chương 1.3.

**Điều bạn mua được:**

- Test rule tính phí: 3 test case, 0 dependency, < 1ms.
- Thêm gRPC endpoint: viết package `grpcapi` mới, không đụng `shipping`.
- Đổi Postgres → MongoDB: viết `mongo.OrderRepo`, đổi 1 dòng trong `main.go`.
- Schema DB đổi: chỉ sửa `postgres/order_repo.go`.

---

## 6. Trade-off

Không có bữa trưa miễn phí. Phiên bản 5.2 trả giá:

| Trục | Phiên bản coupling cao | Phiên bản coupling thấp |
|---|---|---|
| Số file/khái niệm | 1 file, đọc một mạch | 5 file, phải nhảy qua lại |
| Thời gian viết lần đầu | ~15 phút | ~45 phút |
| Chi phí thay đổi năm thứ 1 | Thấp | Ngang hoặc cao hơn chút |
| Chi phí thay đổi năm thứ 3 | Tăng phi tuyến | Gần như tuyến tính |
| Testability | Cần DB + HTTP | Unit test thuần |
| Onboarding | Dễ hiểu 1 file, khó hiểu 30 file như thế | Cần hiểu quy ước, sau đó mọi feature giống nhau |

Nguyên tắc quyết định: **đầu tư giảm coupling tỉ lệ với tuổi thọ kỳ vọng và tần suất thay đổi của hệ thống**. Script chạy một lần: viết kiểu 5.1 là đúng đắn. Hệ thống core sống 5 năm với 10 engineer: kiểu 5.1 là vô trách nhiệm.

Cảnh giác chiều ngược lại: **abstraction cũng là coupling** — coupling với chính abstraction đó. Interface 15 method mà chỉ có 1 implementation là bạn đã trả chi phí indirection mà không mua được flexibility nào. Giảm coupling ≠ thêm interface; giảm coupling = giảm kiến thức lẫn nhau.

---

## 7. Best Practices trong Go

1. **Package đặt tên theo nghiệp vụ, không theo thể loại kỹ thuật.** `shipping`, `order`, `payment` — không phải `models`, `services`, `utils`. Tên package là câu trả lời cho "package này chịu trách nhiệm về *cái gì*", không phải "chứa *loại code* nào".
2. **Interface khai báo ở phía tiêu thụ (consumer side), nhỏ nhất có thể.** `shipping.OrderReader` 1 method, nằm trong package shipping — không phải `repository.IOrderRepository` 20 method nằm cạnh implementation. (Chi tiết ở chương 4.)
3. **Cấm biến global mutable.** Mọi dependency truyền qua constructor. Biến global là common coupling — dạng coupling vô hình và tệ gần nhất.
4. **Truyền đúng dữ liệu cần, không truyền cả thế giới.** Hàm cần `weightKg` và `tier` thì đừng nhận `*gin.Context`.
5. **Không truyền control flag giữa các module.** `Process(o, skipValidation bool)` → tách thành hai hàm hoặc hai use case.
6. **Dùng `internal/` để cưỡng chế ranh giới.** Compiler sẽ chặn package ngoài import vào — kỷ luật bằng công cụ, không bằng lời hứa.
7. **Đo định kỳ.** Đưa `go list -deps` + linter (`depguard`, `gomodguard`) vào CI để chặn import sai hướng ngay tại PR:

```yaml
# .golangci.yml — chặn package nghiệp vụ import hạ tầng
linters-settings:
  depguard:
    rules:
      business-purity:
        files: ["**/internal/shipping/**"]
        deny:
          - pkg: "database/sql"
            desc: "business package must not know about SQL"
          - pkg: "net/http"
            desc: "business package must not know about HTTP"
```

---

## 8. Anti-patterns

### 8.1 Package `utils` / `common` / `shared`

Cohesion thấp nhất (coincidental) + afferent coupling cao nhất: mọi package đều import nó, nó chứa mọi thứ không ai muốn nhận. Hệ quả: đổi 1 hàm trong utils → rebuild và re-test cả hệ thống; utils phình vô hạn vì "tiện thì bỏ vào".
**Khắc phục:** giải tán theo trách nhiệm — `FormatVND` về package `money`, `RetryHTTP` về package `httpclient`. Nếu một hàm chỉ có 1 nơi dùng, chuyển nó về đúng nơi đó, thậm chí unexported.

### 8.2 God package / God struct

Một package `service` chứa `OrderService` 3.000 dòng xử lý order + payment + inventory + email. Mọi thay đổi nghiệp vụ đều chạm vào nó → merge conflict liên tục, không thể phân chia ownership giữa các team.
**Khắc phục:** tách theo lý do thay đổi (xem SRP, chương 1.2). Câu hỏi kiểm tra: "những ai/bộ phận nào sẽ yêu cầu sửa file này?" — nếu nhiều hơn một, tách.

### 8.3 Coupling qua database (integration database)

Hai module "độc lập" cùng SELECT/UPDATE một bảng. Không có import nào giữa chúng, đồ thị package sạch đẹp — nhưng đổi schema là cả hai cùng vỡ. Đây là coupling **vô hình với compiler**, nguy hiểm hơn import chằng chịt.
**Khắc phục:** mỗi bảng có đúng một module làm chủ (owner); module khác truy cập qua API/interface của owner.

### 8.4 Leaky abstraction

`OrderReader` trả về `*sql.Rows` hoặc error kiểu `pq.Error` — interface tồn tại nhưng kiến thức về hạ tầng vẫn rò rỉ qua kiểu dữ liệu. Coupling không giảm, chỉ bị giấu đi.
**Khắc phục:** contract chỉ dùng kiểu domain (`shipping.Order`, `shipping.ErrOrderNotFound`); dịch lỗi hạ tầng sang lỗi domain ngay tại adapter.

### 8.5 "Giảm coupling" bằng cách copy-paste

Sợ phụ thuộc nên mỗi service tự copy logic tính phí. Coupling giảm trên đồ thị nhưng **duplication chính là coupling ngầm**: rule đổi → phải nhớ sửa N nơi, và chắc chắn sẽ quên một nơi.
**Khắc phục:** rule nghiệp vụ có một nguồn chân lý duy nhất; chia sẻ qua package domain hoặc qua service sở hữu rule đó. Duplication chấp nhận được với code *ngẫu nhiên giống nhau* (incidental duplication), không chấp nhận được với *cùng một rule nghiệp vụ*.

---

## 9. Khi nào KHÔNG cần tối ưu coupling/cohesion

- **Script, tool chạy một lần, POC**: tuổi thọ ngắn hơn thời gian thiết kế. Viết thẳng, xóa sau.
- **CRUD admin tool nội bộ < 2.000 dòng**: một file handler thẳng xuống DB là thiết kế *đúng*, không phải thiết kế *tạm*.
- **Chưa biết ranh giới nghiệp vụ ở đâu (MVP giai đoạn khám phá)**: vẽ ranh giới sớm khi chưa hiểu domain sẽ vẽ **sai**, và ranh giới sai còn đắt hơn không có ranh giới — bạn vừa trả phí indirection vừa phải phá ranh giới liên tục. Chiến lược đúng: viết cohesive nhưng ít tầng, đợi các trục thay đổi lộ ra rồi mới tách.
- Kiến trúc phù hợp hơn cho các trường hợp trên: transaction script (handler → SQL thẳng), hoặc monolith một package với quy ước đặt tên tốt.

Quy tắc ngón tay cái: **coupling là khoản vay, abstraction là bảo hiểm**. Vay khi cần tốc độ và biết mình sẽ trả; mua bảo hiểm khi tài sản (codebase) đủ lớn và rủi ro (tần suất thay đổi) đủ cao.

---

## Tóm tắt chương

- Coupling = lượng kiến thức module A biết về module B. Giảm coupling là giảm kiến thức, không phải cắt liên lạc.
- Cohesion = các phần trong module có cùng lý do thay đổi không. Feature-based > layer-based.
- Kiến trúc là nghệ thuật đặt ranh giới; Clean Architecture là một bộ quy tắc đặt ranh giới cụ thể.
- Phụ thuộc phải trỏ về phía ổn định — tiền đề của Dependency Rule.
- Trade-off là thật: trả phí thiết kế hôm nay để mua chi phí thay đổi tuyến tính ngày mai — chỉ đáng khi hệ thống sống đủ lâu.

**Chương tiếp theo:** [SOLID — năm nguyên tắc quản trị phụ thuộc](/series/clean-architect/01-foundation/02-solid/), nơi coupling/cohesion được đóng gói thành quy tắc hành động cụ thể.
