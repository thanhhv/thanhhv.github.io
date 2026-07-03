+++
title = "Bài 1 — Topic, Partition, Producer & Consumer"
date = "2026-05-01T08:00:00+07:00"
draft = false
tags = ["backend", "kafka"]
series = ["Kafka Thực Chiến"]
+++

## Khái niệm nền tảng

**Topic** là một "kênh" logic chứa message, ví dụ `orders` chứa các sự kiện đơn hàng.

**Partition** là điểm mấu chốt phân biệt Kafka với hàng đợi thường. Mỗi topic được chia thành nhiều partition. Mỗi partition là một **log ghi nối tiếp (append-only)**: message có thứ tự tuyệt đối trong partition, và mỗi message mang một số thứ tự gọi là **offset** (0, 1, 2, …).

**Producer** là chương trình ghi message vào topic. **Consumer** là chương trình đọc message ra để xử lý.

Ba điều phải khắc cốt ghi tâm ngay từ đầu:

1. **Thứ tự chỉ được đảm bảo trong MỘT partition**, không phải toàn topic.
2. Producer quyết định message vào partition nào dựa trên **key**: cùng key → cùng partition; không key → chia round-robin/sticky.
3. Kafka **không xóa message sau khi đọc** (khác RabbitMQ). Message nằm đó đến khi hết **retention** (mặc định 7 ngày). Consumer chỉ di chuyển con trỏ offset.

## Offset — khái niệm cốt lõi

Mỗi message có một offset riêng trong từng partition, tăng dần, không bao giờ đổi. Consumer xử lý đến đâu thì "đánh dấu" (commit) offset đến đó. Đây là nền tảng của toàn bộ delivery semantics ở Bài 4.

## Thực hành (Go, confluent-kafka-go)

Producer cơ bản — những điểm quan trọng:

```go
p, _ := kafka.NewProducer(&kafka.ConfigMap{
    "bootstrap.servers": "localhost:9092,localhost:9093,localhost:9094",
    "acks":              "all", // chờ leader + ISR ghi xong (an toàn nhất)
})

// Delivery report (callback) là cách DUY NHẤT biết message ghi thành công ở
// partition/offset nào. Phải kiểm tra TopicPartition.Error.
go func() {
    for e := range p.Events() {
        if m, ok := e.(*kafka.Message); ok {
            if m.TopicPartition.Error != nil { /* xử lý lỗi */ }
        }
    }
}()

p.Produce(&kafka.Message{
    TopicPartition: kafka.TopicPartition{Topic: &topic, Partition: kafka.PartitionAny},
    Key:            []byte("user-42"), // cùng key -> cùng partition -> giữ thứ tự
    Value:          []byte("payload"),
}, nil)

p.Flush(5000) // BẮT BUỘC: bỏ Flush = mất message khi chương trình thoát sớm
```

## Cạm bẫy thực chiến

- **Quên `Flush()`** → message còn trong buffer khi tiến trình thoát = mất.
- **Không kiểm tra delivery report** → "tưởng gửi rồi mà mất". Intern hay mắc.
- **Tạo topic quên `--replication-factor`** → mặc định là **1** (KHÔNG bằng số broker!). Broker chết là mất sạch partition đó. Luôn ghi rõ partitions và replication-factor.
- Mặc định production: `--replication-factor 3` (chịu được mất 1 broker).
- `auto.create.topics.enable=false` ở production để tránh tạo topic rác do gõ sai tên.

## Partition vs Replication-factor (rất hay nhầm)

- **Partition** = chia nhỏ để song song. Là con số bạn THẤY ở client (`partition=0,1,2`).
- **Replication-factor** = số bản sao của mỗi partition để chống mất dữ liệu. Là các bản ẩn (leader + follower), client KHÔNG thấy.
- `kafka-topics --describe`: dòng `Leader / Replicas / Isr` cho biết bản nào ở broker nào.

## Câu hỏi phỏng vấn

> *Muốn mọi sự kiện của `user_id=42` được xử lý đúng thứ tự, làm thế nào?*
> → Đặt **key = "42"**. Cùng key → cùng partition → thứ tự được đảm bảo. (Lưu ý: chỉ đúng khi số partition không đổi.)

## Tóm tắt

Partition quyết định bạn thấy bao nhiêu phần song song; replication-factor quyết định mỗi partition có bao nhiêu bản dự phòng. Offset là con trỏ tiến độ. Key là công cụ giữ thứ tự. Kafka là log bền vững, không phải hàng đợi xóa-sau-khi-đọc.
