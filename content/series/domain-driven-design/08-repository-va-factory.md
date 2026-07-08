+++
title = "Chương 08 — Repository và Factory: Cửa khẩu giữa Domain và hạ tầng lưu trữ"
date = "2026-07-09T15:00:00+07:00"
draft = false
tags = ["backend", "ddd", "architecture"]
series = ["Domain-Driven Design"]
+++

> Vị trí trong bộ tài liệu: chương thứ ba của Tactical Design. Chương [06](/series/domain-driven-design/06-entity-va-value-object/) xây viên gạch (Entity, Value Object), chương [07](/series/domain-driven-design/07-aggregate/) xây bức tường (Aggregate và ranh giới nhất quán). Chương này trả lời câu hỏi còn treo: aggregate được **lấy ra** và **cất vào** đâu, bằng cách nào, mà domain model không bị ORM và SQL xâm thực.

## 1. Problem Statement: Domain model chết đuối trong query

Một codebase NestJS điển hình sau 18 tháng:

```typescript
// order.service.ts — 47 chỗ gọi repository của TypeORM
const orders = await this.orderRepo.find({
  where: { status: In(['PLACED', 'PAID']), customerId },
  relations: ['items', 'items.product', 'customer', 'payments'],
  order: { createdAt: 'DESC' },
});
```

Câu hỏi kiểm tra: đoạn code trên load ra cái gì? Trả lời thật: **không ai biết chắc**. Có chỗ load `relations: ['items']`, có chỗ thêm `'payments'`, có chỗ quên. Cùng một "Order" nhưng lúc đầy lúc vơi tùy query — method `order.total()` chạy đúng hay ném lỗi undefined phụ thuộc vào việc *người gọi cách đó ba tầng* có nhớ load `items` không. Bug loại này không hiện khi dev test (dev luôn load đủ), chỉ hiện ở production path hiếm.

Vấn đề thứ hai nghiêm trọng hơn: **ghi một phần aggregate**. Dev cần sửa địa chỉ giao hàng, tiện tay `orderRepo.update(id, { shippingAddress })` — bỏ qua toàn bộ invariant trong class Order. Chương 07 xây bức tường quanh aggregate; ORM repository mở một cái cổng sau to bằng cả bức tường.

Vấn đề thứ ba: **domain phụ thuộc hạ tầng**. Business logic ở service gọi thẳng TypeORM/GORM → muốn test một rule nghiệp vụ phải dựng database; muốn đổi cách lưu (thêm cache, đổi DB, tách service) phải mổ vào code nghiệp vụ. Doanh nghiệp trả giá bằng tốc độ: mỗi thay đổi nghiệp vụ kéo theo rủi ro hạ tầng và ngược lại.

Nếu không giải quyết: model giàu hành vi xây ở chương 06–07 sẽ bị bào mòn — vì đường vào/ra dữ liệu không tôn trọng ranh giới của nó. Aggregate mà load vơi, save nửa, bypass invariant được thì aggregate chỉ còn là tài liệu.

## 2. Tại sao DDD đưa ra Repository và Factory

**Bối cảnh**: Evans quan sát rằng vòng đời một object nghiệp vụ có ba giai đoạn khó — *sinh ra* (khởi tạo đúng, đủ invariant ngay từ đầu), *tồn tại giữa các lần chạy* (lưu và tái tạo từ storage), *được tìm thấy* (truy xuất giữa hàng triệu bản ghi). Cả ba đều dễ làm domain model rò rỉ chi tiết hạ tầng.

- **Business problem**: rule nghiệp vụ phải đúng bất kể dữ liệu đến từ Postgres, cache, hay file test. Khi rule dính chặt vào cách lưu trữ, mỗi thay đổi nghiệp vụ là một dự án hạ tầng.
- **Technical problem**: object graph trong memory và bảng quan hệ trong DB là hai thế giới lệch trở kháng (impedance mismatch). Phải có một chỗ *duy nhất* chịu trách nhiệm dịch hai chiều — nếu không, việc dịch rải khắp nơi, mỗi chỗ dịch một kiểu.
- **Design problem**: cần một abstraction nói ngôn ngữ domain ("cho tôi Order này", "cất Order này") che hoàn toàn ngôn ngữ hạ tầng ("JOIN ba bảng, map cột, tăng version"). Đó là Repository. Và cần một chỗ đảm bảo object **sinh ra đã hợp lệ** thay vì "new xong rồi set dần" — đó là Factory.

## 3. Bản chất

### 3.1. Repository là collection ảo của Aggregate — không phải cổng SQL của bảng

Hình dung đúng: Repository giả lập một **collection in-memory chứa toàn bộ aggregate** của một loại. `orderRepository.findById(id)` về mặt ngữ nghĩa giống `orders.get(id)` trên một Map khổng lồ; `save(order)` giống `orders.set(id, order)`. Việc phía sau là ba bảng SQL, một document Mongo hay một dòng cache — là chi tiết cài đặt mà domain không cần biết.

Từ hình dung đó suy ra mọi quy tắc:

- **Một repository cho một aggregate root** — không có `OrderLineRepository`, vì OrderLine không tồn tại độc lập ngoài Order. Có repository riêng cho bộ phận của aggregate tức là mở cổng sửa bộ phận mà không qua root — vỡ invariant.
- **Trả về aggregate hoàn chỉnh** — không có chuyện "Order thiếu items". Collection thật không bao giờ trả về object vơi.
- **Save nguyên tử cả aggregate** — một transaction, kiểm version, xong. Không có API "update mỗi field này".
- **Interface thuộc domain layer, implementation thuộc infrastructure** — dependency inversion: domain định nghĩa *nó cần gì*, hạ tầng đáp ứng.

Nó bảo vệ điều gì? **Ranh giới aggregate ở đường ra/vào dữ liệu** — chốt chặn cuối cùng để bức tường chương 07 không bị đào hầm bên dưới. Nó giảm complexity thế nào? Toàn bộ độ phức tạp mapping dồn về một chỗ có tên, có test; phần còn lại của hệ thống nói chuyện bằng ngôn ngữ nghiệp vụ.

### 3.2. Đối chiếu quan trọng: Repository DDD ≠ repository của TypeORM/GORM

Đây là chỗ hiểu sai phổ biến nhất trong các team NestJS/Go, vì **trùng tên**:

| | `Repository<OrderEntity>` của TypeORM / `gorm.DB` | Repository của DDD |
|---|---|---|
| Đơn vị làm việc | Bảng / row | Aggregate |
| API | Generic: `find`, `update`, `delete`, query builder vô hạn | Hẹp, đặt tên nghiệp vụ: `findById`, `save`, `nextIdentity`, vài finder có chủ đích |
| Ai định nghĩa | Framework | Domain layer (interface do bạn viết) |
| Trả về | Entity ORM — có thể vơi tùy `relations` | Aggregate hoàn chỉnh, luôn luôn |
| Cho phép ghi một phần | Có (`update(id, patch)`) | Không — chỉ `save(aggregate)` |
| Vị trí trong kiến trúc | Là hạ tầng | Interface ở domain, hạ tầng chỉ là *một* implementation |

`InjectRepository(OrderEntity)` của NestJS đưa cho bạn một **DAO generic** — công cụ tốt để *cài đặt* repository DDD, nhưng bản thân nó không phải repository DDD. Khi application service inject thẳng DAO đó, ba vấn đề của mục 1 quay lại đủ mặt. Câu thần chú: **ORM repository là chi tiết bên trong; DDD repository là hợp đồng bên ngoài.**

### 3.3. Factory — vì "new xong set dần" là cửa sổ vô hiệu invariant

Aggregate phức tạp có invariant ngay từ lúc sinh: một Booking phải có phòng, khoảng ngày hợp lệ, giá đã chốt; một LoanApplication phải có người vay đã KYC. Mẫu `const b = new Booking(); b.setRoom(...); b.setDates(...)` tạo ra **khoảng thời gian object tồn tại ở trạng thái không hợp lệ** — và không gì đảm bảo người gọi set đủ trước khi dùng. Factory thu hẹp cửa sổ đó về 0: object hoặc sinh ra hợp lệ, hoặc không sinh ra.

Ba cấp độ, dùng cấp thấp nhất đủ dùng:

1. **Static factory method trên aggregate** — đủ cho 80% trường hợp: `Order.draft(id, customerId)`, `Booking.hold(room, dateRange, price)`. Tên method nói được *nghiệp vụ khởi tạo nào* (draft khác với import từ hệ cũ).
2. **Factory class riêng** — khi khởi tạo cần dữ liệu/dịch vụ ngoài aggregate: `BookingFactory` cần `PricingPolicy` để chốt giá. Factory class thuộc domain layer.
3. **Reconstitution** (tái tạo từ DB) — con đường riêng cho repository, **bỏ qua logic khởi tạo nghiệp vụ nhưng không bỏ qua tính hợp lệ cấu trúc**: `Order.reconstitute(props, version)`. Phân biệt rõ với factory nghiệp vụ: tái tạo không phát event, không chạy side effect.

## 4. Cách hoạt động

### 4.1. Dòng chảy chuẩn của một use case ghi

```mermaid
sequenceDiagram
    participant AS as Application Service
    participant R as OrderRepository (interface - domain)
    participant Impl as PgOrderRepository (infrastructure)
    participant DB as PostgreSQL
    participant A as Order (aggregate)

    AS->>R: findById(orderId)
    R->>Impl: (DI resolve)
    Impl->>DB: SELECT orders + order_lines WHERE id=?
    Impl->>Impl: map rows → Order.reconstitute(...)
    Impl-->>AS: Order (hoàn chỉnh, version=7)
    AS->>A: order.cancel("khách đổi ý")
    A->>A: kiểm invariant, đổi state, ghi event OrderCancelled
    AS->>R: save(order)
    Impl->>DB: BEGIN; UPDATE orders ... WHERE id=? AND version=7;<br/>DELETE+INSERT order_lines; INSERT outbox(events); COMMIT
    Impl-->>AS: OK (hoặc ConcurrencyError)
```

Chú ý ba điểm: (1) application service không biết SQL tồn tại; (2) save là nguyên tử cả aggregate + version + event (outbox — chương 13); (3) mapping row↔aggregate chỉ sống trong `PgOrderRepository`.

### 4.2. TypeScript (NestJS) — interface ở domain, implementation ở infrastructure

```typescript
// domain/order/order.repository.ts — KHÔNG import TypeORM
import { Order } from './order.aggregate';

export const ORDER_REPOSITORY = Symbol('OrderRepository');

export interface OrderRepository {
  findById(id: string): Promise<Order | null>;
  /** Finder có chủ đích nghiệp vụ — không phải query builder tự do */
  findDraftsOlderThan(days: number): Promise<Order[]>;
  save(order: Order): Promise<void>;   // nguyên tử: aggregate + version + events
  nextIdentity(): string;
}
```

```typescript
// infrastructure/order/pg-order.repository.ts
import { Injectable } from '@nestjs/common';
import { DataSource } from 'typeorm';
import { Order } from '../../domain/order/order.aggregate';
import { OrderRepository } from '../../domain/order/order.repository';
import { OrderRow, OrderLineRow } from './order.rows'; // ORM entity = chi tiết nội bộ

@Injectable()
export class PgOrderRepository implements OrderRepository {
  constructor(private readonly ds: DataSource) {}

  async findById(id: string): Promise<Order | null> {
    const row = await this.ds.getRepository(OrderRow)
      .findOne({ where: { id }, relations: { lines: true } }); // LUÔN load đủ
    return row ? this.toDomain(row) : null;
  }

  async save(order: Order): Promise<void> {
    await this.ds.transaction(async (tx) => {
      const res = await tx.getRepository(OrderRow).update(
        { id: order.id, version: order.version },        // optimistic lock
        { ...this.toRow(order), version: order.version + 1 },
      );
      if (res.affected === 0) throw new ConcurrencyError(order.id);
      await tx.getRepository(OrderLineRow).delete({ orderId: order.id });
      await tx.getRepository(OrderLineRow).insert(this.toLineRows(order));
      await tx.getRepository(OutboxRow).insert(
        order.pullEvents().map(e => OutboxRow.from(e)),  // event đi cùng transaction
      );
    });
  }

  private toDomain(row: OrderRow): Order { /* map row → Order.reconstitute(...) */ }
  private toRow(order: Order): Partial<OrderRow> { /* map ngược */ }
}
```

```typescript
// order.module.ts — nối interface với implementation
@Module({
  providers: [{ provide: ORDER_REPOSITORY, useClass: PgOrderRepository }],
})
export class OrderModule {}
// application service inject bằng token:
// constructor(@Inject(ORDER_REPOSITORY) private readonly orders: OrderRepository) {}
```

Cái giá phải trả nhìn thấy ngay: hai class thay vì một (`OrderRow` cho ORM, `Order` cho domain) + code mapping. Đổi lại: domain test không cần DB, ORM đổi không chạm nghiệp vụ, và **không tồn tại con đường nào ghi Order mà né được invariant**.

### 4.3. Go — interface trong package domain, implementation trong package infra

```go
// internal/order/domain/repository.go — không import database/sql, gorm
package domain

import "context"

type OrderRepository interface {
    FindByID(ctx context.Context, id string) (*Order, error)
    Save(ctx context.Context, o *Order) error // nguyên tử + version check
}
```

```go
// internal/order/infra/pg_repository.go
package infra

type PgOrderRepository struct{ db *sql.DB }

func (r *PgOrderRepository) Save(ctx context.Context, o *domain.Order) error {
    return runInTx(ctx, r.db, func(tx *sql.Tx) error {
        res, err := tx.ExecContext(ctx,
            `UPDATE orders SET status=$1, version=version+1
             WHERE id=$2 AND version=$3`,
            o.Status(), o.ID(), o.Version)
        if err != nil { return err }
        if n, _ := res.RowsAffected(); n == 0 {
            return domain.ErrConcurrency
        }
        if err := replaceLines(ctx, tx, o); err != nil { return err }
        return insertOutbox(ctx, tx, o.PullEvents())
    })
}
```

Điểm Go đáng chú ý: interface khai báo **ở phía dùng** (package domain) đúng idiom "accept interfaces, return structs" — package infra import domain, không bao giờ ngược lại. Compiler ép chiều phụ thuộc; trong NestJS bạn phải tự kỷ luật bằng token DI và lint rule.

### 4.4. Testing với in-memory repository

```go
// order/domain/testsupport: fake dùng cho application-service test
type InMemoryOrderRepo struct {
    mu sync.Mutex
    m  map[string]*domain.Order
}
func (r *InMemoryOrderRepo) FindByID(_ context.Context, id string) (*domain.Order, error) {
    r.mu.Lock(); defer r.mu.Unlock()
    o, ok := r.m[id]; if !ok { return nil, domain.ErrNotFound }
    return o, nil
}
func (r *InMemoryOrderRepo) Save(_ context.Context, o *domain.Order) error {
    r.mu.Lock(); defer r.mu.Unlock(); r.m[o.ID()] = o; return nil
}
```

Use case test chạy toàn bộ luồng nghiệp vụ bằng map trong memory — mili giây, không docker. Lưu ý giới hạn trung thực: fake **không** kiểm tra được mapping SQL và version check — những cái đó cần integration test riêng cho `PgOrderRepository` (ít, chậm, nhưng bắt buộc có). Hai tầng test bổ sung nhau, không thay thế nhau.

### 4.5. Query needs vs domain needs — đừng ép repository làm báo cáo

Repository phục vụ **đường ghi**: load aggregate → gọi hành vi → save. Nhu cầu *đọc* (danh sách 20 cột JOIN 5 context cho admin, dashboard, search) mà đi qua repository sẽ ép nó mọc thêm finder vô hạn và trả về aggregate vơi — hai anti-pattern cùng lúc. Lời giải: **tách đường đọc** — query service/read model trả DTO thẳng từ SQL, không đụng aggregate (CQRS mức 1, chi tiết chương [13](/series/domain-driven-design/13-ddd-va-distributed-systems/)). Quy tắc ngón tay cái: cần dữ liệu để **ra quyết định ghi** → repository; cần dữ liệu để **hiển thị** → query path.

## 5. Điểm mạnh

- **Chốt chặn cuối của aggregate boundary**: không ai ghi một phần, không ai load vơi — invariant giữ được ở tầng dữ liệu, nơi mọi bug consistency sinh ra.
- **Domain test không cần hạ tầng**: rule nghiệp vụ test bằng in-memory fake, nhanh hơn 100–1000 lần integration test → vòng lặp phát triển ngắn.
- **Điểm đổi hạ tầng duy nhất**: thêm cache-aside, đổi Postgres→CockroachDB, tách bảng — mổ một class, nghiệp vụ không biết gì.
- **API tự tài liệu hóa**: `findDraftsOlderThan(30)` nói được nghiệp vụ; `find({where: ...})` thì không.

## 6. Điểm yếu

- **Boilerplate mapping** — hai model + mapper hai chiều cho mỗi aggregate. Với aggregate 15 field, đây là công việc nhàm và dễ lệch (quên map field mới).
- **Mất tiện ích ORM** ở đường ghi: lazy loading, change tracking, cascade — những thứ ORM làm hộ giờ bạn làm tay (delete+insert lines, version check).
- **Hai tầng test** phải duy trì: fake cho use case, integration cho mapping.
- **Cám dỗ thường trực**: deadline gần, dev inject thẳng `DataSource` "tạm một lần" — không có lint/architecture test chặn import thì ranh giới mòn dần theo từng sprint.

## 7. Trade-off

- **Tách model (domain + ORM) vs dùng chung một class**: tách = thuần khiết, đắt; chung = rẻ, ORM decorator/tag len vào domain và cám dỗ setter quay lại. Thực dụng theo loại subdomain: **Core Domain tách, Supporting dùng chung có kỷ luật** (không setter public, mọi mutation qua method). Đừng trả giá thuần khiết cho phần nghiệp vụ không ai cạnh tranh.
- **Repository hẹp vs generic repository**: hẹp = mỗi finder là quyết định thiết kế có tên; generic = nhanh lúc đầu, rồi query builder rò ra application layer và aggregate boundary thành trang trí. Generic dùng làm **base class nội bộ** của implementation thì được; phơi ra làm hợp đồng công khai thì không.
- **Delete+insert collection con vs diff từng row**: delete+insert đơn giản, đúng nguyên tử, nhưng tốn write và phá FK trỏ vào line; diff phức tạp hơn nhưng ít write. Bắt đầu bằng delete+insert, đo rồi mới tối ưu.
- **Factory class vs constructor**: factory thêm một lớp gián tiếp — chỉ đáng khi khởi tạo *thật sự* có logic nghiệp vụ hoặc phụ thuộc ngoài. Constructor/static method là mặc định đúng.

## 8. Production Considerations

- **N+1 và kích thước aggregate**: repository load đủ aggregate nghĩa là aggregate to sẽ load đắt — đây là *thiết kế phản hồi cho bạn* rằng aggregate quá to (chương 07), đừng chữa bằng lazy loading trong domain (lazy load = aggregate vơi trá hình + query ngoài transaction).
- **Version check phải nằm trong UPDATE** (`WHERE version=?`), không phải SELECT-rồi-so-sánh — check ngoài là TOCTOU race.
- **Outbox cùng transaction với save** (chương 13) — event rời transaction là nguồn "ghi rồi mà không ai biết".
- **Migration khi model đổi**: đổi domain model kéo đổi mapping + schema; giữ backward-compatible migration (expand–migrate–contract) và để mapper đọc được cả schema cũ trong giai đoạn chuyển.
- **Cache**: cache-aside ở implementation (theo aggregate ID, invalidate khi save) — trong suốt với domain. Đừng cache ở application service, sẽ quên invalidate.
- **Team**: quy ước review bắt buộc — mọi import ORM ngoài thư mục `infrastructure/` là red flag; enforce bằng dependency-cruiser (TS) / depguard (Go lint).

## 9. Best Practices

- Một repository cho một **aggregate root**; interface đặt cạnh aggregate trong domain layer.
- API hẹp và mang tên nghiệp vụ; mỗi finder mới phải trả lời được "use case ghi nào cần nó?".
- `save()` nguyên tử: aggregate + version + outbox event trong một transaction.
- Reconstitution tách khỏi factory nghiệp vụ (không phát event khi load).
- Fake in-memory đặt trong package/module test-support chính thức — để mọi use case test dùng chung một fake đúng.
- Với Go: interface ở package domain, `var _ domain.OrderRepository = (*PgOrderRepository)(nil)` để compiler kiểm implementation.
- Với NestJS: inject qua token (Symbol) + interface, không inject class cụ thể; cấm `@InjectRepository` ngoài infrastructure.

## 10. Anti-patterns

- **Generic repository làm hợp đồng công khai** — `IRepository<T>` với `findAll/findWhere/updatePartial`: aggregate boundary chết, mọi service tự viết query, invariant bypass được. Nguy hiểm vì nó *trông* giống DDD (có chữ Repository!) nên qua review dễ dàng.
- **Repository trả về partial aggregate** — tham số `includeLines?: boolean` là lời thú nhận aggregate quá to hoặc query needs đi nhầm cửa.
- **Repository trong domain entity** — `order.save()` hoặc entity gọi repository để tự load thứ khác: Active Record trá hình, domain dính chặt hạ tầng, test lại cần DB.
- **Repository cho non-root** — `OrderLineRepository`: cổng sau xuyên thủng bức tường chương 07.
- **Business logic trong repository** — implementation "tiện tay" kiểm rule trước khi save: rule giờ sống ở hai nơi, và chỉ chạy khi đi qua đường lưu đó.
- **Query màn hình đi qua repository** — 40 finder, mỗi finder một biến thể JOIN: dấu hiệu cần tách read path.

## 11. Khi nào KHÔNG cần Repository/Factory kiểu DDD

Cùng logic mọi chương: khi không có aggregate thật để bảo vệ. Bảng cấu hình, danh mục, log, CRUD admin — `gorm.DB`/`Repository<T>` của framework dùng thẳng là đúng công cụ: nhanh, ít code, ai cũng hiểu. Dựng interface + implementation + mapper + fake cho một bảng `countries` là nghi lễ vô nghĩa, và chính những nghi lễ vô nghĩa này làm team mất niềm tin vào DDD ở chỗ nó *thật sự* cần. Repository DDD là hệ quả của aggregate; không aggregate, không repository.

## Đọc tiếp

Aggregate, Repository đã có — nhưng logic nghiệp vụ nằm vắt qua *nhiều* aggregate thì đặt ở đâu, và tầng nào điều phối use case? [Chương 09 — Domain Service và Application Service](/series/domain-driven-design/09-domain-service-va-application-service/).

- Quay lại: [07 — Aggregate](/series/domain-driven-design/07-aggregate/) · [Mục lục](/series/domain-driven-design/00-muc-luc/)
- Liên quan: [12 — DDD và Kiến trúc](/series/domain-driven-design/12-ddd-va-kien-truc/) (dependency inversion toàn cảnh) · [13 — DDD và Distributed Systems](/series/domain-driven-design/13-ddd-va-distributed-systems/) (outbox, CQRS)
