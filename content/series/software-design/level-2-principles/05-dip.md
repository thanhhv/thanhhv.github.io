+++
title = "2.5 — Dependency Inversion Principle: đảo chiều phụ thuộc"
date = "2026-07-17T08:50:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## 1. Problem Statement

DIP là nguyên tắc **kiến trúc** nhất trong SOLID — nó không nói về một class hay một interface, mà về **chiều của các mũi tên trong toàn hệ thống**. Và nó là nền móng trực tiếp của Clean Architecture, Hexagonal Architecture (Level 5).

Phát biểu gốc gồm hai vế:

> 1. High-level modules không nên phụ thuộc low-level modules. Cả hai nên phụ thuộc abstraction.
> 2. Abstraction không nên phụ thuộc chi tiết. Chi tiết nên phụ thuộc abstraction.

"High-level" = chính sách, nghiệp vụ, lý do phần mềm tồn tại ("đơn hàng được xác nhận thì trừ kho và báo khách"). "Low-level" = cơ chế thực thi (Postgres, Kafka, SendGrid, HTTP). Trực giác nền (chương 1.3): **mũi tên phải trỏ từ thứ hay đổi về thứ ổn định** — và nghiệp vụ ổn định hơn hạ tầng: công ty đổi Redis sang Memcached, đổi REST sang gRPC thường xuyên hơn nhiều so với đổi "đơn xác nhận thì trừ kho".

Luồng phụ thuộc "tự nhiên" (không can thiệp) luôn trỏ SAI chiều: code nghiệp vụ gọi `postgres.Query`, nên nghiệp vụ *import* postgres — chính sách quý giá nhất phụ thuộc chi tiết dễ thay nhất. DIP là hành động **bẻ ngược** mũi tên đó — vì thế mới gọi là *inversion*.

## 2. Code Smell

```go
// ❌ package tính giá — trái tim nghiệp vụ — import 2 hạ tầng
package pricing

import (
    "github.com/redis/go-redis/v9"
    "gorm.io/gorm"
)

type Engine struct {
    db  *gorm.DB
    rdb *redis.Client
}

func (e *Engine) PriceFor(ctx context.Context, sku string, qty int) (int64, error) {
    // cache check lẫn vào giữa logic giá
    if v, err := e.rdb.Get(ctx, "price:"+sku).Int64(); err == nil {
        return v * int64(qty), nil
    }
    var p Product
    if err := e.db.WithContext(ctx).First(&p, "sku = ?", sku).Error; err != nil {
        return 0, err
    }
    price := p.Base
    if qty >= 100 { price = price * 90 / 100 }     // nghiệp vụ thật sự nằm ở 2 dòng
    if p.Clearance { price = price * 70 / 100 }    // giữa rừng hạ tầng
    e.rdb.Set(ctx, "price:"+sku, price, time.Hour)
    return price * int64(qty), nil
}
```

Chi phí cụ thể: test logic chiết khấu cần Postgres + Redis chạy (CI chậm, flaky); nâng major version GORM → mở package nghiệp vụ ra sửa; logic giá muốn tái sử dụng cho batch job → kéo theo cả Redis dù batch không cần cache; và sơ đồ import nói rằng *"pricing được xây trên GORM"* — một câu kiến trúc không ai chủ đích chọn, nó chỉ *xảy ra*.

## 3. First Principles — cơ chế của phép đảo

Điều kỳ diệu đáng hiểu rõ: **luồng gọi lúc runtime giữ nguyên** (pricing vẫn gọi xuống DB), nhưng **chiều phụ thuộc lúc compile đảo lại**. Cơ chế: chèn một interface, và — điểm quyết định — **đặt interface đó ở package nghiệp vụ**:

```
Trước:  pricing ──import──► gorm/redis        (nghiệp vụ quỳ trước hạ tầng)

Sau:    pricing ──định nghĩa──► ProductSource (interface, SỞ HỮU bởi pricing)
                                    ▲
        postgresadapter ──import────┘         (hạ tầng quỳ trước nghiệp vụ)
```

Nơi đặt interface là toàn bộ linh hồn của DIP: interface đặt cạnh implementation (package `postgresadapter.ProductSource`) thì nghiệp vụ vẫn phải import hạ tầng — mũi tên **chưa hề đảo**, bạn chỉ thêm một lớp gián tiếp vô ích. *Abstraction thuộc về phía chính sách* — nó là "nhu cầu của nghiệp vụ được phát biểu bằng interface", trùng khớp với consumer-side interface (1.5) và ISP (2.4). Trong Go, implicit interface làm bước này không cần hạ tầng biết gì về nghiệp vụ.

## 4. Refactoring Journey

**Bước 1 — nghiệp vụ tự phát biểu nhu cầu:**

```go
// ✅ package pricing — import CHỈ stdlib
package pricing

type ProductSource interface {
    BySKU(ctx context.Context, sku string) (Product, error)
}
type PriceCache interface {
    Get(ctx context.Context, sku string) (int64, bool)
    Set(ctx context.Context, sku string, price int64)
}

type Engine struct {
    products ProductSource
    cache    PriceCache
}

func NewEngine(ps ProductSource, c PriceCache) *Engine {
    return &Engine{products: ps, cache: c}
}

func (e *Engine) PriceFor(ctx context.Context, sku string, qty int) (int64, error) {
    if v, ok := e.cache.Get(ctx, sku); ok {
        return v * int64(qty), nil
    }
    p, err := e.products.BySKU(ctx, sku)
    if err != nil {
        return 0, fmt.Errorf("pricing: load %s: %w", sku, err)
    }
    price := p.Base
    if qty >= 100 { price = price * 90 / 100 }
    if p.Clearance { price = price * 70 / 100 }
    e.cache.Set(ctx, sku, price)
    return price * int64(qty), nil
}
```

**Bước 2 — hạ tầng thành adapter, phụ thuộc vào nghiệp vụ:**

```go
// package postgresstore — import gorm VÀ pricing (mũi tên đã đảo)
type ProductStore struct{ db *gorm.DB }

func (s *ProductStore) BySKU(ctx context.Context, sku string) (pricing.Product, error) {
    var row productRow                       // struct DB riêng, có tag gorm
    if err := s.db.WithContext(ctx).First(&row, "sku = ?", sku).Error; err != nil {
        return pricing.Product{}, err
    }
    return row.toDomain(), nil               // dịch DB model → domain model
}
```

**Bước 3 — composition root: nơi DUY NHẤT mọi thứ gặp nhau:**

```go
// main.go — biết tất cả, và là NƠI DUY NHẤT biết tất cả
func main() {
    db := mustOpenGorm(cfg.DSN)
    rdb := redis.NewClient(&redis.Options{Addr: cfg.RedisAddr})

    engine := pricing.NewEngine(
        &postgresstore.ProductStore{DB: db},
        &redisstore.PriceCache{RDB: rdb},
    )
    api := httpapi.New(engine)
    log.Fatal(http.ListenAndServe(":8080", api))
}
```

Đây là **Dependency Injection thủ công** — và với đa số service Go, nó là đủ: tường minh, compile-time check, không magic, đọc `main` là thấy toàn bộ wiring của hệ thống. DI *framework* (Uber Fx, Wire, NestJS container) chỉ đáng giá khi graph hàng trăm node — phân tích ở Level 4. Nhớ phân biệt: **DIP là nguyên tắc về chiều mũi tên; DI chỉ là kỹ thuật truyền dependency; DI container là công cụ tùy chọn cho DI.** Dùng NestJS container cả ngày vẫn có thể vi phạm DIP nếu service inject thẳng `TypeOrmRepository`.

Thành quả: test logic giá bằng fake in-memory — micro giây, không container; đổi GORM→sqlx/Ent chỉ đụng `postgresstore`; và kiến trúc **được thi hành bằng compiler** — ai vô tình import gorm vào pricing sẽ bị review chặn ngay tại khối import (khóa cứng thêm bằng linter `depguard` nếu muốn).

**Node.js/TS đối chiếu**: không sửa được "chiều import" bằng structural typing thuần vì thói quen `import { PrismaService }` thẳng vào service quá tiện. Kỷ luật tương đương: khai `interface ProductSource` trong module domain, implement ở `infrastructure/`, bind ở module root (`useClass`/`useFactory` của NestJS, hoặc hàm factory thuần). Cùng nguyên tắc, công cụ khác.

## 5. Trade-off — nói thẳng phần đắt

- **Mỗi ranh giới đảo chiều = interface + adapter + model dịch (row→domain)**: thêm file, thêm code "vận chuyển". Với service CRUD mỏng — đọc DB, trả JSON, không có nghiệp vụ gì giữa hai đầu — DIP đầy đủ là nghi lễ: bạn viết adapter dịch struct sang struct giống hệt. **Độ dày của tầng nghiệp vụ quyết định mức đầu tư DIP.**
- **Trừu tượng hóa DB có giới hạn thật**: transaction xuyên nhiều repository, query phức tạp, bulk operation — những thứ này *rò* qua interface trừu tượng (bạn sẽ gặp interface `WithinTx(func(...) error)` — vẫn ổn nhưng không còn "thuần khiết"). Chấp nhận: DIP ở ranh giới hạ tầng là **giảm mạnh** coupling, không phải xóa tuyệt đối.
- **Chỗ nào đáng đảo, chỗ nào không**: đảo ở ranh giới *ổn định-về-nhu-cầu nhưng hay-đổi-về-cơ-chế* (storage, messaging, notification, payment gateway — nhu cầu bất biến, nhà cung cấp hay đổi). Đừng đảo quanh stdlib (`time`, `json`) hay quanh thứ cả đời không đổi — interface `Clock` chỉ đáng khi bạn thật sự cần test thời gian.

## 6. Production Examples

- **`database/sql`**: DIP hai tầng mẫu mực — app phụ thuộc `sql.DB` (abstraction), driver phụ thuộc `driver.Driver` (abstraction); không ai phụ thuộc ai trực tiếp. Nghiệp vụ không biết Postgres, Postgres driver không biết nghiệp vụ.
- **`http.Handler`**: bạn viết handler phụ thuộc interface do *stdlib* định nghĩa; server gọi ngược vào code bạn — inversion of control cùng họ hàng: framework gọi bạn, bạn không gọi framework.
- **Kubernetes CSI/CNI/CRI**: nhu cầu của kubelet phát biểu thành spec (gRPC interface); mọi vendor storage/network implement spec đó. DIP ở quy mô ngành — kubelet không import code của AWS.
- **Đối chứng "đủ dùng"**: rất nhiều service Go thành công phụ thuộc thẳng `*pgxpool.Pool` trong repository struct và chỉ đảo chiều ở tầng service→repository interface. Pragmatic DIP: đảo một tầng, nơi lợi ích test lớn nhất — không cần đảo mọi tầng để "đúng sách".

## 7. Anti-pattern

- **Interface đặt nhầm phía** (cạnh implementation): gián tiếp mà không đảo — chi phí của DIP, lợi ích bằng không. Kiểm tra một giây: *package nghiệp vụ còn import package hạ tầng không?* Còn → chưa đảo.
- **Service Locator**: `container.Get("productSource")` rải trong code nghiệp vụ — dependency tàng hình, lỗi wiring nổ lúc runtime thay vì compile, và code nghiệp vụ lại phụ thuộc... cái locator (một hạ tầng khác). So sánh đầy đủ với DI ở Level 4.
- **Abstraction mô phỏng hạ tầng**: interface `Database` có `Query(sql string)` — nghiệp vụ vẫn nói ngôn ngữ SQL, đổi backend vẫn vỡ. Interface phải phát biểu bằng **ngôn ngữ nghiệp vụ** (`BySKU`), không phải bọc mỏng API hạ tầng.
- **Đảo chiều toàn tập + DI framework cho service 3 endpoint**: over-engineering định danh được — magic của container đắt hơn toàn bộ app.

## 8. Khi nào KHÔNG dùng

Script, tool, prototype: import thẳng, gọi thẳng. Service CRUD mỏng: một tầng interface cho repository (phục vụ test) là đủ. Phần mềm mà "hạ tầng" chính là sản phẩm (CLI wrapper cho Postgres, chẳng hạn) — không có "nghiệp vụ độc lập hạ tầng" để bảo vệ. DIP là khoản đầu tư cho **lõi nghiệp vụ sống lâu hơn các lựa chọn hạ tầng** — không có lõi đó thì không có gì để đầu tư.

---

*Tiếp theo: [2.6 — DRY, KISS, YAGNI, Law of Demeter, Tell Don't Ask](/series/software-design/level-2-principles/06-dry-kiss-yagni-lod/)*
