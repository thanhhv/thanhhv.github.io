+++
title = "Chương 13 — Divide and Conquer: Sức mạnh của chia đôi"
date = "2026-07-20T09:10:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 3 – Algorithm Mathematics
> Yêu cầu trước: Chương 06 (Recurrence), Chương 10 (Complexity Analysis)

---

## 1. Problem Statement

Năm 2004, Jeffrey Dean và Sanjay Ghemawat công bố paper MapReduce, mô tả cách Google đánh index toàn bộ web — hàng chục tỷ trang — bằng những cụm máy rẻ tiền, hỏng liên tục. Cộng đồng kỹ thuật lúc đó xem đây là một cuộc cách mạng về hệ thống phân tán. Nhưng nếu đọc kỹ paper, ý tưởng cốt lõi không mới: **chia bài toán khổng lồ thành các mảnh độc lập, giải từng mảnh, rồi ghép kết quả lại**. Đó chính là Divide and Conquer — kỹ thuật mà John von Neumann đã dùng để thiết kế merge sort từ năm 1945, gần 60 năm trước.

Cùng một ý tưởng đó xuất hiện ở quy mô nhỏ hơn hằng ngày: PostgreSQL sort 50GB dữ liệu với chỉ vài MB `work_mem`; Go runtime chia việc cho hàng nghìn goroutine rồi gom kết quả; binary search tìm một dòng trong log 10 tỷ record bằng 34 lần so sánh.

Câu hỏi mà chương này trả lời:

**Điều gì làm cho "chia nhỏ rồi ghép lại" mạnh đến vậy — và khi nào nó KHÔNG hoạt động?**

Nếu không có khung tư duy này, bạn chỉ thấy một tập hợp các thuật toán rời rạc phải học thuộc: merge sort, binary search, quickselect, Karatsuba... Có nó, bạn thấy tất cả là **một mẫu hình duy nhất với một công cụ phân tích duy nhất** — recurrence và Master Theorem của chương 06 — và quan trọng hơn, bạn biết cách tự thiết kế thuật toán mới theo mẫu hình đó.

## 2. Trực giác

### Vì sao "chia đôi" mạnh: số tầng là log n

Hãy làm một phép tính nhỏ. Bạn có bài toán kích thước n = 1.000.000. Mỗi lần chia đôi, kích thước giảm một nửa:

```
1.000.000 → 500.000 → 250.000 → ... → 2 → 1
```

Bao nhiêu lần chia? Chỉ **20 lần** — vì 2²⁰ ≈ 10⁶. Với n = 1 tỷ: 30 lần. Đây là mặt trái của câu chuyện tờ giấy gấp đôi ở chương 10: tăng trưởng nhân đôi bùng nổ nhanh bao nhiêu, thì **thu nhỏ chia đôi sụp đổ nhanh bấy nhiêu**. Bất kỳ quá trình nào giảm kích thước theo *tỷ lệ* (một nửa, một phần ba...) đều chạm đáy sau O(log n) bước.

Điều này cho D&C cấu trúc đặc trưng: một cây đệ quy có **log n tầng**. Nếu tổng chi phí mỗi tầng bị chặn bởi O(n), toàn bộ thuật toán là O(n log n). Toàn bộ "phép màu" của merge sort nằm gọn trong một câu đó.

```
Tầng 0:        [■■■■■■■■]              tổng công việc: n
Tầng 1:      [■■■■]  [■■■■]            tổng công việc: n
Tầng 2:    [■■] [■■] [■■] [■■]         tổng công việc: n
Tầng 3:   [■][■][■][■][■][■][■][■]     tổng công việc: n
          └────── log n tầng ──────┘   → tổng: n·log n
```

### Nhưng chia nhỏ không tự động thắng

Trực giác "chia nhỏ thì dễ hơn" là đúng nhưng chưa đủ. Thử chia bài toán "tìm phần tử lớn nhất" thành hai nửa: tìm max mỗi nửa rồi so sánh — được, nhưng vẫn O(n), không nhanh hơn duyệt thẳng. Thử chia bài toán "tính Fibonacci(n)" thành F(n−1) và F(n−2) — thảm họa O(2ⁿ), vì hai "nửa" gần như to bằng bài toán gốc và **trùng lặp nhau**.

D&C chỉ tạo ra lợi thế khi hội đủ **hai điều kiện**:

1. **Các bài toán con độc lập** — không chia sẻ công việc với nhau, không cần kết quả của nhau. (Nếu trùng lặp → đó là địa hạt của Dynamic Programming, chương 14.)
2. **Bước kết hợp (combine) rẻ** — ghép hai lời giải con thành lời giải lớn phải tốn ít hơn nhiều so với giải từ đầu. Merge hai mảng đã sort chỉ tốn O(n); đó là điểm tựa của merge sort. Nếu combine tốn O(n²), chia đôi vô nghĩa.

Hãy khắc hai điều kiện này vào trí nhớ, vì phần lớn nghệ thuật thiết kế D&C — và phần lớn cái khó của các bài phỏng vấn ở mục 8 — nằm ở việc tìm ra **bước combine rẻ**.

## 3. First Principles

### Nhìn thuật toán như một cái cây

Mọi thuật toán D&C đều có cùng bộ xương:

```
solve(bài toán P):
    nếu P đủ nhỏ: giải trực tiếp          (base case)
    chia P thành P₁, P₂, ..., Pₐ           (divide)
    giải đệ quy từng Pᵢ                    (conquer)
    ghép các lời giải con thành lời giải P  (combine)
```

Chi phí của nó được quyết định bởi đúng ba tham số: **a** — số bài toán con, **b** — hệ số thu nhỏ (mỗi bài toán con có kích thước n/b), và **f(n)** — chi phí divide + combine tại mỗi nút. Ba con số đó cho ta recurrence quen thuộc từ chương 06:

> T(n) = a·T(n/b) + f(n)

Đây là bước trừu tượng hóa quan trọng: ta không cần biết thuật toán *làm gì* để biết nó *tốn bao nhiêu*. Chỉ cần đếm a, b, f.

### Cuộc kéo co giữa hai lực

Cây đệ quy có log_b(n) tầng. Ở tầng thứ i có aⁱ nút, mỗi nút xử lý bài toán cỡ n/bⁱ. Tổng chi phí tầng i là aⁱ·f(n/bⁱ). Câu hỏi quyết định tất cả: **khi đi xuống sâu, tổng chi phí mỗi tầng tăng hay giảm?**

- Nếu số nút nhân lên (×a) nhanh hơn chi phí mỗi nút giảm đi — **lá thống trị**, tổng chi phí là số lá: Θ(n^log_b(a)).
- Nếu chi phí mỗi nút giảm nhanh hơn số nút tăng — **gốc thống trị**, tổng chi phí là f(n).
- Nếu hai lực cân bằng — mỗi tầng tốn như nhau, tổng là f(n) × số tầng: thêm một nhân tử log n.

Master Theorem (chương 06) chỉ là cách phát biểu hình thức của cuộc kéo co này. Con số n^log_b(a) — chi phí của riêng phần lá — là **trọng tâm hấp dẫn** của mọi thuật toán D&C, và như ta sẽ thấy ở Karatsuba, giảm **a** (số bài toán con) có sức công phá lớn hơn nhiều so với giảm f(n).

### Nếu không có khung tư duy này thì sao?

Không có D&C như một *nguyên lý*, mỗi thuật toán là một sáng chế riêng lẻ, và việc phân tích chi phí đệ quy là chuỗi lý luận ad-hoc dễ sai. Có nó, thiết kế thuật toán trở thành việc điền vào ba ô: chia thế nào (b), thành mấy phần (a), ghép ra sao (f) — rồi Master Theorem trả lời ngay "có đáng làm không". Và ở quy mô hệ thống, không có tư duy D&C thì không có MapReduce, không có external sort, không có parallel processing có cấu trúc — vì tất cả đều là câu trả lời cho cùng một câu hỏi: *làm sao biến một bài toán quá lớn cho một đơn vị xử lý thành nhiều bài toán vừa sức, mà bước ghép không nuốt hết lợi ích?*

## 4. Mathematical Model

### Merge sort — ví dụ chuẩn mực

Merge sort là D&C ở dạng tinh khiết nhất: chia đôi (a = 2, b = 2), combine bằng merge tuyến tính (f(n) = Θ(n)).

**Chi phí.** T(n) = 2T(n/2) + Θ(n). Theo Master Theorem: n^log₂2 = n, cùng bậc với f(n) = n → trường hợp cân bằng → **T(n) = Θ(n log n)**. Có thể thấy trực tiếp từ hình vẽ ở mục 2: mỗi tầng merge tổng cộng đúng n phần tử, có log n tầng.

**Tính đúng — chứng minh bằng strong induction (chương 03).** Mệnh đề P(n): "merge sort sắp xếp đúng mọi mảng n phần tử."

- *Base case:* n ≤ 1 — mảng rỗng hoặc một phần tử đã có thứ tự. P(0), P(1) đúng.
- *Bước quy nạp:* giả sử P(k) đúng với mọi k < n. Với mảng n phần tử, thuật toán chia thành hai nửa kích thước ⌊n/2⌋ và ⌈n/2⌉, đều nhỏ hơn n, nên theo giả thiết quy nạp cả hai nửa được sort đúng. Còn lại phải chứng minh **merge đúng**: merge duy trì invariant *"mảng kết quả luôn chứa, theo thứ tự tăng, toàn bộ các phần tử nhỏ hơn hoặc bằng mọi phần tử chưa lấy"* — mỗi bước nó lấy phần tử nhỏ nhất trong hai đầu còn lại, và vì hai nửa đã sort, đầu của mỗi nửa chính là min của nửa đó. Invariant giữ vững đến khi cả hai nửa cạn → kết quả là hoán vị có thứ tự của input. ∎

Cấu trúc chứng minh này đáng chú ý hơn nội dung của nó: **tính đúng của mọi thuật toán D&C đều quy về tính đúng của riêng bước combine**, phần đệ quy được induction "trả tiền" tự động. Khi review code D&C, hãy dồn sự nghi ngờ vào hàm combine và base case — đó là hai nơi duy nhất bug có thể trốn.

### Binary search — D&C suy biến

Binary search là trường hợp a = 1: chia đôi nhưng chỉ **giải một nửa**, nửa kia bị loại bỏ bằng một phép so sánh. Combine là không-làm-gì: f(n) = Θ(1).

> T(n) = T(n/2) + Θ(1) → n^log₂1 = n⁰ = 1, cân bằng với f(n) → T(n) = Θ(log n)

Gọi nó là "suy biến" không phải để hạ thấp — mà để thấy phổ của D&C: từ a = 1 (loại bỏ, chỉ còn log n) đến a = 2 (giải cả hai, n log n) đến a > b (số bài toán con phình nhanh hơn tốc độ thu nhỏ — chi phí đa thức bậc cao). Nhiều bài phỏng vấn khó (Search in Rotated Sorted Array, tìm peak element) thực chất chỉ hỏi một điều: *bạn có tìm được phép so sánh O(1) nào cho phép vứt bỏ một nửa không?*

### Karatsuba — giảm a quan trọng hơn giảm f

Nhân hai số n chữ số theo cách học ở tiểu học tốn Θ(n²). Năm 1960, Kolmogorov phỏng đoán trong seminar rằng n² là tối ưu; sinh viên 23 tuổi Anatoly Karatsuba bác bỏ ông trong một tuần.

Chia mỗi số làm đôi: x = x₁·10^m + x₀, y = y₁·10^m + y₀ (m = n/2). Khi đó:

> x·y = x₁y₁·10^2m + (x₁y₀ + x₀y₁)·10^m + x₀y₀

Cách chia "ngây thơ" cần **4** phép nhân của số n/2 chữ số: T(n) = 4T(n/2) + Θ(n) → n^log₂4 = **n²**. Chia đôi xong vẫn n² — vì lá thống trị, và số lá không giảm. Đây là bài học đầu tiên: *chia nhỏ mà không giảm tổng khối lượng ở lá thì vô ích*.

Nhận xét của Karatsuba: số hạng giữa có thể lấy từ ba phép nhân thay vì bốn, vì:

> x₁y₀ + x₀y₁ = (x₁ + x₀)(y₁ + y₀) − x₁y₁ − x₀y₀

Ba phép nhân: x₁y₁, x₀y₀, và (x₁+x₀)(y₁+y₀) — đổi lại vài phép cộng/trừ Θ(n) rẻ tiền. Recurrence mới:

> T(n) = **3**T(n/2) + Θ(n) → T(n) = Θ(n^log₂3) ≈ **Θ(n^1.585)**

Với n = 10⁶ chữ số: n² là 10¹² phép toán, n^1.585 khoảng 10^9.5 — nhanh hơn **hàng trăm lần**, và khoảng cách nới rộng vô hạn khi n tăng.

Bài học tổng quát, đáng giá hơn bản thân thuật toán: trong T(n) = aT(n/b) + f(n), khi lá thống trị, **a nằm trong số mũ** (n^log_b a) còn f(n) chỉ là số hạng cộng thêm. Giảm a từ 4 xuống 3 đổi cả *hình dạng* của hàm chi phí; tối ưu f(n) chỉ gọt hằng số. Cùng nguyên lý này, Strassen nhân ma trận với 7 phép nhân con thay vì 8 → O(n^2.807); và các thuật toán nhân số nguyên hiện đại (Toom-Cook, FFT — dùng trong thư viện `math/big` khi số đủ lớn) tiếp tục đẩy số mũ xuống sát 1.

### Closest pair of points — combine tinh xảo

Cho n điểm trên mặt phẳng, tìm cặp gần nhau nhất. Brute force: Θ(n²). D&C: chia đôi theo trục x, tìm khoảng cách nhỏ nhất δ trong mỗi nửa — nhưng cặp gần nhất có thể **vắt qua đường chia**. Bước combine tưởng như lại phải xét n/2 × n/2 cặp... cho đến khi hình học cứu ta: chỉ các điểm cách đường chia dưới δ mới đáng xét, và trong dải đó, mỗi điểm chỉ cần so với **tối đa 7 điểm** kế tiếp theo trục y (vì hai điểm cùng nửa không thể gần hơn δ, số điểm nhét vừa vào hình chữ nhật δ×2δ bị chặn bởi hằng số). Combine tuyến tính → T(n) = 2T(n/2) + Θ(n) = Θ(n log n).

Không cần nhớ chi tiết — cần nhớ mẫu hình: **khi combine tưởng chừng đắt, hãy tìm một tính chất cấu trúc (thứ tự, hình học, invariant) chặn số lượng tương tác giữa hai nửa xuống hằng số hoặc tuyến tính.** Đó chính xác là kỹ năng mà Count of Smaller Numbers After Self (mục 8) kiểm tra.

## 5. Thuật toán

### Merge sort trong Go

```go
// MergeSort sắp xếp tăng dần, trả về slice mới.
// T(n) = 2T(n/2) + Θ(n) = Θ(n log n); tốn Θ(n) bộ nhớ phụ.
func MergeSort(a []int) []int {
    if len(a) <= 1 { // base case: đã có thứ tự
        return a
    }
    mid := len(a) / 2
    left := MergeSort(a[:mid])   // conquer nửa trái
    right := MergeSort(a[mid:])  // conquer nửa phải
    return merge(left, right)    // combine — nơi duy nhất cần chứng minh
}

func merge(left, right []int) []int {
    out := make([]int, 0, len(left)+len(right))
    i, j := 0, 0
    // Invariant: out chứa (có thứ tự) đúng các phần tử ≤ mọi phần tử chưa lấy
    for i < len(left) && j < len(right) {
        if left[i] <= right[j] { // <= giữ tính stable
            out = append(out, left[i])
            i++
        } else {
            out = append(out, right[j])
            j++
        }
    }
    out = append(out, left[i:]...)  // phần còn thừa đã có thứ tự
    return append(out, right[j:]...)
}
```

### Quickselect — tìm phần tử thứ k trong O(n)

Muốn tìm median (hoặc phần tử thứ k) không cần sort cả mảng. Quickselect mượn bước partition của quicksort nhưng — như binary search — **chỉ đệ quy vào một phía**:

```go
// Quickselect trả về phần tử nhỏ thứ k (k tính từ 0). Làm thay đổi a.
// Kỳ vọng O(n); worst case O(n²) nếu pivot xui liên tục.
func Quickselect(a []int, k int) int {
    lo, hi := 0, len(a)-1
    for {
        if lo == hi {
            return a[lo]
        }
        // pivot ngẫu nhiên: biến worst case thành biến cố xác suất ~0 (chương 11)
        p := partition(a, lo, hi, lo+rand.Intn(hi-lo+1))
        switch {
        case k == p:
            return a[k]
        case k < p:
            hi = p - 1 // vứt bỏ toàn bộ phía phải
        default:
            lo = p + 1 // vứt bỏ toàn bộ phía trái
        }
    }
}

// partition dồn các phần tử < pivot về trái, trả về vị trí cuối của pivot.
func partition(a []int, lo, hi, pivotIdx int) int {
    pivot := a[pivotIdx]
    a[pivotIdx], a[hi] = a[hi], a[pivotIdx] // cất pivot ra cuối
    store := lo
    for i := lo; i < hi; i++ {
        if a[i] < pivot {
            a[store], a[i] = a[i], a[store]
            store++
        }
    }
    a[store], a[hi] = a[hi], a[store] // đưa pivot về đúng chỗ
    return store
}
```

Vì sao kỳ vọng O(n)? Với pivot ngẫu nhiên, mỗi vòng kỳ vọng loại bỏ một tỷ lệ hằng số của mảng, nên chi phí là chuỗi hình học n + cn + c²n + ... < n/(1−c) = O(n) — đúng kỹ thuật tổng hình học đã dùng cho slice append ở chương 10.

Còn nếu cần **đảm bảo** O(n) worst case? Thuật toán *median of medians* (Blum–Floyd–Pratt–Rivest–Tarjan, 1973) chọn pivot một cách tất định: chia mảng thành các nhóm 5 phần tử, lấy median mỗi nhóm, rồi đệ quy lấy median của các median đó làm pivot. Trực giác vì sao nó hoạt động: pivot này được đảm bảo lớn hơn ít nhất ~30% và nhỏ hơn ít nhất ~30% mảng — nên mỗi vòng *chắc chắn* vứt được ≥30%, cho recurrence T(n) ≤ T(n/5) + T(7n/10) + O(n). Vì 1/5 + 7/10 = 9/10 < 1 — tổng kích thước bài toán con *co lại* — chuỗi hình học hội tụ và T(n) = O(n). Trong thực tế hằng số của nó lớn nên production (kể cả `nth_element` của C++) dùng quickselect ngẫu nhiên kèm fallback; nhưng đây là một trong những kết quả đẹp nhất của lý thuyết: **tìm median không cần sort, và điều đó chứng minh được**.

### Fork-join song song trong Go

D&C có một món quà kèm theo: vì các bài toán con **độc lập** (điều kiện số 1), chúng chạy song song được mà không cần lock. Đây là mẫu fork-join:

```go
// ParallelSum tính tổng bằng cách chia đôi và fork goroutine.
// cutoff tránh trả phí goroutine cho bài toán quá nhỏ — xem mục 6.
func ParallelSum(a []int64) int64 {
    const cutoff = 1 << 14 // ~16k phần tử: dưới ngưỡng này chạy tuần tự
    if len(a) <= cutoff {
        var s int64
        for _, v := range a { // base case tuần tự: cache-friendly
            s += v
        }
        return s
    }
    mid := len(a) / 2
    ch := make(chan int64, 1)
    go func() { ch <- ParallelSum(a[:mid]) }() // fork nửa trái
    right := ParallelSum(a[mid:])              // nửa phải chạy tại chỗ
    return right + <-ch                        // join = combine
}
```

Với p core, span (đường găng) của cây là O(log n) tầng, nên speedup lý tưởng tiệm cận p. Điều kiện độc lập của D&C chính là điều kiện để **không có shared mutable state** — lý do các framework từ `sync.WaitGroup` pattern đến MapReduce đều đứng trên nền D&C.

## 6. Trade-off

**Đệ quy không miễn phí.** Mỗi lời gọi tốn stack frame, mỗi goroutine tốn ~vài KB stack và chi phí scheduler. Vì cây D&C có *rất nhiều* nút nhỏ ở đáy (một nửa số nút là lá!), mọi implementation nghiêm túc đều đặt **cutoff**: dưới ngưỡng nào đó, chuyển sang thuật toán tuần tự đơn giản. `sort.Slice` của Go rơi về insertion sort khi đoạn < 12 phần tử — đúng bài học "hằng số thắng bậc khi n nhỏ" của chương 10.

**Cache locality hai mặt.** Merge sort đọc/ghi tuần tự — tuyệt vời cho cache và cho đĩa (lý do nó thống trị external sort). Nhưng nó cần Θ(n) bộ nhớ phụ, còn quicksort partition tại chỗ. Đây là lý do quicksort thắng trên RAM còn merge sort thắng trên đĩa và trong yêu cầu stable sort.

**Chia đều quan trọng đến mức nào?** Master Theorem giả định chia đều n/b. Nếu chia lệch theo *tỷ lệ* cố định (ví dụ 1/10 và 9/10), số tầng vẫn là O(log n) — chỉ đổi cơ số log, tức đổi hằng số. Nhưng nếu chia lệch *tuyệt đối* (1 và n−1, như quicksort gặp pivot tệ nhất trên mảng đã sort), cây cao n tầng và O(n log n) sụp thành O(n²). Ranh giới giữa "lệch tỷ lệ" và "lệch tuyệt đối" là ranh giới giữa log n và n tầng — một trong những phân biệt tinh tế đáng nhớ nhất của chương này.

**Song song hóa vs chi phí điều phối.** Fork-join cho speedup gần tuyến tính khi công việc mỗi nhánh đủ lớn; với nhánh nhỏ, chi phí tạo goroutine/channel nuốt sạch lợi ích. Quy tắc thực dụng: fork đến khi số task ≈ vài lần số core hoặc kích thước chạm cutoff, không fork đến tận lá.

**Khi nào KHÔNG dùng D&C?** Ba dấu hiệu: (1) bài toán con **trùng lặp** — Fibonacci ngây thơ gọi F(n−2) hai lần, F(n−3) ba lần... cây đệ quy phình thành 2ⁿ nút trong khi chỉ có n bài toán *phân biệt*: dấu hiệu chuyển sang DP (chương 14); (2) **combine đắt ngang giải lại** — chia bài toán shortest path theo kiểu cắt đôi đồ thị thì ghép hai nửa khó chẳng kém bài gốc; (3) dữ liệu quá nhỏ hoặc thuật toán tuyến tính đơn giản đã tồn tại — tìm max bằng D&C đúng nhưng vô nghĩa.

## 7. Production Applications

**MapReduce / Spark — D&C phân tán.** Ánh xạ trực tiếp: *divide* = split input thành các chunk độc lập; *conquer* = hàm map chạy song song trên mỗi chunk, không chia sẻ state; *combine* = shuffle + reduce gom kết quả theo key. Hai điều kiện của D&C hiện nguyên hình thành hai ràng buộc thiết kế nổi tiếng: map phải **stateless/độc lập** (để retry và chạy song song tùy ý — chính là "bài toán con độc lập"), và reduce phải **kết hợp được, lý tưởng là associative + commutative** (để combine theo cây, tầng nào cũng rẻ). Khi bạn viết một job Spark bị "shuffle explosion", đó là lỗi cổ điển của D&C: bước combine đắt hơn bước conquer.

**External merge sort — khi ORDER BY tràn RAM.** PostgreSQL sort bảng 50GB với `work_mem = 64MB` như sau: đọc từng khúc vừa RAM, sort nội bộ (quicksort), ghi ra đĩa thành các *run* đã sort; sau đó k-way merge các run — chỉ cần giữ một trang của mỗi run trong RAM. Đây là merge sort với base case là "khúc vừa work_mem". Vì merge đọc ghi **tuần tự**, nó tận dụng tối đa băng thông đĩa. `EXPLAIN ANALYZE` hiện `Sort Method: external merge Disk: ...` chính là dấu vết của chương này trong production; tăng `work_mem` đủ lớn sẽ thấy nó đổi thành `quicksort` in-memory.

**Go runtime và thư viện chuẩn.** `sort.Sort` dùng pdqsort (pattern-defeating quicksort) từ Go 1.19 — một hybrid D&C: quicksort làm khung, heapsort làm lưới an toàn khi phát hiện phân hoạch lệch liên tục (bảo hiểm O(n log n) worst case), insertion sort ở đáy. `math/big` chuyển từ nhân trường học sang Karatsuba khi số vượt ngưỡng vài chục word. Timsort (Python, Java) cũng là hybrid merge sort khai thác các run có sẵn trong dữ liệu thực. Mẫu chung: **sản phẩm production không bao giờ là một thuật toán D&C thuần — luôn là D&C ở tầng cao + thuật toán hằng-số-nhỏ ở đáy + cơ chế phòng thủ worst case.**

**Parallel processing trong Go.** Mẫu fork-join ở mục 5 chính là cách các thư viện như `errgroup` và các pipeline xử lý ảnh/log chia việc: chia slice theo số core, mỗi goroutine xử lý một khúc độc lập, channel/WaitGroup làm bước join. Nhìn qua lăng kính D&C giúp trả lời câu hỏi thiết kế thường gặp: *chia mấy phần?* — đủ để bão hòa core nhưng mỗi phần trên cutoff; *combine ở đâu?* — nếu combine cần lock chung, hãy tổ chức lại cho mỗi nhánh trả kết quả riêng rồi gộp theo cây.

## 8. Interview

D&C xuất hiện trong phỏng vấn dưới hai dạng: bài trực tiếp ("hãy chia đôi") và — khó hơn — bài mà D&C là lời giải *ẩn* sau một đề bài trông như đếm hoặc tìm kiếm.

**Merge K Sorted Lists (LeetCode 23).** Cách ngây thơ: merge lần lượt list 1 với 2, kết quả với 3... — list đầu bị copy k lần, tổng O(nk). Nhìn theo D&C: merge **theo cặp, theo tầng** như cây merge sort — mỗi tầng tổng chi phí O(n), có log k tầng → O(n log k). Cùng đáp số với cách dùng heap, nhưng lời giải D&C cho thấy bạn hiểu *vì sao* log k xuất hiện: đó là chiều cao cây.

**Kth Largest Element (LeetCode 215).** Chính là quickselect. Bẫy phổ biến: trả lời "sort rồi lấy" O(n log n) và dừng ở đó. Người phỏng vấn muốn nghe: kỳ vọng O(n) với pivot ngẫu nhiên, và *biết về sự tồn tại* của median of medians cho O(n) tất định. Điểm cộng lớn: nói được vì sao k trong heap-solution O(n log k) thắng khi k nhỏ và dữ liệu streaming.

**Search in Rotated Sorted Array (LeetCode 33).** Binary search biến thể — bài kiểm tra xem bạn hiểu *nguyên lý* hay chỉ thuộc code. Mảng xoay không sort toàn cục, nhưng invariant sống sót: **ít nhất một trong hai nửa quanh mid luôn sort hoàn chỉnh** — so sánh a[lo] với a[mid] biết ngay nửa nào, rồi kiểm tra target có nằm trong nửa sort đó không để quyết định vứt nửa nào. Kỹ năng tổng quát: khi đề cho cấu trúc "gần như có thứ tự", hãy đi tìm *câu hỏi O(1) cho phép loại bỏ một nửa* — đó là định nghĩa vận hành của binary search.

**Count of Smaller Numbers After Self (LeetCode 315).** Bài "nhìn ra combine step" điển hình. Đề trông như bài đếm O(n²). Chìa khóa: trong lúc merge sort, khi một phần tử của nửa **phải** được lấy trước phần tử của nửa trái, nó nhỏ hơn *toàn bộ phần còn lại* của nửa trái — đếm được cả loạt nghịch thế trong O(1). Toàn bộ thông tin "bao nhiêu phần tử sau nhỏ hơn" được thu hoạch **miễn phí trong bước combine**. Đây là mẫu tư duy đáng học nhất chương: *bài toán tương tác giữa các cặp (i, j) với i < j có thể tính được trong lúc merge, vì merge là khoảnh khắc duy nhất hai nửa gặp nhau.* Cùng mẫu: Count Inversions, Reverse Pairs (LeetCode 493).

**Lỗi tư duy thường gặp:**

- Viết đệ quy nhưng quên base case cho mảng rỗng/một phần tử — 90% bug D&C nằm ở biên.
- Mặc định "chia đôi là O(n log n)" mà không nhìn chi phí combine: T(n) = 2T(n/2) + O(n²) là O(n²), không phải n² log n cũng không phải n log n — dùng Master Theorem, đừng đoán.
- Không nhận ra bài toán con trùng lặp, viết D&C thuần cho bài DP và nhận TLE với độ phức tạp mũ.

**Cách phân tích thay vì học thuộc:** hỏi tuần tự — (1) nếu tôi có lời giải của hai nửa, tôi cần thêm thông tin gì để trả lời cho toàn mảng? (2) thông tin đó tính được trong O(n) hay O(log n) không? (3) recurrence là gì, Master Theorem nói gì? Nếu câu (1) bế tắc vì hai nửa "dính" nhau — nghĩ DP hoặc đổi cách chia.

## 9. Anti-pattern

**D&C trên bài toán con trùng lặp.** Fibonacci đệ quy thuần là ví dụ giáo khoa, nhưng phiên bản production có thật: hàm tính giá đệ quy trên cây danh mục mà các nhánh chia sẻ node con (thực chất là DAG), gọi lặp lại hàng nghìn lần vào cùng node. Triệu chứng: thời gian chạy tăng theo cấp số nhân với độ sâu. Chẩn đoán bằng một câu hỏi: *số bài toán con **phân biệt** là bao nhiêu so với số nút của cây đệ quy?* Chênh lệch lớn → memoization (chương 14).

**Đệ quy đến tận lá.** Fork goroutine cho mảng 8 phần tử; gọi hàm đệ quy cho đoạn 3 phần tử. Cây nhị phân có n lá thì có n−1 nút trong — chi phí điều phối tỷ lệ với số nút, và số nút bùng nổ ở đáy. Luôn có cutoff.

**Combine giấu chi phí bậc cao.** Nối hai kết quả bằng `append` copy toàn bộ, hoặc combine bằng cách sort lại kết quả gộp — T(n) = 2T(n/2) + O(n log n) vẫn ổn (Θ(n log² n)) nhưng T(n) = 2T(n/2) + O(n²) thì mất trắng lợi ích chia. Nhìn kỹ hàm combine như đã nhìn `s += w` ở chương 10.

**Chia theo giá trị pivot xấu trên dữ liệu adversarial.** Quicksort/quickselect với pivot tất định (phần tử đầu/giữa) trên input do người dùng kiểm soát là lỗ hổng DoS thật — cùng họ với hash flooding ở chương 10. Luôn random hóa pivot, hoặc dùng thư viện chuẩn vốn đã có lưới an toàn.

**MapReduce hóa mọi thứ.** Chạy job phân tán cho dữ liệu 2GB — chi phí điều phối cluster, serialize, shuffle vượt xa việc xử lý trên một máy bằng một vòng for. "Big data" bắt đầu ở nơi dữ liệu không vừa một máy, không sớm hơn. D&C phân tán cũng phải tôn trọng cutoff.

## 10. Best Practices

**Nên:**

- Thiết kế bằng cách trả lời đúng ba câu hỏi theo thứ tự: *chia thế nào để các phần độc lập? combine cần thông tin gì và tốn bao nhiêu? recurrence cho ra gì?* — viết recurrence **trước khi** viết code.
- Dồn effort kiểm thử vào base case và hàm combine — induction đảm bảo phần còn lại. Test biên: mảng rỗng, 1 phần tử, 2 phần tử, toàn phần tử bằng nhau.
- Đặt cutoff và benchmark để chọn ngưỡng (thường 10–50 cho đệ quy thuần, 10³–10⁵ phần tử cho goroutine).
- Khi cần tìm thứ hạng/median/top-k: nghĩ quickselect O(n) trước khi sort O(n log n); khi k nhỏ và dữ liệu streaming: heap.
- Trong hệ phân tán, thiết kế hàm reduce associative + commutative để combine được theo cây và chịu được retry.

**Không nên:**

- Không dùng D&C khi bài toán con trùng lặp (→ DP) hoặc khi lời giải tuyến tính một vòng lặp đã tồn tại.
- Không tin "chia đôi = nhanh" mà bỏ qua chi phí combine — Master Theorem tồn tại để thay thế niềm tin.
- Không viết lại sort/select — `sort.Slice`, `slices.Sort` đã là hybrid được tinh chỉnh hơn mọi bản tự viết.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Karatsuba đổi 4 phép nhân lấy 3 phép nhân cộng thêm vài phép cộng O(n). Vì sao thương vụ này lãi lớn đến vậy, còn việc tối ưu bước cộng thì gần như vô nghĩa?
2. Quicksort chia lệch 1/10 – 9/10 vẫn là O(n log n), nhưng chia lệch 1 – (n−1) là O(n²). Ranh giới bản chất giữa hai kiểu "lệch" này là gì?
3. Trong Count of Smaller Numbers After Self, vì sao thông tin đếm phải được thu hoạch đúng tại bước merge mà không thể lấy sau khi sort xong?

---

*Chương tiếp theo: [14 — Dynamic Programming](/series/math-for-engineers/level-3-algorithm-mathematics/14-dynamic-programming/), nơi ta xử lý chính cái bẫy mà D&C bất lực: khi các bài toán con trùng lặp nhau, đừng chia để trị — hãy nhớ để khỏi tính lại.*
