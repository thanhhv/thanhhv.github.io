+++
title = "Chương 14 — So sánh khách quan: các quyết định lớn"
date = "2026-07-18T09:20:00+07:00"
draft = false
tags = ["backend", "ai", "llm"]
series = ["AI cho Backend Engineer"]
+++

Nguyên tắc đọc chương này: các bảng dưới đây là **khung phân tích**, không phải phán quyết vĩnh viễn. Thị trường AI thay đổi theo quý — hiệu năng và giá cụ thể phải kiểm tra lại tại thời điểm quyết định; cấu trúc trade-off thì bền hơn nhiều.

---

## 14.1. OpenAI vs Anthropic vs Gemini

| Tiêu chí | OpenAI | Anthropic | Gemini (Google) |
|---|---|---|---|
| Dải model | Rộng, nhiều mức giá, hệ sinh thái tool lớn nhất | Tập trung dải Claude (Haiku/Sonnet/Opus), mạnh về coding, agent, văn bản dài | Mạnh multimodal, context rất dài, gắn hệ sinh thái GCP |
| Tính năng platform | Structured outputs, batch, fine-tuning, realtime — trưởng thành | Tool use, prompt caching, MCP, computer use — mạnh về agentic | Tích hợp Vertex AI, grounding với Google Search |
| Enterprise/compliance | Azure OpenAI cho enterprise | AWS Bedrock / GCP Vertex đều có | Vertex AI, region đa dạng |
| Vận hành | Rate limit theo tier; deprecation nhanh | Ổn định API tốt | Quota theo GCP project |
| Khi nào nghiêng về | Cần hệ sinh thái/cộng đồng lớn nhất, nhiều lựa chọn giá | Workload coding/agent/tài liệu dài, ưu tiên chất lượng suy luận và an toàn | Đã ở trên GCP, cần multimodal/context cực dài, tối ưu chi phí |

**Kết luận thực dụng**: chênh lệch chất lượng giữa 3 nhà ở top-tier là nhỏ và đổi chỗ liên tục theo release — **eval trên task + dữ liệu (tiếng Việt) của bạn** là trọng tài duy nhất. Quyết định kiến trúc đúng không phải "chọn ai" mà là **không khóa cứng vào ai**: interface trừu tượng + gateway (Chương 08) để đổi bằng config. Nhiều hệ thống trưởng thành dùng cả 2–3 nhà theo từng loại task.

## 14.2. API vs Self-hosted

(Phân tích đầy đủ ở Chương 09 — bảng tóm tắt)

| | API | Self-hosted (vLLM + open-weights) |
|---|---|---|
| Hiệu năng đỉnh (chất lượng) | Frontier | Thấp hơn cho task khó; đủ cho task hẹp |
| Độ phức tạp | Thấp | Cao (GPU infra, capacity, rollout) |
| Chi phí | Biến đổi thuần, 0 khi 0 dùng | Cố định cao; rẻ hơn CHỈ khi utilization cao + tính đúng nhân sự |
| Mở rộng | Gọi thêm (đến rate limit) | Mua/thuê thêm GPU, autoscale khó |
| Vận hành | Retry/fallback là đủ | Toàn bộ stack serving là của bạn |
| Data control | Theo DPA của provider | Tuyệt đối |
| Use case phù hợp | Mặc định; sản phẩm đang tìm hướng; task đa dạng | Task hẹp volume lớn ổn định; compliance cứng; cần model fine-tune riêng |

## 14.3. RAG vs Fine-tuning

(Phân tích đầy đủ ở Chương 05 — quy tắc nhớ)

| Nhu cầu | Chọn |
|---|---|
| Model cần **biết** thông tin mới/riêng/thay đổi | RAG |
| Model cần **hành xử** theo cách riêng (format, giọng điệu, domain style) | Prompt trước, fine-tuning khi prompt đuối |
| Cần trích dẫn nguồn, kiểm soát truy cập theo user | RAG (fine-tuning không làm được) |
| Cần giảm chi phí model lớn cho task hẹp volume khổng lồ | Fine-tune model nhỏ (distillation) |
| Cả kiến thức lẫn hành vi | Kết hợp: RAG cho kiến thức + fine-tuned model nhỏ trong pipeline |

## 14.4. pgvector vs Qdrant vs Milvus

(Bảng đầy đủ 5 sản phẩm ở Chương 06 — rút gọn 3 lựa chọn phổ biến nhất)

| | pgvector | Qdrant | Milvus |
|---|---|---|---|
| Hiệu năng | Đủ đến vài triệu vector | Rất tốt đến trăm triệu | Tốt nhất ở quy mô tỷ |
| Độ phức tạp vận hành | ~0 nếu đã chạy Postgres | Thấp–trung bình | Cao (kiến trúc phân tán nhiều thành phần) |
| Chi phí | Thấp nhất | Thấp | Hạ tầng + nhân lực đáng kể |
| Điểm mạnh riêng | JOIN/ACID với dữ liệu nghiệp vụ | HNSW tuning, payload filter, hybrid tốt | Scale-out, nhiều loại index |
| Chọn khi | Mặc định cho đa số ứng dụng | Vector là workload trung tâm | Quy mô rất lớn + có đội infra |

## 14.5. Workflow vs Agent

(Phân tích đầy đủ ở Chương 07)

Câu hỏi quyết định: **vẽ được flowchart trước không?** Vẽ được → Workflow (chi phí/latency dự đoán được, debug dễ, tin cậy cao). Không vẽ được vì đường đi phụ thuộc dữ liệu gặp trên đường → Agent (đổi tính linh hoạt lấy phương sai chi phí, độ trễ và độ tin cậy). Sai lầm phổ biến nhất 2024–2026 không phải chọn nhầm workflow — mà là **agent hóa những thứ vốn là workflow**.

## 14.6. LangGraph vs Temporal (orchestration cho AI workflow)

Hai công cụ đến từ hai thế giới đang gặp nhau:

| | LangGraph | Temporal |
|---|---|---|
| Xuất thân | Framework AI (đồ thị trạng thái cho agent/LLM app) | Nền tảng durable execution cho distributed systems |
| Điểm mạnh | Primitives cho LLM: state đồ thị, human-in-the-loop, streaming, tích hợp hệ sinh thái LangChain; prototype nhanh | Độ bền công nghiệp: retry/resume/timeout/versioning workflow đã được tôi luyện; chạy đa ngôn ngữ (Go/Java/TS/Python); ops trưởng thành |
| Điểm yếu | Tính bền/ops phụ thuộc thêm thành phần (checkpointer, platform trả phí); hệ sinh thái đổi nhanh | Không có khái niệm AI nào sẵn — tự viết activity gọi LLM, tự quản state hội thoại; nặng ký cho prototype |
| Hiệu năng/scale | Đủ cho phần lớn app; scale = tự lo hạ tầng state | Scale đã chứng minh ở workload triệu workflow |
| Use case phù hợp | Đội AI-first, vòng lặp agent phức tạp, cần ship nhanh | Đội backend đã dùng/quen Temporal, workflow AI chạy dài xen lẫn nghiệp vụ (thanh toán, đơn hàng), yêu cầu độ bền nghiêm ngặt |

**Góc nhìn kiến trúc**: agent chạy dài *là* bài toán durable execution (Chương 07). Nếu tổ chức đã có Temporal, đặt LLM call vào activity của Temporal thường vững hơn là mang thêm một hệ orchestration mới. Nếu chưa có gì và bài toán thuần AI, LangGraph cho tốc độ. Kết hợp (Temporal điều phối ngoài, LangGraph trong một activity) cũng là pattern hợp lệ.

## 14.7. Golang vs Node.js trong AI Backend

| | Golang | Node.js/TypeScript |
|---|---|---|
| Hiệu năng | Vượt trội cho gateway/proxy/streaming đồng thời cao (goroutine, memory footprint nhỏ) | Đủ tốt — bottleneck thật là LLM latency, không phải runtime |
| Hệ sinh thái AI | Mỏng hơn: SDK chính thức có (OpenAI/Anthropic), framework agent/RAG ít | Dày nhất sau Python: LangChain/LlamaIndex JS, Vercel AI SDK, SDK mọi provider |
| Tốc độ phát triển | Chậm hơn cho logic AI nhiều thử nghiệm | Nhanh, chia sẻ type với frontend, phù hợp iterate prompt/pipeline |
| Vận hành | Binary tĩnh, ít RAM — đẹp cho infra layer | Quen thuộc với đa số đội web |
| Use case phù hợp | **AI Gateway, model router, streaming proxy, high-QPS serving path** | **Application layer: RAG service, agent, prompt pipeline, BFF** |

Kết luận: không phải "chọn một" — phổ biến và hợp lý nhất là **Go ở tầng hạ tầng, TypeScript ở tầng ứng dụng** (đúng như quy ước code của bộ tài liệu này). Python vẫn thống trị mảng ML thuần (training, data science) — nhưng backend AI application thì Go/Node hoàn toàn đủ.

## 14.8. Streaming vs Non-streaming

| | Streaming | Non-streaming |
|---|---|---|
| UX | TTFT ~1s — bắt buộc cho chat/UI người dùng | Chờ trọn (5–30s) — chỉ ổn khi không có người nhìn |
| Độ phức tạp | Cao: SSE/WebSocket, lỗi giữa chừng, guardrail từng đoạn, usage chốt cuối stream, client cancel | Thấp: request-response thường, validate một lần trên toàn bộ output |
| Hiệu năng hệ thống | Giữ connection lâu (chọn runtime chịu concurrent connection tốt) | Ngắn gọn, dễ pool |
| Xử lý output | Khó — parse JSON dở dang, filter đã-phát-thì-không-rút-được | Dễ — có toàn văn rồi mới xử lý |
| Use case | Chat, assistant, coding tool — mọi nơi người dùng chờ đọc | Pipeline máy-máy: extraction, phân loại, batch, agent bước trung gian |

Quy tắc: **có mắt người đang chờ → stream; không → đừng stream**. Hybrid phổ biến: stream ra UI đồng thời buffer đầy đủ ở server để validate/log/mask.

---

## Lời kết của bộ tài liệu

Nếu chỉ giữ lại năm điều từ 14 chương:

1. **AI là một component, không phải phép màu** — service chậm, đắt, non-deterministic, cần được bọc trong kiến trúc tốt như mọi dependency không đáng tin khác.
2. **Chọn công cụ yếu nhất còn giải được bài toán** — rule → ML nhỏ → LLM call → workflow → agent. Mỗi bậc leo phải có bằng chứng (eval) rằng bậc dưới không đủ.
3. **Retrieval và evaluation là hai khoản đầu tư ROI cao nhất** — chất lượng RAG nằm ở retrieval; khả năng cải tiến nằm ở eval. Model chỉ là một tham số cấu hình.
4. **Thiết kế cho thất bại**: model sẽ bịa, provider sẽ sập, chi phí sẽ spike, injection sẽ đến — degradation path, budget cap, guardrail, kill switch là tính năng hạng nhất, không phải việc "làm sau".
5. **Mọi quyết định đo bằng sáu trục: chi phí, độ trễ, chất lượng, khả năng mở rộng, bảo mật, khả năng vận hành** — không bao giờ bằng độ mới của công nghệ.

Chúc bạn xây được hệ thống AI mà 3 giờ sáng không ai phải thức dậy vì nó.
