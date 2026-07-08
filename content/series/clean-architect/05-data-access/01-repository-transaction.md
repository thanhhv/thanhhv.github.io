+++
title = "Chương 5 — Data Access: Repository, Transaction, Unit of Work, ORM vs SQL"
date = "2026-07-08T04:00:00+07:00"
draft = false
tags = ["backend", "golang", "clean-architecture"]
series = ["Clean Architecture với Golang"]
+++

> **Level 3 – Senior** · Data access là nơi Clean Architecture bị thử thách gay gắt nhất trong thực tế — transaction, N+1, mapping — và là nơi nhiều dự án gãy.

---

## 1. Problem Statement

Ba bài toán mà mọi hệ thống phải giải, và giải sai thì Dependency Rule sụp:

1. **Nghiệp vụ cần dữ liệu nhưng không được biết SQL** — ai đứng giữa? (Repository)
2. **Một use case ghi nhiều bảng phải atomic** — transaction là khái niệm hạ tầng, sao use case điều khiển được nó mà không import `database/sql`? (bài toán khó nhất chương)
3. **Model của DB và model của domain khác nhau đến đâu thì tách?** (ORM vs SQL, mapping)

## 2. Repository Pattern — bản chất thật

Repository **không phải** "class bọc mấy câu SQL". Định nghĩa gốc (Evans/Fowler): *ảo giác một collection trong bộ nhớ chứa các domain object* — nghiệp vụ "lấy ra, thao tác, đặt lại", không biết đằng sau là Postgres hay RAM.

Ba đặc trưng phân biệt repository thật với wrapper ORM:

- **Nhận/trả kiểu domain**, không phải row/model DB. Mapping là việc nội bộ của nó.
- **Interface do vòng trong sở hữu**, hình dạng theo *nhu cầu use case* (`ByCustomer`, `Save`) chứ không theo *năng lực SQL* (`FindWhere(cond string)` — rò rỉ query language qua ranh giới).
- **Một repository cho một aggregate** (cụm object nhất quán cùng nhau — chương 9), không phải một bảng. Repository cho `Order` lưu cả `order_items` — vì domain không có khái niệm "lưu riêng item".

Ví dụ đầy đủ đã có ở chương 2.3 (WalletRepo). Điểm bổ sung — **query phức tạp cho hiển thị không cần đi qua domain model**: đó là lối vào CQRS (chương 10); đừng ép mọi SELECT thống kê phải dựng aggregate.

## 3. Transaction — bài toán khó nhất

Use case "chuyển điểm giữa hai ví" phải trừ ví A, cộng ví B atomic. Transaction thuộc vòng 4, use case ở vòng 2. Ba lời giải theo mức độ nghi thức:

### Cách 1 — Transaction trong repository (đủ cho đa số)

Nếu mọi thao tác atomic gói được trong **một** lời gọi repository, transaction là chi tiết nội bộ của adapter:

```go
// Interface (vòng trong) — atomic là NGỮ NGHĨA của contract, cách thực hiện là việc của adapter
type WalletRepo interface {
	TransferPoints(ctx context.Context, from, to string, amount domain.Points) error
}

// postgres (vòng ngoài)
func (r *WalletRepo) TransferPoints(ctx context.Context, from, to string, amount domain.Points) error {
	tx, err := r.db.BeginTx(ctx, nil)
	if err != nil { return err }
	defer tx.Rollback()
	// ... UPDATE 2 ví với kiểm tra balance, khóa theo thứ tự ID tránh deadlock
	return tx.Commit()
}
```

**Giới hạn**: khi rule nghiệp vụ phải chạy *giữa* các bước ghi (đọc → tính bằng entity → ghi), nhét cả quy trình vào repository là chuyển business logic xuống vòng 4 — anti-pattern.

### Cách 2 — Unit of Work / Transactor (khuyến nghị cho use case đa bước)

Vòng trong khai báo một cổng trừu tượng "chạy khối lệnh sau một cách atomic":

```go
// usecase/ports.go — vòng trong, không biết SQL
type Transactor interface {
	// WithinTx chạy fn; mọi thao tác repo trong fn dùng CHUNG một transaction
	// (gắn qua ctx). fn trả lỗi → rollback; nil → commit.
	WithinTx(ctx context.Context, fn func(ctx context.Context) error) error
}
```

```go
// usecase/transfer.go
func (uc *Transfer) Execute(ctx context.Context, in TransferInput) error {
	return uc.tx.WithinTx(ctx, func(ctx context.Context) error {
		src, err := uc.wallets.ByCustomer(ctx, in.From)
		if err != nil { return err }
		dst, err := uc.wallets.ByCustomer(ctx, in.To)
		if err != nil { return err }

		src, err = src.Withdraw(in.Amount)      // domain rules chạy GIỮA các bước I/O
		if err != nil { return err }
		dst = dst.Deposit(in.Amount)

		if err := uc.wallets.Save(ctx, src); err != nil { return err }
		return uc.wallets.Save(ctx, dst)
	})
}
```

Adapter Postgres gắn `*sql.Tx` vào context — kỹ thuật chuẩn trong Go:

```go
// adapter/postgres/transactor.go
type ctxKey struct{}

type Transactor struct{ db *sql.DB }

func (t *Transactor) WithinTx(ctx context.Context, fn func(context.Context) error) error {
	tx, err := t.db.BeginTx(ctx, nil)
	if err != nil { return fmt.Errorf("begin tx: %w", err) }
	defer tx.Rollback() // no-op nếu đã commit

	if err := fn(context.WithValue(ctx, ctxKey{}, tx)); err != nil {
		return err
	}
	return tx.Commit()
}

// Mọi repo Postgres đọc executor từ ctx: có tx dùng tx, không thì dùng pool.
type executor interface {
	QueryRowContext(context.Context, string, ...any) *sql.Row
	ExecContext(context.Context, string, ...any) (sql.Result, error)
}

func (r *WalletRepo) exec(ctx context.Context) executor {
	if tx, ok := ctx.Value(ctxKey{}).(*sql.Tx); ok { return tx }
	return r.db
}

func (r *WalletRepo) Save(ctx context.Context, w domain.Wallet) error {
	_, err := r.exec(ctx).ExecContext(ctx, `INSERT ... ON CONFLICT ...`, ...)
	return err
}
```

**Đánh giá trung thực**: use case giờ biết *khái niệm* "khối atomic" — một rò rỉ trừu tượng có kiểm soát (RAM fake của bạn cũng phải giả lập ngữ nghĩa rollback nếu test kỹ). Nhưng "quy trình này phải nhất quán" thực chất **là một yêu cầu nghiệp vụ**, nên để nó hiện diện ở use case là chấp nhận được và thực dụng. Giấu tx trong `ctx` bị phê là "ẩn" — đúng, đổi lại chữ ký repo không nhiễm `*sql.Tx`; đây là trade-off được đa số codebase Go trưởng thành chấp nhận (pattern này xuất hiện trong nhiều production codebase lớn).

### Cách 3 — Domain events + eventual consistency

Khi hai thứ cần nhất quán nằm ở hai module/service khác nhau, transaction chung là không thể hoặc không nên. Giải bằng outbox + event (chương 7, 11). Quy tắc chọn: **trong một aggregate — strong consistency (cách 1/2); giữa các aggregate/module — eventual (cách 3)**.

## 4. ORM vs SQL thuần trong Go

| Trục | SQL thuần (`database/sql` + `sqlx`/`pgx`) | `sqlc` (codegen từ SQL) | GORM/Ent (ORM đầy đủ) |
|---|---|---|---|
| Kiểm soát query | Tuyệt đối | Tuyệt đối (SQL là nguồn) | Mờ — ORM sinh query |
| Type safety | Scan tay, lỗi runtime | ✅ compile-time | Một phần (Ent tốt hơn GORM) |
| Tốc độ viết CRUD | Chậm | Nhanh sau khi setup | Nhanh nhất ban đầu |
| Nguy cơ rò vào domain | Thấp (kỷ luật mapping) | Thấp — struct sinh ra nằm ở adapter | **Cao**: struct tag, hook, lazy load kéo domain dính ORM |
| N+1, query bất ngờ | Không (bạn viết gì chạy nấy) | Không | Có, kinh điển |
| Migration schema phức tạp | Bạn tự chủ | Bạn tự chủ | Vật lộn với ORM |

Khuyến nghị production cho hệ thống có domain thật: **`sqlc` hoặc `pgx` + mapping tay tại repository**. Điểm mấu chốt kiến trúc: dù chọn gì, **model của ORM/sqlc là DTO của vòng 4** — `RehydrateWallet(row...)` dịch nó sang domain. Khoảnh khắc entity của bạn mọc tag `gorm:"..."` là khoảnh khắc schema DB bắt đầu định hình domain — đảo ngược đúng thứ Clean Architecture bảo vệ.

Với MongoDB: cùng nguyên tắc — document struct (bson tag) sống ở `adapter/mongo`, dịch sang domain tại repo. Mongo "schema-less" không miễn trừ mapping; nó chỉ chuyển chi phí từ migration sang version-handling trong code đọc.

## 5. Testing repository

Repository là code **không nên unit test bằng mock** — giá trị của nó nằm chính ở SQL đúng và mapping đúng, thứ chỉ kiểm chứng được với DB thật. Chiến lược ba tầng (chi tiết chương 8):

- Unit test domain/use case: fake in-memory repo (RAM map) — mili-giây.
- **Integration test repository: testcontainers-go** chạy Postgres thật trong Docker + chạy migration + contract test dùng chung (chương 1.2 mục LSP) cho mọi implementation.
- Không test: getter/setter, mapping hiển nhiên.

```go
func TestWalletRepo(t *testing.T) {
	ctx := context.Background()
	pgc, err := tcpostgres.Run(ctx, "postgres:16-alpine",
		tcpostgres.WithDatabase("test"),
		tcpostgres.BasicWaitStrategies())
	if err != nil { t.Fatal(err) }
	t.Cleanup(func() { pgc.Terminate(ctx) })

	db := mustOpen(t, pgc.MustConnectionString(ctx, "sslmode=disable"))
	mustMigrate(t, db)

	RunWalletRepoContract(t, func(t *testing.T) usecase.WalletRepo {
		truncateAll(t, db)
		return postgres.NewWalletRepo(db)
	})
}
```

## 6. Anti-patterns

- **Generic repository**: `Repository[T any]{ GetByID, GetAll, Save, Delete }` cho mọi entity — ép mọi aggregate vào 4 động từ CRUD, rồi mọc `GetAllWithFilter(filter map[string]any)` — SQL injection bằng cửa sau và contract vô nghĩa. Repository theo nhu cầu use case, mỗi aggregate một hình dạng riêng.
- **Repository trả `*gorm.DB` / query builder** để "linh hoạt" — toàn bộ tầng trên giờ viết query, ranh giới chết.
- **Business logic trong repository**: `SaveOrder` tự tính discount "tiện thể" — rule tàng hình khỏi domain, không unit test được. Repository chỉ đọc/ghi.
- **God repository**: `UserRepo` 40 method phục vụ 15 use case. Tách interface theo consumer (ISP); implementation có thể vẫn một struct.
- **Transaction rò lên delivery**: handler mở tx rồi truyền xuống — HTTP layer quyết định ranh giới nhất quán của nghiệp vụ. Ranh giới tx thuộc use case (cách 2).
- **Test repository bằng sqlmock**: khẳng định chuỗi SQL khớp từng ký tự — test vỡ khi đổi format câu SQL dù hành vi đúng, và không phát hiện SQL sai logic. Dùng DB thật trong container.

## 7. Khi nào đơn giản hóa

CRUD tool nội bộ, service đọc-ghi thẳng không rule: handler → `sqlc` query trực tiếp, bỏ repository interface, bỏ transactor — hoàn toàn hợp lệ. Repository + UoW là chi phí mua khả năng test nghiệp vụ độc lập; nghiệp vụ không có gì đáng test thì đừng trả phí.

## Tóm tắt

- Repository = ảo giác collection domain object; interface theo use case, mapping ở adapter, một repo một aggregate.
- Transaction: gói trong repo khi một lời gọi là đủ; Transactor + ctx cho quy trình đa bước; event/outbox giữa các module.
- ORM model/sqlc struct là DTO vòng 4 — không bao giờ là domain entity.
- Test repo bằng DB thật (testcontainers), không mock SQL.

**Chương tiếp theo:** [Delivery Layer — HTTP, gRPC, GraphQL, CLI, Worker](/series/clean-architect/06-delivery/01-delivery-layer/)
