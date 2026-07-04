+++
title = "Bài 10 — Security cho Backend Systems"
date = "2026-02-01T16:00:00+07:00"
draft = false
tags = ["backend", "infrastructure", "networking"]
series = ["Infrastructure & Networking"]
+++

# Module 10 — Security cho Backend Systems

> Nguyên tắc chi phối toàn module: **defense in depth** (nhiều lớp — vì mỗi lớp SẼ có lỗ) và **least privilege** (mọi thứ chỉ được quyền tối thiểu — để khi bị xuyên, blast radius nhỏ nhất). Security không phải tính năng thêm sau; nó là thuộc tính kiến trúc như latency.

---

## 1. Authentication & Authorization: TLS/mTLS, JWT, OAuth2

### mTLS — máy xác thực máy
TLS thường: client xác thực server (module 01). **mTLS**: cả hai chiều — client cũng trình certificate. Đây là nền của **zero-trust nội bộ**: không tin "vì cùng mạng nội bộ" (perimeter model chết khi một máy bên trong bị chiếm — kẻ tấn công di chuyển ngang tự do). Vấn đề thật của mTLS không phải crypto mà là **vận hành vòng đời cert**: cấp phát, xoay ngắn hạn, thu hồi cho hàng nghìn service. Làm tay = bất khả thi; đây chính là lý do dùng service mesh (Istio/Linkerd tự động toàn bộ: SPIFFE identity, cert 24h, xoay tự động) hoặc SPIRE. Nếu không có nhu cầu compliance/zero-trust thật, cân nhắc kỹ trước khi trả chi phí vận hành mesh.

### JWT — token tự chứa, và cái giá của stateless
JWT = claims được **ký** (không mã hóa — ai cũng đọc được payload, đừng nhét secret/PII). Server verify chữ ký mà không gọi DB → authorization stateless, scale đẹp, đúng chỗ cho **access token sống ngắn**.

Cái giá cốt lõi: **không thu hồi được trước hạn** — token đã phát hành sẽ hợp lệ đến khi hết hạn, dù user đã logout/bị khóa. Thiết kế chuẩn giải quyết bằng cặp: access token ngắn (5–15 phút, stateless, verify local) + **refresh token dài (stateful, thu hồi được, lưu server-side)**. Cần đuổi ai đó, thu refresh token — cửa sổ rủi ro tối đa = TTL access token. Ai nói "JWT thay session hoàn toàn, không cần state" là chưa gặp yêu cầu "khóa tài khoản NGAY".

Lỗi triển khai kinh điển: chấp nhận `alg: none` hoặc để attacker đổi RS256→HS256 (thư viện cũ); không verify `aud`/`iss` (token service A dùng được ở service B); TTL 24h cho access token; ký bằng secret yếu. Dùng thư viện đã kiểm chứng, pin thuật toán, TTL ngắn.

### OAuth2 / OIDC — ủy quyền không đưa mật khẩu
Bài toán: app thứ ba cần truy cập tài nguyên của user mà user không đưa mật khẩu. OAuth2 = framework ủy quyền (access token có scope hẹp, thu hồi được); OIDC = lớp identity trên OAuth2 (ID token — "đăng nhập bằng Google"). Điều cần thuộc: dùng **Authorization Code + PKCE** cho mọi client public (SPA/mobile); Implicit flow đã bị khai tử; `state` chống CSRF; **Client Credentials** flow cho machine-to-machine. Đừng tự viết OAuth server — dùng Keycloak/Auth0/Cognito/ORY; sai sót ở đây là sai sót lộ toàn bộ user.

---

## 2. Secret Management

### Vấn đề và tiến hóa
Secret trong code/git = lộ vĩnh viễn (git history bất tử — đã commit là coi như đã lộ, **rotate ngay** chứ đừng chỉ xóa file). Secret trong env var = khá hơn nhưng rò qua `/proc/<pid>/environ`, crash dump, log framework in env, và không xoay được không restart.

Chuẩn hiện đại — **secret manager** (Vault, AWS Secrets Manager, GCP Secret Manager):
1. Lưu tập trung, mã hóa, **audit log mọi lần đọc** (khi sự cố: biết secret nào bị đọc bởi ai).
2. App **authenticate bằng danh tính nền tảng** thay vì bằng một secret khác (IAM role / Kubernetes ServiceAccount qua OIDC — phá vòng luẩn quẩn "secret zero": không còn credential tĩnh nào để lộ).
3. **Xoay tự động** và secret động (Vault phát credential DB sống 1 giờ, riêng cho từng instance — lộ cũng chết trong 1 giờ, và biết của ai).

Nguyên tắc thực dụng: mọi secret tĩnh sống lâu là nợ; đo lường bằng câu hỏi "**nếu secret này lộ lên GitHub công khai ngay bây giờ, tôi mất bao lâu để vô hiệu nó?**" — câu trả lời tốt là phút, qua một quy trình đã tự động. CI/CD dùng OIDC federation (GitHub Actions → AWS role) thay cho AWS key tĩnh trong repo settings — loại bỏ hẳn loại rò rỉ phổ biến nhất.

---

## 3. Bảo vệ biên: WAF và DDoS

- **WAF** (L7): lọc pattern tấn công (SQLi, XSS, path traversal) + rate limit theo IP/key + bot rules. Sự thật cần biết: WAF là **lưới an toàn, không phải sự thay thế cho code an toàn** (bypass WAF là một môn thể thao); rule mới phải chạy chế độ **count/monitor trước khi block** (WAF chặn nhầm traffic thật là sự cố tự gây kinh điển — đặc biệt API có payload lạ).
- **DDoS**: tấn công volumetric (trăm Gbps) không đỡ được ở server — phải hấp thụ ở **edge có băng thông lớn hơn attacker** (CloudFlare/CloudFront/Shield — anycast phân tán lực đập). Tầng ứng dụng: L7 DDoS (request hợp lệ, số lượng lớn) cần rate limit nhiều chiều (IP, user, endpoint đắt), cache mạnh, và load shedding (module 09). Kiến trúc đúng: origin **không có đường vào trực tiếp** (SG chỉ nhận từ CDN/LB) — attacker tìm được IP origin là toàn bộ lớp edge vô nghĩa.

---

## 4. Container & Kubernetes Security

(Tầng image/runtime đã có ở module 04: non-root, distroless, drop capabilities, không docker.sock, scan + ký. Đây là tầng cluster.)

### RBAC và danh tính
- RBAC theo least privilege: không ai/không service account nào có `cluster-admin` trong vận hành thường ngày; Role theo namespace; quyền `create pod` + `exec` gần tương đương root node — soi kỹ ai có.
- **ServiceAccount token** là mục tiêu số 1 khi pod bị chiếm: tắt automount khi pod không cần gọi API (`automountServiceAccountToken: false` — đa số app KHÔNG cần), dùng bound/expiring token, và IRSA/Workload Identity để pod nhận quyền cloud không qua key tĩnh.

### NetworkPolicy — bulkhead của mạng cluster
Mặc định Kubernetes: **mọi pod nói chuyện được với mọi pod** — một pod bị chiếm là bàn đạp quét cả cluster. NetworkPolicy default-deny theo namespace + allow tường minh (frontend→backend:8080, backend→db:5432) biến "chiếm 1 pod" thành "kẹt trong 1 ô". Chi phí thấp, hiệu quả cao, ít team làm — làm sớm khi đồ thị traffic còn nhỏ (retrofit lên 200 service = dự án khảo cổ).

### Chuỗi phòng thủ còn lại
Pod Security Standards (restricted: chặn privileged/hostPath/root) enforce theo namespace; admission control (Kyverno/Gatekeeper: chỉ image đã ký từ registry công ty, bắt buộc requests/limits); audit log của API server bật và đẩy ra ngoài cluster; etcd encryption at rest (secret nằm trong etcd!); runtime detection (Falco — báo hành vi lạ: shell trong container, đọc /etc/shadow).

### Mô hình ưu tiên khi nguồn lực hữu hạn (thứ tự ROI)
1. Vá đường vào phổ biến nhất: dependency/image CVE (scan + update tự động), secret lộ (mục 2).
2. Least privilege danh tính (IAM/RBAC/ServiceAccount) — thu nhỏ blast radius mọi kịch bản.
3. Network segmentation (SG chặt, NetworkPolicy).
4. Phát hiện & phản ứng (audit log, runtime detection) — vì lớp 1–3 sẽ có ngày thủng; khác biệt giữa "sự cố 1 giờ" và "âm thầm 6 tháng" nằm ở lớp này.

## Checklist
- [ ] TLS mọi nơi; cert tự động (module 01); mTLS nếu zero-trust là yêu cầu thật
- [ ] Access token ngắn + refresh thu hồi được; OAuth2 bằng sản phẩm đã kiểm chứng
- [ ] Không secret tĩnh trong code/CI; secret manager + danh tính nền tảng + xoay tự động
- [ ] Origin không lộ; WAF ở count-mode trước; rate limit đa chiều
- [ ] Image non-root đã ký + scan; RBAC/SA least privilege; NetworkPolicy default-deny; audit log ra ngoài cluster
- [ ] Diễn tập: "secret X lộ ngay bây giờ — làm gì trong 15 phút tới?" có câu trả lời thành văn
