+++
title = "Domain-Driven Design — Bộ tài liệu chuyên sâu"
date = "2026-07-09T07:00:00+07:00"
draft = false
tags = ["backend", "ddd", "architecture"]
series = ["Domain-Driven Design"]
+++

> Tài liệu dành cho Backend Engineer, Senior Backend Engineer, Tech Lead, Solution Architect và Software Architect. Viết bằng tiếng Việt, giữ nguyên thuật ngữ chuyên ngành (Domain, Bounded Context, Aggregate, Entity, Value Object, Domain Event, Repository, Ubiquitous Language...). Code mẫu dùng TypeScript (NestJS) và Go, mỗi khái niệm đều được đối chiếu với cách làm phổ biến trong framework để chỉ ra những điểm dễ hiểu sai.

## Tài liệu này viết theo triết lý nào

Không chương nào mở đầu bằng định nghĩa. Mọi khái niệm đi theo mạch: **vấn đề business có thật → vì sao cách làm truyền thống thất bại → bản chất của lời giải → cách hoạt động → điểm mạnh, điểm yếu, trade-off → cân nhắc production → anti-pattern → khi nào KHÔNG nên dùng**. Mỗi kết luận kỹ thuật đều phải trả lời: tại sao? đánh đổi gì? không áp dụng thì sao? áp dụng sai thì hậu quả gì? Và xuyên suốt: DDD không phải một bộ class — nó là phương pháp tư duy để quản lý sự phức tạp của business; tài liệu không thần thánh hóa nó, mà chỉ rõ chỗ nào nó đáng tiền và chỗ nào nó là gánh nặng.

## Mục lục

### Phần I — Foundations (Level 1)

| Chương | Nội dung chính |
|---|---|
| [01 — Tại sao DDD ra đời](/series/domain-driven-design/01-tai-sao-ddd-ra-doi/) | Business complexity là kẻ thù chính; vì sao thiết kế database-first, transaction script và CRUD thinking sụp đổ khi rule chồng chéo; accidental vs essential complexity; knowledge gap giữa dev và domain expert |
| [02 — Domain và Subdomain](/series/domain-driven-design/02-domain-va-subdomain/) | Core / Supporting / Generic Domain; nhận diện lợi thế cạnh tranh; quyết định build–buy–outsource; "liều lượng DDD" theo từng vùng; hai case study phân rã domain (logistics, fintech) |
| [03 — Ubiquitous Language](/series/domain-driven-design/03-ubiquitous-language/) | Chi phí dịch thuật vô hình; ngôn ngữ chung đi vào code thế nào; Event Storming; một từ nhiều nghĩa dẫn tới Bounded Context; quy ước song ngữ Việt–Anh cho team Việt |

### Phần II — Strategic Design (Level 3)

| Chương | Nội dung chính |
|---|---|
| [04 — Bounded Context](/series/domain-driven-design/04-bounded-context/) | Vì sao Enterprise Model thống nhất luôn thất bại; ranh giới ngữ nghĩa; Bounded Context vs Subdomain (problem space vs solution space); Bounded Context ≠ microservice; Shared Database phá ranh giới thế nào |
| [05 — Context Mapping](/series/domain-driven-design/05-context-mapping/) | Context Map là công cụ quyền lực/tổ chức; đủ 9 pattern: Partnership, Shared Kernel, Customer/Supplier, Conformist, ACL, Open Host Service, Published Language, Separate Ways, Big Ball of Mud; code ACL đầy đủ TS + Go |

### Phần III — Tactical Design (Level 2)

| Chương | Nội dung chính |
|---|---|
| [06 — Entity và Value Object](/series/domain-driven-design/06-entity-va-value-object/) | Identity vs equality by value; immutability; primitive obsession; **Entity DDD ≠ entity ORM (TypeORM/GORM)**; refactor từ anemic sang rich model; chiến lược persistence mapping |
| [07 — Aggregate](/series/domain-driven-design/07-aggregate/) | Chương trọng tâm: consistency boundary = transaction boundary; xác định ranh giới từ invariant thật (không từ ERD); optimistic concurrency; giải phẫu God Aggregate từng bước; **aggregate ≠ nhóm bảng, ≠ relations của ORM** |
| [08 — Repository và Factory](/series/domain-driven-design/08-repository-va-factory/) | Repository là collection ảo của aggregate; interface ở domain, implementation ở infrastructure; **Repository DDD ≠ repository của ORM**; vì sao generic repository nguy hiểm; Factory và reconstitution; in-memory fake cho test |
| [09 — Domain Service và Application Service](/series/domain-driven-design/09-domain-service-va-application-service/) | Logic không thuộc entity nào thì ở đâu; orchestration vs business rule; giải phẫu một OrderService 500 dòng kiểu NestJS/Go và tách từng bước |
| [10 — Domain Event](/series/domain-driven-design/10-domain-event/) | Sự kiện là quá khứ bất biến; **Domain Event vs Integration Event**; collect trong aggregate — publish sau persist; thiết kế payload; event versioning; eventual consistency có kỷ luật |
| [11 — Specification](/series/domain-driven-design/11-specification/) | Business rule dạng điều kiện có nhà; kết hợp and/or/not; ba công dụng và bài toán "dịch sang SQL"; khi nào specification là overkill |

### Phần IV — DDD và Kiến trúc (Level 3–4)

| Chương | Nội dung chính |
|---|---|
| [12 — DDD và Kiến trúc](/series/domain-driven-design/12-ddd-va-kien-truc/) | Layered vs Hexagonal vs Onion vs Clean — bản chất chung là dependency inversion; cấu trúc thư mục cụ thể cho NestJS và Go; Modular Monolith là điểm khởi đầu đúng; Bounded Context là đơn vị chia microservice; distributed monolith |
| [13 — DDD và Distributed Systems](/series/domain-driven-design/13-ddd-va-distributed-systems/) | Event-driven Architecture; CQRS (hai mức, không bắt buộc kèm Event Sourcing); Saga — choreography vs orchestration, compensation; Outbox và bài toán dual-write; Event Sourcing — chi phí thật; Idempotency; vì sao 2PC chết |

### Phần V — Production (Level 4)

| Chương | Nội dung chính |
|---|---|
| [14 — DDD trong Production](/series/domain-driven-design/14-ddd-trong-production/) | Refactor legacy: strangler fig, bubble context, ACL bọc legacy; incremental adoption; Team Organization và Conway's Law, team topologies, inverse Conway; Monolith First; lộ trình tách microservices; testing, versioning, migration dài hạn |

### Phần VI — Case Study 8 ngành

| Chương | Nội dung chính |
|---|---|
| [15a — E-commerce, FinTech, Banking, Logistics](/series/domain-driven-design/15a-case-study-ecommerce-fintech-banking-logistics/) | Mỗi ngành: phân rã domain và lý do, context map, aggregate chủ chốt với invariant cụ thể (Order, Ledger double-entry, Shipment...), luồng event chính, trade-off và các lỗi thiết kế đặc thù |
| [15b — SaaS, Blockchain, Booking, Social Network](/series/domain-driven-design/15b-case-study-saas-blockchain-booking-social/) | Tenant isolation ba tầng chốt; ledger off-chain và ranh giới với on-chain; chống double-booking bằng reservation pattern; fan-out và ModerationCase; bảng so sánh bốn thái cực nhất quán |

### Phần VII — Tổng kết

| Chương | Nội dung chính |
|---|---|
| [16 — Anti-patterns & Khi nào KHÔNG nên dùng DDD](/series/domain-driven-design/16-anti-patterns-va-khi-nao-khong-dung-ddd/) | 11 anti-pattern với cách nhận biết trong code thật và cách sửa; bảng "khi nào không dùng DDD" với phương pháp thay thế; ma trận quyết định liều lượng; checklist 10 câu; tổng kết toàn bộ và lộ trình đọc tiếp |

## Ba lộ trình đọc gợi ý

**Đọc tuần tự (khuyên dùng cho lần đầu)** — 01 → 16: mạch lập luận được thiết kế nối tiếp, khái niệm sau đứng trên khái niệm trước.

**Lộ trình cho người cần tactical ngay** (đang phải thiết kế module tuần này): 01 (đọc nhanh) → 06 → 07 → 08 → 09 → 10 → 16. Cảnh báo: tactical không có strategic dễ thành cargo cult — quay lại phần II sớm nhất có thể.

**Lộ trình cho Tech Lead / Architect** (quyết định ranh giới hệ thống và tổ chức team): 01 → 02 → 03 → 04 → 05 → 12 → 13 → 14 → 16, case study (15a/15b) chọn ngành gần với hệ thống của bạn.

## Quy ước chung của toàn bộ tài liệu

Thuật ngữ DDD giữ tiếng Anh, phần giải thích bằng tiếng Việt. Diagram dùng Mermaid (đọc tốt trên GitHub/GitLab/Obsidian). Code TypeScript mặc định trong bối cảnh NestJS, code Go theo bố cục `internal/<context>/domain|app|infra`. Các ví dụ ngành (tiền tệ VND, COD, đối soát...) lấy từ bối cảnh sản phẩm Việt Nam có chủ đích — để bạn mang thẳng vào cuộc họp với domain expert của mình.
