+++
title = "Bài 15 — Kiến Trúc Thực Tế"
date = "2026-07-01T08:00:00+07:00"
draft = false
tags = ["backend", "golang", "nodejs"]
series = ["NodeJS & Golang"]
+++

# Kiến trúc thực tế — Chọn Go hay Node cho 11 loại hệ thống

> Cấu trúc mỗi mục: đặc tính tải → lựa chọn khuyến nghị → vì sao / vì sao không phương án kia → scale & chi phí → rủi ro chính. Mọi khuyến nghị là **mặc định hợp lý**, bị ghi đè bởi yếu tố con người (chương 14, mục 2.2).

---

## 1. API Gateway

- **Tải:** connection-heavy, mọi request của hệ thống đi qua; việc chính là route, authn, rate limit, transform nhẹ — I/O-bound nhưng **p99 của gateway cộng vào p99 của tất cả**.
- **Khuyến nghị: Go** (không ngẫu nhiên mà Kong gateway data plane, Traefik, KrakenD, Tyk là Go/nền C). Lý do: tail latency ổn định (GC sub-ms), một process ăn mọi core → ít hop LB nội bộ, memory/connection thấp cho hàng trăm nghìn kết nối keep-alive.
- **Vì sao không Node:** một transform plugin viết ẩu block loop là **mọi** route cùng chậm — blast radius cực đại đúng chỗ nhạy cảm nhất. Vẫn hợp lý khi: gateway nhẹ, đội JS, cần viết plugin logic nghiệp vụ nhanh (BFF-gateway lai).
- **Thực tế trưởng thành:** đừng tự viết nếu nhu cầu là chuẩn (authn, rate limit, routing) — dùng Kong/Envoy/Traefik, chỉ viết plugin. Tự viết gateway chỉ đáng khi transform logic là lợi thế cạnh tranh.
- **Rủi ro chính:** gateway tự viết trở thành SPOF + nút cổ chai kiến thức của một người.

## 2. Authentication Service

- **Tải:** RPS cao (mọi service hỏi), **CPU-bound thật sự ở điểm nóng**: bcrypt/argon2 (cố ý đắt ~50-300ms CPU), ký/verify JWT, TLS.
- **Khuyến nghị: Go.** Hash password song song trên mọi core tự nhiên; Node phải đẩy bcrypt qua thread pool libuv (4 thread mặc định — 5 login đồng thời là nghẽn, và nghẽn *chung* với fs/DNS!) hoặc worker pool tự quản.
- **Node vẫn ổn khi:** dùng managed auth (Auth0, Cognito, Keycloak) và service chỉ là lớp mỏng OIDC — lúc này là I/O-bound thuần.
- **Rủi ro chính:** không phụ thuộc ngôn ngữ — là crypto tự chế. Dùng thư viện chuẩn, mua/dùng open source (Keycloak, Ory — Ory viết bằng Go) trước khi tự viết.

## 3. Payment Service

- **Tải:** RPS thường **thấp** (nghìn TPS đã là rất lớn), nhưng đòi hỏi đúng đắn tuyệt đối: idempotency, transaction, audit, reconciliation. I/O-bound (chờ ngân hàng/PSP hàng trăm ms).
- **Khuyến nghị: hòa về hiệu năng — quyết định bằng tính đúng đắn.** Go có lợi thế: static typing chặt + kiểu số rõ ràng (và kỷ luật dùng số nguyên minor unit / thư viện decimal — **không bao giờ float cho tiền**, đúng ở mọi ngôn ngữ; JS `Number` toàn float nên nguy cơ trượt tay cao hơn, phải ép kỷ luật BigInt/decimal.js). Node có lợi thế: tích hợp PSP (Stripe SDK v.v.) tốt.
- **Kiến trúc quan trọng hơn ngôn ngữ:** outbox + idempotency key (chương 6) + saga cho flow nhiều bước + ledger append-only. Một team viết Node kỷ luật ăn đứt team Go cẩu thả ở domain này.
- **Rủi ro chính:** xử lý double-charge/mất tiền do retry không idempotent — lỗi thiết kế, không lỗi runtime.

## 4. Notification Service (email/push/SMS)

- **Tải:** consumer từ queue, fan-out lớn, I/O-bound gần tuyệt đối (chờ FCM/APNs/SMTP), bursty (chiến dịch marketing bắn 10M push).
- **Khuyến nghị: cả hai đều tốt — chọn theo đội.** Node tự nhiên với template render + SDK provider phong phú. Go tự nhiên với worker pool + rate limit per-provider (chương 3, 6) và tốn ít máy hơn khi burst chục triệu.
- **Điểm kiến trúc quyết định:** rate limit theo từng provider (APNs/FCM có quota), retry với DLQ, idempotent theo message ID (gửi trùng notification là lỗi user-facing rất khó chịu), và priority queue (OTP phải vượt mặt marketing).
- **Rủi ro chính:** retry storm tự DDoS provider → bị throttle toàn tổ chức.

## 5. Realtime Chat

- **Tải:** connection-heavy (WebSocket giữ lâu), mỗi connection ít việc; fan-out theo room; presence.
- **Khuyến nghị: Go nghiêng nhẹ ở scale lớn.** 1M kết nối WebSocket: Go ~vài GB (goroutine + buffer/conn kiểm soát được); Node cần nhiều process/pod hơn đáng kể và GC pressure từ churn message object. Ở scale <50K connection: Node + socket.io năng suất hơn (reconnect, room, fallback có sẵn — Go phải tự ghép từ gorilla/nhold + redis pub/sub).
- **Kiến trúc quan trọng hơn:** tách connection layer (stateful, scale theo connection) khỏi business layer (stateless, scale theo RPS); pub/sub (Redis/NATS) nối các node connection; sticky theo user chỉ ở tầng connection.
- **Rủi ro chính:** thundering herd khi deploy/restart node connection (100K client reconnect cùng lúc) — cần jitter reconnect phía client + connection draining.

## 6. Social Network (feed, profile, graph)

- **Tải:** read-heavy chênh lệch lớn (đọc:ghi ~100:1), fan-out feed là bài toán riêng, cache là trung tâm kiến trúc.
- **Khuyến nghị: đa ngôn ngữ theo tầng** — mẫu hình ngành: API/BFF bằng Node (tốc độ ra tính năng, đội full-stack), tầng nền bằng Go/Java/C++ (feed ranking, fanout worker, media processing). Cãi nhau "một ngôn ngữ cho cả social network" là đặt sai câu hỏi khi hệ đủ lớn.
- **Chi phí:** ở scale này hạ tầng >> lương — Go ở các đường nóng (feed API đọc 100K RPS) hoàn vốn nhanh.
- **Rủi ro chính:** cache stampede khi key nóng hết hạn (singleflight — chương 3/12) và hot partition (celebrity problem) — thiết kế fanout hybrid push/pull.

## 7. E-commerce

- **Tải:** hỗn hợp điển hình: catalog read-heavy + cache tốt; checkout write + transactional; search; ảnh; flash sale = burst ×100.
- **Khuyến nghị:** storefront/BFF: **Node** (SSR Next.js, đội frontend chủ động end-to-end); inventory/checkout/pricing: **Go** (concurrency có kỷ luật quanh stock — trừ kho chính xác dưới burst là bài toán lock/atomic đúng sở trường; và flash sale cần headroom + load shedding của chương 3).
- **Vì sao không thuần một thứ:** thuần Node → flash sale và tính tồn kho là hai điểm run tay; thuần Go → đội frontend bị tách khỏi backend BFF, vòng lặp tính năng chậm.
- **Rủi ro chính:** oversell khi flash sale (giải bằng reserve stock atomic — Redis Lua/DB, không giải bằng "hy vọng").

## 8. FinTech (core banking, trading, ledger)

- **Tải & yêu cầu:** đúng đắn > tất cả; audit; p99 chặt (trading); compliance.
- **Khuyến nghị: Go là mặc định tốt cho core** (ledger, matching, risk): static typing, latency ổn, một binary dễ kiểm soát môi trường, ecosystem fintech Go dày (nhiều ngân hàng số lớn xây core bằng Go). Node hợp ở rìa: onboarding flow, dashboard, tích hợp đối tác.
- **Ranh giới cứng:** matching engine HFT độ trễ µs → cả Go GC lẫn Node đều không đủ — C++/Rust/FPGA. Biết giới hạn trên của công cụ là năng lực architect.
- **Rủi ro chính:** dùng float cho tiền (nhắc lần hai vì nó vẫn cứ xảy ra); và eventual consistency đặt nhầm chỗ — số dư khả dụng không được "eventual".

## 9. Streaming Platform (video/audio)

- **Tách bài toán đúng trước khi chọn:** (a) serve video = bài toán của CDN + nginx/object storage, không viết tay; (b) transcode = FFmpeg/GPU do worker điều phối; (c) control plane (playlist, DRM license, analytics ingest) = API thường.
- **Khuyến nghị:** worker điều phối transcode: **Go** (quản lý process, pipeline, tận CPU máy worker); control plane: **Go hoặc Node** đều ổn (I/O-bound); analytics ingest số lớn: Go/Kafka.
- **Rủi ro chính:** tự viết cái mà nginx/CDN/FFmpeg đã giải — chi phí cơ hội khổng lồ.

## 10. Blockchain Infrastructure

- **Thực tế ngành:** node/client blockchain lớn viết bằng Go (go-ethereum, Cosmos SDK, nhiều client Bitcoin alt) và Rust (Solana, Polkadot) — cần: crypto CPU-heavy, P2P networking hàng nghìn peer, kiểm soát binary. Node.js **không phù hợp cho tầng node/consensus** (CPU + GC không kiểm soát đủ).
- **Node.js đúng chỗ ở:** tầng ứng dụng — indexer nhẹ, API đọc chain, bot, dApp backend (ecosystem ethers.js/web3.js mạnh nhất ở JS).
- **Rủi ro chính:** bug trong code chạm tiền là không thể vá hồi tố; audit + test property-based nặng hơn mọi domain khác.

## 11. AI Platform

- **Tách tầng:** training/inference = Python/C++/CUDA (không bàn); **serving & orchestration** mới là chỗ Go/Node cạnh tranh.
- **Khuyến nghị:** model gateway / inference router (batching, quota, streaming token, A/B model): **Go** — throughput + streaming + đếm token CPU nhẹ nhưng RPS lớn. Application layer trên LLM (chatbot product, RAG glue, tool-calling): **Node/TS rất mạnh** — vòng lặp sản phẩm nhanh, SDK AI hạng nhất, streaming SSE/WebSocket tự nhiên với async iterator.
- **Đặc thù đáng chú ý:** workload LLM là I/O-bound kiểu mới (chờ GPU service hàng giây, giữ hàng nghìn stream mở) — cả hai đều hợp; Go tiết kiệm connection overhead, Node ghép UX nhanh.
- **Rủi ro chính:** capacity planning theo token chứ không theo request — sai đơn vị đo là sai hết autoscale/cost model.

---

## Tổng kết mẫu hình (pattern of patterns)

Nhìn lại 11 hệ thống, các quy luật lặp lại:

1. **Tầng chịu tải nền (gateway, connection, worker CPU) nghiêng Go; tầng sản phẩm thay đổi nhanh (BFF, dashboard, glue) nghiêng Node.** Đa số tổ chức trưởng thành hội tụ về mô hình hai tầng này.
2. **Kiến trúc (idempotency, outbox, cache, queue, load shedding) quyết định thành bại nhiều hơn ngôn ngữ** ở 9/11 trường hợp — ngôn ngữ chỉ quyết định ở hai cực: CPU-bound nặng và connection scale cực đại.
3. **"Đừng tự viết" là khuyến nghị xuất hiện nhiều nhất** — gateway, auth, video serving: dùng đồ chín, dồn nội lực vào phần khác biệt hóa.
4. Chi phí chuyển đổi ngôn ngữ giữa chừng rất cao — nhưng **chi phí không bao giờ tách service khỏi ngôn ngữ sai còn cao hơn**: thiết kế ranh giới service sạch (chương 7) chính là quyền chọn (option) cho phép đổi công nghệ từng phần sau này.

---

*Quay lại [Mục lục](/series/nodejs-golang/00-introduciton/)*
