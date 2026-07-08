+++
title = "Chương 04 – CAP & PACELC: Hiểu đúng, dùng đúng, và biết giới hạn"
date = "2026-07-08T20:00:00+07:00"
draft = false
tags = ["backend", "distributed-systems"]
series = ["Distributed Systems"]
+++

## 1. Problem Statement

Khi thiết kế hệ thống có replication, sớm muộn bạn phải trả lời: **khi mạng giữa các replica đứt (network partition), hệ thống làm gì?** Từ chối phục vụ để giữ dữ liệu đúng, hay tiếp tục phục vụ và chấp nhận dữ liệu phân kỳ? CAP Theorem là công cụ tư duy chuẩn hóa cho câu hỏi đó — nhưng nó cũng là định lý **bị hiểu sai nhiều nhất** trong ngành, và hiểu sai dẫn đến quyết định kiến trúc sai ("chọn 2 trong 3" là cách đọc sai phổ biến nhất).

## 2. Tại sao CAP tồn tại

Cuối thập niên 1990, Eric Brewer quan sát: các hệ web quy mô lớn liên tục phải chọn giữa "đúng tuyệt đối" và "sống sót". Ông phát biểu thành phỏng đoán (2000), Gilbert & Lynch chứng minh hình thức (2002). Giá trị lịch sử của CAP không nằm ở toán — nằm ở chỗ nó **hợp pháp hóa** việc xây hệ thống không-strong-consistency một cách có chủ đích (mở đường cho Dynamo, Cassandra, và cả phong trào NoSQL).

## 3. Bản chất — CAP nói gì, chính xác

### 3.1. Ba chữ cái, định nghĩa chặt

- **C — Consistency**: linearizability (nghĩa hẹp của chương 03, *không phải* "dữ liệu đúng" chung chung).
- **A — Availability**: **mọi** request đến node còn sống **phải** nhận được response (không lỗi, không treo vô hạn). Định nghĩa rất mạnh — 99.99% không phải là A của CAP.
- **P — Partition tolerance**: hệ tiếp tục hoạt động khi mạng chia các node thành các nhóm không liên lạc được nhau.

### 3.2. Chứng minh trong 5 dòng

```
Partition chia hệ thành hai phía:   [G1: n1]  ═╪═  [G2: n2]   (không liên lạc được)

Client ghi x=2 vào n1 (G1) → thành công (vì hệ chọn Available)
Client đọc x từ n2 (G2):
  • Trả lời ngay → trả x=1 (cũ) → VI PHẠM C
  • Chờ hỏi n1  → chờ vô hạn (partition) → VI PHẠM A
Không tồn tại lựa chọn thứ ba. ∎
```

### 3.3. Cách đọc ĐÚNG: không phải "chọn 2 trong 3"

**P không phải lựa chọn.** Mạng thật *sẽ* partition — switch hỏng, config sai, cáp đứt, GC pause dài cũng tương đương partition. "Từ bỏ P" nghĩa là hệ thống được phép sai tùy ý khi mạng lỗi — không ai chấp nhận điều đó. Vậy lựa chọn thật chỉ là:

> **Khi partition xảy ra: chọn C (từ chối một phần request) hay chọn A (phục vụ, chấp nhận phân kỳ)?**

- **CP**: phía thiểu số (minority) ngừng phục vụ; phía đa số (majority) tiếp tục — nhờ quorum. Ví dụ: etcd, ZooKeeper, HBase, Spanner. Lưu ý tinh tế: hệ CP *vẫn phục vụ được* ở phía majority — CP không có nghĩa "sập toàn bộ khi partition".
- **AP**: mọi phía tiếp tục nhận read/write; sau khi mạng lành, hòa giải xung đột (vector clock, LWW, CRDT). Ví dụ: Cassandra, DynamoDB (chế độ mặc định), CouchDB, DNS.

### 3.4. Giới hạn của CAP — vì sao nó KHÔNG đủ để thiết kế

1. **CAP chỉ nói về lúc partition** — vốn hiếm. 99.9% thời gian còn lại, trade-off chi phối là **latency vs consistency**, CAP im lặng hoàn toàn về điều này.
2. **C và A đều là định nghĩa cực đoan**: giữa linearizability và "không cam kết gì" có cả một phổ (chương 03); giữa "mọi request đều được trả lời" và "sập" có mọi mức degraded service. Hệ thật sống ở vùng giữa, CAP chỉ mô tả hai góc.
3. **Granularity**: một hệ thống có thể CP cho thao tác này, AP cho thao tác kia (DynamoDB: `ConsistentRead=true` là CP-read, mặc định là AP-read). Dán nhãn cả DB là "CP" hay "AP" là lười tư duy.

## 4. PACELC — bản nâng cấp thực dụng

Daniel Abadi (2012): 

> **P**artition ⇒ chọn **A** hay **C**; **E**lse (bình thường) ⇒ chọn **L**atency hay **C**onsistency.

Vế "Else" chính là phần CAP thiếu — và là trade-off bạn trả tiền **mỗi request, mỗi ngày**: muốn strong consistency lúc bình thường vẫn phải chờ quorum/round-trip; muốn latency thấp nhất thì đọc replica gần nhất và chịu dữ liệu cũ.

| Hệ thống | Khi Partition | Khi bình thường | Ghi chú |
|---|---|---|---|
| DynamoDB / Cassandra (mặc định) | PA | EL | Tunable từng request |
| MongoDB (mặc định) | PC* | EC | *majority write concern; cấu hình yếu từng gây mất ghi (Jepsen) |
| etcd / ZooKeeper | PC | EC | Đúng vai: metadata, lock, election |
| **Spanner / CockroachDB** | **PC** | **EC** | Chấp nhận trả latency để có C toàn cầu |
| PostgreSQL async replica | PC (mặc định failover thủ công) | EL (đọc replica) | Tùy topology triển khai |
| Kafka | Tunable: `acks=all` PC / `acks=1` PA | Tunable | min.insync.replicas quyết định |

### Case: các hệ thực tế "đối xử" với CAP thế nào

**Google Spanner — "CA có phải không?"** Google từng viết bài "Spanner, TrueTime & The CAP Theorem" nói Spanner "về hiệu quả là CA". Đọc kỹ: Spanner **là CP về nguyên tắc** — khi partition thật, minority từ chối write. Nhưng Google sở hữu mạng riêng toàn cầu với redundancy đủ dày để partition hiếm tới mức availability đo được vẫn >5 số 9. Bài học đúng: **không phá được CAP, nhưng mua được xác suất P nhỏ bằng tiền hạ tầng**. Cạnh đó, TrueTime (chương 12) giúp Spanner có external consistency với chi phí latency được kiểm soát (chờ độ bất định đồng hồ ~vài ms).

**DynamoDB**: bản Dynamo gốc 2007 là AP thuần (sloppy quorum, vector clock, conflict do app xử lý — giỏ hàng Amazon hợp nhất bằng union). DynamoDB thương mại ngày nay cho *chọn*: eventually consistent read (rẻ, nhanh, AP) hoặc strongly consistent read (đắt gấp đôi RCU, không phục vụ được nếu partition chạm leader của partition đó — CP). Trade-off được đưa thẳng vào **bảng giá** — minh họa đẹp nhất rằng consistency là thứ có đơn giá.

**Kafka**: `acks=all` + `min.insync.replicas=2` + `unclean.leader.election.enable=false` → nghiêng C (không mất message đã ack, nhưng partition có thể ngừng nhận ghi). `acks=1` + unclean election = nghiêng A (luôn nhận ghi, có thể mất message khi failover). Một dòng config, đổi cả vị trí trên bản đồ CAP — trade-off nằm ở **cấu hình**, không ở logo sản phẩm.

**PostgreSQL**: `synchronous_commit = on` với sync replica → C, nhưng replica chết là write treo (nên thường cấu hình ANY 1 trong 2 sync standby). `synchronous_commit = local` → nhanh, mất dữ liệu nếu leader chết trước khi replica kịp nhận. Failover tự động vội vàng + async replication = **split brain hoặc mất dữ liệu** — sự cố kinh điển của GitHub (2018, 43 giây partition → 24h degraded).

## 5. Trade-off — quyết định thế nào trong thực tế

Khung câu hỏi thay thế cho "chọn CA/CP/AP":

1. **Với từng thao tác ghi**: nếu hai bản ghi xung đột được tạo ở hai phía partition, hòa giải được không, chi phí bao nhiêu? Giỏ hàng: union được → nghiêng A. Trừ tồn kho món hàng cuối: không hòa giải được (đã giao cho 2 người) → nghiêng C.
2. **Với từng thao tác đọc**: dữ liệu cũ X giây gây thiệt hại gì? Feed mạng xã hội: 0 đồng. Số dư trước lệnh chuyển: rất nhiều tiền.
3. **Lúc bình thường**: chịu thêm bao nhiêu ms latency cho consistency? Đa region thì con số là 50–300ms — thường quyết định luôn kiến trúc (chương 13).
4. **Blast radius khi chọn C**: minority ngừng phục vụ ảnh hưởng bao nhiêu % user? Thiết kế topology (3 AZ, quorum 2/3) để "minority" luôn là phần nhỏ.

## 6–9. Production, Best Practices, Anti-patterns, Khi nào không cần quan tâm

**Production considerations:**
- **Diễn tập partition** (chaos engineering): chặn traffic giữa các AZ trong game day và xem hệ thống làm đúng thiết kế không. Đa số hệ "CP trên giấy" hóa ra sập cả hai phía, hoặc "AP trên giấy" hóa ra mất dữ liệu không hòa giải được.
- **Giám sát split-brain risk**: cảnh báo khi hai node cùng nhận mình là leader (fencing token mismatch, epoch tụt lùi — chương 07).
- **Định nghĩa degraded mode tường minh**: khi chọn C và minority ngừng ghi, UX là gì? (đọc-only banner? hàng đợi offline?) — quyết định lúc thiết kế, đừng để lúc 3 giờ sáng.

**Anti-patterns:**
- Dán nhãn CA cho hệ thống của mình ("mạng nội bộ em ổn lắm") — nghĩa là chưa thiết kế hành vi khi partition, tức là hành vi sẽ là *ngẫu nhiên*.
- Chọn AP rồi không xây conflict resolution — phân kỳ âm thầm, phát hiện sau 6 tháng bằng khiếu nại khách hàng.
- Chọn CP cho mọi thứ vì "an toàn" — trả latency và availability cho cả những dữ liệu chẳng cần.
- Tranh luận CAP ở tầng cả-hệ-thống thay vì tầng từng-thao-tác.

**Khi nào không cần quan tâm**: hệ một node (không replication) — không có P thì không có CAP; dữ liệu immutable — không có write xung đột; hệ bất đồng bộ thuần túy nơi mọi consumer đều idempotent và hòa giải được.

## 10. Troubleshooting dưới lăng kính CAP

| Tình huống | Phân tích |
|---|---|
| Một AZ mất kết nối, service ở AZ đó vẫn nhận write vào cache cục bộ | Bạn đang chạy AP ngoài ý muốn → kiểm kê phân kỳ, xây reconciliation |
| etcd cluster 2 node, 1 chết, toàn cluster đọc-ghi tê liệt | Quorum 2/2 bất khả → **luôn chạy số lẻ node** (3/5); 2 node tệ hơn 1 node |
| Failover PostgreSQL xong mất 30 giây dữ liệu | Async replication + promote replica trễ → cần sync replica hoặc chấp nhận RPO>0 tường minh (chương 13) |
| Cassandra hai DC ghi xung đột, LWW chọn "kẻ thắng" sai | Clock skew giữa DC + Last-Write-Wins = mất write âm thầm (chương 12) |

---

## Tổng kết chương

### Những điều bắt buộc phải nhớ
1. P không phải lựa chọn; câu hỏi thật là **khi partition, hy sinh C hay A** — và trả lời **per-operation**.
2. PACELC quan trọng hơn CAP trong vận hành hằng ngày: **latency vs consistency là trade-off bạn trả mỗi request**.
3. CP không nghĩa là sập khi partition (majority vẫn chạy); AP không nghĩa là dữ liệu sai mãi (cần reconciliation).
4. Vị trí trên bản đồ CAP nằm ở **cấu hình** (acks, write concern, consistency level), không ở tên sản phẩm.
5. Spanner không phá CAP — Google mua xác suất partition thấp bằng mạng riêng, và vẫn là CP khi partition thật.

### Hiểu lầm phổ biến
- "Chọn 2 trong 3" — cách đọc sai phổ biến nhất.
- "CA tồn tại cho hệ phân tán" — chỉ khi mạng không bao giờ lỗi, tức là không tồn tại.
- "MongoDB/Cassandra là AP, hết chuyện" — tunable; và cấu hình mặc định của bạn có thể không phải cái bạn tưởng.
- "CAP bảo eventual consistency là bắt buộc để scale" — CAP không nói gì về scale; đó là chuyện của partitioning (chương 06).

### Câu hỏi tự kiểm tra
1. Hệ đặt vé máy bay: thao tác nào phải CP, thao tác nào nên AP? Khi partition giữa 2 DC, UX cụ thể của từng loại?
2. Giải thích vì sao cluster consensus 4 node không tốt hơn 3 node (gợi ý: quorum size và số node chịu chết được).
3. Kafka của bạn `acks=1`, unclean election bật. Vẽ timeline một kịch bản mất message đã ack.
4. PACELC của Redis Sentinel (async replication, tự động failover) là gì? Hệ quả với dữ liệu?

### Tài liệu kinh điển nên đọc
- **"Brewer's Conjecture and the Feasibility of..." (Gilbert & Lynch, 2002)** — chứng minh hình thức, đọc để biết chính xác định lý nói gì (hẹp hơn folklore nhiều).
- **"CAP Twelve Years Later" (Brewer, 2012)** — chính tác giả sửa cách hiểu "2 trong 3", giới thiệu tư duy "quản lý partition" thay vì "chọn phe".
- **"Consistency Tradeoffs in Modern Distributed Database System Design" (Abadi, 2012)** — PACELC gốc.
- **"Spanner, TrueTime & The CAP Theorem" (Brewer, 2017)** — case study "mua" availability bằng hạ tầng.
- **"Please stop calling databases CP or AP" (Kleppmann)** — phê phán sắc bén việc dán nhãn; thuốc giải cho design review.
