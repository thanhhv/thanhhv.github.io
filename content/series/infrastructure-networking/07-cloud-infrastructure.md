+++
title = "Bài 7 — Cloud Infrastructure"
date = "2026-02-01T13:00:00+07:00"
draft = false
tags = ["backend", "infrastructure", "networking"]
series = ["Infrastructure & Networking"]
+++

# Module 07 — Cloud Infrastructure

> Cloud không phải "máy tính của người khác cho thuê". Nó là một mô hình kinh tế + vận hành khác: đổi CAPEX thành OPEX, đổi "quản lý phần cứng" thành "quản lý cấu hình và chi phí". Module này lấy AWS làm hệ quy chiếu; khái niệm ánh xạ 1-1 sang GCP/Azure.

---

## 1. VPC — mạng riêng ảo, nền của mọi thứ

### Problem Statement
Trong datacenter chung của cloud, khách hàng cần một mạng cách ly logic hoàn toàn: tự chọn dải IP, tự quyết định gì nói chuyện với gì, gì thấy Internet. VPC = software-defined network cung cấp đúng điều đó — mọi tài nguyên bạn tạo đều sống trong một VPC, mọi quyết định security/network đầu tiên là quyết định topology VPC.

### Các mảnh ghép và cách chúng khớp nhau

```
VPC 10.0.0.0/16
├── AZ-a
│   ├── Public subnet  10.0.0.0/24  → route 0.0.0.0/0 → Internet Gateway
│   │      [ALB, NAT Gateway]
│   ├── Private subnet 10.0.10.0/24 → route 0.0.0.0/0 → NAT Gateway
│   │      [app / EKS nodes]
│   └── Isolated subnet 10.0.20.0/24 → không có route ra ngoài
│          [RDS, ElastiCache]
├── AZ-b (lặp lại cấu trúc — đối xứng qua AZ)
└── AZ-c (…)
```

- **Subnet thuộc về đúng một AZ** — hệ quả: kiến trúc HA bắt đầu từ việc nhân đôi/ba subnet qua AZ, và dữ liệu qua lại AZ **tính tiền** (cross-AZ transfer ~$0.01–0.02/GB — nghe nhỏ, nhân với microservices chatty là hóa đơn nghìn đô; lý do tồn tại của topology-aware routing).
- **Route table quyết định "public" hay "private"** — không phải cái tên. Subnet public = có route tới **Internet Gateway** (IGW, dịch 1-1 public IP ↔ private IP). Private ra Internet chiều đi qua **NAT Gateway** (mọi bài học NAT ở module 01 áp dụng: stateful, idle timeout 350s, và **tính tiền theo GB xử lý** — NAT GW xử lý 10TB/tháng ≈ $450 chỉ phí xử lý; kéo image, gọi S3 qua NAT là lãng phí kinh điển → dùng **VPC endpoint/Gateway endpoint** cho S3/ECR, vừa rẻ vừa không ra Internet).
- **Security Group** = stateful firewall **gắn vào ENI** (instance), chỉ có rule allow, tự cho phép chiều về. **NACL** = stateless, gắn subnet, có deny. Thực dụng: làm mọi thứ bằng SG (theo nguyên tắc SG-tham-chiếu-SG: "app-sg cho phép từ alb-sg", không hardcode CIDR); NACL để mặc định trừ khi cần chặn thô một dải IP.
- Nối các VPC: **peering** (đơn giản, không transitive — 10 VPC = 45 kết nối) vs **Transit Gateway** (hub-and-spoke, transitive, trả tiền theo GB — chuẩn cho tổ chức nhiều VPC/account). Bài học module 01: **quy hoạch CIDR không trùng ngay từ đầu**, vì sửa sau là dự án nhiều quý.

### Anti-patterns VPC
- Mọi thứ trong public subnet với public IP "cho dễ SSH" → thay bằng SSM Session Manager/bastion, private subnet mặc định.
- Một VPC khổng lồ dùng chung dev/staging/prod → blast radius và IAM rối; tách account/VPC theo môi trường.
- Mở SG `0.0.0.0/0:22` — máy quét Internet tìm thấy trong vài phút.

---

## 2. Load Balancer và CDN trên cloud

Áp dụng toàn bộ module 02, thêm đặc thù cloud:
- **NLB (L4)**: pass-through, latency thấp, static IP theo AZ, bảo toàn source IP; không retry hộ, không HTTP metric. **ALB (L7)**: routing theo host/path/header, WAF gắn được, target theo IP (nối được EKS pod trực tiếp). Chuỗi phổ biến: CloudFront (edge) → ALB → target.
- LB cloud **scale theo traffic nhưng không tức thời** — traffic tăng đột ngột 10× trong 1 phút có thể vượt tốc độ scale của chính LB (AWS có pre-warming qua support cho sự kiện lớn đã biết).
- Cross-zone load balancing: bật cho đều tải, hiểu rằng nó tạo cross-AZ traffic (tiền + latency ~1ms).

---

## 3. Managed Services — quyết định build vs buy quan trọng nhất

### Khung tư duy
Câu hỏi sai: "RDS đắt gấp đôi EC2 tự cài Postgres?" Câu hỏi đúng: "**Tổng chi phí sở hữu** — gồm kỹ sư trực failover 3h sáng, thiết kế backup/restore ĐÃ KIỂM CHỨNG, vá bảo mật, nâng cấp version — bên nào thấp hơn, và vận hành database có phải lợi thế cạnh tranh của công ty tôi không?"

Nguyên tắc: **mặc định managed cho mọi thứ stateful** (RDS/Aurora, ElastiCache, MSK, S3); tự vận hành chỉ khi (1) quy mô đủ lớn để chênh lệch giá nuôi được team chuyên trách, hoặc (2) cần khả năng managed không có (extension đặc thù, kernel tuning), hoặc (3) chiến lược thoát vendor có giá trị thật với doanh nghiệp — đa số team đánh giá quá cao (3) và đánh giá quá thấp chi phí vận hành stateful.

Cái giá thật của managed: trần cấu hình (không sờ được OS), phiên bản theo lịch của vendor, chi phí biên cao ở quy mô rất lớn, và lock-in ở mức **API + operational knowledge** (thứ khó thoát hơn cả code).

---

## 4. Autoscaling trên cloud

Áp dụng phần autoscaling module 05, thêm tầng hạ tầng:
- **Scale theo metric nào**: CPU là proxy dở cho nhiều workload (I/O-bound service nghẽn ở connection pool khi CPU 30%). Metric tốt hơn: request/target (ALB), queue depth (worker), custom SLI. Quy tắc: scale theo **thứ thực sự cạn trước**.
- **Scale-out nhanh, scale-in chậm** (cooldown dài chiều xuống): thà thừa vài phút còn hơn flapping — mỗi lần scale-in giết nhầm capacity là một lần latency spike.
- **Spot/preemptible** (rẻ 60–90%, bị đòi lại với thông báo 30–120s): tuyệt cho stateless/batch với điều kiện đã làm đúng graceful shutdown (module 03) + PDB + đa dạng instance type (giảm xác suất bị đòi đồng loạt). Đừng đặt stateful/singleton lên spot.
- **Trần khu vực có thật**: account limit, AZ hết loại instance (đặc biệt GPU). Sự kiện lớn: reserve trước (On-Demand Capacity Reservation), đừng tin "cloud vô hạn".

---

## 5. Multi-region — đắt gấp 10, cần thiết cho 1% hệ thống

### Sự thật cần nói trước
Multi-AZ (3 AZ trong 1 region) đã cho ~99.99% với chi phí và độ phức tạp chấp nhận được, chống được hỏng datacenter đơn lẻ. **Multi-region chống được sự kiện hiếm hơn nhiều** (cả region sập — vài lần/thập kỷ) với chi phí: nhân đôi hạ tầng + **bài toán dữ liệu khó gấp bội** + độ phức tạp vận hành thường xuyên gây ra nhiều outage hơn là ngăn được. Làm multi-region vì compliance/latency user toàn cầu/RTO cực gắt có thật — không phải vì nó "nghe pro".

### Ba mô hình
| Mô hình | Cách chạy | RTO/RPO | Cái giá |
|---|---|---|---|
| Backup & restore | Backup sang region khác | Giờ / giờ | Rẻ nhất; restore PHẢI được diễn tập |
| Warm standby / pilot light | Region phụ chạy tối thiểu, DB replica async | Phút–chục phút / giây–phút | Nhân hạ tầng một phần; nguy cơ region phụ "mục" vì không dùng |
| Active-active | Cả hai nhận traffic | ~0 / ~0 với dữ liệu conflict-free | Khó nhất: dữ liệu ghi 2 nơi = conflict resolution hoặc phân vùng user theo region; chi phí kỹ sư lớn |

Bài toán trung tâm luôn là **dữ liệu**: replication đồng bộ qua region = +50–200ms mỗi lần ghi (tốc độ ánh sáng — không mua được); async = RPO > 0 (mất giao dịch cuối khi failover). Không có lựa chọn thứ ba, chỉ có chọn theo từng loại dữ liệu. Và **failover không được diễn tập = failover không tồn tại**: DNS TTL bị cache (module 01), capacity region phụ không đủ khi cả thiên hạ cùng failover sang, runbook mục. Route 53/Traffic Manager health-check failover chỉ là mảnh dễ nhất của bức tranh.

---

## 6. Cost Optimization — kỹ năng kiến trúc, không phải kế toán

Cấu trúc hóa đơn điển hình và đòn bẩy lớn nhất:
1. **Compute** (~40–60%): right-size trước (đo thực dùng — VPA/CloudWatch, đa số fleet thừa 2–5×), rồi Savings Plans/Reserved cho baseline (rẻ ~40–60%), spot cho phần chịu chết, tắt môi trường non-prod ngoài giờ (giảm ~65% cho phần đó).
2. **Data transfer** (~10–20%, và là phần gây bất ngờ): cross-AZ chatty microservices, NAT GW processing, egress ra Internet ($0.09/GB — 100TB/tháng = $9k). Đòn: VPC endpoint, topology-aware routing, CDN cho egress, nén.
3. **Storage**: lifecycle policy S3 (IA/Glacier), xóa EBS snapshot/volume mồ côi, gp3 thay gp2.
4. **Observability** (âm thầm phình to — module 08): log ingest là thủ phạm quen mặt.

Nguyên tắc tổ chức: tag bắt buộc (team/env/service) từ ngày đầu, budget alert theo team, cost review là nghi thức kỹ thuật hàng tháng — chi phí là một **thuộc tính kiến trúc** ngang với latency, được quyết ở bản thiết kế chứ không ở cuối quý.
