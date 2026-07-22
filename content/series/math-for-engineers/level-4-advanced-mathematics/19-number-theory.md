+++
title = "Chương 19 — Number Theory: Số học của máy tính và mật mã"
date = "2026-07-20T10:10:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 4 – Advanced Mathematics
> Yêu cầu trước: Chương 03 (Proof Techniques), Chương 11 (Probability), Chương 12 (Hashing)

---

## 1. Problem Statement

Ngày 4 tháng 6 năm 1996, tên lửa Ariane 5 của Cơ quan Vũ trụ châu Âu phát nổ 37 giây sau khi rời bệ phóng, mang theo thiết bị trị giá khoảng 370 triệu USD. Nguyên nhân gốc, sau nhiều tháng điều tra: một đoạn code tái sử dụng từ Ariane 4 chuyển giá trị vận tốc ngang từ số thực 64-bit sang số nguyên 16-bit. Ariane 5 bay nhanh hơn Ariane 4; giá trị vượt quá 32767; phép chuyển đổi tràn số; hệ thống dẫn đường tự tắt. Không phải lỗi thuật toán, không phải lỗi logic — chỉ là một con số vượt ra ngoài chiếc hộp chứa nó.

Gần hai thập kỷ sau, tháng 12/2014, YouTube phải nâng bộ đếm view từ int32 lên int64 vì "Gangnam Style" tiến sát 2.147.483.647 lượt xem — giới hạn của số nguyên 32 bit có dấu. Lần này không có gì phát nổ, vì có người nhìn thấy trước.

Hai câu chuyện, một sự thật mà mọi kỹ sư phải nội tâm hóa: **số nguyên trong máy tính không phải là số nguyên trong toán học**. ℤ là vô hạn; `int64` chỉ có 2⁶⁴ giá trị. Câu hỏi của chương này:

**Số nguyên hữu hạn của máy tính tuân theo hệ thống toán học nào — và hệ thống đó cho ta công cụ gì?**

Câu trả lời bất ngờ ở chỗ: hệ thống đó không phải là "ℤ bị cắt cụt một cách đáng tiếc", mà là một cấu trúc đại số hoàn chỉnh và đẹp — **số học modular** — thứ đồng thời là nền móng của hash function, sharding, ring buffer, ID generation, và toàn bộ mật mã khóa công khai đang bảo vệ kết nối HTTPS bạn dùng để đọc tài liệu này. Không hiểu nó, overflow là bug ngẫu nhiên rình rập; hiểu nó, overflow là tính chất có thể khai thác.

## 2. Trực giác

### Chiếc đồng hồ — máy tính modular đầu tiên của nhân loại

Bây giờ là 22 giờ. 7 tiếng nữa là mấy giờ? Bạn không trả lời "29 giờ" — bạn trả lời "5 giờ sáng". Não bạn vừa thực hiện phép tính 22 + 7 ≡ 5 (mod 24) mà không cần ai dạy. Đồng hồ là một hệ số học **quấn vòng**: đi quá 24 thì quay về 0, và mọi phép cộng đều "sống" trên vòng tròn đó.

`int64` của Go chính xác là một chiếc đồng hồ có 2⁶⁴ vạch. Cộng hai số vượt quá vạch cuối? Quay vòng về đầu. Điều toán học nói với ta: đây không phải hành vi lỗi — đây là phép cộng **trong ℤ/2⁶⁴ℤ** (tập các số dư khi chia cho 2⁶⁴), và phép cộng đó tuân thủ đầy đủ luật kết hợp, giao hoán, phân phối. Máy tính chưa bao giờ tính sai; nó chỉ tính trong một hệ khác với hệ mà lập trình viên *tưởng* mình đang dùng. Bug overflow là bug về **kỳ vọng**, không phải về phép tính.

### Two's complement bỗng trở nên hiển nhiên

Vì sao `-1` trong int8 có bit pattern `11111111` (tức 255 nếu đọc không dấu)? Học vẹt thì thấy tùy tiện; nhìn qua lăng kính modular thì hiển nhiên: trên đồng hồ 256 vạch, **lùi 1 vạch và tiến 255 vạch là cùng một điểm đến**: −1 ≡ 255 (mod 256). Two's complement không phải một "mẹo mã hóa" — nó là quyết định gọi một nửa vòng tròn bằng tên âm. Phần thưởng: phần cứng chỉ cần *một* mạch cộng cho cả số âm lẫn số dương, vì trong ℤ/256ℤ chúng vốn là một.

### Vòng tròn ở khắp nơi trong hệ thống

Một khi nhìn ra "vòng tròn", bạn thấy nó khắp nơi: ring buffer (`pos = (pos+1) % size`), sharding (`shard = hash(key) % n`), consistent hashing ring (vòng tròn 2⁶⁴ điểm), cửa sổ rate limiter (`bucket = timestamp % windowSize`), sequence number của TCP (so sánh "trước/sau" trên vòng 2³² — lý do có phép so sánh serial number arithmetic kỳ lạ trong RFC 1982). Số học modular là hình học của những thứ tuần hoàn — và hệ thống máy tính đầy những thứ tuần hoàn.

### Còn số nguyên tố thì liên quan gì?

Trên đồng hồ 12 vạch, đi từng bước 4 vạch: 0 → 4 → 8 → 0 → ... bạn chỉ ghé 3 điểm, bỏ sót 9 điểm còn lại. Đi từng bước 5 vạch: 0 → 5 → 10 → 3 → 8 → 1 → ... bạn ghé **đủ 12 điểm** rồi mới quay lại. Khác biệt: gcd(4,12) = 4 ≠ 1, còn gcd(5,12) = 1. Khi kích thước vòng là **số nguyên tố**, mọi bước đi (trừ 0) đều phủ kín vòng — không có "cấu trúc con" nào để mắc kẹt vào. Đó là trực giác cho gần như mọi lần số nguyên tố xuất hiện trong kỹ thuật: hash table mod prime, double hashing, và cả lý do sâu xa khiến mật mã xây trên prime — vòng tròn prime là vòng tròn "không có đường tắt".

## 3. First Principles

### Xây lại từ phép chia có dư

Mọi thứ khởi đi từ một định lý tiểu học phát biểu chặt chẽ: với a ∈ ℤ, n > 0, tồn tại **duy nhất** q, r sao cho a = qn + r và 0 ≤ r < n. Số dư r là "vị trí trên vòng tròn n vạch". Hai số **đồng dư** modulo n khi cùng vị trí:

> a ≡ b (mod n) ⟺ n chia hết (a − b)

Quan hệ đồng dư là quan hệ tương đương (chương 02), chia ℤ thành đúng n lớp. Tập các lớp đó, ký hiệu ℤ/nℤ = {0, 1, ..., n−1}, cùng phép cộng và nhân "tính rồi lấy dư", tạo thành một cấu trúc khép kín. Điều phải kiểm tra — và là **tính chất làm việc được** quan trọng nhất chương:

> (a + b) mod n = ((a mod n) + (b mod n)) mod n
> (a · b) mod n = ((a mod n) · (b mod n)) mod n

Nghĩa là: **lấy dư sớm hay muộn đều cho cùng kết quả**. Hệ quả thực dụng khổng lồ: muốn tính `(a·b·c) mod n` với a, b, c khổng lồ, cứ mod sau *từng* phép nhân — không bao giờ cần giữ số lớn. Rolling hash của Rabin-Karp tính hash của cửa sổ mới từ cửa sổ cũ được là nhờ tính chất này; mọi thuật toán mật mã làm việc với số 2048-bit sống sót được cũng nhờ nó.

### Phép chia — nơi mọi thứ khác đi

Cộng, trừ, nhân đều "quấn vòng" êm ả. Chia thì không: trong ℤ/12ℤ, phương trình 4·x ≡ 1 (mod 12) **vô nghiệm** (4x luôn chẵn, không thể ≡ 1). Nhưng 5·x ≡ 1 (mod 12) có nghiệm x = 5 (vì 25 = 24 + 1). "Chia cho 5" tồn tại, "chia cho 4" thì không. Ranh giới nằm đúng ở gcd: **a có nghịch đảo modulo n ⟺ gcd(a, n) = 1** (a và n nguyên tố cùng nhau — coprime). Khi n là prime, *mọi* phần tử khác 0 đều có nghịch đảo — ℤ/pℤ trở thành một **trường (field)**, nơi bốn phép toán đầy đủ như trên số thực. Đây là lý do kỹ thuật khiến prime là "vòng tròn không đường tắt": không có ước chung nào để cấu trúc bị gãy thành vòng con.

Nếu không có khái niệm nghịch đảo modular? Không có RSA (giải mã chính là nhân với nghịch đảo của khóa mã trong số mũ), không có chia trong các hệ chữ ký số, không có cách "hoàn tác" phép nhân trong bất kỳ giao thức nào làm việc trên số nguyên hữu hạn.

### GCD — và thuật toán cổ nhất còn chạy trong production

Ước chung lớn nhất gcd(a, b) là "đơn vị đo chung lớn nhất" của hai số. Euclid (~300 TCN) nhận ra điều then chốt:

> **gcd(a, b) = gcd(b, a mod b)**

Chứng minh gọn: nếu d chia hết cả a và b, thì d chia hết a − qb = a mod b, nên d là ước chung của (b, a mod b). Ngược lại, nếu d chia hết b và a mod b, thì d chia hết qb + (a mod b) = a. Hai tập ước chung **trùng nhau**, nên phần tử lớn nhất trùng nhau. ∎

Lặp phép thu gọn này đến khi số thứ hai bằng 0 — đó là thuật toán Euclid, ~2300 năm tuổi, hiện vẫn chạy mỗi lần bạn tạo khóa TLS. Nó là minh chứng sớm nhất cho tư duy "thu nhỏ bài toán về bản sao nhỏ hơn của chính nó" — divide and conquer trước divide and conquer 23 thế kỷ.

## 4. Mathematical Model

### Độ phức tạp của Euclid — và Fibonacci xuất hiện

Mỗi bước, cặp (a, b) thành (b, a mod b). Quan sát: sau **hai** bước, số lớn giảm quá nửa (nếu b ≤ a/2 thì a mod b < b ≤ a/2; nếu b > a/2 thì a mod b = a − b < a/2). Vậy số bước là O(log min(a, b)).

Đâu là input *xấu nhất*? Cặp làm thuật toán co chậm nhất là cặp mà thương luôn bằng 1: a mod b = a − b. Truy ngược lại: cặp trước của (b, r) là (b + r, b) — đúng công thức Fibonacci. **Hai số Fibonacci liên tiếp là worst case của Euclid** (định lý Lamé, 1844 — được xem là phân tích độ phức tạp đầu tiên trong lịch sử). Vì Fₙ ≈ φⁿ/√5, số bước tối đa ≈ log_φ(min) ≈ 1.44 · log₂(min). Chặt và đẹp.

### Extended Euclid — không chỉ tìm gcd mà tìm cả "công thức pha chế"

Định lý Bézout: gcd(a, b) luôn viết được dạng **ax + by = gcd(a, b)** với x, y nguyên. Thuật toán Euclid mở rộng tính x, y bằng cách lan truyền ngược qua các bước chia. Ứng dụng số một: khi gcd(a, n) = 1, từ ax + ny = 1 suy ra **ax ≡ 1 (mod n)** — tức x chính là **nghịch đảo modular a⁻¹**. Đây là cách RSA tính khóa bí mật d từ khóa công khai e: d = e⁻¹ mod φ(N).

### Số nguyên tố — nguyên tử của số học

**Định lý cơ bản của số học:** mọi số nguyên > 1 phân tích được thành tích các số nguyên tố, **duy nhất** (không kể thứ tự). Prime là "nguyên tử": mọi cấu trúc nhân của số nguyên đều đọc được từ phân tích thừa số. gcd là "giao" của hai phân tích, lcm là "hợp" — nhưng lưu ý: tìm gcd bằng Euclid *không cần* phân tích thừa số, và đó là điều may mắn, vì phân tích thừa số thì **khó** (khó đến mức cả nền mật mã đặt cược vào đó — mục dưới).

**Prime phân bố dày đến đâu?** Prime Number Theorem: số prime ≤ n xấp xỉ n/ln n — tức quanh vùng số n, "mật độ" prime khoảng **1/ln n**. Trực giác về con số: quanh 2¹⁰²⁴ (cỡ prime của RSA-2048), ln(2¹⁰²⁴) ≈ 710, tính riêng số lẻ thì trung bình thử ~355 số ngẫu nhiên là gặp một prime. Prime lớn không hiếm — chúng ở khắp nơi, chỉ cần biết cách *kiểm tra* nhanh. Và đó là mảnh ghép tiếp theo.

### Fermat và Euler — hai định lý gánh cả nền mật mã

> **Định lý nhỏ Fermat:** nếu p prime và p không chia hết a, thì a^(p−1) ≡ 1 (mod p).

Trực giác: nhân toàn bộ {1, 2, ..., p−1} với a (mod p) chỉ **xáo trộn thứ tự** tập đó (vì a có nghịch đảo nên phép nhân là song ánh — chương 02). Tích hai vế bằng nhau: a^(p−1) · (p−1)! ≡ (p−1)! → giản ước (p−1)! (được phép vì mọi thừa số có nghịch đảo) → a^(p−1) ≡ 1. ∎

> **Định lý Euler (tổng quát hóa):** nếu gcd(a, n) = 1 thì a^φ(n) ≡ 1 (mod n), với φ(n) = số các số ≤ n nguyên tố cùng nhau với n. Với n = p·q (hai prime): φ(n) = (p−1)(q−1).

Ý nghĩa vận hành: **số mũ sống trong thế giới modulo φ(n)** — a^k chỉ phụ thuộc k mod φ(n). RSA (chương 26) là hệ quả trực tiếp: chọn e·d ≡ 1 (mod φ(N)), thì (m^e)^d = m^(ed) ≡ m (mod N) — mã hóa rồi giải mã trả về đúng bản gốc. Toàn bộ "phép màu" chỉ là định lý Euler cộng một sự thật tính toán: biết N mà không biết p, q thì không tính nổi φ(N), nên không tính nổi d.

### Vì sao hash nên mod prime — nhìn collision bằng số học

Hash đa thức cho chuỗi: h(s) = (s₀·bᵏ⁻¹ + s₁·bᵏ⁻² + ... + sₖ₋₁) mod m. Nếu chọn m = 2ᵏ (để mod thành phép AND rẻ tiền), tai họa cấu trúc: mod 2ᵏ **chỉ nhìn thấy k bit thấp** — mọi thông tin ở bit cao của các số hạng bậc cao bị nhân với lũy thừa của 2 rồi biến mất khỏi phần dư. Các key chỉ khác nhau ở phần cao sẽ đụng độ **có hệ thống**, không phải ngẫu nhiên như mô hình birthday paradox (chương 12) giả định. Chọn m prime (và b nguyên tố cùng nhau với m): không thừa số chung, mọi bit của input đều "khuấy" vào phần dư, collision trở về gần với ngẫu nhiên đều. Cùng logic, bảng hash dùng double hashing cần bước nhảy nguyên tố cùng nhau với kích thước bảng để probe phủ kín — chính là trực giác đồng hồ 12 vạch ở mục 2.

**CRC** (mức nhắc để hoàn chỉnh bức tranh): checksum CRC-32 trong ZIP, Ethernet chính là phần dư của phép chia đa thức trên trường GF(2) — vẫn là "mod", chỉ thay số nguyên bằng đa thức hệ số bit. Số học modular tổng quát hóa xa hơn số nguyên nhiều.

## 5. Thuật toán

### Fast exponentiation — từ O(n) xuống O(log n)

Tính a^n mod m ngây thơ cần n−1 phép nhân — với n cỡ 2²⁰⁴⁸ của RSA thì vũ trụ tắt trước khi xong. Quan sát cứu rỗi: a¹⁶ = ((a²)²)² — **bình phương liên tiếp** đạt số mũ 2ᵏ chỉ với k phép nhân. Số mũ bất kỳ ghép từ các bit của nó: a¹³ = a⁸·a⁴·a¹ (13 = 1101₂).

```go
// PowMod tính (base^exp) mod m trong O(log exp) phép nhân.
// Nền tảng của RSA, Diffie-Hellman, Miller-Rabin.
func PowMod(base, exp, m uint64) uint64 {
    if m == 1 {
        return 0
    }
    result := uint64(1)
    base %= m
    for exp > 0 {
        if exp&1 == 1 { // bit thấp nhất của exp là 1 → thừa số này góp mặt
            result = mulMod(result, base, m)
        }
        base = mulMod(base, base, m) // bình phương: a^(2^k) → a^(2^(k+1))
        exp >>= 1
    }
    return result
}

// mulMod tránh overflow khi nhân hai số 64-bit trước khi mod:
// a*b có thể cần tới 128 bit — nhân rồi mới mod là sai với m lớn.
func mulMod(a, b, m uint64) uint64 {
    hi, lo := bits.Mul64(a, b)      // tích 128-bit: hi:lo
    _, rem := bits.Div64(hi, lo, m) // chia 128-bit cho m, lấy dư
    return rem
}
```

2048 phép bình phương thay cho 2²⁰⁴⁸ phép nhân — khoảng cách giữa "không thể" và "vài mili giây". Chú ý `mulMod`: đây là chỗ chương này bắt tay chương 10 — *biết* thuật toán đúng chưa đủ, còn phải biết phép nhân trung gian tràn 64 bit. Quên điều này là lỗi số 1 khi implement (mục 8).

### Kiểm tra nguyên tố: từ √n đến Miller-Rabin

**Trial division:** thử chia cho mọi số ≤ √n (vì hợp số n phải có ước ≤ √n — nếu cả hai ước > √n thì tích > n). O(√n): ổn với n < 10¹², vô vọng với số 2048-bit.

**Miller-Rabin** đảo hướng: thay vì *tìm ước*, dùng Fermat làm **máy phát hiện nói dối**. Nếu n prime thì a^(n−1) ≡ 1 (mod n) với mọi a; vậy nếu tìm được a khiến điều đó sai, n **chắc chắn** là hợp số — mà chẳng cần biết ước nào. Miller-Rabin gia cố thêm bằng cách soi cả dãy căn bậc hai trung gian (viết n−1 = 2^s·d, xem chuỗi a^d, a^2d, ..., a^(n−1)): với n hợp số, **ít nhất 3/4 số witness a** sẽ vạch trần nó. Thử k witness ngẫu nhiên độc lập:

> P(hợp số sống sót cả k lần) ≤ 4⁻ᵏ

Với k = 40: sai số < 2⁻⁸⁰ — nhỏ hơn xác suất tia vũ trụ lật bit RAM ngay trong lúc tính. Đây là bài học của chương 11 hiện hình lần nữa: thuật toán ngẫu nhiên với sai số **định lượng được và điều khiển được** là công cụ production hoàn toàn nghiêm túc. `crypto/rsa` của Go, OpenSSL — tất cả kiểm tra prime bằng Miller-Rabin (Go: `(*big.Int).ProbablyPrime`, kết hợp thêm Baillie-PSW). Toàn bộ HTTPS của Internet đặt trên các con số "chỉ" chắc chắn nguyên tố ở mức 1 − 2⁻⁸⁰.

### Sieve of Eratosthenes — liệt kê prime hàng loạt

Cần *mọi* prime ≤ n? Đừng kiểm tra từng số — gieo sàng: viết dãy 2..n, lấy số nhỏ nhất chưa bị đánh dấu (là prime), đánh dấu mọi bội của nó, lặp lại.

```go
// Sieve trả về isPrime[0..n]. Bộ nhớ O(n), thời gian O(n log log n).
func Sieve(n int) []bool {
    isPrime := make([]bool, n+1)
    for i := 2; i <= n; i++ {
        isPrime[i] = true
    }
    for p := 2; p*p <= n; p++ { // chỉ cần sàng tới √n — như trial division
        if isPrime[p] {
            for multiple := p * p; multiple <= n; multiple += p {
                isPrime[multiple] = false // bội nhỏ hơn p*p đã bị prime nhỏ hơn sàng
            }
        }
    }
    return isPrime
}
```

Vì sao O(n log log n)? Tổng công việc là Σ (n/p) trên các prime p ≤ n. Chuỗi điều hòa đầy đủ Σ n/k cho n·ln n; nhưng ta chỉ cộng trên prime — thưa với mật độ 1/ln n (Prime Number Theorem ở mục 4) — nên tổng co lại còn n·ln ln n (định lý Mertens). "log log n" trong thực tế ≤ 5 với mọi n bạn từng gặp: sàng gần như tuyến tính.

### Bức tranh bất đối xứng — nơi mật mã sống

Ghép các mảnh lại, lộ ra một sự bất đối xứng kỳ lạ của tự nhiên:

| Bài toán | Chi phí |
|---|---|
| Kiểm tra n có phải prime | Nhanh — Miller-Rabin O(k·log³n) |
| Sinh prime ngẫu nhiên 1024-bit | Nhanh — thử ~355 ứng viên (mật độ 1/ln n) |
| Nhân hai prime p·q = N | Tầm thường |
| Từ N tìm lại p, q (factoring) | **Không có thuật toán nhanh nào được biết** |

Nhân thì dễ, tách thì khó — một "cửa một chiều" (one-way trapdoor). RSA không phải phát minh ra sự khó đó; nó chỉ *đóng gói* sự khó đó thành giao thức. Chương 26 xây tiếp từ đây.

## 6. Trade-off

**Xác suất vs tất định trong primality.** Tồn tại thuật toán tất định đa thức (AKS, 2002 — đột phá lý thuyết lớn), nhưng production vẫn dùng Miller-Rabin: nhanh hơn nhiều bậc, và sai số 2⁻⁸⁰ nhỏ hơn xác suất lỗi phần cứng — "chắc chắn tuyệt đối" của AKS chạy trên RAM có thể lật bit là chắc chắn ảo. Bài học tổng quát: so sánh sai số thuật toán với sai số của *toàn hệ thống*, đừng đòi một thành phần hoàn hảo hơn môi trường chứa nó.

**Mod 2ᵏ vs mod prime.** Mod 2ᵏ = một lệnh AND (~1 chu kỳ); mod prime = phép chia (~20–40 chu kỳ). Hash table hot path chọn 2ᵏ và bù đắp bằng hash function trộn bit mạnh phía trước (murmur, wyhash — hướng của Go runtime); hash đa thức đơn giản kiểu Rabin-Karp thì cần mod prime vì chính nó không trộn đủ. Chọn *một trong hai chỗ* để gánh tính ngẫu nhiên — gánh cả hai là thừa, gánh không chỗ nào là collision cấu trúc.

**Sieve vs kiểm tra từng số.** Sieve trả giá O(n) bộ nhớ để được O(n log log n) tổng — vô địch khi cần *nhiều* prime nhỏ. Cần *một* prime lớn (2048-bit)? Sieve cần 2²⁰⁴⁸ ô nhớ — phi lý; Miller-Rabin từng ứng viên là lựa chọn duy nhất. Cùng bài toán "tìm prime", cấu trúc truy vấn quyết định thuật toán — không có "thuật toán prime tốt nhất" chung.

**Overflow: khai thác hay phòng thủ.** Số học modular của uint là *tính năng* khi bạn chủ ý (hash, ring buffer, sequence number quấn vòng); là *bug* khi vô ý (Ariane 5, tính tiền, đo thời lượng). Go chọn không panic khi tràn (khác Rust debug mode) — nhanh, nhưng chuyển toàn bộ trách nhiệm nhận diện sang lập trình viên. Ranh giới kỷ luật: mọi phép nhân/cộng trên giá trị từ bên ngoài phải có câu trả lời cho câu hỏi "lớn nhất là bao nhiêu?"; `math/big` khi câu trả lời là "không chặn được".

## 7. Production Applications

**TLS/RSA key generation** — mỗi lần chạy `openssl genrsa` hay Go `rsa.GenerateKey`: sinh số lẻ ngẫu nhiên 1024-bit từ nguồn ngẫu nhiên mật mã, Miller-Rabin đến khi gặp prime (~vài trăm lần thử — Prime Number Theorem bảo đảm không chờ lâu), lấy hai prime p, q, tính N = pq, dùng extended Euclid tính d = e⁻¹ mod φ(N). Toàn bộ mục 3–5 của chương gói trong một API call.

**Consistent hashing ring** (Cassandra, DynamoDB, memcached client): không gian hash là vòng tròn ℤ/2⁶⁴ℤ đúng nghĩa đen — vị trí node và key đều là điểm trên vòng, "node kế tiếp theo chiều kim đồng hồ" là phép so sánh modular. Toán nằm ở chỗ: nhờ tính vòng, thêm/bớt node chỉ xê dịch ranh giới cục bộ thay vì reshuffle toàn bộ như `hash % n` (khi n đổi, *mọi* phần dư đổi — thảm họa cache miss đồng loạt).

**Snowflake ID** (Twitter, và các biến thể ở mọi công ty): ID 64-bit = 41 bit timestamp ‖ 10 bit machine ID ‖ 12 bit sequence. Ghép/tách bằng shift và mask — chính là biểu diễn số theo hệ cơ số hỗn hợp; sequence quấn vòng mod 2¹² mỗi millisecond; 41 bit timestamp là lời cam kết modular có ngày đáo hạn: tràn sau ~69 năm kể từ epoch tự chọn. Ai thiết kế ID scheme đều đang làm số học modular, tự giác hay không.

**Rate limiter cửa sổ thời gian:** `bucket := now.Unix() % int64(windowCount)` — thời gian tuyến tính vô hạn gập thành vòng hữu hạn các counter tái sử dụng; fixed window và sliding window log khác nhau ở cách xử lý ranh giới vòng. Ring buffer của Kafka consumer, LMAX Disruptor: cùng một phép gập.

**Go: `math/big` và cặp `crypto/rand` vs `math/rand`.** `math/big` tính trên số nguyên độ dài tùy ý — mọi phép mod/exp/gcd của RSA đi qua nó, kèm các biến thể constant-time chống timing attack. Và một phân biệt **sống còn**: `math/rand` là PRNG *dự đoán được* (biết vài output là suy ra state, suy ra mọi output tương lai) — dùng cho jitter, shuffle, sampling; `crypto/rand` lấy entropy từ OS — **bắt buộc** cho key, token, session ID, nonce. Sinh token đặt lại mật khẩu bằng `math/rand` là lỗ hổng kinh điển có tên trong nhiều báo cáo pentest; kẻ tấn công tự tính ra token của nạn nhân. Một chữ `crypto/` khác biệt giữa ngẫu nhiên "trông có vẻ" và ngẫu nhiên "không ai đoán nổi".

## 8. Interview

**Pow(x, n)** (LeetCode 50): fast exponentiation ở mục 5, thêm hai bẫy. Bẫy một: n âm → x⁻ⁿ = 1/xⁿ, nhưng đừng viết `n = -n` trước khi xử lý — `n = math.MinInt32` thì `-n` **tràn** (giá trị −2³¹ không có bản dương trong int32; hãy nới lên int64 hoặc xử lý riêng). Bẫy hai: viết đệ quy kiểu `Pow(x, n/2) * Pow(x, n/2)` — gọi *hai lần* thay vì lưu một lần rồi bình phương, độ phức tạp phình ngược về O(n). Bài này nhỏ nhưng đo được cả ba thứ: hiểu bình phương liên tiếp, cảnh giác overflow, và nhìn ra cây đệ quy (chương 06).

**Count Primes** (LeetCode 204): sieve ở mục 5. Điểm cộng khi giải thích được *vì sao* bắt đầu sàng từ p² và vì sao ngoài vòng chỉ chạy tới √n — hai tối ưu cùng đến từ một sự thật "hợp số có ước ≤ căn của nó".

**GCD of Strings** (LeetCode 1071): "chuỗi chia hết cho chuỗi con lặp" có cấu trúc y hệt số nguyên — đáp án là prefix độ dài gcd(len(a), len(b)), với phép kiểm tra a+b == b+a đóng vai trò "tính giao hoán ⟺ cùng đơn vị lặp". Bài toán đo khả năng **nhận ra cấu trúc toán quen trong lốt lạ** — kỹ năng đáng giá nhất mục Interview của mọi chương.

**Happy Number** (LeetCode 202): dãy "tổng bình phương chữ số" hoặc về 1, hoặc **rơi vào chu trình** — vì mọi số ≤ 3 chữ số nào đó thì bị chặn, không gian trạng thái hữu hạn, mà quỹ đạo trong không gian hữu hạn buộc phải lặp (pigeonhole — chương 05). Phát hiện chu trình bằng Floyd (rùa và thỏ) cho O(1) bộ nhớ. Cùng họ với phát hiện vòng trong linked list và trong dãy giả ngẫu nhiên (thuật toán Pollard's rho phân tích thừa số — cũng là "rho" đó).

**Lỗi tư duy phổ biến — hai lỗi đặc trưng của chương này:**

- **Nhân trước, mod sau:** `(a * b) % m` với a, b ~ 10¹⁸ tràn ở phép nhân *trước khi* kịp mod — kết quả sai lặng lẽ, test nhỏ vẫn xanh. Kỷ luật: mod từng toán hạng trước, và nếu tích hai toán hạng vẫn có thể vượt 64 bit thì dùng `bits.Mul64`/`math/big` (như `mulMod` ở mục 5).
- **Mod số âm trong Go:** `-7 % 3 == -1`, không phải `2` như Python. Go (theo C) làm tròn thương về 0, phần dư mang dấu số bị chia. `hash % n` với hash âm → index âm → panic ngoài giờ hành chính. Chuẩn hóa: `((x % n) + n) % n`, hoặc dùng kiểu unsigned từ đầu.

**Từ phỏng vấn sang production:** Pow chính là lõi của mọi phép RSA/Diffie-Hellman bạn gọi qua `crypto/tls` mỗi ngày; Happy Number + Floyd là kỹ thuật dò vòng dùng thật trong phân tích thừa số và dò trạng thái lặp; Count Primes dạy cách *tiền tính hàng loạt* thắng *truy vấn từng cái* — cùng trade-off với việc build index.

## 9. Anti-pattern

**Tin rằng int "đủ lớn nên không bao giờ tràn".** Ariane 5 tin thế với 16 bit; YouTube tin thế với 32 bit; nhiều hệ thống tiền tệ tin thế khi nhân `số_lượng × đơn_giá × tỷ_giá_scaled`. Số nào đến từ thế giới bên ngoài đều lớn hơn bạn nghĩ, và tích của chúng lớn nhanh hơn nữa. Hoặc chứng minh được chặn trên, hoặc dùng `math/big`/kiểm tra tràn tường minh.

**`hash % n` với n thay đổi làm sharding key.** Thêm một shard, n → n+1, gần như *toàn bộ* key đổi chỗ — cache lạnh đồng loạt, DB gánh bão traffic. Đây không phải bug code mà là bug *toán*: phần dư không ổn định theo modulus. Consistent hashing (vòng ℤ/2⁶⁴ℤ) hoặc jump hash sinh ra chính là để sửa tính chất này.

**`math/rand` cho bất kỳ thứ gì mang tính bảo mật.** Token, OTP seed, session ID, nonce của chữ ký — mỗi năm đều có CVE mới thuộc dòng này. PRNG dự đoán được nghĩa là kẻ tấn công *tính* ra bí mật thay vì đoán. Quy tắc không ngoại lệ: chạm đến security → `crypto/rand`.

**Tự chế crypto từ số học của chương này.** Hiểu Fermat, fast exponentiation rồi tự viết RSA cho production là con đường ngắn đến thảm họa: RSA "textbook" (không padding) vỡ trước hàng loạt tấn công đã biết; phép mod không constant-time rò rỉ khóa qua timing. Số học trong chương để *hiểu* thư viện, không phải để *thay* thư viện. Dùng `crypto/*` — được viết bởi người biết cả toán lẫn các cách toán bị phản bội trên phần cứng thật.

**Mod số âm không chuẩn hóa.** Đã nêu ở mục 8 nhưng đáng nhắc lần hai vì nó qua mặt code review dễ dàng: mọi biểu thức `x % n` mà x có thể âm là một `index out of range` đang ủ bệnh.

**Kiểm tra prime bằng trial division cho số lớn.** O(√n) với n 1024-bit là 2⁵¹² phép chia — nhầm lẫn giữa "thuật toán đúng" và "thuật toán khả thi", đúng bài học bảng ngưỡng của chương 10.

## 10. Best Practices

**Nên:**

- Với mỗi phép toán số nguyên trên dữ liệu ngoài, trả lời tường minh: giá trị lớn nhất có thể là bao nhiêu, tích/tổng có vượt 63 bit không? Ghi câu trả lời vào comment nếu nó không hiển nhiên.
- Mod sớm và mod thường xuyên trong chuỗi tính toán modular — tính chất phân phối cho phép, và nó giữ mọi giá trị trung gian nhỏ.
- Chuẩn hóa phần dư khi toán hạng có thể âm: `((x % n) + n) % n`, hoặc thiết kế bằng unsigned ngay từ kiểu dữ liệu.
- Chọn đúng công cụ theo cấu trúc truy vấn: nhiều prime nhỏ → sieve; một prime lớn → Miller-Rabin; số vượt 64 bit → `math/big`; bảo mật → `crypto/rand` và `crypto/*`, không bàn cãi.
- Khi thiết kế ID/sequence/counter, tính ngày đáo hạn của số bit đã cấp (41 bit millisecond ≈ 69 năm) và ghi vào tài liệu thiết kế — người kế nhiệm bạn sẽ biết ơn.

**Không nên:**

- Không nhân hai số 64-bit rồi mới mod khi modulus lớn — dùng `bits.Mul64`/`math/big`.
- Không dùng `hash % n` làm chiến lược phân mảnh cho hệ thống mà n sẽ thay đổi.
- Không tự implement primitive mật mã cho production, kể cả khi — nhất là khi — bạn vừa hiểu toán của nó.
- Không dùng mod 2ᵏ cho hash đa thức yếu; không trả phí mod prime khi hash phía trước đã trộn tốt — hiểu mình đang đứng ở phía nào của trade-off.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Vì sao two's complement cho phép phần cứng dùng chung một mạch cộng cho số âm và số dương? Trả lời bằng ngôn ngữ ℤ/2ⁿℤ.
2. Miller-Rabin với k = 40 có sai số ≤ 4⁻⁴⁰. Lập luận nào biện minh cho việc cả nền HTTPS chấp nhận "prime xác suất" thay vì đòi kiểm tra tất định?
3. Một đồng nghiệp viết `shard := int(hash) % numShards` trong Go, hash là int64 từ thư viện bên ngoài, numShards sắp tăng từ 8 lên 10. Chỉ ra *hai* quả bom trong một dòng code này.

---

*Chương tiếp theo: [20 — Information Theory](/series/math-for-engineers/level-4-advanced-mathematics/20-information-theory/), nơi câu hỏi đổi từ "tính toán trên số như thế nào" sang "thông tin thực chất nặng bao nhiêu bit" — entropy, mã hóa Huffman, và giới hạn toán học của mọi thuật toán nén.*
