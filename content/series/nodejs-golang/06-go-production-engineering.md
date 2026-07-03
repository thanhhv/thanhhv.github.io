+++
title = "Bài 6 — Production Engineering với Go"
date = "2026-06-13T08:00:00+07:00"
draft = false
tags = ["backend", "golang"]
series = ["NodeJS & Golang"]
+++

# Production Engineering với Go — Resilience Patterns

> Graceful Shutdown, Retry, Circuit Breaker, Rate Limiter, Idempotency, Distributed Lock

---

## 1. Problem Statement

Code chạy đúng trên máy dev chỉ là 30% công việc. Production khác ở chỗ: **mọi thứ đều fail** — network đứt giữa chừng, downstream chậm, pod bị kill bất kỳ lúc nào, request bị gửi hai lần. Các pattern trong chương này tồn tại vì một sự thật: *hệ thống phân tán không thể tránh lỗi, chỉ có thể thiết kế để lỗi không lan rộng*.

---

## 2. Graceful Shutdown

### 2.1. Tại sao

Kubernetes gửi SIGTERM khi rolling update / scale down / node drain — tức là **mỗi lần deploy**. Không xử lý → process chết ngay giữa chừng: request đang xử lý bị đứt (user thấy 502), transaction dở dang, message lấy khỏi queue chưa ack bị xử lý lại (hoặc mất). Với dịch vụ deploy 10 lần/ngày, shutdown cẩu thả = 10 sự cố nhỏ/ngày.

### 2.2. Trình tự chuẩn

```go
func main() {
    srv := &http.Server{Addr: ":8080", Handler: routes()}
    go func() {
        if err := srv.ListenAndServe(); err != http.ErrServerClosed {
            log.Fatal(err)
        }
    }()

    stop := make(chan os.Signal, 1)
    signal.Notify(stop, syscall.SIGTERM, syscall.SIGINT)
    <-stop

    // 1. Báo readiness probe fail TRƯỚC → LB ngừng đưa traffic mới
    healthy.Store(false)
    time.Sleep(3 * time.Second) // chờ LB/kube-proxy cập nhật endpoint

    // 2. Ngừng nhận, chờ request đang chạy xong (có deadline)
    ctx, cancel := context.WithTimeout(context.Background(), 25*time.Second)
    defer cancel()
    _ = srv.Shutdown(ctx)

    // 3. Dừng consumer, flush buffer, đóng DB pool — theo thứ tự ngược khởi tạo
    worker.Stop(ctx)
    db.Close()
}
```

Chi tiết hay bị bỏ sót: (1) **fail readiness trước rồi mới ngừng nhận** — nếu ngừng nhận ngay, LB vẫn gửi request tới trong vài giây và nhận connection refused; (2) tổng thời gian phải < `terminationGracePeriodSeconds` của K8s (mặc định 30s), không thì SIGKILL cắt ngang; (3) consumer message queue: ngừng **lấy** message mới trước, xử lý nốt, ack, rồi mới đóng.

---

## 3. Retry — con dao hai lưỡi sắc nhất

### 3.1. Làm đúng

```go
func retry(ctx context.Context, maxAttempts int, fn func() error) error {
    backoff := 100 * time.Millisecond
    for attempt := 1; ; attempt++ {
        err := fn()
        if err == nil || !isRetryable(err) || attempt == maxAttempts {
            return err
        }
        jitter := time.Duration(rand.Int64N(int64(backoff))) // FULL jitter
        select {
        case <-time.After(jitter):
            backoff = min(backoff*2, 5*time.Second) // exponential, có trần
        case <-ctx.Done():
            return ctx.Err()
        }
    }
}
```

Bốn thành phần bắt buộc: **phân loại lỗi** (chỉ retry lỗi tạm thời: timeout, 503, connection reset — không retry 400, 401, business error), **exponential backoff**, **jitter** (không có jitter → mọi client retry cùng nhịp → sóng tải đồng bộ dập downstream đúng lúc nó gượng dậy), **giới hạn tổng** (attempts + budget thời gian qua context).

### 3.2. Retry storm — cách hệ thống tự giết mình

Chuỗi A→B→C, mỗi tầng retry 3 lần. C chậm → B retry 3× → A retry 3× → C nhận **9x tải** đúng lúc yếu nhất → chết hẳn. Đây là mô típ của rất nhiều sự cố lớn trong ngành.

Quy tắc: **retry ở một tầng duy nhất** (thường tầng gần client nhất hoặc tầng có ngữ cảnh nghiệp vụ), các tầng dưới fail-fast + timeout ngắn. Kèm **retry budget** (ví dụ: retry ≤ 10% tổng request) — vượt budget thì thôi retry, trả lỗi luôn.

Và điều kiện tiên quyết thường bị quên: **thao tác được retry phải idempotent** (mục 6) — retry một lệnh trừ tiền không idempotent là tự tạo sự cố tài chính.

---

## 4. Circuit Breaker

### 4.1. Vấn đề: chờ còn tệ hơn lỗi

Downstream chết hẳn thì connection refused trả về ngay — rẻ. Downstream **chậm** mới nguy hiểm: mỗi call treo 30s giữ một goroutine + connection + memory. 1000 RPS × 30s = 30.000 goroutine treo → cạn pool → service của **bạn** chết theo. Lỗi lan ngược dòng (cascading failure).

### 4.2. Cơ chế

```
          lỗi vượt ngưỡng (vd 50% trong 10s)
 CLOSED ────────────────────────────► OPEN
 (cho qua,                        (chặn NGAY, trả lỗi
  đếm lỗi)                         không gọi thật — fail fast)
    ▲                                   │ hết cooldown (vd 30s)
    │ thành công đủ N lần               ▼
    └────────────────────────────  HALF-OPEN
                                  (cho qua vài request thăm dò;
                                   fail → OPEN lại)
```

Giá trị cốt lõi: khi mở, caller nhận lỗi trong **microseconds** thay vì treo 30s — tài nguyên được giải phóng, và downstream được "nghỉ" để hồi phục. Circuit breaker là backpressure dạng nhị phân.

Thực tế Go: dùng `sony/gobreaker` hoặc `failsafe-go`; breaker **theo từng downstream** (không dùng chung — DB chết không nên chặn call tới cache); khi breaker mở phải có **fallback có chủ đích**: giá trị cache cũ, giá trị mặc định, degrade tính năng — "trả lỗi 500" cũng là một fallback hợp lệ nếu được chọn có ý thức.

Metric bắt buộc: trạng thái breaker (gauge) + số lần chuyển trạng thái — breaker mở là tín hiệu sự cố sớm hơn mọi alert latency.

---

## 5. Rate Limiter

### 5.1. Hai phía của rate limit

- **Bảo vệ mình (server-side):** chặn client hung hãn, giữ SLO cho phần còn lại.
- **Bảo vệ downstream (client-side):** API đối tác cho 100 req/s — tự giới hạn mình trước khi bị họ ban.

### 5.2. Thuật toán

| Thuật toán | Cơ chế | Đặc điểm |
|---|---|---|
| Token bucket | Nạp token đều đặn, request tiêu token | Cho phép burst có kiểm soát — chuẩn cho API. `golang.org/x/time/rate` |
| Sliding window | Đếm trong cửa sổ trượt | Mượt, tốn bộ nhớ hơn; chuẩn cho distributed (Redis) |
| Fixed window | Đếm theo phút/giây cố định | Đơn giản nhưng dính burst 2x tại biên cửa sổ |

```go
limiter := rate.NewLimiter(rate.Limit(100), 200) // 100 rps, burst 200
if !limiter.Allow() {
    w.Header().Set("Retry-After", "1")
    http.Error(w, "rate limited", http.StatusTooManyRequests)
    return
}
```

Phân biệt `Allow()` (từ chối ngay — cho server bảo vệ mình) vs `Wait(ctx)` (xếp hàng chờ — cho client tự kiềm chế với downstream). Chọn nhầm chiều là sai ngữ nghĩa.

**Distributed rate limit** (nhiều instance chung một quota): cần trạng thái chung — Redis + Lua script (atomic), hoặc chấp nhận xấp xỉ chia quota tĩnh cho từng instance. Trade-off: chính xác toàn cục (thêm 1 network hop mỗi request) vs xấp xỉ cục bộ (rẻ, lệch khi autoscale). Đa số trường hợp: làm ở API gateway là đủ và rẻ nhất.

---

## 6. Idempotency

### 6.1. Tại sao đây là pattern quan trọng nhất trong hệ phân tán

Sự thật nền tảng: **"exactly-once delivery" không tồn tại** trên network không tin cậy. Client gửi request trừ tiền, timeout — tiền đã trừ hay chưa? *Không thể biết.* Lựa chọn duy nhất: không retry (mất giao dịch) hoặc retry (nguy cơ trừ hai lần). Idempotency biến retry thành an toàn: gọi N lần, hiệu ứng như 1 lần — "at-least-once delivery + idempotent processing = effectively exactly-once".

### 6.2. Triển khai với idempotency key

```go
func (s *PaymentService) Charge(ctx context.Context, req ChargeReq) (*Charge, error) {
    // Client sinh key (UUID) cho MỖI Ý ĐỊNH giao dịch, giữ nguyên khi retry
    // UNIQUE constraint trong DB là trọng tài cuối cùng (atomic thật sự)
    tx, _ := s.db.BeginTx(ctx, nil)
    defer tx.Rollback()

    _, err := tx.Exec(`INSERT INTO idempotency_keys (key, status) VALUES ($1,'processing')`, req.IdemKey)
    if isUniqueViolation(err) {
        return s.loadExistingResult(ctx, req.IdemKey) // đã/đang xử lý → trả kết quả cũ
    }
    charge, err := s.doCharge(ctx, tx, req)          // nghiệp vụ + lưu kết quả CÙNG transaction
    if err != nil { return nil, err }
    _ = tx.Commit()
    return charge, nil
}
```

Chi tiết quyết định thành bại: (1) check-key và ghi-kết-quả phải nằm **cùng transaction** với nghiệp vụ — tách rời là mở cửa sổ race; (2) "check rồi insert" bằng hai lệnh là race condition — dùng UNIQUE constraint làm cơ chế chính; (3) key có TTL (24h) để bảng không phình vô hạn; (4) trạng thái `processing` cần xử lý: request thứ hai đến khi cái đầu chưa xong → trả 409/chờ, đừng xử lý song song.

Nguyên tắc áp dụng: mọi endpoint **ghi** có thể bị retry (bởi client, LB, hoặc message queue redelivery) đều cần chiến lược idempotency. Consumer Kafka/RabbitMQ: dùng message ID làm key tự nhiên.

---

## 7. Distributed Lock

### 7.1. Trước hết: cố gắng không cần nó

Distributed lock là công cụ dễ sai nhất chương này. Trước khi dùng, kiểm tra các giải pháp đơn giản hơn: UNIQUE constraint (như trên), `SELECT ... FOR UPDATE` (lock theo row, transactional), optimistic locking bằng version column, partition công việc theo key (mỗi instance sở hữu một dải — không cần lock). 80% trường hợp "cần distributed lock" giải được bằng một trong bốn cách này, an toàn hơn.

### 7.2. Khi thật sự cần — và các bẫy

Use case hợp lệ: cron job chạy trên N instance nhưng chỉ được 1 instance thực thi; migration một lần.

```go
// Redis: SET lock_key <random_token> NX EX 30
// Nhả lock bằng Lua script: chỉ xóa nếu token là CỦA MÌNH
```

Hai bẫy chết người:

1. **Lock hết hạn khi việc chưa xong:** process A giữ lock TTL 30s, bị GC pause/network chậm 35s → lock hết hạn → B lấy lock → **A và B cùng chạy**. TTL + random token + kiểm tra token khi nhả giúp giảm, nhưng không loại trừ hoàn toàn — khoảng giữa "kiểm tra lock" và "hành động" luôn tồn tại.
2. **Kết luận quan trọng:** lock phân tán trên hệ không có fencing là **tối ưu hóa xác suất, không phải bảo đảm tuyệt đối**. Tác vụ mà chạy-hai-lần gây hậu quả nghiêm trọng thì lock **không đủ** — tầng dưới vẫn phải idempotent hoặc dùng **fencing token** (số tăng đơn điệu từ lock service, storage từ chối token cũ). Redlock của Redis bị tranh cãi đúng vì điểm này (phân tích của Martin Kleppmann). Cần bảo đảm mạnh → dùng hệ có consensus: etcd (lease + revision làm fencing token) hoặc ZooKeeper.

---

## 8. Ghép các pattern — thứ tự lớp giáp

```
Request → [Rate limiter] → [Circuit breaker] → [Timeout (context)] → [Retry] → Call thật
           chặn quá tải      chặn downstream ốm    chặn treo          cứu lỗi thoáng qua
                                                                     (chỉ khi idempotent)
```

Thứ tự có ý nghĩa: retry nằm **trong** circuit breaker (lỗi do retry vẫn được breaker đếm... hay ngược lại tùy chiến lược — quyết định có ý thức và ghi lại); timeout luôn trong cùng — mọi retry đều chịu deadline tổng từ context. Toàn bộ chuỗi này nên đóng gói một lần (middleware/decorator quanh client) thay vì rải rác từng chỗ gọi.

## 9. Anti-patterns tổng hợp

1. Retry không jitter, không phân loại lỗi, ở mọi tầng.
2. Circuit breaker không có fallback — mở ra rồi... vẫn 500 toàn tập nhưng giờ khó hiểu hơn.
3. Idempotency "check rồi insert" hai bước không transaction.
4. Distributed lock cho việc mà UNIQUE constraint giải được.
5. Graceful shutdown không fail readiness trước → vẫn rớt request mỗi lần deploy.
6. Timeout đồng loạt bằng nhau ở mọi tầng (tầng ngoài phải ≥ tổng tầng trong, không thì tầng ngoài luôn timeout trước một cách vô nghĩa).

---

*Chương tiếp theo: [07 — Software Architecture với Go: DI, Clean/Hexagonal Architecture, DDD, Event-driven](/series/nodejs-golang/07-go-architecture/)*
