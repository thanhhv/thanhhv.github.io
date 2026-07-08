+++
title = "Chương 12 – Time: \"Bây giờ\" là một khái niệm nguy hiểm"
date = "2026-07-09T04:00:00+07:00"
draft = false
tags = ["backend", "distributed-systems"]
series = ["Distributed Systems"]
+++

## 1. Problem Statement

Trên một máy, `now()` có một giá trị và các sự kiện có thứ tự hiển nhiên. Trong hệ phân tán, mỗi node có đồng hồ riêng, **trôi khác nhau**, và câu hỏi tưởng tầm thường — *sự kiện A ở node 1 xảy ra trước hay sau sự kiện B ở node 2?* — trở nên không trả lời được bằng timestamp. Trong khi đó, hàng loạt cơ chế lại đang *ngầm* dựa vào timestamp: Last-Write-Wins (chương 05), TTL/lease (chương 07), log ordering, cronjob phối hợp, phát hiện timeout, chứng từ giao dịch.

Nếu lờ đi: LWW **xóa write mới bằng write cũ** vì node ghi sau có đồng hồ chạy chậm (mất dữ liệu âm thầm, không log, không alert); lease "còn 3 giây" theo node này đã hết từ 2 giây trước theo node kia → hai kẻ cùng giữ lock; log các service xếp sai thứ tự → debug ra kết luận nhân quả ngược. Đây là lớp bug khó chịu nhất ngành: **không tái hiện được, không để lại dấu vết, và chỉ xuất hiện khi clock lệch đủ nhiều.**

## 2. Physical Clock — vì sao không tin được

### 2.1. Clock Drift và NTP

Quartz oscillator trôi ~10–50 ppm (≈ 1–4 giây/ngày nếu thả rông). NTP kéo về theo nguồn chuẩn, nhưng: sai số phụ thuộc mạng (LAN ~ms, internet có thể chục–trăm ms); khi lệch lớn NTP có thể **step** — giật đồng hồ *lùi* (đo `duration = end - start` ra số **âm**; chữa: dùng monotonic clock — `CLOCK_MONOTONIC`, `System.nanoTime()` — cho mọi phép đo khoảng thời gian; wall clock chỉ dành cho "mấy giờ rồi", monotonic dành cho "bao lâu"); VM bị pause/migrate thức dậy với đồng hồ sai hàng giây; và **leap second** từng làm sập nhiều hệ lớn (2012) — nay xử lý bằng smearing (bôi 1 giây ra 24 giờ — nghĩa là trong ngày đó, đồng hồ của bạn *cố tình* sai tới 0.5s so với UTC thật).

Ngân sách sai số thực tế phải giả định khi thiết kế: cùng DC với NTP tốt ~0.5–10ms; xuyên internet: chục–trăm ms; trường hợp xấu (NTP hỏng âm thầm, config sai): **không giới hạn**. Kết luận thiết kế: *so sánh timestamp giữa hai máy chỉ có nghĩa khi chênh lệch lớn hơn nhiều lần ngân sách sai số* — và các quyết định đúng/sai dữ liệu không được phép dựa vào so sánh đó.

### 2.2. Hệ quả cụ thể lên các cơ chế đã học

- **LWW (Cassandra)**: node A clock nhanh 200ms; write "cũ" của A (theo thực tế) mang timestamp lớn hơn write "mới" của B → bản mới bị đè, vĩnh viễn, không dấu vết.
- **Lease/lock TTL (chương 07)**: an toàn của lease phụ thuộc hai bên *đồng ý về tốc độ thời gian* — GC pause + clock lệch ăn thủng lease; vì thế cần fencing token (không phụ thuộc clock) làm tầng bảo vệ cuối.
- **Debug xuyên service**: log timestamp lệch → dựng timeline sai → đổ lỗi nhầm service. Distributed tracing (quan hệ cha-con của span) đáng tin hơn timestamp — vì nó dùng *nhân quả*, không dùng đồng hồ. Đó chính là ý tưởng của §3.

## 3. Logical Clock — thay "mấy giờ" bằng "cái gì trước cái gì"

### 3.1. Quan hệ happens-before (Lamport, 1978)

Sự thật nền: điều duy nhất hai node *biết chắc* về thứ tự là **nhân quả**: (a) hai sự kiện cùng process — theo thứ tự chương trình; (b) gửi message xảy ra trước nhận message đó; (c) bắc cầu. Ký hiệu `A → B`. Hai sự kiện không có đường nhân quả nào nối: **concurrent** (`A ∥ B`) — không phải "cùng lúc" mà là "*không thể biết* và **không cần biết** cái nào trước". Đây là thay đổi tư duy quan trọng nhất chương: từ tổng-thứ-tự-theo-đồng-hồ (ảo tưởng) sang thứ-tự-bộ-phận-theo-nhân-quả (sự thật).

### 3.2. Lamport Clock — rẻ, một chiều

Mỗi node giữ một counter: tăng khi có sự kiện; gửi kèm message; nhận thì `C = max(C_local, C_msg) + 1`.

```
P1: 1 ──▶ 2(gửi) ────────▶ 3
P2:    1 ────▶ 4(nhận:max(1,2)+1... ) ──▶ 5(gửi)
P3: 1 ──────────────▶ 6(nhận)
```

Đảm bảo: `A → B ⟹ C(A) < C(B)`. **Chiều ngược sai**: `C(A) < C(B)` không suy ra A trước B (có thể concurrent). Nghĩa là Lamport clock **sắp thứ tự được nhưng không phát hiện được concurrent** — đủ cho: total order phân xử (kết hợp node ID phá hòa — dùng trong mutual exclusion, xếp thứ tự thao tác), version đơn điệu. Không đủ cho: phát hiện xung đột replica.

### 3.3. Vector Clock — đắt hơn, hai chiều

Mỗi node giữ **vector** N counter: `V[i]` = số sự kiện đã biết của node i. Gửi kèm vector; nhận thì max từng phần tử + tăng phần mình.

So sánh: `V(A) < V(B)` (mọi phần tử ≤, ít nhất một <) ⟺ `A → B`. Không so sánh được (mỗi bên lớn hơn ở một phần tử) ⟺ **concurrent — phát hiện được xung đột**. Đây chính là thứ Dynamo/Riak dùng: hai bản ghi có vector không so sánh được = xung đột thật sự → giữ cả hai (sibling) cho tầng trên hòa giải; vector so sánh được = một bản là quá khứ của bản kia → tự động lấy bản sau, *không mất write nào* (khác hẳn LWW).

Giá phải trả: O(N) metadata mỗi bản ghi, N = số node/client từng ghi — phình theo thời gian (Dynamo phải cắt tỉa vector cũ, chấp nhận rủi ro nhỏ hòa giải sai). Trade-off gọn: **Lamport = 1 số, thứ tự một chiều; Vector = N số, phát hiện concurrent.** Chọn theo việc: cần phân xử → Lamport; cần phát hiện xung đột → Vector.

### 3.4. Hybrid Logical Clock (HLC) — hai thế giới bắt tay

HLC = wall clock + counter logic: gần với thời gian thật (đọc được, so được với ngoài đời) nhưng vẫn bảo toàn happens-before khi clock vật lý trục trặc. CockroachDB, MongoDB, YugabyteDB dùng HLC làm timestamp giao dịch — lựa chọn mặc định tốt cho hệ mới cần "timestamp có nghĩa + nhân quả đúng".

## 4. TrueTime — mua sự thật bằng phần cứng

Spanner (Google) đi hướng ngược lại: thay vì né đồng hồ vật lý, **làm đồng hồ vật lý tốt đến mức tin được** — GPS + đồng hồ nguyên tử mỗi DC, và API trung thực về sự bất định: `TT.now()` trả về **khoảng** `[earliest, latest]` (ε điển hình 1–7ms) thay vì một con số dối trá.

Commit-wait: transaction lấy timestamp `s = TT.now().latest`, rồi **chờ đến khi `TT.now().earliest > s`** mới công bố commit — bảo đảm mọi transaction sau đó (theo thời gian thật, ở bất kỳ đâu trên Trái Đất) sẽ nhận timestamp lớn hơn → **external consistency** (linearizability toàn cầu) mà không cần node liên lạc nhau để xếp thứ tự. Cái giá: chờ trung bình ~ε mỗi commit + hạ tầng đồng hồ chuyên dụng. Bài học kiến trúc đẹp: *đo được độ bất định thì có thể chờ cho nó trôi qua* — biến vấn đề phần mềm thành hóa đơn phần cứng. AWS TimeSync ngày nay cũng cung cấp clock bound API (~µs–ms) — hướng này đang phổ cập dần.

## 5. Trade-off tổng hợp

| Cách tiếp cận | Chi phí | Được gì | Đại diện |
|---|---|---|---|
| Wall clock + NTP, tin luôn | ~0 | Đơn giản; sai âm thầm khi drift | LWW Cassandra (mặc định) |
| Lamport | 1 counter | Total order phân xử; không thấy concurrent | Nhiều hệ nội bộ |
| Vector | O(N)/bản ghi | Phát hiện xung đột, không mất write | Dynamo, Riak |
| HLC | ~1 timestamp | Nhân quả + đọc được như giờ thật | CockroachDB, MongoDB |
| TrueTime | Phần cứng + commit-wait | External consistency toàn cầu | Spanner |
| **Né hẳn: single writer/partition** | Thiết kế | Thứ tự tự nhiên theo vị trí trong log, **không cần clock** | Kafka partition, Raft log |

Dòng cuối đáng nhấn mạnh: **cách xử lý thời gian tốt nhất là thiết kế để không cần hỏi giờ** — dồn write cùng entity về một leader/partition (chương 05–06), thứ tự = vị trí trong log; sự kiện bất biến + version tăng đơn điệu thay cho timestamp.

## 6–8. Production Considerations & Best Practices

1. **Monotonic clock cho mọi phép đo duration** (timeout, latency, profiling). Wall clock cho hiển thị. Trộn lẫn là bug nằm vùng.
2. **Giám sát clock offset như một SLI hạng nhất**: node lệch NTP > ngưỡng (vd 50ms) → alert, thậm chí tự loại khỏi cluster (CockroachDB tự crash node lệch quá `max-offset` — hành vi đúng: chết to còn hơn sai âm thầm). NTP daemon chết âm thầm là sự cố có thật và phổ biến.
3. **Chạy nhiều nguồn NTP** (≥4 để phát hiện nguồn nói dối), trong cloud dùng dịch vụ time của nhà cung cấp (AWS TimeSync, GCP NTP có smearing — và **đừng trộn** nguồn smear với nguồn không smear).
4. **Không dùng timestamp làm ID duy nhất hay khóa sắp xếp toàn cục** — dùng ID sinh có cấu trúc (Snowflake = timestamp + node + sequence; UUIDv7) và hiểu rằng phần timestamp trong đó chỉ *xấp xỉ* thứ tự.
5. **Idempotency và fencing không được phụ thuộc clock** — TTL là tầng tiện lợi, token đơn điệu là tầng đúng đắn (chương 07–08).
6. **Kiểm thử với clock lệch**: chaos test chỉnh clock ±500ms trên một node staging — hệ của bạn làm gì? (Đa số team chưa từng thử; đa số hệ có bất ngờ.)
7. Log/trace: kèm **trace ID + span quan hệ** để dựng timeline bằng nhân quả; timestamp chỉ để tham khảo.

## 9. Anti-patterns

- `if (timestampA > timestampB)` giữa hai node để quyết định đúng/sai dữ liệu — chính là LWW tự chế, mất write âm thầm.
- Đo timeout/duration bằng wall clock — NTP step lùi → duration âm → hành vi không định nghĩa.
- Lease/lock chỉ dựa TTL không fencing (chương 07 §6 — kịch bản GC pause).
- Cronjob trên nhiều node "cùng chạy lúc 00:00" như một cơ chế phối hợp — clock lệch + không có lock = chạy đôi hoặc bỏ sót; dùng distributed lock hoặc một scheduler có leader.
- So sánh timestamp do **client** gửi lên với giờ server (client clock hoàn toàn không kiểm soát được — điện thoại lệch hàng giờ là bình thường).
- Tin rằng "trong một DC thì clock chuẩn" — VM pause, NTP hỏng, config sai không phân biệt DC xịn hay không.

## 10. Troubleshooting

| Triệu chứng | Nghi vấn clock | Xác nhận |
|---|---|---|
| Write "biến mất" không log lỗi (Cassandra/LWW store) | Node ghi có clock chậm → bị bản cũ timestamp cao đè | So `writetime()` của bản thắng với thời gian thật; kiểm NTP offset các node |
| Duration/latency âm hoặc lớn vô lý trong metric | Wall clock bị step giữa hai lần đo | Đổi sang monotonic; xem log chronyd/ntpd đúng thời điểm |
| Token/OTP/cert "hết hạn" ngay khi vừa cấp | Clock server cấp lệch với server kiểm | So offset hai node; chuẩn lại NTP |
| Hai node cùng nghĩ mình giữ lease | Clock lệch + GC pause ăn thủng TTL | Fencing token; đo pause + offset |
| Timeline log các service mâu thuẫn nhân quả | Timestamp lệch giữa các node | Dựng lại bằng trace ID; kiểm offset |
| Sự cố hàng loạt đúng 1 thời điểm lịch | Leap second / DST / NTP step đồng loạt | Đối chiếu lịch leap second, chuyển smearing |

---

## Tổng kết chương

### Những điều bắt buộc phải nhớ
1. Timestamp giữa hai máy không xác định thứ tự sự kiện; điều duy nhất chắc chắn là **nhân quả** (happens-before).
2. Wall clock để biết mấy giờ, **monotonic clock để đo bao lâu** — không bao giờ lẫn.
3. Lamport: thứ tự một chiều, 1 số; Vector: phát hiện concurrent, O(N); HLC: nhân quả + đọc được; TrueTime: mua external consistency bằng phần cứng + commit-wait.
4. LWW theo timestamp = chấp nhận mất write âm thầm khi clock lệch — quyết định phải được đưa ra *có ý thức*.
5. Thiết kế đẹp nhất là **né**: single writer per partition → thứ tự theo vị trí log, không cần đồng hồ.
6. Giám sát NTP offset như giám sát disk — nó hỏng âm thầm và kéo theo lớp bug không dấu vết.

### Hiểu lầm phổ biến
- "NTP rồi thì timestamp so được" — sai số ms–trăm ms vẫn quá lớn cho các write cách nhau µs–ms.
- "Vector clock là bản nâng cấp của Lamport, luôn tốt hơn" — trả O(N) metadata; nếu chỉ cần phân xử thứ tự, Lamport đủ và rẻ.
- "Spanner chứng minh strong consistency toàn cầu là miễn phí" — Spanner trả bằng phần cứng chuyên dụng + commit-wait mỗi transaction.
- "Concurrent nghĩa là cùng lúc" — nghĩa là *không có đường nhân quả nối* — hai sự kiện cách nhau 1 giờ vẫn có thể concurrent.

### Câu hỏi tự kiểm tra
1. Hai client cập nhật cùng profile qua hai node Cassandra, node A clock nhanh 300ms. Vẽ timeline một kịch bản bản cập nhật *mới hơn thực tế* bị mất. Đổi sang vector clock thì chuyện gì khác đi?
2. Vì sao đo timeout bằng wall clock là bug? Viết đoạn mã minh họa hành vi sai khi NTP step lùi 2 giây.
3. Cho 3 process với các sự kiện và message như §3.2 — gán Lamport clock và vector clock cho từng sự kiện; chỉ ra một cặp concurrent mà Lamport "xếp nhầm" thành có thứ tự.
4. Thiết kế job "gửi báo cáo 8h sáng, chạy đúng một lần" trên cluster 5 node — không dựa vào clock các node khớp nhau.

### Tài liệu kinh điển nên đọc
- **"Time, Clocks, and the Ordering of Events in a Distributed System" (Lamport, 1978)** — paper được trích dẫn nhiều nhất của ngành; định nghĩa happens-before, khai sinh logical clock. Đọc bản gốc — nó dễ đọc hơn danh tiếng của nó.
- **"Spanner: Google's Globally-Distributed Database" (2012)** — TrueTime và commit-wait; đọc cùng bài "Spanner, TrueTime & CAP" (chương 04).
- **"Logical Physical Clocks" (Kulkarni et al., 2014)** — paper HLC, nền của CockroachDB/MongoDB timestamp.
- **"There is No Now" (Justin Sheehy, ACM Queue)** — bài luận ngắn, sâu, về ảo tưởng "hiện tại" trong hệ phân tán; đọc để đổi tư duy.
- **Kleppmann, DDIA chương 8 "The Trouble with Distributed Systems"** — tổng hợp clock, GC pause, process pause hay nhất trong một chương sách.
