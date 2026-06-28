+++
title = "Bài 3 — Partitioning & Đảm bảo thứ tự"
date = "2026-06-06T08:00:00+07:00"
draft = false
tags = ["backend", "kafka"]
series = ["Kafka Thực Chiến"]
+++

## Producer chọn partition thế nào?

- **Có key:** `partition = hash(key) % số_partition`. Cùng key + số partition không đổi → luôn cùng partition → giữ thứ tự.
- **Không key:** rải đều (sticky/round-robin), tối ưu throughput nhưng **không đảm bảo thứ tự**.

## Ba cạm bẫy quanh partitioning

### Bẫy 1 — Hàm hash khác nhau giữa các client

Client **Java** dùng **murmur2**; **librdkafka** (confluent-kafka-go, Python, C…) mặc định dùng thuật toán khác (`consistent_random`, nền CRC32). → **Cùng một key, producer Java và producer Go có thể đẩy vào hai partition khác nhau!**

Nếu nhiều service khác ngôn ngữ cùng ghi một topic và cần giữ thứ tự theo key → phải ép cùng partitioner. Với Go: `"partitioner": "murmur2_random"` để khớp Java.

### Bẫy 2 — Tăng số partition phá vỡ thứ tự

Vì partition tính theo `% số_partition`, tăng từ 3 lên 6 → `hash(key) % 3 ≠ hash(key) % 6` → message cũ của một key ở partition A, message mới sang partition B. Hai partition do hai consumer khác nhau đọc với tốc độ khác nhau → **message mới có thể được xử lý trước message cũ**. Đây mới là cái nguy hiểm thật sự.

→ Chọn số partition đủ lớn ngay từ đầu.

### Bẫy 3 — Hot partition (lệch tải)

Key phân bố lệch (một `tenant_id` lớn chiếm 80% traffic) → mọi message dồn vào một partition → một consumer quá tải, các consumer khác rảnh. **Cân bằng số lượng partition/consumer KHÔNG cứu được lệch tải.**

Giải pháp **key salting**: ghép thêm thành phần phụ, ví dụ `tenantId + "-" + (id % 10)`. Đổi lại: thứ tự chỉ còn giữ trong từng nhóm con, không phải toàn bộ tenant. Đây là đánh đổi thứ tự ↔ cân tải.

## Nguyên tắc vàng về thứ tự

Kafka chỉ đảm bảo thứ tự **trong một partition**. Phạm vi cần giữ thứ tự gọi là **ordering key**:
- Chọn càng hẹp (per-user) → rải đều, ít hot.
- Chọn càng rộng (chung 1 key) → dễ hot partition.

**Thứ tự và song song hóa kéo ngược nhau, và "đơn vị" của cả hai đều là partition.** Senior là người biết hỏi "thứ tự cần chặt tới đâu" thay vì mặc định làm chặt nhất rồi lãnh hot partition.

## Lưu ý quan trọng: thứ tự ≠ idempotency

Đây là hai vấn đề tách rời:
- **Khác key → không có thứ tự xuyên-key** — thường chấp nhận được, vì hai key là hai thực thể độc lập.
- **Idempotency** sinh ra từ **retry / at-least-once** (Bài 4), để xử lý một message nhiều lần vẫn ra một kết quả — không liên quan tới việc cùng key hay khác key.

## Tình huống thiết kế: payment cần thứ tự per-account nhưng có VIP gây hot

Không có đáp án hoàn hảo. Thứ tự ưu tiên xử lý:

1. **Đo trước**: một partition kham được hàng chục nghìn msg/s — VIP có thật sự vượt ngưỡng? Đừng tối ưu sớm.
2. **Thu hẹp ordering key**: per-account thay vì per-user nếu nghiệp vụ cho phép.
3. **Tách VIP ra luồng/topic riêng** nhiều tài nguyên hơn.
4. **Song song hóa trong consumer theo key nhỏ hơn** (dời chỗ giải quyết song song từ tầng partition xuống tầng ứng dụng).

## Tóm tắt

Partition được tính từ key, không phải bản thân key. Cùng key cùng partition chỉ đúng khi số partition không đổi và cùng client/partitioner. Thiết kế key là bài toán cân bằng giữa giữ thứ tự và tránh hot partition.
