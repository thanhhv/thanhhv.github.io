+++
title = "Bài 2 — Consumer Group & Rebalance"
date = "2026-06-03T08:00:00+07:00"
draft = false
tags = ["backend", "kafka"]
series = ["Kafka Thực Chiến"]
+++

## Khái niệm

**Consumer group** là tập các consumer cùng `group.id`, hợp sức tiêu thụ một topic. Kafka tự chia partition cho các thành viên — gọi là **partition assignment**.

Quan hệ then chốt:

```
Trong 1 group:     1 partition  -> tối đa 1 consumer   (KHÔNG phải 1-n)
                   1 consumer   -> có thể N partition

Trên nhiều group:  1 partition  -> N consumer (mỗi group 1 đứa)   (pub/sub)
```

Hệ quả thực chiến: **số consumer hữu ích tối đa trong một group = số partition.** Topic 3 partition, group 5 consumer → 3 đứa làm việc, 2 đứa ngồi không. **Partition là trần scale của consumer** — muốn xử lý song song hơn phải tăng partition trước.

Cách scale: **không cần sửa code**, chỉ cần chạy nhiều instance cùng `group.id`. Kafka lo phần chia partition.

## Rebalance

Là quá trình Kafka **chia lại** partition khi đội hình thay đổi. Nguyên nhân:

1. Consumer join / leave / **crash / treo** quá `session.timeout.ms` hoặc `max.poll.interval.ms`.
2. Số partition của topic tăng.
3. Consumer đổi danh sách topic đăng ký (subscription).

### Cơ chế ngầm

- Mỗi group có một broker đóng vai **group coordinator** (người chấm công).
- Mỗi consumer định kỳ gửi **heartbeat** lên coordinator.
- Không heartbeat quá `session.timeout.ms` (~45s) → coi là chết → rebalance.

> **Broker ≠ Consumer.** Broker là server Kafka (phía lưu trữ). Consumer là chương trình của bạn (phía ứng dụng). Coordinator là một *broker* được chỉ định quản lý một *group consumer*.

### Hai mốc thời gian hay nhầm (câu hỏi phỏng vấn kinh điển)

- `session.timeout.ms`: lâu nhất không có **heartbeat** thì bị coi là chết. Heartbeat chạy ở **thread nền**.
- `max.poll.interval.ms` (mặc định 5 phút): lâu nhất giữa 2 lần gọi `poll()`. Nếu xử lý một batch **quá lâu** vượt mốc này → consumer bị đá khỏi group **dù vẫn sống** (vẫn heartbeat). Đây là nguyên nhân kinh điển của "rebalance lặp vô tận".

## Chiến lược chia partition (assignment strategy)

- **`range` / `roundrobin`** (eager): khi rebalance, **mọi** consumer trả hết partition rồi nhận lại → "stop-the-world" (ngừng consume vài trăm ms–vài giây, không phải kill tiến trình).
- **`cooperative-sticky`** (hiện đại, nên dùng): chỉ thu hồi **vừa đủ** partition cần di chuyển; các consumer khác vẫn chạy không gián đoạn. Trong log thấy "thu hồi 1, giao 0" nghĩa là "giữ nguyên phần đang có, không nhận thêm".

`range` còn dễ lệch tải khi group đăng ký nhiều topic (ưu tiên consumer đầu danh sách).

## Cạm bẫy thực chiến

- Đọc trạng thái group trên UI là **"stable"** KHÔNG có nghĩa tiến trình còn sống — dùng `ps` để chắc chắn. Sau khi consumer chết, metadata group còn tồn tại tới khi hết session timeout.
- Xử lý chậm → vi phạm `max.poll.interval.ms` → rebalance → throughput tụt dù CPU rảnh. Phải tune mốc này khớp thời gian xử lý thực tế.
- Consumer ID trên UI dạng `rdkafka-<uuid>` là tự sinh; đặt `client.id` có ý nghĩa để dễ debug.

## Câu hỏi phỏng vấn

> *Vì sao rebalance nguy hiểm?*
> → Trong lúc rebalance (nhất là eager), **không message nào được xử lý**. Rebalance liên tục (do xử lý chậm hoặc scale dồn dập) làm throughput sụp dù tài nguyên còn dư. Đó là lý do cooperative-sticky ra đời.

## Tóm tắt

Consumer group = chia tải (queue); nhiều group = phát tán (pub/sub). Một partition chỉ thuộc một consumer trong group — vừa là cơ chế chia tải vừa là cách giữ thứ tự. Rebalance là con dao hai lưỡi: cần thiết nhưng tốn kém; cooperative-sticky giảm đau.
