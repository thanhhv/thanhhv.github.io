+++
title = "Chương 06 — Recurrence Relations: Ngôn ngữ của đệ quy"
date = "2026-07-20T08:00:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 2 – Discrete Mathematics
> Yêu cầu trước: Chương 03 (Proof Techniques), Chương 05 (Counting & Combinatorics)

---

## 1. Problem Statement

Một kỹ sư viết hàm tính giá cho sản phẩm bảo hiểm: giá năm thứ n phụ thuộc vào giá hai năm trước đó theo công thức nghiệp vụ. Cách tự nhiên nhất là dịch thẳng công thức thành đệ quy. Code đúng tuyệt đối, unit test xanh với n = 10, n = 20. Rồi một khách hàng nhập hợp đồng 50 năm — và request treo. Không lỗi, không panic, không log. CPU 100%. Với n = 50, hàm đó thực hiện khoảng **hai mươi tỷ lời gọi** cho một phép tính mà bản chất chỉ cần 50 phép cộng.

Điều đáng sợ: giữa n = 20 (tức thì) và n = 50 (treo vô hạn theo cảm nhận người dùng), *không có dấu hiệu cảnh báo nào* nếu bạn không biết đọc **hình dạng chi phí** của hàm đệ quy. Và hình dạng đó được viết bằng một ngôn ngữ riêng:

**Quan hệ truy hồi (recurrence relation): phương trình định nghĩa một đại lượng tại n thông qua chính nó tại các giá trị nhỏ hơn.**

Đây không phải công cụ chuyên dùng cho vài bài toán lẻ. Mọi hàm đệ quy đều *ngầm định* một recurrence cho chi phí của nó; mọi thuật toán chia để trị đều được phân tích bằng recurrence; mọi lời giải Dynamic Programming đều bắt đầu bằng việc viết ra một recurrence. Câu hỏi của chương này: **cho một recurrence, làm sao biết nó tăng trưởng thế nào — logarit, tuyến tính, n log n, hay bùng nổ mũ — mà không cần chạy thử?** Không trả lời được câu hỏi đó, bạn không có cách nào phân biệt đệ quy vô hại với quả bom hẹn giờ ở đầu chương.

## 2. Trực giác

### Đệ quy là code, recurrence là chi phí của code

Nhìn hai thế giới song song:

```go
// Thế giới code                          // Thế giới toán
func fib(n int) int {                     // F(n) = F(n-1) + F(n-2)
    if n <= 1 {                           // F(0) = 0, F(1) = 1
        return n
    }
    return fib(n-1) + fib(n-2)
}
```

Recurrence bên phải mô tả *giá trị* hàm tính ra. Nhưng còn một recurrence thứ hai, ẩn hơn và quan trọng hơn với kỹ sư — mô tả *chi phí* để tính:

> T(n) = T(n−1) + T(n−2) + O(1)

Mỗi lời gọi `fib(n)` phải trả chi phí của `fib(n-1)`, cộng chi phí của `fib(n-2)`, cộng một chút việc riêng (so sánh, cộng). Cấu trúc lời gọi của code *chính là* cấu trúc của recurrence — dịch code sang recurrence gần như là phép sao chép cú pháp. Kỹ năng thật nằm ở bước sau: nhìn recurrence, đoán ra hình dạng nghiệm.

### Vẽ cây để thấy vụ nổ

Với `fib(5)`, hãy vẽ cây lời gọi:

```
                        fib(5)
                 ┌────────┴────────┐
              fib(4)             fib(3)
           ┌────┴────┐         ┌───┴───┐
        fib(3)     fib(2)   fib(2)   fib(1)
       ┌──┴──┐    ┌──┴──┐  ┌──┴──┐
    fib(2) fib(1) f(1) f(0) f(1) f(0)
   ┌──┴──┐
 fib(1) fib(0)
```

Hai điều đập vào mắt. Thứ nhất, `fib(3)` được tính 2 lần, `fib(2)` 3 lần, `fib(1)` 5 lần — **cùng một việc bị làm đi làm lại**, và số lần lặp lại tăng dần khi xuống sâu (bản thân dãy số lần lặp cũng là... Fibonacci). Thứ hai, cây *gần như* nhân đôi ở mỗi tầng và sâu n tầng — nên tổng số node cỡ 2ⁿ. Chính xác hơn, số lời gọi tăng theo φⁿ với φ ≈ 1.618 (sẽ thấy vì sao ở mục 4) — với n = 50 là khoảng 2 × 10¹⁰ lời gọi. Vụ treo ở đầu chương không còn bí ẩn: nó nằm sẵn trong hình dạng của cây.

Trực giác cần khắc sâu: **đệ quy gọi chính nó k lần với bài toán chỉ nhỏ đi một hằng số → cây phình theo kⁿ → bùng nổ mũ.** Ngược lại, đệ quy gọi chính nó trên bài toán *nhỏ đi theo tỷ lệ* (một nửa, một phần ba) → cây chỉ sâu log n → chi phí lành tính. Ranh giới giữa "trừ đi" và "chia cho" là ranh giới giữa thảm họa và hiệu quả — cùng một trực giác với geometric growth ở phân tích slice append (chương 10).

### Vì sao trực giác thường thất bại?

Vì code đệ quy *trông* ngắn. `return fib(n-1) + fib(n-2)` — một dòng, hai lời gọi, có gì đâu? Mắt người đọc chi phí theo số dòng code; recurrence đọc chi phí theo cấu trúc nhân bản của lời gọi. Một dòng code nhân bản chính nó hai lần mỗi cấp là một dòng code có 2ⁿ hậu duệ.

## 3. First Principles

### Recurrence = induction viết ngược

Ở chương 03, induction chứng minh mệnh đề P(n) bằng cách: chứng minh P(base), rồi chứng minh P(n−1) → P(n). Recurrence *định nghĩa* đại lượng T(n) bằng đúng cấu trúc đó: cho T(base), cho quy tắc T(nhỏ) → T(n). Đệ quy trong code, induction trong chứng minh, recurrence trong phân tích — **cùng một ý tưởng tự quy chiếu, mặc ba bộ quần áo**. Đó là lý do một recurrence luôn cần đủ hai phần: *base case* và *bước truy hồi* — thiếu base case, recurrence không định nghĩa gì cả, hệt như đệ quy thiếu điều kiện dừng thì stack overflow.

### Giải recurrence nghĩa là gì?

Là tìm **closed form** — công thức tính thẳng T(n) không qua tự quy chiếu — hoặc yếu hơn nhưng thường đủ dùng: tìm **bậc tăng trưởng** Θ của nó. Với kỹ sư, 95% nhu cầu là loại sau: không cần biết T(n) = 3n log n + 7n − 2, chỉ cần biết Θ(n log n) để tra bảng ngưỡng chương 10.

### Kỹ thuật 1 — Unrolling (substitution): cứ thay và nhìn pattern

Binary search: T(n) = T(n/2) + 1. Thay chính định nghĩa vào chính nó:

> T(n) = T(n/2) + 1
>      = [T(n/4) + 1] + 1 = T(n/4) + 2
>      = T(n/8) + 3
>      = ... = T(n/2ᵏ) + k

Dừng khi chạm base case: n/2ᵏ = 1 → k = log₂n → **T(n) = Θ(log n)**. Toàn bộ kỹ thuật là: khai triển vài bước, nhận ra pattern theo k, giải phương trình "bao giờ chạm đáy". (Nghiêm ngặt thì pattern đoán được cần xác nhận bằng induction — khép lại vòng tròn với chương 03 — nhưng trong thực hành kỹ sư, pattern rõ ràng là đủ.)

Một ví dụ nữa cho thấy unrolling phát hiện điều phi trực giác. Hàm đệ quy "xử lý phần tử đầu rồi gọi tiếp trên phần còn lại, nhưng mỗi lần copy slice": T(n) = T(n−1) + n. Unroll: T(n) = n + (n−1) + (n−2) + ... + 1 = n(n+1)/2 = **Θ(n²)**. Code trông tuyến tính, chi phí bình phương — đúng cái bẫy `s += w` của chương 10, giờ được nhìn bằng ngôn ngữ recurrence.

### Kỹ thuật 2 — Recursion tree: cộng chi phí theo tầng

Với recurrence phức tạp hơn, vẽ cây và cộng theo tầng. Merge sort: T(n) = 2T(n/2) + n (chia đôi, sort hai nửa, merge tốn n):

```
Tầng 0:              n                    → chi phí tầng: n
                 ┌───┴───┐
Tầng 1:        n/2      n/2               → chi phí tầng: n
             ┌──┴──┐  ┌──┴──┐
Tầng 2:    n/4  n/4  n/4  n/4             → chi phí tầng: n
             ...........
Tầng log n: 1 1 1 ... 1  (n lá)           → chi phí tầng: n
```

Mỗi tầng, số mảnh nhân đôi nhưng mỗi mảnh nhỏ đi một nửa — chi phí mỗi tầng **không đổi, bằng n**. Có log₂n tầng → T(n) = Θ(n log n). Cái nhìn "chi phí mỗi tầng × số tầng" là công cụ tư duy mạnh nhất chương này: nó trả lời được cả những recurrence mà công thức đóng khó chịu, và nó cho *hiểu* chứ không chỉ cho đáp số.

### Nếu không có công cụ này thì sao?

Cách duy nhất còn lại để biết hàm đệ quy đắt cỡ nào là chạy thử với n tăng dần — chính là "benchmark mọi thứ" mà chương 10 đã chỉ ra là bế tắc, và với hàm mũ thì còn tệ hơn: n = 40 chạy được, n = 50 treo, và bạn không có cách nào *ngoại suy* nếu không biết hình dạng hàm. Recurrence là công cụ biến "chạy thử và cầu nguyện" thành phép tính 30 giây trên giấy.

## 4. Mathematical Model

### Master Theorem: bảng tra cho chia để trị

Đại đa số recurrence từ thuật toán chia để trị có dạng chuẩn:

> **T(n) = a·T(n/b) + f(n)**
> — chia bài toán thành **a** bài con, mỗi bài kích thước **n/b**, tốn **f(n)** để chia và gộp.

Nhìn bằng recursion tree: tầng gốc tốn f(n); số node mỗi tầng nhân **a**, kích thước mỗi node chia **b**; cây sâu log_b n; tầng lá có a^(log_b n) = **n^(log_b a)** lá. Toàn bộ Master Theorem là câu trả lời cho một câu hỏi duy nhất: **chi phí dồn về đâu — gốc, lá, hay trải đều?** Tức là cuộc so găng giữa f(n) (chi phí phần "trị" ở gốc) và n^(log_b a) (số lượng lá do phần "chia" sinh ra):

| Case | Điều kiện | Ai thắng | Kết quả | Trực giác |
|---|---|---|---|---|
| 1 | f(n) nhỏ hơn n^(log_b a) (cách nhau một lũy thừa) | **Chia thắng** | T(n) = Θ(n^(log_b a)) | Cây phình quá nhanh, chi phí dồn hết về tầng lá |
| 2 | f(n) = Θ(n^(log_b a)) | **Hòa** | T(n) = Θ(n^(log_b a) · log n) | Mọi tầng tốn ngang nhau, nhân với số tầng |
| 3 | f(n) lớn hơn n^(log_b a) (cách nhau một lũy thừa)* | **Trị thắng** | T(n) = Θ(f(n)) | Gốc đắt áp đảo, các tầng dưới không đáng kể |

*(Case 3 kèm điều kiện kỹ thuật "regularity" — a·f(n/b) ≤ c·f(n) với c < 1 — hầu như luôn thỏa với các f(n) đa thức gặp trong thực tế.)*

Áp dụng cho ba thuật toán quen mặt:

- **Merge sort:** T(n) = 2T(n/2) + n. So sánh: n^(log₂2) = n¹ với f(n) = n — bằng nhau → Case 2 → **Θ(n log n)**. Khớp với recursion tree ở mục 3.
- **Binary search:** T(n) = 1·T(n/2) + 1. n^(log₂1) = n⁰ = 1 với f(n) = 1 — bằng nhau → Case 2 → **Θ(log n)**.
- **Karatsuba** (nhân hai số n chữ số): thay vì 4 phép nhân nửa-kích-thước như cách naive, Karatsuba khéo léo chỉ cần **3**: T(n) = 3T(n/2) + n. So sánh: n^(log₂3) = n^1.585 với n — chia thắng → Case 1 → **Θ(n^1.585)**, thay vì Θ(n²). Bài học sâu hơn cả đáp số: trong Case 1, *số nhánh a là đòn bẩy mạnh nhất* — giảm a từ 4 xuống 3 đổi cả số mũ của nghiệm, trong khi tối ưu f(n) không thay đổi gì. Biết chi phí dồn về đâu cho biết tối ưu chỗ nào là vô ích.

Master Theorem *không* áp dụng cho: bài con giảm theo hiệu (T(n−1) — dùng unrolling), chia không đều (T(n/3) + T(2n/3) — dùng recursion tree), hay số bài con thay đổi theo n. Nó là bảng tra cho dạng chuẩn, không phải chìa khóa vạn năng — và nhận ra "recurrence này không thuộc dạng chuẩn" cũng là một kỹ năng được chấm điểm trong phỏng vấn.

### Recurrence tuyến tính và closed form: vì sao Fibonacci mọc theo lũy thừa

F(n) = F(n−1) + F(n−2) thuộc họ **linear recurrence** — tổ hợp tuyến tính của các giá trị trước. Họ này có lý thuyết nghiệm đẹp, và chỉ cần nắm mức trực giác: **nghiệm của linear recurrence là (tổ hợp của các) hàm mũ**. Vì sao? Thử đoán nghiệm dạng T(n) = xⁿ và thay vào: xⁿ = xⁿ⁻¹ + xⁿ⁻² → chia hai vế cho xⁿ⁻²: **x² = x + 1**. Recurrence vô hạn chiều rút gọn thành một phương trình bậc hai (phương trình đặc trưng), nghiệm x = (1±√5)/2. Nghiệm dương là **golden ratio φ ≈ 1.618**, và:

> F(n) ≈ φⁿ/√5 — Fibonacci là hàm mũ cơ số 1.618 đội lốt phép cộng.

Hai hệ quả thực dụng. Một: chi phí của fib đệ quy naive cũng thỏa gần đúng recurrence ấy, nên số lời gọi ≈ φⁿ — con số hai mươi tỷ ở mục 1 đến từ đây (φ⁵⁰/√5 ≈ 1.2 × 10¹⁰, nhân đôi cho tổng số node). Hai: bất cứ khi nào một đại lượng được định nghĩa "bằng tổng vài giá trị trước của chính nó" — số node cây sau n tầng nhân bản, số request sau n vòng retry khuếch đại — hãy mặc định nó **tăng theo lũy thừa** cho đến khi chứng minh được điều ngược lại.

### Mọi phân tích chia để trị đều quy về đây

Điều đáng nhận ra ở tầm kiến trúc của tài liệu này: chương 10 dùng kết quả "merge sort là Θ(n log n)" như đồ có sẵn; chương 13 (Divide and Conquer) sẽ thiết kế thuật toán bằng cách *nhắm trước* vào recurrence có nghiệm đẹp; chương 14 (DP) là nghệ thuật giải recurrence bằng cách điền bảng thay vì gọi đệ quy. Recurrence là điểm hội tụ: **thiết kế thuật toán = chọn một recurrence; phân tích thuật toán = giải recurrence đó.**

## 5. Thuật toán

Nếu recurrence cho biết fib naive là φⁿ, nó cũng chỉ ra thuốc chữa: cây bùng nổ *vì cùng một bài con bị tính lại*. Số bài con **phân biệt** chỉ có n+1 (fib(0)..fib(n)). Đừng tính lại — nhớ lấy:

```go
// Memoization (top-down): giữ nguyên cấu trúc đệ quy,
// nhưng mỗi bài con chỉ tính một lần. T(n): φⁿ → O(n).
func fibMemo(n int, memo map[int]int) int {
    if n <= 1 {
        return n
    }
    if v, ok := memo[n]; ok {
        return v // bài con đã giải — trả kết quả, cắt cụt cả nhánh cây
    }
    memo[n] = fibMemo(n-1, memo) + fibMemo(n-2, memo)
    return memo[n]
}

// Bottom-up: giải recurrence theo chiều xuôi, từ base case đi lên.
// Cùng O(n) thời gian, nhưng O(1) bộ nhớ vì recurrence chỉ nhìn lùi 2 bước.
func fibIter(n int) int {
    if n <= 1 {
        return n
    }
    prev2, prev1 := 0, 1 // F(0), F(1)
    for i := 2; i <= n; i++ {
        prev2, prev1 = prev1, prev1+prev2 // trượt cửa sổ 2 giá trị
    }
    return prev1
}
```

Cùng một recurrence, ba cách thực thi — đệ quy naive φⁿ, memoization O(n), vòng lặp O(n) với O(1) bộ nhớ. **Recurrence là đặc tả; chiến lược tính là quyết định riêng.** Đây chính là hạt giống của chương 14.

### Kỹ năng 30 giây: đọc hình dạng chi phí từ code đệ quy

Quy trình bốn câu hỏi, áp dụng được cho mọi hàm đệ quy gặp trong code review hay phỏng vấn:

1. **Mỗi lời gọi đẻ ra mấy lời gọi con?** → hệ số a (độ phình của cây).
2. **Bài toán nhỏ đi thế nào?** — chia tỷ lệ (n/b → cây sâu log n) hay trừ hằng số (n−c → cây sâu n/c)?
3. **Việc riêng mỗi lời gọi tốn bao nhiêu?** → f(n), nhớ soi chi phí ẩn (copy slice, nối chuỗi).
4. **Ghép lại:** chia tỷ lệ → Master Theorem; trừ hằng số với a = 1 → cộng f theo chuỗi (unrolling); trừ hằng số với a ≥ 2 → **aⁿ, báo động đỏ** — trừ khi có memoization và số bài con phân biệt nhỏ.

Ví dụ tốc hành: hàm sinh mọi tập con — 2 nhánh, n giảm 1, f = O(1) → 2ⁿ (khớp phép đếm chương 05, không thể tốt hơn vì output chừng đó). Quicksort trung bình — 2 nhánh, chia tỷ lệ, f = n → n log n. Hàm duyệt cây nhị phân — 2 nhánh nhưng *tổng kích thước bài con = n − 1*, mỗi node bị thăm đúng một lần → O(n): khi cây lời gọi phủ mỗi phần tử một lần, đếm theo tổng công việc chứ đừng đếm theo công thức nhân.

## 6. Trade-off

**Master Theorem vs recursion tree vs unrolling.** Master Theorem nhanh nhất nhưng hẹp nhất; recursion tree chậm hơn một phút nhưng xử lý được chia không đều và cho trực giác "chi phí dồn về đâu"; unrolling tổng quát nhất cho recurrence một nhánh. Học cả ba nhưng theo thứ tự ưu tiên ngược: hiểu recursion tree trước, vì Master Theorem chỉ là recursion tree đóng gói sẵn — dùng bảng tra mà không có trực giác cây là học thuộc, và học thuộc thì sụp đổ ngay khi đề lệch chuẩn một milimét.

**Closed form vs mô phỏng.** Closed form của Fibonacci (công thức Binet với φⁿ) đẹp về lý thuyết nhưng dùng số thực — sai số làm hỏng kết quả từ n cỡ 70+ với float64; vòng lặp O(n) số nguyên vừa đúng tuyệt đối vừa quá đủ nhanh. Bài học tổng quát: closed form quý nhất ở chỗ cho biết *bậc tăng trưởng* để ra quyết định; để lấy *giá trị chính xác*, tính theo recurrence thường an toàn hơn.

**Memoization: đổi bộ nhớ lấy thời gian — và không phải lúc nào cũng đáng.** Memoization chỉ thắng khi các bài con *trùng lặp* (chương 14 gọi là overlapping subproblems). Merge sort không có bài con trùng — memoize nó chỉ tốn bộ nhớ vô ích. Và cache kết quả cũng có chi phí quản lý vòng đời: memo lớn lên theo số bài con phân biệt, có thể là O(n²) bộ nhớ với bài toán hai tham số.

**Khi nào KHÔNG cần giải recurrence?** Khi n bị chặn nhỏ bởi nghiệp vụ (đệ quy trên cây tổ chức công ty sâu tối đa 10 cấp) — mọi hình dạng hàm đều vô hại ở n = 10. Ngược lại, *bắt buộc* phải giải khi n do người dùng hoặc dữ liệu quyết định — đó chính là ranh giới bị bỏ qua trong sự cố mở đầu chương.

## 7. Production Applications

**B-Tree: chiều cao là nghiệm của một recurrence.** Số bản ghi chứa được trong B-Tree cao h với bậc m thỏa N(h) = m·N(h−1), N(0) = 1 → N(h) = mʰ, tức h = log_m N. Đảo lại chính là khẳng định trung tâm của chương 07: chiều cao — và số disk I/O mỗi lookup — chỉ tăng thêm 1 khi dữ liệu **nhân** m lần. Mọi phân tích "index tra trong mấy bước" là giải một recurrence hình học.

**Exponential backoff là một recurrence được chọn có chủ đích.** Client retry với delay(k) = 2·delay(k−1) — recurrence hình học, closed form delay(k) = base·2ᵏ, tổng thời gian chờ sau k lần ≈ base·(2ᵏ⁺¹ − 1). Vì sao nhân 2 mà không cộng thêm 1 giây mỗi lần? Cùng lý do slice Go nhân đôi capacity (chương 10), nhìn từ chiều ngược lại: backoff cộng dồn tuyến tính giữ *tần suất* retry gần như không giảm khi hệ thống đang quá tải kéo dài, còn backoff hình học giảm tần suất theo lũy thừa — áp lực lên hệ thống hồi phục tắt dần thay vì duy trì. Chọn hệ số của recurrence là chọn hành vi của cả hệ thống dưới sự cố; AWS SDK, gRPC, Kafka producer đều mã hóa đúng recurrence này (kèm jitter — chương 11 giải thích vì sao cần ngẫu nhiên hóa).

**Fan-out và retry storm: recurrence mà không ai cố ý viết ra.** Service A gọi 3 service con, mỗi service con gọi tiếp 3 service cháu... — số request ở độ sâu d là R(d) = 3·R(d−1) = 3ᵈ. Thêm retry 3 lần mỗi cạnh khi có sự cố, hệ số nhân thành 9. Một kiến trúc microservice sâu 5 tầng với fan-out và retry "vừa phải" có thể khuếch đại một request người dùng thành hàng nghìn request nội bộ — chính là **retry storm** đã đánh sập nhiều hệ thống lớn. Kỹ sư nhìn ra recurrence ẩn sẽ đặt retry budget và cap độ sâu *trước khi* nhìn thấy nó trong postmortem.

**Go runtime — hai recurrence quen thuộc:** stack của goroutine khởi đầu 8KB và *nhân đôi* khi thiếu — lại geometric growth để amortized O(1) như slice; garbage collector kích hoạt khi heap tăng theo **tỷ lệ** (mặc định GOGC=100: heap gấp đôi sau mỗi chu kỳ GC) thay vì theo lượng cố định — cùng một bài học "tăng theo tỷ lệ thì tổng chi phí tuyến tính" áp vào quản lý bộ nhớ.

## 8. Interview

Recurrence xuất hiện trong phỏng vấn dưới hai bộ mặt: bài toán *mô hình hóa bằng* recurrence, và câu hỏi *phân tích độ phức tạp* của code đệ quy cho sẵn.

**Climbing Stairs (LeetCode 70).** Leo n bậc thang, mỗi bước 1 hoặc 2 bậc — bao nhiêu cách? Đừng nhớ đáp án; hãy tái tạo bằng lập luận combinatorial của chương 05: nhìn vào *bước cuối cùng* — hoặc là bước 1 bậc (trước đó đứng ở bậc n−1), hoặc bước 2 bậc (trước đó ở n−2); hai trường hợp rời nhau, rule of sum cho **W(n) = W(n−1) + W(n−2)** — Fibonacci đội lốt. Chuỗi kỹ năng trọn vẹn được kiểm tra: mô hình hóa thành recurrence (chương này) → nhận ra đệ quy naive là φⁿ → memoization/bottom-up O(n) → tối ưu O(1) bộ nhớ vì chỉ nhìn lùi 2 bước. Các biến thể (bước 1/2/3 bậc; mỗi bậc có chi phí — Min Cost Climbing Stairs LC 746) chỉ đổi recurrence, không đổi quy trình — đó là lý do học quy trình thắng học thuộc.

**Phân tích hàm đệ quy cho sẵn** — dạng "đọc code, cho biết Big-O" mà interviewer dùng để tách người phân tích khỏi người học thuộc:

```go
func mystery(n int) int {
    if n <= 1 {
        return 1
    }
    return mystery(n/2) + mystery(n/2) // hai lời gọi, mỗi cái n/2
}
```

Quy trình 30 giây của mục 5: a = 2, chia tỷ lệ b = 2, f = O(1) → so n^(log₂2) = n với 1 → Case 1, chia thắng → **Θ(n)**. Ứng viên vội thường trả lời "log n vì có chia đôi" — quên rằng có *hai* nhánh: cây sâu log n nhưng có n lá. Đổi một ký tự — `2 * mystery(n/2)` (một lời gọi) — đáp án thành Θ(log n). Sự khác biệt giữa hai đáp án nằm trọn trong hệ số a của recurrence, và người vẽ được cây trong đầu không bao giờ nhầm.

**Lỗi tư duy thường gặp:**

- "Có đệ quy chia đôi là O(log n)" — chỉ đúng khi a = 1. Binary search log n; merge sort n log n; mystery ở trên n — cùng chia đôi, ba đáp án.
- "T(n) = 2T(n−1) chắc cỡ n²" — sai thảm: trừ hằng số với a = 2 là **2ⁿ** (ví dụ chuẩn: tháp Hà Nội T(n) = 2T(n−1) + 1 = 2ⁿ − 1). Nhân nhánh + giảm chậm = mũ.
- Quên chi phí ẩn trong f(n): đệ quy truyền `append([]int{}, arr...)` hay nối chuỗi làm f(n) từ O(1) thành O(n), đổi cả case của Master Theorem.
- Tuyên bố "dùng Master Theorem" với T(n) = T(n−1) + n — dạng này ngoài phạm vi định lý; unroll ra tổng 1+2+...+n mới đúng bài.

**Từ phỏng vấn sang production:** "phân tích hàm đệ quy cho sẵn" chính là code review một PR có đệ quy — cùng bốn câu hỏi, cùng 30 giây, chỉ khác là ở production, đáp án sai không mất offer mà mất uptime.

## 9. Anti-pattern

**Đệ quy trùng lặp không memoization.** Bài toán mở đầu chương. Dấu hiệu nhận biết không cần chạy: hàm đệ quy có ≥ 2 lời gọi con *trên miền tham số chồng lấn* (n−1 và n−2 cùng kéo về những giá trị nhỏ giống nhau). Cây lời gọi có node trùng tên là cây đang gào lên đòi memoization.

**Retry cộng dồn thay vì backoff hình học — hoặc backoff không có nắp.** Retry mỗi 1 giây cố định là chọn nhầm recurrence: hệ thống chết càng lâu, tổng số request đấm vào nó càng tuyến tính theo thời gian. Nhưng chiều ngược lại cũng hỏng: backoff 2ᵏ không có cap thì lần retry thứ 20 chờ 12 ngày. Recurrence hình học + cap + jitter — thiếu mảnh nào cũng là bug đã có tên trong nhiều postmortem.

**Tin rằng "n nhỏ mà, đệ quy kiểu gì chả được".** Hàm mũ biến "n nhỏ" thành khái niệm mong manh: 2ⁿ ở n = 20 là một triệu — vẫn tức thì; n = 45 là ba mươi lăm nghìn tỷ — vài giờ CPU. Khoảng cách giữa "chạy tốt trong test" và "treo trong production" chỉ là hơn gấp đôi giá trị n. Với hàm đa thức, dữ liệu tăng 10 lần thì chi phí tăng lũy thừa cố định; với hàm mũ, chi phí *nhân thêm một hệ số* mỗi khi n **cộng thêm 1**. Đó là khác biệt về loài, không phải về mức độ.

**Đệ quy sâu tuyến tính trên dữ liệu người dùng kiểm soát.** Recurrence còn mô tả *độ sâu stack*: D(n) = D(n−1) + 1 nghĩa là stack sâu n frame. Parse JSON lồng nhau bằng đệ quy, input độc 100.000 tầng ngoặc → stack cạn (Go: goroutine stack tự lớn đến giới hạn ~1GB rồi panic; nhiều ngôn ngữ khác chết sớm hơn nhiều). Vì thế `encoding/json` của Go và mọi parser nghiêm túc đều giới hạn độ sâu. Độ sâu đệ quy là tài nguyên, và recurrence của nó cần được giải giống như recurrence thời gian.

**Copy dữ liệu ở mỗi tầng đệ quy.** Truyền `arr[1:]` trong Go là rẻ (slice header, không copy), nhưng `append` tạo mảng mới hay nối chuỗi ở mỗi tầng biến f(n) từ hằng số thành tuyến tính — recurrence trượt từ Θ(n) sang Θ(n²) trong im lặng. Cùng họ với anti-pattern `s += w` của chương 10, phiên bản đệ quy.

## 10. Best Practices

**Nên:**

- Với mọi hàm đệ quy — của bạn hay trong PR người khác — chạy quy trình bốn câu hỏi ở mục 5 trước khi approve: mấy nhánh, nhỏ đi kiểu gì, việc riêng tốn bao nhiêu, ghép lại ra hình gì. Ba mươi giây, đắt hơn mọi comment về naming.
- Nhìn thấy recurrence dạng "tổng các giá trị trước" hay "nhân nhánh + giảm chậm", mặc định nghi ngờ tăng trưởng mũ cho đến khi chứng minh ngược lại — và kiểm tra xem các bài con có trùng lặp để memoize không.
- Khi thiết kế cơ chế tăng trưởng (buffer, backoff, GC trigger, cấp phát tài nguyên): chọn tăng **theo tỷ lệ** nếu muốn tổng chi phí amortized tuyến tính hoặc muốn áp lực giảm dần — và luôn kèm cap.
- Học Master Theorem *sau khi* thành thạo recursion tree — dùng định lý như phím tắt của trực giác, không phải thay thế trực giác.
- Trong phỏng vấn, viết recurrence ra giấy một cách tường minh trước khi kết luận Big-O — nó vừa chống nhầm, vừa cho interviewer thấy quy trình tư duy, thứ họ chấm điểm cao hơn đáp án.

**Không nên:**

- Không áp Master Theorem cho recurrence dạng T(n−c) hay chia không đều — nhận sai dạng còn tệ hơn không biết định lý.
- Không dùng closed form số thực (kiểu Binet) khi cần giá trị nguyên chính xác — bậc tăng trưởng lấy từ closed form, giá trị lấy từ vòng lặp.
- Không viết đệ quy độ sâu tuyến tính trên input mà người dùng kiểm soát kích thước — chuyển sang vòng lặp, stack tường minh, hoặc chặn độ sâu.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. T(n) = 2T(n/2) + n và T(n) = 2T(n−1) + 1 trông gần giống nhau — vì sao nghiệm là n log n và 2ⁿ, khác nhau một trời một vực? Trả lời bằng hình ảnh cây.
2. Vì sao exponential backoff và Go slice growth là *cùng một recurrence* nhìn từ hai phía, và tính chất toán học nào khiến "tăng theo tỷ lệ" đặc biệt?
3. Hàm đệ quy có hai lời gọi con — khi nào đó là Θ(n) vô hại (duyệt cây), khi nào là 2ⁿ thảm họa (fib naive), và tiêu chí nào phân biệt trong 30 giây?

---

*Chương tiếp theo: [07 — Trees](/series/math-for-engineers/level-2-discrete-mathematics/07-trees/), nơi recurrence hình học N(h) = m·N(h−1) trở thành cấu trúc dữ liệu quan trọng nhất của mọi database: cây — và câu trả lời cho việc vì sao index tra được hàng tỷ dòng trong 3-4 lần đọc đĩa.*
