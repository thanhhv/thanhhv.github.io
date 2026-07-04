+++
title = "Bài 1 — Networking Fundamentals"
date = "2026-02-01T07:00:00+07:00"
draft = false
tags = ["backend", "infrastructure", "networking"]
series = ["Infrastructure & Networking"]
+++

# Module 01 — Networking Fundamentals

> Đối tượng: Backend/Senior Engineer trở lên. Mục tiêu: hiểu mạng từ góc nhìn người thiết kế hệ thống — không phải người dùng công cụ.

---

## 1. OSI Model và TCP/IP — Tại sao phải chia tầng?

### Problem Statement
Nếu mọi ứng dụng tự lo từ tín hiệu điện đến format dữ liệu, mỗi phần mềm phải viết lại toàn bộ stack mạng, và không phần mềm nào nói chuyện được với phần mềm khác. Bài toán cốt lõi: **tách trách nhiệm (separation of concerns) để các hệ thống khác nhau tương tác được mà không cần biết chi tiết của nhau**.

### Tại sao tồn tại
Thập niên 1970, mỗi hãng (IBM SNA, DECnet) có stack mạng riêng, không tương thích. OSI (7 tầng) là nỗ lực chuẩn hóa "trên giấy"; TCP/IP (4 tầng) thắng trong thực tế vì nó được **triển khai trước, chuẩn hóa sau** — bài học quan trọng: chuẩn tốt là chuẩn mô tả thứ đang chạy được.

### Bản chất
Mỗi tầng cung cấp một **abstraction** cho tầng trên và chỉ phụ thuộc tầng dưới:

```
Application  (HTTP, DNS, gRPC)     ← dữ liệu có ý nghĩa nghiệp vụ
Transport    (TCP, UDP, QUIC)      ← end-to-end: port, reliability, ordering
Network      (IP, ICMP, routing)   ← best-effort delivery giữa các máy
Link         (Ethernet, ARP, WiFi) ← delivery trong 1 mạng vật lý
```

Khi gửi, dữ liệu bị **encapsulate** dần: HTTP payload → TCP segment → IP packet → Ethernet frame. Mỗi router chỉ nhìn đến tầng Network; chỉ 2 đầu mút nhìn tầng Transport trở lên. Đây là lý do **NAT và middlebox phá vỡ giả định end-to-end** — chúng "nhìn trộm" tầng transport, và là nguồn gốc của vô số sự cố production (idle timeout, connection reset).

### Trade-off của việc chia tầng
- **Simplicity vs Performance**: mỗi tầng copy/xử lý header riêng → overhead. Kernel bypass (DPDK), zero-copy (`sendfile`) tồn tại chính vì overhead này.
- **Abstraction leak**: tầng trên không thể "không biết" tầng dưới. TCP behavior (Nagle, slow start) rò rỉ lên latency HTTP. Kỹ sư senior debug được vì hiểu sự rò rỉ này thay vì tin vào abstraction.

### Ứng dụng thực tế khi debug
Debug theo tầng từ dưới lên là quy trình chuẩn:

```bash
ip link                  # L1/L2: interface up chưa?
ip addr; ip route        # L3: có IP, có route không?
ping <gateway>           # L3: reach được gateway?
dig example.com          # DNS resolve được không?
nc -vz host 443          # L4: TCP connect được không?
curl -v https://host     # L7: TLS + HTTP hoạt động không?
```

---

## 2. IP, Subnet, CIDR, Routing

### Problem Statement
Hàng tỷ thiết bị cần gửi dữ liệu cho nhau mà không thiết bị nào biết toàn bộ topology. Bài toán: **định danh (addressing) và tìm đường (routing) có thể scale**.

### Cách hoạt động bên trong
IP là **best-effort, connectionless**: mỗi packet độc lập, có thể mất, trùng, đến sai thứ tự. Đây là quyết định thiết kế chủ đích — đẩy độ phức tạp (reliability) ra 2 đầu mút (end-to-end principle), giữ router đơn giản để scale.

**CIDR** (`10.0.0.0/16`): prefix càng dài, mạng càng nhỏ. Router chọn route theo **longest prefix match** — route cụ thể nhất thắng. Đây là lý do bạn có thể có default route `0.0.0.0/0` đi Internet Gateway nhưng traffic nội bộ `10.0.0.0/8` vẫn đi peering: prefix dài hơn thắng.

Quy trình 1 máy gửi packet:

```
Đích cùng subnet? (so sánh IP đích với subnet mask)
 ├── Có  → ARP hỏi MAC của đích → gửi thẳng frame
 └── Không → ARP hỏi MAC của gateway → gửi frame cho gateway
```

**ARP** map IP → MAC trong mạng LAN. ARP cache stale là nguồn sự cố kinh điển khi failover IP (VRRP/keepalived): IP đã chuyển sang máy mới nhưng switch/host còn cache MAC cũ → traffic đi vào hố đen vài giây đến vài phút. Giải pháp: **gratuitous ARP** khi failover.

### Subnet trong thiết kế hạ tầng
Chia subnet không phải bài tập số học — nó là **công cụ kiểm soát blast radius và security boundary**:
- Public subnet: chỉ chứa load balancer, NAT gateway.
- Private subnet: app servers — không có route trực tiếp từ Internet.
- Isolated subnet: database — không có cả route ra Internet.

**Anti-pattern**: đặt tất cả vào một subnet `/16` public "cho tiện". Hệ quả: mọi máy đều expose, một máy bị chiếm là toàn bộ mạng nội bộ bị quét.

**Lỗi production phổ biến**: chọn VPC CIDR trùng nhau giữa các môi trường (cùng `10.0.0.0/16`) → sau này không thể peering. Quy hoạch CIDR ngay từ đầu, chừa không gian gấp 4 lần dự kiến.

### NAT
**Problem**: IPv4 chỉ có ~4,3 tỷ địa chỉ. NAT cho phép nhiều máy private dùng chung 1 IP public bằng cách rewrite (IP nguồn, port nguồn) và giữ **bảng connection tracking**.

**Trade-off cốt lõi**: NAT là **stateful** trong một thế giới IP vốn stateless. Hệ quả:
1. Connection chỉ khởi tạo được từ trong ra — bên ngoài không gọi vào được (vừa là feature bảo mật, vừa là lý do WebRTC cần STUN/TURN).
2. Bảng conntrack có giới hạn. Khi đầy (`nf_conntrack: table full, dropping packet`) — kernel drop packet **im lặng**, ứng dụng chỉ thấy timeout. Đây là một trong những sự cố khó chẩn đoán nhất vì không có lỗi ở tầng ứng dụng.
3. NAT gateway idle timeout (AWS: 350s TCP) → connection database giữ lâu không dùng bị cắt im lặng → lỗi "connection reset" ngẫu nhiên. Giải pháp: TCP keepalive < idle timeout.

```bash
# Kiểm tra conntrack
cat /proc/sys/net/netfilter/nf_conntrack_count
cat /proc/sys/net/netfilter/nf_conntrack_max
dmesg | grep conntrack
```

---

## 3. TCP — Giao thức quan trọng nhất bạn phải hiểu tận gốc

### Problem Statement
IP chỉ đảm bảo "cố gắng chuyển packet". Ứng dụng cần: dữ liệu **đến đủ, đúng thứ tự, không làm sập mạng**. Không có TCP, mọi ứng dụng phải tự viết retransmission, ordering, flow control — và lịch sử chứng minh đa số sẽ viết sai (congestion collapse 1986 xảy ra chính vì retry ngây thơ).

### Cách hoạt động bên trong

**3-way handshake** — không phải nghi thức, mà là trao đổi trạng thái tối thiểu để 2 bên đồng bộ sequence number:

```
Client                         Server
  │── SYN (seq=x) ──────────────▶│   SYN_SENT
  │◀───── SYN-ACK (seq=y,ack=x+1)│   SYN_RCVD (vào SYN queue)
  │── ACK (ack=y+1) ────────────▶│   ESTABLISHED (vào accept queue)
```

Hai queue phía server là chỗ sự cố hay xảy ra:
- **SYN queue** đầy (SYN flood hoặc traffic spike) → drop SYN → client retry sau 1s, 2s, 4s... → latency tăng vọt dạng bậc thang (1s, 3s, 7s) — **thấy latency đúng các mốc này gần như chắc chắn là SYN/accept queue drop**.
- **Accept queue** đầy (ứng dụng `accept()` chậm — ví dụ GC pause, thread pool đầy) → tương tự. Kiểm tra: `ss -lnt` xem cột `Recv-Q/Send-Q` trên listen socket; `netstat -s | grep -i "listen"` xem số lần overflow. Tuning: `net.core.somaxconn` và backlog của ứng dụng phải cùng tăng.

**Reliability**: mỗi byte có sequence number; receiver ACK; sender giữ timer, hết timer thì retransmit. Retransmission rate là **metric vàng** đánh giá chất lượng mạng: `netstat -s | grep -i retrans`. Trên 1% là mạng có vấn đề.

**Flow control** (receive window): receiver báo còn bao nhiêu buffer. Window = 0 nghĩa là **ứng dụng phía nhận đọc chậm** (không phải mạng chậm!). `ss -ti` thấy `rwnd` nhỏ → đi debug consumer, không phải mạng.

**Congestion control** (cwnd): sender tự ước lượng khả năng của mạng. Slow start: cwnd tăng gấp đôi mỗi RTT từ ~10 segment. Hệ quả thiết kế quan trọng: **connection mới thì chậm** — 1 request lớn trên connection mới cần nhiều RTT chỉ để cwnd mở. Đây là lý do kỹ thuật số 1 của **connection pooling và keep-alive**: connection "ấm" đã có cwnd lớn.

- Loss-based (CUBIC — default Linux): coi mất gói là tín hiệu nghẽn. Sai trên mạng wireless (mất gói do nhiễu, không phải nghẽn).
- Model-based (**BBR**): ước lượng bandwidth × RTT. Thường tăng đáng kể throughput trên đường dài/lossy. Bật: `net.ipv4.tcp_congestion_control = bbr`.

**Đóng connection và TIME_WAIT**: bên chủ động đóng phải giữ socket ở TIME_WAIT ~60s (chống packet cũ lạc vào connection mới). Hệ quả production: client (hoặc proxy) mở/đóng connection tần suất cao sẽ **cạn ephemeral port** (~28k port mặc định / 64s ≈ 460 conn/s là trần):

```bash
ss -s                                    # đếm timewait
cat /proc/sys/net/ipv4/ip_local_port_range
# Giải pháp đúng: connection pooling. Giải pháp tình thế:
sysctl net.ipv4.tcp_tw_reuse=1           # an toàn cho outbound
# KHÔNG bao giờ dùng tcp_tw_recycle (đã bị xóa khỏi kernel, phá NAT)
```

### Trade-off
| Trade-off | Nguyên nhân kỹ thuật | Hệ quả production |
|---|---|---|
| Reliability vs Latency | Retransmit + in-order delivery → 1 gói mất chặn mọi gói sau (**head-of-line blocking**) | 1% packet loss có thể tăng p99 latency 10× |
| Fairness vs Throughput | Congestion control nhường băng thông | 1 flow không chiếm hết pipe dù pipe rỗng lúc đầu (slow start) |
| Stateful vs Scale | Mỗi connection tốn memory kernel (~vài KB–vài chục KB) | 1M connection cần tuning fd limit, buffer, conntrack |

### Khi nào KHÔNG dùng TCP
Real-time media (game, voice, video call): dữ liệu cũ vô giá trị, retransmit là có hại → UDP + application-level recovery. Đó chính xác là lý do QUIC, WebRTC chọn UDP.

### Troubleshooting TCP — checklist
```bash
ss -ti dst <ip>          # cwnd, rtt, retrans của từng connection
netstat -s               # thống kê toàn hệ thống: retrans, listen drops
tcpdump -i any host <ip> and port 443 -w cap.pcap   # bắt gói, mở bằng Wireshark
sysctl net.ipv4.tcp_max_syn_backlog net.core.somaxconn
```
Dấu hiệu → nguyên nhân thường gặp:
- Latency bậc thang 1s/3s/7s → SYN hoặc accept queue drop, hoặc packet loss khi handshake.
- "Connection reset by peer" ngẫu nhiên sau idle → NAT/firewall/LB idle timeout > keepalive.
- Throughput thấp dù băng thông lớn, RTT cao → window nhỏ (BDP), xem `net.ipv4.tcp_rmem/wmem`.
- Zero window liên tục → consumer chậm, không phải mạng.

---

## 4. UDP

### Problem Statement
TCP áp đặt reliability + ordering cho MỌI ứng dụng. Có lớp ứng dụng không cần (DNS query 1 gói), hoặc bị hại bởi điều đó (real-time). UDP tồn tại như **"IP + port"** — mỏng nhất có thể, trả quyền quyết định cho ứng dụng.

### Bản chất và trade-off
UDP không có handshake → không có trạng thái → server phục vụ hàng triệu client bằng 1 socket. Nhưng:
- Không congestion control → ứng dụng UDP viết ẩu có thể **làm nghẹt mạng của chính mình và người khác**.
- Không handshake → dễ bị **spoofing + amplification attack** (DNS/NTP amplification: gửi query nhỏ với source IP giả, victim nhận response lớn). Đây là lý do mọi UDP service public phải có rate limit và response size limit.
- Firewall/NAT xử lý UDP bằng timeout ngắn (30s) → "connection" UDP qua NAT chết im lặng nhanh hơn TCP.

Dùng UDP khi: DNS, real-time media, telemetry chịu mất mát, và làm nền cho QUIC. Không dùng khi cần đúng-đủ dữ liệu và không muốn tự viết lại TCP (đa số backend service).

---

## 5. DNS — Hệ phân tán lớn nhất thế giới

### Problem Statement
Con người và cấu hình không thể dùng IP trực tiếp: IP thay đổi, một tên cần trỏ nhiều IP (load balancing, failover), một IP phục vụ nhiều tên. DNS là **lớp indirection toàn cầu** — và như mọi indirection, nó vừa là công cụ điều khiển traffic mạnh nhất, vừa là single point of failure kinh điển ("It's always DNS").

### Cách hoạt động bên trong
```
App → stub resolver (/etc/resolv.conf)
    → recursive resolver (ISP/8.8.8.8/VPC DNS)
        → root (.) → TLD (.com) → authoritative (example.com)
    ← answer + TTL → cache ở mọi tầng
```
Toàn bộ hệ thống scale được nhờ **cache theo TTL**. TTL là hợp đồng hai mặt:
- TTL dài → ít query, chịu được outage của authoritative, nhưng failover chậm.
- TTL ngắn → điều khiển traffic nhanh (blue-green, failover), nhưng tăng tải + tăng phụ thuộc vào DNS uptime.

**Thực tế production**: nhiều resolver/thư viện **không tôn trọng TTL** (JVM mặc định cache vĩnh viễn nếu không set `networkaddress.cache.ttl`!). Failover DNS 60s TTL nhưng traffic vẫn đổ về IP cũ 30 phút — nguyên nhân: cache ở tầng ứng dụng. Luôn kiểm tra hành vi cache của runtime.

### Record quan trọng với backend
`A/AAAA` (tên→IP), `CNAME` (alias — không được đặt ở zone apex), `SRV` (host+port, dùng cho service discovery), `TXT` (verification), `NS`. Trong Kubernetes, service discovery chính là DNS (`svc.namespace.svc.cluster.local`).

### Failure cases kinh điển
1. **NXDOMAIN storm / negative cache**: deploy trỏ sai tên → NXDOMAIN bị cache → sửa xong vẫn lỗi đến hết negative TTL.
2. **ndots trong Kubernetes**: mặc định `ndots:5` → mọi tên "ngắn" bị thử qua hàng loạt search domain trước → 4–5 query DNS cho MỖI request ra ngoài → CoreDNS quá tải, latency tăng. Giải pháp: dùng FQDN có dấu chấm cuối (`api.example.com.`) hoặc giảm ndots.
3. **DNS resolver timeout mặc định 5s** (glibc) và retry tuần tự → 1 resolver chết làm mọi request chậm đúng 5s. Thấy latency đúng 5.0s → nghĩ ngay đến DNS.

```bash
dig +trace example.com        # đi cả chuỗi ủy quyền
dig @10.0.0.2 example.com     # hỏi thẳng 1 resolver cụ thể
resolvectl statistics          # cache hit của systemd-resolved
tcpdump -i any port 53         # xem query thật sự đi đâu
```

---

## 6. HTTP/1.1 → HTTP/2 → HTTP/3 (QUIC): một chuỗi giải quyết head-of-line blocking

### Chuỗi vấn đề — vì sao phải tiến hóa
**HTTP/1.1**: 1 connection xử lý tuần tự 1 request tại một thời điểm → trình duyệt/client mở 6+ connection song song → tốn tài nguyên, mỗi connection lại slow-start riêng. Đây là **HOL blocking ở tầng ứng dụng**.

**HTTP/2** giải quyết bằng **multiplexing**: nhiều stream song song trên 1 connection TCP, header nén bằng HPACK. Nhưng lộ ra tầng dưới: **HOL blocking ở tầng TCP** — 1 packet mất chặn TẤT CẢ stream (TCP không biết stream nào cần byte nào). Trên mạng lossy, HTTP/2 có thể **tệ hơn** HTTP/1.1 với 6 connection. Bài học thiết kế: giải quyết vấn đề ở một tầng có thể chỉ đẩy nó xuống tầng dưới.

**HTTP/3 (QUIC)** giải quyết tận gốc: xây transport mới **trên UDP**, stream độc lập thật sự (mất packet chỉ chặn stream chứa nó), handshake gộp transport+TLS 1.3 (1-RTT, 0-RTT khi resume), **connection ID** thay vì 4-tuple → đổi mạng (WiFi→4G) không rớt connection. Chạy trên UDP không phải vì UDP "nhanh" mà vì **middlebox ossification**: không thể deploy giao thức IP mới qua Internet, UDP là cánh cửa duy nhất còn mở.

### Trade-off khi chọn cho backend
| | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---|---|---|---|
| HOL blocking | App layer | TCP layer | Không |
| CPU cost | Thấp | Trung bình | Cao hơn (QUIC ở userspace, chưa tối ưu offload NIC) |
| Phù hợp | Nội bộ đơn giản | gRPC, đa số API | Mobile, mạng kém, edge/user-facing |

Thực tế phổ biến: **HTTP/3 từ user → CDN/edge; HTTP/2 từ CDN → origin; gRPC(HTTP/2) nội bộ**. Đừng vội bật HTTP/3 origin nếu chưa đo: chi phí CPU +20–30% cho lợi ích nhỏ trong mạng datacenter chất lượng cao.

**Lưu ý vận hành HTTP/2**: multiplexing làm 1 connection mang rất nhiều request → **load balancing L4 mất cân bằng nặng** (1 connection dồn vào 1 backend). gRPC nội bộ cần L7 LB (Envoy, service mesh) hoặc client-side LB. Đây là lỗi phổ biến nhất khi đưa gRPC vào Kubernetes: `Service` mặc định là L4 → toàn bộ traffic dồn 1 pod.

---

## 7. TLS — bảo mật tầng transport

### Problem Statement
Mạng trung gian (ISP, WiFi, cloud provider) đọc và sửa được mọi thứ. TLS cung cấp 3 thứ: **encryption** (không đọc được), **integrity** (không sửa được), **authentication** (đúng là server đó — qua certificate + chain of trust về CA).

### Cách hoạt động (TLS 1.3)
```
Client ──ClientHello (+key share, SNI)──▶ Server
       ◀─ServerHello (+key share, cert, Finished)──
       ──Finished──▶  [1 RTT, sau đó dữ liệu mã hóa đối xứng]
```
- Asymmetric crypto chỉ dùng để **trao đổi khóa + xác thực**; dữ liệu thật đi bằng symmetric (AES-GCM/ChaCha20) vì nhanh hơn ~1000×.
- TLS 1.3 bỏ các cipher yếu, giảm còn 1-RTT (1.2 là 2-RTT), hỗ trợ 0-RTT resume (đổi lại rủi ro replay — **không dùng 0-RTT cho request non-idempotent**).
- **SNI**: client báo hostname trong ClientHello để 1 IP phục vụ nhiều cert — nhưng SNI là plaintext (thông tin rò rỉ, trừ khi có ECH).
- **Forward secrecy** (ephemeral key mỗi session): lộ private key của cert không giải mã được traffic quá khứ.

### Production considerations
- **Cert hết hạn là sự cố tự gây ra phổ biến nhất ngành**. Bắt buộc: tự động hóa (ACME/cert-manager) + alert khi cert còn <21 ngày. Không bao giờ quản lý cert thủ công.
- **TLS termination ở đâu?** Ở LB/edge (đơn giản, tập trung cert, mất mã hóa nội bộ) vs end-to-end/mTLS (an toàn hơn, phức tạp vận hành hơn — thường giao cho service mesh). Trade-off chuẩn: bắt đầu terminate ở edge, thêm mTLS nội bộ khi có yêu cầu compliance/zero-trust.
- CPU: handshake đắt, bulk encryption rẻ (AES-NI). Tối ưu bằng **session resumption** và keep-alive, không phải bằng tắt TLS.

```bash
openssl s_client -connect host:443 -servername host </dev/null | openssl x509 -noout -dates -issuer
curl -vI https://host 2>&1 | grep -iE "SSL|TLS|expire"
# Lỗi hay gặp: thiếu intermediate cert trong chain → curl lỗi nhưng browser chạy (browser tự vá chain)
```

---

## Tổng kết tư duy module

1. Mạng là chuỗi abstraction **rò rỉ** — kỹ sư giỏi debug bằng cách biết tầng nào đang rò rỉ lên tầng nào.
2. Mọi thứ stateful ở giữa đường (NAT, LB, firewall, conntrack) đều có **giới hạn bảng và timeout** — và chúng fail im lặng.
3. Connection mới thì đắt (handshake + TLS + slow start) → **tái sử dụng connection** là tối ưu quan trọng nhất, xem Module 02.
4. DNS và cert là 2 nguồn sự cố "tự gây" lớn nhất — tự động hóa và giám sát chúng trước tiên.
