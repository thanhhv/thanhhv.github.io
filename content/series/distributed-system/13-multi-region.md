+++
title = "Chương 13 – Multi-region & Disaster Recovery: Bài toán Principal-level"
date = "2026-07-09T05:00:00+07:00"
draft = false
tags = ["backend", "distributed-systems"]
series = ["Distributed Systems"]
+++

## 1. Problem Statement

Ba lực đẩy một hệ thống ra nhiều region: **(1) Latency** — tốc độ ánh sáng: Singapore↔Frankfurt ~160ms RTT, không tiền nào mua nhanh hơn được, user xa DC là UX tệ; **(2) Disaster Recovery** — cả một region *có thể* chết (AWS us-east-1 đã nhiều lần chứng minh), và với một số nghiệp vụ/regulator, "chúng tôi phụ thuộc một region" là câu trả lời không được chấp nhận; **(3) Data residency** — luật (GDPR, nghị định địa phương hóa dữ liệu) buộc dữ liệu công dân nằm trong biên giới.

Nếu làm sai, multi-region là cách đốt tiền hiệu quả nhất trong kiến trúc: chi phí hạ tầng x2–3, cross-region traffic tính tiền theo GB, độ phức tạp vận hành x5, và **một failover không được diễn tập sẽ thất bại đúng lúc cần nó nhất** — bạn trả tiền cho DR trong nhiều năm và nhận về một cú sập kép.

Trước khi đọc tiếp, câu hỏi principal-level số một: **bạn có thật sự cần multi-region không?** Một region 3 AZ đã chịu được hỏng DC (AZ là các DC tách biệt vật lý); đa số công ty chết vì bug và lỗi vận hành, không vì thiên thạch rơi vào region. Multi-AZ là mặc định đúng; multi-region là quyết định phải được biện minh bằng con số (SLA, luật, phân bố user).

## 2. RPO / RTO — ngôn ngữ của mọi quyết định DR

- **RPO (Recovery Point Objective)**: mất tối đa bao nhiêu *dữ liệu* (tính bằng thời gian). RPO=0 nghĩa là không mất write nào đã ack → **bắt buộc sync replication** → trả bằng latency mỗi write (chương 05).
- **RTO (Recovery Time Objective)**: mất tối đa bao nhiêu *thời gian phục vụ*. RTO phút → failover tự động; RTO giây → phải active-active sẵn.

Hai con số này là **quyết định nghiệp vụ trả bằng tiền**, không phải nguyện vọng: mỗi nấc giảm là một nấc chi phí + phức tạp. Bảng thang chuẩn:

| Chiến lược | RPO | RTO | Chi phí tương đối | Bản chất |
|---|---|---|---|---|
| Backup + restore | Giờ (từ backup cuối) | Giờ–ngày | 1x + storage | Backup chở sang region khác |
| Pilot light | Phút | Chục phút–giờ | ~1.2x | Data replicate liên tục; compute tắt, bật khi cần |
| Warm standby | Giây–phút | Phút | ~1.5–2x | Bản thu nhỏ chạy sẵn, scale lên khi failover |
| Active-Passive (hot) | ~0–giây | Phút | ~2x | Bản đầy đủ chạy sẵn, chỉ chờ nhận traffic |
| Active-Active | ~0 | ~0 (giây) | 2–3x + độ phức tạp **dữ liệu** | Mọi region cùng phục vụ |

## 3. Active-Passive vs Active-Active — khác nhau ở tầng dữ liệu

### 3.1. Active-Passive

Mọi write về region chính; region phụ nhận replication (thường async — vì sync xuyên region = +50–150ms mỗi write, ít ai chịu). Đơn giản nhất về dữ liệu: một leader, không conflict.

Hai cái bẫy lớn:
- **Async lag = RPO thật**: quảng cáo "RPO gần 0" nhưng lag lúc sự cố (thường chính là lúc mạng có vấn đề!) có thể là phút. Đo lag liên tục, coi nó là RPO đang trôi nổi theo thời gian thực.
- **Passive region bị mục (bit rot)**: config trôi, capacity không đủ tải thật, dependency chưa được nhân bản (cái S3 bucket "nhỏ nhỏ" chỉ có ở region chính), secret hết hạn. **Failover chưa từng chạy = failover không tồn tại.** Thuốc duy nhất: diễn tập failover thật định kỳ (quý), lý tưởng là chạy production trên region phụ vài ngày mỗi quý — nếu không dám làm điều đó, bạn không có DR, bạn có niềm tin.

Còn một quyết định hay bị né: **fail-back** (quay về region chính) và **quyết định failover do ai bấm** — tự động (rủi ro split brain khi chỉ là partition giữa 2 region — cần bên thứ ba làm trọng tài/quorum region) hay con người (cộng 15–30 phút vào RTO). Đa số chọn: phát hiện tự động, quyết định con người, thực thi một-nút-bấm đã tập dượt.

### 3.2. Active-Active

Mọi region cùng nhận traffic. Câu hỏi quyết định tất cả: **write đi đâu?** Bốn kiến trúc dữ liệu, từ dễ đến khó:

1. **Read-local, write-global**: đọc tại chỗ (replica địa phương), mọi write về một region leader. Được 80% lợi ích latency (đa số traffic là read) với 20% độ phức tạp. **Điểm khởi đầu đúng cho hầu hết.**
2. **Partition theo địa lý (home region)**: user VN có "nhà" ở Singapore, user EU ở Frankfurt — write của ai về nhà nấy. Mỗi bản ghi vẫn một leader → không conflict; luật residency được chiều luôn. Phải xử lý: user du lịch (write xuyên region về nhà — chậm nhưng đúng), di cư nhà, và dữ liệu *toàn cục thật sự* (catalog, config — thường read-local + write-global).
3. **Multi-leader conflict resolution**: mọi region nhận mọi write, hòa giải sau (chương 05: LWW/CRDT/app-merge) — chỉ cho dữ liệu hòa giải được; đừng cho tiền đi đường này.
4. **Consensus toàn cầu (Spanner, CockroachDB multi-region)**: strong consistency xuyên region, trả bằng latency quorum liên lục địa cho write (giảm nhẹ bằng đặt leaseholder/leader gần nơi ghi nhiều — "regional by row" của CockroachDB là dạng công nghiệp hóa của mục 2).

DynamoDB Global Tables = mục 3 với LWW (chấp nhận mất write xung đột); Aurora Global = mục 1 công nghiệp hóa (storage-level replication <1s lag, promote region phụ ~phút).

### 3.3. Định tuyến + Data Locality

GeoDNS/Anycast (CloudFront, Route53 latency-based) đưa user về region gần; **session stickiness theo region** tránh một phiên nhảy 2 region đọc 2 trạng thái khác nhau (read-your-writes xuyên region — chương 03 phiên bản đắt nhất). Kiến trúc đẹp là kiến trúc *ít phải nói chuyện xuyên region trong hot path nhất*: mọi cross-region call đồng bộ trong đường phục vụ request là một quyết định phải bị chất vấn ở design review.

## 4. Cross-datacenter Replication — chi tiết bên trong

- **Tầng nào replicate?** Storage-level (Aurora Global, GCS dual-region: trong suốt với app, ít kiểm soát) vs database-level (PG logical replication, MySQL GTID: kiểm soát bảng/hướng, tự vận hành) vs **application-level qua event stream** (Kafka MirrorMaker2/Confluent Cluster Linking: linh hoạt nhất — replicate event, mỗi region tự build state; kéo theo toàn bộ chương 09).
- **Băng thông + tiền**: cross-region data transfer ~$0.02/GB (AWS) — hệ ghi 50MB/s là ~$2.5K/tháng *chỉ tiền chuyển*, nhân số hướng replicate. Nén, lọc (chỉ replicate cái region kia cần), và đưa con số này vào business case ngay từ đầu.
- **Thứ tự và trùng lặp xuyên region**: MirrorMaker không bảo toàn offset, ordering chỉ per-partition, và failover consumer xuyên region là bài toán riêng (offset translation). Mọi consumer phải idempotent — lần thứ ba tài liệu này nói câu này, vì nó cứ đúng mãi.

## 5. Disaster Recovery — kỷ luật hơn là kiến trúc

1. **DR bắt đầu bằng phân loại**: không phải mọi service cần cùng RTO/RPO. Tier 0 (thanh toán): active-active; Tier 1 (catalog): warm standby; Tier 2 (analytics): backup là đủ. DR đồng phục cho mọi thứ = trả tiền Tier 0 cho log viewer.
2. **Backup 3-2-1 vẫn là đáy của mọi tầng**: replication không chống được `DELETE` nhầm, ransomware, bug ghi rác (nó chở cái sai đi khắp nơi nhanh hơn — chương 05). Backup **immutable** (S3 Object Lock), **khác account/region**, và **restore drill định kỳ có đo giờ** — con số đo được đó mới là RTO thật của tầng cuối.
3. **Game day**: tắt region phụ thuộc (hoặc chặn mạng tới nó) trong giờ hành chính, có kịch bản, có nút hủy. Netflix chạy Chaos Kong (rút cả region) như thói quen — vì thế các region outage của AWS thường chỉ làm Netflix sượt nhẹ.
4. **Runbook một-nút-bấm**: lúc 3h sáng, người trực không được phải "nhớ ra" 14 bước. Mọi bước failover phải là script/pipeline đã chạy thử, có checklist quyết định (khi nào bấm, ai có quyền bấm, làm sao biết xong).
5. **Cẩn thận failover dây chuyền**: region chết → toàn bộ traffic dồn về region còn lại → region còn lại **phải có capacity cho 200%** (hoặc load shedding có kế hoạch — chương 11). DR không có capacity planning = chuyển sự cố từ region này sang region kia.

## 6. Cost Optimization — góc nhìn Principal

Chi phí là một chiều thiết kế ngang hàng với latency/consistency, không phải việc của tài chính cuối quý:

- **Cross-AZ/region traffic thường là dòng chi phí ẩn lớn nhất** trong hệ phân tán nhiều chatter (mesh, replication, Kafka RF=3 xuyên AZ). Topology-aware routing (đọc replica cùng AZ, Kafka fetch-from-follower) cắt được phần lớn.
- **Đơn vị đo đúng là cost-per-transaction / cost-per-user**, không phải tổng bill — tổng bill tăng theo tăng trưởng là lành mạnh; cost-per-transaction tăng là kiến trúc đang xấu đi.
- Trade-off tường minh giữa **redundancy và tiền**: mỗi số 9, mỗi giảm RPO/RTO có đơn giá; việc của architect là đưa menu có giá cho business chọn, không phải tự chọn max rồi xin tiền.
- Chiêu thực dụng: passive region dùng spot/ít node + autoscale khi failover (đổi RTO lấy tiền — miễn là *đo* được autoscale kịp); tiered storage (Kafka tiered, S3 IA) cho dữ liệu replicate giữ lâu; xem xét "multi-region cho 5% dữ liệu quan trọng, single-region cho phần còn lại".

## 7. Evolutionary Architecture — đường đến multi-region là tiến hóa, không phải big bang

Trình tự trưởng thành đã được nhiều công ty trả học phí:

```
1 máy → 1 region multi-AZ (mặc định đúng cho đa số, dừng ở đây rất lâu)
      → + CDN/edge cho static & cacheable (giải 70% bài latency toàn cầu, rẻ)
      → + read replica ở region xa (read-local, write-global)
      → + DR passive (pilot light → warm) với drill thật
      → active-active theo home region cho phần dữ liệu cần
      → (hiếm khi cần) consensus toàn cầu cho tập giao dịch hẹp
```

Nguyên tắc tiến hóa: mỗi bước phải **đảo ngược được** và được kích hoạt bởi *bằng chứng* (đo latency user thật, yêu cầu regulator bằng văn bản, sự cố có hậu quả đo được) — không phải bởi slide "world-class architecture". Thiết kế hôm nay chỉ cần *không chặn đường* ngày mai: ví dụ, đặt `region_id`/`home_region` vào schema user ngay từ đầu (gần như miễn phí) để 3 năm sau partition theo địa lý không phải là cuộc đại phẫu; giữ mọi giao tiếp xuyên service qua API/event có version (chương 02, 09) để tách region không phải gỡ mì spaghetti.

## 8. Anti-patterns

- **Multi-region như trang sức**: active-active cho hệ 10K user toàn ở Việt Nam.
- **"DR trên giấy"**: có region phụ, chưa từng failover. Đã nói ở §3.1, nhắc lại vì đây là anti-pattern DR số một toàn ngành.
- **Cross-region call đồng bộ trong hot path**: mỗi request chờ một RTT liên lục địa — mất toàn bộ lợi ích multi-region, giữ nguyên hóa đơn.
- **Hai region, quorum chẵn**: consensus 2 region là bài toán không giải được tử tế (mất 1 = mất quorum hoặc split brain) — cần region thứ 3 làm trọng tài (dù chỉ đặt 1 node witness).
- **Failover DB mà không failover các thứ quanh nó**: DNS TTL 24h, secret chỉ có ở region chính, license server một chỗ, job scheduler ghi đôi. Failover là của *cả hệ thống*, không của mình database.
- **Quên fail-back và chạy "tạm" trên region phụ 6 tháng** — giờ region phụ là chính, và bạn lại không có DR.
- **Backup cùng account/region với production** — ransomware và lệnh xóa nhầm không phân biệt.

## 9. Khi nào KHÔNG multi-region

User tập trung một địa lý + SLA chịu được RTO giờ → **multi-AZ + backup cross-region + CDN** là điểm dừng đúng, tiết kiệm cả núi phức tạp. Team platform < 5 người: multi-region ăn toàn bộ băng thông kỹ sư giỏi nhất. Chưa có: monitoring tử tế, IaC toàn phần, on-call trưởng thành — multi-region đứng trên các móng đó; thiếu móng thì nó chỉ nhân đôi sự hỗn loạn ra hai region.

## 10. Troubleshooting

| Triệu chứng | Nguyên nhân khả dĩ | Xử lý |
|---|---|---|
| Failover xong, hệ "lên" nhưng lỗi rải rác khắp nơi | Dependency ẩn còn trỏ region cũ (hardcode endpoint, secret, bucket) | Dependency inventory; test failover *toàn hệ*, không chỉ DB |
| RPO thực tế tệ hơn cam kết sau sự cố | Replication lag lúc sự cố cao (chính sự cố mạng gây ra) | Giám sát lag như SLI; sync/semi-sync cho tier 0; điều chỉnh cam kết |
| User một phiên thấy dữ liệu nhảy qua lại | Phiên bị route 2 region, replication lag giữa chúng | Region stickiness; read-your-writes token xuyên region |
| Bill tăng vọt sau khi bật multi-region | Cross-region traffic + NAT + duplicate storage | Cost allocation theo luồng; nén/lọc replication; topology-aware routing |
| Split brain giữa 2 region sau partition | Cả hai tự promote (không trọng tài) | Region thứ 3 làm quorum witness; fencing; quy trình quyết định failover |
| Region phụ không gánh nổi tải khi failover thật | Capacity passive < 100% production | Diễn tập với traffic thật; autoscale warm-up đo được; load shedding kế hoạch |

---

## Tổng kết chương

### Những điều bắt buộc phải nhớ
1. RPO/RTO là quyết định nghiệp vụ có đơn giá — đưa menu cho business chọn, phân tier thay vì đồng phục.
2. Multi-AZ là mặc định; multi-region phải được biện minh bằng con số (latency user thật, luật, SLA).
3. Active-active khó ở **tầng dữ liệu**: chọn trong 4 nấc — read-local/write-global → home region → conflict resolution → consensus toàn cầu; bắt đầu từ nấc dễ nhất đủ dùng.
4. Async lag là RPO trôi nổi; failover chưa diễn tập là failover không tồn tại; region nhận failover phải có capacity 200% hoặc kế hoạch shedding.
5. Backup immutable khác region/account là đáy của mọi tầng DR — replication không thay được nó.
6. Multi-region là tiến hóa có bằng chứng, mỗi bước đảo ngược được — không phải big bang.

### Hiểu lầm phổ biến
- "Active-active nghĩa là không bao giờ downtime" — sự cố *dữ liệu* (bug ghi rác, xóa nhầm) replicate đi mọi region trong mili giây.
- "Cloud provider lo DR cho mình" — họ lo hạ tầng *của họ*; kiến trúc, dữ liệu, drill là của bạn (shared responsibility).
- "RPO=0 chỉ là chuyện bật sync replication" — bật nó là bắt mọi write trả latency xuyên region, và write treo khi region kia hắt hơi; RPO=0 là lựa chọn đắt toàn diện.
- "Chúng tôi multi-region vì đã deploy app ra 2 region" — app stateless dễ; **multi-region thật là bài toán của dữ liệu**.

### Câu hỏi tự kiểm tra
1. Nghiệp vụ của bạn: ví điện tử VN, 2M user, regulator yêu cầu dữ liệu trong nước, SLA 99.95%. Thiết kế topology (region/AZ/DR tier) và biện minh từng lựa chọn bằng RPO/RTO/tiền.
2. Aurora Global lag 800ms, region chính chết. Trình bày từng bước failover, RPO thực tế, và xử lý 800ms giao dịch "mồ côi" (gợi ý: đối soát + outbox log từ region chết khi nó hồi sinh).
3. Vì sao consensus 2 region không tử tế được? Thiết kế rẻ nhất để có quorum 3 bên khi bạn chỉ muốn trả tiền 2 region đầy đủ.
4. CTO muốn "active-active toàn cầu trong quý tới". Viết 5 câu hỏi bạn đặt lên bàn trước khi bàn kiến trúc.

### Tài liệu kinh điển nên đọc
- **AWS Well-Architected: "Disaster Recovery of Workloads on AWS" (whitepaper)** — phân loại backup/pilot light/warm/active-active chuẩn ngành, kèm số liệu chi phí.
- **"Spanner" (2012) + CockroachDB multi-region docs** — hai lời giải công nghiệp cho dữ liệu toàn cầu strong consistency; đọc phần topology (leaseholder, regional by row) kỹ hơn phần thuật toán.
- **Netflix Tech Blog: Chaos Kong / Regional Evacuation** — diễn tập rút cả region như thói quen vận hành; chuẩn mực về DR drill.
- **Google SRE Workbook: "Non-Abstract Large System Design" + chương DR** — cách tính capacity failover bằng số.
- **"Building Evolutionary Architectures" (Ford, Parsons, Kua)** — fitness function và kiến trúc đảo-ngược-được; khung tư duy cho §7.
