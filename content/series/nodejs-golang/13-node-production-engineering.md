+++
title = "Bài 13 — Production Engineering với Node"
date = "2026-06-27T08:00:00+07:00"
draft = false
tags = ["backend", "nodejs"]
series = ["NodeJS & Golang"]
+++

# Production Engineering với Node.js — Error Handling, Graceful Shutdown, Logging, Scaling, Architecture

> Các pattern chung nền tảng (retry/backoff, circuit breaker, rate limit, idempotency, distributed lock) đã phân tích sâu ở chương 6 — nguyên lý y hệt cho Node (thư viện tương ứng: `cockatiel`, `opossum`, `rate-limiter-flexible`). Chương này tập trung vào phần **đặc thù Node**.

---

## 1. Error Handling — nơi Node khác Go nhiều nhất

### 1.1. Bản đồ các loại lỗi và đường đi của chúng

| Loại | Đường đi | Chính sách đúng |
|---|---|---|
| Lỗi nghiệp vụ dự kiến (validate fail, not found) | throw/reject → catch ở handler | Bắt, map sang HTTP status, **không** log như error |
| Lỗi vận hành (DB timeout, ECONNREFUSED) | reject → catch | Retry/breaker/fallback + log + metric |
| **Programmer error** (undefined.foo, assert fail) | `uncaughtException` / `unhandledRejection` | **Log rồi crash.** Không gượng sống |
| Lỗi trong callback event emitter | event `'error'` | Emitter có thể lỗi PHẢI có listener `'error'` — thiếu là crash |

Vì sao "crash on programmer error" là chính sách đúng (chứ không phải khắt khe): sau một exception không bắt được, process ở **trạng thái không xác định** — connection dở dang, lock chưa nhả, biến toàn cục hỏng. Gượng chạy tiếp là phục vụ các request sau bằng một process hỏng ngầm. Crash + restart bởi orchestrator = trở về trạng thái sạch trong vài giây. Đây là điểm Node giống Erlang "let it crash" — với điều kiện có **process manager** và **nhiều instance** (crash 1/10 pod là non-event; crash pod duy nhất là outage — đừng chạy Node production một instance).

```js
process.on('uncaughtException', (err) => {
  logger.fatal({ err }, 'uncaught exception — exiting');
  process.exit(1);                      // flush log sync trước nếu logger async
});
process.on('unhandledRejection', (reason) => {
  logger.fatal({ reason }, 'unhandled rejection — exiting');
  process.exit(1);
});
```

### 1.2. Cấu trúc lỗi có kỷ luật

JS cho phép `throw "chuỗi"` — đừng. Chuẩn hóa: class `AppError extends Error` với `code` (machine-readable), `statusCode`, `isOperational`; dùng `cause` (ES2022) để chain lỗi giữ ngữ cảnh như `%w` của Go: `throw new AppError('ORDER_SAVE_FAILED', { cause: err })`. Một error-handling middleware duy nhất ở cuối chuỗi (Express 5 mới tự bắt lỗi async — Express 4 phải bọc handler async hoặc dính "lỗi rơi xuyên qua middleware", nguồn crash kinh điển).

---

## 2. Graceful Shutdown — bản Node

Trình tự giống Go (chương 6 mục 2): fail readiness → ngừng nhận → drain → đóng tài nguyên. Khác biệt đặc thù Node:

```js
process.on('SIGTERM', async () => {
  healthy = false;                       // readiness fail trước
  await sleep(3000);                     // chờ LB cập nhật
  server.close();                        // ngừng nhận connection MỚI
  // Bẫy Node: server.close() KHÔNG đóng keep-alive connection đang idle!
  // Node 18.2+: server.closeIdleConnections(); (và closeAllConnections() làm chốt chặn)
  // Thiếu dòng này, close() treo đến vô tận vì client giữ keep-alive
  server.closeIdleConnections();
  const t = setTimeout(() => process.exit(1), 25_000); // deadline cứng
  await Promise.allSettled([queue.stop(), db.end(), redis.quit()]);
  clearTimeout(t); process.exit(0);
});
```

Điểm cần nhớ riêng của Node: (1) keep-alive connections — cái bẫy `server.close()` treo; (2) in-flight promise không có cơ chế "chờ hết goroutine" như WaitGroup — phải tự đếm request đang chạy (middleware tăng/giảm counter) nếu muốn drain đúng nghĩa; (3) nếu dùng cluster/PM2, shutdown phải điều phối từ primary xuống từng worker.

---

## 3. Logging — structured từ ngày đầu

`console.log` trong request path có hai tội: không cấu trúc (không query được) và **`process.stdout` với destination chậm có thể block hoặc tích buffer**. Chuẩn hiện nay: **pino** — JSON, nhanh hơn winston nhiều lần nhờ tránh serialize không cần, hỗ trợ chuyển ghi log sang worker (transport) để main thread không trả giá.

Ba kỷ luật quan trọng hơn chọn thư viện:

1. **Correlation ID xuyên suốt:** Node có `AsyncLocalStorage` — context ngầm theo chuỗi async, tương đương `context.Context` của Go nhưng không phải luồn tham số:

```js
import { AsyncLocalStorage } from 'node:async_hooks';
const als = new AsyncLocalStorage();
app.use((req, res, next) => als.run({ reqId: req.headers['x-request-id'] ?? uuid() }, next));
// Ở bất kỳ đâu sâu bên trong, không cần truyền tham số:
logger.info({ reqId: als.getStore()?.reqId, ...fields }, 'order created');
```

2. **Log level có nghĩa:** error = cần người xử lý; warn = bất thường tự phục hồi; info = sự kiện nghiệp vụ; debug = tắt ở production (và nhớ bài học chương 5: guard tham số đắt tiền khỏi bị evaluate khi level tắt).
3. **Không log secret/PII** — redact paths của pino cấu hình một lần, khỏi trông chờ ý thức từng dòng code.

Observability đầy đủ: OpenTelemetry (auto-instrument HTTP/DB/Redis của Node rất trưởng thành) cho tracing; prom-client cho metrics — cộng bộ metric đặc thù (event loop lag, ELU, GC, heap) từ chương 12.

---

## 4. Scaling Node — từ 1 process đến trăm pod

```
Tầng 1: 1 process, 1 core        → đủ cho phần lớn nội bộ/MVP (Fastify chịu được
                                    hàng chục nghìn req/s I/O nhẹ trên 1 core)
Tầng 2: N pod stateless + LB     → con đường chuẩn trên K8s (chương 10: ưu tiên
                                    hơn cluster module), HPA theo ELU/RPS
Tầng 3: tách theo workload       → CPU-bound tách service riêng (worker pool
                                    hoặc ngôn ngữ khác); WebSocket tách khỏi REST
                                    (đặc tính scale khác nhau: connection-heavy
                                    vs request-heavy)
```

Điều kiện tiên quyết của mọi tầng: **stateless** — session ra Redis, upload ra S3, cache in-memory chỉ cho dữ liệu bất biến hoặc chấp nhận lệch. Sticky session là nợ kiến trúc, chỉ chấp nhận có thời hạn cho WebSocket có state buộc phải thế (và cân nhắc pub/sub Redis để các pod chia sẻ broadcast).

**Message queue trong kiến trúc Node:** vai trò và pattern y chương 6-7 (outbox, idempotent consumer, DLQ). Đặc thù Node đáng lưu ý: BullMQ (Redis) là lựa chọn phổ biến cho job queue nội bộ nhờ tích hợp mượt; consumer Node xử lý message chứa việc CPU nặng vẫn dính giới hạn event loop — **concurrency của consumer ≠ parallelism** (100 job async đồng thời vẫn 1 core). Prefetch/concurrency của consumer phải cấu hình theo bản chất job.

---

## 5. Cấu hình, bảo mật, dependency — vệ sinh production

- **Config:** 12-factor, đọc env một lần lúc boot vào object validate bằng schema (zod) — fail fast khi thiếu biến, thay vì `undefined` lan âm thầm tới runtime. Không đọc `process.env` rải rác trong code.
- **Bảo mật đặc thù Node:** npm supply chain là mặt trận chính — lockfile bắt buộc + `npm ci` trong CI/CD; audit tự động (Dependabot/Snyk/socket.dev); cảnh giác postinstall script; ít dependency hơn = bề mặt tấn công nhỏ hơn (cân nhắc trước khi thêm package 50 dependency con cho việc 20 dòng code). Ngoài ra: helmet cho header, rate limit ở edge, không chạy container bằng root.
- **TypeScript strict** ở codebase > vài nghìn dòng: đây là quyết định vận hành chứ không phải sở thích — nó chuyển một lớp lỗi runtime (trang 8, điểm yếu số 5) về compile time với chi phí gần bằng không nhờ tooling hiện đại (tsx, esbuild).

---

## 6. Anti-patterns production tổng hợp

1. Nuốt `uncaughtException` để "không bị crash" — chạy tiếp bằng process hỏng.
2. Chạy một instance duy nhất rồi coi crash-restart là thảm họa (thay vì thiết kế để crash rẻ).
3. `server.close()` không đóng idle keep-alive → deploy nào cũng treo grace period.
4. Log bằng console.log string tự do — không tra được khi sự cố.
5. Cache in-memory làm source of truth trong khi chạy nhiều pod.
6. Consumer queue concurrency 100 cho job CPU-bound (vẫn 1 core, chỉ tăng WIP và timeout).
7. `npm install` (không lockfile) trong CI — build hôm nay khác hôm qua.

---

*Chương tiếp theo: [14 — So sánh toàn diện Golang vs Node.js](/series/nodejs-golang/14-so-sanh-go-nodejs/)*
