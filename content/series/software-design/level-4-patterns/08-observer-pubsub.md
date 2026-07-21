+++
title = "4.8 — Observer & pub/sub: tách người gây chuyện khỏi người quan tâm"
date = "2026-07-17T11:20:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## 1. Problem Statement — code evolution

Quay lại `ConfirmOrder` (1.2). Khi đó ta tách `Notifier` ra sau interface — nhưng service vẫn *gọi* notifier. Giờ nghiệp vụ phình theo hướng khác: sau khi xác nhận đơn, cần **thêm nhiều việc ăn theo**:

```go
// ❌ V1 — use case chính trở thành danh bạ của mọi bên quan tâm
func (s *Service) ConfirmOrder(ctx context.Context, orderID string) error {
    /* ... load, confirm, save — NGHIỆP VỤ LÕI, 10 dòng ... */

    // và cái đuôi mọc dài mãi:
    s.notifier.OrderConfirmed(ctx, o)          // gửi mail
    s.loyalty.AddPoints(ctx, o.CustomerID, o.Total())   // cộng điểm
    s.analytics.Track(ctx, "order_confirmed", o)        // sự kiện analytics
    s.recommender.InvalidateCache(ctx, o.CustomerID)    // xóa cache gợi ý
    s.fraud.ReportSignal(ctx, o)                        // tín hiệu chống gian lận
    return nil
}
```

Quan sát bệnh sinh: mỗi bên quan tâm mới (team Loyalty, team Analytics...) = **mở use case lõi ra sửa** + thêm một dependency vào struct — trong khi nghiệp vụ "xác nhận đơn" không hề đổi. Mũi tên phụ thuộc trỏ **sai chiều**: module quan trọng nhất (order) phải biết mọi module ăn theo (analytics, recommender — những thứ đổi xoành xoạch). Vi phạm OCP theo trục "bên quan tâm", vi phạm DIP về chiều ổn định. Và câu hỏi khó chịu bắt đầu xuất hiện: *lỗi cộng điểm có nên làm hỏng việc xác nhận đơn không?* — trong V1, có, dù chẳng ai chủ đích quyết như vậy.

## 2. Pattern — đảo loa phóng thanh

**Intent (GoF)**: quan hệ một-nhiều giữa các object — object trung tâm (subject) đổi trạng thái thì mọi object đăng ký (observer) được báo, **mà subject không biết cụ thể ai đang nghe**.

```go
// ✅ V2 — Observer trong Go: subject chỉ biết MỘT interface hẹp
type OrderObserver interface {
    OrderConfirmed(ctx context.Context, o *Order)
}

type Service struct {
    repo      Repository
    observers []OrderObserver          // danh sách MỞ — đăng ký lúc wiring
}

func (s *Service) ConfirmOrder(ctx context.Context, orderID string) error {
    /* ... nghiệp vụ lõi y nguyên ... */
    for _, ob := range s.observers {
        ob.OrderConfirmed(ctx, o)      // subject phát loa, không biết ai nghe
    }
    return nil
}

// main.go — bên quan tâm TỰ đến đăng ký; order không import ai cả:
svc := order.NewService(repo,
    loyaltyObserver, analyticsObserver, fraudObserver)
```

Mũi tên đảo xong: thêm bên quan tâm = viết observer mới + một dòng wiring — use case lõi **đóng**. Team Analytics tự sở hữu observer của mình, không cần PR vào module order.

Dạng Go nhẹ hơn khi observer không cần định danh: `[]func(ctx, *Order)` — callback list, đủ cho nội bộ một package. Dạng có tổ chức hơn khi nhiều loại sự kiện: **event bus trong process** — subject `Publish(OrderConfirmed{...})`, các bên `Subscribe`. Ba mức cùng một pattern; leo theo số loại sự kiện và số bên nghe.

## 3. Những câu hỏi khó — nơi Observer thật sự được thiết kế

Cấu trúc ở trên ai cũng viết được trong 10 phút. Thiết kế thật nằm ở các quyết định ngữ nghĩa mà V1 chưa từng phải trả lời tường minh:

**(a) Đồng bộ hay bất đồng bộ?** Loop gọi observer tuần tự: đơn giản, thứ tự xác định — nhưng use case giờ *chờ* analytics ghi xong; một observer chậm kéo cả request. Bắn goroutine cho mỗi observer: nhanh — nhưng ai recover panic? request kết thúc thì `ctx` bị cancel, observer đang chạy dở thì sao? Lời khuyên thực dụng: **mặc định đồng bộ** (dễ suy luận, dễ debug), chuyển từng observer chậm sang hàng đợi nội bộ (channel + worker) khi *đo thấy* đau — và lúc đó observer phải tự quản context/retry của nó.

**(b) Observer lỗi thì subject làm gì?** Ba lựa chọn, phải chọn tường minh: lan lỗi (analytics hỏng làm hỏng confirm — hiếm khi đúng), nuốt + log + metric (mặc định hợp lý cho side effect phụ), hay chặn (chỉ khi observer là nghiệp vụ bắt buộc — mà nếu bắt buộc, có nên là observer không?). Quy tắc chẩn đoán: **thứ bắt buộc-cùng-thành-bại thuộc về use case; observer là chỗ của thứ được-phép-trễ-hoặc-hỏng.** Interface trả `void` thay vì `error` chính là cách *tuyên bố* ngữ nghĩa đó trong chữ ký (như V2 ở trên).

**(c) Nghe rồi thì dữ liệu ở đâu?** Observer nhận `*Order` mutable → observer sửa được order — coupling tàng hình quay lại (1.5). Chuẩn: **sự kiện là dữ liệu bất biến, quá khứ, đặt tên quá khứ**: `OrderConfirmed{OrderID, Total, At}` — bản ghi việc-đã-xảy-ra, không phải cánh tay vào ruột subject. Đây là bước chuyển tư duy từ "observer nhìn object" sang "**event** như một giá trị" — nền của Domain Events (4.13) và event-driven (Level 5).

## 4. Từ Observer đến pub/sub — cùng ý tưởng, khác quy mô và bảo đảm

| | Observer (in-process) | Pub/sub qua broker (Kafka/NATS/RabbitMQ) |
|---|---|---|
| Ai giữ danh sách người nghe | Subject (slice/bus trong RAM) | Broker — publisher và subscriber không biết nhau *ở mức process* |
| Mất mát khi crash | Mất — RAM là hết | Broker giữ, consumer đọc lại; replay được (Kafka) |
| Thứ tự & delivery | Gọi hàm — chính xác một lần, tuần tự | At-least-once là chuẩn thực tế → **consumer phải idempotent**; thứ tự chỉ trong partition |
| Transaction với nghiệp vụ | Cùng process — cùng transaction được | **Không** — DB commit và publish là hai hệ; khoảng hở này chính là bài toán Outbox (4.13) |
| Chi phí vận hành | ~0 | Cluster, monitoring, schema registry, DLQ... |

Bài học quan trọng nhất của bảng: **đừng nhảy lên broker vì "cho giống event-driven"**. Observer in-process giải quyết coupling y như pub/sub — với chi phí vận hành bằng không và ngữ nghĩa gọi hàm dễ suy luận. Broker mua thêm *độ bền, replay, tách deploy* — trả bằng cả một tầng phức tạp phân tán. Leo khi cần bảo đảm đó thật, và Level 5 sẽ mổ kỹ phần trả giá.

## 5. Production sightings

- **`context.Context` cancellation**: observer cài sâu trong stdlib — `ctx.Done()` trả channel, mọi goroutine quan tâm `select` trên đó; `WithCancel` là subject phát tín hiệu. Chọn channel thay callback vì channel compose được với `select` — quyết định thiết kế đặc trưng Go.
- **Kubernetes watch/informer**: Observer công nghiệp quy mô lớn nhất bạn từng gặp — controller `Watch` API server, nhận stream sự kiện Added/Modified/Deleted; toàn bộ mô hình reconcile (1.3 đã nhắc: coupling qua desired state) đứng trên nó. Chi tiết đáng học: informer cache + resync — vì họ *biết* watch có thể đứt và sự kiện có thể mất, hệ được thiết kế **level-based** (đối chiếu trạng thái) thay vì edge-based (tin từng sự kiện) — bài học chống lại niềm tin ngây thơ vào delivery.
- **Node.js `EventEmitter`**: Observer là *xương sống văn hóa* của Node (`server.on("request")`, stream events). Mặt tối nổi tiếng: emitter nhận listener nhưng không quản lifecycle → **listener leak** (quên `off` khi component chết — memory leak kinh điển); lỗi trong listener sync có thể crash process (`error` event không ai nghe). Go tránh được một phần nhờ không có "bus mặc định trên mọi object" — bạn phải tự dựng, nên tự nghĩ về lifecycle.
- **Prometheus scrape — phản ví dụ chủ đích**: metric *không* dùng push/observer mà dùng **pull** — Prometheus chủ động đến hỏi. Lý do: hệ giám sát cần biết "target chết" (pull fail = tín hiệu), push thì im lặng không phân biệt được chết hay không có gì để nói. Nhắc để thấy: một-nhiều không mặc nhiên nghĩa là push; chiều chủ động cũng là quyết định thiết kế.

## 6. Anti-pattern

- **Luồng nghiệp vụ tàng hình** (đã cảnh báo ở 1.3, giờ đủ ngữ cảnh): nghiệp vụ *bắt buộc tuần tự* bị cắt thành chuỗi event — confirm phát event → observer trừ kho phát event → observer thu tiền... Đọc code không ai thấy luồng; debug là khảo cổ học; thứ tự và transaction không ai bảo đảm. **Quy trình có thứ tự-và-ràng-buộc là một use case (gọi hàm tường minh) hoặc một Saga có điều phối (4.13) — không phải chuỗi domino observer.**
- **Event làm remote control**: `PleaseSendEmailEvent` — mệnh lệnh đội lốt sự kiện; người phát *biết và muốn* đúng một người nghe làm đúng một việc → đó là lời gọi hàm bị làm gián tiếp vô cớ (Command, 4.9, còn hợp hơn). Sự kiện đúng nghĩa kể *điều đã xảy ra*, không ra lệnh *việc phải làm*.
- **Observer sửa subject**: nghe `OrderConfirmed` rồi gọi ngược `order.AddDiscount(...)` — vòng lặp phản hồi, thứ tự observer bỗng quyết định kết quả nghiệp vụ. Event bất biến + observer chỉ tác động ra *ngoài* subject.
- **Nghe hộ tương lai**: dựng event bus + phát đủ loại event "biết đâu sau này cần" khi chưa có người nghe nào — YAGNI; mỗi event công bố là một hợp đồng phải nuôi.

## 7. Khi nào KHÔNG dùng

Một bên quan tâm duy nhất, và về nghiệp vụ nó *thuộc về* luồng chính → gọi hàm thẳng (V1 với một dòng đuôi là code tốt!). Quy trình cần bảo đảm giao dịch cùng thành bại → use case/transaction, không observer. Ứng dụng CRUD nhỏ → danh sách observer rỗng là kiến trúc đúng. Ngưỡng vào pattern: **từ bên quan tâm thứ hai trở đi, và chúng thật sự "phụ" so với nghiệp vụ lõi** — đúng như mọi pattern trong tài liệu này: bằng chứng trước, cấu trúc sau.

---

*Tiếp theo: [4.9 — Command, Chain of Responsibility & Mediator](/series/software-design/level-4-patterns/09-command-chain-mediator/)*
