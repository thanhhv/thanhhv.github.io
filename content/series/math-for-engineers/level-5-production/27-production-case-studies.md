+++
title = "Chương 27 — Production Case Studies: Toán học trong các hệ thống tỷ đô"
date = "2026-07-20T11:30:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 5 – Production
> Yêu cầu trước: toàn bộ tài liệu — chương này là nơi mọi mảnh ghép gặp nhau

---

## Mở đầu: đọc kiến trúc như đọc một chứng minh

Sau 26 chương, bạn đã có trong tay một hộp công cụ: đếm và xác suất, graph và cây, DP và greedy, đại số tuyến tính và thống kê, entropy và số học modular. Chương cuối này làm một việc duy nhất: **mở nắp capo của mười hệ thống tỷ đô và chỉ tay vào đúng chỗ toán học đang nằm**.

Triết lý xuyên suốt: *không có hệ thống lớn nào được xây từ một ý tưởng thiên tài đơn lẻ*. PostgreSQL, Redis, Kafka, Google Search — tất cả đều là **tổ hợp của các khái niệm toán bạn đã học**, lắp ráp lại dưới những ràng buộc kinh doanh cụ thể. Điều phân biệt kỹ sư đọc được kiến trúc với kỹ sư chỉ dùng được kiến trúc là khả năng trả lời câu hỏi: *"quyết định thiết kế này đang giải mô hình toán nào, và đã đánh đổi cái gì?"*

Mỗi case study dưới đây đi theo đúng một khung năm bước:

> **Bài toán kinh doanh → Mô hình toán → Thuật toán → Trade-off đã chọn → Chương liên quan**

Đừng đọc như đọc danh sách sự kiện. Hãy đọc như giải mười bài tập lớn — với lời giải kèm sẵn.

---

## Case 1 — PostgreSQL Query Optimizer: đánh bại vụ nổ giai thừa trong 10 millisecond

**Bài toán kinh doanh.** Người dùng viết SQL — một *đặc tả khai báo* nói "tôi muốn gì", không nói "làm thế nào". Cùng một câu JOIN 8 bảng có thể thực thi theo hàng nghìn cách, cách nhanh nhất và chậm nhất chênh nhau **hàng triệu lần**. Database phải tìm cách tốt trong vài millisecond — vì thời gian lập kế hoạch cũng tính vào độ trễ của query.

**Mô hình toán.** Không gian tìm kiếm bùng nổ theo giai thừa (chương 05): chỉ tính các cây join trái-sâu (left-deep), n bảng có n! thứ tự — 8 bảng là 40.320, 12 bảng là ~479 triệu; cho phép cây rậm (bushy tree) còn tệ hơn nhiều. Duyệt hết là bất khả thi. Nhưng bài toán có **optimal substructure** (chương 14): kế hoạch tốt nhất để join tập bảng S luôn được ghép từ kế hoạch tốt nhất của hai tập con rời nhau của S — nghĩa là DP áp dụng được, thu không gian từ "mọi hoán vị" xuống "mọi tập con": O(n!) → O(3ⁿ) tổng chi phí duyệt các cách tách tập con. Nhánh thứ hai của mô hình là **ước lượng chi phí**: mỗi kế hoạch chỉ so sánh được qua con số dự đoán, và dự đoán dựa trên thống kê mô tả dữ liệu (chương 18) — histogram equi-depth (mặc định 100 bucket trong `pg_stats`), danh sách giá trị phổ biến nhất (MCV), số giá trị khác nhau (`n_distinct`). Selectivity của `WHERE price < 100` được tra từ histogram; của điều kiện AND được *nhân* các selectivity — tức giả định các cột **độc lập xác suất** (chương 11), một giả định sẽ quay lại cắn ở phần trade-off.

**Thuật toán.** Bộ tối ưu System R (IBM, 1979 — vẫn là xương sống của PostgreSQL hôm nay): DP từ dưới lên — tính kế hoạch tốt nhất cho từng bảng đơn, rồi từng cặp, từng bộ ba... mỗi tập con giữ lại kế hoạch rẻ nhất (cộng thêm các kế hoạch có "interesting order" — output đã sort có thể rẻ hơn ở bước sau, một dạng trạng thái DP mở rộng). Khi số bảng vượt `geqo_threshold` (mặc định 12), PostgreSQL thừa nhận DP cũng quá đắt và chuyển sang **GEQO — genetic algorithm** (chương 21): coi mỗi thứ tự join là một "nhiễm sắc thể", lai ghép và đột biến qua nhiều thế hệ, trả về lời giải *tốt* thay vì *tối ưu*.

**Trade-off đã chọn.** PostgreSQL chấp nhận ba thứ: (1) thời gian lập kế hoạch đổi lấy chất lượng kế hoạch — DP đầy đủ dưới 12 bảng, heuristic phía trên; (2) giả định độc lập giữa các cột — rẻ và thường đúng, nhưng với cột tương quan (`city = 'Hà Nội' AND country = 'VN'`) selectivity bị *nhân thừa*, ước lượng thấp đi hàng trăm lần → chọn nested loop thay vì hash join → query chậm 1000 lần. Bản vá là `CREATE STATISTICS` (extended statistics) — bạn khai báo cho optimizer biết cột nào phụ thuộc nhau; (3) thống kê là mẫu (sample) cập nhật định kỳ bởi `ANALYZE` — luôn hơi cũ, đổi lấy chi phí thu thập thấp.

**Chương liên quan:** 05 (bùng nổ tổ hợp), 11 (độc lập xác suất), 14 (DP trên tập con), 18 (histogram, sampling), 21 (genetic/heuristic).

---

## Case 2 — Redis: bốn quyết định toán học trong một event loop

**Bài toán kinh doanh.** Cache và cấu trúc dữ liệu in-memory với độ trễ trăm microsecond, phục vụ hàng trăm nghìn request/giây trên **một thread**. Ràng buộc một thread là chìa khóa đọc mọi thiết kế của Redis: *bất kỳ thao tác nào O(n) với n lớn đều làm đứng toàn bộ server* — nên mọi chi phí lớn phải được cắt nhỏ hoặc xấp xỉ hóa.

**Mô hình toán và thuật toán — bốn lát cắt:**

*Incremental rehashing.* Hash table đầy thì phải rehash — copy hàng triệu key sang bảng mới là O(n), tức một cú đứng hình vài trăm ms, không chấp nhận được. Redis giữ **hai bảng song song** (`ht[0]` cũ, `ht[1]` mới) và mỗi lệnh đọc/ghi tiện tay di cư một bucket. Đây chính là amortized analysis (chương 10) áp dụng có chủ đích: tổng chi phí rehash không đổi, nhưng được *trả góp* đều lên hàng triệu lệnh — worst case mỗi lệnh vẫn O(1), latency P99 phẳng lặng. Cùng một tổng chi phí, phân bố khác, số phận sản phẩm khác.

*Skip list cho Sorted Set.* Cần cấu trúc có thứ tự với insert/search/range O(log n). Red-black tree làm được — vậy sao Antirez chọn skip list (chương 23)? Lý do ông nêu công khai: (1) cài đặt và debug đơn giản hơn hẳn — ít trạng thái, không có sáu ca xoay cây; (2) `ZRANGEBYSCORE` duyệt range tự nhiên bằng con trỏ forward ở tầng đáy; (3) chỉnh hệ số p (xác suất lên tầng, Redis dùng 1/4) là chỉnh trade-off bộ nhớ/tốc độ bằng *một hằng số*. Cân bằng của skip list đến từ xác suất thay vì bất biến cấu trúc — kỳ vọng O(log n) với phương sai nhỏ đủ dùng, và "đơn giản để bảo trì" là một tiêu chí kỹ thuật nghiêm túc, không phải sự lười biếng.

*HyperLogLog.* Đếm số phần tử khác nhau (unique visitors) trong tập hàng tỷ, chính xác tuyệt đối cần O(n) bộ nhớ. Redis dành đúng **12KB** cho mỗi HLL: 16384 register, sai số chuẩn 1.04/√16384 ≈ **0.81%** (chương 23). Với nghiệp vụ "biểu đồ unique user theo ngày", sai 1% là vô hình; tiết kiệm bộ nhớ là sáu bậc độ lớn.

*LRU xấp xỉ.* LRU đúng nghĩa cần doubly-linked list toàn cục (chương 24) — thêm 2 con trỏ cho *mỗi* key và thao tác move-to-front trên mỗi lần đọc. Redis thay bằng **sampling**: mỗi key mang một đồng hồ truy cập 24-bit; khi cần đuổi, lấy mẫu 5 key ngẫu nhiên (`maxmemory-samples`), đuổi key cũ nhất trong mẫu (bản 3.0 thêm pool 16 ứng viên giữ giữa các lần). Xác suất mẫu bỏ lọt toàn bộ nhóm key cũ giảm theo hàm mũ của cỡ mẫu (chương 11) — chất lượng gần LRU thật với chi phí cận zero.

**Trade-off đã chọn.** Mẫu số chung của cả bốn: **đổi sự chính xác/tối ưu tuyệt đối lấy hằng số nhỏ và worst case phẳng**. Redis không có thao tác nào "thỉnh thoảng chậm" — triết lý bắt buộc khi cả server là một thread.

**Chương liên quan:** 10 (amortized), 11 (sampling), 12 (hash table), 23 (skip list, HLL), 24 (LRU).

---

## Case 3 — Kafka: sức mạnh của hằng số và một chiếc log tuần tự

**Bài toán kinh doanh.** Đường ống sự kiện trung tâm của doanh nghiệp: hàng triệu message/giây, không mất dữ liệu, nhiều consumer đọc lại tùy ý. Yêu cầu nghe như cần siêu máy tính — Kafka giải bằng vài máy thường, nhờ tôn trọng một sự thật mà Big-O cố tình che: **hằng số của I/O tuần tự và I/O ngẫu nhiên chênh nhau hàng trăm lần**.

**Mô hình toán.** Ba tầng: (1) *Partition* — message có key được gán vào partition bằng `murmur2(key) % numPartitions` (chương 12): cùng key → cùng partition → **thứ tự chỉ được bảo đảm trong một partition** — Kafka định nghĩa thẳng bằng toán rằng thứ tự toàn cục không tồn tại (bài học chương 25), và bán cho bạn thứ tự cục bộ với giá rẻ; (2) *Độ bền* — ISR + `min.insync.replicas` tạo bất biến giao nhau kiểu quorum (chương 25): `acks=all` với min.insync=2 trên replication factor 3 nghĩa là mọi message được xác nhận nằm trên ≥2 máy, chịu được 1 máy chết; (3) *Nén* — entropy (chương 20): nén từng message thì mô hình thống kê của bộ nén chưa kịp "học" đã hết dữ liệu; nén **cả batch** cho phép các message cùng schema chia sẻ ngữ cảnh — cùng thuật toán, tỷ lệ nén tốt hơn nhiều lần chỉ nhờ đổi *đơn vị nén*.

**Thuật toán.** Mỗi partition là một **append-only log**: ghi luôn là append tuần tự (tận dụng page cache của OS và prefetch của ổ đĩa), đọc là quét tuần tự từ một offset. Consumer offset chỉ là con trỏ số nguyên — "đọc lại từ hôm qua" là phép trừ, không phải truy vấn. Đường truyền dùng **zero-copy** (`sendfile`): dữ liệu đi thẳng từ page cache ra socket, không qua user space — bốn lần copy còn hai. Không có cấu trúc dữ liệu nào thông minh hơn mảng ở đây; toàn bộ độ "khôn" nằm ở việc *sắp mọi thao tác vào hàng thẳng*.

```
Producer ──append──▶ [ 0 | 1 | 2 | 3 | 4 | 5 | 6 | ...  log tuần tự
                                   ▲           ▲
                        consumer A đọc từ 3    consumer B đọc từ 6
```

**Trade-off đã chọn.** Kafka *từ bỏ* những thứ message queue truyền thống có: xóa từng message, thứ tự toàn cục, index phức tạp trên nội dung. Đổi lại: throughput bị chặn gần như chỉ bởi băng thông đĩa/mạng — nghĩa là *dự đoán được* và *scale tuyến tính*. Đây là ví dụ đẹp nhất trong tài liệu này về bài học chương 10: khi hai thiết kế cùng bậc O(n), kẻ thắng là kẻ có hằng số nhỏ hơn trăm lần.

**Chương liên quan:** 10 (hằng số, sequential I/O), 12 (partition bằng hash), 20 (entropy, nén theo batch), 25 (ISR/quorum).

---

## Case 4 — Google PageRank: cả World Wide Web là một ma trận

**Bài toán kinh doanh.** Năm 1998, các search engine xếp hạng bằng nội dung trang — và bị spam từ khóa nuốt chửng. Cần một tín hiệu chất lượng *không nằm trong tay tác giả trang web*. Ý tưởng của Page và Brin: chất lượng của một trang nằm trong **cấu trúc liên kết của toàn bộ web** — mỗi link là một phiếu bầu, và phiếu của trang uy tín nặng hơn.

**Mô hình toán.** Chuỗi ba bước biến đổi, mỗi bước là một chương của tài liệu này: (1) Web là một **directed graph** (chương 08) — trang là đỉnh, hyperlink là cạnh; (2) Định nghĩa "uy tín" đệ quy ("trang uy tín là trang được trang uy tín trỏ tới") nghe như vòng lặp vô hạn, cho đến khi đổi góc nhìn: tưởng tượng một **random surfer** lang thang — mỗi bước nhảy ngẫu nhiên theo một link trên trang hiện tại. Đó là một **Markov chain** (chương 11), và "uy tín" = xác suất dài hạn surfer đứng ở trang đó — phân phối dừng (stationary distribution); (3) Phân phối dừng π thỏa **π = πP** với P là ma trận chuyển — tức π là **eigenvector** của P ứng với eigenvalue 1 (chương 17). Định nghĩa đệ quy tưởng vô hạn hóa ra là một phương trình đại số tuyến tính có nghiệm.

Nhưng web thô làm mô hình gãy ở hai chỗ: **dangling node** (trang không có link ra — surfer kẹt, xác suất "rò" mất) và **spider trap** (cụm trang chỉ trỏ vòng nhau — hút toàn bộ xác suất về mình). Bản vá là **damping factor**: với xác suất d = 0.85 surfer theo link, với 0.15 surfer *dịch chuyển tức thời* đến trang ngẫu nhiên bất kỳ:

> G = 0.85·S + 0.15·(1/n)·J   (S: ma trận chuyển đã vá dangling, J: ma trận toàn 1)

Cú teleport này làm chuỗi Markov **irreducible và aperiodic** — định lý Perron–Frobenius khi đó bảo đảm phân phối dừng *tồn tại và duy nhất*, đồng thời chặn eigenvalue thứ hai |λ₂| ≤ 0.85, quyết định tốc độ hội tụ.

**Thuật toán.** **Power iteration** (chương 17): khởi tạo π đều, lặp π ← πG đến khi hội tụ. Mỗi vòng chỉ là một phép nhân vector với ma trận *cực thưa* (web có ~10 link/trang chứ không phải n link) — O(số cạnh) mỗi vòng, phân tán được trên nghìn máy (MapReduce sinh ra một phần vì bài toán này). Sai số co theo 0.85^k: cần độ chính xác 10⁻⁸ thì k ≈ log(10⁻⁸)/log(0.85) ≈ 113 vòng — **vài chục tỷ trang, hội tụ trong ~100 vòng nhân ma trận**. Damping 0.85 vì thế không phải con số ngẫu nhiên: tăng lên 0.99 thì "trung thực" với cấu trúc web hơn nhưng hội tụ chậm hơn hàng chục lần và nhạy cảm với spider trap; hạ xuống 0.5 thì hội tụ nhanh nhưng xếp hạng bị "pha loãng" về đều nhau.

**Trade-off đã chọn.** PageRank là điểm số *toàn cục, tĩnh, độc lập với query* — tính trước offline, tra cứu O(1) lúc search. Google đổi sự "đúng theo ngữ cảnh" lấy khả năng tính trước ở quy mô web; phần ngữ cảnh được bù bằng hàng trăm tín hiệu khác lúc runtime. Và vì công thức công khai, nó bị tấn công (link farm) — cuộc rượt đuổi spam bắt đầu từ chính thành công của mô hình.

**Chương liên quan:** 08 (đồ thị), 11 (Markov chain, phân phối dừng), 17 (eigenvector, power iteration).

---

## Case 5 — Elasticsearch Ranking: BM25 và nghệ thuật đo "độ liên quan"

**Bài toán kinh doanh.** Người dùng gõ "lỗi kết nối database", kho có 50 triệu document. Trả về document *chứa* các từ đó chưa đủ — phải **xếp hạng**: document nào liên quan nhất? "Liên quan" là khái niệm mờ; muốn máy tính xử lý, phải định lượng nó.

**Mô hình toán.** Trực giác nền TF-IDF gồm hai vế nhân nhau: từ xuất hiện *nhiều trong document này* (TF) thì document liên quan hơn; từ xuất hiện *hiếm trong toàn kho* (IDF) thì mang nhiều thông tin hơn — "database" đáng giá, "the" vô giá trị. IDF = log(N/df) chính là **self-information** của sự kiện "document chứa từ này" (chương 20): từ càng hiếm, sự hiện diện của nó càng bất ngờ, càng nhiều bit thông tin. Nhưng TF thô có hai bệnh: từ lặp 100 lần không đáng giá gấp 100 lần lặp 1 lần (spam từ khóa!), và document dài tự nhiên chứa nhiều từ hơn. **BM25** — chuẩn mặc định của Elasticsearch/Lucene — chữa cả hai:

> score(q, d) = Σ_{t∈q} IDF(t) · ( tf·(k₁+1) ) / ( tf + k₁·(1 − b + b·|d|/avgdl) )

Đọc từng bộ phận: (1) tử và mẫu cùng chứa tf, nên khi tf → ∞ tỷ số tiến tới **giới hạn bão hòa** (k₁+1) — chứ không tăng vô hạn như TF thô; **k₁ ≈ 1.2** điều khiển tốc độ bão hòa: k₁ nhỏ → lần xuất hiện thứ hai gần như vô giá trị, k₁ lớn → tf tiếp tục có tiếng nói; (2) cụm (1 − b + b·|d|/avgdl) **phạt document dài hơn trung bình** (avgdl = độ dài trung bình toàn kho); **b = 0.75** là núm chỉnh mức phạt: b = 0 tắt hẳn (mọi document bình đẳng bất kể độ dài), b = 1 phạt tuyến tính đầy đủ; (3) IDF của BM25 có dạng làm mượt ln(1 + (N − df + 0.5)/(df + 0.5)) — tránh chia cho 0 và tránh IDF âm với từ quá phổ biến. Mỗi hằng số trong công thức là một quyết định mô hình hóa có thể giải thích — không có con số nào "rơi từ trên trời".

**Thuật toán.** Nền tảng tốc độ là **inverted index** (đổi biến — bài học chương 10): thay vì quét document tìm từ, tra từ ra danh sách document (postings list) đã sắp theo doc ID, giao các danh sách bằng thuật toán merge. Còn từ điển hàng trăm triệu term được nén bằng **FST (Finite State Transducer)** — dạng automaton tối tiểu hóa, họ hàng nén chung tiền tố *và hậu tố* của trie (chương 24): cả từ điển term nằm gọn trong RAM, tra cứu O(độ dài từ), hỗ trợ luôn prefix/fuzzy search.

**Trade-off đã chọn.** BM25 là mô hình **bag-of-words**: không hiểu ngữ nghĩa, "ô tô" và "xe hơi" là hai token xa lạ. Elasticsearch chọn nó làm mặc định vì: không cần huấn luyện, giải thích được từng điểm số (debug ranking là nhu cầu thật), và cực rẻ ở quy mô lớn. Tầng ngữ nghĩa (dense vector, kNN — chương 17) được xếp *chồng lên* làm bước re-rank cho top kết quả — kiến trúc hai tầng rẻ-trước-đắt-sau kinh điển.

**Chương liên quan:** 11 (nền xác suất của IR), 20 (IDF là self-information), 24 (trie/FST, inverted index), 17 (tầng vector hiện đại).

---

## Case 6 — Git: Merkle DAG và bài toán diff 40 năm tuổi

**Bài toán kinh doanh.** Hàng nghìn lập trình viên sửa cùng codebase, cần: lịch sử không thể giả mạo, so sánh hai phiên bản *nhanh*, đồng bộ qua mạng *ít dữ liệu*, và hiển thị "ai đổi gì" dễ đọc. Linus Torvalds viết lõi Git trong vài tuần năm 2005 — nhanh vậy được vì mọi bài toán con đều đã có lời giải toán học sẵn.

**Mô hình toán.** Nền móng là **content-addressable storage**: định danh của mọi object = SHA-1 của nội dung (chương 12, 26). Hệ quả tức thì: nội dung giống nhau tự động dedup (một file xuất hiện trong nghìn commit chỉ lưu một lần); nội dung đổi một bit → địa chỉ đổi → **không thể sửa lịch sử mà không bị phát hiện**. Tầng trên là **Merkle DAG** (chương 26): blob (nội dung file) ← tree (thư mục, chứa hash các blob/tree con) ← commit (chứa hash tree gốc + hash commit cha). Hash của commit vì thế "niêm phong" *toàn bộ* trạng thái repo và toàn bộ lịch sử dẫn đến nó — so sánh hai snapshot bắt đầu bằng so sánh **một** hash; khác nhau mới đệ quy xuống đúng nhánh khác nhau. Chi phí so sánh tỷ lệ với *kích thước khác biệt*, không phải kích thước repo — đây cũng chính là lý do `git push` chỉ truyền object hai bên chưa có chung.

**Thuật toán.** `git diff` dùng **thuật toán Myers (1986)**: tìm *shortest edit script* — chuỗi thao tác xóa/chèn ngắn nhất biến file A (N dòng) thành file B (M dòng). Nước cờ đẹp nhất là phiên dịch bài toán: dựng **edit graph** — lưới (N+1)×(M+1), đi ngang = xóa một dòng của A (chi phí 1), đi dọc = chèn một dòng của B (chi phí 1), đi chéo = hai dòng khớp nhau (chi phí 0). Khi đó *shortest edit script = shortest path từ góc (0,0) đến (N,M)* — bài toán edit distance của DP (chương 14) hóa ra là bài shortest path của graph (chương 08), hai chương bắt tay nhau trong một thuật toán. Myers khai thác cấu trúc đặc biệt (cạnh chéo miễn phí) bằng kỹ thuật greedy "furthest reaching path" trên từng đường chéo: O((N+M)·D) với D là cỡ của diff — hai file gần giống nhau (trường hợp áp đảo trong version control) diff gần như tuyến tính. Tầng lưu trữ còn một lớp toán nữa: **packfile** xếp các object *giống nhau* cạnh nhau (cùng tên file qua các phiên bản) và lưu **delta** — object = base + chuỗi chỉnh sửa, chain có giới hạn độ sâu để đọc không quá đắt. Nén lịch sử 10 năm của một file thành kích thước xấp xỉ *một* phiên bản cộng các thay đổi.

**Trade-off đã chọn.** Git lưu **snapshot, không lưu diff** ở tầng logic (diff chỉ được *tính ra* khi cần và ở tầng nén packfile) — hào phóng dung lượng thô để đổi lấy checkout/branch O(1) và mô hình dữ liệu đơn giản. Chọn SHA-1 năm 2005 là hợp lý; khi va chạm SHA-1 thành hiện thực (SHAttered, 2017 — chương 26), Git phải trả nợ kỹ thuật bằng hardened SHA-1 và lộ trình chuyển SHA-256 — lời nhắc rằng giả định mật mã cũng có ngày hết hạn.

**Chương liên quan:** 08 (shortest path trên edit graph), 12 (content addressing), 14 (edit distance/DP), 26 (hash mật mã, Merkle tree).

---

## Case 7 — Kubernetes Scheduler: bin packing NP-hard, giải mỗi giây một lần

**Bài toán kinh doanh.** Cluster 5.000 node, mỗi giây có Pod mới cần chỗ chạy. Đặt Pod vào node nào để: đủ tài nguyên, thỏa ràng buộc (Pod A phải cùng zone với B, tránh node có GPU đang bận...), cluster cân bằng, và chi phí thấp? Đặt xấu là lãng phí hàng triệu đô tài nguyên thừa hoặc cascade failure khi một node quá tải.

**Mô hình toán.** Lột bỏ thuật ngữ, đây là **bin packing đa chiều** (mỗi node là thùng với vector dung lượng CPU/RAM/disk, mỗi Pod là món đồ với vector yêu cầu) cộng **ràng buộc logic** (affinity/anti-affinity, taint/toleration — các mệnh đề phải thỏa, chương 01). Bin packing là **NP-hard** (chương 21): không có thuật toán đa thức tìm cách xếp tối ưu, và scheduler chỉ có ngân sách vài chục millisecond mỗi Pod. Kết luận toán học lạnh lùng: *đừng tìm tối ưu — tìm đủ tốt, nhanh*. Thêm một tầng khó: bài toán là **online** — Pod đến tuần tự, quyết định hôm nay không biết Pod ngày mai, mà lý thuyết online algorithm chứng minh mọi chiến lược online đều có thể bị input tương lai làm cho suboptimal.

**Thuật toán.** Hai pha, tra đúng từ hộp công cụ: **Filter** — loại node vi phạm ràng buộc cứng (đủ tài nguyên? chịu được taint? khớp affinity?) — một bộ lọc constraint satisfaction, mỗi predicate là một mệnh đề AND; **Score** — mỗi plugin chấm node còn lại thang 0–100 (ít tài nguyên đã dùng, ảnh image có sẵn, rải đều topology...), điểm tổng là **weighted sum** (tổ hợp tuyến tính — chương 17, dùng như hàm mục tiêu vô hướng hóa đa mục tiêu, chương 21); chọn node điểm cao nhất — **greedy** thuần túy (chương 15), một Pod một quyết định, không quay lui. Ở cluster nghìn node, scoring toàn bộ cũng đắt → `percentageOfNodesToScore`: chỉ chấm một *mẫu* node đủ dùng (sampling — chương 11, đánh cược rằng node tốt-gần-nhất trong mẫu đủ gần node tốt nhất toàn cục). Và vì greedy online *chắc chắn* trôi dần khỏi tối ưu khi cluster biến động (Pod cũ chết, node mới thêm), hệ sinh thái có **descheduler** — vòng lặp chạy nền tìm Pod đặt "lệch" và đuổi chúng để scheduler xếp lại: một cơ chế *sửa sai định kỳ* thay cho tối ưu toàn cục không thể có.

**Trade-off đã chọn.** Ba lần đánh đổi có tên: tối ưu ↔ độ trễ quyết định (greedy thay vì ILP solver); điểm số tuyến tính ↔ biểu đạt (weighted sum không nói được "chỉ tốt khi cả hai điều kiện cùng xảy ra", nhưng cấu hình được bằng YAML và giải thích được); spread ↔ bin-pack — rải Pod ra (LeastAllocated, mặc định) tăng khả năng chịu lỗi, dồn Pod lại (MostAllocated) tiết kiệm tiền cloud: cùng framework, đổi *một hàm score* là đổi cả triết lý vận hành.

**Chương liên quan:** 01 (predicate logic), 08 (topology/graph constraints), 15 (greedy), 21 (NP-hard, xấp xỉ, weighted objective), 11 (sampling).

---

## Case 8 — Google Maps: khi Dijkstra không đủ nhanh cho một lục địa

**Bài toán kinh doanh.** "Đường nhanh nhất từ Hà Nội đến TP.HCM" trên đồ thị đường bộ hàng chục triệu đỉnh, hàng trăm triệu cạnh — trả lời trong vài millisecond, cho hàng triệu query đồng thời, kèm ETA chính xác đến mức người dùng lấy nó ra hẹn giờ đón con.

**Mô hình toán.** Bản đồ là **weighted graph** (chương 08): giao lộ là đỉnh, đoạn đường là cạnh, trọng số là thời gian di chuyển. Dijkstra giải shortest path đúng trong O(E + V log V) — nhưng trên đồ thị lục địa, một query xuyên quốc gia buộc Dijkstra "loang" qua *hàng triệu* đỉnh: hàng giây CPU. Nhân với triệu QPS — bất khả thi. Quan sát cứu vãn: **đồ thị đường bộ không đổi hàng tháng, còn query đến hàng triệu lần mỗi giây** → đổ chi phí về phía preprocessing (trade-off space/time kinh điển, chương 10). Ở tầng số liệu, khoảng cách "đường chim bay" giữa hai tọa độ phải tính trên mặt cầu — công thức **Haversine** (chương 22) với bán kính Trái Đất ~6371 km — dùng cho ước lượng, tìm điểm gần, và heuristic.

**Thuật toán.** **Contraction Hierarchies (CH)**: giai đoạn tiền xử lý xếp các đỉnh theo "độ quan trọng" (ngõ cụt ít quan trọng, nút giao cao tốc rất quan trọng) rồi **contract** từng đỉnh từ thấp lên cao — gỡ đỉnh ra, và nếu việc gỡ làm mất một đường đi ngắn nhất nào đó giữa hai hàng xóm thì thêm một **shortcut** (cạnh tắt mang đúng tổng trọng số) thay thế. Kết quả là đồ thị gốc + tập cạnh tắt được phân tầng. Lúc query: chạy **Dijkstra hai đầu** (từ điểm đi và điểm đến đồng thời) với luật duy nhất — *chỉ đi lên tầng quan trọng hơn*. Trực giác đời thường chính xác đến bất ngờ: đi xa thì từ ngõ ra phố, ra quốc lộ, chạy cao tốc, rồi mới chui dần xuống ngõ ở đầu kia — không ai xuyên tỉnh bằng đường làng. Không gian tìm kiếm co từ hàng triệu đỉnh về **vài trăm đỉnh mỗi phía**: query từ giây xuống *dưới millisecond*, nhanh hơn Dijkstra thô hàng nghìn lần, và vẫn **chính xác tuyệt đối** (CH là exact algorithm, không phải heuristic — điều làm nó đẹp hơn A* trong bối cảnh này). Còn ETA — trọng số cạnh thật sự — không phải hằng số mà là *dự đoán*: traffic lịch sử + realtime, và từ 2020 Google dùng **Graph Neural Network** (DeepMind) dự đoán tốc độ từng đoạn đường theo ngữ cảnh, giảm sai số ETA tới ~40% ở một số thành phố. Đồ thị chỉ cho *cấu trúc* lời giải; trọng số đúng phải mua bằng machine learning.

**Trade-off đã chọn.** CH trả trước hàng giờ CPU preprocessing + bộ nhớ cho cạnh tắt, đổi lấy query gần như miễn phí — hợp lý *vì* tỷ lệ đọc/ghi cực cao. Điểm yếu lộ ra đúng chỗ giả định gãy: trọng số đổi liên tục theo traffic thì shortcut tính trước sai → cần biến thể **Customizable CH** tách "cấu trúc" (tính lại hàng tuần) khỏi "trọng số" (customize lại trong phút) — một lần nữa, kiến trúc đi theo hình dạng của trade-off.

**Chương liên quan:** 08 (shortest path, Dijkstra hai đầu), 22 (Haversine, hình học cầu), 10 (preprocessing vs query time).

---

## Case 9 — Netflix Recommendation: nén 200 triệu khẩu vị vào vài chục con số

**Bài toán kinh doanh.** Hơn 200 triệu subscriber, hàng chục nghìn nội dung, và một sự thật vận hành: phần lớn giờ xem đến từ đề xuất, không phải từ ô tìm kiếm. Đề xuất trúng giữ chân người dùng; đề xuất trượt là churn — với doanh thu subscription, "gợi ý đúng phim" đáng giá hàng tỷ đô mỗi năm.

**Mô hình toán.** Xếp dữ liệu thành **ma trận user × item** (chương 17): dòng là người, cột là phim, ô là mức độ thích (rating, hoặc tín hiệu ngầm: xem hết/bỏ dở). Ma trận này **thưa khủng khiếp** — mỗi người chỉ chạm ~vài trăm trong hàng chục nghìn nội dung, hơn 99% ô trống — và bài toán đề xuất chính là bài **matrix completion**: điền các ô trống. Giả thuyết làm bài toán giải được: khẩu vị con người có **cấu trúc chiều thấp** — không có 200 triệu kiểu khẩu vị độc lập, chỉ có vài chục "trục" tiềm ẩn (nhiều kịch tính? hài đen? nhịp chậm?...). Toán hóa: **matrix factorization** — tìm R ≈ U·Vᵀ, trong đó mỗi user là một vector u ∈ ℝᵏ, mỗi phim là một vector v ∈ ℝᵏ (k ~ vài chục đến vài trăm **latent factor**), và mức độ hợp nhau là **dot product** u·v. Huấn luyện = tối thiểu hóa Σ(r − u·v)² trên các ô *đã biết*, cộng regularization λ(‖u‖² + ‖v‖²) chống overfit (chương 21). Các trục tiềm ẩn không ai đặt tên trước — chúng *tự hiện ra* từ dữ liệu, họ hàng trực tiếp của SVD/PCA (chương 17).

**Thuật toán.** Hai đường tối ưu phổ biến: **SGD** — lặp qua từng rating, đẩy u và v theo gradient của sai số; hoặc **ALS (Alternating Least Squares)** — cố định V thì bài toán theo U là least squares có nghiệm đóng, giải xen kẽ hai phía; ALS song song hóa đẹp trên Spark nên được ưa ở quy mô lớn. Lịch sử đáng nhớ: **Netflix Prize** (2006–2009, giải thưởng 1 triệu đô cho ai cải thiện 10% RMSE so với hệ thống Cinematch) chính là sự kiện đưa matrix factorization thành chuẩn ngành — và kết cục của nó là một bài học trade-off được trích dẫn mãi: **ensemble vô địch chưa bao giờ được deploy đầy đủ**, vì độ phức tạp kỹ thuật của hàng trăm mô hình trộn nhau không đáng với phần chính xác tăng thêm. Độ chính xác là *một* cột trong bảng chi phí, không phải cả bảng. Hai bài toán con còn lại: **cold start** — user/phim mới chưa có dòng/cột dữ liệu, factorization bó tay → dùng content feature (thể loại, diễn viên) và **bandit** thăm dò-khai thác (chương 11); và **A/B testing mọi thay đổi** (chương 18): mọi thuật toán mới phải thắng trong thí nghiệm ngẫu nhiên có kiểm soát trên retention/giờ xem — với đủ cỡ mẫu và mức ý nghĩa thống kê — chứ không phải thắng trên slide. Netflix nổi tiếng là công ty "văn hóa A/B": cả artwork của phim cũng được thử nghiệm.

**Trade-off đã chọn.** Mô hình tuyến tính chiều thấp ↔ biểu đạt (dot product không nói được "thích phim này *chỉ khi* xem cùng gia đình" — các mô hình deep sau này thêm ngữ cảnh); offline accuracy ↔ online metric (RMSE đẹp không bảo đảm giữ chân người dùng — nên trọng tài cuối cùng luôn là A/B test); và exploitation ↔ exploration — luôn chiều khẩu vị cũ thì không bao giờ biết user có thể thích gì mới.

**Chương liên quan:** 11 (tín hiệu xác suất, bandit), 17 (ma trận, dot product, latent factor/SVD), 18 (A/B testing, ý nghĩa thống kê), 21 (tối ưu hóa có regularization).

---

## Case 10 — Blockchain Consensus: mua sự thật bằng xác suất

**Bài toán kinh doanh.** Một sổ cái tiền tệ không có ngân hàng trung ương: hàng chục nghìn node ẩn danh, không tin nhau, một số *chủ động gian lận*, phải thống nhất về thứ tự giao dịch — bài toán consensus (chương 25) trong điều kiện khắc nghiệt nhất: không biết danh tính, không đếm được "đa số node" (kẻ tấn công tạo triệu node ảo miễn phí — Sybil attack). Nakamoto (2008) đổi tiền tệ của phiếu bầu: **một CPU-cycle, một phiếu** — bỏ phiếu bằng thứ không giả mạo được: công sức tính toán.

**Mô hình toán.** *Proof of Work là một xổ số hash*: tìm nonce sao cho SHA-256(block) < target (chương 26). Vì output hash là "ngẫu nhiên đều" trên [0, 2²⁵⁶), mỗi lần thử là một phép thử Bernoulli với xác suất trúng p = target/2²⁵⁶ — cực nhỏ, và **không có cách nào thông minh hơn thử liên tục** (đó chính là phẩm chất của hash mật mã). Hàng tỷ tỷ phép thử độc lập tần suất cao → số block tìm được theo thời gian là **Poisson process** (chương 11): thời gian chờ giữa hai block là phân phối mũ, *memoryless* — đào 9 phút chưa trúng không hề "sắp trúng" hơn. Mạng giữ nhịp **10 phút/block** bằng **difficulty adjustment**: mỗi 2016 block, so thời gian thực tế với 2016×10 phút và co giãn target theo tỷ lệ — một vòng **feedback control** âm đúng nghĩa: hashrate toàn cầu tăng 10 lần, sau chu kỳ điều chỉnh nhịp vẫn về 10 phút. Con số 10 phút tự nó là trade-off: ngắn hơn → xác nhận nhanh nhưng nhiều fork ngẫu nhiên (block mới lan chưa khắp mạng thì block cạnh tranh đã ra đời — độ trễ lan truyền so với block time, bài học chương 25); dài hơn → ổn định nhưng chậm.

An toàn của người nhận tiền quy về một bài toán cổ điển: kẻ gian giữ tỷ lệ hashrate q (người lương thiện p = 1 − q) bí mật đào nhánh riêng chứa giao dịch đảo ngược. Hiệu số độ dài hai nhánh là một **random walk** (chương 11) thiên vị về phía p: mỗi block mới là một bước ±1 với xác suất p/q. Theo kết quả gambler's ruin, xác suất kẻ gian *đuổi kịp* từ vị trí kém z block là **(q/p)ᶻ** — **giảm theo hàm mũ** của z. Tính toán của Nakamoto (gộp thêm yếu tố Poisson cho tiến độ của kẻ gian trong lúc chờ) cho bảng đáng nhớ: q = 10% thì z = 5–6 confirmation đưa xác suất double-spend xuống dưới 0.1%. Đó là lý do sàn giao dịch chờ **6 confirmation** (~1 giờ) — con số ấy là một dòng trong bảng xác suất, không phải quy ước tùy hứng. Và khi q > 0.5, (q/p)ᶻ ≥ 1 với mọi z: random walk trôi *về phía* kẻ gian, hắn đuổi kịp chắc chắn — **51% attack** không phải lỗ hổng cài đặt mà là *biên giới toán học* của mô hình: an toàn của PoW là mệnh đề có điều kiện "chừng nào đa số hashrate còn lương thiện".

**Thuật toán.** Node theo luật **longest chain** (chính xác: chuỗi nhiều công sức tích lũy nhất): luôn đào nối tiếp nhánh dài nhất từng thấy; fork tự giải quyết vì hai nhánh bằng nhau sẽ sớm bị một block mới phá vỡ đối xứng và cả mạng hội tụ. Tính bất biến của lịch sử được gia cố bằng **hash chaining + Merkle tree** (chương 26): mỗi block chứa hash block trước và Merkle root của các giao dịch — sửa một giao dịch cũ đòi đào lại *toàn bộ* các block phía sau, tức đấu tay đôi với tổng hashrate của mạng.

**Trade-off đã chọn.** Nakamoto consensus mua **tính phi tập trung + chống Sybil** bằng ba cái giá khổng lồ: năng lượng (phiếu bầu phải *đắt* thì mới không giả được — lãng phí là tính năng, không phải bug); độ trễ (finality xác suất — không bao giờ chắc chắn 100%, chỉ "chắc chắn theo hàm mũ", so với finality tức thì của Raft trong môi trường có danh tính, chương 25); và throughput (~7 giao dịch/giây so với hàng chục nghìn của hệ tập trung). Proof of Stake là nỗ lực đổi cấu trúc chi phí — đặt cọc vốn thay vì đốt điện — với những trade-off mới của riêng nó. Không có bữa trưa miễn phí; chỉ có hóa đơn được viết bằng đơn vị khác.

**Chương liên quan:** 11 (Bernoulli, Poisson, random walk/gambler's ruin), 19 (số học, biểu diễn target/difficulty), 26 (hash mật mã, Merkle tree), 25 (consensus, so sánh với Raft).

---

## Ma trận hệ thống × khái niệm toán

Nhìn cả mười case trên một bảng, các hoa văn hiện ra: xác suất và graph có mặt gần như khắp nơi; mọi hệ thống đều đứng trên ít nhất ba chương.

| Hệ thống | Đếm/Tổ hợp (05) | Graph (08) | Xác suất (11) | Hashing (12) | DP (14) | Greedy (15) | LinAlg (17) | Thống kê (18) | Số học (19) | Entropy (20) | Tối ưu (21) | CTDL nâng cao (23/24) | Phân tán (25) | Crypto (26) |
|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| PostgreSQL Optimizer | ✓ | | ✓ | | ✓ | | | ✓ | | | ✓ | | | |
| Redis | | | ✓ | ✓ | | | | | | | | ✓ | | |
| Kafka | | | | ✓ | | | | | | ✓ | | | ✓ | |
| PageRank | | ✓ | ✓ | | | | ✓ | | | | | | | |
| Elasticsearch | | | ✓ | | | | ✓ | | | ✓ | | ✓ | | |
| Git | | ✓ | | ✓ | ✓ | | | | | ✓ | | | | ✓ |
| Kubernetes Scheduler | | ✓ | ✓ | | | ✓ | ✓ | | | | ✓ | | | |
| Google Maps | | ✓ | | | | | | | | | ✓ | | | |
| Netflix | | | ✓ | | | | ✓ | ✓ | | | ✓ | | | |
| Blockchain | | | ✓ | ✓ | | | | | ✓ | | | | ✓ | ✓ |

(Google Maps còn tựa vào chương 22 — Computational Geometry, cột duy nhất bảng này không đủ chỗ chứa. Bảng nào cũng là một phép nén có mất mát — chương 20 dạy ta điều đó.)

Ba quan sát đáng mang theo:

1. **Cột Xác suất dày đặc nhất.** Hệ thống càng lớn càng phải sống chung với sự không chắc chắn — và xử lý sự không chắc chắn một cách *định lượng* chính là nội dung của xác suất. Đây không phải ngẫu nhiên; đây là hệ quả của quy mô.
2. **Không hệ thống nào dùng một khái niệm đơn lẻ.** Kiến trúc sư giỏi không phải người biết một công cụ sâu nhất, mà là người *ghép* đúng ba bốn công cụ dưới ràng buộc kinh doanh cụ thể.
3. **Mọi ô ✓ đều đi kèm một trade-off có tên.** PostgreSQL đổi tối ưu lấy thời gian planning; Redis đổi chính xác lấy latency phẳng; Bitcoin đổi năng lượng lấy phi tập trung. Toán học không chọn hộ bạn — nó chỉ làm cho cái giá của từng lựa chọn *hiện rõ trước khi bạn trả*.

## Lời kết

Nếu phải nén 27 chương thành một thói quen duy nhất, thì là câu hỏi này — hãy hỏi nó mỗi khi bạn gặp một hệ thống mới, đọc một bài blog kiến trúc, hay ngồi trước một buổi system design:

> **"Mô hình toán ẩn dưới hệ thống này là gì?"**

Đằng sau mỗi con số cấu hình có một phân phối xác suất; đằng sau mỗi quyết định caching có một bài toán kỳ vọng; đằng sau mỗi giới hạn "không thể làm được" có một định lý. Công nghệ trong tài liệu này rồi sẽ cũ — Redis, Kafka, Kubernetes đều sẽ có kẻ kế nhiệm. Nhưng pigeonhole không có phiên bản 2.0, eigenvector không bị deprecated, và random walk sẽ vẫn mô tả đúng cuộc rượt đuổi giữa kẻ gian và người lương thiện trong bất kỳ hệ thống nào con người xây tiếp.

Học công nghệ, bạn theo kịp ngành trong vài năm. Học được lớp toán bên dưới, bạn đọc được mọi công nghệ *chưa ra đời*.

---

*Hết tài liệu. Nếu bạn quay lại chỉ một chương, hãy quay lại [04 — Mathematical Modeling](/series/math-for-engineers/level-1-mathematical-thinking/04-mathematical-modeling/) — vì mười case study trên đây, xét cho cùng, chỉ là mười lần lặp lại cùng một động tác: nhìn xuyên qua bài toán kinh doanh để thấy mô hình toán bên dưới.*
