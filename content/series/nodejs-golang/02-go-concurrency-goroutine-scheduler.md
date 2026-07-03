+++
title = "Bài 2 — Goroutine, Scheduler & GPM Model"
date = "2026-06-05T08:00:00+07:00"
draft = false
tags = ["backend", "golang"]
series = ["NodeJS & Golang"]
+++

# Concurrency — Goroutine, Scheduler và GPM Model

---

## 1. Problem Statement

### Bài toán C10K → C10M

Một server hiện đại cần phục vụ 10.000 → 10.000.000 kết nối đồng thời. Với mô hình "1 kết nối = 1 OS thread":

- **Memory:** 10.000 thread × 8MB stack (mặc định Linux) = 80GB chỉ cho stack. Bất khả thi.
- **Context switch:** chuyển đổi thread qua kernel tốn 1-10µs (lưu/khôi phục register, TLB flush, cache pollution). Với 10K thread active, CPU dành phần lớn thời gian cho việc chuyển đổi thay vì làm việc.
- **Scheduler kernel** không biết gì về ngữ nghĩa ứng dụng — nó schedule công bằng theo time-slice, không theo "goroutine này đang chờ I/O".

### Các giải pháp cũ và giới hạn

| Giải pháp | Cách hoạt động | Giới hạn |
|---|---|---|
| Thread pool (Java cổ điển) | Số thread cố định + queue | Thread bị block bởi I/O vẫn chiếm chỗ; sizing pool là nghệ thuật đen |
| Callback / event loop (Node.js, nginx) | 1 thread + non-blocking I/O | Callback hell; 1 tính toán nặng chặn toàn bộ; code bất đồng bộ "lây nhiễm" (colored functions) |
| Async/await (C#, Rust, JS) | Compiler biến đổi thành state machine | Vẫn chia thế giới thành sync/async ("function coloring"); ecosystem phải async hết |

**Go chọn hướng thứ ba:** viết code **tuần tự, blocking** như bình thường — runtime lo việc biến blocking thành non-blocking bên dưới. Không có async/await, không có màu hàm. Đây là điểm bán hàng lớn nhất của Go.

---

## 2. Goroutine — vì sao rẻ đến vậy

```go
go handleRequest(conn)   // tạo goroutine: ~vài trăm ns, 2KB stack
```

Ba con số quyết định:

| | OS Thread | Goroutine |
|---|---|---|
| Stack khởi tạo | 1-8MB (cố định, reserve) | 2KB (grow động tới 1GB) |
| Chi phí tạo | ~10-50µs (syscall) | ~200-500ns (user space) |
| Context switch | ~1-10µs (qua kernel) | ~100-200ns (user space, chỉ lưu 3 register chính: PC, SP, DX) |

Goroutine context switch rẻ vì: (1) không vào kernel, (2) Go scheduler biết chính xác thời điểm switch (tại các điểm hợp tác — function call, channel op, syscall) nên chỉ cần lưu tối thiểu register, (3) không TLB flush vì cùng address space.

**Hệ quả thiết kế:** với Go, "mỗi request một goroutine" là pattern chuẩn — điều bị coi là anti-pattern chết người ở mô hình thread. `net/http` mặc định làm đúng như vậy.

---

## 3. GPM Model — Scheduler bên trong

### 3.1. Ba thực thể

```
G (Goroutine): đơn vị công việc — stack, PC, trạng thái
M (Machine):   OS thread thực sự thực thi code
P (Processor): "giấy phép chạy" — context scheduling, tối đa GOMAXPROCS cái
               (mặc định = số CPU core)

  ┌─────────────────────────────────────────────────┐
  │                Global Run Queue                 │
  └─────────────────────────────────────────────────┘
        ▲                    ▲
   ┌────┴────┐          ┌────┴────┐
   │ P0      │          │ P1      │       ... (GOMAXPROCS P)
   │ LRQ:    │          │ LRQ:    │
   │ G1→G2→G3│          │ G4→G5   │   LRQ = Local Run Queue (max 256 G)
   └────┬────┘          └────┬────┘
        │                    │
   ┌────┴────┐          ┌────┴────┐
   │   M0    │          │   M1    │   M gắn với P để chạy G
   └─────────┘          └─────────┘
```

**Tại sao cần P (tách khỏi M)?** Đây là câu hỏi phỏng vấn senior kinh điển. Nếu chỉ có G và M (như Go trước 2012): mọi M tranh nhau một global queue → lock contention hủy diệt scalability. P mang theo **local run queue** và **mcache** (cache cấp phát memory) — M lấy G từ LRQ của P mà **không cần lock**. P còn là cơ chế giới hạn parallelism thực sự: dù có 1000 M (do syscall), chỉ GOMAXPROCS M được chạy Go code cùng lúc.

### 3.2. Work stealing

Khi P hết việc trong LRQ, theo thứ tự:

1. Lấy từ Global Run Queue (kiểm tra định kỳ mỗi 61 tick để tránh starvation).
2. Kiểm tra netpoller (goroutine chờ I/O network đã sẵn sàng chưa).
3. **Steal một nửa** LRQ của một P ngẫu nhiên khác.

Work stealing giữ mọi core bận mà không cần bộ điều phối trung tâm — thiết kế phi tập trung, scale theo số core.

### 3.3. Xử lý syscall blocking — phần tinh vi nhất

Khi G thực hiện syscall blocking (đọc file, cgo call):

```
Trước:  P ── M1 ── G1 (vào syscall)
Sau:    G1 + M1 tách ra chờ syscall (M1 bị kernel block)
        P  ── M2 (lấy từ pool hoặc tạo mới) ── G2 tiếp tục chạy
Khi syscall xong: G1 vào lại run queue; M1 về pool
```

Đây là lý do một service Go đọc file nhiều có thể có **hàng trăm OS thread** dù GOMAXPROCS=8 — hoàn toàn bình thường, nhưng cần biết khi đọc metrics.

**Với network I/O thì khác:** Go không block M. Netpoller (epoll/kqueue/IOCP) đăng ký fd, G bị "park" (gỡ khỏi M, trạng thái waiting), M chạy G khác ngay. Khi fd sẵn sàng, netpoller đánh thức G. Đây chính là cách Go biến code blocking-style thành non-blocking thực thi — **event loop giấu bên trong runtime**, lập trình viên không bao giờ thấy nó.

### 3.4. Preemption

Go ≤1.13: goroutine chỉ nhường CPU tại function call (cooperative). Vòng lặp `for {}` không call gì có thể **chiếm P vĩnh viễn**, thậm chí chặn GC → cả process đứng hình. 

Go 1.14+ thêm **asynchronous preemption**: sysmon (goroutine giám sát nền) phát hiện G chạy quá ~10ms → gửi signal SIGURG → G bị ngắt tại điểm an toàn. Bài học lịch sử: hầu hết bug "service Go treo cứng" trước 2020 đến từ đây.

---

## 4. Channel — chia sẻ bằng giao tiếp

### 4.1. Bản chất bên trong

Channel là struct `hchan`: một **ring buffer** + hai hàng đợi goroutine chờ (sendq, recvq) + một mutex. Không ma thuật — chỉ là hàng đợi có khóa được tích hợp với scheduler:

- Gửi vào channel đầy → G bị park vào `sendq`, nhường M cho G khác (không busy-wait, không tốn CPU).
- Nhận từ channel rỗng → park vào `recvq`.
- **Tối ưu đáng chú ý:** khi có receiver đang chờ, sender copy dữ liệu **thẳng vào stack của receiver**, bỏ qua buffer — tiết kiệm một lần copy.

### 4.2. Unbuffered vs Buffered — quyết định ngữ nghĩa, không phải hiệu năng

```go
ch := make(chan T)     // unbuffered: gửi/nhận là điểm ĐỒNG BỘ HÓA
ch := make(chan T, N)  // buffered: tách rời sender/receiver tối đa N phần tử
```

- **Unbuffered = bảo đảm bàn giao (guarantee of delivery):** khi lệnh send trả về, receiver *chắc chắn đã nhận*. Dùng khi cần tín hiệu đồng bộ.
- **Buffered = hàng đợi:** hấp thụ burst, nhưng **che giấu backpressure**. Buffer đầy thì vẫn block — buffer chỉ trì hoãn, không giải quyết chênh lệch tốc độ (xem chương 4 về backpressure).

**Anti-pattern phổ biến:** tăng buffer size để "fix" deadlock hoặc chậm. Buffer lớn chỉ giấu vấn đề đến khi production traffic cao hơn buffer — rồi nổ cùng lúc với hậu quả lớn hơn (memory phình + latency đuôi dài).

### 4.3. Luật sống còn với channel (đúc kết production)

1. **Ai tạo/ghi channel, người đó close.** Receiver không bao giờ close. Close một channel đang có sender khác → panic cả process.
2. Gửi vào channel `nil` hoặc đã close → block vĩnh viễn / panic. Kiểm soát lifecycle chặt chẽ.
3. **Goroutine leak** — bug số 1 trong code Go: goroutine block mãi mãi trên channel không ai đọc/ghi. Mỗi leak giữ stack + mọi thứ nó tham chiếu. Triệu chứng: memory tăng tuyến tính theo thời gian, `runtime.NumGoroutine()` tăng không giảm. Phòng ngừa: mọi goroutine phải có đường thoát (context cancel, timeout, close signal).

```go
// LEAK: nếu không ai đọc ch, goroutine kẹt vĩnh viễn
go func() { ch <- doWork() }()

// ĐÚNG: luôn có đường thoát
go func() {
    select {
    case ch <- doWork():
    case <-ctx.Done():   // đường thoát khi caller bỏ cuộc
    }
}()
```

---

## 5. Mutex, RWMutex, Atomic — khi nào dùng gì

Nguyên tắc chọn (theo thứ tự ưu tiên thực dụng):

1. **Channel** khi: chuyển *quyền sở hữu* dữ liệu, phân phối công việc, báo hiệu sự kiện, orchestration.
2. **Mutex** khi: bảo vệ *trạng thái nội bộ* của một struct (cache, counter, map). Đơn giản hơn và **nhanh hơn channel ~5-10x** cho việc này (channel bên trong cũng có mutex + chi phí scheduling).
3. **RWMutex** khi: đọc nhiều ghi ít (tỷ lệ đọc > ~90%) VÀ critical section đọc đủ dài. Lưu ý: RWMutex có chi phí bookkeeping cao hơn Mutex; với critical section ngắn, Mutex thường thắng. **Đo trước khi đổi.**
4. **atomic** khi: counter, flag, con trỏ swap đơn lẻ. Nhanh nhất (~vài ns) nhưng chỉ cho thao tác đơn. `atomic.Value`/`atomic.Pointer[T]` cho pattern "config reload không lock".

```go
// Pattern production: config hot-reload không lock
var config atomic.Pointer[Config]
// reader (hàng triệu lần/s, không lock):
cfg := config.Load()
// writer (hiếm):
config.Store(newCfg)
```

**Race detector:** `go test -race` / `go run -race`. Chạy CI với `-race` là **bắt buộc**, không thương lượng. Data race trong Go là undefined behavior thật sự — có thể làm hỏng nội tạng map, con trỏ, gây crash không tái hiện được.

**Anti-pattern kinh điển:** copy struct chứa mutex (mutex bị copy = hai khóa độc lập, mất tác dụng — `go vet` bắt được); lock rồi gọi hàm ngoài có thể lock lại cùng mutex (Go mutex **không reentrant** → deadlock).

---

## 6. WaitGroup và Context

### 6.1. WaitGroup — chờ nhóm goroutine

```go
var wg sync.WaitGroup
for _, task := range tasks {
    wg.Add(1)                    // Add TRƯỚC khi go — làm trong goroutine là race
    go func(t Task) {
        defer wg.Done()
        process(t)
    }(task)                      // Go <1.22: phải truyền biến vào, tránh capture
}
wg.Wait()
```

### 6.2. Context — hợp đồng hủy bỏ của cả hệ sinh thái

**Problem:** request đến API gateway → gọi 5 service → user đóng kết nối. Không có cơ chế chung, 5 call kia vẫn chạy hết, đốt tài nguyên vô ích. Nhân với 10K RPS → hệ thống làm việc vô ích khổng lồ, và khi downstream chậm, goroutine tồn đọng làm sập chính mình.

`context.Context` là câu trả lời chuẩn hóa: mang **deadline, cancellation signal, và request-scoped values** xuyên qua ranh giới API và goroutine.

```go
func (s *Service) HandleOrder(ctx context.Context, req OrderReq) error {
    ctx, cancel := context.WithTimeout(ctx, 2*time.Second)
    defer cancel()   // QUÊN dòng này = leak timer + context cha giữ tham chiếu

    if err := s.db.QueryContext(ctx, ...); err != nil {
        return fmt.Errorf("query order: %w", err)
    }
    ...
}
```

Quy tắc production:

1. `ctx` là **tham số đầu tiên** của mọi hàm có I/O hoặc có thể chậm. Không giấu trong struct.
2. Luôn `defer cancel()`.
3. Cancellation là **hợp tác**: hàm phải chủ động kiểm tra `ctx.Done()` trong vòng lặp dài / giữa các bước. Context không "giết" goroutine — Go không có cơ chế kill goroutine từ bên ngoài, đây là quyết định thiết kế (kill cưỡng bức để lại trạng thái hỏng).
4. `context.Value` chỉ cho request-scoped metadata (trace ID, user ID) — **không** truyền tham số nghiệp vụ qua đó (mất type safety, ẩn dependency).

---

## 7. Trade-off tổng hợp của mô hình Go

| Trục | Go chọn | Cái giá |
|---|---|---|
| Blocking-style vs async/await | Blocking-style, runtime lo phía dưới | Runtime phức tạp; mất kiểm soát chi tiết lịch chạy |
| Share memory vs message passing | Cả hai, khuyến khích message passing | Hai cơ chế = phải học khi nào dùng gì |
| Kill goroutine vs hợp tác | Hợp tác qua context | Goroutine "cứng đầu" không thể dừng từ ngoài |
| M:N scheduler vs 1:1 | M:N | Debugging khó hơn (stack trace hàng nghìn G); tương tác cgo/FFI phức tạp |

**Nếu làm ngược lại?** Nếu Go dùng async/await: ecosystem chia đôi sync/async như Rust/Python, mất tính đơn giản. Nếu 1:1 thread: mất khả năng chạy triệu concurrent task. Mô hình goroutine là lý do tồn tại của Go — bỏ nó thì Go chỉ là "C có GC".

---

## 8. Anti-patterns tổng hợp

1. **Goroutine không giới hạn:** `for _, x := range huge { go process(x) }` — 10 triệu item = 10 triệu goroutine = OOM hoặc nghẹt downstream. Dùng worker pool / semaphore (chương 4).
2. **Goroutine leak** do channel không có đường thoát (mục 4.3).
3. **Dùng channel làm mutex** — vòng vo, chậm, khó đọc. Bảo vệ state thì dùng mutex.
4. **`time.Sleep` để "đồng bộ hóa"** trong test hoặc code — flaky by design. Dùng channel/WaitGroup.
5. **Bỏ qua `-race` trong CI.**
6. **Panic trong goroutine con không recover** — panic ở bất kỳ goroutine nào không được recover sẽ **sập cả process**. Worker pool production phải có `defer recover()` ở đầu mỗi worker (và log + metric lại).

---

*Chương tiếp theo: [03 — Concurrency Patterns: Worker Pool, Pipeline, Fan-in/Fan-out, Backpressure](/series/nodejs-golang/03-go-concurrency-patterns/)*
