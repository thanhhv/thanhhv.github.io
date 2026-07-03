+++
title = "Bài 9 — Performance Tuning"
date = "2026-05-20T08:00:00+07:00"
draft = false
tags = ["backend", "kafka"]
series = ["Kafka Thực Chiến"]
+++

> Mọi tinh chỉnh là đánh đổi giữa **throughput** (số lượng/giây), **latency** (độ trễ mỗi message), và **durability**. Không có nút "nhanh hơn" miễn phí.

## Phía Producer — gom để nhanh

**`linger.ms` + `batch.size`** (cặp quan trọng nhất). Mặc định `linger.ms=0` = gửi ngay từng message → nhiều request nhỏ. Tăng `linger.ms` (5–20ms) cho producer chờ gom batch rồi gửi một lần. `batch.size` (mặc định 16KB) là trần mỗi batch.
- Đánh đổi: linger cao = throughput tăng mạnh, nhưng latency mỗi message tăng.

**`compression.type`** (`lz4`/`zstd` tốt nhất hiện nay). Nén cả batch: giảm băng thông + giảm đĩa + thường *tăng* throughput. Hiệu quả nhất khi đi cùng batch lớn. **Quick-win rẻ nhất, gần như luôn nên bật cho text/JSON.**

**`acks`** (Bài 5): `all` an toàn nhưng chậm hơn; `1` nhanh hơn, rủi ro mất.

## Phía Consumer — đọc theo lô

- **`fetch.min.bytes`** (mặc định 1): broker chờ đủ bấy nhiêu byte mới trả. Tăng (64KB) → lô lớn, hiệu quả hơn, nhưng chờ lâu hơn. Là "linger.ms phía consumer".
- **`fetch.max.wait.ms`** (500ms): trần thời gian chờ đó.
- **`max.poll.records`**: số message tối đa mỗi poll. Lớn = lô lớn (nhanh) nhưng coi chừng `max.poll.interval.ms`.

## Consumer Lag — metric sức khỏe số 1

```
lag = (offset cuối partition) − (offset consumer đã commit) = số message tồn đọng
```

- Dao động quanh 0 → consumer theo kịp. Khỏe.
- **Tăng đều không dừng** → consumer chậm hơn producer → phải hành động.

Ba hướng khi lag tăng:
1. **Scale ngang**: tăng partition + consumer (nhớ trần = số partition).
2. **Đọc theo lô**: `fetch.min.bytes`, `max.poll.records`.
3. **Tối ưu chính phần xử lý**: batch insert/update DB, song song theo key. Thường nút thắt là **downstream (DB/API)**, không phải Kafka — thêm consumer chỉ làm DB chết nhanh hơn. Senior hỏi "nút thắt thật ở đâu" trước khi scale.

## Nguyên tắc benchmark (bài học quan trọng nhất)

Một lần đo đơn lẻ là **kết quả rác** — dễ bị nhiễu (GC, flush đĩa, Docker bận, leader election). Quy tắc:
- **Warm-up trước, đo sau** (lần chạy đầu tốn khởi tạo kết nối/metadata/cache lạnh).
- **Chạy nhiều lần, lấy trung bình + xem p50/p99**, không tin một lần.
- **Cô lập biến** — mỗi config một lần chạy riêng, tránh ảnh hưởng buffer/GC còn sót.

> "Đo một lần rồi kết luận" là cách ra quyết định tuning sai. Đừng tối ưu mù — đo, đổi MỘT tham số, đo lại.

## Cấu hình theo loại topic

| | Noti (latency thấp) | Analytics (throughput cao) |
|---|---|---|
| `linger.ms` (producer) | 5–10ms | 100–500ms |
| `compression` | none/lz4 | zstd |
| `fetch.min.bytes` (consumer) | nhỏ | lớn |
| ưu tiên | user thấy noti nhanh | nuốt khối lượng lớn, trễ vài giây OK |

Lưu ý: `linger.ms` là tham số **producer**, không phải consumer. Đừng để `linger=0` ở tải cao — quá nhiều request nhỏ có khi *chậm hơn* `linger=5ms`.

## Tóm tắt

Gom batch (linger) + nén (compression) + giảm durability (acks=1) đều tăng throughput, mỗi cái đổi lấy một thứ (latency / CPU / an toàn). Lag là kim chỉ nam. Tuning phải dựa trên đo lường nghiêm túc, không cảm tính.
