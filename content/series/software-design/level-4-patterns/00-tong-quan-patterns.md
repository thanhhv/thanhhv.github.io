+++
title = "4.0 — Design Patterns: cách đọc GoF bằng con mắt 2026"
date = "2026-07-17T10:00:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## 1. Pattern là gì — và cuốn sách GoF thực ra nói gì

Năm 1994, bốn tác giả (Gang of Four) xuất bản *Design Patterns: Elements of Reusable Object-Oriented Software* — danh mục 23 cấu trúc lặp đi lặp lại mà họ quan sát thấy trong các hệ thống C++/Smalltalk tốt. Ý tưởng mượn từ kiến trúc sư Christopher Alexander: pattern = **một giải pháp đã được kiểm chứng cho một bài toán lặp lại trong một ngữ cảnh cụ thể** — kèm tên gọi chung để kỹ sư nói chuyện với nhau bằng một từ thay vì mười phút giải thích.

Ba giá trị thật của pattern, xếp theo thứ tự quan trọng:

1. **Từ vựng chung**: "chỗ này bọc Decorator được" truyền đạt trong 2 giây điều mà không có tên gọi phải vẽ bảng mới xong. Đây là giá trị bền nhất — kể cả khi bạn không bao giờ "cài đặt pattern", bạn vẫn cần đọc được từ vựng này trong code người khác, tài liệu thư viện, và design review.
2. **Kho nghiệm đã trả học phí**: mỗi pattern gói kèm phần "Consequences" — các trade-off mà người đi trước đã va phải. Đọc pattern đúng cách là đọc phần này kỹ nhất.
3. **Giải pháp lắp được**: giá trị thấp nhất và nguy hiểm nhất — vì cấu trúc lớp trong sách viết cho C++ 1994, bê nguyên vào Go/TS 2026 thường tạo ra code cồng kềnh hơn cần thiết.

Và điều chính GoF viết ngay trang đầu mà hầu hết người học bỏ qua: pattern là **descriptive** (mô tả cái đã mọc lên tự nhiên trong code tốt), không phải **prescriptive** (đơn thuốc áp trước). Toàn bộ Level 1–3 của bộ tài liệu này tồn tại để bạn đọc Level 4 đúng tư thế: pattern là *điểm đến của refactoring* — bảng cuối chương 3.4 chính là bản đồ các con đường dẫn vào đây.

## 2. Bản đồ 23 pattern — và tình trạng của chúng trong Go/TS hiện đại

GoF chia ba nhóm theo *mục đích*:

- **Creational** — kiểm soát việc **tạo** object: Factory Method, Abstract Factory, Builder, Prototype, Singleton.
- **Structural** — **ghép** các object thành cấu trúc lớn hơn: Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy.
- **Behavioral** — phân chia **trách nhiệm và luồng liên lạc** giữa các object: Strategy, Observer, Command, State, Chain of Responsibility, Mediator, Template Method, Visitor, Iterator, Memento (+ Interpreter, ít dùng).

Nhưng bản đồ hữu ích hơn cho người đọc 2026 là bản đồ **số phận**: 30 năm tiến hóa ngôn ngữ đã làm các pattern phân hóa thành bốn nhóm số phận rất khác nhau:

| Số phận | Pattern | Vì sao |
|---|---|---|
| **Tan vào ngôn ngữ** — dùng hằng ngày mà không cần gọi tên | Strategy (→ function value), Iterator (→ `for range`, generator), Command (→ closure), Template Method (→ truyền hàm) | First-class function + closure làm cấu trúc "một interface một method + một class implement" sụp xuống còn một hàm |
| **Vẫn nguyên giá trị, đổi hình dạng** | Decorator (middleware), Adapter, Facade, Factory, Builder (→ Functional Options trong Go), Proxy, Observer (→ pub/sub, watch API), State, Chain of Responsibility | Bài toán gốc còn nguyên; cách thể hiện idiomatic theo từng ngôn ngữ |
| **Thu hẹp đất sống** | Singleton (→ DI + composition root), Prototype (→ struct copy, ít khi cần), Visitor (→ type switch/pattern matching), Memento, Mediator | Bài toán gốc hoặc hiếm gặp hơn, hoặc có lời giải rẻ hơn trong ngôn ngữ hiện đại |
| **Mở rộng ra ngoài GoF** — pattern mới của kỷ nguyên backend | Repository, Unit of Work, Middleware, Pipeline, Functional Options, Saga, Outbox, CQRS... | GoF viết cho phần mềm desktop đơn tiến trình; backend phân tán sinh bài toán mới, pattern mới (phần Modern Design) |

Cách trình bày của các chương sau bám theo bản đồ số phận này: pattern "tan vào ngôn ngữ" được trình nhanh ở dạng hiện đại; pattern "nguyên giá trị" được đào sâu đủ 9 mục template; pattern "thu hẹp" được nói thẳng khi nào còn đáng dùng.

## 3. Khung đọc mỗi pattern — 5 câu hỏi bắt buộc

Nhắc lại từ trang đầu bộ tài liệu, vì đây là nơi chúng được thi hành:

1. **Pattern này giải quyết vấn đề gì?** — phải chỉ được smell/chi phí cụ thể (Level 3), không chấp nhận "để code linh hoạt hơn".
2. **Không dùng thì sao?** — mô tả được quỹ đạo xuống cấp của code nếu cứ để nguyên.
3. **Có cách đơn giản hơn không?** — hàm thường, struct thường, copy 10 dòng — luôn xét trước.
4. **Còn phù hợp với Go/TS hiện đại không?** — hình dạng idiomatic là gì, có khi nào ngôn ngữ đã làm hộ?
5. **Khi nào thành anti-pattern?** — ngưỡng mà chi phí gián tiếp vượt lợi ích.

## 4. Quy ước trình bày

- Mỗi chương gộp các pattern **họ hàng gần** để so sánh trực diện (Factory Method + Abstract Factory; Builder + Functional Options; Adapter + Facade; Decorator + Proxy...) — vì trong thực tế câu hỏi khó không phải "pattern X là gì" mà là "X hay Y cho chỗ này?".
- Code Go là chính; TS xuất hiện khi khác biệt đáng kể. UML chỉ vẽ khi cấu trúc đủ phức tạp để lời không tải nổi — và luôn vẽ *sau* khi code đã tự dẫn đến cấu trúc đó.
- Mỗi pattern kết bằng **Production sighting**: chỉ đúng chỗ nó sống trong stdlib/thư viện lớn, kèm phân tích vì sao tác giả chọn nó.

---

## Mục lục Level 4

**Creational**
- [4.1 — Factory Method & Abstract Factory](/series/software-design/level-4-patterns/01-factory/)
- [4.2 — Builder & Functional Options](/series/software-design/level-4-patterns/02-builder-functional-options/)
- [4.3 — Singleton & Prototype](/series/software-design/level-4-patterns/03-singleton-prototype/)

**Structural**
- [4.4 — Adapter & Facade](/series/software-design/level-4-patterns/04-adapter-facade/)
- [4.5 — Decorator & Proxy — và Middleware](/series/software-design/level-4-patterns/05-decorator-proxy/)
- [4.6 — Composite, Bridge & Flyweight](/series/software-design/level-4-patterns/06-composite-bridge-flyweight/)

**Behavioral** *(phần tiếp theo)*
- Strategy & State; Observer & pub/sub; Command, Chain of Responsibility, Mediator; Template Method, Visitor, Iterator, Memento

**Modern Design** *(phần tiếp theo)*
- Dependency Injection & DI container; Repository, Unit of Work, Specification; Middleware & Pipeline; Plugin; Domain Events, Saga, Outbox

---

*Bắt đầu: [4.1 — Factory Method & Abstract Factory](/series/software-design/level-4-patterns/01-factory/)*
