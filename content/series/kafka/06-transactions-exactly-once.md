+++
title = "Bài 6 — Idempotent Producer & Transactions (Exactly-Once)"
date = "2026-06-12T08:00:00+07:00"
draft = false
tags = ["backend", "kafka"]
series = ["Kafka Thực Chiến"]
+++

## Vấn đề 1 — Duplicate do PRODUCER retry

Producer gửi message, broker **đã ghi thành công** nhưng gói ACK **rớt trên đường về**. Producer tưởng thất bại → retry → broker ghi **lần hai**. Message nằm 2 lần trong log dù bạn làm đúng mọi thứ. (Khác với duplicate phía consumer ở Bài 4.)

### Giải pháp — Idempotent Producer

`enable.idempotence=true`: broker cấp mỗi producer một **PID** và mỗi message một **sequence number** theo partition. Retry mang sequence đã thấy → broker **lặng lẽ bỏ qua**. Thêm bảo đảm thứ tự không bị đảo khi retry.

- Chi phí gần như bằng 0 → **bật mặc định, không lý do gì tắt** (client mới tự bật). Tự kéo theo `acks=all`.
- Giới hạn: chỉ chống trùng **trong một phiên producer, trên từng partition**. KHÔNG cứu: app tự gửi lại ở tầng nghiệp vụ, producer restart (PID mới), mọi vấn đề phía consumer.

## Vấn đề 2 — "Consume-transform-produce"

Pipeline: đọc topic A → biến đổi → ghi topic B → commit offset A. Có **2 thao tác ghi** ở hai nơi (message ra B + offset A). Crash giữa chừng → B có message mà offset chưa commit (chạy lại → B **trùng**), hoặc ngược lại (mất).

### Giải pháp — Transactions

Gói nhiều thao tác thành **một khối nguyên tử**: ghi message sang B + commit offset A **cùng thành công hoặc cùng biến mất**.

- `transactional.id`: định danh **bền vững** qua restart. Kèm **epoch**: producer mới khởi động cùng id → epoch tăng → bản cũ (zombie) bị **fence** (từ chối).
- API: `InitTransactions` → `BeginTransaction` → produce + `SendOffsetsToTransaction` → `CommitTransaction` (hoặc `AbortTransaction`).
- Consumer đọc B phải đặt **`isolation.level=read_committed`**: chỉ thấy message của transaction đã commit; message bị abort bị ẩn (nhưng vẫn chiếm offset trong log).

## Transaction Kafka KHÁC transaction RDBMS

| | RDBMS | Kafka |
|---|---|---|
| Rollback | Xóa dấu vết (undo) | **Không** — message abort vẫn nằm trong log, chỉ ghi thêm marker "aborted" để consumer `read_committed` lờ đi |
| Phạm vi | Đọc + ghi, lock, nhiều isolation level | Chỉ gom **các thao tác GHI vào Kafka** (produce + commit offset) |
| Ra ngoài hệ thống | Trong cùng DB | **Không** vươn ra ngoài Kafka (không gom được Kafka + Postgres + email) |

Kafka "rollback" bằng cách *che mắt người đọc*, không phải xóa dữ liệu.

## Chọn công cụ đúng (quan trọng nhất bài)

| Bài toán | Công cụ |
|----------|---------|
| Đọc topic A → xử lý → ghi topic B | **Kafka transaction** |
| Ghi DB + bắn event Kafka, phải khớp nhau | **Outbox pattern** (dùng transaction của DB — Bài 10) |
| Đọc Kafka → ghi DB / gửi noti / gọi API | **At-least-once + idempotency key** (Bài 4) |

> Kafka transaction chỉ nguyên tử **trong phạm vi Kafka**. Đích đến là DB/API/email → transactions không với tới được (không thể rollback email đã gửi). Đây là điều rất nhiều người hiểu sai.

## Câu hỏi phỏng vấn

> *Service đọc Kafka → gửi notification → ghi DB. Có nên dùng transactions?*
> → Không. Đích đến ngoài Kafka. Dùng at-least-once + idempotency key. Transactions chỉ cho pipeline Kafka→Kafka.

## Tóm tắt

Idempotent producer: bật mặc định, chống trùng do retry, gần như miễn phí. Transactions: mạnh nhưng hẹp (chỉ Kafka→Kafka), phức tạp, chậm hơn — phần lớn hệ thống thực tế dùng at-least-once + idempotency thay vì transactions.
