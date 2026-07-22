+++
title = "Chương 04 — Mathematical Modeling: Từ bài toán thực tế đến mô hình toán"
date = "2026-07-20T07:40:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 1 – Mathematical Thinking
> Yêu cầu trước: Chương 01 (Logic), Chương 02 (Set Theory, Functions & Relations), Chương 03 (Proof Techniques)

---

## 1. Problem Statement

Một buổi system design interview. Ứng viên được yêu cầu: "Thiết kế tính năng gợi ý bạn bè cho mạng xã hội". Ứng viên lập tức nói về microservice, Redis cache, Kafka pipeline, sharding database. Người phỏng vấn ngắt lời bằng một câu hỏi duy nhất: *"Khoan — 'bạn của bạn' là gì, nói bằng toán?"*

Ứng viên khựng lại. Và đó chính là khoảnh khắc phân định giữa người *lắp ráp công nghệ* và người *giải bài toán*. Bởi vì trước khi chọn Redis hay Kafka, phải trả lời được: dữ liệu này có cấu trúc toán học gì? Quan hệ bạn bè là một **đồ thị vô hướng**; "gợi ý bạn bè" là "tìm các đỉnh ở khoảng cách đúng bằng 2". Một khi phát biểu được như vậy, lời giải thuật toán gần như tự rơi ra — và mọi lựa chọn công nghệ phía sau chỉ là chi tiết triển khai.

Mathematical modeling — mô hình hóa toán học — là kỹ năng **dịch từ ngôn ngữ đời thực sang ngôn ngữ toán**. Nó là kỹ năng quan trọng nhất trong toàn bộ bộ tài liệu này, và trớ trêu thay, cũng là kỹ năng ít được dạy nhất. Trường học dạy giải phương trình đã cho sẵn; LeetCode cho bạn input/output đã định nghĩa gọn gàng. Nhưng công việc thật bắt đầu sớm hơn một bước: đề bài là một đoạn văn mơ hồ từ product manager, và **bạn** là người quyết định nó là bài toán gì.

Câu hỏi trung tâm của chương này:

**Cho một bài toán phát biểu bằng ngôn ngữ tự nhiên, làm sao chọn được cấu trúc toán học đúng để mô tả nó — và làm sao biết mình đã chọn sai?**

Nếu không có kỹ năng này, ba chương trước (logic, set theory, proof) là công cụ không có nguyên liệu: bạn biết chứng minh, nhưng không có mệnh đề nào để chứng minh; biết đồ thị là gì, nhưng không nhận ra bài toán trước mặt là bài toán đồ thị.

## 2. Trực giác

### Mô hình là một tấm bản đồ

Bản đồ tàu điện ngầm London nổi tiếng vì nó **sai một cách có chủ đích**: khoảng cách trên bản đồ không tỷ lệ với khoảng cách thật, đường ray cong được vẽ thẳng, vị trí địa lý bị bóp méo. Nhưng nó là bản đồ tàu điện tốt nhất từng được thiết kế — vì nó giữ lại đúng thứ hành khách cần (*ga nào nối với ga nào, đổi tuyến ở đâu*) và vứt bỏ mọi thứ còn lại.

Mô hình toán học cũng vậy. Khi ta nói "quan hệ bạn bè là một đồ thị", ta đã vứt bỏ: tên, tuổi, avatar, lịch sử chat, mức độ thân thiết. Ta chỉ giữ lại một điều: **ai nối với ai**. Sự "nghèo nàn" đó không phải khuyết điểm — nó chính là sức mạnh. Vì đồ thị đã bị lột trần khỏi ngữ cảnh, mọi định lý về đồ thị (chương 08) lập tức áp dụng được: BFS tìm khoảng cách, connected component tìm cộng đồng, độ đỉnh (degree) tìm người nổi tiếng.

Nhà thống kê George Box đúc kết điều này trong câu nói được trích dẫn nhiều nhất của ngành: **"All models are wrong, but some are useful"** — mọi mô hình đều sai, nhưng một số hữu ích. Bản đồ tàu điện "sai" về địa lý nhưng hữu ích để đi tàu. Câu hỏi đúng không bao giờ là "mô hình này có đúng với thực tế không?" (không bao giờ đúng hoàn toàn), mà là "**mô hình này giữ lại đúng những gì bài toán cần chưa?**".

### Cùng một hệ thống, nhiều mô hình

Điểm tinh tế nhất: một hệ thống không có *một* mô hình đúng. Xét một hệ thống thanh toán:

- Nếu câu hỏi là "hai giao dịch có xung đột không?" → mô hình là **quan hệ** trên tập giao dịch (chương 02).
- Nếu câu hỏi là "tiền đi từ đâu đến đâu?" → mô hình là **đồ thị có hướng** với trọng số.
- Nếu câu hỏi là "khả năng gian lận của giao dịch này?" → mô hình là **xác suất** (chương 11).
- Nếu câu hỏi là "hệ thống chịu được bao nhiêu giao dịch/giây?" → mô hình là **hàng đợi** và các đại lượng thống kê.

Mô hình không mô tả *hệ thống* — nó mô tả *câu hỏi bạn đang hỏi về hệ thống*. Đổi câu hỏi, phải đổi mô hình. Đây là lý do bước đầu tiên của mọi buổi system design giỏi là làm rõ requirement: không phải vì phép lịch sự, mà vì **chưa có câu hỏi thì chưa thể có mô hình**.

### Nếu không mô hình hóa thì sao?

Thì bạn vẫn code được — bằng cách mô phỏng trực tiếp ngôn ngữ đời thực vào code: class `User` có list `friends`, method `suggestFriends()` với ba vòng lặp lồng nhau và một mớ `if`. Code kiểu này *chạy được*, nhưng: không ai biết nó đúng hay sai (đúng so với cái gì?), không tận dụng được thuật toán có sẵn, và không có ngôn ngữ chung để thảo luận. Mô hình hóa chính là bước biến "một đống code thể hiện ý hiểu của tôi" thành "một bài toán đã được nhân loại nghiên cứu 50 năm, có lời giải tối ưu kèm chứng minh".

## 3. First Principles

### Quy trình bốn bước

Mô hình hóa không phải cảm hứng nghệ thuật — nó là một quy trình có thể luyện tập:

```
 Bài toán thực tế (ngôn ngữ tự nhiên, mơ hồ)
        │
        ▼
 [1] NHẬN DIỆN ĐỐI TƯỢNG      "Danh từ nào là dữ liệu? Có bao nhiêu?"
        │                      → user, request, server, task...
        ▼
 [2] CHỌN CẤU TRÚC TOÁN       "Quan hệ giữa chúng có hình dạng gì?"
        │                      → set? function? graph? probability?
        ▼
 [3] PHÁT BIỂU RÀNG BUỘC      "Điều gì bắt buộc đúng? Tối ưu cái gì?"
        │                      → invariant, mục tiêu, giới hạn
        ▼
 [4] CHỌN THUẬT TOÁN          Lúc này mới là lãnh địa của Level 3
```

Bước 2 là trái tim của chương này. Câu hỏi dẫn đường: **"Quan hệ giữa các đối tượng có hình dạng gì?"**

- Chỉ cần biết *thuộc hay không thuộc*, không quan tâm thứ tự? → **Set**.
- Mỗi input ứng với đúng một output? → **Function** (và hash map là hiện thân của nó trong code).
- Các cặp đối tượng *nối với nhau*, và điều quan trọng là ai nối ai? → **Graph**.
- Kết nối có chiều, và "trước/sau" có ý nghĩa? → **Directed graph**; nếu không được phép quay vòng → **DAG**.
- Kết quả không chắc chắn, nhưng có quy luật khi lặp nhiều lần? → **Probability**.
- Đếm số cách, số trường hợp? → **Combinatorics** (chương 05).
- Trạng thái biến đổi theo bước rời rạc? → **State machine / recurrence** (chương 06).

Bước 3 thường bị bỏ qua nhưng quyết định độ khó: cùng là "xếp task lên server", nếu ràng buộc là "mỗi server tối đa k task" thì là bài toán matching/flow; nếu ràng buộc là "tổng thời gian hoàn thành ngắn nhất" thì là scheduling — hai bài toán khác hẳn nhau về độ khó dù đối tượng giống hệt.

### Nghệ thuật đơn giản hóa: bỏ gì, giữ gì

Nguyên tắc: **giữ lại đại lượng xuất hiện trong câu hỏi và trong ràng buộc; bỏ mọi thứ còn lại — nhưng ghi lại những gì đã bỏ**. Danh sách "những giả định tôi đã đơn giản hóa" chính là danh sách rủi ro của mô hình. Ví dụ khi mô hình hóa load balancing bằng phân phối đều, ta đã *giả định các request có chi phí như nhau* — giả định này sai với các request nặng bất thường, và đó chính xác là chỗ mô hình sẽ gãy trong production (hot key, slow query). Kỹ sư giỏi không phải người có mô hình không giả định — mà là người **biết rõ mô hình của mình gãy ở đâu**.

Sai lầm chết người nhất trong toàn bộ quy trình: **sai ở bước 2 thì mọi thứ sau đều sai**. Chọn nhầm cấu trúc toán rồi thì thuật toán tối ưu đến mấy cũng là tối ưu cho bài toán khác. Debug logic thì tìm được bug; debug *mô hình* thì phải đập đi làm lại từ đầu — và thường được phát hiện sau ba tháng, khi tính năng đã lên production.

## 4. Mathematical Model

Cách tốt nhất để thấy quy trình bốn bước hoạt động là chạy nó trên bốn bài toán thật — bốn bài toán này sẽ theo bạn suốt bộ tài liệu.

### Ví dụ A — Rate limiter: mô hình đếm + thời gian

*Đề bài:* "Mỗi user chỉ được gọi API tối đa 100 lần mỗi phút."

Đối tượng: user, request, thời điểm. Quan hệ có hình dạng gì? Thoạt nhìn là *đếm* — nhưng đếm cái gì? Đếm trong "một phút" nào — phút theo đồng hồ (12:00:00–12:01:00) hay 60 giây gần nhất? Chỉ riêng việc ép mình phát biểu chính xác đã lộ ra hai mô hình khác nhau (fixed window và sliding window) với hành vi khác nhau ở biên: fixed window cho phép 200 request trong 2 giây nếu nằm vắt qua ranh giới phút.

Mô hình thanh lịch nhất là **token bucket**: một cái xô chứa tối đa B token, được rót đều r token/giây; mỗi request lấy 1 token, hết token thì từ chối. Về toán, trạng thái chỉ là một cặp số `(tokens, lastRefillTime)`, và phép cập nhật:

> tokens ← min(B, tokens + r × (now − lastRefillTime))

Cái đẹp: không cần lưu lịch sử request (bộ nhớ O(1) thay vì O(số request)), và mô hình tách bạch hai tham số có ý nghĩa vật lý riêng — r là *thông lượng dài hạn*, B là *độ chịu burst*. Từ một câu requirement mơ hồ, mô hình hóa cho ta một cấu trúc chỉ gồm hai số thực và một công thức.

### Ví dụ B — Gợi ý bạn bè: mô hình đồ thị

*Đề bài:* "Gợi ý những người user có thể quen."

Đối tượng: user và quan hệ bạn bè. Quan hệ đối xứng (A bạn B ⇔ B bạn A — chương 02), không có chiều → **đồ thị vô hướng** G = (V, E). "Có thể quen" dịch thành: đỉnh u sao cho khoảng cách d(user, u) = 2 — không phải bạn, nhưng là bạn của bạn.

```
      A ─── B ─── D        d(A,B) = 1  → đã là bạn, bỏ qua
      │           │        d(A,D) = 2  → gợi ý!
      C ────────── E       d(A,E) = 2  → gợi ý!
                           Xếp hạng: D có 1 bạn chung, E có 1 bạn chung
```

Và mô hình lập tức "tặng thêm": muốn xếp hạng gợi ý? Đếm *số đường đi độ dài 2* — chính là số bạn chung. Muốn gợi ý xa hơn? Tăng khoảng cách lên 3. Muốn "cộng đồng"? Connected component. Một khi dữ liệu đã là đồ thị, cả kho công cụ của chương 08 mở ra miễn phí.

### Ví dụ C — Load balancing: mô hình xác suất

*Đề bài:* "Chia đều request cho n server."

Cạm bẫy: "chia đều" nghe như bài toán tất định, nhưng request đến theo dòng liên tục không dừng, không thể "chia" một tập hữu hạn. Mô hình đúng là **phân phối xác suất**: mỗi request được gán vào server i với xác suất pᵢ; "đều" nghĩa là pᵢ = 1/n. Câu hỏi hay lập tức xuất hiện: chọn ngẫu nhiên đều thì server xui nhất nhận nhiều hơn trung bình bao nhiêu? (Xác suất — chương 11 — trả lời: chênh lệch cỡ √(m/n) với m request; và mô hình "power of two choices" — chọn ngẫu nhiên 2 server, gửi vào bên ít tải hơn — giảm chênh lệch này theo cấp số mũ.) Không có mô hình xác suất, những câu hỏi này thậm chí không phát biểu được, chứ đừng nói trả lời.

### Ví dụ D — Lịch deploy có dependency: mô hình DAG

*Đề bài:* "Service A phải deploy trước B và C; B trước D; C trước D. Tìm thứ tự deploy."

Đối tượng: service. Quan hệ: "phải trước" — có chiều, và *không được phép có vòng* (A trước B, B trước A là mâu thuẫn). Cấu trúc toán khớp hoàn hảo: **DAG** (directed acyclic graph), và "tìm thứ tự hợp lệ" là bài toán **topological sort** kinh điển:

```
   A ──► B ──┐
   │         ▼        Thứ tự hợp lệ: A, B, C, D
   └───► C ──► D      hoặc:          A, C, B, D
```

Mô hình còn trả lời câu hỏi khó hơn: "deploy song song được không?" — các đỉnh không có đường đi giữa chúng (B và C) chạy song song được; số "đợt" tối thiểu là độ dài đường đi dài nhất trong DAG. Đây chính xác là mô hình bên trong `go build`, Makefile, Airflow, và Kubernetes init container ordering.

### Bảng tra: tín hiệu trong đề bài → cấu trúc toán

| Tín hiệu trong đề bài | Cấu trúc toán | Chương |
|---|---|---|
| "thuộc/không thuộc", "duy nhất", "chung/riêng" | Set, các phép ∩ ∪ ∖ | 02 |
| "ứng với", "tra cứu", "ánh xạ", "mỗi X có một Y" | Function / hash map | 02, 12 |
| "kết nối", "quen nhau", "đường đi", "lan truyền" | Graph | 08 |
| "phụ thuộc", "phải trước", "tiền đề" | DAG + topological sort | 08 |
| "phân cấp", "chứa trong", "cha–con" | Tree | 07 |
| "bao nhiêu cách", "khả năng xảy ra trường hợp" | Combinatorics | 05 |
| "trung bình", "ngẫu nhiên", "tỷ lệ lỗi", "kỳ vọng" | Probability | 11 |
| "bước thứ n phụ thuộc bước trước" | Recurrence / DP | 06, 14 |
| "trạng thái", "chuyển trạng thái khi có sự kiện" | State machine | 01, 09 |
| "tối đa/tối thiểu dưới ràng buộc" | Optimization | 21 |

Bảng này không phải để học thuộc mà để **kiểm tra chéo**: khi bạn đã đoán một mô hình, hãy hỏi ngược "đề bài có tín hiệu của cấu trúc này không?". Nếu đề nói "phụ thuộc" mà mô hình của bạn không có chiều, gần như chắc chắn bạn đã mô hình sai.

## 5. Thuật toán — quy trình mô hình hóa như một thủ tục

Quy trình bốn bước ở mục 3, viết thành "pseudocode cho não":

**Bước 1 — Liệt kê danh từ, gắn kích thước.** "User (10⁷), request (10⁴/giây), server (10²)". Kích thước quyết định sau này thuật toán nào khả thi (bảng ngưỡng ở chương 10).

**Bước 2 — Hỏi hình dạng quan hệ**, dùng bảng tra ở mục 4. Nếu hai cấu trúc đều có vẻ khớp, phát biểu bài toán bằng cả hai rồi xem cách nào làm câu hỏi *dễ trả lời hơn* — mô hình tốt là mô hình làm câu hỏi trở nên tầm thường.

**Bước 3 — Viết ràng buộc thành mệnh đề logic** (chương 01): "∀ user, số request trong 60s ≤ 100". Nếu không viết được thành mệnh đề chính xác, requirement còn mơ hồ — quay lại hỏi product manager *trước khi* viết dòng code nào.

**Bước 4 — Nhận diện bài toán chuẩn.** "Đây là topological sort", "đây là BFS khoảng cách 2". Nếu không khớp bài chuẩn nào, phân rã thành các bài nhỏ hơn.

Minh họa bước cuối bằng code: một khi ví dụ D đã được mô hình hóa thành DAG, lời giải chỉ là thuật toán Kahn chép từ sách:

```go
// TopoSort trả về một thứ tự deploy hợp lệ, hoặc báo lỗi nếu có vòng
// dependency (tức là mô hình DAG bị vi phạm ngay từ dữ liệu đầu vào).
func TopoSort(services []string, deps map[string][]string) ([]string, error) {
    inDegree := make(map[string]int)      // số dependency chưa thỏa của mỗi service
    for _, s := range services {
        inDegree[s] = 0
    }
    for _, targets := range deps {        // deps["A"] = {"B","C"}: A phải trước B, C
        for _, t := range targets {
            inDegree[t]++
        }
    }

    queue := []string{}                   // các service đã sẵn sàng deploy
    for s, d := range inDegree {
        if d == 0 {
            queue = append(queue, s)
        }
    }

    order := make([]string, 0, len(services))
    for len(queue) > 0 {
        s := queue[0]
        queue = queue[1:]
        order = append(order, s)
        for _, t := range deps[s] {       // deploy xong s → "mở khóa" các service sau nó
            inDegree[t]--
            if inDegree[t] == 0 {
                queue = append(queue, t)
            }
        }
    }

    if len(order) != len(services) {      // còn đỉnh kẹt lại ⇒ tồn tại vòng phụ thuộc
        return nil, fmt.Errorf("phát hiện vòng dependency: không tồn tại thứ tự deploy hợp lệ")
    }
    return order, nil
}
```

Để ý tỷ lệ công sức: mô hình hóa là phần khó; thuật toán sau đó là 30 dòng code chuẩn. Và để ý cả điều tinh tế hơn: nhánh báo lỗi cuối hàm chính là **mô hình tự kiểm tra giả định của nó** — nếu dữ liệu thật có vòng phụ thuộc, ta biết ngay thay vì deploy sai thứ tự trong im lặng.

Tương tự với ví dụ A — mô hình token bucket "hai số thực và một công thức" dịch sang Go gần như từng chữ:

```go
// TokenBucket giới hạn tốc độ: rót đều rate token/giây, chứa tối đa burst.
// Toàn bộ trạng thái là hai số — đúng như mô hình toán ở mục 4.
type TokenBucket struct {
    rate, burst float64
    tokens      float64   // số token hiện có
    last        time.Time // lần refill gần nhất
}

// Allow trả lời: request tại thời điểm now có được phép đi qua không.
func (b *TokenBucket) Allow(now time.Time) bool {
    // Áp thẳng công thức: tokens ← min(burst, tokens + rate × Δt)
    elapsed := now.Sub(b.last).Seconds()
    b.tokens = math.Min(b.burst, b.tokens+b.rate*elapsed)
    b.last = now

    if b.tokens >= 1 {
        b.tokens-- // request này tiêu một token
        return true
    }
    return false // hết token: đây chính là "vượt rate limit"
}
```

Không một dòng nào trong hàm trên là "kỹ thuật lập trình" — tất cả là mô hình toán được chép lại. Khi mô hình đúng, code trở nên nhàm chán; code phức tạp đầy nhánh đặc biệt thường là triệu chứng của mô hình chưa đúng, không phải của bài toán khó.

## 6. Trade-off

**Chi tiết vs Khả giải (fidelity vs tractability).** Đây là trade-off trung tâm. Mô hình càng sát thực tế càng khó phân tích: load balancing với "mọi request chi phí như nhau" giải được bằng xác suất cơ bản; thêm "chi phí request phân phối heavy-tail, server có warm-up, network có độ trễ biến thiên" thì chỉ còn cách mô phỏng. Nguyên tắc thực dụng: **bắt đầu bằng mô hình đơn giản nhất chưa-sai-với-câu-hỏi**, chỉ thêm chi tiết khi câu trả lời của mô hình đơn giản mâu thuẫn với quan sát thực tế.

**Mô hình chuẩn vs mô hình tự chế.** Ép bài toán vào một bài chuẩn (topological sort, shortest path) cho bạn thuật toán đã được chứng minh và code đã được test bởi cả ngành. Mô hình tự chế khớp thực tế hơn nhưng bạn phải tự chứng minh mọi thứ. Ưu tiên mô hình chuẩn trừ khi độ vênh quá lớn — và "độ vênh" phải được chỉ ra cụ thể, không phải cảm giác.

**Mô hình tĩnh vs động.** Đồ thị bạn bè hôm nay khác ngày mai. Mô hình tĩnh (chụp một snapshot) đơn giản; mô hình động (dòng cập nhật liên tục) sát thực nhưng kéo theo cả họ thuật toán khác (incremental/streaming). Nhiều hệ thống thật chọn lai: mô hình tĩnh tính lại theo chu kỳ (gợi ý bạn bè tính mỗi đêm) — chấp nhận độ trễ dữ liệu để đổi lấy sự đơn giản.

**Khi nào KHÔNG nên mô hình hóa cầu kỳ?** Khi bài toán nhỏ, chạy một lần, và brute force trong ngân sách: kịch bản migration chạy một lần trên 10⁴ record không cần mô hình đẹp. Mô hình hóa là khoản đầu tư — chỉ đầu tư khi bài toán sẽ sống đủ lâu hoặc đủ lớn để sinh lãi. Dấu hiệu ngược lại — *phải* dừng lại mô hình hóa: bạn đang viết `if` thứ mười lăm cho các "trường hợp đặc biệt" mà thật ra là hệ quả của việc chưa gọi tên đúng cấu trúc dữ liệu.

## 7. Production Applications

Mọi bài system design lớn đều bắt đầu bằng một quyết định mô hình hóa — thường ẩn đến mức ta quên nó từng là một lựa chọn:

**Kubernetes scheduler**: bài toán "đặt pod lên node" được mô hình hóa thành **filtering + scoring trên tập hợp**: lọc tập node thỏa ràng buộc (predicate — mệnh đề logic chương 01), rồi tối ưu hàm điểm trên tập còn lại. Toàn bộ khái niệm taint/toleration/affinity chỉ là các ràng buộc bước 3 được đặt tên. Trong khi đó, thứ tự khởi động các resource có dependency (init container, operator reconcile) là mô hình DAG của ví dụ D.

**PostgreSQL query planner**: mỗi câu SQL được mô hình hóa thành **cây đại số quan hệ** (relational algebra — chính là set theory chương 02 với phép chiếu, chọn, join), và tối ưu hóa query là bài toán tìm kiếm trên không gian các cây tương đương. Không có mô hình đại số, "tối ưu query" thậm chí không định nghĩa được — tương đương giữa hai cách viết query là *định lý* trong đại số quan hệ.

**Kafka**: quyết định mô hình hóa nền tảng là "message stream = **append-only log**, consumer = con trỏ (offset) trên log". Từ mô hình tối giản này rơi ra mọi tính chất nổi tiếng: replay (lùi con trỏ), thứ tự trong partition (thứ tự trên log là toàn phần), consumer độc lập nhau (mỗi con trỏ là một số nguyên riêng). Kafka thắng không phải nhờ code nhanh hơn — nhờ **mô hình đúng hơn** cho bài toán streaming so với message queue truyền thống (mô hình "hộp thư xóa-sau-khi-đọc").

**Redis rate limiting / Go `golang.org/x/time/rate`**: token bucket của ví dụ A nằm nguyên trong package `rate` của Go — struct `Limiter` đúng là cặp `(tokens, lastEvent)` với công thức refill tuyến tính theo thời gian. Đọc source code đó là đọc một mô hình toán được dịch thẳng sang Go.

**Git**: lịch sử commit là DAG (merge tạo đỉnh có hai cha); `git rebase`, `git merge-base`, `git bisect` đều là thuật toán đồ thị trên DAG đó. `git bisect` còn là mô hình hóa kép: "tìm commit gây bug" → "binary search trên thứ tự topo" — hai mô hình lồng nhau.

Điểm chung: trong mọi ví dụ, toán không nằm trong một hàm tiện ích nào đó — **toán nằm ở tầng thiết kế**, quyết định hệ thống có khái niệm gì và không thể làm gì.

## 8. Interview

Kỹ năng mô hình hóa trong phỏng vấn có tên khác: **nhận diện mô hình ẩn**. Đề LeetCode hiếm khi nói "đây là bài đồ thị" — nó kể một câu chuyện, và việc của bạn là lột câu chuyện ra khỏi cấu trúc toán.

**Bài chuỗi thực ra là bài đồ thị** — Word Ladder (LeetCode 127): "biến 'hit' thành 'cog', mỗi bước đổi một ký tự, các từ trung gian phải trong từ điển". Bề mặt là bài xử lý chuỗi; bản chất: mỗi từ là một *đỉnh*, hai từ khác nhau một ký tự là một *cạnh*, "số bước ít nhất" là **shortest path trên đồ thị không trọng số → BFS**. Ai nhìn ra mô hình giải trong 15 phút; ai không nhìn ra sẽ viết đệ quy trên chuỗi và chết chìm. Course Schedule (LeetCode 207) tương tự: câu chuyện về môn học tiên quyết, mô hình là phát hiện vòng trong đồ thị có hướng — đúng ví dụ D.

**Bài đếm thực ra là combinatorics/DP** — Climbing Stairs (LeetCode 70): "bao nhiêu cách leo n bậc thang, mỗi bước 1 hoặc 2 bậc". Người chưa mô hình hóa sẽ thử liệt kê. Người mô hình hóa viết ngay quan hệ: cách(n) = cách(n−1) + cách(n−2) — Fibonacci, và bài "leo cầu thang" hóa ra là bài về **recurrence** (chương 06). Unique Paths (LeetCode 62) là bài đếm đường trên lưới — công thức tổ hợp C(m+n−2, m−1) giải trong một dòng.

**Lỗi tư duy phổ biến nhất**: pattern-matching theo *bề mặt* — thấy chuỗi thì nghĩ two-pointer, thấy mảng thì nghĩ sliding window. Bề mặt là kiểu dữ liệu của input; mô hình là **cấu trúc của quan hệ giữa các trạng thái**. Câu hỏi đúng khi đọc đề: "trạng thái của bài này là gì, và trạng thái nào nối với trạng thái nào?" — nếu câu trả lời vẽ được thành các nút và mũi tên, đó là bài đồ thị hoặc DP, bất kể input là chuỗi, số hay ma trận.

**Cách luyện thay vì học thuộc**: với mỗi bài đã giải, ghi lại một dòng "câu chuyện X → mô hình Y, tín hiệu nhận biết là Z". Sau 50 bài, bạn có bảng tra của riêng mình — và nhận ra chỉ có khoảng chục mô hình lặp đi lặp lại sau hàng nghìn câu chuyện khác nhau.

**Từ phỏng vấn sang production**: Word Ladder chính là fuzzy matching / spell correction thu nhỏ (đồ thị các từ theo edit distance); Course Schedule là resolver dependency của npm/Go modules. Người phỏng vấn hỏi các bài này không phải vì bạn sẽ leo cầu thang trong công việc — mà vì **kỹ năng lột mô hình ra khỏi câu chuyện** là kỹ năng bạn dùng mỗi ngày với ticket Jira.

## 9. Anti-pattern

**Nhảy thẳng vào công nghệ.** "Dùng Redis hay Postgres?" trước khi trả lời "dữ liệu này có cấu trúc gì và câu hỏi truy vấn là gì?". Công nghệ là bước 5; mô hình là bước 2. Chọn database trước khi có mô hình giống như chọn vali trước khi biết đi đâu.

**Mô hình hóa theo cấu trúc quen tay.** "Tôi giỏi SQL nên mọi thứ là bảng quan hệ" — rồi lưu đồ thị bạn bè vào bảng và viết query đệ quy 5 tầng JOIN cho khoảng cách 2. Cấu trúc toán phải đến từ hình dạng của quan hệ trong bài toán, không phải từ CV của người giải. (Chiều ngược lại cũng là anti-pattern: dựng graph database cho dữ liệu thuần bảng vì "graph nghe hiện đại".)

**Mô hình đúng nhưng câu hỏi sai.** Mô hình hóa hoàn hảo hệ thống caching bằng lý thuyết xếp hàng, trả lời chính xác câu hỏi "độ trễ trung bình" — trong khi SLA của công ty tính theo p99. Mô hình phục vụ câu hỏi; xác nhận câu hỏi trước khi đánh bóng mô hình.

**Quên các giả định đã đơn giản hóa.** Mô hình token bucket giả định đồng hồ tin cậy — rồi hệ thống chạy trên nhiều node với clock lệch nhau và rate limit thủng. Giả định không được ghi lại là giả định sẽ bị vi phạm trong im lặng. Mỗi mô hình nên đi kèm phần "mô hình này gãy khi..." trong design doc.

**Mô hình hóa một lần rồi đóng đinh.** Dữ liệu tăng 100 lần, câu hỏi nghiệp vụ đổi, nhưng mô hình 3 năm trước vẫn được giữ vì "đang chạy được". Mô hình có vòng đời; nghi thức xét lại mô hình khi requirement đổi cũng quan trọng như refactor code.

## 10. Best Practices

**Nên:**

- Viết mô hình ra giấy **trước khi mở editor**: đối tượng là gì, quan hệ hình gì, ràng buộc là mệnh đề nào, câu hỏi là gì. Năm dòng này tiết kiệm năm ngày code sai hướng — và chính là phần "data model" mà mọi design doc tử tế yêu cầu.
- Đặt tên bài toán chuẩn nếu có thể ("đây là topological sort") — cái tên mở khóa toàn bộ tài liệu, thuật toán, và cả từ khóa để search lỗi.
- Chủ động tìm phản ví dụ cho mô hình của mình (chương 03): "trường hợp nào làm mô hình này trả lời sai?" — người tìm ra là bạn thì rẻ, là production thì đắt.
- Giữ danh sách giả định đơn giản hóa ngay trong design doc, mỗi giả định kèm một dòng "sẽ gãy khi...".
- Khi bí, thử đổi cấu trúc: bài đang bí dưới dạng mảng có thể tầm thường dưới dạng đồ thị. Reformulation là kỹ thuật giải bài mạnh nhất mà không sách thuật toán nào dạy tử tế.

**Không nên:**

- Không thêm chi tiết vào mô hình "cho giống thật" khi câu hỏi chưa đòi hỏi — mỗi chi tiết là một khoản nợ phân tích.
- Không tin mô hình chỉ vì nó đẹp: mô hình thanh lịch trả lời sai câu hỏi vẫn là mô hình sai.
- Không bỏ qua bước phát biểu ràng buộc thành mệnh đề chính xác — mơ hồ ở bước 3 sẽ thành bug ở bước 5, và thành cãi nhau với QA ở bước 7.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Cùng hệ thống thanh toán, hãy chỉ ra hai câu hỏi nghiệp vụ khác nhau dẫn đến hai mô hình toán khác nhau — và điều gì xảy ra nếu dùng nhầm mô hình của câu này cho câu kia?
2. Token bucket đã đơn giản hóa những gì so với thực tế, và từng giả định đó gãy trong tình huống production nào?
3. Word Ladder là bài chuỗi hay bài đồ thị? Tín hiệu nào trong đề bài tiết lộ điều đó — và bạn tổng quát hóa tín hiệu ấy thành quy tắc gì?

---

*Chương tiếp theo: [05 — Counting & Combinatorics](/series/math-for-engineers/level-2-discrete-mathematics/05-counting-combinatorics/), nơi ta trang bị cấu trúc toán đầu tiên trong bảng tra: nghệ thuật đếm — nền của phân tích thuật toán, xác suất, và mọi câu hỏi "có bao nhiêu trường hợp".*
