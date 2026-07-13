+++
title = "13.5. Infrastructure Failures"
date = "2026-07-13T17:00:00+07:00"
draft = false
tags = ["backend", "system-design"]
series = ["System Design — Tư Duy Thiết Kế Hệ Thống"]
+++

> Năm tình huống: GC Pause, Out of Memory, DNS Failure, Region Outage, Third-party API Down. Nhóm này nhắc một sự thật hay bị quên: dưới mọi kiến trúc đẹp là máy thật, mạng thật, và những công ty khác — tất cả đều hỏng theo lịch của riêng chúng.

---

## Case 17 — GC Pause

### Triệu chứng

Latency p99/p999 có **răng cưa chu kỳ** trong khi p50 đẹp; service "đứng hình" 0.5–30 giây rồi sống lại như chưa có gì; hệ quả dây chuyền đặc trưng: bị văng khỏi consumer group (rebalance), mất leadership (không kịp gia hạn lease), health check fail → bị restart oan — node *sống* nhưng bị cả cụm đối xử như *chết* trong đúng khoảng pause ([chương 4.4](/series/system-design/04-distributed-systems/04-clock-partition-split-brain/) — "chậm-như-đứt").

### Root cause & tại sao xảy ra

Runtime có GC (JVM, Go, .NET, Node) phải dừng ứng dụng ("stop-the-world") ít nhất một phần để thu hồi bộ nhớ. Pause dài khi: heap lớn + thuật toán GC thế hệ cũ (full GC trên heap 30GB có thể dừng hàng chục giây); allocation rate cao (tạo object ồ ạt — JSON parse khối lớn, cache in-process không giới hạn); heap gần đầy làm GC chạy liên tục ("GC death spiral": mỗi lần thu được ít, chạy lại ngay, CPU dành cho GC 90% — hệ thống sống dở chết dở, thường là tiền triệu chứng của [OOM](#case-18--out-of-memory)).

### Kiến trúc bị ảnh hưởng

Mọi service trên runtime có GC; **sát thương nhất với hệ nhạy heartbeat**: Kafka broker/consumer, ZooKeeper/etcd client, node giữ lock/lease, DB trên JVM (Cassandra, Elasticsearch — GC tuning là một phần nghề vận hành ES).

### Metric / Dashboard / Alert

- Metric: GC pause duration (max từng lần, không phải trung bình!), GC frequency, % CPU cho GC, heap used sau mỗi full GC (tăng dần qua các lần = leak), allocation rate.
- Dashboard: p999 latency chồng lên GC events — răng cưa khớp nhau là án tại hồ sơ.
- Alert: pause đơn lẻ > ngưỡng (tùy hệ: 1s với API, 200ms với hệ heartbeat); heap-after-full-GC vượt 80%.

### Điều tra

1. Bật GC log (chi phí thấp, nên bật sẵn từ đầu) → khớp thời điểm pause với thời điểm triệu chứng.
2. Pause do full GC hiếm + heap khổng lồ? do allocation spike (khớp với loại request nào)? hay heap-after-GC tăng dần (leak — sang case OOM)?

### Khắc phục & phòng tránh

- **Cầm máu:** restart node bệnh (giải pháp tạm hợp lệ); giảm tải node đó.
- **Chữa gốc:** GC hiện đại cho heap lớn (JVM: G1/ZGC — ZGC pause ~ms với heap hàng trăm GB); heap *vừa đủ* (heap to hơn ≠ tốt hơn — full GC lâu hơn); giảm allocation (streaming thay vì load cả khối, object pool cho hot path, cache in-process có giới hạn kích thước); **nhiều instance nhỏ thay vì một instance heap khổng lồ** — cũng giảm blast radius.
- **Phòng:** timeout/lease/heartbeat của hệ phân tán phải đặt **lớn hơn GC pause tối đa thực đo** (đo bằng GC log qua nhiều tuần) — đây là điểm giao giữa tuning JVM và thiết kế distributed; load test phải chạy đủ lâu để gặp full GC (test 5 phút không bao giờ thấy).

---

## Case 18 — Out of Memory (OOM)

### Triệu chứng

Process biến mất đột ngột — log ứng dụng *im lặng* (chết không kịp trăng trối), chỉ có dấu vết ở tầng dưới: `dmesg` ghi `oom-killer`, K8s ghi `OOMKilled`, exit code 137. Trước đó thường có: memory tăng đều theo thời gian (leak) hoặc spike theo một loại request (khối dữ liệu lớn). Dạng lặp: pod restart → nhận lại traffic → OOM lại → crash loop.

### Root cause & tại sao xảy ra

Ba họ nguyên nhân, cách điều tra khác nhau:

1. **Leak:** giữ tham chiếu không nhả — cache in-process không bound, listener không unregister, connection không đóng, closure giữ context. Memory tăng *tuyến tính theo thời gian*, không theo tải.
2. **Spike hợp pháp:** một request load 2GB (export không phân trang, file upload parse trong RAM, query `SELECT *` bảng to). Memory tăng *theo sự kiện cụ thể*.
3. **Cấu hình sai lệch giữa các tầng giới hạn:** heap limit của runtime > memory limit của container (JVM 8GB heap trong container 6GB — chết chắc, chỉ chờ dịp); quên rằng process còn dùng memory *ngoài heap* (off-heap, page cache, thread stack).

Điểm hệ thống: OOM killer của Linux giết process **điểm cao nhất** — không phải thủ phạm; trên node chung đụng, nạn nhân có thể là hàng xóm vô tội của kẻ phàm ăn.

### Kiến trúc bị ảnh hưởng

Mọi thứ. Nguy hiểm nhân lên khi: node chạy chung nhiều process (blast radius), stateful service (crash = failover + có thể mất dữ liệu chưa flush), crash loop dồn tải sang instance còn lại → [cascading](/series/system-design/13-production-failure-cases/04-distributed-failures/).

### Metric / Dashboard / Alert

- Metric: RSS/working set per process/container so với limit; heap used; *độ dốc* tăng memory (leak lộ ở đạo hàm); OOMKilled count; restart count.
- Alert: memory > 85% limit; **tăng đơn điệu nhiều giờ** (bắt leak trước khi nó thành OOM lúc 3h sáng); OOMKilled > 0.

### Điều tra

1. Xác nhận OOM (dmesg/K8s events/exit 137) — phân biệt với crash khác.
2. Nhìn đường memory trước khi chết: dốc đều (leak) / vách đứng (spike — khớp request nào? access log cùng thời điểm) / lình xình sát limit (limit sai hoặc thiếu thật).
3. Leak → heap dump/profiler (chụp hai thời điểm, so sánh loại object tăng); spike → tìm endpoint/payload thủ phạm.

### Khắc phục & phòng tránh

- **Cầm máu:** tăng limit tạm (mua thời gian — thừa nhận là nợ); restart theo lịch (xấu hổ nhưng thực dụng khi chưa tìm ra leak); chặn endpoint/payload thủ phạm.
- **Chữa gốc:** fix leak (từ heap dump); **streaming + phân trang** cho mọi thao tác dữ liệu lớn (quy tắc: memory của một request phải bị chặn trên, không phụ thuộc kích thước dữ liệu); mọi cache in-process phải **bounded** (LRU + max size); căn thẳng hàng ba tầng limit: runtime heap < container limit < node capacity, có chừa off-heap.
- **Phòng:** load test với payload *lớn bất thường* (không chỉ nhiều request); soak test nhiều giờ để lộ leak; requests/limits đặt từ số đo thực + headroom, không đặt theo cảm giác.

---

## Case 19 — DNS Failure

### Triệu chứng

Sự cố "không thể giải thích": lỗi `could not resolve host` / timeout connect rải rác hoặc đồng loạt; *một số* node lỗi trong khi node khác lành (khác cache DNS); sự cố xuất hiện **đúng lúc mọi thứ khác cũng đang hỏng** (DNS hay là nạn nhân thứ cấp — và làm sự cố gốc khó chẩn đoán gấp đôi); hoặc kinh điển nhất — service *đã* chuyển IP mà một nửa hệ thống vẫn gọi IP cũ hàng giờ.

### Root cause & tại sao xảy ra

DNS là hệ phân tán lâu đời nhất bạn đang dùng và là dependency của **mọi** dependency. Các họ lỗi: resolver quá tải/chết (mỗi request tạo N lần resolve nếu không cache — chính bạn DDoS resolver của mình); TTL bị bỏ qua (JVM mặc định có thể cache DNS *vĩnh viễn* nếu cấu hình sai `networkaddress.cache.ttl`; một số app/OS cache quá tay hoặc không hề cache); registrar/provider DNS bị sự cố (đã từng xảy ra với các provider lớn nhất — nửa internet tê liệt vài giờ); NXDOMAIN cache độc (cache "không tồn tại" trong khi record vừa tạo); trong K8s: CoreDNS quá tải là sự cố kinh điển của cluster lớn (mỗi lookup từ pod có thể nở thành 5 query vì `ndots:5`).

### Kiến trúc bị ảnh hưởng

Tất cả — DNS đứng trước mọi kết nối. Đặc biệt: microservices (service discovery qua DNS = tần suất lookup khổng lồ), failover dựa trên đổi DNS record (RTO thực = TTL + sự tùy hứng của mọi cache trên đường — thường lâu hơn TTL nhiều), kết nối ra bên thứ ba.

### Metric / Dashboard / Alert

- Metric: DNS lookup latency + error rate (export từ app/sidecar — đa số hệ thống *không đo DNS* và trả giá bằng những giờ điều tra mù); QPS vào resolver nội bộ (CoreDNS); NXDOMAIN rate.
- Alert: resolve error > 0.1%; CoreDNS CPU/QPS vượt ngưỡng; synthetic check resolve các domain sống còn từ nhiều vantage point.

### Điều tra

1. Lỗi resolve hay lỗi connect? (`dig` từ chính node bệnh — đừng từ laptop của bạn).
2. Node bệnh và node lành khác nhau gì? (resolver config, cache state, version image).
3. Record vừa đổi? TTL bao nhiêu? Ai còn cache giá trị cũ (theo tầng: app runtime → OS → node resolver → recursive resolver)?

### Khắc phục & phòng tránh

- **Cầm máu:** flush cache theo tầng; trỏ tạm resolver khác; trường hợp cháy nhà — ghi tạm `/etc/hosts`/ConfigMap (nhớ dọn sau, hosts-entry mồ côi là bug hẹn giờ).
- **Chữa gốc & phòng:**
  - Cache DNS **có chủ đích** ở app (TTL hữu hạn, tôn trọng TTL record; kiểm tra cấu hình JVM/language runtime — đừng nhận mặc định mù).
  - Resolver nội bộ HA (CoreDNS đủ replica + autoscale; NodeLocal DNSCache cho K8s lớn).
  - TTL chiến lược: record dùng cho failover → TTL ngắn (30–60s) *từ trước khi cần*; hạ TTL ngay trước thay đổi có kế hoạch.
  - Dùng ≥ 2 DNS provider cho zone công khai sống còn (bài học từ các sự cố provider lớn).
  - Connection pool + keep-alive dài giảm tần suất resolve; nhưng kèm cơ chế re-resolve định kỳ để không ôm IP chết.

---

## Case 20 — Region Outage

### Triệu chứng

Không phải "một service lỗi" — mà **mọi thứ trong một vùng địa lý cùng lỗi**: hạ tầng compute, storage, LB, *và cả console/API của cloud provider* (khiến chính việc xử lý sự cố khó khăn); status page của provider... cũng có thể chết theo. Traffic từ user vùng đó rơi tự do. Nếu chưa có multi-region: sự cố = toàn bộ downtime, việc duy nhất làm được là chờ và thông báo khách hàng.

### Root cause & tại sao xảy ra

Nguyên nhân phía provider (lỗi điện/mát, lỗi mạng backbone, lỗi cascade trong control plane của chính họ — đã xảy ra với mọi cloud lớn, tần suất thô: đáng kể vài năm một lần mỗi region) — nhưng root cause *của bạn* là **quyết định đặt 100% hệ thống vào một failure domain**. Đây là quyết định hợp lý ở quy mô nhỏ (chi phí multi-region không đáng — [giai đoạn 1–8](/series/system-design/12-evolution/00-tong-quan/)) và trở thành rủi ro tồn vong ở quy mô lớn ([giai đoạn 9–10](/series/system-design/12-evolution/09-multi-region/)). Điểm chuyển hợp lý là một phép nhân: xác suất outage × chi phí mỗi giờ downtime × số giờ dự kiến, so với chi phí multi-region mỗi năm.

### Kiến trúc bị ảnh hưởng

Single-region: toàn bộ. Multi-region: phần *không thật sự* độc lập — và sự cố region là kỳ thi lộ ra các dependency ngầm: "region B tự chủ" nhưng auth vẫn gọi về region A, CI/CD nằm ở A (không deploy được bản vá trong lúc cháy!), secret store ở A, bảng routing do A quản.

### Metric / Dashboard / Alert

- Metric: synthetic probe **từ ngoài, đa vùng** đo từng region (nội bộ region chết thì monitoring trong đó chết theo — observability phải ở ngoài blast radius); health tổng hợp theo region; % traffic từng region.
- Alert: region fail probe từ ≥ 2 vantage point trong X phút → quy trình DR ([giai đoạn 10](/series/system-design/12-evolution/10-disaster-recovery/)).

### Điều tra

Ngắn gọn có chủ đích — lúc này *không phải lúc tìm root cause của provider*:

1. Xác nhận phạm vi (provider status + probe của mình + cộng đồng) — region thật sự chết hay chỉ mình bị (khác nhau hoàn toàn!).
2. Kích hoạt quy trình DR đã định: đây là lúc *thực thi* văn bản đã viết lúc bình tĩnh, không phải lúc sáng tác.

### Khắc phục & phòng tránh

- **Khi xảy ra (đã có DR):** tuyên bố thảm họa theo tiêu chí định trước (đừng chờ "thêm 15 phút nữa xem sao" ba lần liên tiếp — đó là cách RTO 15 phút thành 2 giờ); failover theo runbook; chấp nhận RPO đã ký (dữ liệu async chưa kịp sang — quyết định *đã được* duyệt trước, cứ thế làm); truyền thông khách hàng sớm và thật.
- **Khi xảy ra (chưa có DR):** thông báo khách hàng, chờ, và biến nỗi đau thành ngân sách cho giai đoạn 9–10.
- **Phòng:** multi-AZ là mặc định rẻ (chống AZ chết — phổ biến hơn region chết nhiều lần); multi-region theo phân tích chi phí/rủi ro; **DR drill định kỳ** — kiểm kê dependency ngầm bằng cách *thật sự* chạy một mình region B; giữ "công cụ xử lý sự cố" (monitoring, runbook, CI/CD, liên lạc) **ngoài** blast radius của region chính.

---

## Case 21 — Third-party API Down

### Triệu chứng

Một luồng nghiệp vụ cụ thể gãy (thanh toán fail, không gửi được OTP, không tính được phí ship) trong khi hệ của bạn xanh rờn; hoặc tệ hơn — bên thứ ba không *chết* mà *chậm* (timeout 30s thay vì trả lỗi ngay) → giam thread/connection của bạn → [pool exhaustion](/series/system-design/13-production-failure-cases/02-database-failures/) → sự cố *của họ* thành sự cố *toàn hệ thống của bạn*. Biến thể thâm: API sống nhưng trả **dữ liệu sai/format mới** — không alert nào kêu, chỉ có nghiệp vụ âm thầm sai.

### Root cause & tại sao xảy ra

Bên thứ ba là hệ thống của người khác: deploy của họ, sự cố của họ, rate limit của họ, thay đổi contract của họ — theo lịch của họ. Root cause phía bạn: **tích hợp kiểu tin cậy tuyệt đối** — gọi sync trong đường nóng, không timeout riêng, không CB, không fallback, không giám sát *riêng cho họ*. Availability của luồng = availability của bạn × của họ ([chương 1.2](/series/system-design/01-foundations/02-sla-slo-sli/)) — muốn hứa 99.9% trên nền đối tác 99.5% mà không có degradation path là toán sai từ đề bài.

### Kiến trúc bị ảnh hưởng

Mọi hệ thống thật (thanh toán, SMS/OTP, email, bản đồ, vận chuyển, KYC, AI API...). Nặng nhất: luồng doanh thu phụ thuộc một đối tác duy nhất không có phương án hai.

### Metric / Dashboard / Alert

- Metric: **per-dependency**: error rate, latency p99, timeout rate, trạng thái CB, rate-limit-remaining (nếu họ trả header); và **đối soát nghiệp vụ** (số OTP gửi vs nhận xác nhận) để bắt ca "sống mà sai".
- Dashboard: một trang "sức khỏe các đối tác" — người trực nhìn 5 giây biết lỗi của ta hay của họ.
- Alert: error rate của dependency X > ngưỡng; CB mở; *đối soát lệch*.

### Điều tra

1. Phạm vi: chỉ luồng dùng đối tác đó? → khoanh ngay.
2. Status page/community của họ (nhưng đừng tin tuyệt đối — status page nổi tiếng lạc quan); test call độc lập từ môi trường sạch.
3. Lỗi từ bao giờ, có trùng thay đổi phía *mình* không (đổi key, hết hạn credential, vượt quota — "họ chết" hóa ra "mình hết tiền trong tài khoản credit" không hiếm).

### Khắc phục & phòng tránh

- **Cầm máu:** bật degraded mode đã thiết kế sẵn (xem dưới); nếu chưa có — CB thủ công (feature flag tắt tích hợp) để cứu phần còn lại của hệ thống trước.
- **Phòng — thiết kế "tin nhưng rào":**
  - **Timeout riêng, ngắn** cho mọi call ra ngoài (đừng nhận default 30–60s của HTTP client) + CB + retry có budget.
  - **Đẩy khỏi đường nóng khi nghiệp vụ cho phép:** gửi email/OTP/webhook qua queue — đối tác chết thì backlog chờ, user không thấy gì ([giai đoạn 3–4](/series/system-design/12-evolution/03-background-worker/)).
  - **Fallback phân bậc:** provider dự phòng (SMS qua 2 nhà cung cấp, active-passive); giá trị cache/mặc định (phí ship ước tính khi API ship chết — kèm cờ "ước tính"); hoặc **degradation nghiệp vụ có chủ đích** (cho đặt COD khi cổng thẻ chết — quyết định cùng product, trước sự cố).
  - **Contract test + validate response** (schema check) để bắt ca "trả về rác" — đừng tin dữ liệu ngoài chỉ vì HTTP 200.
  - Idempotency key theo chuẩn đối tác cho mọi thao tác tiền (retry an toàn); đối soát định kỳ như lưới cuối.

---

## Tổng kết nhóm infrastructure

| Case | Bài học một dòng |
|---|---|
| GC Pause | Node "sống mà như chết" — timeout/lease phải lớn hơn pause thực đo |
| OOM | Memory mỗi request phải bị chặn trên; ba tầng limit phải thẳng hàng |
| DNS | Dependency của mọi dependency — đo nó, cache nó có chủ đích, đừng tin mặc định |
| Region Outage | Failure domain là quyết định thiết kế; công cụ cứu hỏa phải ở ngoài đám cháy |
| 3rd-party Down | Availability của bạn = tích các thừa số; mọi thừa số ngoài tầm tay cần rào chắn + phương án hai |

---

*Hết Phần 13. Quay lại [tổng quan](/series/system-design/13-production-failure-cases/00-tong-quan/) hoặc [mục lục chính](/series/system-design/00-muc-luc/).*
