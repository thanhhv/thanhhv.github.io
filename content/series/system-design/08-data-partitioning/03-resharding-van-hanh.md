+++
title = "8.3. Resharding & vận hành hệ đã shard"
date = "2026-07-13T12:10:00+07:00"
draft = false
tags = ["backend", "system-design"]
series = ["System Design — Tư Duy Thiết Kế Hệ Thống"]
+++

## 1. Problem Statement

Mọi quyết định sharding đều sai dần theo thời gian: dữ liệu tăng 10× (4 shard thành chật), phân bố lệch dần (một shard 80% đầy trong khi anh em 30%), tenant VIP phình vượt shard của nó, hoặc — đau nhất — **shard key hóa ra chọn sai** ([8.1 §3.2](/series/system-design/08-data-partitioning/01-partitioning-sharding/)). Câu hỏi của chương này: làm sao thay đổi cách chia dữ liệu **trên hệ đang chạy, không downtime, không mất ghi** — bài toán được xếp vào loại khó nhất của vận hành dữ liệu, và là lý do người ta nói "shard key là cam kết hôn nhân".

## 2. First Principles — vì sao resharding khó, và khe cửa nào để lách

Resharding = di chuyển dữ liệu *trong khi* dữ liệu đang bị đọc/ghi. Ba ràng buộc đánh nhau: (1) dữ liệu phải **đủ và đúng** ở đích trước khi cắt chuyển; (2) ghi mới **không được rơi** vào khe hở giữa "đã copy" và "đã chuyển"; (3) hệ phải **phục vụ bình thường** suốt quá trình — nghĩa là không có khoảnh khắc "khóa hết để dọn nhà".

Khe cửa: **đừng chuyển tất cả cùng lúc — chuyển từng đơn vị nhỏ, mỗi đơn vị đi qua một quy trình bốn pha có đường lùi.** Đơn vị nhỏ = slot/chunk/bucket ([8.2 §4 — vì sao mẫu "bucket cố định + bảng tra" quý giá](/series/system-design/08-data-partitioning/02-consistent-hashing/)): resharding trở thành *chuỗi* thao tác nhỏ, mỗi cái vài phút, dừng được, lùi được — thay vì một cú nhảy sinh tử.

## 3. Quy trình chuẩn — bốn pha cho một đơn vị dữ liệu

```mermaid
sequenceDiagram
    participant APP as App/Router
    participant S1 as Shard nguồn
    participant S2 as Shard đích
    Note over S1,S2: Pha 1 — BACKFILL: copy dữ liệu nền<br/>(snapshot + đuổi theo thay đổi qua CDC)
    S1->>S2: copy snapshot + stream thay đổi
    Note over APP: Pha 2 — DUAL-WRITE: ghi cả hai nơi,<br/>đọc vẫn từ nguồn; so khớp liên tục
    APP->>S1: write (nguồn sự thật)
    APP->>S2: write (bản đuổi)
    Note over APP: Pha 3 — VERIFY: đối soát checksum/count<br/>theo dải key; lệch → quay lại pha 1 cho dải đó
    Note over APP: Pha 4 — CUTOVER: đảo đọc sang đích,<br/>giữ dual-write một thời gian (đường lùi!),<br/>rồi tắt ghi nguồn, dọn dữ liệu cũ
    APP->>S2: read + write (nguồn sự thật mới)
```

Bốn điểm sống còn, nơi các sự cố resharding thực tế xảy ra:

1. **Backfill phải throttle:** copy hết tốc độ = ăn hết I/O của shard nguồn đang phục vụ production — chính là tự gây [replica lag/quá tải](/series/system-design/13-production-failure-cases/02-database-failures/); tốc độ backfill là núm chỉnh theo tải thật, và tổng thời gian resharding tính bằng ngày/tuần là bình thường.
2. **Dual-write phải xử lý lệch:** ghi nguồn thành công, ghi đích fail thì sao? — ghi đích qua hàng đợi bền + retry ([outbox-style](/series/system-design/06-communication/08-outbox/)), hoặc chấp nhận lệch và để pha verify bắt; **không bao giờ** để fail đích làm fail request của user (đích đang là bản nháp).
3. **Verify là pha bị cắt xén nhiều nhất và hối hận nhiều nhất:** đối soát bằng checksum theo dải + đếm theo cửa sổ thời gian, chạy máy, liên tục — "trông có vẻ đủ" không phải là verify.
4. **Cutover phải có đường lùi tức thời:** đảo đọc bằng config/flag runtime (không phải deploy); giữ dual-write ngược (đích → nguồn) thêm vài ngày — phát hiện vấn đề sau cutover thì đảo cờ là về nhà cũ, dữ liệu vẫn đủ hai nơi.

Cùng bộ xương này áp cho mọi biến thể: chẻ shard (split), gộp shard, chuyển tenant VIP ra shard riêng, **đổi shard key** (nặng nhất — thực chất là xây một "hệ shard thứ hai" theo key mới rồi migrate toàn bộ qua bốn pha), và cả migrate sang engine khác ([5.7 §5](/series/system-design/05-data-layer/07-so-sanh-lua-chon/)). Các nền tảng trưởng thành (Vitess resharding workflow, MongoDB chunk migration, Redis slot migration) là bốn pha này được đóng gói — hiểu bộ xương để vận hành chúng có ý thức, và để biết chúng đang ở pha nào khi có chuyện.

## 4. Vận hành thường nhật của hệ đã shard

Ngoài resharding, hệ shard đòi một bộ kỷ luật vận hành riêng:

- **Cân bằng liên tục thay vì chiến dịch:** balancer tự động (Mongo, Vitess) hoặc quy trình định kỳ rà độ lệch dung lượng/tải giữa shard ([13.2 — max/median](/series/system-design/13-production-failure-cases/02-database-failures/)) và di chuyển dần từng bucket — để lệch tích tụ 2 năm rồi "làm một trận" là tự chọn chế độ khó.
- **Schema migration × N shard:** một `ALTER` giờ chạy N nơi — cần orchestration (chạy lần lượt, theo dõi, dừng khi shard nào lỗi) và schema các shard **được phép lệch tạm thời** trong lúc rollout → app phải chịu được hai version schema cùng lúc (kỷ luật quen thuộc từ [12.6 — backward compatibility](/series/system-design/12-evolution/06-microservices/), giờ áp cho DB).
- **Query xuyên shard cho phân tích:** đừng scatter-gather trên OLTP shard — CDC toàn bộ shard về một nơi tập trung ([ClickHouse](/series/system-design/05-data-layer/05-clickhouse/)/warehouse) cho analytics; shard OLTP chỉ phục vụ nghiệp vụ trong-shard ([12.8 — CQRS](/series/system-design/12-evolution/08-cqrs/) phiên bản sharded).
- **Capacity planning theo shard nóng nhất, không theo tổng** ([1.4](/series/system-design/01-foundations/04-scale-estimation-capacity-planning/)): trần thật của hệ = trần của shard đầy/nóng nhất; dashboard tăng trưởng per-shard + dự báo "shard nào chạm 70% khi nào" → resharding *có kế hoạch* thay vì cấp cứu — khác biệt giữa hai chế độ đó là toàn bộ chương này chạy lúc 10 giờ sáng thứ Ba hay 2 giờ sáng Chủ nhật.

## 5. Trade-off

| Quyết định | Được | Giá |
|---|---|---|
| Bucket nhiều + nhỏ từ đầu (nghìn bucket) | Resharding = di chuyển bucket, mịn, dừng/lùi được | Metadata to hơn; per-bucket overhead |
| Dual-write dài (thận trọng) | Đường lùi rộng, verify kỹ | Chi phí ghi ×2 kéo dài; code hai đường lâu |
| Balancer tự động | Không tích nợ lệch | Di chuyển nền ăn I/O — phải có cửa sổ và throttle; balancer "tự tiện" giờ cao điểm là nguồn sự cố kinh điển (Mongo cho lịch balancer có lý do) |
| Tự chế resharding ở app | Kiểm soát trọn | Đắt và rủi ro nhất trong mọi lựa chọn của phần này — chỉ khi không còn đường khác |

## 6. Production Considerations

- **Diễn tập trên staging với dữ liệu cỡ thật** trước lần resharding production đầu tiên — đo thời gian từng pha, độ ăn I/O, điểm gãy — đúng nghi thức [12.10 — drill](/series/system-design/12-evolution/10-disaster-recovery/).
- Trong lúc resharding, **quan sát như đang có sự cố:** dashboard riêng cho tiến độ backfill, lệch dual-write, kết quả verify theo dải, error rate của cả hai shard; người trực biết resharding đang chạy (điều tra "vì sao I/O cao" lúc 3h sáng mà không ai ghi lịch là chuyện có thật).
- Mọi bước đổi routing qua **config runtime có audit log** — ai đảo cờ gì lúc nào; và freeze các thay đổi khác (schema, deploy lớn) trong cửa sổ cutover.
- Dọn dữ liệu nguồn **sau cùng, sau nhiều ngày, có backup** — disk rẻ hơn nhiều so với phát hiện thiếu dữ liệu sau khi đã xóa nguồn.

## 7. Anti-patterns

- **Big-bang resharding** (maintenance window một đêm chuyển hết) — mọi ràng buộc §2 dồn vào một khe hẹp; quá giờ là tiến thoái lưỡng nan kinh điển.
- **Bỏ pha verify** hoặc verify bằng mắt — lệch âm thầm lộ ra sau cutover hàng tuần, khi nguồn đã xóa.
- **Cutover bằng deploy** thay vì cờ runtime — đường lùi là một lần deploy nữa, dài đúng bằng khoảng thời gian tồi tệ nhất để chờ.
- **Resharding đồng thời với thay đổi lớn khác** (đổi schema + đổi key + nâng version engine một thể "cho tiện") — ba biến số, một sự cố, không cô lập được nguyên nhân.
- **Không có kế hoạch resharding ngay từ thiết kế sharding** — chọn kiến trúc không có đường resharding (mod N cứng trong code, không bucket, không CDC) là chọn trước cách mình sẽ đau.

## 8. Khi nào KHÔNG nên resharding

Khi bài toán thật là **hot key đơn lẻ** ([13.2 — salting/VIP lane](/series/system-design/13-production-failure-cases/02-database-failures/)) — di chuyển dữ liệu không giúp key nóng nguội đi; khi lệch dung lượng còn chữa được bằng **archive dữ liệu nguội** (rẻ hơn mọi phương án); khi trần sắp chạm là **trần đọc** — replica/cache ([Phần 7](/series/system-design/07-caching/00-tong-quan/)) trước; và khi hệ *chưa* shard — hãy đọc lại checklist [8.1 §7](/series/system-design/08-data-partitioning/01-partitioning-sharding/) một lần nữa trước khi bước vào thế giới mà chương này mô tả. Toàn bộ Phần 8 có thể tóm trong một câu khuyên: **shard muộn nhất có thể, shard theo chủ sở hữu tự nhiên, chia bucket nhỏ từ đầu, và giữ đường lùi ở mọi bước.**

---

*Hết Phần 8. Quay lại [mục lục chính](/series/system-design/00-muc-luc/).*
