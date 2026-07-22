+++
title = "Chương 02 — Set Theory, Functions & Relations: Ngôn ngữ của dữ liệu"
date = "2026-07-20T07:20:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 1 – Mathematical Thinking
> Yêu cầu trước: Chương 01 (Logic)

---

## 1. Problem Statement

Một kỹ sư viết câu JOIN báo cáo doanh thu theo chi nhánh. Query chạy trên môi trường dev với dữ liệu mẫu: đúng. Đưa lên production: bảng kết quả **phình ra gấp 40 lần** số dòng thật, tổng doanh thu đội lên tương ứng, và báo cáo gửi CEO sáng hôm sau nói công ty vừa có quý tăng trưởng thần kỳ. Nguyên nhân: điều kiện JOIN thiếu một cột trong khóa ghép — mỗi dòng bảng trái khớp với *nhiều* dòng bảng phải, và kết quả trở thành một mảnh của **tích Descartes** thay vì phép khớp một-một mà tác giả tưởng tượng.

Cùng tuần đó, một kỹ sư khác debug tại sao hai user khác nhau thỉnh thoảng bị tính là một trong hệ thống analytics. Nguyên nhân: user được định danh bằng hash 32-bit của email. Hai email khác nhau, một hash — **collision**. Anh này coi đó là "bug hy hữu của thư viện hash". Không phải: đó là **định lý**. Một hàm từ tập vô hạn (chuỗi) vào tập hữu hạn (2³² giá trị) *không thể* injective — collision không phải rủi ro, mà là chắc chắn.

Hai sự cố, một gốc rễ:

**Dữ liệu là tập hợp, thao tác trên dữ liệu là phép toán trên tập hợp và ánh xạ giữa các tập — không nắm ngôn ngữ này, ta không dự đoán được hành vi của chính hệ thống mình viết.**

Set theory không phải kiến thức nền "cho biết". Nó là mô hình dữ liệu *theo nghĩa đen* của SQL (relational model của Codd năm 1970 xây thẳng trên lý thuyết tập hợp và logic vị từ), của type system, của partition trong Kafka.

## 2. Trực giác

### Tập hợp: bộ sưu tập không thứ tự, không trùng lặp

Một **tập hợp (Set)** là bộ sưu tập các phần tử phân biệt, không có thứ tự: {1, 2, 3} = {3, 1, 2} = {1, 1, 2, 3}. Chỉ có một câu hỏi có nghĩa với tập hợp: **x có thuộc S không?** (x ∈ S). Không có "phần tử đầu tiên", không có "phần tử xuất hiện hai lần".

Hai cách mô tả tập: liệt kê ({2, 4, 6, ...}) hoặc — quan trọng hơn nhiều — **bằng predicate** từ chương 01: {x ∈ ℕ | x chẵn}. Dạng thứ hai chính là câu `WHERE`: bảng `users` là một tập, `SELECT * FROM users WHERE age >= 18` là tập {u ∈ users | age(u) ≥ 18}. Mỗi predicate *khắc* ra một tập con; logic và tập hợp là hai mặt của cùng đồng xu: ∧ ứng với ∩, ∨ ứng với ∪, ¬ ứng với phần bù — De Morgan của chương 01 dịch nguyên văn sang tập hợp.

Trong Go không có kiểu Set built-in, và idiom chuẩn nói lên đúng bản chất "chỉ có câu hỏi thuộc/không thuộc":

```go
// Set trong Go: map với value rỗng — chỉ lưu sự kiện "có mặt"
seen := make(map[string]struct{})
seen["a"] = struct{}{}                 // thêm phần tử
_, ok := seen["a"]                     // câu hỏi duy nhất: a ∈ seen?
delete(seen, "a")                      // xóa phần tử
// struct{} chiếm 0 byte: ta không lưu giá trị nào cả, đúng tinh thần tập hợp
```

### Các phép toán — và bản dịch SQL của chúng

| Toán học | Ý nghĩa | SQL | Go (trên 2 map) |
|---|---|---|---|
| A ∪ B | thuộc A hoặc B | `UNION` | gộp key hai map |
| A ∩ B | thuộc cả hai | `INTERSECT` | duyệt map nhỏ, hỏi map lớn |
| A \ B | thuộc A, không thuộc B | `EXCEPT` | duyệt A, loại key có trong B |
| A × B | mọi cặp (a, b) | `CROSS JOIN` | hai vòng lặp lồng |
| A ⊆ B | mọi phần tử A đều trong B | `NOT EXISTS (A EXCEPT B)` | ∀a ∈ A: a ∈ B |

Một chi tiết hay bị bỏ qua: SQL mặc định làm việc trên **multiset (bag)** — bảng được phép có dòng trùng — và `UNION` (loại trùng, đúng nghĩa tập hợp) khác `UNION ALL` (giữ trùng, nghĩa multiset). Chọn sai giữa hai từ khóa này là chọn sai *mô hình toán*, và cái giá có thật ở cả hai chiều: `UNION` âm thầm nuốt dòng trùng hợp lệ (hai giao dịch giống hệt nhau!), còn thao tác loại trùng của nó tốn thêm một lần sort/hash toàn bộ kết quả.

### Cartesian product — vì sao JOIN thiếu điều kiện "nổ"

**Tích Descartes** A × B = {(a, b) | a ∈ A, b ∈ B} có đúng |A| × |B| phần tử. Về mặt hình thức, `JOIN ... ON cond` được *định nghĩa* là: lấy tích Descartes rồi lọc bằng predicate `cond`. Nghĩa là mọi JOIN đều khởi đầu (về mặt khái niệm) từ n×m cặp; điều kiện ON là thứ duy nhất kéo con số đó xuống. Điều kiện đúng trên khóa: mỗi dòng trái khớp ≤ 1 dòng phải, kết quả ≤ n dòng. Điều kiện thiếu một cột của khóa ghép: mỗi dòng trái khớp k dòng phải, kết quả n×k — chính là tai nạn mở đầu chương. Trực giác đúng cần nhớ: **JOIN không "ghép" hai bảng; JOIN lọc một tích**. Bảng kết quả phình to bất thường luôn có nghĩa là predicate lọc yếu hơn bạn tưởng.

## 3. First Principles

### Power set — đếm không gian trạng thái

**Power set** P(S) là tập của mọi tập con của S. Nếu |S| = n thì |P(S)| = 2ⁿ — vì mỗi phần tử độc lập chọn "có mặt/vắng mặt", n lựa chọn nhị phân nhân với nhau (lập luận đếm này được hệ thống hóa ở chương 05).

2ⁿ là con số quan trọng bậc nhất mà kỹ sư cần *cảm* được, vì nó là kích thước không gian cấu hình. Hệ thống có 10 feature flag boolean có 2¹⁰ = 1.024 tổ hợp trạng thái; 20 flag là hơn một triệu; 30 flag là một tỷ. Câu "chúng ta đã test kỹ mọi cấu hình" trở thành bất khả thi *về mặt số học* từ rất sớm — và mọi tổ hợp chưa từng chạy là một hành vi chưa từng được quan sát. Đây là lý do các team flag trưởng thành đặt vòng đời cho flag (xóa sau khi rollout xong): không phải vì gọn code, mà vì **mỗi flag xóa đi giảm một nửa không gian trạng thái**.

Bitmask — kỹ thuật quen thuộc trong LeetCode và trong cờ trạng thái (`O_RDONLY|O_APPEND`) — chính là power set được mã hóa: mỗi bit là một phần tử, mỗi giá trị uint là một tập con, phép OR là ∪, AND là ∩, AND NOT là \.

### Function — ánh xạ có kỷ luật

Một **function** f: A → B gán cho *mỗi* phần tử của A đúng *một* phần tử của B. Hai chữ in nghiêng là toàn bộ kỷ luật: không phần tử nào của A bị bỏ sót (total), không phần tử nào cho hai kết quả (deterministic). `map[K]V` trong Go là một function từ tập key sang value (partial — chỉ định nghĩa trên key đã gán). Một hàm Go thuần (không side effect) là function theo nghĩa toán; hàm đọc đồng hồ hay random thì không — và chính vì thế khó test, khó cache, khó suy luận: **mọi lợi ích của pure function đều bắt nguồn từ việc nó là function thật sự**.

Ba tính chất định hình mọi thảo luận về hash, encoding, ID:

| Tính chất | Định nghĩa | Nghĩa là |
|---|---|---|
| **Injective** (đơn ánh) | a₁ ≠ a₂ ⇒ f(a₁) ≠ f(a₂) | không hai input nào chung output |
| **Surjective** (toàn ánh) | ∀b ∈ B, ∃a: f(a) = b | mọi output đều đạt được |
| **Bijective** (song ánh) | cả hai | ghép cặp hoàn hảo — **tồn tại hàm ngược f⁻¹** |

Từ định nghĩa injective, một hệ quả bằng đếm đơn thuần: **nếu |A| > |B| thì f: A → B không thể injective** (nguyên lý Pigeonhole — chương 05 chứng minh và khai thác sâu). Áp vào hash function: tập input là chuỗi bất kỳ (vô hạn), tập output là 2⁶⁴ hay 2²⁵⁶ giá trị (hữu hạn) → **collision tồn tại là định lý, không phải lỗi thư viện**. Toàn bộ kỹ nghệ hashing (chương 12) không phải là "tránh collision" — bất khả — mà là làm collision *hiếm và khó tìm*.

Ngược lại, encoding đúng nghĩa phải **bijective**: `encode` có `decode` khi và chỉ khi ánh xạ là song ánh. Base64 là bijection giữa byte và tập con của chuỗi ASCII — decode được. Hash không có inverse — "decode hash" là câu vô nghĩa về mặt toán học (bảng "giải mã MD5" trên mạng chỉ là tra cứu ngược từ điển các input đã biết). Khi thiết kế serialization, câu hỏi "hai giá trị khác nhau có thể cho cùng chuỗi không, và có chuỗi nào không giải mã được không" chính là hỏi injective/surjective — JSON của Go thất bại tính injective ngay với `map` chứa key kiểu khác nhau hay với `NaN`, và đó là nguồn của các bug so sánh-qua-serialize.

### Relation — khi ánh xạ không đủ

Không phải mối liên hệ nào cũng là function: "user A follow user B" cho phép một user follow nhiều người. Tổng quát hóa: một **relation** trên A là một tập con của A × A — đơn giản là *tập các cặp có liên hệ*. Bảng `follows(follower_id, followee_id)` là relation theo nghĩa đen; chữ R trong RDBMS — relational — nghĩa là "thuộc về relation", không phải "có quan hệ khóa ngoại" như dân gian vẫn hiểu.

Ba tính chất then chốt của relation R trên A:

- **Reflexive**: a R a với mọi a (ai cũng liên hệ với chính mình)
- **Symmetric**: a R b ⇒ b R a (liên hệ hai chiều)
- **Transitive**: a R b và b R c ⇒ a R c (liên hệ bắc cầu)

"Follow" trên mạng xã hội: không symmetric (Twitter/X) hoặc symmetric (kết bạn Facebook) — một bit tính chất toán học quyết định cả schema lẫn product. "Gọi hàm" trong codebase: không symmetric, và bao đóng transitive của nó (transitive closure) chính là câu trả lời cho "sửa hàm này ảnh hưởng những đâu".

## 4. Mathematical Model

### Equivalence relation — toán học của sharding

Relation có đủ cả ba tính chất reflexive + symmetric + transitive gọi là **equivalence relation** (quan hệ tương đương) — cách toán học nói "giống nhau theo một tiêu chí nào đó": cùng số dư khi chia 16, cùng quốc gia, cùng hash bucket.

Định lý nền tảng (chứng minh ngắn đến bất ngờ):

> Mọi equivalence relation trên S **chia S thành các lớp tương đương rời nhau, phủ kín S** — một **partition**. Ngược lại, mọi partition xác định một equivalence relation.

Chứng minh phần thuận: đặt [a] = {x | x R a} (lớp của a). Reflexive ⇒ a ∈ [a] ⇒ các lớp phủ kín S. Còn phải chứng minh hai lớp hoặc trùng nhau hoặc rời nhau: giả sử [a] ∩ [b] ≠ ∅, lấy c chung. Với x ∈ [a] bất kỳ: x R a, a R c (symmetric từ c R a... chính xác hơn: c ∈ [a] nghĩa là c R a, symmetric cho a R c), c R b (c ∈ [b]), transitive hai lần cho x R b, tức x ∈ [b]. Vậy [a] ⊆ [b], đối xứng cho chiều ngược: [a] = [b]. ∎

Ba dòng chứng minh, nhưng hệ quả kỹ thuật rất nặng ký: **sharding/partitioning đúng đắn khi và chỉ khi tiêu chí chia là một equivalence relation**. `hash(key) mod N` là equivalence relation (thừa hưởng từ "bằng nhau sau khi áp hàm") — nên mỗi key thuộc đúng một shard, không key nào bị bỏ rơi, không key nào nằm hai nơi. Đây chính xác là ba tính chất mà định lý bảo đảm: phủ kín + rời nhau. Khi ai đó đề xuất chia dữ liệu theo tiêu chí *không* transitive (ví dụ "hai user gần nhau về địa lý vào cùng shard" — gần không bắc cầu: A gần B, B gần C, A có thể xa C), hệ thống sẽ hoặc nhân bản dữ liệu hoặc mất dữ liệu ở biên — định lý nói trước điều đó, không cần chạy thử.

`GROUP BY` trong SQL là partition theo equivalence "bằng nhau trên các cột nhóm". Union-Find (chương 24) là cấu trúc dữ liệu chuyên trị equivalence relation động. Consistent hashing (chương 25) là câu chuyện điều gì xảy ra khi N thay đổi và partition phải dịch chuyển ít nhất.

### Partial order — toán học của dependency

Đổi symmetric lấy **antisymmetric** (a R b và b R a ⇒ a = b), ta được **partial order** (quan hệ thứ tự bộ phận): reflexive + antisymmetric + transitive. Nguyên mẫu: ⊆ trên các tập, "chia hết" trên số nguyên, và quan trọng nhất với kỹ sư: **"phụ thuộc vào" trên các package/task/migration**.

Chữ "partial" là điểm tinh tế: không phải cặp nào cũng so sánh được. Package A và B có thể chẳng cái nào phụ thuộc cái nào — chúng **incomparable**, và điều đó hợp lệ. So với **total order** (mọi cặp so sánh được — như ≤ trên số), partial order chứa ít thông tin hơn nhưng trung thực hơn với thực tế: ép một total order lên đồ thị dependency (như cách con người xếp danh sách việc tuần tự) là *thêm* ràng buộc không tồn tại — và đánh mất cơ hội chạy song song. Trực giác đáng nhớ: **những cặp incomparable chính là những việc parallel hóa được**. Điều kiện để một relation dependency là partial order hợp lệ nằm ở transitivity + antisymmetry: có chu trình phụ thuộc (A → B → A với A ≠ B) là vi phạm antisymmetry — và mọi package manager đều phải từ chối. Topological sort (chương 08) là thuật toán nhúng một partial order vào một total order để thực thi tuần tự; số cách nhúng nhiều hơn 1 chính là mức độ tự do song song.

### Type như tập hợp giá trị

Một type là **tập các giá trị hợp lệ**: `bool` = {true, false}, `uint8` = {0..255}, `string` = tập vô hạn đếm được. Nhìn qua lăng kính này, nhiều khái niệm type system trở nên trong suốt: subtype là ⊆; union type (Rust enum, TS `A | B`) là ∪ có gắn nhãn; struct là tích Descartes (nên gọi là *product type* — `struct{A bool; B uint8}` có đúng 2 × 256 giá trị); generic constraint của Go (`~int | ~float64`) mô tả một tập các type. Nguyên tắc thiết kế "make illegal states unrepresentable" dịch ra ngôn ngữ tập hợp: **chọn type sao cho tập giá trị của nó = đúng tập trạng thái hợp lệ**, không thừa phần tử nào. Type `string` cho email có tập giá trị thừa gần như 100% — mọi chuỗi không phải email; type `Email` với constructor validate thu tập đó về đúng kích thước.

## 5. Thuật toán — phép toán tập hợp hiệu quả trong code

Bài toán nền: giao hai tập biểu diễn bằng slice. Ba chiến lược, ba độ phức tạp:

```go
// Chiến lược 1: lồng hai vòng — O(n·m). Đúng, và là quả bom hẹn giờ (chương 10).

// Chiến lược 2: hash set — O(n + m) thời gian, O(min(n,m)) bộ nhớ thêm
func intersect(a, b []int) []int {
    if len(a) > len(b) {
        a, b = b, a // build set trên tập NHỎ hơn — tiết kiệm bộ nhớ
    }
    set := make(map[int]struct{}, len(a))
    for _, x := range a {
        set[x] = struct{}{}
    }
    var out []int
    for _, x := range b {
        if _, ok := set[x]; ok {
            out = append(out, x)
            delete(set, x) // giữ ngữ nghĩa TẬP HỢP: mỗi phần tử ra một lần
        }
    }
    return out
}

// Chiến lược 3: nếu cả hai ĐÃ sort — hai con trỏ, O(n + m), KHÔNG tốn bộ nhớ thêm
func intersectSorted(a, b []int) []int {
    var out []int
    i, j := 0, 0
    for i < len(a) && j < len(b) {
        switch {
        case a[i] < b[j]:
            i++
        case a[i] > b[j]:
            j++
        default: // bằng nhau: thuộc giao
            out = append(out, a[i])
            i, j = i+1, j+1
        }
    }
    return out
}
```

Dòng `delete` trong chiến lược 2 đáng một phút suy nghĩ: nó là chỗ ta *chọn mô hình*. Bỏ nó đi, hàm trả kết quả theo ngữ nghĩa khác (phần tử của b có mặt trong a — multiset một phía). Bug loại này không phải bug thuật toán mà là **bug đặc tả**: chưa quyết định mình đang làm việc với set hay bag. SQL bắt bạn quyết định bằng `UNION` vs `UNION ALL`; Go bắt bạn quyết định bằng một dòng `delete`.

Chiến lược 3 chính là **merge join** của database, chiến lược 2 là **hash join** — hai trong ba thuật toán join vật lý của PostgreSQL là hai cách giao tập ở trên, nguyên xi. Chọn giữa chúng (dữ liệu đã sort theo index chưa? đủ RAM build hash table không?) là việc query planner làm mỗi query — và là câu hỏi system design bạn nên trả lời được từ chương này.

## 6. Trade-off

**Set vs slice — ngữ nghĩa đổi lấy thứ tự và bộ nhớ.** Chuyển slice sang map-set được membership O(1) nhưng **mất thứ tự và mất bản sao** — nếu nghiệp vụ cần "danh sách theo thời gian thêm vào, có thể trùng", set là mô hình *sai* dù nhanh hơn. Chọn cấu trúc là chọn ngữ nghĩa trước, hiệu năng sau. Chưa kể với n nhỏ (< ~50), slice + linear scan thắng map về hằng số và cache locality — bài học chương 10 lặp lại.

**Biểu diễn tường minh vs biểu diễn bằng predicate.** Tập "user bị chặn" lưu tường minh (bảng, set trong Redis) thì membership rẻ nhưng phải bảo trì khi thế giới đổi; lưu bằng predicate (`age < 13 || flagged`) thì luôn tươi mới nhưng mỗi câu hỏi membership tốn một lần đánh giá và không đếm được kích thước nếu không quét. Materialized view là điểm giữa: predicate được "đóng băng" thành tập tường minh, đổi độ tươi lấy tốc độ — toàn bộ cuộc chơi caching nằm trong trade-off này.

**Bijective ID vs hash ID.** ID sinh bijective từ dữ liệu nguồn (auto-increment, hoặc encode trực tiếp) đảo ngược được, gọn — nhưng lộ thông tin (đếm được số đơn hàng của đối thủ qua ID tăng dần — "German tank problem") và đòi hỏi điều phối tập trung. Hash/UUID không cần điều phối, không lộ thứ tự — nhưng không đảo ngược và phải sống chung với xác suất collision (nhỏ đến mức chấp nhận được với 128 bit — chương 12 tính chính xác "nhỏ" là bao nhiêu qua birthday paradox).

**Khi nào KHÔNG nên nghĩ bằng tập hợp?** Khi thứ tự và sự lặp là *bản chất* của dữ liệu: log sự kiện, time series, văn bản. Ép chúng vào mô hình tập (ví dụ dedupe log theo nội dung) là mất thông tin không lấy lại được. Câu hỏi kiểm tra: "hai phần tử giống hệt nhau có phải là một không?" và "đổi chỗ hai phần tử có làm dữ liệu sai không?" — trả lời "không phải là một" hoặc "có sai" thì bạn đang cầm sequence/multiset, đừng cầm set.

## 7. Production Applications

**SQL engine — set theory chạy bằng điện.** Relational algebra của Codd gồm đúng các phép toán chương này: selection (σ — predicate lọc tập), projection (π — ánh xạ), tích Descartes, ∪, \, và join định nghĩa từ tích + lọc. Query optimizer biến đổi biểu thức đại số quan hệ bằng các đẳng thức tập hợp: σ đẩy xuống dưới join (predicate pushdown hợp lệ vì σ_p(A ⋈ B) = σ_p(A) ⋈ B khi p chỉ chạm cột của A), join giao hoán/kết hợp cho phép đảo thứ tự join — cả ngành tối ưu query là **đại số tập hợp ứng dụng**. `EXPLAIN` một query nhiều JOIN và bạn thấy planner đang giải bài toán: thứ tự nào cho tích trung gian nhỏ nhất.

**Kafka partition — equivalence relation ở tầng hạ tầng.** Producer gửi message vào partition theo `hash(key) mod numPartitions`: một equivalence relation trên key. Vì các lớp tương đương rời nhau và message cùng key luôn cùng lớp, Kafka **giữ được thứ tự trong từng key** mà vẫn song song hóa giữa các key — toàn bộ mô hình ordering của Kafka đứng trên định lý partition ở mục 4. Và điểm đau nổi tiếng: đổi `numPartitions` là đổi equivalence relation — các lớp cũ vỡ ra, key nhảy partition, thứ tự lịch sử mất. Hiểu bằng toán, bạn biết *trước* rằng "tăng partition cho topic đang chạy" là quyết định phá vỡ bất biến, không phải thao tác vô hại.

**Go map và type system.** `map[K]V` yêu cầu K comparable — vì map là function cần hỏi "hai key có *bằng nhau* không", và bằng nhau phải là equivalence relation. Đây là lý do slice không làm key được (Go từ chối định nghĩa bằng nhau cho slice — theo giá trị hay theo con trỏ đều gây bất ngờ cho một nửa người dùng), còn struct làm được khi mọi field comparable (bằng nhau theo từng thành phần của tích Descartes). Interface satisfaction của Go cũng là quan hệ ⊆: type T thỏa interface I khi tập method của T ⊇ tập method của I.

**Redis Sets.** `SADD`, `SINTER`, `SUNION`, `SDIFF` — phép toán tập hợp là API công khai. Bài toán "gợi ý bạn chung" của mạng xã hội là một dòng: `SINTERCARD friends:alice friends:bob`. Tag-based filtering ("bài viết có tag A và B nhưng không C") là `SINTERSTORE` + `SDIFF`. Điều đáng học từ Redis không phải lệnh, mà là quyết định thiết kế: **đưa đại số tập hợp lên làm giao diện**, để client tổ hợp thay vì tự cài đặt.

**Package manager và scheduler — partial order vận hành.** `go mod`, npm, apt đều làm việc trên partial order "phụ thuộc": phát hiện chu trình (vi phạm antisymmetry ⇒ từ chối), topological sort để cài đúng thứ tự. Kubernetes init container, Airflow DAG, `make` — cùng một bài: thực thi một partial order, song song hóa các phần tử incomparable. Khi CI pipeline của bạn chạy chậm, câu hỏi toán học là: chuỗi phụ thuộc dài nhất (critical path — chính là *chain* dài nhất trong partial order) là bao nhiêu — vì đó là chặn dưới của thời gian chạy dù có vô hạn worker.

## 8. Interview

**Các dạng phổ biến:**

- **Intersection of Two Arrays (LeetCode 349, 350):** bài 349 là giao *tập* (kết quả không trùng), bài 350 là giao *multiset* (giữ số lần xuất hiện min của hai bên). Hai bài liền số nhau tồn tại chính là để kiểm tra bạn có phân biệt set/bag — mô hình trước, code sau. Bản 350 dùng `map[int]int` (multiset = function từ phần tử sang số lần) thay vì `map[int]struct{}`.
- **Isomorphic Strings (LeetCode 205):** "egg" và "add" isomorphic vì tồn tại **bijection** giữa bộ ký tự hai chuỗi bảo toàn cấu trúc. Lỗi phổ biến nhất: chỉ check một chiều (function, có thể nhiều-về-một) — "badc"/"baba" lọt qua. Phải check cả injective (hai ký tự nguồn không map vào một đích) — tức là hai map hai chiều, hoặc một map + một set các đích đã dùng. Ai phát biểu được "đề đang hỏi ánh xạ có bijective không" gần như chắc chắn code đúng ngay lần đầu.
- **Contains Duplicate (217), Happy Number (202), Longest Consecutive Sequence (128):** cùng một ý — dùng set để nhớ "đã thấy", đổi O(n²) hoặc vòng lặp vô hạn lấy O(n). Bài 202 tinh tế hơn: set phát hiện *chu trình* trong quỹ đạo của một function — mô hình "lặp một ánh xạ đến khi quay lại điểm cũ" xuất hiện lại trong Floyd cycle detection.
- **Group Anagrams (49):** partition theo equivalence relation "cùng multiset ký tự"; key của map chính là *đại diện chuẩn tắc* của lớp tương đương (chuỗi đã sort). Nhận ra khuôn "tìm equivalence relation + chọn canonical form làm key" giải được cả họ bài group-by.

**Lỗi tư duy thường gặp:**

- Không hỏi/không để ý input có phần tử trùng không, output được phép trùng không — nhảy vào code khi mô hình (set hay bag?) chưa chốt.
- Dùng map khi cần cả thứ tự — rồi vá bằng sort cuối, trong khi đề cho sẵn dữ liệu sort (hai con trỏ đẹp hơn hẳn).
- Với bài ánh xạ: mặc định "có map là xong" mà không hỏi map cần tính chất gì (injective? đảo được?).

**Cách phân tích thay vì học thuộc:** trước khi nghĩ thuật toán, chốt mô hình bằng bốn câu: (1) đây là set, multiset hay sequence? (2) liên hệ giữa các phần tử là function hay relation tổng quát? (3) nếu là function — cần injective/bijective không? (4) nếu là relation — nó có phải equivalence (bài group) hay partial order (bài dependency/schedule) không? Bốn câu này phân loại được phần lớn bài dùng hash map trên LeetCode — và cũng chính là bốn câu mở đầu buổi system design.

**Từ phỏng vấn sang production:** Two-pointer trên hai mảng sort chính là merge join; group anagrams chính là `GROUP BY` với biểu thức nhóm tự chế; isomorphic strings chính là bài kiểm tra một encoding có decode được không. Người thấy được các phép chiếu này không cần học hai lần.

## 9. Anti-pattern

**JOIN như phép "ghép" thay vì phép lọc tích.** Viết JOIN mà không tự hỏi "quan hệ hai bảng là 1-1, 1-n hay n-m qua điều kiện ON này" — rồi ngạc nhiên khi kết quả phình. Kiểm tra rẻ nhất: `COUNT(*)` trước và sau join; kết quả lớn hơn bảng trái trong join 1-1 kỳ vọng là chuông báo cháy.

**Dedupe hồn nhiên.** Rắc `DISTINCT` lên query hoặc đổ slice vào set để "sửa" dòng trùng mà không hỏi *vì sao* trùng. Dòng trùng thường là triệu chứng của join sai hoặc mô hình sai; `DISTINCT` là thuốc giảm đau che khối u — và còn tốn một lần sort/hash toàn kết quả.

**Coi collision là bug hy hữu.** Thiết kế hệ thống nơi hash collision gây sai dữ liệu (thay vì chỉ giảm hiệu năng) — ví dụ dùng hash làm khóa chính không kèm so sánh giá trị gốc. Với hash 32-bit, chỉ ~77.000 phần tử là xác suất có collision vượt 50% (birthday paradox — chương 12). Định lý "không injective" không thương lượng; thiết kế phải sống chung với nó.

**ID có ngữ nghĩa.** Nhét thông tin vào ID (mã vùng trong số nhân viên, ngày trong mã đơn) — tức là ép ID làm một function có ngữ nghĩa thay vì chỉ định danh. Ngày thế giới đổi (nhân viên chuyển vùng), hoặc ánh xạ mất tính injective (hai chi nhánh gộp), toàn bộ hệ thống hạ nguồn đã parse ID vỡ theo. ID tốt là ID *không là function của bất cứ thứ gì* ngoài sự kiện cấp phát.

**Chia shard theo tiêu chí không phải equivalence relation.** Đã phân tích ở mục 4 — mọi tiêu chí "gần nhau", "tương tự nhau" (không transitive) đều không chia tập sạch sẽ được. Muốn shard theo tương tự, phải *chuẩn hóa* thành equivalence thật (ví dụ: cùng geohash prefix — bằng nhau trên prefix là equivalence) và chấp nhận sai số ở biên ô.

## 10. Best Practices

**Nên:**

- Chốt mô hình trước khi chốt cấu trúc dữ liệu: set / multiset / sequence — ba câu trả lời cho "trùng lặp có nghĩa gì" và "thứ tự có nghĩa gì".
- Dùng `map[T]struct{}` cho set trong Go (ngữ nghĩa rõ, 0 byte value); `map[T]int` khi cần multiset; build set trên tập nhỏ hơn khi giao hai tập.
- Với mỗi JOIN viết ra: gọi tên lực số (cardinality) kỳ vọng — 1-1, 1-n, n-m — và biết `UNION` hay `UNION ALL` là quyết định ngữ nghĩa, không phải phong cách.
- Với mỗi hàm sinh ID/encode/serialize: hỏi tường minh "cần injective không, cần đảo ngược không" và ghi câu trả lời vào doc comment.
- Với mỗi tiêu chí nhóm/chia/shard: kiểm ba tính chất reflexive–symmetric–transitive trong đầu; thiếu một cái là thiết kế có lỗ hổng cấu trúc.

**Không nên:**

- Không dùng `DISTINCT`/dedupe làm băng cứu thương khi chưa tìm ra nguồn sinh trùng lặp.
- Không giả định hai tập "chắc là rời nhau" — kiểm tra ∩ rỗng rẻ hơn nhiều so với dọn hậu quả ghi đè.
- Không ép total order lên thứ tồn tại tự nhiên là partial order (xếp task tuần tự khi chúng độc lập) — mất song song miễn phí; và ngược lại, không giả định thứ tự toàn cục ở nơi hệ thống chỉ hứa thứ tự trong từng partition (Kafka!).

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Vì sao hash function không thể injective, và hệ quả thiết kế nào bắt buộc phải theo sau sự thật đó (gợi ý: điều gì phải xảy ra khi hai key đụng nhau trong hash table, trong content-addressable storage)?
2. Chứng minh hoặc bác bỏ: "cùng năm sinh" là equivalence relation trên tập user; "chênh nhau không quá 5 tuổi" thì sao? Từ đó giải thích vì sao cái trước shard được còn cái sau không.
3. Query JOIN của bạn trả về nhiều dòng hơn bảng lớn nhất tham gia join. Dùng ngôn ngữ tích Descartes và predicate, mô tả chính xác điều gì đang xảy ra và ba bước chẩn đoán.

---

*Chương tiếp theo: [03 — Proof Techniques](/series/math-for-engineers/level-1-mathematical-thinking/03-proof-techniques/), nơi ta ngừng hỏi "mô hình là gì" và bắt đầu hỏi "làm sao biết chắc code đúng" — bằng induction, contradiction, và loop invariant.*
