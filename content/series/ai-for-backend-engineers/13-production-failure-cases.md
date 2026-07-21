+++
title = "Chương 13 — Production Failure Cases: 13 sự cố kinh điển"
date = "2026-07-18T09:10:00+07:00"
draft = false
tags = ["backend", "ai", "llm"]
series = ["AI cho Backend Engineer"]
+++

Chương này là sổ tay trực chiến (runbook). Mỗi case theo khung: **Triệu chứng → Root Cause → Kiến trúc bị ảnh hưởng → Metric/Dashboard/Alert → Điều tra → Khắc phục → Phòng tránh**. Dùng để: (1) tra cứu khi đang cháy, (2) diễn tập game day, (3) checklist review kiến trúc trước khi launch.

---

## Case 1 — Hallucination

- **Triệu chứng**: chatbot trả lời tự tin về chính sách không tồn tại; khách hàng khiếu nại "AI của các anh hứa hoàn tiền 30 ngày"; đôi khi kèm citation nhìn rất thật nhưng trỏ sai nguồn.
- **Root cause**: model bịa khi (a) context thiếu thông tin mà prompt không cho đường lui "không biết", (b) retrieval trả chunk sai/không liên quan, (c) câu hỏi ngoài phạm vi nhưng không bị chặn, (d) temperature cao cho task factual.
- **Kiến trúc bị ảnh hưởng**: RAG pipeline (retrieval → prompt → generation), guardrail layer.
- **Metric**: groundedness score (online judge, sampled), citation accuracy, tỷ lệ câu trả lời không citation, thumbs-down rate, retrieval score trung bình của top-k.
- **Dashboard**: quality dashboard — groundedness trend theo prompt_version; phân phối retrieval similarity.
- **Alert**: groundedness p50 giảm > X% so với baseline 7 ngày; thumbs-down spike.
- **Điều tra**: lấy các trace bị flag → xem chunks được retrieval: đúng tài liệu không? → nếu retrieval đúng mà câu trả lời sai: vấn đề prompt/model; nếu retrieval sai: sang Case 4; kiểm tra câu hỏi có ngoài phạm vi KB không.
- **Khắc phục**: siết grounding prompt (bắt buộc citation, đường lui "không tìm thấy"); bật groundedness check chặn/gắn cảnh báo; temperature về 0 cho luồng factual; thêm out-of-scope detection.
- **Phòng tránh**: golden dataset chứa case "không có trong tài liệu" (kỳ vọng: từ chối trả lời); eval groundedness trong CI; giáo dục người dùng bằng UI (citation + disclaimer).

## Case 2 — Context Overflow

- **Triệu chứng**: lỗi 400 `context_length_exceeded` rải rác; hoặc tệ hơn — không lỗi nhưng model "quên" chỉ dẫn trong system prompt ở các hội thoại dài (truncation lặng lẽ ở một tầng nào đó).
- **Root cause**: history không cắt; user paste tài liệu khổng lồ; RAG top_k cộng dồn; tool result to (JSON 50KB); tổng các phần vượt limit — thường do **không có một điểm duy nhất chịu trách nhiệm đếm token toàn prompt**.
- **Kiến trúc**: session management, context assembly, tool executor.
- **Metric**: phân phối input_tokens (watch p99), tỷ lệ 400 theo loại, độ dài history theo session, kích thước tool result.
- **Dashboard**: token dashboard — input tokens theo feature, top session dài nhất.
- **Alert**: p99 input_tokens > 80% context limit; error `context_length` > 0.1%.
- **Điều tra**: trace request lỗi → bóc từng phần prompt (system/history/RAG/tool) xem phần nào phình → thường tìm thấy một nguồn không bị giới hạn.
- **Khắc phục**: token budgeter trung tâm — phân bổ trần cho từng phần, cắt theo ưu tiên (giữ system + lượt cuối, nén phần giữa); giới hạn kích thước tool result và tài liệu paste.
- **Phòng tránh**: budget enforcement là middleware bắt buộc trước mọi LLM call; test hội thoại 100 lượt trong CI; chọn limit "mềm" nội bộ thấp hơn limit cứng của provider.

## Case 3 — Prompt Injection khai thác thành công

- **Triệu chứng**: hệ thống hành xử lạ theo hướng có chủ đích: tiết lộ system prompt, gửi dữ liệu ra ngoài, tool bị gọi bất thường; đôi khi phát hiện qua honeytoken hoặc báo cáo của user.
- **Root cause**: chỉ dẫn độc trong input/dữ liệu gián tiếp (email, web, tài liệu) + tool có quyền vượt user + thiếu output/egress filter (xem Chương 12).
- **Kiến trúc**: toàn bộ đường văn bản vào context; tool executor; output rendering.
- **Metric**: injection-classifier hit rate theo nguồn; tool call bất thường (tool × user × tần suất); egress đến domain lạ trong output.
- **Dashboard**: security dashboard — guardrail hits, tool call anomaly, honeytoken.
- **Alert**: honeytoken xuất hiện ở bất kỳ đâu; chuỗi tool call nguy hiểm không qua confirm; injection hit spike.
- **Điều tra**: coi là **security incident** (không phải bug thường): bảo toàn trace, xác định payload vào bằng đường nào, phạm vi phiên/tenant bị ảnh hưởng, dữ liệu nào đã ra ngoài.
- **Khắc phục**: kill switch tính năng liên quan; thu hồi phiên/token; vá đường vào (sanitize nguồn đó, hạ quyền tool); thông báo theo nghĩa vụ compliance.
- **Phòng tránh**: kiến trúc blast-radius-nhỏ (Chương 12 mục 4); red team corpus chạy regression; least privilege tool theo user; egress whitelist.

## Case 4 — Vector Search Recall thấp

- **Triệu chứng**: "hỏi đúng nội dung có trong tài liệu mà bot bảo không tìm thấy"; chất lượng RAG tệ dù model tốt; tỷ lệ "không biết" cao bất thường.
- **Root cause**: chunking cắt vỡ ngữ cảnh; query và document lệch dạng diễn đạt (câu hỏi hội thoại vs văn bản hành chính) mà không có query rewrite; thiếu hybrid search (miss keyword hiếm); embedding model yếu với tiếng Việt; filter ACL hậu lọc nuốt kết quả; index suy giảm sau nhiều delete.
- **Kiến trúc**: ingestion pipeline (chunking/embedding), query path (rewrite/hybrid/rerank), vector DB.
- **Metric**: recall@k trên canary queries (chạy định kỳ!), phân phối similarity score của top-1, tỷ lệ "không tìm thấy", tỷ lệ câu trả lời có citation về đúng doc kỳ vọng.
- **Dashboard**: retrieval dashboard — recall trend, similarity distribution, queries similarity thấp (mỏ vàng để cải thiện).
- **Alert**: canary recall giảm dưới ngưỡng; "không tìm thấy" spike sau một lần re-index.
- **Điều tra**: lấy 20 câu fail → chạy tay: embedding của query gần chunk nào? chunk chứa đáp án có tồn tại trong index không (ingestion sót?) → có tồn tại nhưng không match: thử BM25 có ra không (thiếu hybrid) → xem chunk đó có "tự đứng được" không (chunking tệ).
- **Khắc phục**: theo chẩn đoán — bổ sung hybrid, bật query rewrite, sửa chunking + re-index, đổi embedding model (re-index toàn bộ), sửa pre-filter.
- **Phòng tránh**: golden retrieval set từ ngày đầu; mọi thay đổi ingestion qua recall check; đo riêng retrieval khỏi generation — đừng chỉ đo điểm cuối.

## Case 5 — Token Cost tăng đột biến

- **Triệu chứng**: hóa đơn tháng ×4; hoặc alert ngân sách nổ giữa tháng.
- **Root cause** (theo tần suất gặp): vòng lặp retry/agent không giới hạn; tính năng mới quên tối ưu (gửi cả history, top_k=20); một tenant/kẻ lạm dụng chạy script; prompt version mới dài gấp đôi; bug gửi trùng request; provider đổi giá hoặc alias model trỏ sang model đắt.
- **Kiến trúc**: AI gateway (metering, quota), retry logic, agent runtime.
- **Metric**: cost/ngày theo feature/tenant/model; cost/request trend; retry rate; tokens/request p99.
- **Dashboard**: cost dashboard với breakdown đủ chiều — **khả năng trả lời "tiền tăng ở đâu" trong 5 phút là tiêu chí thiết kế**.
- **Alert**: cost ngày > 150% MA7; cost một tenant > ngưỡng; retry rate > 10%; budget tháng chạm 80%.
- **Điều tra**: breakdown theo chiều (feature → tenant → prompt_version → thời điểm bắt đầu) → khớp thời điểm với deploy/config change → xem trace các request đắt nhất.
- **Khắc phục**: tắt/khoanh vùng nguồn (kill feature flag, khóa tenant lạm dụng, rollback prompt); hard cap tạm thời.
- **Phòng tránh**: hard cap chi tiêu ở gateway; budget + max steps cho agent; quota theo tenant; canary cost check khi deploy prompt mới (cost là metric của eval, không chỉ quality).

## Case 6 — Latency cao

- **Triệu chứng**: người dùng phàn nàn "AI chậm"; p95 TTFT từ 1.5s lên 6s; timeout tăng.
- **Root cause**: provider degradation (phổ biến — kiểm tra trước tiên); input token phình (prompt/history/RAG); routing sang model chậm hơn (fallback đang kích hoạt mà không ai để ý); retrieval chậm (index thiếu RAM, filter nặng); mất prompt cache hit (đổi cấu trúc prompt làm phần tĩnh không còn đứng đầu); cold start (self-host).
- **Kiến trúc**: toàn đường request — cần trace phân rã mới biết đoạn nào.
- **Metric**: TTFT/total p50-p95-p99 **phân rã theo span** (retrieval, rerank, LLM) và theo provider/model; input_tokens trend; cache hit rate; fallback rate.
- **Dashboard**: latency waterfall theo giai đoạn; so sánh theo model/provider.
- **Alert**: p95 TTFT vượt SLO 10 phút liên tục; fallback rate > 5% (thường là thủ phạm giấu mặt).
- **Điều tra**: trace 10 request chậm nhất → span nào ăn thời gian? → LLM span chậm: so sánh cùng model hôm qua (provider issue?) + so input_tokens (prompt phình?) + kiểm tra model thực sự được dùng (fallback?).
- **Khắc phục**: theo chẩn đoán — ép routing về provider khỏe, nén prompt, sửa cache, thêm RAM/replica cho vector DB; truyền thông trạng thái cho user (streaming + skeleton UI giảm cảm nhận chờ).
- **Phòng tránh**: SLO + benchmark định kỳ từng provider từ chính hạ tầng của bạn; budget token là budget latency; load test có thành phần LLM giả lập độ trễ thật.

## Case 7 — Rate Limit từ Provider

- **Triệu chứng**: 429 hàng loạt vào giờ cao điểm; tính năng AI chết cục bộ trong khi hệ thống khác bình thường.
- **Root cause**: traffic vượt TPM/RPM của tier hiện tại; retry không backoff khuếch đại (retry storm); một batch job nội bộ nuốt quota của interactive traffic; nhiều team chung một key không có điều phối.
- **Kiến trúc**: AI gateway (outbound throttling, priority queue), batch pipeline.
- **Metric**: 429 rate theo provider/key; usage/limit ratio (%TPM đã dùng); queue depth theo priority.
- **Dashboard**: quota dashboard — usage vs limit từng provider/key/region theo giờ.
- **Alert**: usage > 80% limit; 429 > 1%.
- **Điều tra**: 429 đến từ key nào, feature nào → có job batch nào đang chạy giờ cao điểm? → retry storm? (đồ thị request tăng theo lũy thừa sau lỗi đầu).
- **Khắc phục**: bật priority queue (interactive trước, batch hoãn); backoff + honor Retry-After; trải batch sang giờ thấp điểm / Batch API; xin nâng tier; thêm key/region/provider để cộng quota.
- **Phòng tránh**: outbound rate limiter dưới trần provider; tách quota interactive/batch từ thiết kế; capacity planning theo token trước mỗi launch lớn.

## Case 8 — Model/Provider Downtime

- **Triệu chứng**: 5xx/timeout đồng loạt từ một provider; status page của họ đỏ (hoặc chưa đỏ nhưng bạn đã chết).
- **Root cause**: sự cố phía provider — nằm ngoài kiểm soát; điều nằm trong kiểm soát là hệ của bạn có chịu được không.
- **Kiến trúc**: circuit breaker, fallback chain, degradation path.
- **Metric**: error rate theo provider; circuit breaker state; fallback success rate; health check tổng hợp.
- **Dashboard**: provider health — error/latency từng provider, trạng thái breaker.
- **Alert**: breaker mở; error provider > 5% trong 5 phút (đừng chờ status page của họ).
- **Điều tra**: xác nhận phạm vi (một model hay cả provider? một region?) → kiểm tra fallback có tự kích hoạt như thiết kế → nếu không: vì sao (chưa cấu hình cho luồng này? prompt không tương thích model fallback?).
- **Khắc phục**: ép route sang provider dự phòng; luồng không có fallback → bật degraded mode (thông báo trung thực + đường nghiệp vụ thay thế: form, hàng đợi người xử lý).
- **Phòng tránh**: fallback đa provider cho luồng quan trọng, **test định kỳ bằng game day** (fallback chưa từng chạy = không có fallback); mọi tính năng AI có degradation path được thiết kế từ đầu (Chương 10 bài học 4).

## Case 9 — Memory Growth (session & context phình)

- **Triệu chứng**: RAM Redis/session store tăng tuyến tính không giảm; chi phí mỗi request của user lâu năm cao gấp nhiều lần user mới; hội thoại rất dài bắt đầu chậm và tệ đi.
- **Root cause**: session không TTL; history không nén; "memory" của agent ghi mãi không có chiến lược quên; mỗi request mang theo toàn bộ quá khứ.
- **Kiến trúc**: session store, memory subsystem của agent.
- **Metric**: kích thước session store + tăng trưởng; phân phối độ dài history; input_tokens theo tuổi session.
- **Dashboard**: session dashboard — top session lớn nhất, tăng trưởng storage.
- **Alert**: storage tăng > X%/tuần; session vượt ngưỡng kích thước.
- **Điều tra**: top session lớn → vì sao không bị nén/cắt? → luồng nào tạo session không TTL?
- **Khắc phục**: áp TTL + nén (summary) hồi tố; migration dọn session chết.
- **Phòng tránh**: TTL và trần kích thước là thuộc tính bắt buộc của schema session ngay từ đầu; chiến lược nén history theo thang (Chương 08); "quên" là tính năng của memory, không phải bug.

## Case 10 — Cache Miss / Cache sai

- **Triệu chứng A (miss)**: cost và latency cao hơn kỳ vọng dù "đã có cache"; hit rate ~0.
- **Triệu chứng B (sai — nguy hiểm hơn)**: user nhận câu trả lời của người khác/của phiên bản chính sách cũ; semantic cache trả lời câu gần giống nhưng ý khác.
- **Root cause A**: cache key chứa thành phần biến thiên (timestamp, session_id, thứ tự field JSON không chuẩn hóa); prompt cache của provider mất hiệu lực vì phần động chèn lên đầu prompt.
- **Root cause B**: key thiếu chiều (thiếu user/tenant → lộ chéo; thiếu prompt_version/doc_version → serve đồ cũ); ngưỡng semantic cache lỏng.
- **Kiến trúc**: cache layer các tầng (exact, semantic, provider prompt cache).
- **Metric**: hit rate theo tầng; false-hit rate (đo bằng eval sample trên cache hit); tuổi cache entry được serve.
- **Dashboard**: cache dashboard — hit/miss/latency saved/cost saved theo tầng.
- **Alert**: hit rate rơi đột ngột (ai đó đổi cấu trúc prompt); bất kỳ báo cáo lộ chéo nào = sự cố nghiêm trọng.
- **Điều tra**: log key được build từ gì → so 2 request "đáng lẽ hit" khác nhau ở thành phần nào; với false-hit: truy vết key thiếu chiều nào.
- **Khắc phục**: chuẩn hóa key builder (một hàm duy nhất toàn hệ thống); flush cache bẩn; siết ngưỡng semantic hoặc tắt.
- **Phòng tránh**: key schema có kiểm soát (bắt buộc gồm: model, prompt_version, tenant, doc_version, normalized input); cache invalidation gắn vào pipeline deploy prompt và re-index; test isolation giữa tenant trong CI.

## Case 11 — Embedding Drift / lệch không gian vector

- **Triệu chứng**: sau một thay đổi "vô hại", search tệ dần hoặc tệ ngay: kết quả không liên quan, recall canary rơi; đặc biệt sau khi (a) nâng version embedding model, (b) thêm dữ liệu mới, (c) đổi provider embedding.
- **Root cause**: query embed bằng model X, một phần document embed bằng model Y — hai không gian vector không so sánh được; hoặc cùng model nhưng khác version/normalization; hoặc phân phối nội dung mới lệch xa dữ liệu cũ làm tham số index (IVF centroids) không còn phù hợp.
- **Kiến trúc**: ingestion pipeline, vector DB, embedding service.
- **Metric**: canary recall; embedding model version gắn trên **từng vector** (metadata); phân phối similarity trước/sau thay đổi.
- **Dashboard**: index health — số vector theo model_version (nhìn thấy ngay tình trạng "nửa nọ nửa kia").
- **Alert**: tồn tại > 1 model_version trong một collection đang phục vụ; canary recall rơi sau ingestion job.
- **Điều tra**: query metadata đếm vector theo version → tìm job/worker nào dùng version khác (config lệch giữa các worker là kinh điển).
- **Khắc phục**: re-embed phần lệch về đúng version (từ text gốc — đây là lý do phải lưu text); nếu chuyển model mới: dual-index, backfill, switch nguyên tử.
- **Phòng tránh**: embedding version nằm trong tên collection hoặc metadata bắt buộc + gateway từ chối upsert lệch version; quy trình đổi model = quy trình migration có checklist; luôn giữ khả năng rebuild từ nguồn (Chương 06).

## Case 12 — Sai sót do dữ liệu lỗi thời (Stale Knowledge)

- **Triệu chứng**: AI trả lời đúng theo... chính sách năm ngoái; giá/điều khoản đã đổi nhưng bot vẫn nói số cũ; đôi khi cả citation cũng "đúng" — trỏ về tài liệu cũ chưa bị gỡ.
- **Root cause**: pipeline đồng bộ nguồn → index không bắt sự kiện sửa/xóa (chỉ ingest tài liệu mới); tài liệu cũ không có vòng đời (không ai gỡ bản deprecated); cache trả lời cũ (Case 10); hoặc câu trả lời từ kiến thức train của model thay vì từ tài liệu (grounding hỏng).
- **Kiến trúc**: ingestion sync (CDC/webhook), document lifecycle, cache.
- **Metric**: index freshness lag (thời gian từ khi nguồn đổi đến khi index đổi); số vector mồ côi (nguồn đã xóa); tuổi tài liệu được cite.
- **Dashboard**: freshness dashboard — lag theo nguồn, tài liệu quá hạn review.
- **Alert**: freshness lag > SLO (ví dụ 1 giờ cho chính sách, 1 ngày cho wiki); citation về tài liệu đã deprecated.
- **Điều tra**: từ câu trả lời sai → citation trỏ doc nào, version nào → doc đó trong nguồn đã đổi chưa → sự kiện đổi có phát ra không, worker có xử lý không.
- **Khắc phục**: re-index tài liệu liên quan ngay; purge cache theo doc_version; gỡ bản deprecated khỏi index.
- **Phòng tránh**: đồng bộ theo sự kiện (create/update/**delete**) chứ không chỉ crawl thêm; metadata `effective_date`/`superseded_by` và filter lúc retrieval; cache key chứa doc_version; quy trình quản trị nội dung (owner, review date) — vấn đề này 50% là quy trình, không phải công nghệ.

## Case 13 — Tool Calling thất bại

- **Triệu chứng**: agent/assistant báo "đã làm xong" nhưng không có gì xảy ra; hoặc lặp đi lặp lại một tool lỗi; hoặc gọi tool không tồn tại; hoặc điền tham số bịa (order_id tự chế).
- **Root cause**: schema/mô tả tool mơ hồ → model dùng sai; lỗi tool bị nuốt hoặc trả về không đủ thông tin để model tự sửa; model bịa tên tool/tham số (xác suất, xảy ra ở mọi model); API đích đổi contract mà mô tả tool không cập nhật; thiếu idempotency khi model gọi trùng.
- **Kiến trúc**: tool executor, agent loop, contract giữa tool schema và API thật.
- **Metric**: tool call success rate theo tool; tỷ lệ "unknown tool"/"invalid params"; số vòng lặp trung bình/p95; tỷ lệ hành động được verify độc lập sau khi model báo thành công.
- **Dashboard**: tool dashboard — success/error theo tool, top error message, loop depth.
- **Alert**: success rate một tool rơi (thường do API đích đổi); loop depth p95 tăng; unknown-tool spike sau deploy (mô tả tool bị đổi/mất).
- **Điều tra**: trace các loop fail → đọc lỗi tool trả về: model có đủ thông tin để sửa không? → tham số sai kiểu gì (bịa? hiểu nhầm format?) → so schema tool với API thật.
- **Khắc phục**: sửa mô tả/schema tool (thêm ví dụ, format, ràng buộc); trả lỗi có hướng dẫn ("order_id dạng ORD-xxxxx, hãy hỏi user nếu chưa có"); thêm validation trước khi thực thi; verify độc lập kết quả hành động quan trọng.
- **Phòng tránh**: contract test giữa tool schema và API đích trong CI; eval bộ hội thoại tool-calling; idempotency key mọi tool ghi; giới hạn vòng lặp + budget (Case 5 chờ sẵn nếu không).

---

## Meta-bài học từ 13 case

1. **Hầu hết sự cố AI không có 5xx** — chúng là "hệ thống chạy ngon, kết quả sai/đắt/chậm". Alert phải đứng trên quality/cost/freshness metrics, không chỉ error rate.
2. **Trace chi tiết là điều kiện điều tra** — mọi case ở trên đều bắt đầu bằng "lấy trace ra xem". Không trace = mù.
3. **Các case móc xích nhau**: retry storm (7) gây cost spike (5); prompt đổi (11 của prompt) gây cache miss (10) gây latency (6). Điều tra nên nhìn timeline thay đổi toàn hệ thống.
4. **Phòng tránh rẻ hơn khắc phục** ở mọi case — và phần "phòng tránh" của cả 13 case gộp lại chính là nội dung các Chương 05–12. Đó không phải trùng hợp.

---

**Chương tiếp theo**: [14 — So sánh khách quan](/series/ai-for-backend-engineers/14-comparisons/) — các bảng quyết định cho những lựa chọn lớn.
