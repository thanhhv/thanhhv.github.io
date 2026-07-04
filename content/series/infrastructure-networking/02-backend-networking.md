+++
title = "Bài 2 — Backend Networking"
date = "2026-02-01T08:00:00+07:00"
draft = false
tags = ["backend", "infrastructure", "networking"]
series = ["Infrastructure & Networking"]
+++

# Module 02 — Backend Networking

> Từ "mạng hoạt động thế nào" sang "hệ thống backend nói chuyện với nhau thế nào cho đáng tin cậy ở quy mô lớn".

---

## 1. Connection Lifecycle, Keep-Alive và Connection Pooling

### Problem Statement
Mỗi connection mới có chi phí cố định: TCP handshake (1 RTT) + TLS handshake (1–2 RTT) + slow start (nhiều RTT để đạt throughput). Với RTT 50ms, một HTTPS connection mới tốn ~150–250ms **trước khi gửi byte đầu tiên**. Service gọi nhau 1000 req/s mà mỗi request mở connection mới thì: latency cộng thêm hàng trăm ms, CPU đốt vào handshake, ephemeral port cạn (TIME_WAIT), file descriptor cạn.

### Giải pháp và cách hoạt động
**Keep-alive**: giữ connection sau khi xong request để tái sử dụng. **Connection pool**: quản lý một tập connection "ấm" (đã handshake, cwnd đã mở) và cho request mượn.

Tham số pool và hệ quả của từng cái:

| Tham số | Quá nhỏ | Quá lớn |
|---|---|---|
| max pool size | Request xếp hàng chờ connection → latency, timeout | Đè bẹp backend (DB có max_connections!), tốn memory |
| idle timeout | Tạo/hủy liên tục, mất lợi ích pool | Giữ connection chết (bị NAT/LB cắt im lặng) |
| max lifetime | — | Không bao giờ rebalance sau khi backend scale/failover |
| validation | Dùng phải connection chết → lỗi lẻ tẻ | Mỗi lần mượn tốn 1 round-trip |

**Ba quy tắc sống còn**:
1. **Pool idle timeout < idle timeout của mọi thiết bị trung gian** (NAT gateway 350s, AWS NLB 350s, Azure LB 4 phút, nhiều firewall 30–60s). Vi phạm quy tắc này = lỗi "connection reset" ngẫu nhiên, đặc biệt sau giờ thấp điểm — connection nằm im bị cắt, request đầu tiên buổi sáng fail.
2. **Tổng pool của mọi instance ≤ giới hạn backend**. 50 pod × pool 20 = 1000 connection vào Postgres `max_connections=100` → sập. Đây là lý do tồn tại PgBouncer/RDS Proxy.
3. **Set max lifetime hữu hạn** (vd 5–15 phút) để connection phân phối lại sau khi backend thay đổi (failover DB, scale thêm node sau LB L4).

### Troubleshooting
```bash
ss -tnp | grep <port> | wc -l        # đếm connection thực tế tới backend
ss -tn state time-wait | wc -l       # TIME_WAIT nhiều = thiếu keep-alive
# App metrics cần có: pool active/idle/waiting, wait duration
```
Dấu hiệu: latency p99 cao nhưng backend nhàn rỗi → thường là **chờ mượn connection trong pool** (pool exhausted). Metric "pool wait time" là thứ đầu tiên phải nhìn.

---

## 2. Reverse Proxy và Load Balancer

### Problem Statement
Một máy không phục vụ nổi toàn bộ traffic (giới hạn vật lý), và một máy sẽ hỏng (chắc chắn). Cần: phân phối tải, che giấu topology backend, điểm tập trung cho TLS/routing/rate-limit, và loại bỏ máy hỏng khỏi rotation. Không có LB thì scale = "mua máy to hơn" (trần cứng, đắt theo cấp số) và availability = availability của 1 máy.

### L4 vs L7 — trade-off trung tâm

| | L4 (NLB, IPVS, HAProxy TCP) | L7 (ALB, Nginx, Envoy) |
|---|---|---|
| Nhìn thấy | IP + port | HTTP path, header, cookie, gRPC method |
| Hiệu năng | Rất cao, latency ~µs, pass-through | Terminate + parse → thêm ~ms, tốn CPU |
| Routing | Chỉ theo connection | Theo request → canary, path-based, retry |
| Vấn đề | HTTP/2: 1 conn → 1 backend → lệch tải | Chính nó phải scale, giữ 2× connection |
| Bảo toàn source IP | Có thể (DSR/proxy protocol) | Qua header `X-Forwarded-For` |

Quy tắc chọn: **L4 cho throughput thuần và giao thức không phải HTTP; L7 khi cần routing thông minh, retry, canary — tức là đa số API backend.** Kiến trúc phổ biến: NLB (L4, chịu tải, static IP) → Envoy/Nginx fleet (L7) → services.

### Health check — nơi LB gây sự cố thay vì ngăn sự cố
- Health check quá nhạy (1 lần fail = out) + sự cố thoáng qua → LB loại **tất cả** backend cùng lúc → outage toàn phần do chính LB. Nhiều LB có chế độ "fail open" (nếu tất cả unhealthy thì gửi cho tất cả) — phải biết LB của bạn hành xử thế nào.
- Health check endpoint "sâu" (check cả DB) → DB chậm 1 nhịp = toàn bộ fleet unhealthy = biến sự cố nhỏ thành sự cố lớn. **Best practice: liveness check nông (process sống), readiness check sâu vừa đủ (phụ thuộc critical cục bộ), không bao giờ check phụ thuộc dùng chung trong health check dùng để loại máy.**

### Thuật toán phân phối
Round-robin (đơn giản, tệ khi request không đều), **least-connections** (tốt cho request dài ngắn lẫn lộn), consistent hashing (sticky theo key — cache locality, đổi lại hot key = hot backend), **least-request + power-of-two-choices** (Envoy default — gần tối ưu với chi phí thấp). Với backend autoscale, tránh thuật toán thuần sticky vì node mới không nhận tải ("slow ramp" cần được bật).

### Anti-patterns
- Retry ở LB + retry ở client + retry ở service = **retry storm** khuếch đại 27× khi backend chậm. Retry chỉ nên tồn tại ở MỘT tầng có budget.
- Idle timeout của LB **dài hơn** của backend → LB gửi request vào connection backend vừa đóng → lỗi 502 lẻ tẻ. Quy tắc: timeout backend > timeout LB > timeout client... sai! Đúng là: **keep-alive idle timeout của backend PHẢI > của LB** (backend không được đóng trước LB).

---

## 3. CDN

### Problem Statement
Tốc độ ánh sáng là giới hạn cứng: Việt Nam ↔ US East RTT ~220ms. TLS + TCP handshake xuyên đại dương = gần 1 giây trước byte đầu tiên. Đồng thời origin không nên chịu tải cho nội dung giống nhau gửi triệu lần.

### Cách hoạt động và điểm thường hiểu sai
CDN = mạng lưới edge cache + **anycast/GeoDNS** đưa user tới edge gần nhất. Giá trị không chỉ là cache:
1. **Cache static content** (điều ai cũng biết).
2. **TCP/TLS termination gần user**: cả với nội dung dynamic không cache được, handshake xảy ra ở edge (RTT 5–20ms), edge → origin dùng connection pool ấm sẵn. Giảm 30–50% latency cho API — nhiều team bỏ qua lợi ích này.
3. **Hấp thụ DDoS** ở edge trước khi chạm origin.

### Failure cases
- **Cache stampede**: object hot hết hạn → nghìn request cùng xuyên về origin. Giải pháp: request coalescing (CDN gộp), stale-while-revalidate, TTL jitter.
- **Cache key sai**: quên vary theo header quan trọng → user A thấy dữ liệu user B (sự cố rò rỉ dữ liệu kinh điển — cache response có `Set-Cookie` hoặc cache API cá nhân hóa). Quy tắc: **mặc định không cache, opt-in từng route**, không bao giờ để CDN "cache mọi thứ 200 OK".
- Purge toàn bộ cache lúc traffic cao = tự tạo thundering herd vào origin.

---

## 4. WebSocket và gRPC

### WebSocket
**Problem**: HTTP là request-response; server không chủ động đẩy được. Polling tốn kém và trễ. WebSocket nâng cấp connection HTTP thành kênh 2 chiều bền vững.

**Cái giá của stateful connection** — đây là phần quan trọng:
- LB phải hỗ trợ connection dài, idle timeout phải rất dài hoặc có heartbeat ứng dụng (ping/pong mỗi 30s — vì middlebox sẽ cắt connection im lặng).
- **Deploy trở thành sự kiện**: restart server = rớt đồng loạt N nghìn connection = N nghìn client reconnect cùng lúc = thundering herd vào cả server mới lẫn hệ thống auth. Bắt buộc: reconnect với **jitter + exponential backoff** phía client, drain từ từ phía server.
- Scale khó hơn stateless: cần layer định tuyến message tới đúng node giữ connection (Redis pub/sub, hoặc mesh riêng).
- Cân nhắc SSE (Server-Sent Events) khi chỉ cần server→client: đơn giản hơn nhiều, chạy trên HTTP thuần, tự reconnect.

### gRPC
**Problem**: REST/JSON: serialize đắt, không có contract chặt, không streaming. gRPC = HTTP/2 + Protobuf + codegen: binary nhỏ hơn 3–10×, parse nhanh hơn ~5–10×, contract-first, 4 kiểu streaming, deadline propagation tích hợp.

**Trade-off và bẫy production**:
- **L4 LB không cân bằng được gRPC** (multiplexing trên connection dài — như đã nói ở Module 01): trong Kubernetes phải dùng L7 proxy (Envoy/Linkerd) hoặc headless service + client-side LB.
- Debug khó hơn: không `curl` trực tiếp — dùng `grpcurl`, bật reflection ở môi trường non-prod.
- Browser không nói gRPC trực tiếp → cần gRPC-web/gateway. Quy tắc thực dụng: **gRPC nội bộ service-to-service, REST/JSON cho public API**.
- Deadline propagation là killer feature bị bỏ quên: client set deadline 500ms, deadline đi theo cả chuỗi call — service cuối tự hủy việc vô ích. Không có nó, request đã timeout ở client vẫn tiếp tục đốt tài nguyên qua 5 service phía sau.

---

## 5. Service Discovery

### Problem Statement
Thế giới cũ: IP tĩnh trong file config. Thế giới autoscaling + container: instance sinh/chết liên tục, IP vô nghĩa sau vài phút. Câu hỏi "service B đang ở đâu?" phải có câu trả lời **động và đáng tin**.

### Ba mô hình
1. **DNS-based** (Kubernetes Service, Consul DNS): đơn giản, mọi client hỗ trợ sẵn. Yếu: cache TTL làm thông tin cũ, không mang metadata (health, zone, version), không có port động (trừ SRV mà ít client dùng).
2. **Client-side discovery** (Eureka, Consul API, xDS): client lấy danh sách endpoint + tự chọn (kèm health, zone-aware). Mạnh và nhanh nhất; giá: logic phức tạp trong MỌI client, mọi ngôn ngữ.
3. **Server-side / proxy-based** (LB, service mesh sidecar): client chỉ biết 1 tên, proxy lo phần còn lại. Đơn giản cho app; giá: thêm hop, thêm hạ tầng phải vận hành.

Xu hướng hội tụ: **đẩy độ phức tạp vào platform** (Kubernetes + mesh), app chỉ gọi tên DNS. Đúng cho 90% tổ chức.

### Nguyên tắc thiết kế quan trọng
Registry phải theo **lease/TTL**: instance phải liên tục tự gia hạn; chết thì tự động bị loại. Đăng ký "một lần mãi mãi" = registry đầy xác chết. Và cẩn thận chiều ngược lại: registry mất liên lạc thoáng qua với TẤT CẢ instance (network blip) → loại tất cả → outage. Eureka giải quyết bằng "self-preservation mode" (mất >15% instance đột ngột thì ngừng loại) — mọi hệ discovery tự viết đều phải có cơ chế tương tự: **thà dùng dữ liệu cũ còn hơn dùng danh sách rỗng**.

---

## 6. Timeout, Retry, Circuit Breaker — bộ ba sinh tồn của hệ phân tán

### Problem Statement
Trong hệ phân tán, lỗi nguy hiểm nhất không phải "fail nhanh" mà là **"chậm vô hạn"**: một dependency treo làm thread/connection của caller bị giữ, caller cạn tài nguyên, rồi caller CỦA caller cạn tài nguyên — **cascading failure**. Toàn bộ mục này là về việc chuyển "chậm vô hạn" thành "fail nhanh, có kiểm soát".

### Timeout
- **Mọi network call phải có timeout tường minh.** Rất nhiều thư viện mặc định infinite (Java HttpURLConnection cũ, một số Redis/DB driver) — đây là bug chờ ngày phát nổ.
- Đặt bao nhiêu? Theo dữ liệu: **timeout ≈ p99.9 latency của dependency lúc khỏe + đệm**, không phải con số tròn cảm tính. Timeout 30s cho call bình thường 50ms nghĩa là khi sự cố, bạn giữ tài nguyên lâu gấp 600× bình thường.
- **Timeout phải giảm dần theo chuỗi gọi** (deadline propagation): A(1s) → B(800ms) → C(600ms). Nếu C có timeout 5s trong khi A chỉ chờ 1s: C làm việc vô ích cho request đã chết. Đây là nguồn lãng phí tài nguyên vô hình lớn trong microservices.

### Retry
Retry chữa lỗi **thoáng qua** (packet loss, 1 pod chết) nhưng là **thuốc độc với lỗi quá tải**: backend chậm vì quá tải → client retry → tải tăng gấp bội → chậm hơn → retry nhiều hơn. Vòng xoáy này biến sự cố 1 phút thành outage 1 giờ (metastable failure).

Quy tắc retry an toàn:
1. Chỉ retry lỗi **có khả năng thoáng qua** (timeout connect, 503, reset) — không retry 4xx, không retry lỗi nghiệp vụ.
2. Chỉ retry request **idempotent** (hoặc có idempotency key). Retry một lệnh chuyển tiền không idempotent = chuyển tiền 2 lần.
3. **Exponential backoff + full jitter** (`sleep = random(0, min(cap, base·2^n))`). Không jitter → mọi client retry cùng nhịp → sóng tải đồng bộ đập vào backend theo chu kỳ.
4. **Retry budget** (vd: retry ≤ 10% tổng request — cách của Envoy/Finagle) thay vì "mỗi request retry 3 lần". Budget tự tắt retry khi hệ thống đang cháy — đúng lúc cần tắt nhất.
5. Retry ở **một tầng duy nhất**. 3 tầng × 3 lần = 27 request cho 1 request gốc.

### Circuit Breaker
Retry là quyết định per-request; circuit breaker là **trí nhớ chung**: khi dependency hỏng liên tục, ngừng gọi hẳn một thời gian.

```
CLOSED ──(error rate > ngưỡng trong cửa sổ)──▶ OPEN (fail fast, không gọi)
   ▲                                              │ (hết sleep window)
   └──(probe thành công)── HALF-OPEN ◀────────────┘
              │ (probe fail) → OPEN lại
```

Giá trị kép: (1) caller fail nhanh, trả degraded response, không cạn tài nguyên; (2) **dependency được thở** để hồi phục — không có breaker, backend vừa gượng dậy đã bị dòng retry đè chết tiếp.

**Cấu hình sai phổ biến**:
- Ngưỡng theo % lỗi mà không có volume tối thiểu → 1 lỗi trong 2 request lúc traffic thấp = mở breaker oan.
- Breaker chung cho mọi endpoint của một service → endpoint phụ hỏng làm đứt endpoint chính. Scope breaker theo **dependency + độ quan trọng**.
- Mở breaker mà không có fallback được thiết kế → đổi lỗi timeout thành lỗi ngay lập tức, user vẫn thấy lỗi. Breaker chỉ có ý nghĩa khi đi kèm **graceful degradation** (trả cache cũ, giá trị mặc định, tắt tính năng phụ).

### Network Partition — sự thật nền tảng
Mạng **sẽ** phân mảnh: switch hỏng, AZ mất liên lạc, cáp đứt. Khi partition xảy ra, mỗi phía phải chọn (định lý CAP): tiếp tục phục vụ với dữ liệu có thể cũ (AP) hay từ chối để giữ consistency (CP). Không có lựa chọn thứ ba. Hệ quả thiết kế: với mỗi dữ liệu, hãy trả lời trước "khi partition, tôi chọn gì?" — đừng để runtime chọn hộ bạn một cách ngẫu nhiên. Split-brain (2 phía cùng nghĩ mình là leader) được ngăn bằng **quorum (đa số)** — lý do mọi hệ consensus (etcd, ZooKeeper) cần số node lẻ và ≥3.

---

## Checklist tổng hợp cho một service gọi service khác

- [ ] Connection pool, idle timeout < timeout của mọi middlebox trên đường
- [ ] Timeout tường minh cho connect + request, dựa trên p99.9
- [ ] Deadline propagation xuyên chuỗi gọi
- [ ] Retry: idempotent-only, backoff + jitter, budget, một tầng duy nhất
- [ ] Circuit breaker theo dependency, có fallback thiết kế sẵn
- [ ] Metric: pool wait, in-flight requests, error rate theo loại, latency p50/p99/p999 per-dependency
- [ ] Trả lời được: "dependency này chết 10 phút thì user thấy gì?"
