+++
title = "5.2 — Event-driven Architecture & CQRS: khi nào đáng trả giá cho bất đồng bộ"
date = "2026-07-17T12:40:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## 1. Event-driven Architecture — Observer (4.8) phóng to hết cỡ

### Con đường leo thang — và trạm dừng ở mỗi bậc

Toàn bộ nền đã xây: event là dữ liệu bất biến quá khứ (4.8), domain events sinh trong transaction (4.13), outbox đưa event rời process an toàn (4.13), consumer idempotent (4.9). Event-driven **architecture** là quyết định dùng chuỗi đó làm **xương sống liên lạc giữa các bounded context** (5.0): context phát sự kiện về điều đã xảy ra trong nó; context khác phản ứng — không ai gọi trực tiếp ai.

Cái mua được — ba thứ, đều là coupling (1.3) được cắt ở tầm hệ thống:

- **Tách deploy & availability**: Loyalty chết 2 giờ — Order vẫn nhận đơn bình thường; event nằm chờ trong broker, Loyalty dậy xử lý tiếp. So với chuỗi gọi REST đồng bộ (Order → Loyalty → chết theo), đây là khác biệt sống còn theo nghĩa đen.
- **Thêm consumer không sửa producer**: OCP (2.2) ở tầm tổ chức — team Analytics tự subscribe `OrderConfirmed`, team Order không cần biết, không cần PR, không cần họp.
- **Dòng sự kiện là dữ liệu hạng nhất**: replay được (Kafka), audit được, dựng view mới từ lịch sử được.

Cái trả — nói đủ, vì đây là chỗ các bài "hướng dẫn EDA" hay im lặng:

- **Eventual consistency xuyên context là mặc định**: "đơn confirmed rồi mà điểm chưa cộng" trong N giây/phút — nghiệp vụ *phải chấp nhận và UI phải thiết kế theo* ("điểm sẽ được cộng trong ít phút"). Không đàm phán được điều này với business → đừng event-driven chỗ đó.
- **Debug đổi bản chất**: từ đọc stack trace sang khảo cổ dòng event — không có tracing phân tán (trace-id chảy trong event header), tìm "vì sao đơn này kẹt" là mò kim. Observability không còn là "nice to have" — nó là **điều kiện vào cửa**.
- **Schema event là hợp đồng public vĩnh viễn** (4.8 đã cảnh báo "mỗi event là hợp đồng phải nuôi"): đổi field là vỡ consumer không quen biết — cần schema registry, versioning, và kỷ luật chỉ-thêm-không-sửa.
- **Mọi nghĩa vụ 4.13**: at-least-once, idempotency, thứ tự chỉ trong partition — mỗi consumer viết như thể event đến trùng, đến muộn, đến lộn xứ tự.

**Trạm dừng** — nhắc lại phân tuyến đã cắm ở 4.8/4.13, vì lỗi phổ biến nhất là leo quá bậc: cùng process → observer/domain events in-process (đủ cho monolith modular — và **monolith modular với event nội bộ là kiến trúc bị đánh giá thấp nhất thập kỷ**); cần bền + tách deploy thật → broker + outbox; quy trình nhiều bước có bù → Saga orchestration (4.13). Không leo vì "cho giống Netflix".

## 2. CQRS — tách đường ghi khỏi đường đọc

### Bài toán có thật đằng sau cái tên đáng sợ

Đã gặp mầm của nó hai lần: 2.6 (Tell Don't Ask — "đường ghi thì Tell, đường đọc thì Ask") và 4.12a ("query phức tạp cho hiển thị đi cổng riêng"). Bài toán cụ thể: model aggregate `Order` (5.0) tối ưu cho **gác invariant khi ghi** — nhưng màn hình danh sách đơn cần join 6 bảng, filter 8 chiều, phân trang — nhồi các query đó vào repository của aggregate là phá nó (4.12c). Hai nhu cầu **hình dạng dữ liệu khác nhau** → cho chúng hai mô hình:

```
Command side (ghi):  use case → aggregate (gác luật) → repository → DB ghi
Query side (đọc):    handler → query service → SQL/view tối ưu đọc → DTO phẳng ra thẳng UI
                     (KHÔNG qua aggregate, KHÔNG qua repository của nó — đọc là đọc)
```

**CQRS mức 1 — cùng một DB, hai đường code** — chỉ có vậy, và mức này **rẻ, lành, đáng dùng rộng rãi**: query service đọc thẳng replica/view, trả DTO; aggregate thoát khỏi gánh query; hai đường tiến hóa độc lập (thêm màn hình mới không đụng domain). Nhiều codebase đang làm điều này mà không biết tên nó là CQRS.

**CQRS mức 2 — hai mô hình dữ liệu, đồng bộ bằng event**: đường ghi phát event (outbox 4.13) → projector dựng **read model** riêng (bảng phi chuẩn hóa, Elasticsearch, cache) tối ưu cho từng màn hình. Mua: đọc nhanh tùy ý, scale đọc/ghi độc lập. Trả: eventual consistency ngay trong một context ("vừa đặt đơn xong, refresh chưa thấy" — cần UI xử lý), cộng toàn bộ hóa đơn vận hành event pipeline ở mục 1. Mức này chỉ đáng khi tải đọc/độ phức tạp view thật sự vượt mức 1.

**Event Sourcing — người anh em hay bị đánh đồng** (một đoạn cho đúng vị trí): thay vì lưu *trạng thái hiện tại*, lưu *chuỗi event* làm source of truth; trạng thái = replay (snapshot để tăng tốc — Memento 4.10). Mạnh nhất về audit/time-travel (tự nhiên có "tài khoản này vì sao ra số dư này"); đắt nhất về versioning event dài hạn và độ lạ lẫm với tooling quen thuộc. ES *thường đi kèm* CQRS (read model dựng từ event) nhưng là quyết định **độc lập và nặng hơn nhiều** — ngách hợp: ledger, tài chính, hệ cần audit tuyệt đối. Ngoài ngách: DB quan hệ + audit log là đủ và tỉnh táo hơn.

## 3. Bảng quyết định tổng — Level 5 trên một trang

```
Nghiệp vụ mỏng, CRUD           → monolith 2 file + sqlc (5.1). Dừng ở đây là chuyên nghiệp.
Nghiệp vụ dày, một team        → monolith modular: context (5.0) + hexagonal (5.1)
                                 + domain events in-process (4.13). ĐIỂM NGỌT của đa số công ty.
Màn hình đọc phức tạp          → + CQRS mức 1 (hai đường code, một DB)
Nhiều team giẫm chân deploy    → tách service THEO RANH GIỚI CONTEXT + event qua broker
                                 + outbox + saga cho quy trình xuyên service
Tải đọc khổng lồ / audit tuyệt đối → + CQRS mức 2 / event sourcing — có ngách, có giá
```

Mỗi mũi tên đi xuống là một bậc chi phí vận hành và chi phí hiểu — **đi xuống khi có bằng chứng bậc trên hết chịu nổi, đúng nguyên tắc đã theo suốt từ chương 1.1**: kiến trúc cũng chỉ là quản lý chi phí thay đổi, ở mẫu số lớn hơn.

---

*Tiếp theo: [5.3 — Production Case Studies](/series/software-design/level-5-architecture/03-case-studies/) — soi toàn bộ tài liệu vào code thật đang chạy khắp thế giới.*
