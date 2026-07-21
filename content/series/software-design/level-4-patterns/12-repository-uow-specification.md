+++
title = "4.12 — Repository, Unit of Work & Specification: bộ ba Enterprise Pattern quanh dữ liệu"
date = "2026-07-17T12:00:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

Ba pattern từ danh mục **Enterprise** (Fowler, *Patterns of Enterprise Application Architecture*, 2002 — không phải GoF): sinh ra cho bài toán mà GoF chưa chạm tới — **ranh giới giữa mô hình nghiệp vụ trong RAM và cơ sở dữ liệu**. Cũng là bộ ba bị cargo-cult nhiều bậc nhất trong thế giới backend; chương này dựng chúng từ bài toán thật và vạch rõ ngưỡng lạm dụng.

---

## A. Repository

### 1. Bài toán — đã gặp, giờ gọi đúng tên

Toàn bộ Level 2 đã xây nó từng viên gạch: `ProductSource` của pricing (2.5), `FileStore` với contract test (2.3), `UserGetter` phía consumer (2.4). **Repository = tổng hợp các viên gạch đó thành pattern có tên**: một interface *nói ngôn ngữ domain* đứng giữa nghiệp vụ và persistence, cho nghiệp vụ ảo tưởng có kiểm soát rằng aggregate nằm trong một **collection trong RAM**:

```go
// package order (domain) — interface theo nhu cầu nghiệp vụ, không theo SQL
type Repository interface {
    ByID(ctx context.Context, id OrderID) (*Order, error)          // ErrNotFound theo hợp đồng 2.3
    Save(ctx context.Context, o *Order) error                       // insert-or-update, caller không cần biết
    PendingOlderThan(ctx context.Context, t time.Time) ([]*Order, error)  // câu hỏi NGHIỆP VỤ có tên
}
```

Ba giá trị, đều đã chứng minh ở Level 2: test nghiệp vụ bằng fake in-memory (mà fake phải qua **contract test** — 2.3); đổi ORM/DB chỉ đụng adapter; và — ít được nhắc nhưng giá trị nhất hằng ngày — **query có tên nghiệp vụ**: `PendingOlderThan` thay vì chuỗi `WHERE status = ? AND created_at < ?` lặp rải rác (mỗi query method là một tri thức nghiệp vụ có nhà — DRY đúng nghĩa 2.6).

### 2. Ba đường ranh quyết định chất lượng một Repository

**(a) Trả về domain object, không phải row.** `Repository` trả `*order.Order` (đã qua constructor, invariant nguyên vẹn — 1.2); struct có tag `gorm:"column:..."` là **của adapter**, dịch ở biên (đã thấy `row.toDomain()` ở 2.5). Trộn hai thứ — domain struct đeo tag ORM — là coupling domain vào schema: đổi cột phải mở domain, và ORM "tiện tay" expose khả năng mutate vòng qua invariant.

**(b) Interface theo aggregate, không theo bảng.** Repository cho `Order` (gồm cả items, payment info — một *cụm nhất quán*), không phải `OrderRepo` + `OrderItemRepo` + `OrderPaymentRepo` mỗi bảng một cái — chẻ theo bảng là để schema DB quyết định hình dạng tầng nghiệp vụ (mũi tên ngược chiều, 2.5). Save cả aggregate trong một transaction là việc *bên trong* adapter.

**(c) Đừng hứa cái không giữ được.** Nhắc từ 2.5 vì đây là phê phán *đúng* nhất nhằm vào pattern này: query phức tạp (join báo cáo, full-text, aggregate thống kê) **rò qua** mọi interface trừu tượng. Lời giải trung thực: repository cho **đường ghi và đọc-theo-nghiệp-vụ của aggregate**; đường đọc phức tạp cho hiển thị/báo cáo đi cổng riêng (query service đọc thẳng SQL trả DTO phẳng — mầm CQRS, Level 5). Một interface cố che *mọi* truy vấn sẽ phình thành 40 method (ISP chết) hoặc mọc `FindWhere(sql string)` (leaky, 1.2).

### 3. Khi nào KHÔNG cần

Service CRUD mỏng không có nghiệp vụ giữa HTTP và DB (2.5 đã nói): handler gọi thẳng sqlc/GORM là kiến trúc *đúng* — repository ở đó chỉ photocopy chữ ký (Middle Man, 3.2). Và **sqlc đáng nói riêng** vì nó đổi thế cân bằng trong Go: viết SQL thật → sinh code type-safe → cái "adapter" gần như tự viết; nhiều team dùng luôn interface sinh bởi sqlc làm repository — thực dụng và đủ, miễn tầng domain không import package sinh mã đó trực tiếp khi có nghiệp vụ cần bảo vệ.

---

## B. Unit of Work

### 1. Bài toán — nhiều thay đổi, một số phận

Use case "hoàn tất đơn" (4.4): trừ kho + tạo vận đơn + ghi payment — **cùng thành hoặc cùng bại**. Mỗi repository tự `Save` với connection riêng thì crash giữa chừng để lại dữ liệu xé đôi. Cần một *đơn vị công việc*: gom mọi thay đổi trong phiên, commit một lần.

**Định nghĩa (Fowler)**: object theo dõi mọi thay đổi trong một business transaction và điều phối việc ghi ra DB trong một lần commit. Trong thế giới ORM đầy đủ (Hibernate, EF Core), UoW là *cỗ máy thật*: session theo dõi object bẩn (dirty tracking), tự sinh SQL, tự sắp thứ tự ghi. **Trong Go — không có ORM theo dõi thay đổi kiểu đó — UoW co lại thành đúng phần giá trị nhất: ranh giới transaction tường minh cắt ngang nhiều repository.**

### 2. Hình dạng Go — hai mức, chọn theo độ phức tạp

```go
// Mức 1 — hàm transaction bao lấy công việc (đủ cho đa số):
// package postgres (adapter) — domain KHÔNG biết *sql.Tx
func (s *Store) WithinTx(ctx context.Context, fn func(r order.Repos) error) error {
    tx, err := s.db.BeginTx(ctx, nil)
    if err != nil { return err }
    defer tx.Rollback()                          // no-op nếu đã commit

    if err := fn(s.reposWith(tx)); err != nil {  // repos BUỘC vào tx này
        return err                                // rollback qua defer
    }
    return tx.Commit()
}

// Use case — ranh giới "cùng số phận" đọc được bằng mắt:
err := store.WithinTx(ctx, func(r order.Repos) error {
    if err := r.Inventory.Reserve(ctx, items); err != nil { return err }
    if err := r.Shipping.Create(ctx, shipment); err != nil { return err }
    return r.Payments.Record(ctx, payment)
})
```

Đã hé lộ ở 2.5 ("interface `WithinTx` — vẫn ổn nhưng không còn thuần khiết") — giờ nói đủ: đây là chỗ trừu tượng hóa persistence *rò có kiểm soát*: domain biết khái niệm "các bước này chung số phận" (một khái niệm *nghiệp vụ* chính đáng!) nhưng không biết `*sql.Tx`. Mức 2 — struct UoW giữ danh sách thay đổi rồi `Commit()` — chỉ đáng khi cần gom event domain phát-sau-commit (4.13 Outbox sẽ cần đúng chỗ này).

Bẫy Go đặc thù, gặp ở mọi codebase: **truyền tx qua context** (`ctx = context.WithValue(ctx, txKey, tx)`, repository tự móc ra) — tiện nhưng là dependency tàng hình nguyên chất (4.5, 4.11): chữ ký không nói hàm chạy trong transaction hay không, quên bọc là chạy ngoài tx *im lặng*. `WithinTx` với repos-buộc-tx làm điều đó **không thể sai kiểu** — invalid state unrepresentable (1.3) áp cho transaction.

### 3. Khi nào KHÔNG cần

Use case chạm đúng một aggregate → transaction nằm gọn trong `repo.Save` — UoW là nghi lễ. Nhu cầu "cùng số phận" xuyên *service* (không cùng DB) → không transaction nào cứu — đó là bài toán Saga (4.13), đừng cố kéo UoW qua network.

---

## C. Specification

### 1. Bài toán — rule chọn-lọc bị xé đôi

Rule "đơn đủ điều kiện freeship": dùng ở **hai nơi bản chất khác nhau** — (a) check một đơn trong RAM (`if eligible(o)`), (b) *tìm mọi đơn* thỏa mãn trong DB (`WHERE ...`). Viết hai bản là Duplicate Code cấp 3 (3.2 — trùng tri thức, khác hình thức); đổi ngưỡng sửa một quên một.

**Specification** (Evans & Fowler): đóng gói một *predicate nghiệp vụ* thành object, compose được (AND/OR/NOT — chính là Composite trên rule, đã chỉ ở 4.6), và — tham vọng nhất — *một* định nghĩa dùng cho cả check-trong-RAM lẫn sinh-query.

```go
type Spec[T any] interface {
    IsSatisfiedBy(T) bool
}
func And[T any](specs ...Spec[T]) Spec[T] { /* ... */ }

// Rule nghiệp vụ có tên, có nhà, test bằng bảng:
var FreeshipEligible = And(MinTotal(500_000), NotStatus(Cancelled), InnerCity())
```

### 2. Lời khuyên trung thực — nửa nào đáng lấy

Nửa **in-memory composable rules**: rẻ, sáng, đáng dùng ngay khi rule đủ nhiều và cần tổ hợp (rule engine khuyến mãi, điều kiện duyệt, policy check) — như 4.6 đã nói, đây là chỗ dev backend gặp Composite nhiều nhất.

Nửa **dịch spec thành SQL**: đắt gấp nhiều lần vẻ ngoài — bạn đang viết một **query compiler mini** (duyệt cây spec sinh WHERE + tham số, xử lý index, N+1, dialect...). ORM lớn làm sẵn (Hibernate Criteria, EF Core expression tree — cả một tầng compiler của framework); trong Go, tự viết cho vài rule là over-engineering gần như chắc chắn. Lối thực dụng: spec cho RAM; còn DB thì mỗi spec *tương ứng* một query method có tên trong repository (`FreeshipEligibleOrders(ctx)`) — trùng tri thức được **ghi sổ** (3.2: bản sao có kiểm soát, đặt cạnh nhau + test đối chiếu hai bản cùng bộ dữ liệu) — 90% giá trị, 10% chi phí.

---

## Tổng kết — bộ ba trên một trục

```
Specification: RULE chọn cái gì      (tri thức nghiệp vụ dạng predicate)
Repository:    CỔNG lấy/cất aggregate (dịch domain ↔ persistence)
Unit of Work:  RANH GIỚI số phận      (những thay đổi nào sống chết cùng nhau)
```

Ba pattern ăn khớp: use case mở UoW → qua repository lấy aggregate (lọc bằng spec/query-có-tên) → gọi hành vi domain → save trong cùng UoW → commit. Đó chính là giải phẫu một **application service** chuẩn — và là bộ khung mà Level 5 (Clean/Hexagonal, CQRS) sẽ đặt tên tầng cho từng mảnh. Nhưng giữ chân trên đất: cả ba đều là *công cụ cho nghiệp vụ đủ dày* — service mỏng dùng cả ba là mặc đồ giáp đi mua rau; các mục "khi nào KHÔNG" ở trên là một nửa giá trị của chương này.

---

*Tiếp theo: [4.13 — Pipeline, Plugin, Domain Events, Saga & Outbox](/series/software-design/level-4-patterns/13-pipeline-plugin-events-saga-outbox/)*
