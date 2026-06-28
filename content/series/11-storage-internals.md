+++
title = "Bài 11 (Nâng cao) — Cơ chế lưu trữ: vì sao Kafka nhanh"
date = "2026-06-25T08:00:00+07:00"
draft = false
tags = ["backend", "kafka"]
series = ["Kafka Thực Chiến"]
+++

Tốc độ Kafka đến từ **bốn quyết định thiết kế cộng hưởng**, không phải một mẹo.

## 1. Ghi tuần tự (sequential write)

Trực giác sai: "ghi đĩa chậm, phải dùng RAM". Thực tế: **ghi đĩa *tuần tự* nhanh hơn ghi RAM *ngẫu nhiên*.**
- Random access (DB cập nhật chỗ này chỗ kia): tốn seek, chậm cả HDD lẫn SSD.
- Sequential write: chỉ ghi nối tiếp vào cuối file, không seek — HDD ~100s MB/s, SSD cao hơn.

Kafka chọn **log append-only**: message mới luôn ghi vào *cuối*, không sửa/chèn. Đây là lý do sâu xa Kafka không cho xóa/sửa message — ràng buộc có chủ đích để đổi lấy tốc độ.

## 2. Partition = file, chia thành Segment

Mỗi partition là một thư mục. Log chia thành nhiều **segment** (mặc định 1GB — `KAFKA_LOG_SEGMENT_BYTES`). Chỉ segment cuối (**active**) được ghi; segment cũ đóng băng chỉ đọc.

Vì sao chia segment? Để **xóa dữ liệu cũ rẻ**: hết retention, chỉ việc xóa nguyên file segment cũ — không phải xóa từng message.

Mỗi segment gồm bộ 3 file (tên = offset đầu tiên):
- `.log` — message thật.
- `.index` — ánh xạ **offset → vị trí byte** (để nhảy thẳng tới offset cần, không quét từ đầu).
- `.timeindex` — ánh xạ **timestamp → offset** (để seek theo thời gian).

Index dùng **binary search** và là **sparse** (thưa) để nhỏ, nhét vừa RAM.

## 3. Page Cache — giao việc cache cho OS

Khi ghi, Kafka **không tự ghi thẳng đĩa** mà ghi vào **page cache của OS** (RAM kernel cache file). Kernel flush xuống đĩa sau.
- **Ghi:** trả về gần như tức thì (chỉ vào RAM).
- **Đọc:** consumer theo kịp thường đọc message vừa ghi → còn trong page cache → đọc từ RAM, **không chạm đĩa**.

Hệ quả gây sốc: **Kafka trên JVM nhưng tốc độ không phụ thuộc heap JVM** — nó cố tình giao cache cho OS. Vì vậy broker dùng **heap nhỏ** (vài GB), để **phần lớn RAM cho page cache**. Cho JVM heap thật to là cấu hình sai phổ biến.

Nối Bài 9: consumer **lag cao** đọc dữ liệu cũ không còn trong cache → cache miss → đọc đĩa → tranh I/O với việc ghi → **làm chậm cả cluster**, không chỉ trễ consumer đó.

## 4. Zero-copy (`sendfile`)

Đọc thông thường tốn **4 lần copy** + 2 lần chuyển ngữ cảnh:
```
đĩa → page cache → buffer ứng dụng → socket buffer → card mạng
```
Dữ liệu đi vòng lên tầng ứng dụng rồi xuống lại dù ứng dụng chẳng làm gì với nó.

Kafka dùng `sendfile` (zero-copy): kernel chuyển thẳng
```
đĩa → page cache → card mạng
```
Bỏ 2 lần copy + 2 lần chuyển ngữ cảnh. Điều kiện: message không cần biến đổi giữa chừng → một lý do Kafka khuyến khích nén/giải nén ở **client**, để broker giữ nguyên byte và sendfile thẳng.

## Cộng hưởng

```
append-only  → ghi tuần tự (nhanh)
segment      → xóa/quản lý rẻ + index nhảy nhanh
page cache   → ghi/đọc qua RAM của OS
sendfile     → đọc → mạng không qua tầng ứng dụng
batch + nén  → (Bài 9) ít request, ít byte
```

Gộp lại mới ra throughput hàng triệu msg/s. Toàn bộ đánh đổi: *Kafka từ bỏ khả năng sửa/xóa/random-access của một DB để đổi lấy tốc độ tuần tự của một log.*

## Khám phá thực tế

```bash
# cấu trúc thư mục partition
docker exec -it kafka-1 ls -la /var/lib/kafka/data/vault-0/
# -> 0000...0.log, .index, .timeindex

# đọc nội dung file .log
docker exec -it kafka-1 kafka-dump-log \
  --files /var/lib/kafka/data/vault-0/00000000000000000000.log --print-data-log

# xem sparse index
docker exec -it kafka-1 kafka-dump-log \
  --files /var/lib/kafka/data/vault-0/00000000000000000000.index
```

## Câu hỏi phỏng vấn

> *Kafka chạy trên JVM, vì sao khuyên heap nhỏ cho broker?*
> → Kafka dựa vào **page cache của OS** để đọc/ghi nhanh, không tự cache trong heap. Heap to ăn mất RAM lẽ ra dành cho page cache, và gây áp lực GC. Heap nhỏ + RAM còn lại cho OS cache là tối ưu.

## Tóm tắt

Bốn trụ cột: sequential write, segment + sparse index, page cache, zero-copy sendfile. Hiểu chúng là hiểu bản chất tốc độ Kafka và lý giải được nhiều quyết định vận hành (heap nhỏ, nén ở client, vì sao lag cao hại cả cluster).
