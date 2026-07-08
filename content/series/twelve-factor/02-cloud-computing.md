+++
title = "Chương 2: Cloud Computing — Khi hạ tầng trở thành phần mềm"
date = "2026-07-10T03:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> **Phần 1 – Foundation** | Chương trước: [Ứng dụng truyền thống](/series/twelve-factor/01-ung-dung-truyen-thong/) | Chương sau: [Cloud Native](/series/twelve-factor/03-cloud-native/)

Chương 1 kết thúc với một ứng dụng bị trói vào một máy chủ cụ thể. Chương này trả lời câu hỏi: **điều gì đã thay đổi ở tầng hạ tầng khiến cách viết ứng dụng cũ trở nên lỗi thời?**

---

## 1. Problem Statement

### Bài toán của hạ tầng truyền thống

Trước cloud, muốn có một server bạn phải: dự báo nhu cầu trước 6–12 tháng → xin ngân sách → mua phần cứng → chờ giao hàng → lắp đặt vào datacenter → cài OS → bàn giao. Chu kỳ tính bằng **tháng**.

Hệ quả của chu kỳ đó:

- **Over-provisioning là bắt buộc.** Bạn phải mua đủ máy cho ngày cao điểm nhất của năm (ví dụ: sale 11/11), nghĩa là 364 ngày còn lại phần lớn tài nguyên ngồi chơi. Tỷ lệ sử dụng CPU trung bình của datacenter doanh nghiệp truyền thống thường chỉ 10–20%.
- **Thử nghiệm đắt đỏ.** Muốn thử một ý tưởng cần server mới? Chờ 3 tháng. Kết quả: không ai thử nghiệm.
- **Capacity là quyết định một chiều.** Mua rồi thì không trả lại được.

### Cloud thay đổi điều gì?

Một dòng lệnh, và 60 giây sau bạn có một server. Không cần nữa? Xóa đi, ngừng trả tiền. Cần 100 server trong 2 giờ để chạy batch job? Có ngay, trả tiền đúng 2 giờ.

Sự thay đổi cốt lõi không phải là "server của người khác" — mà là ba tính chất:

1. **Elasticity**: tài nguyên co giãn theo nhu cầu, theo cả hai chiều.
2. **Pay-as-you-go**: chi phí biến đổi theo mức dùng, biến CAPEX thành OPEX.
3. **API-driven**: hạ tầng được điều khiển bằng API — nghĩa là **bằng code**.

Tính chất thứ ba là quan trọng nhất đối với người viết ứng dụng, và thường bị bỏ qua nhất.

---

## 2. Tại sao Cloud buộc ứng dụng phải thay đổi

### 2.1. Hạ tầng trở thành phần mềm → hạ tầng trở nên "tạm bợ" (ephemeral)

Khi server được tạo bằng một API call, nó cũng bị **xóa** bằng một API call. Nhà cung cấp cloud không hứa server của bạn sống mãi:

- VM có thể bị **reboot để bảo trì** host vật lý.
- Spot/Preemptible instance (rẻ hơn 60–90%) có thể bị **thu hồi với thông báo trước 30–120 giây**.
- Autoscaler **thêm và bớt** máy liên tục theo tải.

So sánh triết lý vận hành, thường được gọi là **Pets vs Cattle**:

```
       PETS (thú cưng)                    CATTLE (gia súc)
  ─────────────────────────         ─────────────────────────
  Server có tên riêng                Server có số hiệu
  (zeus, apollo, db-master)          (web-7f8d9, web-a1b2c)
  Ốm thì chữa                        Ốm thì thay
  Sống nhiều năm                     Sống vài giờ đến vài ngày
  Cấu hình tay, tích lũy             Sinh ra từ image, giống hệt nhau
  Chết là thảm họa                   Chết là chuyện thường ngày
```

Ứng dụng viết theo kiểu chương 1 — session trong RAM, file trên disk — là ứng dụng giả định server của nó là **pet**. Trên cloud, giả định đó sai từ gốc: **máy của bạn sẽ chết, và điều đó là bình thường**. Ứng dụng phải được thiết kế để việc một instance biến mất là non-event.

### 2.2. Mô hình chi phí đảo ngược logic tối ưu

On-premise: chi phí cố định → tối ưu nghĩa là *dùng hết* máy đã mua → khuyến khích nhồi nhiều thứ vào một máy (app + DB + cron trên cùng server, như chương 1).

Cloud: chi phí theo mức dùng → tối ưu nghĩa là *dùng đúng lúc, trả đúng thứ cần* → khuyến khích tách nhỏ, co giãn từng phần độc lập. Một ứng dụng không tách được thành các phần scale độc lập sẽ **trả tiền cloud theo kiểu on-premise**: thuê máy to nhất cho giờ cao điểm và để nó chạy 24/7. Đây là lý do rất nhiều doanh nghiệp "lên cloud" xong hóa đơn lại tăng — họ mang kiến trúc pet lên môi trường cattle.

### 2.3. Các tầng trừu tượng của cloud

```
 Mức độ bạn tự quản lý giảm dần ↓

 ┌──────────────┬──────────────┬──────────────┬──────────────┐
 │ On-premise   │ IaaS         │ PaaS / CaaS  │ FaaS         │
 │              │ (EC2, GCE)   │ (Heroku, EKS │ (Lambda,     │
 │              │              │  GKE, Cloud  │  Cloud       │
 │              │              │  Run)        │  Functions)  │
 ├──────────────┼──────────────┼──────────────┼──────────────┤
 │ Bạn quản lý: │ Bạn quản lý: │ Bạn quản lý: │ Bạn quản lý: │
 │ mọi thứ      │ OS trở lên   │ app + config │ chỉ code     │
 └──────────────┴──────────────┴──────────────┴──────────────┘
```

Chi tiết đáng nhớ cho chủ đề của tài liệu này: **12-Factor App được viết bởi các kỹ sư Heroku (2011)** — một PaaS. Họ quan sát hàng nghìn ứng dụng deploy lên nền tảng của mình và đúc kết: ứng dụng nào có những tính chất X, Y, Z thì deploy trơn tru, scale dễ, vận hành nhẹ nhàng; ứng dụng nào thiếu thì liên tục gặp sự cố. 12 factor chính là danh sách X, Y, Z đó. Nói cách khác: **12-Factor là "hợp đồng" giữa ứng dụng và một nền tảng tự động hóa** — bất kể nền tảng đó là Heroku 2011 hay Kubernetes 2026.

---

## 3. Bản chất: Từ "máy chủ" sang "tài nguyên tính toán"

Thay đổi tư duy quan trọng nhất của cloud có thể gói trong một câu:

> **Đơn vị tư duy không còn là "cái máy" mà là "một lượng compute + memory + storage được thuê trong một khoảng thời gian".**

Từ đó suy ra chuỗi hệ quả logic — hãy để ý mỗi hệ quả sẽ trở thành một hoặc nhiều factor ở các chương sau:

1. Máy là tạm bợ → **không được lưu gì quý giá trên máy** → *Factor 6: Processes (stateless)*, *Factor 4: Backing Services*.
2. Máy sinh ra tự động → **không có bàn tay người cài đặt** → app phải tự khai báo dependency → *Factor 2: Dependencies*.
3. Máy nào cũng như máy nào, chỉ khác môi trường → **cùng một artifact, config bơm từ ngoài vào** → *Factor 3: Config*, *Factor 5: Build-Release-Run*.
4. Máy đến và đi liên tục → **app phải khởi động nhanh, tắt êm** → *Factor 9: Disposability*.
5. Nhiều máy chạy song song → **scale bằng cách thêm process** → *Factor 8: Concurrency*.
6. Không SSH vào từng máy để đọc log → **log phải chảy ra ngoài** → *Factor 11: Logs*.

Đây là điểm mấu chốt của toàn bộ tài liệu: **12-Factor không phải 12 quy tắc rời rạc cần học thuộc. Nó là 12 hệ quả tất yếu của một tiền đề duy nhất: hạ tầng là tạm bợ và được tự động hóa.** Nắm được tiền đề, bạn có thể tự suy ra lại các factor — và quan trọng hơn, biết khi nào tiền đề không đúng (hệ thống của bạn không chạy trên hạ tầng tạm bợ) thì factor tương ứng không còn bắt buộc.

---

## 4. Trade-off của Cloud

Cloud không phải là bữa trưa miễn phí. Người thiết kế hệ thống cần nhìn rõ hai mặt:

**Chi phí ở quy mô lớn.** Pay-as-you-go rẻ khi tải biến động hoặc quy mô nhỏ. Với tải lớn và ổn định 24/7, on-premise hoặc thuê chỗ đặt máy (colocation) có thể rẻ hơn nhiều lần. Dropbox tiết kiệm ~75 triệu USD trong 2 năm khi rời AWS về hạ tầng riêng (2015–2017); 37signals/Basecamp công bố tiết kiệm hàng triệu USD/năm khi "cloud exit" (2023). Bài học không phải "cloud đắt" mà là: **elasticity chỉ có giá trị khi bạn thực sự cần elasticity**.

**Độ phức tạp vận hành đổi dạng chứ không biến mất.** Bạn không còn quản lý RAID và UPS, nhưng đổi lại phải quản lý IAM policy, VPC, security group, cost allocation, vendor lock-in. Kỹ năng đội ngũ phải thay đổi tương ứng.

**Latency và data gravity.** Dữ liệu lớn nằm ở đâu thì compute bị hút về đó. Egress fee (phí truyền dữ liệu ra khỏi cloud) là công cụ lock-in hiệu quả nhất của các nhà cung cấp.

| Tiêu chí | On-premise | Cloud |
|---|---|---|
| Thời gian có tài nguyên | Tuần–tháng | Giây–phút |
| Chi phí tải ổn định lớn | Thấp hơn | Cao hơn |
| Chi phí tải biến động | Rất cao (over-provision) | Tối ưu |
| Kỹ năng cần | Sysadmin, network, phần cứng | Cloud architecture, IaC, FinOps |
| Rủi ro chính | Capacity, disaster recovery | Cost sprawl, vendor lock-in |

---

## 5. Khi nào KHÔNG cần tư duy cloud

- **Hệ thống có yêu cầu chủ quyền dữ liệu / air-gapped** (ngân hàng lõi, quốc phòng): on-premise là ràng buộc, không phải lựa chọn.
- **Tải cực kỳ ổn định và lớn**: bài toán FinOps thuần túy, hãy tính toán thay vì chạy theo xu hướng.
- **Latency micro giây** (HFT, telco core): cloud công cộng không đáp ứng được.

Lưu ý: ngay cả khi chạy on-premise, các nguyên lý 12-Factor vẫn có giá trị — vì bạn vẫn có thể (và nên) xây hạ tầng on-premise theo kiểu cattle với ảo hóa + Kubernetes. Điều quyết định không phải là *cloud của ai* mà là *hạ tầng có được tự động hóa và coi là tạm bợ hay không*.

---

## Tóm tắt chương

- Cloud = elasticity + pay-as-you-go + **API-driven infrastructure**. Tính chất thứ ba biến hạ tầng thành phần mềm.
- Hạ tầng tự động hóa ⇒ máy chủ là **cattle, không phải pet** ⇒ máy của bạn sẽ chết và đó là chuyện bình thường.
- **12-Factor là hợp đồng giữa ứng dụng và nền tảng tự động hóa** — mọi factor đều suy ra được từ tiền đề "hạ tầng là tạm bợ".
- Cloud có trade-off thật (chi phí ở quy mô lớn, lock-in); tư duy cattle quan trọng hơn địa điểm đặt máy.

Chương tiếp theo: [Cloud Native](/series/twelve-factor/03-cloud-native/) — khi ứng dụng được thiết kế *cho* môi trường đó ngay từ đầu.
