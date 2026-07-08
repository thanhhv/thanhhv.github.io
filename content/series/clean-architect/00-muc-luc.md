+++
title = "Clean Architecture với Golang — Từ First Principles đến Production"
date = "2026-07-07T18:00:00+07:00"
draft = false
tags = ["backend", "golang", "clean-architecture"]
series = ["Clean Architecture với Golang"]
+++

> Tài liệu chuyên sâu dành cho Backend Engineer → Software Architect.
> Ngôn ngữ minh họa: Go 1.22+, ưu tiên Standard Library.
> Triết lý: **hiểu bản chất và trade-off, không sao chép template.**

## Cách đọc

Tài liệu được viết theo chuỗi nhân quả — mỗi chương đứng trên chương trước:

```
Business Problem → Vì sao code khó bảo trì → Coupling → Dependency → SOLID
  → Clean Architecture ra đời → Dependency Rule → Layer → Production
  → Trade-off → Khi nào KHÔNG nên áp dụng
```

Người mới nên đọc tuần tự. Người có kinh nghiệm có thể nhảy thẳng vào Level 3–4, nhưng nếu thấy một kết luận thiếu căn cứ — căn cứ nằm ở Level 1.

Mỗi chương đều có: problem statement, giải thích bản chất, code Go hoàn chỉnh, sơ đồ, trade-off, best practices, anti-patterns, và mục "khi nào KHÔNG nên dùng".

## Mục lục

### Level 1 — Foundation

| Chương | Nội dung |
|---|---|
| [1.1 Coupling & Cohesion](/series/clean-architect/01-foundation/01-coupling-cohesion/) | Gốc rễ của mọi vấn đề kiến trúc; các mức coupling; đo bằng công cụ Go |
| [1.2 SOLID trong Go](/series/clean-architect/01-foundation/02-solid/) | Năm quy tắc quản trị phụ thuộc, dịch sang Go idiomatic |
| [1.3 Dependency Inversion](/series/clean-architect/01-foundation/03-dependency-inversion/) | Cỗ máy đảo chiều phụ thuộc — chương then chốt của toàn tài liệu |
| [1.4 Separation of Concerns & Composition](/series/clean-architect/01-foundation/04-separation-of-concerns/) | Chọn trục tách; composition thay inheritance trong Go |

### Level 2 — Clean Architecture Core & Engineering

| Chương | Nội dung |
|---|---|
| [2.1 Vì sao Clean Architecture ra đời](/series/clean-architect/02-clean-architecture-core/01-vi-sao-clean-architecture-ra-doi/) | Lịch sử hội tụ Hexagonal/Onion/DDD; nó hứa gì và không hứa gì |
| [2.2 Dependency Rule](/series/clean-architect/02-clean-architecture-core/02-dependency-rule/) | Luật duy nhất; cưỡng chế bằng internal/, linter, test kiến trúc |
| [2.3 Bốn vòng trong Go](/series/clean-architect/02-clean-architecture-core/03-cac-layer/) | Service loyalty hoàn chỉnh: Entity → Use Case → Adapter → Framework |
| [3.1 Tổ chức Package](/series/clean-architect/03-go-project-structure/01-package-organization/) | cmd/, internal/, pkg/; flat vs layer vs feature vs vertical slice |
| [4. Dependency Injection](/series/clean-architect/04-dependency-injection/01-di-trong-go/) | Manual DI, Wire, Fx; interface placement — bảng tra cứu |

### Level 3 — Senior

| Chương | Nội dung |
|---|---|
| [5. Data Access](/series/clean-architect/05-data-access/01-repository-transaction/) | Repository thật vs wrapper ORM; transaction & Unit of Work; sqlc/pgx/GORM |
| [6. Delivery Layer](/series/clean-architect/06-delivery/01-delivery-layer/) | HTTP, gRPC, GraphQL, CLI, Worker — N cửa vào, 1 bộ nghiệp vụ |
| [7. Integration](/series/clean-architect/07-integration/01-integration/) | External API, retry/circuit breaker, Kafka vs RabbitMQ, Redis, outbox |
| [8. Testing Strategy](/series/clean-architect/08-testing/01-testing-strategy/) | Test theo vòng; fake vs mock; testcontainers; contract test |
| [9. Clean Architecture & DDD](/series/clean-architect/09-ddd/01-clean-architecture-va-ddd/) | Value Object, Aggregate, Domain Service; thang liều lượng DDD |
| [10. Clean Architecture & CQRS](/series/clean-architect/10-cqrs/01-clean-architecture-va-cqrs/) | Command/Query tách đường; các mức CQRS và khi nào leo thang |
| [11. Production Concerns](/series/clean-architect/11-production/01-production-concerns/) | Config, logging, observability, graceful shutdown, retry, idempotency |

### Ví dụ tổng hợp — E-commerce ba giai đoạn

| Chương | Nội dung |
|---|---|
| [12.1 Giai đoạn 1: Monolith](/series/clean-architect/12-vi-du-ecommerce/01-giai-doan-1-monolith/) | 3 engineer, MVP — Clean Architecture có chọn lọc theo mật độ rule |
| [12.2 Giai đoạn 2: Modular Monolith](/series/clean-architect/12-vi-du-ecommerce/02-giai-doan-2-modular-monolith/) | 12 engineer — module contract, event + outbox, CQRS, webhook thanh toán |
| [12.3 Giai đoạn 3: Tách service](/series/clean-architect/12-vi-du-ecommerce/03-giai-doan-3-tach-service/) | 40 engineer — lý do đúng/sai để tách; strangler fig; saga |

### Level 4 — Principal

| Chương | Nội dung |
|---|---|
| [13. So sánh các kiến trúc](/series/clean-architect/13-so-sanh/01-so-sanh-cac-kien-truc/) | Clean vs Layered vs Hexagonal vs Onion vs MVC vs Modular Monolith vs DDD |
| [14. Evolutionary Architecture & Migration](/series/clean-architect/14-principal/01-evolutionary-architecture-va-migration/) | Fitness function, ADR, refactor legacy quy mô lớn, Conway, hiệu năng, chiến lược migration |

## Tóm tắt 10 nguyên tắc của toàn bộ tài liệu

1. Coupling = lượng kiến thức module A biết về B; giảm kiến thức, không phải cắt liên lạc.
2. Ranh giới đặt theo trục thay đổi; phụ thuộc trỏ về phía ổn định — mọi thứ khác là hệ quả.
3. Dependency Rule: mọi mũi tên compile-time trỏ vào domain; control flow không cần đổi.
4. Interface thuộc về consumer, nhỏ nhất có thể, nói ngôn ngữ domain, đẻ theo nhu cầu thật.
5. Entity giữ bất biến, use case giữ quy trình, adapter phiên dịch, main lắp ráp.
6. Lỗi: domain định nghĩa, adapter dịch hai chiều, wrap ở giữa, log một lần ở biên.
7. Cross-cutting (log, metric, retry, cache, tx) = decorator/middleware — không rải trong nghiệp vụ.
8. Kiến trúc chỉ tồn tại khi được cưỡng chế bằng máy: internal/, depguard, test kiến trúc, CI.
9. Nghi thức tỉ lệ với mật độ rule và tuổi thọ hệ thống — CRUD nhỏ thì viết thẳng, và đó là thiết kế đúng.
10. Kiến trúc tốt tối đa hóa số quyết định có thể trì hoãn và đảo ngược — monolith → modular → services là con đường, không phải cuộc thi.

---

*Tài liệu biên soạn tháng 7/2026. Code minh họa dùng Go 1.22+.*
