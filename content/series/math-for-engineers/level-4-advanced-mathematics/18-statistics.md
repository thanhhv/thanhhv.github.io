+++
title = "Chương 18 — Statistics: Đọc hiểu hệ thống qua dữ liệu"
date = "2026-07-20T10:00:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 4 – Advanced Mathematics
> Yêu cầu trước: Chương 11 (Probability), Chương 16 (Sorting, Searching & Heap)

---

## 1. Problem Statement

Một buổi sáng thứ Hai, team backend nhận ticket từ customer support: "Nhiều khách hàng phàn nàn app chậm, có người chờ 3–4 giây mới load xong trang chủ." Kỹ sư trực mở dashboard, nhìn vào biểu đồ latency của API gateway: **average response time = 52ms**, ổn định suốt tuần. Anh trả lời ticket: "Hệ thống bình thường, có thể do mạng phía khách hàng."

Hai tuần sau, một khách hàng doanh nghiệp lớn hủy hợp đồng. Post-mortem phát hiện: một truy vấn database thiếu index khiến khoảng 2% request chậm hơn 3 giây. 98% request còn lại nhanh (~30ms) — đủ để kéo *trung bình* xuống 52ms và che giấu hoàn toàn 2% thảm họa. Mà 2% của 10 triệu request mỗi ngày là **200.000 trải nghiệm tồi tệ mỗi ngày** — rơi trúng những khách hàng có nhiều dữ liệu nhất, tức là những khách hàng quan trọng nhất.

Dashboard không nói dối. Nó chỉ trả lời sai câu hỏi. "Trung bình bao nhiêu?" không phải là câu hỏi mà user đang hỏi; user hỏi "*trải nghiệm tệ nhất mà tôi thường gặp là gì?*"

Đây là bài toán mà thống kê giải quyết cho người kỹ sư:

**Làm sao nén hàng triệu con số đo được thành vài con số tóm tắt — mà không đánh mất sự thật quan trọng nhất?**

Và bài toán ngược lại, khó không kém:

**Khi hai bản đo khác nhau (bản A nhanh hơn bản B 3%), làm sao biết đó là khác biệt thật hay chỉ là nhiễu ngẫu nhiên?**

Không có thống kê, kỹ sư chỉ còn hai lựa chọn tồi: tin vào con số tóm tắt sai (như câu chuyện trên), hoặc tin vào cảm giác ("tôi thấy có vẻ nhanh hơn") — thứ còn tệ hơn.

## 2. Trực giác

### Một con số không thể tóm tắt một phân phối

Hãy tưởng tượng bạn phải mô tả chiều cao của dân cư một thành phố bằng đúng một con số. "Trung bình 1m65" nghe ổn — vì chiều cao phân bố khá đối xứng quanh giá trị giữa. Giờ mô tả *thu nhập* bằng một con số: "trung bình 25 triệu/tháng" — trong khi 90% dân số kiếm dưới 15 triệu và một nhóm nhỏ kiếm hàng tỷ. Con số trung bình đúng về mặt số học nhưng không mô tả *bất kỳ ai* trong thành phố.

Sự khác biệt nằm ở **hình dạng phân phối**. Trung bình (mean) chỉ trung thực khi phân phối đối xứng và không có đuôi dài. Thu nhập, tài sản, kích thước file, số follower — và đặc biệt là **latency** — đều lệch phải (right-skewed): đa số giá trị nhỏ, một thiểu số cực lớn.

```
 số request
    │██
    │████
    │██████
    │███████
    │████████
    │█████████▌
    │██████████▊
    │███████████▍____
    │████████████████████▂▂▂▁▁▁▁▁▁▁▁ . . . . . →  đuôi dài
    └──┬────┬─────┬──────────┬──────────┬──── latency
      10ms 30ms  52ms      500ms       3s
            ↑     ↑                     ↑
         median  mean              nơi user đau khổ
                (bị đuôi kéo lên)
```

Latency lệch phải vì lý do vật lý: nó bị **chặn dưới** (không thể nhanh hơn 0ms, thường không nhanh hơn round-trip time của mạng) nhưng **không bị chặn trên** (GC pause, lock contention, retry, disk seek, timeout — mọi thứ xấu đều cộng dồn về một phía). Không tồn tại request "nhanh bất thường 3 giây", nhưng request "chậm bất thường 3 giây" xảy ra hằng ngày.

### Câu hỏi đúng: "bao nhiêu phần trăm user có trải nghiệm tốt hơn X?"

Thay vì hỏi "trung bình bao nhiêu", hãy hỏi: "99% request nhanh hơn giá trị nào?" Đó là **percentile** — và nó chính là ngôn ngữ mà user thực sự nói, chỉ là họ diễn đạt bằng câu "thỉnh thoảng app lag kinh khủng".

### Trực giác thứ hai: mọi phép đo đều là mẫu ngẫu nhiên

Chạy benchmark một lần được 105ns. Chạy lại: 98ns. Lại: 112ns. Con số nào là "sự thật"? Không con số nào cả — mỗi lần chạy là một **mẫu (sample)** rút từ một phân phối ẩn, bị nhiễu bởi scheduler, cache, nhiệt độ CPU, background process. Thống kê cho ta khung để trả lời: cần bao nhiêu mẫu, và kết luận được phép mạnh đến đâu. Đây là nửa sau của chương — và là nửa mà nhiều kỹ sư giàu kinh nghiệm vẫn làm sai.

## 3. First Principles

### Ba cách định nghĩa "điểm giữa" — và chúng trả lời ba câu hỏi khác nhau

Cho tập dữ liệu x₁, x₂, ..., xₙ:

- **Mean** (trung bình cộng): x̄ = (Σxᵢ)/n. Trả lời: "*nếu chia đều tổng cho mọi người thì mỗi người được bao nhiêu?*" Mean gắn với **tổng** — nên nó là con số đúng khi thứ bạn quan tâm là tổng: tổng chi phí, tổng throughput, tổng dung lượng. Capacity planning dùng mean là hợp lý: 10⁶ request/ngày × mean 50ms CPU = tổng CPU-giây cần có.
- **Median** (trung vị): giá trị đứng giữa khi sắp xếp — chính là percentile thứ 50. Trả lời: "*một người điển hình trải nghiệm gì?*" Median miễn nhiễm với outlier: một request 30 giây không xê dịch median dù chỉ 1ms.
- **Mode** (yếu vị): giá trị xuất hiện nhiều nhất. Ít dùng với dữ liệu liên tục, nhưng đắc dụng khi phân phối **đa đỉnh (multimodal)**: nếu histogram latency có hai đỉnh (5ms và 80ms), gần như chắc chắn hệ thống có hai đường đi — cache hit và cache miss. Mean của phân phối hai đỉnh (~40ms) mô tả một loại request *không hề tồn tại*.

Quy tắc nhận diện mean lừa dối: **mean ≫ median → phân phối lệch phải, có đuôi**. Chênh lệch giữa hai con số này là "máy dò đuôi" rẻ nhất mà bạn có.

### Nếu không có percentile thì điều gì xảy ra?

Quay lại câu chuyện mở đầu: hệ thống có 2% request thảm họa mà mọi chỉ số trung bình đều xanh. Không có percentile, cách duy nhất phát hiện là... chờ khách hàng hủy hợp đồng. Percentile tồn tại vì **chất lượng dịch vụ không phải là đại lượng cộng dồn** — một triệu request nhanh không "đền bù" được cho một request khiến user bỏ đi. Trải nghiệm không có tính chất tuyến tính; mean thì có. Đó là sự vênh nhau về bản chất.

### Nếu không có kiểm định giả thuyết thì điều gì xảy ra?

Bạn đổi thuật toán, benchmark nhanh hơn 3%, merge. Tuần sau đồng nghiệp revert một thay đổi khác, benchmark lại "nhanh hơn 2%". Sự thật: cả hai lần đều là nhiễu — biến động giữa hai lần chạy trên cùng một code đã là ±5%. Không có khung thống kê, team sẽ tích lũy hàng trăm "tối ưu hóa" không có thật, mỗi cái kèm chi phí phức tạp rất thật.

## 4. Mathematical Model

### Percentile — định nghĩa và bộ tứ p50/p95/p99/p999

> Percentile thứ k (pₖ) là giá trị v sao cho k% quan sát ≤ v.

Sắp xếp n mẫu tăng dần, p99 là phần tử ở vị trí ⌈0.99·n⌉. Bộ tứ chuẩn của SRE:

| Percentile | Tên gọi | Ý nghĩa vận hành |
|---|---|---|
| p50 | median | Trải nghiệm điển hình — "hệ thống nhanh cỡ nào?" |
| p95 | tail bắt đầu | 1/20 request tệ hơn mức này — user thường xuyên chạm |
| p99 | tail | 1/100 request — với 10⁶ req/ngày là 10.000 lần đau/ngày |
| p999 | deep tail | 1/1000 — mức cam kết của hệ thống tài chính, ads bidding |

Vì sao SLO (Service Level Objective) viết bằng percentile chứ không phải mean? Vì SLO là **lời hứa về phân phối trải nghiệm**: "99% request dưới 300ms" là cam kết kiểm chứng được và khớp với cảm nhận user. "Mean dưới 300ms" có thể đúng trong khi 5% user chờ 10 giây — một lời hứa vô nghĩa. Ngoài ra percentile còn có tính chất phòng thủ: muốn cải thiện p99, bạn *buộc* phải sửa đường đi chậm nhất; muốn cải thiện mean, bạn có thể chỉ cần làm đường đi nhanh... nhanh hơn nữa — tối ưu sai chỗ mà chỉ số vẫn đẹp.

### Tail latency amplification — vì sao hệ phân tán khuếch đại đuôi

Đây là kết quả quan trọng nhất chương, vì nó biến một con số tưởng như vô hại thành quyết định kiến trúc. Giả sử một trang web fan-out gọi **100 service con song song** (search: query hàng trăm leaf node; trang chủ e-commerce: giá, tồn kho, recommendation, ads...), và phải chờ *tất cả* trả lời. Mỗi service có p99 = 100ms, tức xác suất một lời gọi chậm hơn 100ms chỉ là 1%.

Xác suất *toàn bộ* request tổng nhanh (không chạm đuôi của bất kỳ service nào), giả định các service độc lập:

> P(cả 100 đều nhanh) = 0.99¹⁰⁰ ≈ e^(−100×0.01) = e⁻¹ ≈ 0.366

Vậy **P(chạm đuôi của ít nhất 1 service) = 1 − 0.99¹⁰⁰ ≈ 63.4%**.

Đọc lại lần nữa: sự kiện "hiếm" 1% ở mức component trở thành sự kiện **đa số** ở mức hệ thống. p99 của từng service trở thành xấp xỉ *median* của request tổng. Vài mốc đáng nhớ:

| Số service fan-out | P(chạm p99 của ≥1 service) |
|---|---|
| 1 | 1% |
| 10 | ≈ 9.6% |
| 30 | ≈ 26% |
| 100 | ≈ 63% |
| 500 | ≈ 99.3% |

Hệ quả kiến trúc (theo bài "The Tail at Scale" của Dean & Barroso):

- **Đuôi của component là median của hệ thống** — muốn hệ lớn nhanh, phải quản trị p99/p999 của từng phần, không phải mean.
- **Hedged requests**: gửi request; nếu sau p95-latency chưa có trả lời, gửi bản sao tới replica khác, lấy kết quả về trước. Chi phí: ~5% traffic thêm. Lợi ích: cắt cụt đuôi — vì xác suất *cả hai* replica cùng chậm là 0.05 × 0.05 = 0.25% (nếu độc lập). Đây là toán chương 11 (nhân xác suất sự kiện độc lập) trở thành pattern kiến trúc.
- Biến thể: **tied requests** (gửi hai bản ngay, bản nào được xử lý thì cancel bản kia), giới hạn queue, ưu tiên request đã chờ lâu.

### Variance và Standard Deviation — đo độ phân tán

> Variance: σ² = E[(X − μ)²] — trung bình của *bình phương* độ lệch khỏi mean.
> Standard deviation: σ = √σ² — kéo về cùng đơn vị với dữ liệu.

Vì sao bình phương mà không lấy trị tuyệt đối? Ba lý do thực dụng: bình phương phạt nặng độ lệch lớn (đúng với trực giác "jitter lớn tệ hơn nhiều jitter nhỏ"), khả vi (cần cho optimization — chương 21), và có tính chất cộng đẹp: **variance của tổng các biến độc lập bằng tổng các variance** — nền của Central Limit Theorem ngay dưới đây.

Trong network engineering, σ của latency chính là **jitter**. Hai đường mạng cùng mean 50ms nhưng σ = 2ms và σ = 30ms là hai trải nghiệm hoàn toàn khác nhau với video call — codec phải giữ buffer cỡ vài σ để hấp thụ dao động. Mean nói "nhanh bao nhiêu", σ nói "**đáng tin cậy bao nhiêu**"; hệ thống tốt cần cả hai, và nhiều khi σ nhỏ quý hơn mean nhỏ.

### Phân phối chuẩn và Central Limit Theorem

**Central Limit Theorem (CLT)**, mức trực giác: cộng nhiều biến ngẫu nhiên độc lập, cùng cỡ ảnh hưởng — bất kể từng biến phân phối ra sao — thì tổng (hay trung bình) tiến về **phân phối chuẩn** (hình chuông). Vì sao? Vì để tổng lệch xa kỳ vọng, *nhiều* yếu tố phải cùng lệch về *một phía* — mà các cách kết hợp "kẻ lệch trái bù kẻ lệch phải" nhiều hơn áp đảo (đếm tổ hợp — chương 05). Số cách triệt tiêu lấn át số cách cộng hưởng, nên khối lượng xác suất dồn về giữa, mỏng dần ra hai bên: hình chuông.

CLT giải thích vì sao chiều cao, sai số đo đạc, trung bình của mẫu lớn... đều xấp xỉ chuẩn. Và nó là nền tảng cho confidence interval bên dưới: **trung bình của n mẫu** có phân phối xấp xỉ chuẩn quanh mean thật với độ lệch chuẩn σ/√n — căn bậc hai này chính là lý do "muốn giảm nhiễu 10 lần phải đo 100 lần".

**CẢNH BÁO dành riêng cho SRE**: latency **không** phân phối chuẩn — nó lệch phải, đuôi dày, thường đa đỉnh. CLT không áp dụng cho *từng quan sát* latency (chỉ áp dụng cho *trung bình của nhiều* quan sát). Hệ quả: quy tắc "mean ± 3σ chứa 99.7% dữ liệu" chỉ đúng với phân phối chuẩn; đặt ngưỡng alert kiểu `mean + 3·stddev` trên latency sẽ vừa báo động giả (σ bị chính outlier thổi phồng làm ngưỡng nhảy loạn), vừa bỏ sót đuôi (99.7% không còn đúng). **Alert trên percentile đo trực tiếp, đừng alert trên mô hình phân phối mà dữ liệu không tuân theo.**

### Confidence Interval — kết quả đo tin được đến đâu

Chạy benchmark n = 10 lần, được mean x̄ và độ lệch chuẩn mẫu s. Khoảng tin cậy 95% cho mean thật:

> x̄ ± t · s/√n  (t ≈ 2.26 với n = 10, từ phân phối Student-t)

Diễn giải **chính xác**: nếu lặp lại toàn bộ quy trình đo nhiều lần, khoảng 95% các khoảng được dựng ra sẽ chứa giá trị thật. Diễn giải thực dụng cho kỹ sư: **đó là thanh sai số của phép đo**. Khi so sánh hai phiên bản code, nếu hai khoảng tin cậy chồng lên nhau đáng kể — bạn *chưa đo được* sự khác biệt, dù hai con số mean khác nhau.

Đây là lý do chạy benchmark một lần là vô nghĩa: một mẫu không cho biết s, tức không cho biết thanh sai số rộng bao nhiêu. Trong Go:

```bash
go test -bench=BenchmarkParse -count=10 > old.txt
# ... đổi code ...
go test -bench=BenchmarkParse -count=10 > new.txt
benchstat old.txt new.txt
```

`benchstat` làm đúng bài toán thống kê này: ước lượng phân tán, chạy kiểm định (Mann-Whitney U — không giả định phân phối chuẩn), và chỉ in "-5.2% (p=0.002)" khi khác biệt có ý nghĩa; ngược lại in dấu `~` — "không kết luận được". Dấu `~` đó là thống kê đang cứu bạn khỏi việc merge nhiễu.

### Hypothesis Testing và p-value — định nghĩa chính xác

Khung suy luận: muốn chứng minh "phiên bản B tốt hơn A", ta giả định điều ngược lại — **null hypothesis H₀: "không có khác biệt"** — rồi hỏi dữ liệu có đủ bất thường để bác bỏ H₀ không.

> **p-value = xác suất quan sát được dữ liệu cực đoan như (hoặc hơn) dữ liệu thực tế, GIẢ SỬ H₀ đúng.**

p-value nhỏ (quy ước < 0.05) nghĩa là: "nếu thật sự không có khác biệt thì dữ liệu thế này rất khó xảy ra" → bác bỏ H₀. Cấu trúc logic giống chứng minh phản chứng (chương 03), nhưng xác suất hóa: không đạt được chắc chắn, chỉ đạt "H₀ khó sống sót với dữ liệu này".

Các cách hiểu **SAI** phổ biến — sai cả trong giới engineering lẫn khoa học:

| Hiểu sai | Vì sao sai |
|---|---|
| "p = 0.03 nghĩa là 97% khả năng B tốt hơn A" | p-value là P(dữ liệu \| H₀), không phải P(H₀ \| dữ liệu). Đảo ngược điều kiện cần Bayes và prior (chương 11) |
| "p > 0.05 chứng minh không có khác biệt" | Không bác bỏ được ≠ chứng minh H₀. Có thể chỉ là thiếu mẫu |
| "p càng nhỏ, hiệu ứng càng lớn" | p đo *độ chắc*, không đo *độ lớn*. n khổng lồ làm khác biệt 0.01% cũng có p tí hon — có ý nghĩa thống kê nhưng vô nghĩa kinh doanh. Luôn nhìn kèm effect size |

### Sample size và peeking problem — gian lận thống kê vô thức

A/B test cần **quyết định cỡ mẫu trước khi chạy** (dựa trên hiệu ứng tối thiểu muốn phát hiện và độ nhạy mong muốn). Điều cấm kỵ: nhìn kết quả mỗi ngày và **dừng ngay khi p < 0.05**. Vì sao đây là gian lận? p-value dao động ngẫu nhiên trong quá trình thu mẫu; nếu bạn kiểm tra 20 lần và dừng ở lần đầu tiên vượt ngưỡng, xác suất *ít nhất một lần* vượt ngưỡng do may rủi lớn hơn 5% rất nhiều — cùng cấu trúc toán với tail amplification ở trên: 1 − 0.95²⁰ ≈ 64%. Kiểm tra càng nhiều lần, "false positive" gần như chắc chắn. Tên gọi: **peeking problem**. Giải pháp: cố định n trước, hoặc dùng sequential testing (ngưỡng hiệu chỉnh cho phép nhìn liên tục) — các nền tảng A/B testing nghiêm túc cài sẵn điều này.

### Correlation vs Causation

Hệ số tương quan r ∈ [−1, 1] đo mức độ hai đại lượng *cùng biến thiên tuyến tính*. Tương quan **không** suy ra nhân quả, vì còn hai cách giải thích khác luôn phải loại trừ: **biến ẩn chung** và **chiều ngược lại**. Ví dụ vận hành kinh điển: metric "error rate tăng ngay sau các đợt deploy" — deploy gây lỗi? Có thể. Nhưng cũng có thể cả hai cùng bị điều khiển bởi biến ẩn: deploy diễn ra giờ hành chính, đúng lúc traffic đạt đỉnh, và chính traffic đỉnh gây lỗi. Cách duy nhất tách bạch là **can thiệp có kiểm soát**: deploy vào giờ thấp điểm, hoặc canary trên một phần traffic được chọn ngẫu nhiên — randomization chính là công cụ chặt đứt biến ẩn, và là lý do A/B test *phân bổ ngẫu nhiên* chứ không để user tự chọn.

## 5. Thuật toán — tính percentile khi dữ liệu là dòng chảy vô hạn

Tính chính xác p99 cần giữ toàn bộ mẫu và sort — O(n) bộ nhớ, bất khả thi khi n là hàng tỷ request. Ba tầng giải pháp:

**Tầng 1 — Chính xác, offline:** sort rồi lấy phần tử thứ ⌈0.99·n⌉; hoặc Quickselect O(n) average (chương 13). Dùng cho phân tích log sau sự cố.

**Tầng 2 — Histogram bucket cố định (cách của Prometheus):** chia trục latency thành các bucket định trước, mỗi bucket một counter — bộ nhớ O(số bucket), merge được giữa các máy bằng cách cộng counter (điều mà percentile tính sẵn *không* làm được: p99 của hai máy không suy ra p99 tổng!). Đổi lại, percentile phải **nội suy** trong bucket:

```go
// Ước lượng percentile từ histogram — chính là logic histogram_quantile của PromQL
// buckets: biên trên mỗi bucket (tăng dần), counts: số quan sát rơi vào từng bucket
func estimatePercentile(buckets []float64, counts []int, p float64) float64 {
    total := 0
    for _, c := range counts {
        total += c
    }
    target := p * float64(total) // hạng (rank) của quan sát cần tìm
    cum := 0
    for i, c := range counts {
        if float64(cum+c) >= target {
            lo := 0.0
            if i > 0 {
                lo = buckets[i-1]
            }
            // Nội suy tuyến tính: GIẢ ĐỊNH dữ liệu rải đều trong bucket.
            // Sai số tối đa = độ rộng bucket → bucket đặt sai chỗ = percentile sai to.
            frac := (target - float64(cum)) / float64(c)
            return lo + frac*(buckets[i]-lo)
        }
        cum += c
    }
    return buckets[len(buckets)-1]
}
```

Hệ quả cần khắc cốt: nếu SLO là 300ms mà bucket gần nhất là {250ms, 500ms}, mọi giá trị p99 trong khoảng đó đều bị nội suy từ giả định "rải đều" — **p99 báo cáo có thể lệch hàng chục phần trăm**. Luôn đặt một biên bucket trùng ngưỡng SLO.

**Tầng 3 — Cấu trúc thích nghi (mức giới thiệu):** **HDR Histogram** dùng bucket rộng theo lũy thừa (sai số *tương đối* cố định, ví dụ ±1% ở mọi thang từ 1µs đến 1h); **t-digest** tự co giãn cluster — dày đặc ở hai đuôi nơi percentile cần chính xác, thưa ở giữa. Cả hai: bộ nhớ hằng số (vài KB), merge được, sai số có kiểm soát — cùng triết lý "đổi độ chính xác lấy bộ nhớ" của chương 23.

Còn median trên stream với độ chính xác tuyệt đối? Hai heap (mục 8) — O(n) bộ nhớ nhưng O(log n) mỗi phần tử.

## 6. Trade-off

**Mean vs Percentile.** Percentile trung thực với trải nghiệm nhưng *không cộng được*: không thể lấy mean của các p99, không thể cộng p99 hai service nối tiếp để ra p99 tổng (phải đo end-to-end hoặc giữ histogram và convolve). Mean thì cộng, nhân, hợp thành thoải mái — nên capacity planning, cost model vẫn thuộc về mean. Dùng đúng việc: **percentile cho chất lượng, mean cho tài nguyên**.

**Độ chính xác vs bộ nhớ vs khả năng merge.** Sort chính xác tuyệt đối nhưng O(n) và không merge; Prometheus histogram O(k) và merge tốt nhưng sai số phụ thuộc tay người đặt bucket; t-digest/HDR tự thích nghi nhưng cấu trúc phức tạp hơn. Không có lựa chọn trội tuyệt đối — chỉ có lựa chọn khớp với câu hỏi cần trả lời.

**Độ nhạy thống kê vs tốc độ ra quyết định.** Muốn phát hiện khác biệt 1% cần mẫu lớn gấp ~100 lần so với phát hiện khác biệt 10% (sai số co theo 1/√n → hiệu ứng nhỏ 10 lần cần n lớn 100 lần). Chờ đủ mẫu thì chậm ship; dừng sớm thì peeking. Sequential testing là điểm giữa có kỷ luật — trả giá bằng ngưỡng khắt khe hơn.

**False positive vs false negative trong alerting.** Ngưỡng alert nhạy → đánh thức on-call vì nhiễu, dẫn tới alert fatigue và bỏ sót sự cố thật (chi phí của false positive là *mất lòng tin*). Ngưỡng trễ → sự cố cháy 20 phút mới báo. Thống kê không xóa được trade-off này, chỉ giúp đặt nó ở vị trí có chủ đích: alert trên p99 kéo dài qua cửa sổ thời gian (ví dụ "p99 > 500ms liên tục 5 phút") thay vì trên từng điểm dữ liệu.

## 7. Production Applications

**Prometheus histogram + `histogram_quantile`.** Ứng dụng ghi mỗi quan sát vào bucket counter (`http_request_duration_seconds_bucket{le="0.3"}`); server tính `histogram_quantile(0.99, rate(...[5m]))` bằng đúng phép nội suy ở mục 5. Toán nằm ở: chọn biên bucket (quyết định sai số percentile), tính chất cộng được của counter (cho phép aggregate ngang qua hàng nghìn pod), và nội suy tuyến tính trong bucket. Native histogram (bucket theo cấp số nhân, từ Prometheus 2.40) chính là ý tưởng HDR đưa vào chuẩn.

**Grafana/Alertmanager alert trên p99.** Rule kiểu `p99 > SLO trong 5 phút` là quyết định thống kê được mã hóa: percentile (chống mean lừa dối) + cửa sổ thời gian (giảm phương sai của ước lượng — chính là lấy trung bình nhiều mẫu để co nhiễu theo 1/√n).

**A/B testing platform** (nội bộ các công ty lớn, hoặc GrowthBook/Statsig): phân bổ ngẫu nhiên bằng hash(user_id) — randomization chặt biến ẩn; engine tính p-value, confidence interval, và chống peeking bằng sequential testing. Toán của mục 4 chạy trên mọi quyết định sản phẩm.

**Go `benchstat`**: Mann-Whitney U test trên các lần chạy benchmark — kiểm định phi tham số, không giả định phân phối chuẩn (đúng đắn, vì thời gian chạy benchmark cũng lệch phải!).

**Canary analysis** (Kayenta của Netflix/Spinnaker): so sánh phân phối metric giữa canary và baseline bằng kiểm định thống kê thay vì ngưỡng cứng — "error rate canary có *khác một cách có ý nghĩa* so với baseline không?" là một hypothesis test tự động, chạy trước mỗi lần rollout tiếp.

**HDR Histogram / t-digest trong thực địa**: Elasticsearch percentiles aggregation dùng t-digest mặc định, HDR khi cần; Cassandra, Kafka client metrics dùng HDR-style reservoir. Mỗi lần bạn nhìn thấy chữ "p99" trên dashboard của các hệ này, phía sau là một trong hai cấu trúc đó.

## 8. Interview

**System design — thiết kế monitoring/SLO** là nơi thống kê xuất hiện đậm nhất: "Thiết kế hệ thống theo dõi latency cho API 1 triệu RPS." Bộ khung trả lời: (1) vì sao không lưu raw mà lưu histogram (bộ nhớ, merge); (2) chọn bucket quanh ngưỡng SLO; (3) alert trên percentile qua cửa sổ thời gian, không trên mean; (4) nhắc tail amplification nếu có fan-out — nói được con số 1 − 0.99¹⁰⁰ ≈ 63% và pattern hedged request là tín hiệu senior rõ rệt.

**Median from Data Stream** (LeetCode 295): giữ hai heap — max-heap cho nửa dưới, min-heap cho nửa trên, cân bằng kích thước sau mỗi lần thêm; median là top của heap (chương 16). Insert O(log n), query O(1). Câu hỏi mở rộng nhà tuyển dụng thích: "nếu chỉ cần *xấp xỉ* p50 trên hàng tỷ phần tử?" — chuyển sang histogram/t-digest, đổi chính xác lấy bộ nhớ. Trả lời được cả hai tầng cho thấy bạn phân biệt bài toán phỏng vấn và bài toán production.

**Random Pick with Weight** (LeetCode 528): prefix sum + binary search trên giá trị ngẫu nhiên — chính là lấy mẫu từ phân phối rời rạc bằng CDF nghịch đảo. Liên hệ thẳng tới weighted load balancing.

**Câu behavioral-technical "bạn benchmark thế nào cho đúng?"** — nghe mềm nhưng là bẫy phân loại. Câu trả lời điểm cao: chạy nhiều lần (`-count=10`), máy tĩnh (tắt turbo/background), so sánh bằng benchstat và chỉ tin khi p nhỏ, cảnh giác với micro-benchmark bị compiler optimize mất (dùng `b.Loop()` hoặc sink kết quả), và đo cả allocation chứ không riêng ns/op. Kể một lần bạn *tưởng* nhanh hơn nhưng benchstat in dấu `~` — đó là câu chuyện đáng giá hơn mọi lý thuyết.

**Lỗi tư duy thường gặp:** tin một lần chạy benchmark; nói "p-value là xác suất giả thuyết đúng"; đề xuất alert mean ± 3σ cho latency (nhầm phân phối); quên rằng percentile không cộng được khi aggregate nhiều máy; và trong bài hai heap — quên xử lý trường hợp n chẵn/lẻ khi cân bằng heap.

## 9. Anti-pattern

**Dashboard chỉ có mean.** Chính là câu chuyện mở đầu. Mọi dashboard latency phải có tối thiểu p50/p95/p99 — mean chỉ để đối chiếu độ lệch (mean ≫ p50 nghĩa là đuôi đang phình).

**Tính "p99 trung bình" hoặc cộng p99.** `avg(p99)` qua các instance là phép toán không có ý nghĩa thống kê; p99 của chuỗi hai service không phải tổng hai p99. Aggregate đúng: cộng histogram rồi mới tính percentile.

**Alert mean ± 3σ trên latency.** Quy tắc 3-sigma chỉ đúng cho phân phối chuẩn; latency lệch phải và đuôi dày làm ngưỡng vừa nhảy loạn vừa mù đuôi. Đo percentile trực tiếp.

**Peeking A/B test.** Nhìn kết quả mỗi sáng, dừng khi "đã significant" — false positive phi mã như đã tính ở mục 4. Cùng họ tội: chạy 20 metric và báo cáo metric duy nhất "có ý nghĩa" (multiple comparisons — 1 − 0.95²⁰ ≈ 64% có ít nhất một false positive).

**Suy nhân quả từ tương quan trên dashboard.** "Memory tăng cùng lúc error tăng → memory gây lỗi" — có thể cả hai cùng do traffic. Khi chưa can thiệp có kiểm soát (canary, rollback thử), tương quan chỉ là *gợi ý điều tra*, không phải kết luận.

**Benchmark trên laptop rồi kết luận cho server.** Thermal throttling, CPU khác thế hệ, noisy neighbor — mẫu rút từ phân phối khác thì kết luận không chuyển giao. Benchmark so sánh A/B phải cùng máy, cùng phiên, xen kẽ.

## 10. Best Practices

**Nên:**

- Nhìn phân phối trước, con số tóm tắt sau: một histogram nói thật hơn mười con số. Nghi ngờ ngay khi mean và median tách xa nhau.
- Định nghĩa SLO bằng percentile gắn với trải nghiệm ("99% request < 300ms trong cửa sổ 30 ngày") và đặt biên bucket Prometheus trùng ngưỡng SLO.
- Với fan-out lớn: đo và quản trị p99 của *từng* dependency; cân nhắc hedged requests khi đuôi là vấn đề, và nhớ chi phí của nó (traffic thêm, yêu cầu idempotency).
- Benchmark: `-count=10` trở lên + `benchstat`; A/B test: cố định cỡ mẫu hoặc sequential testing; mọi so sánh phải kèm thanh sai số.
- Khi nghi ngờ nhân quả: can thiệp có kiểm soát (canary với phân bổ ngẫu nhiên) thay vì hồi tưởng trên dashboard.

**Không nên:**

- Không cam kết SLA bằng mean; không aggregate percentile bằng phép trung bình.
- Không dùng mô hình phân phối chuẩn cho đại lượng lệch phải bị chặn dưới (latency, kích thước file, duration).
- Không kết luận từ một lần đo, và không dừng thí nghiệm tại thời điểm kết quả "đẹp".
- Không nhầm "có ý nghĩa thống kê" với "đáng để làm" — luôn hỏi kèm effect size.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Vì sao fan-out 100 service biến p99 của component thành xấp xỉ median của hệ thống, và hedged request phá vỡ phép tính đó ở điểm nào?
2. Prometheus lưu histogram thay vì lưu thẳng giá trị p99 đã tính — điều gì sẽ hỏng nếu làm ngược lại khi aggregate 1.000 pod?
3. Một A/B test đạt p = 0.04 sau khi bạn kiểm tra kết quả mỗi ngày trong 3 tuần và dừng hôm nay. Con số 0.04 đó còn mang ý nghĩa gì không, và vì sao?

---

*Chương tiếp theo: [19 — Number Theory](/series/math-for-engineers/level-4-advanced-mathematics/19-number-theory/), rời thế giới liên tục của dữ liệu đo đạc để về thế giới rời rạc tuyệt đối của số nguyên — nơi overflow không phải là bug mà là số học modular, và các số nguyên tố canh giữ toàn bộ mật mã hiện đại.*
