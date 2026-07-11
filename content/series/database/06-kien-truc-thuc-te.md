+++
title = "Bài 6 — Kiến Trúc Thực Tế"
date = "2026-07-02T14:00:00+07:00"
draft = false
tags = ["backend", "database"]
series = ["Database Thực Chiến"]
+++

> **Cách đọc:** mỗi mục trả lời năm câu hỏi — chọn gì, vì sao, vì sao KHÔNG chọn phương án khác, scale/chi phí dự kiến, và rủi ro chính. Các con số là mức định cỡ (order of magnitude) để lập luận, không phải cam kết hiệu năng.

---

## 6.1. E-commerce

**Phân rã workload:** đơn hàng/thanh toán/tồn kho (transactional, bất biến chặt) · catalog sản phẩm (schema đa dạng, đọc nhiều) · hành vi người dùng + funnel (event, tỷ row) · giỏ hàng/session (nóng, TTL).

**Thiết kế tham chiếu:**

- **PostgreSQL — đơn hàng, thanh toán, tồn kho, khách hàng.** Vì: chống oversell cần constraint + serializable/`FOR UPDATE`; đối soát tiền cần ACID. Không chọn MongoDB cho phần này: bất biến "tồn kho ≥ 0" xuyên document phải tự cài ở app — đúng loại code dễ sai nhất. Không chọn ClickHouse: hiển nhiên (không transaction).
- **Catalog:** cả hai đường đều đúng tùy đội — PostgreSQL + JSONB (ít hệ hơn, JOIN được với tồn kho/giá) hoặc MongoDB (thuộc tính cực kỳ đa dạng, đội front-end tự chủ schema). Tiêu chí quyết: nếu catalog < chục triệu SKU và đội nhỏ → JSONB; marketplace trăm triệu SKU đa nguồn → MongoDB đáng cân nhắc.
- **ClickHouse — clickstream, funnel, recommendation features, báo cáo bán hàng.** Nạp qua event stream (Kafka), MV cho dashboard theo giờ/ngày.
- Redis cho giỏ hàng/session; search engine (Elasticsearch/OpenSearch/Typesense) cho tìm kiếm sản phẩm — full-text của PG đủ cho khởi đầu, đuối khi cần facet/typo-tolerance quy mô lớn.

**Scale/chi phí:** đến hàng chục nghìn đơn/ngày: 1 PG (RS) + 1 CH node nhỏ là đủ — đừng over-engineer. **Rủi ro chính:** chạy báo cáo trên PG primary (mục 9, Chương 2); đếm tồn kho bằng cache rồi lệch với PG.

## 6.2. FinTech

**Đặc thù:** đúng tuyệt đối > mọi thứ khác; audit trail bất biến; đối soát; regulator.

- **PostgreSQL là trung tâm không bàn cãi** — ledger kiểu double-entry, append-only, mọi biến động tiền là transaction Serializable (hoặc Read Committed + khóa tường minh được review kỹ). `synchronous_commit=on` + sync replica (RPO=0) cho ledger: mất giao dịch đã báo thành công là sự cố cấp tồn vong, đáng trả giá latency.
- **ClickHouse** — phân tích giao dịch, phát hiện gian lận (feature aggregation trên lịch sử lớn), báo cáo regulator trên hàng tỷ bản ghi. Nạp qua CDC từ PG; **CH không bao giờ là source of truth của tiền**.
- **MongoDB** — vai phụ nếu có: hồ sơ KYC document đa dạng, log tương tác. Nhiều fintech bỏ qua hoàn toàn.
- **Vì sao không MongoDB cho ledger:** multi-document transaction tồn tại nhưng toàn bộ văn hóa engine (schema tự do, tunable consistency) đi ngược yêu cầu "không có đường tắt để sai". Chọn công cụ mà cách dùng *mặc định* là cách dùng *an toàn*.

**Rủi ro chính:** idempotency của API tiền (retry nhân đôi giao dịch — Chương 1, mục 1.11); phân trang/đối soát trên replica bị lag đọc thiếu giao dịch.

## 6.3. Social Network

**Phân rã:** đồ thị quan hệ + nội dung (post, comment) · feed (đọc cực nhiều, fan-out) · engagement analytics (like/view hàng tỷ).

- Khởi đầu và đến quy mô vừa: **PostgreSQL** cho user/post/follow (quan hệ là bản chất), Redis cho feed cache + counter, **ClickHouse** cho engagement/impression analytics.
- **MongoDB** hợp lý cho nội dung dạng document (post + media metadata + comment embed theo pattern subset) khi ghi nội dung cần scale-out theo user.
- Scale lớn (trăm triệu user): không database đơn lẻ nào "giải" social — kiến trúc chuyển sang precomputed feed (fan-out-on-write vào KV store), sharding theo user, và đồ thị chuyên dụng nếu query đồ thị sâu (mutual friends bậc 2+ trên RDBMS là scatter tự sát). Bài học: **đừng bắt database gánh việc của kiến trúc**.

**Rủi ro:** counter (like/view) ghi trực tiếp vào PG từng cái một — đúng bài là gom qua Redis/stream rồi flush batch; hot partition quanh celebrity trong mọi hệ sharded.

## 6.4. SaaS (B2B, multi-tenant)

- **PostgreSQL gần như mặc định.** Ba mô hình tenant: schema-per-tenant (mạnh về cô lập, chết ở nghìn tenant vì catalog phình + migration N lần), **row-based + tenant_id + Row-Level Security** (mặc định đúng cho đa số), database-per-tenant (chỉ cho enterprise tier). JSONB gánh custom fields per-tenant — nhu cầu SaaS kinh điển.
- **ClickHouse** khi bán tính năng analytics cho khách (usage dashboard, audit log search): bảng ORDER BY (tenant_id, ts) + row policy; đây chính xác là mẫu multi-tenant analytics mà CH sinh ra để làm.
- **MongoDB** nếu sản phẩm bản chất là document platform (CMS, form builder, no-code) — dữ liệu người dùng định nghĩa lúc runtime là chỗ schema-on-read thắng thật sự.

**Rủi ro:** noisy neighbor (một tenant to làm chậm tất cả — cần quota tầng app + CH settings profile); quên tenant_id trong một query = rò dữ liệu chéo khách hàng (RLS là lưới an toàn đáng bật dù app đã filter).

## 6.5. Messaging Platform

**Đặc thù:** ghi rất nhiều, gần như append-only; đọc theo conversation gần đây; fan-out online; lịch sử lớn dần vô hạn.

- Quy mô nhỏ–vừa: **PostgreSQL** (messages partition theo thời gian) + Redis pub/sub — đủ đến hàng triệu tin/ngày.
- Quy mô lớn: mẫu ngành là **Cassandra/ScyllaDB** cho message store (LSM write-optimized, partition theo conversation — Discord là case study công khai kinh điển). MongoDB là lựa chọn trung gian hợp lý (bucket pattern theo conversation+ngày, sharding theo conversation).
- **ClickHouse:** analytics vận hành (volume, delivery latency, abuse detection) — không phải message store (đọc theo conversation = point-ish read).
- **Vì sao không PG ở scale lớn:** ghi append tần suất cực cao × index × WAL × vacuum trên bảng vĩnh viễn phình — đúng tổ hợp điểm yếu cấu trúc.

## 6.6. Ride Sharing / Delivery

- **PostgreSQL + PostGIS** — chuyến đi, thanh toán, driver/rider (transactional + geo hạng nhất). Matching real-time: vị trí driver là dữ liệu *nóng, phù du* → Redis (geo set) hoặc in-memory grid, KHÔNG phải PG (update vị trí 5s/lần × trăm nghìn driver = bão UPDATE vào đúng chỗ PG yếu — bloat).
- **ClickHouse** — toàn bộ telemetry GPS lịch sử (tỷ điểm/ngày), phân tích supply-demand, pricing, ETA model features.
- Mẫu chung đáng nhớ: **phân biệt dữ liệu theo vòng đời** — trạng thái hiện tại (Redis) ≠ giao dịch (PG) ≠ lịch sử phân tích (CH). Nhét cả ba vào một hệ là nguồn sự cố.

## 6.7. Blockchain Infrastructure

- Bản thân chain là ledger riêng; database vào vai **indexer**: decode block → **PostgreSQL** cho trạng thái truy vấn được (số dư, token, NFT ownership — cần đúng tuyệt đối, reorg phải rollback được → transaction!), **ClickHouse** cho phân tích on-chain (dòng tiền, hoạt động địa chỉ trên hàng tỷ transfer).
- Đặc thù đáng chú ý: **chain reorg** = dữ liệu "bất biến" hóa ra đảo được vài block cuối → indexer phải thiết kế theo block height + đánh dấu canonical, và đây là nơi transaction của PG cứu mạng.

## 6.8. AI Platform

- **PostgreSQL + pgvector** — metadata (dataset, run, experiment, user, billing) + vector search đến hàng chục triệu vector; một hệ phủ cả metadata lẫn RAG khởi đầu là lợi thế vận hành lớn. Vector DB chuyên dụng (Qdrant/Milvus...) khi trăm triệu vector + filter phức tạp + recall/latency khắt khe.
- **ClickHouse** — LLM observability (log request/response, token, latency, cost): volume lớn, append-only, aggregate theo model/user/ngày — bài mẫu của CH; nhiều nền tảng LLM-observability thương mại xây đúng trên CH.
- **MongoDB** — document store cho prompt/conversation nếu cấu trúc hội thoại đa dạng và truy cập theo session; JSONB thường đủ.

## 6.9. Real-time Analytics (sản phẩm phân tích cho người dùng)

- **ClickHouse là vai chính hiển nhiên** — đây là chính xác bài toán Metrica gốc: ingest liên tục, dashboard dưới giây, ad-hoc filter. Kafka → CH, MV nhiều tầng, quota per-tenant.
- **PostgreSQL** giữ vai metadata (định nghĩa dashboard, user, saved query). **MongoDB** không có vai tự nhiên.
- Cạnh tranh trong hạng mục: Apache Druid/Pinot (điểm mạnh: concurrency truy vấn rất cao kiểu ứng dụng đối mặt người dùng, ingest streaming per-event) — CH thắng ở SQL đầy đủ + vận hành đơn giản hơn + linh hoạt ad-hoc; Druid/Pinot đáng xem khi cần chục nghìn query/s latency thấp trên pattern cố định.

## 6.10. Event Tracking Platform (product analytics kiểu Amplitude/PostHog)

- **ClickHouse làm lõi** (PostHog công khai kiến trúc này): bảng events ORDER BY (team_id, ts), funnel/retention bằng hàm chuyên dụng (`windowFunnel`, `retention`), ReplacingMergeTree cho person profile.
- **PostgreSQL** — định nghĩa project/API key/feature flag.
- Bài học từ chính lịch sử PostHog: bản đầu chạy trên PG thuần — chết ở chục triệu event/ngày đúng như phân tích cấu trúc Chương 4, mục 1; migration sang CH là bước ngoặt sản phẩm. Đây là minh chứng sống cho "PG làm được đến khi không làm được nữa".

## 6.11. Logging Platform

- **ClickHouse** cho log tập trung: nén 10–30× trên log text lặp (log giống nhau đến kinh ngạc), TTL + tiered S3, tìm kiếm bằng skipping index (tokenbf) + brute-force scan vẫn nhanh nhờ columnar. So với Elasticsearch: CH rẻ hơn nhiều lần về storage (không inverted index cho mọi field), thắng ở aggregate; ES thắng ở full-text tinh vi/relevance. Xu hướng ngành (Uber, Cloudflare đều công bố) đang nghiêng về CH-based cho log volume lớn.
- PG/Mongo: không có vai — đừng bao giờ đổ log ứng dụng volume lớn vào OLTP database.

## 6.12. Data Warehouse nội bộ

- **ClickHouse** là ứng viên khi: đội chịu được vận hành (hoặc dùng Cloud), mô hình dữ liệu kiểm soát được (denormalize/star gọn), ưu tiên tốc độ + chi phí.
- Snowflake/BigQuery/Databricks thắng khi: đội data thuần phân tích không muốn vận hành, join hình sao nặng tùy tiện, elastic theo mùa, hệ sinh thái governance. Trung thực mà nói: **warehouse thương mại mua sự tiện bằng tiền; CH mua hiệu năng/chi phí bằng kỷ luật kỹ thuật** — chọn theo cấu trúc đội, không chỉ theo benchmark.
- Nguồn: CDC từ PG/Mongo (Debezium → Kafka → CH) hoặc ELT batch. PG cũng có thể *là* warehouse cho công ty nhỏ (đến trăm GB, dbt + PG là stack hoàn toàn tử tế — đừng mua Snowflake vì slide đẹp).

---

## 6.13. Các mẫu xuyên suốt (tổng kết 12 mục)

1. **Source of truth transactional → PostgreSQL** trong 11/12 kiến trúc — không phải vì "PG tốt nhất" mà vì phần tiền/trạng thái nghiệp vụ luôn tồn tại và luôn cần bảo đảm mạnh nhất.
2. **Dữ liệu event/append-only volume lớn → ClickHouse**, chảy vào qua stream/CDC, một chiều.
3. **MongoDB xuất hiện khi và chỉ khi** dữ liệu là document tự nhiên + truy cập một trục + (schema động hoặc cần scale-out ghi tích hợp). Vai của nó là *lựa chọn có điều kiện*, không phải mặc định.
4. **Redis/cache luôn có mặt** cho trạng thái nóng phù du — nhận diện loại dữ liệu này sớm giúp PG/Mongo thoát khỏi các bão UPDATE vô nghĩa.
5. **Ranh giới hệ thống = ranh giới bảo đảm:** khi dữ liệu rời PG sang CH, nó rời vùng "đúng tuyệt đối" sang vùng "đúng dần" — thiết kế phải nói rõ điều đó với downstream (dashboard lệch vài giây/phút là bình thường và phải được chấp nhận tường minh).

---

# Chương 6B — Decision Framework tổng hợp

## B.1. Cây quyết định chính

```
Bắt đầu: dữ liệu/tính năng mới cần chỗ ở
│
├─ Q1. Có bất biến nghiệp vụ mà vi phạm là mất tiền/mất uy tín?
│      (số dư, tồn kho, booking, ledger)
│      └─ CÓ → PostgreSQL. Dừng. (Sức mạnh constraint+transaction là độc quyền)
│
├─ Q2. Workload là append-mostly + câu hỏi dạng aggregate trên nhiều row?
│      (event, log, metric, telemetry)
│      └─ CÓ → Volume dự kiến?
│              ├─ < ~100 triệu row, dashboard nội bộ → PostgreSQL (partition+BRIN) đủ
│              └─ Lớn hơn / dashboard dưới giây / giữ nhiều năm → ClickHouse
│
├─ Q3. Document tự nhiên + truy cập một trục + (schema động | cần shard ghi)?
│      └─ CÓ → MongoDB (với kỷ luật: validation bật, shard key mô phỏng trước,
│              embed có giới hạn)
│      └─ CÓ nhưng volume nhỏ & đội đã giỏi PG → PostgreSQL + JSONB
│
├─ Q4. Trạng thái nóng, phù du, TTL ngắn? (session, vị trí, counter, cache)
│      └─ CÓ → Redis/KV — đừng để nó chạm database bền
│
└─ Không rơi vào đâu rõ ràng → PostgreSQL làm mặc định,
   thu thập số liệu thật, di cư khi có bằng chứng cấu trúc (không phải cảm giác)
```

## B.2. Bảng điều kiện — đối chiếu nhanh

| Nếu workload có đặc điểm... | Thì... | Vì (gốc kỹ thuật) |
|---|---|---|
| Transaction đa bảng, bất biến chặt, đối soát | **PostgreSQL, bắt buộc** | ACID + constraint + serializable (Ch.2 §3.2, Ch.1 §1.4) |
| JOIN đa chiều, query đa dạng chưa đoán trước | PostgreSQL | Planner + hệ index giàu nhất (Ch.2 §3.5–3.6) |
| Schema biến động nhanh, document một trục | MongoDB (hoặc PG+JSONB nếu nhỏ) | Document = đơn vị atomic + I/O (Ch.3 §3.1) |
| Ghi vượt trần 1 máy, chấp nhận trói query vào shard key | MongoDB sharded (hoặc Citus) | Sharding tích hợp (Ch.3 §3.6) |
| Hàng trăm triệu → nghìn tỷ row, aggregate/filter, append | **ClickHouse** | Columnar × nén × vectorized (Ch.4 §3) |
| Cần transaction mạnh | **Tuyệt đối tránh ClickHouse** | Không có, theo thiết kế (Ch.4 §5) |
| Update/delete thường xuyên theo nghiệp vụ | Tránh ClickHouse; cẩn trọng cả Mongo unbounded-array | Immutable part / document rewrite (Ch.4 §3.5, Ch.3 §3.1) |
| Đọc điểm latency thấp, key-value | Redis/KV; PG/Mongo được; tránh CH | Granule 8192 row (Ch.4 §3.3) |
| Log/metrics tập trung | ClickHouse | Nén text lặp + TTL + scan nhanh (Ch.6 §6.11) |
| Đội 5 người, sản phẩm chưa có product-market fit | PostgreSQL cho (gần như) tất cả | Chi phí vận hành đa hệ > mọi lợi ích ở cỡ này |

## B.3. Quy trình ra quyết định cho architect (checklist)

1. **Viết ra 5–10 query/thao tác quan trọng nhất** với tần suất và SLA — không chọn database trước khi làm bước này.
2. **Định cỡ trung thực:** row/ngày, working set, tỷ lệ đọc/ghi, độ trễ chấp nhận. Nhân 10 cho 2 năm — nhưng đừng nhân 1000 "cho chắc" (thiết kế cho scale không có thật là mua độ phức tạp bằng tiền thật).
3. **Đối chiếu bảng B.2** → thường ra 1–2 ứng viên.
4. **Tìm lý do KHÔNG chọn** (mục 5 và 10 của Chương 2–4) — điểm yếu cấu trúc nào của ứng viên chạm đúng workload mình?
5. **PoC bằng dữ liệu thật, cỡ thật** (hoặc sinh giả đúng phân bố) cho 3 query xấu nhất — benchmark của người khác không thay được của mình.
6. **Tính TCO cả người:** ai trực hệ này 3 giờ sáng? Managed service đáng giá bao nhiêu cho đội mình?
7. **Thiết kế đường lui:** dữ liệu vào bằng event/CDC được không (để sau này đổi engine đích không phải viết lại nguồn)? Đây là quyết định kiến trúc rẻ nhất hôm nay, đắt nhất nếu bỏ qua.

## B.4. Ba câu "khắc cốt" thay lời kết

1. **PostgreSQL là câu trả lời mặc định; gánh nặng chứng minh thuộc về hệ thay thế.**
2. **ClickHouse nhanh vì nó từ chối làm những việc khác — đừng bao giờ giao cho nó việc nó đã từ chối.**
3. **MongoDB thưởng cho đội có kỷ luật modeling và trừng phạt đội không có — schema linh hoạt nghĩa là schema sống trong đầu đội bạn.**
