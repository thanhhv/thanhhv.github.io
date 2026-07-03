+++
title = "Bài 11 — Memory & Garbage Collection trong Node"
date = "2026-06-23T08:00:00+07:00"
draft = false
tags = ["backend", "nodejs"]
series = ["NodeJS & Golang"]
+++

# Memory trong Node.js — V8 Heap, Garbage Collection, Memory Leak

---

## 1. Cấu trúc bộ nhớ V8

```
Process Node
├── V8 Heap  ← object JS sống ở đây, GC quản lý
│   ├── New Space (1-16MB × 2 semispace)  ← object MỚI sinh
│   ├── Old Space                          ← object sống sót 2 lần GC nhỏ
│   ├── Large Object Space                 ← object > ~256KB
│   └── Code Space                         ← machine code JIT
├── External / Off-heap
│   └── Buffer!  ← Buffer của Node nằm NGOÀI V8 heap
├── Stack (call stack JS, ~1MB)
└── libuv, C++ addon, V8 metadata
```

Hai điều bất ngờ với người mới vận hành Node:

1. **Heap limit mặc định hữu hạn:** ~2GB (Node cũ ~1.5GB; Node mới hơn tự theo memory máy một phần). Container 8GB, Node vẫn crash "heap out of memory" ở ~2GB nếu không set `--max-old-space-size=6144`. Ngược với Go (dùng tự do đến khi OS chặn), Node cần **khai báo trần heap tường minh** — quên là gặp OOM khó hiểu đầu tiên trong đời vận hành Node. Quy tắc: max-old-space-size ≈ 75-80% memory limit container (chừa cho Buffer, stack, V8 metadata).
2. **Buffer không nằm trong heap:** service đẩy file/ảnh nhiều có thể RSS 4GB trong khi heap 500MB. Đo `process.memoryUsage()` phải nhìn cả `heapUsed`, `external`, `arrayBuffers`, `rss` — chẩn đoán sai ô là điều tra sai hướng.

---

## 2. Garbage Collection: Generational — khác triết lý với Go

### 2.1. Generational hypothesis

Quan sát thống kê: **đa số object chết trẻ** (biến tạm trong request). V8 khai thác triệt để:

- **Minor GC (Scavenger):** chỉ quét New Space. Copy object *sống* từ semispace này sang semispace kia — chi phí tỷ lệ với **lượng sống sót** (thường rất ít), không tỷ lệ với lượng rác. Chết trẻ = miễn phí. Chạy thường xuyên, ~1ms, phần lớn song song.
- **Major GC (Mark-Sweep-Compact):** toàn heap, có compact chống phân mảnh. Từ Orinoco: concurrent marking, incremental, parallel — pause thường vài ms, nhưng heap lớn (4-8GB) vẫn có thể giật vài chục ms.

### 2.2. So sánh thiết kế GC — Go vs V8 (câu hỏi phỏng vấn architect)

| | Go | V8 |
|---|---|---|
| Generational | Không | Có |
| Moving/Compact | Không | Có |
| Ưu tiên | Pause cực thấp, đều | Throughput allocation + hiệu quả cho object chết trẻ |
| Vũ khí giảm rác | Escape analysis (stack alloc từ gốc) | Scavenger (chết trẻ gần như miễn phí) |

Hai con đường khác nhau đến cùng mục tiêu: Go **tránh tạo rác** (compiler đưa lên stack), V8 **chấp nhận rác nhưng dọn rác trẻ siêu rẻ**. Hệ quả thực dụng: trong JS, object tạm ngắn hạn rẻ hơn cảm giác của người từ Go/Java chuyển sang; thứ đắt là object **sống lâu lưng chừng** — sống sót qua New Space (bị copy 2 lần) rồi mới chết trong Old Space (chờ Major GC). Cache TTL ngắn với churn cao chính là mẫu hình tệ nhất cho V8.

---

## 3. Memory Leak — bệnh mãn tính số một của Node

Vì sao Node leak nhiều hơn Go trong thực tế? Không phải GC kém — mà vì mô hình process: **một process Node sống hàng tuần phục vụ hàng triệu request**; chỉ cần mỗi request rò 1KB qua một closure giữ nhầm tham chiếu, tuần sau là GB. GC không thể dọn thứ mà **vẫn còn ai đó trỏ tới** — leak trong ngôn ngữ có GC là leak *tham chiếu*, không phải leak *cấp phát*.

### 3.1. Danh mục thủ phạm (theo tần suất thực tế)

**1. Cache/Map toàn cục không giới hạn** — quán quân:

```js
const cache = new Map();
app.get('/user/:id', async (req, res) => {
  if (!cache.has(req.params.id))
    cache.set(req.params.id, await load(req.params.id)); // không TTL, không max size
  ...                                                    // → tăng mãi theo số user
});
// Fix: lru-cache với max + ttl. Cache không giới hạn không phải cache — là leak có tên đẹp.
```

**2. Event listener tích lũy:**

```js
function handleConn(socket) {
  globalEmitter.on('config-updated', () => socket.send(...)); // đăng ký theo request
  // socket đóng nhưng KHÔNG removeListener → closure giữ socket → cả hai bất tử
}
// Cảnh báo sớm có sẵn: "MaxListenersExceededWarning: 11 listeners added" — 
// đừng "fix" bằng setMaxListeners(0) (tắt chuông báo cháy)! Fix bằng off()/once()/AbortSignal.
```

**3. Closure trong timer sống dai:** `setInterval` không bao giờ `clearInterval` giữ mọi biến nó nhìn thấy — mãi mãi.

**4. Promise không bao giờ settle:** `await` một promise treo vĩnh viễn → toàn bộ khung async function + biến local bị giữ. Nguồn: thao tác I/O không timeout.

**5. Closure giữ scope lớn hơn tưởng:** một closure nhỏ giữ tham chiếu tới biến 100MB cùng scope dù không dùng trực tiếp (V8 chia sẻ context object giữa các closure cùng scope).

### 3.2. Quy trình điều tra leak (thứ tự chuẩn)

1. **Xác nhận là leak thật:** đồ thị `heapUsed` sau mỗi Major GC có xu hướng tăng đơn điệu qua nhiều giờ? (RSS cao đơn thuần có thể là Buffer/external hoặc heap chưa đến ngưỡng GC.)
2. **Heap snapshot 3 lần** cách nhau X phút dưới tải (`v8.writeHeapSnapshot()` hoặc inspector) → Chrome DevTools → tab Comparison → object nào tăng đều theo thời gian?
3. **Retainer path:** với object nghi vấn, DevTools chỉ ra **chuỗi tham chiếu từ GC root** — trả lời "ai đang giữ nó". Đây là bước phá án.
4. Sửa, deploy, nhìn lại đồ thị 24h. Chưa có đồ thị phẳng = chưa xong.

Sự cố mẫu: gateway WebSocket leak 200MB/ngày, restart mỗi 3 ngày như "giải pháp". Snapshot so sánh: hàng trăm nghìn closure `onConfigUpdate` tăng tuyến tính — đúng mẫu số 2 ở trên, listener đăng ký mỗi kết nối, không hủy khi disconnect. Một dòng `socket.on('close', () => emitter.off(...))` kết thúc sáu tháng restart định kỳ. Chi tiết đáng nhớ: cảnh báo MaxListenersExceeded đã nằm trong log từ tháng đầu — bị mute vì "ồn".

---

## 4. Vận hành memory trong production

- Metric tối thiểu: `heapUsed`, `heapTotal`, `rss`, `external` + GC duration/frequency (qua `perf_hooks` PerformanceObserver `gc` hoặc prom-client mặc định).
- `--max-old-space-size` đặt tường minh theo container limit (mục 1).
- Chấp nhận sự thật vận hành: nhiều tổ chức chạy Node với **restart định kỳ có chủ đích** (PM2 max-memory-restart / K8s liveness theo memory) như lưới an toàn cuối — không phải thay thế cho việc sửa leak, nhưng là bảo hiểm thực dụng cho leak chưa tìm ra.
- Load test trước release lớn phải chạy **đủ dài** (giờ, không phải phút) — leak không lộ trong smoke test 5 phút.

---

*Chương tiếp theo: [12 — Performance trong Node: Profiling, Benchmark, tối ưu I/O](/series/nodejs-golang/12-node-performance/)*
