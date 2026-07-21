+++
title = "Software Design & Design Patterns — Từ First Principles đến Production"
date = "2026-07-17T07:00:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

> Tài liệu chuyên sâu dành cho Backend Engineer, Golang/Node.js Developer, Tech Lead và Solution Architect.
> Viết bởi góc nhìn của một Principal Software Architect: **pattern không phải là mục tiêu — pattern là kết quả của việc giải quyết một vấn đề thiết kế cụ thể.**

---

## Triết lý của bộ tài liệu này

Hầu hết tài liệu về Design Pattern bắt đầu bằng: *"Singleton là... Factory là... Observer là..."* — cách học này tạo ra những kỹ sư thuộc lòng 23 pattern nhưng không biết **khi nào** dùng, và tệ hơn, dùng pattern ở nơi không cần thiết (over-engineering).

Tài liệu này đi theo lộ trình ngược lại:

```
Business Problem
      ↓
Code bắt đầu đơn giản
      ↓
Yêu cầu thay đổi
      ↓
Code Smell xuất hiện
      ↓
Maintenance Cost tăng
      ↓
Refactoring từng bước
      ↓
Design Principle
      ↓
Design Pattern (xuất hiện một cách TỰ NHIÊN)
      ↓
Production
      ↓
Trade-off
```

Mỗi chương đều trả lời 5 câu hỏi:

1. Kỹ thuật này giải quyết vấn đề gì?
2. Nếu không dùng thì chuyện gì xảy ra?
3. Có cách đơn giản hơn không?
4. Nó có còn phù hợp với Golang hiện đại không?
5. Khi nào nó trở thành anti-pattern?

Ví dụ minh họa dùng **Golang** làm chính, so sánh với **TypeScript (Node.js)** khi sự khác biệt ngôn ngữ làm thay đổi cách áp dụng.

---

## Mục lục

### ✅ Level 1 — Software Design Fundamentals

| Chương | Chủ đề | Trạng thái |
|---|---|---|
| [1.1](/series/software-design/level-1-fundamentals/01-vi-sao-code-kho-bao-tri/) | Vì sao code trở nên khó bảo trì — bản chất của Software Design | ✅ |
| [1.2](/series/software-design/level-1-fundamentals/02-abstraction-encapsulation/) | Abstraction & Encapsulation — che giấu đúng thứ cần che | ✅ |
| [1.3](/series/software-design/level-1-fundamentals/03-coupling-cohesion-dependency/) | Coupling, Cohesion & Dependency — thước đo của mọi quyết định thiết kế | ✅ |
| [1.4](/series/software-design/level-1-fundamentals/04-composition-inheritance-polymorphism/) | Composition vs Inheritance & Polymorphism | ✅ |
| [1.5](/series/software-design/level-1-fundamentals/05-interface-immutability/) | Interface & Immutability — hai vũ khí của Go | ✅ |

### ✅ Level 2 — Design Principles

| Chương | Chủ đề | Trạng thái |
|---|---|---|
| [2.0](/series/software-design/level-2-principles/00-solid-tong-quan/) | SOLID — lịch sử ra đời và vấn đề nó giải quyết | ✅ |
| [2.1](/series/software-design/level-2-principles/01-srp/) | Single Responsibility Principle — trục của sự thay đổi | ✅ |
| [2.2](/series/software-design/level-2-principles/02-ocp/) | Open/Closed Principle — mở rộng mà không sửa | ✅ |
| [2.3](/series/software-design/level-2-principles/03-lsp/) | Liskov Substitution Principle — hợp đồng hành vi | ✅ |
| [2.4](/series/software-design/level-2-principles/04-isp/) | Interface Segregation Principle — interface nhỏ, phía consumer | ✅ |
| [2.5](/series/software-design/level-2-principles/05-dip/) | Dependency Inversion Principle — đảo chiều phụ thuộc | ✅ |
| [2.6](/series/software-design/level-2-principles/06-dry-kiss-yagni-lod/) | DRY, KISS, YAGNI, Law of Demeter, Tell Don't Ask | ✅ |

### ✅ Level 3 — Refactoring

| Chương | Chủ đề | Trạng thái |
|---|---|---|
| [3.0](/series/software-design/level-3-refactoring/00-technical-debt-legacy-code/) | Technical Debt & Legacy Code — kinh tế học của việc refactor | ✅ |
| [3.1](/series/software-design/level-3-refactoring/01-smells-phinh-to/) | Code Smells "phình to": Long Method, Large Class, God Object, Primitive Obsession, Data Clumps | ✅ |
| [3.2](/series/software-design/level-3-refactoring/02-smells-coupling/) | Code Smells coupling: Duplicate Code, Shotgun Surgery, Feature Envy, Message Chains, Middle Man | ✅ |
| [3.3](/series/software-design/level-3-refactoring/03-ky-thuat-co-ban/) | Kỹ thuật cơ bản: Extract Method, Move Method, Extract Class, Introduce Parameter Object | ✅ |
| [3.4](/series/software-design/level-3-refactoring/04-ky-thuat-cau-truc/) | Kỹ thuật cấu trúc: Replace Conditional with Polymorphism, Replace Inheritance with Composition, Replace Loop with Strategy | ✅ |

### ✅ Level 4 — Design Patterns

| Chương | Chủ đề | Trạng thái |
|---|---|---|
| [4.0](/series/software-design/level-4-patterns/00-tong-quan-patterns/) | Design Patterns — cách đọc GoF bằng con mắt 2026 | ✅ |
| [4.1](/series/software-design/level-4-patterns/01-factory/) | Factory Method & Abstract Factory | ✅ |
| [4.2](/series/software-design/level-4-patterns/02-builder-functional-options/) | Builder & Functional Options (kèm so sánh Factory vs Builder) | ✅ |
| [4.3](/series/software-design/level-4-patterns/03-singleton-prototype/) | Singleton & Prototype | ✅ |
| [4.4](/series/software-design/level-4-patterns/04-adapter-facade/) | Adapter & Facade (kèm so sánh) | ✅ |
| [4.5](/series/software-design/level-4-patterns/05-decorator-proxy/) | Decorator & Proxy — và Middleware (kèm bảng so sánh 4 pattern "bọc") | ✅ |
| [4.6](/series/software-design/level-4-patterns/06-composite-bridge-flyweight/) | Composite, Bridge & Flyweight | ✅ |
| [4.7](/series/software-design/level-4-patterns/07-strategy-state/) | Strategy & State (kèm so sánh Strategy vs State, Interface vs Generics) | ✅ |
| [4.8](/series/software-design/level-4-patterns/08-observer-pubsub/) | Observer & pub/sub | ✅ |
| [4.9](/series/software-design/level-4-patterns/09-command-chain-mediator/) | Command, Chain of Responsibility & Mediator | ✅ |
| [4.10](/series/software-design/level-4-patterns/10-template-visitor-iterator-memento/) | Template Method, Visitor, Iterator & Memento | ✅ |
| [4.11](/series/software-design/level-4-patterns/11-dependency-injection/) | Dependency Injection & DI Container (kèm so sánh DI vs Service Locator) | ✅ |
| [4.12](/series/software-design/level-4-patterns/12-repository-uow-specification/) | Repository, Unit of Work & Specification | ✅ |
| [4.13](/series/software-design/level-4-patterns/13-pipeline-plugin-events-saga-outbox/) | Pipeline, Plugin, Domain Events, Saga & Outbox — bản đồ GoF/Enterprise/Go idiom | ✅ |

### ✅ Level 5 — Production Architecture

| Chương | Chủ đề | Trạng thái |
|---|---|---|
| [5.0](/series/software-design/level-5-architecture/00-ddd/) | Domain-Driven Design — strategic (bounded context, ubiquitous language, ACL) & tactical (aggregate, value object) | ✅ |
| [5.1](/series/software-design/level-5-architecture/01-clean-hexagonal/) | Clean & Hexagonal Architecture — một ý tưởng, nhiều tên gọi; bản đồ package Go thực dụng | ✅ |
| [5.2](/series/software-design/level-5-architecture/02-event-driven-cqrs/) | Event-driven Architecture & CQRS (+ Event Sourcing) — bảng quyết định kiến trúc tổng | ✅ |
| [5.3](/series/software-design/level-5-architecture/03-case-studies/) | Production Case Studies: io/net/http/context, Gin/Echo, Uber Fx, GORM/Ent/sqlc, Redis/Kafka client, Kubernetes, Docker, Terraform | ✅ |
| [5.4](/series/software-design/level-5-architecture/04-tong-ket/) | Tổng kết toàn bộ tài liệu — sáu câu hỏi cho mọi design review, lộ trình đọc tiếp | ✅ |

---

## Cách đọc tài liệu

**Nếu bạn là Senior Engineer** muốn hệ thống hóa lại kiến thức: đọc tuần tự từ Level 1. Những chương đầu có thể quen thuộc, nhưng góc nhìn "chi phí thay đổi" và các trade-off sẽ khác với cách bạn từng học.

**Nếu bạn là Tech Lead** đang phải review code và giải thích *vì sao* một thiết kế tệ: Level 1.3 (Coupling/Cohesion) và Level 2 (SOLID) cho bạn ngôn ngữ chung để nói chuyện với team.

**Nếu bạn đang refactor legacy code**: đọc Level 1.3 → Level 3 → quay lại Level 4 khi pattern tự xuất hiện trong quá trình refactor.

**Điều quan trọng nhất**: đừng đọc Level 4 (Patterns) trước Level 1–3. Đó chính là cách học đã tạo ra hàng nghìn codebase over-engineered.

---

## Quy ước trong tài liệu

- Thuật ngữ chuyên ngành giữ nguyên tiếng Anh: *coupling, cohesion, interface, refactoring, trade-off...*
- Code Go là code chính; TypeScript xuất hiện khi khác biệt ngôn ngữ đáng để so sánh.
- Mỗi ví dụ code có nhãn phiên bản (V1 → V2 → V3...) thể hiện quá trình tiến hóa, không phải kết quả cuối cùng.
- `// ❌` đánh dấu code có vấn đề, `// ✅` đánh dấu code sau khi cải thiện — nhưng hãy nhớ: **✅ luôn có trade-off riêng của nó**, được phân tích ở cuối mỗi chương.
