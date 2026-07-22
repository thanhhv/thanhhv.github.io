+++
title = "Chương 14 — Dynamic Programming: Đừng tính lại thứ đã tính"
date = "2026-07-20T09:20:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 3 – Algorithm Mathematics
> Yêu cầu trước: Chương 06 (Recurrence), Chương 13 (Divide and Conquer)

---

## 1. Problem Statement

Mỗi lần bạn gõ `git diff`, Git phải trả lời một câu hỏi nghe đơn giản: *chuỗi thao tác thêm/xóa ngắn nhất để biến file cũ thành file mới là gì?* Với hai file 1.000 dòng, số cách "căn chỉnh" hai file với nhau nhiều hơn số nguyên tử trong vũ trụ quan sát được. Vậy mà Git trả lời trong vài mili giây — không phải bằng phép màu, mà bằng một quan sát toán học: hàng tỷ tỷ cách căn chỉnh đó được lắp ráp từ một số lượng **rất nhỏ** các bài toán con phân biệt, mỗi bài chỉ cần giải đúng một lần.

Cùng câu chuyện đó lặp lại ở Elasticsearch khi sửa lỗi chính tả cho query, ở router tính đường đi Bellman-Ford, ở công cụ so trình tự DNA. Tên của kỹ thuật — *Dynamic Programming* — không giúp gì cho việc hiểu nó: Richard Bellman thú nhận đã chọn cái tên này thập niên 1950 chủ yếu vì nó nghe... vô hại, để Bộ trưởng Quốc phòng Mỹ (vốn dị ứng với từ "research") không cắt ngân sách của ông. "Programming" ở đây nghĩa là *lập kế hoạch*, không phải viết code.

Câu hỏi chương này trả lời:

**Khi không gian lời giải bùng nổ theo cấp số nhân, làm sao nhận ra — và khai thác — việc nó thực chất chỉ được xây từ đa thức bài toán con?**

Nếu không có DP, mọi bài toán tối ưu tổ hợp chỉ còn hai lựa chọn: duyệt toàn bộ (chết ở n ≈ 25, theo bảng ngưỡng chương 10) hoặc dùng heuristic không có bảo đảm gì. DP là cây cầu giữa "đúng nhưng mũ" và "nhanh nhưng sai".

## 2. Trực giác

### DP không phải kỹ thuật bí ẩn

Hãy phá bỏ hào quang trước: **DP = đệ quy + ghi nhớ**. Hết. Mọi thứ còn lại là kỹ nghệ.

Xét Fibonacci đệ quy thuần — ví dụ nhàm nhưng không gì thay thế được, vì cây đệ quy của nó *nhìn thấy được* sự lãng phí:

```
                        F(6)
                 ┌───────┴───────┐
               F(5)            F(4)
            ┌───┴───┐        ┌──┴──┐
          F(4)    F(3)     F(3)   F(2)
         ┌─┴─┐   ┌─┴─┐    ┌─┴─┐
       F(3) F(2) F(2) F(1) F(2) F(1)
      ┌─┴─┐
    F(2) F(1)      F(3) tính 3 lần, F(2) tính 5 lần...
```

Cây có ~2ⁿ nút, nhưng đếm số bài toán **phân biệt**: chỉ có n+1 giá trị F(0)...F(n). Ta trả chi phí mũ cho lượng thông tin tuyến tính. Sự chênh lệch giữa *kích thước cây đệ quy* và *số bài toán con phân biệt* chính là thước đo mức lãng phí — và là dấu hiệu nhận biết DP số một.

Sửa chữa hiển nhiên đến mức trẻ con nghĩ ra được: **tính rồi thì ghi lại, lần sau tra sổ**. Thêm một cái map, độ phức tạp rơi từ O(2ⁿ) xuống O(n). Không có insight thuật toán mới nào cả — chỉ là từ chối làm lại việc đã làm.

### Vì sao trực giác thường thất bại ở đây

Nếu đơn giản vậy, sao DP nổi tiếng khó? Vì với Fibonacci, bài toán con lộ thiên (tham số n). Với bài toán thật, bài toán con **ẩn**: "biến file A thành file B" phải được nhìn ra là "biến *tiền tố i* của A thành *tiền tố j* của B" thì cấu trúc trùng lặp mới hiện hình. Nghệ thuật của DP không nằm ở cái map ghi nhớ — nó nằm ở việc **định nghĩa bài toán con sao cho chúng ít, chồng lấn, và đủ thông tin**. Đó là mục 4.

## 3. First Principles

DP đứng trên đúng hai chân. Thiếu một trong hai, nó sụp.

### Điều kiện 1 — Overlapping subproblems

Cây đệ quy phải chứa **cùng một bài toán con nhiều lần**. Đây là điểm phân thủy với chương 13: merge sort chia mảng thành hai nửa *không giao nhau* — mỗi bài toán con xuất hiện đúng một lần, memoization không tiết kiệm được gì (cache miss 100%). D&C và DP là hai nhánh của cùng cây quyết định:

```
                  Bài toán chia được thành bài toán con?
                    │
        ┌───────────┴───────────┐
   con ĐỘC LẬP, không lặp    con TRÙNG LẶP
        │                       │
   Divide & Conquer      Dynamic Programming
   (chương 13)           (chương này)
```

### Điều kiện 2 — Optimal substructure

Định nghĩa chính xác: *lời giải tối ưu của bài toán chứa bên trong nó lời giải tối ưu của các bài toán con* — nói cách khác, nếu thay một mảnh của lời giải tối ưu bằng lời giải tốt hơn cho mảnh đó, tổng thể phải tốt lên. Điều này cho phép xây đáp án từ dưới lên mà không sợ "quyết định đúng cục bộ, hỏng toàn cục".

Nghe hiển nhiên? Nó **không** hiển nhiên — và ví dụ phản chứng sau đáng giá hơn mười ví dụ thuận:

**Longest simple path không có optimal substructure.** Shortest path có: nếu đường ngắn nhất A→C đi qua B, thì đoạn A→B của nó phải là đường ngắn nhất A→B (nếu không, thay bằng đường ngắn hơn là cải thiện được toàn cục — mâu thuẫn). Nhưng với đường đi **đơn dài nhất** (không lặp đỉnh), lập luận đó gãy:

```
      A ──── B
      │      │        Đường đơn dài nhất A→C: A–B–D–C (3 cạnh)
      │      │        Đường đơn dài nhất A→B: A–D–C–B... cũng 3 cạnh,
      D ──── C        nhưng nó ĐÃ DÙNG C và D — ghép tiếp B→C là vi phạm "đơn".
```

Vấn đề bản chất: ràng buộc "không lặp đỉnh" khiến các bài toán con **không còn độc lập** — lời giải của mảnh này tiêu thụ tài nguyên (đỉnh) của mảnh kia. Muốn mô tả bài toán con đầy đủ, state phải kèm theo *tập đỉnh đã dùng* — 2ⁿ state, và DP trở về đúng chi phí mũ của duyệt toàn bộ. Đây là lý do longest simple path là NP-hard trong khi shortest path (không chu trình âm) nằm gọn trong P. **Ranh giới giữa hai bài toán trông giống hệt nhau là một tính chất toán học của cấu trúc bài toán con** — không phải độ khéo của người giải.

### Nếu không có khái niệm này thì sao?

Không nhận diện được optimal substructure, bạn không biết bài toán trước mặt thuộc loại "DP giải ngon trong O(n²)" hay loại "NP-hard, đi tìm approximation đi". Không nhận diện được overlapping subproblems, bạn viết đệ quy mũ cho bài toán đa thức (lỗi Fibonacci) hoặc dựng bảng DP vô ích cho bài toán mà D&C/greedy rẻ hơn. Hai điều kiện này là **bộ lọc chẩn đoán**, không phải nghi thức giáo khoa.

## 4. Mathematical Model

### State — nghệ thuật quan trọng nhất

Một bài DP được xác định hoàn toàn bởi định nghĩa **state**: bài toán con được tham số hóa bằng gì? Tiêu chuẩn vàng:

> State là **lượng thông tin tối thiểu về quá khứ đủ để quyết định tối ưu phần còn lại**.

Hai lỗi đối xứng nhau: state *thiếu* thông tin → recurrence không viết nổi hoặc cho đáp án sai (như longest simple path thiếu tập đỉnh đã dùng); state *thừa* thông tin → số state bùng nổ vô ích. Định nghĩa state đúng là 80% lời giải; recurrence sau đó thường tự viết ra chính nó.

Người học Markov chain (chương 11) sẽ thấy quen: state DP chính là yêu cầu tính Markov — *tương lai chỉ phụ thuộc state hiện tại, không phụ thuộc con đường đi đến nó*.

### Quy trình 4 bước

Mọi lời giải DP, từ Climbing Stairs đến sequence alignment trong bioinformatics, đi qua đúng bốn bước:

| Bước | Câu hỏi | Ví dụ: Edit Distance |
|---|---|---|
| 1. State | Bài toán con là gì? Tham số nào? | dp[i][j] = chi phí ít nhất biến a[:i] thành b[:j] |
| 2. Recurrence | State lớn tính từ state nhỏ thế nào? | min của 3 lựa chọn (dưới đây) |
| 3. Base case | State nhỏ nhất có đáp án hiển nhiên? | dp[i][0] = i, dp[0][j] = j |
| 4. Thứ tự tính | Tính state nào trước để phụ thuộc luôn sẵn? | i tăng dần, j tăng dần |

### Edit distance — phân tích kỹ, vì nó là nền của diff

Cho hai chuỗi a, b. Ba thao tác: thêm, xóa, thay một ký tự. Tìm số thao tác ít nhất biến a thành b (khoảng cách Levenshtein).

**Bước 1 — state.** Suy luận từ nguyên lý, không từ trí nhớ: nhìn vào ký tự **cuối** của a và b. Mọi kịch bản biến đổi đều kết thúc bằng một trong số ít khả năng liên quan đến hai ký tự cuối này, và phần còn lại là bài toán *cùng dạng trên hai tiền tố*. Vậy tiền tố là tham số tự nhiên: dp[i][j] = khoảng cách giữa a[:i] và b[:j]. Số state: (m+1)(n+1) — đa thức, trong khi số cách biến đổi là mũ. Sự nén thông tin này chính là DP.

**Bước 2 — recurrence.** Xét a[i−1] và b[j−1] (ký tự cuối của hai tiền tố):

> Nếu a[i−1] == b[j−1]: dp[i][j] = dp[i−1][j−1] — hai ký tự cuối khớp, miễn phí.
> Nếu khác: dp[i][j] = 1 + min( dp[i−1][j],  dp[i][j−1],  dp[i−1][j−1] )
>                          xóa a[i−1]   thêm b[j−1]   thay a[i−1] thành b[j−1]

Ba nhánh là ba quyết định *cạn kiệt mọi khả năng* cho ký tự cuối — đó là cách kiểm tra recurrence đúng: mỗi lời giải tối ưu phải rơi vào đúng một nhánh, và mỗi nhánh quy về bài toán con thực sự nhỏ hơn.

**Bước 3, 4 — base case và thứ tự.** dp[i][0] = i (xóa sạch), dp[0][j] = j (thêm từ đầu). Mỗi ô cần ô trên, trái, chéo trên-trái → duyệt hàng từ trên xuống, trong hàng từ trái sang.

```
        ""  s   i   t
    "" [ 0  1   2   3 ]      a = "kit", b = "sit"
    k  [ 1  1   2   3 ]      dp[3][3] = 1: thay 'k' → 's'
    i  [ 2  2   1   2 ]      Đường truy vết từ góc phải-dưới
    t  [ 3  3   2   1 ]      ngược về gốc = kịch bản diff
```

Chi phí: Θ(mn) thời gian, và lưu ý đường **truy vết** (đi ngược từ dp[m][n] về dp[0][0], mỗi bước hỏi "ô này đến từ nhánh nào?") chính là chuỗi thao tác — đó là output mà `git diff` cần, không phải con số. LCS (longest common subsequence) là anh em song sinh: cùng state, recurrence đổi min thành max và bỏ thao tác thay.

### Top-down vs Bottom-up

Cùng một mô hình toán có hai cách thi công:

| | Top-down (memoization) | Bottom-up (tabulation) |
|---|---|---|
| Cơ chế | Đệ quy tự nhiên + cache | Vòng lặp điền bảng theo thứ tự |
| Thứ tự tính | Tự động theo đệ quy — không cần nghĩ | Phải tự xác định (bước 4) |
| State không cần đến | Bỏ qua (lazy) — lợi khi bảng thưa | Tính hết, kể cả vô dụng |
| Rủi ro | Stack overflow khi độ sâu lớn; overhead gọi hàm | Không |
| Space optimization | Khó — cache giữ mọi thứ | Dễ — giữ 2 hàng, xem dưới |

Lời khuyên thực dụng: **nghĩ bằng top-down, ship bằng bottom-up**. Đệ quy + memo là cách nhanh nhất kiểm chứng state và recurrence đúng; bảng lặp là dạng nhanh, an toàn stack và tối ưu bộ nhớ được.

### Space optimization — hai hàng thay cả bảng

Nhìn lại recurrence edit distance: dp[i][j] chỉ chạm tới hàng i và i−1. Vậy giữ cả bảng m×n làm gì? Hai hàng đủ:

Θ(mn) bộ nhớ → **Θ(min(m, n))**. Với hai file 100k dòng: từ 40GB xuống 400KB — khác biệt giữa "không chạy nổi" và "chạy trên điện thoại". Cái giá phải trả: mất bảng đầy đủ nghĩa là mất truy vết; nếu cần cả kịch bản diff lẫn tiết kiệm bộ nhớ, phải dùng kỹ thuật chia để trị của Hirschberg — một cú bắt tay đẹp giữa chương 13 và chương này.

### Mọi DP là shortest path trên một DAG ngầm

Insight thống nhất, đáng mang theo suốt sự nghiệp: vẽ mỗi state là một đỉnh, mỗi lựa chọn trong recurrence là một cạnh có trọng số đi tới state nhỏ hơn. Vì state chỉ phụ thuộc state "nhỏ hơn", đồ thị này **không có chu trình** — nó là DAG. Khi đó:

> Giải DP = tìm đường ngắn nhất (hoặc dài nhất) từ base case đến state đích trên DAG đó.
> "Thứ tự tính" của bước 4 = một thứ tự topo của DAG (chương 08).
> Truy vết lời giải = đi ngược cạnh trên đường tối ưu.

Góc nhìn này trả lời tại chỗ nhiều câu hỏi lắt léo: *vì sao DP không chạy được khi recurrence có vòng phụ thuộc lẫn nhau?* — DAG có chu trình thì không có thứ tự topo. *Vì sao longest path trên DAG thì DP được còn trên đồ thị thường thì không?* — chính là ví dụ phản chứng ở mục 3. Và nó nối DP vào chương 08: Bellman-Ford, dijkstra trên DAG, critical path trong scheduling — tất cả là một họ.

## 5. Thuật toán

### Mẫu 1D — Climbing Stairs và House Robber

```go
// ClimbStairs: mỗi bước leo 1 hoặc 2 bậc, đếm số cách lên n bậc.
// State: dp[i] = số cách đến bậc i. Recurrence: dp[i] = dp[i-1] + dp[i-2]
// (bước cuối là 1 bậc hoặc 2 bậc — hai nhánh cạn kiệt, không giao nhau).
// Chỉ cần 2 state gần nhất → O(1) bộ nhớ.
func ClimbStairs(n int) int {
    prev, cur := 1, 1 // dp[0], dp[1]
    for i := 2; i <= n; i++ {
        prev, cur = cur, prev+cur
    }
    return cur
}

// HouseRobber: cướp dãy nhà, không được cướp 2 nhà kề nhau, tối đa hóa tiền.
// State: dp[i] = tiền tối đa xét đến nhà i.
// Recurrence: dp[i] = max(dp[i-1], dp[i-2]+v[i]) — bỏ nhà i, hoặc cướp nhà i
// (khi đó nhà i-1 bị cấm, phần còn lại là bài toán đến i-2).
func HouseRobber(v []int) int {
    skip, take := 0, 0 // dp[i-2], dp[i-1] sau mỗi vòng
    for _, x := range v {
        skip, take = take, max(take, skip+x)
    }
    return take
}
```

Hai bài "dễ" này chứa toàn bộ phương pháp: state là tiền tố, recurrence là phân tích cạn kiệt quyết định cuối cùng, và vì chỉ nhìn lại O(1) state, bộ nhớ là O(1).

### Knapsack 0/1 — và vì sao duyệt ngược

Cho n món đồ (trọng lượng wᵢ, giá trị vᵢ), balo chứa tối đa W. Mỗi món lấy hoặc không. State đầy đủ: dp[i][w] = giá trị tối đa dùng *i món đầu* với sức chứa w. Recurrence: dp[i][w] = max(dp[i−1][w], dp[i−1][w−wᵢ] + vᵢ).

Nén xuống một mảng 1D thì xuất hiện chi tiết kinh điển:

```go
// Knapsack01: dp[w] = giá trị tối đa với sức chứa w.
func Knapsack01(w, v []int, W int) int {
    dp := make([]int, W+1)
    for i := range w {
        // BẮT BUỘC duyệt w giảm dần: dp[x-w[i]] phải còn là giá trị
        // của "vòng trước" (chưa xét món i). Duyệt tăng dần thì
        // dp[x-w[i]] đã chứa món i → món i bị lấy 2 lần
        // → vô tình giải bài unbounded knapsack (và Coin Change thì
        // NGƯỢC LẠI: chính vì cho phép lặp nên duyệt tăng dần).
        for x := W; x >= w[i]; x-- {
            dp[x] = max(dp[x], dp[x-w[i]]+v[i])
        }
    }
    return dp[W]
}
```

Chiều duyệt không phải quy ước tùy hứng — nó **mã hóa ràng buộc "mỗi món một lần"** vào thứ tự tính. Hiểu điều này là hiểu bước 4 của quy trình ở mức sâu: thứ tự tính là một phần của ngữ nghĩa, không chỉ là chi tiết thi công.

Ghi chú lý thuyết đáng biết: knapsack chạy O(nW) trông như đa thức, nhưng W là *giá trị* của input, biểu diễn chỉ tốn log W bit — nên đây là **pseudo-polynomial**, và knapsack vẫn NP-hard. Điều đó không cản DP hữu dụng khi W nhỏ; nó chỉ nhắc ta W = 10¹⁸ thì DP bó tay.

### Interval DP — mức giới thiệu

Khi bài toán diễn ra trên một *đoạn* và quyết định là "cắt đoạn ở đâu", state tự nhiên là cặp đầu mút: dp[i][j] = đáp án cho đoạn [i, j], recurrence duyệt điểm cắt k ở giữa: dp[i][j] = min/max theo k của (dp[i][k] ⊕ dp[k][j] ⊕ chi phí ghép). Thứ tự tính: theo **độ dài đoạn tăng dần** — vì đoạn dài phụ thuộc đoạn ngắn. Chi phí điển hình O(n³). Các đại diện: Matrix Chain Multiplication, Burst Balloons (LeetCode 312), và — bất ngờ thú vị — chính PostgreSQL dùng DP dạng này (thuật toán System R) để chọn thứ tự join khi số bảng ≤ `geqo_threshold`: state là *tập con các bảng đã join*, một DP trên 2ⁿ state, chấp nhận được vì n thường ≤ 12.

## 6. Trade-off

**DP vs D&C vs Greedy — chọn theo cấu trúc, không theo mốt.** D&C khi bài toán con rời nhau; DP khi trùng lặp; greedy (chương 15) khi thậm chí không cần xét nhiều lựa chọn mỗi state. Đi từ mạnh về giả định đến yếu: greedy ⊂ DP ⊂ duyệt toàn bộ — mỗi bậc thang đổi tính tổng quát lấy tốc độ.

**Thời gian đổi bộ nhớ — và giới hạn của nó.** DP bản chất là cache: trả O(số state) bộ nhớ để khỏi trả O(mũ) thời gian. Nhưng khi số state tự nó là mũ (TSP bitmask: n·2ⁿ) thì DP chỉ đẩy lùi giới hạn từ n! xuống 2ⁿ — đáng giá (n = 20 khả thi thay vì n = 11, theo bảng chương 10) nhưng không phải phép màu.

**Memoization vs tabulation.** Bảng thưa (nhiều state không bao giờ cần) → top-down thắng vì lazy. Bảng đặc + cần space optimization hoặc hot path → bottom-up. Trong Go còn một lý do riêng: map cho memo chậm hơn slice hàng chục lần do hashing và cache miss; nếu state đánh số được liên tục, luôn dùng slice.

**Độ chính xác đổi tốc độ.** Khi state space quá lớn, người ta *làm thô* state: gộp các giá trị gần nhau, giữ top-k thay vì tất cả. Đó là bước từ DP chính xác sang approximate DP / beam search — nền của nhiều hệ thống ML và của FPTAS cho knapsack. Nhận thức "DP chính xác không khả thi, nhưng cấu trúc DP vẫn dùng được" là một trade-off trưởng thành.

**Khi nào KHÔNG dùng DP?** Khi greedy có chứng minh đúng (activity selection — chương 15) DP là lãng phí một bậc độ phức tạp; khi bài toán con không trùng lặp, memo chỉ tốn RAM vô ích; khi không có optimal substructure, DP cho **đáp án sai** chứ không chỉ chậm — tệ nhất trong ba loại thất bại.

## 7. Production Applications

**Git diff / Myers algorithm.** `git diff` giải đúng bài LCS/edit distance nhưng bằng thuật toán Myers (1986) — một biến thể chạy O((m+n)·d) với d là kích thước diff, thay vì Θ(mn). Cùng DAG ngầm (lưới i×j, cạnh chéo miễn phí khi hai dòng khớp), Myers khai thác việc *diff thực tế thường nhỏ*: lan sóng theo d tăng dần trên DAG — về bản chất là BFS/Dijkstra trên đồ thị state của DP, minh họa sống động cho insight "DP = shortest path trên DAG". Chi tiết ở chương 27.

**Fuzzy search trong Elasticsearch.** Query `fuzziness: AUTO` tìm mọi term cách từ khóa ≤ k phép sửa. Chạy edit distance với *từng* term trong index thì chết; Lucene biên dịch trước bài DP thành **Levenshtein automaton** — một máy trạng thái hữu hạn nhận đúng các chuỗi trong bán kính k — rồi giao với trie của term dictionary. Bài học kiến trúc: khi một bài DP phải chạy hàng triệu lần với một tham số cố định, hãy *precompile* nó thành cấu trúc dữ liệu.

**Bellman-Ford là DP.** State: dp[k][v] = đường ngắn nhất từ nguồn đến v dùng ≤ k cạnh; recurrence: relax qua từng cạnh; k chạy đến n−1. Đây là DP giáo khoa trên đồ thị có thể có chu trình — trick là đưa "số cạnh đã dùng" vào state để ép ra DAG. Distance-vector routing (RIP) là Bellman-Ford phân tán: mỗi router giữ một hàng của bảng DP và trao đổi với hàng xóm.

**Sequence alignment trong bioinformatics.** Needleman–Wunsch (align toàn cục) và Smith–Waterman (align cục bộ) là edit distance với ma trận chi phí sinh học (thay thế acid amin này bằng acid kia rẻ hay đắt tùy hóa tính) và gap penalty. Toàn bộ ngành so sánh genome đứng trên đúng cái bảng dp[i][j] của mục 4 — cộng thêm heuristic (BLAST) để tránh điền cả bảng cho chuỗi hàng tỷ ký tự.

**Kubernetes bin-packing.** Xếp pod (CPU/memory request) vào node là bài bin packing — NP-hard, họ hàng trực hệ của knapsack. Scheduler của Kubernetes *không* chạy DP (không kịp ở quy mô nghìn node); nó dùng heuristic chấm điểm greedy như `MostAllocated` (dồn cho chặt — tiết kiệm node) hay `LeastAllocated` (trải cho đều). Hiểu gốc knapsack cho bạn hai điều: biết vì sao kết quả có thể suboptimal và cần descheduler dọn lại, và biết đọc các tham số scoring như một *hàm giá trị knapsack* được đơn giản hóa.

## 8. Interview

DP là "trùm cuối" của phỏng vấn thuật toán — không vì nó khó nhất, mà vì nó phơi bày rõ nhất ai học cách tư duy và ai học thuộc 50 lời giải.

**Climbing Stairs (LeetCode 70)** — bài chào sân. Giá trị thật của nó: nhận ra đây là Fibonacci *mà không cần ai bảo*, bằng cách tự phân tích quyết định cuối cùng.

**Coin Change (LeetCode 322)** — dp[x] = số đồng ít nhất tạo tổng x; recurrence min qua mọi mệnh giá. Bài này quan trọng vì là **bằng chứng buộc tội greedy**: với hệ xu {1, 3, 4} và tổng 6, greedy cho 4+1+1 nhưng DP tìm ra 3+3 — cứ liệu chính cho mục "nghi ngờ greedy" của chương 15.

**Longest Increasing Subsequence (LeetCode 300)** — bài học hai tầng. Tầng 1: state dp[i] = độ dài LIS *kết thúc tại i*, recurrence O(n²). Tầng 2 — nơi ứng viên giỏi tách tốp: đổi state! Giữ mảng `tails[k]` = phần tử kết thúc **nhỏ nhất** của một increasing subsequence độ dài k+1; mảng này luôn sorted (chứng minh bằng exchange nhỏ), nên cập nhật bằng binary search → O(n log n). Bài học lớn: *một bài toán có nhiều cách định nghĩa state, và định nghĩa khôn hơn có thể đổi cả bậc độ phức tạp*.

**Edit Distance (LeetCode 72)** — chính là mục 4. Trong phỏng vấn, điểm ăn tiền không phải điền bảng đúng mà là *dẫn dắt được* vì sao state là cặp tiền tố và vì sao ba nhánh cạn kiệt mọi khả năng.

**House Robber (LeetCode 198)** — kiểm tra việc đưa *ràng buộc* (không kề nhau) vào recurrence thay vì vào state. Biến thể vòng tròn (213) kiểm tra tiếp: xử lý ràng buộc toàn cục bằng cách chạy DP hai lần với hai giả định.

**Lỗi tư duy thường gặp:**

- **Học thuộc bảng thay vì học định nghĩa state.** Thuộc lời giải Edit Distance nhưng gặp "Delete Operation for Two Strings" (583 — chỉ cho xóa) là gãy, dù chỉ cần bỏ một nhánh của recurrence. Antidote: với mỗi bài đã giải, tự trả lời "state là gì, *vì sao* nó đủ?" thay vì nhớ hình cái bảng.
- **Nhảy vào code trước khi phát biểu state bằng một câu tiếng Việt hoàn chỉnh.** Nếu không viết được câu "dp[i][j] là ...", bạn chưa có lời giải — chỉ có hy vọng.
- **Lạm dụng DP khi greedy đủ.** Non-overlapping Intervals giải được bằng DP O(n²) nhưng greedy O(n log n) đúng và có chứng minh — trình bày DP ở đây là *trừ điểm* về judgment. Quy tắc: trước khi dựng bảng, dành 60 giây hỏi "có lựa chọn cục bộ nào luôn an toàn không?" (chương 15).
- **Quên rằng người ta thường hỏi truy vết**: "in ra phương án" đòi lưu quyết định hoặc đi ngược bảng — nếu đã space-optimize còn 2 hàng thì mất khả năng này, một trade-off phải nêu chủ động.

**Cách phân tích thay vì học thuộc:** (1) mô tả lời giải như *chuỗi quyết định*; (2) hỏi "quyết định cuối cùng có những khả năng nào?" — mỗi khả năng phải quy về bài toán cùng dạng nhỏ hơn; (3) hỏi "để phần còn lại tự quyết được, nó cần biết gì về những quyết định đã qua?" — đó là state; (4) đếm số state × chi phí mỗi state, so với ràng buộc đề bài; nếu vượt, tìm cách nén state (như LIS tầng 2).

## 9. Anti-pattern

**DP như câu thần chú.** "Bài này chắc DP" rồi dựng bảng 2D trước khi kiểm tra hai điều kiện. Nếu bài toán con không trùng lặp, bạn vừa viết một D&C tốn RAM; nếu không có optimal substructure, bạn vừa viết một chương trình cho đáp án sai với vẻ ngoài tự tin. Luôn chẩn đoán trước, dựng bảng sau.

**State thiếu — bug đáp-án-sai khó lường nhất.** Ví dụ thật: tính chi phí rẻ nhất của chuỗi giao dịch có phí chuyển đổi trạng thái (đang giữ cổ phiếu hay không), nhưng state chỉ có ngày mà thiếu cờ "đang giữ" → recurrence buộc phải "nhìn trộm" quá khứ → sai. Triệu chứng: recurrence viết mãi không gọn, đầy case đặc biệt. Thuốc: quay lại câu hỏi "phần còn lại cần biết *gì* về quá khứ?"

**Bảng đầy đủ khi 2 hàng là đủ — hoặc ngược lại.** Giữ bảng m×n cho hai chuỗi dài rồi OOM trong production; hoặc space-optimize xong mới phát hiện cần truy vết phương án. Quyết định bộ nhớ phải đi *sau* quyết định output.

**Memoization với key đắt.** Dùng `map[string]int` với key là `fmt.Sprintf("%d-%d", i, j)` — mỗi lần tra memo tốn một lần cấp phát chuỗi + hash, hằng số nuốt luôn lợi ích. State số nguyên → slice phẳng `dp[i*n+j]`; struct nhỏ → `map[[2]int]int` cũng đã tốt hơn chuỗi nhiều lần.

**Đệ quy sâu không giới hạn.** Top-down trên chuỗi 10⁶ phần tử với recurrence dp[i] → dp[i−1] tạo chuỗi gọi sâu 10⁶ — goroutine stack của Go giãn được nhưng vẫn có trần, và mỗi frame tốn thật. Độ sâu tuyến tính với n lớn → chuyển bottom-up, vốn chỉ là một vòng for.

## 10. Best Practices

**Nên:**

- Bắt đầu mọi bài bằng **câu văn định nghĩa state** hoàn chỉnh, viết ra được thành lời — đây là deliverable số một, code chỉ là phiên dịch.
- Kiểm chứng recurrence trên ví dụ 3–5 phần tử bằng tay trước khi code; base case kiểm tra riêng (chuỗi rỗng, một phần tử).
- Prototype top-down để xác nhận đúng, chuyển bottom-up khi cần hiệu năng/bộ nhớ; space-optimize là bước *cuối*, chỉ sau khi chốt có cần truy vết không.
- Ước lượng số state × chi phí chuyển trước khi code, đối chiếu bảng ngưỡng chương 10 — n ≤ 20 gợi ý state bitmask 2ⁿ; n ≈ 10³ cho phép bảng n²; n ≈ 10⁵ đòi state 1D với chuyển O(log n).
- Trong Go: state đánh số liên tục → slice thay map; khởi tạo giá trị "chưa tính" bằng sentinel rõ ràng (−1) thay vì zero value dễ nhầm với đáp án thật.

**Không nên:**

- Không dựng bảng khi chưa trả lời được hai câu: bài toán con có trùng lặp không, optimal substructure có đứng vững không (thử tấn công nó như longest simple path).
- Không dùng DP khi greedy có chứng minh — và không dùng greedy khi chưa có (chương 15 nói tiếp).
- Không tin "O(nW) là đa thức" — pseudo-polynomial sẽ phản bội bạn khi W là số tiền tính bằng đồng.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Vì sao shortest path có optimal substructure còn longest simple path thì không — và ràng buộc nào chính xác đã phá vỡ tính độc lập của các bài toán con?
2. Trong knapsack 0/1 nén xuống mảng 1D, điều gì sai về mặt *ngữ nghĩa* nếu duyệt sức chứa tăng dần — và bài toán nào vô tình được giải thay?
3. "Mọi DP là shortest path trên một DAG ngầm" — với Edit Distance, đỉnh, cạnh và trọng số của DAG đó là gì?

---

*Chương tiếp theo: [15 — Greedy Algorithms](/series/math-for-engineers/level-3-algorithm-mathematics/15-greedy/): DP xét mọi lựa chọn tại mỗi state — nhưng đôi khi có một lựa chọn luôn an toàn, và nếu chứng minh được điều đó, ta vứt cả bảng DP đi. Khi nào tham lam là đúng?*
