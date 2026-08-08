+++
title = "Level 3C — Incident Leadership và Engineering Metrics: DORA, SPACE, SLO và Postmortem"
date = "2026-08-01T14:00:00+07:00"
draft = false
tags = ["engineering-leadership", "management", "career"]
series = ["Engineering Leadership & Technical Management"]
+++

# Level 3C — Incident Leadership và Engineering Metrics

2 giờ 47 phút sáng, một ví điện tử ở Hà Nội. Kênh `#alert-prod` đã bắn 214 tin nhắn trong 40 phút.
Trong channel `#war-room` có 31 người: sáu engineer đang thực sự làm gì đó, một Product Manager hỏi
"tình hình sao rồi" lần thứ tư, một founder vừa vào và gõ "sao chưa ai báo tôi", và hai mươi người còn
lại đang đọc. Tuấn — Senior BE giỏi nhất về hệ thống thanh toán — vừa restart một pod, vừa trả lời
founder, vừa mở một PR hotfix. Linh, ở một nhánh khác, đang scale up database mà Tuấn không biết. Ba
mươi phút sau, hệ thống hồi phục, nhưng không ai trong phòng nói được chính xác **hành động nào đã
khiến nó hồi phục**. Sáng hôm sau, câu hỏi đầu tiên từ tầng trên không phải "hệ thống hỏng ở đâu" mà
"ai deploy tối qua".

Tuần sau, cùng loại sự cố xảy ra lần nữa.

Incident là lúc mọi khiếm khuyết của hệ thống kỹ thuật **và** hệ thống quản trị hiện ra cùng lúc,
dưới áp lực thời gian. Thiếu observability hiện ra thành 40 phút mò mẫm. Thiếu cấu trúc chỉ huy hiện
ra thành hai người sửa song song. Thiếu Psychological Safety hiện ra thành một junior im lặng không
dám nói mình vừa đổi config. Thiếu cơ chế học hiện ra thành lần thứ hai, thứ ba. Không có tình huống
nào khác trong đời sống engineering vừa nén nhiều thông tin về chất lượng quản trị vào một khoảng thời
gian ngắn như vậy — và cũng không có tình huống nào mà chi phí của quản trị tồi được tính bằng tiền
nhanh như vậy.

Còn metrics là mặt còn lại của cùng một bài toán. Sau mỗi incident, mọi tổ chức đều "rút ra bài học".
Câu hỏi khó hơn là: sáu tháng sau, bạn có bằng chứng nào cho thấy tổ chức đã tốt lên? Không có hệ đo,
bạn không phân biệt được **đang cải thiện** với **đang bận**. Một team deploy 40 lần một tuần và một
team deploy 2 lần một tuần đều có thể mô tả mình là "làm việc rất tích cực". Metrics là cơ chế duy
nhất biến câu hỏi đó thành trả lời được — và cũng là cơ chế dễ bị biến thành vũ khí nhanh nhất khi
dùng sai tầng.

Trong chuỗi Business Goal → People → Process → Technology → Execution → Feedback → Improvement →
Scaling Team → Organization, chương này chiếm trọn hai mắt **Feedback** và **Improvement**. Chương 05
nói về cách làm cho hàng nghìn quyết định kỹ thuật đi cùng hướng. Chương này nói về việc hệ thống đó
sẽ hỏng — chắc chắn hỏng — và tổ chức của bạn có cơ chế nào để mỗi lần hỏng làm nó mạnh lên thay vì
yếu đi và mất người.

Mục lục nội bộ:

1. [Incident Response — cấu trúc chỉ huy](#1-incident-response--cấu-trúc-chỉ-huy)
2. [Incident Leadership — hành vi của lead trong và sau incident](#2-incident-leadership--hành-vi-của-lead-trong-và-sau-incident)
3. [Postmortem không quy tội (Blameless Postmortem)](#3-postmortem-không-quy-tội-blameless-postmortem)
4. [On-call bền vững](#4-on-call-bền-vững)
5. [SLO, SLI và Error Budget](#5-slo-sli-và-error-budget)
6. [Engineering Metrics: DORA và SPACE](#6-engineering-metrics-dora-và-space)
7. [Learning Organization — biến sự cố thành năng lực](#7-learning-organization--biến-sự-cố-thành-năng-lực)
8. [Tự kiểm tra](#tự-kiểm-tra)
9. [Liên kết chương khác](#liên-kết-chương-khác)

> Mọi con số trong chương này là **số minh hoạ** để bạn thấy cách tính, cách đặt ngưỡng và cách đọc
> tổ hợp, không phải dữ liệu nội bộ của bất kỳ công ty nào. Khi áp dụng, thay bằng số đo được từ hệ
> thống của bạn.

---

## 1. Incident Response — cấu trúc chỉ huy

### Problem Statement

Bốn hiện tượng dưới đây đều đếm được, và mỗi cái là một dấu hiệu tổ chức không có cấu trúc chỉ huy
incident — kể cả khi trên Confluence có một trang tên là "Quy trình xử lý sự cố".

**Hiện tượng một: MTTR bị chi phối bởi thời gian phối hợp, không phải thời gian sửa.** Lấy năm
incident gần nhất, dựng lại timeline theo phút, rồi phân loại mỗi khoảng thời gian vào một trong bốn
nhóm: phát hiện, huy động người, xác định hướng xử lý, thực thi. Ở nhiều tổ chức 30–60 engineer, con
số minh hoạ rơi vào khoảng này: phát hiện 8 phút, huy động 14 phút, xác định hướng 31 phút, thực thi 6
phút. Nghĩa là **thời gian sửa thật chiếm 10% tổng downtime**. Mọi khoản đầu tư vào "làm engineer sửa
nhanh hơn" đang tối ưu cho 10% đó.

**Hiện tượng hai: sau khi hồi phục, không ai nói chính xác được hành động nào đã có tác dụng.** Đây
không phải chuyện nhỏ. Nếu bạn không biết cái gì đã sửa nó, bạn không có cách nào ngăn nó lặp lại, và
bạn cũng không có cách nào rút ngắn lần sau. Dấu hiệu: postmortem viết "hệ thống tự hồi phục sau khi
restart service và scale database", với chữ "và" thay cho một phân tích.

**Hiện tượng ba: số người trong war room tăng nhưng thời gian xử lý không giảm.** Đếm số người trong
channel và số người thực sự có hành động ghi lại được. Tỉ lệ 31/6 ở đầu chương là điển hình. 25 người
còn lại không vô hại — họ tạo ra câu hỏi, và mỗi câu hỏi tiêu một phần băng thông của đúng những người
đang cần tập trung.

**Hiện tượng bốn: stakeholder biết tin từ khách hàng trước khi biết từ engineering.** Sáng hôm sau,
Head of Customer Support nói "tổng đài nhận 180 cuộc gọi từ 2h30, sao không ai báo bên tôi". Đây là
lỗi cấu trúc, không phải lỗi ai đó quên: khi không có ai được phân vai Communications Lead, việc giao
tiếp mặc định rơi vào người đang tay bận sửa nhất, và nó sẽ bị bỏ.

Hậu quả dài hạn có hình dạng khá cố định. Không có cấu trúc → mỗi incident phụ thuộc vào việc ai
tình cờ có mặt → người giỏi nhất bị gọi mọi lần → người đó thành single point of failure của cả quy
trình vận hành → burnout hoặc nghỉ → tổ chức mất luôn năng lực xử lý sự cố. Với fintech, thêm một
nhánh nữa: không có Scribe → không có timeline chính xác → khi cơ quan quản lý hoặc auditor yêu cầu
báo cáo sự cố, tổ chức phải dựng lại từ ký ức, và bản báo cáo đó vừa chậm vừa không đứng vững.

### First Principles

**Cơ chế một: chi phí của mỗi phút trong incident cao và không đối xứng, nên cấu trúc quyết định phải
đổi chế độ.** Ngày thường, một quyết định kỹ thuật sai được sửa trong lần review tiếp theo; chi phí
của một quyết định chậm gần bằng không. Trong incident, mỗi phút là tiền, là giao dịch lỗi, là hợp
đồng SLA, là niềm tin. Ở fintech, một phút downtime giờ cao điểm có thể là hàng nghìn giao dịch không
hoàn tất cộng với nghĩa vụ báo cáo. Khi chi phí của độ trễ tăng vài bậc, cơ chế ra quyết định tối ưu
cũng đổi: consensus là cơ chế tốt cho quyết định có thể sai và sửa được về sau, nhưng nó có độ trễ
tuyến tính theo số người tham gia. Command-and-control có độ trễ gần như hằng số. Đây là lý do kỹ
thuật, không phải lý do thẩm quyền.

**Cơ chế hai: băng thông giao tiếp của nhóm bị bão hoà, và số kênh phối hợp tăng theo n².** Với n
người cùng trao đổi tự do, số cặp giao tiếp là n(n−1)/2. Với 6 người là 15 cặp; với 31 người là 465
cặp. Trong một text channel duy nhất, tất cả các cặp đó chia sẻ **một** kênh tuần tự. Kết quả là hiện
tượng quan sát được ở mọi war room không có cấu trúc: câu hỏi quan trọng bị đẩy lên khỏi màn hình
trước khi người cần đọc nó kịp đọc. Cấu trúc vai (role) là cách giảm 465 cặp về khoảng 5 kênh: mọi
người nói với IC, IC nói với Ops Lead, Communications Lead nói ra ngoài.

**Cơ chế ba: vì sao "ai cũng debug" chậm hơn, dù tổng năng lực cao hơn.** Có bốn cơ chế cộng dồn:

1. **Trùng lặp không quan sát được.** Ba người cùng đọc cùng dashboard, cùng loại trừ cùng một giả
   thuyết. Năng lực cộng lại nhưng vùng tìm kiếm không mở rộng.
2. **Mutation xung đột trên cùng một hệ thống production.** Tuấn restart pod trong khi Linh scale
   database. Nếu hệ thống hồi phục, không ai biết vì cái nào. Nếu hệ thống tệ hơn, không ai biết vì
   cái nào. Cả hai trường hợp đều làm mất tín hiệu.
3. **Nhiễu tín hiệu chẩn đoán do chính hành động của người sửa.** Mỗi restart tạo ra một đợt error
   mới trong log. Sau ba người thao tác, log của hệ thống phản ánh hành vi của những người đang sửa
   nhiều hơn phản ánh nguyên nhân gốc. Đây là dạng đo lường phá đối tượng đo.
4. **Diffusion of responsibility.** Đây là kết quả tâm lý học xã hội đã được công bố rộng rãi: khi
   nhiều người cùng chịu trách nhiệm về một việc, xác suất mỗi người hành động giảm. Trong war room,
   nó biểu hiện thành 25 người chờ ai đó quyết định.

**Cơ chế bốn: mitigate và diagnose là hai mục tiêu khác nhau và cạnh tranh nhau về thời gian.** Hiểu
nguyên nhân gốc là công việc có phương sai thời gian rất cao — có thể 5 phút, có thể 5 giờ. Khôi phục
dịch vụ thường có phương sai thấp hơn nhiều vì tập hành động hữu hạn: rollback, tắt feature flag,
chuyển traffic, tăng capacity, chặn nguồn tải. Khi chi phí mỗi phút cao, chọn hành động có phương sai
thấp trước là quyết định đúng theo lý thuyết quyết định, không phải sự thiếu tò mò.

### Mental Model

**Mô hình một: single writer.** Coi production trong lúc incident như một tài nguyên chỉ cho phép
**một** người ghi tại một thời điểm. Đây đúng là mô hình mutex trong lập trình đồng thời, và lý do
cũng y hệt: không phải vì người khác kém, mà vì ghi đồng thời làm trạng thái không suy luận được.
Trong incident, "quyền ghi" nằm ở Ops Lead, và IC là người cấp quyền. Mọi người khác có thể đọc, phân
tích, đề xuất — nhưng không chạm vào production. Câu nói vận hành mô hình này rất ngắn: *"Không ai
chạy lệnh trên production trừ khi đã nói trong channel và tôi đã xác nhận."*

**Mô hình hai: OODA loop (Observe → Orient → Decide → Act).** Vòng lặp này do John Boyd đề xuất trong
bối cảnh quân sự và được dùng rộng rãi cho các tình huống có địch thủ hoặc bất định cao. Giá trị của
nó ở đây: nó cho bạn một chỗ để chẩn đoán **vòng lặp đang nghẽn ở đâu**.

| Pha nghẽn | Biểu hiện trong war room | Can thiệp của IC |
|---|---|---|
| Observe | "Không biết đang hỏng cái gì", dashboard không mở được | Phân người đi lấy đúng ba số liệu, không phải "mọi người xem log" |
| Orient | Có dữ liệu nhưng không ai đưa được giả thuyết | Yêu cầu phát biểu 2–3 giả thuyết cạnh tranh, gán mỗi giả thuyết cho một người |
| Decide | Có giả thuyết nhưng tranh luận không kết thúc | IC chọn, nói rõ "chúng ta thử hướng A trong 10 phút, không được thì rollback" |
| Act | Đã quyết nhưng không ai chạy vì không rõ ai làm | Gán tên cụ thể + xác nhận đã thực hiện trong channel |

Điểm quan trọng: một IC tồi thường can thiệp vào pha Observe (đi đọc log cùng mọi người) trong khi
nghẽn thật nằm ở pha Decide. Đó chính là lý do IC không được tự sửa — công việc thật của IC nằm ở
Orient và Decide, và cả hai đòi hỏi giữ được mô hình toàn cục.

**Mô hình ba: incident như một hệ thống có hai đồng hồ.** Đồng hồ thứ nhất đếm downtime kỹ thuật.
Đồng hồ thứ hai đếm **thời gian stakeholder ở trong trạng thái không biết gì**. Hai đồng hồ này độc
lập, và đồng hồ thứ hai thường gây thiệt hại về niềm tin lớn hơn. Một incident 90 phút có cập nhật
mỗi 20 phút được nhớ khác hoàn toàn một incident 45 phút mà im lặng suốt. Communications Lead tồn tại
để dừng đồng hồ thứ hai.

### Practical Framework

**Bốn vai bắt buộc.** Với incident nhỏ, một người có thể giữ hai vai; với SEV1 thì không.

| Vai | Làm gì | Tuyệt đối không làm | Ai phù hợp |
|---|---|---|---|
| **Incident Commander (IC)** | Giữ mô hình toàn cục; quyết hướng xử lý; gán việc theo tên; quyết leo thang severity; quyết dừng incident | **Không tự chạy lệnh, không tự sửa code, không tự đọc log sâu**; không đi trả lời stakeholder | Người có khả năng điều phối, không nhất thiết là người giỏi kỹ thuật nhất |
| **Ops Lead** | Người duy nhất được thay đổi production trong phạm vi IC cho phép; đọc số liệu; thực thi mitigation | Không tự leo thang, không tự trả lời stakeholder, không mở rộng phạm vi thay đổi mà không nói | Người quen hệ thống đang hỏng nhất |
| **Communications Lead** | Cập nhật stakeholder theo nhịp cố định; trả lời câu hỏi từ ngoài để IC không bị ngắt; giữ status page | Không tự suy diễn nguyên nhân; không cam kết ETA mà IC chưa nói | EM, PO, hoặc một Tech Lead khác team |
| **Scribe** | Ghi timeline theo dấu thời gian: quan sát, quyết định, hành động, ai làm | Không tham gia sửa | Bất kỳ ai, kể cả junior — đây là vai học được nhiều nhất |

Vai Scribe hay bị coi là thừa. Nó không thừa vì ba lý do: (a) postmortem chất lượng cần timeline
chính xác tới phút, và ký ức con người sau 3 giờ căng thẳng thì không dùng được; (b) ở fintech,
timeline là tài liệu phải nộp; (c) chính hành động ghi lại "ai đã đổi gì" là cơ chế ngăn mutation
xung đột.

**Mức severity với tiêu chí khách quan.** Tiêu chí phải khách quan tới mức người on-call lúc 3 giờ
sáng, đầu chưa tỉnh, vẫn phân loại đúng mà không cần hỏi ai. Bảng dưới đây là mẫu cho một ví điện tử;
số là minh hoạ.

| Mức | Tiêu chí khách quan (thoả 1 trong các dòng) | Ai được tuyên bố | Hành động bắt buộc |
|---|---|---|---|
| **SEV1** | Không thực hiện được giao dịch tiền; hoặc lỗi ảnh hưởng >20% người dùng đang hoạt động; hoặc rủi ro sai lệch số dư; hoặc rò rỉ dữ liệu | **Bất kỳ engineer nào**, kể cả junior, không cần xin phép | Gọi điện thoại (không chỉ Slack) IC on-call; lập war room; cập nhật stakeholder mỗi 30 phút; postmortem bắt buộc trong 5 ngày |
| **SEV2** | Một luồng nghiệp vụ chính bị suy giảm nặng nhưng có đường đi thay thế; error rate >5% ở một service chính; chậm gấp >5 lần bình thường | Bất kỳ engineer nào | War room; IC on-call; cập nhật mỗi 60 phút; postmortem bắt buộc |
| **SEV3** | Lỗi ảnh hưởng nhóm nhỏ người dùng hoặc một tính năng phụ; có workaround | On-call | Xử lý trong giờ làm việc; ghi ticket; postmortem nếu lặp lại lần thứ hai |
| **SEV4** | Bất thường nội bộ, chưa ảnh hưởng người dùng | On-call | Ticket, xử lý theo backlog |

Hai quy tắc kèm bảng này quan trọng hơn bảng:

- **Ai cũng được tuyên bố SEV1, và tuyên bố sai không bị phạt.** Nếu chỉ manager được tuyên bố, bạn
  vừa thêm một bước chờ vào con đường tới hạn. Nếu tuyên bố sai bị chê, người ta sẽ ngần ngừ 20 phút.
  Chi phí của một SEV1 tuyên bố thừa là vài người bị gọi dậy oan. Chi phí của một SEV1 tuyên bố muộn
  30 phút, ở fintech, lớn hơn nhiều bậc.
- **Leo thang một chiều là mặc định.** Trong 15 phút đầu, nếu không rõ, chọn mức cao hơn. Hạ mức về
  sau dễ; kéo lại 20 phút đã mất thì không.

**Timeline 15 phút đầu.** Đây là phần cần thuộc, vì 15 phút đầu quyết định phần lớn tổng thời gian.

| Phút | Việc | Người |
|---|---|---|
| 0–2 | Xác nhận có thật không (một dấu hiệu độc lập với alert: thử luồng người dùng thật, xem đồ thị business metric) | On-call |
| 2–3 | Tuyên bố severity; mở channel; đăng opening message | On-call |
| 3–5 | Nhận vai: ai là IC, ai là Ops Lead, ai Comms, ai Scribe. Nói ra bằng chữ trong channel | IC |
| 5–8 | Trả lời một câu hỏi duy nhất: **có thay đổi nào vừa vào hệ thống trong 2 giờ qua không** (deploy, config, feature flag, migration, thay đổi của đối tác) | Ops Lead |
| 8–12 | Nếu có thay đổi → quyết định rollback. Nếu không → phát biểu 2–3 giả thuyết cạnh tranh và gán người kiểm chứng | IC |
| 12–15 | Cập nhật stakeholder lần đầu, kể cả khi chưa biết nguyên nhân | Comms Lead |

**Quy tắc mitigate trước, hiểu nguyên nhân sau.** Phát biểu vận hành được: *trong khi hệ thống còn
ảnh hưởng người dùng, mọi hành động phải nhằm giảm ảnh hưởng; điều tra nguyên nhân gốc chỉ được chiếm
băng thông sau khi ảnh hưởng đã được chặn, hoặc khi việc điều tra là điều kiện để mitigate.* Bốn công
cụ mitigate theo thứ tự ưu tiên vì phương sai thời gian tăng dần: (1) rollback / tắt feature flag;
(2) chuyển traffic sang vùng hoặc phiên bản khác; (3) giảm tải có chọn lọc — tắt tính năng phụ, bật
rate limit, dừng job batch; (4) tăng capacity. Đặt "sửa code" ra ngoài danh sách này: một hotfix viết
lúc 3 giờ sáng dưới áp lực có xác suất tạo incident thứ hai đủ cao để nó chỉ nên là lựa chọn cuối.

Kèm theo là quy tắc bảo toàn bằng chứng: **trước khi mitigate, chụp lại cái đủ để điều tra sau** —
snapshot log 10 phút, một heap dump nếu rẻ, ảnh dashboard, danh sách pod đang lỗi. Việc này tốn 60–90
giây và cứu cả postmortem.

**Khi nào rollback.** Rollback là hành động mặc định, không phải hành động cuối:

- Có deploy hoặc thay đổi config trong 2 giờ trước khi sự cố bắt đầu, và mốc thời gian khớp → rollback
  ngay, không cần chứng minh nhân quả trước. Nhân quả chứng minh sau, trong postmortem.
- Không rollback khi: thay đổi có migration dữ liệu không nghịch đảo được; rollback làm mất dữ liệu
  đã ghi ở phiên bản mới; phiên bản cũ đã biết là có lỗi nặng hơn. Ba điều kiện này phải được biết
  **trước** khi deploy, không phải phát hiện lúc 3 giờ sáng — đó là một tiêu chí của định nghĩa Done.
- Nếu không rollback được, hãy coi đó là một phát hiện về kiến trúc và ghi thành action item ưu tiên
  cao: một hệ thống không rollback được thì mọi deploy đều là quyết định một chiều.

**Nhịp cập nhật stakeholder.** Nhịp phải cố định và công bố trước, vì giá trị lớn nhất của nó là loại
bỏ việc stakeholder phải hỏi.

| Severity | Nhịp | Kênh | Ai nhận |
|---|---|---|---|
| SEV1 | 30 phút, đúng giờ, kể cả khi không có gì mới | Channel riêng + email + status page | Founder/CTO, Support, Business, Compliance (fintech) |
| SEV2 | 60 phút | Channel riêng | Head of Product, Support, EM liên quan |
| SEV3 | Khi bắt đầu và khi xong | Ticket | Team liên quan |

**TEMPLATE — Incident channel opening message.** Đăng ngay khi mở channel, sửa dần bằng cách edit tin
nhắn đầu và pin nó.

```text
[INCIDENT] SEV2 — Thanh toán thẻ lỗi 40% từ 02:31
Trạng thái: ĐANG ĐIỀU TRA
Bắt đầu (theo dữ liệu): 02:31 (+07)  |  Phát hiện: 02:39  |  Tuyên bố: 02:41

VAI
- Incident Commander: @minh  (người quyết; mọi thay đổi production phải qua tôi)
- Ops Lead:           @tuan  (người DUY NHẤT được thao tác production lúc này)
- Communications:     @trang (cập nhật stakeholder; hỏi gì về tiến độ hỏi tôi)
- Scribe:             @vy    (ghi timeline; ai làm gì báo tôi một dòng)

ẢNH HƯỞNG (cập nhật liên tục)
- Người dùng: ~40% giao dịch thẻ thất bại; ví nội bộ và QR không ảnh hưởng
- Nghiệp vụ: giao dịch không bị treo tiền (đã kiểm tra 3 mẫu), đang xác nhận thêm
- Quy mô: ~1.200 giao dịch lỗi tính từ 02:31 (số đo lúc 02:45)

ĐANG LÀM
- Kiểm tra thay đổi trong 2h qua: deploy payment-svc 01:58 -> @tuan đang xem
- Giả thuyết đang xét: (1) timeout tới cổng thanh toán  (2) connection pool cạn

QUY TẮC CHANNEL
- Không chạy lệnh trên production nếu chưa nói ở đây và IC chưa xác nhận.
- Câu hỏi "bao lâu nữa xong" gửi @trang, không gửi @minh và @tuan.
- Ai vào sau: đọc tin nhắn đã pin này, KHÔNG hỏi "tình hình sao rồi".

CẬP NHẬT TIẾP THEO: 03:15 (+07)
```

Ba chi tiết trong template này làm phần lớn công việc: dòng "người DUY NHẤT được thao tác production"
thực thi mô hình single writer; dòng "ai vào sau đọc pin" cắt hàng loạt câu hỏi lặp; dòng "cập nhật
tiếp theo" biến sự im lặng từ dấu hiệu đáng lo thành trạng thái đã được thông báo.

**TEMPLATE — Cập nhật stakeholder.** Viết cho người không đọc log. Cấm thuật ngữ nội bộ. Không đoán
nguyên nhân trước khi có bằng chứng.

```text
CẬP NHẬT #3 — 03:15 (+07)  |  SEV2 — Lỗi thanh toán thẻ
Trạng thái: ĐÃ GIẢM THIỂU, đang theo dõi

ẢNH HƯỞNG HIỆN TẠI
- Tỉ lệ giao dịch thẻ thất bại: 40% (02:45) -> 3% (03:12), bình thường là 1,5%
- Ví nội bộ, QR, chuyển tiền: không bị ảnh hưởng trong suốt sự cố
- Tiền của khách: không có giao dịch treo. Đã đối soát 02:31–03:10, khớp.

CHÚNG TÔI ĐÃ LÀM GÌ
- 03:04 hoàn tác bản cập nhật lúc 01:58 của service thanh toán
- 03:12 xác nhận tỉ lệ lỗi trở lại mức bình thường

VIỆC CẦN CÁC BÊN LÀM
- Support: dùng mẫu trả lời đã gửi trong #support-notice; ~1.200 giao dịch lỗi,
  khách có thể thanh toán lại bình thường từ 03:12
- Business: chưa cần hành động
- Compliance: hồ sơ sự cố sẽ có bản nháp trước 12:00 hôm nay

CHƯA BIẾT
- Vì sao bản cập nhật 01:58 gây lỗi này: đang điều tra, không ảnh hưởng khách hàng
- Kết luận nguyên nhân gốc sẽ nằm trong Postmortem, hạn 30/07

CẬP NHẬT TIẾP THEO: 03:45, hoặc ngay nếu có thay đổi
```

Phần "CHƯA BIẾT" là phần khiến bản cập nhật này đáng tin. Một bản cập nhật chỉ có tin tốt sẽ bị đọc
như PR nội bộ; một bản cập nhật ghi rõ ranh giới hiểu biết sẽ được đọc như báo cáo kỹ thuật.

### Trade-off

| Cặp đối lập | Nghiêng về bên A khi | Nghiêng về bên B khi | Ai trả giá |
|---|---|---|---|
| **Command-and-control (A) vs consensus (B)** | Chi phí mỗi phút cao; nhiều người tham gia; hành động có tác dụng lên production | Ảnh hưởng đã chặn; bước tiếp theo là quyết định một chiều (migration, xoá dữ liệu) | A: người có ý tưởng tốt bị bỏ qua trong lúc đó. B: người dùng trả bằng downtime |
| **Mitigate trước (A) vs giữ nguyên hiện trường để điều tra (B)** | Sự cố đang ảnh hưởng người dùng và có mitigation phương sai thấp | Lỗi liên quan dữ liệu/tiền và mitigate có thể làm sai lệch không hồi được; hoặc hệ thống đã tự ổn định | A: postmortem mất bằng chứng nếu không snapshot. B: khách hàng chịu thêm downtime |
| **Tuyên bố severity cao sớm (A) vs chờ chắc chắn (B)** | Fintech, hệ thống liên quan tiền, giờ cao điểm campaign | Tổ chức nhỏ, mọi SEV1 làm cả team mất một ngày làm việc | A: chi phí huy động, nguy cơ alert fatigue ở tầng lãnh đạo. B: 20 phút mất đi không lấy lại được |
| **Minh bạch tối đa với stakeholder (A) vs kiểm soát thông tin (B)** | Khách B2B có SLA; đối tác cần biết để ứng phó | Chưa xác nhận phạm vi; thông tin sai lan ra ngoài gây thiệt hại lớn hơn im lặng 20 phút | A: có thể phải rút lại phát biểu. B: mất niềm tin nếu bị phát hiện biết mà không nói |

Một trade-off ít được nói: **cấu trúc chỉ huy chỉ vận hành được nếu tổ chức đã tách quyền chỉ huy khỏi
thâm niên.** Ở bối cảnh Việt Nam, việc một Senior 3 năm kinh nghiệm làm IC và điều phối một Technical
Architect 15 năm là điều nhiều tổ chức thấy khó. Nếu bạn không giải quyết được điều này trước, vai IC
sẽ mặc định rơi về người thâm niên nhất — mà người đó thường cũng là người sửa giỏi nhất, và bạn quay
lại đúng anti-pattern đầu tiên.

### Real-world Scenarios

**Tình huống 1 — Fintech: 41 phút và một câu hỏi bị bỏ.** Ví điện tử ở đầu chương, sau khi áp cấu
trúc. 02:31 error rate cổng thẻ tăng. 02:39 alert. On-call là Vy, mới vào 4 tháng. Vy mở luồng thanh
toán trên điện thoại, thất bại thật, tuyên bố SEV2 lúc 02:41 và gọi điện cho IC on-call là Minh.
02:44 vai được phân trong channel. 02:46 Tuấn (Ops Lead) báo có deploy payment-svc lúc 01:58. Minh
quyết: *"Rollback. Không cần biết vì sao trước."* 03:04 rollback xong, 03:12 tỉ lệ lỗi bình thường.

Điều đáng học không nằm ở phần thành công mà ở một chi tiết: giữa 02:46 và 03:04 có 18 phút, vì
rollback service này cần duyệt thủ công của một người trong nhóm hạ tầng đang ngủ. Trong postmortem,
action item quan trọng nhất không phải "sửa bug trong bản 01:58" mà **"rollback payment-svc phải thực
hiện được bởi on-call trong 5 phút không cần duyệt"**. Đây là kiểu phát hiện chỉ lộ ra khi có Scribe
ghi từng mốc; nếu chỉ có ký ức, 18 phút đó biến thành "rollback mất một lúc".

**Tình huống 2 — E-commerce giữa campaign: khi mitigate đúng là hy sinh tính năng.** 20:15 ngày
11/11, hệ thống check-out chậm dần, p95 từ 800ms lên 9 giây. Không có deploy nào trong 6 giờ. Nguyên
nhân sau này biết là một truy vấn trong service gợi ý sản phẩm ở trang giỏ hàng, chạy đồng bộ trong
đường đi tới hạn. Trong lúc incident, IC là Khoa quyết định điều nhiều người không thích: **tắt hoàn
toàn block gợi ý sản phẩm ở giỏ hàng** bằng feature flag lúc 20:34. p95 về 1,1 giây lúc 20:36.

PO là Hà phản ứng trong channel: "Block đó đóng góp 8% doanh thu giỏ hàng, tắt là mất tiền." Khoa trả
lời: *"Tôi hiểu. Nhưng hiện tại check-out đang mất khoảng 60% người dùng ở bước cuối vì chậm. Tôi ưu
tiên giữ check-out. Nếu 30 phút nữa ổn định, chúng ta bật lại có giới hạn. Quyết định này là của tôi,
tôi ghi vào timeline." Đây là một mẫu hành vi đáng chép lại: IC không tranh luận về con số 8%, không
xin phép, nhưng nói rõ **cơ sở so sánh**, **tính tạm thời** và **ai chịu trách nhiệm**.

**Tình huống 3 — ODC, khách ở múi giờ khác: hai đồng hồ và một cuộc gọi lúc 6 giờ sáng.** Một công ty
ODC ở Đà Nẵng vận hành hệ thống cho khách Nhật. Sự cố bắt đầu 23:40 giờ Việt Nam, tức 01:40 giờ Nhật.
Team xử lý xong lúc 01:10 (03:10 giờ Nhật) và đi ngủ, định báo khách sáng hôm sau. 09:00 giờ Nhật,
khách phát hiện qua báo cáo của chính họ và gửi một email dài, kèm câu hỏi "vì sao chúng tôi phải là
người phát hiện".

Vấn đề không phải chất lượng xử lý kỹ thuật — nó tốt. Vấn đề là đồng hồ thứ hai chạy 7 giờ. Trong bối
cảnh ODC, hai thiết kế bắt buộc: (a) mọi SEV1/SEV2 có một bản cập nhật viết bằng tiếng Anh gửi trong
30 phút đầu, kể cả khi phía khách đang ngủ — dấu thời gian trên email là bằng chứng bạn đã báo ngay;
(b) template cập nhật phải viết sẵn bằng tiếng Anh và Comms Lead là người có tiếng Anh tốt nhất trong
phiên, không phải người rảnh nhất. Rào cản tiếng Anh lúc 1 giờ sáng là một rủi ro vận hành thật, và
cách xử lý nó là chuẩn bị trước bằng template, không phải kỳ vọng ai đó viết hay dưới áp lực.

**Cùng một sự việc, ba góc nhìn.** Lấy tình huống 2 (tắt block gợi ý sản phẩm):

| | Người đó thấy gì | Câu hỏi thật của họ | Sai lầm thường gặp ở góc nhìn này |
|---|---|---|---|
| **IC / Senior BE (Quân, đang debug)** | Một truy vấn N+1 mà anh đã cảnh báo trong review 4 tháng trước và bị hoãn | "Vì sao cảnh báo của tôi không được nghe, và bây giờ tôi lại là người thức đêm?" | Nói "tôi đã bảo rồi" trong channel lúc đang cháy. Đúng về nội dung, sai về thời điểm; nó biến war room thành nơi tính sổ |
| **Tech Lead (Khoa, đang làm IC)** | Một quyết định đánh đổi doanh thu tính năng phụ để giữ luồng chính, phải ra trong 2 phút với dữ liệu không đủ | "Tôi có dám ra một quyết định mà PO không đồng ý, và tôi có bảo vệ được nó sáng mai không?" | Đi tìm sự đồng thuận của PO trước khi hành động, mất 15 phút giữa peak |
| **Manager / EM (Trang)** | Hai người đã thức 3 đêm trong tháng; một PO đang lo doanh thu campaign; một CTO sẽ hỏi vì sao lặp lại | "Đây là sự cố kỹ thuật hay là hệ quả của việc tôi đã để cảnh báo của Quân bị hoãn 4 tháng?" | Xử lý phần kỹ thuật, bỏ phần quản trị: không truy lại vì sao cảnh báo bị hoãn, nên quý sau lặp lại |

Ba góc nhìn này cùng đúng và không thể hoà giải trong lúc incident. Chúng chỉ hoà giải được trong
postmortem — đó là lý do postmortem không phải nghi lễ mà là bộ phận cấu trúc của hệ thống.

### Best Practices

- **Tách người chỉ huy khỏi người sửa, kể cả với incident nhỏ.** Lý do không phải phân công cho đẹp:
  người đang đọc stack trace mất khả năng theo dõi trạng thái tổng thể, và đó là một giới hạn nhận
  thức, không phải vấn đề ý chí. Nếu team quá nhỏ để tách, hãy để người sửa nói to mọi hành động và
  người thứ hai làm IC kiêm Scribe — vẫn tốt hơn một người làm cả.
- **Cho phép mọi người tuyên bố severity cao nhất, và cảm ơn công khai người tuyên bố thừa.** Bạn
  đang mua độ nhạy của hệ thống phát hiện bằng một chi phí huy động nhỏ. Nếu bạn phạt tuyên bố thừa,
  bạn sẽ mất độ nhạy, và bạn sẽ không biết mình đã mất.
- **Diễn tập vai IC với người không phải chuyên gia hệ thống.** Nếu chỉ hai người trong tổ chức làm
  được IC, bạn có một single point of failure về quản trị. Vai IC là kỹ năng điều phối và học được
  trong 2–3 lần đóng vai, nhanh hơn nhiều so với việc học một hệ thống.
- **Viết nhịp cập nhật vào lịch, không dựa vào ý chí.** Đặt hẹn giờ. Con người dưới áp lực sẽ trượt
  mọi việc không có tín hiệu nhắc.
- **Kết thúc incident bằng một tuyên bố rõ ràng, có chữ.** "Incident đóng lúc 04:10. Trạng thái: đã
  khôi phục. Theo dõi thêm 24 giờ. Postmortem: 30/07, chủ trì @minh." Không có tuyên bố đóng, một số
  người vẫn ở trạng thái báo động thêm nhiều giờ và mất một ngày làm việc hôm sau.
- **Ghi mọi thay đổi production vào một dòng trong channel trước khi thực hiện.** Đây là log tường
  thuật rẻ nhất bạn có, và nó là công cụ ngăn mutation xung đột hiệu quả hơn mọi quy định.

### Anti-patterns

- **Người giỏi nhất vừa chỉ huy vừa sửa.** Cơ chế phá hoại: khi người đó chìm vào một giả thuyết, cả
  incident chìm theo, vì không còn ai giữ khung thời gian và không ai dám ngắt. Dấu hiệu sớm: trong
  channel, cùng một tên vừa trả lời câu hỏi của founder, vừa dán output của lệnh, vừa nói "để tôi
  xem". Cách sửa tại chỗ: một người khác gõ "Tuấn, anh tập trung sửa, tôi làm IC từ giờ" — hành động
  này hợp lệ với bất kỳ ai và không cần thẩm quyền.
- **Im lặng với stakeholder vì "chưa có thông tin".** Cơ chế phá hoại: im lặng không tạo ra khoảng
  trống, nó tạo ra chỗ cho suy diễn xấu nhất và cho các cuộc gọi trực tiếp vào đúng người đang sửa.
  Dấu hiệu sớm: câu "chờ có gì chắc chắn rồi báo". Cách sửa: một bản cập nhật hợp lệ chỉ cần ba dòng
  — ảnh hưởng đã biết, đang làm gì, khi nào cập nhật lần sau. Không cần nguyên nhân.
- **Nhiều người sửa song song không ai biết ai đã thay đổi gì.** Cơ chế phá hoại: mất khả năng gán
  nhân quả giữa hành động và kết quả, nên incident có thể "tự khỏi" mà không tạo ra bài học nào; đồng
  thời tăng xác suất tạo incident thứ hai. Dấu hiệu sớm: hai người cùng nói "tôi vừa thử..." trong
  vòng một phút; hoặc trong postmortem không ai dựng lại được thứ tự hành động.
- **War room mở cho tất cả và không ai điều tiết.** Dấu hiệu sớm: hơn 3 câu hỏi "tình hình sao rồi"
  trong 10 phút. Cách sửa: một channel kỹ thuật hẹp cho người có vai, một channel rộng chỉ đọc cho
  người quan sát, Comms Lead nối hai kênh.
- **Severity định nghĩa bằng tính từ.** "SEV1 là sự cố nghiêm trọng ảnh hưởng lớn tới người dùng" là
  một định nghĩa không phân loại được gì lúc 3 giờ sáng. Dấu hiệu sớm: các cuộc tranh luận về mức
  severity chiếm nhiều thời gian hơn hành động.
- **Hotfix trực tiếp lên production như phản xạ đầu tiên.** Dấu hiệu sớm: trong 5 incident gần nhất,
  số lần mitigate bằng hotfix nhiều hơn số lần bằng rollback hoặc feature flag.

### Khi nào KHÔNG nên áp dụng

- **Team dưới 5 người, hệ thống một khối, ít incident.** Bộ bốn vai đầy đủ với 4 người là nghi lễ:
  bạn sẽ có một IC không có ai để điều phối. Phần cần giữ lại chỉ là ba thứ: định nghĩa severity
  khách quan, quy tắc một người thao tác production, và một dòng cập nhật cho stakeholder. Bỏ Scribe
  riêng, dùng chính channel làm timeline.
- **Sự cố mức SEV3/SEV4.** Mở war room cho một lỗi ảnh hưởng 12 người dùng là cách nhanh nhất để làm
  cả tổ chức mất nhạy cảm với war room. Chi phí ngắt việc của 8 người trong 45 phút vượt giá trị.
- **Sự cố đã có runbook rõ và một hành động duy nhất.** Nếu alert "disk đầy trên node báo cáo" có
  runbook 4 bước đã chạy 30 lần, cấu trúc chỉ huy là thủ tục thừa. Ranh giới: khi hành động theo
  runbook không cho kết quả mong đợi trong 10 phút, chuyển sang chế độ incident có cấu trúc.
- **Sự cố không nằm trong tay bạn và không có mitigation.** Khi một nhà cung cấp thanh toán ngoài
  ngừng dịch vụ và bạn không có cổng dự phòng, việc mở war room 20 người trong 3 giờ chỉ tiêu tốn
  người. Đúng hơn: một người theo dõi và cập nhật, Comms Lead làm việc với đối tác và stakeholder,
  còn lại về ngủ. Nhưng lưu ý điều này chỉ đúng **một lần**: sau đó, "không có phương án dự phòng cho
  đối tác" phải trở thành một quyết định kiến trúc được ghi nhận, không phải một sự đã rồi.
- **Giai đoạn tổ chức chưa có bất kỳ observability nào.** Nếu bạn không có dashboard và không có log
  tập trung, đầu tư đầu tiên không phải quy trình incident mà là khả năng nhìn thấy. Cấu trúc chỉ huy
  điều phối hành động dựa trên tín hiệu; không có tín hiệu, nó chỉ điều phối các phỏng đoán.

---

## 2. Incident Leadership — hành vi của lead trong và sau incident

### Problem Statement

Chủ đề 1 nói về cấu trúc. Cấu trúc là điều kiện cần và không đủ, vì bạn có thể có đủ bốn vai, đủ bảng
severity, đủ template — và vẫn có một incident tệ, vì hành vi của người lead làm giảm năng lực của
những người đang xử lý.

Ba hiện tượng quan sát được:

**Hiện tượng một: một engineer giỏi bỗng làm những việc sơ đẳng.** Trong incident, Tuấn — người viết
phần lớn service thanh toán — gõ sai tên namespace hai lần, đọc sai một biểu đồ mà anh xem hàng ngày,
và quên rằng có sẵn một script rollback do chính anh viết. Đây không phải dấu hiệu năng lực; đây là
biểu hiện đo được của việc năng lực nhận thức khả dụng bị chiếm bởi stress. Nếu lead phản ứng bằng cách
tăng áp lực ("nhanh lên, mỗi phút mất mấy chục triệu"), phần khả dụng còn lại giảm thêm.

**Hiện tượng hai: người vừa gây ra lỗi biến mất khỏi cuộc trò chuyện.** Duy, mới vào 3 tháng, deploy
một thay đổi config lúc 22:10. Khi sự cố bắt đầu, Duy nhận ra ngay đó có thể là mình, nhưng không nói.
20 phút sau, Tuấn tình cờ tìm thấy trong audit log. 20 phút đó là phần chi phí trực tiếp của việc tổ
chức chưa làm cho việc thừa nhận trở nên an toàn. Ở bối cảnh Việt Nam, hiện tượng này nặng hơn trung
bình: cơ chế "giữ mặt" khiến việc tự nhận lỗi trước đông người có giá tâm lý cao.

**Hiện tượng ba: incident kết thúc nhưng chi phí con người mới bắt đầu.** Sau một sự cố 5 giờ đêm thứ
Ba, hôm sau ba người vẫn có mặt lúc 9 giờ, dự stand-up, và không ai làm được gì có ý nghĩa. Số minh
hoạ đáng theo dõi: sau một đêm mất 4 giờ ngủ, năng suất ngày hôm sau của một người làm việc trí óc
giảm rất đáng kể, và xác suất người đó tạo ra lỗi mới tăng — nghĩa là để họ tiếp tục làm việc là một
quyết định làm tăng rủi ro chứ không giảm.

Hậu quả tích lũy: các incident được xử lý xong về mặt kỹ thuật nhưng để lại một lớp trầm tích — người
sợ deploy, người tránh làm việc ở service dễ hỏng, người từ chối on-call, và cuối cùng là đơn xin nghỉ
của đúng những người bạn cần nhất. Đó là lý do Incident Leadership không phải phần "mềm" của chủ đề
này; nó là phần quyết định tổ chức còn ai để xử lý incident sau.

### First Principles

**Cơ chế một: dưới stress cao, dung lượng làm việc của nhận thức bị chiếm dụng.** Trí nhớ làm việc của
con người có dung lượng nhỏ và bị chia sẻ. Trong incident, một phần dung lượng đó bị chiếm bởi những
thứ không phải bài toán kỹ thuật: lo bị quy trách nhiệm, theo dõi xem founder có đang đọc, tính xem
mình sẽ nói gì sáng mai, đếm số tiền đang mất. Mọi thứ chiếm dung lượng đó đều lấy đi từ phần dùng để
suy luận về hệ thống. Suy ra một phát biểu vận hành được: **công việc của lead trong incident là giải
phóng dung lượng nhận thức của người đang sửa, không phải bổ sung thông tin về mức độ nghiêm trọng.**
Người đang sửa đã biết nó nghiêm trọng.

**Cơ chế hai: stress làm hẹp trường chú ý (attentional narrowing).** Đây là hiện tượng được mô tả
rộng rãi trong tài liệu về hiệu suất dưới áp lực: khi căng thẳng tăng, con người tập trung sâu vào một
giả thuyết và mất khả năng quét các khả năng khác. Trong incident, nó biểu hiện thành việc một engineer
đào 40 phút vào một hướng đã hết dữ liệu ủng hộ. Người đó không thể tự thoát, vì chính cơ chế đang
giữ họ ở đó cũng làm họ không thấy nó. Chỉ người **không** ở trong trạng thái đó mới ngắt được — đây
là một lý do chức năng nữa cho việc IC không sửa.

**Cơ chế ba: hành vi của người có quyền lực được khuếch đại phi tuyến trong tình huống bất định.** Một
câu hỏi bình thường của founder — "sao chậm thế" — trong bối cảnh ngày thường là một câu hỏi. Lúc 2 giờ
sáng, giữa incident, nó được đọc như một phán xét, và nó tạo ra một dòng công việc nhận thức mới cho
mọi người đọc được. Cơ chế: trong bất định, con người dò tìm tín hiệu về hậu quả cá nhân, và tín hiệu
từ người có quyền được gán trọng số cao nhất. Đây không phải lỗi của founder; đó là thuộc tính của
kênh. Vai của lead là quản lý kênh đó.

**Cơ chế bốn: truy tìm nguyên nhân về người trong lúc đang cháy làm chậm chính việc tìm nguyên nhân về
hệ thống.** Nghe phản trực giác nhưng cơ chế rất trực tiếp: nếu việc "ai đổi gì" gắn với hậu quả cá
nhân, thông tin về thay đổi sẽ đến muộn hơn hoặc bị làm nhẹ. Mà thông tin "có gì vừa thay đổi" là đúng
thông tin đắt giá nhất trong 15 phút đầu (xem chủ đề 1, phút 5–8). Vậy hành vi truy tìm người trực
tiếp làm tăng MTTR. Đây là lập luận công cụ, không phải lập luận đạo đức — nó thuyết phục được cả
những người không quan tâm tới cảm xúc của team.

**Cơ chế năm: sự tỉnh táo suy giảm theo thời gian thức, và nó suy giảm mà người ta không nhận thấy.**
Đặc điểm nguy hiểm nhất của thiếu ngủ là khả năng tự đánh giá suy giảm nhanh hơn năng lực thực tế: sau
4 giờ căng thẳng lúc nửa đêm, người ta vẫn tin mình còn ổn. Nên quyết định luân phiên không thể để
người đang trong incident tự đưa ra; nó phải là một quy tắc được áp từ ngoài theo đồng hồ.

### Mental Model

**Mô hình một: lead như một bộ lọc thông tin hai chiều.** Vẽ ra thì đơn giản và nó quyết định phần lớn
hành vi đúng:

```
Stakeholder / Founder / Support / Khách hàng
        ↓ (câu hỏi, áp lực, deadline, suy diễn)
   [ LEAD = bộ lọc + bộ đệm ]      <-- nén, loại nhiễu, dịch ngôn ngữ
        ↓ (chỉ những gì thay đổi hành động)
   Người đang sửa (Ops Lead, engineer)
        ↑ (trạng thái thô, giả thuyết, độ không chắc chắn)
   [ LEAD = bộ dịch ]              <-- dịch sang ảnh hưởng nghiệp vụ
        ↑
Stakeholder
```

Phép thử cho mỗi thông tin đi xuống: **thông tin này có làm người đang sửa đổi hành động không?** Nếu
không, nó là nhiễu và lead giữ lại. "Founder đang lo" không đổi hành động. "Support nói khách hàng ở
Đà Nẵng vẫn thanh toán được" đổi hành động — đó là dữ liệu về phạm vi.

**Mô hình hai: Cognitive Load Budget của incident.** Coi mỗi người đang xử lý có một ngân sách nhận
thức, và mọi thứ đều tiêu vào đó: bài toán kỹ thuật, lo lắng, câu hỏi, tiếng ping. Lead có ba đòn bẩy
để tăng phần dành cho bài toán: (a) giảm số nguồn ngắt; (b) thu hẹp phạm vi câu hỏi mà người đó phải
trả lời — "anh chỉ cần trả lời một câu: connection pool có cạn không"; (c) gỡ bỏ rủi ro cá nhân bằng
lời nói rõ ràng.

**Mô hình ba: hai chế độ của lead — Incident Mode và Learning Mode, tách nhau bằng một đường vạch
rõ.** Hai chế độ này đòi hỏi hành vi ngược nhau, và phần lớn sai lầm của lead là trộn chúng.

| | Incident Mode | Learning Mode |
|---|---|---|
| Mục tiêu | Giảm ảnh hưởng nhanh nhất | Hiểu hệ thống đã cho phép điều này xảy ra thế nào |
| Câu hỏi hợp lệ | "Cái gì vừa thay đổi?", "Cách nhanh nhất để chặn?" | "Vì sao thay đổi đó qua được mọi lớp kiểm soát?" |
| Câu hỏi bị cấm | "Ai làm?", "Sao lại không test?" | Không có câu hỏi bị cấm |
| Với người gây lỗi | Bảo vệ, giữ họ trong cuộc, dùng thông tin của họ | Phỏng vấn kỹ, nhưng về hệ thống chứ không về ý định |
| Nhịp | Phút | Ngày |

Đường vạch giữa hai chế độ nên được nói ra bằng lời: *"Bây giờ mình chỉ sửa. Mọi câu hỏi vì sao để
postmortem thứ Năm."* Câu này làm hai việc cùng lúc: nó bảo vệ người đang sửa, và nó cam kết rằng câu
hỏi "vì sao" sẽ được hỏi thật — nếu không, nó thành một cách trốn trách nhiệm.

### Practical Framework

**A. 30 phút đầu: điều lead nói và không nói.**

| Thời điểm | Nên nói (và vì sao) | Không nên nói (và cơ chế phá hoại) |
|---|---|---|
| Vừa vào channel | "Tôi đã vào. @minh vẫn là IC, tôi không đổi gì. Cần gì từ tôi thì nói." | "Tình hình sao rồi?" — buộc người đang sửa dừng để tóm tắt lại cho bạn |
| Khi thấy hướng đi có vẻ sai | "Mình đã ở hướng A 15 phút. Tôi muốn nghe một giả thuyết khác trong 2 phút, rồi quyết." | "Không phải cái đó đâu." — phủ định không kèm hướng thay thế làm mất cả hướng cũ |
| Khi có người mới vào | "Đọc tin nhắn pin. Nếu muốn giúp, việc cần là X." | Để mặc, rồi 10 phút sau có 5 câu hỏi trùng |
| Khi biết ai gây ra | "Được rồi, thông tin quan trọng. Duy, cụ thể em đổi giá trị nào?" | "Ai deploy?" — biến thông tin thành rủi ro cá nhân |
| Khi bị hỏi ETA | "Chưa có ETA. 03:15 tôi cập nhật, kể cả nếu chưa xong." | "Khoảng 15 phút nữa" khi không có cơ sở — hết 15 phút bạn mất uy tín và phải giải thích lần hai |
| Khi qua 60 phút | "Tuấn, nghỉ 5 phút, uống nước. Khoa tiếp phần này." | "Cố lên, gần xong rồi" — tăng áp lực, không giảm tải |
| Khi đã mitigate | "Ảnh hưởng đã chặn. Từ giờ không ai đổi gì trên production nữa. Điều tra để mai." | Để mọi người tiếp tục dò nguyên nhân tới 5 giờ sáng vì tò mò |

**B. Xử lý khi cấp trên hoặc founder nhảy vào channel.** Ba bước, làm trong vòng 60 giây:

1. **Nhận biết công khai.** Người có quyền vào channel là đang tìm thông tin và tìm sự an tâm. Bỏ qua
   họ sẽ làm họ hỏi to hơn.
2. **Cung cấp cấu trúc, không cung cấp lời hứa.** Cho họ ba thứ: ảnh hưởng đã biết, việc đang làm,
   mốc cập nhật tiếp theo. Không cho ETA nếu không có cơ sở.
3. **Định tuyến lại kênh.** Chuyển họ sang một cuộc trò chuyện riêng với bạn hoặc Comms Lead, để
   channel kỹ thuật sạch trở lại.

**Script 1 — Founder nhảy vào channel lúc 2 giờ sáng hỏi dồn.**

Bối cảnh: startup fintech Series A. Sơn là founder/CEO. Minh là Engineering Manager, đang làm Comms
Lead. Tuấn là Ops Lead, đang thao tác.

*Phiên bản lead xử lý sai:*

```text
[02:52] Sơn (founder): mọi người ơi tình hình sao rồi
[02:52] Sơn: có ai đang làm gì không, tôi thấy im lặng 10 phút
[02:53] Sơn: bao lâu nữa xong? sáng mai bên đối tác hỏi tôi trả lời sao
[02:53] Minh (EM): Tuấn, anh Sơn đang hỏi, em cập nhật nhanh giúp anh
[02:54] Tuấn: đang xem ạ, chưa rõ nguyên nhân
[02:54] Sơn: chưa rõ là sao? đã 20 phút rồi
[02:55] Minh: em nghĩ khoảng 15-20 phút nữa là xong anh
[02:55] Sơn: ok, mà lần trước cũng bảo 15 phút
[02:56] Minh: Tuấn em có chắc 15 phút không?
[02:57] Tuấn: (không trả lời, đang chạy lệnh)
[03:10] Sơn: sao rồi?
[03:11] Minh: vẫn đang ạ
```

Bốn lỗi. Một: Minh chuyển câu hỏi xuống cho Tuấn — đúng ngược chiều của bộ lọc, làm Tuấn phải dừng sửa
để làm báo cáo. Hai: Minh bịa ETA "15-20 phút" không có cơ sở, và mất uy tín ngay ở lần đầu bị soi
lại. Ba: Minh hỏi lại Tuấn "có chắc không" — biến sự bất định của mình thành áp lực cho Tuấn. Bốn:
không có nhịp cập nhật, nên founder phải hỏi lại, và mỗi lần hỏi lại tiêu băng thông của cả channel.

*Phiên bản lead xử lý đúng:*

```text
[02:52] Sơn (founder): mọi người ơi tình hình sao rồi
[02:52] Minh (EM): Em trả lời anh, Tuấn đang thao tác nên em nói thay.

  ẢNH HƯỞNG: ~40% giao dịch thẻ lỗi từ 02:31. Ví nội bộ, QR, chuyển tiền
  vẫn bình thường. Đã kiểm 3 mẫu: không có giao dịch treo tiền của khách.

  ĐANG LÀM: nghi bản deploy 01:58. Tuấn đang chuẩn bị rollback, cần ~10 phút
  vì phải chờ pipeline.

  CHƯA BIẾT: nguyên nhân chính xác. Em sẽ không đoán.

  CẬP NHẬT: em cập nhật lúc 03:15 và mỗi 30 phút, kể cả khi chưa xong,
  anh không cần hỏi.

[02:54] Sơn: rollback rồi có chắc hết không?
[02:54] Minh: Không chắc 100%. Nếu rollback không hết, phương án tiếp theo là
  chuyển sang cổng thanh toán dự phòng, mất thêm khoảng 20 phút. Em sẽ nói rõ
  ở bản cập nhật 03:15.
[02:55] Sơn: đối tác hỏi thì tôi nói gì
[02:55] Minh: Em gửi anh bản tiếng Anh 4 dòng dùng được ngay, trong 5 phút nữa,
  qua tin nhắn riêng. Anh không cần soạn.
[02:56] Sơn: ok. cần gì nói tôi
[02:56] Minh: Có một việc cần anh: nếu phải dùng cổng dự phòng thì phí giao dịch
  cao hơn khoảng 0,3%. Em cần anh xác nhận là được phép, để lúc đó không phải chờ.
[02:57] Sơn: được, cứ dùng
```

Điểm chuyển: Minh không phòng thủ, không hứa, và ở dòng cuối **giao cho founder một việc thuộc đúng
thẩm quyền của founder**. Đây là kỹ thuật hiệu quả nhất khi người có quyền lực lo lắng: sự lo lắng của
họ đến từ cảm giác không có đòn bẩy nào, và cách nhanh nhất để chuyển họ từ chế độ chất vấn sang chế
độ hợp tác là đưa cho họ một quyết định thật mà chỉ họ ra được.

Với các tổ chức mà văn hoá cấp trên đặc biệt mạnh, thêm một bước làm trước incident: thống nhất với
founder một "hợp đồng incident" gồm ba dòng — trong incident, founder nhận cập nhật theo nhịp và không
hỏi trực tiếp engineer; mọi câu hỏi đi qua một người; founder được quyền yêu cầu leo thang bất kỳ lúc
nào. Thoả thuận này phải ký lúc bình thường, không thương lượng được lúc 2 giờ sáng.

**C. Bảo vệ người vừa gây ra lỗi, ngay tại thời điểm đó.**

**Script 2 — Một junior vừa deploy gây sự cố và đang tê liệt.**

Bối cảnh: Duy, BE 8 tháng kinh nghiệm, đổi một biến môi trường về connection pool lúc 22:10 theo một
ticket tối ưu hiệu năng. 22:31 sự cố. 22:48 Duy nhắn riêng cho Tech Lead là Linh: "Chị ơi em nghĩ là
do em."

*Phiên bản nói sai:*

```text
Linh: Sao em không nói sớm? 17 phút vừa rồi mọi người mò khắp nơi.
Duy: Em... em xin lỗi, em tưởng không phải.
Linh: Em có test trên staging không?
Duy: Có ạ, nhưng staging traffic thấp nên...
Linh: Nên là em deploy production 10 giờ đêm mà không nói với ai?
Duy: ...
Linh: Thôi, em vào channel nói với mọi người đi.
```

Cơ chế phá hoại: Linh đúng về mọi dữ kiện và sai về mọi hiệu ứng. Ba câu hỏi đầu đều là câu hỏi
"vì sao em" — mỗi câu buộc Duy chuyển từ trạng thái cung cấp thông tin sang trạng thái tự bảo vệ. Kết
quả tức thời: Linh vẫn chưa biết Duy **đổi giá trị nào từ bao nhiêu thành bao nhiêu** — đúng thông tin
cần. Kết quả dài hạn: lần sau, Duy sẽ mất hơn 17 phút mới nói, hoặc không nói.

*Phiên bản nói đúng:*

```text
Linh: Tốt, đây là thông tin quan trọng nhất trong 20 phút vừa rồi. Cảm ơn em.
  Trước hết: em không bị làm sao cả, và mình sẽ không nói chuyện đúng sai bây giờ.
  Em trả lời chị ba câu, ngắn thôi:
  1. Em đổi biến nào, từ giá trị bao nhiêu thành bao nhiêu?
  2. Đổi ở service nào, môi trường nào, lúc mấy giờ?
  3. Em có thể hoàn lại giá trị cũ trong 5 phút không?

Duy: 1. DB_POOL_MAX từ 20 lên 120, ở order-svc.
     2. production, 22:10.
     3. Có ạ, em revert được, khoảng 3 phút.

Linh: Rõ rồi. Em revert ngay, và vào channel #war-room gõ đúng câu này,
  chị copy sẵn cho em:

  "Mình đổi DB_POOL_MAX của order-svc từ 20 lên 120 lúc 22:10, đang revert,
   xong trong ~3 phút. Đây có thể là nguyên nhân."

  Em không cần xin lỗi trong channel, không cần giải thích gì thêm.
  Nếu ai hỏi thêm, chị trả lời.

  Còn chuyện vì sao hệ thống cho phép một biến như thế đi lên production
  10 giờ đêm mà không có ai review và không có alert — đó là việc của chị,
  và mình sẽ nói ở postmortem thứ Năm. Không phải lỗi của em.
```

Sáu chi tiết có tác dụng: (1) cảm ơn trước — đảo dấu của việc báo tin xấu; (2) tuyên bố miễn trừ rõ
ràng ngay lập tức, vì trạng thái tê liệt đang tiêu hết dung lượng nhận thức của Duy; (3) ba câu hỏi
đóng, về dữ kiện, không về ý định; (4) viết sẵn câu Duy phải gõ — dưới stress, việc soạn một câu đúng
mực trước 30 người là một tác vụ đắt; (5) "em không cần xin lỗi trong channel" — chặn phiên tự kiểm
điểm công khai, thứ luôn làm chậm incident; (6) chuyển trách nhiệm hệ thống về đúng chỗ của nó, kèm
một cam kết cụ thể có ngày.

Lưu ý ranh giới: điều này không có nghĩa Duy không học gì. Nó có nghĩa việc học xảy ra ở Learning Mode
với đủ thời gian, chứ không phải giữa lúc cháy — và nội dung học không phải "cẩn thận hơn" mà là những
điều cụ thể: cách đọc giới hạn connection của database, quy tắc thay đổi tham số hạ tầng, và cách công
bố thay đổi.

**D. Luân phiên khi incident kéo dài.** Quy tắc theo đồng hồ, không theo cảm nhận:

| Mốc thời gian | Hành động bắt buộc | Ai chịu trách nhiệm thực thi |
|---|---|---|
| 60 phút | Người đang thao tác nghỉ 5 phút, rời màn hình | IC |
| 2 giờ | Đổi IC nếu có người thay; ghi handover 5 dòng vào channel | IC hiện tại |
| 4 giờ | **Bắt buộc** đổi cả IC và Ops Lead; người cũ ở lại 15 phút để handover rồi rời hẳn | EM/lead cấp trên |
| 4 giờ và không có người thay | Hạ mục tiêu: chỉ giữ mitigation, dừng điều tra, đặt monitoring, hẹn tiếp tục sáng | EM |
| Sau incident đêm >2 giờ | Người tham gia nghỉ hết ngày hôm sau, tính là ngày làm việc, không trừ phép | EM, thông báo chủ động |

Nội dung handover 5 dòng: (1) ảnh hưởng hiện tại bằng số; (2) các giả thuyết đã loại và bằng chứng
loại; (3) giả thuyết đang xét; (4) mọi thay đổi đã thực hiện lên production, theo thứ tự; (5) việc
đang chạy dở và ai đang làm. Nếu không viết được 5 dòng này, đó là dấu hiệu incident đang không được
điều phối.

Về việc cho nghỉ sau incident đêm: nó phải là **quy tắc mặc định và được nói trước**, không phải sự
hào phóng được xin. Nếu phải xin, phần lớn engineer sẽ không xin — đặc biệt trong văn hoá coi việc có
mặt là biểu hiện của tinh thần trách nhiệm. Lời nói của EM nên ở dạng: *"Đêm qua em thức 4 giờ. Hôm
nay em không cần online, chị đã nói với PO và đổi lịch demo. Đây không phải phép, không cần bù."*

### Trade-off

| Cặp đối lập | Nghiêng về minh bạch khi | Nghiêng về bảo vệ team khi | Ai trả giá |
|---|---|---|---|
| **Minh bạch với stakeholder vs bảo vệ team khỏi nhiễu** | Stakeholder có quyết định thật cần ra (dùng cổng dự phòng, thông báo khách, kích hoạt quy trình compliance); khách B2B có SLA | Câu hỏi chỉ là để giảm lo lắng; người hỏi không có hành động nào phụ thuộc câu trả lời | Minh bạch quá: người sửa bị ngắt, MTTR tăng. Bảo vệ quá: stakeholder mất niềm tin và sẽ can thiệp mạnh hơn lần sau |
| **Lead ở trong channel liên tục vs rút ra để không tạo áp lực** | Team còn non, cần người quyết nhanh; incident nhiều bên | Team dày kinh nghiệm và có IC tốt; sự hiện diện của lead làm mọi người diễn thay vì làm | Ở quá nhiều: team mất tự chủ và chờ bạn quyết. Rút quá: team gánh cả phần chính trị |
| **Bảo vệ tuyệt đối người gây lỗi vs giữ tín hiệu về accountability** | Trong incident, và với người thiếu kinh nghiệm hoặc thiếu công cụ | Sau incident, và khi vấn đề là hành vi lặp lại có ý thức bỏ qua quy trình đã biết | Bảo vệ mù quáng: các senior thấy tiêu chuẩn không tồn tại. Bảo vệ thiếu: mất dòng thông tin |

Trade-off thứ ba là chỗ nhiều lead trẻ hiểu sai. Bảo vệ trong incident là **vô điều kiện**, vì nó là
tối ưu công cụ. Sau incident, nó có điều kiện — xem ranh giới blameless vs accountability ở chủ đề 3.

### Real-world Scenarios

**Tình huống 1 — Lead biến mất.** Một ODC ở Hà Nội, khách Singapore. Sự cố bắt đầu 20:15, Tech Lead
là Nam trả lời hai tin nhắn lúc 20:20 rồi offline vì "đã bàn giao cho on-call". On-call là Vy, 1 năm
kinh nghiệm. 20:40 Account Manager phía Việt Nam gọi thẳng cho Vy; 20:55 khách hàng gửi email cho cả
nhóm; 21:30 Vy vẫn đang một mình xử lý cả kỹ thuật và giao tiếp bằng tiếng Anh với khách.

Hệ quả không phải downtime — hệ quả là ba tuần sau Vy xin không tham gia on-call nữa, và trong 1-1 nói
một câu mà bất kỳ EM cũng nên coi là báo động: "Em không sợ hệ thống, em sợ lúc đó không có ai." Cách
xử lý đúng không đòi Nam phải trực tuyến mọi đêm: nó đòi một quy tắc rõ ràng rằng người on-call luôn
có một escalation contact có nghĩa vụ nhận cuộc gọi trong 10 phút, và rằng giao tiếp với khách nước
ngoài không bao giờ là việc của người đang sửa.

**Tình huống 2 — Lead micromanage từng lệnh.** Một ngân hàng số. Trong incident, Technical Director là
ông Hải, 18 năm kinh nghiệm, đọc từng lệnh mà Ops Lead gõ và yêu cầu xác nhận trước từng bước: "Sao
lại dùng cờ đó?", "Kiểm tra lại đi", "Cho tôi xem output trước khi enter". Ba hiệu ứng đo được: thời
gian mỗi hành động tăng khoảng gấp ba; Ops Lead ngừng đề xuất và chỉ chờ được chỉ; và sau ba incident
như vậy, không ai trong team muốn làm Ops Lead khi ông Hải có mặt.

Phần khó của tình huống này là ông Hải không sai về kỹ thuật — ông thường phát hiện được sai sót thật.
Trade-off thật: một sai sót được chặn với giá là năng lực của bảy người không phát triển và tốc độ xử
lý giảm ba lần. Cách chuyển hoá đúng: đặt ông Hải vào vai reviewer của **quyết định**, không phải của
**lệnh**. Nghĩa là ông xem hướng xử lý và điều kiện dừng, không xem từng dòng bash. Ở bối cảnh Việt
Nam, việc một Tech Lead trẻ đề nghị điều này với một người 18 năm kinh nghiệm là một cuộc trò chuyện
khó và nên diễn ra ngoài incident, ở dạng: *"Em muốn nhờ anh một việc có giá trị hơn: anh giữ vai
soát hướng xử lý và điều kiện rollback. Nếu anh soát từng lệnh thì em mất anh ở tầng quyết định, mà
tầng đó không ai thay được anh."*

**Tình huống 3 — Truy tìm người ngay lúc đang cháy.** E-commerce, giữa campaign. 20:22, CTO gõ trong
war room: "Ai deploy lúc 19:50? Ai approve cái PR đó?" Trong 6 phút sau đó, channel có 4 tin nhắn về
quy trình phê duyệt và 0 tin nhắn về việc chặn ảnh hưởng. Người deploy — một senior tên Quân — bắt đầu
viết một đoạn dài giải thích rằng thay đổi đã qua review. Đó là 6 phút và một senior bị lấy ra khỏi
việc sửa.

Cách IC nên can thiệp, và điều này cần dũng cảm: *"Anh ơi, câu đó quan trọng và em sẽ trả lời đầy đủ
ở postmortem. Bây giờ em cần Quân tập trung vì anh ấy là người biết luồng này nhất. Em xin phép giữ
channel cho việc chặn ảnh hưởng."* Nếu IC không nói được câu này, hãy để EM nói. Nếu cả hai không nói
được, đó là một phát hiện về tổ chức chứ không phải về incident.

### Best Practices

- **Nói ra rằng bây giờ không phải lúc tìm nguyên nhân về người, và cam kết mốc sẽ tìm.** Hai nửa của
  câu này phải đi cùng. Chỉ nửa đầu là trốn tránh; chỉ nửa sau là đe doạ.
- **Thay câu hỏi mở bằng câu hỏi đóng khi người đối diện đang tải nặng.** "Em thấy thế nào?" là câu
  hỏi tốn kém. "Connection pool có cạn không, có hay không?" là câu hỏi rẻ. Sau khi hạ tải, quay lại
  câu hỏi mở.
- **Viết sẵn câu cho người đang tê liệt.** Kỹ thuật nhỏ, tác dụng lớn: soạn giúp họ tin nhắn cần gõ.
  Bạn đang gỡ một tác vụ nhận thức, không phải làm hộ.
- **Đặt hẹn giờ cho việc nghỉ và việc luân phiên.** Đừng dựa vào việc bạn sẽ nhớ; bạn cũng đang bị
  suy giảm.
- **Cho nghỉ sau incident đêm như quy tắc, thông báo chủ động, không để phải xin.** Và nói rõ trong
  channel công khai, để người khác thấy đó là chuẩn mực chứ không phải ngoại lệ.
- **Sau incident, gửi một tin nhắn riêng cho từng người tham gia trong vòng 24 giờ.** Nội dung: cụ
  thể họ đã làm gì có tác dụng, và một câu hỏi về việc họ đang ổn không. Điều này có hiệu lực cao hơn
  một lời cảm ơn chung trong channel, vì lời cảm ơn chung không phân biệt được ai đã làm gì.
- **Với người vừa gây ra lỗi, cho họ tham gia viết action item.** Đây là cách chuyển họ từ vị trí đối
  tượng sang vị trí tác nhân, và nó là biện pháp phục hồi hiệu quả nhất. Người sửa chính lỗ hổng đã
  bẫy mình thường trở thành người bảo vệ quy trình đó mạnh nhất.

### Anti-patterns

- **Truy tìm ai deploy ngay trong lúc đang cháy.** Cơ chế: biến thông tin về thay đổi thành rủi ro cá
  nhân, làm chậm đúng thông tin đắt nhất; đồng thời rút người biết hệ thống nhất ra khỏi việc sửa. Dấu
  hiệu sớm: từ "ai" xuất hiện trong channel trước khi ảnh hưởng được chặn.
- **Lead biến mất.** Cơ chế: người on-call phải gánh cả phần kỹ thuật và phần chính trị; phần chính
  trị là phần họ không có thẩm quyền để làm, nên nó biến thành lo lắng thuần. Dấu hiệu sớm: on-call
  phải tự trả lời stakeholder; hoặc trong 1-1 xuất hiện câu "lúc đó em không biết gọi ai".
- **Lead micromanage từng lệnh.** Cơ chế: chèn một bước duyệt tuần tự vào con đường tới hạn, và tước
  quyền phán đoán của người thao tác, nên người đó ngừng suy nghĩ và chỉ chờ chỉ dẫn. Dấu hiệu sớm:
  Ops Lead dán lệnh vào channel và chờ trước khi enter mà không ai yêu cầu.
- **Tăng áp lực bằng thông tin về thiệt hại.** "Mỗi phút mất 30 triệu" không đổi hành động của người
  đang sửa; nó chỉ chiếm dung lượng nhận thức. Dấu hiệu sớm: các con số thiệt hại được nhắc lại nhiều
  hơn hai lần trong channel kỹ thuật.
- **Cảm xúc của lead trở thành một thứ team phải quản lý.** Khi lead lo lắng ra mặt, team dành một
  phần năng lượng để trấn an lead. Dấu hiệu sớm: có người nhắn riêng cho bạn "anh yên tâm ạ" trong lúc
  họ nên đang sửa.
- **"Ngày mai họp rút kinh nghiệm" nói bằng giọng cảnh báo.** Câu này, nói sai giọng, biến postmortem
  thành phiên xử trước cả khi nó diễn ra, và mọi người sẽ đến với bản tường thuật đã chuẩn bị để tự
  bảo vệ. Dấu hiệu sớm: trước postmortem, có người xin gặp riêng bạn để "giải thích trước".

### Khi nào KHÔNG nên áp dụng

- **Khi vấn đề thật là hành vi lặp lại, không phải một lỗi trong bối cảnh xấu.** Nếu cùng một người,
  lần thứ ba, thay đổi production ngoài quy trình đã được thống nhất và đã được nhắc riêng hai lần,
  thì cách xử lý không còn là bảo vệ trong incident cộng với action item hệ thống. Nó là một cuộc trò
  chuyện về performance, riêng, có ghi nhận, theo cơ chế ở chương 03. Nhầm lẫn hai loại này làm hỏng
  cả hai: người vi phạm không nhận tín hiệu, và những người tuân thủ thấy tiêu chuẩn là hư cấu.
- **Khi việc "giảm tải nhận thức" trở thành làm hộ.** Với một senior đang xử lý tốt, việc bạn liên tục
  hỏi "cần gì không", viết sẵn câu, kiểm tra trạng thái là một nguồn ngắt mới. Ranh giới: can thiệp
  theo yêu cầu hoặc theo tín hiệu quan sát được (người đó lặp lại một hành động vô hiệu, hoặc im lặng
  quá 10 phút), không theo mặc định.
- **Khi bạn không phải người có thẩm quyền và việc bạn chen vào tạo hai đường chỉ huy.** Nếu bạn là
  Tech Lead của team khác vào giúp, hành vi đúng là hỏi IC "cần gì" và tuân theo, kể cả khi bạn thấy
  hướng đi chưa tối ưu. Hai người điều phối tệ hơn một người điều phối kém.
- **Khi tổ chức đang trong khủng hoảng tồn tại thật.** Trong tình huống hệ thống mất dữ liệu khách
  hàng hoặc rủi ro pháp lý trực tiếp, có những cuộc trò chuyện không thể hoãn tới postmortem — ví dụ
  xác định phạm vi dữ liệu bị ảnh hưởng đòi hỏi hỏi rất kỹ người đã thao tác, ngay lúc đó. Cách giữ
  cả hai: nói rõ mục đích của việc hỏi. *"Anh hỏi rất kỹ không phải để quy trách nhiệm, mà vì mình
  phải biết chính xác bao nhiêu bản ghi bị ảnh hưởng trong 30 phút nữa để báo cơ quan quản lý."*
- **Với đội vận hành đã chuyên nghiệp hoá và có SOP thành thục.** Nhiều thực hành trong mục này là để
  bù cho việc team chưa có phản xạ. Với một đội NOC vận hành 24/7 đã qua hàng trăm incident, việc lead
  vào channel giải thích quy tắc và trấn an là nhiễu. Ở đó, vai của lead thu về hai việc: quyết những
  điều vượt thẩm quyền của họ, và chặn nhiễu từ bên ngoài.

---

## 3. Postmortem không quy tội (Blameless Postmortem)

### Problem Statement

Một cách kiểm tra chất lượng postmortem của tổ chức bạn mà không cần đọc nội dung: mở 10 postmortem
gần nhất, đếm số action item, rồi đếm số action item đã đóng. Ở nhiều tổ chức, con số minh hoạ là 47
action item và 9 đã đóng — tỉ lệ 19%. Rồi làm bước thứ hai, tàn nhẫn hơn: đọc 47 item đó và đếm bao
nhiêu cái là **thay đổi hệ thống** so với bao nhiêu cái là **thay đổi kỳ vọng về hành vi con người**.
Nếu bảng của bạn giống phần lớn tổ chức, bạn sẽ thấy khoảng một nửa là dạng "review kỹ hơn", "cẩn thận
khi deploy giờ cao điểm", "đào tạo lại team về quy trình". Những item đó không có cách nào để đóng, vì
chúng không có trạng thái đóng.

Bốn hiện tượng khác, cụ thể hơn:

**Hiện tượng một: cùng một loại incident lặp lại và mỗi lần đều có postmortem.** Đây là hiện tượng
nghịch lý nhất của chủ đề này: sự tồn tại của postmortem không chứng minh sự tồn tại của việc học.
Nếu ba postmortem trong sáu tháng đều có nguyên nhân gốc thuộc nhóm "thay đổi cấu hình không qua kiểm
soát", thì tổ chức đã ba lần viết đúng nguyên nhân và ba lần không thay đổi hệ thống.

**Hiện tượng hai: nội dung postmortem khác nội dung mọi người nói với nhau ở quán cà phê.** Nếu bản
viết nói "do thiếu monitoring ở tầng cache" mà ba engineer trong bữa trưa nói "thật ra vì bị ép ship
trước hạn campaign nên bỏ bước load test", thì postmortem của bạn là một tài liệu hư cấu và nó không
thể sinh ra action item đúng.

**Hiện tượng ba: cuộc họp postmortem có một người nói 70% thời gian và người đó đang tự giải thích.**
Đây là dấu hiệu buổi họp đã chuyển thành phiên xử, dù không ai chủ ý.

**Hiện tượng bốn: sau postmortem, thứ được nhớ nhất là ai đã làm gì sai.** Kiểm tra bằng cách hỏi ba
người tham dự sau hai tuần: "postmortem lần đó kết luận gì?". Nếu câu trả lời là một cái tên, buổi họp
đã thất bại về mục tiêu.

Hậu quả dài hạn: chất lượng thông tin đầu vào giảm dần. Người ta vẫn dự postmortem, vẫn nói, nhưng chỉ
nói những thứ an toàn. Tổ chức tiếp tục sản xuất tài liệu và tin rằng mình đang học, trong khi các
nguyên nhân thật — áp lực deadline, thiếu môi trường test, một quyết định kiến trúc bị hoãn 4 tháng
trước — không bao giờ được ghi vào bất kỳ đâu.

### First Principles

**Cơ chế một: mục tiêu của postmortem là thay đổi hệ thống, và người không phải là thành phần sửa được
của hệ thống.** Con người sẽ tiếp tục mắc lỗi vì đó là thuộc tính của con người, không phải khuyết
điểm cá biệt. Nếu hệ thống chỉ hoạt động đúng khi mọi người luôn cẩn thận, thì hệ thống đó đã sai
thiết kế, và cách sửa duy nhất bền là làm cho lựa chọn đúng dễ hơn lựa chọn sai. Suy ra một phép thử:
với mỗi kết luận postmortem, hỏi "nếu thay người đó bằng một người khác cùng trình độ, cùng thông tin,
cùng áp lực thời gian, kết quả có khác không?". Nếu câu trả lời là không, nguyên nhân không nằm ở
người.

**Cơ chế hai: chất lượng thông tin phụ thuộc vào hậu quả của việc báo cáo trung thực.** Đây là một
bài toán incentive thuần, thuộc họ Principal-Agent Problem. Nếu việc nói thật làm tăng xác suất bị
đánh giá xấu, bị nhắc trong performance review, hoặc mất mặt trước đông người, thì người hợp lý sẽ nói
ít hơn hoặc nói theo cách khó kiểm chứng. Không có cách nào dùng lời kêu gọi để bù cho một incentive
ngược. Đây là lý do blameless không phải một giá trị đạo đức mà là **một điều kiện kỹ thuật để thu
được dữ liệu đúng**. Ngành hàng không dân dụng đã công khai vận hành nguyên lý này hàng chục năm qua
các hệ thống báo cáo sự cố tự nguyện được miễn trừ: họ chấp nhận không trừng phạt để đổi lấy dòng
thông tin.

**Cơ chế ba: hindsight bias làm mọi lỗi trông hiển nhiên sau khi đã biết kết quả.** Sau khi biết đổi
`DB_POOL_MAX` từ 20 lên 120 gây sập, câu "sao lại không nghĩ tới connection limit của database" nghe
hợp lý tới mức khó cưỡng. Nhưng ở thời điểm 22:10, Duy đứng trước một ticket tối ưu hiệu năng, một
staging chạy tốt, không có tài liệu nào nói về giới hạn, và không có alert nào. Kỹ thuật chống lại bias
này gọi là **luận cứ tại thời điểm** (local rationality): với mỗi hành động trong timeline, hỏi "vì
sao ở thời điểm đó, với thông tin đó, hành động này có vẻ hợp lý?". Câu hỏi này gần như luôn cho ra một
phát hiện về hệ thống — thông tin nào đã thiếu, tín hiệu nào đã không có, tài liệu nào không tồn tại.

**Cơ chế bốn: sự cố trong hệ thống phức tạp gần như không bao giờ có một nguyên nhân duy nhất.** Đây
là quan sát trung tâm của các tài liệu công khai về an toàn hệ thống phức tạp: sự cố xảy ra khi nhiều
lớp phòng thủ vốn đủ để chặn nó lại cùng bị vô hiệu. Trong ví dụ trên có ít nhất năm lớp thất bại: (a)
không có review cho thay đổi tham số hạ tầng; (b) staging không phản ánh tải thật; (c) không có tài
liệu về giới hạn connection của database; (d) không có alert khi số connection tới ngưỡng; (e) không
có quy tắc về cửa sổ thay đổi. Chọn một trong năm và gọi nó là "the root cause" là một lựa chọn tuỳ ý,
và thường người ta chọn cái gần con người nhất.

### Blameless KHÔNG phải là không có accountability

Đây là mục cần viết rõ, vì đây là chỗ khái niệm bị lạm dụng theo cả hai hướng: một số tổ chức dùng
"blameless" để không ai chịu trách nhiệm gì; một số khác dùng lập luận "phải có accountability" để
biến postmortem thành phiên xử.

Ranh giới nằm ở việc phân biệt ba thứ hay bị gộp:

| Khái niệm | Nội dung | Ai giữ | Nơi xử lý |
|---|---|---|---|
| **Blame** | Gán giá trị đạo đức cho một hành động cá nhân: "cẩu thả", "không có trách nhiệm" | Không ai nên giữ | Không có |
| **Accountability** | Nghĩa vụ sở hữu việc sửa: ai chịu trách nhiệm để action item được đóng, ai sở hữu kết quả của hệ thống | Lead / owner của hệ thống, và người sở hữu từng action item | Postmortem và cơ chế theo dõi sau đó |
| **Performance** | Mẫu hành vi lặp lại của một cá nhân, xét trên nhiều sự việc | Manager của người đó | 1-1 riêng, performance review — **không** trong postmortem |

Ba phát biểu làm rõ ranh giới:

- **Blameless nói về nguyên nhân, accountability nói về việc sửa.** Postmortem không hỏi "ai đã gây ra
  chuyện này" nhưng bắt buộc hỏi "ai sở hữu việc làm cho chuyện này không xảy ra lần nữa, và hạn nào".
  Một postmortem không có tên người và ngày ở cột owner là một postmortem không có accountability.
- **Blameless không có nghĩa mọi hành vi đều được chấp nhận.** Vi phạm có ý thức, lặp lại, một quy tắc
  đã được thống nhất và đã được nhắc riêng, là vấn đề performance. Nó được xử lý ở kênh khác, riêng,
  và người bị xử lý phải biết rõ mình đang ở kênh nào. Điều làm hỏng hệ thống không phải việc tồn tại
  hai kênh, mà việc trộn chúng: khi người ta không phân biệt được postmortem với đánh giá, họ sẽ mặc
  định coi mọi postmortem là đánh giá.
- **Người sở hữu accountability cao nhất cho một incident thường là lead, không phải người thao tác
  cuối.** Nếu một junior đổi được một tham số hạ tầng ở production lúc 22 giờ mà không ai review và
  không có alert, câu hỏi accountability đúng là: ai đã quyết định để hệ thống ở trạng thái đó, và
  quyết định đó có được ghi nhận không. Câu trả lời hầu như luôn nằm ở tầng lead — và việc lead nói ra
  điều này trong postmortem là hành động thiết lập chuẩn mực mạnh nhất mà bạn có.

### Mental Model

**Mô hình một: Swiss Cheese Model.** Hình dung mỗi lớp phòng thủ là một miếng phô mai có lỗ; sự cố
xuyên qua khi các lỗ tình cờ thẳng hàng. Hệ quả cho postmortem: nhiệm vụ không phải tìm miếng nào
"có lỗi", mà **liệt kê tất cả các lớp đã bị xuyên qua và chọn lớp nào rẻ nhất để bịt**. Điều này làm
đổi câu hỏi kết luận từ "nguyên nhân gốc là gì" thành "trong năm lớp đã thất bại, bịt lớp nào cho hiệu
quả trên chi phí cao nhất, và lớp nào ta cố tình chấp nhận để hở?".

**Mô hình hai: hai câu chuyện của một incident — the counterfactual và the actual.** Con người có xu
hướng kể lại incident theo dạng "nếu Duy đã kiểm tra thì đã không sao" — đó là counterfactual, một câu
chuyện về một thế giới không xảy ra, và nó không sinh ra hành động nào. Câu chuyện dùng được là câu
chuyện về thế giới đã xảy ra: hệ thống đã ở trạng thái nào để hành động đó được coi là hợp lý và được
phép. Nguyên tắc viết: **loại mọi câu chứa "đã nên", "lẽ ra", "chỉ cần"** khỏi phần phân tích. Chúng
là dấu hiệu ngôn ngữ của hindsight bias.

**Mô hình ba: postmortem như một vòng feedback có độ lợi.** Vòng lặp là: incident → hiểu → thay đổi hệ
thống → giảm xác suất/ảnh hưởng lần sau. Độ lợi của vòng này bằng tỉ lệ action item được đóng nhân với
mức độ action item đó thật sự thay đổi hệ thống. Nếu bạn đóng 19% action item và một nửa số đó là "đào
tạo lại", độ lợi thực của vòng feedback gần bằng không — và một vòng feedback độ lợi bằng không thì
tương đương không có vòng feedback, chỉ đắt hơn vì bạn vẫn trả chi phí họp.

### Practical Framework

**Bước 1 — Thu timeline từ dữ liệu TRƯỚC khi hỏi người (2–4 giờ, làm trong 24 giờ đầu).** Thứ tự này
quan trọng và hay bị làm ngược. Lý do: ký ức con người bị tái cấu trúc theo kết quả đã biết; nếu bạn
hỏi người trước, bạn sẽ nhận về một câu chuyện đã được sắp xếp cho hợp lý. Nguồn dữ liệu theo thứ tự
tin cậy: audit log của thay đổi (deploy, config, feature flag, migration), log hệ thống, biểu đồ
metric, channel incident, ticket. Xuất ra một bảng thô, cột: thời gian — sự kiện — nguồn dữ liệu — ai
liên quan. Chưa diễn giải.

**Bước 2 — Phỏng vấn từng người riêng, 20–30 phút mỗi người, trước cuộc họp chung.** Mục tiêu duy nhất:
thu **luận cứ tại thời điểm**. Bốn câu hỏi, dùng đúng thứ tự này:

1. "Ở thời điểm đó, anh/em nghĩ hệ thống đang ở trạng thái nào?"
2. "Anh/em dựa vào tín hiệu nào để nghĩ vậy?"
3. "Điều gì làm anh/em tin rằng hành động đó là an toàn?"
4. "Nếu có một thông tin nào xuất hiện lúc đó thì anh/em đã làm khác — thông tin gì?"

Câu 4 là câu sinh ra action item tốt nhất, vì nó chỉ thẳng vào tín hiệu còn thiếu trong hệ thống. Cấm
tuyệt đối bốn dạng câu: "sao lại...", "tại sao không...", "anh/em có biết là...", "lẽ ra phải...".

**Bước 3 — Cuộc họp 60–90 phút với cấu trúc cố định và một facilitator không phải người trong sự cố.**

| Phút | Nội dung | Quy tắc |
|---|---|---|
| 0–5 | Facilitator đọc to nguyên tắc: mục tiêu là hệ thống; không dùng tên kèm phán xét; không dùng "lẽ ra" | Đọc to mỗi lần, kể cả lần thứ 20. Đây là nghi thức có chức năng: nó cho phép người trong phòng viện dẫn nó khi bị vi phạm |
| 5–20 | Đi qua timeline dữ liệu, không diễn giải. Bổ sung, sửa mốc | Ai cũng được sửa timeline. Không tranh luận nguyên nhân ở pha này |
| 20–30 | Xác định ảnh hưởng bằng số: người dùng, giao dịch, tiền, SLA, error budget đã tiêu | Không có số thì ghi "chưa đo được" và tạo action item về việc đo |
| 30–55 | Phân tích các lớp đã thất bại (contributing factors), theo mô hình Swiss Cheese | Yêu cầu tối thiểu 4 factor. Nếu chỉ tìm được 1, buổi họp chưa xong |
| 55–75 | Sinh action item, gán owner và ngày ngay trong phòng | Không có owner mặt trong phòng thì không tạo item đó; ghi vào phần "cần đàm phán" |
| 75–85 | Điểm tốt: cái gì đã hoạt động đúng (phát hiện nhanh, rollback được, ai đó nói ra sớm) | Bắt buộc có mục này. Nó cân bằng tín hiệu và củng cố hành vi cần nhân bản |
| 85–90 | Chốt ai viết bản cuối, hạn công bố, ai đọc | |

**Bước 4 — 5 Whys và giới hạn của nó.** 5 Whys hữu ích như một công cụ **đào sâu một nhánh**, và nó
có ba giới hạn cần biết:

- **Nó tuyến tính.** Cấu trúc "vì sao → vì sao" tạo ra một chuỗi duy nhất, trong khi sự cố có cây
  nhiều nhánh. Chuỗi bạn thu được phụ thuộc vào nhánh người dẫn chọn ở bước một.
- **Nó dễ trôi về phía con người.** Chuỗi "vì sao lỗi → vì sao config sai → vì sao không kiểm tra →
  vì sao không cẩn thận" là kết cục mặc định nếu không có quy tắc chặn. Quy tắc chặn hữu dụng: **khi
  câu trả lời cho một "vì sao" là một trạng thái tinh thần của con người (quên, không biết, chủ quan),
  hỏi tiếp một lần nữa nhưng về hệ thống**: "vì sao hệ thống cho phép việc không biết đó dẫn tới sự cố
  mà không có lớp nào chặn?".
- **Nó tạo cảm giác hoàn thành sai.** Khi đến "why" thứ năm và có một câu nghe sâu sắc, người ta dừng.

Cách dùng đúng: dùng 5 Whys để đào **mỗi contributing factor** đã liệt kê, không dùng nó để tìm một
nguyên nhân duy nhất. Rồi chuyển sang bảng contributing factors, đây là dạng đầu ra dùng được:

| Contributing factor | Lớp phòng thủ nào đã hở | Sửa được không | Chi phí sửa (minh hoạ) | Quyết định |
|---|---|---|---|---|
| Thay đổi tham số hạ tầng không cần review | Kiểm soát thay đổi | Có, thêm required review cho thư mục config hạ tầng | 0,5 ngày | Làm — hạn 01/08, owner @linh |
| Staging không phản ánh tải thật | Môi trường test | Có nhưng đắt: cần bộ sinh tải và dữ liệu giống production | 15–20 ngày | Chấp nhận hở, ghi nhận rủi ro; thay bằng canary 5% traffic (3 ngày) |
| Không có alert khi connection pool tới 80% giới hạn | Phát hiện | Có | 1 ngày | Làm — hạn 30/07, owner @tuan |
| Không có tài liệu về giới hạn connection của database | Kiến thức vận hành | Có, thêm vào runbook order-svc | 0,5 ngày | Làm — hạn 05/08, owner @duy |
| Không có cửa sổ thay đổi (change window) | Quy trình | Có, nhưng đánh đổi tốc độ | 2 ngày | Chưa làm — đưa vào bàn ở review kiến trúc quý |

Chú ý dòng thứ hai và dòng cuối: **một postmortem tốt có những lỗ được cố tình để hở, có ghi nhận.**
Đó là dấu hiệu của một tổ chức ra quyết định chứ không phải một tổ chức viết cam kết.

**Bước 5 — Tiêu chí action item hợp lệ.** Bốn điều kiện, thiếu một là không hợp lệ:

| Tiêu chí | Ví dụ KHÔNG hợp lệ | Ví dụ hợp lệ |
|---|---|---|
| Có owner là **một** người có tên | "Team BE" | "@tuan" |
| Có hạn ngày cụ thể | "Sớm nhất có thể", "Q3" | "30/07/2026" |
| Thay đổi **hệ thống**, không phải kỳ vọng hành vi | "Cẩn thận hơn khi sửa config production" | "Thêm required review cho `infra/config/**` trong CODEOWNERS" |
| Có tiêu chí xác minh đã xong | "Cải thiện monitoring" | "Alert bắn khi `db_connections/db_max_connections > 0,8` trong 2 phút; đã test bằng cách bơm tải trên staging" |

Phép thử nhanh cho tiêu chí thứ ba: hỏi "nếu người này nghỉ việc, action item này còn tác dụng không?".
"Cẩn thận hơn" nghỉ việc theo người. Một required review thì không.

**Bước 6 — Theo dõi đến khi đóng.** Đây là bước duy nhất tạo ra kết quả, và là bước bị bỏ nhiều nhất.
Cơ chế tối thiểu: mọi action item thành ticket trong cùng backlog với công việc thường, có nhãn
`incident-followup` và trường liên kết tới ID incident; item mức P0/P1 được đưa vào sprint tiếp theo
như cam kết cứng, không phải "làm nếu có thời gian"; một bảng duy nhất hiển thị tất cả item đang mở
theo tuổi; và trong sprint review hàng tháng, lead đọc to danh sách item quá hạn. Chỉ số cần theo:
**tỉ lệ action item P0/P1 đóng trong hạn** và **tuổi trung vị của item đang mở**. Nếu tuổi trung vị
tăng qua các quý, vòng feedback đang chết dần dù bạn vẫn viết postmortem đều.

**TEMPLATE — Postmortem đầy đủ (ví dụ đã điền, số minh hoạ).**

```markdown
# Postmortem — INC-2026-0143: Lỗi 40% giao dịch thẻ (SEV2)

Trạng thái tài liệu: FINAL  |  Ngày: 30/07/2026
Chủ trì: @minh (facilitator, không tham gia xử lý sự cố)
Tham dự: @tuan (Ops Lead), @duy, @linh (Tech Lead), @trang (Comms), @vy (on-call)
Bản này là BẢN NỘI BỘ. Bản đối ngoại: xem INC-2026-0143-external.

## 1. Tóm tắt (5 dòng, viết cho người không đọc phần còn lại)
Từ 02:31 đến 03:12 ngày 28/07 (41 phút), khoảng 40% giao dịch thanh toán thẻ
thất bại do order-svc chiếm hết connection của database dùng chung sau khi tham số
DB_POOL_MAX được tăng từ 20 lên 120 lúc 22:10 ngày 27/07. Sự cố được giảm thiểu
bằng cách hoàn lại tham số. Không có giao dịch treo tiền; đã đối soát khớp.
Ảnh hưởng: ~1.240 giao dịch lỗi, ~62 triệu VND giá trị giao dịch không hoàn tất
(số minh hoạ). Không vi phạm ngưỡng SLA hợp đồng với đối tác.

## 2. Ảnh hưởng (bằng số, không dùng tính từ)
| Chiều | Số đo | Nguồn |
|---|---|---|
| Thời gian ảnh hưởng | 41 phút (02:31–03:12) | metric error rate |
| Tỉ lệ giao dịch thẻ lỗi | đỉnh 43%, bình thường 1,5% | dashboard payment |
| Số giao dịch lỗi | ~1.240 | truy vấn bảng transaction |
| Giao dịch treo tiền | 0 (đối soát 02:00–04:00) | báo cáo đối soát |
| Luồng KHÔNG ảnh hưởng | ví nội bộ, QR, chuyển tiền nội bộ | metric theo luồng |
| Error budget tháng đã tiêu | 41 / 132 phút = 31% | tính theo SLO 99,7% |
| Cuộc gọi tới tổng đài | 180 | báo cáo Support |
| Nghĩa vụ báo cáo bên ngoài | Không (dưới ngưỡng) | Compliance xác nhận 28/07 |

## 3. Timeline (dựng từ dữ liệu trước, bổ sung bằng phỏng vấn sau)
| Thời gian | Sự kiện | Nguồn |
|---|---|---|
| 27/07 22:10 | DB_POOL_MAX của order-svc: 20 -> 120, deploy qua pipeline thường | audit log |
| 27/07 22:10–28/07 02:30 | Tải thấp, số connection dùng thực tế đỉnh 34 | metric |
| 28/07 02:28 | Job đối soát đêm khởi động, order-svc tăng đồng thời | scheduler log |
| 28/07 02:31 | Số connection tới database đạt giới hạn 200; payment-svc bắt đầu bị từ chối kết nối | metric |
| 28/07 02:39 | Alert error rate payment bắn | alert log |
| 28/07 02:40 | @vy xác nhận lỗi thật bằng luồng người dùng trên điện thoại | channel |
| 28/07 02:41 | Tuyên bố SEV2, mở #inc-0143, gọi điện IC | channel |
| 28/07 02:44 | Phân vai: IC @minh, Ops @tuan, Comms @trang, Scribe @vy | channel |
| 28/07 02:46 | @tuan xác nhận không có deploy nào của payment-svc trong 6 giờ | channel |
| 28/07 02:52 | Founder vào channel; @trang cập nhật, IC không bị ngắt | channel |
| 28/07 02:58 | @duy nhắn riêng @linh: "em nghĩ do em đổi pool" | tin nhắn riêng |
| 28/07 03:00 | @duy công bố trong channel giá trị đã đổi | channel |
| 28/07 03:04 | Hoàn lại DB_POOL_MAX = 20, deploy | audit log |
| 28/07 03:12 | Error rate về 1,6% | metric |
| 28/07 03:20 | Tuyên bố đóng incident, theo dõi 24h | channel |
| 28/07 09:00 | Đối soát xác nhận không có giao dịch treo | báo cáo |

## 4. Luận cứ tại thời điểm (vì sao mỗi hành động là hợp lý lúc đó)
- @duy tăng pool theo ticket PERF-88 (giảm thời gian chờ connection ở giờ cao điểm).
  Staging chạy tốt. Không có tài liệu nào nêu giới hạn 200 connection của cluster
  dùng chung. Không có alert nào về mức sử dụng connection. Trong CODEOWNERS,
  thư mục config không yêu cầu review. -> Ở thời điểm đó, đây là một thay đổi
  bình thường, không có tín hiệu nào cho thấy nó rủi ro.
- @duy chờ 27 phút mới nói: trong phỏng vấn, @duy nói "em không chắc, em sợ nói sai
  làm mọi người mất thời gian". -> Đây là tín hiệu về hệ thống, xem factor #5.
- @tuan tìm ở payment-svc trước vì lỗi hiện ra ở payment-svc. Không có công cụ nào
  cho phép xem tất cả thay đổi trên mọi service trong một khung thời gian.

## 5. Các yếu tố góp phần (contributing factors) — KHÔNG có "một nguyên nhân gốc"
| # | Yếu tố | Lớp phòng thủ hở | Quyết định |
|---|---|---|---|
| 1 | Thay đổi tham số hạ tầng không cần review | Kiểm soát thay đổi | Sửa — AI-1 |
| 2 | Không có alert cho mức sử dụng connection của cluster | Phát hiện | Sửa — AI-2 |
| 3 | Không có tài liệu về giới hạn tài nguyên dùng chung | Kiến thức vận hành | Sửa — AI-3 |
| 4 | Staging không phản ánh tải và không có canary | Kiểm thử | Chấp nhận hở phần staging; làm canary — AI-4 |
| 5 | Không có kênh chuẩn để nói "tôi nghĩ có thể do tôi" | Dòng thông tin | Sửa — AI-5 |
| 6 | Database dùng chung giữa order-svc và payment-svc, không có giới hạn theo service | Kiến trúc / cách ly lỗi | Đưa vào review kiến trúc quý — AI-6 |

## 6. Cái gì đã hoạt động đúng (bắt buộc có mục này)
- @vy, mới 4 tháng, tuyên bố SEV2 đúng và gọi điện thay vì chỉ nhắn Slack.
- Phân vai trong 3 phút; founder được xử lý qua Comms, IC không bị ngắt lần nào.
- Rollback config thực hiện được trong 6 phút.
- Đối soát tự động cho câu trả lời về tiền của khách trong cùng ca.

## 7. Action items
| ID | Nội dung (thay đổi hệ thống) | Owner | Hạn | Mức | Xác minh đã xong bằng gì | Trạng thái |
|---|---|---|---|---|---|---|
| AI-1 | Thêm `infra/config/**` vào CODEOWNERS, bắt buộc 1 review từ nhóm platform | @linh | 01/08 | P1 | PR thử bị chặn khi không có review | OPEN |
| AI-2 | Alert khi `db_connections/db_max > 0,8` trong 2 phút, cho mọi cluster dùng chung | @tuan | 30/07 | P0 | Bơm tải trên staging, alert bắn trong 2 phút | OPEN |
| AI-3 | Thêm mục "giới hạn tài nguyên dùng chung" vào runbook order-svc và payment-svc | @duy | 05/08 | P2 | Runbook có bảng giới hạn, @vy đọc và xác nhận hiểu | OPEN |
| AI-4 | Canary 5% traffic cho mọi thay đổi config của order-svc | @tuan | 15/08 | P1 | Một thay đổi config đi qua canary và tự dừng khi error tăng | OPEN |
| AI-5 | Thêm vào runbook incident dòng "cách báo nghi ngờ thay đổi của mình", kèm câu mẫu | @minh | 31/07 | P1 | Có trong runbook; nhắc trong 2 buổi stand-up | OPEN |
| AI-6 | Đề xuất tách connection pool theo service hoặc tách database | @linh | 20/08 (bản RFC) | P2 | RFC có trong review kiến trúc quý | OPEN |

## 8. Điều chúng ta cố tình KHÔNG làm
- Không dựng staging phản ánh tải production: ước tính 15–20 ngày công, không tương
  xứng với rủi ro còn lại sau AI-2 và AI-4. Xem lại nếu có thêm 2 incident cùng loại.
- Không thêm cửa sổ thay đổi (change window) cấm deploy sau 21:00: đánh đổi tốc độ
  quá lớn với một tổ chức đang cần lead time thấp. Xem lại nếu AI-1 và AI-4 không đủ.

## 9. Bài học ở tầng tổ chức (dành cho incident review hàng tháng)
Đây là incident thứ 3 trong 5 tháng có yếu tố "thay đổi cấu hình không qua kiểm soát"
(INC-0098, INC-0121, INC-0143). Ba postmortem trước đều tạo action item cục bộ cho
service liên quan. Đề nghị xử lý ở tầng tổ chức: chuẩn kiểm soát cấu hình cho mọi
service, không phải cho từng service. Chuyển @linh chủ trì, báo cáo ở review quý.
```

**Xử lý bối cảnh Việt Nam: khi văn hoá "giữ mặt" và cơ chế báo cáo lên cấp trên vô hiệu hoá blameless.**

Đây là phần nếu bỏ thì mọi thứ ở trên là lý thuyết nhập khẩu. Ba lực cụ thể và cách xử lý từng lực:

**Lực một: mất mặt trước đông người có giá tâm lý cao, đặc biệt với người ít thâm niên trước người
nhiều thâm niên.** Hệ quả: trong cuộc họp postmortem đông người, người liên quan nhất sẽ nói ít nhất,
hoặc nói theo cách chung chung. Cách xử lý có tác dụng, theo thứ tự hiệu quả:

- **Chuyển phần lớn việc thu thông tin sang phỏng vấn 1-1 trước họp.** Đây là điều chỉnh quan trọng
  nhất và nó không làm mất tính blameless — nó chỉ thay đổi kênh.
- **Facilitator trình bày timeline, không phải người liên quan.** Ở nhiều tài liệu phương Tây, người
  gây ra sự cố tự kể lại như một hành động tự chủ và được ca ngợi. Ở bối cảnh nhiều tổ chức Việt Nam,
  việc đó tương đương một phiên tự kiểm điểm. Để facilitator đọc timeline dữ liệu và người liên quan
  chỉ bổ sung dữ kiện.
- **Lead nói phần accountability của mình trước, công khai, trước khi bất kỳ ai khác nói.** Ví dụ:
  *"Trước khi mình đi vào timeline, tôi nói phần của tôi: việc thư mục config không có required review
  là quyết định của tôi từ tháng 3, tôi chọn thế để giảm ma sát. Quyết định đó sai và incident này là
  hệ quả trực tiếp."* Một câu như vậy làm nhiều việc hơn cả một trang nguyên tắc: nó chứng minh rằng
  nói về nguyên nhân không kèm hậu quả cá nhân, bằng ví dụ có rủi ro thật.

**Lực hai: cấp trên yêu cầu "cuối cùng thì ai chịu trách nhiệm".** Câu này sẽ được hỏi, và nếu bạn trả
lời "chúng ta blameless nên không quy trách nhiệm ai", bạn sẽ mất cả cuộc tranh luận lẫn quyền tự chủ
cho lần sau. Ba lý do câu trả lời đó thất bại: nó nghe như trốn tránh; nó không phân biệt blame với
accountability; và nó bỏ trống một nhu cầu thật của người hỏi — người đó cần biết có ai đang sở hữu
việc này.

Cách trả lời có tác dụng, ba bước: nhận accountability về mình ở tầng đúng; đưa ra bằng chứng về việc
sửa cụ thể; và giải thích cơ chế bằng lập luận công cụ chứ không bằng giá trị.

```text
CTO: Tôi đọc postmortem rồi. Nhưng cuối cùng thì ai chịu trách nhiệm việc này?
     Không thể để một bạn mới đổi tham số production lúc 10 giờ đêm rồi không ai
     chịu trách nhiệm gì.

Minh (EM): Em chịu trách nhiệm, anh. Cụ thể là hai việc: thư mục config không có
  required review là quyết định của em từ tháng 3, và việc không có alert cho
  connection pool là hạng mục em đã hoãn hai quý. Duy làm đúng những gì hệ thống
  của em cho phép và không có tín hiệu nào cảnh báo bạn ấy.

  Về việc sửa, em có 6 hạng mục, 2 cái mức P0/P1 xong trước 01/08, em báo anh khi
  đóng. Nếu 30 ngày nữa mình vẫn có incident cùng loại, đó là em làm không tới.

CTO: Nhưng bạn đó không có trách nhiệm gì cả à? Vậy lần sau ai cũng làm thế.

Minh: Nếu Duy vi phạm một quy tắc đã được thống nhất và đã được nhắc riêng, thì đó
  là chuyện performance và em xử lý riêng với bạn ấy, không đưa vào postmortem.
  Ở đây không có quy tắc nào bị vi phạm — em đã kiểm tra.

  Còn lý do em không nêu tên trong postmortem không phải vì em muốn bảo vệ ai. Nó
  là vì lý do rất thực dụng: đêm đó Duy mất 27 phút mới dám nói mình đã đổi gì.
  27 phút đó là phần đắt nhất của sự cố. Nếu postmortem này gắn tên với hậu quả,
  lần sau con số đó sẽ không phải 27 phút mà là không bao giờ, và mình sẽ mất đúng
  thông tin cần nhất trong 15 phút đầu.

  Em đề nghị đo bằng kết quả: nếu trong 2 quý tới, tỉ lệ incident có nguyên nhân
  cấu hình không giảm, anh cứ đổi cách làm của em.

CTO: Được. Nhưng tôi muốn thấy danh sách 6 hạng mục đó trong review tháng.
Minh: Em đưa vào slide đầu tiên.
```

Điểm then chốt: Minh không bảo vệ một nguyên lý, mà đưa ra một cam kết đo được và một điều kiện thất
bại. Với lãnh đạo quen tư duy kết quả, đây là dạng lập luận đi được.

**Lực ba: postmortem bị viết để nộp lên, nên nó không thể chứa sự thật.** Nếu bản postmortem sẽ được
gửi cho khách hàng, cho cơ quan quản lý, hoặc cho ban lãnh đạo, thì mọi câu như "chúng tôi đã hoãn
hạng mục này hai quý" trở thành rủi ro. Kết quả là bản viết trở nên trung tính đến mức vô dụng. Cách
xử lý là **tách hai bản một cách chính thức và công bố việc tách đó**:

| | Bản nội bộ | Bản đối ngoại |
|---|---|---|
| Người đọc | Team, lead, CTO | Khách hàng, đối tác, cơ quan quản lý, người dùng |
| Chứa | Timeline chi tiết, luận cứ tại thời điểm, mọi contributing factor kể cả về áp lực deadline và quyết định bị hoãn, những lỗ cố tình để hở, action item đầy đủ | Ảnh hưởng, thời gian, đã khắc phục ra sao, các biện pháp ngăn tái diễn ở mức nhóm, cam kết có mốc |
| Không chứa | — | Tên cá nhân; suy đoán chưa xác nhận; chi tiết hạ tầng có thể bị lợi dụng; các mâu thuẫn nội bộ |
| Độ dài (minh hoạ) | 3–8 trang | 0,5–1 trang |
| Ai duyệt | Lead của hệ thống | Lead + Comms/Legal/Compliance |
| Nguyên tắc bất biến | Không có câu nào trong bản đối ngoại sai so với bản nội bộ; bản đối ngoại chỉ được **ít chi tiết hơn**, không được **khác** | |

Nguyên tắc ở dòng cuối là điều giữ cho việc tách bản không trở thành hai sự thật. Với fintech, thêm
một lưu ý thực tế: hãy xác định trước với Compliance rằng bản nội bộ là tài liệu kỹ thuật phục vụ cải
tiến, và thống nhất trước điều gì sẽ đi vào hồ sơ chính thức. Việc đàm phán này làm lúc bình thường,
không làm lúc đang bị hỏi.

### Trade-off

| Cặp đối lập | Nghiêng về bên A khi | Nghiêng về bên B khi | Ai trả giá |
|---|---|---|---|
| **Blameless triệt để (A) vs tín hiệu răn đe (B)** | Sự cố xuất phát từ khoảng trống hệ thống; người liên quan thiếu công cụ hoặc thông tin | Hành vi lặp lại, có ý thức bỏ qua quy tắc đã thống nhất và đã nhắc riêng | A: senior có thể cảm thấy tiêu chuẩn lỏng. B: mất dòng thông tin, MTTR tăng |
| **Postmortem đầy đủ cho mọi incident (A) vs chọn lọc (B)** | SEV1/SEV2, hoặc lần lặp lại thứ hai của cùng loại | SEV3 đơn lẻ, nguyên nhân đã rõ và đã sửa trong cùng ngày | A: tiêu 6–10 giờ người cho mỗi bản, làm giảm chất lượng những bản quan trọng. B: bỏ sót mẫu chung xuyên các sự cố nhỏ |
| **Tìm một root cause (A) vs liệt kê contributing factors (B)** | Cần một câu trả lời ngắn cho bên ngoài; hệ thống đơn giản, nhân quả rõ | Hệ thống phân tán, nhiều lớp; incident lặp lại | A: sửa một lớp, các lớp khác vẫn hở. B: danh sách dài, nguy cơ không sửa gì nếu không ưu tiên |
| **Công khai postmortem cho toàn công ty (A) vs giới hạn trong team (B)** | Tổ chức đã có Psychological Safety; muốn tạo học tập chéo team | Văn hoá còn quy tội; nội dung có thể bị dùng làm chứng cứ trong đánh giá | A: người viết tự kiểm duyệt nếu chưa đủ an toàn. B: bài học không lan ra, team khác lặp lại |

### Real-world Scenarios

**Tình huống 1 — Postmortem thành phiên xử, và cách một Tech Lead cứu nó giữa cuộc họp.** Một công ty
logistics. Trong postmortem, Giám đốc Công nghệ mở đầu: "Hôm nay mình xem lại xem sai ở đâu, ai làm
gì." Ba mươi phút đầu, người liên quan là Khoa nói 22 phút, tất cả đều ở dạng giải thích. Không có
contributing factor nào được ghi.

Tech Lead là Linh can thiệp: *"Anh cho em đề nghị một cách làm khác cho 30 phút còn lại. Em muốn dừng
phần Khoa kể lại, vì mình đang nghe một phiên bản đã được sắp xếp — không phải lỗi Khoa, ai cũng thế
sau khi đã biết kết quả. Em có timeline lấy từ audit log ở đây. Em đề nghị mình đi từng mốc và với mỗi
mốc hỏi một câu duy nhất: lúc đó hệ thống có tín hiệu nào để người thao tác biết là nguy hiểm không.
Em cá là mình sẽ tìm ra bốn, năm chỗ không có tín hiệu nào."* Kết quả: 25 phút sau, danh sách có 5
contributing factor, và 3 trong số đó không ai biết trước cuộc họp.

Điều làm can thiệp này hiệu quả: Linh không tranh luận về nguyên tắc blameless — cuộc tranh luận đó sẽ
thua trước một Giám đốc Công nghệ đang muốn câu trả lời. Cô đề nghị một **thủ tục** cho ra nhiều thông
tin hơn, và để kết quả tự chứng minh.

**Tình huống 2 — Action item toàn "đào tạo lại".** Một bộ phận IT của doanh nghiệp truyền thống, sau
ba sự cố liên tiếp về nhập liệu sai ở hệ thống ERP. Cả ba postmortem có cùng action item: "tổ chức
đào tạo lại cho nhân viên nghiệp vụ về quy trình nhập liệu". Hai buổi đào tạo đã diễn ra. Sự cố thứ
tư xảy ra sau 5 tuần.

Phân tích: "đào tạo lại" là action item được chọn vì nó rẻ về mặt chính trị — nó không đòi ngân sách,
không đòi thay đổi hệ thống của nhà cung cấp, và nó đặt trách nhiệm vào một nhóm không có mặt trong
cuộc họp. Khi Tech Lead cuối cùng dựng lại luận cứ tại thời điểm bằng cách ngồi cạnh một nhân viên
nhập liệu trong 2 giờ, phát hiện là: ô nhập mã đơn vị và ô nhập mã kho nằm cạnh nhau, cùng độ dài,
không có validation, và người dùng phải nhập 200 dòng mỗi ngày. Action item đúng — thêm validation và
đổi thứ tự hai ô — mất 1,5 ngày công và sự cố dừng hẳn. Bài học tổng quát: **khi action item là "đào
tạo lại", hãy coi đó là tín hiệu rằng chưa ai đi xem người dùng làm việc thật.**

**Tình huống 3 — Postmortem viết cho khách đọc nên không viết được sự thật.** ODC, khách EU. Nguyên
nhân thật của sự cố: khách yêu cầu ship một tính năng trong 5 ngày, team nói cần 12 ngày, Account
Manager cam kết 6 ngày, và bước load test bị bỏ. Bản postmortem gửi khách viết: "nguyên nhân do một
truy vấn chưa được tối ưu". Đúng về kỹ thuật, và bỏ mất yếu tố quyết định.

Hệ quả sau ba tháng: cùng một cơ chế lặp lại hai lần nữa, vì yếu tố "cam kết ngoài năng lực" chưa bao
giờ được ghi vào bất kỳ tài liệu nào. Cách xử lý: bản nội bộ ghi thẳng "cam kết 6 ngày trong khi đánh
giá là 12 ngày, dẫn tới bỏ bước load test", và action item không phải "nói không với khách" mà là một
thay đổi quy trình cụ thể: mọi cam kết thời gian gửi khách phải có chữ ký của Tech Lead; nếu cam kết
ngắn hơn đánh giá, phải ghi rõ bước nào bị bỏ và ai chấp nhận rủi ro đó. Bản đối ngoại vẫn giữ mức
"chúng tôi đã bổ sung bước kiểm thử tải bắt buộc vào quy trình phát hành" — ít chi tiết hơn, không
khác sự thật.

### Best Practices

- **Dựng timeline từ dữ liệu trước khi nói với ai.** Đây là thay đổi có tỉ lệ lợi ích trên chi phí cao
  nhất trong toàn bộ chủ đề, vì nó đồng thời tăng độ chính xác và giảm tính đối đầu của cuộc họp.
- **Facilitator không phải người trong sự cố và không phải manager của người liên quan.** Lý do: người
  đó cần quyền ngắt khi có câu vi phạm nguyên tắc, và cần không có lợi ích trong kết luận.
- **Yêu cầu tối thiểu bốn contributing factor.** Ràng buộc số lượng là cách rẻ nhất để chống lại xu
  hướng dừng ở nguyên nhân đầu tiên. Nếu chỉ tìm được một, gần như chắc chắn bạn chưa xét các lớp phát
  hiện, kiểm thử, kiến trúc và dòng thông tin.
- **Bắt buộc có mục "cái gì đã hoạt động đúng".** Nó không phải để an ủi: nó xác định các lớp phòng
  thủ đang có tác dụng để bạn không vô tình tháo chúng khi tối ưu chỗ khác.
- **Viết mục "điều chúng ta cố tình không làm".** Nó biến postmortem từ danh sách cam kết thành một
  tài liệu quyết định, và nó tạo mốc để xem lại khi rủi ro thay đổi.
- **Đưa action item vào cùng backlog với công việc thường, có nhãn và có mức.** Một backlog riêng cho
  incident followup luôn thua trong cạnh tranh với backlog chính, vì nó không đi qua cùng cơ chế ưu
  tiên.
- **Cho người liên quan nhất sở hữu một action item.** Chuyển vị thế từ đối tượng sang tác nhân, và
  họ hiểu lỗ hổng đó rõ hơn bất kỳ ai.
- **Công bố postmortem theo mức độ an toàn hiện tại của tổ chức, và tăng dần.** Nếu văn hoá còn quy
  tội, bắt đầu bằng phạm vi team; mở rộng khi đã có bằng chứng rằng nội dung không bị dùng để đánh giá.

### Anti-patterns

- **Postmortem thành phiên xử.** Cơ chế: khi mục tiêu ngầm là xác định người, mọi người chuyển sang
  chế độ tự bảo vệ, và thông tin về hệ thống không được đưa ra vì nó không giúp họ tự bảo vệ. Dấu hiệu
  sớm: câu mở đầu chứa từ "ai"; một người nói quá 50% thời gian; có người xin gặp riêng bạn "để giải
  thích trước" trước cuộc họp.
- **Action item toàn "đào tạo lại", "cẩn thận hơn", "review kỹ hơn".** Cơ chế: những item này không có
  trạng thái đóng nên chúng vĩnh viễn mở, làm loãng danh sách, và chúng đặt gánh nặng lên trí nhớ và ý
  chí của con người — hai thứ suy giảm dưới áp lực, đúng điều kiện mà incident xảy ra. Dấu hiệu sớm:
  trên 30% action item không có tiêu chí xác minh.
- **Postmortem viết cho khách đọc nên không chứa sự thật.** Cơ chế: khi tài liệu duy nhất bị ràng buộc
  bởi đối ngoại, các nguyên nhân thuộc tầng quản trị (áp lực deadline, cam kết ngoài năng lực, hạng
  mục bị hoãn) bị loại khỏi văn bản, nên chúng không bao giờ được sửa. Dấu hiệu sớm: bản postmortem
  không chứa từ nào về quyết định ưu tiên hay áp lực thời gian, dù mọi người trong team đều biết đó là
  yếu tố chính.
- **Một root cause duy nhất, và nó thường là con người.** Dấu hiệu sớm: mục "root cause" trong template
  của bạn là một dòng đơn.
- **Postmortem xong đóng file.** Dấu hiệu sớm: không tồn tại một bảng duy nhất xem được tất cả action
  item đang mở; không ai đọc to danh sách quá hạn trong bất kỳ cuộc họp định kỳ nào.
- **Đo số postmortem đã viết như một chỉ số thành tích.** Cơ chế: chuyển mục tiêu từ thay đổi hệ thống
  sang sản xuất tài liệu; khi bị đo, tổ chức sẽ viết nhiều bản mỏng. Dấu hiệu sớm: có slide báo cáo
  "quý này chúng ta hoàn thành 14/14 postmortem" mà không có slide nào về action item.
- **Chờ quá lâu mới làm postmortem.** Sau 2 tuần, chi tiết mất, người liên quan đã kể lại câu chuyện
  cho nhau nhiều lần và các phiên bản đã hội tụ về một bản dễ kể. Ngưỡng thực dụng: thu timeline trong
  24 giờ, họp trong 3–5 ngày làm việc.

### Khi nào KHÔNG nên áp dụng

- **Sự cố SEV3/SEV4 đơn lẻ với nguyên nhân đã rõ và đã sửa.** Một postmortem đầy đủ tiêu 6–10 giờ
  người. Làm cho mọi sự cố nhỏ sẽ dẫn tới việc các postmortem quan trọng bị làm hình thức vì mọi người
  đã mệt với thủ tục. Thay thế: một ghi chú 5 dòng trong ticket, và một cơ chế soát mẫu chung hàng
  tháng (xem chủ đề 7). Ngoại lệ: lần lặp lại thứ hai của cùng loại thì luôn cần postmortem đầy đủ, kể
  cả khi mức thấp.
- **Khi vấn đề thật là performance của một cá nhân.** Postmortem không phải công cụ để nói với ai đó
  rằng họ đang không đạt yêu cầu. Dùng nó cho việc đó sẽ vừa không hiệu quả (thông điệp bị pha loãng
  trong ngôn ngữ hệ thống) vừa phá hỏng công cụ (mọi người sẽ hiểu postmortem là kênh đánh giá).
- **Khi tổ chức chưa có bất kỳ mức Psychological Safety nào và lead không có quyền bảo vệ người.** Nếu
  bạn biết chắc rằng thông tin trong postmortem sẽ đi thẳng vào một cuộc họp kỷ luật mà bạn không kiểm
  soát, thì việc mời mọi người nói thật là đặt họ vào bẫy. Trong điều kiện đó, hãy làm việc ở phạm vi
  hẹp — bạn và một hai người tin cậy dựng timeline, rút action item kỹ thuật, và song song đầu tư vào
  việc đàm phán lại cơ chế với cấp trên. Bắt đầu bằng một postmortem về một sự cố mà bạn là người chịu
  trách nhiệm rõ ràng nhất.
- **Ngay trong hoặc sát sau incident kéo dài đêm.** Làm postmortem lúc 5 giờ sáng khi mọi người vừa
  thức 6 giờ cho ra dữ liệu tệ và cảm xúc xấu. Việc duy nhất nên làm ngay: chốt timeline thô và snapshot
  bằng chứng, rồi cho về ngủ.
- **Khi nguyên nhân hoàn toàn nằm ngoài phạm vi ảnh hưởng của bạn và không có lớp nào bạn kiểm soát
  được.** Ví dụ một nhà cung cấp hạ tầng có sự cố vùng. Vẫn nên ghi lại ảnh hưởng và cách phản ứng,
  nhưng đừng tổ chức một buổi 90 phút để phân tích hệ thống của người khác. Câu hỏi duy nhất đáng hỏi
  là câu về lớp bạn kiểm soát: bạn có phát hiện sớm không, bạn có phương án dự phòng không, và bạn có
  muốn trả giá cho nó không.

---

## 4. On-call bền vững

### Problem Statement

Bốn hiện tượng, tất cả đều đo được từ dữ liệu bạn đang có.

**Hiện tượng một: tỉ lệ alert actionable thấp.** Lấy toàn bộ alert đã bắn trong 30 ngày, phân loại
mỗi cái vào một trong ba nhóm: đã dẫn tới một hành động của con người; tự hết mà không cần làm gì; là
tiếng ồn (ngưỡng sai, alert của môi trường không quan trọng, alert trùng). Ở nhiều tổ chức chưa dọn
alert, con số minh hoạ là 640 alert/tháng, trong đó 71 dẫn tới hành động — **11% actionable**. Nghĩa
là người on-call bị ngắt 640 lần để tìm 71 việc.

**Hiện tượng hai: một người xử lý phần lớn alert đêm.** Đếm số alert đêm (22:00–07:00) theo người
trong 6 tháng. Nếu phân bố là 4 người với tỉ lệ 62% / 19% / 12% / 7%, bạn không có lịch on-call — bạn
có một người on-call và ba người dự bị danh nghĩa. Người 62% đó sẽ nghỉ việc, và khi họ nghỉ, thời
gian xử lý sự cố của tổ chức sẽ tăng gấp nhiều lần trong 3–6 tháng.

**Hiện tượng ba: thông báo bị tắt.** Dấu hiệu này khó đo bằng công cụ nhưng dễ phát hiện bằng cách
hỏi trực tiếp trong 1-1: "em có để chế độ im lặng cho channel alert không?". Khi câu trả lời là có,
hệ thống phát hiện của bạn đã bị vô hiệu ở tầng cuối, và bạn sẽ chỉ biết điều đó trong incident tiếp
theo mà alert đã bắn 40 phút trước khi có người phản hồi.

**Hiện tượng bốn: on-call không tự xử lý được mà phải gọi người khác.** Đếm tỉ lệ alert mà người
on-call phải leo thang cho một chuyên gia cụ thể. Nếu tỉ lệ đó trên 50%, on-call chưa được delegate
thực sự — nó chỉ là một lớp chuyển tiếp cuộc gọi, và chi phí con người đang bị tính hai lần: người
on-call thức, và chuyên gia cũng thức.

Hậu quả tích lũy có một chuỗi rất cố định và có thể quan sát ở mọi tổ chức bỏ qua chủ đề này: alert
nhiều → người on-call ngủ kém → chất lượng quyết định giảm → tạo thêm lỗi → thêm alert → người giỏi
xin rút khỏi on-call → gánh nặng dồn vào ít người hơn → những người đó nghỉ việc → tri thức vận hành
mất theo → MTTR tăng → downtime tăng. Đây là một vòng lặp có phản hồi dương, nghĩa là nó không tự cân
bằng: nó tăng tốc cho tới khi vỡ.

### First Principles

**Cơ chế một: on-call là một cơ chế chuyển rủi ro từ hệ thống sang con người.** Khi hệ thống không tự
phục hồi được, tổ chức mua khả năng phục hồi bằng cách đặt một con người vào trạng thái sẵn sàng. Đây
là một giao dịch hợp lý trong nhiều điều kiện, nhưng nó là một giao dịch — có giá, và giá được trả
bằng giấc ngủ, sự chú ý và cuối cùng là turnover. Suy ra một hệ quả quản trị quan trọng: **nếu không
có cơ chế phản hồi để chi phí này quay lại tác động vào quyết định kỹ thuật, tổ chức sẽ mua quá nhiều
on-call và quá ít tự động hoá**, vì on-call trông như miễn phí trên bảng ngân sách. Vòng phản hồi bắt
buộc: người on-call có quyền và nghĩa vụ tạo ticket sửa nguyên nhân của alert, và ticket đó cạnh tranh
được với feature trong cùng backlog.

**Cơ chế hai: sự chú ý của con người là tài nguyên cạn, và tín hiệu giả làm cạn nó nhanh hơn tín hiệu
thật.** Đây là cơ chế alert fatigue, và nó đã được mô tả rộng rãi trong y tế lâm sàng: khi tỉ lệ báo
động giả cao, nhân viên bắt đầu bỏ qua báo động, kể cả báo động thật. Điểm quan trọng về mặt thiết kế:
mối quan hệ giữa số alert và chất lượng phản hồi không phải tuyến tính giảm mà có một ngưỡng, sau
ngưỡng đó phản hồi sụp về gần không, vì con người chuyển từ chế độ "xử lý từng cái" sang chế độ "lọc
bỏ theo nhóm". Việc thêm alert thứ 200 không làm hệ thống an toàn hơn 0,5%; nó có thể làm hệ thống mất
khả năng phản ứng với alert thứ 1.

**Cơ chế ba: chi phí của việc bị đánh thức không tuyến tính với thời gian xử lý.** Một alert lúc 3 giờ
sáng cần 4 phút để xử lý không tốn 4 phút. Nó tốn 4 phút cộng thời gian không ngủ lại được (thường
20–90 phút), cộng phần suy giảm của ngày hôm sau, cộng chi phí của trạng thái ngủ không sâu suốt cả
tuần on-call vì cơ thể ở chế độ sẵn sàng. Số minh hoạ dùng để ra quyết định: coi mỗi lần bị đánh thức
có chi phí tương đương 2–4 giờ làm việc hiệu quả. Với con số này, một alert đêm không actionable bắn 3
lần một tuần là một khoản chi 6–12 giờ mỗi tuần — thường lớn hơn chi phí sửa nguyên nhân của nó.

**Cơ chế bốn: tri thức không được viết ra thì không delegate được, và một vai không delegate được thì
không thể luân phiên.** Đây là lý do runbook không phải tài liệu phụ trợ mà là **điều kiện cấu trúc**
của on-call bền vững. Nếu chỉ Tuấn xử lý được alert của payment-svc, thì mọi lịch on-call có Tuấn trong
đó chỉ là hình thức. Runbook là cách chuyển tri thức từ trạng thái nằm trong đầu một người sang trạng
thái tài sản của tổ chức — và nó cũng là cơ chế duy nhất để người đó đi nghỉ phép mà không mang laptop.

**Cơ chế năm: sự công bằng cảm nhận được quyết định tính bền của lịch on-call hơn là khối lượng thật.**
Con người chấp nhận gánh nặng cao nếu thấy nó được phân bổ công bằng và được ghi nhận; họ từ chối gánh
nặng thấp nếu thấy chỉ mình gánh. Hệ quả thực tế: minh bạch lịch, minh bạch số alert theo người, và
một cơ chế bù đắp rõ ràng có tác dụng lớn hơn việc giảm số alert 20%.

### Mental Model

**Mô hình một: on-call như một khoản nợ có lãi.** Mỗi alert không được sửa nguyên nhân là một khoản nợ
mà lãi được trả bằng giấc ngủ của người khác, mỗi tuần, mãi mãi. Cách dùng mô hình này khi tranh luận
ưu tiên với PO: chuyển "sửa alert này" từ ngôn ngữ kỹ thuật sang ngôn ngữ chi phí định kỳ. *"Alert này
bắn 3 lần/tuần lúc đêm. Với ước tính mỗi lần mất 2–4 giờ hiệu quả của một senior, đây là khoảng 25–50
giờ mỗi tháng. Sửa nó mất 3 ngày. Nó hoàn vốn trong 2 tuần."* Đây là dạng lập luận đi được với người
quản lý roadmap, khác hẳn với "hệ thống nhiều alert quá".

**Mô hình hai: Queueing Theory cho người on-call.** Coi người on-call là một server duy nhất với hàng
đợi là các alert. Kết quả cơ bản của lý thuyết hàng đợi: khi hệ số sử dụng tiến tới 1, thời gian chờ
tăng theo phi tuyến và tiến tới vô cùng. Ứng dụng: nếu người on-call đã bị chiếm phần lớn thời gian
bởi alert nhỏ, thời gian phản hồi cho một alert lớn sẽ tăng vọt — không phải vì họ chậm mà vì hàng đợi
đã đầy. Đây là lý do định lượng cho việc dọn alert nhỏ: bạn đang mua thời gian phản hồi cho sự cố lớn.

**Mô hình ba: hai loại alert theo mục đích, không theo mức độ.** Phân loại đúng không phải "quan trọng
/ không quan trọng" mà theo **hành động khả thi trong khung thời gian nào**:

```
Alert
 ├── Page (đánh thức người): có hành động con người phải làm NGAY,
 │      và việc chờ tới sáng gây thiệt hại không hồi được
 │      -> Điều kiện: (a) đang/sắp ảnh hưởng người dùng hoặc tiền
 │                    (b) CÓ hành động cụ thể trong runbook
 │                    (c) hệ thống không tự phục hồi
 └── Ticket (chờ được): mọi thứ còn lại
        -> Vào backlog, xem trong giờ làm việc
```

Điều kiện (b) là chỗ hay bị bỏ. **Một alert đánh thức người mà không kèm hành động cụ thể là một alert
sai thiết kế** — nó chỉ chuyển sự lo lắng, không chuyển khả năng xử lý.

### Practical Framework

**A. Thiết kế lịch on-call.**

| Tham số | Ngưỡng thực dụng | Vì sao |
|---|---|---|
| Số người tối thiểu trong một vòng | 4, tốt hơn là 5–6 | Với 3 người, mỗi người on-call 1 tuần trong 3 — quá dày để hồi phục, và một người nghỉ phép là vòng vỡ |
| Độ dài một phiên | 1 tuần | Ngắn hơn (2–3 ngày) thì chi phí handover cao và không ai kịp nắm bối cảnh; dài hơn thì tích lũy thiếu ngủ |
| Ranh giới ca | Đổi phiên vào giữa buổi sáng ngày làm việc, không đổi vào tối hoặc cuối tuần | Handover cần cả hai người tỉnh và có mặt |
| Lớp thứ hai (secondary) | Bắt buộc với hệ thống mức SEV1 | Người primary không phản hồi trong 10 phút thì tự động chuyển; điều này cũng cho phép primary đi tắm |
| Escalation contact | Một lead có nghĩa vụ nhận cuộc gọi trong 10 phút | Đây là cơ chế chống lại tình huống "lead biến mất" ở chủ đề 2 |
| Quyền hạn của on-call | Được rollback, tắt feature flag, scale, chặn traffic **không cần xin phép** | Nếu phải xin phép, bạn đã thêm một bước chờ vào con đường tới hạn |
| Bù đắp | Tiền phụ cấp phiên + nghỉ bù theo số lần bị đánh thức | Xem bảng dưới |
| Tần suất trên mỗi người | Không quá 1 tuần trong 4 | Trên mức này, tình trạng ngủ không sâu trở thành trạng thái thường trực |

Về bù đắp, ba mô hình dùng ở thị trường Việt Nam, số là minh hoạ để bạn thấy cấu trúc:

| Mô hình | Cách tính (minh hoạ) | Ưu | Nhược |
|---|---|---|---|
| Phụ cấp phiên cố định | 1–2 triệu VND/tuần on-call, bất kể có bị gọi hay không | Đơn giản, ghi nhận cả trạng thái sẵn sàng | Không tạo áp lực giảm alert; team on-call yên tĩnh và team ồn ào nhận như nhau |
| Phụ cấp + tính theo lần đánh thức | 1 triệu/tuần + 300.000/lần bị gọi ngoài giờ + nghỉ bù nửa ngày nếu bị gọi sau 00:00 | Tạo tín hiệu tài chính khiến việc dọn alert được ưu tiên; công bằng | Cần cơ chế ghi nhận chính xác; có nguy cơ tạo động cơ ngược nếu người ta không tắt alert của mình |
| Nghỉ bù thuần | Không phụ cấp, nghỉ bù 1 ngày cho mỗi 3 lần bị gọi đêm | Không tốn ngân sách, phù hợp startup thiếu tiền | Nghỉ bù thường không dùng được nếu team đã thiếu người — biến thành lời hứa suông |

Lưu ý cho bối cảnh Việt Nam: nhiều tổ chức không có bất kỳ cơ chế bù nào và coi on-call là phần mặc
định của công việc. Điều này vận hành được trong một giai đoạn vì engineer thường không đòi, nhưng nó
không tạo ra vòng phản hồi nào — chi phí không hiện ra ở đâu, nên nó không được tối ưu. Nếu bạn không
xin được ngân sách, hãy ít nhất làm cho chi phí **hiện hình**: một slide trong review tháng với số lần
đánh thức theo người và theo alert. Số liệu hiện hình là bước đầu tiên của mọi cơ chế phản hồi.

**B. BẢNG tiêu chí phân loại alert.** Đây là bảng cần dán lên tường và dùng để soát toàn bộ alert hiện
có, mỗi quý một lần.

| Tiêu chí | Page (đánh thức lúc 3 giờ sáng) | Ticket (chờ tới sáng) | Xoá alert |
|---|---|---|---|
| **Ảnh hưởng** | Người dùng đang không dùng được, hoặc tiền/dữ liệu đang sai | Suy giảm nhưng còn dùng được; hoặc chỉ ảnh hưởng nội bộ | Không ảnh hưởng ai |
| **Tính không hồi được** | Chờ tới sáng gây thiệt hại không sửa được (mất dữ liệu, sai bút toán, vi phạm SLA hợp đồng) | Chờ được, thiệt hại tuyến tính và sửa được | — |
| **Hành động** | Có runbook với hành động cụ thể, on-call làm được một mình | Cần phân tích, không có hành động ngay | Không có hành động nào |
| **Tự phục hồi** | Hệ thống không tự phục hồi, hoặc tự phục hồi nhưng để lại hệ quả | Tự phục hồi trong vài phút và không để lại hệ quả | Tự phục hồi luôn |
| **Tỉ lệ actionable lịch sử** | >80% các lần bắn dẫn tới hành động | 20–80% | <20% — sửa ngưỡng hoặc xoá |
| **Ví dụ (fintech, minh hoạ)** | Tỉ lệ giao dịch lỗi >5% trong 5 phút; job đối soát thất bại; hàng đợi bút toán không tiêu thụ trong 10 phút; chứng chỉ TLS hết trong 24 giờ | p95 API tăng gấp 2 nhưng dưới ngưỡng SLO; disk 75%; một node bị restart nhưng đã tự lên lại | CPU >80% trong 1 phút; một request lỗi 500 đơn lẻ; alert từ môi trường staging |
| **Ví dụ (e-commerce)** | Check-out error >3% trong 5 phút; không đồng bộ được tồn kho >15 phút trong campaign | Tỉ lệ ảnh sản phẩm lỗi tăng; job đồng bộ giá chậm 30 phút ngoài campaign | Alert "traffic tăng cao" trong đúng ngày campaign |

Quy tắc soát rất đơn giản và nên làm thành nghi thức quý: **mọi alert mức Page phải trả lời được ba câu
— nó ảnh hưởng ai, người on-call phải làm gì (link runbook), và trong 90 ngày qua nó bắn bao nhiêu lần
và bao nhiêu lần dẫn tới hành động.** Alert nào không trả lời được cả ba thì hạ về Ticket hoặc xoá.
Việc xoá alert cần được coi là hành động tích cực, có thẩm quyền rõ ràng, và được nêu trong review —
nếu không, không ai dám xoá vì nỗi lo "nhỡ nó có ích".

**C. Ngưỡng alert noise — đo bằng gì và ngưỡng nào.**

| Chỉ số | Cách tính | Ngưỡng lành (minh hoạ) | Ngưỡng báo động |
|---|---|---|---|
| Tỉ lệ alert actionable | (số alert dẫn tới hành động) / (tổng alert) | >70% với mức Page | <40% — hệ thống phát hiện đang mất giá trị |
| Số lần đánh thức mỗi phiên on-call | Đếm alert trong 22:00–07:00 | 0–2 lần/tuần | >5 lần/tuần — không bền, sẽ mất người |
| Số phiên on-call có 0 lần bị đánh thức | Đếm theo quý | >50% số phiên | <20% |
| Tỉ lệ alert xử lý được bằng runbook mà không cần leo thang | Ghi nhận khi đóng alert | >70% | <50% — on-call chưa delegate được |
| Độ lệch phân bố alert giữa người | Tỉ lệ của người cao nhất | <35% | >50% — bạn có một người on-call, không có lịch |

**D. Runbook là điều kiện để on-call được delegate.** Ba nguyên tắc về runbook đáng nhớ hơn mọi
template: nó viết cho người **chưa biết hệ thống**, đọc lúc **3 giờ sáng**, và mọi bước phải là **lệnh
copy được** chứ không phải mô tả. Phép thử duy nhất có giá trị: đưa runbook cho một engineer khác team,
để họ diễn tập trong giờ làm việc; mọi chỗ họ phải hỏi là một chỗ runbook thiếu.

**TEMPLATE — Runbook skeleton.**

````markdown
# RUNBOOK: [Tên alert đúng như hiện trong hệ thống alert]

Service: order-svc   |   Owner team: Order   |   Cập nhật: 2026-07-20
Người cập nhật cuối: @tuan   |   Lần diễn tập gần nhất: 2026-06-12 (@vy, 14 phút)

## 0. TRẢ LỜI NGAY — 30 giây đầu
- Alert này nghĩa là gì (một câu, ngôn ngữ thường):
  Số connection tới cluster database dùng chung đã vượt 80% giới hạn.
- Ai đang bị ảnh hưởng: chưa ai, NHƯNG nếu đạt 100% thì payment-svc sẽ lỗi ngay.
- Có phải Page không: CÓ. Không chờ tới sáng được vì hệ thống không tự phục hồi.
- Nếu bạn chỉ làm được một việc: chạy Bước 2.1 (giảm pool của order-svc).

## 1. XÁC MINH (2 phút) — alert này có thật không
```bash
# 1.1 Số connection hiện tại và giới hạn
psql -h $DB_HOST -c "SELECT count(*) FROM pg_stat_activity;"
psql -h $DB_HOST -c "SHOW max_connections;"
# 1.2 Ai đang chiếm
psql -h $DB_HOST -c "SELECT application_name, count(*) FROM pg_stat_activity
                     GROUP BY 1 ORDER BY 2 DESC;"
```
Dashboard: https://grafana.internal/d/db-shared  (panel "Connections by service")
Nếu số connection < 60% giới hạn: alert giả. Ghi vào ticket ALERT-NOISE và đi ngủ.

## 2. GIẢM THIỂU — theo thứ tự, dừng khi hết
### 2.1 Giảm pool của service chiếm nhiều nhất (5 phút, an toàn, ưu tiên 1)
```bash
kubectl -n prod set env deploy/order-svc DB_POOL_MAX=20
kubectl -n prod rollout status deploy/order-svc --timeout=180s
```
Kỳ vọng: số connection giảm trong 2–3 phút.
Rủi ro: thời gian chờ connection của order-svc tăng; chấp nhận được.

### 2.2 Nếu 2.1 không đủ: giảm concurrency của job đối soát (3 phút)
```bash
kubectl -n prod scale deploy/recon-worker --replicas=1
```
Job đối soát sẽ chậm. KHÔNG tắt hẳn — nó phải xong trước 06:00.

### 2.3 Nếu vẫn không đủ: kill các connection idle quá 10 phút
```bash
psql -h $DB_HOST -f /runbooks/sql/kill_idle_connections.sql
```
CẢNH BÁO: có thể làm lỗi một số request đang chạy. Chỉ dùng khi payment-svc ĐÃ lỗi.

## 3. KHÔNG ĐƯỢC LÀM
- Không tăng `max_connections` của cluster: cần restart, mất 5–10 phút downtime toàn bộ.
- Không restart cluster database.
- Không tắt recon-worker hoàn toàn (dữ liệu đối soát sẽ thiếu, phải chạy lại thủ công).

## 4. LEO THANG
- Sau 15 phút chưa giảm được -> gọi điện @tuan (0xx-xxx-xxxx), secondary @linh.
- Nếu payment-svc đã lỗi -> tuyên bố SEV2 ngay, không chờ leo thang.

## 5. GIỚI HẠN TÀI NGUYÊN CẦN BIẾT (bảng tra nhanh)
| Tài nguyên | Giới hạn | Ai dùng chung |
|---|---|---|
| max_connections cluster shared-db-1 | 200 | order-svc, payment-svc, recon-worker |
| DB_POOL_MAX order-svc (mặc định) | 20 | |
| DB_POOL_MAX payment-svc (mặc định) | 40 | |

## 6. SAU KHI XONG
- [ ] Ghi vào #inc-log một dòng: thời gian, alert, hành động, kết quả.
- [ ] Tạo ticket nhãn `oncall-followup` nếu nguyên nhân chưa được sửa.
- [ ] Nếu alert này là lần thứ 3 trong 30 ngày: gắn nhãn `alert-recurring`,
      nó sẽ vào danh sách bàn ở incident review tháng.
````

**E. Quy tắc "người on-call có quyền tạo ticket sửa nguyên nhân alert".** Quy tắc này là bộ phận tạo
vòng phản hồi, và nó cần bốn điều kiện để không thành khẩu hiệu:

1. **Quyền tạo, không cần xin.** Người on-call tạo ticket trực tiếp vào backlog của team, nhãn
   `oncall-followup`, tự đặt mức đề nghị.
2. **Một hạn mức cứng trong sprint.** Ví dụ: 15–20% capacity mỗi sprint dành cho `oncall-followup`,
   và đó là cam kết được PO đồng ý trước, không phải đàm phán lại mỗi sprint. Không có hạn mức, ticket
   loại này sẽ luôn thua feature.
3. **Quy tắc leo thang tự động theo tần suất.** Một alert bắn lần thứ 3 trong 30 ngày thì ticket sửa
   nguyên nhân của nó tự động lên mức P1 và vào sprint kế tiếp. Đây là cơ chế thay cho việc phải thuyết
   phục ai đó mỗi lần.
4. **Người on-call có quyền tắt hoặc hạ mức một alert không actionable, ghi lý do.** Nếu việc tắt alert
   cần phê duyệt, không ai làm, và số alert chỉ tăng một chiều.

**F. Quy trình handover cuối phiên (10 phút, bắt buộc).** Người sắp hết phiên viết vào channel: (1)
những alert đã bắn trong tuần và cái nào chưa được sửa nguyên nhân; (2) những thứ đang ở trạng thái
mong manh cần để mắt; (3) các thay đổi lớn dự kiến trong tuần tới (deploy lớn, migration, campaign);
(4) ticket `oncall-followup` đã tạo. Không có handover, mỗi phiên bắt đầu lại từ đầu và các mẫu lặp lại
không được phát hiện.

### Trade-off

| Cặp đối lập | Nghiêng về bên A khi | Nghiêng về bên B khi | Ai trả giá |
|---|---|---|---|
| **SLO chặt (A) vs SLO lỏng (B)** | Hệ thống liên quan tiền, có SLA hợp đồng, khách B2B, chi phí uy tín cao; thị trường có đối thủ dễ chuyển đổi | Sản phẩm sớm, người dùng chấp nhận lỗi, tốc độ ra tính năng quyết định sống còn; hệ thống nội bộ | A: chi phí vận hành cao — hạ tầng dự phòng, nhiều người on-call, tốc độ ra tính năng giảm; engineer chịu áp lực. B: mất khách trong các đợt sự cố, và mất niềm tin khó lấy lại hơn nhiều so với mất tính năng |
| **Dev on-call chính hệ thống mình viết (A) vs đội vận hành riêng (B)** | Muốn feedback loop chặt: người viết code chịu hậu quả của thiết kế, nên họ tự đầu tư vào khả năng vận hành. Hệ thống thay đổi nhanh, tri thức khó viết thành SOP | Hệ thống ổn định, quy trình chuẩn hoá được; cần phủ 24/7 mà team dev quá nhỏ; yêu cầu compliance về phân tách nhiệm vụ (ngân hàng) | A: gánh nặng lên chính đội đang phải ra tính năng; nếu không giảm capacity tương ứng thì đây là làm hai việc với một mức lương. B: mất feedback loop — người viết code không bao giờ biết thiết kế của mình gây khổ thế nào, nên không sửa; thêm một tầng chuyển tiếp làm MTTR tăng |
| **Nhiều alert để không bỏ sót (A) vs ít alert nhưng chuẩn (B)** | Hệ thống mới, chưa biết chỗ nào hỏng, đang trong giai đoạn học | Đã có dữ liệu về lịch sử alert; đội đã có dấu hiệu alert fatigue | A: sau ngưỡng, phản hồi sụp và bạn mất cả những alert quan trọng. B: có rủi ro bỏ sót một dạng lỗi mới trong một thời gian |
| **On-call có quyền rollback không cần phê duyệt (A) vs kiểm soát chặt (B)** | Cần MTTR thấp; hệ thống có cơ chế rollback an toàn và có audit log | Thay đổi có rủi ro dữ liệu; môi trường compliance yêu cầu phê duyệt | A: một rollback sai có thể làm tệ hơn. B: mỗi phút chờ phê duyệt là downtime, và người on-call học được rằng họ không có quyền, nên họ ngừng chủ động |

Một cách dung hoà cặp thứ hai đáng cân nhắc cho tổ chức 30–80 engineer: **đội vận hành xử lý tầng một
theo runbook, dev on-call là tầng hai, và dev bắt buộc nhận báo cáo hàng tuần về mọi alert của hệ thống
mình dù không bị gọi.** Cách này giữ được phần lớn feedback loop mà không đánh thức dev mỗi đêm. Điều
kiện: báo cáo phải được đọc thật và phải sinh ra ticket, nếu không nó là email bị lọc.

### Real-world Scenarios

**Tình huống 1 — 640 alert và một tuần dọn dẹp.** Một e-commerce, 6 team, 34 engineer. On-call gồm 5
người nhưng thực tế Tuấn xử lý 62% alert đêm. Trong 1-1, Tuấn nói câu quyết định: "Em không nghỉ vì
việc nặng. Em nghỉ vì em không nhớ lần cuối em ngủ một đêm không nghĩ tới điện thoại."

Cách EM là Trang xử lý, theo thứ tự: (a) tuần đó, Trang trực tiếp lấy dữ liệu 90 ngày alert và phân
loại theo bảng tiêu chí — mất 6 giờ; (b) kết quả: trong 214 rule alert, 89 chưa bắn lần nào trong 90
ngày, 47 bắn nhiều hơn 20 lần với tỉ lệ actionable dưới 10%, 12 rule chiếm 68% tổng số lần bắn; (c)
Trang xin PO một tuần "alert cleanup" cho 2 người, lập luận bằng chi phí: 12 rule đó gây khoảng 40 lần
đánh thức mỗi tháng, ước tính 80–160 giờ hiệu quả mất đi; (d) sau một tuần: xoá 89 rule chết, hạ 31
rule từ Page về Ticket, sửa ngưỡng 9 rule, và viết 4 runbook cho các alert còn lại ở mức Page.

Kết quả sau hai tháng, số minh hoạ: alert đêm từ 40 xuống 7 lần/tháng; tỉ lệ actionable của mức Page
từ 11% lên 74%; và một hiệu ứng không lường trước nhưng thường xảy ra — số người tình nguyện tham gia
vòng on-call tăng từ 5 lên 8, vì on-call không còn là một tuần khổ hình. Trade-off đã trả: một tuần
capacity của 2 người, và trong hai tháng đó có một sự cố mức SEV3 bị phát hiện muộn 25 phút vì một rule
đã bị hạ mức. Trang ghi nhận điều này công khai trong review, và đó là cách đúng: nếu bạn không thừa
nhận cái giá, lần sau bạn sẽ không được cho dọn nữa.

**Tình huống 2 — ODC và bài toán múi giờ.** Công ty ODC ở Đà Nẵng, khách Mỹ (múi giờ lệch 11–14 giờ).
Giờ cao điểm của khách là đêm ở Việt Nam. Khách yêu cầu phản hồi trong 15 phút, 24/7. Team có 7 người.

Ba phương án đã cân nhắc: (a) toàn bộ team on-call theo vòng, chấp nhận thức đêm — bền được khoảng 4
tháng, sau đó 2 người nghỉ; (b) tuyển 2 người làm ca đêm cố định — chi phí cao, khó tuyển, và người
làm ca đêm cố định bị cô lập khỏi tri thức của team; (c) thoả thuận lại SLA theo mức độ: chỉ SEV1 mới
yêu cầu phản hồi 15 phút 24/7, SEV2 phản hồi trong 4 giờ, SEV3 trong giờ làm việc Việt Nam.

Chọn (c), và điểm cần học là **cách đàm phán**. Lập luận không phải "team chúng tôi cần ngủ" — với
khách, đó là vấn đề nội bộ của nhà cung cấp. Lập luận đi được: *"Theo dữ liệu 6 tháng, 82% ticket ngoài
giờ của chúng tôi thuộc mức không ảnh hưởng giao dịch. Nếu chúng tôi phải phản hồi cả 82% đó trong 15
phút, người xử lý SEV1 sẽ là người đã bị đánh thức 3 lần trong đêm. Chúng tôi đề nghị phân mức để dồn
năng lực phản hồi vào đúng loại việc mà 15 phút thật sự quan trọng."* Kèm theo một cam kết đối ứng: báo
cáo hàng tháng về thời gian phản hồi theo mức, minh bạch cả những lần trượt. Khách đồng ý sau một lần
họp. Bài học: SLA quá chặt thường không phải nhu cầu thật của khách mà là mặc định của hợp đồng chưa
ai xem lại — và cách mở lại nó là bằng dữ liệu, không bằng lời đề nghị thông cảm.

**Tình huống 3 — Ngân hàng số: on-call không có quyền rollback.** Trong một ngân hàng số, quy trình
yêu cầu mọi thay đổi production phải có phê duyệt của trưởng phòng Vận hành. Ban đêm, phê duyệt này
mất trung bình 22 phút (số minh hoạ, đo từ ticket 6 tháng). Nghĩa là mỗi sự cố có sẵn 22 phút downtime
đã được thiết kế vào quy trình.

Cách xử lý không phải phá bỏ kiểm soát — trong môi trường này, kiểm soát tồn tại vì lý do hợp lệ. Cách
xử lý là **phê duyệt trước cho một tập hành động đóng**: một danh sách 6 hành động (rollback về phiên
bản trước gần nhất, tắt feature flag, scale ra, chuyển traffic sang vùng dự phòng, bật rate limit,
dừng job batch) được phê duyệt trước dưới dạng chính sách, ghi rõ điều kiện áp dụng, và mọi lần dùng
đều tự động ghi vào audit log kèm thông báo tới trưởng phòng. Kiểm soát chuyển từ **phê duyệt trước
từng lần** sang **giới hạn phạm vi cộng với kiểm toán sau** — cùng mức đảm bảo, giảm 22 phút. Đây là
một mẫu chuyển hoá dùng được ở mọi tổ chức có compliance nặng, và nó là cuộc trò chuyện phải làm với
bộ phận Risk lúc bình thường, kèm số đo về 22 phút.

### Best Practices

- **Đặt sàn 4–5 người cho một vòng on-call, và nếu không đủ người thì thu hẹp phạm vi hệ thống được
  on-call thay vì chia nhỏ vòng.** Lý do: một vòng 3 người không hồi phục được và sẽ vỡ khi một người
  nghỉ phép — mà nghỉ phép là điều chắc chắn xảy ra.
- **Soát toàn bộ alert mỗi quý theo bảng tiêu chí, và coi việc xoá alert là thành tích.** Không có nghi
  thức này, số alert chỉ tăng: mỗi incident thêm alert mới, không ai bỏ alert cũ.
- **Không tạo alert mức Page nếu chưa có runbook.** Đây là ràng buộc rẻ nhất và hiệu quả nhất trong
  toàn bộ chủ đề. Nó buộc người tạo alert phải trả lời "người bị đánh thức sẽ làm gì" trước khi họ được
  quyền đánh thức ai.
- **Diễn tập runbook trong giờ làm việc, với người không thuộc team sở hữu.** Mỗi runbook nên được một
  người ngoài team chạy thử ít nhất mỗi 6 tháng; mọi câu hỏi họ phải hỏi là một lỗ hổng cần vá. Ghi
  ngày diễn tập vào đầu runbook để biết cái nào đã cũ.
- **Trao quyền hành động và nói ra bằng chữ.** Câu cần có trong tài liệu on-call: "Bạn được phép
  rollback, tắt feature flag, scale, và chặn traffic mà không cần hỏi ai. Nếu quyết định của bạn sai,
  đó là trách nhiệm của người viết chính sách này, không phải của bạn."
- **Công khai số lần đánh thức theo người mỗi tháng.** Nó tạo áp lực đúng chỗ (giảm alert) và làm sự
  bất công hiện hình trước khi nó thành đơn xin nghỉ.
- **Hỏi trong mọi 1-1 với người vừa on-call: tuần rồi bị gọi mấy lần, và cái nào không đáng.** Câu thứ
  hai quan trọng hơn câu thứ nhất; nó là nguồn dữ liệu tốt nhất về alert noise và nó rẻ hơn mọi công cụ.

### Anti-patterns

- **On-call một người mãi mãi.** Cơ chế: tri thức tập trung, nên càng lâu càng không thể luân phiên —
  đây là một vòng phản hồi dương tự khoá. Đồng thời tổ chức mất khả năng biết hệ thống khó vận hành thế
  nào, vì chỉ một người biết và người đó đã quen chịu. Dấu hiệu sớm: một người xử lý trên 50% alert
  đêm; người đó không dám nghỉ phép dài; câu "gọi Tuấn đi" là quy trình leo thang thực tế.
- **Alert nhiều đến mức bị tắt thông báo.** Cơ chế: đã mô tả ở First Principles — sau ngưỡng, phản hồi
  sụp theo nhóm chứ không giảm dần. Điều nguy hiểm là hệ thống trông vẫn hoạt động (alert vẫn bắn, vẫn
  được ghi log) nên không ai biết tầng cuối đã chết. Dấu hiệu sớm: tỉ lệ actionable dưới 40%; có alert
  bắn lại hơn 20 lần mà chưa ai sửa; trong 1-1 có người thừa nhận để chế độ im lặng.
- **On-call không có quyền rollback.** Cơ chế: thêm một bước chờ tuần tự vào con đường tới hạn, và
  đồng thời gửi một thông điệp rằng người on-call chỉ là người báo tin — họ sẽ hành xử đúng như vậy.
  Dấu hiệu sớm: trong timeline các incident, xuất hiện các khoảng "chờ phê duyệt" từ 10 phút trở lên.
- **Alert Page không có runbook.** Dấu hiệu sớm: người on-call phải gọi chuyên gia ở trên 50% alert.
- **Lịch on-call tồn tại nhưng leo thang thực tế luôn về cùng một người.** Đây là phiên bản ngụy trang
  của anti-pattern đầu tiên và khó phát hiện hơn, vì lịch trông đẹp. Dấu hiệu sớm: so sánh danh sách
  "người trong phiên" với danh sách "người thực sự thao tác" trong 10 incident gần nhất.
- **Coi việc thức đêm là biểu hiện của tinh thần trách nhiệm và khen nó công khai.** Cơ chế: bạn đang
  thưởng cho triệu chứng, nên bạn sẽ có nhiều triệu chứng hơn. Người ta sẽ tự nguyện thức, và động lực
  sửa nguyên nhân giảm. Dấu hiệu sớm: trong all-hands, câu chuyện được kể là "team đã thức tới 4 giờ
  sáng để cứu hệ thống" mà không có câu nào về việc vì sao hệ thống cần được cứu.

### Khi nào KHÔNG nên áp dụng

- **Sản phẩm nội bộ hoặc hệ thống mà downtime ban đêm không gây thiệt hại.** Một hệ thống báo cáo nội
  bộ mà không ai dùng sau 20:00 thì không cần on-call đêm; nó cần một alert mức Ticket và một người xem
  lúc 8 giờ sáng. Việc dựng on-call cho hệ thống này tiêu người thật để mua một giá trị bằng không, và
  nó còn làm loãng sự chú ý dành cho hệ thống thật sự quan trọng.
- **Team dưới 4 người.** Với 3 người, on-call 24/7 theo vòng là không bền và bạn nên chọn một trong ba:
  giảm phạm vi cam kết (chỉ phản hồi trong giờ mở rộng, ví dụ 07:00–23:00); mua dịch vụ vận hành tầng
  một; hoặc chấp nhận rằng ban đêm hệ thống không có người và đầu tư vào khả năng tự phục hồi (retry,
  circuit breaker, degradation tự động). Phương án thứ ba thường bị bỏ qua nhưng nó là phương án duy
  nhất tăng năng lực thật.
- **Giai đoạn trước product-market fit, khi hệ thống hầu như không có người dùng.** Đặt hai người vào
  vòng on-call cho một sản phẩm có 200 người dùng đang thử nghiệm là dùng tài nguyên đắt nhất của
  startup vào một rủi ro không tồn tại. Đúng hơn: một người nhận alert qua điện thoại, không cam kết
  thời gian, và toàn bộ năng lượng dồn vào việc tìm PMF.
- **Khi cơ chế bù đắp không thể có và bạn không thể thay đổi điều đó.** Đây là một khuyến nghị không
  thoải mái nhưng thật: nếu tổ chức không cho phép bất kỳ hình thức bù nào — không tiền, không nghỉ bù,
  không giảm capacity — thì việc bạn dựng một lịch on-call chính thức là hợp thức hoá một khoản chi
  không được trả, và bạn sẽ là người mà team quy trách nhiệm. Trong điều kiện đó, hãy giữ on-call ở mức
  không chính thức (best-effort, không cam kết thời gian phản hồi), làm cho chi phí hiện hình bằng số
  liệu, và dùng số liệu đó để đàm phán. Chính thức hoá sau khi có cơ chế bù, không trước.
- **Khi vấn đề thật là hệ thống không tự phục hồi được ở mức cơ bản.** Nếu mỗi tuần có 3 lần một service
  cần được restart bằng tay, đừng thiết kế lịch on-call tốt hơn — hãy thêm health check và tự động
  restart. On-call là cơ chế cho những việc cần phán đoán của con người; dùng nó cho những việc máy làm
  được là đốt người vào chỗ rẻ nhất để tự động hoá.

---

## 5. SLO, SLI và Error Budget

### Problem Statement

Cuộc tranh luận sau đây diễn ra ở mọi tổ chức chưa có SLO, và nó luôn có cùng hình dạng:

```text
PO:   Sprint sau mình cần xong tính năng thanh toán trả góp, campaign 15/09 đã chốt.
Lead: Em cần 5 ngày để làm phần retry và idempotency cho luồng này.
PO:   Cái đó có ảnh hưởng người dùng không? Hay là làm cho chắc?
Lead: Nếu không có, khi cổng thanh toán lỗi thì có thể trừ tiền hai lần.
PO:   Xác suất bao nhiêu?
Lead: Em không biết chính xác. Nhưng nó rủi ro.
PO:   Vậy mình làm sau nhé, campaign quan trọng hơn.
```

Ai thắng cuộc tranh luận này không phụ thuộc vào ai đúng, mà phụ thuộc vào ai có thẩm quyền cao hơn
hoặc ai kiên trì hơn. Đó là dấu hiệu của một cuộc tranh luận **không có dữ liệu chung** — và nó sẽ tái
diễn nguyên văn mỗi sprint, tiêu năng lượng của cả hai bên, đồng thời để lại dư vị rằng engineering
"lúc nào cũng muốn làm thêm" và product "lúc nào cũng ép".

Ba hiện tượng khác:

**Hiện tượng một: mục tiêu ổn định được phát biểu bằng con số vô nghĩa.** Trên slide có "cam kết uptime
99,99%". Hỏi lại ba câu thì không ai trả lời được: uptime của cái gì (một endpoint? cả hệ thống? luồng
người dùng nào?), đo bằng gì (ping? tỉ lệ request thành công? tỉ lệ giao dịch hoàn tất?), và tính trong
cửa sổ nào (tháng? quý? rolling 28 ngày?). Một con số không có ba thứ đó không phải mục tiêu, nó là một
lời hứa không kiểm chứng được — và nó có tác hại thật: nó chặn cuộc thảo luận về mức phù hợp.

**Hiện tượng hai: đầu tư reliability phân bổ theo tiếng ồn, không theo giá trị.** Service nào có người
sở hữu nhiệt tình nhất, hoặc gây ra incident gần nhất, thì được đầu tư. Kết quả điển hình: hệ thống báo
cáo nội bộ có tài liệu vận hành đầy đủ, ba môi trường, monitoring chi tiết; luồng thanh toán thì không
ai biết p99 là bao nhiêu.

**Hiện tượng ba: sau một incident, phản ứng luôn là "phải chặt hơn nữa", và không bao giờ có ai nói
"chỗ này chúng ta đang chặt quá".** Cơ chế lệch một chiều này, tích lũy 2–3 năm, tạo ra một tổ chức có
quy trình phát hành nặng cho mọi thứ, kể cả cho những service mà downtime 4 giờ không ai để ý.

### First Principles

**Cơ chế một: 100% là mục tiêu sai vì chi phí biên tăng phi tuyến ở mỗi số 9 thêm vào.** Đi từ 99% lên
99,9% thường cần monitoring tốt và khả năng rollback nhanh — chi phí vừa phải. Từ 99,9% lên 99,99% cần
loại bỏ mọi điểm hỏng đơn lẻ, dự phòng đa vùng, triển khai tự động không downtime, và một tổ chức trực
24/7 — chi phí gấp nhiều lần bước trước. Từ 99,99% lên 99,999% cần dự phòng chủ động ở mọi tầng kể cả
tầng dữ liệu, diễn tập chuyển vùng định kỳ, và thường phải hy sinh tốc độ ra tính năng gần như hoàn
toàn. Thêm vào đó, có một chặn trên bạn không kiểm soát: nếu nhà cung cấp mạng, thiết bị của người
dùng, và các đối tác thanh toán đã tiêu một phần độ tin cậy, thì việc bạn đạt 99,999% ở tầng của mình
không đổi được trải nghiệm người dùng. Kết luận: mục tiêu đúng không phải "cao nhất có thể" mà là **mức
mà người dùng không còn cảm nhận được sự khác biệt**, vì mọi đồng chi thêm sau điểm đó không tạo giá
trị.

**Cơ chế hai: error budget biến một cuộc tranh cãi giá trị thành một quyết định có dữ liệu.** Cuộc
tranh luận "ship nhanh hay ổn định" không giải được vì hai bên đang so sánh hai đại lượng không cùng
đơn vị: PO nói bằng cơ hội thị trường, engineer nói bằng rủi ro kỹ thuật. Error budget tạo ra một đơn
vị chung: **số phút được phép hỏng trong tháng**. Khi đã có đơn vị chung, cuộc trò chuyện đổi hình
dạng — từ "anh có tin tôi không" sang "chúng ta còn 91 phút, việc này tiêu bao nhiêu". Đây là cơ chế
quan trọng nhất trong toàn bộ chủ đề, và nó là một cơ chế **quản trị** chứ không phải kỹ thuật.

**Cơ chế ba: một mục tiêu không có hậu quả thì không phải mục tiêu.** SLO chỉ có tác dụng nếu việc cháy
error budget dẫn tới một thay đổi hành vi được định trước. Không có chính sách hậu quả, SLO trở thành
một chỉ số trên dashboard mà mọi người xem rồi tiếp tục làm như cũ. Điều này cũng đúng theo chiều
ngược: nếu error budget còn nhiều mà tổ chức vẫn không dám ship nhanh hơn, thì SLO cũng đang không được
dùng.

**Cơ chế bốn: SLI phải đo trải nghiệm người dùng, vì mọi thứ khác đều là đại lượng trung gian có thể
tốt trong lúc người dùng đang khổ.** CPU 40%, memory ổn, không có pod nào restart, và người dùng không
thanh toán được — tình huống này xảy ra thường xuyên. Nguyên tắc chọn SLI: đứng ở vị trí người dùng và
hỏi "họ đang cố làm gì, và điều gì khiến họ coi lần đó là thất bại". Câu trả lời gần như luôn có dạng
"tỉ lệ các lần thử X thành công trong dưới Y giây".

**Cơ chế năm: độ tin cậy của một luồng bằng tích độ tin cậy của các thành phần trên luồng đó.** Nếu
luồng check-out đi qua 5 service, mỗi service 99,9%, thì luồng đó xấp xỉ 99,5% — tệ hơn từng thành
phần. Hệ quả quản trị: **SLO phải đặt ở mức luồng người dùng trước, rồi mới phân bổ xuống service**,
chứ không phải cộng dồn từ dưới lên. Đây cũng là lý do việc tách thêm service luôn phải kèm câu hỏi về
độ tin cậy tổng của luồng.

### Mental Model

**Mô hình một: error budget như một ngân sách chi tiêu.** Bạn có một khoản, bạn được chi, và khi hết
thì hành vi đổi. Chi tiêu hợp lệ gồm: incident, deploy rủi ro, thử nghiệm, migration, chaos drill. Điểm
tinh tế và hay bị bỏ: **error budget còn dư nhiều không phải thành tích.** Nếu cuối tháng bạn còn 95%
budget liên tục trong 6 tháng, điều đó nghĩa là bạn đang thận trọng quá mức so với mục tiêu đã đặt —
hoặc SLO của bạn quá lỏng so với thực tế hệ thống. Cả hai trường hợp đều là thông tin để hành động: một
là ship nhanh hơn, hai là siết SLO.

**Mô hình hai: bảng quy đổi "số 9" — công cụ chấm dứt tranh luận nhanh nhất.** Người ta nói 99,99% vì
nó nghe chuyên nghiệp. Đưa ra bảng này thường làm cuộc trò chuyện đổi ngay:

| SLO | Downtime cho phép / tháng (30 ngày) | / tuần | / quý | Nghĩa thực tế với một incident |
|---|---|---|---|---|
| 99% | 7 giờ 12 phút | 1 giờ 41 phút | 21 giờ 36 phút | Một sự cố nửa ngày mỗi tháng vẫn đạt |
| 99,5% | 3 giờ 36 phút | 50 phút | 10 giờ 48 phút | Hai sự cố 90 phút mỗi tháng |
| 99,9% | 43 phút 12 giây | 10 phút 5 giây | 2 giờ 10 phút | **Một** sự cố 43 phút là hết budget tháng |
| 99,95% | 21 phút 36 giây | 5 phút 2 giây | 1 giờ 5 phút | Một sự cố 20 phút là hết |
| 99,99% | 4 phút 19 giây | 1 phút | 13 phút | Không kịp cho một người đăng nhập vào máy và chẩn đoán → buộc phải tự động phục hồi |
| 99,999% | 26 giây | 6 giây | 78 giây | Con người không nằm trong đường phục hồi được nữa |

Dòng 99,99% là dòng quan trọng nhất trong cuộc trò chuyện với lãnh đạo. Câu nói hiệu quả: *"99,99% có
nghĩa là 4 phút mỗi tháng. Trong 4 phút, người on-call chưa kịp mở laptop. Cam kết mức đó tương đương
cam kết rằng mọi sự cố phải được hệ thống tự xử lý không cần con người — đó là một khoản đầu tư hạ tầng
và tổ chức, không phải một dòng trên slide."*

**Mô hình ba: SLO như một hợp đồng ba bên.** SLO không phải tài liệu kỹ thuật; nó là thoả thuận giữa
engineering (người thực hiện), product (người đánh đổi tốc độ), và business (người chịu hậu quả thương
mại). Nếu SLO được viết chỉ bởi engineering, nó sẽ không được tôn trọng khi có xung đột ưu tiên. Phép
thử: PO của bạn có biết SLO của luồng chính không, và có biết tháng này còn bao nhiêu budget không? Nếu
không, bạn có một chỉ số, không có một hợp đồng.

### Practical Framework

**Bước 1 — Liệt kê các luồng người dùng quan trọng (critical user journeys), không phải service.** Với
một ví điện tử, danh sách này thường có 5–8 mục: đăng nhập, nạp tiền, thanh toán QR, thanh toán thẻ,
chuyển tiền, rút tiền, xem lịch sử giao dịch. Đây là đơn vị đúng để đặt SLO vì nó là đơn vị mà người
dùng cảm nhận và business quan tâm.

**Bước 2 — Với mỗi luồng, chọn 1–2 SLI theo công thức chuẩn.** Công thức: `SLI = (số sự kiện tốt) /
(số sự kiện hợp lệ)`, trong đó "tốt" phải được định nghĩa bằng ngưỡng cụ thể. Ba loại SLI phủ gần hết
nhu cầu: availability (tỉ lệ request không lỗi hệ thống), latency (tỉ lệ request nhanh hơn ngưỡng),
correctness/freshness (tỉ lệ dữ liệu đúng hoặc mới hơn ngưỡng). Hai quy tắc quan trọng khi định nghĩa
"sự kiện hợp lệ": loại lỗi do phía người dùng (mã 4xx do nhập sai) khỏi tử số lỗi, nhưng **không** loại
những lỗi mà bạn gây ra rồi trả về 4xx; và đo càng gần người dùng càng tốt (ở API gateway hoặc từ client
telemetry, không phải ở tầng trong).

**Bước 3 — Đặt SLO dựa trên nhu cầu kinh doanh, khởi điểm từ dữ liệu lịch sử.** Quy trình thực dụng ba
câu hỏi: (a) hiện tại chúng ta đang đạt bao nhiêu (đo 90 ngày qua)? (b) ở mức nào thì người dùng bắt
đầu phàn nàn hoặc rời đi (dữ liệu Support, tỉ lệ bỏ giỏ hàng theo latency)? (c) chúng ta có cam kết hợp
đồng nào không? SLO ban đầu nên đặt **hơi thấp hơn mức đang đạt** — đủ để có budget thật để chi, không
phải một mục tiêu đã cháy từ ngày đầu. Đặt SLO cao hơn mức đang đạt là cách nhanh nhất khiến cả tổ chức
bỏ qua nó.

**BẢNG ví dụ SLO/SLI cho ba loại hệ thống (số minh hoạ).**

| | Hệ thống thanh toán (fintech) | Feed nội dung (mạng xã hội / e-commerce) | Báo cáo nội bộ (BI) |
|---|---|---|---|
| **Luồng** | Thanh toán QR từ lúc nhấn xác nhận tới lúc nhận kết quả | Mở trang chủ và thấy feed | Xuất báo cáo doanh thu ngày |
| **SLI availability** | tỉ lệ giao dịch nhận được kết quả xác định (thành công hoặc lỗi rõ ràng, không treo) / tổng giao dịch | tỉ lệ request feed trả 200 với ≥1 item / tổng request | tỉ lệ lần xuất báo cáo hoàn tất / tổng lần xuất |
| **SLO availability** | 99,95% (rolling 28 ngày) | 99,9% | 99,0% |
| **SLI latency** | tỉ lệ giao dịch có kết quả trong <3 giây | tỉ lệ request feed <800ms | tỉ lệ báo cáo xong trong <5 phút |
| **SLO latency** | 99,0% dưới 3 giây | 95,0% dưới 800ms | 90,0% dưới 5 phút |
| **SLI đặc thù** | Correctness: tỉ lệ giao dịch đối soát khớp trong 15 phút — **SLO 99,999%** | Freshness: tỉ lệ feed chứa nội dung mới hơn 10 phút — SLO 95% | Freshness: dữ liệu tới 23:59 hôm trước có trước 07:00 — SLO 99% các ngày làm việc |
| **Error budget/tháng** | 21 phút 36 giây (availability) | 43 phút 12 giây | 7 giờ 12 phút |
| **Lý do đặt mức này** | Tiền: một giao dịch treo tạo cuộc gọi tổng đài và rủi ro đối soát; nhưng 99,99% đòi tự động phục hồi hoàn toàn, chi phí chưa tương xứng ở giai đoạn hiện tại. Riêng correctness đặt cực chặt vì sai số dư là lỗi không hồi được về mặt niềm tin và pháp lý | Người dùng chịu được feed chậm hoặc thiếu vài phút; điều họ không chịu được là trang trắng. Nên availability chặt hơn freshness | Người dùng là nhân viên nội bộ, có thể chờ và có kênh hỏi lại. Downtime 4 giờ ban đêm không ai biết |
| **Chi phí của việc siết thêm một mức** | Cần đa vùng cho tầng dữ liệu + diễn tập chuyển vùng: ước tính 3–4 tháng công + tăng chi phí hạ tầng ~40% | Cần cache nhiều tầng và degradation graceful: ~1 tháng công | Không đáng làm |

Chú ý dòng "SLI đặc thù" của hệ thống thanh toán: **không phải mọi SLI của một hệ thống đều cùng mức.**
Một hệ thống có thể chấp nhận availability 99,95% nhưng đòi correctness 99,999%, vì hai loại lỗi có
tính hồi phục hoàn toàn khác nhau. Việc phân biệt được điều này là dấu hiệu tổ chức đã hiểu SLO thay
vì đang sao chép nó.

**Bước 4 — Tính error budget và theo dõi mức tiêu.** Công thức: `budget = (1 − SLO) × tổng số sự kiện
trong cửa sổ`. Dùng cửa sổ **rolling 28 ngày** thay vì theo tháng lịch, vì cửa sổ theo tháng tạo hiệu
ứng "reset ngày 1" khiến người ta ship rủi ro vào đầu tháng. Hai thứ cần trên dashboard: tỉ lệ budget
đã tiêu, và **tốc độ tiêu (burn rate)** — tức budget đang bị tiêu nhanh gấp mấy lần mức đều. Burn rate
là chỉ số dùng để alert: một sự cố tiêu 5% budget trong 1 giờ (burn rate ~36 lần) đáng đánh thức người;
một sự cố tiêu 5% trong một tuần thì không.

**Bước 5 — Chính sách khi cháy budget, viết trước và ký trước.** Phần này là phần biến SLO thành công
cụ quản trị. Bảng dưới là mẫu; điều quan trọng là nó được PO và lãnh đạo đồng ý **trước khi** cháy, vì
đàm phán lúc đang cháy thì bên có áp lực thị trường luôn thắng.

| Mức tiêu budget (rolling 28 ngày) | Hành động bắt buộc | Ai quyết ngoại lệ |
|---|---|---|
| <50% | Bình thường. Nếu duy trì <30% trong 3 chu kỳ liên tiếp → xem xét siết SLO hoặc tăng tốc ship | Team |
| 50–75% | Cảnh báo. Mọi deploy có rủi ro cao phải có canary và kế hoạch rollback ghi rõ | Tech Lead |
| 75–100% | Dừng nhận thay đổi mới không phải bug fix hoặc reliability; toàn bộ capacity chuyển sang giảm rủi ro; incident review khẩn | EM + PO cùng ký |
| >100% (đã cháy) | Feature freeze cho luồng liên quan tới khi budget hồi về dưới 75%; bắt buộc một reliability sprint; báo cáo lên tầng trên với danh sách nguyên nhân | Chỉ CTO/VP được cho ngoại lệ, và ngoại lệ phải ghi vào tài liệu kèm rủi ro chấp nhận |

Hai chi tiết làm bảng này vận hành được trong thực tế: (a) **feature freeze phải ở phạm vi luồng, không
phải toàn công ty** — đóng băng cả tổ chức vì một service cháy budget là một hình phạt tập thể và sẽ bị
phá; (b) **ngoại lệ được phép tồn tại nhưng phải có tên người và phải ghi lại** — nếu không cho phép
ngoại lệ, chính sách sẽ bị vô hiệu ngầm, còn nếu ngoại lệ không cần ghi thì nó thành mặc định.

**Bước 6 — Review SLO hàng quý.** Ba câu hỏi cho mỗi SLO: (a) mức thực tế đạt được so với mục tiêu —
lệch hai chiều đều là vấn đề; (b) SLO này có tương ứng với điều người dùng thật sự quan tâm không, kiểm
chứng bằng dữ liệu Support và hành vi (tỉ lệ bỏ giỏ, tỉ lệ retry của client); (c) chi phí duy trì mức
này là bao nhiêu và có đáng không. Đầu ra của review phải là một quyết định: giữ, siết, nới, hoặc bỏ
SLO đó. Một SLO không bao giờ được thay đổi trong 2 năm gần như chắc chắn là một SLO không ai dùng.

### Trade-off

| Cặp đối lập | Nghiêng về bên A khi | Nghiêng về bên B khi | Ai trả giá |
|---|---|---|---|
| **SLO cao cho mọi thứ (A) vs phân tầng theo giá trị (B)** | Hầu như không bao giờ hợp lý; chỉ đúng khi mọi hệ thống của bạn đều nằm trên đường tiền | Gần như luôn: hệ thống có mức quan trọng khác nhau | A: tiêu tài nguyên vào chỗ không tạo giá trị, và làm loãng sự chú ý — khi mọi thứ đều tối quan trọng thì không có gì tối quan trọng. B: phải chấp nhận rằng một số hệ thống sẽ hỏng lâu hơn, và phải bảo vệ lựa chọn đó trước người sở hữu chúng |
| **Error budget dùng làm cơ chế cứng (A) vs chỉ số tham khảo (B)** | Tổ chức đã đủ trưởng thành về dữ liệu; lãnh đạo đã ký chính sách; SLI đo đúng | Đang giai đoạn thử nghiệm SLO; SLI còn nhiễu; hoặc chưa có sự đồng thuận của product | A: nếu SLI đo sai, bạn sẽ đóng băng feature vì một lỗi đo lường — mất uy tín cho cả cơ chế. B: không có hậu quả nên không đổi hành vi; SLO thành trang trí |
| **Đo SLI từ phía server (A) vs từ phía client (B)** | Rẻ, có ngay, dữ liệu sạch, đủ để bắt đầu | Cần biết trải nghiệm thật, đặc biệt với ứng dụng di động ở thị trường có mạng không ổn định | A: bỏ sót lỗi mạng, lỗi client, lỗi CDN — bạn có thể đạt SLO trong khi người dùng đang không dùng được. B: đắt hơn, dữ liệu nhiễu vì lẫn vấn đề thiết bị và mạng của người dùng |
| **SLO chặt để tạo áp lực chất lượng (A) vs SLO thực tế đạt được (B)** | Muốn dùng SLO như một mục tiêu tham vọng | Muốn dùng SLO như một cơ chế quyết định | A: cháy budget liên tục → chính sách bị bỏ qua → cả cơ chế mất hiệu lực. B: có thể bị hiểu là hạ chuẩn |

### Real-world Scenarios

**Tình huống 1 — Error budget kết thúc một cuộc tranh luận kéo dài hai quý.** Ví điện tử ở đầu chương,
sáu tháng sau khi đưa SLO vào. Cùng cuộc trò chuyện ở Problem Statement, phiên bản mới:

```text
PO (Hà):   Sprint sau mình cần thanh toán trả góp, campaign 15/09 đã chốt.
Lead (Linh): Được. Nhưng em báo trước trạng thái: luồng thanh toán thẻ đã tiêu 78%
             error budget trong 28 ngày qua — chủ yếu do sự cố 28/07 và hai lần
             timeline cổng đối tác. Theo chính sách mình đã ký hồi tháng 5, ở mức
             trên 75% thì mình dừng nhận thay đổi mới cho luồng đó.
Hà:        Vậy campaign thì sao?
Linh:      Ba phương án. Một: làm trả góp như một luồng riêng, không chạm vào
           đường đi hiện tại của thanh toán thẻ — thêm 3 ngày, không tiêu budget
           của luồng cũ. Hai: làm trong luồng hiện tại, nhanh hơn 3 ngày, nhưng
           mình cần anh Sơn ký ngoại lệ và ghi rõ rủi ro. Ba: hoãn campaign 2 tuần.
Hà:        Phương án một mất 3 ngày thôi à? Vậy chọn một. Nhưng sau campaign
           mình phải xử lý cái cổng đối tác đó, tháng nào cũng ăn budget.
Linh:      Em đồng ý. Em sẽ đưa số liệu về cổng đối tác vào review quý: nó chiếm
           41% tổng budget tiêu trong 3 tháng.
```

So sánh hai đoạn hội thoại là toàn bộ giá trị của chủ đề này. Không ai phải thuyết phục ai về giá trị
của chất lượng. Cuộc trò chuyện chuyển sang phương án và chi phí — và quan trọng hơn, nó sinh ra một
phát hiện ở tầng cao hơn (cổng đối tác chiếm 41% budget) mà cuộc tranh luận cũ không bao giờ sinh ra
được.

**Tình huống 2 — Startup đặt 99,99% và tự bắn vào chân.** Một startup Series A ở TP.HCM, sản phẩm B2B
SaaS, 12 engineer. Trong hồ sơ bán hàng cho một khách lớn, phần SLA ghi 99,99%. Không ai trong
engineering được hỏi. Sáu tháng sau: hệ thống thực tế đạt khoảng 99,4%; mỗi tháng có 2–3 lần vi phạm
SLA hợp đồng; team dựng thêm hai môi trường, thuê thêm hạ tầng dự phòng, và dành khoảng 40% capacity
cho reliability — trong khi sản phẩm vẫn chưa có đủ tính năng để cạnh tranh.

Cách gỡ, và đây là một tình huống gỡ được: (a) đo và trình bày mức thực tế cùng bảng quy đổi số 9 cho
ban lãnh đạo; (b) tách SLA hợp đồng thành nhiều mức theo luồng — 99,9% cho luồng đọc dữ liệu, 99,5%
cho luồng báo cáo, và ghi rõ các loại downtime được loại trừ (bảo trì có thông báo trước, sự cố của nhà
cung cấp hạ tầng); (c) đàm phán lại với khách, đổi mức SLA thấp hơn lấy một cam kết khác có giá trị với
khách nhưng rẻ hơn với mình — ở trường hợp này là thời gian phản hồi sự cố và một báo cáo minh bạch
hàng tháng. Bài học: **con số SLA thường được viết bởi người không biết chi phí của nó**, và việc của
lead kỹ thuật là làm cho chi phí đó hiện hình trước khi ký, không phải sau.

**Tình huống 3 — Đúng SLO trên dashboard, sai trải nghiệm người dùng.** Một e-commerce đo SLI
availability ở tầng API gateway và luôn đạt 99,95%. Nhưng dữ liệu từ Support cho thấy lượng phản hồi
"app treo khi thanh toán" tăng đều. Nguyên nhân: app di động gọi qua một CDN có vấn đề với một nhà mạng
cụ thể ở một số tỉnh; request không bao giờ tới gateway nên không xuất hiện trong SLI.

Bài học có tính nguyên lý: **SLI đo ở tầng nào thì chỉ bảo vệ được từ tầng đó vào trong.** Cách sửa
không nhất thiết đắt: thêm một SLI phụ đo từ client — tỉ lệ phiên thanh toán mà app nhận được phản hồi
trong 5 giây, gửi về qua telemetry — và một job tổng hợp theo nhà mạng và tỉnh. Với thị trường Việt
Nam, nơi chất lượng mạng di động khác nhau đáng kể theo nhà mạng và khu vực, đây là loại SLI đáng đầu
tư sớm hơn hầu hết các nơi khác.

### Best Practices

- **Bắt đầu bằng đúng một SLO cho luồng quan trọng nhất, và chạy nó ba tháng trước khi thêm cái thứ
  hai.** Lý do: cái khó không phải định nghĩa SLO mà là dựng cơ chế đo tin cậy và tập cho tổ chức thói
  quen dùng nó. Mười SLO không ai dùng thì tệ hơn một SLO được dùng.
- **Đặt SLO hơi thấp hơn mức đang đạt được.** Bạn cần error budget thật để chi. Một SLO đã cháy từ ngày
  đầu không tạo ra quyết định nào, nó chỉ tạo ra sự bất lực.
- **Viết chính sách error budget và lấy ký của product và lãnh đạo trước khi cần dùng.** Đây là công
  việc chính trị và nó phải làm lúc bình yên. Trong lúc cháy budget, mọi lập luận của bạn sẽ bị đọc như
  cái cớ để không làm feature.
- **Dùng burn rate để alert, dùng mức tiêu tích lũy để ra quyết định ưu tiên.** Hai chỉ số cho hai mục
  đích khác nhau; trộn chúng cho ra alert sai và quyết định chậm.
- **Đặt correctness SLO cho hệ thống liên quan tiền và dữ liệu, không chỉ availability.** Lỗi tính toán
  sai âm thầm gây thiệt hại lớn hơn nhiều so với downtime, và nó không xuất hiện trên bất kỳ đồ thị
  uptime nào.
- **Trình bày SLO cho lãnh đạo bằng bảng quy đổi phút, không bằng phần trăm.** "43 phút mỗi tháng" tạo
  ra một cuộc trò chuyện đúng; "99,9%" không.
- **Ghi rõ những gì được loại trừ khỏi SLI, và ghi trước.** Bảo trì có thông báo, sự cố của nhà cung
  cấp, lỗi do người dùng nhập sai. Nếu không định nghĩa trước, bạn sẽ tranh luận về nó lúc đang tính
  budget, và cuộc tranh luận đó không có kết thúc.

### Anti-patterns

- **SLO 99,99% cho mọi service.** Cơ chế: nó tương đương không có SLO, vì không có sự phân biệt nào để
  hướng dẫn phân bổ tài nguyên; đồng thời nó cháy liên tục nên mọi người học cách bỏ qua. Dấu hiệu sớm:
  cùng một con số xuất hiện ở mọi service trong tài liệu; không ai kiểm tra được nó đang được đo thế nào.
- **SLI đo tài nguyên hạ tầng thay vì trải nghiệm.** Dấu hiệu sớm: SLO của bạn nói về CPU, memory, số
  pod chạy, hoặc "uptime của server". Phép thử: nếu SLI của bạn có thể xanh trong khi người dùng không
  thanh toán được, nó là SLI sai.
- **Error budget không có chính sách hậu quả.** Cơ chế: một chỉ số không gắn hậu quả không đổi hành vi;
  nó chỉ thêm một dashboard. Dấu hiệu sớm: budget đã cháy hai tháng liên tiếp và lịch phát hành không
  đổi gì.
- **Dùng error budget để phạt team.** Cơ chế: khi cháy budget dẫn tới hậu quả cá nhân, team sẽ tối ưu
  chỉ số thay vì hệ thống — định nghĩa lại "sự kiện hợp lệ" để loại bớt lỗi, đổi ngưỡng latency, hoặc
  không ghi nhận một số incident. Bạn sẽ mất luôn khả năng đo. Dấu hiệu sớm: có tranh luận về việc một
  sự cố "có tính vào budget hay không" nhiều hơn tranh luận về nguyên nhân của nó.
- **Đặt SLO mà không có ai sở hữu.** Dấu hiệu sớm: khi budget cháy, không rõ ai phải quyết định gì.
- **Chỉ siết, không bao giờ nới.** Cơ chế: quy trình tích lũy một chiều, và sau vài năm mọi thay đổi
  nhỏ ở mọi hệ thống đều phải đi qua một bộ kiểm soát được thiết kế cho hệ thống quan trọng nhất. Dấu
  hiệu sớm: trong các quý gần nhất, không có SLO nào được nới hoặc bỏ.

### Khi nào KHÔNG nên áp dụng

- **Sản phẩm chưa có product-market fit.** Trước PMF, câu hỏi tồn tại là "có ai cần cái này không", và
  hệ thống có thể sẽ bị viết lại hoặc bỏ trong vài tháng. Đặt SLO và chính sách error budget lúc này
  làm hai điều xấu: tiêu thời gian của team nhỏ vào cơ chế đo, và tạo ra một lực cản đối với việc thay
  đổi nhanh — chính năng lực bạn cần nhất ở giai đoạn đó. Việc nên làm thay thế, rẻ hơn nhiều: một alert
  cho luồng đăng ký và luồng thanh toán, và một người xem nó.
- **Hệ thống nội bộ ít người dùng, có kênh liên lạc trực tiếp.** Một công cụ nội bộ 30 người dùng thì
  cơ chế feedback tốt nhất là chính họ nhắn tin cho bạn. Dựng SLI, SLO, budget cho nó là xây một hệ
  thống đo phức tạp hơn đối tượng được đo. Ngoại lệ: nếu hệ thống nội bộ đó nằm trên đường tiền (ví dụ
  công cụ xử lý hoàn tiền của Support), nó không còn là hệ thống nội bộ về mặt rủi ro.
- **Chưa có observability để đo SLI đáng tin.** Nếu bạn không thể trả lời "tỉ lệ giao dịch thành công
  hôm qua là bao nhiêu" trong 5 phút, thì việc công bố một SLO là công bố một con số bạn không biết. Nó
  gây hại thật: khi cháy budget, cuộc tranh luận sẽ về chất lượng dữ liệu, và cơ chế mất uy tín ngay ở
  lần dùng đầu tiên. Thứ tự đúng: đo trước 60–90 ngày, xem phân bố, rồi mới đặt mục tiêu.
- **Khi tổ chức không có quyền quyết định phạm vi công việc.** Trong nhiều ODC, scope và deadline do
  khách quyết, và team không có thẩm quyền dừng feature. Error budget với chính sách feature freeze là
  một cơ chế đòi thẩm quyền mà bạn không có, và việc công bố nó rồi không thực thi được sẽ làm mất uy
  tín. Phiên bản dùng được trong bối cảnh này: đo SLI, báo cáo mức tiêu budget cho khách hàng hàng
  tháng như một dữ kiện, và dùng nó làm cơ sở đàm phán scope — không dùng nó như một cơ chế phong toả.
- **Cho những hệ thống mà lỗi có tính rời rạc và nghiêm trọng thay vì có tính tỉ lệ.** Với một hệ thống
  mà một lần sai là không thể chấp nhận — ví dụ tính toán lãi suất, chuyển tiền liên ngân hàng — thì
  ngôn ngữ "99,9% đúng" không phù hợp, vì 0,1% sai không phải một khoản ngân sách được phép chi. Ở đó,
  cơ chế đúng là kiểm chứng, đối soát và bất biến (invariant), không phải error budget. Đừng cố nhét mọi
  loại rủi ro vào một khung.

---

## 6. Engineering Metrics: DORA và SPACE

### Problem Statement

Một CTO hỏi trong họp ban lãnh đạo: "Engineering team 40 người, chi phí mỗi tháng bằng này, làm sao tôi
biết chúng ta đang tốt lên?" Ba câu trả lời thường gặp, và cả ba đều không dùng được:

- **"Team đang làm rất nhiều, tháng này chúng ta đóng 87 ticket."** Số ticket đóng không nói gì về giá
  trị hay tốc độ dòng chảy; nó nói về cách team chia ticket. Một team chia nhỏ ticket sẽ trông năng suất
  gấp đôi một team chia lớn, với cùng lượng công việc.
- **"Velocity của các team là 42, 38, 55 và 29 điểm."** Bốn con số này không so sánh được với nhau vì
  story point là đơn vị nội bộ của từng team, được hiệu chuẩn theo cách team đó ước lượng. Đặt chúng
  cạnh nhau trên một slide là một lỗi loại đơn vị, và nó gây thiệt hại thật — xem mục Anti-patterns.
- **"Hãy tin tôi, chúng ta đang tốt hơn nhiều so với năm ngoái."** Có thể đúng, và không kiểm chứng
  được. Trong dài hạn, một tổ chức engineering không đo được sẽ mất quyền tự chủ về ngân sách, vì bên
  duy nhất có số liệu là bên tài chính.

Ba hiện tượng cụ thể của việc thiếu hệ đo dòng chảy:

**Hiện tượng một: không ai biết một thay đổi mất bao lâu để đi từ ý tưởng tới người dùng, và các bên
đoán khác nhau hàng bậc.** Hỏi riêng ba người: PO nói "khoảng 3 ngày", Tech Lead nói "một tuần", và dữ
liệu git nói trung vị 11 ngày với phân vị 90 là 34 ngày. Khoảng cách giữa cảm nhận và dữ liệu là chỗ
mọi cam kết roadmap đi chết.

**Hiện tượng hai: mọi cải tiến quy trình đều được đánh giá bằng cảm nhận.** Sau khi đưa vào một cơ chế
mới — pipeline mới, quy trình review mới, cách chia team mới — câu hỏi "nó có hiệu quả không" được trả
lời bằng khảo sát cảm giác. Nếu bạn không có đường cơ sở (baseline), bạn không thể biết, và bạn sẽ giữ
lại cả những cơ chế đang gây hại.

**Hiện tượng ba: tổ chức bận nhưng không nhanh hơn.** Đây là hiện tượng trung tâm của chủ đề. Số người
tăng, số ticket tăng, số giờ họp tăng, và lead time không giảm. Không có hệ đo dòng chảy, sự tăng
trưởng của hoạt động bị đọc như tăng trưởng của kết quả.

### First Principles

**Cơ chế một: mọi metric là proxy, và mọi proxy bị bóp méo khi được dùng để đánh giá con người
(Goodhart's Law).** Phát biểu gốc, được lưu hành rộng rãi dưới nhiều dạng: khi một chỉ số trở thành mục
tiêu, nó ngừng là một chỉ số tốt. Cơ chế thì rất cụ thể: một proxy luôn có khoảng cách với đại lượng
thật mà bạn quan tâm, và khoảng cách đó là không gian để tối ưu chỉ số mà không tạo giá trị. Khoảng cách
này không phải là biểu hiện của sự thiếu trung thực — nó là kết quả tất yếu của việc con người phản ứng
với incentive. Nên câu hỏi thiết kế đúng không phải "làm sao tìm metric không bị game" (không có), mà
là **"metric này bị game thì hệ quả có tệ không, và có metric đối trọng nào không"**.

**Cơ chế hai: metric tốt đo dòng chảy của hệ thống, không đo sản lượng cá nhân.** Có ba lý do:

1. **Lý thuyết hệ thống.** Trong một hệ thống có phụ thuộc, sản lượng tổng bị quyết định bởi ràng buộc
   (Theory of Constraints), không bởi tổng sản lượng của các nút. Tăng sản lượng của một nút không phải
   ràng buộc thì chỉ làm tăng hàng tồn giữa các bước. Đo cá nhân là đo sản lượng nút.
2. **Phần lớn công việc giá trị cao nhất không thuộc về một người.** Một senior dành ba ngày để giúp
   hai người khác gỡ một bài toán kiến trúc sẽ trông tệ trên mọi metric cá nhân. Nếu bạn đo cá nhân, bạn
   đang định giá âm chính hành vi bạn cần nhất.
3. **Metric cá nhân phá cộng tác vì nó biến việc giúp người khác thành chi phí.** Đây là cơ chế trực
   tiếp và nhanh: trong vòng vài tuần sau khi công bố metric cá nhân, hành vi review kỹ cho người khác,
   pair, và viết tài liệu sẽ giảm.

**Cơ chế ba: các chỉ số dòng chảy phải đọc theo cặp, vì mỗi cái đơn lẻ có thể được cải thiện bằng cách
làm hại cái khác.** Deployment frequency tăng bằng cách bỏ test. Change failure rate giảm bằng cách
không deploy. Lead time giảm bằng cách bỏ review. Mỗi chỉ số đơn lẻ là một hướng tối ưu có lối tắt; các
cặp chỉ số chặn lối tắt của nhau. Đây là lý do bốn chỉ số DORA được thiết kế thành hai cặp đối trọng:
tốc độ (deployment frequency, lead time) và ổn định (change failure rate, time to restore). Nghiên cứu
DORA công khai của DevOps Research and Assessment nhiều năm liền báo cáo một kết quả đáng chú ý: hai
nhóm này **không** đánh đổi nhau ở mức tổ chức — các tổ chức làm tốt về tốc độ cũng thường tốt về ổn
định. Điều này phản trực giác nhưng có cơ chế: cả bốn đều là hệ quả của cùng một tập năng lực nền
(triển khai tự động, kiểm thử tự động, thay đổi nhỏ, khả năng rollback, observability).

**Cơ chế bốn: sự tồn tại của một metric làm thay đổi hệ thống được đo (reflexivity).** Bạn không thể đo
một tổ chức mà không tác động vào nó. Suy ra một nguyên tắc thiết kế: hãy quyết định trước bạn muốn
metric này thay đổi hành vi theo hướng nào, và kiểm tra xem hướng đó có phải hướng bạn muốn. "Time to
restore" khuyến khích khả năng rollback nhanh — tốt. "Số ticket đóng" khuyến khích chia ticket nhỏ và
chọn việc dễ — không tốt.

### Mental Model

**Mô hình một: bốn chỉ số DORA như hai trục.** Vẽ tốc độ trên một trục, ổn định trên trục kia; bốn góc
là bốn loại tổ chức, và mỗi góc có một chẩn đoán khác nhau:

```
             Ổn định cao
                  |
   Chậm & chắc    |    Nhanh & chắc
   (quy trình     |    (mục tiêu)
    nặng)         |
   ---------------+---------------  Tốc độ
   Chậm & hỏng    |    Nhanh & hỏng
   (khủng hoảng   |    (thiếu lớp
    hệ thống)     |     an toàn)
                  |
             Ổn định thấp
```

Giá trị của mô hình này là nó biến bốn con số thành **một chẩn đoán**, và mỗi góc có một can thiệp khác
nhau. Góc "chậm và chắc" cần bỏ bớt kiểm soát; góc "nhanh và hỏng" cần thêm lớp an toàn tự động; góc
"chậm và hỏng" thì đừng chạm vào metric, hãy đi tìm ràng buộc thật.

**Mô hình hai: SPACE như bộ đối trọng đa chiều.** SPACE là một khung công khai (Forsgren và cộng sự)
gồm năm chiều, và giá trị chính của nó không phải là một bộ chỉ số mà là một **lời nhắc rằng đo một
chiều thì luôn bóp méo**:

| Chiều SPACE | Nó đo gì | Chỉ số dùng được (minh hoạ) | Nó chặn sự bóp méo nào |
|---|---|---|---|
| **S — Satisfaction & well-being** | Mức độ hài lòng, burnout | Khảo sát quý 5 câu; số lần bị đánh thức mỗi phiên on-call; tỉ lệ nghỉ việc tự nguyện | Chặn việc tăng throughput bằng cách đốt người |
| **P — Performance** | Kết quả đầu ra, không phải hoạt động | Change failure rate; tỉ lệ tính năng đạt mục tiêu kinh doanh đã nêu; số incident do thay đổi | Chặn việc làm nhiều mà không có kết quả |
| **A — Activity** | Lượng hoạt động — chiều dễ đo nhất và dễ lạm dụng nhất | Deployment frequency; số PR mở/merge (chỉ ở mức team, chỉ đọc như tín hiệu) | (Chính chiều này cần bị đối trọng) |
| **C — Communication & collaboration** | Chất lượng phối hợp | Thời gian chờ review (p50, p90); số người có thể review một vùng code; độ phân tán kiến thức | Chặn việc tăng tốc bằng cách bỏ review, và phát hiện knowledge silo |
| **E — Efficiency & flow** | Dòng chảy, thời gian chờ | Lead time for changes; tỉ lệ thời gian chờ so với thời gian làm việc thật; số lần bị ngắt | Chặn việc tối ưu cục bộ tại một bước |

Nguyên tắc dùng SPACE cực kỳ đơn giản và hay bị bỏ: **chọn ít nhất ba chiều, và bắt buộc có S.** Nếu bạn
chỉ đo A và P, bạn sẽ tối ưu tổ chức về phía đốt người, và bạn sẽ không thấy nó trong sáu tháng đầu.

**Mô hình ba: metric như đèn báo trên bảng điều khiển, không phải bảng điểm.** Đèn báo dùng để phát hiện
"có gì đang khác thường, đi xem chỗ đó"; bảng điểm dùng để phán xét. Cùng một con số dùng theo hai cách
cho ra hai tổ chức khác nhau. Phép thử nhanh cho tổ chức của bạn: khi một chỉ số xấu đi, phản ứng đầu
tiên của lãnh đạo là "chuyện gì đang xảy ra" hay "team nào đang kém"? Câu trả lời quyết định việc metric
sẽ được cải thiện hay bị game.

### Practical Framework

**A. Bốn chỉ số DORA — định nghĩa và cách đo thực tế từ dữ liệu bạn đã có.**

| Chỉ số | Định nghĩa vận hành | Lấy dữ liệu từ đâu | Bẫy đo lường |
|---|---|---|---|
| **Deployment frequency** | Số lần một thay đổi được đưa lên production thành công, tính theo team và theo service, trong một tuần | CI/CD: đếm số job deploy thành công vào môi trường production. Hoặc git: đếm số tag/release | Đừng đếm deploy vào staging. Đừng đếm một release gộp 40 commit là 1 — nó đúng về kỹ thuật nhưng che mất kích thước batch, nên hãy đo kèm số commit mỗi lần deploy |
| **Lead time for changes** | Thời gian từ commit đầu tiên của một thay đổi tới lúc thay đổi đó chạy trên production | Git (thời điểm commit) + CI/CD (thời điểm deploy chứa commit đó). Dùng trung vị và phân vị 90, **không dùng trung bình** | Nếu team dùng squash merge và feature branch dài, thời điểm "commit đầu tiên" bị mất. Khi đó dùng thời điểm mở PR làm xấp xỉ và ghi rõ định nghĩa |
| **Change failure rate** | Tỉ lệ lần deploy lên production dẫn tới suy giảm dịch vụ cần hành động khắc phục (rollback, hotfix, tắt flag) | Liên kết incident/ticket hotfix với deploy gần nhất trước đó. Cần một quy ước gắn nhãn: mọi PR hotfix có nhãn `hotfix`, mọi rollback ghi vào cùng một nơi | Đây là chỉ số khó đo nhất và dễ bị làm nhẹ. Nếu team tự khai báo, con số sẽ thấp giả. Cách tốt nhất là suy ra từ dữ liệu rollback/hotfix, không từ khai báo |
| **Time to restore service** | Thời gian từ lúc bắt đầu ảnh hưởng người dùng tới lúc dịch vụ hồi phục | Timeline incident (đây là lý do Scribe ở chủ đề 1 có giá trị vượt ra ngoài postmortem) | Đo từ lúc **bắt đầu ảnh hưởng**, không từ lúc phát hiện — nếu đo từ lúc phát hiện, bạn thưởng cho việc phát hiện muộn |

Ba lưu ý về cách triển khai đo, theo thứ tự nên làm: (a) bắt đầu bằng deployment frequency và lead time
vì chúng lấy được hoàn toàn tự động từ git và CI/CD, không cần ai khai báo gì; (b) change failure rate
và time to restore cần một quy ước gắn nhãn — hãy thống nhất quy ước trước và chấp nhận 2–3 tháng dữ
liệu nhiễu; (c) không dùng công cụ thương mại đắt tiền ở giai đoạn đầu — một script đọc git và CI/CD API
cùng một bảng tính là đủ để có ba tháng dữ liệu đầu tiên, và nó buộc bạn hiểu định nghĩa thay vì tin
một hộp đen.

**B. Đọc theo cặp — BẢNG chẩn đoán tổ hợp DORA.** Đây là bảng dùng được nhất trong chủ đề: mỗi tổ hợp
bất thường ứng với một nguyên nhân hệ thống, và một can thiệp khác nhau. Số trong ngoặc là mốc minh
hoạ để bạn biết "cao" và "thấp" nghĩa là gì trong bối cảnh một tổ chức 30–60 engineer.

| # | Tổ hợp | Nguyên nhân hệ thống thường gặp | Can thiệp đúng | Can thiệp sai (thường được chọn) |
|---|---|---|---|---|
| 1 | Lead time **cao** (trung vị >7 ngày) + change failure **thấp** (<5%) | Quy trình duyệt quá nặng: nhiều tầng approval, review chờ lâu, môi trường test tranh chấp, release theo lịch cố định | Đo thời gian chờ ở từng bước; bỏ tầng duyệt không phát hiện lỗi nào trong 6 tháng qua; tăng số người được review mỗi vùng | Yêu cầu team "làm nhanh hơn" — họ sẽ bỏ những bước duy nhất đang giữ failure rate thấp |
| 2 | Deployment frequency **cao** (>10/tuần/team) + change failure **cao** (>25%) | Thiếu lớp an toàn tự động: không có test tự động ở tầng tích hợp, không có canary hoặc progressive rollout, không có kiểm tra tự động sau deploy | Thêm canary + tự động rollback theo chỉ số; thêm test cho đúng các luồng đã hỏng (dựa vào lịch sử incident, không thêm test tràn lan) | Thêm một cửa phê duyệt thủ công — nó giảm frequency và không giảm failure, vì con người không phát hiện được loại lỗi này |
| 3 | Time to restore **cao** (>2 giờ) + change failure **thấp** | Hiếm sự cố nên tổ chức chưa xây năng lực phản ứng: không có runbook, không rollback được, không ai quen quy trình incident, observability yếu | Diễn tập (game day); làm rollback thành hành động một bước; viết runbook cho 5 luồng quan trọng nhất | Kết luận "chúng ta ổn định nên không cần đầu tư" — đây là tổ chức sẽ có một sự cố dài kỷ lục |
| 4 | Lead time **thấp** + change failure **thấp** + deployment frequency **thấp** (<1/tuần) | Batch lớn: mỗi lần deploy là một gói lớn, và lead time thấp chỉ đo phần cuối vì thời gian chờ nằm ngoài phạm vi đo (ví dụ đo từ lúc mở PR) | Kiểm tra lại định nghĩa lead time — đo từ commit đầu, hoặc từ lúc bắt đầu làm; đo kích thước batch (số commit và số dòng thay đổi mỗi deploy) | Ăn mừng vì hai chỉ số đẹp |
| 5 | Deployment frequency **thấp** + time to restore **cao** | Deploy là sự kiện đáng sợ nên bị dồn lại; càng dồn thì mỗi lần càng rủi ro; càng rủi ro thì càng ít deploy — vòng phản hồi dương | Tách nhỏ thay đổi, dùng feature flag để tách deploy khỏi release; đầu tư vào rollback trước khi đầu tư vào bất cứ thứ gì khác | Thêm quy trình kiểm soát cho mỗi lần deploy, làm vòng lặp chặt hơn |
| 6 | Cả bốn chỉ số **tốt** nhưng team báo cáo burnout, tỉ lệ nghỉ việc tăng | Throughput đang được mua bằng người: overtime thường xuyên, on-call nặng, không có thời gian cho công việc không phải feature | Đây là lý do SPACE tồn tại: thêm chiều S. Đo số lần đánh thức, số giờ ngoài giờ, khảo sát quý | Dùng bốn chỉ số đẹp làm bằng chứng rằng "team đang rất tốt" |
| 7 | Lead time trung vị **thấp** nhưng phân vị 90 **rất cao** (p50 = 2 ngày, p90 = 28 ngày) | Có hai loại công việc trộn trong một dòng: việc nhỏ chảy nhanh, việc lớn hoặc việc phụ thuộc bên ngoài bị treo | Tách dòng theo loại việc và đo riêng; đi tìm nguyên nhân treo của nhóm p90 (thường là chờ quyết định, chờ team khác, chờ khách hàng) | Chỉ báo cáo trung vị lên lãnh đạo — bạn sẽ mất chính thông tin quan trọng nhất |
| 8 | Change failure rate **giảm đột ngột** mà không có thay đổi nào về năng lực | Định nghĩa đang bị bóp méo: hotfix không gắn nhãn, sự cố được ghi là "bảo trì", rollback không ghi lại | Kiểm tra chéo bằng nguồn độc lập: số incident, số cuộc gọi Support, số lần rollback trong log CI/CD | Ăn mừng và báo cáo lên |

Nguyên tắc rút ra: **không có chỉ số nào trong bốn cái được đọc một mình.** Nếu bạn chỉ có thể đưa một
thứ lên slide, đưa cặp (lead time p50/p90, change failure rate) và luôn kèm một chiều SPACE.

**C. Cách trình bày metric cho lãnh đạo.** Ba nguyên tắc và một cấu trúc.

Nguyên tắc một: **trình bày xu hướng, không trình bày điểm.** Một con số đơn lẻ mời gọi so sánh và phán
xét; một đường 6 tháng mời gọi câu hỏi về nguyên nhân. Nguyên tắc hai: **luôn kèm chẩn đoán và một
quyết định cần từ lãnh đạo.** Metric không kèm đề nghị hành động sẽ bị lãnh đạo tự diễn giải, và diễn
giải mặc định thường là về con người. Nguyên tắc ba: **nói trước rằng chỉ số này ở cấp team và không
dùng để so sánh team**, mỗi lần, kể cả lần thứ mười.

```text
SLIDE 1 — Dòng chảy giao hàng, 6 tháng (mức tổ chức, không so sánh team)

  Lead time (trung vị):   11,2 ngày -> 6,4 ngày
  Lead time (p90):        34 ngày   -> 29 ngày     <-- chưa cải thiện
  Deploy/tuần:            4,1       -> 12,7
  Change failure rate:    18%       -> 9%
  Time to restore (p50):  3h10      -> 52 phút
  Số lần đánh thức đêm/người/tháng: 6,2 -> 1,8

CHẨN ĐOÁN (một câu): việc chuẩn hoá pipeline và bật canary đã cải thiện phần
dòng chảy bình thường; phần p90 chưa cải thiện vì nó là các việc phụ thuộc
vào phê duyệt của phòng Nghiệp vụ, trung bình chờ 9 ngày.

ĐIỀU CẦN TỪ BAN LÃNH ĐẠO: một quyết định về ngưỡng giá trị dưới đó phòng
Nghiệp vụ không cần phê duyệt từng thay đổi. Hiện tại mọi thay đổi đều cần,
kể cả sửa nhãn hiển thị.

KHÔNG CÓ TRONG SLIDE NÀY: so sánh giữa các team. Các chỉ số này phụ thuộc
loại hệ thống (legacy vs mới) nên so sánh ngang là sai về phương pháp.
```

**D. Quy trình đưa metric vào tổ chức mà không gây phản ứng phòng thủ (8 tuần).**

| Tuần | Việc | Đầu ra |
|---|---|---|
| 1 | Nói với team **trước** khi đo: đo cái gì, để làm gì, và cam kết rõ là không dùng cho performance review cá nhân | Một trang tài liệu, được lead ký |
| 2–3 | Dựng cơ chế đo tự động cho deployment frequency và lead time từ git/CI-CD | Dashboard đầu tiên, chỉ team xem |
| 4 | Thống nhất quy ước gắn nhãn cho hotfix, rollback, incident | Quy ước viết ra, áp dụng từ tuần 5 |
| 5–6 | Team tự xem số của mình, tự đề xuất một cải tiến | Một cải tiến được chọn, có baseline |
| 7 | Thêm một chiều SPACE, tối thiểu là khảo sát 5 câu về well-being và số lần bị đánh thức | Dữ liệu chiều S đầu tiên |
| 8 | Trình bày lên lãnh đạo lần đầu, ở mức tổ chức, kèm chẩn đoán và một đề nghị | Slide theo mẫu ở mục C |

Thứ tự này quan trọng: **team nhìn thấy số của mình trước lãnh đạo.** Nếu lãnh đạo thấy trước, metric
sẽ được hiểu là công cụ giám sát ngay từ đầu, và bạn sẽ không lấy lại được sự tin cậy đó.

### Trade-off

| Cặp đối lập | Nghiêng về bên A khi | Nghiêng về bên B khi | Ai trả giá |
|---|---|---|---|
| **Metric ở cấp team (A) vs cấp cá nhân (B)** | Gần như luôn. Công việc có phụ thuộc; muốn cải thiện hệ thống; muốn giữ cộng tác | Rất hẹp: khi cần bằng chứng cho một vấn đề performance đã được xác định qua quan sát trực tiếp — và ngay cả lúc đó, metric chỉ là dữ liệu bổ trợ, không phải căn cứ chính | A: một người underperform có thể ẩn sau số của team — phải giải quyết bằng quan sát trực tiếp và 1-1, không bằng dashboard. B: game hoá, phá cộng tác, mất người giỏi làm việc khó đo |
| **Minh bạch metric toàn tổ chức (A) vs chỉ trong team (B)** | Đã có văn hoá không quy tội; muốn học chéo; các team có bối cảnh tương đương | Bối cảnh các team khác nhau nhiều (legacy vs greenfield); có tiền lệ dùng số để so sánh và gây áp lực | A: tạo áp lực so sánh không công bằng, và team có hệ thống legacy sẽ luôn trông tệ dù đang làm tốt hơn | B: bài học không lan; mỗi team phát minh lại giải pháp |
| **Nhiều metric để phủ hết (A) vs ít metric được dùng thật (B)** | Đang chẩn đoán một vấn đề cụ thể, cần dữ liệu rộng trong một giai đoạn | Vận hành thường xuyên | A: dashboard 40 chỉ số không ai xem; chi phí bảo trì cơ chế đo cao | B: có thể bỏ sót một chiều — bù bằng review định kỳ chọn lại chỉ số |
| **Đo tự động từ hệ thống (A) vs khai báo thủ công (B)** | Luôn ưu tiên | Chỉ khi không có cách nào khác, và với các chỉ số về cảm nhận (chiều S) | A: giới hạn bởi những gì hệ thống ghi lại; có thể bỏ sót ngữ cảnh | B: dữ liệu bị bóp méo theo hướng có lợi cho người khai; tốn thời gian, và nó sẽ bị bỏ sau vài tháng |

### Real-world Scenarios

**Tình huống 1 — Velocity dùng để so sánh team, và ba tháng sau đó.** Một công ty e-commerce, CTO mới
về, muốn có "khả năng nhìn". Yêu cầu: mọi team báo cáo velocity hàng sprint lên một dashboard chung.
Sáu team, sáu con số cạnh nhau, chiếu trong họp hàng tháng.

Diễn biến trong ba tháng, theo thứ tự xuất hiện: (1) tuần 2, các team bắt đầu ước lượng cao hơn cho cùng
loại công việc — story point lạm phát, và không ai làm gì sai vì point là đơn vị nội bộ; (2) tuần 5, một
team ngừng nhận việc khó ước lượng (điều tra bug lạ, refactor) vì nó không sinh point ổn định; (3) tuần
7, hai team ngừng giúp nhau vì thời gian giúp team khác không tính vào point của mình; (4) tuần 9, một
team chia mọi ticket thành các đơn vị nhỏ nhất có thể để tăng số lượng và độ ổn định; (5) tuần 12,
velocity trung bình toàn tổ chức tăng 40% và lead time thực đo từ git **không đổi**.

Đây là Goodhart's Law trong dạng thuần khiết nhất, và điều đáng chú ý là **không ai gian dối**. Mỗi
hành vi trên đều là phản ứng hợp lý với incentive. Cách gỡ mà tổ chức này đã dùng: giữ story point như
công cụ nội bộ của team cho việc lập kế hoạch sprint và **cấm nó xuất hiện ngoài team**; thay bằng lead
time và deployment frequency đo tự động từ git ở mức tổ chức; và với mỗi team, một cuộc trò chuyện về
chẩn đoán chứ không phải xếp hạng. Mất khoảng hai quý để hành vi hồi phục, và một số thiệt hại về niềm
tin không hồi phục hoàn toàn.

**Tình huống 2 — Cùng một dữ liệu, ba góc nhìn.** Bối cảnh: một ODC ở Hà Nội, khách Singapore. Dữ liệu
quý: lead time p50 = 3 ngày, p90 = 26 ngày; deployment frequency 9/tuần; change failure rate 6%; time
to restore p50 = 40 phút. Trên giấy, đây là một team tốt. Nhưng tỉ lệ nghỉ việc tự nguyện quý này là
hai người trên chín.

| | Người đó thấy gì | Câu hỏi thật của họ | Sai lầm thường gặp |
|---|---|---|---|
| **IC (Vy, BE 2 năm)** | Số đẹp vì cô và một người nữa gánh phần lớn việc khó và phần lớn on-call; p90 = 26 ngày chính là các ticket cô bị treo vì chờ khách trả lời, và trong lúc chờ cô bị gán thêm việc | "Nếu số của team đẹp thì ai sẽ nhìn thấy việc tôi đang không ổn?" | Không nói gì vì thấy số đang tốt nên phàn nàn có vẻ vô lý. Đây là lý do chiều S không thể là tuỳ chọn |
| **Tech Lead (Khoa)** | Bốn chỉ số DORA tốt, và anh biết chúng tốt phần nào nhờ hai người cụ thể. Anh cũng biết p90 = 26 ngày là vấn đề của khách, không phải của team | "Tôi báo cáo số đẹp thì được yên, nhưng tôi đang tiêu vốn con người. Tôi có dám đưa chiều S vào báo cáo không?" | Chỉ báo cáo bốn chỉ số DORA vì chúng đẹp — và mất người vào quý sau. Hoặc ngược lại: báo cáo vấn đề mà không có số, nên bị đọc là kêu khó |
| **Manager (Trang, EM)** | Hai đơn xin nghỉ, một dashboard xanh, và một khách hàng hài lòng. Đây là dấu hiệu điển hình của throughput được mua bằng người | "Tôi có bằng chứng gì để đàm phán lại scope với khách, khi mọi chỉ số giao hàng đều tốt?" | Xử lý bằng cách tuyển thêm — mất 3–5 tháng để người mới đóng góp, trong lúc gánh nặng dồn lên người còn lại. Đúng hơn: dùng chính p90 = 26 ngày và dữ liệu on-call làm cơ sở đàm phán về quy trình phản hồi của khách |

Ba góc nhìn này cho thấy một điều quan trọng về bản chất: **bốn chỉ số DORA đo dòng chảy của công việc,
không đo tình trạng của hệ thống sản xuất ra dòng chảy đó.** Một dây chuyền có thể chạy nhanh trong khi
đang bị quá tải, và metric dòng chảy sẽ không cho bạn biết cho tới khi dây chuyền dừng.

**Tình huống 3 — Lead time cao và cửa phê duyệt không phát hiện gì.** Một bộ phận IT của doanh nghiệp
truyền thống. Lead time trung vị 19 ngày, change failure rate 3%. Đọc theo bảng chẩn đoán, đây là tổ hợp
số 1: quy trình duyệt quá nặng.

Cách điều tra đã làm: dựng bản đồ thời gian chờ cho 40 thay đổi gần nhất, theo từng bước. Kết quả (số
minh hoạ): viết code 1,8 ngày; chờ code review 2,1 ngày; chờ môi trường test 4,3 ngày; chờ QA 3,2 ngày;
chờ phê duyệt của Hội đồng Thay đổi họp thứ Ba hàng tuần 5,1 ngày; deploy 0,4 ngày. Rồi một câu hỏi
duy nhất được đặt ra cho Hội đồng: trong 12 tháng qua, hội đồng đã từ chối hoặc yêu cầu sửa bao nhiêu
thay đổi? Câu trả lời: 3 trên khoảng 400, và cả 3 đều là vấn đề về thời điểm phát hành, không phải về
kỹ thuật.

Đề xuất được chấp nhận: hội đồng chỉ xét các thay đổi thuộc ba loại (ảnh hưởng dữ liệu khách hàng, thay
đổi tích hợp với hệ thống ngân hàng lõi, thay đổi cần thông báo người dùng); phần còn lại đi theo đường
tự động. Lead time trung vị xuống 9 ngày trong hai tháng, change failure rate không đổi. Điểm đáng học
về phương pháp: **lập luận thắng không phải "quy trình của các anh nặng" mà là "cửa này đã bắt được 3
trên 400, và nó đang tiêu 5,1 ngày mỗi thay đổi"**. Với một tổ chức coi trọng kiểm soát, chỉ có dữ liệu
về hiệu lực của chính cơ chế kiểm soát mới mở được cuộc trò chuyện.

### Best Practices

- **Đo ở mức team và mức tổ chức; không bao giờ ở mức cá nhân.** Và nói ra điều này bằng chữ, có chữ ký
  của lead, trước khi bắt đầu đo. Cam kết này là tài sản duy nhất khiến dữ liệu của bạn đáng tin, vì nó
  là điều kiện để người ta không tối ưu chỉ số.
- **Luôn đọc theo cặp và luôn kèm ít nhất một chiều SPACE, tối thiểu là chiều S.** Bốn chỉ số DORA có
  thể đồng thời đẹp trong một tổ chức đang đốt người.
- **Dùng p50 và p90, không dùng trung bình.** Phân bố của lead time lệch phải rất mạnh; trung bình bị
  chi phối bởi vài giá trị lớn và che mất cả hai câu chuyện. Khoảng cách giữa p50 và p90 thường là phát
  hiện giá trị nhất trong toàn bộ dữ liệu.
- **Đo tự động từ git và CI/CD, không dựa vào khai báo.** Mọi chỉ số cần con người nhập sẽ suy giảm chất
  lượng sau 6–8 tuần, và nó suy giảm theo hướng có lợi cho người nhập.
- **Trước mỗi metric mới, viết ra một câu: "nếu ai đó muốn tối ưu chỉ số này mà không tạo giá trị, họ sẽ
  làm gì?"** Nếu câu trả lời dễ và hậu quả tệ, đừng dùng chỉ số đó, hoặc thêm đối trọng.
- **Gắn mỗi báo cáo metric với một chẩn đoán và một đề nghị hành động.** Số không kèm diễn giải sẽ bị
  diễn giải bởi người có quyền nhất trong phòng.
- **Xem lại bộ chỉ số mỗi hai quý và bỏ cái không dẫn tới quyết định nào.** Một chỉ số không đổi được
  quyết định nào trong 6 tháng là chi phí bảo trì thuần.
- **Khi một chỉ số xấu đi, phản ứng đầu tiên phải là một câu hỏi, không phải một chỉ đạo.** Đây là hành
  vi của lead quyết định việc tổ chức sẽ cải thiện hệ thống hay học cách làm đẹp số.

### Anti-patterns

Bốn anti-pattern dưới đây phổ biến đến mức cần phân tích cơ chế bóp méo cụ thể của từng cái, không chỉ
nói là sai.

- **Đo số dòng code (LOC).** Cơ chế bóp méo: LOC thưởng cho việc thêm code và trừng phạt việc xoá code
  — trong khi phần lớn công việc kỹ thuật giá trị cao nhất ở một hệ thống trưởng thành là **giảm** lượng
  code (gộp trùng lặp, xoá tính năng chết, thay 400 dòng tự viết bằng 20 dòng dùng thư viện). Nó cũng
  thưởng cho sao chép thay vì trừu tượng hoá, và cho việc viết dài dòng. Ở cực đoan, một engineer tối ưu
  LOC sẽ làm đúng ngược lại điều bạn cần. Dấu hiệu sớm: có ai đó hỏi "team mình viết bao nhiêu dòng
  tháng này"; công cụ báo cáo LOC được cài đặt.
- **Đo số commit.** Cơ chế bóp méo: số commit là một lựa chọn về phong cách làm việc, không phải một đại
  lượng về công việc. Một người commit mỗi 20 phút và một người commit mỗi ngày có thể làm cùng lượng
  việc. Khi bị đo, hành vi thay đổi trong vài ngày: người ta chia commit nhỏ hơn, thêm commit sửa lỗi
  chính tả, và commit "wip" — chi phí không phải là gian dối mà là **mất giá trị của lịch sử git như một
  tài liệu**. Một hệ quả tệ hơn: nó trừng phạt việc dành hai ngày đọc code và suy nghĩ trước khi viết,
  tức chính hành vi phân biệt senior với junior. Dấu hiệu sớm: contribution graph được nhắc trong bất kỳ
  cuộc trò chuyện nào về hiệu suất.
- **Đo story point per engineer.** Cơ chế bóp méo: story point là đơn vị được **team** hiệu chuẩn để dự
  báo, và giá trị của nó phụ thuộc hoàn toàn vào việc nó không có hậu quả. Khi chia cho từng người, ba
  méo xảy ra đồng thời: (a) lạm phát ước lượng, vì ước lượng cao thì an toàn hơn và không ai kiểm chứng
  được; (b) người ta tránh việc không ước lượng được — điều tra bug, refactor, hỗ trợ người khác, viết
  tài liệu; (c) việc pair programming và mentoring trở thành thiệt hại thuần cho người dạy, nên nó biến
  mất. Đồng thời, chỉ số này phá luôn công dụng gốc: sau khi bị dùng để đánh giá, point không còn dùng
  để dự báo được nữa, vì nó đã mất tính hiệu chuẩn. Bạn mất cả metric lẫn công cụ lập kế hoạch. Dấu hiệu
  sớm: point xuất hiện trong bất kỳ tài liệu nào ngoài phạm vi team.
- **Dùng velocity để so sánh giữa các team.** Cơ chế bóp méo: đây là một lỗi về đơn vị đo trước khi là
  một lỗi quản trị — velocity của team A và team B được tính bằng hai thước khác nhau, nên phép so sánh
  không có nghĩa toán học. Hệ quả hành vi: các team hiệu chuẩn lại thước của mình theo hướng có lợi (số
  liệu tăng, thực tế không đổi); các team ngừng chia sẻ người và ngừng giúp nhau, vì công việc xuyên team
  không thuộc velocity của ai; và các team nhận hệ thống legacy luôn trông tệ hơn, nên không ai muốn nhận
  việc bảo trì hệ thống cũ — dần dần đó trở thành vùng không ai sở hữu. Dấu hiệu sớm: một slide có
  velocity của nhiều team cạnh nhau; hoặc có ai nói "team X hiệu suất cao hơn team Y".

Ba anti-pattern khác cần nêu:

- **Dashboard 40 chỉ số không ai xem.** Cơ chế: chi phí thu thập và bảo trì là thật, giá trị bằng không;
  và sự tồn tại của dashboard tạo cảm giác đã đo lường, làm giảm động lực đo đúng thứ. Dấu hiệu sớm:
  không ai mở dashboard trong 30 ngày qua ngoài người tạo ra nó.
- **Metric bị đưa vào performance review cá nhân sau khi đã cam kết là không.** Cơ chế: đây là hành vi
  phá huỷ một lần, không hồi phục được. Sau lần đó, mọi cam kết về metric của bạn không còn giá trị, và
  dữ liệu bắt đầu bị bóp méo trên tất cả các chỉ số. Dấu hiệu sớm: một manager hỏi bạn "cho tôi xem số
  của bạn X".
- **Đặt mục tiêu số cho chỉ số DORA (OKR kiểu "tăng deployment frequency lên 20/tuần").** Cơ chế: biến
  chỉ số chẩn đoán thành mục tiêu, và nó sẽ đạt được bằng cách chia nhỏ deploy một cách nhân tạo hoặc
  deploy những thay đổi rỗng. Dùng chỉ số DORA làm **kết quả quan sát** của một cải tiến năng lực, không
  làm mục tiêu trực tiếp. Dấu hiệu sớm: có OKR mà key result là một con số DORA.

### Khi nào KHÔNG nên áp dụng

- **Team dưới 5 người và dữ liệu quá nhiễu.** Với 4 engineer và 6 deploy mỗi tuần, phương sai thống kê
  lớn hơn mọi tín hiệu: một người nghỉ phép làm deployment frequency giảm một nửa, và điều đó không nói
  gì về năng lực. Ở quy mô này, cơ chế feedback tốt hơn nhiều là một cuộc trò chuyện 30 phút mỗi hai
  tuần: cái gì đang chậm, cái gì đang khiến chúng ta phải làm lại. Có một chỉ số duy nhất đáng đo ngay
  cả ở quy mô nhỏ vì nó là nhị phân và không nhiễu: **chúng ta có deploy được trong ngày khi cần không,
  và có rollback được trong 10 phút không.**
- **Giai đoạn khủng hoảng.** Khi tổ chức đang trong một sự cố kéo dài, một đợt sa thải, một cuộc di trú
  hệ thống lớn, hoặc mất một phần đáng kể nhân sự, mọi chỉ số sẽ xấu đi vì những nguyên nhân đã biết.
  Đo và báo cáo lúc này không thêm thông tin và tạo ra một tác hại cụ thể: nó buộc lead tiêu năng lượng
  vào việc giải thích số thay vì xử lý khủng hoảng. Việc nên làm: ghi nhận một đường cơ sở trước khủng
  hoảng, tạm dừng báo cáo định kỳ, và nói rõ khi nào sẽ nối lại.
- **Khi tổ chức chưa có sự tin cậy tối thiểu về việc metric sẽ không dùng để đánh giá cá nhân.** Nếu
  tiền lệ trong công ty là mọi số liệu cuối cùng đều đi vào đánh giá, thì việc bạn bắt đầu đo sẽ bị đọc
  đúng như vậy, và hành vi bóp méo bắt đầu trước cả khi bạn có dữ liệu đầu tiên. Thứ tự đúng: đàm phán
  và ghi thành văn bản với cấp trên về phạm vi sử dụng của dữ liệu **trước**, đo **sau**. Nếu không đàm
  phán được, hãy đo cho riêng bạn để chẩn đoán và không công bố — điều đó vẫn có giá trị.
- **Khi bạn chưa có một câu hỏi cụ thể cần trả lời.** Đo vì "nên đo" cho ra dashboard không ai xem. Mỗi
  chỉ số nên bắt đầu từ một câu hỏi có người thật đang cần câu trả lời: "vì sao mọi thứ mất lâu hơn dự
  kiến", "vì sao chúng ta hay phải hotfix", "vì sao team này kiệt sức". Câu hỏi quyết định chỉ số, không
  phải ngược lại.
- **Với công việc nghiên cứu, khám phá, hoặc giai đoạn thiết kế.** Một team đang thử ba hướng kiến trúc
  cho một bài toán mới sẽ có deployment frequency thấp, lead time cao, và điều đó là đúng — công việc
  của họ là giảm bất định, không phải giao hàng. Áp chỉ số dòng chảy lên loại công việc này sẽ đẩy họ về
  phía làm những việc dễ giao hàng, tức phá đúng giá trị bạn mua khi lập team đó.

---

## 7. Learning Organization — biến sự cố thành năng lực

### Problem Statement

Một công ty logistics có 8 incident trong một quý. Cả 8 đều có postmortem viết đúng template, đều
blameless, đều có action item có chủ và có ngày. Tỉ lệ action item đóng đúng hạn là 71% — một con số
mà nhiều tổ chức sẽ tự hào. Đến quý sau, họ có 9 incident.

Khi Hà — EM mới nhận team — đọc lại cả 8 bản, cô thấy điều mà không ai thấy được khi đọc từng bản
riêng lẻ: 6 trong 8 incident có một câu giống nhau nằm ở những vị trí khác nhau trong timeline.
"Config trên môi trường production khác với staging." Sáu lần. Sáu action item khác nhau đã được viết
và đóng: thêm một dòng vào checklist deploy, thêm một cảnh báo vào runbook của service A, viết một
script so sánh config cho service B, thêm một mục vào Definition of Done, tổ chức một buổi chia sẻ về
config management, và thêm một trường vào form deploy request. Không action item nào sai. Không action
item nào chạm vào nguyên nhân: tổ chức không có nguồn sự thật duy nhất cho configuration, và không có
cơ chế nào phát hiện lệch cấu hình trước khi nó gây sự cố.

Đây là dạng thất bại đặc thù của tầng **Improvement**: postmortem hoạt động ở phạm vi một incident,
nên nó tối ưu cục bộ rất tốt và mù hoàn toàn với mẫu chung. Mỗi bản postmortem đều là một điểm dữ
liệu đúng; tập hợp các điểm dữ liệu không tự trở thành đường xu hướng nếu không có ai vẽ nó.

Hậu quả quan sát được khi tổ chức thiếu tầng học ở cấp hệ thống:

- Cùng một loại nguyên nhân gốc tái diễn trong vòng 6 tháng — đây là chỉ báo trực tiếp và đo được.
- Số lượng action item tồn đọng tăng đơn điệu; đến một ngưỡng, không ai còn tin action item sẽ được
  làm, và postmortem lặng lẽ trở thành nghi lễ.
- Runbook mô tả một hệ thống không còn tồn tại, nên người on-call không dùng, nên runbook càng cũ.
- Kiến thức vận hành tập trung ở 1–2 người; khi họ nghỉ, MTTR tăng gấp đôi mà không ai giải thích được.
- Đội ngũ phát triển một loại chủ nghĩa định mệnh: "hệ thống này nó thế", "đến campaign là phải trực".

### First Principles

Một tổ chức học được khi và chỉ khi có đủ ba điều kiện. Thiếu bất kỳ điều nào, hoạt động "rút ra bài
học" vẫn diễn ra nhưng năng lực không tăng.

**Điều kiện 1 — Tín hiệu trung thực.** Nếu việc báo cáo trung thực gây bất lợi cho người báo cáo, dữ
liệu đầu vào bị bóp méo có hệ thống, và mọi phân tích phía sau đều sai theo cùng một hướng. Đây là
lý do Psychological Safety (`03-team-leadership.md`) không phải một giá trị mềm mà là điều kiện tiên
quyết kỹ thuật cho khả năng học. Chương này nằm sau chương đó trong chuỗi nhân quả, không song song
với nó.

**Điều kiện 2 — Vòng feedback ngắn hơn thời gian tồn tại của giả định.** Nếu bạn biết một quyết định
kiến trúc là sai sau 18 tháng, và người ra quyết định đã chuyển team sau 12 tháng, thì hệ thống không
học được — người học được là người khác, còn tổ chức chỉ ghi nhận một kết quả xấu không rõ nguyên
nhân. Việc rút ngắn vòng feedback (deploy nhỏ, canary, SLO đo liên tục, dự đoán kiểm chứng được trong
decision log) không phải để đi nhanh hơn, mà để **quy được kết quả về quyết định**. Không có quy
kết, không có học.

**Điều kiện 3 — Cơ chế biến bài học thành ràng buộc cấu trúc.** Đây là điều kiện bị bỏ nhiều nhất.
Một bài học tồn tại ở ba dạng, với độ bền hoàn toàn khác nhau:

| Dạng lưu bài học | Cơ chế thực thi | Độ bền | Ví dụ |
|---|---|---|---|
| Trong đầu người | Ký ức, ý chí | Vài tuần đến khi người đó rời đi | "Từ giờ nhớ kiểm tra config" |
| Trong văn bản | Người đọc phải chủ động | Đến khi văn bản lỗi thời | Một dòng thêm vào checklist |
| Trong công cụ / kiến trúc | Không tuân thủ thì không chạy được | Đến khi bị cố ý tháo ra | CI fail nếu config staging ≠ production |

Nguyên lý: **một bài học chỉ được coi là đã học khi nó được dịch xuống dạng thấp nhất khả thi trong
bảng trên.** Sáu action item của công ty logistics đều nằm ở dạng thứ nhất và thứ hai. Đó là lý do
lần thứ bảy vẫn đến.

Có một cơ chế thứ tư đáng nêu, đến từ nghiên cứu công khai về an toàn hệ thống phức tạp: **drift into
failure**. Hệ thống không đứng yên ở trạng thái an toàn rồi bị một cú sốc đánh gãy. Nó trôi dần —
mỗi ngày một quyết định nhỏ hợp lý cục bộ (tắt một alert ồn, bỏ qua một test flaky, deploy tay một
lần cho nhanh) đẩy hệ thống gần hơn tới biên. Incident là lúc biên bị vượt, không phải lúc vấn đề
bắt đầu. Hệ quả quản trị: nếu bạn chỉ học từ incident, bạn học từ 5% các tín hiệu mà hệ thống đã phát
ra. 95% còn lại nằm trong các near-miss, các "may mà phát hiện kịp", các workaround không ai ghi lại.

### Mental Model

**Đơn lẻ vs Tổng hợp — hai vòng học khác nhau.** Hình dung hai vòng lồng nhau:

```
Vòng trong  (mỗi incident, chu kỳ ngày)
  Incident → Postmortem → Action item → Sửa lỗi cụ thể
  Câu hỏi:  "Vì sao lần này hỏng?"
  Người làm: team sở hữu service
  Sản phẩm:  hệ thống bớt một failure mode

Vòng ngoài  (chu kỳ tháng/quý)
  Tập hợp incident → Phân tích mẫu → Thay đổi cơ chế → Sửa lớp sinh ra lỗi
  Câu hỏi:  "Vì sao chúng ta liên tục tạo ra loại lỗi này?"
  Người làm: lead / EM / nhóm platform
  Sản phẩm:  hệ thống bớt một lớp failure mode
```

Đây chính là phân biệt **single-loop learning** và **double-loop learning** (Argyris — nguồn công
khai): vòng trong sửa hành động trong khuôn khổ hiện tại; vòng ngoài xét lại chính khuôn khổ. Đa số
tổ chức chỉ có vòng trong, và vì vòng trong luôn có kết quả nhìn thấy được, họ không cảm thấy thiếu
vòng ngoài. Chi phí của việc thiếu vòng ngoài là một dòng chi phí ẩn: số incident không giảm dù mỗi
incident đều được xử lý tốt.

**Mô hình Swiss Cheese áp cho tổ chức.** Mỗi lớp phòng vệ (test, review, canary, alert, on-call,
runbook) là một miếng phô mai có lỗ. Incident xảy ra khi các lỗ thẳng hàng. Hai hệ quả trái ngược với
trực giác: (a) một incident luôn có nhiều nguyên nhân góp phần, nên câu hỏi "nguyên nhân gốc là gì"
(số ít) thường là câu hỏi sai; (b) cách hiệu quả nhất để giảm incident không phải bịt lỗ vừa bị lọt,
mà là **thêm một lớp mới** hoặc **làm các lớp độc lập với nhau hơn**. Nếu test và review cùng phụ
thuộc vào việc một người cẩn thận, chúng là một lớp chứ không phải hai.

**Runbook như một tài sản có khấu hao.** Runbook không phải tài liệu, nó là một tài sản vận hành mất
giá theo tốc độ thay đổi của hệ thống. Một runbook không được chạy trong 6 tháng có xác suất sai rất
cao — và nó tệ hơn không có runbook, vì nó tạo cảm giác an toàn sai và làm người on-call mất thời
gian đi theo hướng dẫn không còn đúng lúc 3 giờ sáng. Suy ra: cơ chế đúng không phải "viết runbook"
mà "runbook được thực thi định kỳ", bằng game day hoặc bằng chính lần on-call gần nhất.

### Practical Framework

**Nhịp 1 — Incident Review cấp tổ chức (mỗi tháng, 90 phút).**

Đây là vòng ngoài. Không lặp lại nội dung postmortem; mục tiêu duy nhất là tìm mẫu.

| Bước | Thời lượng | Nội dung | Đầu ra |
|---|---|---|---|
| 1. Chuẩn bị | Trước họp | Một người tổng hợp mọi incident tháng, gắn nhãn theo *lớp nguyên nhân* (config, capacity, dependency, data, deploy, code logic, human process) và theo *lớp phòng vệ đã hỏng* | Bảng phân loại |
| 2. Đọc số | 15 phút | Số incident theo severity, MTTR, số action item mở/đóng/quá hạn, số lần tái diễn cùng nhãn | Bức tranh xu hướng |
| 3. Tìm mẫu | 30 phút | Nhãn nào xuất hiện ≥ 2 lần? Lớp phòng vệ nào hỏng nhiều nhất? Service nào chiếm nhiều incident nhất | 1–3 mẫu ứng viên |
| 4. Truy nguyên cơ chế | 25 phút | Với mỗi mẫu: cái gì trong cách tổ chức làm việc **sinh ra** loại lỗi này? Không chấp nhận câu trả lời ở tầng cá nhân | Giả thuyết cấu trúc |
| 5. Chốt một can thiệp | 15 phút | Chọn **một** can thiệp cấu trúc duy nhất cho tháng tới, có chủ, có ngày, có chỉ số kiểm chứng | Một cam kết |
| 6. Kiểm chứng | Họp tháng sau | Nhãn đó có giảm không? Nếu không, giả thuyết sai — xét lại chứ không làm mạnh tay hơn | Học hoặc sửa giả thuyết |

Hai quy tắc cứng của nhịp này. Thứ nhất, **chọn một can thiệp, không phải năm**. Năm can thiệp song
song làm bạn không thể quy kết kết quả cho nguyên nhân nào, tức phá điều kiện 2 ở trên. Thứ hai, can
thiệp phải nằm ở dạng "trong công cụ / kiến trúc" nếu khả thi; nếu bạn thấy mình lại đang viết một
dòng vào checklist, hãy dừng và hỏi cái gì có thể làm dòng đó thành không cần thiết.

Với công ty logistics: can thiệp đúng không phải action item thứ bảy về config, mà là một job trong
CI so sánh cấu hình staging và production ở mỗi lần deploy và fail build khi lệch ngoài danh sách
được phép. Một can thiệp, ở dạng công cụ, làm sáu action item trước đó trở nên không cần thiết.

**Nhịp 2 — Quản lý action item như một dòng công việc thật.**

Action item của postmortem thất bại vì nó cạnh tranh với roadmap mà không có chỗ trong capacity. Cơ
chế tối thiểu để nó không chết:

- Action item vào **cùng backlog** với việc feature, cùng công cụ, cùng nghi thức ưu tiên. Backlog
  riêng cho postmortem là backlog sẽ bị bỏ.
- Mỗi action item phải qua được ba câu hỏi để hợp lệ: *Có chủ là một người (không phải một team)? Có
  ngày? Nếu làm xong thì loại incident này không thể xảy ra theo cùng cơ chế nữa — đúng hay chỉ là
  giảm xác suất?* Câu thứ ba phân biệt sửa cấu trúc với sửa bề mặt.
- Chia làm hai loại và đối xử khác nhau: **P-block** (nếu không làm thì rủi ro tái diễn cao, chiếm
  chỗ trong sprint kế tiếp, không được đẩy) và **P-improve** (giảm rủi ro, vào bucket technical
  health, có thể xếp hàng). Tỉ lệ P-block trên tổng action item nên nhỏ; nếu nó là 80%, tiêu chí
  đang bị lạm phát.
- Chỉ số theo dõi: **tỉ lệ action item P-block quá hạn** và **thời gian trung bình từ postmortem đến
  đóng**. Không dùng chỉ số "số action item đã tạo" — nó thưởng cho việc tạo ra nhiều việc.

**Nhịp 3 — Game day (mỗi quý, 2–3 giờ).**

Mục tiêu không phải phá hệ thống mà là **kiểm tra runbook và kiểm tra con người** trong điều kiện
không có áp lực thật. Phiên bản nhẹ, chạy được cả với team 6 người:

```
GAME DAY — bản tối thiểu

Chuẩn bị (1 tuần trước)
  - Chọn 1 kịch bản từ danh sách incident đã xảy ra ở công ty khác cùng ngành
    hoặc từ near-miss của chính mình.
  - Chọn người chơi: người ÍT kinh nghiệm nhất với hệ thống đó (không phải người giỏi nhất
    — mục tiêu là đo chất lượng runbook, không đo trí nhớ của chuyên gia).
  - Thông báo trước là sẽ có game day, KHÔNG nói kịch bản.

Chạy (90 phút, môi trường staging hoặc production có kiểm soát)
  - Facilitator tiêm lỗi hoặc mô tả triệu chứng.
  - Người chơi xử lý theo đúng quy trình incident thật: mở channel, tuyên bố
    severity, gán vai, dùng runbook.
  - Facilitator ghi: mỗi lần runbook sai/thiếu, mỗi lần phải hỏi một người
    ngoài, mỗi lần không tìm được dashboard.

Đóng (30 phút)
  - Đầu ra KHÔNG phải "đã qua / không qua".
  - Đầu ra là: danh sách chỗ runbook sai (sửa ngay trong buổi), danh sách
    kiến thức chỉ tồn tại trong đầu một người, thời gian tới lúc mitigate.
  - So sánh thời gian này với quý trước — đây là chỉ số học tập thật.
```

Khi nào chạy trên production: chỉ khi đã có SLO và error budget, còn budget, ngoài giờ cao điểm, có
người sẵn sàng abort, và đã thông báo. Với fintech, cần thêm sự chấp thuận của bộ phận compliance và
một bản ghi rõ ràng — game day không được trở thành một incident thật phải báo cáo.

**Nhịp 4 — Runbook có cơ chế chống mục.**

- Mỗi runbook có **owner** và **ngày chạy thật gần nhất** ở đầu file. Không có ngày = coi như không
  đáng tin.
- Người on-call có nghĩa vụ nhẹ: mỗi lần dùng runbook, cập nhật một dòng (đúng / sai ở đâu). Chi phí
  30 giây, giữ tài sản không mục.
- Runbook nào không được dùng và không được chạy trong 2 quý: đưa vào game day kỳ tới hoặc xoá. Giữ
  một runbook sai tệ hơn không có.
- Cấu trúc runbook phục vụ người đang stress lúc 3 giờ sáng, không phục vụ người đọc để hiểu: triệu
  chứng lên đầu, lệnh copy-paste được, ngưỡng số cụ thể, mục "KHÔNG ĐƯỢC LÀM", điều kiện leo thang.

**Nhịp 5 — Học từ near-miss.**

Đây là đòn bẩy cao nhất và gần như không tổ chức nào ở quy mô vừa làm. Cơ chế tối thiểu: một kênh
hoặc một form để bất kỳ ai ghi lại "chuyện gần thành sự cố" với 3 dòng — chuyện gì, cái gì đã cứu,
nếu cái đó không có thì sao. Mỗi tháng đọc trong Incident Review. Điều kiện để cơ chế này sống: ghi
near-miss **không bao giờ** dẫn tới hệ quả tiêu cực cho người ghi, và có ít nhất một near-miss mỗi
quý dẫn tới một thay đổi nhìn thấy được — nếu không, mọi người sẽ ngừng ghi.

### Trade-off

| Trục | Nghiêng về bên A khi | Nghiêng về bên B khi | Cái giá của mỗi bên |
|---|---|---|---|
| **A: Sửa ngay từng incident** vs **B: Đợi thấy mẫu rồi sửa cấu trúc** | Incident đang gây thiệt hại lặp lại; can thiệp nhỏ và rẻ | Đã có ≥ 2 incident cùng nhãn; can thiệp cấu trúc tốn nhiều tuần | A: tích tụ nhiều lớp vá, độ phức tạp tăng, nguyên nhân gốc còn nguyên. B: chịu thêm 1–2 incident trong lúc chờ dữ liệu — chỉ chấp nhận được nếu tác động không phải SEV1 |
| **A: Đầu tư vào phòng ngừa** vs **B: Đầu tư vào khả năng hồi phục** | Lỗi có thiệt hại không hoàn tác được (dữ liệu tài chính, gửi tiền sai, rò rỉ dữ liệu) | Hệ thống phức tạp, nhiều dependency ngoài tầm kiểm soát, thiệt hại tỉ lệ với thời gian | A: chi phí tăng phi tuyến, quy trình nặng dần, làm chậm mọi thay đổi kể cả thay đổi an toàn. B: chấp nhận sẽ hỏng thường xuyên hơn — không dùng được cho nhóm lỗi không hoàn tác được |
| **A: Học công khai toàn công ty** vs **B: Học trong nội bộ team** | Mẫu lỗi có tính hệ thống, nhiều team dùng chung nền tảng | Vấn đề đặc thù một service; hoặc Psychological Safety còn thấp | A: rủi ro postmortem bị đọc như bằng chứng buộc tội nếu văn hoá chưa đủ; B: cùng một bài học bị học lại 5 lần bởi 5 team |
| **A: Chuẩn hoá runbook và quy trình chặt** vs **B: Dựa vào phán đoán của người vận hành** | Hệ thống ổn định, lỗi lặp lại có mẫu, đội ngũ luân chuyển nhiều | Hệ thống thay đổi nhanh, lỗi mới lạ, người vận hành là người viết hệ thống | A: mất khả năng ứng phó với lỗi ngoài danh mục, người vận hành ngừng suy nghĩ. B: MTTR phụ thuộc ai đang trực, bus factor thấp |
| **A: Đo chỉ số học tập** vs **B: Không đo, tin vào cảm nhận** | Tổ chức > 20 engineer, nhiều team, cần quyết định phân bổ nguồn lực | Team nhỏ, dữ liệu quá nhiễu để có ý nghĩa thống kê | A: rủi ro chỉ số bị game (viết postmortem cho có, hạ severity để MTTR đẹp). B: không phân biệt được cải thiện với may mắn |

Một trade-off đáng nói riêng vì nó hay bị né: **thời gian dành cho vòng ngoài là thời gian lấy từ
delivery**. 90 phút họp mỗi tháng cộng thời gian chuẩn bị cộng capacity cho một can thiệp cấu trúc —
với team 8 người, đây là cỡ 3–5% capacity (số minh hoạ). Nếu bạn không nói rõ con số này và không bảo
vệ nó, nó sẽ là thứ đầu tiên bị cắt khi gấp, và tổ chức sẽ quay về trạng thái chỉ có vòng trong.
Cách trình bày với cấp trên nên bằng ngôn ngữ ở `04-decision-making.md`: 3% capacity so với chi phí
của 9 incident/quý, mỗi incident tốn X người-giờ ứng phó cộng Y phút downtime.

### Real-world Scenarios

**Tình huống A — 8 postmortem tốt, 0 cải thiện (công ty logistics, 24 engineer, 4 team).**

*Bối cảnh.* Quý I: 8 incident, tất cả có postmortem, tỉ lệ đóng action item 71%. Quý II: 9 incident.
CTO yêu cầu "siết quy trình deploy". Hà — EM mới — xin hai tuần trước khi siết bất cứ thứ gì.

*Các lựa chọn.*

1. Siết quy trình deploy như CTO đề nghị: thêm một tầng approval trước khi deploy production.
2. Đào tạo lại toàn bộ engineer về config management.
3. Phân tích tổng hợp 17 postmortem của hai quý, tìm mẫu, chọn một can thiệp cấu trúc.
4. Tăng số lượng test và bắt buộc coverage ≥ 80%.

*Trade-off.* Lựa chọn 1 rẻ, làm được trong một ngày, và đúng theo trực giác của cấp trên — nhưng nó
tấn công tầng Process trong khi nguyên nhân nằm ở tầng Technology, nên hệ quả gần như chắc chắn là
lead time tăng và số incident không đổi (đúng tổ hợp DORA đã phân tích ở chủ đề 6). Lựa chọn 2 nằm ở
dạng lưu bài học "trong đầu người" — độ bền vài tuần. Lựa chọn 4 sai nhãn: 6/8 incident là lệch cấu
hình giữa môi trường, loại lỗi mà unit test không thấy được. Lựa chọn 3 chậm hai tuần và tốn uy tín
của Hà nếu không ra kết quả, vì trong hai tuần đó có thể có thêm incident và cô sẽ bị hỏi "tại sao
chưa làm gì".

*Quyết định.* Hà chọn 3, nhưng giảm rủi ro chính trị bằng hai việc: cam kết một ngày cụ thể để đưa
kết luận, và trong lúc chờ vẫn áp một biện pháp tạm rẻ (checklist so config bằng tay khi deploy, có
thời hạn tự hết hiệu lực sau 30 ngày để nó không trở thành di sản vĩnh viễn).

*Hậu quả.* Phân tích cho ra: 6/17 là lệch cấu hình, 4/17 là dependency bên thứ ba không có timeout,
còn lại rải rác. Hai can thiệp cấu trúc được chốt trong hai tháng liên tiếp: (1) CI fail khi cấu
hình lệch ngoài allowlist, (2) một thư viện HTTP client nội bộ có timeout và circuit breaker mặc
định, cộng một lint rule chặn việc tạo client thô. Quý III: 3 incident, không có incident nào thuộc
hai nhãn trên. Chi phí: khoảng 6 tuần-người trải trên hai tháng (số minh hoạ).

*Bài học.* Áp lực từ cấp trên thường đẩy về can thiệp ở tầng Process vì đó là tầng cấp trên nhìn thấy
và điều khiển được. Việc của lead là dịch câu hỏi "siết quy trình gì" thành "lớp phòng vệ nào đang
hỏng", rồi mang dữ liệu quay lại. Và: hai tuần phân tích cần được mua bằng một biện pháp tạm, không
bằng lời hứa.

**Tình huống B — cùng một sự việc, ba góc nhìn.**

Sự việc: sau một SEV2 do queue backlog, action item quan trọng nhất — "thêm autoscaling cho consumer
group" — được gán cho Quân (Senior BE) với hạn 3 tuần. Sáu tuần sau nó vẫn mở. Không ai nhắc.

*Nhìn từ IC (Quân).* "Tôi có action item đó trong Jira, nhưng sprint nào cũng đầy việc roadmap. Không
ai bảo tôi được bỏ việc gì để làm nó. Nếu tôi tự bỏ một story để làm, sprint fail và tôi phải giải
thích trong retro. Tôi đã ping trong standup một lần, không ai phản hồi, nên tôi coi như nó không
gấp." Quân không vô trách nhiệm; anh đọc đúng tín hiệu ưu tiên mà hệ thống phát ra.

*Nhìn từ Tech Lead (Tuấn).* "Tôi biết nó chưa xong. Tôi cũng biết nếu tôi ép, sprint sẽ trượt và
Trang (PO) sẽ hỏi. Tôi đang chờ một sprint nhẹ hơn." Tuấn đang tối ưu cục bộ trong ràng buộc mà anh
tin là không thể thay đổi — nhưng anh chưa từng kiểm chứng niềm tin đó bằng cách nêu trade-off ra với
Trang bằng số. Sai lầm của Tuấn không phải sai lầm về ưu tiên, mà là **giữ một quyết định ưu tiên
trong đầu mình thay vì đưa nó lên đúng người có accountability**.

*Nhìn từ Manager (Hà).* Ở tầng này, sáu tuần quá hạn không phải một việc trễ mà là một **tín hiệu về
cơ chế**: tổ chức không có chỗ cho action item trong capacity, nên mọi action item P-block sẽ trễ,
không riêng cái này. Can thiệp của Hà không phải nhắc Quân, mà là ba thay đổi cơ chế: định nghĩa
P-block và cho nó quyền chiếm chỗ trong sprint kế tiếp; đưa tỉ lệ P-block quá hạn vào báo cáo hàng
tháng cho cả Trang cùng đọc; và nêu rõ với Trang rằng bucket technical health không phải mong muốn
của engineering mà là điều kiện để cam kết SLO mà chính Trang đã hứa với khách.

Điểm mấu chốt: cùng một dòng Jira quá hạn, IC đọc thành tín hiệu ưu tiên, Tech Lead đọc thành ràng
buộc capacity, Manager đọc thành lỗi thiết kế cơ chế. Cả ba đều đúng ở tầng của mình, nhưng chỉ can
thiệp ở tầng thứ ba là dừng được sự tái diễn.

**Tình huống C — near-miss bị bỏ qua (ODC, khách Singapore).**

Trong một lần deploy, Vy phát hiện script migration sắp chạy sẽ xoá một cột đang được một job
báo cáo dùng. Cô phát hiện tình cờ, 10 phút trước giờ deploy, vì đang đọc code cho việc khác. Deploy
được hoãn, không có sự cố, không ai viết gì. Ba tháng sau, một migration khác xoá một cột đang dùng
— lần này không ai tình cờ đọc code, và một job báo cáo cho khách hàng chạy sai số liệu trong 9 ngày
trước khi khách phát hiện. Trong dự án ODC, hậu quả không chỉ là dữ liệu sai mà là một cuộc họp về
chất lượng với khách và một điều khoản kiểm soát chặt hơn được thêm vào quy trình — tức tổ chức phải
trả cả bằng tự chủ.

*Bài học.* Near-miss chứa gần như toàn bộ thông tin của một incident thật nhưng không tốn tiền. Cái
duy nhất thiếu là một cơ chế 3 dòng để ghi lại nó. Cơ chế đó rẻ đến mức lý do không có nó không bao
giờ là chi phí — mà là vì không ai được yêu cầu, và vì "may mà không có gì" không tạo ra cảm giác
cấp bách.

### Best Practices

- **Tách rõ hai vòng học và cho vòng ngoài một nhịp cố định.** Vòng trong (postmortem) do team sở hữu
  service làm; vòng ngoài (incident review) do lead/EM làm. Nếu chỉ có một nhịp, nó sẽ là vòng trong,
  vì vòng trong có tính cấp bách.
- **Một can thiệp cấu trúc mỗi kỳ, có chỉ số kiểm chứng và ngày xét lại.** Nhiều can thiệp song song
  phá khả năng quy kết, và khả năng quy kết là điều kiện để học.
- **Dịch mọi bài học xuống dạng thấp nhất khả thi**: từ ký ức → văn bản → công cụ. Mỗi lần bạn viết
  một dòng vào checklist, hãy hỏi cái gì làm dòng đó thành không cần thiết.
- **Action item sống trong cùng backlog với việc feature**, có phân loại P-block / P-improve, và tỉ
  lệ P-block quá hạn được báo cáo cho cả PO đọc. Tính minh bạch này là cơ chế bảo vệ duy nhất đáng
  tin cho action item.
- **Runbook có owner và ngày chạy thật gần nhất.** Không có ngày thì coi như không đáng tin. Cập nhật
  một dòng sau mỗi lần dùng.
- **Chạy game day với người ít kinh nghiệm nhất về hệ thống đó**, không với chuyên gia — vì bạn đang
  đo chất lượng của runbook và của hệ thống quan sát, không đo trí nhớ.
- **Xây một kênh near-miss với chi phí ghi cực thấp và bảo đảm không có hệ quả tiêu cực**, rồi cho
  thấy ít nhất một near-miss mỗi quý dẫn tới thay đổi thật.
- **Đo thời gian tái diễn cùng nhãn nguyên nhân** như chỉ số học tập chính. Nó khó game hơn số
  postmortem đã viết, và nó trả lời đúng câu hỏi "tổ chức có đang tốt lên không".
- **Viết bài học vào chỗ người ta sẽ đọc lúc cần**, không vào chỗ dễ viết. Một dòng cảnh báo trong
  runbook của service liên quan có giá trị hơn một trang trong wiki tổng hợp.
- **Bảo vệ và công bố con số capacity dành cho việc học** (ví dụ 3–5%). Cái gì không có tên trong
  capacity sẽ bị cắt trước tiên khi gấp.

### Anti-patterns

- **Postmortem xong đóng file.** Cơ chế phá hoại: tổ chức tiêu toàn bộ chi phí của việc học (thời
  gian điều tra, họp, viết) mà không nhận phần lợi ích duy nhất (thay đổi cấu trúc). Tệ hơn, nó dạy
  đội ngũ rằng postmortem là nghi lễ, nên chất lượng đầu vào giảm dần và cuối cùng cả cơ chế mất giá
  trị. Dấu hiệu sớm: không ai hỏi về action item của postmortem trước đó; số lần một postmortem cũ
  được ai đó mở lại trong 6 tháng là 0.
- **Đo số postmortem đã viết như một KPI.** Goodhart's Law áp dụng trực tiếp: khi số lượng bản viết
  là chỉ tiêu, cách rẻ nhất để đạt là viết nhiều bản mỏng cho những việc nhỏ, và song song đó là áp
  lực ngầm hạ severity cho những việc thật để tránh phải viết bản dày. Kết quả là dữ liệu incident bị
  bóp méo ở đúng chỗ quan trọng nhất.
- **Action item "đào tạo lại" và "cẩn thận hơn".** Đây là action item nằm ở dạng lưu bài học yếu nhất,
  và nó ngầm định rằng nguyên nhân là thiếu hiểu biết hoặc thiếu chú ý của một cá nhân. Trong hệ
  thống phức tạp, giả định đó gần như luôn sai — người vận hành đã hành động hợp lý với thông tin họ
  có lúc đó. Dấu hiệu sớm: hơn 30% action item không thay đổi bất cứ dòng code, config hay công cụ
  nào.
- **Bịt đúng cái lỗ vừa lọt.** Sau mỗi incident, thêm một kiểm tra hẹp cho đúng trường hợp vừa xảy
  ra. Sau hai năm, hệ thống có 40 kiểm tra đặc thù, không ai biết cái nào còn cần, và lớp phòng vệ
  vẫn chỉ là một lớp. Cơ chế đúng là hỏi lớp nào đã hỏng và các lớp có độc lập với nhau không.
- **Runbook viết một lần rồi để đó.** Nguy hiểm hơn không có runbook vì nó tạo an toàn giả: người
  on-call tin vào nó, đi theo hướng dẫn sai, mất 20 phút, và mất luôn niềm tin vào toàn bộ runbook
  khác. Dấu hiệu sớm: runbook không có ngày, hoặc có ngày cách đây hơn 6 tháng.
- **Hero Culture trong vận hành được thưởng công khai.** Khi tổ chức khen người thức đêm cứu hệ
  thống mà không hỏi vì sao chỉ người đó cứu được, nó tạo hai hiệu ứng: người đó không có động lực
  chia sẻ kiến thức (giá trị của họ nằm ở việc là người duy nhất), và những người khác học rằng
  đường tới sự công nhận là chờ hệ thống hỏng chứ không phải làm nó không hỏng. Dấu hiệu sớm: cùng
  một tên xuất hiện trong 80% incident timeline.
- **Học được nhưng không lan.** Team A giải quyết xong một loại lỗi; team B, C, D gặp lại đúng loại đó
  trong 6 tháng sau. Cơ chế phá hoại nằm ở tầng Organization: không có kênh nào để bài học đi ngang.
  Với công ty > 3 team, đây thường là dòng chi phí ẩn lớn nhất của tầng Improvement.
- **Chaos engineering như một trò biểu diễn.** Chạy game day trên production khi chưa có SLO, chưa có
  observability đủ, chưa có khả năng abort — biến một bài kiểm tra thành một incident thật, và tiêu
  hết vốn chính trị cho mọi thực hành tương tự trong hai năm tiếp theo.

### Khi nào KHÔNG nên áp dụng

- **Khi Psychological Safety còn thấp, đừng bắt đầu bằng vòng ngoài.** Incident review cấp tổ chức
  đọc dữ liệu xuyên nhiều team; trong môi trường mà postmortem còn bị dùng để quy trách nhiệm, việc
  tổng hợp này sẽ được đọc như xếp hạng team và sẽ tạo áp lực hạ severity, giấu incident nhỏ. Thứ tự
  đúng là làm blameless hoạt động thật ở vòng trong trước (`03-team-leadership.md`), rồi mới mở vòng
  ngoài — thường mất 2–3 quý.
- **Với team dưới 6 người và ít hơn 4–5 incident mỗi quý.** Dữ liệu quá thưa để tìm mẫu có ý nghĩa,
  và cả nhóm đã có toàn bộ ngữ cảnh trong đầu — một nhịp họp riêng chỉ thêm chi phí. Ở quy mô này,
  cơ chế đủ là: một trang ghi nhãn nguyên nhân của từng incident, đọc lại 20 phút mỗi quý.
- **Trong giai đoạn khủng hoảng thật sự** (mất khách hàng lớn, chạy deadline sống-còn, hệ thống đang
  hỏng liên tục hàng ngày). Lúc này tổ chức không có băng thông cho double-loop learning; việc đúng
  là dồn vào ổn định hoá và ghi lại đủ để phân tích **sau**. Nhưng phải có ngày cụ thể để quay lại,
  nếu không "giai đoạn khủng hoảng" sẽ kéo dài vô hạn — đó là cách nhiều tổ chức mất khả năng học
  vĩnh viễn.
- **Game day trên production khi chưa đủ ba điều kiện**: có observability để biết mình vừa gây ra gì,
  có error budget còn dư, có khả năng abort trong vài phút. Thiếu một trong ba, chạy trên staging.
  Với hệ thống thuộc phạm vi compliance, thêm điều kiện thứ tư là sự chấp thuận bằng văn bản.
- **Với hệ thống đang trong quá trình bị thay thế.** Đầu tư vào runbook, game day, và can thiệp cấu
  trúc cho một service sẽ bị tắt trong 4 tháng là chi phí thuần. Chiến lược đúng ở đây là giữ nó vận
  hành với chi phí tối thiểu và dồn năng lực học vào hệ thống thay thế.
- **Khi bạn không có quyền thay đổi cơ chế.** Trong nhiều dự án ODC, quy trình deploy, công cụ, và cả
  quyền tạo job CI thuộc khách hàng. Chạy incident review để tìm ra can thiệp cấu trúc mà bạn không
  được phép thực hiện sẽ tạo ra sự bất lực có tài liệu, làm giảm tinh thần hơn là tăng năng lực. Việc
  đúng trong ràng buộc này: chuyển đầu ra của incident review thành **đề xuất có số liệu gửi khách**
  (dùng cấu trúc ở `02-communication.md`), và song song làm những can thiệp nằm trong quyền của mình
  — thường là ở lớp test, lớp quan sát, và lớp runbook.

---

## Tự kiểm tra

Trả lời bằng con số, tên người, hoặc đường dẫn cụ thể. Câu nào bạn không trả lời được bằng dữ liệu
là một chỗ hệ thống của bạn đang mù.

1. **Ba incident gần nhất của bạn có ai là Incident Commander không, và người đó có tự sửa gì trong
   lúc chỉ huy?** Nếu tên người chỉ huy và tên người sửa là cùng một người trong cả ba lần, bạn chưa
   có cấu trúc chỉ huy — bạn có một người giỏi.
2. **Bao nhiêu phần trăm alert bắn ra trong 30 ngày qua dẫn tới một hành động của con người?** Nếu
   dưới 50%, người on-call của bạn đang học cách bỏ qua alert, và alert quan trọng tiếp theo sẽ bị
   bỏ qua cùng cách.
3. **Ai đang on-call tuần này, và người đó on-call bao nhiêu tuần trong 12 tuần qua?** Nếu một tên
   xuất hiện quá 4 lần, bạn đang có một điểm hỏng đơn lẻ bằng con người.
4. **SLO của service quan trọng nhất của bạn là bao nhiêu, và ai đã đồng ý con số đó — có PO trong
   danh sách không?** Một SLO chỉ engineering biết không phải SLO, nó là một mong muốn.
5. **Trong 6 tháng qua, có loại nguyên nhân gốc nào xuất hiện từ hai lần trở lên?** Gọi tên nó ra.
   Nếu bạn không biết, bạn chưa có vòng học ngoài.
6. **Có bao nhiêu action item P-block đang quá hạn, và cái cũ nhất quá hạn bao lâu?** Nếu con số này
   không có ai theo dõi định kỳ, action item của bạn đang là danh sách mong ước.
7. **Mở runbook của service quan trọng nhất — nó ghi ngày nào lần cuối được chạy thật?** Nếu không có
   ngày, hoặc trên 6 tháng, hãy giả định nó sai và lên kế hoạch game day.
8. **Nếu tôi hỏi bốn chỉ số DORA của team bạn ngay lúc này, bạn lấy được trong bao nhiêu phút?** Nếu
   câu trả lời là "cần vài ngày để tổng hợp", bạn đang điều hành bằng cảm nhận, và mọi tranh luận về
   tốc độ trong tổ chức của bạn sẽ được quyết bằng người nói to nhất.

---

## Liên kết chương khác

- [`00-nen-tang-leadership.md`](/series/engineering-leedership/00-nen-tang-leadership/) — Accountability vs Responsibility là gốc
  của việc vì sao Incident Commander phải là một người, và vì sao blameless không đồng nghĩa với
  không có ai chịu trách nhiệm.
- [`02-communication.md`](/series/engineering-leedership/02-communication/) — cập nhật cho stakeholder trong incident, no-surprise
  principle, và cách viết bản postmortem đối ngoại tách khỏi bản nội bộ.
- [`03-team-leadership.md`](/series/engineering-leedership/03-team-leadership/) — Psychological Safety là điều kiện tiên quyết kỹ
  thuật để postmortem có dữ liệu đầu vào trung thực; Hero Culture là anti-pattern chung của hai
  chương.
- [`04-decision-making.md`](/series/engineering-leedership/04-decision-making/) — quyết định trong incident là quyết định dưới áp
  lực thời gian với thông tin thiếu; error budget là cách biến tranh luận giá trị thành quyết định có
  dữ liệu; ngôn ngữ tiền để xin capacity cho việc học.
- [`05-technical-leadership.md`](/series/engineering-leedership/05-technical-leadership/) — technical debt là nguồn sinh incident
  có thể dự báo được; golden path và standard là dạng lưu bài học ở tầng công cụ, tức bền nhất.
- [`07-project-delivery.md`](/series/engineering-leedership/07-project-delivery/) — action item cạnh tranh capacity với roadmap;
  bucket technical health và cách bảo vệ nó là chỗ quyết định action item sống hay chết.
- [`09-to-chuc-va-scaling.md`](/series/engineering-leedership/09-to-chuc-va-scaling/) — bài học đi ngang giữa các team là bài toán
  tầng Organization; Platform Team và Enabling Team là cơ chế lan bài học ở quy mô lớn.
- [`10-case-studies.md`](/series/engineering-leedership/10-case-studies/) — case study về một incident production nghiêm trọng,
  phân tích đầy đủ từ bối cảnh đến bài học.
- [`12-anti-patterns.md`](/series/engineering-leedership/12-anti-patterns/) — Blame Culture, Hero Culture, và các anti-pattern về
  metric được tổng hợp cùng dấu hiệu sớm và cách tháo gỡ.
