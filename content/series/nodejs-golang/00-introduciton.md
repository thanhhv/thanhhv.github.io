+++
title = "Bài 0 — Giới Thiệu"
date = "2026-06-01T08:00:00+07:00"
draft = false
tags = ["backend", "golang", "nodejs"]
series = ["NodeJS & Golang"]
+++

# Golang & Node.js — Tài liệu chuyên sâu cho Backend Engineer

> Bộ tài liệu 15 chương, viết theo tư duy **Problem → Tại sao tồn tại → Cơ chế bên trong → Trade-off → Production → Anti-pattern → Khi nào không dùng**. Mục tiêu không phải dạy cú pháp — mà giúp bạn hiểu bản chất, phân tích bottleneck và tự đưa ra quyết định kỹ thuật có cơ sở.

**Đối tượng:** Software/Backend Engineer, Senior Engineer, Tech Lead, Solution Architect.

---

## Mục lục

### Phần I — Golang

| # | Chương | Nội dung chính |
|---|---|---|
| 01 | [Triết lý thiết kế Go và nền tảng](/series/nodejs-golang/01-go-triet-ly-va-nen-tang/) | Tại sao Go tồn tại · Compilation model · Packages/Modules (MVS) · Stack vs Heap · Escape analysis · Garbage Collector · GOGC/GOMEMLIMIT |
| 02 | [Concurrency: Goroutine, Scheduler, GPM](/series/nodejs-golang/02-go-concurrency-goroutine-scheduler/) | Bài toán C10K/C10M · GPM model · Work stealing · Netpoller vs syscall · Channel internals · Mutex/RWMutex/Atomic · WaitGroup · Context |
| 03 | [Concurrency Patterns](/series/nodejs-golang/03-go-concurrency-patterns/) | Worker Pool · Pipeline · Fan-in/Fan-out · **Backpressure & load shedding** · singleflight · errgroup · sizing theo Little's Law |
| 04 | [Runtime Internals](/series/nodejs-golang/04-go-runtime-internals/) | Memory allocator (mcache/mcentral/mheap) · GC pacer & mark assist · Write barrier · Netpoller · Sysmon · Chẩn đoán RSS vs heap |
| 05 | [Performance Engineering](/series/nodejs-golang/05-go-performance/) | Benchmark & benchstat · pprof (CPU/heap/goroutine/mutex/trace) · Tối ưu theo tầng · Latency percentile · Case study GC assist |
| 06 | [Production Engineering — Resilience](/series/nodejs-golang/06-go-production-engineering/) | Graceful shutdown · Retry & retry storm · Circuit breaker · Rate limiter · **Idempotency** · Distributed lock & fencing token |
| 07 | [Software Architecture](/series/nodejs-golang/07-go-architecture/) | Dependency Injection kiểu Go · Clean/Hexagonal · DDD (bounded context, aggregate) · Event-driven & Transactional Outbox · Error handling như kiến trúc |

### Phần II — Node.js

| # | Chương | Nội dung chính |
|---|---|---|
| 08 | [Tại sao Node tồn tại, V8, Runtime](/series/nodejs-golang/08-node-nen-tang-v8/) | Bối cảnh 2009 · V8 JIT (hidden class, inline caching, deopt) · libuv & thread pool 4 luồng · Single thread nghĩa là gì |
| 09 | [Event Loop Internals](/series/nodejs-golang/09-node-event-loop/) | Các phase libuv · Microtask/Macrotask · nextTick vs setImmediate · Callback → Promise → Async/Await · AbortController · Event loop blocking |
| 10 | [Concurrency: Worker, Cluster, Stream](/series/nodejs-golang/10-node-concurrency-stream/) | Cluster vs nhiều pod · Worker threads & piscina · Child process · **Stream & backpressure** · pipeline · for await |
| 11 | [Memory: V8 Heap, GC, Leak](/series/nodejs-golang/11-node-memory/) | New/Old space · Generational GC (so với Go) · max-old-space-size · 5 loại memory leak · Điều tra bằng heap snapshot |
| 12 | [Performance Engineering](/series/nodejs-golang/12-node-performance/) | Event loop lag & ELU · CPU profiling (0x, clinic) · Tối ưu I/O (keep-alive, DataLoader) · Case study p99 |
| 13 | [Production Engineering](/series/nodejs-golang/13-node-production-engineering/) | Error handling & crash-fast · Graceful shutdown (bẫy keep-alive) · Logging (pino, AsyncLocalStorage) · Scaling · npm supply chain |

### Phần III — So sánh và quyết định kiến trúc

| # | Chương | Nội dung chính |
|---|---|---|
| 14 | [So sánh toàn diện Go vs Node](/series/nodejs-golang/14-so-sanh-go-nodejs/) | 14 tiêu chí: runtime, concurrency, throughput, latency, memory, scale, productivity, ecosystem, vận hành · Decision heuristics · Phân tích chi phí |
| 15 | [Kiến trúc thực tế — 11 loại hệ thống](/series/nodejs-golang/15-kien-truc-thuc-te/) | API Gateway · Auth · Payment · Notification · Chat · Social · E-commerce · FinTech · Streaming · Blockchain · AI Platform |

---

## Cách đọc

- **Đọc tuần tự** nếu muốn xây nền: 01→07 (Go), 08→13 (Node), rồi 14-15.
- **Đọc theo nhu cầu:**
  - Chuẩn bị phỏng vấn senior → 02, 04, 09, 11, 14.
  - Đang chữa cháy hiệu năng → 05 (Go) hoặc 12 (Node), kèm 04/11.
  - Thiết kế hệ thống mới → 14, 15, rồi 06, 07.
  - Sự cố production lặp lại → 06, 13.
- Các chương tham chiếu chéo nhau; pattern chung (retry, idempotency, outbox...) viết chi tiết một lần ở phần Go (chương 6-7) và được phần Node tham chiếu — nguyên lý giống nhau, khác thư viện.

## Quy ước

- Thuật ngữ chuyên ngành (Goroutine, Event Loop, Backpressure, Idempotency...) giữ nguyên tiếng Anh — đây là ngôn ngữ làm việc thực tế.
- Con số hiệu năng trong tài liệu là **bậc độ lớn để định hướng trực giác**, không phải cam kết — luôn benchmark trên workload và hardware của bạn.
- Code ví dụ tối giản để làm rõ ý — production cần bổ sung đầy đủ error handling, config, test.

---

*Kiến thức nền dựa trên Go 1.2x và Node.js 20/22 LTS.*