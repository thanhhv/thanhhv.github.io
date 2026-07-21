+++
title = "10 – Production Failure Cases"
date = "2026-07-19T08:40:00+07:00"
draft = false
tags = ["backend", "blockchain", "web3"]
series = ["Blockchain cho Backend Engineer"]
+++

> 12 tình huống sự cố có thật trong vận hành hệ Web3. Mỗi tình huống: triệu chứng → root cause → thành phần ảnh hưởng → metric/alert → điều tra → khắc phục → phòng tránh.
> Định dạng gọn để dùng như runbook.

---

## FC-01. Chain Reorganization sâu

**Triệu chứng:** Số dư user "tự giảm"; đơn nạp tiền đã credit biến mất khỏi explorer; indexer báo lỗi parentHash mismatch hoặc — tệ hơn — chạy tiếp êm ả với dữ liệu sai.

**Root cause:** Consensus chọn nhánh khác (Level 2): tự nhiên (network latency), do sự cố client (Polygon từng reorg 100+ block do bug), hoặc tấn công 51% (ETC 2020: 3.000+ block).

**Ảnh hưởng:** Indexer, ledger, mọi service đã phát side-effect từ block bị vứt.

**Metric/Alert:** `chain_reorg_depth` histogram — alert depth ≥ 3 (warning), ≥ policy confirmation (critical — nghĩa là chốt sổ có thể sai). Cách đo: mỗi block mới, so hash block (head-N) hiện tại với hash đã lưu.

**Điều tra:** Xác định fork point (đi lùi tới block có hash khớp); liệt kê mọi tx đã credit trong khoảng [fork_point, head_cũ]; đối chiếu tx nào có/không trên nhánh mới.

**Khắc phục:** Indexer unwind về fork point, replay nhánh mới (Level 5 §4.2); với tx nạp đã credit mà biến mất: đóng băng số dư tương ứng, chờ tx xuất hiện lại (thường được mine lại trong vài block) hoặc mở incident đòi soát thủ công; với payout đã gửi: signed tx vẫn hợp lệ — re-broadcast.

**Phòng tránh:** Chỉ side-effect sau finalized/depth theo policy; indexer có unwind được test bằng cách giả lập reorg (fork local node); tách bảng chưa-final/đã-final.

---

## FC-02. Node Out of Sync (lỗi thầm lặng nguy hiểm nhất)

**Triệu chứng:** Không có lỗi nào cả. User phàn nàn "nạp tiền 30 phút chưa thấy", số dư hiển thị cũ, tx gửi đi bị từ chối nonce-too-low.

**Root cause:** Node ngừng theo kịp head: hết disk (phổ biến nhất), IOPS không đủ khi state lớn, mất peer, consensus client chết trong khi execution client vẫn trả lời RPC, sau hard fork không upgrade client.

**Ảnh hưởng:** Mọi service đọc qua node đó — dữ liệu **cũ nhưng hợp lệ về mặt cú pháp**, không throw error.

**Metric/Alert:** `chain_head_lag{node} > 5 block` (so với max của cụm + nguồn ngoài); `node_disk_free < 15%`; `node_peer_count < 5`; sync status của consensus client.

**Điều tra:** `eth_syncing`, log client (thường thấy "state missing" / disk full / no peers), so head với etherscan/provider.

**Khắc phục:** Loại node khỏi LB (tự động qua health check); disk đầy → mở rộng/prune; hỏng data → restore snapshot + sync phần thiếu.

**Phòng tránh:** Health check theo **độ mới**, không theo HTTP 200 (Level 8 §2.1); ≥ 2 node; disk alert sớm với dự báo tăng trưởng (~50-100GB/tháng Ethereum full node); lịch upgrade client theo hard fork calendar.

---

## FC-03. RPC Timeout / Provider Outage

**Triệu chứng:** Latency tăng vọt, timeout hàng loạt, 429/503 từ provider; sản phẩm "đứng" dù hạ tầng của bạn xanh.

**Root cause:** Provider outage (Infura outage 2020 kéo sập cả MetaMask, nhiều sàn ngừng rút nạp), hết quota/credit, bị rate-limit vì query nặng (getLogs range lớn), hoặc mạng giữa bạn và provider.

**Ảnh hưởng:** Toàn bộ nếu single-provider — đây chính là bài học "phi tập trung ở giao thức, tập trung ở hạ tầng".

**Metric/Alert:** `rpc_errors_total{upstream}` rate > 5%/5min; `rpc_request_duration p95`; số upstream healthy < 2; credits còn lại.

**Điều tra:** Status page provider, tách lỗi theo method (chỉ getLogs lỗi → bị giới hạn query nặng; tất cả lỗi → outage), test trực tiếp bằng curl bỏ qua gateway.

**Khắc phục:** Failover tự động sang upstream khác (gateway Level 8); giảm tải: tạm dừng backfill indexer, tăng cache TTL, hoãn luồng không khẩn.

**Phòng tránh:** ≥ 2 provider + node riêng; kiểm tra định kỳ failover thật (game day); phân bổ query nặng vào node riêng; hợp đồng SLA nếu volume lớn.

---

## FC-04. Mempool Congestion & Gas Spike

**Triệu chứng:** Tx pending hàng loạt; phí ước tính tăng 10-50x; user phàn nàn phí; queue payout dồn ứ; `tx_stuck_total` tăng.

**Root cause:** Sự kiện hot (NFT mint, token launch, thanh lý dây chuyền khi giá sập, airdrop claim) đẩy baseFee lên theo cấp số nhân (Level 3). Đặc biệt hiểm: **thời điểm thị trường sập chính là lúc phí đắt nhất và cũng là lúc user rút tiền nhiều nhất.**

**Ảnh hưởng:** Transaction service (chi phí + độ trễ), mọi SLA liên quan tx.

**Metric/Alert:** baseFee vượt ngưỡng (vd > 100 gwei warning, > 300 critical); `tx_intent_age p95`; chi phí gas/giờ quy USD.

**Điều tra:** Xem nguồn spike (block explorer: contract nào chiếm block space), ước thời gian kéo dài (mint 10k NFT thì hết là xong; thanh lý dây chuyền theo thị trường).

**Khắc phục:** Kích hoạt chế độ phí cao theo playbook: tx khẩn (rút tiền user) trả phí thị trường; tx hoãn được (gom quỹ, payout nội bộ) xếp hàng chờ `maxBaseFee`; thông báo phí động cho user; speed-up các tx đang kẹt mà khẩn.

**Phòng tránh:** Phân loại tx theo mức khẩn ngay trong thiết kế intent; batch payout; chuyển luồng phù hợp sang L2; treasury dự phòng native token đủ cho kịch bản phí ×20.

---

## FC-05. Nonce Conflict / Nonce Gap

**Triệu chứng:** Lỗi `nonce too low` / `replacement transaction underpriced` hàng loạt; hoặc mọi tx của một ví pending vô hạn (gap); `hot_wallet_nonce_gap > 0`.

**Root cause:** Race khi cấp nonce (nhiều worker cùng hỏi RPC); người/CI gửi tx thủ công từ hot wallet ngoài hệ thống; tx nonce N bị drop khỏi mempool trong khi N+1... đã gửi; failover giữa 2 RPC lệch view `pending`.

**Ảnh hưởng:** Toàn bộ pipeline gửi tx của ví đó — nonce là tuần tự, một chỗ tắc là tắc hết.

**Metric/Alert:** So `next_nonce` nội bộ vs `eth_getTransactionCount(pending)` mỗi phút — lệch quá 2 phút → alert; đếm lỗi nonce theo mã lỗi RPC.

**Điều tra:** `eth_getTransactionCount(latest)` vs `(pending)` vs DB; tìm nonce nhỏ nhất chưa mined; kiểm tra tx đó còn trong mempool không (`eth_getTransactionByHash` trên nhiều node — mempool khác nhau giữa node!).

**Khắc phục:** Gap → re-broadcast `signed_raw` của nonce thiếu, hoặc filler tx (self-transfer, cùng nonce, gas cao); conflict → dừng worker, resync `next_nonce` từ chain, resume.

**Phòng tránh:** Single allocator có lock (Level 7 §2.3); cấm tuyệt đối thao tác tay trên hot wallet (ví thao tác tay riêng); lưu signed_raw để re-broadcast; drill kịch bản gap.

---

## FC-06. Transaction Stuck (pending vô hạn)

**Triệu chứng:** Intent ở `broadcast` quá SLA; user hỏi "tiền tôi đâu"; hash có trên hệ nhưng explorer không thấy hoặc thấy pending mãi.

**Root cause:** Gas dưới thị trường (spike sau khi gửi); nonce gap phía trước (FC-05); tx bị drop khỏi mempool im lặng (pool đầy) và chưa re-broadcast; node nhận tx nhưng chết trước khi gossip.

**Metric/Alert:** `tx_stuck_total` (pending > T); tuổi max của trạng thái broadcast.

**Điều tra:** Còn trong mempool không (hỏi nhiều node)? Nonce phía trước sạch chưa? Gas hiện tại của tx so với baseFee thị trường?

**Khắc phục:** Cây quyết định: (a) không còn trong mempool → re-broadcast signed_raw; (b) trong mempool, gas thấp → speed-up cùng nonce gas ×1.15+; (c) không cần nữa → cancel tx (cùng nonce, value 0, gửi về chính mình); (d) gap → xử lý FC-05 trước. Nhớ track mọi candidate hash sau replacement.

**Phòng tránh:** Monitor loop tự động speed-up theo chính sách (Level 7 §2.4); maxFee đặt đệm 2×baseFee từ đầu; re-broadcast định kỳ tx pending (vô hại, idempotent theo hash).

---

## FC-07. Indexer Lag

**Triệu chứng:** Dữ liệu app chậm hơn chain (user thấy tx trên explorer nhưng app chưa hiện); `indexer_lag_blocks` tăng đều không hồi.

**Root cause:** Khối lượng event đột biến (airdrop, mint hot) vượt throughput indexer; RPC chậm/bị rate-limit; DB nghẽn (thiếu index, lock contention, vacuum); reorg unwind lớn; bug memory leak.

**Ảnh hưởng:** Mọi tính năng đọc từ DB indexer; nguy hiểm hơn: các job nghiệp vụ chạy trên dữ liệu trễ (vd. kiểm tra "đã nạp chưa" trả lời sai "chưa").

**Metric/Alert:** `indexer_lag_blocks` > 10 và đang tăng; events/giây xử lý; thời gian xử lý per-block; DB slow query.

**Điều tra:** Lag do khâu nào — fetch (RPC latency), decode (CPU), hay write (DB)? Profile theo stage; xem block gần đây có gì bất thường (block đầy event của một contract hot).

**Khắc phục:** Scale khâu nghẽn: tăng song song fetch (range chia nhỏ), batch insert, tạm bỏ qua contract không quan trọng (nếu thiết kế cho phép filter), nâng DB. Trường hợp cháy: dựng indexer song song từ snapshot + catch-up.

**Phòng tránh:** Load test với kịch bản block "đầy sự kiện của mình"; backpressure tách khỏi đường realtime (backfill queue riêng); mọi consumer dữ liệu indexer phải biết lag hiện tại (API trả kèm `indexed_up_to_block`) thay vì tin dữ liệu là mới.

---

## FC-08. Validator Downtime (khi bạn vận hành validator / phụ thuộc chain nhỏ)

**Triệu chứng:** Với validator của bạn: mất reward, bị jail (Cosmos), leak nhỏ (Ethereum — offline penalty ≈ reward bị mất, không thảm họa; **slashing thật chỉ khi ký sai/double-sign**). Với chain: block time giãn, hoặc chain halt (Tendermint khi > 1/3 offline).

**Root cause:** Hạ tầng validator (disk, mạng); double-sign do chạy 2 instance cùng key khi failover sai — **đây mới là thứ gây slashing, và mỉa mai thay, thường do cố gắng đạt HA một cách sai lầm**.

**Metric/Alert:** Missed attestation/block liên tiếp; với chain phụ thuộc: `time_since_last_block` > 3× block time → alert "chain đang chậm/halt".

**Điều tra/Khắc phục:** Validator: khôi phục node, **tuyệt đối không bật node thứ hai cùng key khi chưa chắc node cũ chết hẳn** (dùng slashing-protection DB, remote signer như Web3Signer/Horcrux). Chain halt: không làm gì được ngoài chờ — tạm treo nạp/rút chain đó, thông báo user.

**Phòng tránh:** Remote signer + slashing protection; runbook failover được diễn tập; với sản phẩm đa chain: circuit breaker per-chain tự treo khi chain bất thường.

---

## FC-09. Network Partition

**Triệu chứng:** Hai nguồn RPC của bạn trả **hai head khác nhau kéo dài**; hoặc node của bạn thấy chain "một mình một kiểu" (nghi eclipse); finality lag tăng (`chain_finalized_lag` — Ethereum từng có sự cố finality stall tháng 5/2023, ~25-60 phút không finalize do bug client).

**Root cause:** Phân mảnh mạng Internet lớn, bug consensus client chiếm tỷ lệ đáng kể của mạng, hoặc partition cục bộ chỉ của hạ tầng bạn.

**Ảnh hưởng:** Nakamoto-chain: hai nhánh cùng dài → reorg lớn khi hàn gắn (FC-01); BFT-chain: halt (FC-08); Ethereum: chain vẫn chạy nhưng không finalize → mọi logic chờ `finalized` treo.

**Metric/Alert:** So head + block hash chéo ≥ 3 nguồn độc lập; `chain_finalized_lag` > 2 epoch → critical.

**Điều tra:** Phân biệt "mạng toàn cầu có vấn đề" (nhiều nguồn ngoài xác nhận, mạng xã hội/status các explorer) với "chỉ mình mình" (nguồn ngoài bình thường → vấn đề node/peer của bạn).

**Khắc phục:** Trong sự cố: **tạm dừng side-effect không đảo ngược được** (rút tiền lớn) — đây là lúc kill switch dùng đến; chờ finality trở lại rồi đối soát. Không hạ ngưỡng an toàn để "chạy tiếp cho êm".

**Phòng tránh:** Logic chờ finalized phải có nhánh timeout + escalate thay vì treo im; đa dạng client + đa nguồn đối chiếu; playbook "finality stall" viết sẵn ngưỡng quyết định.

---

## FC-10. Smart Contract Exploit (của protocol bạn tích hợp)

**Triệu chứng:** Dòng tiền bất thường từ contract đối tác; giá token liên quan sập; social media bùng cháy; đôi khi — chính hot wallet của bạn có approve cho contract đó.

**Root cause:** Reentrancy, oracle manipulation, bug logic, lộ admin key của protocol (Level 4, 9).

**Ảnh hưởng:** Quỹ của bạn trong protocol đó; token bạn đang giữ/list; user của bạn đang dùng tính năng tích hợp.

**Metric/Alert:** Theo dõi on-chain protocol phụ thuộc: outflow bất thường (rút > X% TVL trong Y phút), event admin (Upgraded, OwnershipTransferred, Paused), giá token lệch lớn giữa các venue.

**Điều tra:** Xác minh nguồn (đội protocol xác nhận? tx exploit cụ thể?); xác định exposure: số dư trong protocol, approve còn hiệu lực, vị thế user.

**Khắc phục:** **Tốc độ là tất cả:** rút quỹ khỏi protocol (script thoát viết sẵn cho mọi vị thế lớn); revoke approve; tạm ngừng nạp/rút token liên quan; đóng tính năng tích hợp; thông báo minh bạch.

**Phòng tránh:** Hạn mức exposure per-protocol như hạn mức tín dụng; chỉ approve đúng mức; script thoát + revoke được test; theo dõi alert chuyên (Forta, Hypernative, hoặc tự build từ event stream).

---

## FC-11. Bridge Hack / Bridge Insolvency

**Triệu chứng:** Token "wrapped" trên chain đích mất peg; bridge tạm dừng; tin tức hack; thanh khoản rút ồ ạt.

**Root cause lịch sử:** Ronin $624M (5/9 key một tổ chức, bị social engineering); Wormhole $326M (bug verify chữ ký — mint không thế chấp); Nomad $190M (bug init cho phép replay message với root 0x00 — "hack cộng đồng", ai copy tx cũng rút được). Mẫu chung: **bridge = két khổng lồ + bộ verify message xuyên chain, sai một trong hai là mất két.**

**Ảnh hưởng:** Nếu bạn giữ wrapped asset: tài sản có thể mất giá trị thế chấp về 0; nếu bạn tích hợp luồng bridge: user kẹt giữa hai chain.

**Metric/Alert:** Đối soát định kỳ **tồn quỹ hai đầu bridge** (locked bên A ≥ minted bên B — lệch là báo động đỏ); peg price wrapped/gốc; trạng thái pause của bridge contract.

**Điều tra/Khắc phục:** Ngừng ngay nhận nạp qua bridge đó và định giá lại wrapped asset (margin/collateral!); kiểm tra exposure; chờ thông tin chính thức trước khi mở lại.

**Phòng tránh:** Ưu tiên native bridge của rollup (trust = L1) hơn bridge bên thứ ba; hạn mức theo bridge; không nhận wrapped asset làm thế chấp với hệ số như tài sản gốc; kịch bản "depeg" trong risk model.

---

## FC-12. Sự cố tổng hợp: "Ngày thị trường sập" (case study)

Đáng phân tích vì mọi failure mode xảy ra **cùng lúc** — đúng lúc hệ thống được cần nhất:

```
Giá sập 30% trong 2 giờ →
├─ Thanh lý dây chuyền on-chain → gas spike ×30 (FC-04)
├─ User rút tiền ồ ạt → queue payout dồn (FC-06 hàng loạt)
├─ Khối lượng event ×20 → indexer lag (FC-07)
├─ Ai cũng gọi RPC nhiều hơn → provider rate-limit (FC-03)
└─ Hot wallet cạn nhanh → cần nạp từ cold GẤP (quy trình chậm nhất!)
```

**Bài học thiết kế rút ra:**

1. Capacity planning phải theo kịch bản ×20 bình thường, không phải trung bình.
2. Quy trình nạp cold→hot phải chạy được trong 30-60 phút kể cả 3h sáng (người trực có quyền, phương tiện, và diễn tập).
3. Ưu tiên rõ ràng khi tài nguyên khan: rút tiền user > nạp tiền credit > mọi thứ khác.
4. Degrade có kiểm soát: tăng ngưỡng phí tự chịu/chuyển phí cho user một cách minh bạch, thay vì âm thầm chết.
5. Sau sự cố: post-mortem đủ 12 mục trên — hầu như lần nào cũng lộ thêm 2-3 điểm yếu mới.

---

## Phụ lục: bảng alert tổng hợp (mức khởi điểm, chỉnh theo chain/SLA)

| Alert | Ngưỡng gợi ý | Mức |
|---|---|---|
| Head không tăng | > 3× block time | P1 |
| Node head lag | > 5 block | P2 |
| Finalized lag (Ethereum) | > 2 epoch | P1 |
| Reorg depth | ≥ 3 / ≥ conf policy | P2 / P1 |
| RPC error rate | > 5%/5min | P2 |
| Upstream healthy | < 2 | P1 |
| baseFee | > ngưỡng kinh tế của bạn | P3→P2 |
| tx stuck | > 0 quá 10 phút / > 10 | P2 / P1 |
| Nonce gap | > 0 quá 2 phút | P1 |
| Hot wallet balance | < X ngày chi tiêu | P2 |
| Indexer lag | > 10 block và tăng | P2 |
| Bridge reconcile lệch | > 0 | P1 |
| Webhook DLQ | > 0 | P3 |
