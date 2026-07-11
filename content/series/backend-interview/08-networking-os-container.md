+++
title = "Bài 8 — Networking, Linux & Kubernetes"
date = "2026-07-03T15:00:00+07:00"
draft = false
tags = ["backend", "interview"]
series = ["Backend Interview"]
+++

---

## Câu 1 — [Fundamental → Senior] TCP hoạt động thế nào? TIME_WAIT là gì và tại sao nó làm cạn kiệt kết nối?

### 1. Câu hỏi
"Trình bày TCP 3-way handshake và connection lifecycle. TIME_WAIT là gì, tại sao tồn tại, và tại sao nó gây sự cố 'cannot assign requested address' trên production?"

### 2. Interviewer muốn kiểm tra điều gì?
- Kiến thức mạng nền tảng — nền của mọi thứ backend, và là nơi phân biệt người hiểu sâu.
- Nối được lý thuyết (TCP states) với sự cố thật (port exhaustion, connection pool).
- Hiểu tại sao connection reuse (keep-alive, pooling) quan trọng.

### 3. Câu trả lời ngắn gọn (30 giây)
"TCP thiết lập kết nối qua 3-way handshake (SYN → SYN-ACK → ACK) để hai bên đồng bộ sequence number và xác nhận cả hai gửi/nhận được. Đóng kết nối qua 4 bước (FIN/ACK mỗi chiều). Bên **chủ động đóng** rơi vào **TIME_WAIT** ~2×MSL (thường 60s trên Linux) trước khi giải phóng hoàn toàn — để nuốt các packet lạc đường của kết nối cũ và đảm bảo bên kia nhận được ACK cuối. Sự cố: một service mở nhiều kết nối ngắn ra ngoài (mỗi request một connection tới upstream) và chủ động đóng → hàng chục nghìn socket kẹt TIME_WAIT chiếm hết ephemeral port (~28k mặc định) → 'cannot assign requested address', không mở được kết nối mới dù CPU/RAM rảnh. Fix gốc: **tái sử dụng kết nối** (keep-alive, connection pool) để đừng tạo-hủy liên tục."

### 4. Câu trả lời Senior Level (3–5 phút)
**Handshake và vì sao 3 bước:** cần 3 vì mỗi chiều phải đồng bộ sequence number (ISN) và được xác nhận. SYN (client gửi ISN_c) → SYN-ACK (server ACK ISN_c, gửi ISN_s) → ACK (client ACK ISN_s). 2 bước không đủ (server không biết client nhận được ISN_s chưa); 4 bước thừa (gộp được SYN+ACK của server). Handshake này là lý do latency kết nối mới = 1 RTT trước khi gửi được data (TLS thêm 1–2 RTT nữa) → **connection reuse tiết kiệm RTT là tiết kiệm thật**.

**Đóng kết nối & TIME_WAIT:** 4-way (FIN → ACK → FIN → ACK) vì mỗi chiều đóng độc lập (full-duplex). Bên chủ động đóng vào **TIME_WAIT** giữ (ip,port) cặp trong 2×MSL. Hai lý do: (1) nếu ACK cuối bị mất, bên kia gửi lại FIN, bên này còn ở TIME_WAIT để ACK lại — không thì bên kia kẹt LAST_ACK; (2) nuốt packet trễ của kết nối cũ để chúng không lẫn vào kết nối mới trùng (ip,port).

**Sự cố port exhaustion:** client mở kết nối ra ngoài dùng ephemeral port (dải ~32768–60999, khoảng 28k). Nếu service tạo kết nối ngắn tới upstream cho **mỗi** request rồi chủ động đóng → mỗi kết nối để lại một TIME_WAIT giữ port 60s. Vượt ~28k kết nối/60s (≈470/s) là bắt đầu cạn port. Triệu chứng: `EADDRNOTAVAIL`, connect timeout, trong khi tài nguyên khác rảnh — cực dễ chẩn đoán nhầm.

**Fix theo thứ tự đúng:**
1. **Gốc: tái sử dụng kết nối** — HTTP keep-alive, connection pool (DB, HTTP client). Đây là 90% lời giải: đừng tạo-hủy, giữ và tái dùng.
2. Điều chỉnh OS nếu vẫn cần: `net.ipv4.ip_local_port_range` mở rộng, `tcp_tw_reuse=1` (cho kết nối outbound, an toàn), tăng backlog. **Không** dùng `tcp_tw_recycle` (đã bị xóa khỏi kernel vì hỏng với NAT).
3. Kiến trúc: giảm số kết nối (pooling ở tầng proxy, HTTP/2 multiplexing một connection nhiều request).

### 5. Giải thích bản chất
TCP là nỗ lực xây **kênh tin cậy, có thứ tự trên một mạng không tin cậy, không thứ tự** (IP có thể mất, trùng, đảo thứ tự packet). Mọi cơ chế của TCP suy ra từ mục tiêu đó: sequence number (phát hiện mất/đảo thứ tự), ACK (xác nhận nhận được), retransmit (bù mất mát), và TIME_WAIT chính là cái giá của việc **đảm bảo kết nối cũ chết hẳn trước khi tái dùng định danh của nó** — nếu không, một packet lạc của kết nối cũ có thể bị kết nối mới (cùng ip:port) hiểu nhầm là dữ liệu của mình, phá vỡ tính đúng đắn. Nói cách khác, TIME_WAIT không phải bug hay sự bất tiện — nó là điều kiện cần để đảm bảo tin cậy trên mạng chia sẻ. Hiểu điều này thì thấy "tắt TIME_WAIT cho nhanh" là đánh đổi tính đúng đắn, và lời giải đúng luôn là **dùng ít kết nối hơn**, không phải làm kết nối chết nhanh hơn một cách nguy hiểm.

### 6. Trade-off
- **Kết nối ngắn (mở/đóng mỗi request):** đơn giản, không quản pool ↔ tốn 1 RTT handshake mỗi lần + TIME_WAIT tích lũy → port exhaustion.
- **Connection pool/keep-alive:** tiết kiệm RTT, tránh TIME_WAIT ↔ phải quản pool size, kết nối chết trong pool (stale — cần health check/max lifetime), giữ tài nguyên cả khi idle.
- **`tcp_tw_reuse`:** giảm áp lực port outbound ↔ chỉ an toàn cho outbound, cần hiểu rõ; không phải nút "sửa mọi thứ".
- **HTTP/2 multiplexing:** một connection nhiều stream, ít socket ↔ head-of-line blocking ở tầng TCP (một packet mất chặn mọi stream — HTTP/3/QUIC giải quyết).

### 7. Ví dụ Production
Sự cố kinh điển tôi xử lý nhiều lần: service Node/Go gọi một API nội bộ, dùng HTTP client mặc định **không bật keep-alive** (hoặc pool size = 0). Traffic bình thường ổn; giờ cao điểm ~600 req/s → mỗi request một kết nối mới, chủ động đóng → TIME_WAIT chất đống → sau ~1 phút port cạn → `connect: cannot assign requested address` hàng loạt, service "chết" trong khi CPU 15%. Chẩn đoán: `ss -s` thấy hàng chục nghìn socket TIME_WAIT; `netstat` xác nhận. Fix một dòng: bật keep-alive + đặt pool size hợp lý ở HTTP client → số socket từ 30k về vài trăm, sự cố biến mất. Bài học: **sự cố mạng kịch tính nhất thường có lời giải một dòng config** — nếu bạn hiểu TCP lifecycle. Người không hiểu sẽ thêm máy (vô ích, mỗi máy vẫn cạn port riêng).

### 8. Những câu trả lời chưa đủ tốt
- "TCP là giao thức tin cậy có 3-way handshake." → Định nghĩa sách giáo khoa. Interviewer muốn: tại sao 3, TIME_WAIT làm gì, và nối với sự cố thật.
- "Port exhaustion thì tăng port range." → Vá triệu chứng. Gốc là tạo quá nhiều kết nối; fix đúng là connection reuse.

### 9. Sai lầm phổ biến của ứng viên
- Không biết TIME_WAIT ở phía **chủ động đóng**, và không nối được với port exhaustion.
- Đề xuất `tcp_tw_recycle` (đã bị xóa, hỏng với NAT/load balancer).
- Không phân biệt TIME_WAIT (bên đóng) và CLOSE_WAIT (bên nhận FIN chưa đóng — dấu hiệu **bug ứng dụng** quên close, khác hẳn).
- Không biết connection pool giải quyết gốc; thêm máy để "chia tải" mà mỗi máy vẫn cạn port.
- Nhầm UDP và TCP về đảm bảo (UDP không handshake, không thứ tự, không retransmit).

### 10. Follow-up Questions
- CLOSE_WAIT nhiều nghĩa là gì? (ứng dụng quên close socket — file descriptor leak, khác TIME_WAIT hoàn toàn.)
- TCP flow control (receive window) vs congestion control (cwnd) — khác gì? Nối với backpressure (chương Node stream)?
- Head-of-line blocking ở TCP là gì? HTTP/2 giải quyết ở tầng nào, còn sót ở đâu? HTTP/3 (QUIC over UDP) giải nốt thế nào?
- Nagle's algorithm và delayed ACK tương tác gây latency thế nào? Khi nào tắt `TCP_NODELAY`?
- Nếu thấy latency p99 tăng nhưng throughput bình thường, nghi ngờ gì ở tầng TCP? (retransmit, cwnd reset, buffer bloat.)

### 11. Liên hệ với Production
Mọi service gọi service, gọi DB, gọi API ngoài đều đụng bài toán connection lifecycle; các hệ lớn dùng connection pool ở mọi tầng + HTTP/2/gRPC để giảm số kết nối. Vấn đề nghiêm trọng khi: fan-out cao (một service gọi nhiều upstream), traffic burst, hoặc sidecar/proxy nhân số kết nối. Dấu hiệu cần hành động: socket TIME_WAIT/CLOSE_WAIT cao trong `ss -s`, lỗi `EADDRNOTAVAIL`/`EMFILE`, latency tăng do handshake lặp lại, và p99 nhạy với việc bật/tắt keep-alive.

---

## Câu 2 — [Intermediate → Senior] HTTP/1.1 vs HTTP/2 vs HTTP/3, và TLS handshake

### 1. Câu hỏi
"HTTP/1.1, HTTP/2, HTTP/3 khác nhau ở đâu và mỗi phiên bản giải quyết vấn đề gì của phiên bản trước? TLS handshake tốn bao nhiêu round-trip và tối ưu thế nào?"

### 2. Interviewer muốn kiểm tra điều gì?
- Hiểu sự tiến hóa của protocol theo hướng giải quyết bottleneck cụ thể — tư duy "mỗi phiên bản sửa một vấn đề".
- Head-of-line blocking ở các tầng khác nhau — khái niệm lặp lại.
- Nhận thức chi phí TLS và cách giảm latency.

### 3. Câu trả lời ngắn gọn (30 giây)
"HTTP/1.1: một kết nối xử lý tuần tự từng request (head-of-line blocking ở tầng HTTP), browser phải mở nhiều kết nối song song để bù. HTTP/2: **multiplexing** nhiều stream trên một kết nối TCP + nén header (HPACK) + server push → giải HOL blocking ở tầng HTTP, nhưng vẫn dính HOL ở tầng **TCP** (một packet mất chặn mọi stream vì TCP đảm bảo thứ tự byte). HTTP/3: chạy trên **QUIC (over UDP)** — mỗi stream độc lập ở tầng transport nên một packet mất chỉ chặn stream của nó, cộng handshake nhanh hơn (gộp TLS vào transport, 0-RTT khi reconnect). TLS 1.3 giảm handshake còn 1 RTT (từ 2 của TLS 1.2), và 0-RTT cho kết nối lại. Tối ưu: session resumption, keep-alive, TLS 1.3, CDN gần user."

### 4. Câu trả lời Senior Level (3–5 phút)
**HTTP/1.1 và vấn đề:** mỗi kết nối xử lý một request tại một thời điểm; request chậm chặn các request sau trên cùng kết nối (HOL blocking tầng HTTP). Pipelining (gửi nhiều request không chờ response) thất bại thực tế vì response vẫn phải theo thứ tự. Browser bù bằng 6 kết nối song song/domain → domain sharding hacky. Header lặp lại (cookie, user-agent) mỗi request tốn băng thông.

**HTTP/2 giải:** một kết nối TCP, nhiều **stream** đan xen (multiplexing) — request/response chia thành frame, đan xen, ráp lại theo stream ID. Hết HOL tầng HTTP. HPACK nén header (bảng động hai bên). Server push (đẩy resource trước khi client hỏi — thực tế ít dùng, đã deprecate). **Nhưng:** vẫn một kết nối TCP → một packet TCP mất, kernel giữ lại **mọi** stream chờ retransmit (TCP đảm bảo thứ tự byte toàn cục) → HOL blocking chuyển xuống tầng TCP, lộ rõ trên mạng mất gói (mobile).

**HTTP/3 + QUIC giải nốt:** QUIC xây trên UDP, tự cài đặt reliability + thứ tự **per-stream** ở userspace → packet mất chỉ chặn stream chứa nó, các stream khác chạy tiếp. Thêm: handshake gộp transport + TLS 1.3 (1 RTT, hoặc 0-RTT reconnect), connection migration (đổi mạng WiFi↔4G không rớt kết nối vì định danh bằng connection ID không phải ip:port). Giá: UDP đôi khi bị firewall chặn/bóp, CPU cao hơn (userspace), triển khai mới hơn.

**TLS handshake:**
- TLS 1.2: 2 RTT (thương lượng cipher, trao key, verify cert) → cộng với TCP handshake là 3 RTT trước byte data đầu tiên — với user xa server, mỗi RTT 200ms thì 600ms chỉ để bắt tay.
- TLS 1.3: 1 RTT (gộp bước, bỏ cipher yếu), **0-RTT** cho kết nối lại (gửi data ngay với session ticket cũ — nhưng có rủi ro replay attack, chỉ cho request idempotent).
- Tối ưu: session resumption/ticket, OCSP stapling, TLS termination ở CDN/LB gần user (RTT ngắn), keep-alive để amortize handshake qua nhiều request.

### 5. Giải thích bản chất
Câu chuyện HTTP/1→2→3 là bài học kinh điển về **bottleneck di chuyển xuống tầng thấp hơn khi bạn sửa tầng trên**. Sửa HOL ở tầng HTTP (H2 multiplexing) → lộ HOL ở tầng TCP → phải thay cả transport (H3/QUIC). Đây là hiện tượng phổ quát trong tối ưu hệ thống: **gỡ nút cổ chai này thì nút cổ chai kế tiếp lộ diện** — không có "giải pháp cuối cùng", chỉ có việc đẩy giới hạn xuống chỗ khó chạm hơn. QUIC phải bỏ TCP (thứ đã 40 năm, tối ưu tận xương trong kernel) và xây lại reliability trên UDP ở userspace — một đánh đổi khổng lồ chỉ để thoát khỏi ràng buộc "TCP đảm bảo thứ tự byte toàn cục" mà H2 không thể phá. Về TLS, bản chất là bài toán **thiết lập bí mật chung qua kênh công khai với một bên chưa từng gặp** — cần trao đổi khóa (Diffie-Hellman), xác thực danh tính (certificate/PKI), và mọi RTT là latency người dùng cảm nhận được; lịch sử TLS là lịch sử cắt giảm số RTT mà không hy sinh an toàn.

### 6. Trade-off
- **HTTP/2:** ít kết nối, nén header, multiplexing ↔ HOL tầng TCP trên mạng mất gói, một kết nối chết mất mọi stream, load balancing khó hơn (kết nối dài).
- **HTTP/3:** hết HOL, reconnect nhanh, đổi mạng mượt ↔ UDP bị chặn/bóp ở một số nơi, CPU cao hơn, tooling/debug non trẻ hơn.
- **TLS 1.3 0-RTT:** latency thấp nhất ↔ replay attack cho non-idempotent request — phải giới hạn cẩn thận.
- **TLS termination ở edge (CDN):** handshake gần user, nhanh ↔ traffic từ edge tới origin cần bảo mật riêng, CDN thấy plaintext.

### 7. Ví dụ Production
Tối ưu thật tôi từng làm: app mobile cho thị trường có user ở xa data center (RTT ~250ms), mạng 3G/4G chập chờn. Ban đầu HTTP/1.1 + TLS 1.2: mỗi màn hình mở nhiều kết nối, mỗi kết nối 3 RTT bắt tay (~750ms) trước byte đầu, cộng HOL khi mất gói → cảm giác "lag" kinh khủng. Chuyển sang HTTP/2 + TLS 1.3 + CDN edge gần user: một kết nối, 1 RTT handshake, edge termination cắt RTT → time-to-first-byte giảm ~60%. Sau đó bật HTTP/3 cho phần user mạng kém nhất: HOL blocking do mất gói giảm rõ, connection migration giúp không rớt khi chuyển WiFi↔4G. Bài học: với user ở xa và mạng kém, **protocol và vị trí edge quan trọng hơn tối ưu code** — không dòng code nào mua được tốc độ ánh sáng.

### 8. Những câu trả lời chưa đủ tốt
- "HTTP/2 nhanh hơn vì multiplexing." → Đúng một phần. Còn HOL ở tầng TCP thì sao? Tại sao vẫn cần HTTP/3?
- "HTTPS an toàn hơn HTTP." → Không sai nhưng không trả lời câu hỏi về cơ chế/chi phí handshake và cách tối ưu.

### 9. Sai lầm phổ biến của ứng viên
- Nghĩ HTTP/2 giải quyết hoàn toàn HOL blocking (còn ở tầng TCP).
- Không biết HTTP/3 chạy trên UDP/QUIC và tại sao phải rời TCP.
- Không biết TLS 1.3 giảm RTT, hoặc rủi ro replay của 0-RTT.
- Nhầm "HTTP/2 nhiều kết nối" — thực ra ngược lại, nó dùng **ít** kết nối hơn nhờ multiplexing.
- Không nối được số RTT handshake với latency người dùng thực tế.

### 10. Follow-up Questions
- gRPC dùng HTTP/2 — load balancing gRPC khó ở đâu vì kết nối dài? (L7 LB hoặc client-side LB — nối chương API.)
- 0-RTT replay attack cụ thể nguy hiểm với request nào? Phòng thế nào?
- mTLS thêm gì vào handshake so với TLS thường? (client cũng trình cert — thêm verify.)
- Certificate: PKI hoạt động thế nào, CA tin cậy ra sao, cert pinning giải quyết và gây rủi ro gì?
- Nếu đo thấy TTFB cao chủ yếu do handshake, các đòn bẩy theo thứ tự hiệu quả? (keep-alive, TLS 1.3, session resumption, edge termination.)

### 11. Liên hệ với Production
CDN lớn (Cloudflare, Fastly, Akamai) đẩy HTTP/3 + TLS 1.3 + edge termination làm chuẩn; các app toàn cầu dựa vào edge để cắt RTT. Vấn đề nghiêm trọng khi: user ở xa origin, mạng di động kém, hoặc nhiều request nhỏ per màn hình. Dấu hiệu cần tối ưu: TTFB cao tương quan với khoảng cách địa lý, latency nhạy với chất lượng mạng (mobile), số kết nối TCP cao do HTTP/1.1, và handshake chiếm phần lớn thời gian request ngắn.

---

## Câu 3 — [Senior → Staff] Linux cho backend: file descriptor, epoll, OOM killer, và troubleshooting

### 1. Câu hỏi
"Service của anh chạy chậm/lỗi trên production. Anh SSH vào máy và làm gì đầu tiên? Giải thích file descriptor limit, epoll, và OOM killer — chúng gây sự cố backend thế nào?"

### 2. Interviewer muốn kiểm tra điều gì?
- Kỹ năng vận hành thực chiến — phân biệt người đã trực production và người chỉ code.
- Hiểu các cơ chế OS nền tảng dưới mọi runtime (epoll dưới Node/Go, fd limit, cgroup memory).
- Có phương pháp troubleshooting hệ thống, không đoán mò.

### 3. Câu trả lời ngắn gọn (30 giây)
"Đầu tiên tôi không sửa gì — tôi **quan sát**: `top/htop` (CPU, mem, load), `dmesg | tail` (OOM kill, lỗi kernel), disk `df -h` + `iostat`, network `ss -s` (socket states), và log ứng dụng. Ba thủ phạm OS kinh điển: (1) **file descriptor limit** — mỗi socket/file là một fd, vượt `ulimit -n` (mặc định thấp) là `Too many open files`, service ngừng nhận kết nối; (2) **epoll** — cơ chế kernel để một thread theo dõi hàng nghìn fd không blocking (nền của event loop Node và netpoller Go); (3) **OOM killer** — kernel giết process ngốn RAM khi hết bộ nhớ, thấy trong dmesg, thường là nguyên nhân service 'tự chết' không log. Phương pháp: quan sát → khoanh vùng tài nguyên nghẽn (CPU/mem/disk/net/fd) → đào sâu bằng công cụ chuyên (strace, tcpdump, pprof) → sửa → xác nhận."

### 4. Câu trả lời Senior Level (3–5 phút)
**Quy trình troubleshoot (USE method — Utilization, Saturation, Errors cho từng tài nguyên):**
1. `uptime`/`top`: load average vs số core (load > core = CPU saturated hoặc chờ I/O); %CPU us vs sy vs wa (wa cao = chờ disk).
2. `free -h`, `dmesg | grep -i oom`: RAM còn không, có bị OOM kill không.
3. `df -h`, `iostat -x 1`: đĩa đầy? I/O bão hòa (%util ~100)?
4. `ss -s`, `ss -tan | awk ...`: socket states (TIME_WAIT/CLOSE_WAIT nhiều — chương trước), số kết nối.
5. Ứng dụng: log, và profiler (pprof cho Go, clinic cho Node — các chương trước).

**File descriptor:** Unix "mọi thứ là file" — socket, pipe, file thật đều là fd. Mỗi process có limit (`ulimit -n`, soft/hard; systemd `LimitNOFILE`). Server nhiều kết nối (WebSocket 100k, DB pool lớn) dễ đụng trần → `EMFILE: too many open files` → không accept kết nối mới, service "treo" khó hiểu. Fix: tăng limit đúng chỗ (systemd unit, không chỉ shell), và **kiểm tra fd leak** (`ls /proc/<pid>/fd | wc -l` tăng đơn điệu = quên close — nối CLOSE_WAIT).

**epoll:** trước epoll, `select`/`poll` phải quét toàn bộ danh sách fd mỗi lần → O(n), chết ở C10K. epoll đăng ký fd một lần, kernel chỉ trả về fd **có sự kiện** → O(số sự kiện), scale tới hàng trăm nghìn kết nối. Đây là nền của event loop Node (libuv), netpoller Go, nginx. Không cần code epoll trực tiếp, nhưng hiểu nó giải thích tại sao một thread xử lý được vạn kết nối và tại sao "blocking syscall" phá vỡ mô hình.

**OOM killer:** khi RAM (hoặc cgroup memory limit trong container) cạn, kernel chọn process "tệ nhất" (theo oom_score) giết để cứu hệ thống. Triệu chứng: service biến mất không log ứng dụng, chỉ `dmesg` ghi "Killed process". Trong container, limit là **cgroup** — process bị kill khi vượt limit container dù máy host còn RAM. Nối các chương: Go GOMEMLIMIT, Node max-old-space-size phải đặt **dưới** cgroup limit chừa phần off-heap, nếu không OOM kill thay vì GC.

### 5. Giải thích bản chất
Troubleshooting production là **khoa học thực nghiệm dưới áp lực**: hệ thống là hộp đen đang cháy, bạn chỉ có các đầu dò (metric, log, syscall trace). Nguyên tắc nền: **quan sát trước khi can thiệp** — thay đổi mù khi chưa hiểu làm nhiễu bằng chứng và có thể làm tệ hơn. USE method là cách vét cạn có hệ thống: với mỗi tài nguyên (CPU, RAM, disk, network, fd), hỏi ba câu — dùng bao nhiêu (utilization), có xếp hàng chờ không (saturation), có lỗi không (errors) — đảm bảo không bỏ sót tầng nào. Các cơ chế OS (fd, epoll, OOM, cgroup) đáng học vì chúng là **nền chung dưới mọi ngôn ngữ**: Go, Node, Java, Python đều chạy trên cùng kernel Linux, và khi runtime "hành xử lạ", nguyên nhân thường ở tầng OS mà runtime chỉ là nạn nhân. Người hiểu tầng này debug được thứ mà người chỉ biết framework bó tay.

### 6. Trade-off
- **fd limit cao:** chịu nhiều kết nối ↔ che giấu fd leak (leak vẫn tăng, chỉ chậm lộ hơn); phải kèm monitoring fd count.
- **Quan sát kỹ trước khi sửa:** chẩn đoán đúng gốc ↔ mất thời gian trong lúc đang có sự cố — cân bằng bằng runbook (biết trước nhìn gì) và mitigation nhanh (restart) song song với điều tra.
- **cgroup limit chặt:** cô lập tài nguyên, chống noisy neighbor ↔ OOM kill khi burst, cần chừa headroom cho off-heap.
- **strace/tcpdump:** thông tin sâu ↔ overhead (strace làm chậm process đáng kể — cẩn thận trên production nóng).

### 7. Ví dụ Production
Sự cố "ma" tôi từng gỡ: service Java trong K8s tự restart ngẫu nhiên vài lần/ngày, không exception, không log lỗi. Ứng dụng "khỏe" theo mọi metric app. `kubectl describe pod` → `OOMKilled`. Nhưng heap Java set 512MB, limit container 1GB — sao OOM? `dmesg`/cgroup memory cho thấy RSS thật 1.1GB: heap 512MB + metaspace + thread stack (mỗi thread 1MB × 300 thread) + native buffer (Netty direct memory) + JVM overhead = vượt 1GB. JVM heap limit **không phải** giới hạn RAM tổng của process. Fix: set container limit hợp lý gồm cả off-heap, giảm thread pool, giới hạn direct memory. Bài học lặp lại xuyên bộ tài liệu: **giới hạn heap của runtime ≠ giới hạn RAM của process ≠ cgroup limit** — nhầm ba cái này là nguồn OOM kill số một trong container.

### 8. Những câu trả lời chưa đủ tốt
- "Tôi xem log rồi restart." → Restart là mitigation, không phải chẩn đoán. Interviewer muốn phương pháp tìm gốc rễ.
- "Service chậm thì tôi thêm CPU/RAM." → Trước khi biết nghẽn ở đâu? Có thể nghẽn ở fd, ở lock, ở downstream — thêm tài nguyên vô ích.

### 9. Sai lầm phổ biến của ứng viên
- Sửa trước khi quan sát; đoán nguyên nhân không bằng chứng.
- Không biết OOM killer, tưởng service "tự crash" là bug ứng dụng.
- Nhầm heap limit của runtime với RAM process với cgroup limit.
- Không biết fd bao gồm cả socket; không nối fd leak với CLOSE_WAIT.
- Không biết công cụ cơ bản (`ss`, `dmesg`, `iostat`, `/proc/<pid>/`) — dấu hiệu chưa trực production.

### 10. Follow-up Questions
- Load average 20 trên máy 8 core nghĩa là gì? Phân biệt CPU-bound và I/O-bound load thế nào? (%wa, `vmstat`.)
- Một process ăn 100% một core trong khi 7 core rảnh — nghi ngờ gì? (single-thread hot loop, lock contention, GIL.)
- strace dùng để tìm gì? Nguy hiểm gì trên production? (thấy syscall bị treo ở đâu; overhead lớn.)
- cgroup v1 vs v2 khác gì đáng kể cho việc giới hạn memory/CPU container?
- Nếu disk I/O bão hòa, các lớp nguyên nhân? (log ghi quá nhiều, swap, DB checkpoint, đối thủ noisy neighbor.)

### 11. Liên hệ với Production
SRE/DevOps ở mọi công ty đều sống bằng bộ công cụ này; các hệ trưởng thành có runbook chuẩn hóa (nhìn gì đầu tiên) và dashboard USE/RED để khỏi phải SSH thủ công. Vấn đề nghiêm trọng khi: chạy container với limit cấu hình sai, mật độ cao (nhiều pod/máy — noisy neighbor), hoặc service giữ nhiều kết nối/file. Dấu hiệu cần hành động: OOMKilled trong pod events, `Too many open files` trong log, load average > số core kéo dài, %iowait cao, và MTTR bị chi phối bởi "không biết nhìn đâu" — thiếu runbook và observability (nối chương system design).

---

## Câu 4 — [Senior → Staff] Container & Kubernetes: cơ chế cô lập, probes, và deploy không downtime

### 1. Câu hỏi
"Container khác VM thế nào ở mức cơ chế? Trong Kubernetes, liveness vs readiness probe khác nhau ra sao và cấu hình sai gây sự cố gì? Trình bày rolling/blue-green/canary deploy và graceful shutdown."

### 2. Interviewer muốn kiểm tra điều gì?
- Hiểu container là process cô lập bằng namespace + cgroup, không phải "VM nhẹ".
- Nắm probe — nơi cấu hình sai gây outage phổ biến nhất trong K8s.
- Hiểu deploy an toàn và graceful shutdown — kỹ năng vận hành thật.

### 3. Câu trả lời ngắn gọn (30 giây)
"Container KHÔNG phải VM nhẹ — nó là **process Linux thường** được cô lập bằng hai cơ chế kernel: **namespace** (cô lập tầm nhìn — PID, network, mount, user riêng, tưởng mình là máy độc lập) và **cgroup** (giới hạn tài nguyên — CPU, RAM). Chung kernel với host, nên nhẹ và nhanh khởi động hơn VM (VM ảo hóa cả kernel/phần cứng). Trong K8s: **liveness probe** hỏi 'còn sống không, chết thì restart'; **readiness probe** hỏi 'sẵn sàng nhận traffic chưa, chưa thì rút khỏi load balancer nhưng không restart'. Nhầm hai cái này gây outage: đặt liveness quá nhạy → restart vòng lặp khi service chỉ đang bận; thiếu readiness → traffic đổ vào pod chưa warmup → lỗi hàng loạt. Deploy an toàn: rolling (thay dần), blue-green (đổi toàn bộ, rollback tức thì), canary (thử % nhỏ trước). Graceful shutdown: nhận SIGTERM → ngừng nhận request mới → xử lý nốt request đang bay → thoát, trong grace period."

### 4. Câu trả lời Senior Level (3–5 phút)
**Container internals:** khác biệt cốt lõi với VM — VM có hypervisor ảo hóa phần cứng, mỗi VM chạy kernel riêng (cô lập mạnh, nặng, boot chậm). Container dùng **chung kernel host**, cô lập bằng: **namespaces** (PID namespace → process trong container chỉ thấy process của nó; network namespace → network stack riêng; mount, UTS, user namespace...), **cgroups** (giới hạn + đo CPU, memory, I/O). Hệ quả bảo mật: cô lập container yếu hơn VM (chung kernel → kernel exploit thoát container được) — vì thế workload không tin cậy cần thêm lớp (gVisor, Kata, VM thật). Hệ quả vận hành: các chương trước — cgroup CPU limit gây CFS throttling (Go GOMAXPROCS), cgroup memory gây OOM kill.

**Probes (nguồn outage K8s số một):**
- **Liveness:** thất bại → kubelet **giết và restart** container. Dùng để thoát trạng thái kẹt không tự hồi (deadlock). Đặt sai cực nguy hiểm: nếu liveness gọi một endpoint phụ thuộc DB, và DB chậm → mọi pod fail liveness → restart đồng loạt → càng tải → **cascading restart** giữa lúc cần ổn định nhất.
- **Readiness:** thất bại → kubelet **rút pod khỏi Service endpoints** (ngừng gửi traffic) nhưng **không restart**. Dùng cho: đang warmup, đang load cache, downstream tạm không sẵn sàng. Thiếu readiness → traffic vào pod chưa sẵn sàng → 502/lỗi.
- **Startup probe:** cho app khởi động chậm — hoãn liveness tới khi startup xong, tránh liveness giết app đang boot.
- Nguyên tắc: liveness phải **nông và độc lập** (chỉ check process còn đáp ứng, không check dependency); readiness có thể check dependency (rút khỏi LB khi downstream chết là đúng).

**Deploy strategies:**
- **Rolling update:** thay pod cũ bằng mới từ từ (maxSurge/maxUnavailable) — không downtime, không cần gấp đôi tài nguyên, nhưng hai version chạy song song (cần backward compatibility DB/API) và rollback chậm.
- **Blue-green:** dựng full môi trường mới (green) song song với cũ (blue), test, rồi chuyển toàn bộ traffic — rollback tức thì (đổi lại về blue), nhưng tốn gấp đôi tài nguyên.
- **Canary:** đưa version mới cho % nhỏ traffic (1%→10%→50%→100%), theo dõi metric/lỗi, tự động rollback nếu xấu — an toàn nhất cho thay đổi rủi ro, nhưng cần hạ tầng đo lường + routing tốt.

**Graceful shutdown (bắt buộc cho zero-downtime):** khi scale down/deploy, K8s gửi **SIGTERM** → app phải: (1) fail readiness ngay (rút khỏi LB); (2) ngừng nhận request mới; (3) xử lý nốt request đang bay + đóng kết nối/flush; (4) thoát trước khi hết `terminationGracePeriodSeconds` (mặc định 30s), không thì bị SIGKILL. Bỏ qua bước này → mỗi lần deploy cắt ngang request đang xử lý → lỗi 5xx cho user (nối chương Node/Go — request đang bay bị giết).

### 5. Giải thích bản chất
Container là hiện thân của một insight: **cô lập không nhất thiết cần ảo hóa**. VM giải bài toán cô lập bằng cách giả lập cả một máy tính (tốn kém, tổng quát). Container nhận ra: nếu chỉ cần process "tưởng mình chạy một mình", chỉ cần **nói dối process về những gì nó nhìn thấy** (namespace) và **giới hạn những gì nó lấy được** (cgroup) — không cần ảo hóa phần cứng. Đây là đánh đổi cô lập-lấy-hiệu-năng có chủ đích: nhẹ hơn nhiều, nhưng chung kernel nên ranh giới bảo mật mỏng hơn. Về probes, bản chất là **health check phải phân biệt hai câu hỏi khác nhau về ngữ nghĩa**: "có nên giết nó không" (liveness) vs "có nên gửi việc cho nó không" (readiness) — trộn hai câu này là trộn hai hành động có hậu quả trái ngược (restart vs rút traffic), và đó là lý do cấu hình sai gây outage. Deploy strategies đều là các điểm khác nhau trên trục **an toàn ↔ chi phí/tốc độ**: cùng một mục tiêu (đổi version không hại user), khác nhau ở mức tài nguyên và độ tinh vi routing bạn sẵn sàng trả.

### 6. Trade-off
- **Container vs VM:** nhẹ, nhanh, mật độ cao ↔ cô lập bảo mật yếu hơn (chung kernel).
- **Liveness nhạy:** phát hiện kẹt nhanh ↔ restart nhầm khi chỉ bận/chậm → cascading restart; liveness lười thì ngược lại.
- **Rolling:** rẻ, mượt ↔ hai version song song (cần tương thích), rollback chậm.
- **Blue-green:** rollback tức thì, test trước ↔ gấp đôi tài nguyên, chuyển DB/state phức tạp.
- **Canary:** an toàn nhất cho rủi ro cao ↔ cần observability + routing tinh vi, deploy lâu hơn.

### 7. Ví dụ Production
Outage kinh điển do probe: team đặt **liveness probe gọi `/health` mà endpoint đó check cả kết nối DB**. Một hôm DB có sự cố nhẹ (chậm 5s). Mọi pod fail liveness → K8s restart **toàn bộ** pod cùng lúc → pod mới boot cũng không connect được DB chậm → fail liveness → restart lại → **crash loop toàn service** biến sự cố DB nhẹ thành outage toàn phần, và cản trở luôn việc DB tự hồi (pod restart dồn dập tạo connection storm). Fix: liveness chỉ check process (`/livez` trả 200 nếu event loop còn chạy), readiness mới check DB (rút khỏi LB khi DB chậm, không restart). Bài học tôi luôn nhấn: **liveness probe không bao giờ được phụ thuộc dependency bên ngoài** — nếu không, một sự cố nhỏ ở dependency chung được khuếch đại thành restart storm toàn hệ thống. Đây là ví dụ hoàn hảo của cascading failure ở tầng orchestration.

### 8. Những câu trả lời chưa đủ tốt
- "Container là VM nhẹ." → Sai bản chất. Container là process + namespace + cgroup, chung kernel — khác biệt này quyết định cả hiệu năng lẫn bảo mật.
- "Liveness và readiness đều là health check." → Đúng nhưng thiếu điểm cốt tử: hậu quả khác nhau (restart vs rút traffic), và nhầm gây outage.

### 9. Sai lầm phổ biến của ứng viên
- Gọi container là "máy ảo nhẹ" — không hiểu namespace/cgroup.
- Liveness probe check dependency ngoài → cascading restart.
- Thiếu readiness → traffic vào pod chưa sẵn sàng.
- Không implement graceful shutdown (SIGTERM handler) → mỗi deploy = lỗi 5xx.
- Không biết cgroup limit gây OOM kill/CPU throttling (nối chương Go/Node/Linux).
- Nghĩ K8s tự động cho HA/zero-downtime — thực ra cần cấu hình probe + PDB + graceful shutdown + backward compat đúng.

### 10. Follow-up Questions
- PodDisruptionBudget để làm gì? Chuyện gì khi drain node mà không có PDB?
- Rolling update với DB migration: xử lý thế nào khi hai version schema chạy song song? (expand-contract / backward-compatible migration.)
- Graceful shutdown với kết nối dài (WebSocket) — drain thế nào? (nối chương WebSocket.)
- HPA (autoscale) dựa CPU có vấn đề gì? Khi nào cần custom metric (queue depth, RPS)?
- Container thoát (escape) — các lớp tăng cô lập cho workload không tin cậy? (rootless, seccomp, gVisor, Kata.)

### 11. Liên hệ với Production
K8s là chuẩn de facto cho orchestration; các công ty chạy hàng nghìn pod dựa vào probe + deploy strategy + graceful shutdown để đạt zero-downtime. Vấn đề nghiêm trọng khi: deploy tần suất cao (mỗi lỗi deploy ảnh hưởng user), workload stateful (cần thứ tự shutdown), hoặc mật độ pod cao (resource limit sai gây throttling/OOM lan rộng). Dấu hiệu cần rà soát: crash loop trong pod events, lỗi 5xx tăng mỗi lần deploy (thiếu graceful shutdown/readiness), liveness probe check dependency, và không có canary/rollback tự động cho thay đổi rủi ro — mỗi deploy thành một canh bạc thay vì một thao tác thường quy.
