+++
title = "Bài 4 — Go Runtime Internals"
date = "2026-06-09T08:00:00+07:00"
draft = false
tags = ["backend", "golang"]
series = ["NodeJS & Golang"]
+++

# Go Runtime Internals — Memory Allocator, GC Internals, Netpoller, Sysmon

> Chương này đào sâu phần "bên dưới mui xe" đã phác thảo ở chương 1-2. Đối tượng: senior+ cần debug các vấn đề hiệu năng mà tài liệu bề mặt không giải thích được.

---

## 1. Vì sao phải hiểu runtime

Ba loại sự cố production chỉ giải thích được khi hiểu runtime:

- "Service dùng 2GB RSS nhưng heap profile chỉ thấy 800MB" → allocator giữ memory, page chưa trả OS, fragmentation.
- "p99 tăng vọt định kỳ mỗi 2 phút" → GC cycle, hoặc `forcegcperiod` (GC cưỡng bức mỗi 2 phút khi nhàn rỗi).
- "CPU 30% nhưng throughput không tăng khi thêm load" → GC assist, lock contention trong runtime, netpoller.

---

## 2. Memory Allocator

### 2.1. Thiết kế: phỏng theo TCMalloc

Bài toán: `malloc` toàn cục có lock → N core tranh nhau một lock → allocation trở thành điểm nghẽn. Go giải bằng phân cấp 3 tầng:

```
mcache  (per-P, KHÔNG LOCK)      ← 99% allocation nhỏ đi qua đây
   │ hết thì xin
mcentral (per size-class, có lock nhưng phân tán 68 cái)
   │ hết thì xin
mheap   (toàn cục, lock, nói chuyện với OS qua mmap)
```

- Object ≤ 32KB được làm tròn vào 1 trong ~68 **size class** (8B, 16B, 24B, ... 32KB). Cấp phát từ mcache của P hiện tại — không lock, vài chục ns.
- Object > 32KB đi thẳng mheap (đắt hơn — một lý do nữa để tránh cấp phát buffer lớn trong hot path).
- **Trade-off của size class:** làm tròn gây lãng phí nội bộ (xin 33KB nhận 48KB) — trung bình ~12% overhead, đổi lấy tốc độ và chống phân mảng ngoài.

### 2.2. Vì sao RSS ≠ heap in use

Memory đi một chiều chậm: app free → allocator giữ lại trong span để tái sử dụng → chỉ trả OS khi scavenger chạy nền (dùng `MADV_FREE`/`MADV_DONTNEED`). Với `MADV_FREE` (Linux), kernel chỉ *thu hồi khi cần* — RSS trên metric **không giảm** dù Go đã trả! Đây là nguồn gốc của vô số báo động memory leak giả. 

**Cách đọc đúng:** so `runtime.MemStats.HeapInuse` / metric `go_memstats_*` với RSS. Leak thật = HeapInuse tăng không ngừng. RSS cao + HeapInuse ổn định = bình thường.

### 2.3. sync.Pool — tái sử dụng object

```go
var bufPool = sync.Pool{New: func() any { return new(bytes.Buffer) }}

buf := bufPool.Get().(*bytes.Buffer)
buf.Reset()                 // BẮT BUỘC reset — pool trả object bẩn
defer bufPool.Put(buf)
```

Cơ chế: pool per-P (không lock trên fast path), bị **dọn một phần mỗi GC cycle** (victim cache giữ một thế hệ). Hệ quả: pool là cache cơ hội, không phải kho chứa có bảo đảm — đừng đặt connection hay tài nguyên cần đóng vào đó.

Khi nào đáng dùng: object cấp phát hàng chục nghìn lần/s, kích thước đáng kể (buffer, encoder). Đo bằng benchmark + profile trước/sau; dùng bừa làm code khó đọc mà không được gì.

---

## 3. GC Internals — sâu hơn chương 1

### 3.1. GC Pacer và Assist — phần ít người biết nhất

GC concurrent có vấn đề cố hữu: app cấp phát **trong lúc** GC đang mark. Nếu app cấp phát nhanh hơn GC mark, heap phình vô hạn giữa chu kỳ. Giải pháp: **mark assist** — goroutine nào cấp phát nhiều trong lúc GC chạy bị bắt "đóng thuế": tự làm một phần việc mark tương ứng lượng nó cấp phát.

**Hệ quả production quan trọng:** khi allocation rate quá cao, **latency của chính request** tăng vì goroutine phục vụ request phải đi làm việc GC. Triệu chứng: p99 xấu đi khi traffic tăng, CPU profile thấy `gcAssistAlloc`. Fix: giảm allocation (pool, tránh escape) — không phải chỉnh GOGC.

### 3.2. Write barrier

Trong pha mark, mọi lệnh ghi con trỏ `*p = q` bị compiler chèn thêm code thông báo cho GC (hybrid write barrier từ Go 1.8 — kết hợp Yuasa + Dijkstra). Nó bảo vệ bất biến tri-color: object đen không bao giờ trỏ tới object trắng mà GC không biết. Chi phí: vài % throughput trong pha mark — cái giá của pause sub-millisecond.

### 3.3. Tam giác điều chỉnh

```
        GOGC cao (ít GC, tốn RAM)
           ▲
          /  \
         /    \   ← Bạn chỉ chọn được vị trí trên tam giác,
        /      \     không thoát khỏi nó
       ▼        ▼
  CPU cho GC   GOMEMLIMIT (trần RAM,
  (GOGC thấp)   GC chạy gắt khi gần trần)
```

Cấu hình khuyến nghị container: `GOMEMLIMIT` = 85-90% limit; GOGC giữ mặc định trừ khi profile chứng minh điều ngược lại. Cẩn thận combo `GOMEMLIMIT` chật + heap sát trần → **GC death spiral**: GC chạy liên tục ăn 100% CPU. Go giới hạn GC ở 50% CPU để tránh chết hẳn, nhưng service coi như tê liệt — dấu hiệu cần thêm RAM hoặc giảm allocation, không phải chỉnh tham số tiếp.

---

## 4. Netpoller — trái tim I/O của Go

### 4.1. Cơ chế

Mọi socket trong Go được đặt non-blocking mode từ đầu. Khi goroutine `conn.Read()` mà chưa có dữ liệu:

```
1. runtime nhận EAGAIN từ kernel
2. Đăng ký fd vào epoll (Linux) / kqueue (BSD/macOS) / IOCP (Windows)
3. G bị park (waiting), gỡ khỏi M — M chạy G khác ngay
4. Scheduler/sysmon poll epoll định kỳ (non-blocking hoặc blocking khi nhàn)
5. fd sẵn sàng → netpoller trả danh sách G → đưa vào run queue
6. G tỉnh dậy, Read lại, lần này có dữ liệu
```

**Ý nghĩa kiến trúc:** Go = event loop của nginx/Node **cộng** mô hình lập trình tuần tự. 100K kết nối idle = 100K goroutine parked (mỗi cái ~vài KB) + 1 epoll instance — không tốn thread, không tốn CPU. Đây là câu trả lời cho câu hỏi "Go xử lý C10M kiểu gì".

### 4.2. Điểm cần biết khi vận hành

- **File thường không đi qua netpoller như socket** — disk I/O trên Linux (không io_uring) là blocking syscall → chiếm M (chương 2, mục 3.3). Service đọc file nặng sẽ thấy số OS thread tăng. Cần giới hạn concurrency đọc file bằng semaphore.
- **Timeout tầng socket** (`SetReadDeadline`) được tích hợp vào netpoller bằng timer — rẻ, chính xác, nên dùng thay vì tự chế timeout bằng goroutine phụ.

---

## 5. Sysmon — người gác đêm

Goroutine đặc biệt chạy trên M riêng (không cần P), thức dậy mỗi 20µs-10ms:

1. **Retake P** bị syscall chiếm quá lâu → giao P cho M khác (đảm bảo syscall không làm đói CPU).
2. **Preempt** G chạy quá 10ms (gửi SIGURG — chương 2).
3. Poll netpoller nếu lâu chưa ai poll.
4. Kích hoạt GC nếu 2 phút chưa chạy (forcegc).
5. Trả memory nhàn rỗi cho OS (scavenger).

Biết sysmon giúp giải thích: vì sao p999 có nhiễu ~10ms (preemption granularity), vì sao có GC cycle "vô cớ" mỗi 2 phút trên service nhàn rỗi.

---

## 6. Quan sát runtime trong production

```go
// Expose sẵn qua Prometheus client (promhttp) — các metric quan trọng nhất:
// go_goroutines                 ← leak detector số 1
// go_memstats_heap_inuse_bytes  ← so với RSS
// go_gc_duration_seconds        ← pause p99
// go_memstats_alloc_bytes_total ← allocation rate (đạo hàm)
// go_threads                    ← OS thread (phát hiện syscall storm)
```

Bổ sung `GODEBUG=gctrace=1` in mỗi chu kỳ GC ra stderr khi cần điều tra sâu; `runtime/metrics` (API mới) cho số liệu chi tiết hơn MemStats với chi phí thấp hơn.

**Checklist chẩn đoán nhanh:**

| Triệu chứng | Nghi phạm | Xác nhận bằng |
|---|---|---|
| goroutines tăng mãi | leak (channel không thoát) | pprof goroutine profile → dòng nào block |
| RSS cao, heap thấp | scavenger/MADV_FREE, hoặc cgo/mmap ngoài heap | so HeapInuse vs RSS; kiểm tra cgo |
| p99 răng cưa chu kỳ | GC | gc_duration + gctrace, allocation rate |
| CPU cao khi tải cao, throughput không tăng | GC assist / lock contention | CPU profile: gcAssistAlloc / mutex profile |
| OS thread hàng trăm | blocking syscall (file, cgo, DNS cgo resolver) | go_threads + thread profile |

---

## 7. Trade-off & Khi nào kiến thức này thành hành động

- Runtime Go là **hộp đen có cửa sổ**: bạn không điều khiển scheduler/GC trực tiếp (không có API "chạy G này trên core kia" — trừ `LockOSThread` cho trường hợp đặc biệt như thư viện C cần thread affinity, UI loop). Triết lý: đa số người dùng làm đúng mặc định > số ít người tinh chỉnh sâu.
- Đánh đổi chấp nhận khi chọn Go: **mất quyền kiểm soát vi mô, đổi lấy mặc định tốt**. Nếu hệ thống của bạn cần kiểm soát vi mô thật sự (pin thread, NUMA, custom allocator toàn diện) — đó là tín hiệu cân nhắc Rust/C++, không phải tín hiệu chiến đấu với runtime Go.

---

*Chương tiếp theo: [05 — Performance Engineering: Benchmark, pprof, Optimization](/series/nodejs-golang/05-go-performance/)*
