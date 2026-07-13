+++
title = "Phần 7 — Caching"
date = "2026-07-13T11:00:00+07:00"
draft = false
tags = ["backend", "system-design"]
series = ["System Design — Tư Duy Thiết Kế Hệ Thống"]
+++

> Chủ đề: Cache Aside, Read Through, Write Through, Write Back, Cache Invalidation, Distributed Cache.

## Luận điểm trung tâm của phần này

Cache tồn tại vì hai sự thật: RAM nhanh hơn disk ~1000 lần ([chương 00 §3](/series/system-design/00-tu-duy-thiet-ke/)), và truy cập dữ liệu có **locality** — một phần nhỏ dữ liệu nhận phần lớn truy cập (luật lũy thừa, [13.2 — hot partition](/series/system-design/13-production-failure-cases/02-database-failures/) là mặt tối của cùng quy luật). Không có locality, cache vô dụng; có locality, cache là đòn bẩy hiệu năng rẻ nhất trong toàn bộ hộp đồ nghề.

Nhưng cache là **bản sao** — và mọi bản sao đặt ra câu hỏi consistency ([4.2](/series/system-design/04-distributed-systems/02-replication-consistency/)). Toàn bộ nghệ thuật caching gói trong một câu: **chịu stale bao lâu, và trả giá invalidation ở đâu.** Ba chương của phần này là ba mặt của câu đó:

- [7.1. Bốn chiến lược cache — ai ghi, ai đọc, ai chịu trách nhiệm](/series/system-design/07-caching/01-cache-strategies/)
- [7.2. Cache Invalidation — bài toán khó thứ nhất của khoa học máy tính](/series/system-design/07-caching/02-cache-invalidation/)
- [7.3. Distributed Cache — cache khi một node không đủ](/series/system-design/07-caching/03-distributed-cache/)

## Bản đồ kiến thức cache trong toàn tài liệu

Phần này là "lý thuyết trung tâm"; các mảnh thực chiến đã nằm ở chỗ của chúng:

| Mảnh | Ở đâu |
|---|---|
| Đưa cache vào hệ thống đúng lúc, đúng cách | [12.2 — Thêm Redis](/series/system-design/12-evolution/02-them-redis/) |
| Redis — công cụ cache chủ lực | [5.4](/series/system-design/05-data-layer/04-redis/) |
| Ba sự cố cache kinh điển: Stampede, Avalanche, Thundering Herd | [13.1](/series/system-design/13-production-failure-cases/01-caching-failures/) |
| Hot key | [13.2](/series/system-design/13-production-failure-cases/02-database-failures/), [7.3](/series/system-design/07-caching/03-distributed-cache/) |
| Cache như projection của CQRS | [12.8](/series/system-design/12-evolution/08-cqrs/) |

## Nguyên tắc bất di bất dịch — nhắc lại một lần cho cả phần

**Mọi quyết định ghi đọc từ nguồn sự thật, không đọc cache.** Cache phục vụ hiển thị; transaction phục vụ sự thật. Vi phạm nguyên tắc này là nguồn của các bug tiền bạc khó tái hiện nhất ([12.2 §3](/series/system-design/12-evolution/02-them-redis/), [12.8 §7](/series/system-design/12-evolution/08-cqrs/)).
