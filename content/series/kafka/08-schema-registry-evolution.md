+++
title = "Bài 8 — Schema Registry & Schema Evolution"
date = "2026-05-17T08:00:00+07:00"
draft = false
tags = ["backend", "kafka"]
series = ["Kafka Thực Chiến"]
+++

## Vấn đề

Topic được nhiều team ghi/đọc. Một ngày team A đổi cấu trúc message (đổi tên field, đổi kiểu, bỏ field) → consumer của B, C, D **vỡ ngay**, thường lúc 2h sáng. JSON tự do có 3 điểm yếu ở quy mô lớn: không có schema bắt buộc, tốn băng thông (lặp tên field), không kiểm soát thay đổi.

## Schema Registry hoạt động thế nào

Service riêng lưu và đánh version cho schema:
- Producer đăng ký schema → nhận **schema ID** (số nguyên).
- Message **không nhúng cả schema**, chỉ nhúng **5 byte đầu** (1 magic + 4 schema ID) rồi dữ liệu Avro nhị phân.
- Consumer lấy schema ID, hỏi Registry (có **cache**, không hỏi mỗi message), nhận schema, decode.

→ Message nhỏ gọn, được validate, mọi thay đổi đi qua một cổng kiểm soát.

## Schema Evolution — trái tim của bài

Registry **từ chối** schema mới nếu phá vỡ tương thích. Các chế độ:

| Chế độ | Nghĩa | Cho phép | Nâng cấp trước |
|--------|-------|----------|----------------|
| **BACKWARD** (mặc định) | Consumer schema **mới** đọc được data schema **cũ** | Xóa field, thêm field **có default** | **Consumer** trước |
| **FORWARD** | Consumer schema **cũ** đọc được data schema **mới** | Thêm field, xóa field có default | **Producer** trước |
| **FULL** | Cả hai | | |
| **NONE** | Tắt kiểm tra (nguy hiểm) | | |

### Quy tắc thực chiến

- **Thêm field** an toàn → field phải **có default**.
- **Xóa field** an toàn (backward) → field nên từng có default.
- **Đổi tên field** = xóa + thêm = gần như luôn phá vỡ → đừng đổi tên, hãy thêm field mới + deprecate field cũ.
- **Đổi kiểu** (int → string) hầu như luôn không tương thích.

## Cạm bẫy thực chiến

- **Khi tiến hóa schema, phải xuất phát từ schema ĐÃ ĐĂNG KÝ**, không phải gõ lại bằng trí nhớ. Lệch namespace/thứ tự/kiểu field → Registry coi là record khác → báo `is_compatible:false` dù bạn đã thêm default đúng.
- Default cho field `double` nên là `0.0` (số thực), không phải `0` (một số validator coi là không hợp lệ).
- Vì lý do trên, production thường **không tự sinh schema từ struct** mà viết file `.avsc` tường minh rồi generate code — để kiểm soát chính xác.

## Khi nào KHÔNG cần (tính thực dụng)

Schema Registry là **dao mổ trâu** khi: một team làm chủ cả producer lẫn consumer, chỉ thêm field, lưu lượng vừa phải. Lúc đó **JSON + quy ước "chỉ thêm field" + compression** đã cho 80% lợi ích với 20% công sức (JSON vốn forward/backward compatible tự nhiên khi chỉ thêm field — consumer cũ bỏ qua field lạ).

Nên dùng khi: nhiều team/service chung topic, schema đổi thường xuyên và phức tạp, dữ liệu quan trọng cần validate chặt, throughput cực lớn.

### Hai làm rõ quan trọng

- **Compression ≠ Avro.** Compression (`lz4/zstd`) độc lập với Schema Registry, **trong suốt** với người đọc (UI/consumer tự giải nén) — quick-win nên bật ngay cho JSON. Avro nhị phân mới làm message khó đọc thô (cần UI nối với Registry để decode).
- **Không gọi Registry mỗi message.** Cả hai phía cache schema theo ID. Chi phí thật là giải mã Avro (CPU, thường nhanh hơn parse JSON). Rủi ro: Registry chết lúc gặp schema mới chưa cache → phải chạy HA.

## Tóm tắt

Schema Registry biến "hợp đồng dữ liệu" giữa các team thành thứ máy kiểm soát được, kèm validate tại nguồn và tiết kiệm băng thông. Nhưng nó thêm một thành phần phải nuôi — chỉ dùng khi bài toán đủ lớn để đáng.
