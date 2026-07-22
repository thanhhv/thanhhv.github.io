+++
title = "Chương 21 — Optimization: Tìm cực trị trong không gian ràng buộc"
date = "2026-07-20T10:30:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 4 – Advanced Mathematics
> Yêu cầu trước: Chương 10 (Complexity), Chương 14 (Dynamic Programming), Chương 15 (Greedy)

---

## 1. Problem Statement

Cluster Kubernetes của bạn có 40 node, mỗi node 16 vCPU và 64GB RAM. Có 300 pod cần chạy, mỗi pod khai báo request CPU/memory khác nhau, một số pod phải cùng node với nhau (affinity), một số tuyệt đối không được cùng node (anti-affinity), vài pod cần GPU. Hóa đơn cloud tính theo số node bật. Câu hỏi của sếp rất gọn: **"Xếp thế nào để bật ít node nhất?"**

Thử liệt kê mọi cách xếp? Mỗi pod có tối đa 40 lựa chọn node: 40³⁰⁰ phương án — nhiều hơn số nguyên tử trong vũ trụ quan sát được lũy thừa vài lần. Vũ trụ không đủ chỗ cho brute force. Thử "cứ nhét pod to trước"? Nghe hợp lý, nhưng hợp lý đến đâu — cách tối ưu 5% hay 50%? Không có khung toán, bạn thậm chí không trả lời được câu hỏi đó.

Bài toán này không đặc biệt. Nhìn lại một vòng công việc của kỹ sư: chọn cấu hình autoscaling để tối thiểu chi phí mà vẫn giữ SLA; query optimizer chọn plan rẻ nhất trong hàng nghìn plan; phân bổ ngân sách ads cho các campaign; tìm route giao hàng ngắn nhất; train model ML để tối thiểu loss. Tất cả đều là **một** bài toán mặc những bộ áo khác nhau:

> Tìm x trong không gian phương án X sao cho f(x) nhỏ nhất (hoặc lớn nhất), thỏa mãn các ràng buộc g(x).

Đây là bài toán tối ưu hóa (optimization). Chương này xây khung tư duy chung: cách phát biểu bài toán, bản đồ các lớp bài toán từ "giải được trọn vẹn" đến "NP-hard, đừng mơ exact", và bộ công cụ tương ứng — Branch and Bound, Linear Programming, gradient descent, heuristics. Kỹ năng quan trọng nhất chương này không phải thuật toán nào cả, mà là **nhận diện mình đang đứng ở ô nào trên bản đồ** — vì dùng công cụ của ô này cho bài của ô kia là nguồn lãng phí kỹ thuật kinh điển.

## 2. Trực giác

### Ba thành phần bất biến

Mọi bài toán tối ưu, từ xếp pod đến train GPT, đều gồm đúng ba mảnh:

1. **Biến quyết định (decision variables)** — thứ bạn được phép chọn. Pod i đặt lên node nào. Trọng số w của model. Số replica lúc 9h sáng.
2. **Hàm mục tiêu (objective function)** — thước đo một con số cho "phương án này tốt cỡ nào". Số node bật. Loss trên tập train. Chi phí USD/tháng.
3. **Ràng buộc (constraints)** — ranh giới giữa phương án hợp lệ và không hợp lệ. Tổng CPU request trên node ≤ 16. Anti-affinity. Latency p99 ≤ 200ms.

Nghe hiển nhiên, nhưng đây là nơi phần lớn thất bại xảy ra — *trước cả khi* thuật toán vào cuộc. "Hệ thống tối ưu chưa?" không phải câu hỏi; "objective là gì, ràng buộc là gì?" mới là câu hỏi. Rất nhiều cuộc họp tranh cãi về "giải pháp tốt nhất" thực chất là hai phe đang cầm hai objective khác nhau (chi phí vs latency) mà không ai nói ra. Viết được ba mảnh trên ra giấy là đã đi được nửa đường — và mục Interview sẽ quay lại đúng ý này.

### Địa hình và người leo núi mù

Hình dung không gian phương án như một địa hình: mỗi điểm là một phương án, độ cao là giá trị objective (ta muốn tìm điểm thấp nhất). Người giải bài toán là kẻ leo núi bị bịt mắt: đứng ở một điểm, chỉ "sờ" được vùng lân cận, không nhìn được toàn cảnh.

Ba loại địa hình quyết định độ khó:

```
(a) Lòng chảo (convex)       (b) Gồ ghề (non-convex)     (c) Rời rạc (discrete)

 ╲                 ╱          ╲   ╱╲    ╱╲    ╱           •   •
  ╲               ╱            ╲ ╱  ╲  ╱  ╲  ╱ ╲            •    •   •
   ╲             ╱              ╳    ╲╱    ╲╱   ╲          •  •
    ╲___________╱                    local  local ╲             •  ← không có
         ↑                            min!   min                   "lân cận trơn"
      cứ đi xuống                 đi xuống có thể kẹt          để lần theo
      là tới đáy                  ở hố nông
```

- Địa hình **(a)**: nhắm mắt đi xuống dốc, chắc chắn tới đáy toàn cục. Đây là thế giới convex — thế giới "dễ".
- Địa hình **(b)**: đi xuống dốc dễ kẹt ở hố địa phương (local minimum). Thế giới của deep learning.
- Địa hình **(c)**: các phương án là những điểm rời rạc, không có khái niệm "dốc" để lần theo. Thế giới của xếp pod, chọn route, bật/tắt — tối ưu tổ hợp (combinatorial optimization), thường là nơi khó nhất.

### Bản đồ phân loại — la bàn của cả chương

| Trục phân loại | Nhánh "dễ" hơn | Nhánh "khó" hơn |
|---|---|---|
| Không gian phương án | Continuous (trơn, có gradient) | Discrete (tổ hợp, bùng nổ) |
| Hình dạng objective | Convex (local = global) | Non-convex (nhiều hố) |
| Yêu cầu lời giải | Approximate/heuristic (đủ tốt) | Exact (tối ưu tuyệt đối) |

Và một sự thật giải phóng tư duy: **các thuật toán bạn đã học chính là optimization trên địa hình có cấu trúc đặc biệt**. Greedy (chương 15) là "cứ đi xuống dốc cục bộ" — chỉ đúng khi địa hình có tính chất matroid/exchange khiến dốc cục bộ không đánh lừa. DP (chương 14) là duyệt toàn bộ địa hình một cách thông minh — chỉ khả thi khi optimal substructure gấp được không gian khổng lồ vào bảng đa thức. Dijkstra là tối ưu trên đồ thị với objective cộng dồn không âm. Khi các cấu trúc đặc biệt đó vắng mặt, ta cần vũ khí tổng quát hơn — đó là phần còn lại của chương.

## 3. First Principles

### Vì sao không thể "cứ thử hết"?

Không gian tổ hợp bùng nổ theo cấp số nhân hoặc giai thừa (chương 05): n pod × m node là mⁿ; thứ tự ghé n điểm giao hàng là n!. Bảng ngưỡng của chương 10 nói duyệt 2ⁿ chỉ chịu được n ≈ 25, n! chịu được n ≈ 11. Mọi bài production đều vượt xa ngưỡng đó. Vậy chỉ còn ba con đường, và toàn bộ lĩnh vực optimization là ba con đường này:

1. **Khai thác cấu trúc để không phải thử hết mà vẫn exact** — DP, greedy-có-chứng-minh, Branch and Bound, Linear Programming. Cấu trúc là thứ cho phép nói "cả vùng kia không thể chứa nghiệm tốt hơn, bỏ qua".
2. **Đi theo tín hiệu địa phương** — gradient descent: không nhìn toàn cảnh, chỉ theo hướng dốc nhất tại chỗ. Trả giá bằng rủi ro local minimum, trừ khi địa hình convex.
3. **Chấp nhận gần đúng có kiểm soát** — approximation algorithm (có cam kết toán học về độ xa tối ưu) và heuristic (không cam kết, nhưng thực nghiệm tốt).

### Nghiệm tối ưu nằm ở biên — trực giác nền tảng

Một nguyên lý xuất hiện lặp đi lặp lại: **khi objective "đẩy" theo một hướng, nghiệm tối ưu nằm trên biên của vùng ràng buộc, không nằm giữa**. Muốn tối đa throughput với ràng buộc CPU ≤ 16 core? Nghiệm tối ưu dùng *đúng* 16 core — nếu còn thừa, còn đẩy thêm được. Ràng buộc nào "căng" (active/binding) tại nghiệm tối ưu chính là bottleneck thật của hệ thống; ràng buộc không căng là tài nguyên thừa. Đọc được ràng buộc nào đang căng là đọc được "nên mua thêm gì" — với LP, lý thuyết duality còn gán hẳn một *giá tiền* (shadow price) cho mỗi đơn vị nới ràng buộc. Đây là lý do các nhà kinh tế yêu LP.

### Nếu không có khung optimization thì sao?

Hệ thống vẫn chạy — bằng những quyết định "hợp lý" không đo đếm. Cái giá là ba thứ: (1) **không biết mình đang lãng phí bao nhiêu** — thiếu lower bound, "tiết kiệm 20% chi phí" không phân biệt được với "vẫn đang đốt gấp đôi mức cần"; (2) **tranh cãi không hồi kết** vì objective ngầm định mỗi người một kiểu; (3) **không nhận diện được bài toán khó** — team cam kết deadline cho một bài NP-hard mà không biết, rồi đốt quý này sang quý khác đi tìm exact solution không tồn tại trong thời gian đa thức.

## 4. Mathematical Model

### Linear Programming — hòn ngọc của tối ưu liên tục

Khi cả objective lẫn ràng buộc đều **tuyến tính** (bậc nhất theo biến), ta có Linear Programming (LP) — lớp bài toán lớn đáng kinh ngạc mà nhân loại giải được *trọn vẹn và nhanh*.

Ví dụ mô hình hóa: dịch vụ của bạn có thể serve request bằng instance loại A (xử lý 100 req/s, giá $3/h, tốn 2GB RAM) hoặc loại B (250 req/s, $8/h, 7GB RAM). Tổng RAM của account bị quota 70GB. Cần throughput ≥ 2000 req/s. Tối thiểu chi phí:

```
Biến:        x = số instance A, y = số instance B   (x, y ≥ 0)
Objective:   minimize  3x + 8y
Constraints: 100x + 250y ≥ 2000    (đủ throughput)
             2x + 7y     ≤ 70      (quota RAM)
```

Hình học của LP trong mặt phẳng (x, y): mỗi ràng buộc tuyến tính là một nửa mặt phẳng; giao của chúng là một đa giác lồi (tổng quát: **polytope** — khối lồi nhiều chiều). Objective tuyến tính là một họ đường thẳng song song trượt dần:

```
  y
  │╲  vùng hợp lệ (polytope) = giao các nửa mặt phẳng
  │ ╲
  │  ╲ 100x+250y=2000
  │   ╲              ┌ 2x+7y=70
  │    ╲.........╱───┘
  │     ╲■■■■■■■╱
  │      ╲■■■■■╱   ← trượt đường  3x+8y = c
  │       ╲■■■╱       theo hướng giảm c...
  │        ╲■╱
  │      ●  ╲      ...điểm cuối cùng còn chạm vùng hợp lệ
  │    đỉnh! ╲        luôn là MỘT ĐỈNH của đa giác
  └──────────────────────── x
```

**Định lý nền tảng của LP: nghiệm tối ưu (nếu tồn tại) luôn đạt tại một đỉnh của polytope.** Trực giác: trượt một đường thẳng qua khối lồi, điểm tiếp xúc cuối cùng phải là góc (hoặc cả một cạnh — khi đó hai đỉnh đầu cạnh đều tối ưu). Hệ quả choáng váng: bài toán trên không gian **vô hạn** điểm co về việc xét **hữu hạn** đỉnh. Thuật toán **simplex** (Dantzig, 1947) làm đúng điều đó: xuất phát từ một đỉnh, mỗi bước đi sang đỉnh kề tốt hơn, dừng khi không đỉnh kề nào tốt hơn — và vì polytope lồi, "tối ưu cục bộ giữa các đỉnh kề" chính là tối ưu toàn cục. Simplex là greedy *được convexity bảo hành*. (Worst case của simplex là mũ, nhưng trên bài thực tế nó nhanh đến mức 70 năm sau vẫn là workhorse; interior point method cho đảm bảo đa thức.)

Kỹ sư hiện đại không viết simplex — gọi solver (GLPK, HiGHS, CBC, Gurobi). Kỹ năng đáng tiền là **mô hình hóa**: ép được bài của mình về dạng tuyến tính, chọn biến khéo để ràng buộc thành tuyến tính.

### Convexity — ranh giới giữa "dễ" và "khó"

Một tập là **lồi** nếu đoạn thẳng nối hai điểm bất kỳ trong tập nằm trọn trong tập. Một hàm là **lồi** nếu vùng phía trên đồ thị là tập lồi (đồ thị hình lòng chảo). Định lý làm nên tất cả:

> Với bài toán convex (objective lồi, vùng hợp lệ lồi), **mọi local minimum là global minimum**.

Chứng minh một câu: giả sử x là local min mà tồn tại y tốt hơn hẳn; trượt từ x về phía y trên đoạn thẳng nối chúng (nằm trong vùng hợp lệ vì tập lồi), giá trị hàm lồi trên đoạn này bị chặn bởi nội suy tuyến tính hai đầu nên *giảm ngay lập tức* — mâu thuẫn với x là local min.

Đây là ranh giới thật sự của "dễ/khó" trong tối ưu liên tục — không phải tuyến tính/phi tuyến: bài convex phi tuyến (như logistic regression) giải toàn cục ngon lành; bài non-convex bé tí có thể kẹt vĩnh viễn. LP dễ vì nó convex. Deep learning khó (về lý thuyết) vì loss landscape non-convex — và thú vị thay, thực nghiệm cho thấy ở số chiều rất cao, phần lớn điểm kẹt là saddle point thoát được, còn các local minima tìm thấy thường "đủ tốt như nhau". Lý thuyết convex vẽ ranh giới; thực nghiệm deep learning cho thấy phía bên kia ranh giới không tối tăm đồng đều.

### Integer Programming và LP relaxation

Thêm một ràng buộc bé xíu — "x phải nguyên" — và LP hiền lành biến thành Integer Programming (IP) **NP-hard**. Trực giác vì sao: tính nguyên đập vỡ tính lồi (tập các điểm nguyên không lồi), nghiệm không còn dồn về đỉnh polytope, và "làm tròn nghiệm LP" có thể vừa vi phạm ràng buộc vừa xa tối ưu tùy ý. Nhưng LP không vô dụng với IP — ngược lại:

> **LP relaxation**: xóa ràng buộc nguyên, giải LP (nhanh). Nghiệm LP ≤ nghiệm IP (với minimize) — vì không gian rộng hơn chỉ có thể tốt bằng hoặc hơn. Ta được một **lower bound có chứng chỉ**.

Lower bound này quý hai lần: nó cho biết heuristic của bạn cách tối ưu bao xa ("LP nói không thể dưới 34 node, heuristic ra 36 → gap ≤ 6%, dừng tối ưu được rồi"), và nó là trái tim của Branch and Bound ngay sau đây.

## 5. Thuật toán

### Branch and Bound — duyệt toàn bộ, nhưng có cắt tỉa

Backtracking (duyệt cây phương án, quay lui khi vi phạm ràng buộc) chỉ cắt nhánh **không hợp lệ**. Branch and Bound (B&B) cắt mạnh hơn nhiều: cắt cả nhánh hợp lệ nhưng **chứng minh được là không thể tốt hơn nghiệm đã có**. Cần hai nguyên liệu:

- **Branch**: chia không gian thành các nhánh con (pod 1 vào node A / node B / ...).
- **Bound**: hàm chặn — ước lượng *lạc quan* cho cả nhánh (với maximize: chặn trên "tốt nhất nhánh này có thể đạt"). Nếu bound(nhánh) ≤ best đã tìm thấy → cả nhánh vứt, không mất gì.

**Vì sao không mất nghiệm tối ưu?** Vì bound *lạc quan* (admissible): bound(nhánh) ≥ giá trị thật của mọi phương án trong nhánh. Nhánh bị cắt thỏa bound ≤ best, suy ra mọi phương án trong đó ≤ best — nghiệm tối ưu hoặc đã nằm trong tay, hoặc ở nhánh khác. B&B là **exact algorithm**: kết quả tối ưu tuyệt đối, chỉ có thời gian chạy là không đảm bảo (worst case vẫn mũ — NP-hard không biến mất, chỉ là thực tế bound tốt cắt được phần lớn cây).

Ví dụ 0/1 knapsack (chọn tập đồ vật giá trị max, tổng khối lượng ≤ W) — bound bằng chính LP relaxation: cho phép lấy *phân số* đồ vật, nghiệm greedy theo mật độ giá trị/khối lượng cho ngay chặn trên của nhánh.

```go
package knapsack

// Item đã được sort giảm dần theo value/weight trước khi gọi.
type Item struct {
	Value, Weight int
}

// fractionalBound: chặn trên lạc quan cho nhánh hiện tại —
// nghiệm của LP relaxation (được lấy phân số đồ vật cuối).
// Vì nới lỏng ràng buộc, mọi nghiệm nguyên của nhánh đều ≤ bound này.
func fractionalBound(items []Item, i, capLeft, valueSoFar int) float64 {
	bound := float64(valueSoFar)
	for ; i < len(items) && capLeft > 0; i++ {
		if items[i].Weight <= capLeft {
			bound += float64(items[i].Value)
			capLeft -= items[i].Weight
		} else {
			// lấy phân số phần còn lại — chỉ hợp lệ trong relaxation
			bound += float64(items[i].Value) * float64(capLeft) / float64(items[i].Weight)
			break
		}
	}
	return bound
}

func Solve(items []Item, capacity int) int {
	best := 0
	var dfs func(i, capLeft, value int)
	dfs = func(i, capLeft, value int) {
		if value > best {
			best = value
		}
		if i == len(items) {
			return
		}
		// CẮT TỈA: cả nhánh này không thể vượt best → bỏ, không mất nghiệm.
		if fractionalBound(items, i, capLeft, value) <= float64(best) {
			return
		}
		if items[i].Weight <= capLeft { // nhánh "lấy đồ vật i"
			dfs(i+1, capLeft-items[i].Weight, value+items[i].Value)
		}
		dfs(i+1, capLeft, value) // nhánh "bỏ đồ vật i"
	}
	dfs(0, capacity, 0)
	return best
}
```

Ba đòn bẩy hiệu năng của mọi B&B: (1) bound càng **chặt** cắt càng sâu — đáng đầu tư tính toán hơn cho bound; (2) tìm **nghiệm tốt sớm** (chạy heuristic trước để khởi tạo best) — best cao thì dao cắt sắc ngay từ đầu; (3) **thứ tự duyệt** — nhánh hứa hẹn trước. Các MIP solver thương mại là B&B công nghiệp hóa với hàng trăm kỹ thuật bound/cut — nên một lần nữa: mô hình hóa rồi gọi solver, đừng tự viết.

### Gradient Descent — tối ưu khi chỉ có tín hiệu địa phương

Với hàm liên tục f(w) trên hàng triệu biến (trọng số model), duyệt là vô nghĩa, nhưng ta có thứ không gian rời rạc không có: **đạo hàm**. Gradient ∇f tại một điểm là vector chỉ hướng tăng nhanh nhất của f — không cần giải tích nặng, chỉ cần trực giác người leo núi mù: đứng tại chỗ, sờ độ nghiêng dưới chân, biết hướng nào dốc lên mạnh nhất. Vậy đi ngược lại:

> **w ← w − η · ∇f(w)**

với η là **learning rate** — cỡ bước chân, và là trade-off trần trụi: η quá nhỏ → hội tụ như rùa; η quá lớn → nhảy qua đáy, dao động hoặc phân kỳ. Trên địa hình convex, quy trình này hội tụ về global minimum. Trên địa hình non-convex, nó dừng ở nơi gradient ≈ 0 — có thể là local minimum hay saddle point.

**SGD — khi nhiễu là tính năng.** Tính ∇f trên toàn bộ dataset mỗi bước quá đắt; Stochastic Gradient Descent tính trên mini-batch ngẫu nhiên — một *ước lượng nhiễu* của gradient thật (kỳ vọng đúng, phương sai khác 0 — chương 11). Tưởng là thỏa hiệp, hóa ra là món quà: cú lắc ngẫu nhiên giúp nhảy khỏi local minimum nông và saddle point — hố nông không giữ nổi kẻ say rượu; và thực nghiệm cho thấy nhiễu còn đẩy nghiệm về các đáy *rộng* (flat minima), vốn generalize tốt hơn đáy hẹp. Cùng một nguyên lý "nhiễu giúp thoát kẹt" sẽ quay lại ngay dưới đây trong simulated annealing — đây là một trong những ý tưởng đẹp nhất của optimization.

### Khi bài toán NP-hard: đổi mục tiêu, đừng đổi vũ trụ

Nhận diện nhanh các "khuôn mặt" NP-hard thường gặp: **TSP** (route ngắn nhất qua n điểm), **bin packing** (xếp đồ vào ít thùng nhất — chính là bài xếp pod!), **graph coloring** (tô màu đỉnh kề khác màu — chính là xếp lịch tránh xung đột), knapsack, set cover. Gặp chúng, phản xạ đúng không phải "nghĩ thêm sẽ ra thuật toán nhanh" mà là **đổi yêu cầu**: từ "tối ưu tuyệt đối" sang "tốt có cam kết" hoặc "tốt theo thực nghiệm".

**Approximation algorithm** — heuristic kèm chứng chỉ. Với bin packing: First Fit Decreasing (sort đồ giảm dần, mỗi món bỏ vào thùng đầu tiên còn vừa) có chứng minh **FFD ≤ 11/9 · OPT + 1** — không bao giờ tệ hơn tối ưu quá ~22% cộng một thùng, bất kể input hiểm cỡ nào. Con số 11/9 là một định lý, không phải quan sát thực nghiệm — đó là khác biệt giữa approximation algorithm và heuristic thường. (Cũng có định lý chiều ngược: trừ khi P=NP, không thuật toán đa thức nào đảm bảo tốt hơn 3/2·OPT cho bin packing tổng quát — biết giới hạn dưới giúp bạn ngừng đi tìm thứ không tồn tại.)

**Simulated annealing** — leo núi biết lùi. Local search thuần (luôn nhận nước đi cải thiện) kẹt ở local optimum. Simulated annealing thỉnh thoảng **chấp nhận nước đi tồi** với xác suất e^(−Δ/T): "nhiệt độ" T cao lúc đầu (đi lung tung, thăm dò rộng), nguội dần theo lịch (dần nghiêm túc, hội tụ). Cùng triết lý nhiễu-để-thoát-kẹt của SGD, áp cho không gian rời rạc. Không cam kết chất lượng, nhưng đơn giản, tổng quát, và tốt bất ngờ trên nhiều bài lịch biểu/layout thực tế.

## 6. Trade-off

**Exact vs approximate — mua đảm bảo bằng thời gian.** B&B/MIP solver cho tối ưu tuyệt đối kèm chứng chỉ, nhưng thời gian chạy không chặn được: cùng model, thêm 10 biến có thể từ 2 giây thành 2 ngày. Heuristic chạy ổn định nhanh nhưng khoảng cách tới tối ưu vô danh. Đường giữa khôn ngoan mà solver hiện đại hỗ trợ sẵn: chạy B&B với **optimality gap** — dừng khi nghiệm hiện tại cách bound dưới 1% ("tốt trong vòng 1% của tối ưu" thường là tất cả những gì business cần, với 1% chi phí thời gian).

**Chất lượng model vs độ trung thực.** Bài thực có phi tuyến (giá tier, chiết khấu theo khối lượng)? Bạn chọn: tuyến tính hóa gần đúng để được LP nhanh và exact-trên-model-xấp-xỉ, hay giữ nguyên phi tuyến và chịu tối ưu non-convex. **Nghiệm tối ưu của model sai là nghiệm sai được trình bày tự tin** — sai số mô hình hóa thường lấn át sai số thuật toán, nên đầu tư vào "model đúng bài toán" trước "solver xịn".

**Chi phí tối ưu vs giá trị của tối ưu.** Bản thân việc tối ưu tốn tài nguyên — đây là bài toán tối ưu cấp hai. Query optimizer của PostgreSQL chuyển từ duyệt-DP-exact sang genetic heuristic (GEQO) khi join vượt 12 bảng: với không gian plan bùng nổ, *thời gian đi tìm plan tối ưu* sẽ vượt *thời gian chạy plan tàm tạm*. Tối ưu hóa cũng phải hoàn vốn.

**Optimize một mục tiêu vs nhiều mục tiêu.** Chi phí, latency, độ bền — không cùng đơn vị. Hai cách xử lý tỉnh táo: gộp có chủ đích (weighted sum — và cãi nhau về trọng số *một lần, công khai*), hoặc đưa các mục tiêu phụ xuống làm ràng buộc ("minimize cost *sao cho* p99 ≤ 200ms") — cách thứ hai thường trung thực hơn với ý định thật của business. Điều không được làm: để mỗi thành viên ngầm mang một objective riêng.

**Khi nào KHÔNG cần bộ máy optimization:** không gian nhỏ (duyệt hết trong 1ms — cứ brute force, đơn giản thắng); bài có cấu trúc greedy/DP đã chứng minh (dùng thẳng, chương 14–15); và khi dữ liệu đầu vào nhiễu đến mức chênh lệch 5% giữa heuristic và tối ưu chìm nghỉm trong sai số đo — đừng mua độ chính xác mà input không gánh nổi.

## 7. Production Applications

**Kubernetes scheduler** — constrained optimization đội lốt config YAML. Hai pha đúng sách giáo khoa: **Filter** loại node vi phạm ràng buộc cứng (đủ CPU/RAM, taint/toleration, node selector — chính là g(x)); **Score** chấm điểm node còn lại bằng tổng có trọng số các plugin (spread, ít tài nguyên còn lại, affinity mềm — chính là f(x)), chọn max. Đáng chú ý: scheduler chấm *từng pod một* theo kiểu greedy online — không giải bin packing toàn cục (quá đắt, và pod đến theo thời gian thực). Muốn ép chặt hơn về ít node, đổi scoring strategy sang `MostAllocated` (bin-packing flavor); còn descheduler + cluster autoscaler là vòng ngoài sửa dần nghiệm greedy — một kiến trúc chấp nhận nghiệm xấp xỉ liên tục thay vì tối ưu một lần.

**Cloud cost optimization**: bài "mua bao nhiêu reserved/savings plan, bao nhiêu spot, bao nhiêu on-demand" với nhu cầu dự báo theo giờ là LP/MIP kinh điển — biến là số lượng commit từng loại, ràng buộc là phủ nhu cầu, objective là tổng chi. Các công cụ FinOps thương mại chạy đúng công thức này; team platform tự viết được bằng vài trăm dòng gọi solver mã nguồn mở — một trong những nơi ROI của toán rõ tiền nhất.

**Query optimizer** (gặp lại ở chương 27): không gian phương án là các cây join/phương pháp scan, objective là cost model ước lượng I/O + CPU, thuật toán là DP trên tập con bảng (System R — chính là DP bitmask của chương 14), quá 12 bảng thì rơi về heuristic di truyền. Mọi khái niệm của chương này xuất hiện đủ mặt trong một component bạn dùng mỗi ngày qua `EXPLAIN`.

**ML training**: toàn bộ deep learning là gradient descent non-convex quy mô tỷ biến — mọi thứ về learning rate schedule, momentum, Adam là kỹ nghệ quanh đúng một dòng `w ← w − η∇f`. Hyperparameter tuning là tối ưu *cấp hai* trên hàm đắt-không-đạo-hàm → Bayesian optimization (xây model xác suất của objective, chọn điểm thử kế tiếp tối đa thông tin — duyên nợ với chương 20).

**Ad auction và budget allocation**: phân bổ ngân sách quảng cáo cho các kênh/campaign với ràng buộc tổng chi và mục tiêu conversion là LP; bài toán ghép quảng cáo vào slot với ngân sách nhà quảng cáo là matching/LP online — nền của các hệ ad exchange tỷ đô.

**Logistics — TSP ngoài đời**: route giao hàng của Grab/Giao Hàng Nhanh, thu gom container, lịch bảo trì — biến thể TSP/VRP (vehicle routing), NP-hard cả họ. Ngành công nghiệp giải bằng đúng thực đơn mục 5: MIP cho instance nhỏ, LNS/simulated annealing/tabu cho instance lớn, chấp nhận 2–5% trên tối ưu, chạy lại mỗi đêm. OR-Tools (Google, mã nguồn mở) là hộp công cụ tiêu chuẩn.

## 8. Interview

**Nhận diện lớp bài trước, code sau.** Bài optimization trong phỏng vấn thường ngụy trang thành DP hoặc greedy — "max profit", "min cost", "fewest steps". Cây quyết định đúng: (1) phát biểu biến–objective–ràng buộc; (2) hỏi có **optimal substructure + overlapping subproblems** không → DP (chương 14); (3) có **exchange argument** không → greedy (chương 15); (4) không gian bé (n ≤ 20–25) → duyệt/bitmask, có bound tốt thì B&B; (5) không cái nào khớp và bài trông giống TSP/bin-packing → nói rõ "đây là bài NP-hard, em đề xuất heuristic + đánh giá gap" — với người phỏng vấn senior, *nhận ra bài khó* ăn điểm hơn giả vờ có lời giải đa thức.

**Kỹ năng ăn 50% số điểm: phát biểu objective và constraints TRƯỚC khi code.** Đứng trước "phân task cho worker sao cho hợp lý", ứng viên yếu nhảy vào code; ứng viên mạnh hỏi lại: "Tối thiểu makespan hay tổng thời gian? Task có preempt được không? Worker đồng nhất không?" — mỗi câu trả lời đổi hẳn thuật toán (makespan với worker đồng nhất là bài NP-hard có greedy 4/3-approx; tổng completion time lại có greedy exact). Không chốt objective mà code được nghĩa là bài quá dễ hoặc code sai đề.

**Lỗi tư duy phổ biến:** áp greedy không chứng minh (greedy đúng knapsack phân số nhưng sai 0/1 — phản ví dụ 3 dòng); tưởng thêm ràng buộc làm bài dễ đi (LP → IP là chiều *khó lên*); tối ưu proxy metric rồi ngạc nhiên khi metric thật xấu đi; và quên rằng "làm tròn nghiệm liên tục" có thể vi phạm ràng buộc.

**Câu system design thật: "Thiết kế autoscaling policy".** Đây là bài optimization trá hình và là cơ hội thể hiện toàn bộ chương: biến = số replica theo thời gian; objective = chi phí instance + phạt vi phạm SLA (phải *định giá* được một phút chậm — buộc business nói ra trade-off); ràng buộc = quota, cooldown, tốc độ scale, warm-up time; cấu trúc online = quyết định bây giờ với thông tin tương lai mù mờ → cần dự báo + buffer, và reactive threshold thực chất là greedy heuristic cho bài online này. Ứng viên vẽ được khung đó rồi *mới* bàn HPA/metrics là ứng viên tư duy từ bài toán xuống công cụ, không phải ngược lại.

## 9. Anti-pattern

**Tối ưu mà không phát biểu bài toán.** Sprint mang tên "optimize hệ thống" không định nghĩa objective — kết quả là mỗi PR kéo một hướng: người giảm latency bằng cách đốt thêm cache (tăng cost), người giảm cost bằng cách hạ replica (tăng latency), tổng tiến bộ bằng không. Không có f(x) chung, "tối ưu" chỉ là chuyển hóa đơn từ cột này sang cột khác.

**Đòi exact solution cho bài NP-hard.** Team cam kết "hệ thống xếp lịch tối ưu" cho bài graph coloring trá hình, đốt hai quý vào các thuật toán ngày càng cầu kỳ mà không nhận diện lớp bài toán. Một giờ nhận diện NP-hardness + một tuần FFD/annealing + báo cáo gap so với LP bound sẽ cho kết quả tốt hơn và trung thực hơn.

**Tối ưu proxy thay vì mục tiêu thật — và bị Goodhart trừng phạt.** "Khi một thước đo trở thành mục tiêu, nó không còn là thước đo tốt." Optimize CPU utilization → hệ thống "đạt 90%" bằng busy-work; optimize số bug đóng → bug bị chẻ nhỏ. Thuật toán tối ưu *sẽ* tìm ra kẽ hở giữa proxy và ý định — nó giỏi đúng việc đó. Reward hacking trong RL là phiên bản ML của cùng căn bệnh. Thiết kế objective là công việc nghiêm túc nhất của toàn pipeline.

**Local search từ một điểm xuất phát, kết luận toàn cục.** Chạy hill climbing một lần, tuyên bố "đây là cấu hình tốt nhất". Địa hình non-convex đầy hố nông; tối thiểu phải multi-start (nhiều điểm xuất phát ngẫu nhiên) hoặc thêm nhiễu kiểu annealing, và luôn báo cáo kèm bound nếu có được.

**Nghiệm tối ưu giòn (brittle optimum).** Nghiệm ép sát 100% mọi ràng buộc là nghiệm nằm trên *đỉnh nhọn* của polytope — input lệch 2% là vỡ (node down một cái, bin packing "hoàn hảo" sụp dây chuyền). Production cần robust optimization dạng bình dân: chừa headroom 10–20% như một ràng buộc tường minh. Tối ưu đến kiệt là tối ưu cho đúng một ngày dữ liệu đứng yên.

## 10. Best Practices

**Nên:**

- Viết ba dòng "Biến — Objective — Constraints" ra văn bản trước mọi nỗ lực tối ưu, kể cả (nhất là) trong design doc. Đây là công cụ giao tiếp trước khi là công cụ toán.
- Tìm lower/upper bound sớm (LP relaxation, tổng-tài-nguyên-chia-dung-lượng cho bin packing): biết trần/sàn biến "cải thiện 15%" từ con số PR thành đánh giá kỹ thuật.
- Bắt đầu bằng heuristic đơn giản nhất + đo gap so với bound; chỉ leo thang độ phức tạp (greedy → local search → MIP solver) khi gap chứng minh còn đáng tiền.
- Dùng solver có sẵn (OR-Tools, HiGHS, SciPy) — mô hình hóa là việc của bạn, simplex/B&B là việc của 70 năm nghiên cứu đã đóng gói.
- Kiểm tra độ nhạy của nghiệm: nhiễu input ±10% rồi giải lại; nghiệm đổi hoàn toàn nghĩa là bạn cần robust hơn, không phải optimal hơn.

**Không nên:**

- Không tin greedy khi chưa có exchange argument hoặc phản ví dụ đã thử tìm mà không thấy — kiểm tra bằng brute force trên input nhỏ trước.
- Không nhúng "ràng buộc nguyên" vào model nếu làm tròn sau vẫn chấp nhận được — LP nhanh hơn IP nhiều bậc, và với biến lớn (hàng nghìn instance) sai số làm tròn không đáng kể.
- Không optimize khi chưa chắc chắn về objective — nghiệm tối ưu của câu hỏi sai nguy hiểm hơn nghiệm tàm tạm của câu hỏi đúng, vì nó mặc đồng phục toán học.
- Không quên chi phí của chính việc tối ưu — nếu tính toán lại plan tốn hơn phần tiết kiệm, hãy học PostgreSQL: rơi về heuristic một cách có chủ đích.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Vì sao nghiệm tối ưu của LP luôn đạt tại đỉnh polytope, và tính lồi đóng vai trò gì trong việc simplex "greedy giữa các đỉnh kề" mà vẫn ra tối ưu toàn cục?
2. Bound function của Branch and Bound phải có tính chất gì để đảm bảo không cắt mất nghiệm tối ưu — và bound *chặt hơn* mua được điều gì?
3. Kubernetes scheduler giải bài xếp pod bằng greedy từng-pod thay vì bin packing toàn cục — những trade-off nào biện minh cho lựa chọn đó?

---

*Chương tiếp theo: [22 — Computational Geometry](/series/math-for-engineers/level-4-advanced-mathematics/22-computational-geometry/), nơi không gian tìm kiếm không còn trừu tượng mà là không gian vật lý thật — bản đồ, tọa độ GPS, va chạm trong game — với những cấu trúc dữ liệu sinh ra để hỏi "cái gì ở gần đây?" mà không phải quét cả thế giới.*
