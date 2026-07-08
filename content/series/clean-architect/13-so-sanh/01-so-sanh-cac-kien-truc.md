+++
title = "Chương 13 — So sánh khách quan: Clean vs Layered vs Hexagonal vs Onion vs MVC vs Modular Monolith vs DDD"
date = "2026-07-08T14:00:00+07:00"
draft = false
tags = ["backend", "golang", "clean-architecture"]
series = ["Clean Architecture với Golang"]
+++

> **Level 3–4** · Mục tiêu: thoát khỏi tranh cãi danh pháp, chọn theo bài toán.

---

## 1. Khung so sánh

Mỗi "kiến trúc" thực chất trả lời một tập câu hỏi khác nhau — so sánh chỉ công bằng khi đặt đúng tầng câu hỏi:

- **Tầng tổ chức phụ thuộc trong một đơn vị deploy**: Layered, Hexagonal, Onion, Clean.
- **Tầng tổ chức delivery/UI**: MVC.
- **Tầng topology hệ thống**: Modular Monolith (và Microservices).
- **Tầng mô hình hóa nghiệp vụ**: DDD.

Chúng không loại trừ nhau — một hệ thống có thể đồng thời là: modular monolith (topology) × Clean Architecture (phụ thuộc) × DDD (mô hình) × MVC (trong tầng delivery).

## 2. Nhóm "cùng một ý tưởng": Hexagonal, Onion, Clean

| | Hexagonal (2005) | Onion (2008) | Clean (2012) |
|---|---|---|---|
| Ẩn dụ | Lục giác, ports & adapters | Các lớp vỏ hành | 4 vòng tròn đồng tâm |
| Insight trung tâm | App ở tâm; DB, UI, test đều là "adapter cắm vào port" — đối xứng hóa mọi I/O | Domain model ở lõi; mọi coupling hướng vào tâm | Dependency Rule + đặt tên tầng (Entities/Use Cases/Adapters/Frameworks) |
| Quy định phần trong | Không — chỉ định nghĩa ranh giới với ngoài | Có: Domain Model / Domain Services / Application Services | Có: Entities / Use Cases |
| Điểm mạnh riêng | Khái niệm port/adapter sắc bén nhất; "driving vs driven side" | Nhấn mạnh domain model làm tâm | Phát biểu quy tắc gọn nhất; phổ biến nhất |

**Kết luận thẳng:** ba cái này là **một trường phái**, khác biệt chủ yếu ở trọng tâm trình bày. Nếu team bạn cãi nhau "nên dùng Hexagonal hay Clean" — hai bên đang đồng ý với nhau mà không biết. Chọn từ vựng nào cả team thống nhất là được; tài liệu này dùng từ vựng Clean vì phổ biến, nhưng dùng khái niệm port/adapter của Hexagonal vì sắc hơn.

## 3. Clean vs Layered — khác biệt thật sự duy nhất

| Trục | Layered cổ điển | Clean/Hexagonal/Onion |
|---|---|---|
| Hướng phụ thuộc | Trên → xuống → **DB ở đáy**: business import data access | Mọi thứ → **domain ở tâm**: data access import business |
| Trung tâm hấp dẫn | Database (schema định hình model) | Domain model (nghiệp vụ định hình schema) |
| Test business | Cần DB hoặc mock tầng DAL đặt cạnh implementation | Fake cổng do business sở hữu — thuần RAM |
| Chi phí | Thấp hơn, quen thuộc | +interface, +mapping, +kỷ luật |
| Khi hợp lý | CRUD, báo cáo, ít rule — DB *đúng là* trung tâm thật của bài toán | Rule thật, sống lâu, nhiều delivery |

Một câu: **Layered + đảo mũi tên ở ranh giới business/data = Clean.** Toàn bộ khác biệt là Dependency Inversion (chương 1.3). Vì thế đường migrate Layered → Clean rất cụ thể: giữ nguyên các tầng, chỉ chuyển interface repository về phía business và dịch kiểu dữ liệu về domain — làm dần từng module (chương 14).

## 4. Clean vs MVC

MVC là pattern của **tầng delivery** (tách hiển thị / điều khiển / dữ liệu hiển thị), sinh ra cho GUI. Trong backend hiện đại, "MVC framework" thường nghĩa là: controller (handler) + model (ORM entity) + view (JSON serializer). Vấn đề không nằm ở MVC — mà ở chỗ **MVC không có câu trả lời cho câu hỏi "business logic để đâu"**, nên logic mặc định chảy vào controller (fat controller) hoặc model ORM (god model). Clean Architecture không thay thế MVC; nó **bổ sung phần MVC bỏ trống**: controller của MVC trở thành adapter mỏng gọi use case. So sánh "MVC vs Clean" đúng ra là "MVC-một-mình vs MVC-cộng-lõi-nghiệp-vụ".

## 5. Clean Architecture vs Modular Monolith vs Microservices

Khác tầng câu hỏi: Clean tổ chức **bên trong** một đơn vị; Modular Monolith / Microservices tổ chức **giữa** các đơn vị.

| Trục quyết định | Monolith thường | Modular Monolith | Microservices |
|---|---|---|---|
| Số team | 1–2 | 2–8 | nhiều, cần deploy độc lập thật |
| Ranh giới cưỡng chế bởi | quy ước | compiler (`internal/`) + CI | mạng (tuyệt đối) |
| Chi phí gọi chéo | 0 | 0 | mạng + serialize + partial failure |
| Consistency | tx chung dễ | tx chung được, kỷ luật dữ liệu nên có | phân tán (saga/outbox) bắt buộc |
| Chi phí vận hành | thấp nhất | thấp | cao (discovery, tracing, on-call × N) |
| Sai lầm điển hình | ranh giới mục nát | ranh giới giả (JOIN chéo module) | distributed monolith |

Quan hệ với Clean Architecture: như chuỗi e-commerce (chương 12) chứng minh — **Clean bên trong module là điều kiện để leo thang topology rẻ**. Modular monolith là điểm dừng đúng của đa số hệ thống; microservices là công cụ giải bài toán *tổ chức và vận hành*, không phải bài toán code.

## 6. Clean Architecture vs DDD

Không phải đối thủ — bổ khuyết (toàn bộ chương 9): Clean cho vỏ (ranh giới, hướng phụ thuộc), DDD cho ruột (VO, aggregate, bounded context) và cho **chiến lược chia hệ thống** (context map — thứ Clean hoàn toàn không bàn). Dùng Clean không DDD → nguy cơ vỏ đẹp ruột anemic. Dùng DDD không Clean → model hay nhưng bị framework/ORM ăn mòn dần. Hệ có rule thật: dùng cả hai, liều lượng theo chương 9 mục 7.

## 7. Bảng chọn nhanh theo bối cảnh

| Bối cảnh | Khuyến nghị |
|---|---|
| Script, tool, POC | Không kiến trúc — một package, viết thẳng |
| CRUD nội bộ, 1–2 dev, ít rule | Layered đơn giản hoặc flat + transaction script |
| SaaS có domain thật, 2–8 dev, sống nhiều năm | **Modular monolith × Clean (liều vừa) × DDD mức 1–3** — điểm ngọt của đa số |
| Nghiệp vụ phức tạp, nhiều team | Trên + bounded context nghiêm + event giữa module |
| Cần scale/deploy/compliance tách biệt có số liệu | Tách service *từng cái có lý do*, giữ Clean trong mỗi service |
| Hệ đọc cực nặng / search / feed | + CQRS mức 2–3 cho đúng phần đó |

## 8. Điều các trường phái đều đồng ý (và là thứ đáng nhớ nhất)

Bóc nhãn hiệu đi, mọi kiến trúc tốt hội tụ về bốn điều đã học ở Level 1: ranh giới đặt theo trục thay đổi; phụ thuộc trỏ về phía ổn định; nghiệp vụ tách khỏi cơ chế; ranh giới phải được máy móc cưỡng chế. Ai nắm bốn điều đó có thể *tự chế* kiến trúc phù hợp cho bối cảnh của mình — đó chính là mục tiêu của tài liệu này, thay vì trung thành với bất kỳ nhãn hiệu nào.

**Chương cuối:** [Level 4 — Evolutionary Architecture, Refactor Legacy, Tổ chức & Con người](/series/clean-architect/14-principal/01-evolutionary-architecture-va-migration/)
