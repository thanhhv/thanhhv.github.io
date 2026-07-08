+++
title = "Chương 14 – Case Studies: Ráp toàn bộ kiến thức vào hệ thống thật"
date = "2026-07-09T06:00:00+07:00"
draft = false
tags = ["backend", "distributed-systems"]
series = ["Distributed Systems"]
+++

> Mỗi case study dưới đây là một bài tập tổng hợp: kiến trúc → vì sao → trade-off → failure case → bottleneck → cách mở rộng. Số chương trong ngoặc trỏ về lý thuyết tương ứng. Các con số là ước lượng để tập tư duy định lượng — thói quen quan trọng nhất khi thiết kế hệ thống.

---

## 1. URL Shortener — bài học vỡ lòng nhưng đủ cả trade-off

**Bài toán**: 100M link mới/tháng (~40 write/s), redirect 10K/s — **read:write = 250:1**, latency redirect phải <50ms.

**Kiến trúc**: API tạo link → sinh mã 7 ký tự base62 → ghi DB (key→URL). Redirect: cache-aside (chương 10) trước DB; CDN/edge cache cho link hot.

**Các quyết định đáng học:**
- **Sinh mã**: hash URL (trùng khi 2 người rút gọn cùng URL — chấp nhận hay muối?) vs counter toàn cục (một điểm nghẽn + đoán được ID) vs **dải counter cấp theo lô** (mỗi app node xin lô 100K ID từ coordinator — ghi nhanh, không nghẽn; đây là bài "né consensus trong hot path", chương 07 §8).
- **Storage**: dữ liệu KV thuần, không join → key-value store partition theo mã (hash partition — chương 06); một node PostgreSQL cũng thừa sức ở quy mô này (chương 01: đừng phân tán sớm!).
- **Consistency**: link vừa tạo phải dùng được ngay (read-your-writes — chương 03) nhưng lan tới edge cache có thể trễ giây — chấp nhận được. Xóa/vô hiệu link mới là bài khó: cache TTL + purge CDN (invalidation, chương 10).

**Failure & bottleneck**: hot key (một link viral 100K rps) → CDN hấp thụ, đây là lý do có tầng edge chứ không chỉ Redis (chương 10 §3.4). Đếm click: đừng `UPDATE count+1` mỗi redirect (write amplification vào một hàng — hot row); bắn event vào Kafka, aggregate theo lô (chương 09).

**Bài học chính**: ngay cả hệ "đơn giản nhất" cũng chứa: read/write ratio quyết định kiến trúc, ID generation phân tán, hot key, và analytics tách khỏi hot path.

---

## 2. E-commerce & Flash Sale — tồn kho là bài toán consistency đắt nhất

**Bài toán**: nền tảng 1M đơn/ngày, flash sale 12.12: 500K người tranh 10K suất trong 5 phút.

**Kiến trúc tổng**: tách service theo bounded context (Catalog, Cart, Order, Inventory, Payment) — Catalog read-heavy (cache + search index, CQRS — chương 09); Cart là dữ liệu hòa giải được (Dynamo-style, chương 05 — bài học từ chính Amazon); **Order+Inventory+Payment là saga orchestration** (chương 08): tạo đơn PENDING → giữ chỗ tồn kho (reserve, có TTL) → thanh toán (idempotency key) → xác nhận; mỗi bước có compensation.

**Flash sale — thiết kế riêng vì tải lệch 1000x ngày thường:**
```
User ──▶ CDN (trang tĩnh, đếm ngược) 
     ──▶ Queue chờ ảo (virtual waiting room — load shedding có UX, ch.11)
     ──▶ Trừ suất trong REDIS (Lua script atomic DECR — linearizable một node, ch.03)
            hết suất → từ chối ngay, KHÔNG chạm DB
     ──▶ Chỉ N người thắng đi tiếp vào saga đặt hàng thật
```
Vì sao: 500K request tranh 10K suất = 490K request **chắc chắn thất bại** — cho chúng chết càng sớm càng rẻ (edge > Redis > DB). Số suất là hot key chủ đích → một node Redis atomic thay vì "phân tán cho ngầu" (mã hóa đúng bài: cần linearizability cục bộ, chương 03) — muốn hơn nữa thì chia 10K suất thành 10 bucket × 1K (salting, chương 06 §5).

**Failure case kinh điển**: (1) Oversell — check-then-act không nguyên tử (chương 03 §8); (2) thanh toán timeout không rõ kết quả → query PSP + idempotency key, tuyệt đối không đoán (chương 08); (3) giữ chỗ không TTL → thanh toán bỏ dở khóa hết kho — reserve phải tự hết hạn + đối soát.

**Bài học chính**: phân loại consistency per-operation (catalog eventual, cart merge, tồn kho linearizable-tại-điểm-quyết-định) và "thiết kế cho từ chối sớm" là cốt lõi của mọi hệ bán hàng tải đỉnh.

---

## 3. Banking / Payment System — đúng trước, nhanh sau

**Bài toán**: chuyển tiền, số dư không bao giờ sai, audit 7 năm, đối tác (ngân hàng, PSP) là hệ ngoài không kiểm soát được.

**Kiến trúc lõi — ledger ghi kép, append-only:**
```
Chuyển 500K từ A→B = MỘT transaction ghi 2 entry bất biến:
  [debit  A  500K  txn_9182]
  [credit B  500K  txn_9182]
Số dư = SUM(entries) — có snapshot/materialized view để đọc nhanh (Event Sourcing đúng chỗ, ch.09 §5)
```
Vì sao: UPDATE balance đè mất lịch sử — regulator hỏi "vì sao số dư là X" phải trả lời bằng chứng cứ; entry bất biến còn **né bài toán lost-update** (không ai UPDATE, chỉ INSERT — chương 03 §9).

**Các quyết định đáng học:**
- **Trong một ledger**: transaction ACD I cục bộ trên một DB relational, partition theo account chỉ khi buộc phải (chương 08 thứ tự ưu tiên: vẽ ranh giới để transaction là cục bộ). Core ledger nhiều ngân hàng chạy trên **một** cụm DB mạnh + sync replica (chương 01 §9: scale up là lựa chọn nghiêm túc) — vì thứ họ cần là *đúng*, và 5K TPS ledger là trong tầm một máy.
- **Xuyên hệ (nạp/rút qua ngân hàng khác, PSP)**: không có transaction chung với hệ của người khác → **saga + idempotency key + trạng thái PENDING + đối soát file cuối ngày** (chương 08 §7 — reconciliation không phải phương án dự phòng, nó là *một phần của protocol* ngành tài chính).
- **Chống double-spend khi song song**: với ledger, ràng buộc `balance >= 0` kiểm tại commit (serializable hoặc constraint + retry khi conflict — optimistic, chương 03). Chống double-*submit*: idempotency key theo ý định (chương 08 §5).
- **DR**: RPO=0 là bắt buộc → sync/semi-sync replica (chương 05), multi-AZ, và backup immutable (chương 13).

**Failure case**: PSP trả timeout rồi 2 giờ sau báo thành công qua webhook — luồng phải thiết kế cho "kết quả đến muộn qua kênh khác" (state machine per-transaction, không phải request/response). Webhook đến **hai lần** và **sai thứ tự** — idempotent + state machine chỉ nhận chuyển tiếp hợp lệ.

**Bài học chính**: hệ tiền nong thắng bằng *mô hình dữ liệu* (immutable ledger, state machine) và *quy trình* (idempotency, đối soát) nhiều hơn bằng công nghệ phân tán tân kỳ.

---

## 4. Chat Application — fanout, ordering và presence

**Bài toán**: 50M user, tin nhắn 1-1 + nhóm, "đã xem", online status, lịch sử vô hạn.

**Kiến trúc**: client giữ **WebSocket** tới Gateway (stateful — điều làm chat khó hơn web thường); Gateway route theo `user_id → connection` (registry trong Redis, chương 02 service discovery cho *người*); message đi: sender → Chat service → ghi store → đẩy tới Gateway của người nhận (online) hoặc push notification (offline).

**Các quyết định đáng học:**
- **Ordering**: thứ tự *trong một hội thoại* là bắt buộc, giữa các hội thoại thì không → partition theo `conversation_id` (chương 06 câu hỏi tự kiểm tra số 1 — giờ có đáp án), sequence number cấp per-conversation (single writer per partition — né bài đồng hồ, chương 12 §5).
- **Storage**: `(conversation_id, seq)` làm khóa — Cassandra-style wide row là khớp tự nhiên; TTL/tiering cho media.
- **Nhóm lớn (100K thành viên)**: fanout-on-write (đẩy 100K bản sao) chết → nhóm lớn chuyển **fanout-on-read** (thành viên kéo từ log hội thoại khi mở — chính là mô hình consumer/log, chương 02). Cùng một bài với social feed (case 7).
- **Presence (online/offline)**: dữ liệu *đúng-tạm-thời*, sai vài giây vô hại → heartbeat + TTL trong Redis, eventual thoải mái (chương 03 — chọn model yếu nhất chịu được); đừng đốt engineering vào việc làm presence "chính xác".
- **Exactly-once với người dùng**: mạng di động rớt liên tục → client retry gửi kèm `client_msg_id` (idempotency key), server dedupe; receipt (delivered/read) là event bất đồng bộ, chịu trễ.

**Failure case**: Gateway node chết mang theo 500K connection → bão reconnect (thundering herd, chương 11) → client phải reconnect với backoff + jitter, server có connection rate limit; đây là lý do gateway phải nhiều node nhỏ thay vì ít node to (blast radius, chương 11 bulkhead).

**Bài học chính**: hệ stateful (connection) đòi tư duy khác hệ stateless: sticky routing, drain khi deploy, và bão reconnect là failure mode số một.

---

## 5. Notification Service — hệ "chỉ gửi thông báo" mà chứa cả giáo trình

**Bài toán**: gửi email/SMS/push cho 100M user từ hàng chục service nội bộ, qua provider ngoài (APNs, FCM, SMS gateway) có rate limit và hay chập chờn.

**Kiến trúc**: các service phát event (`OrderShipped`...) → Kafka → Notification service: ưu tiên hóa + template + preference (user tắt gì?) + **rate limit per-provider** (chương 11 §7) + retry queue riêng từng kênh + **DLQ có người trực** (chương 11 §8).

**Các quyết định đáng học:**
- Đây là **command đội lốt event làm đúng cách** (chương 09 §8): upstream phát *domain event* (sự thật), Notification service tự quyết có gửi không — thay vì mọi service tự gọi API email (logic notification vương vãi 30 chỗ).
- **Idempotency tuyệt đối**: gửi trùng email "đặt hàng thành công" là xấu hổ; gửi trùng OTP/SMS là *tốn tiền thật* + khóa tài khoản. Inbox dedupe theo `(user, notification_key)` (chương 08 §4).
- **Provider là dependency không kiểm soát được**: circuit breaker per-provider, failover SMS provider A→B (multi-vendor là bulkhead ở tầng nhà cung cấp), và **backpressure**: APNs giới hạn — queue trước mỗi provider phải bounded + shed theo ưu tiên (OTP > marketing) khi tràn (chương 11 §6).

**Failure case kinh điển**: sự cố làm tồn 5M notification trong queue, sự cố qua đi → hệ gửi 5M tin *cũ* (đêm khuya, nội dung hết hạn). Message phải mang **TTL nghiệp vụ** ("quá 30 phút thì đừng gửi nữa") — chính là bài "serving the dead" (chương 11 §6) ở tầng nghiệp vụ.

**Bài học chính**: một hệ "phụ trợ" tưởng đơn giản chạm đủ: event vs command, idempotency, rate limit, circuit breaker, priority shedding, DLQ, TTL nghiệp vụ.

---

## 6. Ride Hailing — dữ liệu địa lý thay đổi từng giây

**Bài toán**: khớp khách với tài xế gần nhất trong <3s; vị trí 1M tài xế cập nhật mỗi 4 giây (~250K write/s); giá động theo cung-cầu.

**Kiến trúc**:
- **Vị trí tài xế**: dữ liệu *tươi quan trọng hơn bền* — mất 1 điểm GPS chẳng sao, trễ 30 giây mới chết → in-memory store partition theo **geo-cell** (geohash/S2/H3 — partition key là không gian, chương 06 biến thể), TTL ngắn; không ghi DB bền trong hot path (ghi async cho analytics).
- **Matching**: query các cell quanh khách → ứng viên → chấm điểm (ETA, rating) → **khóa tài xế trong lúc mời** (lease + TTL: tài xế không nhận trong 10s thì tự nhả — chương 07 lease, và vì sao TTL chứ không lock vĩnh viễn: process matching có thể chết).
- **Hot cell**: sân bay giờ cao điểm — một cell chứa 5K tài xế + 2K yêu cầu (hot partition, chương 06 §5) → chia nhỏ cell theo mật độ (cell phân cấp — H3 các cấp) thay vì cell đều.
- **Trip state machine**: (requested→matched→arriving→in_trip→completed→paid) — bền, event-sourced-style, vì tranh chấp tiền nong cần lịch sử (chương 09); tách hẳn khỏi luồng vị trí (hai loại dữ liệu, hai yêu cầu, hai hệ — đừng nhét chung).
- **Surge pricing**: aggregate cung-cầu per-cell qua stream processing (Kafka + windowed aggregation) — eventual vài giây là bản chất chấp nhận được.

**Failure case**: mất khu vực matching service → khách thấy "không có xe" (degrade) trong khi tài xế đầy đường; double-match một tài xế cho 2 khách khi lease không nguyên tử → phải CAS trên trạng thái tài xế (chương 03 compare-and-set).

**Bài học chính**: phân loại dữ liệu theo (tươi/bền/đúng) rồi chọn store khác nhau cho từng loại trong *cùng một* nghiệp vụ — kỹ năng principal-level điển hình.

---

## 7. Social Network Feed — fanout và celebrity problem

**Bài toán**: 200M user, follow graph lệch cực đại (người thường 200 follower, celebrity 50M), feed phải <200ms.

**Hai chiến lược nền** (đây là bài trade-off compute-lúc-ghi vs compute-lúc-đọc):
- **Fanout-on-write (push)**: đăng bài → chèn vào feed cache (Redis list) của *mỗi* follower. Đọc feed = O(1). Nhưng celebrity đăng 1 bài = 50M write (write amplification giết hệ thống).
- **Fanout-on-read (pull)**: đọc feed = gom bài mới từ *mỗi* người mình follow rồi trộn. Ghi = O(1). Nhưng đọc của user follow 1.000 người = 1.000 fetch (chương 02 chatty — chết theo hướng ngược lại).

**Lời giải công nghiệp (Twitter công bố): lai** — người thường đi đường push; **celebrity đi đường pull** (bài của họ không fanout; lúc user mở feed, trộn feed-đã-push với "bài mới của các celebrity tôi follow" từ cache bài nóng). Ngưỡng celebrity là con số vận hành (vd >100K follower). Đây là hot key (chương 06/10) giải bằng **đổi hẳn thuật toán cho phần tử nóng** thay vì cào bằng — pattern đáng nhớ.

**Các mảnh còn lại**: đếm like/view — counter phân tán aggregate theo lô (không ai cần con số chính xác từng đơn vị — eventual, chương 03); ranking bằng ML → CQRS: pipeline offline build feature, online chỉ chấm điểm (chương 09); mọi thứ user-facing cần read-your-writes ("bài tôi vừa đăng phải hiện trong feed tôi" — chèn local trước, đường tắt UX rẻ hơn mọi giải pháp hạ tầng, chương 03).

**Failure case**: fanout queue tồn 30 phút → user đăng bài, bạn bè không thấy → nghĩ là mất bài → đăng lại (duplicate do *con người* retry — mọi định luật chương 01 áp dụng cả cho người). Giám sát fanout lag như SLI sản phẩm.

---

## 8. Search Engine (trong sản phẩm) — index là read model khổng lồ

**Bài toán**: tìm kiếm sản phẩm/nội dung trên 500M document, cập nhật "gần real-time", query p99 <300ms.

**Kiến trúc**: nguồn sự thật (DB) → CDC/event (chương 08 outbox, 09) → indexing pipeline → **inverted index** shard theo document (Elasticsearch/OpenSearch); query: scatter tới mọi shard → mỗi shard trả top-k → gather trộn (chương 06 §6 local index — vì search *bản chất* là query đa chiều, không tránh được scatter-gather).

**Hệ quả của scatter-gather** (điểm sâu nhất case này): p99 query = **max** của N shard → một shard chậm kéo cả cụm (tail at scale — chương 10 tài liệu đọc). Đối sách: shard đều (tránh shard to bất thường), **hedged request** (gửi bản sao request đến replica khi bản chính chậm quá p95), adaptive replica selection. Số shard vừa phải — 1000 shard cho 5GB dữ liệu là tự sát bằng overhead.

**Freshness vs cost**: index theo lô 1–5 giây (near-real-time là *thiết kế* của Lucene: segment bất biến + refresh — lại là immutability né consistency, chương 03 §9); sản phẩm vừa đăng chưa tìm thấy ngay là *hợp đồng* phải nói rõ với PM (chương 03 §6 document cam kết). Reindex toàn bộ (đổi mapping): build index mới song song + alias switch (blue-green cho dữ liệu — chương 09 §7 replay có kế hoạch).

**Failure case**: CDC pipeline chết âm thầm 2 ngày → search trả kết quả cũ, không ai alert vì "search vẫn chạy" — freshness lag phải là SLI riêng (đo bằng đối soát mẫu: lấy N bản ghi mới nhất từ DB, hỏi index có chưa).

---

## 9. Video Streaming — CDN là nhân vật chính

**Bài toán**: 10M người xem đồng thời, video 4K, startup <2s, không giật.

**Kiến trúc**: upload → transcoding pipeline (queue + worker fleet — bài toán batch song song, autoscale theo queue depth, chương 11 backpressure) → xuất **HLS/DASH: video cắt thành segment 2–6s nhiều bitrate** → đẩy vào origin storage → **CDN** phục vụ ~95–99% byte; player tự đổi bitrate theo băng thông (adaptive — degrade *ở client*, dạng graceful degradation đẹp nhất: người xem thà mờ hơn là dừng, chương 11 §6).

**Vì sao thiết kế này**: video là dữ liệu **bất biến, đọc cực nhiều, trễ ghi vô hại** — ngược hoàn toàn hồ sơ ngân hàng → mọi lựa chọn nghiêng hết về phía cache/replicate thoải mái (chương 03: immutability làm mọi thứ dễ). Cái *khó* dồn vào: livestream (đuôi latency: segment ngắn ↔ overhead — trade-off latency/throughput nguyên bản chương 01), và **thundering herd lúc sự kiện lớn**: trận đấu bắt đầu, 5M người cùng xin segment mới mỗi 2 giây → origin phải được che bằng **request coalescing tại CDN** (nghìn cache miss cùng key gộp thành 1 request về origin — chính là single-flight của chương 10 §3.1 ở tầng CDN).

**Failure case**: một PoP CDN hỏng → client failover PoP khác (anycast) — người dùng không biết; origin hỏng mới là sự cố thật → origin shield + multi-region origin (chương 13). Bài học phụ: **hệ tốt là hệ mà thành phần hỏng nhiều nhất (edge) được thiết kế hỏng rẻ nhất**.

---

## 10. AI Platform (serving + training) — hệ phân tán gặp GPU

**Bài toán**: phục vụ LLM/model inference cho 10K request/s, GPU đắt và khan hiếm; training job chạy tuần trên nghìn GPU.

**Serving:**
- **GPU là tài nguyên đắt gấp 100x CPU** → utilization là chỉ số sống còn → **batching động** (gom request trong cửa sổ vài ms — trade-off latency/throughput tường minh nhất bạn từng thấy, chương 01 §5; continuous batching cho LLM); model replica + LB nhận thức được độ dài queue từng replica (không round-robin mù — request LLM dài ngắn khác nhau 100x, chương 06 hot partition phiên bản compute).
- **Bài toán state mới: KV-cache** của phiên hội thoại nằm trên GPU node cụ thể → request tiếp theo *nên* quay lại đúng node (sticky routing — hệ stateful như chat, case 4) vs load balance — trade-off locality/balance kinh điển khoác áo mới.
- Backpressure là bắt buộc: GPU bão hòa thì queue dài vô ích → **shed + trả 429 với retry-after** sớm (chương 11), ưu tiên theo tier khách hàng; timeout của inference dài (giây) → càng phải bounded queue kẻo giam tài nguyên.

**Training:**
- Data-parallel: mỗi bước phải **all-reduce gradient giữa nghìn GPU** — đây là *phối hợp đồng bộ* thuần túy: bước đi theo node chậm nhất (straggler — tail latency chi phối, một GPU lỗi nhiệt kéo cả nghìn con chờ) → phát hiện straggler, topology mạng chuyên dụng.
- Job chạy 2 tuần trên 1000 node: xác suất *không* có node nào chết ≈ 0 (chương 01 §3.2) → **checkpoint định kỳ + tự động resume** là tính năng số một của mọi training platform; trade-off tần suất checkpoint (thời gian mất khi crash) vs chi phí ghi (bài RPO của chương 13, khoác áo ML).
- Scheduler cụm GPU (gang scheduling — job cần 512 GPU *cùng lúc* hoặc không gì cả) — bài toán phân bổ tài nguyên phân tán, họ hàng với chương 07 coordination.

**Bài học chính**: AI platform không có định luật phân tán mới — nó là các định luật cũ với **đơn giá tài nguyên cao hơn hai bậc**, nên các trade-off (batching, utilization, checkpoint, stickiness) từ "nên làm" thành "sống còn".

---

## Tổng kết chương

### Những điều bắt buộc phải nhớ — các pattern lặp lại xuyên 10 case
1. **Phân loại dữ liệu theo (tươi / bền / đúng / bất biến?) rồi mới chọn store** — mọi case đều bắt đầu ở đây, không bắt đầu ở "chọn công nghệ gì".
2. **Read:write ratio + phân bố key quyết định kiến trúc** — và phân bố luôn lệch (celebrity, sân bay, flash-sale SKU): thiết kế cho phần tử nóng bằng *đường đi riêng*, không cào bằng.
3. **Từ chối sớm và rẻ**: waiting room, CDN, negative cache, shed theo ưu tiên — hệ chịu tải đỉnh là hệ giỏi nói "không".
4. **Idempotency + state machine + đối soát** — bộ ba xuất hiện ở mọi case có tiền hoặc có hệ ngoài.
5. **Immutability né được cả họ bài toán consistency** — ledger, video segment, Lucene segment, event log.
6. **Hệ stateful (WebSocket, KV-cache GPU) đòi sticky routing + drain + chống bão reconnect** — lớp bài toán riêng mà hệ stateless không có.

### Câu hỏi tự kiểm tra tổng hợp
1. Chọn 2 case bất kỳ và chỉ ra cùng một pattern (vd single-flight) xuất hiện ở cả hai dưới hai cái tên khác nhau.
2. Thiết kế hệ đặt vé concert 50K vé, 2M người chờ — trộn những mảnh nào từ case 2, 3 và 5? Chỉ ra 3 điểm chết người nếu bỏ qua.
3. Với hệ bạn đang làm ở công ty: viết bảng phân loại dữ liệu (tươi/bền/đúng) như case 6, và một quyết định kiến trúc sẽ thay đổi sau khi viết bảng đó.

### Tài liệu kinh điển nên đọc
- **"Designing Data-Intensive Applications" (Kleppmann)** — nếu chỉ đọc một cuốn sách sau bộ tài liệu này, là cuốn này.
- **Twitter: "Timelines at Scale" (InfoQ talk, Raffi Krikorian)** — fanout lai push/pull từ người vận hành thật.
- **Uber Engineering Blog (H3, matching, schemaless)** — chuỗi bài ride-hailing chi tiết hiếm có.
- **"Scaling Memcache at Facebook"** + **"The Tail at Scale"** — hai bài được trích dẫn ở gần như mọi case bên trên; đọc lại lần nữa sau khi đọc chương này sẽ thấy khác lần đầu.
- **AWS Builders' Library** — tuyển tập cách Amazon vận hành: timeouts, retries, shuffle sharding, cell-based architecture — mỏ vàng cho mọi case study.
