+++
title = "2.6 — DRY, KISS, YAGNI, Law of Demeter, Tell Don't Ask"
date = "2026-07-17T09:00:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

Năm nguyên tắc "dân gian" — được trích dẫn nhiều nhất và hiểu sai nhiều nhất. Chương này trả mỗi cái về đúng phát biểu gốc, chỉ ra dạng hiểu sai phổ biến, và quan trọng nhất: **chúng mâu thuẫn nhau ở đâu và trọng tài là gì**.

---

## A. DRY — Don't Repeat Yourself

### Phát biểu gốc — và phần bị quên

The Pragmatic Programmer (1999): *"Every piece of **knowledge** must have a single, unambiguous, authoritative representation within a system."*

Từ khóa là **knowledge (tri thức)** — không phải **text (chữ)**. DRY nói về sự trùng lặp của *quyết định nghiệp vụ/thiết kế*, không phải trùng lặp của ký tự trên màn hình. Đây là chỗ 90% việc áp dụng sai bắt nguồn.

### Bài toán — hai loại trùng lặp trông giống hệt nhau

```go
// Nơi 1 — module giỏ hàng
func cartDiscount(total int64) int64 {
    if total >= 1_000_000 { return total * 10 / 100 }
    return 0
}

// Nơi 2 — module hiển thị badge khuyến mãi
func promoBadgeThreshold(total int64) bool {
    return total >= 1_000_000
}
```

Con số `1_000_000` xuất hiện hai lần. Có vi phạm DRY không? **Câu hỏi quyết định: hai chỗ này có bắt buộc thay đổi cùng nhau không?**

- Nếu cả hai cùng là "ngưỡng khuyến mãi mùa hè" — một tri thức, hai bản sao → vi phạm DRY thật. Marketing đổi ngưỡng thành 2 triệu, dev sửa chỗ này quên chỗ kia → badge hiện mà giảm giá không chạy — bug phát sinh từ *trùng lặp tri thức*.
- Nếu ngưỡng giảm giá và ngưỡng hiện badge chỉ *tình cờ* bằng nhau hôm nay (tuần sau marketing muốn badge hiện sớm hơn để "nhử") → **hai tri thức khác nhau**, gộp chung mới là bug: đổi một cái kéo cái kia đổ theo.

Gộp hai thứ tình cờ giống nhau gọi là **incidental duplication trap** — và thuốc giải là câu thần chú của Sandi Metz: *"Duplication is far cheaper than the wrong abstraction."* Trùng lặp sai thì sửa dễ (gộp lại); abstraction sai thì gỡ rất đau — mọi caller đã dính vào nó, và nó thường được "vá" bằng cờ boolean và tham số tùy chọn cho đến khi thành God Function.

```go
// ❌ Hậu quả kinh điển của DRY mù: hàm "chung" phục vụ 2 chủ nhân
func threshold(kind string, isVIP bool, isSummerSale bool) int64 {
    // 2 năm sau: 6 tham số, 5 nhánh if, không ai dám sửa
}
```

### Quy tắc thực dụng

- **Rule of Three**: chấp nhận bản sao thứ 2; đến bản thứ 3 *và* xác nhận chúng đổi cùng lý do → mới trừu tượng hóa.
- DRY khẩn cấp nhất với: **hằng số nghiệp vụ, công thức tính tiền, validation rule, schema** — nơi hai bản sao lệch nhau tạo bug nghiệp vụ trực tiếp.
- DRY lỏng tay nhất với: **test code** (mỗi test tự chứa, dễ đọc, hơn là xây "test framework" mini), **code khởi tạo/wiring**, và **giữa hai microservice** (chia sẻ thư viện nghiệp vụ giữa hai service = coupling deploy — thường tệ hơn copy).
- Go proverb tương ứng: *"A little copying is better than a little dependency."*

---

## B. KISS & YAGNI — cặp phanh hãm

### KISS — Keep It Simple

Không phải "viết code ít dòng" mà là **chọn giải pháp ít khái niệm nhất đủ giải bài toán**. Thước đo tốt: người mới vào team đọc hiểu trong bao lâu; on-call lúc 3h sáng trace được luồng trong bao lâu.

Phân biệt quan trọng (mượn Rich Hickey): **simple** (ít đan xen, mỗi mảnh một vai) ≠ **easy** (ít gõ phím lúc viết). ORM auto-magic là *easy* nhưng không *simple* (N+1 query ẩn, lazy loading bất ngờ). Viết SQL tường minh *khó* hơn lúc viết nhưng *đơn giản* hơn lúc vận hành. KISS đúng nghĩa tối ưu cho **người đọc và người vận hành**, không phải người viết.

### YAGNI — You Aren't Gonna Need It

Từ Extreme Programming: **đừng xây cho nhu cầu phỏng đoán**. Lý do kinh tế học chứ không phải lười: (a) dự đoán nhu cầu tương lai của dev sai nhiều hơn đúng — bằng chứng là mọi codebase đều có "phần mở rộng cho tương lai" không bao giờ dùng; (b) tính năng xây sớm phải **bảo trì từ hôm nay** — mỗi lần refactor phải khiêng theo nó, mỗi người mới phải học nó; (c) xây khi cần thật luôn rẻ hơn vì lúc đó có thông tin đầy đủ hơn.

YAGNI áp cho **tính năng và flexibility**, KHÔNG áp cho: chất lượng nền (test, log, error handling), và những quyết định **không thể đảo ngược rẻ** (schema DB public, API công bố ra ngoài, wire format) — với nhóm sau, suy nghĩ trước là bắt buộc vì chi phí sửa sau là khổng lồ. Kỹ năng thật của architect: phân loại quyết định *đảo ngược được* (áp YAGNI thẳng tay) và *không đảo ngược được* (đầu tư thiết kế trước).

### Khi các nguyên tắc đánh nhau — và trọng tài

- **DRY vs KISS**: gộp 3 đoạn giống nhau tạo abstraction phức tạp hơn tổng 3 đoạn → KISS thắng, giữ bản sao.
- **OCP vs YAGNI**: xây điểm mở rộng cho loại thanh toán thứ N khi mới có 1 loại → YAGNI thắng; đến loại thứ 2-3 → OCP thắng (chương 2.2 đã cho tiêu chí đếm được).
- **Trọng tài chung**: quay về hàm mục tiêu duy nhất của chương 1.1 — **tổng chi phí thay đổi (hiểu + lan truyền + kiểm chứng + phối hợp), chiết khấu theo xác suất tương lai đó xảy ra**. Nguyên tắc nào giảm tổng đó trong ngữ cảnh cụ thể thì nguyên tắc đó thắng. Nguyên tắc là heuristic; hàm chi phí là sự thật.

---

## C. Law of Demeter — "chỉ nói chuyện với hàng xóm"

### Bài toán

```go
// ❌ chuỗi chấm xuyên 4 tầng object graph
total := order.GetCustomer().GetMembership().GetTier().GetDiscountRate() * order.Subtotal()
```

Phát biểu (1987): một method chỉ nên gọi method của — chính nó; tham số của nó; object nó tạo ra; field trực tiếp của nó. Tức là: **với hàng xóm trực tiếp, đừng với qua hàng xóm để nắn đồ trong nhà hàng xóm của hàng xóm**.

Vì sao chuỗi trên đắt — bằng ngôn ngữ 1.3: caller giờ coupling với **cấu trúc** của 4 type (Order chứa Customer chứa Membership chứa Tier). Đổi bất kỳ khớp nào (membership chuyển sang tính theo điểm, bỏ khái niệm Tier) → vỡ **mọi** chuỗi chấm xuyên qua khớp đó, rải khắp codebase (Shotgun Surgery). Đây là smell *Message Chains* (Level 3).

### Refactor — che cấu trúc sau hành vi

```go
// ✅ Order tự trả lời câu hỏi thuộc phạm vi của nó
func (o *Order) Total() int64 {
    return o.subtotal - o.customer.DiscountFor(o.subtotal)
}
// Customer che giấu cách nó suy ra discount — từ membership, điểm, hay gì đi nữa
func (c *Customer) DiscountFor(amount int64) int64 { /* ... */ }
```

Cấu trúc "ai chứa ai" giờ là bí mật của từng lớp — đổi ruột Membership chỉ đụng `Customer.DiscountFor`.

### Ranh giới quan trọng: LoD áp cho OBJECT, không áp cho DATA

```go
resp.Data.Items[0].Price      // ✅ DTO/JSON thuần — cấu trúc CHÍNH LÀ hợp đồng, chấm thoải mái
cfg.Server.HTTP.Port          // ✅ config struct — data, không phải object có hành vi
order.GetCustomer().GetMembership()...  // ❌ object có hành vi + invariant — che đi
```

Đếm dấu chấm là cách đọc LoD sai và gây ám ảnh vô ích. Câu hỏi đúng: *thứ bị chấm xuyên qua là data (cấu trúc là hợp đồng công khai) hay object (cấu trúc là chi tiết cài đặt)?* Fluent API (`strings.Builder`, query builder — mỗi call trả chính nó) cũng **không** vi phạm: bạn vẫn nói chuyện với một object.

---

## D. Tell, Don't Ask — mặt kia của cùng đồng xu

### Bài toán

```go
// ❌ ASK: hỏi trạng thái, tự quyết định, tự cập nhật — logic của Account tràn ra ngoài
if acc.Balance() >= amount && !acc.IsFrozen() {
    acc.SetBalance(acc.Balance() - amount)
}
```

Ba vấn đề chồng nhau: logic "rút tiền hợp lệ" sống ở **caller** — caller thứ hai sẽ copy nó (DRY violation về tri thức); invariant của Account bị quyết định từ bên ngoài (encapsulation thủng, chương 1.2); và giữa `Balance()` và `SetBalance()` là **race condition** — check-then-act không nguyên tử.

```go
// ✅ TELL: ra lệnh, object tự bảo vệ invariant của nó — nguyên tử, một nơi, mọi caller hưởng
func (a *Account) Withdraw(amount int64) error {
    a.mu.Lock()
    defer a.mu.Unlock()
    if a.frozen {
        return ErrAccountFrozen
    }
    if a.balance < amount {
        return ErrInsufficientFunds
    }
    a.balance -= amount
    return nil
}
```

Tell-Don't-Ask là LoD nhìn từ phía nhận: LoD bảo "đừng với vào trong", TDA bảo "hãy để tôi tự làm". Cả hai cùng đẩy về một đích: **hành vi sống cạnh dữ liệu mà nó bảo vệ** — functional cohesion (1.3) và là thuốc trực tiếp cho Anemic Domain Model (2.1).

Giới hạn áp dụng: quy tắc này dành cho **domain object có invariant**. Query/report (đọc dữ liệu để hiển thị) thì *ask* là bản chất công việc — đừng nhét method render vào domain object nhân danh TDA. CQRS (Level 5) chính là thừa nhận kiến trúc của ranh giới này: đường ghi thì Tell, đường đọc thì Ask.

---

## Tổng kết Level 2

Xâu chuỗi lại toàn bộ — mọi nguyên tắc là một câu trả lời cho **"thay đổi lan đến đâu?"**:

| Nguyên tắc | Một câu đọng lại |
|---|---|
| SRP | Tách theo *lý do thay đổi* (ai yêu cầu sửa), không phải theo "số việc" |
| OCP | Đặt abstraction đúng *trục thay đổi đã có bằng chứng*; exhaustive switch là lựa chọn thay thế hợp lệ |
| LSP | Interface là *hợp đồng ngữ nghĩa*; contract test cho mọi implementation, kể cả fake |
| ISP | Interface nhỏ, đặt phía consumer — Go cho không tính năng này, hãy dùng |
| DIP | Nghiệp vụ định nghĩa interface, hạ tầng implement; kiểm tra bằng khối import |
| DRY | Một *tri thức* một nơi — không phải một *chuỗi ký tự* một nơi |
| KISS/YAGNI | Phanh hãm của mọi nguyên tắc trên; flexibility chưa có bằng chứng = nợ phải nuôi |
| LoD/TDA | Che cấu trúc sau hành vi; object có invariant thì ra lệnh, đừng hỏi |

Bạn giờ có đủ **ngôn ngữ nguyên lý** để bước vào Level 3 (Refactoring — nhận diện smell và gỡ từng bước) và Level 4 (Design Patterns — nơi các nguyên lý này kết tinh thành cấu trúc có tên). Như đã hứa từ đầu: đến lúc gặp Strategy, Decorator, Factory... bạn sẽ nhận ra mình **đã dùng chúng rồi** trong các Refactoring Journey vừa qua — giờ chỉ còn gọi đúng tên và biết đủ biến thể.

---

*Phần tiếp theo của bộ tài liệu: Level 3 — Code Smells & Refactoring Techniques.*
