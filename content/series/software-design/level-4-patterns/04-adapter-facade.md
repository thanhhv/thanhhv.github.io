+++
title = "4.4 — Adapter & Facade: hai cách thuần hóa code không phải của mình"
date = "2026-07-17T10:40:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

Mở màn nhóm Structural bằng cặp pattern hay bị dùng lẫn tên nhất. Cả hai đều đứng **ở ranh giới giữa code của bạn và code không phải của bạn** (thư viện, hệ legacy, service ngoài) — nhưng giải hai bài toán khác hẳn: Adapter đổi **hình dạng** một interface cho khớp ổ cắm có sẵn; Facade **thu nhỏ** cả một mê cung thành vài cánh cửa.

---

## Phần A — Adapter

### 1. Problem Statement — code evolution

Hệ thống notification của bạn (từ chương 1.4) đã chuẩn hóa quanh interface:

```go
type Notifier interface {
    Send(ctx context.Context, msg Message) error
}
```

Mọi thứ chạy ổn: retry wrapper, rate-limit wrapper, registry — cả một hệ sinh thái nội bộ nói chuyện qua `Notifier`. Rồi công ty ký hợp đồng dùng dịch vụ SMS bên thứ ba, SDK của họ trông thế này:

```go
// SDK bên ngoài — KHÔNG SỬA ĐƯỢC, và tất nhiên không biết Notifier của bạn
type VNSMSClient struct{ /* ... */ }

func (c *VNSMSClient) PushText(phone string, content string, opts *PushOpts) (*PushResult, error)
```

Lệch nhau đủ kiểu: tên method khác, không nhận `context`, tham số rời thay vì struct `Message`, trả thêm `*PushResult`. Hai lựa chọn tồi lộ ra ngay: **sửa hệ của mình theo SDK** (N chỗ gọi `Send` giờ phải biết `PushText` — đuôi vẫy chó); hoặc **rải code dịch thuật tại từng chỗ gọi** (dịch Message→phone/content lặp khắp nơi — Shotgun Surgery chờ sẵn khi đổi nhà cung cấp).

### 2. Pattern — một struct dịch thuật, đứng đúng một chỗ

```go
// ✅ Adapter: type MỚI, của BẠN, khoác interface của bạn, ôm SDK của họ
type VNSMSAdapter struct {
    client *vnsms.VNSMSClient
    sender string
}

func (a *VNSMSAdapter) Send(ctx context.Context, msg Message) error {
    // (1) dịch dữ liệu: Message → tham số SDK
    opts := &vnsms.PushOpts{Sender: a.sender}

    // (2) dịch NGỮ NGHĨA — phần dễ quên và quan trọng hơn phần dịch tên:
    if deadline, ok := ctx.Deadline(); ok {          // SDK không biết context
        opts.TimeoutMs = int(time.Until(deadline).Milliseconds())
    }
    res, err := a.client.PushText(msg.Recipient.Phone, msg.Body, opts)
    if err != nil {
        return fmt.Errorf("vnsms: %w", err)
    }
    if res.Code != vnsms.CodeOK {                    // SDK báo lỗi qua result code,
        return fmt.Errorf("vnsms: rejected: %s", res.Desc)   // hợp đồng của bạn báo qua error
    }
    return nil
}
```

Toàn bộ tri thức "nói chuyện với VNSMS thế nào" sống trong **một file**; hệ thống của bạn không biết VNSMS tồn tại; ngày đổi nhà cung cấp — viết adapter mới, đổi một dòng wiring ở main. Đó là toàn bộ pattern: **không có gì thông minh, và đó chính là điểm mạnh** — adapter phải *ngu* một cách kỷ luật: chỉ dịch, không thêm nghiệp vụ, không retry (đã có wrapper riêng — chương 1.4), không cache.

Chú ý mục (2): dịch **ngữ nghĩa** — context/deadline, quy ước lỗi (error vs result code vs exception), đơn vị (ms vs Duration), encoding, nil vs rỗng — mới là phần adapter dễ làm sai. Dịch tên method thì compiler ép bạn làm đúng; dịch ngữ nghĩa sai thì chỉ LSP (2.3) và production dạy bạn: adapter cũng là một implementation của interface, **hợp đồng ngữ nghĩa của interface áp lên nó đầy đủ** — kể cả contract test.

### 3. Adapter trong Go teo nhỏ — nhắc lại và mở rộng

Chương 1.5 đã nêu: implicit interface làm phần lớn nhu cầu adapt *biến mất* — nếu type bên thứ ba tình cờ có đúng method, nó thỏa mãn interface của bạn, không cần lớp dịch. Adapter chỉ còn cần khi *thật sự lệch hình dạng*. Và cho ca lệch nhẹ nhất — bạn có sẵn một **hàm** nhưng cần một **interface** — Go/stdlib có dạng adapter một dòng đáng học thuộc:

```go
// http.HandlerFunc: ADAPTER TỪ FUNCTION SANG INTERFACE — 3 dòng trong stdlib
type HandlerFunc func(ResponseWriter, *Request)
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) { f(w, r) }

// nhờ nó: mux.Handle("/x", http.HandlerFunc(myFunc)) — không cần khai struct nào
```

Mẫu "function adapter" này (`http.HandlerFunc`, `sort` trước generics...) là adapter được dùng nhiều nhất trong Go mà ít ai gọi tên. TypeScript: structural typing cũng xóa phần lớn nhu cầu; phần còn lại thường là hàm wrap 5 dòng thay vì class.

### 4. Khi nào KHÔNG cần

Chỉ một chỗ gọi SDK, và có thể mãi mãi chỉ một → gọi thẳng, bọc làm gì. Adapter trả lãi khi: interface của bạn có *hệ sinh thái* xung quanh (nhiều caller, nhiều wrapper), hoặc nhà cung cấp là thứ *sẽ đổi*. Ngoài ra cảnh giác **adapter hai tầng tự tạo**: tự định nghĩa interface "trừu tượng hóa" một SDK mà bạn chắc chắn không bao giờ đổi (AWS SDK cho hệ chạy 100% trên AWS) — bạn viết lớp dịch để dịch sang... chính nó, mọi thay đổi SDK vẫn xuyên qua đủ hai lớp. Trừu tượng hóa nhà cung cấp là quyết định chiến lược có giá, không phải phản xạ.

---

## Phần B — Facade

### 1. Problem Statement — code evolution

Nghiệp vụ "hoàn tất đơn hàng" chạm 5 hệ con: trừ kho, tạo vận đơn, thu tiền, xuất hóa đơn, gửi thông báo. Handler HTTP hiện tại:

```go
// ❌ handler biết RUỘT của 5 hệ con — 70 dòng điều phối lộ thiên
func (h *Handler) CompleteOrder(w http.ResponseWriter, r *http.Request) {
    /* parse */
    if err := h.inventory.Reserve(ctx, o.Items); err != nil { /* ... */ }
    ship, err := h.shipping.CreateShipment(ctx, o.Address, o.Items)   // phải biết gọi SAU reserve
    if err != nil {
        h.inventory.Release(ctx, o.Items)                             // phải biết bù trừ!
        /* ... */
    }
    pay, err := h.payment.Charge(ctx, o.Total, o.PaymentToken)
    if err != nil {
        h.shipping.Cancel(ctx, ship.ID)                               // bù trừ tầng nữa
        h.inventory.Release(ctx, o.Items)
        /* ... */
    }
    _ = h.invoicing.Issue(ctx, o, pay.ReceiptID)
    _ = h.notify.OrderCompleted(ctx, o)
    /* respond */
}
```

Rồi yêu cầu đến: CLI admin cũng cần hoàn tất đơn; consumer Kafka xử lý đơn từ đối tác cũng cần. Copy 70 dòng điều phối — kèm toàn bộ tri thức thứ tự-và-bù-trừ — ra ba nơi? Đó là Duplicate Code loại nguy hiểm nhất: trùng lặp *quy trình nghiệp vụ*.

### 2. Pattern — thu quy trình về một cửa

```go
// ✅ Facade: một type gói QUY TRÌNH, expose một cửa đơn giản
type OrderCompletion struct {
    inventory InventoryService
    shipping  ShippingService
    payment   PaymentService
    invoicing InvoicingService
    notify    Notifier
}

func (s *OrderCompletion) Complete(ctx context.Context, o *order.Order) (*Receipt, error) {
    // toàn bộ thứ tự + bù trừ sống Ở ĐÂY, một nơi duy nhất
    /* ...70 dòng cũ, giờ có một nhà... */
}
```

Handler/CLI/consumer giờ gọi một dòng `completion.Complete(ctx, o)` — mỗi client mỏng đi 70 dòng, và tri thức quy trình có **một** chủ. Đây là Facade đúng nghĩa GoF: *interface đơn giản hóa cho một subsystem phức tạp* — không thêm chức năng mới, chỉ **tái đóng gói cách dùng đúng** thành API.

Ba điều làm nên facade tốt:

- **Facade không độc quyền**: GoF nhấn rõ — client *vẫn được* đi thẳng vào subsystem khi cần thao tác tinh (đường hiếm dùng đi cửa riêng, đường phổ thông đi facade). Facade là lối tắt, không phải bức tường. (Muốn bức tường thật — cấm đi thẳng — đó là bounded context + anti-corruption layer, Level 5.)
- **Facade mỏng về nghiệp vụ của chính nó**: nó *điều phối* nghiệp vụ của các hệ con; nếu bạn thấy logic tính toán mọc bên trong facade, nó đang biến thành God Object có mã danh đẹp.
- **Facade là ứng viên interface tự nhiên** cho consumer test cách ly: handler chỉ cần `interface { Complete(ctx, *Order) (*Receipt, error) }` — một method, fake 3 dòng (đúng bài ISP 2.4).

Bạn có thể đã nhận ra: `OrderCompletion` chính là cái mà Clean Architecture gọi là **use case / application service** (Level 5). Đúng vậy — tầng use case *là* Facade được nâng lên thành nguyên tắc kiến trúc. Còn ghi chú từ 3.2 ("God Object teo dần qua vỏ facade") là chiều ngược: facade làm *trạm trung chuyển tạm* trong refactor.

### 3. Production sightings — cả hai pattern

- **`http.Get/Post` (stdlib)**: facade 1 dòng che subsystem `http.Client` + `Transport` + connection pool; cần chỉnh sâu thì tự dựng `Client` — đúng tinh thần "lối tắt, không bức tường". Tương tự: `json.Marshal` (che `Encoder`/`Decoder`), `io.ReadAll`.
- **`os/exec.Command`**: facade che fork/exec/pipe/wait — API mê cung của Unix process thành ba dòng.
- **`database/sql` nhìn lại lần ba**: với driver nó là registry (4.1), với app nó là **facade khổng lồ** — che pooling, retry, prepared statement cache sau `Query/Exec`. Một package, nhiều pattern — thực tế luôn như vậy.
- **`sarama` → `franz-go` (Kafka client)**: sarama nổi tiếng phơi mọi chi tiết protocol (client "không facade" — cực mạnh, cực khó dùng đúng); franz-go/confluent-kafka-go bọc consumer group lifecycle sau API gọn. So hai thư viện cùng domain thấy ngay giá của việc có/không có tầng facade.
- **Stripe/AWS SDK (adapter phía provider)**: SDK chính là *facade chính chủ* trên REST API; còn code của bạn bọc SDK đó sau interface `PaymentGateway` — **adapter chồng trên facade**: chuyện thường ngày, mỗi lớp một việc (SDK: che HTTP; adapter: khớp hợp đồng nội bộ của bạn).
- **TS**: `fetch` wrapper mà mọi codebase FE/BE đều có (`api.get<T>(path)`) — facade che headers/auth/retry/parse; axios interceptor cũng đứng cạnh đó.

### 4. So sánh khách quan: Adapter vs Facade

| | Adapter | Facade |
|---|---|---|
| Bài toán | Interface có sẵn nhưng **sai hình dạng** so với ổ cắm | Subsystem đúng chức năng nhưng **quá phức tạp để dùng trực tiếp** |
| Hình dạng interface mới | **Do bên nhận quyết định** (khớp interface có sẵn của hệ bạn) | **Do bạn thiết kế mới** (đơn giản hóa theo use case) |
| Tỉ lệ | 1 interface ↔ 1 type được bọc | 1 cửa ↔ N thành phần bên trong |
| Chứa tri thức gì | Dịch thuật (tên, dữ liệu, ngữ nghĩa) | Quy trình (thứ tự, phối hợp, bù trừ) |
| Thất bại khi | Dịch sai ngữ nghĩa (vi phạm LSP) | Phình thành God Object; hoặc thành Middle Man nếu subsystem vốn đã đơn giản |
| Câu thử nhanh | "Tôi cần cái này *khớp ổ cắm kia*" | "Tôi cần cách dùng *đơn giản hơn* cho cụm này" |

Hai pattern ghép với nhau tự nhiên và cũng dễ lẫn vào **Decorator/Proxy** — cùng là "một type bọc type khác". Bảng phân biệt đầy đủ cả bốn nằm cuối chương sau, khi đủ mặt anh tài.

---

*Tiếp theo: [4.5 — Decorator & Proxy — và Middleware](/series/software-design/level-4-patterns/05-decorator-proxy/)*
