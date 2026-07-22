+++
title = "Chương 08 — Graph Theory: Mô hình của mọi mối quan hệ"
date = "2026-07-20T08:20:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 2 – Discrete Mathematics
> Yêu cầu trước: Chương 02 (Set Theory, Functions & Relations), Chương 03 (Proof Techniques), Chương 07 (Trees)

---

## 1. Problem Statement

Một buổi sáng thứ Hai, hệ thống thanh toán của một công ty fintech đứng hình. Không crash, không error log, không CPU spike — chỉ đơn giản là mọi transaction treo vô hạn. Nguyên nhân: transaction A giữ lock trên bảng `accounts` và chờ lock trên `ledger`; transaction B giữ lock trên `ledger` và chờ lock trên `orders`; transaction C giữ `orders` và chờ `accounts`. Ba transaction chờ nhau theo một vòng tròn khép kín — **deadlock**.

PostgreSQL phát hiện tình huống này trong vài giây và tự động kill một transaction để phá vòng. Làm thế nào? Nó không "hiểu" nghiệp vụ thanh toán. Nó chỉ xây một cấu trúc gọi là **wait-for graph**: mỗi transaction là một điểm, "A chờ B" là một mũi tên từ A đến B — rồi hỏi một câu hỏi thuần túy toán học: *đồ thị này có chu trình không?*

Cùng ngày hôm đó, ở những hệ thống khác:

- `go mod` phải quyết định thứ tự build hàng trăm package sao cho mọi dependency được build trước khi được import.
- Google Maps tìm đường ngắn nhất giữa hai điểm trong đồ thị hàng trăm triệu đoạn đường.
- Facebook gợi ý "người bạn có thể quen" — những người cách bạn đúng 2 bước trên đồ thị bạn bè.
- Kubernetes scheduler ghép hàng nghìn pod vào hàng trăm node sao cho thỏa mãn ràng buộc tài nguyên.

Bốn bài toán trông chẳng liên quan gì đến nhau. Nhưng cả bốn đều là **cùng một loại đối tượng toán học**: một tập các thực thể và một tập các mối quan hệ giữa chúng. Đó là graph.

Bài toán của chương này: **làm sao mô hình hóa "mối quan hệ" một cách đủ tổng quát để một thuật toán viết một lần dùng được cho mọi domain — và những thuật toán nền tảng nào trả lời được những câu hỏi quan trọng nhất trên mô hình đó?**

Nếu không có graph theory, mỗi domain phải phát minh lại từ đầu: đội database viết riêng thuật toán tìm vòng chờ lock, đội build tool viết riêng thuật toán sắp thứ tự package, đội bản đồ viết riêng thuật toán tìm đường — và cả ba đều đang giải cùng một bài toán mà không biết, với ba bộ bug khác nhau.

## 2. Trực giác

### Bỏ hết chi tiết, chỉ giữ lại "cái gì nối với cái gì"

Nhìn lại bốn bài toán ở trên và lột bỏ mọi chi tiết domain:

| Domain | "Điểm" (vertex) | "Mũi tên/đường nối" (edge) | Câu hỏi |
|---|---|---|---|
| Deadlock detection | transaction | "chờ lock của" | có chu trình không? |
| go mod / npm | package | "import" | sắp thứ tự thế nào? |
| Google Maps | giao lộ | đoạn đường (có độ dài) | đường ngắn nhất? |
| Social network | người | "là bạn của" | ai cách tôi 2 bước? |
| Kubernetes | pod, node | "có thể chạy trên" | ghép cặp tối đa? |

Graph chính là phép trừu tượng hóa này: **một tập đỉnh (vertices) và một tập cạnh (edges) nối các cặp đỉnh**. Không hơn. Chính vì "không hơn" nên nó tổng quát: bất cứ khi nào bạn có *các thực thể* và *quan hệ đôi một giữa chúng*, bạn có một graph — dù đề bài không hề nhắc đến chữ "graph".

Ở chương 02 ta đã gặp khái niệm quan hệ (relation) trên một tập hợp. Graph là cách nhìn *hình học* của cùng khái niệm đó: quan hệ R ⊆ V × V vẽ ra thành các mũi tên. Cây ở chương 07 hóa ra chỉ là một trường hợp đặc biệt — graph liên thông không có chu trình. Toàn bộ chương này là câu chuyện: điều gì xảy ra khi ta bỏ đi các ràng buộc của cây và cho phép quan hệ *tùy ý*.

### Ba trục phân loại

Ba câu hỏi quyết định bạn đang làm việc với loại graph nào:

**Quan hệ có hướng không?** "A import B" không có nghĩa "B import A" → **directed graph** (đồ thị có hướng). "A là bạn của B" thì đối xứng → **undirected**. Chọn sai trục này là lỗi mô hình hóa phổ biến nhất: mô hình Twitter follow (có hướng) như Facebook friend (vô hướng) sẽ cho kết quả gợi ý sai hoàn toàn.

**Cạnh có trọng số không?** Đoạn đường có độ dài, network link có latency → **weighted**. Quan hệ chỉ có/không → unweighted. Trọng số quyết định thuật toán: shortest path trên graph không trọng số là BFS, có trọng số là Dijkstra — hai thuật toán khác hẳn nhau.

**Có chu trình không?** Directed graph không có chu trình gọi là **DAG** (Directed Acyclic Graph) — cấu trúc quan trọng bậc nhất trong kỹ thuật phần mềm: dependency graph, Git commit history, Airflow pipeline, blockchain đều là DAG. Sự tồn tại hay vắng mặt của chu trình thay đổi cả tập câu hỏi có nghĩa: "sắp thứ tự topo" chỉ có nghĩa trên DAG; "deadlock" chính là sự xuất hiện của chu trình ở nơi lẽ ra phải là DAG.

### Trực giác về các câu hỏi kinh điển

Hầu hết thuật toán graph trả lời một trong năm câu hỏi, và mỗi câu hỏi có một hình ảnh trực giác:

1. **"Đi từ A tới B được không? Có những vùng nào?"** — thả một giọt mực vào đỉnh A, mực loang đến đâu thì đó là vùng liên thông của A. (BFS/DFS)
2. **"Đường ngắn nhất từ A tới B?"** — thả sóng nước từ A lan đều mọi hướng; sóng chạm B lúc nào thì đó là khoảng cách. (BFS cho unweighted, Dijkstra cho weighted)
3. **"Sắp thứ tự sao cho mọi mũi tên chỉ về phía sau?"** — kéo căng graph thành một hàng dọc. (Topological sort)
4. **"Nối tất cả các điểm với tổng chi phí rẻ nhất?"** — xây mạng cáp quang nối mọi thành phố với ít km cáp nhất. (MST)
5. **"Đẩy được tối đa bao nhiêu 'lưu lượng' từ nguồn đến đích?"** — nước chảy qua hệ thống ống có tiết diện giới hạn. (Max flow)

## 3. First Principles

### Định nghĩa từ tập hợp

Từ chương 02, ta đã có tập hợp và quan hệ. Graph chỉ cần đúng hai nguyên liệu đó:

> Một graph G = (V, E), trong đó V là tập hữu hạn các đỉnh, và E là tập các cạnh.
> - Directed: E ⊆ V × V — mỗi cạnh là một cặp *có thứ tự* (u, v).
> - Undirected: mỗi cạnh là một tập {u, v} — không có thứ tự.
> - Weighted: kèm hàm trọng số w: E → ℝ.

Ký hiệu quen thuộc: n = |V| (số đỉnh), m = |E| (số cạnh). Mọi độ phức tạp trong chương này viết theo hai biến n và m — vì graph "thưa" (m ≈ n, như mạng đường bộ) và graph "dày" (m ≈ n², như ma trận tương quan) hành xử rất khác nhau, và chọn thuật toán/biểu diễn sai loại là nguồn lỗi hiệu năng kinh điển.

Vài khái niệm dẫn xuất, tất cả đều định nghĩa được từ V và E:

- **Degree** của đỉnh v: số cạnh chạm v. Directed graph tách thành in-degree (số mũi tên đi vào) và out-degree (đi ra). In-degree = 0 nghĩa là "không phụ thuộc ai" — chìa khóa của Kahn's algorithm ở mục 5.
- **Path**: dãy đỉnh v₀, v₁, ..., vₖ trong đó mỗi cặp liên tiếp là một cạnh. **Khoảng cách** d(u, v) là số cạnh (hoặc tổng trọng số) của path ngắn nhất.
- **Cycle**: path có v₀ = vₖ và độ dài ≥ 1 (directed) — vòng tròn khép kín.
- **Connected**: undirected graph liên thông nếu mọi cặp đỉnh có path nối nhau.

### Nếu không có phép trừu tượng này thì sao?

Câu trả lời không phải là "không giải được bài toán" — con người vẫn tìm được đường đi trước khi Euler tồn tại. Câu trả lời là **không tái sử dụng được lời giải**. Dijkstra viết thuật toán của ông năm 1956 cho bài toán đường đi giữa hai thành phố Hà Lan; hôm nay chính thuật toán đó chạy trong network routing (OSPF), trong game pathfinding, trong Kubernetes khi tính toán chi phí. Điều đó chỉ khả thi vì thuật toán được phát biểu trên G = (V, E) trừu tượng, không phải trên "thành phố và quốc lộ".

Đây là cùng nguyên tắc với interface trong lập trình: thuật toán graph là hàm nhận interface `Graph`, còn bản đồ, dependency, wait-for lock chỉ là các struct implement nó. Graph theory là **dependency inversion ở tầng toán học**.

### Khi nào KHÔNG mô hình hóa bằng graph?

- Khi quan hệ không phải đôi một. "Ba người cùng thuộc một nhóm chat" là quan hệ ba ngôi — ép vào graph (vẽ 3 cạnh đôi một) sẽ mất thông tin "cùng nhóm". Mô hình đúng là hypergraph hoặc bipartite graph (người — nhóm).
- Khi cấu trúc có ràng buộc mạnh hơn mà bạn nên khai thác: dữ liệu phân cấp thuần túy là **cây** (chương 07) — dùng thuật toán cây đơn giản và nhanh hơn thuật toán graph tổng quát.
- Khi dữ liệu là dạng bảng quan hệ truy vấn theo thuộc tính — SQL và index (chương 07) hiệu quả hơn graph traversal. Graph database (Neo4j) chỉ thắng khi câu hỏi của bạn *là* câu hỏi traversal ("bạn của bạn của bạn"), vì JOIN đệ quy sâu trên RDBMS đắt.

## 4. Mathematical Model

### Biểu diễn: cấu trúc dữ liệu nào cho G = (V, E)?

Định nghĩa toán học nói graph là hai tập hợp — nhưng máy tính cần bố trí bộ nhớ cụ thể, và có hai lựa chọn kinh điển với trade-off đối lập nhau.

**Adjacency matrix** — ma trận A kích thước n×n, A[u][v] = 1 (hoặc = trọng số) nếu có cạnh u→v:

```
      A  B  C  D
   A  0  1  1  0        A ──→ B
   B  0  0  0  1        │
   C  0  0  0  1        ↓
   D  0  0  0  0        C ──→ D ←── B
```

**Adjacency list** — mỗi đỉnh giữ danh sách hàng xóm của nó:

```
A → [B, C]
B → [D]
C → [D]
D → []
```

```go
// Adjacency list — biểu diễn mặc định trong thực tế.
// Đỉnh đánh số 0..n-1; với ID dạng string, thêm một map[string]int để quy đổi.
type Graph struct {
    n   int
    adj [][]int // adj[u] = danh sách đỉnh kề của u
}

func NewGraph(n int) *Graph {
    return &Graph{n: n, adj: make([][]int, n)}
}

func (g *Graph) AddEdge(u, v int) {
    g.adj[u] = append(g.adj[u], v)
    // Với undirected graph, thêm chiều ngược lại:
    // g.adj[v] = append(g.adj[v], u)
}

// Adjacency matrix — chỉ dùng khi graph dày hoặc n nhỏ.
type MatrixGraph struct {
    n int
    a [][]bool // a[u][v] = true nếu có cạnh u→v
}

func (g *MatrixGraph) HasEdge(u, v int) bool {
    return g.a[u][v] // O(1) — điểm mạnh duy nhất nhưng quan trọng
}
```

| Tiêu chí | Adjacency List | Adjacency Matrix |
|---|---|---|
| Bộ nhớ | O(n + m) | O(n²) |
| "Có cạnh u→v không?" | O(degree(u)) | **O(1)** |
| Duyệt mọi hàng xóm của u | **O(degree(u))** | O(n) — kể cả khi u chỉ có 2 hàng xóm |
| Duyệt toàn bộ cạnh | O(n + m) | O(n²) |
| Phù hợp | graph thưa (đại đa số thực tế) | graph dày, n nhỏ, cần tra cạnh O(1) |

Con số biết nói: đồ thị bạn bè 1 tỷ người, trung bình 200 bạn/người. Adjacency list: ~10⁹ + 2×10¹¹ ô nhớ — lớn nhưng khả thi khi phân tán. Adjacency matrix: 10¹⁸ ô — **một exabyte cho toàn số 0**. Graph thực tế hầu hết là thưa, nên adjacency list là mặc định; ma trận chỉ xuất hiện khi n nhỏ (Floyd-Warshall) hoặc khi bản thân phép toán là đại số tuyến tính trên ma trận kề (PageRank — chương 17).

### DAG và thứ tự topo — mô hình của "phụ thuộc"

Một **topological order** của directed graph là cách xếp mọi đỉnh thành một hàng sao cho mọi cạnh đều chỉ từ trái sang phải. Đây chính là "thứ tự build hợp lệ" của dependency graph.

Định lý trung tâm: **directed graph có topological order khi và chỉ khi nó không có chu trình (là DAG)**.

Chiều thuận dễ thấy: nếu có chu trình A → B → ... → A thì A phải đứng trước B và B phải đứng trước A — mâu thuẫn, không tồn tại thứ tự nào. *Đây chính là phát biểu toán học của câu "circular dependency thì không build được"* — không phải hạn chế của npm hay Go toolchain, mà là điều bất khả thi về mặt logic.

Chiều ngược (DAG thì luôn sắp được) chứng minh bằng một bổ đề đáng nhớ:

> **Bổ đề:** mọi DAG hữu hạn, khác rỗng đều có ít nhất một đỉnh với in-degree = 0.
>
> **Chứng minh (contradiction):** giả sử mọi đỉnh đều có mũi tên đi vào. Xuất phát từ đỉnh bất kỳ, đi ngược mũi tên liên tục. Vì mọi đỉnh đều có cạnh vào, hành trình không bao giờ kẹt; vì số đỉnh hữu hạn, đến bước thứ n+1 phải gặp lại một đỉnh đã qua (nguyên lý chuồng bồ câu — chương 05) → tồn tại chu trình → mâu thuẫn với giả thiết DAG. ∎

Bổ đề này cho ngay thuật toán: lấy đỉnh in-degree 0 xếp trước, xóa nó khỏi graph (phần còn lại vẫn là DAG), lặp lại. Đó chính là Kahn's algorithm ở mục 5 — một ví dụ đẹp về việc *chứng minh tồn tại tự nó là thuật toán*.

### Cut property — mô hình của "nối rẻ nhất"

Cho weighted undirected graph liên thông. **Spanning tree** là tập cạnh nối tất cả các đỉnh mà không tạo chu trình (đúng n−1 cạnh — chương 07); **Minimum Spanning Tree (MST)** là spanning tree có tổng trọng số nhỏ nhất. Một **cut** là cách chia V thành hai phần khác rỗng (S, V∖S).

> **Cut property:** với mọi cut, cạnh nhẹ nhất bắc qua cut đó thuộc về một MST nào đó.

Đây là định lý làm nền cho *mọi* thuật toán MST, chứng minh ở mục 5. Ý nghĩa trực giác: chia các thành phố thành hai nhóm bất kỳ — sợi cáp rẻ nhất nối hai nhóm chắc chắn đáng mua. Chính nó là lý do greedy đúng ở bài toán này, trong khi greedy sai ở vô số bài toán khác (chương 15).

## 5. Thuật toán

### BFS — khám phá theo từng lớp sóng

Breadth-First Search xuất phát từ đỉnh s và thăm các đỉnh theo **lớp**: lớp 0 là {s}, lớp 1 là các đỉnh kề s, lớp 2 là các đỉnh kề lớp 1 mà chưa thăm... như sóng nước lan từ tâm.

```go
// BFS trả về dist[v] = số cạnh trên đường ngắn nhất từ s đến v (-1 nếu không tới được).
func BFS(g *Graph, s int) []int {
    dist := make([]int, g.n)
    for i := range dist {
        dist[i] = -1 // chưa thăm
    }
    dist[s] = 0
    queue := []int{s}
    for len(queue) > 0 {
        u := queue[0]
        queue = queue[1:]
        for _, v := range g.adj[u] {
            if dist[v] == -1 { // mỗi đỉnh vào queue đúng một lần
                dist[v] = dist[u] + 1
                queue = append(queue, v)
            }
        }
    }
    return dist
}
```

Mỗi đỉnh vào queue tối đa một lần, mỗi cạnh xét tối đa một lần (hai lần với undirected) → **O(n + m)** — tuyến tính theo kích thước graph, không thể tốt hơn về bậc vì ít nhất phải đọc hết input.

**Vì sao BFS tìm đúng shortest path trên graph không trọng số?** Đây là chỗ đáng chứng minh, vì tính đúng đắn không hiển nhiên — nó đến từ một invariant (chương 03):

> **Invariant:** tại mọi thời điểm, các đỉnh trong queue có dist chỉ nhận tối đa hai giá trị liên tiếp k và k+1, xếp theo thứ tự không giảm; và với mọi đỉnh đã gán dist, giá trị đó bằng đúng khoảng cách thật.
>
> **Chứng minh (induction theo lớp):** lớp 0 đúng hiển nhiên: dist[s] = 0. Giả sử mọi đỉnh có khoảng cách thật ≤ k đều đã được gán dist đúng, và queue hiện chứa đúng các đỉnh lớp k (rồi dần dần lớp k+1). Xét đỉnh v được gán dist = k+1 khi pop một đỉnh u thuộc lớp k:
> - dist[v] không thể *nhỏ hơn* khoảng cách thật, vì tồn tại đường s→u dài k cộng cạnh (u,v): khoảng cách thật của v ≤ k+1.
> - dist[v] không thể *lớn hơn* khoảng cách thật: nếu khoảng cách thật của v là j ≤ k thì v kề một đỉnh lớp j−1 ≤ k−1, mà theo giả thiết quy nạp mọi đỉnh lớp ≤ k−1 đã được pop và đã duyệt hết hàng xóm — v phải được gán dist từ trước, mâu thuẫn với việc dist[v] = −1.
>
> Vậy mọi đỉnh ở lớp k có dist đúng bằng k. ∎

Điểm bản chất: chứng minh dựa hoàn toàn vào việc queue xử lý **lớp k xong hết mới đến lớp k+1** — tức FIFO. Thay queue bằng stack, invariant sụp đổ và ta có DFS, thứ *không* tìm shortest path. Cấu trúc dữ liệu không phải chi tiết cài đặt; nó **là** thuật toán.

### DFS — đi sâu đến cùng, và món quà phụ: phát hiện chu trình

Depth-First Search đi theo một nhánh đến khi kẹt mới quay lui. Tự nhiên nhất là viết đệ quy — call stack chính là "sợi chỉ Ariadne" ghi nhớ đường quay về.

Sức mạnh thật của DFS nằm ở **ba trạng thái** của đỉnh: chưa thăm (trắng), *đang trong stack đệ quy* (xám), đã xong (đen). Trạng thái xám là thứ BFS không có, và nó cho khả năng phát hiện chu trình:

```go
// HasCycle phát hiện chu trình trong directed graph — chính là thuật toán
// mà database dùng trên wait-for graph để phát hiện deadlock.
func HasCycle(g *Graph) bool {
    const (
        white = 0 // chưa thăm
        gray  = 1 // đang trên đường đi hiện tại (trong stack đệ quy)
        black = 2 // đã duyệt xong toàn bộ nhánh con
    )
    state := make([]int, g.n)
    var dfs func(u int) bool
    dfs = func(u int) bool {
        state[u] = gray
        for _, v := range g.adj[u] {
            if state[v] == gray {
                return true // gặp lại đỉnh đang trên đường đi → chu trình
            }
            if state[v] == white && dfs(v) {
                return true
            }
        }
        state[u] = black
        return false
    }
    for u := 0; u < g.n; u++ {
        if state[u] == white && dfs(u) {
            return true
        }
    }
    return false
}
```

Vì sao đúng? Cạnh u→v chạm vào đỉnh xám v nghĩa là v là "tổ tiên" của u trên đường đi hiện tại — tức tồn tại đường v ⇝ u, ghép với cạnh u→v thành chu trình. Ngược lại, chạm đỉnh *đen* thì không sao: nhánh của v đã đóng hoàn toàn, không thể quay lại u. Đây gọi là **back edge** — và toàn bộ phân loại cạnh của DFS (tree edge: cạnh đi tới đỉnh trắng; back edge: tới đỉnh xám; forward/cross edge: tới đỉnh đen) chỉ là cách gọi tên ba trạng thái đó khi bị một cạnh chạm vào. Định lý gọn: **directed graph có chu trình ⇔ DFS gặp back edge**.

Chi phí: O(n + m), như BFS. Chọn giữa hai thuật toán không phải theo tốc độ mà theo *câu hỏi*: cần khoảng cách/lớp → BFS; cần cấu trúc (chu trình, thứ tự, thành phần) → DFS.

### Topological Sort — Kahn's algorithm

Từ bổ đề "DAG luôn có đỉnh in-degree 0" ở mục 4, Kahn's algorithm chỉ là bổ đề đó chạy lặp:

```go
// TopoSort trả về thứ tự topo, hoặc nil nếu graph có chu trình.
// Đây là trái tim của go mod, npm install, Airflow scheduler.
func TopoSort(g *Graph) []int {
    indeg := make([]int, g.n)
    for u := 0; u < g.n; u++ {
        for _, v := range g.adj[u] {
            indeg[v]++
        }
    }
    queue := []int{}
    for u := 0; u < g.n; u++ {
        if indeg[u] == 0 {
            queue = append(queue, u) // các đỉnh "không phụ thuộc ai"
        }
    }
    order := make([]int, 0, g.n)
    for len(queue) > 0 {
        u := queue[0]
        queue = queue[1:]
        order = append(order, u)
        for _, v := range g.adj[u] {
            indeg[v]-- // "xóa" u: mọi phụ thuộc vào u coi như đã thỏa
            if indeg[v] == 0 {
                queue = append(queue, v)
            }
        }
    }
    if len(order) < g.n {
        return nil // còn đỉnh không bao giờ đạt in-degree 0 → có chu trình
    }
    return order
}
```

Chi tiết tinh tế cuối hàm: nếu có chu trình, các đỉnh trên chu trình chờ nhau vĩnh viễn, in-degree không bao giờ về 0, `order` hụt so với n. Vậy Kahn **vừa sắp thứ tự vừa phát hiện chu trình miễn phí** — đó là lý do build tool báo được "circular dependency detected" kèm luôn danh sách package mắc kẹt. Chi phí O(n + m). Một hệ quả thực dụng: dùng queue cho các đỉnh in-degree 0 nghĩa là các đỉnh "sẵn sàng" cùng lúc có thể build **song song** — mức độ song song hóa của build system chính là độ rộng của từng lớp trong topo order.

### Dijkstra — shortest path có trọng số, và vì sao greedy dám đúng

Khi cạnh có trọng số, "lan theo lớp" của BFS không còn đúng: đường 2 cạnh trọng số 1+1 ngắn hơn đường 1 cạnh trọng số 10. Dijkstra (1956) tổng quát hóa: thay vì lan theo *số cạnh*, lan theo *tổng trọng số* — luôn "chốt sổ" đỉnh chưa chốt có khoảng cách tạm nhỏ nhất.

```go
import "container/heap"

type item struct{ v, d int }
type pq []item

func (p pq) Len() int            { return len(p) }
func (p pq) Less(i, j int) bool  { return p[i].d < p[j].d }
func (p pq) Swap(i, j int)       { p[i], p[j] = p[j], p[i] }
func (p *pq) Push(x any)         { *p = append(*p, x.(item)) }
func (p *pq) Pop() any           { old := *p; x := old[len(old)-1]; *p = old[:len(old)-1]; return x }

type WEdge struct{ to, w int }

// Dijkstra: shortest path từ s trên graph có trọng số KHÔNG ÂM.
func Dijkstra(adj [][]WEdge, s int) []int {
    const inf = int(^uint(0) >> 1)
    dist := make([]int, len(adj))
    for i := range dist {
        dist[i] = inf
    }
    dist[s] = 0
    q := &pq{{s, 0}}
    for q.Len() > 0 {
        it := heap.Pop(q).(item)
        if it.d > dist[it.v] {
            continue // bản ghi cũ đã bị cải thiện — bỏ qua (lazy deletion)
        }
        for _, e := range adj[it.v] {
            if nd := it.d + e.w; nd < dist[e.to] {
                dist[e.to] = nd // "relax" cạnh: tìm thấy đường tốt hơn
                heap.Push(q, item{e.to, nd})
            }
        }
    }
    return dist
}
```

Priority queue (heap — chương 16) cho phép lấy "đỉnh tạm gần nhất" trong O(log n); tổng chi phí **O((n + m) log n)**. Không có heap, mỗi lần chọn min tốn O(n) → O(n²) — vẫn đúng, chỉ chậm trên graph thưa lớn.

**Vì sao greedy đúng ở đây?** Khẳng định cần chứng minh: khi đỉnh u được pop lần đầu (được "chốt"), dist[u] là khoảng cách thật δ(u).

> **Chứng minh (contradiction):** giả sử u là đỉnh *đầu tiên* bị chốt sai: dist[u] > δ(u). Xét đường đi ngắn nhất thật sự P từ s đến u. Vì s được chốt đúng còn u chưa, trên P tồn tại cạnh đầu tiên (x, y) vượt biên giới: x đã chốt (đúng, vì u là đỉnh sai đầu tiên), y chưa chốt. Khi x được chốt, thuật toán đã relax cạnh (x, y), nên:
>
> dist[y] ≤ δ(x) + w(x, y) = δ(y)   (y nằm trên đường ngắn nhất tới u)
>
> Vì **trọng số không âm**, đoạn còn lại y ⇝ u trên P có chi phí ≥ 0, nên δ(y) ≤ δ(u) < dist[u]. Chuỗi lại: dist[y] ≤ δ(y) ≤ δ(u) < dist[u] — tức y có dist tạm nhỏ hơn u, heap phải pop y trước u. Mâu thuẫn với việc u được chốt lúc này. ∎

Toàn bộ chứng minh treo trên đúng một mắt xích: *"đoạn còn lại có chi phí ≥ 0"*. **Cho phép cạnh âm, mắt xích đứt và Dijkstra sai thật sự** — một đường tạm-thời-đắt có thể "gỡ vốn" nhờ cạnh âm phía sau, nhưng Dijkstra đã chốt sổ và không bao giờ nhìn lại:

```
      s ──(2)──→ A
      s ──(3)──→ B ──(−2)──→ A
      Dijkstra chốt A với dist = 2, nhưng đường thật qua B chỉ tốn 1.
```

Đây là bài học tổng quát về greedy (chương 15): greedy đúng không phải vì "tham thường thắng" mà vì cấu trúc bài toán *bảo lãnh* cho nó — ở đây là tính không âm của trọng số. Mất bảo lãnh, mất tất cả.

**Bellman-Ford** — khi có cạnh âm, ta trả giá bằng cách khiêm tốn hơn: không chốt sổ gì cả, cứ relax *tất cả m cạnh*, lặp n−1 vòng. Trực giác về tính đúng: sau vòng thứ k, mọi đường đi ngắn nhất dùng ≤ k cạnh đã được tìm thấy (induction theo k — cùng khung chứng minh với BFS); đường đi ngắn nhất không lặp đỉnh dài tối đa n−1 cạnh, nên n−1 vòng là đủ. Chi phí O(n·m) — đắt hơn Dijkstra đáng kể, đó là giá của việc chấp nhận cạnh âm. Món quà kèm theo: nếu vòng thứ n *vẫn* relax được, graph chứa chu trình âm — vòng lặp "càng đi càng lời" khiến khái niệm shortest path vô nghĩa. Trong hệ thống phát hiện arbitrage tiền tệ, "chu trình âm" chính là cơ hội kiếm lời: đổi tỷ giá lấy −log(rate) làm trọng số thì chuỗi nhân tỷ giá > 1 trở thành chu trình âm.

**A\*** — Dijkstra lan đều mọi hướng như sóng nước, kể cả hướng ngược với đích. Nếu có hàm *ước lượng* h(v) ≈ khoảng cách còn lại từ v đến đích, ta ưu tiên pop đỉnh có dist[v] + h(v) nhỏ nhất — sóng bị "hút" về phía đích. Điều kiện để kết quả vẫn tối ưu: h **admissible** — không bao giờ ước lượng *quá* khoảng cách thật (ví dụ: khoảng cách chim bay luôn ≤ khoảng cách đường bộ). Trực giác: ước lượng lạc quan có thể làm ta tốn công khám phá thừa, nhưng không bao giờ khiến ta chốt nhầm đường; ước lượng bi quan có thể che mất đường tối ưu. h ≡ 0 admissible tầm thường → A* suy biến về Dijkstra. Game pathfinding và hệ thống bản đồ đều là A* hoặc hậu duệ của nó (chương 27).

### MST — Kruskal, Prim, và cut property

Chứng minh cut property đã hứa ở mục 4:

> **Chứng minh (exchange argument):** xét cut (S, V∖S) bất kỳ và e = cạnh nhẹ nhất bắc qua cut. Giả sử MST T không chứa e. Thêm e vào T tạo đúng một chu trình (tính chất cây — chương 07); chu trình này bắc qua cut tại e và ít nhất một cạnh khác e′ (vì chu trình đi từ S sang V∖S rồi phải quay về). Thay e′ bằng e: kết quả T′ vẫn liên thông, vẫn n−1 cạnh → vẫn là spanning tree, và w(T′) = w(T) − w(e′) + w(e) ≤ w(T) vì e nhẹ nhất qua cut. Vậy T′ là MST chứa e. ∎

Phép "tháo một cạnh, lắp cạnh rẻ hơn" này là exchange argument — khuôn mẫu chứng minh greedy sẽ gặp lại ở chương 15. Hai thuật toán MST kinh điển chỉ là hai cách *chọn cut* khác nhau:

**Kruskal**: sort mọi cạnh tăng dần theo trọng số, duyệt lần lượt, nhận cạnh nếu nó không tạo chu trình với các cạnh đã nhận. Mỗi cạnh được nhận đều là cạnh nhẹ nhất bắc qua cut "thành phần chứa u | phần còn lại" — cut property bảo lãnh. Câu hỏi "hai đỉnh đã cùng thành phần chưa?" cần trả lời hàng trăm nghìn lần thật nhanh — đó là việc của **Union-Find**:

```go
// Union-Find (Disjoint Set Union) — trả lời "cùng nhóm không?" và "gộp nhóm"
// gần như O(1) amortized. Chi tiết cấu trúc này ở chương 24.
type DSU struct{ parent []int }

func NewDSU(n int) *DSU {
    p := make([]int, n)
    for i := range p {
        p[i] = i // ban đầu mỗi đỉnh là một nhóm riêng
    }
    return &DSU{p}
}

func (d *DSU) Find(x int) int {
    for d.parent[x] != x {
        d.parent[x] = d.parent[d.parent[x]] // path compression: nén đường về gốc
        x = d.parent[x]
    }
    return x
}

func (d *DSU) Union(x, y int) bool {
    rx, ry := d.Find(x), d.Find(y)
    if rx == ry {
        return false // đã cùng nhóm → cạnh này sẽ tạo chu trình
    }
    d.parent[rx] = ry
    return true
}

type Edge struct{ u, v, w int }

// Kruskal: giả định edges đã sort tăng dần theo w.
func Kruskal(n int, edges []Edge) (total int, mst []Edge) {
    d := NewDSU(n)
    for _, e := range edges {
        if d.Union(e.u, e.v) { // nhận cạnh nếu nối hai thành phần khác nhau
            total += e.w
            mst = append(mst, e)
            if len(mst) == n-1 {
                break
            }
        }
    }
    return total, mst
}
```

Chi phí bị chặn bởi bước sort: **O(m log m)**. **Prim** đi hướng khác: nuôi một cây từ một đỉnh gốc, mỗi bước nhận cạnh nhẹ nhất nối cây với phần ngoài cây (cut = "cây | phần còn lại") — cấu trúc code giống Dijkstra đến kinh ngạc, chỉ khác khóa ưu tiên: Dijkstra dùng *tổng khoảng cách từ nguồn*, Prim dùng *trọng số một cạnh nối vào cây*. Cùng O((n + m) log n) với heap. Kruskal tiện khi có sẵn danh sách cạnh (và dễ chia batch); Prim hợp khi graph dày hoặc cần mọc cây từ điểm cho trước.

### Network Flow — mức nhập môn

Bài toán: directed graph với mỗi cạnh có **capacity** (tiết diện ống), một đỉnh nguồn s và đỉnh đích t. Hỏi: đẩy được tối đa bao nhiêu đơn vị lưu lượng từ s đến t, tôn trọng capacity mỗi ống và luật bảo toàn (lưu lượng vào mỗi đỉnh trung gian = lưu lượng ra)?

Định lý trung tâm — **Max-Flow Min-Cut**: lưu lượng cực đại từ s đến t **bằng đúng** tổng capacity của lát cắt hẹp nhất ngăn s khỏi t. Trực giác: mọi giọt nước từ s đến t đều phải chui qua bất kỳ lát cắt nào → flow ≤ mọi cut → max-flow ≤ min-cut; phần sâu sắc của định lý là dấu bằng luôn đạt được — nút cổ chai hẹp nhất *chính xác* quyết định thông lượng, không có tổn thất "ma sát" nào khác. Thuật toán Ford-Fulkerson tìm nó bằng cách lặp: tìm một đường còn dư sức chứa từ s đến t (bằng BFS), đẩy thêm flow, cập nhật sức chứa còn lại — kèm một kỹ thuật then chốt là *cạnh ngược* cho phép "hoàn tác" flow đã đẩy vụng.

Vì sao engineer nên biết khái niệm này dù hiếm khi tự cài? Vì một họ bài toán ghép cặp và phân bổ rất lớn quy về nó. **Bipartite matching**: n worker, m task, mỗi worker làm được một số task, ghép tối đa bao nhiêu cặp worker–task? Thêm nguồn s nối tới mọi worker, mọi task nối tới đích t, mọi cạnh capacity 1 — max-flow chính là số cặp ghép tối đa. Kubernetes scheduling, phân bổ ads vào slot, ghép rider–driver đều mang cấu trúc này. Và min-cut trả lời câu hỏi phòng thủ: "cắt tối thiểu bao nhiêu link thì hệ thống đứt đôi?" — chính là điểm yếu nhất của network topology.

### Bảng tổng kết

| Thuật toán | Câu hỏi | Điều kiện | Chi phí |
|---|---|---|---|
| BFS | shortest path, phân lớp | unweighted | O(n + m) |
| DFS | chu trình, thành phần, thứ tự | — | O(n + m) |
| Kahn (topo sort) | thứ tự phụ thuộc | DAG | O(n + m) |
| Dijkstra | shortest path | trọng số **≥ 0** | O((n+m) log n) |
| Bellman-Ford | shortest path, phát hiện chu trình âm | không có chu trình âm (trên đường đi) | O(n·m) |
| A* | shortest path tới một đích | h admissible | ≤ Dijkstra trong thực tế |
| Kruskal / Prim | nối rẻ nhất (MST) | undirected, liên thông | O(m log m) |
| Ford-Fulkerson (+BFS) | max flow / matching | capacity nguyên | O(n·m²) bản cơ bản |

## 6. Trade-off

**Adjacency list vs matrix** — như phân tích ở mục 4: list thắng tuyệt đối trên graph thưa (bộ nhớ O(n+m), duyệt hàng xóm theo degree thật); matrix chỉ đáng khi cần tra cạnh O(1) trên graph nhỏ/dày, hoặc khi thuật toán vốn là phép toán ma trận. Lưu ý hằng số (bài học chương 10): adjacency list bằng `[][]int` trong Go có cache locality tốt hơn nhiều so với `map[int][]int` hay linked list — với graph nóng trong RAM, khác biệt vài lần là bình thường.

**BFS vs DFS** — cùng O(n + m), khác nhau ở *bộ nhớ và hình dạng khám phá*. BFS giữ cả một lớp trong queue: trên graph "nở" nhanh (social network, mỗi đỉnh 200 hàng xóm), lớp 3 đã là hàng triệu đỉnh — bộ nhớ là điểm chết của BFS. DFS chỉ giữ một đường đi trong stack, nhưng đường đó có thể sâu bằng n — đệ quy trên graph triệu đỉnh sẽ nổ stack (goroutine stack của Go tự giãn nên đỡ hơn C/Java, nhưng bản lặp với stack tường minh vẫn an toàn hơn cho graph lớn). Cần khoảng cách → bắt buộc BFS; cần cấu trúc → DFS.

**Dijkstra vs Bellman-Ford** — tốc độ đổi lấy tính tổng quát: Dijkstra nhanh nhưng đòi trọng số không âm; Bellman-Ford chậm hơn cỡ n lần nhưng chịu được cạnh âm và phát hiện chu trình âm. Chọn Bellman-Ford khi và chỉ khi mô hình *thật sự* có cạnh âm — đại đa số bài toán thực tế (khoảng cách, latency, chi phí) thì không.

**Dijkstra vs A\*** — A* cần một heuristic tốt, thứ chỉ tồn tại khi đồ thị có "hình học" (tọa độ, khoảng cách chim bay). Dependency graph không có khái niệm "hướng về đích" nên A* vô dụng ở đó. Heuristic càng sát thực tế, A* càng nhanh; heuristic ≡ 0 thì A* *là* Dijkstra.

**Một nguồn vs mọi cặp** — Dijkstra từ một nguồn là O((n+m) log n); cần khoảng cách giữa *mọi cặp* đỉnh có thể chạy Dijkstra n lần, hoặc Floyd-Warshall O(n³) gọn gàng trên adjacency matrix khi n nhỏ (vài trăm đỉnh). Với n lớn thì cả hai đều bất khả thi — hệ thống bản đồ thật phải dùng precompute + phân cấp (contraction hierarchies — chương 27), một lời nhắc rằng thuật toán textbook là điểm xuất phát chứ không phải điểm kết thúc.

**Mô hình giàu vs thuật toán nhanh** — nhét mọi chi tiết vào graph (đa loại cạnh, thuộc tính, ràng buộc thời gian) làm mô hình trung thực hơn nhưng loại bỏ dần các thuật toán chuẩn. Kỹ năng thật (chương 04) là *bỏ bớt* đến khi bài toán khớp một mô hình có thuật toán tốt, và kiểm tra phần bị bỏ có làm sai kết quả không.

## 7. Production Applications

**Dependency resolution — go mod, npm, Bazel.** Package là đỉnh, import là cạnh có hướng. Thứ tự build là topological sort; "circular dependency" là thông báo Kahn trả về `nil`. Go còn *thiết kế ngôn ngữ* quanh bất biến này: import cycle là lỗi biên dịch, ép codebase luôn là DAG — nhờ đó compiler build song song theo từng lớp topo và incremental build chỉ cần rebuild "phần DAG phía sau" file thay đổi. Toán nằm ở chỗ: định lý DAG ⇔ tồn tại topo order chính là ranh giới giữa "build được" và "không bao giờ build được".

**Deadlock detection trong database.** PostgreSQL, MySQL InnoDB duy trì wait-for graph: transaction là đỉnh, "chờ lock của" là cạnh. Định kỳ (PostgreSQL: sau `deadlock_timeout`, mặc định 1s) chạy phát hiện chu trình — đúng thuật toán ba màu ở mục 5. Có chu trình → chọn một "nạn nhân" trên chu trình và abort để phá vòng. Câu chuyện mở đầu chương được giải chính xác bằng cách này. Cùng mô hình: Go runtime báo `fatal error: all goroutines are asleep - deadlock!` khi phát hiện mọi goroutine đều chờ nhau.

**Kubernetes scheduling.** Ghép pod vào node với ràng buộc (đủ CPU/RAM, affinity, taint) — về bản chất là bài toán matching trên bipartite graph pod–node, họ hàng của max-flow ở mục 5. Scheduler thực tế không giải tối ưu toàn cục (quá đắt, và pod đến theo dòng thời gian) mà dùng heuristic hai pha lọc-rồi-chấm-điểm cho từng pod; nhưng hiểu mô hình matching cho bạn biết *giới hạn* của heuristic: quyết định tham lam từng pod một có thể kẹt ở phân bổ mà bản chất matching vẫn còn lời giải — lý do tồn tại descheduler và preemption.

**Social network — friend suggestion.** "People you may know" ở dạng đơn giản nhất là: các đỉnh có khoảng cách đúng 2 từ bạn (bạn của bạn nhưng chưa là bạn), xếp hạng theo số đường đi độ dài 2 (số bạn chung) — tức một BFS cắt ở lớp 2. Ở quy mô tỷ đỉnh, cái khó không còn là thuật toán mà là biểu diễn: adjacency list phân mảnh trên nhiều máy, và mọi phép "lan một lớp" trở thành một round trip mạng — lý do các hệ graph processing (Pregel, GraphX) tổ chức tính toán theo superstep đồng bộ, đúng nhịp "từng lớp" của BFS.

**Google Maps và routing.** Giao lộ là đỉnh, đoạn đường là cạnh trọng số (thời gian di chuyển, cập nhật theo traffic). Nền tảng là Dijkstra/A*, nhưng quy mô lục địa đòi precompute phân cấp — chương 27 phân tích trọn vẹn. Cùng họ: network routing OSPF chạy Dijkstra thật sự trên graph topology mạng, mỗi router tự tính shortest path tree của mình.

**Kafka consumer rebalancing.** Phân partition cho consumer trong group là bài toán phân bổ trên bipartite graph partition–consumer. Chiến lược range/round-robin là heuristic đơn giản; sticky assignor giải bài tinh tế hơn: tìm phân bổ *cân bằng* nhưng *ít khác nhất* so với phân bổ cũ (giảm chi phí di chuyển state) — một bài toán tối ưu trên matching có thêm chi phí chuyển đổi. Cooperative rebalancing (incremental) về bản chất là thừa nhận rằng nhảy thẳng đến matching mới tối ưu đắt hơn là đi từng bước nhỏ trên không gian matching.

**Ngoài ra, nhìn đâu cũng thấy graph:** Git commit history là DAG (merge = đỉnh có hai cha, `git log --graph` vẽ nó ra); Airflow/Temporal pipeline là DAG thực thi theo topo order; Terraform xây resource graph và apply theo topo order, song song theo lớp; Elasticsearch cluster state propagation, service mesh dependency, distributed tracing (Jaeger) — mỗi trace là một cây con trong graph gọi giữa service.

## 8. Interview

Graph là chủ đề nặng ký nhất trong phỏng vấn thuật toán, và có một bí mật quan trọng hơn mọi template: **80% bài graph không nói chữ "graph" trong đề**. Đề nói về ma trận đảo–nước, danh sách môn học tiên quyết, từ điển người ngoài hành tinh, số bước biến đổi chuỗi. Kỹ năng số một là *nhận diện*: cứ khi nào có "trạng thái" và "bước chuyển hợp lệ giữa các trạng thái", bạn đang đứng trước một graph — đỉnh là trạng thái, cạnh là bước chuyển.

**Number of Islands (LeetCode 200).** Đếm số đảo trong ma trận đất–nước. Không có chữ graph nào, nhưng: mỗi ô đất là đỉnh, hai ô đất kề nhau là cạnh → "đảo" = connected component, đáp án = số lần phải khởi động lại BFS/DFS từ ô chưa thăm. Nhận ra điều đó xong, bài toán kết thúc — phần code chỉ là DFS trên lưới. Lỗi phổ biến: quên đánh dấu ô đã thăm *ngay khi cho vào queue* (chứ không phải khi pop), khiến một ô vào queue nhiều lần — trên lưới lớn là chậm rõ rệt.

**Course Schedule (LeetCode 207/210).** Cho danh sách môn tiên quyết, hỏi học hết được không / theo thứ tự nào. Đây là bài topo sort trần trụi: 207 hỏi "có chu trình không?", 210 hỏi "in ra topo order". Chạy đúng Kahn ở mục 5. Bẫy tư duy: chiều cạnh — "b là tiên quyết của a" nghĩa là cạnh b→a (học b trước); vẽ ngược chiều thì code đúng trên input sai. Luôn viết rõ quy ước cạnh trước khi code.

**Clone Graph (LeetCode 133).** Deep copy một graph. Bài kiểm tra hiểu *vì sao cần visited*: graph có chu trình, copy đệ quy ngây thơ sẽ lặp vô hạn. Lời giải là DFS/BFS kèm `map[node cũ]node mới` — map này vừa là visited vừa là kết quả. Đây chính là bài toán serialize/deserialize object graph có tham chiếu vòng mà mọi thư viện ORM, JSON encoder phải giải (Go `encoding/json` từ chối cyclic structure — giờ bạn biết vì sao).

**Network Delay Time (LeetCode 743).** Tín hiệu từ node k lan khắp mạng, hỏi thời điểm node cuối cùng nhận được. Đây là Dijkstra nguyên bản: đáp án là max của dist[]. Câu hỏi follow-up tốt mà interviewer hay dùng: "nếu có link 'tăng tốc' trọng số âm thì sao?" — kiểm tra bạn có biết ranh giới Dijkstra/Bellman-Ford và *vì sao* (mục 5) hay không, thay vì chỉ thuộc code.

**Các lỗi tư duy phổ biến:**

- Mặc định graph liên thông. Đề không hứa điều đó — vòng `for` ngoài quét mọi đỉnh chưa thăm (như trong `HasCycle`) là phản xạ bắt buộc.
- Dùng phát hiện chu trình của undirected cho directed hoặc ngược lại. Undirected chỉ cần "gặp đỉnh đã thăm không phải cha"; directed bắt buộc ba màu — gặp đỉnh *đen* không phải chu trình.
- Chọn DFS khi câu hỏi là khoảng cách/số bước ít nhất. "Ít nhất" gần như luôn là BFS (hoặc Dijkstra nếu có trọng số).
- Ước lượng sai độ phức tạp trên lưới: lưới R×C có n = R·C đỉnh và m ≤ 4n cạnh → BFS là O(R·C), không phải O((R·C)²).

**Cách phân tích thay vì học thuộc** — bốn câu hỏi theo thứ tự: (1) Đỉnh là gì, cạnh là gì? — viết rõ ra, đây là 80% lời giải. (2) Có hướng không, có trọng số không, có thể có chu trình không? (3) Câu hỏi thuộc họ nào: reachability/component (BFS/DFS), khoảng cách min (BFS/Dijkstra), thứ tự phụ thuộc (topo), nối rẻ nhất (MST), ghép cặp (flow)? (4) n, m cỡ nào — thuật toán định chọn có vừa ngân sách của bảng ngưỡng chương 10 không?

**Từ phỏng vấn sang production:** Course Schedule *là* go mod. Clone Graph *là* deep copy object có cycle. Network Delay Time *là* OSPF. Number of Islands *là* connected component labeling trong xử lý ảnh và phân cụm user. Người nhận ra các đẳng thức này không cần học thuộc 200 bài — họ chỉ cần một bảng ánh xạ trong đầu từ *dạng câu hỏi* sang *họ thuật toán*.

## 9. Anti-pattern

**Mô hình hóa sai chiều và sai loại cạnh.** Lỗi rẻ nhất để mắc và đắt nhất để tìm: dựng cạnh ngược chiều trong dependency graph, hoặc mô hình quan hệ có hướng (follow, import) như vô hướng. Thuật toán chạy hoàn hảo trên mô hình sai và trả kết quả vô nghĩa — không có exception nào báo cho bạn. Phòng thủ: viết rõ quy ước "cạnh u→v nghĩa là ___" thành comment/doc trước khi viết dòng code đầu tiên, và viết một test với graph 3 đỉnh mà bạn biết đáp án bằng tay.

**Đệ quy DFS trên graph không kiểm soát độ sâu.** Chạy đẹp trên test, nổ stack trên production với chuỗi triệu đỉnh (linked list là graph có đường đi dài n). Go đỡ hơn nhờ stack tự giãn nhưng không miễn nhiễm (mặc định trần 1GB). Graph lớn hoặc không rõ hình dạng → viết bản lặp với stack tường minh.

**Gọi "graph algorithm" khi dữ liệu nằm trong database quan hệ.** Nạp cả bảng edge triệu dòng vào RAM để chạy BFS 3 lớp, trong khi một recursive CTE (`WITH RECURSIVE`) hoặc 2–3 câu JOIN có index làm xong trong database. Ngược lại cũng là anti-pattern: mô phỏng traversal sâu không giới hạn bằng JOIN đệ quy trên RDBMS — lúc đó mới là chỗ của graph database hoặc xử lý offline. Chọn công cụ theo *độ sâu và hình dạng truy vấn*, không theo mốt.

**Chạy lại thuật toán toàn cục cho thay đổi cục bộ.** Graph thêm một cạnh, hệ thống chạy lại toàn bộ Dijkstra/topo sort trên trăm triệu đỉnh. Với dòng cập nhật liên tục (traffic, giá), cần thuật toán incremental hoặc chấp nhận kết quả xấp xỉ có TTL — "tính lại từ đầu mỗi lần" là O(khổng lồ) nhân với tần suất thay đổi.

**Tin rằng deadlock tự phát hiện là đủ.** Database kill nạn nhân giúp hệ thống *sống*, nhưng transaction bị kill là request thất bại của người dùng thật. Anti-pattern là dựa vào detector thay vì *phòng ngừa bằng toán*: ép mọi transaction lấy lock theo cùng một thứ tự toàn cục (sắp xếp resource ID tăng dần) — khi đó wait-for graph không thể có chu trình *by construction*, vì mọi cạnh chờ đều chỉ từ ID thấp đến ID cao. Một dòng quy ước coding thay cho cả hệ thống phát hiện.

**Dùng Dijkstra "cho chắc" trên graph có cạnh âm, hoặc mọi bài shortest path.** Có cạnh âm mà vẫn Dijkstra là sai *im lặng* — kết quả trông hợp lý và thỉnh thoảng đúng. Ngược lại, unweighted mà dùng Dijkstra là trả log n vô ích cho việc BFS làm tuyến tính. Bảng ở cuối mục 5 tồn tại để tra đúng ranh giới điều kiện.

## 10. Best Practices

**Nên:**

- Bắt đầu mọi bài toán quan hệ bằng câu "đỉnh là gì, cạnh là gì, chiều nào, trọng số gì?" — viết ra giấy trước khi viết code. Sai ở bước này thì mọi bước sau đều đúng-một-cách-vô-nghĩa.
- Mặc định adjacency list bằng slice (`[][]int`); ID không phải số nguyên thì quy đổi qua `map[string]int` một lần lúc dựng graph, sau đó mọi thuật toán chạy trên int — nhanh và sạch hơn map lồng nhau.
- Giữ bất biến DAG *chủ động* ở những nơi phụ thuộc: kiểm tra chu trình trong CI cho module nội bộ, lock ordering cho concurrency — rẻ hơn nhiều so với debug lúc 3 giờ sáng.
- Ghi nhớ bảng điều kiện: unweighted → BFS; trọng số ≥ 0 → Dijkstra; cạnh âm → Bellman-Ford; phụ thuộc → topo sort; nối rẻ nhất → MST; ghép cặp/phân bổ → nghĩ đến matching/flow trước khi tự chế heuristic.
- Với graph vừa RAM một máy, cứ dùng thuật toán chuẩn — chúng tuyến tính hoặc gần tuyến tính, và 10⁷–10⁸ cạnh là chuyện nhỏ. Đừng với tới framework phân tán khi `[][]int` còn thừa sức.

**Không nên:**

- Không dùng đệ quy không giới hạn độ sâu trên graph do người dùng cung cấp.
- Không giả định liên thông, không giả định không chu trình, không giả định trọng số dương — trừ khi input được *bảo đảm* như vậy, và assert điều đó lúc nạp dữ liệu.
- Không tối ưu biểu diễn graph trước khi biết truy vấn nào là nóng: cấu trúc tối ưu cho "duyệt hàng xóm" (list) và cho "tra cạnh" (matrix/hash set) là khác nhau, và chọn trước khi biết workload là đoán mò.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Chứng minh tính đúng của Dijkstra đổ vỡ ở đúng câu nào khi cho phép cạnh âm — và vì sao Bellman-Ford không đổ vỡ ở cùng chỗ đó?
2. Vì sao Kahn's algorithm phát hiện được chu trình "miễn phí", và điều đó liên hệ thế nào với bổ đề "DAG luôn có đỉnh in-degree 0"?
3. Quy ước "mọi transaction lấy lock theo thứ tự resource ID tăng dần" ngăn deadlock bằng cách loại bỏ cấu trúc gì trong wait-for graph? Chứng minh ngắn gọn.

---

*Chương tiếp theo: [09 — Boolean Algebra](/series/math-for-engineers/level-2-discrete-mathematics/09-boolean-algebra/) — rời thế giới của các mối quan hệ để xuống tầng thấp nhất: đại số của bật và tắt, nơi mọi phép tính của CPU và mọi bitmap index của database được xây lên từ ba phép toán AND, OR, NOT.*
