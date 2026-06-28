+++
title = "Bài 4 — Offset & Delivery Semantics"
date = "2026-06-08T08:00:00+07:00"
draft = false
tags = ["backend", "kafka"]
series = ["Kafka Thực Chiến"]
+++

> Bài quan trọng nhất về độ tin cậy. Hầu hết bug "mất message" / "xử lý trùng" ở production nằm ở đây.

## Offset commit là gì?

Consumer cần báo cho Kafka biết đã xử lý **đến đâu**, để lần sau (hoặc consumer khác sau rebalance) đọc tiếp đúng chỗ. Việc đó gọi là **commit offset**, lưu trong topic nội bộ `__consumer_offsets`.

Mấu chốt sinh ra mọi vấn đề: **xử lý message** và **commit offset** là hai hành động riêng biệt, giữa chúng consumer có thể crash. Thứ tự hai hành động này quyết định "delivery semantics".

> Offset commit = `last_processed + 1` (vị trí SẼ đọc tiếp), không phải offset vừa xử lý.

## Ba mức đảm bảo (delivery semantics)

| Mức | Cách làm | Rủi ro | Dùng khi |
|-----|----------|--------|----------|
| **At-most-once** | Commit **trước** khi xử lý | Crash giữa chừng → **mất message** | Log, metric không quan trọng |
| **At-least-once** | Xử lý **trước**, commit **sau** | Crash giữa chừng → **xử lý trùng** | Mặc định, phổ biến nhất |
| **Exactly-once** | Transactions (Bài 6) | Phức tạp, chậm hơn | Pipeline Kafka→Kafka |

**Kafka mặc định là at-least-once**, và đó cũng là lựa chọn của đa số hệ thống. Vì có thể trùng → consumer **phải idempotent**.

## Auto-commit vs Manual commit

- `enable.auto.commit=true` (mặc định): tự commit nền mỗi `auto.commit.interval.ms` (5s). **Nguy hiểm** vì commit theo message đã *poll* chứ không phải đã *xử lý xong*. Hành vi **không xác định**: lúc mất (commit trước khi xử lý xong rồi crash), lúc trùng (xử lý xong nhưng chưa tới chu kỳ commit rồi crash).
- **Manual commit** (`enable.auto.commit=false` + tự `CommitMessage` sau khi xử lý xong): kiểm soát chính xác. Lựa chọn cho dữ liệu quan trọng.

```go
// At-least-once: XỬ LÝ trước -> COMMIT sau
process(msg)                 // business logic
c.CommitMessage(msg)         // chỉ commit khi chắc chắn xử lý xong
```

## Idempotency — biến at-least-once thành "đúng một lần về kết quả"

Khi consumer đọc lại message trùng, phải đảm bảo xử lý 2 lần ra cùng một kết quả. Kỹ thuật chuẩn — **idempotency key**:

1. Mỗi sự kiện logic mang một key **ổn định** (UUID sinh **một lần tại nguồn**, giữ nguyên qua mọi retry). **Không** random lại mỗi lần gửi — nếu không dedup vô dụng.
2. Consumer chống trùng bằng **ràng buộc UNIQUE** trên `idempotency_key` + `INSERT ... ON CONFLICT DO NOTHING` (để DB đảm bảo, tránh race condition của "check-rồi-mới-ghi").
3. Lý tưởng: ghi idempotency key và ghi kết quả nghiệp vụ trong **cùng một transaction DB**.
4. Bảng dedup cần **TTL/retention** (7–30 ngày, khớp retention topic) để không phình mãi.

> Mô hình senior phổ biến nhất: **at-least-once (manual commit) + idempotency key có UNIQUE constraint**. Hiệu quả "exactly-once về kết quả nghiệp vụ", rẻ và đơn giản hơn transactions.

## Đọc `__consumer_offsets` — hai loại bản ghi

Topic này chứa HAI loại bản ghi khác nhau (đừng nhầm):

1. **Offset commit**: `{offset, leader_epoch, metadata, commit_timestamp}`. Key = `(group, topic, partition)`.
2. **Group metadata**: `{generation, protocol, leader, members[...]}`. Sinh ra mỗi lần **rebalance**, do coordinator ghi. Key = `group.id`.

Hai cơ chế chống "đồ cũ lỗi thời": `leader_epoch` (cho partition leader ở broker) và `generation` (tăng sau mỗi rebalance, chống zombie consumer). Đây là **compacted topic** (chỉ giữ giá trị mới nhất theo key) nên không phình vô hạn.

## Câu hỏi phỏng vấn

> *Khác biệt at-least-once và at-most-once là gì, và cài đặt thế nào?*
> → Chỉ là **thứ tự commit so với xử lý**. Commit trước = at-most-once (có thể mất). Xử lý trước = at-least-once (có thể trùng, cần idempotency).

## Tóm tắt

Semantics là *mục tiêu*; thứ tự commit là *phương tiện*. Manual commit cho bạn quyền chọn dứt khoát. Trong thực tế gần như luôn là "at-least-once + làm sao cho idempotent".
