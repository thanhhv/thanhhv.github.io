+++
title = "Phần 3 — Availability & Reliability"
date = "2026-07-13T06:50:00+07:00"
draft = false
tags = ["backend", "system-design"]
series = ["System Design — Tư Duy Thiết Kế Hệ Thống"]
+++

> Chủ đề: High Availability, Failover, Replication (phối hợp), Backup, Disaster Recovery, Active-Active, Active-Passive.

## Luận điểm trung tâm của phần này

Availability là **kết quả của thiết kế cho sự cố**, không phải của phần cứng tốt: mọi thành phần sẽ hỏng — câu hỏi duy nhất là hệ thống phản ứng thế nào khi điều đó xảy ra. Ba mệnh đề nền:

1. **Mỗi "số 9" thêm đắt ~10 lần** ([1.2](/series/system-design/01-foundations/02-sla-slo-sli/)); và từ 99.99% trở lên, con người không kịp phản ứng — mọi failover phải tự động, kéo theo toàn bộ [Phần 4](/series/system-design/04-distributed-systems/03-consensus-quorum-leader-election/). Ranh giới 3 số 9 / 4 số 9 là ranh giới kiến trúc, không phải ranh giới nỗ lực.
2. **Redundancy chỉ là tiềm năng — failover mới là năng lực.** N bản sao không cứu được ai nếu quá trình chuyển đổi sai ([13.4 — split brain](/series/system-design/13-production-failure-cases/04-distributed-failures/)); và failover chưa từng diễn tập là failover trên giấy.
3. **Replication ≠ Backup ≠ DR** — ba lớp chống ba loại chết khác nhau (máy hỏng / dữ liệu hỏng / tổ chức không biết làm gì); cần cả ba, thiếu lớp nào lộ lớp đó đúng ngày xấu trời ([12.10](/series/system-design/12-evolution/10-disaster-recovery/)).

## Mục lục

- [3.1. High Availability & Failover — giải phẫu quá trình chuyển đổi](/series/system-design/03-availability-reliability/01-ha-failover/)
- [3.2. Backup & Recovery — lớp phòng thủ cuối cùng](/series/system-design/03-availability-reliability/02-backup-recovery/)
- [3.3. Active-Active vs Active-Passive — hai triết lý redundancy](/series/system-design/03-availability-reliability/03-active-active-passive/)

## Bản đồ chủ đề trong toàn tài liệu

| Mảnh | Ở đâu |
|---|---|
| Định nghĩa và chi phí các "số 9", error budget | [1.2](/series/system-design/01-foundations/02-sla-slo-sli/) |
| Replication chi tiết (sync/async, mô hình) | [4.2](/series/system-design/04-distributed-systems/02-replication-consistency/) |
| Quorum, leader election, fencing — bộ máy failover an toàn | [4.3](/series/system-design/04-distributed-systems/03-consensus-quorum-leader-election/), [4.4](/series/system-design/04-distributed-systems/04-clock-partition-split-brain/) |
| DR như năng lực tổ chức: RPO/RTO, 4 lớp, diễn tập | [12.10](/series/system-design/12-evolution/10-disaster-recovery/) |
| Multi-region | [12.9](/series/system-design/12-evolution/09-multi-region/) |
| Các sự cố ăn mòn availability | [Phần 13](/series/system-design/13-production-failure-cases/00-tong-quan/) toàn bộ |

Phần này là mô-đun nối: đi sâu vào những gì các phần trên chưa phủ — giải phẫu failover, kỹ nghệ backup, và lựa chọn active-active/passive như một khung quyết định.
