+++
title = "Chương 03 – Consistency Models: \"Đọc thấy gì\" là một hợp đồng"
date = "2026-07-08T19:00:00+07:00"
draft = false
tags = ["backend", "distributed-systems"]
series = ["Distributed Systems"]
+++

## 1. Problem Statement

Ngay khi dữ liệu tồn tại ở **hơn một nơi** (replica, cache), xuất hiện câu hỏi không có ở hệ một máy: *đọc ở nơi này có thấy cái vừa ghi ở nơi kia không?* Consistency model là **hợp đồng** giữa hệ thống lưu trữ và ứng dụng: hệ thống cam kết trả lời câu hỏi đó theo quy tắc nào.

Nếu không định nghĩa hợp đồng rõ ràng: user đăng comment, refresh trang, comment biến mất (đọc trúng replica trễ) → user đăng lại → duplicate; hệ thống kiểm tra số dư trên replica trễ → cho rút quá tiền; hai người cùng đặt phòng cuối cùng → double booking. Đây không phải bug hiếm — đây là **hành vi mặc định** khi bạn dùng replication mà không nghĩ về consistency model.

Giải pháp cũ (một node duy nhất, không cache) cho consistency hoàn hảo miễn phí — nhưng chúng ta đã rời bỏ nó vì lý do ở chương 01. Giờ phải trả giá.

## 2. Tại sao nhiều model tồn tại — thay vì một model "đúng"

Vì **consistency mạnh hơn = đắt hơn**, và độ đắt là vật lý, không phải do code tồi:

- Muốn mọi replica nhất trí trước khi trả lời → phải **chờ** round-trip qua mạng (quorum/consensus) → latency tăng theo khoảng cách replica xa nhất.
- Muốn trả lời ngay từ replica gần nhất → không kịp phối hợp → có thể trả **dữ liệu cũ**.

Không có model nào đúng cho mọi nghiệp vụ: số dư tài khoản cần strong consistency (đọc sai = mất tiền), số like của bài viết thì eventual là quá đủ (đọc sai 5 giây = không ai chết). Nghệ thuật là **chọn model yếu nhất mà nghiệp vụ chịu được** — vì mỗi nấc mạnh hơn trả bằng latency, availability và throughput.

## 3. Bản chất — phổ các consistency model

### 3.1. Sắp theo độ mạnh

```
MẠNH  Linearizability (Strong Consistency)
 │        │  hệ thống hành xử như CHỈ CÓ MỘT bản dữ liệu;
 │        │  mọi thao tác có vẻ xảy ra tức thời tại một điểm
 │        ▼
 │    Sequential Consistency
 │        │  mọi process thấy CÙNG một thứ tự, nhưng thứ tự đó
 │        │  không cần khớp thời gian thực
 │        ▼
 │    Causal Consistency
 │        │  chỉ các sự kiện có quan hệ NHÂN QUẢ mới cần đúng thứ tự;
 │        │  sự kiện độc lập nhau ai thấy trước thấy sau tùy ý
 │        ▼
 │    Session Guarantees (Read Your Writes, Monotonic Reads...)
 │        │  cam kết chỉ trong phạm vi MỘT phiên người dùng
 │        ▼
YẾU   Eventual Consistency
          "nếu ngừng ghi, cuối cùng mọi replica sẽ giống nhau"
          — không cam kết gì về ĐỌC THẤY GÌ trong lúc đó
```

### 3.2. Linearizability — vì sao đắt

Định nghĩa first-principles: sau khi một write **hoàn thành** (trả về cho client), *mọi* read tiếp theo — từ bất kỳ client nào, đến bất kỳ replica nào — phải thấy giá trị đó hoặc mới hơn. Hệ thống hành xử **như thể chỉ có một bản dữ liệu**.

Vì sao đắt: để một read ở replica R chắc chắn không trả giá trị cũ, R phải *xác nhận với đa số* rằng không có write mới hơn mà nó chưa biết — tức là **read cũng tốn round-trip**, không chỉ write. Trong Raft, linearizable read cần leader xác nhận mình còn là leader (heartbeat quorum hoặc ReadIndex). Không có cách nào rẻ hơn về nguyên tắc: nếu R trả lời một mình, nó không thể biết mình có bị partition khỏi phần còn lại (nơi write mới vừa xảy ra) hay không. **Đây chính là nội dung của CAP** (chương 04).

Khi nào bắt buộc: distributed lock (lock mà đọc thấy cũ = hai người cùng giữ lock), leader election, unique constraint (username), số dư/tồn kho tại thời điểm quyết định.

### 3.3. Eventual Consistency — yếu hơn bạn tưởng

"Eventually consistent" chỉ hứa: *nếu ngừng ghi*, các replica *cuối cùng* hội tụ. Không hứa bao lâu ("eventually" có thể là 10ms hay 10 phút khi có sự cố replication), không hứa gì về thứ tự đọc. Hai read liên tiếp có thể **đi lùi thời gian** (đọc replica mới rồi đọc replica cũ). Ứng dụng phải tự chịu: hiển thị dữ liệu cũ, xử lý xung đột khi hai replica nhận write đồng thời (chương 05).

Đổi lại: read/write ở replica gần nhất, không chờ ai — latency thấp nhất, availability cao nhất có thể. DynamoDB, Cassandra mặc định chọn phía này.

### 3.4. Causal Consistency — điểm cân bằng bị đánh giá thấp

Yêu cầu duy nhất: nếu sự kiện B *phụ thuộc nhân quả* vào A (B đọc thấy A rồi mới xảy ra), thì ai thấy B phải thấy A trước. Ví dụ kinh điển: câu trả lời không được xuất hiện trước câu hỏi trong mắt bất kỳ ai. Hai write không liên quan nhau thì thứ tự tùy replica — nhờ vậy không cần phối hợp toàn cục, **vẫn available khi partition** (điều linearizability không làm được). Theo dõi nhân quả bằng vector clock/version vector (chương 12). MongoDB (causal sessions), CosmosDB có hỗ trợ trực tiếp.

### 3.5. Session Guarantees — thứ ứng dụng thật cần nhất

Phần lớn "bug consistency" mà người dùng *cảm nhận* được sửa bằng 4 cam kết phạm vi phiên, rẻ hơn nhiều so với strong consistency toàn cục:

| Guarantee | Cam kết | Ví dụ vi phạm | Cách đạt rẻ |
|---|---|---|---|
| **Read Your Writes** | Đọc của tôi thấy write của tôi | Đăng comment xong refresh không thấy | Pin phiên vào leader sau khi write; hoặc đọc replica có LSN ≥ LSN write của mình |
| **Monotonic Reads** | Đọc sau không cũ hơn đọc trước | Thấy tin nhắn rồi refresh lại mất | Pin phiên vào một replica; hoặc client giữ version cuối đã thấy |
| **Monotonic Writes** | Write của tôi áp dụng theo thứ tự tôi ghi | Sửa profile 2 lần, bản cũ đè bản mới | Route write của một phiên qua một đường |
| **Writes Follow Reads** | Write của tôi xếp sau cái tôi đã đọc | Trả lời comment nhưng replica khác thấy trả lời trước comment | Gắn causal token vào write |

Kỹ thuật production phổ biến: client giữ **hybrid logical clock / LSN token** từ lần write cuối, gửi kèm mọi read; replica chỉ trả lời khi đã replay tới token đó, không thì chờ hoặc chuyển leader.

## 4. Quorum — cơ chế bên trong

### 4.1. Công thức và trực giác

N replica, write chờ W bản ack, read hỏi R bản:

```
W + R > N  ⟹  tập read và tập write GIAO NHAU ít nhất 1 node
              ⟹ read luôn chạm ít nhất 1 bản có giá trị mới nhất

N=3, W=2, R=2:
  Write ──▶ [n1 ✓][n2 ✓][n3 ✗ chậm]
  Read  ──▶      [n2 ✓][n3 ✓]  → n2 có bản mới (chọn theo version cao nhất)
```

Núm vặn trade-off tường minh:

| Cấu hình (N=3) | Đặc tính | Dùng khi |
|---|---|---|
| W=3, R=1 | Read nhanh nhất; write chậm, 1 node chết là không ghi được | Read-heavy, dữ liệu quan trọng |
| W=1, R=3 | Write nhanh nhất; read đắt | Write-heavy, ít đọc |
| W=2, R=2 | Cân bằng, chịu 1 node chết cả đọc lẫn ghi | Mặc định phổ biến |
| W=1, R=1 | Nhanh nhất, **không giao nhau** → eventual | Metrics, log, đếm like |

### 4.2. Vì sao quorum W+R>N vẫn CHƯA phải linearizability

Sự thật hay bị bỏ qua: quorum kiểu Dynamo (leaderless) với W+R>N vẫn có thể vi phạm linearizability, vì write đến các replica **không nguyên tử** — trong lúc write đang lan đến n1 rồi n2, một read R=2 có thể thấy giá trị mới, read *sau đó* của client khác lại thấy giá trị cũ (chạm cặp replica khác). Sửa cần read-repair đồng bộ hoặc consensus thật sự. Bài học: **quorum là điều kiện cần, không đủ**; đừng nói "tôi dùng QUORUM nên strong consistency" — Cassandra QUORUM cho bạn cam kết mạnh *hơn* nhưng vẫn có góc khuất (đặc biệt khi kết hợp timestamp-based conflict resolution với clock skew — chương 12).

### 4.3. Sloppy Quorum & Hinted Handoff

Dynamo-style: khi node trong "home set" chết, write tạm sang node khác kèm "hint", sau này trả về chỗ cũ. Tăng availability cho write (hầu như luôn ghi được), nhưng phá vỡ cả cái giao nhau của quorum → thêm một nguồn đọc-cũ nữa. Availability mua bằng consistency, đúng chủ đề xuyên suốt.

## 5. Trade-off

- **Consistency vs Latency** (còn quan trọng hơn vs Availability trong vận hành hằng ngày — xem PACELC, chương 04): mỗi nấc mạnh hơn thêm ít nhất một round-trip. Cùng region: +1–2ms, chấp nhận được. Đa region: +50–300ms — strong consistency xuyên region là quyết định phải cân nhắc bằng tiền và UX.
- **Consistency vs Throughput**: strong consistency thường đi qua leader/consensus → một điểm tuần tự hóa → trần throughput (USL, chương 01).
- **Consistency vs Độ phức tạp ứng dụng**: model yếu đẩy độ phức tạp *lên tầng ứng dụng* — code phải xử lý đọc-cũ, xung đột, out-of-order. Chi phí này vô hình lúc thiết kế, rất đắt lúc bảo trì. Một hệ eventual "rẻ" ở tầng hạ tầng có thể "đắt" tổng thể vì mọi feature phải nghĩ về conflict.
- **Granularity**: không chọn một model cho cả hệ thống — chọn **per-operation**. Cùng một app: ghi đơn hàng strong, đếm view eventual, timeline causal. Các DB tốt cho phép chỉnh per-request (Cassandra consistency level, DynamoDB `ConsistentRead`, MongoDB read/write concern).

## 6. Production Considerations

- **Đo replication lag như một SLO**: p99 lag của mọi replica, alert khi vượt ngưỡng nghiệp vụ chịu được. Lag tăng đột biến thường báo trước sự cố lớn (replica sắp bị loại, failover sắp mất dữ liệu — chương 05).
- **Kiểm thử với lag nhân tạo**: staging có lag ~0 nên bug consistency không bao giờ lộ trước production. Tiêm lag giả (proxy delay) vào môi trường test; chạy Jepsen-style test cho hệ tự xây.
- **Fallback có kiểm soát**: đọc strong thất bại (leader unavailable) → fallback đọc eventual + đánh dấu dữ liệu "có thể cũ" cho tầng trên quyết định, thay vì fail toàn bộ. Nhưng phải là quyết định nghiệp vụ tường minh, không phải mặc định âm thầm.
- **Document hợp đồng**: mỗi API ghi rõ cam kết consistency ("sau khi POST trả 200, GET có thể trễ tối đa X trong điều kiện bình thường"). Không document = mỗi team tự giả định = bug tích hợp.

## 7. Best Practices

1. **Phân loại dữ liệu theo nhu cầu consistency trước khi chọn công nghệ** — làm bảng: entity × thao tác × model tối thiểu × hậu quả nếu vi phạm.
2. **Mặc định session guarantees cho mọi thứ user-facing** — Read Your Writes + Monotonic Reads xóa 90% bug "user thấy kỳ kỳ" với chi phí thấp.
3. **Strong consistency cho quyết định, eventual cho hiển thị**: kiểm tra tồn kho lúc *checkout* phải strong; badge "còn 3 sản phẩm" trên trang list thì eventual thoải mái.
4. **Đừng tự xây conflict resolution nếu tránh được** — dồn write cùng entity về một leader/partition để có thứ tự tự nhiên.
5. **Version mọi thứ**: optimistic concurrency (compare-and-set với version) rẻ và chặn được lost update phổ biến nhất.

## 8. Anti-patterns

- **Read-modify-write không version trên storage eventual**: đọc số dư → cộng → ghi đè. Hai request song song = lost update. Phải dùng conditional write / atomic increment.
- **Kiểm tra rồi hành động (check-then-act) qua hai request**: "username chưa tồn tại" → "tạo username" — giữa hai bước, người khác tạo mất. Cần unique constraint nguyên tử tại storage.
- **Trộn nguồn đọc trong một luồng**: nửa đầu request đọc leader, nửa sau đọc replica → dữ liệu tự mâu thuẫn trong cùng một response.
- **Coi cache là nguồn strong consistency**: cache về bản chất là replica eventual (chương 10).
- **"Consistency = ACID C"**: chữ C trong ACID (ràng buộc toàn vẹn) khác hoàn toàn C trong CAP (linearizability). Dùng lẫn là nguồn hiểu lầm bất tận trong design review.

## 9. Khi nào KHÔNG cần nghĩ về vấn đề này

Một node PostgreSQL, không replica, không cache: linearizability miễn phí — thêm lý do yêu quý kiến trúc đơn giản khi tải cho phép. Read từ chính leader (không qua replica): các session guarantee tự có. Dữ liệu bất biến (immutable — ảnh đã upload, event log): không có write thứ hai thì không có xung đột — thiết kế dữ liệu immutable là cách *né* bài toán consistency thanh lịch nhất.

## 10. Troubleshooting

| Triệu chứng | Nguyên nhân khả dĩ | Hướng xử lý |
|---|---|---|
| User không thấy dữ liệu vừa tạo | Read đến replica trễ | Read-your-writes: pin leader/LSN token |
| Dữ liệu "nhấp nháy" cũ–mới–cũ | Đọc luân phiên replica lệch nhau | Monotonic reads: sticky replica / version token |
| Lost update (ghi đè lẫn nhau) | Read-modify-write không version | CAS, atomic op, hoặc dồn về một writer |
| Duplicate entity (2 user cùng username) | Check-then-act không nguyên tử | Unique constraint tại storage, xử lý lỗi trùng |
| Cassandra: đọc thấy dữ liệu "hồi sinh" sau khi xóa | Tombstone hết hạn trước khi repair chạy | Chạy repair đều đặn < gc_grace_seconds |
| Xung đột dữ liệu đa master | Hai leader cùng nhận write một entity | Xem chương 05: conflict resolution, hoặc bỏ multi-leader |

---

## Tổng kết chương

### Những điều bắt buộc phải nhớ
1. Consistency model là **hợp đồng đọc-thấy-gì**; không chọn tường minh nghĩa là bạn đã chọn eventual một cách vô thức.
2. Mạnh hơn = ít nhất một round-trip đắt hơn — chi phí là vật lý, không tối ưu code mà hết được.
3. Chọn **per-operation**, không per-system; model yếu nhất mà nghiệp vụ chịu được.
4. Session guarantees (Read Your Writes, Monotonic Reads) giải quyết phần lớn trải nghiệm người dùng với giá rẻ.
5. W+R>N là cần, không đủ, cho linearizability.
6. Model yếu không xóa độ phức tạp — nó **chuyển** độ phức tạp từ hạ tầng lên code ứng dụng.

### Hiểu lầm phổ biến
- "Eventual consistency = vài ms là xong" — không có cam kết thời gian; lúc sự cố có thể là phút/giờ, và thiết kế phải sống được với điều đó.
- "Quorum = strong consistency" — xem 4.2.
- "Strong consistency là lựa chọn an toàn mặc định" — an toàn về dữ liệu, nhưng trả bằng latency/availability; hệ chậm và hay từ chối phục vụ cũng là hệ hỏng dưới mắt người dùng.
- "C trong ACID = C trong CAP" — hai khái niệm khác nhau hoàn toàn.

### Câu hỏi tự kiểm tra
1. Với app ngân hàng: xem số dư, chuyển tiền, xem lịch sử 30 ngày — chọn model cho từng thao tác và giải thích bằng hậu quả-nếu-sai.
2. Cassandra N=3, W=QUORUM, R=ONE: cam kết gì? Kịch bản cụ thể nào làm user đọc thấy dữ liệu cũ?
3. Thiết kế Read Your Writes cho hệ có 1 leader + 5 replica mà không dồn hết read về leader.
4. Vì sao dữ liệu immutable né được bài toán consistency? Áp dụng vào thiết kế bảng "lịch sử giao dịch" thế nào?

### Tài liệu kinh điển nên đọc
- **"Linearizability: A Correctness Condition for Concurrent Objects" (Herlihy & Wing, 1990)** — định nghĩa gốc, chuẩn mực của "strong consistency".
- **"Session Guarantees for Weakly Consistent Replicated Data" (Terry et al., 1994)** — nguồn của Read Your Writes/Monotonic Reads; 30 năm sau vẫn là thứ bạn cần hằng ngày.
- **"Dynamo: Amazon's Highly Available Key-value Store" (2007)** — tuyên ngôn của phe eventual: sloppy quorum, hinted handoff, vector clock trong production.
- **Jepsen analyses (jepsen.io)** — kiểm chứng độc lập cam kết consistency của các DB thật; đọc để thấy khoảng cách giữa marketing và thực tế.
- **"Highly Available Transactions: Virtues and Limitations" (Bailis et al., 2013)** — bản đồ model nào đạt được mà không hy sinh availability — chỗ đứng của causal consistency.
