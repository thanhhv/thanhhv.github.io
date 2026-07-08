+++
title = "Chương 16: Anti-patterns & Khi nào KHÔNG nên dùng DDD"
date = "2026-07-10T00:00:00+07:00"
draft = false
tags = ["backend", "ddd", "architecture"]
series = ["Domain-Driven Design"]
+++

> **Vị trí chương này trong tài liệu:** Đây là chương cuối cùng. Sau khi đã đi qua toàn bộ strategic design (chương 01–05), tactical patterns (06–11), kiến trúc và distributed systems (12–13), đưa vào production (14) và các case study (15a, 15b), chương này làm hai việc mà một tài liệu trung thực bắt buộc phải làm: **liệt kê những cách người ta làm hỏng DDD trong thực tế** — với code thật, dấu hiệu nhận biết thật — và **nói thẳng khi nào đừng dùng DDD**. Cuối chương là phần tổng kết toàn bộ tài liệu và lộ trình học tiếp.

Sau 20 năm nhìn các dự án "làm DDD", tôi rút ra một quan sát khó nghe: **số dự án thất bại vì áp dụng DDD sai nhiều hơn số dự án thất bại vì không dùng DDD.** Không dùng DDD, tệ nhất bạn có một mớ code khó bảo trì — vẫn ship được. Dùng DDD sai, bạn có mớ code khó bảo trì đó *cộng thêm* ba tầng abstraction, năm loại object trung gian, một team kiệt sức vì tranh cãi thuật ngữ, và một management đã mất niềm tin vào mọi đề xuất kiến trúc trong 5 năm tới.

Vì vậy chương này không phải phụ lục. Nó là phanh xe. Một công cụ mạnh mà không có phanh thì không phải công cụ — là tai nạn đang chờ lịch.

---

## Mục lục chương

- [16.1 Danh mục anti-pattern](#161-danh-mục-anti-pattern)
- [16.2 Khi nào KHÔNG nên dùng DDD](#162-khi-nào-không-nên-dùng-ddd)
- [16.3 Ma trận quyết định](#163-ma-trận-quyết-định-mức-độ-áp-dụng-ddd)
- [16.4 Checklist trước khi quyết định áp dụng DDD](#164-checklist-trước-khi-quyết-định-áp-dụng-ddd)
- [16.5 Tổng kết toàn bộ tài liệu & lộ trình học tiếp](#165-tổng-kết-toàn-bộ-tài-liệu--lộ-trình-học-tiếp)

---

## 16.1 Danh mục anti-pattern

Mỗi anti-pattern dưới đây trình bày theo cùng khung: **mô tả → cách nhận biết trong code thật → vì sao nguy hiểm → cách sửa**. Thứ tự sắp theo tần suất tôi gặp trong audit thực tế, từ phổ biến nhất.

### 16.1.1 Anemic Domain Model — mô hình thiếu máu

**Mô tả.** Domain object chỉ có dữ liệu (getter/setter), toàn bộ logic nghiệp vụ nằm ở tầng service. Object trở thành túi đựng dữ liệu có tên nghiệp vụ nhưng không có hành vi nghiệp vụ. Đây là anti-pattern được Fowler đặt tên từ 2003 và đến nay vẫn là số 1 về độ phổ biến — phần vì ORM và code generator khuyến khích nó, phần vì nó *trông giống* code "chuẩn layer".

**Nhận biết trong code thật (TypeScript/NestJS):**

```typescript
// ANTI-PATTERN: entity chỉ là túi dữ liệu
@Entity()
export class Order {
  @PrimaryColumn() id: string;
  @Column() status: string;          // string tự do — dấu hiệu đi kèm kinh điển
  @Column() totalAmount: number;
  @OneToMany(() => OrderLine, l => l.order) lines: OrderLine[];
  // 40 dòng decorator, 0 dòng logic
}

// Toàn bộ "domain logic" nằm ở service — và bị lặp ở service khác
@Injectable()
export class OrderService {
  async addLine(orderId: string, dto: AddLineDto) {
    const order = await this.repo.findOne(orderId);
    // rule nghiệp vụ nằm ngoài object mà nó bảo vệ:
    if (order.status === 'PAID' || order.status === 'SHIPPED') {
      throw new BadRequestException('Cannot modify paid order');
    }
    order.lines.push(this.mapper.toLine(dto));
    order.totalAmount = order.lines.reduce((s, l) => s + l.price * l.qty, 0);
    await this.repo.save(order);
  }
}

// ...trong khi đó ở AdminOrderService, một đồng nghiệp khác viết:
//   if (order.status === 'PAID') { ... }   ← quên mất 'SHIPPED'. Bug ủ sẵn.
```

Dấu hiệu nhận biết nhanh khi audit: (1) entity không có method nào ngoài getter/setter/constructor; (2) cùng một cụm `if (x.status === ...)` xuất hiện ở ≥ 2 service (grep là ra); (3) mọi field đều public hoặc có setter — nghĩa là bất kỳ ai cũng đặt được object vào trạng thái vô nghĩa (`status = 'PAID'` nhưng `paidAt = null`).

**Vì sao nguy hiểm.** Invariant không có chủ. Rule "đơn đã thanh toán không được sửa" phải được *mọi* service nhớ thi hành — và xác suất mọi người cùng nhớ mãi mãi là zero. Mỗi service mới chạm vào `Order` là một cơ hội bỏ sót rule. Bug loại này không nổ ngay: nó ủ vài tháng rồi hiện ra dưới dạng dữ liệu bẩn trong DB — loại bug đắt nhất để truy nguồn. Và lưu ý một điều thường bị hiểu nhầm: **anemic model chỉ là anti-pattern khi hệ thống có invariant đáng bảo vệ.** Với CRUD thuần, "anemic" chính là thiết kế đúng (xem 16.2).

**Cách sửa.** Kéo hành vi về nơi có dữ liệu; đóng cửa mọi lối đi vòng:

```typescript
// SỬA: rule sống bên trong object mà nó bảo vệ
export class Order {
  private constructor(
    readonly id: OrderId,
    private status: OrderStatus,       // value object, hết string tự do
    private lines: OrderLine[],
  ) {}

  addLine(product: ProductRef, qty: Quantity): void {
    if (!this.status.isModifiable()) {
      // Rule có MỘT chỗ ở. Ai gọi addLine cũng bị kiểm, không cần nhớ.
      throw new OrderNotModifiableError(this.id, this.status);
    }
    this.lines.push(OrderLine.of(product, qty));
  }

  get total(): Money {                 // total là giá trị dẫn xuất — không ai set được
    return this.lines.reduce((s, l) => s.add(l.subtotal()), Money.zero(Currency.VND));
  }
}

// Service teo lại thành điều phối — đúng vai của nó (xem chương 09)
@Injectable()
export class OrderAppService {
  async addLine(cmd: AddLineCommand) {
    const order = await this.orders.byId(cmd.orderId);
    order.addLine(ProductRef.of(cmd.productId), Quantity.of(cmd.qty));
    await this.orders.save(order);
  }
}
```

Chiến thuật refactor khi codebase đã lớn: đừng sửa cả entity một lần. Chọn *một* invariant đang gây bug, chuyển nó vào entity, xóa mọi bản sao rải rác (grep theo tên field), lặp lại. Sau 5–7 vòng, entity tự "đầy máu" trở lại.

### 16.1.2 God Aggregate — aggregate thần thánh ôm cả thế giới

**Mô tả.** Một aggregate phình ra ôm mọi thứ liên quan xa gần: `Customer` chứa orders, payments, support tickets, loyalty points, addresses, preferences... Thường sinh ra từ tư duy ERD: "customer *có* các orders" → nhét orders vào aggregate Customer. Nhưng aggregate không phải object graph — nó là **ranh giới nhất quán giao dịch** (chương 07), và "có quan hệ" không có nghĩa "cần nhất quán tức thì cùng nhau".

**Nhận biết trong code thật (Go):**

```go
// ANTI-PATTERN: aggregate "Customer" 1.500 dòng
type Customer struct {
    ID            CustomerID
    Profile       Profile
    Addresses     []Address
    Orders        []Order          // 4 năm mua hàng = 3.000 orders
    Payments      []Payment
    LoyaltyPoints LoyaltyAccount
    Tickets       []SupportTicket
    // ... 23 field nữa
}

// Mọi thao tác đều phải load cả vũ trụ
func (s *CustomerService) AddLoyaltyPoints(ctx context.Context, id CustomerID, pts int) error {
    c, err := s.repo.Load(ctx, id) // kéo 3.000 orders từ DB để... cộng điểm
    if err != nil { return err }
    c.LoyaltyPoints.Add(pts)
    return s.repo.Save(ctx, c)     // và ghi đè cả vũ trụ, đè luôn thay đổi của người khác
}
```

Dấu hiệu: repository load mất > 100ms và kéo N bảng; `Save` ghi lại những thứ không đổi; lock contention hoặc optimistic-lock conflict giữa hai thao tác *chẳng liên quan gì nhau* (cộng điểm loyalty fail vì ai đó vừa đổi địa chỉ); test cần builder 200 dòng để dựng nổi một instance.

**Vì sao nguy hiểm.** Ba mặt. *Performance:* mọi thao tác trả chi phí load/save của thao tác nặng nhất. *Concurrency:* aggregate là đơn vị lock — gom cả thế giới vào một aggregate nghĩa là serialize mọi hoạt động của một khách hàng; hệ thống có traffic là nghẽn ngay điểm này. *Tổ chức:* mọi team đều phải sửa cùng một struct — merge conflict và coupling team vĩnh viễn.

**Cách sửa.** Câu hỏi kim chỉ nam của Vernon: *"invariant nào cần nhất quán TỨC THÌ trong cùng một transaction?"* — chỉ những thứ đó ở chung aggregate. Còn lại tách ra, tham chiếu nhau **bằng ID**, đồng bộ bằng event nếu cần:

```go
// SỬA: mỗi ranh giới nhất quán một aggregate — tham chiếu bằng ID
type Customer struct {          // chỉ còn identity + profile + invariant của chính nó
    ID        CustomerID
    Profile   Profile
    Addresses []Address         // ok ở lại: đổi address có rule ràng với profile
}

type Order struct {             // aggregate riêng — invariant của đơn không dính profile
    ID         OrderID
    CustomerID CustomerID       // tham chiếu bằng ID, KHÔNG nhúng object
    Lines      []OrderLine
}

type LoyaltyAccount struct {    // aggregate riêng — điểm cộng trừ độc lập, hay bị đụng đồng thời
    CustomerID CustomerID
    Balance    Points
}

func (s *LoyaltyService) AddPoints(ctx context.Context, id CustomerID, pts Points) error {
    acc, err := s.loyalty.ByCustomer(ctx, id) // load MỘT dòng, lock MỘT dòng
    if err != nil { return err }
    acc.Add(pts)
    return s.loyalty.Save(ctx, acc)
}
```

Đánh đổi phải nói rõ: tách xong, những rule *xuyên* aggregate (ví dụ "tổng nợ mọi đơn ≤ hạn mức khách") chuyển từ nhất quán tức thì sang **eventual consistency hoặc cần thiết kế riêng** (aggregate `CreditLimit` chuyên trách, hoặc chấp nhận kiểm tra "gần đúng" + đối soát). Đó không phải khiếm khuyết của cách sửa — đó là chi phí thật mà God Aggregate đã *giấu* bằng cách trả nó ở dạng lock contention.

### 16.1.3 Shared Domain Model giữa các context

**Mô tả.** Một class `Product`/`Customer`/`Order` "dùng chung" cho mọi bounded context — thường nằm trong package `common/`, `shared/`, hoặc một thư viện `company-domain-models` được mọi service import. Xuất phát từ trực giác DRY: "Product là Product, viết hai lần làm gì?". Nhưng như chương [04](/series/domain-driven-design/04-bounded-context/) đã chỉ ra: `Product` của Catalog (mô tả, ảnh, SEO) và `Product` của Inventory (tồn kho, vị trí kệ) và `Product` của Pricing (giá, thuế) là **ba khái niệm khác nhau tình cờ trùng tên**. DRY áp dụng cho *tri thức*, không phải cho *chữ*.

**Nhận biết trong code thật (TypeScript):**

```typescript
// ANTI-PATTERN: libs/shared-domain/src/product.ts — "một Product cho tất cả"
export class Product {
  id: string;
  name: string;
  description: string;      // chỉ Catalog cần
  seoSlug: string;          // chỉ Catalog cần
  stockQuantity: number;    // chỉ Inventory cần
  warehouseLocation: string;// chỉ Inventory cần
  basePrice: number;        // chỉ Pricing cần
  taxCategory: string;      // chỉ Pricing cần
  weightGrams: number;      // chỉ Shipping cần
  // 31 field — mỗi context dùng ~20%, nhưng chịu 100% thay đổi
}
```

Dấu hiệu: class có nhóm field "chỉ dùng ở X"; PR đổi model phải được 4 team approve; version bump của lib `shared-domain` gây deploy dây chuyền toàn công ty; field kiểu `status2`, `priceV2` xuất hiện vì không ai dám đổi field cũ.

**Vì sao nguy hiểm.** Đây là coupling ở mức nguy hiểm nhất — **coupling ngữ nghĩa**. Mọi context bị xích vào nhịp thay đổi của mọi context khác: Pricing muốn đổi `basePrice: number` thành cấu trúc `Money` phải đàm phán với Shipping — bên chẳng liên quan gì đến tiền. Kết quả sau 2 năm: model chung đóng băng (không ai dám đổi), mọi tiến hóa nghiệp vụ diễn ra *bên ngoài* model (field phụ, bảng phụ, map "extra"), và bạn quay lại đúng cái big ball of mud mà DDD sinh ra để tránh — chỉ là phiên bản phân tán qua npm registry.

**Cách sửa.** Mỗi context tự định nghĩa model của mình, chỉ chứa những gì *nó* cần, theo ngôn ngữ của *nó*. Dữ liệu chảy giữa các context qua **contract** (event/API DTO) — thứ được phép chung — chứ không qua domain class:

```typescript
// catalog/domain/product.ts — Product theo nghĩa của Catalog
export class Product {
  constructor(readonly id: ProductId, private name: ProductName,
              private description: RichText, private seo: SeoMetadata) {}
  rename(name: ProductName) { /* rule đặt tên của Catalog */ }
}

// pricing/domain/priced-item.ts — cùng "sản phẩm", khái niệm khác, tên khác
export class PricedItem {
  constructor(readonly productId: ProductId,   // CHỈ chia sẻ identity
              private base: Money, private tax: TaxCategory) {}
  priceFor(tier: CustomerTier): Money { /* rule giá */ }
}

// Cái DUY NHẤT được chia sẻ: contract sự kiện (published language, chương 05)
// libs/contracts/catalog-events.ts — DTO thuần, version hóa, không hành vi
export interface ProductRegisteredV1 {
  productId: string;
  name: string;
  occurredAt: string;
}
```

Đánh đổi: có duplication *hình thức* (ba class cùng mang ID sản phẩm), thêm mapping ở biên. Chấp nhận — chi phí mapping là tuyến tính và nhìn thấy được; chi phí coupling ngữ nghĩa là phi tuyến và vô hình cho đến ngày nó khóa chết mọi thay đổi. Thứ duy nhất nên nằm trong `shared/`: value object thật sự phổ quát và ổn định (`Money`, `Email`), và các contract DTO có version.

### 16.1.4 Shared Database — nhiều context/service chung một database

**Mô tả.** Nhiều bounded context (hoặc tệ hơn, nhiều microservice) cùng đọc-ghi một schema. Anh em song sinh của 16.1.3 nhưng ở tầng dữ liệu — và còn nguy hiểm hơn, vì ranh giới bị xuyên thủng một cách *vô hình*: không có import statement nào để lint bắt, chỉ có những câu SQL rải rác.

**Nhận biết trong code thật:**

```go
// ANTI-PATTERN: service "reporting" JOIN thẳng vào bảng của Ordering và Pricing
// reporting/query.go
rows, err := db.QueryContext(ctx, `
    SELECT o.id, o.status, p.final_price, c.tier
    FROM orders o                          -- bảng của Ordering context
    JOIN price_quotes p ON p.order_id = o.id   -- bảng của Pricing context
    JOIN customers c    ON c.id = o.customer_id -- bảng của CRM context
    WHERE o.created_at > $1`, since)
```

Dấu hiệu: một bảng xuất hiện trong connection string của ≥ 2 service; team Ordering muốn đổi tên cột `status` phải gửi email "to: all-engineering"; migration của context A làm đỏ dashboard của context B; và dấu hiệu tổ chức đặc trưng — *không ai dám xóa bất kỳ cột nào* vì không biết ai đang đọc.

**Vì sao nguy hiểm.** Schema trở thành **public API không có version, không có contract, không có deprecation policy**. Mọi cột là surface area. Encapsulation của aggregate trở nên vô nghĩa: bạn viết invariant cẩn thận trong `Order.addLine()`, nhưng service khác `UPDATE orders SET status = 'PAID'` thẳng vào bảng — invariant chết không kèn không trống. Với microservices, shared database còn triệt tiêu chính lý do tồn tại của việc tách service: không deploy độc lập được (migration là chuyện chung), không scale độc lập được (chung connection pool, chung IOPS), fault không cô lập được (một service làm nghẽn DB là chết cả cụm).

**Cách sửa.** Theo mức độ khả thi tăng dần chi phí:

1. **Mức tối thiểu (trong monolith):** quy ước ownership — mỗi bảng thuộc đúng một context, context khác muốn dữ liệu phải qua public API của module. Enforce bằng review + grep định kỳ (`grep -rn "FROM orders" --include="*.go" | grep -v ordering/`).
2. **Mức khá:** schema riêng cho từng context trong cùng DB instance, user DB riêng với quyền chỉ trên schema của mình — vi phạm ranh giới thành *lỗi permission* thay vì thành thói quen.
3. **Với nhu cầu đọc chéo (như reporting ở trên):** không JOIN chéo — mỗi context publish event, reporting xây read model riêng của nó từ các event đó. Đắt hơn, đúng — nhưng reporting giờ tùy biến thoải mái mà không xích chân ai.
4. **Khi đã tách service:** database riêng thật sự, dữ liệu chảy qua event/API. Đây là điều kiện *định nghĩa* của microservice — chưa tách được DB thì chưa tách xong service, chỉ là monolith phân tán.

**Không sửa thì sao?** Hệ thống vẫn chạy — shared database là anti-pattern "sống dai" nhất vì nó *tiện* mỗi ngày và chỉ *đắt* mỗi quý. Hóa đơn đến dưới dạng: mọi thay đổi schema cần họp liên team (velocity giảm dần đều), và một ngày nào đó, một `UPDATE` từ service lạ tạo ra trạng thái dữ liệu mà domain model coi là bất khả — và on-call mất ba ngày để hiểu chuyện gì xảy ra.

### 16.1.5 Transaction xuyên nhiều aggregate

**Mô tả.** Một transaction database gói việc thay đổi 2+ aggregate — thường kèm lý luận "cho chắc, đằng nào cũng chung DB". Điều này phá vỡ hợp đồng thiết kế của aggregate: aggregate được *định nghĩa* là ranh giới transaction (chương 07). Nếu bạn thường xuyên cần sửa hai aggregate trong một transaction, thì hoặc ranh giới aggregate sai, hoặc bạn đang cưỡng ép nhất quán tức thì cho thứ nghiệp vụ vốn chấp nhận eventual.

**Nhận biết trong code thật (Go):**

```go
// ANTI-PATTERN: một transaction, ba aggregate
func (s *OrderService) PlaceOrder(ctx context.Context, cmd PlaceOrderCmd) error {
    return s.db.WithinTx(ctx, func(tx *Tx) error {
        order := NewOrder(cmd)
        if err := s.orders.SaveTx(tx, order); err != nil { return err }

        // Aggregate thứ 2 trong cùng tx:
        inv, _ := s.inventory.LoadTx(tx, cmd.ProductID) // + row lock
        if err := inv.Reserve(cmd.Qty); err != nil { return err }
        if err := s.inventory.SaveTx(tx, inv); err != nil { return err }

        // Aggregate thứ 3, khác context luôn:
        loyalty, _ := s.loyalty.LoadTx(tx, cmd.CustomerID)
        loyalty.AddPoints(pointsFor(order))
        return s.loyalty.SaveTx(tx, loyalty)
    })
}
```

Dấu hiệu: hàm `WithinTx`/`@Transactional` ở application service mà bên trong gọi save cho nhiều repository; deadlock xuất hiện theo cặp thao tác cụ thể (place-order vs adjust-inventory); latency ghi tăng theo số aggregate bị gom; và khi ai đó đề xuất tách service, phát hiện *không tách nổi* vì transaction dính chùm.

**Vì sao nguy hiểm.** *Kỹ thuật:* transaction càng rộng, lock giữ càng lâu, deadlock và contention tăng theo cấp số; hot aggregate (Inventory của sản phẩm đang sale) thành nút cổ chai toàn hệ thống. *Kiến trúc:* nó hàn cứng các aggregate — và cả các context — vào một DB vĩnh viễn; mọi kế hoạch tách sau này chết từ đây. *Nghiệp vụ — điểm ít người để ý nhất:* nó thường **mã hóa sai nghiệp vụ thật**. Hỏi domain expert: "đơn đặt thành công nhưng cộng điểm loyalty trễ 5 giây thì có sao không?" — "Không sao." Nghĩa là nhất quán tức thì ở đây là *yêu cầu bịa* của engineer, và bạn đang trả giá hiệu năng + coupling cho một yêu cầu không tồn tại.

**Cách sửa.** Quy tắc: **một transaction — một aggregate; phần còn lại đi bằng domain event** (chương [10](/series/domain-driven-design/10-domain-event/)), với outbox pattern để không mất event (chương [13](/series/domain-driven-design/13-ddd-va-distributed-systems/)):

```go
// SỬA: transaction chỉ ôm Order + outbox; phần còn lại eventual
func (s *OrderService) PlaceOrder(ctx context.Context, cmd PlaceOrderCmd) error {
    return s.db.WithinTx(ctx, func(tx *Tx) error {
        order := NewOrder(cmd)
        if err := s.orders.SaveTx(tx, order); err != nil { return err }
        // Event ghi CÙNG transaction với aggregate → không bao giờ lạc
        return s.outbox.AppendTx(tx, OrderPlaced{
            OrderID: order.ID, CustomerID: cmd.CustomerID,
            Lines: order.LineSnapshots(),
        })
    })
    // Consumer riêng (idempotent): InventoryReserver xử lý OrderPlaced → reserve;
    // LoyaltyAccrual xử lý OrderPlaced → cộng điểm.
    // Reserve THẤT BẠI thì sao? → event ReservationFailed → Order chuyển
    // trạng thái/hủy — đây là saga tối giản, và là nghiệp vụ THẬT:
    // ngoài đời, hết hàng sau khi đặt là chuyện có quy trình xử lý hẳn hoi.
}
```

Đánh đổi — nói đủ hai chiều: bạn đổi tính đơn giản của strong consistency lấy khả năng scale và tách rời, và phải trả bằng idempotency, retry, bù trừ (compensation), UI phản ánh trạng thái trung gian ("đang giữ hàng..."). **Nếu nghiệp vụ thật sự đòi hai thứ thay đổi nguyên tử cùng nhau** (trừ tiền tài khoản A cộng tài khoản B nội bộ một ledger) — thì đó là *một* aggregate bị vẽ nhầm thành hai; sửa ranh giới, đừng sửa bằng transaction rộng. Transaction xuyên aggregate hợp lệ duy nhất ở một chỗ: cùng aggregate type, thao tác batch có chủ đích, hiếm và được cô lập.

### 16.1.6 Generic Repository lạm dụng

**Mô tả.** Một `Repository<T>` với đủ bộ `findAll / findById / save / delete / findBy(criteria)` cho *mọi* entity, thường kèm base class và query builder tự chế. Trông rất DRY, rất "framework". Nhưng repository trong DDD không phải lớp bọc DB — nó là **cửa khẩu của aggregate**, và interface của nó phải nói ngôn ngữ nghiệp vụ: aggregate này được lấy lên *theo cách nào* và *vì mục đích gì*.

**Nhận biết trong code thật (TypeScript):**

```typescript
// ANTI-PATTERN
export class GenericRepository<T> {
  findAll(): Promise<T[]> { ... }                    // ai gọi findAll trên bảng 40 triệu dòng?
  findBy(criteria: Partial<T>): Promise<T[]> { ... } // query ngẫu hứng không giới hạn
  save(entity: T): Promise<void> { ... }
  delete(id: string): Promise<void> { ... }          // xóa Order? nghiệp vụ có cho phép xóa?
}

// Hệ quả tất yếu — logic nghiệp vụ TRÀN ra caller dưới dạng criteria:
const overdue = await orderRepo.findBy({ status: 'CONFIRMED' })
  .then(os => os.filter(o =>
    o.confirmedAt < subDays(new Date(), 3) && !o.remindedAt)); // rule "quá hạn" là gì? — tri thức nghiệp vụ nằm vất vưởng ở đây
```

Dấu hiệu: mọi repository giống hệt nhau; các rule chọn lọc nghiệp vụ ("đơn quá hạn", "khách rủi ro") tồn tại dưới dạng tổ hợp filter lặp lại ở nhiều caller; method `delete` tồn tại cho cả những khái niệm nghiệp vụ *không bao giờ được xóa* (lệnh chuyển tiền!); và performance thảm vì filter chạy trong RAM thay vì trong DB.

**Vì sao nguy hiểm.** Ba lý do. Một: **tri thức nghiệp vụ rò rỉ và phân mảnh** — "đơn quá hạn nhắc" định nghĩa ở 3 nơi, sửa 2 nơi, bug ở nơi thứ 3. Hai: **interface rộng vô hạn = không kiểm soát được truy cập dữ liệu** — không ai trả lời được câu "những đường truy vấn nào tồn tại vào bảng orders?", nên không tối ưu index nổi, không audit nổi. Ba: **nó xóa vai trò thiết kế của repository**: chặn được ai xóa Order đâu, khi `delete` nằm sẵn trên base class.

**Cách sửa.** Repository riêng cho từng aggregate, interface hẹp, mỗi method là một *nhu cầu nghiệp vụ có tên*:

```typescript
// SỬA: interface do domain định nghĩa, nói tiếng nghiệp vụ
export interface OrderRepository {
  byId(id: OrderId): Promise<Order | null>;
  save(order: Order): Promise<void>;
  // Rule "quá hạn nhắc" có MỘT tên, MỘT định nghĩa, chạy bằng SQL có index:
  overdueForReminder(asOf: Date): Promise<Order[]>;
  // KHÔNG có delete — Order không bao giờ bị xóa, chỉ bị cancel (một method trên Order)
  // KHÔNG có findAll — không tồn tại nhu cầu nghiệp vụ "lấy tất cả đơn"
}
```

Hai lưu ý thực dụng. (1) Dùng generic repository làm **chi tiết cài đặt nội bộ** thì được — `class PgOrderRepository extends BaseRepo implements OrderRepository` tái dùng plumbing thoải mái, miễn là *interface public* vẫn hẹp và mang tên nghiệp vụ. Cái bị cấm là phơi generic interface ra cho application service. (2) Nhu cầu đọc màn hình (danh sách, filter tùy ý của admin UI) đừng nhét vào repository của aggregate — đó là việc của đường đọc/read model (chương 14, mục 14.6.5): query DTO trực tiếp, tự do và nhanh, không đi qua domain.

### 16.1.7 Cargo-cult DDD — đủ lễ phục, rỗng nội dung

**Mô tả**: codebase có đầy đủ `domain/`, `application/`, `infrastructure/`, có class tên `*.aggregate.ts`, có `ValueObject` base class — nhưng aggregate toàn getter/setter, business rule vẫn nằm hết trong service, và "domain event" chỉ được phát chứ không ai subscribe. Hình thức của DDD, linh hồn của CRUD.

**Nhận biết trong code thật**:

```typescript
// order.aggregate.ts — tên là aggregate, thân là túi dữ liệu
export class Order extends AggregateRoot {
  @Column() status: string;              // public, ai gán gì cũng được
  getStatus() { return this.status; }
  setStatus(s: string) { this.status = s; }   // "hành vi" duy nhất
}

// order.service.ts — rule vẫn ở đây, như chưa hề có DDD
if (order.getStatus() === 'PAID') throw new Error('cannot cancel');
order.setStatus('CANCELLED');
```

Bài kiểm 30 giây: mở aggregate lớn nhất của codebase, đếm số method mang **tên nghiệp vụ có kiểm tra invariant bên trong**. Bằng 0 → cargo cult.

**Vì sao nguy hiểm**: đây là anti-pattern đắt nhất về mặt niềm tin — team trả đủ chi phí của DDD (nhiều file, nhiều tầng, học từ vựng mới) mà nhận zero lợi ích, rồi kết luận "DDD là vẽ vời". Lần sau ai đề xuất domain model thật sẽ bị dí vào mặt chính cái codebase này làm bằng chứng chống lại.

**Cách sửa**: không sửa cấu trúc thư mục — sửa *một* luồng nghiệp vụ quan trọng nhất: kéo rule từ service vào aggregate, xóa setter của các field liên quan, viết test invariant. Một luồng làm thật đáng giá hơn mười thư mục đúng chuẩn.

### 16.1.8 Ubiquitous Language chỉ tồn tại trong tài liệu

**Mô tả**: có glossary trên Confluence, có buổi event storming chụp ảnh đăng Slack — nhưng code vẫn `data`, `info`, `processOrder(payload)`, ticket vẫn viết bằng từ của dev, và domain expert sáu tháng chưa được hỏi lại câu nào. **Nhận biết**: lấy 5 từ quan trọng nhất trong glossary, grep codebase — không ra kết quả nào là ngôn ngữ đã chết lâm sàng. **Vì sao nguy hiểm**: mọi pattern tactical phía sau đặt tên theo một ngôn ngữ không ai nói — model đúng hình thức nhưng lệch ngữ nghĩa, và độ lệch tăng dần theo mỗi tính năng (chương [03](/series/domain-driven-design/03-ubiquitous-language/) đã phân tích cơ chế). **Cách sửa**: đưa ngôn ngữ vào Definition of Done của PR; mỗi sprint một lần rename theo glossary; và quan trọng nhất — nối lại kênh nói chuyện định kỳ với domain expert, vì ngôn ngữ chết là *triệu chứng* của kênh giao tiếp chết.

### 16.1.9 Event như RPC trá hình

**Mô tả**: phát "event" `SendWelcomeEmailRequested`, `DeductInventoryCommand` — tên là event, ngữ nghĩa là mệnh lệnh chờ đúng một kẻ thi hành. **Nhận biết**: tên event chứa động từ nguyên thể/mệnh lệnh thay vì quá khứ; producer *biết và cần* consumer cụ thể làm gì; nếu consumer đó tắt thì luồng nghiệp vụ của producer coi như fail. **Vì sao nguy hiểm**: bạn nhận đủ nhược điểm của cả hai thế giới — coupling ngữ nghĩa chặt như gọi hàm trực tiếp, nhưng mất stack trace, mất type-check, thêm độ trễ và thêm broker phải nuôi. Khi cần ra lệnh, hãy ra lệnh tường minh (command + handler, hoặc gọi API); event chỉ dành cho *thông báo điều đã xảy ra* mà producer không quan tâm ai phản ứng (chương [10](/series/domain-driven-design/10-domain-event/), mục anti-pattern).

### 16.1.10 Bounded Context = Microservice một cách máy móc

**Mô tả**: đọc xong chương strategic design, team vẽ được 8 bounded context — và lập tức dựng 8 repo, 8 pipeline, 8 database. **Vì sao nguy hiểm**: bounded context là ranh giới *ngữ nghĩa của model*; microservice là ranh giới *triển khai và vận hành*. Trùng nhau được thì tốt, nhưng quan hệ đúng là "một service không nên chứa nhiều hơn một context" chứ không phải "mỗi context một service". Tách 8 service với một team 6 người là mua toàn bộ chi phí chương [13](/series/domain-driven-design/13-ddd-va-distributed-systems/) (outbox, saga, idempotency, distributed debugging) để giải một bài toán tổ chức... không tồn tại — vì chỉ có một team. Kết quả kinh điển: distributed monolith — 8 service deploy phải cùng nhau, gọi nhau đồng bộ dây chuyền, một cái chết kéo cả chùm. **Cách sửa**: modular monolith với ranh giới module = ranh giới context (chương [12](/series/domain-driven-design/12-ddd-va-kien-truc/)); tách service khi và chỉ khi có *lực kéo vận hành đo được* — team đông phải chia, nhu cầu scale lệch nhau, nhịp deploy xung đột (chương [14](/series/domain-driven-design/14-ddd-trong-production/)).

### 16.1.11 DDD-lite mọi nơi — trải mỏng thay vì đầu tư đúng chỗ

**Mô tả**: áp một "chuẩn DDD toàn công ty" đồng đều cho mọi module — CRUD danh mục cũng aggregate + repository interface + 4 tầng như core domain. Nghe có vẻ kỷ luật, thực chất là phiên bản tổ chức của cargo cult: chi phí trải đều, lợi ích dồn cục. **Vì sao nguy hiểm**: vi phạm chính nguyên lý chiến lược nền tảng nhất của DDD (chương [02](/series/domain-driven-design/02-domain-va-subdomain/)) — nguồn lực phải dồn cho Core Domain. Team tiêu 40% quỹ thời gian để duy trì nghi lễ ở các module Supporting/Generic, và đến lượt core domain cần modeling sâu thì hết quỹ. **Cách sửa**: bản đồ subdomain quyết định "liều lượng" — core: full tactical + strategic; supporting: transaction script sạch sẽ; generic: mua/dùng framework thẳng. Sự không đồng đều này là *có chủ đích và lành mạnh*.

---

## 16.2 Khi nào KHÔNG nên dùng DDD

Nguyên tắc gốc để suy mọi trường hợp: **DDD là khoản đầu tư trả trước để mua khả năng chịu đựng thay đổi của business phức tạp**. Không có business phức tạp, hoặc không có tương lai dài để thu hồi vốn — khoản đầu tư lỗ. Cụ thể:

| Trường hợp | Vì sao DDD không mang lại giá trị | Phương pháp phù hợp hơn |
|---|---|---|
| **CRUD app thuần** (quản lý danh mục, form nhập liệu) | Không có business rule để bảo vệ — model "giàu" mà không có gì để giàu | Active Record / scaffold của framework (NestJS + TypeORM thẳng, Rails-style), validation ở DTO |
| **CMS / website nội dung** | Complexity nằm ở trình bày và quản lý nội dung — bài toán đã được ngành giải sẵn | WordPress/Strapi/headless CMS — mua Generic, đừng xây |
| **Landing page / web giới thiệu** | Không có domain — chỉ có giao diện | Static site, no-code |
| **MVP chưa có product-market fit** | Nghiệp vụ đổi hàng tuần theo khám phá thị trường; model hóa sâu thứ tuần sau vứt là lãng phí kép (xây + phá). Điều MVP cần là *tốc độ học*, không phải độ bền | Transaction script + khung code sạch tối thiểu; đặt tên tử tế (Ubiquitous Language mức 0 — miễn phí); ghi lại các quyết định để ngày pivot thành công thì biết đường refactor |
| **Tool nội bộ nhỏ, ít người dùng** | Chi phí sai của nó thấp — bug thì sửa, người dùng ngồi cạnh | CRUD framework, thậm chí Airtable/Retool |
| **Hệ thống tích hợp/ETL thuần** (đọc chỗ này ghi chỗ kia) | Complexity nằm ở data mapping và vận hành pipeline, không ở business rule | Pipeline tooling (Airflow...), schema contract |
| **Team không có đường tới domain expert** | DDD không có nguyên liệu — model xây từ phỏng đoán của dev là model sai có tổ chức, tệ hơn CRUD trung thực | Làm CRUD tốt + đầu tư *giành lấy* kênh domain expert trước, DDD sau |
| **Business đơn giản thật sự** (quy tắc đếm trên một bàn tay, ổn định nhiều năm) | Phần thưởng của model hóa ~ 0; transaction script đọc hiểu trong một buổi | Giữ nguyên — đơn giản là một tài sản, đừng phá |

Điểm cần trung thực nói thêm: "không dùng DDD" hiếm khi là quyết định vĩnh viễn cho cả công ty. Câu hỏi đúng luôn ở mức **từng bộ phận của hệ thống, tại thời điểm này**: sàn TMĐT có thể CRUD ở quản lý banner nhưng rất cần DDD ở pricing; startup có thể transaction script toàn bộ năm nay và cần aggregate cho module thanh toán năm sau. Và chiều ngược cũng đúng — hệ thống *đang* DDD đầy đủ có thể hạ liều lượng ở module đã ổn định vĩnh viễn.

---

## 16.3 Ma trận quyết định mức độ áp dụng DDD

Ba biến đầu vào: **độ phức tạp business rule** (đếm: số rule ràng buộc nhiều dữ liệu, số trạng thái và chuyển trạng thái, tần suất business đổi rule), **tuổi thọ kỳ vọng của hệ thống**, và **quy mô team/tổ chức**.

```mermaid
quadrantChart
    title Liều lượng DDD theo complexity và tuổi thọ
    x-axis Tuổi thọ ngắn --> Tuổi thọ dài
    y-axis Business đơn giản --> Business phức tạp
    quadrant-1 Full DDD: strategic + tactical
    quadrant-2 Tactical chọn lọc, sẵn sàng nâng cấp
    quadrant-3 CRUD / transaction script
    quadrant-4 CRUD sạch + Ubiquitous Language
```

Diễn giải thành bốn mức, kèm ngưỡng gợi ý:

| Mức | Điều kiện | Áp dụng cụ thể |
|---|---|---|
| **0 — Không DDD** | Business đơn giản, tuổi thọ < 1 năm, team ≤ 3 | CRUD trung thực. Chỉ giữ: đặt tên tử tế |
| **1 — Nền móng miễn phí** | Business bắt đầu có rule, hệ thống sẽ sống > 1 năm | Ubiquitous Language + tách module theo hướng business capability + value object cho tiền/thời gian (chống primitive obsession). Chưa cần aggregate/repository nghi lễ |
| **2 — Tactical chọn lọc** | Có ≥ 1 vùng rule dày (tính giá, tồn kho, trạng thái đơn), team 5–15 | Vùng rule dày: aggregate + domain event + repository interface đúng bài. Vùng còn lại: mức 1. Modular monolith |
| **3 — Full strategic + tactical** | Nhiều team, nhiều context, business phức tạp, hệ thống sống nhiều năm | Toàn bộ tài liệu này: subdomain map, context map, team alignment, và các pattern chương 13 khi tách service |

Hai quy tắc dùng ma trận: (1) **đánh giá theo từng bounded context**, không theo cả công ty; (2) **được phép lên mức dần** — mức 1 hôm nay không cản mức 3 sau này, ngược lại chính là đường nâng cấp tự nhiên (chương [14](/series/domain-driven-design/14-ddd-trong-production/), incremental adoption). Sai lầm cần tránh là *nhảy* mức: từ 0 lên 3 trong một quý là công thức cargo cult.

---

## 16.4 Checklist trước khi quyết định áp dụng DDD

Trả lời trung thực 10 câu — mỗi "có" là một điểm:

1. Business rule có ràng buộc **nhiều dữ liệu phải đúng cùng nhau** không (không chỉ validation từng field)?
2. Domain expert có **tồn tại và tiếp cận được** ít nhất hàng tuần không?
3. Hệ thống có sống **≥ 2 năm** và tiếp tục được phát triển (không chỉ bảo trì) không?
4. Business có **đổi rule thường xuyên** (hàng quý trở lên) không?
5. Đã từng có **bug do hai nơi hiểu một khái niệm khác nhau** chưa?
6. Có **≥ 2 team** cùng đụng vào hệ thống không?
7. Có vùng nghiệp vụ là **lợi thế cạnh tranh thật** (làm tốt hơn đối thủ thì thắng) không?
8. Team có ít nhất **1–2 người đã làm domain model thật** (không chỉ đọc sách) không?
9. Tổ chức có chấp nhận **đầu tư chậm hơn 20–30% trong 2–3 tháng đầu** để nhanh hơn về sau không?
10. Có **kênh phản hồi từ vận hành/CS về dev** để model được sửa theo thực tế không?

Thang điểm gợi ý: **0–3**: mức 0–1 của ma trận — DDD đầy đủ sẽ là gánh nặng. **4–6**: mức 1–2 — bắt đầu từ Ubiquitous Language và một context thí điểm. **7–10**: mức 2–3 — không những nên, mà *không* làm mới là rủi ro. Lưu ý câu 2 và câu 8 là hai câu **phủ quyết**: thiếu domain expert thì DDD không có nguyên liệu; thiếu người từng làm thì hãy thuê/mượn một người dẫn (hoặc thí điểm phạm vi rất nhỏ để tự học) trước khi cam kết cả hệ thống.

---

## 16.5 Tổng kết toàn bộ tài liệu & lộ trình học tiếp

### Sợi chỉ đỏ xuyên 16 chương

Nếu phải nén toàn bộ tài liệu vào năm câu: (1) Kẻ thù chính của phần mềm doanh nghiệp là **business complexity**, không phải technical complexity — và không kỹ thuật nào giết được nó, chỉ có thể *tổ chức* nó (chương 01). (2) Tổ chức bắt đầu bằng **chiến lược**: biết chỗ nào đáng đầu tư (02), nói một thứ tiếng (03), vẽ ranh giới ngữ nghĩa (04) và quản trị quan hệ giữa các ranh giới (05). (3) Bên trong ranh giới, **tactical patterns** không phải nghi lễ mà là các công cụ bảo vệ: VO bảo vệ tính đúng của giá trị (06), aggregate bảo vệ invariant trước concurrency (07), repository bảo vệ ranh giới ở tầng dữ liệu (08), service tách "điều phối" khỏi "nghiệp vụ" (09), event tách "việc chính" khỏi "hậu quả" (10), specification cho rule-dạng-câu-hỏi một mái nhà (11). (4) Kiến trúc và hạ tầng phân tán (12, 13) là **hệ quả** của ranh giới đã vẽ — không phải điểm xuất phát. (5) Và tất cả chỉ sống được trong một tổ chức biết đưa nó vào từng bước, tôn trọng Conway's Law, và trung thực về chi phí (14, 15, 16).

DDD, cuối cùng, không phải là một bộ class. Nó là kỷ luật giữ cho **model trong code** và **thực tại trong đầu người làm nghiệp vụ** không trôi xa nhau — vì mọi hệ thống lớn sụp đổ đều bắt đầu từ khoảng cách đó.

### Lộ trình đọc tiếp

Theo thứ tự khuyên đọc: **Vaughn Vernon — "Implementing Domain-Driven Design"** (thực hành đầy đủ nhất, đặc biệt phần aggregate và context mapping); **Vlad Khononov — "Learning Domain-Driven Design"** (2021, hiện đại và cô đọng, cây cầu tốt nhất giữa strategic design và kiến trúc hiện đại); **Eric Evans — "Domain-Driven Design"** (2003 — đọc *sau* khi đã thực hành, phần I–II và XIV–XVII vẫn không có bản thay thế); **Matthew Skelton & Manuel Pais — "Team Topologies"** (mảnh ghép tổ chức mà chương 14 chỉ chạm bề mặt); và **Alberto Brandolini — "Introducing EventStorming"** (kỹ năng workshop — thứ đưa mọi lý thuyết ở đây vào phòng họp thật). Kèm hai nguồn thực hành: repo mẫu `ddd-forum`/`ddd-crew` trên GitHub, và các bài viết của Khononov về "balancing coupling".

Chúc bạn xây được những hệ thống mà ba năm sau người kế nhiệm mở ra vẫn đọc được nghiệp vụ trong code — đó là thước đo thật của mọi thứ trong tài liệu này.

---

- Quay lại: [15b — Case Study: SaaS, Blockchain, Booking, Social](/series/domain-driven-design/15b-case-study-saas-blockchain-booking-social/) · [Mục lục](/series/domain-driven-design/00-muc-luc/)
