+++
title = "Level 2B — Team Leadership: Delegation, Mentoring, Conflict và Psychological Safety"
date = "2026-08-01T11:00:00+07:00"
draft = false
tags = ["engineering-leadership", "management", "career"]
series = ["Engineering Leadership & Technical Management"]
+++

# Level 2B — Team Leadership

Một team 8 người ở một công ty logistics tại TP.HCM. Trên giấy tờ: 3 Senior, 3 Mid, 2 Junior, tổng
cộng 34 năm kinh nghiệm. Thực tế: throughput của team bằng khoảng 3,5 người. Không ai lười. Lý do
nằm ở chỗ khác: mọi quyết định kiến trúc đi qua một người, mọi PR chờ cùng một reviewer, hai Senior
đã ba tháng không nói chuyện trực tiếp sau một tranh luận về cách tách service, và người duy nhất
hiểu module thanh toán vừa nộp đơn xin nghỉ. Tổng năng lực cá nhân không giảm. Cái giảm là năng lực
của hệ thống chứa các cá nhân đó.

Một team không phải tập hợp các cá nhân giỏi. Nó là một **hệ thống** với ba thuộc tính mà không cá
nhân nào có: nó có **chi phí phối hợp** tăng phi tuyến theo số người; nó có **bộ nhớ** tồn tại ngoài
đầu từng người (hoặc không tồn tại, và khi đó mỗi lần có người rời đi là một lần mất dữ liệu); và nó
có **cơ chế phân bổ rủi ro** — ai được phép sai, sai ở đâu thì hệ thống chịu được, ai gánh hậu quả
khi sai. Ba thuộc tính này đều là thứ được thiết kế, dù người lead có ý thức thiết kế hay không.
Không thiết kế thì chúng vẫn hình thành, theo hướng mặc định, và hướng mặc định gần như luôn là: một
người thành điểm nghẽn, tri thức nằm trong đầu người đó, và rủi ro dồn vào người ít quyền nhất.

Level này nói về việc thiết kế và vận hành hệ thống đó. Bảy chủ đề dưới đây không phải bảy kỹ năng
mềm rời rạc. Chúng là bảy cơ chế điều khiển của cùng một hệ thống: Delegation phân bổ quyền ra quyết
định, Mentoring và Coaching nâng năng lực của các điểm quyết định đó, Motivation quyết định các điểm
đó có hoạt động khi không ai nhìn hay không, Conflict Resolution xử lý va chạm khi hai điểm quyết
định xung đột, Psychological Safety quyết định tín hiệu lỗi có đi lên được hay không, và Team
Dynamics là các thuộc tính nổi lên (emergent) của toàn bộ cấu hình. Trong chuỗi Business Goal →
People → Process → Technology → Execution, chương này nằm ở tầng People, nhưng nó là điều kiện để
tầng Process không biến thành nghi lễ và để tầng Feedback có tín hiệu thật.

Mục lục nội bộ:

1. [Delegation](#1-delegation)
2. [Mentoring](#2-mentoring)
3. [Coaching](#3-coaching)
4. [Motivation](#4-motivation)
5. [Conflict Resolution](#5-conflict-resolution)
6. [Psychological Safety](#6-psychological-safety)
7. [Team Dynamics và vòng đời team](#7-team-dynamics-và-vòng-đời-team)
8. [Tự kiểm tra](#tự-kiểm-tra)
9. [Liên kết chương khác](#liên-kết-chương-khác)

---

## 1. Delegation

### Problem Statement

Delegation là cơ chế phân bổ quyền ra quyết định trong team. Khi nó thiếu hoặc sai, hậu quả không
hiện ra dưới dạng "lead bận" — nó hiện ra dưới dạng các con số đếm được:

- **Số item đang chờ một người.** Mở Jira, đếm số ticket có assignee hoặc reviewer là lead. Ở team
  logistics đầu chương: 11 trong 23 ticket đang mở có lead ở một trong hai vị trí đó. Đây là định
  nghĩa vận hành của "lead là điểm nghẽn", không phải cảm giác.
- **Review latency.** Thời gian trung vị từ lúc PR mở tới lúc có review đầu tiên. Nếu con số này lớn
  hơn 8 giờ làm việc và 70% review đến từ một người, team đang chạy trên một server duy nhất.
- **Decision latency.** Đếm số message dạng "cái này chờ anh X xác nhận" trong Slack một tuần, và
  thời gian từ khi hỏi tới khi có câu trả lời. Mỗi giờ chờ ở đây là thời gian n người khác không làm
  được việc.
- **Bus factor theo module.** Với mỗi module quan trọng, đếm số người đã commit ít nhất 20 lần trong
  6 tháng. Nếu có module mà con số là 1, tổ chức đang có một single point of failure bằng người, và
  người đó không được đi nghỉ.
- **Tỉ lệ thời gian lead làm việc IC.** Nếu một Tech Lead có 8 người report kỹ thuật mà 70% thời gian
  vẫn viết feature code, hai việc đang bị bỏ: thiết kế hệ thống làm việc, và phát triển người.
- **Trần velocity.** Team không tăng throughput dù tuyển thêm người. Đây là dấu hiệu phần tuần tự
  (serialized) của quy trình đang chi phối, và phần tuần tự đó thường chính là một người.

Nếu không sửa, chuỗi hậu quả có thứ tự khá cố định: lead làm đêm → chất lượng quyết định giảm (mục
"quản lý năng lượng" trong `01-self-leadership.md` giải thích cơ chế) → lead trở thành người duy nhất
hiểu hệ thống → team mất động lực vì không được quyết gì → Senior giỏi nghỉ vì không thấy đường phát
triển → lead càng phải làm nhiều hơn. Đây là vòng phản hồi dương, và nó không tự dừng.

### First Principles

**Cơ chế một: băng thông của một người là hằng số, còn nhu cầu ra quyết định tăng theo số người và
số hệ thống.** Một người có khoảng 4–6 giờ tập trung sâu mỗi ngày, và số quyết định chất lượng cao
mỗi ngày là hữu hạn. Con số này không tăng theo title, không tăng theo kinh nghiệm, và gần như không
tăng theo nỗ lực — nỗ lực chỉ dịch chuyển nó sang thời gian ngoài giờ, nơi chất lượng quyết định
thấp hơn. Trong khi đó, số quyết định cần ra tăng theo số người trong team và số hệ thống họ đang
chạm vào. Đến một điểm nào đó, hai đường cắt nhau. Sau điểm đó, **mọi giải pháp bằng cách cố gắng
hơn đều thất bại về mặt số học**, không phải về mặt ý chí.

Suy ra: team không scale bằng cách nhân số bàn tay, mà bằng cách **nhân số điểm ra quyết định độc
lập**. Thêm một người mà không thêm một điểm quyết định thì bạn vừa thêm một hàng đợi vào cùng một
server — hệ thống chậm hơn, không nhanh hơn. Đây là lý do một team 5 người có 4 điểm quyết định
thường nhanh hơn team 12 người có 1 điểm quyết định.

**Cơ chế hai: định luật Amdahl áp cho tổ chức.** Nếu p là tỉ lệ công việc có thể làm song song và
(1−p) là phần buộc phải tuần tự qua một người, tốc độ tối đa bị chặn bởi 1/(1−p) bất kể thêm bao
nhiêu người. Một lead review 100% PR và quyết 100% thiết kế đang giữ (1−p) ≈ 0,3–0,4. Trần tăng tốc
của team đó là khoảng 2,5–3 lần, đạt được từ người thứ ba, và mọi người tuyển thêm sau đó chỉ tạo
thêm chi phí phối hợp. Delegation, đọc theo cách này, không phải hành vi đạo đức hay phong cách quản
lý — nó là **thao tác giảm (1−p)**.

**Cơ chế ba: Principal-Agent Problem và vì sao giao task không đủ.** Khi A giao việc cho B, có hai
khoảng cách: **thông tin bất đối xứng** (A biết ràng buộc kinh doanh, lịch sử hệ thống, cái đã thử
và thất bại; B không biết) và **hàm mục tiêu khác nhau** (A tối ưu cho outcome sáu tháng, B có thể
tối ưu cho việc bị đánh giá tốt trong sprint này, hoặc cho việc được học công nghệ mới). Giao task
chỉ giải quyết phần "làm gì". Nó không đóng hai khoảng cách trên. Kết quả điển hình: B làm đúng chữ,
sai ý. Cách đóng khoảng cách một là **giao context**: vì sao việc này tồn tại, nó phục vụ mục tiêu
kinh doanh nào, ràng buộc nào không được vi phạm, cái gì đã từng thử. Cách đóng khoảng cách hai là
**giao quyền cùng với tiêu chí thành công**: nếu B được quyết và được đo bằng outcome, hàm mục tiêu
của B dịch về gần hàm mục tiêu của A. Netflix gọi nguyên lý này là context over control, và phần
"context" mới là phần đắt.

**Cơ chế bốn: vì sao delegation thất bại — kỳ vọng kết quả giống như mình tự làm.** Đây là một lỗi
suy luận có tên: so sánh một mẫu (kết quả của B lần đầu) với giá trị trung bình của một phân phối đã
được huấn luyện nhiều năm (kết quả của A). A đã làm việc này 30 lần; 27 lần đầu của A cũng đạt 70%,
nhưng A không còn nhớ. Khi A dùng chính mình làm baseline, mọi delegation đều trông như thất bại, và
kết luận rút ra luôn là "tự làm nhanh hơn" — kết luận này đúng cho lần này và sai cho mười lần sau.

**Cơ chế năm: học đòi hỏi tín hiệu lỗi.** Năng lực hình thành qua vòng: hành động → hậu quả → điều
chỉnh. Nếu lead can thiệp trước khi hậu quả xuất hiện, vòng học bị cắt và người kia học được một
thứ duy nhất: chờ lead. Tức là **một lượng sai sót nhất định không phải chi phí của delegation, nó
là nội dung của delegation**. Việc của lead không phải triệt tiêu sai sót, mà là **chọn nơi cho phép
sai** sao cho hậu quả có thể phục hồi được (xem Reversible vs Irreversible trong
`04-decision-making.md`).

### Mental Model

**Theory of Constraints.** Hệ thống có throughput bằng throughput của ràng buộc. Nếu lead là ràng
buộc, mọi tối ưu ở nơi khác (tuyển thêm dev, mua CI nhanh hơn, dùng framework mới) đều không tăng
throughput. Ba câu hỏi vận hành: ràng buộc hiện tại là ai/cái gì? Đang có bao nhiêu công việc đứng
chờ nó? Loại công việc nào có thể chuyển ra khỏi nó mà không tăng rủi ro quá ngưỡng?

**Queueing theory.** Với một server, thời gian chờ tăng theo 1/(1−ρ) với ρ là utilization. Ở ρ = 0,9
thời gian chờ gấp 10 lần ở ρ = 0. Hệ quả phản trực giác nhưng quan trọng: **một lead nên chủ động
giữ utilization của mình dưới 70%**, không phải vì cần nghỉ, mà vì trên ngưỡng đó thời gian chờ của
n người khác bùng nổ. Một lead "kín lịch" là một lead đang làm cả team chậm lại.

**Delegation như một khoản đầu tư có thời gian hoàn vốn.** Gọi t_s là thời gian bạn tự làm, t_g là
thời gian giao + giải thích + review, r là chi phí rework kỳ vọng lần đầu. Lần đầu bạn lỗ:
t_g + r > t_s. Nhưng từ lần thứ k, người kia tự làm với chi phí giám sát gần bằng 0. Break-even xảy
ra khi số lần lặp lại đủ lớn. Suy ra tiêu chí chọn việc để giao: **ưu tiên việc tái diễn**. Một task
one-off phức tạp thường không đáng giao vì payback không bao giờ đến — trừ khi mục tiêu là phát triển
người, và khi đó bạn phải ghi nhận nó là chi phí đào tạo, không phải chi phí delivery.

### Practical Framework

**Bước 1 — Thang 7 mức delegation.** Sai lầm phổ biến nhất là coi delegation là công tắc bật/tắt.
Nó là một thang. Thang dưới đây là biến thể phổ biến của delegation poker (Management 3.0), diễn đạt
lại cho ngữ cảnh kỹ thuật:

| Mức | Tên | Ai quyết | Câu nói của lead |
|---|---|---|---|
| 1 | Chỉ dẫn | Lead quyết, giải thích tối thiểu | "Làm đúng theo cách này, bước 1... bước 2..." |
| 2 | Chỉ dẫn + lý do | Lead quyết, giải thích cơ chế | "Làm theo cách này, vì nếu làm cách kia thì X" |
| 3 | Hỏi ý rồi lead quyết | Lead quyết sau khi nghe | "Em xem phương án nào ổn, đưa anh 2 lựa chọn, anh chốt" |
| 4 | Cùng quyết | Đồng thuận hai người | "Mình cùng ngồi xuống chốt cái này" |
| 5 | Em quyết, anh tư vấn nếu em hỏi | Người nhận quyết | "Em chốt đi. Cần bàn thì gọi anh, không cần xin phép" |
| 6 | Em quyết, báo lại kết quả | Người nhận quyết | "Em chốt và làm. Xong thì cho anh biết đã chọn gì" |
| 7 | Toàn quyền | Người nhận quyết | "Phần này của em. Anh không cần biết chi tiết" |

Hai quy tắc dùng thang này. Thứ nhất, **mức phải được nói thành lời**. Phần lớn tai nạn delegation
xảy ra vì lead nghĩ mình đang ở mức 6 còn người nhận nghĩ mình đang ở mức 3 (nên chờ), hoặc ngược
lại: lead nghĩ mức 3 (chờ tôi chốt) còn người nhận nghĩ mức 6 (tôi chốt luôn) và đã merge lên
production. Thứ hai, **mức gắn với loại việc, không gắn với người**. Cùng một Senior có thể ở mức 7
với API design và mức 2 với việc thay đổi schema bảng giao dịch tiền.

**Bước 2 — Chọn mức bằng matrix năng lực × động lực.** Năng lực ở đây là năng lực *cho task cụ thể
này*, không phải title.

| | Động lực cao | Động lực thấp |
|---|---|---|
| **Năng lực cao** | Mức 6–7. Giao trọn, tránh can thiệp. Rủi ro chính: bạn giữ lại vì thói quen → người này nghỉ việc trong 6–9 tháng. | Mức 5–6 về hình thức, nhưng vấn đề thật không phải delegation. Đây là bài toán động lực (chủ đề 4) hoặc fit; giao thêm việc không chữa được. |
| **Năng lực thấp** | Mức 2–4. Đây là ô đầu tư: cho việc vừa quá tầm, kèm checkpoint dày. Mentoring (chủ đề 2) là công cụ chính. | Mức 1–2 và giới hạn scope. Nếu ô này kéo dài hơn một quý, đây là vấn đề performance, xem `08-hiring-va-phat-trien.md`. |

**Bước 3 — Delegation brief.** Giao việc bằng lời trong hành lang là nguồn của phần lớn rework. Với
việc có giá trị lớn hơn khoảng 3 ngày công, viết ra. Mẫu dưới đây vừa đủ để dán vào ticket:

```
DELEGATION BRIEF

1. Việc gì (một câu, dạng outcome không phải task)
   Ví dụ: "Hệ thống chịu được 3x traffic đợt sale 9/9 mà không tăng p99 quá 20%"
   (KHÔNG viết: "thêm cache cho API danh sách sản phẩm")

2. Vì sao bây giờ / phục vụ mục tiêu kinh doanh nào
   Ví dụ: "Đợt 9/9 chiếm 18% doanh thu quý. Năm ngoái sập 40 phút, mất ~X đơn."

3. Mức delegation: [1-7] = 6 (em quyết, báo lại kết quả)
   Cái em ĐƯỢC quyết: chọn cache strategy, chọn TTL, chọn có thêm read replica hay không
   Cái em KHÔNG được quyết một mình: thay đổi schema DB đơn hàng, thêm dependency mới
   vào critical path, bất kỳ thứ gì làm downtime > 0

4. Ràng buộc không được vi phạm
   - Không thay đổi contract API với app mobile (đang ở version cũ, không force update kịp)
   - Budget hạ tầng thêm <= 8 triệu/tháng
   - Code freeze từ 05/09

5. Định nghĩa "xong" (phải đo được, không dùng tính từ)
   - Load test 3x traffic peak năm ngoái, p99 <= 400ms, error rate < 0.1%
   - Có runbook rollback, đã thử rollback trên staging thành công
   - Có dashboard + alert cho 3 metric chính, đã review với on-call
   - ADR ghi lại phương án đã chọn và 2 phương án bị loại

6. Điểm kiểm tra (do người nhận chủ động báo, không phải lead đi hỏi)
   - Ngày 3: chốt phương án, 1 trang gửi anh đọc (đây là checkpoint để anh phản biện
     hướng, KHÔNG phải để anh phê duyệt)
   - Ngày 8: kết quả load test lần 1
   - Ngày 12: demo + runbook
   Nếu lệch tiến độ > 2 ngày hoặc phát hiện phải phá ràng buộc ở mục 4 -> nói ngay,
   không chờ tới checkpoint.

7. Cái gì đã từng thử và thất bại
   Q2 đã thử cache ở tầng nginx, invalidate không kịp, dữ liệu giá sai 12 phút.

8. Ai là người em cần nói chuyện
   Hà (mobile), Khoa (infra), Trang (PO — chốt được cái gì có thể bỏ nếu thiếu thời gian)
```

Mục 3 và 5 là hai mục hay bị bỏ nhất, và cũng là hai mục tạo ra phần lớn giá trị. Mục 6 có một chi
tiết cố ý: checkpoint là nghĩa vụ báo của người nhận, không phải quyền đi hỏi của lead. Đảo chiều
này giữ được ownership.

**Bước 4 — Điều gì làm ở checkpoint.** Ở checkpoint, lead chỉ được làm ba việc: (a) hỏi để hiểu suy
luận, (b) nêu rủi ro mà người kia có thể chưa thấy, (c) nếu và chỉ nếu ràng buộc ở mục 4 đang bị vi
phạm hoặc rủi ro là không phục hồi được, thì nâng mức kiểm soát — và **nói rõ rằng mình đang nâng
mức, cùng lý do**. Việc âm thầm chuyển từ mức 6 về mức 2 là cách nhanh nhất để phá trust.

**Bước 5 — Đóng vòng.** Sau khi xong, làm hai việc: ghi lại mức delegation cho loại việc này lần sau
(thường là tăng một mức nếu kết quả đạt), và feedback cụ thể theo mô hình trong `02-communication.md`.
Nếu bạn không tăng mức sau khi người ta làm tốt, bạn đã không delegation — bạn chỉ nhờ việc.

### Trade-off

| Trục | Nghiêng về Autonomy khi | Nghiêng về Control khi |
|---|---|---|
| **Autonomy vs Control** | Quyết định reversible; hậu quả sai giới hạn trong một service; người nhận có năng lực gần đủ; bạn cần scale số điểm quyết định | Quyết định irreversible (migration dữ liệu tiền, xoá dữ liệu, thay đổi contract công khai); ràng buộc compliance/audit; đang trong incident; sai một lần là mất khách hàng |
| **Tốc độ ngắn hạn vs năng lực dài hạn** | Việc tái diễn, còn ít nhất 2 quý để thu hồi đầu tư; team đang cần thêm điểm quyết định | Deadline cứng do hợp đồng, không có slack; task one-off không lặp lại; bạn không có băng thông để review tử tế |
| **Consistency vs Ownership** | Đa dạng cách làm chấp nhận được; muốn người ta thực sự sở hữu kết quả | Hệ thống cần tính nhất quán cao (naming, error contract, security pattern); nhiều team dùng chung |
| **Giao rộng vs giao hẹp** | Người nhận cần học ra quyết định, không chỉ học code | Rủi ro tập trung; cần giới hạn blast radius trong lần đầu |

Câu hỏi phân bổ mất mát: **ai chịu phần mất?** Khi bạn chọn Control để an toàn, phần mất do người
nhận chịu (không được học, không có cơ hội chứng minh, đến kỳ Performance Review không có bằng chứng
để promote) và tổ chức chịu (bus factor không tăng). Phần được thì bạn hưởng (sprint này không có sự
cố). Bất đối xứng này là lý do Control là lựa chọn mặc định của phần lớn lead mới, và là lý do nó
sai một cách có hệ thống.

### Real-world Scenarios

**Tình huống A — Giao module quan trọng cho một Mid-level, kết quả đạt 70%.**

Bối cảnh: Công ty fintech Việt, khoảng 45 engineer. Team Payment 7 người. Cần xây module đối soát
(reconciliation) với một cổng thanh toán đối tác: mỗi ngày pull file giao dịch, so với dữ liệu nội
bộ, sinh báo cáo lệch cho bộ phận vận hành. Không phải critical path realtime, nhưng sai thì bộ phận
kế toán mất nhiều giờ dò tay và có rủi ro báo cáo sai. Tuấn — Mid-level 3 năm, mạnh về code, chưa
từng tự thiết kế một module đầu-đến-cuối. Minh — Tech Lead — biết mình có thể tự làm trong 5 ngày.

Minh chọn giao, ở mức 5, với brief như mẫu trên, checkpoint ngày 2 và ngày 6, deadline 12 ngày. Lý
do: đối soát là loại việc sẽ tái diễn (còn 3 đối tác nữa sẽ tích hợp trong 2 quý tới), nên payback
đến; và blast radius giới hạn — sai thì báo cáo sai, không mất tiền thật, có thể chạy lại.

Ngày 12, kết quả: luồng chính chạy đúng. Nhưng ba thứ thiếu — không xử lý trường hợp file đối tác
đến trễ hoặc đến hai lần (idempotency), không có alert khi job fail (chỉ log), và phần so khớp viết
thành một hàm 300 dòng không test được. Đạt khoảng 70%.

Đây là điểm mà phần lớn lead làm sai, và làm sai theo hai hướng đối xứng. Hướng sai thứ nhất: Minh
tự viết lại trong hai ngày, không nói gì, để kịp deadline. Hậu quả: Tuấn không biết mình đã thiếu
gì, học được rằng "cứ làm phần chính, phần khó anh Minh sẽ lo", và lần sau sẽ lặp lại chính xác cùng
ba thiếu sót. Đồng thời Tuấn cảm nhận được sự can thiệp và mất động lực — cảm giác "làm gì cũng bị
sửa" mạnh hơn cảm giác được giúp. Hướng sai thứ hai: Minh cho qua vì "cũng chạy được", nợ kỹ thuật
vào production, ba tháng sau job fail âm thầm 4 ngày và kế toán phát hiện lệch 200 giao dịch. Lúc đó
Minh nói "tại Tuấn làm không kỹ" — nhưng accountability của kết quả này thuộc Minh, không thuộc Tuấn
(ranh giới Accountability vs Responsibility ở `00-nen-tang-leadership.md`).

Cách xử lý đúng bắt đầu bằng việc **phân tách 70% đó thành ba loại khác nhau**, vì ba loại cần ba
cách xử lý:

- **Loại 1 — cái Tuấn không thể biết vì thiếu context.** Việc file đối tác đến hai lần là bài học
  vận hành mà Minh đã trải qua với đối tác khác năm trước và không viết vào brief. Đây là **lỗi của
  Minh**, không phải của Tuấn. Nói thẳng điều đó ra là hành động tạo Psychological Safety mạnh nhất
  trong cả tình huống.
- **Loại 2 — cái Tuấn phải biết ở level của mình.** Alert khi job fail là chuẩn tối thiểu của một
  scheduled job trong hệ thống tiền. Đây là gap năng lực thật, cần feedback rõ và cần đưa vào chuẩn
  chung (definition of done cho mọi job) để không phụ thuộc vào việc nhớ.
- **Loại 3 — cái thuộc phong cách, không phải chất lượng.** Hàm 300 dòng: cần sửa, nhưng đây là vấn
  đề testability chứ không phải thẩm mỹ. Đặt tiêu chí đo được ("mỗi rule đối soát có unit test riêng
  chạy được không cần DB") thay vì nói "code chưa clean".

Script — **phiên bản nói sai**:

> **Minh:** "Tuấn ơi, anh xem module đối soát rồi. Nói thật là chưa được. Em làm thiếu nhiều thứ
> quá, không có alert, không handle duplicate, hàm thì dài 300 dòng không ai đọc được. Cái này lên
> production là chết. Thôi để anh sửa cho nhanh, deadline tuần này rồi. Lần sau em chú ý làm kỹ hơn
> nhé, cẩn thận vào."

Vấn đề của phiên bản này, theo thứ tự nghiêm trọng: (1) "chưa được" là đánh giá tổng thể vào con
người, không phải vào từng gap, nên Tuấn phải phòng vệ chứ không thể học; (2) gộp cả ba loại gap
thành một đống, trong đó có loại mà lỗi thuộc Minh; (3) "để anh sửa" lấy lại việc, chấm dứt vòng
học và phát tín hiệu không tin tưởng; (4) "chú ý làm kỹ hơn" là chỉ dẫn không thực thi được — Tuấn
không biết cụ thể phải thêm hành vi nào vào lần sau.

Script — **phiên bản nói đúng**:

> **Minh:** "Anh đọc hết module đối soát rồi. Trước hết: phần so khớp core em làm đúng, cả case
> partial settlement mà anh không nói trong brief em cũng xử lý — cái đó anh không nghĩ tới lúc
> giao. Giờ anh muốn đi qua ba nhóm việc còn lại, và anh chia làm ba nhóm vì tính chất khác nhau.
>
> Nhóm một là lỗi brief của anh. Anh không nói với em chuyện đối tác từng gửi trùng file hai lần —
> hồi tích hợp cổng cũ năm ngoái nó xảy ra và bọn anh double-count 300 giao dịch. Em không có cách
> nào biết. Cái này anh nhận, và anh sẽ thêm vào tài liệu tích hợp đối tác để lần sau không mất.
> Em ước tính thêm idempotency key theo (partner, file_date, checksum) mất bao lâu?
>
> **Tuấn:** "Chắc một ngày, tại em phải thêm bảng lưu trạng thái file đã xử lý."
>
> **Minh:** "Được. Nhóm hai thì khác: job này không có alert khi fail, chỉ ghi log. Cái này anh coi
> là chuẩn tối thiểu ở level của em, với hệ thống chạm vào tiền. Em nghĩ nếu job fail lúc 2 giờ sáng
> thứ Bảy thì ai biết, và biết sau bao lâu?"
>
> **Tuấn:** "...không ai biết. Tới thứ Hai kế toán mở báo cáo mới thấy thiếu."
>
> **Minh:** "Đúng. Và lúc đó đã lệch hai ngày dữ liệu. Anh không nghĩ em không biết cần alert — anh
> nghĩ nó không nằm trong checklist của em. Nên hai việc: em thêm alert, và anh sẽ đưa 'scheduled
> job phải có alert on failure + dashboard last_success_at' vào definition of done chung, để nó
> không phụ thuộc vào việc ai nhớ.
>
> Nhóm ba là hàm so khớp 300 dòng. Anh không đòi 'clean code' — tiêu chí cụ thể là mỗi rule đối soát
> phải có unit test chạy được không cần DB. Hiện tại em test được không?
>
> **Tuấn:** "Không. Em phải seed cả DB mới chạy được một case."
>
> **Minh:** "Đó là chi phí thật: mỗi lần đối tác đổi format, em mất nửa ngày để verify. Còn ba đối
> tác nữa sắp tích hợp. Nên tách phần rule ra thành function thuần là việc trả lại được. Tổng ba
> nhóm em cần bao lâu?"
>
> **Tuấn:** "Ba, bốn ngày."
>
> **Minh:** "Anh xin thêm 4 ngày với Trang, phần báo cáo nâng cao đẩy sang sprint sau. Em làm tiếp,
> vẫn là module của em. Và một câu để em tự trả lời, không cần trả lời anh hôm nay: nếu bây giờ giao
> lại từ đầu, em sẽ hỏi anh thêm câu gì ở ngày đầu tiên?"

Kết quả 5 tháng sau: Tuấn tích hợp hai đối tác tiếp ở mức 6, không cần checkpoint giữa, và tự viết
được checklist tích hợp mà team dùng lại. Chi phí của cách xử lý đúng: 4 ngày trễ và một lần xin
scope với PO. Nếu Minh chọn tự sửa, chi phí ngắn hạn thấp hơn 2 ngày, và chi phí dài hạn là Minh
vẫn là người duy nhất làm được đối soát ở quý sau.

**Ba góc nhìn về cùng sự việc.** Đây là chỗ nhiều tổ chức đọc sai, vì mỗi tầng nhìn thấy một hàm
mục tiêu khác nhau và cả ba đều có lý trong hệ quy chiếu của mình:

- **Nhìn từ IC (Tuấn).** "Em được giao một module lớn nhất từ khi vào công ty. Em làm xong luồng
  chính đúng hạn và em nghĩ nó chạy được. Nếu anh Minh nói 'chưa được' và lấy về sửa, tín hiệu em
  nhận được là em không đủ tin tưởng — và lần sau em sẽ hỏi anh trước mọi quyết định để an toàn, kể
  cả những thứ em tự quyết được. Cái em cần không phải lời động viên, mà là biết chính xác chuẩn ở
  level của em là gì, để em không phải đoán."
- **Nhìn từ Tech Lead (Minh).** "Tôi đang tối ưu hai thứ xung đột nhau trong cùng một sprint:
  delivery của module này, và năng lực của team ở quý sau. Nếu tôi chỉ tối ưu cái đầu, tôi vẫn là
  điểm nghẽn khi tích hợp ba đối tác nữa và tôi biết mình không có băng thông cho việc đó. 4 ngày
  trễ là giá tôi trả để mua một điểm quyết định mới. Việc khó nhất không phải nói ra ba nhóm gap —
  mà là nhận nhóm một là lỗi của tôi, vì nếu tôi không nhận, tôi sẽ dạy cả team rằng brief thiếu là
  chuyện bình thường và người thực thi luôn là người sai."
- **Nhìn từ Manager (EM/Trang phía Product).** "Tôi thấy một module trễ 4 ngày, và tôi cần biết đây
  là trễ do năng lực, do estimate sai, hay do đầu tư có chủ đích. Ba thứ đó cần ba phản ứng khác
  nhau: nếu là năng lực thì tôi phải xem lại việc phân bổ; nếu là estimate sai thì quy trình
  Estimation có vấn đề; nếu là đầu tư thì tôi cần Minh nói ra để tôi tính vào capacity và không kết
  luận sai về Tuấn ở kỳ review. Điều tệ nhất với tôi không phải trễ 4 ngày — mà là trễ mà tôi biết
  muộn, hoặc trễ mà tôi không biết nó thuộc loại nào."

Điểm rút ra: cùng một sự kiện (module đạt 70%), IC đọc là **tín hiệu về mức tin tưởng**, Tech Lead
đọc là **bài toán phân bổ giữa delivery ngắn hạn và capability dài hạn**, Manager đọc là **thông tin
để phân loại nguyên nhân và cập nhật capacity**. Một lead giỏi phải phát tín hiệu đủ cho cả ba tầng
đọc đúng: nói rõ với Tuấn đây là gì, nói rõ với Trang đây là loại trễ nào.

**Tình huống B — Delegation ngược trong một ODC.** Team 12 người làm cho khách hàng Nhật, Tech Lead
là Linh. Mỗi ngày Linh nhận khoảng 15–20 câu hỏi dạng "chị ơi cái này em làm sao?", "chị xem giúp
em cái này đúng chưa?", "khách hỏi cái này em trả lời thế nào?". Linh trả lời hết, nhanh, chính xác
— và làm việc đến 9 giờ tối. Đây là **delegation ngược** (reverse delegation): quyết định được đẩy
trở lại về lead, và Linh vô tình huấn luyện hành vi đó bằng cách trả lời ngay và trả lời đúng.

Cơ chế: với một engineer, hỏi Linh có chi phí thấp (30 giây) và rủi ro bằng 0 (nếu sai thì Linh đã
xác nhận). Tự quyết có chi phí cao (30 phút suy nghĩ) và rủi ro cá nhân. Hành vi tối ưu cá nhân là
hỏi. Hệ thống thưởng cho việc hỏi, nên hệ thống nhận được việc hỏi. Đây không phải vấn đề thái độ,
đây là vấn đề thiết kế incentive.

Can thiệp của Linh, theo thứ tự hiệu lực tăng dần: (1) đổi câu trả lời từ đáp án sang câu hỏi — "em
đang nghiêng phương án nào và vì sao?"; (2) yêu cầu định dạng khi hỏi — "muốn hỏi chị thì gửi kèm:
em đã thử gì, em nghĩ có mấy lựa chọn, em nghiêng cái nào" (việc này đẩy chi phí của việc hỏi lên và
đồng thời làm người hỏi suy nghĩ trước); (3) phân vùng quyền quyết định thành văn bản — danh sách
loại quyết định mà Senior tự quyết không cần hỏi; (4) chỉ định người trả lời không phải Linh cho
từng vùng (ai là người hỏi về DB, ai về deploy). Sau 6 tuần, số câu hỏi tới Linh giảm khoảng một
nửa; điều quan trọng hơn là 3 người bắt đầu trở thành điểm quyết định cho vùng của mình.

### Best Practices

- **Giao outcome, không giao task, khi mức delegation ≥ 5.** Lý do: nếu bạn mô tả task, bạn đã ra
  quyết định thiết kế và người nhận chỉ còn là người gõ. Ownership không thể tồn tại nếu không gian
  quyết định bằng 0.
- **Nói mức delegation thành lời, mỗi lần.** Lý do: cơ chế illusion of transparency
  (`02-communication.md`) đảm bảo rằng cái bạn tưởng đã rõ thì chưa rõ. Một câu "cái này em chốt,
  không cần hỏi anh" đắt 5 giây và tiết kiệm hàng ngày chờ đợi.
- **Định nghĩa "xong" bằng đại lượng đo được trước khi bắt đầu.** Lý do: nếu tiêu chí xuất hiện sau
  khi có kết quả, mọi feedback đều bị đọc là dịch chuyển cột gôn, kể cả khi nó đúng.
- **Đặt checkpoint theo mốc rủi ro, không theo lịch đều.** Checkpoint có giá trị nhất ở ngay sau khi
  phương án được chọn (còn rẻ để đổi hướng) và ngay trước điểm không thể quay lại. Checkpoint hàng
  ngày biến delegation thành micromanagement có lịch.
- **Giao cả việc bạn thích làm.** Lý do: nếu bạn chỉ giao việc nhàm, bạn đang dạy team rằng
  delegation là cơ chế xả rác. Ngoài ra việc thú vị mới là việc phát triển được năng lực.
- **Chịu hậu quả công khai của việc mình đã giao.** Khi kết quả xấu đi ra ngoài team, người trả lời
  stakeholder là bạn, không phải người bạn giao. Đây là điều kiện để lần sau còn ai dám nhận việc.
- **Tăng mức sau mỗi lần thành công, và nói rõ là đang tăng.** Lý do: đó là phần "lương" phi tiền tệ
  của người làm tốt, và là bằng chứng cho hồ sơ promotion sau này.

### Anti-patterns

- **"Tự làm nhanh hơn".** Cơ chế phá hoại: đúng ở phạm vi một task, sai ở phạm vi một quý. Nó tạo
  vòng phản hồi dương — càng tự làm càng là người duy nhất làm được, càng thấy tự làm nhanh hơn.
  Dấu hiệu sớm: lead có commit trong hầu hết module; lead nói câu "để anh làm cho nhanh" nhiều hơn
  một lần một tuần.
- **Seagull delegation.** Giao xong biến mất, đến gần deadline bay vào, phê phán, đổi hướng, rồi bay
  đi. Cơ chế: người nhận đã đi 80% đường theo một hướng, chi phí đổi hướng ở thời điểm đó là cao
  nhất trong toàn bộ vòng đời task. Dấu hiệu sớm: không có checkpoint nào ở 1/3 đầu.
- **Giao Responsibility mà không giao Authority.** Người nhận chịu trách nhiệm cho kết quả nhưng
  không được quyết bất cứ thứ gì ảnh hưởng kết quả (không được đổi scope, không được nói với PO,
  không được chọn công cụ). Cơ chế: tạo ra trạng thái bất lực có trách nhiệm — nguồn burnout mạnh
  nhất trong công việc kỹ thuật. Dấu hiệu sớm: câu "em không có quyền quyết cái đó nhưng em bị hỏi
  về nó".
- **Delegation ngược được thưởng.** Xem tình huống B. Dấu hiệu sớm: lead trả lời câu hỏi kỹ thuật
  nhanh hơn 2 phút và tỉ lệ câu hỏi có dạng "em nên làm gì" cao hơn dạng "em đang nghĩ X, anh thấy
  sao".
- **Delegation cho người rảnh nhất thay vì người phù hợp nhất.** Cơ chế: tối ưu utilization thay vì
  tối ưu throughput và learning. Kết quả là việc quan trọng vào tay người có ít context nhất, còn
  người phù hợp thì tiếp tục quá tải.
- **Micromanagement đội lốt "review kỹ".** Cơ chế: comment vào từng dòng về phong cách trong khi
  không phản biện thiết kế. Nó tiêu thụ trust cao và tạo ra giá trị thấp. Dấu hiệu sớm: số comment
  về đặt tên biến nhiều hơn số comment về failure mode.

### Khi nào KHÔNG nên áp dụng

- **Đang trong incident production.** Trong incident, cơ chế cần là command-and-control với một
  Incident Commander, vì chi phí của mỗi phút cao và việc tranh luận phương án là xa xỉ. Delegation
  ở đây có nghĩa hẹp: giao rõ ai làm gì trong 10 phút tới, không phải giao quyền quyết hướng xử lý.
  Chi tiết ở `06-incident-va-metrics.md`.
- **Quyết định irreversible với blast radius lớn.** Migration dữ liệu tài chính, xoá dữ liệu người
  dùng, đổi contract API công khai, thay đổi ảnh hưởng compliance. Ở đây điều đúng là mức 3–4 và
  bốn con mắt, không phải mức 6. Lưu ý: hãy giao *phần thiết kế và chuẩn bị* ở mức cao, chỉ giữ mức
  thấp ở *điểm bấm nút*.
- **Khi bạn không có băng thông để làm phần của mình.** Delegation không phải hành động miễn phí; nó
  đòi thời gian viết brief, thời gian ở checkpoint, thời gian review. Giao việc rồi không xuất hiện
  tệ hơn tự làm, vì nó tiêu thụ thời gian của người khác và kết thúc bằng thất bại mà họ mang tên.
  Nếu tuần này bạn không có 3 giờ cho việc này, hãy giao một task nhỏ hơn.
- **Khi task là one-off và không có mục tiêu phát triển người.** Payback không đến. Trường hợp này
  hoặc tự làm, hoặc giao ở mức 1–2 với hướng dẫn cụ thể, và không kỳ vọng năng lực được tạo ra.
- **Khi người nhận đang ở ô "năng lực thấp, động lực thấp" và nguyên nhân chưa được chẩn đoán.** Giao
  việc khó cho người đang mất động lực không phải một cú hích, nó là một cách tạo thất bại có tài
  liệu. Chẩn đoán trước (chủ đề 4), rồi mới giao.
- **Khi ràng buộc hợp đồng quy định người thực hiện.** Trong một số dự án ODC và dự án khu vực công,
  hợp đồng ràng buộc ai được chạm vào cái gì (clearance, chứng chỉ, danh sách nhân sự đã duyệt).
  Delegation vượt ràng buộc này là rủi ro pháp lý, không phải sự linh hoạt.

---

## 2. Mentoring

### Problem Statement

Một công ty product Việt 40 engineer, bảng lương ghi 11 Senior. Đếm lại theo một tiêu chí khác — số
người đã tự thiết kế và đưa vào production một module có state, tự chịu on-call cho nó, tự bảo vệ
được thiết kế trước phản biện — con số là 3. Tám người còn lại là Senior theo số năm và theo mức
lương thị trường, không theo năng lực ra quyết định. Đây không phải chuyện của một công ty; nó là
hệ quả trực tiếp của một thị trường job hopping nhanh, title inflation, và gần như không có cơ chế
chuyển giao tri thức nào ngoài "làm nhiều rồi tự khắc biết".

Thiếu Mentoring không hiện ra dưới dạng "team chưa giỏi". Nó hiện ra dưới dạng:

- **Time-to-independence theo module.** Số tuần từ khi một người nhận việc trong module X đến khi
  merge được thay đổi không tầm thường mà không cần ai cầm tay. Nếu con số này giống nhau ở người
  vào 3 tháng và người vào 18 tháng, tri thức không đang chảy.
- **Số lần cùng một lỗi thiết kế xuất hiện lại.** Đếm trong 6 tháng: bao nhiêu lần một PR bị chặn vì
  đúng cái lý do đã chặn PR của người khác trước đó. Nếu lớn hơn 2, tri thức đang được *sửa* mỗi lần
  chứ không được *chuyển giao* một lần.
- **Tỉ lệ quyết định kỹ thuật do người không phải lead ra.** Đây cũng là chỉ số của Delegation, và đó
  không phải trùng lặp: Delegation mở cửa, Mentoring làm cho người đi qua cửa đó không rơi.
- **Số người có thể nhận on-call cho mỗi service.** Nếu là 1, mọi thứ khác chỉ là chi tiết.
- **Câu hỏi lặp lại trong Slack.** Cùng một câu hỏi được hỏi lần thứ ba bởi ba người khác nhau là
  bằng chứng rằng câu trả lời lần một đã giải quyết task, không giải quyết mô hình tư duy.
- **Lý do nghỉ việc trong exit interview.** "Em không thấy mình học được gì thêm ở đây" là lý do phổ
  biến hơn lương ở nhóm Mid có năng lực — và nó xuất hiện trước khi lương xuất hiện.

Hậu quả dài hạn có thứ tự: không có người kế cận → lead không dám nghỉ và không dám lên vai trò
rộng hơn → tổ chức không mở được team thứ hai vì không có ai lead được → dự án mới bị từ chối hoặc
nhận rồi làm hỏng. Mentoring không phải phúc lợi cho cá nhân. Nó là điều kiện để tổ chức có thêm
đơn vị vận hành độc lập.

### First Principles

**Cơ chế một: phần đắt nhất của tri thức kỹ thuật là tacit, và tacit không đi qua tài liệu.** Phân
biệt của Polanyi: explicit knowledge là cái viết ra được (cú pháp, cấu hình, quy trình deploy);
tacit knowledge là cái người ta biết nhưng không phát biểu thành lời được — nhận ra mùi của một
thiết kế sắp gãy, biết khi nào một con số bất thường đáng lo và khi nào không, biết câu hỏi nào cần
hỏi PO trước khi bắt đầu. Đây là lý do một wiki 200 trang vẫn không làm người mới tự chủ được: wiki
chứa toàn bộ explicit và gần như không chứa tacit. Tacit chỉ chuyển giao qua **đồng thực hiện** và
qua việc **nghe người có kinh nghiệm nghĩ thành tiếng**. Suy ra một hệ quả vận hành rất cụ thể: giá
trị của một buổi pairing không nằm ở đoạn code viết ra, mà nằm ở những câu mentor nói ra trong khi
viết — "chỗ này tôi lo vì...", "tôi bỏ qua case kia vì...".

**Cơ chế hai: năng lực là hàm của số mẫu đã gặp nhân với chất lượng phản hồi, không phải hàm của số
năm.** Mười năm làm cùng một loại task với feedback bằng 0 tạo ra năng lực của một năm lặp lại mười
lần. Deliberate practice cần ba điều kiện: task ở rìa năng lực, phản hồi nhanh và cụ thể, có cơ hội
làm lại. Một tổ chức có thể trả đủ lương Senior mà không cung cấp bất cứ điều kiện nào trong ba điều
kiện đó — và khi đó nó đang mua thời gian, không mua sự phát triển.

**Cơ chế ba: mentoring nén thời gian học bằng cách cho người khác dùng bản đồ sai lầm của mình.**
Cái đắt nhất trong 15 năm kinh nghiệm không phải danh mục thứ mình biết làm — phần đó có trên
Internet. Nó là **danh mục các cách thất bại**: cache invalidation sai làm giá hiển thị lệch 12
phút, retry không idempotent nhân đôi giao dịch, migration chạy 40 phút trong giờ cao điểm vì không
ai thử trên bản sao dữ liệu thật. Học các bài đó bằng trải nghiệm cá nhân là học **tuần tự**: mỗi
bài học tốn một sự cố và một khoảng thời gian không nén được. Mentoring cho phép học **song song**:
mentee mượn danh mục thất bại của mentor và trả bằng thời gian nghe, không bằng sự cố. Đây là toàn
bộ lý do mentoring có ROI dương ở cấp tổ chức, dù nó âm ở cấp cá nhân người mentor trong ngắn hạn.

**Cơ chế bốn: Zone of Proximal Development.** Vygotsky mô tả ba vùng: cái người ta làm được một
mình, cái làm được khi có hỗ trợ, cái chưa làm được kể cả có hỗ trợ. Học chỉ xảy ra ở vùng giữa.
Task dưới vùng đó tạo ra sự nhàm chán và không tạo năng lực. Task trên vùng đó tạo ra thất bại,
và thất bại quá tầm không dạy được gì vì mentee không có đủ mô hình để giải thích tại sao mình
thất bại — nó chỉ hạ self-efficacy. Việc chọn bài tập, vì vậy, là hành động kỹ thuật khó nhất của
mentoring, khó hơn việc giảng.

**Cơ chế năm: mentoring bị under-invest một cách hệ thống, vì lý do kinh tế chứ không vì lý do đạo
đức.** Mentor trả chi phí ngay (thời gian, delivery chậm) và thu lợi ích muộn, không chắc chắn (nếu
mentee nghỉ sau 8 tháng thì lợi ích chuyển sang công ty khác). Với mọi cá nhân duy lý, mức đầu tư
tối ưu cá nhân thấp hơn mức tối ưu cho tổ chức. Sửa việc này bằng lời kêu gọi ("hãy chia sẻ kiến
thức") không có hiệu lực. Sửa được bằng cách đưa nó vào hàm mục tiêu của lead: một Tech Lead được
đánh giá bằng số người có thể thay mình, không phải bằng số dòng code mình viết.

### Mental Model

**Mô hình học nghề bốn giai đoạn (apprenticeship).** Bốn trạng thái, và điều quan trọng là mỗi lần
chuyển trạng thái phải **rút bớt giàn giáo**, không phải thêm:

```
1. Tôi làm — em xem      (shadowing; mentor nói ra suy luận, không chỉ hành động)
2. Cùng làm              (pairing; mentee gõ, mentor hỏi "tại sao chọn cái này")
3. Em làm — tôi xem      (reverse pairing; mentor im lặng, chỉ can khi rủi ro không phục hồi)
4. Em làm — báo lại      (autonomous; mentor chỉ đọc kết quả)
```

Ánh xạ sang thang 7 mức delegation ở chủ đề 1: bốn giai đoạn này tương ứng mức 1–2, 3–4, 5, 6–7.
Mentoring và Delegation là hai mặt của cùng một thao tác — Delegation là phía quyền, Mentoring là
phía năng lực. Nâng quyền mà không nâng năng lực tạo ra thất bại; nâng năng lực mà không nâng quyền
tạo ra người giỏi bỏ đi.

**Độ trễ của vòng phản hồi là biến điều khiển.** Tốc độ học không tỉ lệ với số giờ mentor giảng, nó
tỉ lệ nghịch với độ trễ giữa hành động của mentee và tín hiệu đúng/sai. Một mentor review trong 2
giờ tạo ra tốc độ học cao hơn nhiều lần một mentor giảng 2 giờ mỗi tuần nhưng review sau 3 ngày —
vì sau 3 ngày, mentee đã mất ngữ cảnh trong đầu và đã xây thêm ba lớp lên trên quyết định sai.

**Mentoring là thao tác di chuyển ràng buộc.** Nếu ràng buộc của hệ thống là "chỉ một người hiểu
module thanh toán", mọi cải thiện khác không tăng throughput. Mentoring là cách duy nhất di chuyển
ràng buộc đó mà không cần tuyển người — và tuyển người thường không giải quyết được, vì người mới
cũng không biết module đó.

### Practical Framework

Nguyên tắc trung tâm: **mentoring không phải một quan hệ, nó là một chương trình có mục tiêu, có
thời hạn, có tiêu chí kết thúc.** "Anh sẽ mentor em" mà không có ba thứ đó sẽ suy biến thành việc
trả lời câu hỏi khi được hỏi — hữu ích, nhưng không nén được thời gian học.

**Bước 1 — Chuyển mục tiêu mơ hồ thành gap quan sát được.** "Cần lên Senior" không dùng được. Viết
lại thành danh sách năng lực thiếu, mỗi cái có bằng chứng:

| Gap mơ hồ | Gap dùng được |
|---|---|
| "Cần kỹ năng thiết kế" | "Chưa từng thiết kế module có state và có xử lý bất đồng bộ. Bằng chứng: 3 module gần nhất đều là CRUD đồng bộ" |
| "Cần chủ động hơn" | "Chưa từng nêu rủi ro trong buổi review thiết kế của người khác. Bằng chứng: 0 comment về failure mode trong 6 tháng" |
| "Cần hiểu production" | "Không đọc được dashboard để chẩn đoán. Bằng chứng: trong 2 incident gần nhất, chờ người khác nói nguyên nhân" |
| "Cần communication tốt hơn" | "RFC viết ra chưa bảo vệ được: 2 RFC gần nhất bị hỏi 'còn phương án nào khác' và không trả lời được" |

**Bước 2 — Mentoring contract.** Viết ra, hai người cùng đọc, 8–12 tuần. Dài hơn 12 tuần thì mục
tiêu quá to hoặc không ai còn nhớ nó tồn tại.

```
MENTORING CONTRACT

Mentee: Tuấn (Mid BE, 3 năm)          Mentor: Minh (Tech Lead)
Chu kỳ: 10 tuần, 15/03 -> 24/05       Nhịp: 45 phút/tuần + review trong 4 giờ làm việc

1. MỤC TIÊU (tối đa 3, phát biểu bằng năng lực, không bằng task)
   G1. Tự thiết kế được một module bất đồng bộ có state và bảo vệ được thiết kế đó
   G2. Tự chẩn đoán được sự cố của module mình sở hữu bằng metric/log, không cần hỏi
   G3. Nêu được rủi ro trong thiết kế của người khác (không chỉ trong thiết kế của mình)

2. BÀI TẬP VỪA QUÁ TẦM (real work, không phải bài tập giả)
   Job đối soát 3 đối tác còn lại + webhook retry có idempotency
   Mức delegation: 5 (Tuấn chốt, hỏi khi cần) — xem chủ đề 1
   Blast radius: sai thì báo cáo sai, chạy lại được, không mất tiền thật

3. BẰNG CHỨNG ĐẠT (đo được, không dùng tính từ)
   G1: RFC được Quân và Khoa phản biện, trả lời được >= 80% câu hỏi không cần Minh đỡ
   G2: 1 lần chủ động phát hiện sự cố qua alert trước khi vận hành báo; viết postmortem
   G3: >= 3 comment về failure mode trong RFC/PR của người khác, mỗi comment nêu được
       điều kiện xảy ra chứ không chỉ nêu lo ngại

4. CHẾ ĐỘ TRUYỀN (theo tuần)
   T1-2: shadowing — Tuấn ngồi cùng Minh khi Minh thiết kế module notification
   T3-5: pairing 2 buổi/tuần x 90 phút, Tuấn gõ
   T6-8: reverse — Tuấn dẫn, Minh chỉ hỏi, không sửa trực tiếp
   T9-10: tự chủ, Minh chỉ đọc RFC và kết quả

5. CÁI MENTOR CAM KẾT
   - Không lấy lại việc, kể cả khi thấy chậm (chỉ can nếu vi phạm ràng buộc ở brief)
   - Review trong 4 giờ làm việc; nếu không kịp thì nói trước, không để im lặng
   - Nói ra suy luận của mình, kể cả phần chưa chắc

6. ĐIỀU KIỆN DỪNG SỚM
   - Tuấn không còn muốn tiếp tục (nói ra, không cần lý do)
   - Sau 4 tuần không có bằng chứng tiến bộ nào -> xem lại giả thuyết: đây là gap
     kỹ năng, hay là vấn đề động lực / fit (chủ đề 4)
```

**Bước 3 — Chọn bài tập vừa quá tầm, bốn tiêu chí.** (a) Là việc thật, có người dùng thật — bài tập
giả không tạo được áp lực nhận thức cần thiết, và mentee biết nó là giả. (b) Khoảng 60–70% dựa trên
cái mentee đã làm được, 30–40% là mới. (c) Blast radius phục hồi được: sai thì tốn thời gian, không
tốn tiền hoặc uy tín không lấy lại được. (d) Có deadline nhưng có slack ít nhất 30% — nếu không có
slack, mentee sẽ chọn cách an toàn nhất và không học gì.

**Bước 4 — Chế độ truyền, và chi tiết dễ bỏ nhất.** Trong shadowing, cái cần truyền là **suy luận
đang diễn ra**, không phải kết quả. Mentor phải nói ra những câu bình thường không ai nói: "tôi đang
đọc cột này trước vì kinh nghiệm cho thấy lỗi thường ở đây", "tôi chưa biết cái này, tôi sẽ tra".
Câu cuối quan trọng nhất — nó là hành động tạo Psychological Safety và đồng thời dạy mentee rằng
không biết là trạng thái bình thường của công việc kỹ thuật, không phải dấu hiệu bất tài.

**Bước 5 — Đo tiến bộ bằng gì.** Không đo bằng cảm giác "em thấy tự tin hơn". Bốn loại bằng chứng,
theo thứ tự khó làm giả tăng dần:

| Loại bằng chứng | Ví dụ đo được |
|---|---|
| Chất lượng câu hỏi | Câu hỏi chuyển từ "em nên làm gì" sang "em đang nghiêng X vì A, B; rủi ro em thấy là C — anh thấy còn gì em chưa thấy" |
| Phạm vi quyết định tự chịu | Mức delegation tăng từ 3 lên 5 lên 6 cho cùng loại việc, và không có sự cố phát sinh |
| Chuyển giao sang bối cảnh mới | Áp dụng được nguyên lý đã học vào một module chưa từng làm — đây là bằng chứng học được mô hình, không chỉ học được cách làm một việc |
| Trở thành nguồn cho người khác | Người thứ ba bắt đầu hỏi mentee thay vì hỏi mentor; mentee review được PR của người khác và bắt được failure mode |

Loại thứ ba là ranh giới thật giữa "đã được chỉ" và "đã học". Rất nhiều chương trình mentoring dừng
ở loại một và tuyên bố thành công.

**Bước 6 — Phân biệt Mentoring, Coaching và Sponsorship.** Ba thứ này bị gộp thành một ở phần lớn
tổ chức Việt Nam, và việc gộp đó gây ra một hậu quả cụ thể: có những người được dạy rất nhiều trong
ba năm và không được promote, trong khi người khác được promote sớm hơn mà không ai giải thích được
vì sao.

| | Mentoring | Coaching | Sponsorship |
|---|---|---|---|
| **Người nói** | Mentor nói nhiều (chia sẻ kinh nghiệm, mô hình, bản đồ sai lầm) | Coachee nói nhiều; coach chỉ hỏi | Sponsor nói **về** người kia, ở phòng họp người kia không có mặt |
| **Giả định** | Mentee thiếu kiến thức/mô hình mà mentor có | Coachee có đủ dữ kiện, thiếu sự sáng tỏ | Người kia đã đủ năng lực, thiếu **cơ hội và độ nhìn thấy** |
| **Nội dung** | "Hồi tôi làm việc này, chỗ hay gãy là..." | "Nếu bỏ ràng buộc đó ra thì em sẽ làm gì?" | "Việc migration này giao cho Tuấn. Tôi bảo lãnh." |
| **Đo bằng** | Năng lực tăng | Chất lượng và tốc độ quyết định của coachee | Promotion, dự án quan trọng, việc có độ nhìn thấy cao |
| **Rủi ro của mentor/sponsor** | Thời gian | Chậm hơn khi cần nhanh | **Uy tín cá nhân** — đây là lý do sponsorship khan hiếm |

Nói thẳng phần thường bị bỏ qua: **mentoring không tạo ra promotion, sponsorship tạo ra promotion.**
Mentoring nói *với* bạn; sponsorship nói *về* bạn khi bạn không có mặt. Ở phần lớn tổ chức, quyết
định promotion được hình thành trong những cuộc trao đổi mà ứng viên không tham dự, và biến quyết
định mạnh nhất là **có người nào chịu đặt uy tín của mình vào tên bạn hay không**. Một lead có thể
mentor rất tận tâm trong hai năm và vẫn đang giữ người đó lại, nếu lead không bao giờ (a) giao cho
người đó việc có độ nhìn thấy cao trước mặt cấp trên, (b) nêu tên người đó trong buổi calibration,
(c) để người đó tự trình bày trước stakeholder thay vì trình bày thay. Ở bối cảnh Việt Nam, việc
này nghiêm trọng hơn vì hai lý do: nhiều tổ chức không có ladder rõ nên quyết định promotion phụ
thuộc gần như hoàn toàn vào tiếng nói của một hai người; và văn hoá "làm tốt thì tự khắc được thấy"
khiến người giỏi nhưng ít nói bị bỏ qua có hệ thống. Nếu bạn là lead, câu tự kiểm tra là: **quý vừa
rồi bạn đã nói tên ai, với ai, cho việc gì?** Nếu không trả lời được bằng tên người cụ thể, bạn chưa
sponsor ai cả.

### Trade-off

| Trục | Nghiêng về bên A khi | Nghiêng về bên B khi |
|---|---|---|
| **Đầu tư mentoring (A) vs delivery quý này (B)** | Còn ≥ 2 quý để thu hồi; ràng buộc hiện tại là "chỉ một người biết"; có người sẵn sàng và đủ nền | Deadline hợp đồng cứng, không slack; team đang trong giai đoạn cháy; người mentee sắp chuyển team |
| **Mentor là lead (A) vs mentor là peer Senior (B)** | Gap thuộc loại phán đoán và ngữ cảnh kinh doanh; cần cả sponsorship | Gap thuộc kỹ thuật cụ thể; mentee cần không gian sai mà không sợ bị đánh giá — quan hệ có quyền lực làm giảm việc thừa nhận không biết |
| **1-1 sâu (A) vs guild/brown-bag cho nhiều người (B)** | Gap là tacit, cần đồng thực hiện; số người ít | Gap là explicit và lặp lại nhiều người (cách viết migration an toàn, cách đọc dashboard); cần chi phí trên đầu người thấp |
| **Chuẩn hoá chương trình (A) vs cá nhân hoá (B)** | Tổ chức > 30 engineer, cần onboarding lặp lại được, cần công bằng giữa các team | Team nhỏ; gap của mỗi người rất khác nhau; chương trình chuẩn sẽ thành thủ tục điền form |

Ai chịu phần mất: khi chọn delivery quý này, phần mất do mentee chịu (mất một chu kỳ phát triển, và
thị trường không chờ) và do tổ chức chịu ở quý sau (vẫn một điểm nghẽn). Phần được thuộc về lead và
sprint hiện tại. Bất đối xứng này giống với bất đối xứng ở Delegation, và giải thích vì sao mentoring
luôn là việc bị hoãn.

### Real-world Scenarios

**Tình huống A — chọn bài tập vừa quá tầm, và cách nói về nó.** Tiếp tình huống ở chủ đề 1. Minh có
hai lựa chọn cho Tuấn: (1) giao tiếp hai job đối soát tương tự — an toàn, Tuấn làm tốt, nhưng đây là
việc dưới vùng phát triển và sau 6 tháng Tuấn sẽ là người "chuyên đối soát", một dạng bẫy nghề
nghiệp; (2) giao module webhook retry — có state, bất đồng bộ, cần idempotency ở tầng khó hơn, và
Tuấn chưa từng làm. Minh chọn (2), và điểm quyết định chất lượng nằm ở cách giao.

Script — **phiên bản nói sai**:

> **Minh:** "Tuấn ơi, anh thấy em làm đối soát tốt rồi, giờ anh muốn thử thách em cái khó hơn. Cái
> webhook retry này khó lắm nha, cỡ Senior mới làm được. Anh tin em làm được. Cứ làm đi, có gì hỏi
> anh."

Ba vấn đề. "Khó lắm, cỡ Senior mới làm được" đặt cái giá của thất bại vào danh tính chứ không vào
công việc — nếu Tuấn không làm được thì kết luận ngầm là Tuấn không xứng Senior, nên hành vi tối ưu
của Tuấn là chọn phương án bảo thủ nhất và che các phần mình chưa hiểu. "Anh tin em làm được" là lời
động viên không mang thông tin. "Có gì hỏi anh" nghe như mở cửa nhưng thực ra đẩy toàn bộ chi phí
khởi xướng sang Tuấn, và với một task mới, Tuấn không biết mình chưa biết cái gì để mà hỏi.

Script — **phiên bản nói đúng**:

> **Minh:** "Anh muốn giao em webhook retry cho ba đối tác mới. Nói trước cho em rõ vì sao anh chọn
> việc này: nó dựa trên phần em đã làm rồi — idempotency ở job đối soát — nhưng thêm một tầng em
> chưa gặp, là retry với state và với thứ tự event không đảm bảo. Anh ước lượng khoảng 60% em đã có,
> 40% là mới.
>
> Phần 40% mới đó anh không kỳ vọng em tự nghĩ ra hết. Nên hai tuần đầu mình pair 2 buổi, em gõ,
> anh sẽ nói ra tại sao anh lo chỗ này chỗ kia — vì phần đó anh không viết được thành tài liệu.
> Từ tuần 3 em tự làm, anh chỉ đọc RFC.
>
> Một điều anh nói rõ để em không phải đoán: việc này anh coi là đầu tư, không phải bài kiểm tra. Nếu
> em cần thêm một tuần, anh xin scope với Trang, và điều đó không đi vào đánh giá của em. Cái đi vào
> đánh giá là em có nêu ra được các failure mode của thiết kế hay không.
>
> **Tuấn:** "Em chưa từng làm event ordering. Em bắt đầu từ đâu?"
>
> **Minh:** "Câu hỏi tốt hơn là: nếu cùng một webhook đến hai lần, cách nhau 3 giây, đơn hàng đang ở
> trạng thái nào thì hệ thống sai? Em thử liệt kê trong hai ngày, chưa cần code. Rồi mình pair để đi
> qua danh sách của em — anh sẽ thêm những case anh đã gặp mà em không có cách nào đoán được."

Điểm khác biệt về cơ chế: phiên bản đúng tách **độ khó của việc** khỏi **giá trị của người**, nói rõ
tỉ lệ mới/cũ để mentee định vị được mình, cam kết giàn giáo có thời hạn và cam kết rút giàn, và
chuyển câu hỏi mở đầu thành một bài tập liệt kê failure mode — tức là dạy mô hình tư duy trước khi
dạy giải pháp.

**Tình huống B — mentoring không phải công cụ đúng.** ODC khách Nhật, Tech Lead Linh. Duy, Mid 4
năm, sáu tháng qua chất lượng giảm rõ: PR sơ sài, hai lần miss requirement từ khách, im lặng trong
họp. Linh kết luận "Duy thiếu kỹ năng phân tích yêu cầu" và lập chương trình mentoring 12 tuần.
Tuần 5, không có tiến bộ nào; Duy đến buổi 1-1 đúng giờ, ghi chép, và không thay đổi hành vi.

Điều Linh bỏ qua: Duy vào công ty 4 năm trước để làm product, hiện đang là năm thứ ba viết CRUD cho
một hệ thống nội bộ của khách mà Duy chưa từng thấy một người dùng cuối nào. Duy không thiếu năng
lực phân tích; Duy không còn lý do để phân tích. Chương trình mentoring 12 tuần ở đây làm ba việc
có hại: tiêu thời gian của cả hai, tạo ra một bộ tài liệu "đã được hỗ trợ nhưng không cải thiện" mà
sau này sẽ được dùng làm bằng chứng trong performance review, và gửi cho Duy tín hiệu rằng tổ chức
đọc sai vấn đề của mình — điều này làm giảm động lực thêm một bậc.

Cách đọc đúng: trước khi thiết kế can thiệp năng lực, phải chạy chẩn đoán ở chủ đề 4 để loại trừ
bốn giả thuyết khác. Ở tuần 5, dấu hiệu để dừng là rất rõ và Linh đã có nó: **mentee dự đủ buổi
nhưng không thay đổi hành vi giữa các buổi**. Người thiếu kỹ năng nhưng có động lực sẽ thử, làm sai,
rồi hỏi lại. Người không có động lực sẽ tuân thủ hình thức. Hai dạng đó phân biệt được trong hai
tuần nếu người mentor biết mình đang tìm gì.

**Tình huống C — mentoring người hơn tuổi hoặc thâm niên hơn.** Ở fintech, Minh 31 tuổi, Tech Lead;
trong team có một engineer 42 tuổi, 15 năm kinh nghiệm, mạnh về hệ thống cũ nhưng chưa làm việc với
kiến trúc bất đồng bộ. Trong bối cảnh Việt Nam, đặt quan hệ này thành "mentor–mentee" là gần như
chắc chắn thất bại: nó xúc phạm một trục uy tín (tuổi, thâm niên) để phục vụ một trục khác (kỹ
thuật cụ thể), và người kia sẽ hoặc rút lui, hoặc phản kháng ngầm. Cách làm được: **đảo chiều thành
trao đổi tri thức hai phía và không gọi tên nó là mentoring.** Minh nói: "Anh Sơn nắm luồng nghiệp
vụ đối soát cũ hơn tất cả bọn em, phần đó em phải học anh. Còn phần event-driven thì em vừa làm ba
tháng, để em đi qua với anh." Cùng một nội dung chuyển giao, khác nhau ở chỗ nó không yêu cầu một
người thừa nhận vị thế thấp hơn. Đây không phải xảo thuật giao tiếp — nó là việc thừa nhận đúng
rằng uy tín có nhiều trục, và mentoring chỉ hoạt động trên trục mà cả hai đồng ý là có khoảng cách.

### Best Practices

- **Nói ra suy luận, không chỉ kết luận.** Lý do: kết luận là explicit và mentee đọc được trong tài
  liệu; suy luận là tacit và chỉ tồn tại nếu bạn phát âm nó. Một review viết "chỗ này nên dùng
  transaction" dạy được một việc; "chỗ này tôi lo vì nếu process chết giữa hai lần ghi thì trạng thái
  không nhất quán, và tôi từng mất 3 giờ dò một ca như vậy" dạy được một lớp việc.
- **Cam kết độ trễ review, và coi nó là cam kết cứng.** Lý do: độ trễ phản hồi là biến điều khiển
  tốc độ học. Review sau 3 ngày có giá trị bằng khoảng một nửa review sau 3 giờ, vì ngữ cảnh trong
  đầu mentee đã bay hơi.
- **Thừa nhận cái mình không biết, sớm và cụ thể.** Lý do: nó vừa hiệu chỉnh kỳ vọng (mentee không
  học được cái mentor không có), vừa dạy hành vi nêu điểm mù — hành vi nền của Psychological Safety.
- **Rút giàn giáo theo lịch, không theo cảm giác.** Lý do: cảm giác "chưa yên tâm" sẽ không bao giờ
  hết, vì nó là hàm của rủi ro chứ không của năng lực mentee. Đặt trước tuần nào dừng pairing.
- **Mentor cho việc, không mentor cho người một cách chung chung.** Lý do: "anh mentor em" không có
  tiêu chí kết thúc nên nó không bao giờ kết thúc và không bao giờ đo được. Gắn vào một bài tập cụ
  thể thì cả hai biết mình đang ở đâu.
- **Tách vai người mentor và người quyết định lương khi có thể.** Lý do: mentee sẽ không thừa nhận
  điểm yếu thật với người viết đánh giá của mình. Nếu không tách được (team nhỏ, bạn là cả hai), thì
  ít nhất nói rõ ranh giới: cái gì trong buổi mentoring sẽ và sẽ không đi vào review.
- **Sponsor, đừng chỉ mentor.** Lý do: năng lực không tự chuyển thành cơ hội. Mỗi quý, đặt tên một
  người vào một việc có độ nhìn thấy cao và để họ tự trình bày.

### Anti-patterns

- **Mentoring bằng cách kể chuyện mình.** Mọi câu hỏi được trả lời bằng "hồi anh làm ở công ty X...".
  Cơ chế phá hoại: chuyển buổi mentoring thành buổi thoả mãn nhu cầu được công nhận của mentor; mentee
  học được rằng cách để được tôn trọng là kể kinh nghiệm, không phải phân tích. Dấu hiệu sớm: tỉ lệ
  thời gian nói của mentor > 70% và mentee không hỏi lại câu nào.
- **Giao việc dễ để mentee "thành công".** Cơ chế: giữ mentee dưới vùng phát triển, tạo ra chuỗi
  thành công không mang thông tin, và đến kỳ promotion không có bằng chứng nào về việc xử lý phức
  tạp. Dấu hiệu sớm: mentee 18 tháng không có một task nào từng trễ hoặc từng phải đổi hướng.
- **Rescue — cứu ngay khi thấy mentee mắc.** Cơ chế: cắt vòng hành động–hậu quả, và dạy rằng chờ là
  chiến lược đúng. Dấu hiệu sớm: mentor có commit trong branch của mentee.
- **Chương trình mentoring như thủ tục hành chính.** Ghép cặp toàn công ty, có form, có KPI "số buổi
  mentoring/quý". Cơ chế: đo hoạt động thay vì đo kết quả, nên hệ thống tối ưu số buổi và cả hai bên
  điền form. Dấu hiệu sớm: không ai nêu được một gap cụ thể nào đã được đóng.
- **Dùng mentoring làm vỏ cho vấn đề performance.** Lập chương trình mentoring khi thực ra đã quyết
  định cho nghỉ, để có hồ sơ. Cơ chế: phá huỷ độ tin cậy của toàn bộ khái niệm mentoring trong tổ
  chức — sau lần thứ hai, mọi người hiểu "được mentor" là dấu hiệu xấu. Nếu đó là vấn đề performance,
  hãy gọi đúng tên nó (`08-hiring-va-phat-trien.md`).
- **Mentoring chỉ cho người đã giỏi.** Cơ chế: chọn người dễ mentor nhất (có nền tốt, giống mình) làm
  tăng bất bình đẳng cơ hội trong team và bỏ mất phần lớn tiềm năng nâng năng lực. Dấu hiệu sớm: tất
  cả người được lead đầu tư đều cùng background, cùng trường, cùng ngôn ngữ lập trình yêu thích.

### Khi nào KHÔNG nên áp dụng

- **Khi mentee không muốn.** Mentoring là hoạt động đòi mentee tự nguyện phơi ra chỗ mình chưa biết.
  Nếu người đó không muốn — vì đã có kế hoạch nghỉ, vì không tin bạn, vì thấy bị áp đặt — thì mọi
  buổi họp chỉ tạo ra sự tuân thủ hình thức, và tệ hơn, tạo ra hồ sơ giả rằng tổ chức đã hỗ trợ. Hỏi
  thẳng và chấp nhận câu trả lời không: "em có muốn dành 10 tuần cho việc này không, nói thật cũng
  được, anh không đánh giá gì".
- **Khi vấn đề thực chất là động lực hoặc fit.** Đây là chẩn đoán sai phổ biến nhất và đắt nhất. Dấu
  hiệu phân biệt: người thiếu kỹ năng nhưng có động lực sẽ thử, thất bại, rồi quay lại hỏi; người
  thiếu động lực sẽ dự đủ buổi và không thay đổi hành vi. Nếu là vấn đề fit — người đó giỏi nhưng ở
  sai vị trí — thì can thiệp đúng là đổi việc, không phải dạy thêm.
- **Khi mentor không có năng lực đó.** Không mentor được điều mình chưa từng làm; cố làm sẽ tạo ra
  lời khuyên theo mốt và mentee mất niềm tin sau lần thứ hai. Việc đúng là kết nối với người khác
  hoặc chấp nhận rằng gap này phải đóng bằng nguồn ngoài team.
- **Khi tổ chức không có băng thông thật.** Mentoring cần khoảng 2–4 giờ mentor mỗi tuần và cần
  slack trong lịch mentee. Trong một giai đoạn cháy (chuẩn bị audit, sắp go-live hợp đồng, incident
  kéo dài), khởi động chương trình mentoring là cách tạo ra thất bại cho cả hai. Hoãn công khai, có
  ngày cụ thể, tốt hơn khởi động rồi bỏ giữa.
- **Khi gap là explicit và lặp lại nhiều người.** Nếu năm người cùng không biết viết migration an
  toàn, mentoring 1-1 năm lần là lãng phí. Việc đúng là một buổi 90 phút + một checklist + một lint
  rule. Dùng công cụ đắt nhất cho vấn đề rẻ nhất là một dạng lãng phí khó thấy.
- **Khi quan hệ quyền lực làm mentee không thể thừa nhận điểm yếu.** Nếu bạn là người quyết lương và
  quyết promotion, có những gap mentee sẽ không bao giờ nói với bạn. Với các gap đó, mentor đúng là
  một peer hoặc người ngoài team, và việc của bạn là thu xếp điều đó, không phải tự nhận vai.

---

## 3. Coaching

### Problem Statement

Minh — Tech Lead ở fintech — có 14 buổi 1-1 mỗi tháng và cảm giác chúng đều hữu ích: người ta hỏi,
Minh trả lời, câu trả lời đúng, mọi người đi ra hài lòng. Sau bốn tháng, ba hiện tượng cùng xuất
hiện. Số câu hỏi tới Minh không giảm. Trong hai lần Minh nghỉ phép, hai quyết định kiến trúc bị hoãn
đến khi Minh về. Và một Senior nộp đơn với lý do "em không thấy mình được quyết gì" — trong khi Minh
nhớ rõ đã nhiều lần nói "em cứ quyết đi".

Vấn đề không nằm ở việc Minh trả lời sai. Nó nằm ở chỗ mỗi câu trả lời đúng đã giải quyết một task
và đồng thời tước đi một lần tập ra quyết định. Coaching là công cụ cho đúng lớp vấn đề đó: khi người
đối diện **đã có đủ dữ kiện nhưng chưa có sự sáng tỏ**, và cái thiếu là một kết luận họ tự tin thực
thi, không phải một đáp án.

Thiếu Coaching, các hiện tượng đếm được:

- **Tỉ lệ dạng câu hỏi.** Chia câu hỏi tới lead làm hai dạng: "em nên làm gì" và "em đang nghiêng X
  vì A, B, rủi ro em thấy là C, anh thấy còn gì". Nếu dạng một chiếm trên 60% sau một năm làm việc
  cùng nhau, lead đang huấn luyện sự phụ thuộc.
- **Decision latency khi lead vắng.** Đếm số quyết định bị hoãn trong một tuần lead nghỉ. Con số này
  là thước đo trực tiếp của việc năng lực ra quyết định có được phân tán hay không.
- **Nội dung buổi 1-1.** Nếu 1-1 chủ yếu là báo cáo tiến độ (thứ đã có trong Jira) và hỏi–đáp kỹ
  thuật, nó đang thay thế Slack, không đang tạo năng lực.
- **Số lần cùng một loại quyết định phải hỏi lại.** Người đã tự rút ra kết luận về cách chọn giữa hai
  công nghệ sẽ không hỏi lại lần sau cho một cặp công nghệ khác. Người được cho đáp án sẽ hỏi lại.
- **Tỉ lệ nói trong buổi 1-1.** Nếu lead nói hơn 60% thời gian, đó là buổi giảng bài, không phải 1-1.

### First Principles

**Cơ chế một: người ta thực thi tốt hơn với kết luận mình tự rút ra.** Ba lý do độc lập cộng lại.
Thứ nhất, hiệu ứng cam kết–nhất quán: một kết luận do mình phát ngôn trở thành một cam kết công
khai với chính mình, và người ta chịu chi phí để giữ nhất quán với nó. Thứ hai, generation effect
trong tâm lý học nhận thức: thông tin tự sinh ra được ghi nhớ và truy xuất tốt hơn thông tin được
cung cấp — vì quá trình sinh ra tạo nhiều đường liên kết hơn. Thứ ba, và quan trọng nhất về mặt vận
hành: khi gặp tình huống bất ngờ giữa lúc thực thi, người có **mô hình lý do** sẽ tự điều chỉnh
được; người chỉ có **đáp án** sẽ dừng lại và đi hỏi. Đáp án chuyển giao một quyết định. Coaching
chuyển giao hàm sinh ra quyết định.

**Cơ chế hai: phản xạ đưa giải pháp được hệ thống thưởng, nên nó rất khó bỏ.** Với một người kỹ
thuật giỏi, việc nhận ra đáp án trong 20 giây và nói ra nó tạo ba phần thưởng tức thì: cảm giác năng
lực, sự công nhận của người hỏi, và việc kết thúc cuộc hội thoại nhanh. Coaching thì đắt ngay và trả
lãi muộn. Đây là bất đối xứng giống bất đối xứng của Delegation và Mentoring, và nó giải thích vì sao
gần như mọi lead mới đều mặc định directing dù đã đọc về coaching. Sửa bằng ý chí thì không bền; sửa
được bằng **luật cứng về hành vi** (xem quy tắc 10 phút ở phần Framework), vì luật hoạt động ở thời
điểm quyết định, còn ý chí thì không có mặt lúc đó.

**Cơ chế ba: coaching dịch chuyển locus of control vào bên trong.** Khi kết luận đến từ bên ngoài,
người thực thi đọc kết quả xấu là "cái anh ấy chọn không đúng"; khi kết luận đến từ bên trong, kết
quả xấu trở thành dữ liệu để cập nhật mô hình của chính mình. Chỉ trong trường hợp thứ hai vòng học
mới đóng. Đây cũng là lý do coaching và accountability đi cùng nhau: không thể quy accountability
cho một người mà mọi kết luận đều do người khác đưa.

**Cơ chế bốn — giới hạn của coaching, và đây là phần thường bị bỏ:** coaching giả định coachee **có
đủ kiến thức nền, chỉ thiếu sự sáng tỏ**. Khi giả định đó sai, coaching biến thành một trò đoán chữ.
Một Junior chưa từng nghe về idempotency sẽ không tự "khám phá" ra nó bằng bất kỳ chuỗi câu hỏi mở
nào; những câu hỏi đó sẽ được cảm nhận như một cuộc thẩm vấn mà người hỏi đã biết đáp án và đang chờ
mình nói đúng từ khoá. Hậu quả: coachee mất tự tin và học được rằng thừa nhận không biết là nguy
hiểm — đúng ngược mục tiêu. Quy tắc phân định: **thiếu nhận thức thì coaching; thiếu kiến thức thì
mentoring hoặc dạy thẳng.** Cách kiểm tra nhanh trong 60 giây: hỏi "em thấy có mấy lựa chọn?" Nếu
người kia liệt kê được ≥ 2 lựa chọn hợp lý, họ có nền — coaching được. Nếu họ liệt kê được 0 hoặc 1,
họ thiếu nền — chuyển sang cung cấp thông tin trước, coaching sau.

**Cơ chế năm: chi phí thời gian là thật và không đối xứng theo tình huống.** Coaching một quyết định
tốn 30–45 phút; directing tốn 3 phút. Trong tình huống mà chi phí mỗi phút cao (production đang
xuống, deadline hợp đồng đêm nay), phép tính đảo chiều hoàn toàn. Coaching không phải một giá trị
đạo đức luôn đúng; nó là một khoản đầu tư có điều kiện biên rõ ràng.

### Mental Model

**Ladder of inference và việc tháo ràng buộc.** Phần lớn bế tắc kỹ thuật không phải do thiếu phương
án mà do một giả định không được nói ra đang chặn không gian tìm kiếm ("phải dùng đúng stack hiện
tại", "không thể xin thêm hai tuần", "khách sẽ không đồng ý"). Chức năng cơ học của câu hỏi coaching
là **làm hiện lên các giả định và thử tháo từng cái**: "nếu ràng buộc đó không có thì em sẽ làm gì?"
Câu hỏi này thường mở ra phương án mà cả hai đều chưa thấy, kể cả người coach.

**Circle of Influence.** Người bế tắc thường đang tiêu toàn bộ năng lượng vào vùng quan tâm mà mình
không kiểm soát (khách hàng đổi ý, công ty không có ngân sách, team khác chậm). Câu hỏi coaching kéo
sự chú ý về vùng ảnh hưởng: "trong việc này, cái gì em quyết được?" Đây là can thiệp có hiệu lực cao
nhất trên một người đang ở trạng thái nạn nhân, và nó không đòi hỏi coach biết đáp án.

**Đường cong năng lực vs decision latency.** Directing làm giảm latency của quyết định *này* và giữ
nguyên năng lực. Coaching làm tăng latency của quyết định *này* và giảm latency của cả lớp quyết
định *sau này*. Chọn giữa hai cái là chọn giữa hai điểm trên đường cong — và tiêu chí chọn là: loại
quyết định này còn lặp lại không, và có bao nhiêu thời gian trước khi cần kết quả.

### Practical Framework

**GROW, diễn đạt lại cho ngữ cảnh kỹ thuật.** Bốn pha, và giá trị nằm ở việc **không nhảy pha**.
Sai lầm phổ biến nhất là nhảy từ G sang W trong hai phút.

| Pha | Mục tiêu của pha | Câu hỏi dùng được | Dấu hiệu chưa xong pha này |
|---|---|---|---|
| **G**oal | Chốt cái đang cần quyết, và tiêu chí thế nào là tốt | "Cuối buổi này em muốn ra khỏi đây với cái gì?" / "Quyết định này phục vụ kết quả nào?" / "Nếu ba tháng nữa nhìn lại, thế nào là đã chọn đúng?" | Coachee mô tả triệu chứng thay vì quyết định cần ra |
| **R**eality | Dữ kiện thật, phân biệt cái đã đo với cái đang đoán | "Con số hiện tại là bao nhiêu?" / "Cái nào em đã đo, cái nào em đang giả định?" / "Em đã thử gì, kết quả ra sao?" | Toàn bộ mô tả là tính từ ("chậm", "không ổn định") mà không có số |
| **O**ptions | Mở rộng không gian phương án trước khi thu hẹp | "Có mấy cách?" / "Nếu không được thêm người thì sao?" / "Nếu ràng buộc X không có thì sao?" / "Cách nào rẻ nhất để biết mình sai?" | Chỉ có 1 phương án, hoặc 2 phương án trong đó 1 là bù nhìn |
| **W**ay forward | Chuyển thành hành động có chủ thể, có mốc, có tiêu chí | "Em làm gì trước, khi nào?" / "Cái gì có thể làm việc này chệch, và em phát hiện bằng cách nào?" / "Em cần gì từ anh?" | Kết thúc bằng "em sẽ suy nghĩ thêm" |

**Thang câu hỏi, năm bậc theo độ đắt.** Đi từ dưới lên; nhảy lên bậc 4–5 quá sớm bị cảm nhận như
công kích.

```
Bậc 1 — Mở:        "Kể anh nghe tình hình."            (thu thập, chưa định hướng)
Bậc 2 — Chẩn đoán: "Cụ thể chậm ở đâu, đo bằng gì?"    (buộc đưa về dữ kiện)
Bậc 3 — Phương án: "Em thấy có mấy cách? Còn cách nào nữa?"
Bậc 4 — Giả định:  "Cái gì phải đúng thì phương án đó mới đúng?"
                   "Nếu ràng buộc đó không tồn tại, em chọn gì?"
Bậc 5 — Hệ quả:    "Nếu chọn cái này, sáu tháng nữa cái gì sẽ khó hơn?"
                   "Rẻ nhất để biết mình sai là gì?"
Chốt — Cam kết:    "Em quyết gì? Ai làm? Bao giờ? Anh cần biết gì để không hỏi lại?"
```

**Quy tắc "không đưa giải pháp trong 10 phút đầu".** Đây là một luật hành vi, không phải một lời
khuyên. Cách thực hiện: bật đồng hồ, và trong 10 phút đầu chỉ được dùng ba loại phát ngôn — câu hỏi,
diễn đạt lại điều vừa nghe ("ý em là..."), và im lặng. Ba tác dụng theo cơ chế: (a) coachee buộc phải
đi hết vấn đề nên thường tự tìm ra phương án ở phút thứ 6–8; (b) coach thu thập đủ dữ kiện — rất
nhiều lời khuyên tệ được sinh ra ở phút thứ hai vì coach đang giải một bài toán khác với bài toán
thật; (c) coachee học rằng buổi này là không gian để suy nghĩ, không phải quầy phát đáp án. Sau 10
phút, coach được phép nói — và nên nói, nếu có thông tin coachee không thể có. Ngoại lệ khai báo
trước: nếu đang trong incident hoặc coachee đang đi vào một quyết định không thể phục hồi, bỏ quy
tắc, nhưng **nói ra là mình đang bỏ**: "Anh cắt vào đây vì cái này không phục hồi được, không phải
vì anh không tin em."

**Cấu trúc buổi 1-1 45 phút dùng được từ tuần sau.**

```
0-5':   Coachee đặt agenda. Câu mở: "Hôm nay em muốn dùng 45 phút này cho việc gì?"
        (Nếu lead đặt agenda, buổi này là buổi của lead — và nó sẽ thành báo cáo tiến độ.)
5-20':  Reality + Options. Lead chỉ hỏi. KHÔNG đưa giải pháp.
20-35': Lead được nói: thông tin coachee không thể có (bối cảnh kinh doanh, lịch sử,
        ràng buộc từ trên), rủi ro coachee chưa thấy. Nói rõ đây là input, không phải lệnh.
35-42': Chốt Way forward. Coachee phát ngôn quyết định, không phải lead.
42-45': Lead ghi lại quyết định + cam kết của chính lead ("anh sẽ nói với Trang về scope").
```

**Cách phân biệt trong 60 giây cần coaching hay cần dạy.** Hỏi: "em thấy có mấy lựa chọn, và mỗi cái
được gì mất gì?" Trả lời có ≥ 2 lựa chọn với trade-off → coaching. Trả lời 0–1 lựa chọn hoặc trade-off
sai bản chất → cung cấp kiến thức, rồi coaching sau. Việc chẩn đoán này phải làm mỗi lần, vì cùng một
người có nền ở vùng này và không có nền ở vùng khác.

### Trade-off

| Trục | Nghiêng về Coaching khi | Nghiêng về Directing khi |
|---|---|---|
| **Coaching vs Directing** | Loại quyết định này còn lặp lại; có thời gian; coachee có kiến thức nền; hậu quả sai phục hồi được; mục tiêu là tăng số điểm quyết định | Production đang xuống; deadline trong vài giờ; quyết định irreversible; coachee thiếu nền; ràng buộc compliance không thương lượng |
| **Chậm bây giờ vs phụ thuộc lâu dài** | Còn ≥ 2 quý cùng nhau; đang thấy dấu hiệu phụ thuộc (tỉ lệ câu hỏi "em nên làm gì" cao) | Người đó sắp rời team; việc one-off; bạn không có 45 phút và nói dối rằng có thì tệ hơn |
| **Coaching vs Mentoring** | Coachee liệt kê được phương án và trade-off | Coachee không biết phương án nào tồn tại — khi đó hỏi mở là thẩm vấn |
| **Coaching 1-1 vs quyết định nhóm** | Vấn đề thuộc phạm vi một người sở hữu | Quyết định ảnh hưởng nhiều người — khi đó cần RFC và phản biện công khai (`04-decision-making.md`), coaching riêng lẻ sẽ tạo ra sự đồng thuận giả |

Ai chịu phần mất khi chọn Directing: coachee mất một lần tập và mất cơ hội tích luỹ bằng chứng cho
promotion; tổ chức mất một điểm quyết định tiềm năng; lead được lợi ngay (nhanh, chắc). Khi chọn
Coaching trong tình huống sai (production đang cháy): tổ chức chịu chi phí downtime, và coachee chịu
cảm giác bị bỏ mặc — "lúc cần thì anh ấy hỏi triết lý". Cả hai chiều đều có thiệt hại thật.

### Real-world Scenarios

**Tình huống A — "anh nghĩ nên chọn Kafka hay RabbitMQ?"** Quân, Senior BE ở fintech, 6 năm kinh
nghiệm, đang thiết kế luồng xử lý webhook từ các đối tác thanh toán. Quân bước vào bàn Minh với câu
hỏi trên. Đây là tình huống xảy ra mỗi tuần ở mọi công ty, và cách trả lời nó quyết định nhiều thứ
hơn là chọn được đúng message broker.

Script — **phiên bản trả lời trực tiếp**:

> **Quân:** "Anh Minh, cái luồng webhook em đang thiết kế, anh nghĩ nên chọn Kafka hay RabbitMQ?"
>
> **Minh:** "Kafka. Vì mình sẽ cần replay event khi đối soát, mà RabbitMQ thì message mất sau khi
> ack nên replay phải tự lưu thêm. Với lại mình sắp có team analytics muốn consume cùng stream, Kafka
> có consumer group nên thêm consumer không ảnh hưởng luồng cũ. Em cứ dùng Kafka, để anh gửi em cái
> config của cluster hiện tại."
>
> **Quân:** "Ok anh, vậy em làm Kafka."

Đây là một câu trả lời **đúng về kỹ thuật**, đưa ra trong 90 giây, có nêu lý do. Nó tốt hơn nhiều
mức trung bình. Nhưng hãy đọc các hệ quả:

1. **Quân không sở hữu quyết định.** Ba tháng sau, khi Kafka gây ra vấn đề (rebalance làm consumer
   lag tăng đột biến lúc peak), phản ứng tự nhiên của Quân là "cái này anh Minh chọn". Điều này không
   phải Quân thiếu tinh thần trách nhiệm — nó là hệ quả logic của việc ai đã ra quyết định.
2. **Lớp quyết định không được chuyển giao.** Lần sau, khi cần chọn giữa Redis Streams và SQS cho
   một luồng khác, Quân sẽ lại hỏi. Minh đã cấp một đáp án, không cấp một phương pháp.
3. **Minh có thể đã trả lời một câu hỏi khác với câu hỏi thật.** Minh không hỏi throughput dự kiến,
   không hỏi có cần thứ tự theo partition key hay không, không hỏi team có ai vận hành được Kafka
   lúc 2 giờ sáng hay không. Trong trường hợp cụ thể này, khối lượng thật là khoảng 200 event/giây
   với yêu cầu at-least-once và không cần replay quá 7 ngày — RabbitMQ với một bảng outbox có thể
   là lựa chọn có tổng chi phí vận hành thấp hơn nhiều, đặc biệt khi team chưa có ai từng chạy Kafka
   trên production. Câu trả lời nhanh đã đóng khảo sát trước khi mở.
4. **Tín hiệu tổ chức.** Quân là Senior 6 năm. Việc một Senior hỏi câu này và nhận đáp án củng cố
   một chuẩn mực: quyết định kiến trúc thuộc về Minh. Đây chính là cơ chế sinh ra hiện tượng ở đầu
   chủ đề — hai quyết định bị hoãn khi Minh nghỉ phép.

Script — **phiên bản coaching**:

> **Quân:** "Anh Minh, cái luồng webhook em đang thiết kế, anh nghĩ nên chọn Kafka hay RabbitMQ?"
>
> **Minh:** "Anh có ý kiến, và anh sẽ nói — nhưng để sau. Trước hết em cho anh biết: quyết định này
> đang phục vụ cái gì? Tức là nếu chọn sai thì cái gì sẽ hỏng?"
>
> **Quân:** "Webhook từ đối tác đang xử lý đồng bộ, giờ cao điểm timeout, đối tác retry, mình
> double-count. Em cần tách ra async để không mất event và không xử lý trùng."
>
> **Minh:** "Vậy yêu cầu thật là không mất event và không trùng. Em đã đo được số nào?"
>
> **Quân:** "Peak khoảng 200 event/giây, ngày sale gấp ba. Payload nhỏ. Em cần giữ được thứ tự theo
> từng đơn hàng, giữa các đơn thì không cần."
>
> **Minh:** "Tốt, đó là ràng buộc quan trọng nhất em vừa nói ra. Em thấy có mấy cách, không chỉ hai
> cái em nêu?"
>
> **Quân:** "Kafka, RabbitMQ... hoặc là outbox table cộng worker tự viết. Hoặc SQS nếu mình chấp
> nhận thứ tự chỉ trong FIFO group."
>
> **Minh:** "Bốn cách. Giờ câu này: cái gì phải đúng thì Kafka mới là lựa chọn tốt hơn?"
>
> **Quân:** "Phải là... cần replay dài hạn, cần nhiều consumer độc lập, và throughput cao hơn hẳn
> mức hiện tại."
>
> **Minh:** "Ba điều kiện đó, cái nào mình đang có thật, cái nào là mình đoán mình sẽ cần?"
>
> **Quân:** "Thật thì... không có cái nào. Nhiều consumer là em đoán, team analytics có nói muốn
> nhưng chưa có kế hoạch. Replay thì em nghĩ đối soát sẽ cần, nhưng đối soát đang đọc từ DB."
>
> **Minh:** "Đây là chỗ anh có thông tin em không có, nên anh nói: Q4 năm ngoái bọn anh dựng một
> cluster Kafka cho luồng notification. Nó chạy được, nhưng hai lần rebalance lúc peak làm lag lên 40
> phút, và cả team chỉ có Khoa xử lý được. Khoa hiện đã sang platform team. Đó là chi phí vận hành
> thật, không nằm trong tài liệu so sánh nào.
>
> Bây giờ câu quyết định: với 200 event/giây, thứ tự theo đơn hàng, và team hiện tại không có ai vận
> hành được Kafka lúc 2 giờ sáng — em chọn gì, và cách rẻ nhất để biết em chọn sai là gì?"
>
> **Quân:** "Em chọn RabbitMQ với routing key theo order_id, cộng outbox table để không mất event khi
> publish fail. Cách biết mình sai: nếu trong 6 tháng có consumer thứ ba cần cùng dữ liệu, hoặc
> throughput vượt 1.000/giây, thì thiết kế này phải đổi. Em sẽ ghi hai ngưỡng đó vào ADR làm điều kiện
> xem lại."
>
> **Minh:** "Anh đồng ý với lựa chọn đó, và anh muốn em ghi thêm một dòng: nếu đổi thì đổi bằng cách
> nào, ước lượng bao lâu. Em chốt, không cần anh duyệt. Anh chỉ đọc ADR."

So sánh hệ quả của hai phiên bản, theo bốn trục:

| | Phiên bản trực tiếp | Phiên bản coaching |
|---|---|---|
| Thời gian | ~2 phút | ~20 phút |
| Chất lượng quyết định | Có thể sai vì thiếu dữ kiện (throughput thật, ai vận hành được) | Cao hơn — dữ kiện được đưa ra trước khi kết luận |
| Ownership | Thuộc Minh; khi hỏng sẽ có tranh luận về ai chọn | Thuộc Quân; Quân bảo vệ và điều chỉnh được |
| Lớp quyết định sau | Quân hỏi lại lần sau | Quân có phương pháp: yêu cầu → điều kiện phải đúng → chi phí vận hành → ngưỡng xem lại |
| Tri thức tổ chức | Không có gì được ghi lại | Có ADR với điều kiện xem lại — quyết định trở thành tài sản chứ không phải giai thoại |

Điểm mấu chốt về mặt kỹ thuật coaching: Minh **vẫn đưa thông tin** (bài học Kafka Q4, việc Khoa đã
chuyển team). Coaching không có nghĩa là giữ lại điều mình biết — giữ lại thông tin mà người kia
không thể tự có là một dạng thao túng, không phải coaching. Coaching là **thứ tự**: dữ kiện và phương
án trước, thông tin bổ sung của lead sau, kết luận do coachee phát ngôn cuối cùng.

**Tình huống B — coaching sai đối tượng.** Vy, Junior 8 tháng, hỏi Minh: "anh ơi API list đơn hàng
chậm, em nên làm gì?" Minh, đang cố thực hành coaching, hỏi: "em thấy có mấy hướng?" Vy: "em... chưa
biết ạ." Minh hỏi tiếp: "em thử nghĩ xem cái gì có thể làm nó chậm?" Vy im lặng 20 giây, rồi nói "chắc
do code em viết chưa tối ưu". Buổi đó kết thúc sau 25 phút, Vy không có hướng đi, và bài học Vy rút ra
là mình đáng lẽ phải biết mà lại không biết.

Chẩn đoán: Vy không thiếu sự sáng tỏ, Vy thiếu **danh mục nguyên nhân**. Không ai tự khám phá ra sự
tồn tại của N+1 query, thiếu index, hoặc pagination bằng offset lớn nếu chưa từng nghe về chúng.
Việc đúng là chuyển chế độ, và nói rõ là mình đang chuyển: "Câu này anh không hỏi tiếp, anh chỉ luôn:
với API list chậm, có bốn nguyên nhân thường gặp — N+1, thiếu index, offset lớn, và trả quá nhiều
field. Anh chỉ em cách kiểm tra từng cái. Lần sau khi gặp API chậm, em chạy bốn kiểm tra này trước
rồi mang kết quả tới, lúc đó mình bàn." Sáu tuần sau, cùng câu hỏi, Vy đã có nền — và lúc đó coaching
mới hoạt động.

**Tình huống C — coaching trong ODC nơi quyết định không thuộc mình.** Linh coach Duy về một thiết kế
mà khách hàng Nhật đã chốt và không thay đổi. Coaching trong không gian bằng 0 là hình thức tệ nhất
của coaching: nó tạo ra ảo giác về quyền quyết định và kết thúc bằng việc coachee bị bác bỏ. Cách làm
đúng là **nói rõ biên trước khi coach**: "Phần kiến trúc tổng thể khách đã chốt, mình không đổi được.
Phần em quyết được là cách chia module bên trong, chiến lược test, và thứ tự triển khai. Anh muốn
coach em ở đúng ba phần đó." Coaching có hiệu lực tỉ lệ với độ rộng của không gian quyết định thật;
nếu không gian đó bằng 0, việc cần làm là mở rộng không gian (đàm phán với khách) hoặc nói thật rằng
không có, không phải coach.

### Best Practices

- **Nói ra chế độ mình đang dùng.** "Câu này anh trả lời thẳng" hoặc "câu này anh muốn hỏi em vài
  câu trước". Lý do: người đối diện không đọc được ý định của bạn, và một chuỗi câu hỏi không được
  khai báo sẽ bị đọc là thẩm vấn hoặc là bài kiểm tra.
- **Để coachee phát ngôn quyết định cuối cùng.** Lý do: người phát ngôn là người sở hữu. Nếu lead
  tóm lại "vậy em làm A nhé", ownership vừa chuyển về lead trong một câu.
- **Dùng im lặng.** Sau một câu hỏi khó, chờ đủ 8–10 giây. Lý do: người ta cần thời gian truy xuất,
  và lead lấp khoảng trống bằng gợi ý là cách phá vỡ chính xác cái mình đang cố tạo ra.
- **Hỏi "cái gì phải đúng thì phương án này đúng".** Lý do: đây là câu hỏi có tỉ lệ giá trị/chi phí
  cao nhất trong toàn bộ danh mục, vì nó biến một sở thích công nghệ thành một tập giả định kiểm tra
  được.
- **Kết bằng "em cần gì từ anh".** Lý do: nó đảo chiều dòng yêu cầu và làm lộ ra các blocker thuộc
  quyền lead (xin scope, nói với team khác) mà coachee thường không dám nêu.
- **Đưa thông tin coachee không thể tự có, và đưa sau chứ không trước.** Lý do: đưa trước thì nó
  đóng khung không gian tìm kiếm; đưa sau thì nó hiệu chỉnh một không gian đã mở.
- **Ghi lại quyết định vào ADR/RFC ngay sau buổi coaching.** Lý do: nếu không ghi, buổi coaching tạo
  ra năng lực cá nhân nhưng không tạo ra tri thức tổ chức, và sáu tháng sau không ai nhớ điều kiện
  nào đã dẫn tới lựa chọn đó.

### Anti-patterns

- **Coaching như một cách trốn trách nhiệm.** Lead không có quan điểm, hỏi vòng vo, để coachee tự
  quyết cả những việc thuộc thẩm quyền lead (ưu tiên, scope, phân bổ người). Cơ chế phá hoại: chuyển
  gánh nặng quyết định xuống người không có đủ thông tin và không có quyền. Dấu hiệu sớm: coachee ra
  khỏi buổi 1-1 với nhiều câu hỏi hơn lúc vào.
- **Câu hỏi dẫn dắt tới đáp án có sẵn.** "Em có nghĩ là dùng cache thì tốt hơn không?" Cơ chế: đây là
  directing đội lốt coaching, và nó tệ hơn directing thẳng vì nó tiêu thời gian của cả hai và làm
  coachee cảm thấy bị lừa. Dấu hiệu sớm: mọi câu hỏi của lead đều là câu hỏi đóng có/không.
- **Coaching khi người kia thiếu kiến thức nền.** Xem tình huống B. Dấu hiệu sớm: coachee trả lời
  "em không biết" hai lần liên tiếp, hoặc thời gian im lặng dài kèm biểu hiện lo lắng.
- **Biến 1-1 thành buổi status report.** Cơ chế: lead lấy thông tin đã có trong Jira và không tạo ra
  giá trị nào, nên 1-1 dần bị coi là nghi lễ và bị huỷ đầu tiên khi bận. Dấu hiệu sớm: agenda do lead
  đặt; câu mở đầu là "tuần này em làm được gì rồi".
- **Coaching trong lúc cháy.** Cơ chế: chi phí mỗi phút cao, và câu hỏi mở trong incident bị đọc là
  lead không biết phải làm gì. Dấu hiệu sớm: lead hỏi "em nghĩ nên làm gì" khi biểu đồ error rate
  đang dựng đứng.
- **Coaching thành trị liệu.** Lead đi vào vùng cảm xúc, gia đình, động lực sống của coachee mà không
  có năng lực và không có vai. Cơ chế: vượt biên vai trò, tạo ra sự phụ thuộc cảm xúc và rủi ro
  nghiêm trọng khi cùng người đó viết đánh giá hiệu suất. Dấu hiệu sớm: buổi 1-1 kéo dài 90 phút và
  nội dung không liên quan gì tới công việc.

### Khi nào KHÔNG nên áp dụng

- **Trong incident hoặc khủng hoảng có đồng hồ đếm.** Cơ chế cần là ra chỉ thị rõ ràng và ngắn
  (`06-incident-va-metrics.md`). Coaching hoãn lại đến postmortem — và ở postmortem thì nó là công cụ
  tốt nhất có. Sự khác biệt không phải về triết lý mà về chi phí mỗi phút.
- **Khi coachee thiếu kiến thức nền cho lớp vấn đề đó.** Hỏi mở trong tình trạng này gây hại: nó tạo
  ra cảm giác bị kiểm tra và dạy rằng thừa nhận không biết là nguy hiểm. Kiểm tra bằng câu hỏi "em
  thấy có mấy lựa chọn" trước khi chọn chế độ.
- **Khi quyết định không thuộc coachee.** Nếu ràng buộc từ hợp đồng, compliance, hoặc cấp trên đã
  đóng không gian, coaching tạo ra kỳ vọng sai và kết thúc bằng sự bất tín. Nói thẳng ràng buộc, rồi
  coach trong phần không gian còn lại.
- **Khi cái cần là một tiêu chuẩn chung, không phải một quyết định cá nhân.** Nếu năm người cùng cần
  chọn cách xử lý lỗi trong API, kết quả của năm buổi coaching là năm cách khác nhau. Việc đúng là
  một chuẩn viết ra và một buổi phản biện công khai.
- **Khi có bất đối xứng quyền lực làm coachee không thể nói thật.** Nếu bạn là người quyết lương và
  chủ đề là một thất bại của coachee, coachee sẽ trình bày phiên bản an toàn. Coaching trên dữ kiện
  đã bị lọc sẽ dẫn tới kết luận sai. Với các chủ đề đó, cần người coach khác hoặc cần tách rõ ranh
  giới trước.
- **Khi vấn đề là performance đã được xác định và cần ranh giới rõ.** Người đang trong tình trạng
  không đạt kỳ vọng cần biết chính xác kỳ vọng là gì và hậu quả nếu không đạt. Câu hỏi mở ở thời
  điểm đó bị đọc là mơ hồ và trì hoãn, và tước đi của họ quyền được biết mình đang ở đâu.

---

## 4. Motivation

### Problem Statement

Một team e-commerce 9 người ở Hà Nội. Velocity trên biểu đồ đều đặn suốt hai quý, không có sprint
nào cháy, không ai nghỉ. Nhìn từ dashboard: khoẻ. Nhìn kỹ hơn: trong 6 tháng không có một đề xuất cải
tiến nào xuất phát từ trong team; mọi ticket đều được làm đúng phạm vi mô tả và không có gì hơn; hai
alert bị mute từ tháng trước và không ai hỏi; buổi retro có 9 người và trung bình 4 người phát biểu,
luôn là 4 người đó; và khi một đối thủ trả cao hơn 25%, ba người rời đi trong sáu tuần — cả ba đều
nói trong exit interview rằng họ đã nghĩ đến việc này từ bốn, năm tháng trước.

Đây là hình thái phổ biến nhất của mất động lực trong công việc kỹ thuật: **không phải sự sụt giảm
sản lượng, mà là sự biến mất của nỗ lực tuỳ ý (discretionary effort)** — phần công việc không ai yêu
cầu và không ai đo, nhưng chiếm phần lớn giá trị của một engineer giỏi: nhận ra một cảnh báo bất
thường, viết thêm test cho một ca không ai nghĩ tới, nói ra rằng thiết kế này sẽ gãy ở quý sau.

Các đại lượng đếm được:

- **Số đề xuất cải tiến xuất phát từ trong team trong 3 tháng**, không đếm những đề xuất do lead gợi
  ý. Bằng 0 là một con số rất xấu, kể cả khi velocity tốt.
- **Số người tự nguyện nhận việc khó/không hào nhoáng** (sửa flaky test, viết runbook, nhận on-call
  thêm ca). Nếu luôn là cùng một hoặc hai người, phần còn lại đã rút về mức tối thiểu.
- **Tỉ lệ phát biểu trong retro** và số action item do người không phải lead đề xuất.
- **Regretted attrition** — tỉ lệ nghỉ việc của những người mà tổ chức muốn giữ, tách khỏi tỉ lệ nghỉ
  chung. Đây là chỉ số duy nhất về động lực mà không thể tự lừa dối.
- **Khoảng thời gian từ khi một người bắt đầu tìm việc đến khi nộp đơn.** Ở phần lớn trường hợp là
  3–6 tháng. Nếu lead chỉ biết ở tuần cuối, hệ thống tín hiệu của lead không hoạt động.
- **Số câu hỏi "vì sao mình làm cái này" trong grooming.** Bằng 0 không phải dấu hiệu mọi thứ đã rõ;
  thường là dấu hiệu không ai còn thấy việc hỏi có ích.

### First Principles

**Cơ chế một: ba nguồn động lực nội tại, và chúng không thay thế được nhau.** Self-Determination
Theory (Deci và Ryan) xác định ba nhu cầu tâm lý cơ bản; Daniel Pink phổ biến chúng dưới dạng
Autonomy, Mastery, Purpose. Diễn đạt cho công việc kỹ thuật: **Autonomy** là có không gian quyết định
thật về cách làm; **Mastery** là cảm giác đang giỏi lên ở việc mình coi là quan trọng; **Purpose** là
thấy được mối liên hệ giữa việc mình làm và một kết quả mình quan tâm. Điểm then chốt về mặt vận
hành: ba cái này **không bù trừ**. Purpose rất cao không bù được Autonomy bằng 0 — đó là mô tả chính
xác của trạng thái burnout ở người làm việc trong tổ chức phi lợi nhuận hoặc trong các đội cứu hộ sự
cố kéo dài. Mastery cao không bù được Purpose bằng 0 — đó là mô tả của một engineer giỏi viết CRUD
cho hệ thống nội bộ mà mình chưa từng thấy người dùng.

**Cơ chế hai: crowding-out effect.** Khi một hoạt động vốn được làm vì lý do nội tại bị gắn với phần
thưởng ngoại tại, động lực nội tại có thể giảm. Cơ chế là sự chuyển dịch quy gán: từ "tôi làm vì tôi
muốn làm" sang "tôi làm vì được trả cho việc này". Hệ quả vận hành cụ thể: đặt tiền thưởng cho việc
viết tài liệu hoặc cho số PR review sẽ có tác dụng ngắn hạn, sau đó khi tiền thưởng dừng, mức độ thực
hiện thường thấp hơn mức ban đầu — vì lý do nội tại đã bị thay thế và không tự quay lại. Điều này
không có nghĩa là không nên thưởng; nó có nghĩa là thưởng nên gắn với **kết quả khó lượng hoá thành
đầu ra đếm được** và nên bất ngờ hơn là theo công thức.

**Cơ chế ba: hygiene factor và motivator là hai trục khác nhau, không phải hai đầu của một trục.**
Herzberg: nhóm yếu tố vệ sinh (lương, điều kiện làm việc, chính sách, quan hệ với cấp trên, sự công
bằng) khi thiếu thì gây bất mãn; khi đủ thì **chỉ xoá bất mãn**, không tạo ra engagement. Nhóm động
lực (thành tựu, sự công nhận, bản chất công việc, trách nhiệm, sự phát triển) mới tạo ra engagement.
Diễn đạt thẳng cho bối cảnh Việt Nam: **lương xoá được cảm giác bị đối xử bất công, nhưng không mua
được nỗ lực tuỳ ý.** Một người được tăng 20% sẽ hết bất mãn về lương trong khoảng 2–4 tháng, rồi
quay lại đúng mức engagement trước đó nếu ba yếu tố Autonomy–Mastery–Purpose không đổi. Đây là lý do
đối thoại giữ người bằng tiền hầu như luôn chỉ trì hoãn việc ra đi thêm một hai quý.

**Cơ chế bốn: động lực là hàm của so sánh, không phải hàm của mức tuyệt đối.** Equity theory (Adams):
người ta đánh giá phần thưởng của mình bằng cách so tỉ lệ (đóng góp / phần thưởng) của mình với một
người tham chiếu. Hệ quả phản trực giác nhưng rất quan trọng: **tăng lương 25% cho một người có thể
làm giảm động lực của ba người khác nhiều hơn phần tăng lên ở người được tăng** — nếu ba người kia
biết và không hiểu tiêu chí. Ở thị trường Việt Nam, hiệu ứng này bị khuếch đại bởi hai đặc thù: lương
biến động nhanh nên người mới vào có thể cao hơn người cũ cùng level (salary compression), và việc
đối chiếu lương giữa bạn bè cùng khoá diễn ra thường xuyên và không đầy đủ ngữ cảnh — người ta so con
số gross mà không so scope công việc, không so mức độ ổn định, không so phần học được.

**Cơ chế năm: progress principle.** Nghiên cứu của Teresa Amabile trên hàng nghìn nhật ký làm việc
cho kết quả rằng yếu tố dự báo mạnh nhất của trạng thái nội tâm tích cực trong một ngày làm việc là
**cảm giác tiến bộ trong công việc có ý nghĩa**, mạnh hơn cả sự công nhận hay phần thưởng. Suy ra một
can thiệp rẻ mà hầu như không ai làm: loại bỏ những thứ chặn cảm giác tiến bộ — CI chạy 40 phút, PR
chờ review 3 ngày, task bị đổi ưu tiên giữa sprint, môi trường staging hỏng một tuần. Những thứ này
thường bị xếp vào "vấn đề kỹ thuật", nhưng chúng là **vấn đề động lực có nguyên nhân kỹ thuật**, và
chúng ăn vào động lực đều đặn mỗi ngày.

### Mental Model

**Động lực không phải một bình chứa trong người, nó là một hàm của môi trường.** Câu "bạn ấy thiếu
động lực" đặt nguyên nhân vào tính cách và dẫn tới can thiệp vào tính cách (động viên, nhắc nhở, nói
chuyện về thái độ) — loại can thiệp có hiệu lực gần bằng 0. Mô hình dùng được:

```
Motivation = f(Clarity, Capability, Autonomy, Fairness, Meaning)
```

Năm biến, mỗi biến có triệu chứng riêng và can thiệp riêng. Việc của lead không phải "tăng động lực"
mà là **xác định biến nào đang bằng thấp** — vì can thiệp vào biến sai không chỉ vô hiệu mà còn có
hại: nó chứng minh cho người kia rằng lead không hiểu vấn đề của họ.

**Expectancy theory (Vroom) như một chuỗi ba mắt.** Nỗ lực xảy ra khi cả ba mắt đều nối: (1) nếu tôi
cố thì tôi làm được không? (2) nếu tôi làm được thì có dẫn tới kết quả tôi mong không? (3) kết quả đó
tôi có coi là đáng không? Ba mắt đứt cho ba triệu chứng hoàn toàn khác nhau: mắt một đứt tạo ra sự né
tránh việc khó; mắt hai đứt tạo ra sự cynicism ("làm tốt cũng thế thôi"); mắt ba đứt tạo ra sự thờ ơ
lịch sự. Nghe câu người ta nói là đủ để đoán mắt nào đứt.

### Practical Framework

**Bước 1 — Bảng chẩn đoán năm giả thuyết.** Đây là công cụ trung tâm của chủ đề này. Nguyên tắc:
chẩn đoán trước, can thiệp sau; và mỗi lần chỉ thay đổi một biến để biết cái gì có tác dụng.

| Giả thuyết | Triệu chứng quan sát được | Câu hỏi kiểm tra trong 1-1 | Can thiệp đúng | Can thiệp sai thường dùng | Thấy kết quả sau |
|---|---|---|---|---|---|
| **Thiếu Clarity** — không biết cái gì là quan trọng, hoặc kỳ vọng mâu thuẫn | Làm đúng chữ sai ý; hỏi lại nhiều lần cùng một việc; ưu tiên sai; "em tưởng cái kia gấp hơn" | "Theo em, ba việc quan trọng nhất của em quý này là gì?" — rồi so với danh sách của bạn | Viết ra kỳ vọng theo level; định nghĩa "xong"; nói rõ thứ tự ưu tiên và cái gì được phép bỏ | Động viên; tăng số buổi họp | 1–2 tuần |
| **Thiếu Capability** — muốn nhưng chưa làm được | Trì hoãn việc khó; nhận việc rồi im; chất lượng dao động lớn; tránh vùng mới | "Việc nào trong tuần làm em thấy khó nhất, và khó ở chỗ nào?" | Mentoring (chủ đề 2); chia task nhỏ hơn để có tiến bộ thấy được; pairing | Giao thêm việc khó để "thử thách"; feedback về thái độ | 4–10 tuần |
| **Thiếu Autonomy** — không có không gian quyết định | Chờ duyệt mọi thứ; không đề xuất; "cái đó anh quyết đi"; làm đúng phạm vi, không hơn | "Trong tháng qua có quyết định nào em muốn tự quyết mà phải xin phép?" | Tăng mức delegation cho một vùng cụ thể và nói ra (chủ đề 1); giao ownership một module | Cho tham gia thêm họp; hỏi ý kiến nhưng vẫn tự quyết | 2–6 tuần |
| **Thiếu Fairness** — thấy tỉ lệ đóng góp/phần thưởng bất công | So sánh với người khác; nói về lương/title của người khác; giảm nỗ lực có chọn lọc; ngừng giúp đồng đội | "Có điều gì em thấy không công bằng mà em chưa nói không? Anh không hứa sửa được, nhưng anh muốn biết" | Sửa bất công thật (điều chỉnh lương, sửa việc gán công); làm minh bạch tiêu chí; nếu không sửa được thì nói thật lý do và mốc thời gian | Giải thích rằng cảm giác đó không đúng; nói "công ty đang khó" mà không có mốc | Ngay khi sửa; nếu không sửa thì không hồi phục |
| **Thiếu Meaning** — không thấy việc mình làm dẫn tới đâu | "Làm cái này để làm gì"; thờ ơ lịch sự; chất lượng đủ dùng; không quan tâm kết quả sau release | "Em có biết ai đang dùng cái em làm, và nó thay đổi việc gì của họ không?" | Cho tiếp xúc người dùng/khách hàng thật; đưa số liệu tác động vào demo; nối task với kết quả kinh doanh | Slogan về tầm nhìn công ty; team building | 2–8 tuần |

**Bước 2 — Quy trình chẩn đoán, bốn bước.**

1. **Thu tín hiệu trước khi hỏi.** Nghe câu người ta nói (mỗi giả thuyết có ngôn ngữ riêng, xem cột
   triệu chứng), xem hành vi thay đổi từ khi nào, xem có thay đổi ngữ cảnh nào trùng thời điểm (đổi
   project, đổi lead, một lần bị bác bỏ công khai, một đợt tăng lương).
2. **1-1 chẩn đoán, không phải 1-1 giải quyết.** Mục tiêu duy nhất của buổi này là tìm biến nào đang
   thấp. Không đề xuất giải pháp trong buổi đầu — vì giải pháp sai làm người kia đóng lại.
3. **Can thiệp một biến, có mốc.** Nói rõ: đang thử cái gì, trong bao lâu, và sẽ cùng xem lại.
4. **Đo lại sau 4 tuần bằng hành vi, không bằng cảm giác.** Ví dụ: người thiếu Autonomy sau can thiệp
   phải có ít nhất một quyết định tự chốt không xin phép; người thiếu Meaning phải có ít nhất một câu
   hỏi về tác động của tính năng.

**Bước 3 — Phân loại cái mình giải quyết được và cái không.** Đây là bước lead thường bỏ, và việc bỏ
nó dẫn tới hai lỗi đối xứng: hứa cái mình không quyết được, hoặc im lặng về cái mình quyết được.

```
CÁI LEAD QUYẾT ĐƯỢC (làm ngay, trong tuần)
- Phân bổ việc, mức delegation, ownership module
- Ai được trình bày trước stakeholder
- Bỏ ceremony vô nghĩa; giảm thời gian chờ review; sửa CI chậm
- Nói rõ kỳ vọng theo level; công nhận cụ thể và công khai
- Cho tiếp xúc người dùng cuối (kể cả trong ODC: xin một buổi với BA/end user của khách)

CÁI LEAD ẢNH HƯỞNG ĐƯỢC (cần vận động, tính bằng tuần đến tháng)
- Điều chỉnh lương ngoài kỳ (cần dữ liệu thị trường + hồ sơ đóng góp)
- Promotion (cần sponsorship — chủ đề 2)
- Đổi project / rotation
- Ngân sách học tập, conference

CÁI LEAD KHÔNG QUYẾT ĐƯỢC (phải nói thật, không được hứa)
- Dải lương toàn công ty; chính sách tăng lương định kỳ
- Bản chất hợp đồng ODC (scope do khách quyết)
- Chiến lược sản phẩm; việc công ty có gọi được vốn hay không
Với nhóm này, câu đúng là: "Cái này anh không quyết được. Anh sẽ nói với ai, khi nào,
và anh sẽ báo lại em kết quả kể cả khi kết quả là không." Câu sai là im lặng, và câu
tệ nhất là "để anh xem" rồi không quay lại.
```

### Trade-off

| Trục | Nghiêng về bên A khi | Nghiêng về bên B khi |
|---|---|---|
| **Tăng lương giữ người (A) vs giữ equity nội bộ (B)** | Người đó thực sự ở mức đóng góp cao hơn mức lương hiện tại; sửa được bằng dữ liệu và giải thích được nếu người khác biết | Không giải thích được bằng tiêu chí; sẽ tạo ra ba trường hợp tương tự trong hai tháng; vấn đề thật không phải lương (khi đó tiền chỉ trì hoãn) |
| **Tự chủ tối đa (A) vs tính nhất quán hệ thống (B)** | Người có năng lực và blast radius giới hạn; mục tiêu là giữ và phát triển | Hệ thống cần chuẩn chung; nhiều team dùng chung; rủi ro compliance |
| **Giao việc thú vị theo mong muốn (A) vs giao việc theo nhu cầu hệ thống (B)** | Có nhiều người muốn cùng loại việc và có thể luân phiên; người đó đang ở nguy cơ rời đi | Có việc không hào nhoáng nhưng bắt buộc; nếu luôn tránh, nó dồn vào một người và người đó mới là người sắp nghỉ |
| **Minh bạch dải lương/tiêu chí (A) vs giữ kín (B)** | Muốn giảm cảm giác bất công do suy đoán; tiêu chí thật sự tồn tại và bảo vệ được | Chưa có ladder, các trường hợp lịch sử không giải thích được — minh bạch lúc này sẽ phơi ra bất công thật mà chưa có ngân sách sửa (khi đó việc đúng là sửa trước, minh bạch sau, có mốc) |

Ai chịu phần mất: khi chọn giữ kín và không sửa, phần mất do những người bị trả thấp chịu, và họ trả
bằng cách rời đi ở thời điểm bất lợi nhất cho tổ chức. Khi chọn tăng lương cho một người mà không xử
lý tiêu chí, phần mất do những người còn lại chịu, và nó xuất hiện muộn hơn khoảng một quý — nên
thường bị quy sai nguyên nhân.

### Real-world Scenarios

**Tình huống A — Tuấn nhận offer cao hơn 40%.** 14 tháng sau tình huống ở chủ đề 1, Tuấn giờ là
người sở hữu toàn bộ luồng đối soát của bốn đối tác. Một công ty khác mời với mức lương cao hơn 40%.
Tuấn nói với Minh — điều này tự nó là tín hiệu tốt, vì nghĩa là Tuấn còn muốn được thuyết phục.

Script — **phiên bản nói sai**:

> **Minh:** "Ơ, sao em không nói với anh sớm? 40% thì cao thật đấy nhưng em suy nghĩ kỹ chưa, sang
> đó chưa biết môi trường thế nào đâu, nghe nói công ty đó làm việc rất áp lực. Ở đây em có anh em,
> có team, mọi người quen nhau rồi. Để anh xin cho em tăng lương, anh nói với chị Trang. Em đừng
> quyết gì lúc này nhé."

Bốn lỗi. (1) "Sao không nói sớm" đặt Tuấn vào thế phải xin lỗi vì đã có lựa chọn — lần sau Tuấn sẽ
không nói, sẽ chỉ nộp đơn. (2) Hạ thấp lựa chọn kia bằng tin nghe được, không bằng dữ kiện. (3) Dùng
quan hệ tình cảm làm lập luận giữ người — nó tạo cảm giác bị mắc nợ, và người ta trả nợ bằng cách ở
thêm ba tháng rồi đi. (4) Nhảy ngay sang tiền, tức là chấp nhận giả định rằng đây là bài toán tiền,
trong khi chưa chẩn đoán.

Script — **phiên bản nói đúng**:

> **Minh:** "Cảm ơn em nói với anh, anh biết nói ra không dễ. Anh muốn hỏi trước khi bàn về tiền, và
> anh hỏi thật: nếu bên đó trả đúng bằng mức hiện tại của em, em còn muốn sang không?"
>
> **Tuấn:** "...Em nghĩ là vẫn hơi muốn. Bên đó làm hệ thống realtime, quy mô lớn hơn."
>
> **Minh:** "Vậy phần lớn không phải tiền. Em nói cụ thể hơn giúp anh: cái em muốn là quy mô, hay là
> loại bài toán, hay là học được thứ mới?"
>
> **Tuấn:** "Loại bài toán. Em làm đối soát 14 tháng rồi, giờ em làm nhanh nhưng em không học thêm
> gì. Em thấy mình đang thành người-chuyên-đối-soát."
>
> **Minh:** "Đó là một chẩn đoán đúng, và một phần là lỗi của anh — anh giao đối soát cho em vì em
> làm tốt, đó là tối ưu cho anh, không phải cho em. Giờ anh nói thẳng ba việc.
>
> Một, về tiền: anh không giữ được 40%. Mức anh có thể đấu tranh, với hồ sơ 14 tháng của em, là
> khoảng 15–18% và anh cần ba tuần để đi qua chị Trang và HR. Anh sẽ nói kết quả kể cả nếu bị từ chối.
> Anh không muốn em ở lại chỉ vì tiền, vì như vậy sáu tháng nữa mình lại ngồi đây.
>
> Hai, về loại việc: Q3 mình phải làm lại luồng xử lý giao dịch realtime, phần này là bài toán khó
> nhất trong hệ thống. Anh muốn em là người thiết kế, không phải người tham gia. Anh nói với em điều
> này hôm nay, chứ không phải để dành làm quân bài — nếu em vẫn đi thì anh vẫn cần một người làm nó.
>
> Ba, về việc em thành người-chuyên-đối-soát: anh cần chuyển đối soát sang người khác trong hai tháng
> và em là người bàn giao. Việc này anh làm bất kể em ở hay đi, vì nó là lỗi thiết kế của anh.
>
> Và nói thật: nếu sau khi biết cả ba việc này em vẫn chọn sang đó, anh nghĩ đó là lựa chọn hợp lý và
> anh sẽ viết thư giới thiệu. Anh không coi đó là em bỏ team."

Điểm quan trọng về mặt cơ chế: câu hỏi "nếu bằng lương thì em còn muốn sang không" là công cụ chẩn
đoán rẻ nhất và mạnh nhất trong toàn bộ tình huống. Nếu câu trả lời là "không", đây là bài toán
Fairness/hygiene và có thể xử lý bằng tiền cùng với việc giải thích tiêu chí. Nếu câu trả lời là "vẫn
muốn", tiền chỉ mua được thêm một hai quý và mọi đồng chi ra đều bị phân bổ sai.

**Ba góc nhìn về cùng sự việc:**

- **Nhìn từ IC (Tuấn).** "Em không mặc cả. Em nói ra vì em muốn biết ở đây còn gì cho em không. Cái
  em sợ nhất không phải lương thấp, mà là ba năm nữa em vẫn giỏi đúng một thứ và thị trường không cần
  thứ đó nữa. Nếu anh Minh chỉ nói về tiền, em hiểu là ở đây không có câu trả lời cho câu hỏi thật
  của em. Và nếu anh ấy khuyên em đừng đi vì 'tình cảm anh em', em sẽ đi và sẽ không nói lý do thật."
- **Nhìn từ Tech Lead (Minh).** "Tôi đang nhìn một sự thất bại thiết kế của chính tôi: tôi tối ưu
  ngắn hạn bằng cách để người làm tốt nhất làm mãi một việc, và bây giờ tôi phải trả giá đúng lúc
  đắt nhất. Tôi có hai bài toán tách rời — giữ người này, và sửa cơ chế đã tạo ra tình huống này. Nếu
  tôi chỉ làm bài toán một, sáu tháng nữa sẽ có Tuấn thứ hai. Việc khó nhất là không dùng quan hệ cá
  nhân làm đòn giữ người, vì nó có tác dụng ngắn hạn và phá huỷ độ tin cậy của mọi cuộc nói chuyện
  sau này."
- **Nhìn từ Manager (Trang/EM).** "Tôi cần biết đây là trường hợp đơn lẻ hay là tín hiệu hệ thống. Một
  người nhận offer +40% có thể là thị trường nóng; ba người trong hai quý là dải lương của chúng tôi
  đã lệch và tôi phải mang dữ liệu đi xin ngân sách. Tôi cũng cần Minh nói thật con số mình có thể
  hứa, vì cái tệ nhất là Minh hứa 20% với Tuấn rồi tôi chỉ duyệt được 10% — lúc đó chúng tôi mất cả
  người lẫn uy tín. Và nếu buộc phải để Tuấn đi, tôi cần biết trước 6 tuần để lo bàn giao, chứ không
  phải biết cùng lúc với đơn xin nghỉ."

Cùng một sự kiện: IC đọc là **câu hỏi về tương lai năng lực của mình**, Tech Lead đọc là **hậu quả
của một lỗi phân bổ việc**, Manager đọc là **dữ liệu về dải lương và rủi ro bàn giao**. Cách xử lý
sai phổ biến nhất là cả ba tầng đều đọc nó là bài toán tiền.

**Tình huống B — thiếu Meaning trong ODC.** Trở lại Duy ở chủ đề 2: bốn năm, ba năm gần nhất viết
CRUD cho hệ thống nội bộ của khách Nhật, chưa từng thấy một người dùng cuối. Đây là dạng thiếu Meaning
có tính cấu trúc, phổ biến trong ODC và thường bị chẩn đoán sai thành thiếu thái độ.

Can thiệp của Linh, theo thứ tự chi phí tăng dần: (1) xin khách 45 phút cho một buổi để hai người ở
bộ phận nghiệp vụ của khách demo cách họ dùng hệ thống hằng ngày — chi phí gần bằng 0, và đây là can
thiệp có hiệu lực cao nhất; (2) yêu cầu khách cung cấp một vài số liệu sử dụng (số đơn xử lý mỗi
ngày, thời gian tiết kiệm được) và đưa vào đầu mỗi buổi sprint review; (3) đổi cách viết ticket: mỗi
ticket có một dòng "ai gặp vấn đề gì" thay vì chỉ mô tả kỹ thuật; (4) rotation — Duy sang một dự án
khác trong sáu tháng. Sau ba tuần thực hiện (1) và (3), Duy tự đề xuất hai thay đổi UX mà khách chấp
nhận — đây là nỗ lực tuỳ ý quay lại, và nó là bằng chứng rằng chẩn đoán ban đầu ("thiếu kỹ năng phân
tích") đã sai.

**Tình huống C — đối chiếu lương giữa bạn bè.** Vy, Junior 14 tháng, biết một bạn cùng khoá lương cao
hơn 30% ở một công ty khác, và trong hai tuần chất lượng làm việc giảm rõ. Cách xử lý sai: giải thích
rằng "so sánh như vậy không đúng đâu" — điều này bác bỏ trải nghiệm của người đối diện và kết thúc cuộc
đối thoại. Cách xử lý được: đưa so sánh về đầy đủ chiều ("bạn em làm scope gì, ai review code của bạn
ấy, sáu tháng qua bạn ấy học được gì"), rồi nói thật về dải lương và tiêu chí lên level tiếp theo với
mốc thời gian cụ thể. Nếu sau khi có đầy đủ dữ kiện mà mức của Vy vẫn thấp so với thị trường, đó là
bất công thật và phải sửa bằng tiền — không sửa được bằng giải thích.

### Best Practices

- **Chẩn đoán trước khi can thiệp, và can thiệp một biến một lần.** Lý do: can thiệp vào biến sai
  chứng minh cho người kia rằng bạn không hiểu vấn đề của họ, và điều đó tốn nhiều hơn số tiền hay
  thời gian bạn vừa bỏ ra.
- **Xoá các thứ chặn cảm giác tiến bộ trước khi nghĩ đến phần thưởng.** Lý do: progress principle —
  CI 40 phút, PR chờ 3 ngày, staging hỏng một tuần ăn vào động lực mỗi ngày, đều đặn, và chúng thuộc
  quyền quyết định của lead.
- **Công nhận cụ thể, gần thời điểm, và trước mặt người có ảnh hưởng.** Lý do: "em làm tốt lắm" không
  mang thông tin; "cách em tách phần rule đối soát ra khiến ba đối tác sau tích hợp mất một tuần thay
  vì ba" mang thông tin, và nói câu đó trước mặt EM là hành động sponsorship.
- **Nói thật về cái mình không quyết được, kèm mốc thời gian sẽ báo lại.** Lý do: sự bất định gây hại
  hơn tin xấu; và một lời hứa mơ hồ tiêu tốn uy tín cho mọi cuộc nói chuyện sau.
- **Hỏi "nếu lương bằng nhau thì em còn muốn đi không" khi có offer.** Lý do: đây là phép chẩn đoán
  tách hygiene khỏi motivator trong một câu.
- **Luân phiên việc không hào nhoáng.** Lý do: nếu flaky test, on-call, và viết runbook luôn thuộc
  cùng một người, người đó là người sắp nghỉ, và họ nghỉ vì Fairness chứ không vì công việc nặng.
- **Theo dõi regretted attrition, không phải attrition chung.** Lý do: tỉ lệ nghỉ chung có thể tốt
  trong khi bạn đang mất đúng những người tạo ra phần lớn giá trị.

### Anti-patterns

- **Dùng team building và ăn uống để giải quyết vấn đề cấu trúc.** Team mất động lực vì không có
  Autonomy, vì phân bổ việc bất công, vì ba tháng không thấy sản phẩm mình làm đi tới đâu — và can
  thiệp là một chuyến team building hai ngày ở Đà Lạt. Cơ chế phá hoại: nó tạo ra một khoảng lệch
  công khai giữa vấn đề và phản ứng, và khoảng lệch đó là bằng chứng cho team rằng tổ chức không nhìn
  thấy vấn đề thật hoặc thấy mà không xử lý. Sau chuyến đi, mức engagement thường thấp hơn trước, vì
  hy vọng đã bị thử và thất bại. Dấu hiệu sớm: ngân sách team building tăng trong khi không có một
  thay đổi nào về cách phân bổ quyền quyết định. Điều này không có nghĩa là hoạt động chung vô ích —
  chúng có tác dụng thật lên quan hệ (relatedness) khi các biến cấu trúc đã ổn; chúng chỉ không thay
  thế được việc sửa cấu trúc.
- **Gắn thưởng vào chỉ số đếm được của hành vi nội tại.** Thưởng theo số PR review, số dòng tài liệu,
  số commit. Cơ chế: crowding-out cộng với Goodhart — chỉ số tăng, giá trị giảm, và động lực nội tại
  không quay lại sau khi bỏ thưởng. Dấu hiệu sớm: review có nội dung "LGTM" tăng đột biến.
- **Xem lương là công cụ tạo động lực.** Cơ chế: lương là hygiene; nó xoá bất mãn trong 2–4 tháng rồi
  trở thành mức tham chiếu mới. Dấu hiệu sớm: cùng một người xin tăng lương hai lần trong một năm mà
  không có thay đổi nào về scope công việc.
- **Chẩn đoán mất động lực thành vấn đề thái độ.** Cơ chế: quy nguyên nhân vào tính cách (attribution
  error) dẫn tới can thiệp vào tính cách, loại can thiệp không có hiệu lực và đồng thời tạo ra sự
  phòng vệ. Dấu hiệu sớm: trong ghi chép 1-1 của lead xuất hiện các từ "chưa nhiệt tình", "thiếu chủ
  động" mà không có mô tả hành vi cụ thể.
- **Giữ người bằng cảm giác mắc nợ.** "Anh đã tạo điều kiện cho em rất nhiều." Cơ chế: tạo nghĩa vụ
  đạo đức thay cho lý do hợp lý; người ta ở thêm một thời gian ngắn với mức engagement thấp, và mọi
  cuộc nói chuyện thật sau đó không còn xảy ra. Dấu hiệu sớm: lead nhắc lại những gì mình đã làm cho
  người khác.
- **Tăng lương im lặng cho một người trong team.** Cơ chế: thông tin luôn rò rỉ, và khi nó rò rỉ mà
  không có tiêu chí kèm theo, ba người khác kết luận rằng phần thưởng phụ thuộc vào việc biết mặc cả.
  Dấu hiệu sớm: trong hai tháng sau đó xuất hiện thêm hai người xin nói chuyện về lương.

### Khi nào KHÔNG nên áp dụng

- **Khi vấn đề thật là fit, không phải động lực.** Một người giỏi ở sai vị trí sẽ mất động lực, và
  mọi can thiệp vào năm biến kia đều không đủ. Can thiệp đúng là đổi vai hoặc đổi team, và trong một
  số trường hợp là chia tay tử tế. Kéo dài việc "tăng động lực" ở đây gây hại cho cả hai phía.
- **Khi bất công là thật và bạn chưa sửa được.** Nếu một người đang bị trả thấp hơn thị trường 30%,
  mọi can thiệp về Meaning và Autonomy sẽ bị đọc là sự lảng tránh — và đọc như vậy là đúng. Thứ tự
  bắt buộc: xử lý hygiene trước, hoặc nói thật rằng chưa xử lý được và tại sao, rồi mới nói đến phần
  còn lại.
- **Khi người đó đã ra quyết định rời đi.** Sau khi một người đã ký offer khác, các can thiệp động
  lực trở thành áp lực. Việc đúng là bàn giao tốt, giữ quan hệ, và học từ nguyên nhân — một người rời
  đi trong sự tôn trọng là kênh tuyển dụng và là nguồn thông tin trung thực nhất bạn có.
- **Khi nguyên nhân nằm ngoài phạm vi công việc.** Vấn đề sức khoẻ, gia đình, tài chính cá nhân. Vai
  của lead ở đây là linh hoạt về mặt vận hành (giảm tải, đổi giờ, hỗ trợ nghỉ) và kết nối tới nguồn
  hỗ trợ đúng, không phải chẩn đoán và can thiệp. Vượt biên vai trò ở đây tạo ra rủi ro thật cho cả
  hai.
- **Trong giai đoạn khủng hoảng ngắn có mục tiêu rõ.** Trước một đợt audit hoặc một go-live hợp đồng,
  điều tạo ra nỗ lực là mục tiêu rõ, thời hạn hữu hạn có ngày kết thúc thật, và sự hiện diện của
  lead — không phải một chương trình engagement. Nhưng điều kiện biên là **hữu hạn**: nếu "giai đoạn
  đặc biệt" kéo dài quá hai tháng, nó không còn là khủng hoảng, nó là cách tổ chức vận hành, và động
  lực sẽ sụp theo cách không hồi phục được.
- **Khi bạn không có ý định thay đổi gì sau khi hỏi.** Chạy survey engagement rồi không công bố kết
  quả và không thay đổi gì tệ hơn không chạy: nó dạy cả team rằng việc nói ra là vô ích, và làm hỏng
  kênh tín hiệu cho lần sau.

---

## 5. Conflict Resolution

### Problem Statement

Quay lại team logistics ở đầu chương. Hai Senior — Dũng và Phúc — ba tháng không nói chuyện trực
tiếp. Nguồn gốc: một tranh luận về việc có tách service quản lý vận đơn ra khỏi monolith hay không.
Tranh luận đó không bao giờ được kết thúc; nó chỉ dừng lại. Từ đó, các dấu hiệu sau tích tụ:

- **Quyết định bị treo.** Mọi thay đổi chạm vào ranh giới module vận đơn đều phải "chờ thống nhất", và
  việc thống nhất không bao giờ diễn ra. Đếm được: 4 ticket đã nằm trong backlog trên 8 tuần với lý
  do duy nhất là chưa chốt hướng.
- **Xung đột di chuyển vào code review.** Comment trong PR của nhau dài hơn, lạnh hơn, nhiều câu hỏi
  tu từ hơn ("bạn có chắc cách này scale được không?"). Thời gian review giữa hai người này gấp ba lần
  giữa các cặp khác.
- **Kênh ngầm.** Cả hai không phản đối trong họp. Cả hai nói với người thứ ba sau họp. Team dần chia
  thành hai nhóm không chính thức, và người mới vào học rất nhanh rằng "muốn làm phần vận đơn thì hỏi
  anh Dũng, đừng hỏi anh Phúc".
- **Quyết định lại nhiều lần.** Cùng một chủ đề được đưa ra bàn lần thứ tư, mỗi lần bắt đầu lại từ
  đầu. Đây là dấu hiệu chắc chắn nhất của một xung đột chưa được chốt: chi phí không nằm ở cuộc tranh
  luận, nó nằm ở số lần tranh luận lặp lại.
- **Tín hiệu kỹ thuật bị chặn.** Phúc thấy một rủi ro trong thiết kế của Dũng và không nói, vì nói sẽ
  bị đọc là tiếp tục cuộc chiến cũ. Đây là chi phí đắt nhất và khó thấy nhất.
- **Escalation lặp lại từ cùng một cặp người.** Nếu Tech Lead phải phân xử giữa cùng hai người mỗi hai
  tuần, vấn đề không còn là nội dung tranh luận.

Nếu không xử lý, đường đi tiếp theo khá cố định: một trong hai người rời đi (thường là người ít quyền
hơn hoặc người có lựa chọn tốt hơn trên thị trường), tri thức về module đi theo, và team học được rằng
cách xử lý bất đồng là chờ cho một bên bỏ đi.

### First Principles

**Cơ chế một: xung đột là tín hiệu, không phải lỗi.** Bất đồng kỹ thuật giữa hai người có năng lực
gần như luôn có nghĩa là một trong ba điều: họ đang tối ưu cho hai hàm mục tiêu khác nhau (một người
tối ưu thời gian ra thị trường, người kia tối ưu chi phí bảo trì hai năm tới); họ có hai tập thông
tin khác nhau (một người đã trải qua một lần tách service thất bại, người kia chưa); hoặc họ đang
tranh luận với hai định nghĩa khác nhau về vấn đề. Cả ba đều là **thông tin có giá trị**, và cả ba
đều bị mất nếu xung đột bị dập bằng quyền lực hoặc bị tránh. Hệ quả đối xứng cũng đúng và ít được
nói: **một team không có task conflict nào không phải team hoà thuận** — nó là team thiếu
Psychological Safety, thiếu sự quan tâm, hoặc thiếu người có đủ năng lực để phản biện.

**Cơ chế hai: ba loại xung đột có ba dấu hiệu và ba cách xử lý khác nhau.** Phân loại này (Jehn và
các nghiên cứu sau) là công cụ chẩn đoán quan trọng nhất trong chủ đề:

| Loại | Về cái gì | Tác động | Xử lý đúng |
|---|---|---|---|
| **Task conflict** | Nội dung: kiến trúc, ưu tiên, cách tiếp cận | Ở mức vừa phải và trong điều kiện có Psychological Safety, nó **tăng** chất lượng quyết định — vì nó buộc các giả định hiện ra | Khuyến khích, và **chốt trong thời hạn**. Cái hại không phải nó tồn tại, mà nó kéo dài |
| **Process conflict** | Ai làm cái gì, ra quyết định thế nào, ai được quyết | Gần như luôn có hại nếu kéo dài; nó tiêu năng lượng mà không tạo thông tin về nội dung | Chốt bằng quy tắc viết ra (ai quyết cái gì), không bằng thảo luận từng lần |
| **Relationship conflict** | Con người: động cơ, tính cách, sự tôn trọng | Luôn có hại; nó làm giảm khả năng tiếp nhận thông tin từ đối phương, kể cả thông tin đúng | Không giải quyết bằng lập luận kỹ thuật. Cần can thiệp riêng, thường cần người thứ ba |

**Cơ chế ba — cơ chế quan trọng nhất của chủ đề này: task conflict không được chốt sẽ chuyển hoá
thành relationship conflict.** Đường đi có bốn bước và nó gần như tự động:

```
1. Bất đồng nội dung (task):    "Tôi nghĩ tách service tốt hơn."
2. Không chốt, lặp lại:         Lần 2, lần 3, lần 4 cùng chủ đề, không kết luận.
3. Cần lời giải thích:           Não người phải giải thích vì sao người kia vẫn không đồng ý
                                 với lập luận rõ ràng như vậy. Lời giải thích rẻ nhất và có
                                 sẵn nhất là quy về con người: "anh ấy bảo thủ", "anh ấy chỉ
                                 muốn chứng tỏ", "anh ấy sợ mất phần việc của mình".
                                 (Đây là fundamental attribution error: quy hành vi của người
                                 khác về tính cách, quy hành vi của mình về hoàn cảnh.)
4. Relationship conflict:        Từ lúc này, mọi lập luận kỹ thuật của đối phương đều bị đọc
                                 qua bộ lọc động cơ, và dữ liệu không còn thay đổi được kết luận.
```

Suy ra một nguyên tắc vận hành cứng: **biến điều khiển không phải cường độ tranh luận, mà là thời
gian tới lúc chốt.** Một tranh luận gay gắt được chốt trong ba ngày để lại một team mạnh hơn. Một
tranh luận lịch sự kéo dài ba tháng để lại hai người không nói chuyện với nhau. Đây là lý do sai lầm
lớn nhất của lead trong xung đột kỹ thuật không phải xử lý thô, mà là **trì hoãn**.

**Cơ chế bốn: chi phí của xung đột không giải quyết là chi phí che thông tin.** Khi hai người ở trạng
thái relationship conflict, mỗi người bắt đầu lọc thông tin gửi cho người kia: không nêu rủi ro mình
thấy trong thiết kế của họ (sẽ bị đọc là công kích), không hỏi khi không hiểu (sẽ bị đọc là yếu),
không chia sẻ context mình có. Với hai người sở hữu hai module ghép nhau, đây là nguồn sự cố có xác
suất cao và nguyên nhân không bao giờ được ghi vào postmortem.

**Cơ chế năm — đặc thù bối cảnh Việt Nam.** Trong một môi trường coi trọng việc giữ mặt và tránh đối
đầu trực diện, xung đột không biến mất mà **đổi kênh**. Ba biểu hiện cụ thể và phải được đọc đúng:
(a) **im lặng trong họp không phải đồng thuận** — nó thường là sự trì hoãn phản đối tới thời điểm ít
rủi ro hơn, và thời điểm đó thường là lúc triển khai; (b) **phản đối được gửi qua người thứ ba** hoặc
qua kênh riêng với lead, khiến lead nhận được hai phiên bản không đối chất được với nhau; (c) **tuổi
tác và thâm niên hoạt động như một trục uy tín độc lập với năng lực kỹ thuật**, nên một Mid 26 tuổi có
lập luận đúng có thể không nêu ra được trước một người 40 tuổi, và việc không nêu ra đó bị lead đọc là
"không có ý kiến". Hệ quả thiết kế: nếu lead chỉ dựa vào việc "ai không đồng ý thì nói" thì lead sẽ
vận hành trên dữ liệu thiếu một cách hệ thống. Cần cơ chế chủ động lấy phản đối (viết trước khi nói,
hỏi từng người, hỏi riêng rồi tổng hợp công khai).

### Mental Model

**Position vs Interest.** Từ Fisher và Ury: position là phương án người ta phát ngôn ("tách service"),
interest là điều họ thực sự cần ("tôi không muốn mỗi lần deploy phải kéo cả hệ thống và chịu rủi ro
cho code của người khác"). Hai position xung đột nhau thường ứng với hai interest **không** xung đột
nhau — và khi hiện ra ở mức interest, thường tồn tại phương án thứ ba mà không ai đề xuất ban đầu. Kỹ
thuật vận hành: cấm phát ngôn phương án trong 20 phút đầu, chỉ cho phát ngôn tiêu chí và ràng buộc.

**Xung đột kiến trúc thường là một quyết định chưa có chủ.** Nếu không ai được chỉ định là người
quyết, tranh luận sẽ chạy đến khi một bên kiệt sức. Rất nhiều xung đột "về con người" biến mất sau
khi trả lời được một câu hành chính: **ai là người có accountability cho kết quả của quyết định này?**
Người đó quyết. Đây là lý do Conflict Resolution và Decision Making không tách rời được
(`04-decision-making.md`).

**Thomas–Kilmann: năm chế độ, không có chế độ nào luôn đúng.** Trục dọc là mức độ theo đuổi mục tiêu
của mình (assertiveness), trục ngang là mức độ hợp tác với mục tiêu người khác (cooperativeness).

| Chế độ | Dùng khi | Chi phí / rủi ro khi dùng sai |
|---|---|---|
| **Competing** (áp đặt) | Incident đang diễn ra; vấn đề an toàn/compliance không thương lượng; quyết định phải có trong vài giờ và bạn có accountability | Nếu dùng thường xuyên: người ta ngừng nêu phản biện, và bạn mất kênh thông tin. Đắt nhất ở chỗ nó âm thầm |
| **Collaborating** (cùng tìm phương án thứ ba) | Quyết định quan trọng, có thời gian, cả hai bên có thông tin thật cần được tích hợp | Tốn nhiều thời gian; dùng cho vấn đề nhỏ là lãng phí và làm team thấy mọi việc đều nặng nề |
| **Compromising** (mỗi bên nhường một nửa) | Cần một quyết định đủ tốt nhanh; hai bên có quyền lực tương đương; vấn đề không ở tầng nguyên tắc | Trong kiến trúc, thoả hiệp giữa hai thiết kế thường tạo ra phương án tệ hơn cả hai — "nửa monolith nửa service" là ví dụ điển hình |
| **Avoiding** (tránh) | Vấn đề sẽ tự hết; chi phí bàn cao hơn giá trị; đang có cảm xúc cao và cần hoãn vài giờ có mục đích | Trở thành mặc định thì tạo ra đúng tình huống Dũng–Phúc. Tránh có chủ đích và có ngày quay lại thì được; tránh vì không thoải mái thì không |
| **Accommodating** (nhường) | Bạn sai; vấn đề quan trọng với họ hơn với bạn; bạn cần đầu tư vào quan hệ cho một cuộc thương lượng lớn hơn sắp tới | Nhường liên tục tạo ra kỳ vọng và làm bạn mất tư cách trong lần cần giữ. Nếu bạn là lead, nhường sai sẽ được đọc là chuẩn mực mới |

### Practical Framework

**Quy trình hoà giải bất đồng kiến trúc, năm bước.** Áp dụng cho tranh luận kỹ thuật giữa hai người
có năng lực, cả hai đều có lý — tức là trường hợp phổ biến nhất.

**Bước 1 — Tách tiêu chí ra khỏi phương án.** Đây là bước quyết định toàn bộ chất lượng của buổi hoà
giải, và là bước hầu như luôn bị bỏ. Trước buổi họp, mỗi bên viết riêng (không thảo luận) hai thứ:
danh sách tiêu chí đánh giá theo thứ tự ưu tiên, và ràng buộc không được vi phạm. **Không viết phương
án.** Trong buổi, phần đầu chỉ làm một việc: hợp nhất hai danh sách tiêu chí thành một danh sách có
thứ tự. Lý do cơ học: khi hai người đã phát ngôn phương án, mọi lập luận sau đó trở thành bảo vệ danh
tính; khi họ phát ngôn tiêu chí trước, họ thường phát hiện ra mình đồng ý 70% và bất đồng thật nằm ở
**thứ tự ưu tiên giữa hai tiêu chí**, chứ không nằm ở kỹ thuật. Bất đồng về thứ tự ưu tiên là bất
đồng về mục tiêu kinh doanh — và loại bất đồng đó không giải được bằng lập luận kỹ thuật, nó cần PO
hoặc người có accountability trả lời.

**Bước 2 — Đưa về dữ liệu, và định nghĩa thí nghiệm rẻ nhất.** Với mỗi bất đồng còn lại, hỏi hai câu:
*số nào sẽ làm bạn đổi ý?* và *cách rẻ nhất để có số đó là gì?* Rất nhiều tranh luận kiến trúc chạy
trên các giả định định lượng chưa được kiểm tra ("deploy sẽ chậm", "team không vận hành được", "traffic
sẽ tăng 5 lần"). Một spike hai ngày hoặc một lần đo trên dữ liệu thật thường kết thúc tranh luận mà
không cần ai nhường ai — và quan trọng hơn, nó chuyển cuộc chơi từ tranh luận sang tìm hiểu, tức là
đổi cả quan hệ giữa hai người.

**Bước 3 — Tìm điều kiện mà mỗi phương án đúng.** Câu hỏi: "cái gì phải đúng thì phương án của anh là
lựa chọn tốt hơn?" Đặt câu này cho cả hai bên, và bắt mỗi người trả lời cho **phương án của người
kia**. Tác dụng: nó buộc mỗi người xây dựng mô hình tốt nhất của lập luận đối phương (steelman), và nó
thường lộ ra rằng cả hai đúng trong hai kịch bản tương lai khác nhau. Khi đó câu hỏi thật không còn là
"ai đúng" mà là "kịch bản nào có xác suất cao hơn, và phương án nào sai ít tốn kém hơn nếu ta đoán sai
kịch bản".

**Bước 4 — Nếu không hội tụ, chốt bằng người có accountability.** Đặt trần thời gian trước khi bắt đầu
(ví dụ: một buổi 90 phút, cộng tối đa một spike hai ngày). Nếu hết trần mà không hội tụ, người có
accountability cho kết quả quyết định — và phải **nói rõ đây là quyết định do thiếu hội tụ, không phải
kết luận rằng bên kia sai**. Ghi lại quan điểm phản đối vào biên bản, có tên người, kèm điều kiện xem
lại. Việc ghi dissent không phải hình thức: nó cho người không được chọn một cách giữ được tư cách kỹ
thuật của mình mà không cần tiếp tục chiến đấu, và nó tạo ra một cơ chế đúng để xem lại nếu tương lai
chứng minh họ đúng.

**Bước 5 — Cam kết công khai.** Sau khi chốt, cả hai bên phát ngôn cam kết thực thi trước team
("disagree and commit", và phần commit phải được nói thành lời, không mặc định). Lead phải nói rõ một
điều: từ giờ, hành vi phá hoại ngầm phương án đã chốt là vấn đề nghiêm trọng hơn bất kỳ bất đồng kỹ
thuật nào.

**Template — biên bản hoà giải quyết định kiến trúc.** Dán vào repo cạnh ADR.

```
BIÊN BẢN GIẢI QUYẾT BẤT ĐỒNG KIẾN TRÚC
ID: ADR-conflict-014        Ngày: 12/08        Người điều phối: Nam (Tech Lead)
Người có accountability cho kết quả: Nam
Tham gia: Dũng (Senior BE), Phúc (Senior BE), Ngọc (Head of Eng — chỉ dự bước 4)

1. QUYẾT ĐỊNH CẦN RA (một câu, ở dạng câu hỏi đóng)
   Trong 2 quý tới, module vận đơn có được tách thành service riêng hay không?

2. TIÊU CHÍ THỐNG NHẤT (hợp nhất từ hai danh sách viết riêng, theo thứ tự ưu tiên)
   T1. Không làm chậm phát hành tính năng cam kết với đối tác trong Q4  [cả hai đồng ý #1]
   T2. Giảm rủi ro một lỗi ở module vận đơn làm sập toàn hệ thống
   T3. Chi phí vận hành tăng thêm <= 15 triệu/tháng
   T4. Vận hành được với đội hiện tại (2 người biết Kubernetes)
   ĐIỂM BẤT ĐỒNG THẬT: thứ tự giữa T1 và T2. Dũng đặt T1 trước, Phúc đặt T2 trước.
   -> Đây là bất đồng về ưu tiên kinh doanh, không phải kỹ thuật. Đã hỏi Ngọc: T1 trước
      trong Q4, T2 trở thành ưu tiên #1 từ Q1 năm sau.

3. GIẢ ĐỊNH ĐỊNH LƯỢNG VÀ CÁCH KIỂM TRA
   G1 (Phúc): "lỗi vận đơn đã gây downtime toàn hệ thống"
      -> Kiểm tra: đọc lại 6 incident 12 tháng qua. KẾT QUẢ: 2/6 có nguồn ở vận đơn,
         cả 2 lần sập toàn hệ thống. Giả định ĐÚNG.
   G2 (Dũng): "tách service sẽ làm chậm phát hành Q4 khoảng 3-4 tuần"
      -> Kiểm tra: spike 2 ngày dựng khung service + đo pipeline. KẾT QUẢ: ước lượng
         5-6 tuần, cao hơn Dũng nói. Giả định ĐÚNG và mạnh hơn dự kiến.
   G3 (Dũng): "team không vận hành được thêm một service"
      -> KẾT QUẢ: có 2 người vận hành được, đủ nhưng bus factor = 2. Giả định MỘT PHẦN.

4. ĐIỀU KIỆN MỖI PHƯƠNG ÁN ĐÚNG (mỗi người viết cho phương án của người kia)
   Tách service đúng nếu: tần suất lỗi vận đơn tiếp tục ở mức hiện tại VÀ có >= 6 tuần slack
   Giữ monolith đúng nếu: cô lập được lỗi vận đơn bằng cách khác VÀ Q4 là ràng buộc cứng

5. QUYẾT ĐỊNH
   Không tách trong Q4. Thay vào đó: cô lập module vận đơn trong monolith (bulkhead —
   thread pool riêng, circuit breaker, timeout riêng) để đạt T2 ở mức 60-70% với 1 tuần
   công. Tách service đưa vào kế hoạch Q1, Phúc là người thiết kế và sở hữu.
   Loại quyết định: hội tụ được (không phải quyết định do thiếu hội tụ)

6. QUAN ĐIỂM PHẢN ĐỐI ĐƯỢC GHI NHẬN
   Phúc: "bulkhead chỉ giảm chứ không xoá rủi ro chia sẻ DB; nếu vận đơn khoá bảng thì
   toàn hệ thống vẫn ảnh hưởng." — Ghi nhận là đúng, không được xử lý trong phạm vi này.

7. ĐIỀU KIỆN XEM LẠI (định lượng, có ngày)
   - Nếu có >= 1 incident do vận đơn gây ảnh hưởng ngoài module trong Q4 -> họp lại ngay
   - 15/01: xem lại bất kể có sự cố hay không, Phúc trình kế hoạch tách
   - Nếu p99 của vận đơn > 800ms trong 3 ngày liên tiếp -> họp lại

8. CAM KẾT THỰC THI (phát ngôn trước team, ghi tên)
   Dũng: thực thi bulkhead, xong 26/08. Phúc: review thiết kế bulkhead và không phản
   đối phương án này ngoài kênh chính thức; sở hữu kế hoạch tách từ Q1.
```

**Leo thang (escalation) đúng cách.** Escalation bị coi là hành vi xấu ở nhiều tổ chức Việt Nam ("báo
cáo lên trên"), nên nó bị làm muộn và làm sai. Ba điều kiện để escalation là đúng: (1) đã thử giải
quyết ở tầng của mình và có bằng chứng cụ thể về việc đã thử; (2) bất đồng thuộc loại chỉ tầng trên
quyết được (ưu tiên kinh doanh, ngân sách, tài nguyên giữa các team); (3) chi phí của việc treo tiếp
lớn hơn chi phí làm mất thời gian của cấp trên. Cách trình bày đúng, ba câu: đây là quyết định gì và
đến hạn khi nào; hai phương án cùng lý do của mỗi bên, trình bày trung thực cả hai; đây là điều tôi
cần từ anh/chị (một câu trả lời cụ thể, không phải "xin ý kiến"). Cách trình bày sai: mang một bên
lên và mô tả bên kia; hoặc mang lên mà không có đề xuất của chính mình.

### Trade-off

| Trục | Nghiêng về bên A khi | Nghiêng về bên B khi |
|---|---|---|
| **Chốt nhanh bằng quyền (A) vs để hội tụ tự nhiên (B)** | Quyết định đang chặn nhiều người; đã qua hai vòng tranh luận không có thông tin mới; quyết định reversible nên sai còn sửa được; có dấu hiệu bắt đầu chuyển sang relationship conflict | Quyết định irreversible và thông tin còn phân tán ở hai bên; người thực thi cần buy-in thật để làm tốt; còn thời gian thật (không phải cảm giác còn thời gian) |
| **Khuyến khích tranh luận nội dung (A) vs bảo vệ sự gắn kết (B)** | Team có Psychological Safety đủ; quyết định đắt; đang có dấu hiệu đồng thuận giả (không ai phản đối một phương án phức tạp) | Team vừa qua một xung đột nặng chưa lành; hai bên đã ở trạng thái quy gán động cơ — khi đó thêm tranh luận nội dung chỉ cấp thêm đạn cho relationship conflict, cần xử lý quan hệ trước |
| **Lead phân xử (A) vs hai bên tự giải quyết (B)** | Đã quá trần thời gian; có bất đối xứng quyền lực giữa hai bên; lead là người có accountability cho kết quả | Hai bên tương đương và đã từng giải quyết được với nhau; vấn đề nằm trong phạm vi họ sở hữu; lead phân xử sẽ làm giảm năng lực tự xử lý xung đột của team về lâu dài |
| **Escalate sớm (A) vs giải quyết trong tầng (B)** | Bất đồng thực chất là về ưu tiên kinh doanh hoặc ngân sách — loại chỉ tầng trên quyết được; chi phí treo tiếp đã lớn | Chưa thử gì ở tầng mình; escalation sẽ được đọc là tố cáo trong bối cảnh tổ chức hiện tại; vấn đề thuộc thẩm quyền của bạn và việc đẩy lên làm bạn mất tư cách quyết định lần sau |

Ai chịu phần mất: khi lead chọn trì hoãn để "giữ hoà khí", phần mất do cả hai bên xung đột chịu (quan
hệ xấu dần theo cơ chế ở First Principles) và do team chịu dưới dạng các quyết định treo — trong khi
lead được lợi ngay là không phải bước vào một cuộc nói chuyện khó. Đây là bất đối xứng giải thích vì
sao Avoiding là chế độ mặc định phổ biến nhất và là chế độ gây thiệt hại lớn nhất trong dài hạn. Ngược
lại, khi lead chốt nhanh bằng quyền quá thường xuyên, phần mất là kênh thông tin: bên bị bác bỏ ngừng
mang thông tin lên, và mất mát đó không quan sát được cho đến lần sự cố mà lẽ ra đã được cảnh báo.

### Real-world Scenarios

**Tình huống A — hai Senior bất đồng monolith vs tách service, cả hai đều có lý.** Team logistics 8
người, Tech Lead Nam. Dũng, Senior BE 7 năm, ở công ty từ đầu, viết phần lớn monolith, phản đối tách:
lập luận là team không có ai vận hành được thêm service, Q4 có cam kết với ba đối tác, và "chúng ta
không phải Netflix". Phúc, Senior BE 6 năm, vào công ty 14 tháng, đến từ một công ty có kiến trúc
service, muốn tách: lập luận là mỗi lần deploy phải kéo cả hệ thống, hai lần sự cố ở vận đơn đã làm
sập toàn bộ, và nợ kỹ thuật đang tăng theo hàm số.

Cả hai đều đúng — và đây là chi tiết quan trọng nhất, vì nếu một bên sai thì đây không phải bài toán
xung đột, nó là bài toán kỹ thuật.

**Ba cách xử lý sai, theo thứ tự phổ biến:**

*Sai kiểu một — tránh.* Nam nói "hai anh trao đổi thêm rồi thống nhất cho em biết". Cơ chế: chuyển
gánh nặng ra quyết định cho hai người không có accountability và không có quyền phân xử, trong một
tình huống mà lập luận kỹ thuật đã cạn. Kết quả là ba tháng, bốn cuộc tranh luận lặp lại, và bước
chuyển hoá sang relationship conflict. Đây chính xác là con đường đã dẫn tới hiện trạng ở đầu chương.

*Sai kiểu hai — chọn bên theo trục uy tín thay vì theo tiêu chí.* Nam chọn phương án của Dũng vì Dũng
ở lâu hơn, hiểu hệ thống hơn, và nói to hơn trong họp. Cơ chế phá hoại: nó dạy cả team rằng thắng một
tranh luận kỹ thuật phụ thuộc vào thâm niên chứ không phụ thuộc vào lập luận, nên (a) người mới ngừng
mang thông tin từ bên ngoài vào — đó là lý do chính người ta tuyển người mới, và (b) Phúc bắt đầu đếm
ngày. Ở bối cảnh Việt Nam, đây là lỗi phổ biến nhất và ít bị nhận ra nhất, vì nó trông giống sự tôn
trọng.

*Sai kiểu ba — ép thoả hiệp.* Nam quyết "tách một nửa": để một phần logic vận đơn ra service mới,
phần còn lại ở monolith, dùng chung database. Cơ chế: trong thiết kế hệ thống, trung bình của hai
phương án tốt thường là một phương án tệ hơn cả hai — ở đây nó tạo ra hai nơi phải deploy đồng bộ, một
database chia sẻ (nên không có cô lập lỗi thật, tức là không đạt được lợi ích của việc tách), và độ
phức tạp vận hành của kiến trúc service. Cả Dũng và Phúc đều thấy nó tệ, và cả hai đều mất niềm tin
vào năng lực phán đoán kỹ thuật của Nam. Compromising là chế độ hợp lý cho việc chia tài nguyên; nó
là chế độ tệ cho các quyết định có tính toàn vẹn nội tại.

**Cách xử lý đúng — script buổi hoà giải 90 phút:**

> **Nam:** "Hôm nay mình chốt câu này: trong hai quý tới, vận đơn có tách hay không. Ba quy tắc cho
> 90 phút này. Một, ba mươi phút đầu không ai được nói tên phương án — chỉ nói tiêu chí và ràng buộc.
> Hai, mỗi người phải trả lời được 'cái gì phải đúng thì phương án của người kia là tốt hơn'. Ba, nếu
> hết 90 phút không hội tụ thì em chốt, vì accountability cho uptime và cho cam kết Q4 là của em. Em
> nói trước để không ai bất ngờ ở phút thứ 85.
>
> Bắt đầu bằng tiêu chí. Cả hai anh đã viết riêng, em đọc lên."
>
> *(Nam đọc hai danh sách. Bốn tiêu chí trùng nhau về nội dung. Khác nhau ở thứ tự: Dũng đặt "không
> làm chậm cam kết Q4" trước, Phúc đặt "giảm rủi ro sập toàn hệ thống" trước.)*
>
> **Nam:** "Vậy hai anh đồng ý về tiêu chí. Bất đồng thật là thứ tự giữa hai cái đầu. Đây không phải
> câu hỏi kỹ thuật — đây là câu hỏi ưu tiên kinh doanh, và không phải câu hỏi của bọn mình. Em đã hỏi
> chị Ngọc trước buổi này: Q4 cam kết đối tác là ràng buộc cứng vì có điều khoản phạt; từ Q1 thì rủi
> ro hệ thống thành ưu tiên số một. Hai anh có thấy cách đọc đó có gì sai không?"
>
> **Phúc:** "Không sai. Nhưng nếu Q4 sập vì vận đơn thì cam kết đối tác cũng mất."
>
> **Nam:** "Đúng, và đó là một giả định định lượng được. Anh nói con số nào sẽ làm anh Dũng đổi ý?"
>
> **Phúc:** "Nếu trong 12 tháng qua có ít nhất hai incident mà nguồn ở vận đơn và ảnh hưởng ra ngoài
> module, thì rủi ro là thật, không phải lo xa."
>
> **Dũng:** "Tôi đồng ý với tiêu chí đó. Nếu đúng hai lần trở lên thì tôi nhận là rủi ro thật."
>
> *(Kiểm tra tại chỗ: 2 trong 6 incident 12 tháng qua có nguồn ở vận đơn, cả hai lần ảnh hưởng toàn
> hệ thống.)*
>
> **Dũng:** "Vậy là đúng. Nhưng tách trong Q4 thì tôi vẫn phản đối vì thời gian."
>
> **Nam:** "Được, giờ đây là câu hỏi cuối. Anh Phúc, cái gì phải đúng thì việc giữ monolith là lựa
> chọn tốt hơn?"
>
> **Phúc:** "Nếu cô lập được lỗi vận đơn bằng cách khác mà không tách — bulkhead, circuit breaker,
> pool riêng — thì đạt được 60-70% lợi ích với chi phí một tuần. Nhưng nó không xoá được rủi ro chia
> sẻ database."
>
> **Nam:** "Đó là phương án thứ ba, và nó đến từ chính anh. Anh Dũng, anh thấy một tuần công cho
> bulkhead có phá cam kết Q4 không?"
>
> **Dũng:** "Không. Một tuần thì được, và thật ra tôi nên làm việc đó từ lần incident thứ nhất."
>
> **Nam:** "Vậy quyết định: Q4 làm bulkhead, anh Dũng thực thi. Kế hoạch tách vào Q1, anh Phúc thiết
> kế và sở hữu, trình ngày 15/01. Điều anh Phúc nêu về database chia sẻ em ghi vào biên bản là rủi ro
> chưa xử lý, kèm ba điều kiện họp lại. Và một câu cuối, em nói thẳng: ba tháng qua hai anh không nói
> chuyện trực tiếp về việc này, và cái đó tốn của team bốn ticket treo tám tuần. Việc treo là lỗi của
> em, vì em đã không chốt. Từ nay bất đồng kiến trúc có trần một tuần: hết một tuần không hội tụ thì
> đưa em, em chốt."

Sau đó, Nam làm một việc riêng và tách biệt: hai buổi 1-1 với từng người về **phần relationship** —
vì phần đó không được giải quyết bởi việc chốt nội dung. Với Dũng: "em cần anh nói ra khi anh thấy
người mới đề xuất thay đổi hệ thống anh xây, thay vì phản đối bằng cách chỉ ra chi tiết kỹ thuật nhỏ."
Với Phúc: "anh vào 14 tháng và anh đúng về rủi ro; nhưng cách anh nói 'hệ thống này thiết kế sai từ
đầu' làm người viết nó không thể đồng ý với anh mà không thừa nhận mình kém. Anh có thể đúng và vẫn
nói theo cách người kia nhận được."

Kết quả sáu tháng sau: bulkhead giảm số incident lan ra ngoài module về 0 trong Q4; việc tách service
diễn ra ở Q1 với Phúc thiết kế và Dũng review — và Dũng là người tìm ra hai vấn đề dữ liệu quan trọng
nhất trong quá trình migration, đúng vì Dũng hiểu hệ thống cũ hơn ai hết. Giá trị này chỉ tồn tại vì
xung đột được chốt trước khi nó chuyển hoá hoàn toàn.

**Tình huống B — im lặng trong họp, phản đối lúc triển khai.** Ở fintech, Minh trình bày phương án
chuẩn hoá error contract cho toàn bộ API. Trong họp 45 phút, không ai phản đối; Minh hỏi "mọi người
ok chưa", ba người gật, còn lại im lặng. Hai tuần sau, khi triển khai, ba team không làm theo, mỗi
team một lý do kỹ thuật hợp lý mà đáng lẽ phải được nêu trong họp.

Chẩn đoán: đây không phải sự bất tuân, đây là **xung đột đã chuyển kênh**. Cơ chế: trong buổi họp,
chi phí của việc phản đối cao (phải phản biện công khai người có vị thế cao hơn, có nguy cơ bị đọc là
gây khó) và lợi ích thấp (phương án nghe như đã chốt). Ở lúc triển khai, chi phí phản đối bằng gần 0
(chỉ cần không làm) và có thể biện minh bằng ràng buộc kỹ thuật cụ thể. Hệ thống thưởng cho im lặng,
nên hệ thống nhận được im lặng.

Can thiệp, theo hiệu lực tăng dần: (1) gửi phương án bằng văn bản 48 giờ trước, yêu cầu comment viết
— viết dễ hơn nói với người ngại đối đầu trực diện; (2) trong họp, thay câu "có ai phản đối không"
bằng "anh cần mỗi người nói một rủi ro của phương án này, ai không thấy rủi ro nào thì nói không
thấy" — biến việc nêu vấn đề từ hành vi tự nguyện có rủi ro thành nghĩa vụ được phân công; (3) chỉ
định một người làm phản biện chính thức (red team) và luân phiên vai đó, để việc phản đối là vai chứ
không phải thái độ; (4) hỏi riêng những người im lặng sau họp, rồi mang nội dung — không mang tên —
trở lại kênh công khai; (5) chốt bằng văn bản có ghi "ai đã đọc và không có phản đối", để im lặng trở
thành một cam kết có tên chứ không phải một trạng thái trung tính.

**Tình huống C — bất đồng với người hơn tuổi.** Ở ODC, Duy (28) thấy một lỗi thiết kế trong phương án
của một anh Technical Architect 45 tuổi phía khách hàng, và không nêu. Linh phát hiện ba tuần sau, khi
chi phí sửa đã gấp năm lần. Cách xử lý sai: nói với Duy "lần sau em phải mạnh dạn lên". Câu đó chuyển
toàn bộ chi phí của một vấn đề cấu trúc sang một cá nhân đang ở vị thế yếu nhất trong cấu trúc đó.
Cách xử lý được: tạo cơ chế không đòi hỏi cá nhân phải chịu rủi ro — Duy viết nhận xét kỹ thuật bằng
văn bản gửi Linh, Linh chuyển vào kênh chính thức với tư cách của mình và ghi rõ nguồn ("đây là quan
sát của Duy"); đồng thời thiết lập một bước review thiết kế bắt buộc có checklist, để việc nêu vấn đề
là một bước quy trình chứ không phải hành vi thách thức một người lớn tuổi hơn.

### Best Practices

- **Đặt trần thời gian cho mọi bất đồng kỹ thuật, và nói trần đó ra từ đầu.** Lý do: biến gây hại là
  thời gian, không phải cường độ. Trần một tuần cho bất đồng ở tầng team là mức dùng được.
- **Bắt viết tiêu chí trước khi nói phương án.** Lý do: sau khi phát ngôn phương án, người ta bảo vệ
  danh tính chứ không bảo vệ lập luận. Trước đó, họ thường phát hiện mình đồng ý phần lớn.
- **Bắt mỗi bên phát biểu phương án của bên kia ở dạng mạnh nhất.** Lý do: nó chống lại phiên bản bù
  nhìn, và nó là cách rẻ nhất để hạ nhiệt — rất khó tiếp tục coi người kia là bảo thủ ngay sau khi
  vừa trình bày lập luận của họ một cách thuyết phục.
- **Ghi dissent có tên vào biên bản.** Lý do: nó cho người không được chọn một đường thoát giữ được
  tư cách, và tạo cơ chế xem lại đúng nếu sau này họ đúng.
- **Tách việc chốt nội dung khỏi việc xử lý quan hệ, và làm cả hai.** Lý do: chốt nội dung không xoá
  được relationship conflict đã hình thành; và xử lý quan hệ mà không chốt nội dung sẽ tái diễn.
- **Nhận phần lỗi của lead khi xung đột kéo dài.** Lý do: xung đột treo ba tháng gần như luôn là lỗi
  của người đã không chốt. Nói ra điều đó khôi phục tính chính danh của quy trình mới.
- **Cấm rõ ràng việc phá hoại ngầm sau khi đã chốt, và cấm nó mạnh hơn cấm bất đồng.** Lý do: một tổ
  chức chịu được bất đồng ồn ào; nó không chịu được việc quyết định bị vô hiệu hoá âm thầm ở tầng thực
  thi.

### Anti-patterns

- **Hoà giải bằng cách chia đôi thiết kế.** Cơ chế: kiến trúc có tính toàn vẹn nội tại; trung bình
  của hai phương án đúng thường vi phạm cả hai bộ giả định và tạo ra hệ thống có nhược điểm của cả
  hai. Dấu hiệu sớm: quyết định được mô tả bằng cụm "làm một phần thôi" mà không ai định nghĩa được
  phần nào.
- **Phân xử theo trục uy tín (thâm niên, tuổi, độ ồn) thay vì theo tiêu chí.** Cơ chế: tối ưu hoà khí
  ngắn hạn, phá cơ chế đưa thông tin mới vào tổ chức. Dấu hiệu sớm: người mới ngừng đề xuất sau 2–3
  tháng; các quyết định luôn nghiêng về cùng một người.
- **Coi mọi xung đột là vấn đề tính cách.** Cơ chế: chẩn đoán sai loại xung đột dẫn tới can thiệp sai
  — dùng team building cho một task conflict chưa chốt, hoặc dùng dữ liệu cho một relationship conflict
  đã hình thành. Dấu hiệu sớm: lead dùng từ "hai người đó không hợp nhau" trước khi kiểm tra xem có
  quyết định nào đang treo.
- **Lead làm người truyền tin giữa hai bên.** Cơ chế: mỗi bên chỉ nói với lead, lead diễn giải lại,
  và cả hai xây mô hình về nhau qua một kênh nhiễu. Dấu hiệu sớm: lead nói "anh ấy có ý là..." nhiều
  hơn hai lần trong một tuần.
- **Ép hoà thuận bề mặt.** Yêu cầu hai người "bắt tay làm hoà" mà không chốt nội dung. Cơ chế: xung
  đột chuyển hoàn toàn vào kênh ngầm, và giờ đây có thêm một lệnh cấm nói ra. Dấu hiệu sớm: hai người
  lịch sự bất thường với nhau trong họp và không tương tác trong PR.
- **Escalation như một hành động trừng phạt.** Mang xung đột lên cấp trên để người kia bị khiển trách.
  Cơ chế: biến một bất đồng nội dung thành một cuộc tranh chấp quyền lực, và sau đó không ai escalate
  đúng lúc nữa vì escalation đã mang nghĩa tố cáo. Dấu hiệu sớm: escalation được thực hiện mà bên kia
  không biết trước.

### Khi nào KHÔNG nên áp dụng

- **Trong incident.** Không hoà giải trong lúc hệ thống đang xuống. Chế độ đúng là competing với một
  Incident Commander duy nhất; mọi bất đồng về nguyên nhân và phương án được ghi lại và mang vào
  postmortem. Cố dàn xếp một bất đồng kỹ thuật ở phút thứ 20 của một sự cố là cách kéo dài sự cố.
- **Khi vấn đề là hành vi vượt ranh giới, không phải bất đồng.** Quấy rối, xúc phạm, phân biệt đối xử,
  gian dối về kết quả công việc. Đây không phải xung đột hai chiều và không được xử lý bằng quy trình
  hoà giải đối xứng — làm vậy là hợp thức hoá hành vi và gây hại thêm cho người bị ảnh hưởng. Đường đi
  đúng là quy trình kỷ luật và HR.
- **Khi có bất đối xứng quyền lực lớn giữa hai bên.** Hoà giải giữa một Junior và một Tech Lead theo
  quy trình đối xứng sẽ cho ra kết quả nghiêng về bên mạnh, kể cả khi người điều phối trung lập. Ở đây
  cần can thiệp không đối xứng: lấy quan điểm của bên yếu bằng văn bản trước, và người điều phối phải
  ở cấp cao hơn bên mạnh.
- **Khi bất đồng thật thuộc tầng cao hơn.** Nếu hai người đang tranh luận vì hai mục tiêu kinh doanh
  chưa được xếp ưu tiên, hoà giải ở tầng team chỉ tạo ra một thoả hiệp không có cơ sở. Việc đúng là
  đưa câu hỏi ưu tiên lên đúng người trong một câu hỏi đóng.
- **Khi chi phí bàn cao hơn giá trị.** Bất đồng về quy ước đặt tên biến, thứ tự import, hoặc chi tiết
  không có hậu quả vận hành: không hoà giải, chọn một chuẩn bằng công cụ tự động (linter, formatter) và
  đóng chủ đề. Dành cơ chế đắt cho quyết định đắt.
- **Khi một bên đã quyết định rời tổ chức.** Đầu tư vào việc phục hồi quan hệ dài hạn ở đây không có
  người thu hoạch. Việc đúng là chốt nội dung để bàn giao sạch và giữ quan hệ ở mức chuyên nghiệp.

---

## 6. Psychological Safety

### Problem Statement

Một ví điện tử, sự cố ngày 28 Tết: hệ thống nạp tiền lỗi 47 phút vào giờ cao điểm. Trong postmortem,
một chi tiết xuất hiện muộn và gần như tình cờ: ba người đã biết trước. Một Mid nhìn thấy connection
pool gần cạn trên dashboard từ hai ngày trước và nghĩ "có lẽ mọi người đã biết". Một Junior đã hỏi
trong Slack channel của team, không ai trả lời, và không hỏi lại. Một Senior đã thấy phương án của
người khác thiếu xử lý một trường hợp và không nêu, vì tuần trước vừa có một tranh luận căng và không
muốn tiếp tục bị coi là người hay bắt lỗi.

Thông tin cần thiết để ngăn sự cố đã tồn tại trong tổ chức. Nó chỉ không đi lên được. Đây là điều mà
Psychological Safety nói về, và lý do nó không phải một chủ đề về sự dễ chịu: **nó là chủ đề về chất
lượng luồng thông tin từ dưới lên.**

Các đại lượng đếm được khi thiếu:

- **Độ trễ tin xấu.** Thời gian từ lúc người đầu tiên trong team biết một vấn đề đến lúc lead biết.
  Nếu bạn thường biết tin xấu cùng lúc với việc nó thành sự cố, kênh đã tắc.
- **Phân bố phát biểu trong họp.** Trong một buổi review thiết kế 8 người, đếm số người phát biểu ít
  nhất một lần có nội dung. Nếu ≤ 3 và luôn là 3 người đó, 5 người còn lại đang ngồi để có mặt.
- **Số câu hỏi của người mới trong 30 ngày đầu.** Người mới không hỏi không có nghĩa là họ hiểu.
- **Nội dung postmortem.** Nếu nguyên nhân gốc luôn dừng ở "thiếu kiểm tra" hoặc "lỗi con người" và
  không bao giờ chạm đến "áp lực deadline khiến chúng tôi bỏ bước X" hay "tôi không hiểu phần này và
  không nói", postmortem đang chạy trên phiên bản đã lọc.
- **Số lần một rủi ro được nêu và bị bỏ qua.** Đây là chỉ số quan trọng nhất và ít ai đo. Một rủi ro
  bị nêu và bị bỏ qua không có phản hồi sẽ dạy cả team về giá trị của việc nêu rủi ro, và bài học đó
  lan nhanh hơn mọi lời khuyến khích.
- **Sự khác biệt giữa nội dung nói trong họp và nội dung nói ở kênh riêng.** Nếu bạn nhận được nhiều
  thông tin quan trọng qua tin nhắn riêng hơn là trong kênh chung, tổ chức đang có một hệ thống thông
  tin song song, và hệ thống chính thức là hệ thống rỗng.

### First Principles

**Định nghĩa chính xác, và nó hẹp hơn cách dùng phổ biến.** Amy Edmondson (Harvard Business School,
nghiên cứu công bố từ 1999) định nghĩa Psychological Safety là **niềm tin được chia sẻ trong một nhóm
rằng nhóm đó an toàn cho việc chấp nhận rủi ro liên cá nhân** — tức là bốn hành vi cụ thể không bị
trừng phạt hay làm mất mặt: nêu một rủi ro, đặt một câu hỏi (kể cả câu hỏi có vẻ ngớ ngẩn), thừa nhận
một sai sót hoặc một điều mình không biết, và đề xuất một ý tưởng chưa hoàn chỉnh. Điểm gốc trong
nghiên cứu của Edmondson đáng nhắc lại vì nó chống lại cách hiểu sai thông dụng: dữ liệu ban đầu ở các
đơn vị điều trị trong bệnh viện cho thấy các đơn vị **báo cáo nhiều lỗi hơn** lại là các đơn vị có kết
quả tốt hơn. Không phải họ mắc nhiều lỗi hơn — họ *nói ra*. Đơn vị im lặng không phải đơn vị ít lỗi;
nó là đơn vị không biết mình có lỗi.

**Nó KHÔNG phải bốn thứ sau, và mỗi cái là một nhầm lẫn có hậu quả:**

| Không phải là | Vì sao lẫn | Thực tế |
|---|---|---|
| Sự dễ chịu, hoà thuận | Cả hai đều làm không khí bớt căng | Team dễ chịu thường tránh task conflict — tức là ngược với safety. Safety làm cho **tranh luận nội dung** trở nên khả thi |
| Bỏ hoặc hạ tiêu chuẩn | Vì cùng liên quan đến việc "không bị mắng khi sai" | Safety và Standard là hai trục độc lập. Tổ chức học được nằm ở góc cao–cao |
| Luôn được đồng ý, ý kiến nào cũng được làm | Vì "được nói" bị hiểu thành "được nghe theo" | Được nêu và được trả lời có lý do là safety. Được chấp nhận mọi lúc là vô chính phủ trong ra quyết định |
| Không có feedback khó, không có hệ quả | Vì phê bình dễ làm người ta cảm thấy không an toàn | Feedback thẳng về **công việc** là điều kiện của safety; nó chỉ phá safety khi nhắm vào con người hoặc khi bất ngờ |

**Cơ chế một: học của tổ chức đòi hỏi tín hiệu lỗi, và tín hiệu lỗi đi từ dưới lên.** Thông tin về
những gì đang sai — trong code, trong thiết kế, trong ước lượng, trong quy trình — nằm dày nhất ở tầng
thực thi. Tầng quản lý chỉ tiếp cận được nó qua báo cáo tự nguyện. Nếu chi phí của việc báo cáo dương,
tín hiệu bị lọc ở nguồn, và điều nghiêm trọng là **sự lọc này không quan sát được**: một tổ chức mất
khả năng học sẽ thấy các dashboard bình thường, các buổi họp êm ả, và các sự cố "không thể lường
trước". Im lặng không tạo ra dữ liệu.

**Cơ chế hai: bất đối xứng chi phí–lợi ích của việc lên tiếng.** Với một cá nhân, nêu một rủi ro có
chi phí **chắc chắn và cá nhân** (bị coi là tiêu cực, làm chậm cuộc họp, thách thức người có vị thế
cao hơn) và lợi ích **không chắc chắn và tập thể** (có thể sự cố không xảy ra, và nếu không xảy ra thì
không ai biết bạn đã đúng). Im lặng có chi phí bằng 0 nếu không có gì xảy ra, và nếu có gì xảy ra thì
trách nhiệm phân tán. Với phép tính đó, **im lặng là hành vi duy lý** — đây là điểm quan trọng nhất
của toàn bộ chủ đề. Không thể sửa bằng cách kêu gọi mọi người dũng cảm hơn; phải sửa bằng cách **thay
đổi phép tính**: giảm chi phí của việc lên tiếng (phản hồi tôn trọng, ghi nhận công khai, cho phép nêu
bằng văn bản) và tăng chi phí của việc im lặng (biến việc nêu rủi ro thành nghĩa vụ được phân công
trong quy trình, ví dụ mỗi người phải nêu một rủi ro trong review thiết kế).

**Cơ chế ba: safety là thuộc tính của nhóm cục bộ, không phải của công ty.** Nó được tạo ra gần như
hoàn toàn bởi hành vi của người có quyền lực gần nhất — Tech Lead trực tiếp. Hai team trong cùng công
ty, cùng chính sách, cùng phúc lợi có thể ở hai mức safety hoàn toàn khác nhau. Hệ quả thực tế: nếu
bạn là lead, bạn kiểm soát biến này nhiều hơn bạn nghĩ, và bạn không thể viện dẫn văn hoá công ty.

**Về Google Project Aristotle.** Nghiên cứu nội bộ của Google về hiệu quả team, được công bố rộng rãi,
kết luận rằng Psychological Safety là yếu tố dự báo mạnh nhất trong số các yếu tố họ xem xét — mạnh
hơn cả thành phần năng lực của team. Điều đáng lấy từ nghiên cứu này không phải kết luận dùng làm khẩu
hiệu, mà là **cơ chế**: các yếu tố tiếp theo trong danh sách của họ (dependability, structure and
clarity, meaning, impact) chỉ hoạt động được khi yếu tố đầu có mặt, vì cả bốn đều đòi hỏi người ta nói
ra sự thật về tiến độ, về sự rõ ràng, và về việc mình có thấy ý nghĩa hay không. Safety không phải một
yếu tố ngang hàng trong danh sách; nó là điều kiện để các yếu tố khác đo được.

### Mental Model

**Ma trận Safety × Standard.** Đây là mô hình cần nhất để chống lại nhầm lẫn "safety là dễ dãi":

```
            Standard THẤP                    Standard CAO
        +--------------------------+--------------------------+
Safety  |  COMFORT                 |  LEARNING                |
CAO     |  Vui vẻ, quan hệ tốt,    |  Nêu rủi ro được, sai    |
        |  không ai chịu áp lực.   |  được, nhưng tiêu chuẩn  |
        |  Nói được nhưng nói gì   |  không hạ. Tổ chức học   |
        |  cũng không đổi gì.      |  nhanh nhất ở ô này.     |
        |  Rời đi vì nhàm.         |                          |
        +--------------------------+--------------------------+
Safety  |  APATHY                  |  ANXIETY                 |
THẤP    |  Làm cho xong, không ai  |  Áp lực cao, không dám   |
        |  quan tâm. Thường là     |  nói. Chất lượng ngắn hạn|
        |  giai đoạn cuối trước    |  có thể tốt, nhưng tín   |
        |  khi team tan.           |  hiệu xấu bị che, sự cố  |
        |                          |  đến bất ngờ. Burnout.   |
        +--------------------------+--------------------------+
```

Ô nguy hiểm nhất không phải Apathy — ô đó dễ nhìn thấy. Nguy hiểm nhất là **Anxiety**, vì nó trông
giống hiệu suất cao: deadline được giữ, không ai phàn nàn trong họp, lead cảm thấy team đang "chạy tốt".
Nó chuyển thành khủng hoảng đột ngột, thường qua ba con đường: một sự cố lớn mà thông tin đã có sẵn,
một loạt người nghỉ trong hai tháng, hoặc một phát hiện muộn rằng tiến độ báo cáo không đúng thực tế.

**Vòng phản hồi của tín hiệu.** Mỗi lần một người nêu rủi ro, hệ thống trả về một trong ba kết quả:
được ghi nhận và xử lý (tín hiệu dương, hành vi tăng), được ghi nhận và quyết định không xử lý kèm lý
do (tín hiệu trung tính đến dương — quan trọng: người ta chấp nhận việc không được làm theo, họ không
chấp nhận việc không được trả lời), hoặc bị bỏ qua/bị phạt (tín hiệu âm mạnh, hành vi giảm và giảm cho
cả những người chỉ *quan sát* sự việc). Điều then chốt về mặt cơ chế: **tác động lớn nhất của một lần
xử lý sai không nằm ở người bị xử lý sai, mà ở những người chứng kiến** — họ cập nhật mô hình về chi
phí của việc lên tiếng mà không cần trải nghiệm trực tiếp.

### Practical Framework

**Bước 1 — Bốn hành vi của lead tạo ra safety.** Đây không phải thái độ, là hành vi có thể đếm.

| Hành vi | Cách làm cụ thể | Cơ chế |
|---|---|---|
| **Nêu điều mình chưa biết, trước** | Trong một buổi thiết kế, câu đầu tiên của lead: "Phần replication này anh chưa nắm, ai giải thích giúp anh." | Người có quyền lực cao nhất đi trước làm cho hành vi đó trở nên khả thi cho người khác. Không ai đi trước thì không ai đi |
| **Hỏi trước khi phán** | Khi thấy một quyết định lạ: "Em cho anh biết em đang tối ưu cho cái gì?" thay vì "sao lại làm thế này?" | Câu hỏi thứ hai giả định đã có kết luận; nó buộc người kia phòng vệ và bạn mất phần thông tin quan trọng nhất |
| **Phản hồi mọi lần có người nêu rủi ro, kể cả khi không xử lý** | "Cảm ơn Vy đã nêu. Anh ghi vào danh sách rủi ro. Lần này mình chấp nhận vì X, và mình sẽ giám sát bằng Y. Nếu Y vượt ngưỡng thì đây là việc số một." | Người ta chịu được việc rủi ro của mình không được ưu tiên; họ không chịu được sự im lặng, vì im lặng không phân biệt được với sự coi nhẹ |
| **Tách người khỏi lỗi** | "Điều gì trong hệ thống của chúng ta cho phép lỗi này đi tới production?" thay vì "ai đẩy commit này?" | Chuyển đối tượng phân tích từ người sang hệ thống. Không phải để bỏ trách nhiệm, mà vì nguyên nhân có thể sửa được gần như luôn nằm ở hệ thống |

Thêm một hành vi có hiệu lực cao và chi phí gần bằng 0: **nói ra sai của chính mình một cách cụ thể**,
không phải dạng khiêm tốn chung chung. "Brief anh viết cho Tuấn thiếu phần đối tác gửi trùng file, và
đó là lý do module thiếu idempotency" tạo ra nhiều safety hơn mười lần câu "anh cũng sai nhiều mà".

**Bước 2 — Thiết kế lại phép tính lên tiếng.** Vì im lặng là duy lý, cần các cơ chế thay đổi chi phí:

```
GIẢM CHI PHÍ CỦA VIỆC LÊN TIẾNG
- Cho phép nêu bằng văn bản trước họp (comment trên RFC) — người ngại đối đầu trực
  diện viết dễ hơn nói, đặc biệt khi phải phản biện người lớn tuổi hơn
- Vòng phát biểu bắt buộc: đi từng người, người ít thâm niên nói trước
- Vai phản biện luân phiên (red team) — biến việc phản đối thành vai được phân công,
  không phải thái độ cá nhân
- Kênh nêu rủi ro không cần tên (một form) cho các vấn đề khó nói

TĂNG CHI PHÍ CỦA VIỆC IM LẶNG
- Trong review thiết kế: mỗi người phải nêu một rủi ro hoặc nói rõ "tôi không thấy rủi ro nào"
- Ghi vào biên bản: ai đã đọc và không phản đối (im lặng trở thành cam kết có tên)
- Trong postmortem, câu hỏi bắt buộc: "có ai đã biết điều gì trước sự cố mà không nói ra
  được không, và vì sao không nói được?" — hỏi về cơ chế, không truy người
```

**Bước 3 — Đo bằng gì.** Hai nhóm chỉ số: khảo sát và quan sát.

Khảo sát — bảy câu, thang 1–5, ẩn danh, chạy mỗi quý (dạng câu hỏi này bám theo bộ thang đo của
Edmondson, diễn đạt lại cho ngữ cảnh kỹ thuật). Chú ý: bốn câu đầu đảo chiều (điểm cao = xấu):

1. Nếu tôi mắc một lỗi trong team này, nó thường bị quy về cá nhân tôi.
2. Khó nêu ra một vấn đề hoặc một rủi ro trong team này.
3. Trong team này, đôi khi người ta bị đối xử khác đi vì cách nghĩ khác.
4. Nhận sự giúp đỡ từ đồng đội trong team này là việc khó làm.
5. Tôi có thể nói "tôi không biết" hoặc "tôi chưa hiểu" trong một buổi họp mà không lo bị đánh giá.
6. Khi tôi nêu một rủi ro kỹ thuật, tôi nhận được câu trả lời rõ ràng — kể cả khi câu trả lời là
   "chúng ta chấp nhận rủi ro này".
7. Kỹ năng và đóng góp riêng của tôi được sử dụng và được ghi nhận trong team này.

Điều quan trọng hơn con số: **so sánh giữa các team trong cùng công ty và so sánh theo thời gian của
cùng một team**, không so với chuẩn ngành. Và nếu bạn chạy khảo sát này, bạn phải công bố kết quả cùng
với ít nhất một thay đổi cụ thể — nếu không, việc chạy khảo sát là một tín hiệu âm mạnh hơn việc không
chạy.

Quan sát — các tín hiệu đọc được mà không cần khảo sát:

| Nơi quan sát | Tín hiệu tốt | Tín hiệu xấu |
|---|---|---|
| **Retro** | Có vấn đề thuộc về lead hoặc thuộc về quy trình được nêu; action item do người không phải lead đề xuất; có nhắc tới việc đã làm sai | Chỉ nói về công cụ và về team khác; mọi action item nhắm ra ngoài team; không ai nhắc chuyện tuần trước bị hoãn |
| **Postmortem** | Có câu "tôi đã không hiểu phần này"; có nguyên nhân thuộc về áp lực tiến độ; có người thừa nhận đã bỏ một bước | Nguyên nhân gốc luôn là "thiếu test"; không ai nói mình đã biết trước; lời lẽ dạng bị động không có chủ thể |
| **Review thiết kế** | Junior đặt câu hỏi cho Senior; có người nói "cái này tôi chưa biết" | Chỉ Senior nói; câu hỏi duy nhất là câu hỏi làm rõ, không có câu hỏi thách thức |
| **Slack/kênh chung** | Câu hỏi kỹ thuật được đặt công khai; có người trả lời "tôi cũng không biết" | Câu hỏi quan trọng đi qua tin nhắn riêng; kênh chung chỉ có thông báo |
| **Ước lượng** | Có người nói "tôi không chắc, độ tin cậy thấp" | Mọi estimate đều tự tin và mọi estimate đều trễ |

### Trade-off

| Trục | Nghiêng về bên A khi | Nghiêng về bên B khi |
|---|---|---|
| **Mở tranh luận rộng (A) vs chốt nhanh (B)** | Quyết định irreversible, nhiều thông tin nằm rải rác trong team, có thời gian | Đang trong incident hoặc có deadline theo giờ; quyết định reversible và rẻ để sửa. Lưu ý: chốt nhanh không phá safety **nếu** lead nói rõ vì sao đang chốt nhanh |
| **Minh bạch thông tin xấu (A) vs bảo vệ team khỏi lo lắng (B)** | Thông tin ảnh hưởng trực tiếp đến quyết định của team; thông tin sẽ rò rỉ dù sao (runway, mất khách hàng lớn) | Thông tin chưa chắc chắn và team không quyết được gì với nó. Nguyên tắc: nói cái đã chắc, nói rõ cái chưa chắc, và nói khi nào sẽ có thêm tin — sự bất định có quản lý ít độc hơn tin xấu bị che |
| **Blameless (A) vs làm rõ trách nhiệm (B)** | Phân tích nguyên nhân sự cố — mục tiêu là học và sửa hệ thống | Đánh giá hiệu suất định kỳ, hoặc khi có mẫu hành vi lặp lại nhiều lần sau khi đã được feedback rõ. Hai bối cảnh này phải tách nhau về thời gian và về phòng họp |
| **Chấp nhận ý tưởng chưa chín (A) vs giữ chất lượng đầu ra (B)** | Ở pha khám phá phương án — mục tiêu là số lượng phương án | Ở pha chốt và triển khai — mục tiêu là tính đúng đắn. Nói rõ đang ở pha nào là cách giữ được cả hai |

Ai chịu phần mất: khi lead chọn chốt nhanh và không giải thích, phần mất do những người có thông tin
nhưng không kịp nói chịu — và tổ chức chịu ở lần sự cố sau. Khi lead chọn mở tranh luận cho mọi quyết
định, phần mất là decision latency, và nó do toàn team chịu dưới dạng công việc bị treo.

### Real-world Scenarios

**Tình huống A — một rủi ro được nêu và bị bỏ qua.** Ở ví điện tử, ba ngày trước đợt khuyến mãi lớn,
Vy (Junior 14 tháng) nhắn vào kênh team: "Em thấy connection pool của service nạp tiền đang dùng 85%
lúc peak thường, đợt sale gấp 3 thì có sao không ạ?" Không ai trả lời trong 6 giờ. Minh đọc, thấy hợp
lý nhưng đang chạy ba việc khác, và không phản hồi.

Script — **phiên bản nói sai** (khi Vy nhắc lại trong họp standup):

> **Vy:** "Anh ơi cái connection pool em nói hôm trước..."
>
> **Minh:** "À, cái đó anh nghĩ ổn thôi, năm ngoái sale cũng thế mà không sao. Với lại giờ mình đang
> gấp phần voucher, để sau sale rồi tính nhé. Em tập trung vào task của em đi."

Bốn hệ quả, và hệ quả thứ tư là hệ quả đắt nhất. (1) Lập luận "năm ngoái không sao" là lập luận bằng
sự may mắn đã xảy ra, không phải bằng dữ liệu về tải. (2) "Task của em" định nghĩa lại phạm vi quan
tâm của Vy hẹp hơn — một cách trực tiếp giảm nỗ lực tuỳ ý. (3) Không có ai được giao việc kiểm tra,
nên rủi ro biến mất khỏi hệ thống theo dõi. (4) **Bảy người khác đọc được đoạn hội thoại này**, và
mỗi người cập nhật một chút: nêu rủi ro ngoài phạm vi task của mình là việc không được đón nhận. Chi
phí của việc này không xuất hiện trong sự cố lần này; nó xuất hiện ở sự cố lần sau, dưới dạng ba người
biết trước mà không nói.

Script — **phiên bản nói đúng**:

> **Minh:** "Vy, cảm ơn em, và anh xin lỗi vì đọc mà không trả lời hôm qua — cái đó là anh sai, câu
> hỏi này quan trọng hơn phần lớn tin nhắn trong kênh. Anh nói lại cho cả team nghe luôn.
>
> 85% ở tải thường mà sale gấp ba là không an toàn, vì pool không scale tuyến tính với traffic —
> khi latency của DB tăng, connection bị giữ lâu hơn nên nhu cầu pool tăng nhanh hơn cả mức tăng
> traffic. Đây đúng là loại rủi ro cần xử lý trước sale, không phải sau.
>
> Cụ thể: Khoa, em kiểm tra pool size và max connection của DB, cho anh con số trong hôm nay. Nếu tăng
> được pool mà DB chịu được thì làm, nếu không thì mình thêm queue ở tầng nạp tiền. Vy, em viết lại
> quan sát của em vào danh sách rủi ro trước sale, và em là người theo dõi metric này trong hai ngày
> tới — em báo trực tiếp cho anh nếu vượt 90%.
>
> Một điều anh nói với cả team: cái Vy vừa làm là việc anh cần nhiều hơn, không phải ít hơn. Nêu một
> rủi ro không thuộc task của mình là việc khó làm nhất trong team, vì nó có vẻ như đang xen vào việc
> người khác. Nếu ai thấy cái gì tương tự, anh muốn biết — kể cả khi nó hoá ra không có gì."

Ba việc đã xảy ra trong 90 giây: rủi ro được đưa vào hệ thống theo dõi có người sở hữu; lead nhận sai
của mình (mô hình hoá hành vi thừa nhận sai); và tín hiệu tới bảy người quan sát được đảo chiều. Chú ý
một chi tiết cố ý: Vy được giao việc theo dõi metric — người nêu rủi ro được cho một phần ownership,
đây là cách biến việc lên tiếng thành có lợi thay vì có phí.

**Tình huống B — blameless bị hiểu sai thành không có trách nhiệm.** Sau khi công ty áp dụng
postmortem blameless, một Senior đẩy một thay đổi cấu hình trực tiếp lên production không qua review
lần thứ ba trong bốn tháng, gây downtime 20 phút. Trong postmortem, không ai nhắc đến việc đây là lần
thứ ba; kết luận là "cần thêm guardrail trong pipeline". Ba tuần sau, chuyện lặp lại.

Đây là hình thái phổ biến nhất của việc hiểu sai. Blameless có nghĩa là **trong buổi phân tích nguyên
nhân, đối tượng phân tích là hệ thống, không phải cá nhân** — vì mục tiêu của buổi đó là có được mô tả
trung thực về chuỗi sự kiện, và mô tả trung thực không tồn tại nếu người liên quan đang tự bảo vệ.
Nó **không** có nghĩa là hành vi lặp lại sau khi đã được chỉ ra thì không có hệ quả. Cách xử lý đúng
là tách hai việc theo thời gian và theo phòng họp: postmortem giữ blameless nghiêm ngặt và tập trung
vào guardrail; sau đó, trong một buổi 1-1 riêng, lead nói thẳng về mẫu hành vi lặp lại: "Đây là lần
thứ ba, và hai lần trước mình đã nói. Từ giờ thay đổi cấu hình production của em cần một người review,
đây không phải hình phạt mà là điều kiện làm việc, và mình sẽ xem lại sau hai tháng." Trộn hai việc
này vào một buổi phá cả hai: postmortem mất tính trung thực, và cuộc nói chuyện về accountability mất
tính rõ ràng.

**Tình huống C — safety nội bộ trong ODC khi đối ngoại không được phép sai.** Ở team của Linh, khách
Nhật kỳ vọng mọi cam kết được giữ và mọi câu trả lời phải chắc chắn. Điều này tạo áp lực đẩy sự bất
định xuống dưới: người nào không chắc cũng phải nói chắc. Kết quả là Linh nhận được tiến độ báo cáo
không đúng thực tế và biết vào tuần cuối.

Cách xử lý được: **tách hai lớp một cách tường minh.** Bên trong, quy tắc là báo cáo độ tin cậy kèm
theo ước lượng ("xong 70%, độ tin cậy thấp vì phần tích hợp chưa thử") và việc báo tin xấu sớm là hành
vi được ghi nhận, không bị phạt. Bên ngoài, Linh là người duy nhất giao tiếp trạng thái với khách, và
Linh chuyển hoá độ bất định thành ngôn ngữ khách nhận được (nêu rủi ro kèm phương án dự phòng, sớm,
kèm mốc quyết định). Điểm cần nói thẳng: nếu lead đòi cấp dưới báo cáo tin xấu sớm nhưng lại phạt họ
khi khách phản ứng, thì trong hai tháng hệ thống sẽ trở lại trạng thái cũ. Giá của safety nội bộ là
lead phải hấp thụ áp lực ở biên, và đó là một phần công việc không thể chuyển giao.

### Best Practices

- **Đi trước trong việc thừa nhận.** Lý do: hành vi rủi ro chỉ trở nên khả thi khi người có quyền lực
  cao nhất làm nó trước. Cụ thể và có thể làm ngay: mỗi buổi thiết kế, nói ra một điều bạn chưa biết.
- **Trả lời mọi rủi ro được nêu, trong 24 giờ, kể cả khi câu trả lời là không xử lý.** Lý do: người ta
  chịu được việc không được ưu tiên; họ không chịu được sự im lặng, vì im lặng không phân biệt được với
  sự coi nhẹ.
- **Biến việc nêu rủi ro thành nghĩa vụ được phân công.** Lý do: nó chuyển hành vi từ "dũng cảm cá
  nhân" sang "làm đúng vai", và loại bỏ chi phí liên cá nhân — đặc biệt quan trọng trong môi trường
  coi trọng giữ mặt.
- **Cho người nêu rủi ro một phần ownership của việc theo dõi nó.** Lý do: nó biến việc lên tiếng
  thành có lợi, và đồng thời tạo cơ hội phát triển.
- **Cho phép nêu bằng văn bản trước khi họp.** Lý do: với người ngại đối đầu trực diện hoặc phải phản
  biện người lớn tuổi hơn, viết là kênh khả thi trong khi nói thì không.
- **Giữ Standard cao và nói rõ rằng bạn đang giữ.** Lý do: safety không kèm standard tạo ra ô Comfort,
  và người giỏi rời khỏi ô đó. Câu dùng được: "Anh muốn ai cũng nói được rằng mình chưa hiểu. Và tiêu
  chuẩn cho code lên production thì không đổi."
- **Tách postmortem khỏi đánh giá hiệu suất, về thời gian và về phòng họp.** Lý do: hai buổi này có
  hai mục tiêu loại trừ nhau — một cần sự thật, một cần phán quyết.

### Anti-patterns

- **"Cứ thoải mái nói" mà không đổi cơ chế.** Cơ chế phá hoại: lời mời không thay đổi phép tính chi
  phí–lợi ích, nên hành vi không đổi; nhưng bây giờ lead tin rằng đã mở kênh và đọc sự im lặng là sự
  đồng thuận. Dấu hiệu sớm: lead nói "anh đã bảo mọi người cứ nói mà" khi có sự cố.
- **Trừng phạt người mang tin xấu bằng những cách nhỏ.** Không mắng, nhưng thở dài, nhưng hỏi "sao
  giờ mới nói", nhưng giao thêm việc điều tra như một hệ quả. Cơ chế: phạt vi mô đủ để thay đổi hành
  vi và đủ nhỏ để lead không nhận ra mình đang phạt. Dấu hiệu sớm: tin xấu bắt đầu đến qua người thứ
  ba.
- **Dùng safety làm lý do không đưa feedback khó.** Cơ chế: chuyển vào ô Comfort; người dưới chuẩn
  không biết mình dưới chuẩn cho đến kỳ review, và những người còn lại thấy tiêu chuẩn không được thực
  thi — điều này làm giảm safety của người làm tốt, vì họ mất niềm tin vào tính công bằng. Dấu hiệu
  sớm: không ai trong team nhận được feedback tiêu cực nào trong sáu tháng.
- **Blameless như một lệnh cấm nói về hành vi.** Cơ chế: xem tình huống B — mẫu hành vi lặp lại không
  bị xử lý, và những người làm đúng học được rằng cẩn thận là tự nguyện. Dấu hiệu sớm: cùng một loại
  sự cố xuất hiện lần thứ ba với cùng một nguyên nhân trực tiếp.
- **Khảo sát rồi im lặng.** Cơ chế: chạy survey là một hành động hỏi; không công bố và không thay đổi
  gì là một câu trả lời, và câu trả lời đó là "việc nói ra không dẫn tới đâu". Kênh tín hiệu bị đóng
  chặt hơn trước. Dấu hiệu sớm: tỉ lệ tham gia survey giảm ở kỳ thứ hai.
- **Đồng nhất safety với sự đồng ý.** Lead chấp nhận mọi đề xuất để không ai cảm thấy bị bác bỏ. Cơ
  chế: phá huỷ tính quyết đoán của việc ra quyết định và thực ra làm giảm safety — người ta mất niềm
  tin vào việc có ai đang thực sự cầm lái. Dấu hiệu sớm: các quyết định kiến trúc trở thành danh sách
  nguyện vọng ghép lại.

### Khi nào KHÔNG nên áp dụng

Mục này cần xử lý thẳng một nhầm lẫn thay vì liệt kê điều kiện biên, vì nhầm lẫn đó là nguyên nhân
phần lớn thất bại khi triển khai Psychological Safety trong thực tế.

**"Safety nghĩa là bỏ accountability" — đây là nhầm lẫn, và cần nói rõ nó sai ở chỗ nào.** Safety là
điều kiện để accountability hoạt động được, không phải cái thay thế nó. Cơ chế: accountability đòi hỏi
thông tin trung thực về việc gì đã xảy ra và ai đã cam kết điều gì; trong môi trường không có safety,
thông tin đó bị lọc, nên mọi phán quyết về accountability đều dựa trên phiên bản đã bị chỉnh sửa —
tức là bạn có hình thức của accountability mà không có nội dung. Ngược lại, safety không kèm
accountability tạo ra ô Comfort, và ô đó mất người giỏi nhanh nhất. Hai thứ này là hai trục, và mục
tiêu là góc cao–cao. Ba đường phân định dùng được: **safety bảo vệ việc nói ra, không bảo vệ việc
không làm**; **safety áp dụng cho sai sót lần đầu và cho việc thừa nhận, không áp dụng cho mẫu hành vi
lặp lại sau khi đã được feedback rõ**; **safety nói về cách xử lý con người, không nói về tiêu chuẩn
đầu ra.**

Các điều kiện biên còn lại:

- **Trong incident đang diễn ra, không mở không gian thảo luận rộng.** Cơ chế cần là chỉ thị rõ và một
  người quyết. Safety trong bối cảnh này có nghĩa hẹp và cụ thể: bất kỳ ai cũng có thể nói "tôi thấy
  một điều bất thường" và được nghe trong 10 giây; nó không có nghĩa là mọi người cùng bàn phương án.
- **Khi hành vi vượt ranh giới.** Quấy rối, gian dối về kết quả, vi phạm bảo mật dữ liệu khách hàng.
  Áp dụng blameless ở đây là bảo vệ người vi phạm và phá safety của những người bị ảnh hưởng — vì
  safety của một nhóm phụ thuộc vào việc các ranh giới có được thực thi hay không. Đường đi đúng là
  quy trình kỷ luật.
- **Khi người dưới chuẩn cần biết rõ mình đang ở đâu.** Sự mơ hồ tử tế là hình thức đối xử tệ. Người
  không đạt kỳ vọng có quyền được biết chính xác kỳ vọng là gì, khoảng cách là gì, và hậu quả nếu
  không đóng được khoảng cách. Làm việc đó rõ ràng và có tính dự báo là một hành động **tạo** safety,
  không phải phá.
- **Khi việc nói ra không đi kèm bất kỳ khả năng thay đổi nào.** Mở một diễn đàn để mọi người nêu vấn
  đề về những thứ chắc chắn không thay đổi được (bản chất hợp đồng, quyết định của tập đoàn mẹ) tạo ra
  sự bất mãn có tổ chức. Việc đúng là nói rõ biên: cái gì trong phạm vi thảo luận, cái gì không, và vì
  sao.
- **Khi bạn chưa sẵn sàng thay đổi hành vi của chính mình.** Đây là điều kiện biên thật và thường bị
  bỏ. Nếu lead khởi động một chương trình về safety trong khi vẫn cắt lời người khác trong họp, vẫn
  phản ứng phòng vệ khi bị phản biện, thì chương trình đó làm khoảng cách giữa lời nói và hành vi trở
  nên rõ ràng hơn — và khoảng cách nhìn thấy được làm giảm safety nhiều hơn là không làm gì. Thứ tự
  đúng: đổi hành vi của lead trong 4–6 tuần trước, rồi mới nói về khái niệm.

---

## 7. Team Dynamics và vòng đời team

### Problem Statement

Trở lại con số ở đầu chương: team 8 người, 34 năm kinh nghiệm cộng lại, throughput tương đương 3,5
người. Không ai lười, không ai kém. Đây là hình thái mất mát khó thấy nhất trong quản trị kỹ thuật, vì
nó không nằm ở bất kỳ cá nhân nào — nó nằm ở cấu hình của hệ thống chứa họ. Các đại lượng cho phép
chẩn đoán:

- **Số giờ họp mỗi người mỗi tuần.** Nếu vượt 8 giờ với một engineer, phần thời gian tập trung sâu còn
  lại không đủ cho một task phức tạp — và cần nhớ rằng chi phí thật của một cuộc họp lúc 3 giờ chiều
  không phải một giờ, nó là cả buổi chiều bị chia đôi.
- **Số handoff cho một ticket.** Đếm số lần một ticket chuyển từ người này sang người khác trước khi
  xong (BE → FE → QA → DevOps → review). Mỗi handoff là một hàng đợi, và thời gian chờ ở hàng đợi
  thường lớn hơn thời gian làm.
- **Tỉ lệ thời gian ticket ở trạng thái chờ so với thời gian đang được làm.** Ở phần lớn team chưa tối
  ưu, tỉ lệ này là 3:1 hoặc tệ hơn. Điều đó có nghĩa: tăng tốc độ code 30% cải thiện tổng thời gian
  khoảng 7%, còn giảm thời gian chờ một nửa cải thiện hơn 30%.
- **WIP — số việc đang mở chia số người.** Nếu lớn hơn 1,5, mọi người đang chuyển ngữ cảnh, và chi phí
  chuyển ngữ cảnh không hiển thị trên bất kỳ báo cáo nào.
- **Bus factor theo module.** Bằng 1 ở bất kỳ module quan trọng nào là một single point of failure bằng
  người.
- **Tỉ lệ đóng góp lệch.** Nếu hai người tạo ra 60% output, phần còn lại đang ở trạng thái nào cần được
  chẩn đoán — không tự động là do thiếu năng lực.
- **Thời gian để một người mới tạo PR có ý nghĩa đầu tiên.** Nếu quá 3 tuần, chi phí onboarding đang
  ăn hết phần lợi ích của việc tuyển thêm trong quý đầu.

### First Principles

**Cơ chế một: chi phí phối hợp tăng theo bình phương số người.** Số kênh giao tiếp đôi một trong nhóm
n người là n(n−1)/2. Với 4 người: 6 kênh. Với 8 người: 28. Với 12 người: 66. Sản lượng cộng thêm của
mỗi người mới là tuyến tính (hoặc dưới tuyến tính), còn chi phí phối hợp là bậc hai — nên tồn tại một
điểm sau đó thêm người làm giảm throughput. Đây là nội dung định lượng của định luật Brooks ("thêm
người vào một dự án đang trễ làm nó trễ hơn"), và là lý do của quy tắc two-pizza team ở Amazon: giới
hạn kích thước không phải để tiết kiệm, mà để giữ số kênh phối hợp trong ngưỡng con người xử lý được.
Ngưỡng thực tế cho một team kỹ thuật có tính phụ thuộc lẫn nhau cao là **5–8 người**; trên đó cần chia
thành các đơn vị có ranh giới rõ, không phải quản lý bằng thêm quy trình.

Một hệ quả ít được nói: chi phí phối hợp có thể **giảm bằng thiết kế ranh giới**, chứ không chỉ bằng
giảm số người. Nếu 8 người được chia thành hai nhóm 4 người sở hữu hai vùng có giao diện rõ ràng, số
kênh cần hoạt động thường xuyên giảm từ 28 xuống 6 + 6 + một số kênh liên nhóm. Đây là toàn bộ ý tưởng
của Team Topologies và là lý do Cognitive Load — lượng thứ một team phải giữ trong đầu — là biến thiết
kế quan trọng hơn số đầu người.

**Cơ chế hai: Ringelmann effect và social loafing.** Thí nghiệm kéo dây của Ringelmann (cuối thế kỷ 19)
cho kết quả rằng lực kéo trung bình mỗi người **giảm** khi số người tăng: không phải vì ai đó cố ý lười,
mà vì hai nguyên nhân cộng lại — mất phối hợp (lực không cùng hướng, cùng thời điểm) và giảm động lực.
Phần giảm động lực về sau được gọi là social loafing, với ba cơ chế: (a) **đóng góp cá nhân không nhận
diện được** — nếu không ai biết ai đã làm gì, nỗ lực biên không có người hưởng; (b) **phân tán trách
nhiệm** — "chắc có người khác lo"; (c) **sucker effect** — người làm nhiều nhận ra mình đang gánh cho
người khác và tự điều chỉnh xuống. Suy ra ba điều kiện khử, và cả ba đều là việc lead làm được: đóng
góp phải nhận diện được (ownership theo module, tên người trong quyết định), mục tiêu nhóm phải rõ và
đo được, và nhóm phải nhỏ. Chú ý cơ chế (c): trong một team có Hero, chính Hero là nguồn của social
loafing ở những người còn lại — không phải hệ quả của nó.

**Cơ chế ba: hiệu suất team là hàm của clarity và interdependence, và hai biến này tương tác.** Xét bốn
tổ hợp: interdependence cao + clarity cao là team hiệu quả; interdependence cao + clarity thấp là trạng
thái tệ nhất trong bốn (mọi người chặn nhau và không ai biết mình đang chờ ai — đây là mô tả chính xác
của team logistics); interdependence thấp + clarity cao là một nhóm cá nhân làm việc song song hiệu quả,
nhưng nó không cần các thực hành team và áp các thực hành đó vào là tạo ra nghi lễ; interdependence
thấp + clarity thấp là một nhóm hành chính. Hệ quả quan trọng cho người mới làm lead: **trước khi áp dụng bất
kỳ thực hành team nào, phải xác định mức interdependence thật** — bao nhiêu phần công việc đòi hỏi hai
người trở lên phối hợp trong cùng một tuần. Nếu con số đó thấp, cách tăng hiệu suất là giảm phối hợp,
không phải tăng.

**Cơ chế bốn: team có bộ nhớ tồn tại ngoài đầu từng người, và bộ nhớ đó đắt.** Hai dạng: shared mental
model (hiểu chung về hệ thống, về cách làm việc, về cái gì quan trọng) và transactive memory (biết ai
biết cái gì). Cả hai được xây bằng thời gian làm việc cùng nhau và đều bị reset một phần khi thành viên
thay đổi. Đây là lý do định lượng cho việc giữ team ổn định: chi phí của việc thay một người không phải
chi phí tuyển và onboard người đó, mà là chi phí xây lại bộ nhớ chung — thường tính bằng quý, không
bằng tuần. Và nó cũng là lý do tổ chức lại team quá thường xuyên (mỗi quý một lần "để tối ưu nguồn
lực") có thể phá nhiều hơn tạo.

### Mental Model

**Tuckman như một công cụ chẩn đoán, không phải một nhãn dán.** Bốn giai đoạn — Forming, Storming,
Norming, Performing — có giá trị khi dùng để **đọc tín hiệu và chọn can thiệp**, và mất giá trị hoàn
toàn khi dùng để dán nhãn ("team mình đang ở Storming, chờ qua giai đoạn này thôi"). Hai điểm thường bị
hiểu sai: các giai đoạn không đi một chiều — **mỗi lần thay đổi thành viên, đổi lead, hoặc đổi mục tiêu
lớn, team lùi lại một phần**, và một team Performing sau khi nhận hai người mới không còn là team
Performing; và Storming không phải một giai đoạn phải chịu đựng thụ động — nó là giai đoạn mà can thiệp
của lead có hiệu lực cao nhất trong toàn bộ vòng đời.

**Little's Law và WIP.** Thời gian trung bình một việc nằm trong hệ thống = số việc đang trong hệ thống
chia thông lượng. Hệ quả trực tiếp và rất hữu dụng: **giảm số việc đang mở làm giảm thời gian hoàn thành
mỗi việc, mà không cần ai làm nhanh hơn.** Đây là can thiệp có tỉ lệ hiệu quả/chi phí cao nhất mà một
Tech Lead có, và nó gần như luôn bị bỏ qua vì nó trông giống việc "làm ít hơn".

**Conway's Law.** Cấu trúc hệ thống có xu hướng phản chiếu cấu trúc giao tiếp của tổ chức tạo ra nó. Hai
chiều sử dụng: đọc kiến trúc để suy ra vấn đề tổ chức (nếu ranh giới service liên tục bị vi phạm, có thể
vì hai team không có ranh giới trách nhiệm rõ); và thiết kế tổ chức để có được kiến trúc mong muốn
(inverse Conway). Chi tiết ở `09-to-chuc-va-scaling.md`.

### Practical Framework

**Bước 1 — Chẩn đoán giai đoạn bằng tín hiệu quan sát được.**

| Giai đoạn | Tín hiệu quan sát | Can thiệp đúng của lead | Sai lầm thường gặp |
|---|---|---|---|
| **Forming** | Lịch sự quá mức; hỏi lead mọi thứ; không ai phản biện ai; câu hỏi về vai và quy trình nhiều hơn về nội dung | Cung cấp cấu trúc: ai làm gì, quyết định thế nào, chuẩn code, định nghĩa xong. Chế độ directing là đúng ở đây | Áp dụng "tự tổ chức" ngay từ đầu — team chưa có nền để tự tổ chức, kết quả là sự trì trệ lễ phép |
| **Storming** | Bất đồng về cách làm; tranh chấp ngầm về ai quyết; phàn nàn về quy trình; xuất hiện các cặp không hợp nhau | Chốt các quyết định process bằng văn bản (working agreement); giải quyết xung đột task trong thời hạn (chủ đề 5); làm rõ ranh giới quyết định | Coi đây là vấn đề tính cách và tìm cách "giữ hoà khí". Hoặc chờ nó tự qua — nó không tự qua, nó chuyển thành relationship conflict |
| **Norming** | Có chuẩn không viết mà mọi người theo; xung đột được xử lý trong nhóm không cần lead; bắt đầu tự phân việc | Rút dần: tăng mức delegation; chuyển từ quyết định sang thiết lập tiêu chí; bắt đầu mentoring có hệ thống | Tiếp tục can thiệp theo thói quen — đây là điểm mà lead trở thành lực cản và không nhận ra |
| **Performing** | Team tự xử lý phần lớn; tín hiệu xấu đi lên nhanh; người mới được onboard bởi team chứ không bởi lead | Bảo vệ ranh giới với bên ngoài; đầu tư vào bus factor và vào phát triển người; chuẩn bị cho việc chia team | Tưởng trạng thái này bền. Thêm hai người, đổi mục tiêu, hoặc mất một người là quay về Storming. Cần chủ động lặp lại các bước Forming khi có thay đổi |

**Bước 2 — Working agreement (team charter).** Đây là công cụ trung tâm cho giai đoạn Forming và
Storming. Mục đích không phải để đẹp trên wiki, mà để **chuyển các kỳ vọng ngầm thành các cam kết có
thể chỉ vào được** — vì phần lớn xung đột process xuất phát từ hai người có hai giả định khác nhau mà
không ai biết là mình đang giả định.

```
WORKING AGREEMENT — Team Vận đơn
Phiên bản 3, cập nhật 12/08. Xem lại: mỗi quý hoặc khi có thành viên mới.
Ai cũng có quyền đề nghị sửa. Sửa thì cả team đọc lại trong 15 phút ở retro gần nhất.

1. QUYẾT ĐỊNH — AI QUYẾT CÁI GÌ (chi tiết ở delegation brief từng việc)
   - Thiết kế trong một module: người sở hữu module quyết. Không cần xin phép.
   - Thay đổi giao diện giữa hai module: hai người sở hữu cùng chốt; không hội tụ trong
     3 ngày -> Tech Lead chốt.
   - Thay đổi schema DB đơn hàng, contract API công khai, bất cứ gì gây downtime:
     Tech Lead + 1 review, ghi ADR.
   - Trần thời gian cho mọi bất đồng kiến trúc: 1 tuần. Hết thời hạn -> Tech Lead chốt,
     ghi dissent vào biên bản.

2. GIAO TIẾP
   - Câu hỏi kỹ thuật: kênh chung, không nhắn riêng (để người khác học được và tìm lại được).
   - Kỳ vọng phản hồi: kênh chung trong 4 giờ làm việc; tag trực tiếp trong 1 giờ;
     ngoài giờ làm việc: không kỳ vọng trả lời, trừ khi đang on-call.
   - Muốn hỏi lead một quyết định: gửi kèm "em đã thử gì / có mấy lựa chọn / em nghiêng cái nào".
   - Tin xấu báo sớm không bị phạt. Tin xấu báo muộn là vấn đề.

3. CODE VÀ REVIEW
   - PR <= 400 dòng thay đổi. Lớn hơn thì tách hoặc nói trước lý do.
   - Review đầu tiên trong 4 giờ làm việc. Ai bị tag mà không review được thì nói ra
     trong 30 phút để người khác nhận.
   - Mỗi module có >= 2 người review được (bus factor >= 2). Danh sách công khai.
   - Comment về phong cách: dùng công cụ tự động, không dùng người.
   - Không tự merge vào main khi chỉ có một cặp mắt, trừ hotfix có người xác nhận sau.

4. NHỊP LÀM VIỆC
   - Standup 10 phút, 9:15, nội dung: cái gì đang chặn tôi, tôi đang chặn ai.
     KHÔNG báo cáo tiến độ (đã có trên bảng).
   - Không họp vào buổi chiều thứ Ba và thứ Năm (khối thời gian tập trung của cả team).
   - Sprint review: demo trên môi trường thật, có số liệu người dùng nếu có.
   - Retro 45 phút mỗi 2 tuần. Mỗi retro chốt tối đa 2 action item, có tên người và ngày.
     Action item của retro trước chưa xong -> không nhận thêm action mới.

5. WIP VÀ ƯU TIÊN
   - Mỗi người tối đa 1 việc đang làm + 1 việc chờ review. Muốn nhận việc thứ ba: nói ra.
   - Thay đổi ưu tiên giữa sprint: chỉ PO đề nghị, và phải nói rõ việc gì bị đẩy ra.

6. ON-CALL VÀ SỰ CỐ
   - Luân phiên theo tuần, không ai on-call hai tuần liền.
   - Ai on-call thì tuần đó không nhận việc có deadline.
   - Sự cố: có Incident Commander, mọi người khác theo chỉ thị. Postmortem trong 3 ngày,
     blameless, đối tượng phân tích là hệ thống.

7. NGHỈ VÀ BÀN GIAO
   - Nghỉ >= 3 ngày: bàn giao viết ra, có người nhận tên cụ thể.
   - Không ai là người duy nhất biết một quy trình vận hành. Phát hiện ra thì đó là
     việc ưu tiên của tuần sau.
```

**Bước 3 — Nhịp làm việc: ceremony nào giữ, ceremony nào bỏ.** Tiêu chí duy nhất: mỗi ceremony phải
trả lời được một câu hỏi mà nếu không có nó thì không ai trả lời được. Nếu câu hỏi đó đã được trả lời
ở nơi khác, ceremony là nghi lễ và phải bỏ — vì nó không trung tính, nó tiêu thời gian tập trung và dạy
team rằng thời gian của họ không có giá.

| Ceremony | Câu hỏi nó trả lời | Bỏ hoặc đổi khi | Dấu hiệu nó đã thành nghi lễ |
|---|---|---|---|
| **Standup** | Ai đang bị chặn, và tôi đang chặn ai? | Team ≤ 3 người và ngồi cùng nhau; hoặc công việc gần như không phụ thuộc nhau → chuyển sang async | Mỗi người đọc lại nội dung đã có trên bảng; lead là người duy nhất hỏi; kéo dài hơn 15 phút |
| **Sprint planning** | Hai tuần tới cam kết cái gì, và cái gì bị bỏ ra? | Team đã chạy flow liên tục và ưu tiên rõ theo tuần → thay bằng grooming ngắn hàng tuần | Ước lượng được điền để cho đủ; không ai nhắc tới việc gì bị bỏ ra |
| **Retro** | Cái gì trong cách làm việc của chúng ta cần đổi? | Không bao giờ bỏ, nhưng giảm tần suất nếu action item không được thực thi — vấn đề khi đó là thực thi, không phải tần suất | Cùng một vấn đề xuất hiện lần thứ ba; action item không có tên người; chỉ nói về team khác |
| **Sprint review / demo** | Cái ta làm có đúng cái người dùng cần không? | Không bỏ nếu có người dùng thật. Trong ODC: đây là ceremony có giá trị cao nhất vì nó là kênh duy nhất nối engineer với người dùng | Demo bằng slide thay vì bằng hệ thống chạy; không có ai ngoài team dự |
| **Design review** | Thiết kế này sẽ gãy ở đâu? | Không bỏ cho quyết định irreversible; bỏ cho thay đổi trong một module có người sở hữu rõ | Chỉ có Senior nói; không ai nêu failure mode; kết thúc bằng "ổn rồi" |
| **1-1** | Người này đang ở đâu, và cái gì đang chặn họ? | Không bỏ. Giảm tần suất với người ở mức delegation 6–7, nhưng không dưới 1 lần/tháng | Trở thành báo cáo tiến độ; bị huỷ ba lần liên tiếp khi bận (đây là tín hiệu về ưu tiên, và team đọc được nó) |

**Bước 4 — Onboarding người mới vào một team đang chạy.** Điều kiện biên khó nhất: không có thời gian
dừng lại để dạy, và người mới không có bộ nhớ chung của team.

```
NGÀY 1        Môi trường chạy được trên máy người mới (chuẩn bị TRƯỚC khi họ đến;
              nếu ngày 1 mất vào cài đặt, đó là lỗi của team, không phải của họ)
              Chỉ định buddy — không phải lead. Buddy có nghĩa vụ chủ động hỏi mỗi ngày,
              không phải chờ được hỏi.
NGÀY 2-3      PR đầu tiên lên production: nhỏ, thật, có người dùng thật (sửa một
              text, thêm một log, sửa một bug đã được xác định). Mục tiêu không phải
              giá trị của PR — mục tiêu là đi hết vòng commit -> review -> deploy để
              biết đường và để có một thành công sớm.
TUẦN 1        Đọc 3 thứ, theo thứ tự: working agreement, 5 ADR gần nhất (để hiểu vì sao
              hệ thống như hiện tại), 3 postmortem gần nhất (để hiểu hệ thống hay gãy ở đâu).
              Gặp 1-1 với từng thành viên, 20 phút, câu hỏi cho mỗi người: "phần nào của
              hệ thống anh/chị sở hữu, và cái gì làm anh/chị lo nhất về nó?"
TUẦN 2-4      Một task vừa tầm trong một module có buddy sở hữu. Mức delegation 3-4.
              Tiêu chí ngày 30: tự tạo được PR có ý nghĩa, biết hỏi ai về vùng nào,
              đã tham gia một buổi design review và đặt ít nhất một câu hỏi.
NGÀY 60       Sở hữu được một phần nhỏ (một module con, một job). Mức delegation 5.
              Tiêu chí: xử lý được một ticket từ đầu đến production không cần cầm tay.
NGÀY 90       Có thể nhận on-call cùng một người khác. Tiêu chí: đọc được dashboard,
              biết runbook ở đâu, đã từng chẩn đoán một vấn đề thật.
              Nếu ngày 90 chưa đạt: đây là tín hiệu về onboarding hoặc về fit, và cần
              nói thẳng ở thời điểm này chứ không phải ở kỳ review đầu tiên.

VIỆC CỦA LEAD Ở TUẦN 1: nói ra hai thứ không có trong tài liệu — (1) các quyết định
              lịch sử và vì sao chúng đã hợp lý ở thời điểm đó (tránh việc người mới
              phán xét hệ thống và mất lòng người đã xây nó); (2) bản đồ ai biết cái gì.
```

Một chi tiết về động lực học nhóm: khi một người mới vào, **team lùi lại một phần về Forming**, và
điều đó là bình thường chứ không phải dấu hiệu người mới có vấn đề. Nên nói ra điều này với cả team,
và nên lặp lại một số việc của giai đoạn Forming: đọc lại working agreement, làm rõ lại ranh giới
ownership, tăng tạm thời tần suất đồng bộ trong 3–4 tuần.

### Trade-off

| Trục | Nghiêng về bên A khi | Nghiêng về bên B khi |
|---|---|---|
| **Team lớn (A) vs chia nhỏ (B)** | Công việc có tính phụ thuộc cao, không tách được ranh giới rõ; cần đủ người để có bus factor ≥ 2 cho nhiều module | Vượt 8 người; có thể vạch ranh giới miền rõ ràng; chi phí phối hợp đã hiện ra dưới dạng họp nhiều và quyết định chậm |
| **Chuyên môn hoá (A) vs dự phòng/bus factor (B)** | Miền cần độ sâu (thanh toán, bảo mật, tối ưu hiệu năng); người đó đang phát triển năng lực sâu | Có module bus factor = 1; người sở hữu sắp nghỉ phép dài hoặc có nguy cơ rời đi; hệ thống chạm vào tiền hoặc compliance |
| **Ổn định thành viên (A) vs luân chuyển (B)** | Team đang ở Performing và có mục tiêu quan trọng trong quý; bộ nhớ chung đang là tài sản chính | Có người bị mắc trong một loại việc quá lâu (nguy cơ rời đi vì thiếu Mastery); cần phá bus factor = 1; cần lan tri thức giữa các team |
| **Đồng bộ (A) vs bất đồng bộ (B)** | Giai đoạn Forming/Storming; quyết định phức tạp nhiều bên; cần xây shared mental model | Team đã Norming; công việc ít phụ thuộc; có nhiều múi giờ (khách EU/Mỹ); cần bảo vệ khối thời gian tập trung |

Ai chịu phần mất: khi chọn chuyên môn hoá, phần mất là rủi ro tập trung, và người chịu là chính người
chuyên sâu đó — họ không được đi nghỉ, không được đổi việc, và cuối cùng rời đi vì kiệt sức hoặc vì
nhàm. Khi chọn ổn định thành viên quá lâu, phần mất do người bị mắc trong một loại việc chịu, và tổ
chức chỉ nhìn thấy nó dưới dạng một đơn xin nghỉ bất ngờ.

### Real-world Scenarios

**Tình huống A — chẩn đoán và sửa team logistics 8 người.** Tech Lead Nam đo trước khi can thiệp, và
đây là điểm quan trọng nhất của tình huống: hầu hết lead ở vị trí này bắt đầu bằng việc thay đổi quy
trình theo cảm nhận. Số liệu Nam thu được trong hai tuần: WIP = 19 việc đang mở / 8 người = 2,4; số
giờ họp trung bình 11 giờ/người/tuần; thời gian trung vị từ khi PR mở tới review đầu tiên là 19 giờ
làm việc, và 72% review đến từ Dũng; ba module có bus factor = 1; tỉ lệ thời gian chờ so với thời gian
làm của một ticket là khoảng 4:1.

Kết luận đọc từ số: ràng buộc không phải năng lực viết code, mà là **thời gian chờ ở hai hàng đợi** —
hàng đợi review (một server tên Dũng) và hàng đợi chuyển ngữ cảnh (WIP 2,4). Bốn can thiệp theo thứ tự
tỉ lệ hiệu quả/chi phí, thực hiện trong 6 tuần, mỗi tuần một thay đổi để đo được cái gì có tác dụng:

1. **Giới hạn WIP** = 1 việc đang làm + 1 chờ review mỗi người. Chi phí gần bằng 0. Kết quả sau 2
   tuần: thời gian hoàn thành trung vị của một ticket giảm khoảng 35% mà không ai làm nhanh hơn — đúng
   như Little's Law dự đoán. Đây gần như luôn là can thiệp đầu tiên nên làm.
2. **Phân vùng ownership và người review theo vùng**: mỗi module có hai người review được, danh sách
   công khai trong working agreement. Kết quả: tỉ lệ review của Dũng giảm từ 72% xuống 41% trong 4
   tuần; thời gian chờ review trung vị giảm còn 6 giờ.
3. **Cắt họp**: bỏ một họp trạng thái tuần (nội dung đã có trên bảng), rút standup còn 10 phút và đổi
   nội dung sang "cái gì chặn tôi, tôi chặn ai", chặn hai buổi chiều không họp. Từ 11 xuống còn 5,5
   giờ/người/tuần.
4. **Xử lý xung đột Dũng–Phúc** theo quy trình ở chủ đề 5, và ghi trần một tuần cho bất đồng kiến trúc
   vào working agreement.

Sau 10 tuần, throughput đo bằng số ticket có giá trị hoàn thành mỗi tuần tăng khoảng 70%, không tuyển
thêm ai và không ai làm thêm giờ. Bài học về cơ chế: phần lớn năng lực bị mất của một team nằm ở
**thời gian chờ và chi phí phối hợp**, không nằm ở tốc độ của các cá nhân — nên các can thiệp có hiệu
lực cao nhất là các can thiệp vào hàng đợi, và chúng thường rẻ và trông không giống "quản lý".

**Tình huống B — Hero culture.** Ở fintech, Khoa (infra) là người duy nhất hiểu toàn bộ hệ thống
triển khai. Mỗi lần có sự cố, Khoa xử lý trong 15 phút. Khoa được khen, được thưởng, và trong hai năm
liền là người được nhắc đến nhiều nhất trong các buổi tổng kết. Đọc từ góc độ hệ thống, đây là một
cấu hình thất bại đang được củng cố tích cực: bus factor = 1 ở vùng rủi ro cao nhất; những người khác
không có cơ hội học (Khoa luôn xử lý trước khi họ kịp thử) nên năng lực của họ không tăng; và cơ chế
khen thưởng đang trả tiền cho việc **giữ** trạng thái đó. Thêm nữa, Khoa không thể đi nghỉ, và khi
Khoa cuối cùng chuyển sang platform team, ba tháng sau đó là ba tháng tệ nhất của team.

Script — **phiên bản nói sai** (Minh nói trong buổi tổng kết quý):

> "Quý này phải nói là nhờ Khoa rất nhiều. Hai lần hệ thống có vấn đề lúc nửa đêm, Khoa dậy xử lý
> trong 15 phút. Không có Khoa thì không biết thế nào. Mọi người học tinh thần trách nhiệm của Khoa."

Cơ chế phá hoại: câu này khen đúng người nhưng khen **sai hành vi**. Nó tuyên dương việc cứu hoả cá
nhân, biến bus factor = 1 thành một phẩm chất đạo đức, và ngầm định rằng những người không dậy lúc nửa
đêm thì thiếu tinh thần trách nhiệm — trong khi họ không dậy vì họ không có quyền truy cập và không có
runbook. Sau câu này, xác suất có người thứ hai học được vùng của Khoa giảm chứ không tăng.

Script — **phiên bản nói đúng**:

> **Minh:** "Quý này có hai sự cố nửa đêm và cả hai đều do Khoa xử lý trong 15 phút. Anh cảm ơn Khoa,
> và anh muốn nói thẳng: việc đó là một rủi ro của team, không phải một thành tích của team.
>
> Cụ thể là nếu đêm đó Khoa mất điện thoại, mình không có phương án nào. Đó là lỗi thiết kế của anh,
> không phải của Khoa. Nên hai việc trong quý sau, và đây là ưu tiên có tên trong sprint chứ không
> phải việc làm khi rảnh.
>
> Một: Khoa viết runbook cho ba loại sự cố hay gặp nhất — và cách kiểm tra runbook đó có dùng được
> không là Tuấn thực hiện nó trong một buổi diễn tập, Khoa chỉ ngồi xem, không được chạm bàn phím.
> Nếu Tuấn không làm được theo runbook thì runbook chưa xong.
>
> Hai: từ tháng sau on-call luân phiên hai người một ca, Khoa là người thứ hai trong ba ca đầu — tức
> là người kia xử lý, Khoa chỉ hỗ trợ khi được gọi.
>
> Và anh nói rõ điều này với Khoa: việc anh đo Khoa trong quý sau không phải số sự cố Khoa xử lý, mà
> là số người khác xử lý được sự cố mà không cần Khoa. Nếu quý sau Khoa xử lý ít sự cố hơn, đó là
> Khoa làm tốt hơn, không phải kém hơn."

Điểm then chốt: thay đổi thứ được đo và được khen. Hero culture không tồn tại vì có người thích làm
hero; nó tồn tại vì tổ chức đang trả tiền cho vai đó.

**Tình huống C — người mới vào giữa sprint có deadline.** Ở ODC, Linh nhận một Mid mới vào tuần thứ ba
của một sprint đang trễ. Cách xử lý sai và rất phổ biến: giao ngay một task trong đường tới hạn để
"đóng góp được luôn" — kết quả gần như chắc chắn là task đó trễ hơn nếu người cũ làm, người mới có
trải nghiệm đầu tiên là thất bại, và team mất thời gian giải thích nhiều hơn thời gian tiết kiệm được
(Brooks's law ở dạng nhỏ nhất). Cách xử lý được: trong hai tuần đầu, người mới **không** nằm trong
đường tới hạn; họ làm các việc thật nhưng ngoài đường tới hạn (sửa bug đã xác định, thêm test cho
module sẽ phải chạm tới, viết lại một phần tài liệu tích hợp), và họ được tính là 0% capacity trong
kế hoạch sprint. Nói con số 0% ra với PO là việc bắt buộc — nếu không nói, người mới sẽ bị tính vào
capacity và cả hai bên đều thất bại theo một cách có thể dự báo trước.

### Best Practices

- **Đo trước khi thay đổi quy trình.** Lý do: bốn con số (WIP, giờ họp/người, thời gian chờ review, bus
  factor theo module) chỉ ra ràng buộc thật, và ràng buộc thật thường không phải cái team đang phàn
  nàn. Thay đổi quy trình theo cảm nhận có xác suất cao là tối ưu ở chỗ không phải ràng buộc.
- **Giới hạn WIP trước mọi can thiệp khác.** Lý do: rẻ nhất, nhanh nhất, và tác động qua Little's Law
  không phụ thuộc vào việc ai đó phải cố gắng hơn.
- **Giữ team ở 5–8 người và thiết kế ranh giới thay vì thêm quy trình.** Lý do: chi phí phối hợp là
  bậc hai; quy trình thêm vào chỉ làm chậm hơn chứ không giảm số kênh.
- **Mỗi module có tối thiểu hai người, và kiểm tra điều đó bằng diễn tập, không bằng danh sách.** Lý
  do: bus factor trên giấy khác bus factor thật; cách duy nhất để biết là để người thứ hai thực hiện
  trong khi người thứ nhất ngồi im.
- **Viết working agreement, và xem lại nó mỗi lần team thay đổi.** Lý do: nó chuyển kỳ vọng ngầm thành
  cam kết chỉ vào được, và phần lớn process conflict là xung đột giữa hai giả định không được nói ra.
- **Coi mỗi lần thay đổi thành viên là một lần lùi về Forming, và xử lý chủ động.** Lý do: nếu không
  làm rõ lại ranh giới và chuẩn, team sẽ tự tìm lại chúng bằng cách va vào nhau — chậm hơn và tốn quan
  hệ hơn.
- **Bảo vệ khối thời gian tập trung như một tài sản của team.** Lý do: một cuộc họp giữa buổi chiều
  không tốn một giờ, nó tốn cả buổi chiều của n người. Chặn lịch theo team, không theo cá nhân.

### Anti-patterns

- **Hero culture.** Cơ chế phá hoại: hệ thống khen thưởng việc cứu hoả cá nhân, nên nó nhận được nhiều
  cứu hoả và ít phòng ngừa hơn; đồng thời hero chặn vòng học của những người khác và trở thành ràng
  buộc của toàn hệ thống. Ở tầng động lực, chính hero là nguồn của social loafing ở phần còn lại
  (sucker effect đảo chiều: khi có người luôn gánh, người khác giảm nỗ lực một cách duy lý). Dấu hiệu
  sớm: tên một người xuất hiện trong hầu hết các lần xử lý sự cố; người đó không đi nghỉ trong 12
  tháng; các buổi tổng kết khen "tinh thần trách nhiệm" thay vì khen việc giảm rủi ro.
- **Bus factor = 1 được chấp nhận như một thực tế.** Cơ chế: rủi ro tích luỹ âm thầm và chỉ hiện ra ở
  thời điểm tệ nhất (người đó nghỉ, ốm, hoặc nhận offer khác). Dấu hiệu sớm: câu "cái đó chỉ có anh X
  làm được" xuất hiện trong cuộc họp mà không ai coi đó là vấn đề cần đưa vào backlog.
- **Meeting overload.** Cơ chế: mỗi ceremony được thêm vào để giải quyết một vấn đề phối hợp cụ thể và
  không bao giờ bị bỏ khi vấn đề đó hết; tổng thời gian tập trung sâu giảm xuống dưới ngưỡng cần cho
  công việc phức tạp, và team phản ứng bằng cách làm việc ngoài giờ — điều này bị đọc là cam kết cao.
  Dấu hiệu sớm: hơn 8 giờ họp/người/tuần; không có khối thời gian nào dài hơn 2 giờ không bị chia cắt;
  có người mang laptop vào họp để làm việc khác.
- **Tuckman như một nhãn dán để không phải làm gì.** Cơ chế: "team đang Storming, chờ qua giai đoạn"
  chuyển một chẩn đoán thành một lý do trì hoãn, và Storming không tự kết thúc — nó chuyển thành
  relationship conflict. Dấu hiệu sớm: giai đoạn được nhắc đến trong cuộc họp nhưng không có can thiệp
  nào gắn với nó.
- **Tổ chức lại team theo mỗi quý.** Cơ chế: mỗi lần tái cấu trúc phá bộ nhớ chung (shared mental model
  và transactive memory) và reset về Forming; nếu chu kỳ tái cấu trúc ngắn hơn thời gian xây lại bộ
  nhớ, team không bao giờ đạt Performing. Dấu hiệu sớm: không ai chắc mình sở hữu module nào; mọi người
  nói "trước đây" nhiều hơn nói "chúng ta".
- **Team ảo — nhóm hành chính được gọi là team.** Sáu người báo cáo cùng một quản lý, làm sáu việc
  không liên quan, họp chung mỗi ngày. Cơ chế: áp các thực hành đòi hỏi interdependence lên một nhóm
  không có interdependence tạo ra toàn bộ chi phí phối hợp và không có lợi ích nào. Dấu hiệu sớm: trong
  standup, không ai có thông tin liên quan đến ai.
- **Đo năng suất bằng đầu ra cá nhân.** Cơ chế: khi số commit hay số story point cá nhân được đo, hành
  vi tối ưu cá nhân là tránh việc giúp người khác, tránh review, tránh việc không sinh ra artifact —
  tức là phá chính xác các hành vi làm team hoạt động. Dấu hiệu sớm: có người từ chối pairing với lý do
  "em còn task của em".

### Khi nào KHÔNG nên áp dụng

- **Khi interdependence thật của công việc thấp.** Nếu sáu người làm sáu hệ thống độc lập cho sáu khách
  hàng, các thực hành team (standup chung, retro chung, sprint chung) tạo ra chi phí phối hợp mà không
  tạo giá trị. Việc đúng là giảm phối hợp xuống mức tối thiểu, giữ 1-1 và một cơ chế chia sẻ tri thức
  định kỳ, và không gọi nó là team chỉ vì nó nằm trên cùng một dòng trong sơ đồ tổ chức.
- **Khi team dưới 3 người.** Working agreement 7 mục, ceremony đầy đủ, và ma trận ownership cho hai
  người là nghi lễ. Với nhóm rất nhỏ, giao tiếp trực tiếp có băng thông đủ; cái duy nhất vẫn cần viết
  ra là bus factor và bàn giao khi nghỉ.
- **Khi team chỉ tồn tại vài tuần.** Một task force lập ra để xử lý một vấn đề trong ba tuần không cần
  đi qua vòng đời Tuckman; nó cần chế độ directing rõ ràng, vai xác định từ ngày đầu, và một ngày kết
  thúc. Đầu tư xây văn hoá nhóm ở đây không có người thu hoạch.
- **Trong incident hoặc trong hai tuần trước một mốc hợp đồng cứng.** Không tái cấu trúc, không đổi
  ceremony, không luân chuyển ownership, không onboard người mới vào đường tới hạn. Mọi thay đổi cấu
  trúc đều có chi phí chuyển đổi và chi phí đó đến ngay lập tức trong khi lợi ích đến muộn.
- **Khi luân chuyển sẽ đặt người chưa đủ năng lực vào vùng rủi ro không phục hồi được.** Phá bus
  factor = 1 là mục tiêu đúng, nhưng cách làm phải là runbook + diễn tập + on-call đôi, không phải giao
  thẳng quyền vận hành hệ thống thanh toán cho người mới học trong tuần này. Ở các miền có ràng buộc
  compliance, còn có giới hạn pháp lý về ai được truy cập cái gì.
- **Khi vấn đề thật nằm ở tầng tổ chức, không ở tầng team.** Nếu team liên tục bị đổi ưu tiên bởi ba
  stakeholder không thống nhất với nhau, mọi cải thiện nội bộ đều bị hấp thụ hết. Working agreement
  không sửa được việc đó; việc đúng là đưa vấn đề lên đúng tầng (`09-to-chuc-va-scaling.md`) và trong
  lúc chờ, dựng một hàng đợi duy nhất có một người chịu trách nhiệm ưu tiên.

---

## Tự kiểm tra

Bảy câu dưới đây chỉ có giá trị nếu bạn trả lời bằng **tên người cụ thể** hoặc **con số cụ thể** lấy
từ team thật của bạn. Câu trả lời dạng "chắc là ổn" tương đương với không có câu trả lời.

1. **Delegation.** Mở Jira/board của bạn ngay bây giờ. Bao nhiêu ticket đang mở có bạn là assignee
   hoặc reviewer? Con số đó chia tổng số ticket đang mở bằng bao nhiêu phần trăm? Và với mỗi người
   trong team, bạn đang giao ở mức nào trên thang 7 mức — bạn đã **nói ra** mức đó với họ chưa, hay
   họ đang phải đoán?

2. **Mentoring và Sponsorship.** Kể tên người bạn đang mentor với một mục tiêu và một thời hạn viết ra
   được. Nếu không có tên, đó là câu trả lời. Câu thứ hai, khó hơn: quý vừa rồi bạn đã **nói tên ai,
   với ai, cho việc gì** — tức là bạn đã sponsor ai? Nếu không nêu được tên người nghe và việc cụ thể,
   bạn chưa sponsor ai cả.

3. **Coaching.** Trong 10 câu hỏi kỹ thuật gần nhất bạn nhận được, bao nhiêu câu ở dạng "em nên làm
   gì" và bao nhiêu câu ở dạng "em đang nghiêng X vì A, B, rủi ro là C"? Tỉ lệ đó là thước đo trực tiếp
   việc bạn đang huấn luyện sự phụ thuộc hay sự tự chủ. Và tuần trước, bạn đã đưa giải pháp trong 10
   phút đầu bao nhiêu lần?

4. **Motivation.** Chọn người bạn lo nhất về động lực. Trong năm biến — Clarity, Capability, Autonomy,
   Fairness, Meaning — biến nào đang thấp? Bằng chứng cụ thể nào (một câu họ đã nói, một hành vi đã
   thay đổi từ khi nào) khiến bạn chọn biến đó thay vì bốn biến kia? Nếu bạn không có bằng chứng, mọi
   can thiệp của bạn đang là phỏng đoán.

5. **Conflict Resolution.** Có bất đồng nào trong team bạn đã kéo dài hơn ba tuần mà chưa được chốt?
   Tên hai người, và ngày nó bắt đầu. Nó đang ở loại nào — task, process, hay đã chuyển thành
   relationship? Nếu bạn không biết ngày nó bắt đầu, nó đã kéo dài hơn bạn nghĩ.

6. **Psychological Safety.** Lần gần nhất một người trong team nói với bạn "em không hiểu" hoặc "em
   đã làm sai" là ai, ngày nào? Lần gần nhất **bạn** nói một trong hai câu đó trước mặt team là khi
   nào? Và trong ba lần cuối có người nêu một rủi ro, bao nhiêu lần bạn phản hồi trong 24 giờ kèm một
   quyết định rõ ràng?

7. **Team Dynamics.** Bốn con số, lấy ngay hôm nay: WIP (số việc đang mở chia số người), số giờ họp
   trung bình mỗi người mỗi tuần, thời gian trung vị từ khi PR mở tới review đầu tiên, và danh sách
   module có bus factor = 1 kèm tên người duy nhất biết mỗi module. Nếu bạn không lấy được bốn con số
   này trong 30 phút, đó cũng là một kết quả chẩn đoán.

---

## Liên kết chương khác

- [`00-nen-tang-leadership.md`](/series/engineering-leadership/00-nen-tang-leadership/) — ranh giới Accountability vs Responsibility
  vs Ownership, nền của toàn bộ chương này: Delegation chuyển Responsibility nhưng không chuyển
  Accountability, và mọi tình huống ở đây đều quay về ranh giới đó.
- [`01-self-leadership.md`](/series/engineering-leadership/01-self-leadership/) — quản lý năng lượng và sự chú ý của chính bạn.
  Delegation và Coaching đều đòi băng thông; một lead ở utilization 95% không thực hiện được bất kỳ
  thực hành nào trong chương này, kể cả khi hiểu hết.
- [`02-communication.md`](/series/engineering-leadership/02-communication/) — mô hình đưa feedback, illusion of transparency, và
  cách viết để người đọc hành động. Mọi script trong chương này là ứng dụng của các nguyên lý ở đó.
- [`04-decision-making.md`](/series/engineering-leadership/04-decision-making/) — Reversible vs Irreversible, ADR, và cách chọn
  người quyết. Conflict Resolution ở chủ đề 5 chỉ hoạt động khi câu hỏi "ai có accountability cho
  quyết định này" đã có câu trả lời từ chương đó.
- [`05-technical-leadership.md`](/series/engineering-leadership/05-technical-leadership/) — chuẩn kỹ thuật, RFC, review kiến trúc,
  quản lý Technical Debt. Đây là nội dung mà Delegation phân quyền lên và là chất liệu của phần lớn
  task conflict.
- [`06-incident-va-metrics.md`](/series/engineering-leadership/06-incident-va-metrics/) — vận hành incident, postmortem blameless,
  DORA và SPACE. Là điều kiện biên xuất hiện trong cả bảy chủ đề: trong incident, Delegation, Coaching
  và thảo luận mở đều đổi chế độ.
- [`08-hiring-va-phat-trien.md`](/series/engineering-leadership/08-hiring-va-phat-trien/) — Career Ladder, Performance Review, và
  xử lý underperformance. Là nơi tiếp nhận các trường hợp mà chẩn đoán ở chương này kết luận không
  phải vấn đề kỹ năng hay động lực.
- [`09-to-chuc-va-scaling.md`](/series/engineering-leadership/09-to-chuc-va-scaling/) — Team Topologies, Conway's Law, chia team khi
  vượt ngưỡng phối hợp. Là bước tiếp theo khi các can thiệp ở chủ đề 7 đã đạt trần.
- [`10-case-studies.md`](/series/engineering-leadership/10-case-studies/) — các case dài đi hết chuỗi bối cảnh → lựa chọn →
  trade-off → hậu quả, trong đó có phiên bản mở rộng của tình huống team logistics và xung đột kiến
  trúc ở chương này.
- [`12-anti-patterns.md`](/series/engineering-leadership/12-anti-patterns/) — danh mục hợp nhất các anti-pattern kèm dấu hiệu sớm;
  Hero Culture, bus factor = 1, micromanagement và team building thay cho sửa cấu trúc được phân tích
  sâu hơn ở đó.
