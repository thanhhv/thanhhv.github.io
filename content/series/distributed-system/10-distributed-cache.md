+++
title = "Chương 10 – Distributed Cache: Nhanh hơn, và những cách nó phản bội bạn"
date = "2026-07-09T02:00:00+07:00"
draft = false
tags = ["backend", "distributed-systems"]
series = ["Distributed Systems"]
+++

## 1. Problem Statement

Database là thành phần đắt nhất và khó scale nhất (chương 05–06). Trong khi đó read:write điển hình là 100:1 và phần lớn read lặp lại trên một nhóm nhỏ key (power law). Cache khai thác sự lặp lại đó: trả lời từ RAM (~100µs qua mạng nội bộ, ~100ns local) thay vì disk + query planner (~1–50ms), chặn 90–99% read trước khi chạm DB.

Nếu không có cache ở quy mô lớn: DB cần gấp 10–100 lần capacity — có khi bất khả thi về vật lý chứ không chỉ tiền. Nhưng cache về bản chất là **một replica eventual-consistency tự chế, không có replication protocol tử tế** — và vì thế toàn bộ chương 03/05 quay lại ám bạn dưới các cái tên mới: stale read, invalidation race, stampede, avalanche. Có câu đùa nghiêm túc: *"Chỉ có hai bài toán khó trong khoa học máy tính: cache invalidation và đặt tên."*

## 2. Các mẫu đọc/ghi — cơ chế và cái giá

### 2.1. Cache-Aside (Lazy Loading) — mặc định của ngành

```
Read:  app → cache. HIT → trả về.
       MISS → app → DB → app GHI vào cache (kèm TTL) → trả về.
Write: app → DB → app XÓA (không update!) cache key.
```

App tự điều khiển; cache chết thì hệ vẫn chạy (chậm). Vì sao **xóa** thay vì **update** khi write: hai update đồng thời có thể đến cache theo thứ tự ngược với DB (race) → cache giữ giá trị cũ *vĩnh viễn*; xóa thì tệ nhất chỉ là một lần miss. Nhưng xóa cũng không kín race:

```
Race kinh điển của cache-aside (hiếm nhưng có thật):
A: read miss → đọc DB được giá trị v1
B: write v2 vào DB → xóa cache
A: (chậm chân) ghi v1 vào cache  ← cache giờ giữ v1 cũ đến hết TTL
```

Vì lỗ này, **TTL là bắt buộc** trên mọi key — TTL là lưới an toàn cuối, chặn "cũ vĩnh viễn" thành "cũ tối đa X phút".

### 2.2. Read-Through / Write-Through / Write-Behind

- **Read-Through**: cache tự đọc DB khi miss (logic nạp nằm ở tầng cache/thư viện). Gọn cho app; kém linh hoạt hơn cache-aside.
- **Write-Through**: write đi qua cache → cache ghi DB đồng bộ → cache luôn tươi cho key đã ghi. Giá: write latency tăng (2 hệ), và ghi cả những key không bao giờ được đọc lại.
- **Write-Behind (Write-Back)**: write vào cache, ghi DB **async theo batch**. Write latency cực thấp, gộp write (tốt cho counter/view count). Giá: **cache chết = mất write chưa flush** — chỉ dùng cho dữ liệu chịu mất được, hoặc cache có replication + persistence (và lúc đó bạn đang vận hành một database thứ hai, hãy thành thật với điều đó).

| | Cache-Aside | Read-Through | Write-Through | Write-Behind |
|---|---|---|---|---|
| Độ tươi sau write | Cũ đến khi xóa/TTL | Như cache-aside | Tươi (key đã ghi) | Cache tươi, **DB cũ** |
| Write latency | DB | DB | DB + cache | Thấp nhất |
| Rủi ro mất dữ liệu | Không | Không | Không | **Có** |
| Chống miss lần đầu | Không | Không | Không (trừ khi warm) | — |
| Dùng phổ biến | ✅ mặc định | Thư viện/CDN | Dữ liệu đọc-ngay-sau-ghi | Counter, metrics, buffer |

## 3. Bốn thảm họa mang tên Cache — phân biệt cho đúng

Bốn từ hay bị dùng lẫn; chúng là bốn bệnh khác nhau với bốn thuốc khác nhau:

### 3.1. Cache Stampede / Breakdown (một key nóng hết hạn)

Key hot (trang chủ, config, celebrity profile) hết TTL → **hàng nghìn request đồng thời cùng miss → cùng đổ vào DB query một thứ** → DB quỵ → mọi thứ sau nó quỵ. Còn gọi là dog-piling; "breakdown" trong tài liệu Trung/Việt chính là bệnh này.

Thuốc:
- **Lock/single-flight**: chỉ MỘT request đi nạp (mutex per-key — Redis `SET NX`, Go singleflight), số còn lại chờ hoặc **nhận tạm giá trị cũ** (serve-stale — thường đúng đắn nhất về UX).
- **Xác suất hết hạn sớm (probabilistic early expiration)**: mỗi read có xác suất nhỏ tự refresh trước TTL — hot key gần như luôn được refresh trước khi chết.
- **Refresh chủ động**: job nền refresh các key hot theo lịch; TTL chỉ là lưới an toàn.
- **Không bao giờ để key sống còn của toàn trang có TTL "tròn giờ"**.

### 3.2. Cache Avalanche (chết hàng loạt)

Hàng loạt key hết hạn **cùng lúc** (cùng TTL 3600s nạp lúc deploy; hoặc cả cluster cache restart) → DB nhận cả thác miss. Thuốc: **TTL + jitter ngẫu nhiên** (3600 ± random 300s) — một dòng code, giá trị bằng cả một cụm DB; cache HA (replica, persistence để restart ấm); warm-up trước khi nhận traffic; và tầng dưới phải tự vệ (rate limit, circuit breaker — chương 11) vì avalanche *sẽ* có ngày xảy ra.

### 3.3. Cache Penetration (xuyên thủng bằng key không tồn tại)

Query key **không có trong DB** (user_id=-1, SKU bịa) → miss → DB → không có gì để cache → lặp mãi. Kẻ tấn công (hoặc bug client) bơm key rác = mọi request xuyên thẳng DB, cache thành đồ trang trí. Thuốc: **cache cả kết quả rỗng** (negative caching, TTL ngắn 30–60s); **Bloom filter** trước cache (chắc chắn-không-có → chặn ngay, không tốn cache memory cho key rác); validate input sớm (id âm, format sai → 400 ngay ở edge).

### 3.4. Hot Key (một key vượt sức một node cache)

Một key nhận triệu RPS — vượt trần MỘT node Redis (~100K–1M ops/s tùy payload) — cluster to đến đâu key đó vẫn nằm trên một node (chương 06 nguyên văn). Thuốc: **local cache tầng app** (L1 in-process vài giây TTL — hấp thụ phần lớn; trả bằng thêm một tầng staleness); **key replication** (`key#1..#N`, đọc random — trả bằng invalidation N bản); read replica của node cache đó.

## 4. Kiến trúc cache nhiều tầng & invalidation

```
Client ─▶ CDN (tĩnh + API cacheable, TTL ngắn)
        ─▶ L1: in-process (Caffeine/Guava — ns, per-instance, nhỏ, TTL giây)
        ─▶ L2: Redis/Memcached cluster (µs–ms, shared, to)
        ─▶ Database
```

Mỗi tầng thêm là một bản sao thêm → **bài toán invalidation nhân lên**. L1 per-instance đặc biệt hóc: xóa L2 dễ, nhưng 200 instance mỗi con giữ L1 riêng — cần pub/sub invalidation (Redis pub/sub bắn "key X đổi" cho mọi instance) hoặc chấp nhận TTL ngắn là mức staleness của L1. Phần lớn hệ chọn cách hai — đơn giản thắng.

**Invalidation nghiêm túc bằng CDC**: Debezium đọc binlog/WAL → mọi write vào DB (kể cả từ script, migration, service khác) sinh sự kiện xóa cache — kín hơn hẳn "app tự nhớ xóa" (app *sẽ* quên ở đâu đó). Facebook memcache (paper 2013) dùng chính binlog pipeline (McSqueal) cho việc này, cộng thêm **lease** chống race set-sau-delete — bài toán §2.1 ở quy mô tỷ user.

**Redis vs Memcached** (chọn nhanh): Memcached — thuần KV, multi-thread, đơn giản, LRU tốt → pure cache lớn. Redis — cấu trúc dữ liệu (sorted set cho leaderboard, hash, stream), Lua, persistence, replication, cluster → "cache cộng thêm". Đa số chọn Redis vì tính năng; chọn Memcached khi chỉ cần cache thuần và muốn tận dụng multi-core.

## 5. Trade-off

- **Freshness vs Load**: TTL dài → DB nhẹ, dữ liệu cũ lâu; TTL ngắn → tươi, DB gánh. Chọn TTL = trả lời câu hỏi nghiệp vụ "dữ liệu này cũ X giây thì thiệt hại gì?" — giá sản phẩm 60s thường ổn, số dư khả dụng thì không.
- **Hit ratio vs Memory (tiền)**: đuôi dài của phân phối truy cập cho hit ratio tăng theo log của dung lượng — 99%→99.5% có thể tốn gấp đôi RAM. Tối ưu *cái được cache* (đừng cache blob 2MB ít đọc) trước khi mua RAM.
- **Consistency vs Simplicity**: cache "đúng tuyệt đối" cần CDC + lease + pub/sub + đối soát — một hệ thống thật sự. Đa số nghiệp vụ chỉ cần "cũ tối đa 30 giây" — TTL + jitter + xóa-khi-ghi là đủ và *đơn giản hơn 10 lần*. Đừng xây pipeline Facebook cho trang blog.
- **Cache như một dependency mới**: thêm cache là thêm một hệ để chết. Nghịch lý capacity: hệ chạy 10x tải nhờ cache hit 95% → **cache chết là DB nhận 20x tải bình thường** → sập tức thì. Cache lâu ngày trở thành *bắt buộc* để hệ sống — lúc đó nó không còn là "tối ưu" nữa mà là hạ tầng critical, phải HA như DB.

## 6. Production Considerations

- **Metrics bắt buộc**: hit ratio (per key-pattern, không chỉ tổng — tổng 95% có thể che một pattern 40%), p99 latency, eviction rate (eviction cao = thiếu RAM = hit ratio sắp rơi), memory fragmentation (Redis), số connection.
- **Eviction policy**: hiểu `maxmemory-policy` của Redis — `allkeys-lru` cho pure cache; `noeviction` (mặc định!) làm write **fail** khi đầy RAM — nhiều sự cố production bắt đầu bằng "Redis tự nhiên từ chối ghi".
- **Expire ≠ eviction**: key hết TTL trong Redis bị xóa lazy + active sampling — memory không giải phóng tức thì; đừng ngạc nhiên khi memory cao dù "mọi key có TTL".
- **Capacity & failover**: cache cluster cũng cần chương 05–06 (Redis Cluster: 16384 slot, replica per shard, và cẩn thận split brain khi partition — `min-replicas-to-write` chỉ giảm nhẹ). Diễn tập "cache chết toàn phần": hệ có sống nổi không, degrade thế nào?
- **Security**: Redis không auth mở ra internet là cửa hậu kinh điển (RCE qua CONFIG SET). VPC-only, AUTH/ACL, TLS.
- **Serialize**: chi phí serialize/deserialize + băng thông thường là bottleneck thật của cache "chậm" — đo trước khi đổ lỗi cho Redis; nén payload lớn, chọn format gọn (msgpack/proto thay JSON dài dòng).

## 7. Best Practices

1. TTL trên **mọi** key + jitter. Không ngoại lệ — key "vĩnh viễn" là bug chờ ngày phát tác.
2. Xóa (delete) khi write, đừng update; qua CDC nếu nguồn write đa dạng.
3. Single-flight cho mọi đường nạp cache — chi phí một mutex, chặn cả lớp stampede.
4. Negative caching + validate input cho key ngoài kiểm soát.
5. Namespace + version trong key (`v2:user:123:profile`) — "invalidate toàn bộ" = tăng version, không cần FLUSHALL.
6. Load test **với cache lạnh** — con số ấy mới là capacity thật của bạn khi có sự cố.
7. Đặt ngân sách staleness cho từng loại dữ liệu, ghi vào doc như một phần hợp đồng API (chương 03).

## 8. Anti-patterns

- **Cache làm source of truth bất đắc dĩ**: dữ liệu chỉ tồn tại trong Redis (session, giỏ hàng) mà Redis không persistence/replication đúng mức → một lần failover là "đăng xuất toàn dân".
- **Cache-then-write** (ghi cache trước, DB sau, không phải write-behind có thiết kế): crash giữa chừng = cache nói dối.
- **FLUSHALL lúc cao điểm** / deploy xóa sạch cache: tự tạo avalanche. Warm trước, xóa theo version key.
- **Cache mọi thứ theo phản xạ**: query nhanh sẵn (PK lookup, 0.5ms) cache vào chỉ thêm một tầng staleness + race; cache là công cụ cho *đọc đắt + lặp lại*.
- **KEYS / SCAN pattern trong hot path** (Redis single-thread bị chặn) — KEYS trên production là lệnh cấm.
- **Một Redis cho tất cả**: cache + queue + lock + session chung một instance — sự cố một vai kéo sập mọi vai; tách theo vai trò.
- **TTL đồng loạt tròn số** nạp cùng lúc — avalanche hẹn giờ.

## 9. Khi nào KHÔNG cần cache

Dữ liệu đọc ít hoặc không lặp (báo cáo ad-hoc, admin query) — hit ratio thấp, chỉ thêm phức tạp. DB còn thừa capacity và latency đạt SLA — thêm cache là tối ưu sớm; index đúng + query đúng thường rẻ hơn cả về tổng thể. Dữ liệu đòi strong consistency tuyệt đối trong mọi read — cache về bản chất mâu thuẫn với yêu cầu này; đọc thẳng DB (và scale DB) là câu trả lời thẳng thắn. Payload rất lớn ít đọc — băng thông + RAM không bù lại được.

## 10. Troubleshooting

| Triệu chứng | Chẩn đoán | Xử lý |
|---|---|---|
| DB load tăng vọt định kỳ đúng chu kỳ | Avalanche hẹn giờ (TTL đồng loạt) | Jitter; xem phân bố TTL theo thời gian |
| DB spike mỗi khi một key hết hạn | Stampede key hot | Single-flight + serve-stale + early refresh |
| Hit ratio tụt dần theo tuần | Dung lượng không theo kịp dữ liệu; eviction tăng; key pattern mới không cache được | Eviction rate; phân tích hit theo pattern |
| Một node Redis CPU 100%, các node khác nhàn | Hot key (hoặc lệnh O(N) — SMEMBERS to, KEYS) | redis-cli --hotkeys; SLOWLOG; L1/key-replication |
| Dữ liệu cũ dai dẳng quá TTL | Race set-sau-delete (§2.1); hoặc L1 không được invalidate | Lease/version check khi set; pub/sub cho L1; rút TTL |
| Latency cache p99 nhảy | Fragmentation/swap; connection storm; lệnh chặn; persistence fork (RDB/AOF rewrite) | INFO memory; pool connection; tách persistence sang replica |
| Redis từ chối ghi "OOM" | maxmemory + noeviction | Đổi policy phù hợp vai trò; thêm RAM; xem key to bất thường |

---

## Tổng kết chương

### Những điều bắt buộc phải nhớ
1. Cache là **replica eventual tự chế** — mọi định luật chương 03/05 áp dụng, chỉ là không ai viết protocol hộ bạn.
2. Cache-aside + delete-on-write + TTL-mọi-key + jitter là bộ mặc định đúng; TTL là lưới an toàn chống "cũ vĩnh viễn".
3. Bốn bệnh bốn thuốc: stampede→single-flight/serve-stale; avalanche→jitter/HA; penetration→negative cache/Bloom; hot key→L1/replication.
4. Hệ sống nhờ cache hit 95% sẽ **chết trong vài giây** khi cache chết — cache lâu ngày là hạ tầng critical, phải HA và phải diễn tập cold-cache.
5. Write-behind = nhận rủi ro mất dữ liệu có chủ đích; đừng nhận nó một cách vô tình.
6. Đo hit ratio theo pattern và eviction rate — hai chỉ số kể trước tương lai.

### Hiểu lầm phổ biến
- "Thêm cache luôn nhanh hơn" — thêm một hop mạng cho dữ liệu vốn rẻ là chậm đi; và serialize có thể đắt hơn query.
- "Xóa cache khi ghi là đủ consistency" — race §2.1 vẫn tồn tại; TTL mới là lưới cuối.
- "Redis Cluster giải quyết hot key" — không; key vẫn ở một node, đó là bài của chương 06.
- "Cache invalidation khó vì thiếu công cụ" — khó vì nó *là* bài toán replication consistency, bản chất khó (chương 03).

### Câu hỏi tự kiểm tra
1. Trang sản phẩm 50K RPS, dữ liệu đổi vài lần/ngày, đội flash-sale cần giá đúng trong 5 giây sau khi đổi. Thiết kế tầng cache + invalidation + con số TTL, giải thích từng lựa chọn.
2. Vẽ timeline race set-sau-delete của cache-aside và hai cách vá (lease của Facebook; version check).
3. Redis của bạn hit 96%, DB chạy 30% capacity. Cache chết toàn phần: tính DB nhận bao nhiêu x tải, hệ sống không, và ba việc bạn làm *trước* để sống.
4. Khi nào bạn cache negative result với TTL dài là nguy hiểm? (dữ liệu sẽ-sớm-tồn-tại: user vừa đăng ký bị "không tồn tại" 10 phút.)

### Tài liệu kinh điển nên đọc
- **"Scaling Memcache at Facebook" (NSDI 2013)** — giáo trình thực chiến số một: lease chống stampede lẫn race, invalidation qua binlog (McSqueal), regional pool, cold cluster warm-up. Đọc kỹ hơn một lần.
- **"Optimal Probabilistic Cache Stampede Prevention" (Vattani et al., 2015)** — nền toán của early expiration (XFetch), ngắn và dùng được ngay.
- **Redis documentation: "Redis cluster specification" + "LRU/LFU eviction"** — hiểu công cụ mình vận hành ở mức spec, không ở mức tutorial.
- **"The Tail at Scale" (Dean & Barroso, 2013)** — vì sao p99 chi phối trải nghiệm và các kỹ thuật hedge — bối cảnh cho mọi quyết định cache/latency.
- **RFC 5861 (stale-while-revalidate)** — serve-stale chuẩn hóa ở tầng HTTP/CDN; pattern đáng mang vào cả cache nội bộ.
