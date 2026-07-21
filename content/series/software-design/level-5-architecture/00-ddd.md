+++
title = "5.0 — Domain-Driven Design: thiết kế theo nghiệp vụ, không theo công nghệ"
date = "2026-07-17T12:20:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## 1. Problem Statement — căn bệnh mà DDD sinh ra để chữa

Level 1–4 làm việc ở quy mô hàm, struct, package. Level 5 hỏi câu hỏi quy mô hệ thống — và bắt đầu bằng căn bệnh phổ biến nhất của hệ thống lớn, một căn bệnh **không phải về code**:

> Team kỹ thuật và team nghiệp vụ nói hai ngôn ngữ khác nhau về cùng một thứ.

Triệu chứng cụ thể, quen thuộc đến đau lòng: PM nói "khách hàng", codebase có `User`, `Account`, `Customer`, `Client`, `Member` — năm tên cho không-rõ-mấy khái niệm; bảng `orders` có cột `status` 14 giá trị trong đó 5 giá trị "không còn dùng nhưng không ai dám xóa"; tính năng mới cần ba cuộc họp để thống nhất "ý anh nói *đơn hủy* là hủy kiểu nào?"; và bug nghiệp vụ sinh ra không phải vì code sai thuật toán, mà vì **dev hiểu sai luật chơi của domain**.

Eric Evans (*Domain-Driven Design*, 2003) chẩn đoán: với phần mềm có nghiệp vụ phức tạp, **độ phức tạp lớn nhất nằm ở domain, không nằm ở công nghệ** — và do đó thiết kế phải xoay quanh **mô hình domain** được cả dev lẫn chuyên gia nghiệp vụ cùng xây, cùng nói. DDD gồm hai nửa — và thứ tự quan trọng:

- **Strategic design** (nửa quan trọng hơn, hay bị bỏ qua): chia hệ thống theo ranh giới nghiệp vụ — bounded context, ubiquitous language, context mapping.
- **Tactical design** (nửa nổi tiếng hơn, hay bị cargo-cult): các viên gạch trong một context — entity, value object, aggregate, repository, domain event.

Sự thật trớ trêu của ngành: đa số "dự án DDD" chỉ bê tactical patterns (repository! aggregate!) vào codebase mà bỏ qua strategic — tức là mua bộ lego mà vứt bản thiết kế. Chương này đi đúng thứ tự.

## 2. Strategic Design — nửa đáng tiền nhất

### Ubiquitous Language — ngôn ngữ chung, có thi hành

Không phải "glossary trong wiki". Là kỷ luật: **từ vựng nghiệp vụ mà PM/domain expert dùng phải xuất hiện NGUYÊN VĂN trong code** — tên type, method, event, và ngược lại: dev không phát minh từ riêng. PM nói "đơn chờ đối soát" → code có `AwaitingReconciliation`, không phải `Status7` hay `PendingCheck`. Khi PM đổi cách gọi, code refactor-rename theo.

Vì sao đáng đầu tư — nhìn bằng khung 1.1: **chi phí dịch thuật giữa hai ngôn ngữ là chi phí hiểu trả mãi mãi**, ở mọi cuộc họp, mọi lần đọc code, mọi lần onboard người mới; và mỗi lần dịch là một cơ hội hiểu sai (bug nghiệp vụ). Level 3 đã có công cụ thi hành: rename là refactoring rẻ nhất (3.3) — dùng nó thường xuyên theo ngôn ngữ nghiệp vụ. Các event `OrderConfirmed`, `ShipmentDelayed` (4.13) chính là ubiquitous language ở dạng thuần khiết nhất.

### Bounded Context — phát hiện trung tâm của DDD

Quan sát nền tảng: **cùng một từ mang nghĩa khác nhau ở các mảng nghiệp vụ khác nhau — và điều đó ĐÚNG, đừng chống lại**:

```
"Sản phẩm" trong Catalog:    tên, mô tả, ảnh, SEO, danh mục
"Sản phẩm" trong Kho:        mã SKU, vị trí kệ, số lượng, hạn dùng
"Sản phẩm" trong Vận chuyển: cân nặng, kích thước, dễ vỡ, nguy hiểm
"Sản phẩm" trong Kế toán:    giá vốn, thuế suất, mã ngành hàng
```

Cách chống lại — xây MỘT class `Product` "đầy đủ" dùng chung — cho ra God Object 60 field (3.1) mà mọi team cùng sửa (mọi bệnh coupling của 1.3), và mô hình không khớp với *bất kỳ* mảng nghiệp vụ nào. Cách thuận theo — **bounded context**: mỗi mảng có mô hình `Product` **riêng**, tối ưu cho nghiệp vụ của nó, với ranh giới tường minh; các context liên lạc qua hợp đồng (event, API), không chia sẻ model.

Nhận ra điều quen thuộc: bounded context là **SRP + cohesion (2.1, 1.3) nâng lên quy mô hệ thống** — "một mô hình, một ngữ cảnh nghiệp vụ, một team, một nhịp thay đổi". Và nó là *tiền đề* của microservices đúng nghĩa: ranh giới service **phải** là ranh giới context — tách service mà không tách context cho ra distributed monolith (1.3 đã cảnh báo; giờ bạn có tên cho ranh giới đúng).

### Context Mapping & Anti-Corruption Layer

Các context phải liên lạc — quan hệ giữa chúng cũng cần thiết kế. Đáng nhớ nhất cho dev thực chiến là **Anti-Corruption Layer (ACL)**: khi context của bạn phải nói chuyện với hệ ngoài/hệ legacy có mô hình xấu, đừng để mô hình đó **rò vào** — dựng một lớp dịch (chính là Adapter + Facade, 4.4, với sứ mệnh chiến lược): dịch model bẩn của họ sang ubiquitous language của bạn tại biên, và chỉ tại biên.

```go
// ACL: package tích hợp với hệ ERP legacy
// KHÔNG cho erpsoap.ProductDTO (40 field, tên tiếng-Đức-viết-tắt) chạm vào domain
func (a *ERPAdapter) ProductBySKU(ctx context.Context, sku catalog.SKU) (catalog.Product, error) {
    dto, err := a.soapClient.GetArtikelStamm(ctx, string(sku))  // thế giới của HỌ
    if err != nil { return catalog.Product{}, a.translateErr(err) }
    return a.toDomain(dto)                                       // thế giới của BẠN
}
```

Không có ACL, mô hình legacy lan như mực vào codebase mới — và 5 năm sau hệ "mới" mang nguyên di sản của hệ cũ. Đây là một trong những khoản đầu tư kiến trúc có ROI rõ nhất khi tích hợp.

## 3. Tactical Design — các viên gạch, hầu hết đã quen mặt

Điểm danh nhanh — vì tài liệu này đã xây từng viên từ Level 1:

- **Value Object**: định danh bằng *giá trị*, immutable — `Money`, `Email`, `Address` (1.5, 3.1 Primitive Obsession). Viên gạch rẻ nhất, đáng dùng sớm nhất, kể cả khi bạn không "làm DDD".
- **Entity**: định danh bằng *ID xuyên thời gian*, trạng thái đổi qua hành vi có kiểm soát — `Order` với `Confirm()/Cancel()` (1.2 encapsulation, 2.6 Tell Don't Ask, 4.7 State).
- **Aggregate** — viên gạch duy nhất thật sự mới, đáng mổ kỹ:

### Aggregate — ranh giới nhất quán, không phải cây object

Định nghĩa qua bài toán: `Order` chứa `[]OrderItem`; invariant "tổng tiền đơn = tổng tiền item" và "đơn confirmed không thêm item" phải **luôn đúng**. Nếu code bất kỳ đâu load `OrderItem` riêng lẻ và sửa — invariant không ai gác (đúng bài content coupling 1.3). Aggregate là câu trả lời:

> **Aggregate = cụm object được sửa đổi như MỘT đơn vị, qua MỘT cửa (aggregate root), lưu như MỘT transaction.**

Quy tắc thực hành: bên ngoài chỉ giữ tham chiếu đến **root** (`Order`), không bao giờ đến ruột (`OrderItem`); mọi sửa đổi đi qua method của root (`order.AddItem(...)` — root gác invariant); repository theo aggregate, không theo bảng (đã nói ở 4.12b — giờ bạn biết vì sao); tham chiếu **giữa các aggregate bằng ID**, không bằng con trỏ (`Order` giữ `CustomerID`, không ôm `*Customer` — nếu ôm, "một transaction" sẽ nuốt dần cả object graph).

Câu hỏi thiết kế khó nhất — **aggregate to cỡ nào?** — có nguyên tắc trả lời: *đủ to để chứa invariant phải-đúng-ngay-lập-tức, và không to hơn*. Invariant "tổng đơn = tổng item" cần đúng tức thì → item trong aggregate Order. Còn "khách không vượt hạn mức công nợ tổng"? Kéo mọi đơn của khách vào một aggregate khổng lồ → transaction đụng nhau, lock contention. Lời giải trưởng thành: chấp nhận invariant đó là **eventual** — kiểm tra khi tạo đơn, đối soát bằng event sau (4.13) — và đây chính là cánh cửa dẫn sang chương event-driven: **kích thước aggregate là quyết định về ranh giới nhất quán tức thì vs nhất quán cuối cùng.**

- **Domain Service**: nghiệp vụ không thuộc về một entity nào (tính điểm cần Order + Campaign — đã gặp ở 3.2 Feature Envy phần ngoại lệ). Dùng dè — mọi thứ trôi hết vào service là anemic model quay lại (2.1).
- **Repository, Domain Events**: nguyên như 4.12, 4.13 — DDD chính là quê gốc của chúng.

## 4. Trade-off — DDD có giá bao nhiêu, và ai không nên mua

- **Chi phí cố định cao**: ubiquitous language cần *tiếp cận được domain expert* thường xuyên — không có expert (hoặc PM chỉ chuyển ticket) thì nửa strategic chết từ đầu, chỉ còn vỏ tactical. Aggregate + value object + ACL là nhiều code hơn CRUD thẳng — **chỉ hoàn vốn khi nghiệp vụ đủ phức tạp và sống đủ lâu**.
- **Thước đo "đủ phức tạp"** — thử ba câu: nghiệp vụ có *luật* không (invariant, quy trình, ngoại lệ — hay chỉ là form nhập rồi lưu)? luật có *đổi thường xuyên* không? hiểu sai luật có *đắt* không (tiền, pháp lý)? Ba câu "có" → DDD hoàn vốn. Ba câu "không" → CRUD + transaction script là kiến trúc **đúng**, và Evans cũng nói vậy.
- **Không phải nhị phân toàn-hệ-thống**: hệ lớn có context lõi phức tạp (pricing, risk — đáng DDD đầy đủ) cạnh context phụ trợ đơn giản (quản lý banner — CRUD thẳng tay). Chọn *theo từng context* — chính bounded context cho phép mỗi vùng một mức đầu tư.

## 5. Anti-pattern

- **DDD-cargo-cult**: repository + entity + "aggregate" cho app CRUD 5 bảng — toàn bộ chi phí, không lợi ích nào, vì không có invariant nào để gác (đã cảnh báo suốt 4.12).
- **Anemic Domain Model đội lốt**: có đủ folder `domain/`, `entity/` nhưng entity toàn getter/setter, luật nằm hết ở service — DDD về hình thức, transaction script về bản chất (2.1). Không xấu *nếu tự nhận là transaction script*; xấu vì tự nhận nhầm.
- **Shared kernel lười**: các context "dùng chung cho tiện" package model — ranh giới ngôn ngữ sụp, coupling deploy quay lại; 5 năm sau không tách nổi.
- **Aggregate tham lam**: root ôm cả object graph "cho chắc invariant" → transaction to, lock nhiều, hiệu năng chết — quên mất quy tắc tham-chiếu-bằng-ID.

---

*Tiếp theo: [5.1 — Clean & Hexagonal Architecture](/series/software-design/level-5-architecture/01-clean-hexagonal/) — cho mô hình domain một chỗ ở trong codebase.*
