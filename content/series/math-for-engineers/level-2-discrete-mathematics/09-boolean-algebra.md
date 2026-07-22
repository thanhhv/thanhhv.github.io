+++
title = "Chương 09 — Boolean Algebra: Đại số của bật và tắt"
date = "2026-07-20T08:30:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 2 – Discrete Mathematics
> Yêu cầu trước: Chương 01 (Logic), Chương 02 (Set Theory)

---

## 1. Problem Statement

Một team analytics cần trả lời câu hỏi tưởng như tầm thường: "hôm qua có bao nhiêu user active, và bao nhiêu user active cả 7 ngày liên tiếp?" Hệ thống có 100 triệu user. Giải pháp đầu tiên: bảng `daily_active(user_id, date)`, mỗi ngày ghi vài chục triệu dòng, câu hỏi "active 7 ngày liên tiếp" trở thành JOIN 7 bảng con — chạy hàng phút, tốn hàng chục GB.

Giải pháp thứ hai gói gọn trong một cấu trúc bé đến khó tin: mỗi ngày một chuỗi **100 triệu bit** (12.5MB), bit thứ i bật nếu user i active. "Active hôm qua" = `BITCOUNT` — đếm bit 1. "Active cả 7 ngày" = AND 7 chuỗi bit rồi đếm. Redis làm việc này trong vài chục millisecond, và toàn bộ 7 ngày dữ liệu chiếm chưa đến 90MB.

Điều gì khiến 12.5MB làm được việc của hàng chục GB? Không phải phép màu nén dữ liệu, mà là một sự trùng khớp toán học: **phép giao tập hợp và phép AND trên bit là cùng một phép toán**, và CPU thực hiện AND trên 64 bit song song trong một chu kỳ. Ta đang mượn sức của đại số.

Đại số đó — Boolean algebra, do George Boole xây dựng năm 1854 để "toán học hóa tư duy" — hóa ra là nền móng theo nghĩa đen của máy tính: mọi mạch trong CPU là tổ hợp AND/OR/NOT, mọi phép cộng nhân là mạch logic, mọi `if` là biểu thức Boole, mọi bitmap index là đại số tập hợp chạy bằng phần cứng.

Bài toán của chương này: **hiểu hệ đại số hai phần tử {0, 1} như một hệ tiên đề hoàn chỉnh — và khai thác việc CPU thực thi hệ đó ở tốc độ phần cứng, 64 phần tử mỗi chu kỳ.**

Nếu không có góc nhìn này, bit manipulation mãi là bộ sưu tập "trick ma thuật" chép từ Stack Overflow — dùng được nhưng không dám sửa, và không bao giờ tự nghĩ ra trick tiếp theo.

## 2. Trực giác

### Đại số của thế giới chỉ có hai giá trị

Đại số quen thuộc làm việc trên số: cộng, nhân, các luật giao hoán, kết hợp, phân phối. Boole đặt câu hỏi: nếu thế giới chỉ có hai giá trị — đúng/sai, bật/tắt, có dòng điện/không — thì "cộng" và "nhân" là gì, và các luật nào còn đúng?

Câu trả lời: AND đóng vai phép nhân (1·1 = 1, mọi thứ khác ra 0 — giống hệt nhân số), OR đóng vai phép cộng (có ít nhất một số 1 thì ra 1), NOT là phép lật. Và điều bất ngờ: hệ này *giàu luật hơn* đại số số học — nó có những đẳng thức mà số học không có, như x·x = x (AND với chính mình không đổi) hay luật De Morgan.

Điểm trực giác quan trọng nhất: **cùng một đại số này xuất hiện ở ba lớp áo khác nhau**, và bạn đã gặp cả ba:

| Lớp áo | "Nhân" (AND) | "Cộng" (OR) | "Phủ định" (NOT) | Đã gặp ở |
|---|---|---|---|---|
| Logic mệnh đề | p ∧ q | p ∨ q | ¬p | Chương 01 |
| Tập hợp | A ∩ B | A ∪ B | phần bù Aᶜ | Chương 02 |
| Bit | `a & b` | `a \| b` | `^a` (Go) | chương này |

Ba lớp áo, một cơ thể. Đây là lý do một identity chứng minh ở lớp logic tự động đúng ở lớp bit — và là lý do bitmap "cài đặt" được phép giao tập hợp: nó không *mô phỏng* phép giao, nó *là* phép giao, viết trong hệ ký hiệu khác.

### Vì sao bit lại nhanh đến thế?

Một số `uint64` là 64 giá trị Boole xếp cạnh nhau, và lệnh `AND` của CPU xử lý cả 64 vị trí **trong một chu kỳ, song song, không rẽ nhánh**. So với vòng lặp 64 lần `if`: không có branch misprediction, không có bộ nhớ rải rác, chỉ một lệnh. Đây là hình thức song song rẻ nhất trong máy tính — có sẵn, miễn phí, trước cả khi nói đến SIMD hay goroutine. Bitmap của Redis trong bài toán mở đầu chạy nhanh chính vì nó biến "100 triệu phép giao logic" thành "1.5 triệu lệnh AND 64-bit" — và các lệnh này đọc bộ nhớ tuần tự, thứ prefetcher yêu thích (chương 10).

## 3. First Principles

### Xây từ tiên đề

Boolean algebra là một hệ tiên đề: tập {0, 1} với ba phép toán, và một nhóm luật cơ bản mà mọi đẳng thức khác suy ra được. Các luật đáng nhớ nhất — mỗi luật đều "hiển nhiên" khi đọc bằng lớp áo tập hợp hoặc logic:

| Luật | Dạng AND | Dạng OR |
|---|---|---|
| Identity | x ∧ 1 = x | x ∨ 0 = x |
| Annihilator | x ∧ 0 = 0 | x ∨ 1 = 1 |
| Idempotent | x ∧ x = x | x ∨ x = x |
| Complement | x ∧ ¬x = 0 | x ∨ ¬x = 1 |
| Commutative / Associative | như số học | như số học |
| **Distributive** | x ∧ (y ∨ z) = (x∧y) ∨ (x∧z) | x ∨ (y ∧ z) = (x∨y) ∧ (x∨z) |
| **De Morgan** | ¬(x ∧ y) = ¬x ∨ ¬y | ¬(x ∨ y) = ¬x ∧ ¬y |

Hai điều đáng dừng lại. Thứ nhất, luật phân phối đúng **cả hai chiều** — OR phân phối qua AND, điều *không có* trong số học (a + b·c ≠ (a+b)·(a+c)). Boolean algebra đối xứng một cách hoàn hảo: mọi định lý vẫn đúng nếu tráo AND↔OR và 0↔1 cùng lúc (nguyên lý duality). Thứ hai, De Morgan là luật "đắt giá" nhất trong thực hành: nó cho phép đẩy phủ định qua ngoặc, biến `!(a && b)` thành `!a || !b` — nền tảng của việc refactor điều kiện, của query optimizer khi chuẩn hóa mệnh đề WHERE, và của việc xây mọi mạch chỉ từ cổng NAND (chương 01 đã chứng minh NAND là universal).

Vì sao hệ tiên đề đáng quan tâm với engineer? Vì nó cho **quyền biến đổi máy móc mà vẫn bảo toàn ngữ nghĩa**. Khi bạn đơn giản hóa điều kiện `if` rối rắm, bạn đang làm đại số; nếu mỗi bước là một luật hợp lệ, kết quả *chắc chắn* tương đương — không cần test lại mọi tổ hợp. Compiler và query optimizer làm chính xác điều này, hàng triệu lần mỗi giây.

### Đơn giản hóa: từ đại số đến Karnaugh map

Biểu thức Boole nào cũng có vô số dạng tương đương; dạng ngắn hơn nghĩa là ít cổng logic hơn (phần cứng), ít phép so sánh hơn (phần mềm). Đơn giản hóa bằng biến đổi đại số thuần túy đòi hỏi "nhìn ra" bước tiếp theo. **Karnaugh map** là công cụ trực quan hóa cho ≤ 4 biến: kẻ bảng chân trị thành lưới sao cho hai ô kề nhau chỉ khác đúng một biến, rồi khoanh các khối ô-1 kích thước lũy thừa 2 — mỗi khối là một tích đã rút gọn (biến nào đổi giá trị trong khối thì biến mất).

```
        yz=00  01   11   10           f = x∧y ∨ x∧¬y  ?
  x=0 │  0    0    0    0
  x=1 │  1    1    1    1   ← khoanh cả hàng: y, z đều đổi → f = x
```

Ví dụ trên: `(x∧y∧z) ∨ (x∧y∧¬z) ∨ (x∧¬y∧z) ∨ (x∧¬y∧¬z)` rút về đúng một chữ `x`. Với engineer phần mềm, Karnaugh map ít khi dùng trực tiếp (quá 4 biến thì bất trị — công cụ tự động như Quine-McCluskey và ESPRESSO thay thế), nhưng nó cho một trực giác đáng giữ: **điều kiện `if` phức tạp thường là dạng chưa rút gọn của một điều kiện đơn giản** — và luật đại số là con đường an toàn để tìm dạng gọn.

### Từ đại số đến CPU: half adder

Tuyên bố "mọi phép tính đều là mạch AND/OR/NOT" nghe to tát, nhưng kiểm chứng được bằng ví dụ nhỏ nhất: cộng hai bit a + b cho kết quả 2 bit (sum, carry):

```
 a b │ carry sum          sum   = a XOR b   (khác nhau thì 1)
 0 0 │   0    0           carry = a AND b   (cả hai là 1 mới nhớ)
 0 1 │   0    1
 1 0 │   0    1           a ──┬──[XOR]── sum
 1 1 │   1    0           b ──┴──[AND]── carry
```

Đây là **half adder** — hai cổng logic. Ghép hai half adder thành full adder (cộng cả bit nhớ từ hàng trước), xâu 64 full adder thành bộ cộng 64-bit, và từ cộng suy ra trừ (bù hai), nhân (cộng có dịch), so sánh (trừ rồi xét dấu). Mọi phép tính số học bạn từng chạy đều tan rã, ở đáy cùng, thành bảng chân trị của chương 01 nối dây với nhau. Boolean algebra không phải "liên quan đến" phần cứng — nó *là* phần cứng.

Còn XOR — nhân vật sẽ chiếm sân khấu mục sau — không phải phép toán nguyên thủy mà định nghĩa được: a⊕b = (a∨b) ∧ ¬(a∧b), "hoặc nhưng không phải cả hai". Nhìn từ half adder: XOR chính là **phép cộng vứt bỏ bit nhớ** — quan sát nhỏ này là chìa khóa hiểu mọi trick XOR.

## 4. Mathematical Model

### Số nguyên như vector Boole — và các phép toán trong Go

Mô hình trung tâm của nửa sau chương: `uint64` là vector 64 chiều trên {0,1}, các toán tử bitwise là phép toán Boole áp **song song từng vị trí** (component-wise):

```go
a & b   // AND từng bit — giao tập hợp
a | b   // OR từng bit  — hợp tập hợp
a ^ b   // XOR từng bit — hiệu đối xứng (Go dùng ^ cho cả NOT một ngôi: ^a)
a &^ b  // AND NOT ("bit clear") — hiệu tập hợp A ∖ B, toán tử riêng của Go
a << k  // dịch trái k vị trí — nhân 2ᵏ
a >> k  // dịch phải k vị trí — chia 2ᵏ (unsigned: điền 0; signed: điền bit dấu)
```

Lưu ý ngữ nghĩa Go: không có toán tử `~` như C — NOT viết là `^a`; và `&^` là đặc sản giúp viết "tắt các bit này đi" không cần NOT rồi AND.

### XOR: nhóm giao hoán trong lòng CPU

XOR đặc biệt vì nó cho {0,1}ⁿ cấu trúc **nhóm giao hoán** (abelian group) — một hệ với đầy đủ tính chất đại số đẹp:

1. **Kết hợp & giao hoán**: (a⊕b)⊕c = a⊕(b⊕c), a⊕b = b⊕a — XOR một dãy số không phụ thuộc thứ tự và cách nhóm.
2. **Phần tử trung hòa**: a⊕0 = a.
3. **Mọi phần tử là nghịch đảo của chính nó**: a⊕a = 0.

Ba tính chất này — không hơn — sinh ra toàn bộ họ "trick XOR". Hệ quả tức thì: XOR của một dãy số sẽ **tự triệt tiêu mọi phần tử xuất hiện chẵn lần**. Nếu mảng có mọi số xuất hiện đúng 2 lần trừ một số xuất hiện 1 lần:

```go
// Tìm số xuất hiện lẻ lần — O(n) thời gian, O(1) bộ nhớ, không cần map.
func singleNumber(nums []int) int {
    x := 0
    for _, v := range nums {
        x ^= v // các cặp giống nhau tự hủy: a⊕a = 0; còn lại a⊕0 = a
    }
    return x
}
```

Không có cấu trúc nhóm, bài này cần hash map O(n) bộ nhớ. Cũng từ ba tính chất đó: **swap không cần biến phụ** — `a ^= b; b ^= a; a ^= b` (kiểm chứng: bước 2 cho b = (a⊕b)⊕b = a; bước 3 cho a = (a⊕b)⊕a = b). Trick này đáng biết để *đọc hiểu* code cũ và để hiểu XOR, nhưng trong Go hãy viết `a, b = b, a` — rõ hơn và không chậm hơn. Cùng nguyên lý a⊕a = 0 chạy trong: RAID 5 (parity block = XOR các block dữ liệu — mất block nào, XOR phần còn lại với parity là khôi phục), stream cipher (ciphertext = plaintext ⊕ keystream, giải mã là XOR lần nữa — chương 26), và rsync/diff nhị phân dò khối khác nhau.

### Hai đẳng thức nền của bit manipulation

**`x & (x-1)` xóa bit 1 thấp nhất.** Vì sao đúng? Viết x theo dạng nhị phân quanh bit 1 thấp nhất của nó:

> x = (phần cao)`1`(toàn số 0) — tức x = p·2ᵏ⁺¹ + 2ᵏ với 2ᵏ là bit 1 thấp nhất.
>
> Trừ 1 mượn qua toàn bộ đuôi 0: x−1 = (phần cao)`0`(toàn số 1).
>
> AND hai dòng: phần cao giữ nguyên (giống nhau), vị trí k: 1∧0 = 0, đuôi: 0∧1 = 0.
>
> Kết quả: đúng x nhưng bit 1 thấp nhất đã tắt. ∎

Hệ quả lập tức: đếm số bit 1 bằng cách lặp `x &= x-1` cho đến khi x = 0 — số vòng lặp đúng bằng số bit 1 (thuật toán Kernighan), và **kiểm tra lũy thừa của 2** trong một biểu thức: `x > 0 && x&(x-1) == 0` (lũy thừa của 2 là số có đúng một bit 1 — xóa nó đi phải còn 0).

**`x & -x` tách ra đúng bit 1 thấp nhất.** Trong biểu diễn bù hai, −x = ¬x + 1. Lấy ¬x lật mọi bit: (phần cao lật)`0`(toàn số 1); cộng 1 làm đuôi số 1 tràn ngược lên: (phần cao lật)`1`(toàn số 0). AND với x: phần cao gặp bản lật của chính nó → 0; vị trí k: 1∧1 = 1. Kết quả: chỉ còn 2ᵏ. Đây là viên gạch của Fenwick tree (chương 24) — cấu trúc dùng `x & -x` để nhảy giữa các khoảng tổng.

Điểm cần rút ra không phải hai công thức, mà là **phương pháp chứng minh**: mọi trick bit đều kiểm chứng được bằng cách chia số thành ba vùng (trước bit đang xét — tại bit — sau bit) và theo dõi từng vùng qua phép toán. Nắm phương pháp, bạn tự suy ra trick mới thay vì tra cứu.

### Bitmask làm tập hợp

Với tập nền ≤ 64 phần tử, một `uint64` biểu diễn trọn một tập con — và đại số tập hợp chương 02 trở thành các lệnh đơn:

| Phép tập hợp | Bitmask | Phép tập hợp | Bitmask |
|---|---|---|---|
| thêm phần tử i | `s \| (1 << i)` | giao A ∩ B | `a & b` |
| xóa phần tử i | `s &^ (1 << i)` | hợp A ∪ B | `a \| b` |
| i có trong tập? | `s & (1 << i) != 0` | hiệu A ∖ B | `a &^ b` |
| bật/tắt i | `s ^ (1 << i)` | lực lượng \|A\| | `bits.OnesCount64(s)` |

Sức mạnh thật lộ ra khi cần **duyệt mọi tập con**: 2ⁿ tập con của tập n phần tử tương ứng 1-1 với các số 0 .. 2ⁿ−1 (chương 05 — đây chính là chứng minh |P(A)| = 2ⁿ bằng song ánh):

```go
// Duyệt toàn bộ 2^n tập con của {0..n-1}. Khả thi với n ≤ ~25 (bảng ngưỡng chương 10).
for mask := 0; mask < 1<<n; mask++ {
    for i := 0; i < n; i++ {
        if mask&(1<<i) != 0 {
            // phần tử i thuộc tập con hiện tại
        }
    }
}
```

Bitmask còn là chìa khóa của DP trên tập hợp ("bitmask DP" — trạng thái là tập các phần tử đã dùng, chương 14) và là lý do ràng buộc n ≤ 20 trong đề bài gần như hét lên "duyệt tập con đi".

## 5. Thuật toán

### Đếm bit — từ vòng lặp đến lệnh phần cứng

Bài toán nhỏ mà minh họa trọn con đường tối ưu: đếm số bit 1 trong một từ (population count).

```go
import "math/bits"

// Cách 1 — Kernighan: chạy đúng (số bit 1) vòng, dựa trên x&(x-1).
func popcountLoop(x uint64) int {
    n := 0
    for x != 0 {
        x &= x - 1 // mỗi vòng tắt một bit 1
        n++
    }
    return n
}

// Cách 2 — chuẩn production: biên dịch thẳng thành lệnh POPCNT của CPU.
func popcount(x uint64) int {
    return bits.OnesCount64(x)
}
```

`math/bits` là câu trả lời đúng trong Go cho gần như mọi thao tác bit mức từ: `OnesCount` (đếm bit), `TrailingZeros` (vị trí bit 1 thấp nhất — bản "trả về chỉ số" của `x & -x`), `LeadingZeros` (suy ra ⌊log₂x⌋), `Len` (số bit cần để biểu diễn), `RotateLeft`, `Reverse`. Chúng được compiler nhận diện là intrinsic và thay bằng một lệnh phần cứng — vòng lặp tay không bao giờ đọ được, và ý định của code cũng rõ hơn.

### Flag pattern với iota — Boolean algebra trong API design

Kiểu "tập hợp cờ" là ứng dụng bitmask phổ biến nhất trong code Go hằng ngày, với idiom `iota` dịch trái:

```go
type Perm uint8

const (
    PermRead  Perm = 1 << iota // 1 — bit 0
    PermWrite                  // 2 — bit 1
    PermExec                   // 4 — bit 2
)

func (p Perm) Has(flag Perm) bool { return p&flag != 0 }

// Dùng: gọn, type-safe, kiểm tra bằng một lệnh AND
p := PermRead | PermWrite
if p.Has(PermWrite) { /* ... */ }
```

Đây chính xác là mô hình quyền `rwx` của Unix: `chmod 754` là ba cụm bit `111 101 100` viết dưới dạng bát phân — owner đủ rwx (7), group đọc và chạy (5), others chỉ đọc (4). Chuẩn thư viện Go đầy pattern này: `os.O_RDWR|os.O_CREATE|os.O_TRUNC`, các cờ của `net/http`, `syscall`. Một `uint32` chở được 32 quyết định có/không — so với struct 32 trường bool: nhỏ hơn, so sánh cả cụm bằng một phép `==`, giao hai tập quyền bằng một phép `&`.

### Bitmap index — đại số Boole làm database engine

Mở rộng ý tưởng bitmap của bài toán mở đầu thành nguyên lý tổng quát: với cột có ít giá trị phân biệt (status, country, gender), mỗi giá trị giữ một bitmap dài n bit — bit i bật nếu dòng i mang giá trị đó. Khi đó mệnh đề WHERE **là** biểu thức Boole trên các bitmap:

```
WHERE status='active' AND country='VN' AND NOT premium

  bitmap(active):  10110101...
& bitmap(VN):      11010100...          → mỗi từ 64-bit là một lệnh AND
&^ bitmap(premium):00100000...          → kết quả: bitmap các dòng thỏa mãn
```

Query planner không "duyệt và kiểm tra từng dòng" — nó thực hiện đại số tập hợp bằng phần cứng, 64 dòng mỗi lệnh, rồi mới chạm vào những dòng có bit bật. Toán nằm ở chỗ: tính đúng đắn của phép biến đổi "WHERE → biểu thức bitmap" được bảo lãnh bởi sự đẳng cấu giữa ba lớp áo ở mục 2 — logic mệnh đề trên dòng ↔ đại số tập hợp trên tập dòng ↔ phép bit trên bitmap.

## 6. Trade-off

**Bitmap vs danh sách ID (dày vs thưa).** Bitmap 100 triệu bit luôn tốn 12.5MB *bất kể* bao nhiêu bit bật. Tập 1.000 user active biểu diễn bằng danh sách ID chỉ tốn 8KB. Điểm hòa vốn thô: bitmap thắng khi mật độ bit bật vượt ~1/64 (một ID 64-bit đắt bằng 64 bit của bitmap). Thực tế không phải chọn nhị phân: **Roaring Bitmap** — dùng trong Elasticsearch, ClickHouse, Druid, Pilosa — chia không gian thành khối 2¹⁶ bit, khối dày lưu bitmap, khối thưa lưu danh sách, tự thích nghi theo mật độ từng vùng.

**Bit-packing vs khả năng đọc.** Nén 8 trường bool vào một byte tiết kiệm bộ nhớ và giúp so sánh/copy nguyên khối, nhưng mọi truy cập đều thành mask-shift, khó debug (giá trị hiển thị là số vô hồn) và dễ sai khi thêm trường. Quy tắc thực dụng: struct bool cho business logic thông thường; bit-packing khi có *hàng triệu* instance (mỗi byte nhân triệu lần mới đáng), khi giao thức/format quy định sẵn, hoặc khi cần phép toán trên cả tập cờ.

**XOR trick vs code rõ nghĩa.** Swap bằng XOR, kiểm tra dấu bằng bit — thông minh nhưng compiler hiện đại làm tốt hơn với code thẳng: `a, b = b, a` sinh mã tối ưu, còn XOR swap phá vỡ khả năng phân tích của compiler và *sai âm thầm* khi hai toán hạng là cùng một vị trí bộ nhớ (a⊕a = 0 — tự xóa mình). Trick đáng dùng là trick mang lại **bậc phức tạp khác** (Single Number: O(1) bộ nhớ thay vì O(n)) hoặc là intrinsic chuẩn (`math/bits`), không phải trick tiết kiệm một dòng.

**Karnaugh/tối giản hóa vs điều kiện đọc được.** Biểu thức `if` tối giản nhất về mặt đại số không phải lúc nào cũng dễ đọc nhất về mặt nghiệp vụ: `if user.IsAdmin || (user.IsOwner && doc.Editable)` mang tên các khái niệm domain; dạng "rút gọn" trộn các mệnh đề có thể ngắn hơn nhưng xóa mất ngữ nghĩa. Tối giản hóa máy móc thuộc về compiler và optimizer; con người tối ưu cho người đọc.

## 7. Production Applications

**Redis bitmap — SETBIT/BITCOUNT/BITOP.** Chính là lời giải bài toán mở đầu: `SETBIT active:2026-07-15 <user_id> 1` khi user hoạt động; `BITCOUNT` đếm DAU; `BITOP AND dest key1 ... key7` rồi `BITCOUNT dest` cho "active 7 ngày liên tiếp"; `BITOP OR` cho "active ít nhất một ngày trong tuần" (≈ MAU). 100 triệu user = 12.5MB/ngày. Toán nằm ở: đẳng cấu tập hợp ↔ bit biến câu hỏi phân tích thành phép AND/OR chạy tốc độ RAM tuần tự. Giới hạn cũng từ toán mà ra: user_id phải là số nguyên đặc (dense) — ID thưa/UUID làm bitmap phình vô ích (trade-off mật độ ở mục 6).

**PostgreSQL bitmap index scan.** Khi WHERE có nhiều điều kiện trên các index khác nhau (`a = 1 AND b > 5`), planner không chọn một index rồi lọc tay phần còn lại: nó quét *từng* index, dựng một bitmap các trang dữ liệu thỏa từng điều kiện, **AND/OR các bitmap** (BitmapAnd/BitmapOr trong EXPLAIN), rồi mới đọc những trang có bit bật — theo thứ tự vật lý, biến random I/O thành sequential. Đây là Boolean algebra làm cầu nối cho phép *kết hợp nhiều index* — điều mà index scan đơn lẻ không làm được.

**Bloom filter (chương 23).** Cấu trúc "có thể có / chắc chắn không" đứng trước mọi lần truy cập đắt (SSTable của Cassandra/RocksDB, cache miss check) — ruột của nó là một bit array với k hàm hash: thêm phần tử = bật k bit (OR), kiểm tra = AND của k bit. Toàn bộ phần "xác suất sai" để dành chương 23; phần "vì sao nhanh và nhỏ" nằm trọn ở chương này.

**Permission và flags xuyên suốt hệ thống.** Unix `rwx`/octal như mục 5; ngoài ra: file descriptor flags (`O_NONBLOCK`), TCP flags trong header (SYN, ACK, FIN — mỗi cờ một bit, handshake là chuỗi tổ hợp bit), CPU feature flags (`/proc/cpuinfo`), Linux capabilities, IP subnet mask — `ip & mask == network & mask` quyết định gói tin đi đâu: routing table tra bằng một phép AND. Kubernetes label selector về bản chất cũng là mệnh đề Boole trên tập label.

**Go runtime và compiler.** Kích thước size class của allocator, hash bucket của map đều là lũy thừa 2 để thay chia lấy dư bằng `h & (size-1)` (đúng khi size là 2ᵏ — vì modulo lũy thừa 2 chính là "giữ k bit thấp"); GC dùng bitmap đánh dấu con trỏ trên heap; compiler tự biến `x*8` thành `x<<3`, `x%16` thành `x&15` (với unsigned). Bạn hưởng các phép biến đổi đại số này mỗi lần chạy chương trình mà không cần viết chúng.

## 8. Interview

Bit manipulation trong phỏng vấn là bài kiểm tra "hiểu bản chất hay học vẹt" rõ nhất: các trick đều một dòng, nên interviewer không hỏi *code gì* mà hỏi *vì sao đúng* — chính là các chứng minh ở mục 4.

**Single Number (LeetCode 136).** Mọi số xuất hiện 2 lần trừ một số — chính là ví dụ `singleNumber` ở mục 4, ăn trọn ba tiên đề nhóm của XOR. Follow-up kinh điển kiểm tra chiều sâu: *Single Number II* (mọi số xuất hiện 3 lần trừ một số) — XOR bó tay vì a⊕a⊕a = a ≠ 0; lời giải tổng quát là đếm bit theo từng vị trí modulo 3 — cho thấy trick XOR là trường hợp riêng (modulo 2) của kỹ thuật đếm theo cột tổng quát hơn.

**Number of 1 Bits (LeetCode 191).** Trả lời tốt là trình bày được *thang* lời giải: lặp kiểm tra từng bit O(số bit) → Kernighan `x &= x-1` O(số bit 1, kèm chứng minh vì sao xóa đúng bit thấp nhất) → `bits.OnesCount` một lệnh phần cứng. Interviewer tìm người biết cả ba *và biết dùng cái cuối* trong production.

**Subsets (LeetCode 78).** Sinh mọi tập con — vòng lặp `mask` ở mục 4. Điểm cộng lớn trong phỏng vấn: nói rõ song ánh "mỗi số 0..2ⁿ−1 ↔ một tập con, bit i = có mặt phần tử i", thay vì chỉ viết hai vòng lặp. So sánh được với lời giải backtracking (chương 05) và chỉ ra khi nào bitmask thắng: cần *lưu* tập con làm key của map/DP state — số nguyên làm key rẻ hơn slice.

**Counting Bits (LeetCode 338).** Đếm bit 1 của mọi số 0..n trong O(n) tổng — bài đẹp vì nối bit vào DP: `count[x] = count[x>>1] + x&1` (bỏ bit cuối, cộng lại nếu nó là 1) hoặc `count[x] = count[x&(x-1)] + 1` (đúng theo đẳng thức đã chứng minh: `x&(x-1)` có ít hơn x đúng một bit 1). Người hiểu mục 4 *nhìn ra* công thức thứ hai thay vì nhớ nó.

**Lỗi tư duy phổ biến:**

- **Quên độ ưu tiên toán tử.** Trong C/Java, `x & 1 == 0` bị parse thành `x & (1 == 0)` — bug kinh điển. Go xử lý khác (`&` có độ ưu tiên *cao hơn* `==` trong Go, và Go cấm trộn kiểu int/bool nên nhiều bug loại này thành lỗi biên dịch), nhưng bài học giữ nguyên: **luôn đóng ngoặc biểu thức bit** — `(x & 1) == 0` — vì code của bạn sẽ được đọc bởi người mang phản xạ từ ngôn ngữ khác.
- **Nhầm logical với bitwise.** `&&` làm việc trên bool và short-circuit (vế phải có thể không được thực thi — nếu nó có side effect thì ngữ nghĩa khác hẳn); `&` làm việc trên số nguyên, luôn tính cả hai vế. Go tách bạch kiểu nên khó nhầm hơn C, nhưng câu hỏi "khác nhau thế nào?" vẫn là câu screening phổ biến.
- **Quên số âm và bù hai.** `>>` trên số có dấu điền bit dấu (arithmetic shift): `-8 >> 1 == -4`, không phải số dương khổng lồ; `x & -x` chỉ giải thích được qua bù hai. Nói được "−x = ¬x + 1" là tín hiệu hiểu biểu diễn số, chủ đề chương 19.
- **Dịch quá độ rộng kiểu.** `1 << 40` với `1` là int 32-bit (Java/C) là undefined/sai; Go: hằng untyped thì an toàn hơn nhưng `uint32(1) << 40` vẫn cho 0. Luôn hỏi "toán hạng của tôi rộng bao nhiêu bit?".

**Cách phân tích thay vì học thuộc:** gặp bài bit lạ, (1) viết vài ví dụ ra nhị phân bằng tay — 4 bit là đủ thấy pattern; (2) chia số thành ba vùng quanh bit đang quan tâm và theo dõi từng vùng (phương pháp mục 4); (3) tự hỏi phép toán nào có *tính chất đại số* khớp yêu cầu: cần "tự triệt tiêu theo cặp" → XOR; cần "chặn/lọc" → AND với mask; cần "gom" → OR; cần "nhân chia 2ᵏ / truy cập bit k" → shift.

**Từ phỏng vấn sang production:** Single Number là nguyên lý parity của RAID 5 thu nhỏ; Subsets bằng bitmask là cách feature flag service liệt kê tổ hợp experiment; Counting Bits là popcount trên bitmap index — `BITCOUNT` của Redis chạy chính lệnh mà LeetCode 191 mô phỏng.

## 9. Anti-pattern

**Bit-packing mọi thứ vì "tiết kiệm bộ nhớ".** Nén 5 trường bool của một struct chỉ có vài nghìn instance vào một `uint8` — tiết kiệm được vài KB, trả giá bằng mask-shift ở mọi điểm truy cập, debugger hiển thị số vô nghĩa, và một field mới là một lần soát toàn bộ mask. Bit-packing là tối ưu có điều kiện kích hoạt (hàng triệu instance, format cố định, cần phép toán trên cả tập) — không phải style mặc định.

**Magic number không tên.** `if flags & 0x2004 != 0` — sáu tháng sau không ai (kể cả tác giả) biết bit 2 và bit 13 là gì. Mọi bit phải có tên hằng (`const FlagRetry = 1 << 2`), mọi tổ hợp hay dùng phải có tên riêng (`const FlagsDefault = FlagRetry | FlagLog`). Bit layout là API — không có tên thì không có contract.

**Tự viết lại `math/bits`.** Vòng lặp đếm bit tay, lookup table tự chế cho popcount, "clever hack" từ Hacker's Delight chép vào — chậm hơn intrinsic một lệnh phần cứng, dài hơn, và sai ở biên (kiểu có dấu, độ rộng). Trong Go, thao tác bit mức từ đã có người viết, kiểm thử và tối ưu hộ bạn.

**Phủ định điều kiện phức tạp bằng cách... thêm dấu `!` bên ngoài.** `if !(a && (b || !c) && !(d || e))` — mỗi lần cần nhánh ngược lại thêm một tầng phủ định, biểu thức thành hộp đen không ai dám sửa. De Morgan tồn tại để đẩy phủ định vào trong đến từng mệnh đề đơn; sau vài bước máy móc, biểu thức thành chuỗi điều kiện dương dễ đọc — hoặc lộ ra rằng nó rút gọn được (Karnaugh tư duy). Query trên DB cũng vậy: điều kiện chuẩn hóa tốt giúp optimizer dùng index; `NOT` bọc ngoài cả cụm thường làm mất khả năng đó.

**Dùng bitmap cho không gian ID thưa.** SETBIT với user_id là số ngẫu nhiên 64-bit hoặc hash của UUID: Redis cấp phát bitmap dài đến bit cao nhất từng bật — một ID lớn cấp phát hàng trăm MB cho toàn số 0. Bitmap chỉ đúng với ID đặc, tăng dần; ID thưa cần Roaring Bitmap hoặc set thường. Đây là bài toán mật độ ở mục 6 bị bỏ qua.

**Tin rằng bool trong struct "tốn 1 bit".** `bool` trong Go tốn 1 byte, và alignment có thể độn thêm: struct `{bool; int64; bool}` tốn 24 byte trong khi `{int64; bool; bool}` tốn 16. Trước khi bit-packing, chỉ cần *sắp lại thứ tự field* đã thu hồi phần lớn bộ nhớ lãng phí — tối ưu rẻ hơn và không đổi cách đọc code.

## 10. Best Practices

**Nên:**

- Dùng `math/bits` cho mọi thao tác mức từ: `OnesCount`, `TrailingZeros`, `LeadingZeros`, `Len`, `RotateLeft` — nhanh (intrinsic), đúng, và tự tài liệu hóa ý định.
- Định nghĩa flag bằng `1 << iota`, kèm method `Has/Set/Clear` — che mask-shift sau API có tên, người dùng flag không cần nghĩ về bit.
- Đóng ngoặc mọi biểu thức bit trong điều kiện: `(x & mask) != 0`. Rẻ, và miễn dịch với khác biệt độ ưu tiên giữa các ngôn ngữ.
- Khi gặp trick bit lạ, kiểm chứng bằng phương pháp ba vùng (mục 4) trên ví dụ 4-bit trước khi tin và trước khi dùng — và ghi chứng minh một dòng vào comment.
- Cân nhắc bitmap/bitset khi: tập nền là số nguyên đặc, cần phép giao/hợp/đếm hàng loạt, kích thước hàng triệu phần tử. Cân nhắc Roaring khi mật độ không đều.
- Áp De Morgan chủ động khi refactor điều kiện: đẩy `!` vào trong, đặt tên cho mệnh đề con (`isEligible := ...`), để cấu trúc logic lộ ra ánh sáng.

**Không nên:**

- Không XOR-swap, không thay `x*2` bằng `x<<1` trong code thường — compiler làm rồi, bạn chỉ làm code khó đọc đi.
- Không bit-packing khi chưa đo được lợi ích bộ nhớ *và* chưa thử sắp lại field struct.
- Không dùng số có dấu cho bitmask (shift phải kéo bit dấu theo, so sánh và mở rộng kiểu đầy bẫy) — mask là `uint`, kích thước cố định (`uint32`/`uint64`).

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Chứng minh `x & (x-1)` xóa đúng bit 1 thấp nhất — và dùng nó giải thích vì sao `x > 0 && x&(x-1) == 0` kiểm tra được lũy thừa của 2.
2. Ba tính chất nhóm nào của XOR làm nên lời giải Single Number, và vì sao chính lời giải đó thất bại khi các số xuất hiện 3 lần?
3. PostgreSQL bitmap index scan kết hợp được nhiều index trong một query nhờ sự đẳng cấu giữa những cấu trúc nào? Bitmap thua danh sách ID khi nào?

---

*Chương tiếp theo: [10 — Complexity Analysis](/series/math-for-engineers/level-3-algorithm-mathematics/10-complexity-analysis/) — bước sang Level 3, nơi mọi thuật toán từ đầu tài liệu đến giờ được đặt lên bàn cân chung: Big-O, amortized analysis, và nghệ thuật dự đoán hành vi của code trên dữ liệu lớn mà không cần chạy nó.*
