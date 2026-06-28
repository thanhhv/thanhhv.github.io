+++
title = "Bài 7 — Retry, Backoff & Dead Letter Queue"
date = "2026-06-14T08:00:00+07:00"
draft = false
tags = ["backend", "kafka"]
series = ["Kafka Thực Chiến"]
+++

> Triết lý: **fail gracefully, đừng fail catastrophically.** Một message hỏng là chuyện bình thường; mục tiêu là nó **không kéo cả partition/hệ thống sập**.

## Hai loại lỗi — phân biệt được mới xử lý đúng

- **Tạm thời (transient):** DB timeout, API phụ thuộc down, mạng chập chờn. → **retry** sẽ qua.
- **Vĩnh viễn (permanent / "poison"):** sai format, thiếu field, vi phạm nghiệp vụ. → retry vô ích, đẩy đi chỗ khác.

Ánh xạ HTTP status hữu ích:
- **4xx (trừ 408, 429)** = vĩnh viễn → DLQ ngay (404, 400, 422).
- **5xx** = tạm thời → retry + backoff (500, 502, 503).
- **429, 408** = tạm thời đặc biệt → retry, tôn trọng `Retry-After`.

## Hai cạm bẫy chết người

1. **Retry mù tại chỗ (blocking):** ngồi retry message hiện tại mãi → **chặn cả partition**, message phía sau không bao giờ được xử lý. Một message hỏng làm tắc toàn bộ.
2. **Retry vô hạn không backoff:** dội bom service đang ốm → nó càng không gượng dậy. Cần **exponential backoff** (1s, 2s, 4s…) + **jitter** (ngẫu nhiên hóa) để tránh nhiều consumer retry đồng loạt.

## Dead Letter Queue (DLQ)

Topic riêng (ví dụ `orders.DLT`) chứa message **không xử lý được sau N lần**. Triết lý: thà bỏ qua một message hỏng và đi tiếp, còn hơn để nó chặn cả dòng.

Khi đẩy vào DLQ, kèm **metadata** (header): lỗi gì, đã thử mấy lần, từ topic/partition/offset nào, lúc nào, và **trace_id** để nối vào distributed tracing (Jaeger/Tempo) + log (Kibana/Loki).

> DLQ không phải nơi message đi để chết, mà là nơi chờ được cứu. Vòng đời đầy đủ: **lỗi → DLQ → alert → điều tra/sửa → replay** (đọc lại từ DLQ bơm về topic chính). Header `x-origin-*` phục vụ việc replay.

Đặt **alert trên tốc độ message vào DLQ** — tăng đột biến = có gì đó hỏng.

## Chiến lược retry (đơn giản → phức tạp)

- **DLQ ngay, không retry:** hợp khi lỗi tạm thời hiếm.
- **Retry tại chỗ có giới hạn + backoff ngắn:** hợp lỗi tạm thời ngắn. Rủi ro chặn partition nếu backoff dài.
- **Retry topics (tiered):** `retry-5s` → `retry-1m` → `retry-10m` → DLQ. Message lỗi đẩy sang topic retry (không chặn topic chính), consumer riêng xử lý độ trễ. Phức tạp nhưng không chặn dòng chính.

### Vì sao retry-topic ưu việt (nối Bài 2)

Retry tại chỗ dùng `time.Sleep` để chờ backoff → consumer **không poll** trong lúc đó → nếu sleep lâu sẽ vi phạm **`max.poll.interval.ms`** → bị đá khỏi group → rebalance. (Lưu ý: heartbeat vẫn chạy ở thread nền, cái bị phá là `max.poll.interval.ms`, không phải `session.timeout.ms`.) Retry-topic tách phần "chờ" ra khỏi vòng poll chính → topic chính không bao giờ phải sleep.

## Retry luôn cần idempotency

Retry về bản chất là cố tình tạo at-least-once. Mỗi lần retry là một khả năng "việc đã làm một phần rồi vẫn thử lại". Không có idempotency, retry biến từ cơ chế tăng độ tin cậy thành cơ chế **nhân bản lỗi**.

## Bốn lớp phòng vệ

```
retry có trần  +  backoff tăng dần  +  DLQ  +  non-blocking
```

## Tóm tắt

Phân loại lỗi (tạm thời/vĩnh viễn), retry có trần + backoff cho lỗi tạm thời, DLQ cho lỗi vĩnh viễn, non-blocking để không vi phạm max.poll.interval. Cộng với idempotency và quy trình replay là một hệ thống tiêu thụ message trưởng thành.
