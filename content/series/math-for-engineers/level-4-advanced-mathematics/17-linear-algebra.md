+++
title = "Chương 17 — Linear Algebra: Ngôn ngữ của AI và không gian nhiều chiều"
date = "2026-07-20T09:50:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 4 – Advanced Mathematics
> Yêu cầu trước: Chương 02 (Set Theory, Functions & Relations), Chương 10 (Complexity Analysis), Chương 11 (Probability)

---

## 1. Problem Statement

Người dùng gõ vào ô tìm kiếm của trang thương mại điện tử: "đồ giữ ấm cho bé mùa đông". Kho hàng có sản phẩm tên "Áo khoác lông cừu trẻ em" — không chung một từ nào với query. Full-text search truyền thống (kể cả Elasticsearch với inverted index từ chương 10) trả về **0 kết quả**: nó tìm *từ*, không tìm *nghĩa*. Team thử vá bằng từ điển đồng nghĩa — và nhanh chóng chết chìm: "giữ ấm" ~ "khoác" ~ "len" ~ "nỉ" ~ "lông cừu"..., tổ hợp nổ theo cấp số nhân, tiếng Việt lại còn "bé" ~ "trẻ em" ~ "em bé" ~ "kids". Không ai bảo trì nổi.

Lời giải mà mọi hệ thống semantic search hiện đại dùng nghe như khoa học viễn tưởng nếu nói vào năm 2010: **biến mỗi câu thành một điểm trong không gian ~1000 chiều**, sao cho các câu gần nghĩa nằm gần nhau. "Đồ giữ ấm cho bé" và "Áo khoác lông cừu trẻ em" — hai chuỗi ký tự không giao nhau — trở thành hai vector chỉ lệch nhau vài độ. Tìm kiếm ngữ nghĩa = tìm hàng xóm gần nhất trong không gian đó.

Câu hỏi lập tức nảy ra, và chúng là nội dung của cả chương:

1. "Điểm trong không gian 1000 chiều" nghĩa là gì, và **"gần nhau" đo bằng gì**? (vector, dot product, cosine similarity)
2. Model biến đổi dữ liệu qua từng tầng bằng công cụ nào? (matrix — và vì sao nó là *phép biến đổi*, không phải bảng số)
3. Không gian 1000 chiều quá đắt — **nén xuống còn 100 chiều mà giữ được cấu trúc** bằng cách nào? (eigenvector, PCA)
4. Vì sao mọi thứ trên chạy trên GPU, và vì sao AI engineer bị ám ảnh bởi phép nhân ma trận?

Linear algebra là môn nhiều engineer từng học ở đại học dưới dạng "giải hệ phương trình và tính định thức" rồi quên sạch — vì chưa ai nói cho họ rằng nó là **ngôn ngữ mô tả không gian và biến đổi**, và rằng recommendation của Netflix, ranking của Google, và mọi LLM đều viết bằng ngôn ngữ đó.

## 2. Trực giác

### Vector không phải "mảng số"

Trong code, vector là `[]float64`. Nhưng nếu chỉ nhìn nó như mảng số, toàn bộ hình học biến mất — mà hình học mới là thứ đáng tiền. Một vector là **một điểm (hoặc một mũi tên từ gốc tọa độ) trong không gian**:

```
        y
        │      ● b = (1, 3)
      3 ┤     ╱
        │    ╱        góc θ giữa a và b nhỏ
      2 ┤   ╱    ● a = (3, 2)   → a và b "cùng hướng"
        │  ╱  ╱
      1 ┤ ╱ ╱
        │╱╱          ● c = (2, -1): gần vuông góc với b
        └─┬──┬──┬──── x           → "không liên quan"
          1  2  3
```

Trong 2–3 chiều, mỗi tọa độ là một thuộc tính không gian. Bước nhảy trực giác quan trọng nhất chương: **tọa độ không nhất thiết là không gian vật lý — nó có thể là bất kỳ thuộc tính đo được nào**. Một bộ phim là một điểm trong không gian (độ hành động, độ lãng mạn, độ hài, năm sản xuất...). Một người dùng cũng là một điểm trong *cùng không gian đó*: tọa độ là mức độ họ thích từng thuộc tính. Đột nhiên "gợi ý phim" = "tìm phim gần vị trí của user" — một bài toán hình học.

**Embedding** chính là bước cuối của trực giác này: thay vì người thiết kế chọn thuộc tính bằng tay, một model học ra không gian ~300–3000 chiều nơi *khoảng cách phản ánh ngữ nghĩa*. Không ai biết chiều 217 của không gian embedding "nghĩa là gì" — không cần biết. Điều được đảm bảo (bằng cách model được huấn luyện) là: câu gần nghĩa → vector gần nhau. Embedding là **vector hóa ngữ nghĩa**: nén thứ mờ nhòe như "ý nghĩa" thành thứ đo đếm được như tọa độ.

### Matrix không phải "bảng số"

Sai lầm trực giác thứ hai cần đập bỏ: matrix là bảng Excel. Cách nhìn đúng — và là cách duy nhất khiến mọi công thức trở nên hiển nhiên — matrix là **một phép biến đổi không gian**: một cái máy nhận vector vào, trả vector ra, và làm điều đó một cách "tuyến tính" (mục 3 định nghĩa chặt). Ma trận 2×2 là máy biến đổi mặt phẳng:

```
   Phép xoay 90° ngược kim đồng hồ          Phép scale x2 theo trục x
        ┌         ┐                              ┌       ┐
    R = │  0  -1  │                          S = │ 2   0 │
        │  1   0  │                              │ 0   1 │
        └         ┘                              └       ┘

    ▲ (0,1)                  ▲ (0,1)         hình vuông → chữ nhật
    │     R·(1,0) = (0,1)    │
    │  ●──── (1,0)           │  ●──────────── (2,0)
    └──────►                 └──────────────►
   "xoay cả mặt phẳng"       "kéo giãn cả mặt phẳng"
```

Mỗi cột của matrix trả lời một câu hỏi cụ thể: **vector cơ sở thứ i bay đến đâu?** Cột 1 của R là (0,1) — nghĩa là trục x sau khi xoay chỉ lên trời. Biết số phận của các vector cơ sở là biết số phận của *mọi* vector (vì tính tuyến tính). Đó là toàn bộ bí mật của matrix: nó là **bảng ghi số phận của các trục tọa độ**.

Từ cách nhìn này, các sự thật khô khan trở thành hiển nhiên:

- **Nhân matrix với vector** = áp phép biến đổi lên điểm.
- **Nhân hai matrix** = ghép nối hai phép biến đổi (làm B trước rồi A: A·B). Và lập tức hiểu **vì sao không giao hoán**: "xoay 90° rồi kéo giãn trục x" cho hình khác hẳn "kéo giãn trục x rồi xoay 90°" — thứ tự biến đổi có nghĩa, hệt như thứ tự middleware trong HTTP stack.
- **Một layer neural network** = một phép biến đổi không gian (nhân matrix) + một cú bẻ cong phi tuyến (activation). Deep learning = chuỗi dài các phép biến đổi không gian, học ra được.

### Nếu không có ngôn ngữ này thì sao?

Không có linear algebra, mỗi bài toán trên phải phát minh lại từ đầu bằng vòng lặp và if: so khớp văn bản bằng luật, gợi ý bằng bảng tra thủ công, đồ họa 3D bằng lượng giác chắp vá từng trường hợp. Linear algebra làm cho hàng nghìn bài toán bề ngoài khác nhau — ranking trang web, nén ảnh, khử nhiễu, gợi ý phim, xoay camera trong game — trở thành *cùng vài phép toán*, và vì thế phần cứng (GPU/TPU) chỉ cần tăng tốc đúng vài phép toán đó để tăng tốc tất cả.

## 3. First Principles

### "Tuyến tính" nghĩa là gì, chính xác?

Một phép biến đổi f là **tuyến tính** khi nó tôn trọng hai phép toán của không gian vector:

> f(u + v) = f(u) + f(v)  và  f(c·u) = c·f(u)

Diễn giải hình học: tuyến tính = **lưới tọa độ sau biến đổi vẫn là lưới** — các đường thẳng song song cách đều vẫn song song cách đều, gốc tọa độ đứng yên. Được phép: xoay, kéo giãn, lật, chiếu, trượt nghiêng (shear). Không được phép: uốn cong, dịch chuyển gốc, phóng đại chỗ này ít chỗ kia nhiều.

Hệ quả nền tảng — lý do matrix tồn tại: một phép tuyến tính bị **xác định hoàn toàn bởi ảnh của các vector cơ sở**. Vì mọi v = x·e₁ + y·e₂ nên f(v) = x·f(e₁) + y·f(e₂). Chỉ cần lưu f(e₁), f(e₂), ... — xếp chúng thành các cột, và đó *chính là* matrix của f. Matrix không phải là quy ước ghi chép tùy hứng; nó là lượng thông tin tối thiểu đủ để tái tạo cả phép biến đổi. (So sánh với interface: một phép tuyến tính là implementation, matrix là bản serialize gọn nhất của nó.)

### Dot product: một con số đo "độ cùng hướng"

Dot product của hai vector n chiều:

> a·b = a₁b₁ + a₂b₂ + ... + aₙbₙ

Định nghĩa đại số này trông vô hồn cho đến khi thấy mặt hình học của nó:

> **a·b = |a|·|b|·cos θ**, với θ là góc giữa hai vector

Dẫn xuất trực quan không cần lượng giác nặng: xét trường hợp đơn giản nhất — b nằm dọc trục x, tức b = (|b|, 0). Khi đó a·b = a₁·|b| + a₂·0 = a₁·|b|. Mà a₁ chính là **hình chiếu của a lên trục x** = |a|·cosθ. Vậy a·b = |a|·|b|·cosθ trong trường hợp này. Trường hợp tổng quát chỉ là xoay hệ tọa độ cho b về trục x — và phép xoay không đổi độ dài lẫn góc, cũng không đổi dot product. Bản chất: **dot product = độ dài a chiếu lên hướng b, nhân độ dài b** — nó đo "a có bao nhiêu phần đi theo hướng của b".

Từ đó có bộ ba trạng thái đáng nhớ:

| a·b | θ | Ý nghĩa |
|---|---|---|
| > 0 | < 90° | cùng hướng — "đồng thuận" |
| = 0 | = 90° | trực giao — "không liên quan" |
| < 0 | > 90° | ngược hướng — "đối nghịch" |

### Cosine similarity: vì sao search dùng góc thay vì khoảng cách

Đảo công thức trên:

> cos θ = (a·b) / (|a|·|b|)

Đây là **cosine similarity** — thước đo mặc định của semantic search và recommendation. Vì sao không dùng khoảng cách Euclid |a−b| cho thẳng? Vì với text embedding, **độ dài vector thường nhiễm thông tin không liên quan đến ngữ nghĩa** — độ dài văn bản, tần suất từ, biên độ mà model tình cờ gán. Một tài liệu dài về "cà phê" và một câu ngắn về "cà phê" cho hai vector *cùng hướng* nhưng khác độ lớn: Euclid thấy chúng xa nhau, cosine thấy chúng trùng chủ đề. Cosine **bất biến với scale**: cos(a, 10a) = 1. Ta muốn hỏi "hai văn bản nói về cùng thứ không?", không phải "hai văn bản dài bằng nhau không?" — chọn metric là chọn câu hỏi.

Mẹo production quan trọng suy ra ngay từ công thức: nếu **chuẩn hóa mọi vector về độ dài 1** ngay khi ghi vào hệ thống, cosine similarity thoái hóa thành dot product trần — rẻ hơn (khỏi tính hai căn bậc hai mỗi lần so), và khi đó xếp hạng theo cosine, theo dot product, theo Euclid cho *cùng thứ tự*. Đây là lý do các vector database khuyên (hoặc tự động) normalize embedding.

Và câu "khi nào KHÔNG dùng cosine": khi độ lớn *mang nghĩa*. Vector (số đơn hàng, doanh thu) của khách hàng — khách chi 10 triệu và khách chi 100 nghìn cùng tỷ lệ mua sắm sẽ có cosine = 1, nhưng gộp họ làm một là sai nghiệp vụ. Cosine vứt bỏ độ lớn; chỉ vứt khi độ lớn đáng vứt.

## 4. Mathematical Model

### Matrix multiply: hợp thành biến đổi

Với A kích thước m×n và B kích thước n×p, tích C = A·B có C[i][j] = Σₖ A[i][k]·B[k][j] — tức **C[i][j] = dot product của hàng i (A) với cột j (B)**. Công thức trông tùy tiện cho đến khi hỏi: nó *phải* là gì để A·B biểu diễn "biến đổi B trước, A sau"? Cột j của C phải là ảnh qua A của (cột j của B) — khai triển đúng ra công thức trên. Phép nhân ma trận không được "định nghĩa", nó được **suy ra** từ yêu cầu hợp thành.

Chi phí: O(m·n·p) — với hai matrix vuông n×n là O(n³) phép nhân-cộng. Đây là con quái vật ăn điện của AI: một lần suy luận LLM là hàng nghìn phép nhân matrix cỡ hàng nghìn. (Thuật toán Strassen O(n^2.81) tồn tại nhưng hằng số và cache khiến nó hiếm khi thắng trong thực tế — đúng bài học chương 10.)

### Matrix-vector multiply = n dot product, và bài học cache

A·x (A: n×n, x: n) là n dot product — hàng i của A chấm với x. Đây là phép toán nền của mọi layer neural network, và là nơi lý thuyết chạm phần cứng. Go (như C) lưu matrix 2 chiều theo **row-major**: các phần tử cùng hàng nằm liền kề trong bộ nhớ. Duyệt theo hàng → cache prefetch hoạt động; duyệt theo cột → mỗi bước nhảy xa n×8 byte, gần như mỗi truy cập một cache miss.

Với matrix multiply C = A·B, cùng một công thức có 6 cách xếp 3 vòng lặp, *cùng số phép toán*, khác nhau nhiều lần về tốc độ:

```go
// i-j-k: cách "chép từ định nghĩa" — chậm
// B[k][j] với k chạy: nhảy cột trên B → cache miss liên tục
for i := 0; i < n; i++ {
    for j := 0; j < n; j++ {
        sum := 0.0
        for k := 0; k < n; k++ {
            sum += a[i][k] * b[k][j] // b nhảy dọc cột: tội đồ ở đây
        }
        c[i][j] = sum
    }
}

// i-k-j: đảo hai vòng trong — nhanh hơn nhiều lần
// Cố định a[i][k], quét b[k][j] với j chạy: cả B lẫn C đều đi theo hàng
for i := 0; i < n; i++ {
    for k := 0; k < n; k++ {
        aik := a[i][k]
        for j := 0; j < n; j++ {
            c[i][j] += aik * b[k][j] // mọi truy cập đều tuần tự
        }
    }
}
```

Trên máy x86 hiện đại với n = 1024 (float64), bản i-j-k thường mất cỡ 3–5 giây, bản i-k-j cỡ 0.5–1 giây — **chênh 5–8 lần chỉ nhờ đổi thứ tự hai dòng `for`**, zero thay đổi về Big-O. Thư viện BLAS còn đi xa hơn (blocking để khối con nằm gọn trong L1/L2, SIMD, đa luồng) và nhanh hơn bản naive 50–100 lần. Bài học gắn với chương 10: khi bậc đã kịch trần, **hằng số là chiến trường**, và hằng số trên phần cứng hiện đại đánh vần là c-a-c-h-e.

### Eigenvalue / Eigenvector: hướng bất biến của biến đổi

Trực giác trước công thức. Một phép biến đổi nói chung vừa xoay vừa kéo giãn các vector. Nhưng hầu hết matrix có vài **hướng đặc biệt**: vector nằm trên hướng đó bị biến đổi *chỉ kéo giãn, không đổi hướng* — như trục quay của một vật đang xoay, hay thớ gỗ của một tấm ván bị kéo. Hướng đó là **eigenvector**, hệ số kéo giãn là **eigenvalue** λ:

> A·v = λ·v  (v ≠ 0)

Vì sao engineer cần quan tâm? Vì eigenvector trả lời câu hỏi: **lặp lại biến đổi này nhiều lần thì hệ đi về đâu?** Viết x theo các eigenvector: x = c₁v₁ + c₂v₂ + ...; sau k lần áp A:

> Aᵏx = c₁λ₁ᵏv₁ + c₂λ₂ᵏv₂ + ...

Các λᵏ tăng/tụt theo cấp số nhân, nên thành phần có |λ| lớn nhất **nuốt chửng** mọi thành phần khác: dù xuất phát gần như bất kỳ đâu, Aᵏx tiến về hướng v₁. Eigenvector lớn nhất là "điểm hút" của phép lặp — trạng thái dừng của hệ.

### PageRank: eigenvector trị giá một đế chế

Bài toán của Google năm 1998: xếp hạng trang web sao cho không thể gian lận bằng nhồi từ khóa. Ý tưởng: mô hình hóa "người lướt web ngẫu nhiên" — mỗi bước click ngẫu nhiên một link trên trang hiện tại (thỉnh thoảng nhảy ngẫu nhiên bất kỳ, hệ số damping ~0.85, để không mắc kẹt). Câu hỏi "trang nào quan trọng?" trở thành "**về lâu dài, người lướt ở trang nào thường xuyên nhất?**".

Gọi xₖ là vector phân bố xác suất trên các trang tại bước k, M là ma trận chuyển (M[i][j] = xác suất từ trang j click sang trang i). Thì xₖ₊₁ = M·xₖ — mỗi bước thời gian là một phép nhân matrix. Phân bố dừng x* thỏa:

> M·x* = x*

Tức x* là **eigenvector của M với eigenvalue 1**. PageRank của một trang = tọa độ của nó trong eigenvector đó. Toán học của chuỗi Markov đảm bảo (với damping) eigenvalue 1 là lớn nhất và duy nhất — nên phép lặp hội tụ về đúng một đáp án, không phụ thuộc điểm xuất phát.

### PCA: nén không gian bằng eigenvector

Bài toán: embedding 1536 chiều quá đắt để lưu và search; giảm còn 256 chiều mà mất ít thông tin nhất. Trực giác hình học: đám mây điểm dữ liệu thường "dẹt" — trải rộng theo vài hướng, mỏng dính theo các hướng còn lại (các chiều tương quan nhau). **Hướng đáng giữ là hướng dữ liệu trải rộng nhất** — phương sai lớn nhất — vì đó là hướng phân biệt các điểm với nhau; hướng mỏng gần như hằng số, vứt đi mất ít.

```
      ▲        ·  ·                 v₁: hướng phương sai lớn nhất
      │      · · ·● ╱                   (eigenvector lớn nhất của covariance)
      │    · ·●· ╱· ·
      │   · ·● ╱ ·  ·               v₂ ⊥ v₁: phương sai còn lại
      │  · · ╱ ●· ·                 → chiếu mọi điểm lên v₁:
      │   ·╱ · ·                      2D → 1D, giữ phần lớn "độ khác nhau"
      └────────────────►
```

Kết nối đẹp nhất chương: hướng phương sai lớn nhất **chính là eigenvector lớn nhất của covariance matrix** C (ma trận n×n với C[i][j] = covariance giữa chiều i và j — chương 18). Phác thảo lý do: phương sai của dữ liệu chiếu lên hướng đơn vị v là vᵀCv; cực đại hóa đại lượng này với ràng buộc |v| = 1 đạt được tại eigenvector ứng với λ lớn nhất, và phương sai đạt được đúng bằng λ. PCA = tính C, lấy k eigenvector đầu, chiếu dữ liệu lên chúng. Giảm chiều embedding, nén ảnh, khử nhiễu, trực quan hóa dữ liệu cao chiều xuống 2D — cùng một phép toán.

### Matrix factorization: recommendation là bài toán phân rã

Netflix có ma trận R: hàng = user, cột = phim, R[u][i] = điểm user u chấm phim i — và 99% ô **trống**. Dự đoán ô trống = gợi ý phim. Ý tưởng then chốt: sở thích con người không tùy tiện — nó bị chi phối bởi một số nhỏ **latent factor** (độ hành động, độ hài, độ "nghệ"...). Nếu đúng vậy, R xấp xỉ được bằng tích hai matrix gầy:

> R (m×n) ≈ U (m×k) · Vᵀ (k×n),  k ~ 50–200 ≪ m, n

Hàng u của U = "gu" của user trong không gian k latent factor; cột i của Vᵀ = "chất" của phim trong cùng không gian. Điểm dự đoán = **dot product** của hai vector đó — lại là "độ cùng hướng": user hợp gu phim nào thì vector cùng hướng với phim đó. Huấn luyện = tìm U, V khớp các ô đã biết (gradient descent hoặc ALS — chương 21); các ô trống được điền *tự động* bởi cấu trúc thấp chiều. Không ai định nghĩa các factor — chúng **nổi lên từ dữ liệu**, hệt như chiều của embedding. Thực tế, matrix factorization chính là tổ tiên trực hệ của embedding hiện đại: cùng một ý tưởng "nén thực thể thành vector k chiều sao cho dot product có nghĩa".

## 5. Thuật toán

### Bộ đồ nghề vector trong Go

```go
// Dot product — viên gạch của mọi thứ trong chương này.
func dot(a, b []float64) float64 {
    var s float64
    for i := range a {
        s += a[i] * b[i] // compiler tự vector hóa (SIMD) vòng lặp đơn giản này
    }
    return s
}

func norm(a []float64) float64 { return math.Sqrt(dot(a, a)) }

// Cosine similarity. Trong production: normalize sẵn khi ghi,
// lúc đọc chỉ cần dot() — tiết kiệm 2 lần norm mỗi phép so sánh.
func cosine(a, b []float64) float64 {
    return dot(a, b) / (norm(a) * norm(b))
}

// Matrix-vector multiply: y = A·x, tức n dot product.
// A lưu phẳng row-major (a[i*n+j]) — một allocation, cache-friendly,
// nhanh hơn [][]float64 vốn rải các hàng khắp heap.
func matVec(a []float64, x, y []float64, n int) {
    for i := 0; i < n; i++ {
        y[i] = dot(a[i*n:(i+1)*n], x) // hàng i nằm liên tục trong bộ nhớ
    }
}
```

Chi tiết `a[i*n+j]` thay cho `[][]float64` không phải là micro-optimization tùy hứng: slice của slice khiến mỗi hàng là một allocation riêng nằm rải rác trên heap — mất tính tuần tự mà mục 4 vừa chứng minh là quyết định hiệu năng. Mọi thư viện tính toán (gonum, NumPy, BLAS) đều lưu phẳng.

### Power iteration: tìm eigenvector lớn nhất bằng 15 dòng

Từ trực giác "Aᵏx tiến về hướng eigenvector lớn nhất" ở mục 4, thuật toán tự viết ra: nhân lặp đi lặp lại và chuẩn hóa (để số không nổ tung):

```go
// powerIteration tìm eigenvector ứng với |eigenvalue| lớn nhất của A (n×n).
// Đây chính là bộ máy bên trong PageRank.
func powerIteration(a []float64, n, iters int) []float64 {
    x := make([]float64, n)
    for i := range x {
        x[i] = 1 / float64(n) // xuất phát bất kỳ, miễn không trực giao với v₁
    }
    y := make([]float64, n)
    for k := 0; k < iters; k++ {
        matVec(a, x, y, n)         // y = A·x: thành phần theo v₁ được khuếch đại
        s := norm(y)
        for i := range y {
            x[i] = y[i] / s        // chuẩn hóa: chỉ giữ HƯỚNG, vứt độ lớn
        }
        // Sản phẩm phụ: s hội tụ về |λ₁| — có luôn eigenvalue
    }
    return x
}
```

Mỗi vòng, tỷ trọng của v₁ trong x tăng theo tỷ số |λ₁/λ₂| — hội tụ tuyến tính với tốc độ phụ thuộc **khoảng cách giữa hai eigenvalue lớn nhất** (spectral gap). Với PageRank, damping 0.85 đảm bảo gap đủ rộng để ~50–100 vòng lặp là đủ cho cả web graph — mỗi vòng chỉ là một phép nhân matrix thưa với vector, phân tán được trên MapReduce. Google xếp hạng cả Internet không phải bằng thuật toán bí hiểm, mà bằng phép nhân matrix-vector lặp lại — cái bạn vừa viết trong 15 dòng Go.

Khi nào KHÔNG dùng power iteration: cần *nhiều* eigenvector (PCA lấy top-k → dùng Lanczos/SVD của thư viện), hoặc λ₁ ≈ λ₂ (hội tụ rùa bò), hoặc matrix nhỏ (gọi thẳng gonum cho lành). Giá trị của power iteration với engineer là **hiểu bản chất** — nó là mô hình tư duy đúng về eigenvector: điểm hút của phép lặp.

### Từ đây đến hệ thống thật

Semantic search vận hành đúng pipeline các phép toán trên: (1) embedding model biến query thành vector q — chuỗi matVec + phi tuyến; (2) so q với hàng triệu vector tài liệu bằng dot product; (3) lấy top-k bằng heap (chương 16). Bản brute-force O(n·d) mỗi query chịu được đến ~10⁵–10⁶ vector; vượt ngưỡng đó cần approximate nearest neighbor (HNSW — đồ thị "small world" nhiều tầng cho search ~O(log n), đổi một chút recall lấy nhiều bậc tốc độ; chi tiết thuộc chương 24). Điểm cần thấy: **toàn bộ trí thông minh nằm ở không gian embedding; phần "search" chỉ là hình học + cấu trúc dữ liệu.**

## 6. Trade-off

**Cosine vs Euclidean vs dot product trần.** Cosine bất biến độ dài — đúng cho text/semantic; Euclid giữ độ lớn — đúng khi tọa độ là số đo vật lý cùng đơn vị; dot product trần (không chuẩn hóa) cho phép độ dài vector mã hóa "độ phổ biến" (item hot có norm lớn → được ưu tiên) — một số hệ recommendation *cố tình* dùng nó. Chọn metric = chọn cái gì được phép ảnh hưởng kết quả; đổi metric sau khi đã đánh index là phải đánh lại toàn bộ.

**Exact vs approximate.** Brute-force dot product: đúng 100%, O(n·d), song song hóa tầm thường, zero cấu trúc phụ. HNSW: nhanh hơn hàng trăm lần, recall ~95–99%, nhưng tốn RAM cho đồ thị, build index chậm, và delete/update phiền phức. Dưới ~10⁵ vector, brute-force thường là lựa chọn *đúng* — đừng trả phí ANN khi n chưa đáng.

**Số chiều: biểu đạt vs chi phí.** Chiều cao hơn → nhồi được nhiều sắc thái ngữ nghĩa, nhưng chi phí lưu + search tuyến tính theo d, và **curse of dimensionality**: chiều càng cao, khoảng cách giữa mọi cặp điểm càng xích lại gần nhau, contrast giữa "gần" và "xa" nhạt dần (liên hệ chương 22). PCA/quantization giảm chiều là dao hai lưỡi: rẻ hơn nhiều, mất một ít recall — đo trên dữ liệu của bạn, đừng tin số của người khác.

**Tự viết vs BLAS.** Vòng lặp Go thuần: dễ đọc, dễ debug, đủ nhanh cho vector lẻ tẻ. gonum/BLAS: nhanh hơn 10–100 lần cho matrix lớn nhờ blocking + SIMD, đổi lấy dependency (cgo với OpenBLAS) và stack trace khó hiểu. Ranh giới thực dụng: phép toán trên *một* vector d ≤ vài nghìn — tự viết vô tư; nhân matrix từ nghìn×nghìn trở lên hoặc nằm trong vòng nóng — dùng thư viện, đừng anh hùng.

**Precompute vs compute-on-read.** Normalize embedding lúc ghi (một lần) so với lúc đọc (mỗi query × mỗi ứng viên). Materialize ma trận U·Vᵀ cho top user? Cùng họ trade-off với index của chương 16: trả trước ở đường ghi để đường đọc rẻ — đáng khi read/write ratio cao.

## 7. Production Applications

**Vector database — pgvector, Milvus, Qdrant.** `CREATE EXTENSION vector` biến PostgreSQL thành semantic search engine: cột `vector(1536)`, toán tử `<=>` là cosine distance, index HNSW/IVFFlat cho ANN. Toàn bộ giá trị cột này nằm trong mục 3–4 của chương: distance là dot product + chuẩn hóa, index là cấu trúc đồ thị trên không gian mà *metric quyết định hình dạng*. RAG — kiến trúc LLM phổ biến nhất hiện nay — về mặt hạ tầng chỉ là: embed tài liệu, lưu vector, cosine top-k, nhét vào prompt.

**Netflix / Spotify recommendation.** Netflix Prize (2006–2009) đưa matrix factorization thành chuẩn công nghiệp: R ≈ U·Vᵀ như mục 4, huấn luyện bằng ALS trên Spark. Spotify dùng biến thể cho implicit feedback (nghe/bỏ qua thay vì chấm sao). Ngày nay các model phức tạp hơn (two-tower network) nhưng giao diện cuối vẫn hệt: **user vector chấm item vector, lấy top-k** — nghĩa là hạ tầng serving của recommendation và của semantic search là *cùng một hệ thống*: ANN trên không gian vector.

**Neural network và câu trả lời "vì sao GPU".** Một layer fully-connected là y = σ(W·x + b): nhân matrix chiếm >90% FLOP của cả forward lẫn backward. GPU về bản chất là **phần cứng nhân ma trận**: hàng chục nghìn core làm cùng một phép nhân-cộng trên dữ liệu khác nhau; tensor core của NVIDIA còn chuyên hơn — mỗi chu kỳ nhân một khối matrix 4×4. "AI bùng nổ nhờ GPU" dịch sát nghĩa: AI bùng nổ nhờ phần cứng làm linear algebra rẻ đi ba bậc. Ai hiểu mục 4 (cache, memory layout) hiểu luôn vì sao tối ưu inference xoay quanh memory bandwidth chứ không phải FLOP.

**Graphics pipeline.** Mỗi khung hình game, mọi đỉnh của mọi model 3D đi qua chuỗi biến đổi: model→world→camera→projection — mỗi mũi tên là một matrix 4×4, cả chuỗi được **nhân gộp trước thành một matrix duy nhất** (hợp thành biến đổi — mục 4) rồi áp cho hàng triệu đỉnh. GPU sinh ra năm 1999 để làm đúng việc này; AI mượn lại sau — không phải trùng hợp, mà vì cùng ngôn ngữ toán.

**BLAS/LAPACK — thư viện lâu đài của ngành.** Fortran interface từ 1979, được tối ưu đến mức viết tay assembly cho từng đời CPU (OpenBLAS, Intel MKL) và là backend của NumPy, PyTorch, R, MATLAB, gonum. Ba cấp (vector-vector, matrix-vector, matrix-matrix) phản ánh đúng thang cache-intensity ở mục 4. Bài học kiến trúc: chuẩn hóa *phép toán nguyên thủy* đúng cách một lần, cả ngành hưởng 45 năm.

## 8. Interview

**Rotate Image (LeetCode 48)** — xoay ma trận n×n tại chỗ 90° theo chiều kim đồng hồ. Lời giải phổ biến: "transpose rồi reverse từng hàng" — thường được dạy như trick thuộc lòng. Linear algebra cho lời giải *có lý do*: transpose là phép **lật qua đường chéo chính**; reverse mỗi hàng là phép **lật qua trục dọc giữa**; và định lý hình học phẳng nói *hợp thành hai phép lật là một phép xoay, góc xoay = 2× góc giữa hai trục lật*. Hai trục (chéo chính, dọc giữa) lệch nhau 45° → xoay 90°. Kiểm chứng bằng tọa độ: transpose đưa (i,j)→(j,i), reverse hàng đưa (j,i)→(j, n−1−i); hợp lại (i,j)→(j, n−1−i) — đúng công thức xoay 90° CW của lưới. Người hiểu điều này tự trả lời được mọi biến thể trong 30 giây: xoay ngược chiều = transpose + reverse *cột*; xoay 180° = hai lần 90° = reverse cả hàng lẫn cột. Không còn gì để thuộc.

**Spiral Matrix (54)** — thuần kỹ năng quản lý biên (4 con trỏ top/bottom/left/right thu hẹp dần — lại là invariant, chương 03). Giá trị phỏng vấn của nó là code sạch dưới áp lực; giá trị "toán" là nhắc bạn phân biệt: đây là bài *duyệt* bảng số, còn Rotate Image là bài *biến đổi* — đúng ranh giới "bảng số vs phép biến đổi" của mục 2.

**Matrix Chain Multiplication** — nhân chuỗi A₁A₂...Aₙ theo thứ tự kết hợp nào để ít phép nhân nhất? Tính kết hợp (associativity) cho quyền chọn chỗ đặt ngoặc, và lựa chọn chênh nhau khủng khiếp: A(10×1000)·B(1000×5)·C(5×5000) — tính (AB)C tốn 10·1000·5 + 10·5·5000 = 300K phép nhân; A(BC) tốn 1000·5·5000 + 10·1000·5000 = 75M — **chênh 250 lần** chỉ do chỗ đặt ngoặc. Bài toán tối ưu là DP khoảng kinh điển O(n³) (chương 14). Trong thực tế đây là việc query optimizer làm với thứ tự join, và các framework tensor làm khi hợp nhất chuỗi phép biến đổi.

**Câu hỏi hệ thống: "Thiết kế semantic search cho 10 triệu tài liệu"** — bài linear algebra trá hình, và là nơi chương này thành lợi thế cạnh tranh. Khung trả lời: (1) chọn embedding model, số chiều — trade-off mục 6; (2) chuẩn hóa vector lúc ghi để cosine → dot product; (3) dưới 10⁵ dùng brute-force, trên đó HNSW/pgvector — nêu trade-off recall/RAM/update; (4) top-k bằng heap; (5) đo bằng recall@k so với brute-force làm ground truth. Ứng viên nói được *vì sao cosine* và *vì sao phải normalize* nổi bật ngay khỏi đám đông chỉ biết niệm chú "dùng vector DB".

**Lỗi tư duy thường gặp:** học thuộc "transpose + reverse" mà không biết vì sao (bị hỏi xoay ngược chiều là gãy); tin rằng phải "giỏi giải tích" mới đụng được AI — trong khi 90% là linear algebra ở mức chương này; nhầm cosine similarity (lớn = gần) với cosine distance (= 1 − similarity, nhỏ = gần) — bug đảo ngược ranking có thật trong production; và so sánh embedding của *hai model khác nhau* — hai không gian tọa độ khác nhau, khoảng cách giữa chúng vô nghĩa.

## 9. Anti-pattern

**Nhân ma trận lớn bằng vòng lặp ngây thơ.** Bản i-j-k tự viết chậm hơn BLAS 50–100 lần — trong vòng nóng, đó là khác biệt giữa 1 GPU và 100 GPU tiền điện. Tự viết để học; production dùng gonum/BLAS, hoặc tốt hơn: đặt câu hỏi vì sao phép nhân đó không nằm trong model serving framework vốn đã tối ưu sẵn.

**Đo khoảng cách Euclid trên embedding chưa chuẩn hóa** rồi kết luận "semantic search không hoạt động". Norm của embedding nhiễm độ dài văn bản; hai đoạn cùng chủ đề khác độ dài bị đẩy ra xa. Triệu chứng kinh điển: tài liệu dài luôn/không bao giờ xuất hiện trong kết quả. Chữa: normalize, hoặc dùng cosine — và nhất quán *cùng một metric* từ index đến query.

**Trộn embedding từ hai model (hoặc hai version của một model).** Nâng cấp model embedding mà chỉ re-embed tài liệu mới → kho vector hai hệ tọa độ trộn lẫn, so sánh chéo vô nghĩa, chất lượng search suy giảm âm thầm không có error log nào. Re-embed toàn bộ hoặc lưu version vào metadata và tách index.

**Coi matrix là bảng và mô phỏng biến đổi bằng rừng if.** Code xử lý hình học viết bằng `if quadrant == 1 {...}` cho từng trường hợp xoay/lật — nở tổ hợp, đầy bug góc cạnh. Một matrix 2×2 (hoặc 4×4 đồng nhất) biểu diễn *mọi* biến đổi tuyến tính và hợp thành bằng phép nhân — ít code hơn, đúng theo cấu trúc thay vì đúng theo từng case.

**PCA/giảm chiều mà không kiểm tra phương sai giữ lại.** Cắt 1536 → 128 chiều vì "cho rẻ" mà không nhìn phổ eigenvalue: nếu 128 chiều đầu chỉ giữ 70% phương sai, bạn đã âm thầm vứt 30% "độ khác nhau" của dữ liệu. Luôn in tỷ lệ phương sai tích lũy và đo recall trước/sau trên tập query thật.

**Tối ưu FLOP khi nghẽn ở memory.** Đếm phép nhân rồi kết luận "còn headroom" trong khi matVec lớn là bài toán **memory-bound**: mỗi phần tử matrix chỉ được dùng một lần, băng thông RAM mới là trần. Đúng bài học pprof của chương 10: đo cái đang nghẽn, không phải cái dễ đếm.

## 10. Best Practices

**Nên:**

- Nghĩ về vector như điểm/hướng có hình học, matrix như phép biến đổi — mọi công thức của chương suy lại được từ hai hình ảnh này khi quên.
- Chuẩn hóa embedding ngay khi ghi (norm = 1), lưu metadata: model, version, số chiều, metric. Coi bộ ba (model, metric, index) là một contract — đổi một phần là re-index.
- Lưu matrix phẳng row-major trong Go; sắp vòng lặp cho truy cập tuần tự; vượt cỡ nghìn×nghìn trong vòng nóng thì chuyển gonum/BLAS.
- Bắt đầu vector search bằng brute-force làm baseline + ground truth; chỉ nâng cấp ANN khi p99 latency bắt buộc, và luôn báo cáo recall@k so với baseline.
- Khi gặp hệ "lặp một phép biến đổi đến ổn định" (ranking, lan truyền ảnh hưởng, Markov chain), nhận diện ngay: trạng thái dừng = eigenvector — có cả kho lý thuyết và thuật toán dùng sẵn.

**Không nên:**

- Không dùng cosine khi độ lớn vector mang nghĩa nghiệp vụ (chi tiêu, số lượng) — và ngược lại, không dùng Euclid trên embedding văn bản thô.
- Không tự cài PCA/SVD/eigendecomposition cho production — độ ổn định số (numerical stability) là bãi mìn; dùng gonum/LAPACK.
- Không đánh giá chất lượng semantic search bằng cảm quan vài query — dựng tập đánh giá nhỏ có ground truth trước khi chỉnh bất kỳ tham số nào.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Vì sao cosine similarity phù hợp với text embedding hơn khoảng cách Euclid, và khi nào điều ngược lại đúng?
2. Không nhìn code, giải thích vì sao đổi thứ tự vòng lặp từ i-j-k sang i-k-j tăng tốc nhân ma trận nhiều lần dù số phép toán không đổi.
3. PageRank giải bài toán gì, và vì sao đáp án của nó là một eigenvector? Phép lặp nào tìm ra eigenvector đó và mỗi vòng lặp tốn gì?

---

*Chương tiếp theo: [18 — Statistics](/series/math-for-engineers/level-4-advanced-mathematics/18-statistics/), nơi các vector dữ liệu của chương này được nhìn bằng con mắt khác: không hỏi "điểm này gần điểm kia không?" mà hỏi "đám mây điểm này nói gì về tổng thể?" — percentile cho monitoring, confidence interval cho A/B testing, và lý do p99 quan trọng hơn trung bình.*
