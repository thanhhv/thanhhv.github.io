+++
title = "Phần 14 — Case Studies"
date = "2026-07-13T17:10:00+07:00"
draft = false
tags = ["backend", "system-design"]
series = ["System Design — Tư Duy Thiết Kế Hệ Thống"]
+++

## Mục lục

- [14.1. URL Shortener — bài tập khởi động hoàn hảo](/series/system-design/14-case-studies/01-url-shortener/)
- [14.2. Social Network — fan-out và celebrity problem](/series/system-design/14-case-studies/02-social-network/)
- [14.3. Chat Application — triệu kết nối sống](/series/system-design/14-case-studies/03-chat-application/)
- [14.4. Notification System — fan-out đa kênh qua những bên không tin được](/series/system-design/14-case-studies/04-notification-system/)
- [14.5. Banking & FinTech — khi sai một đồng là sai tất cả](/series/system-design/14-case-studies/05-banking-fintech/)
- [14.6. Video Streaming — băng thông là kiến trúc](/series/system-design/14-case-studies/06-video-streaming/)
- [14.7. Ride Hailing — geo real-time và dữ liệu phù du](/series/system-design/14-case-studies/07-ride-hailing/)
- [14.8. SaaS Platform — multi-tenancy và noisy neighbor](/series/system-design/14-case-studies/08-saas-platform/)
- [14.9. AI Platform — GPU đắt và hai chế độ phục vụ](/series/system-design/14-case-studies/09-ai-platform/)
- [14.10. Search System — Phần 9 trong hành động](/series/system-design/14-case-studies/10-search-system/)

*(E-commerce — case lớn nhất — là toàn bộ [Phần 12](/series/system-design/12-evolution/00-tong-quan/); Search System đóng vai case tổng hợp của [Phần 9](/series/system-design/09-search/00-tong-quan/).)*

---

> 12 hệ thống, thiết kế từng bước theo đúng chuỗi tư duy của [chương 00](/series/system-design/00-tu-duy-thiet-ke/): Business → FR → NFR → Scale Estimation → Constraint → Bottleneck → Pattern → Trade-off → Production → Evolution. **Không có sơ đồ cuối cùng cho sẵn** — giá trị nằm ở quá trình ra quyết định.

## Cách mỗi case study sẽ được viết

Mỗi bài bắt đầu từ một doanh nghiệp cụ thể với ràng buộc cụ thể (ngân sách, team, deadline), đi qua ước lượng bằng số, chỉ ra bottleneck *đặc trưng của domain đó*, rồi tiến hóa kiến trúc qua 2–3 mốc quy mô. Người đọc phải thấy được: cùng một domain, đổi một ràng buộc là đổi kiến trúc.

## Danh sách và "bài toán định hình" của từng hệ

| Case | Bài toán định hình (bottleneck đặc trưng) | Khái niệm trọng tâm |
|---|---|---|
| **E-commerce** | Tranh chấp tồn kho + flash sale spike | Đã viết trọn vẹn tại [Phần 12](/series/system-design/12-evolution/00-tong-quan/) |
| **Social Network** | Fan-out (1 post → N follower) + celebrity problem | Push vs pull feed, hot key, cache đa tầng |
| **Banking** | Đúng tuyệt đối + audit + compliance; throughput thấp một cách đáng ngạc nhiên | Ledger append-only, idempotency, reconciliation, DR khắt khe |
| **FinTech (ví điện tử)** | Như banking + tích hợp bên thứ ba mong manh + scale tiêu dùng | Saga cho giao dịch xuyên đối tác, [13.5 — 3rd party](/series/system-design/13-production-failure-cases/05-infrastructure-failures/) |
| **Chat Application** | Triệu kết nối dài (WebSocket) + thứ tự tin nhắn + presence | Stateful connection layer, message ordering per-conversation, sync đa thiết bị |
| **Video Streaming** | Băng thông là chi phí thống trị; ghi một lần đọc triệu lần | CDN là kiến trúc chứ không phải phụ kiện, transcode pipeline, adaptive bitrate |
| **Ride Hailing** | Dữ liệu vị trí cập nhật liên tục + matching thời gian thực theo địa lý | Geo-index, dữ liệu phù du (không cần durability), surge theo cell |
| **SaaS Platform** | Multi-tenancy: cô lập dữ liệu + noisy neighbor + tenant lớn gấp 10.000 lần tenant nhỏ | Shared vs isolated tenancy, [hot partition](/series/system-design/13-production-failure-cases/02-database-failures/), per-tenant limit |
| **AI Platform** | GPU đắt + inference latency + throughput batch | Queue cho batch, streaming cho chat, model versioning, cost-aware routing |
| **URL Shortener** | Bài "nhỏ mà thâm": đọc ≫ ghi 1000:1, latency là tất cả | Bài tập hoàn hảo cho [1.4 — estimation](/series/system-design/01-foundations/04-scale-estimation-capacity-planning/), cache, key generation |
| **Notification System** | Fan-out đa kênh + đối tác không tin được + không gửi trùng | [Idempotency, backlog, retry](/series/system-design/13-production-failure-cases/03-messaging-failures/) tổng hợp |
| **Search System** | Index lag vs độ tươi + relevance + tiếng Việt | Toàn bộ [Phần 9](/series/system-design/09-search/00-tong-quan/) trong hành động |

## Dùng phần này thế nào khi nó hoàn thành

Đừng đọc như đáp án. Với mỗi case: tự làm bước estimation và đoán bottleneck **trước khi đọc tiếp** — rồi so với bài. Khoảng cách giữa dự đoán của bạn và phân tích trong bài chính là thứ cần luyện; sự trùng khớp ngày càng tăng chính là "tư duy thiết kế hệ thống" đang hình thành.
