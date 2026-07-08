+++
title = "Chương 25 — So sánh khách quan: đặt 12-Factor cạnh các lựa chọn khác"
date = "2026-07-11T02:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> **Phần 4** | Chương trước: [Case study](/series/twelve-factor/24-case-study-rest-api/) | Chương sau: [Khi nào KHÔNG nên áp dụng](/series/twelve-factor/26-khi-nao-khong-nen-ap-dung/)

Các chương trước đã rải trade-off theo từng chủ đề. Chương này gom chúng thành các bảng so sánh trực diện — công cụ cho các buổi tranh luận kiến trúc, nơi câu hỏi không phải "cái nào hay hơn" mà là **"cái nào đúng cho ràng buộc của chúng ta"**.

---

## 1. Traditional Deployment vs 12-Factor App

| Tiêu chí | Traditional (VM + deploy tay) | 12-Factor trên nền tảng tự động |
|---|---|---|
| Chi phí khởi đầu | **Rất thấp** — một VM, một binary | Trung bình — pipeline, registry, nền tảng |
| Tần suất release an toàn | Tuần–quý | Ngày–giờ |
| Rollback | Cầu may | Một lệnh, một phút |
| Scale | Dọc, có trần, chậm | Ngang, theo phút, hai chiều |
| Phục hồi sự cố máy | Giờ–ngày (snowflake) | Tự động (cattle) |
| Kỹ năng đòi hỏi | Sysadmin cơ bản | Container, CI/CD, orchestration |
| Điểm hòa vốn | Hệ nhỏ, ít thay đổi, 1–2 người | Nhiều release, nhiều instance, nhiều người |

**Điểm giống** thường bị bỏ qua: cả hai đều cần kỷ luật version control, backup, giám sát — 12-Factor không thay thế những cái đó, nó xây trên chúng. **Khi nào chọn traditional**: đã phân tích xuyên suốt (chương 1, 26) — hệ nhỏ ổn định, một máy đủ, đội mỏng. Lưu ý chiều ngược: chọn traditional *rồi vẫn giữ* config-tách-code, log-stdout, Git — 4 factor rẻ nhất không có lý do bỏ ở bất kỳ mô hình nào.

## 2. VM vs Container

| Tiêu chí | VM | Container |
|---|---|---|
| Cô lập | Kernel riêng — mạnh nhất | Chung kernel — đủ cho đa số, không đủ cho multi-tenant thù địch |
| Khởi động | Phút | Mili giây–giây → F9, autoscale có nghĩa |
| Mật độ / chi phí | Chục/host | Trăm/host |
| Artifact | Image VM (to, chậm build) | Image layer (MB, build giây, registry) → F5 |
| Vòng đời điển hình | Tháng–năm | Giờ–ngày |

**Không phải hoặc–hoặc**: kiến trúc phổ biến nhất hiện nay là container *chạy trên* VM (node K8s là VM). Chọn VM thuần khi: cô lập cứng theo compliance, phần mềm đóng gói kiểu cũ (license theo máy), workload cần kernel/driver riêng (GPU đặc thù, kernel module). Chọn container cho mọi workload app thông thường — và microVM (Firecracker) là điểm giữa cho multi-tenant cần cả hai.

## 3. Config File vs Environment Variable (vs Secret Manager)

Đã phân tích sâu ở chương 8 (bảng 3 cột đầy đủ). Kết luận rút gọn: trục quan trọng không phải "file hay env" mà là **config có tách khỏi artifact không** — file mount lúc runtime và env var đều đạt; file nướng vào image và const trong code đều trượt. Thực hành 2026: env cho non-secret; file mount/secret manager cho secret; workload identity để *xóa sổ* secret khi có thể.

## 4. Stateful vs Stateless (application tier)

| | Stateful process | Stateless process |
|---|---|---|
| Latency state | RAM — nhanh nhất | + network hop tới Redis/DB (~0.5–2ms) |
| Scale ngang / self-healing / rolling | Khó hoặc mất dữ liệu | Tự do |
| Độ phức tạp hạ tầng | Thấp (một máy) | Cần backing services |
| Nơi đúng chỗ | Database, broker, game world, collab engine — *có công cụ riêng* (StatefulSet, consensus) | **Toàn bộ application tier** |

Câu trả lời trưởng thành không phải "stateless mọi thứ" mà là **dồn state về các thành phần chuyên trách state** (đã hardening: DB, cache, queue) và giữ tầng app — nơi thay đổi nhanh nhất, deploy nhiều nhất — sạch state. Đẩy độ khó về nơi ít thay đổi.

## 5. Monolith vs Microservices — và "Cloud-Native Monolith"

Nhầm lẫn phổ biến nhất cần đập tan: **12-Factor không bảo bạn chia microservices.** Tài liệu gốc viết cho *một* app. Ma trận đúng là hai trục độc lập:

```
                        KHÔNG 12-Factor              CÓ 12-Factor
                 ┌──────────────────────────┬────────────────────────────┐
   MONOLITH      │ Legacy điển hình          │ ★ CLOUD-NATIVE MONOLITH:   │
                 │ (chương 1)                │ một app stateless, scale    │
                 │                           │ ngang, CI/CD, trên PaaS/K8s │
                 │                           │ → điểm khởi đầu ĐÚNG cho    │
                 │                           │   đa số sản phẩm            │
                 ├──────────────────────────┼────────────────────────────┤
   MICROSERVICES │ Distributed monolith —    │ Kiến trúc các công ty lớn — │
                 │ tệ nhất hai thế giới      │ trả chi phí phối hợp để mua │
                 │ (N service stateful       │ tự chủ cho N team           │
                 │  deploy tay, chung DB)    │                             │
                 └──────────────────────────┴────────────────────────────┘
```

Microservices giải bài toán **tổ chức** (nhiều team deploy độc lập, scale/công nghệ khác nhau theo domain) với cái giá kỹ thuật rất thật (network thay function call, distributed transaction, vận hành N hệ). Đường đi được khuyến nghị rộng rãi: **cloud-native monolith trước, tách service khi có vết cắt rõ ràng và cơn đau tổ chức thật** — và vì monolith đã 12-Factor, việc tách sau này rẻ hơn nhiều (module ranh giới sạch, hạ tầng sẵn).

## 6. 12-Factor App và Kubernetes — hợp đồng và bên thi hành

Không phải hai lựa chọn cạnh tranh mà là hai mặt của một quan hệ (chương 20 đã lập bảng ánh xạ đầy đủ):

- **Giống**: cùng giả định hạ tầng tạm bợ; cùng triết lý khai báo, bất biến, tự động.
- **Khác**: 12-Factor là *tính chất của app* (viết thế nào), K8s là *nền tảng thi hành* (chạy ở đâu); 12-Factor tồn tại trước K8s 3 năm và chạy tốt trên Heroku/Cloud Run/ECS — K8s không phải điều kiện.
- **Bất đối xứng đáng nhớ**: app 12-Factor không cần K8s (PaaS phục vụ tốt); nhưng **K8s rất cần app 12-Factor** — app stateful, start chậm, không bắt SIGTERM sẽ vô hiệu hóa lần lượt từng tính năng của K8s và biến nó thành cỗ máy đắt tiền chạy pet.
- **Khi nào chọn gì**: thang ở chương 20 mục 8 — PaaS cho fleet nhỏ, K8s khi quy mô tổ chức đòi chuẩn hóa.

## 7. Bảng quyết định tổng hợp — dùng trong 5 phút

| Tình huống của bạn | Khuyến nghị mặc định |
|---|---|
| MVP, 1–3 dev, tìm product-market fit | Cloud-native monolith (Go + Postgres managed) trên PaaS; đủ 4 factor rẻ (F1/F3/F11 + env parity bằng compose) |
| SaaS đang lớn, 5–15 dev, release hàng tuần | Monolith + worker tách (F8), managed K8s hoặc PaaS mạnh, CI/CD đầy đủ, observability RED+SLO |
| Nhiều team, nhiều domain, xung đột deploy | Bắt đầu tách service theo ranh giới team; GitOps, platform engineering, progressive delivery (chương 23) |
| Hệ nội bộ ổn định, ít thay đổi | Traditional có kỷ luật (Git, config tách, backup test định kỳ) — đừng migrate vì thời thượng |
| Hệ stateful lõi (DB, broker tự vận hành) | StatefulSet + operator trưởng thành, hoặc trả tiền managed — đừng tự chế |

---

## Tóm tắt

- Mọi so sánh quy về một câu: **mua tính lựa chọn cho tương lai bằng độ phức tạp hôm nay — mua đúng lúc, đúng giá**.
- Hai trục hay bị trộn: 12-Factor ⊥ microservices (cloud-native monolith là giao điểm bị đánh giá thấp nhất), 12-Factor ⊥ Kubernetes (hợp đồng vs bên thi hành).
- Bốn factor rẻ (codebase, config tách, log stdout, dependency tường minh) thắng trong *mọi* cột của *mọi* bảng — đó là mẫu số chung an toàn tuyệt đối của cả tài liệu.
