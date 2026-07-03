+++
title = "Bài 10 — Kiến trúc thực chiến"
date = "2026-05-23T08:00:00+07:00"
draft = false
tags = ["backend", "kafka"]
series = ["Kafka Thực Chiến"]
+++

## Outbox Pattern — giải bài toán "dual write"

Service order phải ghi vào **DB** và bắn event lên **Kafka**. Làm tuần tự thì luôn có khe hở:

```
ghi DB OK  ->  bắn Kafka FAIL   => DB có đơn, hệ thống khác không biết (lệch)
bắn Kafka OK  ->  ghi DB FAIL   => event bắn rồi nhưng đơn không tồn tại (ma)
```

Hai hệ thống khác nhau không chia sẻ transaction → không thể "cùng thành công hoặc cùng thất bại" trực tiếp.

**Lời giải Outbox:** trong **cùng một transaction DB**, ghi *cả* đơn hàng *và* một bản ghi event vào bảng `outbox`. Cùng transaction Postgres → nguyên tử tuyệt đối. Sau đó tiến trình riêng đọc `outbox` bắn lên Kafka:
- **Polling publisher:** job định kỳ `SELECT ... WHERE sent=false`, bắn, đánh dấu `sent=true`. Đơn giản, có độ trễ, tải DB.
- **CDC (Debezium):** đọc thẳng **transaction log** của DB (WAL Postgres / binlog MySQL), thấy row outbox mới là đẩy lên Kafka ngay. Real-time, không tải DB. Cách hiện đại.

> Outbox đảm bảo at-least-once → consumer vẫn cần idempotency. Mọi con đường dẫn về idempotency.

### Ranh giới trách nhiệm (hiểu nhầm phổ biến)

- **Outbox chỉ đảm bảo: "sự thật được ghi nhận và phát đi đáng tin cậy."** Trách nhiệm dừng ở mép Kafka.
- **Consumer thành công hay không là bài toán KHÁC**, giải bằng at-least-once + retry + DLQ + idempotency (Bài 4, 7). Message không "mất" vì consumer fail — nó nằm trong topic chờ.
- **Saga ≠ retry.** Saga để **hoàn tác (compensate)** khi một bước fail làm các bước trước vô nghĩa. Câu hỏi quyết định: *"Bước này fail vĩnh viễn thì việc đã làm trước có cần UNDO không?"* Noti fail KHÔNG hủy đơn hàng → **không cần saga**. Đặt vé (giữ chỗ → trừ tiền → xuất vé) mà trừ tiền fail → phải nhả chỗ → **cần saga**.

| Tình huống | Công cụ |
|---|---|
| Đảm bảo event rời producer đáng tin cậy | Outbox |
| Consumer fail tạm thời | Retry + backoff |
| Consumer fail vĩnh viễn, không cần undo | DLQ |
| Tránh xử lý trùng khi retry | Idempotency key |
| Một bước fail làm bước trước phải bị HỦY về nghiệp vụ | Saga |

## Choreography vs Orchestration

- **Orchestration:** có "nhạc trưởng" gọi lần lượt từng service, biết toàn cục flow. Hợp quy trình nghiệp vụ phức tạp cần kiểm soát thứ tự/trạng thái.
- **Choreography (Kafka event-driven):** producer chỉ **công bố một sự thật** rồi quên. Ai quan tâm tự lắng nghe và phản ứng. **Decoupling tuyệt đối** — thêm service mới chỉ cần subscribe, không sửa producer.

Hệ thống thật thường **trộn cả hai**: choreography cho event broadcast, orchestration/saga cho quy trình phức tạp.

## Hệ sinh thái Kafka

- **Kafka Connect:** bơm dữ liệu vào/ra Kafka không cần viết code (source: DB→Kafka như Debezium; sink: Kafka→ES/S3/DB).
- **Kafka Streams / ksqlDB:** xử lý luồng trên Kafka (lọc, join, windowed aggregation). Nơi exactly-once tỏa sáng (Kafka→Kafka). Là thư viện Java; với Go thường tự xử lý hoặc dùng ksqlDB.
- **Schema Registry** (Bài 8): hợp đồng dữ liệu.

## Khi nào KHÔNG dùng Kafka

Senior được trả lương để **không** over-engineer. Tránh Kafka khi:
- Chỉ cần task queue đơn giản → RabbitMQ / Redis / SQS.
- Cần RPC / request-response đồng bộ → gRPC / HTTP (Kafka một chiều).
- Cần độ trễ siêu thấp <1ms hoặc ACID xuyên nhiều bảng → DB.
- Hệ thống nhỏ, ít service, lưu lượng thấp → chi phí vận hành Kafka không đáng; một bảng DB làm queue có khi đủ.

Kafka tỏa sáng khi: nhiều consumer độc lập cần cùng một luồng (pub/sub), cần replay lịch sử, throughput cực lớn, event-driven/event-sourcing, hoặc cần decouple producer khỏi consumer.

## Tóm tắt

Outbox đảm bảo event rời producer đáng tin cậy; consumer tự lo phần mình; saga chỉ khi cần undo nghiệp vụ. Kafka mạnh nhất ở choreography event-driven. Và biết khi nào KHÔNG dùng Kafka quan trọng ngang biết dùng.
