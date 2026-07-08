+++
title = "Chương 07 – Consensus: Làm sao nhiều máy nhất trí một điều"
date = "2026-07-08T23:00:00+07:00"
draft = false
tags = ["backend", "distributed-systems"]
series = ["Distributed Systems"]
+++

## 1. Problem Statement

Nhiều bài toán ở các chương trước đều quy về một câu hỏi: **làm sao một nhóm node, trên mạng không đáng tin cậy, nhất trí về MỘT giá trị** — ai là leader (chương 05 failover), thứ tự các write trong log, bản đồ partition→node (chương 06), ai giữ lock. 

Nếu giải sai: **split brain** — hai node cùng tin mình là leader, cùng nhận write → hai nhánh sự thật phân kỳ, và không có cách nào hợp nhất tự động dữ liệu ledger đã phân kỳ. Đây là failure mode đắt nhất của hệ phân tán. Giải pháp cũ — "để con người quyết" — có MTTR hàng giờ; "node nào nhanh hơn thì thắng" — chính là split brain.

Điều làm bài toán khó *về nguyên tắc*: **FLP Impossibility (1985)** — trong hệ bất đồng bộ thuần túy (không giới hạn trên độ trễ message), không tồn tại thuật toán consensus tất định nào *đảm bảo* kết thúc, dù chỉ một node có thể chết. Nguyên nhân sâu: không thể phân biệt node chết với node chậm (chương 01). Các thuật toán thật (Paxos, Raft) "lách" FLP bằng cách hy sinh **liveness** trong thời gian xấu, không bao giờ hy sinh **safety**: có thể tạm không bầu được leader (hệ đứng), nhưng không bao giờ có hai quyết định mâu thuẫn.

## 2. Failure Detection — nền móng phỏng đoán

Mọi consensus bắt đầu từ việc đoán ai còn sống. Vì timeout không phân biệt chết/chậm, failure detector về bản chất là **đánh đổi giữa phát hiện nhanh (timeout ngắn → nhiều false positive → failover thừa, flapping) và chính xác (timeout dài → MTTR cao)**.

Cơ chế thực tế:
- **Heartbeat + timeout** (đa số hệ thống): tinh chỉnh timeout theo p99 latency mạng + GC pause thực tế. Timeout ngắn hơn GC pause của chính node = tự bắn vào chân.
- **Phi Accrual detector** (Cassandra, Akka): thay câu trả lời nhị phân bằng *mức nghi ngờ* liên tục dựa trên phân phối inter-arrival của heartbeat — thích nghi khi mạng chậm dần.
- **Gossip / SWIM** (Cassandra, Consul, memberlist): node kiểm tra lẫn nhau ngẫu nhiên, lan truyền kết quả — scale O(n) message thay vì all-to-all; kèm indirect probe để giảm false positive do đường mạng cục bộ.
- **Lease**: node giữ vai trò (leader, lock) chỉ trong khoảng thời gian có hạn, phải gia hạn — biến "phát hiện chết" thành "hết hạn tự nhiên", nhưng phụ thuộc giả định về clock (chương 12).

## 3. Bản chất Consensus & Quorum

### 3.1. Consensus phải đảm bảo

- **Agreement**: mọi node quyết định cùng một giá trị (safety).
- **Validity**: giá trị được quyết phải do ai đó đề xuất (safety).
- **Termination**: cuối cùng phải quyết được (liveness — thứ FLP nói không thể *đảm bảo*).

### 3.2. Vì sao quorum đa số là trái tim của mọi thuật toán

**Hai quorum đa số của cùng một cụm LUÔN giao nhau ít nhất một node.** ⌈(N+1)/2⌉ + ⌈(N+1)/2⌉ > N. Hệ quả:

1. Không thể có hai leader cùng term/epoch được hai đa số bầu (node giao nhau chỉ vote một lần) → chặn split brain **về mặt toán học**, không phụ thuộc đoán đúng ai chết.
2. Quyết định đã được đa số chấp nhận sẽ được *nhìn thấy* bởi bất kỳ đa số nào sau này → quyết định không bao giờ bị lãng quên dù thiểu số chết.

Đây cũng là lý do **cluster 3 hoặc 5 node, không bao giờ chẵn**: N=3 chịu 1 chết (quorum 2), N=4 quorum 3 — vẫn chỉ chịu 1 chết nhưng thêm chi phí; N=5 chịu 2 chết. Và N=2 quorum 2: chết 1 node là tê liệt — tệ hơn 1 node.

## 4. Raft — mổ xẻ cơ chế

Raft (2014) thiết kế với mục tiêu số một là **dễ hiểu** (Paxos đúng nhưng nổi tiếng khó dạy, khó triển khai đúng). Raft = **Replicated State Machine**: mọi node áp dụng cùng một log lệnh theo cùng thứ tự → cùng trạng thái. Consensus dùng để nhất trí về *nội dung log*.

### 4.1. Ba trạng thái, một nhiệm kỳ

```
                 timeout, bầu cử         thắng đa số
   [Follower] ──────────────▶ [Candidate] ──────────▶ [Leader]
        ▲                        │    ▲                   │
        │  thấy term cao hơn     │    │ chia phiếu,       │ thấy term
        └────────────────────────┴    │ timeout mới       │ cao hơn
        ◀─────────────────────────────┴───────────────────┘
```

**Term** (nhiệm kỳ, tăng đơn điệu) là logical clock của Raft: mọi message mang term; node thấy term cao hơn lập tức thành follower. Term chính là **epoch/fencing token** — vũ khí chống leader "ma" (leader cũ tỉnh dậy sau GC pause mang term thấp → mọi node từ chối).

### 4.2. Leader Election

Follower không nhận được heartbeat trong `election timeout` (**ngẫu nhiên hóa** 150–300ms — chìa khóa chống chia phiếu vĩnh viễn: hiếm khi hai node cùng timeout một lúc) → tăng term, thành candidate, xin vote. Node chỉ vote một lần mỗi term, và **chỉ vote cho candidate có log ít nhất mới bằng mình** — đảm bảo leader mới chứa mọi entry đã commit (không mất dữ liệu đã hứa).

### 4.3. Log Replication & Commit

```
Client ──▶ Leader: append entry (term=3, index=8)
             │ AppendEntries song song
             ├──▶ F1: ack ✓        ┐ đa số (2/3 + leader)
             ├──▶ F2: ack ✓        ┘ → COMMITTED
             └──▶ F3: (chậm/chết)     → sẽ được đồng bộ sau
Leader ──▶ Client: OK (sau khi committed)
```

Entry **committed** khi đa số đã ghi. AppendEntries mang `(prevLogIndex, prevLogTerm)` — follower từ chối nếu không khớp → leader lùi dần và ghi đè phần log phân kỳ của follower. **Log của leader là chân lý; log chưa-commit của follower có thể bị cắt bỏ** — an toàn vì chưa commit nghĩa là chưa hứa với ai.

**Linearizable read**: leader không được trả lời read một mình — nó có thể đã bị phế truất mà chưa biết. Cần ReadIndex (xác nhận quorum mình còn là leader) hoặc lease-based read (nhanh hơn, đổi bằng giả định clock). Đây là chi phí ẩn của "đọc strong" mà chương 03 đã hứa giải thích.

### 4.4. Trong production

Raft ở khắp nơi: **etcd** (não của Kubernetes), **Consul**, **CockroachDB/TiDB/YugabyteDB** (Raft per range — hàng nghìn nhóm Raft), **Kafka KRaft** (metadata, thay ZooKeeper), **RabbitMQ quorum queues**, **MongoDB** (election protocol "Raft-inspired").

Vận hành: cluster consensus nhạy với **disk fsync latency** (mỗi entry phải fsync trước khi ack — đĩa chậm là cluster chậm) và **network jitter** (election timeout phải > p99 RTT + GC). etcd khuyến nghị SSD riêng, cluster nhỏ (3–5 node), dữ liệu nhỏ (<8GB) — consensus là **control plane**, không phải data plane.

## 5. Paxos — mức khái niệm và so sánh

Paxos (Lamport, 1989/1998) giải single-value consensus qua 2 pha (Prepare/Promise với ballot number, Accept/Accepted với quorum). Multi-Paxos chạy liên tiếp cho một log + tối ưu bỏ pha Prepare khi leader ổn định — đến đây *về chức năng tương đương Raft*.

| | Paxos / Multi-Paxos | Raft |
|---|---|---|
| Đơn vị tư duy | Từng giá trị đơn lẻ, ghép thành log | Log là khái niệm gốc |
| Leader | Tối ưu hóa (không bắt buộc trong lý thuyết) | Khái niệm trung tâm |
| Log có lỗ hổng | Cho phép (linh hoạt hơn, khó đúng hơn) | Không — log liên tục, đơn giản hơn |
| Độ khó triển khai đúng | Rất cao ("Paxos Made Live" của Google kể phải sáng tạo thêm rất nhiều thứ paper không nói) | Thiết kế để triển khai được |
| Dùng thật | Chubby (Google), Spanner, Megastore, Cassandra LWT (biến thể) | etcd, Consul, CockroachDB, TiKV, KRaft... |
| Biến thể đáng biết | EPaxos, Flexible Paxos (quorum Q1+Q2>N thay vì hai đa số) | Joint consensus (đổi membership) |

Kết luận thực dụng: **hiểu Raft kỹ, biết Paxos tồn tại và vì sao Google dùng nó trước** (vì lúc đó chưa có gì khác). Đừng bao giờ tự triển khai — dùng etcd/ZooKeeper/thư viện đã qua Jepsen.

## 6. Split Brain & Fencing — từ lý thuyết đến tai nạn thật

Consensus chặn split brain *bên trong cluster consensus*. Nhưng tai nạn xảy ra ở **ranh giới với hệ bên ngoài**:

```
Kịch bản kinh điển (distributed lock):
t1: Client A lấy lock từ etcd (lease 10s)
t2: A bị GC pause 15 giây ████████████
t3: lease hết hạn → B lấy được lock, bắt đầu ghi storage
t4: A tỉnh dậy, KHÔNG BIẾT mình mất lock, tiếp tục ghi storage
    → hai writer đồng thời, dữ liệu hỏng — dù etcd không hề sai!
```

Giải pháp: **fencing token** — lock service phát số tăng đơn điệu kèm lock (Raft term, ZooKeeper zxid, etcd revision); **storage phía dưới phải kiểm tra và từ chối token cũ**. Không có sự hợp tác của resource cuối cùng, distributed lock chỉ là "lời khuyên". Đây là phê phán nổi tiếng của Kleppmann với Redlock (Redis): an toàn của nó phụ thuộc giả định timing, và thiếu fencing thì mọi lock-qua-lease đều có lỗ hổng GC-pause ở client.

## 7. Trade-off

- **Safety vs Liveness**: Raft/Paxos tuyệt đối safety, chấp nhận đứng hình khi không đủ quorum. Hệ AP (chương 04) chọn ngược. Không có thuật toán nào cho cả hai khi partition — đây là CAP mặc đồng phục toán học.
- **Latency**: mỗi write consensus = 1 RTT tới đa số + fsync. Cùng AZ: ~1–2ms. Xuyên region: 30–150ms — lý do consensus xuyên region (Spanner, CockroachDB multi-region) phải thiết kế topology cẩn thận (leaseholder gần user, quorum trong châu lục).
- **Throughput**: một nhóm Raft = một leader = trần một node. Scale bằng **nhiều nhóm Raft** (per-partition — CockroachDB hàng nghìn range), đổi bằng: transaction xuyên nhóm cần 2PC phía trên (chương 08).
- **Cluster size**: 5 node chịu 2 chết nhưng mỗi write chờ 3 ack; 3 node nhanh hơn, chịu 1. Consensus **không phải công cụ scale** — là công cụ *đúng đắn*; đừng cho 9 node để "tăng throughput" (chỉ tăng độ trễ).

## 8. Production / Best Practices / Anti-patterns

**Best practices:**
1. Không tự viết consensus. Dùng etcd/ZooKeeper/Consul hoặc DB có sẵn.
2. Cluster 3 hoặc 5, số lẻ, trải 3 AZ (2 AZ là bẫy: AZ chứa 2/3 node chết = mất quorum).
3. Dành cho control plane (metadata, lock, election, config) — không nhét data plane throughput cao vào etcd.
4. Fencing token cho mọi tài nguyên được bảo vệ bởi lock/lease.
5. Giám sát: leader changes/giờ (flapping = mạng/GC có vấn đề), proposal latency, fsync latency, quorum health.
6. Backup định kỳ cluster consensus (etcd snapshot) — mất quorum vĩnh viễn (đa số node chết không hồi) phải restore từ snapshot, chấp nhận mất phần sau đó.

**Anti-patterns:**
- Chạy 2 hoặc 4 node. — Dùng etcd làm message queue / kho KV dung lượng lớn. — Election timeout < GC pause p99. — Lock không fencing. — Consensus xuyên region cho dữ liệu chỉ cần trong một region. — Đặt cả 3 node trên cùng hypervisor/rack ("3 node" trên giấy, 1 failure domain trên thực tế).

**Khi nào KHÔNG cần consensus**: dữ liệu hòa giải được bằng CRDT/merge (không cần thứ tự toàn cục); coordination tránh được bằng thiết kế (partition ownership tĩnh, idempotency thay lock); single node + failover thủ công đủ cho SLA (nhiều hệ nội bộ!). Quorum lease đơn giản (1 bản ghi CAS trên DynamoDB/PostgreSQL với TTL + fencing) thay được "cluster ZooKeeper để làm mỗi leader election cho 2 worker".

## 9–10. Troubleshooting

| Triệu chứng | Nguyên nhân khả dĩ | Xử lý |
|---|---|---|
| Leader election liên tục (flapping) | Election timeout < p99(RTT+GC+fsync); mạng chập chờn | Tăng timeout; SSD riêng cho WAL; xem GC log |
| Cluster đứng hình, không leader | Mất quorum (đa số node chết/partition) | Đếm node theo failure domain; khôi phục node, đừng vội force-new-cluster |
| Write consensus chậm | fsync chậm (đĩa share), follower chậm kéo p99 | Disk riêng; xem follower lag; giảm payload |
| Hai process cùng tin mình giữ lock | Lease + GC pause, không fencing | Thêm fencing token, resource kiểm tra token |
| etcd "database space exceeded" | Không compaction/defrag, revision phình | Auto-compaction, defrag định kỳ, đừng dùng làm data store |
| Split brain sau khi "force reconfigure" | Con người ép hai phía cùng thành cluster mới | Quy trình force-recovery chỉ một phía; kiểm kê dữ liệu phân kỳ |

---

## Tổng kết chương

### Những điều bắt buộc phải nhớ
1. FLP: không thể *đảm bảo* consensus kết thúc trong hệ bất đồng bộ — mọi thuật toán thật hy sinh liveness lúc xấu trời, **không bao giờ** hy sinh safety.
2. Quorum đa số giao nhau → không thể hai leader cùng term → split brain bị chặn bằng toán, không bằng phỏng đoán.
3. Cluster số lẻ, 3 hoặc 5, trải đủ failure domain.
4. Term/epoch là fencing token; **mọi lock/lease không có fencing đều có lỗ hổng GC-pause**.
5. Consensus là công cụ control-plane cho *đúng đắn*, không phải công cụ scale — scale bằng nhiều nhóm consensus per-partition.
6. Linearizable read cũng phải trả giá quorum — đọc từ leader "chay" là không đủ.

### Hiểu lầm phổ biến
- "Raft dễ nên yếu hơn Paxos" — tương đương về đảm bảo; khác về khả năng triển khai đúng.
- "Có ZooKeeper là hết split brain" — chỉ trong phạm vi ZK; ranh giới với storage bên ngoài vẫn cần fencing.
- "Thêm node consensus tăng availability VÀ hiệu năng" — availability có, hiệu năng *giảm*.
- "Failure detector có thể chính xác" — không thể, chỉ có thể *đủ tốt có chủ đích*.

### Câu hỏi tự kiểm tra
1. Cluster Raft 5 node, 2 node bị partition khỏi 3 node còn lại. Phía nào phục vụ write? Client kẹt phía thiểu số trải nghiệm gì? Ánh xạ sang CAP.
2. Vẽ timeline: leader Raft bị GC pause 20s (election timeout 5s). Chuyện gì xảy ra khi nó tỉnh? Vì sao nó KHÔNG gây split brain (khác gì kịch bản lock ở §6)?
3. Thiết kế distributed lock trên PostgreSQL cho 5 worker (chỉ 1 chạy cron): schema, TTL, fencing, và kịch bản GC-pause.
4. Vì sao CockroachDB chạy hàng nghìn nhóm Raft thay vì một nhóm to? Chi phí phát sinh là gì?

### Tài liệu kinh điển nên đọc
- **"In Search of an Understandable Consensus Algorithm" (Ongaro & Ousterhout, 2014)** — paper Raft; kèm visualization tại raft.github.io — cách học consensus tốt nhất hiện có.
- **"Paxos Made Simple" (Lamport, 2001)** — Paxos bằng văn xuôi; đọc sau Raft sẽ thấy "à, cùng một thứ".
- **"Paxos Made Live" (Google, 2007)** — khoảng cách tàn nhẫn giữa paper và production: đĩa hỏng, membership change, và mọi thứ paper không nói.
- **"Impossibility of Distributed Consensus with One Faulty Process" (FLP, 1985)** — giới hạn nền tảng; đọc phần intro và kết luận là đủ giá trị.
- **"How to do distributed locking" (Kleppmann) + phản hồi của antirez** — cuộc tranh luận Redlock: bài học về fencing và giả định timing, bắt buộc đọc cả hai phía.
- **"ZooKeeper: Wait-free coordination..." (2010)** — thiết kế coordination service tổng quát đầu tiên được dùng đại trà.
