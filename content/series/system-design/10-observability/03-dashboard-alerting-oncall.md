+++
title = "10.3. Dashboard, Alerting & On-call — từ tín hiệu đến con người"
date = "2026-07-13T13:30:00+07:00"
draft = false
tags = ["backend", "system-design"]
series = ["System Design — Tư Duy Thiết Kế Hệ Thống"]
+++

## 1. Problem Statement

Tín hiệu hoàn hảo ([10.1](/series/system-design/10-observability/01-ba-tru/)) chảy qua pipeline chuẩn ([10.2](/series/system-design/10-observability/02-opentelemetry-pipeline/)) vẫn chưa cứu được ai — khâu cuối là **con người ra quyết định dưới áp lực lúc 3h sáng**. Khâu này hỏng theo hai cách đối xứng: alert quá nhiều (người trực tê liệt, tắt thông báo, bỏ lỡ cái thật — *alert fatigue* là nguyên nhân gốc của nhiều sự cố kéo dài hơn là thiếu monitoring) và alert quá ít/quá trễ (khách hàng là người phát hiện sự cố). Thiết kế giao diện máy–người này là một bài thiết kế hệ thống đúng nghĩa — có SLO, có failure mode, có evolution.

## 2. Alerting — triết lý trước, ngưỡng sau

**Nguyên tắc nền: alert trên triệu chứng user cảm nhận; điều tra bằng nguyên nhân** ([1.2 §7](/series/system-design/01-foundations/02-sla-slo-sli/)). "CPU 85%" không phải sự cố nếu user vẫn vui ([1.3 — utilization cao có chủ đích](/series/system-design/01-foundations/03-throughput-latency/)); "checkout error 2%" là sự cố bất kể CPU bao nhiêu. Alert theo nguyên nhân sinh ra hàng trăm rule mãi không đủ; alert theo triệu chứng chỉ cần vài rule cho mỗi luồng nghiệp vụ — và **burn rate đa cửa sổ** ([1.2 §4](/series/system-design/01-foundations/02-sla-slo-sli/)) là dạng chín nhất: nhanh với cháy lớn, nhạy với rò rỉ chậm, im với nhiễu thoáng qua.

**Phân tầng phản ứng — mỗi alert thuộc đúng một tầng:**

| Tầng | Tiêu chí | Đích |
|---|---|---|
| **Page** (đánh thức) | User đang đau *và* cần người trong phút | Điện thoại on-call |
| **Ticket** (giờ hành chính) | Cần xử nhưng không cháy: budget mòn chậm, disk 70%, cert sắp hạn | Queue của team |
| **FYI/log** | Thông tin ngữ cảnh: failover xảy ra, deploy xong, CB mở | Kênh chat, không notify |

Kiểm tra chất lượng từng page alert bằng ba câu: *actionable* (người nhận làm được gì ngay?), *urgent* (đợi sáng có sao?), *có runbook* (link ngay trong alert)? Trượt câu nào → hạ tầng đó xuống ticket/FYI. Và một vòng vệ sinh: alert nào bị ack-rồi-bỏ-qua 3 lần liên tiếp phải bị sửa hoặc xóa — alert mà phản xạ chuẩn là "à, cái đó kệ nó" đang dạy người trực bỏ qua *mọi* alert.

## 3. Dashboard — thiết kế theo persona và theo câu hỏi

Dashboard không phải nơi trưng bày mọi metric — nó là **câu trả lời soạn sẵn cho một lớp câu hỏi của một loại người dùng**:

- **On-call, 30 giây đầu:** một trang tổng quan theo luồng nghiệp vụ (checkout, search, thanh toán) — SLI, error budget, burn rate; trả lời đúng một câu "*cái gì* đang đau, *nặng* bao nhiêu". Mọi thứ khác là nhiễu ở thời điểm này.
- **Điều tra, 30 phút sau:** dashboard per-service chuẩn hóa (RED + saturation + dependency — sinh tự động từ service template để mọi service *trông giống nhau*, người trực không phải học lại bản đồ mỗi lần) + đường nhảy sang trace/log ([10.1 §5 — ba mũi tên](/series/system-design/10-observability/01-ba-tru/)). Kèm **overlay sự kiện thay đổi** (deploy, migration, config, scale) lên mọi biểu đồ — vì "cái gì vừa thay đổi?" là câu hỏi ăn tiền nhất của mọi cuộc điều tra ([13.README §nguyên tắc 2](/series/system-design/13-production-failure-cases/00-tong-quan/)).
- **Capacity/chi phí, hằng quý:** utilization vs trần đã đo, tăng trưởng, dự báo chạm ngưỡng ([1.4 §6](/series/system-design/01-foundations/04-scale-estimation-capacity-planning/)).

Ba persona, ba nhịp, ba trang — một dashboard 60 panel cố phục vụ cả ba là phục vụ không ai cả.

## 4. On-call & Postmortem — nửa con người của hệ thống

- **On-call bền vững là bài toán thiết kế, không phải sức chịu đựng:** rotation đủ người (≥4–6 để không cháy), có escalation rõ (không phản hồi X phút → người kế), và — quan trọng nhất — **khối lượng page nằm trong ngân sách** (thô: >2–3 page/đêm kéo dài là hệ thống đang vận hành bằng cách đốt người; nguồn gốc phải sửa ở tầng alert hoặc tầng kiến trúc, không phải bằng tuyển thêm người trực).
- **Runbook cho mỗi page alert:** triệu chứng → các nguyên nhân đã biết → lệnh chẩn đoán → hành động cầm máu → khi nào escalate. Viết lúc bình tĩnh, link trong alert, cập nhật sau mỗi sự cố ([12.10 §3.3 — quyết định đau đớn quyết trước](/series/system-design/12-evolution/10-disaster-recovery/)). Runbook không có nghĩa là biến người trực thành robot — nó giải phóng năng lực tư duy cho phần *chưa* biết của sự cố.
- **Postmortem không đổ lỗi** ([13.README §nguyên tắc 5](/series/system-design/13-production-failure-cases/00-tong-quan/)): timeline (ghi *trong lúc* xử lý — [13.README §4](/series/system-design/13-production-failure-cases/00-tong-quan/)), tác động đo bằng error budget ([1.2 §6](/series/system-design/01-foundations/02-sla-slo-sli/)), chuỗi nguyên nhân (5-whys đến tầng hệ thống, dừng ở "con người bất cẩn" là dừng quá sớm — người bất cẩn được *hệ thống nào* cho phép gây sự cố?), action item có chủ + deadline, và một mục thường bị quên: **"phát hiện có thể nhanh hơn không?"** — nuôi ngược lại tầng alert. Sự cố lặp lại nguyên xi là postmortem trước đó đã thất bại.

## 5. Trade-off

| Quyết định | Được | Giá |
|---|---|---|
| Alert triệu chứng (SLO/burn-rate) thay vì nguyên nhân | Ít rule, ít nhiễu, khớp trải nghiệm user | Cần nền SLI/SLO tử tế trước ([1.2](/series/system-design/01-foundations/02-sla-slo-sli/)); biết "đau" trước khi biết "vì sao" |
| Ngưỡng nhạy | Bắt sớm | Dương tính giả → fatigue → bỏ lỡ cái thật; nhiễu là thuế đánh vào niềm tin |
| Ngưỡng trễ | Ít nhiễu | MTTR tăng; sự cố nhỏ âm ỉ lọt lưới — bù bằng ticket-tầng cho rò rỉ chậm |
| Dashboard chuẩn hóa tự sinh | Mọi service đọc giống nhau, rẻ bảo trì | Ít chỗ cho tùy biến sâu; panel đặc thù nghiệp vụ vẫn phải làm tay |
| Runbook chi tiết | MTTR giảm, on-call đỡ sợ | Phải nuôi — runbook lỗi thời *nguy hiểm hơn* không có (tự tin sai) |

## 6. Production Considerations

- **Đường alert phải HA hơn hệ thống nó canh:** kênh page độc lập hạ tầng chính (SMS/gọi qua nhà cung cấp ngoài — Slack chết cùng SSO của bạn thì sao?), test đường page định kỳ (một alert giả mỗi tuần có người ack), và escalation tự động khi không ai phản hồi ([13.5 — công cụ cứu hỏa ngoài đám cháy](/series/system-design/13-production-failure-cases/05-infrastructure-failures/)).
- **Silence/maintenance window có kỷ luật:** deploy lớn, drill, migration — silence *có hạn giờ và có chủ*; silence vô hạn bị bỏ quên là cách sự cố thật trôi qua trong im lặng được cấp phép.
- Đo sức khỏe của chính hệ alert: page/tuần theo team, tỷ lệ actionable (hỏi người trực!), MTTA (thời gian đến ack), % page có runbook — bốn con số này là SLI của khâu con người.
- Với hệ nhiều team: alert routing theo **ownership** ([12.6 — service có owner rõ](/series/system-design/12-evolution/06-microservices/)) — alert đi lạc team là alert chết trong hộp thư người không sửa được nó.

## 7. Anti-patterns

- **Alert mọi thứ "cho an toàn"** — con đường ngắn nhất đến chỗ không ai nhìn gì nữa; an toàn thật nằm ở ít-mà-đáng-tin.
- **Page cho việc không làm được gì lúc 3h sáng** (disk sẽ đầy trong 3 tuần) — mỗi lần đánh thức vô cớ là một khoản rút từ tài khoản niềm tin.
- **Dashboard-tường:** 8 màn hình 200 panel không ai biết panel nào quan trọng — trang trí, không phải công cụ.
- **Threshold tĩnh copy từ blog** ("CPU > 80% là xấu") không gắn với capacity đã đo của chính mình ([1.4 §3.1](/series/system-design/01-foundations/04-scale-estimation-capacity-planning/)).
- **Postmortem kết luận "do dev X thiếu cẩn thận, đã nhắc nhở"** — vừa độc với văn hóa vừa vô dụng với hệ thống: lần sau người khác sẽ "thiếu cẩn thận" đúng chỗ đó.
- **Anh hùng on-call:** một người giỏi gánh mọi sự cố bằng trí nhớ — bus factor bằng 1, và là dấu hiệu runbook + alert đang nợ chính người đó.

## 8. Khi nào đơn giản là đủ

Team nhỏ, hệ nhỏ: uptime check bên ngoài + Sentry + một dashboard RED + nhóm chat nhận cảnh báo — chưa cần rotation chính thức hay burn-rate đa cửa sổ ([1.2 §9](/series/system-design/01-foundations/02-sla-slo-sli/)). Bộ máy đầy đủ của chương này kích hoạt cùng những thứ đã quen thuộc trong tài liệu: nhiều team, nhiều service, SLO có khách hàng trả tiền — và như mọi lần, xây *khi tín hiệu đòi hỏi*, không phải khi đọc xong một chương sách.

---

*Hết Phần 10. Quay lại [mục lục chính](/series/system-design/00-muc-luc/).*
