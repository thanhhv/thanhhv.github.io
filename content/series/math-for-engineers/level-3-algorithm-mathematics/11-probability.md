+++
title = "Chương 11 — Probability: Đánh cược có tính toán"
date = "2026-07-20T08:50:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 3 – Algorithm Mathematics
> Yêu cầu trước: Chương 05 (Counting & Combinatorics), Chương 10 (Complexity Analysis)

---

## 1. Problem Statement

Ngày 24 tháng 12 năm 2012, Netflix sập đúng đêm Giáng sinh. Nguyên nhân gốc nằm ở AWS Elastic Load Balancer, nhưng điều biến một sự cố cục bộ thành sự cố toàn vùng là thứ khác: khi dịch vụ chập chờn, **hàng triệu client retry cùng một lúc, theo cùng một lịch**. Mỗi client đều làm điều "đúng đắn" — thử lại sau 1 giây, rồi 2 giây, rồi 4 giây. Nhưng vì tất cả thất bại cùng thời điểm, tất cả cũng quay lại gõ cửa cùng thời điểm, như một đội quân diễu hành làm sập cây cầu bằng nhịp bước đồng bộ. Backend vừa ngóc đầu dậy lại bị đạp xuống. Giải pháp mà mọi cloud provider ngày nay khuyến cáo nghe như một trò đùa: **thêm ngẫu nhiên vào thời gian chờ**. Sự hỗn loạn có chủ đích cứu hệ thống mà kỷ luật tuyệt đối đã giết chết.

Đây không phải trường hợp cá biệt. Nhìn vào bất kỳ hệ thống lớn nào, bạn sẽ thấy xác suất ở khắp nơi, thường ở đúng những chỗ quan trọng nhất:

- Hash table cam kết O(1) — nhưng chỉ là O(1) **kỳ vọng**, dựa trên giả định phân bố ngẫu nhiên.
- Quicksort thống trị các thư viện chuẩn nhờ Θ(n log n) **average case**, dù worst case là O(n²).
- Load balancer hiện đại chọn server **ngẫu nhiên** (có tính toán) thay vì theo dõi chính xác tải của từng máy.
- TLS, UUID, hash seed của Go map — tất cả đứng trên nền tảng "kẻ tấn công không thể đoán được".

Vì sao? Vì **tính tất định (determinism) không scale**. Muốn biết chính xác server nào đang ít tải nhất, bạn phải hỏi tất cả server — chi phí điều phối đó tăng theo kích thước hệ thống và tự nó trở thành bottleneck. Muốn quicksort *chắc chắn* nhanh trên mọi input, bạn phải trả chi phí chọn pivot hoàn hảo — đắt hơn cả lợi ích thu được. Ở quy mô lớn, tri thức đầy đủ là xa xỉ phẩm; hệ thống buộc phải hành động dưới sự bất định.

Bài toán của chương này:

**Làm sao ra quyết định đúng — và chứng minh được nó đúng — khi không biết chắc điều gì sẽ xảy ra?**

Xác suất chính là bộ công cụ toán học cho việc đó: nó không loại bỏ sự bất định, nó **định lượng** sự bất định để ta có thể tính toán với nó như tính toán với số.

## 2. Trực giác

### Trực giác con người tệ với xác suất đến mức nào

Trước khi xây công cụ, cần thấy rõ vì sao ta cần nó. Hãy làm một bài kiểm tra mà phần lớn bác sĩ — những người ra quyết định dựa trên xét nghiệm hằng ngày — trả lời sai trong các khảo sát thực tế:

> Một bệnh hiếm có tỷ lệ mắc 1/1000. Xét nghiệm phát hiện đúng 99% người bệnh (độ nhạy 99%), và báo nhầm dương tính cho 1% người khỏe. Một người ngẫu nhiên xét nghiệm dương tính. Xác suất người đó thực sự mắc bệnh là bao nhiêu?

Trực giác gào lên: "99%! Xét nghiệm chính xác 99% mà!". Hãy đếm bằng số cụ thể trên 100.000 người:

```
100.000 người
├── 100 người bệnh (1/1000)
│   ├── 99 dương tính  ✓ (true positive)
│   └── 1 âm tính      (test bỏ sót)
└── 99.900 người khỏe
    ├── 999 dương tính ✗ (false positive — 1% của 99.900)
    └── 98.901 âm tính
```

Tổng số người dương tính: 99 + 999 = 1.098. Trong đó chỉ 99 người thực sự bệnh:

> P(bệnh | dương tính) = 99 / 1.098 ≈ **9%**

Không phải 99%. Chín phần trăm. Xét nghiệm "chính xác 99%" cho kết quả dương tính mà 10 lần thì 9 lần là báo động giả. Lý do: người khỏe **đông hơn người bệnh 999 lần**, nên dù tỷ lệ báo nhầm chỉ 1%, đội quân false positive vẫn áp đảo. Trực giác bỏ quên tỷ lệ nền (base rate) — lỗi này có tên riêng: **base rate fallacy**.

### Cùng một lỗi, mặc đồng phục kỹ sư

Thay "bệnh hiếm" bằng "sự cố production" và bạn có bài toán alerting quen thuộc: hệ thống của bạn có sự cố thật 1 lần trong 1000 khoảng thời gian giám sát. Alert rule của bạn bắt được 99% sự cố, và báo nhầm 1% thời gian bình thường. Khi PagerDuty kêu lúc 3 giờ sáng, xác suất có sự cố thật chỉ khoảng 9% — và đó chính là lý do toán học khiến on-call engineer bị **alert fatigue** rồi bắt đầu bỏ qua alert. Vấn đề không nằm ở ý thức của engineer; nó nằm ở số học. Muốn alert đáng tin, giảm false positive rate quan trọng hơn nhiều so với tăng độ nhạy — một kết luận phản trực giác mà chỉ cần một phép chia để chứng minh.

### Điều xác suất thực sự là

Vậy "xác suất" là gì mà tính được như trên? Trực giác làm việc tốt nhất: xác suất là **tỷ lệ đếm được trên một tổng thể tưởng tượng**. P(A) = 9% nghĩa là: nếu tình huống này lặp lại 100.000 lần, khoảng 9.000 lần A xảy ra. Toàn bộ phép tính ở trên không có gì ngoài đếm — chia nhóm, đếm từng nhóm, lấy tỷ lệ. Xác suất là môn đếm của chương 05, thêm một phép chia.

Điểm mạnh chết người của cách nhìn này: mọi công thức xác suất trong chương đều có thể kiểm chứng lại bằng cách "vẽ cây 100.000 người" như trên. Khi nghi ngờ một kết quả xác suất, hãy đếm. Khi hai trực giác đánh nhau, hãy đếm.

## 3. First Principles

### Sample space và biến cố — chọn đúng vũ trụ

Mọi bài toán xác suất bắt đầu bằng việc trả lời: **những kết cục nào có thể xảy ra?** Tập tất cả kết cục đó là **sample space** Ω. Một **biến cố (event)** là một tập con của Ω — chính là tập hợp của chương 02.

- Tung xúc xắc: Ω = {1,2,3,4,5,6}; biến cố "chẵn" = {2,4,6}.
- Hash 2 key vào bảng m slot: Ω = mọi cặp (slot₁, slot₂), tức m² kết cục; biến cố "collision" = {(i,i) : i = 1..m}.
- Request đến trong 1 giây: Ω = {0, 1, 2, 3, ...} — vô hạn đếm được.

Hàm xác suất P gán cho mỗi biến cố một số, tuân ba tiên đề: P(A) ≥ 0; P(Ω) = 1; nếu A, B rời nhau thì P(A ∪ B) = P(A) + P(B). Chỉ ba luật đó — mọi thứ còn lại của chương này đều suy ra được từ chúng, cộng với định nghĩa xác suất có điều kiện.

Phần lớn lỗi xác suất trong thực tế không phải lỗi tính toán mà là **lỗi chọn sai Ω** — tính xác suất trên vũ trụ không khớp với câu hỏi. Bài toán xét nghiệm ở trên chính là vậy: "99% chính xác" là xác suất trên vũ trụ *người đã biết bệnh/khỏe*, còn câu hỏi thực tế nằm trên vũ trụ *người đã biết kết quả xét nghiệm*. Hai vũ trụ khác nhau, hai con số khác nhau.

### Conditional probability — cập nhật vũ trụ khi có thông tin

Khi biết B đã xảy ra, vũ trụ co lại từ Ω xuống B. Xác suất của A trong vũ trụ mới đó:

> P(A|B) = P(A ∩ B) / P(B)

Đọc bằng ngôn ngữ đếm: trong số các kết cục thuộc B, bao nhiêu phần cũng thuộc A. Trong bài xét nghiệm: B = "dương tính" (1.098 người), A ∩ B = "bệnh và dương tính" (99 người), tỷ lệ 9%. Định nghĩa này không phải định lý — nó là *định nghĩa* của việc "cập nhật niềm tin khi có bằng chứng", và nó chính xác là thao tác một query `WHERE` rồi tính tỷ lệ trong SQL.

### Independence — khái niệm bị lạm dụng nhất

A và B **độc lập** khi biết B xảy ra không thay đổi gì về A: P(A|B) = P(A), tương đương P(A ∩ B) = P(A)·P(B). Độc lập là siêu năng lực tính toán — nó cho phép nhân xác suất — nhưng cũng là **giả định nguy hiểm nhất trong kỹ nghệ**.

Ví dụ kinh điển: "mỗi ổ đĩa hỏng với xác suất 1%/năm, ta có 2 bản sao, vậy xác suất mất dữ liệu là 0.01 × 0.01 = 1/10.000". Phép nhân đó chỉ đúng nếu hai ổ hỏng **độc lập**. Nhưng hai ổ cùng lô sản xuất, cùng rack, cùng nguồn điện, cùng bị một đợt nắng nóng — chúng hỏng *cùng nhau* thường xuyên hơn nhiều so với mô hình độc lập dự đoán. Sự cố mất dữ liệu thực tế hầu như luôn là **correlated failure**: cả một rack mất điện, một bug firmware giết cả lô đĩa, một engineer xóa nhầm cả ba replica bằng một câu lệnh. Câu chuyện retry đêm Giáng sinh ở đầu chương cũng là correlated behavior: các client không độc lập vì chúng cùng phản ứng với một sự kiện.

Quy tắc sống còn: **độc lập phải được thiết kế ra hoặc chứng minh, không được giả định.** Availability zone của AWS tồn tại chính là để *sản xuất* sự độc lập — nguồn điện riêng, mạng riêng, tòa nhà riêng — vì độc lập không tự nhiên mà có.

### Nếu không có xác suất thì sao?

Không có ngôn ngữ xác suất, kỹ sư chỉ còn hai chế độ tư duy: "chắc chắn đúng" và "hên xui". Chế độ một dẫn đến các thiết kế over-engineered không scale (đồng bộ hóa mọi thứ, kiểm tra mọi thứ); chế độ hai dẫn đến các quyết định không thể phân tích hậu nghiệm ("retry 3 lần vì... 3 là số đẹp"). Xác suất mở ra chế độ thứ ba: **đánh cược có tính toán** — biết chính xác mình đang cược gì, cửa thắng bao nhiêu, và thua thì mất gì.

## 4. Mathematical Model

### Bayes' Theorem — đảo chiều điều kiện trong ba dòng

Điều ta thường *đo được* là P(bằng chứng | nguyên nhân): độ nhạy của xét nghiệm, tỷ lệ email spam chứa từ "FREE". Điều ta *cần biết* là chiều ngược lại: P(nguyên nhân | bằng chứng). Bayes là cây cầu, và nó suy ra từ định nghĩa conditional probability trong ba dòng:

> P(A|B)·P(B) = P(A ∩ B)            (định nghĩa, nhân chéo)
> P(B|A)·P(A) = P(A ∩ B)            (định nghĩa, viết cho chiều kia)
> ⟹ **P(A|B) = P(B|A) · P(A) / P(B)**

Từng thành phần có tên riêng vì mỗi cái trả lời một câu hỏi khác nhau:

| Thành phần | Tên | Ý nghĩa |
|---|---|---|
| P(A) | prior | niềm tin *trước khi* thấy bằng chứng (base rate — thứ trực giác hay quên) |
| P(B\|A) | likelihood | nếu A đúng, bằng chứng B dễ xuất hiện cỡ nào (thứ đo được từ dữ liệu) |
| P(A\|B) | posterior | niềm tin *sau khi* thấy bằng chứng — thứ ta cần |
| P(B) | evidence | tần suất chung của bằng chứng, đóng vai trò chuẩn hóa |

**Spam filter** là Bayes chạy bằng cơm: từ kho email đã gắn nhãn, đếm được P(chứa "FREE" | spam) = 0.4 và P(chứa "FREE" | ham) = 0.01; biết prior P(spam) = 0.3. Email mới chứa "FREE":

> P(spam | "FREE") = 0.4 × 0.3 / (0.4 × 0.3 + 0.01 × 0.7) = 0.12 / 0.127 ≈ **94.5%**

Naive Bayes classifier mở rộng ra nhiều từ bằng cách *giả định các từ độc lập với nhau khi biết nhãn* — giả định sai rành rành ("FREE" và "VIAGRA" rõ ràng tương quan), nhưng sai theo hướng ít gây hại cho việc *xếp hạng*, nên nó vẫn hoạt động tốt suốt hai thập kỷ. Đây là bài học tinh tế: mô hình sai vẫn hữu ích, miễn là bạn biết nó sai ở đâu và điều đó ảnh hưởng gì đến quyết định cuối cùng.

Bayes cũng chính là khung tư duy debug đúng đắn: "test này fail" là bằng chứng B; các nguyên nhân khả dĩ A₁, A₂... có prior khác nhau (code mới sửa có prior cao hơn thư viện chuẩn của Go); nguyên nhân đáng điều tra nhất là nguyên nhân có posterior cao nhất, không phải nguyên nhân có likelihood cao nhất. "Chắc là bug của compiler" có likelihood giải thích được hiện tượng, nhưng prior của nó nhỏ đến mức posterior gần như luôn thua "bug trong code mình vừa viết".

### Random variable và Expected Value

**Biến ngẫu nhiên (random variable)** X là một hàm gán mỗi kết cục trong Ω một con số: số lần retry, độ dài chain trong hash table, thời gian phản hồi. Nó biến "kết cục" thành "đại lượng đo được" — bước chuyển từ định tính sang định lượng.

**Kỳ vọng (expected value)** là trung bình có trọng số theo xác suất:

> E[X] = Σ x · P(X = x)

E[X] trả lời câu hỏi thực dụng nhất: *nếu chuyện này lặp lại nhiều lần, trung bình mỗi lần tốn bao nhiêu?* Với hệ thống phục vụ hàng triệu request, "trung bình mỗi lần" nhân với số lần chính là hóa đơn tổng — kỳ vọng là đơn vị tiền tệ của phân tích hệ thống.

### Linearity of Expectation — công cụ mạnh nhất chương này

> **E[X + Y] = E[X] + E[Y] — luôn đúng, kể cả khi X và Y phụ thuộc nhau chặt chẽ.**

Hãy dừng lại ở vế sau. Xác suất của giao P(A ∩ B) cần độc lập mới tách được thành tích; phương sai của tổng cần độc lập mới tách được thành tổng. Nhưng kỳ vọng của tổng **luôn** tách được, vô điều kiện. Đây là kẽ hở hào phóng hiếm hoi mà toán học ban tặng, và giới phân tích thuật toán khai thác nó đến tận xương.

Công thức phối hợp: với **biến chỉ thị (indicator variable)** Xᵢ = 1 nếu sự kiện i xảy ra, 0 nếu không, ta có E[Xᵢ] = P(sự kiện i xảy ra). Chiến lược chuẩn: *đếm tổng = tổng các chỉ thị → kỳ vọng của tổng = tổng các xác suất*. Không cần biết các sự kiện tương tác với nhau ra sao.

**Ứng dụng 1 — số collision kỳ vọng trong hash table.** Ném n key vào m slot ngẫu nhiên đều. Với mỗi cặp key (i, j), đặt X_{ij} = 1 nếu chúng đụng nhau; P(X_{ij} = 1) = 1/m. Số cặp là C(n,2) (chương 05), nên:

> E[số collision] = C(n,2) · (1/m) = n(n−1)/(2m)

Ba dòng, không cần quan tâm việc "key i đụng key j" và "key j đụng key k" có phụ thuộc nhau không (có, chúng phụ thuộc!). Kết quả này là nền của toàn bộ phân tích load factor ở chương 12.

**Ứng dụng 2 — quicksort average case Θ(n log n), mức trực giác.** Gọi các phần tử theo thứ tự đã sort là z₁ < z₂ < ... < zₙ. Nhận xét then chốt: zᵢ và zⱼ được so sánh với nhau **khi và chỉ khi** một trong hai được chọn làm pivot trước tất cả các phần tử nằm giữa chúng — vì nếu pivot rơi vào giữa, hai phần tử bị tách về hai phía và vĩnh viễn không gặp nhau. Trong khối {zᵢ, ..., zⱼ} gồm j−i+1 phần tử, mỗi phần tử đều có cơ hội ngang nhau làm pivot đầu tiên của khối, nên:

> P(zᵢ so sánh với zⱼ) = 2/(j−i+1)

Tổng kỳ vọng số phép so sánh, nhờ linearity, là tổng của các xác suất này trên mọi cặp — với mỗi i, tổng theo j cho ra chuỗi điều hòa 2(1/2 + 1/3 + ... ) ≈ 2 ln n, nhân với n phần tử: **E[so sánh] ≈ 2n ln n ≈ 1.39 n log₂n = Θ(n log n)**. Toàn bộ phân tích average case của thuật toán sort quan trọng nhất thế giới gói trong một trang, nhờ đúng một định lý — và định lý đó không đòi hỏi độc lập.

### Variance — kỳ vọng chưa phải toàn bộ câu chuyện

Hai hệ thống cùng latency kỳ vọng 100ms: hệ A luôn trả lời trong 95–105ms; hệ B trả lời 10ms trong 90% trường hợp và 910ms trong 10% còn lại. Cùng E[X], trải nghiệm khác nhau một trời một vực. Đại lượng đo độ phân tán quanh kỳ vọng:

> Var(X) = E[(X − E[X])²]

Với hệ thống, variance cao nghĩa là **tail latency** tệ — và người dùng nhớ p99, không nhớ trung bình. Chương này chỉ cần bạn mang theo một ý: *mọi cam kết dựa trên kỳ vọng phải đi kèm câu hỏi "phân tán quanh kỳ vọng rộng cỡ nào?"*. Chương 18 (Statistics) sẽ trả nợ chi tiết với percentile và confidence interval.

### Bốn phân phối mà engineer gặp hằng ngày

Một **phân phối (distribution)** là bản mô tả đầy đủ hành vi của biến ngẫu nhiên. Bốn gương mặt quen thuộc:

| Phân phối | Mô hình hóa | E[X] | Xuất hiện ở |
|---|---|---|---|
| Uniform trên m giá trị | mọi kết cục ngang nhau | — | hash lý tưởng, random LB, sampling |
| Bernoulli(p) / Binomial(n,p) | 1 phép thử / đếm thành công trong n phép thử | p / np | packet loss, số node hỏng trong n node |
| Geometric(p) | số lần thử **đến khi** thành công đầu tiên | 1/p | retry, số probe trong open addressing |
| Poisson(λ) | số sự kiện trong một khoảng thời gian | λ | request/giây đến server |

**Geometric** đáng thuộc lòng công thức kỳ vọng: mỗi lần thử thành công với xác suất p thì trung bình cần **1/p lần thử**. Request thành công 99% → E[số lần gửi] = 1/0.99 ≈ 1.01, retry gần như miễn phí. Nhưng khi hệ thống suy thoái còn p = 0.1 → kỳ vọng 10 lần thử — lưu lượng *nhân 10 đúng lúc hệ thống yếu nhất*. Đây là lý do toán học khiến retry không giới hạn là chất độc: nó biến suy thoái nhẹ thành sụp đổ, một cơ chế khuếch đại dương tính.

**Poisson** mô tả số sự kiện trong một khoảng, khi các sự kiện đến độc lập với cường độ trung bình λ: P(X = k) = e^{−λ}λᵏ/k!. Server nhận trung bình λ = 100 request/giây không có nghĩa mỗi giây nhận đúng 100 — Poisson cho biết xác suất nhận 130 request trong một giây nào đó là bao nhiêu, tức là **capacity planning phải chừa headroom cho dao động tự nhiên**, ngay cả khi traffic "ổn định". Poisson là viên gạch đầu tiên của queueing theory — nền toán học của mọi cuộc thảo luận về utilization và latency (và là lý do đẩy utilization lên 95% khiến queue dài ra phi tuyến).

### Birthday paradox — √ là con số phải nhớ

Bao nhiêu người trong phòng thì có 50% khả năng hai người trùng ngày sinh? Trực giác: ~183 (nửa của 365). Đáp án: **23**. Dẫn xuất xấp xỉ đáng nhớ hơn con số:

P(không ai trùng) với n người, m ngày = ∏ᵢ(1 − i/m). Dùng 1−x ≈ e^{−x} (chính xác khi x nhỏ):

> P(không trùng) ≈ e^{−(1+2+...+(n−1))/m} = e^{−n(n−1)/2m} ≈ e^{−n²/2m}

Đặt bằng 1/2, giải ra:

> **n ≈ 1.18·√m**

Căn bậc hai — đó là điều trực giác tuyến tính không bao giờ đoán được. Collision xuất hiện khi số phần tử đạt cỡ **căn bậc hai** của kích thước không gian, không phải cỡ kích thước không gian. Hệ quả trực tiếp cho engineer:

| Không gian | m | 50% collision tại n ≈ 1.18√m |
|---|---|---|
| Hash 32-bit | 2³² | **~77.000 phần tử** |
| Hash 64-bit | 2⁶⁴ | ~5 tỷ phần tử |
| UUID v4 (122 bit ngẫu nhiên) | 2¹²² | ~2.7 × 10¹⁸ |

Hash 32-bit "nghe" như 4 tỷ chỗ, nhưng chỉ cần 77 nghìn phần tử là khả năng đụng độ đã năm ăn năm thua — nếu code của bạn dùng hash 32-bit làm định danh duy nhất, nó đã sai từ thiết kế. UUID v4 thì an toàn không phải vì "không thể trùng" mà vì 2.7 tỷ tỷ phần tử là ngưỡng nhân loại không chạm tới. Cùng một công thức, hai kết luận ngược nhau — sự khác biệt nằm ở con số, và chỉ ở con số.

## 5. Thuật toán — bơm ngẫu nhiên vào code một cách có chủ đích

Xác suất đi vào thuật toán theo hai tư cách: **công cụ phân tích** (input ngẫu nhiên, thuật toán tất định — như phân tích quicksort ở trên) và **nguyên liệu thiết kế** (thuật toán tự gieo xúc xắc — randomized algorithms). Tư cách thứ hai thú vị hơn, vì nó nghe ngược đời: cố tình thêm hỗn loạn để hệ thống *đáng tin hơn*.

**Randomized quicksort** là ví dụ nguyên mẫu. Quicksort chọn pivot cố định (phần tử đầu) có worst case O(n²) trên input đã sort — và input đã sort *phổ biến* trong thực tế, thậm chí kẻ tấn công có thể cố tình gửi nó. Chọn pivot **ngẫu nhiên** không xóa worst case, nhưng thay đổi bản chất của nó: worst case không còn gắn với *input* nào nữa mà chỉ xảy ra khi *xúc xắc* xui liên tục — xác suất chuỗi lựa chọn tệ giảm theo cấp số nhân. Kẻ thù có thể chọn input xấu nhất, nhưng không thể chọn số ngẫu nhiên của bạn. **Randomization là vũ khí chống adversarial input**: nó chuyển rủi ro từ chỗ đối thủ kiểm soát được sang chỗ không ai kiểm soát được. Skip list (chương 23) dựa trên cùng triết lý — thay vì tự cân bằng cây một cách cầu kỳ, tung đồng xu để quyết định tầng, và cân bằng *kỳ vọng* tự xuất hiện.

**Fisher–Yates shuffle** — xáo trộn công bằng trong O(n):

```go
// Shuffle xáo trộn slice sao cho mọi hoán vị có xác suất bằng nhau: 1/n!
func Shuffle(a []int) {
    for i := len(a) - 1; i > 0; i-- {
        j := rand.IntN(i + 1) // chọn đều trong [0, i] — QUAN TRỌNG: gồm cả i
        a[i], a[j] = a[j], a[i]
    }
}
```

Chứng minh tính công bằng gọn bằng phép đếm: thuật toán thực hiện chuỗi lựa chọn độc lập với số khả năng n × (n−1) × ... × 2 = n!, mỗi chuỗi lựa chọn cho ra đúng một hoán vị và các chuỗi khác nhau cho hoán vị khác nhau. n! chuỗi đều khả năng như nhau ánh xạ 1–1 vào n! hoán vị → mỗi hoán vị có xác suất đúng 1/n!. Lưu ý phiên bản sai kinh điển: `j := rand.IntN(n)` (chọn trên toàn mảng thay vì [0, i]) tạo ra nⁿ chuỗi lựa chọn — và vì nⁿ **không chia hết** cho n! (với n > 2), không cách nào các hoán vị đều xác suất bằng nhau. Một bug bias từng xuất hiện trong code trộn bài của các trang poker thật, bị phát hiện chính bằng lập luận chia hết này.

**Reservoir sampling** — chọn ngẫu nhiên công bằng khi không biết trước n:

```go
// PickRandom chọn 1 phần tử đều xác suất từ stream không biết trước độ dài.
// Bộ nhớ O(1) — không lưu stream.
func PickRandom(stream <-chan int) int {
    var chosen int
    n := 0
    for v := range stream {
        n++
        if rand.IntN(n) == 0 { // xác suất 1/n thay phần tử đang giữ
            chosen = v
        }
    }
    return chosen
}
```

Chứng minh bằng induction ngắn: phần tử thứ k được chọn tại thời điểm k với xác suất 1/k, và *sống sót* qua các bước k+1..n với xác suất (k/(k+1)) × ((k+1)/(k+2)) × ... × ((n−1)/n) — chuỗi rút gọn kiểu telescoping còn k/n. Nhân lại: (1/k)·(k/n)·... = **1/n cho mọi k**. Kỹ thuật này không phải đồ chơi phỏng vấn: nó là cách chuẩn để lấy mẫu từ stream vô hạn — sample log để trace, sample event để ước lượng phân phối — với bộ nhớ hằng số.

**Exponential backoff + jitter** — bài học Giáng sinh 2012 đúc thành code:

```go
// BackoffFull trả về thời gian chờ trước lần retry thứ attempt.
// Full jitter: chờ ngẫu nhiên trong [0, min(cap, base·2^attempt)].
func BackoffFull(base, cap time.Duration, attempt int) time.Duration {
    max := min(cap, base<<attempt)          // trần tăng theo cấp số nhân
    return time.Duration(rand.Int64N(int64(max))) // jitter trải đều dưới trần
}
```

Backoff mũ giải quyết *tổng lưu lượng* retry (giảm dần theo thời gian), nhưng không giải quyết *sự đồng bộ*: nếu 10.000 client cùng fail lúc t=0, backoff thuần túy khiến cả 10.000 quay lại đúng t=1s, rồi đúng t=3s — những đợt sóng vỗ nhịp. Jitter phá vỡ sự đồng pha bằng cách trải các lần retry **đều trên một khoảng**: thay vì 10.000 request tại một thời điểm, ta có ~10.000/max request rải mỗi đơn vị thời gian. Thí nghiệm nổi tiếng của AWS Architecture Blog cho thấy full jitter giảm tổng số call cần thiết để hệ thống hồi phục xuống nhiều lần so với backoff không jitter. Chú ý điều tinh tế: từng client retry *chậm hơn một chút* so với tối ưu cục bộ của nó — mỗi cá thể hy sinh một ít kỳ vọng để tổng thể sống sót. Ngẫu nhiên ở đây là công cụ **chống tương quan** (decorrelation), đúng nghịch đảo của lỗi giả định độc lập ở mục 3: khi độc lập không tự có, ta dùng randomness để chế tạo nó.

## 6. Trade-off

**Đảm bảo tất định vs đảm bảo kỳ vọng.** Randomized quicksort cho Θ(n log n) kỳ vọng với hằng số đẹp; heap sort cho O(n log n) *mọi trường hợp* với hằng số xấu hơn. Chọn cái nào là câu hỏi về hợp đồng: hệ thống real-time cứng (điều khiển, trading khớp lệnh) cần trần tất định; hầu hết hệ thống còn lại giàu lên nhờ nhận trung bình tốt và chịu tail hiếm. Không có câu trả lời chung — chỉ có câu hỏi đúng: *"nếu rơi vào đuôi xấu, cái giá là gì, và ai trả?"*

**Kỳ vọng vs phương sai.** Hai chiến lược cùng kỳ vọng có thể khác hẳn nhau về độ ổn định. Load balancer random và round-robin cùng cho tải kỳ vọng bằng nhau trên mỗi server, nhưng phương sai của random cao hơn — và server chết vì *đỉnh* tải, không chết vì tải trung bình. Khi so sánh hai phương án, hỏi cả hai con số.

**Chi phí của randomness.** Số ngẫu nhiên không miễn phí: CSPRNG (crypto/rand) đắt hơn PRNG thường (math/rand/v2) hàng chục lần; PRNG có state cần đồng bộ giữa goroutine. Và **chất lượng** randomness là một trục trade-off thật: dùng math/rand cho token bảo mật là lỗ hổng nghiêm trọng (seed đoán được → toàn bộ chuỗi đoán được), dùng crypto/rand để chọn pivot quicksort là lãng phí thuần túy.

**Khi nào KHÔNG nên dùng xác suất.** Ba vùng cấm rõ ràng. Một: **đếm tiền** — billing, ledger, inventory không có chỗ cho "đúng với xác suất cao"; sai một xu là sai. Hai: **n nhỏ** — luật số lớn cần số lớn; với 5 server, "kỳ vọng tải đều" gần như vô nghĩa vì phương sai tương đối khổng lồ, cứ round-robin tất định cho lành. Ba: **khi bạn không thể mô tả phân phối** — mọi phân tích ở chương này đứng trên giả định về phân phối input; nếu không biết gì về phân phối và hậu quả sai là lớn, phân tích worst-case của chương 10 là nơi trú ẩn đúng.

## 7. Production Applications

**Cache và expected latency — kỳ vọng thành công thức định cỡ.** Với hit ratio h, latency khi hit t_hit, khi miss t_miss:

> E[latency] = h·t_hit + (1−h)·t_miss

Con số này dạy một bài phản trực giác: Redis hit 1ms, database miss 100ms, hit ratio 90% → E = 0.9×1 + 0.1×100 = 10.9ms. Nâng hit ratio lên 99% → E = 1.99ms. **Mười điểm phần trăm cuối cùng của hit ratio giảm latency 5.5 lần** — vì kỳ vọng bị thống trị bởi số hạng miss. Hệ quả kỹ nghệ: đo và tối ưu *miss path* quan trọng ngang tối ưu cache; và một sự kiện làm rơi hit ratio (deploy xóa cache, cache stampede) làm latency kỳ vọng nhảy vọt phi tuyến — đó là lý do các hệ lớn làm nóng cache (warm-up) trước khi nhận traffic.

**Load balancing — the power of two choices.** Ném n request vào n server thuần ngẫu nhiên: tải kỳ vọng mỗi server là 1, nhưng server xui nhất nhận cỡ **Θ(log n / log log n)** request — với hệ lớn là chênh lệch đáng kể, và tail latency của cả hệ do server xui nhất quyết định. Bây giờ đổi một chi tiết nhỏ: **chọn ngẫu nhiên 2 server, hỏi tải của đúng 2 server đó, gửi vào bên nhẹ hơn**. Max load rơi xuống **Θ(log log n)** — gần như hằng số. Trực giác của bước nhảy vọt: để một server đạt tải k, cả *hai* lựa chọn đều phải rơi vào vùng đã nặng; xác suất vùng nặng co lại kiểu lũy thừa kép theo từng mức k, nên các mức tải cao gần như tuyệt chủng. Cái giá phải trả gần bằng không (thêm một lần hỏi tải), phần thưởng là cải thiện *hàm mũ* — một trong những tỷ lệ lợi ích/chi phí đẹp nhất của khoa học máy tính, và là thuật toán mặc định trong Envoy (`LEAST_REQUEST`) và NGINX (`random two least_conn`). Bài học lớn hơn: một *chút* thông tin (2 mẫu) tốt hơn không có thông tin một cách phi tuyến, trong khi thông tin *đầy đủ* (hỏi tất cả server) không đáng chi phí điều phối.

**Retry, timeout và số học của suy thoái.** Từ phân phối geometric: hệ khỏe (p → 1) retry rẻ như cho; hệ yếu (p nhỏ) retry là xăng đổ vào lửa. Các hệ trưởng thành mã hóa nhận thức này thành **retry budget** (ví dụ tối đa 20% lưu lượng là retry — có trong gRPC và các service mesh) và **circuit breaker** — về bản chất là công tắc chuyển chế độ giữa "tin vào kỳ vọng" và "ngừng đánh cược". Kèm jitter như mục 5 để chống đồng pha.

**Go runtime** dùng randomness ở những chỗ ít ai để ý: hash seed ngẫu nhiên per-process chống hash-flooding (chương 12 phân tích kỹ); **thứ tự duyệt map được cố ý ngẫu nhiên hóa** mỗi lần duyệt — quyết định thiết kế nhằm bẻ gãy các chương trình vô tình phụ thuộc thứ tự (một dạng dùng randomness để *ép* contract được tôn trọng); scheduler thỉnh thoảng random hóa lựa chọn goroutine để tránh starvation có hệ thống.

**Kafka và Elasticsearch** dựa vào tính chất "uniform hash ⇒ cân bằng kỳ vọng" để rải dữ liệu vào partition/shard mà **không cần điều phối viên trung tâm** — mỗi producer tự tính hash, độc lập, và cân bằng tổng thể tự xuất hiện như một định lý thay vì như một dịch vụ. Đổi lại là rủi ro phân phối input lệch: một key nóng (celebrity user) phá vỡ giả định uniform, tạo hot partition — lời nhắc rằng mọi cam kết xác suất chỉ tốt bằng giả định phân phối của nó.

## 8. Interview

Xác suất xuất hiện trong phỏng vấn theo hai lối: bài toán randomized algorithm trực tiếp, và câu hỏi "expected complexity" cài trong bài khác. Ba bài kinh điển:

**Random Pick with Weight (LeetCode 528).** Chọn index i với xác suất w[i]/Σw. Insight: biến bài xác suất thành bài *hình học trên trục số* — nối các đoạn có độ dài w[i] liền nhau thành đoạn [0, Σw), gieo một điểm uniform, điểm rơi vào đoạn nào chọn index đó. Cài đặt: mảng prefix sum + binary search điểm rơi → O(n) chuẩn bị, O(log n) mỗi lần chọn. Đây chính là cách mọi hệ thống weighted routing (canary deployment 5% traffic, A/B testing) hoạt động — bài phỏng vấn này *là* production code.

**Shuffle an Array (LeetCode 384).** Fisher–Yates như mục 5. Interviewer giỏi sẽ hỏi đúng một câu: *"chứng minh mỗi hoán vị đều xác suất 1/n!"* — và lập luận đếm n! chuỗi lựa chọn ánh xạ 1–1 vào n! hoán vị là câu trả lời trọn vẹn trong ba câu nói. Bẫy chấm điểm: bản sai `rand.IntN(n)` chạy qua mọi unit test thông thường — chỉ toán học (hoặc test thống kê trên hàng triệu lần chạy) phát hiện được bias. Một minh họa đắt giá cho giới hạn của testing: **test chứng minh được sự tồn tại của bug, không chứng minh được sự vắng mặt của bias.**

**Linked List Random Node (LeetCode 382).** Reservoir sampling như mục 5, kèm chứng minh telescoping. Câu mở rộng tự nhiên: chọn k phần tử thay vì 1 (giữ reservoir k chỗ, phần tử thứ i thay một chỗ ngẫu nhiên với xác suất k/i); rồi "áp dụng vào đâu?" — sampling log/trace stream chính là câu trả lời production.

**Lỗi tư duy thường gặp:**

- Cộng xác suất của các biến cố *không rời nhau*: P(A hoặc B) = P(A) + P(B) − P(A∩B), quên số hạng cuối là lỗi số một. (Inclusion-Exclusion — chương 05.)
- Nhân xác suất của các biến cố *không độc lập* — lỗi RAID ở mục 3.
- Tính kỳ vọng rồi trả lời như thể nó là giá trị chắc chắn xảy ra ("trung bình 1.01 lần retry nên timeout 2 lần là đủ" — đuôi phân phối không đồng ý).
- Với bài "expected time", cố phân tích worst case — nhận diện từ khóa "average/expected/random" để chuyển sang công cụ linearity + indicator variables thay vì đếm trực tiếp.

**Cách phân tích thay vì học thuộc:** gặp bài xác suất, đi theo trình tự: (1) sample space là gì, mỗi kết cục có đều nhau không; (2) đại lượng cần tính là xác suất một biến cố hay kỳ vọng một biến ngẫu nhiên; (3) nếu là kỳ vọng của một *tổng đếm được* — tách thành indicator, dùng linearity, xong; (4) nếu là xác suất có điều kiện — vẽ cây đếm 100.000 trường hợp như mục 2, đừng tin trực giác. Bốn bước này giải được đại đa số câu hỏi xác suất trong phỏng vấn kỹ sư.

## 9. Anti-pattern

**Giả định độc lập vì phép nhân tiện.** "Mỗi service uptime 99.9%, ba service thì 99.7%" — chỉ đúng khi các service không chia chung DNS, chung certificate authority, chung region, chung pipeline deploy. Trong thực tế chúng chung gần hết, và sự cố lớn hầu hết là correlated failure. Trước khi nhân xác suất, liệt kê **các nguồn chung** (shared fate) — đó là bài kiểm tra độc lập thực dụng nhất.

**Retry không jitter, không budget.** Backoff mũ mà không jitter là những đợt sóng đồng pha; retry không budget là bộ khuếch đại lưu lượng có hệ số 1/p bùng nổ đúng lúc p sụp. Cả hai đều là "làm đúng một nửa" — và một nửa còn thiếu là nửa mang tính xác suất.

**Dùng nhầm loại randomness.** `math/rand` cho session token: kẻ tấn công tái tạo được chuỗi. `time.Now().UnixNano() % n` làm "số ngẫu nhiên": tương quan chặt giữa các lần gọi gần nhau — đám client khởi động cùng lúc sẽ "ngẫu nhiên" ra cùng giá trị, tái hiện chính thundering herd mà bạn định tránh. Quy tắc: bảo mật → `crypto/rand`; còn lại → `math/rand/v2`; không bao giờ tự chế từ timestamp.

**Quyết định theo trung bình, mù với đuôi.** Autoscale theo CPU trung bình khi tải đến theo Poisson: các burst tự nhiên vượt trung bình thường xuyên và đủ dài để làm rớt request trước khi scaler kịp phản ứng. Trung bình là con số dễ đo nhất và dễ gây ảo tưởng nhất; hệ thống chết ở percentile cao.

**Alert được tối ưu cho độ nhạy, bỏ quên base rate.** Như mục 2: sự cố thật hiếm, nên alert nhạy mà không đặc hiệu sẽ có precision một chữ số phần trăm, và con người sẽ (hợp lý!) ngừng tin nó. Định kỳ tính P(sự cố thật | alert) từ lịch sử on-call — nếu dưới ~50%, alert đang đào tạo team bỏ qua chính nó.

## 10. Best Practices

**Nên:**

- Khi cần kỳ vọng của một đại lượng đếm được, mặc định dùng **indicator + linearity** — không cần độc lập, không cần phân phối đầy đủ, thường ra kết quả trong vài dòng.
- Kiểm chứng mọi trực giác xác suất bằng **phép đếm trên quần thể cụ thể** (cây 100.000 người) hoặc bằng mô phỏng Monte Carlo 20 dòng Go — rẻ, và trực giác xác suất sai thường xuyên đủ để đáng làm.
- Nhớ hai hằng số thực dụng: collision đạt 50% ở **n ≈ 1.18√m**, và retry kỳ vọng **1/p** lần — chúng trả lời tại chỗ các câu hỏi thiết kế về kích thước hash, độ dài ID, và chính sách retry.
- Mọi retry phải có backoff mũ **và** jitter **và** trần số lần thử; mọi con số "kỳ vọng" trong tài liệu thiết kế phải kèm phát biểu về phân tán (tối thiểu: p99).
- Thiết kế sự độc lập một cách tường minh (multi-AZ, nguồn randomness riêng, tránh shared fate) thay vì giả định nó.

**Không nên:**

- Không dùng xác suất cho bất biến nghiệp vụ tuyệt đối (tiền, tính duy nhất của ID trong hệ nhỏ mà một cột auto-increment giải quyết được).
- Không nhân xác suất khi chưa trả lời được "hai sự kiện này có nguồn chung nào không".
- Không tin đảm bảo kỳ vọng khi input do đối thủ kiểm soát mà randomness của bạn lại đoán được — kỳ vọng chỉ có nghĩa khi xúc xắc thật sự ngẫu nhiên với kẻ tấn công.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Vì sao linearity of expectation không cần các biến độc lập, và điều đó được khai thác thế nào trong phân tích quicksort — nơi các sự kiện "so sánh cặp (i,j)" rõ ràng phụ thuộc nhau?
2. Xét nghiệm chính xác 99% cho kết quả dương tính — hãy giải thích cho một người không học toán vì sao xác suất mắc bệnh thật có thể chỉ là 9%, chỉ bằng phép đếm.
3. Backoff mũ đã giảm tổng lưu lượng retry theo thời gian; vậy jitter giải quyết thêm vấn đề gì mà backoff không chạm tới được?

---

*Chương tiếp theo: [12 — Hashing](/series/math-for-engineers/level-3-algorithm-mathematics/12-hashing/), nơi các công cụ vừa xây — uniform distribution, linearity of expectation, birthday paradox — hợp sức giải thích cấu trúc dữ liệu quan trọng bậc nhất của lập trình hiện đại: hash table.*
