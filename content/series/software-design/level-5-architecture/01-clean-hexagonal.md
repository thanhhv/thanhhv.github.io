+++
title = "5.1 — Clean & Hexagonal Architecture: một ý tưởng, nhiều tên gọi"
date = "2026-07-17T12:30:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## 1. Gỡ rối tên gọi trước — vì chúng là MỘT ý tưởng

Hexagonal/Ports & Adapters (Cockburn 2005), Onion (Palermo 2008), Clean Architecture (Martin 2012) — ba "trường phái" mà thực chất là **ba bản vẽ của cùng một nguyên tắc**, nguyên tắc mà bạn đã học trọn ở chương 2.5:

> **Nghiệp vụ ở trung tâm, không biết gì về thế giới ngoài. Mọi mũi tên phụ thuộc trỏ VÀO trong. Thế giới ngoài (DB, HTTP, queue, UI) nói chuyện với trung tâm qua các cổng (interface) do TRUNG TÂM định nghĩa.**

Tức là: **DIP áp dụng có hệ thống ở quy mô toàn codebase** — không hơn, không kém. Khác biệt giữa các trường phái chỉ là cách vẽ và mức chia tầng:

```
Hexagonal:  domain ⟵ ports (interface) ⟵ adapters (HTTP, DB, CLI, test...)
            — nhấn: MỌI thứ ngoài là adapter ngang hàng nhau, kể cả UI và DB

Clean:      entities ⟵ use cases ⟵ interface adapters ⟵ frameworks/drivers
            — nhấn: thêm tầng use case tường minh giữa domain và thế giới

Cả hai:     "The Dependency Rule" — source code dependency chỉ trỏ vào trong
```

Điểm sâu nhất của Hexagonal, hay bị bỏ qua: **DB và UI ngang hàng nhau** — đều là "chi tiết bên ngoài". Điều này đảo ngược trực giác "ứng dụng = code bọc quanh database" đã thống trị nhiều thập kỷ. Test cũng chỉ là một adapter nữa (driving adapter gọi vào port) — **khả năng test không phải thứ thêm vào, nó là hệ quả hình học của kiến trúc**: mọi thứ đã được xây suốt Level 2 (fake của 2.3–2.5) giờ có tên tầng.

## 2. Hình dạng Go cụ thể — từ nguyên tắc đến cây thư mục

Lý thuyết đã đủ từ 2.5; giá trị chương này là **bản đồ package thực dụng**. Một service Go theo tinh thần hexagonal, không cuồng tín:

```
/cmd/api/main.go          ← composition root: wiring (2.5, 4.11), config, lifecycle
/internal/
    /order/               ← DOMAIN + USE CASE của context "order" (gộp — xem ghi chú)
        order.go          ← entity, value object, invariant (1.2, 5.0)
        service.go        ← use case: ConfirmOrder, CompleteOrder (facade 4.4)
        ports.go          ← interface MÀ ORDER CẦN: Repository, Notifier (2.4, 2.5)
        events.go         ← OrderConfirmed... (4.13)
        service_test.go   ← test nghiệp vụ với fake — nhanh, không I/O
    /pricing/             ← context khác, cùng cấu trúc
    /platform/            ← ADAPTERS dùng chung
        /postgres/        ← implement order.Repository, pricing.ProductSource
        /kafka/           ← implement event publisher (outbox relay 4.13)
        /httpapi/         ← driving adapter: handler, middleware (4.5), DTO
```

Bốn quyết định đáng chú ý trong bản đồ này — mỗi cái là một trade-off có chủ đích:

**(a) Gộp domain + use case một package, chia theo bounded context** — thay vì tầng ngang toàn cục (`/entities`, `/usecases`, `/repositories` — logical cohesion, đúng cái 1.3 đã chê). Chia dọc theo context: mọi thứ về order ở một chỗ (functional cohesion), context là đơn vị tách service sau này (5.0). Go không cần bốn tầng vật lý của Clean để giữ Dependency Rule — **hai vùng (trong/ngoài) + import cycle cấm của compiler** là đủ.

**(b) `ports.go` — interface phía consumer** (2.4): order tự khai nó cần gì; postgres/kafka implement mà không được order biết mặt. Kiểm tra kiến trúc = đọc import: `internal/order` chỉ import stdlib + các context anh em (qua ID/event, không qua model — 5.0). Khóa bằng linter (`depguard`) nếu team đông.

**(c) DTO ba lớp có chọn lọc**: HTTP request/response struct (của httpapi), domain object (của order), row struct (của postgres) — ba hình dạng, dịch ở biên (2.5, 4.4 ACL). Chi phí dịch là thật; với type *không có invariant và trùng hình dạng* (một struct config đọc-only), cho phép dùng chung — cuồng tín ba-lớp-cho-mọi-thứ là bệnh đã cảnh báo ở 4.12a.

**(d) Use case là hàm/struct thường, không "framework use case"**: `func (s *Service) ConfirmOrder(ctx, cmd) error` — không cần `UseCaseInterface`, không cần command bus (4.9 đã dặn). Clean Architecture là *quy tắc phụ thuộc*, không phải bộ nghi lễ class.

## 3. Trade-off — kiến trúc này KHÔNG miễn phí, và không phải cho mọi service

Toàn bộ hồ sơ chi phí đã lập ở 2.5 mục 5 (adapter + model dịch + interface cho mỗi ranh giới; trừu tượng hóa DB có giới hạn; service CRUD mỏng thì thành nghi lễ) — nguyên giá trị ở quy mô kiến trúc, nhân theo số ranh giới. Bổ sung ba điều ở tầm hệ thống:

- **Thước đo quyết định vẫn là độ dày nghiệp vụ** (2.5): service có luật, invariant, quy trình → hexagonal hoàn vốn qua test nhanh + hạ tầng thay được. Service dịch-JSON-sang-SQL → hai file `main.go` + `handlers.go` với sqlc là kiến trúc *đúng*; đọc thẳng, sửa nhanh, không tầng nào để lạc.
- **Chi phí thật nằm ở kỷ luật duy trì, không ở setup**: vẽ cây thư mục mất một giờ; giữ cho dev thứ mười không import `platform/postgres` vào `order` "cho tiện deadline" mất... mãi mãi. Kiến trúc sống bằng review + linter, không bằng README.
- **Đừng nhầm tầng với thư mục**: gặp codebase đủ 4 folder Clean mà handler vẫn viết SQL — kiến trúc là *chiều mũi tên import*, không phải tên thư mục. Ngược lại, codebase phẳng 6 file mà domain thuần, I/O ở mép (functional core 2.1, 3.0) — hexagonal về bản chất dù không có folder nào tên "port".

Một dòng cho tranh luận cộng đồng Go: nhiều tiếng nói (kể cả trong Google) chê "Clean Architecture kiểu Java" là quá nghi lễ cho Go — và họ đúng *về phần nghi lễ*; nhưng phần lõi (domain không import hạ tầng) thì chính stdlib Go là minh chứng sống (case studies, 5.3). Giữ lõi, bỏ nghi lễ — đó là vị trí của tài liệu này.

---

*Tiếp theo: [5.2 — Event-driven & CQRS](/series/software-design/level-5-architecture/02-event-driven-cqrs/)*
