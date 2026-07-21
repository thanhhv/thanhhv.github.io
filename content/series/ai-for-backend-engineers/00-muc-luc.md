+++
title = "AI cho Backend Engineer — Từ First Principles đến Production"
date = "2026-07-18T07:00:00+07:00"
draft = false
tags = ["backend", "ai", "llm"]
series = ["AI cho Backend Engineer"]
+++

> Bộ tài liệu chuyên sâu về AI Engineering dành cho Backend Engineer, Tech Lead và Solution Architect.
> Không dạy Machine Learning. Không dạy toán. Dạy cách **thiết kế, tích hợp, vận hành và tối ưu** AI như một thành phần trong hệ thống phần mềm.

## Triết lý của bộ tài liệu

AI không phải là phép màu. Với Backend Engineer, LLM là **một service có độ trễ cao, chi phí biến động, đầu ra không xác định (non-deterministic), và không có SLA về tính đúng đắn**. Nhiệm vụ của chúng ta là bọc thành phần đó trong một kiến trúc đủ tốt để hệ thống tổng thể vẫn **đáng tin cậy, quan sát được, mở rộng được và kiểm soát được chi phí**.

Mọi chương đều đi theo mạch:

```
Business Problem → Traditional Software → Giới hạn → LLM giải quyết gì
→ AI Architecture → Backend Integration → Production → Scaling → Trade-off → Failure Cases
```

Mọi kết luận đều trả lời 4 câu hỏi:

1. Vì sao AI được dùng cho bài toán này?
2. Nếu không dùng AI thì giải pháp truyền thống là gì?
3. AI mang lại lợi ích gì và đánh đổi điều gì?
4. Khi nào **không** nên dùng AI?

## Mục lục

### Phần I — Nền tảng (Level 1 & 2)

| Chương | Nội dung |
|---|---|
| [01 — AI Fundamentals cho Engineer](/series/ai-for-backend-engineers/01-ai-fundamentals/) | AI vs ML vs Deep Learning vs LLM, Transformer ở mức kiến trúc, Inference |
| [02 — LLM Fundamentals](/series/ai-for-backend-engineers/02-llm-fundamentals/) | Token, Tokenization, Context Window, Embedding, Attention, Temperature, Top-k, Top-p — và ảnh hưởng đến chất lượng & chi phí |

### Phần II — AI Engineering (Level 3)

| Chương | Nội dung |
|---|---|
| [03 — Prompt Engineering & Structured Output](/series/ai-for-backend-engineers/03-prompt-engineering/) | Prompt Structure, Few-shot, Chain of Thought, Role Prompt, Prompt Template, JSON Mode |
| [04 — Function Calling, Tool Calling & MCP](/series/ai-for-backend-engineers/04-function-calling-mcp/) | Cách LLM "gọi" hệ thống của bạn, Model Context Protocol |
| [05 — RAG (Retrieval-Augmented Generation)](/series/ai-for-backend-engineers/05-rag/) | Kiến trúc RAG, Chunking, Vector Search, Hybrid Search, Re-ranking, Context Assembly, RAG vs Fine-tuning |
| [06 — Vector Database](/series/ai-for-backend-engineers/06-vector-database/) | pgvector vs Qdrant vs Milvus vs Weaviate vs Pinecone; HNSW, IVF, ANN, Recall vs Latency |
| [07 — AI Agents](/series/ai-for-backend-engineers/07-ai-agents/) | Agent, Planning, Memory, Workflow vs Agent, Multi-agent, MCP trong hệ agent |

### Phần III — AI Backend (Level 4)

| Chương | Nội dung |
|---|---|
| [08 — AI Backend Architecture](/series/ai-for-backend-engineers/08-ai-backend-architecture/) | AI Gateway, Model Routing, Caching, Streaming, Session, Conversation History, Rate Limiting, Retry, Fallback |
| [09 — Model Serving](/series/ai-for-backend-engineers/09-model-serving/) | OpenAI/Anthropic/Gemini API, Ollama, vLLM, TensorRT-LLM; API vs Self-hosted |
| [10 — AI System Design](/series/ai-for-backend-engineers/10-ai-system-design/) | Thiết kế Chatbot, AI Search, Customer Support, Coding Assistant, Document QA, Knowledge Base, Email Assistant |

### Phần IV — AI Production (Level 5)

| Chương | Nội dung |
|---|---|
| [11 — Production: Observability, Evaluation & MLOps](/series/ai-for-backend-engineers/11-ai-production/) | Observability, Evaluation, Guardrails, Cost Optimization, Prompt Versioning, A/B Testing, Hallucination Detection, Rollback |
| [12 — Security](/series/ai-for-backend-engineers/12-security/) | Prompt Injection, Jailbreak, Data Leakage, PII Protection, Secret Management, Output Filtering |
| [13 — Production Failure Cases](/series/ai-for-backend-engineers/13-production-failure-cases/) | 13 sự cố production điển hình: triệu chứng, root cause, metric, alert, cách khắc phục và phòng tránh |
| [14 — So sánh khách quan](/series/ai-for-backend-engineers/14-comparisons/) | OpenAI vs Anthropic vs Gemini, API vs Self-hosted, RAG vs Fine-tuning, Workflow vs Agent, LangGraph vs Temporal, Go vs Node.js, Streaming vs Non-streaming |

## Lộ trình đọc gợi ý

- **Backend Engineer mới tiếp cận AI**: đọc tuần tự 01 → 14.
- **Đã build ứng dụng LLM, muốn lên production**: 05, 06, 08, 11, 12, 13.
- **Tech Lead / Architect cần ra quyết định công nghệ**: 05, 07, 09, 10, 14.
- **Đang gặp sự cố production**: 13 (tra cứu theo triệu chứng), sau đó quay lại chương tương ứng.

## Quy ước

- Thuật ngữ kỹ thuật (Token, Embedding, Context Window, RAG, Inference...) giữ nguyên tiếng Anh.
- Sơ đồ dùng [Mermaid](https://mermaid.js.org/) — render trực tiếp trên GitHub, GitLab, Obsidian, VS Code.
- Code mẫu: **Golang** cho phần infrastructure (gateway, routing, streaming), **Node.js/TypeScript** cho phần application (RAG, agent, prompt).
- Giá token, tên model, benchmark là số liệu **minh họa mang tính thời điểm** — luôn kiểm tra lại trước khi ra quyết định, vì thị trường model thay đổi theo quý.
