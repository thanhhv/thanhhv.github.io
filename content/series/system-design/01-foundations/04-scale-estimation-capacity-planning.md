+++
title = "1.4. Scale Estimation & Capacity Planning"
date = "2026-07-13T05:50:00+07:00"
draft = false
tags = ["backend", "system-design"]
series = ["System Design — Tư Duy Thiết Kế Hệ Thống"]
+++

## 1. Problem Statement

Hai câu hỏi mà mọi thiết kế phải trả lời bằng **con số**, không phải bằng tính từ:

1. **Scale Estimation:** hệ thống phải chịu bao nhiêu — user, request, dữ liệu?
2. **Capacity Planning:** cần bao nhiêu tài nguyên — máy, RAM, disk, băng thông — để chịu mức đó, hôm nay và 12 tháng tới?

Không có hai con số này, mọi tranh luận kiến trúc ("có cần sharding không?", "Postgres chịu nổi không?") là tranh luận cảm tính. Với con số, phần lớn tranh luận tự biến mất — vì đáp án trở nên hiển nhiên.

## 2. Kỹ thuật ước lượng — back-of-the-envelope có kỷ luật

Mục tiêu là **đúng bậc độ lớn** (order of magnitude), không phải đúng từng phần trăm. Sai 2× vô hại; sai 100× làm thiết kế vô nghĩa.

### 2.1. Các quy đổi nên thuộc lòng

- 1 ngày ≈ 86,400 giây ≈ **10⁵ giây** (làm tròn để chia nhẩm).
- 1 triệu request/ngày ≈ **12 RPS** trung bình.
- Peak thường gấp **2–5×** trung bình (hệ thống tiêu dùng VN: peak tối 20–22h; e-commerce có flash sale: 10–50×).
- 1 ký tự ≈ 1–4 byte (UTF-8); 1 row DB điển hình 0.5–2KB; 1 ảnh nén 100KB–1MB; 1 phút video 720p ≈ 30–50MB.
- Mỗi bậc của phễu: 10M user đăng ký → ~1M MAU → ~100–300K DAU → mỗi DAU 10–50 request/ngày.

### 2.2. Quy trình 5 bước

Ví dụ xuyên suốt: **sàn e-commerce Việt Nam, mục tiêu 2M user đăng ký sau 18 tháng.**

**Bước 1 — Từ business ra DAU:** 2M đăng ký → giả định 15% hoạt động ngày thường → 300K DAU. *Ghi rõ giả định 15% để kiểm chứng lại sau.*

**Bước 2 — Từ DAU ra RPS:**

- Mỗi DAU: ~20 lượt xem trang, mỗi trang ~5 API call → 100 request/user/ngày.
- Tổng: 300K × 100 = 30M request/ngày ≈ **350 RPS trung bình → ~1,200 RPS peak** (×3.5, tối).
- Flash sale 12.12: ước ×10 trung bình = **3,500+ RPS**, dồn vào một nhóm endpoint hẹp.

**Bước 3 — Tách đọc/ghi:** tỷ lệ đọc:ghi e-commerce điển hình 30:1 – 100:1. Với 30:1 → ghi ~12 RPS avg, 40 RPS peak. Kết luận lập tức: **bottleneck là đọc, không phải ghi** → cache + read replica là hướng chính, chưa cần nghĩ đến sharding ghi. Một phép chia vừa loại bỏ cả một nhánh kiến trúc.

**Bước 4 — Dữ liệu:**

- Đơn hàng: 300K DAU × 2% conversion × 1 đơn = 6K đơn/ngày × 2KB ≈ 12MB/ngày ≈ **4.4GB/năm** — bé không đáng kể.
- Sản phẩm: 500K SKU × 5KB ≈ 2.5GB. Ảnh: 500K × 4 ảnh × 300KB ≈ **600GB** → nằm ở object storage + CDN, không nằm trong DB.
- Log/event: 30M request/ngày × 1KB ≈ 30GB/ngày ≈ **11TB/năm** — đây mới là con số lớn nhất hệ thống! → cần retention policy và cold storage ngay từ đầu.

**Bước 5 — Băng thông:** 1,200 RPS × 20KB response ≈ 24MB/s ≈ 200Mbps (chưa kể ảnh — ảnh đi CDN). Trong tầm một cụm server bình thường.

### 2.3. Đọc kết quả

Bảng ước lượng trên nói: hệ thống này ở quy mô **1 RDBMS tốt + cache + CDN + read replica**. Không cần NoSQL, không cần sharding, không cần Kafka cho đường chính. Điểm cần thiết kế đặc biệt duy nhất: **flash sale** (spike ×10 vào endpoint tồn kho — bài toán tranh chấp, giải bằng queue + cache, xem [Phần 12](/series/system-design/12-evolution/00-tong-quan/)) và **pipeline log** (11TB/năm).

Đây chính là giá trị của estimation: nó **thu hẹp không gian thiết kế** trước khi vẽ bất kỳ diagram nào.

## 3. Capacity Planning — từ RPS ra số máy

### 3.1. Đo capacity thực của một đơn vị

Capacity không tính được từ spec sheet — phải **đo bằng load test**: tăng tải dần đến khi p99 latency vượt SLO → RPS tại điểm đó là capacity thực của 1 instance.

Ví dụ đo được: 1 instance app (4 vCPU) chịu 400 RPS ở p99 = 180ms; đến 550 RPS thì p99 gãy lên 2s. → Capacity danh định: 400 RPS/instance.

### 3.2. Công thức số máy

```
N = ceil( Peak_RPS × Hệ_số_an_toàn / Capacity_1_instance ) + Đệm_failover
```

- Hệ số an toàn 1.25–1.5 (ước lượng luôn lạc quan).
- Đệm failover: quy tắc **N+1** (hoặc N+2 với hệ thống quan trọng) — cụm phải chịu được tải khi 1 máy chết hoặc đang deploy.

Với ví dụ trên: peak 1,200 RPS × 1.4 / 400 = 4.2 → 5 instance + 1 = **6 instance**. Flash sale 3,500 RPS → cần 13–14 instance → đây là lúc autoscaling hoặc pre-scaling theo lịch có giá trị.

### 3.3. Capacity của tầng dữ liệu — phần khó nhất

App instance thêm dễ (stateless). DB không thế. Vài mốc tham khảo cho single-node PostgreSQL/MySQL trên phần cứng tốt (NVMe, đủ RAM cho working set):

- Đọc điểm (point query có index, cache nóng): **hàng chục nghìn QPS**.
- Ghi transactional: **hàng nghìn đến ~vài chục nghìn TPS** tùy độ phức tạp và cấu hình durability.
- Kích thước làm việc thoải mái: **hàng trăm GB đến vài TB**; vượt xa RAM thì mọi thứ xấu dần.

*Các con số này dao động lớn theo schema, index và tuning — luôn benchmark với workload thật của bạn.* Điểm mấu chốt: RPS ước lượng của bạn (40 write/s, vài nghìn read/s sau cache) so với các mốc này cho biết bạn còn cách trần single-node bao xa — thường là **xa hơn nhiều người nghĩ**.

## 4. First Principles

**Vì sao ước lượng trước, thiết kế sau?** Vì không gian giải pháp phụ thuộc phi tuyến vào quy mô. 100 RPS và 100K RPS không phải cùng bài toán khó gấp 1000 lần — chúng là hai bài toán khác nhau về bản chất. Thiết kế trước khi ước lượng nghĩa là chọn ngẫu nhiên một trong hai bài toán để giải.

**Nếu bỏ qua capacity planning thì sao?** Hệ thống sẽ được cấp tài nguyên theo một trong hai cách đều tệ: theo cảm giác an toàn (over-provision — đốt tiền), hoặc theo sự cố (under-provision — user chịu latency gãy cho đến khi đủ đau để nâng cấp).

**Giả định phải kiểm chứng:** mọi ước lượng đứng trên giả định (15% DAU, 30:1 đọc/ghi, peak ×3.5). Viết giả định ra, đo lại sau khi vận hành 1–3 tháng, sửa mô hình. Ước lượng là vòng lặp, không phải nghi thức làm một lần.

## 5. Trade-off

| Chiến lược | Được | Mất |
|---|---|---|
| Over-provision (mua dư) | An toàn, đơn giản, đêm ngủ ngon | Đốt tiền; utilization 15% là bình thường ở các cty over-provision |
| Just-in-time + autoscaling | Chi phí sát nhu cầu | Phức tạp; scale-out mất 1–5 phút — không kịp với spike dựng đứng; cần warm pool cho sự kiện biết trước |
| Scale theo dự báo (pre-scale lịch flash sale) | Rẻ + kịp spike | Cần kỷ luật quy trình; dự báo sai vẫn chết |

Thực tế tốt nhất là **kết hợp**: baseline over-provision nhẹ (chạy ≤ 60–70% peak thường nhật — xem đường cong utilization ở [chương 1.3](/series/system-design/01-foundations/03-throughput-latency/)), autoscaling cho dao động ngày, pre-scale cho sự kiện biết trước.

## 6. Production Considerations

- Dashboard capacity: utilization từng tầng (app CPU, DB CPU/IOPS/connections, cache memory, queue depth) so với trần đã đo bằng load test.
- Alert ở **70% trần** — đủ sớm để hành động (mua máy, tối ưu, sharding) trước khi user cảm nhận.
- Load test lại **mỗi quý** và sau mỗi thay đổi lớn: capacity là con số sống, một index bị xóa nhầm có thể cắt nó đi 5 lần.
- Theo dõi **tăng trưởng dữ liệu** riêng: DB 5TB không thể restore trong 30 phút — capacity planning bao gồm cả thời gian backup/restore (gắn với RTO, xem [giai đoạn 10, Phần 12](/series/system-design/12-evolution/10-disaster-recovery/)).
- Ghi capacity model vào tài liệu vận hành: người trực cần biết "trần của chúng ta ở đâu" khi traffic tăng bất thường.

## 7. Best Practices

- Ước lượng theo **luồng nghiệp vụ**, không theo hệ thống gộp: RPS của "xem sản phẩm" và "thanh toán" cần độ chính xác khác nhau.
- Làm 3 kịch bản: kỳ vọng / ×3 / ÷3. Thiết kế cho kịch bản kỳ vọng, kiểm tra rằng kịch bản ×3 không đòi viết lại (chỉ đòi thêm máy).
- Khi trình bày thiết kế, trình bày ước lượng trước — nó là phần thuyết phục nhất của mọi design review.
- Đừng quên các trần "mềm" hay bị bỏ sót: số connection DB, file descriptor, băng thông NAT gateway, quota của cloud provider, giới hạn API bên thứ ba.

## 8. Anti-patterns

- **Thiết kế cho quy mô tưởng tượng:** "lỡ viral thì sao" → sharding từ ngày đầu cho 500 user. Chi phí cơ hội khổng lồ; viral có xác suất thấp và cache + read replica mua đủ thời gian tái thiết kế.
- **Ước lượng một lần rồi đóng khung:** mô hình 18 tháng trước vẫn được trích dẫn khi thực tế đã lệch 5×.
- **Capacity theo trung bình:** hệ thống chết vào peak, không chết vào trung bình.
- **Chỉ đếm happy path:** quên retry (nhân traffic khi có sự cố), quên bot/crawler (có thể là 30–50% traffic), quên chính hệ thống giám sát.

## 9. Khi nào KHÔNG cần

Không bao giờ được bỏ hẳn — nhưng độ sâu co giãn theo giai đoạn. MVP: 30 phút ước lượng nháp để chắc rằng "1 con RDS vừa là đủ" là toàn bộ capacity planning cần thiết. Nghi thức hoá nó thành spreadsheet 20 tab cho sản phẩm chưa có user là lãng phí — nhưng 30 phút đó vẫn bắt buộc, vì nó có thể tiết kiệm 6 tháng xây nhầm hệ thống.

---

*Tiếp theo: [1.5. Bottleneck Analysis](/series/system-design/01-foundations/05-bottleneck-analysis/)*
