+++
title = "Chương 12 — Hashing: Nén vô hạn vào hữu hạn"
date = "2026-07-20T09:00:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 3 – Algorithm Mathematics
> Yêu cầu trước: Chương 05 (Counting & Combinatorics — Pigeonhole), Chương 11 (Probability)

---

## 1. Problem Statement

Tháng 12 năm 2011, tại hội nghị bảo mật 28C3 ở Berlin, hai nhà nghiên cứu trình diễn một cuộc tấn công khiến cả ngành web giật mình: với **một request POST duy nhất** cỡ vài trăm KB, họ ghim CPU của một server PHP ở mức 100% trong nhiều phút. Không tràn bộ đệm, không SQL injection, không lỗi nào trong code ứng dụng. Vũ khí là toán học: họ tính trước hàng chục nghìn chuỗi **có cùng giá trị hash**, gửi chúng làm tên tham số form. Hash table đựng tham số — cấu trúc "O(1)" mà mọi framework tin tưởng — suy biến thành một danh sách liên kết dài, mỗi thao tác chèn thành O(n), tổng xử lý một request thành O(n²). PHP, Java, Python, Ruby, ASP.NET đều dính. Các bản vá khẩn cấp được tung ra trong vài tuần, và ngôn ngữ sinh sau như Go được thiết kế với hash seed ngẫu nhiên ngay từ đầu vì bài học này.

Câu chuyện phơi bày cả hai mặt của hashing: nó là cấu trúc nền của gần như mọi phần mềm (mỗi request web đi qua hàng chục hash lookup), và các đảm bảo của nó **thuần túy có tính xác suất** — đủ tốt để cả thế giới dựa vào, đủ mong manh để một đối thủ hiểu toán bẻ gãy.

Nhưng hãy lùi lại bài toán gốc, thứ khiến hashing tồn tại:

**Cho một key bất kỳ (chuỗi, struct, URL...), tìm dữ liệu gắn với nó trong thời gian O(1) — không phụ thuộc số lượng key đang lưu.**

Mảng đã cho ta O(1) từ chương đầu đời lập trình: `a[i]` là một phép cộng địa chỉ. Nhưng mảng chỉ nhận index là số nguyên nhỏ. Thế giới thực dùng key là email, UUID, đường dẫn file — những thứ sống trong không gian vô hạn. Hashing là cây cầu: **biến key thành index**. Và ngay tại nhát búa đầu tiên đó, một định lý đếm từ chương 05 đã đứng chờ sẵn để đòi nợ.

## 2. Trực giác

### Từ tủ hồ sơ đến hàm băm

Hãy tưởng tượng một văn phòng lưu 10.000 hồ sơ khách hàng theo tên. Cách ngây thơ: xếp chồng, tìm ai thì lật từng tờ — O(n). Cách của mọi văn phòng thật: **26 ngăn kéo theo chữ cái đầu**. Tìm "Nguyễn Văn A" → mở ngăn N → chỉ lật trong ngăn đó. Ta vừa phát minh ra hash function: `h(tên) = chữ cái đầu`, và hash table 26 slot.

Cách này lộ ngay hai vấn đề, và cả hai đều là chủ đề chính của chương:

1. **Nhiều hồ sơ chung một ngăn** — "collision". Ngăn N ở Việt Nam sẽ nổ tung vì họ Nguyễn chiếm gần 40% dân số, trong khi ngăn W gần rỗng. Hàm "chữ cái đầu" là hash function *tồi*: nó không rải đều.
2. **Ngăn đầy thì làm gì** — cần chiến lược chứa nhiều hồ sơ trong một chỗ (chaining), hoặc tràn sang ngăn bên cạnh (open addressing), và đến lúc nào đó phải sắm tủ to hơn (resize).

Toàn bộ lý thuyết hash table là hai câu hỏi này được trả lời chặt chẽ: *làm sao rải đều?* và *sống chung với đụng độ thế nào?*

### Collision không phải rủi ro — nó là định mệnh

Điểm nhận thức quan trọng nhất chương: đừng hy vọng thiết kế được hash function "không đụng độ". Hash function là ánh xạ từ không gian key **vô hạn** (mọi chuỗi byte) vào không gian index **hữu hạn** (m slot, hay 2⁶⁴ giá trị). Theo nguyên lý chuồng bồ câu (Pigeonhole — chương 05): nhét nhiều hơn m con chim vào m chuồng thì có chuồng chứa ít nhất hai con. Với không gian key vô hạn, **tồn tại vô số key đụng độ nhau — với mọi hash function, kể cả SHA-256**. Đây là hệ quả logic của việc nén vô hạn vào hữu hạn, không phải khiếm khuyết kỹ thuật.

Vậy nên câu hỏi đúng không phải "làm sao tránh collision?" (bất khả thi) mà là:

- Làm collision **hiếm** cỡ nào? → xác suất, birthday paradox (chương 11).
- Khi collision xảy ra, **trả giá** bao nhiêu? → collision resolution, load factor.
- Kẻ xấu có thể **cố tình tạo** collision không? → universal hashing, câu chuyện 28C3.

Chấp nhận collision như chấp nhận trọng lực, rồi thiết kế quanh nó — đó là tư duy đúng, và là lý do chương 11 phải đứng trước chương này: không có xác suất, ta không có ngôn ngữ nào để nói về "hiếm" và "trả giá kỳ vọng".

## 3. First Principles

### Hash function là gì, về mặt toán học

Một hash function là một hàm (chương 02):

> h : U → {0, 1, ..., m−1}

với U là không gian key (thường vô hạn hoặc khổng lồ), m là số slot. Vì |U| > m, h **không thể injective** (đơn ánh) — hai key khác nhau có thể chung ảnh. Ba tính chất định nghĩa một hash function *tốt*:

**Deterministic** — cùng key luôn cho cùng hash, trong cùng một "phiên". Nghe hiển nhiên nhưng là nguồn bug thật: hash một struct chứa con trỏ, hash một float NaN, hash phụ thuộc thứ tự duyệt map — đều phá tính deterministic và làm key "biến mất" khỏi table. (Lưu ý cái bẫy chiều ngược: hash của Go *cố ý* khác nhau giữa các lần chạy process — deterministic trong phiên, ngẫu nhiên giữa các phiên. Ai lưu giá trị hash ra disk rồi mong đọc lại được sẽ vỡ mộng.)

**Uniform** — key "thực tế" rải đều trên m slot, không dồn cục như ngăn kéo chữ N. Về mặt toán, ta mô hình hóa bằng giả định **Simple Uniform Hashing**: mỗi key rơi vào mỗi slot với xác suất 1/m, độc lập với các key khác. Mọi phân tích đẹp đẽ phía dưới đứng trên giả định này — và mọi thảm họa hash table đều bắt đầu từ chỗ giả định này gãy.

**Avalanche** — thay đổi 1 bit của input lật trung bình một nửa số bit của output. Đây là cơ chế tạo ra uniform trong thực tế: key thật không ngẫu nhiên (các URL chung prefix `https://`, các ID liên tiếp nhau 1, 2, 3...), avalanche nghiền nát mọi cấu trúc đó thành nhiễu trắng. Hàm thiếu avalanche — như `h(x) = x mod m` trên ID tuần tự — giữ nguyên cấu trúc của input trong output: ID tuần tự cho slot tuần tự, và một pattern trong dữ liệu (ví dụ ID toàn chẵn với m chẵn) thành pattern trong phân bố slot, giết chết uniform.

### Nếu không có hashing thì sao?

Mọi lookup theo key phải qua so sánh: cây cân bằng O(log n) hoặc scan O(n). Nghe không tệ? log₂(10⁹) ≈ 30 phép so sánh chuỗi, mỗi phép có thể duyệt cả trăm ký tự và nhảy con trỏ khắp bộ nhớ. Hashing thay 30 lần so sánh-và-nhảy bằng *một* lần tính toán số học trên key rồi *một* cú nhảy thẳng đến dữ liệu. Quan trọng hơn, hashing là nền của những thứ vượt xa lookup: rải dữ liệu vào partition không cần điều phối (Kafka), định danh nội dung (Git), khử trùng lặp, chữ ký, checksum. Bỏ hashing khỏi một hệ thống hiện đại giống rút xương sống.

## 4. Mathematical Model

### Load factor và chiều dài chain kỳ vọng — chương 11 trả cổ tức

Đặt **load factor**:

> α = n/m  (n key trong m slot)

α là *biến trạng thái quan trọng nhất* của hash table. Với chaining (mỗi slot một danh sách), chiều dài chain kỳ vọng tính bằng đúng kỹ thuật indicator + linearity của chương 11: xét slot s và key i, đặt X_i = 1 nếu key i rơi vào s. Theo giả định uniform, E[X_i] = 1/m. Chiều dài chain tại s là ΣX_i, nên:

> E[chain] = Σᵢ E[Xᵢ] = n · (1/m) = **α**

Không cần độc lập giữa các key, không cần biết phân phối chi tiết — linearity cân tất. Hệ quả: một lần lookup *miss* duyệt kỳ vọng α phần tử; lookup *hit* duyệt kỳ vọng ~1 + α/2. Chừng nào ta **giữ α bị chặn bởi hằng số** (Go giữ dưới 6.5/8 ≈ 0.81; Java dưới 0.75), lookup là **O(1) kỳ vọng** — đó chính xác là nội dung của lời cam kết "hash table O(1)", với cả ba chữ in nghiêng: *kỳ vọng*, *dưới giả định uniform*, *khi α bị chặn*.

```
α nhỏ (0.3):  [k]  [ ]  [k]  [ ]  [k,k]  [ ]  [k]  ...   chain ngắn, tốn RAM
α lớn (5.0):  [kkkkk] [kkkkkkk] [kkkk] [kkkkkk] ...      tiết kiệm RAM, lookup chậm dần
```

### Resize — vì sao và giá bao nhiêu

n tăng mà m đứng yên thì α tăng, O(α) trượt dần khỏi "hằng số". Giải pháp: khi α chạm ngưỡng, cấp phát bảng mới lớn hơn **theo tỷ lệ** (thường ×2) và rehash toàn bộ. Một lần resize tốn O(n) — nghe đáng sợ, nhưng đây là bản sao y nguyên bài toán slice append ở chương 10: chuỗi resize có tổng chi phí 1 + 2 + 4 + ... + n < 2n, nên **insert là O(1) amortized**. Và cũng như chương 10 đã cảnh báo: phải tăng theo tỷ lệ; tăng m thêm hằng số mỗi lần sẽ trả O(n²) tổng. Amortized O(1) không có nghĩa là "mọi insert đều nhanh" — thao tác thứ 2ᵏ vẫn đắt đột biến, một sự thật mà Redis phải thiết kế riêng để né (mục 7).

### Birthday paradox — hash space bao nhiêu bit là đủ?

Câu hỏi khác với load factor: không phải "n key trong m *slot*" mà "n hash value trong không gian 2ᵇ — bao giờ hai key khác nhau *trùng hash hoàn toàn*?". Chương 11 đã cho công thức: 50% collision tại **n ≈ 1.18·√m**. Áp vào các kích thước hash phổ biến:

| Hash | Không gian m | ~50% có collision tại | Phán quyết |
|---|---|---|---|
| 32-bit | 2³² ≈ 4.3 tỷ | **~77.000 phần tử** | KHÔNG dùng làm định danh |
| 64-bit | 2⁶⁴ | ~5 tỷ phần tử | định danh tạm trong RAM: tạm ổn, vẫn phải xử lý trùng |
| 128-bit | 2¹²⁸ | ~2×10¹⁹ | an toàn thực tế cho định danh |
| SHA-256 | 2²⁵⁶ | ~4×10³⁸ | an toàn cả trước đối thủ chủ động |

Con số 77.000 đáng đóng khung: một hệ thống dùng CRC32 hay hash 32-bit làm "ID duy nhất" cho object sẽ gặp trùng lặp gần như chắc chắn ngay khi có vài trăm nghìn object — không phải xui, mà là căn bậc hai làm việc của nó. Còn bên trong hash table thì ngược lại: collision là chuyện thường ngày đã có chain xử lý, ta chỉ cần hash **so sánh được rồi kiểm tra lại key thật** — đó là lý do mọi hash table đều lưu cả key, không chỉ hash.

### Universal hashing — vá lỗ hổng 28C3 bằng định nghĩa

Giả định uniform gãy tan khi input do đối thủ chọn: với hash function *cố định và công khai*, kẻ tấn công tính trước n key cùng rơi vào một slot, biến α cục bộ thành n và lookup thành O(n). Nhận xét then chốt: vấn đề không nằm ở một hàm cụ thể yếu, mà ở chỗ **hàm là cố định** — mọi hàm cố định đều có tập input xấu của nó (pigeonhole, lần nữa).

Lối thoát: đừng dùng một hàm — dùng một **họ hàm** H, và bốc ngẫu nhiên một hàm khi khởi tạo. Họ H gọi là **universal** nếu:

> ∀ x ≠ y:  P_{h∈H}(h(x) = h(y)) ≤ 1/m

— tức là với *mọi cặp key kẻ tấn công chọn trước*, xác suất đụng độ (trên lựa chọn hàm ngẫu nhiên của ta) không tệ hơn ném xúc xắc đều. Toàn bộ phân tích E[chain] = α sống lại nguyên vẹn, nhưng bây giờ randomness nằm ở **xúc xắc của ta** thay vì ở lòng tốt của input — đúng nước cờ randomized quicksort của chương 11: chuyển rủi ro từ chỗ đối thủ kiểm soát sang chỗ không ai kiểm soát. Đây chính là điều Go làm: mỗi map nhận một seed ngẫu nhiên lúc tạo, hai lần chạy chương trình dùng hai hàm hash khác nhau, kẻ tấn công không thể tính trước tập key đụng độ vì không biết mình đang đấu với hàm nào.

### Cryptographic vs non-cryptographic — hai nghề khác nhau

Hai họ hash function phục vụ hai mô hình đe dọa khác nhau, và nhầm lẫn giữa chúng gây lỗi cả hai chiều:

| | Non-crypto (xxHash, MurmurHash, FNV) | Crypto (SHA-256, BLAKE3) |
|---|---|---|
| Cam kết | rải đều, avalanche tốt, cực nhanh | thêm: **không thể tìm** collision/preimage dù cố tình |
| Tốc độ | ~chục GB/s (xxHash3) | chậm hơn cỡ 10× (SHA-256 có tăng tốc phần cứng vẫn thua xa) |
| Dùng cho | hash table, partitioning, checksum chống lỗi *ngẫu nhiên* | chữ ký, lưu password (qua KDF), content address, chống đối thủ *chủ động* |

Ranh giới quyết định là câu hỏi: **"có ai được lợi khi cố tình tạo collision không?"**. Hash table nội bộ sau universal hashing: không → xxHash. Định danh object mà client tự tính và hệ thống tin (Git, deduplication giữa các user): có → phải crypto. Lịch sử SHA-1 minh họa cái giá của ranh giới này: năm 2017, Google công bố hai file PDF khác nhau cùng SHA-1 (SHAttered) với chi phí tính toán lớn nhưng khả thi — "không thể tìm collision" của SHA-1 hết hạn, và Git đã phải bắt đầu hành trình dời sang SHA-256 dù hệ thống vẫn chạy tốt với collision *ngẫu nhiên* (xác suất 2⁻⁸⁰ vẫn là thiên văn).

## 5. Thuật toán

### Chaining vs Open Addressing

Hai trường phái xử lý đụng độ, khác nhau ở chỗ *phần tử tràn đi đâu*:

```
Chaining:  slot trỏ tới danh sách            Open addressing: tràn sang slot kế
[0] → (k3,v) → (k9,v)                        [0] k3   [1] k9   [2] ---
[1] → (k1,v)                                 [3] k1   [4] k7   [5] ---
[2] → nil                                    probe: h(k), h(k)+1, h(k)+2, ...
```

**Chaining**: mỗi slot là một linked list (hoặc mảng nhỏ). Đơn giản, chịu được α > 1, xóa dễ. Giá: mỗi node một lần cấp phát, và duyệt chain là nhảy con trỏ — mỗi bước một nguy cơ cache miss.

**Open addressing**: tất cả nằm trong một mảng phẳng; đụng độ thì dò (probe) sang slot khác theo quy luật — linear probing dò tuần tự i, i+1, i+2... Ưu thế quyết định trên phần cứng hiện đại: các slot kế nhau nằm **cùng cache line**, dò 5 slot liên tiếp có thể rẻ hơn đuổi theo 2 con trỏ của chaining. Giá: bắt buộc α < 1 và hiệu năng suy sập khi α → 1 (số probe kỳ vọng của lookup miss với linear probing cỡ (1 + 1/(1−α)²)/2 — tại α = 0.9 là ~50 probe); xóa phải dùng "tombstone" vì đục lỗ giữa dãy probe sẽ làm lookup dừng sớm; và các cụm đầy có xu hướng phình to (primary clustering — cụm càng dài càng dễ hứng thêm key, một vòng lặp dương tính).

Minh họa linear probing tối giản:

```go
// Bảng open addressing với linear probing — minh họa khái niệm.
// (Thực tế hãy dùng map của Go; đây là để thấy cơ chế.)
type entry struct {
    key     string
    val     int
    used    bool // slot đã từng chứa dữ liệu
    deleted bool // tombstone — đã xóa nhưng không phá dãy probe
}

func (t *table) lookup(key string) (int, bool) {
    i := t.hash(key) % uint64(len(t.slots))
    for t.slots[i].used { // gặp slot chưa-từng-dùng nghĩa là chắc chắn miss
        s := &t.slots[i]
        if !s.deleted && s.key == key { // hash trùng chưa đủ — phải so key thật
            return s.val, true
        }
        i = (i + 1) % uint64(len(t.slots)) // dò tuyến tính: slot kế tiếp
    }
    return 0, false
}
```

### Rolling hash — Rabin–Karp và cửa sổ trượt O(1)

Bài toán: tìm pattern độ dài k trong văn bản độ dài n. So sánh trực tiếp tại mỗi vị trí là O(nk). Ý tưởng Rabin–Karp: so **hash của cửa sổ** với hash của pattern trước, chỉ so chuỗi thật khi hash trùng. Nhưng nếu tính lại hash mỗi cửa sổ mất O(k) thì công cốc. Phép màu nằm ở việc chọn hash có cấu trúc đại số: coi cửa sổ là **một con số viết trong hệ cơ số b**, lấy modulo M:

> H(s[i..i+k−1]) = (s[i]·b^{k−1} + s[i+1]·b^{k−2} + ... + s[i+k−1]) mod M

Trượt cửa sổ sang phải một ký tự = bỏ chữ số đầu, dịch trái, thêm chữ số cuối:

> H_mới = ((H_cũ − s[i]·b^{k−1}) · b + s[i+k]) mod M

— đúng ba phép nhân cộng, **O(1) mỗi bước trượt**, tổng O(n + k) kỳ vọng.

```go
// Tìm mọi vị trí xuất hiện của pat trong txt bằng Rabin–Karp.
func RabinKarp(txt, pat string) []int {
    const b, M = 256, 1_000_000_007 // cơ số và modulo nguyên tố lớn
    n, k := len(txt), len(pat)
    if k > n {
        return nil
    }
    var pow, hp, ht int64 = 1, 0, 0
    for i := 0; i < k; i++ {
        hp = (hp*b + int64(pat[i])) % M   // hash của pattern
        ht = (ht*b + int64(txt[i])) % M   // hash cửa sổ đầu tiên
        if i > 0 {
            pow = pow * b % M // pow = b^(k-1) mod M, dùng khi bỏ ký tự đầu
        }
    }
    var out []int
    for i := 0; ; i++ {
        // hash trùng chỉ là NGHI VẤN — phải so chuỗi thật để loại false positive
        if hp == ht && txt[i:i+k] == pat {
            out = append(out, i)
        }
        if i+k == n {
            return out
        }
        ht = ((ht-int64(txt[i])*pow%M+M)*b + int64(txt[i+k])) % M // trượt O(1)
    }
}
```

Rolling hash là ví dụ hiếm hoi hash function được chọn vì **cấu trúc đại số** thay vì vì avalanche — ta *muốn* hash của các cửa sổ kề nhau liên hệ với nhau. Cùng kỹ thuật này chạy trong rsync (đồng bộ file theo khối trượt) và các hệ chunking để deduplication (content-defined chunking): ranh giới chunk được đặt tại nơi rolling hash thỏa điều kiện, nhờ đó chèn 1 byte vào đầu file chỉ làm thay đổi ranh giới cục bộ thay vì dịch chuyển mọi chunk phía sau.

## 6. Trade-off

**Chaining vs open addressing** là trade-off giữa *đơn giản, linh hoạt* và *cache locality*. Trên CPU hiện đại, một cache miss (~100ns) đắt bằng hàng trăm phép so sánh, nên xu hướng chung của các runtime là về phe mảng phẳng: map của Go là dạng lai (bucket 8 slot phẳng + overflow), Swiss Tables của Google (được Go 1.24 tiếp nhận) là open addressing thuần với probe theo nhóm 16 slot kiểm tra bằng lệnh SIMD. Bài học chương 10 lặp lại: trong cùng bậc O(1), hằng số cache quyết định kẻ thắng.

**Load factor cao vs thấp** là RAM đổi lấy tốc độ, với một đường cong phi tuyến: từ α = 0.5 lên 0.7 gần như miễn phí, từ 0.9 lên 0.95 là thảm họa cho open addressing (1/(1−α) nổ tung). Ngưỡng resize của các runtime (0.75–0.87) là điểm cân bằng đã được đo đạc kỹ trên workload thực.

**Tốc độ hash vs chất lượng hash.** Hash nhanh nhưng rải kém → chain dài → thua ở lookup; hash "xịn" quá mức (dùng SHA-256 cho hash table nội bộ) → thua gấp 10 lần ở khâu tính hash mà không mua thêm được gì, vì mô hình đe dọa không cần crypto. Chi phí hash tỷ lệ với **độ dài key** — điều hay bị quên: hash table với key là URL 2KB thì "O(1)" của bạn mang hằng số 2000; nếu key dài và lookup nhiều, cân nhắc hash trước một lần rồi dùng hash làm key (interning), hoặc so sánh cây tiền tố (chương 24, Trie).

**Hash table vs cây cân bằng.** Hash cho O(1) kỳ vọng nhưng **mất thứ tự**: không range query, không "key nhỏ nhất lớn hơn x", không duyệt có thứ tự. B-Tree của database (chương 07) chấp nhận O(log n) để giữ thứ tự — đó là lý do PostgreSQL index mặc định là B-Tree chứ không phải hash index, dù hash index tồn tại. Chọn cấu trúc theo *tập câu hỏi* bạn cần trả lời, không theo con số Big-O đơn lẻ.

**Khi nào KHÔNG nên dùng hash table:** cần thứ tự/range (dùng cây); n nhỏ hơn ~50 (mảng + linear scan thắng nhờ cache, như chương 10 đã đo); key có cấu trúc khai thác được (số nguyên liên tiếp → mảng thẳng; tiền tố chung → trie); cần worst-case cứng từng thao tác (real-time — một lần resize hoặc chain dài phá deadline; dùng cây hoặc hash table incremental).

## 7. Production Applications

**Go map internals.** Map của Go (trước 1.24) là hash table lai: mảng các **bucket, mỗi bucket 8 slot phẳng**. Hash 64-bit của key được chẻ đôi việc: các bit thấp chọn bucket, còn 8 bit cao lưu thành **tophash** — một mảng 8 byte đầu bucket. Lookup quét 8 byte tophash (nằm gọn trong một cache line) trước, chỉ khi tophash khớp mới đụng đến key thật — lọc rẻ trước, so đắt sau, cùng triết lý với Rabin–Karp. Bucket đầy thì móc thêm overflow bucket (một nhúm chaining), và khi α vượt 6.5/8 map **grow incrementally**: bảng mới gấp đôi được cấp phát nhưng dữ liệu chỉ dời dần, mỗi thao tác write "vác hộ" một hai bucket cũ — trải chi phí O(n) của resize thành từng mẩu O(1), tránh cú khựng đột biến. Cộng với hash seed ngẫu nhiên per-map (universal hashing thực chiến) và thứ tự duyệt cố ý ngẫu nhiên — gần như mọi định lý của chương này đều đang chạy trong một dòng `m[k] = v`.

**Redis incremental rehashing.** Redis là single-threaded event loop: một lần rehash 100 triệu key kiểu stop-the-world nghĩa là *đứng phục vụ* nhiều giây — chết SLA. Giải pháp: dict của Redis giữ **hai bảng song song** khi rehash (ht[0] cũ, ht[1] mới); mỗi lệnh chạm dict dời một bucket, cron dời thêm từng lát 1ms; lookup trong giai đoạn chuyển tiếp tra cả hai bảng. Amortized cost của chương 10 được thi công thành kiến trúc: tổng chi phí không đổi, nhưng **phân phối chi phí theo thời gian** được san phẳng — nhắc ta rằng amortized O(1) là phát biểu về *tổng*, còn latency spike là chuyện về *phân phối*, và production quan tâm cả hai.

**Kafka partitioning.** Producer mặc định gửi message vào partition `hash(key) % n` (murmur2). Uniform hash ⇒ tải rải đều *kỳ vọng* giữa n partition mà không cần điều phối viên — chương 11 gọi tên hiện tượng này rồi. Nhưng phép modulo giấu một quả mìn: **đổi n là đổi gần như toàn bộ ánh xạ** — tăng 10 partition lên 12, `hash % 10` và `hash % 12` khác nhau ở ~5/6 số key, phá vỡ cam kết "cùng key vào cùng partition" mà consumer đang dựa vào để giữ thứ tự. Bài toán "resize ánh xạ hash mà xáo trộn tối thiểu" chính là cửa ngõ dẫn tới **consistent hashing** — chương 25.

**Database hash join.** Bài Two Sum phóng đại lên hàng triệu dòng: để join hai bảng theo `R.id = S.id`, PostgreSQL build hash table trên bảng nhỏ O(|R|), rồi probe với từng dòng bảng lớn O(|S|) — tổng O(|R| + |S|), so với nested loop O(|R|·|S|). Planner chọn hash join khi thiếu index và bảng nhỏ vừa RAM (`work_mem`) — một lần nữa complexity analysis chạy tự động trong `EXPLAIN`.

**Content-addressable storage — Git.** Git đảo ngược quan hệ tên–nội dung: **địa chỉ của object là hash của nội dung** (SHA-1, đang chuyển dần SHA-256). Hệ quả rơi ra như định lý: khử trùng lặp tự động (hai file giống nhau là một object); kiểm tra toàn vẹn miễn phí (đọc ra hash lại mà lệch là hỏng); và Merkle structure — commit hash phụ thuộc tree hash phụ thuộc blob hash, nên **một hash 40 ký tự niêm phong toàn bộ lịch sử** — đổi một bit của một file mười năm trước là đổi mọi commit hash phía sau. Tính chất này đứng hoàn toàn trên "không thể tìm collision" của crypto hash — và là lý do vụ SHAttered buộc Git phải lên kế hoạch di cư.

**HTTP ETag** — hash của response làm "phiên bản": client gửi `If-None-Match: <etag>`, server so hash, khớp thì trả `304 Not Modified` rỗng thay vì body. So sánh nội dung tùy ý bị nén thành so sánh chuỗi ngắn — cùng nguyên lý với việc so tophash trước khi so key, xuất hiện ở tầng HTTP.

## 8. Interview

Hash table là cấu trúc được "mặc định phải thành thạo" trong phỏng vấn — nó xuất hiện không phải như chủ đề riêng mà như **nước đi giảm bậc**: gặp O(n²) vì tra cứu lồng trong vòng lặp, đổi tra cứu thành hash lookup, còn O(n).

**Two Sum (LeetCode 1).** Bài mở màn kinh điển: tìm cặp có tổng bằng target. Brute force O(n²); nhận ra "với mỗi x, mình cần biết `target − x` **đã từng xuất hiện chưa**" — câu hỏi membership, chính là nghề của hash set → một lượt O(n). Nước đi tổng quát đáng nhớ: *"mình đang tìm lại thứ đã nhìn thấy"* ⇒ hash.

**Group Anagrams (LeetCode 49).** Gom các từ là đảo chữ của nhau. Điểm ăn tiền không phải cấu trúc dữ liệu mà là **thiết kế hash key**: cần một hàm mà mọi anagram cho cùng key, mọi không-anagram cho khác key — tức là một **canonical form** (dạng chuẩn tắc). Hai ứng viên: chuỗi đã sort (`"eat" → "aet"`, tốn k log k mỗi từ) hoặc vector đếm 26 ký tự (`[1,0,...,1,...,1,0]`, tốn k). Bài này dạy kỹ năng giá trị hơn chính lời giải: nhiều bài "khó" trở thành tầm thường ngay khi chọn đúng biểu diễn key — "hash cái gì" quan trọng hơn "hash thế nào".

**Longest Consecutive Sequence (LeetCode 128).** Tìm dải số liên tiếp dài nhất trong mảng chưa sort, yêu cầu O(n). Sort là O(n log n) — đề cấm khéo. Lời giải: đổ hết vào hash set, rồi với mỗi x mà `x−1` **không** có trong set (x là đầu dải), đếm dài dải bằng cách hỏi x+1, x+2... Mỗi phần tử bị hỏi O(1) lần amortized → O(n). Bài này kiểm tra đúng hiểu biết cốt lõi: hash set biến câu hỏi "tồn tại không?" thành O(1), cho phép thuật toán *nhảy* theo cấu trúc bài toán thay vì theo thứ tự sắp xếp.

**LRU Cache (LeetCode 146).** Ghép hash map (lookup O(1)) với doubly linked list (đổi thứ tự recency O(1)) — hai cấu trúc bù khuyết cho nhau: map tìm nhanh nhưng vô trật tự, list có trật tự nhưng tìm chậm. Phân tích đầy đủ ở chương 24; ở đây nó minh họa nguyên tắc **composite structure**: khi không cấu trúc nào trả lời đủ tập câu hỏi, ghép hai cái và giữ chúng đồng bộ.

**Lỗi tư duy thường gặp:**

- **Coi hash lookup là O(1) tuyệt đối, vô điều kiện.** Chính xác phải là: O(1) *kỳ vọng*, *cộng chi phí hash key* (O(k) với key dài k!), *amortized* qua resize, *dưới giả định input không adversarial hoặc có universal hashing*. Nói được bốn dấu sao này trong phỏng vấn là tín hiệu senior thật sự. Với key là chuỗi dài, thuật toán "O(n) lần lookup" thực chất là O(n·k) — đôi khi chậm hơn giải pháp "O(n log n)" so sánh số nguyên.
- Quên rằng hash trùng ≠ key trùng — thiết kế dedup chỉ lưu hash 32/64-bit rồi coi trùng hash là trùng nội dung (birthday paradox đòi nợ ở mục 4).
- Dùng thứ gì đó mutable làm map key, hoặc trông đợi thứ tự duyệt map ổn định (Go cố tình random hóa để bạn khỏi kịp mắc lỗi này).
- Đề bài yêu cầu range/thứ tự mà vẫn cố nhét hash table — nhận diện tập câu hỏi trước khi chọn cấu trúc.

**Cách phân tích thay vì học thuộc:** thấy vòng lặp lồng vòng lặp mà vòng trong chỉ để *tìm* thứ gì đó — hỏi "điều kiện tìm kiếm có biến thành phép tra cứu bằng key được không?"; nếu được, thiết kế key (canonical form là kỹ năng trung tâm), rồi trả lời tiếp "chi phí tính key là bao nhiêu?" và "có cần thứ tự không?". Ba câu hỏi đó giải quyết đại đa số bài dùng hash trong phỏng vấn — và cũng là đúng ba câu hỏi khi chọn partitioning key cho Kafka hay shard key cho database.

## 9. Anti-pattern

**Tự chế hash function.** `hash = tổng mã ASCII` — mọi anagram đụng nhau, avalanche bằng không. Hash function tốt là sản phẩm của nhiều năm phân tích thống kê và tấn công thử (xem SMHasher test suite); dùng maphash/xxHash cho non-crypto, SHA-2/BLAKE3 cho crypto, và coi việc tự viết là red flag trong code review.

**Dùng hash % n làm sharding "tạm thời".** Nó chạy tốt cho đến ngày thêm node — rồi 90% key đổi chỗ, cache lạnh toàn cụm, database gánh cú đấm cache miss đồng loạt. Nếu ánh xạ hash-sang-node có ngày phải đổi n, dùng consistent hashing **từ ngày đầu** (chương 25); "tạm thời" trong hạ tầng nghĩa là vĩnh viễn cho đến khi nó gây sự cố.

**Lưu giá trị hash không-crypto ra ngoài process.** Hash seed của Go đổi mỗi lần chạy; xxHash đổi kết quả theo seed; thậm chí nâng version thư viện cũng có thể đổi thuật toán. Persist hash ra disk/DB rồi so với hash tính lại sau này là bug hẹn giờ. Ranh giới đúng: hash non-crypto sống trong RAM của một process; cái gì cần bền vững qua thời gian và máy móc thì dùng crypto hash có version rõ ràng.

**So sánh bằng hash mà không so lại giá trị thật** — trong hash table (bug tìm nhầm value), trong dedup (mất dữ liệu thật khi 2 file khác nhau trùng hash 64-bit), trong cache key (trả nhầm response của người khác — sự cố rò dữ liệu có thật ở nhiều CDN). Hash là bộ lọc nghi vấn, không phải bằng chứng; chỉ crypto hash trên toàn nội dung, trong mô hình đe dọa đã cân nhắc, mới được phép làm bằng chứng.

**Băm mật khẩu bằng hash nhanh.** SHA-256 *quá nhanh* cho việc này: GPU thử hàng tỷ mật khẩu/giây. Lưu password cần hàm **cố tình chậm và ngốn RAM** (bcrypt, argon2) — trường hợp hiếm hoi trong ngành nơi "chậm" là tính năng được đặt hàng. Nhầm mô hình đe dọa nghiêm trọng nhất chính là ở đây.

**Để hash table lớn mãi không co.** Map của Go không bao giờ thu nhỏ: nạp 10 triệu entry rồi xóa còn 100, các bucket vẫn chiếm RAM (và GC vẫn quét). Pattern xử lý: tạo map mới copy phần còn sống, hoặc dùng cấu trúc khác cho workload phình-xẹp mạnh.

## 10. Best Practices

**Nên:**

- Phát biểu đầy đủ về hiệu năng: "O(1) kỳ vọng amortized, cộng O(len(key)) cho hash, với giả định seed ngẫu nhiên" — và thiết kế theo phát biểu đầy đủ, không theo bản rút gọn.
- Chọn hash theo mô hình đe dọa: đối thủ chủ động hoặc dữ liệu bền vững → crypto (SHA-256/BLAKE3); nội bộ, trong RAM → xxHash/maphash với seed ngẫu nhiên. Trả lời câu "ai được lợi khi tạo collision?" trước khi chọn.
- Định cỡ hash space bằng birthday bound: cần định danh cho N phần tử với xác suất trùng ≤ p thì cần b ≥ log₂(N²/2p) bit — tính ra con số, đừng đoán. Và luôn có kế hoạch cho ngày trùng xảy ra (so nội dung thật, log cảnh báo), kể cả khi "gần như không thể".
- Pre-size khi biết trước kích thước: `make(map[K]V, n)` né toàn bộ chuỗi resize trung gian — tiết kiệm thật, đo được, miễn phí.
- Với hệ nhạy latency, kiểm tra hành vi resize của cấu trúc bạn dùng (stop-the-world hay incremental) — amortized tốt không cứu được p99.

**Không nên:**

- Không dùng hash table khi cần range query, thứ tự, hoặc n bé — cây, sorted slice, mảng thẳng đều có đất riêng.
- Không dùng hash 32-bit làm định danh cho quá vài nghìn object; không coi trùng hash là trùng nội dung nếu chưa so nội dung.
- Không tự viết hash function, không lưu hash non-crypto ra ngoài process, không băm password bằng hash đa dụng.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Vì sao collision là hệ quả logic tất yếu của mọi hash function (kể cả SHA-256), và điều đó thay đổi câu hỏi thiết kế từ gì sang gì?
2. Dẫn lại trong ba dòng: vì sao E[chain length] = α, và giả định nào phải đúng để phép tính đó có nghĩa? Universal hashing cứu giả định đó bằng cách nào khi input do đối thủ chọn?
3. Hệ thống của bạn dùng `hash(user_id) % 16` để chia shard và sắp phải tăng lên 20 shard — chuyện gì sẽ xảy ra với cache và database trong giờ đầu tiên, và vì sao?

---

*Chương tiếp theo: [13 — Divide and Conquer](/series/math-for-engineers/level-3-algorithm-mathematics/13-divide-and-conquer/) — rời thế giới xác suất để quay về chiến lược tất định mạnh nhất của thiết kế thuật toán: chia bài toán làm đôi, và lý do "chia đôi" chứ không phải "chia ba" xuất hiện ở khắp nơi.*
