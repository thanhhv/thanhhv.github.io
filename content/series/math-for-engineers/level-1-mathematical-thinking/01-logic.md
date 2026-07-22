+++
title = "Chương 01 — Logic: Nền tảng của mọi dòng code"
date = "2026-07-20T07:10:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 1 – Mathematical Thinking
> Yêu cầu trước: Không — đây là chương mở đầu của toàn bộ tài liệu

---

## 1. Problem Statement

Một team backend nhận ticket: "Tìm tất cả khách hàng chưa từng đặt hàng để gửi email khuyến mãi". Kỹ sư viết câu SQL trông hoàn toàn hợp lý:

```sql
SELECT * FROM customers
WHERE id NOT IN (SELECT customer_id FROM orders);
```

Query chạy không lỗi, trả về... **0 dòng**. Trong khi ai cũng biết có hàng nghìn khách hàng chưa đặt hàng. Không exception, không warning, không log. Nguyên nhân: bảng `orders` có *một* dòng với `customer_id IS NULL` (đơn hàng của khách vãng lai). Và trong logic ba giá trị của SQL, `x NOT IN (1, 2, NULL)` không bao giờ là TRUE — với *mọi* x.

Đây không phải là bug của SQL. Đây là bug của **lập trình viên không nắm vững logic** — cụ thể là không biết hệ logic mình đang đứng trong đó có hai giá trị hay ba giá trị, và phép phủ định hoạt động thế nào trong hệ đó.

Bài toán của chương này:

**Làm sao suy luận chính xác về điều kiện — thứ xuất hiện trong mọi câu `if`, mọi mệnh đề `WHERE`, mọi guard clause — để code làm đúng điều ta nghĩ, chứ không phải điều ta viết?**

Nếu không có logic hình thức, lập trình viên chỉ còn cách "đọc điều kiện và cảm nhận". Với điều kiện 2 biến thì cảm nhận đủ dùng. Với điều kiện 5 biến lồng phủ định — kiểu `!(a && (!b || c)) || (d && !e)` — cảm nhận là con đường ngắn nhất đến bug production lúc 3 giờ sáng.

## 2. Trực giác

### Logic là gì, dưới góc nhìn của kỹ sư?

Logic là môn học về **các câu khẳng định và cách kết hợp chúng mà vẫn giữ được tính đúng/sai**. Với lập trình viên, nó không trừu tượng chút nào: mỗi biểu thức boolean trong code là một câu khẳng định, và compiler/runtime là cỗ máy đánh giá tính đúng sai của câu đó hàng tỷ lần mỗi giây.

Một **mệnh đề (proposition)** là câu khẳng định có giá trị đúng hoặc sai xác định: "user đã đăng nhập", "số dư ≥ 0", "x > 10". Câu "hãy đăng nhập đi" hay "x có lớn không?" không phải mệnh đề. Trong code, mệnh đề chính là biểu thức kiểu `bool`.

Ba phép kết hợp cơ bản ai cũng dùng hằng ngày:

| Toán học | Go | Đọc là |
|---|---|---|
| p ∧ q | `p && q` | p VÀ q |
| p ∨ q | `p \|\| q` | p HOẶC q (hoặc cả hai!) |
| ¬p | `!p` | KHÔNG p |

Chú ý ngay một cái bẫy trực giác: **OR trong logic là "hoặc bao gồm" (inclusive or)** — `p || q` đúng khi cả hai cùng đúng. Trong tiếng Việt (và tiếng Anh) đời thường, "hoặc" thường mang nghĩa loại trừ: "trà hoặc cà phê" ngụ ý chọn một. Sự lệch pha giữa ngôn ngữ tự nhiên và logic hình thức chính là nguồn gốc của rất nhiều điều kiện viết sai — khi PM nói "hoặc", việc đầu tiên là hỏi lại: *có được phép cả hai không?*

### Implication — phép toán bị hiểu sai nhiều nhất

Phần lớn lập trình viên thuộc lòng AND/OR/NOT nhưng lúng túng với **implication**: p → q, đọc là "nếu p thì q". Bảng chân trị của nó gây bối rối:

| p | q | p → q |
|---|---|---|
| T | T | **T** |
| T | F | **F** |
| F | T | **T** ← khó chịu ở đây |
| F | F | **T** ← và ở đây |

Vì sao "p sai" lại khiến cả mệnh đề đúng? Hãy nghĩ implication như một **lời hứa**, không phải quan hệ nhân quả. "Nếu trời mưa, tôi sẽ mang ô" — lời hứa này chỉ *bị phá vỡ* trong đúng một tình huống: trời mưa mà tôi không mang ô. Trời không mưa? Tôi mang ô hay không đều không phá vỡ lời hứa. Lời hứa chưa bị phá vỡ = mệnh đề đúng.

Với kỹ sư, cách nhìn hữu ích nhất: **p → q tương đương với ¬p ∨ q**. Đây chính là guard clause:

```go
// "Nếu req không nil thì xử lý" — implication trong code
func handle(req *Request) error {
    if req == nil {
        return nil // p sai → cả cam kết vẫn "đúng", thoát sớm
    }
    return process(req) // chỉ khi p đúng, q mới phải được thực hiện
}
```

Contract kiểu "nếu input hợp lệ thì output đúng đắn" là một implication. Nó **không hứa gì cả** khi input không hợp lệ — hiểu điều này giúp bạn đọc đúng documentation: hàm ghi "nếu slice đã sort, trả về vị trí phần tử" hoàn toàn được phép trả rác khi slice chưa sort, và đó không phải bug.

### Converse và Contrapositive — chỗ suy luận trượt chân

Từ p → q, có ba biến thể, và chỉ **một** trong số đó tương đương với gốc:

| Tên | Dạng | Tương đương với p → q? |
|---|---|---|
| Converse (đảo) | q → p | **KHÔNG** |
| Inverse (phản) | ¬p → ¬q | **KHÔNG** |
| Contrapositive (phản đảo) | ¬q → ¬p | **CÓ** |

Lỗi suy luận kinh điển là dùng converse: "Nếu code có bug thì test fail. Test pass, vậy code không có bug." — sai. Mệnh đề gốc là `bug → test fail`; từ `test pass` (¬(test fail)), theo contrapositive ta chỉ suy ra được... ơ không, đúng ra được ¬bug? Không! Cẩn thận: mệnh đề gốc thực tế trong đời là "nếu test fail thì có bug" (`test fail → bug`). Test pass là ¬p, và từ ¬p không suy ra được gì về q. **Test pass không chứng minh không có bug** — đây chính là câu nói nổi tiếng của Dijkstra mà chương 03 sẽ quay lại.

Contrapositive thì hợp lệ và cực kỳ hữu ích trong debugging: "Nếu config đúng thì service khởi động được. Service không khởi động được → config sai *hoặc mệnh đề của tôi thiếu điều kiện*". Debug giỏi phần lớn là kỹ năng áp contrapositive một cách kỷ luật: từ triệu chứng (¬q) truy ngược về phủ định của nguyên nhân giả định (¬p).

## 3. First Principles

### Vì sao chỉ cần đúng/sai là đủ xây mọi thứ?

Toàn bộ logic mệnh đề đứng trên một quyết định trừu tượng hóa táo bạo: **mọi câu khẳng định quy về đúng một bit**. Ta vứt bỏ sắc thái, ngữ cảnh, mức độ — đổi lấy khả năng *tính toán được* trên các câu khẳng định. Giống Big-O vứt hằng số để được tính bất biến, logic vứt sắc thái để được tính cơ giới: máy móc kiểm tra được, biến đổi được, chứng minh được.

Từ hai giá trị và vài phép nối, ta được một **đại số** — nghĩa là các biểu thức biến đổi được theo luật, giống hệt biến đổi phương trình số học:

| Luật | Dạng |
|---|---|
| Giao hoán | p ∧ q ≡ q ∧ p |
| Kết hợp | (p ∧ q) ∧ r ≡ p ∧ (q ∧ r) |
| Phân phối | p ∧ (q ∨ r) ≡ (p ∧ q) ∨ (p ∧ r) |
| **De Morgan** | ¬(p ∧ q) ≡ ¬p ∨ ¬q |
| **De Morgan** | ¬(p ∨ q) ≡ ¬p ∧ ¬q |
| Phủ định kép | ¬¬p ≡ p |

Chữ ≡ nghĩa là hai biểu thức có cùng giá trị với **mọi** cách gán đúng/sai cho các biến — kiểm chứng được bằng truth table. Đây là điểm then chốt: hai đoạn code điều kiện tương đương logic thì **thay thế được cho nhau một cách an toàn tuyệt đối**, không cần test, không cần "chắc là đúng". Refactor điều kiện là một trong số ít chỗ trong nghề mà ta có được sự chắc chắn toán học.

### De Morgan — luật đáng giá nhất cho việc đọc code

Trực giác của De Morgan: phủ định của "cả hai" là "ít nhất một cái không"; phủ định của "ít nhất một" là "cả hai đều không". Phủ định đi *xuyên qua* ngoặc và **lật AND thành OR** (và ngược lại).

Xem một điều kiện thật thường gặp trong review:

```go
// Trước: đọc 3 lần vẫn chưa chắc hiểu đúng
if !(user.IsActive && !user.IsBanned) {
    return ErrAccessDenied
}

// Áp De Morgan: !(A && !B) ≡ !A || B
if !user.IsActive || user.IsBanned {
    return ErrAccessDenied
}
// Giờ đọc thành lời được ngay: "chưa active HOẶC đã bị ban thì chặn"
```

Quy tắc thực dụng rút ra: **đẩy phủ định vào sát biến** (inward), vì `!user.IsActive` đọc hiểu ngay còn `!(...)` bắt não người đọc tự chạy De Morgan trong đầu — và não người chạy De Morgan sai với tỷ lệ đáng báo động khi có hơn hai toán hạng. Nhiều linter (như `staticcheck` của Go) có rule tự động gợi ý biến đổi này.

### Nếu không có logic hình thức thì sao?

Không có khái niệm "tương đương logic", mọi refactor điều kiện đều là canh bạc: đổi `!(a && b)` thành `!a && !b` (sai!) và chỉ phát hiện khi khách hàng phàn nàn. Không có khái niệm implication, ta đọc sai contract của thư viện. Không có quantifier (ngay dưới đây), ta không thể nói chính xác những câu như "mọi request đều được authenticate" — mà nói không chính xác được thì cũng không kiểm tra được.

## 4. Mathematical Model

### Predicate và Quantifier — khi mệnh đề có biến

Mệnh đề "x > 10" chưa đúng chưa sai — nó phụ thuộc x. Đó là một **predicate**: hàm từ giá trị sang đúng/sai. Trong Go, predicate là `func(T) bool`. Trong SQL, mọi thứ sau `WHERE` là predicate trên từng dòng.

Predicate trở thành mệnh đề khi ta **lượng hóa (quantify)** nó trên một tập:

> **∀x ∈ S: P(x)** — "với mọi x trong S, P(x) đúng" (universal quantifier)
> **∃x ∈ S: P(x)** — "tồn tại ít nhất một x trong S sao cho P(x) đúng" (existential quantifier)

Lập trình viên dùng chúng hằng ngày mà không gọi tên:

| Toán | Go / SQL | Bản chất |
|---|---|---|
| ∀x: P(x) | `all(...)`, vòng lặp + return false sớm | AND trên cả tập |
| ∃x: P(x) | `any(...)`, `slices.ContainsFunc`, SQL `EXISTS` | OR trên cả tập |
| ¬∃x: P(x) | `NOT EXISTS` | phủ định của OR |

Và De Morgan tổng quát hóa lên quantifier một cách đẹp đẽ:

> **¬(∀x: P(x)) ≡ ∃x: ¬P(x)** — "không phải ai cũng đúng" = "có ít nhất một người sai"
> **¬(∃x: P(x)) ≡ ∀x: ¬P(x)** — "không ai đúng cả" = "tất cả đều sai"

Đây chính là lý do `NOT EXISTS` thường là cách *đúng* để viết query "khách hàng chưa từng đặt hàng" — nó là dạng ∀ được biểu diễn qua ¬∃, và không dính bẫy NULL của `NOT IN` (phân tích ở mục 7).

Một điểm tinh tế sống còn: **thứ tự quantifier quan trọng**. "∀ user ∃ server phục vụ user đó" (mỗi user có server nào đó — có thể khác nhau) khác hẳn "∃ server ∀ user" (một server duy nhất phục vụ tất cả). Trong thiết kế hệ thống, nhầm hai câu này là nhầm giữa "có replica" và "có single point of failure".

### Vacuous truth — sự thật rỗng

∀x ∈ ∅: P(x) — "mọi phần tử của tập rỗng đều thỏa P" — có giá trị gì? Toán học trả lời dứt khoát: **TRUE**, với mọi P. Vì để mệnh đề ∀ sai, phải *tồn tại* một phần tử vi phạm — tập rỗng không có phần tử nào để vi phạm.

Điều này không phải là quy ước tùy tiện mà là hệ quả bắt buộc để đại số nhất quán: ∀ là AND trên cả tập, và AND của không toán hạng nào phải là phần tử trung hòa của AND, tức TRUE (giống tổng của dãy rỗng là 0, tích của dãy rỗng là 1).

Code phản ánh trung thực điều này:

```go
// allPositive trả về true với slice RỖNG — và đó là hành vi đúng
func allPositive(nums []int) bool {
    for _, n := range nums { // slice rỗng: thân vòng lặp không chạy
        if n <= 0 {
            return false
        }
    }
    return true // vacuous truth
}
```

Hệ quả thực tế: "mọi test đều pass" khi **không có test nào** là true — CI xanh trên project chưa viết test. "Mọi replica đều ack" khi danh sách replica rỗng là true — quorum check viết ngây thơ sẽ cho commit khi cluster mất hết replica. Vacuous truth không phải bug của logic; **quên kiểm tra tập rỗng mới là bug của kỹ sư**. Khi viết điều kiện dạng ∀, luôn tự hỏi: tập rỗng thì hành vi này có phải điều tôi muốn không?

### Truth table — công cụ kiểm chứng cơ giới

Khi nghi ngờ hai điều kiện có tương đương không, đừng cãi nhau bằng lời: liệt kê. Với n biến có 2ⁿ dòng — n ≤ 4 thì làm tay được, và trong thực tế điều kiện if hiếm khi quá 4 biến độc lập:

```
p q | !(p && q) | !p || !q
T T |    F      |    F      ✓
T F |    T      |    T      ✓
F T |    T      |    T      ✓
F F |    T      |    T      ✓  → tương đương, refactor an toàn
```

Truth table cũng là nền của **exhaustive testing cho logic thuần**: một hàm chỉ nhận vài biến bool có thể được test *toàn bộ* không gian input — một trong những nơi hiếm hoi test thực sự chứng minh được tính đúng (vì nó duyệt hết mọi trường hợp, không phải lấy mẫu).

## 5. Thuật toán — đơn giản hóa điều kiện một cách có kỷ luật

Gặp một điều kiện phức tạp cần refactor, quy trình cơ giới gồm bốn bước:

**Bước 1 — Đặt tên mệnh đề.** Gán mỗi biểu thức con nguyên tử một chữ cái: `a = user.IsActive`, `b = user.IsBanned`, `c = req.IsInternal`. Điều kiện dài một dòng rưỡi giờ thành công thức 5 ký tự.

**Bước 2 — Đẩy phủ định vào trong** bằng De Morgan và khử phủ định kép, đến khi ¬ chỉ còn đứng cạnh biến.

**Bước 3 — Rút gọn bằng luật hấp thụ và phân phối:** p ∨ (p ∧ q) ≡ p; (p ∧ q) ∨ (p ∧ ¬q) ≡ p. Với biểu thức rối, viết truth table và đọc ra dạng tối giản (bài bản hơn có Karnaugh map / Quine–McCluskey — chương 09 Boolean Algebra).

**Bước 4 — Dịch ngược ra code, cân nhắc tách hàm đặt tên.**

```go
// Trước — điều kiện thật từ một codebase thật (đã đổi tên):
if !(order.Status != "cancelled" && (order.Total > 0 || !order.IsFree)) {
    return skip(order)
}

// Bước 1: a = (status == "cancelled"), b = (total > 0), c = order.IsFree
// Điều kiện: !( !a && (b || !c) )
// Bước 2 (De Morgan): a || !(b || !c) ≡ a || (!b && c)
// Bước 4: dịch ngược —
if order.Status == "cancelled" || (order.Total <= 0 && order.IsFree) {
    return skip(order)
}
// Đọc được thành lời: "đơn đã hủy, hoặc đơn miễn phí không có giá trị"
```

Lưu ý bước dịch `!b`: phủ định của `total > 0` là `total <= 0`, **không phải** `total < 0` — lỗi lật dấu so sánh thiếu dấu bằng là anh em song sinh của lỗi De Morgan.

### Short-circuit evaluation — logic gặp thứ tự thực thi

Go (như hầu hết ngôn ngữ) đánh giá `&&`/`||` kiểu **short-circuit**: `p && q` không đánh giá q nếu p đã false. Đây vừa là tính năng an toàn:

```go
// Idiom kinh điển: kiểm tra nil TRƯỚC khi dereference — thứ tự là sống còn
if user != nil && user.IsAdmin {
    ...
}
```

vừa là điểm khiến `&&` trong code **không hoàn toàn là ∧ trong toán**: ∧ giao hoán, nhưng `user.IsAdmin && user != nil` panic. Khi toán hạng có side effect hoặc có thể panic, thứ tự mang ngữ nghĩa. Bài học kép: (1) sắp toán hạng theo thứ tự guard trước — rẻ trước — đắt sau; (2) đừng giấu side effect trong biểu thức điều kiện, vì nó phá vỡ quyền tự do biến đổi đại số mà logic hứa hẹn.

## 6. Trade-off

**Điều kiện gộp vs guard clause.** Cùng một logic viết được hai kiểu: một biểu thức boolean lớn, hoặc chuỗi `if ... return` sớm. Biểu thức gộp gần đại số hơn — dễ kiểm chứng tương đương; guard clause dễ đọc tuần tự hơn và cho phép trả về lỗi *khác nhau* cho từng nhánh. Kinh nghiệm chung: logic quyết định thuần (cho ra bool) thì gộp và rút gọn; logic validate (cho ra error cụ thể) thì guard clause.

**Rút gọn tối đa vs giữ cấu trúc nghiệp vụ.** Dạng logic tối giản không phải lúc nào cũng là dạng dễ bảo trì. `(isVIP && overLimit) || (isVIP && flagged)` rút gọn được thành `isVIP && (overLimit || flagged)` — thường tốt. Nhưng đôi khi hai vế phản ánh hai *quy tắc nghiệp vụ độc lập* sẽ tiến hóa khác nhau; gộp chúng là tối ưu hóa sớm về mặt cấu trúc. Logic cho bạn *quyền* biến đổi; nghiệp vụ quyết định *có nên* không.

**Logic hai giá trị vs ba giá trị.** SQL chọn thêm UNKNOWN để mô hình hóa dữ liệu thiếu — cái giá là mọi luật quen thuộc phải xét lại (mục 7). Go chọn bool hai giá trị thuần và buộc bạn mô hình hóa "thiếu" tường minh bằng `*bool` hoặc `ok` pattern. Không hệ nào "đúng" hơn — nhưng **phải biết mình đang đứng trong hệ nào**, và cẩn trọng gấp đôi ở biên giới giữa hai hệ (mọi lớp ORM/driver mapping NULL ↔ Go đều là biên giới đó).

**Khi nào KHÔNG cần bộ máy hình thức?** Điều kiện 1–2 biến, đọc phát hiểu ngay — cứ viết tự nhiên. Bộ máy đặt tên mệnh đề, De Morgan, truth table chỉ đáng giá khi (a) có phủ định lồng ngoặc, (b) từ 3 biến trở lên, hoặc (c) bạn sắp *thay đổi* một điều kiện đang chạy đúng trong production. Dùng dao mổ trâu giết gà làm code review chậm đi vô ích.

## 7. Production Applications

**SQL three-valued logic và bug `NOT IN` kinh điển.** SQL có ba giá trị chân lý: TRUE, FALSE, UNKNOWN. Mọi phép so sánh với NULL đều cho UNKNOWN (`NULL = NULL` cũng là UNKNOWN — vì NULL nghĩa là "không biết", và "không biết có bằng không biết không" thì... không biết). `WHERE` chỉ giữ dòng có predicate TRUE — UNKNOWN bị loại như FALSE. Giờ mổ xẻ bug mở đầu chương:

```sql
id NOT IN (1, 2, NULL)
≡ NOT (id = 1 OR id = 2 OR id = NULL)
≡ NOT (id = 1 OR id = 2 OR UNKNOWN)
-- nếu id ∉ {1,2}: NOT (F OR F OR UNKNOWN) = NOT UNKNOWN = UNKNOWN → dòng bị loại!
```

De Morgan vẫn đúng trong logic ba giá trị — chính vì áp nó một cách máy móc ta *thấy được* UNKNOWN lan truyền qua phủ định. Cách chữa: `NOT EXISTS (SELECT 1 FROM orders o WHERE o.customer_id = c.id)` — EXISTS chỉ hỏi "có dòng nào TRUE không", UNKNOWN không lọt qua được. Đây là bug mà **hiểu logic thì tránh được trong 5 giây, không hiểu thì mất một buổi chiều với debugger**.

**Query optimizer rewrite điều kiện.** PostgreSQL biến đổi mệnh đề WHERE bằng chính các luật ở mục 3: đẩy phủ định vào trong, phân phối để tách `(a OR b) AND c` thành dạng cho phép dùng index trên từng nhánh, loại mệnh đề luôn-đúng, phát hiện mâu thuẫn `x > 5 AND x < 3` để trả rỗng không cần đọc đĩa (constraint exclusion — chính là phát hiện p ∧ ¬p ≡ FALSE). Predicate pushdown trong các engine phân tán (Spark, Trino) cũng là biến đổi logic được chứng minh bảo toàn tương đương: đẩy filter xuống gần dữ liệu chỉ hợp lệ vì các luật giao hoán/phân phối cho phép.

**Static analysis và compiler.** `go vet` và staticcheck phát hiện điều kiện hằng (`x != nil` sau khi đã dereference x), nhánh chết, biểu thức luôn true — bằng cách lan truyền các sự kiện logic qua luồng điều khiển. Type checker về bản chất là một prover nhỏ: chứng minh mệnh đề "không tồn tại đường thực thi nào gán string vào int". Rust borrow checker chứng minh mệnh đề ∀ về aliasing. Mức công nghiệp nặng: SMT solver (Z3) đứng sau symbolic execution và nhiều công cụ tìm lỗ hổng — chúng *giải* các công thức logic hàng triệu biến.

**Short-circuit trong Go runtime và API design.** Chuỗi middleware HTTP là một AND lười: auth fail thì handler sau không chạy. `ctx.Err() != nil` check đầu hàm là guard ¬p → thoát. Cả pattern `val, ok := m[key]; if ok && val.Ready {...}` — toàn bộ Go idiomatic code là logic mệnh đề được sắp thứ tự thực thi cẩn thận.

**Feature flag và cấu hình.** Hệ thống flag như LaunchDarkly đánh giá cây điều kiện (segment, percentage, prerequisite) — một biểu thức logic trên thuộc tính user. Flag lồng flag tạo implication ngầm: flag B chỉ có nghĩa khi flag A bật — quên mối quan hệ p → q này là nguồn của các tổ hợp cấu hình chưa từng được test (chương 02 sẽ đếm chính xác có bao nhiêu tổ hợp — gợi ý: 2ⁿ).

## 8. Interview

Logic hiếm khi là *câu hỏi* phỏng vấn trực tiếp, nhưng là *lý do trượt* phổ biến: điều kiện biên viết sai, vòng while có điều kiện dừng lật ngược, xử lý case rỗng sai.

**Các dạng phổ biến:**

- Viết điều kiện phức tạp đúng ngay lần đầu: Valid Parentheses (LeetCode 20), Valid Number (LeetCode 65 — bài mà bản chất là viết một predicate lớn không sai một nhánh).
- Điều kiện dừng vòng lặp: `lo < hi` hay `lo <= hi` trong binary search — trả lời chắc chắn được nhờ invariant (chương 03), nhưng phát biểu invariant đòi hỏi quantifier: "∀ i < lo: a[i] < target".
- Boolean logic trực tiếp: đôi khi được yêu cầu rút gọn biểu thức hoặc giải thích vì sao hai đoạn điều kiện tương đương/không tương đương.
- Bài đếm/kiểm tra trên tập có thể rỗng: hãy chú ý interviewer gần như *luôn* hỏi "nếu mảng rỗng thì sao?" — trả lời bằng vacuous truth một cách tự tin ("mọi phần tử của mảng rỗng đều thỏa điều kiện, nên trả về true/0/[] tùy ngữ nghĩa") là điểm cộng rõ rệt.

**Lỗi tư duy thường gặp:**

- Lật phủ định quên đổi AND↔OR, hoặc đổi `>` thành `<` mà quên trường hợp bằng.
- Khẳng định converse: "hàm trả true thì input hợp lệ" từ spec "input hợp lệ thì hàm trả true".
- Kiểm tra `if len(s) == 0` rồi xử lý riêng một cách không cần thiết — nhiều thuật toán đúng *tự nhiên* với input rỗng nhờ vacuous truth; thêm nhánh riêng vừa thừa vừa dễ sai.
- Viết `if cond { return true } else { return false }` — dấu hiệu chưa nhìn ra biểu thức bool là giá trị hạng nhất: `return cond`.

**Cách phân tích thay vì học thuộc:** với mọi điều kiện sắp viết, tự trả lời ba câu: (1) mệnh đề nguyên tử là gì — đặt tên được không; (2) tập rỗng/nil/zero value đi qua điều kiện này cho kết quả gì, có đúng ý không; (3) phủ định của điều kiện này — tức là nhánh else — phát biểu thành lời được không. Nếu câu (3) ấp úng, điều kiện đang quá phức tạp.

**Từ phỏng vấn sang production:** kỹ năng "phát biểu điều kiện thành lời" chính là kỹ năng viết PR description và định nghĩa alert rule. Một alert `error_rate > 1% AND NOT deploy_in_progress` là một mệnh đề — và alert nhiễu (false positive) thường là mệnh đề thiếu một liên từ.

## 9. Anti-pattern

**Phủ định lồng phủ định.** `if !notReady` — cờ đặt tên phủ định (`disabled`, `notFound`, `skipValidation`) buộc mọi chỗ dùng phải nhân phủ định trong đầu. Đặt tên mệnh đề ở **thể khẳng định** (`enabled`, `found`, `validate`), để dấu `!` xuất hiện tường minh và duy nhất tại chỗ dùng.

**So sánh với true/false.** `if isValid == true`, tệ hơn nữa `if isValid != false` — thừa và mở cửa cho lỗi ở ngôn ngữ có truthiness. Trong logic, p ≡ (p = true); viết vế phải là chưa tin bool là giá trị.

**Điều kiện trùng lặp lệch nhau.** Cùng một quy tắc nghiệp vụ ("đơn được hoàn tiền khi...") viết inline ở 5 nơi, rồi 4 nơi được sửa khi quy tắc đổi. Đây là vi phạm nguyên tắc mỗi mệnh đề nghiệp vụ nên có **một định nghĩa duy nhất** — một hàm predicate được đặt tên: `func (o *Order) Refundable() bool`.

**Tin rằng test pass nghĩa là logic đúng.** Điều kiện 4 biến có 16 tổ hợp; 3 unit test phủ 3 tổ hợp. Suy "3 pass → đúng cả 16" là affirming the consequent phiên bản thống kê. Với logic thuần ít biến: test hết bảng chân trị, hoặc property-based testing (chương 03).

**Mang trực giác hai giá trị vào vùng ba giá trị.** Mọi lần chạm SQL với cột nullable, `x <> 5` **không** bắt được dòng NULL; `SUM` trên tập rỗng trả NULL chứ không phải 0; hai NULL không bằng nhau trong `=` nhưng lại "bằng nhau" trong `DISTINCT` và `GROUP BY`. Không nhớ chi tiết cũng được — nhưng phải nhớ *rằng mình đang ở hệ logic khác* để tra cứu đúng lúc.

## 10. Best Practices

**Nên:**

- Đặt tên biến bool ở thể khẳng định; tách điều kiện ≥ 3 biến thành predicate có tên. Tên hàm tốt là bản dịch tiếng người của công thức logic.
- Áp De Morgan chủ động khi gặp `!(...)` trong review — biến đổi 10 giây, ngăn hiểu nhầm 10 ngày.
- Xếp toán hạng `&&`/`||` theo thứ tự: guard an toàn (nil check) → rẻ → đắt; không side effect trong điều kiện.
- Với mọi vòng lặp/hàm nhận collection: nghĩ về tập rỗng *một cách tường minh*, quyết định xem vacuous truth có phải hành vi mong muốn, và viết test cho case đó.
- Khi viết SQL trên cột nullable: mặc định dùng `NOT EXISTS` thay `NOT IN`, và tự hỏi "NULL đi qua predicate này thành gì?".

**Không nên:**

- Không refactor điều kiện production "theo cảm giác" — hoặc chứng minh tương đương (truth table 5 phút), hoặc đừng đụng.
- Không dùng converse trong lập luận debug: "deploy xong thì lỗi xuất hiện" không suy ra "lỗi do deploy" — nó chỉ là một giả thuyết cần kiểm chứng bằng contrapositive (rollback mà lỗi còn thì giả thuyết sập).
- Không trộn quy ước "hoặc loại trừ" của ngôn ngữ đời thường vào spec kỹ thuật — khi requirement nói "hoặc", xác nhận lại inclusive hay exclusive rồi mới viết code.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Vì sao `NOT IN` với subquery chứa NULL trả về 0 dòng, và dạng viết lại nào miễn nhiễm với vấn đề đó? Giải thích bằng cách lan truyền UNKNOWN qua De Morgan.
2. "Nếu cache hit thì latency < 5ms." Từ quan sát "latency = 20ms", bạn suy ra được gì, và từ "cache miss" bạn suy ra được gì? Cái nào là contrapositive, cái nào là bẫy inverse?
3. Hàm `allMatch(items, pred)` trả gì với `items` rỗng, vì sao đó là lựa chọn duy nhất nhất quán với việc ∀ là AND trên cả tập, và hãy kể một tình huống production mà hành vi này gây tai nạn nếu quên.

---

*Chương tiếp theo: [02 — Set Theory, Functions & Relations](/series/math-for-engineers/level-1-mathematical-thinking/02-set-theory-functions-relations/), nơi các predicate của chương này định nghĩa nên các tập hợp — và tập hợp hóa ra chính là ngôn ngữ nền của SQL, type system và sharding.*
