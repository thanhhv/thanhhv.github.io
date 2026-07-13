+++
title = "13.4. Distributed Failures"
date = "2026-07-13T16:50:00+07:00"
draft = false
tags = ["backend", "system-design"]
series = ["System Design — Tư Duy Thiết Kế Hệ Thống"]
+++

> Ba tình huống: Cascading Failure, Split Brain, Leader Election Failure. Đây là các sự cố "cấp hệ thống" — không thành phần nào hỏng nặng, nhưng tương tác giữa chúng giết cả hệ thống. Nền lý thuyết: [Phần 4](/series/system-design/04-distributed-systems/01-cap-pacelc/).

---

## Case 14 — Cascading Failure

### Triệu chứng

Bắt đầu bằng một sự cố *nhỏ, cục bộ* (một service chậm, một node chết, một deploy xấu) → lan như domino: service A chậm → upstream B cạn thread chờ A → B chậm → C cạn theo... → trong 5–15 phút, **toàn hệ thống đỏ**, kể cả những service không liên quan gì đến điểm khởi phát. Dashboard nhìn đâu cũng cháy — chính điều đó làm mất phương hướng.

### Root cause & tại sao xảy ra

Ba cơ chế lan truyền, thường cùng lúc:

1. **Giam tài nguyên ngược dòng:** B gọi A đồng bộ, A chậm → thread/connection của B bị giam theo thời gian chờ A ([pool exhaustion](/series/system-design/13-production-failure-cases/02-database-failures/)) → B hết tài nguyên phục vụ *mọi thứ khác* — lỗi lan **ngược chiều mũi tên gọi**.
2. **Retry khuếch đại xuôi dòng:** upstream retry vào điểm ốm → ốm thành chết ([retry storm](/series/system-design/13-production-failure-cases/03-messaging-failures/)).
3. **Dồn tải sang hàng xóm:** 1 trong 4 node chết → 3 node còn lại nhận +33% tải; nếu chúng đang chạy 80% → quá tải → chết tiếp → còn 2 node nhận tải của 4 → sụp nhanh dần. Đây là lý do headroom ([chương 1.3](/series/system-design/01-foundations/03-throughput-latency/)) không phải lãng phí: **cụm không đủ đệm cho failover là cụm được thiết kế để sụp dây chuyền.**

Điều kiện nền chung: hệ thống **coupling chặt về thời gian** (sync call chuỗi dài) + không có ngắt mạch giữa các tầng. Giống cháy rừng: một tia lửa chỉ thành thảm họa khi rừng khô và không có băng cản lửa.

### Kiến trúc bị ảnh hưởng

Microservices gọi nhau đồng bộ chuỗi sâu là môi trường kinh điển; nhưng monolith cũng dính (qua DB chung, pool chung). Càng nhiều tầng sync không CB, bán kính càng rộng.

### Metric / Dashboard / Alert

- Metric: error rate + latency **từng cạnh gọi** (không chỉ từng service); pool/thread utilization từng service; retry rate từng cạnh; trạng thái circuit breaker.
- Dashboard quan trọng nhất khi cháy: **service dependency map có màu theo sức khỏe** — để tìm *điểm khởi phát* giữa một rừng màu đỏ.
- Alert: nhiều service cùng suy giảm trong < 5 phút = nghi cascade (meta-alert); CB nào mở = thông báo.

### Điều tra

1. **Tìm gốc theo thời gian:** service nào đỏ *đầu tiên*? (dashboard nhìn về 15–30 phút trước — mọi thứ đang đỏ *bây giờ* không nói lên gì).
2. Hoặc tìm theo topology: đi xuôi chiều mũi tên gọi đến service *sâu nhất* có lỗi — thường là gốc; những gì phía trên là nạn nhân.
3. "Cái gì vừa thay đổi?" ở gốc: deploy, config, traffic, dependency ngoài.

### Khắc phục & phòng tránh

- **Cầm máu — nguyên tắc: thu nhỏ đám cháy trước, cứu gốc sau:**
  1. Cô lập gốc: mở circuit breaker cưỡng bức / route bypass service ốm, cho phần còn lại degraded mode (trang chủ không có recommendation vẫn là trang chủ).
  2. Shed load ở edge: giảm lượng vào toàn hệ (429 một phần traffic) — hệ đang sụp thường cứu được bằng cách *cho nó ít việc hơn* trong vài phút.
  3. Chữa gốc: rollback deploy, restart node, scale.
  4. **Gỡ dần theo tầng từ trong ra** — mở lại 100% ngay thường sụp lần hai (cache lạnh, connection storm — chính là [thundering herd](/series/system-design/13-production-failure-cases/01-caching-failures/) của sự hồi phục).
- **Phòng — xây băng cản lửa:**
  - **Timeout ngắn + circuit breaker ở mọi cạnh sync** — không thương lượng. CB biến "chậm lây lan" thành "lỗi cục bộ nhanh".
  - **Bulkhead:** pool riêng theo dependency — A chết chỉ giam pool-A của B, B vẫn thở.
  - **Degraded mode thiết kế sẵn** cho mọi dependency không thiết yếu: câu hỏi design review — "service X chết thì trang này hiện gì?"
  - Headroom N+1/N+2 và autoscaling đủ nhanh.
  - Giảm độ sâu chuỗi sync; chuyển cạnh không cần kết quả ngay sang async — event không giam thread của ai.
  - Chaos engineering: giết service ở staging định kỳ và xem đám cháy dừng ở đâu — băng cản lửa chưa test là băng cản lửa trên giấy.

---

## Case 15 — Split Brain

### Triệu chứng

Hai node/nhóm **cùng nhận mình là primary**: hai node DB cùng nhận ghi, hai instance scheduler cùng chạy job, hai "leader" xử lý cùng partition. Hệ quả nhìn thấy: dữ liệu phân kỳ (hai user thấy hai giá trị khác nhau *đều "đúng"*), job chạy đúp, unique constraint vỡ khi hai nhánh gặp lại nhau. Thường phát lộ **sau** sự cố mạng, khi các nhánh tái hợp — lúc dữ liệu đã kịp phân kỳ hàng giờ.

### Root cause & tại sao xảy ra

Chuỗi ba mắt xích (chi tiết ở [chương 4.4](/series/system-design/04-distributed-systems/04-clock-partition-split-brain/)):

1. Phát hiện lỗi bằng **timeout** — mà timeout không phân biệt được "chết" với "chậm/mất mạng tạm thời" (giới hạn nguyên lý, không phải bug).
2. Cơ chế promote **không cần đa số** (script tự chế, cụm 2 node, quorum cấu hình sai).
3. Node cũ **không bị chặn** (không fencing) — tỉnh lại sau GC pause/partition, vẫn tưởng mình là leader, tiếp tục nhận ghi.

Đủ ba mắt xích = split brain chỉ còn chờ dịp. Dịp: network partition, GC pause dài, node treo nửa vời, VM migrate.

### Kiến trúc bị ảnh hưởng

Mọi hệ có vai trò "duy nhất" (primary DB, scheduler, lock holder) + failover tự động tự chế. Cụm **2 node** là cấu hình rủi ro kinh điển — không tồn tại đa số ([chương 4.3](/series/system-design/04-distributed-systems/03-consensus-quorum-leader-election/)).

### Metric / Dashboard / Alert

- Metric/Alert quan trọng nhất — rẻ mà cứu mạng: **đếm số node tự nhận primary, alert khi ≠ 1** (mỗi node export gauge `is_primary`; tổng > 1 = split brain ĐANG xảy ra, tổng = 0 = không ai lãnh đạo). Kèm: replication topology thay đổi, số failover event, khoảng cách heartbeat giữa các node.
- Dashboard: topology hiện hành (ai là primary, ai nghĩ ai là primary) — hai cột này khác nhau là còi báo động.

### Điều tra

1. Xác nhận: hai node cùng nhận ghi từ bao giờ? (log promote + log ghi ở cả hai).
2. Khoanh vùng phân kỳ: tập bản ghi được ghi ở mỗi nhánh trong cửa sổ đó (theo timestamp/LSN/GTID).
3. **Đừng vội "sửa"** — mọi thao tác hấp tấp (restart, re-attach replica) có thể xóa một nhánh dữ liệu vĩnh viễn. Chụp trạng thái (backup cả hai nhánh) trước.

### Khắc phục & phòng tránh

- **Cầm máu:** chọn một nhánh làm nguồn sự thật (thường nhánh nhiều ghi hơn / nghiệp vụ quan trọng hơn) → **fence nhánh kia ngay** (chặn ghi) → backup cả hai → hòa giải: bản ghi chỉ có ở nhánh phụ được replay/merge vào nhánh chính theo quy tắc nghiệp vụ — có những xung đột chỉ nghiệp vụ quyết được (hai lần trừ cùng tồn kho), cần người phân xử, minh bạch với khách hàng bị ảnh hưởng.
- **Phòng — đúng ba lớp của [chương 4.4](/series/system-design/04-distributed-systems/04-clock-partition-split-brain/):**
  1. Quorum thật cho quyết định promote (3 node lẻ, 3 AZ; dùng Patroni/orchestrator thay vì script nhà làm).
  2. Lease cho leadership: leader không gia hạn được thì *tự* rút trước khi phía kia bầu mới.
  3. Fencing: token tăng đơn điệu mà storage kiểm tra, hoặc STONITH.
  - Và test: diễn tập partition ở staging — cụm HA chưa từng bị cắt mạng thử là cụm HA chưa được kiểm chứng.

---

## Case 16 — Leader Election Failure

### Triệu chứng

Hai họ triệu chứng ngược nhau:

- **Không bầu được leader (no leader):** cụm sống nhưng từ chối ghi — etcd báo "no leader", Kafka partition không có leader → producer lỗi, Patroni không promote được → DB read-only kéo dài. Hệ thống "đứng im một cách lịch sự".
- **Bầu cử liên miên (election churn / flapping):** leader đổi liên tục mỗi vài giây/phút — mỗi lần đổi là một khoảng dừng ghi + client phải re-discover → latency giật cục, lỗi rải rác không ngừng, log đầy `term changed`, `leadership transferred`.

### Root cause & tại sao xảy ra

Cả hai họ đều là **điều kiện hoạt động của consensus bị phá** (Raft cần: đa số node sống, liên lạc ổn, disk kịp fsync, timeout hợp lý — [chương 4.3](/series/system-design/04-distributed-systems/03-consensus-quorum-leader-election/)):

- *No leader:* mất quorum (≥ nửa cụm chết/mất mạng — hay gặp: 3 node đặt 2 AZ, AZ chứa 2 node sập); hoặc các node sống nhưng không node nào thắng cử (partition đối xứng, cấu hình mạng một chiều).
- *Churn:* election timeout quá ngắn so với thực tế mạng/disk → heartbeat trễ chút là follower tưởng leader chết, khởi động bầu cử oan; leader bị GC pause/CPU steal định kỳ; disk chậm làm leader không kịp fsync heartbeat/log; network flapping. Nhớ FLP: khi môi trường đủ nhiễu, consensus *an toàn nhưng không tiến* — churn chính là hình hài của định lý đó ngoài đời.

### Kiến trúc bị ảnh hưởng

Mọi thứ đứng trên consensus: etcd (→ toàn bộ Kubernetes control plane), ZooKeeper (→ Kafka cũ, HBase), Consul, Kafka KRaft, Patroni. Đặc điểm đáng sợ: các hệ này là **control plane** — chúng ốm thì bộ máy *điều khiển* ốm, và sự cố data plane sau đó không ai chỉ huy xử lý.

### Metric / Dashboard / Alert

- Metric: **leader elections per hour** (baseline gần 0 — vài lần/giờ đã là bệnh); leader tenure (thời gian tại vị trung bình); quorum status (số node khỏe vs cần); proposal/commit latency p99; **disk fsync latency** trên node consensus (thủ phạm bị bỏ sót nhiều nhất); heartbeat RTT giữa các node.
- Alert: elections > 2–3/giờ; mất 1 node của cụm 3 (còn đúng quorum tối thiểu — một sự cố nữa là đứng); fsync p99 > ngưỡng (etcd khuyến nghị < 10ms).

### Điều tra

1. No leader hay churn? — hai nhánh khác nhau.
2. No leader: đếm node sống và liên lạc chéo (đủ đa số không?); node sống có thấy *nhau* không (partition)?
3. Churn: node nào khởi xướng bầu cử (log `starting election`)? Vì sao nó mất heartbeat — mạng (RTT spike?), disk (fsync chậm?), hay leader ngộp (GC log, CPU)?
4. Timeout cấu hình vs thực tế: election timeout có lớn hơn hẳn RTT p99 + fsync p99 không?

### Khắc phục & phòng tránh

- **Cầm máu:** no leader do thiếu node → khôi phục node/mạng để đủ đa số (cùng lắm: quy trình khôi phục disaster chính thức của hệ đó — làm đúng sách, sai một bước là mất an toàn dữ liệu); churn → tăng election timeout tạm thời, giảm tải node consensus, chuyển leader sang node khỏe (`leadership transfer` chủ động).
- **Chữa gốc:** node consensus cần **disk nhanh riêng** (SSD/NVMe local, không chung disk với DB nghiệp vụ), đủ CPU/RAM không tranh chấp (không đặt etcd chung node với workload nặng); timeout đặt theo đo đạc (RTT, fsync) chứ không theo mặc định/blog; 3–5 node trên 3 AZ.
- **Phòng:** giám sát cụm consensus như công dân hạng nhất (nó thường bị quên vì "ổn định quá"); drill: giết leader chủ động mỗi tháng và đo thời gian hồi — con số đó là RTO thật của control plane; giữ cụm **nhàm chán** — không nhét thêm dữ liệu, không dùng chung cho việc khác.

---

## Tổng kết nhóm distributed

| Case | Bản chất | Một câu phòng ngừa |
|---|---|---|
| Cascading | Coupling thời gian + không có băng cản lửa | Timeout + CB + bulkhead ở mọi cạnh sync; headroom cho failover |
| Split Brain | Timeout ≠ chết; promote không quorum; không fencing | Quorum + lease + fencing — đủ ba, không tự chế |
| Election Failure | Điều kiện hoạt động của consensus bị phá | Disk nhanh, timeout theo đo đạc, 3 AZ, giám sát elections/hour |

Cả ba đều minh họa một nguyên lý: **trong hệ phân tán, sự cố lớn hiếm khi do một thành phần hỏng nặng — chúng do các thành phần lành mạnh phản ứng sai với một trục trặc nhỏ.** Thiết kế chống sự cố phân tán = thiết kế *phản ứng* (timeout, retry, failover, election) cẩn thận hơn thiết kế *đường vui* (happy path).
