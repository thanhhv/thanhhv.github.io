+++
title = "Level 4A — Project & Delivery: Agile, Estimation, Roadmap và Risk Management"
date = "2026-08-01T15:00:00+07:00"
draft = false
tags = ["engineering-leadership", "management", "career"]
series = ["Engineering Leadership & Technical Management"]
+++

# Level 4A — Project & Delivery

Ngày 12 tháng 9, một product startup Việt ở Series A. Dự án "Tính năng thanh toán trả góp" được chốt
launch ngày 30 tháng 11. Sáng 12 tháng 11 — còn 18 ngày — Minh, Tech Lead, mở buổi họp tiến độ hàng
tuần và báo: "Bọn em đang ở khoảng 85%." Trang, PO, ghi vào slide gửi founder. Ngày 25 tháng 11, con
số vẫn là 85%. Ngày 28, Minh nói thật: cần thêm bốn tuần.

Nhưng dự án đó không trễ vào ngày 25 tháng 11. Nó trễ từ ngày 8 tháng 9 — bốn ngày trước khi bắt đầu.
Hôm đó, trong buổi kick-off, có ba việc xảy ra mà không ai ghi lại. Một: khi Trang hỏi "hai tháng đủ
không", Minh trả lời "đủ, nếu mọi thứ suôn sẻ" — và cả phòng nghe thành "đủ". Hai: phần tích hợp với
đối tác trả góp được ghi trong kế hoạch là "chờ đối tác cung cấp sandbox", không có ngày, không có
người chịu trách nhiệm theo đuổi. Ba: hai trong năm engineer của dự án cũng đang là người duy nhất
biết hệ thống đối soát cũ, và tháng 11 là tháng chốt sổ. Cả ba việc đó đều đã biết vào ngày 8 tháng 9.
Không ai nói ra, và không có cơ chế nào bắt buộc nói ra.

Đó là hình dạng thật của "dự án trễ". Phần lớn độ trễ được nạp vào hệ thống ở tuần đầu, dưới dạng
những giả định không được kiểm tra, những phụ thuộc không có chủ, và những con số được phát ngôn như
cam kết nhưng được nghĩ như hy vọng. Phần còn lại của dự án chỉ là quá trình độ trễ đó dần trở nên
không thể che được nữa. Việc bạn làm ở tuần 10 gần như không thay đổi kết cục; việc bạn làm ở tuần 1
thay đổi gần như toàn bộ.

Delivery là tầng mà mọi quyết định ở các tầng trên bị kiểm chứng bằng thực tế. Một chiến lược kỹ thuật
đúng nhưng không giao được thì không phân biệt được với một chiến lược sai. Một team có Psychological
Safety cao nhưng không có cơ chế phát hiện trượt tiến độ vẫn sẽ trượt — chỉ là mọi người nói với nhau
thoải mái hơn trong lúc trượt. Ngược lại, một cơ chế delivery tốt là nơi các khiếm khuyết ở tầng trên
hiện ra sớm và rẻ: yêu cầu không rõ hiện ra ở sprint planning thứ nhất chứ không phải ở UAT; phụ thuộc
không kiểm soát hiện ra ở bản đồ dependency chứ không phải ở tuần cuối; nợ kỹ thuật hiện ra thành cycle
time tăng chứ không phải thành một incident lúc 3 giờ sáng.

Trong chuỗi Business Goal → People → Process → Technology → Execution → Feedback → Improvement →
Scaling Team → Organization, chương này chiếm trọn mắt **Process** và phần lớn mắt **Execution**. Nó
là bộ máy biến ý định thành thứ chạy được trong production, và biến thực tế production thành thông tin
để sửa ý định. Chương 04 nói về cách ra một quyết định tốt. Chương này nói về việc hàng trăm quyết định
nhỏ trong một quý được sắp xếp, giới hạn, đo đếm và điều chỉnh như thế nào để tổng của chúng không phải
là một cuộc chạy nước rút tháng cuối.

Mục lục nội bộ:

1. [Agile như một cơ chế xử lý bất định, không phải một bộ nghi lễ](#1-agile-như-một-cơ-chế-xử-lý-bất-định-không-phải-một-bộ-nghi-lễ)
2. [Scrum trong thực tế](#2-scrum-trong-thực-tế)
3. [Kanban và quản lý dòng chảy](#3-kanban-và-quản-lý-dòng-chảy)
4. [Estimation](#4-estimation)
5. [Roadmap và Planning](#5-roadmap-và-planning)
6. [Dependency Management](#6-dependency-management)
7. [Risk Management trong delivery](#7-risk-management-trong-delivery)
8. [Delivery dưới ràng buộc: cắt cái gì khi không đủ thời gian](#8-delivery-dưới-ràng-buộc-cắt-cái-gì-khi-không-đủ-thời-gian)
9. [Tự kiểm tra](#tự-kiểm-tra)
10. [Liên kết chương khác](#liên-kết-chương-khác)

> Mọi con số trong chương này là **số minh hoạ** để bạn thấy cách tính, cách đặt ngưỡng và cách đọc
> tổ hợp, không phải dữ liệu nội bộ của bất kỳ công ty nào. Khi áp dụng, thay bằng số đo được từ hệ
> thống của bạn — và nếu chưa đo được thì việc đầu tiên là đo, không phải chọn framework.

---

## 1. Agile như một cơ chế xử lý bất định, không phải một bộ nghi lễ

### Problem Statement

Một ODC ở TP.HCM, 40 engineer, khách hàng Nhật. Trên tường có poster "We are Agile". Lịch tuần của một
team 7 người như sau: standup 15 phút mỗi ngày (thực tế 35 phút), sprint planning 2 giờ thứ Hai,
grooming 1,5 giờ thứ Tư, sprint review 1 giờ thứ Sáu, retro 1 giờ thứ Sáu. Tổng nghi lễ: khoảng 8,4
giờ/người/tuần trên 40 giờ — 21% capacity (số minh hoạ, tính từ lịch thật của một team dạng này).

Bây giờ là phần đáng chú ý. Scope của dự án được chốt trong SOW đã ký từ tháng trước, gồm 214 user
story đã được khách viết chi tiết đến mức có cả text của thông báo lỗi. Deadline chốt trong hợp đồng.
Ngân sách chốt. Trong 6 sprint đã chạy, số story bị thay đổi nội dung sau khi vào sprint: 3. Số story
bị bỏ khỏi scope: 0. Số story được thêm vào do học được điều gì mới: 0.

Team này đang trả 21% capacity cho một bộ cơ chế được thiết kế để xử lý sự thay đổi, trong một dự án
gần như không có sự thay đổi. Đó không phải Agile. Đó là waterfall có standup.

Hiện tượng đối xứng cũng phổ biến không kém, và tốn hơn. Một startup Việt 18 engineer, sản phẩm B2C
mới, chưa có product-market fit. Họ chạy "waterfall vì cần chắc chắn": một bản spec 60 trang, kế hoạch
5 tháng, ba milestone. Tháng thứ 4, họ demo cho 30 khách hàng đầu tiên và phát hiện luồng đăng ký sai
mô hình kinh doanh — thứ mà một prototype 2 tuần ở tháng thứ nhất đã phát hiện được. Chi phí của cái
sai đó: 3,5 tháng × 18 người, cộng với 4 tháng lợi thế thời gian trên thị trường.

Hai hiện tượng, một nguyên nhân: chọn cơ chế theo thời trang hoặc theo quán tính, không theo hình dạng
của bất định trong bài toán. Dấu hiệu nhận biết bạn đang ở trong đó, đếm được:

| Dấu hiệu | Cách đo | Ngưỡng đáng lo (số minh hoạ) |
|---|---|---|
| Nghi lễ chiếm quá nhiều capacity mà không đổi hành vi | Giờ họp process / giờ làm việc | > 15% mà không có quyết định nào bị đảo trong 3 sprint |
| Retro không sinh ra thay đổi cơ chế | Số action từ retro đã hoàn thành / tổng | < 40% trong 3 sprint liên tiếp |
| Standup là báo cáo, không phải đồng bộ | Số lần một người nói với người khác (không phải với lead) | < 30% lượt phát ngôn |
| Backlog không hề đổi | Số story bị bỏ hoặc thay thế mỗi sprint | = 0 trong 5 sprint (thì bạn không cần sprint) |
| Kế hoạch dài không hề đúng | Sai số milestone | > 40% ở milestone đầu (thì bạn không nên lập kế hoạch dài) |

Nếu thiếu năng lực này, hậu quả không phải "team không Agile". Hậu quả là: bạn trả chi phí của một cơ
chế mà không nhận được lợi ích của nó, và bạn mất khả năng biện luận với stakeholder vì mọi lựa chọn
process của bạn đều là "vì người ta làm vậy".

### First Principles

**Vòng lặp ngắn không làm bạn đúng hơn. Nó làm cái sai rẻ hơn.**

Đây là toàn bộ nội dung kinh tế học của Agile, và nó thường bị nói thành "Agile giúp làm nhanh hơn" —
một phát biểu sai. Xét một quyết định sản phẩm có xác suất sai là $p$. Chi phí của một lần sai gồm hai
phần: công đã bỏ ra trước khi phát hiện ($W$), và chi phí sửa cấu trúc đã xây trên nền sai ($R$). Cả
$W$ và $R$ đều tăng theo thời gian từ lúc quyết định đến lúc phát hiện — $W$ tăng gần như tuyến tính
theo số người-ngày đã đầu tư, $R$ tăng nhanh hơn tuyến tính vì mỗi lớp code xây trên giả định sai lại
tạo thêm chỗ phải sửa.

Chi phí kỳ vọng của bất định:

```
E[cost] = p × (W(t) + R(t))
```

Có hai cách giảm nó. Cách một: giảm $p$ — nghĩ kỹ hơn, phân tích nhiều hơn, viết spec dày hơn. Cách
hai: giảm $t$ — cắt khoảng thời gian giữa quyết định và phản hồi. Agile là cách hai. Waterfall là cách
một. Không cái nào đúng phổ quát; cái nào rẻ hơn phụ thuộc vào việc $p$ có giảm được không.

Và đó chính là biến quyết định: **$p$ có giảm được bằng cách nghĩ thêm không?**

- Nếu yêu cầu đã cố định và đã đúng (khách đã dùng hệ thống tương tự 10 năm, chỉ cần làm lại; hoặc
  yêu cầu do luật quy định), thì $p$ vốn đã thấp. Nghĩ thêm không giảm được nữa vì không còn gì để
  giảm. Lúc này vòng lặp ngắn chỉ là chi phí thuần: mỗi sprint boundary là một lần overhead lập kế
  hoạch, một lần chuyển ngữ cảnh, một lần đóng gói dở dang. Kế hoạch tuần tự rẻ hơn thật.
- Nếu yêu cầu chưa biết đúng hay sai (sản phẩm mới, hành vi người dùng chưa quan sát được), thì $p$
  không giảm được bằng suy nghĩ, vì thông tin cần thiết không nằm trong phòng họp — nó nằm ngoài thị
  trường. Mọi giờ bỏ vào phân tích thêm chỉ làm bạn tự tin hơn vào một con số không đổi. Lúc này chỉ
  còn cách hai.

Đây là lý do một phát biểu như "Agile tốt hơn Waterfall" là vô nghĩa về mặt cấu trúc: nó so sánh hai
công cụ giảm hai đại lượng khác nhau mà không nói bài toán có đại lượng nào lớn.

**Cơ chế thứ hai: giới hạn nhận thức về tương lai.** Con người không dự báo được kém đều nhau ở mọi
khoảng thời gian. Chúng ta dự báo khá tốt những gì tương tự việc vừa làm (tuần tới), và dự báo rất tệ
những gì phải suy diễn qua nhiều bước (quý sau). Một kế hoạch 6 tháng chi tiết đến tuần không phải là
kế hoạch chính xác hơn kế hoạch 6 tháng chi tiết đến tháng — nó chỉ là cùng độ chính xác được trình
bày với độ tự tin cao hơn. Việc trình bày này gây hại thật: stakeholder ra quyết định (thuê người, ký
hợp đồng marketing, hứa với nhà đầu tư) dựa trên độ chi tiết mà họ đọc thành độ tin cậy.

**Cơ chế thứ ba: batch size và feedback.** Một lô lớn (6 tháng công việc, release một lần) trì hoãn
toàn bộ thông tin về chất lượng đến cuối. Một lô nhỏ trả thông tin liên tục. Điều này không chỉ đúng
với sản phẩm mà đúng với chính team: một team release 2 tuần một lần biết trong 2 tuần rằng quy trình
QA của mình bị nghẽn; một team release 6 tháng một lần biết điều đó sau 6 tháng, khi không còn thời
gian sửa. Chi tiết định lượng của cơ chế này nằm ở chủ đề 3 (Little's Law) và chủ đề 4 (batch size vs
độ chính xác ước lượng).

### Mental Model

**Mô hình 1: Cone of Uncertainty.** Độ bất định của một ước lượng không cố định mà co lại theo lượng
thông tin đã thu được. Ở điểm bắt đầu một dự án loại chưa từng làm, khoảng ước lượng thực tế có thể là
0,5x–2x (số minh hoạ, theo khoảng thường được nhắc trong tài liệu công khai về software estimation);
sau khi thiết kế xong, thu về khoảng 0,8x–1,25x; sau khi làm 50% thì hẹp hơn nữa.

```
Sai số ước lượng
   4x |*
      | *
   2x |   *
      |      *
   1x |----------*-----------*-----------*----→ thời gian / thông tin
      |      *
  0.5x|   *
      | *
  0.25x|*
      Kick-off   Thiết kế   50% code   Trước release
```

Hai hệ quả dùng được. Một: hình nón chỉ co lại nếu bạn *làm* — nó không co theo thời gian trôi. Ba
tuần họp phân tích không thu hẹp nón bằng ba ngày prototype. Hai: mọi cam kết bạn phát ngôn nên mang
theo độ rộng của nón tại thời điểm đó. Phát ngôn một con số duy nhất ở kick-off là hành vi che thông
tin, không phải hành vi quyết đoán.

**Mô hình 2: Stacey Matrix.** Đặt bài toán lên hai trục: mức đồng thuận về *cái gì cần làm* (bất định
yêu cầu) và mức chắc chắn về *làm thế nào* (bất định công nghệ). Bốn vùng, bốn cơ chế khác nhau:

| Vùng | Bất định yêu cầu | Bất định công nghệ | Cơ chế phù hợp |
|---|---|---|---|
| Simple | Thấp | Thấp | Kế hoạch tuần tự, checklist, tối ưu throughput. Kanban hoặc waterfall đều được |
| Complicated (kỹ thuật) | Thấp | Cao | Spike / prototype có timebox trước, rồi kế hoạch tuần tự cho phần còn lại |
| Complicated (yêu cầu) | Cao | Thấp | Vòng lặp ngắn có demo cho người dùng thật. Đây là vùng Scrum sinh ra để giải |
| Complex / Chaotic | Cao | Cao | Lô cực nhỏ, timebox cứng, sẵn sàng bỏ. Discovery track riêng, không cam kết ngày |

Sai lầm thường gặp nhất là dùng cơ chế của vùng "Complicated (yêu cầu)" cho vùng "Simple" — đó chính
là ODC ở phần Problem Statement. Sai lầm đắt nhất là dùng cơ chế của "Simple" cho vùng "Complex" — đó
là startup viết spec 60 trang.

**Mô hình 3: Inspect–Adapt Loop.** Mọi cơ chế Agile là một vòng feedback ba thành phần: một tín hiệu
(signal), một chu kỳ đọc tín hiệu (cadence), và một quyền thay đổi (authority to adapt). Bỏ bất kỳ
thành phần nào thì vòng lặp chết, và cái còn lại là nghi lễ:

- Có tín hiệu + có chu kỳ, không có quyền → team họp đều đặn, nhìn thấy vấn đề, không đổi được gì.
  Đây là dạng phổ biến nhất ở ODC và ở IT nội bộ doanh nghiệp truyền thống. Retro biến thành nơi xả.
- Có chu kỳ + có quyền, không có tín hiệu → đổi liên tục theo cảm tính, mỗi sprint một hướng.
- Có tín hiệu + có quyền, không có chu kỳ → chỉ thay đổi khi có khủng hoảng.

Khi chẩn đoán một tổ chức "làm Agile mà không hiệu quả", hỏi ba câu theo mô hình này trước khi hỏi bất
kỳ câu nào về công cụ hay lịch họp.

### Practical Framework

**Bước 1 — Định vị bài toán trên hai trục.** Với mỗi initiative sắp tới (không phải "team của bạn" —
một team có thể có nhiều initiative ở các vùng khác nhau), trả lời bằng bằng chứng, không bằng cảm
giác:

Bất định yêu cầu — cho điểm 0–3, cộng lại:

- Người dùng cuối đã dùng thứ tương tự chưa? (chưa: +1)
- Có ai trong tổ chức chốt được yêu cầu và không đổi ý trong 8 tuần qua? (không: +1)
- Kết quả kinh doanh của tính năng này đã được kiểm chứng ở đâu đó chưa? (chưa: +1)

Bất định công nghệ — cho điểm 0–3:

- Team đã từng làm dạng tích hợp / dạng hệ thống này chưa? (chưa: +1)
- Có phụ thuộc vào hệ thống mà team không đọc được code hoặc không có tài liệu? (có: +1)
- Có yêu cầu phi chức năng chưa từng đạt (throughput, latency, compliance mới)? (có: +1)

**Bước 2 — Chọn hình dạng vòng lặp theo bảng chẩn đoán:**

| Điểm yêu cầu | Điểm công nghệ | Hình dạng vòng lặp đề xuất | Điều gì phải có |
|---|---|---|---|
| 0–1 | 0–1 | Kanban với WIP limit, không cần sprint | Cycle time đo được, board phản ánh dòng chảy thật |
| 0–1 | 2–3 | Spike timebox 1–2 tuần → kế hoạch tuần tự | Tiêu chí kết thúc spike viết trước, được phép kết luận "không khả thi" |
| 2–3 | 0–1 | Sprint 1–2 tuần, demo cho người dùng thật mỗi vòng | Có người dùng thật xem demo, có quyền bỏ story |
| 2–3 | 2–3 | Lô rất nhỏ, discovery track riêng, không cam kết ngày cho scope | Có ngân sách được phép "đốt" và tiêu chí dừng |

**Bước 3 — Kiểm tra từng nghi lễ theo nguyên tắc "vấn đề trước, nghi lễ sau".** Đây là phần có giá trị
nhất và hầu như không tổ chức nào làm. Mỗi nghi lễ tồn tại để giải một vấn đề cụ thể. Nếu tổ chức của
bạn không có vấn đề đó, nghi lễ là chi phí thuần. Bảng dưới là công cụ audit — chạy nó mỗi quý:

| Nghi lễ | Tồn tại để giải vấn đề gì | Dấu hiệu bạn KHÔNG có vấn đề đó | Thay bằng gì nếu bỏ |
|---|---|---|---|
| Daily standup | Công việc của các thành viên phụ thuộc nhau trong ngày; blocker cần phát hiện trong < 24h | Mọi người làm việc độc lập, blocker luôn được nêu trên Slack trong 1 giờ | Kênh async + quy tắc "blocker báo ngay" |
| Sprint planning | Cần cam kết một lô công việc và bảo vệ nó khỏi thay đổi giữa chu kỳ | Không ai đổi ý giữa chu kỳ; hoặc ngược lại: đổi ý liên tục và không ai bảo vệ được | Kanban pull + refinement liên tục |
| Backlog refinement | Story vào sprint thường chưa đủ rõ để bắt đầu | Story luôn rõ; hoặc dev luôn tự hỏi trực tiếp PO và được trả lời trong 1 giờ | Quy tắc Definition of Ready + PO trực |
| Sprint review / demo | Người ra quyết định sản phẩm không thấy được sản phẩm giữa chu kỳ | Stakeholder dùng staging hàng ngày | Bản ghi demo 5 phút + link staging |
| Retrospective | Cơ chế làm việc có khiếm khuyết mà không có kênh nào sửa | Team đã sửa cơ chế liên tục ngoài retro và có bằng chứng | Không bỏ — đây là nghi lễ ít nên bỏ nhất |
| Estimation session | Cần dự báo cho quyết định bên ngoài team | Không ai dùng con số đó để quyết định gì | Đếm số item, đo cycle time (xem chủ đề 4) |

Nguyên tắc áp dụng: chỉ bỏ nghi lễ khi bạn nêu được vấn đề nó giải và chứng minh được bạn không có vấn
đề đó, **bằng dữ liệu 3 sprint gần nhất**. Bỏ vì "thấy mất thời gian" là cách nhanh nhất để mất luôn
cơ chế feedback.

**Bước 4 — Ghi lại quyết định process như một ADR.** Process là quyết định kiến trúc của hệ thống xã
hội. Ghi: bối cảnh (điểm hai trục), lựa chọn, cái bị từ bỏ, và điều kiện xem lại. Không có nó, sáu
tháng sau không ai nhớ vì sao team này không có standup, và người mới sẽ thêm lại.

**Output và tiêu chí xong:** một trang cho mỗi initiative, ghi điểm hai trục, hình dạng vòng lặp, danh
sách nghi lễ giữ lại kèm vấn đề nó giải. Xong khi bạn trả lời được câu "vì sao team ta họp thứ Sáu 1
giờ" bằng một vấn đề đếm được, không bằng "vì Scrum yêu cầu".

### Trade-off

**Vòng lặp ngắn vs chi phí overhead mỗi vòng.** Mỗi biên chu kỳ có chi phí cố định: lập kế hoạch, đóng
gói, kiểm thử hồi quy, chuyển ngữ cảnh. Giả sử overhead cố định là 6 giờ/người/chu kỳ (số minh hoạ).
Sprint 1 tuần: 6/40 = 15% capacity. Sprint 2 tuần: 7,5%. Sprint 4 tuần: 3,75%. Đổi lại, độ trễ phát
hiện sai lệch tăng theo độ dài chu kỳ. Nghiêng về chu kỳ ngắn khi: yêu cầu hay đổi, khách xem demo
được thường xuyên, chi phí release thấp (CI/CD tốt). Nghiêng về chu kỳ dài khi: chi phí release cao
(release cần UAT của khách, cần cửa sổ triển khai của ngân hàng), yêu cầu ổn định, team phân tán múi
giờ khiến mỗi lần đồng bộ đắt.

**Khả năng thích ứng vs khả năng dự báo.** Đây là trade-off gốc mà mọi trade-off khác trong chương này
là biến thể. Một hệ thống thích ứng tốt là hệ thống chấp nhận đổi hướng, và mỗi lần đổi hướng làm mọi
dự báo trước đó vô hiệu. Không thể có cả hai ở mức cao cùng lúc trên cùng một phạm vi. Điều làm được:
tách phạm vi. Cam kết chắc ở phạm vi hẹp và gần (sprint goal 2 tuần), giữ linh hoạt ở phạm vi rộng và
xa (Later trong roadmap — chủ đề 5). Ai chịu phần mất: nếu bạn chọn thích ứng, bộ phận marketing/sales
chịu vì họ không hứa được ngày; nếu bạn chọn dự báo, team engineering chịu vì họ phải cắt chất lượng
để giữ ngày (chủ đề 8).

**Tự chủ của team về process vs chuẩn hoá toàn tổ chức.** Ở 3 team, mỗi team tự chọn process là tối ưu
cục bộ tốt. Ở 15 team, mỗi team một process làm việc điều phối liên team gần như bất khả thi: không có
đơn vị thời gian chung để đồng bộ, không so sánh được tiến độ, người chuyển team phải học lại. Ngưỡng
thực tế thường vào khoảng 5–8 team (số minh hoạ): dưới ngưỡng ưu tiên tự chủ; trên ngưỡng chuẩn hoá
phần *giao diện* (cadence chung, định dạng báo cáo tiến độ, Definition of Done tối thiểu) và để tự do
phần *bên trong* (team họp thế nào, dùng board gì). Chi tiết ở chương 09.

**Nghi lễ như công cụ vs nghi lễ như bằng chứng tuân thủ.** Ở ODC, nghi lễ có một chức năng thứ hai
hợp pháp: chứng minh với khách rằng có quy trình. Đây là giá trị thật (giữ hợp đồng), không nên phủ
nhận. Nhưng phải gọi đúng tên: nếu standup tồn tại vì khách muốn thấy standup, hãy ghi đúng như vậy
trong ADR process, và tối ưu nó cho mục đích đó (15 phút, có biên bản gửi khách) thay vì giả vờ nó là
cơ chế đồng bộ và thất vọng vì nó không đồng bộ được gì.

### Real-world Scenarios

**Tình huống A — "Agile" trong ODC khi khách chốt scope và deadline từ đầu.**

Bối cảnh: ODC 25 engineer, khách Nhật, dự án 9 tháng làm lại hệ thống quản lý kho. Hợp đồng fixed-price,
scope 214 story trong SOW, deadline hợp đồng. Khách yêu cầu "chạy Scrum" và yêu cầu báo velocity mỗi
sprint. Hà, EM, thấy team kiệt sức vì họp và vì mỗi sprint đều "không đạt velocity cam kết".

Chẩn đoán bằng framework: điểm bất định yêu cầu = 0 (khách đã dùng hệ thống cũ 12 năm, spec chi tiết,
không đổi ý). Điểm bất định công nghệ = 2 (team chưa làm tích hợp với hệ thống ERP của khách, không có
tài liệu). Vùng: Complicated (kỹ thuật). Cơ chế đúng: spike có timebox cho phần tích hợp ERP, phần còn
lại chạy dòng chảy tối ưu throughput.

Điều Hà làm sai lúc đầu: đề nghị khách bỏ Scrum. Khách từ chối, và quan hệ xấu đi một bậc, vì Hà đã
tấn công vào thứ khách dùng để cảm thấy kiểm soát được — mà không đưa gì thay thế.

Điều Hà làm sau đó: giữ nguyên vỏ nghi lễ, đổi ruột. Cụ thể (đây là mẫu dùng được cho nhiều ODC):

- Giữ sprint 2 tuần và báo cáo sprint, vì đó là nhu cầu kiểm soát của khách — chính đáng.
- Đổi nội dung cam kết sprint: từ "velocity 45 điểm" sang "hoàn thành 3 luồng nghiệp vụ có thể UAT".
  Lý do nói với khách: điểm không kiểm được, luồng UAT được thì kiểm được.
- Bỏ estimation session chi tiết cho toàn bộ 214 story (đang tốn khoảng 4 giờ/sprint × 7 người). Thay
  bằng đếm số story theo t-shirt size một lần cho cả release, và đo cycle time thực tế theo tuần.
  Sau 3 sprint có dữ liệu thật thì dự báo bằng dữ liệu, không bằng ước lượng.
- Tách phần tích hợp ERP thành một spike 2 tuần có tiêu chí kết thúc viết trước, và **nói rõ với khách
  rằng nếu spike kết luận cần thay đổi thiết kế thì đó là change request**. Đây là bước quan trọng
  nhất: nó chuyển một rủi ro ẩn thành một mục đàm phán.
- Bỏ grooming định kỳ, thay bằng quy tắc Definition of Ready và một slot 30 phút/ngày PO trực.

Kết quả sau 4 sprint (số minh hoạ): giờ họp process giảm từ khoảng 8,4 xuống 4,5 giờ/người/tuần; khách
vẫn nhận báo cáo sprint đều; spike ERP phát hiện một hạn chế của API khách ở tuần 3 của dự án thay vì
tuần 20, dẫn tới một change request được ký thay vì một tháng trễ không được trả tiền.

Bài học: khi khách hàng ràng buộc process, đòn bẩy của bạn không phải bỏ nghi lễ mà là **đổi nội dung
của cam kết bên trong nghi lễ**, và dùng nghi lễ đó làm kênh đưa rủi ro ra sớm.

**Tình huống B — "Scrum" bị dùng như công cụ theo dõi tiến độ cá nhân.**

Bối cảnh: bộ phận IT của một doanh nghiệp truyền thống, 12 engineer, chuyển đổi số. Giám đốc IT yêu
cầu mỗi engineer cập nhật Jira "còn lại bao nhiêu giờ" mỗi cuối ngày, và burndown được chiếu lên màn
hình lớn ở phòng. Trong standup, mỗi người báo lần lượt cho lead; ai có task quá 3 ngày bị hỏi tại sao
trước cả phòng.

Cơ chế hỏng, theo mô hình Inspect–Adapt: tín hiệu bị nhiễm. Khi tín hiệu được dùng để đánh giá cá
nhân, người tạo tín hiệu có động lực làm tín hiệu đẹp. Cụ thể quan sát được: engineer chia task thành
các mảnh nhỏ hơn thực tế để đóng ticket đều; "còn lại bao nhiêu giờ" luôn giảm tuyến tính vì mọi người
điền theo kế hoạch chứ không theo thực tế; không ai báo blocker sớm vì báo blocker bị đọc thành không
tự giải quyết được việc.

Ba góc nhìn cùng một sự việc — hôm nay Khoa (Senior BE) có một task đã 5 ngày, và trong standup Khoa
nói "vẫn đang xử lý, sắp xong":

- **Nhìn từ IC (Khoa):** tôi mắc ở một race condition trong module đối soát mà tôi chưa hiểu hết. Nói
  ra chi tiết trước 12 người, trong đó có giám đốc IT, có nghĩa là thừa nhận tôi không hiểu code mình
  đang sở hữu. Chi phí uy tín cao, lợi ích thấp — vì tôi biết không ai trong phòng giúp được. Nói "sắp
  xong" là hành vi hợp lý theo động lực tôi đang chịu.
- **Nhìn từ Tech Lead (Minh):** tôi thấy một tín hiệu — cùng một câu trả lời ba ngày liền. Vấn đề của
  tôi không phải Khoa mà là tôi không có kênh nào để Khoa nói thật. Việc đúng: kéo Khoa ra sau standup
  hỏi riêng, không hỏi trước phòng; và về lâu dài, đổi cấu trúc standup từ "báo cáo lần lượt cho lead"
  sang "đi qua từng cột trên board, ai liên quan thì nói" — cách này làm chủ thể của standup là công
  việc chứ không phải con người.
- **Nhìn từ Manager (Hà, EM):** hai tín hiệu, không phải một. Tín hiệu gần: một task rủi ro cần
  unblock. Tín hiệu xa và quan trọng hơn: hệ thống đo của tôi đang tạo ra dữ liệu giả, nghĩa là mọi
  báo cáo tôi gửi lên trong 6 tháng qua đều không đáng tin. Việc đúng: tách hẳn hai kênh — kênh tiến
  độ công việc (board, cấp team, không tên người) và kênh đánh giá năng lực (1-1, chu kỳ quý, không
  dùng số ticket). Và phải đàm phán với giám đốc IT về màn hình burndown, bằng lập luận "số này đang
  không đúng, và tôi giải thích được vì sao", không bằng "làm vậy team mệt".

Điểm quan trọng: cả ba đều nhìn đúng ở tầng của mình, và không tầng nào tự sửa được vấn đề của tầng
khác. Khoa không sửa được cấu trúc standup. Minh không sửa được yêu cầu báo giờ. Hà không sửa được
việc Khoa mắc race condition. Đây là lý do vấn đề process cần được xử lý ở đúng tầng có quyền — nếu
không, mọi bên đều cố gắng và hệ thống vẫn hỏng.

### Best Practices

**Chọn process theo hai trục bất định, và ghi lại lựa chọn kèm điều kiện xem lại.** Lý do: nó chuyển
process từ đối tượng của niềm tin thành đối tượng của lập luận. Khi có người mới muốn thêm nghi lễ,
bạn có văn bản để thảo luận thay vì tranh luận về sở thích.

**Mỗi nghi lễ phải có một câu "tồn tại để giải X" mà cả team đọc được.** Lý do: nghi lễ không có mục
đích rõ sẽ trôi dạt về mục đích mặc định của tổ chức — thường là kiểm soát và báo cáo lên trên, vì đó
là gradient tự nhiên của mọi hệ thống có cấp bậc.

**Tách tín hiệu tiến độ khỏi tín hiệu đánh giá cá nhân, bằng cấu trúc chứ không bằng lời hứa.** Lý do:
Goodhart's law không quan tâm ý định tốt của bạn. Nếu một số có thể ảnh hưởng lương thưởng, nó sẽ bị
tối ưu. Cách tách bằng cấu trúc: metrics tiến độ chỉ tồn tại ở mức team và mức luồng công việc; không
có báo cáo nào có cột "số ticket theo người" đi ra khỏi team.

**Bắt đầu bằng chu kỳ ngắn hơn mức bạn nghĩ là thoải mái, rồi kéo dài nếu đo được overhead quá cao.**
Lý do: sai lệch mặc định của tổ chức là chu kỳ dài (vì mỗi biên chu kỳ gây đau ngay và lợi ích thì
chậm), nên bắt đầu ngắn để bù sai lệch đó. Ngưỡng cụ thể: nếu overhead nghi lễ + release vượt 15%
capacity thì kéo dài chu kỳ.

**Với ODC bị khách ràng buộc process: giữ vỏ, đổi ruột, và dùng nghi lễ làm kênh đưa rủi ro ra.** Lý do
đã phân tích ở tình huống A. Bổ sung: mọi thay đổi ruột phải nói trước với khách bằng lợi ích của
khách ("số này kiểm được, số kia không"), không bằng lợi ích của team.

**Retro là nghi lễ cuối cùng bạn nên bỏ, và là nghi lễ đầu tiên bạn nên đo.** Lý do: nó là thành phần
"authority to adapt" của vòng lặp. Đo bằng một số duy nhất: tỉ lệ action từ retro đã hoàn thành trong
2 sprint. Dưới 40% thì retro của bạn đang là nơi xả, và việc cần sửa là quyền hạn, không phải format.

### Anti-patterns

**Cargo cult agile — sao chép hình thức của một tổ chức khác mà không sao chép điều kiện.** Cơ chế phá
hệ thống: hình thức (standup, sprint, board, thậm chí "Spotify model" với squad/tribe) là phần rẻ và
dễ nhìn; điều kiện làm nó hoạt động (quyền của team về scope, khả năng release nhanh, người dùng thật
xem demo được) là phần đắt và không nhìn thấy. Sao chép phần rẻ tạo ra chi phí mà không tạo ra lợi ích,
và tệ hơn: nó tiêu hết uy tín của từ "Agile" trong tổ chức, làm cho lần cải tiến process tiếp theo khó
bán hơn nhiều. Dấu hiệu sớm: khi hỏi "vì sao ta có nghi lễ này", câu trả lời phổ biến nhất là tên một
công ty khác hoặc tên một chứng chỉ.

**Velocity làm KPI.** Cơ chế: velocity là đơn vị tự định nghĩa bởi chính team được đo. Khi nó trở
thành mục tiêu, cách rẻ nhất để tăng là lạm phát đơn vị — cùng một việc, tuần này ước lượng 5 điểm,
quý sau ước lượng 8 điểm. Không ai nói dối một cách có ý thức; ước lượng chỉ trôi dần. Hệ quả nghiêm
trọng hơn con số sai: team mất khả năng dùng story point cho mục đích gốc của nó (so sánh kích thước
tương đối để lập kế hoạch sprint), vì đơn vị không còn ổn định. Dấu hiệu sớm: velocity tăng đều 3 quý
liền trong khi cycle time và lead time không đổi — nghĩa là chỉ có nhãn đổi, dòng chảy không đổi.

**Standup thành báo cáo cho lead.** Cơ chế: standup được thiết kế để thành viên đồng bộ với *nhau* và
tự phát hiện chỗ chồng chéo hoặc chỗ có thể giúp. Khi hướng phát ngôn chuyển thành mỗi người nói với
lead, thông tin phải đi qua một điểm trung tâm, và điểm đó trở thành nghẽn: lead phải nhớ hết và phân
phối lại. Đồng thời, vì lead là người đánh giá, nội dung phát ngôn bị chọn lọc theo hướng an toàn.
Dấu hiệu sớm: đếm hướng phát ngôn trong một standup — nếu dưới 30% lượt là nói với đồng đội, hoặc nếu
lead vắng thì standup bị huỷ, bạn đang có báo cáo chứ không có đồng bộ.

**Thêm nghi lễ để chữa một vấn đề văn hoá.** Ví dụ: có người không nói blocker sớm → thêm một cuộc họp
"blocker review" giữa tuần. Cơ chế: vấn đề gốc là chi phí uy tín của việc nói ra tin xấu; thêm một cái
họp làm chi phí đó *tăng* (giờ phải nói trước nhiều người hơn, có biên bản). Dấu hiệu sớm: mọi giải
pháp được đề xuất trong retro đều có dạng "thêm một buổi họp" hoặc "thêm một trường trong Jira".

**Đổi process khi đang chạy nước rút.** Cơ chế: thay đổi cơ chế phối hợp có chi phí học, và chi phí đó
luôn rơi vào lúc bạn ít dư địa nhất. Thời điểm đúng để đổi process là ngay sau khi kết thúc một chu kỳ
giao hàng lớn. Dấu hiệu sớm: câu "sau khi release xong chúng ta sẽ làm cho tử tế" xuất hiện lần thứ ba.

### Khi nào KHÔNG nên áp dụng

**Khi yêu cầu thực sự cố định bởi bên ngoài và chi phí sai gần bằng không.** Ví dụ cụ thể: implement
một chuẩn báo cáo do Ngân hàng Nhà nước quy định, có tài liệu đặc tả, có bộ test đối chiếu. Không có
gì để "inspect and adapt" về yêu cầu. Ở đây, vòng lặp ngắn cho *yêu cầu* là chi phí thuần. Vẫn giữ
vòng lặp ngắn cho *kỹ thuật* (integrate liên tục, test sớm), nhưng bỏ demo định kỳ cho stakeholder và
bỏ việc mở lại scope mỗi sprint.

**Khi chi phí của một biên chu kỳ cao hơn giá trị thông tin thu được.** Ví dụ: hệ thống on-premise ở
ngân hàng, mỗi lần triển khai cần cửa sổ đêm cuối tuần, cần phê duyệt change advisory board 2 tuần
trước, cần đội vận hành của khách. Sprint 1 tuần ở đây là vô nghĩa vì bạn không release được mỗi tuần.
Cơ chế đúng: chu kỳ nội bộ ngắn để tích hợp và kiểm thử, chu kỳ release dài theo ràng buộc thật, và
đừng gọi chu kỳ nội bộ là sprint để tránh tạo kỳ vọng sai với stakeholder.

**Khi team đang trong khủng hoảng vận hành.** Một team đang có 4 incident P1 mỗi tuần không có capacity
nhận thức để chạy một vòng lặp cải tiến process. Thứ tự đúng: ổn định vận hành trước (chương 06), rồi
mới nói về hình dạng vòng lặp. Áp Scrum lên một team đang chữa cháy tạo ra hiện tượng đặc trưng: mỗi
sprint đều thất bại, và team học được rằng cam kết là vô nghĩa — làm hỏng cơ chế cam kết về sau.

**Khi tổ chức chưa có quyền thay đổi ở tầng team.** Nếu PO không có quyền bỏ story, nếu team không có
quyền đổi cách làm việc, nếu mọi quyết định scope nằm ở một giám đốc gặp được mỗi tháng một lần, thì
cài Scrum vào chỉ tạo ra một vòng lặp không có khâu adapt. Việc phải làm trước là đàm phán quyền —
hoặc chấp nhận rằng bạn đang chạy một hệ thống tuần tự và tối ưu nó cho tuần tự (giảm WIP, giảm thời
gian chờ phê duyệt), thay vì giả vờ nó là Agile. Cách tối ưu hệ thống tuần tự ở chủ đề 3.

**Khi bất định cao nhưng tổ chức không chịu được việc bỏ công đã làm.** Vòng lặp ngắn chỉ có giá trị
nếu kết luận "cái này sai, dừng lại" là kết luận được phép. Ở tổ chức mà mỗi tính năng bị bỏ là một
lần mất mặt của người đã đề xuất, vòng lặp ngắn biến thành vòng lặp bổ sung: mỗi sprint thêm việc,
không bao giờ bỏ việc. Kết quả tệ hơn waterfall, vì scope tăng đơn điệu mà không có ai gác cổng. Điều
kiện tiên quyết phải xử lý trước là ở chương 00 và 03: ai được phép nói "sai rồi, dừng" mà không mất
gì.

---

## 2. Scrum trong thực tế

### Problem Statement

Một startup Việt, 22 engineer, ba team. Team Payment chạy sprint 2 tuần. Lấy dữ liệu 6 sprint gần nhất
(số minh hoạ, dạng dữ liệu bạn tự lấy được từ Jira trong 20 phút):

| Sprint | Story cam kết đầu sprint | Story thêm vào giữa sprint | Story hoàn thành | Story carry-over |
|---|---|---|---|---|
| S1 | 12 | 4 | 9 | 7 |
| S2 | 11 | 6 | 8 | 9 |
| S3 | 14 | 3 | 10 | 7 |
| S4 | 10 | 7 | 7 | 10 |
| S5 | 13 | 5 | 9 | 9 |
| S6 | 12 | 8 | 8 | 12 |

Ba hiện tượng đọc ra từ bảng này. Một: trung bình 5,5 story được nhét vào mỗi sprint sau khi sprint đã
bắt đầu — bằng 45% lượng cam kết ban đầu. Hai: carry-over tăng đơn điệu, từ 7 lên 12. Ba: sprint goal
tồn tại trên giấy nhưng không có sprint nào đạt, và sau S3 thì team ngừng viết sprint goal vì "viết
cũng không đạt".

Team này có tất cả nghi lễ của Scrum và không có bất kỳ lợi ích nào của Scrum. Họ trả chi phí lập kế
hoạch hai tuần một lần cho một kế hoạch bị vô hiệu hoá trong 48 giờ. Hiện tượng phái sinh, đắt hơn:
sau 6 sprint, khi Minh (Tech Lead) nói "sprint này chúng ta cam kết xong luồng refund", không ai trong
team, kể cả Minh, tin điều đó. Cơ chế cam kết đã bị phá — và cam kết là thứ đắt nhất, chậm nhất để xây
lại trong một tổ chức.

Hậu quả nếu thiếu năng lực này, đếm được: dự báo cho stakeholder có sai số vượt 50% và không cải thiện
theo thời gian (vì không có chu kỳ nào hoàn tất để học); WIP tăng dần khiến cycle time tăng dần (chủ đề
3); và tỉ lệ nghỉ việc của engineer trong các team này cao hơn, với lý do phỏng vấn thôi việc lặp lại:
"làm cả ngày mà không thấy xong việc gì".

### First Principles

**Scrum là một hệ thống pull có giới hạn WIP theo lô thời gian, cộng với một cơ chế bảo vệ.**

Bỏ hết từ vựng đi, Scrum chỉ có ba cơ chế cơ học:

1. **Giới hạn WIP theo lô.** Sprint backlog là một hàng đợi có kích thước cố định. Team không nhận thêm
   việc cho đến khi chu kỳ kết thúc. Đây là WIP limit — cùng bản chất với WIP limit của Kanban, chỉ
   khác cách đặt (theo lô thời gian thay vì theo cột).
2. **Cơ chế bảo vệ khỏi thay đổi giữa chu kỳ.** Trong sprint, chỉ team được thay đổi sprint backlog.
   Người ngoài không nhét việc vào được. Đây là *toàn bộ* giá trị kinh tế của Scrum so với một hàng đợi
   thông thường.
3. **Điểm đồng bộ định kỳ.** Cuối chu kỳ, mọi bên gặp nhau, xem kết quả thật, và điều chỉnh.

Vì sao cơ chế bảo vệ là phần cốt lõi, không phải phần phụ? Vì chi phí của việc gián đoạn không tuyến
tính. Khi một việc mới được nhét vào giữa sprint, chi phí không phải là "thêm 3 ngày công". Chi phí là:

- **Chi phí chuyển ngữ cảnh:** một engineer đang giữ một mô hình tinh thần phức tạp về module đang làm.
  Chuyển sang việc khác rồi quay lại tốn thời gian nạp lại — trong tài liệu công khai về công việc tri
  thức, con số thường được nhắc là 15–30 phút cho mỗi lần chuyển sâu, và với công việc kiến trúc phức
  tạp thì cao hơn.
- **Chi phí WIP tăng:** việc cũ không bị xoá, nó chỉ chuyển sang trạng thái "đang làm nhưng không ai
  làm". Theo Little's Law (chủ đề 3), WIP tăng làm cycle time của *mọi* việc trong hệ thống tăng —
  kể cả việc gấp vừa nhét vào.
- **Chi phí phá dự báo:** mọi phát ngôn về tiến độ trước đó trở nên sai, và không ai cập nhật lại, nên
  stakeholder tiếp tục ra quyết định dựa trên thông tin cũ.
- **Chi phí phá cam kết:** đây là chi phí lớn nhất và không xuất hiện trong bất kỳ báo cáo nào. Khi
  cam kết bị vô hiệu hoá từ bên ngoài nhiều lần, team học được rằng cam kết là hình thức. Từ đó, mọi
  cơ chế dựa trên cam kết — bao gồm Definition of Done, bao gồm cả việc một engineer nói "tôi sẽ xong
  thứ Năm" — mất giá trị thông tin.

Vì vậy: **nếu bỏ cơ chế bảo vệ, phần còn lại của Scrum không chỉ vô nghĩa mà có hại.** Vô nghĩa vì bạn
mất lợi ích. Có hại vì bạn vẫn trả chi phí — chi phí lập kế hoạch một lô cố định, chi phí họp planning,
chi phí ước lượng — cho một lô sẽ không được giữ nguyên. Một hệ thống chấp nhận việc mới liên tục thì
đúng cơ chế của nó là Kanban với WIP limit và phân loại ưu tiên, không phải Scrum. Chạy Scrum không có
bảo vệ là trường hợp xấu nhất trong cả hai lựa chọn.

**Cơ chế thứ hai: vì sao Scrum cần một vai giữ ưu tiên (PO) tách khỏi vai giữ chất lượng (team).** Đây
là một ứng dụng của Principal–Agent Problem đảo ngược. Nếu cùng một người quyết định *làm gì* và
*đủ tốt là thế nào*, thì dưới áp lực thời gian, mục tiêu ngắn hạn (giao đúng ngày) luôn thắng mục tiêu
dài hạn (giữ khả năng thay đổi hệ thống), vì mục tiêu ngắn hạn có người đòi và có ngày, còn mục tiêu
dài hạn thì không. Việc tách hai vai không phải vì thiếu tin tưởng; nó là một cấu trúc cân bằng cố ý,
tương tự việc tách người đề xuất chi tiêu và người phê duyệt chi tiêu trong tài chính.

**Cơ chế thứ ba: vì sao Definition of Done phải kiểm được, không phải mô tả được.** "Code chạy tốt" là
một mô tả; nó phụ thuộc vào người đọc. "Coverage của module mới ≥ 70% và pipeline xanh" là một tiêu chí
kiểm được; nó không phụ thuộc người đọc. Sự khác biệt quan trọng vì Definition of Done là ranh giới mà
áp lực deadline sẽ đẩy vào. Một ranh giới mô tả được sẽ trôi dần mà không ai phát hiện; một ranh giới
kiểm được thì để vượt qua phải có người *nói ra* rằng ta đang bỏ nó — và việc phải nói ra là toàn bộ
tác dụng.

### Mental Model

**Mô hình 1: Sprint như một hợp đồng hai chiều.** Cách hữu ích nhất để nghĩ về sprint không phải "một
đơn vị thời gian" mà "một hợp đồng có hai điều khoản đối xứng":

```
Team hứa:      trong 2 tuần này, chúng tôi tập trung vào mục tiêu X,
               và cuối chu kỳ có thứ chạy được để bạn xem.
Tổ chức hứa:   trong 2 tuần này, chúng tôi không đổi mục tiêu X,
               và không nhét việc mới vào.
```

Cả hai điều khoản hoặc cùng có hiệu lực, hoặc cùng vô hiệu. Cách dùng mô hình này trong thực tế: khi
stakeholder nhét việc, đừng tranh luận về việc đó gấp hay không gấp. Nói: "được, nhưng đây là một lần
mở lại hợp đồng, nên phần cam kết của team cũng mở lại — story nào ra khỏi sprint?" Câu này chuyển
cuộc trao đổi từ tranh luận về tính cấp thiết (bạn luôn thua, vì mọi thứ đều gấp trong mắt người đề
xuất) sang trao đổi về đánh đổi (bạn ở thế cân bằng, vì người kia phải chọn).

**Mô hình 2: Ba loại thay đổi giữa sprint, ba cách xử lý khác nhau.** Không phải mọi gián đoạn đều
giống nhau, và cách xử lý đồng nhất là sai:

| Loại | Ví dụ | Đặc điểm | Cách xử lý |
|---|---|---|---|
| Sự cố production | Thanh toán lỗi, dữ liệu sai | Không hoãn được, không dự báo được từng cái nhưng dự báo được tổng | Buffer capacity cố định + on-call rotation |
| Việc gấp thật từ kinh doanh | Đối tác đổi API với deadline luật định | Hoãn được vài ngày, không hoãn được vài tuần | Swap có giá: vào thì phải có cái ra |
| Việc "gấp" do thiếu kế hoạch | Founder vừa gặp một khách và có ý mới | Hoãn được đến sprint sau mà không mất gì | Vào backlog, ưu tiên ở planning tiếp |

Đa số tổ chức xử lý cả ba theo cách của loại một (nhận ngay), nên cả ba loại đều đục vào cam kết. Việc
đầu tiên một Tech Lead nên làm là *phân loại được* — vì loại ba thường chiếm phần lớn số lượng.

**Mô hình 3: Theory of Constraints áp lên sprint.** Trong một sprint, luôn có một nghẽn: có thể là
review, có thể là QA, có thể là môi trường staging, có thể là một người duy nhất biết một module. Việc
nạp thêm công việc vào một hệ thống mà nghẽn không đổi thì không tăng đầu ra — nó chỉ tăng hàng đợi
trước nghẽn. Ứng dụng cụ thể ở sprint planning: trước khi hỏi "chúng ta nhận được bao nhiêu story",
hỏi "nghẽn của sprint trước là gì, và sprint này có gì thay đổi ở nghẽn đó". Nếu không có gì thay đổi
ở nghẽn thì đừng nhận nhiều hơn sprint trước, bất kể team cảm thấy thế nào.

### Practical Framework

**Bước 1 — Vẽ rõ ranh giới quyền, thành văn bản, một trang.** Đây là bước hầu như luôn bị bỏ, và là
nguyên nhân của phần lớn xung đột về sau. Bảng dưới là mẫu; điền tên thật vào và dán lên chỗ ai cũng
đọc được:

| Quyết định | Ai quyết | Ai được tham vấn | Ai không quyết |
|---|---|---|---|
| Thứ tự ưu tiên của backlog | PO (Trang) | Tech Lead, EM, sales | Không ai override trực tiếp ngoài quy trình escalation |
| Nội dung sprint backlog (nhận bao nhiêu, món nào) | Team | PO | PO không được ép số lượng |
| Đổi scope giữa sprint | PO đề xuất, team đồng ý swap | Tech Lead | Không ai khác được thêm việc trực tiếp cho engineer |
| Cách làm về mặt kỹ thuật | Team, Tech Lead có tiếng quyết khi bất đồng | Architect nếu có | PO không quyết |
| Definition of Done | Team + Tech Lead, EM phê duyệt | PO, QA | PO không được hạ DoD để giữ ngày |
| Tuyên bố một story là Done | Người review theo DoD | — | Không phải PO, không phải người viết code |
| Hoãn hoặc bỏ một tính năng | PO, có thông báo cho stakeholder | Team | — |
| Kéo dài sprint | Không ai (sprint không được kéo dài) | — | — |

Hai dòng quan trọng nhất: "đổi scope giữa sprint" và "Definition of Done". Nếu PO một mình quyết được
cả hai, bạn không có cấu trúc cân bằng, và chất lượng bên trong sẽ bị xói mòn có hệ thống (chủ đề 8).

**Bước 2 — Sprint planning thực dụng, 90 phút, ba phần.** Phần lớn buổi planning 3 giờ tốn thời gian ở
việc ước lượng lại những thứ đã bàn. Cấu trúc dùng được:

- *Phần A (15 phút) — Nhìn lại thực tế:* sprint trước hoàn thành bao nhiêu, carry-over là gì và vì sao,
  nghẽn ở đâu. Không phân tích lỗi cá nhân. Output: một con số capacity thực tế cho sprint này (đã trừ
  nghỉ phép, on-call, buffer).
- *Phần B (20 phút) — Chốt sprint goal trước, chốt story sau:* PO trình bày một mục tiêu, team hỏi cho
  rõ. Đây là thứ tự quan trọng: nếu chọn story trước rồi viết goal sau, goal sẽ là một câu tóm tắt danh
  sách story (vô dụng). Nếu chọn goal trước, story được chọn để phục vụ goal, và khi phải cắt thì biết
  cắt cái nào.
- *Phần C (45 phút) — Chọn story và làm rõ đủ để bắt đầu:* chỉ chia nhỏ đến mức đủ để người bắt đầu
  ngày mai không bị chặn. Không thiết kế chi tiết cho story sẽ làm ở ngày thứ 8.
- *Đóng (10 phút):* đọc lại sprint goal, xác nhận capacity còn lại cho buffer, xác nhận ai làm gì trong
  hai ngày đầu.

Tiêu chí biết là xong: mỗi người trong team trả lời được "mục tiêu sprint này là gì" bằng một câu, và
"nếu tôi xong việc của tôi sớm thì tôi lấy gì tiếp".

**Bước 3 — Template sprint goal.** Sprint goal xấu: "Hoàn thành 12 story trong sprint backlog". Sprint
goal tốt phải nêu được *ai được gì* và *kiểm được bằng gì*:

```
SPRINT GOAL — Sprint 24 (04/08 → 15/08)

Mục tiêu (một câu, hướng người dùng hoặc hướng kinh doanh):
  Người dùng đã có ví có thể hoàn tất một giao dịch trả góp 3 kỳ với
  đối tác A, từ chọn kỳ hạn đến nhận xác nhận, trên môi trường staging.

Vì sao mục tiêu này, sprint này:
  Điều kiện tiên quyết để mở UAT với đối tác A ngày 20/08 — ngày này
  do đối tác chốt, không dịch được.

Kiểm bằng gì (demo được, không cần giải thích):
  - Chạy end-to-end trên staging với 1 tài khoản test: thành công.
  - Kịch bản đối tác từ chối: hiển thị đúng thông báo, không tạo giao dịch treo.
  - Đối soát: 1 giao dịch trả góp xuất hiện đúng trong báo cáo ngày.

Ngoài phạm vi sprint này (nói rõ để không ai kỳ vọng):
  - Trả góp 6 và 12 kỳ.
  - Đối tác B.
  - Màn hình quản trị cho CS.

Nếu phải cắt để giữ mục tiêu, cắt theo thứ tự:
  1. Màn hình lịch sử trả góp (chấp nhận dùng bảng tạm)
  2. Email thông báo (chấp nhận chỉ có push)
  KHÔNG cắt: đối soát, xử lý lỗi từ đối tác.

Điều kiện coi là thất bại (nói trước, để không tranh luận sau):
  Không chạy được end-to-end trên staging vào hết ngày 14/08.
```

Giá trị của mục "cắt theo thứ tự" và "điều kiện thất bại": chúng được viết lúc bình tĩnh, và dùng lúc
căng thẳng. Ngày 13/08, khi phải quyết định nhanh, bạn không đàm phán lại từ đầu.

**Bước 4 — Template Definition of Done.** Nguyên tắc: mỗi dòng phải kiểm được bằng một lệnh, một cái
nhìn vào dashboard, hoặc một tên người xác nhận. Nếu một dòng cần tranh luận để biết đã đạt chưa, viết
lại dòng đó.

```
DEFINITION OF DONE — Team Payment, phiên bản 2026-07, review lại mỗi quý

Áp dụng cho: mọi story đưa vào sprint backlog.
Không áp dụng cho: spike (có tiêu chí riêng), hotfix P1 (có luồng riêng, xem 06).

A. CODE
[ ] Merge vào main, không còn branch treo.
[ ] Ít nhất 1 approve từ người không viết code chính.
[ ] Không có TODO mới không kèm số ticket.
[ ] Không thêm dependency mới chưa qua checklist license/bảo mật.

B. TEST  (đây là mục hay bị cắt ngầm — nên phải kiểm được)
[ ] Unit test cho logic nghiệp vụ mới: coverage phần thay đổi >= 70%
    (đo bằng báo cáo diff-coverage của pipeline, không đo bằng cảm nhận).
[ ] Ít nhất 1 test cho luồng lỗi, không chỉ luồng thành công.
[ ] Với thay đổi liên quan tiền: 1 test đối soát số dư trước/sau.
[ ] Pipeline xanh trên main, không có test bị skip mới.

C. QUAN SÁT ĐƯỢC  (liên kết chương 06)
[ ] Log có correlation id ở các điểm vào/ra.
[ ] Có metric hoặc alert cho luồng mới, hoặc ghi rõ lý do không cần.
[ ] Dashboard hiện có không bị vỡ.

D. VẬN HÀNH
[ ] Có đường rollback: feature flag, hoặc migration đảo được, hoặc ghi rõ cách hoàn tác.
[ ] Migration dữ liệu đã chạy thử trên bản sao dữ liệu production.
[ ] Nếu đổi API công khai: version cũ còn hoạt động, hoặc consumer đã được thông báo.

E. TÀI LIỆU VÀ BÀN GIAO
[ ] README/runbook cập nhật nếu đổi cách chạy hoặc cách xử lý sự cố.
[ ] Với thay đổi kiến trúc: có ADR (xem 05).
[ ] CS/Ops biết cách xử lý nếu người dùng phàn nàn về luồng này.

F. XÁC NHẬN
[ ] Đã deploy lên staging và PO xem được.
[ ] Người xác nhận Done: ______ (không phải người viết code)

--- QUY TẮC VỀ NGOẠI LỆ ---
Bỏ bất kỳ dòng nào ở mục B hoặc D phải:
  1. Được Tech Lead đồng ý bằng comment trên ticket, nêu lý do.
  2. Tạo ticket nợ có nhãn `dod-exception`, có người sở hữu và sprint dự kiến trả.
  3. Xuất hiện trong báo cáo cuối sprint.
Không có ngoại lệ ngầm. Nếu tháng nào cũng có > 3 ngoại lệ,
DoD của bạn sai với thực tế — sửa DoD, đừng tiếp tục vi phạm nó.
```

Dòng cuối quan trọng: một DoD bị vi phạm thường xuyên thì tệ hơn một DoD thấp hơn nhưng được tôn trọng,
vì nó dạy team rằng quy tắc là thứ có thể lờ đi.

**Bước 5 — Retro có đầu ra thay đổi cơ chế.** Retro thất bại có một hình dạng nhận biết được: đầu ra là
những câu dạng "cần giao tiếp tốt hơn", "mọi người nên chủ động hơn". Đó là mong muốn, không phải thay
đổi cơ chế. Quy tắc: mỗi retro ra tối đa **hai** action, mỗi action phải trả lời được ba câu:

- Nó đổi *cái gì* trong cơ chế? (một quy tắc, một bước trong pipeline, một trường trong template, một
  người có quyền mới, một cái họp bị bỏ)
- Ai sở hữu, xong khi nào? (tên, ngày)
- Làm sao biết nó có tác dụng? (một quan sát được ở retro sau)

Ví dụ chuyển hoá: "cần review code nhanh hơn" → "thêm quy tắc: PR mở quá 24 giờ chưa có review thì bot
ping vào channel team; Khoa cài, xong 08/08; kiểm ở retro sau bằng thời gian chờ review trung vị, hiện
là 19 giờ (số minh hoạ)". Giới hạn hai action là cố ý: một team hoàn thành 2/2 action ba sprint liền
sẽ tin vào retro; một team có 8 action và làm được 2 sẽ học rằng retro không dẫn tới gì.

**Bước 6 — Cơ chế xử lý việc gấp giữa sprint.** Ba tầng, dùng theo thứ tự:

- *Tầng 1 — Buffer định trước.* Dành một tỉ lệ capacity cố định cho việc không dự báo được. Cách đặt:
  lấy trung bình lượng việc gấp thực tế 6 sprint qua. Nếu 6 sprint qua trung bình 20% capacity đi vào
  việc gấp (số minh hoạ), thì cam kết ở mức 80% — không phải để "an toàn" mà vì 100% là con số sai.
  Điều then chốt: buffer không dùng hết thì *không* nhét thêm story vào; dùng để trả nợ kỹ thuật hoặc
  kết thúc sớm. Nếu buffer luôn bị nhét đầy, nó không còn là buffer.
- *Tầng 2 — Swap có giá.* Việc mới vào thì việc cũ ra, với kích thước tương đương, và PO là người chọn
  cái ra. Ghi lại trên ticket. Sau vài lần, log swap trở thành bằng chứng định lượng khi bạn cần nói
  với ban lãnh đạo rằng "sprint bị mở lại 5,5 lần mỗi chu kỳ".
- *Tầng 3 — Escalation khi không swap được.* Khi việc mới không thể hoãn và không có gì ra được, sprint
  goal bị đặt vào rủi ro. Đây là lúc dừng và đưa quyết định lên người có accountability — dùng template
  ở chủ đề 7. Không tự gánh, vì tự gánh nghĩa là bạn hấp thụ rủi ro của tổ chức bằng chất lượng của
  team.

### Trade-off

**Giữ sprint bất biến vs cho phép thay đổi giữa sprint.** Đây là quyết định trung tâm của chủ đề này,
nên phân tích đầy đủ:

| | Giữ bất biến | Cho phép đổi |
|---|---|---|
| Throughput | Cao hơn: ít chuyển ngữ cảnh, WIP ổn định | Thấp hơn 15–30% ở mức gián đoạn cao (số minh hoạ) |
| Thời gian phản ứng với việc gấp | Chậm: tối đa một chu kỳ | Nhanh: trong ngày |
| Khả năng dự báo | Có: sau 4–6 sprint có dữ liệu ổn định để dự báo | Gần như không: mỗi chu kỳ có đầu vào khác nhau |
| WIP | Ổn định | Tăng dần, vì việc cũ ít bị xoá khi việc mới vào |
| Sức khoẻ cơ chế cam kết | Được củng cố mỗi chu kỳ | Bị xói mòn mỗi lần mở lại |
| Chi phí rơi vào ai | Stakeholder (phải chờ) | Team (chất lượng, cycle time) và bản thân việc gấp (vào hàng đợi dài hơn) |

Điều kiện nghiêng về **giữ bất biến**: tỉ lệ việc thực sự không hoãn được dưới 15% capacity; có người
đủ quyền để nói "không" với stakeholder (thường là EM hoặc chính PO có uy tín); tổ chức cần dự báo cho
quyết định bên ngoài (hợp đồng, marketing, tuyển dụng).

Điều kiện nghiêng về **cho phép đổi**: bản chất công việc là phản ứng (team vận hành, team support,
team hạ tầng nhận yêu cầu từ team khác); tỉ lệ việc không hoãn được vượt 30–40% capacity — ở mức này
Scrum không còn phù hợp và bạn nên chuyển sang Kanban với lớp ưu tiên rõ ràng, chứ không phải "Scrum
linh hoạt"; hoặc công ty đang trong giai đoạn tìm product-market fit mà một tuần chậm có thể là mất
một khách hàng đầu tiên.

Ranh giới quan trọng: **không có phương án giữa hai cái này**. "Sprint nhưng linh hoạt" là tên gọi khác
của "không có cơ chế nào". Nếu bạn ở giữa, hãy chọn Kanban có WIP limit — nó trung thực với thực tế và
vẫn cho bạn một đòn bẩy (chủ đề 3).

**Sprint 1 tuần vs 2 tuần vs 3 tuần.** Sprint 1 tuần: overhead cao (khoảng 15% nếu planning + review +
retro tốn 6 giờ), nhưng phát hiện lệch nhanh, và đặc biệt phù hợp team mới hoặc team đang mất khả năng
dự báo — vì bốn chu kỳ trong một tháng cho bốn lần học. Sprint 3 tuần: overhead thấp, nhưng đủ dài để
"còn thời gian mà" tồn tại đến giữa chu kỳ, và đủ dài để một sai lệch tuần đầu không bị phát hiện. Với
đa số team Việt trong bối cảnh có gián đoạn trung bình, 2 tuần là điểm cân bằng thực tế; chuyển sang 1
tuần trong 4–6 chu kỳ là một can thiệp tốt khi team đang liên tục không đạt cam kết.

**Định lượng Definition of Done chặt vs nhẹ.** DoD chặt làm cycle time từng story dài hơn và làm số
story hoàn thành mỗi sprint ít hơn — điều này gây khó chịu ngay và có thể bị đọc là "team chậm". DoD
nhẹ làm số story tăng và nợ tích lại, biểu hiện muộn hơn 2–3 quý dưới dạng cycle time tăng và số
incident tăng. Nghiêng về chặt khi: hệ thống xử lý tiền hoặc dữ liệu không hoàn tác được; team dự kiến
duy trì hệ thống này trên 2 năm; đã có tiền lệ nợ kỹ thuật gây incident. Nghiêng về nhẹ khi: đang
validate ý tưởng và xác suất bỏ hẳn code này trong 8 tuần trên 50% — nhưng khi đó phải nói rõ đây là
prototype và có ngày xử lý (chủ đề 8).

### Real-world Scenarios

**Tình huống A — Sprint 2 tuần nhưng nhận việc mới mỗi ngày (team Payment ở Problem Statement).**

Minh, Tech Lead, có dữ liệu 6 sprint. Điều Minh làm sai lần đầu: mang bảng vào retro và nói "chúng ta
bị nhét việc quá nhiều, cần dừng việc này". Kết quả: Trang (PO) phòng vệ vì nghe thành lời buộc tội,
và founder — người nhét phần lớn việc — không có mặt trong retro. Không có gì thay đổi.

Điều Minh làm lần hai, đúng hơn về mặt cơ chế:

1. **Định lượng bằng tiền và bằng thời gian chờ, không bằng cảm xúc.** Minh tính: 5,5 story nhét thêm
   mỗi sprint, và cycle time trung vị của một story tăng từ 4,5 ngày (S1) lên 9 ngày (S6) — số minh
   hoạ. Rồi lấy một việc gấp mà chính founder quan tâm và chỉ ra nó mất 11 ngày từ lúc yêu cầu đến lúc
   lên production, trong đó chỉ 2 ngày là làm việc thật. Đây là lập luận mạnh: *việc gián đoạn đang làm
   chậm chính những việc gấp*.
2. **Không xin bỏ gián đoạn. Đề nghị một cơ chế có giá.** Minh đề xuất: 20% capacity là buffer công
   khai cho việc gấp; ngoài 20% đó, mỗi việc vào phải có một việc ra do Trang chọn; và mỗi cuối sprint
   có một dòng báo cáo "sprint này bị mở lại N lần, N story bị đẩy lùi".
3. **Đưa quyết định lên đúng tầng.** Minh và Hà (EM) trình bày với founder, framing: "chúng tôi không
   đề nghị nói không với anh. Chúng tôi đề nghị mỗi lần anh thêm việc, anh thấy ngay cái gì bị đẩy ra."
4. **Đo lại sau 4 sprint.** Số liệu minh hoạ sau 4 sprint: story nhét thêm giảm từ 5,5 xuống 1,8/sprint
   — không phải vì ai bị cấm, mà vì khi phải chọn cái ra, phần lớn "việc gấp" hoá ra hoãn được. Cycle
   time trung vị về 5 ngày. Sprint goal đạt 3/4 sprint.

Cơ chế hoạt động ở đây không phải kỷ luật mà là **hiện hoá chi phí**. Việc nhét thêm trước đó miễn phí
với người nhét; sau khi có luật swap, nó có giá, và lượng cầu giảm theo giá — hoàn toàn dự đoán được.

**Tình huống B — PO đồng thời là người ép deadline.**

Bối cảnh: startup 30 người. Trang là PO của team, đồng thời là người báo cáo tiến độ cho founder và
được đánh giá theo số tính năng ra mắt mỗi quý. Trong sprint review, Trang nói: "Story này về cơ bản
xong rồi, cứ tính là Done để báo cáo, tuần sau bọn em hoàn thiện test."

Vấn đề cấu trúc, không phải vấn đề con người: Trang đang giữ cả hai vai — người đòi (đại diện nhu cầu
kinh doanh) và người xác nhận đủ tốt. Trong cấu trúc đó, mọi trường hợp mơ hồ sẽ nghiêng về phía giao
hàng, một cách có hệ thống, dù Trang thiện chí. Người chịu hậu quả là team, ở một quý sau.

Cách Vy (Tech Lead) xử lý:

- *Ngay lúc đó, không tranh luận về story này:* "Theo DoD của mình thì story này chưa Done vì thiếu
  test luồng lỗi. Em ghi là 'chưa Done, còn 1,5 ngày'. Nếu cần báo cáo cho anh founder thì em đề xuất
  báo là 'đang ở staging, dự kiến production thứ Tư' — như vậy đúng và cũng nghe được."
- *Sau đó, sửa cấu trúc:* đưa vào bảng ranh giới quyền một dòng: người tuyên bố Done là người review
  theo DoD, không phải PO. Có Hà (EM) phê duyệt bảng này để nó không phải là ý kiến của Vy chống Trang.
- *Xử lý gốc rễ ở tầng trên:* Hà nói với founder về việc PO đang bị đo bằng số tính năng, và đề nghị
  thêm một chỉ số đối trọng vào đánh giá của PO — ví dụ số defect phát hiện sau release hoặc số ngoại
  lệ DoD. Không có bước này thì Trang vẫn chịu áp lực cũ và sẽ tìm đường khác.

Lưu ý bối cảnh Việt Nam: nếu Trang lớn tuổi hơn hoặc thâm niên hơn Vy, cuộc trao đổi trực tiếp trước
cả phòng có xác suất cao bị đọc thành thách thức chứ thành trao đổi kỹ thuật. Kỹ thuật giảm rủi ro:
neo vào văn bản chung ("theo DoD của mình") thay vì vào phán đoán cá nhân ("anh nghĩ chưa xong"), và
xử lý phần cấu trúc riêng, không trước phòng. Chi tiết cách trao đổi bất đồng ở chương 02.

### Best Practices

**Viết ra ranh giới quyền trước khi có xung đột, không phải trong lúc xung đột.** Lý do: trong xung
đột, mọi đề xuất quy tắc đều bị đọc là chiếm lợi thế cho một phía. Cùng một quy tắc, viết lúc bình
thường thì trung tính, viết lúc căng thẳng thì là vũ khí.

**Chốt sprint goal trước khi chọn story.** Lý do: goal viết sau danh sách chỉ là tóm tắt danh sách, nên
không dùng được để cắt. Goal viết trước tạo tiêu chí chọn và tiêu chí cắt — giá trị của nó xuất hiện ở
ngày thứ 8, không phải ngày thứ nhất.

**Mỗi dòng của Definition of Done phải kiểm được bằng lệnh, dashboard, hoặc tên người.** Lý do: đã phân
tích ở First Principles — ranh giới mô tả được sẽ trôi im lặng; ranh giới kiểm được buộc phải có người
nói ra khi vượt qua.

**Cam kết ở mức capacity thực tế đã trừ buffer, và công khai con số buffer.** Lý do: cam kết 100% là
cam kết chắc chắn thất bại, và thất bại có hệ thống làm cam kết mất giá trị. Công khai buffer để nó
không bị hiểu là team giữ chỗ cho mình.

**Giới hạn hai action mỗi retro, có tên và ngày, kiểm ở retro sau.** Lý do: retro là bộ phận "adapt"
của vòng lặp; nó chỉ tồn tại được nếu team có bằng chứng rằng nó dẫn tới thay đổi. Hai action hoàn
thành có giá trị hơn tám action bị bỏ.

**Ghi log mọi lần swap và báo cáo tổng mỗi sprint.** Lý do: nó chuyển một hiện tượng vô hình (sprint bị
mở lại) thành một con số mà stakeholder phải nhìn. Không có con số này, cuộc trao đổi về gián đoạn mãi
là "team cảm thấy bị nhét việc" đối lại "tôi thấy có mấy việc thôi".

**Khi tỉ lệ gián đoạn vượt 30–40%, đổi cơ chế thay vì cố kỷ luật hơn.** Lý do: ở mức đó, chi phí duy
trì hình thức sprint vượt lợi ích, và việc liên tục thất bại cam kết gây hại nhiều hơn việc thừa nhận
rằng bản chất công việc là dòng chảy.

### Anti-patterns

**Sprint 2 tuần nhưng nhận việc mới mỗi ngày.** Cơ chế phá hệ thống: bạn trả toàn bộ chi phí của lô cố
định (planning, ước lượng, đóng gói) mà không nhận lợi ích của lô cố định (tập trung, dự báo). Đồng
thời cam kết thất bại đều đặn dạy team rằng cam kết là hình thức, làm hỏng công cụ này cho mọi mục
đích về sau. Dấu hiệu sớm: carry-over tăng ba sprint liền; số story thêm giữa sprint vượt 25% cam kết;
team ngừng viết sprint goal.

**Retro không có action, hoặc có action không ai sở hữu.** Cơ chế: retro thành nơi xả cảm xúc. Xả cảm
xúc mà không có thay đổi cơ chế tạo ra learned helplessness — team học được rằng việc nêu vấn đề không
dẫn tới gì, và dừng nêu. Điều này đắt hơn việc không có retro, vì bạn mất luôn kênh thông tin sớm. Dấu
hiệu sớm: cùng một vấn đề xuất hiện ở ba retro liên tiếp; action có dạng "cần cải thiện X" không có
tên và ngày; tỉ lệ hoàn thành action dưới 40%.

**PO đồng thời là người ép deadline và là người xác nhận Done.** Cơ chế: đã phân tích. Bổ sung dấu hiệu
sớm: PO là người đóng ticket trên Jira; trong sprint review có câu "cứ tính là xong"; DoD có ngoại lệ
mà không có ticket nợ tương ứng.

**Sprint planning biến thành phiên phân task cho từng người.** Cơ chế: khi mỗi story được gán tên ngay
ở planning, sprint backlog trở thành 7 danh sách cá nhân song song thay vì một hàng đợi kéo. Hệ quả:
WIP bằng số người (mỗi người làm việc của mình từ ngày đầu), không ai giúp ai, và nếu một người bị
chặn thì việc của người đó đứng đến cuối sprint. Dấu hiệu sớm: ngày thứ 2 của sprint, số story ở trạng
thái In Progress bằng số thành viên.

**Kéo dài sprint để "kịp xong".** Cơ chế: sprint là đơn vị đo. Kéo dài đơn vị đo để kết quả vừa với nó
làm mất khả năng đo, và làm mất cả điểm đồng bộ với bên ngoài. Sau hai lần kéo dài, không ai còn biết
capacity thật của team là bao nhiêu. Dấu hiệu sớm: câu "sprint này mình để đến thứ Ba tuần sau cho
tròn". Cách đúng: sprint kết thúc đúng ngày, việc chưa xong quay về backlog và được ưu tiên lại.

**Dùng story point làm đơn vị báo cáo ra ngoài team.** Cơ chế: điểm là đơn vị nội bộ, tương đối, không
so sánh được liên team. Khi nó đi ra ngoài, nó bị so sánh, và bị tối ưu. Xem thêm chủ đề 4. Dấu hiệu
sớm: có slide so sánh velocity giữa các team.

### Khi nào KHÔNG nên áp dụng

**Khi bản chất công việc của team là phản ứng.** Team support, team vận hành, team hạ tầng nhận yêu cầu
từ 10 team khác — với những team này, hơn một nửa công việc đến mà không thể dự báo từng cái. Áp Scrum
lên đây tạo ra thất bại cam kết có hệ thống. Cơ chế đúng: Kanban với lớp dịch vụ (class of service) và
WIP limit, cộng với một phần capacity dành cho công việc chủ động có kế hoạch. Chi tiết ở chủ đề 3.

**Khi tổ chức không cho team quyền bảo vệ sprint.** Nếu founder hoặc giám đốc có thể và sẽ nhét việc
trực tiếp cho engineer, và không có ai đủ quyền để đặt luật swap, thì cơ chế bảo vệ không tồn tại. Chạy
Scrum ở đây là tự tạo bằng chứng thất bại đều đặn cho chính team mình. Việc phải làm trước là đàm phán
quyền (dùng lập luận định lượng ở tình huống A). Nếu đàm phán thất bại, chọn Kanban — nó ít nói dối về
thực tế hơn.

**Khi team dưới 3 người hoặc mọi người làm việc độc lập hoàn toàn.** Với 2 engineer làm hai việc không
liên quan, gần như toàn bộ cơ chế phối hợp của Scrum là overhead. Giữ lại một thứ: một cuộc gặp định kỳ
để xem lại ưu tiên và một cơ chế nói ra blocker. Bỏ phần còn lại.

**Khi đang trong giai đoạn discovery thuần.** Nếu công việc trong 6 tuần tới là "tìm hiểu xem có nên
làm cái này không", thì cam kết một lô đầu ra là sai loại cam kết. Cơ chế đúng: timebox có ngân sách và
tiêu chí dừng, đánh giá theo câu hỏi đã trả lời được, không theo story đã hoàn thành. Trộn discovery
vào sprint chung với delivery làm cả hai loại công việc đều bị đo sai — discovery bị coi là chậm, và
delivery bị nhiễu.

**Khi chu kỳ release bị ràng buộc bên ngoài dài hơn sprint nhiều lần.** Đã nêu ở chủ đề 1: nếu release
phải qua CAB của khách và cửa sổ triển khai mỗi tháng, thì điểm đồng bộ thật là tháng. Vẫn có thể giữ
chu kỳ nội bộ hai tuần để tích hợp và kiểm thử, nhưng đừng gọi nó là sprint có demo cho stakeholder,
và đừng đo thành công bằng "đã lên production" khi việc lên production không nằm trong tay team.

---

## 3. Kanban và quản lý dòng chảy

### Problem Statement

Một e-commerce Việt, team Checkout, 6 engineer. Board Jira có 5 cột: Todo, In Progress, Code Review,
QA, Done. Chụp board lúc 10 giờ sáng thứ Ba:

```
Todo (23)        In Progress (11)     Code Review (9)    QA (6)    Done (tuần này: 2)
[.....]          [KH-201 Khoa 3d]     [KH-180  6d]       [KH-155 4d]
                 [KH-204 Khoa 1d]     [KH-182  5d]       [KH-161 3d]
                 [KH-198 Quân 8d]     [KH-186  5d]       [KH-163 3d]
                 [KH-205 Quân 2d]     [KH-188  3d]       [KH-170 2d]
                 [KH-191 Nam 12d]     [KH-190  3d]       [KH-172 2d]
                 [KH-206 Nam 1d]      [KH-193  2d]       [KH-174 1d]
                 [KH-199 Vy 6d]       [KH-195  2d]
                 [KH-207 Vy 1d]       [KH-196  1d]
                 [KH-200 Duy 5d]      [KH-197  1d]
                 [KH-208 Duy 1d]
                 [KH-210 Sơn 2d]
```

Sáu người, 26 item đang trong hệ thống, 11 item "In Progress". Mỗi engineer có 2 item đang làm. Mỗi
tuần đóng được 2 item. Founder hỏi: "Cần tuyển thêm mấy người để đi nhanh hơn?"

Câu trả lời đúng là: không cần người nào, và tuyển thêm sẽ làm chậm hơn. Nhưng để nói được câu đó và
được tin, bạn cần số. Bốn hiện tượng đọc ra từ board này, tất cả đều đếm được:

- **Cột Code Review có 9 item, item cũ nhất chờ 6 ngày.** Đây là nghẽn. Toàn bộ công sức thêm vào cột
  In Progress chỉ làm cột này dài hơn.
- **Mỗi người có 2 item In Progress.** Không phải vì ai cũng làm hai việc cùng lúc, mà vì item thứ nhất
  bị chặn (chờ review, chờ trả lời từ PO, chờ môi trường) nên người đó bắt đầu item thứ hai. WIP tăng
  là *hệ quả* của thời gian chờ, không phải nguyên nhân — và cũng làm thời gian chờ tăng thêm. Đây là
  một vòng phản hồi dương.
- **Cột QA có 6 item và không có ai tên trong đó.** QA là một người dùng chung cho 3 team.
- **Cột "Done" không phải Done.** Hỏi ra: item ở Done vẫn chờ release window thứ Năm hàng tuần, và có
  2 item ở Done từ tuần trước chưa lên production.

Hậu quả nếu thiếu năng lực đọc dòng chảy: tổ chức phản ứng với việc chậm bằng đòn bẩy duy nhất mà nó
biết — thêm người và thêm áp lực. Thêm người vào một hệ thống có nghẽn ở review chỉ tạo thêm PR chờ
review. Thêm áp lực làm mọi người bắt đầu nhiều việc hơn, tăng WIP, tăng cycle time. Cả hai đòn bẩy đều
đi sai hướng, và tổ chức không biết vì nó không đo cái cần đo: nó đếm số ticket đóng, không đo thời
gian một việc đi từ đầu đến cuối.

### First Principles

**Little's Law: throughput là hàm của WIP và cycle time.**

```
WIP = Throughput × Cycle Time
```

Hay dạng dùng được hơn cho lead:

```
Cycle Time = WIP / Throughput
```

Đây là một đẳng thức của lý thuyết hàng đợi, đúng cho mọi hệ thống ổn định, không phụ thuộc phân phối.
Nó không phải một heuristic quản trị — nó là một ràng buộc toán học. Áp vào board ở trên:

| Đại lượng | Giá trị (số minh hoạ) | Cách đo trong thực tế |
|---|---|---|
| WIP (item đã bắt đầu, chưa lên production) | 26 | Đếm thẻ ngoài Todo và ngoài Done-thật |
| Throughput | 2 item/tuần | Đếm item lên production mỗi tuần, lấy trung bình 6 tuần |
| Cycle Time trung bình suy ra | 26 / 2 = **13 tuần** | Kiểm chứng: đo thời gian thật từ In Progress đến production |

Con số 13 tuần thường gây sốc, vì cảm nhận của team là "mỗi việc làm mất khoảng 3 ngày". Cả hai đều
đúng: 3 ngày là thời gian *làm*, 13 tuần là thời gian *đi qua hệ thống*. Chênh lệch giữa hai con số là
thời gian chờ, và thời gian chờ không xuất hiện trên bất kỳ báo cáo nào của bất kỳ ai.

Bây giờ là phần có giá trị: đẳng thức này cho hai đòn bẩy, và chỉ một trong hai là miễn phí.

| Muốn giảm Cycle Time từ 13 tuần xuống ~4 tuần | Cách làm | Chi phí |
|---|---|---|
| Tăng Throughput lên 6,5 item/tuần | Tuyển thêm ~3x người, hoặc tăng năng suất 3x | Rất đắt, chậm (3–6 tháng để người mới hiệu quả), và không khả thi nếu nghẽn không nằm ở số người |
| Giảm WIP từ 26 xuống 8 | Ngừng bắt đầu việc mới, hoàn tất việc đang mở | **Miễn phí. Làm được trong một buổi họp.** |

Đây là một trong rất ít đòn bẩy miễn phí trong quản trị delivery. Không cần ngân sách, không cần phê
duyệt, không cần công cụ mới, không cần ai học kỹ năng mới. Chỉ cần một quyết định: ngừng bắt đầu, bắt
đầu hoàn tất.

Điểm tinh tế: giảm WIP không làm throughput giảm tương ứng — thường nó làm throughput *tăng*, vì các
chi phí phái sinh của WIP cao biến mất: ít chuyển ngữ cảnh hơn, ít merge conflict hơn (nhánh sống ngắn
hơn), ít việc phải làm lại hơn (feedback đến sớm hơn khi context còn tươi), và ít thời gian dành cho
việc quản lý danh sách việc đang mở.

**Queueing theory: vì sao utilization gần 100% làm thời gian chờ tăng vọt.**

Với một hệ thống hàng đợi có biến động (mọi hệ thống phát triển software đều có biến động: task không
đều nhau, người nghỉ, incident xảy ra), thời gian chờ tăng theo utilization $\rho$ theo dạng:

```
Thời gian chờ  ∝  ρ / (1 - ρ)
```

Bảng giá trị (số minh hoạ, tính từ công thức trên với hệ số 1):

| Utilization | ρ/(1−ρ) | Đọc thế nào |
|---|---|---|
| 50% | 1,0 | Thời gian chờ bằng thời gian làm |
| 70% | 2,3 | Chờ gấp 2,3 lần thời gian làm |
| 80% | 4,0 | Chờ gấp 4 lần |
| 90% | 9,0 | Chờ gấp 9 lần |
| 95% | 19,0 | Chờ gấp 19 lần |
| 100% | ∞ | Hàng đợi tăng vô hạn |

Đây là lý do cơ học của một hiện tượng mà mọi lead đều đã thấy nhưng ít người giải thích được: một team
được "lấp đầy 100% capacity" đi chậm hơn rất nhiều một team được lấp 80%. Không phải vì team lười ở 20%
còn lại, mà vì ở utilization cao, mọi biến động nhỏ (một người nghỉ ốm, một incident, một PR khó) đều
tạo hàng đợi và hàng đợi không tự tiêu được vì không có dư địa.

Ứng dụng trực tiếp và phản trực giác nhất: **một QA dùng chung cho 3 team, được lấp 95% thời gian, là
một nghẽn tạo thời gian chờ gấp 19 lần thời gian test thật.** Nếu test một story mất 2 giờ, thời gian
từ lúc story vào QA đến lúc ra khỏi QA là khoảng 38 giờ — gần một tuần làm việc. Giải pháp không phải
"QA làm nhanh hơn" mà là giảm utilization của QA (thêm người, hoặc chuyển một phần test sang tự động,
hoặc giảm lượng việc đến).

Cùng cơ chế áp cho: người duy nhất biết một module (bus factor 1 là một nghẽn utilization cao), người
duy nhất được phê duyệt deploy, môi trường staging duy nhất.

**Flow efficiency: đại lượng nói ra sự thật nhanh nhất.**

```
Flow Efficiency = Thời gian làm việc thật / Tổng thời gian trong hệ thống
```

Với team Checkout ở trên: một story mất khoảng 3 ngày làm việc thật, và khoảng 13–15 ngày lịch để đi
qua (số minh hoạ, đo từ Jira). Flow efficiency ≈ 3/14 ≈ 21%. Trong tài liệu công khai về lean/flow, các
tổ chức chưa tối ưu thường ở mức 5–25%, và mức 40% được coi là tốt. Ý nghĩa quản trị của con số này rất
lớn: nếu flow efficiency là 21%, thì làm mọi engineer code nhanh hơn 30% chỉ cải thiện tổng cycle time
khoảng 6%. Toàn bộ dư địa nằm ở 79% thời gian chờ — tức nằm ở process, không nằm ở kỹ năng cá nhân.

Đây cũng là câu trả lời cơ học cho việc vì sao "tuyển thêm senior" thường không cải thiện delivery ở
những tổ chức có flow efficiency thấp.

### Mental Model

**Mô hình 1: Theory of Constraints.** Một hệ thống có duy nhất một nghẽn tại mỗi thời điểm. Throughput
của hệ thống bằng throughput của nghẽn. Mọi cải thiện ở chỗ không phải nghẽn là ảo — nó chỉ làm hàng
đợi trước nghẽn dài hơn. Năm bước của Goldratt, dịch sang ngữ cảnh delivery:

1. **Xác định nghẽn** — cột nào tích thẻ nhiều nhất và có thời gian chờ dài nhất.
2. **Khai thác nghẽn** — đảm bảo nghẽn không bao giờ rảnh vì thiếu đầu vào, và không làm việc lãng phí
   (ví dụ: QA không nên test thứ chưa qua smoke test).
3. **Phụ thuộc mọi thứ khác vào nghẽn** — nhịp nạp việc vào hệ thống theo nhịp nghẽn, không theo nhịp
   người rảnh.
4. **Mở rộng nghẽn** — thêm năng lực (người, tự động hoá, đào tạo thêm người review được).
5. **Quay lại bước 1** — vì nghẽn sẽ dịch sang chỗ khác. Đây là bước hay bị bỏ, dẫn tới việc tối ưu mãi
   một chỗ đã không còn là nghẽn.

**Mô hình 2: Board là một mô hình của hệ thống, và mô hình sai thì quyết định sai.** Một board có 5 cột
đẹp nhưng thực tế có 9 trạng thái là một mô hình sai. Nguyên tắc: **mọi nơi công việc dừng lại phải là
một cột hoặc một hàng đợi hiển thị**. Đặc biệt các cột chờ, thứ hay bị ẩn:

```
Board thường thấy (mô hình sai):
Todo → In Progress → Code Review → QA → Done

Dòng chảy thật (mô hình đúng):
Todo → [chờ làm rõ yêu cầu] → In Progress → [chờ review] → Review
     → [chờ sửa theo review] → [chờ môi trường QA] → QA
     → [chờ PO xác nhận] → [chờ release window] → Production
```

Bốn hàng đợi ẩn trong ví dụ này. Khi chúng bị ẩn, thời gian nằm trong chúng được tính vào "In Progress"
hoặc "QA", và bạn kết luận sai rằng dev chậm hoặc QA chậm.

**Mô hình 3: Đọc hình dạng board như đọc X-quang.** Hình dạng phân bố thẻ trên board là một chẩn đoán:

| Hình dạng board | Chẩn đoán | Can thiệp đầu tiên |
|---|---|---|
| Thẻ dồn ở một cột giữa | Nghẽn ở cột đó | Đặt WIP limit cho cột *trước* nó; tăng năng lực cột đó |
| Thẻ rải đều mọi cột, mỗi người 2–3 thẻ | WIP quá cao toàn hệ thống, ai cũng multitask | Đóng băng việc mới, hoàn tất từ phải sang trái |
| Cột đầu (Todo) rất dài | Không phải vấn đề dòng chảy; vấn đề ưu tiên và gác cổng | Giới hạn backlog, bỏ thẻ cũ hơn 90 ngày |
| Thẻ ở cột cuối tích lại | Nghẽn ở release/deploy | Tăng tần suất release, tự động hoá |
| Cột Review dài, cột QA rỗng | Review là nghẽn, QA đang bỏ trống năng lực | Mở rộng số người review được; ghép review vào cặp |
| Board sạch nhưng cycle time dài | Công việc bị làm ngoài board | Đưa mọi việc lên board, kể cả việc gấp và việc hỗ trợ |
| Nhiều thẻ "Blocked" | Phụ thuộc bên ngoài (xem chủ đề 6) | Đo tổng thời gian blocked, escalate theo dữ liệu |
| Thẻ đi ngược (Review → In Progress) nhiều | Chất lượng đầu vào thấp hoặc DoD không rõ | Sửa Definition of Ready và DoD, không tăng tốc |

### Practical Framework

**Bước 1 — Dựng board phản ánh dòng chảy thật (nửa ngày).** Ngồi cùng team, lấy 5 story đã hoàn thành
gần nhất, và với mỗi story hỏi: "từ lúc bắt đầu đến lúc lên production, nó dừng ở những đâu và bao
lâu?" Ghi mọi điểm dừng, kể cả điểm dừng khó chịu (chờ founder trả lời một câu hỏi, chờ tài khoản test
từ đối tác). Kết quả thường là 8–12 trạng thái, trong khi board hiện tại có 4–5.

Rồi biến mỗi điểm dừng dài thành một cột hoặc một hàng đợi hiển thị. Quy ước hữu ích: phân biệt cột
*đang làm* (có người đang tác động) và cột *đang chờ* (không ai tác động) bằng màu hoặc bằng cặp cột
"Doing / Done-waiting". Tổng thời gian ở các cột chờ chính là dư địa cải thiện của bạn.

**Bước 2 — Đo ba số, trong hai tuần, trước khi đổi bất cứ gì.**

- **Cycle time** của từng item: từ lúc bắt đầu làm đến lúc lên production. Báo cáo bằng trung vị và
  phân vị 85, không bằng trung bình (phân phối lệch phải, trung bình bị đuôi kéo).
- **Throughput**: số item lên production mỗi tuần.
- **Flow efficiency**: cần chút công — với 5–10 item, cộng thời gian ở các cột "đang làm" chia cho tổng
  thời gian. Không cần chính xác; cần biết bạn đang ở 20% hay 50%.

Lý do đo trước khi đổi: nếu không có đường cơ sở, mọi can thiệp về sau đều không chứng minh được, và
bạn sẽ không bảo vệ được nó khi có người muốn quay lại cách cũ.

**Bước 3 — Đặt WIP limit.** Công thức khởi đầu thực dụng, không phải công thức tối ưu:

```
WIP limit toàn hệ thống ≈ số người làm việc thực tế × 1.0 đến 1.5
```

Với team 6 người: giới hạn 6–9 item đang trong hệ thống (không tính Todo). So với 26 hiện tại, đây là
một cú giảm lớn — và đó là điểm.

Phân bổ theo cột (ví dụ cho team 6 người, số minh hoạ):

| Cột | WIP limit | Lý do |
|---|---|---|
| In Progress | 4 | Ít hơn số người, để buộc ghép cặp hoặc buộc đi giúp cột sau |
| Chờ review | 3 | Vượt 3 thì người tiếp theo phải review thay vì bắt đầu việc mới |
| QA | 3 | Bằng năng lực thực tế của QA trong 2 ngày |
| Chờ release | không giới hạn nhưng đo | Nếu tích thì vấn đề là tần suất release |

Quy tắc đi kèm quan trọng hơn con số: **khi một cột đầy, người vừa xong việc không được bắt đầu việc
mới; họ phải đi giúp cột đang đầy.** Không có quy tắc này thì WIP limit chỉ là một con số trang trí.
Quy tắc này gây khó chịu ban đầu — một senior BE phải đi review PR frontend, hoặc phải đi test tay —
và sự khó chịu đó chính là tín hiệu hệ thống đang buộc mọi người tối ưu tổng thể thay vì tối ưu cục bộ.

**Bước 4 — Nghi thức "ngừng bắt đầu, bắt đầu hoàn tất" (một buổi, làm ngay).** Với board 26 item:

1. Đóng băng: không item mới vào In Progress đến khi WIP về dưới giới hạn.
2. Đi từ phải sang trái: đẩy hết item ở QA và Review về Done trước, vì chúng gần giá trị nhất.
3. Với mỗi item In Progress, quyết định một trong ba: *hoàn tất trong tuần này*, *đưa về Todo và ghi
   nhận công đã bỏ ra*, hoặc *bỏ hẳn*. Không có lựa chọn "để đó".
4. Ghi lại số item bị đưa về Todo hoặc bỏ — đây là số đo lượng công việc dở dang mà tổ chức đã tích, và
   là dữ liệu để nói chuyện với stakeholder.

**Bước 5 — Nhịp đọc dòng chảy hàng tuần, 20 phút.** Không phải standup. Một cuộc gặp tuần với ba câu:

- Item nào đang ở trong hệ thống lâu hơn phân vị 85 của cycle time? (item già là rủi ro, không phải
  item bình thường bị chậm)
- Cột nào đang là nghẽn tuần này? Có gì đổi so với tuần trước?
- Có bao nhiêu item đang Blocked, tổng bao nhiêu ngày blocked, và blocked bởi ai? (đầu vào cho chủ đề 6)

**Output và tiêu chí xong:** board có cột chờ hiển thị; ba số được đo và cập nhật tự động nếu được; WIP
limit dán trên board; và bạn trả lời được câu hỏi của founder ở Problem Statement bằng số — "không cần
tuyển; nghẽn của ta là review với 6 ngày chờ trung bình; nếu ta đưa WIP từ 26 về 8 thì theo Little's
Law cycle time đi từ 13 tuần về khoảng 4 tuần với cùng số người."

### Trade-off

**Kanban vs Scrum — điều kiện chọn.**

| | Kanban | Scrum |
|---|---|---|
| Phù hợp loại công việc | Dòng chảy liên tục, item độc lập, nhiều interrupt | Công việc hướng một mục tiêu chu kỳ, cần đồng bộ nhiều người |
| Đơn vị cam kết | Từng item (hoặc SLA theo lớp dịch vụ) | Một lô theo chu kỳ (sprint goal) |
| Cơ chế giới hạn WIP | Theo cột, liên tục | Theo lô thời gian |
| Khả năng phản ứng việc gấp | Cao: đổi thứ tự hàng đợi ngay | Thấp: tối đa một chu kỳ |
| Khả năng dự báo | Dự báo theo phân phối cycle time lịch sử (thường chính xác hơn) | Dự báo theo velocity (dễ bị lạm phát đơn vị) |
| Áp lực cải tiến process | Liên tục nhưng khuếch tán, dễ trôi | Định kỳ và cưỡng bức bởi retro |
| Rủi ro thất bại đặc trưng | Không bao giờ có thời điểm "xong", team thiếu cảm giác hoàn thành | Cam kết thất bại đều đặn khi bị nhét việc |
| Phù hợp team | Vận hành, support, hạ tầng, maintenance, team trưởng thành | Product team làm tính năng mới, team cần nhịp học |

Điều kiện nghiêng về **Kanban**: hơn 30% công việc đến không dự báo được; item độc lập nhau; team đã có
kỷ luật tự quản (Kanban đòi nhiều kỷ luật hơn Scrum, vì không có nghi lễ cưỡng bức); cần dự báo theo
SLA từng item hơn là theo lô.

Điều kiện nghiêng về **Scrum**: cần đồng bộ nhiều người quanh một mục tiêu; team mới hoặc team đang mất
khả năng dự báo (nhịp cưỡng bức là công cụ dạy); stakeholder cần một điểm đồng bộ định kỳ; công việc có
mục tiêu kinh doanh theo chu kỳ.

**Mô hình lai, và cách làm cho nó không trở thành cái tệ nhất của cả hai.** Mô hình lai phổ biến và
dùng được:

- Giữ chu kỳ 2 tuần với sprint goal và retro (lấy phần nhịp học và điểm đồng bộ của Scrum).
- Trong chu kỳ, chạy board có WIP limit theo cột và đo cycle time (lấy phần dòng chảy của Kanban).
- Bỏ story point, dự báo bằng cycle time lịch sử và số item (bỏ phần dễ bị lạm phát).
- Có một lớp dịch vụ "expedite" giới hạn 1 item cùng lúc, dùng cho việc thực sự không hoãn được.

Điều kiện để mô hình lai không hỏng: sprint goal phải là mục tiêu, không phải danh sách; và WIP limit
phải có quy tắc "cột đầy thì đi giúp". Thiếu một trong hai thì bạn có Scrum lỏng cộng board đẹp.

**Đo cycle time vs đo số ticket đóng.** Số ticket đóng dễ đo, dễ hiểu, và sai hệ thống: nó tối ưu được
bằng cách chia nhỏ ticket, và nó không phản ánh thời gian khách hàng phải chờ. Cycle time khó đo hơn
một chút và đúng hơn nhiều. Nghiêng về số ticket chỉ khi item thực sự đồng đều về kích thước (rất ít
khi). Chi phí của việc dùng số ticket: bạn không thấy được nghẽn, vì một hệ thống nghẽn nặng vẫn đóng
đủ số ticket nhỏ.

**Utilization cao vs dư địa.** Ở góc nhìn tài chính, một engineer "chỉ làm 80% thời gian" trông như
20% lãng phí — và ở ODC, utilization rate là chỉ số được báo cáo cho khách và cho ban lãnh đạo. Ở góc
nhìn dòng chảy, 20% đó là thứ giữ cho cycle time hữu hạn. Đây là một xung đột thật, không giải được
bằng cách nói ai đúng. Cách xử lý thực tế: chấp nhận utilization cao ở nơi công việc *đều* và có thể
lập kế hoạch trước (ví dụ: một đợt migration đã biết hết việc), giữ dư địa ở nơi công việc *biến động*
(team có on-call, team nhận yêu cầu từ bên khác). Và khi bị chất vấn, dùng bảng ρ/(1−ρ) ở trên — đó là
lập luận định lượng, không phải lời xin.

### Real-world Scenarios

**Tình huống A — Cột "Done" nhưng thực tế còn chờ QA và deploy.**

Bối cảnh: logistics Việt, team 8 người, board có cột Done. Mỗi cuối sprint team báo "hoàn thành 14
story". Nhưng dữ liệu production cho thấy trung bình 5 tuần từ lúc story được đánh Done đến lúc khách
hàng dùng được (số minh hoạ).

Cơ chế của vấn đề: cột "Done" của team được định nghĩa là "code đã merge". Sau đó còn: chờ QA của phòng
QA trung tâm (trung bình 8 ngày), chờ ký UAT của phòng nghiệp vụ (6 ngày), chờ cửa sổ release hàng
tháng (trung bình 11 ngày). Không đơn vị nào trong ba đơn vị đó nằm trên board của team, nên không ai
sở hữu 25 ngày đó, và không ai báo cáo nó.

Hệ quả cụ thể và đắt: PO cam kết với phòng kinh doanh dựa trên "team báo xong tuần này", rồi phải xin
lỗi 5 tuần liền. Sau ba lần, phòng kinh doanh ngừng tin PO và bắt đầu gọi trực tiếp cho engineer.

Cách xử lý của Sơn (Tech Lead):

1. **Đưa ba hàng đợi lên board**, dù ba giai đoạn đó không do team làm. Lập luận với các phòng: "chúng
   tôi không quản việc của các anh; chúng tôi chỉ hiển thị nơi công việc đang nằm."
2. **Đổi tên cột:** "Done" thành "Dev Done"; thêm cột "Live". Chỉ đo throughput ở cột Live. Đây là thay
   đổi có tác động lớn nhất và tốn ít công nhất — nó đổi cái mà tổ chức nhìn.
3. **Đo và công bố tổng thời gian ở ba hàng đợi ngoài team**: 25 trong 33 ngày cycle time nằm ngoài
   team. Con số này chuyển cuộc thảo luận từ "team dev chậm" sang "hệ thống của công ty chậm".
4. **Tấn công hàng đợi lớn nhất trước:** cửa sổ release hàng tháng. Đề xuất release hai tuần một lần
   cho các thay đổi không ảnh hưởng luồng tiền, giữ nguyên nhịp tháng cho phần còn lại. Đây là đòn bẩy
   rẻ nhất vì nó không cần thêm người.
5. **Với UAT:** đề xuất phòng nghiệp vụ chỉ định một người trực UAT 2 giờ mỗi thứ Ba và thứ Năm, thay
   vì "khi nào rảnh". Giảm biến động quan trọng hơn giảm thời gian.

Sau một quý (số minh hoạ): cycle time đến Live từ 33 xuống 17 ngày, trong đó phần trong team gần như
không đổi (8 xuống 7 ngày). Toàn bộ cải thiện đến từ hàng đợi — và không ai phải làm việc nhanh hơn.

**Tình huống B — Không có WIP limit nên mọi việc "đang làm": ba góc nhìn.**

Cùng board của team Checkout ở Problem Statement, ngày thứ Ba, WIP = 26. Ba tầng đọc cùng một hiện
tượng:

- **Nhìn từ IC (Nam, Senior BE, có KH-191 đã 12 ngày và KH-206 mới 1 ngày):** "Tôi mở KH-206 vì KH-191
  đang chờ Vy review từ thứ Năm tuần trước, và tôi không muốn ngồi không — vừa vì cảm giác, vừa vì tôi
  biết cuối sprint người ta đếm số ticket. Nếu tôi chỉ ngồi chờ review, trên board tôi trông như không
  làm gì." Nam đang hành xử hợp lý theo động lực Nam chịu. Việc Nam *có thể* làm khác: đi review PR của
  người khác thay vì mở việc mới — nhưng Nam không làm vì review không được đếm là đầu ra của Nam.
- **Nhìn từ Tech Lead (Vy):** "WIP 26 với 6 người là con số nói lên hệ thống, không nói lên ai lười.
  Nghẽn của tôi là review — 9 PR chờ, cũ nhất 6 ngày — và tôi là một phần của nghẽn đó vì tôi là người
  duy nhất review được phần payment. Ba việc tôi phải làm: (a) ngay tuần này, đóng băng việc mới và
  dồn vào đóng cột review; (b) đặt WIP limit và quy tắc 'cột đầy thì đi giúp'; (c) trong hai tháng, làm
  cho ít nhất hai người nữa review được phần payment — đây là việc quan trọng nhất và cũng là việc dễ
  bị trì hoãn nhất, vì nó không có deadline."
- **Nhìn từ Manager (Hà, EM):** "Tôi thấy hai vấn đề khác. Một: hệ thống đánh giá của tôi đang khuyến
  khích Nam mở việc mới — nếu review và pair không được tính là đóng góp trong performance review thì
  không ai làm. Đây là vấn đề của tôi, không của Nam. Hai: bus factor 1 ở payment là rủi ro tổ chức, và
  tôi đã biết điều này ba quý và chưa cấp thời gian cho nó vì mỗi quý đều có việc gấp hơn. Việc của
  tôi: đưa 'mở rộng số người review được payment' thành một mục có capacity trong quarterly plan (chủ
  đề 5), và đổi tiêu chí đánh giá để việc unblock người khác được tính (chương 08)."

Điểm rút ra: WIP cao là hiện tượng ở tầng Execution nhưng nguyên nhân nằm ở ba tầng khác nhau — thói
quen cá nhân, thiếu quy tắc dòng chảy, và hệ thống động lực. Một Tech Lead chỉ đặt WIP limit mà không
có EM sửa hệ thống đánh giá sẽ thấy WIP limit bị lách trong hai tháng.

### Best Practices

**Đo cycle time và flow efficiency trước khi đổi bất cứ gì, và báo cáo bằng trung vị và phân vị 85.**
Lý do: trung bình bị đuôi phải kéo lệch; phân vị 85 là con số dùng được để cam kết với stakeholder
("85% story lên production trong vòng 12 ngày" là một phát biểu kiểm được).

**Hiển thị mọi hàng đợi, kể cả hàng đợi thuộc phòng khác.** Lý do: thời gian chờ không sở hữu bởi ai
thì không được ai cải thiện. Hiển thị là bước duy nhất cần thiết để tạo chủ sở hữu.

**Đặt WIP limit thấp hơn mức thoải mái, kèm quy tắc "cột đầy thì đi giúp".** Lý do: WIP limit không có
quy tắc đi kèm chỉ tạo ra việc lách (mở ticket phụ, làm việc ngoài board). Quy tắc đi giúp là thứ biến
giới hạn thành hành vi.

**Tấn công nghẽn, không tấn công người ở nghẽn.** Lý do: người ở nghẽn thường là người giỏi nhất và
đang bị quá tải nhất. Tăng áp lực lên họ làm biến động tăng (kiệt sức, nghỉ việc), và biến động tăng
làm thời gian chờ tăng — ngược hướng mong muốn.

**Coi item già hơn phân vị 85 là rủi ro cần xử lý riêng, không phải việc chậm bình thường.** Lý do:
phân phối cycle time có đuôi dài, và đuôi đó là nơi tập trung rủi ro dự án. Một item ở trong hệ thống
40 ngày thường có nguyên nhân khác hẳn (phụ thuộc, yêu cầu không rõ, người sở hữu đã chuyển việc), và
xử lý nó bằng cách "cố gắng hơn" không có tác dụng.

**Giữ dư địa capacity ở nơi công việc biến động, và bảo vệ nó bằng lập luận định lượng.** Lý do: bảng
ρ/(1−ρ). Cách bảo vệ: đưa nó thành một bucket có tên trong capacity plan (chủ đề 5), không phải một
khoảng trống không tên — khoảng trống không tên luôn bị lấp.

**Ưu tiên giảm biến động trước khi giảm thời gian trung bình.** Lý do: trong lý thuyết hàng đợi, biến
động của thời gian phục vụ ảnh hưởng đến thời gian chờ mạnh gần như utilization. Một QA test đều đặn 2
giờ mỗi story tạo ít hàng đợi hơn một QA test có khi 20 phút có khi 2 ngày, dù trung bình bằng nhau.

### Anti-patterns

**Cột "Done" nhưng thực tế còn chờ QA hoặc deploy.** Cơ chế: nó tạo ra hai hệ quả cùng lúc. Một, tổ
chức đo throughput ở một điểm không phải điểm tạo giá trị, nên tối ưu sai chỗ. Hai, nó tạo một khối
thời gian không ai sở hữu, và khối đó luôn phình. Dấu hiệu sớm: khoảng cách giữa "team báo xong" và
"khách dùng được" trên 5 ngày mà không có ai báo cáo con số đó; hoặc có cột nào trên board mà khi hỏi
"ai chịu trách nhiệm đẩy thẻ ra khỏi cột này" thì không có câu trả lời.

**Không có WIP limit nên mọi việc "đang làm".** Cơ chế: WIP cao là một vòng phản hồi dương — chờ nhiều
làm người ta bắt đầu việc mới, việc mới làm chờ nhiều hơn. Hệ thống tự đi tới trạng thái WIP tối đa mà
con người chịu được, và ở đó cycle time là tệ nhất có thể. Dấu hiệu sớm: số item In Progress vượt số
người; câu "tôi đang làm ba việc song song" được nói ra như một thành tích.

**Đo số ticket đóng thay vì cycle time.** Cơ chế: ticket là đơn vị do người bị đo tự định nghĩa. Nó bị
tối ưu bằng cách chia nhỏ, và việc chia nhỏ để đẹp số làm tăng overhead phối hợp. Đồng thời nó ẩn hoàn
toàn thời gian chờ, tức ẩn 70–80% cycle time ở tổ chức chưa tối ưu. Dấu hiệu sớm: số ticket đóng tăng
trong khi thời gian từ yêu cầu đến production không giảm; kích thước trung bình của ticket giảm dần
theo quý.

**Đặt WIP limit rồi lách bằng cách làm việc ngoài board.** Cơ chế: khi giới hạn có hiệu lực nhưng áp
lực đầu vào không giảm, công việc chuyển sang kênh không nhìn thấy — Slack riêng, task ngầm, "giúp một
tay 30 phút". Board sạch nhưng cycle time không cải thiện, và giờ bạn còn mất cả khả năng quan sát. Dấu
hiệu sớm: board trông tốt mà team vẫn quá tải; các item xuất hiện trên board đã ở trạng thái gần xong.

**Tối ưu utilization của từng người.** Cơ chế: mỗi người được lấp 100% thời gian nghĩa là không ai có
dư địa để giúp cột đang nghẽn, và mọi biến động biến thành hàng đợi. Đây là tối ưu cục bộ điển hình:
mỗi phần đạt hiệu suất cao nhất, tổng thể chậm nhất. Dấu hiệu sớm: có báo cáo utilization theo người và
nó được dùng để phân việc; câu "anh này đang rảnh, giao thêm việc đi".

**Đổi công cụ thay vì đổi dòng chảy.** Cơ chế: chuyển từ Jira sang một công cụ khác không đổi thời gian
chờ review, không đổi tần suất release, không đổi bus factor. Nó tiêu 2–6 tuần và tạo cảm giác đã làm
gì đó. Dấu hiệu sớm: giải pháp cho vấn đề dòng chảy được đề xuất dưới dạng tên một sản phẩm SaaS.

### Khi nào KHÔNG nên áp dụng

**Khi công việc là một khối lớn không chia được, và chia nhỏ tốn kém hơn lợi ích.** Ví dụ: một migration
database phải làm nguyên khối, hoặc một chứng chỉ compliance phải đạt toàn bộ hoặc không gì. Ở đây board
với 8 cột và WIP limit không cho bạn gì; công cụ đúng là một kế hoạch tuần tự có milestone và một danh
sách rủi ro. Vẫn giữ một thứ từ Kanban: đo thời gian chờ ở các điểm phê duyệt.

**Khi team quá nhỏ để có hàng đợi.** Với 2 người, WIP tự nhiên đã thấp và nghẽn thì rõ ràng không cần
board để thấy. Nghi thức đo dòng chảy hàng tuần ở đây là overhead. Điều vẫn nên làm: quy tắc "không mở
việc thứ hai khi việc thứ nhất chưa xong".

**Khi Kanban được dùng để tránh cam kết.** Có một dạng lạm dụng cần gọi tên: team không muốn cam kết
ngày nên chuyển sang Kanban và nói "chúng tôi làm theo dòng chảy, không cam kết". Điều này bỏ mất phần
đắt nhất mà stakeholder cần. Kanban trưởng thành *có* cam kết — dưới dạng SLA theo lớp dịch vụ ("việc
loại Standard: 85% xong trong 12 ngày") và dự báo theo phân phối lịch sử. Nếu bạn chuyển sang Kanban mà
không thiết lập được cam kết dạng đó, bạn đang lùi khỏi trách nhiệm, không đang cải tiến process.

**Khi tổ chức chưa chịu được việc nhìn thấy sự thật.** Đưa hàng đợi của phòng khác lên board là một
hành động chính trị. Ở tổ chức mà việc chỉ ra một phòng khác là nghẽn bị đọc thành cáo buộc, làm bước
này quá sớm có thể tạo phản ứng đủ mạnh để bạn mất luôn cả những cải tiến trong tầm tay. Thứ tự đúng:
tối ưu phần trong tầm kiểm soát trước (giảm WIP nội bộ, giảm thời gian review), tích luỹ uy tín bằng
kết quả, rồi mới đưa dữ liệu về hàng đợi bên ngoài — và khi đưa thì đưa cho người có thẩm quyền trên cả
hai phòng, không đưa ra họp chung. Cách xây dựng ảnh hưởng ngoài phạm vi quyền hạn ở chương 02 và 09.

**Khi nghẽn thật nằm ở tầng trên và không ở dòng chảy.** Nếu cột Todo dài 23 item vì tổ chức không có
cơ chế ưu tiên, thì tối ưu dòng chảy chỉ làm bạn giao nhanh hơn những thứ có thể không nên làm. Đây là
sai lầm dễ mắc ở người mới học flow: tăng hiệu suất của một hệ thống đang làm việc sai là làm cho việc
sai xảy ra nhanh hơn. Kiểm tra trước: trong 10 item vừa lên production, bao nhiêu item có bằng chứng
tạo giá trị? Nếu dưới một nửa, vấn đề của bạn ở chủ đề 5, không ở chủ đề 3.


---

## 4. Estimation

### Problem Statement

Một ví điện tử, team 6 người. Tháng 3, Trang (PO) hỏi: "Tính năng rút tiền về thẻ mất bao lâu?" Team
họp 3 giờ, chia ra 41 story, ước lượng bằng story point, tổng 128 điểm. Velocity 6 sprint gần nhất:
38, 45, 31, 52, 29, 44 (số minh hoạ). Trung bình 39,8. Chia ra: 3,2 sprint. Minh (Tech Lead) báo:
"khoảng 3 sprint, tức 6 tuần." Câu đó đi vào slide gửi ban lãnh đạo dưới dạng một dòng: *Rút tiền về
thẻ — 6 tuần*. Marketing lên lịch chiến dịch tuần thứ 8. Thực tế tính năng lên production ở tuần 11.

Hãy mổ con số 6 tuần đó. Dãy velocity trên có độ lệch chuẩn khoảng 8,8 — hệ số biến động 22%. Chỉ tính
riêng biến động của velocity, 128 điểm đã cho khoảng 2,5–4,2 sprint, tức 5–8,5 tuần. Nhưng khoảng đó
vẫn không giải thích được 11 tuần, vì nguyên nhân thật nằm ngoài mô hình:

| Nguồn của 5 tuần chênh | Tuần | Có nằm trong ước lượng ban đầu? |
|---|---|---|
| 4 story phát sinh sau khi làm rõ quy tắc với ngân hàng đối tác | 1,5 | Không — chưa biết lúc ước lượng |
| Chờ ngân hàng cấp môi trường test (yêu cầu ngày 12/3, có ngày 26/3) | 2,0 | Không — không có ai sở hữu việc theo đuổi |
| Đối soát ngược cho giao dịch thất bại, phát hiện ở UAT | 1,0 | Không — nằm trong DoD nhưng không nằm trong story nào |
| Biến động velocity thông thường | 0,5 | Có |

Bốn trong năm nguyên nhân không phải lỗi cộng số. Nghĩa là toàn bộ 3 giờ họp ước lượng — 18 giờ-người —
đã đầu tư vào việc làm chính xác hơn phần chỉ chiếm 10% sai số. Đây là hình dạng phổ biến nhất của bài
toán estimation: nỗ lực đổ vào phần đã biết, còn sai số đến từ phần chưa biết.

Hậu quả quan sát được khi tổ chức không có năng lực này, xếp theo thứ tự xuất hiện:

- **Chu kỳ 1:** một con số nội bộ ("6 tuần") được truyền ra ngoài mà mất hết ngữ cảnh và điều kiện. Các
  quyết định bên ngoài — lịch marketing, ngày ký với đối tác, kế hoạch tuyển dụng — được gắn vào nó.
- **Chu kỳ 2:** team crunch tuần cuối để cứu con số, cắt test ngầm (chủ đề 8), nợ được tạo mà không ai
  ghi.
- **Chu kỳ 3:** stakeholder học được hệ số hiệu chỉnh riêng của họ. Câu nói thật xuất hiện trong hành
  lang: "Minh nói 6 tuần thì cứ tính 10." Từ lúc này, mọi ước lượng của team mất giá trị thông tin —
  kể cả những ước lượng đúng.
- **Chu kỳ 4:** trò chơi hình thành. Team padding ngầm 40%, PO cắt xuống 25% theo phản xạ, và không bên
  nào đang nói về công việc thật nữa. Đây là trạng thái hấp thụ: rất khó ra, vì để ra được thì một bên
  phải nói thật trước trong khi bên kia còn đang chơi.

Ba câu hỏi chẩn đoán nhanh cho tổ chức của bạn:

| Câu hỏi | Nếu câu trả lời là... | Thì bạn đang ở |
|---|---|---|
| Ước lượng cuối cùng bạn đưa ra là một con số hay một khoảng? | Một con số | Chu kỳ 1 hoặc 2 |
| Stakeholder của bạn có nhân hệ số vào con số của bạn không? | Có, và họ nói ra | Chu kỳ 3 |
| Team có padding mà không nói ra là padding không? | Có | Chu kỳ 4 |

### First Principles

**Nguyên lý 1 — Thời gian hoàn thành là một phân phối lệch phải, không phải một con số.**

Cơ chế: thời gian làm xong một việc phần mềm là tổng của nhiều biến ngẫu nhiên bị chặn dưới nhưng không
bị chặn trên. Có một giới hạn vật lý cho việc "nhanh nhất có thể" — không ai viết xong module đối soát
trong 2 giờ — nhưng không có giới hạn cho việc "chậm đến đâu": một API của đối tác trả sai định dạng có
thể tiêu hai tuần. Phân phối như vậy lệch phải, và hệ quả toán học của lệch phải là **trung bình lớn hơn
trung vị**. Người ước lượng hình dung kịch bản điển hình, tức gần trung vị. Người nghe hiểu đó là điều
sẽ xảy ra, tức trung bình. Hai người đang nói về hai con số khác nhau bằng cùng một từ.

Cùng một câu "6 tuần" có ba nghĩa hoàn toàn khác nhau, và việc không phân biệt chúng là nguồn gốc của
phần lớn xung đột về ước lượng:

| Cách đọc "6 tuần" | Nghĩa | Xác suất đúng | Ai thường đọc theo cách này |
|---|---|---|---|
| P50 (trung vị) | Một nửa khả năng xong trước, một nửa sau | 50% | Engineer nói ra con số |
| Kỳ vọng | Trung bình nếu chạy dự án này 100 lần | 40–45% với phân phối lệch phải | Không ai — nhưng đây mới là số đúng để lập kế hoạch tổng hợp |
| Cam kết | Sẽ xong ngày đó, nếu không là thất bại | Bị đọc là 95% | Stakeholder nghe con số |

Khoảng cách giữa dòng 1 và dòng 3 là 45 điểm phần trăm xác suất. Không có kỹ thuật ước lượng nào bù được
khoảng cách đó; chỉ có việc nói rõ mình đang nói dòng nào.

**Nguyên lý 2 — Sai số ước lượng là sai số hệ thống, không phải sai số ngẫu nhiên.**

Nếu sai số là ngẫu nhiên, ước lượng sẽ lệch hai phía và trung bình dài hạn sẽ đúng. Thực tế nó gần như
luôn lệch một phía. Bốn cơ chế, và điều quan trọng là chúng cộng dồn chứ không bù nhau:

- *Planning fallacy* (Kahneman và Tversky, nghiên cứu công khai): con người ước lượng bằng cách dựng một
  kịch bản trong đầu (inside view), chứ không bằng cách tra dữ liệu quá khứ của những việc cùng loại
  (outside view). Kịch bản dựng trong đầu bao giờ cũng là kịch bản không có gì sai — vì để hình dung ra
  một sai cụ thể, bạn phải đã biết nó, và cái đã biết thì đã được tính vào rồi.
- *Optimism bias và áp lực xã hội*: trong một phòng có PO và có người trả lương, con số thấp được đón
  nhận tốt hơn. Không cần ai gây áp lực rõ ràng; chỉ cần biểu cảm khi nghe con số cao là đủ. Ở bối cảnh
  Việt Nam, thêm một tầng: nói con số cao trước mặt người lớn tuổi hơn hoặc cấp trên dễ bị đọc là năng
  lực kém hoặc thiếu tinh thần, nên engineer trẻ có động lực rõ ràng để nói thấp.
- *Công việc chưa biết không được ước lượng*: đây là nguyên nhân lớn nhất và ít được nói nhất. Ước lượng
  chỉ bao phủ những gì đã hình dung được thành story. Phần chưa hình dung được — quy tắc nghiệp vụ chưa
  ai hỏi, trường hợp lỗi của đối tác, việc đối soát, việc di trú dữ liệu cũ — bằng định nghĩa là 0 trong
  bảng tính, và bằng thực nghiệm là 20–50% khối lượng thật ở một dự án tích hợp.
- *Chi phí phối hợp tăng theo bình phương số người*: mỗi người thêm vào không cộng thêm capacity tuyến
  tính, vì số cặp cần đồng bộ là n(n−1)/2. Ước lượng dạng "128 điểm chia cho 6 người" ngầm giả định phối
  hợp miễn phí.

| Số người | Số cặp cần đồng bộ | Nếu mỗi cặp tốn 1 giờ/tuần | % của một tuần 40 giờ mỗi người |
|---|---|---|---|
| 3 | 3 | 3 giờ | 2,5% |
| 5 | 10 | 10 giờ | 5% |
| 8 | 28 | 28 giờ | 8,75% |
| 12 | 66 | 66 giờ | 13,75% |

Số minh hoạ, và cố tình đơn giản hoá — nhưng độ dốc là thật, và nó giải thích vì sao thêm người vào một
dự án đang trễ thường làm nó trễ hơn (chủ đề 7).

Thêm một cơ chế phá đệm: *student syndrome* — đệm đặt trong từng task bị tiêu ngay ở đầu task, vì việc
chỉ bắt đầu thật khi deadline đủ gần. Và *Parkinson's law* — công việc giãn ra lấp hết thời gian có sẵn.
Kết hợp lại: đệm phân tán vào từng task gần như luôn bị mất, còn phần trễ vẫn tích lại.

**Nguyên lý 3 — Vì sao ước lượng chính xác hơn không giải quyết được bài toán, mà giảm kích thước lô mới
giải quyết.**

Đây là điểm quan trọng nhất của chủ đề này, và cũng là điểm phản trực giác nhất.

Nguồn của phương sai là bất định về *nội dung* công việc, không phải kỹ năng cộng số của team. Đầu tư
thêm giờ vào ước lượng làm giảm phương sai của phần đã biết. Nó không chạm được vào phương sai của phần
chưa biết, và phần chưa biết chiếm phần lớn. Đây là lý do một team đầu tư 6 giờ ước lượng thay vì 3 giờ
không cho kết quả tốt hơn hai lần; thường không tốt hơn chút nào, chỉ tự tin hơn — mà tự tin không kèm
độ chính xác là một sự xuống cấp, vì nó làm bạn bỏ điểm kiểm tra.

Cái thực sự hoạt động: giảm kích thước lô. Cơ chế có hai phần độc lập, và cả hai đều mạnh.

*Phần một — sai số tuyệt đối tỉ lệ với kích thước lô.* Nếu sai số tương đối của bạn ổn định ở khoảng
±60% (con số hợp lý cho việc chưa từng làm), thì một lô 12 tuần cho sai số ±7 tuần, còn một lô 2 tuần
cho sai số ±1,2 tuần. Cùng một năng lực ước lượng, hậu quả khác nhau 6 lần.

*Phần hai — lô nhỏ tạo điểm hiệu chỉnh.* Với một lô 12 tuần, bạn biết mình sai ở tuần 11. Với sáu lô 2
tuần, bạn biết mình sai ở tuần 2, và còn 5 cơ hội để điều chỉnh scope, xin thêm người, hoặc báo sớm.

| | Một lô 12 tuần | Sáu lô 2 tuần |
|---|---|---|
| Sai số ước lượng (±60% tương đối) | ±7,2 tuần | ±1,2 tuần mỗi lô |
| Biết mình lệch vào thời điểm | Tuần 10–11 | Tuần 2 |
| Số lần hiệu chỉnh được | 0 | 5 |
| Chi phí hiệu chỉnh mỗi lần | Rất cao (đã đầu tư 10 tuần) | Thấp |
| Giá trị đến tay người dùng | Tuần 12 hoặc muộn hơn | Từ tuần 2 (nếu lô giao được độc lập) |

Số minh hoạ. Điều kiện then chốt ở dòng cuối: lô chỉ có giá trị này khi mỗi lô *giao được* hoặc ít nhất
*kiểm chứng được* độc lập. Chia một khối 12 tuần thành sáu mảnh chỉ ghép lại được ở cuối thì bạn có sáu
lần báo cáo, không có sáu lần học.

Câu hỏi đúng, do đó, không phải "làm sao ước lượng chính xác hơn" mà là **"làm sao để việc ước lượng sai
gây thiệt hại ít nhất"**. Hai câu hỏi này dẫn tới hai loại đầu tư hoàn toàn khác nhau: câu thứ nhất dẫn
tới nhiều buổi họp hơn; câu thứ hai dẫn tới lô nhỏ hơn, feature flag, release thường xuyên hơn, và cam
kết có điều kiện.

### Mental Model

**Cone of Uncertainty.** Độ rộng của khoảng ước lượng thu hẹp theo *tiến trình thực tế của công việc*,
không theo nỗ lực bỏ vào việc ước lượng. Ở thời điểm chỉ có một ý tưởng, khoảng thực tế là khoảng
0,25x–4x — nghĩa là "6 tuần" thật ra là "1,5 đến 24 tuần". Sau khi làm rõ yêu cầu, khoảng còn khoảng
0,6x–1,6x. Sau khi hoàn thành 30% công việc thật, khoảng còn khoảng 0,85x–1,15x. Cách map vào công việc:
khi ai đó yêu cầu một con số hẹp ở giai đoạn sớm, họ đang yêu cầu một thứ không tồn tại. Phản hồi đúng
không phải từ chối, mà là *bán cho họ cách thu hẹp hình nón*: "Cho tôi 5 ngày spike, tôi sẽ đưa anh
khoảng hẹp hơn ba lần." Bạn đổi thời gian lấy độ chắc chắn — đó là một giao dịch mà stakeholder hiểu
được, khác với việc từ chối trả lời.

**Reference Class Forecasting (outside view).** Thay vì phân tích việc này từ trong ra, tìm lớp việc
tương tự đã làm và dùng phân phối thực tế của lớp đó. Cách map: câu hỏi đổi từ "việc này gồm những gì và
mỗi phần mất bao lâu" sang "bốn lần trước ta tích hợp một cổng thanh toán mới, thực tế mất bao lâu".
Điểm mạnh: nó tự động bao gồm cả những cái không biết, vì những cái không biết đã nằm trong số liệu quá
khứ. Điểm yếu: cần dữ liệu, và cần trung thực về việc lần này có thật cùng lớp không. Đây là mô hình có
tỉ lệ hiệu quả/công sức cao nhất trong toàn bộ chủ đề, và cũng là mô hình bị bỏ nhiều nhất — vì nó nhàm,
không có thảo luận kỹ thuật thú vị, và nó thường cho ra con số mà không ai muốn nghe.

**Buffer tập trung (từ Critical Chain).** Thay vì mỗi task mang đệm riêng — nơi đệm bị student syndrome
tiêu mất — gom đệm về một chỗ ở cấp dự án và quản trị nó công khai. Cách map: ước lượng từng khối ở mức
P50 (không padding), rồi thêm một buffer dự án bằng phần chênh tới P90 của tổng, và **hiển thị buffer đó
như một dòng riêng trong kế hoạch**. Hệ quả hành vi quan trọng: khi buffer là một dòng nhìn thấy được,
việc tiêu 30% buffer ở tuần 2 trở thành một tín hiệu có thể báo cáo, chứ không phải một cảm giác. Đây
cũng là cách hợp pháp hoá padding: bạn vẫn có đệm, nhưng nó được nói ra, có kích thước, và có người theo
dõi mức tiêu.

| Mô hình | Trả lời câu hỏi nào | Dùng ở thời điểm |
|---|---|---|
| Cone of Uncertainty | "Con số này chắc đến đâu, và làm sao chắc hơn?" | Khi bị ép cho số sớm |
| Reference Class Forecasting | "Việc cùng loại thực tế mất bao lâu?" | Khi có lịch sử; luôn nên làm trước inside view |
| Buffer tập trung | "Đệm của tôi ở đâu và còn bao nhiêu?" | Khi lập kế hoạch nhiều tuần trở lên |

### Practical Framework

**Bước 1 — Phân loại câu hỏi trước khi ước lượng.** Phần lớn buổi ước lượng chi tiết được tổ chức để
trả lời một câu hỏi không cần ước lượng chi tiết. Bảng dưới là bước rẻ nhất trong toàn bộ framework:

| Câu hỏi thật của stakeholder | Công cụ đúng | Chi phí | Công cụ sai thường dùng |
|---|---|---|---|
| "Việc này có khả thi không?" | Spike có timebox, tiêu chí kết thúc viết trước | 2–5 ngày, 1 người | Họp ước lượng 3 giờ × 7 người |
| "Nên làm cái nào trước?" | So sánh tương đối (t-shirt size) + giá trị | 30 phút | Ước lượng tuyệt đối từng story |
| "Kịp ngày 30/11 không?" | Reference class + P50/P90 + danh sách điều kiện | 2–4 giờ | Một con số duy nhất |
| "Bao giờ xong toàn bộ?" | Dự báo từ throughput lịch sử (chủ đề 3) | 20 phút nếu có dữ liệu | Cộng story point của cả backlog |
| "Tôi cần cam kết cho hợp đồng" | Ước lượng khoảng + assumption list thành điều khoản | 1–2 ngày | Padding ngầm |

Tiêu chí biết là xong bước này: bạn viết được một câu "quyết định mà con số này phục vụ là ___". Nếu
không viết được, đừng ước lượng — hỏi lại.

**Bước 2 — Ước lượng khoảng P50/P90 theo khối, không theo story.** Quy trình:

1. Chia công việc thành 4–8 khối ở mức có thể mô tả bằng một câu. Không chia nhỏ hơn ở giai đoạn này:
   chia nhỏ tạo cảm giác chính xác mà không tạo độ chính xác.
2. Mỗi khối, 3 người ước lượng độc lập (viết ra trước khi nói) hai số: P50 và P90. Định nghĩa phải đọc
   thành tiếng: *"P90 nghĩa là nếu ta làm việc này 10 lần, có 9 lần xong trong khoảng đó."*
3. Chỗ nào ba người lệch nhau quá 2 lần ở P90 — dừng lại, đó không phải bất đồng về ước lượng mà là bất
   đồng về *phạm vi công việc*. Làm rõ phạm vi rồi ước lượng lại. Đây là giá trị lớn nhất của việc ước
   lượng nhiều người: nó là công cụ phát hiện hiểu khác nhau, không phải công cụ lấy trung bình.
4. Lấy P50 và P90 đồng thuận cho mỗi khối. Với mỗi khối tính độ phân tán: σ ≈ (P90 − P50) / 1,28.
5. Cộng khối: P50 tổng = Σ P50. σ tổng = √(Σ σ²). P90 tổng = P50 tổng + 1,28 × σ tổng.

Đừng cộng P90 của các khối lại với nhau — nó cho một con số quá bảo thủ đến mức không ai tin, vì nó giả
định mọi khối cùng lúc gặp trường hợp xấu. Ví dụ tính, số minh hoạ:

| Khối | P50 (tuần) | P90 (tuần) | σ = (P90−P50)/1,28 | σ² |
|---|---|---|---|---|
| Luồng rút tiền phía app | 1,5 | 2,5 | 0,78 | 0,61 |
| Tích hợp API ngân hàng | 2,0 | 5,0 | 2,34 | 5,48 |
| Đối soát và xử lý giao dịch lỗi | 1,5 | 3,5 | 1,56 | 2,43 |
| Màn hình quản trị cho CS | 1,0 | 1,5 | 0,39 | 0,15 |
| Kiểm thử, UAT, phát hành | 1,0 | 2,0 | 0,78 | 0,61 |
| **Tổng** | **7,0** | (không cộng) | √9,28 = **3,05** | **9,28** |

P50 tổng = 7,0 tuần. P90 tổng = 7,0 + 1,28 × 3,05 = **10,9 tuần**. Cộng P90 thô sẽ cho 14,5 tuần — quá
bảo thủ. Cách trình bày với stakeholder: *"Trung vị 7 tuần, và 9 trên 10 khả năng xong trong 11 tuần.
Khoảng rộng vì một khối duy nhất — tích hợp API ngân hàng — chiếm 59% toàn bộ phương sai."*

Dòng cuối là phần có giá trị nhất và hay bị bỏ: bảng này không chỉ cho con số, nó cho biết **rủi ro tập
trung ở đâu**. σ² = 5,48 trên tổng 9,28 nghĩa là nếu bạn chỉ được giảm bất định ở một chỗ, chỗ đó là API
ngân hàng — và cách giảm là một spike hoặc một cuộc gọi với đối tác trong tuần này, không phải một buổi
ước lượng nữa.

Một cảnh báo về công thức: √(Σσ²) giả định các khối độc lập. Nếu hai khối cùng phụ thuộc một thứ (cùng
chờ môi trường của ngân hàng), chúng tương quan và công thức cho kết quả lạc quan. Cách xử lý thực dụng:
gộp các khối chia chung một phụ thuộc thành một khối, hoặc nhân σ tổng với 1,3.

**Bước 3 — Reference class forecasting, quy trình 5 bước.**

1. Phát biểu lớp: "tích hợp một cổng thanh toán/ngân hàng mới, từ kick-off đến production".
2. Liệt kê mọi lần đã làm trong 24 tháng — kể cả lần đã bị bỏ giữa (đặc biệt là lần đó).
3. Ghi thời gian *thực tế theo lịch* (không phải giờ-người, không phải "thời gian làm thuần").
4. Sắp xếp, lấy trung vị và giá trị ở khoảng 90%.
5. Chỉ điều chỉnh khi có lý do *cấu trúc* nêu tên được, và ghi rõ mức điều chỉnh.

| Lần | Đối tác | Thời gian thực tế (tuần) | Ghi chú |
|---|---|---|---|
| 1 | Cổng A | 6 | Lần đầu, không có kinh nghiệm |
| 2 | Cổng B | 4 | Tài liệu tốt, sandbox có ngay |
| 3 | Ngân hàng C | 9 | Chờ môi trường 3 tuần |
| 4 | Ngân hàng D | 7 | Đối soát phức tạp |
| **Lớp tham chiếu** | | **P50 ≈ 6,5 · P90 ≈ 9** | |

Cách dùng: nếu inside view của bạn cho 4 tuần trong khi lớp tham chiếu cho P50 6,5 tuần, mặc định tin
lớp tham chiếu. Muốn dùng 4 tuần thì phải nêu được thay đổi cấu trúc: "lần này đối tác đã có sandbox mở
sẵn từ đầu, và đây là lần thứ hai ta dùng cùng SDK" — hai điều kiện kiểm được, không phải cảm giác "lần
này ta giỏi hơn".

**Bước 4 — Chọn đơn vị ước lượng.** Không có đơn vị tốt nhất; có đơn vị đúng cho một hoàn cảnh.

| Đơn vị | Dùng khi | Điều kiện tiên quyết | Cách nó hỏng |
|---|---|---|---|
| **Story point** | Team ổn định 4+ sprint, cần dự báo nhiều sprint, backlog đồng chất | Không quy đổi ra giờ; không so giữa các team; có 6+ chu kỳ dữ liệu | Bị quy đổi thành giờ; thành chỉ tiêu năng suất; velocity bị so sánh giữa team |
| **T-shirt size (S/M/L/XL)** | Giai đoạn sớm, cần xếp thứ tự ưu tiên, hoặc lượng hoá cả release | Có quy ước khoảng thời gian tương ứng và ngưỡng "quá to phải chia" | Ai cũng chọn M; không có quy tắc chia XL |
| **#NoEstimates (đếm item + throughput)** | Item đã được chuẩn hoá kích thước; có dữ liệu cycle time lịch sử; dòng chảy ổn định | Kỷ luật giữ item nhỏ tương đương; đo được cycle time theo phân vị | Item kích thước chênh 10 lần; dùng để tránh cam kết |
| **Ngày-người tuyệt đối** | Bắt buộc cho hợp đồng, bid, hoặc kế hoạch tài chính | Phải kèm khoảng và assumption list; phải trừ hệ số capacity thực | Bị đọc là lịch; bỏ chi phí phối hợp và thời gian chờ |
| **Timebox (không ước lượng)** | Bất định quá cao để ước lượng có nghĩa; hoặc giá trị chưa chắc | Tiêu chí kết thúc và quyết định sau timebox viết trước | Timebox bị gia hạn im lặng ba lần |

Quy tắc chọn thực dụng: nếu bạn đang tranh luận 3 điểm hay 5 điểm, đơn vị của bạn quá mịn cho mục đích
đang có. Nếu bạn không phân biệt được một việc 2 ngày với một việc 2 tuần, nó quá thô.

**Bước 5 — Chuyển ước lượng thành cam kết mà không hứa điều không kiểm soát được.** Đây là bước quyết
định uy tín dài hạn của bạn. Nguyên tắc: bạn chỉ cam kết những gì bạn kiểm soát, và bạn nói rõ những gì
bạn không kiểm soát cùng với ai kiểm soát nó.

```
CAM KẾT CÓ ĐIỀU KIỆN — Rút tiền về thẻ
Người cam kết: Minh (Tech Lead) · Ngày phát hành: 14/03 · Phiên bản 1

1. TÔI CAM KẾT (những thứ trong tầm kiểm soát của team)
   - 21/03: xong luồng rút tiền phía app, demo được trên staging với tài khoản test.
   - 28/03: xong đối soát và xử lý giao dịch lỗi, có test cho luồng thất bại.
   - Mỗi thứ Sáu 16:00: gửi cập nhật một trang, có nêu lệch so với kế hoạch nếu có.
   - Nếu tôi thấy khả năng trễ trên 1 tuần, tôi báo trong 24 giờ kể từ lúc thấy,
     không chờ đến khi chắc.

2. DỰ BÁO (không phải cam kết — đây là phân phối, không phải ngày)
   - P50: 7 tuần → tuần của 02/05.
   - P90: 11 tuần → tuần của 30/05.
   - Nguồn phương sai lớn nhất: tích hợp API ngân hàng (59% tổng phương sai).

3. ĐIỀU KIỆN — dự báo trên chỉ đúng nếu các điều dưới đây đúng
   | # | Điều kiện | Ai sở hữu | Cần xong trước | Nếu trượt thì |
   |---|-----------|-----------|----------------|---------------|
   | C1| Ngân hàng cấp sandbox có đủ 3 API rút tiền | Trang (PO) | 20/03 | +1 tuần cho mỗi tuần trượt |
   | C2| Quy tắc phí và hạn mức được nghiệp vụ chốt bằng văn bản | Trang | 18/03 | +0,5 tuần, và rủi ro làm lại |
   | C3| Không rút Khoa sang việc khác quá 20% thời gian | Hà (EM) | Suốt dự án | +1,5 tuần |
   | C4| Pháp chế xác nhận không cần thêm bước KYC | Trang | 25/03 | Phải ước lượng lại toàn bộ |

4. ĐIỂM KIỂM TRA (nơi con số được cập nhật công khai)
   - 21/03: sau khối 1 xong → thu hẹp khoảng, phát hành phiên bản 2 của tài liệu này.
   - 04/04: sau khi có sandbox thật → chốt hay không chốt được ngày cụ thể.
   - Quy tắc: mỗi điểm kiểm tra tôi đưa lại khoảng mới, kể cả khi khoảng không đổi.

5. ĐIỀU TÔI KHÔNG CAM KẾT (nói rõ để không ai tưởng)
   - Ngày ngân hàng cấp môi trường — không nằm trong tầm kiểm soát của team.
   - Thời gian pháp chế xem xét.
   - Việc scope không thay đổi.
```

Tiêu chí biết là xong: mỗi dòng ở mục 3 có tên người và ngày. Một điều kiện không có chủ là một điều
kiện sẽ trượt.

**Bước 6 — Xử lý câu "cho anh một con số thôi".** Trước khi trả lời, nhận diện: câu đó gần như không bao
giờ là yêu cầu về ước lượng. Nó là yêu cầu để giải một bài toán khác.

| Nhu cầu thật đằng sau | Dấu hiệu nhận biết | Cách đáp đúng |
|---|---|---|
| Cần một ngày để nói với bên thứ ba | Có tên đối tác/khách/nhà đầu tư trong câu | Đưa P90 làm ngày công bố, P50 làm ngày nội bộ, nói rõ hai cái khác nhau |
| Cần biết có nên bắt đầu không | "Có đáng làm không" xuất hiện | Trả lời bằng khoảng thô và chi phí spike, không bằng ngày |
| Cần cảm giác kiểm soát | Hỏi lại nhiều lần cùng một câu | Đổi cái được cấp: cho điểm kiểm tra hàng tuần thay vì cho số hẹp hơn |
| Đang chuẩn bị cắt scope và cần đòn bẩy | Con số bị đàm phán ngay khi vừa nói | Đưa bảng "cắt gì thì ngày đổi thế nào" (chủ đề 8) |
| Thực sự có deadline cứng bên ngoài | Nêu được lý do ngoài (mùa vụ, quy định, hợp đồng) | Đảo bài toán: cố định ngày, đàm phán scope |

Nếu bạn buộc phải cho đúng một con số duy nhất, quy tắc: **cho P90, và nói rõ đó là P90.** Lý do là bất
đối xứng thiệt hại — xong sớm hơn P90 gây ra một sự ngạc nhiên dễ chịu và không ai bị thiệt; xong muộn
hơn P50 gây ra hỏng kế hoạch của ba bộ phận khác. Nhưng phải nói ra, vì nếu không, bạn đang padding ngầm
và mất khả năng giải thích khi xong sớm.

### Trade-off

**Đầu tư vào ước lượng vs bắt đầu làm và học.** Đây là trade-off trung tâm, và nó thường bị hiểu sai
thành "ước lượng nhiều hay ít". Thực chất là: bạn mua thông tin bằng cách phân tích, hay bằng cách thực
thi?

| | Đầu tư vào ước lượng | Bắt đầu làm và học |
|---|---|---|
| Chi phí điển hình | 3 giờ × 7 người = 21 giờ-người | Spike 3 ngày × 1 người = 24 giờ-người |
| Loại bất định giảm được | Bất định về *hiểu chung* trong team | Bất định về *thực tế kỹ thuật* |
| Thứ không giảm được | Cái chưa ai biết là mình chưa biết | Phần cần đồng thuận về phạm vi |
| Sản phẩm phụ | Team hiểu giống nhau về scope | Code, hoặc một kết luận kiểm được |
| Rủi ro | Tự tin giả; con số bị coi là cam kết | Kết luận từ spike bị coi là kiến trúc cuối |
| Nghiêng về khi | Chi phí quyết định sai rất cao và không đảo được: ký fixed-price, đặt lịch chiến dịch quốc gia, cam kết với cơ quan quản lý | Quyết định đảo được rẻ; bất định là kỹ thuật; có thể giao lô nhỏ |

Điểm cân bằng thực dụng, và nó gần như luôn đúng: **đầu tư đủ để phát hiện bất đồng về phạm vi, rồi
chuyển phần còn lại của ngân sách bất định sang spike và lô nhỏ.** Cụ thể: 60–90 phút ước lượng khối để
lộ ra chỗ ba người hiểu khác nhau, sau đó tiêu 3–5 ngày spike cho khối có σ² lớn nhất. Tổng chi phí
tương đương một buổi ước lượng chi tiết, nhưng phần lớn ngân sách đi vào việc mua thông tin thật.

**Khoảng rộng trung thực vs con số hẹp dễ nghe.** Khoảng rộng đúng về mặt thông tin nhưng có ba chi phí
thật: nó bị đọc là thiếu năng lực ("sao anh không biết"); ở nhiều tổ chức, biên dưới của khoảng lập tức
trở thành kế hoạch, nghĩa là bạn nói P50–P90 và người ta nghe P50; và nó chuyển gánh nặng quyết định
sang stakeholder, điều mà không phải stakeholder nào cũng muốn nhận. Con số hẹp thì dễ hành động nhưng
sai với xác suất cao và làm xói mòn uy tín theo chu kỳ.

Nghiêng về **khoảng** khi: người nghe có năng lực đọc xác suất; bạn còn quan hệ dài hạn để xây uy tín;
quyết định của họ thật sự khác nhau giữa 7 tuần và 11 tuần. Nghiêng về **một con số (là P90, nói rõ)**
khi: bạn đã thử đưa khoảng hai lần và cả hai lần biên dưới bị lấy làm kế hoạch; hoặc người nghe cần một
ngày để nói với bên thứ ba và họ không được phép nói "khoảng".

**Ước lượng theo team vs theo cá nhân.** Ước lượng theo cá nhân chính xác hơn cho người đó (mỗi người
biết tốc độ của mình) nhưng tạo hai tác hại: nó gắn con số với danh tính, nên sai số trở thành vấn đề
uy tín cá nhân; và nó khoá việc cho người đó, làm giảm khả năng ai cũng nhận được việc. Ước lượng theo
team mất độ chính xác cá nhân nhưng giữ được tính thay thế được và giữ ước lượng ở dạng một thuộc tính
của công việc chứ không phải của người. Nghiêng về cá nhân khi công việc thực sự chỉ một người làm được
và bạn cần con số chính xác cho một cam kết cứng; nghiêng về team trong mọi trường hợp khác.

### Real-world Scenarios

**Tình huống A — Founder yêu cầu một ngày cụ thể cho việc còn nhiều bất định.**

Bối cảnh: product startup Việt, Series A, 28 engineer. Sơn (founder, kiêm vai stakeholder chính) đang
đàm phán với một chuỗi bán lẻ để tích hợp thanh toán. Bên đối tác hỏi "khi nào các bạn xong tính năng
rút tiền về thẻ". Sơn cần một ngày để trả lời email chiều nay. Minh (Tech Lead) có bảng P50/P90 ở trên:
P50 7 tuần, P90 11 tuần, và sandbox của ngân hàng chưa có.

*Phiên bản nói sai — Minh hứa:*

> **Sơn:** Minh, anh cần một ngày. Chiều nay anh phải trả lời bên đối tác. Bao giờ xong?
>
> **Minh:** Ờ... team em tính khoảng 7 tuần ạ.
>
> **Sơn:** 7 tuần là ngày 2/5. Anh chốt 2/5 nhé?
>
> **Minh:** Vâng, em nghĩ là được. Nếu mọi thứ suôn sẻ thì kịp.
>
> **Sơn:** Tốt. Anh ghi 2/5. Có gì thì em nói sớm nhé.
>
> **Minh:** Vâng ạ.

Kết quả có thể dự đoán: Sơn ghi 2/5 vào email, đối tác lên kế hoạch truyền thông theo ngày đó. Sandbox
ngân hàng có ngày 26/3 thay vì 20/3. Ngày 25/4, Minh vẫn nói "đang cố". Ngày 28/4, Minh báo cần thêm ba
tuần. Sơn mất uy tín với đối tác, và mất một phần niềm tin vào mọi con số Minh đưa sau này. Điều đáng
chú ý: Minh không nói dối ở bất kỳ câu nào. Anh ta nói "7 tuần" (đúng, đó là P50) và "nếu mọi thứ suôn
sẻ" (đúng, đó là điều kiện). Nhưng cả hai thông tin quan trọng nhất — độ rộng của khoảng và danh sách
điều kiện có chủ — không được truyền đi.

*Phiên bản nói đúng — khoảng, điều kiện, điểm kiểm tra:*

> **Sơn:** Minh, anh cần một ngày. Chiều nay anh phải trả lời bên đối tác. Bao giờ xong?
>
> **Minh:** Em trả lời được, nhưng em cần biết anh dùng ngày đó để làm gì — vì nếu là ngày đưa vào
> email cho đối tác thì em đưa anh con số khác với ngày em dùng để lập kế hoạch nội bộ.
>
> **Sơn:** Để đưa vào email. Họ sẽ lên kế hoạch truyền thông theo ngày đó.
>
> **Minh:** Vậy anh dùng **30/5**. Đây là lý do: trung vị của bọn em là 2/5, nghĩa là năm mươi-năm mươi.
> Không ai nên đưa một con số năm mươi-năm mươi vào email cho đối tác. 30/5 là mức mà 9 trên 10 khả năng
> bọn em xong trước.
>
> **Sơn:** Bốn tuần chênh nhau là nhiều đấy. Sao khoảng lại rộng thế?
>
> **Minh:** Vì gần 60% độ bất định nằm ở đúng một chỗ: tích hợp API ngân hàng, mà bọn em chưa có sandbox
> để thử. Bốn lần trước ta tích hợp ngân hàng, thực tế mất 4, 6, 7 và 9 tuần — lần 9 tuần là vì chờ môi
> trường ba tuần. Em không có cơ sở để nói lần này khác.
>
> **Sơn:** Anh không thích 30/5. Nó làm ta chậm hơn đối thủ.
>
> **Minh:** Em hiểu. Có ba cách thu hẹp, anh chọn:
> Một — em có sandbox trước 20/3 thì ngày 04/04 em đưa lại khoảng, khả năng cao hẹp hơn một nửa. Việc
> này cần anh hoặc Trang đẩy phía ngân hàng, em không có kênh.
> Hai — cắt scope phiên bản đầu: bỏ màn hình quản trị cho CS và chỉ hỗ trợ một ngân hàng, khoảng lùi về
> P90 khoảng 8 tuần. Nhưng CS sẽ phải xử lý tay trong 6 tuần đầu.
> Ba — anh cho em thêm một người có kinh nghiệm đối soát. Cái này em phải nói thật là chỉ giúp được nếu
> vào trước tuần sau; vào sau đó thì gần như không giúp.
>
> **Sơn:** Anh sẽ gọi ngân hàng. Cứ ghi 30/5 với đối tác, nhưng nội bộ ta nhắm đầu tháng 5.
>
> **Minh:** Đúng như vậy. Em gửi anh một trang có bốn điều kiện, ai sở hữu, và ngày cần xong — trong đó
> có điều kiện sandbox 20/3 thuộc anh. Và em cam kết một việc riêng: nếu em thấy khả năng trễ trên một
> tuần, em báo trong 24 giờ, không chờ đến lúc chắc. Điểm kiểm tra là 21/3 và 4/4.

Bảng mổ xẻ:

| # | Điều Minh làm ở phiên bản đúng | Cơ chế nó bảo vệ | Điều gì hỏng nếu thiếu |
|---|---|---|---|
| 1 | Hỏi ngày này dùng để làm gì | Phân biệt con số cho quyết định nội bộ và con số công bố | Một con số P50 đi vào email đối tác |
| 2 | Đưa P90 làm ngày công bố, nói rõ là P90 | Bất đối xứng thiệt hại: trễ đắt hơn sớm | Kế hoạch của ba bên gắn vào một con số 50% |
| 3 | Chỉ ra nguồn phương sai tập trung (59% ở một khối) | Chuyển thảo luận từ "con số" sang "rủi ro cụ thể" | Sơn chỉ đàm phán con số, không ai gỡ rủi ro |
| 4 | Dẫn lớp tham chiếu (4, 6, 7, 9 tuần) | Outside view chống planning fallacy và chống áp lực xã hội | Con số bị đàm phán bằng ý chí, không bằng dữ liệu |
| 5 | Đưa ba phương án có trade-off, để Sơn chọn | Quyết định về trade-off kinh doanh thuộc người có accountability | Minh tự quyết cắt scope, hoặc tự gánh bằng crunch |
| 6 | Nói rõ "thêm người chỉ giúp nếu vào trước tuần sau" | Chặn Brooks's Law trước khi nó xảy ra (chủ đề 7) | Tuần thứ 8 có ba người mới vào và dự án chậm hơn |
| 7 | Giao điều kiện sandbox cho Sơn với một ngày | Chuyển phụ thuộc không kiểm soát được thành việc có chủ (chủ đề 6) | "Chờ ngân hàng" nằm trong kế hoạch mà không ai đẩy |
| 8 | Cam kết cơ chế báo lệch trong 24 giờ | No-surprise: đổi cam kết về ngày thành cam kết về minh bạch | Ngày 28/4 mới biết, mất khả năng ứng phó |

Điều Minh *không* làm cũng quan trọng: anh không từ chối cho số, không nói "Agile thì không ước lượng",
không giảng về Cone of Uncertainty. Anh đưa một con số dùng được ngay chiều nay, kèm cấu trúc để con số
đó được cập nhật.

**Tình huống B — Velocity của team A bị áp cho team B.**

Bối cảnh: một công ty logistics scale từ một team lên ba team trong 8 tháng. Team A (làm 3 năm, hệ thống
đơn hàng) có velocity ổn định 45 điểm/sprint. Team C mới lập, 5 người, nhận hệ thống định tuyến legacy.
Sprint đầu team C làm được 18 điểm. Một VP nhìn bảng tổng hợp và hỏi trong họp: "Sao team C chỉ bằng 40%
team A?"

Cơ chế hỏng: story point là đơn vị *do team tự định nghĩa*, hiệu chỉnh theo lịch sử của chính team đó.
Một điểm của team A và một điểm của team C không cùng đơn vị, cũng như hai người dùng hai thước khác
nhau đo hai căn phòng. So sánh chúng không sai một chút — nó là phép so sánh không có nghĩa. Nhưng tác
hại thì rất thật, và nó xuất hiện nhanh: team C có đúng một cách để "cải thiện" là lạm phát điểm. Ba
sprint sau, velocity team C là 44. Không có gì trong dòng chảy thay đổi. Cái duy nhất thay đổi là đơn vị
đo, và giờ tổ chức mất luôn công cụ dự báo của cả hai team.

Điều nên làm, theo thứ tự: (1) Với VP, không tranh luận về story point — đưa thay thế bằng đơn vị so
sánh được: cycle time trung vị, số item lên production, số incident. Ba số này cùng đơn vị giữa các
team. (2) Đổi cách trình bày báo cáo: velocity chỉ hiện trong ngữ cảnh của chính team đó theo thời gian
(xu hướng), không bao giờ đặt cạnh nhau trên cùng một biểu đồ. (3) Nói thẳng nguyên nhân thật của 18 vs
45: team C làm hệ thống legacy không có test, bus factor 1, và 40% thời gian sprint đầu đi vào đọc code.
Đó là thông tin hữu ích cho VP; "40% năng suất" thì không.

**Tình huống C — Buộc phải cho một con số cho hợp đồng fixed-price.**

Bối cảnh: ODC, khách Singapore, RFP yêu cầu báo giá cố định cho một hệ thống quản lý bảo hành, spec 30
trang, deadline nộp 5 ngày. Không có lựa chọn "cho khoảng".

Cách làm đúng ở đây không phải padding ngầm mà là **chuyển bất định thành điều khoản**. Cụ thể: ước
lượng khối P50/P90 như bước 2; báo giá dựa trên P90; và quan trọng nhất, đưa vào hợp đồng một
assumption list 12–20 dòng, mỗi dòng là một giả định kiểm được ("khách cung cấp môi trường test trong
10 ngày làm việc kể từ kick-off"; "tối đa 2 vòng review cho mỗi màn hình"; "dữ liệu di trú ở định dạng
X, tối đa 3 nguồn"). Mỗi dòng bị vi phạm là một change request có công thức tính.

Lý do đây là cách đúng: padding ngầm 40% làm bạn thua bid (đối thủ không pad, hoặc pad ít hơn), và nếu
thắng thì bạn vẫn không có cơ sở đàm phán khi scope trượt. Assumption list làm giá của bạn cạnh tranh
hơn *và* cho bạn một cơ chế bảo vệ. Nó cũng là tín hiệu chuyên nghiệp: khách hàng Nhật và Singapore
thường đọc assumption list như bằng chứng bên bán đã hiểu bài toán.

### Best Practices

**Luôn phát hành ước lượng dưới dạng hai số, và gọi tên hai số đó.** Lý do: một số duy nhất bị đọc theo
nghĩa có lợi cho người đọc, và người đọc luôn là người có nhiều quyền hơn bạn. Hai số buộc cuộc trò
chuyện phải xảy ra ở cấp độ xác suất, nơi nó thuộc về.

**Làm outside view trước inside view.** Lý do: khi bạn đã dựng kịch bản trong đầu, mọi số liệu quá khứ
sẽ bị điều chỉnh về phía kịch bản đó (anchoring). Thứ tự ngược lại thì lớp tham chiếu làm neo, và inside
view chỉ được dùng để điều chỉnh có lý do nêu tên được.

**Giữ một sổ lớp tham chiếu, cập nhật sau mỗi dự án, mất 10 phút.** Nội dung tối thiểu: tên việc, lớp,
ước lượng ban đầu, thực tế, ba nguyên nhân chênh lớn nhất. Lý do: đây là tài sản có lãi kép cao nhất
trong toàn bộ chủ đề. Sau 18 tháng bạn có một công cụ mà không ai trong công ty có thể tranh luận bằng ý
chí, và nó cũng là cách chuyển tri thức khi người rời team.

**Ghi lại ước lượng ban đầu ở nơi không sửa được, kèm ngày và giả định.** Lý do: không có bản gốc thì
không có học. Hiện tượng phổ biến: sau khi dự án trễ, mọi người nhớ rằng "hồi đó ta cũng biết là khó" —
đó là hindsight bias, và nó xoá sạch bài học. Một comment có ngày trên ticket là đủ.

**Ước lượng khối thay vì ước lượng story ở giai đoạn sớm.** Lý do: chia nhỏ ở giai đoạn chưa hiểu tạo
độ chi tiết giả. Bạn tiêu ba giờ và ra một con số có hai chữ số thập phân cho một thứ mà biên độ thật là
0,25x–4x.

**Dùng bất đồng trong ước lượng như tín hiệu chẩn đoán, không lấy trung bình để dập nó.** Lý do: hai
người lệch nhau bốn lần gần như luôn có nghĩa họ đang ước lượng hai công việc khác nhau. Lấy trung bình
là xoá mất phát hiện quý nhất của buổi họp.

**Cập nhật ước lượng công khai ở điểm kiểm tra định trước, kể cả khi không đổi.** Lý do: nếu bạn chỉ cập
nhật khi có tin xấu, việc cập nhật trở thành tín hiệu tin xấu, và bạn sẽ có động lực trì hoãn nó. Cập
nhật định kỳ làm hành vi báo cáo trung tính về mặt cảm xúc.

**Tách bạch "ước lượng" và "cam kết" bằng từ vựng khác nhau, và sửa người dùng sai từ.** Lý do: hai khái
niệm này có chủ thể trách nhiệm khác nhau. Trong thực tế, việc kiên trì nói "đây là dự báo, không phải
cam kết" trong 5–6 lần họp làm đổi cả từ vựng của tổ chức. Đây là một trong ít can thiệp văn hoá mà một
Tech Lead làm được không cần quyền.

### Anti-patterns

**Ước lượng bị đàm phán xuống rồi được coi như cam kết.** Hình dạng: team nói 8 tuần, PO nói "6 tuần
được không, cố lên", team im lặng hoặc nói "bọn em thử", và 6 tuần trở thành ngày trong kế hoạch. Cơ
chế: ước lượng là một phát biểu về thực tế; nó không phản ứng với đàm phán, cũng như trọng lượng của
một chiếc xe không giảm khi bạn thương lượng. Nhưng vì con số được phát ngôn bởi con người, nó bị đối
xử như một đề nghị. Kết quả là tổ chức có một kế hoạch dựa trên một con số mà không ai tin, và mọi người
đều biết là không ai tin — nhưng không ai nói, vì nói ra là mở lại một cuộc đàm phán đã "xong". Dấu hiệu
sớm: trong biên bản họp, con số cuối khác con số team đưa mà không có thay đổi nào về scope hay nguồn
lực đi kèm. Cách chặn: quy tắc phát ngôn — *"Ước lượng đổi khi scope đổi hoặc thông tin đổi. Nếu anh
muốn 6 tuần, ta hãy bàn cắt gì."* Nói câu này một lần, bình tĩnh, và không thoả hiệp ở lần đầu, sẽ tiết
kiệm cho bạn nhiều quý.

**Dùng velocity của team này áp cho team khác.** Cơ chế: đã phân tích ở tình huống B — story point là
đơn vị nội bộ, và việc so sánh tạo áp lực lạm phát điểm. Hậu quả sâu hơn: sau khi lạm phát xảy ra, tổ
chức mất khả năng dự báo ở *cả hai* team, và mất luôn năng lực xây lại vì đơn vị đo đã mất tính tin cậy.
Dấu hiệu sớm: có một slide đặt velocity của nhiều team cạnh nhau; hoặc một quản lý dùng từ "năng suất"
khi nói về story point; hoặc velocity của một team tăng đều 15% mỗi quý mà cycle time không giảm.

**Padding ngầm thay vì nói rõ rủi ro.** Hình dạng: team ước 6 tuần, tự cộng thêm 3 tuần "cho chắc", báo
9 tuần, không nói phần 3 tuần là gì. Cơ chế phá hệ thống ở ba tầng: (1) đệm ẩn không thể phòng thủ —
khi bị ép, bạn không có lập luận nào để bảo vệ 9 tuần, nên nó bị cắt về 7 và bạn mất hết đệm; (2) đệm ẩn
bị tiêu bởi student syndrome vì không ai biết nó tồn tại nên không ai theo dõi mức tiêu; (3) nó dạy tổ
chức rằng con số của bạn có nước, nên tổ chức sẽ cắt theo phản xạ mãi mãi, kể cả khi bạn nói thật. Dấu
hiệu sớm: khi bị hỏi "3 tuần này là cho cái gì", câu trả lời là "cho an toàn" thay vì tên một rủi ro cụ
thể. Thay thế: đưa P50 trần trụi, cộng một dòng buffer có tên, và một bảng điều kiện có chủ — cùng
lượng thời gian, nhưng phòng thủ được và theo dõi được.

**Người không làm việc đó lại là người ước lượng.** Hình dạng: Tech Lead hoặc PM ước lượng cho cả team,
hoặc một senior đã làm hệ thống này 3 năm ước lượng cho một người mới nhận. Cơ chế: ước lượng phản ánh
mức độ hiểu của người ước lượng, không phản ánh thời gian người thực thi cần. Người đã biết hệ thống bỏ
qua toàn bộ chi phí học. Dấu hiệu sớm: người nhận việc không nhớ con số ước lượng của việc mình đang
làm; hoặc câu "ai ước lượng cái này?" không có câu trả lời rõ.

**Quy đổi story point thành giờ.** Hình dạng: "1 điểm = 4 giờ", rồi bảng tính capacity theo giờ. Cơ chế:
story point tồn tại để tránh đúng phép quy đổi này, vì giá trị của nó là tính *tương đối* — nó hiệu chỉnh
tự động theo năng lực team qua velocity. Khi quy đổi thành giờ, bạn mất tính tự hiệu chỉnh và có thêm một
lớp trung gian vô nghĩa; đơn giản hơn là ước lượng bằng giờ luôn. Nhưng hậu quả nặng hơn là về hành vi:
điểm quy ra giờ thì đo được theo người, và ước lượng lập tức trở thành công cụ đánh giá cá nhân. Dấu
hiệu sớm: có một hằng số quy đổi được viết ở đâu đó; báo cáo capacity tính bằng giờ-người trên story
point.

**Đo "độ chính xác ước lượng" của cá nhân và đưa vào Performance Review.** Cơ chế: bạn đang tối ưu một
đại lượng mà người bị đo kiểm soát trực tiếp bằng cách nói số cao hơn. Kết quả tất yếu: mọi ước lượng
trở thành P95 trong vòng hai quý, và tổ chức mất thông tin về thời gian thực. Đây là trường hợp kinh
điển của Goodhart's law. Dấu hiệu sớm: một dòng "estimation accuracy" xuất hiện trong template review;
hoặc con số ước lượng của một người tăng đều mà kích thước việc không tăng.

**Ước lượng lại toàn bộ backlog mỗi quý.** Cơ chế: phần lớn backlog sẽ không bao giờ được làm — ở một
backlog lành mạnh, tỉ lệ item bị bỏ hoặc thay đổi hoàn toàn trước khi tới lượt thường trên 40%. Ước
lượng chúng là làm việc trên hàng tồn kho. Dấu hiệu sớm: có một buổi "grooming backlog" định kỳ dài
hơn 2 giờ; có item được ước lượng lại lần thứ ba mà chưa từng được làm.

### Khi nào KHÔNG nên áp dụng

**Khi việc nhỏ hơn ngưỡng mà chi phí ước lượng vượt lợi ích.** Với item dưới khoảng 2 ngày, phương sai
tuyệt đối nhỏ đến mức không ảnh hưởng quyết định nào, còn chi phí họp thì cố định. Ngưỡng thực dụng:
nếu 80% item của bạn dưới 3 ngày và cycle time trung vị dưới 5 ngày, hãy bỏ hẳn ước lượng từng item và
dự báo bằng throughput (chủ đề 3). Điều vẫn phải giữ: quy tắc "item nào ước trên 5 ngày thì phải chia
hoặc phải làm spike" — tức bạn vẫn cần một bộ lọc kích thước, chỉ không cần con số.

**Khi quyết định không phụ thuộc vào con số.** Nếu việc này đã được quyết làm dù mất 6 hay 12 tuần —
vì nó là yêu cầu compliance, hoặc vì nó là toàn bộ chiến lược của quý — thì ước lượng chỉ dùng để xếp
hàng, và một t-shirt size là đủ. Kiểm tra: hỏi "nếu con số là gấp đôi, anh có làm khác không?". Nếu
câu trả lời là không, đừng đầu tư vào con số; đầu tư vào việc chia nhỏ và giao sớm.

**Khi tổ chức đang dùng ước lượng làm công cụ đánh giá cá nhân.** Trong điều kiện này, mọi kỹ thuật ở
trên sẽ tạo ra số liệu giả có vẻ tinh vi hơn — P50/P90 sẽ trở thành P90/P99, và bảng điều kiện sẽ trở
thành danh sách miễn trừ trách nhiệm. Thứ tự đúng: sửa hệ đánh giá trước (chương 08), hoặc nếu chưa sửa
được, đừng nâng cấp công cụ ước lượng — chuyển sang timebox và giao lô nhỏ, vì hai thứ đó không cần con
số để hoạt động.

**Khi bất định cao đến mức khoảng ước lượng vượt 3 lần.** Nếu P90/P50 lớn hơn 3, con số không dùng được
cho bất kỳ quyết định nào; nó chỉ tạo ảo giác đã lập kế hoạch. Ở vùng này, công cụ đúng là timebox có
tiêu chí kết thúc: "5 ngày, mục tiêu trả lời được ba câu hỏi X, Y, Z; hết 5 ngày ta họp 30 phút và
quyết định tiếp tục, đổi cách, hay bỏ". Bạn đang mua thông tin với giá cố định thay vì mua một con số
không dùng được. Sai lầm phổ biến ở đây là cố ước lượng chính xác hơn — nó không khả thi, vì bất định
không nằm ở phép tính.

**Khi việc ước lượng đang thay thế cho việc đối thoại về ưu tiên.** Có một dạng lạm dụng khó thấy: tổ
chức không dám quyết định thứ tự ưu tiên, nên yêu cầu ước lượng thật chi tiết với hy vọng con số sẽ tự
quyết định thay. Dấu hiệu: mỗi lần họp ưu tiên đều kết thúc bằng "để tôi hỏi team ước lượng thêm đã".
Ở đây, thêm ước lượng không giúp gì, vì bài toán không phải thiếu thông tin mà là thiếu người dám nói
"cái này không làm quý này" (chủ đề 5). Ước lượng thêm chỉ trì hoãn quyết định và tiêu capacity thật.

---

## 5. Roadmap và Planning

### Problem Statement

Một fintech Việt, 90 engineer, 6 team. Roadmap 2025 là một file spreadsheet: 47 dòng feature, mỗi dòng
có cột "Target month", màu xanh cho Q1, vàng Q2, cam Q3, đỏ Q4. File này được trình bày trong buổi họp
toàn công ty tháng 1, được in ra dán ở tường, và được bộ phận Sales dùng làm tài liệu bán hàng.

Đến cuối tháng 4, đo lại (số minh hoạ, nhưng là hình dạng rất phổ biến):

| Chỉ số sau 4 tháng | Giá trị |
|---|---|
| Số dòng feature đã giao đúng tháng mục tiêu | 6/47 |
| Số dòng đã bị đẩy sang quý sau ít nhất một lần | 29/47 |
| Số dòng bị xoá khỏi roadmap mà không ai thông báo | 8/47 |
| Số dòng được thêm vào sau tháng 1 (không có trong bản gốc) | 14 |
| Số giờ-người tiêu cho việc cập nhật lại file này | ~120 giờ trong 4 tháng |
| Số lần Sales đã bán một feature ở cột Q3 như thể sắp có | ít nhất 3 |

Ba hậu quả quan sát được, không phải cảm giác. Một: stakeholder ngừng đọc roadmap. Khi được hỏi "quý
sau team Payment làm gì", Giám đốc Kinh doanh không mở file mà nhắn trực tiếp cho EM — nghĩa là công cụ
phối hợp đã chết và chi phí phối hợp quay về dạng đắt nhất là hỏi từng người. Hai: mỗi lần đổi roadmap
tốn khoảng ba tuần lead time (họp lại, chỉnh file, trình lại, thông báo lại), nên team tránh đổi, nên
họ tiếp tục làm những việc đã hết giá trị. Ba: không ai biết cái gì đang *không* được làm. Danh sách 47
dòng không hề nói rằng việc thay thế hệ thống hạn mức tín dụng — thứ đang gây 40% incident P2 — không
nằm trong kế hoạch năm nay. Nó chỉ đơn giản là không có trong file, và "không có trong file" không phải
là một quyết định được ai ký.

Điểm cần nhìn thẳng: cái file đó không hỏng vì thiếu công cụ, thiếu Jira plugin hay thiếu kỷ luật cập
nhật. Nó hỏng vì nó cố làm một việc bất khả thi — tuyên bố ngày cho những thứ mà thông tin để xác định
ngày còn chưa tồn tại. Và vì nó hỏng theo cách âm thầm, tổ chức không sửa nó; tổ chức chỉ ngừng dùng nó
rồi vẫn tiếp tục sản xuất nó mỗi năm.

Không có Roadmap hoạt động được thì hậu quả cụ thể là: engineer không biết việc mình làm phục vụ mục
tiêu nào nên không tự ra được quyết định nhỏ (mỗi quyết định phải hỏi lên); Sales bán cái không tồn tại;
tuyển dụng lệch pha với nhu cầu 6 tháng tới; và mọi ưu tiên trở thành hàm của việc ai nói to nhất trong
tuần này.

### First Principles

**Nguyên lý 1 — Roadmap là công cụ truyền đạt ý định và trade-off, không phải hợp đồng thời gian.**
Chức năng kinh tế của nó là giảm chi phí phối hợp. Trong một tổ chức nhiều bộ phận, mỗi bộ phận cần biết
ý định của bộ phận khác để tự lập kế hoạch của mình: Marketing cần biết đại khái khi nào có gì để chuẩn
bị chiến dịch, Tuyển dụng cần biết hướng đi để mở đúng vị trí, Support cần biết để viết tài liệu, đối tác
cần biết để xếp lịch tích hợp. Nếu không có tài liệu ý định chung, mỗi cặp bộ phận phải trao đổi trực
tiếp, và chi phí đó tăng theo số cặp — n(n−1)/2. Roadmap là một điểm phát sóng thay cho n(n−1)/2 kênh
song phương. Đó là toàn bộ giá trị của nó. Khi bị dùng làm hợp đồng thời gian, nó không thực hiện tốt
hơn chức năng gốc; nó chỉ thêm một hệ quả là người phát sóng bị trừng phạt khi thực tế khác dự báo, nên
người phát sóng sẽ giảm lượng thông tin phát ra. Bạn có thể quan sát điều này rất rõ: các tổ chức coi
roadmap là hợp đồng dần có roadmap mơ hồ hơn, không phải chính xác hơn.

**Nguyên lý 2 — Độ chi tiết của kế hoạch tỉ lệ nghịch với khoảng cách thời gian.** Đây không phải một
lời khuyên về phong cách; nó là hệ quả của việc thông tin về tương lai được sinh ra dần theo thời gian.
Ở tháng 1, thông tin cần để lập kế hoạch chi tiết cho tháng 10 chưa tồn tại — nó sẽ được sinh ra bởi
chính những gì bạn học từ tháng 2 đến tháng 9. Viết chi tiết cho tháng 10 vào tháng 1 không phải là lập
kế hoạch sớm; nó là bịa thông tin. Hệ quả kép rất ít người tính: chi tiết bịa ra có chi phí duy trì.
Mỗi item được viết chi tiết là một item phải được cập nhật, giải thích, và bảo vệ khi thay đổi. Đây
chính là tồn kho trong lập kế hoạch — planning inventory — và nó tuân theo cùng logic tồn kho ở chủ đề 3:
tồn kho lớn thì chi phí thay đổi cao, nên hệ thống trở nên cứng đúng lúc nó cần mềm nhất.

| Khoảng cách | Thông tin đã có | Đơn vị nên dùng | Sai lầm điển hình |
|---|---|---|---|
| 0–6 tuần | Đã đủ để biết việc gì, ai làm, cần gì | Item có tiêu chí xong, có người, có ngày kiểm tra | Chưa chốt được nhưng vẫn khởi động |
| 6 tuần–1 quý | Biết vấn đề, chưa biết giải pháp cuối | Outcome + khoảng, mức cam kết "đang chuẩn bị" | Cam kết ngày như tầng Now |
| 1–3 quý | Biết chủ đề và giả định kinh doanh | Chủ đề (theme) + câu hỏi cần trả lời | Ghi tên feature kèm tháng |
| >3 quý | Chỉ biết hướng | Định hướng, không có item | Có Gantt |

**Nguyên lý 3 — Một kế hoạch không nói cái gì sẽ không làm thì không chứa thông tin.** Điều này là logic
thuần: một tuyên bố không loại trừ khả năng nào thì không thể sai, và một tuyên bố không thể sai thì
không giúp ai ra quyết định. Danh sách 47 dòng ở trên không loại trừ gì cả — nó tương thích với mọi thực
tế có thể xảy ra, vì bất cứ điều gì team làm cũng có thể được coi là một trong 47 dòng đó, hoặc là "việc
phát sinh". Nội dung thật của một roadmap nằm ở cột "Later" và ở danh sách "không làm quý này", vì đó là
những chỗ có người bị từ chối. Nếu không ai bị từ chối, không có ưu tiên nào được thực hiện.

**Nguyên lý 4 — Dự báo và cam kết là hai loại phát ngôn khác nhau, và trộn chúng là nguồn hỏng lớn nhất.**
Dự báo là một phân phối (chủ đề 4). Cam kết là một ràng buộc có hệ quả khi vi phạm. Một Gantt chart vẽ
mọi thứ bằng cùng một loại thanh màu, nên người đọc không có cách nào phân biệt "chúng tôi nghĩ khoảng
tháng 8" với "chúng tôi đã hứa ngày 15 tháng 8 với cơ quan quản lý". Khi hai loại này không phân biệt
được, hai chuyện xảy ra cùng lúc: dự báo bị xử như cam kết (nên trượt dự báo bị coi là thất hứa), và cam
kết bị xử như dự báo (nên cái thật sự phải giao đúng ngày lại bị đẩy như mọi cái khác).

### Mental Model

**Cone of Uncertainty → ba mức cam kết.** Hình dung một hình nón mở rộng theo trục thời gian: điểm hôm
nay hẹp, càng xa càng rộng. Sai lầm phổ biến là cố vẽ một đường mảnh chạy giữa hình nón và gọi đó là kế
hoạch. Cách dùng đúng: chia hình nón thành ba vùng theo độ rộng, và gán cho mỗi vùng một *loại phát ngôn*
khác nhau — Now là cam kết, Next là ý định có điều kiện, Later là giả thuyết. Mô hình này map trực tiếp
vào cấu trúc Now/Next/Later ở phần Framework, và giá trị của nó là buộc bạn dùng từ ngữ khác nhau cho
ba vùng, chứ không chỉ màu khác nhau.

**Real Options — Later là quyền chọn, không phải nghĩa vụ.** Trong tài chính, một quyền chọn có giá trị
vì nó cho bạn quyền hành động khi thông tin đã rõ hơn mà không bắt buộc phải hành động. Mỗi item ở tầng
Later là một quyền chọn: bạn giữ quyền làm nó, và bạn chưa trả phí. Cam kết sớm là hành động thực thi
quyền chọn trước hạn — bạn trả toàn bộ phí (thiết kế chi tiết, hứa với người khác, phân bổ người) để
nhận về một thứ mà giá trị của nó chưa xác định. Hệ quả thực hành: giá trị của tầng Later nằm ở chỗ nó
*được ghi ra mà không được cam kết*. Một tổ chức không phân biệt được hai việc đó sẽ hoặc không dám ghi
gì vào Later (mất khả năng phối hợp dài hạn), hoặc coi mọi thứ ghi vào Later là đã hứa (mất optionality).

**Portfolio Allocation — capacity là một danh mục đầu tư có tỉ lệ phân bổ.** Một quỹ đầu tư không quyết
định từng khoản theo cảm tính từng tuần; nó đặt tỉ lệ phân bổ theo lớp tài sản trước, rồi chọn khoản
trong từng lớp. Lý do là để chống lại một thiên lệch có thật: khoản nào đang gây cảm xúc mạnh nhất sẽ
hút hết vốn. Trong delivery, thiên lệch tương ứng là việc gấp lấn việc quan trọng — bucket technical
health không bao giờ thắng trong một cuộc so sánh trực tiếp với một feature mà founder đang muốn, vì
lợi ích của nó ở tương lai và phân tán, còn chi phí của việc trì hoãn nó không xuất hiện trong bất kỳ
báo cáo nào của tuần này. Cách chống lại không phải là tranh luận giỏi hơn mỗi tuần; đó là cuộc chiến
bạn sẽ thua dài hạn. Cách chống lại là đặt tỉ lệ phân bổ trước ở cấp quý, và chuyển cuộc tranh luận từ
"cái này hay cái kia" thành "đổi tỉ lệ phân bổ hay không" — một cuộc tranh luận diễn ra bốn lần một năm
với người có thẩm quyền, thay vì mỗi ngày với người không có.

### Practical Framework

**Bước 1 — Đặt từ vựng cam kết ba mức và dùng đúng từ đó ở mọi nơi.** Đây là bước rẻ nhất và bị bỏ nhiều
nhất. Không có từ vựng chung, mọi cấu trúc bên dưới đều vô hiệu vì người đọc tự gán mức cam kết theo
kinh nghiệm riêng của họ.

| Tầng | Khoảng thời gian | Mức cam kết | Câu phải nói kèm | Điều gì làm nó thay đổi |
|---|---|---|---|---|
| **Now** | Quý hiện tại, đang làm | Cam kết có điều kiện | "Chúng tôi cam kết giao X, với các điều kiện C1–C3, cập nhật mỗi hai tuần" | Chỉ thay đổi qua một quyết định có ghi lại, có người ký |
| **Next** | Quý kế tiếp | Ý định — đã xác nhận vấn đề, chưa cam kết ngày | "Chúng tôi dự định làm X quý sau. Chưa có ngày. Sẽ chốt vào buổi planning ngày dd/mm" | Thay đổi được ở quarterly planning, không cần leo cấp |
| **Later** | 2–4 quý sau | Giả thuyết — có thể bị bỏ hoàn toàn | "X đang nằm trong danh sách cân nhắc. Xác suất làm trong 12 tháng khoảng 50%. Đừng bán nó" | Thay đổi tự do, không cần thông báo ai |

Quy tắc vận hành kèm theo: **cấm ghi tháng ở tầng Next và Later.** Nếu một item ở Later có tháng, nó sẽ
được đọc là cam kết bất kể bạn viết chú thích gì bên cạnh. Thay tháng bằng thứ tự và bằng điều kiện mở
khoá ("sẽ vào Next nếu thử nghiệm A cho kết quả dương").

**Bước 2 — Viết roadmap theo cấu trúc dưới đây.** Đây là template dùng được nguyên; số là số minh hoạ.

```
ROADMAP — Nhóm Payments · Bản phát hành 05/07 · Người sở hữu: Trang (PO) + Hà (EM)
Chu kỳ cập nhật: mỗi 2 tuần (Now), mỗi quý (Next/Later). Bản trước: 21/06.

MỤC TIÊU KINH DOANH QUÝ NÀY (trích từ OKR công ty)
  O: Giảm tỉ lệ rơi ở bước thanh toán để tăng doanh thu trên mỗi phiên.
  KR1: tỉ lệ hoàn tất thanh toán 71% → 80% (đo trên toàn bộ đơn, cuối quý).
  KR2: p95 thời gian phản hồi cổng thanh toán 4,2s → 2,0s.

=== NOW (Q3, đang làm — CAM KẾT CÓ ĐIỀU KIỆN) ===
N1. Thanh toán một chạm cho khách đã lưu thẻ
    Kết quả mong đợi: +4 điểm tỉ lệ hoàn tất (giả thuyết, đo bằng A/B).
    Đặt cược vào: KR1
    Cam kết: bật A/B cho 10% traffic trước 15/08; quyết định mở rộng 29/08.
    Điều kiện: pháp chế xác nhận lưu token thẻ (chủ: Trang, hạn 20/07).
    Trạng thái: đang làm · dự báo P50 08/08 · P90 22/08
N2. Chuyển cổng thanh toán sang kết nối trực tiếp (giảm 1 hop)
    Đặt cược vào: KR2 · Cam kết: xong 30/09 · Điều kiện: đối tác cấp SLA test 01/08
    Trạng thái: chờ hợp đồng đối tác — RỦI RO ĐANG MỞ (xem risk register R-14)

=== NEXT (Q4 — Ý ĐỊNH, CHƯA CÓ NGÀY) ===
X1. Thanh toán trả sau cho nhóm khách hàng thân thiết
    Vấn đề đã xác nhận: 18% giỏ hàng bị bỏ ở bước chọn phương thức (số minh hoạ).
    Chưa biết: mô hình rủi ro tín dụng, yêu cầu pháp lý.
    Sẽ chốt phạm vi và ngày tại: quarterly planning 12/09.
X2. Đối soát tự động với 3 đối tác lớn nhất
    Lý do vào Next: chi phí vận hành thủ công 1,5 người/tháng.

=== LATER (2–4 quý — GIẢ THUYẾT, ĐỪNG BÁN) ===
L1. Ví nội bộ (chờ kết quả nghiên cứu người dùng tháng 10)
L2. Thanh toán xuyên biên giới (mở khoá nếu công ty mở thị trường thứ hai)
L3. Thay thế hệ thống hạn mức (mở khoá nếu tỉ lệ incident P2 do hạn mức > 25% hai quý liền)

=== KHÔNG LÀM TRONG 12 THÁNG TỚI (và vì sao) ===
- Hỗ trợ tiền điện tử: chưa có khung pháp lý rõ; chi phí compliance vượt doanh thu dự kiến.
- Tự xây cổng thanh toán: không phải năng lực cạnh tranh; mua rẻ hơn xây (quyết định ADR-88).
- Redesign toàn bộ trang checkout: đang có A/B chạy; đổi giao diện lớn sẽ làm nhiễu số đo của KR1.
  Xem lại sau khi KR1 chốt.
```

Mục cuối cùng là mục quan trọng nhất và là mục hay bị bỏ. Nó biến roadmap từ một danh sách mong muốn
thành một tài liệu có thể sai — và vì thế có thể dùng được.

**Bước 3 — Quarterly planning thực dụng.** Mục tiêu của buổi này không phải lập lịch 13 tuần. Mục tiêu
là: (a) chốt tỉ lệ phân bổ capacity, (b) chọn 3–5 việc lớn nhất cho tầng Now, (c) nói ra danh sách không
làm, (d) tìm và ghi phụ thuộc liên team (chủ đề 6). Quy trình hai tuần, tổng chi phí họp khoảng 6–8 giờ
cho mỗi team — nếu vượt 12 giờ, bạn đang lập kế hoạch quá chi tiết.

| Thời điểm | Việc | Ai | Output kiểm được |
|---|---|---|---|
| T−2 tuần | Thu thập input: dữ liệu quý trước (throughput, cycle time, tỉ lệ việc phát sinh), danh sách đề xuất từ các bên | PO + EM | Một trang dữ liệu, không phải ý kiến |
| T−1 tuần | Tính capacity thật (trừ nghỉ phép, on-call, onboarding), chốt tỉ lệ bucket | EM + Tech Lead | Bảng capacity (Bước 4) |
| T−3 ngày | Team đọc trước đề xuất, ghi câu hỏi và rủi ro bằng văn bản | Cả team | Danh sách câu hỏi trước buổi họp |
| T (2,5 giờ) | Chốt Now, chốt danh sách không làm, tìm phụ thuộc | PO + EM + Tech Lead + 2 đại diện team | Roadmap bản mới + dependency register |
| T+2 ngày | Công bố, kèm phần "đổi gì so với bản trước và vì sao" | PO | Bản phát hành có số phiên bản |
| Giữa quý (tuần 6–7) | Kiểm tra giữa kỳ: điều chỉnh Now, không lập lại kế hoạch | EM | Một trang cập nhật |

Tiêu chí biết là xong: có thể trả lời được ba câu — quý này chúng ta đặt cược vào điều gì, cái gì bị bỏ,
và ai đang chờ ai.

**Bước 4 — Phân bổ capacity theo bucket, và tính bằng số thật.**

```
CAPACITY PLAN — Nhóm Payments · Q3 · 6 engineer (số minh hoạ)

A. TÍNH CAPACITY THẬT
   6 người × 13 tuần                                 = 78 người-tuần (gộp)
   − Nghỉ phép + lễ (đã đăng ký)                     = −5,0
   − On-call và xử lý sự cố (trung bình 4 quý: 9%)   = −7,0
   − Onboarding 1 người mới (60% hiệu suất 6 tuần)   = −2,4
   − Nghi lễ + họp liên team (12% — đo từ lịch)      = −9,4
   ------------------------------------------------------------
   CAPACITY KHẢ DỤNG                                  = 54,2 người-tuần (69% của con số gộp)

B. PHÂN BỔ THEO BUCKET
   | Bucket                  | Tỉ lệ | Người-tuần | Cách bảo vệ                          |
   | Roadmap (feature mới)   |  55%  |    29,8    | Là phần bị cắt trước khi cắt bucket khác |
   | Technical health        |  20%  |    10,8    | Slot cố định: 2 người, tuần 1-2 mỗi sprint; có tên item |
   | Keep-the-lights-on      |  15%  |     8,1    | WIP limit 2; việc vượt hạn mức thì đẩy sang sprint sau |
   | Khám phá / spike        |  10%  |     5,4    | Timebox 5 ngày mỗi spike, có tiêu chí kết thúc |

C. QUY TẮC ĐỔI (viết trước, để không phải đàm phán lúc đang nóng)
   1. Muốn thêm việc vào Roadmap giữa quý: phải nêu item nào ra khỏi Roadmap. Đổi 1:1 theo kích thước.
   2. Không lấy từ Technical health để bù cho Roadmap. Nếu buộc phải lấy: cần Hà (EM) + Sơn (founder)
      đồng ý bằng văn bản, và ghi vào debt register kèm ngày trả.
   3. KTLO vượt 20% hai sprint liền → coi là tín hiệu hệ thống, mở một item technical health,
      không tiếp tục hút capacity Roadmap.
   4. Bucket khám phá không được dùng để làm feature nhỏ. Nếu hết việc khám phá, trả về Roadmap.

D. NGƯỠNG BÁO ĐỘNG
   - Technical health tiêu dưới 50% phân bổ trong một quý → báo cáo lý do ở quarterly review.
   - Việc phát sinh ngoài kế hoạch vượt 30% throughput → dừng, xem lại chất lượng đầu vào.
```

Ba điểm về bảo vệ bucket đáng nói rõ, vì đây là chỗ mọi kế hoạch tốt chết. **Một:** bucket chỉ tồn tại
nếu nó có tên item cụ thể. "20% cho technical health" không có item là 0% trên thực tế, vì khi cần chỗ,
cái không có tên là cái dễ lấy nhất. **Hai:** bucket được bảo vệ bằng cấu trúc, không bằng ý chí — hai
người cố định trong hai tuần đầu sprint là cấu trúc; "mọi người nhớ dành thời gian refactor" là ý chí,
và ý chí thua deadline mỗi lần. **Ba:** phải có một quy tắc đổi viết trước. Lý do tâm lý rất cụ thể: khi
founder đang đứng trước mặt bạn nói cần thêm một feature, bạn không có năng lực nhận thức để thiết kế
một quy tắc công bằng ngay lúc đó; bạn chỉ có năng lực áp dụng một quy tắc đã có.

**Bước 5 — Liên kết roadmap với OKR mà không biến OKR thành danh sách task.** Phép kiểm tra một dòng:
*nếu bạn làm xong toàn bộ công việc đã lên kế hoạch mà Key Result vẫn không đạt, KR đó có được coi là
thất bại không?* Nếu câu trả lời là "không, vì chúng ta đã làm hết việc" thì OKR của bạn là một danh sách
task được đổi tên. Nếu câu trả lời là "có" thì bạn đang có OKR thật, và điều đó đồng nghĩa với việc bạn
chấp nhận rủi ro giả thuyết sai — đó chính là nội dung của OKR.

| Sai — OKR là task list | Đúng — OKR là outcome | Vì sao khác biệt quan trọng |
|---|---|---|
| KR: "Launch tính năng một chạm trong Q3" | KR: "Tỉ lệ hoàn tất thanh toán 71% → 80%" | Bản sai đạt được kể cả khi tính năng vô dụng |
| KR: "Hoàn thành 12 story của module đối soát" | KR: "Thời gian đối soát cuối ngày 4h → 30 phút" | Bản sai khuyến khích chia nhỏ story để đủ số |
| KR: "Migrate 100% service sang cluster mới" | KR: "p95 latency < 300ms và chi phí hạ tầng giảm 20%" | Bản sai đúng khi migrate xong mà không ai được lợi |
| KR: "Tăng test coverage lên 80%" | KR: "Change failure rate 18% → 8%" | Bản sai đạt được bằng test vô nghĩa |

Cách nối hai tài liệu: mỗi item ở tầng Now ghi một dòng "Đặt cược vào: KR nào", và mỗi KR có ít nhất
một item nhưng không quá ba. Nếu một KR có bảy item, bạn không có giả thuyết, bạn có hy vọng phân tán.
Nếu một item không đặt cược vào KR nào, nó thuộc bucket khác (technical health hoặc KTLO) — và điều đó
hoàn toàn hợp lệ, miễn là nói rõ, vì OKR không nên bao trùm toàn bộ công việc của một team.

Số lượng: tối đa 3 objective cho một team trong một quý, mỗi objective tối đa 3 KR. Không phải vì con
số 3 có tính chất thần bí, mà vì với capacity 54 người-tuần và ba bucket ngoài roadmap, bạn chỉ đủ chỗ
cho 3–5 việc lớn — nhiều objective hơn thế thì mỗi objective nhận dưới một người-tuần mỗi tuần, tức là
không được làm.

### Trade-off

**Cam kết ngày cụ thể vs cam kết theo outcome.** Đây là trade-off thật, không phải câu chuyện "outcome
luôn tốt hơn". Có những tình huống mà cam kết ngày là lựa chọn đúng, và người lead phải biết phân biệt.

| | Cam kết theo ngày | Cam kết theo outcome |
|---|---|---|
| Cái bạn hứa | "Ngày 15/09 tính năng X có trên production" | "Trong Q3 chúng tôi đưa tỉ lệ hoàn tất lên 80%" |
| Cái bị cố định | Thời gian và phạm vi | Kết quả cần đạt |
| Cái được linh hoạt | Chất lượng (thường bị cắt ngầm — xem chủ đề 8) | Giải pháp, phạm vi, và một phần thời gian |
| Ai chịu rủi ro bất định | Team kỹ thuật | Chia sẻ giữa team và người đặt mục tiêu |
| Điều kiện cần để hoạt động | Phạm vi rõ, ít phụ thuộc, đã làm việc tương tự | Đo được kết quả trong chu kỳ; team có quyền đổi giải pháp |
| Hỏng thế nào | Đúng ngày nhưng vô dụng; nợ kỹ thuật tích âm thầm | Không ai biết khi nào có gì để phối hợp; dễ thành lời bào chữa |
| Nghiêng về khi | Có ràng buộc bên ngoài thật: mùa vụ (11/11, Tết), hợp đồng, quy định của cơ quan quản lý, sự kiện đã bán vé | Vấn đề còn nhiều bất định về giải pháp; giá trị của việc làm chưa được chứng minh |

Kết luận thực dụng: dùng cả hai, ở hai tầng khác nhau. Cam kết outcome ở tầng mục tiêu quý; cam kết ngày
ở tầng lát cắt nhỏ trong 2–4 tuần tới, nơi bất định đã đủ hẹp. Điều không được làm là cam kết ngày cho
outcome ở khoảng cách một quý — đó là hứa cả hai biến cùng lúc trong khi chỉ kiểm soát được một.

**Chi tiết vs khả năng thích ứng.** Kế hoạch chi tiết hơn cho phối hợp tốt hơn trong ngắn hạn (ai cũng
biết chính xác phải làm gì) nhưng làm tăng chi phí thay đổi, vì mỗi thay đổi phải lan qua toàn bộ chi
tiết đã viết. Nghiêng về chi tiết khi: chi phí phối hợp sai rất cao (nhiều team, hoặc có sự kiện vật lý
như in ấn, quảng cáo TV), và tần suất thông tin mới thấp. Nghiêng về thô khi: thông tin mới đến nhanh
(sản phẩm đang tìm product-market fit), hoặc một team tự chủ toàn bộ luồng.

**Kế hoạch tập trung vs quyền tự chủ của team.** Nếu lãnh đạo quyết định đến mức feature, team mất khả
năng đề xuất giải pháp rẻ hơn — và chính team là nơi có thông tin về chi phí kỹ thuật. Nếu lãnh đạo chỉ
nêu outcome, có nguy cơ nhiều team giải cùng một vấn đề theo ba hướng không tương thích. Điểm cân bằng
dùng được: lãnh đạo sở hữu *vấn đề và thứ tự ưu tiên*, team sở hữu *giải pháp và ước lượng*, và ranh
giới này được ghi ra thay vì để mỗi bên giả định. Khi tổ chức nhỏ hơn khoảng 30 engineer, lệch về tự chủ
gần như luôn đúng; qua ngưỡng đó, cần thêm cơ chế đồng bộ hướng giải pháp ở tầng kiến trúc (chương 05).

### Real-world Scenarios

**Tình huống A — Sales đã bán một item ở cột Later.**

Bối cảnh: e-commerce B2B, 45 engineer. Trong roadmap, "Tích hợp kế toán MISA" nằm ở Later với chú thích
"chưa cam kết". Tháng 5, một Sales ký hợp đồng 1,8 tỉ với điều khoản "có tích hợp MISA trong tháng 8".
Founder gọi Hà (EM) và Minh (Tech Lead) vào phòng: "Cái này có trong roadmap rồi mà, sao lại nói không
làm được?"

Cách xử lý sai và hay gặp: tranh luận về việc ai đúng ai sai ("em ghi rõ là chưa cam kết mà"). Nó thoả
mãn về mặt cảm xúc nhưng không giải quyết gì, vì hợp đồng đã ký và tiền là thật. Cách xử lý đúng đi theo
ba nhịp, theo thứ tự này.

Nhịp một, tách bài toán hôm nay khỏi bài toán hệ thống. Hôm nay: hợp đồng đã ký, cần một phương án. Minh
đưa ba phương án theo mẫu ở chủ đề 8 — bản đầy đủ 9 tuần (không kịp tháng 8), bản một chiều đồng bộ
không tự động hoá đối soát 4 tuần (kịp, đủ cho khách hàng này dùng, nợ lại phần đối soát), và bản dùng
file CSV thủ công 1 tuần (kịp, nhưng khách sẽ không hài lòng lâu). Founder chọn phương án hai. Ghi lại
lựa chọn, ghi nợ, đặt ngày trả nợ vào Q4 và lấy chỗ từ bucket technical health của Q4 — không phải "sẽ
làm khi có thời gian".

Nhịp hai, sửa cơ chế đã cho phép việc này xảy ra. Ba thay đổi cụ thể: (1) bản roadmap gửi cho Sales là
một bản khác, chỉ có Now và Next, không có Later; (2) mỗi item ở Next có một dòng "được phép nói gì với
khách" viết bằng ngôn ngữ Sales dùng được; (3) một quy tắc: bất kỳ điều khoản hợp đồng có tên tính năng
và ngày phải có chữ ký của Hà trước khi ký. Điểm quan trọng: không thay đổi nào trong ba cái này là "Sales
phải đọc kỹ hơn". Cơ chế phải chịu được người dùng bình thường ở trạng thái đang chịu áp lực chỉ tiêu.

Nhịp ba, chấp nhận một phần lỗi thuộc về roadmap. Cột Later có tên tính năng cụ thể và tên đối tác thật
thì gần như chắc chắn sẽ bị bán. Nếu Later phải chứa item dạng đó, hãy viết ở mức vấn đề: "giảm công
nhập liệu kế toán thủ công cho khách doanh nghiệp" thay vì "tích hợp MISA".

**Tình huống B — Bucket technical health bị lấy hết quý thứ ba liên tiếp, nhìn từ ba tầng.**

Bối cảnh: fintech, team 7 người. Q1, Q2, Q3 đều phân bổ 20% cho technical health. Thực tế tiêu: 6%, 3%,
0% (số minh hoạ). Việc bị hoãn nhiều nhất là tách module hạn mức tín dụng ra khỏi monolith — đã nằm
trong kế hoạch ba quý.

- **Nhìn từ IC (Khoa, Senior BE):** "Tôi đã viết design doc cho việc này ba lần, mỗi lần một quý, và
  mỗi lần nó bị đẩy. Lần thứ ba tôi không viết lại nữa, tôi chỉ đổi ngày trên file cũ. Cái tôi học được
  không phải là 'công ty ưu tiên feature' — cái tôi học được là *viết design doc cho technical health
  là việc vô ích*. Tôi vẫn làm việc tốt, nhưng tôi đã ngừng đề xuất. Mỗi lần có bug ở module hạn mức,
  tôi sửa tại chỗ, nhanh hơn và không phải giải thích với ai. Tôi đang tối ưu đúng theo tín hiệu mà tổ
  chức phát ra." Điểm cần thấy: tổn thất lớn nhất ở đây không phải module chưa được tách, mà là một
  senior đã học được rằng đề xuất cải thiện không có đường dẫn tới kết quả. Cái đó không hồi phục nhanh
  khi bạn cấp lại capacity, vì nó là niềm tin về cơ chế, không phải về một item.

- **Nhìn từ Tech Lead (Minh):** "Tôi thấy dữ liệu: cycle time trung vị của việc chạm vào module hạn mức
  là 11 ngày, so với 4 ngày ở phần còn lại của hệ thống; 40% incident P2 quý này liên quan tới nó; và
  hai người duy nhất dám sửa nó đều là single point of failure. Tôi cũng thấy lỗi của mình: ba quý liền
  tôi trình bày việc này như một việc kỹ thuật, bằng ngôn ngữ kỹ thuật, đặt cạnh những feature có con số
  doanh thu. Trong cuộc so sánh đó tôi thua là hợp lý. Lần thứ tư tôi sẽ trình bày khác: chi phí của
  việc không làm, tính bằng người-tuần đã tiêu cho việc chữa cháy quanh module này trong ba quý — 14
  người-tuần, gần bằng chi phí tách module là 9 người-tuần. Và tôi sẽ không xin 20% chung; tôi xin đúng
  một slot có tên và có ngày."

- **Nhìn từ Manager (Hà, EM):** "Tôi là người đã đồng ý lấy capacity đó ba lần, và mỗi lần tôi đều có
  lý do hợp lý ở thời điểm đó: quý 1 là vòng gọi vốn, quý 2 là mất một khách lớn, quý 3 là đối thủ ra
  tính năng tương tự. Nhìn từng lần thì không có quyết định nào sai. Nhìn ba lần thì có một mẫu, và mẫu
  đó là lỗi của tôi chứ không phải của hoàn cảnh — vì tôi đã không đặt bất kỳ ngưỡng nào. Cái tôi phải
  sửa là ở tầng cơ chế: từ Q4, bucket technical health có tên item, có người, có slot lịch, và việc lấy
  nó cần chữ ký của founder chứ không phải của tôi. Lý do đưa lên founder không phải để đẩy trách nhiệm;
  đó là để chi phí của quyết định hiện ra ở đúng chỗ có thẩm quyền đánh đổi doanh thu với rủi ro hệ
  thống. Tôi cũng phải nói với Khoa một điều khó hơn: rằng tôi hiểu vì sao anh ta ngừng đề xuất, và rằng
  việc lấy lại niềm tin đó là việc của tôi, cần một quý làm đúng chứ không cần một buổi nói chuyện."

Cùng một sự việc: IC đọc thành tín hiệu về giá trị của việc đề xuất; Tech Lead đọc thành lỗi trong cách
đóng gói lập luận; Manager đọc thành thiếu cơ chế và thiếu ngưỡng. Cả ba đều đúng, và chỉ có tầng thứ ba
sửa được vấn đề vĩnh viễn — nhưng nếu chỉ tầng ba hành động mà tầng hai không đổi cách trình bày, slot
mới sẽ lại bị lấy ở quý sau bởi một lý do hợp lý khác.

**Tình huống C — "Cho anh xem kế hoạch cả năm."**

Bối cảnh: một doanh nghiệp truyền thống đang chuyển đổi số. Giám đốc yêu cầu Hà trình bày kế hoạch 12
tháng có ngày, để đưa vào kế hoạch ngân sách của công ty.

*Phiên bản nói sai:*

> **Giám đốc:** Chị cần một kế hoạch 12 tháng, có mốc từng tháng, để trình hội đồng tuần sau.
>
> **Hà:** Dạ, Agile thì bọn em không lập kế hoạch dài như vậy được ạ. Bất định cao quá, ước lượng xa
> thì không chính xác.
>
> **Giám đốc:** Vậy tôi trình hội đồng cái gì? Bộ phận nào cũng có kế hoạch năm, chỉ IT là không có.
>
> **Hà:** Em sẽ cố làm một bản, nhưng anh hiểu là nó sẽ đổi nhiều nhé.
>
> **Giám đốc:** Được, cứ làm.

Kết quả: Hà làm một Gantt 12 tháng mà chính chị không tin, hội đồng phê duyệt ngân sách theo Gantt đó,
và tháng 4 mọi thứ lệch. Chị đã thua hai lần: mất tài liệu (vì nó sai), và mất uy tín (vì chị đã nộp một
thứ chị biết là sai). Sai lầm gốc nằm ở nhịp đầu: chị phản đối *hình thức* của yêu cầu mà không tìm hiểu
*nhu cầu* đằng sau. Nhu cầu thật ở đây là ngân sách, không phải lịch.

*Phiên bản nói đúng:*

> **Giám đốc:** Chị cần một kế hoạch 12 tháng, có mốc từng tháng, để trình hội đồng tuần sau.
>
> **Hà:** Em làm được. Trước khi em làm, em muốn chắc là em làm đúng thứ anh cần: hội đồng sẽ dùng tài
> liệu này để quyết định gì?
>
> **Giám đốc:** Để duyệt ngân sách. Họ cần biết tiền đi vào đâu và khi nào thấy kết quả.
>
> **Hà:** Vậy em đề nghị cấu trúc thế này, và em nói trước là nó khác Gantt. Ba phần. Phần một: ngân
> sách cả năm chia theo bốn nhóm đầu tư — nền tảng, tự động hoá nghiệp vụ, tuân thủ, và vận hành. Con
> số này em cam kết được, vì nó là tỉ lệ phân bổ, và nó đúng là thứ hội đồng cần để duyệt tiền. Phần
> hai: quý 3 — em cam kết bốn kết quả cụ thể, có ngày, có tiêu chí đo. Phần ba: quý 4 đến quý 2 sang
> năm — em nêu bốn chủ đề theo thứ tự ưu tiên và câu hỏi mỗi chủ đề cần trả lời, không có ngày.
>
> **Giám đốc:** Hội đồng sẽ hỏi tại sao không có ngày cho quý 4.
>
> **Hà:** Em sẽ trả lời thẳng trong slide: "Chúng tôi có thể đưa ngày cho quý 4, nhưng độ chính xác của
> nó là cộng trừ hai tháng, và nếu hội đồng lập kế hoạch dựa trên nó thì rủi ro nằm ở phía hội đồng.
> Thay vào đó, chúng tôi cam kết bốn điểm rà soát — cuối mỗi quý — và ở mỗi điểm chúng tôi đưa ngày cho
> quý kế tiếp với sai số dưới hai tuần." Với anh, em cam kết thêm một việc: nếu ngân sách của một nhóm
> đầu tư lệch quá 15%, em báo trong vòng hai tuần chứ không chờ hết quý.
>
> **Giám đốc:** Cái đó tôi bảo vệ được. Nhưng phải có một mốc lớn trong năm để hội đồng nhớ.
>
> **Hà:** Có: hệ thống duyệt chi mới thay thế quy trình giấy, cam kết chạy thật ở một chi nhánh trước
> 30/11. Đó là mốc em dám cam kết ngày, vì nó thuộc quý 3–4 và bọn em đã làm một hệ thống tương tự.
> Những mốc còn lại em không cam kết ngày, và em sẽ nói rõ cái nào là cam kết, cái nào là dự báo.

Bảng mổ xẻ:

| Nhịp | Bản sai | Bản đúng | Cơ chế đằng sau |
|---|---|---|---|
| Phản ứng đầu | Phản đối hình thức ("Agile không làm thế") | Hỏi tài liệu dùng để quyết định gì | Thương lượng về nhu cầu, không về định dạng; và tránh dùng phương pháp luận làm lá chắn |
| Cấu trúc đề xuất | Nhận làm Gantt mình không tin | Tách tài liệu thành ba phần theo ba mức cam kết | Mỗi phần trả lời một nhu cầu khác nhau với độ chắc khác nhau |
| Xử lý ràng buộc thật | Bỏ qua nhu cầu ngân sách | Cam kết tỉ lệ phân bổ — thứ ổn định hơn ngày rất nhiều | Ngân sách theo bucket không cần biết ngày từng feature |
| Trả lời câu hỏi khó | "Nó sẽ đổi nhiều nhé" | Nói rõ sai số và chuyển rủi ro về đúng người | Nêu độ rộng của khoảng là cách giữ uy tín, không phải cách chối |
| Cái được đưa thay thế | Không có | Bốn điểm rà soát + một mốc lớn dám cam kết | Cấp cho stakeholder cơ chế kiểm soát thay vì cấp con số giả |
| Uy tín sau 4 tháng | Mất, vì kế hoạch sai | Còn, vì đã tuyên bố trước độ chính xác | Uy tín đến từ việc dự báo khớp với thực tế, kể cả khi dự báo là "không chắc" |

### Best Practices

**Đánh số phiên bản roadmap và luôn kèm phần "đổi gì so với bản trước".** Lý do: thay đổi là bình thường,
thay đổi *âm thầm* mới phá niềm tin. Khi người đọc thấy phần diff, họ học được rằng roadmap này sống và
đáng đọc. Khi không thấy, họ giả định bản mới nhất là bản duy nhất từng tồn tại và mọi thay đổi là lỗi.

**Phát hành nhiều phiên bản cho nhiều đối tượng, từ cùng một nguồn.** Bản cho Sales chỉ có Now/Next và
kèm "được phép nói gì". Bản cho engineer có đầy đủ điều kiện và phụ thuộc. Bản cho hội đồng có phân bổ
ngân sách. Đây không phải là che thông tin; đây là điều chỉnh độ phân giải theo quyết định người đọc cần
ra. Một tài liệu duy nhất cho mọi đối tượng sẽ hoặc quá chi tiết để dùng, hoặc quá thô để tin.

**Viết mục "không làm" trước khi viết mục "sẽ làm".** Cơ chế: nếu bạn viết danh sách sẽ làm trước, bạn
sẽ dừng khi hết chỗ và không bao giờ đối diện với việc từ chối ai. Viết danh sách không làm trước buộc
bạn nói ra các đánh đổi trong lúc bạn còn bình tĩnh, và tạo ra một tài liệu bạn có thể chỉ vào khi bị
ép giữa quý.

**Gắn mỗi item Now với một KR và một giả thuyết đo được.** Lý do kép: nó buộc kiểm tra xem việc này có
phục vụ mục tiêu nào không, và nó tạo ra một điểm học — cuối quý bạn biết giả thuyết đúng hay sai, chứ
không chỉ biết việc đã làm hay chưa.

**Chốt tỉ lệ phân bổ capacity trước khi chọn item.** Thứ tự này quan trọng vì nó quyết định cuộc tranh
luận nào diễn ra. Chọn item trước rồi tính capacity sau nghĩa là bạn sẽ nhét cho vừa, và cái bị nhét ra
là thứ không có người bảo vệ.

**Đặt một điểm rà soát giữa quý và giới hạn nó là "điều chỉnh", không phải "lập lại kế hoạch".** Lý do:
không có điểm rà soát thì kế hoạch chỉ được sửa khi đã hỏng; có quá nhiều điểm thì kế hoạch mất tác dụng
làm neo phối hợp. Một lần giữa quý là điểm cân bằng dùng được cho chu kỳ 13 tuần.

**Tính capacity bằng số đo lịch sử, không bằng số lý thuyết.** Cụ thể: dùng tỉ lệ on-call, tỉ lệ việc
phát sinh, tỉ lệ họp của bốn quý gần nhất. Lý do: sai số hệ thống của kế hoạch nằm gần hết ở chỗ này —
người ta lập kế hoạch cho 100% thời gian trong khi thực tế khả dụng khoảng 65–70%.

**Nói rõ mức cam kết bằng lời, mỗi lần trình bày, kể cả khi nó đã ghi trong tài liệu.** Người nghe không
đọc chú thích. Câu "phần này là cam kết, phần này là dự định, phần này có thể bỏ hoàn toàn" mất mười
giây và ngăn được phần lớn hiểu sai.

### Anti-patterns

**Roadmap là danh sách feature có ngày cho 12 tháng.** Hình dạng: spreadsheet hoặc Gantt, mỗi dòng một
feature, mỗi feature một tháng, độ chi tiết đồng đều từ tháng 1 đến tháng 12. Cơ chế phá hệ thống ở ba
tầng. Tầng một, nó tuyên bố thông tin chưa tồn tại, nên nó sai một cách hệ thống — và khi một tài liệu
sai đủ nhiều lần, người đọc ngừng đọc, khiến tổ chức mất kênh phối hợp mà không nhận ra. Tầng hai, nó
tạo tồn kho kế hoạch: 47 dòng chi tiết là 47 thứ phải bảo trì, nên chi phí thay đổi cao, nên tổ chức
kháng lại thông tin mới đúng lúc thông tin đó có giá trị nhất. Tầng ba, nó xoá mất phân biệt giữa dự báo
và cam kết, nên vừa làm hỏng dự báo vừa làm hỏng cam kết. Dấu hiệu sớm: cột "target month" có giá trị
cho mọi dòng; không có mục "không làm"; roadmap được cập nhật bởi một người trong Excel chứ không sinh
ra từ một buổi có mặt người quyết định.

**OKR có chín objective.** Hình dạng: một team 7 người có 9 objective, 26 key result, và một dashboard
màu. Cơ chế: với capacity khả dụng khoảng 54 người-tuần một quý, 9 objective nhận trung bình 6 người-tuần
mỗi cái — không đủ để hoàn thành bất cứ thứ gì có ý nghĩa, nên cuối quý mọi objective đều ở 40–60% và
không có thông tin nào được sinh ra. Sâu hơn: mục đích của OKR là buộc *lựa chọn*; 9 objective là bằng
chứng rằng không có lựa chọn nào được thực hiện, tức là công cụ đã bị vô hiệu trong khi vẫn tiêu chi
phí quản trị. Dấu hiệu sớm: mỗi bộ phận đề xuất được thêm một objective và không cái nào bị bác; hoặc
có objective mà không ai trong team nhớ được nội dung mà không mở file.

**Planning không nói rõ cái gì sẽ không làm.** Hình dạng: buổi planning kết thúc với danh sách việc sẽ
làm, không có danh sách việc bị gạt, không có tên người được thông báo là yêu cầu của họ bị hoãn. Cơ
chế: những yêu cầu không được từ chối tường minh không biến mất — chúng chuyển sang kênh phi chính thức
và quay lại giữa quý dưới dạng việc gấp do người đề xuất nhắn trực tiếp cho engineer. Kết quả là tỉ lệ
việc ngoài kế hoạch cao, và ngược lại điều đó làm mọi kế hoạch tiếp theo trở nên vô nghĩa, tạo vòng lặp
tự củng cố. Dấu hiệu sớm: khi hỏi "quý này ta quyết định không làm gì", không ai trả lời được bằng một
danh sách; hoặc trên 30% throughput là việc không có trong kế hoạch quý.

**Roadmap do một người viết trong phòng kín.** Cơ chế: item không được người thực thi kiểm tra tính khả
thi sẽ chứa những giả định sai mà chỉ engineer biết là sai, và độ trễ sinh ra ở giữa quý thay vì ở buổi
planning — đúng lúc sửa đắt nhất. Dấu hiệu sớm: team nghe về roadmap lần đầu ở buổi công bố; hoặc câu
"cái này ai hứa?" xuất hiện.

**Roadmap không có phụ thuộc liên team.** Cơ chế: mỗi team lập kế hoạch cho 100% capacity của mình, và
thời gian chờ team khác không nằm trong kế hoạch của bất kỳ ai (chủ đề 6). Dấu hiệu sớm: hai team có
item liên quan nhau nhưng không có ngày giao diện nào trùng nhau trong hai kế hoạch.

**Đẩy item sang quý sau như thao tác mặc định.** Hình dạng: cuối quý, mọi việc chưa xong được kéo sang
Q kế tiếp mà không xem lại giá trị. Cơ chế: nó biến roadmap thành hàng đợi FIFO không có cơ chế bỏ, nên
những việc mất giá trị vẫn tiêu capacity, và những việc mới có giá trị cao không có chỗ. Dấu hiệu sớm:
có item đã bị đẩy ba quý; không có item nào từng bị *xoá* khỏi backlog trong sáu tháng.

**Dùng roadmap để quản lý tiến độ hàng ngày.** Cơ chế: roadmap là công cụ ở tầng quý; dùng nó ở tầng
tuần nghĩa là cập nhật liên tục, và cập nhật liên tục làm nó mất tính neo. Dấu hiệu sớm: roadmap được
mở trong standup.

### Khi nào KHÔNG nên áp dụng

**Khi bạn ở ODC và khách hàng sở hữu roadmap.** Trong hợp đồng fixed-scope đã ký SOW, roadmap thật nằm
trong hợp đồng và bạn không có quyền đổi nó. Cố áp Now/Next/Later ở đây tạo ra một tài liệu song song
không có thẩm quyền, và tệ hơn, có thể bị khách đọc là bạn đang tự ý sắp xếp lại phạm vi đã ký. Cái nên
làm thay: một bản kế hoạch giao hàng theo mốc hợp đồng, cộng một tài liệu riêng ghi phụ thuộc và điều
kiện phía khách (chủ đề 6) — vì trong bối cảnh này, rủi ro lớn nhất không phải ưu tiên sai mà là chờ
quyết định và môi trường từ phía khách. Nếu bạn muốn có tiếng nói về ưu tiên, chỗ để đấu tranh là vòng
đàm phán change request, không phải một roadmap nội bộ.

**Khi tổ chức dưới khoảng 10–15 engineer và chưa có product-market fit.** Ở quy mô này, chi phí phối hợp
mà roadmap giúp tiết kiệm là gần bằng không — mọi người ngồi cùng phòng và biết nhau đang làm gì. Ngược
lại, chu kỳ học rút ngắn xuống một đến hai tuần, nên một tài liệu chu kỳ quý sẽ lỗi thời trước khi được
đọc lần thứ hai. Cái nên làm thay: một danh sách 3–5 đặt cược đang chạy, mỗi cái có một câu hỏi cần trả
lời và một ngày quyết định, cập nhật hàng tuần. Ngưỡng chuyển đổi thực dụng: khi có từ ba team trở lên,
hoặc khi có một bộ phận không phải kỹ thuật cần biết trước để lập kế hoạch (Sales, Marketing có ngân
sách chiến dịch), thì bắt đầu cần Now/Next/Later.

**Khi cả quý chỉ có một mục tiêu do ràng buộc bên ngoài.** Ví dụ: một ngân hàng phải tuân thủ một quy
định mới có hiệu lực ngày cố định, và toàn bộ capacity của hai quý dùng cho việc đó. Trong tình huống
này, phân bổ bốn bucket là nghi lễ — không có gì để phân bổ. Cái nên làm thay: một kế hoạch theo mốc
tuân thủ, một risk register (chủ đề 7), và một quyết định tường minh rằng technical health bị hoãn cùng
với ngày sẽ xem lại. Điểm cần giữ: vẫn phải ghi lại rằng nó bị hoãn, nếu không quý sau sẽ bắt đầu từ
giả định rằng 0% là bình thường.

**Khi tổ chức chưa có khả năng nói không.** Đây là điều kiện biên khó nhất. Nếu mỗi item được ghi vào
Later đều bị đọc là một lời hứa — vì văn hoá tổ chức không cho phép ai từ chối một yêu cầu đã được ghi
ra — thì tầng Later biến thành một hàng đợi hứa hẹn và roadmap làm hại nhiều hơn lợi. Bạn sẽ nhận diện
được điều này khi mọi cuộc thảo luận về Later đều kết thúc bằng "vậy khi nào làm". Thứ tự đúng: tạm thời
chỉ công bố Now và một danh sách vấn đề chưa xếp thứ tự (không phải giải pháp có tên), song song với việc
xây năng lực từ chối ở tầng lãnh đạo (chương 00 và 04). Công cụ lập kế hoạch không thể sửa vấn đề thẩm
quyền, và nếu bạn cố dùng nó để sửa, nó sẽ hấp thụ toàn bộ trách nhiệm của một quyết định mà không có
quyền lực để thực thi.

**Khi bạn đang trong một quý phục hồi sau sự cố lớn hoặc sau khi mất nhiều người.** Lập kế hoạch quý
trong điều kiện chưa biết team còn lại những ai và hệ thống còn hỏng chỗ nào sẽ cho ra một kế hoạch phải
huỷ trong ba tuần, và cái giá thật là uy tín của chính công cụ. Cái nên làm thay: chu kỳ bốn tuần với
một mục tiêu duy nhất, cộng một điều kiện tường minh để quay lại nhịp quý ("khi tỉ lệ việc phát sinh
dưới 20% hai chu kỳ liền").

---

## 6. Dependency Management

### Problem Statement

Một e-commerce Việt, 6 team, 70 engineer. Tính năng "đổi trả hàng tự động" được giao cho team Order.
Ngày bắt đầu 3/3, ngày lên production 19/5. Sau khi xong, Vy (Tech Lead của Order) làm một việc ít ai
làm: dựng lại dòng thời gian của từng ngày và phân loại thành thời gian *có người đang làm* và thời gian
*đang chờ*.

| Giai đoạn | Ngày | Loại | Ai đang chờ ai |
|---|---|---|---|
| Thiết kế + thống nhất luồng | 3/3–10/3 | Làm việc: 6 ngày | — |
| Chờ team Payment mở API hoàn tiền | 11/3–2/4 | Chờ: 16 ngày | Order chờ Payment |
| Làm phần Order | 3/4–14/4 | Làm việc: 9 ngày | — |
| Chờ team Logistics xác nhận format nhãn trả hàng | 15/4–24/4 | Chờ: 7 ngày | Order chờ Logistics |
| Chờ quyết định của nghiệp vụ về phí đổi trả | 15/4–29/4 | Chờ: 10 ngày (chồng lấn) | Order chờ Vận hành |
| Tích hợp + sửa lệch hợp đồng API | 30/4–8/5 | Làm việc: 6 ngày | — |
| Chờ cửa sổ release và ký duyệt | 9/5–19/5 | Chờ: 7 ngày | Order chờ quy trình |
| **Tổng** | **78 ngày lịch** | **21 ngày làm việc · 40 ngày chờ** | |

Flow efficiency ở đây là 21/78, khoảng 27% (số minh hoạ, nhưng nằm đúng trong dải hay gặp: các tổ chức
nhiều team thường ở 15–40%). Điểm đáng chú ý không phải con số. Điểm đáng chú ý là: trong suốt 78 ngày
đó, **không có báo cáo tiến độ nào của bất kỳ team nào thể hiện 40 ngày chờ.** Team Order báo "đang làm,
bị block" — và cột "bị block" trên board không có tuổi. Team Payment báo sprint của họ hoàn thành 92%
story point, đúng như vậy: yêu cầu của Order không nằm trong sprint của Payment nên không xuất hiện ở đâu
cả. Team Logistics thậm chí không biết mình đang là đường tới hạn của người khác. Nghiệp vụ nghĩ họ đã
trả lời "đang xem xét" tức là mọi thứ vẫn ổn.

Hậu quả nếu không có cơ chế quản lý dependency, nói bằng hiện tượng đo được: lead time gấp ba đến bốn
lần tổng thời gian làm việc; mọi ước lượng của từng team đều đúng trong khi ngày giao của tính năng trượt
sáu tuần; và khi truy nguyên, không có ai để hỏi trách nhiệm, vì mỗi team đều đã hoàn thành phần của
mình theo cam kết của mình.

### First Principles

**Nguyên lý 1 — Thời gian chờ vô hình vì hệ thống báo cáo được tổ chức theo team, còn giá trị được tạo
ra xuyên team.** Đây là bản chất của vấn đề, và nó là vấn đề *cấu trúc thông tin*, không phải vấn đề thái
độ. Mỗi team có một board, một sprint, một velocity. Đơn vị được đo là "công việc team này hoàn thành".
Một yêu cầu từ team khác, chưa được nhận vào sprint, không tồn tại trên bất kỳ đơn vị đo nào: nó không
làm giảm velocity của Payment, không làm tăng cycle time của Payment, không xuất hiện trong báo cáo của
Payment. Với Order, nó xuất hiện, nhưng dưới dạng một thẻ bị block — và ở phần lớn công cụ, thẻ bị block
vẫn nằm trong "In Progress", nên nó thậm chí làm cho board của Order trông có vẻ đang hoạt động. Kết
luận thực hành đi thẳng từ đây: **muốn quản lý dependency, việc đầu tiên là làm cho thời gian chờ trở
thành một con số có tên, có tuổi, có chủ.** Mọi thứ khác là thứ yếu.

**Nguyên lý 2 — Chi phí phối hợp tăng theo số cặp phụ thuộc, và xác suất đúng hạn giảm theo tích.** Với
n thành phần cần phối hợp, số cặp tiềm năng là n(n−1)/2: 3 team có 3 cặp, 6 team có 15 cặp, 10 team có
45 cặp. Nhưng phần đáng lo hơn là số học của xác suất. Nếu mỗi phụ thuộc được đáp ứng đúng hạn với xác
suất p, và bạn cần k phụ thuộc cùng đúng hạn thì xác suất tổng là p^k:

| Số phụ thuộc cần đúng hạn | p = 0,9 | p = 0,8 | p = 0,7 |
|---|---|---|---|
| 1 | 90% | 80% | 70% |
| 3 | 73% | 51% | 34% |
| 5 | 59% | 33% | 17% |
| 8 | 43% | 17% | 6% |

Đọc bảng này cho một hệ quả quan trọng: với 5 phụ thuộc và tỉ lệ đúng hạn 80% mỗi cái — một tỉ lệ mà
phần lớn team sẽ tự đánh giá là "khá tốt" — xác suất kế hoạch của bạn đúng là một phần ba. Điều đó có
nghĩa là không có cách nào cải thiện đáng kể bằng việc "đôn đốc tốt hơn": để đi từ 33% lên 70% bằng cách
tăng p, bạn cần p = 0,93 cho cả năm phụ thuộc. Đòn bẩy nằm ở việc *giảm k*, không ở việc tăng p. Đây là
lý do chiến lược số một trong Framework là loại bỏ phụ thuộc, không phải quản lý nó tốt hơn.

**Nguyên lý 3 — Một phụ thuộc là một hàng đợi mà bạn không sở hữu server.** Từ Queueing Theory: thời gian
chờ trong hàng đợi tăng theo ρ/(1−ρ) với ρ là mức tận dụng của server. Nếu team Payment đang chạy ở mức
tận dụng 90% — nghĩa là gần như toàn bộ capacity của họ đã được cam kết cho công việc của chính họ —
thì yêu cầu của bạn chờ trung bình khoảng 9 lần thời gian xử lý thực tế. Một API cần 2 ngày để làm sẽ
mất khoảng 18 ngày để có. Hai kết luận đi ra từ đây, và cả hai đều phản trực giác với cách phần lớn tổ
chức hành xử. Thứ nhất: hiện tượng "team platform lúc nào cũng chậm" thường không phải vấn đề năng lực
hay thái độ, mà là hệ quả toán học của việc họ bị lập kế hoạch ở mức tận dụng cao. Thứ hai: cách sửa
đúng là để team bị phụ thuộc nhiều có *slack có chủ đích* — ví dụ 20% capacity dành cho yêu cầu liên
team — chứ không phải yêu cầu họ làm nhanh hơn. Một team platform ở mức tận dụng 70% sẽ phục vụ nhanh
hơn nhiều lần cùng một team ở mức 95%, với cùng số người.

**Nguyên lý 4 — Ở điểm hợp lưu, phương sai cộng dồn nhưng phần xong sớm không được cộng.** Nếu ba nhánh
phải xong trước khi bạn tích hợp, ngày tích hợp bằng ngày của nhánh muộn nhất. Một nhánh xong sớm ba
ngày không cứu được nhánh muộn hai ngày. Đây là merge bias trong Critical Chain: mỗi điểm hợp lưu là một
bộ khuếch đại độ trễ một chiều. Hệ quả thực hành: đừng đặt buffer đều ở từng nhánh; đặt buffer *tại điểm
hợp lưu*, và giảm số điểm hợp lưu bằng cách tích hợp liên tục từng phần thay vì tích hợp một lần ở cuối.

### Mental Model

**Theory of Constraints với ràng buộc nằm ngoài quyền hạn của bạn.** Ở chủ đề 3, ràng buộc nằm trong
team và bạn có thể điều chỉnh WIP hoặc phân bổ người. Ở đây, ràng buộc là một team khác — bạn thấy nó,
nhưng bạn không có thẩm quyền đối với nó. Điều này thay đổi bộ công cụ hoàn toàn: bạn không còn tối ưu
được ràng buộc, bạn chỉ có ba lựa chọn — đi đường khác (loại bỏ phụ thuộc), giảm phụ thuộc thời gian với
nó (contract và mock), hoặc dùng thẩm quyền của người khác (escalation). Nhận ra mình đang ở tình huống
"ràng buộc ngoài quyền hạn" là nửa đầu của việc xử lý đúng; nửa sau là chọn đúng một trong ba, thay vì
làm cái mặc định là gửi thêm tin nhắn nhắc.

**Conway's Law đọc ngược.** Bản đồ dependency của bạn là ảnh chụp của kiến trúc và của ranh giới sở hữu.
Nếu một tính năng người dùng nào cũng cần ba team chạm vào, đó không phải vấn đề phối hợp — đó là ranh
giới team hoặc ranh giới service đặt sai so với luồng giá trị. Cách dùng mô hình: mỗi quý, nhìn danh sách
phụ thuộc và tìm mẫu lặp. Một phụ thuộc xuất hiện một lần là việc phối hợp; cùng một phụ thuộc xuất hiện
trong bảy trên mười tính năng là một tín hiệu tổ chức, và cách sửa nó nằm ở chương 09 (Team Topologies,
đặt lại ranh giới), không nằm ở việc thêm một buổi sync.

**Phụ thuộc cứng vs phụ thuộc mềm.** Đây là mô hình đơn giản nhất nhưng có tác dụng lớn nhất trong thực
tế, vì phần lớn phụ thuộc được đối xử như cứng trong khi chúng mềm. Cứng: không có nó thì việc không thể
hoàn thành theo bất kỳ cách nào (không có giấy phép pháp lý thì không được phát hành). Mềm: có nó thì
tốt hơn, rẻ hơn, đúng chuẩn hơn — nhưng có đường thay thế với chi phí xác định được. Bài kiểm tra: hỏi
"nếu thứ này không bao giờ đến, việc này còn cách nào hoàn thành không, và giá của cách đó là bao nhiêu?"
Nếu trả lời được bằng một con số, phụ thuộc đó là mềm và bạn đang có một quyết định, không phải một sự
bế tắc.

### Practical Framework

**Bước 1 — Vẽ bản đồ phụ thuộc, trong buổi planning, bằng bộ câu hỏi cố định.** Đừng hỏi chung "có phụ
thuộc gì không" — câu đó luôn nhận về "không có gì đáng kể". Hỏi sáu câu cụ thể:

1. Việc này cần dữ liệu hoặc API nào không do team ta sở hữu?
2. Cần môi trường, tài khoản, quyền truy cập, hoặc credential do ai cấp?
3. Cần một quyết định nghiệp vụ, pháp chế, hoặc thiết kế mà hiện chưa có bằng văn bản?
4. Cần một người cụ thể ngoài team (DBA, security, một senior duy nhất biết module X)?
5. Có ai đang chờ *ta*? Họ cần gì, khi nào?
6. Việc phát hành cần ai ký, cửa sổ nào, và ai chuẩn bị được từ bây giờ?

Câu 5 là câu hay bị bỏ và là câu tạo ra nhiều giá trị nhất, vì phụ thuộc chỉ hiện ra đầy đủ khi cả hai
chiều được ghi. Câu 3 và 4 là hai nguồn phụ thuộc lớn nhất mà không có tên trên bất kỳ board nào.

**Bước 2 — Chọn chiến lược theo đúng thứ tự ưu tiên này.** Sai lầm phổ biến là nhảy thẳng xuống chiến
lược 4 (chấp nhận và theo dõi) vì nó dễ nhất, trong khi 1 và 2 rẻ hơn nhiều nếu xét đến hết hậu quả.

| # | Chiến lược | Cách làm | Chi phí | Dùng khi | Không dùng khi |
|---|---|---|---|---|---|
| 1 | **Loại bỏ bằng đổi thiết kế** | Thu hẹp phạm vi để không cần thứ đó; hoặc chọn giải pháp kỹ thuật không cần chạm vào hệ thống của team khác; hoặc tự sở hữu phần nhỏ cần dùng | Thường là mất một phần tính năng hoặc chấp nhận giải pháp kém đẹp hơn | Luôn xét đầu tiên. Đặc biệt khi phụ thuộc rơi vào team có mức tận dụng cao | Khi việc tự làm tạo trùng lặp ở chỗ chắc chắn sẽ phân kỳ (bảo mật, tính toán tiền) |
| 2 | **Tách bằng contract và mock** | Chốt hợp đồng giao diện trước, hai bên làm song song, bên phụ thuộc dùng mock/stub và feature flag | 1–3 ngày để thống nhất contract + chi phí duy trì mock | Phụ thuộc là dữ liệu hoặc API; hai bên đều đồng ý được về hình dạng giao diện | Khi miền nghiệp vụ chưa hiểu, contract sẽ sai và chi phí làm lại vượt lợi ích |
| 3 | **Đàm phán thứ tự** | Xin đưa việc của bạn lên trước trong hàng đợi của team kia, bằng cách đánh đổi cụ thể | Chi phí quan hệ; bạn nợ họ một lần | Phụ thuộc nhỏ (1–3 ngày công của họ) và bạn nêu được vì sao việc bạn ưu tiên hơn | Khi bạn không có gì để đổi và không có ràng buộc kinh doanh rõ để dựa vào |
| 4 | **Chấp nhận và làm rõ ngày** | Ghi vào dependency register, có tên người, ngày cam kết, hành động nếu trượt, ngày escalation | Thấp về công, cao về rủi ro còn lại | Phụ thuộc thật sự cứng: pháp lý, hạ tầng, đối tác bên ngoài | Khi bạn dùng nó cho mọi phụ thuộc vì nó dễ nhất |

Quy tắc kèm theo: với mỗi phụ thuộc, phải ghi lại *đã xét chiến lược 1 và 2 chưa và vì sao loại*. Không
có dòng đó, mặc định của tổ chức sẽ luôn là chiến lược 4.

**Bước 3 — Contract-first và feature flag: hai công cụ phá phụ thuộc thời gian.** Ý tưởng gốc: phụ thuộc
có hai thành phần — phụ thuộc *thông tin* (bạn cần biết hình dạng của giao diện) và phụ thuộc *thời gian*
(bạn cần thứ thật tồn tại). Contract-first tách hai cái đó: bạn lấy phụ thuộc thông tin sớm (một buổi 90
phút) và trả phụ thuộc thời gian về cuối, khi chỉ còn việc thay mock bằng hàng thật.

```
INTERFACE AGREEMENT — IA-2025-07 · Hoàn tiền cho đơn đổi trả
Bên cung cấp: team Payment (Tuấn — Tech Lead) · Bên tiêu thụ: team Order (Vy — Tech Lead)
Ngày chốt contract: 12/03 · Ngày dự kiến có hàng thật: 02/04 · Trạng thái: contract đã đóng băng

1. HỢP ĐỒNG (nguồn sự thật: openapi/payments/refund-v1.yaml, commit a91f2c)
   POST /v1/refunds
   Request : { order_id, amount_minor:int, currency:"VND", reason_code, idempotency_key }
   Response: 202 { refund_id, status:"pending" }
   Lỗi     : 400 invalid_amount · 409 duplicate_idempotency_key · 422 order_not_refundable
             503 provider_unavailable (bên tiêu thụ PHẢI retry với backoff)
   Webhook : refund.succeeded | refund.failed  (at-least-once, có thể trùng — phải idempotent)
   SLO cam kết: p95 < 800ms · thời gian tới trạng thái cuối < 2 giờ (99%)

2. CÁCH LÀM SONG SONG
   - Payment cung cấp: file OpenAPI + bộ 12 ví dụ response (gồm 5 ca lỗi) trong 2 ngày.
   - Order dựng mock từ file đó (WireMock), chạy trong CI, không chờ hàng thật.
   - Contract test chạy hai chiều: Payment chạy provider test, Order chạy consumer test.
     CI của Payment fail nếu vi phạm contract. Đây là cơ chế duy nhất khiến contract có hiệu lực.
   - Order bọc toàn bộ luồng bằng flag: order.auto_refund (mặc định off).

3. QUY TẮC ĐỔI CONTRACT SAU KHI ĐÓNG BĂNG
   - Thay đổi tương thích (thêm field optional): thông báo trước 1 ngày, không cần họp.
   - Thay đổi phá vỡ: cần cả hai Tech Lead đồng ý, và bên đề xuất chịu chi phí sửa của bên kia.
     Ghi lại trong file này, tăng số phiên bản.

4. ĐIỀU GÌ XẢY RA NẾU HÀNG THẬT TRỄ
   - Order vẫn hoàn thành và merge phần của mình trước 02/04, flag off, có test trên mock.
   - Nếu tới 09/04 chưa có hàng thật: bật phương án tạm — hoàn tiền thủ công qua công cụ nội bộ
     (Vận hành xử lý, giới hạn 50 đơn/ngày). Chủ phương án tạm: Vy. Ngày rút phương án tạm: khi
     flag bật 100%.
   - Escalation: nếu 09/04 chưa có, Vy báo Hà (EM) trong ngày; không chờ tới 02/04 mới nói.

5. CÁI CONTRACT NÀY KHÔNG BAO GỒM
   - Đối soát cuối ngày giữa refund và sổ kế toán (IA riêng, chưa mở).
   - Hoàn tiền một phần (out of scope v1 — đã quyết định, xem ADR-91).
```

Điểm quan trọng nhất trong template trên là mục 2, dòng contract test. Một contract không có test tự động
kiểm tra là một tài liệu, và tài liệu sẽ phân kỳ với thực tế trong vòng hai tuần. Điểm quan trọng thứ hai
là mục 4: contract-first không loại bỏ rủi ro, nó chuyển rủi ro từ "toàn bộ tính năng trễ" sang "một phần
tính năng chạy ở chế độ tạm". Phải viết trước phần chế độ tạm, vì lúc đang trễ thì không ai còn bình tĩnh
để thiết kế nó.

Feature flag đóng vai trò bổ sung: nó cho phép merge code chưa dùng được vào nhánh chính, nên phụ thuộc
không còn kéo theo một nhánh dài hạn — và nhánh dài hạn là nguồn xung đột merge, thứ khiến chi phí chờ
tăng theo bình phương thời gian chờ. Kỷ luật kèm theo: mỗi flag có ngày hết hạn và người sở hữu, nếu
không bạn đổi một loại nợ lấy một loại nợ khác (chương 05).

**Bước 4 — Dependency register.** Một bảng duy nhất, ở nơi ai cũng thấy, cập nhật mỗi tuần. Đây là công
cụ làm cho thời gian chờ có tên và có tuổi.

| ID | Ta cần gì | Từ ai (tên người) | Loại | Cứng/Mềm | Ngày cần | Ngày họ cam kết | Tuổi chờ | Nếu trượt thì | Phương án dự phòng | Ngày escalation | Trạng thái |
|---|---|---|---|---|---|---|---|---|---|---|---|
| D-01 | API hoàn tiền v1 | Tuấn (Payment) | API | Mềm | 02/04 | 02/04 | 0 | +0 (đã có mock) | Hoàn tiền thủ công ≤50 đơn/ngày | 09/04 | Đúng hạn |
| D-02 | Format nhãn trả hàng | Sơn (Logistics) | Đặc tả | Cứng | 20/03 | 27/03 | 9 ngày | +5 ngày cho Order | Dùng nhãn hiện tại, in thêm mã QR tạm | 24/03 | Trễ — đang theo |
| D-03 | Chốt biểu phí đổi trả | Trang (PO) → Vận hành | Quyết định | Cứng | 18/03 | chưa có ngày | 14 ngày | Chặn phần tính tiền | Hardcode 0đ, bật cấu hình sau | 21/03 | Đã escalate 21/03 |
| D-04 | Môi trường UAT của khách | Khách (bà K., IT Manager) | Môi trường | Cứng | 25/03 | 01/04 | 7 ngày | Toàn bộ UAT lùi | Test trên môi trường nội bộ, ghi rõ khác biệt | 26/03 | Trễ — đã có văn bản |
| D-05 | Review bảo mật luồng hoàn tiền | Nam (Security) | Người | Cứng | 10/04 | 10/04 | 0 | Không được phát hành | Đặt lịch sớm 3 tuần trước | 07/04 | Đã đặt lịch |
| D-06 | *Team Growth chờ ta* — event đổi trả | Ta → Linh (Growth) | API | Mềm | 15/04 | 15/04 (ta cam kết) | — | Họ lùi 1 sprint | — | — | Ta đang nợ |

Bốn cột thường bị thiếu và mỗi cột giải một vấn đề cụ thể: **Tuổi chờ** làm thời gian chờ trở thành số,
nên nó tăng lên và gây khó chịu, đó là mục đích. **Nếu trượt thì** buộc lượng hoá hậu quả trước khi nó
xảy ra, nên cuộc thảo luận escalation sau này có dữ liệu. **Ngày escalation** đặt trước quyết định leo
cấp, nên nó không phụ thuộc vào việc bạn có đủ dũng cảm vào ngày đó hay không. Và dòng D-06 — phụ thuộc
mà *bạn* là bên nợ — là dòng làm register này trở thành tài liệu hai chiều, thứ khiến team khác cũng muốn
dùng nó thay vì coi nó là công cụ tố cáo.

**Bước 5 — Cơ chế theo dõi liên team.** Ba lớp, và điểm chung là mỗi lớp đều rẻ:

| Lớp | Nhịp | Ai | Nội dung | Điều không được làm |
|---|---|---|---|---|
| Board có cột "chờ team khác" với tuổi | Liên tục | Tech Lead | Mỗi thẻ chờ hiện số ngày chờ; quá 5 ngày đổi màu | Không để thẻ bị block nằm trong "In Progress" |
| Dependency sync liên team | 20 phút/tuần, cố định | Các Tech Lead + 1 PO | Chỉ đi qua register: dòng nào đổi, dòng nào tới ngày escalation. Không báo cáo trạng thái chung | Không biến thành họp cập nhật tiến độ 60 phút |
| Rà soát cấp EM | 2 tuần/lần | EM các team | Dòng nào đã escalate, mẫu lặp nào cần sửa ở tầng cấu trúc | Không dùng để đánh giá team nào "hay chặn người khác" |

Quy tắc "hai ngày trước hạn": người phụ thuộc chủ động hỏi lại hai ngày trước ngày cam kết, và câu hỏi
phải cụ thể — "đến chiều mai anh có gửi được file OpenAPI không, hay em nên bật phương án tạm?" Lý do
thiết kế câu hỏi như vậy: nó cho phép người kia nói không mà không mất mặt, và nó gắn câu trả lời với
một hành động của bạn, nên nó không thể được trả lời bằng "đang làm".

**Bước 6 — Bối cảnh ODC: phụ thuộc vào quyết định và môi trường của khách.** Đây là dạng phụ thuộc phổ
biến nhất và bị quản lý tệ nhất trong outsourcing, vì nó có ba đặc điểm khó: bạn không có thẩm quyền,
người bên khách thường không nhận thức được mình đang là đường tới hạn, và việc nêu ra có thể bị đọc là
phàn nàn về khách hàng.

1. Lập một **decision log** riêng, tách khỏi issue tracker: mỗi dòng là một câu hỏi cần khách quyết,
   có ngày hỏi, ngày cần trả lời, người bên khách, và cột "chi phí mỗi ngày chậm" tính bằng người-ngày
   bị treo. Cột cuối là cột làm thay đổi hành vi, vì nó dịch việc chờ thành tiền.
2. Đặt **SLA quyết định** vào thoả thuận vận hành từ đầu dự án, không đợi tới lúc có vấn đề: "câu hỏi
   chặn tiến độ được trả lời trong 3 ngày làm việc; nếu không, team chuyển sang giả định mặc định đã
   ghi và chi phí làm lại thuộc phạm vi change request."
3. Báo cáo tuần cho khách có một mục cố định, đặt ở đầu chứ không ở cuối: **"Chúng tôi đang chờ gì từ
   phía các anh/chị"** — kèm ngày và số người-ngày bị ảnh hưởng. Khi mục này xuất hiện đều đặn từ tuần
   đầu, nó là thông tin; khi nó chỉ xuất hiện lúc dự án trễ, nó là lời buộc tội.
4. Với môi trường: không bao giờ để việc "chờ môi trường khách" nằm trên đường tới hạn mà không có phương
   án nội bộ. Yêu cầu môi trường ở tuần đầu tiên của dự án, kể cả khi chưa cần, vì thời gian cấp môi
   trường trong doanh nghiệp lớn thường là 3–6 tuần và hoàn toàn không tương quan với mức độ gấp của bạn.

**Bước 7 — Phụ thuộc bên thứ ba không kiểm soát được SLA.** Cổng thanh toán, đối tác vận chuyển, dịch vụ
KYC, nhà cung cấp SMS OTP. Nguyên tắc: **xử lý chúng như rủi ro, không như một dòng trong kế hoạch.**
Cụ thể bốn việc: (a) lấy sandbox trước khi ước lượng, không sau — chi phí tích hợp thật chỉ biết được sau
khi gọi được một lần; (b) mọi ước lượng liên quan tích hợp bên thứ ba lấy từ lớp tham chiếu (chủ đề 4)
với P90, không P50; (c) thiết kế có đường suy giảm: hàng đợi retry, chế độ chờ, hoặc nhà cung cấp thứ hai
cho đường quan trọng; (d) đưa vào hợp đồng thương mại các mốc kỹ thuật cụ thể — ngày cấp sandbox, ngày
cấp credential production, người liên hệ kỹ thuật có tên — vì đây là những thứ chỉ có đòn bẩy ở giai đoạn
đàm phán và không còn đòn bẩy nào sau khi ký.

### Trade-off

**Tự làm phần mình cần vs chờ team sở hữu.** Đây là quyết định xuất hiện hàng tuần và thường được quyết
bằng cảm tính về sự lịch sự.

| | Tự làm (chấp nhận trùng lặp) | Chờ team sở hữu |
|---|---|---|
| Lead time | Ngắn, do bạn kiểm soát | Dài, do hàng đợi của họ |
| Chi phí tức thời | Thời gian của team bạn | Thời gian chờ + chi phí chuyển ngữ cảnh khi có hàng |
| Chi phí dài hạn | Hai bản cần đồng bộ mãi; phân kỳ hành vi | Không có trùng lặp; ranh giới sở hữu rõ |
| Rủi ro nghiêm trọng | Hai bản logic tiền hoặc phân quyền lệch nhau — loại lỗi khó phát hiện và đắt | Team khác trở thành ràng buộc lặp lại của bạn |
| Nghiêng về khi | Phần cần dùng nhỏ, ổn định, không phải logic tiền/bảo mật; hoặc bạn giao lại được cho họ sau và ghi rõ nợ | Phần đó là core domain của họ; có tính đúng đắn phải duy nhất; hoặc bạn sẽ cần nó lặp lại nhiều lần |

Một cách trung gian đáng dùng và thường bị bỏ qua: bạn tự làm nhưng làm *trong repo của họ*, theo chuẩn
của họ, và họ review. Bạn trả bằng thời gian của mình, họ trả bằng thời gian review — thường rẻ hơn nhiều
lần thời gian implement — và tổ chức không sinh ra bản trùng lặp. Điều kiện để cách này hoạt động: họ
phải cho phép, và code base phải đủ tốt để người ngoài đóng góp được. Nếu điều kiện thứ hai không đạt,
đó là một thông tin quan trọng về nợ kỹ thuật.

**Contract-first với mock vs chờ giao diện thật.** Được: song song hoá, lead time ngắn hơn nhiều. Mất:
chi phí duy trì mock, và rủi ro contract sai — khi hàng thật đến, hành vi thật khác mock ở những chi tiết
không ai nghĩ tới (thứ tự webhook, mã lỗi không có trong tài liệu, hành vi khi timeout). Chi phí sửa lệch
này thực tế thường là 15–30% chi phí tích hợp (số minh hoạ, nhưng đúng theo hướng: nó không nhỏ). Nghiêng
về contract-first khi: miền nghiệp vụ đã hiểu rõ, có contract test tự động hai chiều, và độ trễ của hàng
thật trên 1 tuần. Nghiêng về chờ khi: giao diện đang được thiết kế cùng lúc với việc học miền nghiệp vụ
(contract sẽ đổi ba lần), hoặc hàng thật chỉ trễ 2–3 ngày.

**Đồng bộ chặt vs tự chủ.** Thêm cơ chế đồng bộ giảm lệch nhưng tiêu capacity và làm chậm quyết định
trong team. Bảng ngắn: 2 team, 1 phụ thuộc → một tin nhắn là đủ, thêm họp là lãng phí. 6 team, 15 cặp →
cần một register và một nhịp tuần. 15 team → cần cả cấu trúc (đổi ranh giới team, platform team có SLA
công bố), vì không có lượng họp nào giải được 105 cặp. Ngưỡng nhận biết bạn đang ở vùng cần đổi cấu trúc
thay vì đổi quy trình: khi thời lượng họp phối hợp của một Tech Lead vượt khoảng 8 giờ một tuần.

### Real-world Scenarios

**Tình huống A — ODC: 21 ngày chờ quyết định của khách, và hai cách nói.**

Bối cảnh: ODC 25 người ở Đà Nẵng, khách hàng Nhật, dự án thay thế hệ thống quản lý kho. Duy (Tech Lead
phía Việt Nam) đã gửi ba câu hỏi về quy tắc kiểm kê từ 5/3. Đến 26/3 chưa có câu trả lời. Ba engineer
đang làm việc khác không quan trọng để không bị treo, tức là dự án mất khoảng 12 người-ngày. Buổi họp
tuần với khách diễn ra sáng 27/3.

*Phiên bản nói sai:*

> **Duy:** Về tiến độ thì tuần này bọn em làm được phần nhập kho, còn phần kiểm kê thì vẫn đang chờ ạ.
>
> **Khách (ông S., PM phía Nhật):** Chờ gì?
>
> **Duy:** Bọn em có gửi mấy câu hỏi rồi ạ, chưa thấy trả lời. Không sao ạ, bọn em vẫn làm việc khác
> được. Khi nào anh trả lời thì bọn em làm tiếp.
>
> **Khách:** À vâng, tôi sẽ hỏi lại bộ phận nghiệp vụ. Tiến độ chung vẫn ổn chứ?
>
> **Duy:** Vẫn ổn ạ.

Bốn lỗi trong đoạn ngắn này, và mỗi lỗi có hậu quả cụ thể. Một: "vẫn đang chờ" không có tuổi, không có
số, nên nó không tạo cảm giác cấp thiết nào — thông tin bị mã hoá dưới dạng không thể hành động. Hai:
"không sao ạ" xoá luôn hậu quả, khiến việc trả lời câu hỏi trở thành ưu tiên thấp một cách hợp lý ở phía
khách. Ba: "tiến độ chung vẫn ổn" là một phát ngôn sai sẽ được ghi vào biên bản, và ba tuần sau khi dự
án trễ, biên bản này sẽ được dùng để chứng minh rằng bên Việt Nam đã không cảnh báo. Bốn: không có câu
hỏi nào có ngày, nên không có gì để theo dõi ở tuần sau.

*Phiên bản nói đúng:*

> **Duy:** Trước khi báo tiến độ, em có một mục cần anh quyết. Ba câu hỏi về quy tắc kiểm kê em gửi ngày
> 5/3, đến nay là 21 ngày chưa có câu trả lời. Hệ quả tính được: module kiểm kê chưa bắt đầu được, và
> bọn em đã chuyển ba người sang việc khác, nên đã mất khoảng 12 người-ngày cho việc chuyển đổi và làm
> việc không nằm trên đường tới hạn.
>
> **Khách:** Tôi hiểu. Bộ phận nghiệp vụ đang bận đợt chốt sổ.
>
> **Duy:** Vâng, em hiểu ạ. Em đề nghị hai việc. Thứ nhất, trong ba câu hỏi thì chỉ câu số một là chặn
> — quy tắc làm tròn khi số lượng lệch. Hai câu còn lại em có thể tự đặt giả định và ghi lại, anh xác
> nhận sau. Như vậy anh chỉ cần một câu trả lời, không phải ba.
>
> **Khách:** Được, câu một thì tôi hỏi được trong tuần này.
>
> **Duy:** Cho em xin một ngày cụ thể để em đưa vào bảng theo dõi: 31/3 chiều được không ạ?
>
> **Khách:** Được.
>
> **Duy:** Em ghi 31/3. Và em nói trước phần này để không có bất ngờ: nếu đến 31/3 chưa có, bọn em sẽ
> làm theo giả định là làm tròn xuống và ghi vào decision log; nếu sau này nghiệp vụ quyết khác, phần
> sửa lại khoảng 3 ngày công và em sẽ đưa vào change request. Em không nói vậy để tạo áp lực, em nói để
> anh biết chi phí của mỗi phương án.
>
> **Khách:** Hợp lý. Cứ làm vậy.
>
> **Duy:** Việc thứ hai: từ tuần sau, báo cáo tuần của bọn em sẽ có một bảng ở đầu tên là "đang chờ phía
> khách", với ngày hỏi và số người-ngày bị ảnh hưởng. Không phải để phàn nàn — để cả hai bên thấy cùng
> một con số. Nếu bảng đó trống thì đó là tuần tốt.

Bảng mổ xẻ:

| Yếu tố | Bản sai | Bản đúng | Vì sao khác biệt tạo ra kết quả khác |
|---|---|---|---|
| Vị trí trong buổi họp | Cuối, sau tiến độ | Đầu, trước tiến độ | Thứ tự truyền tín hiệu mức độ quan trọng; mục cuối bị nghe như thủ tục |
| Lượng hoá | "đang chờ" | "21 ngày, 12 người-ngày" | Con số tạo được hành động; tính từ thì không |
| Hậu quả | "không sao ạ" | Nêu rõ hậu quả và ai chịu | Che hậu quả làm phía kia ra quyết định ưu tiên trên thông tin sai |
| Giảm tải cho người quyết | Ba câu hỏi như nhau | Tách một câu chặn khỏi hai câu không chặn | Người bận sẽ hoãn một khối ba câu, nhưng trả lời được một câu |
| Ngày | Không có | Xin một ngày cụ thể và ghi lại | Không có ngày thì không có gì để theo dõi và không có điểm escalation |
| Đường đi nếu trượt | Không có | Giả định mặc định + chi phí sửa + change request | Chuyển bế tắc thành một quyết định có giá, và bảo vệ được phạm vi |
| Cơ chế lâu dài | Không có | Mục cố định trong báo cáo tuần | Đặt cơ chế lúc bình thường, để lúc có vấn đề nó không bị đọc là buộc tội |

**Tình huống B — Đối tác vận chuyển đổi API trước mùa cao điểm.**

Bối cảnh: e-commerce, tháng 10, chuẩn bị cho 11/11. Đối tác vận chuyển lớn nhất thông báo API tạo vận
đơn sẽ đổi version, bản cũ ngừng hoạt động ngày 5/11 — thông báo đến ngày 14/10, tức 22 ngày. Team
Logistics đang có toàn bộ capacity dành cho việc chịu tải mùa cao điểm.

Cách xử lý: việc đầu tiên không phải lập kế hoạch code, mà là xác minh giả định — gọi cho người liên hệ
kỹ thuật để xác nhận ngày tắt bản cũ là cứng hay mềm, và xin sandbox ngay trong ngày. Hai thông tin này
quyết định toàn bộ phần còn lại, và cả hai đều nằm ngoài hệ thống của bạn. Kết quả: ngày 5/11 là cứng
(bên họ đã lên kế hoạch tắt hạ tầng), nhưng họ có chế độ tương thích ngược cho khách hàng lớn tới 30/11
nếu đăng ký trước 20/10 — một thông tin không có trong email thông báo và chỉ có được vì đã gọi. Đăng ký
ngay, và thời hạn thật chuyển từ 22 ngày thành 47 ngày, ra ngoài mùa cao điểm.

Bài học có thể tổng quát hoá: với phụ thuộc bên thứ ba, thông tin quan trọng nhất thường không nằm trong
tài liệu chính thức mà nằm ở người liên hệ kỹ thuật. Việc có tên và số điện thoại của người đó, từ lúc
ký hợp đồng chứ không lúc khủng hoảng, là một biện pháp giảm rủi ro có chi phí bằng không. Và một điểm
về thiết kế: sau lần này, team bọc mọi tích hợp vận chuyển sau một lớp adapter nội bộ, không phải vì
kiến trúc đẹp mà vì đã đo được rằng đối tác đổi API trung bình 1,5 lần một năm và mỗi lần trước đây phải
sửa ở 14 chỗ.

**Tình huống C — Team platform ở mức tận dụng 95%.**

Bốn team đều chờ team Platform 5 người. Hàng đợi yêu cầu có 23 item. Mỗi item cần trung bình 2 ngày công.
Thời gian chờ trung bình quan sát được: 4 tuần. Phản ứng mặc định của tổ chức là "Platform cần làm nhanh
hơn" hoặc "tuyển thêm cho Platform". Cả hai đều bỏ qua nguyên lý 3: ở mức tận dụng 95%, thời gian chờ
gấp khoảng 19 lần thời gian xử lý, nên vấn đề không phải tốc độ xử lý.

Ba can thiệp theo thứ tự hiệu quả trên chi phí: (1) Platform công bố dành 30% capacity cho yêu cầu liên
team và giữ nó như một bucket được bảo vệ (chủ đề 5) — mức tận dụng cho phần công việc của chính họ giảm,
thời gian chờ giảm mạnh hơn nhiều so với tỉ lệ capacity bị chuyển; (2) chuyển từ mô hình "làm hộ" sang
mô hình self-service cho ba loại yêu cầu phổ biến nhất chiếm 60% hàng đợi — đây là chi phí một lần đổi
lấy việc loại bỏ vĩnh viễn phần lớn phụ thuộc; (3) công bố hàng đợi và thời gian chờ dự kiến để các team
tự quyết định có nên chờ hay tự làm — nghe nhỏ nhưng nó chuyển quyết định về đúng người có thông tin về
giá trị. Việc *không* nên làm đầu tiên: thêm một buổi họp ưu tiên hàng tuần giữa bốn team để tranh nhau
thứ tự. Nó không tăng thông lượng, chỉ chuyển việc xếp hàng thành việc đàm phán và tiêu thêm thời gian
của chính người đang là ràng buộc.

### Best Practices

**Ghi phụ thuộc ở buổi planning, không ở lúc bị chặn.** Cơ chế: khi bị chặn, bạn đã mất toàn bộ thời gian
lẽ ra dùng để đàm phán thứ tự hoặc dựng mock. Chi phí xử lý cùng một phụ thuộc tăng theo thời điểm phát
hiện, và độ dốc rất lớn ở giai đoạn cuối.

**Mỗi phụ thuộc có tên một người, không tên một team.** "Chờ team Payment" không có ai trả lời được; "chờ
Tuấn" thì có. Tên team là cách né việc phải đi hỏi ai đó cụ thể, và nó khiến phụ thuộc không có chủ ở cả
hai đầu.

**Xét chiến lược loại bỏ trước, và ghi lại lý do nếu không loại bỏ được.** Vì mặc định của mọi tổ chức
là chấp nhận và theo dõi, chỉ có một bước ghi chép bắt buộc mới thay đổi được mặc định đó.

**Đặt buffer ở điểm hợp lưu, không rải đều.** Ba nhánh mỗi nhánh cộng 2 ngày đệm cho kết quả kém hơn một
đệm 4 ngày đặt sau điểm tích hợp, vì đệm phân tán bị tiêu bởi student syndrome trong khi đệm tập trung
được theo dõi.

**Yêu cầu môi trường, quyền truy cập, và lịch review bảo mật ở tuần đầu tiên.** Ba loại này có lead time
dài, gần như không co giãn được theo mức độ gấp, và chi phí xin sớm gần bằng không.

**Trả nợ phụ thuộc của mình đúng hạn, kể cả khi bên kia không nhắc.** Đây không phải lời khuyên về đạo
đức mà là về cơ chế: dependency register chỉ hoạt động nếu nó hai chiều, và nó chỉ hai chiều nếu team
của bạn cũng là bên bị theo dõi. Ngoài ra, năng lực đàm phán thứ tự (chiến lược 3) là một dạng tín dụng,
và nó được tích luỹ bằng những lần bạn giao đúng cho người khác.

**Escalate theo ngày đã định trước, không theo cảm giác.** Cơ chế: quyết định leo cấp bị chi phối bởi
chi phí xã hội tức thời, còn lợi ích của nó ở tương lai và phân tán — nên nếu để tuỳ cảm giác, nó luôn
xảy ra muộn. Đặt ngày trước biến nó thành một việc theo lịch, và tách nó khỏi câu hỏi "mình có đang làm
căng quá không".

**Nói với bên bị escalate trước khi escalate.** Một câu: "nếu đến thứ Năm chưa có, mình phải báo lên Hà,
mình nói trước để anh không bị bất ngờ." Nó giữ được quan hệ, và trong bối cảnh Việt Nam nơi việc leo cấp
dễ bị đọc là mách, bước này là điều kiện để cơ chế escalation sống được quá ba lần dùng.

### Anti-patterns

**Giả định team khác xong đúng hạn.** Hình dạng: kế hoạch của bạn có dòng "23/4: nhận API từ Payment",
được vẽ như một sự thật, và mọi việc sau đó xếp lịch dựa trên nó. Cơ chế phá hệ thống: bạn đã nhập một
biến ngẫu nhiên vào kế hoạch dưới dạng một hằng số, nên toàn bộ phần sau của kế hoạch có xác suất đúng
bằng xác suất của biến đó — và như bảng p^k ở trên, với vài phụ thuộc thì con số này sụp rất nhanh. Tệ
hơn, khi nó trượt, bạn không có phương án nào vì bạn chưa từng nghĩ tới khả năng đó. Dấu hiệu sớm: trong
kế hoạch, các dòng phụ thuộc không có phương án dự phòng; hoặc không có dòng nào ghi "nếu trượt thì";
hoặc bạn không biết ngày đó do ai cam kết và cam kết ở đâu.

**Giải quyết dependency bằng cách thêm họp đồng bộ.** Hình dạng: phát hiện nhiều phụ thuộc, lập một buổi
"cross-team sync" 60 phút mỗi tuần, rồi một buổi nữa cho nhóm kỹ thuật, rồi một Slack channel với daily
update. Cơ chế: họp làm tăng *chất lượng thông tin* về phụ thuộc nhưng không giảm *số lượng* phụ thuộc,
trong khi đòn bẩy thật nằm ở số lượng (nguyên lý 2). Đồng thời nó tiêu capacity của chính những người
đang là ràng buộc — Tech Lead của team bị phụ thuộc nhiều nhất sẽ ngồi trong nhiều buổi nhất, làm thời
gian xử lý của họ tăng, làm hàng đợi dài thêm. Đây là một vòng phản hồi dương: càng nhiều phụ thuộc,
càng nhiều họp, càng ít capacity, càng nhiều chờ. Dấu hiệu sớm: số buổi họp liên team tăng mà tuổi trung
bình của thẻ bị chặn không giảm; hoặc một Tech Lead có trên 12 giờ họp một tuần.

**Escalation quá muộn.** Hình dạng: phụ thuộc trượt hạn ngày 20/3, được nhắc lại nhẹ nhàng hàng tuần,
và được báo lên cấp trên ngày 5/5 khi đã không còn phương án nào. Cơ chế: giá trị của escalation nằm ở
số lựa chọn còn lại tại thời điểm leo cấp, và số lựa chọn đó giảm gần như tuyến tính theo thời gian còn
lại — một EM biết trước sáu tuần có thể đổi ưu tiên, đổi người, đàm phán phạm vi, hoặc mua giải pháp;
cùng EM đó biết trước ba ngày chỉ có thể chọn giữa trễ và cắt chất lượng. Trong bối cảnh Việt Nam, cơ
chế trì hoãn thường không phải sự cẩu thả mà là chi phí xã hội: leo cấp bị hiểu là tố cáo đồng nghiệp,
và người leo cấp bị đánh giá là không tự xử lý được việc của mình. Vì vậy cách sửa phải ở tầng cơ chế —
ngày escalation định trước, và người leo cấp không bị coi là người mang tin xấu vì việc đó có trong quy
trình. Dấu hiệu sớm: có thẻ bị chặn trên 10 ngày mà chưa có ai ngoài team biết; hoặc mọi lần escalation
trong sáu tháng qua đều diễn ra trong hai tuần cuối của một dự án.

**Coi mọi phụ thuộc là cứng.** Cơ chế: nó chuyển bạn từ vai người ra quyết định sang vai người chờ, và
xoá mất toàn bộ không gian của chiến lược 1 và 2. Dấu hiệu sớm: khi được hỏi "nếu thứ này không bao giờ
đến thì sao", câu trả lời là "thì không làm được" mà không kèm phân tích.

**Dependency register tồn tại nhưng không ai cập nhật.** Cơ chế: một register cũ tệ hơn không có register,
vì nó tạo cảm giác đã kiểm soát trong khi thông tin đã sai. Dấu hiệu sớm: cột "tuổi chờ" được điền bằng
tay và không đổi trong hai tuần; hoặc lần cập nhật cuối cách đây trên 10 ngày.

**Dùng dependency register để đo xem team nào hay chặn người khác.** Cơ chế: khoảnh khắc register trở
thành công cụ đánh giá, các team sẽ ngừng ghi phụ thuộc vào nó và chuyển sang thoả thuận riêng — bạn mất
đúng thứ dữ liệu mà bạn cần. Dấu hiệu sớm: một slide xếp hạng team theo số lần gây chặn; hoặc một Tech
Lead từ chối ghi một dòng vào register vì "để anh nói riêng với Tuấn".

### Khi nào KHÔNG nên áp dụng

**Khi bạn chỉ có một team và mọi thứ trong tầm tay.** Với một team sở hữu toàn bộ luồng từ UI tới database
và không phụ thuộc ai, bộ máy này là chi phí thuần. Cái vẫn cần giữ, và đây là điều dễ bỏ sót: hai loại
phụ thuộc vẫn tồn tại ngay cả ở team đơn lẻ — phụ thuộc vào *quyết định* (nghiệp vụ, pháp chế, thiết kế)
và phụ thuộc vào *người duy nhất biết một thứ*. Hai loại đó nên được theo dõi bằng một danh sách năm dòng,
không cần register đầy đủ.

**Khi phụ thuộc chỉ mất một đến hai ngày và bên kia ở cùng phòng.** Chi phí của một dòng register cộng
một lần rà soát tuần vượt lợi ích. Ngưỡng thực dụng: ghi vào register nếu thời gian chờ dự kiến trên 3
ngày, hoặc nếu bên kia thuộc tổ chức khác, hoặc nếu việc đó nằm trên đường tới hạn. Dưới ngưỡng đó, một
tin nhắn là công cụ đúng.

**Khi vấn đề thật là ranh giới tổ chức và bạn đang dùng quy trình để che nó.** Nếu cùng một phụ thuộc
xuất hiện trong phần lớn công việc của bạn — ví dụ mọi tính năng đều cần team Platform sửa schema — thì
việc quản lý phụ thuộc tốt hơn chỉ làm cho một cấu trúc sai chạy được lâu hơn, và nó có một cái giá ẩn:
nó xoá đi tín hiệu đau lẽ ra dẫn tới việc sửa cấu trúc. Ở đây, việc đúng là leo lên tầng thiết kế tổ chức
(chương 09): chuyển quyền sở hữu, tách service, hoặc chuyển Platform sang mô hình self-service. Dấu hiệu
định lượng để phân biệt: nếu trên 50% item của bạn có cùng một phụ thuộc, đó là vấn đề cấu trúc.

**Khi contract-first sẽ tạo ra một contract chắc chắn sai.** Nếu miền nghiệp vụ đang được khám phá — hai
bên còn chưa thống nhất khái niệm "hoàn tiền một phần" nghĩa là gì — thì đóng băng contract sớm sẽ khoá
một mô hình sai vào cả hai code base, và chi phí tháo ra lớn hơn nhiều chi phí chờ. Trong điều kiện này,
cái nên làm là ngược lại: một người của hai team làm chung một spike 3–5 ngày trên cùng một nhánh để
tìm ra mô hình đúng, rồi mới chốt contract. Bạn đổi song song hoá lấy tính đúng của mô hình, và đó là
đánh đổi đúng khi mô hình còn chưa chắc.

**Khi bạn không có bất kỳ đòn bẩy nào và việc làm rõ chỉ tạo xung đột.** Có tình huống thật: một phụ
thuộc vào một bộ phận có quyền lực chính trị lớn hơn bạn, không có SLA, không có ai bảo trợ bạn ở cấp
trên. Trong điều kiện đó, ghi tên họ vào một register công khai với cột "tuổi chờ" có thể tạo ra xung
đột mà bạn không thắng được, và làm mất luôn kênh không chính thức đang hoạt động. Cách xử lý thực tế:
giữ bản ghi cho riêng mình và cho EM của bạn (để có dữ liệu khi cần), làm việc qua quan hệ cá nhân, và
tập trung toàn bộ năng lượng vào chiến lược 1 — thiết kế lại để không cần họ. Đồng thời nói rõ với EM
rằng rủi ro này không thể quản lý bằng công cụ delivery, nó cần được xử lý ở tầng tổ chức. Không nhận
diện được điều kiện biên này là cách một Tech Lead giỏi tự đưa mình vào một cuộc chiến sai.

---

## 7. Risk Management trong delivery

### Problem Statement

Quay lại dự án ở đầu chương: "Tính năng thanh toán trả góp", chốt launch 30/11, báo 85% suốt hai tuần,
đến 28/11 mới nói cần thêm bốn tuần. Sau đó Hà (EM) làm một postmortem về *quá trình*, không về sản phẩm.
Câu hỏi duy nhất của buổi đó: **thông tin "dự án này sẽ trễ" tồn tại trong đầu ai, từ ngày nào?**

| Thông tin | Ai biết | Biết từ ngày | Nói ra ngày | Nói với ai |
|---|---|---|---|---|
| Sandbox đối tác trả góp chưa có ngày cấp | Minh, Trang | 08/09 | 28/11 | Không ai, tới cuối |
| Khoa là người duy nhất biết hệ thống đối soát, tháng 11 là tháng chốt sổ | Hà | 08/09 | Không nói | — |
| Luồng xử lý giao dịch lỗi phức tạp hơn dự tính khoảng 2 tuần | Khoa | 02/10 | 15/10, trong standup | Chỉ team nghe, không ghi lại |
| Quy tắc tính lãi chưa được pháp chế xác nhận | Trang | 20/10 | 24/11 | Minh |
| Ba story lớn nhất đều "gần xong" từ 05/11 | Cả team | 05/11 | — | Không ai đặt câu hỏi |

Kết luận của buổi đó, và nó là kết luận thường gặp: **không có rủi ro nào trong danh sách trên là rủi ro
mới, bất ngờ, hoặc không lường được.** Tất cả đều đã được biết bởi một người cụ thể, ở một ngày cụ thể,
trung bình trước ngày công bố trễ khoảng 6 tuần. Vấn đề không phải năng lực dự báo. Vấn đề là quãng đường
từ "một người biết" đến "tổ chức biết và hành động" — và quãng đường đó không có đường đi.

Hậu quả quan sát được khi thiếu cơ chế này, ngoài việc trễ: stakeholder mất khả năng ứng phó (nếu biết
trước 6 tuần, Sơn có thể lùi lịch truyền thông, hoặc chấp nhận bản rút gọn, hoặc đàm phán lại với đối
tác — biết trước 2 ngày thì không còn lựa chọn nào); uy tín của Tech Lead bị đánh giá theo lần công bố
cuối cùng chứ không theo chất lượng công việc; và tổ chức học được một bài học sai — rằng cần kiểm soát
chặt hơn — nên quý sau thêm báo cáo hàng ngày, làm tăng chi phí mà không sửa cơ chế im lặng.

### First Principles

**Nguyên lý 1 — Phần lớn rủi ro delivery là rủi ro đã biết nhưng chưa được nói ra.** Điều này khác căn
bản với rủi ro kỹ thuật ở chương 04, nơi phần lớn công việc là *phát hiện* thứ chưa ai nghĩ tới. Ở delivery,
tỉ lệ nghịch: nếu bạn đi phỏng vấn riêng từng người trong một dự án đang trễ và hỏi "theo em, cái gì sẽ
làm dự án này trễ", bạn thường nhận được một danh sách gần đủ trước khi bất kỳ điều nào xảy ra. Hệ quả
thực hành rất mạnh: công cụ chính của bạn không phải mô hình dự báo tinh vi, mà là *cơ chế trích xuất
thông tin đã có trong đầu người khác*. Đó là lý do một buổi pre-mortem 45 phút thường cho giá trị cao hơn
một tuần phân tích.

**Nguyên lý 2 — Cơ chế im lặng: người biết không có động lực nói.** Đây là phần cần nói thẳng, vì nó là
nguyên nhân gốc và nó là một bài toán kinh tế học chứ không phải một vấn đề tính cách.

Xét Khoa, người biết từ 02/10 rằng luồng xử lý lỗi sẽ mất thêm hai tuần. Bảng lợi ích và chi phí của việc
Khoa nói to điều đó, tính từ góc nhìn của Khoa:

| | Nếu Khoa nói ra ngày 02/10 | Nếu Khoa im |
|---|---|---|
| Chi phí tức thời | Bị hỏi "sao lúc ước lượng không thấy", có thể bị coi là chậm | Không có |
| Ai gánh chi phí | Khoa, ngay lập tức, một mình | Không ai, lúc này |
| Lợi ích | Dự án có 8 tuần để ứng phó | Có thể Khoa cày bù được, và không ai biết |
| Ai nhận lợi ích | Tổ chức, phân tán, ở tương lai | Khoa (tránh được tình huống khó xử) |
| Xác suất bị quy trách nhiệm cá nhân | Cao (anh là người mang tin xấu) | Thấp — nếu trễ, đó là "dự án trễ", trách nhiệm tập thể |

Cấu trúc này có một tên trong tâm lý học tổ chức là MUM effect: người ta tránh truyền thông điệp không
mong muốn, đặc biệt lên trên. Nhưng điểm quan trọng để hành động là: **chi phí tập trung và tức thời cho
người nói, lợi ích phân tán và trì hoãn cho tổ chức.** Bất kỳ cơ chế nào không thay đổi bảng này sẽ không
thay đổi hành vi, bất kể bạn nói bao nhiêu lần rằng "hãy báo sớm". Ba cách thực sự đổi được bảng: (a) làm
việc báo sớm trở thành *nghĩa vụ theo lịch* thay vì một hành động tự nguyện — khi nó là một dòng trong
biểu mẫu tuần, việc điền nó không còn là hành vi tố cáo bản thân; (b) tách rõ ràng việc báo rủi ro khỏi
việc đánh giá con người, và chứng minh bằng hành vi lặp lại chứ bằng lời nói (chương 03); (c) làm chi phí
của việc *im* trở nên hiện hữu — ví dụ một quy tắc rằng độ trễ được báo muộn hơn 2 tuần so với lúc biết
sẽ được nêu trong postmortem như một lỗi cơ chế có tên.

Trong bối cảnh Việt Nam, bảng trên còn lệch thêm về phía im lặng bởi ba yếu tố: việc nói tin xấu về tiến
độ dễ bị đọc là phê phán năng lực của người đã cam kết (thường là cấp trên hoặc người lớn tuổi hơn); khái
niệm giữ mặt làm cho việc nói trong phòng đông người trở nên đắt hơn nhiều so với nói riêng; và cấu trúc
thâm niên làm cho một junior gần như không có đường nói rằng kế hoạch của một senior sai. Hệ quả thiết
kế: kênh báo rủi ro phải có một đường *không công khai* và *không đối đầu* — ví dụ một trường trong biểu
mẫu cập nhật tuần mà chỉ Tech Lead và EM đọc, hoặc câu hỏi cố định trong 1-1 — chứ không chỉ dựa vào việc
mọi người phát biểu trong retro.

**Nguyên lý 3 — Giá trị của một cảnh báo bằng số lựa chọn còn lại tại thời điểm cảnh báo.** Đây là công
thức để trả lời câu "báo sớm để làm gì nếu không chắc". Cảnh báo không có giá trị vì nó chính xác; nó có
giá trị vì nó mở ra hành động. Với một dự án 12 tuần, số lựa chọn của stakeholder theo thời điểm biết tin:

| Biết trước ngày launch | Lựa chọn còn lại |
|---|---|
| 8 tuần | Đổi phạm vi, đổi người, mua giải pháp, lùi lịch truyền thông, đàm phán với đối tác, chấp nhận bản rút gọn |
| 4 tuần | Đổi phạm vi (một phần), lùi lịch truyền thông, chấp nhận bản rút gọn |
| 2 tuần | Chấp nhận bản rút gọn, hoặc trễ |
| 2 ngày | Trễ. Hoặc cắt chất lượng và trễ muộn hơn |

Bảng này là lập luận duy nhất bạn cần khi ai đó nói "để chắc rồi báo". Chắc chắn hơn không mua được gì
nếu không còn lựa chọn nào để chọn.

**Nguyên lý 4 — Trạng thái dự án bị làm hồng dần khi đi lên qua các tầng.** Mỗi tầng có động lực làm mềm
tin xấu một chút: Khoa nói "hơi chậm" thay vì "hai tuần"; Minh nói "có rủi ro nhưng đang xử lý"; Hà nói
"đang theo dõi sát"; đến Sơn thì thành "vẫn ổn". Không ai nói dối, mỗi bước chỉ mất khoảng 20–30% mức
độ nghiêm trọng, nhưng qua ba bước thì tín hiệu gần như bằng không. Hệ quả thiết kế: cần ít nhất một kênh
*không qua trung gian* — người ra quyết định cuối phải đọc được một tài liệu do người gần việc nhất viết,
ít nhất một lần mỗi vài tuần. Đây là lý do bản cập nhật một trang nên do Tech Lead viết và được gửi nguyên
văn, không được biên tập lại bởi tầng trên.

### Mental Model

**Principal-Agent Problem áp vào báo cáo tiến độ.** Người ra quyết định (principal — founder, khách hàng)
không quan sát được công việc; người thực thi (agent — team) quan sát được nhưng lợi ích không hoàn toàn
trùng khớp. Trong cấu trúc này, thông tin không tự chảy; nó chảy khi có cơ chế làm cho việc báo cáo đúng
có lợi hoặc ít nhất không tốn kém. Cách dùng mô hình: mỗi khi bạn thấy mình sắp nói "team cần chủ động
báo cáo hơn", hãy dịch câu đó thành "cơ chế hiện tại làm cho việc báo cáo đúng tốn kém — chỗ nào?" Câu
thứ hai dẫn tới hành động, câu thứ nhất chỉ dẫn tới sự thất vọng lặp lại.

**Leading vs lagging indicator.** Ngày launch trễ là lagging indicator — khi bạn đo được nó, không còn gì
làm được. Cycle time tăng, số thẻ "gần xong" tăng, tỉ lệ defect mở/đóng lớn hơn 1, tuổi thẻ chờ tăng: đó
là leading indicator. Nội dung của risk management trong delivery gần như toàn bộ nằm ở việc chọn 4–6
leading indicator, đặt ngưỡng trước, và cam kết một hành động khi vượt ngưỡng. Đặt ngưỡng *trước* là điểm
then chốt, vì sau khi số liệu xuất hiện, luôn có một cách giải thích hợp lý cho nó.

**Swiss cheese model từ an toàn hàng không.** Không có một lớp bảo vệ nào bắt được mọi thứ; bạn xếp nhiều
lớp mỏng, mỗi lớp có lỗ, và mong các lỗ không thẳng hàng. Áp vào đây: pre-mortem bắt được rủi ro về giả
định; register bắt được rủi ro đã biết nhưng chưa có chủ; ngưỡng số liệu bắt được thứ không ai nói; 1-1
bắt được thứ không ai muốn nói trong phòng đông; và điểm rà soát giữa kỳ bắt được thứ đã đổi. Đừng tìm
một cơ chế hoàn hảo; hãy dựng bốn cơ chế rẻ, không tương quan với nhau.

### Practical Framework

**Bước 1 — Risk register cho delivery.** Đây là tài liệu khác với risk register kỹ thuật ở chương 04.
Register kỹ thuật hỏi "hệ thống có thể hỏng thế nào"; register delivery hỏi "cam kết này có thể không giao
được thế nào". Sáu nhóm, và gần như toàn bộ rủi ro delivery nằm trong sáu nhóm này:

| Nhóm | Rủi ro điển hình | Tín hiệu sớm quan sát được | Ai thường sở hữu |
|---|---|---|---|
| **Scope** | Yêu cầu chưa rõ được coi là đã rõ; scope creep từng chút; tiêu chí chấp nhận chưa viết | Số câu hỏi mở trong story tăng; story bị sửa nội dung sau khi vào sprint > 15% | PO |
| **Người** | Một người là điểm đơn nhất; nghỉ phép chồng với giai đoạn tới hạn; người mới cần onboarding; một senior sắp nghỉ việc | Trên 40% commit của một module đến từ một người; lịch nghỉ chưa được đối chiếu với kế hoạch | EM |
| **Phụ thuộc** | Chờ team khác, chờ khách, chờ bên thứ ba (chủ đề 6) | Tuổi thẻ chờ; phụ thuộc chưa có ngày cam kết | Tech Lead |
| **Môi trường / hạ tầng** | Chưa có môi trường UAT; dữ liệu test không thật; không có cách test tải | Ngày cấp môi trường chưa xác định ở tuần 2 | EM / Platform |
| **Phê duyệt** | Chờ pháp chế, security review, CAB, cửa sổ release, ký của khách | Chưa đặt lịch review khi còn dưới 3 tuần | PO / EM |
| **Chất lượng** | Nợ test dồn về cuối; defect mở tăng; regression chưa tự động | Tỉ lệ defect mở/đóng > 1 trong 2 tuần; test viết sau khi code merge | Tech Lead |

Cách viết một dòng register cho đúng: **"Nếu [điều kiện] thì [hậu quả lượng hoá được]"** — không viết
rủi ro dưới dạng một danh từ. "Rủi ro tích hợp" là vô dụng. "Nếu sandbox đối tác không có trước 20/10
thì phần tích hợp lùi tương ứng, mỗi tuần chậm bằng 1 tuần trễ launch" là dùng được, vì nó nêu điều kiện
kiểm được và hậu quả tính được.

| ID | Nếu... thì... | Nhóm | Xác suất | Tác động (tuần trễ) | Điểm | Chủ | Hành động giảm rủi ro | Tín hiệu theo dõi | Rà soát |
|---|---|---|---|---|---|---|---|---|---|
| R-01 | Nếu sandbox đối tác trả góp không có trước 20/10 thì tích hợp lùi 1 tuần cho mỗi tuần chậm | Phụ thuộc | Cao (0,6) | 1–3 | 9 | Trang | Có văn bản cam kết ngày từ đối tác; dựng mock từ tài liệu để làm song song | Ngày cấp sandbox trong email đối tác | Mỗi tuần |
| R-02 | Nếu Khoa bị hút vào chốt sổ tháng 11 thì phần đối soát mất thêm 1,5 tuần | Người | Cao (0,7) | 1,5 | 8 | Hà | Đưa Quân vào pair với Khoa từ tuần này; viết runbook đối soát | Số giờ Khoa ghi cho việc ngoài dự án | Mỗi tuần |
| R-03 | Nếu pháp chế không xác nhận quy tắc lãi trước 05/11 thì không được phát hành dù code xong | Phê duyệt | Trung bình (0,4) | 2+ | 8 | Trang | Đặt lịch với pháp chế ngay; gửi bản quy tắc để review trước | Đã có lịch họp hay chưa | Mỗi tuần |
| R-04 | Nếu luồng giao dịch lỗi phức tạp hơn dự tính thì +2 tuần | Scope | Trung bình (0,5) | 2 | 7 | Minh | Spike 3 ngày tuần này để chốt độ phức tạp; nếu vượt, cắt phạm vi v1 | Kết quả spike ngày 10/10 | Sau spike |
| R-05 | Nếu regression chưa tự động hoá thì mỗi lần release tốn 2 ngày kiểm thử tay | Chất lượng | Cao (0,8) | 0,5 mỗi release | 5 | Minh | Tự động hoá 12 ca chính, không phải toàn bộ | Số ca tự động / tổng ca chính | 2 tuần |

Ba quy tắc vận hành, và chúng quyết định register này sống hay chết. Một: **tối đa 8–10 dòng.** Một
register 40 dòng không được đọc, và việc không được đọc còn tệ hơn không tồn tại vì nó tạo cảm giác đã
kiểm soát. Hai: **mỗi dòng có một tên người, không tên một nhóm** — chủ của rủi ro là người có thể thực
hiện hành động giảm rủi ro, không phải người sẽ bị thiệt. Ba: **mỗi dòng có một tín hiệu quan sát được**,
để việc rà soát tuần là kiểm tra dữ liệu chứ không phải hỏi cảm giác. Nếu một dòng không có tín hiệu quan
sát được, nó là một mối lo, chưa phải một rủi ro được quản lý.

**Bước 2 — Đặt ngưỡng cho tín hiệu sớm, trước khi cần dùng.** Đây là bước biến rủi ro từ chủ đề thảo luận
thành cơ chế. Ngưỡng dưới đây là số minh hoạ và nên được hiệu chỉnh theo dữ liệu của bạn, nhưng độ lớn
của chúng có cơ sở.

| Tín hiệu | Cách đo | Ngưỡng báo động | Nó nói lên điều gì | Hành động bắt buộc khi vượt |
|---|---|---|---|---|
| Burnup lệch so với đường cần thiết | Điểm hoặc số item hoàn thành tích luỹ vs đường tới đích | Lệch > 15% hai sprint liên tiếp | Không phải nhiễu; tốc độ thật khác kế hoạch | Cập nhật dự báo và báo stakeholder trong 3 ngày, kèm 3 phương án (chủ đề 8) |
| Số item "gần xong" | Đếm item ở trạng thái In Progress hoặc Review quá 1,5 lần cycle time trung vị | Tăng 2 tuần liền, hoặc > 30% WIP | Việc đang bị chặn hoặc bị đánh giá thấp độ phức tạp; đây là dấu hiệu sớm nhất và đáng tin nhất | Đi qua từng item, hỏi "còn thiếu đúng cái gì để merge"; không hỏi "bao nhiêu phần trăm" |
| Cùng một % hai lần báo cáo liền | So sánh báo cáo tuần | 2 tuần liền cùng con số | % đang được suy ra từ cảm giác, không từ việc đã đóng | Chuyển sang đo bằng số item đã đóng và số item còn lại; bỏ đơn vị % |
| Defect mở vs đóng | Đếm theo tuần | Mở/đóng > 1 trong 2 tuần liền | Chất lượng đang giảm nhanh hơn khả năng sửa; ngày kết thúc không xác định được | Dừng nhận việc mới vào sprint, ưu tiên đóng; xem lại định nghĩa Done |
| Tỉ lệ việc phát sinh ngoài kế hoạch | Người-ngày ngoài kế hoạch / tổng | > 30% một sprint, hoặc > 20% hai sprint liền | Kế hoạch không còn mô tả thực tế | Lập lại cam kết công khai với phạm vi mới, không âm thầm trượt |
| Tuổi thẻ bị chặn | Ngày ở trạng thái chờ | Bất kỳ thẻ > 5 ngày | Phụ thuộc đang không được xử lý | Escalation theo chủ đề 6 |
| Scope tăng | Số item thêm sau khi chốt / số item ban đầu | > 10% mà không có item nào bị bỏ | Đang có scope creep không được ai phê duyệt | Đưa ra quyết định đổi 1:1, có người ký |
| Giờ làm thêm | Tự khai, hoặc giờ commit sau 20:00 | Tăng 2 tuần liền | Team đang bù bằng sức người, tức độ trễ đang bị che | Coi như tín hiệu trễ đã xảy ra; báo ngay |

Tín hiệu cuối cùng đáng nói riêng: **giờ làm thêm tăng là một tín hiệu trễ, không phải một giải pháp.**
Khi team bắt đầu cày, độ trễ đã tồn tại và đang được chuyển từ dạng "ngày trễ" sang dạng "nợ sức khoẻ và
nợ chất lượng" — hai dạng không xuất hiện trong báo cáo và sẽ đến hạn muộn hơn với lãi.

**Bước 3 — Escalation có cấu trúc.** Bốn mức, mỗi mức có điều kiện kích hoạt, người nhận, và thời hạn.
Viết ra và dán vào tài liệu dự án, vì mục đích của nó là loại bỏ nhu cầu phải dũng cảm tại thời điểm quyết
định.

| Mức | Kích hoạt | Ai xử lý | Trong bao lâu | Output |
|---|---|---|---|---|
| **L0 — Trong team** | Một tín hiệu vượt ngưỡng lần đầu | Tech Lead + người liên quan | 24 giờ | Hành động cụ thể hoặc một dòng trong register |
| **L1 — Tech Lead + PO + EM** | Rủi ro có thể ảnh hưởng ngày giao, hoặc L0 không giải quyết trong 3 ngày | Minh + Trang + Hà | 2 ngày làm việc | Quyết định: nhận thêm rủi ro, cắt phạm vi, hay đổi ngày. Có ghi lại |
| **L2 — Stakeholder / người có accountability** | Dự báo lệch > 1 tuần so với cam kết, hoặc cần quyết định vượt thẩm quyền L1 | Hà báo Sơn (founder) hoặc khách hàng | 3 ngày làm việc | Chọn một trong ba phương án; ghi lại lựa chọn và nợ phát sinh |
| **L3 — Tái lập cam kết** | Lệch > 3 tuần, hoặc phạm vi thay đổi bản chất | Sơn + Hà + Trang | 1 tuần | Cam kết mới, công bố lại, thông báo các bên bị ảnh hưởng |

Quy tắc kèm theo, và đây là quy tắc quan trọng nhất trong toàn bộ chủ đề này: **quy tắc 24 giờ.** Từ lúc
bạn *tin rằng* khả năng trễ vượt một tuần, bạn có 24 giờ để nói với L1 — không chờ xác nhận, không chờ
đủ dữ liệu, không chờ tới buổi họp tuần đã có trong lịch. Lý do: chờ tới buổi họp tuần trung bình làm mất
3,5 ngày, và trong một dự án 12 tuần thì đó là 4% thời gian còn lại, mỗi lần.

**Bước 4 — Template escalation update.** Một trang, cùng cấu trúc mỗi lần, để người đọc không phải học
lại cách đọc.

```
CẬP NHẬT RỦI RO — Thanh toán trả góp · 12/11 · Người viết: Minh (Tech Lead)
Gửi: Sơn (founder), Trang (PO), Hà (EM) · Mức escalation: L2
Trạng thái: ĐỎ (đổi từ VÀNG ngày 05/11)

1. KẾT LUẬN TRƯỚC (đọc dòng này là đủ nếu anh chỉ có 30 giây)
   Dự báo mới: launch 21/12, trễ 3 tuần so với cam kết 30/11.
   Độ chắc: P50 = 21/12, P90 = 04/01. Tôi cần một quyết định từ anh trước 15/11.

2. VÌ SAO — ba nguyên nhân, có số
   a) Sandbox đối tác có ngày 26/10 thay vì 10/10 → mất 2,3 tuần thực tế (R-01).
   b) Luồng xử lý giao dịch lỗi phức tạp hơn: 14 ca lỗi cần xử lý thay vì 6 → +1,5 tuần (R-04).
   c) Khoa bị hút sang chốt sổ tháng 11, mất khoảng 40% thời gian trong 2 tuần → +0,8 tuần (R-02).
   Đã bù được: 1,6 tuần bằng cách cắt phần báo cáo đối soát tự động khỏi v1 (đã quyết 02/11).

3. NHỮNG GÌ TÔI ĐÃ LÀM TRƯỚC KHI BÁO
   - Cắt phạm vi trong thẩm quyền của tôi: bỏ báo cáo tự động, bỏ 2 màn hình cấu hình.
   - Chuyển Quân sang pair với Khoa để giảm phụ thuộc một người.
   - Dùng mock để làm song song trong lúc chờ sandbox.
   Tôi không xin thêm người, vì thêm người vào giai đoạn này làm chậm thêm (đã tính ở mục 5).

4. BA PHƯƠNG ÁN — tôi cần anh chọn, không cần anh giải
   A. Giữ 30/11, cắt phạm vi: chỉ hỗ trợ 1 đối tác thay vì 3, không có luồng huỷ hợp đồng.
      Được: đúng ngày. Mất: 60% khách hàng mục tiêu chưa dùng được. Nợ: 2 tuần trả trong Q1.
   B. Lùi sang 21/12, giữ nguyên phạm vi. Được: sản phẩm đủ dùng. Mất: lệch mùa mua sắm cuối năm.
   C. Lùi 14/12, cắt luồng huỷ hợp đồng (làm thủ công qua CSKH trong 6 tuần đầu).
      Được: gần mùa vụ hơn. Mất: CSKH tăng tải, ước 20 ca/ngày. Nợ: 1 tuần trả trong Q1.
   Khuyến nghị của tôi: C. Lý do: giữ được phần lớn giá trị mùa vụ, phần cắt có phương án tay
   chịu được ở khối lượng hiện tại, và nợ nhỏ nhất trong ba phương án.

5. NHỮNG GÌ TÔI KHÔNG LÀM VÀ VÌ SAO
   - Không thêm người: 3 tuần onboarding cho phần đối soát, trong khi chỉ còn 6 tuần.
   - Không cắt test cho luồng tiền: đây là ranh giới tôi không đề xuất vượt (xem chủ đề 8).
   - Không hứa "cày bù": đã có 2 tuần làm thêm, và hiệu suất tuần này thấp hơn tuần trước.

6. ĐIỀU KIỆN CỦA DỰ BÁO MỚI (nếu một trong các điều này sai, tôi báo lại trong 24 giờ)
   - Pháp chế xác nhận quy tắc lãi trước 20/11 (chủ: Trang).
   - Không thêm phạm vi mới từ nay tới launch.
   - Khoa ở lại dự án tối thiểu 80% thời gian (chủ: Hà).

7. ĐIỂM KIỂM TRA TIẾP THEO: 19/11, tôi gửi bản cập nhật kể cả khi không có gì đổi.
```

Bốn đặc điểm làm template này hoạt động: kết luận đặt đầu tiên (người đọc cấp cao đọc dòng đầu và dừng);
có số cho từng nguyên nhân (nên không thể bị đọc là bào chữa); nêu những gì đã tự làm trước khi xin (nên
không bị đọc là đẩy việc lên); và đưa ba phương án kèm khuyến nghị nhưng để người có accountability chọn.
Mục 5 — những gì bạn *không* làm và vì sao — là mục ít ai viết và có tác dụng lớn nhất, vì nó chặn trước
ba đề xuất phản xạ mà stakeholder sẽ đưa ra (thêm người, cắt test, cày bù).

**Bước 5 — Cơ chế no-surprise.** Bốn thành phần rẻ, đặt ngay từ tuần đầu dự án:

1. **Bản cập nhật một trang, cùng ngày cùng giờ mỗi tuần**, do người gần việc nhất viết, gửi nguyên văn
   không qua biên tập. Có một trường bắt buộc: "rủi ro mới hoặc thay đổi tuần này". Trường này bắt buộc
   điền, và điền "không có" là một lựa chọn hợp lệ — chính vì được phép điền "không có" mà nó không bị
   coi là hành động tố cáo, nên nó được điền thật.
2. **Pre-mortem 45 phút ở tuần đầu.** Câu hỏi mở đầu: "Giả sử hôm nay là ngày launch và chúng ta đã trễ
   một tháng. Viết ra ba lý do." Viết cá nhân 5 phút trước khi nói — bước này quan trọng vì nó tránh
   hiệu ứng người nói đầu tiên định hình cả phòng, và trong bối cảnh nơi junior khó phản biện senior, nó
   là cách duy nhất để lấy được thông tin từ họ. Sau đó gom nhóm, chọn 5 rủi ro điểm cao nhất, gán chủ,
   đưa vào register. Toàn bộ output là 5 dòng.
3. **Định nghĩa Xanh/Vàng/Đỏ bằng tiêu chí, không bằng cảm giác.** Ví dụ: Xanh = dự báo P50 nằm trong
   cam kết và không có tín hiệu nào vượt ngưỡng. Vàng = P50 trong cam kết nhưng P90 vượt, hoặc có 1 tín
   hiệu vượt ngưỡng. Đỏ = P50 vượt cam kết, hoặc từ 2 tín hiệu vượt ngưỡng. Không có tiêu chí thì mọi
   dự án đều Xanh cho tới khi thành Đỏ trong một tuần.
4. **Một câu hỏi cố định trong 1-1:** "Có điều gì em nghĩ sẽ làm dự án này trễ mà em chưa nói trong họp
   không?" Lý do nó hoạt động: nó ở kênh riêng (rẻ hơn về mặt giữ mặt), nó giả định trước rằng có thứ
   chưa được nói (nên trả lời "có" không phải là thừa nhận vi phạm), và nó được hỏi mọi lần nên không bị
   đọc là một cuộc điều tra.

### Trade-off

**Báo sớm khi chưa chắc vs báo muộn khi đã chắc.** Đây là quyết định thật, xuất hiện vài lần mỗi quý, và
không có câu trả lời đúng cho mọi tình huống. Hãy phân tích nghiêm túc cả hai phía.

Phía báo sớm: bạn đưa tin khi xác suất trễ khoảng 50–60%, còn 8 tuần. Cái được: stakeholder có toàn bộ
tập lựa chọn (bảng ở nguyên lý 3). Cái mất, và nó là mất thật: nếu sau đó dự án về đúng hạn, bạn đã tiêu
một lượng vốn uy tín. Chi phí đó không phải tưởng tượng — có một hiệu ứng đo được: người báo động nhiều
lần mà không xảy ra sẽ bị giảm trọng số trong các lần sau, và ở lần thứ tư thì cảnh báo của họ không còn
gây hành động. Ngoài ra, một cảnh báo sớm có thể kích hoạt phản ứng quá mức từ trên: thêm người, thêm
báo cáo hàng ngày, một buổi họp khẩn hàng ngày — tất cả đều làm chậm team thêm và có thể tự biến cảnh báo
thành hiện thực.

Phía báo muộn: bạn chờ tới khi xác suất 90%, còn 2 tuần. Cái được: bạn gần như luôn đúng, nên mỗi lần
bạn nói, tổ chức tin. Cái mất: stakeholder mất khả năng ứng phó, và điều này thường tốn nhiều hơn nhiều
lần so với sai số của một cảnh báo sớm. Thêm một cái mất khó thấy: bạn dạy tổ chức rằng dự án chỉ có hai
trạng thái, ổn và hỏng — nên không ai đầu tư vào leading indicator, nên vấn đề lặp lại.

Đây là cách tôi giải, và nó dựa trên bất đối xứng thiệt hại chứ không dựa trên nguyên tắc đạo đức:

| Điều kiện | Nghiêng về báo sớm | Nghiêng về chờ thêm dữ liệu |
|---|---|---|
| Còn nhiều thời gian (> 4 tuần) | Có — tập lựa chọn còn rộng, giá của báo sớm thấp | |
| Còn ít thời gian (< 2 tuần) | | Không phải "chờ", mà là báo ngay kèm phương án — vì lúc này thông tin thêm không đổi được gì |
| Chi phí ứng phó của stakeholder cao (đã đặt quảng cáo, đã hứa đối tác, đã lên lịch sự kiện) | Có, rất mạnh — họ cần lead time dài hơn bạn | |
| Nguyên nhân nằm ngoài tầm kiểm soát của team (phụ thuộc, phê duyệt) | Có — chỉ stakeholder mới xử lý được | |
| Nguyên nhân trong tầm kiểm soát và bạn có phương án đang chạy | | Có thể chờ tới điểm kiểm tra gần nhất, nếu điểm đó trong 5 ngày |
| Bạn đã báo động 3 lần trong quý mà không lần nào xảy ra | | Có — xem lại ngưỡng của bạn; ngưỡng quá nhạy làm hỏng kênh |
| Tổ chức phản ứng với tin xấu bằng cách thêm kiểm soát | Vẫn báo, nhưng đổi cách: báo kèm phương án đã có và một yêu cầu cụ thể ("em cần anh quyết X", không phải "em cần giúp") | |

Kỹ thuật giải quyết phần lớn trade-off này: **báo sớm nhưng báo đúng loại phát ngôn.** Câu "dự án sẽ trễ"
ở tuần 4 là một dự báo bị phát ngôn như một sự thật, và đó là nguồn của cả hai vấn đề. Thay bằng: "Xác
suất trễ hiện khoảng 50%. Tôi chưa cần anh làm gì. Tôi cần anh biết, và tôi cần anh biết rằng nếu đến
ngày 20/10 điều kiện C1 chưa xong thì xác suất lên 85% và lúc đó anh sẽ phải chọn giữa ba phương án này."
Phát ngôn này có ba tính chất tốt: nó không tốn uy tín nếu không xảy ra (vì bạn đã nói 50%), nó cho
stakeholder lead time để suy nghĩ, và nó chuyển cuộc đối thoại sang một ngày kiểm tra cụ thể thay vì để
mở. Trong thực tế, phần lớn các trường hợp "báo sớm mất uy tín" là do phát ngôn sai loại, không do báo sớm.

**Minh bạch toàn bộ vs lọc theo mức độ.** Báo mọi rủi ro cho mọi người nghe có vẻ đúng về mặt đạo đức
nhưng phá chức năng tín hiệu: nếu register 40 dòng được gửi cho founder hàng tuần, ông ấy không thể phân
biệt dòng nào quan trọng, nên ông ấy sẽ hoặc bỏ qua tất cả, hoặc phản ứng với dòng ngẫu nhiên nào gây lo
lắng nhất. Nguyên tắc phân tầng: L1 thấy toàn bộ register; L2 chỉ thấy những dòng có thể ảnh hưởng cam
kết hoặc cần quyết định của họ; và mọi dòng bị lọc ra phải vẫn tồn tại ở đâu đó kiểm được, để việc lọc
không biến thành việc che. Ranh giới giữa lọc và che nằm ở một câu hỏi: nếu sau này người đó biết rằng
bạn đã biết điều này và không nói, họ có thấy đó là hợp lý không? Nếu không chắc, nói.

**Buffer công khai vs buffer ẩn.** Đã bàn ở chủ đề 4 dưới góc ước lượng; ở đây thêm một góc về rủi ro:
buffer ẩn làm cho hệ thống cảnh báo mất hiệu lực, vì bạn không biết mình đã tiêu bao nhiêu phần đệm. Với
buffer công khai có tên ("2 tuần đệm cho rủi ro tích hợp"), bạn có thêm một leading indicator rất tốt là
tỉ lệ đệm đã tiêu so với tỉ lệ công việc đã xong — nếu tiêu 80% đệm khi mới xong 40% việc, bạn có tín
hiệu rõ ràng ở tuần 5 thay vì tuần 10.

### Real-world Scenarios

**Tình huống A — Báo stakeholder rằng dự án trễ 3 tuần.**

Bối cảnh: chính dự án ở đầu chương, nhưng lần này Minh phát hiện ở ngày 12/11 thay vì 28/11. Anh có bảng
số, có ba phương án, và có 30 phút với Sơn (founder).

*Phiên bản nói sai:*

> **Minh:** Anh ơi, em nghĩ là mình khó kịp 30/11.
>
> **Sơn:** Khó là sao? Tuần trước em nói 85%.
>
> **Minh:** Dạ, thì 85% nhưng phần còn lại nó phức tạp hơn bọn em tưởng. Cái sandbox của đối tác cũng
> chậm, rồi Khoa bị kéo sang chốt sổ nữa ạ.
>
> **Sơn:** Vậy bao giờ xong?
>
> **Minh:** Em chưa dám nói chắc. Có thể giữa tháng 12, nhưng còn tuỳ.
>
> **Sơn:** Tuỳ cái gì? Em cần thêm gì? Anh điều Vy sang giúp được không?
>
> **Minh:** Vâng, có thêm người thì tốt ạ. Bọn em cũng đang cố cày.
>
> **Sơn:** Được, anh điều Vy sang. Và từ mai em báo cáo tiến độ hàng ngày cho anh.
>
> **Minh:** Vâng ạ.

Kết cục dự đoán được: Vy mất 2 tuần để hiểu code, và tiêu thời gian của Khoa để onboarding, làm chậm thêm
khoảng 1 tuần; báo cáo hàng ngày tiêu 45 phút mỗi ngày của Minh cộng sự gián đoạn; và vì Minh không đưa
được ngày nào, Sơn tự đặt kỳ vọng 15/12 trong đầu và sẽ coi mọi thứ sau đó là thất bại thứ hai. Điều đáng
chú ý: Minh nói thật ở mọi câu. Nhưng anh đã trao cho Sơn ba thứ mà Sơn không nên nhận — sự mơ hồ, một
bài toán cần Sơn tự giải, và một khoảng trống mà Sơn phải điền bằng phản xạ quản lý.

*Phiên bản nói đúng:*

> **Minh:** Anh cho em 15 phút, em có một tin xấu và ba phương án. Kết luận trước: dự báo mới của em là
> 21/12, trễ 3 tuần so với 30/11. Em cần anh chọn một phương án trước 15/11.
>
> **Sơn:** Ba tuần. Tuần trước em nói 85%.
>
> **Minh:** Con số 85% đó là sai, và cái sai là của em — em báo % thay vì báo số việc còn lại. Từ tuần
> này em bỏ đơn vị %. Nguyên nhân có ba, đều có số: sandbox đối tác về ngày 26/10 thay vì 10/10, mất 2,3
> tuần; luồng xử lý lỗi có 14 ca thay vì 6, thêm 1,5 tuần; Khoa mất 40% thời gian cho chốt sổ, thêm 0,8
> tuần. Em đã bù lại được 1,6 tuần bằng cách cắt báo cáo đối soát tự động khỏi bản một.
>
> **Sơn:** Sao giờ này mới nói?
>
> **Minh:** Câu hỏi đúng. Em biết về sandbox từ 08/09 và em không báo, vì lúc đó em nghĩ mình bù được.
> Đó là lỗi cơ chế của em và em đã sửa: từ tuần này, bản cập nhật tuần của em có một dòng bắt buộc về
> rủi ro, và em cam kết quy tắc 24 giờ — khi em tin khả năng trễ trên một tuần, em nói trong 24 giờ,
> không chờ chắc.
>
> **Sơn:** Được. Ba phương án là gì?
>
> **Minh:** A: giữ 30/11, chỉ hỗ trợ 1 đối tác thay vì 3, không có luồng huỷ hợp đồng — đúng ngày, nhưng
> khoảng 60% khách mục tiêu chưa dùng được. B: lùi 21/12, giữ nguyên phạm vi — sản phẩm đủ, nhưng lệch
> mùa mua sắm. C: lùi 14/12, cắt luồng huỷ hợp đồng và làm thủ công qua CSKH sáu tuần đầu — ước 20 ca
> một ngày, chị Linh đã xác nhận chịu được ở mức đó. Em khuyến nghị C, vì nó giữ phần lớn giá trị mùa vụ
> và nợ phát sinh nhỏ nhất.
>
> **Sơn:** Em cần thêm người không? Anh điều Vy sang được.
>
> **Minh:** Em đã tính và em không đề nghị điều Vy. Phần còn lại nằm trong module đối soát, onboarding
> mất khoảng 3 tuần và sẽ tiêu thời gian của Khoa — đúng người đang là điểm nghẽn. Nếu anh muốn giúp bằng
> nguồn lực, thứ giúp được nhiều nhất là: một, giữ Khoa ở lại dự án tối thiểu 80% thời gian, việc này
> cần anh nói với bên tài chính; hai, giúp em có câu trả lời từ pháp chế trước 20/11.
>
> **Sơn:** Anh làm được cả hai. Anh chọn C. Nhưng anh cần biết chắc là không có tin xấu thứ hai.
>
> **Minh:** Em không hứa được là không có. Em hứa được ba việc: dự báo mới của em là P50 14/12 và P90
> 28/12 với phương án C; ba điều kiện của dự báo đó em ghi rõ trong email gửi anh chiều nay; và nếu bất
> kỳ điều kiện nào sai, anh biết trong 24 giờ. Điểm kiểm tra tiếp theo là 19/11, em gửi cập nhật kể cả
> khi không có gì đổi.

Bảng mổ xẻ:

| Yếu tố | Bản sai | Bản đúng | Cơ chế |
|---|---|---|---|
| Câu mở đầu | "em nghĩ là khó kịp" | Kết luận, con số, và điều cần quyết | Người nghe cấp cao xử lý thông tin theo thứ tự nhận được; mơ hồ ở câu đầu làm mọi câu sau bị nghe qua bộ lọc nghi ngờ |
| Xử lý con số cũ | Lảng tránh | Nhận sai cụ thể và nêu cách sửa cơ chế | Nhận lỗi về *cơ chế* phục hồi được uy tín; nhận lỗi chung ("em xin lỗi") thì không |
| Nguyên nhân | Ba lý do không có số, nghe như bào chữa | Ba nguyên nhân có số cộng phần đã tự bù | Số làm cho nguyên nhân trở thành phân tích; thiếu số làm nó thành lời giải thích |
| Câu "sao giờ mới nói" | Không có câu trả lời | Trả lời thẳng, nêu ngày mình đã biết | Đây là câu quyết định uy tín. Né nó thì mất nhiều hơn là thừa nhận |
| Cấu trúc quyết định | Để Sơn tự tìm cách | Ba phương án + khuyến nghị + để Sơn chọn | Giữ đúng ranh giới: bạn sở hữu phân tích, người có accountability sở hữu lựa chọn |
| Đề nghị thêm người | Nhận vì khó từ chối | Từ chối kèm lý do định lượng, đổi bằng hai yêu cầu cụ thể | Chuyển mong muốn giúp của stakeholder sang chỗ nó thật sự có tác dụng |
| Cam kết cuối | "bọn em sẽ cố" | Khoảng + điều kiện + quy tắc 24 giờ + ngày kiểm tra | Cam kết về *hành vi báo cáo* là cam kết bạn kiểm soát được 100%, nên nó xây lại được niềm tin |

**Tình huống B — Thêm người vào dự án trễ, quan sát bằng số.**

Bối cảnh: một ODC, dự án 4 tháng đang trễ, còn 6 tuần. Khách yêu cầu thêm 3 người vào team 6 người.
Quản lý đồng ý vì việc từ chối khó nói với khách. Kết quả sau 6 tuần, đo lại (số minh hoạ, nhưng cơ chế
là chuẩn xác):

| Hạng mục | Trước khi thêm người | Sau 6 tuần với 9 người |
|---|---|---|
| Throughput (item/tuần) | 7,5 | 6,8 |
| Thời gian 3 senior dành cho onboarding | 0 | ~90 giờ (tương đương 2,2 người-tuần) |
| Số kênh phối hợp (n(n−1)/2) | 15 | 36 |
| Defect mới do người mới chưa hiểu ngữ cảnh | — | 11 |
| Kết quả | | Trễ thêm 1,5 tuần so với dự báo không thêm người |

Đây là Brooks's Law hoạt động, và cơ chế của nó có ba phần đáng tách riêng: chi phí onboarding lấy đúng
thời gian của người giỏi nhất (những người đang trên đường tới hạn); chi phí phối hợp tăng theo bình
phương số người; và công việc trong giai đoạn cuối dự án thường không chia nhỏ được — nó là tích hợp,
sửa lỗi, và đưa vào ổn định, những việc mà thêm người song song không giúp. Ngoại lệ có thật, và cần nêu
để không tuyệt đối hoá: thêm người *có* giúp khi công việc còn lại tách được thành phần độc lập không cần
ngữ cảnh sâu (viết test cho module đã ổn định, làm tài liệu, sửa lỗi UI đơn giản có mô tả rõ), hoặc khi
người thêm vào đã từng làm chính hệ thống đó. Ranh giới thực dụng: nếu người mới cần trên một tuần để
tạo ra giá trị và bạn còn dưới 6 tuần, đừng thêm; hãy dùng cùng nguồn lực đó để lấy bớt việc *khác* ra
khỏi vai team hiện tại — trực on-call, hỗ trợ bộ phận khác, họp — vì đó là cách tăng capacity không tốn
chi phí onboarding.

### Best Practices

**Chạy pre-mortem ở tuần đầu, viết cá nhân trước khi nói.** Cơ chế: nó chuyển thời điểm phát hiện rủi ro
về đầu dự án, nơi chi phí xử lý thấp nhất; và bước viết cá nhân là điều kiện để lấy được thông tin từ
người ít quyền phát biểu nhất, thường là người gần code nhất.

**Giữ register dưới 10 dòng và rà soát 15 phút mỗi tuần.** Một register dài không được đọc; một register
không được rà soát thì lỗi thời sau hai tuần và tạo cảm giác kiểm soát giả — trạng thái tệ hơn không có gì.

**Đặt ngưỡng cho tín hiệu trước khi có dữ liệu.** Nếu ngưỡng được đặt sau, nó sẽ được đặt vừa đủ để số
hiện tại không vượt. Đây không phải sự bất lương; đó là cách nhận thức con người hoạt động khi có động
lực, và cách chống lại là quyết định trước.

**Cam kết một quy tắc 24 giờ và nêu nó ra thành lời với stakeholder.** Việc nói ra thành lời làm hai
việc: nó biến hành vi báo sớm thành một cam kết công khai (nên dễ thực hiện hơn), và nó cho stakeholder
một thứ để tin ngay lập tức, ngay cả khi họ chưa tin dự báo của bạn.

**Báo tin xấu kèm ba phương án, nhưng đừng quyết thay người có accountability.** Cơ chế: đưa phương án
giữ bạn ở vai người phân tích và ngăn phản xạ quản lý của stakeholder; nhưng quyết thay họ nghĩa là bạn
nhận trách nhiệm về một đánh đổi kinh doanh mà bạn không có toàn bộ thông tin, và về lâu dài nó khiến họ
không còn thực sự sở hữu quyết định.

**Trả lời thẳng câu "sao giờ này mới nói".** Nêu ngày bạn biết, và nêu cơ chế bạn sửa. Cơ chế phục hồi
uy tín không phải là chứng minh mình không sai, mà là chứng minh rằng lần sau sẽ khác vì cấu trúc đã đổi.

**Phân biệt rõ trong mọi lần báo cáo: đây là cam kết, đây là dự báo, đây là điều kiện của người khác.**
Ba loại này bị trộn là nguồn của phần lớn cảm giác "team hay hứa rồi không làm được".

**Ghi lại quyết định và nợ phát sinh ngay tại buổi họp, gửi lại trong ngày.** Cơ chế: sau 4 tuần, ký ức
của các bên về việc "ta đã đồng ý cắt cái gì" sẽ phân kỳ, và bên có ít quyền lực hơn sẽ là bên thua trong
cuộc phân kỳ đó.

### Anti-patterns

**Che giấu và hy vọng bù được.** Hình dạng: biết trễ ở tuần 4, quyết định tự cày để bù, không báo. Cơ chế
phá hệ thống ba tầng. Tầng một, số học: nếu bạn đã tiêu hết đệm ở tuần 4, khả năng bù bằng cường độ là
thấp, vì cường độ chỉ tăng được khoảng 10–20% trong ngắn hạn và giảm sau 2–3 tuần do sai sót tăng. Tầng
hai, quyền quyết định: bạn đã tự quyết một đánh đổi kinh doanh (giá trị của việc đúng hạn so với phạm vi
và chất lượng) mà bạn không có thẩm quyền và không có thông tin — có thể Sơn sẵn sàng lùi hai tuần mà bạn
không biết. Tầng ba, lãi kép của sự im lặng: khi cuối cùng phải nói, bạn không chỉ báo trễ mà còn báo
rằng thông tin đã bị giữ, nên mất mát về uy tín lớn hơn nhiều lần. Dấu hiệu sớm: cùng một % hai tuần
liền; giờ làm thêm tăng mà phạm vi không đổi; Tech Lead bắt đầu tự nhận việc code của người khác; câu
"để em xem thêm vài ngày nữa rồi báo anh" xuất hiện lần thứ hai.

**Thêm người vào dự án đang trễ.** Hình dạng: đã phân tích ở tình huống B. Điểm cần nhấn về cơ chế nhận
thức: đề nghị này rất khó từ chối vì nó là một hành động *giúp đỡ*, và từ chối sự giúp đỡ trong bối cảnh
Việt Nam đặc biệt khó khi người đề nghị là cấp trên hoặc khách hàng. Vì vậy cần chuẩn bị trước một câu
trả lời có dạng "không, nhưng có": không thêm người vào phần tới hạn, nhưng đây là hai việc anh giúp
được và chúng có tác dụng lớn hơn. Dấu hiệu sớm: câu "hay điều thêm người sang" xuất hiện trong buổi báo
trễ; hoặc một người mới được thêm vào mà không ai xác định được việc cụ thể họ làm trong hai tuần đầu.

**Cắt test để giữ ngày.** Hình dạng: tuần cuối, bỏ regression, bỏ test cho các ca lỗi, gộp việc kiểm thử
vào "sẽ test trên production". Cơ chế: nó không tiết kiệm thời gian mà chuyển thời gian sang một khoảng
khác không nằm trong kế hoạch — thời gian sửa lỗi sau launch, thời gian hỗ trợ, thời gian rollback — và
khoảng đó có phương sai cao hơn nhiều. Ở lĩnh vực tiền, thêm một cơ chế nữa: lỗi tính toán tiền không
chỉ tốn thời gian sửa mà tạo nghĩa vụ đối soát và bồi hoàn, và trong một số trường hợp là nghĩa vụ báo
cáo với cơ quan quản lý. Điều làm anti-pattern này đặc biệt nguy hiểm là nó *không cần ai phê duyệt* —
đó là nội dung chính của chủ đề 8. Dấu hiệu sớm: tỉ lệ test tự động chạy trong CI giảm; định nghĩa Done
được sửa im lặng; người QA nói "không kịp test" trong standup và không ai ghi lại điều đó thành một quyết
định.

**Watermelon status — xanh ngoài đỏ trong.** Hình dạng: báo cáo Xanh ba tuần liền rồi chuyển Đỏ. Cơ chế:
màu được chọn theo cảm giác và theo mong muốn tránh câu hỏi, nên nó phản ánh mức chịu đựng của người báo
chứ không phản ánh trạng thái dự án. Dấu hiệu sớm: không có định nghĩa bằng tiêu chí cho ba màu; hoặc
lịch sử màu của mọi dự án trong sáu tháng qua đều là Xanh cho tới hai tuần cuối.

**Risk register như một nghi lễ.** Hình dạng: có file, 35 dòng, cột xác suất và tác động điền bằng
Cao/Trung bình/Thấp, không có chủ, không có tín hiệu, cập nhật một lần khi bắt đầu dự án. Cơ chế: nó
thoả mãn yêu cầu quy trình mà không thay đổi hành vi nào, và nó chiếm chỗ của cơ chế thật — khi ai đó
hỏi "ta có quản lý rủi ro không", câu trả lời là "có, có file". Dấu hiệu sớm: không dòng nào từng bị
đóng hoặc bị nâng mức; không có ngày rà soát; hoặc không ai ngoài người tạo file mở nó trong 30 ngày.

**Escalation bị dùng như hành vi buộc tội.** Hình dạng: bản escalation viết theo hướng chỉ ra team nào
gây chậm. Cơ chế: nó khiến các bên phòng thủ, nên thông tin ở lần sau bị che tốt hơn, nên hệ thống cảnh
báo xấu đi chính xác ở nơi bạn vừa dùng nó. Dấu hiệu sớm: bản escalation có tên team ở phần nguyên nhân
nhưng không có phần "chúng tôi đã tự làm gì"; hoặc sau một lần escalation, một kênh thông tin không chính
thức biến mất.

### Khi nào KHÔNG nên áp dụng

**Khi dự án nhỏ hơn ngưỡng mà bộ máy này có nghĩa.** Với một việc 2 tuần, một người, không phụ thuộc:
register, pre-mortem, ngưỡng, và bốn mức escalation là chi phí thuần. Cái vẫn nên giữ, vì nó gần như miễn
phí: quy tắc 24 giờ, và một câu hỏi ở giữa kỳ — "còn đúng như dự tính không". Ngưỡng thực dụng để bắt đầu
dùng bộ đầy đủ: dự án trên 4 tuần, hoặc có trên 2 người, hoặc có bất kỳ phụ thuộc ngoài team, hoặc có
một cam kết đã công bố ra ngoài.

**Khi tổ chức trừng phạt người báo tin xấu.** Đây là điều kiện biên quan trọng nhất trong chủ đề này. Nếu
lần gần nhất một Tech Lead báo trễ sớm, kết quả là anh ta bị đặt câu hỏi về năng lực trong một buổi họp
đông người, thì việc triển khai register và quy tắc 24 giờ sẽ tạo ra một hệ thống tài liệu đẹp với nội
dung sai — mọi rủi ro được ghi ở mức thấp, mọi màu là Xanh, và bạn đã thêm chi phí mà không thêm thông
tin. Tệ hơn, register lúc đó trở thành bằng chứng dùng để quy trách nhiệm cá nhân về sau, nên nó tích cực
làm hại. Thứ tự đúng: bắt đầu bằng một quy tắc hành vi ở tầng lãnh đạo — phản ứng đầu tiên với một tin
xấu là cảm ơn và hỏi "cần gì", không phải hỏi "tại sao" (chương 03) — và chứng minh nó bằng ba lần liên
tiếp trước khi triển khai công cụ. Trong lúc chưa có điều kiện đó, cái dùng được là kênh riêng: bạn giữ
register cho mình và EM, và bạn dùng 1-1 làm kênh thu thập chính.

**Khi rủi ro không có hành động nào khả thi.** Có những rủi ro thật mà bạn không làm gì được và cũng
không ai làm gì được: một thay đổi quy định có thể xảy ra trong sáu tháng tới, một đối tác có thể bị mua
lại. Đưa chúng vào register tuần làm loãng tài liệu và tiêu thời gian rà soát. Cách xử lý đúng: ghi vào
một danh sách "giả định nền" được xem lại mỗi quý, kèm một tín hiệu nếu có, và không đưa vào chu kỳ tuần.
Bài kiểm tra để phân loại: "nếu điều này xảy ra vào tuần sau, có hành động nào chúng ta làm khác đi từ
hôm nay không?" Nếu không, nó thuộc danh sách quý.

**Khi bạn đã báo động quá nhiều lần và kênh đã bị bão hoà.** Nếu ba lần gần nhất bạn chuyển trạng thái
Đỏ mà dự án đều về đúng hạn, vấn đề nằm ở ngưỡng của bạn, không ở người nghe. Trong điều kiện này, thêm
một cảnh báo nữa không tạo hành động, chỉ giảm thêm trọng số. Việc đúng: hiệu chỉnh ngưỡng bằng dữ liệu
lịch sử của chính bạn, công khai việc hiệu chỉnh đó ("ba lần trước em báo sớm quá, em đã đổi ngưỡng từ
10% lên 15% lệch hai sprint liền"), và trong lúc chờ xây lại độ tin cậy, báo bằng xác suất kèm điểm kiểm
tra thay vì bằng màu.

**Khi register có hệ quả hợp đồng và bạn chưa hiểu hệ quả đó.** Trong ODC, một dòng register viết rằng
"nếu khách không cấp môi trường trước 25/3 thì trễ 2 tuần" là tài liệu tốt về mặt delivery, nhưng nó cũng
là bằng chứng trong một tranh chấp hợp đồng — theo cả hai chiều. Đừng bỏ nó, nhưng đừng viết một mình:
trước khi phát hành register cho khách, thống nhất với người sở hữu hợp đồng về từ ngữ và về việc điều gì
được ghi ở kênh nào. Đây là chỗ mà sự thẳng thắn kỹ thuật cần đi kèm hiểu biết thương mại, và một Tech
Lead không nên tự quyết định một mình cách phát ngôn rủi ro ra ngoài biên tổ chức.

---

## 8. Delivery dưới ràng buộc: cắt cái gì khi không đủ thời gian

### Problem Statement

Còn ba tuần đến ngày mở campaign 9.9 của một sàn thương mại điện tử. Ước lượng còn lại của team là
năm tuần. Ngày campaign không dời được — ngân sách quảng cáo đã đặt, đối tác đã ký, và bộ phận
marketing đã chạy teaser.

Điều xảy ra ở phần lớn tổ chức không phải một cuộc họp để quyết định cắt gì. Điều xảy ra là **không
có cuộc họp nào cả**. Team làm đến 9 giờ tối, rồi 11 giờ. Ai đó bỏ viết test cho module tính khuyến
mãi vì "để sau, giờ chưa kịp". Một người merge thẳng vào main không chờ review vì reviewer đang bận.
Feature flag không được làm vì tốn thêm một ngày. Ngày 9.9 hệ thống lên, và trong 4 tiếng đầu có
hai lỗi tính sai giá trị voucher — loại lỗi mà một test đơn giản sẽ bắt được.

Đây là dạng thất bại đặc trưng nhất của tầng Execution, và nó có một đặc điểm chẩn đoán rõ: **không
ai từng đưa ra quyết định cắt chất lượng, nhưng chất lượng đã bị cắt.** Không có biên bản, không có
người chịu trách nhiệm, không có bản ghi nợ. Ba tháng sau, khi ai đó hỏi vì sao module khuyến mãi
khó sửa đến vậy, câu trả lời "vì hồi 9.9 gấp quá" không giúp được gì cho việc phân bổ nguồn lực.

Hậu quả khi tổ chức không có cơ chế cắt có chủ đích:

- Nợ kỹ thuật phát sinh vô danh — không ai biết đã vay bao nhiêu, ở đâu, để đổi lấy cái gì.
- Stakeholder học được rằng deadline không thể dời thì team vẫn "làm được", nên deadline tiếp theo
  sẽ chặt hơn. Đây là vòng phản hồi dương và nó không tự dừng.
- Người giỏi nhất kiệt sức trước, vì họ là người gánh phần khó nhất trong giai đoạn crunch.
- Lần sau, ước lượng của team sẽ được đệm ngầm — vì đó là cách phòng vệ duy nhất còn lại khi việc
  nói thẳng không có tác dụng. Tổ chức mất luôn khả năng dự báo.

### First Principles

Một cam kết giao hàng có **bốn biến**, và ba trong bốn biến chỉ thay đổi được khi có người phê duyệt:

| Biến | Ai phải đồng ý để thay đổi | Mức độ nhìn thấy được |
|---|---|---|
| Scope | PO / stakeholder / khách hàng | Cao — phải họp, phải thông báo |
| Thời gian | Stakeholder / marketing / khách | Cao — có ngày trên lịch |
| Nguồn lực | Manager / bộ phận tài chính | Trung bình — có chi phí trên sổ |
| **Chất lượng** | **Không ai** | **Gần bằng không** |

Đây là bất đối xứng cấu trúc, và nó giải thích toàn bộ hiện tượng ở mục trên. Chất lượng là biến duy
nhất một engineer đang mệt lúc 11 giờ đêm có thể tự điều chỉnh, một mình, không cần xin phép, không
để lại dấu vết trong bất kỳ hệ thống theo dõi nào. Trong một hệ thống mà ba biến bị khoá và một biến
tự do, **áp lực sẽ dồn hết vào biến tự do** — không phải vì ai đó vô trách nhiệm, mà vì đó là con
đường có lực cản nhỏ nhất.

Hệ quả quản trị đầu tiên: việc của lead không phải là hô hào giữ chất lượng. Việc của lead là **làm
cho việc cắt chất lượng trở nên nhìn thấy được và cần phê duyệt** — tức chuyển nó từ biến tự do thành
biến có cổng, ngang hàng với ba biến kia.

**Phân biệt hai loại chất lượng.** Đây là phân biệt quan trọng nhất trong chủ đề này, và nó thường bị
gộp làm một:

- **Chất lượng bên ngoài** (external quality): những gì người dùng cảm nhận được — tính năng có
  chạy đúng không, có nhanh không, có lỗi không, giao diện có dùng được không. Đây là thứ được đàm
  phán công khai với PO, và cắt nó là một **quyết định sản phẩm hợp lệ**. Ra một tính năng chỉ hỗ trợ
  một phương thức thanh toán thay vì ba là cắt scope, không phải hạ chuẩn.
- **Chất lượng bên trong** (internal quality): cấu trúc code, test, khả năng quan sát, khả năng
  rollback, tài liệu quyết định. Người dùng không thấy nó. Nhưng nó quyết định **chi phí của mọi
  thay đổi trong tương lai**.

Cắt chất lượng bên ngoài là *tiết kiệm*: bạn làm ít việc hơn, và bạn nhận ít giá trị hơn — một trao
đổi sòng phẳng, kết thúc tại đó. Cắt chất lượng bên trong là *vay nợ*: bạn không làm ít việc hơn, bạn
chuyển việc sang tương lai với lãi suất. Lãi ở đây là thời gian thêm cho mọi thay đổi sau đó trong
vùng code bị ảnh hưởng, và nó là **lãi kép** vì mã xấu thu hút mã xấu (chi tiết cơ chế ở
`05-technical-leadership.md`).

Hệ quả thứ hai: khi buộc phải cắt, **thứ tự ưu tiên đúng gần như luôn là cắt scope trước, cắt chất
lượng bên trong sau cùng** — bởi vì cắt scope có chi phí biết trước và dừng lại, còn cắt chất lượng
bên trong có chi phí không biết trước và tiếp tục tăng.

**Vì sao thêm người không cứu được.** Một dự án đang trễ có ba loại chi phí khi thêm người: thời gian
onboarding (người mới lấy thời gian của người cũ), chi phí phối hợp tăng theo số cặp giao tiếp
(n(n−1)/2), và việc phân rã lại công việc thành các phần độc lập — mà nếu công việc phân rã được dễ
dàng như vậy thì nó đã được phân rã từ đầu. Với dự án còn dưới bốn tuần, ba chi phí này gần như luôn
lớn hơn phần đóng góp. Đây là Brooks's Law, và nó vẫn đúng dù đã hơn năm mươi năm, vì nó không nói về
công cụ mà nói về cấu trúc của công việc tri thức có tính liên đới cao.

### Mental Model

**Tam giác ràng buộc có bốn đỉnh — và đỉnh thứ tư nằm dưới mặt bàn.** Mô hình cổ điển "scope – time –
cost, chọn hai" bỏ sót chất lượng vì nó ngầm giả định chất lượng là hằng số. Trong thực tế nó là biến
điều chỉnh mặc định. Hình dung đúng hơn:

```
        Scope
         / \
        /   \          Ba đỉnh này có cổng phê duyệt.
   Time ----- Cost      Ai muốn đổi phải xin.

    [ Chất lượng ]      Đỉnh thứ tư: không có cổng.
                        Mọi áp lực rò rỉ xuống đây.

Việc của lead: lắp một cổng vào đỉnh thứ tư,
để nó cũng đắt như ba đỉnh kia khi muốn đổi.
```

**Nợ có ghi sổ vs nợ vô danh.** Cùng một hành động kỹ thuật — bỏ test cho module khuyến mãi — có hai
kết cục hoàn toàn khác nhau tuỳ vào việc nó có được ghi lại hay không:

| | Nợ có ghi sổ | Nợ vô danh |
|---|---|---|
| Ai biết | PO, lead, team, có trong backlog | Người viết code, trong vài tuần |
| Ước lượng lãi | Có (ví dụ: +1 ngày cho mỗi thay đổi trong module) | Không ai biết |
| Có ngày trả | Có, và có chỗ trong capacity | Không |
| Khi có người mới vào | Đọc được, biết cần cẩn thận | Bước vào mìn |
| Tác động lên đàm phán lần sau | Có dữ liệu để nói "lần trước vay X, chưa trả" | Không có gì để nói |

Điểm mấu chốt: hành động kỹ thuật giống hệt nhau, giá trị quản trị khác nhau hoàn toàn. **Ghi nợ tốn
mười lăm phút và là phần có tỉ suất sinh lợi cao nhất trong toàn bộ quy trình cắt.**

**Ba loại deadline.** Không phải deadline nào cũng cùng bản chất, và cách xử lý khác nhau hoàn toàn:

| Loại | Đặc điểm | Ví dụ | Cách xử lý |
|---|---|---|---|
| Deadline cứng bên ngoài | Có sự kiện thật ngoài tầm kiểm soát, dời thì mất giá trị hoặc mất tiền thật | Campaign 9.9, hạn compliance của cơ quan quản lý, hợp đồng có phạt, mùa mua sắm | Cắt scope, không đàm phán ngày |
| Deadline mềm nội bộ | Do ai đó đặt để tạo áp lực hoặc để đồng bộ kế hoạch | "Cuối quý phải xong", ngày trong roadmap | Đàm phán được — hỏi chi phí thật của việc trễ hai tuần |
| Deadline giả | Không ai nhớ vì sao có ngày đó | Ngày trong một slide sáu tháng trước | Truy nguồn gốc; thường tan khi bị hỏi |

Sai lầm phổ biến của lead là đối xử với cả ba như loại thứ nhất. Câu hỏi đầu tiên khi nhận một
deadline không phải "làm sao kịp" mà **"nếu trễ hai tuần thì điều gì thực sự xảy ra, và ai chịu?"**
Câu trả lời phân loại deadline, và phân loại quyết định toàn bộ chiến lược phía sau.

### Practical Framework

**Quy trình bốn bước khi phát hiện không đủ thời gian.**

*Bước 1 — Làm rõ mục tiêu kinh doanh thật đằng sau ngày.* Trước khi bàn cắt gì, phải biết ngày đó
phục vụ điều gì. Ba câu hỏi cho stakeholder: (a) Nếu trễ hai tuần thì mất gì, tính được bằng tiền
không? (b) Phần nào của phạm vi tạo ra phần lớn giá trị đó? (c) Có cách nào đạt mục tiêu kinh doanh
mà không cần toàn bộ phạm vi này không? Câu (c) thường mở ra phương án rẻ nhất mà không ai nghĩ tới —
ví dụ với campaign 9.9, mục tiêu là **doanh thu trong 72 giờ**, không phải "ra đủ ba loại voucher".

*Bước 2 — Dựng ba phương án, không phải một.* Đây là khác biệt giữa một lead và một người đưa tin.
Đưa một phương án là đẩy quyết định lên trên mà không kèm thông tin. Đưa ba phương án có trade-off rõ
là trao cho người có accountability đúng thứ họ cần để quyết. Ba phương án phải khác nhau về **biến
bị cắt**, không phải khác nhau về mức độ lạc quan.

*Bước 3 — Để người có accountability chọn, và ghi lại lựa chọn.* Lead không tự quyết cắt scope —
scope thuộc PO. Lead không tự quyết dời ngày — ngày thuộc stakeholder. Lead **có** accountability về
việc nói rõ trade-off và về chất lượng bên trong. Ranh giới này phải được nói ra, nếu không lead sẽ
hoặc vượt quyền, hoặc im lặng gánh.

*Bước 4 — Ghi nợ ngay tại thời điểm vay.* Không phải "sẽ ghi sau". Mỗi khoản cắt chất lượng bên
trong tạo một ticket, có chủ, có ước lượng lãi, có chỗ trong capacity của quý sau.

**TEMPLATE — Ba phương án.** Đây là văn bản gửi trước cuộc họp, không phải trình bày miệng:

```
QUYẾT ĐỊNH CẦN CHỐT: Phạm vi campaign 9.9
Ngày:        [ngày]        Người cần quyết: Trang (PO) + [Head of Growth]
Người soạn:  Tuấn (Tech Lead)
Hạn chốt:    [ngày], sau ngày này phương án B và C không còn khả thi

BỐI CẢNH
- Ngày 9.9 không dời được: ngân sách quảng cáo [X] đã đặt, đối tác đã ký. (số minh hoạ)
- Phạm vi hiện tại ước lượng còn 5 tuần (P50) / 7 tuần (P90). Còn 3 tuần.
- Khoảng cách này không đóng được bằng thêm người: xem ghi chú cuối.

PHƯƠNG ÁN A — Giữ nguyên phạm vi, dời ngày 2 tuần
  Được:   Không nợ kỹ thuật. Chất lượng đầy đủ.
  Mất:    Mất cửa sổ campaign. Chi phí ước tính: [số minh hoạ].
  Ai chịu: Growth / Marketing.
  Khả thi: Chỉ nếu ngày là deadline mềm. Cần xác nhận từ [tên].

PHƯƠNG ÁN B — Giữ ngày, cắt phạm vi  ← KHUYẾN NGHỊ
  Cắt:    Voucher combo (3 tuần → 0), trang dashboard cho seller (1 tuần → 0).
  Giữ:    Flash sale + voucher đơn — chiếm ~80% doanh thu dự kiến. (số minh hoạ)
  Được:   Đúng ngày, chất lượng bên trong nguyên vẹn, không nợ.
  Mất:    Hai tính năng lùi sang tháng 10. Seller phải dùng báo cáo thủ công 1 tháng.
  Ai chịu: Product, và bộ phận vận hành seller (cần báo trước).

PHƯƠNG ÁN C — Giữ ngày, giữ phạm vi, vay nợ kỹ thuật có kiểm soát
  Cắt:    Test tự động cho voucher combo (thay bằng test tay theo checklist),
          bỏ dashboard seller khỏi giám sát tự động trong tháng đầu.
  KHÔNG cắt: kiểm tra tính đúng của số tiền, khả năng rollback, audit log,
          kiểm soát truy cập. Bốn mục này không nằm trong phạm vi đàm phán.
  Nợ phát sinh: ước tính 8 ngày công, ticket TD-31..TD-34, trả trong quý IV.
  Rủi ro:  Xác suất lỗi tính giá trong 72h đầu tăng đáng kể. Cần 2 người trực
          suốt campaign (đã tính chi phí).
  Ai chịu: Engineering chịu nợ; Growth chịu rủi ro lỗi trong campaign.

GHI CHÚ — vì sao không thêm người
  Thêm 2 người vào tuần này: onboarding lấy ~1 tuần của 2 người hiện tại,
  công việc còn lại nằm trên cùng một module nên khó tách. Kỳ vọng thực tế
  là chậm hơn, không nhanh hơn.

TÔI CẦN GÌ TỪ CUỘC HỌP NÀY
  Một lựa chọn A / B / C. Nếu chọn C, cần xác nhận bằng văn bản rằng
  nợ TD-31..TD-34 có chỗ trong capacity quý IV.
```

**Danh sách cái gì cắt được và cái gì không.** Đây là ranh giới mà lead phải giữ kể cả khi bị ép, vì
nó là phần accountability không chia được:

| Cắt được (là quyết định sản phẩm hợp lệ) | Không bao giờ cắt |
|---|---|
| Phạm vi tính năng, số trường hợp hỗ trợ | Tính đúng của dữ liệu tiền và giao dịch |
| Độ hoàn thiện giao diện, animation | Kiểm soát truy cập và xác thực |
| Tối ưu hiệu năng chưa cần thiết | Khả năng rollback |
| Tự động hoá quy trình nội bộ (làm tay tạm) | Audit trail — bắt buộc trong fintech, ngân hàng |
| Test cho luồng phụ, độ phủ ở vùng ít rủi ro | Test cho luồng tiền và luồng dữ liệu cá nhân |
| Tài liệu hướng dẫn người dùng | Bản ghi quyết định đã cắt gì (nợ phải có sổ) |
| Dashboard, báo cáo nội bộ | Khả năng phát hiện sự cố (alert cho luồng chính) |

Nguyên tắc phân định: **cắt được những gì làm sản phẩm nhỏ hơn; không cắt được những gì làm hệ thống
mù hoặc không thể quay lui.** Khi hệ thống mù và không quay lui được, một lỗi nhỏ trở thành một
incident dài, và chi phí vượt xa toàn bộ phần tiết kiệm.

**TEMPLATE — Ghi nợ ngay lúc vay:**

```
TD-31 · Nợ phát sinh từ quyết định giao hàng
Ngày vay:      [ngày]
Vay để đổi lấy: giữ ngày 9.9 (quyết định của Trang + Growth, biên bản [link])
Đã cắt:        test tự động cho tính giá voucher combo (12 case)
Lãi ước tính:  +0,5–1 ngày cho mỗi thay đổi trong module pricing;
               rủi ro lỗi tính giá không được phát hiện tự động
Bù tạm:        checklist test tay [link], chạy trước mỗi deploy tới 30/09
Ngày trả:      quý IV, sprint 2 — đã có chỗ trong capacity plan
Chủ nợ:        Quân
Điều kiện leo thang: nếu có ≥ 1 lỗi tính giá lọt production trước ngày trả,
               nâng lên P-block và trả ngay sprint kế tiếp
```

### Trade-off

| Trục | Nghiêng về A khi | Nghiêng về B khi | Cái giá của mỗi bên |
|---|---|---|---|
| **A: Cắt scope** vs **B: Cắt chất lượng bên trong** | Gần như mọi trường hợp; đặc biệt khi sản phẩm sẽ sống lâu và còn phát triển tiếp | Phần code chắc chắn bị bỏ trong vài tháng (prototype, campaign một lần, thử nghiệm A/B) | A: mất giá trị thị trường, có thể mất cửa sổ cạnh tranh. B: chi phí không biết trước, tăng theo thời gian, và người trả không phải người quyết |
| **A: Dời ngày** vs **B: Giữ ngày và cắt** | Deadline mềm hoặc giả; chi phí trễ nhỏ hơn chi phí nợ | Deadline cứng có sự kiện thật ngoài tổ chức | A: mất uy tín dự báo nếu lặp lại nhiều lần; ảnh hưởng kế hoạch của bộ phận khác. B: sản phẩm ra thị trường nhỏ hơn kỳ vọng |
| **A: Crunch có giới hạn** vs **B: Không crunch** | Có ngày kết thúc rõ, dưới hai tuần, có bù sau, và cả team đồng ý sau khi biết trade-off | Không có ngày kết thúc; đã crunch trong quý gần đây; team có người đang có dấu hiệu kiệt sức | A: chất lượng quyết định giảm rõ sau tuần thứ hai, tỉ lệ lỗi tăng, và rủi ro mất người sau campaign. B: khả năng trễ cao hơn |
| **A: Thêm người** vs **B: Giữ nguyên team** | Còn > 8 tuần; công việc tách được thành phần độc lập; người mới đã quen hệ thống | Dưới 4 tuần; công việc tập trung một module | A: gần như luôn làm chậm trong ngắn hạn (Brooks's Law), và lấy thời gian của người đang là điểm nghẽn |
| **A: Ghi nợ công khai** vs **B: Sửa lặng lẽ sau** | Luôn — kể cả khi bạn tin mình sẽ sửa trong hai tuần | Không có trường hợp nào đáng | B: nợ trở thành vô danh, và khả năng đàm phán lần sau của bạn mất đi cơ sở dữ liệu duy nhất |

Trade-off khó nhất, và cũng là chỗ hay bị né: **giữ ranh giới "không bao giờ cắt" có thể khiến bạn bị
đánh giá là cứng nhắc**. Một lead nói "không" với việc bỏ audit trail trong hệ thống thanh toán có
thể bị nhìn là không hợp tác, trong khi người đồng ý mọi thứ được coi là "làm được việc" — cho đến
khi có sự cố. Cách giảm chi phí chính trị của việc giữ ranh giới: đừng nói "không được", hãy nói
"cái đó tôi không có quyền cắt một mình — nếu chúng ta muốn cắt, cần [tên người có thẩm quyền
compliance] đồng ý bằng văn bản." Câu này chuyển một cuộc đối đầu cá nhân thành một câu hỏi về quy
trình, và trong hầu hết trường hợp nó kết thúc cuộc tranh luận.

### Real-world Scenarios

**Tình huống A — Campaign 9.9: ba tuần, năm tuần việc.**

*Bối cảnh.* Sàn thương mại điện tử, team 9 người (6 BE, 2 FE, 1 QA), Tuấn là Tech Lead, Trang là PO.
Phạm vi campaign: flash sale theo khung giờ, voucher đơn, voucher combo (mua A tặng B), và dashboard
theo dõi cho seller. Ước lượng còn lại: 5 tuần P50, 7 tuần P90 (số minh hoạ). Còn 3 tuần.

*Các lựa chọn.*

1. Không nói gì, cả team làm tăng ca ba tuần, hy vọng kịp.
2. Báo lên rằng không kịp, đề nghị dời ngày.
3. Thêm hai người từ team khác.
4. Đưa ba phương án cho Trang và Head of Growth chọn.

*Trade-off.* Lựa chọn 1 là mặc định nếu lead không hành động, và nó có một sức hấp dẫn thật: không
phải đối mặt với cuộc trò chuyện khó, và có xác suất khác không là sẽ kịp. Nhưng nó chuyển toàn bộ
rủi ro sang chất lượng bên trong và sức khoẻ của team, và nó lấy đi của stakeholder ba tuần mà họ có
thể dùng để ứng phó — nếu Growth biết sớm, họ có thể đổi thông điệp quảng cáo cho phù hợp phạm vi
nhỏ hơn. Lựa chọn 2 đúng về tính trung thực nhưng sai về phân loại: ngày 9.9 là deadline cứng bên
ngoài, đề nghị dời là đề nghị bỏ campaign. Lựa chọn 3 vi phạm Brooks's Law trong điều kiện xấu nhất
(dưới 4 tuần, công việc tập trung ở module pricing).

*Quyết định.* Tuấn chọn 4, gửi văn bản ba phương án trước cuộc họp 24 giờ. Trong họp, câu hỏi của
Head of Growth là câu hỏi hữu ích nhất: "Voucher combo đóng góp bao nhiêu phần trăm doanh thu dự
kiến?" Không ai có con số chắc chắn, nhưng ước tính dựa trên campaign 6.6 trước đó là dưới 15%.
Quyết định: phương án B, có điều chỉnh — cắt voucher combo và dashboard seller, nhưng giữ một phần
nhỏ của dashboard (một bảng số liệu tĩnh cập nhật hàng giờ, làm trong 2 ngày thay vì 5 ngày) vì bộ
phận vận hành seller cần con số để trả lời seller trong campaign.

*Hậu quả.* Campaign chạy đúng ngày với phạm vi nhỏ hơn. Doanh thu 72 giờ đạt 91% so với dự báo ban
đầu (số minh hoạ) — phần thiếu đúng bằng phần voucher combo. Không có incident nào về tính giá. Team
làm thêm giờ ở tuần cuối nhưng không quá 2 giờ mỗi ngày và có nghỉ bù. Voucher combo ra vào tháng 10
với chất lượng đầy đủ và được dùng cho campaign 11.11.

*Bài học.* (a) Câu hỏi mở khoá tình huống này không phải câu hỏi kỹ thuật mà là "phần nào của phạm
vi tạo ra phần lớn giá trị" — và người trả lời được câu đó không phải engineering. Việc của lead là
đặt câu hỏi đó vào đúng chỗ. (b) Ba phương án bằng văn bản gửi trước 24 giờ thay đổi bản chất cuộc
họp: từ "engineering xin xỏ" thành "stakeholder ra quyết định có thông tin". (c) Phương án cuối cùng
không phải một trong ba phương án ban đầu — đó là dấu hiệu quy trình hoạt động đúng, vì mục tiêu của
ba phương án là mở không gian lựa chọn, không phải ép chọn một trong ba.

**Tình huống B — cùng một quyết định, ba góc nhìn.**

Sự việc: trong tuần cuối trước 9.9, Quân (Senior BE) bỏ viết test cho phần tính giá voucher đơn để
kịp, và không nói với ai.

*Nhìn từ IC (Quân).* "Còn bốn ngày, phần logic tính giá tôi đã viết ba lần ở các dự án trước, tôi
tự tin nó đúng. Viết test cho nó tốn một ngày rưỡi vì phải dựng fixture cho cấu hình khuyến mãi.
Nếu tôi nói ra, sẽ có một cuộc họp và tôi mất thêm nửa ngày. Tôi sẽ test tay kỹ." Quân đang tối ưu
đúng theo thông tin và động lực của mình. Anh không sai về mặt kỹ thuật — anh sai ở chỗ đã ra một
quyết định thuộc thẩm quyền của người khác, một mình, và không để lại dấu vết.

*Nhìn từ Tech Lead (Tuấn).* Vấn đề với Tuấn không phải Quân bỏ test — trong phương án B đã chốt,
việc điều chỉnh mức test ở vùng ít rủi ro là chấp nhận được. Vấn đề là **Tuấn không biết**, nên bức
tranh rủi ro của anh sai, nên khi Trang hỏi "chúng ta có yên tâm về tính giá không" anh trả lời
"có" một cách trung thực nhưng không đúng. Can thiệp của Tuấn không nên là cấm Quân tự quyết, mà là
tạo một kênh chi phí cực thấp để những quyết định như vậy hiện ra: một dòng trong channel, một
ticket nợ ba dòng. Nếu chi phí báo cáo cao hơn chi phí im lặng, im lặng sẽ thắng.

*Nhìn từ Manager (Hà).* Ở tầng này, một Senior tự cắt test và không báo trong tuần cuối là **tín
hiệu về cơ chế, không phải về con người**. Nó nói rằng: tổ chức chưa làm cho việc ghi nợ trở nên rẻ
và an toàn; và có thể team đã học được rằng nêu vấn đề trong giai đoạn gấp sẽ dẫn tới một cuộc họp
chứ không dẫn tới một sự trợ giúp. Can thiệp của Hà không nằm ở tuần này mà ở chu kỳ sau: đưa "ghi
nợ" vào Definition of Done như một checkbox 15 giây, và trong retro sau campaign, hỏi thẳng có bao
nhiêu quyết định cắt đã xảy ra mà không ai ghi — với cam kết rõ ràng rằng câu trả lời không dẫn tới
hệ quả tiêu cực cho ai.

Điểm mấu chốt: cùng một hành động, IC đọc là tối ưu hợp lý dưới ràng buộc, Tech Lead đọc là lỗ hổng
thông tin làm sai bức tranh rủi ro, Manager đọc là bằng chứng rằng chi phí của việc nói ra đang cao
hơn chi phí của việc im lặng. Chỉ can thiệp ở tầng thứ ba mới ngăn được lần sau.

### Best Practices

- **Phân loại deadline trước khi bàn cách kịp.** Câu hỏi đầu tiên là "nếu trễ hai tuần thì thực sự
  mất gì, và ai chịu?" — không phải "làm sao kịp". Câu trả lời quyết định toàn bộ chiến lược.
- **Luôn đưa ba phương án khác nhau về biến bị cắt**, bằng văn bản, gửi trước cuộc họp ít nhất 24
  giờ. Một phương án là đẩy quyết định; ba phương án là trang bị cho người quyết.
- **Cắt scope trước, cắt chất lượng bên trong sau cùng.** Chi phí của cắt scope biết trước và dừng
  lại; chi phí của cắt chất lượng bên trong không biết trước và tiếp tục tăng.
- **Giữ danh sách "không bao giờ cắt" ngắn và tuyệt đối**, và nói ra nó từ đầu dự án chứ không phải
  lúc bị ép. Bốn mục tối thiểu: tính đúng của tiền, kiểm soát truy cập, khả năng rollback, audit
  trail nếu thuộc phạm vi compliance.
- **Ghi nợ tại thời điểm vay, không phải sau.** Ticket có chủ, có ước lượng lãi, có ngày trả, có chỗ
  trong capacity. Mười lăm phút này là phần sinh lợi cao nhất của toàn bộ quy trình.
- **Làm cho việc báo cáo một quyết định cắt rẻ hơn việc im lặng.** Một dòng trong channel, một
  checkbox trong Definition of Done. Nếu báo cáo dẫn tới một cuộc họp, mọi người sẽ ngừng báo cáo.
- **Nếu crunch, đặt ngày kết thúc trước khi bắt đầu**, nói rõ với cả team, và có bù sau. Crunch
  không có ngày kết thúc là một cách mất người có tài liệu đầy đủ.
- **Sau mỗi lần giao hàng dưới áp lực, chạy một retro riêng về quyết định cắt**: đã cắt gì, ai
  quyết, nợ đã ghi chưa, cái gì lần sau nên phát hiện sớm hơn. Đây là dữ liệu duy nhất giúp lần đàm
  phán sau có cơ sở.
- **Chuyển "không được" thành "cần ai đồng ý".** Khi phải giữ ranh giới, hãy chuyển câu chuyện từ
  đối đầu cá nhân sang câu hỏi về thẩm quyền — nó vừa giữ được ranh giới vừa giữ được quan hệ.

### Anti-patterns

- **Crunch không giới hạn.** Cơ chế phá hoại theo ba giai đoạn: tuần một tăng sản lượng thật; tuần
  hai chất lượng quyết định giảm và tỉ lệ lỗi tăng, nên sản lượng ròng bắt đầu giảm; từ tuần ba trở
  đi sản lượng ròng âm — mỗi giờ thêm tạo ra nhiều việc sửa hơn giá trị nó tạo ra. Hậu quả kéo dài
  nhất không phải là lỗi mà là mất người: người giỏi nhất có nhiều lựa chọn nhất, nên họ đi trước.
  Dấu hiệu sớm: tuần thứ hai liên tiếp có người làm quá 10 giờ; số lỗi phát sinh trong tuần cao hơn
  số lỗi đóng.
- **Im lặng cắt chất lượng.** Đã phân tích ở trên. Dấu hiệu sớm: sau một giai đoạn gấp, không có
  ticket nợ nào được tạo — điều này gần như chắc chắn có nghĩa là nợ đã phát sinh nhưng không được
  ghi, chứ không phải không có nợ.
- **Thêm người vào dự án đang trễ.** Ngoài Brooks's Law, có một tác hại phụ ít được nói: nó gửi tín
  hiệu rằng vấn đề là thiếu nhân lực, nên nguyên nhân thật (ước lượng sai, phạm vi trôi, phụ thuộc
  không được quản lý) không được xem xét, và sẽ lặp lại ở dự án sau.
- **Hứa "sẽ dọn sau" mà không có ngày và không có chỗ trong capacity.** Đây không phải một kế hoạch,
  nó là một cách kết thúc cuộc trò chuyện. Kiểm tra đơn giản: nếu bạn không nói được sprint nào và
  ai làm, thì nó sẽ không xảy ra.
- **Cắt QA thay vì cắt scope.** Cắt QA giữ nguyên phạm vi trên giấy nhưng chuyển việc phát hiện lỗi
  sang người dùng. Trong e-commerce và fintech, chi phí của một lỗi phát hiện bởi người dùng lớn hơn
  nhiều lần chi phí phát hiện nội bộ — cộng thêm chi phí xử lý khủng hoảng truyền thông mà không ai
  tính vào lúc quyết định.
- **Deadline đàm phán ngược.** Stakeholder hỏi "ba tuần có kịp không", lead nói không, stakeholder
  hỏi "thế cần gì để kịp", và cuộc trò chuyện kết thúc bằng việc team nhận ba tuần mà không có thay
  đổi thật nào về bốn biến. Cơ chế: câu hỏi "cần gì để kịp" nghe như hợp tác nhưng thực chất chuyển
  gánh nặng chứng minh sang engineering. Cách chặn: trả lời bằng ba phương án bằng văn bản, không
  trả lời miệng trong cuộc họp.
- **Ăn mừng việc kịp deadline mà không nói tới cái giá.** Nếu tổ chức chỉ ghi nhận việc giao đúng
  ngày và không bao giờ nhắc tới nợ đã vay hay giờ đã làm thêm, nó đang thưởng cho hành vi tạo ra chi
  phí ẩn — và nó sẽ có thêm hành vi đó.

### Khi nào KHÔNG nên áp dụng

- **Khi phần code sẽ bị bỏ trong vòng vài tháng.** Prototype để kiểm chứng giả thuyết thị trường,
  landing page cho một campaign một lần, script chạy một lần để di trú dữ liệu. Ở đây, chất lượng
  bên trong không có người thừa kế, nên vay nợ là quyết định đúng và ghi nợ chi tiết là chi phí
  thuần. Điều kiện đi kèm: phải có cơ chế thật để xoá nó (ngày hết hạn, feature flag, thư mục riêng
  được đánh dấu), vì "code tạm" sống sót là một trong những cách phổ biến nhất để tích tụ nợ.
- **Khi bạn không có quyền và không có kênh tới người có quyền.** Trong một số dự án ODC, phạm vi và
  ngày thuộc khách hàng, và người quản lý tài khoản không muốn engineering nói trực tiếp. Chạy quy
  trình ba phương án nội bộ rồi không gửi được đi đâu sẽ tạo ra sự bất lực có tài liệu. Việc đúng ở
  đây: gửi ba phương án cho người sở hữu quan hệ khách hàng, bằng ngôn ngữ thương mại (`02-communication.md`),
  và song song làm rõ bằng văn bản rằng rủi ro đã được nêu — đây là phần accountability bạn kiểm soát
  được.
- **Trong incident đang diễn ra.** Cơ chế lúc đó là command-and-control (`06-incident-va-metrics.md`),
  không phải đàm phán ba phương án. Việc cắt góc để mitigate nhanh là đúng; ghi nợ diễn ra trong
  postmortem, không phải trong lúc cháy.
- **Khi deadline là loại giả và bạn đã xác minh được điều đó.** Nếu không ai nhớ vì sao có ngày đó và
  không có sự kiện thật nào phía sau, chạy toàn bộ quy trình cắt là hợp thức hoá một ràng buộc không
  tồn tại. Việc đúng là truy nguồn gốc của ngày trước — thường mất một cuộc trò chuyện 15 phút và
  tiết kiệm ba tuần.
- **Với team dưới 4 người và dự án dưới hai tuần.** Chi phí của văn bản ba phương án gần bằng chi phí
  của việc làm. Ở quy mô này, cơ chế đủ là một cuộc trò chuyện 20 phút với PO và một dòng ghi lại
  trong channel — nhưng dòng ghi lại đó vẫn bắt buộc, vì nó là phần rẻ nhất và có giá trị nhất.
- **Khi chính bạn là người đặt deadline.** Nếu ngày do bạn đặt và bạn đang ép team bằng nó, việc chạy
  quy trình ba phương án với chính mình là một nghi lễ. Việc trung thực hơn là xem lại vì sao bạn đặt
  ngày đó, và điều gì trong hệ thống khuyến khích khiến bạn cần một deadline nhân tạo để tạo áp lực.

---

## Tự kiểm tra

Trả lời bằng con số, tên người, hoặc ngày cụ thể. Câu nào không trả lời được bằng dữ liệu là một chỗ
hệ thống delivery của bạn đang mù.

1. **Dự án hiện tại của bạn đang chạy theo nhịp nào, và mỗi nghi lễ trong nhịp đó tồn tại để giải
   quyết vấn đề gì?** Nếu có nghi lễ nào bạn không nêu được vấn đề nó giải quyết, hãy thử bỏ nó một
   tháng và đo.
2. **Trong sprint gần nhất, có bao nhiêu việc được thêm vào sau khi sprint đã bắt đầu, và ai đã phê
   duyệt?** Nếu con số lớn hơn 20% và không có ai phê duyệt, cơ chế bảo vệ chu kỳ của bạn không tồn
   tại.
3. **Cycle time trung vị của một thay đổi từ lúc bắt đầu code đến lúc lên production là bao nhiêu
   ngày, và flow efficiency là bao nhiêu phần trăm?** Nếu bạn chưa đo, hãy đo 20 item gần nhất trước
   khi bàn tới việc thuê thêm người.
4. **Ước lượng gần nhất bạn đưa ra cho stakeholder là một con số hay một khoảng?** Nếu là một con số,
   nó đã ngầm hứa điều gì mà bạn không kiểm soát được?
5. **Roadmap hiện tại của bạn có nói rõ cái gì sẽ KHÔNG làm trong quý này không?** Nếu không, nó là
   một danh sách mong muốn chứ chưa phải một kế hoạch.
6. **Liệt kê ba phụ thuộc bên ngoài team đang chặn hoặc sắp chặn công việc — mỗi cái có tên người,
   ngày cam kết, và ngày kiểm tra gần nhất không?** Phụ thuộc không có tên người là phụ thuộc sẽ trễ.
7. **Lần gần nhất một dự án của bạn trễ, stakeholder biết trước bao nhiêu ngày?** Nếu dưới một tuần,
   bạn đã lấy đi của họ khả năng ứng phó, và điều đó tốn uy tín nhiều hơn chính việc trễ.
8. **Sau lần giao hàng gấp gần nhất, có bao nhiêu ticket nợ kỹ thuật được tạo, và bao nhiêu cái đã
   được trả?** Nếu con số đầu là 0, nợ vẫn phát sinh — chỉ là bạn không có sổ.

---

## Liên kết chương khác

- [`00-nen-tang-leadership.md`](/series/engineering-leadership/00-nen-tang-leadership/) — Accountability quyết định ai được cắt
  scope, ai được dời ngày, và phần nào của quyết định giao hàng thuộc về lead mà không chia được.
- [`01-self-leadership.md`](/series/engineering-leadership/01-self-leadership/) — Prioritization ở cấp cá nhân và quản lý năng
  lượng; crunch có giới hạn là chủ đề chung của hai chương, nhìn từ hai phía.
- [`02-communication.md`](/series/engineering-leadership/02-communication/) — Stakeholder Management, no-surprise principle, và
  cách trình bày ba phương án cho người không kỹ thuật bằng ngôn ngữ rủi ro và chi phí.
- [`03-team-leadership.md`](/series/engineering-leadership/03-team-leadership/) — Psychological Safety quyết định việc rủi ro
  delivery có được nói ra sớm hay không; đây là biến số ẩn phía sau mọi dự án trễ muộn màng.
- [`04-decision-making.md`](/series/engineering-leadership/04-decision-making/) — Prioritization Frameworks, cost of delay, và
  ngôn ngữ tiền để định giá cả việc trễ lẫn việc vay nợ kỹ thuật.
- [`05-technical-leadership.md`](/series/engineering-leadership/05-technical-leadership/) — cơ chế lãi kép của technical debt, và
  vì sao chất lượng bên trong là biến không nên cắt trừ trường hợp đã nêu.
- [`06-incident-va-metrics.md`](/series/engineering-leadership/06-incident-va-metrics/) — action item của postmortem cạnh tranh
  capacity với roadmap; bucket technical health là chỗ quyết định chúng sống hay chết.
- [`08-hiring-va-phat-trien.md`](/series/engineering-leadership/08-hiring-va-phat-trien/) — vì sao thêm người không cứu được dự án
  trễ, và onboarding cần bao lâu trước khi một người mới tạo ra đóng góp ròng.
- [`09-to-chuc-va-scaling.md`](/series/engineering-leadership/09-to-chuc-va-scaling/) — dependency giữa các team là bài toán cấu
  trúc tổ chức; Conway's Law giải thích vì sao một số phụ thuộc không thể quản lý mà chỉ có thể thiết
  kế lại.
- [`10-case-studies.md`](/series/engineering-leadership/10-case-studies/) — case study "ship nhanh hay làm đúng" và case study dự
  án thất bại vì giao tiếp kém, phân tích đầy đủ từ bối cảnh đến bài học.
- [`12-anti-patterns.md`](/series/engineering-leadership/12-anti-patterns/) — Hero Culture, crunch, và các anti-pattern delivery
  được tổng hợp cùng dấu hiệu sớm và cách tháo gỡ.
