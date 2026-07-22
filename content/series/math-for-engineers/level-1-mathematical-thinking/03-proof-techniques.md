+++
title = "Chương 03 — Proof Techniques: Vì sao code của bạn đúng"
date = "2026-07-20T07:30:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 1 – Mathematical Thinking
> Yêu cầu trước: Chương 01 (Logic), Chương 02 (Set Theory, Functions & Relations)

---

## 1. Problem Statement

Năm 1996, tên lửa Ariane 5 nổ tung 37 giây sau khi rời bệ phóng — nửa tỷ đô la bốc hơi vì một phép chuyển đổi số 64-bit sang 16-bit tràn số, trong một module đã chạy hoàn hảo nhiều năm... trên Ariane 4, nơi giá trị đó *không bao giờ* vượt ngưỡng. Code được test kỹ. Test đều pass. Nhưng test chỉ kiểm tra các giá trị *đã từng xảy ra*, còn tính đúng đắn là câu khẳng định về *mọi* giá trị có thể xảy ra.

Không cần đến tên lửa. Hãy nhìn hàm binary search — thuật toán 6 dòng mà Jon Bentley từng cho các lập trình viên chuyên nghiệp viết trong 2 giờ với đầy đủ thời gian test: **90% viết sai**. Và bản binary search trong thư viện chuẩn Java mang một bug tràn số (`(lo + hi) / 2`) suốt **9 năm** trước khi bị phát hiện năm 2006 — bởi chính Joshua Bloch, người đã "chứng minh" nó đúng trong luận văn nhưng chứng minh trên số nguyên toán học, không phải int 32-bit.

Dijkstra tóm tắt vấn đề trong một câu đáng khắc lên tường mọi phòng engineering:

> **"Testing shows the presence, not the absence of bugs."** — Test chứng minh *có* bug, không bao giờ chứng minh *không có* bug.

Dưới ánh sáng chương 01, điều này hiển nhiên: "code đúng" là mệnh đề ∀ ("với *mọi* input hợp lệ, output đúng"), còn một test pass chỉ xác lập một mệnh đề ∃ ("*tồn tại* input cho output đúng"). Không có lượng ∃ nào cộng lại thành ∀ khi không gian input vô hạn hoặc quá lớn — 100 test trên hàm nhận hai int64 phủ được 100 điểm trong không gian 2¹²⁸ điểm.

Bài toán của chương này:

**Làm sao xác lập được mệnh đề ∀ — "code này đúng với mọi input" — bằng lập luận hữu hạn, và làm điều đó đủ nhẹ để dùng hằng ngày chứ không chỉ trong luận văn?**

## 2. Trực giác

### Chứng minh là gì, bỏ lớp áo hàn lâm?

Một chứng minh là **chuỗi suy luận mà mỗi bước không thể cãi được**, đi từ điều đã chấp nhận đến điều cần xác lập. Kỹ sư làm việc này thường xuyên hơn họ tưởng: một comment giải thích "vì sao không cần lock ở đây", một câu trả lời code review "case rỗng ổn vì vòng lặp không chạy và ta trả về giá trị khởi tạo" — đều là chứng minh mini. Sự khác biệt giữa "cảm thấy đúng" và "chứng minh được" là sự khác biệt giữa "tôi chưa nghĩ ra counterexample" và "counterexample không tồn tại".

Ba kỹ thuật phủ gần hết nhu cầu của kỹ sư:

| Kỹ thuật | Ý tưởng một dòng | Dùng khi |
|---|---|---|
| Direct proof | đi thẳng: giả thiết → biến đổi → kết luận | mệnh đề có đường suy diễn ngắn |
| Contradiction | giả sử điều ngược lại, suy ra điều vô lý | chứng minh "không tồn tại", "không thể" |
| Induction | đúng cho case nhỏ nhất + đúng n kéo theo đúng n+1 | mệnh đề trên mọi n; cấu trúc đệ quy/lặp |

Và một kỹ thuật thứ tư — **loop invariant** — thực chất là induction mặc đồng phục kỹ sư, sẽ chiếm trung tâm chương này.

### Direct proof — khởi động

Mệnh đề: *tổng hai số chẵn là số chẵn.* Chứng minh trực tiếp: a chẵn nghĩa là a = 2i; b chẵn nghĩa là b = 2j; a + b = 2(i + j) — có dạng 2·(số nguyên), tức chẵn. ∎

Tầm thường? Nhìn kỹ cấu trúc: **mở định nghĩa ra → biến đổi đại số → đóng định nghĩa lại**. Đây chính là khuôn của các lập luận kiểu "hàm này idempotent vì f(f(x)) = f(x) do lần hai không thay đổi trường nào", hay "phép này giữ tổng số dư bằng 0 vì mỗi giao dịch cộng vào một tài khoản đúng số tiền trừ ở tài khoản kia". Direct proof là văn xuôi kỹ thuật có kỷ luật.

### Contradiction — sức mạnh của việc giả sử điều ngược lại

Có những mệnh đề không có đường suy diễn xuôi, đặc biệt là mệnh đề **phủ định tồn tại**: "không có input nào làm hàm này panic", "không tồn tại thuật toán nhanh hơn". Làm sao chứng minh cái *không có*? Không thể duyệt hết. Thay vào đó: **giả sử nó có, rồi cho thấy giả định đó tự hủy diệt** — dẫn đến p ∧ ¬p, điều mà logic chương 01 cấm tuyệt đối. Giả định gây ra mâu thuẫn thì giả định sai, tức mệnh đề gốc đúng.

Bản mẫu kinh điển, gọn trong bốn dòng — √2 là số vô tỉ:

> Giả sử √2 = a/b với a, b nguyên, phân số tối giản (a, b không cùng chẵn).
> Bình phương: a² = 2b² → a² chẵn → a chẵn (số lẻ bình phương ra lẻ) → a = 2k.
> Thay vào: 4k² = 2b² → b² = 2k² → b chẵn.
> Cả a và b chẵn — mâu thuẫn với "tối giản". Vậy không tồn tại phân số nào như thế. ∎

Điều đáng học không phải kết quả mà là *hình dạng lập luận*: từ giả định, mọi bước đều hợp lệ, và đích đến là điều phi lý — nên cánh cửa đã vào là cửa giả. Kỹ sư dùng đúng hình dạng này khi lập luận về hệ phân tán: "giả sử hai node cùng thành công acquire lock; cả hai phải ghi được vào ô đó với fencing token của mình; nhưng CAS chỉ cho một lần ghi thành công trên một giá trị cũ — mâu thuẫn; vậy mutual exclusion được bảo đảm."

## 3. First Principles

### Hai bất khả thi mọi kỹ sư nên biết — ở mức trực giác

**Không thể sort bằng so sánh nhanh hơn n log n.** Thuật toán sort chỉ-dùng-so-sánh, nhìn từ ngoài, là cỗ máy đặt câu hỏi có/không ("a[i] < a[j]?") rồi đưa ra kết luận là một hoán vị. Có n! hoán vị có thể là đáp án, và đáp án nào cũng phải *có khả năng* được đưa ra — nếu hai input có thứ tự khác nhau mà thuật toán trả lời giống nhau, nó sai ở ít nhất một input. Mỗi câu hỏi có/không chỉ chia không gian khả năng làm đôi. Muốn phân biệt n! khả năng cần ít nhất log₂(n!) câu hỏi, và log₂(n!) ≈ n log₂ n (xấp xỉ Stirling — chương 05). Vậy **mọi** thuật toán sort so sánh cần Ω(n log n) so sánh trong worst case. ∎ (dạng đầy đủ ở chương 16)

Hãy dừng lại ở sức nặng của kết luận: đây là khẳng định về mọi thuật toán *đã và sẽ được viết*, bởi bất kỳ ai, trên bất kỳ máy nào. Không benchmark nào, không lượng kinh nghiệm nào xác lập nổi điều đó. Và nó chỉ ra đúng chỗ lách: radix sort vượt "giới hạn" vì không so sánh — nó đọc cấu trúc bit của key, tức là chơi trò chơi khác. Hiểu chứng minh cho bạn biết giới hạn nằm ở *giả thiết nào*, và đổi giả thiết nào thì thoát.

**Halting problem — có những câu hỏi không chương trình nào trả lời được.** Giả sử tồn tại hàm `halts(program, input)` trả lời đúng câu hỏi "program có dừng trên input không" cho *mọi* chương trình. Xây chương trình nghịch tử:

```go
func rebel(p Program) {
    if halts(p, p) {   // nếu oracle bảo p dừng khi ăn chính nó...
        for {}         // ...thì ta chạy vĩnh viễn
    }                  // ngược lại: dừng ngay
}
```

Hỏi: `rebel(rebel)` dừng hay không? Nếu dừng → `halts` trả true → theo code, nó loop vĩnh viễn — mâu thuẫn. Nếu không dừng → `halts` trả false → theo code, nó dừng ngay — mâu thuẫn. Mọi ngả đều phi lý, vậy `halts` không tồn tại. ∎ Đây là lý do *nguyên tắc* khiến static analysis phải chấp nhận false positive/negative, compiler không thể loại mọi dead code, và không tool nào hứa "phát hiện mọi vòng lặp vô hạn" — các câu hỏi hành vi không tầm thường về chương trình tùy ý đều kéo theo halting problem (định lý Rice). Biết điều này, kỹ sư ngừng đòi hỏi tool điều bất khả và bắt đầu hỏi câu đúng: tool này *xấp xỉ về phía nào* — báo thừa hay bỏ sót?

### Induction — cỗ máy domino

Muốn chứng minh P(n) đúng với mọi số tự nhiên n, không cần vô hạn chứng minh. Chỉ cần hai điều:

> **Base case:** P(0) đúng.
> **Inductive step:** với mọi k, *nếu* P(k) đúng (giả thiết quy nạp) *thì* P(k+1) đúng.

Domino: quân đầu đổ, và mỗi quân đổ kéo quân sau đổ → cả hàng đổ. Điểm mà người mới hay nghẹn: "giả sử P(k) đúng — thế chẳng phải giả sử điều cần chứng minh sao?" Không: ta không giả sử P đúng với *mọi* k; ta chứng minh **một implication** (chương 01!) — P(k) → P(k+1). Implication này cộng với base case mới kéo ra tất cả.

Lập trình viên đã tin induction từ lâu mà không gọi tên, vì **đệ quy và induction là một cấu trúc nhìn từ hai phía**:

```go
// Vì sao dám tin hàm này đúng với MỌI cây, kể cả cây triệu node?
func size(t *Node) int {
    if t == nil {
        return 0            // base case — cây rỗng, đáp án 0: hiển nhiên đúng
    }
    return 1 + size(t.Left) + size(t.Right)
    // inductive step: GIẢ SỬ hai lời gọi con trả đúng kích thước hai cây con
    // (giả thiết quy nạp!), thì 1 + trái + phải đúng là kích thước cả cây
}
```

Niềm tin "lời gọi đệ quy cứ thế mà đúng" chính là giả thiết quy nạp; điều kiện dừng chính là base case; và yêu cầu "lời gọi con phải trên bài toán *nhỏ hơn*" chính là điều kiện để dây domino hữu hạn — vi phạm nó (đệ quy không giảm kích thước) là stack overflow, phiên bản runtime của một chứng minh quy nạp không có đáy.

**Strong induction** cho phép giả thiết mạnh hơn: P đúng với *mọi* giá trị < k+1 (không chỉ k). Cần thiết khi bài toán con không phải "nhỏ hơn 1 đơn vị": merge sort gọi trên n/2, không phải n−1; lập luận về nó cần được phép giả sử tính đúng cho *mọi* cỡ nhỏ hơn. Với đệ quy, strong induction là dạng tự nhiên: "mọi lời gọi trên input nhỏ hơn đều đúng" — bất kể nhỏ hơn bao nhiêu.

### Nếu không có các kỹ thuật này thì sao?

Không có induction, mọi khẳng định về vòng lặp/đệ quy/cấu trúc dữ liệu chỉ kiểm được đến cỡ đã test — và Ariane 5 là chuyện gì xảy ra ở cỡ chưa test. Không có contradiction, ta không thể *biết* giới hạn — và sẽ có người tiêu cả sự nghiệp tối ưu sort so sánh xuống O(n). Không có invariant (ngay sau đây), mỗi lần sửa một vòng lặp là một lần đánh cược.

## 4. Mathematical Model

### Loop invariant — induction cho vòng lặp

Vòng lặp là đệ quy được duỗi thẳng, nên cũng chứng minh được bằng induction — nhưng dạng chuyên dụng của nó tiện hơn nhiều. Một **loop invariant** là mệnh đề về trạng thái chương trình thỏa ba điều kiện:

> **Initialization:** đúng ngay trước lần lặp đầu tiên. *(base case)*
> **Maintenance:** nếu đúng trước một lần lặp, thì đúng sau lần lặp đó. *(inductive step)*
> **Termination:** vòng lặp có kết thúc, và khi kết thúc, invariant ∧ điều-kiện-thoát ⇒ điều cần chứng minh.

Điều kiện thứ ba là phần người ta hay quên và là phần trả công: invariant tự nó vô dụng nếu lúc thoát nó không *cộng hưởng* với điều kiện thoát để cho ra kết luận.

### Binary search — chứng minh thay cho 2 giờ debug

Đây là thuật toán mà 90% chuyên gia viết sai khi chỉ dựa vào trực giác. Giờ viết nó bằng cách **chọn invariant trước, để invariant ép ra từng dòng code**:

Invariant được chọn: *nếu target có trong mảng thì nó nằm trong đoạn chỉ số [lo, hi]* — tương đương: **∀i < lo: a[i] < target, và ∀i > hi: a[i] > target** (quantifier của chương 01 làm việc).

```go
func binarySearch(a []int, target int) int {
    lo, hi := 0, len(a)-1
    // Initialization: [lo,hi] = [0,n-1] là cả mảng — chưa loại ai, invariant đúng rỗng
    for lo <= hi {
        mid := lo + (hi-lo)/2 // KHÔNG viết (lo+hi)/2 — bug 9 năm của Java: lo+hi tràn int
        switch {
        case a[mid] == target:
            return mid
        case a[mid] < target:
            lo = mid + 1 // a[mid] < target và mảng sort → mọi i ≤ mid đều < target
                         // → gán lo = mid+1 GIỮ NGUYÊN invariant (Maintenance nửa trái)
        default:
            hi = mid - 1 // đối xứng: mọi i ≥ mid đều > target (Maintenance nửa phải)
        }
    }
    // Termination: lo > hi → đoạn [lo,hi] rỗng → theo invariant, target không tồn tại
    return -1
}
```

Giờ mọi câu hỏi kinh điển đều có đáp án *suy ra được* thay vì nhớ:

- `lo <= hi` hay `lo < hi`? Invariant nói vùng ứng viên là [lo, hi] — đoạn một phần tử (lo = hi) vẫn phải xét, vậy điều kiện chạy là `lo <= hi`. Nếu bạn chọn invariant kiểu nửa-mở [lo, hi) — cũng hợp lệ! — thì thành `lo < hi` và `hi = mid`. **Hai trường phái không phải khẩu vị; chúng là hai invariant khác nhau, và mỗi dòng code phải trung thành với invariant đã chọn.** Bug xuất hiện khi trộn nửa nọ với nửa kia.
- `lo = mid + 1` hay `lo = mid`? Invariant đòi mọi chỉ số < lo đều mang giá trị < target. Ta vừa biết a[mid] < target, nên lo được phép nhảy tới mid+1. Viết `lo = mid` không sai invariant nhưng vi phạm Termination: với đoạn 2 phần tử, mid = lo, vòng lặp đứng yên — treo vĩnh viễn. Termination cần một **đại lượng giảm ngặt** (hi − lo giảm mỗi vòng) — khái niệm gọi là *variant*, người anh em của invariant.
- Off-by-one? Không còn chỗ sống: mỗi ±1 đều là mệnh lệnh của invariant, không phải phép thử-sai.

### Insertion sort — invariant phát biểu "sort là gì"

```go
func insertionSort(a []int) {
    // Invariant: trước vòng lặp với biến i, đoạn a[0:i] là PHIÊN BẢN ĐÃ SORT
    // của chính i phần tử đầu ban đầu (sort + đúng multiset — thiếu vế sau,
    // hàm ghi đè toàn mảng bằng số 0 cũng "thỏa" sort!)
    for i := 1; i < len(a); i++ {
        key := a[i]
        j := i - 1
        // vòng trong có invariant riêng: a[j+2:i+1] đều > key và đã dịch phải một ô
        for j >= 0 && a[j] > key {
            a[j+1] = a[j]
            j--
        }
        a[j+1] = key
        // Maintenance: chèn key vào đúng vị trí trong đoạn sort
        // → a[0:i+1] sort và vẫn là đúng bộ phần tử cũ
    }
    // Termination: i = n → a[0:n] sort — chính là điều cần chứng minh
}
```

Ghi chú trong ngoặc đáng giá nhất chương: invariant "a[0:i] đã sort" **chưa đủ** — phải kèm "và là hoán vị của các phần tử ban đầu". Đặc tả thiếu vế bảo toàn là loại lỗi đặc tả có thật (hàm dedupe "giữ mảng sort" bằng cách... xóa nhầm phần tử vẫn qua được test chỉ kiểm tra tính sort). Viết invariant buộc ta viết *đặc tả đầy đủ* — và đó thường là lúc phát hiện mình chưa từng nghĩ kỹ hàm này thực sự hứa gì.

### Invariant vượt khỏi vòng lặp — bất biến của hệ thống

Khái niệm không dừng ở vòng lặp. Một **system invariant** là mệnh đề đúng ở *mọi trạng thái quan sát được* của hệ thống:

- **Money conservation trong ledger:** tổng mọi bút toán của một giao dịch bằng 0 (double-entry). Mọi thao tác chuyển tiền phải là *đơn vị nguyên tử* gồm ghi nợ + ghi có; DB transaction tồn tại chính là để các trạng thái trung gian vi phạm invariant không bao giờ được nhìn thấy. Chữ C trong ACID — Consistency — định nghĩa chính xác là "transaction đưa DB từ trạng thái thỏa invariant sang trạng thái thỏa invariant".
- **Cấu trúc dữ liệu:** "cây đỏ-đen không có hai node đỏ liền nhau và mọi đường xuống lá cùng số node đen" — mọi phép insert/delete phải bảo toàn (Maintenance!), đổi lại được chiều cao O(log n) vĩnh viễn. B-Tree của PostgreSQL, skip list của Redis — mỗi cấu trúc là một bó invariant cộng các thao tác được *chứng minh* bảo toàn chúng.
- **Concurrency:** mutex bảo vệ invariant của vùng dữ liệu — trong critical section, invariant *được phép* tạm vỡ; ra khỏi đó phải lành lặn. Nhìn vậy, câu hỏi "lock ở granularity nào" có dạng toán học: biên của lock phải bao trọn một lần vỡ-rồi-lành của invariant.

## 5. Thuật toán — quy trình dùng chứng minh trong công việc thật

Không ai viết chứng minh hình thức cho CRUD handler. Kỹ năng thực dụng là **chọn đúng liều lượng**:

**Bước 1 — Viết invariant thành một câu, trước hoặc ngay sau khi viết vòng lặp không tầm thường.** Dạng chuẩn: "trước lần lặp thứ i, X đúng về đoạn/tập đã xử lý". Nếu không viết nổi câu đó, bạn chưa hiểu vòng lặp của chính mình — và đó là thông tin quý.

**Bước 2 — Kiểm ba chốt:** đúng lúc vào? mỗi vòng giữ được? lúc thoát cộng với điều kiện thoát cho ra điều muốn? Ba câu hỏi, mỗi câu 30 giây với vòng lặp thường gặp.

**Bước 3 — Tìm variant cho termination:** đại lượng nguyên không âm giảm ngặt mỗi vòng (hi − lo, số phần tử chưa xử lý, độ sâu còn lại). Vòng lặp `for` trên range tự có; vòng `for` điều kiện và đệ quy thì phải chỉ ra được.

**Bước 4 — Mã hóa những gì mã hóa được:**

```go
// Invariant rẻ → assert thẳng trong dev/test build
func (l *Ledger) applyTx(tx Tx) {
    var sum int64
    for _, e := range tx.Entries {
        sum += e.Amount
    }
    if sum != 0 { // money conservation — vi phạm là dừng ngay tại nguồn
        panic("ledger invariant violated: entries do not sum to zero")
    }
    // ... ghi các entry trong cùng một DB transaction
}
```

**Bước 5 — Với invariant trên không gian input lớn: property-based testing.** Thay vì chọn từng input, phát biểu *tính chất* (chính là invariant/đặc tả) và để máy bắn hàng nghìn input ngẫu nhiên, tự thu nhỏ counterexample:

```go
// go test với testing/quick — kiểm tính chất thay vì từng case
func TestSortProperties(t *testing.T) {
    f := func(a []int) bool {
        b := slices.Clone(a)
        insertionSort(b)
        return slices.IsSorted(b) &&              // tính chất 1: sort
            sameMultiset(a, b)                     // tính chất 2: bảo toàn phần tử
    }
    if err := quick.Check(f, nil); err != nil {
        t.Error(err) // quick tự in input gây vỡ tính chất
    }
}
```

Property-based testing là điểm giữa cây cầu test–chứng minh: vẫn là ∃ (mẫu hữu hạn), nhưng buộc bạn *viết ra mệnh đề ∀* — và một nửa giá trị nằm ở hành động viết đó. Điểm cuối cây cầu là **TLA+**: ngôn ngữ đặc tả của Leslie Lamport, nơi bạn mô tả hệ thống như máy trạng thái và model checker duyệt *toàn bộ* không gian trạng thái (đến cỡ cấu hình cho trước) để kiểm invariant. AWS dùng TLA+ cho DynamoDB, S3 và tìm ra bug chỉ hiện hình sau 35 bước tương tác — thứ không test tích hợp nào với xác suất thực tế chạm tới. Kỹ sư thường không cần viết TLA+; nhưng cần biết nó tồn tại để, khi thiết kế protocol phân tán có tiền hoặc tính mạng phía sau, biết mình đang thiếu công cụ gì.

## 6. Trade-off

**Chi phí chứng minh vs chi phí sai.** Trục quyết định là *blast radius* của cái sai. CRUD hiển thị danh sách: test ví dụ là đủ, chứng minh là xa xỉ. Thuật toán lõi chạy tỷ lần/ngày, hàm tính tiền, giao thức replication: một invariant viết ra + property test là khoản đầu tư rẻ nhất so với hậu quả. Quy tắc thô: **độ hình thức tỷ lệ với (tần suất chạy × chi phí một lần sai × độ khó phát hiện khi sai)**.

**Chứng minh trên mô hình vs hành vi trên máy thật.** Bài học Joshua Bloch: chứng minh đúng trên số nguyên toán học, code chạy trên int64 có tràn, float có làm tròn, map Go có thứ tự duyệt ngẫu nhiên. Chứng minh chỉ chuyển tính đúng *từ giả thiết xuống kết luận* — sai giả thiết (mô hình máy) thì kết luận vô giá trị. Đây không phải lý do bỏ chứng minh; là lý do phát biểu giả thiết tường minh: "với n ≤ 2⁶² thì lo+hi không tràn" tự nó đã là một dòng phòng thủ.

**Invariant chặt vs linh hoạt tiến hóa.** Invariant càng mạnh càng dễ suy luận nhưng càng bó tay thay đổi. "Mọi user có đúng một email" đơn giản hóa mọi query — đến ngày product cần multi-email và cả codebase đã ngầm dựa vào invariant đó. Nghệ thuật là chọn invariant *bản chất* (money conservation — không bao giờ đổi) làm nền, và dè dặt với invariant *tình cờ* (định dạng ID, số lượng tối đa hiện tại).

**Assert luôn-chạy vs chỉ-dev.** Check invariant trong hot path tốn chu kỳ CPU; tắt trong production lại mù đúng lúc cần mắt. Thực hành phổ biến: invariant rẻ (so sánh vài số — như tổng bút toán) giữ vĩnh viễn, coi như hàng rào cuối trước khi hỏng dữ liệu; invariant đắt (quét toàn cấu trúc kiểm tính sort) chỉ bật trong test/canary. Vi phạm invariant khác lỗi thường: nó nghĩa là *mô hình thế giới của code đã sai* — fail fast gần như luôn rẻ hơn chạy tiếp trên nền móng đã nứt.

**Khi nào KHÔNG dùng?** Khi đặc tả còn đang đổi hằng tuần (startup tìm product-market fit), chứng minh tính đúng so với đặc tả là chứng minh so với mục tiêu di động — hãy đầu tư vào khả năng *thay code nhanh* thay vì *code bất biến đúng*. Và đừng dùng chứng minh thay đo lường: "thuật toán này O(n log n) nên nhanh hơn" là mệnh đề về tiệm cận, không phải về 10ms latency budget của bạn (chương 10).

## 7. Production Applications

**Database transaction và ràng buộc khai báo.** `CHECK`, `UNIQUE`, `FOREIGN KEY`, `EXCLUDE` của PostgreSQL là invariant *do engine cưỡng chế* — Maintenance được kiểm mọi lần ghi, và serializable isolation tồn tại để invariant nhiều-dòng (tổng số dư ≥ 0 trên hai tài khoản) không bị hai transaction đan xen phá theo cách từng transaction riêng lẻ vô tội. Chọn isolation level thấp hơn chính là *chọn danh sách invariant mình chấp nhận có thể vỡ tạm thời* — hiểu vậy, cấu hình `READ COMMITTED` vs `SERIALIZABLE` thành quyết định có nội dung thay vì nghi thức.

**Go runtime tự vệ bằng invariant.** Race detector kiểm invariant "không hai goroutine cùng chạm một ô nhớ không đồng bộ"; map runtime phát hiện ghi đồng thời và panic chủ động — fail fast trên vi phạm invariant thay vì âm thầm hỏng bucket; GC tri-color dựa trên bất biến "không con trỏ từ object đen tới object trắng" — write barrier tồn tại chỉ để Maintenance bất biến đó trong khi chương trình đang chạy song song với GC. Đọc tài liệu GC bằng con mắt invariant, mọi mảnh thiết kế đột nhiên có lý do.

**Kubernetes — reconciliation là Maintenance bằng vòng lặp vĩnh viễn.** Controller không "thực hiện lệnh"; nó liên tục so trạng thái thật với trạng thái khai báo và sửa lệch — một cỗ máy đưa hệ về invariant "actual = desired" sau mọi nhiễu loạn. Đây là invariant kiểu *tự phục hồi* (convergence): thay vì chứng minh không bao giờ vỡ (bất khả trong hạ tầng thật — node cứ chết), chứng minh *mọi trạng thái vỡ đều được kéo về trạng thái lành*. CRDT trong hệ phân tán chơi cùng nước cờ: các bản sao được phép lệch, phép merge được chứng minh (giao hoán + kết hợp + idempotent — ba tính chất đại số kiểm được!) hội tụ về một giá trị.

**Property-based testing trong hệ thống thật.** Kỹ thuật sinh ở Haskell (QuickCheck) giờ là công cụ công nghiệp: fuzzing của Go (`go test -fuzz`) là property testing với input sinh có hướng dẫn bởi coverage — chuyên bắt invariant "không panic, không out-of-bounds với *mọi* byte input"; Jepsen kiểm invariant linearizability của database phân tán bằng cách bơm lỗi mạng rồi *kiểm chứng lịch sử thao tác so với đặc tả* — chuỗi báo cáo Jepsen làm lộ vi phạm ở hàng chục DB thương mại từng quảng cáo "đã test kỹ". Cùng một bài học Dijkstra: nhà sản xuất test sự *có mặt* của tính đúng trên kịch bản đẹp; Jepsen truy lùng sự *vắng mặt* của nó.

**Code review như một phiên chứng minh xã hội.** Những comment review giá trị nhất có dạng chứng minh hoặc phản chứng minh: "nếu hai request cùng qua check này trước khi cái nào kịp ghi, cả hai cùng pass — counterexample cho invariant unique"; "vòng while này thiếu variant — client giữ kết nối là ta quay vô hạn". Team giỏi không review *code*, họ review *lập luận vì sao code đúng* — và PR description tốt nhất là bản phác cái lập luận đó sẵn cho người review kiểm.

## 8. Interview

Phỏng vấn thuật toán thực chất chấm hai bài: tìm ra lời giải, và **chứng tỏ nó đúng**. Ứng viên trung bình dừng ở "chạy thử ví dụ thấy ra kết quả"; ứng viên mạnh nói được *vì sao mọi input đều ổn* — và người phỏng vấn có kinh nghiệm phân biệt hai hạng người này trong ba phút.

**Trình bày invariant khi phỏng vấn — khuôn ba câu:** (1) "Bất biến của vòng lặp này là: [mệnh đề về phần đã xử lý]". (2) "Nó đúng lúc vào vì [khởi tạo], và mỗi bước giữ nó vì [thao tác]". (3) "Khi thoát, [bất biến + điều kiện thoát] cho đúng kết quả đề hỏi". Ba câu, dưới một phút, áp được cho hầu hết bài mảng/hai con trỏ/sliding window — và thường *tự vá* off-by-one trước khi kịp viết sai.

**Các bài mà invariant là lời giải:**

- **Binary Search (LeetCode 704) và cả họ biến thể** (35 Search Insert, 33 Rotated Array, 278 First Bad Version, 162 Peak): mỗi biến thể chỉ là một invariant khác. Bài 278: "∀i < lo: version i tốt; ∀i ≥ hi: version i hỏng" — hai biên khép lại đúng điểm gãy. Ai học thuộc template sẽ chết ở biến thể; ai viết invariant thì biến thể chỉ là thay một mệnh đề.
- **Remove Duplicates from Sorted Array (26), Move Zeroes (283):** invariant hai con trỏ — "a[0:w] là kết quả đúng của phần đã đọc". Toàn bộ họ bài in-place rewrite là một invariant này.
- **Kadane (53 Maximum Subarray):** invariant "best là đáp án cho a[0:i], cur là tổng lớn nhất của đoạn *kết thúc tại* i" — phát biểu được hai vế này thì thuật toán 4 dòng tự rơi ra, và bạn vừa làm dynamic programming mà chưa cần gọi tên (chương 14).
- **Câu "vì sao đúng?" sau lời giải greedy:** interview cấp cao hay hỏi tiếp — và exchange argument (chương 15) là một proof by contradiction: "giả sử tồn tại lời giải tốt hơn khác lựa chọn tham lam ở bước đầu tiên; hoán đổi cho thấy lời giải đó không tốt hơn — mâu thuẫn".

**Lỗi tư duy thường gặp:**

- Kiểm 2–3 ví dụ rồi tuyên bố đúng — ∃ đội lốt ∀, người phỏng vấn chỉ cần một câu "input nào làm nó sai?" là lộ.
- Sửa off-by-one bằng cách thử `±1` đến khi test pass — thay vì hỏi invariant đòi biên nào. Nhìn từ ngoài, hai người cùng sửa xong; nhìn từ ghế phỏng vấn, một người dò mìn và một người đọc bản đồ mìn.
- Vòng while thiếu lập luận termination — đặc biệt khi hai con trỏ có nhánh không dịch chuyển con trỏ nào.
- Đệ quy không phát biểu được "hàm này hứa gì" — thiếu giả thiết quy nạp thì không thể kiểm bước gọi con có dùng đúng lời hứa.

**Từ phỏng vấn sang production:** ba câu invariant của buổi phỏng vấn chính là ba dòng comment nên nằm trên vòng lặp phức tạp trong codebase, là mục "correctness" trong design doc, là câu trả lời cho reviewer hỏi "sao chỗ này không cần lock". Một kỹ năng, ba thị trường.

## 9. Anti-pattern

**"Test pass rồi, đúng rồi."** Phiên bản công nghiệp của ∃ → ∀. Test là *kiểm tra tại điểm*; tính đúng là *mệnh đề trên toàn miền*. Cặp đôi lành mạnh: chứng minh phác (invariant) cho cấu trúc + test cho hiện thực và cho hồi quy — cái trước trả lời "thiết kế có đúng không", cái sau trả lời "code có làm như thiết kế không, và còn thế sau này không".

**Chứng minh bằng ví dụ, bác bỏ bằng uy tín.** Trong design review: "cách này ổn, hệ X cũng làm vậy" (ví dụ) đấu với "senior bảo không được" (uy tín) — cả hai đều không phải lập luận. Một counterexample cụ thể bác bỏ được mệnh đề ∀ của bất kỳ ai — đây là tính dân chủ đẹp nhất của toán: junior chỉ ra kịch bản hai request đan xen phá invariant thì thiết kế sai, chức danh không cứu được.

**Assert rồi nuốt.** `if invariantBroken { log.Warn(...); continue }` — phát hiện nền móng nứt và... ghi nhật ký rồi xây tiếp. Dữ liệu hỏng lan theo mọi luồng ghi sau đó; đến lúc phát hiện qua khiếu nại khách hàng, forensic phải đào hàng tuần ngược về chỗ có sẵn dòng warn không ai đọc. Vi phạm invariant là lỗi loại "dừng lại", hiếm ngoại lệ.

**Đặc tả một nửa.** Chứng minh hàm sort trả mảng sort — quên "cùng phần tử"; chứng minh cache trả nhanh — quên "trả *đúng*"; chứng minh retry rồi cũng thành công — quên "không thành công *hai lần*" (idempotency). Cái hàm bị chứng minh đúng so với đặc tả thiếu là con ngựa thành Troia: có dấu kiểm chất lượng ngay trên bụng rỗng.

**Sùng bái hình thức.** Chiều ngược lại cũng là anti-pattern: đòi invariant cho mọi vòng `for i := range`, viết TLA+ cho form đăng ký nhận bản tin. Chứng minh là công cụ quản trị rủi ro; dùng nó nơi rủi ro không đáng là đốt ngân sách chú ý của team — thứ tài nguyên hữu hạn nhất.

## 10. Best Practices

**Nên:**

- Với mỗi vòng lặp không tầm thường: viết invariant một câu (comment ngay trên vòng lặp), kiểm nhẩm ba chốt Initialization–Maintenance–Termination.
- Với mỗi vòng `while`/đệ quy: chỉ ra variant — đại lượng không âm giảm ngặt. Không chỉ ra được là chưa được merge.
- Xếp hạng invariant của hệ thống mình đang giữ (tiền, uniqueness, ordering...) và mã hóa những cái rẻ thành assert/constraint chạy cả production; những cái đắt thành property test.
- Dùng contrapositive khi debug (chương 01) và contradiction khi thiết kế: "giả sử sự cố X xảy ra — chuỗi nào dẫn tới nó, mắt xích nào bị invariant chặn?"
- Viết PR description như bản phác chứng minh: đây là bất biến, đây là vì sao thao tác mới giữ nó.

**Không nên:**

- Không kết luận ∀ từ ∃ — bất kể bao nhiêu test xanh, demo mượt, hay "chạy ba năm chưa sao" (Ariane 4 cũng vậy).
- Không sửa lỗi biên bằng thử-sai ±1; quay lại invariant, để nó phán xử.
- Không quên kiểm giả thiết của chứng minh trên máy thật: tràn số, làm tròn float, thứ tự duyệt map, reorder của bộ nhớ — mô hình sai thì chứng minh đẹp mấy cũng vô hiệu.
- Không cho code "chạy tiếp xem sao" sau khi tự tay phát hiện invariant vỡ.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Phát biểu invariant đầy đủ của binary search trên đoạn nửa-mở [lo, hi), rồi suy ra — không nhớ lại — điều kiện vòng lặp, hai phép gán biên, và giá trị trả về khi không tìm thấy.
2. Vì sao "test pass" là mệnh đề ∃ còn "code đúng" là mệnh đề ∀, và property-based testing đứng ở đâu giữa hai bờ đó — nó thay đổi được gì và không thay đổi được gì?
3. Chọn một hệ bạn đang làm: kể ra ba invariant nó phải giữ, cái nào đang được cưỡng chế bằng máy (constraint, assert, type), cái nào chỉ sống trong đầu team — và cái thứ hai sẽ vỡ kiểu gì khi người mới join?

---

*Chương tiếp theo: [04 — Mathematical Modeling](/series/math-for-engineers/level-1-mathematical-thinking/04-mathematical-modeling/), nơi ta lùi một bước trước cả chứng minh: làm sao biến một bài toán thực tế lộn xộn thành mô hình toán — chọn đúng cấu trúc để mọi công cụ của ba chương qua có đất dụng võ.*
