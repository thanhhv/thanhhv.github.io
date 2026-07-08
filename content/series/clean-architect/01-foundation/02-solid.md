+++
title = "Chương 1.2 — SOLID trong Go: Năm quy tắc quản trị phụ thuộc"
date = "2026-07-07T20:00:00+07:00"
draft = false
tags = ["backend", "golang", "clean-architecture"]
series = ["Clean Architecture với Golang"]
+++

> **Level 1 – Foundation** · Yêu cầu: đã đọc [Coupling & Cohesion](/series/clean-architect/01-foundation/01-coupling-cohesion/)

---

## 1. Problem Statement

Chương trước kết luận: kiến trúc tốt = coupling thấp + cohesion cao. Nhưng đó là *đại lượng đo*, không phải *quy tắc hành động*. Khi đứng trước một PR, engineer cần câu trả lời cụ thể: struct này nên tách không? Interface đặt ở đâu? Thêm case vào switch này có ổn không?

SOLID (Robert C. Martin tổng hợp, đầu những năm 2000) là năm quy tắc hành động trả lời chính xác các câu hỏi đó. **Không có SOLID**, mỗi engineer quyết định theo cảm tính → codebase thành tập hợp các phong cách mâu thuẫn, review kiến trúc thành tranh cãi ý kiến cá nhân.

**Hạn chế của cách tiếp cận cũ** ("code sao cho clean là được"): "clean" không đo được và không dạy được. SOLID biến trực giác của người giỏi thành checklist mà cả team dùng chung.

Lưu ý quan trọng cho Go: SOLID sinh ra trong thế giới Java/C++ (class, inheritance). Áp dụng nguyên xi vào Go là sai. Chương này trình bày **bản chất** từng nguyên tắc rồi dịch sang Go idiomatic — đôi chỗ bản dịch khác đáng kể nguyên gốc.

## 2. Tại sao SOLID tồn tại

- **Business Problem**: chi phí thay đổi phần mềm phải dự đoán được. SOLID là bộ quy tắc giữ cho "một yêu cầu nghiệp vụ mới = một vùng code phải sửa".
- **Technical Problem**: hệ thống hướng đối tượng/module hóa cho phép phụ thuộc mọc tự do; cần quy tắc quyết định phụ thuộc nào hợp lệ.
- **Design Problem**: cần ngôn ngữ chung để thảo luận thiết kế. "Vi phạm SRP" truyền đạt được trong 3 giây điều mà "tôi cảm thấy struct này hơi to" không truyền đạt được.

---

## 3. S — Single Responsibility Principle

### Bản chất

Phát biểu gốc thường bị hiểu sai thành "mỗi hàm làm một việc". Phát biểu đúng của Martin: **"A module should have one, and only one, reason to change"** — và "reason to change" nghĩa là **một nhóm người/vai trò trong tổ chức yêu cầu thay đổi** (actor).

SRP không nói về kích thước code. Nó nói về **trục thay đổi**: nếu phòng Kế toán và phòng Vận hành đều có thể yêu cầu sửa cùng một struct, struct đó vi phạm SRP — dù chỉ dài 50 dòng. Ngược lại, một hàm 300 dòng thuần một nghiệp vụ có thể hoàn toàn hợp lệ.

SRP là cohesion được phát biểu thành quy tắc: gom những gì thay đổi cùng nhau, tách những gì thay đổi vì lý do khác nhau.

### Vi phạm điển hình trong Go

```go
// ReportService phục vụ 3 actor khác nhau:
// - Kế toán quyết định CÁCH TÍNH doanh thu (CalculateRevenue)
// - Team platform quyết định ĐỊNH DẠNG file (ExportExcel)
// - DBA quyết định CÁCH LƯU (SaveToS3)
type ReportService struct{ db *sql.DB; s3 *s3.Client }

func (s *ReportService) CalculateRevenue(month time.Month) (Money, error) { ... }
func (s *ReportService) ExportExcel(rev Money) ([]byte, error)            { ... }
func (s *ReportService) SaveToS3(data []byte) error                       { ... }
```

Hậu quả thực tế: kế toán đổi công thức → PR đụng file mà team platform đang sửa format Excel → merge conflict, test của nhau vỡ, deploy phải phối hợp. **SRP vi phạm biến thay đổi độc lập thành thay đổi phải phối hợp** — chi phí tổ chức, không chỉ chi phí code.

### Sửa

```go
// package revenue — actor: Kế toán
func Calculate(orders []Order, month time.Month) Money { ... }

// package excelexport — actor: Platform
func Render(rev revenue.Money) ([]byte, error) { ... }

// package reportstore — actor: DBA/Infra
type S3Store struct{ client *s3.Client }
func (s *S3Store) Save(ctx context.Context, name string, data []byte) error { ... }

// Use case ghép chúng lại — thay đổi của use case là "quy trình", một actor thứ 4
func (u *MonthlyReport) Run(ctx context.Context, m time.Month) error {
	rev := revenue.Calculate(u.orders, m)
	file, err := excelexport.Render(rev)
	if err != nil { return err }
	return u.store.Save(ctx, fmt.Sprintf("report-%d.xlsx", m), file)
}
```

Câu hỏi kiểm tra khi review: *"Nếu tôi liệt kê các bên có thể yêu cầu sửa file này, danh sách có dài hơn 1 không?"*

---

## 4. O — Open/Closed Principle

### Bản chất

**"Open for extension, closed for modification"** — thêm hành vi mới mà không sửa code đã có. Nghe có vẻ ma thuật, thực chất là: **đặt điểm mở rộng (extension point) tại nơi bạn dự đoán biến thể sẽ xuất hiện**.

Điều OCP bảo vệ: code đã test, đã chạy production. Mỗi lần sửa file cũ là một cơ hội tạo regression; thêm file mới thì không. Điều OCP giảm: coupling giữa "khung xử lý" và "các biến thể".

### Trong Go: interface + registration, không phải inheritance

Vi phạm điển hình — switch mọc vô hạn:

```go
func (s *PaymentService) Charge(ctx context.Context, method string, amt Money) error {
	switch method {
	case "momo":    // 50 dòng gọi API MoMo
	case "vnpay":   // 50 dòng gọi API VNPay
	case "stripe":  // 50 dòng gọi Stripe
	// Mỗi cổng thanh toán mới = sửa hàm này = re-test toàn bộ = rủi ro
	}
}
```

Sửa theo OCP:

```go
// Contract — đóng với sửa đổi
type Gateway interface {
	Charge(ctx context.Context, req ChargeRequest) (ChargeResult, error)
}

type Service struct {
	gateways map[string]Gateway // mở với mở rộng
}

func (s *Service) Register(name string, g Gateway) { s.gateways[name] = g }

func (s *Service) Charge(ctx context.Context, method string, req ChargeRequest) (ChargeResult, error) {
	g, ok := s.gateways[method]
	if !ok {
		return ChargeResult{}, fmt.Errorf("payment: unsupported method %q", method)
	}
	return g.Charge(ctx, req)
}

// Thêm ZaloPay = thêm package mới + 1 dòng Register trong main.go.
// Không đụng vào service, không đụng vào MoMo/VNPay đã chạy ổn.
```

### Trade-off — OCP là con dao hai lưỡi

OCP chỉ đáng giá **tại trục biến thể có thật**. Nếu bạn abstract hóa mọi thứ "phòng khi cần", bạn tạo ra indirection vô nghĩa (xem anti-pattern "interface ở khắp mọi nơi", mục 9). Kinh nghiệm thực chiến: **lần đầu viết cụ thể; lần thứ hai xuất hiện biến thể mới trích interface**. Đến lúc đó bạn có 2 data point thật để thiết kế contract đúng, thay vì đoán.

Ngược lại, một `switch` trong Go **không tự động là vi phạm OCP**: switch trên tập đóng, ít đổi (trạng thái đơn hàng, ngày trong tuần) là hoàn toàn idiomatic. Vi phạm chỉ xảy ra khi tập case mở và mọc theo nghiệp vụ.

---

## 5. L — Liskov Substitution Principle

### Bản chất

**Mọi implementation của một contract phải thay thế được cho nhau mà caller không cần biết và không bị bất ngờ.** LSP là nguyên tắc về *hành vi*, không phải *chữ ký hàm*. Compiler Go kiểm tra chữ ký; LSP yêu cầu thêm: cùng ngữ nghĩa lỗi, cùng bất biến (invariant), không thêm tiền điều kiện, không làm yếu hậu điều kiện.

Điều LSP bảo vệ: **giá trị của abstraction**. Interface chỉ có ích nếu caller viết code một lần chạy đúng với mọi implementation. Một implementation "gian lận" phá hủy giá trị đó — caller buộc phải `if _, ok := repo.(*MongoRepo); ok { ... }` và abstraction chết.

### Vi phạm tinh vi trong Go

```go
type OrderRepo interface {
	// Contract ngầm định: trả ErrNotFound nếu không có
	ByID(ctx context.Context, id string) (Order, error)
}

// PostgresRepo: trả domain.ErrNotFound đúng contract ✓
// MongoRepo:    trả mongo.ErrNoDocuments — caller errors.Is(err, ErrNotFound) fail ✗
// CachedRepo:   trả (Order{}, nil) khi miss — zero value giả làm đơn hàng thật ✗✗
```

Cả ba compile. Hai cái sau tạo bug production khó lần: hành vi khác nhau tùy môi trường wire repo nào.

### Thực hành

- **Viết contract thành văn trong doc comment của interface**, đặc biệt: ngữ nghĩa lỗi, có được gọi song song không, có idempotent không, nil/zero value nghĩa là gì.
- **Viết contract test dùng chung** cho mọi implementation:

```go
// RunOrderRepoContract chạy được với Postgres, Mongo, InMemory —
// mọi implementation phải pass cùng một bộ test.
func RunOrderRepoContract(t *testing.T, newRepo func(t *testing.T) OrderRepo) {
	t.Run("not found tra dung sentinel error", func(t *testing.T) {
		repo := newRepo(t)
		_, err := repo.ByID(context.Background(), "khong-ton-tai")
		if !errors.Is(err, domain.ErrNotFound) {
			t.Fatalf("want ErrNotFound, got %v", err)
		}
	})
	// ... các bất biến khác
}
```

- Dịch lỗi hạ tầng sang lỗi domain **ngay tại adapter**, không để rò lên caller.

---

## 6. I — Interface Segregation Principle

### Bản chất

**Client không được buộc phụ thuộc vào method nó không dùng.** Mỗi method thừa trong interface là kiến thức thừa — tức coupling thừa: client bị recompile/re-mock/re-hiểu khi method nó không hề gọi thay đổi.

Go là ngôn ngữ hiếm hoi mà ISP được **văn hóa hóa**: *"The bigger the interface, the weaker the abstraction"* (Rob Pike). Stdlib là bằng chứng — `io.Reader`, `io.Writer` mỗi cái 1 method, và chúng compose thành cả hệ sinh thái.

### Vi phạm điển hình — "header interface"

```go
// Interface 12 method mọc theo implementation, không theo nhu cầu client
type UserRepository interface {
	Create(...) error
	Update(...) error
	Delete(...) error
	ByID(...) (User, error)
	ByEmail(...) (User, error)
	List(...) ([]User, error)
	Count(...) (int, error)
	Search(...) ([]User, error)
	BulkInsert(...) error
	UpdatePassword(...) error
	SetAvatar(...) error
	Deactivate(...) error
}
// Use case "đăng nhập" chỉ cần ByEmail — nhưng mock của nó phải stub 12 method.
```

### Sửa — interface theo nhu cầu từng use case

```go
// Trong package login — khai báo đúng cái login cần
type UserByEmail interface {
	ByEmail(ctx context.Context, email string) (User, error)
}

// Trong package register
type UserCreator interface {
	Create(ctx context.Context, u User) error
	ByEmail(ctx context.Context, email string) (User, error) // check trùng
}

// MỘT struct postgres.UserRepo với đầy đủ method thỏa mãn CẢ HAI
// interface một cách ngầm định. Implementation to, contract nhỏ.
```

Structural typing của Go làm ISP gần như miễn phí: bạn không cần sửa implementation hay khai báo `implements` — chỉ cần khai interface hẹp ở phía consumer. Đây là lý do quy tắc **"accept interfaces, return structs"** và **"interface thuộc về consumer"** là chuẩn mực Go (chi tiết chương 4).

---

## 7. D — Dependency Inversion Principle

### Bản chất (tóm tắt — cả chương 1.3 dành riêng cho nó)

**Module cấp cao (chính sách, nghiệp vụ) không được phụ thuộc module cấp thấp (chi tiết, hạ tầng). Cả hai phụ thuộc vào abstraction. Abstraction không phụ thuộc chi tiết; chi tiết phụ thuộc abstraction.**

Từ khóa là *inversion* — đảo. Mặc định tự nhiên khi viết code: cái gọi phụ thuộc cái bị gọi (`service` import `postgres`). DIP đảo mũi tên compile-time trong khi giữ nguyên control flow runtime: `service` khai báo interface nó cần; `postgres` import `service` để lấy kiểu và thỏa mãn interface đó.

DIP là **cơ chế duy nhất** cho phép business logic — thứ quý nhất, ổn định nhất — đứng ở tâm mà không biết gì về thế giới hạ tầng xoay quanh. Dependency Rule của Clean Architecture là DIP áp dụng đồng loạt lên toàn hệ thống. Chương 1.3 sẽ chứng minh từng bước bằng Go.

---

## 8. Tổng hợp: SOLID quy về một mối

| Nguyên tắc | Câu hỏi nó trả lời | Bản chất theo coupling/cohesion |
|---|---|---|
| SRP | Tách hay gộp module? | Cohesion: một trục thay đổi cho mỗi module |
| OCP | Biến thể mới đặt ở đâu? | Coupling: khung không biết chi tiết các biến thể |
| LSP | Implementation thế nào là hợp lệ? | Giữ cho abstraction thực sự cắt được coupling |
| ISP | Contract to cỡ nào? | Coupling: client chỉ biết đúng cái nó cần |
| DIP | Mũi tên phụ thuộc trỏ hướng nào? | Coupling trỏ về phía ổn định |

Cả năm là các góc chiếu của **một** nguyên lý: *quản lý xem ai được biết gì về ai, sao cho thay đổi được khoanh vùng*.

---

## 9. Anti-patterns khi áp dụng SOLID trong Go

**Interface ở khắp mọi nơi ("Java hóa Go").** Mỗi struct một interface `IXxxService` cùng file, một implementation duy nhất, mãi mãi. Chi phí: nhảy định nghĩa 2 bước, mock generation ồ ạt, đọc code như đọc qua sương mù — mà không mua được flexibility nào (biến thể không tồn tại). *Khắc phục:* chỉ trích interface khi (a) có ≥2 implementation thật, (b) cần cắt phụ thuộc cho unit test, hoặc (c) cần đảo chiều phụ thuộc qua ranh giới kiến trúc.

**SRP nghĩa đen — hàm 5 dòng, 40 file cho một feature.** Tách theo kích thước thay vì theo actor. Change amplification quay lại theo đường khác: một thay đổi nghiệp vụ giờ chạm 12 hàm tí hon. *Khắc phục:* đếm actor, không đếm dòng.

**OCP phòng ngừa — plugin system cho thứ không bao giờ có plugin thứ hai.** *Khắc phục:* rule of two — chờ biến thể thứ hai xuất hiện.

**LSP bằng type assertion.** `if pg, ok := repo.(*PostgresRepo); ok { pg.SpecialMethod() }` — thừa nhận abstraction đã chết nhưng vẫn giữ xác nó. *Khắc phục:* hoặc nâng method vào contract (nếu mọi impl cần), hoặc tách interface, hoặc bỏ interface.

**DIP nửa vời.** Có interface nhưng khai báo ở package hạ tầng (`postgres.Repository`), business import `postgres` để lấy interface → mũi tên vẫn trỏ sai hướng, chỉ thêm một tầng gián tiếp vô ích. *Khắc phục:* interface về phía consumer (chương 1.3 và 4).

---

## 10. Khi nào KHÔNG cần SOLID

Cùng logic với chương trước: SOLID là công cụ kiểm soát chi phí thay đổi, nên nó vô nghĩa khi không có thay đổi:

- Script/migration chạy một lần; prototype sẽ vứt.
- CRUD tool nội bộ nhỏ, một người maintain.
- Code ở tầng ngoài cùng vốn dĩ mỏng (main.go, wiring): main.go *phải* biết mọi thứ — đó là nhiệm vụ của nó, đừng "SOLID hóa" composition root.

Với các trường hợp này: hàm thẳng, struct cụ thể, zero interface là thiết kế đúng.

---

## Tóm tắt chương

- SOLID = coupling/cohesion đóng gói thành 5 quy tắc hành động.
- Trong Go: SRP đếm theo actor; OCP dùng interface + registry thay inheritance; LSP cần contract test; ISP gần miễn phí nhờ structural typing; DIP là nền của Clean Architecture.
- Nguy hiểm lớn nhất khi áp dụng SOLID vào Go không phải là thiếu mà là **thừa**: interface và tầng gián tiếp mọc trước nhu cầu thật.

**Chương tiếp theo:** [Dependency Inversion Principle — mổ xẻ đến tận gốc](/series/clean-architect/01-foundation/03-dependency-inversion/).
