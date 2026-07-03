+++
title = "Bài 12 — Performance Engineering trong Node"
date = "2026-06-25T08:00:00+07:00"
draft = false
tags = ["backend", "nodejs"]
series = ["NodeJS & Golang"]
+++

# Performance Engineering trong Node.js

---

## 1. Đặc thù của bài toán hiệu năng Node

Nguyên tắc số 0 giống Go (đo trước, tối ưu sau — xem chương 5), nhưng **thứ tự nghi phạm khác hẳn**. Với Node, khi service chậm, xác suất theo kinh nghiệm:

1. **Event loop bị block** (CPU trên main thread) — nghi phạm số 1, đặc sản Node.
2. I/O không song song hóa (await tuần tự) hoặc N+1 query.
3. GC pressure / heap gần trần (GC chạy điên cuồng).
4. Thread pool libuv nghẹt (fs/DNS/crypto — chương 8).
5. Code JS thật sự chậm (deopt, megamorphic) — hiếm hơn nhiều so với 4 cái trên.

Tối ưu Node hiệu quả = chẩn đoán đúng tầng trước khi sờ vào code.

---

## 2. Bộ công cụ chẩn đoán

### 2.1. CPU profiling

```bash
node --cpu-prof app.js                 # ghi .cpuprofile → mở bằng Chrome DevTools
# Hoặc attach process đang chạy production:
node --inspect app.js                  # rồi chrome://inspect → Profiler tab
# Hoặc sample bằng OS-level (thấy cả C++/libuv):
npx 0x app.js                          # flame graph một lệnh
clinic flame -- node app.js            # bộ clinic.js: flame / doctor / bubbleprof
```

Đọc flame graph Node có một điểm riêng: thời gian **idle** (chờ I/O trong epoll) không phải vấn đề — cái cần soi là các khung JS **rộng liên tục** trên main thread (đó chính là block). `clinic doctor` tự động chẩn đoán phân loại "event loop blocked vs I/O vs GC" — điểm khởi đầu tốt khi chưa biết nhìn đâu.

### 2.2. Event loop metrics — dashboard bắt buộc

```js
import { monitorEventLoopDelay, performance } from 'node:perf_hooks';
const h = monitorEventLoopDelay({ resolution: 20 });
h.enable();
// export định kỳ: h.percentile(99) → metric "eventloop_lag_p99"
// và performance.eventLoopUtilization() → tỷ lệ loop bận (ELU)
```

Hai chỉ số này với Node tương đương "CPU + load average" với server truyền thống: **ELU > ~0.7 kéo dài = đã đến lúc scale hoặc tối ưu**; lag p99 tăng đột biến = có code block. Autoscale theo ELU chính xác hơn autoscale theo CPU (CPU của container gồm cả GC/thread pool, không phản ánh độ nghẽn của main thread).

### 2.3. Benchmark

Micro: `node:perf_hooks` hoặc thư viện (`tinybench`, `mitata`) — cùng các bẫy như Go (JIT warm-up còn nặng hơn: phải chạy nóng hàng nghìn lần trước khi đo; dead code elimination của TurboFan cũng tinh vi). Macro: `autocannon` (cùng tác giả clinic.js) cho HTTP load test nhanh; k6/vegeta cho kịch bản nghiêm túc — nhớ vấn đề coordinated omission (chương 5, mục 5).

---

## 3. Tối ưu theo tầng

### Tầng 1 — I/O concurrency (ROI cao nhất, gần như luôn là thủ phạm thật)

```js
// Trước: 4 × RTT
const user = await getUser(id);
const perms = await getPerms(id);
const prefs = await getPrefs(id);
const flags = await getFlags(id);
// Sau: 1 × RTT — không cần "tối ưu code", chỉ cần nhìn ra tính độc lập
const [user, perms, prefs, flags] = await Promise.all([...]);
```

Cùng họ: bật HTTP keep-alive cho outbound (Node <19 mặc định **tắt** — mỗi call API tốn TCP+TLS handshake mới, thêm 5-50ms; `new Agent({ keepAlive: true })` là một dòng ăn cả chục ms); connection pool DB đúng kích thước; batch query (DataLoader pattern gộp N lần load-theo-id trong một tick thành 1 query `WHERE id IN` — sinh ra cho GraphQL N+1 nhưng dùng tốt mọi nơi); cache + singleflight (bản JS: giữ map `key → promise đang bay`, các caller sau await chung promise — chống stampede rẻ tiền mà hiệu quả).

### Tầng 2 — Gỡ block main thread

Theo chương 9-10: diệt `*Sync`, chia batch bằng `await setImmediate()`, việc nặng ra piscina, giới hạn payload, RE2 cho regex input người dùng. Bổ sung: **JSON**: với response lớn lặp cấu trúc, `fast-json-stringify` (serialize theo schema, nhanh 2-5x `JSON.stringify` — chính là cách Fastify nhanh); với parse, đừng parse cái không cần (route body limit, stream parse cho NDJSON).

### Tầng 3 — GC và allocation

- Heap sát trần → GC chạy liên tục ăn CPU (dấu hiệu: GC time > 10-20% qua PerformanceObserver). Fix theo thứ tự: tìm leak (chương 11) → giảm giữ object sống lâu → tăng heap limit nếu đơn thuần thiếu.
- Tránh churn object lưng chừng tuổi (chương 11 mục 2.2): cache TTL ngắn churn cao → cân nhắc cache ngoài process (Redis) hoặc TTL dài hơn + invalidation chủ động.
- Buffer tái sử dụng cho hot path nhị phân; tránh `Buffer.concat` dây chuyền (mỗi lần một allocation + copy — gom một lần với tổng size biết trước).

### Tầng 4 — Vi mô V8 (chỉ khi profile chỉ đích danh)

Shape ổn định, tránh megamorphic call site trong hot loop, tránh `delete`, tránh try/catch bao vòng lặp cực nóng (ngày nay ít ảnh hưởng hơn xưa nhưng vẫn đáng đo), `for` thường thay `forEach` trong hot path (callback = allocation + call overhead). Nhắc lại: tầng này hiếm khi là vấn đề thật của service I/O-bound — đừng bắt đầu từ đây.

---

## 4. Case study tổng hợp

**Triệu chứng:** API search p50 = 40ms nhưng p99 = 3s, CPU container ~50%, RAM ổn. Đội đã thử scale ngang ×2 — p99 giảm không đáng kể (manh mối quan trọng: nếu là bão hòa tài nguyên thì scale phải đỡ).

1. Dashboard event loop lag: p99 lag = 2.8s, **khớp hình dạng** với latency p99 → xác nhận block, không phải downstream chậm (nếu downstream chậm, lag phẳng).
2. `--cpu-prof` 5 phút production → flame graph: 61% một khung `buildFacets` — hàm gom thống kê filter chạy trên **toàn bộ** kết quả (đôi khi 300K document) ngay trên main thread, chỉ với query dải rộng. Vì chỉ ~1% query rơi vào nhánh này, p50 vô can — đúng chữ ký "p50 đẹp, p99 thảm".
3. Fix ba lớp: (a) đẩy tính facet xuống Elasticsearch aggregation (đúng chỗ của nó); (b) trong lúc chờ migrate, cap kết quả facet ở 10K + `setImmediate` mỗi 1K vòng; (c) thêm alert event loop lag p99 > 200ms.
4. Kết quả: p99 = 120ms. Scale ngang trở lại ×1 — **tiết kiệm 50% chi phí**, vì vấn đề chưa bao giờ là thiếu máy.

Bài học: với Node, *scale ngang không chữa được block* — 2 process block vẫn là block cho request rơi vào process đó; và tương quan giữa đồ thị event-loop-lag với đồ thị latency là phép chẩn đoán phân biệt nhanh nhất giữa "mình chậm" và "downstream chậm".

---

## 5. Checklist hiệu năng Node production

1. Dashboard: event loop lag p99, ELU, GC time %, heap sau GC, RPS, latency percentiles.
2. Keep-alive outbound, pool DB đúng cỡ, `UV_THREADPOOL_SIZE` theo workload.
3. Không `*Sync` sau boot; body limit mọi endpoint; RE2 cho regex user input.
4. `Promise.all` những gì độc lập; p-limit những gì hàng loạt.
5. Load test dài hơi + open model trước release lớn; so sánh bằng percentile.
6. Continuous profiling nếu tổ chức đủ lớn (Pyroscope hỗ trợ Node).

---

*Chương tiếp theo: [13 — Production Engineering với Node.js](/series/nodejs-golang/13-node-production-engineering/)*
