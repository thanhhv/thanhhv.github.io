+++
title = "Phần 5 — Data Layer"
date = "2026-07-13T08:10:00+07:00"
draft = false
tags = ["backend", "system-design"]
series = ["System Design — Tư Duy Thiết Kế Hệ Thống"]
+++

> Chủ đề: PostgreSQL, MySQL, MongoDB, Redis, ClickHouse, Elasticsearch — và quan trọng nhất: **khi nào chọn cái gì**.

## Luận điểm trung tâm của phần này

Không có database "tốt nhất" — chỉ có database có **mô hình lưu trữ và truy cập khớp với workload**. Mọi khác biệt giữa các hệ quy về vài quyết định gốc mà mỗi engine đã chọn *thay bạn* từ ngày nó được thiết kế:

1. **Cấu trúc lưu trữ:** B-tree (đọc điểm nhanh, ghi tại chỗ) vs LSM-tree (ghi tuần tự nhanh, đọc phải dò nhiều tầng) vs cột (quét/nén cực nhanh, sửa từng hàng cực dở) vs inverted index (tìm "chứa từ X" nhanh, không phải nơi lưu sự thật).
2. **Đơn vị nhất quán:** transaction đa hàng (RDBMS) vs document đơn (MongoDB) vs thao tác đơn (Redis) vs không có (ClickHouse, ES).
3. **Trục tối ưu:** đọc điểm vs ghi dòng chảy vs tổng hợp vs tìm kiếm — không engine nào tối ưu cả bốn, vì các trục mâu thuẫn nhau ở tầng vật lý.

Hiểu ba quyết định gốc này của từng engine thì không cần học thuộc "khi nào dùng gì" — câu trả lời tự suy ra từ workload.

## Mục lục

- [5.1. PostgreSQL — mặc định đúng cho dữ liệu nghiệp vụ](/series/system-design/05-data-layer/01-postgresql/)
- [5.2. MySQL — người anh em song sinh khác tính cách](/series/system-design/05-data-layer/02-mysql/)
- [5.3. MongoDB — khi dữ liệu thật sự là document](/series/system-design/05-data-layer/03-mongodb/)
- [5.4. Redis — cấu trúc dữ liệu trong RAM](/series/system-design/05-data-layer/04-redis/)
- [5.5. ClickHouse — cỗ máy quét tỷ hàng](/series/system-design/05-data-layer/05-clickhouse/)
- [5.6. Elasticsearch — index, không phải database](/series/system-design/05-data-layer/06-elasticsearch/)
- [5.7. So sánh & khung quyết định lựa chọn](/series/system-design/05-data-layer/07-so-sanh-lua-chon/) — **đọc chương này nếu chỉ đọc một chương**

## Nguyên tắc phối hợp (polyglot có kỷ luật)

1. **Một nguồn sự thật** (thường PostgreSQL/MySQL) — các hệ còn lại là **dẫn xuất** (cache, index, projection) dựng lại được từ nguồn ([CQRS, 12.8](/series/system-design/12-evolution/08-cqrs/)).
2. Mỗi hệ thêm vào là một nghề vận hành mới — thêm khi có bằng chứng workload, không phải khi đọc blog ([Phần 12, bài học 1](/series/system-design/12-evolution/00-tong-quan/)).
3. Đồng bộ nguồn → dẫn xuất qua CDC/outbox ([12.7](/series/system-design/12-evolution/07-kafka-event-driven/)), kèm đối soát drift định kỳ.

## Lưu ý về benchmark trong phần này

Mọi con số hiệu năng trong phần này là **bậc độ lớn định hướng** trên phần cứng server hiện đại (NVMe, RAM đủ cho working set), dùng để ước lượng nhanh theo tinh thần [chương 1.4](/series/system-design/01-foundations/04-scale-estimation-capacity-planning/). Chúng dao động vài lần theo schema, cấu hình, phiên bản — **luôn benchmark với workload thật của bạn** trước quyết định lớn.
