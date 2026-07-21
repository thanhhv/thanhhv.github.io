+++
title = "1.2 — Abstraction & Encapsulation: che giấu đúng thứ cần che"
date = "2026-07-17T07:20:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## 1. Problem Statement

Tiếp tục câu chuyện e-commerce. Team bạn cần gửi email xác nhận đơn hàng. Một dev viết:

```go
// V1 — trong order_service.go
func (s *OrderService) ConfirmOrder(ctx context.Context, orderID string) error {
    order, err := s.repo.Find(ctx, orderID)
    if err != nil {
        return err
    }
    order.Status = "confirmed"
    if err := s.repo.Save(ctx, order); err != nil {
        return err
    }

    // Gửi email — chi tiết SMTP nằm ngay tại đây
    auth := smtp.PlainAuth("", "noreply@shop.vn", os.Getenv("SMTP_PASS"), "smtp.gmail.com")
    body := fmt.Sprintf("Subject: Đơn hàng %s đã xác nhận\r\n\r\nCảm ơn bạn!", order.ID)
    return smtp.SendMail("smtp.gmail.com:587", auth,
        "noreply@shop.vn", []string{order.CustomerEmail}, []byte(body))
}
```

Chạy được. Ship được. Nhưng 3 tháng sau:

- Marketing muốn email HTML có template đẹp.
- Công ty chuyển từ Gmail SMTP sang SendGrid vì bị rate limit.
- Cần gửi thêm SMS cho đơn giá trị cao.
- QA muốn test `ConfirmOrder` mà **không gửi email thật**.

Cả 4 yêu cầu đều buộc mở `OrderService` ra sửa — dù *nghiệp vụ xác nhận đơn hàng không hề đổi*. Đây là dấu hiệu nhận biết quan trọng nhất: **chi tiết kỹ thuật (gửi email thế nào) đang rò rỉ vào nơi chứa nghiệp vụ (xác nhận đơn là gì)**.

## 2. Code Smell — phân tích

Vì sao V1 có vấn đề, cụ thể theo 4 loại chi phí ở chương 1.1:

- **Chi phí hiểu**: người đọc `ConfirmOrder` để hiểu nghiệp vụ phải lội qua chuỗi SMTP auth, format header email — nhiễu hoàn toàn không liên quan đến "xác nhận đơn".
- **Chi phí lan truyền**: đổi nhà cung cấp email = sửa mọi service có gửi email (báo trước smell *Shotgun Surgery* ở Level 3).
- **Chi phí kiểm chứng**: không thể unit test — gọi hàm là bắn email thật ra internet. Muốn test phải dựng SMTP giả hoặc... không test.
- **Mức độ trừu tượng trộn lẫn**: hàm này nói hai "ngôn ngữ" cùng lúc — ngôn ngữ nghiệp vụ (order, confirm) và ngôn ngữ hạ tầng (SMTP, port 587, auth). Mỗi lần não chuyển ngôn ngữ là một lần tốn năng lượng.

## 3. First Principles

### Abstraction là gì — định nghĩa thực dụng

> **Abstraction là quyết định: ở tầng này, người đọc cần biết GÌ và không cần biết GÌ.**

Abstraction tốt không phải là "thêm interface". Nó là **nén thông tin**: gói một cụm chi tiết phức tạp sau một cái tên mà người dùng có thể tin tưởng *mà không cần mở ra xem*. `notifier.OrderConfirmed(order)` là một abstraction: 30 dòng SMTP nén thành một câu có nghĩa nghiệp vụ.

Thước đo abstraction tốt (mượn từ John Ousterhout — *A Philosophy of Software Design*): **module sâu (deep module)** = interface nhỏ, chức năng lớn phía sau. Abstraction tồi = **module nông (shallow)**: interface phức tạp gần bằng chính phần nó che giấu — bạn tốn công đi qua lớp gián tiếp mà không được giảm độ phức tạp nào.

```
Deep module (tốt)             Shallow module (tồi)
┌────┐  interface nhỏ         ┌──────────────────┐ interface to gần bằng ruột
│    │                        │                  │
│    │                        └──────────────────┘
│    │  nhiều chức năng       ┌──────────────────┐
│    │  phía sau              │                  │
└────┘                        └──────────────────┘
```

Ví dụ deep module kinh điển: `io.Reader` của Go — interface đúng **một method** `Read(p []byte) (int, error)`, nhưng che giấu mọi thứ từ file, network socket, gzip stream, đến HTTP body.

### Encapsulation là gì — và khác Abstraction ở đâu

Hai khái niệm hay bị dùng lẫn lộn:

- **Abstraction**: che giấu **độ phức tạp** — người dùng không cần biết *cách* làm.
- **Encapsulation**: che giấu **trạng thái và quyền sửa đổi** — người ngoài không được *phá* invariant (bất biến) của module.

Invariant là điều kiện luôn phải đúng: "tổng tiền đơn hàng = tổng tiền các item", "trạng thái đơn chỉ đi theo chiều pending → confirmed → shipped". Encapsulation tồn tại để **bảo vệ invariant bằng compiler thay vì bằng kỷ luật con người**.

```go
// ❌ Không encapsulation — invariant chỉ được bảo vệ bằng "mọi người nhớ nhé"
type Order struct {
    Status string  // ai cũng có thể order.Status = "shipped" từ pending
    Items  []Item
    Total  int64   // ai cũng có thể sửa Total mà quên sửa Items
}

// ✅ Encapsulation — invariant được bảo vệ bằng compiler
type Order struct {
    status string      // chữ thường: private trong package
    items  []Item
    // Total không lưu — tính từ items, không thể lệch pha
}

func (o *Order) Confirm() error {
    if o.status != "pending" {
        return fmt.Errorf("cannot confirm order in status %q", o.status)
    }
    o.status = "confirmed"
    return nil
}

func (o *Order) Total() int64 {
    var t int64
    for _, it := range o.items {
        t += it.Price * int64(it.Qty)
    }
    return t
}
```

Lưu ý đặc thù Go: đơn vị encapsulation là **package**, không phải struct. Mọi code cùng package thấy field private. Hệ quả thiết kế: package nên nhỏ và tập trung — package `order` chứa mọi thứ về order và chỉ về order. Trong TypeScript, đơn vị là class (`private`/`#field`) — nhưng nhớ rằng `private` của TS chỉ là compile-time; `#field` mới thực sự ẩn ở runtime.

### Nếu bỏ qua nguyên lý này thì sao?

Không encapsulation → mọi nơi trong codebase đều *có thể* sửa trạng thái → khi có bug "đơn hàng nhảy trạng thái sai", phạm vi điều tra là **toàn bộ codebase** thay vì một file. Chi phí debug tỉ lệ với số nơi có quyền ghi.

## 4. Refactoring Journey

Quay lại `ConfirmOrder`. Đi từng bước — không nhảy thẳng đến "kiến trúc đẹp".

**Bước 1 — Extract Method: tách mức trừu tượng.** Chưa cần interface, chỉ cần tách hàm:

```go
// V2 — nghiệp vụ đọc được bằng một hơi thở
func (s *OrderService) ConfirmOrder(ctx context.Context, orderID string) error {
    order, err := s.repo.Find(ctx, orderID)
    if err != nil {
        return err
    }
    if err := order.Confirm(); err != nil {
        return err
    }
    if err := s.repo.Save(ctx, order); err != nil {
        return err
    }
    return s.sendConfirmationEmail(order) // chi tiết đẩy xuống dưới
}

func (s *OrderService) sendConfirmationEmail(order *Order) error {
    /* ... SMTP ở đây ... */
}
```

Được gì: chi phí hiểu giảm ngay — mỗi hàm một mức trừu tượng. Chưa được gì: vẫn không test được, vẫn phải sửa service khi đổi SendGrid. Với nhiều codebase nhỏ, **dừng ở đây là đủ** — đó là quyết định hợp lệ.

**Bước 2 — Tách interface theo nhu cầu của người dùng.** Khi yêu cầu "test không gửi email thật" và "đổi sang SendGrid" xuất hiện **thật** (không phải phỏng đoán), đó là tín hiệu tách:

```go
// V3 — interface đặt ở phía NGƯỜI DÙNG (package order), không phải phía email
package order

// Notifier nói ngôn ngữ nghiệp vụ, không nói "email" hay "SMTP"
type Notifier interface {
    OrderConfirmed(ctx context.Context, o *Order) error
}

type Service struct {
    repo     Repository
    notifier Notifier
}

func (s *Service) ConfirmOrder(ctx context.Context, orderID string) error {
    o, err := s.repo.Find(ctx, orderID)
    if err != nil {
        return err
    }
    if err := o.Confirm(); err != nil {
        return err
    }
    if err := s.repo.Save(ctx, o); err != nil {
        return err
    }
    return s.notifier.OrderConfirmed(ctx, o)
}
```

```go
// package emailnotify — một implementation, ở NGOÀI package order
type SendGridNotifier struct{ client *sendgrid.Client }

func (n *SendGridNotifier) OrderConfirmed(ctx context.Context, o *order.Order) error {
    /* chi tiết SendGrid */
}
```

Quan sát điều tinh tế nhất ở đây: interface tên `Notifier` với method `OrderConfirmed` — **không phải** `EmailSender.Send(to, subject, body)`. Vì sao?

- `OrderConfirmed` là **nhu cầu của nghiệp vụ**; email/SMS/push chỉ là cách thực hiện. Khi thêm SMS, `OrderService` không đổi một dòng.
- Nếu interface là `Send(to, subject, body)`, service vẫn phải biết soạn subject, body — chi tiết vẫn rò rỉ, chỉ là rò rỉ qua tham số thay vì qua import.

Đây chính là mầm của **Dependency Inversion** (chương 2.5): interface thuộc về phía sử dụng, và được định nghĩa theo ngôn ngữ của phía sử dụng.

**Test giờ trở nên tầm thường:**

```go
type fakeNotifier struct{ confirmed []*order.Order }

func (f *fakeNotifier) OrderConfirmed(_ context.Context, o *order.Order) error {
    f.confirmed = append(f.confirmed, o)
    return nil
}
// => unit test ConfirmOrder chạy trong micro giây, không mạng, không SMTP
```

So sánh TypeScript: cùng ý tưởng, nhưng TS thường khai báo `implements Notifier` tường minh, và test hay dùng `jest.mock` thay vì viết fake tay. Fake tay (như Go) thường cho test dễ đọc và ít mong manh hơn mock framework — bài học này áp dụng được cho cả hai ngôn ngữ.

## 5. Trade-off

| Lựa chọn | Được | Mất |
|---|---|---|
| V1 (inline tất cả) | Đọc một chỗ thấy hết; không có gián tiếp | Không test được; mọi thay đổi hạ tầng đụng nghiệp vụ |
| V2 (extract method) | Chi phí hiểu giảm mạnh, gần như miễn phí | Vẫn coupling cứng với SMTP |
| V3 (interface) | Test được; đổi hạ tầng không đụng nghiệp vụ; thêm kênh thông báo dễ | Thêm một lớp gián tiếp; người đọc muốn biết "email gửi thế nào" phải nhảy file; thêm khái niệm phải đặt tên đúng |

Quy tắc thực dụng: **Extract method thì làm ngay lập tức, luôn luôn** (chi phí ~0). **Interface thì đợi lý do thật** — nhu cầu test, hoặc implementation thứ hai xuất hiện, hoặc ranh giới hạ tầng/nghiệp vụ rõ ràng (email, DB, message queue gần như luôn xứng đáng có interface vì bản chất chúng là chi tiết thay thế được).

## 6. Production Examples

- **`io.Reader`/`io.Writer` (Go stdlib)**: abstraction sâu nhất của Go. `json.NewDecoder(r io.Reader)` không biết và không quan tâm data đến từ file, network hay memory — nhờ đó một decoder dùng được cho mọi nguồn. Đây là lý do bạn stream được file 10GB qua HTTP mà không load vào RAM.
- **`database/sql` (Go stdlib)**: `sql.DB` là abstraction che giấu connection pooling, retry, prepared statement caching. Driver (Postgres, MySQL) chỉ implement interface nhỏ trong `database/sql/driver`. Bạn đổi DB mà code nghiệp vụ giữ nguyên — đúng mô hình V3 ở trên, ở quy mô công nghiệp.
- **`http.Handler`**: một method `ServeHTTP(w, r)` — mà cả hệ sinh thái middleware, framework (chi tiết ở Level 4, Middleware Pattern) xây trên đó.
- **Node.js Stream API**: cùng vai trò `io.Reader/Writer`, nhưng lịch sử phức tạp hơn (stream1 → stream2 → stream3, callback vs async iterator) — minh họa rằng abstraction **khó sửa sau khi công bố**: hàng triệu package phụ thuộc, mọi thay đổi phải tương thích ngược.

## 7. Anti-pattern

**Leaky Abstraction** — abstraction rò rỉ chi tiết nó hứa che giấu:

```go
// ❌ Interface "trừu tượng" nhưng bắt caller biết SQL
type UserRepo interface {
    FindWhere(sqlWhereClause string) ([]User, error) // caller phải viết SQL!
}
```

Đổi sang MongoDB? Mọi caller vỡ. Abstraction này có hình thức của interface nhưng không có giá trị của abstraction.

**Over-Abstraction / Speculative Generality** — interface cho thứ không bao giờ có implementation thứ hai:

```go
// ❌ 5 tầng cho một app CRUD: mỗi lần đọc code nhảy 5 file,
// mỗi tầng chỉ forward call — toàn shallow module
Controller → IOrderFacade → OrderFacade → IOrderManager → OrderManager → IOrderRepo → ...
```

**Getter/Setter cho mọi field** — hình thức của encapsulation, không có bản chất:

```go
func (o *Order) SetStatus(s string) { o.status = s } // ❌
```

Cái này không bảo vệ invariant nào — nó chỉ là public field đội mũ. Encapsulation thật là `Confirm()`, `Cancel()`: expose **hành vi nghiệp vụ**, không phải expose quyền ghi dữ liệu thô. (Đây cũng là tinh thần *Tell, Don't Ask* — chương 2.6.)

## 8. Khi nào KHÔNG cần

- Script, tool nội bộ, code dùng một lần: inline thẳng, V1 là đúng.
- Struct thuần dữ liệu (DTO, config, row từ DB): public field là bình thường và idiomatic trong Go — không phải struct nào cũng cần encapsulation, chỉ struct **có invariant** mới cần.
- Đừng tạo interface khi chỉ có một implementation và không có nhu cầu test cách ly — trong Go, thêm interface sau này **rẻ** (implicit interface, không cần sửa type cũ), nên chờ đợi gần như không tốn gì. Đây là khác biệt lớn với Java/C#: ở Go, YAGNI với interface gần như luôn thắng.

---

*Chương tiếp theo: [1.3 — Coupling, Cohesion & Dependency](/series/software-design/level-1-fundamentals/03-coupling-cohesion-dependency/) — thước đo định lượng cho mọi điều vừa nói.*
