+++
title = "Chương 23 — Chủ đề Principal: Multi-region, Platform Engineering, GitOps nâng cao, Progressive Delivery, Multi-tenant, Cost"
date = "2026-07-11T00:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> **Phần 4 – Principal** | Chương trước: [Observability](/series/twelve-factor/22-observability/) | Chương sau: [Case study REST API](/series/twelve-factor/24-case-study-rest-api/)

Đến đây, các nguyên lý đã đầy đủ. Chương này trả lời câu hỏi của tầng principal: **khi hệ thống và tổ chức lớn lên 10–100 lần, các nguyên lý đó biến hình thế nào?** Mỗi mục đi nhanh nhưng đủ khung quyết định.

---

## 1. Multi-region Deployment

**Bài toán.** Một region là một miền sự cố (AWS us-east-1 đã nhiều lần chứng minh); latency vật lý (Việt Nam → Singapore ~30–40ms, → US ~200ms); và yêu cầu pháp lý về nơi lưu dữ liệu (data residency).

**Điểm mấu chốt: app 12-Factor gần như miễn phí cho multi-region — dữ liệu mới là bài toán thật.** Vì app stateless (F6), config từ môi trường (F3), artifact bất biến (F5): deploy thêm một region = chạy cùng image với bộ config region đó. Toàn bộ độ khó dồn về backing services:

```
  Mô hình         App tier                DB                        Độ khó
  ─────────────  ──────────────────────  ────────────────────────  ─────────
  Active-passive  chạy cả 2, traffic 1    primary + replica khác    Thấp-vừa
                                          region, failover có RTO/RPO
  Active-active   chạy và nhận traffic    (a) mỗi region một DB     Cao
  (đích thật)     cả 2 (GeoDNS/anycast)      theo phân vùng user
                                          (b) DB phân tán toàn cầu
                                             (Spanner, Cockroach,
                                              DynamoDB global)
```

Khung quyết định: bắt đầu bằng **câu hỏi RPO/RTO của business** (mất bao nhiêu phút dữ liệu/downtime là chấp nhận được, giá bao nhiêu?) — không phải bằng công nghệ. Đa số hệ thống Việt Nam phục vụ user Việt Nam: một region + backup cross-region + CDN là điểm cân bằng đúng; active-active toàn cầu là bài của công ty toàn cầu, với cái giá là đối mặt CAP theorem ở mọi dòng code (conflict resolution, clock skew, read-your-writes). Đừng trả giá đó khi chưa ai bắt.

**Anti-pattern**: "multi-region" chỉ ở tầng app còn DB một region — thêm 200ms mỗi query cho region xa, được sự phức tạp mà không được resilience; sự cố region DB vẫn sập tất.

## 2. Platform Engineering

**Bài toán.** Mọi chương trước cộng lại là một *khối lượng tri thức khổng lồ*: probe, PDB, GitOps, OTel, secret, pipeline... Bắt 20 team product mỗi team tự làm chủ toàn bộ là bất khả — kết quả thực tế là 20 phiên bản chắp vá khác nhau (cognitive overload chính là lý do phong trào "DevOps mỗi team tự lo" thoái trào).

**Ý tưởng.** Một team platform xây **Internal Developer Platform (IDP)**: đóng gói mọi best practice thành **golden path** — con đường trải sẵn mà team product chỉ cần khai báo phần *khác biệt* của mình:

```yaml
# Team product chỉ viết chừng này (ví dụ kiểu Score/Humanitec hay template nội bộ):
app: order-service
team: checkout
port: 8080
env:
  DATABASE_URL: { fromSecret: order-db }
resources: { size: small }        # platform dịch ra requests/limits chuẩn
# → platform tự sinh: Deployment đủ probe/PDB/securityContext, pipeline,
#   dashboard RED, alert SLO, log routing, GitOps app — TẤT CẢ các chương trước
```

Nguyên tắc thành công quan trọng nhất: **platform là sản phẩm, team product là khách hàng** — golden path phải *dễ hơn* làm tay (không thì bị bỏ qua), và là đường trải thảm chứ không phải chuồng (cho phép "mở nắp" khi cần, với trách nhiệm đi kèm). 12-Factor chính là *hợp đồng interface* giữa hai bên: app tuân 12-Factor thì platform phục vụ được bằng máy; app lệch chuẩn là ticket thủ công vĩnh viễn.

**Khi nào chưa cần**: dưới ~15–20 engineer, "platform" là một bộ template + tài liệu tốt; dựng team platform riêng quá sớm là tự tạo tầng quan liêu.

## 3. GitOps nâng cao & Progressive Delivery

Chương 21 dựng GitOps cơ bản. Ở quy mô lớn, mảnh còn thiếu là: **deploy xong không có nghĩa là an toàn** — staging không bao giờ bằng production (F10 đã thừa nhận), nên câu trả lời trưởng thành là *đưa thay đổi ra production một cách từ từ, có đo lường, tự đảo ngược*:

```
  Rolling update (ch20):  thay dần pod — nhưng 100% traffic vẫn đổ vào bản mới ngay
  Blue/Green:             hai môi trường đầy đủ, chuyển traffic một nhát, quay lại một nhát
                          (đắt gấp đôi lúc deploy; DB schema vẫn phải expand-contract)
  Canary:                 bản mới nhận 1% → 5% → 25% → 100% traffic;
                          MỖI BƯỚC đối chiếu metrics (error rate, p99) với baseline;
                          xấu → TỰ ĐỘNG rollback. Argo Rollouts / Flagger.
```

```yaml
# Argo Rollouts (trích ý tưởng): canary có phân tích tự động
strategy:
  canary:
    steps:
      - setWeight: 5
      - pause: { duration: 5m }
      - analysis:                     # đọc Prometheus: error rate bản canary
          templates: [{ templateName: success-rate }]
      - setWeight: 25
      # ...
# → deploy trở thành THÍ NGHIỆM CÓ KIỂM SOÁT: blast radius 5% thay vì 100%
```

Đây là điểm hội tụ đẹp nhất của cả tài liệu: canary chỉ khả thi khi có **F5** (hai release chạy song song), **F6** (bản mới/cũ thay nhau vô hại), **F9** (chuyển đổi êm), **observability** (chương 22 — không có metrics thì "phân tích canary" bằng gì?). Progressive delivery là phần thưởng cuối cùng cho việc giữ mọi hợp đồng trước đó.

Cùng họ: **feature flags** — tách *deploy* (đưa code lên) khỏi *release* (bật tính năng cho user); flag theo tenant/percentage; và kill-switch tắt tính năng hỏng trong giây thay vì rollback deploy. Kỷ luật đi kèm: flag có ngày hết hạn, dọn flag chết như dọn nợ.

## 4. Multi-tenant Architecture

**Bài toán** (đã hẹn từ chương 6): một hệ thống phục vụ nhiều khách hàng (tenant) — SaaS B2B điển hình — mà **không fork codebase**.

Trục quyết định là **mức cô lập**, gần như luôn quy về dữ liệu:

```
  Pool (chung hết)          Bridge (lai)              Silo (riêng hết)
  ──────────────────        ──────────────────        ──────────────────
  1 DB, mọi bảng có         schema/database riêng     stack riêng mỗi tenant
  tenant_id                 mỗi tenant, app chung     (namespace/cluster riêng)
  Rẻ nhất, vận hành 1 hệ    Cô lập khá, migrate N lần Cô lập tuyệt đối, đắt,
  Rủi ro: 1 câu WHERE       Vận hành trung bình       vận hành = số tenant
  thiếu tenant_id = thảm                              (nhưng: 12-Factor + GitOps
  họa rò dữ liệu chéo                                  làm "N stack" khả thi!)
```

Khuyến nghị mặc định cho SaaS: **pool + hàng rào nhiều lớp** — `tenant_id` NOT NULL mọi bảng, Postgres **Row-Level Security** làm lưới an toàn cuối (quên WHERE vẫn không rò), tenant context đi trong `context.Context` từ middleware auth xuống repository (đường ống của chương 18 lại trả cổ tức), test tự động thử truy cập chéo tenant. Tenant lớn/nhạy cảm trả thêm tiền → nâng lên bridge/silo — **cùng một image** (F5), khác config (F3): multi-tenant chính là "một codebase, nhiều deploy" (F1) ở cấp độ sản phẩm.

Đừng quên cô lập **hiệu năng**: rate limit theo tenant, quota queue theo tenant — một tenant chạy báo cáo nặng không được làm 99 tenant còn lại chậm (noisy neighbor).

## 5. Cost Optimization (FinOps)

Chuỗi logic của cả tài liệu có một hệ quả tài chính: **12-Factor làm chi phí trở nên *điều khiển được*** — app stateless scale xuống được (kể cả về 0), artifact bất biến chạy trên spot instance không sợ (F9 chịu được thu hồi), tách process type trả tiền đúng phần nóng (F8). Ứng dụng pet không cho bạn nút vặn nào.

Thứ tự đòn bẩy theo tỷ lệ giá trị/công sức (kinh nghiệm phổ biến):

1. **Right-sizing**: requests đặt theo đo đạc (VPA recommend) thay vì copy-paste — cluster điển hình lãng phí 50–70% tài nguyên đã cấp phát; đây gần như luôn là khoản lớn nhất.
2. **Autoscale xuống**: HPA min thấp ngoài giờ, scale-to-zero cho môi trường dev/staging ban đêm (một CronJob chỉnh replicas — tiết kiệm ~60% giờ chạy của non-prod).
3. **Spot/preemptible cho workload disposable**: worker, CI runner, stateless web dư HA — rẻ hơn 60–90%; điều kiện *chính xác* là F9 (chịu được thu hồi 30–120s) — một lần nữa, factor cũ mở khóa tiền mới.
4. **Commitment (Reserved/Savings Plans)** cho phần baseline ổn định — sau khi đã right-size (commit trên mức lãng phí là khóa chặt lãng phí).
5. **Kiến trúc**: egress fee giữa AZ/region (thiết kế data locality), log/telemetry retention (chương 22), NAT gateway phí ẩn kinh điển.

Nguyên tắc tổ chức: cost là **metric kỹ thuật** như latency — gắn label/namespace để phân bổ theo team (showback), đưa vào review định kỳ; không phải việc riêng của phòng tài chính mỗi cuối quý.

---

## Tóm tắt

- **Multi-region**: app 12-Factor miễn phí, dữ liệu trả giá — quyết định bằng RPO/RTO của business; đa số chưa cần active-active.
- **Platform engineering**: đóng gói toàn bộ tài liệu này thành golden path; 12-Factor là hợp đồng interface giữa platform và product team; platform là sản phẩm, không phải cảnh sát.
- **Progressive delivery**: canary + phân tích tự động biến deploy thành thí nghiệm blast-radius nhỏ — phần thưởng hội tụ của F5+F6+F9+observability; feature flags tách deploy khỏi release.
- **Multi-tenant**: một codebase mọi tenant (F1 cấp sản phẩm); pool + RLS + tenant context là mặc định, nâng cấp cô lập theo giá trị hợp đồng.
- **FinOps**: 12-Factor biến chi phí thành thứ điều khiển được; thứ tự đòn bẩy: right-size → autoscale xuống → spot → commitment → kiến trúc.
