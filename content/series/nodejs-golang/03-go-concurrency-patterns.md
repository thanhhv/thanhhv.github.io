+++
title = "Bài 3 — Concurrency Patterns"
date = "2026-06-07T08:00:00+07:00"
draft = false
tags = ["backend", "golang"]
series = ["NodeJS & Golang"]
+++

# Concurrency Patterns — Worker Pool, Pipeline, Fan-in/Fan-out, Backpressure

---

## 1. Problem Statement

Goroutine và channel là **nguyên liệu**, không phải **kiến trúc**. Bài toán thực tế của mọi hệ thống concurrent:

1. **Giới hạn tài nguyên:** downstream (DB, API bên thứ ba) chỉ chịu được N kết nối đồng thời. Spawn goroutine tự do = tự DDoS downstream của chính mình.
2. **Chênh lệch tốc độ:** producer sinh 10K item/s, consumer xử lý 2K item/s. Không xử lý → memory phình → OOM.
3. **Điều phối kết quả:** chia việc cho N worker rồi gom kết quả, xử lý lỗi từng phần, hủy toàn bộ khi một phần fail.

Các pattern trong chương này là câu trả lời chuẩn hóa cho ba bài toán trên.

---

## 2. Worker Pool — giới hạn concurrency

### 2.1. Khi nào cần

Không phải để "tiết kiệm chi phí tạo goroutine" (goroutine rẻ). Mục đích thật: **giới hạn concurrency chạm vào tài nguyên hữu hạn** — connection DB, file descriptor, memory per task, rate limit của API ngoài.

### 2.2. Ví dụ production

```go
func processJobs(ctx context.Context, jobs <-chan Job, workers int) error {
    g, ctx := errgroup.WithContext(ctx) // golang.org/x/sync/errgroup
    for i := 0; i < workers; i++ {
        g.Go(func() error {
            for {
                select {
                case <-ctx.Done():
                    return ctx.Err()
                case job, ok := <-jobs:
                    if !ok {
                        return nil // channel đóng: hết việc
                    }
                    if err := handle(ctx, job); err != nil {
                        return fmt.Errorf("job %s: %w", job.ID, err)
                        // errgroup: lỗi đầu tiên → cancel ctx → mọi worker dừng
                    }
                }
            }
        })
    }
    return g.Wait()
}
```

`errgroup` là chuẩn de facto: gom lỗi + tự cancel + wait — thay cho bộ ba WaitGroup + error channel + cancel thủ công viết tay dễ sai.

Khi chỉ cần **giới hạn** chứ không cần pool cố định, semaphore gọn hơn:

```go
sem := make(chan struct{}, 100) // tối đa 100 đồng thời
for _, t := range tasks {
    sem <- struct{}{}
    go func(t Task) {
        defer func() { <-sem }()
        process(t)
    }(t)
}
```

Hoặc `errgroup.SetLimit(n)` (bản mới) — một dòng thay cả pool.

### 2.3. Sizing — nghệ thuật có công thức

- **CPU-bound:** workers = `runtime.GOMAXPROCS(0)`. Nhiều hơn chỉ tăng context switch.
- **I/O-bound:** workers = mức downstream chịu được (số connection DB pool, rate limit API). Công thức khởi điểm: `workers ≈ target_throughput × avg_latency` (Little's Law), rồi đo và điều chỉnh.
- **Ví dụ thất bại thực tế:** pool 500 worker ghi vào Postgres có `max_connections=100` → 400 worker chờ connection, timeout dây chuyền, retry storm. Worker pool phải sizing theo **mắt xích yếu nhất**.

---

## 3. Pipeline — chia xử lý thành giai đoạn

### 3.1. Mô hình

```
generator ──ch1──► stage1 (parse) ──ch2──► stage2 (enrich) ──ch3──► sink (write)
```

Mỗi stage là (nhóm) goroutine, nhận input channel, trả output channel. Lợi ích: các stage chạy **song song theo kiểu streaming** — item 2 được parse trong khi item 1 đang enrich; mỗi stage scale độc lập (stage chậm cho nhiều worker hơn); memory bounded theo buffer giữa các stage thay vì load toàn bộ dataset.

### 3.2. Quy tắc viết pipeline đúng

```go
func stage(ctx context.Context, in <-chan Item) <-chan Result {
    out := make(chan Result)
    go func() {
        defer close(out)              // (1) stage sở hữu out → stage close
        for item := range in {        // (2) tự thoát khi in đóng
            r, err := transform(item)
            if err != nil { /* đưa err vào Result, không nuốt */ }
            select {
            case out <- r:
            case <-ctx.Done():        // (3) đường thoát khi hủy
                return
            }
        }
    }()
    return out
}
```

Ba quy tắc đánh số ở trên là bất biến: thiếu (1) → consumer treo; thiếu (3) → goroutine leak khi caller bỏ cuộc. Lỗi truyền **trong band** (field trong Result) để giữ thứ tự và ngữ cảnh, hoặc dùng errgroup hủy toàn pipeline.

---

## 4. Fan-out / Fan-in

- **Fan-out:** nhiều goroutine cùng đọc một channel — cách tự nhiên nhất để parallel hóa một stage chậm (channel an toàn cho nhiều reader).
- **Fan-in:** gom nhiều channel về một:

```go
func fanIn[T any](ctx context.Context, chans ...<-chan T) <-chan T {
    out := make(chan T)
    var wg sync.WaitGroup
    for _, c := range chans {
        wg.Add(1)
        go func(c <-chan T) {
            defer wg.Done()
            for v := range c {
                select {
                case out <- v:
                case <-ctx.Done(): return
                }
            }
        }(c)
    }
    go func() { wg.Wait(); close(out) }() // close SAU KHI mọi nguồn cạn
    return out
}
```

**Lưu ý ngữ nghĩa:** fan-out làm mất thứ tự. Nếu cần giữ thứ tự (ví dụ xử lý event theo user), phải partition theo key (hash user ID → worker cố định) — giống hệt lý do Kafka có partition.

---

## 5. Backpressure — khái niệm sống còn

### 5.1. Bản chất vấn đề

Hệ thống ổn định chỉ khi **tốc độ vào ≤ tốc độ xử lý** về dài hạn. Khi vào > ra, chênh lệch phải đi đâu đó:

1. **Queue lớn dần** → memory phình → OOM → chết đột tử (tệ nhất: chết mang theo toàn bộ queue chưa xử lý).
2. **Chặn ngược nguồn** (backpressure) → nguồn chậm lại → hệ thống tự cân bằng.
3. **Chủ động từ chối** (load shedding) → trả 429, giữ latency tốt cho phần còn lại.

Go có backpressure **tự nhiên và miễn phí**: channel đầy thì sender block. Đây là ưu điểm kiến trúc lớn — trong Node.js, bạn phải chủ động gọi `pause()`/kiểm tra return của `write()`; trong Go, không làm gì cả là đã đúng.

### 5.2. Khi backpressure không lan ngược được

Nguồn là **network bên ngoài** (client HTTP, Kafka) không cảm nhận được channel đầy của bạn. Khi đó phải chọn chính sách rõ ràng:

```go
select {
case queue <- req:
    // nhận
default:
    // queue đầy → LOAD SHEDDING: từ chối nhanh, có kiểm soát
    http.Error(w, "server busy", http.StatusTooManyRequests)
    metrics.Inc("load_shed_total")
}
```

**Nguyên lý production quan trọng nhất chương này:** *từ chối sớm có kiểm soát luôn tốt hơn chết muộn không kiểm soát.* Hệ thống nhận mọi thứ rồi OOM là hệ thống mất cả 100% traffic; hệ thống shed 10% giữ được 90%.

### 5.3. Ví dụ thất bại kinh điển (dạng sự cố thật gặp ở nhiều công ty)

Service nhận webhook, đẩy vào `make(chan Event, 1_000_000)` "cho an toàn", 10 worker ghi DB. Ngày thường ổn. Ngày khuyến mãi: event vào 20K/s, DB ghi 5K/s → buffer 1M đầy sau ~66 giây → sender block → webhook timeout → phía gửi retry → **lượng vào tăng gấp đôi** → sập. 

Ba lỗi chồng nhau: buffer khổng lồ chỉ trì hoãn và che giấu vấn đề trong 66 giây quý giá (thay vì lộ ra ngay từ giây đầu qua metric); không có load shedding; không hiểu rằng retry của upstream biến quá tải thành quá tải hơn (retry storm). Fix đúng: buffer nhỏ (đủ hấp thụ jitter vài giây) + shed kèm `Retry-After` + metric độ sâu queue để autoscale worker/DB trước khi đầy.

---

## 6. Các pattern phụ trợ production

### 6.1. Timeout cho mọi thao tác chờ

```go
select {
case res := <-resultCh:
    return res, nil
case <-time.After(2 * time.Second): // lưu ý: time.After leak timer nếu gọi trong vòng lặp nóng → dùng time.NewTimer + Stop
    return nil, ErrTimeout
case <-ctx.Done():
    return nil, ctx.Err()
}
```

### 6.2. Or-done / first-response

Gọi N replica, lấy kết quả đầu tiên (hedge request — giảm p99 mạnh):

```go
ch := make(chan Result, len(replicas)) // buffer = N: loser không leak
for _, r := range replicas {
    go func(r Replica) { ch <- r.Query(ctx) }(r)
}
first := <-ch
```

Buffer đúng bằng N là chi tiết sống còn: các goroutine "thua cuộc" vẫn gửi được và thoát, không leak.

### 6.3. singleflight — chống cache stampede

Cache miss + 10K request đồng thời cùng một key → 10K query DB giống hệt nhau (thundering herd). `golang.org/x/sync/singleflight` gộp thành **một** call, mọi caller chờ chung kết quả:

```go
v, err, _ := group.Do(key, func() (any, error) {
    return db.LoadUser(ctx, key)
})
```

Đây là một trong những vũ khí chống sự cố cache hiệu quả nhất mà ít người biết.

---

## 7. Best Practices tổng hợp

1. Mọi goroutine phải trả lời được: **khi nào nó dừng, ai ra lệnh dừng?** Không trả lời được = leak đang chờ xảy ra.
2. Giới hạn concurrency theo tài nguyên yếu nhất, không theo "cảm giác".
3. Buffer channel nhỏ và có lý do; buffer lớn phải kèm chính sách khi đầy.
4. Đo `runtime.NumGoroutine()` và độ sâu queue thành metric — hai chỉ số cảnh báo sớm giá trị nhất.
5. Ưu tiên `errgroup`/`singleflight`/`semaphore` từ `golang.org/x/sync` thay vì tự viết.
6. Test concurrency với `-race` và với tải (không chỉ happy path một goroutine).

## 8. Khi nào KHÔNG dùng các pattern này

- Xử lý tuần tự đủ nhanh → **đừng thêm concurrency**. Concurrency là công cụ trả chi phí phức tạp để mua throughput/latency; xử lý 100 item mất 50ms thì pipeline 4 stage chỉ thêm bug.
- Cần thứ tự tuyệt đối toàn cục → fan-out phá thứ tự; cân nhắc xử lý đơn luồng theo partition.
- Job dài hàng phút, cần survive restart → worker pool in-memory sai công cụ; dùng message queue bền vững (Kafka, RabbitMQ, river/asynq) — queue trong RAM chết là mất việc.

---

*Chương tiếp theo: [04 — Runtime Internals: Scheduler, GC, Memory Allocator, Netpoller](/series/nodejs-golang/04-go-runtime-internals/)*
