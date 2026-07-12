+++
title = "Chương 11 — Network Stack: hành trình một packet xuyên kernel"
date = "2026-02-21T18:00:00+07:00"
draft = false
tags = ["backend", "linux", "operating-system"]
series = ["Operating Systems cho Backend Engineer"]
+++

## 1. Problem Statement

NIC là phần cứng nhận/gửi **frame Ethernet thô**. Ứng dụng muốn một thứ hoàn toàn khác: "một kết nối đáng tin cậy tới host X, gửi byte theo thứ tự, đừng làm mất, đừng làm nghẽn mạng". Khoảng cách giữa hai thế giới đó — checksum, định tuyến, phân mảnh, retransmit, congestion control, ghép kênh theo port — chính là network stack trong kernel. Đây là mã đường-nóng lớn nhất và được tối ưu khốc liệt nhất của Linux: mọi request tới backend của bạn đi qua nó **hai lần** (vào và ra).

Nếu không có: mỗi app tự viết TCP (sai, không tương thích, không chia sẻ được NIC), hoặc thế giới không có Internet như ta biết.

## 2. Tại sao nó tồn tại trong kernel (mà không phải thư viện)

- **Chia sẻ + ghép kênh**: một NIC, nghìn socket của trăm process — phải có trọng tài phân phối packet theo (protocol, IP, port).
- **Bảo mật**: quyền bind port < 1024, firewall (netfilter), không cho process đọc traffic của nhau.
- **Hiệu năng**: xử lý trong interrupt/softirq ngay khi packet đến, không đợi app được schedule.
- Ngoại lệ chứng minh quy tắc: DPDK/XDP bypass kernel khi cần triệu packet/giây — trả giá bằng việc tự làm lại tất cả những điều trên.

## 3. Internal Architecture

```
  ỨNG DỤNG          socket API: socket/bind/listen/accept/connect/send/recv
─────────────────────────┬─────────────────────────────────────────────
  KERNEL                 ▼
  ┌── Socket layer ──────────────────────────────────────────────┐
  │  struct socket ↔ struct sock (TCP: tcp_sock)                 │
  │  SEND BUFFER (sk_sndbuf) │ RECEIVE BUFFER (sk_rcvbuf)        │ ← nơi app
  └────────────┬─────────────────────────▲───────────────────────┘   gặp kernel
  ┌── TCP ─────▼─────────────────────────┴───────────────────────┐
  │ state machine, seq/ack, retransmit (RTO), congestion control │
  │ (cubic mặc định / bbr), flow control (rwnd), Nagle, ...      │
  └────────────┬─────────────────────────▲───────────────────────┘
  ┌── IP ──────▼─────────────────────────┴───────────────────────┐
  │ routing (FIB lookup), netfilter hooks (iptables/nftables),   │
  │ fragmentation                                                │
  └────────────┬─────────────────────────▲───────────────────────┘
  ┌── Device ──▼─────────────────────────┴───────────────────────┐
  │ qdisc (fq_codel...), driver, RING BUFFER tx/rx               │
  └────────────┬─────────────────────────▲───────────────────────┘
─────────────────────────┼───────────────┼─────────────────────────
  HARDWARE      NIC ──DMA + interrupt/NAPI──  (RSS: nhiều queue → nhiều CPU)
```

Dữ liệu trong kernel di chuyển dưới dạng **sk_buff (skb)** — struct mô tả packet với con trỏ head/data/tail: các tầng thêm/bớt header bằng cách **dịch con trỏ, không copy** — bài học zero-copy nội bộ đầu tiên.

### 3.1. Đường nhận (RX) — từ dây đồng đến `recv()`

```
1. Frame đến NIC → NIC DMA vào RX ring buffer (RAM) → interrupt (lần đầu)
2. NAPI: kernel TẮT interrupt của NIC, chuyển sang POLL trong softirq NET_RX
   (tải cao = poll gộp nhiều frame/lần — đúng trade-off interrupt vs poll, chương 02)
3. GRO: gộp các segment liên tiếp thành super-packet (đỡ n lần xử lý stack)
4. IP: checksum, route → "cho máy này" → TCP
5. TCP: tra socket theo 4-tuple → đúng connection
   - đúng thứ tự → xếp vào receive buffer; sai thứ tự → out-of-order queue
   - cập nhật rwnd, gửi ACK (có thể delayed)
6. Socket có data → ep_poll_callback → đánh thức epoll/app (chương 10)
7. App: recv() → copy_to_user từ receive buffer → app có dữ liệu
```

Hai điểm production ngay từ đường RX: **RX ring đầy** (app/softirq không kịp gặt) → NIC vứt frame → `ethtool -S eth0 | grep -i drop`; **một CPU gánh mọi softirq** (không RSS/RPS) → `%si` 100% một core trong khi core khác rảnh — trần throughput vô hình.

### 3.2. Đường gửi (TX) và các bộ đệm

`send()` **không gửi packet** — nó copy vào send buffer rồi về. TCP quyết định *khi nào* dữ liệu thực sự rời máy, dựa trên: cwnd (congestion), rwnd (bên nhận), Nagle (gom byte nhỏ — tương tác thảm họa với delayed ACK: cụm 40ms latency; RPC framework tắt bằng `TCP_NODELAY` gần như mặc định). Xuống dưới: TSO/GSO đẩy việc cắt segment cho NIC làm bằng phần cứng.

**Chuỗi backpressure tự nhiên**: app ghi nhanh hơn mạng rút → send buffer đầy → `send()` block (hoặc EAGAIN) → app phải chậm lại. Service proxy/streaming *phải* thiết kế cho tín hiệu này (client chậm kéo lùi upstream — hoặc buffer nổ memory).

### 3.3. accept queue và backlog — cửa ải bị bỏ quên

```
SYN đến ──► SYN QUEUE (half-open,       ──3-way xong──► ACCEPT QUEUE ──accept()──► app
            chống đầy bằng syncookies)      (size = min(backlog, somaxconn))
                                             ĐẦY? → drop/RST ← "connection refused
                                                     /timeout" khi app chậm accept
```

Hai hàng đợi này là hai case sự cố riêng (SYN flood & backlog full — chương 16). Số phải nhớ: `net.core.somaxconn` (mặc định lịch sử 128, hiện 4096) **kẹp trên** tham số backlog mà app truyền vào `listen()` — chỉnh một trong hai là chưa đủ.

### 3.4. Zero copy: sendfile và splice

Bài toán: gửi file 1GB ra socket. Đường ngây thơ `read()+write()`: disk→page cache→**copy sang user buf**→**copy về socket buf**→NIC. Hai lần CPU copy + 4 lần vượt biên hoàn toàn vô nghĩa — user space không nhìn nội dung, chỉ trung chuyển.

- **`sendfile(out_sock, in_fd, off, len)`**: dữ liệu đi page cache → socket thẳng trong kernel; với NIC scatter-gather thì *0 lần CPU copy* — NIC DMA trực tiếp từ page cache. Kafka phục vụ consumer và Nginx serve static file sống bằng syscall này (chương 17).
- **`splice`**: tổng quát hóa qua pipe — trung chuyển giữa hai fd bất kỳ bằng cách chuyển *tham chiếu page*, không chuyển data. Proxy dùng để nối hai socket.
- Benchmark đại diện (file 10GB nóng cache → localhost): read+write ~2.5GB/s, CPU 90%; sendfile ~8-9GB/s, CPU 25% — khác biệt chính là hai lần copy và syscall per-chunk.

## 4. Cách hoạt động — TCP connection nhìn từ kernel, trọn vòng đời

```
client connect() ─SYN→ server (kernel TỰ trả SYN+ACK, app chưa biết gì)
        ←SYN+ACK─
        ─ACK→    connection vào accept queue → app accept() → có fd
[trao đổi: send/recv chỉ chạm BUFFER; TCP lo seq/ack/retransmit ngầm]
app close() ─FIN→ ... ←FIN─ ─ACK→
        → bên đóng trước vào TIME_WAIT 60s ← giữ chỗ chống lẫn packet cũ
```

TIME_WAIT là nguồn hiểu lầm kinh niên: hàng chục nghìn TIME_WAIT **gần như vô hại với server** (vài trăm byte/entry) — vấn đề thật chỉ có ở phía **client** tạo connection dồn dập tới cùng đích (cạn ephemeral port): fix đúng là `net.ipv4.tcp_tw_reuse=1` + connection pool/keep-alive; `tcp_tw_recycle` đã bị xóa khỏi kernel vì phá NAT — blog nào còn khuyên là blog đó hết hạn.

Điều đáng khắc sâu: **TCP state machine sống trong kernel, độc lập với app**. App treo GC 30 giây? Kernel vẫn ACK, vẫn giữ connection sống. App chết? Kernel gửi RST/FIN. Đây là lý do healthcheck tầng TCP ("port mở") gần như vô giá trị — phải check tầng ứng dụng.

## 5. Trade-off

| Quyết định | Được | Mất |
|---|---|---|
| Stack trong kernel | Chia sẻ, an toàn, app đơn giản | Copy user↔kernel; syscall; trần ~triệu pps/core → DPDK/XDP cho ai cần hơn |
| Buffer mọi tầng | Hấp thụ burst, throughput | Bufferbloat — latency ẩn trong hàng đợi; "send xong" ≠ "đã tới nơi" |
| TCP tự động hết | App khỏi lo mất mát/nghẽn | Hành vi ngầm khó thấy: Nagle, delayed ACK, RTO min 200ms, cwnd reset khi idle |
| Autotuning buffer (`tcp_rmem/wmem`) | Tự thích nghi BDP | Mặc định trần có thể thấp cho đường dài băng rộng (điều duy nhất thường đáng tăng) |
| syncookies | Sống sót SYN flood | Mất một số TCP option khi kích hoạt — chỉ là chế độ khẩn |

## 6. Production

- **`ss` là công cụ số một** (thay netstat): `ss -tnp state established '( dport = :5432 )'`; `ss -tmi` xem buffer/cwnd/rtt từng connection; Recv-Q/Send-Q cột đầu — Recv-Q cao = app đọc chậm, Send-Q cao = mạng/bên kia chậm. Với listen socket: Recv-Q/Send-Q = độ đầy/size accept queue.
- Drop & retransmit: `nstat -az | grep -Ei 'retrans|drop|overflow'` (TcpRetransSegs, ListenOverflows, ListenDrops — hai cái sau chính là backlog full); `ethtool -S` cho drop mức NIC.
- Latency phân đoạn: RTT trong `ss -i`; eBPF `tcprtt`, `tcpretrans`, `tcpdrop` — thấy từng sự kiện với stack trace, trả lời "mạng hay app".
- Sysctl đáng biết (đo trước khi chỉnh — chương 17 có bảng đầy đủ): `net.core.somaxconn`, `net.ipv4.tcp_max_syn_backlog`, `net.ipv4.ip_local_port_range`, `net.ipv4.tcp_rmem/wmem` (max), `net.core.netdev_max_backlog`, congestion control (`bbr` cho đường dài/lossy — đo với workload thật).
- Capture có kỷ luật: `tcpdump -w` với filter chặt + `-c` giới hạn; phân tích offline bằng tshark/wireshark. Trên máy 10Gbps, tcpdump không filter tự nó là sự cố.

## 7. Anti-pattern

- Không dùng connection pool/keep-alive — mỗi request một handshake (+TLS!) rồi đổ lỗi "mạng chậm"; đồng thời tự tạo bão TIME_WAIT phía client.
- Chỉnh sysctl network hàng loạt theo "gist tuning tối thượng" — có dòng vô hại, có dòng (`tcp_tw_recycle` xưa, buffer max khổng lồ gây bufferbloat) gây sự cố tinh vi.
- Bỏ qua `TCP_NODELAY` cho protocol request-response nhỏ → cụm 40ms bí ẩn ở p99.
- Đọc "connection refused" mà chỉ nghi app — không kiểm tra ListenOverflows (accept queue đầy do app chậm accept sau GC pause là kịch bản rất phổ biến).
- Load balancer healthcheck TCP-open trong khi app đã treo — traffic vẫn đổ vào xác sống.

## 8. Failure Analysis — case mẫu: timeout lác đác giờ cao điểm, mạng "bình thường"

- **Triệu chứng**: 0.5% request timeout lúc peak; ping/iperf giữa các máy đẹp; team mạng và team app đổ lỗi nhau hai tuần.
- **Điều tra**: `nstat` trên server app: `TcpExtListenOverflows` tăng đều đúng khung giờ sự cố. `ss -lnt`: listen socket Recv-Q chạm Send-Q (=128). App là Java, GC pause p99 ~800ms lúc peak.
- **Root cause**: GC pause → thread accept đứng → accept queue (128, vì somaxconn mặc định cũ) đầy trong milli-giây ở 5k conn/s → kernel drop SYN-ACK-hoàn-tất → client timeout. Không packet nào "mất trên mạng" cả — chúng chết ở cửa.
- **Fix**: somaxconn + backlog app → 4096 (mua thời gian sống qua pause); tuning GC giảm pause; sau đó chuyển service sang G1/ZGC. **Prevention**: alert trên ListenOverflows > 0 (mọi máy, mọi lúc — số này phải là 0 tuyệt đối), dashboard GC pause cạnh connection rate.

## 9. Khi nào không nên tối ưu

Nếu retransmit ~0, ListenOverflows = 0, RTT ổn, buffer autotuning chưa chạm trần — network stack không phải vấn đề của bạn, và BBR/sysctl/zero-copy sẽ không cứu được cái N+1 query đang chiếm 400ms trong request 450ms. Zero-copy chỉ có nghĩa khi CPU copy đo được là thành phần lớn (proxy/file server băng thông cao). Kernel bypass (DPDK/XDP) là bước cuối cùng có chi phí vận hành khổng lồ — dành cho tầng hạ tầng (LB, DDoS scrubbing), không phải service nghiệp vụ.

---

**Chương tiếp theo**: đã đủ nền để nói chuyện đo đạc nghiêm túc: [Chương 12: Linux Performance Tools](/series/linux-os-for-backend/12-performance-tools/) — cơ chế hoạt động của perf, eBPF, ftrace và bộ đồ nghề cổ điển.
