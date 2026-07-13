+++
title = "Phần 11 — Security"
date = "2026-07-13T13:40:00+07:00"
draft = false
tags = ["backend", "system-design"]
series = ["System Design — Tư Duy Thiết Kế Hệ Thống"]
+++

> Chủ đề: Authentication, Authorization, OAuth2, JWT, Rate Limiting, WAF, API Gateway.

## Luận điểm trung tâm của phần này

Security là **thuộc tính kiến trúc, không phải tính năng gắn sau** — và giống durability ([1.1 §9](/series/system-design/01-foundations/01-requirements/)), lỗi của nó không sửa được bằng refactor: dữ liệu đã rò là đã rò. Ba nguyên tắc gốc chi phối mọi chương:

1. **Defense in depth:** nhiều lớp, không lớp nào được tin là đủ — kẻ tấn công phải vượt *tất cả*, còn bạn chỉ cần *một* lớp bắt được.
2. **Least privilege:** mọi thành phần (người, service, token) chỉ có quyền tối thiểu cho việc của nó — blast radius của một credential bị lộ tỷ lệ thuận với quyền nó mang.
3. **Không tự chế crypto/auth:** dùng chuẩn đã được soi (OAuth2/OIDC, JWT ký chuẩn, thư viện đã kiểm toán) — sáng tạo trong security là sáng tạo cách thua.

## Mục lục

- [11.1. Authentication & Authorization — bạn là ai, bạn được làm gì](/series/system-design/11-security/01-authn-authz/)
- [11.2. OAuth2, OIDC & JWT — ủy quyền và token trong hệ phân tán](/series/system-design/11-security/02-oauth2-jwt/)
- [11.3. Biên phòng thủ — API Gateway, Rate Limiting, WAF](/series/system-design/11-security/03-gateway-ratelimit-waf/)

## Bản đồ security trong toàn tài liệu

| Mảnh | Ở đâu |
|---|---|
| Compliance là NFR (Nghị định 13, PCI-DSS) | [1.1 §3.2](/series/system-design/01-foundations/01-requirements/) |
| Che PII trong log/tín hiệu | [10.1 §3](/series/system-design/10-observability/01-ba-tru/), [10.2 §6](/series/system-design/10-observability/02-opentelemetry-pipeline/) |
| Backup mã hóa, immutable, chống ransomware | [3.2](/series/system-design/03-availability-reliability/02-backup-recovery/) |
| ACL cho topic/queue — ai được phát/nghe | [6.5 §6](/series/system-design/06-communication/05-kafka/) |
| Idempotency chống replay nghiệp vụ | [13.3](/series/system-design/13-production-failure-cases/03-messaging-failures/) |
| Rate limiting như công cụ reliability | [13.1 — thundering herd](/series/system-design/13-production-failure-cases/01-caching-failures/) |

Phần này đứng ở góc **kiến trúc sư hệ backend** — identity, token, và biên — không thay thế được chuyên môn AppSec/PenTest; nó bảo đảm phần *thiết kế* không tạo ra những lỗ mà không PenTest nào vá nổi.
