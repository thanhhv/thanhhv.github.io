+++
title = "Chương 23 — Probabilistic Data Structures: Đổi độ chính xác lấy bộ nhớ"
date = "2026-07-20T10:50:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 5 – Production
> Yêu cầu trước: Chương 10 (Complexity), Chương 11 (Probability), Chương 12 (Hashing)

---

## 1. Problem Statement

Bạn đang xây dựng một web crawler quy mô lớn. Trước khi tải một URL, crawler phải trả lời câu hỏi tưởng như tầm thường: **"URL này đã crawl chưa?"**

Cách hiển nhiên: một hash set chứa mọi URL đã thấy. Làm phép tính với 10 tỷ URL, mỗi URL trung bình 80 byte, cộng overhead của hash table (con trỏ, bucket, hệ số load factor):

> 10⁹ × 10 × (80 + ~40) byte ≈ **hơn 1TB RAM** — kể cả khi chỉ lưu hash 64-bit của URL thay vì chuỗi gốc, vẫn cần ~800GB khi tính đủ overhead.

Không máy nào chứa nổi. Sharding ra 100 máy thì được — nhưng giờ mỗi lần kiểm tra một URL là một network round-trip, và bạn kiểm tra hàng triệu lần mỗi giây.

Bây giờ đổi góc nhìn. Điều gì xảy ra nếu câu trả lời *thỉnh thoảng* sai? Nếu cấu trúc nói "đã crawl rồi" trong khi thực ra chưa, ta bỏ qua một URL — mất một trang trong 10 tỷ trang. Nếu tỷ lệ sai đó là 1%, và ta **chứng minh được bằng toán** rằng nó không bao giờ vượt 1%, thì cái giá đó hoàn toàn chấp nhận được. Đổi lại: bộ nhớ giảm từ 800GB xuống **~12GB** — chưa tới 10 bit mỗi URL.

Đây là triết lý chung của cả chương:

**Chấp nhận sai số CÓ KIỂM SOÁT TOÁN HỌC để đổi lấy mức giảm bộ nhớ hàng trăm đến hàng nghìn lần.**

Chữ quan trọng nhất là "có kiểm soát". Đây không phải heuristic "chắc là đúng" — mỗi cấu trúc trong chương này đi kèm một công thức cho phép bạn *chọn trước* sai số, và toán học đảm bảo sai số thực không vượt quá. Không có xác suất (chương 11) và hashing (chương 12), cả họ cấu trúc này không tồn tại.

## 2. Trực giác

### Sai số nào bán được, sai số nào không?

Không phải câu hỏi nào cũng cần câu trả lời chính xác. Hãy phân loại:

| Câu hỏi | Cần chính xác tuyệt đối? | Vì sao |
|---|---|---|
| "Số dư tài khoản của user?" | Có | Tiền bạc — sai 1 đồng cũng là bug |
| "URL này crawl chưa?" | Không | Sai 1% → crawl thừa/thiếu 1% — vô hại |
| "Hôm nay có bao nhiêu unique visitor?" | Không | 998 triệu hay 1.003 tỷ — quyết định kinh doanh y hệt nhau |
| "Key này có trong SSTable trên disk không?" | Không, nếu sai theo một chiều | Sai → đọc disk thừa một lần, vẫn đúng kết quả cuối |

Hàng thứ tư hé lộ một ý tinh tế: có những sai số **một phía** (one-sided error). Nếu cấu trúc chỉ sai theo chiều "báo có trong khi không có" (false positive) mà *không bao giờ* sai chiều ngược lại (false negative), ta có thể dùng nó làm **bộ lọc đứng trước nguồn sự thật đắt đỏ**: câu trả lời "không có" được tin ngay lập tức, câu trả lời "có" mới cần kiểm chứng. Đó chính là kiến trúc của Bloom filter trước SSTable, và là lý do chữ "filter" nằm trong tên.

### Vì sao ngẫu nhiên lại tiết kiệm được bộ nhớ?

Trực giác nền tảng: một câu trả lời chính xác buộc phải *phân biệt được mọi tập input có thể* — về lý thuyết thông tin (chương 20), điều đó đòi hỏi số bit tỷ lệ với lượng thông tin của tập. Nhưng một câu trả lời **xấp xỉ** chỉ cần phân biệt các tập "khác nhau đủ nhiều". Số trạng thái cần phân biệt sụt giảm khủng khiếp, kéo theo số bit cần lưu.

Ví dụ cực đoan nhất chương này: để đếm *chính xác* 1 tỷ phần tử distinct, bạn buộc phải nhớ đủ thông tin phân biệt "đã thấy x" với "chưa thấy x" cho mọi x — hàng GB. Để đếm *xấp xỉ sai số dưới 1%*, HyperLogLog cần **12KB**. Không phải 12GB. Mười hai kilobyte — nhỏ hơn file ảnh favicon.

### Nếu không có các cấu trúc này thì sao?

Mọi bài toán membership/counting/frequency trên dữ liệu lớn đều quy về: thuê thêm RAM (đắt gấp trăm lần), chấp nhận đọc disk (chậm gấp vạn lần), hoặc shard ra nhiều máy (phức tạp gấp mười lần). Probabilistic data structure là lựa chọn thứ tư: trả bằng một loại tiền tệ khác — độ chính xác — vốn đang thừa trong rất nhiều bài toán.

**Khi nào KHÔNG nên dùng?** Khi sai số không bán được: tiền, quyền truy cập (authorization), dedup thanh toán, bất cứ chỗ nào một false positive gây hậu quả không thể sửa. Và khi n nhỏ: một triệu phần tử trong hash set chỉ tốn vài chục MB — đừng đổi sự chính xác lấy thứ bạn không thiếu.

## 3. First Principles

Cả bốn cấu trúc trong chương xây trên đúng hai viên gạch:

**Viên gạch 1 — Hash function như nguồn ngẫu nhiên đồng nhất (chương 12).** Một hash function tốt biến input bất kỳ thành một số "như thể" được rút ngẫu nhiên đều từ [0, 2⁶⁴). Điều này cho phép ta áp toàn bộ máy móc xác suất của chương 11 lên dữ liệu *thật* — dữ liệu không hề ngẫu nhiên, nhưng ảnh của nó qua hash thì có.

**Viên gạch 2 — Quan sát một "dấu vết" thay vì lưu toàn bộ.** Thay vì nhớ từng phần tử, ta chỉ nhớ một dấu vết nén cực mạnh mà phần tử để lại: vài bit bật lên (Bloom), số lượng số 0 dẫn đầu của hash (HyperLogLog), vài counter tăng lên (Count-Min). Dấu vết mất thông tin — nhiều tập input khác nhau để lại cùng dấu vết — và *chính đó* là nguồn gốc của sai số. Nhưng vì hash là ngẫu nhiên đồng nhất, ta tính được **xác suất** hai tập khác nhau va vào cùng dấu vết, tức là định lượng được sai số.

Chuỗi suy luận chung, đáng ghi nhớ vì nó lặp lại bốn lần trong chương:

```
dữ liệu thật ──hash──► ngẫu nhiên đều ──ghi dấu vết nén──► mất thông tin
                                                              │
                            sai số  ◄──tính được bằng xác suất┘
```

Sai số ở đây không phải "hy vọng là nhỏ" — nó là một biến ngẫu nhiên có kỳ vọng và phương sai tính được, nghĩa là ta điều khiển được nó bằng cách chỉnh tham số (m, k, số register...). Đó là ranh giới giữa kỹ thuật và cầu may.

## 4. Mathematical Model

### 4.1 Bloom Filter

**Cấu trúc:** một mảng m bit (khởi tạo toàn 0) và k hash function độc lập h₁...h_k.

- **Add(x):** bật k bit tại vị trí h₁(x), ..., h_k(x).
- **Contains(x):** trả về true nếu **cả k** bit đó đều đang bật.

```
        h₁(x)=2      h₂(x)=5           h₃(x)=11
          │            │                  │
          ▼            ▼                  ▼
bit:  0  0  1  0  0  1  0  0  1  0  0  1  0  0
idx:  0  1  2  3  4  5  6  7  8  9 10 11 12 13
                          ▲
                 bit 8 do phần tử khác bật
```

**Vì sao không bao giờ có false negative — chứng minh.** Nếu x đã được Add, cả k bit của x đã bật. Bit trong Bloom filter chỉ có một chiều chuyển trạng thái 0 → 1, không bao giờ bị tắt (không có thao tác delete). Vậy tại thời điểm query, cả k bit vẫn bật, Contains(x) chắc chắn trả về true. Đây là một **invariant** kiểu chương 03: "mọi bit của mọi phần tử đã add đều bật" được bảo toàn qua mọi thao tác. ∎

**Vì sao có false positive.** Contains(y) với y chưa từng add có thể true nếu cả k bit của y *tình cờ* đã bị các phần tử khác bật. Câu hỏi định lượng: xác suất đó bao nhiêu?

**Dẫn xuất từng bước — chỉ dùng xác suất cơ bản của chương 11:**

Bước 1. Một hash ghi vào một vị trí ngẫu nhiên đều trong m vị trí. Xác suất một bit *cụ thể* KHÔNG bị ghi bởi một lần hash:

> 1 − 1/m

Bước 2. Add n phần tử với k hash mỗi phần tử = kn lần ghi độc lập. Xác suất bit đó vẫn là 0 sau tất cả:

> (1 − 1/m)^(kn)

Bước 3. Dùng giới hạn quen thuộc (1 − 1/m)^m → e⁻¹ khi m lớn:

> (1 − 1/m)^(kn) ≈ e^(−kn/m)

Bước 4. Vậy xác suất một bit cụ thể đang **bật** là 1 − e^(−kn/m). False positive xảy ra khi cả k bit mà y trỏ tới đều bật:

> **p ≈ (1 − e^(−kn/m))^k**

(Bước 4 giả định k bit độc lập — không chính xác tuyệt đối nhưng sai lệch không đáng kể trong thực tế; bản phân tích chặt cho cùng công thức tiệm cận.)

**Tối ưu k.** Với m, n cố định, p là hàm của k: k nhỏ → mỗi query ít bit phải trùng (dễ FP), k lớn → filter đầy nhanh (cũng dễ FP). Lấy đạo hàm theo k và giải, cực tiểu tại:

> **k* = (m/n) · ln 2 ≈ 0.693 · m/n**

Tại k*, mỗi bit bật với xác suất đúng ½ (filter "nửa đầy" — trạng thái entropy tối đa, chương 20 sẽ thấy đây không phải trùng hợp), và:

> p = (1/2)^k* ⟺ **m/n = −log₂(p) / ln 2 ≈ 1.44 · log₂(1/p)**

**Bảng tra thực dụng** (số bit mỗi phần tử, không phụ thuộc kích thước phần tử — URL 80 byte hay object 8KB đều tốn như nhau):

| FP mong muốn | bit/phần tử (m/n) | k tối ưu |
|---|---|---|
| 10% | 4.8 | 3 |
| 1% | **9.6** | 7 |
| 0.1% | 14.4 | 10 |
| 0.01% | 19.2 | 13 |

So sánh với hash set: chỉ riêng lưu hash 64-bit mỗi phần tử đã tốn 64 bit, cộng con trỏ + bucket overhead thực tế là 100–150 bit. Bloom filter 1% FP: **9.6 bit** — giảm hơn 10 lần so với phiên bản "chỉ lưu hash", và hàng trăm lần so với lưu chuỗi gốc. Bài toán 10 tỷ URL mở đầu chương: 10¹⁰ × 9.6 bit = 12GB, chạy vừa một máy.

Chú ý điều công thức *nói thầm*: p phụ thuộc n. Filter được cấp phát cho n phần tử mà bạn nhét 2n vào thì FP rate không tăng gấp đôi — nó tăng **theo cấp lũy thừa** (với cấu hình 1%, nhét gấp đôi đẩy FP lên ~9%). Bloom filter không bao giờ từ chối Add; nó chỉ âm thầm trở nên vô dụng. Phải biết n trước, hoặc dùng scalable bloom filter (chuỗi filter lớn dần).

### 4.2 HyperLogLog — đếm distinct bằng cách quan sát điều hiếm

Bài toán: đếm số phần tử **khác nhau** trong stream (unique visitors, distinct IP...). Đếm chính xác cần nhớ mọi phần tử đã gặp. HyperLogLog dựa trên một trực giác thiên tài:

**Sự kiện càng hiếm, việc nó đã xảy ra càng nói lên bạn đã thử nhiều lần.**

Hash mỗi phần tử thành chuỗi bit ngẫu nhiên đều. Xác suất một hash bắt đầu bằng đúng 20 bit 0 là 2⁻²⁰ ≈ 1 phần triệu. Vậy nếu trong stream bạn *đã thấy* một hash có 20 số 0 dẫn đầu, khả năng cao bạn đã gặp cỡ **2²⁰ ≈ 1 triệu** phần tử distinct. (Phần tử lặp lại không đổi được gì — cùng input cho cùng hash, không cho thêm "lượt thử" mới. Đó chính là lý do cấu trúc này đếm *distinct* một cách tự nhiên.)

Quan sát duy nhất cần nhớ: **R = số 0 dẫn đầu dài nhất từng thấy**. Ước lượng: n ≈ 2^R. Bộ nhớ: một số nguyên. Vấn đề: quá nhiễu. Chỉ cần *một* phần tử xui xẻo hash ra 30 số 0, ước lượng lập tức thành 2³⁰ — một outlier phá hỏng tất cả, và phương sai của 2^R là khổng lồ (chương 11: kỳ vọng đúng không cứu được phương sai lớn).

**Giải pháp: chia để lấy trung bình.** Dùng b bit đầu của hash để chia stream thành m = 2^b substream, mỗi substream có register riêng ghi max số 0 dẫn đầu (của phần bit còn lại). Giờ ta có m quan sát gần độc lập thay vì một.

Trung bình kiểu gì? Không phải trung bình cộng. Các ước lượng 2^Rⱼ có phân phối lệch nặng về phía trên — một register outlier lớn sẽ thống trị trung bình cộng y như trước. HyperLogLog dùng **harmonic mean**:

> E = α_m · m² / Σⱼ 2^(−Rⱼ)

Harmonic mean có tính chất then chốt: nó bị chi phối bởi các giá trị *nhỏ* và gần như miễn nhiễm với outlier *lớn* — đúng chiều nhiễu của bài toán này. (α_m ≈ 0.7213/(1 + 1.079/m) là hằng số sửa bias, dẫn xuất trong paper Flajolet 2007; hiểu vai trò của nó là đủ.)

**Sai số chuẩn:**

> **σ/n ≈ 1.04 / √m**

Con số phải nhớ: m = 2¹⁴ = 16384 register, mỗi register 6 bit (đủ đếm tới 2⁶⁴ số 0) → **12KB bộ nhớ, sai số 1.04/√16384 ≈ 0.81%** — bất kể stream có một nghìn hay một trăm tỷ phần tử. Đây chính xác là cấu hình của Redis `PFCOUNT`. Chi phí không phụ thuộc n: đó là điều không cấu trúc chính xác nào làm được, vì nó vi phạm lý thuyết thông tin — trừ khi bạn trả bằng sai số.

Một tính chất production quý giá: hai HLL **merge được** bằng cách lấy max từng register — cho phép đếm distinct phân tán trên nhiều máy rồi gộp, không cần chuyển dữ liệu thô.

### 4.3 Count-Min Sketch — tần suất trên stream

Bloom trả lời "có/không", HLL trả lời "bao nhiêu distinct". Câu hỏi thứ ba: "phần tử x xuất hiện **bao nhiêu lần**?" — trên stream quá lớn để giữ một map[item]count.

Cấu trúc: ma trận d hàng × w cột counter, mỗi hàng một hash function.

- **Add(x):** với mỗi hàng i, tăng counter[i][hᵢ(x)].
- **Count(x):** trả về **min** trên d hàng của counter[i][hᵢ(x)].

Vì sao min? Mỗi counter mà x trỏ tới chứa số đếm thật của x **cộng** rác từ các phần tử khác va chạm vào cùng ô — collision chỉ *cộng thêm*, không bao giờ trừ đi. Vậy mọi counter đều ≥ giá trị thật: sai số **một phía, chỉ đếm dư**. Min của d ước lượng-đều-dư là ước lượng tốt nhất, và chỉ cần *một* hàng may mắn không va chạm là ta có giá trị gần đúng.

Định lượng: với w = ⌈e/ε⌉ và d = ⌈ln(1/δ)⌉, sai số dư ≤ ε·N (N = tổng số phần tử đã add) với xác suất ≥ 1−δ. Ví dụ ε = 0.1%, δ = 1%: w ≈ 2719, d = 5 → ~54KB đếm tần suất trên stream vô hạn. Hệ quả thực dụng: Count-Min ước lượng tốt các phần tử **nặng** (heavy hitters — sai số ε·N là nhỏ so với count của chúng) và tệ với phần tử hiếm — may mắn thay, production hầu như chỉ quan tâm heavy hitters (hot key, top-K).

### 4.4 Skip List — cấu trúc có thứ tự bằng đồng xu

Ba cấu trúc trên đánh đổi *độ chính xác của câu trả lời*. Skip list đánh đổi thứ khác: dùng ngẫu nhiên để đổi lấy *sự đơn giản của cấu trúc* — câu trả lời vẫn chính xác 100%, chỉ có **thời gian chạy** là biến ngẫu nhiên.

Sorted linked list tìm kiếm O(n). Ý tưởng: xây "tầng cao tốc" — tầng 1 chứa mỗi phần tử với xác suất ½, tầng 2 là ½ của tầng 1... Tìm kiếm đi từ tầng cao nhất, lướt xa rồi hạ dần xuống:

```
L3:  1 ───────────────────────────────► 42
L2:  1 ───────────► 17 ───────────────► 42
L1:  1 ────► 9 ───► 17 ───► 25 ───────► 42 ───► 60
L0:  1 → 5 → 9 → 12 → 17 → 20 → 25 → 31 → 42 → 55 → 60
                 tìm 20: L3 hạ, L2 tới 17, L1 tới 17, L0 tới 20
```

Điểm khác biệt với balanced tree: tầng của mỗi node do **tung đồng xu** quyết định lúc insert (liên hệ chương 11: số lần tung đến khi ra mặt sấp ~ phân phối hình học, kỳ vọng 2 → chi phí bộ nhớ kỳ vọng chỉ 2 con trỏ/node), không do bất kỳ logic cân bằng nào. Phân tích trực giác vì sao expected O(log n): kỳ vọng số node ở tầng ℓ là n/2^ℓ, nên có ~log₂n tầng hữu dụng; đi ngược đường tìm kiếm từ đích lên đỉnh, tại mỗi node xác suất "leo lên tầng trên" là ½, nên kỳ vọng chỉ đi ngang ~2 bước mỗi tầng trước khi được leo — tổng kỳ vọng ~2log₂n bước. Worst case O(n) tồn tại nhưng với xác suất nhỏ theo cấp mũ.

So với red-black tree: không cần rotation, không cần case analysis 5 nhánh, insert/delete chỉ chạm vào các con trỏ lân cận (thân thiện với lock-free/fine-grained locking — lý do `java.util.concurrent.ConcurrentSkipListMap` tồn tại), và duyệt range tự nhiên vì tầng 0 là linked list có thứ tự sẵn.

## 5. Thuật toán

### Bloom Filter — Go đầy đủ

Trick production quan trọng: không cần k hash function thật. Kirsch–Mitzenmacher chứng minh gᵢ(x) = h₁(x) + i·h₂(x) mod m giữ nguyên tiệm cận FP — chỉ tốn 2 lần hash.

```go
package bloom

import "hash/fnv"

type BloomFilter struct {
    bits []uint64 // mảng bit, đóng gói 64 bit/word
    m    uint64   // tổng số bit
    k    uint64   // số hash
}

// New cấp phát filter cho n phần tử với tỷ lệ false positive p.
// m = -n·ln(p)/ln²2, k = (m/n)·ln2 — công thức mục 4.1.
func New(n uint64, p float64) *BloomFilter {
    m := uint64(math.Ceil(-float64(n) * math.Log(p) / (math.Ln2 * math.Ln2)))
    k := uint64(math.Round(float64(m) / float64(n) * math.Ln2))
    return &BloomFilter{bits: make([]uint64, (m+63)/64), m: m, k: k}
}

// hashPair sinh 2 hash độc lập từ một lần chạy FNV-64
// (double hashing: g_i = h1 + i*h2 thay cho k hash thật).
func (b *BloomFilter) hashPair(data []byte) (uint64, uint64) {
    h := fnv.New64a()
    h.Write(data)
    h1 := h.Sum64()
    h2 := h1>>33 | h1<<31 // trộn lại làm hash thứ hai
    if h2 == 0 {
        h2 = 0x9e3779b97f4a7c15 // h2 = 0 sẽ làm k probe trùng nhau
    }
    return h1, h2
}

func (b *BloomFilter) Add(data []byte) {
    h1, h2 := b.hashPair(data)
    for i := uint64(0); i < b.k; i++ {
        pos := (h1 + i*h2) % b.m
        b.bits[pos/64] |= 1 << (pos % 64) // bật bit — không bao giờ tắt
    }
}

// Contains: false = CHẮC CHẮN chưa add; true = có thể đã add (xác suất sai ≤ p).
func (b *BloomFilter) Contains(data []byte) bool {
    h1, h2 := b.hashPair(data)
    for i := uint64(0); i < b.k; i++ {
        pos := (h1 + i*h2) % b.m
        if b.bits[pos/64]&(1<<(pos%64)) == 0 {
            return false // một bit tắt là đủ kết luận tuyệt đối
        }
    }
    return true
}
```

**Biến thể — Counting Bloom Filter:** thay mỗi bit bằng counter 4 bit; Add tăng, Delete giảm, Contains kiểm tra > 0. Có delete, trả giá gấp 4 lần bộ nhớ và một rủi ro mới: delete phần tử *chưa từng add* (hoặc bị FP) sẽ giảm counter của phần tử khác → **tạo ra false negative**, phá vỡ lời hứa quan trọng nhất. Chỉ delete những gì chắc chắn đã add.

**Biến thể — Cuckoo Filter (mức giới thiệu):** lưu fingerprint ngắn (~8–12 bit) của phần tử trong bảng cuckoo hashing 2 vị trí ứng viên. Hỗ trợ delete an toàn hơn counting bloom, FP thấp hơn Bloom ở cùng bộ nhớ khi filter đầy > 95%, lookup đúng 2 lần truy cập bộ nhớ (Bloom cần k lần rải rác — tệ cho cache). Giá: insert có thể thất bại khi bảng quá đầy. Đáng cân nhắc khi cần delete hoặc khi cache locality là bottleneck.

### HyperLogLog — Go rút gọn

```go
package hll

import "math/bits"

const b = 14           // 14 bit chọn register
const m = 1 << b       // 16384 register — sai số 1.04/√m ≈ 0.81%

type HLL struct {
    reg [m]uint8 // mỗi register: max(số 0 dẫn đầu + 1) từng thấy
}

func (h *HLL) Add(hash uint64) {
    j := hash >> (64 - b)                       // b bit đầu → chọn register
    rest := hash << b                           // phần còn lại
    rho := uint8(bits.LeadingZeros64(rest)) + 1 // vị trí bit 1 đầu tiên
    if rho > h.reg[j] {
        h.reg[j] = rho // chỉ nhớ kỷ lục — add trùng không đổi gì
    }
}

func (h *HLL) Count() float64 {
    sum := 0.0
    for _, r := range h.reg {
        sum += 1.0 / float64(uint64(1)<<r) // Σ 2^(-R_j)
    }
    alpha := 0.7213 / (1 + 1.079/float64(m))
    return alpha * m * m / sum // harmonic mean — mục 4.2
    // Bản production cần thêm sửa bias khi n nhỏ (dùng linear counting)
    // và khi n gần 2^64 — xem paper HyperLogLog++ của Google.
}

// Merge: union hai HLL = max từng register — nền tảng cho đếm phân tán.
func (h *HLL) Merge(o *HLL) {
    for i := range h.reg {
        if o.reg[i] > h.reg[i] {
            h.reg[i] = o.reg[i]
        }
    }
}
```

Count-Min Sketch cài đặt tương tự bloom về kỹ thuật (d hàng hash + counter), skip list đầy đủ nằm ở mục Interview (LeetCode 1206) — khung code chính là ba vòng: hạ tầng khi node kế lớn hơn target, đi ngang khi nhỏ hơn, tung xu chọn tầng khi insert.

## 6. Trade-off

| Cấu trúc | Trả lời | Sai theo chiều nào | Bộ nhớ | Cái giá thật sự |
|---|---|---|---|---|
| Bloom filter | x ∈ S? | Chỉ false positive | ~10 bit/phần tử (1%) | Không delete, không đếm, phải biết n trước |
| Counting bloom | x ∈ S? (có delete) | FP, và FN nếu delete ẩu | ~40 bit/phần tử | Gấp 4 bloom |
| HyperLogLog | \|S\| ≈ ? | Hai phía, ±0.81% | 12KB cố định | Chỉ đếm, không liệt kê, không hỏi membership |
| Count-Min | count(x) ≈ ? | Chỉ đếm dư | vài chục KB | Tệ với phần tử hiếm |
| Skip list | sorted map | Không sai — chỉ thời gian là ngẫu nhiên | ~2 con trỏ/node (kỳ vọng) | Cache locality kém hơn B-tree |

Ba trục đánh đổi cần cân nhắc trước khi chọn:

**Chính xác vs bộ nhớ — trục chính.** Toàn bộ chương nằm trên trục này, nhưng lưu ý đường cong không tuyến tính: từ 1% xuống 0.1% FP chỉ tốn thêm 50% bit (9.6 → 14.4). Sai số rẻ nhất ở vùng "vừa phải" — đòi 0.0001% là bắt đầu trả đắt, và đến một điểm thì hash set chính xác lại rẻ hơn.

**Một phía vs hai phía.** Sai số một phía (Bloom, Count-Min) *ghép nối được* với nguồn sự thật: filter đứng trước, nguồn thật đứng sau, kết quả cuối vẫn chính xác tuyệt đối — sai số chỉ còn là chi phí hiệu năng. Sai số hai phía (HLL) thì kết quả cuối là xấp xỉ thật sự. Hệ thống của bạn nuốt được loại nào?

**Ngẫu nhiên trong dữ liệu vs trong thuật toán.** Bloom/HLL/CMS đặt cược vào hash của *dữ liệu* — nghĩa là kẻ tấn công biết hash function có thể chế tạo input toàn collision (adversarial input, bài học chương 10). Dùng seed ngẫu nhiên hoặc keyed hash khi bề mặt tiếp xúc công khai. Skip list đặt cược vào random generator *của chính nó* — miễn nhiễm với input, chỉ cần RNG không dự đoán được.

## 7. Production Applications

**Redis** là bảo tàng sống của chương này. `PFADD`/`PFCOUNT` là HyperLogLog nguyên bản: 12KB/key, sai số 0.81%, dùng dense encoding 16384 register × 6 bit (và sparse encoding khi ít phần tử). Sorted set (`ZADD`/`ZRANGEBYSCORE`) là **skip list** — Antirez giải thích lựa chọn thay vì red-black tree bằng đúng hai lý do của mục 4.4: (1) cài đặt và debug đơn giản hơn hẳn, (2) `ZRANGEBYSCORE` là duyệt linked list tầng 0 sau một lần tìm O(log n) — range query tự nhiên hơn tree traversal. Bloom filter có qua module RedisBloom (`BF.ADD`/`BF.EXISTS`), hoặc tự chế bằng `SETBIT`/`GETBIT` trên string key.

**Cassandra / RocksDB / LevelDB** — bloom filter ở đúng vị trí kinh điển: **trước mỗi SSTable trên disk**. Một read có thể phải hỏi hàng chục SSTable; mỗi SSTable giữ một bloom filter trong RAM (~10 bit/key), câu trả lời "không có" (chắc chắn đúng — không có false negative!) tiết kiệm một lần disk seek. Với FP 1%, chỉ 1% lần đọc phải chạm disk oan. Toán ở đây: chọn `bloom_filter_fp_chance` chính là chọn điểm trên đường cong m/n = 1.44·log₂(1/p).

**Chrome Safe Browsing**: kiểm tra URL độc hại. Danh sách đầy đủ nằm trên server Google; client giữ một cấu trúc xác suất nén (lịch sử là bloom filter, nay là prefix set cùng họ). URL "sạch" (đa số tuyệt đối) được xác nhận cục bộ tức thì; chỉ khi cấu trúc báo "có thể độc" mới hỏi server — false positive chỉ tốn một network call, false negative thì không được phép có, và toán đảm bảo điều đó.

**Kafka / stream dedup**: exactly-once tự chế thường dựa trên "đã xử lý message ID này chưa?" — bloom filter theo cửa sổ thời gian (mỗi giờ một filter, xoay vòng) chặn phần lớn duplicate mà không cần state store khổng lồ; phần lọt qua đi vào kiểm tra chính xác. Lại là kiến trúc filter-trước-nguồn-thật.

**CDN / API gateway — hot key detection**: Count-Min sketch đếm tần suất mọi key trong cửa sổ trượt với vài chục KB; key nào vượt ngưỡng ε·N nổi lên là heavy hitter → kích hoạt cache riêng hoặc rate limit. Sai số một phía đúng chiều cần: thà báo nhầm một key nguội thành nóng còn hơn bỏ sót key nóng.

**PostgreSQL** — extension `postgresql-hll` cho phép cột kiểu `hll`: `SELECT date, #hll_union_agg(visitors) FROM daily_stats GROUP BY date` đếm distinct hàng tỷ dòng trong mili giây, và nhờ tính chất merge của HLL, pre-aggregate theo ngày rồi union theo tháng **vẫn đúng** — điều `COUNT(DISTINCT)` không bao giờ cho phép (distinct không cộng được qua partition).

## 8. Interview

Chương này xuất hiện ở hai cửa: câu hỏi system design (thường xuyên) và implement trực tiếp (thỉnh thoảng).

**Dạng 1 — "Design a web crawler / URL shortener" (system design kinh điển):** điểm chấm nằm ở câu "làm sao biết URL đã crawl / short code đã cấp?". Trả lời hash set là trả lời đúng-nhưng-junior. Trả lời đủ: ước lượng bộ nhớ hash set (làm phép nhân ra GB/TB ngay trước mặt interviewer), đề xuất bloom filter, **nêu rõ hệ quả của FP** (URL shortener: FP khi kiểm tra code đã cấp → bỏ phí một code, vô hại; nhưng FP trong "URL này đã crawl" → mất trang — nêu được chiều sai số nào rơi vào đâu là điểm senior), chốt bằng con số 9.6 bit/phần tử cho 1%.

**Dạng 2 — "Đếm unique visitor của 1 tỷ event/ngày":** đáp án mong đợi là HyperLogLog. Chuỗi trình bày ghi điểm: vì sao count distinct chính xác không scale (memory tỷ lệ n, không merge được qua shard) → trực giác leading zeros → 12KB/0.81% → merge được nên đếm phân tán và đếm theo nhiều chiều (mỗi country một HLL, union ra global) gần như miễn phí.

**Dạng 3 — Design Skiplist (LeetCode 1206):** một trong số ít bài LeetCode yêu cầu cài cấu trúc ngẫu nhiên. Khung: node có mảng forward con trỏ; search đi từ tầng cao, `for level--` hạ dần; insert tung xu (`rand.Intn(2)`) quyết định số tầng, nối lại forward tại mỗi tầng đi qua. Bẫy phổ biến: quên giới hạn max level (~log₂ của n dự kiến), và cài delete quên nối lại *tất cả* các tầng của node.

**Lỗi tư duy thường gặp:**

- **Dùng bloom filter khi workload cần delete.** "User unsubscribe thì xóa khỏi filter" — không xóa được; bit chung với phần tử khác. Cần delete → counting bloom/cuckoo filter, hoặc nghĩ lại thiết kế (filter theo thế hệ, rebuild định kỳ).
- **Quên FP rate tăng khi filter đầy.** Filter cấp cho 10M phần tử là hợp đồng 10M phần tử; hệ thống lớn gấp ba sau hai năm thì FP không còn 1% mà có thể lên hàng chục phần trăm — quả bom hẹn giờ đúng kiểu chương 10, phát nổ khi công ty thành công.
- **Trả lời "dùng HLL" cho bài cần danh sách phần tử.** HLL đếm được nhưng không liệt kê được, không hỏi membership được — chọn cấu trúc theo *truy vấn*, không theo độ nổi tiếng.
- **Không nêu được chuyện gì xảy ra khi sai.** Câu hỏi follow-up chắc chắn có: "FP thì hệ thống của bạn làm gì?" Không có kịch bản cho câu này nghĩa là chưa hiểu mình vừa đánh đổi cái gì.

## 9. Anti-pattern

**Dùng cấu trúc xác suất cho dữ liệu không được phép sai.** Bloom filter chặn "email đã gửi chưa" cho email marketing: ổn. Cùng filter đó cho "giao dịch đã thanh toán chưa": thảm họa — FP nghĩa là *từ chối thanh toán hợp lệ* hoặc tệ hơn. Câu hỏi bắt buộc trước khi dùng: "false positive ở đây dịch ra hành vi gì của hệ thống, và hành vi đó có sửa được không?"

**Delete trên bloom filter thường.** Tắt bit của phần tử này là tắt bit của phần tử khác → false negative → mất luôn tính chất duy nhất khiến bloom filter đáng dùng. Đây là lỗi triển khai thật, khó phát hiện vì hệ thống vẫn chạy — chỉ có điều thỉnh thoảng nói dối theo chiều không ai kiểm tra.

**Cấp phát theo n hôm nay.** Đã nói ở Interview nhưng đáng nhắc như anti-pattern vận hành: bloom filter không có cảnh báo "sắp đầy". Hãy export metric tỷ lệ bit đã bật (đo được bằng popcount); vượt 50% (điểm tối ưu k = m/n·ln2) là lúc rebuild với m lớn hơn.

**Tin HLL ở vùng n nhỏ mà không sửa bias.** Công thức harmonic mean lệch đáng kể khi n < ~2.5m; bản production (Redis, HLL++ của Google) chuyển sang linear counting ở vùng này. Tự cài bản "rút gọn" của mục 5 rồi dùng đếm những tập vài nghìn phần tử sẽ cho số lệch có hệ thống — dùng thư viện đã sửa bias, hoặc biết mình đang ở vùng nào của đường cong.

**Skip list với RNG dự đoán được ở bề mặt công khai.** Expected O(log n) đứng trên giả định đồng xu công bằng và *không dự đoán được*. Seed cố định + kẻ xấu điều khiển thứ tự insert = mọi node đều tầng 0 = O(n) — phiên bản skip list của hash flooding (chương 10, 12).

## 10. Best Practices

**Nên:**

- Bắt đầu từ truy vấn, không từ cấu trúc: membership → Bloom/Cuckoo; đếm distinct → HLL; tần suất/top-K → Count-Min; sorted map concurrent → skip list. Mỗi cấu trúc trả lời đúng một câu hỏi — bảng ở mục 6 là bản đồ chọn.
- Viết rõ **error budget** vào design doc như một con số: "FP ≤ 1%, hệ quả: ≤1% request đọc disk thừa" — sai số có kiểm soát chỉ có nghĩa khi được kiểm soát *thành văn*.
- Dùng thư viện đã kiểm chứng (RedisBloom, `axiomhq/hyperloglog`, `bits-and-blooms/bloom` cho Go) — code mục 5 để hiểu bản chất; bản production cần sửa bias, sparse encoding, và test thống kê.
- Monitor các đại lượng toán học nói cho bạn biết: tỷ lệ bit bật của bloom filter, n hiện tại so với n thiết kế.
- Nhớ ba con số neo: **9.6 bit** (bloom 1%), **12KB / 0.81%** (HLL chuẩn), **1.04/√m** (đường cong sai số HLL).

**Không nên:**

- Không dùng cho tiền, quyền, và mọi thứ có chữ "audit" — chính xác tuyệt đối hoặc không gì cả.
- Không dùng khi n nhỏ: dưới vài triệu phần tử, hash set vài chục MB đơn giản hơn, chính xác hơn, và không ai phải hiểu công thức nào lúc 3 giờ sáng.
- Không ghép nhiều cấu trúc xác suất nối tiếp mà quên sai số **tích lũy**: bloom 1% đứng trước count-min ε·N không cho hệ thống "1%" — cộng/nhân sai số như chương 11 dạy trước khi hứa SLA.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Chứng minh trong ba câu: vì sao Bloom filter không bao giờ false negative, và thao tác nào (nếu thêm vào) sẽ phá vỡ tính chất đó?
2. HyperLogLog dùng harmonic mean thay vì arithmetic mean của các ước lượng 2^Rⱼ — điều gì sẽ hỏng nếu đổi sang arithmetic mean, và nó liên quan gì đến phương sai của chương 11?
3. Hệ thống của bạn cần "kiểm tra sản phẩm đã index chưa" và thỉnh thoảng phải xóa sản phẩm khỏi index. Những lựa chọn nào khả dĩ, và bạn hỏi thêm điều gì về workload trước khi chọn?

---

*Chương tiếp theo: [24 — Advanced Data Structures](/series/math-for-engineers/level-5-production/24-advanced-data-structures/) — khi câu trả lời phải chính xác tuyệt đối, mỗi loại truy vấn chuyên dụng có một cấu trúc chuyên dụng: Trie, Segment Tree, Union-Find, và bài học thiết kế đẹp nhất mọi thời — LRU Cache.*
