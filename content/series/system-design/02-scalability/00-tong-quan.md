+++
title = "Phần 2 — Scalability"
date = "2026-07-13T06:10:00+07:00"
draft = false
tags = ["backend", "system-design"]
series = ["System Design — Tư Duy Thiết Kế Hệ Thống"]
+++

> Chủ đề: Vertical Scaling, Horizontal Scaling, Stateless vs Stateful, Sticky Session, Load Balancer, Auto Scaling.

## Luận điểm trung tâm của phần này

Scalability không phải "chịu được nhiều tải" — mà là **chi phí tăng tuyến tính (hoặc chậm hơn) theo tải, không cần thiết kế lại** ([1.1](/series/system-design/01-foundations/01-requirements/)). Hệ scale kém không phải hệ chậm — là hệ mà tải tăng 2× thì chi phí tăng 10× hoặc phải viết lại.

Ba mệnh đề chi phối cả phần:

1. **Scale-up trước, scale-out sau** — máy to hơn không đòi thay đổi kiến trúc nào; dùng hết lá bài rẻ đó trước ([1.5 — thứ tự thử](/series/system-design/01-foundations/05-bottleneck-analysis/)).
2. **Scale-out đòi stateless — và state không biến mất, nó chỉ dọn nhà.** Toàn bộ nghệ thuật scale ngang tầng app là nghệ thuật *di dời state* về đúng chỗ.
3. **Load Balancer là nơi mọi lời hứa scale-out được thực thi hoặc phản bội** — health check, thuật toán phân tải, connection draining quyết định N máy có thật sự bằng N máy không.

## Mục lục

- [2.1. Vertical vs Horizontal Scaling — và bài toán state](/series/system-design/02-scalability/01-vertical-horizontal-scaling/)
- [2.2. Load Balancer — người gác cổng của scale-out](/series/system-design/02-scalability/02-load-balancer/)
- [2.3. Auto Scaling — co giãn theo tải mà không thức đêm](/series/system-design/02-scalability/03-auto-scaling/)

## Vị trí của phần này trong bức tranh lớn

Phần này lo **tầng app/stateless** — nơi scale ngang rẻ và sạch. Khi bottleneck chuyển xuống tầng dữ liệu, lời giải đổi hẳn bản chất: replica cho đọc ([4.2](/series/system-design/04-distributed-systems/02-replication-consistency/)), cache ([Phần 7](/series/system-design/07-caching/00-tong-quan/)), sharding cho ghi ([Phần 8](/series/system-design/08-data-partitioning/00-tong-quan/)). Nhầm tầng là anti-pattern số một của scaling: **thêm app instance khi nghẽn ở DB không những vô ích mà còn có hại** (thêm connection đè DB — [13.2](/series/system-design/13-production-failure-cases/02-database-failures/)).
