+++
title = "Chương 05 — Counting & Combinatorics: Đếm mà không cần liệt kê"
date = "2026-07-20T07:50:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 2 – Discrete Mathematics
> Yêu cầu trước: Chương 01 (Logic), Chương 02 (Set Theory, Functions & Relations)

---

## 1. Problem Statement

Một team platform chuẩn bị release tính năng mới đứng sau 12 feature flags. QA lead hỏi câu tưởng chừng đơn giản: "Chúng ta có cần test mọi tổ hợp bật/tắt không?" Một kỹ sư mở laptop định viết script liệt kê tất cả các tổ hợp ra file để đếm. Script chạy được vài giây thì anh dừng lại và làm phép tính nhẩm: mỗi flag có 2 trạng thái, 12 flags → 2¹² = 4.096 tổ hợp. Nếu thêm 3 môi trường deploy và 4 tier khách hàng, con số thành 4.096 × 3 × 4 = **49.152 kịch bản test**. Với 5 phút mỗi kịch bản, đó là 6 tháng làm việc liên tục.

Điều đáng chú ý không phải là con số, mà là **cách có được con số**: không cần liệt kê một kịch bản nào. Chỉ cần nhân vài số với nhau, team biết ngay "test hết mọi tổ hợp" là bất khả thi và phải chuyển sang chiến lược khác (pairwise testing — sẽ quay lại ở mục 7).

Đây chính là bài toán của Combinatorics:

**Làm sao biết một tập hợp có bao nhiêu phần tử, mà không cần — và thường là không thể — liệt kê chúng?**

Nếu không có công cụ này, kỹ sư mất khả năng trả lời hàng loạt câu hỏi thiết kế: brute force bài này có chạy nổi không? Hash table với chừng này bucket thì collision có tránh được không? Bao nhiêu cách gán replica lên broker? State space của hệ thống lớn cỡ nào — tức là có bao nhiêu "trạng thái có thể xảy ra" mà ta phải reasoning về chúng? Tất cả đều là câu hỏi **đếm**, và tất cả đều có tập cần đếm lớn đến mức việc liệt kê là vô vọng.

## 2. Trực giác

### Hai phép toán nền tảng: "hoặc" thì cộng, "rồi" thì nhân

Toàn bộ combinatorics đứng trên hai quy tắc mà bạn đã dùng từ tiểu học mà không gọi tên:

**Rule of Sum** — nếu một việc có thể làm theo cách A *hoặc* cách B, và hai nhóm cách không trùng nhau, thì tổng số cách là |A| + |B|. Đi từ nhà đến công ty bằng 3 tuyến bus hoặc 2 tuyến metro → 5 lựa chọn.

**Rule of Product** — nếu một việc gồm bước 1 *rồi đến* bước 2, và số lựa chọn ở bước 2 không phụ thuộc vào việc bước 1 chọn gì, thì tổng số cách là |bước 1| × |bước 2|. Mỗi flag có 2 trạng thái, 12 flags độc lập → 2¹².

Nghe tầm thường, nhưng để ý điều kiện in nghiêng: *không trùng nhau* và *không phụ thuộc*. Hầu hết lỗi đếm trong thực tế — từ ước lượng sai selectivity trong database đến ước lượng sai effort test — đều do áp dụng hai quy tắc này khi điều kiện của chúng không thỏa. Cộng hai tập giao nhau thì đếm trùng; nhân hai lựa chọn phụ thuộc nhau thì đếm khống. Toàn bộ phần còn lại của chương là công cụ xử lý khi điều kiện đẹp đẽ đó bị vi phạm.

### Đếm là ánh xạ, không phải liệt kê

Trực giác quan trọng thứ hai: đếm một tập khó thực chất là **tìm một song ánh (bijection — chương 02) sang một tập dễ**. Ta không đếm 49.152 kịch bản test; ta nhận ra mỗi kịch bản tương ứng 1-1 với một bộ (12 bit, môi trường, tier), và tập các bộ đó thì đếm được bằng rule of product. Kỹ năng combinatorics không phải là nhớ công thức — mà là nhìn ra "tập này thực chất là tập kia mặc áo khác".

### Khi nào trực giác thất bại?

Trực giác con người ước lượng số tổ hợp **tệ một cách hệ thống** — luôn đoán thấp. Bao nhiêu cách xáo trộn một bộ bài 52 lá? Đáp án là 52! ≈ 8 × 10⁶⁷ — lớn hơn số nguyên tử của Trái Đất. Mỗi lần bạn xáo bài tử tế, gần như chắc chắn tạo ra một thứ tự **chưa từng tồn tại trong lịch sử loài người**. Không ai đoán được điều đó bằng cảm giác. Đây cũng chính là lý do brute force chết nhanh hơn lập trình viên tưởng: state space nổ theo giai thừa và lũy thừa, còn trực giác thì nội suy tuyến tính.

## 3. First Principles

### Xây mọi thứ từ rule of product

Hãy dẫn xuất các công thức thay vì chấp nhận chúng — vì cách dẫn xuất mới là thứ dùng được khi gặp bài toán lạ.

**Permutation — chọn k phần tử có thứ tự từ n phần tử.** Tưởng tượng xếp k người vào k ghế đánh số. Ghế 1 có n lựa chọn, ghế 2 còn n−1 (một người đã ngồi), ... ghế k còn n−k+1. Rule of product:

> P(n, k) = n × (n−1) × ... × (n−k+1) = n! / (n−k)!

**Combination — chọn k phần tử không quan tâm thứ tự.** Đây là chỗ đáng dừng lại lâu nhất. C(n, k) không phải là một công thức mới — nó là P(n, k) **sau khi sửa lỗi đếm trùng**. Khi đếm P(n, k), mỗi *nhóm* k người bị đếm đi đếm lại một lần cho mỗi cách xếp thứ tự nội bộ của nhóm — tức k! lần. Muốn đếm nhóm (không thứ tự), ta chia cho số lần đếm trùng:

> C(n, k) = P(n, k) / k! = n! / (k!(n−k)!)

Phép chia k! không phải là chi tiết kỹ thuật — nó là **nguyên lý tổng quát**: *khi mỗi đối tượng cần đếm bị đếm trùng đúng m lần, hãy đếm bản dễ rồi chia m*. Nguyên lý này giải được vô số bài không có công thức sẵn: đếm chuỗi vòng (chia cho số phép xoay), đếm cách chia đội không đánh số (chia cho số hoán vị của các đội).

### Lập luận combinatorial: chứng minh bằng cách kể chuyện đếm

Đẳng thức C(n, k) = C(n−1, k−1) + C(n−1, k) có thể chứng minh bằng biến đổi đại số nhàm chán. Hoặc bằng một câu chuyện: chọn k người từ n người, hãy nhìn vào người cuối cùng — **hoặc** cô ấy được chọn (còn phải chọn k−1 từ n−1 người), **hoặc** không (chọn đủ k từ n−1 người). Rule of sum cho ngay đẳng thức. Không một dòng đại số.

Kiểu lập luận này — gọi là **combinatorial argument** hay "double counting" — đáng giá hơn công thức vì hai lý do. Thứ nhất, nó cho hiểu biết thay vì ký hiệu. Thứ hai, nó chính là **cấu trúc của lời giải Dynamic Programming**: "hoặc chọn phần tử này, hoặc không" là câu thần chú của Knapsack, Subsets, và hàng chục bài DP khác (chương 14). Ai quen lập luận combinatorial thì nhìn bài DP ra ngay recurrence.

### Nếu không có các khái niệm này thì sao?

Mọi câu hỏi về kích thước state space chỉ còn trả lời được bằng cách... chạy thử và đợi. "Brute force có nổi không" trở thành câu hỏi thực nghiệm tốn hàng giờ thay vì phép nhân 5 giây. Và những khẳng định dạng "chắc chắn xảy ra" (mục Pigeonhole dưới đây) hoàn toàn không thể chứng minh bằng thực nghiệm — bạn không thể thử hết mọi trường hợp để biết collision là *tất yếu*.

## 4. Mathematical Model

### Pigeonhole Principle: nguyên lý một dòng, hệ quả khắp hệ thống

> Nếu đặt n vật vào m hộp với n > m, thì **ít nhất một hộp chứa từ 2 vật trở lên**. Dạng tổng quát: có ít nhất một hộp chứa **≥ ⌈n/m⌉** vật.

Chứng minh bằng phản chứng (chương 03): nếu mọi hộp chứa ≤ 1 vật thì tổng ≤ m < n — mâu thuẫn. Tầm thường về mặt toán, nhưng sức mạnh nằm ở chữ "**chắc chắn**": không phải "có thể", không phải "xác suất cao" — mà là *không tồn tại cách nào tránh được*. Ba hệ quả trực tiếp cho kỹ sư:

**Hash collision là tất yếu, không phải rủi ro.** Hash function ánh xạ không gian key (ví dụ mọi chuỗi UTF-8 — vô hạn) vào không gian giá trị hữu hạn (2⁶⁴ với hash 64-bit, hay số bucket của hash table). Số key > số bucket → theo pigeonhole, **tồn tại** hai key khác nhau cùng bucket. Vì vậy mọi hash table đều *phải* có cơ chế xử lý collision (chaining, open addressing — chương 12); "chọn hash function tốt để không bao giờ collision" là mục tiêu bất khả thi về mặt toán học, không phải về mặt kỹ thuật. Hash function tốt chỉ làm collision *hiếm và đều*, không làm nó biến mất.

**Trong 367 người, chắc chắn có 2 người trùng ngày sinh.** Có tối đa 366 ngày sinh khả dĩ (kể cả 29/2); 367 người vào 366 "hộp" → có hộp chứa ≥ 2. Lưu ý sự tương phản với **birthday paradox** (chương 12): để trùng *chắc chắn* cần 367 người, nhưng để trùng *với xác suất > 50%* chỉ cần 23. Pigeonhole cho ngưỡng chắc chắn; xác suất cho ngưỡng thực dụng — hai công cụ khác nhau cho hai câu hỏi khác nhau.

**Load balancing: luôn có server nhận ≥ trung bình.** Phân phối n request vào m server, theo dạng tổng quát của pigeonhole, ít nhất một server nhận ≥ ⌈n/m⌉ request. Nghe hiển nhiên, nhưng hệ quả thiết kế thì không: **capacity planning không được phép dựa trên tải trung bình**. "Mỗi server chịu trung bình 1.000 RPS" không cứu bạn khi một server nhận 3.000 — và pigeonhole bảo đảm luôn có server nhận ít nhất mức trung bình, còn thực tế (hot key, hash không đều) đẩy con số vượt xa trung bình. Đây là nền của bài toán "power of two choices" và consistent hashing (chương 25).

### Inclusion-Exclusion: sửa lỗi đếm trùng khi các tập giao nhau

Rule of sum đòi hỏi các tập rời nhau. Khi chúng giao nhau, cộng thô sẽ đếm phần giao hai lần. Nguyên lý bù trừ sửa lại:

> |A ∪ B| = |A| + |B| − |A ∩ B|
> |A ∪ B ∪ C| = |A| + |B| + |C| − |A∩B| − |A∩C| − |B∩C| + |A∩B∩C|

Trực giác: cộng hết (phần giao bị đếm 2 lần) → trừ các phần giao đôi (phần giao ba giờ bị trừ quá tay) → cộng lại phần giao ba. Cứ thế đan xen cộng-trừ với nhiều tập hơn.

```
   ┌───────────┐
   │ A         │        Đếm |A| + |B|:
   │      ┌────┼──────┐  vùng giữa (A∩B)
   │      │ ▒▒ │      │  bị đếm 2 lần
   │      │ ▒▒ │  B   │  → trừ đi 1 lần
   └──────┼────┘      │
          └───────────┘
```

Với kỹ sư backend, đây chính là bài toán "đếm bản ghi thỏa **ít nhất một** điều kiện": `WHERE status = 'failed' OR retries > 3`. Query planner không thể cộng thô số dòng thỏa từng điều kiện — có những dòng thỏa cả hai. PostgreSQL ước lượng selectivity của mệnh đề OR chính xác theo công thức bù trừ: `sel(A OR B) = sel(A) + sel(B) − sel(A)·sel(B)`, trong đó phần giao được *xấp xỉ* bằng tích vì planner giả định hai điều kiện độc lập (giả định này khi sai sẽ gây thảm họa — xem mục 7).

### Binomial theorem: vì sao C(n, k) xuất hiện khắp nơi

> (x + y)ⁿ = Σₖ C(n, k) · xᵏ · yⁿ⁻ᵏ

Không cần khai triển đại số — hãy nhìn bằng mắt đếm. (x+y)ⁿ là tích của n cặp ngoặc; khai triển nghĩa là **từ mỗi ngoặc chọn x hoặc y** rồi nhân lại. Số cách tạo ra số hạng xᵏyⁿ⁻ᵏ = số cách chọn ra k ngoặc đóng góp x = C(n, k). Định lý nhị thức chỉ là rule of product + combination kể lại bằng ký hiệu đa thức.

Hệ quả đáng nhớ nhất: đặt x = y = 1 được **Σₖ C(n, k) = 2ⁿ** — tổng số tập con của n phần tử. Hai cách đếm cùng một thứ: đếm theo kích thước tập con (vế trái), hay đếm theo chuỗi n quyết định chọn/không (vế phải). Và hình dạng của dãy C(n, k) — phình to ở giữa, C(n, n/2) ≈ 2ⁿ/√n — chính là lý do phân phối nhị thức dồn về trung tâm, nền tảng cho các ước lượng xác suất ở chương 11.

### Đếm state space: công cụ ước lượng feasibility

Ghép tất cả lại thành bảng tra dùng hằng ngày:

| Câu hỏi | Cấu trúc đếm | Kích thước |
|---|---|---|
| Tổ hợp của n feature flags | mỗi flag chọn/không | 2ⁿ |
| Tập con của n phần tử | như trên | 2ⁿ |
| Thứ tự xử lý n task | hoán vị | n! |
| Chọn k server từ n | combination | C(n, k) |
| Chuỗi dài k trên bảng chữ cái m ký tự | product | mᵏ |
| Gán n request vào m server | mỗi request chọn server | mⁿ |

Kết hợp với bảng ngưỡng "10⁸ phép toán/giây" ở chương 10: n! vượt 10⁸ ngay tại n = 12; 2ⁿ tại n = 27. Đọc ràng buộc đề bài hay spec hệ thống, nhân vài con số, và bạn biết ngay biên giới giữa "duyệt hết được" và "phải thông minh hơn".

## 5. Thuật toán

Tính C(n, k) trong code có một cái bẫy kinh điển: `n! / (k! * (n-k)!)` tràn số ngay lập tức — 21! đã vượt int64. Hai cách đúng:

```go
// Cách 1: Tính nhân dần, chia sớm — C(n,k) = C(n,k-1) * (n-k+1) / k
// Kết quả trung gian luôn là một hệ số nhị thức, nên phép chia luôn chẵn
// và giá trị trung gian không phình to hơn kết quả cuối.
func binomial(n, k int) int {
    if k < 0 || k > n {
        return 0
    }
    if k > n-k {
        k = n - k // tận dụng đối xứng C(n,k) = C(n,n-k), giảm số vòng lặp
    }
    result := 1
    for i := 1; i <= k; i++ {
        result = result * (n - k + i) / i
    }
    return result
}

// Cách 2: Tam giác Pascal — dùng khi cần NHIỀU giá trị C(n,k),
// hoặc khi tính theo modulo (phép chia không còn dùng được trực tiếp).
// Chính là đẳng thức C(n,k) = C(n-1,k-1) + C(n-1,k) đã chứng minh
// bằng lập luận combinatorial ở mục 3 — mã hóa thành DP.
func pascalTriangle(n int) [][]int {
    c := make([][]int, n+1)
    for i := 0; i <= n; i++ {
        c[i] = make([]int, i+1)
        c[i][0], c[i][i] = 1, 1
        for j := 1; j < i; j++ {
            c[i][j] = c[i-1][j-1] + c[i-1][j] // "chọn hay không chọn phần tử cuối"
        }
    }
    return c
}
```

Điểm đáng suy ngẫm: tam giác Pascal là ví dụ đầu tiên trong tài liệu này về việc **một chứng minh combinatorial trở thành một thuật toán DP**. Đẳng thức đếm là recurrence; bảng DP là cách điền recurrence. Mối liên hệ này sẽ trở thành chủ đề trung tâm ở chương 14.

Còn khi cần *liệt kê* (không chỉ đếm) — sinh mọi tập con bằng bitmask:

```go
// Duyệt mọi tập con của n phần tử — chỉ khả thi khi n ≤ ~25 (2^25 ≈ 3×10^7).
// Chính phép đếm 2^n cho ta biết TRƯỚC KHI VIẾT CODE là hàm này chạy nổi hay không.
func enumerateSubsets(items []string, visit func(subset []string)) {
    n := len(items)
    for mask := 0; mask < 1<<n; mask++ {
        var subset []string
        for i := 0; i < n; i++ {
            if mask&(1<<i) != 0 { // bit i bật = phần tử i được chọn
                subset = append(subset, items[i])
            }
        }
        visit(subset)
    }
}
```

Song ánh {tập con} ↔ {số nguyên 0..2ⁿ−1} không chỉ để đếm — nó là *cách biểu diễn*: mỗi tập con nén thành một int, so sánh và lưu trữ bằng phép toán bit. Đếm và mã hóa là hai mặt của cùng một song ánh.

## 6. Trade-off

**Đếm chính xác vs ước lượng.** Công thức đóng cho kết quả chính xác nhưng chỉ tồn tại khi cấu trúc bài toán "đẹp" (độc lập, đối xứng). Thực tế thường xuyên không đẹp: đếm số dòng thỏa `WHERE` phức tạp trên dữ liệu tương quan không có công thức đóng. Khi đó hệ thống thật chuyển sang **ước lượng bằng mẫu** (sampling, histogram, sketch — chương 18, 23), chấp nhận sai số để đổi lấy tính khả thi. Combinatorics cho biên chính xác; thống kê cho con số gần đúng nhanh — kỹ sư giỏi biết mình đang cần cái nào.

**Công thức vs DP vs liệt kê.** Cùng tính C(n, k): công thức nhân O(k) nhưng cần cẩn thận overflow; tam giác Pascal O(n²) bộ nhớ nhưng an toàn với modulo; liệt kê O(2ⁿ) nhưng là lựa chọn duy nhất khi cần *xem* từng cấu hình chứ không chỉ đếm. Không có lựa chọn đúng tuyệt đối — có lựa chọn đúng cho từng câu hỏi.

**Khi nào KHÔNG dùng rule of product?** Khi các lựa chọn phụ thuộc nhau. Số cách cấu hình 12 flags là 2¹² *chỉ khi* mọi tổ hợp đều hợp lệ; nếu flag A bật đòi hỏi flag B bật, con số thật nhỏ hơn và phải đếm bằng cấu trúc khác (thường là DP trên đồ thị ràng buộc). Nhân bừa các lựa chọn phụ thuộc cho ước lượng **quá cao**; ngược lại, giả định độc lập trong ước lượng xác suất thường cho kết quả **quá thấp** ở phần giao. Câu hỏi đầu tiên trước mọi phép nhân: *các lựa chọn này có thật sự độc lập không?*

**Pigeonhole cho biết tồn tại, không cho biết ở đâu.** Nguyên lý bảo đảm có hai key trùng bucket nhưng không chỉ ra cặp nào; bảo đảm có server quá tải nhưng không nói server nào. Nó là công cụ **thiết kế** (buộc bạn chuẩn bị cơ chế xử lý) chứ không phải công cụ **vận hành** (không thay được monitoring).

## 7. Production Applications

**PostgreSQL selectivity estimation** — combinatorics và giả định độc lập chạy trong mọi query plan. Với `WHERE city = 'Hanoi' AND age > 30`, planner ước lượng selectivity từng điều kiện từ histogram rồi **nhân với nhau** — đúng rule of product trên xác suất, hợp lệ chỉ khi hai cột độc lập. Khi dữ liệu tương quan (`city = 'Hanoi' AND country = 'VN'` — điều kiện sau gần như bao hàm điều kiện trước), phép nhân cho ước lượng thấp hơn thực tế hàng nghìn lần, planner chọn nested loop join thay vì hash join, và query chạy chậm 100 lần. Đây là một trong những nguyên nhân phổ biến nhất của "query đột nhiên chậm" trong thực tế, và là lý do PostgreSQL 10+ thêm `CREATE STATISTICS` — cho phép khai báo "hai cột này phụ thuộc nhau, đừng nhân bừa". Với mệnh đề `OR`, planner áp dụng đúng công thức inclusion-exclusion như mục 4. Hiểu đếm trùng và tính độc lập nghĩa là đọc được `EXPLAIN` sâu hơn 90% người dùng.

**Kafka partition và replica assignment.** Một topic 100 partition, mỗi partition 3 replica, đặt lên 10 broker: mỗi partition có C(10, 3) = 120 cách chọn bộ broker chứa replica. Không gian gán toàn cục là 120¹⁰⁰ — vô vọng cho tìm kiếm tối ưu toàn cục, và đó là lý do Kafka dùng thuật toán gán round-robin có xáo trộn thay vì "tìm phương án tối ưu". Đồng thời, pigeonhole cho một khẳng định cứng: 100 partition lên 10 broker thì có broker gánh ≥ 10 partition — nếu partition có hot key, sự chênh lệch tải là chắc chắn có, chỉ chưa biết ở đâu. Chọn số partition chia hết cho số broker và chọn partition key phân bố đều — cả hai đều là quyết định combinatorial.

**Ước lượng số test case và pairwise testing.** Quay lại câu chuyện mở đầu: 49.152 tổ hợp không test hết được. Lối thoát cũng đến từ combinatorics: thống kê thực nghiệm cho thấy phần lớn bug do *tương tác của tối đa 2 tham số*. Thay vì phủ mọi tổ hợp đầy đủ, chỉ cần bộ test phủ **mọi cặp giá trị của mọi cặp tham số** — bài toán covering design kinh điển, giảm từ hàng chục nghìn xuống vài chục test case. Công cụ như `pict` (Microsoft) làm đúng việc này. Không biết đếm thì không biết mình đang bỏ sót gì và tiết kiệm được gì.

**Cardinality và sizing khắp nơi:** số shard × số replica trong Elasticsearch quyết định số shard vật lý phải phân bổ (và pigeonhole lại xuất hiện: nhiều shard hơn node thì có node gánh nhiều shard); số tổ hợp label trong Prometheus là tích số giá trị mỗi label — lý do "label có cardinality cao" làm nổ tung bộ nhớ TSDB: thêm một label `user_id` với 10⁶ giá trị nghĩa là *nhân* số time series lên 10⁶ lần, đúng theo rule of product.

## 8. Interview

Các bài đếm trong phỏng vấn kiểm tra đúng một kỹ năng: **nhận diện "bài này đang đếm cái gì"** trước khi viết bất cứ dòng code nào.

**Unique Paths (LeetCode 62).** Robot đi từ góc trên-trái đến góc dưới-phải lưới m×n, chỉ đi phải hoặc xuống. Cách nhìn combinatorial: mọi đường đi đều gồm đúng (m−1) bước xuống và (n−1) bước phải — tổng m+n−2 bước, và một đường đi được xác định *hoàn toàn* bởi việc chọn vị trí các bước xuống trong chuỗi. Đó là song ánh sang bài chọn: đáp án = C(m+n−2, m−1), tính được trong O(m+n) bằng hàm `binomial` ở mục 5. Lời giải DP O(m·n) cũng đúng — và chính bảng DP đó *là* tam giác Pascal bị bóp méo. Nói được cả hai cách và mối liên hệ giữa chúng là điểm khác biệt giữa "giải được" và "hiểu".

**Subsets (LeetCode 78) và Permutations (LeetCode 46).** Trước khi code, hãy đếm output: 2ⁿ tập con, n! hoán vị. Phép đếm này trả lời ngay câu hỏi interviewer chắc chắn sẽ hỏi — "độ phức tạp bao nhiêu?": không lời giải nào tốt hơn O(2ⁿ) / O(n!·n), vì **riêng việc ghi output đã tốn chừng đó**. Đây là một dạng lower bound argument mà rất nhiều ứng viên bỏ lỡ: độ phức tạp bị chặn dưới bởi kích thước output, và kích thước output là một bài toán đếm.

**Lỗi tư duy thường gặp:**

- Không phân biệt "có thứ tự hay không" — câu hỏi đầu tiên phải tự hỏi với mọi bài đếm. Chọn 3 người vào một *nhóm* là C(n,3); vào ba *vai trò khác nhau* là P(n,3).
- Đếm trùng mà không biết: sinh tổ hợp bằng đệ quy nhưng cho phép chọn lại phần tử đứng trước → mỗi tập con xuất hiện k! lần. Fix chuẩn (chỉ chọn phần tử có index lớn hơn phần tử vừa chọn) chính là cách *ép một thứ tự chuẩn hóa* để mỗi tập được sinh đúng một lần — tư duy "chia cho số lần đếm trùng" thể hiện trong code.
- Áp công thức mà không kiểm tra điều kiện: Unique Paths có chướng ngại vật (LeetCode 63) thì song ánh sang C(m+n−2, m−1) sụp đổ — phải quay về DP. Biết *vì sao* công thức đúng thì biết ngay *khi nào* nó hết đúng.

**Cách phân tích thay vì học thuộc:** với mọi bài đếm, hỏi tuần tự: (1) một đối tượng cần đếm được xác định bởi *chuỗi quyết định* nào? (2) các quyết định có độc lập không — nhân được không? (3) thứ tự có nghĩa không — có đang đếm trùng không, trùng bao nhiêu lần? (4) có tách được thành các trường hợp rời nhau để cộng không? Bốn câu hỏi này giải quyết đại đa số bài đếm mà không cần nhớ công thức nào ngoài P và C.

**Từ phỏng vấn sang production:** "Subsets" chính là bài toán liệt kê tổ hợp feature flags; "đếm trước khi liệt kê" chính là kỹ năng ước lượng feasibility trước khi viết batch job. Bài toán giống nhau, chỉ khác quy mô và tên biến.

## 9. Anti-pattern

**Nhân các lựa chọn phụ thuộc nhau.** "Hệ thống có 10 service, mỗi service 3 phiên bản → 3¹⁰ = 59.049 tổ hợp deploy cần test tương thích." Nghe logic, nhưng nếu contract giữa các service được version hóa và kiểm tra độc lập từng cặp, con số thật là tổng theo từng cặp tương tác — nhỏ hơn nhiều bậc. Ước lượng quá cao cũng nguy hiểm như quá thấp: nó dẫn đến quyết định "không thể test được, thôi bỏ" trong khi bài toán thật hoàn toàn khả thi.

**Thiết kế như thể collision không tồn tại.** Sinh ID ngẫu nhiên 8 ký tự rồi `INSERT` không có unique constraint và không có retry — vì "8 ký tự là nhiều lắm rồi". Pigeonhole (và birthday paradox, chương 12) nói collision là chuyện *khi nào* chứ không phải *có hay không*. Mọi ánh xạ từ tập lớn vào tập nhỏ đều cần đường xử lý va chạm, ngay từ thiết kế.

**Capacity planning bằng số trung bình.** "1 triệu request/ngày chia 10 server = 100k/server, mỗi server chịu được 150k → ổn." Pigeonhole đảm bảo có server nhận ≥ trung bình; phân bố thực tế (giờ cao điểm, hot tenant, hash lệch) đẩy max vượt xa mức đó. Thiết kế phải dựa trên phân phối và đuôi của nó (chương 18), không phải trên phép chia đều lý tưởng.

**Liệt kê để đếm.** Viết script sinh toàn bộ tổ hợp chỉ để lấy `len()` — chính là khoảnh khắc mở đầu chương. Với n nhỏ thì vô hại; thành thói quen thì có ngày script chạy mãi không xong và bạn không hiểu vì sao. Đếm trước, liệt kê sau, và chỉ liệt kê khi phép đếm nói rằng làm thế là khả thi.

**Tin giả định độc lập mà không kiểm chứng.** Bài học từ PostgreSQL ở mục 7 áp dụng cho chính code của bạn: mọi ước lượng dạng "xác suất cả hai cùng xảy ra = tích hai xác suất" đều đứng trên giả định độc lập. Hai service cùng phụ thuộc một database thì downtime của chúng *không* độc lập — và ước lượng availability bằng phép nhân sẽ lạc quan một cách nguy hiểm (chương 11).

## 10. Best Practices

**Nên:**

- Trước mọi lời giải brute force, backtracking hay batch job tổ hợp: **đếm kích thước state space trước, viết code sau.** So với ngưỡng ~10⁸ phép toán/giây để kết luận khả thi hay không trong 30 giây.
- Với mọi phép nhân trong ước lượng, tự hỏi thành tiếng: "các lựa chọn/sự kiện này có độc lập không?" Với mọi phép cộng: "các trường hợp có rời nhau không?" Hai câu hỏi này chặn được đa số lỗi đếm.
- Ưu tiên lập luận combinatorial ("chọn hay không chọn", "nhìn vào phần tử cuối") hơn biến đổi công thức — nó vừa khó sai hơn, vừa dẫn thẳng đến recurrence khi cần chuyển sang DP.
- Dùng pigeonhole như checklist thiết kế: mọi ánh xạ nhiều-vào-ít trong hệ thống (hash, partition, load balancer, cache key) đều cần câu trả lời cho "chuyện gì xảy ra khi va chạm / khi lệch tải".
- Khi tính C(n, k) trong code: nhân dần chia sớm hoặc tam giác Pascal — không bao giờ tính n! trực tiếp.

**Không nên:**

- Không kết luận "không thể xảy ra" cho bất kỳ sự kiện nào mà phép đếm cho thấy là tất yếu hoặc có xác suất tích lũy đáng kể theo thời gian.
- Không dùng số trung bình làm số thiết kế khi pigeonhole đã bảo đảm có phần tử vượt trung bình.
- Không học thuộc công thức đếm cho từng dạng bài — học bốn câu hỏi phân tích ở mục 8; công thức nào cũng dẫn xuất lại được từ rule of sum/product trong một phút.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Vì sao C(n, k) phải chia cho k!, và nguyên lý "chia cho số lần đếm trùng" áp dụng thế nào khi đếm số cách chia 2n người thành n cặp (không đánh số cặp)?
2. Pigeonhole và birthday paradox cùng nói về collision — mỗi cái trả lời câu hỏi gì, và khi thiết kế hệ thống sinh ID bạn cần cái nào?
3. Query `WHERE a = 1 AND b = 2` bị PostgreSQL ước lượng sai 1000 lần — giả định toán học nào đã đổ vỡ, và công cụ nào của PostgreSQL sửa nó?

---

*Chương tiếp theo: [06 — Recurrence Relations](/series/math-for-engineers/level-2-discrete-mathematics/06-recurrence/), nơi đẳng thức Pascal C(n,k) = C(n−1,k−1) + C(n−1,k) — một quan hệ truy hồi — được nghiên cứu như một đối tượng độc lập: ngôn ngữ mô tả mọi thuật toán đệ quy và chi phí của chúng.*
