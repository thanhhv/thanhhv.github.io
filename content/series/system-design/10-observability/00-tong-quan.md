+++
title = "Phần 10 — Observability"
date = "2026-07-13T13:00:00+07:00"
draft = false
tags = ["backend", "system-design"]
series = ["System Design — Tư Duy Thiết Kế Hệ Thống"]
+++

> Chủ đề: Logging, Metrics, Tracing, OpenTelemetry, Dashboard, Alerting.

## Luận điểm trung tâm của phần này

Observability là khả năng **trả lời câu hỏi chưa biết trước** về hệ thống từ tín hiệu nó phát ra — khác monitoring (canh các câu hỏi đã biết). Trong hệ phân tán, nó không phải tiện nghi mà là điều kiện vận hành: toàn bộ [Phần 13](/series/system-design/13-production-failure-cases/00-tong-quan/) — mọi mục Metric/Dashboard/Alert/Điều tra — chính là phần này trong hành động.

Ba mệnh đề xuyên suốt:

1. **Ba trụ là ba câu hỏi, không phải ba công cụ:** metrics — *có chuyện gì, xu hướng nào*; logs — *chi tiết chuyện gì đã xảy ra*; traces — *request này đi đâu, chậm ở chặng nào*. Đủ ba mới khép kín vòng điều tra.
2. **Chi phí observability là chi phí kiến trúc thật** — log vô kỷ luật là hạng mục dữ liệu lớn nhất hệ thống ([1.4 §2.2 — 11TB/năm](/series/system-design/01-foundations/04-scale-estimation-capacity-planning/)); thiết kế tín hiệu cũng cần capacity planning như thiết kế dữ liệu.
3. **Alert trên triệu chứng user cảm nhận, điều tra bằng nguyên nhân** ([1.2 — SLO/burn rate](/series/system-design/01-foundations/02-sla-slo-sli/)) — đảo ngược thứ tự này là ngập trong alert hoặc im lặng chết người.

## Mục lục

- [10.1. Ba trụ — Logging, Metrics, Tracing](/series/system-design/10-observability/01-ba-tru/)
- [10.2. OpenTelemetry & pipeline tín hiệu — thu, xử lý, lưu, trả tiền](/series/system-design/10-observability/02-opentelemetry-pipeline/)
- [10.3. Dashboard, Alerting & On-call — từ tín hiệu đến con người](/series/system-design/10-observability/03-dashboard-alerting-oncall/)

## Phân vai với các phần khác

[1.2](/series/system-design/01-foundations/02-sla-slo-sli/) đặt khung SLI/SLO/error budget — *đo cái gì và cam kết gì*; [1.5](/series/system-design/01-foundations/05-bottleneck-analysis/) dạy quy trình chẩn đoán — *dùng tín hiệu như thế nào*; [Phần 13](/series/system-design/13-production-failure-cases/00-tong-quan/) là 21 bài thực hành. Phần này lo phần còn lại: *xây bộ máy tín hiệu* — cấu trúc từng loại, pipeline vận chuyển, và giao diện giữa máy và người trực.
