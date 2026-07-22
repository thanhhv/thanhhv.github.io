+++
title = "Chương 16 — Sorting, Searching & Heap: Trật tự và cái giá của nó"
date = "2026-07-20T09:40:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 3 – Algorithm Mathematics
> Yêu cầu trước: Chương 05 (Counting & Combinatorics), Chương 10 (Complexity Analysis), Chương 11 (Probability), Chương 13 (Divide and Conquer)

---

## 1. Problem Statement

Một team backend nhận yêu cầu tưởng như tầm thường: trang admin cần hiển thị "100 đơn hàng giá trị cao nhất trong ngày" từ một bảng 50 triệu dòng. Phiên bản đầu tiên: `ORDER BY amount DESC LIMIT 100`. Query mất 40 giây và đẩy database vào swap — PostgreSQL đang sort toàn bộ 50 triệu dòng chỉ để vứt đi 49.999.900 dòng đầu ra.

Phiên bản thứ hai, một kỹ sư thêm index trên cột `amount`. Query xuống 3ms. Phiên bản thứ ba, một kỹ sư khác chỉ ra rằng ngay cả *không có index*, PostgreSQL từ 8.3 đã đủ thông minh để không sort toàn bộ: nó giữ một **heap 100 phần tử** và quét một lượt — bộ nhớ O(k), thời gian O(n log k), không bao giờ chạm swap.

Ba phiên bản, ba cách trả lời cùng một câu hỏi, chênh nhau bốn bậc độ lớn. Điểm chung của cả ba: chúng đều xoay quanh một tài nguyên vô hình — **trật tự** (order). Có trật tự sẵn (index) thì tìm kiếm gần như miễn phí. Chưa có thì phải trả tiền tạo ra nó (sort), và câu hỏi trở thành: trả bao nhiêu, trả cho toàn bộ hay chỉ phần cần (heap)?

Chương này trả lời ba câu hỏi nền tảng:

1. **Tạo ra trật tự tốn tối thiểu bao nhiêu?** — và đây là một trong số ít định lý "bất khả thi" mà mọi engineer phải biết: không tồn tại comparison sort nhanh hơn Ω(n log n), *bất kể bạn thông minh đến đâu*.
2. **Khai thác trật tự như thế nào?** — binary search, và dạng tổng quát hóa mạnh hơn nhiều so với "tìm phần tử trong mảng".
3. **Khi chỉ cần trật tự một phần thì sao?** — heap, cấu trúc "trật tự vừa đủ" đứng sau priority queue, timer, top-k của gần như mọi hệ thống lớn.

## 2. Trực giác

### Sort một lần, search mãi mãi

Vì sao sort là bài toán trung tâm của khoa học máy tính — đến mức Knuth dành nguyên một tập TAOCP cho nó? Không phải vì "sắp xếp danh sách" tự nó quan trọng, mà vì sort là dạng **preprocessing** thuần khiết nhất: đầu tư một lần Θ(n log n), đổi lấy vô số lần khai thác rẻ về sau.

| Bài toán | Trên dữ liệu thô | Sau khi sort |
|---|---|---|
| Tìm một phần tử | O(n) | O(log n) — binary search |
| Tìm min/max | O(n) | O(1) |
| Tìm phần tử trùng | O(n²) hoặc O(n) + hash | O(n) — trùng thì đứng cạnh nhau |
| Range query [a, b] | O(n) | O(log n + k) |
| Median, percentile | O(n) (khó viết đúng) | O(1) |
| Merge với tập khác đã sort | O(n·m) | O(n + m) |

Trực giác kinh tế: nếu dữ liệu được đọc nhiều lần hơn được ghi (read-heavy — đặc trưng của đa số hệ thống), trả trước chi phí sort gần như luôn lãi. Database index, log đã sort theo offset của Kafka, SSTable của LSM-tree — tất cả là các biến thể của cùng một quyết định đầu tư này. Ngược lại, với dữ liệu ghi một lần đọc một lần (pipeline ETL streaming), sort toàn bộ thường là lãng phí — đây là câu "khi nào KHÔNG nên dùng" đầu tiên của chương.

### Trật tự là thông tin

Trực giác thứ hai, sâu hơn: **một mảng đã sort chứa nhiều thông tin hơn mảng chưa sort**. Nghe lạ — cùng n con số cơ mà? Nhưng mảng đã sort cho bạn biết thêm một điều về *mọi cặp phần tử*: phần tử nào đứng trước phần tử nào. Sort chính là quá trình *sản xuất thông tin đó*, và thông tin không miễn phí. Mỗi phép so sánh `a < b` chỉ thu về đúng **một bit** thông tin (kết quả đúng/sai). Lượng thông tin cần sản xuất chia cho tốc độ sản xuất tối đa cho ra chi phí tối thiểu — đó chính là ý tưởng của chứng minh lower bound ở mục 4, một trong những lập luận đẹp nhất mà một engineer có thể sở hữu.

### Heap: trật tự vừa đủ

Giữa "hỗn loạn hoàn toàn" (mảng thô) và "trật tự hoàn toàn" (mảng đã sort) có một điểm giữa cực kỳ hữu dụng: cấu trúc chỉ đảm bảo *phần tử nhỏ nhất luôn đứng đầu*, còn phần còn lại lộn xộn tùy ý. Đó là heap. Vì hứa ít hơn nên nó trả ít hơn: thêm/xóa O(log n), xây từ đầu O(n) — rẻ hơn sort. Bài toán ORDER BY LIMIT ở mở đầu chính là chỗ heap tỏa sáng: bạn không cần trật tự của 50 triệu dòng, chỉ cần trật tự của 100 dòng.

## 3. First Principles

### Comparison sort là gì, chính xác?

Trước khi chứng minh "không thể nhanh hơn", phải định nghĩa rõ "nhanh hơn cái gì, trong luật chơi nào". Một **comparison sort** là thuật toán chỉ được thu thập thông tin về input qua duy nhất một loại câu hỏi: "aᵢ < aⱼ không?". Nó không được nhìn vào giá trị, không được lấy digit, không được hash. Quicksort, mergesort, heapsort, insertion sort, Timsort — tất cả đều sống trong luật chơi này. Định nghĩa chặt luật chơi là bước quan trọng nhất: lower bound sắp chứng minh chỉ đúng *bên trong* luật này, và mục 5 sẽ cho thấy cách bước ra ngoài để phá nó.

### Sort là bài toán nhận diện hoán vị

Nhìn từ First Principles, sort không phải là "di chuyển phần tử". Với n phần tử phân biệt, input là một trong **n! hoán vị** có thể (chương 05). Nhiệm vụ thật sự của thuật toán là **xác định input đang là hoán vị nào** — vì biết hoán vị rồi thì việc đưa về đúng chỗ chỉ là chuyện cơ học. Mỗi hoán vị khác nhau đòi hỏi dãy hành động sửa chữa khác nhau, nên thuật toán *bắt buộc* phải phân biệt được cả n! trường hợp.

Đến đây, hai mảnh ghép khớp vào nhau:

- Cần phân biệt **n! khả năng**.
- Mỗi phép so sánh cho tối đa **1 bit** — chia không gian khả năng làm hai phần.

Sau k phép so sánh, thuật toán phân biệt được tối đa 2ᵏ trường hợp. Muốn 2ᵏ ≥ n! thì k ≥ log₂(n!). Toàn bộ chứng minh ở mục 4 chỉ là làm chặt chẽ lập luận ba dòng này.

### Nếu không có khái niệm lower bound thì sao?

Không có lower bound, mọi cuộc tối ưu đều là cuộc đua không có vạch đích: bạn không bao giờ biết mình nên dừng ở đâu, và sẽ có người đốt hàng tháng đi tìm thuật toán so sánh O(n) — thứ toán học chứng minh là không tồn tại, tương tự việc đi tìm động cơ vĩnh cửu. Lower bound biến "chưa ai tìm ra" thành "không thể tồn tại" — hai mệnh đề khác nhau về bản chất, và chỉ toán học phân biệt được (benchmark thì không, như chương 10 đã nói). Nó cũng chỉ đường tối ưu đúng hướng: muốn nhanh hơn n log n, đừng cải tiến phép so sánh — hãy **đổi luật chơi**.

## 4. Mathematical Model

### Decision tree: mọi comparison sort là một cái cây

Cố định n. Bất kỳ comparison sort *tất định* nào cũng có thể vẽ thành một **cây quyết định nhị phân**: mỗi node trong là một phép so sánh "aᵢ < aⱼ?", hai nhánh là hai kết quả, và mỗi lá là kết luận cuối cùng — một hoán vị cụ thể mà thuật toán tuyên bố là thứ tự đúng. Chạy thuật toán trên một input = đi một đường từ gốc xuống một lá.

```
                 [a₁ < a₂?]                 ← so sánh đầu tiên
                 ╱        ╲
              yes          no
              ╱              ╲
        [a₂ < a₃?]        [a₁ < a₃?]
         ╱      ╲          ╱      ╲
     (1,2,3)  [a₁<a₃?] (2,1,3)  [a₂<a₃?]
              ╱     ╲            ╱     ╲
         (1,3,2) (3,1,2)    (2,3,1) (3,2,1)

   n = 3: cây phải có đủ 3! = 6 lá
   → chiều cao tối thiểu ⌈log₂6⌉ = 3 phép so sánh worst case
```

Ba quan sát, mỗi quan sát một dòng, ghép lại thành định lý:

**Quan sát 1 — cây phải có ít nhất n! lá.** Mỗi hoán vị input khác nhau phải đi đến kết luận khác nhau. Nếu hai hoán vị khác nhau cùng rơi vào một lá, thuật toán trả cùng một đáp án cho hai input cần đáp án khác nhau — sai với ít nhất một trong hai. (Đây là chứng minh bằng phản chứng, chương 03.)

**Quan sát 2 — cây nhị phân cao h có tối đa 2ʰ lá.** Quy nạp theo h: mỗi tầng đi xuống, số node tối đa nhân đôi.

**Quan sát 3 — worst case của thuật toán = chiều cao cây.** Đường dài nhất từ gốc xuống lá chính là input xui xẻo nhất.

Ghép lại: 2ʰ ≥ (số lá) ≥ n!, suy ra:

> **h ≥ log₂(n!)**

### Vì sao log₂(n!) = Θ(n log n) — Stirling ở mức trực giác

Công thức Stirling nói n! ≈ (n/e)ⁿ·√(2πn), nhưng không cần nhớ nó. Chỉ cần một cặp chặn thô mà bạn tự dựng lại được trong 30 giây:

- **Chặn trên:** n! = n·(n−1)···1 ≤ nⁿ → log₂(n!) ≤ n log₂ n.
- **Chặn dưới:** trong n thừa số, một nửa đầu (từ n xuống n/2) đều ≥ n/2, nên n! ≥ (n/2)^(n/2) → log₂(n!) ≥ (n/2)·log₂(n/2).

Cả hai vế đều là hằng số nhân với n log n, vậy **log₂(n!) = Θ(n log n)**. Định lý hoàn tất:

> **Mọi comparison sort cần Ω(n log n) phép so sánh trong worst case.**

(Với thuật toán ngẫu nhiên như quicksort, lập luận trung bình hóa trên các cây quyết định cho cùng kết luận với *expected* số so sánh — lower bound không có lỗ hổng "dùng random để lách".)

Con số cụ thể đáng cảm nhận: n = 10⁶ thì log₂(n!) ≈ 18,5 triệu phép so sánh tối thiểu. Mergesort thực hiện ~20 triệu. Nghĩa là mergesort chỉ cách trần lý thuyết vài phần trăm — nhân loại về cơ bản đã giải xong bài toán này *trong luật chơi so sánh*, từ những năm 1960. Mọi cải tiến sau đó (Timsort, pdqsort) là cải tiến hằng số, cache, và các trường hợp đặc biệt — không phải cải tiến bậc.

### Phá lower bound: ngừng so sánh

Lower bound có một giả định duy nhất — và đó là cửa thoát. Nếu được *nhìn vào giá trị*, mỗi thao tác thu về nhiều hơn 1 bit:

**Counting sort.** Khi giá trị là số nguyên trong [0, k): đếm số lần xuất hiện của từng giá trị vào mảng phụ, rồi ghi lại theo thứ tự. Θ(n + k) — tuyến tính khi k = O(n). Một lần đọc `count[v]++` "phân loại" phần tử vào một trong k nhóm — thu log₂k bit thông tin, không phải 1 bit.

**Radix sort.** Sort số d chữ số bằng d lượt counting sort, từ chữ số thấp lên cao (bắt buộc dùng counting sort *stable* — thứ tự lượt trước phải được bảo toàn để làm tie-break cho lượt sau). Θ(d·(n + k)). Với 10⁷ số uint32, d = 4 lượt theo từng byte (k = 256) đánh bại mọi comparison sort.

**Bucket sort.** Khi input phân bố *đều* trên một khoảng: chia n bucket, mỗi bucket kỳ vọng O(1) phần tử, sort từng bucket. Expected Θ(n) — nhưng chữ "expected" gắn chặt vào giả định phân bố; input dồn cục vào một bucket thì thoái hóa về thuật toán con.

Vậy vì sao chúng không thay thế quicksort làm sort mặc định? Vì mỗi cái mua tốc độ bằng một **giả định về dữ liệu**:

| Thuật toán | Điều kiện áp dụng | Sụp đổ khi |
|---|---|---|
| Counting sort | số nguyên, miền k nhỏ | k = 2⁶⁴, hoặc key là struct/string |
| Radix sort | key phân rã được thành digit | comparator tùy ý (`less func(a,b)`) |
| Bucket sort | biết phân bố, gần uniform | phân bố lệch, adversarial input |

Comparison sort chậm hơn nhưng **phổ dụng**: chỉ cần một hàm `less`, sort được mọi thứ — struct theo ba trường, string theo collation tiếng Việt, bất kỳ total order nào. Đây là trade-off giữa tốc độ và tính tổng quát của contract, giống hệt việc chọn interface hẹp hay rộng khi thiết kế API.

### Các tính chất production của một thuật toán sort

Big-O không phải chiều duy nhất. Ba tính chất sau quyết định lựa chọn trong thực tế:

**Stable** — hai phần tử "bằng nhau" giữ nguyên thứ tự tương đối ban đầu. Nghe như chi tiết vặt cho đến khi bạn sort nhiều tiêu chí: sort danh sách đơn hàng theo `ngày`, rồi sort *stable* theo `khách hàng` → kết quả nhóm theo khách hàng, trong mỗi nhóm đúng thứ tự ngày. Với sort không stable, lượt sort sau *phá hủy* thành quả lượt trước. Go tách hẳn hai API: `sort.Slice` (pdqsort, không stable, nhanh hơn) và `sort.SliceStable` (chèn merge, stable, chậm hơn ~2 lần và tốn thêm bộ nhớ) — buộc bạn tuyên bố mình cần gì. Chọn nhầm `sort.Slice` khi cần stable là loại bug chỉ lộ ra khi có dữ liệu trùng key, tức là lộ ra ở production chứ không phải ở test.

**In-place** — chỉ dùng O(1) hoặc O(log n) bộ nhớ phụ. Quicksort và heapsort in-place; mergesort cần O(n) phụ. Trên dataset chiếm 60% RAM, đây là khác biệt giữa chạy được và OOM.

**Adaptive** — chạy nhanh hơn khi input *gần* sorted. Timsort (mặc định của Python, Java) phát hiện các "run" tăng dần sẵn có và chỉ merge chúng: input đã sort → O(n). Điều này quan trọng vì dữ liệu thực hiếm khi ngẫu nhiên đều — log gần như sorted theo thời gian, bảng được sort lại sau vài dòng chèn thêm.

## 5. Thuật toán

### Quicksort: đánh bạc có tính toán

Quicksort chọn một pivot, partition mảng thành phần nhỏ hơn và lớn hơn pivot, đệ quy hai bên. Worst case O(n²): nếu pivot luôn là min/max, mỗi lượt chỉ bóc được một phần tử — và với pivot tất định (ví dụ "luôn lấy phần tử đầu"), kẻ xấu *dựng được* input độc, hoặc tệ hơn: input tự nhiên phổ biến nhất — mảng đã sort — chính là input độc.

Giải pháp: **chọn pivot ngẫu nhiên**, biến worst case từ "input nào đó" thành "vận rủi nào đó" — không input nào là input độc nữa. Phân tích average case là một viên ngọc của linearity of expectation (chương 11). Gọi z₁ < z₂ < ... < zₙ là các phần tử theo thứ tự đúng. Đặt biến chỉ thị Xᵢⱼ = 1 nếu zᵢ và zⱼ từng được so sánh với nhau. Nhận xét then chốt: zᵢ và zⱼ được so sánh **khi và chỉ khi** một trong hai được chọn làm pivot *trước* mọi phần tử nằm giữa chúng — vì nếu pivot rơi vào giữa, hai phần tử bị tách về hai phía và vĩnh viễn không gặp nhau. Trong khối {zᵢ, ..., zⱼ} gồm j−i+1 phần tử, mỗi phần tử đều bình đẳng về xác suất được làm pivot đầu tiên của khối:

> P(zᵢ so sánh với zⱼ) = 2/(j − i + 1)

Tổng kỳ vọng số phép so sánh:

> E[X] = Σᵢ<ⱼ 2/(j−i+1) ≤ Σᵢ Σ_d 2/d ≈ 2n·ln n ≈ 1,39·n log₂ n

Không cần giải recurrence phức tạp nào — kỳ vọng của tổng bằng tổng kỳ vọng, kể cả khi các biến phụ thuộc chằng chịt lẫn nhau. Kết quả: expected Θ(n log n) với hằng số nhỏ và cache locality tuyệt vời (partition quét tuần tự), lý do quicksort thắng mergesort trong thực chiến dù "yếu hơn" trên giấy về worst case.

**pdqsort — cách Go 1.19+ phòng thủ.** `sort.Slice` dùng pattern-defeating quicksort: quicksort làm đường chính; insertion sort cho đoạn < 12 phần tử; phát hiện pattern (đã sort, sort ngược) để chạy O(n); và quan trọng nhất — đếm số lần partition mất cân bằng, vượt ngưỡng thì **chuyển sang heapsort**, chốt chặn worst case O(n log n) tuyệt đối. Đây là kỹ thuật introsort: dùng thuật toán nhanh-trung-bình, nhưng cài "cầu dao" bằng thuật toán chắc-worst-case. Một bài học thiết kế hệ thống thu nhỏ: optimistic path + circuit breaker.

### Binary search: viết đúng bằng invariant

Binary search nổi tiếng là thuật toán 5 dòng mà sách "Programming Pearls" ghi nhận ~90% lập trình viên chuyên nghiệp viết sai lần đầu. Ba lỗi kinh điển:

1. **Overflow**: `mid := (lo + hi) / 2` tràn số khi lo + hi vượt giới hạn int (bug nằm trong JDK gần 10 năm). Viết `mid := lo + (hi-lo)/2`.
2. **Vòng lặp vô hạn**: khi khoảng còn 2 phần tử mà cập nhật `lo = mid` (không +1), mid tính ra lại bằng lo — kẹt vĩnh viễn.
3. **Lệch biên một đơn vị**: `<` hay `<=`, `hi = mid` hay `mid - 1` — mỗi tổ hợp đúng với một biến thể và sai với biến thể khác.

Thuốc chữa cả ba không phải là học thuộc code mẫu, mà là **một invariant duy nhất** (chương 03). Dạng tổng quát hóa đáng học: cho predicate đơn điệu p — sai, sai, ..., sai, **đúng**, đúng, ..., đúng — tìm vị trí `đúng` đầu tiên:

```go
// Tìm chỉ số nhỏ nhất trong [0, n) mà p(i) == true; trả về n nếu không có.
// Đây chính là contract của sort.Search trong thư viện chuẩn Go.
func search(n int, p func(int) bool) int {
    lo, hi := 0, n
    // Invariant: p sai với mọi i < lo; p đúng với mọi i >= hi.
    // Đáp án luôn nằm trong [lo, hi].
    for lo < hi {
        mid := lo + (hi-lo)/2 // mid < hi nên không lặp vô hạn
        if p(mid) {
            hi = mid // p(mid) đúng → đáp án ở [lo, mid]
        } else {
            lo = mid + 1 // p(mid) sai → đáp án ở [mid+1, hi]
        }
    }
    return lo // lo == hi: điểm gãy sai→đúng
}
```

Mỗi lần cập nhật đều *bảo toàn invariant*, khoảng [lo, hi) co lại ít nhất 1 mỗi vòng, và khi lo == hi thì invariant ép đáp án bằng lo. Đúng theo cấu trúc, không cần dò từng biên.

Sức mạnh thật sự nằm ở chỗ **p là hàm bất kỳ, miễn đơn điệu** — không cần mảng nào cả. "Tìm x trong mảng sorted" chỉ là trường hợp p(i) = (a[i] ≥ x). Các trường hợp khác của cùng khuôn: tìm commit đầu tiên làm hỏng build (`git bisect` — p(commit) = "build fail"), tìm capacity nhỏ nhất mà hệ thống chịu tải (p(c) = "load test pass"), và cả họ bài "binary search the answer" ở mục 8. Không gian nào có trật tự đơn điệu, không gian đó search được trong log.

### Heap: cây hoàn hảo sống trong mảng

Binary heap là **complete binary tree** (đầy mọi tầng trừ tầng cuối, tầng cuối dồn trái) thỏa heap property: parent ≤ con (min-heap). Vì cây complete không có "lỗ hổng", nó nhúng được vào mảng phẳng theo thứ tự từng tầng, và quan hệ cha–con trở thành **số học chỉ số**:

```
      chỉ số:   0   1   2   3   4   5   6
      mảng:  [ 2 | 5 | 3 | 9 | 7 | 8 | 4 ]

                    2(0)
                  ╱      ╲
               5(1)      3(2)        parent(i) = (i-1)/2
              ╱    ╲    ╱    ╲       left(i)   = 2i+1
            9(3)  7(4) 8(5) 4(6)     right(i)  = 2i+2
```

Đây là lý do heap không cần con trỏ — và vì thế thắng lớn trên phần cứng thật: không allocation cho node, không pointer chasing, các phần tử nằm sát nhau trong cache. Cùng là "cây" nhưng chi phí hằng số khác BST một trời một vực.

Hai thao tác nền tảng, cả hai O(log n) vì cây complete cao ⌊log₂n⌋:

```go
// sift-up: phần tử mới chèn ở đáy nổi lên vị trí đúng
func siftUp(h []int, i int) {
    for i > 0 {
        p := (i - 1) / 2
        if h[p] <= h[i] { break } // heap property đã thỏa
        h[p], h[i] = h[i], h[p]
        i = p
    }
}

// sift-down: phần tử ở gốc chìm xuống vị trí đúng
func siftDown(h []int, i, n int) {
    for {
        min, l, r := i, 2*i+1, 2*i+2
        if l < n && h[l] < h[min] { min = l }
        if r < n && h[r] < h[min] { min = r }
        if min == i { return }
        h[i], h[min] = h[min], h[i]
        i = min
    }
}
```

**Build-heap O(n) — kết quả phản trực giác.** Xây heap từ mảng thô bằng cách sift-down từ node giữa về gốc. Trực giác nói n node × O(log n) = O(n log n). Nhưng đếm kỹ hơn: sift-down tốn tỷ lệ với *chiều cao còn lại bên dưới*, và tuyệt đại đa số node nằm sát đáy nơi chiều cao gần 0 — một nửa số node có chiều cao 0, một phần tư có chiều cao 1... Tổng chi phí:

> Σₕ (số node cao h)·h ≤ Σₕ (n/2ʰ⁺¹)·h = n·Σₕ h/2ʰ⁺¹ = n·(1/2)·2 = **O(n)**

trong đó Σ h/2ʰ = 2 là chuỗi hội tụ quen thuộc (đạo hàm của chuỗi hình học). Cùng một khuôn với chứng minh amortized append ở chương 10: chi phí lớn hiếm, chi phí nhỏ áp đảo, tổng lại tuyến tính. Hệ quả thực dụng: cần heap từ n phần tử có sẵn thì `heap.Init` một lần (O(n)), đừng Push n lần (O(n log n)).

Từ heap suy ra **heapsort**: build-heap rồi n lần lấy gốc — O(n log n) worst case, in-place, không cần bộ nhớ phụ; nhược điểm là truy cập mảng nhảy cóc nên cache kém, thua quicksort về hằng số (vì thế chỉ làm "cầu dao" trong pdqsort). Và **priority queue** — heap với tên gọi theo interface: `container/heap` của Go yêu cầu bạn implement `sort.Interface` + `Push/Pop`, còn sift-up/sift-down do runtime lo. Ứng dụng gặp hằng ngày: **k-way merge** — trộn k dòng dữ liệu đã sort bằng heap k phần tử chứa đầu mỗi dòng, mỗi bước lấy min và nạp phần tử kế của dòng đó, O(N log k) tổng — xương sống của external sort và compaction trong LSM-tree.

## 6. Trade-off

**Sort trước vs tìm trực tiếp.** Tìm một lần trong dữ liệu chưa sort: O(n) — đừng sort. Tìm m lần: sort trước hòa vốn khi n log n < m·n, tức m > log n — con số nhỏ bất ngờ (n = 10⁶ → chỉ cần 20 lần tìm là sort đã lãi). Nhưng nếu chỉ cần *membership* chứ không cần range/order, hash map O(1) thắng cả hai — sort chỉ đáng giá khi bạn cần **trật tự**, không chỉ tồn tại.

**Sort toàn bộ vs heap top-k.** Cần k phần tử lớn nhất từ n: sort O(n log n); heap k phần tử O(n log k); quickselect O(n) average. Với k ≪ n khác biệt là nhiều bậc. Nhưng quickselect phá hủy mảng, không streaming được và không cho k phần tử *theo thứ tự*; heap streaming tự nhiên (mỗi phần tử đến, so với gốc, thay nếu tốt hơn) — vì thế hệ thống dòng chảy (log, metrics) chọn heap dù quickselect "nhanh hơn trên giấy".

**Stable vs tốc độ/bộ nhớ.** Stability không miễn phí: mergesort stable nhưng O(n) bộ nhớ phụ; các stable sort in-place đều chậm hằng số lớn. Đó là lý do Go bắt bạn chọn `sort.Slice`/`sort.SliceStable` một cách tường minh thay vì mặc định stable như Python — hai triết lý API: an toàn mặc định vs chi phí tường minh.

**Comparison vs non-comparison.** Radix sort nhanh hơn nhưng đóng đinh vào kiểu key; đổi key từ uint32 sang string có collation là phải viết lại. Comparison sort đổi hành vi chỉ bằng đổi closure `less`. Tính tổng quát cũng là một tài nguyên kỹ thuật.

**Heap vs sorted structure.** Heap cho min O(1), insert O(log n) — nhưng *chỉ* min; tìm phần tử bất kỳ là O(n), duyệt theo thứ tự phải phá heap. Cần cả hai đầu, cần range query → skip list / B-tree, trả giá hằng số và bộ nhớ cao hơn. Chọn cấu trúc là chọn *tập câu hỏi* bạn sẽ hỏi dữ liệu.

## 7. Production Applications

**PostgreSQL — cả kho thuật toán của chương này.** `ORDER BY` trên dữ liệu vượt `work_mem` chạy **external merge sort**: sort từng khối vừa bộ nhớ (quicksort), ghi ra đĩa thành các "run", rồi **k-way merge bằng heap** — đúng nghịch cảnh mà mergesort sinh ra để giải từ thời băng từ, vì merge chỉ cần đọc/ghi *tuần tự*. Với `ORDER BY x LIMIT k`, planner chuyển sang node `top-N heapsort`: heap k phần tử, quét một lượt, O(n log k) và bộ nhớ O(k) — chính là lời giải phiên bản ba ở mở đầu chương. Còn khi có index B-tree trên `x` thì khỏi sort: index *là* trật tự trả trước, mỗi lần INSERT/UPDATE đóng góp O(log n) để duy trì — "sort được trả góp theo từng lần ghi". `EXPLAIN` cho bạn thấy đúng ba chữ: `Sort`, `top-N heapsort`, `Index Scan` — ba điểm trên đường trade-off của chương này.

**Index seek vs scan.** Câu hỏi muôn thuở "sao query có index vẫn seq scan?" trả lời bằng toán của chương 10 + 16: index seek là O(log n + k) với random I/O đắt mỗi bước; seq scan là O(n) tuần tự với hằng số tí hon. k lớn (query lấy nhiều % bảng) → scan thắng. Trật tự chỉ đáng giá khi bạn khai thác được *tính chọn lọc* của nó.

**Kafka** — hệ thống được thiết kế để **không bao giờ phải sort**: log là append-only, message tự động sorted theo offset ngay lúc ghi. Consumer tìm vị trí đọc bằng binary search trên file index thưa (sparse index: mỗi ~4KB một entry offset→vị trí file), rồi scan tuần tự đoạn ngắn còn lại. Toàn bộ kiến trúc là câu trả lời cho câu hỏi "làm sao có trật tự mà không trả n log n?" — bằng cách chọn trật tự *trùng với thứ tự ghi tự nhiên*.

**Elasticsearch** trả về top-10 kết quả từ hàng triệu document khớp bằng đúng heap top-k ở mục 5 — mỗi shard giữ một priority queue kích thước k khi quét posting list, rồi coordinator k-way merge các top-k của từng shard. Không có chỗ nào trong đường đi đó sort toàn bộ kết quả.

**Go runtime** dùng heap ở nơi nhạy cảm nhất: **timer**. Mỗi P (processor) giữ các timer trong một **heap bậc 4** (4 con mỗi node thay vì 2) — cây nông hơn một nửa, mỗi node so sánh nhiều hơn nhưng 4 con nằm gọn trong cache line, ăn đứt heap nhị phân về hằng số. Mỗi `time.After`, mỗi context timeout trong hàng triệu goroutine của bạn là một phần tử trong heap đó; scheduler chỉ cần nhìn gốc heap để biết "bao giờ phải dậy". Kubernetes cũng vậy: priority queue của scheduler quyết định pod nào được xếp node trước.

## 8. Interview

Sorting/searching/heap phủ có lẽ 1/3 số câu phỏng vấn thuật toán. Các dạng trọng tâm:

**Binary search các biến thể** — Find First and Last Position (LeetCode 34), Search Insert Position (35), Search in Rotated Sorted Array (33). Tất cả quy về *một* khuôn `search(n, p)` ở mục 5 với predicate khác nhau: first position là p(i) = a[i] ≥ x, last position là (chỉ số đầu mà a[i] > x) − 1. Người học thuộc bốn template `lo<=hi / lo<hi / hi=mid / hi=mid-1` sẽ lẫn lộn dưới áp lực; người nắm invariant chỉ cần trả lời "predicate của tôi là gì, nó đơn điệu không".

**Binary search the answer** — Koko Eating Bananas (875): Koko ăn chuối tốc độ k, đống i tốn ⌈pile[i]/k⌉ giờ, tìm k nhỏ nhất để ăn hết trong h giờ. Nhận diện: hàm p(k) = "tốc độ k đủ để kịp" là **đơn điệu** (ăn nhanh hơn không bao giờ trễ hơn) → binary search trên *không gian đáp án* [1, max(pile)], mỗi lần thử tốn O(n). Cùng họ: Split Array Largest Sum (410), Capacity to Ship Packages (1011). Đây là kỹ thuật đáng giá nhất mục này vì nó đảo ngược tư duy: không xây đáp án — **đoán đáp án và kiểm tra**, miễn kiểm tra rẻ và không gian đáp án đơn điệu. Trong production nó là mọi bài toán capacity planning: "số replica tối thiểu chịu được tải X".

**Kth Largest Element (215)** — bài kiểm tra trade-off kinh điển: heap k phần tử O(n log k), streaming được, worst case chắc chắn; quickselect O(n) average nhưng O(n²) worst nếu không random pivot, và cần cả mảng trong tay. Người phỏng vấn muốn nghe bạn *so sánh*, không phải chọn một. Nói thêm "nếu dữ liệu là stream thì bắt buộc heap" là điểm cộng lớn.

**Merge Intervals (56)** — minh họa "sort làm bài toán khó thành dễ": intervals lộn xộn thì kiểm tra chồng lấn là O(n²); sort theo điểm bắt đầu xong, chỉ interval *liền kề* mới có thể merge — một lượt quét O(n). Rất nhiều bài dạng "cặp phần tử tương tác nhau" tan biến sau một lần sort đúng tiêu chí.

**Top K Frequent Elements (347)** — hash map đếm tần suất + heap k phần tử trên (phần tử, tần suất): O(n log k). Follow-up hay gặp: làm được O(n) không? — bucket sort theo tần suất, vì tần suất bị chặn bởi n: một ví dụ non-comparison sort sống trong bài phỏng vấn.

**Lỗi tư duy thường gặp:** viết binary search bằng trí nhớ rồi sửa biên theo test case — thay vì phát biểu invariant trước; trả lời "sort là O(n log n)" như tiên đề mà không biết đó là lower bound có chứng minh (câu hỏi "vì sao không nhanh hơn được?" tách ứng viên đọc hiểu khỏi ứng viên học vẹt); dùng quickselect cho bài streaming; quên rằng `heap.Init` là O(n) khi bị hỏi tối ưu build.

## 9. Anti-pattern

**Sort trong vòng lặp.** `for` mỗi request lại `sort.Slice` cùng một slice cấu hình/route table — O(n log n) nhân với QPS. Trật tự là tài sản đầu tư: tạo một lần lúc load config, tái sử dụng; sort lại chỉ khi dữ liệu đổi.

**Sort toàn bộ để lấy top-k.** `ORDER BY ... LIMIT 10` tự viết bằng cách kéo hết về service rồi sort là phiên bản một của câu chuyện mở đầu — trả n log n cho thứ đáng giá n log k, và tệ hơn: trả cả network + memory cho n dòng. Đẩy LIMIT xuống database, hoặc dùng heap.

**Tin rằng "tôi sẽ tìm được cách sort so sánh O(n)".** Tương đương chế động cơ vĩnh cửu. Khi ai đó khoe sort "tuyến tính", câu hỏi đúng là: *giả định gì về dữ liệu?* — luôn có một giả định (miền giá trị, phân bố, độ dài key), và giả định đó chính là chỗ hệ thống sẽ gãy khi dữ liệu đổi tính chất.

**Dùng sort không stable cho sort nhiều tiêu chí nối tiếp.** `sort.Slice` theo tiêu chí phụ rồi `sort.Slice` theo tiêu chí chính — lượt hai xáo trộn tự do các phần tử bằng nhau, phá thành quả lượt một. Hoặc dùng `sort.SliceStable` cho lượt sau, hoặc gộp mọi tiêu chí vào một comparator (thường tốt hơn: một lượt sort, ý định tường minh).

**Poll thay vì priority queue.** Scheduler quét toàn bộ danh sách task mỗi giây tìm task đến hạn: O(n) mỗi tick, và độ trễ bằng chu kỳ tick. Heap theo deadline: O(log n) mỗi thao tác, gốc heap cho biết *chính xác* thời điểm cần dậy. Go runtime timer làm mẫu sẵn cho bạn.

**Binary search trên dữ liệu không đảm bảo sorted.** Hàm nhận slice và binary search, nhưng không nơi nào trong contract nói slice phải sorted — code chạy đúng nhiều tháng vì tình cờ upstream sort sẵn, rồi một refactor upstream đổi thứ tự và bug xuất hiện *lặng lẽ* (kết quả sai, không panic). Trật tự là precondition — hoặc kiểm tra (`sort.SliceIsSorted` trong test/debug build), hoặc ghi vào tên và doc của hàm.

## 10. Best Practices

**Nên:**

- Trả lời câu hỏi "trật tự này được dùng mấy lần?" trước khi sort. Một lần → tìm trực tiếp; nhiều lần → sort/index; chỉ cần k phần tử biên → heap.
- Viết mọi binary search bằng khuôn predicate + invariant (`sort.Search` của Go chính là khuôn đó — dùng nó thay vì tự viết khi được).
- Tuyên bố tường minh nhu cầu stability: `sort.SliceStable` khi có multi-key hoặc thứ tự ban đầu mang nghĩa; comment lý do để người sau không "tối ưu" ngược về `sort.Slice`.
- Cần heap trong Go, dùng `container/heap` và nhớ `heap.Init` O(n) cho dữ liệu có sẵn; cần top-k streaming, giữ heap kích thước k và so sánh với gốc trước khi push.
- Khi n có miền giá trị nhỏ và là số nguyên (đếm tần suất, histogram, sort theo byte), nhớ đến counting/radix sort — nhưng viết rõ giả định vào comment.

**Không nên:**

- Không tự cài quicksort/heapsort cho production khi thư viện chuẩn có sẵn phiên bản đã phòng thủ (pdqsort chống input độc, đã fuzz nhiều năm) — tự cài chỉ để học.
- Không dùng average case của quickselect cho đường đi nhạy latency mà không có phương án chặn worst case.
- Không so sánh hai float bằng `<` trong comparator mà quên NaN — NaN phá tính total order, `sort.Slice` với comparator không nhất quán cho kết quả không xác định.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Chứng minh lower bound Ω(n log n) dựa trên những giả định nào, và counting sort "lách" qua giả định nào trong số đó?
2. Vì sao build-heap là O(n) trong khi n lần push là O(n log n) — trực giác "phần lớn node nằm gần đáy" áp dụng ra sao vào tổng chuỗi?
3. Bạn cần k = 50 giá trị lớn nhất từ stream 10⁹ event/ngày. Vì sao heap là lựa chọn đúng thay vì quickselect, dù quickselect O(n) "nhanh hơn"?

---

*Chương tiếp theo: [17 — Linear Algebra](/series/math-for-engineers/level-4-advanced-mathematics/17-linear-algebra/), rời thế giới rời rạc của hoán vị và so sánh để bước vào không gian liên tục nhiều chiều — nơi "giống nhau về ngữ nghĩa" trở thành "gần nhau về góc", và AI hiện đại hóa ra là chuỗi phép nhân ma trận.*
