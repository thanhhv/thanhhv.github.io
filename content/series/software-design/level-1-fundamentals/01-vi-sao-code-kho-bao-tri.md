+++
title = "1.1 — Vì sao code trở nên khó bảo trì: bản chất của Software Design"
date = "2026-07-17T07:10:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## 1. Problem Statement

Hãy bắt đầu bằng một sự thật mà mọi kỹ sư có kinh nghiệm đều biết nhưng ít khi nói thành lời:

> **Chi phí của phần mềm không nằm ở việc viết code lần đầu. Nó nằm ở việc thay đổi code sau đó.**

Một hệ thống backend điển hình sống 5–10 năm. Trong vòng đời đó, code được **đọc** nhiều gấp 10 lần được viết, và được **sửa** nhiều gấp nhiều lần được viết mới. Nghiên cứu kinh điển về chi phí phần mềm ước tính 60–80% tổng chi phí là maintenance — không phải development ban đầu.

Vậy Software Design là gì? Không phải UML. Không phải pattern. Định nghĩa thực dụng nhất:

> **Software Design là nghệ thuật tổ chức code sao cho chi phí thay đổi trong tương lai là thấp nhất, với chi phí hiện tại chấp nhận được.**

Hai vế của câu này quan trọng như nhau. Vế đầu chống lại code rối. Vế sau chống lại over-engineering.

## 2. Một câu chuyện thực tế: hệ thống tính phí vận chuyển

Xem một ví dụ sẽ theo chúng ta xuyên suốt tài liệu. Công ty bạn làm e-commerce, cần tính phí ship.

**Tuần 1 — Yêu cầu**: phí ship = 20.000đ, đơn trên 500.000đ miễn phí.

```go
// V1 — hoàn toàn hợp lý cho yêu cầu hiện tại
func ShippingFee(orderTotal int64) int64 {
    if orderTotal >= 500_000 {
        return 0
    }
    return 20_000
}
```

Code này **tốt**. Đơn giản, đọc 3 giây hiểu ngay, test 2 case là đủ. Đừng để ai nói với bạn rằng nó cần một `ShippingFeeCalculatorFactory`.

**Tháng 2 — Business thay đổi**: thêm giao nhanh (express) giá gấp đôi.

```go
// V2 — thêm một tham số, vẫn ổn
func ShippingFee(orderTotal int64, express bool) int64 {
    if orderTotal >= 500_000 {
        return 0
    }
    if express {
        return 40_000
    }
    return 20_000
}
```

**Tháng 6 — Business thay đổi tiếp**: khu vực ngoại thành +10.000đ; khách VIP giảm 50% phí ship nhưng không áp dụng cho express; chương trình khuyến mãi tháng 6 miễn phí ship cho đơn đầu tiên; đơn hàng nặng trên 5kg tính phụ phí theo kg.

```go
// V3 — ❌ và đây là lúc mọi thứ bắt đầu sụp đổ
func ShippingFee(orderTotal int64, express bool, district string,
    customerType string, isFirstOrder bool, weightKg float64,
    now time.Time) int64 {

    if isFirstOrder && now.Month() == time.June {
        return 0
    }
    fee := int64(20_000)
    if express {
        fee = 40_000
    }
    if district != "inner" {
        fee += 10_000
    }
    if weightKg > 5 {
        fee += int64((weightKg - 5) * 5_000)
    }
    if customerType == "vip" && !express {
        fee = fee / 2
    }
    if orderTotal >= 500_000 {
        // Khoan... miễn phí có áp dụng cho phụ phí cân nặng không?
        // Business chưa nói. Dev tự quyết định. Bug tương lai ra đời ở đây.
        return 0
    }
    return fee
}
```

Hãy quan sát điều gì vừa xảy ra — vì đây chính là cách **mọi** codebase trở nên khó bảo trì:

1. **Không ai viết code tệ ngay từ đầu.** V1 hoàn hảo. V2 vẫn ổn. Sự xuống cấp diễn ra từng bước nhỏ, mỗi bước đều "hợp lý" tại thời điểm đó.
2. **Thứ tự các câu `if` bắt đầu mang ngữ nghĩa business ngầm.** VIP giảm 50% *trước* hay *sau* khi cộng phụ phí ngoại thành? Kết quả khác nhau. Ngữ nghĩa đó không được viết ra ở đâu cả — nó chỉ tồn tại trong thứ tự dòng code.
3. **Tham số phình ra** (7 tham số), và mỗi caller phải biết truyền gì — kể cả những caller không quan tâm đến cân nặng hay khuyến mãi.
4. **Test bùng nổ tổ hợp**: 2 × 2 × 2 × 2 × 2 × (vài mức cân nặng) — hàng chục case để cover một hàm.
5. **Mỗi yêu cầu mới đều sửa đúng một hàm này** — team 5 người thì hàm này thành điểm nghẽn conflict git, và mỗi lần sửa đều có nguy cơ phá vỡ logic cũ.

## 3. First Principles: chi phí thay đổi đến từ đâu?

Bóc tách vấn đề đến tận gốc, chi phí thay đổi một đoạn code gồm 4 thành phần:

**(1) Chi phí hiểu (comprehension cost)** — trước khi sửa, bạn phải đọc và dựng lại mô hình trong đầu: hàm này làm gì, thứ tự nào quan trọng, giả định ngầm nào đang tồn tại. Ở V3, chi phí này đã cao: bạn phải đọc *toàn bộ* hàm để sửa *bất kỳ* rule nào, vì các rule đan vào nhau.

**(2) Chi phí thay đổi lan truyền (ripple cost)** — sửa chỗ này có phải sửa chỗ khác không? Thêm tham số thứ 8 vào `ShippingFee` nghĩa là sửa mọi call site. Đây là hệ quả trực tiếp của **coupling** (chương 1.3).

**(3) Chi phí kiểm chứng (verification cost)** — làm sao biết mình không phá gì? Hàm V3 không thể test từng rule độc lập; mọi test đều phải đi qua toàn bộ rừng `if`. Đây là hệ quả của **cohesion thấp**: một đơn vị code chứa nhiều lý do thay đổi.

**(4) Chi phí phối hợp (coordination cost)** — bao nhiêu người phải đụng vào cùng một chỗ? Một hàm là nơi hội tụ của mọi thay đổi thì trở thành điểm nghẽn của cả team.

Mọi nguyên lý thiết kế (SOLID, DRY...), mọi pattern (Strategy, Decorator...) mà bạn sẽ gặp trong tài liệu này đều chỉ là **các chiến thuật khác nhau để giảm 4 loại chi phí trên**. Nếu một pattern không giảm được chi phí nào trong 4 loại này cho ngữ cảnh của bạn — nó là over-engineering.

## 4. Hai lực kéo ngược chiều

Mọi quyết định thiết kế đều nằm giữa hai lực:

```
  Under-engineering ◄──────────────────────► Over-engineering
  (rối, cứng, sợ sửa)                        (trừu tượng thừa, gián tiếp thừa)

  V3 ở trên: mọi logic          Một "ShippingFeeStrategyFactoryProvider"
  trộn trong một hàm,           với 12 interface cho một business
  thêm rule = sửa cả rừng if    chỉ có 2 rule tính phí
```

Cả hai đầu đều đắt. Điểm khác biệt: chi phí under-engineering **tăng dần và lộ ra khi thay đổi**; chi phí over-engineering **trả trước và trả mãi** (mỗi lần đọc code phải nhảy qua 5 tầng gián tiếp).

Kỹ sư giỏi không phải người luôn thiết kế "linh hoạt nhất", mà là người **đoán đúng trục thay đổi** (axis of change): thứ gì trong hệ thống này sẽ thay đổi thường xuyên, thứ gì gần như không bao giờ. Đầu tư flexibility vào đúng trục thay đổi, giữ mọi thứ khác đơn giản tối đa.

Với bài toán phí ship: *các rule tính phí* rõ ràng là trục thay đổi (business đổi 4 lần trong 6 tháng). Còn *chữ ký "nhận đơn hàng, trả về số tiền"* gần như bất biến. Thiết kế tốt sẽ làm cho việc thêm rule rẻ, và giữ phần còn lại đơn giản. Đó chính là con đường dẫn đến Strategy/Chain of Responsibility mà ta sẽ đi **từng bước** ở các chương sau — không nhảy cóc.

## 5. So sánh Go và TypeScript ngay từ đầu

Một điểm cần thiết lập sớm vì nó ảnh hưởng đến mọi chương sau:

| Khía cạnh | Go | TypeScript/Node.js |
|---|---|---|
| Triết lý | "A little copying is better than a little dependency" — ưu tiên đơn giản, tường minh | Linh hoạt, nhiều paradigm, hệ sinh thái nghiêng về DI container, decorator |
| Interface | Implicit (structural) — type tự thỏa mãn interface, không cần `implements` | Structural nhưng thường dùng tường minh `implements`; interface biến mất khi runtime |
| Kế thừa | **Không có** class inheritance — buộc bạn dùng composition | Có `extends`, cám dỗ deep inheritance vẫn tồn tại |
| Hệ quả cho pattern | Nhiều GoF pattern đơn giản hóa hoặc biến mất (Strategy = function value, Template Method hiếm dùng) | Pattern GoF ánh xạ trực tiếp hơn, framework (NestJS) áp đặt sẵn nhiều pattern |

Điều này nghĩa là: **học pattern qua Go buộc bạn hiểu bản chất**, vì Go không cho bạn công cụ để bê nguyên cấu trúc class từ sách GoF (viết cho C++/Smalltalk năm 1994) vào code.

## 6. Trade-off & Kết luận chương

- Software Design = tối ưu **chi phí thay đổi tương lai** với **chi phí hiện tại chấp nhận được**. Cả hai vế đều quan trọng.
- Code tệ không sinh ra từ dev tệ, mà từ **sự tích tụ của các thay đổi nhỏ hợp lý** không kèm refactoring.
- 4 loại chi phí: hiểu, lan truyền, kiểm chứng, phối hợp. Mọi nguyên lý và pattern là chiến thuật giảm các chi phí này.
- Nghệ thuật nằm ở việc **đoán đúng trục thay đổi** — không phải flexibility tối đa ở mọi nơi.
- V1 với 2 câu `if` là thiết kế **đúng** cho thời điểm của nó. Đừng bao giờ xấu hổ vì code đơn giản.

**Khi nào KHÔNG cần quan tâm những điều trong tài liệu này**: script chạy một lần, prototype sẽ vứt đi, MVP đang tìm product-market fit mà tốc độ ra tính năng quyết định sống còn. Trong các ngữ cảnh đó, V3 xấu xí ở trên có thể là quyết định kinh doanh đúng — miễn là bạn **biết** mình đang vay nợ (technical debt) và có ý thức trả khi hệ thống sống sót.

---

*Chương tiếp theo: [1.2 — Abstraction & Encapsulation](/series/software-design/level-1-fundamentals/02-abstraction-encapsulation/) — công cụ đầu tiên để giảm chi phí hiểu.*
