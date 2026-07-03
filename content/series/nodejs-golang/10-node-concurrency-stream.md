+++
title = "Bài 10 — Concurrency & Stream trong Node"
date = "2026-06-21T08:00:00+07:00"
draft = false
tags = ["backend", "nodejs"]
series = ["NodeJS & Golang"]
+++

# Concurrency trong Node — Worker Threads, Cluster, Child Process, Stream & Backpressure

---

## 1. Problem Statement

Node đơn luồng JS → hai bài toán không tự giải được:

1. **Tận dụng nhiều core:** máy 16 core, một process Node dùng ~1 core cho JS. 15 core còn lại đứng nhìn.
2. **Việc CPU nặng không được chặn loop** (chương 9).

Node cung cấp ba công cụ với ba mục đích **khác nhau** — chọn nhầm công cụ là mô típ sai lầm phổ biến:

| | Cluster | Worker Threads | Child Process |
|---|---|---|---|
| Bản chất | N process Node, chia socket | N thread V8 trong 1 process | Process con bất kỳ |
| Mục đích | **Scale HTTP ra nhiều core** | **Việc CPU-bound** | Chạy chương trình ngoài |
| Memory | Cách ly hoàn toàn, N × heap | Heap riêng/worker, share được SharedArrayBuffer | Cách ly hoàn toàn |
| Giao tiếp | IPC (serialize) | postMessage (structured clone) + transferable + SAB | stdio/IPC |
| Chi phí tạo | ~30-80ms, ~30-100MB | ~5-15ms, ~5-10MB | Cao (spawn OS process) |

---

## 2. Cluster — nhân bản process theo core

```js
import cluster from 'node:cluster';
import { availableParallelism } from 'node:os';

if (cluster.isPrimary) {
  for (let i = 0; i < availableParallelism(); i++) cluster.fork();
  cluster.on('exit', (worker, code) => {
    log.error({ pid: worker.process.pid, code }, 'worker died');
    cluster.fork();                     // hồi sinh — bắt buộc
  });
} else {
  http.createServer(app).listen(8080);  // các worker CHIA SẺ port
}
```

Cơ chế chia kết nối: primary nhận connection, phát cho worker round-robin (mặc định trên Linux từ Node 16 — trước đó để kernel tranh chấp, phân phối lệch).

**Điểm cần hiểu ở mức kiến trúc:**

- Mỗi worker là **process cách ly**: không share biến, không share cache in-memory, không share connection pool. Cache trong RAM của worker 1 vô hình với worker 2 → cache hit rate chia N, và **state trong RAM là bug chờ nổ** (sticky session cần LB hỗ trợ). Bài học đúng đắn hơn: thiết kế **stateless**, state ra Redis — và khi đã stateless, câu hỏi tiếp theo là...
- **Cluster vs nhiều container:** trong Kubernetes, chạy N pod (mỗi pod 1 process Node, 1 core) thường tốt hơn 1 pod N worker: orchestrator quản lý health/restart/scale từng đơn vị, resource limit rõ, đơn giản hơn. Cluster module hợp với VM/bare metal hoặc PM2. Trên K8s, cluster module gần như hết vai trò.
- Chi phí thật của cluster: **N × baseline memory** (mỗi worker chứa nguyên V8 + app). 16 core × 250MB = 4GB trước khi phục vụ request nào — so với Go: 1 process dùng cả 16 core với 1 heap chung (~50-100MB). Đây là một trong những khác biệt chi phí hạ tầng lớn nhất giữa hai nền tảng.

---

## 3. Worker Threads — thread thật cho việc CPU

### 3.1. Mô hình

Mỗi worker = một V8 isolate riêng (heap riêng, event loop riêng) trong cùng process. Giao tiếp qua `postMessage` — dữ liệu bị **structured clone** (copy). Ba cách chuyển dữ liệu, chi phí khác hẳn nhau:

```js
worker.postMessage(bigObject);                    // 1. CLONE: copy toàn bộ — 100MB là 100MB copy
worker.postMessage(buf, [buf.buffer]);            // 2. TRANSFER: chuyển quyền sở hữu ArrayBuffer,
                                                  //    zero-copy, bên gửi mất quyền dùng
const sab = new SharedArrayBuffer(n);             // 3. SHARE: bộ nhớ chung thật sự
                                                  //    → quay lại thế giới Atomics/race như Go!
```

Điểm trớ trêu đáng ghi nhớ: dùng SharedArrayBuffer là từ bỏ ưu thế "không data race" của Node — bạn nhận lại đúng những vấn đề mutex/atomic mà single-thread đã giải thoát, nhưng với tooling nghèo nàn hơn Go (không race detector).

### 3.2. Pattern đúng: worker pool

Spawn worker mỗi request là anti-pattern (10-15ms + memory mỗi lần). Chuẩn: pool cố định tạo lúc boot, phát việc qua queue — dùng `piscina` thay vì tự viết:

```js
import Piscina from 'piscina';
const pool = new Piscina({ filename: './heavy-task.js',
                           maxThreads: availableParallelism() - 1 }); // chừa main thread
app.post('/render', async (req, res) => {
  res.json(await pool.run(req.body));   // main thread TỰ DO trong lúc worker tính
});
```

**Ngưỡng đáng dùng:** việc CPU > ~10-20ms và đủ thường xuyên. Việc 2ms mà đẩy qua worker thì chi phí serialize + scheduling ăn hết lợi ích. Và nếu 80% workload của service là CPU-bound — dấu hiệu nên viết service đó bằng Go/Rust thay vì xây tháp worker trong Node.

---

## 4. Child Process — cổng ra thế giới ngoài

`spawn` (stream, cho output lớn — ffmpeg, backup), `execFile` (gọn, output nhỏ), `fork` (child Node có kênh IPC). Hai quy tắc an toàn sống còn: **không bao giờ** dùng `exec` nối chuỗi input người dùng (command injection — `execFile` với mảng args tách riêng); luôn đặt timeout + `maxBuffer`, xử lý cả `error` lẫn `exit` event. Zombie process từ child không được reap là leak tài nguyên kinh điển trong container.

---

## 5. Stream và Backpressure — phần quan trọng nhất chương

### 5.1. Vấn đề: dữ liệu lớn hơn RAM

```js
// TAI HỌA: đọc cả file 2GB vào RAM để trả cho client
const data = await fs.promises.readFile('video.mp4');
res.end(data);
// 10 request đồng thời = 20GB RAM = OOM
```

Stream xử lý dữ liệu **theo khúc (chunk)**: memory là O(chunk) thay vì O(file). Bốn loại: Readable, Writable, Duplex, Transform.

### 5.2. Backpressure — cơ chế và cách nó bị phá vỡ

Chênh lệch tốc độ: disk đọc 500MB/s, client mạng yếu nhận 1MB/s. Không kiểm soát → 499MB/s tích vào RAM.

Cơ chế trong Node:

```
readable.pipe(writable) bên trong làm:
  chunk = read()
  ok = writable.write(chunk)
  if (!ok)                        ← buffer nội bộ vượt highWaterMark (mặc định 16-64KB)
      readable.pause()            ← DỪNG nguồn
  writable.on('drain', () =>      ← buffer đã xả
      readable.resume())          ← chạy tiếp
```

**Điểm khác biệt triết học với Go đáng suy ngẫm:** trong Go, backpressure là *mặc định* (channel đầy → block, không làm gì cũng đúng). Trong Node, backpressure là *giao thức tự nguyện* — `write()` **không bao giờ từ chối**, nó chỉ *gợi ý* qua giá trị trả về `false`. Code bỏ qua gợi ý vẫn chạy... cho đến ngày OOM:

```js
// BOM HẸN GIỜ: bỏ qua return value của write
for (const row of millionRows) {
  res.write(render(row));   // client chậm → triệu chunk chất trong RAM process
}
```

### 5.3. Cách viết đúng hiện đại

```js
import { pipeline } from 'node:stream/promises';

await pipeline(                        // backpressure tự động + cleanup + error
  fs.createReadStream('huge.csv'),
  csvParse(),                          // Transform
  new TransformEnrich(),               // Transform async của bạn
  createGzip(),
  fs.createWriteStream('out.gz'),
  { signal }                           // hủy được bằng AbortController
);
```

**Luôn dùng `pipeline`, không dùng `.pipe()` trần:** `.pipe()` không destroy các stream còn lại khi một cái lỗi → file descriptor leak + memory leak âm ỉ — một trong những leak khó tìm nhất của Node. `pipeline` dọn dẹp tất cả và trả lỗi về một chỗ.

Từ Node 16+, Readable có `.map()`, `.filter()`, và **`for await...of`** — xử lý stream như vòng lặp thường, backpressure tự nhiên (không đọc chunk kế khi thân vòng lặp chưa xong):

```js
for await (const chunk of readable) {
  await process(chunk);    // nguồn tự dừng chờ mình — backpressure miễn phí
}
```

### 5.4. Ví dụ thất bại thực tế (mô típ phổ biến)

Service export báo cáo: query 2 triệu row → `rows.map(toCSVLine).join('\n')` → `res.send()`. Chạy tốt ở staging (10K row). Production tháng thứ ba, khách hàng lớn export 2M row: 1.2GB string trong heap → vượt heap limit → **crash cả process, rớt toàn bộ request đang chạy của mọi user khác**. Fix: stream từ DB cursor (`pg-query-stream`) → Transform CSV → response, pipeline; memory từ 1.2GB xuống ~30MB bất kể kích thước export. Bài học kép: (1) dữ liệu production luôn lớn hơn staging 100 lần; (2) trong Node, một request tham ăn giết tất cả — blast radius của lỗi memory là cả process.

---

## 6. Trade-off tổng hợp & lời khuyên

| Nhu cầu | Công cụ đúng |
|---|---|
| HTTP tận dụng N core | Nhiều pod/container (K8s) > cluster module |
| Tác vụ CPU 10ms-10s, thường xuyên | Worker pool (piscina) |
| Đa số workload là CPU-bound | Đổi công nghệ cho service đó (Go/Rust) |
| Chạy tool ngoài (ffmpeg, git) | child_process spawn/execFile |
| Dữ liệu lớn/không biết kích thước | Stream + pipeline, luôn luôn |

Anti-pattern tổng hợp: spawn worker mỗi request; SharedArrayBuffer khi postMessage đủ dùng (mua race, bán simplicity); state trong RAM khi chạy cluster; `.pipe()` không xử lý lỗi; đọc cả file khi có thể stream; bỏ qua giá trị trả về của `write()`.

---

*Chương tiếp theo: [11 — Memory trong Node: V8 Heap, GC, Memory Leak](/series/nodejs-golang/11-node-memory/)*
