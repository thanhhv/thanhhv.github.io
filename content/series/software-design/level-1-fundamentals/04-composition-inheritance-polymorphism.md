+++
title = "1.4 — Composition vs Inheritance & Polymorphism"
date = "2026-07-17T07:40:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## 1. Problem Statement

Bài toán: hệ thống có nhiều loại thông báo — Email, SMS, Push. Chúng chia sẻ logic chung (retry, logging, rate-limit) nhưng khác nhau ở cách gửi. Làm sao tái sử dụng phần chung mà không copy-paste?

Đây là bài toán mà **inheritance (kế thừa)** sinh ra để giải. Và cũng là bài toán cho thấy vì sao inheritance thường là câu trả lời sai — đến mức chính GoF, trong cuốn sách 1994, đã viết ở trang 20: *"Favor object composition over class inheritance"*. Go đi xa hơn: **loại bỏ hẳn class inheritance khỏi ngôn ngữ**.

## 2. Code Smell — hành trình xuống dốc của một cây kế thừa

Xem bằng TypeScript, vì Go không cho phép viết cái sai này:

```typescript
// V1 — hợp lý ở ngày đầu
abstract class Notifier {
  send(msg: Message): void {
    this.log(msg);
    for (let i = 0; i < 3; i++) {          // retry chung
      try { this.doSend(msg); return; } catch (e) { /* ... */ }
    }
  }
  protected abstract doSend(msg: Message): void;
  protected log(msg: Message) { /* ... */ }
}

class EmailNotifier extends Notifier {
  protected doSend(msg: Message) { /* SMTP */ }
}
class SmsNotifier extends Notifier {
  protected doSend(msg: Message) { /* Twilio */ }
}
```

Đây là **Template Method pattern** — và ở quy mô này nó ổn. Rồi yêu cầu đổi:

- SMS cần rate-limit (nhà mạng chặn spam), Email không cần.
- Push không cần retry (fire-and-forget), nhưng cần batch.
- Email marketing cần template engine, Email giao dịch không.

```typescript
// V2 — ❌ cây bắt đầu mọc dại
abstract class Notifier { /* retry + log */ }
abstract class RateLimitedNotifier extends Notifier { /* + rate limit */ }
class SmsNotifier extends RateLimitedNotifier {}
class PushNotifier extends Notifier {
  // Push không cần retry → override để... TẮT logic cha
  override send(msg: Message) { this.log(msg); this.doSend(msg); }
}
class MarketingEmailNotifier extends EmailNotifier { /* + template */ }
```

Các triệu chứng — đúng chuẩn từng được ghi trong sách giáo khoa:

1. **Tổ hợp bùng nổ**: mỗi tính năng (retry, rate-limit, batch, template) nhân đôi số lớp tiềm năng. 4 tính năng độc lập = 16 tổ hợp — cây class không biểu diễn nổi tổ hợp; nó chỉ biểu diễn *phân cấp*.
2. **Con phủ định cha**: `PushNotifier` override `send` để *tắt* retry — lớp con không còn "là một" lớp cha theo đúng hành vi. Đây là mầm vi phạm Liskov (chương 2.3).
3. **Fragile Base Class**: sửa `Notifier.send` (thêm metric chẳng hạn) → mọi lớp con đổi hành vi, kể cả những lớp bạn không biết tồn tại (ở codebase khác import thư viện của bạn). Lớp cha **không thể sửa an toàn** nữa — inheritance là dạng coupling mạnh nhất: con dính cha ở cả state, hành vi, và *thứ tự gọi ngầm* giữa các method protected.
4. **Đọc một lớp phải đọc cả tổ tiên**: hành vi của `MarketingEmailNotifier.send` trải trên 3 lớp — yo-yo problem: mắt nhảy lên nhảy xuống cây để lần một luồng chạy.

## 3. First Principles

### Inheritance thực chất trộn 3 thứ khác nhau

Người ta dùng `extends` cho 3 mục đích rất khác nhau, và đây là gốc của mọi rắc rối:

1. **Tái sử dụng code** — con muốn dùng lại method của cha.
2. **Polymorphism (đa hình)** — chỗ nào nhận `Notifier` thì nhận được `SmsNotifier`.
3. **Mô hình hóa quan hệ "is-a"** — SMS *là một* notifier.

Nhận ra điều then chốt: **ba nhu cầu này không cần đi cùng nhau, và mỗi cái có công cụ riêng tốt hơn**:

- Tái sử dụng code → **composition**: chứa một đối tượng và ủy quyền (delegate).
- Polymorphism → **interface**: hợp đồng hành vi, không kèm code, không kèm state.
- Quan hệ is-a → thường là ảo giác của việc mô hình hóa theo danh từ; cái hệ thống thực sự cần là "behaves-as" — và đó lại là interface.

Go tách bạch đúng như vậy: **embedding** cho tái sử dụng, **interface** cho polymorphism, và không có gì cho is-a — vì không cần.

### Vì sao composition thắng — nhìn từ coupling

Ngôn ngữ chương 1.3: inheritance coupling con↔cha ở mức cao nhất (content coupling — con thấy protected state, chen vào giữa luồng chạy của cha). Composition coupling qua interface công khai — mức lỏng nhất. Và composition **chọn lúc runtime** được (tiêm dependency khác nhau), inheritance cố định lúc compile.

## 4. Refactoring Journey — về đất Go

**Bước 1 — Polymorphism bằng interface, tối giản:**

```go
// Interface một method — nhỏ nhất có thể
type Notifier interface {
    Send(ctx context.Context, msg Message) error
}

type EmailNotifier struct{ client *smtp.Client }
func (n *EmailNotifier) Send(ctx context.Context, msg Message) error { /* SMTP */ }

type SmsNotifier struct{ client *twilio.Client }
func (n *SmsNotifier) Send(ctx context.Context, msg Message) error { /* Twilio */ }
```

Lưu ý: không có `implements`. `EmailNotifier` thỏa mãn `Notifier` **ngầm** — chương 1.5 phân tích vì sao đây là tính năng thiết kế quan trọng nhất của Go.

**Bước 2 — Tính năng chung trở thành wrapper, không phải lớp cha:**

```go
// Retry là MỘT NOTIFIER bọc một notifier khác
type RetryNotifier struct {
    next     Notifier
    attempts int
}

func (r *RetryNotifier) Send(ctx context.Context, msg Message) error {
    var err error
    for i := 0; i < r.attempts; i++ {
        if err = r.next.Send(ctx, msg); err == nil {
            return nil
        }
    }
    return fmt.Errorf("after %d attempts: %w", r.attempts, err)
}

// Tương tự: RateLimitNotifier, LoggingNotifier — mỗi cái một file nhỏ, một việc
```

**Bước 3 — Tổ hợp tự do lúc runtime:**

```go
// SMS: rate-limit + retry.  Push: chỉ log. Email: retry + log.
sms   := &LoggingNotifier{next: &RateLimitNotifier{next: &RetryNotifier{next: twilioSender, attempts: 3}}}
push  := &LoggingNotifier{next: fcmSender}
email := &LoggingNotifier{next: &RetryNotifier{next: smtpSender, attempts: 3}}
```

Nhìn lại điều vừa xảy ra: 4 tính năng × N kênh không còn là cây 16 lớp — mà là **4 wrapper + N sender, tổ hợp tùy ý**. Push "không retry" không cần override để tắt gì cả — đơn giản là *không bọc* RetryNotifier. Mỗi mảnh test độc lập được.

Và bạn vừa **phát minh lại Decorator pattern** — không phải vì học thuộc nó, mà vì nó là điểm đến tự nhiên khi tách tính năng chung ra khỏi cây kế thừa. Đây chính là cách `net/http` middleware hoạt động (Level 4 sẽ đào sâu).

**Embedding của Go — dùng đúng chỗ:**

```go
// Embedding = composition với cú pháp gọn: tự động delegate
type Server struct {
    *http.Server        // embed: Server "có một" http.Server, method được nâng lên
    logger *slog.Logger
}
// s.ListenAndServe() hoạt động — delegate tự động, KHÔNG phải kế thừa:
// không có override ảo, không có super, method của http.Server
// không hề biết nó đang nằm trong struct nào
```

Khác biệt sống còn với inheritance: nếu `http.Server.ListenAndServe` bên trong gọi method khác của nó, nó gọi *phiên bản của chính nó* — không bao giờ gọi "override" của bạn. Không có dynamic dispatch ngược về con → không có fragile base class. Embedding là **tái sử dụng code thuần túy**, tách rời hoàn toàn khỏi polymorphism.

## 5. Polymorphism trong Go — rộng hơn bạn nghĩ

Đa hình = "một điểm gọi, nhiều hành vi". Go có ba cơ chế, chọn theo bài toán:

**(1) Interface (dynamic dispatch)** — khi các hành vi là các *type* khác nhau sống cùng lúc (như Notifier ở trên).

**(2) Function value — đa hình không cần type:**

```go
// Strategy pattern trong Go thường chỉ là... một function
type FeeRule func(o Order) int64

func TotalFee(o Order, rules []FeeRule) int64 {
    var total int64
    for _, r := range rules {
        total += r(o)
    }
    return total
}
```

Khi hành vi không có state và chỉ có một method — **function value là Strategy pattern đã tinh giản**. Đừng tạo interface + 3 struct cho thứ mà 3 hàm giải quyết được. (TypeScript cũng vậy: truyền hàm/closure thay vì class Strategy.)

**(3) Generics (parametric polymorphism, Go 1.18+)** — khi *thuật toán* giữ nguyên còn *kiểu dữ liệu* thay đổi:

```go
func Map[T, U any](xs []T, f func(T) U) []U { /* ... */ }
```

Quy tắc chọn nhanh: hành vi khác nhau lúc **runtime** → interface/function value; kiểu khác nhau lúc **compile-time**, hành vi giống hệt → generics. Dùng generics để giả lập hệ thống class là dùng sai công cụ. (So sánh sâu Interface vs Generics ở Level 4.)

## 6. Trade-off

| | Inheritance | Composition + Interface |
|---|---|---|
| Tái sử dụng | Ngắn gọn lúc đầu, "miễn phí" | Phải viết delegate/wrapper (Go embedding giảm gần hết chi phí này) |
| Coupling | Con↔cha: chặt nhất (state + hành vi + thứ tự ngầm) | Qua interface: lỏng nhất |
| Tổ hợp tính năng | Bùng nổ lớp theo cấp số nhân | Tuyến tính: mỗi tính năng một wrapper |
| Thay đổi runtime | Không — cố định lúc compile | Có — tiêm/tổ hợp tùy ý |
| Chi phí đọc | Yo-yo qua cây tổ tiên | Lần theo chuỗi wrapper (cũng có giá! chuỗi 5 decorator cũng khó trace) |
| Khi hợp lý | Cây nông (≤2 tầng), ổn định, framework kiểm soát cả cây, is-a thật sự về hành vi | Mặc định cho mọi trường hợp còn lại |

Trung thực mà nói: composition không miễn phí. Chuỗi decorator dài cũng khó debug (stack trace sâu, khó biết wrapper nào đang can thiệp). Nhưng nó **thất bại một cách cục bộ** — sửa một wrapper không rung chuyển các wrapper khác, còn inheritance thất bại **một cách lan truyền**.

## 7. Production Examples

- **`io` package (Go)**: `gzip.NewReader(r io.Reader)` trả về thứ cũng là `io.Reader` → bọc chồng vô hạn: `bufio.NewReader(gzip.NewReader(networkConn))`. Toàn bộ hệ sinh thái streaming của Go là composition của interface một-method. Không một `extends` nào.
- **`http.Handler` middleware**: `func(next http.Handler) http.Handler` — mọi framework Go (Gin, Echo, Chi) đều là biến thể của chuỗi composition này.
- **Kubernetes client-go**: `Informer`, `Lister`, `Workqueue` là các mảnh compose vào controller — không có cây `BaseController` để extends; bạn lắp ráp.
- **React (Node.js ecosystem)**: bài học cùng chiều ở frontend — React chính thức khuyến cáo *"use composition instead of inheritance"*; hooks thay thế class inheritance hoàn toàn.
- **NestJS**: ví dụ đối trọng — framework chọn class + decorator + DI container kiểu Angular/Spring. Hoạt động tốt vì *framework sở hữu cả cây* và quy ước chặt. Bài học: inheritance chịu được khi có một chủ thể duy nhất kiểm soát toàn bộ hierarchy — điều hiếm khi đúng với code nghiệp vụ của bạn.

## 8. Anti-pattern

- **Deep inheritance (>2 tầng)**: `BaseService → CrudService → AuditedCrudService → OrderService` — mỗi tầng một ít magic, đọc `OrderService` phải mở 4 file, sửa Base là đánh xổ số.
- **Inheritance để lấy vài method tiện ích**: `class OrderService extends Loggable` — quan hệ is-a giả; hãy inject logger.
- **Embedding interface để "giả" kế thừa trong Go**: embed interface vào struct rồi chỉ implement một phần — phần còn lại panic nil pointer lúc runtime. Embedding interface có chỗ dùng hợp lệ (mock một phần trong test) nhưng trong production code thường là smell.
- **Override để tắt hành vi cha**: như `PushNotifier` ở trên — tín hiệu rõ nhất cho thấy hierarchy đã sai, và là vi phạm Liskov kinh điển.

## 9. Khi nào KHÔNG cần bận tâm

- Hai loại notifier, logic chung 10 dòng? **Copy 10 dòng đó.** Go proverb: *"A little copying is better than a little dependency."* Trừu tượng hóa khi có 3 điểm lặp và chúng thay đổi *cùng lý do* (chi tiết ở DRY, chương 2.6).
- Cây kế thừa nông, ổn định, trong khuôn khổ framework (NestJS controller, GORM model) — theo quy ước framework, đừng chống lại nó.
- Đừng refactor cây kế thừa đang chạy ổn và *không ai phải sửa* — chi phí refactor phải được trả bằng thay đổi tương lai; không có thay đổi thì không có lợi nhuận.

---

*Chương tiếp theo: [1.5 — Interface & Immutability](/series/software-design/level-1-fundamentals/05-interface-immutability/) — đào sâu hai công cụ đã xuất hiện ở mọi chương trước.*
