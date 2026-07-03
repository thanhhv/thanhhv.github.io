+++
title = "Bài 9 — Event Loop Internals"
date = "2026-06-19T08:00:00+07:00"
draft = false
tags = ["backend", "nodejs"]
series = ["NodeJS & Golang"]
+++

# Event Loop Internals — Phases, Microtask/Macrotask, Callback → Promise → Async/Await

---

## 1. Problem Statement

Event loop là **scheduler duy nhất** của Node. Trong Go, không hiểu scheduler bạn vẫn sống ổn (runtime che chắn); trong Node, **không hiểu event loop thì không debug nổi** các sự cố dạng: "setTimeout(100) chạy sau 2 giây", "một request chậm kéo mọi request chậm", "process ăn 100% CPU mà không throughput". Vì mọi thứ chạy trên một luồng, sự công bằng giữa các tác vụ hoàn toàn phụ thuộc việc code của bạn có *nhả* loop đúng lúc không.

---

## 2. Cấu trúc event loop (libuv)

Mỗi vòng lặp (tick) đi qua các phase theo thứ tự:

```
   ┌─►┌──────────────┐
   │  │ timers       │  callback của setTimeout / setInterval đã đến hạn
   │  ├──────────────┤
   │  │ pending      │  vài callback I/O hệ thống bị hoãn
   │  ├──────────────┤
   │  │ idle/prepare │  nội bộ
   │  ├──────────────┤
   │  │ POLL         │  ★ trái tim: lấy sự kiện I/O mới (epoll),
   │  │              │    chạy callback I/O; BLOCK CHỜ ở đây khi nhàn rỗi
   │  ├──────────────┤
   │  │ check        │  setImmediate
   │  ├──────────────┤
   │  │ close        │  callback 'close' (socket.destroy...)
   │  └──────┬───────┘
   └─────────┘
   
   ★ Giữa MỌI callback: chạy CẠN KIỆT hàng đợi microtask
     (1) process.nextTick queue  (ưu tiên cao nhất, đặc sản Node)
     (2) Promise microtask queue
```

Ba điều quan trọng hơn việc thuộc tên phase:

1. **Timer không chính xác.** `setTimeout(fn, 100)` nghĩa là "không sớm hơn 100ms" — fn chỉ chạy khi loop *quay lại* phase timers. Nếu một callback nào đó chạy 2 giây, timer trễ 2 giây. **Timer trễ chính là triệu chứng số một của event loop bị block** — và là nguyên lý của metric event loop lag (đo mức trễ này để giám sát sức khỏe).
2. **Microtask chạy cạn kiệt giữa các callback.** Promise callback không chờ tick sau — chúng chạy ngay sau callback hiện tại, *trước* mọi I/O. Hệ quả nguy hiểm: vòng lặp đệ quy microtask (`process.nextTick` gọi lại chính nó, hoặc promise chain vô hạn) **bỏ đói toàn bộ I/O** — loop không bao giờ tới phase poll. CPU 100%, không request nào được phục vụ, không lỗi nào được log.
3. **Phase poll là nơi Node "ngủ".** Khi không có việc, Node block trong epoll_wait — 0% CPU. Server 100K kết nối idle không tốn gì. Đây là điều mô hình thread không làm được với chi phí tương đương.

### setImmediate vs setTimeout(0) vs nextTick — chọn gì?

- `setImmediate(fn)`: chạy ở phase check của tick hiện tại/kế — **cách chuẩn để "nhường loop rồi làm tiếp"** (chia nhỏ việc nặng).
- `setTimeout(fn, 0)`: tương tự nhưng vòng qua phase timers, overhead nhỉnh hơn.
- `process.nextTick(fn)`: chạy TRƯỚC cả I/O, không nhường gì cả — chỉ dùng để chuẩn hóa API thành async (tránh callback "zalgo" — lúc sync lúc async); dùng làm vòng lặp là tự bắn vào chân.

---

## 3. Tiến hóa: Callback → Promise → Async/Await

### 3.1. Callback và cái giá của nó

```js
getUser(id, (err, user) => {
  if (err) return handle(err);
  getOrders(user.id, (err, orders) => {
    if (err) return handle(err);            // lặp lại xử lý lỗi mỗi tầng
    enrich(orders, (err, result) => { ... }); // "pyramid of doom"
  });
});
```

Ba vấn đề cấu trúc: kim tự tháp lồng nhau; lỗi phải xử lý thủ công **mỗi tầng** (quên một chỗ là lỗi rơi vào hư vô); không compose được (chạy 3 việc song song rồi gom kết quả = viết bộ đếm thủ công dễ sai).

### 3.2. Promise — biến async thành GIÁ TRỊ

Đột phá khái niệm: kết quả-tương-lai trở thành **object truyền tay được**, thay vì "hành động sẽ gọi anh sau". Vì là giá trị nên compose được:

```js
const [user, config] = await Promise.all([getUser(id), getConfig()]);
```

- Lỗi **tự lan truyền** theo chain đến `.catch` gần nhất — mô phỏng lại được ngữ nghĩa exception.
- Bốn combinator cần thuộc: `all` (fail-fast, đủ cả), `allSettled` (chờ hết bất kể thành bại), `race` (cái nào xong/fail trước — nền tảng làm timeout), `any` (cái thành công đầu tiên — hedge request).

### 3.3. Async/Await — cú pháp đồng bộ, bản chất bất đồng bộ

```js
async function handleOrder(req) {
  const user = await getUser(req.userId);   // "await" = tạm dừng HÀM NÀY,
  const order = await createOrder(user);    //  trả quyền cho event loop
  return order;                              //  KHÔNG chặn thread
}
```

Bản chất: compiler biến hàm async thành **state machine**; mỗi `await` là điểm cắt — hàm bị treo, đăng ký continuation vào microtask queue, event loop chạy việc khác. Cùng ý tưởng "viết tuần tự, chạy bất đồng bộ" như goroutine — khác ở chỗ Go giấu điểm cắt (mọi hàm đều có thể nhường), JS **hiện điểm cắt tường minh** bằng từ khóa `await`.

Trade-off của tường minh: (+) đọc code biết chính xác chỗ nào có thể xen kẽ (interleaving) — dễ suy luận về race logic; (−) "function coloring": hàm async gọi từ hàm sync phải async hóa cả chuỗi bên trên, chia ecosystem thành hai thế giới.

### 3.4. Các lỗi async/await đắt giá nhất trong production

```js
// 1. TUẦN TỰ HÓA VÔ THỨC — lỗi hiệu năng phổ biến nhất của Node
const a = await getA();   // 100ms
const b = await getB();   // 100ms, KHÔNG phụ thuộc a → tổng 200ms vô lý
// Đúng: const [a, b] = await Promise.all([getA(), getB()]); → 100ms

// 2. QUÊN AWAIT — lỗi im lặng nguy hiểm nhất
saveAudit(event);         // không await: lỗi thành unhandledRejection,
                          // hàm return trước khi ghi xong, thứ tự không bảo đảm
// Bật ESLint @typescript-eslint/no-floating-promises — bắt buộc với codebase nghiêm túc

// 3. await TRONG VÒNG LẶP khi có thể song song
for (const id of ids) { await notify(id); }         // N × latency
await Promise.all(ids.map(id => notify(id)));       // 1 × latency
// NHƯNG: 10.000 id = 10.000 call đồng thời = tự DDoS downstream
// → cần giới hạn concurrency (p-limit) — Node KHÔNG có sẵn semaphore như Go

// 4. Promise.all fail-fast bỏ rơi promise khác
// Một cái reject → all reject NGAY, nhưng các promise kia VẪN CHẠY
// (JS không có cancellation tự nhiên!) → dùng AbortController truyền signal
```

Điểm số 4 đáng dừng lại: **JS không có cơ chế hủy promise chuẩn** như `context.Done()` của Go. `AbortController` + `signal` là giải pháp chuẩn hiện nay (fetch, nhiều thư viện hỗ trợ), nhưng nó opt-in — hàm không nhận signal thì không hủy được. Thiết kế API nội bộ: **mọi hàm async có I/O nên nhận `{ signal }`**, tương đương kỷ luật "ctx là tham số đầu" bên Go.

### 3.5. unhandledRejection — chính sách bắt buộc

Từ Node 15, promise rejection không ai bắt sẽ **crash process** (trước đó chỉ warning). Đúng đắn: crash-fast tốt hơn chạy tiếp với trạng thái không xác định. Production cần: handler `process.on('unhandledRejection')` để **log đầy đủ rồi exit(1)** (đừng nuốt để chạy tiếp!), process manager (K8s, PM2) restart, và alert khi tần suất tăng.

---

## 4. Event Loop Blocking — kẻ giết người thầm lặng

### 4.1. Định lượng cho trực giác

Server 1.000 RPS, một handler có đoạn tính toán 50ms CPU. Trong 50ms đó, ~50 request khác đến — **tất cả xếp hàng**. Request thứ 50 chờ gần 50ms *chỉ để bắt đầu*. Chỉ cần 2% request dính handler này, p99 của **toàn bộ** service sụp — kể cả các endpoint "không liên quan". Đây là khác biệt bản chất với Go: một goroutine bận 50ms không ảnh hưởng goroutine khác (có P khác, có preemption); trong Node, **mọi request chung một hàng đợi CPU**.

### 4.2. Thủ phạm phổ biến (xếp theo tần suất thực tế)

1. `JSON.parse`/`JSON.stringify` payload lớn (1MB ≈ vài chục ms).
2. Regex tai họa (catastrophic backtracking — ReDoS): một input hiểm chạy hàng giây. Đã có sự cố lớn trong ngành vì một regex như vậy.
3. Vòng lặp xử lý mảng trăm nghìn phần tử (sort, map, filter chồng nhau).
4. Crypto đồng bộ (`pbkdf2Sync`, `bcrypt.hashSync`), `zlib.gzipSync`, `fs.*Sync` — mọi hàm có đuôi `Sync` sau khi server đã nhận traffic là bug.
5. Template render/serialize HTML lớn.

### 4.3. Phòng và chữa

- **Đo:** metric event loop lag/utilization (`perf_hooks.monitorEventLoopDelay`, `performance.eventLoopUtilization()`) — chỉ số sức khỏe số 1 của Node, alert khi p99 lag > ~100ms.
- **Chặn từ nguồn:** giới hạn kích thước body (`express.json({ limit })`); validate/timeout regex (dùng RE2 cho input người dùng); phân trang dữ liệu.
- **Chia nhỏ:** việc dài chia batch, giữa các batch `await setImmediate()` để nhả loop (throughput giảm nhẹ, latency công bằng trở lại).
- **Đẩy ra ngoài:** việc nặng thường xuyên → worker_threads (chương 10) hoặc tách service (cân nhắc Go cho phần đó).

---

## 5. Best Practices tổng hợp

1. Cấm `*Sync` API sau khi server khởi động xong (lúc boot thì được phép).
2. `Promise.all` cho việc độc lập; `p-limit`/queue cho việc hàng loạt; ESLint no-floating-promises.
3. Mọi hàm async nhận `AbortSignal`; đặt timeout ở mọi I/O ra ngoài (fetch có `AbortSignal.timeout(ms)`).
4. Giám sát event loop lag như giám sát CPU.
5. `unhandledRejection` → log + crash + restart, không nuốt.
6. Đừng dùng `process.nextTick` trừ khi viết thư viện và hiểu rõ vì sao.

---

*Chương tiếp theo: [10 — Concurrency trong Node: Worker Threads, Cluster, Child Process, Stream & Backpressure](/series/nodejs-golang/10-node-concurrency-stream/)*
