+++
title = "Backend Security — Bộ tài liệu chuyên sâu (Tiếng Việt)"
date = "2026-07-07T07:00:00+07:00"
draft = false
tags = ["backend", "security"]
series = ["Backend Security"]
+++

> Bộ tài liệu 10 tập về **Backend Security**, viết cho Backend Engineer, Senior Backend Engineer, Tech Lead, Solution Architect và Software Architect.
>
> **Triết lý:** không giải thích "cách dùng", mà giúp người đọc **hiểu bản chất** — attacker suy nghĩ thế nào, vì sao cơ chế bảo mật tồn tại, nó bảo vệ điều gì, bị khai thác ra sao, và khi nào *không* nên dùng.

## Cách bộ tài liệu được xây dựng

Mỗi chủ đề bám một mạch tư duy cố định, **không bắt đầu bằng định nghĩa**:

**Asset → Threat → Attack → Vulnerability → Defense → Trade-off → Production Best Practice.**

Mỗi công nghệ được soi qua 10 lăng kính: Problem Statement · Threat Model · Vì sao tồn tại · Cách hoạt động bên trong · Trade-off (Security/Performance/DX/Cost/Complexity/Scalability) · Best Practice · Anti-pattern · Production Case Study · Troubleshooting · Khi nào KHÔNG nên dùng.

- **Định dạng:** Markdown, sơ đồ dùng **Mermaid** (render trên GitHub, GitLab, Obsidian, VS Code, và nhiều wiki).
- **Thuật ngữ chuyên ngành** (Authentication, JWT, OAuth2, TLS, CSRF, SSRF...) giữ nguyên tiếng Anh.
- **Tổng ~45.800 từ**, ~40 sơ đồ Mermaid.

## Mục lục 10 tập

| # | Tập | Nội dung chính |
|---|-----|----------------|
| 01 | [Security Foundations](/series/backend-security/01-security-foundations/) | CIA Triad · Threat Modeling (STRIDE) · Least Privilege · Defense in Depth · Zero Trust · AuthN vs AuthZ |
| 02 | [Authentication](/series/backend-security/02-authentication/) | Basic · Session · Cookie vs Token · Token/Stateless · JWT · OAuth2 · OIDC · Refresh/Rotation/Revocation |
| 03 | [Password Security](/series/backend-security/03-password-security/) | Storage · Salt · Pepper · MD5 · SHA · PBKDF2 · bcrypt · scrypt · Argon2 · vì sao hash phải chậm |
| 04 | [Transport Security](/series/backend-security/04-transport-security/) | HTTP/HTTPS · SSL vs TLS · TLS Handshake · Certificate & CA · Mutual TLS |
| 05 | [Browser Security](/series/backend-security/05-browser-security/) | Same-Origin Policy · CORS · XSS · CSRF · CSP · Cookie (Secure/HttpOnly/SameSite) |
| 06 | [API Security](/series/backend-security/06-api-security/) | API Key · Rate Limiting · Replay · Idempotency · Request Signing/HMAC · BOLA |
| 07 | [OWASP Top 10](/series/backend-security/07-owasp-top-10/) | A01→A10 đầy đủ, mỗi mục: Cơ chế · Attack · Demo · Phòng tránh |
| 08 | [Server Security](/series/backend-security/08-server-security/) | Secret Mgmt · Env Vars · Firewall · Reverse Proxy · WAF · Docker/K8s · File Perm · SSH |
| 09 | [API Best Practices](/series/backend-security/09-api-security-best-practices/) | Hành trình request qua 8 trạm phòng thủ · validation · logging · rotation · versioning |
| 10 | [Kiến trúc thực tế](/series/backend-security/10-real-world-architectures/) | E-commerce · Banking · FinTech · SaaS · Mobile · Microservices · Public API |

## Gợi ý lộ trình đọc

- **Người mới với security:** đọc tuần tự 01 → 10. Tập 01 là nền tảng tư duy cho tất cả.
- **Đã có nền, cần chiều sâu cụ thể:** đọc 01 (khung tư duy) rồi nhảy tới tập quan tâm; mỗi tập đều liên kết ngược về các tập liên quan.
- **Ôn nhanh trước phỏng vấn/review:** phần "Tổng kết" cuối mỗi tập + bảng tổng hợp ở Tập 09 và 10.

## Mười sợi chỉ đỏ xuyên suốt

1. Security là môn học về **đối thủ** — hỏi "attacker muốn gì, đi đường nào" trước.
2. **Giả định thất bại** — thiết kế để giới hạn thiệt hại + phát hiện, không chỉ ngăn chặn.
3. **CIA cho từng tài sản** — "an toàn" phải cụ thể theo trục và tài sản.
4. **Least Privilege** ở mọi tầng — giới hạn bán kính vụ nổ.
5. **Defense in Depth** — không lớp nào là điểm chết; không tin lớp trước; fail securely.
6. **Zero Trust** — không tin theo vị trí mạng; verify mọi truy cập.
7. **AuthN ≠ AuthZ** — nhầm lẫn là rủi ro #1 (Broken Access Control).
8. **Không tin client** — mọi kiểm soát ở server.
9. **Không thần thánh hóa công nghệ** — mỗi thứ là một trade-off, đều có chỗ không nên dùng.
10. **Phát hiện quan trọng ngang phòng ngừa** — logging/audit/monitoring là điều kiện vận hành mọi phòng thủ.
