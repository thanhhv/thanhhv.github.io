+++
title = "Bài 0 — Giới Thiệu Series"
date = "2026-02-01T06:00:00+07:00"
draft = false
tags = ["backend", "infrastructure", "networking"]
series = ["Infrastructure & Networking"]
+++

# Infrastructure Handbook — Từ First Principles đến Production

Bộ tài liệu chuyên sâu về hạ tầng cho hệ thống backend, viết cho Software/Backend/Senior Engineer, Tech Lead và Architect. Mục tiêu không phải liệt kê khái niệm — mà là giúp bạn **hiểu tại sao mỗi công nghệ tồn tại, nó đánh đổi điều gì, và làm sao thiết kế/vận hành/debug hệ thống production**.

## Cấu trúc

| Module | Nội dung | Vì sao đọc |
|---|---|---|
| [01 — Networking Fundamentals](/series/infrastructure-networking/01-networking-fundamentals/) | OSI/TCP-IP, IP/CIDR/NAT, TCP, UDP, DNS, HTTP 1.1→3, QUIC, TLS | Nền của mọi thứ; 80% sự cố "bí ẩn" nằm ở đây |
| [02 — Backend Networking](/series/infrastructure-networking/02-backend-networking/) | Connection pooling, LB, CDN, WebSocket, gRPC, service discovery, timeout/retry/circuit breaker | Cách service nói chuyện với nhau cho đáng tin cậy |
| [03 — Linux Internals](/series/infrastructure-networking/03-linux-internals/) | Process/thread/scheduler, memory, fd, epoll, signal, cgroup/namespace, OOM | Mọi thứ deploy đều là process Linux; hiểu nó = debug được mọi tầng trên |
| [04 — Container](/series/infrastructure-networking/04-container/) | Docker/containerd/runc, image/layer/overlayfs, networking, storage, security | Container = process được cách ly, không phải VM nhẹ |
| [05 — Kubernetes](/series/infrastructure-networking/05-kubernetes/) | Reconciliation model, Pod/probe, controllers, Service/Ingress, scheduler, autoscaling, volume | Hệ điều khiển hội tụ trạng thái — nắm mô hình, chi tiết tự sáng |
| [06 — CI/CD](/series/infrastructure-networking/06-cicd/) | Pipeline, artifact bất biến, rolling/blue-green/canary, rollback, feature flag, GitOps | Quản lý rủi ro thay đổi — nguồn sự cố số 1 |
| [07 — Cloud Infrastructure](/series/infrastructure-networking/07-cloud-infrastructure/) | VPC/subnet/routing, SG, NAT, managed services, autoscaling, multi-region, cost | Topology và kinh tế học của cloud |
| [08 — Observability](/series/infrastructure-networking/08-observability/) | Logging, metrics/Prometheus, tracing/OTel, SLI/SLO/SLA, alerting | Không đo được thì không vận hành được |
| [09 — Reliability Engineering](/series/infrastructure-networking/09-reliability-engineering/) | HA, DR/RTO/RPO, bulkhead, degradation, load shedding, chaos engineering, postmortem | Thiết kế để phục vụ user khi mọi thứ đang hỏng |
| [10 — Security](/series/infrastructure-networking/10-security/) | TLS/mTLS, JWT/OAuth2, secret management, WAF/DDoS, container & K8s security | Defense in depth + least privilege như thuộc tính kiến trúc |
| [11 — Kiến trúc thực tế](/series/infrastructure-networking/11-real-world-architectures/) | E-commerce, FinTech, Social, SaaS, Video, Messaging, Blockchain, AI Platform | Bài tập tổng hợp: từ workload suy ra kiến trúc |

## Cách đọc

- **Đọc tuần tự lần đầu**: các module xây trên nhau có chủ đích (TCP slow start ở module 01 giải thích connection pooling ở module 02; cgroup ở module 03 giải thích CPU throttling ở module 05; retry ở module 02 quay lại trong FinTech ở module 11).
- **Lộ trình theo vai trò**:
  - Backend Engineer: 01 → 02 → 03 → 08, rồi phần còn lại.
  - Tiếp quản vận hành Kubernetes: 03 → 04 → 05 → 08 → 09.
  - Chuẩn bị lên Architect: đọc hết, dừng lâu ở mọi bảng trade-off và module 11.
- **Dùng làm tài liệu tra cứu sự cố**: mỗi module có bảng troubleshooting/lệnh Linux; module 03 có bảng "triệu chứng → lệnh đầu tiên" đáng in ra.

## Triết lý của bộ tài liệu

Mỗi chủ đề đi theo mạch: **vấn đề gì → vì sao giải pháp cũ không đủ → công nghệ này hoạt động thế nào bên trong → đánh đổi cái gì → dùng ở production ra sao → khi nào KHÔNG dùng → hỏng thì debug thế nào**. Mọi kết luận kỹ thuật đều phải trả lời được: tại sao, đánh đổi gì, và điều gì xảy ra nếu làm ngược lại.

Ba bất biến bạn sẽ gặp đi gặp lại:
1. Mọi abstraction đều rò rỉ — kỹ sư giỏi biết tầng nào đang rò rỉ lên tầng nào.
2. Mọi thứ stateful ở giữa đường đều có giới hạn và timeout, và chúng fail im lặng.
3. Mọi cơ chế an toàn đều có giá — nghề kiến trúc là biết chỗ nào đáng trả và chỗ nào không.
