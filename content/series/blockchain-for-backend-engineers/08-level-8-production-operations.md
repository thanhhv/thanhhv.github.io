+++
title = "Level 8 – Production Operations"
date = "2026-07-19T08:20:00+07:00"
draft = false
tags = ["backend", "blockchain", "web3"]
series = ["Blockchain cho Backend Engineer"]
+++

> **Câu hỏi trung tâm:** Vận hành hệ Web3 24/7: node, RPC, monitoring, DR — khác gì vận hành backend thường, và các con số cụ thể.

---

## 1. Node Deployment

### 1.1. Yêu cầu phần cứng (Ethereum mainnet, 2025-2026)

| Thành phần | Full node | Archive node |
|---|---|---|
| CPU | 8+ cores | 16+ cores |
| RAM | 32 GB | 64+ GB |
| Disk | **2 TB NVMe** (TLC, không QLC/SATA — IOPS là nút cổ chai số 1) | 16-20+ TB NVMe |
| Mạng | 25+ Mbps ổn định, chú ý băng thông ra (peer serving) | tương tự |
| Sync ban đầu | ~1-3 ngày (snap sync) | 2-6 tuần (full/archive sync!) |

Bài học vận hành đau nhất: **thời gian sync**. Node chết mất data = chờ nhiều ngày để có lại. Vì vậy: snapshot disk định kỳ (vd. mỗi 6-12h, dừng node hoặc dùng filesystem snapshot nhất quán), giữ ≥ 2 node để một chết vẫn còn một, và coi "restore từ snapshot + sync phần còn thiếu" là quy trình DR được diễn tập.

Solana khắc nghiệt hơn hẳn (256+ GB RAM, 10Gbps); Cosmos chain nhẹ hơn. Chạy trong Kubernetes được (StatefulSet + local NVMe PV) nhưng nhiều team chọn bare-metal/VM riêng cho node vì IO và vì node không cần scale ngang kiểu stateless.

### 1.2. Client diversity

Chạy ≥ 2 implementation khác nhau (vd Geth + Nethermind) nếu quy mô đủ lớn: bug trong một client (đã xảy ra: Geth v1.10.8 consensus bug 2021 tách chain) không hạ toàn bộ hạ tầng của bạn.

## 2. RPC Load Balancing & High Availability

### 2.1. Vấn đề đặc thù: load balance theo ĐỘ MỚI, không chỉ theo sống/chết

Node "sống" (trả lời HTTP 200) nhưng **lệch head 50 block** là node độc — trả dữ liệu cũ một cách hợp lệ. Health check phải so head:

```go
// Golang: RPC gateway health-check theo block height
type Upstream struct {
	URL     string
	Client  *ethclient.Client
	Healthy atomic.Bool
	Head    atomic.Uint64
}

func (g *Gateway) healthLoop(ctx context.Context) {
	t := time.NewTicker(5 * time.Second)
	for range t.C {
		var maxHead uint64
		heads := make([]uint64, len(g.ups))
		for i, u := range g.ups {
			cctx, cancel := context.WithTimeout(ctx, 2*time.Second)
			n, err := u.Client.BlockNumber(cctx)
			cancel()
			if err != nil { u.Healthy.Store(false); continue }
			heads[i] = n
			if n > maxHead { maxHead = n }
		}
		for i, u := range g.ups {
			// lành mạnh = trả lời được VÀ không tụt quá 3 block so với node tốt nhất
			u.Healthy.Store(heads[i] > 0 && maxHead-heads[i] <= 3)
			u.Head.Store(heads[i])
		}
	}
}
```

### 2.2. Chính sách routing thực dụng

- **Ưu tiên node tự vận hành, failover sang provider** (hoặc ngược lại tùy chiến lược chi phí/trust).
- **Sticky theo phiên logic:** chuỗi thao tác phụ thuộc nhau (đọc nonce → gửi tx; đọc block N → đọc state tại N) phải cùng upstream — hai upstream có thể lệch nhau 1-2 block.
- **Gửi tx: broadcast ra NHIỀU upstream cùng lúc** — tx idempotent theo hash, gửi trùng vô hại, tăng xác suất vào mempool nhanh.
- **Tách pool theo loại tải:** `eth_getLogs`/trace (nặng) đi node archive/riêng; `eth_call`/balance (nhẹ, cacheable) đi pool chung; đừng để một query getLogs 10k block làm nghẽn đường gửi tx.
- Retry: chỉ retry lỗi mạng/5xx sang upstream khác; **không retry mù** lỗi thực thi (`revert`) — nó sẽ revert ở mọi node.

### 2.3. Cache

| Dữ liệu | Cache | TTL |
|---|---|---|
| Block/tx/receipt đã **finalized** | Redis/CDN | Vô hạn (immutable — món quà lớn nhất của blockchain cho caching) |
| Balance, `eth_call` @ latest | Redis | 1-12 giây (1 block time), key kèm block number |
| Gas price/baseFee | Redis | 3-12 giây |
| Token metadata (name, decimals) | Redis/DB | Ngày (bất biến trên thực tế) |
| ENS/tên miền | Redis | Giờ |

Quy tắc: **key cache phải chứa block context** (`balance:0xabc:block:19000000` hoặc `:latest` với TTL ngắn). Cache "latest" quá lâu = phục vụ số dư sai sau khi user vừa nạp tiền — ticket support kinh điển.

## 3. Monitoring & Observability

### 3.1. Metrics bắt buộc (Prometheus)

**Tầng chain/node:**

```
chain_head_block{node}              — head từng node; ALERT: không tăng sau 3×block_time
chain_head_lag{node}                — maxHead - head(node); ALERT: > 5
chain_finalized_lag                 — head - finalized; ALERT: > ~64 slot (Ethereum
                                      finality stall — sự cố mạng nghiêm trọng)
chain_peer_count{node}              — ALERT: < 5
chain_reorg_depth                   — histogram; ALERT: depth ≥ 3
```

**Tầng RPC:**

```
rpc_request_duration_seconds{method,upstream}  — p50/p95/p99 per method
rpc_errors_total{method,upstream,code}         — ALERT: error rate > 5%/5min
rpc_upstream_healthy{upstream}                 — ALERT: số upstream lành < 2
rpc_provider_credits_used                      — chống bill shock & hết quota
```

**Tầng nghiệp vụ (quan trọng nhất — SLO đặt ở đây):**

```
tx_intent_age_seconds{status}       — histogram tuổi intent theo trạng thái
                                      ALERT: p95 broadcast→mined > 5 phút
tx_stuck_total                      — đang pending quá ngưỡng; ALERT: > 0 (P2), > 10 (P1)
tx_confirmed_total / tx_failed_total{reason}
hot_wallet_balance{chain,wallet}    — ALERT: dưới X ngày chi tiêu dự kiến (nạp thủ công từ cold cần thời gian!)
hot_wallet_nonce_gap               — ALERT: > 0 quá 2 phút
indexer_lag_blocks                  — head - block đã index; ALERT: > 10 và tăng
webhook_dlq_size                    — ALERT: > 0
```

### 3.2. Dashboard tối thiểu

1. **Chain health:** head/finalized/lag per node, peer, reorg events, gas price (baseFee) theo thời gian.
2. **Transaction pipeline:** funnel created→broadcast→mined→confirmed, tuổi theo trạng thái, stuck list, hot wallet balances + nonce.
3. **Indexer:** lag, events/giây, reorg unwinds, outbox backlog, Kafka consumer lag.
4. **RPC:** trạng thái upstream, latency per method, error rate, chi phí provider.

### 3.3. Logging & Tracing

- Correlation id xuyên suốt: `intent_id` ↔ `tx_hash` (nhớ: một intent nhiều hash sau speed-up) — log có cấu trúc `{intent_id, chain_id, tx_hash, nonce, status_from, status_to}`.
- Tracing (OpenTelemetry): span cho từng RPC call với `method`, `upstream` — điểm nghẽn thường lộ ra ở đây (getLogs range lớn, provider chậm ngầm).
- **Đừng bao giờ log private key, signed raw tx của ví nóng (chứa đủ thông tin để broadcast), seed, hay full payload ký.**

## 4. Incident Response đặc thù Web3

Khác biệt lớn nhất với backend thường: **không có rollback**. Tiền đã gửi nhầm là mất; contract đã gọi là đã gọi. Vì vậy:

- **Kill switch phải có sẵn:** cờ tắt payout tự động, tắt webhook, pause contract (nếu thiết kế có pause) — thao tác 1 lệnh, được diễn tập.
- Playbook cho các sự cố định kỳ xảy ra: gas spike, provider outage, reorg sâu, chain halt, stuck queue (chi tiết từng playbook trong file **10 – Production Failure Cases**).
- Nguyên tắc khi nghi ngờ lộ key: **coi như đã mất — sơ tán quỹ sang ví mới ngay lập tức**, điều tra sau. Tốc độ sơ tán là thứ quyết định; script sơ tán viết sẵn, test sẵn.

## 5. Disaster Recovery

| Tài sản | RPO/RTO | Chiến lược |
|---|---|---|
| Node data | RTO vài giờ | Snapshot disk 6-12h + ≥2 node; restore + catch-up sync |
| Indexer DB | RPO ~0 | PostgreSQL streaming replication + PITR; tệ nhất: re-index từ chain (chain là backup của chính nó — chậm nhưng luôn khả thi) |
| tx_intents DB | RPO = 0 bắt buộc | Sync replication; mất bảng này = không biết đã trả tiền cho ai |
| **Keys** | Mất = mất tiền vĩnh viễn | Nghi thức riêng: Shamir/multi-sig backup, nhiều địa điểm vật lý, diễn tập khôi phục **và** diễn tập sơ tán |
| Config (địa chỉ contract, ABI, policy) | RPO ~0 | Git + secret manager |

Điểm hay ít người nhận ra: với dữ liệu *đọc từ chain*, chain chính là backup — indexer DB cháy vẫn re-index lại được 100%. Với dữ liệu *chỉ của bạn* (intents, key, mapping user↔address) thì ngược lại: mất là mất tuyệt đối. Phân loại dữ liệu theo trục này quyết định đầu tư backup.

## 6. Cost Optimization

**Chi phí hạ tầng:**

- RPC provider tính theo compute units — `eth_getLogs`, `trace_*`, `debug_*` đắt gấp nhiều lần `eth_call`. Indexer tự vận hành + cache tốt thường cắt 60-80% bill provider.
- Cache immutable data vĩnh viễn (mục 2.3) — miễn phí và đúng tuyệt đối.
- Archive node chỉ khi thật sự cần state lịch sử; nhiều use case chỉ cần full node + indexer DB.

**Chi phí gas (thường lớn hơn chi phí hạ tầng!):**

- **Batching:** gom N payout vào 1 tx qua contract multisend — tiết kiệm ~40-70% (mỗi tx tiết kiệm 21k gas base + overhead).
- **Timing:** gas rẻ lúc thấp điểm (cuối tuần, đêm UTC); payout không gấp xếp hàng chờ ngưỡng giá — máy trạng thái intent (Level 7) hỗ trợ tự nhiên: thêm điều kiện `maxBaseFee` cho phép gửi.
- **L2:** chuyển luồng phù hợp sang L2 giảm phí 10-100x (đánh đổi ở Level 6).
- Đo lường: metric `gas_spent_total{flow}` quy ra USD theo giá native token — nhiều team không biết flow nào đốt tiền nhất cho đến khi đo.

## 7. Security Operations

- Signing service: mạng riêng, mTLS, allowlist destination + limit theo policy, audit log mọi lần ký (append-only).
- Phân quyền theo nguyên tắc: **không cá nhân nào một mình chuyển được số tiền lớn** (multi-approve trong policy engine, multisig với cold).
- Dependency audit: npm/go supply chain attack nhắm riêng vào ứng dụng crypto là có thật (event-stream 2018, hàng loạt package độc 2023-2025 nhắm ví) — pin version, lock file, scan.
- Định kỳ diễn tập: khôi phục key từ backup, sơ tán hot wallet, restore node từ snapshot, failover RPC. **Playbook chưa diễn tập = playbook không tồn tại.**

## 8. Tóm tắt Level 8

- Node: NVMe IOPS là vua; sync lại mất nhiều ngày → snapshot + node dư thừa; client diversity nếu đủ lớn.
- RPC HA: health = độ mới head, sticky theo phiên logic, broadcast tx đa upstream, tách pool theo loại tải.
- Cache: immutable = cache vĩnh viễn; latest = TTL 1 block; key chứa block context.
- Monitoring: SLO đặt ở tầng nghiệp vụ (intent age, stuck, hot wallet balance, indexer lag) — không chỉ tầng hạ tầng.
- Không có rollback trong Web3: kill switch + playbook + diễn tập; nghi lộ key = sơ tán ngay.
- DR: chain là backup của dữ liệu đọc-từ-chain; intents và keys là thứ mất-là-hết.
- Chi phí gas thường lớn hơn chi phí server — batch, timing, L2, và đo.

**Tiếp theo:** file **09 – Security**, **10 – Production Failure Cases** (12 tình huống sự cố chi tiết), **11 – System Design** (thiết kế 6 hệ thống thực tế), **12 – So sánh tổng hợp**.
