+++
title = "Chương 24 — Advanced Data Structures: Công cụ chuyên dụng cho truy vấn chuyên dụng"
date = "2026-07-20T11:00:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 5 – Production
> Yêu cầu trước: Chương 07 (Trees), Chương 09 (Boolean Algebra), Chương 12 (Hashing)

---

## 1. Problem Statement

Một team xây tính năng autocomplete cho ô tìm kiếm sản phẩm. Dữ liệu: 2 triệu tên sản phẩm. Giải pháp đầu tiên rất tự nhiên: đổ tất cả vào hash map... và bế tắc ngay lập tức. Hash map trả lời "chuỗi này *có tồn tại* không?" trong O(1) — nhưng autocomplete hỏi câu hoàn toàn khác: "chuỗi nào *bắt đầu bằng* `iph`?". Hash của `iph` không nói gì về hash của `iphone 15 pro` — hàm hash tốt **cố tình phá hủy** mọi cấu trúc của key (chương 12). Truy vấn prefix trên hash map là full scan.

Team chuyển sang sorted slice + binary search — chạy được, cho đến khi có yêu cầu cập nhật danh mục theo thời gian thực: mỗi insert vào giữa slice là O(n) memmove, và catalog thay đổi hàng trăm lần mỗi phút.

Câu chuyện này lặp lại khắp nơi với những bộ mặt khác nhau: "tính tổng doanh thu từ ngày X đến ngày Y, số liệu sửa liên tục", "hai node này còn thuộc cùng một partition không, khi link rớt và hồi phục liên tục", "cache 10GB, evict thế nào trong O(1)". Điểm chung:

**Mỗi cấu trúc dữ liệu là một loại truy vấn đã được tối ưu hóa. Câu hỏi đúng không phải "cấu trúc nào xịn nhất?" mà là "workload của tôi cần truy vấn gì?"**

Nếu không có tư duy này, kỹ sư chỉ có hai công cụ là hash map và slice — và sẽ trả giá bằng O(n) ở đúng chỗ hệ thống cần O(log n), hoặc tệ hơn, tự cài B-Tree khi một slice đã sort là đủ.

## 2. Trực giác

### Cấu trúc dữ liệu là "câu trả lời tính sẵn"

Hãy nhìn lại các cấu trúc đã học qua lăng kính này:

| Cấu trúc | Truy vấn được tối ưu | Cái giá |
|---|---|---|
| Hash map | "key này có không?" — điểm | mất thứ tự, mất prefix, mất range |
| Sorted slice | "phần tử ≥ x đầu tiên?" — range tĩnh | insert O(n) |
| Heap | "min/max hiện tại?" | không tìm được phần tử bất kỳ |
| BST cân bằng | điểm + range + thứ tự, có update | hằng số lớn, code phức tạp |

Không có bữa trưa miễn phí: tối ưu cho truy vấn này là bỏ rơi truy vấn khác. Chương này thêm năm công cụ vào bảng — Trie (prefix), Segment/Fenwick Tree (range động), Union-Find (liên thông động), LRU (truy cập theo thời gian) — và quan trọng hơn, thêm thói quen **đọc workload trước khi chọn cấu trúc**.

### Trực giác chung: trả trước một phần công việc

Mọi cấu trúc trong chương này đều làm cùng một việc ở tầng triết lý: **dịch chuyển chi phí từ lúc truy vấn sang lúc xây/cập nhật**. Segment tree tính sẵn tổng của các đoạn "chuẩn" để mọi đoạn bất kỳ ghép được từ O(log n) mảnh. Trie trải key ra thành đường đi để prefix trở thành... một node. Union-Find nén đường đi *trong lúc* truy vấn để lần sau rẻ hơn — trả sau nhưng trả góp. Câu hỏi "nếu không có cấu trúc X?" luôn có cùng dạng trả lời: bạn trả toàn bộ chi phí tại thời điểm truy vấn, mỗi lần truy vấn.

### Khi nào KHÔNG nên dùng?

Ngay từ đầu chương phải nói rõ: các cấu trúc này đáng giá khi **cả đọc lẫn ghi đều nhiều và đan xen**. Dữ liệu tĩnh? Sorted slice + binary search hoặc prefix sum tính một lần thắng mọi cây về hằng số lẫn độ đơn giản. n < vài nghìn? Quét thẳng. Đây là chương của những công cụ chuyên dụng — và công cụ chuyên dụng dùng sai chỗ là nợ kỹ thuật.

## 3. First Principles

### Nguyên lý 1: cấu trúc của key là tài nguyên — đừng phá nếu cần dùng

Hash map phá cấu trúc key để đổi lấy phân bố đều. Nhưng khi truy vấn *khai thác* cấu trúc key (prefix, thứ tự, khoảng), ta cần cấu trúc dữ liệu **bảo toàn** cấu trúc đó. Trie bảo toàn prefix. BST bảo toàn thứ tự. Chọn giữa hash và cây, về bản chất, là trả lời: "truy vấn của tôi có cần cấu trúc của key không?"

### Nguyên lý 2: phân rã bài toán lớn thành các khối chuẩn tính sẵn

Muốn tổng đoạn [l, r) bất kỳ trong O(log n) với dữ liệu thay đổi? Không thể tính sẵn mọi đoạn (O(n²) đoạn). Thay vào đó, tính sẵn một **bộ khung O(n) đoạn chuẩn** sao cho mọi đoạn bất kỳ phân rã thành O(log n) đoạn chuẩn. Đây là cùng ý tưởng với biểu diễn nhị phân: mọi số phân rã thành O(log n) lũy thừa của 2 (chương 09). Segment tree và Fenwick tree là hai cách hiện thực hóa cùng nguyên lý này.

### Nguyên lý 3: hai cấu trúc yếu ghép thành một cấu trúc mạnh

Không cấu trúc đơn lẻ nào cho đủ mọi truy vấn — nhưng có thể **ghép hai cấu trúc, mỗi cái che khuyết điểm của cái kia**, miễn là giữ được invariant đồng bộ giữa chúng. LRU cache (mục 5) là ví dụ giáo khoa: hash map không có thứ tự thời gian, linked list không tra cứu được theo key — ghép lại thì cả get lẫn put đều O(1). Rất nhiều thiết kế hệ thống thực chất là bài toán ghép cấu trúc kiểu này (index + heap trong priority queue có update key; map + slice trong RandomizedSet).

### Nguyên lý 4: amortization — cho phép thao tác đắt nếu nó hiếm

Path compression của Union-Find làm *một* lần find đắt hơn để *mọi* lần sau rẻ đi. Phân tích đúng phải nhìn cả chuỗi thao tác (amortized — chương 10), và kết quả α(n) của Union-Find là một trong những kết quả amortized đẹp nhất toàn bộ khoa học máy tính.

## 4. Mathematical Model

### Trie: key là đường đi, không phải giá trị

Trie là cây mà mỗi cạnh mang một ký tự; một key là **đường đi từ gốc**. Hệ quả tức thì: hai key chung prefix chung đoạn đường đầu — prefix không phải thứ cần tìm kiếm, nó là một **địa chỉ**.

```
        (gốc)
        /    \
       c      d
       a      o
      / \     g ●        "ca" rẽ nhánh thành "cat", "car"
     t●  r●              ● = kết thúc một key
```

Chi phí tra cứu: O(m) với m = độ dài key — **không phụ thuộc số key n**. So với hash map O(m) để hash + O(1) tra: cùng bậc, nhưng trie trả thêm được câu hỏi prefix mà hash map bó tay. Cái giá: mỗi node một mảng/map con trỏ con — bộ nhớ phình theo bảng chữ cái. **Radix tree** (patricia trie) nén mọi chuỗi node chỉ-một-con thành một cạnh mang cả chuỗi ký tự — đây là dạng dùng trong production: router của Go `httprouter`/`gin` match URL pattern bằng radix tree, và bảng định tuyến IP là radix tree trên bit của địa chỉ, nơi **longest prefix match** — truy vấn tự nhiên nhất của trie — quyết định gói tin đi cổng nào.

### Segment tree và Fenwick tree: phân rã đoạn

Segment tree trên mảng n phần tử: node gốc quản [0, n), mỗi node chia đôi đoạn của nó, lá là một phần tử. Định lý làm nên O(log n): **mọi đoạn truy vấn [l, r) phân rã thành ≤ 2⌈log₂n⌉ node của cây** — vì ở mỗi tầng, đoạn truy vấn "gặm" tối đa 2 node không nằm trọn (một mép trái, một mép phải), các node nằm trọn thì dừng ngay không đi xuống tiếp. Point update chỉ chạm O(log n) node trên đường từ lá lên gốc.

Fenwick tree (BIT) đạt cùng mục tiêu cho **prefix sum** bằng mẹo số học thuần túy: ô thứ i quản lý đoạn có độ dài `i & (-i)` — bit 1 thấp nhất của i, chính là lowbit của chương 09 (số bù hai: `-i` lật mọi bit trên lowbit, AND giữ lại đúng nó). Prefix sum [1..i] = nhảy `i -= i & (-i)` cho đến 0; update ngược hướng `i += i & (-i)`. Mỗi số n có O(log n) bit → O(log n) bước. Fenwick chính là "biểu diễn nhị phân của prefix" đóng vai cấu trúc dữ liệu.

### Union-Find và hàm Ackermann ngược

Mô hình: quan hệ tương đương động — `union(a, b)` trộn hai lớp, `find(x)` trả đại diện lớp. Cài bằng rừng cây con-trỏ-lên-cha. Hai tối ưu:

- **Union by rank**: gắn cây thấp vào gốc cây cao → chiều cao O(log n) (chứng minh quy nạp: cây rank r có ≥ 2^r node).
- **Path compression**: sau mỗi find, trỏ thẳng mọi node trên đường đi về gốc.

Kết hợp cả hai, Tarjan chứng minh m thao tác tốn O(m·α(n)) — α là **hàm Ackermann ngược**, tăng chậm đến mức phi thực tế: α(n) ≤ 4 với mọi n nhỏ hơn số nguyên tử trong vũ trụ quan sát được. Trực giác đáng mang theo: α(n) là "hằng số về mặt thực dụng nhưng không phải hằng số về mặt toán học" — và người ta đã chứng minh không thể bỏ nó đi: bài toán này *thật sự* không phải O(1) amortized. Một trong những chỗ hiếm hoi ranh giới giữa "gần như hằng số" và "hằng số" được vẽ chính xác tuyệt đối.

### LRU: bài toán ghép cấu trúc

Yêu cầu: `get(key)` O(1), `put(key, value)` O(1), khi đầy thì evict phần tử **ít được dùng gần đây nhất**. Phân tích từng nguyên liệu:

- Hash map: get/put theo key O(1) ✓ — nhưng không biết phần tử nào cũ nhất ✗.
- Doubly linked list: giữ thứ tự truy cập, lấy đuôi O(1), chuyển node lên đầu O(1) *nếu đã cầm con trỏ node* ✓ — nhưng tìm node theo key là O(n) ✗.

Khuyết điểm của cấu trúc này đúng là sở trường của cấu trúc kia. Ghép: **map từ key → con trỏ node trong list**. Map lo "tìm theo key", list lo "thứ tự thời gian"; con trỏ trong map là cây cầu nối hai thế giới. Vì sao list phải là *doubly* linked? Vì gỡ một node khỏi giữa list trong O(1) cần con trỏ prev — singly linked list bắt bạn đi tìm node đứng trước, O(n). Mỗi chi tiết của thiết kế đều bị ép ra từ yêu cầu O(1), không có chi tiết nào tùy hứng — đó là lý do LRU là bài học thiết kế đẹp nhất chương.

## 5. Thuật toán

### Trie — autocomplete trong O(độ dài prefix)

```go
type TrieNode struct {
    children map[byte]*TrieNode
    isEnd    bool // có key kết thúc tại node này không
}

type Trie struct{ root *TrieNode }

func NewTrie() *Trie {
    return &Trie{root: &TrieNode{children: map[byte]*TrieNode{}}}
}

func (t *Trie) Insert(word string) {
    n := t.root
    for i := 0; i < len(word); i++ {
        c := word[i]
        if n.children[c] == nil {
            n.children[c] = &TrieNode{children: map[byte]*TrieNode{}}
        }
        n = n.children[c]
    }
    n.isEnd = true
}

// StartsWith trả về node ứng với prefix (nil nếu không tồn tại).
// Autocomplete = tìm node này rồi DFS thu thập các key bên dưới.
func (t *Trie) StartsWith(prefix string) *TrieNode {
    n := t.root
    for i := 0; i < len(prefix); i++ {
        if n = n.children[prefix[i]]; n == nil {
            return nil
        }
    }
    return n
}
```

Production sẽ thêm: giới hạn số gợi ý (lưu sẵn top-k tại mỗi node thay vì DFS lúc truy vấn — lại là "trả trước công việc"), và nén thành radix tree khi bộ nhớ là vấn đề.

### Segment tree — bản rút gọn iterative

```go
// SegTree: range sum query + point update, đánh index kiểu heap.
type SegTree struct {
    n    int
    tree []int64 // tree[1] là gốc; lá i nằm tại tree[n+i]
}

func NewSegTree(a []int64) *SegTree {
    n := len(a)
    t := &SegTree{n: n, tree: make([]int64, 2*n)}
    copy(t.tree[n:], a)
    for i := n - 1; i >= 1; i-- {
        t.tree[i] = t.tree[2*i] + t.tree[2*i+1] // cha = tổng hai con
    }
    return t
}

func (t *SegTree) Update(i int, v int64) {
    i += t.n
    t.tree[i] = v
    for i > 1 { // đi từ lá lên gốc, sửa lại các tổng bị ảnh hưởng
        i /= 2
        t.tree[i] = t.tree[2*i] + t.tree[2*i+1]
    }
}

// Query trả về tổng đoạn [l, r).
func (t *SegTree) Query(l, r int) int64 {
    var s int64
    for l, r = l+t.n, r+t.n; l < r; l, r = l/2, r/2 {
        if l%2 == 1 { s += t.tree[l]; l++ } // l là con phải → lấy rồi né sang phải
        if r%2 == 1 { r--; s += t.tree[r] } // r là con phải → lùi lại và lấy
    }
    return s
}
```

Sức mạnh thật của segment tree không nằm ở phép cộng — thay `+` bằng **bất kỳ phép toán kết hợp nào** (min, max, gcd, nhân ma trận) là được cấu trúc mới. Cần **range update** (cộng cả đoạn) lẫn range query? Đó là lúc cần **lazy propagation**: node giữ một "khoản nợ" cập nhật chưa đẩy xuống con, chỉ đẩy khi buộc phải đi qua — giới thiệu để biết tên, cài đúng nó là bài tập không tầm thường.

### Fenwick tree — mười dòng thay thế

```go
type BIT struct{ t []int64 } // index từ 1

func NewBIT(n int) *BIT { return &BIT{t: make([]int64, n+1)} }

func (b *BIT) Add(i int, delta int64) {
    for ; i < len(b.t); i += i & (-i) { b.t[i] += delta }
}

func (b *BIT) PrefixSum(i int) (s int64) {
    for ; i > 0; i -= i & (-i) { s += b.t[i] }
    return
}
// Tổng đoạn [l, r] = PrefixSum(r) - PrefixSum(l-1)
```

So với segment tree: chỉ làm được phép toán **khả nghịch** (cộng được, min không — vì range = hiệu hai prefix), nhưng 10 dòng code, nửa bộ nhớ, hằng số nhỏ hơn hẳn (vòng lặp gọn, cache-friendly). Quy tắc chọn: cần range sum động → Fenwick; cần min/max/range update → segment tree; dữ liệu bất biến → prefix sum thường, đừng động đến cả hai.

### Union-Find — đầy đủ trong 25 dòng

```go
type DSU struct {
    parent []int
    rank   []int
}

func NewDSU(n int) *DSU {
    d := &DSU{parent: make([]int, n), rank: make([]int, n)}
    for i := range d.parent { d.parent[i] = i } // mỗi phần tử tự làm gốc
    return d
}

func (d *DSU) Find(x int) int {
    if d.parent[x] != x {
        d.parent[x] = d.Find(d.parent[x]) // path compression: trỏ thẳng về gốc
    }
    return d.parent[x]
}

// Union trả về false nếu x, y đã cùng thành phần (phát hiện chu trình).
func (d *DSU) Union(x, y int) bool {
    rx, ry := d.Find(x), d.Find(y)
    if rx == ry { return false }
    if d.rank[rx] < d.rank[ry] { rx, ry = ry, rx } // by rank: cây thấp gắn vào cao
    d.parent[ry] = rx
    if d.rank[rx] == d.rank[ry] { d.rank[rx]++ }
    return true
}
```

Giá trị trả về của `Union` chính là trái tim của **Kruskal** (chương 08): duyệt cạnh theo trọng số tăng, `Union` trả false nghĩa là cạnh tạo chu trình — bỏ. Trong hệ phân tán, cùng cấu trúc này trả lời "sau chuỗi sự kiện link up/link down, node A và B còn cùng partition không" — với lưu ý union-find thuần không hỗ trợ *xóa* cạnh; bài toán fully-dynamic connectivity khó hơn nhiều, đó là lý do các hệ monitoring thường rebuild từ snapshot thay vì cập nhật ngược.

### LRU Cache — ghép nối hoàn chỉnh

```go
// LRU dùng container/list của thư viện chuẩn làm doubly linked list.
type entry struct {
    key   string
    value []byte
}

type LRU struct {
    cap   int
    ll    *list.List               // đầu list = mới dùng nhất
    items map[string]*list.Element // key → node trong list
}

func NewLRU(capacity int) *LRU {
    return &LRU{cap: capacity, ll: list.New(), items: map[string]*list.Element{}}
}

func (c *LRU) Get(key string) ([]byte, bool) {
    el, ok := c.items[key]
    if !ok {
        return nil, false
    }
    c.ll.MoveToFront(el) // vừa được dùng → lên đầu, O(1) nhờ doubly linked
    return el.Value.(*entry).value, true
}

func (c *LRU) Put(key string, value []byte) {
    if el, ok := c.items[key]; ok {
        el.Value.(*entry).value = value
        c.ll.MoveToFront(el)
        return
    }
    if c.ll.Len() >= c.cap {
        oldest := c.ll.Back() // đuôi list = lâu không dùng nhất
        if oldest != nil {
            c.ll.Remove(oldest)
            delete(c.items, oldest.Value.(*entry).key)
        }
    }
    c.items[key] = c.ll.PushFront(&entry{key, value})
}
```

Chú ý invariant đồng bộ: **mọi key trong map đều trỏ đến một node đang sống trong list, và ngược lại** — mọi bug LRU tự cài (kể cả trong phỏng vấn) đều là một lần phá invariant này: quên xóa map khi evict, quên lưu key trong node để evict xóa được map.

## 6. Trade-off

**Fenwick vs Segment tree vs sorted slice.** Ba mức trên cùng một trục "tính năng đổi lấy đơn giản": prefix sum tĩnh (5 dòng, dữ liệu bất biến) → Fenwick (10 dòng, sum động) → segment tree (50 dòng, mọi phép kết hợp, lazy). Đi lên một mức chỉ khi workload thật sự đòi hỏi — mỗi mức là một bậc chi phí bảo trì.

**Trie vs hash map.** Trie thắng khi và chỉ khi truy vấn dùng cấu trúc prefix. Chỉ cần membership? Hash map thắng cả về bộ nhớ (trie: mỗi ký tự một node + overhead con trỏ, cache miss mỗi bước xuống) lẫn tốc độ thực tế. Radix tree thu hẹp khoảng cách bộ nhớ nhưng không xóa được nó.

**LRU chính xác vs LRU xấp xỉ.** LRU chuẩn tốn: 2 con trỏ list + 1 entry map cho *mỗi* key, và mỗi lần `Get` là một lần ghi (MoveToFront) — nghĩa là cache đọc nhiều vẫn ghi liên tục, tệ cho concurrency (lock trên list). Redis từ chối trả giá đó: mỗi key chỉ giữ timestamp 24-bit, khi cần evict thì **sample ngẫu nhiên** ~5 key và đuổi key cũ nhất trong mẫu. Kết quả xấp xỉ LRU với chi phí gần như bằng không — tiết kiệm 24+ byte con trỏ mỗi key trên hàng trăm triệu key, và không có cấu trúc list toàn cục nào để tranh chấp. Bài học: khi n đủ lớn, "đúng theo phân phối" thường đáng giá hơn "đúng từng phần tử" (tinh thần chương 23).

**LRU vs TinyLFU vs Clock.** LRU chỉ nhìn *recency* — một lần scan tuần tự (backup đọc hết bảng) đuổi sạch working set nóng. **TinyLFU** (thư viện Ristretto của Dgraph) chặn ở cửa vào: dùng count-min sketch (chương 23) đếm xấp xỉ tần suất, phần tử mới chỉ được **admit** nếu tần suất của nó cao hơn nạn nhân sắp bị đuổi — chống scan pollution bằng bộ đếm vài bit mỗi counter. **Clock/second-chance** (PostgreSQL buffer pool) là LRU xấp xỉ khác: kim đồng hồ quét vòng, mỗi trang một reference bit, bit 1 thì tha và xóa bit, bit 0 thì evict — không cần di chuyển node nào, thân thiện với concurrent access. Ba biến thể, ba điểm khác nhau trên trục chính xác/chi phí/chống nhiễu.

**Khi nào không dùng gì trong chương này?** Dữ liệu đọc-nhiều-ghi-không: precompute. n nhỏ: quét. Truy vấn điểm đơn thuần: hash map. Các cấu trúc chương này là công cụ của vùng workload "đọc ghi đan xen ở quy mô lớn" — bên ngoài vùng đó, chúng là over-engineering.

## 7. Production Applications

**Go httprouter / gin** — radix tree match URL: `/users/:id/orders` và `/users/:id/profile` chung đường `/users/:id/`, rẽ nhánh ở ký tự khác đầu tiên. Toán nằm ở tính chất trie: thời gian match tỷ lệ độ dài path, **không phụ thuộc số route** — 10 route hay 10.000 route, latency routing như nhau.

**PostgreSQL GIN index** — Generalized Inverted Index cho full-text search, JSONB, array: ánh xạ mỗi phần tử/lexeme → danh sách row chứa nó. Câu truy vấn "document chứa cả X và Y" là **giao hai posting list** — đại số tập hợp (chương 02) chạy trên B-Tree của các key. Cùng câu chuyện "đổi biến n" của inverted index chương 10, nay thành index bạn `CREATE` mỗi tuần.

**Elasticsearch FST** — term dictionary của Lucene lưu bằng Finite State Transducer: trie nén cả prefix (như radix) **lẫn suffix** (các đuôi giống nhau chia sẻ node — trie trở thành DAG), tối thiểu hóa như automaton. Hàng chục triệu term nằm gọn trong RAM; mọi query prefix/wildcard/fuzzy đều chạy trên cấu trúc này. Đây là trie ở dạng tiến hóa cao nhất trong production.

**Prometheus TSDB index** — mỗi cặp label (`job="api"`) → posting list các series ID; query nhiều label = giao các posting list đã sort. Kết hợp inverted index cho label và cây symbol cho tên metric — lý do query hàng triệu series vẫn trả lời trong millisecond.

**ClickHouse sparse index** — minh họa ngược đầy chủ đích: ClickHouse **không** index từng row. Dữ liệu sort theo primary key, index chỉ ghi mốc mỗi 8192 row (granule) — binary search trên mốc rồi scan granule. Với workload analytics đọc dải lớn, "index thưa + scan tuần tự cực nhanh" thắng B-Tree per-row. Đúng tinh thần chương: không có cấu trúc tốt nhất, chỉ có cấu trúc khớp truy vấn nhất.

**Kruskal & network** — union-find trong bài toán MST của thiết kế mạng/clustering (single-linkage clustering *chính là* Kruskal dừng sớm), và trong các tool phân tích liên thông đồ thị dependency, phát hiện split-brain từ log sự kiện.

## 8. Interview

**Implement Trie (LeetCode 208)** — bài "cài đặt theo mô tả" chuẩn mực; code như mục 5. Lỗi phổ biến: nhầm `isEnd` với "có con hay không" (key `"app"` tồn tại độc lập với `"apple"`). Mở rộng production người phỏng vấn thích nghe: top-k tại node cho autocomplete, radix tree cho router.

**Range Sum Query – Mutable (LeetCode 307)** — bài nhận diện workload lộ liễu nhất: chữ "Mutable" trong đề *là* lời giải. Immutable (LC 303) → prefix sum thường; mutable → Fenwick hoặc segment tree. Nói được "tôi chọn Fenwick vì chỉ cần sum, code ngắn hơn 5 lần" là điểm cộng lớn hơn cài segment tree đầy đủ.

**Number of Islands II (LeetCode 305)** — đảo *mọc dần* từng ô: bản động của Number of Islands (LC 200). BFS/DFS lại từ đầu sau mỗi ô mới là O(k·mn); union-find cho O(k·α): mỗi ô mới +1 đảo, union với mỗi ô láng giềng đã là đất thì −1. Bài mẫu về chuyển tư duy "tính lại từ đầu" → "duy trì cấu trúc động".

**LRU Cache (LeetCode 146)** — bài thiết kế kinh điển nhất lịch sử phỏng vấn, đủ mặt ở mọi công ty lớn. Người phỏng vấn muốn thấy **quá trình suy luận ở mục 4**: nêu yêu cầu O(1), phân tích từng cấu trúc đơn lẻ vì sao thất bại, rồi ghép. Nói ngay "dùng map + doubly linked list" như thuộc lòng mà không giải thích *vì sao cần cả hai và vì sao phải doubly* là trượt phần đẹp nhất của bài. Trong Go, biết `container/list` là điểm cộng thực chiến. Follow-up thường gặp: thread-safe thế nào (một mutex — hay sharded LRU để giảm contention), TTL thêm vào đâu.

**LFU Cache (LeetCode 460)** — nâng cấp độ khó: evict theo *tần suất*, tie-break bằng recency. Lời giải O(1): map key→node + **map tần suất→doubly linked list riêng** + biến minFreq. Đối chiếu production: đây chính là họ hàng chính xác-nhưng-đắt của TinyLFU — nói ra liên hệ đó là câu trả lời cấp senior.

**Lỗi tư duy lớn nhất của cả chương**: rút vũ khí to cho bài toán nhỏ. Cài segment tree khi mảng bất biến (prefix sum đủ), khi chỉ query một lần (quét đủ), khi "sorted slice + binary search" giải trong 5 dòng. Người phỏng vấn hỏi "dữ liệu có thay đổi không?" không phải để làm khó — đó là câu hỏi *chọn cấu trúc*, và thí sinh giỏi tự hỏi nó trước khi code.

## 9. Anti-pattern

**Chọn cấu trúc theo độ ngầu, không theo truy vấn.** Segment tree trong service mà mảng rebuild mỗi đêm và chỉ đọc ban ngày; trie tự cài cho 200 route. Cấu trúc càng mạnh càng nhiều code, càng nhiều invariant phải giữ, càng nhiều chỗ cho bug — sức mạnh không dùng đến là thuần chi phí.

**LRU tự chế thiếu đồng bộ hai nửa.** Evict khỏi list mà quên `delete` map (leak — map giữ con trỏ node chết), update value mà quên MoveToFront (thứ tự sai âm thầm), và trong bản concurrent: lock map nhưng không lock list. Invariant "map và list là hai hình chiếu của cùng một tập entry" phải đúng **sau mỗi thao tác**, kể cả nhánh lỗi.

**Cache LRU thuần trước workload có scan.** Job báo cáo đọc tuần tự 10 triệu key mỗi sáng thổi bay toàn bộ working set — hit rate sập trong giờ cao điểm kế tiếp. Đây không phải bug code mà bug *chọn chính sách*: cần admission (TinyLFU) hoặc tách cache cho traffic scan.

**Trie với alphabet lớn cài bằng mảng con.** `[256]*TrieNode` mỗi node = 2KB con trỏ hầu hết nil; với Unicode thì bất khả thi. Dùng map con (như code mục 5) khi thưa, mảng khi alphabet nhỏ và dày (26 chữ cái), radix khi chuỗi dài ít rẽ nhánh.

**Quên rằng union-find không biết xóa.** Model hóa "link down" bằng cách... không có cách nào trong DSU thuần. Nếu bài toán có cả thêm lẫn xóa cạnh, DSU cho kết quả sai lặng lẽ (thành phần chỉ ngày càng to). Fully-dynamic connectivity cần cấu trúc khác hẳn hoặc rebuild định kỳ — nhận ra giới hạn này quan trọng hơn nhớ cách cài.

## 10. Best Practices

**Nên:**

- Bắt đầu mọi lựa chọn bằng bảng kê truy vấn: loại truy vấn nào, tần suất đọc/ghi, dữ liệu tĩnh hay động, n dự kiến. Cấu trúc dữ liệu là *hệ quả* của bảng này.
- Leo thang độ phức tạp từng nấc: slice → sorted slice → Fenwick → segment tree; map → trie → radix — dừng ở nấc đầu tiên thỏa yêu cầu.
- Với cache production: dùng thư viện đã kiểm chứng (`hashicorp/golang-lru`, `ristretto`) thay vì tự cài; tự cài LRU là bài phỏng vấn, không phải task sprint. Đo hit rate trước và sau khi đổi chính sách — đó là metric duy nhất có tiếng nói.
- Khi cài cấu trúc ghép (map + list), viết invariant đồng bộ thành comment và test nó: property-based test "sau chuỗi thao tác ngẫu nhiên, len(map) == list.Len()" bắt được hầu hết bug.
- Nhớ các bản production-grade để không tái phát minh: FST (term dictionary), radix (router, IP), GIN/posting list (search), clock (buffer pool).

**Không nên:**

- Không cài segment tree khi sorted slice + binary search (hoặc Fenwick) đủ — và không cài lazy propagation khi point update đủ.
- Không dùng LRU chính xác cho cache trăm triệu key — chi phí con trỏ và lock đáng giá hơn độ chính xác eviction.
- Không tin "O(1) amortized" của mình khi chưa nhìn cả chuỗi thao tác — và không quên rằng α(n) tuy "coi như 4" nhưng câu chuyện vì sao nó ở đó là thứ tách senior khỏi người học thuộc.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Vì sao LRU cần *cả* hash map *lẫn* doubly linked list — cụ thể mỗi cấu trúc thất bại ở truy vấn nào nếu đứng một mình, và vì sao singly linked không đủ?
2. Fenwick tree làm được range sum nhưng không làm được range min — tính chất toán học nào của phép toán quyết định ranh giới đó?
3. Redis chấp nhận LRU xấp xỉ bằng sampling còn PostgreSQL chọn clock — hai lựa chọn này đánh đổi gì so với LRU chính xác, và vì sao ở quy mô của họ đó là lựa chọn đúng?

---

*Chương tiếp theo: [25 — Distributed Systems Mathematics](/series/math-for-engineers/level-5-production/25-distributed-systems-math/), nơi các cấu trúc một-máy của chương này va vào thực tế nhiều máy: đồng hồ logic, quorum, và lý do toán học khiến "chính xác tuyệt đối" trở thành xa xỉ phẩm trong hệ phân tán.*
