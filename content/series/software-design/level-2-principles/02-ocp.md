+++
title = "2.2 — Open/Closed Principle: mở rộng mà không sửa"
date = "2026-07-17T08:20:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## 1. Problem Statement

Bài toán quen thuộc đến mức mọi backend engineer đều từng viết nó: **thanh toán qua nhiều cổng**. Ngày đầu chỉ có COD. Rồi thêm Momo. Rồi VNPay, ZaloPay, thẻ quốc tế, trả góp...

```go
// V1 — hoàn toàn ổn khi có 2 phương thức
func ProcessPayment(o *Order) error {
    switch o.PaymentMethod {
    case "cod":
        o.Status = "awaiting_delivery"
        return nil
    case "momo":
        return callMomoAPI(o)
    default:
        return fmt.Errorf("unsupported payment method: %s", o.PaymentMethod)
    }
}
```

18 tháng sau, hàm này 400 dòng, 9 case, mỗi case gọi API khác nhau với retry/timeout khác nhau. Và điều tệ hơn: **cùng cái switch `o.PaymentMethod` đó bị lặp ở 5 nơi khác** — tính phí giao dịch, render UI text, xử lý refund, đối soát, webhook. Thêm ZaloPay nghĩa là tìm đủ 6 cái switch rải rác để thêm case — sót một cái là bug âm thầm (default nuốt lỗi hoặc panic ở production).

Định nghĩa OCP (Bertrand Meyer 1988, Martin phổ biến lại):

> **Software entities should be open for extension, but closed for modification** — thêm hành vi mới bằng cách THÊM code, không phải SỬA code đang chạy.

Vì sao đáng mong muốn? Vì sửa code đang chạy ổn luôn mang rủi ro hồi quy (regression), đòi re-review, re-test những đường đi cũ. Code không bị mở ra sửa thì không thể bị làm vỡ.

## 2. Code Smell — nhận diện chính xác

Tín hiệu cần OCP **không phải** là "có câu switch". Một switch đơn lẻ, sống một chỗ, là code tốt — tường minh, dễ đọc, compiler check (với enum + linter `exhaustive`). Tín hiệu thật là **cụm triệu chứng**:

1. Cùng một phép phân loại (`switch` trên cùng một trường) lặp ở nhiều nơi — mỗi lần thêm loại mới phải "đi săn" đủ mọi bản sao (đây là smell *Shotgun Surgery*).
2. Tần suất thêm loại mới cao và dự đoán tiếp diễn (payment gateway: chắc chắn còn thêm).
3. Mỗi nhánh phình to thành logic phức tạp riêng (retry, config, API riêng).
4. Nhiều người/team khác nhau sở hữu các nhánh khác nhau.

Đủ 2/4 trở lên → đầu tư điểm mở rộng. Chỉ có 1 switch ba case ở một chỗ → để nguyên.

## 3. First Principles

Làm sao *thêm hành vi mà không sửa code cũ* — về mặt cơ chế? Chỉ có một cách: code cũ phải gọi hành vi **qua một điểm gián tiếp** (indirection) mà hành vi mới có thể "cắm" vào. Điểm gián tiếp đó là abstraction: interface, function value, map lookup, hay plugin registry. Nói cách khác: **OCP = xác định trục thay đổi (chương 1.1) rồi đặt một abstraction đúng trên trục đó.**

Hệ quả nghiêm túc: OCP **không thể** đạt được với mọi loại thay đổi. Mỗi abstraction chỉ mở với loại thay đổi nó lường trước (thêm gateway mới: dễ) và vẫn đóng với loại nó không lường (đổi từ "trả ngay" sang "trả góp nhiều kỳ" đổi cả vòng đời giao dịch: vẫn phải mổ). Thiết kế là đặt cược có hiểu biết về *hướng* thay đổi — không phải mua bảo hiểm mọi hướng, vì mua mọi hướng = trừu tượng hóa mọi thứ = codebase không đọc nổi.

## 4. Refactoring Journey

**Bước 1 — gom hành vi mỗi loại về một chỗ (interface consumer-side):**

```go
// V2 — hợp đồng cho MỘT phương thức thanh toán
type Gateway interface {
    Charge(ctx context.Context, o *order.Order) (*Receipt, error)
    Refund(ctx context.Context, r *Receipt) error
    Fee(amount int64) int64
}

type MomoGateway struct{ client *momo.Client; cfg MomoConfig }
func (g *MomoGateway) Charge(ctx context.Context, o *order.Order) (*Receipt, error) { /* API + retry riêng */ }
func (g *MomoGateway) Refund(ctx context.Context, r *Receipt) error { /* ... */ }
func (g *MomoGateway) Fee(amount int64) int64 { return amount * 22 / 1000 }

// CODGateway, VNPayGateway... — mỗi cái MỘT FILE, đổi Momo không mở file VNPay
```

Sáu cái switch rải rác giờ quy về: mọi nơi chỉ gọi `gw.Charge(...)`, `gw.Fee(...)`. Shotgun Surgery biến mất — cohesion theo *loại* thay thế cohesion theo *thao tác*.

**Bước 2 — registry: điểm phân loại CUỐI CÙNG còn lại:**

```go
// V3 — đăng ký một lần, tra cứu bằng data thay vì control flow
type Registry struct{ gws map[string]Gateway }

func (r *Registry) Register(name string, gw Gateway) { r.gws[name] = gw }

func (r *Registry) Get(name string) (Gateway, error) {
    gw, ok := r.gws[name]
    if !ok {
        return nil, fmt.Errorf("unsupported payment method: %q", name)
    }
    return gw, nil
}

// main.go — NƠI DUY NHẤT biết danh sách đầy đủ (composition root)
reg.Register("cod", &CODGateway{})
reg.Register("momo", NewMomoGateway(cfg.Momo))
reg.Register("vnpay", NewVNPayGateway(cfg.VNPay))
```

Giờ "thêm ZaloPay" = **thêm một file mới + một dòng đăng ký**. Không file cũ nào bị mở. PR nhỏ, review dễ, blast radius gần bằng không. Đây là **Strategy pattern + Registry** — xuất hiện tự nhiên như điểm đến của refactoring, đúng tinh thần tài liệu này.

**Biến thể Go thuần cho case đơn giản** — khi hợp đồng chỉ có một thao tác, khỏi cần interface:

```go
var feeRules = map[string]func(amount int64) int64{
    "cod":   func(a int64) int64 { return 15_000 },
    "momo":  func(a int64) int64 { return a * 22 / 1000 },
}
```

**So sánh TypeScript**: cùng cấu trúc, nhưng TS có thêm công cụ mà Go (trước 1.18, và cả sau) không có tương đương: **discriminated union + exhaustiveness check**:

```typescript
type Payment = { kind: "cod" } | { kind: "momo"; walletId: string };

function fee(p: Payment): number {
  switch (p.kind) {
    case "cod":  return 15_000;
    case "momo": return 22;
    default:     const _exhaustive: never = p; return _exhaustive;
    // Thêm kind mới mà quên case → LỖI COMPILE ở MỌI switch
  }
}
```

Điểm thú vị về thiết kế: exhaustive switch là **giải pháp thay thế hợp lệ cho OCP** — bạn *chấp nhận* sửa nhiều chỗ khi thêm loại, đổi lấy việc compiler chỉ tận nơi mọi chỗ cần sửa. "Sửa code cũ" hết đáng sợ khi máy tìm giúp bạn mọi điểm sửa. Functional programming gọi đây là trade-off **sum type vs interface**: interface mở theo *loại* (thêm loại dễ, thêm thao tác khó), sum type mở theo *thao tác* (thêm thao tác dễ, thêm loại phải sửa mọi switch — nhưng có compiler dẫn đường). Đây là "expression problem" kinh điển — chọn theo hướng thay đổi thường xuyên hơn của bài toán bạn.

## 5. Trade-off

| | Switch tập trung | Interface + Registry |
|---|---|---|
| Thêm loại mới | Sửa N chỗ (có thể compiler-checked với union/linter) | Thêm 1 file + 1 dòng đăng ký |
| Thêm thao tác chung mới | Thêm 1 hàm switch mới — dễ | Sửa interface → sửa MỌI implementation |
| Đọc luồng | Một chỗ thấy hết mọi loại | Phải tìm xem implementation nào được đăng ký lúc runtime |
| Debug | Nhảy thẳng vào case | Qua một tầng gián tiếp, "cái gì trong registry?" là câu hỏi runtime |
| Phù hợp | Ít loại, ổn định, một chỗ dùng | Nhiều loại, tăng thường xuyên, nhiều chỗ dùng, nhiều team |

## 6. Production Examples

- **`database/sql` + driver (Go)**: stdlib *đóng* — không sửa khi thế giới thêm DB mới; driver *mở* — Postgres, MySQL, SQLite tự `sql.Register("postgres", &Driver{})` trong `init()`. Đây chính xác là Registry V3, ở quy mô hệ sinh thái, chạy ổn định 12+ năm. Cái giá họ trả: interface `driver` phải cực kỳ ổn định — sửa nó là vỡ mọi driver, nên nó tiến hóa bằng cách *thêm interface tùy chọn* (`driver.QueryerContext`...) và type-assert — một kỹ thuật OCP nâng cao đáng học.
- **`http.Handler` + middleware**: thêm logging/auth/rate-limit vào server mà không sửa handler nào — OCP bằng composition (Decorator), không cần registry.
- **Terraform provider / Kubernetes CRD + controller**: OCP ở mức *process* — mở rộng hệ thống bằng cách deploy binary/manifest mới, core không đổi. Plugin architecture là OCP đẩy tới giới hạn, trả giá bằng versioning hợp đồng giữa core và plugin (gRPC protocol của Terraform provider là một hợp đồng phải giữ tương thích vĩnh viễn).
- **ESLint/Webpack/Fastify (Node.js)**: plugin ecosystem — cùng nguyên lý; Fastify plugin còn kiểm soát cả encapsulation scope, cho thấy plugin system trưởng thành phải giải quyết cả vấn đề cách ly, không chỉ điểm cắm.

## 7. Anti-pattern

- **Speculative Generality**: xây registry + interface cho thứ mãi mãi có một implementation. Bạn trả chi phí gián tiếp hằng ngày cho flexibility không ai dùng. Tín hiệu: interface một implementation, không có test cần fake, và không có kế hoạch cụ thể cho cái thứ hai.
- **Config-driven everything**: đẩy OCP đến mức hành vi định nghĩa trong YAML/DB "để khỏi deploy lại" — bạn vừa xây một ngôn ngữ lập trình tồi không có type-check, không có test, không có git blame. Logic thuộc về code; config nên là *tham số*, không phải *chương trình*.
- **Default nuốt lỗi**: `default: return nil` trong switch phân loại — thêm loại mới quên xử lý và hệ thống *âm thầm* làm sai. Nếu chọn switch, default phải fail to (error/panic có ngữ cảnh).

## 8. Khi nào KHÔNG dùng

Ứng dụng nội bộ bạn sở hữu 100% code và loại mới xuất hiện mỗi năm một lần: mở file sửa switch, chạy test, xong — đơn giản thắng. OCP đắt nhất ở chi phí *đọc* (gián tiếp) và chỉ hoàn vốn khi tần suất mở rộng đủ cao hoặc người mở rộng *không thể* sửa code gốc (thư viện public, plugin bên thứ ba, team khác). Viết thư viện public → OCP gần như bắt buộc. Viết app CRUD nội bộ → exhaustive switch + test tốt là lựa chọn chuyên nghiệp, không phải "thiếu hiểu biết".

---

*Tiếp theo: [2.3 — Liskov Substitution Principle](/series/software-design/level-2-principles/03-lsp/)*
