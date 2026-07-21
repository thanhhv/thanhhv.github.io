+++
title = "09 – Security cho Blockchain Backend"
date = "2026-07-19T08:30:00+07:00"
draft = false
tags = ["backend", "blockchain", "web3"]
series = ["Blockchain cho Backend Engineer"]
+++

> Bổ sung cho phần security rải rác ở các level: gom thành bức tranh threat model đầy đủ, từ key đến giao thức đến con người.

---

## 1. Threat model — nghĩ như kẻ tấn công

Hệ Web3 khác backend thường ở một điểm định mệnh: **phần thưởng tấn công là tiền, thanh khoản ngay, không đảo ngược được, và thường ẩn danh được**. Backend thường bị tấn công để lấy data (bán lại vòng vo); backend Web3 bị tấn công để **rút tiền trực tiếp**. Mọi quyết định bảo mật phải xuất phát từ đây: bạn đang vận hành một cái két, không phải một cái CRM.

Bề mặt tấn công xếp theo tần suất gây thiệt hại thực tế:

1. **Private key / quyền ký** (chiếm phần lớn thiệt hại lịch sử: Ronin $624M, Mt. Gox, Coincheck $530M, hàng loạt vụ 2023-2025).
2. **Bug smart contract** (The DAO, Wormhole $326M, Nomad $190M, vô số DeFi exploit).
3. **Hạ tầng backend** (RPC giả, DNS hijack, dependency độc, nhân viên bị lừa/mua chuộc).
4. **Tấn công mức giao thức** (51%, eclipse — hiếm hơn nhưng cần hiểu).

## 2. Private Key Management (mở rộng Level 5)

### 2.1. Nguyên lý nền

- **Key = tiền.** Không phải "credential dẫn tới tiền" — chính nó là tiền. Ai đọc được một lần là xong.
- **Không đổi được.** Lộ password thì rotate; lộ key thì chỉ có sơ tán quỹ sang key mới.
- Thiết kế mọi thứ quanh 2 câu hỏi: *(1) key sống ở đâu và ai/cái gì đọc được nó? (2) một lần ký được phê duyệt bởi những gì?*

### 2.2. HSM vs MPC — hai triết lý

| | HSM | MPC (Threshold Signature) |
|---|---|---|
| Ý tưởng | Key nằm trong phần cứng chuyên dụng, không bao giờ xuất ra; phần cứng thực hiện phép ký | Key **không tồn tại nguyên vẹn ở bất kỳ đâu**; n bên giữ n mảnh, t-of-n cùng tính toán ra chữ ký mà không ghép key |
| Single point | Chính cái HSM (và admin của nó) | Không — phải chiếm ≥ t bên |
| Chữ ký ra | Chuẩn (ECDSA) | Chuẩn (on-chain không phân biệt được) |
| Vận hành | Trưởng thành, đơn giản hơn | Phức tạp (giao thức tương tác nhiều vòng), nhưng linh hoạt chính sách |
| Dùng bởi | Ngân hàng, sàn thế hệ đầu | Fireblocks, Coinbase, custody hiện đại |

Multisig on-chain (Gnosis Safe) là lựa chọn thứ ba: ngưỡng phê duyệt nằm **trong contract, công khai kiểm chứng được** — chuẩn cho quỹ DAO/treasury; nhược là mỗi chữ ký một tx (lộ policy, tốn gas) và chỉ dùng được trên chain hỗ trợ contract.

### 2.3. Policy engine — nơi bảo mật thật sự nằm

HSM/MPC chỉ trả lời "ký thế nào". Câu hỏi quan trọng hơn: **"có nên ký không?"** Signing service phải kiểm tra trước khi ký:

```
- Destination có trong allowlist? (rút về địa chỉ lạ → chặn/escalate)
- Tổng chi trong 24h của ví này còn trong hạn mức?
- Số tiền > ngưỡng X → yêu cầu phê duyệt của người thứ hai (4-eyes)
- Calldata decode ra có khớp template được phép? (chặn "payout" thật ra là approve() cho kẻ lạ!)
- Nguồn yêu cầu hợp lệ? (mTLS + service identity, không phải cứ trong VPC là tin)
```

Vụ Bybit 2025 (~$1.5B, lớn nhất lịch sử) là bài học đắt nhất về điểm mù này: kẻ tấn công không phá HSM — chúng **thao túng giao diện phê duyệt** (compromise máy/UI của người ký Safe) để người ký duyệt một payload khác với thứ họ tưởng. Bài học: người phê duyệt phải verify **calldata thật trên thiết bị độc lập** (hardware wallet hiển thị, script decode riêng), không tin UI duy nhất.

## 3. Các tấn công mức ứng dụng backend phải tự phòng

### 3.1. Replay Attack

- **Cross-chain replay:** tx hợp lệ trên chain A phát lại trên chain B (cùng address, cùng nonce). Phòng ở giao thức: chainId trong chữ ký (EIP-155). Phòng ở ứng dụng: chữ ký login/off-chain phải chứa chainId + domain + nonce + expiry (EIP-4361, EIP-712).
- **Signature replay trong contract:** contract nhận chữ ký (permit, meta-tx, claim) phải track nonce/`usedSignatures` — kẻ tấn công gửi lại chữ ký cũ để claim lần hai.

### 3.2. Double Spend từ góc nhìn dịch vụ nhận tiền

Với sàn/payment gateway, "double spend" hiện đại không phải phá consensus mà là **lợi dụng cửa sổ chưa-final của bạn**: nạp tiền → chờ bạn credit sớm → rút/tiêu → reorg (tự nhiên hoặc thuê 51% với chain nhỏ) làm tx nạp biến mất. Phòng: confirmation depth theo *giá trị* và theo *chi phí thuê hashrate của chain đó* (chain PoW nhỏ cần depth rất sâu hoặc từ chối); tách "số dư khả dụng để rút" khỏi "số dư hiển thị".

### 3.3. Front-running / MEV (từ góc nhìn người xây sản phẩm)

- Tx của chính backend (thanh lý, arbitrage, mint) có thể bị front-run → dùng private relay (Flashbots Protect) cho tx nhạy cảm.
- Sản phẩm cho user swap: bắt buộc slippage limit + deadline trong calldata; cảnh báo khi pool mỏng.
- Không thiết kế cơ chế "ai gửi tx trước được quyền X" — đó là lời mời bot chiến tranh gas.

### 3.4. Sybil và Eclipse (mức giao thức, cần biết để đánh giá rủi ro)

- **Sybil:** tạo hàng loạt identity giả — bị chặn ở consensus bằng PoW/PoS (Level 2), nhưng **sống lại ở tầng ứng dụng**: airdrop farming, vote governance bằng nghìn ví. Phòng ở app: proof-of-personhood, stake-weighted, phân tích đồ thị ví.
- **Eclipse:** cô lập một node bằng cách chiếm toàn bộ peer slots của nó → node thấy một phiên bản chain giả do kẻ tấn công dựng. Hệ quả cho backend: node "của bạn" cũng có thể bị lừa → **đối chiếu head/block hash giữa nhiều nguồn độc lập** (node riêng + ≥1 provider) là phòng tuyến rẻ mà hiệu quả; lệch nhau bất thường = alert.

## 4. Smart contract security từ ghế người tích hợp

Bạn có thể không viết contract, nhưng backend của bạn *tin* contract:

- **Token lạ là input độc:** fee-on-transfer (nhận được ít hơn amount), rebasing (số dư tự đổi), blacklist (USDT/USDC đóng băng được địa chỉ — hot wallet của bạn có thể bị freeze!), token revert-có-điều-kiện, decimals nói dối. Sàn phải có quy trình thẩm định token trước khi list + test trên fork.
- **Approve tối thiểu:** backend giữ token và phải approve cho contract ngoài (router, bridge) → approve đúng số cần cho từng lần hoặc dùng permit2; approve vô hạn cho contract ngoài = số dư của bạn phụ thuộc bug của họ.
- **Theo dõi admin/upgrade events** của mọi contract phụ thuộc (Level 4): impl đổi, owner đổi, pause bật — đều phải alert.
- Đọc audit report của protocol trước khi tích hợp; không audit / audit hạng thấp / deploy khác bản audit = rủi ro tín dụng tương ứng.

## 5. Bảo mật vận hành (OpSec) — nơi hầu hết vụ lớn bắt đầu

Các vụ lớn gần đây (Ronin, Bybit, hàng loạt 2024-2025) đều bắt đầu bằng **con người và endpoint**, không phải mật mã:

- Ronin: social engineering nhân viên (fake job offer chứa mã độc) → chiếm máy → lấy 5/9 validator key.
- Chuẩn phòng thủ: máy ký riêng biệt (không email/browse), hardware wallet cho mọi phê duyệt người, phân tách môi trường dev/prod tuyệt đối với quyền ký, đào tạo chống phishing có kiểm tra, và giả định "một laptop nhân viên đã bị chiếm" khi thiết kế policy (một máy bị chiếm không được đủ để mất tiền).
- Giới hạn blast radius là tư duy trung tâm: hot wallet nhỏ, hạn mức theo ngày, alert bất thường theo hành vi (đột nhiên rút về địa chỉ mới, khối lượng lạ giờ lạ).

## 6. Checklist bảo mật tổng hợp

**Key & Signing**
- [ ] Key không bao giờ ở dạng plaintext trong app/env/log/backup thường
- [ ] Signing service tách biệt + policy engine (allowlist, limit, 4-eyes)
- [ ] Phân tầng hot/warm/cold; hot ≤ mức chấp-nhận-mất
- [ ] Diễn tập sơ tán và khôi phục key định kỳ

**Ứng dụng**
- [ ] Chữ ký có domain, nonce, expiry, chainId; chống replay hai chiều
- [ ] Confirmation policy theo giá trị + theo chain; số dư khả-rút tách khỏi hiển thị
- [ ] Idempotency mọi luồng tiền; không float
- [ ] Token mới qua thẩm định + test fork trước khi chạm tiền thật

**Hạ tầng**
- [ ] ≥ 2 nguồn RPC độc lập, đối chiếu chéo head
- [ ] Dependency pin + scan; build reproducible cho service chạm key
- [ ] Audit log append-only mọi thao tác ký/phê duyệt
- [ ] Kill switch từng luồng, đã diễn tập

## 7. Tóm tắt

- Web3 backend = vận hành két tiền công khai trên Internet; threat model xoay quanh key, quyền ký và tính bất khả đảo ngược.
- Công nghệ ký (HSM/MPC/multisig) chỉ là nửa bài; nửa quyết định là **policy engine và quy trình phê duyệt chống thao túng UI**.
- Replay, double-spend-qua-reorg, MEV, token độc: phòng ở tầng ứng dụng bằng chữ ký có ngữ cảnh, confirmation theo giá trị, slippage, thẩm định token.
- Hầu hết thảm họa bắt đầu từ OpSec con người — thiết kế sao cho một máy bị chiếm không đủ để mất tiền.
