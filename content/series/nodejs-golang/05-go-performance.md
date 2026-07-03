+++
title = "Bài 5 — Performance Engineering trong Go"
date = "2026-06-11T08:00:00+07:00"
draft = false
tags = ["backend", "golang"]
series = ["NodeJS & Golang"]
+++

# Performance Engineering trong Go — Benchmark, pprof, Optimization

---

## 1. Problem Statement — và nguyên tắc số 0

**Nguyên tắc số 0: không đo thì không tối ưu.** Trực giác về hiệu năng sai nhiều hơn đúng, kể cả với engineer giỏi. Chi phí của tối ưu mù: code khó đọc hơn để "tối ưu" một đoạn chiếm 0.1% thời gian, trong khi 80% thời gian nằm ở một query N+1 không ai nhìn.

Quy trình chuẩn: **đặt mục tiêu định lượng (SLO) → đo → tìm bottleneck lớn nhất → sửa → đo lại → lặp.** Dừng khi đạt SLO, không phải khi "hết chỗ tối ưu".

---

## 2. Benchmark — đo vi mô đúng cách

### 2.1. Công cụ chuẩn

```go
func BenchmarkParseJSON(b *testing.B) {
    data := loadFixture()
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        parse(data)
    }
}
// go test -bench=. -benchmem -count=10 > new.txt
// benchstat old.txt new.txt   ← so sánh CÓ THỐNG KÊ, không so 1 lần chạy
```

`-benchmem` hiện `allocs/op` và `B/op` — với Go, **số allocation thường quan trọng ngang thời gian** vì nó là thuế GC trả sau.

### 2.2. Bẫy benchmark kinh điển

1. **Compiler xóa mất code chết:** kết quả không dùng → dead code elimination → đo được 0.2ns "nhanh như thần". Fix: gán kết quả vào biến package-level hoặc `runtime.KeepAlive`.
2. **So sánh single-run:** nhiễu máy (thermal, scheduler OS) ±10% là thường. Luôn `-count=10` + `benchstat` (cho p-value).
3. **Benchmark trên laptop, kết luận cho server:** khác CPU, khác NUMA, khác turbo. Số tuyệt đối chỉ tin khi đo trên hardware tương đương production.
4. **Microbenchmark nói dối về hệ thống:** hàm nhanh hơn 30% trong benchmark có thể vô nghĩa khi hệ thống nghẽn ở I/O. Micro-benchmark chọn giải pháp, **load test xác nhận giá trị**.

---

## 3. pprof — kính hiển vi production

### 3.1. Kích hoạt

```go
import _ "net/http/pprof" // đăng ký handler vào DefaultServeMux
go http.ListenAndServe("localhost:6060", nil) // CHỈ bind localhost/internal!
```

**Cảnh báo bảo mật:** expose pprof ra internet = lộ source path, heap content, và tự mở cửa DoS. Bind nội bộ, chặn ở ingress.

### 3.2. Các loại profile và câu hỏi chúng trả lời

| Profile | Câu hỏi | Lệnh |
|---|---|---|
| CPU | Thời gian CPU đi đâu? | `go tool pprof http://:6060/debug/pprof/profile?seconds=30` |
| Heap (inuse) | Cái gì đang giữ memory? | `.../heap` |
| Heap (alloc) | Cái gì cấp phát nhiều nhất (thuế GC)? | `.../heap` rồi `-sample_index=alloc_space` |
| Goroutine | 50K goroutine đang kẹt ở đâu? | `.../goroutine?debug=2` |
| Block | Chờ channel/select tốn bao lâu? | cần `runtime.SetBlockProfileRate` |
| Mutex | Lock nào contention? | cần `runtime.SetMutexProfileFraction` |
| Trace | Chuyện gì xảy ra theo thời gian thực? (GC, scheduling, syscall) | `go tool trace` |

Cách đọc hiệu quả nhất: `pprof -http=:8080 profile.pb.gz` → **flame graph**. Nhìn từ dưới lên: khung rộng = tốn nhiều; ưu tiên sửa khung rộng nhất thuộc code mình.

**Continuous profiling** (Pyroscope, Parca, Grafana Phlare, Cloud Profiler): profile luôn-bật với overhead ~1-2%, cho phép so sánh "hôm nay vs tuần trước" khi p99 xấu đi từ từ — quý hơn vàng khi điều tra hồi quy hiệu năng.

### 3.3. execution trace — khi pprof không đủ

pprof trả lời "cái gì tốn", trace trả lời "**tại sao chậm dù CPU nhàn**": goroutine chờ nhau, GC chen ngang, chỉ 2/16 P có việc. `curl :6060/debug/pprof/trace?seconds=5` → `go tool trace trace.out`. Nặng nhưng là công cụ duy nhất thấy được scheduling latency.

---

## 4. Chiến lược tối ưu theo tầng (từ rẻ đến đắt)

### Tầng 1 — Thuật toán & I/O (ROI lớn nhất, luôn làm trước)

- N+1 query → batch/JOIN. Một dòng code sửa được điều mà không tối ưu allocation nào bì kịp.
- Cache kết quả đắt (kèm singleflight — chương 3).
- O(n²) → O(n log n). Map thay vòng lặp tìm kiếm lồng nhau.

### Tầng 2 — Giảm allocation (đặc sản Go)

```go
// 1. Preallocate khi biết size
s := make([]Item, 0, len(input))       // tránh grow-copy nhiều lần

// 2. Tái sử dụng buffer qua sync.Pool (chương 4)

// 3. strings.Builder thay + trong vòng lặp
var b strings.Builder
b.Grow(estimatedLen)
for _, p := range parts { b.WriteString(p) }

// 4. Tránh []byte ↔ string chuyển đổi không cần (mỗi lần là một copy + alloc)

// 5. Chú ý escape: trả value thay pointer cho struct nhỏ;
//    kiểm tra bằng -gcflags="-m"
```

JSON là hot spot phổ biến: `encoding/json` dùng reflection, alloc nhiều. Khi profile chỉ vào JSON: cân nhắc `json/v2` (Go 1.25+), `sonic`, hoặc đổi format (protobuf) — nhưng chỉ khi profile chỉ vào đó.

### Tầng 3 — Concurrency & lock

- Mutex contention (thấy qua mutex profile) → thu nhỏ critical section; shard lock (N mutex theo hash key); RWMutex nếu đọc áp đảo; atomic cho counter.
- False sharing: hai counter atomic của hai goroutine nằm cùng cache line 64B → hai core "giật" cache line qua lại. Fix: pad struct. Chỉ quan tâm khi profile chứng minh — đây là tối ưu cấp cuối.

### Tầng 4 — Cơ học vi mô (chỉ khi SLO còn chưa đạt và profile chỉ đích)

Bounds-check elimination, sắp field struct giảm padding, inline hint, tránh interface trong hot loop (dynamic dispatch chặn inline). ROI thấp, chi phí đọc-hiểu cao — cần comment giải thích và benchmark đính kèm trong PR.

---

## 5. Latency Analysis — tư duy percentile

- **Trung bình là số dối trá.** SLO đặt trên p50/p95/p99. Một service p50=5ms, p99=2s là service tồi cho 1% người dùng — và với trang gọi 100 API, gần **63%** người dùng dính ít nhất một lần p99 (1 − 0.99¹⁰⁰).
- **Nguồn latency đuôi trong Go xếp theo tần suất gặp:** GC assist khi allocation cao; goroutine chờ P khi CPU bão hòa (scheduling latency — thấy qua trace); connection pool cạn; retry ẩn trong client; DNS lookup không cache; TLS handshake không keep-alive.
- **Coordinated omission:** tool load test chờ response xong mới gửi tiếp sẽ *che* latency thật khi server nghẽn. Dùng tool open-model phát tải theo nhịp cố định (vegeta, k6 arrival-rate, wrk2) để thấy con số trung thực.

---

## 6. Case study tổng hợp (mô típ thật, đã gặp nhiều biến thể)

**Triệu chứng:** API p99 từ 80ms lên 400ms sau khi traffic tăng 3x; CPU 60%; RAM ổn.

1. CPU profile → 35% `runtime.gcAssistAlloc` + `mallocgc`. → GC assist: allocation rate là thủ phạm, không phải "code chậm".
2. Alloc profile → 70% allocation từ hàm serialize log: mỗi request marshal cả struct request vào log JSON ở level Debug... nhưng logger vẫn **evaluate tham số** dù level Info đang bật.
3. Fix: guard `if logger.Enabled(Debug)` + chuyển sang zerolog (zero-alloc). Allocation giảm 8x.
4. Kết quả: p99 về 90ms, CPU 35%. **Không một dòng logic nghiệp vụ nào bị đổi.**

Bài học: (1) đuôi latency của Go thường là chuyện allocation; (2) logging là nguồn allocation bị bỏ quên số 1; (3) không có profile thì bước 1 đã là "chắc tại DB" và mất một tuần vô ích.

---

## 7. Anti-patterns

1. Tối ưu không có profile ("tôi nghĩ chỗ này chậm").
2. Tin microbenchmark tuyệt đối, bỏ qua load test.
3. `sync.Pool` mọi thứ / hoặc pool object chứa tài nguyên cần đóng.
4. Chỉnh GOGC lung tung thay vì giảm allocation gốc.
5. Bỏ metric latency percentile, chỉ nhìn average.
6. Tối ưu vi mô trong PR không kèm benchmark chứng minh — reviewer không thể phân biệt tối ưu với cargo cult.

---

*Chương tiếp theo: [06 — Production Engineering với Go](/series/nodejs-golang/06-go-production-engineering/)*
