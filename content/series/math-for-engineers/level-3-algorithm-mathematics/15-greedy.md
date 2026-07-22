+++
title = "Chương 15 — Greedy Algorithms: Khi nào tham lam là đúng"
date = "2026-07-20T09:30:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 3 – Algorithm Mathematics
> Yêu cầu trước: Chương 10 (Complexity Analysis), Chương 14 (Dynamic Programming)

---

## 1. Problem Statement

Một team platform triển khai autoscaler tự viết: mỗi khi tải tăng, chọn ngay hành động rẻ nhất *tại thời điểm đó* — thêm một pod vào node đang trống nhất. Từng quyết định riêng lẻ đều hợp lý. Ba tháng sau, cluster rơi vào trạng thái kỳ dị: pod rải vụn khắp nơi, không node nào trống đủ để nhận workload lớn, và chi phí cao hơn 40% so với phương án xếp lại từ đầu. Không quyết định nào sai — nhưng **chuỗi quyết định** thì sai.

Đây là hình dạng tổng quát của một cạm bẫy tư duy: *tối ưu cục bộ tại từng bước có dẫn đến tối ưu toàn cục không?* Chiến lược "tại mỗi bước, chọn phương án tốt nhất trước mắt và không bao giờ quay đầu" gọi là **greedy** (tham lam). Nó hấp dẫn chết người: code ngắn, chạy nhanh, và *nghe* rất thuyết phục. Vấn đề là nó **thường sai** — và tệ hơn, sai một cách kín đáo: cho đáp án hợp lệ nhưng không tối ưu, qua được test nhỏ, chỉ lộ mặt trên dữ liệu thật.

Nhưng greedy không phải luôn sai. Dijkstra là greedy. Kruskal là greedy. Huffman coding là greedy. Và cả ba đều **đúng tuyệt đối, có chứng minh**. Vậy câu hỏi trung tâm của chương này:

**Điều gì phân biệt bài toán mà greedy cho lời giải tối ưu với bài toán mà greedy thất bại — và làm sao chứng minh mình đang ở phía nào?**

Nếu không có câu trả lời, bạn chỉ còn hai lựa chọn tồi: không bao giờ dám dùng greedy (bỏ phí những thuật toán nhanh nhất, đẹp nhất của ngành), hoặc dùng theo cảm tính (và trở thành tác giả của autoscaler ở trên).

## 2. Trực giác

### Leo núi trong sương mù

Hình dung bạn leo núi trong sương mù dày, chỉ nhìn được một mét quanh chân. Chiến lược greedy: luôn bước về hướng dốc lên cao nhất. Với một quả đồi trơn duy nhất, chiến lược này đưa bạn lên đỉnh. Nhưng với dãy núi nhiều đỉnh, nó đưa bạn lên **đỉnh đồi gần nhất** — có thể chỉ là một mô đất, trong khi đỉnh núi thật nằm bên kia thung lũng mà bạn không bao giờ chịu bước xuống để băng qua.

Đây là toàn bộ câu chuyện greedy trong một hình ảnh: greedy đúng khi và chỉ khi **địa hình của bài toán không có "mô đất giả"** — khi tốt-nhất-cục-bộ nằm trên đường đi đến tốt-nhất-toàn-cục. May mắn là, khác với người leo núi, ta có toán để *chứng minh* địa hình một bài toán có mô đất giả hay không, thay vì phải leo thử.

### Phản ví dụ 60 giây: coin change

Bài toán đổi tiền: hệ xu {1, 3, 4}, đổi số tiền 6 với ít xu nhất. Greedy tự nhiên: luôn lấy xu lớn nhất còn dùng được.

```
Greedy:  6 → lấy 4 (còn 2) → lấy 1 (còn 1) → lấy 1 (còn 0)   = 3 xu: 4+1+1
Tối ưu:  6 = 3 + 3                                            = 2 xu
```

Chuyện gì đã xảy ra? Việc lấy xu 4 — quyết định tốt nhất trước mắt — đẩy bài toán con còn lại (đổi 2) vào thế xấu: 2 chỉ đổi được bằng 1+1. Xu 3 "kém hơn" tại chỗ nhưng để lại bài toán con đẹp hơn (đổi 3 bằng đúng một xu). **Greedy mù với hậu quả**: nó đánh giá lựa chọn bằng giá trị tức thời, không bằng chất lượng của phần còn lại.

Điều thú vị: với hệ xu {1, 5, 10, 25} (tiền Mỹ) greedy lại **đúng**. Tính đúng/sai của greedy không nằm ở thuật toán — nằm ở **cấu trúc của dữ liệu bài toán**. Cùng một chiến lược, hệ xu này đúng, hệ xu kia sai. Đây là lý do greedy phải được chứng minh *cho từng bài toán cụ thể*, không có "greedy đúng nói chung".

### Vậy khi nào tham lam là an toàn?

Trực giác trước khi vào công thức: greedy an toàn khi lựa chọn tham lam **không bao giờ đóng sập cánh cửa nào mà lời giải tối ưu cần**. Ở coin change {1,3,4}, lấy xu 4 đóng sập cánh cửa "6 = 3+3". Ở bài xếp lịch họp (sẽ chứng minh ở mục 4), chọn cuộc họp kết thúc sớm nhất không đóng cánh cửa nào cả — mọi lời giải tốt đều có thể "sửa" để bắt đầu bằng lựa chọn đó mà không xấu đi. Khả năng "sửa lời giải tối ưu cho khớp với lựa chọn greedy" chính là linh hồn của kỹ thuật chứng minh exchange argument.

## 3. First Principles

### Hai tính chất bắt buộc

Một bài toán tối ưu giải được bằng greedy khi và chỉ khi nó có đủ hai tính chất (CLRS gọi đây là "greedy strategy ingredients"):

**1. Greedy-choice property**: tồn tại một lời giải tối ưu toàn cục *chứa lựa chọn tham lam đầu tiên*. Nói cách khác: bước tham lam đầu tiên không bao giờ là bước sai — có thể có nhiều lời giải tối ưu, nhưng ít nhất một trong số đó tương thích với lựa chọn greedy.

**2. Optimal substructure**: sau khi thực hiện lựa chọn tham lam, phần còn lại là bài toán con cùng dạng, và lời-giải-tối-ưu-của-bài-lớn = lựa-chọn-greedy + lời-giải-tối-ưu-của-bài-con. Tính chất này quen thuộc — nó chính là nền của DP (chương 14).

Ghép hai tính chất lại, phép quy nạp (chương 03) hoàn tất chứng minh: bước 1 đúng (greedy-choice), phần còn lại là bài cùng dạng nhỏ hơn (optimal substructure), nên bước 2 đúng, bước 3 đúng... đến hết. Coin change {1,3,4} có optimal substructure (vì thế DP giải được) nhưng **thiếu greedy-choice property**: không lời giải tối ưu nào của "đổi 6" chứa xu 4.

### Greedy là DP thoái hóa

Cách nhìn sâu nhất về greedy đến từ chương 14. DP tại mỗi state phải xét **mọi** lựa chọn rồi lấy min/max, vì không biết trước lựa chọn nào dẫn đến tối ưu:

```
DP:      dp[state] = best( choice₁ + dp[state₁], choice₂ + dp[state₂], ... )
                     ─────── phải tính TẤT CẢ các nhánh ───────

Greedy:  dp[state] = choiceₖ + dp[stateₖ]
                     ─ có ĐỊNH LÝ bảo đảm nhánh k luôn thắng ─
```

**Greedy là DP mà tại mỗi state, một định lý cho phép ta biết trước nhánh thắng cuộc mà không cần tính các nhánh kia.** Hệ quả trực tiếp: cây bài toán con sụp xuống thành một đường thẳng — độ phức tạp rơi từ "số state × số lựa chọn" xuống "số bước", thường là từ O(n²) hay O(nW) xuống O(n log n) (log là chi phí sort hoặc heap để tìm lựa chọn tham lam nhanh).

Cách nhìn này trả lời luôn câu "nếu không có greedy thì sao?": thì vẫn giải được bằng DP hoặc duyệt toàn bộ — greedy không mở rộng tập bài giải được, nó là **phần thưởng tốc độ** cho những bài toán có cấu trúc đặc biệt. Và nó cho quy trình làm việc đúng: *nghĩ DP trước cho chắc đúng, rồi tự hỏi "có định lý nào cho phép bỏ qua các nhánh không?"* — chứ không phải nghĩ greedy trước rồi cầu nguyện.

### Matroid — một đoạn cho người tò mò

Tồn tại một lý thuyết đại số gọi tên chính xác họ bài toán mà greedy luôn đúng: **matroid**. Trực giác: một hệ gồm các "tập độc lập" (ví dụ: các tập cạnh không tạo chu trình trong đồ thị) thỏa tính chất trao đổi — nếu tập độc lập A nhỏ hơn tập độc lập B, luôn lấy được một phần tử của B thêm vào A mà A vẫn độc lập. Định lý Rado–Edmonds: trên một matroid với trọng số bất kỳ, greedy "lấy phần tử tốt nhất còn thêm được" **luôn** cho tập độc lập tối đa có tổng trọng số tối ưu. Kruskal đúng vì các rừng (forest) của đồ thị lập thành matroid; 0/1 knapsack sai vì các tập đồ vật nhét vừa balo không lập thành matroid (thiếu tính chất trao đổi). Với công việc kỹ sư, chỉ cần nhớ: **có một ranh giới đại số thật sự**, không phải may rủi — còn chứng minh từng bài cụ thể thì exchange argument (mục 4) là đủ và thực dụng hơn.

## 4. Mathematical Model

### Bài toán mẫu: Activity Selection

Cho n hoạt động, hoạt động i có thời gian [sᵢ, fᵢ). Chọn nhiều hoạt động nhất sao cho không hai hoạt động nào chồng lấn. (Phiên bản phỏng vấn: một phòng họp, nhét được nhiều cuộc họp nhất.)

Các chiến lược greedy ứng viên — và số phận của chúng:

| Chiến lược "chọn trước hoạt động..." | Phản ví dụ |
|---|---|
| bắt đầu sớm nhất | một hoạt động dài từ sáng đến tối, bắt đầu sớm nhất, giết mọi hoạt động khác |
| ngắn nhất | hoạt động ngắn nằm vắt ngang chỗ nối của hai hoạt động dài không chồng nhau |
| ít xung đột nhất | dựng được phản ví dụ 11 hoạt động (bài tập kinh điển của CLRS) |
| **kết thúc sớm nhất** | không tồn tại phản ví dụ — và ta sẽ chứng minh |

Riêng việc ba chiến lược "nghe hợp lý" đầu tiên đều sai đã là bài học: với greedy, *nghe hợp lý* không có giá trị pháp lý.

### Chứng minh bằng Exchange Argument

**Định lý.** Chiến lược "luôn chọn hoạt động kết thúc sớm nhất trong các hoạt động tương thích còn lại" cho lời giải tối ưu.

**Chứng minh greedy-choice property.** Gọi g là hoạt động kết thúc sớm nhất trong toàn bộ n hoạt động. Ta chứng minh: tồn tại lời giải tối ưu chứa g.

Lấy một lời giải tối ưu bất kỳ OPT, sắp các hoạt động của nó theo thời gian kết thúc: a₁, a₂, ..., aₖ. Nếu a₁ = g, xong. Nếu không, thực hiện **phép trao đổi**: thay a₁ bằng g, được OPT′ = {g, a₂, ..., aₖ}. Cần kiểm tra OPT′ hợp lệ:

- g kết thúc sớm nhất toàn cục nên f(g) ≤ f(a₁);
- a₂ tương thích với a₁ nghĩa là s(a₂) ≥ f(a₁) ≥ f(g), vậy a₂ (và mọi hoạt động sau) cũng tương thích với g.

```
 a₁:   ├────────┤                        OPT  = {a₁, a₂, a₃}
 g :  ├─────┤          ↑ f(g) ≤ f(a₁)
 a₂:              ├───┤                  OPT′ = {g,  a₂, a₃}  — vẫn k hoạt động,
 a₃:                     ├────┤                  vẫn hợp lệ  ⇒ vẫn tối ưu
```

OPT′ hợp lệ và có đúng k hoạt động — bằng OPT — nên OPT′ cũng tối ưu, và nó chứa g. ∎

**Chứng minh optimal substructure.** Sau khi chọn g, bài còn lại là "chọn nhiều hoạt động nhất trong tập S′ = {i : sᵢ ≥ f(g)}" — cùng dạng bài gốc. Nếu phần lời giải trên S′ không tối ưu cho S′, thay nó bằng lời giải tốt hơn của S′ sẽ cải thiện lời giải toàn cục (cut-and-paste — đúng lập luận của chương 14), mâu thuẫn với tính tối ưu. ∎

Quy nạp trên số bước ghép hai bổ đề thành định lý đầy đủ.

**Khuôn mẫu để tái sử dụng** — mọi exchange argument đều có bộ xương này:

1. Lấy lời giải tối ưu OPT bất kỳ, *không giả định gì thêm* về nó.
2. Nếu OPT đã khớp lựa chọn greedy → xong. Nếu chưa, chỉ ra phép **trao đổi** một phần tử của OPT bằng lựa chọn greedy.
3. Chứng minh sau trao đổi: lời giải **vẫn hợp lệ** và **không xấu đi**.
4. Kết luận: tồn tại lời giải tối ưu chứa lựa chọn greedy; quy nạp phần còn lại.

Chứng minh tính đúng của Dijkstra và Kruskal (chương 08) cũng theo đúng khuôn này — với Kruskal, phép trao đổi chạy trên chu trình tạo bởi cạnh thêm vào; với MST nói chung, nó được đóng gói thành **cut property**: cạnh nhẹ nhất vắt qua một lát cắt luôn thuộc một MST nào đó. "Cạnh nhẹ nhất qua lát cắt" chính là "hoạt động kết thúc sớm nhất" trong thế giới đồ thị — cùng một ý tưởng, hai bộ trang phục.

### Ranh giới đẹp: fractional vs 0/1 knapsack

Không đâu ranh giới đúng/sai của greedy sắc nét bằng bài balo. Cho các đồ vật (giá trị vᵢ, khối lượng wᵢ), balo chịu tối đa W.

**Fractional knapsack** (được cắt đồ vật): greedy theo mật độ giá trị vᵢ/wᵢ giảm dần, lấy hết mức có thể, vật cuối cắt phần vừa khít — **tối ưu, có chứng minh** bằng exchange argument (hoán đổi một đơn vị khối lượng của vật mật độ thấp trong OPT bằng một đơn vị của vật mật độ cao hơn: hợp lệ vì khối lượng không đổi, không xấu đi vì mật độ cao hơn).

**0/1 knapsack** (lấy nguyên hoặc bỏ): cùng chiến lược, **sai**. Phản ví dụ kinh điển của CLRS: W = 50; vật A (v=60, w=10, mật độ 6.0), vật B (v=100, w=20, mật độ 5.0), vật C (v=120, w=30, mật độ 4.0). Greedy theo mật độ lấy A rồi B → còn 20kg, không nhét nổi C → tổng 160. Tối ưu: B + C = 220, không thèm đụng vào vật có mật độ cao nhất. (Bản fractional của cùng dữ liệu: lấy A, B, và 2/3 của C = 240 — greedy đúng.) Vì sao phép trao đổi gãy ở bản 0/1? Vì **không cắt được**: đổi "một phần" vật A lấy "một phần" vật B là nước đi bất hợp pháp, bước 3 của khuôn mẫu (lời giải sau trao đổi vẫn hợp lệ) sụp đổ. Tính chia được (divisibility) chính là thứ làm bài toán "trơn", không còn mô đất giả; bỏ nó đi, bài toán thành NP-hard và phải quay về DP O(nW) của chương 14. **Một ràng buộc nguyên (integrality) nhỏ xíu tách hai thế giới: greedy O(n log n) và NP-hard.**

## 5. Thuật toán

Activity selection, bản đầy đủ:

```go
type Activity struct {
    Start, Finish int
}

// SelectActivities chọn số hoạt động không chồng lấn tối đa.
// Chiến lược greedy: luôn chọn hoạt động kết thúc sớm nhất còn tương thích
// (tính đúng đã chứng minh bằng exchange argument ở mục 4).
func SelectActivities(acts []Activity) []Activity {
    if len(acts) == 0 {
        return nil
    }
    // Bước 1: sort theo thời gian kết thúc — O(n log n).
    // Đây là chi phí chính; phần greedy phía sau chỉ O(n).
    sort.Slice(acts, func(i, j int) bool {
        return acts[i].Finish < acts[j].Finish
    })

    selected := []Activity{acts[0]}   // hoạt động kết thúc sớm nhất toàn cục
    lastFinish := acts[0].Finish

    // Bước 2: quét một lượt, mỗi hoạt động quyết định đúng MỘT lần,
    // không bao giờ xét lại — dấu hiệu nhận dạng của mọi thuật toán greedy.
    for _, a := range acts[1:] {
        if a.Start >= lastFinish {    // tương thích với lựa chọn gần nhất
            selected = append(selected, a)
            lastFinish = a.Finish
        }
    }
    return selected
}
```

Cấu trúc hai pha này — **sort (hoặc heap) theo tiêu chí greedy, rồi quét tuyến tính** — là bộ xương của đa số thuật toán greedy: Kruskal (sort cạnh theo trọng số), Huffman (min-heap theo tần suất), fractional knapsack (sort theo mật độ), Dijkstra (min-heap theo khoảng cách tạm thời). Vì vậy độ phức tạp "mặc định" của greedy là **O(n log n)** — và khi bạn phân tích một lời giải greedy trong phỏng vấn, hầu như luôn là "sort chiếm chủ đạo".

Fractional knapsack theo cùng bộ xương — sort theo tiêu chí greedy rồi quét, cộng thêm một bước "cắt" ở cuối:

```go
type Item struct {
    Value, Weight float64
}

// FractionalKnapsack: tối ưu có chứng minh NHỜ được phép cắt đồ vật.
// Xóa dòng "cắt phần vừa khít" đi là thành 0/1 knapsack — và greedy này sai ngay.
func FractionalKnapsack(items []Item, capacity float64) float64 {
    // Tiêu chí greedy: mật độ giá trị value/weight giảm dần
    sort.Slice(items, func(i, j int) bool {
        return items[i].Value/items[i].Weight > items[j].Value/items[j].Weight
    })

    total := 0.0
    for _, it := range items {
        if capacity <= 0 {
            break
        }
        take := math.Min(it.Weight, capacity) // vật cuối: cắt phần vừa khít
        total += it.Value * (take / it.Weight)
        capacity -= take
    }
    return total
}
```

Hai thành viên kinh điển khác của gia đình, để định vị trên bản đồ:

- **Huffman coding** (chi tiết ở chương 20 — Information Theory): lặp lại "gộp hai nút tần suất nhỏ nhất". Exchange argument: trong cây mã tối ưu, luôn sửa được để hai ký tự hiếm nhất là anh em ở tầng sâu nhất — hoán đổi vị trí lá không làm tăng độ dài mã kỳ vọng.
- **Dijkstra / Kruskal** (chương 08): greedy trên đồ thị, chứng minh dựa trên cut property. Đáng nhớ: Dijkstra *mất hiệu lực* với cạnh âm — vì "đỉnh gần nhất hiện tại" không còn chắc chắn là chung cuộc, tức greedy-choice property gãy. Cùng một bài học: greedy đúng nhờ một tiền đề về dữ liệu, rút tiền đề là sập.

## 6. Trade-off

**Tốc độ đổi lấy phạm vi áp dụng.** Greedy thường O(n log n) và bộ nhớ O(1)–O(n); DP tương ứng thường O(n²) hay O(nW) kèm bảng nhớ. Nhưng greedy chỉ đúng trên họ bài toán hẹp có greedy-choice property. Chọn greedy là đánh đổi: hiệu năng tối đa, kèm nghĩa vụ **chứng minh** — không chứng minh được thì hiệu năng đó là hiệu năng của một đáp án sai.

**Tối ưu tuyệt đối vs xấp xỉ nhanh.** Với bài NP-hard, greedy thường được dùng *có chủ đích* làm thuật toán xấp xỉ với cận đã chứng minh: greedy cho set cover đạt tỷ lệ ln(n), cho bin packing (first-fit decreasing) đạt 11/9·OPT + hằng số. Đây là cách dùng greedy trưởng thành nhất: **biết nó không tối ưu, và định lượng được khoảng cách**. Tệ nhất không phải là dùng greedy suboptimal — là dùng mà tưởng nó tối ưu.

**Online vs offline.** Nhiều hệ thống *buộc phải* greedy vì quyết định phải đưa ra ngay khi dữ liệu đến, không được nhìn tương lai (online problem): load balancer không thể chờ biết hết request của ngày hôm nay rồi mới phân phối. Ở thế giới online, câu hỏi đổi từ "greedy có tối ưu không?" (thường là không) sang "competitive ratio là bao nhiêu so với đối thủ biết trước tương lai?". Nhận ra bài toán của mình là online giúp ngừng dằn vặt vì không tối ưu — tối ưu offline là mục tiêu sai.

**Khi nào KHÔNG nên dùng greedy?** Khi không chứng minh được và cũng không tìm được counter-example sau nỗ lực nghiêm túc → dùng DP nếu kích thước cho phép: chậm hơn nhưng đúng chắc chắn. Khi các quyết định tương tác xa nhau qua ràng buộc toàn cục (tổng ngân sách, capacity nguyên) — đó là mùi của DP/flow/ILP. Và khi bài toán nhỏ: brute force đúng tuyệt đối, dễ review, là lựa chọn *khiêm tốn đúng đắn* cho n ≤ 20.

## 7. Production Applications

**TCP congestion control — AIMD.** Mỗi kết nối TCP tham lam tăng tuyến tính cửa sổ gửi (additive increase) đến khi mất gói thì cắt nửa (multiplicative decrease). Mỗi endpoint chỉ nhìn tín hiệu cục bộ của chính nó — không ai có bức tranh toàn mạng — vậy mà có chứng minh (Chiu–Jain, 1989): AIMD hội tụ về điểm chia sẻ băng thông công bằng giữa các luồng. Đây là greedy ở dạng đẹp nhất: quyết định cục bộ, kèm định lý về hành vi toàn cục. Ngược lại, "greedy thô" — gửi hết tốc độ khi thấy mạng trống — chính là nguyên nhân congestion collapse của Internet năm 1986, sự cố khai sinh ra AIMD.

**Kubernetes scheduler.** Đặt pod là bin packing — NP-hard — nên scheduler dùng greedy: filter node khả thi, chấm điểm, **chọn node điểm cao nhất, không quay đầu**. Hệ quả đúng như lý thuyết dự báo: quyết định tốt-tại-thời-điểm tích lũy thành phân mảnh tài nguyên (câu chuyện mở chương). Kubernetes thừa nhận điều này bằng kiến trúc: **descheduler** — thành phần định kỳ đuổi pod khỏi node để xếp lại — chính là "cho phép quay đầu" được vá vào một thuật toán vốn không quay đầu. Khi đọc thiết kế này, bạn đang đọc một trade-off của mục 6 được viết thành YAML.

**Huffman trong zstd/gzip/DEFLATE.** Mỗi lần bạn tải một trang web (Content-Encoding: gzip) hay đọc dữ liệu từ một hệ lưu trữ nén bằng zstd, một cây Huffman — greedy có chứng minh tối ưu cho mã tiền tố từng ký hiệu — được dựng hoặc tra cứu. Chi tiết và mối nối với entropy ở chương 20; điểm cần nhớ ở đây: một thuật toán greedy 70 tuổi đang chạy hàng tỷ lần mỗi giây trên hạ tầng Internet, *vì nó có chứng minh*.

**Load balancing least-connections.** HAProxy/Envoy với chính sách least-connections: gửi request mới vào backend đang có ít kết nối nhất — greedy trên tín hiệu tức thời. Hoạt động tốt phần lớn thời gian, và thất bại theo đúng kịch bản sách giáo khoa: backend vừa khởi động (0 kết nối vì... vừa khởi động, cache lạnh) hút trọn dòng request và sập — hiện tượng thundering herd lên node mới. Bản vá công nghiệp là *slow start* (tăng dần trọng số node mới): một lần nữa, nhận diện chỗ greedy-choice property gãy (tín hiệu cục bộ "ít kết nối" không phản ánh chi phí thật) rồi vá đúng chỗ đó.

Mẫu số chung của cả bốn: production dùng greedy vì **tốc độ và tính cục bộ của quyết định là ràng buộc cứng**, rồi bù đắp phần thiếu tối ưu bằng cơ chế sửa sai định kỳ (descheduler, slow start, rebalancing). Nhận ra cặp "greedy + bộ sửa sai" giúp bạn đọc hiểu kiến trúc của rất nhiều hệ thống.

## 8. Interview

Greedy là chủ đề phỏng vấn *nguy hiểm* nhất — không vì khó code (code thường 10 dòng) mà vì **quyết định dùng nó là một tuyên bố toán học**. Kỹ năng số một: **nghi ngờ trước, tin sau**. Khi lóe lên ý tưởng greedy, dành đúng 60 giây săn counter-example trước khi viết dòng code nào: thử input nhỏ (n = 2, 3), thử phần tử cực đoan (rất lớn, rất nhỏ, bằng nhau), thử chỗ nối biên. Tìm ra phản ví dụ trong 60 giây rẻ hơn vô hạn so với interviewer tìm ra nó sau 20 phút — và câu nói "để em thử phá chiến lược này đã" tự nó đã là điểm cộng lớn.

**Các bài kinh điển và bài học từng bài:**

- **Jump Game (LC 55)**: duy trì "tầm với xa nhất" khi quét — greedy đúng, và đáng suy ngẫm *vì sao*: các tầm với lồng nhau, với xa hơn không bao giờ hại (không có cánh cửa nào bị đóng). So sánh với DP O(n²) tự nhiên để thấy "greedy = DP bỏ được các nhánh".
- **Gas Station (LC 134)**: nếu tổng gas ≥ tổng cost thì tồn tại điểm xuất phát; greedy: cạn xăng ở đâu, khởi điểm mới là trạm kế đó. Bài học: tính đúng dựa trên một **bổ đề phải phát biểu được** ("mọi trạm trong đoạn vừa thất bại đều thất bại") — nói được bổ đề là qua bài, code chỉ là thủ tục.
- **Meeting Rooms II (LC 253)**: số phòng tối thiểu = số cuộc họp chồng lấn tối đa tại một thời điểm — sort mốc thời gian hoặc min-heap theo thời gian kết thúc. Anh em ruột của activity selection.
- **Non-overlapping Intervals (LC 435)**: "xóa ít interval nhất để hết chồng lấn" = "giữ nhiều interval nhất không chồng lấn" — *chính là* activity selection đội mũ khác. Nhận ra phép quy về này là kỹ năng mô hình hóa của chương 04.
- **Task Scheduler (LC 621)**: xếp task có cooldown — greedy theo tần suất cao nhất, hoặc công thức đếm khe trực tiếp. Bài học: một số bài "greedy" thật ra giải bằng **đếm** (chương 05) gọn hơn mô phỏng.

**Lỗi tư duy hay gặp:** dùng greedy vì "bài trước giống giống cũng greedy" — coin change {1,5,10,25} đúng không cứu được {1,3,4}, sự giống nhau bề mặt không di truyền tính đúng. Tin một chiến lược vì qua hết example của đề — example của đề *được thiết kế để không tiết lộ* góc chết. Sort theo tiêu chí đầu tiên nghĩ ra — bảng bốn chiến lược ở mục 4 cho thấy ba tiêu chí hợp lý đầu tiên đều sai. Và ngược lại: dùng DP nặng nề cho bài có greedy đúng hiển nhiên (Jump Game bằng DP O(n²) sẽ TLE — nghi ngờ greedy không có nghĩa là cấm nó).

**Cách phân tích thay vì học thuộc**, tuần tự: (1) đây có phải bài tối ưu hóa trên chuỗi quyết định không; (2) đề xuất tiêu chí greedy, *săn counter-example 60 giây*; (3) sống sót → phác exchange argument một câu ("mọi lời giải tối ưu sửa được để bắt đầu bằng lựa chọn của em vì..."); (4) không phác nổi → rơi về DP. Trong phỏng vấn, nói to bốn bước này ra — quy trình tư duy đúng đáng giá hơn đáp án đúng.

**Từ phỏng vấn sang production:** Meeting Rooms II là bài toán cấp phát connection pool / thread pool nguyên bản; Non-overlapping Intervals là chọn job không xung đột trên một tài nguyên độc quyền; Gas Station là kiểm tra tính khả thi của một chu trình tiêu thụ tài nguyên. Interval scheduling nói chung có mặt ở mọi nơi có "một tài nguyên, nhiều yêu cầu theo thời gian".

## 9. Anti-pattern

**Greedy không chứng minh, không phản ví dụ, chỉ có niềm tin.** Anti-pattern số một, đã đủ nói ở trên. Dạng nhận biết trong code review: PR mô tả "chọn X tốt nhất tại mỗi bước" mà phần *why* trống trơn. Câu hỏi review đúng: "phản ví dụ đâu, hoặc phác chứng minh đâu?"

**Kiểm chứng bằng test thay vì bằng toán.** Greedy sai vẫn qua 95% test — nó cho đáp án *hợp lệ*, chỉ không *tối ưu*, mà test thường chỉ kiểm tra hợp lệ. Nếu buộc phải kiểm bằng thực nghiệm, cách đúng là **so với oracle**: chạy brute force trên hàng nghìn input nhỏ ngẫu nhiên và so đáp án. Khuôn mẫu bằng Go:

```go
// TestGreedyAgainstOracle: săn counter-example tự động.
// Trên input nhỏ, brute force là "sự thật"; greedy phải khớp nó tuyệt đối.
func TestGreedyAgainstOracle(t *testing.T) {
    rng := rand.New(rand.NewSource(42)) // seed cố định để tái lập lỗi
    for i := 0; i < 10000; i++ {
        coins := randomCoinSystem(rng) // hệ xu ngẫu nhiên, luôn chứa xu 1
        amount := rng.Intn(50) + 1

        want := bruteForceMinCoins(coins, amount) // đúng chắc chắn, O(2ⁿ) — chấp nhận vì input nhỏ
        got := greedyMinCoins(coins, amount)

        if got != want {
            // In ra counter-example NHỎ NHẤT tìm được — món quà cho việc debug
            t.Fatalf("counter-example: hệ xu %v, số tiền %d: greedy=%d, tối ưu=%d",
                coins, amount, got, want)
        }
    }
}
```

Chạy test này với hệ xu {1,3,4} sẽ in ra đúng phản ví dụ ở mục 2 trong vài mili giây. Vẫn không phải chứng minh — 10.000 input ngẫu nhiên không phủ nổi không gian vô hạn — nhưng hơn xa "pass CI với example của đề", và là kỹ thuật đáng dùng cho *mọi* thuật toán tối ưu hóa bạn tự viết.

**Tối ưu cục bộ theo metric cục bộ trong hệ thống.** Phiên bản kiến trúc của greedy sai: mỗi service tự tối ưu latency của mình bằng cache/retry hung hãn — cục bộ đẹp, toàn cục sập vì retry storm khuếch đại sự cố. Quyết định cục bộ tốt + tương tác toàn cục = không có greedy-choice property; hệ phân tán đầy những bài toán như vậy, và "mỗi team tự tối ưu KPI của mình" là thuật toán greedy chạy trên tổ chức.

**Nhầm heuristic với thuật toán tối ưu.** Dùng greedy cho bài NP-hard là hợp lệ — miễn gọi đúng tên nó trong design doc: "heuristic, không đảm bảo tối ưu, cận xấp xỉ (nếu biết) là X". Đặt tên sai làm người vận hành sau này tin nhầm rằng output đã là tốt nhất có thể, và bỏ qua cả một chiều cải tiến.

**Không có cơ chế sửa sai đi kèm.** Đã chọn greedy suboptimal cho hệ thống dài hạn (scheduler, allocator) mà không có vòng rebalance định kỳ — phân mảnh và drift sẽ tích lũy vô hạn. Bài học Kubernetes: greedy production-grade luôn đi thành cặp với bộ sửa sai của nó.

## 10. Best Practices

**Nên:**

- Mặc định nghi ngờ: mọi ý tưởng greedy bắt đầu vòng đời ở trạng thái "chưa đáng tin", chỉ thăng cấp sau khi sống sót qua săn counter-example và có phác thảo exchange argument.
- Nghĩ theo DP trước để chắc chắn về state và tính đúng, rồi hạ xuống greedy nếu chứng minh được rằng tại mỗi state chỉ một lựa chọn đáng xét — đúng thứ tự "đúng trước, nhanh sau".
- Khi chứng minh, dùng khuôn exchange argument bốn bước ở mục 4; toàn bộ độ khó luôn dồn vào bước 3 (trao đổi xong vẫn hợp lệ và không xấu đi) — nếu bước 3 trơn tru đáng ngờ, thường là bạn quên một ràng buộc.
- Với bài NP-hard, cứ dùng greedy — nhưng ghi rõ trạng thái heuristic, tìm cận xấp xỉ trong tài liệu, và cân nhắc bộ sửa sai định kỳ.
- Test greedy bằng oracle brute force trên input nhỏ ngẫu nhiên, không chỉ bằng example có sẵn.

**Không nên:**

- Không suy diễn tính đúng qua sự tương tự bề mặt giữa hai bài toán — tính đúng của greedy nằm ở cấu trúc dữ liệu bài toán (hệ xu nào, cạnh âm hay không), không ở dạng phát biểu.
- Không dùng greedy khi các quyết định ràng buộc lẫn nhau qua một ngân sách toàn cục nguyên — đó là lãnh địa của DP/flow.
- Không quên chiều ngược: né greedy đã chứng minh được để "chơi an toàn" bằng DP là trả giá hiệu năng vô ích — Dijkstra bằng Bellman-Ford là một sự lãng phí có tên riêng.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Với hệ xu {1, 3, 4} và số tiền 6, hãy chỉ ra chính xác mệnh đề nào trong cặp (greedy-choice property, optimal substructure) bị vi phạm — và vì sao DP vẫn giải được.
2. Trong chứng minh activity selection, bước nào của exchange argument sẽ gãy nếu đổi tiêu chí thành "chọn hoạt động ngắn nhất"? Hãy dựng phản ví dụ 3 hoạt động minh họa đúng chỗ gãy đó.
3. Vì sao fractional knapsack giải được bằng greedy còn 0/1 knapsack thì không, dù chỉ khác nhau một ràng buộc nguyên? Liên hệ với một quyết định "chia nhỏ được hay không" trong hệ thống bạn đang làm.

---

*Chương tiếp theo: [16 — Sorting, Searching & Heap](/series/math-for-engineers/level-3-algorithm-mathematics/16-sorting-searching-heap/) — chính là bộ máy đứng sau pha "sort theo tiêu chí greedy" của chương này: vì sao Ω(n log n) là bức tường của mọi comparison sort, và heap làm gì để Dijkstra với Huffman chạy nhanh.*
