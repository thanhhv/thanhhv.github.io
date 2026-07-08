+++
title = "Chương 05 – Replication: Nhiều bản sao, một sự thật (hoặc không)"
date = "2026-07-08T21:00:00+07:00"
draft = false
tags = ["backend", "distributed-systems"]
series = ["Distributed Systems"]
+++

## 1. Problem Statement

Replication = giữ cùng dữ liệu trên nhiều node. Ba lý do buộc phải làm: **sống sót khi node chết** (durability + availability), **scale read** (dàn read ra nhiều replica), **giảm latency theo địa lý** (bản sao gần user). Nếu không replicate: node chết = mất dữ liệu + downtime đến khi restore từ backup (RPO = từ lần backup cuối, RTO = hàng giờ). Backup không thay thế replication — backup cứu bạn khỏi *mất vĩnh viễn*, replication cứu bạn khỏi *ngừng phục vụ*.

Vấn đề mới sinh ra: dữ liệu đổi liên tục, các bản sao phải được cập nhật — **bằng cách nào, theo thứ tự nào, và ai được nhận write?** Ba câu trả lời khác nhau tạo thành ba kiến trúc của chương này.

## 2. Leader-Follower (Single-Leader) — kiến trúc phổ biến nhất

### 2.1. Cơ chế bên trong

```
              write                    replication log
Client ────────────▶ [LEADER] ──┬────────▶ [Follower 1]
                        │       └────────▶ [Follower 2]
Client ◀──── read ──────┴──── read ────────┘ (tùy chọn)
```

Mọi write đi qua **một** leader → leader quyết định **thứ tự toàn cục** của các write, ghi vào replication log (WAL của PostgreSQL, binlog của MySQL, oplog của MongoDB), follower replay log **theo đúng thứ tự đó**. Đây là lý do sâu xa single-leader phổ biến: **thứ tự write được giải quyết miễn phí bằng kiến trúc** — không cần vector clock, không cần conflict resolution. Toàn bộ độ khó dồn vào một chỗ: *chuyện gì xảy ra khi leader chết* (→ chương 07).

### 2.2. Sync vs Async — quyết định quan trọng nhất

| | Synchronous | Asynchronous | Semi-sync (thực dụng) |
|---|---|---|---|
| Leader chờ gì trước khi ack client | Follower xác nhận đã nhận | Không chờ ai | Chờ **1 trong N** follower |
| Mất dữ liệu khi leader chết | Không | Có (phần lag) | Không (bản sao ở follower sync) |
| Latency write | + RTT đến follower chậm nhất | Thấp nhất | + RTT đến follower nhanh nhất |
| Follower chết thì write | Treo (nếu chờ đúng follower đó) | Không ảnh hưởng | Chuyển chờ follower khác |
| Ví dụ | PG `synchronous_commit=remote_apply` | MySQL mặc định, Redis | PG `ANY 1 (s1,s2)`, MySQL semi-sync |

Sync thuần với *mọi* follower gần như không dùng được (một follower GC pause là toàn hệ treo write). Async thuần là mặc định của đa số — và là nguồn của sự cố mất-dữ-liệu-khi-failover kinh điển. **Semi-sync là điểm cân bằng production**: durability của sync, độ bền vận hành của async.

### 2.3. Replication Lag và hệ quả

Follower async trễ hơn leader: bình thường mili giây, khi tải cao/vacuum/network hiccup có thể phút. Mọi vấn đề của chương 03 (read-your-writes, monotonic reads) sinh ra từ đây. Lag còn quyết định **RPO khi failover**: promote follower đang lag 30 giây = mất 30 giây write đã-ack. Đo lag bằng đơn vị *thời gian* lẫn *byte WAL*, alert theo cả hai.

### 2.4. Failover — nơi mọi thứ đổ vỡ

Quy trình: phát hiện leader chết (timeout — lại là phỏng đoán!) → chọn follower mới nhất → promote → trỏ client sang. Mỗi bước đều có bẫy:

- **Phát hiện nhầm** (leader chỉ GC pause): promote xong leader cũ tỉnh dậy → **hai leader** (split brain) → hai nhánh dữ liệu phân kỳ. Phòng: fencing (STONITH — bắn chết leader cũ trước khi promote; epoch/fencing token để storage từ chối leader cũ — chương 07).
- **Promote follower trễ**: mất write. GitHub 2018: partition 43 giây, failover cross-DC, dữ liệu hai DC phân kỳ → 24 giờ khắc phục.
- **Client cache DNS/connection cũ**: vẫn ghi vào leader cũ.

Bài học: failover tự động là bắt buộc ở quy mô lớn (MTTR!), nhưng phải được **thiết kế và diễn tập** như một tính năng hạng nhất, không phải script chắp vá.

## 3. Multi-Leader

Nhiều node cùng nhận write (thường mỗi DC một leader), replicate chéo async.

**Dùng khi**: multi-datacenter (write latency cục bộ tại mỗi DC), ứng dụng offline-first (mỗi thiết bị là một "leader"), collaborative editing.

**Cái giá — write conflict là chuyện thường trực**: hai leader nhận write cùng entity trong cùng cửa sổ replication → xung đột. Các chiến lược:

| Chiến lược | Cơ chế | Rủi ro |
|---|---|---|
| Last-Write-Wins (LWW) | Timestamp cao hơn thắng | **Mất write âm thầm** + phụ thuộc clock sync (ch. 12). Cassandra chọn cách này — chấp nhận được cho dữ liệu kiểu "trạng thái mới nhất", nguy hiểm cho counter/số dư |
| Merge tự động (CRDT) | Kiểu dữ liệu tự hội tụ về mặt toán học (G-Counter, OR-Set) | Chỉ áp dụng cho cấu trúc phù hợp; ngữ nghĩa merge phải khớp nghiệp vụ (Riak, Redis CRDB) |
| App-level resolution | Lưu cả hai version (sibling), app/user quyết | Đẩy độ phức tạp lên app (Dynamo gốc: hợp nhất giỏ hàng — item đã xóa "hồi sinh" là hệ quả có thật) |
| Tránh conflict | Route write của mỗi entity về **một** leader cố định (home DC theo user) | Mất lợi ích latency khi user di chuyển; thực ra là single-leader-per-entity — và thường là **câu trả lời đúng** |

Khuyến nghị: multi-leader là công cụ chuyên dụng. Nếu có thể partition theo entity và gán home leader — làm vậy, đơn giản hơn nhiều.

## 4. Leaderless (Dynamo-style)

Không leader; client (hoặc coordinator) ghi thẳng N replica, quorum W/R (chương 03), kèm bộ cơ chế chống-phân-kỳ:

- **Read repair**: đọc thấy replica lệch version → ghi bản mới cho replica cũ ngay trên đường đọc.
- **Anti-entropy**: tiến trình nền so sánh Merkle tree giữa replica, đồng bộ phần lệch — bắt các key ít được đọc.
- **Hinted handoff / sloppy quorum**: node home chết → ghi tạm node khác kèm hint (chương 03).

**Được**: không failover (không leader để chết!) — write availability rất cao, mọi node ngang hàng, vận hành đơn giản về topology. **Mất**: không có thứ tự write toàn cục → conflict là bản chất (version vector/LWW), transaction đa key gần như không, và các góc khuất linearizability (ch. 03 §4.2). Cassandra, ScyllaDB, Riak, DynamoDB (bên trong) thuộc họ này.

## 5. So sánh & Trade-off tổng hợp

| Tiêu chí | Single-Leader | Multi-Leader | Leaderless |
|---|---|---|---|
| Thứ tự write | Toàn cục, miễn phí | Không — conflict resolution | Không — version vector |
| Write availability | Gián đoạn khi failover | Cao (mỗi DC tự ghi) | Cao nhất |
| Write latency đa DC | Cao (về leader) | Thấp (leader cục bộ) | Thấp (quorum cục bộ nếu topology cho phép) |
| Consistency đạt được dễ | Mạnh (đọc leader) | Yếu | Tunable, trần thấp hơn |
| Transaction | Dễ nhất | Khó | Rất khó |
| Vận hành | Failover là điểm đau | Conflict là điểm đau | Repair/tombstone là điểm đau |
| Đại diện | PostgreSQL, MySQL, MongoDB, Kafka (per-partition) | Aurora Global (write forwarding), CouchDB, CRDT stores | Cassandra, DynamoDB, Riak |

Trade-off gốc cần nhớ: **replication càng "dân chủ" (nhiều nơi nhận write) thì availability càng cao và consistency càng đắt** — vì thứ tự write toàn cục là thứ chỉ có được khi write đi qua một điểm (hoặc qua consensus, tức là trả latency).

Ghi chú kiến trúc hiện đại: **Kafka và các hệ consensus-based (etcd, CockroachDB range) là single-leader per partition/range** — kết hợp thứ-tự-miễn-phí của single leader với scale-out của partitioning. Đây là pattern thống trị hiện nay: đừng chọn "một leader cho cả hệ thống", hãy chọn "một leader cho mỗi mảnh dữ liệu".

## 6. Production Considerations

- **Giám sát bắt buộc**: replication lag (giây + byte) per-replica; trạng thái slot/binlog position; số conflict/giây (multi-leader); tiến độ repair và tuổi tombstone (leaderless).
- **Capacity**: replica không miễn phí — mỗi write nhân N lần disk IO + network. Cross-AZ replication traffic là dòng tiền thật trên cloud. Read replica chỉ scale *read*; write vẫn một leader (nhắc lại từ chương 01 vì lỗi này quá phổ biến).
- **Backup vẫn bắt buộc dù có N replica**: `DELETE FROM users` replicate sang mọi replica trong mili giây. Replication sao chép cả sai lầm. Cần backup + PITR (point-in-time recovery) và **diễn tập restore định kỳ** — backup chưa restore thử là backup trên niềm tin.
- **Rolling upgrade**: nâng follower trước → failover có chủ đích → nâng leader cũ. Cần client tự reconnect tử tế.
- **Diễn tập failover hàng quý** trong giờ hành chính. Failover chưa diễn tập = failover sẽ thất bại lúc 3h sáng.

## 7. Best Practices

1. Semi-sync (chờ 1 trong N replica) làm mặc định cho dữ liệu quan trọng.
2. Failover tự động + fencing tử tế (epoch token), hoặc failover thủ công có runbook — đừng chọn "tự động nhưng ngây thơ".
3. Read replica: route theo yêu cầu consistency (chương 03), không round-robin mù.
4. Multi-leader: tránh conflict bằng home-leader-per-entity trước khi nghĩ đến conflict resolution.
5. Leaderless: chạy repair đều đặn, hiểu gc_grace_seconds trước khi chỉnh.
6. Kiểm thử failure: giết leader trong load test và đo RPO/RTO thật.

## 8. Anti-patterns

- **Failover tự động ngây thơ** (health check 3 lần fail → promote, không fencing): sản xuất split brain công nghiệp.
- **Coi read replica là node "miễn phí"**: query analytics nặng trên replica làm lag tăng → ảnh hưởng cả failover RPO.
- **Multi-leader "cho oai"** khi chỉ có một DC: nhận toàn bộ chi phí conflict, không nhận lợi ích nào.
- **LWW cho dữ liệu cộng dồn** (counter, số dư): hai increment đồng thời → một cái biến mất, không dấu vết.
- **Chạy 2 node consensus/quorum**: chết 1 là tê liệt (quorum 2/2) — tệ hơn 1 node.
- **Không đo lag, chỉ đo "replica sống"**: replica sống mà lag 2 giờ thì gần như đã chết, nhưng dashboard vẫn xanh.

## 9. Khi nào KHÔNG cần replication (hoặc cần tối thiểu)

Dữ liệu tái tạo được từ nguồn khác (cache, index — rebuild được): chỉ cần capacity, không cần durability replication. Batch/analytics nội bộ chịu được downtime giờ: 1 node + backup tốt là đủ. Dev/staging: replication topology giống production chỉ khi bạn test failover — còn lại là lãng phí.

## 10. Troubleshooting

| Triệu chứng | Nguyên nhân khả dĩ | Xử lý |
|---|---|---|
| Lag tăng không hồi | Follower yếu hơn leader; long transaction chặn replay; network bão hòa | Nâng follower; kill long query trên replica; QoS băng thông replication |
| Mất dữ liệu sau failover | Async + promote replica trễ | Semi-sync; hoặc chấp nhận RPO tường minh + reconciliation từ WAL leader cũ |
| Split brain | Failover không fencing | Epoch/fencing token; STONITH; sửa quy trình trước khi sửa dữ liệu |
| Write chậm đột ngột (sync setup) | Follower sync chậm/chết, leader chờ | Semi-sync ANY; giám sát follower như giám sát leader |
| Dữ liệu "hồi sinh" (Cassandra) | Tombstone bị GC trước khi repair phủ hết replica | repair interval < gc_grace_seconds |
| Hai DC phân kỳ (multi-leader) | Conflict resolution không phủ entity nào đó | Kiểm kê bằng checksum/Merkle; định nghĩa lại chiến lược merge |

---

## Tổng kết chương

### Những điều bắt buộc phải nhớ
1. Single-leader mua **thứ tự write toàn cục** bằng một điểm nghẽn write và bài toán failover; multi-leader/leaderless mua availability bằng **conflict**.
2. Async replication = cửa sổ mất dữ liệu khi failover; semi-sync là cân bằng thực dụng.
3. Failover là tính năng phải thiết kế + diễn tập; split brain sinh ra từ failover ngây thơ, phòng bằng fencing.
4. Replication không thay backup — nó replicate cả lệnh DELETE của bạn.
5. Pattern hiện đại: leader-per-partition (Kafka, CockroachDB) — thứ tự cục bộ + scale toàn cục.
6. LWW = chấp nhận mất write âm thầm; chỉ dùng cho dữ liệu kiểu "trạng thái mới nhất".

### Hiểu lầm phổ biến
- "Có 3 replica là an toàn" — 3 replica async cùng nhận lệnh xóa nhầm trong 5ms.
- "Thêm replica tăng khả năng ghi" — không, chỉ read (và làm write đắt hơn).
- "Failover tự động luôn tốt hơn thủ công" — tự động *sai* tệ hơn thủ công chậm; đầu tư vào fencing trước khi đầu tư vào tốc độ.
- "Leaderless nghĩa là không có consistency" — tunable qua W/R; nhưng trần thấp hơn single-leader.

### Câu hỏi tự kiểm tra
1. PostgreSQL async, lag p99 = 8 giây. Leader chết. RPO thực tế? Nghiệp vụ nào của bạn chịu được, nghiệp vụ nào không — và bạn làm gì với nhóm sau?
2. Thiết kế fencing cho failover Redis: leader cũ tỉnh dậy sau GC pause 40 giây thì chuyện gì xảy ra, từng bước?
3. Hệ multi-DC của bạn cần write latency thấp ở cả Singapore và Frankfurt cho user *của từng vùng*. Multi-leader hay home-leader-per-user? Vì sao?
4. Vì sao Kafka chọn leader-per-partition thay vì leaderless quorum kiểu Cassandra? (gợi ý: ordering là tính năng cốt lõi của log.)

### Tài liệu kinh điển nên đọc
- **"Dynamo" (Amazon, 2007)** — leaderless hoàn chỉnh: sloppy quorum, hinted handoff, Merkle anti-entropy, vector clock.
- **"Chain Replication" (van Renesse & Schneider, 2004)** — biến thể ít người biết nhưng nền của nhiều hệ storage lớn (đọc để mở rộng không gian thiết kế).
- **Kleppmann, "Designing Data-Intensive Applications", chương 5** — trình bày replication hay nhất từng được in.
- **GitHub post-mortem Oct 2018** — case study thật về failover cross-DC + split brain; đọc kỹ timeline.
- **"CRDTs: Consistency without Concurrency Control" (Shapiro et al.)** — nền toán của merge tự động.
