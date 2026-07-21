+++
title = "4.9 — Command, Chain of Responsibility & Mediator"
date = "2026-07-17T11:30:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

Ba pattern behavioral về **tổ chức lời gọi**: Command biến lời gọi thành *dữ liệu*; Chain of Responsibility cho lời gọi *chạy qua một hàng người xử lý*; Mediator gom các lời gọi chéo nhau về *một trung tâm điều phối*.

---

## Phần A — Command

### 1. Bài toán — khi lời gọi hàm cần được cầm nắm

Lời gọi hàm thường: xảy ra ngay, không lưu được, không xếp hàng được, không hoàn tác được, không gửi đi xa được. Một loạt nhu cầu thực tế đòi hỏi ngược lại:

- **Job queue**: "gửi email này" cần *xếp hàng, retry, chạy bởi worker khác* — lời gọi phải sống lâu hơn request tạo ra nó.
- **Audit/undo**: thao tác admin cần ghi lại *ai làm gì với tham số nào*, có khi cần hoàn tác.
- **Scheduling**: "chạy việc này lúc 2h sáng" — lời gọi phải tồn tại trước khi được thực thi.

**Intent (GoF)**: đóng gói một yêu cầu thành **object** — từ đó tham số hóa, xếp hàng, ghi log, hoàn tác được.

### 2. Hình dạng hiện đại — closure cho gần, dữ liệu cho xa

GoF cần `interface Command { execute() }` + N class. Ngôn ngữ có closure thì **một lời gọi đóng gói = một closure**:

```go
// Command trong process = closure. Hết chuyện.
type Task func(ctx context.Context) error

queue <- func(ctx context.Context) error {   // đóng gói "gửi mail cho đơn 123"
    return mailer.SendConfirmation(ctx, orderID)
}
// worker: for t := range queue { t(ctx) }  — retry/log bọc quanh t như decorator
```

Nhưng closure **không serialize được** — không sống qua restart, không gửi sang process khác. Khi command phải đi xa hoặc bền, nó phải là **dữ liệu thuần + tên**:

```go
// Command bền = struct dữ liệu + registry handler (4.1) — chính là mô hình MỌI job queue
type SendConfirmation struct {
    OrderID string `json:"order_id"`
}

// đăng ký: tên lệnh → cách thực thi
registry.Register("send_confirmation", func(ctx context.Context, raw []byte) error {
    var cmd SendConfirmation
    if err := json.Unmarshal(raw, &cmd); err != nil { return err }
    return mailer.SendConfirmation(ctx, cmd.OrderID)
})
```

Đây chính xác là kiến trúc của **asynq, River, BullMQ, Sidekiq, Temporal activity** — Command pattern là *xương sống của mọi job queue* mà bạn dùng hằng ngày không gọi tên. Và ngay khi command thành dữ liệu-đi-xa, ba nghĩa vụ phát sinh (không phải tùy chọn): **idempotency** (queue giao at-least-once — handler chạy hai lần phải vô hại), **versioning** (command nằm trong queue lúc deploy code mới — field đổi là nổ), **không nhét cả object nghiệp vụ** (nhét `OrderID` để handler load bản mới nhất, đừng nhét nguyên `Order` — dữ liệu cũ lúc thực thi).

Còn `Undo` — phần nổi tiếng nhất của Command trong sách — nói thẳng: đất của nó là **editor/UI** (mỗi command giữ inverse, stack undo/redo). Backend hiện đại hiếm khi undo bằng inverse command; nó dùng **compensation trong Saga** (4.13) — cùng ý tưởng "hành động có hành động bù", khung khác.

### 3. Khi nào KHÔNG dùng

Gọi được ngay và không cần lưu/xếp/gửi → gọi hàm. Command-hóa mọi use case (`CreateOrderCommand` + `CommandBus` cho app CRUD monolith) là nghi lễ CQRS-cargo-cult — gián tiếp thêm một tầng để... gọi hàm trong cùng process. CQRS thật có lý do riêng (Level 5); "mọi thứ qua bus cho đồng bộ kiến trúc" không phải lý do.

---

## Phần B — Chain of Responsibility

### 1. Bài toán & hình dạng

**Intent (GoF)**: chuỗi người xử lý; yêu cầu chạy dọc chuỗi đến khi **một người nhận xử lý** — người gửi không biết ai sẽ xử lý.

Điểm cần nói thẳng: dạng *thuần GoF* (đúng-một-người-xử-lý) trong backend ngày nay khá hiếm — còn dạng *biến thể* thì bạn đã gặp và dùng rồi: **middleware (4.5) chính là CoR biến thể** "ai cũng xử lý một phần + ai cũng có quyền chặn". Vì vậy phần này chỉ mổ chỗ dạng thuần còn sống tốt:

```go
// Duyệt chi phí: đơn < 5tr trưởng nhóm duyệt; < 50tr trưởng phòng; còn lại CFO
type Approver interface {
    Approve(ctx context.Context, r ExpenseRequest) (Decision, error)
}

type teamLead struct{ next Approver }
func (a teamLead) Approve(ctx context.Context, r ExpenseRequest) (Decision, error) {
    if r.Amount < 5_000_000 {
        return Decision{By: "team_lead", OK: true}, nil     // TÔI xử lý — dừng chuỗi
    }
    return a.next.Approve(ctx, r)                            // không phải việc tôi — chuyển
}
/* deptHead, cfo tương tự; cfo là cuối chuỗi — KHÔNG next */
```

Cùng họ: escalation policy (PagerDuty), fallback provider (gọi gateway A, lỗi thì B, lỗi nữa thì C), phân giải handler theo loại request. Đặc trưng nhận dạng so với middleware: **mỗi mắt xích hoặc-xử-lý-hoặc-chuyển**, không phải ai-cũng-góp-một-tay.

### 2. Hai quyết định thiết kế thật

**Rơi hết chuỗi thì sao?** — bắt buộc trả lời tường minh: default handler cuối chuỗi (từ chối/báo lỗi có ngữ cảnh), đừng để nil pointer hay im lặng nuốt. **Chuỗi lắp ở đâu?** — một nơi (composition root), theo config/data khi thứ tự là chính sách nghiệp vụ (bảng ngưỡng duyệt trong DB → dựng chuỗi từ bảng — lúc này CoR + data-driven cho phép đổi chính sách không deploy).

Và phép thử ngược trước khi dựng chuỗi: bài duyệt chi ở trên *thực ra* viết được bằng một hàm với bảng ngưỡng — `for _, tier := range tiers { if r.Amount < tier.Limit { return tier.Decide(r) } }`. CoR chỉ thắng bảng-trong-vòng-lặp khi các mắt xích **khác nhau về bản chất xử lý** (không quy về cùng một hàm ngưỡng) hoặc **thuộc sở hữu các module khác nhau**. Không thì — bảng thắng, đơn giản hơn nhiều.

---

## Phần C — Mediator

### 1. Bài toán — n² đường chéo

Nhiều thành phần ngang hàng tương tác chéo nhau: form UI (field này đổi → field kia disable → nút kia enable), hoặc backend: các bước trong quy trình phức tạp gọi nhau tay đôi. Mỗi cặp một đường nói chuyện → n thành phần, n² đường, mỗi thành phần biết mọi thành phần khác — coupling lưới.

**Intent (GoF)**: gom tương tác về **một trung tâm**; các thành phần chỉ nói chuyện với mediator, không nói với nhau. Lưới n² thành hình sao n đường.

### 2. Đánh giá trung thực — pattern dễ thành nhãn dán

Mediator là pattern *mô tả* nhiều hơn *chỉ dẫn* — vì "gom điều phối về một chỗ" chính là điều bạn đã làm suốt tài liệu này dưới các tên khác: **use case/Facade `OrderCompletion` (4.4) là mediator** giữa inventory/shipping/payment; **Saga orchestrator (4.13) là mediator** cho giao dịch phân tán; **event bus có điều phối** là mediator cho observer. Giá trị còn lại của cái tên nằm ở hai điều:

- **Chẩn đoán**: thấy các module ngang hàng import chéo nhau gọi tay đôi (order gọi inventory, inventory gọi ngược order, cả hai gọi shipping...) — tên bệnh là "thiếu mediator", thuốc là kéo các lời gọi chéo lên một tầng điều phối đứng trên; các module hạ xuống thành thụ động, được-gọi.
- **Cảnh báo trade-off**: mediator hút toàn bộ logic tương tác về mình — n² coupling biến mất khỏi các thành phần nhưng **độ phức tạp không biến mất, nó dọn nhà vào mediator**. Không kỷ luật thì mediator = God Object có giấy phép (3.1). Giữ nó sống sót: mediator chỉ chứa *trình tự và điều kiện phối hợp*; mọi logic nghiệp vụ của từng phần ở nguyên trong phần đó.

MediatR (.NET) và các "mediator" của thế giới NestJS/CQRS-lite thực chất dùng tên này cho **command bus** (dispatch request → handler, một-một) — gần Command hơn Mediator GoF; biết để khỏi lẫn khi đọc tài liệu đa hệ sinh thái.

---

## Tổng kết chương — ba pattern, một bảng định vị

| | Hình dạng lời gọi | Ai biết ai | Khi nào cầm đến |
|---|---|---|---|
| **Command** | Lời gọi → **object/dữ liệu** | Người tạo không cần biết người thực thi (registry nối) | Cần xếp hàng, lưu bền, retry, audit, gửi xa |
| **Chain of Responsibility** | Lời gọi → **chạy dọc hàng** | Mỗi mắt xích chỉ biết mắt kế | Nhiều ứng viên xử lý, chọn theo thứ tự ưu tiên; fallback/escalation |
| **Mediator** | Lời gọi chéo → **qua trung tâm** | Chỉ mediator biết tất cả | Tương tác lưới n² giữa các thành phần ngang hàng |

Cả ba đều đổi *lời gọi trực tiếp* lấy *gián tiếp có tổ chức* — và vì thế cả ba đều chung một anti-pattern gốc: dùng khi lời gọi trực tiếp vẫn ổn. Một hàm gọi một hàm là điều tốt đẹp bình thường; chỉ nâng cấp thành pattern khi lời gọi cần một *năng lực* mà lời gọi thẳng không có (bền/xếp hàng — Command; chọn người xử lý — CoR; gỡ lưới — Mediator).

---

*Tiếp theo: [4.10 — Template Method, Visitor, Iterator & Memento](/series/software-design/level-4-patterns/10-template-visitor-iterator-memento/)*
