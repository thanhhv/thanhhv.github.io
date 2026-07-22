+++
title = "Chương 20 — Information Theory: Đo lường thông tin và giới hạn của nén"
date = "2026-07-20T10:20:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 4 – Advanced Mathematics
> Yêu cầu trước: Chương 07 (Trees), Chương 11 (Probability), Chương 15 (Greedy)

---

## 1. Problem Statement

Hãy làm một thí nghiệm mất 30 giây. Lấy một file log text 100MB, chạy `gzip`: file co lại còn ~25MB — nén được 75%. Lấy một file JPEG 100MB, chạy `gzip`: file thu được ~99.9MB, thậm chí đôi khi *lớn hơn* file gốc. Cùng một thuật toán, cùng 100MB đầu vào, một bên nén được 75%, một bên 0%.

Câu hỏi ngây thơ là "gzip không hợp với ảnh". Câu hỏi đúng bản chất là: **cái gì trong dữ liệu quyết định nó nén được bao nhiêu — và có tồn tại giới hạn mà không thuật toán nào vượt qua được không?**

Câu hỏi này không hàn lâm chút nào. Nó xuất hiện ở khắp nơi trong hệ thống bạn vận hành:

- Bật compression cho Kafka topic — chọn gzip, lz4 hay zstd, và kỳ vọng tiết kiệm bao nhiêu bandwidth?
- Vì sao Parquet nén dữ liệu tốt hơn CSV nhiều lần, dù cùng một nội dung?
- Vì sao nén một file đã mã hóa (encrypted) luôn vô vọng?
- Password 12 ký tự "mạnh" đến mức nào — đo bằng gì?
- Hàm loss cross-entropy mà mọi model deep learning đang minimize — nó đo cái gì?

Năm 1948, Claude Shannon trả lời tất cả những câu hỏi này bằng một bài báo duy nhất, khai sinh Information Theory. Đóng góp nền tảng của ông: **thông tin là một đại lượng đo được**, có đơn vị (bit), có công thức, và có định lý về giới hạn. Nếu không có nó, "nén dữ liệu" chỉ là bộ sưu tập mẹo vặt — không ai biết mình đang cách trần tối ưu bao xa, và không ai chứng minh được rằng trần đó tồn tại.

## 2. Trực giác

### Thông tin = độ bất ngờ

Xét ba bản tin bạn nhận được sáng nay:

1. "Mặt trời mọc hướng Đông." — bạn không học được gì. Thông tin: **0**.
2. "Ngày mai Hà Nội có mưa." — có chút thông tin, nhưng mưa ở Hà Nội không hiếm.
3. "Ngày mai Hà Nội có tuyết." — bạn dừng mọi việc đang làm. Thông tin: **rất lớn**.

Trực giác then chốt của Shannon: **lượng thông tin của một sự kiện tỷ lệ nghịch với xác suất của nó**. Sự kiện chắc chắn xảy ra (p = 1) mang 0 thông tin. Sự kiện càng hiếm, việc nó xảy ra càng "bất ngờ", càng nhiều thông tin. Thông tin không nằm trong bản thân dữ liệu — nó nằm trong **mức độ khó đoán** của dữ liệu.

### Áp vào bài toán nén

Nén dữ liệu, nhìn từ góc này, là trò chơi đặt cược vào sự dự đoán được:

- File log text: ký tự `e`, `t`, space xuất hiện dày đặc; sau `ERRO` gần như chắc chắn là `R`. Rất dễ đoán → ít bất ngờ → **ít thông tin thật sự trong mỗi byte** → nén được nhiều. Mỗi byte của log đang "phung phí" 8 bit để chở một lượng thông tin ít hơn 8 bit nhiều.
- File JPEG: bản thân JPEG đã là kết quả của một lần nén. Byte tiếp theo gần như ngẫu nhiên hoàn toàn, không đoán được → mỗi byte đã chở gần đủ 8 bit thông tin → **không còn gì để vắt**.

Ý tưởng nén vì thế rất tự nhiên: **ký hiệu xuất hiện thường xuyên → gán mã ngắn; ký hiệu hiếm → gán mã dài**. Morse code làm đúng điều này từ năm 1838: `E` (chữ phổ biến nhất tiếng Anh) là một dấu chấm, `Q` là bốn ký hiệu. Shannon biến trực giác này thành định lý: ông chỉ ra chính xác *ngắn đến đâu là hết cỡ*.

### Nếu không có khái niệm này thì sao?

Không có thước đo thông tin, bạn không phân biệt được "thuật toán nén của tôi kém" với "dữ liệu này hết nén được rồi". Team sẽ tốn hàng tuần thử nén dữ liệu đã mã hóa, hoặc ngược lại, hài lòng với tỷ lệ nén 2:1 trên dữ liệu mà lý thuyết cho phép 10:1. Entropy chính là đường baseline: nó nói cho bạn biết trần tối ưu nằm ở đâu **trước khi** viết bất kỳ dòng code nào.

## 3. First Principles

### Vì sao lại là logarit?

Ta muốn một hàm I(p) đo "độ bất ngờ" của sự kiện có xác suất p. Ba yêu cầu tối thiểu, đều xuất phát từ lẽ thường:

1. **I(1) = 0** — sự kiện chắc chắn không có gì bất ngờ.
2. **p càng nhỏ, I(p) càng lớn** — hiếm hơn thì bất ngờ hơn.
3. **I(p·q) = I(p) + I(q)** với hai sự kiện độc lập — biết kết quả hai lần tung xúc xắc độc lập, thông tin nhận được phải bằng *tổng* thông tin từng lần. Thông tin phải cộng được, như byte cộng được.

Yêu cầu thứ ba là chốt chặn: hàm biến phép nhân thành phép cộng, thỏa cả (1) và (2), về bản chất chỉ có một họ — logarit với dấu âm:

> **I(p) = −log₂ p** (self-information, đơn vị: bit)

Chọn cơ số 2 để đơn vị là bit — trùng khớp đẹp đẽ với đơn vị lưu trữ của máy tính. Kiểm tra lại trực giác: sự kiện p = 1/2 (tung đồng xu) mang −log₂(1/2) = 1 bit. Sự kiện p = 1/1024 mang 10 bit. Một sự kiện càng hiếm gấp đôi, thông tin tăng thêm đúng 1 bit.

### Từ một sự kiện đến một nguồn dữ liệu: entropy

Một nguồn dữ liệu (source) phát ra các ký hiệu với phân phối xác suất P. Mỗi ký hiệu x mang I(x) = −log₂ p(x) bit. Lượng thông tin **trung bình** mỗi ký hiệu chính là kỳ vọng của độ bất ngờ — đúng khái niệm expected value của chương 11:

> **H(P) = E[I] = Σ −p(x) · log₂ p(x)**

Entropy là "độ bất ngờ trung bình" của nguồn. Nguồn dễ đoán → entropy thấp. Nguồn hỗn loạn hoàn toàn → entropy đạt max = log₂(số ký hiệu) khi phân phối đều (mọi ký hiệu đồng xác suất — không có gì để đặt cược).

### Ý nghĩa vận hành: entropy = số bit tối thiểu

Đây là bước nhảy từ triết học sang kỹ thuật, và là nội dung của **Shannon's Source Coding Theorem**:

> Một nguồn có entropy H bit/ký hiệu **không thể** được mã hóa (không mất mát) với ít hơn H bit/ký hiệu trung bình; và luôn **tồn tại** cách mã hóa đạt trung bình gần H tùy ý (khi mã hóa theo khối đủ dài).

Trực giác của nửa "không thể": mã hóa là gán mỗi chuỗi nguồn một chuỗi bit sao cho giải mã được duy nhất. Số chuỗi bit độ dài k chỉ có 2ᵏ. Nguồn n ký hiệu về cơ bản tập trung xác suất vào ~2^(nH) chuỗi "điển hình" (typical set); muốn phân biệt được từng ấy chuỗi, cần ít nhất nH bit — đây là nguyên lý chuồng bồ câu (chương 05) mặc áo xác suất. Nửa "tồn tại" thì Huffman ở mục 5 sẽ cho ta cầm nắm được.

Hệ quả đọc được ngay: **tỷ lệ nén tối đa = H / (số bit đang dùng)**. Text ASCII dùng 8 bit/ký tự nhưng entropy tiếng Anh thực tế chỉ ~1–1.5 bit/ký tự (nhờ ngữ cảnh) → trần nén khoảng 80%+, và gzip đạt 70–75% là đã khá gần trần. JPEG có entropy ≈ 8 bit/byte → trần nén 0%. Bí ẩn mở đầu chương được giải trong một dòng.

## 4. Mathematical Model

### Tính entropy bằng tay — hai đồng xu

**Đồng xu công bằng** (p = 0.5 mỗi mặt):

> H = −0.5·log₂0.5 − 0.5·log₂0.5 = 0.5 + 0.5 = **1 bit**

Hợp lý: mỗi lần tung cần đúng 1 bit để ghi lại, không cách nào tiết kiệm hơn. Dữ liệu ngẫu nhiên đều là dữ liệu "đặc" nhất.

**Đồng xu lệch** (p = 0.9 ngửa, 0.1 sấp):

> H = −0.9·log₂0.9 − 0.1·log₂0.1 ≈ 0.9·0.152 + 0.1·3.322 ≈ **0.469 bit**

Kết quả gây ngạc nhiên lần đầu nhìn thấy: một chuỗi tung đồng xu lệch — vẫn là chuỗi H/T trông "ngẫu nhiên" — về lý thuyết nén được xuống **dưới một nửa**. Cách hiện thực: gom khối. Ví dụ mã hóa từng cặp kết quả: cặp HH (xác suất 0.81) gán mã `0`, HT (0.09) gán `10`, TH (0.09) gán `110`, TT (0.01) gán `111`. Trung bình ≈ 1.29 bit/cặp ≈ 0.645 bit/lần tung — đã dưới 1 bit, và khối càng dài càng tiến sát 0.469. Đây chính là cơ chế mọi thuật toán nén khai thác: **phân phối càng lệch, entropy càng thấp, nén càng sâu**.

```
H(p) của đồng xu lệch xác suất p:

 1.0 ┤          ╭────╮
     │        ╭─╯    ╰─╮
     │      ╭─╯        ╰─╮
 0.5 ┤    ╭─╯            ╰─╮
     │  ╭─╯                ╰─╮
     │ ╭╯                    ╰╮
 0.0 ┼─┬──────────┬──────────┬─
     0.0         0.5        1.0  p

Max tại p = 0.5 (khó đoán nhất). Bằng 0 tại p = 0 và p = 1 (chắc chắn).
```

### Entropy của password: đo độ mạnh bằng bit

"Bits of entropy" của password chính là entropy của **quy trình sinh ra nó**. Password chọn đều ngẫu nhiên từ bảng chữ cái A ký tự, độ dài L: H = L·log₂A.

| Cách sinh password | Entropy | Thời gian brute-force @10¹⁰ lần thử/giây |
|---|---|---|
| 8 chữ thường (26 ký tự) | 8·4.7 ≈ 37.6 bit | ~20 giây |
| 12 ký tự đủ loại (94 ký tự) | 12·6.55 ≈ 78.6 bit | ~1.4 triệu năm |
| 4 từ ngẫu nhiên từ danh sách 7776 từ (Diceware) | 4·12.9 ≈ 51.7 bit | ~8 ngày |
| "P@ssw0rd2024!" | rất thấp | phút — vì kẻ tấn công dò theo *phân phối* password người thật |

Bài học then chốt: entropy đo **không gian lựa chọn của quy trình sinh**, không đo hình thức của chuỗi kết quả. "P@ssw0rd2024!" trông phức tạp nhưng nằm trong vùng xác suất cao của phân phối password con người — kẻ tấn công dùng dictionary attack chính là đang khai thác entropy thấp đó, hệt như gzip khai thác entropy thấp của text.

### Cross-entropy và KL divergence — cây cầu sang Machine Learning

Điều gì xảy ra khi bạn mã hóa dữ liệu có phân phối thật P bằng bộ mã thiết kế cho phân phối Q (đoán sai phân phối)? Chi phí trung bình trở thành:

> **H(P, Q) = Σ −p(x) · log₂ q(x)** (cross-entropy)

Luôn có H(P, Q) ≥ H(P), dấu bằng khi và chỉ khi Q = P. Phần trội ra là cái giá của việc đoán sai:

> **D_KL(P‖Q) = H(P, Q) − H(P) ≥ 0** (Kullback–Leibler divergence)

KL divergence đo "khoảng cách" từ Q đến P bằng đơn vị bit lãng phí. (Lưu ý: không phải metric thật — không đối xứng, D(P‖Q) ≠ D(Q‖P).)

Giờ nhìn lại deep learning: model phân loại của bạn xuất ra phân phối dự đoán Q; nhãn thật là phân phối P (one-hot). Hàm loss cross-entropy mà optimizer đang minimize **chính là H(P, Q)** ở trên. Vì H(P) cố định theo dữ liệu, minimize cross-entropy ≡ minimize KL divergence ≡ **kéo phân phối dự đoán của model về sát phân phối thật**. Huấn luyện một language model là dạy nó *dự đoán* — và theo Shannon, dự đoán tốt và nén tốt là một: một LLM tốt về bản chất là một bộ nén text cực mạnh (các benchmark "LLM as compressor" xác nhận điều này bằng thực nghiệm).

## 5. Thuật toán

### Huffman coding — greedy chạm trần entropy

Bài toán: cho tần suất các ký hiệu, xây bộ mã prefix-free (không mã nào là tiền tố của mã khác — để giải mã không cần dấu phân cách) với độ dài trung bình nhỏ nhất. Một bộ mã prefix-free tương ứng 1-1 với một cây nhị phân: ký hiệu ở lá, đường đi từ gốc (trái = 0, phải = 1) là mã. Độ dài mã = độ sâu lá.

Ý tưởng greedy của Huffman (1952): **hai ký hiệu hiếm nhất xứng đáng nằm sâu nhất** — gộp chúng thành một node, coi như một "ký hiệu ảo" có tần suất bằng tổng, rồi lặp lại.

```go
package huffman

import "container/heap"

// Node là một node trong cây Huffman.
type Node struct {
	Symbol      byte  // chỉ có nghĩa ở lá
	Freq        int   // tần suất (hoặc tổng tần suất cây con)
	Left, Right *Node // nil nếu là lá
}

// nodeHeap: min-heap theo tần suất (xem chương 16).
type nodeHeap []*Node

func (h nodeHeap) Len() int            { return len(h) }
func (h nodeHeap) Less(i, j int) bool  { return h[i].Freq < h[j].Freq }
func (h nodeHeap) Swap(i, j int)       { h[i], h[j] = h[j], h[i] }
func (h *nodeHeap) Push(x any)         { *h = append(*h, x.(*Node)) }
func (h *nodeHeap) Pop() any {
	old := *h
	n := len(old)
	x := old[n-1]
	*h = old[:n-1]
	return x
}

// Build dựng cây Huffman từ bảng tần suất.
// Độ phức tạp: O(k log k) với k = số ký hiệu phân biệt.
func Build(freq map[byte]int) *Node {
	h := &nodeHeap{}
	for sym, f := range freq {
		*h = append(*h, &Node{Symbol: sym, Freq: f})
	}
	heap.Init(h)

	for h.Len() > 1 {
		// Greedy: luôn gộp hai node có tần suất nhỏ nhất.
		a := heap.Pop(h).(*Node)
		b := heap.Pop(h).(*Node)
		heap.Push(h, &Node{Freq: a.Freq + b.Freq, Left: a, Right: b})
	}
	return heap.Pop(h).(*Node) // node cuối cùng là gốc
}

// Codes duyệt cây, sinh bảng mã: ký hiệu → chuỗi bit.
func Codes(root *Node) map[byte]string {
	table := make(map[byte]string)
	var walk func(n *Node, prefix string)
	walk = func(n *Node, prefix string) {
		if n.Left == nil && n.Right == nil { // lá
			table[n.Symbol] = prefix
			return
		}
		walk(n.Left, prefix+"0")
		walk(n.Right, prefix+"1")
	}
	walk(root, "")
	return table
}
```

Chạy trên nguồn `A:45, B:25, C:15, D:10, E:5`:

```
              [100]
             0/    \1
           [55]    A(45)      A → 1       (1 bit, ký hiệu phổ biến nhất)
          0/   \1             B → 00      (2 bit)
        B(25)  [30]           C → 010     (3 bit)
              0/   \1         D → 0111    (4 bit)
            C(15)  [15]       E → 0110    (4 bit)
                  0/  \1
                E(5)  D(10)

Độ dài TB = 0.45·1 + 0.25·2 + 0.15·3 + 0.10·4 + 0.05·4 = 2.00 bit
Entropy H ≈ 1.99 bit → Huffman cách trần Shannon 0.01 bit. So với 3 bit
của mã cố định (⌈log₂5⌉), tiết kiệm 33%.
```

**Vì sao greedy tối ưu ở đây?** Bằng exchange argument quen thuộc từ chương 15. Xét cây mã tối ưu bất kỳ: (1) hai lá sâu nhất phải là anh em (nếu một lá sâu nhất mồ côi, cắt bỏ node cha thừa sẽ được cây tốt hơn — mâu thuẫn); (2) nếu ở độ sâu lớn nhất đó *không phải* là hai ký hiệu hiếm nhất, hoán đổi ký hiệu hiếm nhất xuống đó với ký hiệu đang ở đó — tổng chi phí Σ freq·depth không tăng (vật nhẹ xuống sâu, vật nặng lên nông). Vậy **tồn tại cây tối ưu trong đó hai ký hiệu hiếm nhất là anh em ở đáy** — đúng nước đi đầu tiên của Huffman. Quy nạp trên bài toán đã gộp hai ký hiệu đó cho phần còn lại. Đây là mẫu chứng minh greedy kinh điển: không chứng minh "mọi lời giải tối ưu giống greedy", chỉ cần "luôn tồn tại lời giải tối ưu *chứa* nước đi greedy".

### Giới hạn của Huffman → arithmetic coding và ANS

Huffman tối ưu **trong lớp mã gán số bit nguyên cho từng ký hiệu**. Nhưng entropy thì không nguyên: ký hiệu p = 0.9 chỉ đáng 0.152 bit, Huffman vẫn buộc phải chi trọn 1 bit — lãng phí gần 6 lần. Với phân phối lệch mạnh, khoảng cách tới trần Shannon trở nên đáng kể (tối đa gần 1 bit/ký hiệu).

**Arithmetic coding** phá rào bằng cách bỏ hẳn ý tưởng "mỗi ký hiệu một mã": nó mã hóa **cả chuỗi thành một số thực duy nhất** trong [0, 1). Trực giác: chia [0,1) thành các đoạn tỷ lệ với xác suất ký hiệu đầu; ký hiệu nào xuất hiện, "zoom" vào đoạn đó rồi chia tiếp theo cùng tỷ lệ cho ký hiệu thứ hai; cứ thế. Chuỗi càng "dễ đoán" thì đoạn kết quả càng rộng → cần càng ít chữ số nhị phân để chỉ định một điểm trong đoạn. Chi phí mỗi ký hiệu tiệm cận đúng −log₂p — trả **đúng phần lẻ**, không làm tròn. **ANS (Asymmetric Numeral Systems)** là hậu duệ hiện đại: cùng chất lượng nén sát entropy nhưng tốc độ ngang Huffman nhờ chỉ dùng số học nguyên trên một biến state — đây là entropy coder bên trong zstd (tANS) và JPEG XL. Bộ ba Huffman → arithmetic → ANS là ví dụ đẹp về một định lý (Shannon) treo mục tiêu suốt 60 năm để các thuật toán lần lượt tiến sát.

Ghi chú kiến trúc: các bộ nén thực tế (gzip, zstd) là **hai tầng** — tầng modeling (LZ77: thay chuỗi lặp bằng tham chiếu ngược, khai thác cấu trúc/ngữ cảnh) rồi tầng entropy coding (Huffman trong gzip, tANS trong zstd) vắt kiệt phân phối lệch của output tầng một. Entropy coding là tầng "toán thuần"; modeling là tầng "hiểu dữ liệu".

### Vì sao dữ liệu đã nén/mã hóa không nén thêm được

Giờ có thể trả lời chặt chẽ: bộ nén tốt tạo ra output có phân phối gần như đều trên các byte — vì nếu output còn lệch (còn dự đoán được), tức là còn nén được nữa, mâu thuẫn với "bộ nén tốt". Output gần-đều có entropy ≈ 8 bit/byte = đã kịch trần. Dữ liệu mã hóa còn triệt để hơn: ciphertext *phải* không phân biệt được với ngẫu nhiên (nếu không, kẻ tấn công có chỗ bám). Hệ quả vận hành quan trọng: **nén trước, mã hóa sau** — đảo thứ tự là vứt bỏ toàn bộ lợi ích nén. Và một mẹo giám sát thực dụng: entropy của payload là detector rẻ tiền cho dữ liệu đã nén/mã hóa — ransomware detection và DLP dùng đúng kỹ thuật này.

## 6. Trade-off

**Tỷ lệ nén vs tốc độ vs CPU.** Không tồn tại bộ nén "tốt nhất" — chỉ có điểm phù hợp trên đường cong trade-off. Số liệu điển hình trên text/log (thứ tự độ lớn, đo thực tế dao động theo dữ liệu):

| Codec | Tỷ lệ nén | Tốc độ nén | Tốc độ giải nén | Điểm dùng đúng |
|---|---|---|---|---|
| none | 1.0× | ∞ | ∞ | dữ liệu đã nén, latency cực nhạy |
| lz4 | ~2.1× | ~700 MB/s | ~4 GB/s | hot path, service-to-service |
| snappy | ~2.0× | ~500 MB/s | ~1.8 GB/s | tương tự lz4 (Google legacy) |
| zstd (level 3) | ~2.9× | ~350 MB/s | ~1 GB/s | mặc định tốt cho đa số hệ thống |
| zstd (level 19) | ~3.5× | ~5 MB/s | ~1 GB/s | nén một lần, đọc nhiều lần |
| gzip (level 6) | ~2.8× | ~60 MB/s | ~400 MB/s | tương thích phổ quát |
| xz/lzma | ~4.0× | ~3 MB/s | ~80 MB/s | archive lạnh |

Hai bất đối xứng đáng khắc cốt: (1) **giải nén nhanh hơn nén nhiều lần** và gần như không đổi theo level — nên với dữ liệu write-once-read-many, tăng level nén là gần như "miễn phí" phía đọc; (2) lz4/snappy chấp nhận bỏ xa trần entropy để đổi lấy tốc độ — chúng cố tình chỉ làm LZ modeling sơ sài với entropy coding tối giản.

**Nén sâu hơn vs khả năng truy cập.** Nén cả file đạt tỷ lệ tốt nhất nhưng muốn đọc 1 record phải giải nén từ đầu. Nén theo block (Parquet page, Kafka batch) hy sinh vài phần trăm tỷ lệ (mỗi block khởi động lại "ngữ cảnh") đổi lấy random access và parallelism. Kích thước block chính là núm vặn trade-off này.

**Lossless vs lossy.** Shannon chặn dưới cho nén *không mất mát*. JPEG/MP3/H.264 nén 10–100× vì chúng chơi trò khác: **vứt bớt thông tin** mà mắt/tai người không nhận ra (rate–distortion theory — nhánh thứ hai của cùng lý thuyết). Với dữ liệu hệ thống, tương đương lossy là sampling và aggregation trong observability: trace sampling 1% chính là quyết định rate–distortion.

**Khi nào KHÔNG nén:** payload nhỏ (< vài trăm byte — header và block overhead nuốt hết lợi ích), dữ liệu đã nén (ảnh, video, ciphertext — kiểm tra entropy trước khi bật codec!), và hot path nơi p99 latency quý hơn bandwidth.

## 7. Production Applications

**Kafka compression** — nơi bảng trade-off trên thành file config. Producer nén cả *batch* message (`compression.type`), broker lưu nguyên dạng nén (zero-copy ra consumer). Nén theo batch là quyết định thấm nhuần entropy: các message cùng topic giống nhau (cùng schema JSON, key lặp) → gộp batch làm phân phối lệch lộ rõ cho codec khai thác — batch càng lớn (`linger.ms`) nén càng sâu. Thực tế ngành hội tụ về **lz4 hoặc zstd**: gzip từng là mặc định phổ biến nhưng ngốn CPU producer gấp nhiều lần trong khi zstd nén tốt hơn với chi phí thấp hơn; lz4 cho pipeline nhạy latency. Riêng dữ liệu đã nén (ảnh trong message) thì `none` — đúng bài học mục 5.

**Parquet và columnar encoding** — vì sao cột nén tốt hơn hàng đến vậy. Toán nằm ở chỗ: **entropy của từng cột thấp hơn nhiều entropy của hàng trộn lẫn**. Cột `country` có 200 giá trị phân biệt với phân phối lệch nặng → dictionary encoding (thay chuỗi bằng index nhỏ) + RLE (run-length: chuỗi giá trị lặp thành cặp (giá trị, số lần)) ép cột về sát entropy thật của nó, đôi khi vài *phần trăm* kích thước gốc. Cùng dữ liệu đó dưới dạng CSV theo hàng, các cột entropy thấp bị trộn với cột entropy cao (id, timestamp), phân phối tổng hợp phẳng ra, codec mất chỗ bám. Tầng cuối Parquet mới gọi zstd/snappy quét phần còn sót — encoding theo cột mới là người hùng thầm lặng.

**PostgreSQL TOAST**: giá trị lớn (text, jsonb) vượt ~2KB được nén bằng pglz hoặc lz4 (từ PG14, `default_toast_compression = lz4` — nhanh hơn rõ rệt với tỷ lệ tương đương) trước khi cắt lát lưu ngoài bảng. JSONB với key lặp đi lặp lại là mồi ngon entropy thấp. Chi tiết tinh tế: TOAST nén *từng giá trị riêng lẻ* nên không khai thác được sự giống nhau *giữa các hàng* — đó là lý do cùng dữ liệu chuyển sang columnar store nén sâu hơn hẳn.

**Machine learning**: hàm loss của mọi bài toán phân loại và của việc pretrain LLM là cross-entropy H(P, Q) — mục 4. Con số "loss 3.2" trên dashboard training đọc được bằng ngôn ngữ Shannon: model đang cần trung bình 3.2 nat (~4.6 bit) cho mỗi token, tức nếu dùng model này làm bộ nén thì đạt từng ấy bit/token; loss giảm = model nén ngôn ngữ tốt hơn = dự đoán tốt hơn. Perplexity = 2^(cross-entropy theo bit) — cùng một đại lượng, đổi thang đo.

**Log compression trong observability pipeline**: log là dữ liệu entropy thấp bậc nhất trong hệ thống (template lặp, chỉ vài field biến thiên). Loki tách nhãn khỏi nội dung rồi nén chunk; các hệ chuyên dụng (CLZ-style, template mining) tách log thành *template + biến số* — bản chất là tách phần entropy ~0 (template) khỏi phần mang thông tin thật (tham số), đạt 20–50×. Khi trả tiền lưu trữ log theo GB, hiểu entropy là hiểu hóa đơn.

## 8. Interview

Information theory hiếm khi được hỏi trực diện kiểu "phát biểu định lý Shannon", nhưng nó ẩn dưới nhiều câu quen mặt:

**Dạng 1 — Huffman và biến thể.** Huffman ít khi bị yêu cầu code trọn trong 45 phút, nhưng hay xuất hiện dưới dạng: "cho tần suất, xây mã prefix-free ngắn nhất", hoặc bài con của nó — nhận diện được "gộp hai phần tử nhỏ nhất bằng min-heap" là mẫu chung với các bài như nối dây thừng chi phí nhỏ nhất (bản chất cùng một bài với Huffman). Lỗi tư duy phổ biến: quên tính chất prefix-free và đề xuất mã `A=0, B=1, C=01` — không giải mã được `01`; và quên rằng heap cho O(k log k), sort một lần rồi hai queue cho O(k) khi tần suất đã sắp.

**Dạng 2 — Encode and Decode Strings (LeetCode 271).** Câu hỏi bề ngoài về string, bản chất về **mã hóa giải mã được duy nhất**: nối chuỗi bằng delimiter ngây thơ chết ngay khi delimiter xuất hiện trong dữ liệu. Lời giải length-prefixing (`4#abcd`) chính là ý tưởng self-describing code; các phương án khác (escape, prefix-free) đều là bài tập nhỏ của information theory. Nói được "delimiter chỉ an toàn khi nó không thể xuất hiện trong payload — vậy hoặc escape, hoặc prefix độ dài" là trúng tim đề.

**Dạng 3 — Câu hỏi entropy trong ML interview.** Rất phổ biến với vị trí AI/ML: "Vì sao dùng cross-entropy thay vì MSE cho phân loại?" (cross-entropy = maximum likelihood cho phân phối categorical; gradient không bị bão hòa như MSE qua sigmoid). "KL divergence có phải distance không?" (không — bất đối xứng, vi phạm bất đẳng thức tam giác). "Entropy của phân phối đều trên 8 lớp?" (log₂8 = 3 bit — trả lời được trong một giây thể hiện đã hiểu chứ không thuộc). "Information gain trong decision tree là gì?" (mức giảm entropy sau khi split — cây quyết định là thuật toán tham lam theo entropy).

**Dạng 4 — System design: "Thiết kế hệ thống thu thập và lưu trữ log".** Câu system design thật, và phần compression là điểm ăn điểm: nén ở đâu (agent hay ingestion — trade-off CPU edge vs bandwidth), codec nào theo tier (lz4 cho hot, zstd level cao cho cold), nén theo block bao lớn (random access vs ratio), và câu trả lời cấp cao nhất: tách template/tham số để hạ entropy trước khi codec chạy. Ứng viên nói được "log có entropy thấp nên mục tiêu 10–20× là hợp lý, còn metric đã delta-encode thì đừng kỳ vọng thế" là ứng viên hiểu bản chất.

**Cách phân tích chung:** gặp bất kỳ bài nén/mã hóa nào, hỏi ba câu theo thứ tự: (1) phân phối dữ liệu lệch ở đâu — cái gì lặp, cái gì đoán được? (2) giới hạn lý thuyết ước chừng bao nhiêu — có đáng làm không? (3) ràng buộc vận hành (random access? streaming? CPU budget?) đẩy ta về điểm nào trên đường trade-off?

## 9. Anti-pattern

**Nén dữ liệu đã nén.** Bật gzip ở nginx cho response ảnh/video, bật compression Kafka cho topic chứa Protobuf đã nén sẵn payload — tốn CPU hai đầu, thêm latency, tỷ lệ ~1.0× hoặc tệ hơn (header codec là overhead thuần). Luôn hỏi: entropy nguồn còn chỗ trống không?

**Mã hóa trước, nén sau.** Pipeline `encrypt → compress` vô hiệu hóa hoàn toàn bước nén vì ciphertext có entropy tối đa theo thiết kế. Thứ tự đúng: `compress → encrypt`. (Chiều sâu hơn: nén-trước-mã-hóa trong ngữ cảnh attacker điều khiển một phần plaintext từng tạo ra lỗ hổng thật — CRIME/BREACH trên TLS: *kích thước* output nén rò rỉ thông tin về nội dung, vì compression ratio chính là hàm của nội dung. Chính vì vậy TLS 1.3 bỏ hẳn compression. Bài học kép: nén là con dao thông tin hai lưỡi.)

**Đánh giá độ mạnh password bằng quy tắc hình thức.** Rule "phải có hoa, thường, số, ký tự đặc biệt" tạo ra `P@ssw0rd1!` entropy thấp nhưng chặn `correct horse battery staple` entropy cao. Chuẩn hiện đại (NIST 800-63B) chuyển sang độ dài + kiểm tra chống danh sách lộ — tức là chuyển từ đo hình thức sang đo entropy của quy trình sinh.

**Tin vào "tỷ lệ nén trung bình" khi lập kế hoạch dung lượng.** Tỷ lệ nén là hàm của entropy dữ liệu, mà entropy thay đổi theo nguồn: cùng một pipeline, log JSON nén 8×, còn trace ID ngẫu nhiên trong log làm tỷ lệ tụt còn 3×. Ước lượng dung lượng phải đo trên phân phối dữ liệu thật của *bạn*, và theo dõi compression ratio như một metric — nó tụt đột ngột thường có nghĩa dữ liệu đổi tính chất (hoặc ai đó nhét blob mã hóa vào topic).

**Dùng KL divergence như khoảng cách đối xứng.** Trong ML/monitoring (so sánh phân phối traffic hôm nay với baseline — drift detection), D(P‖Q) và D(Q‖P) khác nhau và bùng nổ khi Q(x) = 0 tại điểm P(x) > 0. Khi cần đối xứng và bị chặn, dùng Jensen–Shannon divergence. Chọn sai chiều KL trong alert rule là nguồn false positive kinh niên.

## 10. Best Practices

**Nên:**

- Ước lượng trần trước khi tối ưu: chạy `zstd -19` một lần trên mẫu dữ liệu thật để biết entropy thực tế cho phép bao nhiêu — mọi nỗ lực sau đó có baseline để so.
- Mặc định zstd level thấp (3) cho hệ thống mới; hạ xuống lz4 khi profile chỉ ra CPU nén là bottleneck; nâng level cho tier lạnh write-once-read-many.
- Hạ entropy bằng cấu trúc *trước khi* trông cậy codec: columnar thay row, dictionary/RLE, tách template khỏi tham số trong log, sort dữ liệu theo cột lặp nhiều trước khi ghi Parquet (sort làm run dài ra → RLE ăn sâu hơn).
- Đo compression ratio như một metric vận hành và alert khi nó thay đổi đột ngột.
- Với password/token/secret: đo độ mạnh bằng bits of entropy của quy trình sinh (dùng CSPRNG — chương 19), không bằng độ "ngoằn ngoèo" của chuỗi.

**Không nên:**

- Không nén payload nhỏ hơn vài trăm byte, không nén dữ liệu đã nén/mã hóa.
- Không bật compression trong kênh mã hóa khi attacker có thể tiêm một phần nội dung (bài học CRIME/BREACH).
- Không chọn codec theo benchmark của người khác trên dữ liệu của họ — entropy là thuộc tính của dữ liệu *bạn*.
- Không kỳ vọng lossless vượt trần Shannon — nếu một vendor hứa "nén mọi dữ liệu 90%", pigeonhole principle (chương 05) đủ để bác bỏ: không thể ánh xạ 2ⁿ input vào ít hơn 2ⁿ output mà vẫn khả nghịch.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Một nguồn phát ký hiệu với phân phối 0.5/0.25/0.125/0.125 — entropy bằng bao nhiêu, và bộ mã Huffman nào đạt *đúng* entropy đó (khi nào Huffman chạm trần chính xác)?
2. Vì sao `compress → encrypt` đúng còn `encrypt → compress` vô dụng — giải thích bằng entropy của ciphertext?
3. Loss cross-entropy của language model giảm từ 4 bit/token xuống 3 bit/token — diễn giải điều đó theo ngôn ngữ nén dữ liệu như thế nào?

---

*Chương tiếp theo: [21 — Optimization](/series/math-for-engineers/level-4-advanced-mathematics/21-optimization/), nơi câu hỏi đổi từ "giới hạn của thông tin là gì" sang "trong không gian phương án khổng lồ và đầy ràng buộc, làm sao tìm được phương án tốt nhất — và khi nào nên ngừng đòi hỏi tối ưu tuyệt đối".*
