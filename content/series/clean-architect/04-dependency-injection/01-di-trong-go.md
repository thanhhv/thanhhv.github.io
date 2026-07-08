+++
title = "Chương 4 — Dependency Injection trong Go: Manual, Wire, Fx và Interface Placement"
date = "2026-07-08T03:00:00+07:00"
draft = false
tags = ["backend", "golang", "clean-architecture"]
series = ["Clean Architecture với Golang"]
+++

> **Level 2–3** · DI là cơ chế *thi công* của Dependency Rule: mọi mũi tên đã đảo ở thiết kế phải được nối lại ở runtime — chương này bàn nối ở đâu, bằng gì.

---

## 1. Problem Statement

Sau khi áp dụng DIP, các component chỉ biết interface. Nhưng lúc chạy, ai đó phải quyết định `order.Service` dùng `postgres.WalletRepo` hay `mongo.WalletRepo`, và tạo chúng theo đúng thứ tự với đúng config. Câu hỏi của chương: **việc lắp ráp đó viết thế nào để (a) tường minh, (b) không rò rỉ kiến thức lắp ráp vào nghiệp vụ, (c) không thành 500 dòng spaghetti trong main.go khi hệ thống lớn?**

Nếu làm sai:

- Component **tự tạo** dependency (`NewService()` gọi `sql.Open` bên trong) → không thay thế được, không test được, config vương vãi — DIP bị vô hiệu ngay tại constructor.
- Dùng **global/singleton** (`db.Get()`, `config.Instance`) → common coupling, thứ tự khởi tạo mờ ám, race trong test song song.
- **Service locator** (`container.Get("orderRepo")`) → dependency graph vô hình, lỗi wiring chỉ lộ lúc runtime, "go to definition" chết.

## 2. Nguyên tắc nền: Constructor Injection + Composition Root

Go idiomatic chỉ cần hai quy ước:

```go
// 1. Mọi component nhận dependency qua constructor, lưu vào field unexported.
func NewRedeem(w WalletRepo, v VoucherIssuer, now func() time.Time) *Redeem {
	return &Redeem{wallets: w, vouchers: v, now: now}
}

// 2. Toàn bộ việc lắp ráp dồn về MỘT nơi: composition root (cmd/*/main.go).
```

Hệ quả tốt đẹp: dependency graph **hiện hình trong chữ ký hàm** — đọc constructor biết ngay component cần gì; compiler kiểm tra wiring (thiếu tham số = không compile); test tự lắp fake không cần nghi lễ.

Quy tắc phụ trợ:

- Constructor **không làm I/O, không side-effect** — chỉ gán field. Mở kết nối là việc của composition root.
- Nhận **interface**, trả **con trỏ struct cụ thể** ("accept interfaces, return structs").
- Dependency tùy chọn → functional options hoặc setter có default, đừng đục lỗ constructor 12 tham số:

```go
func NewServer(svc *order.Service, opts ...Option) *Server {
	s := &Server{svc: svc, timeout: 30 * time.Second, log: slog.Default()}
	for _, o := range opts { o(s) }
	return s
}
type Option func(*Server)
func WithTimeout(d time.Duration) Option { return func(s *Server) { s.timeout = d } }
```

## 3. Manual DI — mặc định đúng cho đa số hệ thống

Main.go của service cỡ vừa (~15–30 component) trông như sau, và **đây là code tốt, không phải code tạm**:

```go
func main() {
	cfg := config.MustLoad()
	logger := newLogger(cfg)

	// ---- hạ tầng (vòng 4) ----
	db := mustOpenPostgres(cfg.DatabaseURL)
	defer db.Close()
	kafkaWriter := kafka.NewWriter(cfg.KafkaBrokers)
	defer kafkaWriter.Close()

	// ---- adapters (vòng 3) ----
	walletRepo := postgres.NewWalletRepo(db)
	voucherCli := voucherapi.New(cfg.VoucherURL, httpClientWithRetry(cfg))
	publisher  := kafkapub.New(kafkaWriter)

	// ---- use cases (vòng 2) ----
	redeem := usecase.NewRedeem(walletRepo, voucherCli, publisher, time.Now)
	earn   := usecase.NewEarn(walletRepo, publisher, time.Now)

	// ---- delivery ----
	handler := httpapi.New(redeem, earn)
	runServer(cfg, handler, logger)
}
```

Mọi thứ tường minh: thứ tự đọc từ trên xuống là thứ tự tầng từ ngoài vào trong; muốn biết Redeem dùng repo nào — nhìn một dòng. Khi hệ thống lớn, chống phình bằng cách nhóm theo module thay vì đổi công cụ:

```go
// internal/loyalty/module.go — mỗi module tự lắp phần của mình
func NewModule(db *sql.DB, pub events.Publisher) *Module {
	repo := postgres.NewWalletRepo(db)
	return &Module{
		Redeem: usecase.NewRedeem(repo, ..., pub, time.Now),
		Earn:   usecase.NewEarn(repo, pub, time.Now),
	}
}
// main.go: loyaltyMod := loyalty.NewModule(db, pub); orderMod := order.NewModule(db, pub, loyaltyMod.Earn)
```

## 4. Wire — codegen khi graph thật sự lớn

[google/wire](https://github.com/google/wire) sinh code lắp ráp lúc **compile-time**: bạn khai báo các provider (constructor), Wire tự tính thứ tự và sinh ra chính xác đoạn main ở mục 3.

```go
//go:build wireinject
// wire.go
func InitializeApp(cfg config.Config) (*App, func(), error) {
	wire.Build(
		newPostgres, postgres.NewWalletRepo,
		voucherapi.New, kafkapub.New,
		usecase.NewRedeem, usecase.NewEarn,
		httpapi.New, newApp,
	)
	return nil, nil, nil // thân hàm bị thay bằng code sinh ra
}
```

`wire` gen ra `wire_gen.go` — code Go thường, đọc được, debug được. Lỗi wiring (thiếu provider, trùng kiểu) bắn ra **lúc build**, không phải lúc chạy.

**Được**: hết boilerplate nối dây thủ công khi graph > ~50 component; vẫn zero runtime cost, zero reflection; cleanup function tự sinh đúng thứ tự ngược.
**Mất**: bước codegen trong build; thông báo lỗi của Wire khó đọc; provider phân biệt bằng **kiểu** nên hai `*sql.DB` (primary/replica) phải bọc kiểu mới (`type ReplicaDB *sql.DB`); người mới phải học semantics của Wire thay vì đọc Go thẳng.

## 5. Fx — runtime container, dùng có điều kiện

[uber/fx](https://github.com/uber-go/fx) là DI container runtime dựa trên reflection, kèm quản lý lifecycle (OnStart/OnStop hook):

```go
fx.New(
	fx.Provide(newPostgres, postgres.NewWalletRepo, usecase.NewRedeem, httpapi.New),
	fx.Invoke(registerRoutes),
).Run()
```

**Được**: lifecycle chuẩn hóa (graceful start/stop từng component); module hóa (`fx.Module`) mạnh cho codebase rất lớn nhiều team chung quy ước (mô hình Uber); giảm hẳn boilerplate.
**Mất — và mất đáng kể**: lỗi wiring chỉ lộ **lúc runtime**; dependency graph vô hình với IDE (provide/inject nối nhau bằng reflection — "find usages" của constructor không cho biết ai dùng); stack trace lúc khởi động dài và lạ; toàn bộ app xoay quanh vòng đời của framework — chính là loại coupling framework mà Clean Architecture khuyên tránh, mỉa mai thay.

## 6. Chọn thế nào

| Bối cảnh | Khuyến nghị |
|---|---|
| Hầu hết service (< ~50 component) | **Manual DI** — tường minh là tính năng, không phải chi phí |
| Graph lớn, nhiều binary chung provider, team chịu codegen | Wire |
| Tổ chức rất lớn, chuẩn hóa cưỡng bức đa team, cần lifecycle framework | Fx (chấp nhận trade-off runtime) |

Nguyên tắc: **framework DI giải bài toán *quy mô của việc nối dây*, không giải bài toán thiết kế.** Nếu manual DI của bạn rối, nguyên nhân thường là dependency graph rối (component biết quá nhiều) — framework chỉ giấu triệu chứng. Sửa thiết kế trước, đổi công cụ sau. Và một điểm quan trọng: **cả ba cách cho ra cùng một kiến trúc** — component code không đổi một dòng khi chuyển giữa manual/Wire/Fx. Đó là dấu hiệu bạn đã làm đúng: DI mechanism là chi tiết của vòng 4.

## 7. Interface Placement — tổng kết quy tắc

Chủ đề đã chạm ở 1.3 và 6, chốt lại thành bảng tra:

| Tình huống | Interface đặt ở | Lý do |
|---|---|---|
| Use case cần repo/gateway | Package use case (consumer) | DIP: nhu cầu thuộc về người dùng nó |
| Nhiều module cần chung một contract nghiệp vụ (hiếm) | Package domain chung của bounded context | Contract là khái niệm nghiệp vụ chung |
| Thư viện public cho người ngoài | Cạnh implementation, interface **nhỏ** cho điểm mở rộng | Người ngoài không sửa được code bạn để thêm interface |
| Chỉ để mock trong test của chính package | Khai trong file test hoặc không cần (dùng fake struct) | Không bắt production code trả phí cho test |

Và quy tắc âm: **đừng tạo interface khi chưa có consumer thứ hai hoặc nhu cầu fake** — interface đẻ trước nhu cầu hầu như luôn sai hình dạng khi nhu cầu thật xuất hiện.

## 8. Anti-patterns

- **Constructor gọi `sql.Open`/`config.Load`**: component tự tìm dependency = service locator trá hình. Mọi thứ đi qua tham số.
- **Inject `*Container` / `*App` to đùng**: `NewService(app *App)` — nhận cả thế giới để dùng 2 thứ; stamp coupling, không biết service thật sự cần gì. Inject đúng từng dependency.
- **Global `var DB *sql.DB` + hàm `init()`**: thứ tự init phụ thuộc thứ tự import — bug kinh điển; test song song đạp nhau. Cấm `init()` có side-effect I/O.
- **Interface cho mọi thứ để "dễ inject"**: DI hoạt động tốt với kiểu cụ thể; chỉ cần interface tại ranh giới cần thay thế/fake.
- **Hai container**: nửa app dùng Fx, nửa dùng manual vì "đang chuyển đổi" ba năm chưa xong — tệ hơn cả hai lựa chọn.

## Tóm tắt

- DI trong Go = constructor injection + composition root; đó là toàn bộ "framework" đa số hệ thống cần.
- Wire khi graph lớn (lỗi compile-time), Fx khi tổ chức lớn chấp nhận runtime container; cả ba không thay đổi kiến trúc.
- Interface theo consumer, đẻ theo nhu cầu thật — không theo lễ nghi.

**Chương tiếp theo:** [Data Access — Repository, Transaction, Unit of Work](/series/clean-architect/05-data-access/01-repository-transaction/)
