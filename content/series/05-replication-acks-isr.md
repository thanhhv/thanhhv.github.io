+++
title = "Bài 5 — Replication, acks & ISR"
date = "2026-06-10T08:00:00+07:00"
draft = false
tags = ["backend", "kafka"]
series = ["Kafka Thực Chiến"]
+++

## Replica, Leader, Follower

Mỗi partition có N bản sao (replication-factor):
- **1 leader**: phục vụ mọi đọc/ghi của client.
- **N-1 follower**: liên tục kéo dữ liệu từ leader để bắt kịp. Ẩn với client.

Client **chỉ làm việc với leader**. Follower chỉ để chống mất dữ liệu khi broker chứa leader chết.

## ISR (In-Sync Replicas)

Tập các replica **đang theo kịp** leader (gồm cả leader). Follower tụt quá `replica.lag.time.max.ms` (~30s) bị **đá khỏi ISR**. Khi leader chết, Kafka chỉ bầu leader mới **từ trong ISR** (vì chỉ chúng chắc chắn có đủ dữ liệu đã xác nhận).

> RF là con số **cố định** (mục tiêu); ISR là con số **dao động** (thực tế, co lại khi sự cố). Khỏe mạnh: ISR = RF.

## `acks` — producer đợi xác nhận tới đâu

| acks | Đợi gì | Rủi ro |
|------|--------|--------|
| `0` | Không đợi | Mất message không hay biết |
| `1` | Leader ghi xong | Leader chết ngay sau ack mà follower chưa copy → **mất** |
| `all` | Mọi replica trong ISR ghi xong | An toàn nhất |

**Lỗ hổng của `acks=all`:** "all" = tất cả trong **ISR hiện tại**. Nếu follower rớt hết, ISR co còn mình leader → `acks=all` thực chất = `acks=1`. → cần `min.insync.replicas`.

## `min.insync.replicas`

Số replica tối thiểu phải có trong ISR thì mới **cho phép ghi**. ISR thấp hơn mức này → broker **từ chối ghi** (`NOT_ENOUGH_REPLICAS`) — thà ngừng nhận còn hơn nhận mà không an toàn. **Chỉ có tác dụng khi đi kèm `acks=all`.**

## Công thức vàng production (thuộc lòng)

```
replication.factor = 3  +  min.insync.replicas = 2  +  acks = all
```

→ Chịu được **1 broker chết** mà vẫn đọc/ghi, không mất dữ liệu đã xác nhận. Chết broker thứ 2 → **từ chối ghi** (bảo toàn dữ liệu, hy sinh khả dụng ghi). Đây là một lựa chọn cụ thể trong đánh đổi Consistency vs Availability.

## Bảng đánh đổi

| Cấu hình | Chết 1 broker | Chết 2 broker | Mất data đã ack? |
|----------|---------------|----------------|------------------|
| `acks=1` | ghi tiếp | ghi tiếp | **Có — mất im lặng** |
| `acks=all, min.insync=2` | ghi tiếp | **từ chối ghi** | Không |
| `acks=all, min.insync=3` | **từ chối ghi** | từ chối ghi | Không |

`min.insync=3` trên cụm 3 broker: durability tối đa nhưng write-availability tệ nhất (chết 1 broker là ngừng ghi). Hầu như không ai dùng.

## Unclean leader election

Mặc định `unclean.leader.election.enable=false`: khi leader duy nhất chết, partition thành **offline**, Kafka **chờ** đúng broker đó sống lại chứ không bầu một bản ngoài ISR (vì thế là chấp nhận mất dữ liệu). Bật `true` = đổi mất-dữ-liệu lấy uptime. Đặt được **theo từng topic**.

## Hai profile đối lập

| | Topic "rẻ" (metric, log) | Topic "quý" (payment, order) |
|---|---|---|
| acks | 0 hoặc 1 | all |
| RF | 1–2 | 3 |
| min.insync.replicas | (không cần) | 2 |
| unclean election | có thể bật | tuyệt đối tắt |
| retention | ngắn | dài |
| ưu tiên | throughput, uptime | không mất, không lệch |

## Vận hành

- **URP (Under-Replicated Partitions)** là metric cảnh báo quan trọng nhất: số partition có replica tụt khỏi ISR. Khỏe = 0. Đặt alert trên nó.
- **Active Controller** phải luôn = 1 (broker "nhạc trưởng" quản lý metadata, điều phối bầu leader — khác với *group coordinator* của từng group).

## Câu hỏi phỏng vấn

> *`acks=all` đã đảm bảo không mất dữ liệu chưa?*
> → Chưa, nếu thiếu `min.insync.replicas`. Khi ISR co còn 1, `acks=all`退化 thành `acks=1`. Phải kết hợp `acks=all` + `min.insync.replicas>=2`.

## Tóm tắt

Không có cấu hình "đúng", chỉ có cấu hình khớp với giá trị của dữ liệu. Công thức vàng `RF=3 + min.insync=2 + acks=all` không phải câu thần chú mà là một quyết định đánh đổi C-vs-A có ý thức.
