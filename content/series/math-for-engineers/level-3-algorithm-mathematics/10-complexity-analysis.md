+++
title = "Chương 10 — Complexity Analysis: Ngôn ngữ đo lường thuật toán"
date = "2026-07-20T08:40:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 3 – Algorithm Mathematics
> Yêu cầu trước: Chương 03 (Proof Techniques), Chương 06 (Recurrence)

---

## 1. Problem Statement

Năm 2010, một kỹ sư tại một công ty thương mại điện tử viết đoạn code kiểm tra sản phẩm trùng lặp: với mỗi sản phẩm, duyệt qua toàn bộ danh sách để tìm bản sao. Code chạy hoàn hảo trên môi trường dev với 1.000 sản phẩm — mất 2ms. Sáu tháng sau, catalog đạt 2 triệu sản phẩm, và cron job đó chiếm trọn một CPU core trong 4 giờ mỗi đêm.

Không có bug. Không có lỗi logic. Code "đúng" theo mọi unit test. Vấn đề nằm ở chỗ khác: **chi phí của thuật toán tăng theo bình phương kích thước dữ liệu**, và không có cách nào phát hiện điều đó bằng cách chạy thử trên dữ liệu nhỏ.

Đây chính là bài toán mà Complexity Analysis sinh ra để giải quyết:

**Làm sao dự đoán hành vi của một thuật toán trên dữ liệu lớn, mà không cần chạy nó trên dữ liệu lớn?**

Nếu không có công cụ này, lập trình viên chỉ còn hai lựa chọn tồi:

1. **Benchmark mọi thứ trên dữ liệu production** — đắt, chậm, và thường bất khả thi (bạn không thể benchmark trên 10 tỷ record trước khi có 10 tỷ record).
2. **Đoán mò** — và trực giác con người cực kỳ tệ với các hàm tăng trưởng phi tuyến.

## 2. Trực giác

### Câu chuyện tờ giấy gấp đôi

Hãy bắt đầu bằng một câu hỏi kinh điển: gấp đôi một tờ giấy dày 0.1mm bao nhiêu lần thì chồng giấy chạm tới Mặt Trăng?

Trực giác nói: hàng triệu lần. Đáp án thực: **42 lần**. Vì mỗi lần gấp *nhân đôi* độ dày: 2⁴² × 0.1mm ≈ 440.000 km.

Trực giác con người là **tuyến tính**. Chúng ta ước lượng tốt các đại lượng cộng dồn ("mỗi ngày tiết kiệm 100k, một năm được 36.5 triệu"), nhưng thất bại hoàn toàn với tăng trưởng theo cấp số nhân — và cả chiều ngược lại, logarit.

### Áp vào code

Xét hai đoạn code cùng giải bài toán "tìm phần tử trong danh sách đã sắp xếp":

```go
// Cách 1: Linear scan — duyệt từ đầu
func linearSearch(a []int, target int) int {
    for i, v := range a {
        if v == target {
            return i
        }
    }
    return -1
}

// Cách 2: Binary search — mỗi bước loại bỏ một nửa
func binarySearch(a []int, target int) int {
    lo, hi := 0, len(a)-1
    for lo <= hi {
        mid := lo + (hi-lo)/2 // tránh overflow, xem chương 19
        switch {
        case a[mid] == target:
            return mid
        case a[mid] < target:
            lo = mid + 1
        default:
            hi = mid - 1
        }
    }
    return -1
}
```

Với 1.000 phần tử, cả hai đều "nhanh" — vài trăm nanosecond, mắt thường không phân biệt được. Nhưng nhìn số bước tối đa khi dữ liệu tăng:

| n | Linear (n bước) | Binary (log₂n bước) |
|---|---|---|
| 1.000 | 1.000 | 10 |
| 1.000.000 | 1.000.000 | 20 |
| 1.000.000.000 | 1.000.000.000 | 30 |

Dữ liệu tăng **1 triệu lần**, binary search chỉ tốn thêm **20 bước**. Đây là điều trực giác tuyến tính không bao giờ tự nhận ra: sự khác biệt giữa hai thuật toán không phải là "nhanh gấp đôi" mà là **khác nhau về bản chất tăng trưởng**.

### Giới hạn của trực giác — và của benchmark

Vậy tại sao không benchmark là xong? Vì benchmark chỉ trả lời "nhanh bao nhiêu **tại kích thước này, trên máy này**". Nó không trả lời:

- Khi dữ liệu tăng 100 lần, thời gian tăng bao nhiêu lần?
- Con số đo được là do thuật toán, hay do CPU/cache/compiler?
- Thuật toán A nhanh hơn B ở n = 10⁴, nhưng ở n = 10⁸ thì sao?

Chúng ta cần một công cụ **trừu tượng hóa khỏi phần cứng** và chỉ giữ lại điều bất biến: *hình dạng của hàm tăng trưởng*. Đó là việc của toán học.

## 3. First Principles

### Bước trừu tượng hóa thứ nhất: đếm phép toán thay vì đo giây

Thời gian chạy phụ thuộc CPU, cache, compiler, nhiệt độ phòng server... Nhưng **số phép toán cơ bản** mà thuật toán thực hiện thì không. Ta định nghĩa hàm chi phí:

> T(n) = số phép toán cơ bản khi input có kích thước n

Với linear search trong trường hợp xấu nhất: T(n) = n lần so sánh (cộng vài phép gán). Với binary search: T(n) ≈ log₂n.

### Bước trừu tượng hóa thứ hai: chỉ giữ hình dạng tăng trưởng

Giả sử đếm chi li, ta được T(n) = 3n² + 5n + 20. Khi n = 10⁶:

- 3n² = 3.000.000.000.000
- 5n = 5.000.000
- 20 = 20

Thành phần n² chiếm **99.9998%** tổng chi phí. Các thành phần còn lại — và cả hệ số 3 — không thay đổi *câu trả lời cho câu hỏi ta quan tâm*: "khi n tăng 10 lần, chi phí tăng ~100 lần". Vì vậy toán học cho phép ta vứt bỏ chúng một cách **có kỷ luật**, và đó chính là ký hiệu tiệm cận (asymptotic notation).

Điểm mấu chốt cần hiểu: Big-O không phải là "cách viết tắt lười biếng". Nó là một **phép trừu tượng hóa có chủ đích** — ta chấp nhận mất thông tin về hằng số để đổi lấy một bất biến đúng trên mọi phần cứng, mọi ngôn ngữ, mọi compiler. Giống như khi thiết kế API, ta giấu implementation detail để giữ contract ổn định.

### Nếu không có nó thì sao?

Không có ký hiệu tiệm cận, mọi thảo luận về hiệu năng thuật toán đều phải kèm theo model CPU, phiên bản compiler và cờ tối ưu hóa. Paper nghiên cứu năm 1975 sẽ vô dụng năm 2026. Big-O là lý do một kết quả về quicksort từ thập niên 60 vẫn đúng nguyên vẹn trên chip M-series hôm nay.

## 4. Mathematical Model

### Định nghĩa Big-O

> f(n) = O(g(n)) khi và chỉ khi tồn tại hằng số c > 0 và n₀ sao cho:
> **f(n) ≤ c·g(n) với mọi n ≥ n₀**

Diễn giải từng thành phần — vì mỗi mảnh của định nghĩa đều trả lời một câu hỏi thực tế:

- **c·g(n)**: "nhân với hằng số nào đó" — chính là chỗ ta *hợp pháp hóa* việc bỏ qua hệ số. 3n² và 300n² đều là O(n²).
- **n ≥ n₀**: "từ một điểm nào đó trở đi" — ta chỉ quan tâm hành vi khi n lớn. Với n nhỏ, mọi thuật toán đều nhanh; sự khác biệt chỉ lộ ra ở quy mô.
- **≤**: Big-O là **chặn trên** (upper bound). Nói "thuật toán này là O(n²)" nghĩa là "không tệ hơn n²", chứ không phải "đúng bằng n²". Linear search cũng là O(n²) — về mặt kỹ thuật đúng, nhưng không có ích.

Trực quan hóa:

```
chi phí
  │                    c·g(n)
  │                   ╱
  │                  ╱   ← từ n₀ trở đi, f(n) luôn
  │                 ╱      nằm dưới c·g(n)
  │          ✕ ✕ ╱✕
  │        ✕   ╱
  │      ✕  ╱     f(n)
  │    ✕ ╱
  │   ╱✕
  └──┼──────────────────────── n
     n₀
```

### Big-Omega và Big-Theta

Big-O một mình chưa đủ, vì nó chỉ nói "không tệ hơn". Bộ ba đầy đủ:

| Ký hiệu | Ý nghĩa | Câu hỏi nó trả lời |
|---|---|---|
| O(g) | chặn trên: f ≤ c·g | "Tệ nhất là bao nhiêu?" |
| Ω(g) | chặn dưới: f ≥ c·g | "Ít nhất phải tốn bao nhiêu?" |
| Θ(g) | chặn chặt: cả hai | "Chính xác tăng trưởng thế nào?" |

Ω đặc biệt quan trọng vì nó cho phép chứng minh **giới hạn của mọi thuật toán**, không chỉ một thuật toán cụ thể. Ví dụ kinh điển: *mọi* thuật toán sort dựa trên so sánh cần Ω(n log n) phép so sánh (chứng minh ở chương 16 — dựa trên việc cây quyết định phải có n! lá). Đây là kiểu tri thức chỉ toán học mang lại được: benchmark không bao giờ chứng minh được "không tồn tại cách nhanh hơn".

### Worst case, Average case và tại sao phải phân biệt

T(n) thực ra không phải là một hàm — cùng kích thước n có nhiều input khác nhau với chi phí khác nhau. Ta phải chọn góc nhìn:

- **Worst case**: max trên mọi input kích thước n. Dùng cho cam kết SLA, hệ thống real-time, và phòng thủ trước adversarial input.
- **Average case**: kỳ vọng (expected value — chương 11) trên phân phối input. Dùng khi input "ngẫu nhiên tự nhiên". Quicksort là O(n²) worst case nhưng Θ(n log n) average case — và đó là lý do nó vẫn thống trị.
- **Amortized**: chi phí trung bình trên một *chuỗi* thao tác, ngay cả khi từng thao tác riêng lẻ có thể đắt. Đây là khái niệm hay bị hiểu nhầm nhất, và đáng để phân tích kỹ.

### Amortized Analysis: trường hợp slice của Go

Khi bạn `append` vào một slice Go đã đầy, runtime cấp phát mảng mới (lớn hơn ~2 lần khi slice nhỏ, ~1.25 lần khi lớn) và copy toàn bộ. Thao tác append đó tốn O(n). Vậy có nên nói "append là O(n)" không?

Không — và đây là chỗ toán học tinh tế hơn trực giác. Xét chuỗi n lần append với chiến lược nhân đôi capacity, tổng chi phí copy là:

> 1 + 2 + 4 + 8 + ... + n/2 + n < 2n

Tổng chuỗi hình học này bị chặn bởi 2n, nên **n thao tác tốn O(n) tổng cộng → mỗi thao tác tốn O(1) amortized**. Từng lần append riêng lẻ có thể đắt, nhưng những lần đắt hiếm đến mức "trả góp" đều ra thì mỗi lần chỉ tốn hằng số.

Kỹ thuật chứng minh chuẩn là **phương pháp kế toán (accounting method)**: mỗi lần append "trả trước" 3 đơn vị — 1 cho việc ghi phần tử, 2 gửi tiết kiệm để trả cho lần copy tương lai. Có thể kiểm tra rằng số dư tiết kiệm không bao giờ âm, nên tổng chi phí thật ≤ tổng đã trả = 3n. Đây chính là một **invariant** (chương 03) áp dụng vào phân tích chi phí.

Câu hỏi kiểm tra hiểu bản chất: *tại sao nhân đôi mà không phải cộng thêm 100 slot mỗi lần?* Nếu tăng tuyến tính, tổng chi phí copy là 100 + 200 + 300 + ... = O(n²/100) — vẫn là bình phương. **Chỉ tăng theo tỷ lệ (geometric growth) mới cho amortized O(1).** Hằng số không cứu được hình dạng của hàm.

## 5. Thuật toán — phân tích như thế nào trong thực tế

Phân tích độ phức tạp không phải là tra bảng, mà là ba kỹ thuật cơ bản:

**Kỹ thuật 1 — Đếm vòng lặp theo cấu trúc:**

```go
for i := 0; i < n; i++ {        // n lần
    for j := i + 1; j < n; j++ { // n-1, n-2, ..., 1 lần
        // so sánh cặp (i, j)
    }
}
```

Tổng số lần chạy vòng trong: (n-1) + (n-2) + ... + 1 = n(n-1)/2 = Θ(n²). Lưu ý: vòng lặp lồng nhau *không tự động* là n² — phải nhìn vào biên chạy thật. Ví dụ vòng trong chạy `j *= 2` thì tổng là Θ(n log n).

**Kỹ thuật 2 — Recurrence cho đệ quy** (chi tiết ở chương 06): merge sort chia đôi và merge tuyến tính cho T(n) = 2T(n/2) + Θ(n), Master Theorem cho ngay Θ(n log n).

**Kỹ thuật 3 — Nhìn xuyên qua abstraction:** dòng code ngắn nhất có thể đắt nhất.

```go
// Trông như O(n), thực ra là O(n²)
s := ""
for _, w := range words {
    s += w // mỗi lần += copy toàn bộ chuỗi cũ: 1+2+...+n ký tự
}

// Đúng cách: strings.Builder — amortized O(1) mỗi lần ghi
var b strings.Builder
for _, w := range words {
    b.WriteString(w)
}
```

String trong Go là immutable, nên `s += w` cấp phát chuỗi mới và copy. Đây là lỗi production thật sự phổ biến — và là ví dụ hoàn hảo cho việc độ phức tạp nằm ở **ngữ nghĩa của phép toán**, không phải ở số dòng code.

### Bảng tra nhanh và ngưỡng thực dụng

Với CPU hiện đại xử lý ~10⁸–10⁹ phép toán đơn giản mỗi giây, quy tắc ngón tay cái khi ước lượng "thuật toán này có chạy nổi không":

| Độ phức tạp | n tối đa xử lý trong ~1 giây | Ví dụ |
|---|---|---|
| O(log n) | gần như vô hạn | binary search |
| O(n) | ~10⁸ | scan, hash lookup toàn bộ |
| O(n log n) | ~10⁷ | sort |
| O(n²) | ~10⁴ | so sánh mọi cặp |
| O(2ⁿ) | ~25 | duyệt mọi tập con |
| O(n!) | ~11 | duyệt mọi hoán vị |

Bảng này đáng nhớ hơn mọi công thức: nó biến Big-O từ lý thuyết thành công cụ ước lượng feasibility trong 5 giây — kể cả trong phỏng vấn ("n = 10⁵ → nghĩ ngay n log n hoặc tốt hơn, quên O(n²) đi").

## 6. Trade-off

**Big-O vs constant factor.** Big-O cố tình vứt hằng số, nhưng production không vứt. Một ví dụ có thật về hai cách tìm kiếm trên dữ liệu nhỏ:

```go
// Với n < ~64, linear scan trên slice thường NHANH HƠN map lookup:
// - slice nằm liên tục trong bộ nhớ → cache line prefetch hoạt động tối đa
// - map lookup: hash + truy cập bucket rải rác → cache miss
```

Đây là lý do `sort.Sort` của Go (và pdqsort từ Go 1.19) chuyển sang **insertion sort O(n²)** khi phân đoạn nhỏ hơn ~12 phần tử: ở kích thước đó, hằng số nhỏ và cache locality của insertion sort thắng mọi thuật toán O(n log n). Thuật toán "tệ về lý thuyết" là lựa chọn đúng khi n nhỏ và ta *biết* n nhỏ.

**Time vs Space.** Hash map biến tìm kiếm O(n) thành O(1) bằng cách tốn thêm O(n) bộ nhớ. Precompute, cache, index — cùng một trade-off. Chiều ngược lại cũng tồn tại: HyperLogLog (chương 23) đếm hàng tỷ phần tử với 12KB bằng cách hy sinh độ chính xác.

**Worst case vs Average case.** Chọn tối ưu theo cái nào là quyết định kiến trúc:

- Hệ thống chịu adversarial input (API công khai) phải nghĩ theo worst case: hash flooding attack từng khai thác chính khoảng cách giữa average O(1) và worst O(n) của hash table — đây là lý do Go dùng hash seed ngẫu nhiên cho mỗi map.
- Batch job nội bộ có thể tin average case và thắng lớn về hiệu năng trung bình.

**Đơn giản vs Tối ưu.** Thuật toán O(n log log n) tồn tại cho nhiều bài toán, nhưng production dùng bản O(n log n) đơn giản — vì chi phí bảo trì, rủi ro bug và khả năng đọc hiểu của team cũng là chi phí. Độ phức tạp của *code* cũng là một loại complexity.

## 7. Production Applications

**PostgreSQL Query Planner** — nơi complexity analysis chạy tự động hàng triệu lần mỗi giây. Với mỗi query, planner ước lượng chi phí của từng chiến lược: sequential scan là O(n) với hằng số nhỏ (đọc tuần tự, prefetch tốt); index scan là O(log n + k) với hằng số lớn hơn mỗi lần truy cập (random I/O). Đó là lý do PostgreSQL **cố tình chọn seq scan** khi bảng nhỏ hoặc query lấy phần lớn số dòng — một minh họa sống động rằng Big-O chỉ là một biến trong hàm chi phí, không phải toàn bộ. `EXPLAIN ANALYZE` chính là "máy phân tích độ phức tạp" bạn dùng hằng ngày.

**Redis** chọn cấu trúc dữ liệu theo đúng bảng trade-off trên: sorted set nhỏ lưu bằng listpack (mảng phẳng, O(n) nhưng hằng số tí hon, cache-friendly), tự động chuyển sang skip list O(log n) khi vượt ngưỡng `zset-max-listpack-entries`. Cùng một API, hai thuật toán, chuyển đổi dựa trên n — đây là complexity analysis được mã hóa thành config.

**Go runtime**: map là hash table với average O(1) và hash seed ngẫu nhiên chống adversarial input; slice append là amortized O(1) như đã phân tích; garbage collector được thiết kế quanh cam kết *độ trễ* — tri-color concurrent mark-sweep giữ pause ngắn (đơn vị trăm microsecond) thay vì stop-the-world tỷ lệ với heap — một dạng "amortize chi phí GC theo thời gian thực thi".

**Kafka** đạt throughput khổng lồ không phải nhờ thuật toán lạ mà nhờ tôn trọng hằng số: mọi thao tác ghi/đọc log đều là **sequential I/O** — cùng là "đọc n byte" O(n), nhưng đọc tuần tự trên SSD/HDD nhanh hơn random hàng chục đến hàng trăm lần. Consumer offset là con trỏ vào log — seek O(1) thay vì tìm kiếm.

**Elasticsearch**: inverted index biến tìm kiếm text từ O(tổng số document × độ dài) thành O(số document chứa term) — thay đổi cả *biến n* mà chi phí phụ thuộc vào. Bài học: cách tối ưu mạnh nhất thường không phải giảm bậc của hàm, mà là **đổi biến** — làm cho chi phí phụ thuộc vào đại lượng nhỏ hơn nhiều.

## 8. Interview

Complexity analysis xuất hiện trong **100%** buổi phỏng vấn thuật toán — không phải như một câu hỏi riêng mà là câu "độ phức tạp bao nhiêu?" sau mỗi lời giải.

**Các dạng phổ biến:**

- Phân tích code cho sẵn (đặc biệt vòng lặp có biên chạy lạ: `j *= 2`, `j += i`)
- Two Sum (LeetCode 1): từ O(n²) brute force → O(n) với hash map — bài kiểm tra chuẩn về trade-off time/space
- Phân tích thuật toán đệ quy (thường cần Master Theorem hoặc vẽ cây đệ quy)
- "Có thể làm tốt hơn không?" — kiểm tra bạn có biết lower bound không (ví dụ: tìm max của mảng không sort được nhanh hơn Ω(n) — mỗi phần tử phải được nhìn ít nhất một lần)

**Lỗi tư duy thường gặp:**

- Nói "hai vòng lặp lồng nhau nên là O(n²)" mà không nhìn biên chạy. Vòng trong chạy trên biến khác (m) thì là O(n·m); biên logarit thì là O(n log n).
- Quên chi phí ẩn: `strings.Contains`, slice copy, sort bên trong vòng lặp.
- Nhầm O với Θ: "thuật toán này là O(n log n)" đúng cả với thuật toán O(n) — dùng dấu nào thể hiện bạn hiểu mình đang nói chặn trên hay chặn chặt.
- Với amortized: khẳng định "append có thể tốn O(n) nên vòng lặp append n lần là O(n²)" — sai, là O(n) tổng, như đã chứng minh ở mục 4.

**Cách phân tích thay vì học thuộc:** khi gặp bài mới, hỏi tuần tự: (1) input size là biến nào — có thể có nhiều hơn một; (2) với ràng buộc n đề cho, bảng ngưỡng ở mục 5 gợi ý độ phức tạp mục tiêu nào; (3) chi phí mỗi bước lặp/đệ quy là gì, có chi phí ẩn không; (4) nghiệm của recurrence là gì. Ràng buộc n trong đề bài chính là **gợi ý lời giải bị mã hóa**: n ≤ 20 gần như hét lên "bitmask/duyệt tập con được phép".

**Từ phỏng vấn sang production:** Two Sum với hash map chính là bài toán join trong database thu nhỏ — hash join của PostgreSQL là "Two Sum trên hàng triệu dòng". Kỹ năng nhận diện "bài này về bản chất là bài kia" đáng giá hơn mọi lời giải thuộc lòng.

## 9. Anti-pattern

**Sùng bái Big-O, bỏ quên hằng số và cache.** Thay linked list "O(1) insert" cho slice "O(n) insert" rồi thấy hệ thống *chậm đi*: mỗi node linked list là một lần cấp phát heap + cache miss khi duyệt, trong khi slice hưởng trọn prefetcher. Trên phần cứng hiện đại, **cache locality thường quyết định nhiều hơn một bậc log**.

**Tối ưu độ phức tạp của phần không phải bottleneck.** Dành hai ngày đưa một hàm từ O(n log n) về O(n) trong khi 95% thời gian request nằm ở network I/O. Complexity analysis cho biết hàm nào *có thể* chậm; profiler (`pprof`) cho biết hàm nào *đang* chậm. Cần cả hai, theo đúng thứ tự: đo trước, tối ưu sau.

**Phân tích trên biến sai.** "API của tôi là O(1) vì chỉ query một dòng" — nhưng query đó là `LIKE '%keyword%'` khiến database scan O(n) trên bảng. Độ phức tạp của hệ thống là của **toàn bộ đường đi**, tính trên kích thước dữ liệu thật, không phải của riêng đoạn code bạn viết.

**Giả định n nhỏ mãi mãi.** Câu chuyện mở đầu chương này. Mọi bảng đều lớn dần; code O(n²) là quả bom hẹn giờ có ngòi nổ bằng tốc độ tăng trưởng của công ty — nó phát nổ đúng lúc công ty thành công.

**Dùng average case cho hệ thống đối mặt kẻ xấu.** Hash table "O(1)" trở thành O(n) dưới hash flooding. Regex backtracking "thường nhanh" trở thành 2ⁿ với input độc (ReDoS) — lý do Go chọn engine RE2 với đảm bảo O(n) worst case, chấp nhận bỏ backreference. Một quyết định thiết kế ngôn ngữ dựa thẳng trên worst-case analysis.

## 10. Best Practices

**Nên:**

- Ước lượng độ phức tạp **trước khi viết code**, dựa trên kích thước dữ liệu dự kiến sau 2–3 năm chứ không phải hôm nay. Bảng ngưỡng ở mục 5 đủ cho 90% quyết định.
- Nghĩ worst case cho mọi bề mặt tiếp xúc với người dùng/kẻ tấn công; cho phép average case với dữ liệu nội bộ kiểm soát được.
- Benchmark để đo hằng số khi hai lựa chọn cùng bậc — `go test -bench` với nhiều cỡ n (10³, 10⁴, 10⁵...) và nhìn *tỷ lệ tăng* giữa các cỡ: thời gian tăng ~10 lần khi n tăng 10 lần → tuyến tính; ~100 lần → bình phương. Đây là cách "đo" Big-O bằng thực nghiệm.
- Khi tối ưu, ưu tiên theo thứ tự: đổi biến (index/inverted index) → giảm bậc (thuật toán tốt hơn) → giảm hằng số (cache locality, giảm allocation) — mỗi bậc sau chỉ làm khi bậc trước không còn khả thi.

**Không nên:**

- Không tối ưu khi chưa profile — trực giác về bottleneck sai nhiều hơn đúng, kể cả với kỹ sư giàu kinh nghiệm.
- Không thay cấu trúc dữ liệu đơn giản bằng cấu trúc "tốt hơn về lý thuyết" khi n < vài nghìn — đo trước.
- Không cam kết SLA dựa trên average case.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Tại sao amortized O(1) của slice append đòi hỏi tăng capacity theo tỷ lệ, không phải cộng hằng số?
2. Vì sao PostgreSQL đôi khi cố tình chọn seq scan O(n) thay vì index scan O(log n)?
3. Benchmark cho thấy thuật toán A nhanh hơn B ở n = 10⁴. Kết luận gì được và không được phép rút ra?

---

*Chương tiếp theo: [11 — Probability](/series/math-for-engineers/level-3-algorithm-mathematics/11-probability/), nơi "average case" của chương này được đặt trên nền móng chính xác: kỳ vọng, phân phối, và lý do các hệ thống lớn dám đánh cược vào xác suất.*
