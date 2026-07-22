+++
title = "Chương 26 — Cryptography Mathematics: Bảo mật xây trên nền toán học"
date = "2026-07-20T11:20:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 5 – Production
> Yêu cầu trước: Chương 11 (Probability), Chương 12 (Hashing), Chương 19 (Number Theory)

---

## 1. Problem Statement

Hãy tưởng tượng tình huống này: bạn và một người lạ chưa từng gặp nhau, chưa từng trao đổi bất kỳ bí mật nào, đứng ở hai đầu một căn phòng đông người. Mọi câu các bạn nói đều bị cả phòng nghe thấy. Nhiệm vụ: hai người phải thỏa thuận được một con số bí mật mà **không ai khác trong phòng biết** — dù họ nghe được từng từ.

Nghe như câu đố mẹo không có lời giải. Mọi thông tin bạn gửi đi đều công khai; kẻ nghe lén có đúng những dữ kiện mà đối phương của bạn có. Làm sao đối phương suy ra được bí mật mà kẻ nghe lén thì không?

Vậy mà bạn vừa làm điều đó vài giây trước — khi trình duyệt mở trang web này qua HTTPS. Máy của bạn và server chưa từng "quen biết", mọi gói tin đi qua hàng chục router có thể bị đọc trộm, và sau vài millisecond hai bên đã có chung một khóa mã hóa mà không kẻ nghe lén nào tính ra được.

Đây chính là bài toán trung tâm của chương này:

**Làm sao tạo ra bí mật chung, xác thực danh tính, và bảo đảm toàn vẹn dữ liệu — trên một kênh truyền hoàn toàn công khai?**

Nếu không có lời giải toán học cho bài toán này, thương mại điện tử không tồn tại: mọi cặp máy muốn nói chuyện riêng tư sẽ phải trao khóa trước qua một kênh an toàn vật lý (gặp mặt, chuyển phát tín cẩn) — mô hình "valise ngoại giao" mà các đại sứ quán dùng suốt thế kỷ 20, và hoàn toàn không thể scale cho hàng tỷ kết nối TLS mỗi giây.

Điều đáng kinh ngạc: lời giải không đến từ kỹ thuật giấu giếm tinh vi hơn, mà từ một quan sát toán học — **có những phép toán dễ làm theo một chiều nhưng khó đảo ngược đến mức bất khả thi trong thực tế**.

## 2. Trực giác

### Trộn màu sơn

Trực giác đơn giản nhất cho trao đổi khóa là thí nghiệm trộn sơn:

1. Alice và Bob công khai thống nhất một màu **chung** — vàng.
2. Mỗi người chọn một màu **bí mật** riêng: Alice chọn đỏ, Bob chọn xanh.
3. Mỗi người trộn màu bí mật của mình với màu vàng chung, rồi gửi hỗn hợp qua kênh công khai: Alice gửi cam (vàng+đỏ), Bob gửi lục (vàng+xanh).
4. Mỗi người trộn màu bí mật của mình vào hỗn hợp nhận được: Alice có (vàng+xanh)+đỏ, Bob có (vàng+đỏ)+xanh — **cùng một màu nâu**.

Eve nghe lén thấy: vàng, cam, lục. Muốn có màu nâu, cô phải **tách** cam trở lại thành vàng+đỏ — nhưng trộn sơn thì dễ, tách sơn thì gần như không thể. Toàn bộ bí mật nằm ở sự **bất đối xứng giữa hai chiều của một phép toán**.

Mật mã hiện đại thay "trộn sơn" bằng các phép toán số học có đúng tính chất đó: lũy thừa modulo dễ tính (chương 19 — fast exponentiation, O(log n) phép nhân) nhưng phép ngược của nó — logarit rời rạc — thì chưa ai biết cách tính nhanh.

### Hai nguyên lý nền móng

**One-way function** — hàm một chiều: tính f(x) từ x thì rẻ, tìm x từ f(x) thì đắt đến mức vô vọng. Hai ứng viên kinh điển:

| Chiều dễ | Chiều khó | Hệ mật dựa trên |
|---|---|---|
| Nhân hai số nguyên tố lớn: p × q = n | Phân tích n ngược lại thành p, q (factoring) | RSA |
| Lũy thừa modulo: gᵃ mod p | Tìm a từ gᵃ mod p (discrete logarithm) | Diffie-Hellman, ECC |

Cần nói thẳng một sự thật mà nhiều tài liệu lướt qua: **chưa ai chứng minh được one-way function tồn tại**. Nếu chứng minh được, ta lập tức có P ≠ NP — một trong bảy bài toán thiên niên kỷ. Toàn bộ mật mã hiện đại đứng trên các **giả định độ khó**: "nhân loại đã cố phân tích thừa số hiệu quả suốt hàng thế kỷ (từ Gauss) và chưa ai làm được". Đó là bằng chứng thực nghiệm rất mạnh, nhưng không phải chứng minh. Kỹ sư trưởng thành cần hiểu: bảo mật là **kinh tế học của độ khó tính toán** — ta không làm việc phá mã thành bất khả thi, ta làm nó đắt hơn giá trị của thứ được bảo vệ, trong khoảng thời gian thứ đó còn giá trị.

**Kerckhoffs's principle** (1883): hệ mật phải an toàn ngay cả khi kẻ tấn công biết **mọi thứ trừ khóa**. Bảo mật nằm ở key, không nằm ở thuật toán. Lý do rất thực dụng: thuật toán không thể giữ bí mật lâu (reverse engineering, nhân viên nghỉ việc, source code lộ), nhưng khóa thì thay được trong một giây. Đây cũng là lý do các thuật toán tốt nhất đều công khai và bị soi hàng chục năm — AES, SHA-3 đều là kết quả của cuộc thi công khai. Một "thuật toán mã hóa độc quyền bí mật" trong sản phẩm thương mại gần như luôn là red flag.

### Nếu không có các khái niệm này thì sao?

Không có one-way function: mọi khóa phải phân phối trước qua kênh vật lý an toàn — với n bên liên lạc cần quản lý O(n²) khóa chia sẻ (chương 05 — đếm số cặp). Không có Kerckhoffs: mỗi lần thuật toán lộ (chắc chắn sẽ lộ), toàn bộ hệ thống phải đập đi xây lại thay vì chỉ rotate key.

## 3. First Principles

Xây lại từ gốc: ta cần một phép toán trên số nguyên sao cho chiều xuôi rẻ, chiều ngược đắt. Tại sao lũy thừa **modulo** làm được điều mà lũy thừa thường không làm được?

Xét hàm f(a) = gᵃ trên số thực: hàm đơn điệu tăng, mượt mà — biết gᵃ thì tìm a bằng logarit thường, hoặc thô hơn là binary search, vì giá trị hàm "tiết lộ" nó đang ở đâu. Nhưng thêm `mod p` vào, mọi cấu trúc trơn tru biến mất:

```
g = 5, p = 23:  a =  1   2   3   4   5   6   7   8   9  10 ...
             5^a mod 23 =  5   2  10   4  20  15   6   7  12  14 ...
```

Chuỗi giá trị nhảy loạn xạ trong [1, 22], không đơn điệu, không có gradient để lần theo. Biết gᵃ mod p = 15 không cho manh mối nào rằng a "khoảng bao nhiêu" — binary search vô dụng, và với p cỡ 2048 bit, thử lần lượt tốn 2²⁰⁴⁸ bước. Modular arithmetic (chương 19) chính là cách **gấp** một hàm trơn thành mê cung: phép tính xuôi vẫn rẻ (square-and-multiply chỉ cần ~log₂ a bước), nhưng mọi thông tin định hướng cho chiều ngược bị nghiền nát.

Đây là first principle của toàn chương: **mật mã khai thác khoảng cách giữa độ phức tạp của hai chiều một phép toán** — O(log n) chiều xuôi, (được tin là) siêu đa thức chiều ngược. Toàn bộ phần còn lại — DH, RSA, ECC — chỉ là ba cách khác nhau dựng giao thức trên khoảng cách đó. Và hash function mật mã là phiên bản cực đoan: chiều ngược không những khó mà còn *không tồn tại* (hàm nén, mất thông tin) — muốn "đảo" chỉ còn cách đoán.

## 4. Mathematical Model

### Diffie-Hellman: trộn sơn bằng số học

Giao thức (1976) dịch thí nghiệm trộn sơn sang lũy thừa modulo. Công khai: số nguyên tố p và phần tử sinh g. Đi từng bước với **g = 5, p = 23**:

```
                Alice                    kênh công khai                Bob
  bí mật:       a = 6                                                b = 15
  tính:         A = 5^6 mod 23                                 B = 5^15 mod 23
                  = 15625 mod 23 = 8                              = ... = 19
  gửi:          ────────── A = 8 ──────────▶
                ◀────────── B = 19 ─────────
  khóa chung:   K = B^a mod 23             K = A^b mod 23
                  = 19^6 mod 23 = 2          = 8^15 mod 23 = 2
```

Hai bên cùng ra **K = 2** vì lũy thừa giao hoán được với nhau:

> Bᵃ = (gᵇ)ᵃ = g^(ab) = (gᵃ)ᵇ = Aᵇ (mod p)

Đó là toàn bộ "phép màu": g^(ab) tính được theo hai đường, mỗi bên đi một đường bằng bí mật riêng của mình.

**Vì sao Eve bó tay?** Eve thấy p = 23, g = 5, A = 8, B = 19. Muốn có K = g^(ab), cô cần a hoặc b — tức phải giải **discrete logarithm**: tìm a sao cho 5ᵃ ≡ 8 (mod 23). Với p = 23 thì thử 22 lần là xong; với p 2048-bit, thuật toán tốt nhất được biết (index calculus / GNFS) vẫn cần thời gian dưới-hàm-mũ nhưng trên-đa-thức — hàng tỷ năm CPU. Khoảng cách giữa "Alice tính 11 phép bình phương" và "Eve tính 2¹⁰⁰⁺ bước" chính là toàn bộ an ninh của giao thức.

**Lỗ hổng cấu trúc: man-in-the-middle.** DH chống nghe lén *thụ động*, nhưng nếu Mallory đứng giữa và **thay được gói tin**, cô chạy hai phiên DH riêng — một với Alice (giả làm Bob), một với Bob (giả làm Alice) — rồi ngồi giữa giải mã, đọc, mã hóa lại. Alice không có cách nào biết "8 = gᵇ này" đến từ Bob hay từ Mallory, vì **DH trao đổi bí mật nhưng không xác thực danh tính**. Bài học kiến trúc quan trọng: key exchange phải đi kèm authentication — đó là việc của chữ ký số và certificate, ta gặp lại ở phần TLS.

### RSA: từ định lý Euler đến public-key encryption

DH tạo khóa chung; RSA (1977) đi xa hơn — cho phép **bất kỳ ai** mã hóa gửi bạn, chỉ **mình bạn** giải mã được. Nền móng là định lý Euler (chương 19):

> Nếu gcd(m, n) = 1 thì m^φ(n) ≡ 1 (mod n), với φ(n) là số các số nguyên dương ≤ n nguyên tố cùng nhau với n.

Xây dựng từng bước:

1. Chọn hai số nguyên tố p, q; đặt **n = pq**. Khi đó φ(n) = (p−1)(q−1) — đếm bằng inclusion-exclusion (chương 05): loại các bội của p và của q.
2. Chọn e nguyên tố cùng nhau với φ(n); tìm d sao cho **e·d ≡ 1 (mod φ(n))** — bằng extended Euclid (chương 19).
3. Công khai (n, e); giữ bí mật d. Mã hóa: c = mᵉ mod n. Giải mã: cᵈ mod n.

Vì sao giải mã trả lại đúng m? Vì e·d = 1 + k·φ(n) với k nào đó, nên:

> cᵈ = m^(ed) = m^(1 + k·φ(n)) = m · (m^φ(n))ᵏ ≡ m · 1ᵏ = m (mod n)

Toàn bộ RSA là **một dòng hệ quả của định lý Euler**. Điểm then chốt: ai biết φ(n) thì tính được d từ e trong tích tắc — mà tính φ(n) = (p−1)(q−1) đòi hỏi biết p, q, tức là **phân tích thừa số n**. An ninh RSA = độ khó của factoring.

Walk through với số nhỏ (ví dụ kinh điển): p = 61, q = 53 → n = 3233, φ(n) = 60 × 52 = 3120. Chọn e = 17; extended Euclid cho d = 2753 (kiểm tra: 17 × 2753 = 46801 = 15 × 3120 + 1 ✓). Mã hóa m = 65: c = 65¹⁷ mod 3233 = **2790**. Giải mã: 2790²⁷⁵³ mod 3233 = **65** ✓. Cả hai phép lũy thừa với số mũ hàng nghìn đều xong trong ~12 và ~17 phép nhân modulo nhờ fast exponentiation (chương 19) — không có thuật toán O(log e) đó, RSA chỉ là trò chơi trên giấy.

**Textbook RSA vs RSA thực tế.** Phiên bản vừa trình bày gọi là *textbook RSA* và **không được dùng nguyên trạng** vì nó deterministic: cùng m luôn cho cùng c. Hệ quả chết người — kẻ tấn công biết không gian bản rõ nhỏ (ví dụ "YES"/"NO", hay số thẻ 16 chữ số) chỉ cần mã hóa thử mọi khả năng bằng public key rồi so với c. Thêm nữa, RSA có tính nhân (c₁·c₂ = (m₁·m₂)ᵉ) nên kẻ tấn công có thể "nhào nặn" ciphertext đại số. Thực tế phải dùng **padding có ngẫu nhiên** — chuẩn hiện đại là OAEP: trộn m với random bytes qua hai lượt hash trước khi lũy thừa, biến RSA thành xác suất hóa và phá vỡ cấu trúc đại số. Mức khái niệm cần nhớ: *thiếu padding không phải là "kém an toàn hơn một chút" mà là vỡ hoàn toàn trước tấn công rất rẻ.*

### ECC: nhóm điểm trên đường cong

RSA có vấn đề tăng trưởng: thuật toán factoring tốt nhất (GNFS) chạy dưới-hàm-mũ, nên mỗi lần máy tính mạnh lên, key RSA phải phình ra *nhanh hơn tuyến tính*. Elliptic Curve Cryptography đổi sân chơi: thay nhóm số (Z_p, ×) bằng **nhóm các điểm trên đường cong** y² = x³ + ax + b trên trường hữu hạn.

Phép toán nhóm định nghĩa bằng hình học thuần túy: muốn "cộng" hai điểm P và Q, kẻ đường thẳng qua chúng — đường này cắt đường cong tại đúng một điểm thứ ba R'; lấy đối xứng R' qua trục hoành được R = P + Q:

```
        y │        ╭──╮
          │   Q   ╱    R' (giao điểm thứ 3)
          │  ●───●────●
          │ ╱   ╱      ╲
          │● P ╱        │
          │  ╲│         │  lật qua trục x
        ──┼───┼─────────┼────── x
          │   │         │
          │   ╰╮        ●  R = P + Q
          │    ╰───────╯
```

Có thể kiểm chứng phép cộng này thỏa mọi tiên đề nhóm (kết hợp, phần tử trung hòa, nghịch đảo — chương 02 về cấu trúc quan hệ). Từ phép cộng, định nghĩa **nhân vô hướng**: kP = P + P + ... + P (k lần), tính nhanh bằng double-and-add — đúng cấu trúc square-and-multiply của chương 19, O(log k) bước.

One-way function của ECC: **biết P và Q = kP, tìm k** (elliptic curve discrete logarithm — ECDLP). Điểm ăn tiền: các kỹ thuật dưới-hàm-mũ phá discrete log trên số (index calculus) **không hoạt động** trên đường cong, vì nhóm điểm không có cấu trúc "số nguyên tố nhỏ" để khai thác. Thuật toán tốt nhất còn lại là loại √n generic (Pollard rho — chính là birthday bound √ của chương 12). Hệ quả: độ an toàn k bit chỉ cần nhóm cỡ 2^2k — key ngắn hơn RSA một trời một vực:

| Mức an toàn (bit) | RSA key | ECC key | Tỷ lệ |
|---|---|---|---|
| 80 | 1024 | 160 | 6.4× |
| 112 | 2048 | 224 | 9.1× |
| **128** | **3072** | **256** | **12×** |
| 192 | 7680 | 384 | 20× |
| 256 | 15360 | 521 | 29× |

Key ngắn → chữ ký ngắn, handshake ít byte, tính toán nhanh — lý do TLS hiện đại, SSH, Signal đều mặc định ECC. Đường cong phổ biến nhất là **Curve25519** (Daniel Bernstein): tham số chọn minh bạch "nothing up my sleeve", cài đặt tự nhiên chạy constant-time, tránh cả các bẫy cài đặt lẫn nghi ngờ backdoor từng phủ bóng các đường cong NIST.

### Hash function mật mã: one-way không cần cửa hậu

DH và RSA là one-way function *có trapdoor* (biết bí mật thì đảo được). Hash mật mã là one-way **tuyệt đối** — không ai đảo được, kể cả người tạo. Ba tính chất, xếp từ yếu đến mạnh:

| Tính chất | Phát biểu | Kẻ tấn công cần |
|---|---|---|
| Preimage resistance | cho h, khó tìm m sao cho H(m) = h | ~2ⁿ phép thử |
| Second preimage | cho m₁, khó tìm m₂ ≠ m₁ cùng hash | ~2ⁿ phép thử |
| Collision resistance | khó tìm *bất kỳ* cặp m₁ ≠ m₂ cùng hash | **~2^(n/2)** phép thử |

Quan hệ suy diễn: phá được second preimage thì phá được collision (chỉ việc dùng m₂ tìm được); nên collision resistance là tính chất **mạnh nhất** — mất nó trước tiên. Vì sao collision chỉ cần 2^(n/2)? Đây chính là **birthday paradox** (chương 11, 12): kẻ tấn công không cần khớp một giá trị định trước, chỉ cần *hai giá trị bất kỳ khớp nhau* — số cặp tăng theo bình phương số phép thử, nên xác suất đụng độ chạm 50% ở √(2ⁿ) = 2^(n/2). Hệ quả thiết kế mà giờ bạn giải thích được từ gốc: **SHA-256 cho 256-bit preimage resistance nhưng chỉ 128-bit collision resistance** — và 128 bit là lý do nó vẫn được coi là đủ, trong khi MD5 (64-bit collision) và SHA-1 (80-bit, đã bị phá thực tế năm 2017 với chi phí ~2⁶³) phải khai tử khỏi mọi ngữ cảnh cần chống collision.

**HMAC — vì sao hash(key‖msg) chưa đủ?** Muốn xác thực message bằng khóa chung, cách ngây thơ là gửi tag = SHA-256(key‖msg). Nhưng SHA-256 thuộc họ Merkle–Damgård: hash được tính bằng cách nuốt từng block và **trạng thái nội bộ cuối cùng chính là digest**. Kẻ tấn công cầm tag hợp lệ có thể *tiếp tục* băm từ trạng thái đó — tính được SHA-256(key‖msg‖padding‖extra) cho phần extra tùy ý **mà không cần biết key**. Đó là length extension attack: ví dụ API nhận `?cmd=read&sig=...` có thể bị nối thêm `&cmd=delete` với chữ ký vẫn hợp lệ. HMAC vá triệt để bằng hai lớp hash lồng nhau: HMAC(K, m) = H((K⊕opad) ‖ H((K⊕ipad) ‖ m)) — lớp ngoài "niêm phong" trạng thái nội bộ, chấm dứt trò nối đuôi. Bài học: đừng chế công thức MAC — dùng `crypto/hmac`.

**Password hashing: hash mà PHẢI chậm.** Mọi hash ở trên được thiết kế nhanh nhất có thể — và đó chính là thảm họa cho lưu password. Khi database lộ, kẻ tấn công chạy offline dictionary attack; GPU hiện đại thử **hàng chục tỷ** SHA-256 mỗi giây, quét sạch phần lớn password thực tế trong vài giờ. Password hash (bcrypt, scrypt, Argon2) đảo ngược mục tiêu thiết kế: **cố tình đắt** — lặp hàng nghìn vòng (cost factor điều chỉnh được, tăng dần theo định luật Moore) và với Argon2 còn ngốn RAM để vô hiệu hóa GPU/ASIC (memory-hard). Cùng chi phí 100ms: người dùng đăng nhập không cảm nhận được, kẻ tấn công từ 10 tỷ lần thử/giây tụt còn 10 lần/giây — chậm đi **một tỷ lần**. Kèm salt ngẫu nhiên cho mỗi user để hai người trùng password không trùng hash và rainbow table thành vô dụng (chương 12).

### Chữ ký số: phép mã hóa lộn ngược

Nhìn lại RSA: hai khóa e, d đối xứng nhau qua quan hệ ed ≡ 1 (mod φ(n)). Đảo vai trò: dùng khóa **private** để "mã hóa" digest của tài liệu — s = H(m)ᵈ mod n — thì **bất kỳ ai** cầm public key đều verify được: sᵉ mod n có bằng H(m) không. Chỉ người giữ d tạo được s hợp lệ, nhưng cả thế giới kiểm tra được — đúng nghĩa "chữ ký": khó giả mạo, dễ kiểm chứng, và không chối bỏ được (non-repudiation). Ký trên H(m) thay vì m vừa nhanh vừa an toàn — miễn là hash chống collision, vì hai tài liệu cùng digest sẽ dùng chung được một chữ ký (đây là đòn đã khai tử MD5 trong certificate: kẻ tấn công tạo cặp certificate collision, xin ký bản lành, dùng chữ ký cho bản độc). Trong thực tế chữ ký dùng ECDSA/Ed25519 (nhanh, ngắn) nhiều hơn RSA, nhưng nguyên lý "ký bằng private, verify bằng public" không đổi.

## 5. Thuật toán

Xâu chuỗi các khối toán học trên thành code chạy được. Cảnh báo bắt buộc trước tiên:

> ⚠️ **Code dưới đây chỉ minh họa khái niệm toán học** — số nhỏ, textbook RSA, không padding, không constant-time. **Không bao giờ tự cài crypto cho production** — dùng `crypto/tls`, `crypto/rsa`, `crypto/ecdsa`, `crypto/hmac`, `golang.org/x/crypto/argon2` của Go, những thư viện đã được audit và xử lý vô số bẫy cài đặt (timing, padding, RNG) mà code minh họa bỏ qua.

Trái tim của mọi thứ là fast exponentiation (chương 19):

```go
// modExp tính base^exp mod m bằng square-and-multiply — O(log exp) phép nhân.
// Không có hàm này, RSA/DH với số mũ 2048-bit là bất khả thi.
func modExp(base, exp, m uint64) uint64 {
    result := uint64(1)
    base %= m
    for exp > 0 {
        if exp&1 == 1 {          // bit thấp nhất của exp là 1
            result = result * base % m
        }
        base = base * base % m   // bình phương liên tiếp: g, g², g⁴, g⁸...
        exp >>= 1
    }
    return result
}

// Diffie-Hellman với số nhỏ — đúng ví dụ ở mục 4.
func demoDH() {
    p, g := uint64(23), uint64(5)      // công khai
    a, b := uint64(6), uint64(15)      // bí mật của Alice, Bob
    A := modExp(g, a, p)               // Alice gửi 8
    B := modExp(g, b, p)               // Bob gửi 19
    kAlice := modExp(B, a, p)          // 19^6  mod 23 = 2
    kBob := modExp(A, b, p)            // 8^15  mod 23 = 2
    fmt.Println(kAlice == kBob)        // true — khóa chung, Eve không tính được
}

// Textbook RSA — CHỈ để kiểm chứng định lý Euler bằng số của mục 4.
func demoRSA() {
    n := uint64(3233)                  // 61 × 53, công khai
    e, d := uint64(17), uint64(2753)   // e·d ≡ 1 (mod 3120)
    m := uint64(65)
    c := modExp(m, e, n)               // encrypt: 2790
    fmt.Println(modExp(c, d, n) == m)  // decrypt: true
}
```

### Merkle tree: hash của hash

Bây giờ đến cấu trúc dữ liệu quan trọng nhất sinh ra từ hash mật mã. Bài toán: có n block dữ liệu, muốn (1) một "vân tay" duy nhất cho toàn bộ, và (2) chứng minh một block thuộc tập hợp **mà không gửi cả tập hợp**. Giải pháp của Ralph Merkle (1979): băm từng block làm lá, rồi băm từng cặp đi lên đến gốc:

```
                     root = H(H12 ‖ H34)
                    ╱                    ╲
            H12 = H(H1‖H2)         H34 = H(H3‖H4)
             ╱        ╲              ╱        ╲
        H1=H(D1)  H2=H(D2)      H3=H(D3)  H4=H(D4)
           │         │             │         │
          D1        D2            D3        D4
```

**Membership proof cho D3**: người chứng minh chỉ gửi D3 kèm hai hash "anh em" trên đường lên gốc — [H4, H12]. Người kiểm tra tự tính: H3 = H(D3) → H34 = H(H3‖H4) → root' = H(H12‖H34), rồi so root' với root đã tin cậy. Khớp ⟹ D3 thuộc cây — vì muốn giả mạo, kẻ tấn công phải tìm collision ở đâu đó trên đường đi. Proof chỉ dài **O(log n)**: cây 1 tỷ lá cần đúng 30 hash (~1KB) thay vì gửi cả terabyte.

```go
// Xác minh Merkle proof: leo từ lá lên gốc, mỗi tầng băm với hash anh em.
// isLeft[i] cho biết sibling[i] nằm bên trái hay phải (thứ tự nối quan trọng!).
func verifyProof(leaf []byte, siblings [][]byte, isLeft []bool, root []byte) bool {
    h := sha256.Sum256(leaf)
    cur := h[:]
    for i, sib := range siblings {
        var pair []byte
        if isLeft[i] {
            pair = append(append([]byte{}, sib...), cur...) // sibling ‖ cur
        } else {
            pair = append(append([]byte{}, cur...), sib...) // cur ‖ sibling
        }
        next := sha256.Sum256(pair)
        cur = next[:]
    }
    return bytes.Equal(cur, root) // so sánh root tính lại với root tin cậy
}
```

Tính chất lan truyền then chốt: **đổi 1 bit ở bất kỳ lá nào → đổi mọi hash trên đường lên gốc → đổi root**. Root là commitment cho toàn bộ nội dung — nền tảng của Git, blockchain, certificate transparency (mục 7).

### TLS handshake: bản giao hưởng tổng hợp

Mọi mảnh ghép của chương hội tụ trong vài millisecond đầu của một kết nối HTTPS (TLS 1.3, giản lược):

```
Client                                                Server
  │── ClientHello: random, các cipher suite hỗ trợ,     │
  │   ECDHE public key (gᵃ trên Curve25519) ───────────▶│
  │                                                     │
  │◀── ServerHello: ECDHE public key (gᵇ)               │
  │    + Certificate (public key server, được CA ký)    │
  │    + CertificateVerify (chữ ký bằng private key     │
  │      của server trên transcript) ───────────────────│
  │                                                     │
  │  [cả hai tính khóa chung g^(ab) → derive AES key]   │
  │══════ dữ liệu mã hóa AES-GCM, xác thực tag ═════════│
```

Đọc luồng này bằng con mắt toán học: **ECDHE** (Diffie-Hellman trên elliptic curve, ephemeral — khóa mới mỗi phiên, lộ private key server cũng không giải mã được phiên đã ghi âm: forward secrecy) tạo bí mật chung; **chữ ký số + certificate** chặn đúng lỗ hổng man-in-the-middle của DH thuần — client verify chữ ký bằng public key trong certificate, certificate lại được CA ký, tạo chain of trust; rồi hai bên chuyển sang **AES** đối xứng vì mã hóa bất đối xứng đắt gấp hàng nghìn lần — bất đối xứng chỉ để "bắt tay", đối xứng để "nói chuyện". Đây gọi là hybrid encryption, và là mẫu thiết kế của gần như mọi giao thức bảo mật hiện đại.

## 6. Trade-off

**Bất đối xứng vs đối xứng.** RSA-2048 giải mã chậm hơn AES-128 khoảng 3–4 bậc độ lớn. Không hệ thống nào mã hóa dữ liệu lớn bằng RSA — public-key chỉ dùng trao khóa/ký, phần còn lại giao cho symmetric cipher. Trade-off này cứng đến mức nó là *kiến trúc mặc định* (hybrid) chứ không còn là lựa chọn.

**RSA vs ECC.** ECC thắng gần như mọi mặt (key ngắn 12×, ký nhanh, handshake nhẹ) nhưng RSA verify chữ ký *nhanh hơn* (e nhỏ, thường 65537) — lý do certificate root CA sống lâu vẫn hay là RSA: ký một lần, verify hàng tỷ lần. Chọn theo tỷ lệ sign/verify của workload.

**An toàn vs tốc độ — chọn cả hai đúng chỗ.** Hash cho checksum/cấu trúc dữ liệu phải nhanh; hash cho password phải chậm. Dùng nhầm hướng nào cũng trả giá: SHA-256 cho password là tặng quà cho GPU cracker; bcrypt cho HMAC API là tự bóp throughput. Cost factor của bcrypt/Argon2 là **nút vặn tường minh** trên trade-off này — và phải tăng theo thời gian, vì phần cứng của kẻ tấn công không đứng yên.

**Mức an toàn vs chi phí vận hành.** Tăng từ RSA-2048 lên RSA-4096 tăng chi phí handshake ~6–8 lần cho ~+16 bit an toàn; chuyển sang ECC P-256 được 128-bit với chi phí thấp hơn cả RSA-2048. Nhiều tổ chức tốn tiền sai chỗ: mua "key to hơn" trong khi lỗ hổng thật nằm ở quản lý key và code so sánh MAC không constant-time.

**Giả định độ khó vs tương lai lượng tử.** Thuật toán Shor trên máy tính lượng tử đủ lớn phá cả factoring lẫn discrete log (kể cả ECC) trong thời gian đa thức — hai giả định trụ cột sụp cùng lúc. Hash và AES chỉ yếu đi một nửa số bit (Grover), tăng size là xong. Đó là lý do NIST chuẩn hóa post-quantum (Kyber/ML-KEM dựa trên lattice) và TLS đang triển khai hybrid key exchange: cổ điển + hậu lượng tử song song. Bài học meta: vì an ninh dựa trên *giả định*, kiến trúc tốt phải có **crypto agility** — khả năng thay thuật toán mà không đập hệ thống.

## 7. Production Applications

**`crypto/tls` và hệ sinh thái Go.** Toàn bộ mục 5 đóng gói sau một dòng `http.ListenAndServeTLS`. Go cũng là ngôn ngữ hiếm có RNG mật mã làm mặc định đúng: token, key, nonce phải sinh từ `crypto/rand` (entropy từ OS) — `math/rand` là PRNG gieo được, đoán được state là đoán được mọi token quá khứ lẫn tương lai.

**JWT — nơi lỗi khái niệm thành CVE.** JWT = header.payload.signature, chữ ký là HMAC (HS256) hoặc chữ ký bất đối xứng (RS256/ES256). Hai lỗi kinh điển đều là lỗi *toán học bị hiểu sai*: (1) chấp nhận `alg: none` — token tự tuyên bố "tôi không cần chữ ký" và thư viện ngoan ngoãn bỏ verify; (2) nhầm lẫn HS256/RS256 — server verify RS256 bằng public key, kẻ tấn công đổi header thành HS256 và dùng **chính public key đó** (công khai!) làm khóa HMAC để ký token giả. Phòng thủ: server chỉ định cứng thuật toán chấp nhận, không bao giờ tin trường `alg` do client gửi — một áp dụng trực tiếp của Kerckhoffs: đừng để kẻ tấn công chọn thuật toán.

**API signature — AWS SigV4.** Mọi request AWS được ký bằng chuỗi HMAC-SHA256 lồng nhau: key ký ngày = HMAC(secret, date), rồi HMAC tiếp region, service, cuối cùng ký canonical request. Request tự chứng minh: đến từ người giữ secret, không bị sửa trên đường, không replay được quá 15 phút (timestamp nằm trong dữ liệu ký). Đây là HMAC của mục 4 nhân lên quy mô hàng nghìn tỷ request/ngày — và là mẫu chuẩn khi bạn thiết kế webhook/internal API signature.

**bcrypt trong auth service.** `golang.org/x/crypto/bcrypt` với cost 12 (~250ms) là baseline; Argon2id khi cần chống GPU mạnh hơn. Lưu ý vận hành hay bị quên: cost là tham số *sống* — hệ thống tốt re-hash password bằng cost mới ngay lần đăng nhập thành công kế tiếp, nâng cấp dần cả database mà không cần reset ai.

**Merkle tree khắp nơi.** **Git**: mỗi commit là hash của tree, tree chứa hash các blob/subtree — một Merkle DAG; đổi 1 file → đổi hash blob → đổi mọi tree cha → đổi commit hash, nên lịch sử là immutable-by-math và `git fetch` chỉ cần so hash để biết thiếu gì. **Cassandra/DynamoDB anti-entropy repair**: hai replica so cây Merkle của dải dữ liệu, đi xuống đúng nhánh có hash lệch — tìm diff của hàng tỷ row bằng O(log n) vòng trao đổi. **Certificate transparency**: mọi certificate được ghi vào append-only Merkle log công khai; CA gian lận cấp cert lén sẽ bị phát hiện vì cert không có proof trong log. **Bitcoin/Ethereum**: block header chứa Merkle root của các giao dịch — light client verify một giao dịch bằng proof O(log n) thay vì tải cả chain.

**Blockchain consensus — proof of work là bài toán preimage một phần.** Miner tìm nonce sao cho SHA-256(SHA-256(header‖nonce)) < target — tức digest phải mở đầu bằng ~k bit 0. Không có đường tắt (hash là one-way), chỉ còn brute force: kỳ vọng 2ᵏ phép thử (chương 11 — phân phối hình học). Mạng đo thời gian ra block thực tế và **điều chỉnh difficulty** mỗi 2016 block để giữ nhịp 10 phút — một vòng feedback control xây trên xác suất thuần túy. Chi tiết hơn ở chương 27.

## 8. Interview

Cryptography hiếm khi bị hỏi kiểu "cài RSA trên bảng trắng" — nhưng xuất hiện dày đặc trong **system design**: "thiết kế auth system", "bảo vệ API công khai", "thiết kế webhook an toàn". Người phỏng vấn kiểm tra bạn dùng đúng *khái niệm* cho đúng *bài toán*:

| Bài toán trong đề | Khái niệm đúng |
|---|---|
| Lưu password | bcrypt/Argon2 + salt — KHÔNG phải SHA-256 |
| Verify webhook/request | HMAC + timestamp chống replay |
| Session/API token | random từ `crypto/rand`, so sánh constant-time |
| Truyền dữ liệu giữa service | TLS (mTLS nếu cần xác thực hai chiều) — không tự chế |
| "Mã hóa hay hash?" | đảo được và cần đọc lại → mã hóa; chỉ cần so khớp → hash |

**Các lỗi tư duy chết người** — nói ra một trong số này trong phỏng vấn (hoặc tệ hơn, trong code review) là red flag lớn:

- **Tự cài crypto** ("tôi viết thuật toán mã hóa riêng cho chắc") — vi phạm Kerckhoffs và bỏ qua 40 năm cryptanalysis công khai.
- **`math/rand` cho token/session ID** — seed và state đoán được; đã có các vụ hack casino online đúng theo cách này.
- **So sánh MAC/token bằng `==`** — so sánh byte thường thoát sớm ở byte sai đầu tiên, thời gian phản hồi rò rỉ "khớp được bao nhiêu byte", cho phép dựng MAC hợp lệ từng byte một (timing attack). Go có sẵn thuốc: `subtle.ConstantTimeCompare`, và `hmac.Equal` dùng nó bên trong.
- **SHA-256 thuần cho password** — nhanh chính là điểm chết, như mục 4 đã phân tích.
- **"Encrypt là đủ toàn vẹn"** — mã hóa không chống sửa đổi; cần authenticated encryption (AES-GCM) hoặc encrypt-then-MAC.

Câu trả lời ghi điểm cao nhất luôn có dạng: nêu đúng primitive + nói được **giả định độ khó nào** đang gánh an ninh + chỉ ra chế độ thất bại ("HMAC an toàn *miễn là* secret không lộ và so sánh constant-time").

## 9. Anti-pattern

**Roll your own crypto — ở mọi tầng.** Không chỉ "tự nghĩ thuật toán mã hóa"; các biến thể tinh vi hơn cũng chết người y hệt: tự chế công thức MAC (hash(key‖msg) → length extension), tự chế giao thức trao khóa (DH không authentication → MITM), tự cài RSA "theo sách" (không padding → phá được bằng brute force bản rõ). Ranh giới an toàn: bạn được *dùng* và *ghép* các primitive đã chuẩn hóa theo đúng pattern đã chuẩn hóa; mọi sáng tạo ở tầng dưới đó cần một đội cryptographer và vài năm review công khai.

**Bảo mật bằng che giấu (security through obscurity).** Giấu thuật toán, đổi tên endpoint, base64 "cho khó đọc" — tất cả sụp đổ trước một lần decompile. Phản đề trực tiếp của Kerckhoffs: nếu bản thiết kế lộ ra mà hệ thống sụp, hệ thống đã sụp sẵn rồi.

**Dùng đúng primitive, sai chế độ.** AES-ECB mã hóa từng block độc lập nên block giống nhau cho ciphertext giống nhau — bức ảnh "ECB penguin" nổi tiếng vẫn lộ nguyên hình chú chim cánh cụt sau khi mã hóa. Reuse nonce trong AES-GCM còn tệ hơn: lộ luôn khóa xác thực. Primitive đúng + mode sai = không an toàn.

**Hardcode secret trong source/repo.** Key trong code là key đã lộ (git history nhớ mãi mãi). Secret thuộc về secret manager, được rotate định kỳ — hệ quả vận hành của nguyên tắc "an ninh nằm ở key": key phải *thay được* thì nguyên tắc mới có nghĩa.

**Tin dữ liệu do client tự khai về cách verify chính nó.** JWT `alg: none` là ví dụ thuần khiết nhất: để kẻ tấn công chọn thuật toán kiểm tra chữ ký của chính hắn. Mọi tham số an ninh (thuật toán, key ID, cost) phải do server quyết định hoặc whitelist chặt.

**Nhầm encoding với encryption.** Base64/hex là biểu diễn, không phải bảo mật — không có key thì không có bí mật. Nghe hiển nhiên, nhưng "password được mã hóa base64 trong config" vẫn xuất hiện trong audit hằng năm.

## 10. Best Practices

**Nên:**

- Dùng thư viện chuẩn và các default của nó: `crypto/tls` (version/cipher mặc định của Go là lựa chọn tốt), `crypto/rand` cho mọi thứ cần ngẫu nhiên bảo mật, `bcrypt`/`argon2` cho password, `crypto/hmac` + `hmac.Equal` cho MAC, AES-GCM hoặc `golang.org/x/crypto/nacl` cho mã hóa dữ liệu.
- Thiết kế **crypto agility**: version hóa định danh thuật toán trong dữ liệu lưu trữ (bcrypt tự làm: `$2b$12$...`), để 5 năm sau nâng cấp thuật toán không cần migration đau đớn.
- Rotate key định kỳ và có kịch bản thu hồi; log việc *dùng* key, không bao giờ log *giá trị* key.
- Nghĩ về an ninh theo tầng khái niệm khi review thiết kế: bí mật (encryption), toàn vẹn (MAC/signature), danh tính (certificate/signature), tươi mới (nonce/timestamp) — mỗi tầng một câu hỏi "cái gì đang bảo đảm điều này?".
- Nhớ đơn vị đo an ninh là **bit**: mỗi bit gấp đôi công phá mã; 128 bit là chuẩn mặc định hiện nay; collision resistance = một nửa số bit của hash.

**Không nên:**

- Không tự cài primitive hay giao thức — kể cả "chỉ để dùng nội bộ".
- Không so sánh bất kỳ giá trị bí mật nào bằng `==`/`bytes.Equal` — luôn constant-time.
- Không dùng MD5/SHA-1 ở bất kỳ ngữ cảnh nào cần chống collision (chữ ký, certificate, dedup nội dung không tin cậy).
- Không coi "đã mã hóa" là "đã xác thực" — luôn authenticated encryption.
- Không trì hoãn nâng cost factor của password hash "vì đang chạy ổn" — kẻ tấn công nâng cấp GPU đều đặn hơn bạn nâng cấp config.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Diffie-Hellman chống được kẻ nghe lén thụ động nhưng thua man-in-the-middle chủ động — chính xác thì giao thức *thiếu* tính chất gì, và TLS bù bằng cơ chế nào?
2. Vì sao SHA-256 chỉ cho 128-bit collision resistance, và con số 128 đó liên hệ thế nào với birthday paradox?
3. Cùng là hash, vì sao hash cho HMAC phải nhanh còn hash cho password phải chậm? Điều gì xảy ra nếu tráo đổi hai lựa chọn đó?

---

*Chương tiếp theo: [27 — Production Case Studies](/series/math-for-engineers/level-5-production/27-production-case-studies/), nơi mọi khái niệm của 26 chương hội tụ: mười hệ thống tỷ đô được mổ xẻ để chỉ ra chính xác toán học nằm ở dòng nào của kiến trúc.*
