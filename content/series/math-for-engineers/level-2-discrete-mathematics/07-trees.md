+++
title = "Chương 07 — Trees: Vì sao database index có dạng cây"
date = "2026-07-20T08:10:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 2 – Discrete Mathematics
> Yêu cầu trước: Chương 03 (Proof Techniques), Chương 06 (Recurrence Relations)

---

## 1. Problem Statement

Một bảng PostgreSQL chứa 1 tỷ dòng. Bạn chạy `SELECT * FROM orders WHERE id = 4815162342` và nhận kết quả sau **dưới 1 millisecond**. Dừng lại và thấy điều đó vô lý đến mức nào: 1 tỷ dòng, mỗi dòng vài trăm byte — bảng nặng vài trăm GB. Đọc tuần tự từ SSD với tốc độ 2 GB/s mất vài phút. Ngay cả khi toàn bộ dữ liệu nằm sẵn trong RAM, scan tuyến tính 1 tỷ dòng cũng tốn cả giây. Vậy mà database trả lời trong thời gian đủ để ánh sáng đi được 300 km.

Câu trả lời, tất nhiên, là *index*. Nhưng "có index nên nhanh" là câu trả lời của người dùng; câu hỏi của kỹ sư sâu hơn ba tầng:

**Vì sao cấu trúc cho phép tìm kiếm trong hàng tỷ phần tử với vài bước lại là một cái cây? Vì sao là cây có hình dạng cụ thể đó — và vì sao KHÔNG phải binary tree, dù mọi giáo trình dạy binary tree trước tiên?**

Cây là cấu trúc dữ liệu phổ biến nhất trong hạ tầng phần mềm: B-Tree trong mọi database quan hệ, LSM tree trong RocksDB/Cassandra, Merkle tree trong Git và blockchain, DOM tree trong browser, cây thư mục trong filesystem, cây quyết định trong query planner. Không phải ngẫu nhiên — cây là hình dạng toán học của hai ý tưởng nền tảng: **phân cấp** (mỗi thứ có đúng một cha) và **loại trừ theo tỷ lệ** (mỗi bước đi xuống vứt bỏ một phần cố định của không gian tìm kiếm). Chương này đi từ định nghĩa toán học đến con số 8KB rất cụ thể quyết định hình dạng index của PostgreSQL.

## 2. Trực giác

### Trò chơi đoán số và ý tưởng "loại trừ theo tỷ lệ"

Trò chơi cũ: tôi nghĩ một số từ 1 đến 1 tỷ, bạn hỏi các câu yes/no. Chiến lược tốt nhất ai cũng biết: mỗi câu hỏi chẻ đôi khoảng còn lại. Số câu cần: log₂(10⁹) ≈ 30. Đó chính là binary search (chương 10) — và cây nhị phân tìm kiếm chỉ là *binary search được đông cứng thành cấu trúc dữ liệu*: thay vì tính lại điểm giữa mỗi lần, ta xây sẵn "cây các điểm giữa" và tìm kiếm là đi một đường từ gốc xuống lá.

Bây giờ đổi luật: mỗi lượt, thay vì một câu yes/no, bạn được hỏi một câu có **500 đáp án** ("số nằm trong khoảng nào trong 500 khoảng này?"). Số lượt cần: log₅₀₀(10⁹) ≈ **3.3** — bốn lượt là chắc chắn xong. Đây là toàn bộ bí mật của database index: *nếu mỗi bước rẽ được nhiều nhánh hơn, cây thấp hơn*. Câu hỏi còn lại — vì sao database "được phép" hỏi câu 500 đáp án với chi phí ngang một câu yes/no — sẽ được trả lời ở mục 4, và câu trả lời nằm ở phần cứng, không nằm ở toán.

### log n: con số ma thuật đến từ đâu

Chiều cao cây chính là nghiệm của recurrence hình học ở chương 06: cây bậc m cao h chứa được cỡ mʰ lá → muốn chứa n phần tử cần h = log_m n. Chiều đảo mới là điều kỳ diệu đáng khắc ghi: **dữ liệu nhân m lần, cây chỉ cao thêm 1**. Một cấu trúc mà chi phí truy cập gần như *miễn nhiễm với tăng trưởng dữ liệu* — đó là lý do log n xuất hiện ở mọi nơi có chữ "index", "sorted", "hierarchy".

| n | log₂ n (binary) | log₅₀₀ n (B-Tree) |
|---|---|---|
| 1.000 | 10 | 1.1 |
| 10⁶ | 20 | 2.2 |
| 10⁹ | 30 | 3.3 |
| 10¹² | 40 | 4.5 |

### Khi nào trực giác cây thất bại?

Cây chỉ thấp khi nó **cân**. Trực giác "cây = log n" ngầm giả định các nhánh chia đều — và giả định đó sụp đổ dễ hơn nhiều người nghĩ: chèn dữ liệu *đã sắp xếp* vào một BST ngây thơ tạo ra "cây" nghiêng hẳn một bên — thực chất là linked list đội lốt, mọi thao tác O(n). Phần lớn nội dung kỹ thuật về cây (AVL, Red-Black, B-Tree split) là các cơ chế **bảo vệ chữ log** khỏi dữ liệu không thân thiện. Cây không tự cân; ai đó phải trả chi phí giữ nó cân.

## 3. First Principles

### Cây là gì, nói bằng ngôn ngữ đồ thị

> **Cây = đồ thị vô hướng liên thông và không có chu trình.**

Hai điều kiện, không thừa không thiếu: *liên thông* — đi được từ node bất kỳ đến node bất kỳ; *không chu trình* — không có đường vòng. Từ định nghĩa tối giản này rơi ra một loạt tính chất tương đương, mà quan trọng nhất:

**Cây n đỉnh có đúng n − 1 cạnh.** Chứng minh bằng induction (chương 03), và phép induction này đáng xem vì nó trực quan như xếp hình: cây 1 đỉnh có 0 cạnh (base case). Cây n đỉnh luôn có ít nhất một *lá* — node chỉ dính 1 cạnh (nếu mọi node có ≥ 2 cạnh, đi mãi không lặp lại node sẽ vô hạn, mà đồ thị hữu hạn thì buộc phải lặp → có chu trình → mâu thuẫn). Ngắt lá đó ra: còn lại cây n−1 đỉnh, theo giả thiết quy nạp có n−2 cạnh; cộng lại cạnh vừa ngắt: n−1 cạnh. ∎

Con số n−1 mang một ý nghĩa thiết kế: cây là đồ thị liên thông **tối giản** — đúng đủ cạnh để mọi thứ nối với nhau, không dư một cạnh nào. Bớt 1 cạnh: đứt liên thông. Thêm 1 cạnh: sinh chu trình. Hệ quả trực tiếp: **giữa hai node bất kỳ có đúng một đường đi** — không có chuyện "đến đích bằng hai ngả". Tính chất này là nền của spanning tree trong thiết kế mạng (chương 08), của việc Git lần về commit tổ tiên theo một đường duy nhất trong lịch sử tuyến tính, và của việc filesystem không cho một thư mục có hai cha (hard link cho *file* chính là chỗ filesystem cố tình phá tính chất cây — và mọi rắc rối của nó đến từ đó).

### Rooted tree: cây có phương hướng

Chọn một node làm **gốc (root)**, mọi cạnh tự nhiên có hướng "xa dần gốc", và bộ từ vựng quen thuộc xuất hiện: cha/con, tổ tiên/hậu duệ, **lá** (không con), **độ sâu** của node (khoảng cách tới gốc), **chiều cao** của cây (độ sâu lớn nhất), **subtree** — cây con treo tại một node. Từ khóa cuối cùng quan trọng nhất: subtree của mỗi node *tự nó là một cây hoàn chỉnh*. Cây là cấu trúc **đệ quy tự nhiên** — định nghĩa của nó, thuật toán trên nó, và chứng minh về nó đều có dạng "xử lý gốc + đệ quy trên các subtree". Cây là nơi chương 03 (induction), chương 06 (recurrence) và cấu trúc dữ liệu gặp nhau tại một điểm.

### Nếu không có cấu trúc cây thì sao?

Hãy thử sống thiếu nó. Cần tìm kiếm nhanh theo khóa *và* duyệt theo thứ tự? Mảng sorted tra nhanh O(log n) nhưng chèn O(n) — mỗi insert đẩy nửa mảng. Linked list chèn O(1) nhưng tra O(n). Hash map tra O(1) nhưng **vô trật tự** — không trả lời được `WHERE id BETWEEN a AND b` hay `ORDER BY` mà không sort lại từ đầu. Cây cân bằng là cấu trúc *duy nhất trong bộ đồ nghề cơ bản* cho đồng thời: tìm, chèn, xóa O(log n) **và** duyệt có thứ tự O(n). Range query chính là lý do tồn tại của nó — và là lý do index database mặc định là B-Tree chứ không phải hash, dù hash tra điểm nhanh hơn.

## 4. Mathematical Model

### Binary Search Tree và invariant của nó

**Binary tree**: mỗi node tối đa 2 con. **BST** thêm một invariant (chương 03) toàn cục:

> Với mọi node x: mọi khóa trong subtree **trái** < khóa của x < mọi khóa trong subtree **phải**.

Chữ đậm "mọi khóa trong subtree" — không phải "con trái và con phải" — là chỗ 50% ứng viên phỏng vấn trượt bài Validate BST, quay lại ở mục 8. Invariant này làm tìm kiếm thành việc đi một đường từ gốc xuống: tại mỗi node, so sánh một lần và vứt bỏ trọn một subtree. Chi phí = số node trên đường đi ≤ chiều cao h. Toàn bộ vấn đề dồn về một câu: **h bằng bao nhiêu?**

- Cây cân hoàn hảo: mỗi tầng gấp đôi số node, tầng 0..h phủ 2^(h+1) − 1 node → **h ≈ log₂ n**.
- Cây suy biến (chèn 1, 2, 3, ..., n theo thứ tự): mỗi node mới lớn hơn tất cả → luôn rẽ phải → **h = n − 1**.

```
   Chèn 4,2,6,1,3,5,7          Chèn 1,2,3,4,5,6,7
         4                          1
       ┌─┴─┐                         └─2
       2   6                           └─3
      ┌┴┐ ┌┴┐                            └─4      ← "cây" này
      1 3 5 7                              └─...     là linked list
   h = 2, tra 3 bước            h = 6, tra 7 bước
```

Cùng 7 khóa, cùng invariant BST, hiệu năng khác nhau về bản chất. Và trường hợp xấu không hề hiếm: dữ liệu đến theo thứ tự thời gian, ID tự tăng — chính là kịch bản *phổ biến nhất* trong thực tế. BST ngây thơ là cấu trúc thất bại đúng vào input thường gặp nhất.

### Cây tự cân bằng: mua bảo hiểm cho chữ log

AVL tree và Red-Black tree giải quyết bằng cùng một chiến lược: sau mỗi insert/delete, nếu một phép đo độ lệch vượt ngưỡng (AVL: chênh lệch chiều cao hai subtree > 1; Red-Black: các quy tắc tô màu), thực hiện **rotation** — thao tác O(1) xoay cục bộ ba node và các subtree treo trên chúng:

```
      z                             y
     / \        xoay phải         /   \
    y   D      ─────────→        x     z
   / \          tại z           / \   / \
  x   C                        A   B C   D
 / \
A   B        (thứ tự in-order A x B y C z D không đổi —
              rotation đổi HÌNH DẠNG, bảo toàn INVARIANT)
```

Đó là toàn bộ trực giác cần có: rotation là phép biến hình bảo toàn thứ tự, đủ rẻ để làm thường xuyên, và làm đủ thường xuyên thì h bị ép ở mức O(log n) (AVL chặt hơn: h ≤ 1.44·log₂n; Red-Black lỏng hơn: h ≤ 2·log₂n — đổi lại ít rotation hơn khi ghi). Chi tiết các case tô màu là việc của người viết thư viện; kỹ sư dùng cần nhớ đúng một điều: **`sorted map` trong mọi ngôn ngữ (C++ `std::map`, Java `TreeMap`) là Red-Black tree, và chữ O(log n) trên nhãn của nó là thứ được mua bằng rotation ở mỗi lần ghi.**

### B-Tree: khi thước đo chi phí đổi từ "phép so sánh" sang "lần đọc đĩa"

Đến câu hỏi định danh của chương: vì sao database không dùng binary tree cân bằng — thứ đã hoàn hảo về mặt lý thuyết O(log n)?

Vì **mô hình chi phí của lý thuyết đếm sai thứ trên đĩa**. Với dữ liệu trên disk, chi phí không phải số phép so sánh mà là số **block I/O**: đĩa (và cả SSD) đọc theo khối — muốn 1 byte vẫn phải đọc nguyên block, PostgreSQL dùng page **8KB**. Một node binary tree chứa 1 khóa + 2 con trỏ ≈ vài chục byte; đọc nó tốn nguyên một page — **dùng 0.3% dữ liệu của lần I/O đã trả tiền**. Cây cao 30 tầng = 30 lần I/O ngẫu nhiên ≈ 3ms với SSD — chậm không chấp nhận được, và đó là *mỗi lookup*.

B-Tree lật lại thiết kế từ ràng buộc phần cứng: **một node = một page, nhét khóa đầy page**. Page 8KB chứa vài trăm cặp (khóa, con trỏ) — lấy tròn bậc m ≈ 500. Giờ recurrence chiều cao chạy với cơ số 500:

> h = log₅₀₀(10⁹) = ln(10⁹)/ln(500) ≈ 20.7/6.2 ≈ **3.3 → cây cao 4 tầng cho 1 tỷ khóa**

Và còn tốt hơn trên thực tế: gốc và toàn bộ tầng 2 (khoảng 500 page = 4MB) gần như chắc chắn nằm sẵn trong buffer cache. Một lookup trên 1 tỷ dòng = **1–2 lần I/O thật sự**. Bí ẩn mở đầu chương được giải: không phải phép màu, mà là log cơ số 500 cộng với cache.

So sánh câu chuyện với binary tree trong RAM: tại đó, "câu hỏi 500 đáp án" không miễn phí — bên trong node, vẫn phải binary search giữa 500 khóa, tốn log₂500 ≈ 9 phép so sánh; tổng số so sánh vẫn là log₂n, *không tiết kiệm gì*. B-Tree không thắng binary tree về số phép so sánh — nó thắng vì **gom các phép so sánh vào cùng một block đã phải trả tiền I/O nguyên khối**. Đây là ví dụ đẹp nhất trong tài liệu này về nguyên tắc của chương 10: Big-O chỉ có nghĩa khi *đếm đúng phép toán đắt tiền* — và trên đĩa, phép toán đắt tiền là I/O, không phải so sánh.

**B+Tree** — biến thể mà PostgreSQL, MySQL/InnoDB thực dùng — dồn toàn bộ dữ liệu xuống lá và **nối các lá thành danh sách liên kết**: node trong chỉ chứa khóa dẫn đường (nên chứa được nhiều khóa hơn → cây càng thấp), còn range query `BETWEEN` thành: tra xuống lá đầu tiên O(log n), rồi *trượt ngang* qua các lá kế tiếp — I/O tuần tự, thứ mà đĩa và prefetcher yêu thích nhất (bài học Kafka của chương 10, lần này trong lòng database).

### Traversal: một cấu trúc, nhiều thứ tự đọc

Ba cách duyệt cây nhị phân khác nhau đúng ở vị trí "thăm node hiện tại" so với hai đệ quy con: **pre-order** (node → trái → phải — dùng để copy/serialize cây, vì gặp cha trước con), **in-order** (trái → node → phải), **post-order** (trái → phải → node — dùng để xóa/tính toán từ dưới lên, vì con xong mới đến cha). Đáng nhớ nhất là tính chất của in-order trên BST: **in-order traversal cho ra dãy khóa đã sắp xếp** — hệ quả trực tiếp của invariant BST, chứng minh một dòng bằng induction. Đây là mảnh ghép cuối của bức tranh "vì sao index là cây": `ORDER BY` trên cột có B-Tree index không cần sort — chỉ cần đọc lá từ trái sang phải.

Còn **heap** — cây nhị phân với invariant lỏng hơn nhiều (cha ≥ con, không ràng buộc trái/phải) — đổi khả năng tìm kiếm lấy việc lấy max O(log n) và biểu diễn phẳng bằng mảng không cần con trỏ. Nhắc để thấy: cùng hình dạng cây, invariant khác nhau tạo ra công cụ khác nhau. Chi tiết ở chương 16.

## 5. Thuật toán

BST đủ nhỏ để viết trọn vẹn — và để thấy cấu trúc đệ quy của cây phản chiếu vào code:

```go
type Node struct {
    Key         int
    Left, Right *Node
}

// Insert: đi xuống theo invariant BST đến chỗ trống rồi treo node mới.
// Chi phí O(h) — chữ h này chính là toàn bộ câu chuyện mục 4.
func insert(root *Node, key int) *Node {
    if root == nil {
        return &Node{Key: key} // chạm đáy: đây là chỗ của key
    }
    if key < root.Key {
        root.Left = insert(root.Left, key)
    } else if key > root.Key {
        root.Right = insert(root.Right, key)
    }
    return root // key trùng: bỏ qua (quy ước của ta)
}

// In-order traversal: trái → node → phải. Trên BST, visit được gọi
// theo thứ tự khóa TĂNG DẦN — "sorted output miễn phí".
func inorder(root *Node, visit func(int)) {
    if root == nil {
        return
    }
    inorder(root.Left, visit)
    visit(root.Key)
    inorder(root.Right, visit)
}
```

Đệ quy trên cây là Θ(n) — hai lời gọi con nhưng tổng kích thước bài con là n−1, mỗi node bị thăm đúng một lần (đúng case "đếm theo tổng công việc" của chương 06). Nhưng đệ quy dùng call stack sâu bằng h — với cây suy biến là n frame, và ta đã biết từ chương 06 vì sao đó là vấn đề với input người dùng kiểm soát. Bản không đệ quy thay call stack bằng stack tường minh:

```go
// In-order không đệ quy: stack tường minh mô phỏng call stack.
// Bất biến: đẩy toàn bộ "sườn trái" vào stack, pop ra thăm, rồi
// chuyển sang subtree phải của node vừa thăm.
func inorderIter(root *Node, visit func(int)) {
    stack := []*Node{}
    curr := root
    for curr != nil || len(stack) > 0 {
        for curr != nil { // trượt hết sườn trái
            stack = append(stack, curr)
            curr = curr.Left
        }
        curr = stack[len(stack)-1] // node nhỏ nhất chưa thăm
        stack = stack[:len(stack)-1]
        visit(curr.Key)
        curr = curr.Right // tiếp tục với subtree phải
    }
}
```

Một ghi chú riêng cho Go: ngôn ngữ này **không có cây trong thư viện chuẩn** — không `TreeMap`, không `sorted map`. Đây là lựa chọn thiết kế có chủ đích: đại đa số nhu cầu được phủ bởi `map` (tra điểm O(1)) cộng `slice + sort.Search` (dữ liệu sorted ít thay đổi, binary search O(log n), cache locality tuyệt đối). Cây cân bằng chỉ thật sự cần khi *vừa ghi thường xuyên vừa cần thứ tự* — khi đó dùng `google/btree` (chính là B-Tree in-memory, được etcd dùng làm treeIndex). Bài học trade-off: đừng trả chi phí con trỏ + rotation của cây khi map + slice rẻ hơn và đủ dùng — cùng tinh thần "linked list O(1) thua slice O(n)" của chương 10.

## 6. Trade-off

**Bậc thấp vs bậc cao — RAM vs disk.** Binary tree tối ưu cho mô hình "mỗi phép so sánh giá ngang nhau" (RAM lý tưởng); B-Tree tối ưu cho "mỗi lần chạm bộ nhớ mới là một block đắt tiền" (disk, và cả cache line của CPU — vì thế B-Tree bậc nhỏ in-memory như `google/btree` với degree 32 vẫn thắng Red-Black tree trên phần cứng thật). Chọn bậc của cây là khai báo mô hình chi phí của bạn.

**Chi phí giữ cân bằng vs chi phí tra cứu.** Cây tự cân bằng trả thêm ở mỗi lần ghi (rotation, split node) để giữ mọi lần đọc O(log n). Nếu workload là "nạp một lần, đọc mãi mãi", sorted slice + binary search rẻ hơn mọi cây. Nếu workload ghi áp đảo — trade-off nghiêng hẳn sang cấu trúc khác: LSM tree (mục 7).

**Index không miễn phí.** Mỗi B-Tree index là một cây *riêng* phải cập nhật ở mỗi INSERT/UPDATE: một bảng 5 index trả 5 lần chi phí ghi cây cho mỗi dòng chèn, chưa kể write amplification khi node split. "Đánh index mọi cột cho chắc" là chuyển chi phí từ SELECT sang mọi con đường ghi — thường là lỗ.

**Cây vs hash.** Hash tra điểm O(1) thắng cây O(log n) — nhưng mất thứ tự: không range query, không ORDER BY, không prefix scan. PostgreSQL có hash index nhưng B-Tree là mặc định vì SQL sống bằng range và order. Quy tắc: truy cập *chỉ có* dạng "đúng khóa này" → hash; có bất kỳ nhu cầu "khoảng, lân cận, thứ tự" → cây.

**Khi nào KHÔNG dùng cây?** Dữ liệu nhỏ (n < vài nghìn — scan slice thắng nhờ cache), truy cập chỉ theo khóa chính xác (hash), dữ liệu bất biến sorted sẵn (binary search trên mảng), hay quan hệ *không phải* phân cấp — nhiều cha, có chu trình — khi đó ép vào cây là bóp méo mô hình, thứ cần là đồ thị tổng quát (chương 08).

## 7. Production Applications

**PostgreSQL B-Tree index — làm phép tính cụ thể.** Index trên cột `bigint` (8 byte khóa + ~16 byte tuple header/con trỏ): mỗi page lá 8KB chứa ~300–400 entry, mỗi page trong chứa ~400–500 khóa dẫn đường. Bảng 1 tỷ dòng: lá ≈ 10⁹/350 ≈ 2.9 triệu page (~23GB... của riêng index — lý do dung lượng index là hạng mục capacity planning riêng); tầng trên lá ≈ 2.9M/450 ≈ 6.400 page; tầng nữa ≈ 15 page; cộng gốc → **cây cao 4**. Root + tầng 2 chiếm ~120KB + 50MB — nằm gọn trong `shared_buffers`, nên một index lookup lạnh tốn 1–2 I/O thật. Đây chính xác là phép tính log₅₀₀(10⁹) của mục 4, kiểm chứng được bằng extension `pageinspect`. Mọi lần bạn đọc `Index Scan using orders_pkey` trong EXPLAIN, cái giá đằng sau là con số này.

**LSM tree — đối thủ của B-Tree, và trade-off ghi/đọc.** B-Tree ghi *tại chỗ*: mỗi insert có thể chạm và ghi lại vài page rải rác — random write. LSM tree (RocksDB, Cassandra, LevelDB, ScyllaDB) chọn ngược: ghi vào memtable trong RAM, đầy thì *đổ nguyên khối* xuống đĩa thành file sorted bất biến (SSTable) — **mọi ghi đĩa đều tuần tự**, throughput ghi vượt B-Tree nhiều lần. Hóa đơn được chuyển sang phía đọc: một khóa có thể nằm ở memtable *hoặc* bất kỳ SSTable nào, đọc phải kiểm tra lần lượt (giảm nhẹ bằng Bloom filter — chương 23), và nền phải chạy compaction — merge các SSTable, chính là merge sort của chương 06 chạy vĩnh viễn dưới nền, trả bằng write amplification. Chọn engine lưu trữ là chọn nghiệm cho bất đẳng thức: **write-heavy, đọc chấp nhận đắt hơn → LSM; read-heavy với range query → B-Tree.** Không có cấu trúc thắng tuyệt đối, chỉ có workload quyết định.

**Merkle tree** — cây mà mỗi node chứa hash của các con: đổi câu hỏi "hai bản sao dữ liệu có khớp nhau không" từ so sánh O(n) thành so sánh gốc O(1), và *tìm chỗ khác nhau* thành đi xuống O(log n). Git, Cassandra anti-entropy repair, blockchain đều đứng trên nó — chi tiết ở chương 26; ở đây chỉ cần thấy: lại là "loại trừ theo tỷ lệ mỗi bước xuống", lần này cho bài toán so khớp thay vì tìm kiếm.

**DOM tree** — ví dụ cây bạn render mỗi ngày: HTML lồng nhau là rooted tree, CSS selector là ngôn ngữ truy vấn đường đi trên cây, còn React reconciliation là bài toán so khớp hai cây — so khớp cây tổng quát tốn O(n³), nên React cắt xuống O(n) bằng heuristic (chỉ so cùng tầng, key để nhận diện) — một trade-off "chính xác đổi khả thi" kinh điển.

**Kubernetes và etcd**: mọi `kubectl get` đi qua etcd, nơi treeIndex (B-Tree in-memory, `google/btree`) ánh xạ key → revision và bbolt (B+Tree trên đĩa) lưu dữ liệu. Cả control plane của Kubernetes đứng trên hai cái cây.

## 8. Interview

Cây là chủ đề phỏng vấn nặng ký nhất sau array/string — vì nó kiểm tra đồng thời đệ quy, invariant, và phân tích chi phí.

**Validate BST (LeetCode 98) — bài học về invariant toàn cục.** Lỗi kinh điển mà interviewer *chờ sẵn*: kiểm tra mỗi node chỉ so với **cha trực tiếp** (`node.Left.Key < node.Key < node.Right.Key`). Cây sau vượt qua phép kiểm tra đó nhưng không phải BST:

```
        5
      ┌─┴─┐
      3   8
         ┌┴┐
         6 9      ← 6 > cha (8)? Sai chiều... 6 < 8 ✓, nhưng 6 nằm trong
                    subtree PHẢI của 5 mà 6 > 5 ✓... còn 4 thì sao?
        5
      ┌─┴─┐
      3   8
         ┌┴┐
         4 9      ← 4 < 8 ✓ (thỏa "so với cha") nhưng 4 < 5 mà nằm bên PHẢI của 5 → KHÔNG phải BST
```

Invariant BST là mệnh đề về *mọi tổ tiên*, không phải về cha — đúng như đã nhấn ở mục 4. Lời giải đúng truyền **khoảng cho phép** (min, max) xuống mỗi lời gọi: con trái của x nhận (min, x.Key), con phải nhận (x.Key, max) — mỗi node kiểm tra O(1), tổng O(n). Hoặc tinh tế hơn: in-order traversal phải cho dãy tăng ngặt — dùng luôn định lý ở mục 4 làm máy kiểm tra. Bài này là phép thử xem ứng viên hiểu invariant là thuộc tính *toàn cục được duy trì đệ quy* hay chỉ nhớ hình vẽ.

**Lowest Common Ancestor (LeetCode 235/236).** Phiên bản BST (235) thưởng cho người dùng invariant: LCA là node *đầu tiên trên đường từ gốc* mà hai khóa nằm về hai phía (hoặc trúng node) — đi một đường O(h), không cần đệ quy hai nhánh. Phiên bản cây thường (236) mất invariant, phải đệ quy cả hai con: "node này là LCA nếu hai target nằm ở hai subtree khác nhau" — O(n). Cặp bài này minh họa nguyên lý đáng giá cả trong thiết kế hệ thống: **invariant mạnh hơn mua được thuật toán rẻ hơn**; khi bạn bỏ tiền duy trì thứ tự (BST, index), hãy nhớ dùng nó.

**Traversal không đệ quy (LeetCode 94/144/145)** — kiểm tra bạn có hiểu đệ quy chỉ là stack ngầm: chuyển được sang stack tường minh (mục 5) chứng tỏ hiểu cơ chế, không chỉ cú pháp. Câu hỏi nối tiếp thường gặp: "độ sâu stack tối đa bao nhiêu?" — đáp án O(h), và nếu bạn chủ động nói thêm "với cây suy biến là O(n), nên input không tin cậy thì bản iterative an toàn hơn" — bạn vừa trả lời như một kỹ sư chứ không phải một người luyện đề.

**Lỗi tư duy thường gặp:** mặc định cây trong đề là cân (độ phức tạp phải trả lời theo h, và nói rõ h ∈ [log n, n]); quên trường hợp cây rỗng/một node (base case của induction chính là test case đầu tiên); dùng giá trị node làm định danh khi cây có khóa trùng; và với mọi bài cây — không nhận ra rằng lời giải đệ quy chuẩn chỉ là *một câu induction*: "giả sử hàm đúng với hai subtree, ghép thế nào cho node hiện tại?" Ai nghĩ được câu đó thì mọi bài cây mới đều quy về việc điền một mẫu có sẵn.

## 9. Anti-pattern

**BST ngây thơ nhận dữ liệu sorted.** Nạp ID tự tăng, timestamp, hay bất kỳ dòng dữ liệu gần-đơn-điệu nào vào BST không cân bằng — nhận về linked list O(n) đội lốt cây. Nếu tự cài BST (hiếm khi nên), input sorted là *trường hợp phải test đầu tiên* chứ không phải trường hợp hiếm. Hiện tượng này có họ hàng production thật: chèn khóa đơn điệu (ID tăng dần) vào B-Tree khiến mọi insert dồn vào lá phải cùng — PostgreSQL có tối ưu riêng (fastpath cho rightmost leaf) chính vì pattern này phổ biến.

**Đệ quy không chặn độ sâu trên cây do người dùng dựng.** JSON/XML/YAML lồng sâu, cây biểu thức từ query độc — mỗi tầng một stack frame, và kẻ xấu gửi 10⁵ tầng. Bài học chương 06, phiên bản cấu trúc dữ liệu: **chiều cao cây là tài nguyên do input kiểm soát** — chặn nó ở biên, hoặc dùng stack tường minh.

**Chọn cây khi map + slice đủ dùng.** Tự cài Red-Black tree (hoặc kéo dependency) cho một config lookup 200 phần tử đọc nghìn lần ghi một lần — trong khi `map` hoặc sorted slice + `sort.Search` nhanh hơn, ít bug hơn, ai cũng đọc được. Cây cân bằng là công cụ cho *ghi nhiều + cần thứ tự + n lớn*; thiếu một trong ba vế, tồn tại lựa chọn đơn giản hơn đang thắng.

**Đánh index như rắc muối.** Mỗi index là một B-Tree sống: chiếm đĩa (index 23GB ở mục 7 là số thật), ăn RAM cache, và đánh thuế mọi lần ghi. Index không được query nào dùng là thuế không hoàn — `pg_stat_user_indexes` tồn tại để tìm và xử chúng.

**Mô hình hóa quan hệ không phân cấp bằng cây.** Sơ đồ tổ chức có nhân viên hai quản lý, danh mục sản phẩm đa chiều, dependency có chu trình — ép vào cây (bảng `parent_id` duy nhất) rồi vá bằng bản ghi trùng lặp và cờ đặc biệt. Định lý n−1 cạnh là máy phát hiện: đếm thấy quan hệ nhiều hơn "số thực thể trừ một", hoặc một thực thể có hai cha — mô hình của bạn là đồ thị, hãy đối xử với nó như đồ thị (chương 08).

## 10. Best Practices

**Nên:**

- Trước khi chọn cấu trúc có thứ tự, hỏi ba câu: có range/order query không (không → hash/map); ghi có thường xuyên không (không → sorted slice + binary search); n có lớn không (không → scan phẳng). Cây cân bằng là đáp án khi cả ba cùng "có".
- Ước lượng chiều cao B-Tree index của bảng lớn nhất bạn quản lý (số dòng, kích thước khóa, page 8KB — phép tính 2 phút như mục 7) để biết mỗi index lookup *thật sự* tốn mấy I/O, và index tốn bao nhiêu GB.
- Với mọi thuật toán trên cây, trả lời chi phí theo **h** rồi mới thay h = log n *kèm điều kiện cân bằng* — thói quen này tự động lộ ra worst case O(n).
- Viết thuật toán cây theo mẫu induction: base case (nil), giả thiết hai subtree đã đúng, bước ghép tại node. Chứng minh và code là một.
- Khi chọn storage engine, đặt câu hỏi B-Tree vs LSM một cách tường minh theo tỷ lệ đọc/ghi và loại query — đừng nhận mặc định của vendor làm chân lý.

**Không nên:**

- Không dùng đệ quy độ sâu O(h) trên cây mà kẻ khác kiểm soát hình dạng.
- Không tự cài cây cân bằng cho production khi `google/btree` hoặc cấu trúc đơn giản hơn tồn tại — rotation là code dễ sai bậc nhất, và bug của nó im lặng làm hỏng invariant thay vì crash.
- Không quên rằng in-order = sorted chỉ đúng với BST — heap và cây tổng quát không cho bạn thứ tự miễn phí.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. B-Tree không giảm số phép so sánh so với binary tree cân bằng — vậy chính xác nó tiết kiệm cái gì, và tại sao lợi thế đó biến mất (một phần) khi dữ liệu nằm trọn trong RAM?
2. Chứng minh cây n đỉnh có n−1 cạnh dùng thao tác "ngắt một lá" — vì sao một cây hữu hạn luôn có lá, và bước đó liên hệ thế nào với base case của thuật toán đệ quy trên cây?
3. Hệ thống của bạn ghi 50.000 event/giây và thỉnh thoảng đọc lại theo khoảng thời gian — B-Tree hay LSM, và bạn đang trả giá bằng gì ở phía còn lại?

---

*Chương tiếp theo: [08 — Graph Theory](/series/math-for-engineers/level-2-discrete-mathematics/08-graphs/), nơi ta thả lỏng hai ràng buộc của cây — cho phép chu trình và nhiều đường đi — và nhận về mô hình của mạng máy tính, dependency, microservices: đồ thị, cùng bộ thuật toán BFS, DFS, shortest path và topological sort.*
