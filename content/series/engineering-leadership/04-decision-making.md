+++
title = "Level 3A — Decision Making: Ra quyết định kỹ thuật dưới ràng buộc"
date = "2026-08-01T12:00:00+07:00"
draft = false
tags = ["engineering-leadership", "management", "career"]
series = ["Engineering Leadership & Technical Management"]
+++

# Level 3A — Decision Making

Một team fintech ở Hà Nội, 14 engineer. Trong sáu tháng, cùng một câu hỏi — "luồng đối soát dùng
Kafka hay RabbitMQ" — được mở lại bốn lần: lần đầu ở kickoff, lần hai khi người đề xuất ban đầu nghỉ
việc, lần ba sau một incident mất message, lần bốn khi một Senior mới vào và thấy lựa chọn hiện tại
"không hợp lý". Mỗi lần khoảng 5–6 giờ họp với 5 người tham gia. Cộng lại hơn 100 người-giờ, và cuối
sáu tháng hệ thống vẫn đang chạy đúng thứ đã chọn ở lần một — chỉ khác là bây giờ không ai còn nhớ vì
sao. Không có bản ghi nào nói lần trước đã cân nhắc phương án nào, loại cái gì, vì lý do gì, và với
giả định nào.

Đây không phải vấn đề về Kafka. Đây là vấn đề về **năng lực sản xuất quyết định** của một tổ chức.
Công việc của một lead, bỏ hết title và mô tả công việc đi, là sản xuất quyết định dưới hai ràng buộc
không thể tháo: thông tin luôn thiếu, và thời gian luôn hữu hạn. Bạn không bao giờ có đủ dữ liệu, và
thời điểm bạn có đủ dữ liệu thì cửa sổ cơ hội đã đóng. Level này nói về cách làm việc đó có kỷ luật.

Điểm khó nhất, và cũng là điểm đảo ngược trực giác nhất: **chất lượng của một lead không đo được
bằng kết quả của từng quyết định riêng lẻ.** Hệ thống kỹ thuật và tổ chức đều có nhiễu (một quyết
định tốt vẫn có thể ra kết quả tệ vì lý do ngoài kiểm soát) và có độ trễ (hậu quả thật của một quyết
định kiến trúc thường hiện ra sau 12–24 tháng, lúc người quyết đã đổi việc). Với nhiễu cao và độ trễ
dài, dùng outcome của từng lần để đánh giá người quyết là dùng một tín hiệu có tỉ lệ nhiễu/tín hiệu
rất xấu. Cái đo được, và cái duy nhất bạn kiểm soát được, là **quá trình**: có liệt kê phương án hay
không, tiêu chí có được chốt trước khi biết phương án nào thắng hay không, ai chịu accountability, có
điều kiện xét lại hay không, có bản ghi để học hay không.

Trong chuỗi Business Goal → People → Process → Technology → Execution → Feedback, chương này nằm ở
tầng Process, nhưng nó là bộ chuyển đổi hai chiều: nó dịch Business Goal thành ràng buộc kỹ thuật cụ
thể, và dịch tín hiệu từ tầng Feedback trở lại thành sửa đổi cho lần quyết định sau. Không có tầng
này, tầng Technology sẽ được chọn theo sở thích, và tầng Feedback sẽ tạo ra dữ liệu mà không ai dùng
để cải thiện.

Mục lục nội bộ:

1. [Bản chất của một quyết định kỹ thuật](#1-bản-chất-của-một-quyết-định-kỹ-thuật)
2. [Reversible vs Irreversible Decisions](#2-reversible-vs-irreversible-decisions)
3. [Decision Matrix và ra quyết định có tiêu chí](#3-decision-matrix-và-ra-quyết-định-có-tiêu-chí)
4. [Cost-Benefit Analysis và ngôn ngữ tiền](#4-cost-benefit-analysis-và-ngôn-ngữ-tiền)
5. [Risk Assessment](#5-risk-assessment)
6. [Prioritization Frameworks ở cấp team và sản phẩm](#6-prioritization-frameworks-ở-cấp-team-và-sản-phẩm)
7. [Ra quyết định trong nhóm: consensus, consent và người quyết cuối](#7-ra-quyết-định-trong-nhóm-consensus-consent-và-người-quyết-cuối)
8. [Decision Log và học từ quyết định](#8-decision-log-và-học-từ-quyết-định)
9. [Tự kiểm tra](#tự-kiểm-tra)
10. [Liên kết chương khác](#liên-kết-chương-khác)

> Mọi con số trong chương này là **số minh hoạ** để bạn thấy cách tính, không phải dữ liệu nội bộ của
> bất kỳ công ty nào. Khi áp dụng, thay bằng số của tổ chức bạn.

---

## 1. Bản chất của một quyết định kỹ thuật

### Problem Statement

Trong một team, phần lớn thứ được gọi là "quyết định" thực ra không phải quyết định. Đây là các hiện
tượng đếm được cho biết tổ chức đang không thực sự ra quyết định:

- **Cuộc họp kết thúc mà không ai biết đã chốt gì.** Kiểm tra: sau buổi họp, nhắn riêng cho ba người
  tham dự cùng một câu "theo anh/chị thì mình đã chốt gì?". Nếu nhận về ba câu trả lời khác nhau,
  buổi họp đó đã sản xuất ra một cảm giác đồng thuận, không phải một quyết định.
- **Cùng một chủ đề xuất hiện trong ba buổi họp cách nhau hơn một tháng.** Đếm số chủ đề như vậy
  trong quý. Ở team fintech đầu chương, con số là 5. Mỗi chủ đề tái mở là bằng chứng lần trước không
  đóng được.
- **Quyết định chỉ có một phương án.** Đọc lại 10 ADR hoặc RFC gần nhất, đếm số cái có mục "phương án
  đã loại và lý do". Nếu dưới 3/10, tổ chức đang hợp thức hoá lựa chọn chứ không so sánh lựa chọn.
- **Không xác định được ai accountable.** Hỏi "nếu tháng sau việc này sai thì ai là người phải giải
  thích?" — nếu câu trả lời là "cả team" hoặc một khoảng lặng, thì không có ai.
- **Thời gian từ lúc vấn đề được nêu tới lúc có quyết định.** Decision latency. Đo bằng ngày. Nếu
  trung vị vượt 10 ngày làm việc cho các quyết định mức team, engineer sẽ tự quyết trong im lặng và
  bạn sẽ biết về quyết định đó qua PR, không qua thảo luận.

Hậu quả không phải "team hơi lộn xộn". Nó có chuỗi khá cố định: quyết định không rõ → mỗi người triển
khai theo cách hiểu riêng → xuất hiện hai cách làm cùng một việc trong codebase → chi phí bảo trì
tăng → khi có sự cố, không truy được vì sao hệ thống ở trạng thái này → mọi tranh luận thành tranh
luận về sở thích vì không có tiêu chí nào được ghi lại → người có tiếng nói to nhất hoặc thâm niên
cao nhất thắng → engineer giỏi nhưng ít thâm niên rút khỏi thảo luận kỹ thuật. Bước cuối là bước đắt
nhất và khó đảo nhất.

### First Principles

**Cơ chế một: quyết định là một cam kết phân bổ tài nguyên dưới bất định, không phải một phát biểu ý
kiến.** Ba điều kiện để một việc là quyết định: (a) có ít nhất hai đường đi loại trừ nhau; (b) sau
khi chọn, một lượng tài nguyên (người-giờ, tiền hạ tầng, sự chú ý của tổ chức) bị khoá vào đường đã
chọn; (c) tại thời điểm chọn, thông tin không đủ để biết chắc đường nào đúng. Thiếu (a) thì đó là
thực thi. Thiếu (b) thì đó là ý kiến. Thiếu (c) thì đó là phép tính, và bạn không cần lead để làm
phép tính.

**Cơ chế hai: chi phí cơ hội là chi phí thật, và thời gian là tài nguyên duy nhất không hoàn lại
được.** Khi team dành 6 tuần refactor module thanh toán, chi phí không phải "6 tuần lương". Chi phí
là **thứ tốt nhất trong số những thứ khác team có thể làm trong 6 tuần đó**, cộng với 6 tuần thị
trường không đứng chờ. Tiền có thể huy động thêm; người có thể tuyển thêm (với độ trễ 2–4 tháng và
chi phí onboarding); thời gian thì không. Đây là lý do vì sao trong hầu hết bài toán ưu tiên ở
startup, biến ràng buộc thật không phải tiền mà là số tuần còn lại trước một mốc bên ngoài (vòng gọi
vốn, mùa campaign, deadline compliance).

**Cơ chế ba: thông tin đến muộn hơn thời điểm cần quyết, và điều đó là cấu trúc chứ không phải khuyết
điểm.** Giá trị của thông tin bổ sung giảm dần, còn chi phí chờ tăng đều. Đường cong "giá trị của
việc biết thêm" gần như luôn phẳng ra sau vài ngày điều tra đầu tiên, trong khi cost of delay tích
lũy tuyến tính. Suy ra: tồn tại một điểm mà **quyết định ngay với 70% thông tin có giá trị kỳ vọng
cao hơn quyết định sau ba tuần với 85% thông tin**. Việc của lead là tìm điểm đó, không phải đẩy nó
về phía 100%.

**Cơ chế bốn: ba loại phát biểu thường bị gộp làm một.**

| Loại | Định nghĩa vận hành | Kiểm tra bằng câu hỏi | Ví dụ |
|---|---|---|---|
| **Decision** (chọn) | Chọn một trong nhiều đường đi loại trừ nhau, có cam kết tài nguyên | "Sau việc này, cái gì bị khoá lại?" | "Chúng ta dùng PostgreSQL cho service này, không dùng MongoDB" |
| **Judgment** (đánh giá) | Ước lượng một trạng thái hoặc giá trị của thế giới, có thể sai/đúng | "Cái này có thể kiểm chứng bằng dữ liệu nào?" | "Tôi cho rằng tải peak sẽ khoảng 3.000 TPS" |
| **Guess** (đoán) | Phát biểu không dựa trên mô hình hay dữ liệu nào có thể nêu ra | "Anh dựa vào đâu?" — không trả lời được | "Kafka chắc chịu được thôi" |

Quan hệ: một decision tốt được xây trên các judgment được nêu tường minh. Bệnh phổ biến nhất trong
họp kỹ thuật là **guess được phát ngôn với ngữ điệu của judgment, rồi được đối xử như decision**.
Cách chữa duy nhất có hiệu lực là bắt buộc tách bạch: mỗi judgment phải được viết ra kèm mức tin cậy
và cách kiểm chứng. Khi Senior nói "Kafka chịu được", câu hỏi tiếp theo không phải "có chắc không"
(sẽ nhận được "chắc") mà là "anh đang giả định bao nhiêu message/giây, message bao nhiêu KB, và mình
kiểm cái đó bằng benchmark nào trong bao lâu?".

### Mental Model

**Bounded Rationality (Herbert Simon).** Con người không tối ưu, vì ba giới hạn cứng: thông tin không
đầy đủ, năng lực xử lý hữu hạn, và thời gian hữu hạn. Simon rút ra rằng hành vi hợp lý thực tế không
phải *maximizing* mà là **satisficing** — đặt trước một ngưỡng "đủ tốt" theo tiêu chí đã định, rồi
chọn phương án đầu tiên vượt ngưỡng. Map vào công việc lead: thay vì hỏi "công cụ nào tốt nhất", hỏi
"ngưỡng nào là đủ cho bài toán này trong 18 tháng tới, và phương án nào vượt ngưỡng với chi phí thấp
nhất". Sự khác biệt không phải về tinh thần cầu toàn — nó quyết định việc bạn dừng tìm kiếm sau 3
ngày hay sau 3 tuần.

Hệ quả thực tế của satisficing: bạn phải viết ngưỡng ra **trước** khi khảo sát phương án. Nếu viết
sau, ngưỡng sẽ tự động khớp với phương án bạn đã thích.

**Expected Value như một khuôn tính, không phải một con số.** EV = Σ (xác suất × giá trị). Trong kỹ
thuật, bạn gần như không bao giờ có xác suất thật, và điều đó không làm khuôn tính vô dụng. Giá trị
của nó nằm ở chỗ nó buộc phát biểu ba thứ vốn bị để ngầm: các kết cục có thể, độ lớn tương đối của
mỗi kết cục, và niềm tin của bạn về khả năng xảy ra. Ví dụ minh hoạ: chọn tự viết một service
gateway thay vì mua dịch vụ có sẵn. Kết cục tốt (60%): tiết kiệm 300 triệu/năm phí license. Kết cục
xấu (40%): chậm 8 tuần và mất 1 engineer vào việc bảo trì vĩnh viễn, quy ra khoảng 600 triệu/năm.
EV = 0,6 × 300 − 0,4 × 600 = 180 − 240 = **−60 triệu/năm**. Kết luận không phải "đừng làm" — kết
luận là **quyết định này nhạy với xác suất trượt 8 tuần**, nên việc đáng làm tiếp theo là giảm xác
suất đó (spike 1 tuần) chứ không phải tranh luận thêm về triết lý build-vs-buy.

**Process quality vs Outcome quality — và cái bẫy resulting.** Annie Duke gọi việc suy từ kết quả về
chất lượng quyết định là *resulting*; tâm lý học gọi nó là outcome bias. Ma trận 2×2:

| | Kết quả tốt | Kết quả xấu |
|---|---|---|
| **Quá trình tốt** | Xứng đáng. Ghi lại vì sao đúng. | **Ô nguy hiểm nhất.** Nếu tổ chức phạt ô này, không ai dám quyết nữa. |
| **Quá trình xấu** | **Ô độc hại nhất.** Tổ chức học sai bài học và sẽ lặp lại. | Dễ xử lý nhất — ai cũng thấy cần sửa. |

Hai ô cần chú ý là hai ô chéo. Ô "quá trình xấu + kết quả tốt" là ô độc hại nhất vì nó tạo ra một
câu chuyện thành công quanh một hành vi sai, và câu chuyện đó sẽ được lặp lại với xác suất trượt cao
hơn ở lần sau. Ví dụ: một Senior tự deploy trực tiếp lên production để cứu một bug lúc 11 giờ đêm,
thành công, được cả team khen. Bài học tổ chức vừa học là "bỏ qua quy trình khi gấp thì được thưởng".
Việc của lead ở đây là tách hai lời khen: cảm ơn hành động cứu hệ thống, đồng thời sửa quy trình để
lần sau không cần đến hành vi đó.

### Practical Framework

**Cấu trúc tối thiểu của một quyết định có thể kiểm toán được (auditable decision).** Bảy trường
dưới đây là mức sàn. Thiếu bất kỳ trường nào là thiếu, không phải "gọn nhẹ".

| # | Trường | Câu hỏi phải trả lời | Tiêu chí biết là xong |
|---|---|---|---|
| 1 | **Problem** | Vấn đề gì, ai đang chịu đau, đo bằng gì | Viết được một câu có số trong đó |
| 2 | **Constraints** | Ràng buộc không thể vi phạm (thời gian, tiền, compliance, năng lực team) | Mỗi ràng buộc có nguồn: ai áp, dựa trên gì |
| 3 | **Options (≥3)** | Ít nhất ba phương án thật, trong đó có "không làm gì" | Mỗi phương án có người sẵn sàng bảo vệ nó |
| 4 | **Criteria** | Tiêu chí đánh giá và trọng số, **chốt trước khi cho điểm** | Có timestamp chứng minh chốt trước |
| 5 | **Chọn** | Chọn cái nào, và vì sao không chọn hai cái kia | Mỗi phương án bị loại có một dòng lý do cụ thể |
| 6 | **Điều kiện xét lại** | Tín hiệu nào (số liệu, sự kiện, mốc thời gian) làm quyết định này được mở lại | Viết dạng "nếu X vượt Y trước ngày Z" |
| 7 | **Accountable** | Một tên người (không phải team) chịu giải thích kết quả | Người đó đã đọc và xác nhận |

Ba lưu ý về cách dùng bảng này trên thực tế:

**Về Options.** Yêu cầu "ít nhất ba" không phải con số tuỳ ý. Với hai phương án, não người chuyển
sang chế độ so sánh nhị phân và bị *narrow framing*: bạn đánh giá A so với B, không đánh giá A so với
không gian khả năng. Phương án thứ ba thường tiết lộ chiều nào của bài toán thật sự quan trọng. Nếu
không nghĩ ra phương án thứ ba, dùng ba câu mồi: "nếu chỉ có 1/5 thời gian thì làm gì?", "nếu không
được thêm người thì làm gì?", "nếu không làm gì cả thì sáu tháng nữa hệ thống ra sao?". Phương án
"không làm gì" phải luôn nằm trong danh sách vì nó là baseline để tính chi phí cơ hội.

**Về Criteria chốt trước.** Đây là cơ chế chống motivated reasoning mạnh nhất và rẻ nhất. Cách thực
thi: viết tiêu chí vào một file, commit, rồi mới điền điểm ở commit sau. Git history trở thành bằng
chứng thứ tự. Nghe vụn vặt, nhưng nó chuyển "tôi tin là mình khách quan" thành một dữ kiện kiểm được.

**Về Điều kiện xét lại.** Đây là trường bị bỏ nhiều nhất và là trường ngăn được cái đau đầu chương:
một quyết định không có điều kiện xét lại sẽ bị mở lại theo cảm xúc và nhân sự, còn một quyết định có
điều kiện xét lại thì chỉ mở lại khi điều kiện xảy ra. Nó biến câu hỏi "sao mình lại dùng RabbitMQ"
từ một cuộc tranh luận thành một phép kiểm tra: điều kiện xét lại đã xảy ra chưa? Chưa? Vậy chưa mở
lại, và cuộc thảo luận kết thúc trong 2 phút thay vì 6 giờ.

Template khối code cho quyết định mức nhỏ (dùng ngay trong comment PR hoặc kênh Slack của team):

```
DECISION: <một câu, thể chủ động, có động từ>
Ngày: 2026-07-25   |   Accountable: Tuấn (Tech Lead)
Problem: <1-2 câu, có số>
Constraints: <liệt kê, mỗi cái kèm nguồn>
Options:
  A. <phương án>  → loại vì <lý do>
  B. <phương án>  → CHỌN
  C. Không làm gì → loại vì <lý do>
Criteria (chốt lúc 2026-07-22, trước khi khảo sát):
  1. <tiêu chí> (trọng số)
  2. ...
Giả định có thể sai: <liệt kê 1-3 giả định + cách kiểm>
Điều kiện xét lại: nếu <X vượt Y> hoặc trước <ngày Z>
```

### Trade-off

**Tốc độ ra quyết định vs độ chắc của quyết định.** Nghiêng về tốc độ khi: quyết định khả đảo, blast
radius nhỏ, cost of delay cao, và team có năng lực phát hiện sai sớm (monitoring tốt, release nhanh).
Nghiêng về độ chắc khi: khó đảo, ảnh hưởng dữ liệu hoặc contract bên ngoài, hoặc tổ chức không có
khả năng rollback nhanh. Cái mất khi chọn tốc độ: một số quyết định sẽ sai và phải sửa, tạo ra rework
mà người ngoài dễ đọc thành "team làm hớ". Cái mất khi chọn độ chắc: cost of delay, và một hiệu ứng
phụ ít ai nói — team học chậm hơn, vì học chủ yếu đến từ vòng quyết-định-quan-sát-sửa.

**Tường minh hoá vs chi phí quy trình.** Viết ra bảy trường ở trên tốn 20–60 phút cho một quyết định
mức trung. Với một team ra 3 quyết định lớn mỗi quý, đó là chi phí không đáng kể so với một lần tái
mở. Với một team ra 30 quyết định nhỏ mỗi tuần, áp cùng cấu trúc là tự sát về năng suất. Đây chính là
lý do chủ đề 2 tồn tại: phải phân loại quyết định trước khi chọn mức quy trình.

**Ai quyết: người gần thông tin nhất vs người gánh hậu quả.** Người gần thông tin nhất (engineer đang
viết code) quyết nhanh và đúng chi tiết, nhưng có thể không thấy chi phí lan ra hệ thống khác. Người
gánh hậu quả (lead, EM) thấy chi phí toàn cục, nhưng thông tin đã qua 1–2 lớp truyền và mất chi tiết.
Nghiêng về người gần thông tin khi hậu quả bị chặn trong một service; nghiêng về người gánh hậu quả
khi hậu quả vượt biên giới team. Cái mất ở cực thứ hai: engineer dần thôi suy nghĩ về hậu quả, vì
"đằng nào cũng có người khác quyết".

### Real-world Scenarios

**Tình huống: đổi cách xử lý retry trong service thanh toán.**

Bối cảnh minh hoạ: một ví điện tử, khoảng 40 engineer. Service thanh toán gọi sang cổng đối tác. Khi
đối tác timeout, code hiện tại retry 3 lần cách nhau 1 giây. Khoa (Senior BE) phát hiện có trường hợp
đối tác đã ghi nhận giao dịch nhưng phản hồi timeout, nên retry tạo giao dịch trùng — trong 30 ngày
có 47 case trùng, tổng giá trị khoảng 210 triệu đồng (số minh hoạ), đã hoàn tiền thủ công. Khoa đề
xuất chuyển sang idempotency key và bỏ retry mù.

Cùng một sự việc, ba tầng đọc khác nhau:

**Nhìn từ IC (Khoa, Senior BE).** Vấn đề với Khoa là một bug rõ ràng có cách sửa rõ ràng. Cái Khoa
thấy: code sai, có số liệu, có giải pháp chuẩn ngành. Cái Khoa không thấy: đối tác cần 3–4 tuần để
hỗ trợ idempotency key ở phía họ; và trong 3–4 tuần đó vẫn phải có phương án tạm. Sai lầm điển hình
của tầng này là **trình bày một giải pháp thay vì trình bày một quyết định** — nói "mình nên dùng
idempotency key" thì lead phải tự đi tìm phương án B và C, và nếu lead bận, đề xuất sẽ nằm im.

Cách làm khác: Khoa viết ba phương án. (A) Idempotency key hai phía — đúng nhất, cần đối tác, 4 tuần.
(B) Bảng ghi nhận giao dịch đang bay ở phía mình + kiểm tra trạng thái trước khi retry — làm được
trong 3 ngày, không cần đối tác, không đúng hoàn toàn nhưng đóng được ~90% case. (C) Bỏ retry hoàn
toàn — 1 giờ code, giảm tỉ lệ thành công của giao dịch. Với ba phương án trên bàn, quyết định trở
thành hiển nhiên: làm B ngay tuần này, đồng thời khởi động A với đối tác, và đóng B lại khi A xong.
Không ai phải tranh luận về triết lý.

**Nhìn từ Tech Lead (Tuấn).** Với Tuấn, đây không phải một bug mà là một **lớp lỗi**. Câu hỏi của
Tuấn: còn bao nhiêu chỗ khác trong hệ thống đang retry mù một thao tác không idempotent? Nếu câu trả
lời là 6 chỗ, thì quyết định thật không phải "sửa retry ở service thanh toán" mà là "chuẩn hoá cách
gọi thao tác có tác dụng phụ trên toàn hệ thống", và đó là quyết định có blast radius khác hẳn, cần
ADR, cần thời hạn phản biện. Cái Tuấn phải cưỡng lại là quyết ngay theo phương án của Khoa vì nó
đúng — đúng cục bộ không có nghĩa là đủ rộng.

**Nhìn từ Manager (Linh, EM).** Linh đọc sự việc qua ba trục mà hai người trên không nhìn: (1) 47
case trùng đã được xử lý thủ công bởi ai, và người đó có đang gánh việc vô hình không? (2) Đối tác
timeout ở tần suất này có phải vấn đề hợp đồng cần escalate lên business, chứ không chỉ vấn đề code?
(3) Việc Khoa phát hiện ra sau 30 ngày cho biết monitoring và quy trình đối soát có lỗ — đó là vấn đề
hệ thống phát hiện lỗi, thuộc `06-incident-va-metrics.md`. Nếu Linh chỉ approve fix và đi tiếp, ba
vấn đề kia còn nguyên và sẽ trở lại dưới hình dạng khác.

Bài học chung: cùng một dữ kiện, ba tầng nhìn ra ba quyết định khác nhau, và cả ba đều cần. Cơ chế
tránh bỏ sót là câu hỏi bắt buộc trong mỗi buổi review: "quyết định này là quyết định mức nào — mức
code, mức hệ thống, hay mức tổ chức?".

### Best Practices

- **Bắt buộc phương án thứ ba, và "không làm gì" luôn là một phương án.** Lý do: nó phá narrow
  framing và cho baseline để tính chi phí cơ hội. Cách thực thi rẻ nhất: template RFC của team có ba
  ô Options, không cho submit khi thiếu.
- **Chốt tiêu chí trước khi khảo sát phương án, có dấu vết thời gian.** Lý do: đây là biện pháp duy
  nhất khiến motivated reasoning bị phát hiện được từ bên ngoài, kể cả bởi chính người quyết.
- **Mỗi quyết định có đúng một cái tên người ở trường Accountable.** Lý do: accountability không chia
  được; chia là mất. "Team quyết" nghĩa là khi sai, không có ai để hỏi *vì sao lúc đó anh nghĩ thế*,
  và learning loop bị cắt.
- **Viết giả định kèm cách kiểm chứng, không viết giả định trần.** "Giả định peak 3.000 TPS" là vô
  dụng; "giả định peak 3.000 TPS, kiểm bằng log 3 campaign gần nhất, Hà làm trong 1 ngày" là dùng
  được. Giả định không kiểm được là guess mang mặt nạ judgment.
- **Đặt deadline cho chính quyết định, không chỉ cho việc thực thi.** Câu chuẩn: "chúng ta chốt việc
  này trước 17h thứ Năm; nếu đến lúc đó chưa hội tụ thì tôi chọn phương án B và chúng ta đi tiếp".
  Deadline cho quyết định biến decision latency từ biến ngẫu nhiên thành biến kiểm soát được.
- **Tách rõ lời khen cho kết quả và lời khen cho quá trình.** Lý do: nếu chỉ khen kết quả, tổ chức
  học được rằng ô "quá trình xấu + kết quả tốt" là ô đáng nhắm tới.

### Anti-patterns

**Quyết định không có phương án thứ ba.** Biểu hiện: RFC trình bày một giải pháp và một danh sách lý
do vì sao nó tốt. Cơ chế phá hoại: khi chỉ có một phương án, buổi họp chuyển từ *chọn* sang *phê
duyệt*, và người phản biện bị đặt vào vai người cản đường — chi phí xã hội của việc phản biện tăng,
nên số phản biện giảm, nên chất lượng giảm. Dấu hiệu sớm: trong họp, câu hỏi duy nhất được đặt ra là
"có ai phản đối không" thay vì "còn cách nào khác không".

**Quyết định không ghi lại, nên bị mở lại mỗi quý.** Biểu hiện: y hệt đoạn mở chương. Cơ chế phá
hoại: bộ nhớ tổ chức nằm trong đầu người, và người thì đổi việc; mỗi lần mở lại tốn 100% chi phí của
lần đầu nhưng tạo ra 0% giá trị mới, vì bối cảnh thường không đổi. Tệ hơn: lần mở lại thứ hai thường
được quyết bởi người ít bối cảnh nhất nhưng mới nhất (do có uy tín "cái nhìn tươi mới"). Dấu hiệu
sớm: có người trong team nói "hình như trước mình đã bàn cái này rồi" mà không ai tìm được bản ghi.

**Gộp guess vào decision.** Biểu hiện: một con số quan trọng trong quyết định không truy được nguồn
("tải sẽ tăng 5 lần" — ai nói, dựa trên gì). Cơ chế: quyết định thừa hưởng độ bất định của giả định
yếu nhất, nhưng độ bất định đó bị ẩn đi khi viết thành một con số tròn trịa. Dấu hiệu sớm: các con số
trong tài liệu quyết định đều là số tròn (10x, 50%, 1 triệu user) và không có khoảng.

**Resulting — đánh giá người quyết bằng kết quả một lần.** Biểu hiện: trong retro hoặc performance
review, câu "quyết định đó sai" được dùng khi ý thực là "kết quả xấu". Cơ chế: người trong tổ chức
học rất nhanh rằng cách an toàn nhất là không quyết, hoặc chỉ quyết những thứ chắc thắng, hoặc đẩy
quyết định lên trên. Kết quả sau 2–3 quý là decision latency tăng và không ai nhận accountability.
Dấu hiệu sớm: số RFC do người không phải lead khởi tạo giảm dần về 0.

**Analysis paralysis được biện minh bằng sự cẩn trọng.** Biểu hiện: tuần thứ tư của một spike "để
đánh giá thêm", trong khi cost of delay chưa từng được tính. Cơ chế: chờ thêm thông tin luôn cảm giác
an toàn hơn quyết, vì chi phí của việc chờ không hiện trên bảng nào còn chi phí của quyết định sai
thì có tên người gắn vào. Dấu hiệu sớm: không ai trong cuộc có thể nói ra một con số cho câu "chậm
một tuần thì mất gì".

### Khi nào KHÔNG nên áp dụng

Cấu trúc bảy trường ở trên là chi phí, và có những vùng chi phí đó lớn hơn lợi ích:

- **Quyết định mức thực thi trong ngày, khả đảo trong vài giờ.** Đặt tên biến, chọn thứ tự tham số,
  tách một hàm hay không. Áp cấu trúc quyết định vào đây làm tê liệt công việc và, tệ hơn, dạy team
  rằng quy trình là nghi lễ. Với vùng này, quy tắc đúng là: người viết code quyết, code review là cơ
  chế sửa, không cần bản ghi nào.
- **Trong incident đang diễn ra.** Khi hệ thống đang down, hàm mục tiêu đổi từ "chọn đúng" sang "phục
  hồi nhanh và không làm hỏng thêm". Cấu trúc lúc đó là Incident Command (một người quyết, những
  người khác thực thi và ghi lại), không phải decision matrix. Việc so sánh phương án diễn ra ở
  Postmortem — xem `06-incident-va-metrics.md`.
- **Khi quyết định thực chất đã bị chặn bởi ràng buộc bên ngoài.** Nếu khách hàng ODC đã ký hợp đồng
  chỉ định .NET và Azure, việc dựng ba phương án về ngôn ngữ là diễn kịch. Điều đúng cần làm là ghi
  ràng buộc đó vào trường Constraints, nêu rõ ai áp và trên cơ sở gì, rồi dồn năng lượng vào các
  quyết định còn tự do. Một cách thất bại phổ biến trong ODC là tiêu ba tuần "phân tích lựa chọn" cho
  một thứ đã bị khoá, rồi hết thời gian cho những thứ thật sự còn mở.
- **Khi số lượng quyết định quá lớn để xử lý từng cái.** Team platform nhận 40 yêu cầu ngoại lệ mỗi
  tháng. Cách đúng không phải 40 bản ghi quyết định, mà là **một quyết định mức chính sách** (policy)
  cộng một cơ chế tự động áp dụng. Khi bạn thấy mình ra cùng một loại quyết định lần thứ năm, việc
  cần làm là chuyển nó thành quy tắc, không phải ghi tốt hơn.
- **Khi quyết định là một canh bạc rẻ để lấy thông tin.** Nếu chi phí thử là 2 ngày và thông tin thu
  được sẽ trả lời câu hỏi mà không tài liệu nào trả lời được, thì thử là hành động đúng và phân tích
  là hành động sai. Điều kiện: phải đặt trước tiêu chí dừng và ngưỡng đánh giá, nếu không "thử" biến
  thành một phương án được chọn ngầm.

---

## 2. Reversible vs Irreversible Decisions

### Problem Statement

Một team e-commerce khoảng 25 engineer áp một quy trình duy nhất cho mọi quyết định kỹ thuật: RFC
viết theo template 6 trang, 5 ngày cho phản biện, họp Architecture Review vào chiều thứ Sáu, cần 2
approve từ Senior trở lên. Quy trình này được lập sau một sự cố migration làm mất index trên bảng
đơn hàng, và nó hợp lý cho loại việc đó. Sáu tháng sau, các hiện tượng đếm được:

- **Số RFC tồn đọng chờ review: 14**, trung vị thời gian chờ 11 ngày. Trong đó 9 RFC là những việc
  như "thêm thư viện validate", "đổi cấu trúc folder trong một module", "chọn lib format ngày".
- **Số quyết định đi đường tắt: không đếm được, nhưng nhìn thấy trong git.** Ba thư viện mới xuất
  hiện trong `package.json` mà không có RFC nào. Khi quy trình đắt hơn giá trị nó tạo ra, người ta
  không phản đối quy trình — họ đi vòng qua nó trong im lặng.
- **Chi phí họp Architecture Review: 2 giờ × 6 người × 4 tuần = 48 người-giờ/tháng**, trong đó theo
  đánh giá của chính team, khoảng 60% thời lượng dành cho các quyết định mà nếu chọn sai thì sửa mất
  dưới một ngày.
- **Đồng thời**, ở đầu bên kia: hai tuần trước, một service mới được deploy với contract API công
  khai cho đối tác, đặt tên field theo kiểu `data.info.value`, không qua review nào vì "đây chỉ là
  service nhỏ". Sáu tháng sau, 4 đối tác đã tích hợp và không thể đổi contract nữa.

Đây là bệnh kép: **quy trình nặng đặt sai chỗ và quy trình nhẹ đặt sai chỗ**. Nguyên nhân gốc không
phải quy trình quá nặng hay quá nhẹ, mà là tổ chức không phân loại quyết định trước khi chọn mức quy
trình. Hậu quả nếu không sửa: engineer giỏi mất niềm tin vào quy trình (họ thấy nó chặn việc dễ và
không chặn việc nguy hiểm), và tổ chức tích luỹ dần các quyết định không thể đảo mà chưa ai từng
nhìn kỹ.

### First Principles

**Cơ chế một: tính khả đảo là một tài sản có giá, gọi là option value.** Trong tài chính, một
option (quyền chọn) có giá trị dương vì nó cho bạn quyền hành động sau khi có thêm thông tin, mà
không bắt buộc phải hành động. Giá trị đó tăng theo độ bất định: càng không biết tương lai, quyền
chọn càng đáng tiền. Chuyển sang kỹ thuật: một quyết định khả đảo là một quyết định giữ được option;
một quyết định không khả đảo là một quyết định **bán option đi** để lấy một lợi ích tức thời (đơn
giản hơn, nhanh hơn, rẻ hơn). Việc bán option không sai — sai là bán mà không biết mình đang bán,
hoặc bán rẻ.

Suy ra một quy tắc có thể dùng: **khi độ bất định về yêu cầu còn cao, ưu tiên các quyết định giữ
option; khi yêu cầu đã ổn định, bán option để lấy sự đơn giản.** Đây là lý do vì sao "đừng chọn
database quá sớm" là lời khuyên hợp lý ở tháng thứ hai của một sản phẩm mới và là lời khuyên tệ ở
tháng thứ ba mươi.

**Cơ chế hai: cost of being wrong = xác suất sai × thiệt hại nếu sai × độ khó hoàn tác.** Ba nhân
tử này độc lập với nhau và người ta thường chỉ nhìn một.

| Nhân tử | Nó phụ thuộc vào gì | Cách giảm nó |
|---|---|---|
| Xác suất sai | Chất lượng thông tin, kinh nghiệm team với loại bài toán này | Spike, benchmark, prototype, hỏi người đã làm |
| Thiệt hại nếu sai | Blast radius: bao nhiêu hệ thống/người/dữ liệu bị ảnh hưởng | Thu hẹp phạm vi áp dụng, feature flag, chạy song song |
| Độ khó hoàn tác | Số bên phụ thuộc, dữ liệu đã sinh ra, contract đã công bố | Giữ dữ liệu cũ, versioning, abstraction layer (có giá) |

Điểm quan trọng: **hai trong ba nhân tử là biến bạn thiết kế được, không phải hằng số của bài toán.**
Một quyết định "không thể đảo" thường có thể được biến thành "khả đảo trong 3 tháng đầu" bằng cách
đầu tư vào nhân tử thứ ba. Câu hỏi lead cần đặt không phải "quyết định này có khả đảo không" mà là
"**làm gì để nó khả đảo, và cái đó tốn bao nhiêu**".

**Cơ chế ba: entropy của quyết định — tính khả đảo giảm theo thời gian, không phải hằng số.** Một
quyết định về contract API là two-way door ở tuần đầu (chưa ai tích hợp), là one-way door ở tháng thứ
sáu (4 đối tác đã tích hợp). Một schema là dễ đổi khi có 1.000 dòng, gần như không đổi được khi có
400 triệu dòng và bảy báo cáo tài chính phụ thuộc vào nó. Nói cách khác: **tính khả đảo là hàm giảm
theo lượng thứ đã bám vào quyết định đó** — dữ liệu, tích hợp, code, thói quen người dùng, số điều
khoản hợp đồng.

Hệ quả vận hành rất cụ thể: có một **cửa sổ khả đảo** cho mỗi quyết định, và việc của lead là biết
cửa sổ đó dài bao lâu và chủ động dùng nó. Nếu bạn có 8 tuần trước khi đối tác đầu tiên tích hợp, thì
8 tuần đó là thời gian rẻ nhất để sửa contract, và một buổi review 90 phút ở tuần thứ hai đáng giá
hơn 10 buổi review ở tháng thứ tám.

### Mental Model

**Two-way door / one-way door.** Amazon công khai mô tả cách phân loại này trong thư gửi cổ đông năm
2015 của Jeff Bezos: các quyết định "type 2" (two-way door) có thể đi qua, thấy không ổn thì quay lại
— nên cần được quyết nhanh bởi cá nhân hoặc nhóm nhỏ, chấp nhận sai; các quyết định "type 1"
(one-way door) không quay lại được — nên cần chậm, cần nhiều góc nhìn. Luận điểm gốc đáng chú ý hơn
cả cách phân loại: **cái làm tổ chức lớn chậm lại không phải là dùng quy trình nặng, mà là dùng quy
trình nặng cho type 2** — vì số lượng quyết định type 2 lớn hơn type 1 vài chục lần.

Cách map vào công việc hàng ngày: mỗi khi bạn định mở một buổi họp cho một quyết định, hỏi trước "nếu
chúng ta chọn sai cái này, sáu tuần nữa phát hiện, thì việc quay lại tốn bao nhiêu người-tuần?". Nếu
câu trả lời dưới một tuần, đừng họp — giao cho một người quyết và đi tiếp.

**Blast radius.** Vay từ thuật ngữ hạ tầng: khi cái này nổ, nó phá tới đâu. Bốn vòng tròn đồng tâm
dùng được cho mọi quyết định kỹ thuật: (1) trong một file/module, (2) trong một service, (3) qua
nhiều service hoặc nhiều team, (4) vượt ra ngoài tổ chức — đối tác, khách hàng, cơ quan quản lý.
Blast radius quyết định *số người cần tham gia*; tính khả đảo quyết định *thời gian dành cho quyết
định*. Đây là hai trục khác nhau và trộn chúng là nguồn gốc của phần lớn quy trình sai.

**Ratchet (bánh xe một chiều).** Một số quyết định kỹ thuật hoạt động như bánh xe cóc: đi tiếp thì dễ,
lùi lại thì có chốt chặn. Thêm một field vào API là ratchet (bỏ đi sẽ phá client). Bật một feature
cho toàn bộ user là ratchet (tắt đi là lấy đi cái người dùng đã có). Nhận diện ratchet trước khi kéo
là kỹ năng riêng, và test đơn giản nhất là câu hỏi: "**ai sẽ khó chịu nếu chúng ta hoàn tác, và họ có
quyền gì?**".

### Practical Framework

**Bảng phân loại 4 mức và ngưỡng quy trình tương ứng.** Đây là bảng nên dán vào trang chủ wiki của
team và tham chiếu bằng tên mức trong mọi thảo luận ("cái này mức 2, đừng đưa lên Architecture
Review").

| Mức | Đặc điểm | Ai quyết | Số người review | Thời gian cho quyết định | Rollback plan | Bản ghi |
|---|---|---|---|---|---|---|
| **1. Dễ đảo, tác động cục bộ** | Sửa trong <1 ngày, ảnh hưởng 1 module | Người viết code | 1 (code review thường) | Ngay, trong luồng làm việc | Không cần | Không cần (commit message là đủ) |
| **2. Đảo được, tác động 1 service** | Sửa trong 1–5 ngày, không đụng dữ liệu đã sinh | Tech Lead hoặc owner service | 1–2 người trong team | ≤ 2 ngày | Nêu một câu trong PR | Ghi ngắn (Slack pinned / ADR-lite 10 dòng) |
| **3. Khó đảo, tác động nhiều team** | Sửa mất 2–8 tuần, có bên phụ thuộc nội bộ | Tech Lead + owner các team liên quan; người accountable là 1 tên | 3–5, có ít nhất 1 người ngoài team | 1–2 tuần, có thời hạn phản biện công khai | Bắt buộc, có bước cụ thể và người thực hiện | ADR đầy đủ + điều kiện xét lại |
| **4. Không đảo, tác động toàn hệ thống / ra ngoài tổ chức** | Không quay lại được, hoặc quay lại tốn >1 quý; dữ liệu, contract public, compliance | Head of Engineering / CTO, có chữ ký của business khi liên quan | 5+, có người ngoài engineering (legal, finance, ops) | 2–6 tuần, có pre-mortem bắt buộc | Bắt buộc, đã **diễn thử** trên môi trường giống production | ADR + risk register + biên bản phê duyệt |

**Phân loại các ví dụ kỹ thuật cụ thể.** Bảng này là phần đáng tranh luận nhất, và tranh luận về nó
trong team là bài tập tốt hơn cả việc đọc nó.

| Quyết định | Mức | Vì sao — và cái gì làm mức thay đổi |
|---|---|---|
| Chọn tên biến, tên hàm | 1 | Đổi bằng refactor tự động. Lên mức 2 nếu tên đó xuất hiện trong log được dashboard/alert parse. |
| Chọn thư viện log | 1–2 | Bản thân lib dễ đổi; cái khó đổi là **format log** vì dashboard, alert, và pipeline log phụ thuộc vào nó. Quyết định thật ở đây là format, không phải lib. |
| Chọn ORM | 2–3 | Mức 2 ở service mới, ít query. Lên mức 3 khi đã có ~50+ repository class và các query phức tạp bám vào API riêng của ORM: chi phí đổi là vài tuần, và team phải học lại. |
| Chọn message broker | 3 | Đổi broker kéo theo đổi mô hình delivery guarantee, cách xử lý ordering, cách retry, monitoring, và kỹ năng vận hành. 4–8 tuần. Không phải mức 4 vì vẫn đảo được nếu chấp nhận trả giá. |
| Chọn cloud provider | 4 | Managed service, IAM, networking, hợp đồng cam kết chi tiêu, kỹ năng team, chi phí egress khi rời đi. Thực tế là one-way door trong 2–4 năm. |
| Thay đổi contract API public (breaking) | 4 | Bên tiêu thụ ở ngoài tổ chức. Không kiểm soát được lịch nâng cấp của họ. Nếu **cùng thay đổi đó** làm ở tuần đầu khi chưa ai tích hợp: mức 2. Cùng một thay đổi kỹ thuật, hai mức khác nhau — đây là entropy của quyết định. |
| Migration schema dữ liệu tài chính | 4 | Dữ liệu đã sinh ra không tạo lại được; số dư và bút toán phải khớp qua audit trail; sai là vấn đề pháp lý chứ không phải bug. Cần dual-write, đối soát song song, và rollback đã diễn thử. |
| Đổi ngôn ngữ chính của backend | 4 | Không phải vì code, mà vì **con người**: tuyển dụng, library ecosystem, kỹ năng debug production, và 2–3 năm codebase song ngữ. Chi phí thật nằm ở tầng People, không phải Technology. |
| Bật/tắt một feature cho 100% user | 3–4 | Kỹ thuật là mức 1 (một flag). Ratchet nằm ở kỳ vọng người dùng: lấy đi thứ đã có là mức 3–4. |
| Thêm một index vào bảng lớn | 2 | Đảo được, nhưng blast radius trong lúc chạy có thể lớn → cần cửa sổ và rollback plan, không cần review 5 người. |

**Quy trình phân loại — 4 câu hỏi, 5 phút, làm trước khi mở bất kỳ buổi họp nào:**

```
CLASSIFY (5 phút, làm trước khi lên lịch họp)
1. Nếu chọn sai và phát hiện sau 6 tuần, hoàn tác tốn bao nhiêu người-tuần?
   → <1 : mức 1-2   |   1-8 : mức 3   |   >8 hoặc không hoàn tác được : mức 4
2. Blast radius: module / service / nhiều team / ra ngoài tổ chức?
   → quyết định SỐ NGƯỜI review, không quyết định thời gian
3. Có sinh ra dữ liệu hoặc contract mà bên khác bám vào không?
   → Có = tự động tối thiểu mức 3, kể cả khi code nhỏ
4. Cửa sổ khả đảo còn bao lâu? (tới khi nào thì mức tăng lên?)
   → ghi ngày cụ thể; đây là deadline thật của quyết định
OUTPUT: một dòng "Mức N, người quyết: <tên>, chốt trước: <ngày>"
```

Câu 3 là câu bắt được nhiều lỗi nhất trong thực tế: những quyết định nhìn nhỏ về code nhưng sinh ra
dữ liệu hoặc contract bền vững. Ví dụ điển hình: chọn cách encode ID (số tự tăng vs UUID vs mã có ý
nghĩa) — 20 dòng code, nhưng ID đã phát ra cho khách hàng thì không thu về được.

### Trade-off

**Tốc độ vs độ chắc, đọc lại theo trục khả đảo.** Với mức 1–2, chi phí của một quyết định sai thấp
hơn chi phí của quy trình để tránh nó — nên tối ưu cho tốc độ và cho số lần thử. Với mức 4, ngược
hoàn toàn: một buổi pre-mortem 2 giờ có expected value cực cao vì nó nhân với một thiệt hại rất lớn.
Cái mất khi làm nhanh ở mức 4: những thiệt hại này thường không hiện ra ngay, nên tổ chức không nhận
được tín hiệu sửa sai — chúng hiện ra 18 tháng sau dưới dạng "hệ thống của mình khó thay đổi quá".
Cái mất khi làm chậm ở mức 1–2: engineer đi vòng qua quy trình, và bạn mất luôn cả khả năng quan sát
các quyết định đang được ra.

**Over-engineering để giữ tính khả đảo cũng có giá — và giá đó thường bị đánh giá thấp.** Ví dụ minh
hoạ: team quyết định bọc mọi truy cập dữ liệu qua một lớp repository abstraction "để sau này đổi
database dễ". Chi phí: khoảng 15–20% thời gian phát triển mọi feature liên quan dữ liệu, vĩnh viễn;
thêm một lớp phải đọc khi debug; và mất khả năng dùng các tính năng riêng của database (window
function, JSONB index, full-text search) vì abstraction phải là mẫu số chung nhỏ nhất. Nếu xác suất
đổi database trong 3 năm là 10%, và việc đổi khi không có abstraction tốn 8 người-tuần, thì EV của
việc tránh được = 0,1 × 8 = 0,8 người-tuần. Chi phí của abstraction trong 3 năm với team 5 người, với
40% công việc chạm dữ liệu: khoảng 0,15 × 0,4 × 5 × 150 tuần ≈ 45 người-tuần (số minh hoạ). Tỉ lệ
xấp xỉ 1:56. Đây là ví dụ điển hình của việc **mua option quá đắt**.

Nguyên tắc rút ra: giữ tính khả đảo bằng những biện pháp có chi phí gần bằng không (feature flag,
versioning, không xoá dữ liệu cũ ngay, đặt tên rộng cho contract) thay vì bằng abstraction đắt. Và
khi phải mua option đắt, viết ra xác suất bạn tin là sẽ cần dùng nó — con số đó thường tự bác bỏ
quyết định.

**Standardization vs Flexibility qua lăng kính khả đảo.** Chuẩn hoá (mọi service dùng cùng stack)
giảm cognitive load và làm việc luân chuyển người dễ, nhưng biến một số quyết định mức 2 thành mức 3
vì đổi ở một chỗ nghĩa là đổi chuẩn. Nghiêng về chuẩn hoá khi team dưới 30 người hoặc khi luân chuyển
người nhiều; nghiêng về linh hoạt khi các service có bài toán khác nhau về bản chất và mỗi team có
năng lực vận hành độc lập. Ai chịu phần mất: khi chuẩn hoá quá mức, team có bài toán ngoại lệ trả giá
bằng việc dùng công cụ sai; khi linh hoạt quá mức, người trả giá là người trực on-call phải hiểu bảy
stack lúc 2 giờ sáng.

### Real-world Scenarios

**Tình huống A: contract API cho đối tác — đúng công cụ, sai thời điểm.**

Bối cảnh minh hoạ: công ty logistics, team 25 người. Quân (Senior BE) build một API tracking đơn hàng
để đối tác tích hợp. Quân theo đúng quy trình team: viết RFC, chờ review, chờ họp thứ Sáu. Vì hàng
đợi RFC đang 11 ngày, Quân chờ. Đến ngày thứ 12, áp lực deadline lên cao, Tuấn (Tech Lead) nói "thôi
cứ làm, sửa sau". API lên production tuần sau với contract chưa ai đọc kỹ.

Sai ở đâu: quy trình đã coi đây là một RFC như mọi RFC khác, tức là xếp cùng hàng đợi với "chọn lib
format ngày". Nếu áp bảng phân loại: câu hỏi 3 ("có sinh ra contract mà bên ngoài bám vào không") trả
lời Có → tự động mức 4. Mức 4 không phải là *chậm hơn*, nó là **được ưu tiên vượt hàng đợi và có
người accountable rõ**. Nghịch lý: quyết định quan trọng nhất lại là quyết định bị quy trình làm cho
chậm nhất, rồi vì chậm nên bị bỏ qua hoàn toàn.

Cách xử lý đúng: tách hàng đợi theo mức. Mức 1–2 không vào hàng đợi RFC. Mức 3–4 có SLA riêng: Tech
Lead phải phản hồi trong 2 ngày làm việc, và nếu Architecture Review không họp kịp thì họp riêng
30 phút với 3 người. Kết quả kỳ vọng: hàng đợi RFC giảm từ 14 xuống 3–5 item, và những item còn lại
đều là thứ đáng dành 2 giờ họp.

**Tình huống B: một quyết định mức 4 bị đóng gói dưới dạng ticket mức 2.**

Bối cảnh minh hoạ: fintech, cần đổi cách lưu số tiền từ `float` sang `decimal` trên 6 bảng, khoảng
80 triệu bản ghi. Ticket được tạo với tiêu đề "Fix kiểu dữ liệu tiền", story point 5, giao cho một
Mid-level. Đây là mức 4 được gắn nhãn mức 2: dữ liệu tài chính đã sinh ra, có audit trail, có báo cáo
đối soát với ngân hàng, và làm sai thì hậu quả là sai lệch số dư — vấn đề pháp lý, không phải bug.

Ba tín hiệu lẽ ra phải phát hiện được ngay từ lúc grooming: (1) tiêu đề có chữ "fix" nhưng nội dung
là migration; (2) đối tượng là dữ liệu không tạo lại được; (3) không ai trong buổi grooming trả lời
được "nếu chạy nửa đường mà lỗi thì trạng thái dữ liệu là gì". Câu hỏi (3) là câu hỏi rẻ nhất và
mạnh nhất để phát hiện mức 4 đang ẩn dưới lớp ngôn ngữ mức 2.

Cách xử lý: nâng mức, đổi người accountable lên Tech Lead, và yêu cầu ba thứ trước khi viết dòng code
đầu tiên — script rollback đã chạy thử trên bản sao production, kế hoạch đối soát song song trong 2
tuần, và một mốc "no-go" (nếu sai lệch vượt ngưỡng thì dừng và quay lại). Chi phí thêm khoảng 1,5–2
tuần. So với thiệt hại nếu sai lệch số dư đi vào báo cáo gửi ngân hàng, đây là mức phí bảo hiểm rẻ.

### Best Practices

- **Phân loại trước khi thảo luận, và nói mức ra thành lời.** Câu mở đầu chuẩn cho mọi thảo luận
  quyết định: "theo tôi đây là mức 2, ai thấy nó là mức 3 thì nói ngay". Lý do: tranh luận về mức
  mất 3 phút và định hình toàn bộ chi phí của cuộc thảo luận sau đó.
- **Tách hàng đợi review theo mức, đừng dùng một hàng đợi chung.** Lý do: hàng đợi chung làm quyết
  định quan trọng bị pha loãng bởi số lượng quyết định tầm thường — đây là bài toán queueing kinh
  điển, và giải pháp là phân lớp ưu tiên, không phải làm việc nhanh hơn.
- **Với mọi quyết định mức 3–4, ghi ngày hết hạn cửa sổ khả đảo.** Lý do: nó chuyển một sự thật ngầm
  ("càng để lâu càng khó đổi") thành một mốc thời gian mà lịch có thể nhắc.
- **Đầu tư vào tính khả đảo rẻ trước, đắt sau.** Thứ tự ưu tiên: đặt sau feature flag → giới hạn
  phạm vi triển khai (một khách hàng, một region) → chạy song song hai đường → versioning →
  abstraction layer. Bốn cái đầu gần như miễn phí; cái cuối thường đắt hơn dự tính 3–5 lần.
- **Với mức 4, bắt buộc diễn thử rollback, không chỉ viết rollback plan.** Lý do: một rollback plan
  chưa chạy thử có xác suất hoạt động thấp hơn nhiều so với cảm nhận của người viết nó — nó là code
  chưa được test.
- **Khi một quyết định mức 2 được lặp lại lần thứ ba, biến nó thành chính sách.** Lý do: chi phí ra
  quyết định lặp lại là chi phí thuần; một dòng trong coding convention xoá vĩnh viễn chi phí đó.

### Anti-patterns

**Dùng quy trình one-way door cho mọi thứ.** Biểu hiện: mọi thay đổi cần RFC; mọi RFC cần Architecture
Review; hàng đợi review dài hơn 1 tuần. Cơ chế phá hoại có hai lớp. Lớp một là chi phí trực tiếp:
người-giờ họp và decision latency. Lớp hai, nguy hiểm hơn: **quy trình quá đắt tạo ra hành vi đi vòng
qua quy trình**, và khi đó tổ chức mất hẳn tầm nhìn về các quyết định đang được ra — bạn còn nguyên
quy trình nhưng không còn dữ liệu. Dấu hiệu sớm: có commit đưa vào dependency mới mà không ai nhớ đã
bàn; hoặc câu "cái này nhỏ mà, không cần RFC" trở thành câu quen trong team.

**Dùng quy trình two-way door cho quyết định one-way door.** Biểu hiện: migration dữ liệu tài chính
nằm trong một ticket sprint bình thường; contract public được đặt tên field trong lúc code; bật
feature cho 100% user vì "flag đã sẵn". Cơ chế: các quyết định one-way door thường **trông nhỏ về mặt
code**, nên hệ thống phân loại dựa vào cảm nhận kích thước code sẽ bỏ sót chúng một cách hệ thống.
Dấu hiệu sớm: trong grooming không ai trả lời được câu "hoàn tác cái này thế nào".

**Coi tính khả đảo là thuộc tính cố định.** Biểu hiện: "contract này đổi được mà, mình mới làm xong"
— nói ở tháng thứ tám. Cơ chế: entropy tăng âm thầm, không có sự kiện nào báo hiệu cửa sổ đã đóng.
Dấu hiệu sớm: không ai trong team biết hiện có bao nhiêu bên đang tiêu thụ contract đó.

**Mua option value bằng abstraction đắt mà không viết ra xác suất sẽ dùng.** Biểu hiện: "để sau này
đổi X cho dễ" xuất hiện trong lý do thiết kế mà không kèm ước lượng nào. Cơ chế: chi phí abstraction
là chắc chắn và trả ngay, lợi ích là bất định và ở tương lai xa — bất đối xứng này luôn nghiêng về
phía over-engineering nếu không có kỷ luật định lượng. Dấu hiệu sớm: trong codebase có abstraction
với đúng một implementation, tồn tại trên 12 tháng.

**Dùng mức để né trách nhiệm.** Biểu hiện: một người muốn tự quyết thì gọi việc đó là mức 2; muốn
đẩy lên trên thì gọi là mức 4. Cơ chế: hệ thống phân loại nào cũng bị gaming nếu tiêu chí phân loại
không tường minh. Chống lại bằng cách để tiêu chí phân loại thành 4 câu hỏi có thể kiểm chứng từ bên
ngoài (như khối code ở trên), không để nó là phán đoán chủ quan. Dấu hiệu sớm: mức của một quyết định
thay đổi tuỳ theo ai đang trình bày nó.

### Khi nào KHÔNG nên áp dụng

- **Team dưới 5 người, sản phẩm chưa có product-market fit.** Ở giai đoạn này gần như mọi thứ là
  two-way door, vì lượng dữ liệu và số bên phụ thuộc còn nhỏ, và xác suất toàn bộ hướng sản phẩm bị
  bỏ là cao. Dựng bảng 4 mức lúc này là tạo ra chi phí quy trình cho một tổ chức chưa có vấn đề đó.
  Ngoại lệ không thương lượng: những thứ chạm tiền và dữ liệu định danh khách hàng vẫn là mức 4 ngay
  từ ngày đầu, vì hậu quả pháp lý không tỉ lệ với kích thước công ty.
- **Khi tổ chức không có năng lực thực thi mức 4.** Nếu bạn không có môi trường staging giống
  production, không có bản sao dữ liệu để diễn thử, thì "bắt buộc diễn thử rollback" là một yêu cầu
  không thể đáp ứng, và mọi yêu cầu không thể đáp ứng sẽ bị làm cho có. Thứ tự đúng: xây năng lực
  trước, áp chuẩn sau. Trong thời gian chưa có năng lực, cách giảm rủi ro là thu nhỏ phạm vi (migrate
  1% dữ liệu trước) chứ không phải viết thêm tài liệu.
- **Trong ODC nơi khách hàng giữ quyền quyết định kiến trúc.** Bảng phân loại vẫn hữu ích nhưng đổi
  mục đích: nó không dùng để tự quyết mà để **truyền đạt rủi ro lên khách hàng**. Câu hỏi thực tế
  không phải "ai quyết" mà "làm sao để khách hàng hiểu đây là quyết định mức 4 và dành cho nó thời
  gian tương ứng". Xem thêm phần bối cảnh ODC ở chủ đề 5 và `02-communication.md`.
- **Khi cửa sổ khả đảo đã đóng.** Nếu contract đã có 4 đối tác tích hợp, việc phân loại lại nó thành
  mức 4 không có tác dụng gì ngoài việc làm mọi người thấy tệ. Lúc đó bài toán đã đổi: không còn là
  quyết định mà là quản lý di sản — versioning, deprecation timeline, hỗ trợ song song. Áp khung
  quyết định vào một việc đã hết tự do là tiêu thời gian vào nơi không đổi được kết quả.

---

## 3. Decision Matrix và ra quyết định có tiêu chí

### Problem Statement

Buổi họp kiến trúc dài 3 giờ, 5 người, chủ đề chọn message broker. Ghi âm lại và đếm, bạn sẽ thấy
cấu trúc quen thuộc: phút 0–20 mỗi người nêu công cụ mình biết; phút 20–90 tranh luận về throughput
và "Kafka nặng lắm"; phút 90–140 lạc sang chuyện Kubernetes; phút 140–170 quay lại, ai cũng mệt;
phút 170–180 người có thâm niên cao nhất nói "thôi dùng Kafka đi, chỗ nào cũng dùng", không ai phản
đối. Kết quả có thể đúng. Quá trình thì không tạo ra thứ gì tái dùng được.

Hiện tượng đếm được của một tổ chức thiếu ra quyết định có tiêu chí:

- **Tỉ lệ thời gian họp dành cho tranh luận không hội tụ.** Đo thô: đếm số lần trong buổi họp mà cùng
  một luận điểm được nêu lại bởi cùng một người. Nếu ≥3 lần, cuộc thảo luận đang chạy vòng, và nguyên
  nhân gần như luôn là hai người đang tối ưu hai tiêu chí khác nhau mà không ai nói ra tiêu chí đó.
- **Không ai viết được lý do bằng một câu.** Sau khi chốt, hỏi ba người "vì sao chọn cái này". Nếu ba
  câu trả lời khác nhau về tiêu chí (một người nói vì throughput, một người nói vì team quen, một
  người nói vì "chuẩn ngành"), tổ chức vừa đồng ý về phương án mà không đồng ý về lý do — và lần sau
  gặp bài toán tương tự, không suy ra được gì.
- **Quyết định lật ngược khi có người mới.** Người mới có tiêu chí khác, và vì tiêu chí cũ chưa từng
  được viết ra, không có gì để tranh luận trừ sở thích.
- **Số phương án được xem xét thực chất là 2.** Với hơn 3 phương án và không có bảng, con người mặc
  định gạt bớt xuống 2 để so được trong đầu, và việc gạt bớt đó diễn ra không ai nhận thức được.

Hậu quả: quyết định trở thành hàm số của người có tiếng nói to nhất và thâm niên cao nhất, không phải
hàm số của bài toán. Trong bối cảnh Việt Nam, cơ chế này còn mạnh hơn một bậc vì phản biện người lớn
tuổi hoặc cấp trên có chi phí xã hội cao — nên kể cả khi người ít thâm niên có lập luận tốt hơn, lập
luận đó không vào được cuộc họp.

### First Principles

**Cơ chế một: working memory của con người chứa được khoảng 4 phần tử, không phải 7.** Con số "7±2"
của Miller đã được các nghiên cứu sau (Cowan) hiệu chỉnh xuống khoảng 4 chunk cho working memory hoạt
động. Một bài toán so sánh 4 phương án × 5 tiêu chí là 20 ô. Để chọn, bạn phải giữ đồng thời 20 giá
trị và các trọng số tương đối — vượt xa dung lượng. Cái xảy ra khi vượt dung lượng không phải là bạn
suy nghĩ chậm hơn; nó là **não tự động đơn giản hoá bài toán mà không báo cho bạn biết**, thường bằng
hai cách: bỏ bớt tiêu chí (chỉ còn 1–2 tiêu chí nổi bật nhất), hoặc bỏ bớt phương án (còn 2). Bảng
giấy là một bộ nhớ ngoài; nó không làm bạn thông minh hơn, nó chỉ ngăn việc đơn giản hoá âm thầm.

**Cơ chế hai: tranh luận không hội tụ khi tiêu chí chưa được đồng thuận, và đây là vấn đề logic, không
phải vấn đề thái độ.** Nếu Minh tối ưu cho "thời gian ra production" và Khoa tối ưu cho "durability",
thì cả hai đều đúng trong hệ tiêu chí của mình, và không có lượng dữ liệu nào làm họ hội tụ — vì họ
đang trả lời hai câu hỏi khác nhau. Mọi dữ liệu mới đưa vào sẽ được mỗi người đọc theo tiêu chí của
mình, và tranh luận kéo dài vô hạn. Suy ra một quy tắc rất mạnh: **thứ tự đúng là chốt tiêu chí trước,
liệt kê phương án sau.** Đảo thứ tự này là nguyên nhân của phần lớn buổi họp kiến trúc 3 giờ không có
kết quả.

**Cơ chế ba: motivated reasoning và vì sao trọng số phải được chốt trước.** Khi con người đã có phương
án ưa thích (thường vì quen, vì mới học, vì đã dùng ở công ty cũ), họ không bịa dữ liệu — họ **chọn
tiêu chí và trọng số** làm phương án đó thắng, và quá trình này diễn ra dưới mức ý thức. Đây là lý do
một bảng decision matrix được điền sau khi đã biết đáp án còn tệ hơn không có bảng: nó khoác cho một
lựa chọn sẵn có bộ áo của phân tích khách quan, khiến người khác khó phản biện hơn.

**Cơ chế bốn: lượng hoá làm ẩn giả định, nên phải có bước kiểm tra độ nhạy.** Một điểm số 4/5 cho
"ecosystem" là kết tinh của hàng chục phán đoán, và nó xuất hiện trên bảng như một dữ kiện. Cách duy
nhất giữ được sự trung thực là **sensitivity analysis**: nếu đổi trọng số hoặc điểm số trong biên
hợp lý mà kết luận không đổi, quyết định là *robust* và bạn có thể đi tiếp mà không cần chính xác hơn.
Nếu kết luận đổi, bạn vừa tìm ra chính xác thứ cần điều tra thêm — và đó là thông tin đáng giá hơn cả
bảng.

### Mental Model

**Weighted Decision Matrix.** Bảng phương án × tiêu chí, mỗi tiêu chí có trọng số, mỗi ô có điểm,
tổng có trọng số quyết định thứ hạng. Điều đáng hiểu về mô hình này: **giá trị của nó không nằm ở con
số cuối cùng.** Nó nằm ở ba tác dụng phụ — buộc phát biểu tiêu chí, buộc so sánh tất cả phương án
trên tất cả tiêu chí (không bỏ sót ô nào), và tạo ra một hiện vật để phản biện vào từng ô thay vì
tranh luận vào cảm nhận tổng thể. Khi Khoa không đồng ý, Khoa không nói "tôi thấy Kafka tốt hơn" mà
nói "ô durability của RabbitMQ tôi cho 3 chứ không phải 4, vì lý do X" — tranh luận thu nhỏ lại thành
một ô kiểm được.

**Satisficing với ngưỡng loại (knockout criteria).** Trước khi cho điểm, tách các tiêu chí thành hai
loại: **knockout** (không đạt là loại, không cần tính điểm) và **scoring** (cho điểm và cân). Ví dụ:
"phải hỗ trợ at-least-once delivery" là knockout cho hệ thanh toán; "ecosystem tốt" là scoring. Việc
tách này giảm số phương án phải cân trước khi bước vào phần đắt của bài toán, và nó bảo vệ khỏi một
lỗi kinh điển của weighted matrix: **một phương án dở ở tiêu chí sống-còn vẫn có thể thắng tổng điểm
nếu nó xuất sắc ở các tiêu chí phụ.** Weighted sum không mã hoá được "điều kiện cần" — knockout làm
việc đó.

**Sensitivity analysis như một dạng stress test.** Mượn tư duy từ kỹ thuật: đừng hỏi "hệ thống có
chạy không", hỏi "nó vỡ ở đâu". Với decision matrix: đừng hỏi "phương án nào thắng", hỏi "**cần đổi
gì để phương án khác thắng, và điều đó có khả năng đúng không**". Nếu chỉ cần đổi trọng số của một
tiêu chí từ 20% xuống 15% là kết quả đảo, thì quyết định của bạn thực chất đang treo trên một con số
mà không ai bảo vệ được.

### Practical Framework

**Quy trình 7 bước. Bước 1–3 làm trước, trong một buổi riêng, không có tên công cụ nào trên bảng.**

```
BƯỚC 1 — Viết bài toán (30 phút, 1 người viết, cả nhóm đọc)
  Một đoạn: hệ thống này phải làm gì, ở quy mô nào, trong bao lâu, ràng buộc gì.
  Không nhắc tên công cụ nào. Nếu không viết nổi, chưa đủ hiểu để chọn.

BƯỚC 2 — Liệt kê tiêu chí (45 phút, cả nhóm)
  - Mỗi người viết riêng 5 tiêu chí trước khi nói ra (chống anchoring bởi người nói đầu).
  - Gộp, loại trùng. Giữ tối đa 6-7 tiêu chí.
  - Tách: KNOCKOUT (đạt/không đạt) vs SCORING (cho điểm 1-5).
  - Định nghĩa từng tiêu chí bằng một câu KIỂM ĐƯỢC.
    Xấu:  "dễ vận hành"
    Tốt:  "team hiện tại tự vận hành được không cần thuê thêm người,
           đo bằng: có runbook cho 5 sự cố phổ biến và 1 người có thể học trong 2 tuần"

BƯỚC 3 — Gán trọng số CÔNG KHAI, tổng = 100% (20 phút)
  - Mỗi người phân bổ 100 điểm riêng, viết vào giấy/form.
  - Hiện tất cả cùng lúc. Chỗ lệch > 15 điểm giữa hai người = có bất đồng về
    mục tiêu, không phải về công cụ. Bàn chỗ đó trước, bàn công cụ sau.
  - Chốt trọng số. GHI TIMESTAMP. Từ đây trọng số ĐÓNG BĂNG.

--- ĐẾN ĐÂY MỚI ĐƯỢC NÓI TÊN CÔNG CỤ ---

BƯỚC 4 — Liệt kê phương án (≥3, có "không làm gì"/"giữ nguyên")
BƯỚC 5 — Áp knockout: loại phương án không đạt, ghi lý do một dòng
BƯỚC 6 — Cho điểm 1-5 từng ô, mỗi ô có 1 dòng CĂN CỨ (link benchmark, doc, kinh nghiệm ai)
         Ô nào không có căn cứ → đánh dấu (?) và ước lượng chi phí để biết
BƯỚC 7 — Tính tổng, rồi SENSITIVITY:
         a) đổi từng trọng số ±5 điểm → kết quả đổi không?
         b) hạ điểm phương án thắng 1 bậc ở tiêu chí nặng nhất → còn thắng không?
         c) nếu kết quả đảo dễ → xác định đúng 1-2 ô cần điều tra, đặt deadline
OUTPUT: bảng + 1 đoạn kết luận + điều kiện xét lại + tên người accountable
```

**Bảng mẫu hoàn chỉnh: chọn message broker cho hệ thống thanh toán.**

Bối cảnh minh hoạ: ví điện tử Việt Nam, 40 engineer, khoảng 800.000 giao dịch/ngày, peak 1.200 TPS
trong campaign, cần luồng bất đồng bộ cho ghi nhận giao dịch, gửi thông báo, và đối soát cuối ngày.
Team backend 12 người, không có ai từng vận hành Kafka ở production, có một người từng dùng RabbitMQ.
Không có team platform riêng. Deadline nghiệp vụ: luồng mới phải lên production trong 10 tuần.

*Tiêu chí knockout (chốt ngày 08/06, trước khi khảo sát):*

| Knockout | Định nghĩa kiểm được | Kafka | RabbitMQ | SQS + SNS | Redis Streams |
|---|---|---|---|---|---|
| At-least-once delivery có bảo đảm | Có tài liệu chính thức về guarantee + ack cơ chế rõ | Đạt | Đạt | Đạt | Đạt (yếu hơn khi failover) |
| Message không mất khi 1 node chết | Replication ≥ 2, đã có case study công khai | Đạt | Đạt (quorum queue) | Đạt (managed) | **Không đạt** — AOF/replica async, có cửa sổ mất dữ liệu |
| Chạy được trên hạ tầng hiện tại | VM tự quản, không dùng managed service ngoài VPC vì yêu cầu compliance nội bộ | Đạt | Đạt | **Không đạt** — dữ liệu giao dịch ra ngoài VPC, cần xin phê duyệt compliance ~8 tuần | Đạt |

Hai phương án bị loại ở bước 5, mỗi cái một dòng lý do. Ghi lại quan trọng: nếu 6 tháng sau ràng buộc
compliance đổi, SQS quay lại bàn mà không cần khảo sát lại từ đầu.

*Ma trận có trọng số — 3 phương án còn lại (điểm 1–5, số minh hoạ):*

| Tiêu chí (trọng số) | Kafka | RabbitMQ | Giữ nguyên: DB polling + cron |
|---|---|---|---|
| **Durability & bảo đảm không mất message (25%)** | 5 | 4 | 3 |
| **Độ quen của team hiện tại (20%)** | 2 | 4 | 5 |
| **Chi phí vận hành: người + hạ tầng (20%)** | 2 | 3 | 4 |
| **Ecosystem: client lib, tooling, tài liệu, người tuyển được (10%)** | 5 | 4 | 3 |
| **Hỗ trợ ordering theo key (15%)** | 5 | 2 | 4 |
| **Thời gian ra production (10%)** | 2 | 4 | 5 |
| **TỔNG CÓ TRỌNG SỐ** | **3,45** | **3,50** | **3,95** |

Cách tính cột Kafka để đối chiếu: 5(0,25) + 2(0,20) + 2(0,20) + 5(0,10) + 5(0,15) + 2(0,10)
= 1,25 + 0,40 + 0,40 + 0,50 + 0,75 + 0,20 = **3,45**.

*Căn cứ cho các ô gây tranh cãi nhất (bắt buộc có, mỗi ô một dòng):*

| Ô | Điểm | Căn cứ |
|---|---|---|
| Kafka / độ quen team | 2 | 0/12 người từng vận hành production; Hà đã học qua course nhưng chưa xử lý sự cố thật |
| Kafka / chi phí vận hành | 2 | Cần 3 broker + ZooKeeper hoặc KRaft; ước lượng 0,5 FTE vận hành trong 6 tháng đầu (số minh hoạ) |
| RabbitMQ / ordering | 2 | Ordering chỉ đảm bảo trong một queue với một consumer; bài toán cần ordering theo `merchant_id` → phải tự shard queue, thêm phức tạp |
| Giữ nguyên / durability | 3 | Không mất dữ liệu (nằm trong DB) nhưng có cửa sổ chậm 30–60s và đã gây 2 lần trùng lặp trong 3 tháng |
| Giữ nguyên / ordering | 4 | Xử lý tuần tự theo `merchant_id` được, nhưng throughput trần khoảng 300 TPS theo đo thực tế (?) — cần benchmark lại |

*Sensitivity analysis — phần quan trọng nhất và thường bị bỏ:*

| Thay đổi giả định | Kafka | RabbitMQ | Giữ nguyên | Kết luận đổi? |
|---|---|---|---|---|
| Nguyên bản | 3,45 | 3,50 | **3,95** | — |
| Ordering từ 15% → 25%, độ quen 20% → 10% | 3,75 | 3,30 | **3,85** | Chưa đổi, nhưng khoảng cách thu hẹp về 0,10 |
| Thêm tiêu chí "chịu được 5× tải hiện tại trong 18 tháng" (15%, lấy từ trọng số chi phí vận hành) | **3,80** | 3,35 | 3,50 | **Đảo** — Kafka thắng |
| Giữ nguyên: điểm throughput hạ từ 4 → 2 sau benchmark | 3,45 | **3,50** | 3,50 | Đảo — bỏ phương án giữ nguyên |

Kết quả sensitivity nói điều mà tổng điểm không nói: **quyết định này gần như hoàn toàn phụ thuộc vào
một câu hỏi chưa được trả lời — tải trong 18 tháng tới là bao nhiêu, và giải pháp DB polling có trần
ở đâu.** Đó là thông tin đáng đi mua.

*Kết luận thực tế của bảng này (và đây là dạng kết luận đúng của một decision matrix tốt):*

Quyết định không phải "chọn Kafka" hay "chọn RabbitMQ". Quyết định là: **dành 1 tuần benchmark trần
throughput của luồng DB polling hiện tại với dữ liệu thật, và cùng lúc yêu cầu PO đưa ra dự báo tăng
trưởng giao dịch 18 tháng.** Nếu trần > 3× tải peak dự kiến, giữ nguyên và đầu tư vào việc sửa lỗi
trùng lặp (rẻ nhất, ra production trong 2 tuần). Nếu trần < 2×, đi Kafka và chấp nhận thuê/mượn năng
lực vận hành trong 6 tháng đầu. Chi phí của tuần benchmark: khoảng 1 người-tuần. Nó thay đổi một
quyết định mức 3 có chi phí 8–16 người-tuần.

### Trade-off

**Minh bạch vs bị gaming.** Lượng hoá làm quyết định kiểm toán được: người ngoài đọc bảng biết được
lý do, và phản biện được vào từng ô. Nhưng nó cũng tạo ra một bề mặt tấn công mới — trọng số. Một
người đủ khéo có thể điều khiển kết luận mà không nói câu nào không đúng sự thật. Cách xử lý không
phải bỏ bảng mà là ba biện pháp: chốt trọng số trước khi có tên phương án, cho mỗi người phân bổ
trọng số riêng rồi hiện đồng thời, và luôn chạy sensitivity. Cái mất: quy trình dài hơn khoảng 1–1,5
giờ. Với quyết định mức 3–4, đây là mức phí rẻ.

**Consensus về tiêu chí vs speed.** Bước 2–3 (chốt tiêu chí và trọng số) tốn khoảng 1 giờ và cảm giác
như đang trì hoãn. Trên thực tế nó thường **rút ngắn** tổng thời gian, vì nó chuyển tranh luận từ
không gian vô hạn (mọi thứ về mọi công cụ) sang không gian có biên (6 tiêu chí, 3 phương án). Điều
kiện để bước này không đáng làm: khi bài toán chỉ có 2 phương án thật và cả nhóm đã đồng ý ngầm về
tiêu chí — lúc đó viết bảng là nghi lễ.

**Bảng đơn giản vs bảng đầy đủ.** Càng nhiều tiêu chí, bảng càng "công bằng" và càng vô dụng: 10 tiêu
chí với trọng số trung bình 10% mỗi cái làm mọi phương án tiến về điểm trung bình, vì các khác biệt
thật bị pha loãng bởi các tiêu chí không quan trọng. Nguyên tắc: **6 tiêu chí là trần**, và nếu tiêu
chí thứ 7 quan trọng thì phải có tiêu chí nào bị bỏ ra. Việc buộc phải bỏ ra là phần có giá trị nhất
của ràng buộc này.

**Định lượng vs phán đoán chuyên gia.** Với bài toán mà team đã làm 5 lần, một Senior có phán đoán
tốt hơn bất kỳ bảng nào và bảng chỉ là chi phí. Với bài toán mới, phán đoán chuyên gia là ngoại suy
từ bối cảnh khác — và bảng là cơ chế phát hiện chỗ ngoại suy không hợp lệ. Nghiêng về phán đoán khi
độ tương tự với bài toán cũ cao và hậu quả sai thấp; nghiêng về bảng khi bài toán mới hoặc khi có
nhiều người phải cùng cam kết vào kết quả.

### Real-world Scenarios

**Tình huống: chọn framework frontend ở một ODC, và trọng số bị chỉnh sau.**

Bối cảnh minh hoạ: một ODC 60 người ở Đà Nẵng, dự án mới cho khách hàng Nhật, 18 tháng. Nam (Tech
Lead) muốn dùng framework A vì vừa học và muốn có nó trong CV của team. Nam làm đúng thủ tục: lập
decision matrix với 6 tiêu chí, trình bày trong buổi review, framework A thắng 4,1 so với 3,6.

Trang (Engineering Manager) đọc bảng và đặt ba câu hỏi:

1. "Trọng số này chốt ngày nào, so với ngày em bắt đầu khảo sát?" — Nam thừa nhận điền cùng lúc.
2. "Tiêu chí 'khả năng tuyển người ở Đà Nẵng' có trọng số 5%. Dự án 18 tháng, mình dự kiến cần thêm
   4 người. Vì sao 5%?" — không có câu trả lời dựa trên dữ liệu.
3. "Nếu tăng tiêu chí đó lên 20% và giảm 'developer experience' từ 25% xuống 10% thì kết quả thế
   nào?" — kết quả đảo, framework B thắng.

Điều đáng chú ý: Nam không gian dối. Nam thật lòng tin framework A tốt hơn, và các trọng số Nam chọn
đều cảm giác hợp lý với Nam. Đây chính là motivated reasoning — nó không có cảm giác gì cả từ bên
trong. Cái phát hiện được nó không phải sự trung thực mà là **thứ tự thao tác và sensitivity**.

Cách Trang xử lý (đúng): không nói "em đang thiên vị" — câu đó tấn công vào con người và sẽ tạo phòng
vệ. Trang nói: "Bảng của em làm tốt phần khó nhất là liệt kê được tiêu chí. Vấn đề là nó chưa qua
stress test. Em chạy lại phần trọng số theo cách này: bốn người trong team, mỗi người phân 100 điểm
riêng, gửi cho anh trước 10h mai, anh mở cùng lúc. Nếu framework A vẫn thắng thì mình đi A và anh
support hết mình." Kết quả: trung bình trọng số của bốn người đặt "khả năng tuyển người" ở 18%, và
framework B thắng. Nam thực thi B mà không mất mặt, vì cái bị bác không phải Nam mà là một bộ trọng
số.

Ba tầng đọc cùng tình huống này:

- **IC**: "bảng bị chỉnh cho khớp đáp án" — đúng, nhưng dừng ở đó thì chỉ tạo ra sự bất tín.
- **Tech Lead**: "quy trình của mình thiếu bước chốt trọng số trước" — đây là cấp độ sửa được.
- **Manager**: "vì sao một Tech Lead cần đưa công nghệ mới vào CV của team?" — có thể là dấu hiệu
  của vấn đề career path và learning budget, thuộc `03-team-leadership.md`. Nếu chỉ chặn quyết định
  mà không xử lý động lực nằm dưới, nó sẽ quay lại ở dự án sau.

### Best Practices

- **Tiêu chí trước, phương án sau — không có ngoại lệ cho quyết định mức 3–4.** Lý do: đây là cơ chế
  duy nhất chuyển tranh luận từ so sánh công cụ (không hội tụ) sang so sánh mục tiêu (hội tụ được).
- **Mỗi người phân bổ trọng số độc lập rồi hiện đồng thời.** Lý do: chống anchoring bởi người phát
  biểu đầu tiên và người có thâm niên cao nhất. Trong bối cảnh Việt Nam, đây là biện pháp có tỉ lệ
  hiệu quả/chi phí cao nhất trong cả chương — nó cho người ít thâm niên một kênh phát biểu không
  đối đầu.
- **Mỗi tiêu chí phải có một câu định nghĩa kiểm được.** Lý do: tiêu chí mơ hồ ("scalable", "modern")
  cho phép mỗi người cho điểm theo hệ riêng, và bảng trở thành tổng của các thang đo khác nhau.
- **Mỗi ô điểm phải có một dòng căn cứ; ô không có căn cứ đánh dấu (?).** Lý do: nó biến "chỗ mình
  không biết" thành một danh sách có thể lên kế hoạch điều tra, thay vì một con số giả tin cậy.
- **Luôn chạy sensitivity và ghi kết quả vào ADR.** Lý do: kết luận robust cho phép dừng phân tích với
  lương tâm sạch; kết luận không robust chỉ ra đúng chỗ cần điều tra. Cả hai đều tiết kiệm thời gian.
- **Kết luận của bảng có thể là "đi mua thêm một thông tin cụ thể", và đó là kết luận tốt.** Lý do:
  bảng đo được cả độ nhạy của quyết định với thông tin thiếu — dùng nó để nhắm spike thay vì spike
  bừa.
- **Giữ bảng đã bị loại.** Lý do: khi ràng buộc đổi (compliance, ngân sách, tải), bạn quay lại được
  bảng cũ và chỉ cập nhật phần đổi, thay vì khảo sát lại 4 phương án từ đầu.

### Anti-patterns

**Chỉnh trọng số sau khi đã biết mình muốn chọn gì.** Biểu hiện: trọng số và điểm được điền trong
cùng một buổi, bởi cùng một người, sau khi người đó đã đọc tài liệu về phương án ưa thích. Cơ chế phá
hoại nghiêm trọng hơn việc chọn sai: bảng cho lựa chọn đó **tính chính danh của phân tích khách quan**,
làm người phản biện phải tấn công vào một hiện vật trông có vẻ chặt chẽ, nên số phản biện giảm. Nói
cách khác, công cụ chống bias bị chuyển thành công cụ khuếch đại bias. Dấu hiệu sớm: không có timestamp
tách biệt giữa trọng số và điểm; hoặc trọng số có những con số rất cụ thể (17%, 23%) — dấu vết của
việc tinh chỉnh để đạt tổng mong muốn.

**Bảng có 10+ tiêu chí trọng số gần đều.** Biểu hiện: mọi phương án tổng điểm chênh nhau dưới 0,2.
Cơ chế: pha loãng — các khác biệt quyết định bị các tiêu chí thứ yếu trung hoà, và kết quả về gần
trung bình, nghĩa là bảng không phân biệt được gì. Dấu hiệu sớm: người đọc bảng phải nhìn tới chữ số
thập phân thứ hai để biết ai thắng.

**Weighted sum không có knockout.** Biểu hiện: một phương án không đáp ứng yêu cầu sống-còn (mất
message, không qua được compliance) vẫn nằm trên bảng và thắng nhờ điểm cao ở developer experience.
Cơ chế: phép cộng có trọng số mã hoá quan hệ *bù trừ*, nhưng điều kiện cần không bù trừ được — không
có lượng developer experience nào bù cho việc mất giao dịch. Dấu hiệu sớm: có tiêu chí mà cả nhóm nói
"cái này bắt buộc" nhưng nó vẫn được cho điểm 1–5.

**Bảng làm xong rồi bỏ, quyết định thật diễn ra ngoài phòng.** Biểu hiện: bảng được trình bày trong
họp, sau đó CTO nhắn riêng "mình đi cái kia nhé". Cơ chế: nếu quyết định thật không dùng bảng, cả tổ
chức học được rằng bảng là nghi lễ, và lần sau không ai làm nghiêm túc. Dấu hiệu sớm: quyết định cuối
không tham chiếu tới bất kỳ ô nào trong bảng.

**Dùng bảng để tránh nhận accountability.** Biểu hiện: "bảng chọn thế, không phải tôi chọn". Cơ chế:
bảng là công cụ hỗ trợ phán đoán, không phải chủ thể quyết định; khi một người nấp sau nó, không còn
ai để hỏi vì sao và learning loop bị cắt. Dấu hiệu sớm: trường Accountable trong ADR để trống hoặc
ghi "Architecture Review Board".

### Khi nào KHÔNG nên áp dụng

- **Quyết định mức 1–2 theo bảng phân loại ở chủ đề 2.** Chi phí của decision matrix là 2–4 giờ của
  3–5 người. Với một quyết định sửa được trong một ngày, đây là lỗ ròng. Cách đúng: người owner quyết,
  ghi một dòng lý do trong PR.
- **Khi có một tiêu chí áp đảo tất cả.** Nếu compliance yêu cầu dữ liệu phải nằm trong lãnh thổ Việt
  Nam, thì bài toán không phải weighted matrix mà là một filter: liệt kê các phương án đạt, chọn cái
  rẻ nhất. Dựng bảng 6 tiêu chí lúc này tạo ra ảo giác rằng các tiêu chí kia có thể bù cho tiêu chí
  bắt buộc.
- **Khi số phương án là 1 trên thực tế.** Nếu tổ chức đã chuẩn hoá một stack và việc dùng khác đi cần
  phê duyệt cấp CTO, thì bài toán thật là "có xin ngoại lệ hay không" — một bài toán khác, cần lập
  luận về vì sao trường hợp này khác biệt, không cần bảng so sánh.
- **Khi các tiêu chí không thể so sánh trên cùng thang.** Có những quyết định mà một trục là tiền và
  một trục là rủi ro pháp lý hoặc niềm tin của khách hàng. Cộng chúng lại bằng trọng số là một phép
  toán sai về bản chất, và con số cuối sẽ che mất điều duy nhất quan trọng. Với loại này, cách đúng
  là trình bày trade-off dưới dạng câu điều kiện cho người có quyền quyết: "để tiết kiệm 800 triệu,
  chúng ta chấp nhận xác suất 5% bị phạt hành chính — anh có chấp nhận đánh đổi đó không?".
- **Khi thời gian ra quyết định ngắn hơn thời gian làm bảng.** Trong incident, hoặc khi cửa sổ cơ hội
  còn 24 giờ, dùng phiên bản rút gọn: viết 3 phương án và một tiêu chí quan trọng nhất lên whiteboard,
  quyết trong 15 phút, ghi lại để review sau. Bảng đầy đủ đến sau khi quyết định đã hết giá trị là
  công việc khảo cổ, không phải ra quyết định.

---

## 4. Cost-Benefit Analysis và ngôn ngữ tiền

### Problem Statement

Một Tech Lead trình bày trước founder: "Module thanh toán nợ kỹ thuật nặng, code khó bảo trì, không
có test, mỗi lần sửa là sợ. Em cần 6 tuần để refactor." Founder nghe, gật đầu, và nói: "Anh hiểu, nhưng
quý này mình phải ra tính năng trả góp, đối thủ đã có rồi. Refactor để quý sau nhé." Quý sau, cùng
cuộc hội thoại đó lặp lại. Sau bốn quý, module thanh toán vẫn nguyên, và bây giờ mỗi tính năng liên
quan mất gấp ba lần thời gian so với ước lượng.

Hiện tượng đếm được của một tổ chức nơi engineering không nói bằng ngôn ngữ tiền:

- **Tỉ lệ đề xuất technical health được duyệt.** Đếm trong 4 quý: bao nhiêu đề xuất về nợ kỹ thuật,
  hạ tầng, tooling được đưa ra và bao nhiêu được cấp thời gian. Nếu dưới 30%, engineering đang thua
  hệ thống một cách hệ thống, không phải thua từng lần.
- **Lệch giữa ước lượng và thực tế trên các module nợ nặng.** Nếu module A có tỉ lệ ước
  lượng/thực tế là 1:1,2 và module B là 1:3, thì B đang thu một khoản thuế mỗi sprint, và khoản thuế
  đó chưa từng xuất hiện trên bất kỳ báo cáo tài chính nào.
- **Số giờ engineer dành cho việc lặp lại thủ công mỗi tuần.** Hỗ trợ CS tra cứu, chạy script tay,
  fix dữ liệu. Nếu là 15 giờ/tuần cho một team 10 người, đó là 7,5% capacity, tương đương 0,75 người
  — nhưng nó không nằm trong headcount plan nào.
- **Ai đang làm việc quy đổi.** Đây là chỉ số quan trọng nhất. Nếu Tech Lead trình bày bằng ngôn ngữ
  "code khó bảo trì" và founder tự quy đổi sang "chi phí bao nhiêu", thì phép quy đổi đó vẫn diễn ra
  — chỉ là bởi người ít thông tin kỹ thuật nhất, và kết quả gần như luôn nghiêng về phía đánh giá
  thấp chi phí kỹ thuật.

Đây là điểm mấu chốt của chủ đề này: **việc quy đổi kỹ thuật sang chi phí/lợi ích luôn xảy ra.** Câu
hỏi duy nhất là ai làm nó. Nếu lead không làm, người khác sẽ làm — và họ làm với dữ liệu tệ hơn, mô
hình sai hơn, thường theo hướng bất lợi cho engineering, bởi lẽ chi phí kỹ thuật ẩn còn lợi ích kinh
doanh thì hiện.

### First Principles

**Cơ chế một: tổ chức phân bổ tài nguyên theo các đơn vị so sánh được, và đơn vị so sánh được duy
nhất trong doanh nghiệp là tiền.** Founder không so sánh "code sạch" với "tính năng trả góp" — hai
thứ này không cùng thang đo. Họ so sánh những gì họ quy được về doanh thu, chi phí, rủi ro. Một đề
xuất không quy được về ba trục đó sẽ không tham gia được cuộc thi phân bổ, bất kể nó đúng đến đâu về
mặt kỹ thuật. Đây không phải sự thiếu hiểu biết của business — đó là cách mọi hệ thống phân bổ tài
nguyên khan hiếm phải hoạt động.

**Cơ chế hai: bất đối xứng về khả năng quan sát.** Lợi ích của một tính năng mới là *hiện*: có thể
demo, có thể đo bằng số user, có thể kể cho nhà đầu tư. Chi phí của nợ kỹ thuật là *ẩn*: nó biểu hiện
dưới dạng ước lượng cao hơn, nhưng ước lượng cao hơn lại bị đọc thành "team làm chậm". Với hai dòng
tín hiệu có độ quan sát chênh lệch như vậy, kết quả tích luỹ qua nhiều quý là tất yếu, không phải
ngẫu nhiên. Cách duy nhất sửa được cơ chế này là **làm cho chi phí ẩn trở nên quan sát được** —
nghĩa là đo và trình bày nó, đều đặn, bằng số.

**Cơ chế ba: cost of delay là chi phí thật, và nó thường lớn hơn chi phí phát triển.** Với một tính
năng tạo ra dòng giá trị V mỗi tháng, hoãn nó một tháng làm mất V. Nhưng có bốn dạng cost of delay
khác nhau về hình dạng, và trộn chúng là lỗi phổ biến:

| Dạng | Hình dạng | Ví dụ | Hệ quả cho ưu tiên |
|---|---|---|---|
| Tuyến tính | Mất V mỗi đơn vị thời gian | Tính năng tăng conversion đều đặn | Làm sớm hơn tốt hơn, nhưng không có cliff |
| Có deadline cứng (cliff) | Giá trị gần như 0 sau mốc | Tuân thủ quy định có hiệu lực 01/01; tính năng cho campaign 11/11 | Trước mốc là tất cả, sau mốc là vô nghĩa |
| Giảm dần theo cạnh tranh | Giá trị giảm khi đối thủ ra trước | Tính năng dễ copy trong thị trường đông | Cửa sổ hẹp, tốc độ quan trọng hơn hoàn hảo |
| Tích luỹ nợ (chi phí tăng theo thời gian chờ) | Chi phí sửa tăng theo thời gian | Nợ kỹ thuật, migration, dữ liệu sai đang tiếp tục sinh ra | Hoãn không phải tiết kiệm, là vay với lãi |

Dạng thứ tư là dạng mà nợ kỹ thuật thuộc về, và là dạng bị mô hình hoá sai nhiều nhất. Khi founder
nói "refactor để quý sau", giả định ngầm là chi phí refactor không đổi. Trên thực tế nó tăng, vì
trong quý đó thêm code mới bám vào phần cũ. Việc của lead là **nói ra tốc độ tăng đó bằng số**.

**Cơ chế bốn: rủi ro là một khoản chi phí có thể quy đổi, gọi là expected loss.** Chi phí rủi ro =
xác suất × thiệt hại. Một hệ thống có 15% xác suất mỗi năm gặp sự cố mất 4 giờ, mỗi giờ downtime mất
150 triệu doanh thu (số minh hoạ), đang mang một chi phí rủi ro kỳ vọng 0,15 × 4 × 150 = 90 triệu/năm.
Con số này có thể đặt cạnh chi phí của việc giảm rủi ro và so sánh được. Đây là ngôn ngữ mà bộ phận
tài chính và bảo hiểm dùng hàng ngày, và nó là cây cầu mạnh nhất giữa engineering và business.

### Mental Model

**Bốn loại chi phí, ba loại lợi ích.** Đây là danh mục để không bỏ sót — mỗi lần làm cost-benefit, đi
qua đủ bảy dòng, dòng nào bằng 0 thì viết 0 và giải thích tại sao.

| Loại | Cách quy đổi | Cạm bẫy phổ biến |
|---|---|---|
| **C1. Engineering cost** | Người-tháng × chi phí đầy đủ một engineer/tháng (lương + thuế + phúc lợi + chi phí chung, thường 1,3–1,5× lương gross) | Dùng lương gross thay vì chi phí đầy đủ → thiếu 30–50% |
| **C2. Chi phí hạ tầng** | Chênh lệch chi phí cloud/license/vận hành mỗi tháng × số tháng | Bỏ qua chi phí tăng theo tải; bỏ qua egress và backup |
| **C3. Chi phí rủi ro** | Xác suất × thiệt hại, tính theo năm | Chỉ tính rủi ro của phương án mới, không tính rủi ro của việc **không làm gì** |
| **C4. Cost of delay** | Giá trị/đơn vị thời gian × thời gian trễ, theo đúng hình dạng ở bảng trên | Áp dạng tuyến tính cho mọi thứ |
| **B1. Doanh thu tăng** | Số user ảnh hưởng × tỉ lệ chuyển đổi × giá trị trung bình | Đếm doanh thu gộp thay vì doanh thu tăng thêm (incremental) |
| **B2. Chi phí giảm** | Giờ người tiết kiệm × chi phí giờ, hoặc chi phí hạ tầng giảm | Tính giờ tiết kiệm mà không kiểm xem giờ đó có được dùng cho việc giá trị cao hơn không |
| **B3. Rủi ro giảm** | Chi phí rủi ro trước − chi phí rủi ro sau | Giả định giảm về 0; hầu hết biện pháp chỉ giảm một phần |

**Nguyên tắc "so với cái gì" (counterfactual).** Mọi con số chi phí và lợi ích chỉ có nghĩa khi so với
một baseline cụ thể. Baseline mặc định phải là **không làm gì**, và điều quan trọng nhất là: **không
làm gì cũng có chi phí, và chi phí đó thường bị đặt bằng 0.** Đây là chỗ đề xuất technical health hay
thua nhất — người ta so "6 tuần refactor" với "6 tuần làm tính năng" nhưng quên rằng nhánh thứ hai
cũng mang theo một khoản chi phí tích luỹ.

**Trục thời gian và payback period.** Một khoản chi 6 tuần tạo ra tiết kiệm 20% thời gian phát triển
trên module đó. Nếu module đó chiếm 40% công việc của team 8 người, tiết kiệm là 0,2 × 0,4 × 8 = 0,64
người/tháng ≈ 0,64 người-tháng/tháng. Khoản chi là 6 tuần × 2 người = 3 người-tháng. Payback ≈ 3 /
0,64 ≈ **4,7 tháng**. Đây là dạng câu mà founder xử lý được ngay: "khoản này hoàn vốn sau 5 tháng và
sinh lợi từ tháng thứ 6". Không cần NPV, không cần discount rate — payday period là đủ cho hầu hết
quyết định ở tầm team.

### Practical Framework

**Quy trình 6 bước để dựng một cost-benefit dùng được trong 90 phút:**

```
1. XÁC ĐỊNH BASELINE (10 phút)
   "Nếu không làm gì trong 12 tháng tới, điều gì xảy ra?" — viết bằng số.
   Đây là cột so sánh. Không có nó, mọi con số khác vô nghĩa.

2. LIỆT KÊ C1-C4 CHO TỪNG PHƯƠNG ÁN (30 phút)
   Mỗi con số ghi 3 thứ: giá trị, nguồn, độ tin cậy (Cao/TB/Thấp).
   Dùng KHOẢNG cho số không chắc: "8-14 người-tuần", không "11 người-tuần".

3. LIỆT KÊ B1-B3 (20 phút)
   Cùng quy tắc. Nếu một lợi ích không quy được, đưa vào mục 5, KHÔNG bỏ.

4. TÍNH: net, payback period, và ĐIỂM HOÀ VỐN của biến bất định nhất
   "Phương án này có lợi nếu tỉ lệ chuyển đổi tăng ít nhất 0,4%."
   → chuyển từ dự báo (dễ sai) sang ngưỡng (kiểm được sau khi làm)

5. GHI PHẦN KHÔNG LƯỢNG HOÁ ĐƯỢC — thành một mục riêng có tiêu đề
   Mỗi cái viết theo mẫu: <yếu tố> — <hướng tác động> — <ai chịu ảnh hưởng>
   VÀ một câu: "để phần này thay đổi kết luận, nó cần đáng ít nhất X đồng"

6. VIẾT SLIDE MỘT TRANG cho người quyết (mẫu ở dưới)
```

**Ví dụ tính toán: "trả technical debt module thanh toán 6 tuần" vs "làm tính năng trả góp 6 tuần".**

Bối cảnh minh hoạ: fintech Việt Nam, team 8 engineer, 2 người sẽ làm việc này. Mọi con số dưới đây là
**số minh hoạ** để thể hiện cách tính, không phải dữ liệu thật của công ty nào.

*Giả định chung (số minh hoạ):*

| Tham số | Giá trị | Nguồn |
|---|---|---|
| Chi phí đầy đủ 1 engineer/tháng | 60 triệu VND | lương gross 42tr × 1,4 |
| 6 tuần × 2 engineer | 3 người-tháng = **180 triệu VND** | tính trực tiếp |
| Doanh thu hiện tại | 4 tỉ VND/tháng | báo cáo tài chính |
| % công việc team chạm module thanh toán | 40% | phân tích 3 sprint gần nhất theo label Jira |
| Tỉ lệ ước lượng/thực tế của module thanh toán | 1 : 2,4 | so 18 ticket trong 3 sprint |
| Tỉ lệ đó ở các module khác | 1 : 1,3 | cùng nguồn |
| Số incident 6 tháng có nguồn từ module thanh toán | 5 / 11 tổng | incident log |
| Thiệt hại trung bình 1 incident (doanh thu mất + xử lý thủ công + hoàn tiền) | 120 triệu VND | 5 incident gần nhất |

*Phương án A — Refactor module thanh toán (6 tuần, 2 người):*

| Dòng | Tính toán | Giá trị (12 tháng) |
|---|---|---|
| C1 Engineering | 3 người-tháng × 60tr | −180 tr |
| C2 Hạ tầng | không đổi | 0 |
| C3 Rủi ro **mới** phát sinh do refactor | 20% × 120tr (1 incident do refactor gây ra) | −24 tr |
| C4 Cost of delay của trả góp | Trả góp lùi 6 tuần; xem phương án B: 90tr/tháng × 1,5 tháng | −135 tr |
| B2 Chi phí giảm — hết thuế ước lượng | Giảm tỉ lệ từ 1:2,4 xuống 1:1,4. Tiết kiệm ≈ 40% công việc × 8 người × (1 − 1,4/2,4) = 0,4 × 8 × 0,42 = 1,34 người-tháng/tháng... nhưng chỉ 40% công việc nên tính lại: capacity thu hồi ≈ 1,34 người-tháng/tháng | +1,34 × 60tr × 12 ≈ **+965 tr** (giá trị dưới dạng capacity, không phải tiền mặt) |
| B3 Rủi ro giảm | Incident từ module này: 5/6 tháng = 10/năm → giảm còn ~4/năm. 6 × 120tr | +720 tr |
| **Net 12 tháng** | | **+1.346 tr** (trong đó 965tr là capacity, 720tr là tiền/rủi ro thật) |

*Phương án B — Làm tính năng trả góp (6 tuần, 2 người):*

| Dòng | Tính toán | Giá trị (12 tháng) |
|---|---|---|
| C1 Engineering | 3 người-tháng × 60tr | −180 tr |
| C2 Hạ tầng + phí đối tác tài chính | 15tr/tháng × 10,5 tháng còn lại | −158 tr |
| C3 Rủi ro | Xây tính năng mới trên module nợ nặng: 35% × 120tr | −42 tr |
| C4 Cost of delay của refactor | Nợ tiếp tục tích luỹ 1,5 tháng: thuế ước lượng 1,34 người-tháng/tháng × 1,5 | −121 tr |
| B1 Doanh thu tăng | Giả định 6% giao dịch chuyển sang trả góp, biên phí thêm 1,8%: 4.000tr × 6% × 1,8% = 4,3tr/tháng... quá nhỏ. Mô hình thứ hai: trả góp mở thêm giỏ hàng giá trị cao, tăng GMV 2,5% → 100tr/tháng, biên 3% → **3tr/tháng**. Mô hình thứ ba (được PO dùng): tăng conversion 0,8 điểm % → +90tr doanh thu/tháng, biên 12% → 10,8tr/tháng | +10,8tr × 10,5 ≈ **+113 tr** |
| B3 Rủi ro giảm | 0 | 0 |
| **Net 12 tháng** | | **−388 tr** |

Điều đáng chú ý nhất trong ví dụ này không phải kết luận, mà là **ba mô hình khác nhau cho B1 cho ra
ba con số chênh nhau 3,6 lần** (3tr, 4,3tr, 10,8tr mỗi tháng). Đây là trạng thái bình thường của mọi
cost-benefit về tính năng mới. Cách xử lý đúng không phải chọn con số dễ chịu nhất, mà là:

1. Trình bày cả ba mô hình và nói rõ mô hình nào đang được dùng.
2. Chuyển sang **ngưỡng hoà vốn**: "tính năng trả góp có lợi hơn refactor nếu nó tạo ra ít nhất
   **150 triệu doanh thu tăng thêm mỗi tháng**". Con số 150tr này là câu hỏi cho PO, không cho
   engineering — và nó là câu hỏi cụ thể mà PO có thể đi tìm dữ liệu để trả lời.
3. Nếu PO không tự tin về ngưỡng đó, phương án thứ ba xuất hiện: làm một phiên bản trả góp nhỏ nhất
   trong 1,5 tuần để đo conversion thật, rồi mới quyết. Chi phí 0,75 người-tháng để loại bỏ độ bất
   định lớn nhất của cả bài toán.

*Cách trình bày trước founder/CFO — một trang, thứ tự này:*

```
[1 câu kết luận và đề nghị]
"Em đề nghị dành 6 tuần cho module thanh toán trước, và làm trả góp bắt đầu từ giữa tháng 9."

[3 con số]
- Module thanh toán đang lấy 1,34 người/tháng dưới dạng thuế ước lượng — bằng 17% capacity team.
- Nó là nguồn của 5/11 incident 6 tháng qua; mỗi incident khoảng 120 triệu.
- Refactor hoàn vốn sau ~4,5 tháng; sau đó team nhanh hơn 17% vĩnh viễn trên phần việc đó.

[Chi phí và cái phải hy sinh — nói trước khi bị hỏi]
- Chi phí: 180 triệu, và trả góp chậm 6 tuần.
- Nếu trả góp tạo hơn 150 triệu doanh thu tăng thêm/tháng, quyết định của em SAI và mình nên
  làm trả góp trước. Anh/chị có dữ liệu nào để ước lượng con số đó không?

[Phần em không lượng hoá được]
- Sau refactor, module có test → team dám sửa → thời gian phản hồi yêu cầu business nhanh hơn.
  Em không quy được ra tiền, nhưng để phần này không quan trọng thì nó phải đáng dưới 100 triệu/năm.

[Điều kiện xét lại]
- Nếu sau 3 tuần refactor mà tỉ lệ ước lượng/thực tế chưa cải thiện, em dừng và báo lại.
```

Bốn đặc điểm làm slide này hoạt động: (1) kết luận ở dòng đầu; (2) tự nêu chi phí và cái phải hy sinh
trước khi bị hỏi — điều này mua được rất nhiều tín nhiệm; (3) nói rõ điều kiện mà mình sai, biến cuộc
họp từ thuyết phục thành cùng giải bài toán; (4) có điều kiện dừng, giảm rủi ro cảm nhận của người
phê duyệt.

### Trade-off

**Lượng hoá cái không lượng hoá được: bỏ qua vs gán số ẩu.** Ba thứ thường không quy được ra tiền:
morale, learning/năng lực team, và optionality (giữ được lựa chọn cho tương lai). Hai cách xử lý đều
sai: bỏ qua chúng (thì chúng bị coi bằng 0, và trong một phép so sánh, 0 là một phát biểu mạnh) hoặc
gán một con số bịa ("morale đáng 200 triệu").

Cách xử lý thứ ba, dùng được: **đưa chúng vào một mục riêng có tiêu đề, mô tả hướng tác động, và
chuyển câu hỏi thành ngưỡng.** Mẫu câu: "chúng ta chưa quy được giá trị của việc team học được cách
vận hành Kafka. Nhưng phép tính hiện tại đang cho phương án khác thắng 200 triệu. Nghĩa là để chọn
Kafka một cách hợp lý, chúng ta phải tin rằng năng lực đó đáng hơn 200 triệu cho công ty trong 2 năm
tới. Tôi tin là có, vì lý do X. Anh/chị nghĩ sao?" Cấu trúc này giữ nguyên độ nghiêm ngặt của phép
tính, không giả vờ có số, và đặt phán đoán vào tay người có quyền phán đoán. Đây cũng là cách duy nhất
để phần định tính không bị âm thầm loại khỏi bàn.

**Độ chính xác vs độ hữu dụng.** Một mô hình 3 dòng với sai số ±50% thường ra quyết định giống một mô
hình 40 dòng với sai số ±20%, và tốn 1/10 thời gian. Nghiêng về mô hình thô khi hai phương án chênh
nhau trên 2 lần (lúc đó sai số không đảo được kết luận); nghiêng về mô hình chi tiết khi chúng chênh
dưới 30% — nhưng lúc đó, kết luận đúng thường là "hai phương án tương đương, chọn cái khả đảo hơn"
chứ không phải "tính chính xác hơn để tìm người thắng".

**Ngôn ngữ tiền tạo ảnh hưởng vs nó thu hẹp cuộc thảo luận.** Được: engineering vào được bàn phân bổ
tài nguyên. Mất: khi mọi thứ phải quy ra tiền, những thứ khó quy đổi (accessibility, chất lượng nội
tại, sự tử tế với người dùng) bị đẩy ra rìa một cách hệ thống. Ai chịu phần mất: thường là người dùng
ở nhóm nhỏ và engineer phải sống trong codebase. Biện pháp giảm nhẹ: giữ một bucket capacity không
cần biện minh bằng ROI (xem chủ đề 6) — đây là cách thừa nhận rằng không phải mọi thứ đáng làm đều
chứng minh được bằng số.

### Real-world Scenarios

**Tình huống: hai cách trình bày cùng một đề xuất trước founder.**

Bối cảnh minh hoạ: startup e-commerce, Series A, founder kiêm PO. Minh (Tech Lead) cần 4 tuần để tách
service tìm kiếm ra khỏi monolith vì search chiếm 60% CPU và làm chậm toàn bộ site trong campaign.

*Phiên bản nói sai:*

> **Minh:** Anh ơi, em cần 4 tuần để tách search service. Monolith đang bị search chiếm hết CPU, mỗi
> lần campaign là toàn bộ site chậm. Kiến trúc hiện tại không scale được, mình phải làm sớm.
>
> **Founder:** Chậm là chậm bao nhiêu? Khách có phàn nàn không?
>
> **Minh:** Cũng có, mà chủ yếu là em thấy P95 lên 4 giây. Với lại code search bây giờ rất khó sửa,
> mỗi lần thêm filter là phải test cả site.
>
> **Founder:** Ừ nhưng 11/11 còn 5 tuần nữa, mình đang cần tính năng voucher combo. Search chậm thì
> mình tăng server lên trước đi, xong campaign rồi tính.
>
> **Minh:** Tăng server không giải quyết được, vì vấn đề là kiến trúc...
>
> **Founder:** Thì mình cứ tăng đi, đắt hơn mấy chục triệu chứ có bao nhiêu. Voucher combo quan trọng
> hơn.

Ba lỗi cụ thể: (1) Minh trình bày bằng ngôn ngữ trạng thái hệ thống ("không scale được") thay vì hậu
quả kinh doanh; (2) khi bị hỏi về khách hàng, Minh trả lời bằng metric kỹ thuật mà không quy đổi;
(3) Minh phản đối phương án của founder ("tăng server không giải quyết được") mà không định lượng —
nên nó thành ý kiến chống ý kiến, và trong tình huống đó người có quyền thắng. Đáng chú ý: founder
không hề vô lý. Với thông tin founder có, quyết định của founder là hợp lý.

*Phiên bản nói đúng:*

> **Minh:** Anh cho em 10 phút về search trước 11/11. Em có ba con số. Một: trong campaign 6/6, P95
> của trang tìm kiếm là 4,2 giây, và theo log của mình, tỉ lệ bỏ giỏ ở nhóm session có search chậm
> hơn 3 giây cao hơn nhóm còn lại 9 điểm phần trăm. Với lưu lượng 11/11 dự kiến, số session bị ảnh
> hưởng khoảng 40 nghìn. Nếu 9 điểm đó đúng và giá trị đơn trung bình 450 nghìn, phần doanh thu rủi
> ro khoảng 1,6 tỉ trong 3 ngày campaign. Em nói rõ: con số 9 điểm này là tương quan, chưa phải nhân
> quả, độ tin cậy trung bình.
>
> **Founder:** Con số đó lớn đấy. Nhưng 4 tuần thì trễ voucher combo.
>
> **Minh:** Đúng, nên em không đề nghị 4 tuần. Em có ba phương án. Một: tăng server — em đã thử trên
> staging, giảm P95 xuống 3,1 giây, tốn thêm 28 triệu/tháng, làm trong 2 ngày. Hai: tách search 4
> tuần, P95 xuống dưới 1 giây, nhưng voucher combo trễ và em không khuyến nghị trước campaign. Ba:
> làm read replica riêng cho search plus cache kết quả 60 giây — 5 ngày làm, một người, P95 ước
> khoảng 1,5 giây, tốn thêm 12 triệu/tháng. Phương án ba là em đề nghị: 5 ngày của Khoa, voucher
> combo vẫn kịp, và em vẫn giữ được việc tách service cho sau campaign.
>
> **Founder:** Sao không làm phương án ba từ đầu?
>
> **Minh:** Vì nó là giải pháp tạm, và nó tạo thêm một chỗ phải bảo trì. Em chấp nhận cái đó để đổi
> lấy 11/11. Nhưng em muốn ghi lại: sau campaign, nếu mình không tách search trước tháng 3, cache 60
> giây sẽ bắt đầu gây lỗi hiển thị tồn kho và em sẽ phải quay lại xin thời gian, lúc đó đắt hơn.
>
> **Founder:** Ghi vào đâu?
>
> **Minh:** Em tạo một dòng trong decision log, ngày xét lại 15/1, người accountable là em. Anh chỉ
> cần biết là nó không mất.

Cái làm phiên bản này hoạt động không phải sự lưu loát. Nó là bốn thứ có cấu trúc: quy đổi sang doanh
thu rủi ro với độ tin cậy được khai báo trung thực; ba phương án có chi phí và thời gian cụ thể; tự
nêu nhược điểm của phương án mình đề nghị; và tạo một điều kiện xét lại để món nợ tạm thời không biến
mất khỏi bộ nhớ tổ chức. Founder trong phiên bản này không bị thuyết phục — founder được cung cấp đủ
để tự quyết đúng.

### Best Practices

- **Luôn có cột "không làm gì" với số khác 0.** Lý do: baseline bằng 0 là giả định mạnh nhất và sai
  nhất trong mọi cost-benefit về nợ kỹ thuật.
- **Dùng khoảng và khai báo độ tin cậy cho mỗi con số.** Lý do: một con số đơn lẻ truyền đạt độ chắc
  mà bạn không có; khi nó sai, uy tín của bạn mất theo, và lần sau không ai tin cả những con số đúng.
- **Chuyển dự báo thành ngưỡng hoà vốn.** Lý do: bạn không dự báo được conversion, nhưng bạn tính
  được "cần bao nhiêu thì quyết định này đúng" — và ngưỡng là thứ kiểm chứng được sau 1 tháng.
- **Quy nợ kỹ thuật thành capacity, rồi mới thành tiền.** Lý do: "17% capacity của team" là đơn vị mà
  cả engineering và business đều hiểu và không tranh cãi được; "code khó bảo trì" thì không.
- **Nêu cái mình phải hy sinh trước khi bị hỏi.** Lý do: nó chuyển vai của bạn từ người bán hàng sang
  người phân tích, và đó là thay đổi có tác dụng lớn nhất trong toàn bộ chủ đề này.
- **Đo và báo cáo một chỉ số technical health đều đặn, không chỉ khi cần xin thời gian.** Lý do: nếu
  con số chỉ xuất hiện lúc bạn cần gì đó, nó bị đọc là công cụ đàm phán. Nếu nó xuất hiện hàng tháng
  trong cùng một báo cáo, nó thành dữ kiện.
- **Ghi lại cả các cost-benefit đã bị bác.** Lý do: khi đề xuất bị bác ba lần và lần thứ tư hậu quả
  xảy ra, bản ghi là thứ chuyển cuộc thảo luận từ "engineering làm chậm" sang "chúng ta đã chọn rủi
  ro này một cách có ý thức" — và điều đó bảo vệ được cả tổ chức, không chỉ bạn.

### Anti-patterns

**Số giả chính xác dùng để thắng tranh luận.** Biểu hiện: "refactor này tiết kiệm 847 triệu/năm".
Cơ chế phá hoại: độ chính xác giả tạo tạo ra tin cậy giả tạo; khi ai đó truy nguồn và thấy nó xây trên
ba giả định không có căn cứ, uy tín của bạn sụp không chỉ ở con số đó mà ở toàn bộ khả năng phân tích
— và tổn thất đó lâu dài hơn nhiều so với lợi ích của việc thắng một buổi họp. Dấu hiệu sớm: con số có
3 chữ số có nghĩa nhưng không có mục giả định kèm theo.

**Chỉ tính chi phí của phương án mới, không tính chi phí của trạng thái hiện tại.** Biểu hiện: bảng
so sánh có cột A, cột B, không có cột "giữ nguyên". Cơ chế: status quo bias được hợp thức hoá bằng
cấu trúc bảng. Dấu hiệu sớm: mọi đề xuất thay đổi đều trông đắt.

**Đếm giờ tiết kiệm như tiền đã thu.** Biểu hiện: "tự động hoá này tiết kiệm 10 giờ/tuần, tức 480
triệu/năm". Cơ chế: giờ tiết kiệm chỉ thành giá trị nếu nó được dùng cho việc giá trị cao hơn; nếu nó
biến thành thời gian trống rải rác 15 phút, giá trị thực gần 0. Dấu hiệu sớm: không ai nói được 10 giờ
đó sẽ chuyển sang việc gì.

**Lượng hoá để né phán đoán.** Biểu hiện: dành hai tuần xây mô hình tài chính cho một quyết định mà
người quyết đã sẵn sàng quyết trong 30 phút nếu có ba con số thô. Cơ chế: mô hình phức tạp là chỗ trú
an toàn khỏi việc phải nói "tôi tin là nên làm X". Dấu hiệu sớm: mô hình có nhiều tham số hơn số dữ
liệu thật bạn có.

**Nói bằng metric kỹ thuật với người không có hàm quy đổi.** Biểu hiện: trình bày P95, error rate,
cyclomatic complexity cho founder hoặc CFO. Cơ chế: người nghe không im lặng vì đồng ý mà vì không có
cách đánh giá; họ sẽ quyết bằng thứ họ hiểu được, thường là deadline. Dấu hiệu sớm: sau khi bạn trình
bày 10 phút, câu hỏi đầu tiên là "vậy nó ảnh hưởng gì đến ngày ra mắt".

**Dùng cost-benefit cho mọi thứ, kể cả những thứ hiển nhiên.** Biểu hiện: viết mô hình ROI cho việc
thêm CI. Cơ chế: chi phí phân tích vượt giá trị, và tệ hơn, nó dạy tổ chức rằng mọi thứ phải chứng
minh bằng số — trong khi có những thực hành mà chi phí chứng minh cao hơn chi phí làm luôn.

### Khi nào KHÔNG nên áp dụng

- **Khi chi phí thực hiện thấp hơn chi phí phân tích.** Việc cần 2 ngày và không có rủi ro dữ liệu:
  làm, đừng phân tích. Ngưỡng thực dụng: nếu chi phí phương án dưới khoảng 1 người-tuần và khả đảo,
  quyết định thuộc mức 1–2 và không cần cost-benefit.
- **Khi quyết định thuộc vùng không thương lượng.** Bảo mật dữ liệu cá nhân, tuân thủ pháp luật, an
  toàn tiền của khách hàng. Ở đây phép tính ROI không chỉ vô dụng mà có hại: nó tạo tiền lệ rằng
  những thứ này có thể bị đánh đổi nếu con số đúng. Cách trình bày đúng: "đây là ràng buộc, không
  phải phương án", và đưa nó vào trường Constraints ở chủ đề 1.
- **Khi bạn không có dữ liệu và không có cách lấy dữ liệu trong khung thời gian quyết định.** Xây một
  mô hình từ ba giả định bịa ra không tạo ra thông tin, nó chỉ chuyển sự bất định thành một con số làm
  người khác tin nhầm. Lúc đó nói thẳng: "tôi không lượng hoá được cái này. Đây là phán đoán của tôi
  dựa trên X, độ tin cậy thấp, và đây là cách rẻ nhất để mua thêm thông tin."
- **Trong thương lượng scope với khách hàng ODC nơi giá đã cố định.** Nếu hợp đồng fixed-price đã ký,
  phép tính ROI cho khách hàng thường không di chuyển được gì, vì lợi ích thuộc về khách mà chi phí
  thuộc về bạn — hai bên có hàm mục tiêu khác nhau. Ngôn ngữ hiệu quả hơn trong bối cảnh này là rủi
  ro và tác động lên lịch giao hàng của khách, không phải ROI.
- **Khi mối quan hệ tin cậy chưa được xây.** Một cost-benefit chi tiết trình bày bởi người mà
  stakeholder chưa tin sẽ bị đọc là mưu tính, không phải phân tích. Với người mới nhận vai lead, thứ
  tự đúng là: giao đúng hẹn vài lần để tạo tín nhiệm, đồng thời báo cáo số liệu đều đặn, rồi mới dùng
  số liệu đó để xin thay đổi phân bổ tài nguyên. Xem `01-self-leadership.md` về xây tín nhiệm và
  `02-communication.md` về trình bày với stakeholder.

---

## 5. Risk Assessment

### Problem Statement

Ngày thứ tư của một migration database ở một công ty ví điện tử. Kế hoạch dự kiến 3 ngày. Sự cố hiện
tại: bảng `transactions` đã chuyển sang cluster mới, nhưng job đối soát cuối ngày vẫn đọc cluster cũ,
nên báo cáo gửi ngân hàng đối tác thiếu 6 giờ giao dịch. Không ai phát hiện trong 4 ngày. Người phát
hiện là nhân viên bên đối tác.

Trong tài liệu kế hoạch migration có một mục tên "Risks", ba dòng: "Rủi ro downtime — mitigate: làm
ban đêm", "Rủi ro mất dữ liệu — mitigate: backup trước", "Rủi ro chậm tiến độ — mitigate: theo dõi
sát". Ba dòng này được viết trong 5 phút cuối buổi họp kickoff và không được mở lại lần nào. Không có
dòng nào về các hệ thống hạ nguồn đọc dữ liệu đó.

Hiện tượng đếm được của một tổ chức làm risk assessment kiểu nghi lễ:

- **Tỉ lệ incident có nguyên nhân đã từng được nêu trong một tài liệu rủi ro.** Ở nhiều tổ chức con
  số này là 0 — không phải vì rủi ro không được nêu, mà vì rủi ro được nêu ở mức khái quát không dùng
  được ("rủi ro về dữ liệu") nên không dẫn tới hành động nào.
- **Số rủi ro trong register có mức "Medium".** Nếu trên 70% ở giữa, register không phân biệt được gì
  và mọi ưu tiên đều mất — đây là dấu hiệu người viết đang né việc phải nói một rủi ro là nghiêm trọng.
- **Ngày cập nhật cuối của tài liệu rủi ro.** Nếu bằng ngày tạo, tài liệu đó là artifact tuân thủ, không
  phải công cụ.
- **Số rủi ro có tên người cụ thể ở cột chủ rủi ro.** Nếu là 0, không rủi ro nào đang được ai theo.
- **Số rủi ro có "tín hiệu sớm" cụ thể.** Đây là cột phân biệt register dùng được với register trang
  trí. Không có tín hiệu sớm thì bạn chỉ biết rủi ro đã thành hiện thực khi thiệt hại đã xảy ra — như
  trường hợp trên, khi đối tác gọi điện.

### First Principles

**Cơ chế một: rủi ro = xác suất × tác động, và con người sai ở cả hai nhân tử theo hướng dự đoán
được.**

| Bias | Cơ chế | Biểu hiện trong kỹ thuật | Cách bù |
|---|---|---|---|
| **Availability bias** | Ta ước lượng xác suất bằng độ dễ nhớ lại ví dụ, không bằng tần suất thật | Sau một incident về Redis, cả team đánh giá quá cao rủi ro Redis và bỏ qua rủi ro chưa từng xảy ra | Dùng dữ liệu tần suất từ incident log, không dùng cảm nhận |
| **Normalcy bias** | Ta giả định tương lai giống hiện tại vì hiện tại chưa hỏng | "Chạy 3 năm không sao" dùng làm bằng chứng cho an toàn, trong khi tải đã tăng 8 lần | Hỏi "cái gì đã đổi so với lúc thiết kế" |
| **Bỏ qua rủi ro đuôi dài** | Sự kiện xác suất thấp × thiệt hại rất lớn có EV cao nhưng cảm giác không đáng bàn | Mất toàn bộ backup; leak dữ liệu KYC; sai lệch số dư diện rộng | Tính EV tường minh; với thiệt hại vượt ngưỡng tồn tại, xử theo ràng buộc chứ không theo EV |
| **Planning fallacy** | Ta ước lượng theo kịch bản tốt nhất và bỏ qua tỉ lệ cơ sở | Migration 3 ngày thành 9 ngày; đây là hiện tượng có tính hệ thống | Dùng tỉ lệ cơ sở: các migration trước của chính team mất bao lâu so với ước lượng |
| **Optimism về khả năng phát hiện** | Ta giả định sẽ biết ngay khi có vấn đề | Sai lệch dữ liệu 4 ngày mới có người ngoài phát hiện | Với mỗi rủi ro, hỏi "cái gì sẽ báo cho tôi, trong bao lâu" |

Dòng cuối là dòng quan trọng nhất và ít được xử lý nhất. Rủi ro thực tế không phải "có sự cố" mà là
"**có sự cố mà không ai biết trong N ngày**". Thiệt hại gần như luôn tỉ lệ với N.

**Cơ chế hai: ba tầng bất định khác nhau, cần ba loại phản ứng khác nhau.** Đây là phân biệt của
Knight, và nó có hệ quả vận hành trực tiếp:

| Tầng | Định nghĩa | Ví dụ kỹ thuật | Phản ứng đúng |
|---|---|---|---|
| **Risk** | Biết các kết cục và biết (hoặc ước lượng được) phân phối | Tỉ lệ lỗi của một API đối tác, đã có 6 tháng số liệu | Tính toán, mua bảo hiểm, thiết kế theo ngưỡng |
| **Uncertainty** | Biết các kết cục nhưng không biết phân phối | Tải trong campaign đầu tiên của một sản phẩm mới | Thử nghiệm rẻ để lấy phân phối; thiết kế để chịu được biên rộng; giữ khả năng đảo |
| **Unknown unknowns** | Không biết cả kết cục nào có thể xảy ra | Một tương tác giữa ba hệ thống mà không ai vẽ được sơ đồ | Không tính được. Chỉ có ba cách: giảm blast radius, tăng khả năng phát hiện, tăng tốc độ hồi phục |

Hệ quả quan trọng: **với unknown unknowns, mọi nỗ lực phân tích thêm đều không có tác dụng.** Tài
nguyên phải chuyển từ dự đoán sang khả năng chống chịu — monitoring, rollback, blast radius nhỏ,
canary. Đây là lý do một tổ chức có DORA metrics tốt (deploy nhỏ, thường, hồi phục nhanh) chịu được
bất định tốt hơn một tổ chức có nhiều tài liệu rủi ro hơn.

**Cơ chế ba: rủi ro không cộng tuyến tính vì các hệ thống có phụ thuộc.** Hai rủi ro độc lập xác suất
10% cho xác suất ít nhất một xảy ra là 19%. Nhưng trong hệ thống thật, chúng thường tương quan: tải
tăng làm cả database chậm và cache miss tăng và job nền dồn lại. Suy ra: **những kịch bản đáng lo nhất
là kịch bản nhiều thứ xảy ra cùng lúc**, và chúng gần như không bao giờ có trong risk register được
viết theo dòng độc lập. Pre-mortem tồn tại một phần để bắt loại này.

### Mental Model

**Risk Register như một hàng đợi công việc, không phải một tài liệu.** Thay đổi tư duy quyết định:
mỗi dòng trong register phải sinh ra hoặc một hành động có người và ngày, hoặc một quyết định chấp
nhận có người ký. Dòng nào không sinh ra cái nào trong hai thứ đó thì xoá — nó chỉ làm loãng.

**Pre-mortem (Gary Klein).** Thay vì hỏi "có rủi ro gì", đặt cả nhóm vào tương lai: "Sáu tháng nữa,
dự án này đã thất bại hoàn toàn. Viết ra vì sao." Cơ chế tâm lý phía sau, gọi là *prospective
hindsight*: khi được yêu cầu giải thích một kết cục đã cho, con người sinh ra nhiều nguyên nhân cụ
thể hơn đáng kể so với khi được hỏi dự đoán mở. Thêm một tác dụng quan trọng trong bối cảnh Việt Nam:
nó **hợp thức hoá việc nói điều tiêu cực**. Người ngại phản biện trực diện vẫn nói được, vì hình thức
của bài tập là mô tả một tương lai giả định, không phải chỉ trích kế hoạch của ai.

**Bốn phản ứng với rủi ro — mitigate / avoid / transfer / accept.**

| Phản ứng | Nghĩa | Ví dụ kỹ thuật Việt Nam | Khi nào chọn |
|---|---|---|---|
| **Mitigate** | Giảm xác suất hoặc giảm tác động | Dual-write + đối soát; canary 5%; circuit breaker | Rủi ro trung bình–cao, có biện pháp chi phí hợp lý |
| **Avoid** | Không đi đường đó nữa | Không migrate trong mùa campaign; không dùng managed service ngoài VPC khi chưa qua compliance | Khi thiệt hại vượt ngưỡng chấp nhận được của tổ chức |
| **Transfer** | Chuyển rủi ro sang bên khác có năng lực chịu tốt hơn | Dùng cổng thanh toán có chứng nhận PCI DSS thay vì tự lưu thẻ; mua managed database có SLA; bảo hiểm trách nhiệm | Khi bên kia chịu rẻ hơn bạn nhiều lần |
| **Accept** | Chấp nhận có ý thức, có người ký, có ngày xét lại | "Chấp nhận cửa sổ mất dữ liệu 5 phút của replica async cho service báo cáo — Linh ký, xét lại 01/10" | Khi chi phí giảm rủi ro vượt expected loss |

Điểm thường bị làm sai: **accept mà không ai ký thì không phải accept, đó là bỏ qua.** Khác biệt nằm ở
chỗ accept có tên người và ngày xét lại, nên khi rủi ro thành hiện thực, tổ chức biết đây là đánh đổi
đã chọn và học được điều đúng thay vì đi tìm người để trách.

### Practical Framework

**Risk register mẫu cho một dự án migration.** Bối cảnh minh hoạ: fintech Việt Nam, migrate bảng
`transactions` (khoảng 400 triệu bản ghi) từ MySQL đơn lẻ sang cluster mới, có 7 hệ thống hạ nguồn
đọc dữ liệu. Cửa sổ dự kiến 3 tuần. Số liệu là số minh hoạ.

| # | Rủi ro (mô tả cụ thể, có động từ) | Xác suất | Tác động | Tín hiệu sớm (và ai thấy nó) | Biện pháp | Loại | Chủ rủi ro | Xét lại |
|---|---|---|---|---|---|---|---|---|
| R1 | Hệ thống hạ nguồn vẫn đọc cluster cũ sau khi cắt, làm báo cáo đối soát thiếu dữ liệu | Cao (60%) | Nghiêm trọng — sai số liệu gửi ngân hàng, vấn đề audit | Job đối soát 6h sáng có chênh lệch > 0 giữa hai cluster; alert vào kênh #recon-alert | Kiểm kê 7 hệ thống hạ nguồn, xác nhận từng chủ sở hữu bằng văn bản; dựng job so sánh count+sum theo giờ chạy song song 2 tuần | Mitigate | Khoa (Senior BE) | Mỗi ngày trong cửa sổ cắt |
| R2 | Migration chạy quá cửa sổ đêm, phải dừng giữa đường | Cao (50%) | Trung bình — dữ liệu ở trạng thái nửa vời | Sau 2 giờ đầu, tốc độ copy < 60% mức cần thiết | Chia migration theo lô theo tháng, mỗi lô độc lập và có thể dừng an toàn; đo tốc độ trên bản sao trước | Mitigate | Hà (BE) | Trước mỗi đêm chạy |
| R3 | Sai lệch số dư do giao dịch ghi vào cluster cũ trong lúc chuyển | Trung bình (25%) | Thảm khốc — vấn đề pháp lý, không chỉ kỹ thuật | Chênh lệch tổng số dư giữa hai cluster ≠ 0 tại bất kỳ checkpoint 15 phút | Dual-write + read-your-write từ cluster mới; freeze ghi 90 giây tại điểm cắt; đối soát tự động mỗi 15 phút với ngưỡng no-go | Mitigate | Tuấn (Tech Lead) | Hàng ngày |
| R4 | Rollback không hoạt động khi cần (script chưa từng chạy thật) | Trung bình (30%) | Thảm khốc — mắc kẹt ở trạng thái lỗi | Diễn thử rollback trên bản sao production thất bại hoặc mất > 30 phút | Diễn thử rollback đầy đủ 2 lần trên bản sao production trước ngày cắt; ghi thời gian thực tế | Mitigate | Khoa | Trước ngày cắt |
| R5 | Cửa sổ migration trùng campaign khuyến mãi của business | Trung bình (35%) | Nghiêm trọng — tải cao + hệ thống đang chuyển | Lịch marketing quý có sự kiện trong cửa sổ | Đọc lịch campaign 8 tuần tới; chốt bằng văn bản với PO rằng không có campaign trong cửa sổ; nếu có thì hoãn | Avoid | Linh (EM) | Hàng tuần |
| R6 | Kiểm toán viên yêu cầu audit trail liên tục qua thời điểm chuyển, mà thiết kế mới không giữ được | Thấp (15%) | Nghiêm trọng — phát hiện muộn thì phải làm lại | Hỏi bộ phận compliance trước, có văn bản trả lời | Xác nhận yêu cầu audit trail với compliance trước khi thiết kế; giữ bảng cũ read-only 12 tháng | Avoid | Linh | Một lần, trước thiết kế |
| R7 | Người duy nhất hiểu logic đối soát (Khoa) nghỉ hoặc ốm trong cửa sổ | Thấp (10%) | Nghiêm trọng — dừng dự án | Khoa là người duy nhất commit vào module đối soát 6 tháng qua | Pair Hà với Khoa toàn bộ giai đoạn; runbook viết bởi Hà, không bởi Khoa (kiểm tra tính đọc được) | Mitigate | Linh | Hàng tuần |
| R8 | Cluster mới có hiệu năng kém hơn ở một dạng query chưa được test | Trung bình (30%) | Trung bình — chậm, không mất dữ liệu | P95 của 10 query nặng nhất trên cluster mới trong shadow traffic | Chạy shadow read 1 tuần với traffic thật; so P95 từng query | Mitigate | Hà | Hàng ngày trong tuần shadow |
| R9 | Chi phí hạ tầng cluster mới cao hơn dự toán trên 50% | Thấp (20%) | Thấp — tiền, không phải rủi ro hệ thống | Chi phí 2 tuần đầu vượt 1/26 dự toán năm | Đặt budget alert; chấp nhận nếu vượt dưới 50% | Accept (Tuấn ký) | Tuấn | Cuối tháng đầu |

Ba đặc điểm làm bảng này khác một mục "Risks" ba dòng: mỗi rủi ro có **một câu có động từ và đối
tượng cụ thể** (không phải "rủi ro dữ liệu"); mỗi rủi ro có **tín hiệu sớm kèm người/kênh sẽ thấy tín
hiệu đó**; và cột xét lại có tần suất khác nhau theo rủi ro, nên register trở thành một phần của nhịp
làm việc hàng ngày chứ không phải tài liệu đính kèm.

Về việc dùng thang xác suất: dùng ba mức nhưng **gắn số vào mỗi mức** (Thấp ≤ 20%, Trung bình 20–50%,
Cao > 50%). Lý do: từ ngữ trần bị mỗi người đọc theo một thang riêng, và cùng chữ "Trung bình" có thể
nghĩa 10% với người này và 50% với người kia. Gắn số không làm ước lượng chính xác hơn, nhưng nó làm
mọi người đang nói về cùng một thứ.

**Pre-mortem 45 phút — quy trình cụ thể:**

```
PRE-MORTEM 45 PHÚT
Chuẩn bị: kế hoạch dự án đã viết (1-3 trang), gửi trước 24h.
Người tham gia: 5-8, BẮT BUỘC có ít nhất 1 người ngoài team (SRE, QA, người
  vận hành hạ nguồn, CS). Người ngoài nhìn ra rủi ro liên hệ thống.
Vai: 1 người điều phối (KHÔNG phải người chủ dự án — người chủ dự án dễ phòng vệ).

00:00-00:03  Điều phối đọc nguyên văn:
  "Hôm nay là <ngày kết thúc dự án + 1 tháng>. Dự án đã thất bại. Không phải
   thất bại nhỏ — nó gây ra sự cố phải báo cáo lên cấp trên. Các bạn có 8 phút
   viết ra tất cả nguyên nhân. Viết riêng, không thảo luận."

00:03-00:11  VIẾT IM LẶNG, cá nhân. (Bước này không được bỏ: nó chống anchoring
  và cho người ngại nói một kênh phát biểu ngang bằng.)

00:11-00:25  Đọc vòng tròn, mỗi người 1 nguyên nhân/lượt, đi nhiều vòng.
  Điều phối ghi lên bảng, KHÔNG cho phản biện, KHÔNG cho giải thích "cái đó
  mình đã xử lý rồi". Chỉ thu thập. Người chủ dự án nói SAU CÙNG.

00:25-00:33  Gộp trùng, mỗi rủi ro chấm xác suất và tác động (dùng thang có số).
  Bỏ phiếu: mỗi người 3 phiếu cho rủi ro mình thấy nghiêm trọng nhất.

00:33-00:43  Với top 5 theo phiếu, mỗi rủi ro trả lời 3 câu:
  a) Tín hiệu sớm nhất là gì, ai/hệ thống nào thấy nó, trong bao lâu?
  b) Biện pháp: mitigate / avoid / transfer / accept?
  c) Ai là chủ rủi ro (một tên) và xét lại khi nào?

00:43-00:45  Điều phối đọc lại 5 dòng vừa chốt. Chuyển vào risk register trong
  vòng 24h, có link từ kế hoạch dự án.

TIÊU CHÍ BIẾT LÀ XONG: register có ≥5 dòng mới, mỗi dòng có tên người và ngày.
Nếu buổi pre-mortem không sinh ra thay đổi nào trong kế hoạch, hoặc bạn đã có
kế hoạch hoàn hảo (khả năng thấp), hoặc buổi đó đã thất bại.
```

Một chi tiết đáng làm: mời một người **đã từng làm migration thất bại** ở nơi khác. Giá trị của họ
không phải kinh nghiệm chung mà là các nguyên nhân cụ thể mà người chưa trải qua không nghĩ ra.

### Trade-off

**Đầu tư vào dự đoán vs đầu tư vào chống chịu.** Với risk (biết phân phối), dự đoán có lợi nhuận cao:
một phép tính đơn giản chỉ ra biện pháp đúng. Với uncertainty và unknown unknowns, mọi đồng chi cho
dự đoán thêm gần như mất trắng, và cùng số tiền đó chi cho khả năng phát hiện và hồi phục cho lợi
nhuận cao hơn nhiều. Cách phân bổ thực dụng: dành phần lớn nỗ lực phân tích cho 5 rủi ro nghiêm trọng
nhất, và dành phần còn lại cho ba năng lực chung — monitoring có ngưỡng, rollback đã diễn thử, và
triển khai theo lô nhỏ. Cái mất khi nghiêng quá về chống chịu: bạn sẽ gặp những sự cố mà một buổi
phân tích 2 giờ đã tránh được.

**Register đầy đủ vs register dùng được.** Một register 40 dòng đầy đủ về mặt tuân thủ nhưng không ai
đọc; một register 8 dòng bỏ sót vài rủi ro nhưng được xem lại hàng tuần. Cái thứ hai có giá trị vận
hành cao hơn nhiều. Nguyên tắc: **giới hạn cứng 10 dòng cho phần được theo dõi tích cực**, phần còn
lại đưa xuống mục "đã xem xét, chấp nhận, không theo dõi" — vẫn ghi để không mất bộ nhớ, nhưng không
làm loãng phần đang hoạt động.

**Nói thật về rủi ro vs quản lý cảm nhận của stakeholder.** Trình bày đầy đủ rủi ro cho khách hàng
hoặc founder có hai tác động ngược nhau: nó tăng tín nhiệm dài hạn (bạn là người nhìn thấy vấn đề
trước) và nó tăng lo lắng ngắn hạn (có thể dẫn tới việc dự án bị hoãn hoặc bị can thiệp vi mô).
Nghiêng về nói đầy đủ khi rủi ro thuộc loại stakeholder có thể hành động (đổi lịch, cấp thêm người,
điều chỉnh scope); nghiêng về nói tóm lược kèm biện pháp khi rủi ro thuần kỹ thuật và bạn đã có kế
hoạch. Nguyên tắc không thương lượng: rủi ro có thể ảnh hưởng tới tiền, dữ liệu khách hàng, hoặc nghĩa
vụ pháp lý phải được báo lên, đầy đủ, dù bất tiện. Không báo là chuyển rủi ro thành rủi ro cá nhân của
chính bạn.

### Real-world Scenarios

**Bối cảnh fintech — rủi ro compliance và audit.** Ở fintech, có một lớp rủi ro mà engineering thường
không nhìn thấy vì nó không biểu hiện dưới dạng lỗi hệ thống: rủi ro **không chứng minh được**. Hệ
thống chạy đúng nhưng không giữ được audit trail đủ để trả lời câu hỏi của kiểm toán, hoặc không tái
tạo được trạng thái số dư ở một thời điểm quá khứ. Dòng R6 trong bảng trên là dạng này. Đặc điểm của
nó: xác suất thấp, phát hiện rất muộn (thường ở kỳ kiểm toán, 6–12 tháng sau), và chi phí khắc phục
gần bằng làm lại. Cách xử lý duy nhất hiệu quả là **đưa compliance vào từ bước thiết kế**, cụ thể là
một câu hỏi bắt buộc trong template thiết kế: "nếu kiểm toán viên hỏi trạng thái của bản ghi X vào
ngày Y, chúng ta trả lời bằng dữ liệu nào?". Câu hỏi mất 5 phút và bắt được một lớp rủi ro không có
cách nào bắt sau.

**Bối cảnh e-commerce — peak campaign.** Rủi ro ở đây có hình dạng khác: xác suất cao, thời điểm biết
trước, và cửa sổ thiệt hại hẹp nhưng dày. Ba đặc thù cần vào register: (1) rủi ro không đến từ một hệ
thống mà từ **tương quan** — tải tăng làm cache miss tăng làm database chậm làm timeout làm retry làm
tải tăng thêm; register viết theo dòng độc lập bỏ sót toàn bộ vòng lặp này, nên phải có một dòng riêng
cho kịch bản dây chuyền; (2) rủi ro từ đối tác — cổng thanh toán, đơn vị vận chuyển, và bạn không kiểm
soát được họ, nên biện pháp là transfer (SLA có điều khoản) cộng mitigate (circuit breaker, hàng đợi
đệm, chế độ suy giảm ưu tuyển); (3) **code freeze** là một biện pháp avoid rẻ và hiệu quả, và điều
đáng thương lượng trước với business là số ngày freeze, thương lượng vào tháng trước campaign chứ
không phải tuần trước.

**Bối cảnh ODC — rủi ro phụ thuộc quyết định của khách hàng.** Đây là lớp rủi ro đặc thù và bị quản lý
kém nhất trong ODC Việt Nam. Dạng điển hình: team không thể tiếp tục vì chờ khách xác nhận một quyết
định kiến trúc, chờ khách cấp quyền truy cập môi trường, chờ khách phê duyệt thư viện. Mỗi ngày chờ là
một ngày utilization bị đốt, và trong hợp đồng fixed-price thì chi phí đó thuộc về bên bạn.

Cách xử lý cụ thể, khác với rủi ro nội bộ:

| Đặc điểm | Rủi ro nội bộ | Rủi ro phụ thuộc khách hàng |
|---|---|---|
| Biện pháp mitigate | Bạn tự làm | Bạn không tự làm được — chỉ có thể **làm cho nó hiện ra** |
| Tín hiệu sớm | Metric hệ thống | Số ngày một câu hỏi đang mở; đếm bằng bảng công khai |
| Công cụ chính | Kỹ thuật | Hợp đồng và giao tiếp: SLA phản hồi, danh sách dependency công khai, báo cáo tuần có cột "đang chờ khách" |
| Chủ rủi ro | Engineer | Account Manager / Delivery Lead, không phải Tech Lead |

Thực hành có hiệu lực nhất: một bảng **Open Decisions** gửi kèm báo cáo tuần cho khách, mỗi dòng có
câu hỏi, ngày hỏi, số ngày đang chờ, và **hậu quả cụ thể nếu chưa có câu trả lời trước ngày X**. Cột
cuối là cột làm việc: "nếu chưa có xác nhận về định dạng file trước 12/8, module import sẽ trễ 2 tuần
và ảnh hưởng mốc UAT". Nó chuyển rủi ro từ chỗ vô hình sang chỗ khách hàng phải quyết định có ý thức,
và nó tạo bản ghi bảo vệ bạn khi mốc bị trễ.

### Best Practices

- **Mỗi rủi ro viết bằng một câu có chủ thể, động từ và hậu quả.** So sánh: "rủi ro dữ liệu" vs "hệ
  thống hạ nguồn vẫn đọc cluster cũ sau khi cắt, làm báo cáo đối soát thiếu dữ liệu". Chỉ câu thứ hai
  suy ra được biện pháp.
- **Bắt buộc cột tín hiệu sớm, và tín hiệu phải có người hoặc hệ thống cụ thể sẽ thấy nó.** Lý do:
  thiệt hại tỉ lệ với thời gian phát hiện; đây là cột giảm N trong "N ngày không ai biết".
- **Gắn số vào các mức xác suất, và cấm mức giữa cho các rủi ro top 5.** Lý do: buộc phải cam kết một
  đánh giá. Với top 5, yêu cầu người viết chọn Cao hoặc Thấp và giải thích — điều này lộ ra chỗ thực
  sự không biết.
- **Một tên người ở cột chủ rủi ro, và người đó phải biết mình được ghi tên.** Lý do: rủi ro không có
  chủ thì không ai theo dõi tín hiệu sớm.
- **Xem lại register theo nhịp, gắn vào một nghi thức đã có.** Lý do: một tài liệu chỉ được xem lại
  khi ai đó nhớ ra thì sẽ không được xem lại. Gắn vào weekly của team hoặc vào định nghĩa "sẵn sàng
  chạy" của từng mốc, mất 10 phút.
- **Rủi ro accept phải có tên người ký và ngày xét lại.** Lý do: nó là khác biệt duy nhất giữa một
  đánh đổi có ý thức và một sự bỏ qua — và khác biệt đó quyết định tổ chức học được gì khi rủi ro
  thành hiện thực.
- **Sau mỗi incident, kiểm tra xem nguyên nhân có trong register hay không, và trả lời vì sao không.**
  Lý do: đây là learning loop của chính hoạt động risk assessment. Nếu nguyên nhân có trong register
  mà vẫn xảy ra, vấn đề là ở biện pháp; nếu không có, vấn đề là ở cách sinh ra danh sách rủi ro — hai
  vấn đề khác nhau cần hai cách sửa khác nhau. Xem `06-incident-va-metrics.md`.

### Anti-patterns

**Risk register viết một lần rồi không xem lại.** Biểu hiện: ngày cập nhật = ngày tạo; register nằm
trong một tab của tài liệu kế hoạch mà không ai mở. Cơ chế phá hoại kép: nó tiêu tốn thời gian thật
để tạo, và nó tạo ra cảm giác đã xử lý rủi ro — cảm giác này làm giảm sự cảnh giác, nên tổ chức tệ
hơn so với việc không có register nào. Dấu hiệu sớm: khi hỏi "rủi ro lớn nhất của dự án này bây giờ là
gì", không ai trả lời bằng một dòng trong register.

**Đưa mọi rủi ro về mức "Medium".** Biểu hiện: trên 70% dòng có xác suất và tác động ở mức giữa. Cơ
chế: đây là hành vi né tránh, không phải hành vi đánh giá. Nói "Cao" nghĩa là phải làm gì đó và có thể
phải báo lên; nói "Thấp" nghĩa là chịu trách nhiệm nếu nó xảy ra; nói "Medium" là trạng thái an toàn
về mặt xã hội và vô dụng về mặt thông tin. Hậu quả: register không phân biệt được gì nên không ưu tiên
được gì. Dấu hiệu sớm: mọi dòng có cùng màu trong heat map.

**Register là bản sao của danh sách công việc.** Biểu hiện: các dòng như "rủi ro: chưa viết test",
"rủi ro: chưa setup CI". Cơ chế: những thứ đó là việc chưa làm, không phải rủi ro. Trộn hai loại làm
register phình ra và làm mất khả năng nhận diện rủi ro thật, tức là những thứ có thể xảy ra ngoài dự
định. Dấu hiệu sớm: mỗi dòng register có một ticket tương ứng 1:1.

**Bỏ qua rủi ro đuôi dài vì xác suất thấp.** Biểu hiện: "backup chưa bao giờ cần restore, xác suất
thấp, để sau". Cơ chế: EV của các sự kiện đuôi dài có thể rất cao vì thiệt hại lớn, và một số thiệt
hại thuộc loại tồn tại (mất toàn bộ dữ liệu khách hàng, leak dữ liệu KYC) mà không có phép tính EV nào
áp dụng được — chúng phải được xử như ràng buộc. Dấu hiệu sớm: chưa từng có bài kiểm tra restore thành
công từ backup trong 12 tháng qua.

**Pre-mortem biến thành buổi bảo vệ kế hoạch.** Biểu hiện: mỗi rủi ro nêu ra đều nhận được câu "cái đó
mình đã xử lý rồi". Cơ chế: người chủ dự án điều phối buổi họp, nên buổi họp phục vụ việc xác nhận kế
hoạch thay vì tìm lỗ hổng. Cách chặn: người điều phối không phải người chủ dự án, và cấm phản biện
trong pha thu thập. Dấu hiệu sớm: buổi pre-mortem kết thúc mà không có dòng nào mới trong register.

**Risk assessment như thủ tục để được phê duyệt.** Biểu hiện: register được viết vì template dự án yêu
cầu, có 3 dòng chung chung, hoàn thành trong 5 phút cuối buổi họp. Cơ chế: khi mục đích là qua cửa,
sản phẩm sẽ tối ưu cho việc qua cửa. Dấu hiệu sớm: các dòng trong register giống nhau giữa các dự án
khác nhau.

### Khi nào KHÔNG nên áp dụng

- **Dự án nhỏ, khả đảo, blast radius trong một service.** Với một việc 1–2 tuần mà rollback là revert
  một commit, dựng risk register là chi phí thuần. Phiên bản đủ dùng: một dòng trong PR description
  ("rủi ro chính: X; nếu xảy ra thì revert, mất khoảng 1 giờ").
- **Trong incident đang diễn ra.** Risk assessment là hoạt động trước sự kiện. Trong incident, việc
  cần làm là phục hồi và ghi timeline; phân tích rủi ro thuộc về Postmortem. Cố làm risk assessment
  giữa incident vừa chậm vừa tạo tranh luận sai thời điểm.
- **Khi tổ chức chưa có năng lực hành động trên rủi ro đã nhận diện.** Nếu bạn dựng một register 10
  dòng nhưng team không có capacity để thực hiện biện pháp nào, register trở thành danh sách lo lắng
  và có tác dụng làm giảm morale. Trong trường hợp này, việc đúng là chọn đúng **hai** rủi ro nghiêm
  trọng nhất và dồn toàn bộ vào đó, đồng thời báo lên rằng các rủi ro còn lại đang ở trạng thái
  accept — có tên người ký. Điều này khác hẳn với việc liệt kê 10 rủi ro và không làm gì.
- **Khi bất định thuộc loại unknown unknowns và bạn đang cố phân tích nó.** Ba buổi họp để nghĩ ra
  những gì có thể sai với một hệ thống chưa ai từng vận hành sẽ cho ít giá trị hơn một tuần chạy
  canary 1% traffic. Dấu hiệu bạn đang ở vùng này: các rủi ro nêu ra ngày càng chung chung và không
  ai đưa được tín hiệu sớm cụ thể. Lúc đó chuyển ngân sách từ phân tích sang quan sát.
- **Khi việc trình bày rủi ro sẽ bị dùng để chặn một việc cần làm.** Có bối cảnh tổ chức mà một
  register đầy đủ được dùng bởi một bên khác để trì hoãn thay đổi ("nhiều rủi ro thế thì chưa làm").
  Đây không phải lý do để che rủi ro, nhưng nó là lý do để trình bày rủi ro **luôn kèm biện pháp và
  chi phí biện pháp**, và kèm rủi ro của việc không làm gì. Một register không có cột rủi ro của
  phương án status quo là một register nghiêng về phía không thay đổi.

---

## 6. Prioritization Frameworks ở cấp team và sản phẩm

### Problem Statement

Backlog của một team product startup 30 engineer, mở Jira ra: 214 item, trong đó 68 item gắn nhãn P0
và 51 item gắn nhãn P1. Sprint hiện tại có 9 item đang In Progress cho 6 engineer. Ba item đã ở trạng
thái In Progress hơn bốn tuần. Trong retro tuần trước, câu được nhắc nhiều nhất là "mình làm nhiều mà
chẳng thấy cái gì xong".

Đây không phải vấn đề về công cụ ưu tiên. Đây là hiện tượng của một hệ thống không có ràng buộc trên
lượng công việc đang chạy đồng thời, cộng với một hệ thống nhãn ưu tiên đã mất khả năng phân biệt.

Hiện tượng đếm được:

- **Số item gắn P0 chia cho capacity một sprint.** Nếu tỉ lệ trên 5, nhãn P0 không còn nghĩa. Ở ví dụ
  trên: 68 / khoảng 12 item mỗi sprint ≈ 5,7 sprint chỉ để làm hết P0.
- **Số item đang In Progress chia cho số engineer.** Nếu trên 1,5, mỗi người đang giữ nhiều hơn một
  việc và chi phí context switch đang được trả liên tục mà không hiện trên bảng nào.
- **Tuổi trung vị của item In Progress.** Nếu vượt 2× độ dài sprint, công việc đang bị chặn chứ không
  đang được làm, và điểm chặn là thứ cần tìm.
- **Số lần đổi ưu tiên giữa sprint trong một quý.** Đếm số item bị đưa vào sprint sau khi sprint đã
  bắt đầu. Nếu trên 25% dung lượng sprint, kế hoạch sprint đang là hư cấu.
- **Tỉ lệ capacity thực tế dành cho technical health trong 4 quý.** Nếu con số cam kết là 20% và con
  số thực tế là 4%, tổ chức có một chính sách trên giấy và một chính sách thật.

Hậu quả nếu không sửa: throughput giảm dù giờ làm tăng (do context switch và do work-in-progress cao);
nợ kỹ thuật tích luỹ đến điểm mỗi tính năng mất gấp 2–3 lần; và một hậu quả về người — engineer mất
cảm giác hoàn thành, đây là một trong hai lý do nghỉ việc phổ biến nhất ở tầng Senior mà không xuất
hiện trong exit interview dưới dạng đó.

### First Principles

**Cơ chế một: prioritization là bài toán phân bổ tài nguyên khan hiếm trên các dòng giá trị có cost of
delay khác nhau — và cost of delay là biến bị bỏ sót nhiều nhất.** Nếu mọi việc có cost of delay bằng
nhau, thứ tự không quan trọng và bạn chỉ cần làm cái nhanh nhất trước. Trong thực tế, cost of delay
chênh nhau hàng chục lần: một tính năng tuân thủ quy định có hiệu lực 01/01 có cost of delay bằng
vô cực sau mốc đó, còn một cải tiến UI nhỏ có cost of delay gần bằng 0. Suy ra quy tắc tối ưu, được
biết trong lý thuyết lập lịch là **WSJF — Weighted Shortest Job First**: sắp xếp theo tỉ lệ cost of
delay / thời gian thực hiện. Đây không phải một framework do ai phát minh mà là kết quả toán học của
bài toán tối thiểu hoá tổng chi phí trễ.

**Cơ chế hai: Theory of Constraints — tối ưu ngoài điểm nghẽn không tạo throughput.** Goldratt: trong
mọi hệ thống dòng chảy, có một ràng buộc quyết định throughput toàn hệ thống, và cải thiện ở bất kỳ
chỗ nào khác chỉ tạo ra hàng tồn kho trước ràng buộc. Map vào team: nếu ràng buộc là code review (mọi
PR chờ một người) thì tuyển thêm developer làm tăng số PR chờ, không tăng số feature ra production.
Nếu ràng buộc là QA thủ công thì tăng tốc dev làm tăng hàng đợi QA. Nếu ràng buộc là quyết định của
PO thì mọi thứ khác đều là tối ưu vô ích.

Hệ quả cho prioritization, và nó khá triệt để: **câu hỏi đầu tiên không phải "làm gì trước" mà là
"ràng buộc của hệ thống này ở đâu".** Với một team có ràng buộc ở review, việc ưu tiên đúng nhất trong
quý có thể là "đào tạo hai người nữa review được module thanh toán" — một việc không nằm trong bất kỳ
backlog sản phẩm nào.

**Cơ chế ba: Little's Law và chi phí của work-in-progress cao.** L = λ × W: số việc trong hệ thống
bằng tốc độ vào nhân thời gian trong hệ thống. Đảo lại: W = L / λ — **thời gian hoàn thành một việc tỉ
lệ thuận với số việc đang chạy song song.** Nếu team giữ 9 việc đồng thời thay vì 4, mỗi việc mất hơn
hai lần thời gian để xong, dù tổng công sức không đổi. Đây là lý do "làm nhiều mà chẳng thấy cái gì
xong" là mô tả chính xác về mặt vật lý, không phải cảm giác.

**Cơ chế bốn: context switch có chi phí bậc thang, không phải chi phí tuyến tính.** Chuyển giữa hai
việc trong cùng một module tốn vài phút; chuyển giữa hai module khác nhau tốn 20–40 phút để nạp lại
mô hình tinh thần; chuyển giữa hai loại công việc khác nhau (viết feature vs debug production) tốn cả
buổi vì nó đổi cả chế độ làm việc. Suy ra: chi phí của một lần đổi ưu tiên giữa sprint không phải "mất
15 phút" mà là hàm của khoảng cách giữa hai việc.

### Mental Model

**So sánh 5 framework — cái nào cho bài toán nào, cần dữ liệu gì, hỏng ở đâu.**

| Framework | Công thức / cơ chế | Phù hợp bài toán | Cần dữ liệu gì | Hỏng ở đâu |
|---|---|---|---|---|
| **RICE** | (Reach × Impact × Confidence) / Effort | So sánh nhiều tính năng sản phẩm cùng loại, có dữ liệu người dùng | Số user ảnh hưởng/tháng, ước lượng tác động, ước lượng effort | Impact là thang tự đặt (0,25/0,5/1/2/3) nên dễ gaming; **không mã hoá được deadline cứng**; Reach không dùng được cho việc hạ tầng |
| **WSJF** | Cost of Delay / Job Duration; CoD = giá trị kinh doanh + tính cấp thời + giảm rủi ro/tạo cơ hội | Danh mục hỗn hợp có cả feature, hạ tầng, compliance; có deadline khác nhau | Ước lượng CoD theo thang tương đối (Fibonacci), ước lượng thời gian | Ba thành phần của CoD dễ bị cộng lẫn; đòi hỏi PO và engineering cùng ước lượng, nếu chỉ một bên làm thì lệch |
| **Kano** | Phân loại tính năng: must-be / performance / delighter / indifferent / reverse | Quyết định *mức độ hoàn thiện* cho từng tính năng, không phải thứ tự | Khảo sát người dùng dạng cặp câu hỏi (chức năng/phi chức năng) | Cần khảo sát thật; kết quả **dịch chuyển theo thời gian** (delighter hôm nay là must-be sau 2 năm); không dùng được cho backlog kỹ thuật |
| **Opportunity Scoring** | Importance − Satisfaction, tìm chỗ quan trọng nhưng làm chưa tốt | Tìm chỗ đầu tư có lợi nhất trong sản phẩm đã có người dùng | Khảo sát mức quan trọng và mức hài lòng theo từng job | Vô dụng với sản phẩm mới chưa có user; đo cảm nhận nên nhiễu cao |
| **MoSCoW** | Must / Should / Could / Won't have | Chốt scope cho một mốc giao hàng cố định, đặc biệt trong ODC và hợp đồng | Chỉ cần đồng thuận giữa các bên | Sụp nếu không giới hạn tỉ lệ Must — thực tế Must thường phình lên 80%; **không cho thứ tự trong nhóm Must** |

Cách chọn thực dụng: **WSJF cho danh mục hỗn hợp ở cấp quý** (vì nó là framework duy nhất trong nhóm
xử lý được deadline cứng và giá trị của việc giảm rủi ro, nên nó là framework duy nhất cho phép việc
hạ tầng cạnh tranh công bằng với feature); **RICE cho so sánh trong nhóm feature** khi đã có dữ liệu
người dùng; **MoSCoW cho chốt scope một mốc**; **Kano và Opportunity Scoring là công cụ của PO** và
lead nên biết đủ để đọc kết quả, không nên tự chạy.

Điểm quan trọng hơn cả việc chọn framework: **mọi framework tính điểm đều là cách làm cho các giả định
hiện ra, không phải cách tìm đáp án.** Nếu hai framework cho hai thứ tự khác nhau, cái đáng làm là tìm
hiểu giả định nào khác nhau, không phải chọn framework có kết quả dễ chịu hơn.

### Practical Framework

**Phân bổ capacity theo bucket.** Đây là công cụ có tác dụng lớn nhất trong chủ đề này, vì nó chuyển
cuộc tranh luận từ "việc này có được làm không" (mỗi lần là một cuộc chiến) sang "việc này thuộc bucket
nào" (một cuộc thảo luận ngắn).

| Bucket | Tỉ lệ ví dụ | Gồm gì | Ai quyết nội dung bucket | Cơ chế bảo vệ |
|---|---|---|---|---|
| **Roadmap sản phẩm** | 60% | Tính năng mới, cải tiến theo kế hoạch quý | PO / Product Manager | Không cần bảo vệ, đây là bucket luôn được ưu tiên |
| **Technical health** | 20% | Nợ kỹ thuật, hạ tầng, tooling, test, cải thiện độ tin cậy | Tech Lead quyết nội dung, không cần PO phê duyệt từng item | Xem phần dưới — đây là bucket bị xâm phạm |
| **Keep-the-lights-on** | 15% | Bug production, hỗ trợ CS, yêu cầu dữ liệu, on-call | Ai đang trực, theo hàng đợi | Đo và báo cáo; nếu vượt 15% liên tục thì đó là tín hiệu về chất lượng, không phải về ưu tiên |
| **Khám phá** | 5% | Spike, thử công nghệ, prototype, learning có mục đích | Engineer đề xuất, Tech Lead duyệt | Bucket đầu tiên bị cắt khi áp lực; cần được bảo vệ có ý thức |

Về con số: 60/20/15/5 là **số minh hoạ cho một team product ở trạng thái tương đối ổn định**. Các cấu
hình khác hợp lý trong bối cảnh khác: một team đang cứu một hệ thống legacy có thể cần 40/40/20/0
trong hai quý; một team ODC theo hợp đồng có thể chỉ có 80/10/10/0 và phải thương lượng bucket
technical health vào giá; một startup trước product-market fit có thể hợp lý ở 80/5/10/5 vì xác suất
codebase bị bỏ là cao. Điều quan trọng không phải con số mà là **con số được nói ra, được đo, và
được báo cáo**.

**Sáu cách bảo vệ bucket technical health — theo thứ tự hiệu lực tăng dần:**

```
1. THOẢ THUẬN BẰNG LỜI (hiệu lực thấp nhất)
   "Mình dành 20% cho technical health nhé." → sụp trong sprint đầu có áp lực.

2. ĐO VÀ BÁO CÁO HÀNG SPRINT
   Một biểu đồ cột trong báo cáo sprint: cam kết vs thực tế theo bucket.
   Không tranh luận, chỉ hiển thị. Sau 3 sprint, con số tự nói.

3. GẮN VÀO CÙNG MỘT BẢNG VỚI FEATURE
   Item technical health có ticket, có estimate, nằm trong sprint board.
   Việc không có ticket là việc không tồn tại với tổ chức.

4. QUY ĐỔI SANG TÁC ĐỘNG (dùng chủ đề 4)
   "Module này đang lấy 17% capacity dưới dạng thuế ước lượng."
   Chuyển từ xin phép sang trình bày một khoản lỗ đang chảy.

5. GẮN VÀO MỘT MỐC BÊN NGOÀI
   Peak campaign, kỳ audit, cam kết SLO với khách hàng. Bucket technical
   health được bảo vệ tốt nhất khi nó là điều kiện cần cho một mốc mà
   business đã cam kết với bên thứ ba.

6. LÀM THÀNH ĐIỀU KIỆN CỦA MỘT CAM KẾT (hiệu lực cao nhất)
   "Em cam kết mốc 15/11 với điều kiện 20% capacity cho technical health
   được giữ. Nếu nó bị cắt trong 2 sprint liên tiếp, em sẽ báo lại rằng
   mốc 15/11 không còn khả thi và cần dời."
   Điều này chỉ dùng được khi bạn thực sự làm đúng như đã nói.
```

Cấp 6 là công cụ mạnh và có thể dùng sai. Nó chỉ hoạt động khi ba điều kiện đủ: bạn có dữ liệu chứng
minh mối liên hệ giữa technical health và khả năng giao hàng; bạn thực sự sẽ báo lại khi điều kiện bị
vi phạm (nếu bạn không làm, lần sau không ai tin); và bạn dùng nó không quá một hai lần mỗi năm. Dùng
thường xuyên, nó biến thành đe dọa và hỏng quan hệ với business.

**Quy trình ưu tiên cấp quý — 5 bước:**

```
BƯỚC 1 — TÌM RÀNG BUỘC (30 phút, làm TRƯỚC mọi việc khác)
  Vẽ dòng chảy: idea → spec → dev → review → QA → deploy.
  Đo thời gian chờ ở mỗi chặng trong 20 item gần nhất.
  Chặng có thời gian chờ lớn nhất là ràng buộc.
  → Nếu ràng buộc KHÔNG phải "thời gian dev", thì phần lớn việc ưu tiên
    backlog sản phẩm sẽ không tăng throughput. Sửa ràng buộc trước.

BƯỚC 2 — CHỐT TỈ LỆ BUCKET CHO QUÝ (30 phút, với PO và EM)
  Viết ra, ký, gửi cho stakeholder. Số này thay đổi được giữa quý nhưng
  phải là một quyết định tường minh, có người quyết.

BƯỚC 3 — ƯU TIÊN TRONG TỪNG BUCKET (không so item khác bucket với nhau)
  Bucket roadmap: RICE hoặc WSJF, do PO chủ trì, Tech Lead cho estimate.
  Bucket technical health: WSJF với CoD = (thuế capacity đang trả) +
    (rủi ro incident) + (chặn tính năng nào trong roadmap).
  Bucket KTLO: hàng đợi theo mức độ ảnh hưởng, không cần framework.

BƯỚC 4 — ĐẶT GIỚI HẠN WIP
  Số item In Progress ≤ số engineer × 1,2. Viết vào cấu hình board nếu
  công cụ hỗ trợ. Đây là biện pháp có tỉ lệ hiệu quả/chi phí cao nhất
  trong toàn bộ chủ đề, và nó không cần ai phê duyệt.

BƯỚC 5 — ĐỊNH NGHĨA TRƯỚC QUY TẮC ĐỔI ƯU TIÊN GIỮA SPRINT
  Mẫu: "Item mới chỉ vào sprint đang chạy nếu (a) production đang lỗi,
  hoặc (b) có mốc pháp lý/hợp đồng, hoặc (c) PO đồng ý đẩy một item có
  cùng size ra khỏi sprint. Trường hợp (c) phải nêu tên item bị đẩy ra."
  Quy tắc (c) là quy tắc quan trọng nhất: nó làm chi phí của việc đổi
  ưu tiên hiện ra tại thời điểm đổi, cho chính người quyết đổi.
```

### Trade-off

**Linh hoạt đổi ưu tiên vs ổn định ưu tiên.** Đây là trade-off trung tâm và thường bị trình bày lệch,
như thể linh hoạt luôn tốt.

| | Nghiêng về linh hoạt | Nghiêng về ổn định |
|---|---|---|
| Điều kiện thị trường | Bất định cao, đang tìm product-market fit, đối thủ động nhanh | Sản phẩm đã ổn định, khách hàng doanh nghiệp, có cam kết hợp đồng |
| Loại công việc | Item nhỏ, độc lập, ít phụ thuộc | Item lớn, nhiều phụ thuộc, cần nạp mô hình tinh thần sâu |
| Được gì | Đáp ứng cơ hội và sự cố nhanh; không đổ tiền vào việc đã hết giá trị | Throughput cao hơn do ít context switch; ước lượng đáng tin hơn; team ít kiệt sức hơn |
| Mất gì | Throughput giảm; ước lượng mất ý nghĩa; niềm tin của team vào kế hoạch xói mòn | Có lúc làm xong một thứ đã hết cần thiết |
| Ai chịu phần mất | Engineer — họ trả bằng công việc dở dang và cảm giác vô nghĩa | Business — họ trả bằng độ trễ phản ứng |

**Chi phí thật của một lần đổi ưu tiên giữa sprint** — cần nói bằng số, vì nếu không, người quyết đổi
sẽ mặc định là gần 0:

| Thành phần chi phí | Ước lượng (số minh hoạ) | Cơ chế |
|---|---|---|
| Nạp lại mô hình tinh thần của người bị chuyển việc | 0,5–1 ngày | Chuyển module khác nhau; càng phức tạp càng lâu |
| Công việc dở dang bị treo | 30–70% công đã bỏ ra vào item cũ, nếu treo trên 2 tuần thì tiến tới 100% (phải đọc lại từ đầu, code có thể conflict) | Hàng tồn kho mất giá |
| Chi phí phối hợp | 1–3 giờ của 2–4 người: họp lại, cập nhật board, thông báo bên phụ thuộc | Coordination overhead |
| Chi phí lên ước lượng tương lai | Không định lượng được trực tiếp; biểu hiện là team bắt đầu đệm ước lượng 1,5–2× | Team học rằng cam kết sẽ bị phá, nên tự bảo hiểm |
| **Tổng cho một lần đổi ảnh hưởng 2 người** | **khoảng 2–4 người-ngày** | |

Với một team 6 người và sprint 2 tuần (60 người-ngày), ba lần đổi ưu tiên mỗi sprint tiêu khoảng
10–20% capacity vào chi phí chuyển đổi. Đây là con số đáng đưa ra bàn với PO, không phải để cấm việc
đổi ưu tiên — mà để việc đổi được quyết định với thông tin đúng.

**Framework tính điểm vs phán đoán của PO.** Framework cho tính minh bạch và khả năng phản biện, nhưng
nó cũng tạo ra ảo giác khách quan cho những con số vốn là phán đoán (Impact 2 vs 3 là gì?). Nghiêng về
framework khi có nhiều bên tranh chấp ưu tiên hoặc khi cần giải thích quyết định lên trên; nghiêng về
phán đoán khi PO có tín nhiệm cao và bối cảnh thay đổi nhanh hơn tốc độ cập nhật bảng điểm. Cái mất
khi nghiêng về phán đoán: khi PO đổi, không có gì được truyền lại.

### Real-world Scenarios

**Tình huống: bucket technical health bị cắt lần thứ ba, và hai cách phản ứng.**

Bối cảnh minh hoạ: startup e-commerce, team 6 engineer, sprint 2 tuần. Đã thoả thuận 20% capacity cho
technical health. Sprint này là sprint thứ ba liên tiếp bucket đó bằng 0, vì mỗi lần đều có một việc
"gấp hơn".

*Phiên bản nói sai — trong sprint planning:*

> **Tuấn (Tech Lead):** Chị ơi, đây là sprint thứ ba mình cắt technical health rồi. Mình đã thoả thuận
> 20% mà lần nào cũng cắt. Như vậy thì thoả thuận để làm gì?
>
> **Trang (PO):** Anh biết, nhưng lần này thật sự gấp, đối tác vận chuyển đổi API và mình phải theo,
> không thì đơn không đẩy đi được.
>
> **Tuấn:** Lần nào cũng thật sự gấp. Team em đang rất mệt vì suốt ngày chữa cháy trong code mà không
> được dọn.
>
> **Trang:** Anh hiểu mà. Sprint sau mình ưu tiên nhé, anh hứa.
>
> **Tuấn:** Chị hứa hai lần rồi.
>
> **Trang:** ...

Vấn đề của phiên bản này: nó đúng về nội dung và thất bại về cơ chế. Nó dựa vào lời hứa (không có hiệu
lực), nó đưa cảm xúc của team làm luận cứ chính (Trang không có cách đánh giá và không có công cụ hành
động trên nó), và nó đặt Trang vào vị thế phải nhận lỗi — nên phản ứng tự nhiên là phòng vệ và lời hứa
rẻ tiền. Sau cuộc này, không có gì trong hệ thống thay đổi, nên sprint thứ tư sẽ giống hệt.

*Phiên bản nói đúng:*

> **Tuấn:** Chị, em muốn nói về API vận chuyển và về một con số. API vận chuyển em đồng ý là phải làm
> sprint này, không tranh luận — nó chặn đơn hàng, cost of delay cao nhất trong backlog.
>
> **Trang:** Tốt, cảm ơn anh.
>
> **Tuấn:** Nhưng em cần chị nhìn cái này. Đây là biểu đồ ba sprint: cam kết 20% technical health,
> thực tế 0%, 0%, 0%. Và đây là con số hệ quả: tỉ lệ ước lượng chia thực tế của module đơn hàng đã đi
> từ 1,4 lên 2,1 trong ba sprint đó. Nghĩa là mọi việc chị đưa vào module đó bây giờ mất gấp hai lần
> so với đầu quý. Riêng sprint này, phần chênh do đó là khoảng 9 người-ngày trong tổng 60.
>
> **Trang:** Nghĩa là mình đang tự làm chậm mình?
>
> **Tuấn:** Đúng, và nó tăng dần. Em không đề nghị dừng gì cả. Em đề nghị ba việc. Một: sprint này em
> lấy 2 ngày của Khoa để bọc test quanh phần đơn hàng mà API vận chuyển sẽ chạm — cái này giảm rủi ro
> ngay cho chính việc chị cần. Hai: từ sprint sau, quy tắc là item mới vào sprint đang chạy thì chị
> nêu tên item bị đẩy ra, kể cả khi item bị đẩy là technical health — em chỉ cần nó hiện ra, không cần
> nó không bao giờ xảy ra. Ba: em đưa biểu đồ này vào báo cáo sprint hàng tuần, để mình không phải nói
> lại chuyện này bằng cảm giác nữa.
>
> **Trang:** Cái thứ hai nghĩa là em muốn chị phải chọn công khai.
>
> **Tuấn:** Đúng. Không phải để làm khó chị. Là vì hiện tại chi phí của việc cắt nằm hết ở phía team
> và chị không nhìn thấy nó lúc quyết. Em muốn chị có đủ thông tin để quyết, kể cả khi chị vẫn quyết
> cắt.

Bốn khác biệt về cơ chế: (1) Tuấn nhượng bộ trước ở điểm không đáng tranh (API vận chuyển) nên phần
còn lại không bị đọc là kháng cự; (2) luận cứ là một chỉ số có thể kiểm tra, không phải cảm xúc; (3)
đề nghị gắn technical health vào chính việc PO đang cần, nên nó không cạnh tranh mà bổ trợ; (4) yêu
cầu cốt lõi không phải "đừng cắt" mà là **làm cho chi phí hiện ra tại thời điểm quyết định** — một yêu
cầu về quy trình, dễ đồng ý hơn nhiều và có hiệu lực lâu hơn một lời hứa.

**Cùng tình huống, ba tầng đọc:**

- **IC (Khoa, Senior BE):** thấy một codebase ngày càng khó làm và một lời hứa bị phá ba lần. Kết luận
  tự nhiên: "ở đây không ai quan tâm chất lượng". Đây là con đường ngắn nhất tới việc rút khỏi việc
  cải thiện và cuối cùng là rời công ty.
- **Tech Lead (Tuấn):** thấy một cơ chế phân bổ đang hỏng và biết rằng cách sửa là làm cho chi phí ẩn
  trở nên quan sát được. Việc của Tuấn không phải chiến thắng Trang mà là thay đổi thông tin đầu vào
  của quyết định.
- **Manager (Linh, EM):** thấy ba thứ khác. Một, việc bucket bị cắt ba lần liên tiếp có thể là dấu
  hiệu tỉ lệ 20% đặt sai từ đầu (nếu KTLO thực tế là 30% thì công thức không khớp thực tế). Hai, việc
  cả ba lần đều có "việc gấp" là dấu hiệu quy trình lập kế hoạch quý không nhìn ra được các phụ thuộc
  bên ngoài như API đối tác — một vấn đề thuộc `07-project-delivery.md`. Ba, Khoa đang ở vùng nguy cơ
  nghỉ việc, và đó là rủi ro cần vào register.

### Best Practices

- **Tìm ràng buộc trước khi ưu tiên.** Lý do: nếu ràng buộc không ở chỗ bạn đang tối ưu, việc sắp xếp
  backlog không tạo throughput. Đây là bước bị bỏ trong hầu hết các buổi planning.
- **Đặt giới hạn WIP và bảo vệ nó cứng hơn bảo vệ thứ tự ưu tiên.** Lý do: theo Little's Law, giảm WIP
  cải thiện thời gian hoàn thành ngay lập tức và không cần ai phê duyệt. Nó là biện pháp có tác dụng
  nhanh nhất trong toàn bộ chương này.
- **Ưu tiên trong bucket, không so item khác bucket với nhau.** Lý do: so một tính năng với một
  refactor bằng cùng thang điểm luôn thua cho tính năng, vì lợi ích của tính năng dễ quan sát hơn. Chia
  bucket là cách chuyển cuộc chiến đó lên cấp quý và giải nó một lần thay vì mỗi sprint.
- **Đo bucket thực tế và báo cáo đều đặn, kể cả khi con số xấu.** Lý do: một cam kết không được đo
  không phải cam kết. Và biểu đồ ba sprint có sức thuyết phục mà không lập luận nào có được.
- **Quy tắc "một vào thì một ra, nêu tên cái ra".** Lý do: nó không chặn việc đổi ưu tiên (chặn là
  không thực tế), nó chỉ làm chi phí hiện ra cho đúng người, tại đúng thời điểm.
- **Dùng WSJF khi cần cho việc hạ tầng cạnh tranh công bằng với feature.** Lý do: thành phần "giảm rủi
  ro và tạo cơ hội" trong CoD là chỗ duy nhất trong các framework phổ biến mà giá trị của technical
  health được mã hoá vào công thức.
- **Cho mỗi P0 một cái giá: nếu mọi thứ là P0 thì yêu cầu xếp hạng tuyến tính 1..N.** Lý do: nhãn cho
  phép nhiều item cùng mức nên nó luôn phình; danh sách xếp hạng tuyến tính buộc phải chọn. Câu hỏi
  vận hành: "trong 68 item P0 này, item nào là số 1, số 2, số 3?".

### Anti-patterns

**Mọi thứ là P0.** Biểu hiện: 68/214 item gắn P0. Cơ chế phá hoại: nhãn ưu tiên là một kênh truyền
thông tin có băng thông hữu hạn; khi mọi item đều mang mức cao nhất, kênh truyền tải 0 bit và việc
chọn thứ tự thực tế rơi xuống cho engineer, người có ít thông tin nhất về giá trị kinh doanh. Dấu hiệu
sớm: có người trong team hỏi "mấy cái P0 này thì làm cái nào trước".

**HiPPO — Highest Paid Person's Opinion.** Biểu hiện: thứ tự backlog đổi sau mỗi lần founder dùng thử
sản phẩm. Cơ chế: nó không sai vì founder kém phán đoán — founder thường có thông tin tốt nhất về
chiến lược. Nó sai vì (a) nó bỏ qua cost of delay của những thứ đang chạy, (b) nó không để lại lý do
nên không ai học được gì, và (c) nó dạy tổ chức rằng con đường ngắn nhất để việc của mình được làm là
đi qua founder, làm mọi kênh ưu tiên khác chết dần. Dấu hiệu sớm: PO bắt đầu nói "cái này anh CEO muốn"
thay vì nói lý do.

**Ưu tiên theo khách hàng gào to nhất.** Biểu hiện: tính năng được xây cho một khách hàng doanh nghiệp
vừa gửi email phàn nàn, trong khi 200 khách hàng nhỏ đang gặp một vấn đề lớn hơn nhưng không ai viết
email. Cơ chế: âm lượng phản hồi tương quan với quyền lực và tính cách của khách, không tương quan với
giá trị. Ngoại lệ hợp lý: nếu khách đó chiếm 30% doanh thu, họ *đáng* được ưu tiên — nhưng lúc đó lý
do phải được nói ra là doanh thu, không phải là họ vừa phàn nàn. Dấu hiệu sớm: backlog có các item mà
Reach = 1.

**Đổi ưu tiên giữa sprint mà không lấy gì ra.** Biểu hiện: sprint bắt đầu với 12 item, kết thúc với 15
item trong scope và 6 item xong. Cơ chế: capacity là hằng số trong sprint; thêm vào mà không lấy ra là
một phát biểu số học sai, và hệ thống giải nó bằng cách để mọi thứ dở dang. Dấu hiệu sớm: tuổi trung
vị của item In Progress tăng dần qua các sprint.

**Bucket technical health tồn tại trên slide, bằng 0 trong thực tế.** Biểu hiện: đã mô tả ở trên. Cơ
chế: một cam kết không được đo và không có cơ chế cưỡng chế sẽ thua mọi áp lực ngắn hạn, mỗi lần đều
với một lý do hợp lý. Dấu hiệu sớm: không ai đo được con số thực tế của bucket đó trong quý vừa rồi.

**Dùng framework tính điểm để né trách nhiệm ưu tiên.** Biểu hiện: "RICE cho ra thứ tự này". Cơ chế:
các đầu vào của RICE phần lớn là phán đoán, nên điểm số chỉ chuyển phán đoán từ chỗ hiện sang chỗ ẩn.
Dấu hiệu sớm: không ai chất vấn được vì sao Impact của một item là 2 mà không phải 1.

### Khi nào KHÔNG nên áp dụng

- **Team dưới 5 người với một dòng công việc duy nhất.** Với 4 engineer làm một sản phẩm, một danh
  sách xếp hạng tuyến tính do PO và Tech Lead cùng duy trì là đủ, và mọi framework tính điểm là chi phí
  thuần. Bucket cũng không cần — 4 người có thể nói với nhau về việc dành hai ngày dọn dẹp.
- **Trong giai đoạn khủng hoảng.** Khi hệ thống đang hỏng nặng hoặc công ty đang cạn runway, ưu tiên
  không đi qua framework mà đi qua một người quyết và một danh sách rất ngắn. Cố duy trì tỉ lệ bucket
  trong giai đoạn này là ưu tiên hình thức trên thực chất. Điều kiện quan trọng: **nói rõ đây là chế
  độ tạm thời và nêu điều kiện kết thúc**, vì chế độ khủng hoảng không có điều kiện kết thúc sẽ trở
  thành văn hoá vĩnh viễn — đó là một trong những cách phổ biến nhất mà một tổ chức kỹ thuật tự phá
  huỷ.
- **Khi bạn không kiểm soát scope.** Trong ODC với hợp đồng đã chốt scope và thứ tự, việc chạy WSJF
  nội bộ không thay đổi được gì. Cái dùng được ở đó là MoSCoW trong phạm vi mỗi mốc, và quan trọng hơn
  là bảng dependency công khai với khách hàng (xem chủ đề 5). Nơi để thương lượng bucket technical
  health trong ODC là lúc báo giá, không phải lúc đang chạy sprint.
- **Khi bài toán thật là năng lực chứ không phải thứ tự.** Nếu team không xong việc vì thiếu kỹ năng ở
  một vùng (không ai biết tối ưu query, không ai hiểu hệ thống thanh toán), thì sắp lại thứ tự backlog
  không giúp gì. Cái cần là mentoring, tuyển, hoặc thu hẹp phạm vi hệ thống mà team phải sở hữu — xem
  `03-team-leadership.md` và `09-to-chuc-va-scaling.md`.
- **Khi việc ưu tiên đang được dùng để né một quyết định về scope tổng thể.** Có trường hợp backlog
  214 item không phải vấn đề ưu tiên mà là vấn đề tổ chức đã cam kết quá nhiều thứ. Sắp xếp lại 214
  item là một cách rất tốn kém để tránh cuộc hội thoại thật: cái gì trong danh sách này chúng ta sẽ
  không làm, và ai nói điều đó với người đã yêu cầu nó. Test đơn giản: nếu bạn xoá 60% backlog thì có
  ai phát hiện trong 3 tháng? Nếu câu trả lời là không, việc cần làm là xoá, không phải xếp.

---

## 7. Ra quyết định trong nhóm: consensus, consent và người quyết cuối

### Problem Statement

Buổi họp kiến trúc thứ ba về cùng một chủ đề: tách module đặt hàng thành service riêng hay giữ trong
monolith và tách theo module boundary. Khoa và Quân, hai Senior BE, đã tranh luận 90 phút và không hội
tụ. Cả hai đều có lập luận đúng. Cả hai đều đã nhượng bộ ở các điểm nhỏ và giữ nguyên ở điểm chính.
Đến phút 90, Tuấn (Tech Lead) nói "thôi để mình nghĩ thêm, tuần sau bàn tiếp". Tuần sau, buổi thứ tư.

Bảy tuần sau, việc vẫn chưa bắt đầu. Trong bảy tuần đó, ba tính năng mới đã được thêm vào module đặt
hàng theo cách cũ, làm cả hai phương án đều đắt hơn khoảng 20%.

Hiện tượng đếm được:

- **Số buổi họp cho một quyết định, và tổng người-giờ.** Bốn buổi × 90 phút × 5 người = 30 người-giờ,
  chưa tính thời gian chuẩn bị và thời gian nghĩ ngoài họp.
- **Decision latency và chi phí trễ.** Bảy tuần trễ, trong đó chi phí của cả hai phương án tăng 20%.
- **Số người trong phòng có accountability thật.** Ở buổi họp trên: 1 (Tuấn). Bốn người còn lại có
  quyền input. Nhưng cấu trúc buổi họp đối xử với cả 5 như nhau, nên nó tìm consensus giữa 5 người —
  một mục tiêu tốn kém và không cần thiết.
- **Số người im lặng trong họp rồi phản đối lúc triển khai.** Ở nhiều team Việt Nam đây là con số lớn
  nhất và ẩn nhất. Dấu hiệu: quyết định được "đồng ý" trong họp, rồi hai tuần sau xuất hiện dưới dạng
  PR làm theo cách khác, hoặc dưới dạng một cuộc trò chuyện ở hành lang.

Hậu quả nếu không sửa: quyết định kỹ thuật rơi vào một trong hai chế độ hỏng. Chế độ một là bế tắc
vĩnh viễn — không ai chốt, hệ thống trôi theo quán tính, và phương án được chọn thực chất là "cái gì
dễ nhất cho người viết code tiếp theo". Chế độ hai là chốt bằng quyền lực — người có title cao nhất
quyết, thảo luận thành nghi lễ, và những người giỏi nhất về mặt kỹ thuật dần thôi tham gia.

### First Principles

**Cơ chế một: chi phí ra quyết định nhóm tăng nhanh hơn tuyến tính theo số người.** Số kênh giao tiếp
trong nhóm n người là n(n−1)/2. Với 3 người: 3 kênh. Với 6 người: 15 kênh. Với 10 người: 45 kênh. Mỗi
kênh là một chỗ cần đồng bộ hiểu biết. Đồng thời, xác suất tất cả n người đồng ý giảm theo n, nên thời
gian để đạt đồng thuận tăng nhanh. Suy ra: **số người trong một quyết định phải được chọn có ý thức,
và mặc định nên nhỏ hơn bạn nghĩ.** Quy tắc thực dụng: 3–5 người cho một quyết định kỹ thuật mức 3;
thêm người chỉ khi họ mang vào một loại thông tin không ai khác có.

**Cơ chế hai: consensus tạo ra quyết định trung bình, và trung bình trong không gian thiết kế thường
là điểm tệ.** Khi phải làm tất cả đồng ý, các phương án gây tranh cãi bị loại trước, kể cả những phương
án có kỳ vọng cao nhưng phương sai cao. Cái còn lại là phương án ít ai phản đối, thường là phương án
nhạt. Tệ hơn, trong thiết kế hệ thống, việc thoả hiệp giữa hai kiến trúc mạch lạc thường cho một kiến
trúc không mạch lạc — nửa microservice nửa monolith, có cả hai loại chi phí và không có lợi ích nào
trọn vẹn. **Đây là điểm khác biệt then chốt so với các quyết định có thể nội suy được**: có những trục
mà điểm giữa tệ hơn cả hai đầu.

**Cơ chế ba: sự đồng ý và sự cam kết thực thi là hai thứ khác nhau, và chỉ cái thứ hai là bắt buộc.**
Đây là lý do tồn tại của disagree-and-commit (Amazon nêu nguyên tắc này công khai trong bộ Leadership
Principles của họ, dưới tên "Have Backbone; Disagree and Commit"). Cơ chế: nếu tổ chức yêu cầu mọi
người phải *đồng ý* trước khi thực thi, thì mọi người bất đồng có quyền veto ngầm bằng cách thực thi
kém. Nếu tổ chức tách hai thứ ra — bạn được quyền không đồng ý, được ghi lại lý do không đồng ý, nhưng
bạn cam kết thực thi hết mình — thì tổ chức có được cả tốc độ lẫn sự trung thực. Điều kiện để nó hoạt
động: (a) người bất đồng phải thực sự được nghe trước khi chốt, (b) lý do bất đồng được ghi lại, (c)
có điều kiện xét lại, để người bất đồng biết mình có cơ hội đúng và được chứng minh sau. Thiếu ba điều
kiện này, disagree-and-commit biến thành "im đi và làm".

**Cơ chế bốn: quyền input và quyền veto phải tách rời, và nguyên tắc tách là accountability.** Nếu một
người không phải chịu hậu quả của quyết định thì họ không nên có quyền chặn nó. Lý do không phải về
phẩm chất của người đó mà về sự lệch giữa quyền và hậu quả: người có quyền chặn mà không gánh hậu quả
sẽ tối ưu cho việc tránh rủi ro cho bản thân, không cho outcome của hệ thống. Đây là một dạng
Principal-Agent Problem. Phát biểu vận hành: **ai không có accountability thì có quyền input, không có
quyền veto.**

### Mental Model

**DACI và RAPID — hai cách gán vai trong một quyết định.**

| Vai (DACI) | Nghĩa | Số người nên có | Sai lầm phổ biến |
|---|---|---|---|
| **Driver** | Người chạy quy trình: thu thập, tổng hợp, giữ lịch | 1 | Gộp Driver với Approver, làm người chạy quy trình cũng là người quyết → mất tính trung lập |
| **Approver** | Người quyết cuối và chịu accountability | **Đúng 1** | Để 2–3 người → không ai quyết |
| **Contributor** | Người có thông tin, có quyền input | 2–6 | Cho họ quyền veto ngầm bằng cách chờ "mọi người ok" |
| **Informed** | Người bị ảnh hưởng, cần biết kết quả | Không giới hạn | Không thông báo → quyết định đúng vẫn thất bại ở tầng thực thi |

RAPID (Bain) là biến thể tách chi tiết hơn: **R**ecommend, **A**gree (bên có quyền phủ quyết theo quy
định, thường là legal/compliance/security), **P**erform, **I**nput, **D**ecide. Điểm hữu ích riêng của
RAPID là nó tách "Agree" ra khỏi "Input" — có những bên thực sự có quyền chặn theo quy định (compliance
trong fintech, security với dữ liệu cá nhân) và cần được đối xử khác với người chỉ có ý kiến. Với hầu
hết quyết định kỹ thuật ở team Việt Nam, DACI đủ; dùng RAPID khi có bên có quyền chặn hợp pháp.

**Consensus vs Consent — phân biệt quan trọng và ít được biết.** Consensus: mọi người đồng ý đây là
phương án tốt nhất. Consent (vay từ Sociocracy): **không ai có phản đối có căn cứ rằng phương án này
gây hại cho mục tiêu chung.** Consent là ngưỡng thấp hơn nhiều và đạt được nhanh hơn nhiều, và nó hỏi
một câu hỏi khác về chất: không phải "anh có thích không" mà "anh có thấy lý do nào khiến việc này gây
hại không". Câu hỏi thứ hai loại bỏ được phần lớn phản đối dựa trên sở thích, và giữ lại phần phản đối
dựa trên rủi ro — đúng phần cần giữ.

Câu hỏi chốt phòng theo kiểu consent, nên dùng thay cho "mọi người ok chứ": **"Có ai thấy phương án
này gây rủi ro nghiêm trọng mà chúng ta chưa nói tới không? Nếu có, nói bây giờ."** Khác biệt giữa hai
câu này lớn hơn vẻ ngoài: câu đầu mời sự đồng ý (và im lặng dễ được đọc là đồng ý), câu sau mời một
loại thông tin cụ thể (và im lặng nghĩa là không có thông tin đó).

**Thang mức tham gia — công cụ để nói rõ luật chơi trước khi bắt đầu.**

| Mức | Tên | Lead nói gì ở đầu buổi | Khi nào dùng |
|---|---|---|---|
| 1 | Thông báo | "Việc này đã được quyết, tôi giải thích lý do và trả lời câu hỏi" | Quyết định đã bị ràng buộc bên ngoài (hợp đồng, compliance) |
| 2 | Hỏi ý kiến rồi tôi quyết | "Tôi cần input của các bạn, tôi sẽ quyết trước thứ Năm" | Mặc định cho quyết định mức 3 |
| 3 | Nhóm quyết theo consent, tôi có quyền chặn | "Các bạn chốt, tôi chỉ can thiệp nếu thấy rủi ro nghiêm trọng" | Quyết định trong phạm vi team, team có năng lực |
| 4 | Nhóm quyết, tôi thực thi theo | "Tôi không tham gia quyết, các bạn chốt và tôi hỗ trợ" | Quyết định về cách làm việc nội bộ của team |

Nói mức ra ở đầu buổi mất 15 giây và loại bỏ nguồn bất mãn phổ biến nhất trong họp kỹ thuật: người
tham gia tưởng mình đang ở mức 3 trong khi lead đang ở mức 2, nên khi lead quyết khác, họ cảm thấy bị
lừa. Cùng một quyết định, cùng một kết quả, cảm nhận hoàn toàn khác — và cảm nhận đó quyết định chất
lượng thực thi.

### Practical Framework

**Quy trình 5 bước cho một quyết định kiến trúc gây tranh cãi:**

```
BƯỚC 1 — THU TIÊU CHÍ TRƯỚC, KHÔNG BÀN PHƯƠNG ÁN (async, 2 ngày)
  Driver gửi câu hỏi: "Với bài toán X, theo bạn 5 tiêu chí quan trọng nhất
  để đánh giá một giải pháp là gì? Trả lời riêng, đừng nêu giải pháp."
  Thu về, gộp, công bố danh sách tiêu chí + trọng số đề xuất.
  → Bước này thường tiết lộ rằng hai bên tranh luận vì tối ưu hai tiêu chí
    khác nhau, và khi thấy điều đó, 30-50% tranh luận tự tan.

BƯỚC 2 — RFC (async, mỗi phương án một tác giả, 3-5 ngày)
  Mỗi phương án được viết bởi người BẢO VỆ nó, theo cùng template, cùng
  độ dài trần (2 trang). Mỗi RFC phải có mục "phương án này tệ nhất ở đâu"
  do chính tác giả viết. Mục này là mục quan trọng nhất.

BƯỚC 3 — THỜI HẠN PHẢN BIỆN CÔNG KHAI (async, 3 ngày, có ngày giờ cụ thể)
  Comment trên tài liệu. Quy tắc: phản biện phải trỏ vào một tiêu chí đã
  chốt ở bước 1. "Tôi không thích cách này" không phải phản biện.
  Async là điểm then chốt trong bối cảnh Việt Nam: người ngại phản biện
  trực diện trong phòng thường viết rất tốt bằng văn bản.

BƯỚC 4 — HỌP CHỐT (60 phút, tối đa 5 người, người accountable chủ trì)
  00-10  Driver đọc lại tiêu chí và điểm bất đồng còn lại (KHÔNG đọc lại
         toàn bộ phương án — mọi người đã đọc)
  10-35  Mỗi tác giả 5 phút trả lời phản biện nặng nhất về phương án mình
  35-45  Vòng bắt buộc: mỗi người nói 1 câu về phương án mình chọn và
         tiêu chí quyết định. Người ít thâm niên nhất nói TRƯỚC.
  45-55  Người accountable chốt, nói rõ tiêu chí nào đã quyết định
  55-60  Ghi ADR ngay trong phòng: quyết định, lý do, ai bất đồng và vì
         sao, điều kiện xét lại

BƯỚC 5 — ADR + ĐIỀU KIỆN XÉT LẠI (trong 24h)
  ADR ghi cả tên người bất đồng và lập luận của họ — không phải để ghi
  công hay ghi tội, mà vì đó là dữ liệu cho lần review quyết định sau.
  Điều kiện xét lại phải là thứ người bất đồng đồng ý là bằng chứng.
  Ví dụ: "nếu thời gian build vượt 12 phút hoặc số lần deploy hỏng do
  coupling vượt 2 lần/quý trước 31/12, Quân mở lại quyết định này."

TIÊU CHÍ BIẾT LÀ XONG: có ADR, có 1 tên người accountable, mọi người
biết mình phải làm gì sáng thứ Hai, và người bất đồng biết điều kiện
nào cho phép họ mở lại.
```

Chi tiết ở bước 4, mốc 35–45: **người ít thâm niên nhất nói trước.** Lý do là chống anchoring, và
trong bối cảnh Việt Nam nó còn giải quyết một vấn đề mạnh hơn — sau khi người thâm niên cao đã nêu
quan điểm, chi phí xã hội của việc nói khác đi tăng lên đáng kể. Đảo thứ tự phát biểu là biện pháp
gần như miễn phí và có tác dụng lớn.

**Script hội thoại: chốt một quyết định khi hai Senior không hội tụ sau 90 phút.**

Bối cảnh: buổi họp thứ ba, phút 88. Khoa muốn tách service, Quân muốn giữ monolith module hoá. Tuấn là
Tech Lead, accountable.

*Phiên bản nói sai — cách 1, hoãn:*

> **Tuấn:** Thôi hai bạn nói đều có lý. Để mình nghĩ thêm, tuần sau bàn tiếp nhé.

Chi phí: một tuần trễ, và một tín hiệu rằng Tuấn không quyết được. Sau hai lần như vậy, Khoa và Quân
sẽ không mang tranh luận tới Tuấn nữa — họ sẽ tự giải quyết bằng cách ai viết code trước thì thắng.

*Phiên bản nói sai — cách 2, thoả hiệp giữa:*

> **Tuấn:** Hay là mình làm nửa nửa? Tách phần thanh toán ra service riêng, còn lại giữ trong monolith?

Chi phí: có thể tạo ra kiến trúc kém mạch lạc hơn cả hai phương án gốc, và nó khiến cả Khoa và Quân
đều không cam kết vì không ai thấy đây là phương án của mình. Thoả hiệp về mặt xã hội tạo ra nợ về mặt
kỹ thuật.

*Phiên bản nói sai — cách 3, dùng quyền:*

> **Tuấn:** Mình là lead, mình quyết tách service. Khoa làm đi.

Chi phí: có thể quyết định đúng, nhưng Quân vừa dành 90 phút để không được nghe. Lần sau Quân sẽ nói
ít hơn, và tổ chức mất phần thông tin mà Quân có.

*Phiên bản nói đúng:*

> **Tuấn:** Dừng ở đây. Chúng ta đã dùng 90 phút và tôi thấy chúng ta không hội tụ thêm, không phải
> vì ai lập luận yếu mà vì hai bạn đang tối ưu hai thứ khác nhau. Để tôi kiểm tra xem tôi hiểu đúng
> không.
>
> Khoa, bạn ưu tiên **khả năng deploy độc lập** — bạn lo rằng trong 12 tháng nữa team sẽ 20 người và
> mọi thay đổi đặt hàng phải deploy cả monolith. Đúng chưa?
>
> **Khoa:** Đúng, và cả chuyện scale riêng phần đặt hàng trong campaign.
>
> **Tuấn:** Quân, bạn ưu tiên **chi phí vận hành và độ phức tạp hiện tại** — bạn lo rằng mình chưa có
> distributed tracing, chưa có ai từng debug một transaction qua ba service, và bài toán consistency
> giữa đơn hàng với tồn kho sẽ khó hơn nhiều. Đúng chưa?
>
> **Quân:** Đúng. Và mình chỉ có 6 người.
>
> **Tuấn:** Cả hai lo lắng đều thật và tôi không nghĩ dữ liệu nào trong phòng này giải quyết được sự
> khác nhau đó, vì nó phụ thuộc vào một thứ chưa xảy ra: team có thành 20 người trong 12 tháng hay
> không. Cái đó không phải câu hỏi kỹ thuật.
>
> Tôi là người accountable cho quyết định này, nên tôi chốt. Tôi chọn phương án của Quân: giữ monolith,
> tách theo module boundary rõ ràng, có ranh giới ở tầng code chứ chưa ở tầng deploy. Tiêu chí quyết
> định của tôi là **chi phí vận hành so với năng lực hiện tại của team**, và tôi cho tiêu chí đó trọng
> số cao hơn khả năng deploy độc lập trong 12 tháng tới, vì nếu mình sai ở tiêu chí này thì mình sai
> ngay bây giờ, còn nếu mình sai ở tiêu chí kia thì mình sai vào năm sau và lúc đó vẫn còn đường đi.
>
> Khoa, tôi muốn nói rõ ba việc. Một: lập luận của bạn không bị bác, nó thua ở trọng số, và trọng số
> đó là phán đoán của tôi — nếu tôi sai thì tôi sai, không phải bạn. Hai: tôi ghi vào ADR rằng bạn bất
> đồng và ghi nguyên văn lý do, để sáu tháng nữa khi mình review lại thì có dữ liệu. Ba: tôi muốn điều
> kiện xét lại do bạn đặt. Tín hiệu nào, đo được, sẽ chứng minh rằng đã đến lúc tách?
>
> **Khoa:** Nếu thời gian build và test vượt 15 phút, hoặc nếu có hơn 2 lần trong một quý mà deploy
> module khác làm hỏng luồng đặt hàng.
>
> **Tuấn:** Ghi vào. Cộng thêm một mốc thời gian: xét lại vào 31/12 dù hai điều kiện kia chưa xảy ra.
> Bạn là chủ của điều kiện xét lại này, tức là bạn có quyền mở lại mà không cần xin phép tôi khi nó
> xảy ra.
>
> Còn một việc nữa và nó quan trọng: từ chiều nay, khi ai hỏi về quyết định này, cả hai bạn nói cùng
> một câu trả lời — mình giữ monolith module hoá, đây là lý do, đây là điều kiện xét lại. Khoa, bạn
> được quyền nói thêm rằng bạn đã đề xuất khác và bạn không đồng ý — điều đó ghi trong ADR và tôi không
> yêu cầu bạn giả vờ đồng ý. Cái tôi yêu cầu là bạn không triển khai theo cách thứ ba, và nếu bạn thấy
> quyết định này đang gây hại thì bạn nói với tôi trước khi nói với người khác. Được không?
>
> **Khoa:** Được.

*Câu đóng cuộc họp mà không phá quan hệ — phân tích cấu trúc:*

Câu chốt của Tuấn làm sáu việc, và mỗi việc xử lý một cơ chế cụ thể:

| Thành phần | Câu trong script | Cơ chế nó xử lý |
|---|---|---|
| Nêu lại lập luận của cả hai bằng lời của mình, chính xác | "Khoa, bạn ưu tiên khả năng deploy độc lập..." | Người ta chấp nhận thua khi được nghe; và việc nêu lại chứng minh đã nghe, không chỉ tuyên bố đã nghe |
| Đặt tên cho nguồn bất đồng | "hai bạn đang tối ưu hai thứ khác nhau" | Chuyển từ xung đột giữa người sang khác biệt về tiêu chí — thứ có thể bàn mà không mất mặt |
| Nêu tiêu chí đã quyết định, không chỉ kết quả | "tiêu chí quyết định của tôi là chi phí vận hành..." | Tạo ra thứ có thể phản biện và tái dùng; nếu chỉ nêu kết quả, quyết định trở thành ý chí |
| Nhận accountability tường minh | "nếu tôi sai thì tôi sai, không phải bạn" | Gỡ rủi ro cá nhân khỏi người bất đồng, làm việc cam kết thực thi rẻ hơn về mặt tâm lý |
| Cho người bất đồng quyền sở hữu điều kiện xét lại | "tín hiệu nào, đo được, sẽ chứng minh..." | Biến "thua" thành "hoãn có điều kiện"; người bất đồng còn một con đường hợp pháp |
| Nói rõ ranh giới của disagree-and-commit | "bạn được quyền nói bạn không đồng ý... cái tôi yêu cầu là bạn không triển khai theo cách thứ ba" | Tách sự đồng ý khỏi sự cam kết, và định nghĩa cụ thể hành vi được và không được |

Điều Tuấn **không** làm, và đây là phần đáng chú ý: không yêu cầu Khoa nói "em đồng ý", không nói "cả
hai đều đúng", không dùng "mình là lead" làm luận cứ, và không thoả hiệp về mặt kỹ thuật để làm dịu về
mặt xã hội.

### Trade-off

**Consensus vs Speed.** Consensus cho chất lượng cam kết cao nhất và chi phí thời gian cao nhất.
Nghiêng về consensus khi: quyết định cần mọi người thay đổi hành vi hàng ngày (coding convention, quy
trình review, cách làm việc), vì ở đó việc thực thi phân tán và không thể cưỡng chế. Nghiêng về một
người quyết khi: quyết định kỹ thuật có một người rõ ràng gánh hậu quả, hoặc khi cost of delay cao.
Cái mất khi luôn dùng consensus: quyết định trung bình và decision latency dài. Cái mất khi luôn dùng
một người quyết: tổ chức không phát triển được năng lực ra quyết định ở tầng dưới, và bạn thành điểm
nghẽn — xem phần Delegation trong `03-team-leadership.md`.

**Nhiều người tham gia (chất lượng thông tin) vs ít người (tốc độ và rõ ràng).** Thêm người thứ 6, 7
vào một quyết định thường thêm 0 thông tin mới và thêm 30–50% thời gian, vì thông tin trong nhóm nhỏ
đã bao phủ phần lớn không gian. Ngoại lệ đáng giá: người mang vào một *loại* thông tin không ai khác
có (người vận hành hệ thống hạ nguồn, người trực on-call, người từng làm việc này thất bại ở nơi khác).
Quy tắc: mỗi người thêm vào phải trả lời được câu "thông tin gì mà không ai khác trong phòng có".

**Tường minh về quyền lực vs không khí bình đẳng.** Nói rõ "tôi là người quyết" làm hierarchy hiện ra
và có thể cảm giác trái với văn hoá team phẳng. Nhưng hierarchy vẫn tồn tại dù không nói ra, và khi
không nói ra, nó vận hành theo cách tệ hơn: quyết định được đưa ra bởi người có ảnh hưởng xã hội cao
nhất mà không ai chịu accountability. Nghiêng về tường minh trong hầu hết trường hợp; cái mất là một
chút cảm giác bình đẳng, cái được là mọi người biết luật chơi và không bị bất ngờ.

### Real-world Scenarios

**Bối cảnh Việt Nam: người im lặng trong họp rồi phản đối lúc triển khai.**

Đây là mẫu hành vi phổ biến nhất và tốn kém nhất trong ra quyết định nhóm ở nhiều team Việt Nam. Bối
cảnh minh hoạ: Nam (Senior, 8 năm kinh nghiệm, hơn Tuấn 3 tuổi) không nói gì trong buổi họp chốt về
việc chuẩn hoá cách viết test. Hai tuần sau, PR của Nam không có test theo chuẩn mới. Khi được hỏi,
Nam nói "chuẩn đó không phù hợp với module này". Trong ba tuần tiếp theo, hai Mid-level thấy Nam không
làm nên cũng không làm.

Cơ chế phía sau, cần hiểu đúng để xử đúng — đây không phải sự thiếu trung thực:

1. **Chi phí xã hội của việc phản đối công khai người trẻ hơn hoặc trong phòng đông là cao.** Phản đối
   trực diện có thể bị đọc là làm mất mặt người đề xuất, và ở đây có thêm yếu tố tuổi tác.
2. **Im lặng không có nghĩa đồng ý, và trong nhiều bối cảnh nó là cách nói "tôi không đồng ý nhưng
   không muốn nói ở đây".** Người trong cùng bối cảnh văn hoá thường hiểu tín hiệu này; người lead
   thiếu kinh nghiệm thì đọc im lặng thành đồng thuận.
3. **Không có kênh nào để bất đồng với chi phí xã hội thấp.** Nếu kênh duy nhất là phát biểu trong
   phòng đông người, thì bất đồng sẽ chuyển sang kênh còn lại: hành vi lúc triển khai.

Bốn biện pháp cụ thể, xếp theo thứ tự nên làm:

**Một — tạo kênh bất đồng bằng văn bản, async, trước buổi họp.** Thời hạn phản biện trên tài liệu (bước
3 của quy trình trên) là biện pháp có hiệu lực cao nhất, vì viết không cần đối đầu trực diện và cho
người viết thời gian chọn từ. Nhiều người Việt phản biện rất thẳng và rất tốt bằng văn bản, cùng người
đó im lặng trong phòng.

**Hai — đổi câu hỏi chốt phòng.** Không dùng "mọi người ok chứ" (câu này mời im lặng). Dùng vòng bắt
buộc mỗi người một câu, người ít thâm niên nói trước, và câu hỏi theo kiểu consent:

> *Nói sai:* "Vậy chốt nhé, mọi người ok chứ? ... Không ai nói gì, vậy là ok."
>
> *Nói đúng:* "Tôi đi một vòng, mỗi người một câu, không được nói 'em đồng ý'. Câu hỏi là: **nếu
> phương án này sai, nó sẽ sai ở đâu?** Hà nói trước, rồi Nam, rồi Khoa."

Câu hỏi "nó sẽ sai ở đâu" làm ba việc: nó hợp thức hoá việc nói điều tiêu cực (bạn đang được yêu cầu
làm việc đó), nó không đòi ai phải phản đối ai (bạn đang nói về phương án, không về người), và nó cho
ra thông tin dùng được kể cả từ người đồng ý.

**Ba — với người như Nam, nói riêng trước buổi họp, không phải trong buổi họp.** Một cuộc trò chuyện
1-1 15 phút trước buổi họp: "Anh Nam, chiều nay em định chốt chuyện chuẩn test. Em muốn hỏi anh trước
vì anh làm module thanh toán lâu nhất. Có gì trong đề xuất này sẽ gây khó cho module đó không?" Đây
không phải chính trị hành lang — đây là thu thập thông tin qua kênh có chi phí xã hội thấp, và nó có
thêm tác dụng cho Nam vị thế được hỏi ý kiến trước, làm giảm nhu cầu khẳng định vị thế trong phòng.

**Bốn — khi hành vi đã xảy ra, xử ở tầng hành vi, riêng, và không dùng buổi họp làm phiên toà.**

> *Nói sai (trong họp team):* "Anh Nam, PR của anh không theo chuẩn mình đã thống nhất. Mình đã chốt
> trong họp rồi mà."
>
> *Nói đúng (1-1):* "Anh Nam, em thấy PR module thanh toán không theo chuẩn test mới. Em nghĩ là anh
> có lý do và em muốn nghe. Nhưng trước đó em cần nói một việc về cách làm việc: trong buổi họp hôm đó
> anh không nói gì, nên em hiểu là không có phản đối, và em chốt. Nếu lúc đó em hiểu sai thì lỗi thuộc
> về cách em chốt — em đã hỏi 'mọi người ok chứ' và đó là câu hỏi tệ. Từ giờ em sẽ đi một vòng hỏi từng
> người. Nhưng đổi lại em cần một cam kết: nếu anh không đồng ý, anh nói với em — trong họp, hoặc
> comment trên tài liệu, hoặc nhắn riêng cho em, kênh nào cũng được. Cái em không làm việc được là
> quyết định bị đảo ở tầng code mà em biết sau ba tuần. Vì lúc đó hai bạn Mid cũng làm theo anh, và em
> mất cả chuẩn lẫn khả năng biết mình đang ở đâu. Giờ anh nói cho em nghe vấn đề với module thanh toán."

Cấu trúc của đoạn này: lead nhận phần lỗi thuộc về mình trước (câu hỏi chốt phòng tệ) — điều này gỡ
thế đối đầu; nêu hậu quả hệ thống cụ thể (hai Mid làm theo, mất khả năng quan sát) thay vì nêu sự vi
phạm; đưa ra nhiều kênh để bất đồng thay vì đòi hỏi một kênh; và kết bằng việc thực sự hỏi về vấn đề
kỹ thuật — vì có khả năng Nam đúng, và nếu Nam đúng thì quyết định cần đổi.

**Ba tầng đọc tình huống Nam:**

- **IC:** "Nam không tuân thủ, lead phải ép Nam." Cách này có thể tạo tuân thủ ngắn hạn và mất Nam
  trong sáu tháng.
- **Tech Lead:** "cách tôi chốt quyết định đang tạo ra consensus giả." Đây là tầng sửa được, và sửa
  bằng công cụ: vòng bắt buộc, thời hạn phản biện async, hỏi riêng trước.
- **Manager:** "vì sao một Senior 8 năm không có kênh nào để nói không?" Có thể là vấn đề Psychological
  Safety trong team, hoặc vấn đề vai — Nam có thâm niên nhưng không có vai chính thức nào trong quyết
  định kỹ thuật, và hành vi ở tầng code là cách duy nhất Nam có để thể hiện chuyên môn. Nếu đúng vậy,
  giải pháp không nằm ở decision process mà ở thiết kế vai — xem `09-to-chuc-va-scaling.md`.

### Best Practices

- **Nói mức tham gia ở đầu buổi họp, bằng một câu.** Lý do: nguồn bất mãn lớn nhất không phải kết quả
  quyết định mà là kỳ vọng sai về quyền của mình trong quyết định đó.
- **Đúng một tên người ở vai Approver, và tên đó được nói ra trong phòng.** Lý do: hai người quyết
  nghĩa là không ai quyết; và nếu tên không được nói ra, mọi người sẽ đoán, thường đoán là người có
  title cao nhất.
- **Người ít thâm niên phát biểu trước trong vòng bắt buộc.** Lý do: chống anchoring, và giảm chi phí
  xã hội của việc nói khác người thâm niên cao. Đây là biện pháp rẻ nhất có tác dụng lớn nhất trong
  bối cảnh Việt Nam.
- **Dùng câu hỏi consent ("có rủi ro nghiêm trọng nào không") thay câu hỏi consensus ("mọi người ok
  chứ").** Lý do: nó hạ ngưỡng cần đạt và nó hỏi đúng loại thông tin bạn cần.
- **Ghi tên người bất đồng và lập luận của họ vào ADR.** Lý do: nó là dữ liệu cho lần review sau; nó
  cho người bất đồng thấy họ được ghi nhận, làm việc cam kết thực thi khả thi hơn; và nó chống việc lịch
  sử bị viết lại.
- **Cho người bất đồng quyền sở hữu điều kiện xét lại.** Lý do: nó chuyển trạng thái từ "tôi thua"
  sang "tôi có một con đường hợp pháp và một mốc thời gian", và nó tạo ra một người có động lực theo
  dõi tín hiệu — thường là người theo dõi tốt nhất.
- **Đặt deadline cho quyết định trước khi bắt đầu thảo luận.** Lý do: nó biến bế tắc từ một kết cục
  khả thi thành một kết cục không khả thi. Câu chuẩn: "chúng ta chốt trước 17h thứ Năm; nếu chưa hội
  tụ, tôi quyết."

### Anti-patterns

**Decision by Highest Paid Person (HiPPO).** Biểu hiện: sau khi CTO nêu quan điểm, thảo luận kết thúc
trong 2 phút. Cơ chế phá hoại: nó không chỉ có thể cho quyết định sai, nó **triệt tiêu đầu vào cho các
quyết định tương lai** — người trong phòng học được rằng phát biểu trước khi biết ý người cao nhất là
rủi ro, nên họ chờ, nên tổ chức mất phần thông tin phân tán vốn là lý do duy nhất để họp. Dấu hiệu
sớm: có người hỏi bạn "anh nghĩ CTO muốn cái nào?" trước buổi họp. Biện pháp cụ thể cho người có
title cao: **phát biểu cuối cùng, và nói rõ mình đang đặt câu hỏi chứ không đang chỉ đạo** — trong tổ
chức có khoảng cách quyền lực, một câu hỏi từ người cao nhất thường được nghe như một chỉ thị, nên
phải nói rõ.

**Giả consensus.** Biểu hiện: "mọi người ok chứ?" — im lặng — "vậy chốt nhé". Cơ chế: im lặng có nhiều
nghĩa (đồng ý, không hiểu, không quan tâm, không đồng ý nhưng không muốn nói, đang nghĩ về việc khác),
và cách chốt này gộp tất cả thành "đồng ý". Chi phí không hiện ra trong buổi họp mà hiện ra 2–4 tuần
sau ở tầng thực thi, lúc đó đắt hơn nhiều. Dấu hiệu sớm: các buổi họp quyết định của bạn thường kết
thúc sớm hơn dự kiến và không có ai đặt câu hỏi khó.

**Consensus như một yêu cầu ngầm.** Biểu hiện: lead không chốt vì "còn Khoa chưa đồng ý". Cơ chế: nó
trao quyền veto cho mọi người có mặt, kể cả người không có accountability; hệ quả là quyết định bị bắt
làm con tin bởi người kiên trì nhất, không phải người đúng nhất. Dấu hiệu sớm: cùng một chủ đề vào
buổi họp thứ ba.

**Thoả hiệp kỹ thuật để giải quyết xung đột xã hội.** Biểu hiện: "nửa nửa" — tách một phần service,
giữ một phần monolith, không theo nguyên tắc nào ngoài việc làm cả hai bên đỡ mất mặt. Cơ chế: trên
các trục thiết kế không nội suy được, điểm giữa mang chi phí của cả hai đầu và lợi ích của không đầu
nào; thêm nữa, không ai cam kết vì không ai sở hữu phương án. Dấu hiệu sớm: phương án được chốt không
được nêu trong bất kỳ RFC nào và xuất hiện lần đầu ở phút 85 của buổi họp.

**Disagree-and-commit dùng làm công cụ dập phản biện.** Biểu hiện: "chốt rồi, disagree and commit
nhé" nói ngay sau khi lead nêu quyết định, không có bước nghe. Cơ chế: nguyên tắc này chỉ hợp lệ khi
phần "được nghe" đã xảy ra; thiếu nó, nó là một câu lệnh khoác vỏ ngoài của một nguyên tắc quản trị,
và team sẽ nhận ra rất nhanh. Dấu hiệu sớm: cụm từ này xuất hiện trong họp thường xuyên hơn cụm từ
"để tôi nhắc lại lập luận của bạn".

**Mời quá nhiều người vào một quyết định để chia sẻ rủi ro.** Biểu hiện: buổi họp kiến trúc có 9 người,
trong đó 5 người không nói gì. Cơ chế: lead đang phân tán rủi ro của một quyết định khó bằng cách tăng
số người "đã đồng ý", nhưng accountability không chia được — nó chỉ biến mất. Dấu hiệu sớm: khi được
hỏi ai quyết, câu trả lời là "cả nhóm đã thống nhất".

### Khi nào KHÔNG nên áp dụng

- **Quyết định mức 1–2.** Quy trình 5 bước cho một quyết định sửa được trong một ngày là lãng phí và
  gây khó chịu. Với vùng này, quy tắc là owner quyết, code review là cơ chế điều chỉnh.
- **Trong incident.** Ra quyết định nhóm là chế độ sai khi hệ thống đang down. Chế độ đúng là Incident
  Command: một người quyết, những người khác thực thi và báo cáo trạng thái, không tranh luận về
  phương án. Bàn luận diễn ra ở Postmortem, và ở đó thì các nguyên tắc trong chủ đề này áp dụng đầy đủ.
- **Khi bạn không thực sự để nhóm quyết mà chỉ muốn nhóm cảm thấy được tham gia.** Nếu quyết định đã
  được chốt ở tầng trên, tổ chức một buổi thảo luận "mở" là hành vi có hại hơn cả việc thông báo trực
  tiếp: nó tiêu thời gian của mọi người, và khi họ nhận ra (họ sẽ nhận ra), nó phá tín nhiệm của bạn
  cho mọi buổi thảo luận thật sau này. Trong trường hợp này dùng mức 1 của thang tham gia: thông báo,
  giải thích lý do, nhận câu hỏi.
- **Khi cost of delay cực cao và cửa sổ tính bằng giờ.** Cửa sổ cơ hội 24 giờ, đối tác chờ câu trả lời
  trong buổi chiều, hoặc một lỗ hổng bảo mật đang bị khai thác. Dùng phiên bản rút gọn: người
  accountable quyết trong 30 phút với input của 1–2 người, ghi lại 5 dòng, review sau. Điểm cần giữ dù
  rút gọn: vẫn phải có một tên người và một bản ghi ngắn.
- **Khi bất đồng thực chất không phải về kỹ thuật.** Nếu Khoa và Quân tranh luận bảy tuần và bạn nghi
  rằng vấn đề thật là ai được coi là người có tiếng nói kỹ thuật trong team, thì mọi cải tiến quy trình
  quyết định sẽ chỉ chuyển xung đột sang chủ đề khác. Kiểm tra bằng câu hỏi: nếu tôi chốt phương án
  của người này, người kia sẽ thực sự vấn đề với kiến trúc, hay với việc mình bị bỏ qua? Nếu là cái
  thứ hai, đây là bài toán conflict resolution và thiết kế vai, không phải bài toán decision process
  — xem `03-team-leadership.md`.

---

## 8. Decision Log và học từ quyết định

### Problem Statement

Một Tech Lead được hỏi trong buổi review quý: "Sáu tháng trước mình chọn kiến trúc event-driven cho
luồng đơn hàng. Bây giờ nhìn lại, quyết định đó đúng không?" Câu trả lời: "Em nghĩ là đúng. Lúc đó
mình biết là sẽ có nhiều consumer." Kiểm tra lại tài liệu thiết kế viết sáu tháng trước: lý do chính
được nêu là giảm coupling giữa hai module, và số consumer dự kiến là 2. Hiện tại có 2 consumer. Lập
luận "biết là sẽ có nhiều consumer" không tồn tại vào thời điểm quyết định — nó được ký ức tạo ra sau
khi đã biết kết quả.

Đây không phải sự thiếu trung thực. Đây là hindsight bias, và nó vận hành ở mọi người, kể cả người
biết về nó.

Hiện tượng đếm được của một tổ chức không có decision log:

- **Số quyết định kiến trúc trong 12 tháng qua mà bạn tìm được bản ghi kèm lý do.** Nếu dưới 30%,
  tổ chức không có bộ nhớ quyết định.
- **Số quyết định được xem lại có chủ ý sau khi kết quả đã hiện ra.** Ở phần lớn tổ chức con số này là
  0. Nghĩa là hoạt động ra quyết định là hoạt động duy nhất trong engineering không có feedback loop —
  trong khi code có test, hệ thống có monitoring, sprint có retro.
- **Thời gian trung bình để trả lời câu hỏi "vì sao hệ thống ở trạng thái này".** Nếu một engineer mới
  cần hỏi ba người và mất hai ngày để hiểu vì sao có hai cách gọi API trong cùng codebase, chi phí đó
  nhân với mỗi người mới.
- **Số lần một quyết định bị mở lại mà không có tín hiệu mới nào.** Đã đo ở chủ đề 1.
- **Khoảng cách giữa lý do được nhớ và lý do được ghi.** Bài kiểm tra thú vị và khó chịu: chọn ba quyết
  định 6–12 tháng trước, hỏi người quyết nêu lý do từ ký ức, rồi so với bản ghi. Ở phần lớn trường
  hợp, khoảng cách lớn hơn người đó tưởng.

### First Principles

**Cơ chế một: không có bản ghi thì không có learning loop, và không có learning loop thì kinh nghiệm
không chuyển thành năng lực.** Để học từ một quyết định cần ba dữ kiện: bạn đã dự đoán gì, thực tế xảy
ra gì, và khoảng cách đến từ đâu. Thiếu dữ kiện đầu, hai dữ kiện còn lại không tạo ra bài học nào —
bạn chỉ quan sát được kết quả và gán cho nó một câu chuyện. Đây là lý do "20 năm kinh nghiệm" có thể
nghĩa là 20 năm học hoặc một năm học lặp 20 lần: khác biệt nằm ở chỗ có ghi lại dự đoán hay không.

**Cơ chế hai: hindsight bias — con người tự viết lại lịch sử, có hệ thống và không tự phát hiện được.**
Sau khi biết kết quả, ký ức về mức tin cậy trước đó bị dịch về phía kết quả đã biết ("tôi biết mà").
Hiện tượng này đã được nghiên cứu rộng rãi từ công trình của Fischhoff những năm 1970 và nó không giảm
khi người ta được cảnh báo về nó. Ba hệ quả trực tiếp cho công việc lead:

1. **Ký ức về lý do quyết định không đáng tin**, nên mọi phân tích hồi cứu dựa trên ký ức đều bị nhiễm.
2. Trong Postmortem, hindsight bias làm mọi sự cố trông như *lẽ ra phải thấy trước*, dẫn tới việc trách
   người thay vì sửa hệ thống — đây là một trong hai cơ chế phá hoại một Postmortem, cùng với việc
   thiếu Blameless.
3. Nó làm **quá trình tốt bị đánh giá thấp và may mắn bị đánh giá cao**, vì sau khi biết kết quả, con
   đường dẫn tới nó luôn trông tất yếu.

Biện pháp duy nhất có hiệu lực là ghi lại **trước khi biết kết quả**. Không phải vì chữ viết chính xác
hơn ký ức nói chung, mà vì nó được tạo ra ở thời điểm chưa bị nhiễm.

**Cơ chế ba: quyết định sai và kết quả xấu là hai biến khác nhau, và tổ chức chỉ học được nếu tách
chúng.** Đã dựng ma trận 2×2 ở chủ đề 1. Điều cần thêm ở đây là **cách tách trên thực tế**, bằng ba câu
hỏi áp dụng theo thứ tự:

| Câu hỏi | Nếu câu trả lời là Có | Nếu câu trả lời là Không |
|---|---|---|
| 1. Thông tin làm kết quả xấu có sẵn ở thời điểm quyết định không? | Đây là **lỗi quá trình** — quá trình đã bỏ sót thông tin có sẵn. Sửa quá trình (checklist, thêm ai vào phòng, thêm bước). | Đi câu 2. |
| 2. Có cách nào chi phí hợp lý để lấy được thông tin đó không? | Đây là **lỗi phạm vi điều tra** — quá trình dừng quá sớm. Sửa: ngưỡng "khi nào cần spike". | Đi câu 3. |
| 3. Giả định đã được viết ra và mức tin cậy đã được khai báo không? | Đây là **kết quả xấu với quyết định tốt**. Không sửa gì ở người quyết. Sửa ở khả năng phát hiện sớm và hồi phục. | Đây là **lỗi tường minh hoá** — quyết định dựa trên giả định ngầm. Sửa: bắt buộc viết giả định. |

Cấu trúc ba câu này quan trọng vì nó cho ra bốn kết luận khác nhau với bốn hành động sửa khác nhau,
thay vì một kết luận nhị phân "ai sai". Và nó dùng được cả trong Postmortem lẫn trong Performance
Review.

**Cơ chế bốn: giá trị của bản ghi tăng theo số lần được tra cứu, và giảm theo chi phí tạo ra nó.** Suy
ra hai điều: chỉ ghi những quyết định có khả năng được tra cứu (mức 3–4 theo chủ đề 2), và giữ chi phí
ghi ở mức thấp nhất có thể — dưới 15 phút cho một entry. Một hệ thống decision log đòi 60 phút mỗi
entry sẽ không được dùng, và một hệ thống không được dùng có giá trị bằng 0 bất kể thiết kế tốt đến đâu.

### Mental Model

**Decision log như một tập dữ liệu huấn luyện cho phán đoán của bạn.** Cách hình dung hữu ích: mỗi
entry là một cặp (đặc trưng đầu vào, dự đoán). Sau 12 tháng bạn có 20–30 cặp, và việc so dự đoán với
thực tế là quá trình hiệu chỉnh (calibration) phán đoán của bạn. Đây là điểm khác biệt giữa lead có 5
năm kinh nghiệm hiệu chỉnh và lead có 15 năm kinh nghiệm chưa từng hiệu chỉnh: người thứ hai có nhiều
dữ liệu hơn nhưng chưa bao giờ chạy quá trình học.

**Prediction với mức tin cậy — kỹ thuật để phán đoán trở nên kiểm được.** Thay vì viết "tôi nghĩ
migration sẽ mất khoảng 3 tuần", viết: "tôi tin 80% rằng migration xong trong 5 tuần; tôi tin 50% rằng
xong trong 3 tuần". Hai câu này khác nhau về loại: câu đầu không thể sai (khoảng bao nhiêu là bao
nhiêu?), câu sau có thể. Sau 10 dự đoán ở mức 80%, nếu chỉ 4 cái đúng, bạn biết mình đang tự tin quá
mức và biết mình lệch bao nhiêu. Đó là thông tin mà 15 năm kinh nghiệm không tự cho bạn.

**Phân biệt ADR và Decision Log.** Hai thứ khác nhau và cần cả hai:

| | ADR (Architecture Decision Record) | Decision Log |
|---|---|---|
| Nằm ở đâu | Trong repo, cạnh code, versioned | Một nơi tập trung cho cả team/tổ chức |
| Phạm vi | Quyết định kỹ thuật ảnh hưởng cấu trúc code | Cả kỹ thuật, quy trình, tổ chức, con người |
| Người đọc chính | Engineer sẽ sửa code đó | Lead, người mới, chính bạn sáu tháng sau |
| Mục đích chính | Giải thích *vì sao code như thế này* | Học từ chất lượng quá trình ra quyết định |
| Có prediction không | Thường không | **Có** — đây là điểm khác biệt quan trọng nhất |

Nhiều tổ chức có ADR và không có Decision Log, nên họ giải thích được hệ thống nhưng không học được về
cách mình quyết định.

### Practical Framework

**Template decision log entry — mức tối thiểu, mục tiêu dưới 15 phút:**

```markdown
## DL-2026-014: Giữ monolith module hoá cho luồng đặt hàng

**Ngày:** 2026-07-25
**Mức (theo bảng phân loại):** 3 — khó đảo, ảnh hưởng nhiều team
**Accountable:** Tuấn (Tech Lead)
**Người tham gia:** Khoa, Quân, Hà, Linh (EM)

### Bối cảnh
Team 6 engineer, monolith 4 năm tuổi, module đặt hàng có 3 module khác phụ
thuộc. Thời gian build+test hiện tại 9 phút. Chưa có distributed tracing.
Không ai trong team từng vận hành hệ thống nhiều service ở production.
Áp lực: campaign 11/11 cần scale phần đặt hàng riêng.

### Phương án đã loại và lý do
- **Tách service riêng (Khoa đề xuất):** loại — chi phí vận hành so với năng
  lực hiện tại của team. Cụ thể: cần distributed tracing, cần giải bài toán
  consistency đơn hàng/tồn kho, ước lượng 6-8 tuần và không ai có kinh nghiệm
  debug production đa service.
- **Không làm gì:** loại — build time đang tăng 1 phút mỗi 2 tháng, và ranh
  giới module hiện tại đã bị vi phạm ở 7 chỗ (đếm bằng script phân tích import).

### Đã chọn
Giữ monolith, tách theo module boundary ở tầng code: enforce ranh giới bằng
lint rule cấm import chéo, tách schema theo module, chuẩn bị interface để
sau này tách service được.

### Giả định (và cách kiểm)
1. Team KHÔNG vượt 12 người trong 12 tháng tới. (Kiểm: headcount plan của
   Linh, xem lại mỗi quý.) — Tin cậy: TRUNG BÌNH
2. Peak campaign 11/11 xử lý được bằng scale ngang toàn monolith, chi phí
   thêm dưới 40 triệu/tháng trong 1 tháng. (Kiểm: load test trước 15/10.)
   — Tin cậy: THẤP, đây là giả định yếu nhất
3. Lint rule đủ để giữ ranh giới module. (Kiểm: đếm số exception được thêm
   vào lint config mỗi tháng; nếu >2/tháng thì giả định sai.) — Tin cậy: TRUNG BÌNH

### Dự đoán kiểm chứng được
- Tin cậy 80%: build+test vẫn dưới 15 phút vào 31/12/2026.
- Tin cậy 70%: không có incident production nào có nguyên nhân là coupling
  giữa đặt hàng và module khác trong Q3-Q4.
- Tin cậy 60%: đến 30/6/2027 chúng ta VẪN chưa cần tách service.
- Tin cậy 90%: campaign 11/11 không cần tách service để chịu tải.

### Bất đồng được ghi nhận
Khoa không đồng ý. Lập luận nguyên văn: "Chi phí tách sau sẽ cao hơn tách bây
giờ ít nhất 2 lần vì thêm 12 tháng code bám vào. Mình đang chọn trả nhiều hơn
để trả muộn hơn." Khoa cam kết thực thi phương án đã chọn.

### Điều kiện xét lại
Bất kỳ điều nào xảy ra → Khoa có quyền mở lại KHÔNG cần xin phép:
- build+test vượt 15 phút, HOẶC
- >2 lần/quý deploy module khác làm hỏng luồng đặt hàng, HOẶC
- >2 lint exception/tháng trong 2 tháng liên tiếp, HOẶC
- ngày 31/12/2026, dù không điều kiện nào xảy ra.

### Chi phí nếu quyết định này sai
Ước lượng 6-8 tuần tách service ở thời điểm sau, cộng khoảng 30% phụ trội do
code tích luỹ. Không có thiệt hại dữ liệu. Khả đảo được.
```

Ba trường quan trọng nhất và lý do:

**Prediction có mức tin cậy** — đây là trường biến decision log từ tài liệu thành công cụ học. Không có
nó, việc review sáu tháng sau sẽ chỉ là kể lại câu chuyện.

**Giả định kèm cách kiểm và mức tin cậy** — nó chỉ ra chỗ cần theo dõi, và nó thường tự tiết lộ rằng
quyết định đang treo trên một giả định yếu (ở ví dụ trên: giả định 2, tin cậy THẤP). Khi thấy điều đó,
hành động đúng thường là đi mua thông tin, không phải tranh luận thêm.

**Bất đồng được ghi nguyên văn** — ba tác dụng: dữ liệu cho lần review; sự tôn trọng làm việc cam kết
thực thi khả thi; và chống việc lịch sử bị viết lại theo hướng "hồi đó ai cũng đồng ý".

**Nghi thức review quyết định hàng quý — 90 phút:**

```
QUARTERLY DECISION REVIEW (90 phút, mỗi quý, lead + 2-4 người tham gia quyết định)

CHUẨN BỊ (Driver, 60 phút trước buổi họp)
  Lọc decision log: các entry có ngày xét lại trong quý này, cộng các entry
  có prediction đã đến hạn kiểm chứng. Thường 4-8 entry.
  Với mỗi entry: điền kết quả thực tế vào cạnh prediction. KHÔNG bình luận.

00-05  Đọc to nguyên tắc: "Chúng ta đang đánh giá QUÁ TRÌNH, không đánh giá
       NGƯỜI. Một quyết định tốt có thể ra kết quả xấu và ngược lại."
       (Đọc mỗi quý. Nghe kỳ nhưng nó có tác dụng.)

05-25  BẢNG ĐIỂM DỰ ĐOÁN (calibration)
       Với mọi prediction đã đến hạn: đúng/sai. Tính:
       - Trong các prediction mức 80%, tỉ lệ đúng thực tế là bao nhiêu?
       - Chúng ta lệch về phía tự tin quá mức hay quá thận trọng?
       - Loại prediction nào lệch nhiều nhất? (thời gian? tải? hành vi user?)
       → Output: 1-2 câu về hướng hiệu chỉnh. Ví dụ: "ước lượng thời gian
         migration của mình lệch 2,2x một cách hệ thống — từ giờ nhân 2".

25-55  VỚI 2-3 QUYẾT ĐỊNH ĐÁNG HỌC NHẤT, chạy 3 câu hỏi tách process/outcome
       (bảng ở mục First Principles). Xếp mỗi quyết định vào 1 trong 4 ô:
       - Quá trình tốt + kết quả tốt → ghi lại cái gì đã hoạt động
       - Quá trình tốt + kết quả xấu → KHÔNG sửa người. Sửa khả năng phát
         hiện sớm. Nói rõ điều này ra thành lời.
       - Quá trình xấu + kết quả tốt → Ô NGUY HIỂM NHẤT. Dành nhiều thời
         gian cho ô này. Sửa quá trình dù kết quả tốt.
       - Quá trình xấu + kết quả xấu → sửa quá trình, dễ đồng thuận

55-75  ĐIỀU KIỆN XÉT LẠI ĐÃ ĐẾN HẠN
       Với mỗi entry đến hạn: điều kiện đã xảy ra chưa?
       - Chưa → gia hạn, đặt ngày mới. Mất 2 phút, không mở lại tranh luận.
       - Rồi → mở lại quyết định, tạo entry mới. KHÔNG sửa entry cũ.

75-85  RÚT 1-2 THAY ĐỔI QUY TRÌNH cho quý tới, có người và ngày.
       Nếu buổi review không sinh ra thay đổi nào, hoặc quá trình của mình
       đã tốt (khả năng thấp), hoặc buổi review vừa thất bại.

85-90  Cập nhật log, đặt lịch quý sau.
```

Quy tắc quan trọng nhất của nghi thức này: **không sửa entry cũ.** Khi quyết định được mở lại, tạo
entry mới có link tới entry cũ. Lý do: entry cũ là bằng chứng về trạng thái hiểu biết ở thời điểm đó,
và sửa nó là phá đúng thứ tài sản mà bạn đang cố tạo ra.

### Trade-off

**Chi phí ghi vs giá trị tra cứu.** Mỗi entry tốn 10–20 phút. Với 3 quyết định mức 3–4 mỗi tháng, đó là
khoảng 1 giờ/tháng — rẻ so với một lần tái mở quyết định (30 người-giờ ở ví dụ chủ đề 7). Nhưng nếu
áp cho mọi quyết định, chi phí vượt giá trị và hệ thống sẽ chết. Điểm cân bằng: **chỉ ghi mức 3–4, và
giữ template ngắn**. Dấu hiệu bạn đang ở sai phía: entry cuối cùng được viết cách đây hơn hai tháng
(quá ít), hoặc có người phàn nàn về việc phải viết log (quá nhiều hoặc quá nặng).

**Ghi công khai vs ghi riêng.** Log công khai trong tổ chức cho giá trị lớn nhất về mặt học và
onboarding, nhưng nó làm người viết tự kiểm duyệt: mức tin cậy sẽ được viết an toàn hơn, giả định yếu
sẽ được viết mờ hơn, và bất đồng sẽ được viết ngoại giao hơn. Log riêng của cá nhân lead thì trung
thực hơn nhưng không tạo bộ nhớ tổ chức. Cách xử lý thực dụng: log quyết định công khai, và một sổ dự
đoán riêng của lead cho các phán đoán mà bạn chưa muốn công bố (ví dụ: dự đoán về việc ai sẽ nghỉ, dự
đoán về việc một dự án sẽ trễ). Cái thứ hai là công cụ hiệu chỉnh cá nhân mạnh nhất và không cần ai
biết.

**Minh bạch về sai sót vs an toàn chính trị.** Một decision log trung thực chứa bằng chứng về các
quyết định của bạn đã ra kết quả xấu. Trong tổ chức có văn hoá Blameless, đó là tài sản. Trong tổ chức
mà những thứ đó được dùng trong đánh giá cuối năm, đó là rủi ro cá nhân. Đánh giá thực tế trước khi
làm: nếu tổ chức của bạn thuộc loại thứ hai, cách vào đúng là bắt đầu bằng log của riêng bạn và bằng
việc ghi *prediction* (dễ chấp nhận, trông như kỷ luật cá nhân), không phải bằng việc đề xuất một nghi
thức review toàn tổ chức. Xây văn hoá Blameless trước — xem `06-incident-va-metrics.md`.

### Real-world Scenarios

**Tình huống: buổi review quyết định đầu tiên của một team, và cái bẫy ô "quá trình xấu + kết quả
tốt".**

Bối cảnh minh hoạ: team fintech 14 người, quý đầu tiên chạy Quarterly Decision Review. Bốn entry đến
hạn. Entry gây tranh luận nhất: sáu tháng trước, Hà (Mid-level BE) tự quyết dùng một thư viện xử lý
tiền tệ ít phổ biến (khoảng 400 star trên GitHub, một maintainer) thay vì thư viện chuẩn của ngôn ngữ,
vì nó có API tiện hơn. Không có RFC, không có review về lựa chọn này — nó đi vào codebase qua một PR
bình thường. Kết quả sau sáu tháng: thư viện chạy tốt, không có bug nào, và team tiết kiệm được khoảng
hai tuần so với việc tự viết lớp bọc quanh thư viện chuẩn.

Cách xử lý sai — và đây là cách hầu hết tổ chức xử lý: khen Hà vì đã chủ động và tiết kiệm hai tuần, đi
tiếp. Vấn đề: tổ chức vừa học rằng "tự quyết mà không review thì được thưởng nếu may". Ba tháng sau,
một người khác sẽ áp dụng bài học đó cho một thư viện xử lý mã hoá, và phân phối kết quả của loại quyết
định đó không giống phân phối của lần này.

Cách xử lý đúng, và cách nói:

> **Tuấn:** Việc này rơi vào ô mà tôi muốn dành nhiều thời gian nhất: quá trình chưa ổn, kết quả tốt.
> Hà, tôi nói phần kết quả trước: thư viện chạy tốt, tiết kiệm hai tuần, và bạn đã đọc source của nó
> trước khi dùng — điều đó tốt hơn nhiều người làm.
>
> Nhưng tôi muốn tách hai thứ. Nếu chúng ta chạy lại quyết định đó 20 lần với các thư viện khác nhau
> có cùng đặc điểm — 400 star, một maintainer, chưa có ai trong ngành mình dùng cho việc xử lý tiền —
> thì bao nhiêu lần trong 20 lần đó chúng ta gặp vấn đề? Tôi không biết chính xác, nhưng tôi nghĩ không
> phải 0. Rủi ro cụ thể: maintainer dừng, có lỗi làm tròn ở một trường hợp biên mà không ai phát hiện,
> hoặc không có bản vá khi có lỗ hổng bảo mật.
>
> Nên bài học tôi muốn ghi không phải "đừng làm như Hà". Nó là: theo bảng phân loại của mình, thư viện
> xử lý tiền là mức 3, không phải mức 1 — vì nó sinh ra dữ liệu tiền và đổi ra thì phải kiểm lại toàn
> bộ phép tính. Chúng ta đã không phân loại nó đúng, và đó là lỗi của quy trình, không phải của Hà,
> vì lúc đó bảng phân loại chưa có.
>
> Thay đổi tôi đề xuất cho quý tới: thêm một dòng vào checklist review PR — nếu PR thêm dependency mới
> chạm vào tiền, dữ liệu cá nhân, hoặc mã hoá, thì cần một dòng lý do và một người thứ hai xác nhận.
> Mất thêm 5 phút mỗi lần, và tần suất là khoảng một lần mỗi tháng. Hà, bạn viết dòng checklist đó
> được không?

Ba đặc điểm của cách nói này: nó khen phần đáng khen một cách cụ thể (đã đọc source), nó dùng lập luận
phân phối ("nếu chạy lại 20 lần") để tách kết quả khỏi quá trình mà không phủ nhận kết quả, và nó
chuyển lỗi về phía quy trình rồi giao cho Hà việc sửa quy trình — biến người có thể cảm thấy bị phê
bình thành người sở hữu giải pháp.

**Ba tầng đọc cùng buổi review đó:**

- **IC (Hà):** ban đầu cảm nhận là "mình làm tốt mà vẫn bị nói". Nếu buổi review không tách rõ hai
  trục, Hà sẽ học được bài học sai — hoặc là "đừng chủ động", hoặc là "cứ làm, nếu ra kết quả tốt thì
  không sao". Cả hai đều tệ.
- **Tech Lead (Tuấn):** thấy đây là cơ hội cài đặt một quy tắc phân loại vào chỗ nó có tác dụng nhất
  (checklist PR, tức là tầng Execution) với chi phí gần bằng 0. Việc quan trọng nhất Tuấn làm là ngăn
  tổ chức học bài học sai từ một kết quả tốt.
- **Manager (Linh, EM):** thấy hai điều. Một, việc một Mid-level tự quyết một thứ mức 3 mà không ai
  chặn là dấu hiệu team chưa có ngôn ngữ chung về mức độ quyết định — vấn đề đào tạo, không phải kỷ
  luật. Hai, cần theo dõi Hà trong hai tháng tới: nếu Hà bắt đầu hỏi trước mọi thứ, thì buổi review đã
  vô tình dạy sự thụ động, và đó là chi phí lớn hơn lợi ích của cả buổi.

### Best Practices

- **Ghi trước khi biết kết quả — nếu không giữ được điều này thì đừng ghi.** Lý do: đây là toàn bộ giá
  trị của decision log, và một entry
  viết sau khi kết quả đã hiện ra là một câu chuyện, không phải dữ liệu.
- **Mỗi entry có ít nhất một prediction kèm mức tin cậy bằng số.** Lý do: nó là thứ duy nhất cho phép
  hiệu chỉnh phán đoán, và hiệu chỉnh là cách kinh nghiệm chuyển thành năng lực.
- **Giữ template dưới 15 phút và dưới một trang.** Lý do: chi phí là lý do số một khiến các hệ thống
  này chết. Nếu phải chọn, hy sinh độ đầy đủ để giữ tần suất.
- **Gắn ngày xét lại vào lịch, không để trong tài liệu.** Lý do: một ngày ghi trong file sẽ không được
  ai nhớ; một lời nhắc trong calendar thì có. Chi phí: 30 giây khi tạo entry.
- **Không sửa entry cũ; quyết định mới tạo entry mới có link.** Lý do: bạn đang xây một bản ghi lịch
  sử, và giá trị của nó nằm chính ở chỗ nó không được cập nhật theo hiểu biết hiện tại.
- **Tách rõ hai câu "quyết định sai" và "kết quả xấu" trong mọi cuộc thảo luận, kể cả bằng cách sửa từ
  của người khác.** Lý do: từ ngữ định hình cái tổ chức học. Câu can thiệp cụ thể: "kết quả xấu thì
  đúng rồi, nhưng mình xem lại quá trình đã: lúc đó thông tin nào có sẵn mà mình không dùng?"
- **Bắt đầu bằng log của riêng bạn, không bằng một đề xuất quy trình toàn tổ chức.** Lý do: nó không
  cần ai đồng ý, nó cho bạn dữ liệu để chứng minh giá trị sau một quý, và nó tránh việc một thực hành
  tốt bị đóng dấu là "quy trình mới" và bị kháng cự.
- **Giữ một mục "quyết định tôi đã không ra" trong log riêng.** Lý do: việc quyết định không làm gì
  cũng là quyết định, và nó là loại bị bỏ ra khỏi mọi bản ghi. Ví dụ: quyết định không nói với một
  Senior về vấn đề hiệu suất của họ, quyết định không escalate một rủi ro. Đây thường là các quyết định
  có hậu quả lớn nhất.

### Anti-patterns

**Viết log sau khi biết kết quả.** Biểu hiện: entry được tạo trong buổi review, hoặc entry có mục "lý
do" phù hợp một cách đáng ngạc nhiên với kết quả đã xảy ra. Cơ chế: hindsight bias được ghi thành văn
bản, tạo ra một bản ghi *sai* mà tổ chức sẽ tin — tệ hơn không có bản ghi, vì nó dạy các bài học không
đúng với thực tế. Dấu hiệu sớm: ngày tạo file (theo git) muộn hơn nhiều so với ngày ghi trong entry.

**Log không có prediction.** Biểu hiện: entry có bối cảnh, phương án, lý do, nhưng không có gì kiểm
chứng được. Cơ chế: nó thành tài liệu giải thích (hữu ích cho onboarding) nhưng mất hoàn toàn chức năng
học; buổi review sáu tháng sau sẽ chỉ là mọi người kể lại và đồng ý với nhau. Dấu hiệu sớm: buổi review
quyết định kết thúc mà không ai bị bất ngờ về điều gì.

**Log dùng làm bằng chứng buộc tội.** Biểu hiện: trong Performance Review, ai đó dẫn một entry để chứng
minh một người đã ra quyết định sai. Cơ chế phá hoại là tức thời và không hồi phục: sau lần đầu tiên,
mọi entry sẽ được viết phòng vệ — mức tin cậy an toàn, giả định mờ, phương án loại bỏ được viết cho
người ngoài đọc. Bạn vẫn có tài liệu nhưng không còn dữ liệu. Dấu hiệu sớm: các mức tin cậy trong log
đều tập trung ở 50–60% (mức không thể sai).

**Nghi thức review tồn tại nhưng không sinh ra thay đổi.** Biểu hiện: mỗi quý họp 90 phút, kết luận
"nhìn chung mình quyết khá tốt". Cơ chế: buổi review không có output bắt buộc nên nó tối ưu cho sự dễ
chịu. Chống lại bằng cách đặt một output cứng: 1–2 thay đổi quy trình, có người và ngày. Dấu hiệu sớm:
không ai chuẩn bị trước buổi review.

**Ghi mọi thứ.** Biểu hiện: 40 entry mỗi tháng, gồm cả việc đặt tên biến và chọn thư viện format ngày.
Cơ chế: chi phí vượt giá trị, và tệ hơn, log bị pha loãng đến mức không tìm được entry quan trọng khi
cần. Dấu hiệu sớm: không ai tra cứu log; hoặc có người tra cứu và không tìm thấy thứ mình cần vì nhiễu.

**Sửa entry cũ cho khớp hiểu biết hiện tại.** Biểu hiện: mục giả định của một entry cũ được cập nhật
"cho chính xác". Cơ chế: nó xoá đúng thứ tài sản mà log tồn tại để tạo ra — bằng chứng về trạng thái
hiểu biết tại thời điểm quyết định. Dấu hiệu sớm: git history của file log có nhiều commit sửa các
entry cũ.

### Khi nào KHÔNG nên áp dụng

- **Quyết định hàng ngày, reversible, blast radius nhỏ.** Đây là điều kiện biên quan trọng nhất của
  chủ đề này. Với quyết định mức 1–2 — đặt tên, tách hàm, chọn thứ tự tham số, chọn thư viện tiện ích
  không chạm dữ liệu — ghi log là **chi phí thuần, không có phần lợi ích nào**. Lý do cụ thể: khả năng
  entry đó được tra cứu gần bằng 0; nếu quyết định sai thì sửa rẻ hơn chi phí đã bỏ ra để ghi; và code
  cùng commit message đã là bản ghi đủ tốt. Một tổ chức áp decision log cho vùng này sẽ tạo ra 90%
  nhiễu và làm chết chính hệ thống ở vùng 10% mà nó có giá trị.
- **Khi tổ chức chưa có văn hoá Blameless.** Trong môi trường mà sai sót được dùng để đánh giá người,
  decision log công khai là một rủi ro cá nhân cho mọi người viết nó, và nó sẽ được viết phòng vệ ngay
  từ entry đầu. Thứ tự đúng: xây Blameless ở Postmortem trước (nơi có động lực rõ ràng vì ai cũng muốn
  hệ thống ít sự cố hơn), rồi mở rộng sang decision log. Trong lúc chờ, dùng log riêng.
- **Khi tần suất quyết định mức 3–4 rất thấp.** Một team ODC làm maintenance có thể ra 2 quyết định mức
  3 mỗi năm. Với tần suất đó, không có đủ dữ liệu để hiệu chỉnh và không cần nghi thức review hàng quý
  — chỉ cần ghi ADR cho hai quyết định đó và xem lại khi có việc liên quan.
- **Khi bạn đang ở giai đoạn phải chứng minh khả năng giao hàng.** Với một lead mới nhận vai và chưa có
  tín nhiệm, việc đề xuất một nghi thức review quyết định hàng quý sẽ bị đọc là thêm quy trình. Thứ tự
  đúng: giao được vài lần, rồi mới đề xuất. Đây là ràng buộc về thứ tự, không phải về giá trị của thực
  hành.
- **Khi việc ghi lại làm chậm một quyết định cần ra ngay.** Nếu cửa sổ là 2 giờ, quyết trước, ghi sau
  trong ngày. Cái không được làm là để việc ghi lại thành lý do trì hoãn — bản ghi phục vụ quyết định,
  không phải ngược lại.

---

## Tự kiểm tra

Bảy câu dưới đây chỉ có giá trị nếu bạn trả lời bằng **tên người, con số, hoặc ngày cụ thể** của tổ
chức bạn đang làm. Câu trả lời dạng "chúng tôi làm khá tốt" nghĩa là chưa trả lời.

1. **Ba quyết định kỹ thuật mức 3–4 gần nhất của team bạn là gì, và với mỗi cái, ai là người
   accountable (một tên người)?** Nếu bạn không nêu được ba quyết định, hoặc nếu có cái nào mà câu trả
   lời là "cả team", hãy viết ra ngay hôm nay tên người cho từng cái — và nhắn cho người đó biết.

2. **Trong 12 tháng qua, có quyết định nào bị mở lại từ đầu mà không có tín hiệu mới nào không? Bao
   nhiêu buổi họp, bao nhiêu người, tổng bao nhiêu người-giờ?** Nhân với chi phí giờ của team bạn để có
   con số tiền. Đó là số tiền một decision log có thể tiết kiệm.

3. **Mở backlog của bạn ra: có bao nhiêu item gắn nhãn ưu tiên cao nhất, và capacity một sprint của
   team là bao nhiêu item?** Chia hai số đó. Nếu tỉ lệ trên 5, hãy viết ra ba item là số 1, số 2, số 3
   — xếp hạng tuyến tính, không dùng nhãn — trước cuối tuần này.

4. **Tỉ lệ capacity dành cho technical health mà team bạn đã cam kết là bao nhiêu, và con số thực tế
   của quý vừa rồi là bao nhiêu?** Nếu bạn không đo được con số thực tế, thì cam kết đó chưa tồn tại.
   Ngày nào bạn sẽ có con số đầu tiên?

5. **Rủi ro nghiêm trọng nhất của dự án đang chạy là gì, ai là chủ rủi ro đó (một tên người), tín hiệu
   sớm nhất là gì, và ai hoặc hệ thống nào sẽ thấy tín hiệu đó?** Nếu bạn không trả lời được cả bốn
   phần, hãy dành 45 phút cho một pre-mortem theo quy trình ở chủ đề 5 trước ngày nào?

6. **Lần gần nhất bạn chốt một quyết định trong họp, bạn đã dùng câu gì để đóng?** Viết lại nguyên văn.
   Nếu đó là "mọi người ok chứ" hoặc tương đương, và nếu có ai đó trong team bạn sau đó triển khai khác
   với quyết định — người đó tên gì, và bạn đã nói chuyện riêng với họ chưa?

7. **Viết ra ba dự đoán kèm mức tin cậy bằng số về ba quyết định bạn đang ra trong tháng này, và một
   ngày cụ thể để kiểm chứng chúng.** Ví dụ: "tin cậy 70% rằng migration xong trước 30/9". Đặt lời nhắc
   trong calendar vào ngày đó. Sau ba tháng, đếm tỉ lệ đúng ở mỗi mức tin cậy — đó là dữ liệu đầu tiên
   về phán đoán của chính bạn, và không ai có thể cho bạn nó.

---

## Liên kết chương khác

- [`00-nen-tang-leadership.md`](/series/engineering-leedership/00-nen-tang-leadership/) — Accountability và Ownership là điều kiện
  tiên quyết của chương này: mọi framework ở đây sụp nếu trường "một tên người accountable" không có
  nghĩa gì trong tổ chức của bạn.
- [`01-self-leadership.md`](/series/engineering-leedership/01-self-leadership/) — Số lượng quyết định chất lượng cao mỗi ngày là
  hữu hạn và phụ thuộc vào quản lý năng lượng; chương đó giải thích cơ chế, chương này giải thích cách
  chi tiêu quỹ đó.
- [`02-communication.md`](/series/engineering-leedership/02-communication/) — Cost-Benefit Analysis (chủ đề 4) chỉ có tác dụng nếu
  bạn trình bày được nó với stakeholder không có nền kỹ thuật; chương đó là phần kỹ thuật truyền đạt,
  chương này là phần nội dung được truyền đạt.
- [`03-team-leadership.md`](/series/engineering-leedership/03-team-leadership/) — Delegation là việc phân bổ *quyền ra quyết định*;
  bảng phân loại 4 mức ở chủ đề 2 là công cụ cụ thể để quyết định delegate cái gì cho ai, và
  Psychological Safety là điều kiện để câu hỏi "nó sẽ sai ở đâu" nhận được câu trả lời thật.
- [`05-technical-leadership.md`](/series/engineering-leedership/05-technical-leadership/) — ADR, RFC và Technical Debt là các hiện
  vật mà chương này tạo ra và tiêu thụ; chương đó nói về nội dung kỹ thuật của quyết định, chương này
  nói về quá trình sinh ra nó.
- [`06-incident-va-metrics.md`](/series/engineering-leedership/06-incident-va-metrics/) — Postmortem là nơi vòng học của quyết định
  đóng lại; nguyên tắc tách "quyết định sai" khỏi "kết quả xấu" (chủ đề 8) là nguyên tắc Blameless áp
  cho quyết định, và DORA metrics là dữ liệu đầu vào cho cả risk assessment lẫn cost-benefit.
- [`07-project-delivery.md`](/series/engineering-leedership/07-project-delivery/) — Prioritization (chủ đề 6) và Risk Assessment
  (chủ đề 5) là hai đầu vào trực tiếp của lập kế hoạch và Estimation; cost of delay là biến nối giữa
  hai chương.
- [`09-to-chuc-va-scaling.md`](/series/engineering-leedership/09-to-chuc-va-scaling/) — Khi tổ chức lớn, chi phí ra quyết định nhóm
  tăng theo n²; chương đó nói về cách thiết kế ranh giới team để giảm số quyết định phải phối hợp, tức
  là giải bài toán ở tầng cấu trúc thay vì tầng quy trình.
- [`10-case-studies.md`](/series/engineering-leedership/10-case-studies/) — Các tình huống dài đi qua trọn vẹn chuỗi bối cảnh →
  phương án → trade-off → quyết định → hậu quả → bài học, dùng đúng các framework của chương này.
- [`12-anti-patterns.md`](/series/engineering-leedership/12-anti-patterns/) — Tổng hợp các anti-pattern xuyên chương; các mẫu HiPPO,
  giả consensus, analysis paralysis và "mọi thứ là P0" xuất hiện ở đó cùng các biểu hiện ở tầng tổ chức.

