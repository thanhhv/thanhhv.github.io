+++
title = "Level 4B — Hiring và Phát triển con người: Interview, Onboarding, Career Ladder và Performance"
date = "2026-08-01T16:00:00+07:00"
draft = false
tags = ["engineering-leadership", "management", "career"]
series = ["Engineering Leadership & Technical Management"]
+++

# Level 4B — Hiring và Phát triển con người

Tháng 3, một công ty ODC ở Hà Nội, 45 engineer. Hà — EM của ba squad — ký offer cho một Senior Backend
sau bốn vòng phỏng vấn trải hai tuần rưỡi. Panel gồm năm người, tổng cộng khoảng chín giờ công. Ứng
viên nhận offer, vào ngày 1 tháng 4. Ngày 15 tháng 7, sau ba tháng rưỡi, Hà và Tuấn ngồi lại và thừa
nhận với nhau rằng người này sẽ không trụ được. Không phải vì kỹ thuật kém — bài take-home của anh ta
là bài tốt nhất quý đó. Mà vì trong ba tháng rưỡi, anh ta chưa một lần tự đưa ra quyết định kỹ thuật
nào mà không hỏi lại Tuấn, chưa từng đọc hết một luồng nghiệp vụ nào, và mỗi lần review PR của người
khác đều chỉ để lại đúng một comment: "LGTM".

Điều đáng chú ý không phải là lần tuyển sai. Lần tuyển sai là chuyện thống kê, sẽ xảy ra. Điều đáng
chú ý là bảng cân đối: chín giờ công cho việc quyết định nhận người này, và — đếm thật trên lịch —
khoảng bốn giờ công cho toàn bộ việc làm cho người này trở nên hiệu quả sau khi vào. Một buổi giới
thiệu công ty của HR, một buổi Tuấn dẫn qua kiến trúc trên Miro, một link tới Confluence có 180 trang
tài liệu trong đó 60 trang đã lỗi thời, và một câu quen thuộc: "Em cứ đọc đi, có gì hỏi anh."

Tỉ lệ đó — đầu tư vào việc chọn người gấp nhiều lần đầu tư vào việc làm cho người đã chọn thành công —
là mẫu hình phổ biến đến mức hầu như không ai nhận ra nó là một lựa chọn. Nó có logic ngầm: phỏng vấn
có deadline, có người bên ngoài chờ, có sự kiện rõ ràng gọi là "ra quyết định"; còn onboarding không
có deadline, không có ai bên ngoài chờ, và không có khoảnh khắc nào để nói "xong". Việc gì có deadline
sẽ ăn hết thời gian của việc không có deadline. Đó là toàn bộ cơ chế.

Thêm một tính chất nữa làm cho tầng này khác mọi tầng khác trong bộ tài liệu: quyết định về con người
gần như không thể đảo ngược với chi phí thấp. Bạn chọn sai một framework, sáu tháng sau bạn thay, đau
nhưng đo được. Bạn chọn sai một người, bạn không "thay" được — bạn phải trải qua một quy trình dài,
tốn kém về pháp lý và tinh thần, để lại vết trên cả người ra đi lẫn người ở lại. Trong chuỗi Business
Goal → **People** → Process → Technology → Execution → Feedback → Improvement → Scaling Team →
Organization, chương này chiếm trọn mắt **People**, và là mắt duy nhất mà một sai lầm không thể được
sửa bằng một quyết định kỹ thuật tốt hơn ở tầng dưới. Một team gồm sai người, với process hoàn hảo,
vẫn là một team sai.

Chương này không dạy bạn "nhìn người". Không ai nhìn người giỏi một cách đáng tin cậy, kể cả những
người tin rằng mình giỏi việc đó — đặc biệt là những người tin rằng mình giỏi việc đó. Chương này nói
về việc thiết kế các hệ thống có sai số đo được: cách xác định thật sự đang thiếu năng lực gì, cách
thiết kế vòng phỏng vấn để nó đo được cái nó tuyên bố đo, cách ra quyết định nhóm mà không bị
information cascade nuốt mất, và — phần bị bỏ quên nhiều nhất — cách vận hành vòng đời của một người
sau khi họ đã vào: onboarding, career ladder, review, promotion, xử lý underperformance, và giữ người.

Mục lục nội bộ:

1. [Xác định nhu cầu tuyển dụng — trước khi mở JD](#1-xác-định-nhu-cầu-tuyển-dụng--trước-khi-mở-jd)
2. [Interview Design](#2-interview-design)
3. [Candidate Evaluation và ra quyết định tuyển](#3-candidate-evaluation-và-ra-quyết-định-tuyển)
4. [Onboarding](#4-onboarding)
5. [Career Ladder](#5-career-ladder)
6. [Performance Review và Feedback định kỳ](#6-performance-review-và-feedback-định-kỳ)
7. [Promotion Framework](#7-promotion-framework)
8. [Xử lý underperformance](#8-xử-lý-underperformance)
9. [Retention và mất người](#9-retention-và-mất-người)
10. [Tự kiểm tra](#tự-kiểm-tra)
11. [Liên kết chương khác](#liên-kết-chương-khác)

> Mọi con số trong chương này là **số minh hoạ** — dùng để bạn thấy cách tính và cách đọc tổ hợp, không
> phải dữ liệu nội bộ của bất kỳ công ty nào. Khi dẫn nghiên cứu, chương này chỉ dẫn những nguồn đã công
> khai rộng rãi và nói rõ đó là nguồn công khai. Phần liên quan đến chấm dứt hợp đồng lao động là mô tả
> nguyên tắc quản trị, **không phải tư vấn pháp lý** — mọi bước thực thi phải phối hợp với HR và bộ phận
> pháp chế của tổ chức bạn.

---

## 1. Xác định nhu cầu tuyển dụng — trước khi mở JD

### Problem Statement

Một e-commerce Việt, 30 engineer, quý IV. Trang (PO) báo backlog đang phình: 42 story chưa làm, velocity
trung bình 18 điểm/sprint, tồn đọng tương đương 6 sprint. CTO kết luận: "Thiếu người. Tuyển thêm hai
Backend." JD mở ngày 5 tháng 10. Người đầu tiên vào ngày 20 tháng 12. Người thứ hai vào tháng 2 năm sau.

Tháng 4 năm sau, backlog vẫn 40 story, velocity 19 điểm/sprint. Team đông hơn 2 người, tồn đọng gần như
không đổi.

Khi mổ xẻ, ba sự thật hiện ra. Một: 60% thời gian của team không nằm ở việc viết feature mới mà ở việc
xử lý lỗi tích hợp với hai đối tác vận chuyển — một vấn đề về thiết kế chống lỗi, không phải về số
lượng tay. Hai: hai người mới mỗi người tiêu khoảng 8 giờ/tuần của senior trong 10 tuần đầu, tức là
trong quý đó team **mất** năng lực ròng. Ba: trong 42 story, 15 story chưa từng được ai xác nhận là còn
cần thiết; khi Trang rà lại, 9 story bị bỏ.

Đây là hình dạng chuẩn của lỗi ở bước này: **nhu cầu được phát biểu bằng số ghế thay vì bằng năng lực
còn thiếu**, và tuyển dụng được chọn mặc định thay vì được chọn sau khi so sánh với các phương án khác.
Hậu quả quan sát được: thời gian tới khi năng lực thực sự tăng bị kéo dài 4–6 tháng, chi phí cố định
tăng vĩnh viễn, và nguyên nhân gốc vẫn nguyên vẹn.

Dấu hiệu bạn đang ở trong lỗi này, đọc được ngay hôm nay:

- JD hiện tại của team bạn mô tả *công nghệ* ("Golang, Kafka, 3 năm kinh nghiệm") chứ không mô tả *kết
  quả* ("chịu trách nhiệm giảm p95 latency của luồng thanh toán").
- Bạn không trả lời được câu: "Nếu người này vào và làm tốt, sau 12 tháng cái gì khác đi?"
- Con số headcount xin được quyết định trước khi phân tích năng lực thiếu, không phải sau.

### First Principles

**Thiếu năng lực có bốn cách giải quyết, tuyển là một trong bốn — và thường là cách đắt nhất, chậm nhất.**

| Cách | Thời gian tới khi có tác dụng | Chi phí | Reversible? |
|---|---|---|---|
| Tuyển người mới | 4–7 tháng (tìm + notice + onboard tới net-positive) | Cao, cố định, vĩnh viễn | Rất khó |
| Đào tạo / chuyển vai người đang có | 1–3 tháng | Trung bình, chủ yếu là thời gian mentor | Dễ |
| Cắt phạm vi (làm ít hơn, làm khác đi) | Tức thì | Gần bằng 0 về tiền, cao về chính trị | Dễ |
| Mua công cụ / thuê dịch vụ | 2–8 tuần | Chi phí vận hành, thường thấp hơn 1 headcount | Trung bình |

Ba dòng dưới không phải là "mẹo tiết kiệm". Chúng là các phương án phải được **loại trừ có lý do** trước
khi chọn dòng đầu. Nếu bạn không viết được một câu giải thích vì sao ba phương án kia không đủ, bạn chưa
phân tích nhu cầu — bạn mới chỉ cảm thấy quá tải.

**Chi phí thật của một lần tuyển sai.** Đây là con số hầu như không bao giờ được tính, vì mỗi thành phần
nằm ở một ngân sách khác nhau. Ghi ra một chỗ (số minh hoạ, cho một Senior BE ở thị trường Việt Nam,
đơn vị: tháng-lương của chính vị trí đó):

```
CHI PHÍ MỘT LẦN TUYỂN SAI — Senior BE, phát hiện sau 5 tháng
(số minh hoạ, đơn vị = tháng lương của vị trí đó)

1. Lương + phụ cấp + BHXH đã trả trong 5 tháng           ≈ 6.0
2. Thời gian phỏng vấn của panel (5 người × ~9h)         ≈ 0.5
3. Chi phí tuyển (agency/JD/sourcing, nếu có)            ≈ 1.0–2.0
4. Onboarding: thời gian senior bị tiêu (8h/tuần × 10w)  ≈ 0.8
5. Chi phí cơ hội: ghế bị chiếm, việc không được làm      ≈ 3.0–6.0  ← lớn nhất, hay bị bỏ
6. Rework: code phải sửa/viết lại sau khi người đó đi     ≈ 0.5–2.0
7. Chi phí chấm dứt (trợ cấp, thủ tục, thời gian HR/lead) ≈ 1.0–2.0
8. Tác động tinh thần team: khó lượng hoá, biểu hiện ở
   việc người khác gánh việc + niềm tin vào quy trình      —
------------------------------------------------------------
TỔNG ≈ 12–19 tháng lương  (tức 1.0–1.6 năm lương của vị trí)
```

Dòng 5 là dòng quan trọng nhất và là dòng duy nhất không xuất hiện trong bất kỳ báo cáo tài chính nào.
Khi một ghế bị chiếm bởi người không phù hợp, bạn không mất "0 đóng góp" — bạn mất **đóng góp của người
đúng lẽ ra đã ngồi đó**, cộng với thời gian quản lý bị hút vào.

Hệ quả suy ra trực tiếp từ bảng trên: nếu chi phí một lần tuyển sai bằng 12–19 tháng lương, còn chi phí
để một ghế trống thêm hai tháng bằng khoảng 2 tháng đóng góp (thường thấp hơn vì team hấp thụ được một
phần), thì **bất đối xứng chi phí nghiêng mạnh về phía không tuyển khi phân vân**. Đây không phải là
tuyên ngôn đạo đức về "chuẩn cao", nó là số học. Chúng ta sẽ dùng lại kết luận này ở chủ đề 3.

**Vì sao nhu cầu được phát biểu sai.** Ba cơ chế:

1. *Ảo giác tuyến tính về năng lực.* Trực giác nói: 5 người làm được X thì 7 người làm được 1.4X. Thực
   tế, mỗi người thêm vào làm tăng số kênh giao tiếp theo n(n-1)/2, tăng Cognitive Load của lead, và
   tiêu năng lực của người đang có trong 2–3 tháng. Brooks đã mô tả cơ chế này công khai từ *The
   Mythical Man-Month* (1975): thêm người vào một dự án đang trễ làm nó trễ thêm.
2. *Headcount là loại tiền dễ xin nhất.* Trong nhiều tổ chức, xin 1 headcount dễ hơn xin ngân sách công
   cụ tương đương, vì headcount thuộc "đầu tư vào con người" còn công cụ thuộc "chi phí". Điều này bóp
   méo lựa chọn theo hướng có hệ thống.
3. *Quá tải cảm nhận không phân biệt được nguyên nhân.* Team mệt vì thiếu người, vì việc bị ngắt quãng
   liên tục, vì phạm vi phình, hay vì hệ thống fragile — cả bốn đều cho cùng một cảm giác. Cảm giác
   không phải là chẩn đoán.

### Mental Model

**Theory of Constraints áp vào năng lực team.** Một hệ thống chỉ nhanh bằng nút thắt của nó. Trước khi
thêm công suất, phải xác định nút thắt nằm ở đâu. Trong một team engineering, nút thắt thường nằm ở một
trong sáu chỗ, và chỉ chỗ số 1 mới được giải bằng tuyển:

```
[Ý tưởng] → [Làm rõ yêu cầu] → [Thiết kế] → [Code] → [Review] → [QA] → [Deploy] → [Vận hành]
                  ↑                              ↑        ↑         ↑                  ↑
              nút thắt PO/BA              (1) tay code  (2) người   (3) môi trường   (4) toil
                                                        review đủ    test            vận hành
                                                        thẩm quyền
```

Nếu nút thắt là (2) — chỉ có hai người đủ thẩm quyền review và họ là bottleneck — thì tuyển thêm hai
junior làm nút thắt **tệ hơn**, vì tăng lượng PR đổ vào cùng hai người đó. Nếu nút thắt là (4) — 60%
thời gian dành cho toil vận hành — thì đầu tư vào automation cho tác dụng nhanh hơn và rẻ hơn tuyển.
Chỉ khi nút thắt thực sự là năng lực sản xuất và bạn đã hết cách nâng thông lượng của nút thắt hiện tại
thì tuyển mới là câu trả lời đúng.

**Little's Law để kiểm tra bằng số.** Thời gian chờ trung bình = số việc trong hệ thống / thông lượng.
Nếu backlog 42 story, thông lượng 6 story/sprint, thì thời gian chờ trung bình của một story mới là 7
sprint. Tăng thông lượng bằng cách thêm người chỉ có tác dụng nếu người thêm vào thực sự tăng thông
lượng của nút thắt — mà điều đó thường sai trong 2 quý đầu. Giảm số việc trong hệ thống (cắt phạm vi)
cho tác dụng **ngay lập tức** lên cùng một chỉ số. Đây là lý do toán học vì sao "cắt phạm vi" luôn phải
được xét trước "tuyển thêm", chứ không phải vì lý do đạo đức tinh gọn.

### Practical Framework

Quy trình sáu bước, chạy được trong hai buổi làm việc.

**Bước 1 — Phát biểu nhu cầu bằng kết quả, không bằng ghế.**

Viết đúng một câu theo mẫu: *"Trong 12 tháng tới, chúng ta cần [kết quả đo được] mà hiện tại không đạt
được vì [năng lực còn thiếu cụ thể]."*

- Sai: "Cần 2 Backend vì team quá tải."
- Đúng: "Trong 12 tháng tới, luồng thanh toán cần chịu được 5.000 TPS ở peak campaign với p99 < 800ms.
  Hiện tại không ai trong team đã từng thiết kế và vận hành hệ thống ở mức tải này, và ba lần scale
  trước đều xử lý bằng cách tăng instance cho tới khi DB nghẽn."

Câu thứ hai gợi ra một loại người rất khác câu thứ nhất — và cũng gợi ra rằng có thể phương án đúng là
thuê một consultant 3 tháng chứ không phải tuyển full-time.

**Bước 2 — Bảng năng lực còn thiếu (capability gap), không phải bảng đầu người.**

| Năng lực cần cho 12 tháng tới | Ai đang có | Mức đủ? | Rủi ro tập trung | Cách lấp khả dĩ |
|---|---|---|---|---|
| Thiết kế hệ thống chịu tải cao | Không ai | Không | — | Tuyển Senior / thuê consultant |
| Kafka vận hành production | Quân | Vừa đủ | **Bus factor = 1** | Đào tạo Nam + Vy (2 tháng) |
| Tích hợp đối tác vận chuyển | Khoa, Duy | Đủ | OK | Không cần |
| Mobile release pipeline | Sơn | Vừa đủ | Bus factor = 1 | Mua dịch vụ CI có sẵn |
| Data pipeline báo cáo | Không ai | Không | — | Cắt phạm vi: dùng BI tool |

Bảng này thường tạo ra một phát hiện đáng giá hơn cả quyết định tuyển: **rủi ro bus factor = 1**. Nhiều
tổ chức tuyển thêm người vì thấy "quá tải", trong khi rủi ro thật của họ là bốn hệ thống trọng yếu mỗi
cái chỉ một người biết. Hai vấn đề khác nhau, hai cách giải khác nhau.

**Bước 3 — Loại trừ ba phương án còn lại, bằng văn bản.**

```
QUYẾT ĐỊNH: Tuyển 1 Senior BE (Distributed Systems)

Đã xét và loại:
- Đào tạo nội bộ: Quân có nền tảng nhưng cần ~9 tháng để tự tin thiết kế ở mức tải này;
  campaign lớn rơi vào tháng thứ 5 → không kịp. Vẫn sẽ đào tạo song song.
- Cắt phạm vi: đã cắt (bỏ tính năng đặt trước, hoãn multi-currency). Phần còn lại
  là ràng buộc doanh thu, không cắt tiếp được.
- Mua công cụ: managed Kafka đã dùng. Không có công cụ nào thay được việc thiết kế
  lại luồng ghi.

Nếu không tuyển: rủi ro là campaign tháng 11 sập ở peak; ước lượng ảnh hưởng doanh thu
1 ngày downtime ≈ [số minh hoạ]. Đây là lý do chấp nhận chi phí headcount vĩnh viễn.
```

Ba dòng "đã xét và loại" này có một tác dụng phụ quan trọng: khi bạn trình lên CTO hoặc founder, nó
biến cuộc trò chuyện từ "xin thêm người" (dễ bị từ chối theo cảm tính ngân sách) thành "đã phân tích
bốn phương án, đây là phương án ít tệ nhất" (khó từ chối mà không đưa ra phương án thứ năm).

**Bước 4 — Role Scorecard.** Đây là artifact trung tâm. Nó là đầu vào cho JD, cho thiết kế vòng phỏng
vấn (chủ đề 2), cho quyết định tuyển (chủ đề 3), và cho kế hoạch 30/60/90 (chủ đề 4). Một scorecard
viết tốt tiết kiệm nhiều thời gian hơn bất kỳ công cụ tuyển dụng nào.

```
ROLE SCORECARD — Senior Backend Engineer (Payments Platform)
Ngày viết: __/__/____    Hiring Manager: Hà (EM)    Panel owner: Tuấn (Tech Lead)

A. SỨ MỆNH (1 câu)
   Làm cho luồng thanh toán chịu được tải campaign mà không cần người trực đêm.

B. KẾT QUẢ MONG ĐỢI TRONG 12 THÁNG (đo được, tối đa 5 dòng)
   1. p99 latency luồng thanh toán < 800ms ở 5.000 TPS, chứng minh bằng load test định kỳ.
   2. Không còn incident Sev1 do nghẽn DB ở campaign (baseline hiện tại: 3 lần/năm).
   3. Ít nhất 2 engineer khác trong team tự vận hành được Kafka pipeline (giảm bus factor).
   4. Có ADR cho thiết kế idempotency của toàn bộ luồng ghi, được team review và áp dụng.
   5. Runbook cho 5 kịch bản sự cố thanh toán hàng đầu, đã diễn tập ít nhất 1 lần.

C. NĂNG LỰC BẮT BUỘC (mỗi năng lực phải map được sang ít nhất 2 vòng phỏng vấn)
   C1. Thiết kế hệ thống phân tán dưới ràng buộc consistency  → vòng System Design, vòng Deep Dive
   C2. Debug hiệu năng ở tầng DB/queue có phương pháp         → vòng Take-home, vòng Deep Dive
   C3. Viết được và bảo vệ được quyết định kỹ thuật bằng văn bản → vòng Take-home (ADR), vòng Behavioral
   C4. Nâng năng lực người khác (mentor, review có nội dung)  → vòng Behavioral, vòng Bar Raiser

D. KHÔNG CẦN (ghi rõ để panel không tự nâng chuẩn)
   - Không cần kinh nghiệm domain fintech; domain học được trong 6 tuần.
   - Không cần từng làm quản lý.
   - Không cần frontend.
   - Không cần tiếng Anh giao tiếp trôi chảy (team nội địa; đọc hiểu tài liệu là đủ).

E. DEAL-BREAKER (loại thẳng, không thương lượng)
   - Không có bằng chứng nào về việc từng vận hành hệ thống mình viết trong production.
   - Trong Behavioral, mọi thất bại đều được quy cho người khác.

F. SENIORITY & NGÂN SÁCH
   Bậc mục tiêu: L4 (xem Career Ladder, chủ đề 5). Dải lương: [số minh hoạ].
   Nếu ứng viên ở L3 nhưng có quỹ đạo tốt: cân nhắc, nhưng phải sửa lại kỳ vọng ở mục B.
```

Mục **D — Không cần** là mục hay bị bỏ nhất và có giá trị cao nhất. Không có nó, mỗi người trong panel
sẽ tự thêm tiêu chí của riêng mình vào đầu, và bạn kết thúc với một "ứng viên lý tưởng" mà không ai
định nghĩa, không ai tồn tại, và ghế trống thêm ba tháng.

**Bước 5 — Quyết định seniority mix.** Một team không phải là một tập hợp các cá nhân giỏi; nó là một
cấu trúc. Bảng tham chiếu (số minh hoạ, cho team 8 người làm product):

| Cấu hình | Ưu | Nhược | Phù hợp khi |
|---|---|---|---|
| 5 Senior + 3 Mid, 0 Junior | Thông lượng cao ngay, ít cần mentor | Chi phí cao; không ai muốn làm việc nhàm; khó giữ vì thiếu đường lên | Deadline sát, sản phẩm phức tạp, giai đoạn ngắn |
| 2 Senior + 4 Mid + 2 Junior | Chi phí hợp lý, có đường phát triển, senior có đòn bẩy | Cần 20–30% thời gian senior cho mentor; chậm hơn trong 6 tháng đầu | Team ổn định, có kế hoạch 2+ năm |
| 1 Senior + 6 Junior | Rẻ | Senior thành nút thắt review; chất lượng dao động mạnh; senior kiệt sức rồi nghỉ | Gần như không bao giờ, trừ khi bài toán rất đồng dạng và có khuôn mẫu chặt |

Quy tắc thực dụng: **mỗi Senior có thể mentor hiệu quả tối đa 2 người ít kinh nghiệm hơn cùng lúc** mà
không đánh mất năng lực sản xuất của chính họ. Nếu tỉ lệ vượt quá đó, bạn không có team, bạn có một
trung tâm đào tạo mà không ai được trả tiền để dạy.

**Bước 6 — Đặt ngày kiểm tra lại giả định.** Ghi vào lịch: sau 6 tháng kể từ ngày người mới vào, mở lại
scorecard, đối chiếu mục B. Không có bước này, không bao giờ có feedback loop cho chất lượng quyết định
tuyển, và tổ chức lặp lại cùng một sai lầm mãi mãi.

### Trade-off

**Tuyển Senior vs xây từ Mid.** Senior cho thông lượng nhanh, ít cần mentor, mang theo mẫu hình từ nơi
khác — nhưng đắt, khó tìm ở thị trường Việt (dải thật sự đạt chuẩn Senior mỏng hơn số CV ghi "Senior"
rất nhiều), và có xu hướng rời đi nhanh hơn nếu không có bài toán đủ khó. Mid được nâng lên rẻ hơn,
trung thành hơn, hiểu domain sâu hơn — nhưng cần 9–18 tháng và cần một Senior có thời gian thật để
mentor, thứ mà hầu hết tổ chức tuyên bố có mà thực tế không có. Điều kiện nghiêng: nếu bài toán cần
**mẫu hình chưa từng có trong tổ chức** (ví dụ: chưa ai từng làm distributed transaction), phải tuyển
Senior — không thể tự sinh ra kiến thức mình không có. Nếu bài toán chỉ cần **nhiều hơn cùng loại việc
đang làm tốt**, nâng Mid rẻ hơn và ổn định hơn.

**Tuyển sớm (trước khi thật sự nghẽn) vs tuyển muộn (khi đã nghẽn).** Tuyển sớm cho phép onboard thong
thả, người mới đạt net-positive đúng lúc nhu cầu tới — nhưng bạn trả lương cho công suất chưa dùng và
có rủi ro nhu cầu không đến (roadmap đổi). Tuyển muộn tiết kiệm tiền nhưng bạn onboard người mới trong
lúc khủng hoảng, tức là onboard tệ, tức là tăng xác suất mất người. Điều kiện nghiêng: nếu chu kỳ
tuyển + onboard của bạn dài hơn horizon dự báo roadmap đáng tin (ở startup thường là 1 quý), tuyển sớm là
đánh cược vào một dự báo bạn biết là không đáng tin — nghiêng về tuyển muộn nhưng đầu tư mạnh vào tốc
độ onboarding. Ở tổ chức có roadmap 12 tháng ổn định (fintech, ngân hàng), nghiêng về tuyển sớm.

**Chuẩn cao vs lấp ghế nhanh.** Xem phân tích số học ở First Principles; xem thêm chủ đề 3 để có script
xử lý áp lực từ chính panel.

### Real-world Scenarios

**Tình huống A — Áp lực headcount cuối năm (product startup, Series A, 35 engineer).**

Ngày 20 tháng 11, Head of Ops gửi tin nhắn cho Hà: "Ngân sách headcount 2026 chốt trước 15/12. Năm nay
mình không dùng hết 3 slot, sang năm bị cắt. Chị cứ xin đủ 3 đi, thiếu thì tính sau."

Đây là một trong những cơ chế bóp méo mạnh nhất và ít được nói ra nhất ở các tổ chức Việt Nam có quy
trình ngân sách theo năm: **headcount không dùng thì mất, nên có động cơ dùng bất kể có cần hay không**.
Kết quả điển hình: JD được mở vội trong tháng 12, scorecard viết trong 20 phút, chuẩn bị hạ dần từ
tháng 2 khi áp lực "đã xin rồi mà không tuyển được" bắt đầu.

Cách xử lý sai, và vì sao nó hỏng: xin đủ 3 slot rồi tuyển dần. Hỏng vì slot đã được duyệt tạo ra một
cam kết chính trị — sang tháng 4, câu hỏi từ trên xuống không còn là "team có cần không" mà là "sao xin
3 mà mới tuyển 1", và câu hỏi đó chỉ có một cách trả lời dễ chịu là hạ chuẩn.

Cách xử lý tốt hơn: tách quyết định *giữ quyền* khỏi quyết định *dùng quyền*, và ghi rõ điều kiện kích
hoạt.

> **Hà:** Em xin 3 slot, nhưng em muốn ghi rõ trong kế hoạch điều kiện kích hoạt từng slot.
> Slot 1 kích hoạt ngay tháng 1 — đây là năng lực distributed systems mình đang thiếu, đã phân tích rồi.
> Slot 2 kích hoạt khi module thanh toán mới được duyệt vào roadmap — nếu không được duyệt thì không
> tuyển. Slot 3 là dự phòng cho attrition, chỉ dùng khi có người nghỉ.
> Anh giúp em ghi ba điều kiện này vào file ngân sách được không? Để đến tháng 4 mình không phải giải
> thích lại vì sao mới tuyển 1 người.

Câu cuối là câu quan trọng nhất. Nó không xin thêm gì, nó chỉ chuyển một hiểu ngầm thành văn bản — và
văn bản đó là thứ bảo vệ chuẩn tuyển của bạn bốn tháng sau, khi không ai còn nhớ cuộc trò chuyện này.

**Tình huống B — "Senior 3 năm kinh nghiệm" (ODC, khách hàng Nhật).**

Khách hàng yêu cầu bổ sung "2 Senior Java" vào team, định nghĩa Senior trong hợp đồng khung là "3+ năm
kinh nghiệm Java". Sales đã ký. Tuấn phải tuyển.

Vấn đề: 3 năm là ngưỡng đủ cho Mid ở hầu hết bối cảnh, không đủ cho Senior nếu Senior nghĩa là tự chủ
thiết kế và chịu trách nhiệm end-to-end. Nhưng hợp đồng đã định nghĩa thế, rate đã tính theo đó. Đây là
title inflation ở dạng có ràng buộc thương mại — không sửa được bằng cách tranh luận về ngữ nghĩa.

Cách xử lý thực tế: chấp nhận nhãn trong hợp đồng, nhưng **tách nhãn khỏi scorecard nội bộ**. Tuấn viết
scorecard theo kết quả cần thật (mục B), tuyển theo scorecard đó, và ghi vào kế hoạch nhân sự nội bộ
bậc thật của người được tuyển theo ladder công ty (chủ đề 5). Người này được gọi là "Senior" với khách,
được trả và được kỳ vọng theo bậc thật, và có lộ trình rõ để lên bậc thật. Điều tuyệt đối phải tránh là
để nhãn hợp đồng chảy ngược vào hệ thống nội bộ — vì lúc đó ladder của bạn bị định nghĩa bởi phòng
Sales của khách hàng, và mọi so sánh nội bộ về công bằng sụp đổ.

Rủi ro còn lại phải quản lý: nếu người này bị khách kỳ vọng như Senior thật và không đáp ứng, hậu quả
rơi vào mối quan hệ khách hàng. Tuấn phải chủ động ghép người này vào phần công việc mà bậc thật của họ
xử lý được, và không để họ một mình đối diện với kiến trúc sư phía khách.

**Tình huống C — Cùng một sự việc, ba góc nhìn.**

Bối cảnh: team 7 người, backlog phình, Hà đề xuất tuyển 2 người. Cùng một dữ kiện, ba vai đọc khác nhau.

| | Nhìn từ IC (Vy, Mid BE) | Nhìn từ Tech Lead (Tuấn) | Nhìn từ Manager (Hà, EM) |
|---|---|---|---|
| **Điều nhìn thấy** | "Tôi làm 10 tiếng/ngày, sprint nào cũng carry-over. Rõ ràng thiếu người." | "Tôi review 14 PR/tuần, không còn thời gian code. Thêm người là thêm PR." | "Cost per engineer đang tăng, output không tăng tương ứng. Sếp sẽ hỏi." |
| **Nút thắt họ tin là thật** | Số tay code | Băng thông review và thẩm quyền quyết định | Cấu trúc team và ưu tiên |
| **Điều họ không nhìn thấy** | 9 story trong backlog không còn cần thiết; 60% thời gian team đổ vào lỗi tích hợp | Rằng chính việc mình giữ độc quyền review là nút thắt có thể phân quyền | Rằng cảm giác quá tải của IC là thật và nếu bỏ qua sẽ mất người trong 2 quý |
| **Hành động đúng cho vai đó** | Ghi lại 2 tuần thời gian thực đổ vào đâu, đưa dữ liệu chứ đưa cảm giác | Phân quyền review cho Quân và Khoa; đo lại băng thông sau 1 tháng | Chạy bảng capability gap; cắt phạm vi trước; nếu vẫn thiếu thì tuyển 1 chứ không 2 |
| **Sai lầm điển hình của vai đó** | Đẩy yêu cầu "tuyển thêm" lên như giải pháp duy nhất | Từ chối tuyển vì "người mới tốn thời gian của tôi" — tối ưu cục bộ | Duyệt 2 headcount để làm dịu team, mua sự yên ổn ngắn hạn bằng chi phí vĩnh viễn |

Điểm đáng chú ý: cả ba đều đúng về phần mình quan sát được, và cả ba đều sai nếu hành động chỉ dựa trên
phần đó. Vai trò của Hà không phải là chọn một trong ba, mà là ghép ba tầm nhìn thành một chẩn đoán —
đó là công việc thật của mắt **People** trong chuỗi.

### Best Practices

- **Viết scorecard trước khi viết JD, và viết JD từ scorecard.** Lý do: JD là văn bản marketing hướng ra
  ngoài, scorecard là văn bản quyết định hướng vào trong. Nếu chỉ có JD, panel sẽ phỏng vấn theo JD, tức
  là phỏng vấn theo tài liệu marketing.
- **Bắt buộc mục "Không cần" trong mọi scorecard.** Lý do: nó chặn hiện tượng mỗi thành viên panel âm
  thầm thêm tiêu chí của mình, thứ làm ghế trống lâu mà không ai chịu trách nhiệm.
- **Ghi lại lý do loại ba phương án còn lại.** Lý do: vừa buộc bạn thật sự xét chúng, vừa tạo tài liệu
  bảo vệ quyết định khi bị chất vấn sau này.
- **Đo bus factor trước khi đo quá tải.** Lý do: hai vấn đề này cho cùng cảm giác nhưng cần hai cách giải
  hoàn toàn khác; giải sai bài thì tiền tiêu đi mà rủi ro vẫn nguyên.
- **Giới hạn tỉ lệ mentor 1 Senior : 2 người ít kinh nghiệm hơn.** Lý do: vượt ngưỡng này thì Senior mất
  năng lực sản xuất, chất lượng review giảm, và bạn mất Senior trong 2–3 quý.
- **Đặt lịch review lại scorecard sau 6 tháng.** Lý do: đây là feedback loop duy nhất cho chất lượng
  quyết định tuyển; không có nó thì mọi cải tiến quy trình phỏng vấn đều là phỏng đoán.

### Anti-patterns

**"Tuyển theo số ghế được duyệt."** Nhu cầu được phát biểu là "3 headcount", không ai viết ra 3 người
này để làm gì. Cơ chế phá hoại: khi không có tiêu chí kết quả, panel mặc định đo "giỏi chung chung", tức
là đo mức độ ứng viên giống người phỏng vấn. Dấu hiệu sớm: trong buổi debrief, không ai trích dẫn được
một dòng nào từ scorecard, vì không có scorecard.

**"JD copy từ công ty khác."** Lấy JD của một công ty lớn về sửa tên. Cơ chế phá hoại: JD đó được viết
cho một bối cảnh khác, thường liệt kê 12 công nghệ, và tạo ra hai hiệu ứng cùng lúc — ứng viên phù hợp
tự loại mình vì thấy thiếu 4/12 mục, ứng viên biết cách đọc JD thì bỏ qua hoàn toàn. Dấu hiệu sớm: JD
có mục "Yêu cầu" dài hơn mục "Bạn sẽ làm gì".

**"Tuyển người giống người vừa nghỉ."** Quân nghỉ, mở JD tìm "một Quân khác". Cơ chế phá hoại: bạn đang
tuyển cho cấu hình team của quá khứ, không phải cho nhu cầu 12 tháng tới; và bạn bỏ lỡ cơ hội duy nhất
để điều chỉnh cấu trúc team. Dấu hiệu sớm: scorecard mô tả công việc hiện Quân đang làm chứ không mô tả
kết quả cần đạt.

**"Headcount cuối năm không dùng thì phí."** Xem Tình huống A. Cơ chế phá hoại: động cơ ngân sách thay
thế động cơ năng lực; chất lượng tuyển bị hạ một cách hệ thống vào quý I hằng năm. Dấu hiệu sớm: số JD
mở trong tháng 12 và tháng 1 cao bất thường so với phần còn lại của năm.

**"Chờ ứng viên hoàn hảo."** Ngược lại của anti-pattern trên, cũng có hại. Scorecard có 14 năng lực bắt
buộc, không ai qua nổi, ghế trống 8 tháng, team kiệt sức và mất 2 người. Cơ chế: chi phí ghế trống là chi
phí ẩn, phân tán lên nhiều người, nên không ai chịu trách nhiệm cho nó; còn chi phí tuyển sai là chi phí
hiện, có tên người ký. Bất đối xứng trách nhiệm dẫn tới quá thận trọng. Dấu hiệu sớm: scorecard có nhiều
hơn 5 năng lực bắt buộc, hoặc mục "Không cần" trống.

### Khi nào KHÔNG nên áp dụng

**Khi tổ chức quá nhỏ và bạn đang tuyển người số 3.** Ở team 2–5 người, mỗi người làm mọi thứ, và
scorecard chi tiết theo năng lực là công cụ sai — bạn không tuyển cho một vai, bạn tuyển cho một giai
đoạn. Cái cần thay thế: một câu về loại bài toán sẽ giao trong 6 tháng, và một đánh giá thẳng thắn về
khả năng làm việc trong điều kiện không có quy trình. Áp bộ máy scorecard 6 bước vào đây tạo ra nghi lễ
không có tác dụng đo lường.

**Khi tuyển thay thế khẩn cấp một vai vận hành có ràng buộc cứng.** Người duy nhất giữ quyền vận hành hệ
thống thanh toán nghỉ đột ngột, hợp đồng SLA với đối tác có phạt. Ở đây thời gian chi phối tất cả; phân
tích bốn phương án là xa xỉ. Nhưng vẫn phải làm một việc: ghi lại rằng đây là quyết định dưới ràng buộc
khẩn cấp, và đặt lịch xem lại cấu trúc vai này trong 2 quý — nếu không, giải pháp khẩn cấp trở thành
cấu trúc vĩnh viễn.

**Khi bạn không có quyền quyết định phạm vi hoặc công cụ.** Ở nhiều ODC, phạm vi do khách quyết, công cụ
do khách quyết, và bạn chỉ có duy nhất một đòn bẩy là số người. Trong điều kiện đó, việc "xét bốn phương
án" là diễn. Điều nên làm thay: dùng chính bảng bốn phương án làm tài liệu trao đổi **với khách hàng**,
chuyển nó từ phân tích nội bộ thành đề xuất thương mại ("nếu quý vị cho phép cắt module X, chúng tôi
không cần bổ sung 2 kỹ sư và tiết kiệm được Y").

**Khi thị trường đang có cơ hội hiếm.** Một Staff Engineer bạn theo đuổi hai năm bỗng sẵn sàng nói
chuyện. Không có scorecard nào cho vai này vì vai này chưa tồn tại. Đây là trường hợp hợp lệ để tuyển
trước, thiết kế vai sau — nhưng chỉ hợp lệ nếu bạn thành thật rằng đó là điều bạn đang làm, và nếu bạn
có đủ bài toán khó để người đó không chán trong 6 tháng. Tuyển một người giỏi vào chỗ không có bài toán
xứng đáng là cách tốn kém nhất để mất họ.

---

## 2. Interview Design

### Problem Statement

Một fintech Việt, 60 engineer. Trong 12 tháng, họ phỏng vấn 140 ứng viên Backend, offer 11, nhận 8.
Trong 8 người đó, sau 12 tháng: 3 người được đánh giá vượt kỳ vọng, 3 người đạt, 2 người đã rời hoặc
đang trong kế hoạch cải thiện. Tỉ lệ 2/8 không phải là con số tệ so với mặt bằng. Vấn đề nằm ở chỗ khác.

Khi Hà rà lại 8 hồ sơ phỏng vấn để tìm xem tín hiệu nào đã dự báo đúng, cô phát hiện: **không tìm được
gì để rà**. Ghi chép của panel gồm những dòng như "solid", "communication tốt", "cần cải thiện SQL",
"cảm giác không phù hợp culture". Không có rubric, không có thang điểm chung, mỗi người hỏi một bộ câu
khác nhau, và hai người phỏng vấn cùng một ứng viên ở hai vòng khác nhau đôi khi đưa ra kết luận trái
ngược mà không ai buồn giải thích. Hệ thống tuyển dụng của họ không phải là một hệ thống tệ — nó không
phải là một hệ thống. Nó là 140 cuộc trò chuyện độc lập.

Hậu quả quan sát được khi thiếu thiết kế phỏng vấn:

- Không thể cải tiến. Không có dữ liệu chuẩn hoá thì không có cách nào biết vòng nào có giá trị dự báo,
  vòng nào chỉ tiêu thời gian.
- Sai số giữa người phỏng vấn lớn hơn sai số giữa ứng viên. Nói cách khác: kết quả của ứng viên phụ thuộc
  vào *ai* phỏng vấn nhiều hơn phụ thuộc vào *năng lực của họ*. Đây là định nghĩa của một phép đo hỏng.
- Ứng viên tốt bị mất vì quy trình lê thê, thiếu nhất quán, và họ đọc được sự thiếu chuẩn bị đó.
- Kết luận cuối cùng bị quyết định bởi người nói to nhất hoặc người có chức cao nhất trong phòng debrief.

### First Principles

**Phỏng vấn là một bài toán đo lường có tỉ lệ nhiễu trên tín hiệu rất cao.** Bạn đang cố dự báo hiệu suất
trong 2–3 năm dựa trên 3–6 giờ quan sát trong một môi trường nhân tạo, với một người đang lo lắng và
đang tự trình bày có chọn lọc. Không thiết kế nào làm cho bài toán này dễ. Thiết kế tốt chỉ làm một
việc: **giảm nhiễu**, để phần tín hiệu ít ỏi không bị chôn.

Về hiệu lực dự báo, có một nền tảng công khai rộng rãi đáng nhắc: bài phân tích tổng hợp của Schmidt và
Hunter (1998, *Psychological Bulletin*) về hiệu lực dự báo của các phương pháp tuyển chọn, cùng các bản
cập nhật và phê phán sau này (đáng chú ý là bản xem xét lại của Sackett và cộng sự, 2022, hạ thấp một
số hệ số ước lượng ban đầu). Đây là các nguồn công khai, dễ tra cứu. Điểm chung giữa các bản gốc và bản
hiệu chỉnh — thứ đáng dùng để ra quyết định:

- **Work sample test** (mẫu công việc thật) và **structured interview** (phỏng vấn có cấu trúc) nằm ở
  nhóm có hiệu lực dự báo cao nhất trong các phương pháp thực tế.
- **Unstructured interview** (phỏng vấn tự do) có hiệu lực dự báo **thấp hơn rõ rệt** so với structured
  interview, dù cả hai đều là "nói chuyện với ứng viên trong một giờ".
- Kết hợp nhiều phương pháp bổ trợ nhau tốt hơn dựa vào một phương pháp duy nhất.

Con số tuyệt đối của các hệ số đang được tranh luận trong giới học thuật; thứ tự tương đối thì ổn định
qua nhiều thập kỷ. Với người làm quản trị, thứ tự tương đối là đủ để ra quyết định thiết kế.

**Vì sao unstructured interview đo sai.** Ba cơ chế, cả ba đều là đặc tính của nhận thức con người chứ
không phải lỗi của người phỏng vấn kém:

1. *Không có hệ quy chiếu chung.* Nếu tôi hỏi ứng viên A về Kafka và ứng viên B về Redis, tôi không so
   sánh hai người — tôi so sánh hai ấn tượng. Trong đầu tôi chúng nằm trên cùng một thang, nhưng thang
   đó không tồn tại.
2. *Similarity bias.* Khi không có tiêu chí, não dùng tiêu chí sẵn có nhất: "người này suy nghĩ có giống
   cách tôi suy nghĩ không". Người giống ta cho ta cảm giác thông minh, vì họ đi tới cùng kết luận qua
   cùng con đường. Đây là lý do unstructured interview nổi tiếng là đo mức độ giống người phỏng vấn hơn
   là đo năng lực — và cũng là cơ chế làm team ngày càng đồng chất qua các vòng tuyển.
3. *Sensemaking sau khi đã kết luận.* Con người ra phán đoán nhanh rồi xây lập luận sau. Trong một cuộc
   trò chuyện tự do, người phỏng vấn có toàn quyền chọn câu hỏi tiếp theo — nên họ vô thức chọn những
   câu củng cố phán đoán đã có. Đây là confirmation bias có công cụ thực thi.

Rubric và câu hỏi chuẩn hoá không làm người phỏng vấn thông minh hơn. Chúng **tước bớt quyền tự do** của
người phỏng vấn ở đúng những chỗ mà quyền tự do gây hại.

**Vì sao work sample mạnh.** Vì nó thu hẹp khoảng cách suy luận. Khi bạn hỏi "bạn sẽ làm gì nếu...", bạn
đo khả năng *nói về* công việc. Khi bạn đưa một đoạn code có bug thật từ hệ thống của bạn và ngồi xem
họ tìm, bạn đo chính hành vi bạn cần. Mọi bước suy luận từ bằng chứng tới kết luận đều là chỗ để nhiễu
lọt vào; work sample có ít bước nhất.

### Mental Model

**Signal Detection: mỗi vòng là một cảm biến có độ nhạy và độ đặc hiệu riêng.**

```
                 Ứng viên thật sự phù hợp?
                     Có           Không
Vòng nói "đạt"    True Positive   False Positive  ← chi phí: tuyển sai (12–19 tháng lương)
Vòng nói "trượt"  False Negative  True Negative   ← chi phí: mất người tốt (khó thấy, phân tán)
```

Không có cảm biến nào hoàn hảo. Thiết kế vòng phỏng vấn là bài toán **xếp chuỗi cảm biến** sao cho:

- Vòng đầu rẻ, độ nhạy cao (ít bỏ sót người tốt), chấp nhận nhiều false positive → sàng lọc thô.
- Vòng sau đắt, độ đặc hiệu cao (ít nhận nhầm người không phù hợp) → xác nhận.
- Mỗi năng lực trong scorecard được đo bởi **ít nhất hai cảm biến độc lập**. Hai phép đo độc lập cùng
  chỉ một hướng cho độ tin cậy cao hơn nhiều so với một phép đo, kể cả khi từng phép đo đều nhiễu. Đây
  là nguyên lý dư thừa (redundancy) trong thiết kế hệ thống, áp vào con người.

Chữ "độc lập" là chữ then chốt. Nếu hai người cùng xem một bài take-home và cùng bị ảnh hưởng bởi ấn
tượng từ CV, đó không phải hai phép đo độc lập — đó là một phép đo được lặp lại hai lần với cùng sai số
hệ thống.

**Funnel với tỉ lệ chuyển đổi.** Mỗi vòng loại người, và mỗi vòng cũng mất người vì họ bỏ cuộc. Bảng
minh hoạ (số minh hoạ, Senior BE, thị trường Việt Nam, không dùng agency):

| Giai đoạn | Số ứng viên | Tỉ lệ qua | Ghi chú |
|---|---|---|---|
| CV nhận được | 120 | — | |
| CV qua sàng lọc | 30 | 25% | Sàng theo scorecard, không theo trường/công ty |
| Phone screen (30 phút) | 22 | 73% | 8 người không hồi đáp hoặc đã có offer khác |
| Take-home / Live coding | 14 | 64% | 5 từ chối làm take-home ← chi phí thật của lựa chọn này |
| System Design | 7 | 50% | |
| Behavioral + Bar raiser | 5 | 71% | |
| Offer | 2 | 40% | |
| Nhận offer | 1–2 | 50–100% | Mất vì lương, vì chậm, vì counter-offer |

Đọc bảng này theo chiều dọc cho một kết luận thực dụng: **tốc độ là một tiêu chí thiết kế, không phải
một thứ xa xỉ**. Ở thị trường Việt Nam với ứng viên Senior tốt, thời gian từ CV tới offer trên 3 tuần
là nơi bạn mất nhiều ứng viên nhất, và bạn mất chính những người có nhiều lựa chọn nhất — tức là những
người bạn muốn nhất. Mỗi vòng bạn thêm vào phải trả giá bằng xác suất mất người.

### Practical Framework

**Bước 1 — Dựng ma trận năng lực × vòng, đi ngược từ scorecard.**

Lấy mục C của Role Scorecard (chủ đề 1). Mỗi năng lực phải có ít nhất hai dấu X trên hàng của nó. Mỗi
vòng nên đo tối đa 3 năng lực — nhiều hơn thì trong 60 phút không vòng nào đo được gì tử tế.

| Năng lực | Phone screen | Take-home | Deep dive (review bài) | System Design | Behavioral | Bar raiser |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| C1. Thiết kế hệ thống phân tán | | | X | **X** | | X |
| C2. Debug hiệu năng có phương pháp | | **X** | X | | | |
| C3. Viết & bảo vệ quyết định kỹ thuật | | X | **X** | X | | |
| C4. Nâng năng lực người khác | | | | | **X** | X |
| Deal-breaker: quy lỗi cho người khác | | | | | X | X |

Chữ đậm = vòng chịu trách nhiệm chính cho năng lực đó. Nếu một hàng chỉ có một dấu X, bạn đang ra quyết
định về năng lực đó dựa trên một phép đo — hoặc thêm vòng, hoặc hạ năng lực đó xuống mục "nice to have".
Nếu một cột không có chữ đậm nào, vòng đó không có lý do tồn tại; xoá nó và tiết kiệm 60 phút của 2 người
cho mỗi ứng viên.

**Bước 2 — Chọn loại vòng theo cái nó đo được và cái nó không đo được.**

| Loại vòng | Đo được | KHÔNG đo được | Chi phí | Rủi ro chính |
|---|---|---|---|---|
| Phone screen 30' | Động lực, kỳ vọng lương, deal-breaker rõ ràng, khả năng diễn đạt | Gần như mọi thứ về kỹ thuật | Thấp | Loại nhầm người nói kém nhưng làm giỏi |
| Live coding 60' | Viết code chạy được, tư duy dưới áp lực, phản ứng với gợi ý | Chất lượng thiết kế dài hạn, sự kỹ lưỡng, khả năng làm việc dài hơi | Trung bình | Đo lo âu thay vì đo năng lực |
| Take-home (≤4h) | Chất lượng code thật, cấu trúc, test, tài liệu, gu kỹ thuật | Tốc độ thật, khả năng cộng tác, tính trung thực về tác giả | Cao cho ứng viên, thấp cho panel | Loại ứng viên bận; nhờ người khác làm |
| Deep dive vào bài take-home | Độ sâu hiểu biết về chính bài mình nộp, khả năng bảo vệ quyết định, trung thực | Năng lực ngoài phạm vi bài | Thấp (đã có bài) | Bỏ qua nếu không làm → mất phần lớn giá trị của take-home |
| System Design 60' | Tư duy trade-off, xử lý ràng buộc mơ hồ, độ rộng kiến thức | Khả năng thực thi, chất lượng code | Trung bình | Biến thành bài kiểm tra thuộc lòng kiến trúc mẫu |
| Behavioral (STAR) | Mẫu hành vi quá khứ: xung đột, thất bại, ảnh hưởng lên người khác | Năng lực kỹ thuật | Trung bình | Ứng viên luyện sẵn kịch bản |
| Bar raiser (người ngoài team) | Chuẩn nhất quán toàn tổ chức, phản biện thiên vị của hiring manager | Chi tiết chuyên môn hẹp của team | Trung bình | Trở thành quyền phủ quyết tuỳ tiện nếu không có rubric |

Vòng **Deep dive vào bài take-home** là vòng có tỉ lệ giá trị trên chi phí cao nhất và bị bỏ qua nhiều
nhất. Nó giải quyết gần như toàn bộ nhược điểm của take-home: bạn ngồi cùng ứng viên, hỏi "vì sao chọn
cách này", "nếu tải tăng 10 lần thì phần nào hỏng trước", "chỗ nào bạn không hài lòng". Người tự làm bài
trả lời trôi chảy; người không tự làm lộ ra trong 5 phút. Không có vòng này, take-home vừa tốn thời gian
của ứng viên vừa không đáng tin.

**Bước 3 — Viết rubric 1–4 với mô tả hành vi quan sát được.**

Nguyên tắc viết rubric: mỗi mức phải mô tả **cái người phỏng vấn nhìn thấy hoặc nghe thấy**, không mô tả
phẩm chất bên trong. "Tư duy tốt" không phải mô tả hành vi. "Nêu ra ít nhất hai phương án và so sánh
chúng bằng một tiêu chí rõ ràng trước khi chọn" là mô tả hành vi.

Dùng thang 4 mức (không dùng 5) — vì thang lẻ tạo ra mức giữa an toàn, và người phỏng vấn không chắc
chắn sẽ dồn về đó, làm mất thông tin. Thang chẵn buộc phải nghiêng về một phía.

```
RUBRIC — VÒNG SYSTEM DESIGN (60 phút)
Bài toán: "Thiết kế hệ thống xử lý đơn hàng cho campaign flash sale.
           Peak 5.000 đơn/giây trong 10 phút, ngày thường 50 đơn/giây.
           Không được bán quá tồn kho. Thanh toán qua cổng bên thứ ba có thể timeout."

Người phỏng vấn cung cấp thêm khi được hỏi (không tự đưa ra):
  - Tồn kho ~200.000 SKU, chỉ ~500 SKU tham gia flash sale.
  - Chấp nhận đơn hàng được xác nhận trễ tới 30 giây.
  - Đã có PostgreSQL và Kafka trong tổ chức.

--- TIÊU CHÍ 1: LÀM RÕ YÊU CẦU TRƯỚC KHI THIẾT KẾ (trọng số 20%) ---
1 — Bắt tay vẽ kiến trúc trong 2 phút đầu, không hỏi câu nào về ràng buộc.
2 — Hỏi 1–2 câu về quy mô, rồi thiết kế; không hỏi về ràng buộc tính đúng đắn
    (bán quá tồn kho, timeout thanh toán).
3 — Hỏi rõ về quy mô, mẫu tải, ràng buộc tính đúng đắn; phát biểu lại yêu cầu bằng
    lời mình trước khi vẽ; nêu rõ điều gì mình đang giả định.
4 — Như mức 3, và chủ động phân biệt yêu cầu cứng với yêu cầu có thể thương lượng,
    hỏi "nếu chấp nhận X thì bài toán dễ đi nhiều — có chấp nhận được không?".

--- TIÊU CHÍ 2: XỬ LÝ RÀNG BUỘC TÍNH ĐÚNG ĐẮN (trọng số 30%) ---
1 — Không nhận ra vấn đề bán quá tồn kho, hoặc đề xuất "dùng transaction" mà không
    xét điểm nghẽn ghi.
2 — Nhận ra vấn đề, đề xuất khoá pessimistic trên hàng tồn kho; không nhận ra rằng
    500 SKU nóng sẽ tạo hot row contention ở 5.000 TPS.
3 — Nhận ra hot row; đề xuất một cơ chế hợp lý (reserve trước qua counter trong Redis/
    Kafka partition theo SKU / phân mảnh tồn kho thành N ô) và nêu được nhược điểm
    của cơ chế đó.
4 — Như mức 3, và xử lý được đường biên: điều gì xảy ra khi Redis mất dữ liệu, làm sao
    đối soát lại tồn kho, chấp nhận oversell bao nhiêu và cơ chế bù (huỷ + hoàn tiền)
    có rẻ hơn việc chặn tuyệt đối không.

--- TIÊU CHÍ 3: XỬ LÝ LỖI VÀ IDEMPOTENCY (trọng số 25%) ---
1 — Không đề cập tới việc thanh toán có thể timeout hoặc trả về kết quả hai lần.
2 — Đề cập retry nhưng không xử lý trùng lặp; hoặc nói "dùng idempotency key" mà không
    nói khoá đó sinh ra từ đâu và lưu ở đâu.
3 — Thiết kế được luồng idempotent end-to-end: nguồn sinh khoá, nơi lưu, thời gian sống,
    hành vi khi trạng thái không xác định (timeout không biết thành công hay thất bại).
4 — Như mức 3, và nêu cơ chế đối soát định kỳ với cổng thanh toán cho các giao dịch treo,
    kèm suy nghĩ về việc ai xử lý phần lệch (con người hay tự động).

--- TIÊU CHÍ 4: TRAO ĐỔI VÀ TIẾP THU (trọng số 25%) ---
1 — Không giải thích lý do lựa chọn; phòng thủ khi bị hỏi ngược; hoặc im lặng dài không
    nói mình đang nghĩ gì.
2 — Giải thích được khi được hỏi, nhưng không chủ động nêu trade-off; khi người phỏng vấn
    chỉ ra vấn đề thì hoặc bỏ ngay phương án cũ hoặc bảo vệ tới cùng, không cân nhắc.
3 — Chủ động nêu trade-off của lựa chọn của mình; khi bị phản biện thì hỏi lại để hiểu,
    rồi hoặc điều chỉnh có lý do hoặc giữ nguyên có lý do.
4 — Như mức 3, và điều chỉnh mức chi tiết theo người nghe; kiểm tra xem người nghe có
    theo kịp không; tự đánh dấu chỗ mình không chắc thay vì nói lấp liếm.

--- QUY TẮC CHẤM ---
- Ghi ít nhất một bằng chứng nguyên văn cho mỗi tiêu chí. Không có bằng chứng = không chấm.
- Mức 3 = đạt chuẩn cho bậc L4 (Senior). Mức 2 ở tiêu chí trọng số ≥25% = không đạt bậc L4.
- Điểm tổng KHÔNG phải trung bình cộng cơ học; ghi rõ nếu một tiêu chí là yếu tố quyết định.
- Nộp đánh giá trong vòng 4 giờ sau buổi, TRƯỚC khi đọc đánh giá của người khác.
```

Hai dòng cuối của quy tắc chấm quan trọng ngang phần nội dung. "Nộp trước khi đọc của người khác" là
biện pháp chống information cascade, sẽ phân tích kỹ ở chủ đề 3.

**Bước 4 — Chuẩn hoá câu hỏi và cách chấm điểm cho vòng Behavioral.**

Behavioral là vòng dễ trở thành trò chuyện phiếm nhất. Chuẩn hoá bằng cách: cố định 3–4 câu gốc cho mọi
ứng viên cùng vai, và chuẩn bị sẵn câu đào sâu.

```
VÒNG BEHAVIORAL — Senior BE (45 phút, 3 câu gốc + đào sâu)
Đo: C3 (bảo vệ quyết định kỹ thuật), C4 (nâng năng lực người khác), deal-breaker (quy lỗi)

CÂU 1 (C3) — "Kể về một quyết định kỹ thuật bạn đưa ra mà sau đó hoá ra là sai.
             Bạn phát hiện ra bằng cách nào và bạn đã làm gì?"
  Đào sâu bắt buộc: Ai chịu ảnh hưởng? Bạn nói với họ thế nào? Nếu làm lại, bạn thay
                    đổi ở bước nào của quy trình ra quyết định, không phải ở kết luận?
  Cờ đỏ: không nhớ được lần nào sai; mọi cái sai đều do yêu cầu đổi hoặc do người khác.

CÂU 2 (C4) — "Kể về một người bạn từng giúp giỏi lên rõ rệt. Cụ thể bạn đã làm gì?"
  Đào sâu bắt buộc: Bạn biết họ giỏi lên nhờ tín hiệu nào? Có lần nào bạn giúp mà
                    không hiệu quả? Khác biệt giữa hai lần là gì?
  Cờ đỏ: chỉ mô tả việc mình trả lời câu hỏi khi được hỏi; không có ví dụ cụ thể.

CÂU 3 (C3+C4) — "Kể về một lần bạn không đồng ý với quyết định kỹ thuật của người có
                thẩm quyền cao hơn. Chuyện gì đã xảy ra?"
  Đào sâu bắt buộc: Bạn nêu ý kiến ở đâu, trước mặt ai? Sau khi quyết định cuối được
                    chốt ngược ý bạn, bạn làm gì? Kết quả thực tế ra sao?
  Cờ đỏ: chưa từng bất đồng; hoặc bất đồng rồi âm thầm làm theo ý mình.

QUY TẮC: bám sát STAR (Situation - Task - Action - Result). Nếu ứng viên nói bằng
"chúng tôi", hỏi lại "phần nào cụ thể là bạn làm?" — đây là câu đào sâu có giá trị
phân biệt cao nhất trong toàn bộ vòng.
```

**Bước 5 — Chốt vòng lặp: sau mỗi 10 lần tuyển, đối chiếu điểm phỏng vấn với hiệu suất thực tế sau 12
tháng.** Vòng nào có tương quan gần bằng 0 là vòng nên thiết kế lại hoặc bỏ. Rất ít tổ chức làm bước
này, và đó là lý do quy trình phỏng vấn của họ không cải thiện qua nhiều năm dù đã chạy hàng trăm lần.

### Trade-off

**Take-home vs live coding.** Take-home đo gần thực tế hơn: ứng viên có môi trường quen thuộc, có Google,
có thời gian suy nghĩ — đúng như điều kiện làm việc thật. Nhưng nó đánh thuế nặng và không đồng đều lên
ứng viên: người có con nhỏ, người đang làm full-time ở nơi OT nhiều, người đang phỏng vấn 4 công ty cùng
lúc sẽ bỏ. Ở bảng funnel trên, 5/19 người từ chối take-home — và không có lý do gì tin rằng 5 người đó
yếu hơn 14 người còn lại; nhiều khả năng họ có nhiều lựa chọn hơn.

Live coding nhanh, công bằng về thời gian, và cho thấy cách người ta phản ứng khi bí. Nhưng áp lực trong
live coding là áp lực nhân tạo — không giống bất kỳ tình huống công việc nào — và nó phạt nặng người lo
âu, người ít kinh nghiệm phỏng vấn, và người không quen nói trong lúc nghĩ.

Điều kiện nghiêng:

- Nghiêng về take-home khi: vai cần chất lượng code và tính kỹ lưỡng (platform, thư viện nội bộ), thị
  trường ứng viên không quá cạnh tranh, và bạn cam kết giới hạn ≤4 giờ **và** trả công cho bài (một số
  công ty Việt đã làm, và đây là tín hiệu tuyển dụng rất mạnh).
- Nghiêng về live coding khi: cần tốc độ, ứng viên Senior có nhiều offer, hoặc vai cần cộng tác thời gian
  thực nhiều.
- Phương án lai đáng cân nhắc: cho ứng viên **chọn** giữa take-home và một buổi pair programming 90 phút.
  Chi phí: hai rubric phải hiệu chỉnh về cùng thang. Lợi ích: không loại người vì hoàn cảnh sống.

**Nhiều vòng vs ít vòng.** Thêm một vòng làm tăng độ tin cậy của phép đo — nhưng theo quy luật lợi ích
giảm dần, trong khi chi phí (thời gian panel, thời gian ứng viên, xác suất ứng viên bỏ giữa chừng) tăng
tuyến tính. Bảng minh hoạ:

| Số vòng | Độ tin cậy tương đối | Thời gian từ CV tới offer | Tỉ lệ ứng viên bỏ giữa chừng (số minh hoạ) |
|---|---|---|---|
| 2 | Thấp | 5–7 ngày | ~10% |
| 4 | Khá | 12–18 ngày | ~25% |
| 6 | Nhỉnh hơn 4 chút | 25–40 ngày | ~45% |
| 8 | Không cao hơn 6 một cách đáng kể | 40+ ngày | ~60% |

Điểm gãy thường nằm ở 4–5 vòng. Nghiêng về nhiều vòng khi chi phí tuyển sai cực cao (vai có quyền truy
cập dữ liệu nhạy cảm, vai lãnh đạo) hoặc khi bạn có lợi thế thương hiệu để ứng viên chịu chờ. Nghiêng về
ít vòng khi bạn là công ty ít tên tuổi cạnh tranh với công ty lớn — trong trường hợp đó **tốc độ là lợi
thế cạnh tranh khả dụng duy nhất của bạn**, và bạn nên dùng nó một cách có chủ đích: gộp vòng, quyết
định trong 48 giờ, offer trong 72 giờ.

**Chuẩn hoá vs linh hoạt cho ứng viên đặc biệt.** Rubric cứng đảm bảo công bằng và so sánh được, nhưng
đôi khi bạn gặp một người mà bộ vòng hiện tại không đo đúng — ví dụ một người 15 năm kinh nghiệm chưa
từng làm live coding trong đời. Cách xử lý ít tệ nhất: **giữ nguyên năng lực cần đo, thay đổi cách đo,
ghi rõ vào hồ sơ rằng đã thay đổi và vì sao**. Điều không được làm là bỏ bớt năng lực cần đo — đó không
phải linh hoạt, đó là hạ chuẩn có chọn lọc, và nó là cửa ngõ chuẩn cho bias.

### Real-world Scenarios

**Tình huống A — Câu đố thuật toán trong công ty làm CRUD (ODC, khách hàng Singapore).**

Panel của Tuấn dùng một bài invert binary tree và một bài dynamic programming làm vòng coding, kế thừa
từ ba năm trước. Công việc thật của team: tích hợp API, viết business logic, tối ưu query. Trong hai
năm, không một dòng code nào của team liên quan tới cây nhị phân.

Quân, Senior BE trong panel, bảo vệ: "Bài này lọc được người có nền tảng. Người giải được thì học gì
cũng nhanh."

Lập luận này không hoàn toàn vô lý — có tương quan giữa năng lực giải thuật và năng lực chung. Nhưng nó
bỏ qua hai chi phí. Một: bài như vậy đo mạnh **mức độ gần đây ứng viên có luyện LeetCode**, tức là ưu
tiên sinh viên mới ra trường và người đang chủ động chuyển việc, và phạt người đã làm 8 năm trong
production. Hai: nó gửi tín hiệu sai về công việc, dẫn tới người nhận offer rồi vỡ mộng trong 3 tháng.

Cách sửa mà Tuấn áp dụng: thay bằng một bài lấy từ chính codebase. Một service thật, đã được rút gọn,
có một bug về xử lý timezone làm sai báo cáo doanh thu cuối tháng, kèm log và một test đang fail. Ứng
viên có 50 phút để tìm và sửa. Rubric đo: cách thu hẹp phạm vi tìm kiếm, có đọc test trước không, có
hỏi về dữ liệu đầu vào không, chất lượng bản vá và có thêm test hồi quy không.

Kết quả sau 6 tháng (số minh hoạ): tỉ lệ ứng viên phàn nàn về vòng coding giảm rõ; và quan trọng hơn,
điểm vòng này bắt đầu **tương quan** với đánh giá 6 tháng của người được tuyển, điều bài binary tree
chưa bao giờ làm được.

**Tình huống B — Quyết định trong 5 phút đầu.**

Khoa phỏng vấn một ứng viên tốt nghiệp trường không nổi tiếng, nói nhỏ, 5 phút đầu trả lời lúng túng.
Trong đầu Khoa, kết luận đã hình thành. 55 phút còn lại, Khoa hỏi những câu ngày càng khó, nghe câu trả
lời qua bộ lọc "tôi đã biết người này yếu", và ghi vào form: "Nền tảng không vững, không recommend."

Bốn tháng sau, người đó được một công ty khác tuyển và Khoa gặp lại trong một meetup, nơi anh ta trình
bày về hệ thống mình vừa xây.

Cơ chế đã xảy ra không phải là Khoa thiếu năng lực. Là confirmation bias vận hành đúng như thiết kế của
não người: sau khi có giả thuyết, chi phí nhận thức để tìm bằng chứng phản bác cao hơn chi phí tìm bằng
chứng ủng hộ, nên não chọn cái rẻ. Trong unstructured interview, người phỏng vấn còn có thêm công cụ để
thực thi thiên vị đó — quyền chọn câu hỏi tiếp theo.

Ba biện pháp có tác dụng, xếp theo hiệu quả:

1. **Câu hỏi cố định cho mọi ứng viên.** Tước quyền chọn câu hỏi theo cảm nhận. Đây là biện pháp mạnh
   nhất và cũng là biện pháp bị chống đối nhiều nhất, vì nó làm người phỏng vấn thấy mình bị hạn chế —
   đó chính xác là mục đích.
2. **Ghi bằng chứng nguyên văn trong lúc phỏng vấn, chấm điểm sau.** Tách việc quan sát khỏi việc phán
   xét. Khi bạn phải ghi lại đúng câu ứng viên nói, bạn buộc phải nghe.
3. **Chấm từng tiêu chí trước, kết luận tổng sau.** Nếu bạn viết kết luận trước rồi điền điểm, điểm sẽ
   được điền để khớp kết luận (halo effect nội bộ). Thứ tự trên form quan trọng hơn người ta tưởng.

**Tình huống C — Panel năm người, năm phong cách.**

Trong một buổi debrief, Hà nhận ra: Quân luôn hỏi về Kafka vì đó là thứ Quân giỏi; Khoa luôn hỏi về
clean code và cho điểm thấp ai không dùng dependency injection; Linh hỏi toàn câu về quy trình Scrum;
Tuấn hỏi những câu mở kiểu "bạn đam mê gì trong công nghệ". Năm người, năm bộ tiêu chí, không ai biết
mình đang đo cái gì trong scorecard.

Đây là trạng thái mặc định của mọi panel không được thiết kế. Nó không sinh ra từ sự cẩu thả mà từ một
điều tự nhiên: **khi không được giao tiêu chí, con người dùng tiêu chí của chính mình**, và tiêu chí của
chính mình luôn là thứ mình giỏi.

Cách sửa của Hà, ba việc, làm trong một buổi 90 phút:

1. Gán mỗi người **một vòng cố định với một bộ tiêu chí cố định**. Quân không còn "phỏng vấn kỹ thuật"
   chung chung; Quân sở hữu vòng System Design với rubric ở trên và chỉ chấm 4 tiêu chí đó.
2. **Calibration bằng ứng viên giả lập**: cả panel cùng xem một bản ghi (hoặc một transcript đã ẩn danh)
   của một buổi System Design, mỗi người chấm độc lập, rồi so điểm. Lần đầu, độ lệch thường là 1.5–2 mức
   trên thang 4 — đây là con số làm mọi người nghiêm túc hơn bất kỳ bài giảng nào về bias.
3. **Người mới vào panel phải shadow 2 buổi và được shadow 2 buổi** trước khi chấm độc lập.

### Best Practices

- **Thiết kế vòng đi ngược từ scorecard, không từ "chúng ta thường phỏng vấn thế nào".** Lý do: nếu
  không neo vào scorecard, mỗi vòng sẽ trôi dần về thứ người phỏng vấn thấy thú vị.
- **Mỗi năng lực ≥ 2 vòng độc lập; mỗi vòng ≤ 3 năng lực.** Lý do: dư thừa chống nhiễu; giới hạn trên
  chống việc đo hời hợt nhiều thứ.
- **Ghi bằng chứng nguyên văn, không ghi kết luận.** Lý do: bằng chứng có thể được người khác đánh giá
  lại; kết luận thì không. Một hồ sơ chỉ có kết luận là hồ sơ không kiểm toán được.
- **Nộp đánh giá trước khi đọc của người khác, hạn 4–24 giờ.** Lý do: chống cascade (chủ đề 3) và chống
  quên chi tiết.
- **Ưu tiên bài lấy từ codebase thật đã rút gọn hơn bài chuẩn từ Internet.** Lý do: gần với công việc
  hơn, khó luyện thuộc hơn, và đồng thời là công cụ tuyển dụng — ứng viên tốt thích thấy công việc thật.
- **Đo và công bố thời gian trung bình từ CV tới offer.** Lý do: nó là chỉ số duy nhất buộc tổ chức đối
  mặt với chi phí của việc thêm vòng.
- **Nói trước cho ứng viên biết cấu trúc từng vòng và tiêu chí đánh giá.** Lý do: giảm nhiễu do lo âu và
  do không hiểu luật chơi; bạn muốn đo năng lực, không muốn đo khả năng đoán ý người phỏng vấn.

### Anti-patterns

**"Câu đố thông minh".** Hỏi những câu kiểu ước lượng số cửa sổ ở Hà Nội, hoặc câu đố logic. Cơ chế phá
hoại: không có bằng chứng công khai nào cho thấy loại câu này dự báo hiệu suất; chúng chủ yếu đo mức độ
quen thuộc với thể loại câu đố và tạo cảm giác quyền lực cho người hỏi. Dấu hiệu sớm: người phỏng vấn
kể lại câu hỏi của mình với vẻ thích thú trước khi kể câu trả lời của ứng viên.

**"Vòng phỏng vấn không có chủ."** Ba người cùng dự một vòng, không ai chịu trách nhiệm chấm. Cơ chế:
trách nhiệm phân tán → không ai ghi chép kỹ → đánh giá dựa trên trí nhớ mờ. Dấu hiệu sớm: form đánh giá
của vòng đó thường xuyên trống hoặc nộp muộn.

**"Đánh giá bằng tính từ."** "Sharp", "solid", "hơi weak", "vibe không hợp". Cơ chế: tính từ không kiểm
chứng được, không tranh luận được, và là vỏ bọc hoàn hảo cho bias. Dấu hiệu sớm: đọc 10 form gần nhất,
đếm số form không chứa một câu nguyên văn nào của ứng viên.

**"Nâng chuẩn theo tâm trạng của panel."** Sau khi tuyển hụt một người giỏi, panel vô thức chấm gắt hơn
với mọi người sau đó; sau khi ghế trống 5 tháng, chấm lỏng hơn. Cơ chế: thang đo trôi theo trạng thái
cảm xúc của tổ chức. Dấu hiệu sớm: điểm trung bình theo tháng dao động mạnh mà không giải thích được
bằng chất lượng nguồn ứng viên.

**"Phỏng vấn theo CV."** Toàn bộ buổi phỏng vấn là đi qua từng dòng CV. Cơ chế: CV do ứng viên soạn, nên
bạn để ứng viên chọn nội dung phép đo; và nó neo bạn vào công ty cũ, trường cũ (anchoring — chủ đề 3).
Dấu hiệu sớm: hai ứng viên cùng vai được hỏi hai bộ câu hoàn toàn khác nhau.

**"Bar raiser thành quyền phủ quyết cá nhân."** Vai bar raiser được lập ra để giữ chuẩn nhất quán, nhưng
nếu không có rubric thì nó trở thành một người có quyền nói "không" mà không cần bằng chứng. Cơ chế: tập
trung quyền lực không kèm trách nhiệm giải trình. Dấu hiệu sớm: bar raiser từ chối ứng viên bằng lý do
không map được vào bất kỳ tiêu chí nào trong scorecard.

### Khi nào KHÔNG nên áp dụng

**Khi bạn tuyển người thứ 2, thứ 3 của công ty.** Ở giai đoạn này, "phù hợp" nghĩa là chịu được sự hỗn
loạn, tự tạo cấu trúc, và làm mọi việc. Bộ rubric 4 tiêu chí cho System Design không đo được điều đó, và
việc dựng quy trình 5 vòng cho một công ty 4 người là nghi lễ. Cái nên làm thay: làm việc thật cùng nhau
2–4 tuần dưới dạng hợp đồng ngắn hạn nếu hai bên chấp nhận được. Đây là work sample ở dạng mạnh nhất, và
nó chỉ khả thi ở quy mô rất nhỏ.

**Khi tuyển vai mà bạn chưa hiểu.** Bạn cần tuyển ML Engineer đầu tiên nhưng trong công ty không ai làm
ML. Viết rubric trong tình huống này là viết ra ảo tưởng có cấu trúc — tệ hơn không có rubric, vì nó tạo
vẻ ngoài của sự chặt chẽ. Cái nên làm thay: mượn một người ngoài đáng tin (cố vấn, bạn nghề ở công ty
khác) làm vòng chuyên môn, và bạn chỉ chấm những gì bạn thật sự đánh giá được.

**Khi ứng viên đến từ giới thiệu nội bộ mạnh và bạn đã có bằng chứng dài hạn.** Một người đã làm cùng
Tuấn ba năm ở công ty cũ, Tuấn có dữ liệu về họ nhiều hơn bất kỳ vòng phỏng vấn nào cung cấp được. Chạy
đủ 5 vòng ở đây không thêm thông tin, chỉ thêm rủi ro mất người và gửi tín hiệu thiếu tin tưởng. Cái nên
làm thay: rút gọn còn 2 vòng, nhưng **bắt buộc giữ một vòng do người không quen ứng viên phụ trách** —
để có ít nhất một phép đo không bị nhiễm bởi mối quan hệ, và để tránh việc mạng lưới quan hệ của một
người trở thành cửa sau vào tổ chức.

**Khi thị trường buộc bạn phải quyết trong 48 giờ.** Ứng viên Senior tốt ở Việt Nam thường có 2–3 quy
trình chạy song song. Nếu bạn biết mình đang ở thế yếu về thương hiệu và lương, quy trình 5 vòng trong
3 tuần đảm bảo bạn thua. Cái nên làm thay: gộp thành một ngày phỏng vấn duy nhất (3 vòng liên tiếp, có
nghỉ), debrief ngay chiều hôm đó, trả lời trong 24 giờ. Bạn giữ nguyên số phép đo, chỉ nén thời gian —
đây là cách đúng để đánh đổi, khác hoàn toàn với việc bỏ bớt vòng.

---

## 3. Candidate Evaluation và ra quyết định tuyển

### Problem Statement

Phòng họp, 5 giờ chiều thứ Sáu. Bốn người trong panel, một ứng viên vừa xong vòng cuối. Hà mở đầu:
"Ok, mọi người thấy sao?" Tuấn — người có chức vụ cao nhất và nói nhanh nhất — trả lời trước: "Anh thấy
ổn đấy. System design chắc tay, thái độ được." Ba giây im lặng. Linh gật: "Em cũng thấy ổn." Khoa:
"Vòng em thì bình thường thôi, nhưng nếu mọi người thấy ổn thì em không phản đối." Quân: "Em không có ý
kiến gì mạnh." Hà: "Ok vậy chốt hire."

Tổng thời gian: 4 phút. Kết quả: một quyết định trị giá 12–19 tháng lương nếu sai.

Điều đã xảy ra không phải là mọi người đồng ý. Điều đã xảy ra là **một người phát biểu và ba người còn
lại cập nhật niềm tin của mình theo phát biểu đó**. Nếu Hà hỏi từng người viết đánh giá độc lập trước khi
mở miệng, phân bố ý kiến rất có thể đã khác. Khoa nói "vòng em bình thường thôi" — đó là một tín hiệu
âm, được phát ra dưới dạng nhượng bộ, và bị chôn ngay lập tức.

Hậu quả quan sát được của việc thiếu quy trình debrief:

- Quyết định hội tụ về ý kiến của người phát biểu đầu tiên hoặc người có quyền lực nhất, chứ không về
  tổng hợp bằng chứng.
- Tín hiệu âm yếu — thứ có giá trị dự báo cao nhất — bị mất. Người ta ngại là người duy nhất nói "không"
  với một ứng viên mà cả phòng đang gật.
- Không có hồ sơ để rà lại. Sáu tháng sau, khi người này gặp vấn đề, không ai truy được là đã bỏ qua tín
  hiệu nào.
- Ở bối cảnh Việt Nam, cơ chế này mạnh hơn: chuẩn mực về việc không phản bác người lớn tuổi hoặc cấp trên
  trước mặt đông người làm cho cascade gần như không thể tránh nếu không có biện pháp cấu trúc.

### First Principles

**Bốn dạng bias có tác động lớn nhất trong đánh giá ứng viên**, và cơ chế của từng dạng:

| Bias | Cơ chế | Biểu hiện cụ thể trong debrief | Biện pháp đối kháng |
|---|---|---|---|
| **Halo effect** | Một đặc điểm nổi bật lan sang mọi đánh giá khác | "Bạn ấy giải bài coding rất nhanh" → tự động cho điểm cao cả communication lẫn ownership | Chấm từng tiêu chí riêng, không cho phép điểm tổng trước |
| **Similarity bias** | Não dùng "giống tôi" làm proxy cho "giỏi" | "Cách bạn ấy tư duy giống mình hồi mới vào" | Rubric mô tả hành vi; panel đa dạng về nền tảng |
| **Anchoring vào công ty/trường cũ** | Điểm neo đầu tiên chi phối mọi điều chỉnh sau | "Từng làm ở [công ty lớn] thì chắc ổn" / "Trường đó thì hơi lo" | Ẩn tên công ty và trường ở vòng kỹ thuật nếu khả thi; cấm dùng làm tiêu chí |
| **Ấn tượng đầu (thin-slicing)** | Phán đoán hình thành trong 1–5 phút, phần còn lại là tìm bằng chứng ủng hộ | Câu hỏi ngày càng dễ hoặc ngày càng khó tuỳ ấn tượng ban đầu | Câu hỏi cố định; ghi bằng chứng nguyên văn; chấm sau buổi |

Điểm chung của bốn dạng: chúng đều là **cách não tiết kiệm năng lượng** khi phải phán đoán dưới bất định.
Chúng không biến mất khi bạn biết về chúng — đây là phát hiện quan trọng và hay bị bỏ qua. Đọc một bài
về bias không làm bạn ít bias hơn; nó chỉ làm bạn tự tin hơn rằng mình ít bias, điều này còn tệ hơn. Chỉ
có **thay đổi cấu trúc quy trình** mới có tác dụng đo được.

**Information cascade: vì sao thảo luận trước khi viết làm hỏng quyết định.**

Cơ chế toán học đơn giản. Giả sử mỗi người trong panel có một tín hiệu riêng, độc lập, mỗi tín hiệu đúng
với xác suất 70%. Nếu bốn người viết độc lập rồi tổng hợp, xác suất đa số đúng cao hơn 70% đáng kể — đây
là lợi ích của tổng hợp nhiều quan sát độc lập, cùng nguyên lý với việc lấy trung bình nhiều phép đo
nhiễu.

Nhưng nếu người thứ nhất phát biểu trước, người thứ hai không còn dùng tín hiệu riêng của mình một cách
độc lập nữa — họ dùng tín hiệu riêng **cộng với** tín hiệu công khai của người thứ nhất. Nếu tín hiệu
công khai đủ mạnh (đặc biệt khi người thứ nhất là sếp), người thứ hai sẽ bỏ qua tín hiệu riêng. Người
thứ ba giờ thấy hai người đồng ý và càng dễ bỏ tín hiệu riêng hơn. Đến người thứ tư, tín hiệu riêng gần
như không còn trọng lượng.

Kết quả: bạn tưởng mình có bốn quan sát độc lập, thực tế bạn có **một quan sát được lặp lại bốn lần**.
Toàn bộ lợi ích của việc có panel bị triệt tiêu, trong khi chi phí (4 người × 1 giờ) vẫn phải trả. Đây
là cơ chế được mô tả rộng rãi trong tài liệu công khai về ra quyết định nhóm — cùng họ với groupthink
nhưng khác cơ chế: cascade không cần áp lực xã hội, chỉ cần thứ tự phát biểu.

Hệ quả thiết kế, rất cụ thể: **thứ tự "viết độc lập → đọc → thảo luận" không phải là hình thức, nó là
biện pháp duy nhất bảo toàn tính độc lập của các phép đo.** Đảo thứ tự này làm hỏng phần lớn giá trị của
toàn bộ quy trình phỏng vấn phía trước.

**Bất đối xứng chi phí giữa false positive và false negative.** Đã tính ở chủ đề 1: tuyển sai ≈ 12–19
tháng lương; ghế trống thêm 2 tháng ≈ 2 tháng đóng góp, phần nào được hấp thụ bởi team. Tỉ lệ khoảng 6:1
đến 9:1. Đó là cơ sở số học của nguyên tắc **"khi phân vân thì không tuyển"**.

Nhưng phải trung thực về mặt còn lại của bất đối xứng đó, thứ hầu như không ai viết ra: chi phí ghế trống
**không rơi vào người ra quyết định**. Nó rơi vào team đang gánh việc. Người quản lý áp dụng "khi phân
vân thì không tuyển" một cách máy móc trong 8 tháng liền đang chuyển chi phí sang một nhóm người không
có tiếng nói trong quyết định đó. Nguyên tắc này đúng, nhưng nó phải đi kèm một nghĩa vụ: **nếu bạn giữ
chuẩn cao, bạn phải đồng thời cắt phạm vi công việc của team, chứ không được vừa giữ chuẩn vừa giữ
nguyên khối lượng.** Đó là phần thường bị bỏ.

### Mental Model

**Bayesian updating: mỗi vòng là một cập nhật xác suất, không phải một phán quyết.**

Bắt đầu với một prior: tỉ lệ ứng viên vào tới vòng cuối mà thực sự phù hợp, ở tổ chức của bạn, dựa trên
lịch sử. Giả sử là 35% (số minh hoạ). Mỗi vòng cho một tín hiệu làm dịch xác suất đó lên hoặc xuống. Cách
tư duy này thay đổi hai điều trong thực hành:

1. Một vòng "đạt" không kết luận được gì. Nó chỉ dịch xác suất từ 35% lên có lẽ 55%. Người mắc lỗi phổ
   biến nhất là người coi vòng mình phụ trách là phán quyết cuối.
2. **Sức mạnh của một tín hiệu tỉ lệ nghịch với tần suất của nó.** "Trả lời được câu hỏi cơ bản" là tín
   hiệu yếu vì hầu hết ứng viên vào tới đó đều làm được. "Chủ động nêu ra một trade-off mà người phỏng
   vấn chưa nghĩ tới" là tín hiệu mạnh vì hiếm. Rubric tốt phải gán trọng số theo độ hiếm, không theo
   độ dễ quan sát.

**Ma trận quyết định 2×2 với ngưỡng bất đối xứng.** Vì chi phí hai loại sai lệch không bằng nhau, ngưỡng
quyết định không nằm ở 50%. Với tỉ lệ chi phí ~7:1, ngưỡng hợp lý nằm quanh 80–85% — nghĩa là bạn chỉ
nên offer khi bằng chứng đưa bạn tới mức tin tưởng cao, không phải khi bạn "nghiêng về phía có". Đây là
nội dung toán học của câu "when in doubt, don't hire", và nó cũng giải thích vì sao câu đó không mâu
thuẫn với việc không được cầu toàn: 85% không phải 100%.

### Practical Framework

**Bước 1 — Thang quyết định 4 mức, định nghĩa bằng hành vi cam kết, không bằng cảm giác.**

Vấn đề của mọi thang đánh giá là mọi người hiểu các mức khác nhau. Cách chữa: định nghĩa mỗi mức bằng
**điều bạn sẵn sàng cam kết**, không bằng mức độ ưa thích.

| Mức | Định nghĩa bằng cam kết | Ý nghĩa với quyết định |
|---|---|---|
| **Strong Hire** | "Tôi sẵn sàng nhận người này vào team của chính tôi, ngay bây giờ, và tôi tin họ sẽ nâng chuẩn của team." | Tín hiệu mạnh nhất; hiếm (nên chiếm <15% số ứng viên tới vòng cuối) |
| **Hire** | "Tôi tin người này đạt chuẩn của bậc đang tuyển. Tôi không lo lắng gì lớn." | Đủ để offer nếu không có No Hire nào không giải thích được |
| **No Hire** | "Tôi có ít nhất một lo ngại cụ thể, nêu được bằng chứng, mà tôi không biết cách giảm thiểu trong 6 tháng đầu." | Phải nêu bằng chứng; không được nói "cảm giác" |
| **Strong No Hire** | "Tôi sẽ phản đối việc tuyển người này kể cả khi tất cả những người khác đồng ý, và tôi sẵn sàng giải thích lý do bằng văn bản." | Quyền phủ quyết thực tế; phải dùng hiếm để giữ giá trị |

Ba đặc điểm quan trọng của thang này:

- **Không có mức giữa.** Không có "leaning hire". Mọi thang có mức giữa đều thấy phần lớn đánh giá dồn về
  đó, và bạn mất thông tin.
- **Strong No Hire là quyền phủ quyết, nhưng phải trả giá bằng văn bản.** Quyền lực đi kèm nghĩa vụ giải
  trình. Nếu ai đó dùng Strong No Hire ba lần trong một quý mà không lần nào viết được lý do map vào
  scorecard, đó là vấn đề của người đó, không phải của các ứng viên.
- **No Hire không tự động chặn.** Nó buộc phải thảo luận và phải được giải quyết bằng bằng chứng.

**Bước 2 — Quy trình debrief, đúng thứ tự này.**

```
QUY TRÌNH DEBRIEF — 45 phút, không quá 6 người

T-24h đến T-4h: MỖI NGƯỜI NỘP ĐỘC LẬP
   [ ] Điểm từng tiêu chí theo rubric của vòng mình phụ trách
   [ ] Ít nhất một bằng chứng nguyên văn cho mỗi tiêu chí
   [ ] Mức quyết định (Strong Hire / Hire / No Hire / Strong No Hire)
   [ ] Một câu: "Lo ngại lớn nhất của tôi về người này là ___"
       (bắt buộc điền kể cả khi chấm Strong Hire — mọi ứng viên đều có ít nhất một lo ngại;
        không điền được thường nghĩa là chưa nghĩ đủ)
   [ ] Hệ thống KHOÁ: không ai xem được đánh giá của người khác cho tới khi tất cả đã nộp.

T+0 (5 phút): ĐỌC TRONG IM LẶNG
   Facilitator (thường là hiring manager) chiếu bảng tổng hợp. Không ai nói. Mọi người đọc
   toàn bộ đánh giá của nhau. Mô hình này mượn từ thực hành công khai của Amazon với
   6-page memo: đọc trước, tranh luận sau.

T+5 (5 phút): LẬP BẢN ĐỒ ĐỒNG THUẬN VÀ LỆCH
   Facilitator nói to: "Cả bốn người đều thấy C1 đạt. Có hai điểm lệch: C4 — Khoa cho 2,
   Linh cho 4. Và mức quyết định: ba Hire, một No Hire. Ta chỉ nói về hai điểm này."

T+10 (25 phút): THẢO LUẬN CHỈ ĐIỂM LỆCH
   Quy tắc: người có điểm THẤP nhất nói TRƯỚC (đảo ngược cascade — người dễ bị áp lực
   nhất được nói khi chưa có áp lực).
   Quy tắc: chỉ tranh luận về bằng chứng. Câu hợp lệ: "Bạn ấy nói X, tôi đọc X là dấu hiệu
   của Y." Câu không hợp lệ: "Tôi cảm thấy bạn ấy ổn."
   Quy tắc: được phép đổi ý sau khi nghe bằng chứng mới, PHẢI nói rõ bằng chứng nào làm
   mình đổi ý. Đổi ý vì "mọi người đều thấy thế" không hợp lệ.

T+35 (5 phút): QUYẾT
   Hiring manager quyết. Đây KHÔNG phải bỏ phiếu đa số — đây là quyết định có chủ,
   dựa trên bằng chứng đã tổng hợp. (Xem 04-decision-making.md về phân biệt
   consensus / consultative / command.)
   Ghi lại: quyết định, ba bằng chứng chính, lo ngại còn lại và kế hoạch giảm thiểu
   trong onboarding.

T+40 (5 phút): NẾU HIRE — CHUYỂN LO NGẠI SANG KẾ HOẠCH ONBOARDING
   Mọi lo ngại chưa được giải quyết phải trở thành một mục theo dõi trong kế hoạch 30/60/90.
   Đây là bước nối chủ đề 3 sang chủ đề 4 và là bước bị bỏ qua ở hầu hết mọi nơi.
```

Bước cuối đáng nhấn mạnh. Nếu panel lo rằng ứng viên "hơi yếu về khả năng viết tài liệu", thông tin đó
cực kỳ có giá trị — và ở hầu hết tổ chức, nó bốc hơi ngay khi buổi debrief kết thúc. Chuyển nó thành một
dòng trong kế hoạch 30 ngày ("viết một ADR trong tháng đầu, Tuấn review") biến một lo ngại thành một
phép đo có ngày.

**Bước 3 — Quy tắc xử lý bất đồng.**

| Tình huống | Cách xử lý |
|---|---|
| Tất cả Hire hoặc Strong Hire | Offer. Vẫn ghi lo ngại vào kế hoạch onboarding. |
| Đa số Hire, 1 No Hire | Bắt buộc thảo luận. Nếu No Hire nêu được bằng chứng map vào một năng lực bắt buộc mà không ai phản bác được bằng bằng chứng khác → **không tuyển**. Nếu lo ngại nằm ngoài scorecard → không tính. |
| Chia đôi | Không tuyển ở bậc này. Cân nhắc: (a) thêm một vòng nhắm đúng điểm lệch, (b) offer ở bậc thấp hơn nếu bậc thấp hơn thật sự phù hợp và ứng viên chấp nhận. Không "cứ thử xem sao". |
| Có 1 Strong No Hire | Không tuyển, trừ khi người đó rút lại sau khi nghe bằng chứng. Không được ép rút. |
| Panel toàn "Hire" nhưng không ai viết nổi lo ngại nào | Cờ đỏ về chất lượng phỏng vấn, không phải tín hiệu tốt về ứng viên. Xem lại chất lượng ghi chép trước khi quyết. |

**Bước 4 — Reference check đúng cách.**

Reference check thường vô dụng vì được làm sai: gọi cho người ứng viên chọn, hỏi câu đóng, nhận câu trả
lời lịch sự. Cách làm cho nó có giá trị:

- Hỏi **câu mở và so sánh**, không hỏi câu có/không. "Nếu xếp bạn ấy so với những engineer khác cùng bậc
  mà anh từng làm việc cùng, bạn ấy ở đâu?" — câu này buộc phải định vị, khó trả lời lấp liếm.
- Hỏi **câu nhắm vào lo ngại còn lại từ debrief**, không hỏi chung chung. Nếu lo ngại là khả năng làm
  việc độc lập: "Có lần nào bạn ấy phải tự quyết trong lúc anh không có mặt không? Chuyện gì xảy ra?"
- Hỏi câu **cho phép nói điều tiêu cực một cách an toàn**: "Nếu bạn ấy vào team em, em nên chú ý hỗ trợ
  điều gì trong ba tháng đầu?" Câu này thường cho nhiều thông tin nhất, vì nó đóng khung việc nói nhược
  điểm thành việc giúp đỡ.
- Chú ý **cái không được nói**. Một reference nói rất nhiệt tình về tính cách và không nói gì về công
  việc là một tín hiệu.
- **Không gọi cho công ty hiện tại của ứng viên khi chưa được phép rõ ràng.** Ở thị trường Việt Nam nơi
  mạng lưới hẹp, một cuộc gọi thiếu cẩn trọng có thể làm ứng viên mất việc hiện tại. Đây là ranh giới
  đạo đức, không phải thủ tục.
- **Đặt reference đúng vị trí trong chuỗi bằng chứng**: nó là tín hiệu bổ sung để xác nhận hoặc phản bác
  một lo ngại cụ thể, không phải là vòng quyết định. Đảo ngược một quyết định dựa trên bằng chứng trực
  tiếp từ 5 giờ phỏng vấn chỉ vì một cuộc gọi 15 phút là đánh giá sai trọng số.

### Trade-off

**Nâng chuẩn vs hạ chuẩn — phân tích hai chiều đầy đủ.**

| | Giữ / nâng chuẩn | Hạ chuẩn để lấp ghế |
|---|---|---|
| **Được** | Chất lượng team ổn định; chuẩn được bảo vệ cho các lần tuyển sau; người giỏi thấy đồng nghiệp xứng đáng và ở lại lâu hơn | Ghế được lấp trong 3–6 tuần thay vì 5 tháng; team giảm tải ngay; áp lực từ trên giảm |
| **Mất** | Ghế trống 4–8 tháng; team hiện tại quá tải, tăng rủi ro mất chính người giỏi; roadmap trượt; lead tiêu 20–30% thời gian cho tuyển dụng | Chi phí quản lý tăng liên tục (review kỹ hơn, giao việc hẹp hơn); chuẩn của team dịch xuống một nấc và rất khó dịch lên lại; rủi ro phải xử lý underperformance trong 2–4 quý |
| **Ai chịu phần mất** | Team hiện tại (những người không tham gia quyết định) | Team hiện tại, dưới dạng khác: gánh việc của người mới + mất niềm tin vào chuẩn |
| **Nghiêng về khi** | Vai có đòn bẩy cao (Senior, Lead, người sẽ định hình kiến trúc); tổ chức nhỏ nơi một người sai làm hỏng nhiều; đang có runway | Vai có phạm vi hẹp và giám sát được; có Senior dư băng thông để mentor thật; công việc đồng dạng có khuôn mẫu rõ |

Điều kiện thị trường Việt Nam cần nói thẳng, vì nó thay đổi phép tính:

1. **Khoảng cách giữa title và năng lực rất rộng.** Một "Senior" 4 năm ở công ty A có thể tương đương
   "Mid" ở công ty B. Hệ quả: bạn không thể dùng title trên CV làm bộ lọc; và bạn phải chuẩn bị tinh
   thần rằng số CV ghi Senior nhiều gấp nhiều lần số người đạt chuẩn Senior của bạn. Đây không phải lý
   do để bi quan mà là lý do để đo bằng rubric thay vì đo bằng nhãn.
2. **Dải ứng viên thật sự Senior mỏng và di chuyển nhanh.** Người giỏi thường có 2–3 quy trình song song
   và ra quyết định trong 1–2 tuần. Giữ chuẩn cao mà chậm = không tuyển được ai. Giữ chuẩn cao mà nhanh
   = khả thi. Tốc độ và chuẩn không phải là hai đầu của cùng một trục — đây là hiểu lầm tốn kém nhất.
3. **Nếu tổ chức bạn không thể trả top-of-market**, giữ chuẩn cao ở mọi năng lực là không khả thi. Lựa
   chọn trung thực: giữ chuẩn tuyệt đối ở **2–3 năng lực cốt lõi** và chấp nhận thấp hơn ở phần còn lại,
   với kế hoạch đào tạo cụ thể — chứ không phải hạ chuẩn đều ở mọi thứ. Hạ đều tạo ra team trung bình ở
   mọi mặt; hạ có chọn lọc tạo ra team mạnh ở đúng chỗ cần mạnh.

**Tốc độ ra quyết định vs độ chắc chắn.** Chờ thêm một ứng viên nữa để so sánh làm bạn có thông tin về
phân bố tốt hơn — nhưng bạn có thể mất ứng viên đang có. Điều kiện nghiêng: nếu ứng viên hiện tại đạt
mức Hire rõ ràng theo rubric, việc chờ so sánh với một ứng viên giả định là sai lầm về mặt xác suất —
bạn đang so một thứ đã đo được với một thứ chưa tồn tại. Chỉ nên chờ khi ứng viên hiện tại ở vùng phân
vân, vì lúc đó bạn dù sao cũng không nên offer.

### Real-world Scenarios

**Tình huống A — "Team đang thiếu người quá" trong phòng debrief.**

Bối cảnh: product startup, ghế Backend trống 4 tháng. Ứng viên vừa phỏng vấn xong. Kết quả độc lập: Tuấn
— Hire; Linh — Hire; Quân — No Hire (bằng chứng: trong vòng deep dive, không giải thích được vì sao chọn
cấu trúc dữ liệu trong chính bài mình nộp, và khi bị hỏi lại thì đổi câu trả lời ba lần); Khoa — No Hire
(bằng chứng: trong Behavioral, cả ba câu về thất bại đều quy nguyên nhân cho PO hoặc cho người khác).

Trong buổi debrief, Tuấn nói:

> **Phiên bản nói sai (Hà — hiring manager):**
> "Ừ, anh hiểu lo ngại của Quân và Khoa. Nhưng mà thật sự team mình đang thiếu người quá, sprint nào
> cũng carry-over. Bạn này ít nhất cũng làm được việc, mình tuyển vào rồi kèm dần. Không tuyển thì
> biết bao giờ mới có người. Thôi mình chốt hire nhé, mọi người thấy sao?"

Bốn thứ hỏng trong đoạn này. Một: đưa ràng buộc nguồn lực vào làm tiêu chí đánh giá năng lực — hai thứ
này phải tách. Hai: "kèm dần" là một cam kết không có tên người, không có thời gian, không có tiêu chí;
nó sẽ không xảy ra. Ba: câu hỏi cuối "mọi người thấy sao" sau khi sếp đã nêu quan điểm không phải là câu
hỏi, nó là yêu cầu đồng thuận. Bốn: bằng chứng cụ thể của Quân và Khoa bị gói lại thành "lo ngại" rồi bỏ
qua mà không được phản bác bằng bằng chứng.

> **Phiên bản nói đúng (Hà):**
> "Dừng lại một chút. Anh muốn tách hai việc ra, vì mình đang trộn chúng vào nhau.
>
> Việc thứ nhất: bạn này có đạt chuẩn L4 không. Quân, em nói trước vì em chấm thấp nhất. Em nói bạn ấy
> đổi câu trả lời ba lần về chính bài mình nộp — em đọc cái đó là gì? Là bạn ấy không tự làm bài, hay là
> bạn ấy làm nhưng không nhớ, hay là bạn ấy hoảng khi bị hỏi dồn?
>
> [Quân trả lời: có hai đoạn code trong bài dùng style hoàn toàn khác nhau, và khi hỏi về đoạn thứ hai
> thì bạn ấy không giải thích được logic của chính hàm đó.]
>
> Ok. Đó là bằng chứng cụ thể, không phải cảm giác. Tuấn, anh chấm Hire — anh có bằng chứng nào phản bác
> cái đó không? Không phải bằng chứng rằng bạn ấy giỏi mặt khác, mà bằng chứng phản bác đúng điểm này.
>
> [Tuấn: không có; vòng của anh là system design, không chạm vào code.]
>
> Vậy thì về mặt bằng chứng, mình có một tín hiệu âm cụ thể về tính trung thực của bài nộp và không có
> bằng chứng nào phản bác. Cộng thêm mẫu quy lỗi cho người khác trong cả ba câu Behavioral của Khoa. Hai
> tín hiệu này độc lập nhau và cùng chỉ một hướng. Quyết định của anh là không tuyển.
>
> Việc thứ hai — và đây là việc anh chịu trách nhiệm, không phải các em: team thiếu người 4 tháng nay là
> thật, và nó không được phép trở thành lý do hạ chuẩn, vì nếu nó thành lý do thì chuẩn của mình từ giờ
> là hàm của mức độ mình mệt. Anh sẽ xử lý bằng cách khác. Thứ Hai anh làm việc với Trang để cắt hai
> module khỏi quý này, và anh sẽ nói rõ với founder rằng đó là hệ quả của việc mình không hạ chuẩn. Nếu
> đến tháng sau vẫn không tuyển được, mình sẽ xem lại scorecard xem có tiêu chí nào mình đặt cao hơn nhu
> cầu thật không — nhưng đó là quyết định xem lại tiêu chí một cách có ý thức, khác với việc nhắm mắt
> cho một ứng viên đi qua."

Đoạn cuối là phần khó nhất và cũng là phần làm cho toàn bộ hệ thống đứng vững. Nếu bạn nói "không tuyển"
mà không đồng thời giải quyết áp lực đang đè lên team, lần sau sẽ không ai buồn chấm No Hire nữa — vì họ
học được rằng chấm No Hire chỉ có nghĩa là chính họ phải làm thêm việc.

**Tình huống B — Ứng viên từ công ty lớn, ba góc nhìn về cùng một tín hiệu.**

Ứng viên: 6 năm ở một tập đoàn công nghệ lớn, CV đẹp, nói năng mạch lạc. Trong vòng System Design, mô tả
rất trơn tru một kiến trúc microservices 12 service với service mesh, event sourcing, CQRS. Khi được hỏi
"nếu team chỉ có 6 người thì sao", trả lời: "Thì cần thêm người."

| | Nhìn từ IC (Vy, Mid BE trong panel) | Nhìn từ Tech Lead (Tuấn) | Nhìn từ Manager (Hà, EM) |
|---|---|---|---|
| **Đọc tín hiệu** | "Anh này giỏi thật, biết nhiều thứ mình chưa từng làm." | "Anh này mô tả được kiến trúc nhưng không biết chi phí vận hành của nó. Ở công ty cũ có platform team lo phần đó." | "Rủi ro là anh này sẽ đề xuất kiến trúc phù hợp với tổ chức 500 người vào một tổ chức 30 người, và team sẽ chết vì Cognitive Load." |
| **Bias đang hoạt động** | Anchoring vào thương hiệu công ty cũ + halo từ khả năng diễn đạt | Ít bias hơn ở điểm này nhưng có rủi ro ngược: coi thường kinh nghiệm quy mô lớn vì bản thân chưa có | Rủi ro over-correction: loại mọi ứng viên từ big tech vì "không phù hợp startup" — cũng là một dạng định kiến |
| **Câu hỏi cần hỏi thêm** | — | "Kể về một lần bạn chọn giải pháp đơn giản hơn giải pháp bạn thích. Vì sao?" | "Ở công ty cũ, phần nào của hệ thống bạn tự vận hành, phần nào có team khác lo?" |
| **Kết luận nếu chỉ nghe vai này** | Strong Hire (sai — dựa trên halo) | No Hire (có thể sai — chưa kiểm tra khả năng thích nghi) | Hoãn — cần thêm bằng chứng về khả năng làm việc dưới ràng buộc nguồn lực |

Điều đúng: thêm một câu hỏi có mục tiêu vào vòng còn lại, chứ không kết luận từ một quan sát. Tín hiệu
"quen làm ở tổ chức có platform team" không phải là tín hiệu xấu — nó là một **giả thuyết cần kiểm tra**
về khả năng vận hành trong điều kiện thiếu hạ tầng. Nhiều người từ big tech thích nghi rất tốt; nhiều
người không. Chỉ có câu hỏi nhắm đúng mới phân biệt được, và cả ba vai ở trên đều chưa hỏi câu đó.

### Best Practices

- **Khoá đánh giá: không ai xem được của người khác cho tới khi tất cả đã nộp.** Lý do: đây là biện pháp
  kỹ thuật duy nhất thực sự chặn được cascade; nhắc nhở bằng lời không có tác dụng.
- **Người chấm thấp nhất nói trước trong debrief.** Lý do: đảo ngược gradient áp lực xã hội, đặc biệt
  quan trọng ở môi trường có khoảng cách quyền lực rõ như nhiều tổ chức Việt Nam.
- **Bắt buộc mỗi người viết một "lo ngại lớn nhất", kể cả khi chấm Strong Hire.** Lý do: buộc tìm bằng
  chứng phản bác chính kết luận của mình — biện pháp đối kháng trực tiếp với confirmation bias.
- **Chỉ tranh luận trên bằng chứng nằm trong scorecard.** Lý do: mọi tiêu chí phát sinh trong phòng
  debrief đều là tiêu chí không được áp dụng đồng đều cho các ứng viên khác, tức là bất công và vô dụng
  cho việc so sánh.
- **Hiring manager quyết, không bỏ phiếu đa số.** Lý do: bỏ phiếu phân tán Accountability và thưởng cho
  việc theo số đông; quyết định có chủ tạo được người chịu trách nhiệm khi rà lại sau 12 tháng.
- **Chuyển mọi lo ngại còn lại vào kế hoạch onboarding.** Lý do: thông tin đắt nhất bạn có về người mới
  chính là những gì panel còn nghi ngờ; để nó bốc hơi là lãng phí toàn bộ chi phí phỏng vấn.
- **Lưu hồ sơ debrief tối thiểu 18 tháng và rà lại sau mỗi 10 lần tuyển.** Lý do: đây là dữ liệu duy nhất
  để biết panel của bạn đang dự báo đúng hay đang tự tin sai.

### Anti-patterns

**"Tuyển vì tuyệt vọng."** Ghế trống lâu → chuẩn trôi xuống mà không ai tuyên bố. Cơ chế: chuẩn là một
thứ ngầm, và mọi thứ ngầm đều trôi theo áp lực. Dấu hiệu sớm: tỉ lệ offer trên số ứng viên vòng cuối tăng
dần theo số tuần ghế trống; trong debrief xuất hiện các câu bắt đầu bằng "thôi thì" hoặc "ít nhất là".

**"Đánh giá dựa trên trường hoặc công ty cũ."** Cơ chế: anchoring. Điểm neo mạnh tới mức mọi bằng chứng
sau đó chỉ tạo ra điều chỉnh nhỏ quanh nó. Đặc biệt tai hại ở Việt Nam vì tương quan giữa trường và năng
lực nghề nghiệp sau 5 năm rất yếu, trong khi định kiến về trường thì rất mạnh. Dấu hiệu sớm: câu "trường
đó/công ty đó thì..." xuất hiện trong debrief; hoặc CV bị loại ở vòng sàng lọc mà lý do ghi lại là tên
tổ chức.

**"Bỏ qua tín hiệu về cách ứng xử vì kỹ thuật giỏi."** Ứng viên nói xấu đồng nghiệp cũ, ngắt lời người
phỏng vấn, hoặc thể hiện coi thường một công nghệ và người dùng nó — nhưng vòng coding xuất sắc. Panel
tự nhủ "kỹ năng mềm rèn được". Cơ chế phá hoại có hai tầng: (a) mẫu hành vi ổn định hơn kỹ năng kỹ thuật
rất nhiều, và người này sẽ ứng xử với đồng nghiệp mới đúng như đã ứng xử với đồng nghiệp cũ; (b) tác động
của một người phá vỡ Psychological Safety là **âm và lan rộng** — họ không chỉ đóng góp ít, họ làm giảm
đóng góp của người khác, nên năng suất kỹ thuật cao của họ không bù được. Dấu hiệu sớm: trong debrief có
câu "hơi thẳng tính" hoặc "chắc do áp lực thôi" đi kèm điểm kỹ thuật cao.

**"Cứ thử 2 tháng thử việc rồi tính."** Coi thử việc là phần kéo dài của phỏng vấn. Cơ chế: chi phí thật
của việc kết thúc trong thử việc cao hơn nhiều so với tưởng tượng — về pháp lý, về tinh thần người đó, về
tín hiệu gửi tới team, và về thời gian onboarding đã đổ vào. Thực tế ở phần lớn tổ chức, tỉ lệ chấm dứt
trong thử việc rất thấp không phải vì tuyển đúng mà vì không ai muốn làm việc đó. Dấu hiệu sớm: câu "thử
việc mà" xuất hiện như một lập luận trong debrief.

**"Debrief bằng chat."** Panel trao đổi trong một nhóm chat, người nhắn đầu tiên định khung toàn bộ. Cơ
chế: cascade ở dạng thuần khiết nhất, cộng thêm việc không ai đọc kỹ đánh giá của nhau. Dấu hiệu sớm:
không tồn tại buổi debrief nào trên lịch.

### Khi nào KHÔNG nên áp dụng

**Khi chỉ có một người phỏng vấn.** Ở công ty 5 người, quy trình debrief 6 bước với việc khoá đánh giá là
vô nghĩa — không có ai để chống cascade với. Cái nên làm thay: viết đánh giá theo rubric **trước khi**
nói chuyện với founder hoặc bất kỳ ai; và nhờ một người ngoài tổ chức làm một vòng để có ít nhất một phép
đo độc lập.

**Khi tuyển vai mà bạn là người duy nhất chịu hậu quả trực tiếp và tức thì.** Ví dụ: một freelancer làm
2 tuần cho một việc rời rạc. Chi phí sai thấp, có thể đảo ngược, và bộ máy đánh giá tốn nhiều hơn giá trị
nó tạo ra. Cái nên làm thay: giao một việc nhỏ có trả tiền, đánh giá kết quả thật.

**Khi ứng viên là người nội bộ chuyển team.** Bạn đã có 12–24 tháng dữ liệu hiệu suất thực tế — nguồn
bằng chứng mạnh hơn mọi vòng phỏng vấn. Chạy full loop ở đây vừa lãng phí vừa gửi tín hiệu rằng tổ chức
không tin dữ liệu của chính mình, và nó làm giảm động lực di chuyển nội bộ. Cái nên làm thay: một vòng
đánh giá năng lực **cụ thể còn thiếu** cho vai mới, cộng với trao đổi có cấu trúc với manager hiện tại
theo định dạng reference check ở trên.

**Khi ngưỡng bất đối xứng không còn đúng.** Nguyên tắc "khi phân vân thì không tuyển" dựa trên giả định
chi phí tuyển sai ≫ chi phí ghế trống. Có bối cảnh mà giả định này đảo: một dự án ODC có ràng buộc hợp
đồng về số người onboard trước ngày X, phạt vi phạm rất nặng, và vai có phạm vi hẹp, giám sát chặt. Ở đó
phép tính khác, và giả vờ nó không khác là tự lừa mình. Điều phải làm là **nói rõ ra rằng bạn đang tuyển
dưới ràng buộc khác thường**, ghi lại rủi ro đã chấp nhận, và thiết kế giám sát chặt hơn trong 90 ngày
đầu — chứ không phải lặng lẽ hạ chuẩn rồi vờ như đó vẫn là chuẩn cũ.

---

## 4. Onboarding

### Problem Statement

Nam vào ngày thứ Hai. Buổi sáng: HR nói về bảo hiểm, phát laptop, chụp ảnh thẻ. Buổi chiều: Tuấn dành 45
phút vẽ kiến trúc lên bảng, nói "hệ thống mình có 9 service, đây là service quan trọng nhất", rồi gửi
link Confluence và nói "em cứ đọc, có gì hỏi anh".

Tuần 1: Nam đọc tài liệu. Ba trong bốn hướng dẫn setup môi trường không chạy được; Nam mất hai ngày sửa,
không dám hỏi vì thấy ai cũng bận. Tuần 2: Nam được giao một task "làm quen" — sửa một dòng text trên
giao diện admin. Merge ngày thứ Năm. Tuần 3–4: Nam ngồi trong sprint planning, không hiểu 60% từ viết
tắt được nói ra, không hỏi. Tuần 5: được giao task thật, làm 4 ngày, PR bị 23 comment, Nam sửa 3 vòng.
Tuần 8: Nam vẫn hỏi Tuấn trước mọi quyết định. Tuần 12: trong 1-1, Nam nói "em thấy mình chưa đóng góp
được gì nhiều".

Câu đó chính xác. Và nó không phải lỗi của Nam.

Hiện tượng đo được khi onboarding hỏng:

- **Thời gian tới PR đầu tiên**: 9 ngày (nên là 1 ngày).
- **Thời gian tới quyết định độc lập đầu tiên**: chưa xảy ra sau 12 tuần.
- **Thời gian tới net-positive** (đóng góp của họ vượt lượng thời gian họ tiêu của người khác): ước lượng
  5–6 tháng thay vì 2–3 tháng.
- **Tỉ lệ rời trong 6 tháng đầu**: ở các tổ chức có onboarding kiểu này, đây là cửa thoát lớn nhất, và
  gần như hoàn toàn tránh được.

Chi phí của việc kéo dài giai đoạn này thêm 3 tháng, tính theo bảng ở chủ đề 1: khoảng 3 tháng lương chi
phí cơ hội trực tiếp, cộng với xác suất mất người tăng lên đáng kể — và nếu mất, toàn bộ 12–19 tháng
lương của một lần tuyển sai phải trả lại từ đầu. Nói cách khác: **một tuần đầu tư vào onboarding có tỉ
suất hoàn vốn cao hơn gần như mọi việc khác một lead có thể làm trong tuần đó.**

### First Principles

**Giá trị của một người mới bằng 0 hoặc âm trong giai đoạn đầu.** Điều này không phải là nhận xét tiêu
cực, nó là số học. Một người mới:

- Chưa sản xuất được gì có giá trị vì chưa biết hệ thống.
- **Tiêu** thời gian của người đang có: mỗi câu hỏi làm gián đoạn một người khác; mỗi PR cần review kỹ
  hơn bình thường; mỗi quyết định của họ cần được kiểm tra.

Đường cong điển hình (số minh hoạ, Senior BE, hệ thống trung bình phức tạp):

```
Đóng góp ròng (đóng góp của họ − thời gian họ tiêu của người khác)

  +100% |                                        ┌──────── (ổn định)
        |                                   ┌────┘
   +50% |                              ┌────┘
        |                         ┌────┘
     0  |─────────────────┬───────┘  ← điểm net-positive
        |            ┌────┘
   -50% |───────┬────┘
        |───────┘
  -100% └──┬────┬────┬────┬────┬────┬────┬────┬────┬────►
        T0  W2   W4   W6   W8  W10  W12  W16  W20  W24

Đường trên: onboarding có thiết kế → net-positive ở khoảng tuần 6–8
Đường điển hình khi thả tự bơi → net-positive ở khoảng tuần 18–24
Diện tích giữa hai đường = giá trị của việc thiết kế onboarding
```

Mục tiêu của onboarding không phải là "làm người mới thấy được chào đón". Đó là hệ quả phụ, tốt nhưng
không phải mục tiêu. Mục tiêu là **rút ngắn hai khoảng thời gian đo được**:

1. **Thời gian tới đóng góp ròng dương** — khi họ tạo ra nhiều hơn họ tiêu.
2. **Thời gian tới quyết định độc lập đầu tiên** — khi họ tự quyết một việc có hệ quả thật mà không hỏi.

Chỉ số thứ hai quan trọng hơn và ít được đo hơn. Một người có thể viết code suốt 6 tháng mà chưa từng tự
quyết gì; người đó chưa được onboard, họ chỉ đang được sai việc.

**Ba lớp kiến thức phải chuyển giao — và lớp thứ ba là lớp không ai viết ra.**

| Lớp | Nội dung | Nguồn hiện có | Thời gian tự học nếu không được dạy |
|---|---|---|---|
| **1. Hệ thống** | Kiến trúc, dịch vụ, dữ liệu, môi trường, công cụ | Tài liệu, code | 4–8 tuần (khả thi nhưng chậm) |
| **2. Quy trình** | Cách một thay đổi đi từ ý tưởng tới production; quy tắc review; cách release; cách xử lý incident | Tài liệu (thường lỗi thời) | 3–6 tuần |
| **3. Bối cảnh xã hội** | Ai biết cái gì; ai thực sự ra quyết định gì; việc nào có lịch sử tranh cãi và vì sao; hỏi ai thì được trả lời, hỏi ai thì bị coi thường; điều gì nói được trong họp và điều gì chỉ nói riêng | **Không có ở đâu** | 6–12 tháng, hoặc không bao giờ |

Lớp 3 là lớp quyết định tốc độ thật, và là lớp duy nhất không thể học bằng cách đọc. Nó cũng là lý do vì
sao một người rất giỏi có thể trông như kém hiệu quả trong 4 tháng đầu: họ biết làm, nhưng không biết
**làm với ai và theo đường nào**. Onboarding có thiết kế là onboarding coi lớp 3 là nội dung chính thức
cần chuyển giao, chứ không phải thứ "rồi khắc biết".

**Vì sao onboarding luôn bị bỏ.** Cơ chế đã nêu ở mở đầu chương: nó không có deadline và không có sự kiện
kết thúc. Thêm hai cơ chế:

- *Curse of knowledge.* Người đã ở lâu không còn nhớ mình đã từng không biết gì. Họ nói "chỉ cần đọc
  README là hiểu" — README mà chính họ đã dành 8 tháng để xây dựng ngữ cảnh xung quanh.
- *Chi phí onboarding rơi vào một người, lợi ích rơi vào tổ chức.* Tuấn bỏ 8 giờ/tuần để onboard Nam; sản
  lượng cá nhân của Tuấn giảm; sản lượng của tổ chức tăng — sau ba tháng. Nếu Tuấn được đánh giá theo sản
  lượng cá nhân, hệ thống khuyến khích đang chống lại onboarding. Đây là một Principal-Agent Problem cổ
  điển, và nó chỉ sửa được ở tầng đánh giá hiệu suất (chủ đề 6), không sửa được bằng cách nhắc Tuấn cố
  gắng hơn.

### Mental Model

**Onboarding là một quy trình giảm bất định, không phải một quy trình truyền tin.**

Người mới có một tập giả định về hệ thống, phần lớn sai. Mỗi lần họ hành động và nhận phản hồi, một giả
định được kiểm tra. Tốc độ onboarding tỉ lệ thuận với **số vòng phản hồi họ hoàn thành trên đơn vị thời
gian**, không tỉ lệ với lượng tài liệu họ đọc.

```
Đọc tài liệu:     [nạp thông tin] → (không có phản hồi) → giả định sai vẫn còn nguyên
Làm việc thật:    [hành động] → [phản hồi] → [sửa mô hình] → lặp

Tốc độ onboarding ≈ số vòng lặp / tuần
```

Suy ra trực tiếp: mọi thiết kế onboarding nên tối đa hoá số vòng lặp sớm, kể cả khi mỗi vòng nhỏ. Một
người làm 10 thay đổi nhỏ trong tuần đầu học nhanh hơn nhiều so với người đọc 200 trang trong tuần đầu
rồi làm một việc lớn ở tuần ba. Đây cũng là lý do nguyên tắc "PR đầu tiên trong ngày đầu tiên" không phải
nghi lễ chào mừng — nó là cách khởi động vòng lặp sớm nhất có thể.

**Cognitive Load Theory.** Người mới đang chịu ba loại tải cùng lúc: tải nội tại (bài toán khó), tải
ngoại lai (công cụ lạ, quy trình lạ, tên viết tắt lạ), và tải xây dựng mô hình. Tổng công suất nhận thức
là hữu hạn. Mọi thứ bạn loại bỏ khỏi tải ngoại lai — script setup chạy một lệnh, bảng thuật ngữ viết tắt,
sơ đồ một trang — đều chuyển trực tiếp thành công suất cho việc học cái thực sự quan trọng. Đây là lý do
kỹ thuật vì sao "mất hai ngày sửa script setup" không phải là chuyện nhỏ: nó tiêu 40% công suất nhận thức
của tuần đầu vào thứ có giá trị bằng 0.

### Practical Framework

**Nguyên tắc 1 — PR đầu tiên trong ngày đầu tiên.**

Điều kiện để làm được: có sẵn một danh sách "task ngày đầu" được duy trì liên tục — lỗi chính tả trong
tài liệu, thêm một log, sửa một message lỗi, thêm một test case nhỏ. Việc này nhỏ về nội dung nhưng lớn
về tác dụng: nó buộc người mới đi qua **toàn bộ đường ống** trong ngày đầu (clone, build, chạy test, tạo
branch, mở PR, được review, merge, thấy nó lên staging). Mọi chỗ hỏng trong đường ống lộ ra ngay ngày
đầu, khi có người sẵn sàng ngồi cùng, thay vì lộ ra ở tuần ba khi người mới đã kết luận rằng mình chậm.

Tác dụng phụ có giá trị: người mới có một lần thành công trong 8 giờ đầu. Đây là điều kiện tâm lý cần cho
việc dám hỏi trong tuần tiếp theo.

**Nguyên tắc 2 — Buddy khác Manager, và khác Mentor.** Ba vai, ba mục đích, không được gộp:

| Vai | Ai | Tần suất | Trả lời câu gì | Không làm gì |
|---|---|---|---|---|
| **Buddy** | Một đồng nghiệp ngang cấp hoặc hơn một bậc, cùng team | Hằng ngày trong 2 tuần đầu, sau đó theo nhu cầu | "Câu hỏi ngu ngốc": lệnh này chạy thế nào, ai là người hỏi về X, từ viết tắt này nghĩa là gì, giờ ăn trưa mọi người làm gì | Không đánh giá, không báo cáo lên manager |
| **Manager** | EM hoặc Tech Lead | 1-1 hằng tuần trong 3 tháng đầu (thay vì 2 tuần/lần) | Kỳ vọng, tiến độ, phản hồi, chặn đường | Không phải nơi hỏi câu vận hành hằng ngày |
| **Mentor kỹ thuật** | Senior sở hữu mảng liên quan | 2 lần/tuần trong 4 tuần đầu | Vì sao hệ thống thế này, lịch sử quyết định, chỗ nào nguy hiểm | Không làm hộ |

Điểm then chốt về Buddy: **buddy phải an toàn**. Nếu buddy báo cáo ấn tượng về người mới cho manager,
người mới sẽ ngừng hỏi câu ngu ngốc trong tuần thứ hai, và bạn mất kênh chuyển giao lớp 3. Ranh giới này
phải được nói rõ với cả hai người vào ngày đầu.

**Nguyên tắc 3 — Chuyển lo ngại từ debrief thành mục theo dõi.** Đã nêu ở chủ đề 3. Nếu panel lo về khả
năng viết tài liệu, kế hoạch 30 ngày phải có một mục viết tài liệu có người review và có ngày.

**Template kế hoạch 30/60/90.**

```
KẾ HOẠCH 30/60/90 — [Tên], [Vai], bậc L__
Manager: Hà (EM)    Buddy: Vy    Mentor kỹ thuật: Quân
Ngày bắt đầu: __/__/____
Lo ngại từ debrief cần theo dõi: [chép nguyên văn từ hồ sơ debrief]

════════════════ NGÀY 1 ════════════════
[ ] Môi trường chạy được: build + test suite pass trên máy cá nhân   (Buddy ngồi cùng)
[ ] Truy cập: repo, CI, staging, log, monitoring, kênh chat, lịch    (chuẩn bị TRƯỚC ngày 1)
[ ] Một PR được merge (từ danh sách task ngày đầu)
[ ] Gặp Buddy 30', gặp Manager 30', biết tên và vai của cả team
[ ] Nhận: sơ đồ kiến trúc 1 trang + bảng thuật ngữ viết tắt + danh sách "ai biết cái gì"

════════════════ 30 NGÀY — MỤC TIÊU: HIỂU VÀ ĐÓNG GÓP NHỎ ĐỘC LẬP ════════════════
Kết quả kiểm được:
[ ] Đã merge ≥ 5 PR, trong đó ≥ 2 PR chạm vào logic nghiệp vụ (không chỉ sửa text/config)
[ ] Tự setup lại được môi trường từ đầu mà không cần hỏi — và ĐÃ SỬA tài liệu setup
    ở những chỗ sai (đây vừa là bài kiểm tra vừa là đóng góp thật)
[ ] Vẽ lại được sơ đồ luồng chính của hệ thống bằng trí nhớ và giải thích cho Buddy
    nghe trong 15 phút; Buddy ghi lại 3 chỗ hiểu sai
[ ] Đã review ≥ 3 PR của người khác với comment có nội dung (không chỉ "LGTM")
[ ] Đã trực on-call cùng người khác ít nhất 1 ca (shadow, không chịu trách nhiệm)
[ ] Biết hỏi ai cho 5 chủ đề: hạ tầng, dữ liệu, nghiệp vụ thanh toán, frontend, đối tác
Manager kiểm tra vào ngày 30:
    - Người này còn bị chặn ở đâu? Chặn vì kiến thức hay vì quyền truy cập?
    - Họ đã hỏi bao nhiêu câu trong tuần vừa rồi? (Quá ít cũng là cờ đỏ, không chỉ quá nhiều)

════════════════ 60 NGÀY — MỤC TIÊU: SỞ HỮU MỘT MẢNG NHỎ ════════════════
Kết quả kiểm được:
[ ] Sở hữu một thành phần cụ thể (tên: ________). "Sở hữu" nghĩa là: người khác hỏi họ
    về thành phần đó, và họ là người review PR chạm vào nó.
[ ] Đã hoàn thành một task cỡ vừa (3–5 ngày) end-to-end: từ làm rõ yêu cầu với PO
    tới deploy lên production
[ ] Đã đưa ra ít nhất 1 quyết định kỹ thuật độc lập (không hỏi trước) và giải thích được
    lý do khi được hỏi sau  ← CHỈ SỐ QUAN TRỌNG NHẤT CỦA MỐC NÀY
[ ] Đã trực on-call độc lập ít nhất 1 ca với backup
[ ] Đã chỉ ra được ít nhất 1 điều team đang làm mà họ thấy nên khác đi (không cần đúng —
    cần chứng tỏ họ đã có mô hình riêng và dám nói)
Manager kiểm tra vào ngày 60:
    - Nếu chưa có quyết định độc lập nào: nguyên nhân là thiếu kiến thức, thiếu quyền,
      hay thiếu Psychological Safety? Ba nguyên nhân, ba cách chữa khác nhau.

════════════════ 90 NGÀY — MỤC TIÊU: NET-POSITIVE VÀ CÓ HƯỚNG ════════════════
Kết quả kiểm được:
[ ] Đóng góp ròng dương: đã hết cần thời gian mentor định kỳ; PR qua review với số vòng
    tương đương người cùng bậc trong team
[ ] Đã dẫn dắt một việc có sự tham gia của người khác (một task đa người, một buổi
    thiết kế, hoặc onboard chính người mới tiếp theo)
[ ] Với bậc Senior trở lên: đã viết ít nhất 1 ADR hoặc tài liệu thiết kế được team review
[ ] Có bản đồ phát triển 12 tháng: mục tiêu bậc, năng lực cần xây (nối vào Career Ladder)
Manager kiểm tra vào ngày 90:
    - Đối chiếu với mục B của Role Scorecard: người này có đang trên quỹ đạo đạt
      các kết quả 12 tháng không?
    - Nếu KHÔNG: đây là thời điểm rẻ nhất để can thiệp. Xem chủ đề 8.
    - Buổi feedback hai chiều: hỏi người mới "3 điều gì trong onboarding của em là lãng phí
      thời gian nhất?" và SỬA cho người tiếp theo.
```

Câu cuối cùng là cơ chế duy nhất làm cho onboarding tự cải thiện. Người mới là người duy nhất trong tổ
chức nhìn thấy được sự bất hợp lý của tổ chức đó, và cửa sổ nhìn thấy đó đóng lại sau khoảng 3 tháng. Nếu
bạn không hỏi trong cửa sổ đó, thông tin mất vĩnh viễn.

**Onboarding Senior khác Junior thế nào.** Đây là chỗ sai phổ biến nhất: giả định rằng Senior cần ít
onboarding hơn. Sai. Họ cần onboarding **khác loại**, và thường cần nhiều hơn ở lớp 3.

| | Junior / Mid | Senior / Lead |
|---|---|---|
| **Thiếu chủ yếu** | Lớp 1 (hệ thống) và lớp 2 (quy trình) | Lớp 3 (bối cảnh xã hội, lịch sử quyết định, ai có quyền gì) |
| **Rủi ro chính** | Chìm trong độ phức tạp, không dám hỏi, làm sai âm thầm | Áp mẫu hình từ nơi cũ vào bối cảnh không phù hợp; hoặc mất uy tín vì đề xuất thứ đã từng bị bác và không biết vì sao |
| **Nội dung cần đặc thù** | Task nhỏ có cấu trúc rõ, pair programming nhiều, checklist chi tiết | Lịch sử các quyết định lớn (đọc ADR cũ, kể cả ADR bị bác); bản đồ quyền lực thực tế; danh sách "những việc đã thử và thất bại, và vì sao" |
| **Sai lầm điển hình của manager** | Giao việc thật quá muộn | Giả định họ tự bơi được; không giới thiệu vào các cuộc trao đổi có ảnh hưởng; để họ phát biểu đề xuất lớn ở tuần 2 mà không cảnh báo về mìn |
| **Mốc "quyết định độc lập đầu tiên"** | Tuần 8–10 | Tuần 3–4 — và nếu chưa xảy ra ở tuần 6 thì có vấn đề |
| **Việc manager phải chủ động làm** | Bảo vệ khỏi việc quá tải | Chủ động tạo cơ hội cho họ thể hiện năng lực trước những người sẽ đánh giá họ — uy tín của Senior mới hình thành trong 6 tuần đầu và rất khó sửa sau đó |

Dòng cuối đáng dừng lại. Với một Senior mới, **uy tín kỹ thuật được thiết lập rất sớm và rất dính**. Nếu
6 tuần đầu họ chỉ làm việc lặt vặt vì "chưa quen hệ thống", team sẽ mặc định định vị họ ở mức đó, và họ
phải mất nhiều tháng để leo lên vị trí lẽ ra thuộc về họ. Ở bối cảnh Việt Nam nơi tuổi tác và thâm niên
ảnh hưởng tới uy tín kỹ thuật, hiệu ứng này còn mạnh hơn — một Senior mới trẻ tuổi hơn các thành viên cũ
cần được manager chủ động tạo sân khấu, nếu không họ sẽ bị đọc là "lính mới" bất kể năng lực.

**Đo hiệu quả onboarding bằng gì.**

| Chỉ số | Cách đo | Ngưỡng tham chiếu (số minh hoạ) |
|---|---|---|
| Thời gian tới PR đầu tiên được merge | Từ log Git | ≤ 1 ngày |
| Thời gian tới PR chạm logic nghiệp vụ | Từ log Git + phân loại | ≤ 2 tuần |
| Thời gian tới quyết định độc lập đầu tiên | Manager ghi nhận trong 1-1 | Senior ≤ 4 tuần; Mid ≤ 8 tuần |
| Số vòng review trung bình của PR, theo tuần | Từ công cụ | Hội tụ về mức của team trong 8–12 tuần |
| Số câu hỏi/tuần | Quan sát định tính | Tăng trong 2 tuần đầu, giảm dần; **giảm sớm là cờ đỏ** |
| Tỉ lệ rời trong 6 tháng đầu | HR | Theo dõi theo cohort, không theo cá nhân |
| Điểm khảo sát ngày 30 và 90 | 3 câu hỏi cố định | So sánh giữa các cohort để thấy xu hướng |

Chỉ số "số câu hỏi/tuần giảm sớm là cờ đỏ" hay bị đọc ngược. Người mới ngừng hỏi ở tuần 3 thường không
phải vì đã hiểu, mà vì đã học được rằng hỏi thì phiền người khác. Từ điểm đó, họ bắt đầu đoán, và bạn sẽ
phát hiện hậu quả ở tuần 10.

### Trade-off

**Onboarding cấu trúc chặt vs thả cho tự khám phá.** Cấu trúc chặt cho tốc độ đồng đều, giảm lo âu, đảm
bảo không ai bị bỏ sót — nhưng tốn công duy trì (một checklist lỗi thời còn tệ hơn không có), và với
người rất chủ động thì nó gây khó chịu, cho cảm giác bị coi thường. Thả tự khám phá rẻ và phù hợp với
người tự lực cao — nhưng tạo phương sai lớn: một số người bơi rất tốt, một số chìm âm thầm, và bạn chỉ
biết sau ba tháng. Điều kiện nghiêng: nếu bạn tuyển ≥ 4 người/năm cho cùng loại vai, cấu trúc có lợi rõ
(chi phí duy trì được phân bổ). Nếu tuyển 1–2 người/năm, đầu tư vào **một buddy tốt** cho lợi ích cao hơn
đầu tư vào tài liệu.

**Giao việc thật sớm vs cho thời gian học.** Giao sớm khởi động vòng phản hồi và cho người mới cảm giác
hữu ích — nhưng rủi ro là họ đưa ra quyết định sai trên nền hiểu biết mỏng, tạo nợ kỹ thuật mà người khác
phải trả. Cho thời gian học giảm rủi ro đó nhưng kéo dài giai đoạn âm và làm mờ tín hiệu về năng lực thật.
Điều kiện nghiêng: sớm hay muộn không phải là trục đúng. Trục đúng là **kích thước và tính đảo ngược của
việc được giao**. Giao ngay ngày đầu, nhưng giao việc nhỏ và dễ đảo ngược; tăng dần kích thước theo số
vòng phản hồi đã hoàn thành, không theo số ngày đã trôi qua.

**Đầu tư vào tài liệu vs đầu tư vào thời gian người.** Tài liệu mở rộng được (viết một lần, dùng nhiều
lần) nhưng lỗi thời nhanh và không truyền được lớp 3. Thời gian người truyền được lớp 3 nhưng đắt và
không mở rộng được. Điều kiện nghiêng: dùng tài liệu cho lớp 1 và lớp 2 (những thứ ổn định, kiểm chứng
được), dùng thời gian người cho lớp 3 (những thứ không viết ra được). Cố gắng viết tài liệu cho lớp 3 —
"văn hoá công ty", "cách chúng ta làm việc" — hầu như luôn tạo ra văn bản không ai đọc.

### Real-world Scenarios

**Tình huống A — Onboarding vào dự án ODC đang chạy, khách hàng Nhật, hạn chế truy cập.**

Bối cảnh: dự án đã chạy 14 tháng, 9 người phía Việt Nam, khách hàng cấp quyền truy cập theo quy trình
phê duyệt mất 5–10 ngày làm việc. Code nằm trên hạ tầng của khách. Không được cài công cụ ngoài danh
sách duyệt. Tài liệu thiết kế bằng tiếng Nhật, một phần bằng tiếng Anh dịch máy.

Đây là bối cảnh mà onboarding tiêu chuẩn sụp đổ hoàn toàn, vì nguyên tắc "PR trong ngày đầu" bất khả thi
về mặt vật lý. Cách xử lý:

1. **Bắt đầu quy trình xin quyền truy cập từ ngày ký offer, không phải ngày vào làm.** Đây là việc rẻ
   nhất và bị bỏ qua nhiều nhất. Nếu quy trình mất 10 ngày và bạn bắt đầu vào ngày đầu tiên, bạn đã tự
   nguyện đốt hai tuần.
2. **Dựng một môi trường thay thế.** Không truy cập được hệ thống thật thì dựng một bản sao rút gọn với
   dữ liệu giả, chạy trên hạ tầng nội bộ. Không hoàn hảo, nhưng nó cho phép vòng lặp phản hồi bắt đầu.
3. **Đổi định nghĩa "PR đầu tiên".** Nếu không commit được vào repo khách, thì sản phẩm ngày đầu là một
   thứ khác có phản hồi: tóm tắt một tài liệu thiết kế và trình bày lại cho Tech Lead nghe trong 15 phút,
   để lead chỉ ra chỗ hiểu sai. Vẫn là một vòng lặp hoàn chỉnh.
4. **Chuyển lớp 3 sang thành nội dung chính thức.** Ở ODC, lớp 3 có thêm một tầng: bối cảnh xã hội phía
   khách hàng. Ai bên khách có quyền quyết? Câu hỏi nào gửi qua BrSE, câu nào hỏi trực tiếp? Mức độ chi
   tiết khách kỳ vọng trong báo cáo? Việc gì tuyệt đối không được tự quyết? Viết ra một trang gọi là
   "cách làm việc với khách hàng này" và giao cho người mới đọc trong ngày đầu — đây là tài liệu có giá
   trị cao nhất trong toàn bộ bộ onboarding của một dự án ODC.
5. **Ghép cặp trong 2 tuần đầu thay vì giao task riêng.** Khi môi trường bị hạn chế, pair programming là
   cách duy nhất giữ được số vòng phản hồi cao.

**Tình huống B — Senior tự bơi (product startup, 40 engineer).**

Quân, Senior BE 8 năm kinh nghiệm, vào team. Hà nghĩ: "Anh này senior rồi, không cần cầm tay." Không có
buddy, không có kế hoạch 30/60/90, 1-1 hai tuần một lần.

Tuần 3, trong buổi thiết kế, Quân đề xuất chuyển từ REST sang gRPC cho giao tiếp nội bộ, trình bày rất
thuyết phục. Phòng im lặng. Sau buổi họp, Tuấn nhắn riêng cho Hà: "Chuyện này team đã tranh luận 6 tháng
trước, quyết định không làm vì hai team frontend không có băng thông. Bây giờ nói lại làm mọi người khó
chịu."

Không ai nói với Quân điều này. Quân đọc sự im lặng là "team bảo thủ". Tuần 6, Quân bắt đầu ít phát biểu.
Tuần 12, Quân vẫn làm việc tốt về mặt kỹ thuật nhưng đã tự định vị mình ở ngoài các cuộc thảo luận có ảnh
hưởng.

Điều đã hỏng: Hà nhầm giữa "cần ít hướng dẫn kỹ thuật" và "cần ít onboarding". Quân không thiếu lớp 1;
Quân thiếu lớp 3, và không có ai được giao nhiệm vụ chuyển giao nó.

Cách sửa, và nó rẻ hơn nhiều so với vẻ ngoài:

- Một tài liệu một trang: **"Những quyết định lớn của team trong 18 tháng qua, và vì sao"** — mỗi mục ba
  dòng: quyết định gì, bối cảnh lúc đó, còn đúng không. Bao gồm cả những đề xuất đã bị bác. Tài liệu này
  mất khoảng 2 giờ để viết và tiết kiệm hàng tháng cho mỗi Senior mới.
- Một buổi 60 phút với Tuấn trong tuần đầu, nội dung duy nhất: "kể cho em nghe những cuộc tranh luận lớn
  đã xảy ra ở đây".
- Một câu Hà nên nói vào ngày đầu:

> **Nói sai:** "Anh cứ tự nhiên, có gì hỏi mọi người. Bên này khá thoải mái."
>
> **Nói đúng:** "Có ba việc em muốn nói rõ với anh về 6 tuần đầu.
> Một: uy tín của anh trong team sẽ hình thành trong khoảng 6 tuần này và sau đó rất khó đổi, nên em sẽ
> chủ động ghép anh vào hai việc có độ hiển thị cao — không phải để thử anh, mà để team thấy anh làm
> việc.
> Hai: trước khi anh đề xuất bất kỳ thay đổi kiến trúc nào ở buổi họp chung, anh nói với Tuấn trước một
> lượt. Không phải để xin phép — mà vì có một số thứ team đã tranh luận và quyết rồi, anh cần biết lịch
> sử để không mất công. Em sẽ gửi anh tài liệu các quyết định cũ.
> Ba: em kỳ vọng trong 4 tuần đầu anh tự quyết ít nhất một việc kỹ thuật mà không hỏi ai. Nếu tới tuần 4
> mà chưa có, đó là tín hiệu em đã không giao đủ quyền cho anh, và em muốn biết."

Ba câu này làm được ba việc mà không câu chào mừng nào làm được: đặt kỳ vọng đo được, cảnh báo về mìn, và
đặt trách nhiệm về việc trao quyền lên chính manager.

**Tình huống C — Cùng một tuần đầu, ba góc nhìn.**

Người mới: Duy, Mid BE, tuần thứ nhất, đã hỏi Vy khoảng 20 câu.

| | Nhìn từ IC (Vy — buddy) | Nhìn từ Tech Lead (Tuấn) | Nhìn từ Manager (Hà) |
|---|---|---|---|
| **Điều nhìn thấy** | "Mình mất 2 tiếng/ngày trả lời câu hỏi, sprint của mình đang trễ." | "Duy hỏi nhiều nhưng câu hỏi ngày càng sâu — dấu hiệu tốt." | "Vy đang bị tính velocity như bình thường trong khi làm buddy. Đó là lỗi của hệ thống, không phải của Vy." |
| **Rủi ro nếu không xử lý** | Vy bắt đầu trả lời qua loa, Duy ngừng hỏi | Đánh giá đúng nhưng không có hành động → Vy kiệt sức | Nếu bỏ qua: lần sau không ai nhận làm buddy |
| **Hành động đúng** | Nói thẳng với Hà về chi phí thời gian, thay vì âm thầm chịu; gom câu hỏi vào 2 khung giờ/ngày | Ghi lại các câu hỏi của Duy làm đầu vào sửa tài liệu | Giảm cam kết sprint của Vy 30% trong 3 tuần, nói rõ với cả team rằng đây là công việc được tính, không phải việc làm thêm |

Bài học ở tình huống này không nằm ở người mới mà nằm ở **cấu trúc khuyến khích**. Nếu công sức onboarding
không được tính vào khối lượng công việc chính thức, nó sẽ được làm qua loa — không phải vì ai đó lười,
mà vì hệ thống đo lường đang trừng phạt người làm nó. Đây là điểm nối trực tiếp sang chủ đề 6.

### Best Practices

- **Chuẩn bị toàn bộ quyền truy cập trước ngày đầu tiên.** Lý do: mỗi ngày chờ quyền là một ngày tiêu
  công suất nhận thức vào việc chờ đợi, và là tín hiệu đầu tiên người mới nhận được về mức độ tổ chức
  quan tâm tới họ.
- **Duy trì một danh sách "task ngày đầu" luôn có sẵn 5–10 mục.** Lý do: không có nó thì nguyên tắc PR
  ngày đầu không thực thi được, và mọi người sẽ nói "lần này không có việc phù hợp".
- **Giao cho người mới nhiệm vụ sửa tài liệu onboarding.** Lý do: họ là người duy nhất phát hiện được chỗ
  sai, đây là đóng góp thật ngay tuần đầu, và nó tạo vòng lặp tự cải thiện cho hệ thống.
- **Tính công việc buddy vào khối lượng chính thức, giảm cam kết sprint tương ứng.** Lý do: xem Tình
  huống C — nếu không làm, hệ thống khuyến khích chống lại chính chính sách của bạn.
- **1-1 hằng tuần trong 3 tháng đầu, không phải hai tuần một lần.** Lý do: đây là giai đoạn có nhiều bất
  định nhất và cũng là giai đoạn mà một hiểu lầm nhỏ nếu không được sửa sẽ đông cứng thành mô hình sai.
- **Viết "danh sách quyết định lớn và vì sao" cho team, cập nhật mỗi quý.** Lý do: đây là cách duy nhất
  chuyển giao lớp 3 ở dạng viết được, và nó có giá trị cho cả người cũ.
- **Hỏi người mới ở ngày 30 và 90: "cái gì lãng phí thời gian nhất?" và sửa.** Lý do: cửa sổ nhìn thấy
  bất hợp lý của tổ chức chỉ mở trong khoảng 3 tháng.

### Anti-patterns

**"Đây là tài liệu, em đọc đi."** Chuyển giao dưới dạng nạp thông tin một chiều. Cơ chế phá hoại: không
có vòng phản hồi, nên giả định sai không được sửa; và tài liệu luôn lỗi thời ở đúng những chỗ quan trọng
nhất. Dấu hiệu sớm: người mới trong tuần đầu chủ yếu đọc, không chạy được gì.

**"Giao việc thật quá muộn."** Ba tuần đầu chỉ làm task rác vì "chưa quen". Cơ chế: người mới không xây
được mô hình vì không có phản hồi từ việc có hệ quả; đồng thời họ hình thành nhận thức rằng mình không
được tin tưởng, thứ ảnh hưởng tới mức độ chủ động trong nhiều tháng sau. Dấu hiệu sớm: tới ngày 21, người
mới chưa chạm vào code có logic nghiệp vụ.

**"Senior thì tự bơi được."** Xem Tình huống B. Cơ chế: nhầm lớp 1 với lớp 3. Dấu hiệu sớm: Senior mới
không có buddy; hoặc tới tuần 6 chưa được mời vào bất kỳ cuộc thảo luận thiết kế nào.

**"Onboarding là việc của HR."** HR làm phần hành chính và văn hoá chung; phần kỹ thuật và lớp 3 không ai
sở hữu. Cơ chế: trách nhiệm rơi vào khoảng trống giữa hai bộ phận. Dấu hiệu sớm: khi hỏi "ai chịu trách
nhiệm cho việc Nam đạt net-positive trong 8 tuần", không có tên cụ thể.

**"Onboard 5 người cùng một lúc."** Sau một đợt tuyển, 5 người vào cùng tuần. Cơ chế: năng lực mentor của
team là hữu hạn và không co giãn; vượt tỉ lệ 1 Senior : 2 người mới thì chất lượng onboarding sụp đổ cho
tất cả, kể cả những người lẽ ra đã ổn. Dấu hiệu sớm: lịch của các Senior trong tháng đó kín các buổi
"hướng dẫn"; velocity của team giảm mạnh và không hồi phục sau 6 tuần.

**"Người mới im lặng = người mới ổn."** Cơ chế: im lặng có hai nguyên nhân ngược nhau (đã hiểu hết, hoặc
đã bỏ cuộc việc hỏi) và nguyên nhân thứ hai phổ biến hơn nhiều — đặc biệt trong môi trường có khoảng cách
quyền lực rõ. Dấu hiệu sớm: số câu hỏi giảm mạnh sau tuần 2 mà không kèm theo tăng số PR độc lập.

### Khi nào KHÔNG nên áp dụng

**Khi người vào làm hợp đồng rất ngắn cho một việc hẹp.** Một freelancer 3 tuần làm một tích hợp độc lập.
Kế hoạch 30/60/90 vô nghĩa vì họ sẽ rời trước mốc 60. Cái nên làm thay: một buổi 2 giờ về đúng phạm vi
cần thiết, một người liên hệ duy nhất, và tiêu chí nghiệm thu rõ. Đầu tư vào lớp 3 ở đây là lãng phí vì
họ không cần điều hướng tổ chức.

**Khi tổ chức đang trong khủng hoảng thật sự.** Đang có incident kéo dài nhiều tuần, hoặc đang chạy nước
rút cho một deadline sống còn. Trong điều kiện đó, một kế hoạch onboarding đầy đủ là thứ sẽ được viết ra
rồi không ai thực hiện, và việc đó tệ hơn không viết — nó dạy tổ chức rằng kế hoạch onboarding là giấy
tờ. Cái nên làm thay: hoãn ngày vào làm nếu có thể; nếu không, thu gọn còn ba việc (quyền truy cập, một
buddy được giao rõ ràng và được giảm tải, một phạm vi hẹp và an toàn), và nói thật với người mới rằng đây
là giai đoạn bất thường kèm một ngày cụ thể sẽ quay lại onboarding đầy đủ.

**Khi người mới là người cũ quay lại (boomerang) sau dưới 12 tháng.** Họ đã có lớp 3, phần lớn lớp 1
và lớp 2. Chạy đủ quy trình là nghi lễ gây khó chịu. Cái nên làm thay: một tài liệu "những gì đã thay đổi kể từ
khi bạn đi" và cập nhật quyền truy cập. Nhưng có một cạm bẫy phải tránh: giả định lớp 3 còn nguyên vẹn.
Con người và quan hệ quyền lực có thể đã đổi hoàn toàn trong 12 tháng; cần một buổi nói riêng về việc ai
giờ đây quyết định gì.

**Khi người mới sẽ làm một vai chưa từng tồn tại trong tổ chức.** Ví dụ SRE đầu tiên, Data Engineer đầu
tiên. Không có buddy nào chuyển giao được lớp 1 của vai đó, không có mentor kỹ thuật, và checklist bạn
viết ra sẽ dựa trên hiểu biết mơ hồ của bạn về vai. Cái nên làm thay: đảo ngược một phần — giao cho họ
nhiệm vụ tuần đầu là **phỏng vấn 8 người trong tổ chức** và viết lại bức tranh vấn đề theo cách hiểu của
họ. Bạn onboard họ vào bối cảnh; họ onboard bạn vào chuyên môn. Việc bạn vẫn phải làm đầy đủ là lớp 3,
vì đó là phần họ không thể tự lấy được.

---

## 5. Career Ladder

### Problem Statement

Tháng 12, một product startup Việt 35 engineer. Hà phải chốt danh sách promotion cho chu kỳ cuối năm. Ba
trường hợp trên bàn:

- **Linh** — 3 năm ở công ty, làm việc ổn định, không có sự cố, được mọi người quý, chủ động nhắn Hà ba
  lần trong quý hỏi về lộ trình lên Senior.
- **Khoa** — 2 năm, là người duy nhất hiểu hệ thống đối soát, đã ngăn hai sự cố lớn bằng cách phát hiện
  vấn đề trước khi lên production, nhưng không bao giờ nói về việc mình làm, không tham gia họp nhiều.
- **Sơn** — 4 năm, thâm niên nhất trong nhóm, code chất lượng trung bình, nhưng đã "đợi lâu rồi" và
  trong buổi 1-1 gần nhất có nói bóng gió về việc đang có người liên hệ mời làm nơi khác.

Hà không có tài liệu nào để đối chiếu. Cô quyết định dựa trên cảm nhận, và cảm nhận nói: Linh thì rõ ràng
tích cực và đã chủ động hỏi; Sơn thì nếu không promote sẽ mất người; Khoa thì "để kỳ sau, năm nay chưa
nổi bật lắm".

Tháng 3 năm sau, Khoa nghỉ. Trong exit interview, câu Khoa nói là: "Em không biết em cần làm gì để lên
bậc. Em thấy người nói nhiều thì được lên."

Đây là hiện tượng chuẩn khi thiếu Career Ladder: **promotion trở thành hàm của mức độ hiển thị và mức độ
thân thiết, không phải hàm của đóng góp.** Cơ chế không phải là Hà thiên vị — Hà không có công cụ nào
khác ngoài trí nhớ, và trí nhớ ưu ái người tương tác nhiều với mình.

Hậu quả quan sát được khi thiếu ladder:

- Người đóng góp lặng lẽ rời đi; người giỏi tự quảng bá ở lại và được thăng tiến. Team dịch dần về hướng
  hiển thị thay vì hướng tác động.
- Mọi cuộc trò chuyện về lương trở thành đàm phán cá nhân, và kết quả phụ thuộc vào khả năng đàm phán chứ
  không phải bậc năng lực. Chênh lệch lương giữa hai người cùng năng lực trong cùng team có thể lên tới
  30–40% mà không ai giải thích được (số minh hoạ, nhưng mẫu hình này rất phổ biến).
- Không ai biết cần làm gì để tiến lên, nên không ai đầu tư vào việc đúng. Người ta đầu tư vào việc **có
  vẻ** dẫn tới thăng tiến, thường là việc dễ nhìn thấy.
- Manager mất rất nhiều thời gian cho mỗi cuộc trò chuyện về sự nghiệp vì phải bịa ra tiêu chí tại chỗ,
  và các tiêu chí bịa ra ở hai cuộc trò chuyện khác nhau sẽ mâu thuẫn nhau.

### First Principles

**Career Ladder giải quyết ba vấn đề khác nhau, và nhầm lẫn giữa chúng là nguồn gốc của hầu hết ladder
tồi.**

| Vấn đề | Ladder giải quyết bằng cách | Nếu thiếu |
|---|---|---|
| **1. Làm rõ kỳ vọng** | Mô tả bằng hành vi quan sát được: ở bậc này, người ta làm gì mà bậc dưới không làm | Mỗi manager có kỳ vọng riêng; người ta bị đánh giá theo tiêu chuẩn họ không biết tồn tại |
| **2. Tạo cơ sở công bằng cho lương và promotion** | Gắn dải lương vào bậc, gắn bậc vào tiêu chí; quyết định cá nhân trở thành việc xác định vị trí trên thang chung | Lương là kết quả đàm phán; người ít đàm phán bị thiệt có hệ thống; không giải thích được chênh lệch |
| **3. Định hướng phát triển** | Cho biết khoảng cách giữa vị trí hiện tại và bậc tiếp theo là gì, cụ thể | Người ta phát triển ngẫu nhiên hoặc theo hướng dễ nhìn thấy |

Ba vấn đề này có thể mâu thuẫn nhau. Ladder tối ưu cho vấn đề 2 (công bằng, chống kiện tụng) có xu hướng
trở nên rất chi tiết và pháp lý hoá. Ladder tối ưu cho vấn đề 3 (định hướng) cần ngắn gọn, dễ nhớ, tạo
cảm hứng hành động. Khi thiết kế, phải chọn vấn đề nào là chính ở tổ chức của bạn — cố tối ưu cả ba cùng
lúc cho ra một tài liệu 40 trang mà không ai đọc.

**Vì sao không dùng số năm kinh nghiệm làm trục.** Ba lý do, mỗi lý do đủ để loại:

1. *Tương quan yếu.* Kinh nghiệm là **thời gian tiếp xúc**, không phải **năng lực tích luỹ**. Một người
   làm 5 năm ở một hệ thống không đổi với cùng loại task có thể có ít năng lực hơn người làm 2 năm qua ba
   bối cảnh khác nhau với phản hồi chặt. Câu nói cũ trong nghề: "10 năm kinh nghiệm hay 1 năm kinh nghiệm
   lặp lại 10 lần" mô tả đúng cơ chế này.
2. *Nó tạo động lực sai.* Nếu bậc là hàm của thời gian, hành động tối ưu để lên bậc là **ở lại và chờ**,
   không phải là mở rộng phạm vi tác động. Bạn nhận được đúng hành vi bạn thưởng.
3. *Nó không nói gì về việc cần làm gì.* "Cần thêm 2 năm" không phải là định hướng phát triển; nó là một
   lời từ chối được diễn đạt lịch sự.

**Các trục thật của một ladder.** Cái phân biệt bậc này với bậc kia không phải là "giỏi hơn" mà là **loại
bài toán khác đi**. Bốn trục có sức phân biệt cao nhất:

| Trục | Câu hỏi phân biệt | Vì sao là trục đúng |
|---|---|---|
| **Phạm vi ảnh hưởng (Scope)** | Việc người này làm ảnh hưởng tới: task của mình / hệ thống của team / nhiều team / toàn tổ chức? | Đo được từ bên ngoài; không phụ thuộc cảm nhận; tăng theo bậc một cách tự nhiên |
| **Độ phức tạp bài toán** | Bài toán được giao đã rõ ràng, hay mơ hồ và phải tự định nghĩa? Có tiền lệ không? Có bao nhiêu ràng buộc mâu thuẫn? | Đây là thứ thật sự khó lên khi lên bậc; kỹ năng code không tăng tuyến tính, khả năng xử lý mơ hồ thì có |
| **Mức độ tự chủ** | Cần được giao việc / được giao mục tiêu / tự tìm ra vấn đề đáng làm? Ai kiểm tra kết quả? | Đo trực tiếp lượng thời gian quản lý mà người này tiêu tốn hoặc giải phóng |
| **Tác động lên người khác** | Chỉ làm việc của mình / nâng năng lực người trong team / thay đổi cách nhiều team làm việc? | Đây là trục phân biệt Senior với Staff rõ nhất, và là trục hay bị bỏ vì khó đo |

Bốn trục này không tăng đồng đều. Một người có thể ở mức Senior về scope nhưng mức Mid về tác động lên
người khác. Đó là thông tin hữu ích — nó chỉ đúng chỗ cần đầu tư — và nó chỉ hiện ra nếu ladder tách các
trục thay vì gộp thành một số.

**Vì sao cần dual track.** Nếu con đường duy nhất để tăng lương và ảnh hưởng là làm quản lý, ba hậu quả
xảy ra một cách máy móc:

1. Bạn mất một engineer giỏi và được một manager tệ. Peter Principle ở dạng chuẩn.
2. Những người thực sự muốn và hợp làm quản lý bị lẫn với những người chỉ muốn tăng lương, và bạn không
   phân biệt được ai là ai lúc phỏng vấn nội bộ.
3. Kiến thức kỹ thuật sâu nhất của tổ chức nằm trong đầu những người giờ đang họp cả ngày.

Ở bối cảnh Việt Nam, cơ chế này còn được khuếch đại bởi kỳ vọng gia đình và xã hội: "làm quản lý" là dấu
hiệu thành công dễ giải thích với bố mẹ, còn "Staff Engineer" thì không giải thích được. Đây là áp lực
thật, không nên coi thường, và nó có nghĩa là chỉ tạo ra bậc IC cao trên giấy là không đủ — bậc đó phải
có quyền lực, có sự công nhận công khai, và có dải lương thật ngang bậc quản lý tương đương.

### Mental Model

**Ladder là một hàm ánh xạ, không phải một chiếc thang.** Hình ảnh "thang" gợi ý một chiều duy nhất và
việc leo là mục tiêu. Mô hình chính xác hơn: ladder là một hàm ánh xạ từ **loại đóng góp** sang **mức bù
đắp và mức kỳ vọng**. Người ta không "leo"; họ thay đổi loại đóng góp, và ladder ghi nhận sự thay đổi đó.

Hệ quả thực hành của cách nhìn này: câu hỏi đúng trong buổi 1-1 không phải "em muốn lên bậc nào" mà "loại
bài toán nào em muốn làm trong hai năm tới". Bậc là hệ quả, không phải mục tiêu.

**Đường cong scope, và vì sao mỗi bậc khó gấp bội bậc trước.**

```
Phạm vi ảnh hưởng
  Tổ chức    |                                          ● L6 (Principal)
             |                                   
  Nhiều team |                          ● L5 (Staff)
             |                    
  Cả team    |              ● L4 (Senior)
             |         
  Vài module |      ● L3 (Mid)
  Một task   |  ● L2 (Junior)
             └────────────────────────────────────────────►
                Độ mơ hồ của bài toán được giao

Khoảng cách L2→L3: chủ yếu là kỹ năng kỹ thuật. Học được bằng làm nhiều.
Khoảng cách L3→L4: kỹ thuật + tự chủ. Học được bằng được giao việc mơ hồ hơn.
Khoảng cách L4→L5: KHÔNG phải kỹ thuật sâu hơn. Là ảnh hưởng qua người khác
                   và chọn đúng bài toán. Đây là chỗ nhiều người dừng lại vĩnh viễn,
                   vì họ tiếp tục làm giỏi hơn thứ đã đưa họ tới L4.
```

Điểm gãy L4→L5 đáng được nói riêng, vì nó là nguồn của rất nhiều hiểu lầm trong các buổi 1-1. Một Senior
xuất sắc đề nghị lên Staff và lập luận bằng độ khó kỹ thuật của những gì mình đã làm. Nhưng thứ phân biệt
L5 không phải là code khó hơn — mà là **tác động của họ vượt ra ngoài phạm vi tay họ chạm tới**. Nếu bạn
không nói rõ điều này bằng văn bản, cuộc trò chuyện sẽ luôn kết thúc bằng cảm giác bị đối xử bất công ở
cả hai phía.

### Practical Framework

**Bảng ladder mẫu rút gọn — 6 bậc × 4 trục.** Đây là bản dùng được cho một tổ chức 30–80 engineer. Với
tổ chức nhỏ hơn, xem phần "áp vào công ty nhỏ" ở dưới.

| Bậc | Phạm vi ảnh hưởng | Độ phức tạp bài toán | Mức độ tự chủ | Tác động lên người khác |
|---|---|---|---|---|
| **L2 — Engineer** | Task được giao, trong một module | Bài toán đã được định nghĩa rõ, có tiền lệ trong codebase | Cần review kỹ; hỏi khi bí; hoàn thành task trong 1–3 ngày mà không cần nhắc | Đặt câu hỏi tốt; không làm chậm người khác |
| **L3 — Engineer II** | Một vài module; hiểu cả luồng nghiệp vụ mình chạm | Bài toán rõ mục tiêu nhưng cách làm phải tự chọn; ước lượng được công việc của chính mình | Tự chia nhỏ task; tự phát hiện trường hợp biên; PR qua review với ít vòng | Review PR có nội dung; giúp người mới trong phạm vi mình biết |
| **L4 — Senior** | Một hệ thống hoặc một mảng lớn của team; hệ quả kéo dài nhiều quý | Bài toán mơ hồ, nhiều ràng buộc mâu thuẫn; phải tự làm rõ yêu cầu với PO; chọn trade-off và bảo vệ được | Tự tìm ra việc cần làm trong mảng của mình; manager không cần theo dõi chi tiết; chịu trách nhiệm vận hành cái mình xây | Mentor 1–2 người có kết quả đo được; viết ADR/tài liệu thiết kế mà người khác dùng; nâng chuẩn code review của team |
| **L5 — Staff** | Nhiều team hoặc một vấn đề xuyên suốt tổ chức (kiến trúc, hiệu năng, nền tảng) | Bài toán chưa ai định nghĩa; phải tự xác định vấn đề nào đáng làm và thuyết phục người khác rằng nó đáng làm | Tự đặt ưu tiên trong một lĩnh vực rộng; manager thảo luận chứ không giao việc | Thay đổi cách nhiều người làm việc; kết quả đạt được chủ yếu **thông qua** người khác chứ không bằng tay mình; là người được hỏi khi có quyết định kỹ thuật khó |
| **L6 — Principal** | Toàn tổ chức engineering; hệ quả nhiều năm | Bài toán ở mức chiến lược kỹ thuật: mua hay xây, nền tảng nào, đánh đổi nào chấp nhận trong 3 năm tới | Hoạt động như một đối tác của lãnh đạo; tự chịu trách nhiệm về việc chọn đúng bài toán | Định hình chuẩn kỹ thuật của tổ chức; phát triển các L5; tiếng nói có trọng lượng trong quyết định kinh doanh có yếu tố kỹ thuật |

**Dual track — bảng tương đương.**

| Bậc | Track IC | Track Manager | Phạm vi tương đương | Ghi chú |
|---|---|---|---|---|
| L4 | Senior Engineer | (chưa có) | Một hệ thống / một mảng | Chưa nên có manager ở bậc này; quản lý là bậc L5 trở lên |
| L5 | Staff Engineer | Engineering Manager | Nhiều team hoặc 1 team 5–9 người | **Dải lương phải bằng nhau.** Nếu không bằng, dual track chỉ là trang trí |
| L6 | Principal Engineer | Senior EM / Head of Eng | Tổ chức / nhiều team | |
| L7 | Distinguished / Fellow | Director / VP Eng | Nhiều tổ chức / công ty | Hầu hết công ty Việt Nam dưới 200 engineer không cần bậc này |

Ba quy tắc bắt buộc để dual track không thành trang trí:

1. **Dải lương bằng nhau ở bậc tương đương.** Không có ngoại lệ. Nếu EM L5 kiếm nhiều hơn Staff L5, mọi
   người sẽ nhận ra trong 6 tháng và track IC chết.
2. **Chuyển track được cả hai chiều, và không bị coi là thất bại.** Một EM quay về IC là chuyện bình
   thường; nếu tổ chức đọc nó là xuống cấp, không ai dám thử làm manager, và bạn chỉ có manager từ những
   người chắc chắn về điều họ chưa từng làm.
3. **Bậc IC cao phải có quyền thật.** Staff Engineer phải có quyền phủ quyết kỹ thuật ở phạm vi của họ, có
   mặt trong các cuộc họp nơi quyết định được đưa ra, và được nhắc tên công khai khi việc thành công. Nếu
   không, đó chỉ là một cái title đắt tiền.

**Cách viết mô tả bậc bằng hành vi quan sát được.**

Quy tắc kiểm tra: nếu hai manager đọc mô tả và không thể độc lập đưa ra cùng kết luận về một người cụ
thể, mô tả đó chưa đủ hành vi.

```
KIỂM TRA CHẤT LƯỢNG MỘT DÒNG MÔ TẢ BẬC

Viết một dòng, rồi hỏi ba câu:
  1. Tôi có thể chỉ ra một sự việc CỤ THỂ trong 6 tháng qua chứng minh dòng này không?
  2. Hai manager khác nhau đọc dòng này về cùng một người có kết luận giống nhau không?
  3. Dòng này có phân biệt được bậc N với bậc N-1 không, hay nó đúng với mọi bậc?

VÍ DỤ — L4, trục "Tác động lên người khác":
  ✗ "Là tấm gương cho team"                → hỏng cả 3 câu
  ✗ "Có tinh thần trách nhiệm cao"         → hỏng câu 1 và 3 (đúng với mọi bậc)
  ✗ "Mentor tốt"                           → hỏng câu 2 ("tốt" là gì?)
  ✓ "Trong 6 tháng qua đã mentor ít nhất 1 người và người đó đã tự chủ được một
     phần việc mà trước đó phải nhờ; nêu được cụ thể mình đã làm gì"
  ✓ "Đã viết ít nhất 1 tài liệu thiết kế được ít nhất 2 người khác dùng làm cơ sở
     để ra quyết định"
  ✓ "Comment review của họ thay đổi thiết kế của PR, không chỉ bắt lỗi cú pháp —
     có thể chỉ ra ví dụ"

VÍ DỤ — L5, trục "Phạm vi ảnh hưởng":
  ✗ "Có tầm nhìn kiến trúc"
  ✓ "Đã dẫn dắt một thay đổi mà ít nhất 2 team khác phải điều chỉnh cách làm việc,
     và đã tự làm phần thuyết phục/điều phối đó chứ không nhờ manager làm hộ"
  ✓ "Được các team khác chủ động mời vào các quyết định kỹ thuật ngoài phạm vi
     team của mình — đếm được số lần"
```

**Áp ladder vào công ty nhỏ (10–30 người) mà không tạo bureaucracy.**

Ở quy mô này, bảng 6 bậc × 4 trục là quá nặng. Nhưng "không có ladder" cũng không phải lựa chọn — vấn đề
công bằng lương và định hướng phát triển vẫn tồn tại, chỉ là ở dạng ngầm và tuỳ tiện. Phiên bản tối thiểu
khả thi:

```
LADDER TỐI THIỂU — cho tổ chức 10–30 engineer
Một trang. Ba bậc. Mỗi bậc một đoạn văn.

L3 — Engineer
  Được giao một bài toán có mục tiêu rõ, tự chọn cách làm, tự phát hiện trường hợp biên,
  hoàn thành mà không cần ai nhắc. Manager biết việc đang chạy tới đâu vì bạn chủ động
  nói, không phải vì hỏi.

L4 — Senior Engineer
  Sở hữu một mảng của hệ thống. Khi có vấn đề trong mảng đó, bạn là người được hỏi và
  là người quyết. Bạn làm rõ yêu cầu mơ hồ với PO thay vì chờ được làm rõ. Bạn chịu
  trách nhiệm vận hành thứ bạn xây. Có ít nhất một người trong team giỏi lên rõ rệt
  nhờ bạn.

L5 — Staff Engineer / Engineering Manager
  Tác động của bạn chủ yếu thông qua người khác. Bạn nhìn thấy vấn đề mà chưa ai nêu ra
  và làm cho tổ chức xử lý nó. Với track IC: bạn định hình cách nhiều người làm việc.
  Với track Manager: bạn chịu trách nhiệm về kết quả và sự phát triển của một nhóm.
  Hai track cùng dải lương.

QUY TẮC ÁP DỤNG Ở QUY MÔ NÀY:
- Xem lại một lần mỗi năm, không phải mỗi quý.
- Không tạo bậc phụ (L4.1, L4 Senior+). Nếu cần phân biệt tinh hơn, phân biệt bằng
  lương trong dải, không bằng title mới.
- Ba bậc là đủ cho tới khoảng 40 engineer. Thêm bậc khi có ít nhất 3 người thật sự
  đang hoạt động ở khoảng giữa hai bậc hiện có — không thêm bậc để dự phòng.
- Không viết quá một trang. Nếu cần trang thứ hai, bạn đang giải bài toán số 2
  (công bằng pháp lý) chứ không phải bài toán số 3 (định hướng), và ở quy mô này
  bài toán số 3 quan trọng hơn.
```

**Quy trình đưa ladder vào tổ chức chưa có.** Đây là phần dễ hỏng nhất, vì việc gán bậc lần đầu cho những
người đang làm là một sự kiện chính trị.

1. **Viết nháp ladder trước khi nghĩ tới ai.** Nếu bạn viết ladder trong lúc nghĩ về Sơn, bạn sẽ viết một
   ladder mà Sơn vừa vặn. Viết trừu tượng trước, kiểm tra bằng người thật sau.
2. **Kiểm tra bằng cách gán thử bậc cho toàn bộ team, một mình, không công bố.** Nếu kết quả gán thử làm
   bạn ngạc nhiên ở 1–2 chỗ, ladder đang hoạt động (nó cho thông tin mới). Nếu nó làm bạn ngạc nhiên ở
   một nửa số người, ladder đang đo sai thứ.
3. **Hiệu chỉnh với ít nhất một manager khác hoặc một người ngoài đáng tin.**
4. **Công bố ladder trước, gán bậc sau ít nhất 4 tuần.** Cho mọi người thời gian đọc, hỏi, phản biện khi
   nó còn trừu tượng. Công bố ladder và danh sách bậc cùng lúc biến mọi cuộc thảo luận về ladder thành
   cuộc thảo luận về "vì sao tôi ở bậc đó".
5. **Gán bậc lần đầu: không ai bị giảm lương, và không ai bị "xuống bậc" so với title đang có.** Nếu title
   hiện tại lạm phát so với ladder mới, giữ title, ghi bậc thật vào hệ thống nội bộ, và làm rõ lộ trình.
   Việc hạ title công khai của người đang làm là chi phí chính trị lớn hơn nhiều so với lợi ích.
6. **Chấp nhận rằng lần đầu sẽ có 10–20% trường hợp gây tranh cãi.** Chuẩn bị sẵn cách xử lý, và xử lý
   từng trường hợp riêng chứ không sửa ladder để chiều một trường hợp.

### Trade-off

**Ladder chi tiết vs ladder ngắn gọn.**

| | Ladder chi tiết (nhiều trục, nhiều tiêu chí, ví dụ cụ thể) | Ladder ngắn gọn (3–4 bậc, mỗi bậc một đoạn) |
|---|---|---|
| **Được** | Giảm thiên vị vì có tiêu chí để đối chiếu; dễ hiệu chỉnh giữa nhiều manager; bảo vệ được quyết định khi bị chất vấn | Đọc được, nhớ được, dùng được trong 1-1; rẻ để duy trì; không tạo ảo giác chính xác |
| **Mất** | Tranh cãi câu chữ ("em có làm cái này rồi, dòng 4 mục 2 đó"); cứng nhắc với trường hợp bất thường; tốn công duy trì; tạo cảm giác quan liêu ở tổ chức nhỏ | Phụ thuộc nhiều vào phán đoán của manager, nên dễ thiên vị; khó hiệu chỉnh khi có nhiều manager; khó bảo vệ quyết định |
| **Nghiêng về khi** | > 50 engineer; nhiều manager; đã từng có tranh chấp về công bằng; có yêu cầu tuân thủ | < 40 engineer; ít manager; văn hoá tin tưởng cao; đang thay đổi tổ chức nhanh |

Một quan sát thực tế đáng lưu ý: ladder chi tiết thường được viết ra để chống thiên vị, nhưng nếu văn hoá
hiệu chỉnh yếu thì nó chỉ chuyển thiên vị từ chỗ "quyết định ai lên bậc" sang chỗ "diễn giải câu chữ nào
áp cho ai". Ladder không thay thế được calibration; nó chỉ làm cho calibration khả thi.

**Ladder gắn chặt với lương vs tách rời.** Gắn chặt (mỗi bậc một dải lương công khai) cho công bằng cao
nhất và ít đàm phán nhất — nhưng làm mọi cuộc trò chuyện phát triển trở thành cuộc trò chuyện tiền bạc,
và làm tổ chức mất linh hoạt khi thị trường biến động (bạn không thể trả cao hơn dải cho một kỹ năng đang
khan hiếm). Tách rời cho linh hoạt nhưng mở lại cửa cho bất công. Nghiêng về gắn chặt khi bạn đã có vấn
đề về công bằng lương hoặc khi tổ chức đủ lớn để sự bất nhất trở nên hiển nhiên; nghiêng về tách rời một
phần (dải rộng, có chồng lấn giữa các bậc) khi bạn cần cạnh tranh cho các kỹ năng hiếm.

**Công khai bậc của mọi người vs giữ riêng tư.** Công khai (ai ở bậc nào, mọi người đều biết) tạo minh
bạch, làm cho ladder có ý nghĩa thật và tạo áp lực hiệu chỉnh lên manager. Nhưng nó cũng tạo so sánh liên
tục, làm những người ở bậc thấp hơn cảm thấy bị định vị công khai, và ở văn hoá coi trọng thể diện thì
chi phí này cao hơn nhiều so với ở phương Tây. Nghiêng về công khai title/bậc nhưng **không công khai
lương cá nhân**; công khai dải lương theo bậc. Đây là điểm cân bằng mà nhiều tổ chức Việt Nam đạt được mà
không gây tổn thất về thể diện.

### Real-world Scenarios

**Tình huống A — "Bạn em ở công ty khác đã là Tech Lead rồi".**

Bối cảnh: Duy, L3, 2.5 năm kinh nghiệm, trong buổi 1-1 với Hà.

> **Duy:** Chị ơi, em muốn hỏi về lộ trình. Bạn em học cùng khoá, làm ở [công ty khác], giờ nó là Tech
> Lead rồi, quản 4 người. Em thì vẫn Engineer. Em thấy hơi lo là mình đi chậm.

Đây là cuộc trò chuyện xảy ra hằng tuần ở các tổ chức Việt Nam, và cách xử lý nó quyết định rất nhiều
thứ.

> **Phản ứng sai thứ nhất (hạ thấp công ty kia):**
> "Em đừng so sánh. Title ở mấy công ty đó không có nghĩa gì đâu, Tech Lead của họ chỉ tương đương Mid
> bên mình thôi."
>
> Vì sao hỏng: có thể đúng về mặt sự thật, nhưng nó không trả lời câu hỏi thật của Duy (em có đang đi
> đúng hướng không), nó nghe như phòng thủ, và nó dạy Duy rằng nêu vấn đề này lên là không được chào đón.
> Duy sẽ không nêu nữa — và ba tháng sau sẽ nộp đơn.
>
> **Phản ứng sai thứ hai (hứa hẹn để xoa dịu):**
> "Chị hiểu. Em cứ yên tâm, cuối năm chị sẽ xem xét cho em lên Senior."
>
> Vì sao hỏng: tạo một cam kết mà Hà chưa kiểm tra được, dựa trên áp lực chứ không dựa trên bằng chứng.
> Nếu cuối năm không thực hiện được, Hà mất uy tín và Duy mất niềm tin vào toàn bộ hệ thống. Đây là cách
> nhanh nhất để phá hỏng một ladder vừa xây.
>
> **Phản ứng đúng:**
> "Câu hỏi thật của em là gì — em muốn cái title đó, hay em muốn biết mình có đang phát triển đúng hướng
> không? Chị hỏi thật, vì hai cái đó cần hai câu trả lời khác nhau.
>
> [Duy: cả hai, nhưng chủ yếu là cái thứ hai.]
>
> Ok. Vậy mình mở ladder ra. Chị nói thẳng vị trí của em: em đang chắc ở L3, và em đã chạm được hai
> trong bốn trục của L4 — em tự chia việc tốt, em phát hiện được trường hợp biên mà người khác bỏ sót.
> Hai trục em chưa chạm: một là em chưa sở hữu mảng nào — mỗi sprint em làm một chỗ khác nhau, và đó một
> phần là lỗi của chị vì chị đang giao việc như vậy. Hai là tác động lên người khác — em review PR nhưng
> comment chủ yếu về style.
>
> Về cái title của bạn em: chị không biết công ty đó định nghĩa Tech Lead thế nào, nên chị không so sánh
> được, và chị nghĩ em cũng không nên so. Nhưng chị nói được điều này: nếu em sang một công ty khác ngay
> bây giờ, khả năng cao em cũng lấy được một title cao hơn, vì thị trường đang lạm phát title. Câu hỏi
> là sau đó thì sao. Title không đi theo em; năng lực thì có.
>
> Đây là đề nghị của chị cho 6 tháng tới: chị giao em sở hữu module thông báo — toàn bộ, kể cả phần nói
> chuyện với PO về yêu cầu và phần trực khi nó lỗi. Và chị muốn em kèm Sơn trong việc đó. Sau 6 tháng
> mình đối chiếu lại bốn trục này. Chị không hứa promotion — chị hứa rằng nếu em làm được hai việc đó ở
> mức chị mô tả, hồ sơ của em sẽ đủ mạnh và chị sẽ là người đưa nó ra hội đồng."

Ba đặc điểm của phản ứng đúng: nó dùng một tài liệu chung thay vì ý kiến cá nhân; nó thừa nhận phần lỗi
của manager (giao việc rời rạc nên Duy không sở hữu được gì); và nó hứa **quy trình** chứ không hứa **kết
quả**.

**Tình huống B — IC giỏi bị đẩy sang quản lý.**

Bối cảnh: Quân, Senior BE xuất sắc, 4 năm ở công ty, là người mạnh nhất về kỹ thuật. Công ty đang mở
thêm một team. CTO nói với Hà: "Cho Quân lên làm Tech Lead team mới. Nó xứng đáng, mà cũng là cách duy
nhất để tăng lương cho nó — dải lương Senior kịch trần rồi."

Câu cuối là chỗ hỏng, và nó là lý do phổ biến nhất khiến IC giỏi bị đẩy sang quản lý ở các công ty Việt
Nam quy mô vừa: **không có bậc IC nào cao hơn Senior, nên trần lương của track IC thấp hơn trần lương của
track quản lý một cách cấu trúc.** Vấn đề không phải là Quân không hợp làm quản lý — có thể hợp, có thể
không. Vấn đề là quyết định đang được đưa ra bởi ràng buộc của bảng lương, không bởi câu hỏi "việc gì
đúng cho Quân và cho tổ chức".

Ba góc nhìn:

| | Nhìn từ IC (Quân) | Nhìn từ Tech Lead (Tuấn — người sẽ mất Quân khỏi vai kỹ thuật) | Nhìn từ Manager (Hà) |
|---|---|---|---|
| **Điều nhìn thấy** | "Đây là cơ hội duy nhất để lương tăng đáng kể. Với lại bố mẹ hỏi mãi bao giờ lên quản lý." | "Quân là người duy nhất xử lý được các vấn đề khó nhất. Mất Quân khỏi code là mất năng lực kỹ thuật của cả tổ chức." | "Nếu không cho Quân lên, Quân sẽ nghỉ trong 6 tháng. Nếu cho lên và Quân không hợp, mình mất một engineer giỏi và có một manager tệ." |
| **Điều không nhìn thấy** | Rằng công việc quản lý khác hoàn toàn công việc hiện tại, và điều làm Quân giỏi hiện nay không dùng được nhiều; rằng nếu không hợp thì quay lại rất khó về mặt thể diện | Rằng giữ Quân ở vai IC mà không có đường tăng thu nhập là giữ bằng cách bỏ mặc | Rằng bài toán thật không phải là "Quân có nên làm quản lý không" mà là "vì sao track IC của tổ chức này có trần" |
| **Hành động đúng** | Hỏi rõ: công việc hằng ngày sẽ là gì; nếu 6 tháng thấy không hợp thì quay lại thế nào; và nói thẳng về động cơ lương thay vì giấu | Nói rõ với Hà chi phí kỹ thuật của việc mất Quân, bằng cụ thể (những việc nào sẽ không ai làm được), không bằng cảm tính | Giải bài toán cấu trúc: đề xuất tạo bậc Staff Engineer với dải lương ngang EM. Đây là việc mất 2 tháng nhưng giải quyết vấn đề cho cả những người sau Quân |

Cách xử lý của Hà, thực tế và có thể làm được:

1. Tách hai quyết định: quyết định về lương của Quân (giải ngay, bằng cách xin ngoại lệ dải lương hoặc
   thưởng giữ chân) và quyết định về vai (giải chậm, đúng cách).
2. Nói thẳng với Quân rằng vai quản lý là một **thay đổi nghề**, không phải một phần thưởng, và mô tả cụ
   thể một tuần của EM trông như thế nào.
3. Nếu Quân vẫn muốn thử: thiết kế đường quay lại **trước khi bắt đầu**, bằng văn bản, và nói công khai
   với team rằng đây là thử nghiệm 6 tháng. Đường quay lại chỉ có tác dụng nếu nó được công bố trước —
   công bố sau khi thất bại thì nó là sự sa thải được che đậy.
4. Song song: đẩy việc tạo bậc Staff. Đây mới là hành động thật sự giải quyết vấn đề.

### Best Practices

- **Viết ladder bằng hành vi quan sát được, kiểm tra bằng ba câu ở trên.** Lý do: mô tả không kiểm chứng
  được sẽ được diễn giải theo hướng có lợi cho người mà manager quý mến — tức là ladder trở thành vỏ bọc
  cho thiên vị thay vì biện pháp chống nó.
- **Tách trục, không gộp thành một điểm.** Lý do: người ta phát triển không đồng đều; bức tranh tách trục
  chỉ ra chính xác nơi cần đầu tư, còn một điểm tổng thì không nói gì.
- **Dải lương bằng nhau giữa hai track ở bậc tương đương, và công khai điều đó.** Lý do: đây là phép thử
  duy nhất cho biết dual track có thật hay chỉ là trang trí; mọi người sẽ tự kiểm tra dù bạn có công khai
  hay không.
- **Công bố ladder trước, gán bậc sau ít nhất 4 tuần.** Lý do: tách cuộc thảo luận về nguyên tắc khỏi
  cuộc thảo luận về cá nhân; nếu gộp, mọi phản biện về nguyên tắc đều bị đọc là bất mãn cá nhân.
- **Xem lại ladder mỗi 12–18 tháng, không thường xuyên hơn.** Lý do: ladder chỉ có giá trị nếu nó ổn
  định; một tiêu chuẩn thay đổi mỗi quý không phải là tiêu chuẩn.
- **Dùng ladder trong 1-1 định kỳ, không chỉ trong mùa promotion.** Lý do: nếu tài liệu chỉ xuất hiện khi
  có tranh chấp, nó được đọc là công cụ từ chối. Dùng đều đặn, nó trở thành ngôn ngữ chung.
- **Bắt đầu với 3 bậc ở tổ chức nhỏ và thêm bậc chỉ khi có người thật đang ở giữa.** Lý do: bậc dự phòng
  tạo kỳ vọng về những nấc thang không tồn tại và làm chậm mọi người.

### Anti-patterns

**"Ladder theo số năm."** "L4 = 5 năm kinh nghiệm". Cơ chế phá hoại: thưởng cho việc ở lại thay vì việc
mở rộng tác động; và tạo ra một lời hứa ngầm mà tổ chức không định giữ (người đủ 5 năm sẽ yêu cầu bậc,
và bạn sẽ phải hoặc trao hoặc phá vỡ lời hứa). Dấu hiệu sớm: trong mô tả bậc có bất kỳ con số năm nào.

**"Tạo title mới để giữ người."** Sơn doạ nghỉ, được đổi thành "Senior Technical Specialist" mà phạm vi
công việc không đổi. Cơ chế phá hoại: (a) mọi người khác nhìn thấy và học được rằng cách lên bậc là doạ
nghỉ, nên bạn vừa dạy cả tổ chức một chiến thuật đàm phán; (b) ladder mất ý nghĩa vì có một người ở bậc
đó mà không làm việc của bậc đó, và mọi người dùng người đó làm chuẩn tham chiếu; (c) chính Sơn cũng
không được lợi — họ giữ một title mà đồng nghiệp biết là rỗng. Dấu hiệu sớm: xuất hiện các title không có
trong ladder; hoặc số bậc/title trong công ty tăng nhanh hơn số người.

**"Ladder viết xong rồi để đó."** Có file trên Confluence, không ai mở trong 11 tháng, đến mùa review thì
mở ra tìm câu chữ hợp lý hoá quyết định đã có. Cơ chế: ladder trở thành công cụ biện minh hậu kiểm thay
vì công cụ định hướng. Dấu hiệu sớm: hỏi 5 engineer bậc hiện tại của họ và tiêu chí của bậc kế tiếp — nếu
3 người không trả lời được, ladder không tồn tại về mặt chức năng.

**"Sao chép ladder của một công ty lớn."** Lấy ladder của một big tech về dùng nguyên. Cơ chế: ladder đó
mô tả các bậc trong một tổ chức hàng nghìn người với các vấn đề mà tổ chức 30 người không có; nó tạo ra
những bậc không ai đạt được và những kỳ vọng không liên quan. Dấu hiệu sớm: ladder có bậc mà chưa ai
trong công ty từng đạt và cũng không có lộ trình khả dĩ để đạt trong 3 năm.

**"Một trục duy nhất: độ khó kỹ thuật."** Bậc được xác định bởi việc ai giải được bài khó nhất. Cơ chế:
bỏ hoàn toàn trục tác động lên người khác, nên tổ chức đầy những người rất giỏi làm việc một mình và
không ai nâng ai lên; đồng thời phạt những người dành thời gian mentor vì thời gian đó không hiện ra
trong độ khó kỹ thuật của output cá nhân. Dấu hiệu sớm: không ai trong team tự nguyện nhận việc mentor
hoặc onboarding.

### Khi nào KHÔNG nên áp dụng

**Khi tổ chức dưới 10 engineer và mọi người làm mọi thứ.** Ở quy mô này, phân biệt bậc là phân biệt trên
một mẫu quá nhỏ, và tài liệu ladder tạo ra sự trang trọng không tương xứng với thực tế — nó khiến tổ chức
trông như đang giả vờ là công ty lớn. Cái nên làm thay: một cuộc trò chuyện thẳng thắn với từng người mỗi
6 tháng về việc họ đang mạnh gì, thiếu gì, và một cam kết rõ ràng về nguyên tắc trả lương (ví dụ: "chúng
ta trả theo mức thị trường cho năng lực, không theo đàm phán"). Viết ladder khi bạn có manager thứ hai —
vì lúc đó bạn mới có vấn đề về sự nhất quán giữa các người đánh giá.

**Khi tổ chức đang tái cấu trúc lớn.** Đang sáp nhập, đang cắt giảm, đang đổi mô hình kinh doanh. Ladder
công bố trong giai đoạn này sẽ bị đọc qua lăng kính "ai sẽ bị cắt", bất kể ý định của bạn, và mọi thảo
luận về nó sẽ nhiễm nỗi lo đó. Cái nên làm thay: hoãn tới khi cấu trúc ổn định; trong lúc chờ, xử lý các
vấn đề công bằng lương cấp bách một cách riêng lẻ và kín đáo.

**Khi bạn không có quyền quyết định lương.** Ở nhiều ODC và nhiều bộ phận IT của doanh nghiệp truyền
thống, thang lương do HR tập đoàn quyết theo hệ thống ngạch bậc chung cho toàn công ty, và bạn không thể
gắn ladder kỹ thuật vào lương. Công bố một ladder mà không có hệ quả về lương tạo ra kỳ vọng bạn không
thể đáp ứng, và sự thất vọng sau đó tệ hơn việc không có ladder. Cái nên làm thay: dùng ladder **chỉ cho
mục đích 1 và 3** (làm rõ kỳ vọng, định hướng phát triển), nói thẳng ngay từ đầu rằng nó không gắn với
lương và giải thích vì sao, và song song vận động ở tầng trên để nối hai hệ thống. Điều không được làm là
im lặng để mọi người tự suy diễn rằng lên bậc sẽ có tiền.

**Khi ladder được dùng để thay thế cho việc quản lý.** Nếu bạn đang hy vọng rằng có ladder thì không cần
các cuộc trò chuyện khó về hiệu suất và định hướng nữa, ladder sẽ làm bạn thất vọng và làm nhân viên của
bạn tức giận. Nó là ngôn ngữ chung cho các cuộc trò chuyện đó, không phải vật thay thế chúng. Dấu hiệu
bạn đang rơi vào đây: bạn thấy nhẹ nhõm khi có thể trả lời một câu hỏi về sự nghiệp bằng cách gửi link
tài liệu.

---

## 6. Performance Review và Feedback định kỳ

### Problem Statement

Ngày 18 tháng 12, Hà ngồi viết 9 bản review, hạn nộp là ngày 20. Cô mở lịch, cuộn ngược lại 12 tháng, và
phát hiện mình không nhớ được gì về quý I. Những gì cô nhớ rõ: Linh đã làm rất tốt vụ migration hồi tháng
11; Sơn đã để lọt một bug lên production cách đây ba tuần; Vy thì... Vy đã làm gì trong năm nay nhỉ?

Bản review cuối cùng của Sơn có một câu: "Cần cẩn thận hơn trong việc kiểm thử trước khi release." Sơn
đọc, và trong đầu Sơn hiện lên một sự việc duy nhất — cái bug tháng 12 — mà không hiện lên việc anh đã
giữ hệ thống đối soát chạy suốt 12 tháng không sự cố. Sơn không phản đối trong buổi review (đó không phải
điều người ta làm), nhưng anh về nhà và mở LinkedIn.

Ba hiện tượng quan sát được, xuất hiện gần như luôn luôn ở tổ chức chỉ review một năm một lần:

1. **Nội dung review là ba tháng gần nhất được kể như thể là cả năm.** Đây không phải sự lười biếng, đây
   là recency bias — trí nhớ con người không lưu trữ đồng đều theo thời gian.
2. **Không có gì trong review là mới với người nghe, hoặc mọi thứ đều mới.** Cả hai đều là dấu hiệu hỏng.
   Trường hợp thứ hai — nhân viên nghe một lời phê bình lần đầu tiên trong buổi review chính thức — là
   dạng thất bại nghiêm trọng của quản lý: một vấn đề đã tồn tại nhiều tháng mà không ai nói.
3. **Người nhận review dành toàn bộ năng lượng cho việc bảo vệ mình, không cho việc học.** Vì trong cùng
   một buổi, họ vừa nghe đánh giá phát triển vừa nghe kết quả về lương.

Chi phí thật: một chu kỳ review 9 người tiêu khoảng 40–60 giờ của manager (số minh hoạ, bao gồm viết,
calibration, họp). Nếu đầu ra là một tài liệu mà người nhận không tin và không dùng để thay đổi hành vi,
đó là 40–60 giờ tạo ra giá trị âm — vì nó còn làm giảm niềm tin vào toàn bộ hệ thống.

### First Principles

**Review định kỳ tồn tại vì hai nhu cầu khác nhau, và trộn chúng làm hỏng cả hai.**

| | Review phát triển (developmental) | Review phân bổ (evaluative) |
|---|---|---|
| **Phục vụ ai** | Cá nhân | Tổ chức |
| **Mục đích** | Cung cấp tín hiệu hiệu chỉnh để người đó giỏi lên | Phân bổ nguồn lực hữu hạn: lương, thưởng, promotion, và trong trường hợp xấu là cắt giảm |
| **Trạng thái tâm lý cần có ở người nghe** | Cởi mở, sẵn sàng nghe điều khó chịu, tò mò | Phòng thủ (hợp lý — đây là quyết định về thu nhập của họ) |
| **Tần suất tối ưu** | Liên tục, càng gần sự việc càng tốt | Định kỳ, đồng bộ toàn tổ chức |
| **Hình thức tối ưu** | Trò chuyện hai chiều, không ghi điểm | Văn bản, có bằng chứng, kiểm toán được |

Xung đột nằm ở dòng thứ ba. Khi một người biết rằng cuộc trò chuyện này sẽ quyết định lương của họ, hệ
thần kinh của họ chuyển sang chế độ phòng thủ. Ở trạng thái đó, khả năng tiếp thu phản hồi mang tính xây
dựng giảm mạnh — họ nghe mọi nhận xét như một cáo buộc cần bác bỏ. Đây không phải là vấn đề về thái độ;
đó là phản ứng hợp lý với một tình huống có rủi ro tài chính.

Hệ quả thiết kế: **tách buổi nói về phát triển khỏi buổi nói về lương, cách nhau ít nhất 2–4 tuần.** Việc
này gần như không tốn thêm gì và tạo ra khác biệt lớn. Ở buổi phát triển, không nhắc tới tiền. Ở buổi
lương, không cố dạy dỗ — chỉ thông báo kết quả, giải thích cơ sở, và trả lời câu hỏi.

**Vì sao review một năm một lần thất bại — ba cơ chế độc lập.**

1. *Feedback trễ mất giá trị hiệu chỉnh.* Đây là nguyên lý điều khiển học cơ bản: độ trễ trong vòng phản
   hồi làm giảm khả năng ổn định hệ thống. Nói với ai đó vào tháng 12 rằng cách họ viết tài liệu thiết kế
   hồi tháng 3 chưa tốt — họ không thể sửa tháng 3, họ đã viết thêm 20 tài liệu theo cách cũ, và giá trị
   của thông tin đã bốc hơi gần hết. Feedback có giá trị cao nhất trong khoảng 24–72 giờ sau sự việc.
2. *Recency bias.* Trí nhớ con người không phải là bản ghi tuyến tính. Ba tháng gần nhất chiếm khoảng
   70–80% nội dung của một bản review dựa trên trí nhớ (quan sát định tính phổ biến, không phải số đo).
   Hệ quả bất công cụ thể: người có một thành công lớn vào tháng 11 được đánh giá cao hơn người có bốn
   thành công vừa vào tháng 2 đến tháng 8, dù tổng đóng góp ngược lại.
3. *Không ai nhớ, kể cả người được review.* Khi nhân viên không nhớ chi tiết việc mình làm 10 tháng
   trước, họ không thể phản biện một đánh giá sai. Bất đối xứng thông tin này biến review thành một sự
   kiện một chiều.

**Vì sao forced distribution (phân phối bắt buộc) gây hại, và cơ chế cụ thể.**

Nhiều tổ chức áp một phân phối cứng lên kết quả review: ví dụ tối đa 20% "vượt kỳ vọng", tối thiểu 10%
"dưới kỳ vọng", cho mọi team bất kể quy mô. Lập luận ủng hộ: chống lạm phát điểm, buộc manager phân biệt.
Lập luận này không hoàn toàn sai — lạm phát điểm là vấn đề thật. Nhưng cơ chế gây hại thì rõ ràng:

- *Sai giả định thống kê.* Phân phối chuẩn xuất hiện khi bạn lấy mẫu ngẫu nhiên từ một quần thể lớn. Một
  team 6 người **đã qua sàng lọc tuyển dụng** không phải là mẫu ngẫu nhiên. Nếu bạn tuyển tốt, phân phối
  thật lệch phải. Ép nó thành hình chuông là bịa ra dữ liệu.
- *Phá hợp tác trong nội bộ team.* Nếu số suất "vượt kỳ vọng" là hữu hạn trong team, thì giúp đồng nghiệp
  giỏi lên làm giảm cơ hội của chính mình. Bạn vừa tạo ra một trò chơi tổng bằng không giữa những người
  bạn muốn hợp tác. Hành vi quan sát được: người ta ngừng chia sẻ, ngừng nhận việc chung, và tranh nhau
  các task có độ hiển thị cao.
- *Bất công theo team.* Một người bình thường trong team yếu được xếp "vượt kỳ vọng"; một người giỏi trong
  team mạnh bị xếp "đạt". Xếp hạng trở thành hàm của việc bạn ngồi cạnh ai.
- *Nó phạt manager tuyển tốt.* Nếu bạn xây được một team mà mọi người đều mạnh, bạn vẫn phải xếp 10% dưới
  kỳ vọng. Bạn học được rằng nên giữ vài người yếu trong team làm "đệm". Đây là hành vi có thật và nó
  hoàn toàn hợp lý dưới hệ khuyến khích đó.

Nếu công ty bạn áp forced distribution và bạn không có quyền bỏ nó, ba việc giảm thiểu được tác hại:

1. Đàm phán để phân phối áp ở cấp **tổ chức lớn** (50+ người) thay vì cấp team (5–9 người). Ở mẫu lớn,
   giả định thống kê ít sai hơn nhiều.
2. Tách hoàn toàn việc **xếp hạng** khỏi việc **phản hồi phát triển**, và nói thẳng với team rằng đây là
   ràng buộc từ hệ thống, không phải đánh giá của bạn về giá trị của họ. Sự trung thực này tốn ít và giữ
   được nhiều.
3. Ghi chép lại các trường hợp bị ép xếp sai và dùng làm bằng chứng để vận động thay đổi chính sách. Một
   trường hợp cụ thể có sức thuyết phục hơn mười lập luận về lý thuyết.

### Mental Model

**Feedback là một vòng điều khiển (control loop), và tần suất lấy mẫu quyết định chất lượng điều khiển.**

```
        Hành vi mong muốn
              │
              ▼
    ┌──► [Hành động] ──► [Kết quả] ──► [Quan sát] ──┐
    │                                                │
    └──────────── [Tín hiệu hiệu chỉnh] ◄───────────┘
                        ↑
                  ĐỘ TRỄ ở đây quyết định tất cả

Độ trễ 1–3 ngày:  người ta còn nhớ ngữ cảnh → sửa được hành vi cụ thể
Độ trễ 1 tháng:   nhớ mơ hồ → sửa được xu hướng, không sửa được hành vi
Độ trễ 12 tháng:  không nhớ → không sửa được gì, chỉ tạo cảm xúc
```

Suy ra một kết luận thực hành mạnh: **nếu bạn chỉ có thể đầu tư vào một thứ, đầu tư vào giảm độ trễ, chứ
không phải vào chất lượng biểu mẫu review.** Một câu nói ngay sau buổi họp có giá trị hơn ba trang viết
sau sáu tháng.

**Ba loại tín hiệu, ba cách xử lý khác nhau.** Nhầm lẫn giữa chúng là nguồn của phần lớn feedback tồi:

| Loại | Ví dụ | Nên xử lý thế nào | Đưa vào review chính thức? |
|---|---|---|---|
| **Nhiễu** | Một lần đến muộn; một PR viết vội vì đang gấp | Bỏ qua. Phản ứng với nhiễu làm người ta cảm thấy bị soi | Không |
| **Sự việc đơn lẻ có hệ quả** | Một incident do bỏ qua bước kiểm tra | Nói ngay, trong 24–72 giờ, tập trung vào cơ chế chứ không vào người | Chỉ khi hệ quả lớn hoặc là một phần của mẫu |
| **Mẫu hình lặp lại** | Ba quý liên tiếp ước lượng thấp hơn thực tế 2 lần | Đây là nội dung chính của review; cần bằng chứng nhiều điểm | Có — và phải đã được nói ít nhất 2 lần trước đó |

Quy tắc rút ra: **review chính thức chỉ nên chứa mẫu hình, không chứa sự việc đơn lẻ.** Nếu bạn đang định
đưa một sự việc đơn lẻ vào review, hãy hỏi: nó có lặp lại không? Nếu không, nó thuộc về cuộc trò chuyện
tuần đó, không thuộc về đây.

### Practical Framework

**Nhịp ba tầng.**

| Tầng | Tần suất | Thời lượng | Nội dung | Ai chuẩn bị gì |
|---|---|---|---|---|
| **Feedback liên tục** | Trong 24–72 giờ sau sự việc | 2–10 phút | Một sự việc cụ thể, một quan sát, một tác động | Không chuẩn bị; nói ngay |
| **Check-in hàng quý** | 1 lần/quý | 45–60 phút | Xu hướng của quý, đối chiếu mục tiêu, điều chỉnh hướng, chuẩn bị cho review chính thức | Cả hai bên chuẩn bị; nhân viên mang brag document |
| **Review chính thức** | 1–2 lần/năm | 60 phút + văn bản | Tổng hợp bằng chứng cả kỳ, xác định bậc/mức, cơ sở cho lương | Manager viết; đã calibration; **không có gì bất ngờ** |

Nguyên tắc nối ba tầng: **mọi thứ xuất hiện ở tầng 3 phải đã xuất hiện ít nhất hai lần ở tầng 1 hoặc 2.**
Đây là quy tắc kiểm tra đơn giản và mạnh nhất trong toàn bộ chủ đề này. Nếu bạn viết một câu trong review
mà người nhận sẽ nghe lần đầu, câu đó là bằng chứng bạn đã không làm việc của mình trong 6 tháng qua.

**Thu thập bằng chứng liên tục — hai công cụ.**

*Brag document (do nhân viên giữ).* Một file, mỗi tháng một mục, mỗi mục 2–5 dòng: đã làm gì, tác động
gì, ai xác nhận được. Đây là thực hành được nói tới rộng rãi trong cộng đồng engineering (Julia Evans có
một bài viết công khai phổ biến về chủ đề này). Lợi ích thật của nó không phải là "tự quảng bá" mà là
**bù đắp cho sự bất đối xứng trí nhớ**: manager không thể nhớ hết, và người duy nhất có động lực và
thông tin để ghi lại là chính người làm.

Ở bối cảnh Việt Nam, có một rào cản văn hoá đáng nói: việc tự kể công bị đọc là khoe khoang, và nhiều
engineer giỏi sẽ không làm dù được khuyến khích. Cách vượt qua có tác dụng: **đóng khung nó như một nghĩa
vụ đối với manager chứ không phải một hành động tự quảng bá.** "Anh cần cái này để viết review cho em cho
đúng. Nếu em không ghi, anh sẽ viết dựa trên trí nhớ của anh, và trí nhớ của anh thiên vị những việc anh
nhìn thấy." Cách nói này chuyển hành động từ "khoe" sang "giúp sếp làm đúng việc", và tỉ lệ tuân thủ tăng
rõ rệt.

*Feedback log (do manager giữ).* Một file riêng cho mỗi người, mỗi lần có quan sát đáng nhớ thì ghi 1–2
dòng kèm ngày. Không đánh giá, chỉ ghi sự việc. Mất khoảng 3 phút/tuần/người. Đây là khoản đầu tư có tỉ
suất hoàn vốn cao nhất trong toàn bộ công việc quản lý con người, và gần như không ai làm.

```
FEEDBACK LOG — Vy (L3 → đang hướng L4)
Quy tắc: ghi SỰ VIỆC, không ghi kết luận. Ghi trong tuần xảy ra. Không quá 2 dòng.

2026-02-14  Trong buổi thiết kế notification, chủ động chỉ ra trường hợp user đổi
            số điện thoại giữa chừng — không ai nghĩ tới. Tiết kiệm 1 vòng làm lại.
2026-02-27  PR #1841: comment review cho Duy có nội dung về thiết kế (đề xuất tách
            interface), không chỉ style. Duy làm theo. ← tín hiệu trục "tác động lên người khác"
2026-03-09  Ước lượng task API export: nói 2 ngày, mất 5 ngày. Nguyên nhân: không tính
            phần migration dữ liệu cũ. Lần thứ 2 trong quý (lần 1: 2026-01-22).
            ← đang thành MẪU, cần nói
2026-03-11  Đã nói với Vy về mẫu ước lượng. Vy tự đề xuất: từ giờ sẽ viết ra danh sách
            các phần việc trước khi đưa số. Theo dõi tới cuối quý.
2026-04-30  Ước lượng 3 task gần nhất đều trong sai số 20%. Mẫu đã sửa. ← ghi nhận trong check-in
```

Chú ý mẫu hình trong log này: một vấn đề được phát hiện (09/03), được nói ngay (11/03), có cam kết cụ thể
từ chính nhân viên, và được xác nhận đã sửa (30/04). Đến kỳ review, Hà không viết "Vy cần cải thiện kỹ
năng ước lượng" — cô viết "Vy đã tự nhận diện và sửa được một điểm yếu trong ước lượng trong vòng một
quý", một câu hoàn toàn khác về ý nghĩa và hoàn toàn có bằng chứng.

**Calibration giữa các manager.** Không có bước này, mọi ladder và mọi rubric đều vô nghĩa, vì chuẩn của
mỗi manager khác nhau 1–2 bậc.

```
QUY TRÌNH CALIBRATION — 2 giờ, 4–8 manager, làm TRƯỚC khi thông báo cho nhân viên

CHUẨN BỊ (mỗi manager, trước buổi):
  [ ] Nộp mức đề xuất cho từng người kèm 3 bằng chứng chính
  [ ] Nộp một câu: "người tôi tự tin nhất về mức đề xuất" và "người tôi ít tự tin nhất"

BUỔI CALIBRATION:
  1. (20') Neo chuẩn: chọn 2 người ở mức giữa từ hai team khác nhau, cả phòng cùng
     đọc bằng chứng và thống nhất "đây là mức 'đạt' trông như thế nào". Không có bước
     này thì mọi so sánh sau đó không có gốc.
  2. (60') Đi qua từng người ở các mức cao nhất và thấp nhất trước (đây là chỗ sai
     số lớn nhất và hệ quả lớn nhất). Mức giữa đi nhanh.
     Câu hỏi chuẩn cho mỗi trường hợp: "Nếu người này ở team của bạn, bạn có xếp
     cùng mức không? Vì sao không?"
  3. (20') Kiểm tra chéo các dạng lệch có hệ thống:
     - Có manager nào xếp cao/thấp hơn hẳn mặt bằng không? (lệch chuẩn cá nhân)
     - Người làm việc trên hạ tầng/nền tảng có bị xếp thấp hơn người làm feature
       không? (bias hiển thị — rất phổ biến)
     - Người mới vào 6 tháng có bị xếp thấp một cách máy móc không?
     - Có mẫu lệch nào theo giới, tuổi, hoặc theo việc ai hay phát biểu trong họp không?
  4. (20') Chốt. Manager nào phải đổi mức của người mình quản lý thì phải hiểu rõ lý do,
     vì chính họ là người sẽ giải thích với nhân viên.

SAU BUỔI:
  [ ] Ghi lại các quyết định đã đổi và lý do (dùng cho chu kỳ sau)
  [ ] KHÔNG được nói với nhân viên rằng "anh đề xuất mức cao hơn nhưng bị calibration
      hạ xuống". Đây là hành vi phá hoại: nó chuyển trách nhiệm sang một hệ thống vô
      danh, làm nhân viên mất niềm tin vào cả hệ thống lẫn manager, và về bản chất là
      hèn. Sau calibration, mức đó LÀ mức của bạn và bạn bảo vệ nó.
```

**Template review một trang.**

```
PERFORMANCE REVIEW — [Tên] — Kỳ: H1/2026 — Bậc hiện tại: L__ — Manager: ___

════ 1. TÓM TẮT (3 dòng, viết sau cùng) ════
Mức tổng thể: [Vượt kỳ vọng / Đạt kỳ vọng / Dưới kỳ vọng]
Một câu về đóng góp nổi bật nhất:
Một câu về điều cần khác đi trong 6 tháng tới:

════ 2. ĐÓNG GÓP CHÍNH TRONG KỲ (3–5 mục, mỗi mục theo mẫu) ════
[Việc gì] → [Tác động đo được hoặc quan sát được] → [Vai trò cụ thể của người này]

VD ✓: "Dẫn dắt migration hệ thống thông báo sang queue mới → giảm thời gian gửi
      thông báo từ trung bình 40s xuống 3s, loại bỏ 2 loại incident tái diễn →
      tự thiết kế, tự điều phối với team mobile, tự chạy rollout 3 giai đoạn"
VD ✗: "Đã hoàn thành tốt các task được giao trong sprint"  (không có tác động,
      không có vai trò, không phân biệt được với bất kỳ ai)

════ 3. ĐỐI CHIẾU VỚI BẬC (dùng 4 trục của ladder) ════
| Trục | Mức quan sát được | Bằng chứng | So với bậc hiện tại |
| Phạm vi ảnh hưởng      |  |  | Đạt / Vượt / Chưa đạt |
| Độ phức tạp bài toán   |  |  | |
| Mức độ tự chủ          |  |  | |
| Tác động lên người khác|  |  | |

════ 4. ĐIỀU CẦN KHÁC ĐI (tối đa 2 mục — nhiều hơn 2 thì không ai sửa được) ════
Mục 1:
  - Mẫu quan sát được (≥2 sự việc, có ngày):
  - Tác động của mẫu này lên team/hệ thống:
  - Đã trao đổi lần đầu vào ngày: ___  ← nếu ô này trống, KHÔNG được đưa vào review
  - Việc cụ thể cần khác đi:
  - Cách biết là đã khác: (tiêu chí kiểm được, có ngày)
  - Manager sẽ hỗ trợ gì:

════ 5. HƯỚNG 6 THÁNG TỚI ════
Hai đến ba mục tiêu, mỗi mục tiêu nối được với một trục của ladder.

════ 6. TỰ ĐÁNH GIÁ CỦA NHÂN VIÊN (điền TRƯỚC khi đọc phần trên) ════
[Viết độc lập — cùng lý do với debrief ở chủ đề 3: tránh cascade]

════ KIỂM TRA TRƯỚC KHI GỬI ════
[ ] Mọi nhận xét đều có ≥1 bằng chứng cụ thể có ngày tháng
[ ] Không có nhận xét nào về tính cách ("hơi khép kín", "thiếu nhiệt tình") —
    chỉ có nhận xét về hành vi và tác động
[ ] Không có gì trong mục 4 là mới với người nhận
[ ] Bằng chứng trải đều cả kỳ, không dồn vào 2 tháng cuối
[ ] Đã qua calibration
```

Dòng "không có nhận xét nào về tính cách" đáng nhấn mạnh. "Em hơi thiếu chủ động" là nhận xét về con
người, không phản bác được, không hành động được, và gây tổn thương. "Trong 3 buổi thiết kế gần nhất, em
không nêu ý kiến nào cho tới khi được hỏi trực tiếp; kết quả là hai vấn đề em đã thấy trước lại xuất hiện
ở giai đoạn sau" là mô tả hành vi và tác động — nó thảo luận được, phản bác được, và sửa được.

### Trade-off

**Review thường xuyên vs review thưa.** Thường xuyên (mỗi quý) cho độ trễ thấp, ít bất ngờ, dễ điều chỉnh
— nhưng tiêu nhiều thời gian manager (một chu kỳ 9 người × 4 lần/năm là công việc đáng kể) và có nguy cơ
làm mọi người sống trong trạng thái bị đánh giá liên tục, thứ làm giảm khả năng chấp nhận rủi ro. Thưa
(một lần/năm) rẻ hơn nhưng mắc mọi bệnh đã nêu ở Problem Statement. Điểm cân bằng thực tế cho hầu hết tổ
chức: **check-in nhẹ hàng quý (không điểm, không văn bản chính thức) + review chính thức 2 lần/năm.**
Phần chi phí nằm ở review chính thức; phần giá trị hiệu chỉnh nằm ở check-in.

**Đánh giá cá nhân vs đánh giá theo team.** Đánh giá cá nhân cho tín hiệu rõ, phân biệt được đóng góp,
công bằng với người xuất sắc — nhưng khuyến khích tối ưu cục bộ, làm người ta tránh việc chung không có
tên mình, và bỏ sót phần lớn công việc thực sự tạo giá trị (thứ hầu như luôn là kết quả tập thể). Đánh
giá theo team khuyến khích hợp tác nhưng che giấu cả người xuất sắc lẫn người không đóng góp, và người
xuất sắc sẽ rời đi. Điều kiện nghiêng: giữ đánh giá cá nhân, nhưng **đưa "tác động lên người khác" thành
một trục có trọng số thật** (như trong ladder ở chủ đề 5). Đây là cách giải quyết vấn đề mà không phải
chọn một trong hai cực.

**Minh bạch về mức của mọi người vs riêng tư.** Minh bạch buộc manager phải nhất quán và cho phép nhân
viên tự hiệu chỉnh kỳ vọng — nhưng ở văn hoá coi trọng thể diện, việc công khai ai bị xếp "dưới kỳ vọng"
là tổn thương lớn và có thể phá hỏng quan hệ trong team lâu dài. Nghiêng về: công khai **phân bố tổng**
(bao nhiêu phần trăm ở mỗi mức) và **tiêu chí**, giữ riêng tư **mức của từng cá nhân**. Bạn được phần lớn
lợi ích của minh bạch mà không trả phần lớn chi phí.

### Real-world Scenarios

**Tình huống A — Nói với một người rằng họ "đạt kỳ vọng" trong khi họ nghĩ mình xuất sắc.**

Bối cảnh: Linh, L4, kỳ H1. Linh làm việc rất chăm, luôn nhận thêm việc, hay ở lại muộn, và trong tự đánh
giá đã tự xếp "Vượt kỳ vọng". Hà xếp "Đạt kỳ vọng" — vì đối chiếu bốn trục, Linh làm rất nhiều nhưng phạm
vi và độ phức tạp không vượt bậc, và trục tác động lên người khác không có bằng chứng.

Đây là cuộc trò chuyện khó nhất trong toàn bộ chu kỳ review, khó hơn nhiều so với nói chuyện với người
dưới kỳ vọng — vì "đạt kỳ vọng" là kết quả tốt, nhưng nó được nghe như một lời từ chối.

> **Phiên bản nói sai (vòng vo, để giữ hoà khí):**
> "Chị đánh giá em năm nay rất tốt. Em làm việc chăm chỉ, ai cũng thấy. Chỉ là năm nay hệ thống của mình
> hơi chặt, mà cũng có nhiều bạn khác cũng làm tốt, nên mức của em là 'đạt kỳ vọng'. Nhưng em đừng buồn,
> mức đó là mức tốt mà. Năm sau chị sẽ cố gắng."
>
> Hỏng ở bốn chỗ. Một: mở đầu bằng lời khen chung chung làm cho kết luận sau đó nghe như mâu thuẫn, và
> Linh sẽ tập trung vào phần khen. Hai: đổ cho "hệ thống chặt" và "nhiều bạn khác cũng tốt" — Linh sẽ
> hiểu rằng đây là vấn đề hạn ngạch chứ không phải vấn đề đóng góp, nên không có gì để cải thiện. Ba:
> "đừng buồn" là yêu cầu Linh quản lý cảm xúc để Hà đỡ khó xử. Bốn: "năm sau chị sẽ cố gắng" là một lời
> hứa mơ hồ không có nội dung.
>
> Kết quả thực tế của phiên bản này: Linh ra khỏi phòng không hiểu mình cần làm gì khác, tin rằng mình bị
> thiệt vì hạn ngạch, và mất niềm tin vào ladder.

> **Phiên bản nói đúng:**
> "Chị muốn nói thẳng ngay từ đầu để em không phải đoán: mức của em kỳ này là 'đạt kỳ vọng'. Chị biết em
> tự đánh giá là 'vượt', nên chị muốn dành phần lớn buổi hôm nay để giải thích chị đọc bằng chứng như thế
> nào, và nghe em phản biện. Nếu chị bỏ sót gì, chị muốn biết.
>
> Trước hết, 'đạt kỳ vọng' ở bậc L4 không phải là mức tầm thường. Nó có nghĩa là em đang làm đúng công
> việc của một Senior Engineer, và ở tổ chức này phần lớn mọi người ở mức đó. Chị nói vậy không phải để
> an ủi mà để đặt đúng thang.
>
> Bây giờ chị nói vì sao không phải 'vượt'. Chị dùng bốn trục của ladder.
>
> Về khối lượng: em làm nhiều nhất team, chị công nhận. Em đã hoàn thành 23 task trong kỳ, cao nhất. Nhưng
> khối lượng không phải là một trục trong ladder, và đây là điểm chị nghĩ mình đã không nói rõ với em từ
> đầu kỳ — đó là lỗi của chị.
>
> Về phạm vi: các task của em nằm trong module em đã thạo. Không có việc nào mở rộng ra ngoài phạm vi đó.
> Vượt kỳ vọng ở L4 nghĩa là bắt đầu chạm vào những thứ của L5 — ví dụ dẫn một việc mà team khác phải
> thay đổi cách làm.
>
> Về tác động lên người khác: chị tìm trong 6 tháng và không thấy bằng chứng. Em review PR nhanh nhưng
> phần lớn comment là về style. Không có ai trong team giỏi lên rõ rệt nhờ em. Đây là trục em yếu nhất và
> cũng là trục có đòn bẩy lớn nhất cho bậc tiếp theo.
>
> Chị dừng ở đây. Em có bằng chứng nào chị bỏ sót không? Chị hỏi thật — chị chỉ nhìn được phần chị nhìn
> thấy.
>
> [Linh nêu hai việc Hà không biết: đã hỗ trợ team mobile debug một vấn đề trong 3 ngày, và đã viết một
> script mà hai team khác đang dùng.]
>
> Hai việc đó chị hoàn toàn không biết. Việc script được hai team dùng là bằng chứng cho trục phạm vi, và
> nó thật sự đáng tính. Nó không đủ để đổi mức tổng thể, và chị sẽ giải thích vì sao: một việc là một sự
> kiện, ba việc là một mẫu, và bậc được xác định bằng mẫu. Nhưng chị sẽ ghi nó vào hồ sơ, và nó là điểm
> khởi đầu tốt cho kỳ sau.
>
> Và có một điều chị muốn em nghe rõ: việc chị không biết hai việc đó là một vấn đề của hệ thống, không
> phải lỗi của em. Từ kỳ sau, chị muốn em giữ một file ghi lại những việc như vậy, mỗi tháng vài dòng.
> Không phải để khoe — mà vì nếu em không ghi thì chị sẽ viết review dựa trên những gì chị tình cờ nhìn
> thấy, và cái đó thiên vị những người ngồi gần chị và hay phát biểu trong họp.
>
> Cuối cùng, việc cụ thể cho 6 tháng tới. Chị đề nghị hai việc, và chị muốn em nói lại xem có hợp lý
> không: một, em nhận mentor Duy — không phải trả lời khi Duy hỏi, mà là cùng Duy đặt mục tiêu và theo
> dõi. Hai, chị sẽ giao em dẫn phần tích hợp với team logistics, việc này bắt buộc phải làm việc với hai
> team khác. Cả hai việc này nhắm thẳng vào trục em đang yếu. Nếu sau 6 tháng em làm được và có bằng chứng,
> hồ sơ của em sẽ mạnh. Chị không hứa mức, chị hứa rằng chị sẽ đưa hồ sơ đó ra và chị sẽ ủng hộ nó."

Bảy đặc điểm làm phiên bản này hoạt động: nói kết luận trước (không để người nghe treo lơ lửng); tách
"đạt" khỏi hàm ý tiêu cực bằng cách đặt lại thang; dùng tài liệu chung thay vì ý kiến cá nhân; thừa nhận
phần lỗi của manager (hai lần); hỏi phản biện và **thực sự thay đổi hồ sơ** khi nhận bằng chứng mới; giải
thích nguyên tắc "một sự kiện không phải một mẫu" thay vì chỉ bác bỏ; và kết bằng hành động cụ thể nhắm
đúng khoảng cách.

**Tình huống B — Cùng một bản review, ba góc nhìn.**

Sự việc: Khoa, L4, làm việc trên hạ tầng. Kỳ này Khoa được xếp "Đạt kỳ vọng", trong khi Linh làm feature
được xếp "Vượt kỳ vọng".

| | Nhìn từ IC (Khoa) | Nhìn từ Tech Lead (Tuấn) | Nhìn từ Manager (Hà) |
|---|---|---|---|
| **Điều nhìn thấy** | "Tôi giữ cho hệ thống không sập. Không có sự cố nào cả năm. Nhưng vì không có sự cố nên không ai thấy tôi làm gì." | "Khoa làm phần khó nhất và ít được nhìn thấy nhất. Nếu Khoa nghỉ, ba tháng sau mọi thứ bắt đầu vỡ." | "Tôi không có cách nào chứng minh giá trị của việc không có sự cố trong buổi calibration. Bằng chứng của Linh nhìn thấy được; bằng chứng của Khoa là sự vắng mặt của vấn đề." |
| **Cơ chế đang hoạt động** | Bias hiển thị: công việc phòng ngừa vô hình theo bản chất | Biết vấn đề nhưng không có ngôn ngữ để đưa vào hệ thống đánh giá | Hệ thống đo lường đang đo output nhìn thấy được, không đo rủi ro được loại bỏ |
| **Rủi ro nếu không xử lý** | Khoa rời đi trong 2–3 quý; đây là dạng regretted turnover đắt nhất | Mất năng lực vận hành, phát hiện sau 3–6 tháng | Sau khi Khoa đi, chi phí incident tăng và không ai nối được nguyên nhân với quyết định review 2 quý trước |
| **Hành động đúng** | Ghi brag document theo dạng "rủi ro đã loại bỏ": "phát hiện và sửa X trước khi lên prod, nếu lọt thì hậu quả là Y" | Chủ động viết một đoạn xác nhận đóng góp của Khoa để Hà dùng trong calibration — feedback ngang cấp có trọng lượng cao trong calibration | Đưa "bias hiển thị" thành một mục kiểm tra bắt buộc trong quy trình calibration (đã có trong template ở trên); và đo công việc phòng ngừa bằng chỉ số proxy: số incident tránh được, MTTR, số lần rollback thành công |

Đây là một trong những thất bại có hệ thống lớn nhất của performance review trong engineering, và nó
không sửa được bằng cách "cố gắng công bằng hơn". Nó chỉ sửa được bằng cách **thay đổi cái được đo** —
tức là bằng thiết kế, không bằng thiện chí.

### Best Practices

- **Ghi feedback log 3 phút/tuần/người.** Lý do: đây là biện pháp duy nhất chống recency bias có hiệu quả
  thật; mọi biện pháp khác đều dựa vào trí nhớ, và trí nhớ thì thiên vị có hệ thống.
- **Không có gì trong review được phép là bất ngờ.** Lý do: nếu một vấn đề đáng đưa vào review, nó đáng
  được nói sớm hơn để người ta có cơ hội sửa; đưa vào review lần đầu nghĩa là bạn đã chọn không cho họ cơ
  hội đó.
- **Tách buổi review phát triển khỏi buổi thông báo lương, cách nhau 2–4 tuần.** Lý do: hai trạng thái
  tâm lý xung khắc; gộp lại thì phần phát triển không được tiếp thu.
- **Bắt buộc calibration trước khi thông báo bất kỳ mức nào.** Lý do: chuẩn cá nhân của mỗi manager lệch
  1–2 bậc; không calibration thì mức phản ánh manager nhiều hơn phản ánh nhân viên.
- **Sau calibration, mức đó là mức của bạn — không đổ cho hệ thống.** Lý do: đổ lỗi cho hệ thống mua được
  một chút thoải mái tức thời bằng cách phá huỷ niềm tin dài hạn vào cả bạn lẫn quy trình.
- **Tối đa 2 mục "cần khác đi".** Lý do: khả năng thay đổi hành vi có ý thức của con người là hữu hạn;
  danh sách 6 mục dẫn tới thay đổi 0 mục.
- **Yêu cầu tự đánh giá được viết trước khi nhân viên đọc đánh giá của manager.** Lý do: cùng cơ chế
  chống cascade như debrief tuyển dụng; và độ lệch giữa hai bản là thông tin có giá trị cao.
- **Kiểm tra bias hiển thị trong mọi kỳ calibration.** Lý do: công việc phòng ngừa và công việc nền tảng
  bị đánh giá thấp một cách có hệ thống, và hậu quả xuất hiện 2–4 quý sau dưới dạng mất người và tăng
  incident.

### Anti-patterns

**"Review là bất ngờ."** Nhân viên nghe một lời phê bình lần đầu trong buổi review. Cơ chế phá hoại: nó
chứng minh manager đã quan sát vấn đề nhưng chọn im lặng, nên nhân viên hiểu rằng manager sẽ ghi nhớ
thầm và tính sổ sau — điều này giết Psychological Safety nhanh hơn gần như mọi hành vi khác. Dấu hiệu
sớm: manager tránh các cuộc trò chuyện khó trong quý; các buổi 1-1 chỉ nói về tiến độ công việc.

**"Đánh giá dựa trên ba tuần gần nhất."** Cơ chế: recency bias không được bù đắp. Dấu hiệu sớm: đọc lại
5 bản review gần nhất và kiểm tra ngày tháng của các bằng chứng — nếu chúng dồn vào 2 tháng cuối kỳ, bạn
đang mắc.

**"Dùng review để xả bức xúc tích tụ."** Manager đã khó chịu về một hành vi suốt 8 tháng, không nói, và
buổi review trở thành nơi tất cả đổ ra. Cơ chế: cảm xúc tích tụ làm mất tỉ lệ — một vấn đề nhỏ được nói
với cường độ của tám tháng, và người nghe phản ứng với cường độ đó chứ không với nội dung. Kết quả là
quan hệ hỏng và vấn đề vẫn không được sửa. Dấu hiệu sớm: bạn thấy mình "đang chờ tới kỳ review để nói".
Nếu bạn có suy nghĩ đó, đó chính là dấu hiệu phải nói ngay tuần này.

**"Sandwich feedback."** Kẹp lời phê bình giữa hai lời khen. Cơ chế: người nghe học được rằng lời khen là
tín hiệu báo trước lời chê, nên mọi lời khen về sau đều mất giá trị; và nội dung phê bình bị làm mờ tới
mức người nghe không nhận ra đó là phê bình. Dấu hiệu sớm: sau buổi feedback, khi hỏi lại "em hiểu điều
gì cần khác đi", nhân viên không trả lời được.

**"Điểm số không có bằng chứng."** Bản review có mức nhưng không có sự việc nào có ngày tháng. Cơ chế:
không phản biện được, không học được từ, và không kiểm toán được khi có tranh chấp. Dấu hiệu sớm: bản
review dài dưới nửa trang, hoặc dài nhưng toàn tính từ.

**"Forced ranking áp lên team 6 người."** Cơ chế đã phân tích ở First Principles. Dấu hiệu sớm: manager
bắt đầu nói về nhân viên bằng ngôn ngữ hạn ngạch ("năm nay chị chỉ có một suất vượt"); hoặc xuất hiện
hành vi tránh việc chung trong team.

### Khi nào KHÔNG nên áp dụng

**Trong 6 tháng đầu của một người mới.** Áp bộ máy review đầy đủ vào người còn đang onboard là đo lường
trên một mẫu chưa ổn định, và nó gửi tín hiệu sai (bạn đang bị đánh giá trước khi bạn kịp học). Cái nên
làm thay: dùng khung 30/60/90 ở chủ đề 4 với các mốc kiểm được, và một cuộc trò chuyện ở ngày 90 không
có điểm số.

**Ở tổ chức dưới 8 người nơi mọi người làm việc cạnh nhau hằng ngày.** Ở quy mô đó, thông tin về hiệu suất
là hiển nhiên với tất cả mọi người, và một quy trình review chính thức tạo ra nghi thức nặng nề không
thêm thông tin. Cái nên làm thay: feedback liên tục thật sự (tầng 1), và một cuộc trò chuyện thẳng thắn
hai lần/năm về lương và định hướng — vẫn cần văn bản ngắn để có bằng chứng khi tổ chức lớn lên, nhưng
không cần bộ máy.

**Khi tổ chức đang trong khủng hoảng cấp tính.** Đang có sự cố kéo dài, đang chạy nước rút sống còn, hoặc
vừa có cắt giảm nhân sự. Review trong giai đoạn này sẽ được đọc qua lăng kính sinh tồn: "mức của tôi có
nghĩa là tôi có bị cắt không". Không có cuộc trò chuyện phát triển nào diễn ra được trong trạng thái đó.
Cái nên làm thay: hoãn 4–8 tuần, nói rõ vì sao hoãn và khi nào sẽ làm, và trong lúc đó vẫn duy trì
feedback liên tục.

**Khi bạn đã quyết định chấm dứt hợp tác với một người.** Dùng buổi review để lần đầu tiên báo hiệu vấn
đề nghiêm trọng, trong khi quyết định đã có, là không trung thực và về mặt thủ tục là nguy hiểm. Vấn đề
underperformance có quy trình riêng, bắt đầu sớm hơn nhiều và độc lập với chu kỳ review — xem chủ đề 8.
Review không phải là công cụ để chuẩn bị hồ sơ sa thải; nếu bạn đang dùng nó như vậy, cả hệ thống review
của bạn sẽ bị mọi người đọc theo cách đó trong nhiều năm.

---

## 7. Promotion Framework

### Problem Statement

Khoa được promote lên Tech Lead vào tháng 3. Đến tháng 9, anh xin quay lại vai trò IC. Trong sáu
tháng đó: hai người trong team xin chuyển sang team khác, hai dự án trễ, và Khoa — người trước đó là
engineer được tin cậy nhất về hệ thống thanh toán — mất phần lớn uy tín mà anh đã xây trong bốn năm.

Đọc lại quá trình ra quyết định, không có ai làm sai một cách rõ ràng. Khoa là engineer giỏi nhất
team. Anh đã ở công ty lâu nhất. Anh vừa nhận một offer bên ngoài và công ty muốn giữ. Vị trí Tech
Lead đang trống. Bốn dữ kiện đó, cộng lại, tạo thành một quyết định mà mọi người đều thấy hợp lý ở
thời điểm đó. Cái không ai hỏi: **trong sáu tháng trước đó, Khoa đã từng làm việc gì ở mức Tech Lead
chưa?** Câu trả lời là chưa — anh chưa từng dẫn một quyết định kiến trúc có nhiều bên liên quan, chưa
từng đưa feedback khó cho ai, chưa từng đại diện team trong một cuộc họp có xung đột ưu tiên.

Đây là dạng thất bại đặc trưng của promotion: nó được dùng như một **lời hứa** thay vì một **sự xác
nhận**. Hậu quả quan sát được khi tổ chức không có framework promotion dựa trên bằng chứng:

- Người được promote thất bại ở bậc mới, và thất bại này khó đảo ngược hơn nhiều so với việc không
  promote — quay lại bậc cũ gần như luôn kèm theo việc người đó rời công ty trong vòng một năm.
- Những người khác đọc được quy luật thật: promotion là hàm của thâm niên, của việc có offer bên
  ngoài, hoặc của mức độ thân thiết với manager. Họ điều chỉnh hành vi theo đúng quy luật đó.
- Người làm việc tốt nhưng ít nói và không có ai nói hộ bị bỏ qua nhiều chu kỳ liên tiếp, và thường
  họ không phàn nàn — họ chỉ lặng lẽ nộp hồ sơ chỗ khác.
- Manager mất khả năng nói "chưa" một cách có cơ sở, vì không có tiêu chí nào để chỉ vào. Mỗi cuộc
  trò chuyện về promotion trở thành một cuộc đàm phán cá nhân.

### First Principles

**Promotion là xác nhận, không phải lời hứa.** Nguyên tắc cốt lõi: một người được promote lên bậc N
khi họ **đã hoạt động ở bậc N trong khoảng hai quý**, một cách nhất quán và có bằng chứng. Lý do
không phải là sự khắt khe hành chính, mà là bất đối xứng rủi ro:

| | Promote quá sớm | Promote quá muộn |
|---|---|---|
| Ai chịu hậu quả | Người được promote, team, và tổ chức | Chủ yếu tổ chức (rủi ro mất người) |
| Khả năng sửa | Rất khó — hạ bậc gần như đồng nghĩa với mất người | Dễ — promote ở chu kỳ sau, có thể kèm điều chỉnh lương |
| Thời gian phát hiện | 4–8 tháng, sau khi thiệt hại đã xảy ra | Ngay, nếu người đó nói ra |
| Tác động lên uy tín hệ thống | Người khác thấy bậc đó không có nghĩa gì | Người khác thấy bậc đó khó nhưng công bằng |

Bất đối xứng này giải thích vì sao quy tắc đúng nghiêng về phía thận trọng — nhưng chỉ khi tổ chức có
cơ chế bù: nói rõ với người đó họ đang thiếu gì, có lộ trình cụ thể, và có cách điều chỉnh lương độc
lập với việc đổi bậc. Thiếu cơ chế bù, "thận trọng" chỉ là một cách mất người có mỹ từ.

**Peter Principle và vì sao nó không phải chuyện đùa.** Nguyên lý (nguồn công khai) nói rằng trong
một hệ thống promote dựa trên hiệu suất ở bậc hiện tại, người ta được thăng tiến cho tới khi tới bậc
mà họ không còn làm tốt — và dừng lại ở đó. Cơ chế của nó nằm ở một giả định sai: rằng hiệu suất ở
bậc N dự báo hiệu suất ở bậc N+1. Trong engineering, giả định này sai đặc biệt mạnh ở một chỗ: bước
từ IC sang Tech Lead hoặc Manager là bước **đổi hàm mục tiêu**, không phải bước tăng độ khó. Một
người viết code xuất sắc và một người làm cho tám người viết code tốt hơn là hai năng lực khác nhau,
tương quan yếu.

Cách chặn Peter Principle không phải là promote ít hơn, mà là **đo năng lực của bậc mới trước khi
promote** — tức tạo cơ hội cho người đó làm thử phần việc của bậc đó trong phạm vi giới hạn, có hỗ
trợ, và quan sát kết quả. Đây là lý do "đã hoạt động ở bậc N trong hai quý" không phải một rào cản
hành chính mà là **phép đo duy nhất có hiệu lực dự báo**.

**Sponsorship — cơ chế ít được nói tới ở Việt Nam nhưng quyết định kết quả.** Có ba dạng hỗ trợ khác
nhau, và chúng thường bị gộp làm một:

- *Mentoring*: nói **với** bạn. Chuyển giao kiến thức và kinh nghiệm.
- *Coaching*: hỏi bạn. Giúp bạn tự rút ra kết luận.
- *Sponsorship*: nói **về** bạn khi bạn không có mặt trong phòng. Đặt tên bạn vào danh sách khi có
  cơ hội. Bảo vệ hồ sơ của bạn trong buổi calibration.

Trong bất kỳ tổ chức nào lớn hơn hai team, quyết định promotion được ra trong một căn phòng mà ứng
viên không có mặt. Chất lượng công việc của bạn được truyền vào phòng đó **qua một người**. Không có
người đó, công việc tốt vẫn tồn tại nhưng không được nhìn thấy. Đây là cơ chế giải thích một hiện
tượng phổ biến: người làm việc chăm chỉ, ít phàn nàn, không tự quảng bá thường bị bỏ qua nhiều chu
kỳ — không phải vì ai đó thiên vị có chủ đích, mà vì trong phòng không có ai đại diện cho họ.

Hệ quả cho lead: **sponsorship là một phần công việc của bạn, không phải một ân huệ.** Nếu bạn có
người trong team làm việc ở mức bậc trên mà bạn không viết hồ sơ cho họ, bạn đang để hệ thống ăn mòn
động lực của chính người bạn cần giữ nhất.

### Mental Model

Ba câu hỏi tạo thành khung quyết định promotion, và chúng phải được trả lời theo đúng thứ tự:

```
1. PHẠM VI ĐÃ MỞ RỘNG CHƯA?
   Người này đã và đang giải quyết loại bài toán của bậc trên chưa?
   (Không: "họ có khả năng làm được" — mà: "họ đã làm, đây là bằng chứng")
        ↓ có
2. CÓ CHỖ CHO PHẠM VI ĐÓ KHÔNG?
   Tổ chức có nhu cầu thật ở bậc đó không? (Với vai trò quản lý:
   có team để dẫn không? Với Staff+: có bài toán đủ lớn không?)
        ↓ có
3. NÓ CÓ BỀN VỮNG KHÔNG?
   Hai quý liên tiếp, qua nhiều loại tình huống, không phải một dự án
   xuất sắc duy nhất?
        ↓ có
   → Viết hồ sơ, đưa vào calibration.
```

Câu 2 là câu hay bị bỏ và là nguồn của một loại đau khổ riêng: promote ai đó lên Staff Engineer trong
một tổ chức 15 người không có bài toán ở tầm đó tạo ra một người có title mà không có việc tương ứng
— và trong vòng một năm, người đó hoặc tự tạo ra công việc phức tạp không cần thiết, hoặc rời đi vì
title mới hoá ra rỗng.

**Ma trận "sẵn sàng × có chỗ":**

| | Có chỗ ở bậc trên | Chưa có chỗ |
|---|---|---|
| **Đã hoạt động ở bậc trên** | Promote. Đây là trường hợp duy nhất rõ ràng. | Nói thật về ràng buộc tổ chức; điều chỉnh lương trong bậc hiện tại; giúp họ tìm phạm vi ở nơi khác trong công ty. Nếu không làm gì, bạn sẽ mất họ trong 2–3 quý. |
| **Chưa hoạt động ở bậc trên** | Tạo cơ hội có kiểm soát để họ thử, có hỗ trợ, có tiêu chí. Không promote trước rồi hy vọng. | Nói rõ lộ trình và điều kiện; đừng hứa ngày. |

### Practical Framework

**Bước 1 — Hồ sơ dựa trên bằng chứng.** Hồ sơ do manager viết (không phải do ứng viên tự viết, dù
ứng viên đóng góp dữ liệu), và nó phải trả lời được một câu duy nhất: *người này đã hoạt động ở bậc
mới chưa, và bằng chứng nào?*

```
HỒ SƠ PROMOTION
Người:        Quân · Hiện tại: Senior Engineer (L4) · Đề nghị: Staff Engineer (L5)
Manager:      Hà · Chu kỳ: Q3/2026 · Ngày nộp: [ngày]

1. TÓM TẮT ĐỀ NGHỊ (5 dòng)
   Quân đã hoạt động ở phạm vi L5 từ Q1/2026: dẫn quyết định kiến trúc vượt
   ranh giới hai team, và tác động của công việc anh làm được đo bằng thay đổi
   ở chỉ số cấp tổ chức, không chỉ trong team.

2. BẰNG CHỨNG THEO TỪNG TRỤC CỦA LADDER
   [Trục: Phạm vi ảnh hưởng — yêu cầu L5: vượt ra ngoài một team]
     - Dẫn RFC-042 (thư viện HTTP client có timeout mặc định), áp dụng bởi
       4 team. Số incident do dependency treo giảm từ 4 xuống 0 trong 2 quý.
       (số minh hoạ) · Link: [RFC] [dashboard]
     - Người phản biện chính cho ADR-031 của team Payment — team anh không
       thuộc về. Xác nhận từ Tuấn (Tech Lead Payment): [link nhận xét]

   [Trục: Độ phức tạp bài toán — yêu cầu L5: bài toán mơ hồ, chưa có định nghĩa]
     - Được giao câu hỏi mở "vì sao lead time của chúng ta tăng gấp đôi trong
       6 tháng". Tự định nghĩa bài toán, thu dữ liệu, đưa 3 phương án. Kết quả:
       [link phân tích] · Quyết định của tổ chức dựa trên phân tích này.

   [Trục: Tác động lên người khác — yêu cầu L5: nâng năng lực nhiều người]
     - Mentor cho Vy và Duy; cả hai đã tự dẫn được design review trong Q3.
     - Viết chuẩn review cho toàn bộ backend guild: [link]

   [Trục: Tự chủ — yêu cầu L5: tự tìm ra việc cần làm]
     - 3/4 công việc lớn của Quân trong 2 quý qua do anh tự xác định, không
       phải do manager giao. Danh sách: [link]

3. ĐIỀU CHƯA MẠNH (bắt buộc có — hồ sơ không có mục này sẽ bị trả lại)
   - Viết văn bản còn dài và khó theo dõi với người ngoài lĩnh vực. Đã có
     feedback, đang cải thiện, chưa đạt mức mong đợi ở L5.
   - Chưa có kinh nghiệm dẫn một quyết định mà anh là người phải nói "không"
     với một stakeholder cấp cao hơn.

4. NGƯỜI XÁC NHẬN (không phải chỉ manager)
   Tuấn (TL Payment), Trang (PO), Linh (Senior, team khác) — nhận xét: [link]

5. NẾU KHÔNG PROMOTE LẦN NÀY THÌ CÒN THIẾU GÌ CỤ THỂ
   [Bắt buộc điền, kể cả khi bạn tin hồ sơ sẽ được duyệt. Đây là nội dung
    bạn sẽ nói với Quân nếu calibration không thông qua.]
```

Mục 3 và mục 5 là hai mục làm nên sự khác biệt giữa một hệ thống promotion đáng tin và một hệ thống
vận động hành lang. Hồ sơ chỉ nói điều tốt là hồ sơ bán hàng, và người đọc calibration sẽ đối xử với
nó như vậy.

**Bước 2 — Calibration liên team.** Mục tiêu duy nhất: giảm phương sai giữa các manager. Không có
calibration, "Senior" ở team của Hà và "Senior" ở team bên cạnh là hai thứ khác nhau, và trong vòng
hai năm không ai còn tin vào bậc.

| Bước | Nội dung | Quy tắc cứng |
|---|---|---|
| Trước họp | Mọi manager đọc toàn bộ hồ sơ, ghi đánh giá độc lập | Không thảo luận trước — tránh information cascade |
| Mở đầu | Đọc lại định nghĩa bậc, không đọc tên người | Neo vào tiêu chí trước khi neo vào con người |
| Trình bày | Manager trình bày bằng chứng, không trình bày cảm nhận | Câu "tôi tin bạn ấy sẵn sàng" không phải bằng chứng |
| Phản biện | Người ngoài team hỏi: bằng chứng này có ở bậc dưới không? | Bài kiểm tra phân biệt: một L4 xuất sắc cũng làm được điều này chứ? |
| So chiều ngang | Đặt cạnh một người đã ở bậc đó | Nếu hồ sơ yếu hơn rõ rệt, chưa đủ |
| Chốt | Người có accountability quyết, ghi lý do | Lý do phải viết lại được cho ứng viên nghe |

**Bước 3 — Thông báo kết quả.** Đây là phần quyết định uy tín của cả hệ thống, và nó thường bị làm
qua loa vì khó.

*Với người được promote*: nói rõ bằng chứng nào đã thuyết phục, và nói rõ **kỳ vọng mới** — vì bậc
mới không phải phần thưởng mà là một tập kỳ vọng khác. Nhiều người được promote xong vẫn làm y như
cũ, rồi ba quý sau bị đánh giá là không đáp ứng, và họ có lý do chính đáng để thấy bất công.

*Với người không được promote*: đây là cuộc trò chuyện quan trọng hơn. Nguyên tắc:

- **Không được là bất ngờ.** Nếu người đó ngạc nhiên, lỗi thuộc về bạn, không thuộc về họ. Kỳ vọng
  và khoảng cách phải đã được nói trong các 1-1 nhiều tháng trước.
- **Nói khoảng cách bằng hành vi quan sát được, không bằng cảm nhận.** "Anh cần chín chắn hơn" là
  câu vô dụng. "Trong hai quý qua, mọi công việc lớn của em đều do anh giao; ở bậc L5, khoảng một nửa
  phải do em tự xác định — đây là khoảng cách cụ thể" là câu dùng được.
- **Đừng hứa ngày.** "Quý sau chắc chắn được" là lời hứa bạn không kiểm soát (calibration, ngân sách,
  nhu cầu tổ chức đều có thể đổi). Hứa rồi không giữ hai lần liên tiếp là cách phá hỏng quan hệ tin
  cậy nhanh nhất trong quản trị con người.
- **Đưa ra lộ trình có tiêu chí kiểm được và hai điểm kiểm tra.** Nếu bạn không viết được lộ trình cụ
  thể, có nghĩa là bạn cũng không biết khoảng cách nằm ở đâu — và trong trường hợp đó, vấn đề là ở
  đánh giá của bạn chứ không phải ở người kia.

```
LỘ TRÌNH 2 QUÝ · Quân · L4 → L5
Viết ngày [ngày] · Xem lại: [ngày +6 tuần], [ngày +12 tuần], [ngày +24 tuần]

KHOẢNG CÁCH 1 — Tự xác định công việc
  Hiện tại: 1/4 công việc lớn do Quân tự xác định.
  Cần thấy: ≥ 2 công việc lớn trong 2 quý mà Quân tự phát hiện vấn đề,
            tự dựng phương án, và thuyết phục được người khác đầu tư.
  Hỗ trợ:   Hà sẽ ngừng giao việc lớn trong Q4, thay bằng nêu vùng vấn đề.
  Bằng chứng sẽ dùng: RFC hoặc phân tích có tên Quân là người khởi xướng.

KHOẢNG CÁCH 2 — Viết cho người ngoài lĩnh vực
  Hiện tại: văn bản dài, người ngoài không theo được.
  Cần thấy: 2 văn bản mà một stakeholder không kỹ thuật đọc và ra quyết
            định được, không cần hỏi lại.
  Hỗ trợ:   Trang (PO) đồng ý làm người đọc thử và cho feedback trong 24h.

ĐIỀU HÀ CAM KẾT
  - Tạo ít nhất một cơ hội để Quân dẫn một quyết định có xung đột lợi ích.
  - Đưa hồ sơ vào calibration Q1 nếu bằng chứng đạt, bất kể ngân sách —
    nếu ngân sách là ràng buộc, Hà sẽ nói thẳng thay vì dùng tiêu chí năng lực
    để che một vấn đề ngân sách.
```

Cam kết cuối cùng đáng chú ý: **không được dùng lý do năng lực để che một ràng buộc ngân sách hoặc
tổ chức.** Đây là hành vi phổ biến vì nó tránh được một cuộc trò chuyện khó, và nó phá hoại nghiêm
trọng: người đó sẽ dành hai quý để sửa một khoảng cách không tồn tại, rồi phát hiện ra sự thật, rồi
không bao giờ tin bạn nữa.

### Trade-off

| Trục | Nghiêng về A khi | Nghiêng về B khi | Cái giá của mỗi bên |
|---|---|---|---|
| **A: Promote sau khi đã chứng minh** vs **B: Promote để tạo động lực** | Bậc mới đổi bản chất công việc (IC → Lead/Manager); tổ chức đủ lớn để bậc có ý nghĩa so sánh | Bậc mới chỉ là mở rộng tuyến tính; công ty nhỏ đang cần tín hiệu giữ người gấp | A: rủi ro mất người trong lúc chờ, và bị đối thủ tuyển mất bằng title. B: rủi ro thất bại khó đảo, và làm loãng ý nghĩa của bậc cho mọi người khác |
| **A: Ladder chung toàn công ty** vs **B: Tiêu chí theo từng team** | > 3 team; cần so sánh ngang và di chuyển nội bộ | Các team làm việc bản chất rất khác (nghiên cứu vs vận hành) | A: một số team thấy tiêu chí không khớp thực tế của họ. B: bậc mất khả năng so sánh, và cảm nhận bất công tăng |
| **A: Promotion theo chu kỳ cố định** vs **B: Promote bất cứ lúc nào đủ điều kiện** | Cần calibration công bằng; tổ chức > 30 người | Tổ chức nhỏ, linh hoạt, ít rủi ro lệch chuẩn | A: người sẵn sàng phải chờ tới 6 tháng. B: phương sai giữa manager tăng nhanh, và người có manager "hào phóng" được lợi |
| **A: Counter-promote khi có offer ngoài** vs **B: Giữ nguyên nguyên tắc** | Người đó thật sự đã ở bậc trên và bạn chỉ đang chậm; bạn thừa nhận lỗi của mình | Hồ sơ chưa đạt và bạn đang phản ứng với áp lực | A: nếu người đó xứng đáng, đây là sửa sai chính đáng — nhưng phải nói rõ đó là sửa sai, không phải phần thưởng cho việc có offer. B: có thể mất người, nhưng giữ được tính toàn vẹn của hệ thống. Chọn A nhiều lần = dạy cả tổ chức rằng cách nhanh nhất để được promote là đi phỏng vấn chỗ khác |

Trade-off cuối cùng đáng phân tích kỹ vì nó rất phổ biến ở thị trường Việt Nam khi thị trường nóng.
Vấn đề không phải là "counter-promote luôn sai". Vấn đề là **tín hiệu bạn phát ra**. Nếu trong hai
năm có bốn người được promote sau khi có offer bên ngoài và không ai được promote mà không có offer,
mọi người trong tổ chức đã học được quy luật thật, và họ sẽ hành động theo nó. Cách duy nhất để dùng
A mà không phá hệ thống: nói thẳng, cả với người đó và với calibration, rằng "chúng tôi đã chậm, đây
là sửa sai" — và đồng thời rà lại xem còn ai đang ở tình trạng tương tự mà chưa có offer.

### Real-world Scenarios

**Tình huống A — Người xứng đáng nhưng không có chỗ (startup product, 22 engineer).**

*Bối cảnh.* Linh đã hoạt động ở mức Staff Engineer trong ba quý: dẫn hai quyết định kiến trúc xuyên
team, xây dựng nền tảng observability mà cả ba team dùng, mentor thành công hai Senior. Nhưng công ty
chưa có bậc Staff — ladder dừng ở Senior, và bậc tiếp theo trên giấy tờ là Engineering Manager. Linh
không muốn làm quản lý.

*Các lựa chọn.* (1) Promote lên EM và hy vọng cô thích nó. (2) Tạo title "Senior II" như một cách xoa
dịu. (3) Tăng lương trong bậc Senior, không đổi title, giải thích ràng buộc. (4) Tạo bậc Staff thật
với định nghĩa đầy đủ, chấp nhận chi phí thiết kế ladder.

*Trade-off.* Lựa chọn 1 là con đường mặc định ở phần lớn công ty Việt Nam quy mô này, và nó là cách
phổ biến nhất để mất một IC xuất sắc — hoặc mất cô ngay (cô từ chối), hoặc mất cô chậm (cô nhận, làm
không hợp, rồi rời đi sau một năm cùng với việc team mất một EM). Lựa chọn 2 rẻ nhưng tạo ra một bậc
rỗng: không có định nghĩa, không có kỳ vọng khác, nên trong sáu tháng nó thành vô nghĩa và Linh biết
điều đó. Lựa chọn 3 trung thực nhưng không giải quyết vấn đề dài hạn — cô vẫn không có đường đi. Lựa
chọn 4 tốn khoảng ba đến bốn tuần công của Head of Engineering để viết ladder, calibration lại toàn
bộ nhân sự hiện có, và tạo ra một cuộc trò chuyện khó với những Senior khác chưa đạt Staff.

*Quyết định.* Chọn 4, nhưng làm tối giản: ladder một trang, bốn trục, sáu bậc, không cố hoàn hảo.
Đồng thời nói thẳng với Linh về tiến độ và mốc thời gian, và tăng lương trong bậc hiện tại ngay lập
tức để không dùng thời gian thiết kế ladder như một cái cớ trì hoãn.

*Hậu quả.* Linh trở thành Staff Engineer đầu tiên của công ty sau hai tháng. Quan trọng hơn: hai
Senior khác lần đầu tiên nhìn thấy một con đường IC không dẫn tới quản lý, và một trong hai người đã
định chuyển sang nhánh quản lý "vì không còn đường nào khác" đã đổi ý.

*Bài học.* Thiếu bậc không phải một vấn đề hành chính, nó là một khiếm khuyết cấu trúc đẩy IC giỏi
vào vai trò sai. Chi phí sửa (ba tuần viết ladder) nhỏ hơn nhiều so với chi phí không sửa (mất một
Staff-level IC và có thêm một EM không phù hợp). Và: ladder tối giản dùng được tốt hơn nhiều so với
ladder hoàn hảo chưa viết xong.

**Tình huống B — cùng một quyết định "chưa promote", ba góc nhìn.**

Sự việc: Vy (Mid-level, 3 năm kinh nghiệm) không được promote lên Senior trong chu kỳ Q3. Cô đã được
manager nói từ Q1 rằng "khả năng cao là Q3".

*Nhìn từ IC (Vy).* "Tôi đã làm mọi thứ được yêu cầu. Tôi giao hàng đúng hạn, chất lượng tốt, không ai
phàn nàn gì. Bạn tôi ở công ty khác đã lên Senior với hai năm kinh nghiệm. Nếu ở đây tôi phải chờ
thêm, có lẽ tôi nên xem thị trường." Vy đang so sánh với một tham chiếu bên ngoài mà cô không biết
tiêu chí, và cô đang đo bản thân bằng thứ dễ đo nhất — sản lượng và độ tin cậy. Không ai từng nói với
cô rằng bậc Senior không đo bằng sản lượng mà đo bằng **phạm vi tự chủ** và **tác động lên người
khác**.

*Nhìn từ Tech Lead (Tuấn).* "Vy làm việc rất tốt trong phạm vi được giao. Nhưng khi có một bài toán
mơ hồ, cô ấy chờ được giao thay vì tự định nghĩa. Trong ba lần design review gần đây, cô ấy không đưa
ý kiến nào cho thiết kế của người khác. Đó là khoảng cách." Tuấn nhìn đúng khoảng cách nhưng anh đã
mắc một lỗi: anh chưa bao giờ nói điều này với Vy bằng những từ cụ thể như vậy. Trong 1-1, anh nói
"em cứ tiếp tục thế này là ổn" — vì đó là câu dễ nói.

*Nhìn từ Manager (Hà).* Ở tầng này, một người bị bất ngờ khi không được promote là **lỗi của hệ
thống feedback, không phải lỗi của người đó hay của quyết định calibration**. Can thiệp của Hà không
nằm ở việc giải thích khéo léo hơn với Vy, mà ở ba chỗ: (a) câu "khả năng cao là Q3" ở Q1 là một lời
hứa không được phép nói — cô sẽ nói lại với toàn bộ manager về việc này; (b) mẫu 1-1 cần có một mục
bắt buộc về khoảng cách tới bậc tiếp theo, để nó không phụ thuộc vào việc manager có đủ can đảm nêu
hay không; (c) định nghĩa bậc phải được đưa cho mọi người đọc từ đầu, không phải chỉ manager có.

Điểm mấu chốt: cùng một kết quả calibration, IC đọc thành bất công so với thị trường, Tech Lead đọc
thành khoảng cách năng lực có thật, Manager đọc thành lỗi thiết kế của vòng feedback. Chỉ can thiệp ở
tầng thứ ba mới ngăn được việc này lặp lại với người tiếp theo.

### Best Practices

- **Promote để xác nhận, không để hứa.** Bằng chứng phải là "đã làm", không phải "có khả năng làm".
  Với bước IC → Lead/Manager, yêu cầu này chặt hơn nữa vì hàm mục tiêu đổi.
- **Luôn kiểm tra câu hỏi "có chỗ không" trước khi bắt đầu quy trình.** Promote vào một phạm vi không
  tồn tại tạo ra title rỗng và mất người trong vòng một năm.
- **Hồ sơ phải có mục "điều chưa mạnh" và mục "nếu không promote thì thiếu gì".** Hồ sơ chỉ toàn điểm
  tốt sẽ bị calibration đọc như tài liệu bán hàng, và nó làm hại chính ứng viên.
- **Coi sponsorship là nhiệm vụ, không phải ân huệ.** Rà soát định kỳ: trong team có ai đang làm việc
  ở bậc trên mà chưa có ai viết hồ sơ cho họ không?
- **Không bao giờ để kết quả là bất ngờ.** Khoảng cách phải được nói bằng hành vi quan sát được, lặp
  lại trong nhiều 1-1, trước chu kỳ ít nhất hai quý.
- **Không hứa ngày.** Hứa tiêu chí và hứa quy trình; ngày phụ thuộc những thứ bạn không kiểm soát.
- **Không dùng lý do năng lực để che ràng buộc ngân sách.** Nếu ràng buộc là tiền hoặc là chỗ, hãy
  nói đó là tiền hoặc chỗ.
- **Nói rõ kỳ vọng mới ngay khi promote.** Bậc mới là một tập kỳ vọng khác, không phải một phần
  thưởng cho quá khứ.
- **Nếu counter-promote vì offer bên ngoài, gọi tên nó là sửa sai** và rà soát xem còn ai ở tình
  trạng tương tự mà chưa lên tiếng.
- **Tách quyết định lương khỏi quyết định bậc ở mức có thể.** Điều này cho bạn công cụ để ghi nhận
  người xứng đáng khi chưa có chỗ ở bậc trên, thay vì buộc phải chọn giữa promote sớm và mất người.

### Anti-patterns

- **Promotion như phần thưởng thâm niên.** Cơ chế phá hoại: nó dạy cả tổ chức rằng thời gian, không
  phải phạm vi, là biến quyết định — nên hành vi tối ưu trở thành ở lại và chờ. Đồng thời nó tạo ra
  những người ở bậc cao mà không có năng lực tương ứng, làm bậc mất ý nghĩa cho tất cả những người
  khác. Dấu hiệu sớm: khi giải thích một promotion, câu đầu tiên là "bạn ấy ở đây lâu rồi".
- **Promote engineer giỏi lên quản lý như con đường duy nhất để tăng lương.** Đây là anti-pattern
  gây thiệt hại kép có hệ thống nhất trong ngành: tổ chức mất một IC xuất sắc và có thêm một manager
  không phù hợp, và cả hai tổn thất đều khó đảo. Dấu hiệu sớm: ladder không có bậc IC nào cao hơn
  Senior; hoặc có trên giấy nhưng chưa ai từng đạt.
- **Hứa "quý sau" nhiều lần.** Mỗi lần hứa và không giữ, chi phí không phải là sự thất vọng — mà là
  người đó ngừng tin vào mọi phát biểu khác của bạn, kể cả những phát biểu đúng. Sau lần thứ hai, họ
  bắt đầu tìm việc, và họ sẽ không nói cho bạn biết.
- **Hồ sơ do ứng viên tự viết toàn bộ và manager chỉ ký.** Điều này chuyển promotion thành cuộc thi
  viết lách và tự quảng bá, làm lợi cho người tự tin và ngôn ngữ tốt, làm hại người làm nhiều nói ít.
  Ứng viên nên đóng góp dữ liệu; manager phải chịu trách nhiệm về lập luận.
- **Calibration mà mọi người đã bàn với nhau từ trước.** Information cascade: ý kiến của người nói
  đầu tiên hoặc người có vị thế cao nhất neo toàn bộ phòng họp. Cơ chế chặn duy nhất hoạt động là
  viết đánh giá độc lập trước khi thảo luận.
- **Định nghĩa bậc chỉ manager được đọc.** Nếu người ta không biết tiêu chí, họ không thể tự điều
  chỉnh, và mọi quyết định promotion trông như tuỳ tiện — kể cả khi nó không tuỳ tiện.
- **Tạo title mới để giữ người thay vì mở rộng phạm vi thật.** "Senior II", "Lead Engineer" không có
  định nghĩa, "Technical Manager" không quản ai. Người nhận biết nó rỗng, đồng nghiệp cũng biết, và
  nó làm giảm giá trị của mọi title khác trong tổ chức.

### Khi nào KHÔNG nên áp dụng

- **Công ty dưới 10 engineer.** Ở quy mô này, một ladder sáu bậc với calibration liên team là chi phí
  thuần và tạo ra bureaucracy mà không giải quyết vấn đề gì — mọi người đều nhìn thấy công việc của
  nhau. Cơ chế đủ: một trang mô tả ba mức phạm vi, và một cuộc trò chuyện trung thực mỗi nửa năm.
  Nhưng vẫn phải giữ hai nguyên tắc: promote để xác nhận, và không để kết quả là bất ngờ.
- **Trong giai đoạn tổ chức đang tái cấu trúc hoặc cắt giảm.** Chạy chu kỳ promotion bình thường
  trong lúc đóng băng ngân sách sẽ dẫn tới việc dùng tiêu chí năng lực để che ràng buộc tài chính —
  chính xác điều bị cấm ở trên. Việc trung thực hơn: nói rõ chu kỳ này bị hoãn vì lý do gì, cam kết
  ngày xem lại, và vẫn ghi nhận bằng chứng để không mất dữ liệu.
- **Với vai trò mà tổ chức chưa từng có ai đảm nhiệm.** Khi bạn promote người đầu tiên lên Staff hoặc
  lên Director, không có tham chiếu nội bộ để calibration. Ở đây, thay vì giả vờ có quy trình chuẩn,
  hãy làm rõ đây là quyết định có bất định cao, dùng tham chiếu bên ngoài (định nghĩa công khai của
  các công ty khác), và thiết kế điểm kiểm tra sau 6 tháng với sự đồng thuận của chính người đó.
- **Khi bạn mới nhận team dưới ba tháng.** Bạn chưa có đủ quan sát trực tiếp để viết một hồ sơ dựa
  trên bằng chứng, và dựa hoàn toàn vào đánh giá của người tiền nhiệm là chuyển giao cả bias của họ.
  Trung thực hơn: nói với ứng viên rằng bạn cần một quý để quan sát, và nói rõ bạn sẽ quan sát gì.
- **Khi mục tiêu thật là giữ người chứ không phải xác nhận năng lực.** Trong trường hợp đó, promotion
  là công cụ sai. Công cụ đúng là điều chỉnh lương, mở rộng phạm vi công việc, hoặc thay đổi điều
  kiện làm việc — và một cuộc trò chuyện thẳng thắn về điều họ thực sự muốn (xem chủ đề 9).

---

## 8. Xử lý underperformance

### Problem Statement

Nam đã ở dưới kỳ vọng trong bốn quý. Mọi người trong team đều biết. Tuấn — Tech Lead — biết. Hà — EM
— biết. Không ai từng nói với Nam bằng những từ mà anh không thể hiểu nhầm.

Cái đã diễn ra thay vào đó: các task khó dần được giao cho người khác. Code của Nam được review kỹ
hơn nhưng không ai giải thích vì sao. Trong sprint planning, anh nhận ít điểm hơn. Trong 1-1, Tuấn
nói "cũng ổn, em cứ tiếp tục". Nam cảm nhận có gì đó không đúng nhưng không có dữ liệu, nên anh giải
thích nó bằng cách dễ chịu nhất: có lẽ do team đang bận.

Đến quý thứ năm, Hà bắt đầu quy trình chấm dứt. Với Nam, đây là một cú sốc hoàn toàn. Với team, đây
là bằng chứng rằng đánh giá ở đây không minh bạch — nếu Nam bị cho nghỉ mà không có cảnh báo nào,
điều đó cũng có thể xảy ra với họ. Với Hà, đây là một quý mất vào quy trình HR, một ghế trống, và
một team mất niềm tin.

Điều đáng chú ý: **thiệt hại lớn nhất không đến từ việc Nam làm dưới kỳ vọng. Nó đến từ bốn quý không
ai nói.** Chi phí của việc trì hoãn, tính được:

- Người xung quanh gánh phần việc khó, và họ biết mình đang gánh. Đây là một trong những nguyên nhân
  hàng đầu khiến người giỏi rời đi — không phải vì công việc nặng, mà vì họ thấy sự bất công không
  được xử lý.
- Chuẩn của team hạ xuống. Nếu mức đó chấp nhận được với Nam, nó chấp nhận được với mọi người. Đây là
  cơ chế lan truyền, không phải một sự kiện đơn lẻ.
- Uy tín của lead giảm. Team đọc sự im lặng là hoặc lead không nhận ra, hoặc lead không dám xử lý.
  Cả hai cách đọc đều làm giảm khả năng lead dẫn dắt trong những việc khác.
- **Và chính Nam mất bốn quý** ở một nơi anh không thành công, không nhận được thông tin để sửa, và
  không có cơ hội tìm chỗ phù hợp hơn. Đây là phần chi phí hay bị bỏ qua khi người ta biện minh cho
  sự trì hoãn bằng lòng tốt.

### First Principles

**Underperformance gần như luôn có nhiều nguyên nhân góp phần, và can thiệp sai nguyên nhân sẽ thất
bại.** Đây là nguyên lý quan trọng nhất của chủ đề này. Phản xạ mặc định — "người này không đủ giỏi"
— là một trong năm giả thuyết, và thường không phải giả thuyết đúng.

| Nguyên nhân | Câu hỏi phân biệt | Can thiệp đúng | Can thiệp sai (và vì sao hỏng) |
|---|---|---|---|
| **Kỳ vọng không rõ** | Nếu tôi hỏi người này "công việc tốt ở vai trò của bạn trông như thế nào", câu trả lời có khớp với của tôi không? | Viết kỳ vọng ra, cụ thể, có ví dụ. Kiểm tra lại sau 2 tuần | Đào tạo kỹ năng — vô ích vì họ không thiếu kỹ năng, họ đang nhắm sai đích |
| **Thiếu năng lực** | Họ có làm được việc này khi có người ngồi cạnh không? Đây là kỹ năng học được trong 1–2 quý hay là năng lực nền? | Mentoring có mục tiêu, thu hẹp phạm vi, pairing, bài tập vừa quá tầm | Gây áp lực — làm giảm thêm hiệu suất; hoặc chuyển sang việc khó hơn để "thử thách" |
| **Sai vị trí** | Họ có mạnh rõ rệt ở một loại việc khác không? Vấn đề xuất hiện ở mọi loại việc hay chỉ một loại? | Đổi vai trò hoặc đổi team — đây là kết quả tốt, không phải thất bại | Cố ép họ giỏi thứ họ không hợp; hoặc chuyển đi mà không nói thật với team nhận |
| **Vấn đề cá nhân** | Hiệu suất giảm đột ngột từ một thời điểm cụ thể? Có thay đổi trong hành vi ngoài công việc? | Hỏi thẳng nhưng không xâm phạm, cho không gian và hỗ trợ có thời hạn, phối hợp HR | Áp kế hoạch cải thiện — làm tăng áp lực đúng lúc người đó ít chịu được nhất |
| **Môi trường / quản lý** | Người này có từng làm tốt không? Có ai khác trong cùng điều kiện cũng gặp vấn đề không? | Sửa điều kiện: khối lượng, sự rõ ràng, công cụ, xung đột trong team, chính cách quản lý của bạn | Quy về cá nhân — bạn sẽ thay người và gặp lại đúng vấn đề với người tiếp theo |

Bài kiểm tra chẩn đoán mạnh nhất: **người này có từng làm tốt ở đâu đó không?** Nếu có, giả thuyết
"thiếu năng lực nền" yếu đi rõ rệt và bạn nên tìm ở bốn nguyên nhân còn lại trước.

**Vì sao việc nói ra bị trì hoãn — cơ chế, không phải tính cách.** Người quản lý trì hoãn không phải
vì họ yếu đuối. Họ trì hoãn vì cấu trúc phần thưởng: chi phí của cuộc trò chuyện là **tức thì, cụ
thể, và thuộc về bạn** (khó chịu, rủi ro phản ứng tiêu cực, có thể mất quan hệ). Lợi ích thì **trễ,
phân tán, và thuộc về người khác** (team đỡ gánh sau vài tháng, người đó có cơ hội sửa). Đây là bài
toán chiết khấu thời gian kinh điển, và nó lý giải vì sao ngay cả những manager giỏi cũng trì hoãn.

Ở bối cảnh Việt Nam, cơ chế này được khuếch đại bởi hai yếu tố: chi phí "mất mặt" khiến cả người nói
và người nghe đều muốn tránh, và thói quen nói giảm khiến thông điệp bị làm mềm tới mức mất nội dung.
Một câu như "em cố gắng thêm chút nữa nhé" được người nói coi là đã cảnh báo, và được người nghe coi
là lời động viên bình thường. Cả hai đều thành thật, và khoảng cách giữa hai cách hiểu là bốn quý.

Cách chặn duy nhất hoạt động: **có một cơ chế bắt buộc**, không phụ thuộc vào can đảm của từng người.
Ví dụ, một mục cố định trong mẫu 1-1 hàng quý: "so với kỳ vọng của bậc hiện tại, bạn đang ở đâu?" —
câu hỏi này buộc cuộc trò chuyện xảy ra kể cả khi cả hai bên đều muốn tránh.

### Mental Model

**Thang leo có bốn nấc, và mỗi nấc phải rõ ràng hơn nấc trước:**

```
Nấc 1 — TÍN HIỆU (trong 1-1 thường kỳ, tuần đầu tiên bạn nhận ra)
  "Anh nhận thấy [quan sát cụ thể]. Anh muốn hiểu chuyện gì đang xảy ra."
  Mục tiêu: chẩn đoán. Chưa phải cảnh báo.
  Nếu bỏ qua nấc này → mọi nấc sau đều trở thành bất ngờ.

Nấc 2 — NÊU RÕ KHOẢNG CÁCH (trong 2–4 tuần, nếu nấc 1 không đổi)
  "Khoảng cách là [X]. Kỳ vọng ở bậc này là [Y]. Đây là điều cần thay đổi."
  Mục tiêu: đảm bảo không thể hiểu nhầm. Có ghi chép.

Nấc 3 — KẾ HOẠCH CẢI THIỆN CÓ THỜI HẠN (nếu nấc 2 không đổi sau 4–6 tuần)
  Tiêu chí kiểm được, mốc thời gian, hỗ trợ cụ thể, hệ quả nếu không đạt.
  Mục tiêu: cho cơ hội thật với điều kiện rõ ràng.

Nấc 4 — QUYẾT ĐỊNH (đổi vai trò hoặc chấm dứt)
  Mục tiêu: kết thúc tình trạng không bền vững cho cả hai bên.
```

Quy tắc: **không được nhảy nấc.** Nhảy từ nấc 1 sang nấc 3 là điều làm cho kế hoạch cải thiện bị đọc
như thủ tục sa thải — và một khi nó bị đọc như vậy, người đó sẽ dành thời gian tìm việc thay vì cải
thiện, và bạn đã tự tạo ra kết quả mình muốn tránh.

**Phân biệt hai loại vấn đề, vì cách xử lý khác nhau hoàn toàn:**

| | Vấn đề năng lực (can't) | Vấn đề hành vi / cam kết (won't) |
|---|---|---|
| Biểu hiện | Cố gắng thật, kết quả không đạt | Không cố gắng, hoặc hành vi gây hại cho người khác |
| Thời gian cho phép | Rộng hơn — học cần thời gian | Hẹp hơn — hành vi đổi được nhanh nếu người đó muốn |
| Can thiệp | Mentoring, thu hẹp phạm vi, pairing | Nêu rõ ranh giới và hệ quả |
| Thái độ của lead | Kiên nhẫn, đầu tư | Rõ ràng, không thương lượng về chuẩn |
| Nguy hiểm nếu nhầm | Coi vấn đề năng lực là thái độ → bất công và tàn nhẫn | Coi vấn đề hành vi là năng lực → đầu tư mentoring vô ích và team mất niềm tin |

Trường hợp đặc biệt cần nêu riêng: **người giỏi kỹ thuật nhưng gây hại cho môi trường xung quanh**
(hạ thấp người khác trong review, không hợp tác, tạo bus factor bằng cách giữ kiến thức). Đây là loại
underperformance mà tổ chức hay bỏ qua nhất vì đầu ra cá nhân nhìn có vẻ tốt. Nhưng tính theo output
hệ thống (xem `00-nen-tang-leadership.md`), người này thường có đóng góp ròng âm: họ tạo ra x đơn vị
giá trị và làm giảm 2x của những người xung quanh. Dung thứ hành vi này vì lý do kỹ thuật là cách phá
văn hoá nhanh nhất, vì nó phát tín hiệu rằng chuẩn hành vi có thể mua được bằng năng lực.

### Practical Framework

**Cuộc trò chuyện nấc 2 — nêu rõ khoảng cách.** Đây là cuộc trò chuyện quyết định. Làm đúng, nó mở
ra khả năng cải thiện thật; làm sai, nó hoặc không truyền được thông điệp, hoặc phá quan hệ.

*Phiên bản nói vòng vo (rất phổ biến, và vô dụng):*

> **Tuấn:** Em dạo này thế nào? Ổn không?
> **Nam:** Dạ cũng bình thường ạ.
> **Tuấn:** Ừ... anh thấy sprint vừa rồi cũng hơi chậm một chút. Chắc do task khó.
> **Nam:** Dạ, cái phần integration nó nhiều case quá ạ.
> **Tuấn:** Ừ anh hiểu. Thôi em cứ cố gắng thêm nhé. Có gì khó cứ hỏi anh.
> **Nam:** Dạ vâng em cảm ơn anh.

Tuấn tin rằng anh đã cảnh báo. Nam nghe được: sếp thông cảm, task khó, không có gì nghiêm trọng. Cả
hai rời cuộc họp với hai bản ghi khác nhau về chuyện vừa xảy ra. Bốn quý sau, khi Nam nói "chưa ai
nói với em điều này", anh không nói dối.

*Phiên bản nói rõ:*

> **Tuấn:** Nam, hôm nay anh muốn nói một chuyện khó, và anh muốn nói rõ để em không phải đoán. Anh
> sẽ nói quan sát của anh trước, rồi anh muốn nghe em.
>
> Ba sprint gần đây, các task em nhận đều vượt ước lượng từ hai đến ba lần: task API voucher ước lượng
> 3 ngày, mất 8; task migration ước lượng 2 ngày, mất 6. Và trong cả ba lần, anh chỉ biết là chậm khi
> đã quá hạn, không phải trước đó. Ở mức Mid-level, kỳ vọng là em tự phát hiện mình đang bị chặn
> trong vòng một ngày và nói ra. Đây là khoảng cách anh thấy — không phải về tốc độ, mà về việc phát
> hiện và nói sớm.
>
> Anh nói điều này không phải để trách. Anh nói vì anh cần em biết chính xác chỗ nào cần thay đổi.
> Bây giờ anh muốn nghe: từ phía em, chuyện gì đang diễn ra?
>
> **Nam:** *(im lặng một lúc)* Thật ra... em bị kẹt ở phần integration khá lâu. Em không muốn nói vì
> sợ anh nghĩ em không làm được.
>
> **Tuấn:** Cảm ơn em đã nói thẳng. Vậy vấn đề không phải là em làm chậm — mà là em đã học được rằng
> nói ra thì rủi ro hơn im lặng. Anh cần sửa phần đó, vì đó là chuyện của anh. Còn phần của em: từ
> giờ, quy tắc là nếu em kẹt quá bốn tiếng ở cùng một chỗ, em nhắn anh. Không phải xin phép, chỉ là
> báo. Anh sẽ không hỏi vì sao. Hai tuần nữa mình xem lại xem cách này có chạy không.

Bảng mổ xẻ khác biệt:

| Yếu tố | Phiên bản vòng vo | Phiên bản rõ |
|---|---|---|
| Dữ liệu | "hơi chậm một chút" | Ba sự việc, có số, có ngày |
| Kỳ vọng | Không nêu | Nêu rõ kỳ vọng của bậc, và tách khỏi tốc độ |
| Ai nói phần lớn | Tuấn nói, Nam đồng ý | Tuấn nêu quan sát rồi nhường lời — và điều quan trọng nhất xuất hiện từ phía Nam |
| Chẩn đoán | Không có, giả định là "task khó" | Phát hiện nguyên nhân thật: Nam không dám nói mình kẹt |
| Ai chịu phần nào | Ngầm định lỗi thuộc về Nam | Tuấn nhận phần của mình (môi trường) và giao phần của Nam (hành vi báo sớm) |
| Đầu ra | Không có | Một quy tắc cụ thể, một mốc kiểm tra |

Điểm quan trọng nhất trong ví dụ này: nguyên nhân thật hoá ra thuộc nhóm "môi trường / quản lý", chứ
không phải "thiếu năng lực" — và nó chỉ lộ ra vì Tuấn để khoảng lặng sau khi nêu quan sát. Nếu anh
nhảy thẳng sang kế hoạch cải thiện, anh sẽ can thiệp sai nguyên nhân và thất bại.

**Kế hoạch cải thiện (nấc 3).** Nguyên tắc: đây phải là **cơ hội thật**, không phải thủ tục. Nếu bạn
đã quyết định trong đầu rằng người này sẽ ra đi, việc chạy một kế hoạch giả là không trung thực với
họ và tốn thời gian của cả hai.

```
KẾ HOẠCH CẢI THIỆN · [Tên] · [Vai trò]
Người viết: [Manager] · Ngày bắt đầu: [ngày] · Thời hạn: 8 tuần
Điểm kiểm tra: tuần 2, tuần 4, tuần 6, tuần 8 (đã đặt lịch)
Đã phối hợp với: HR ([tên]) — mọi bước thực thi tuân theo quy định
                 nội bộ và pháp luật lao động hiện hành

1. KHOẢNG CÁCH CỤ THỂ (mỗi mục có bằng chứng, không có tính từ)
   1.1 [Mô tả bằng hành vi quan sát được] · Bằng chứng: [3 sự việc có ngày]
   1.2 ...

2. TIÊU CHÍ ĐẠT — kiểm được, không cần diễn giải
   2.1 Trong 8 tuần, mọi task vượt ước lượng > 50% đều được báo trước
       khi quá hạn. Đo bằng: [cách đo cụ thể]
   2.2 ...
   [Nếu bạn không viết được tiêu chí kiểm được, khoảng cách chưa được
    chẩn đoán đủ rõ — quay lại bước 1, đừng bắt đầu kế hoạch]

3. HỖ TRỢ TỪ TỔ CHỨC (bắt buộc có — kế hoạch một chiều là thủ tục, không phải cơ hội)
   3.1 Pairing 4 giờ/tuần với [tên], đã thống nhất với người đó
   3.2 Phạm vi task được thu hẹp trong 4 tuần đầu để tạo điều kiện
   3.3 1-1 tăng lên hàng tuần thay vì hai tuần một lần

4. HỆ QUẢ NẾU KHÔNG ĐẠT — nói thẳng, không úp mở
   [Nêu rõ. Người nhận có quyền biết chính xác điều gì đang được đặt cược.]

5. ĐIỀU MANAGER CAM KẾT
   - Feedback trong vòng 48 giờ sau mỗi sự việc liên quan, không tích luỹ
     tới điểm kiểm tra.
   - Không thay đổi tiêu chí giữa chừng.
   - Nếu tôi thấy kế hoạch này không còn phù hợp (ví dụ chẩn đoán ban đầu
     sai), tôi sẽ nói ngay thay vì để nó chạy hết thời hạn.

Xác nhận đã đọc và hiểu: [chữ ký/ghi nhận của cả hai bên]
```

**Khi nào chuyển vai trò thay vì chấm dứt.** Nếu chẩn đoán là "sai vị trí" và người đó có điểm mạnh
rõ ràng ở loại việc khác, chuyển team hoặc đổi vai trò là kết quả tốt nhất cho tất cả. Hai điều kiện
bắt buộc, không được bỏ:

1. **Nói thật với team nhận.** Chuyển một người có vấn đề sang team khác mà không nói (anti-pattern
   "passing the trash") là hành vi phá hoại lòng tin giữa các lead, và nó sẽ quay lại với bạn — lần
   sau không ai nhận người từ team bạn.
2. **Nói thật với người đó** rằng đây là một cơ hội thứ hai có điều kiện, không phải một sự sắp xếp
   trung tính.

**Chấm dứt tôn trọng và nói với team.** Phần thực thi (thủ tục, thời hạn báo trước, trợ cấp, giấy tờ)
**phải do HR và bộ phận pháp chế dẫn dắt** — luật lao động có yêu cầu cụ thể về căn cứ và trình tự,
và một lead làm sai bước ở đây tạo rủi ro pháp lý thật cho tổ chức. Phần thuộc về lead:

- Cuộc trò chuyện ngắn, rõ, không tranh luận lại về bằng chứng (thời điểm tranh luận đã qua), và
  không kèm lời an ủi làm mờ thông điệp.
- Giữ phẩm giá của người đó: thời gian bàn giao đàng hoàng, lời cảm ơn cho phần họ đã đóng góp là
  thật, hỗ trợ trong khả năng cho việc tìm chỗ tiếp theo nếu bạn thật lòng làm được.
- **Với team**: nói ngắn gọn rằng người đó không còn làm việc ở đây, cảm ơn đóng góp, và nêu kế hoạch
  phân bổ công việc. **Không** kể chi tiết lý do — vừa là vấn đề riêng tư của người đã đi, vừa vì
  team đọc cách bạn nói về người vắng mặt như tín hiệu về cách bạn sẽ nói về họ. Nếu có ai hỏi riêng,
  câu trả lời trung thực và đủ: "Đây là chuyện giữa công ty và bạn ấy, anh không chia sẻ chi tiết —
  nhưng anh khẳng định đây không phải một quyết định đột ngột, và bạn ấy đã biết trước từ lâu."

### Trade-off

| Trục | Nghiêng về A khi | Nghiêng về B khi | Cái giá của mỗi bên |
|---|---|---|---|
| **A: Nói sớm khi chưa chắc** vs **B: Chờ tới khi có đủ bằng chứng** | Bạn có ít nhất hai sự việc cụ thể; tác động đang lan sang người khác | Chỉ mới một sự việc, có thể là ngoại lệ; bạn mới nhận team | A: rủi ro nói sai và làm giảm động lực của người đang ổn. B: mỗi tuần chờ là một tuần người đó không có cơ hội sửa, và team gánh thêm |
| **A: Đầu tư cải thiện** vs **B: Chuyển vai trò / chấm dứt sớm** | Chẩn đoán là kỳ vọng không rõ, môi trường, hoặc kỹ năng học được trong 1–2 quý; người đó có động lực | Đã qua hai chu kỳ can thiệp không đổi; hoặc vấn đề là hành vi gây hại | A: tốn thời gian quản lý đáng kể (thực tế 4–8 giờ/tuần), và nếu thất bại thì đã mất thêm một quý. B: mất một người có thể đã cứu được, và team đọc là tổ chức không kiên nhẫn |
| **A: Giữ chuẩn** vs **B: Hạ kỳ vọng cho phù hợp thực tế** | Chuẩn đó cần thiết cho chất lượng hoặc an toàn hệ thống | Chuẩn được đặt ra tuỳ tiện; hoặc thị trường không cung cấp được mức đó ở mức lương này | A: có thể mất người trong thị trường khan hiếm. B: chuẩn hạ một lần sẽ khó nâng lại, và nó áp cho tất cả chứ không riêng một người |
| **A: Minh bạch với team về tình hình** vs **B: Giữ riêng tư tuyệt đối** | Team đang gánh và họ có quyền biết bạn đã nhận ra | Luôn ưu tiên B về **chi tiết**; A chỉ áp dụng cho **thông điệp rằng vấn đề đang được xử lý** | A quá mức: xâm phạm riêng tư và làm người đó mất mặt trước khi có kết luận. B quá mức: team tin rằng lead không nhìn thấy gì |

Trade-off khó nhất: **kiên nhẫn với người này là bất công với người khác.** Đây là trade-off thật và
không có cách né. Mỗi tuần bạn giữ tình trạng chưa giải quyết, một hoặc hai người khác trong team
đang làm thêm phần việc mà họ không được ghi nhận. Cách xử lý duy nhất trung thực: đặt một thời hạn
rõ cho quá trình can thiệp (không phải một thời hạn mơ hồ), và với những người đang gánh, nói rằng
bạn biết họ đang gánh và bạn đang xử lý — không cần nói chi tiết.

### Real-world Scenarios

**Tình huống A — Senior giỏi kỹ thuật nhưng phá môi trường (fintech, team 11 người).**

*Bối cảnh.* Sơn là người hiểu hệ thống đối soát sâu nhất công ty. Anh cũng là người mà hai junior đã
xin đổi team để tránh, và là người mà trong ba tháng qua có bốn comment review bị người khác chuyển
cho Hà xem vì "không biết trả lời thế nào". Sơn không sai về kỹ thuật trong bất kỳ comment nào.

*Các lựa chọn.* (1) Bỏ qua — anh là người duy nhất hiểu hệ thống đối soát, và fintech không đùa được
với đối soát. (2) Tách anh khỏi việc review code người khác, để anh làm việc một mình. (3) Nêu vấn đề
như một vấn đề hiệu suất, có tiêu chí và thời hạn. (4) Chấm dứt.

*Trade-off.* Lựa chọn 1 có sức hấp dẫn thật vì rủi ro kỹ thuật là có thật — nhưng nó phát tín hiệu
rằng chuẩn hành vi mua được bằng năng lực, và tín hiệu đó áp cho toàn bộ tổ chức, không riêng team
này. Lựa chọn 2 giải quyết triệu chứng và tạo ra bus factor bằng 1 ở đúng chỗ nguy hiểm nhất — nó
biến một vấn đề con người thành một rủi ro hệ thống. Lựa chọn 4 vào lúc này là bỏ qua nấc 1 và 2;
Sơn chưa từng được nói rõ rằng đây là vấn đề nghiêm trọng.

*Quyết định.* Hà chọn 3, với hai đặc điểm quan trọng. Thứ nhất, cô nêu tác động bằng dữ liệu chứ
không bằng cảm nhận: hai người đã xin đổi team, và cô nói thẳng con số này với Sơn. Thứ hai, cô tách
rõ hai thứ trong cuộc trò chuyện: "Nội dung kỹ thuật của em đúng, anh không yêu cầu em hạ chuẩn. Cách
em nói làm hai người rời team. Anh yêu cầu em đổi cách nói, không đổi chuẩn." Song song, cô khởi động
việc giảm bus factor của hệ thống đối soát — không phải như một biện pháp trừng phạt mà như một việc
lẽ ra phải làm từ lâu, và cô nói rõ điều đó với Sơn để anh không đọc nó thành sự chuẩn bị cho việc
loại bỏ anh.

*Hậu quả.* Sơn phản ứng phòng vệ trong hai tuần đầu, sau đó thay đổi rõ rệt khi Hà cho anh một công
cụ cụ thể: phân tầng comment thành blocking / nên sửa / nit / hỏi để học (xem
`05-technical-leadership.md`), và yêu cầu mọi comment blocking phải nêu failure mode cụ thể. Sáu
tháng sau, Sơn trở thành người mentor cho một trong hai junior đã từng tránh anh. Bus factor của hệ
thống đối soát giảm từ 1 xuống 3.

*Bài học.* (a) Vấn đề hành vi thường đổi được nhanh hơn vấn đề năng lực — nếu người đó được nói rõ và
được cho một công cụ cụ thể thay vì một lời khuyên trừu tượng như "em nói nhẹ nhàng hơn". (b) Việc
tách "nội dung đúng" khỏi "cách nói sai" là điều khiến Sơn nghe được thay vì phòng vệ hoàn toàn. (c)
Đừng để rủi ro bus factor trở thành lý do dung thứ hành vi — hãy giải quyết cả hai, và nói rõ rằng
bạn đang giải quyết cả hai.

**Tình huống B — cùng một người dưới kỳ vọng, ba góc nhìn.**

Sự việc: Duy (Mid-level) đã ba sprint liên tiếp không hoàn thành cam kết.

*Nhìn từ IC đồng nghiệp (Vy).* "Tôi phải nhận lại hai task của Duy trong sprint vừa rồi, và không ai
nói gì về chuyện đó. Tôi không biết đây là tình huống tạm thời hay là bình thường mới. Nếu là bình
thường mới, tôi cần biết, vì nó ảnh hưởng tới việc tôi có nhận thêm việc gì không." Vy không muốn
Duy bị xử lý — cô muốn **biết rằng có ai đó đang xử lý**. Sự im lặng của lead, chứ không phải hiệu
suất của Duy, là thứ đang làm cô mất niềm tin.

*Nhìn từ Tech Lead (Tuấn).* "Tôi thấy Duy chậm nhưng tôi không chắc là do năng lực hay do tôi giao
task không rõ. Hai task đó đúng là mô tả sơ sài. Nếu tôi nêu vấn đề mà hoá ra lỗi ở tôi thì sao?"
Sự do dự của Tuấn hợp lý về mặt chẩn đoán — nhưng anh đang dùng sự không chắc chắn như lý do để không
làm gì, trong khi hành động đúng ở nấc 1 chính là **hỏi để chẩn đoán**, không phải kết luận. Cuộc trò
chuyện nấc 1 không đòi hỏi anh phải chắc chắn.

*Nhìn từ Manager (Hà).* Ở tầng này, ba sprint mà không có cuộc trò chuyện nào là **lỗi của cơ chế
feedback, không phải của Duy hay Tuấn**. Can thiệp của Hà: (a) coach Tuấn về cuộc trò chuyện nấc 1 —
cụ thể là dạy anh rằng nêu quan sát không đồng nghĩa với kết tội, và anh được phép nói "anh chưa biết
nguyên nhân"; (b) thêm vào mẫu 1-1 một câu hỏi bắt buộc để cuộc trò chuyện không phụ thuộc vào can
đảm của từng người; (c) với Vy, nói ngắn gọn rằng cô đã nhận thấy và đang xử lý, không nói chi tiết.

Điểm mấu chốt: cùng ba sprint trễ, đồng nghiệp đọc thành sự bất công không được xử lý, Tech Lead đọc
thành sự không chắc chắn về chẩn đoán, Manager đọc thành thiếu cơ chế buộc feedback xảy ra. Can thiệp
ở tầng thứ ba là cái duy nhất ngăn được lần sau.

### Best Practices

- **Chẩn đoán trước khi can thiệp.** Chạy qua năm nguyên nhân, và bắt đầu bằng câu hỏi "người này có
  từng làm tốt ở đâu đó không". Can thiệp sai nguyên nhân không chỉ vô ích, nó làm tình hình xấu đi.
- **Nói ở nấc 1 trong vòng một đến hai tuần từ khi bạn nhận ra.** Cuộc trò chuyện sớm là cuộc trò
  chuyện dễ nhất bạn sẽ có; mỗi tuần trì hoãn làm nó khó hơn và làm bằng chứng cần nhiều hơn.
- **Không nhảy nấc.** Kế hoạch cải thiện xuất hiện mà chưa từng có cảnh báo rõ ràng sẽ bị đọc là thủ
  tục sa thải, và khi đó nó không còn là cơ hội thật.
- **Dùng hành vi quan sát được và sự việc có ngày, không dùng tính từ.** "Em thiếu chủ động" không
  hành động được. "Trong ba lần task quá hạn, cả ba lần anh biết sau khi đã quá hạn" thì có.
- **Nêu quan sát rồi im lặng.** Nguyên nhân thật thường xuất hiện trong khoảng lặng, không xuất hiện
  trong phần bạn nói.
- **Kế hoạch cải thiện phải có phần hỗ trợ từ tổ chức và phần cam kết của manager.** Kế hoạch một
  chiều là thủ tục, không phải cơ hội.
- **Ghi chép mọi cuộc trò chuyện** — không phải để xây hồ sơ chống lại ai, mà vì trí nhớ của cả hai
  bên sẽ khác nhau sau ba tháng, và vì HR sẽ cần nó nếu đi tới bước cuối.
- **Với hành vi gây hại cho người khác, thời gian phải ngắn hơn.** Chuẩn hành vi không thương lượng
  được bằng năng lực kỹ thuật; dung thứ một lần là đặt lại chuẩn cho cả tổ chức.
- **Với bước chấm dứt, để HR và pháp chế dẫn phần thực thi.** Đây không phải chỗ để ứng biến.
- **Sau khi kết thúc, hỏi mình một câu**: mình có thể phát hiện sớm hơn bao nhiêu tháng, và cái gì
  trong hệ thống đã ngăn điều đó? Câu trả lời thường chỉ về quy trình 1-1 hoặc về sự rõ ràng của kỳ
  vọng, chứ không về cá nhân nào.

### Anti-patterns

- **Hy vọng tự tốt lên.** Cơ chế phá hoại: không có tín hiệu hiệu chỉnh thì hành vi hiện tại được
  người đó hiểu là chấp nhận được, nên nó sẽ tiếp tục — và mỗi tháng trôi qua làm cho việc nêu vấn đề
  càng khó biện minh ("sao bây giờ mới nói?"). Dấu hiệu sớm: bạn đã nghĩ về việc này ba lần trong
  một tháng nhưng chưa nói câu nào.
- **Giao ít việc quan trọng dần mà không nói.** Đây là cách trừng phạt thụ động phổ biến nhất và cũng
  tàn nhẫn nhất, vì người đó cảm nhận được sự thay đổi nhưng không có thông tin để hành động. Nó cũng
  làm hỏng khả năng cải thiện: bạn vừa lấy đi cơ hội duy nhất để họ chứng minh điều ngược lại.
- **Chuyển người sang team khác mà không nói thật.** Ngoài việc phá lòng tin giữa các lead, nó gây
  hại cho chính người được chuyển: họ bước vào một team không biết cách hỗ trợ họ, thất bại lần hai,
  và bây giờ có hai bản ghi thất bại thay vì một.
- **PIP như thủ tục hợp thức hoá.** Nếu quyết định đã được ra trước khi kế hoạch bắt đầu, mọi người
  đều biết — người trong cuộc biết, team biết. Chi phí: bạn mất khả năng dùng kế hoạch cải thiện như
  một công cụ thật với bất kỳ ai khác trong tương lai, vì nó đã được đọc là án tử có thời hạn.
- **Nêu vấn đề lần đầu tiên trong performance review.** Review là nơi tổng kết những gì đã được nói,
  không phải nơi tiết lộ. Xem chủ đề 6.
- **Xử lý bằng email hoặc chat.** Cuộc trò chuyện nấc 1 và 2 phải là trực tiếp — văn bản không có
  khoảng lặng, không đọc được phản ứng, và gần như luôn bị đọc nặng hơn ý định. Văn bản là để **ghi
  lại sau** cuộc trò chuyện, không phải để thay thế nó.
- **Dung thứ hành vi gây hại vì lý do kỹ thuật.** Đã phân tích ở trên. Dấu hiệu sớm: bạn thấy mình
  đang dùng câu "nhưng bạn ấy là người duy nhất biết..." như một lý lẽ.

### Khi nào KHÔNG nên áp dụng

- **Khi bạn mới nhận team dưới hai tháng.** Bạn chưa phân biệt được vấn đề cá nhân với vấn đề hệ
  thống mà bạn vừa thừa kế. Chạy quy trình can thiệp lúc này có xác suất cao là can thiệp sai nguyên
  nhân. Việc đúng: quan sát, hỏi nhiều, và bắt đầu từ việc làm rõ kỳ vọng cho **cả team** — thường
  điều này một mình đã giải quyết một phần đáng kể các trường hợp trông như underperformance.
- **Khi nguyên nhân là vấn đề cá nhân hoặc sức khoẻ.** Áp một kế hoạch cải thiện có thời hạn lên
  người đang gặp khủng hoảng gia đình hoặc vấn đề sức khoẻ là can thiệp sai nguyên nhân, và nó làm
  tình hình xấu đi ở đúng lúc người đó ít chịu đựng được nhất. Việc đúng: hỏi thẳng nhưng không xâm
  phạm, phối hợp với HR về các hình thức hỗ trợ mà tổ chức có (nghỉ phép, giảm tải có thời hạn), và
  đặt lại mốc xem xét thay vì chạy đồng hồ. Nếu có dấu hiệu về sức khoẻ tinh thần, việc của bạn là
  tạo điều kiện để người đó tiếp cận hỗ trợ chuyên môn, không phải tự xử lý.
- **Khi vấn đề là do khối lượng công việc hoặc điều kiện chung của cả team.** Nếu ba trong tám người
  đang dưới kỳ vọng, đó không phải ba vấn đề cá nhân — đó là một vấn đề hệ thống, và can thiệp cá
  nhân sẽ vừa vô ích vừa bất công. Kiểm tra trước: tỉ lệ người dưới kỳ vọng trong team là bao nhiêu?
- **Khi chính bạn là nguyên nhân góp phần chính.** Nếu task giao không rõ, ưu tiên đổi liên tục, hoặc
  bạn không có mặt khi cần — hãy sửa phần của bạn trước và cho ít nhất một chu kỳ để quan sát lại.
  Chạy quy trình underperformance trong tình huống này là chuyển chi phí của lỗi quản lý sang một cá
  nhân.
- **Trong giai đoạn khủng hoảng ngắn hạn của tổ chức** (incident kéo dài, tái cấu trúc, mất khách
  hàng lớn). Hiệu suất của mọi người đều bị nhiễu, dữ liệu không đáng tin, và người nhận sẽ đọc quy
  trình này qua lăng kính sợ hãi chung. Hoãn tới khi có mặt bằng ổn định, nhưng ghi lại quan sát để
  không mất dữ liệu.

---

## 9. Retention và mất người

### Problem Statement

Quân nộp đơn vào một sáng thứ Ba. Trong exit interview do chính manager anh thực hiện, lý do được ghi
là "cơ hội mới có mức lương tốt hơn". Công ty đưa counter-offer tăng 25%. Quân nhận, ở lại, và rời đi
bảy tháng sau.

Cả hai lần, tổ chức đều không biết chuyện gì thực sự xảy ra. Điều Quân không nói trong exit
interview: trong mười tháng qua, anh đã ba lần đề xuất tái cấu trúc module đối soát và cả ba lần bị
gác lại mà không có lý do; anh đã hỏi về lộ trình lên Staff hai lần và nhận được câu "để anh xem";
và anh đã ngừng đề xuất từ tháng thứ bảy. Lương là lý do dễ nói nhất, không gây khó xử cho ai, và
không đòi hỏi anh phải phê bình người đang ngồi đối diện.

Hậu quả khi tổ chức không đọc được tín hiệu thật:

- Counter-offer giải quyết một triệu chứng và để nguyên nguyên nhân, nên nó chỉ hoãn việc ra đi —
  thường vài tháng, với chi phí đã tăng.
- Cùng nguyên nhân đó tiếp tục tác động lên những người còn lại, và tổ chức sẽ gặp lại nó với người
  tiếp theo.
- Chỉ số turnover tổng gộp cả người mà bạn tiếc lẫn người mà việc họ rời đi là kết quả tốt, nên nó
  không nói được điều gì hữu ích và thường dẫn tới can thiệp sai.
- Tri thức đi cùng người. Với một Senior đã ở ba năm, phần tri thức không có trong tài liệu — vì sao
  hệ thống được thiết kế như vậy, cái gì đã thử và thất bại, ai cần được hỏi trước khi đổi cái gì —
  thường lớn hơn phần có trong tài liệu.

### First Principles

**Lý do nêu ra không phải lý do thật, và điều này không phải vì người ta nói dối.** Ba cơ chế:

1. *Chi phí xã hội của sự thật.* Nói "tôi đi vì manager không bao giờ phản hồi đề xuất của tôi" tạo
   xung đột trực tiếp với người có thể sẽ là người tham chiếu (reference) trong tương lai. Nói "lương"
   thì an toàn, đúng một phần, và kết thúc cuộc trò chuyện. Ở bối cảnh Việt Nam, nơi việc giữ quan hệ
   sau khi rời công ty được coi trọng và mạng lưới ngành hẹp, cơ chế này mạnh hơn.
2. *Người ta không tự biết đầy đủ.* Quyết định rời đi hiếm khi đến từ một sự kiện; nó là tích luỹ của
   nhiều tín hiệu nhỏ. Khi được hỏi, người ta tìm một lý do mạch lạc — và lý do mạch lạc nhất thường
   là lý do gần nhất về thời gian, không phải nguyên nhân lớn nhất.
3. *Người hỏi là một phần của vấn đề.* Nếu exit interview do manager trực tiếp thực hiện, và manager
   là một trong các nguyên nhân, dữ liệu thu được gần như vô giá trị.

Hệ quả thiết kế: **thông tin hữu ích phải được thu thập trước khi người ta quyết định đi, không phải
sau** — và không phải bởi người mà họ có thể đang có vấn đề.

**Hai loại turnover, và vì sao chỉ số tổng là vô nghĩa.**

| | Regretted | Non-regretted |
|---|---|---|
| Định nghĩa | Bạn sẽ tuyển lại người này ngay | Việc họ rời đi là kết quả tốt cho cả hai |
| Ý nghĩa của con số cao | Vấn đề nghiêm trọng, cần điều tra ngay | Có thể là dấu hiệu hệ thống hiring hoặc quản lý hiệu suất đang hoạt động |
| Can thiệp | Sửa nguyên nhân giữ người | Nhìn ngược lên quy trình tuyển và onboarding |

Một tổ chức có turnover 20% với 18% là non-regretted đang ở tình trạng rất khác một tổ chức có
turnover 12% với 11% là regretted. Chỉ số tổng gộp hai tình huống này lại thành một con số, và bất kỳ
quyết định nào dựa trên nó đều có xác suất cao là sai hướng.

**Chi phí thay thế một Senior — vì sao nó lớn hơn cảm nhận.** Bốn phần, chỉ phần đầu là hiển hiện:

- Chi phí tuyển: thời gian phỏng vấn của nhiều người (một vòng đầy đủ tốn khoảng 12–20 giờ công của
  đội ngũ — số minh hoạ), phí tuyển dụng nếu có, và thời gian ghế trống.
- Chi phí đưa tới năng suất: một Senior thường cần 3–6 tháng để đạt mức đóng góp ròng tương đương,
  và trong 1–2 tháng đầu họ tiêu thời gian của người khác.
- Mất tri thức không có trong tài liệu: lý do đằng sau các quyết định cũ, bản đồ các cạm bẫy trong
  hệ thống, quan hệ với các team khác.
- Tác động lên người ở lại: gánh nặng tăng, và — thường bị bỏ qua — một Senior rời đi làm những người
  còn lại đặt câu hỏi về lựa chọn của chính họ. Ảnh hưởng này lớn nhất khi người ra đi là người được
  tôn trọng nhất.

Cộng lại, chi phí thực tế thường tương đương 6–12 tháng lương của vị trí đó (số minh hoạ, thay đổi
mạnh theo mức độ chuyên biệt của hệ thống). Đây là con số nên được đặt cạnh chi phí của việc **không**
xử lý nguyên nhân giữ người — thường nhỏ hơn nhiều.

### Mental Model

**Rời đi là một hàm của ba biến, không phải một.** Một cách hình dung hữu ích: người ta ở lại khi
tổng của ba thứ đủ lớn, và rời đi khi một trong ba sụt đủ sâu để kéo tổng xuống dưới ngưỡng — ngưỡng
này chính là giá trị của lựa chọn thay thế mà thị trường đang chào.

```
Ở LẠI  ⟵  [ Công việc ]  +  [ Con người ]  +  [ Điều kiện ]   vs   [ Lựa chọn bên ngoài ]

Công việc:  có ý nghĩa không, có học được không, có tự chủ không,
            có thấy tác động của mình không
Con người:  quan hệ với manager, với đồng đội, cảm giác được tôn trọng
Điều kiện:  lương, giờ giấc, địa điểm, sự ổn định

Ba biến này KHÔNG thay thế được cho nhau quá một biên độ nhất định.
Tăng lương 30% bù được cho một công việc nhàm chán trong khoảng vài tháng,
không bù được lâu dài. Đây là lý do cơ chế của counter-offer thất bại.
```

Điểm quan trọng: **thị trường quyết định ngưỡng, còn bạn quyết định ba biến bên trái.** Khi thị trường
nóng (như giai đoạn 2021–2022 ở Việt Nam), ngưỡng tăng, và những vấn đề vốn chịu đựng được bỗng trở
thành lý do đi. Điều này có nghĩa: một tổ chức chỉ cạnh tranh bằng biến "Điều kiện" sẽ phải cạnh
tranh lại mỗi khi thị trường biến động, còn tổ chức mạnh ở hai biến kia có biên an toàn rộng hơn.

**Vì sao counter-offer thường thất bại — cơ chế, không phải quy tắc.** Bốn lý do, hoạt động cùng lúc:

1. Nếu nguyên nhân thật nằm ở "Công việc" hoặc "Con người", tiền không chạm vào nó. Sau khoảng 3–6
   tháng, sự phấn khởi từ mức lương mới bão hoà và nguyên nhân gốc vẫn nguyên vẹn.
2. Người đó đã vượt qua rào cản tâm lý của việc tìm việc, phỏng vấn, và hình dung mình ở nơi khác.
   Rào cản đó không dựng lại được.
3. Quan hệ đã đổi. Cả hai bên đều biết rằng người đó đã sẵn sàng đi, và điều đó ảnh hưởng tới việc
   giao việc dài hạn và cân nhắc promotion — thường một cách ngầm.
4. Nó phát tín hiệu cho toàn tổ chức rằng cách nhanh nhất để được tăng lương là đi phỏng vấn chỗ
   khác. Tín hiệu này lan nhanh và tốn kém hơn nhiều so với một lần mất người.

Counter-offer hợp lý trong đúng một trường hợp: **khi bạn thừa nhận rằng mình đã trả dưới giá thị
trường hoặc chậm promotion, và bạn gọi tên nó là sửa sai** — đồng thời rà soát ngay xem còn ai đang ở
tình trạng tương tự mà chưa lên tiếng. Sửa cho một người và để nguyên năm người khác là cách tạo ra
năm lần ra đi tiếp theo.

### Practical Framework

**Bước 1 — Đọc tín hiệu sớm.** Không có tín hiệu nào tự nó kết luận được, nhưng một tổ hợp thay đổi
so với chính người đó ba tháng trước thì đáng chú ý:

| Tín hiệu | Vì sao nó có ý nghĩa | Cách kiểm chứng (không phải cách suy diễn) |
|---|---|---|
| Ngừng đề xuất, ngừng phản biện trong design review | Người ta chỉ đầu tư vào việc cải thiện thứ mình còn thấy mình thuộc về | Hỏi trong 1-1: "Anh nhận thấy dạo này em ít đưa ý kiến hơn trước. Có phải anh đang bỏ lỡ điều gì không?" |
| Rút khỏi các việc có chân trời dài (không nhận task 6 tháng, không tham gia lập kế hoạch quý) | Người sắp đi tránh cam kết mà họ biết mình sẽ không hoàn thành | Giao một việc dài hạn có ý nghĩa và quan sát phản ứng |
| Thay đổi mẫu nghỉ phép: nhiều buổi lẻ, nghỉ vào buổi sáng | Phỏng vấn thường diễn ra vào giờ hành chính | Không hành động dựa trên riêng tín hiệu này — nó có quá nhiều cách giải thích khác |
| Giảm tham gia vào các việc không bắt buộc (guild, chia sẻ nội bộ, mentoring) | Rút dần khỏi phần "Con người" | Mời tham gia cụ thể vào một việc và xem phản hồi |
| Ngôn ngữ đổi từ "chúng ta" sang "các anh" | Chuyển bản dạng ra khỏi tổ chức | Rất đáng chú ý; kết hợp với tín hiệu khác |
| Ngừng phàn nàn sau một thời gian dài phàn nàn | Phàn nàn là dấu hiệu còn quan tâm; ngừng phàn nàn thường nghiêm trọng hơn | Đây là tín hiệu bị đọc sai nhiều nhất — lead hay thở phào khi ai đó ngừng phàn nàn |

Nguyên tắc quan trọng: **thấy tín hiệu thì hỏi, không phải thì suy diễn và tự xử lý.** Và không được
dùng những quan sát này để giám sát hay để loại người khỏi cơ hội — làm vậy là biến một công cụ chẩn
đoán thành một cơ chế trừng phạt, và nó sẽ được nhận ra.

**Bước 2 — Stay interview.** Đây là công cụ có tỉ suất sinh lợi cao nhất trong chủ đề này và gần như
không tổ chức nào ở quy mô vừa làm. Một cuộc trò chuyện 45 phút, mỗi 6 tháng, với mỗi người bạn không
muốn mất — tách biệt hoàn toàn khỏi review và khỏi bàn công việc hiện tại.

```
STAY INTERVIEW · mỗi 6 tháng · 45 phút · không trùng với review

Mở đầu (nói nguyên văn, để người đó biết cuộc trò chuyện này an toàn):
  "Đây không phải review. Không có gì em nói hôm nay ảnh hưởng tới đánh giá
   hay lương. Mục đích của anh là hiểu điều gì làm em muốn ở lại và điều gì
   làm em cân nhắc, khi mọi thứ còn sửa được."

Câu hỏi (chọn 5–6, đừng hỏi hết):
  1. Điều gì trong công việc hiện tại làm em thấy đáng để đến làm mỗi ngày?
  2. Điều gì làm em thấy nản nhất trong ba tháng qua?
  3. Nếu em có thể đổi một thứ trong cách team làm việc, đó là gì?
  4. Em đang học được gì? Có phải thứ em muốn học không?
  5. Nếu một recruiter gọi cho em tuần này, điều gì sẽ khiến em muốn nghe hết?
  6. Em muốn công việc của mình trong 12 tháng nữa khác gì hiện tại?
  7. Có điều gì em từng nêu mà em cảm thấy không được phản hồi không?
  8. Anh đang làm gì khiến công việc của em khó hơn mức cần thiết?

Sau cuộc trò chuyện — phần quyết định giá trị của cả công cụ:
  - Chọn MỘT điều trong danh sách và thay đổi nó trong vòng 30 ngày.
  - Quay lại nói với người đó rằng bạn đã thay đổi vì cuộc trò chuyện đó.
  - Nếu có điều bạn KHÔNG thể thay đổi, nói thẳng vì sao — đừng im lặng.

Nếu bạn hỏi mà không thay đổi gì, lần sau bạn sẽ nhận được câu trả lời lịch sự
và vô nghĩa, và bạn đã tiêu mất công cụ này vĩnh viễn.
```

Câu 8 là câu khó hỏi nhất và cho thông tin tốt nhất. Ở bối cảnh Việt Nam, câu này thường nhận được
"dạ không có gì đâu anh" ở lần đầu. Cách xử lý: đừng ép trong lần đó; thay vào đó, hãy tự nêu một
điều bạn nghĩ mình đang làm chưa tốt trước, rồi hỏi lại. Việc bạn đi trước hạ chi phí của việc nói
thật xuống mức chấp nhận được.

**Bước 3 — Khi người ta báo nghỉ.** Trình tự đúng, và nó khác trực giác:

*Phiên bản phản ứng sai:*

> **Hà:** Ơ, sao đột ngột thế? Bên kia trả bao nhiêu? Để chị nói chuyện với sếp, chị nghĩ mình
> match được. Em đừng vội quyết, mình còn nhiều dự án hay sắp tới mà.

Ba lỗi trong bốn câu: chuyển ngay sang đàm phán tiền (đóng cửa với mọi nguyên nhân khác), ngầm trách
người đó đột ngột (khiến họ phòng vệ và ngừng nói thật), và hứa mơ hồ về tương lai (chính xác loại
lời hứa có thể đã góp phần làm họ đi).

*Phiên bản đúng:*

> **Hà:** Cảm ơn em đã nói trực tiếp với chị. Trước khi bàn bất cứ điều gì khác, chị muốn hiểu — và
> chị hỏi thật lòng, không phải để thuyết phục em ở lại. Nếu chị nhìn lại sáu tháng qua, thời điểm
> nào em bắt đầu nghĩ tới việc đi?
>
> **Quân:** *(im lặng)* Chắc là khoảng tháng ba ạ. Sau cái RFC về đối soát.
>
> **Hà:** Cái bị gác lại mà không có lý do?
>
> **Quân:** Vâng. Không phải vì nó bị gác — em hiểu là có ưu tiên khác. Mà là em không biết vì sao.
> Em hỏi hai lần rồi thôi không hỏi nữa.
>
> **Hà:** Chị hiểu rồi. Đó là lỗi của chị, và chị sẽ không cố sửa nó trong mười lăm phút để giữ em
> lại — như thế không công bằng với em. Chị có hai việc muốn làm. Một: nếu em còn muốn nói thêm về
> những chỗ tương tự, chị muốn nghe, vì nó ảnh hưởng tới những người còn ở đây. Hai: chị muốn bàn về
> bàn giao sao cho em ra đi mà quan hệ giữa mình vẫn tốt. Còn chuyện counter-offer, chị sẽ nói thẳng:
> chị không đưa, vì vấn đề không phải tiền và chị không muốn dùng tiền để hoãn một quyết định đúng
> của em.

Điều Hà làm được mà phiên bản đầu không làm được: cô lấy được **nguyên nhân thật**, và nguyên nhân
đó có giá trị cho năm người còn lại trong team. Cô cũng giữ được quan hệ — và trong một thị trường
ngành hẹp như Việt Nam, một người rời đi trong sự tôn trọng là một nguồn giới thiệu ứng viên và có
khả năng quay lại (boomerang hire) trong hai đến ba năm.

**Bước 4 — Bàn giao để không mất tri thức.** Checklist tối thiểu, bắt đầu ngay ngày đầu tiên của thời
gian báo trước:

```
CHECKLIST BÀN GIAO · [Tên] · Ngày cuối: [ngày]

TRI THỨC (ưu tiên cao nhất — đây là phần không mua lại được)
  [ ] Danh sách "những gì tôi biết mà không ai khác biết" — do chính
      người đi viết, 30 phút, không cần đẹp
  [ ] Với mỗi mục: ghi âm/ghi hình một buổi 30 phút giải thích, hoặc
      pairing với người nhận
  [ ] Các quyết định cũ và lý do đằng sau — bổ sung vào ADR nếu thiếu
  [ ] "Bản đồ cạm bẫy": chỗ nào trong hệ thống dễ gây sự cố và vì sao
  [ ] Quan hệ bên ngoài: ai ở team/công ty khác cần được giới thiệu lại

VẬN HÀNH
  [ ] Runbook của các service đang sở hữu — chạy thử với người nhận
  [ ] Chuyển ownership trong hệ thống (on-call, alert, repo, tài liệu)
  [ ] Danh sách việc đang dở, trạng thái thật, và rủi ro của từng cái

QUY TRÌNH
  [ ] Thu hồi truy cập theo quy trình HR/bảo mật (phối hợp, không tự làm)
  [ ] Exit interview do người KHÁC manager trực tiếp thực hiện
  [ ] Cập nhật tài liệu team: ai sở hữu cái gì sau ngày [ngày]

Nguyên tắc: người nhận bàn giao phải THỰC HIỆN được, không chỉ NGHE.
Một buổi giải thích không được kiểm chứng bằng việc người nhận tự làm
là một buổi giải thích chưa xảy ra.
```

### Trade-off

| Trục | Nghiêng về A khi | Nghiêng về B khi | Cái giá của mỗi bên |
|---|---|---|---|
| **A: Giữ bằng lương** vs **B: Sửa nguyên nhân** | Bạn thật sự đang trả dưới thị trường và đó là nguyên nhân chính | Nguyên nhân nằm ở công việc, quan hệ, hoặc lộ trình | A: không sửa gì, tạo tiền lệ, và tín hiệu "muốn tăng lương thì đi phỏng vấn" lan ra toàn tổ chức. B: chậm, có thể không kịp cho người này |
| **A: Đầu tư giữ người** vs **B: Chấp nhận turnover và tối ưu tuyển + onboarding** | Hệ thống chuyên biệt, tri thức ẩn nhiều, thời gian tới năng suất dài | Công việc chuẩn hoá cao, onboarding nhanh, thị trường cung dồi dào | A: chi phí quản lý cao, và rủi ro giữ cả người nên đi. B: mất tri thức liên tục, và chất lượng kiến trúc dài hạn giảm vì không ai ở đủ lâu để chịu hậu quả quyết định của mình |
| **A: Minh bạch về lương trong tổ chức** vs **B: Giữ kín** | Muốn giảm bất công cảm nhận và giảm rủi ro người ta chỉ biết giá của mình qua thị trường bên ngoài | Cấu trúc lương hiện tại chưa hợp lý và bạn chưa sửa được | A: mọi bất hợp lý hiện có sẽ lộ ra cùng lúc và phải xử lý ngay. B: thông tin vẫn rò rỉ nhưng dưới dạng tin đồn và so sánh lệch, thường tệ hơn sự thật |
| **A: Nói với team khi có người sắp nghỉ** vs **B: Chờ tới khi chính thức** | Cần thời gian bàn giao thật; team sẽ biết dù bạn không nói | Người đó chưa muốn công khai | Luôn hỏi người đó muốn thông báo thế nào và khi nào. A không có sự đồng ý là vi phạm sự tin cậy; B quá lâu làm bàn giao thành hình thức |

### Real-world Scenarios

**Tình huống — Senior báo offer +40% (product startup, 26 engineer).**

*Bối cảnh.* Linh, Staff Engineer, ba năm ở công ty, người thiết kế nền tảng observability mà cả ba
team dùng. Cô báo với Hà rằng có offer từ một công ty lớn hơn, mức lương cao hơn 40%, vai trò tương
đương. Ngân sách của công ty không cho phép match mức đó.

*Các lựa chọn.* (1) Cố match bằng mọi giá, xin ngoại lệ ngân sách. (2) Không match, chấp nhận mất
người, tập trung vào bàn giao. (3) Tìm hiểu nguyên nhân thật trước khi quyết định làm gì. (4) Đưa
một gói phi tiền (thời gian linh hoạt, phạm vi lớn hơn, ngân sách học tập).

*Trade-off.* Lựa chọn 1, nếu làm được, tạo một mức lương lệch hẳn khỏi thang của công ty — và trong
một tổ chức 26 người, thông tin này sẽ được biết trong vòng một quý, kéo theo áp lực điều chỉnh cho
những người khác mà ngân sách không chịu nổi. Lựa chọn 2 đúng về tài chính nhưng bỏ qua khả năng
nguyên nhân không phải tiền. Lựa chọn 4 có thể là câu trả lời nhưng chưa biết có đúng bệnh không.

*Quyết định.* Hà chọn 3 trước, và làm rõ ngay từ đầu rằng cô không hỏi để mặc cả. Điều phát hiện ra:
Linh không đi vì tiền. Cô đi vì ở công ty hiện tại, cô là người duy nhất ở tầm đó và cô không còn ai
để học. "Em không có ai review được thiết kế của em nữa. Em không biết mình đang giỏi lên hay đang
lặp lại." Mức lương +40% là thứ khiến cô bắt đầu nói chuyện, không phải thứ khiến cô đi.

Hà không match lương. Cô đưa ba việc, và nói rõ rằng cô không chắc chúng đủ: ngân sách để Linh làm
việc với một cố vấn kỹ thuật bên ngoài mỗi tháng; đưa Linh vào các cuộc thảo luận chiến lược kỹ
thuật ở cấp công ty mà trước đó cô không tham gia; và cam kết bằng văn bản về việc tuyển thêm một
người ở tầm tương đương trong hai quý — kèm điều kiện rằng nếu sau hai quý không tuyển được, Hà sẽ
là người chủ động nêu lại chuyện này.

*Hậu quả.* Linh ở lại. Mười bốn tháng sau, cô vẫn ở đó và đang là người phỏng vấn cho vị trí Staff
thứ hai. Nhưng kết quả này không phải điều Hà kiểm soát được — nếu Linh vẫn đi, quy trình vẫn đúng.

*Bài học.* (a) Con số trong offer là thứ dễ nhìn nhất và hiếm khi là biến quyết định — nhưng bạn chỉ
biết điều đó nếu hỏi trước khi trả giá. (b) Cam kết có kiểm chứng ("nếu sau hai quý không tuyển được,
tôi sẽ chủ động nêu lại") có giá trị hơn nhiều so với lời hứa mở, vì nó đặt trách nhiệm lên người
hứa. (c) Hà không giấu việc mình không chắc — và chính sự thẳng thắn đó làm cho phần cô cam kết trở
nên đáng tin.

### Best Practices

- **Chạy stay interview mỗi 6 tháng với những người bạn không muốn mất**, tách khỏi review, và thay
  đổi ít nhất một điều sau mỗi lần. Công cụ này mất giá trị vĩnh viễn nếu bạn hỏi mà không hành động.
- **Tách turnover thành regretted và non-regretted** trước khi nhìn bất kỳ con số nào. Hai loại này
  đòi hỏi hai can thiệp ngược nhau.
- **Khi có người báo nghỉ, hỏi nguyên nhân trước khi bàn bất cứ điều gì khác** — và nói rõ rằng bạn
  hỏi không phải để thuyết phục. Đây là điều kiện để nhận được câu trả lời thật.
- **Exit interview phải do người khác manager trực tiếp thực hiện**, và kết quả tổng hợp theo mẫu
  chứ không theo từng ca.
- **Nếu counter-offer, gọi tên nó là sửa sai và rà soát ngay những người tương tự.** Sửa cho một
  người và bỏ qua năm người là tạo ra năm lần ra đi tiếp theo.
- **Coi tín hiệu "ngừng đề xuất" là nghiêm trọng hơn tín hiệu "hay phàn nàn".** Phàn nàn là bằng
  chứng còn quan tâm.
- **Bắt đầu bàn giao tri thức từ ngày đầu của thời gian báo trước**, và ưu tiên phần tri thức không
  có trong tài liệu trước phần thủ tục.
- **Giữ quan hệ tốt với người rời đi.** Trong một ngành có mạng lưới hẹp, họ là nguồn giới thiệu ứng
  viên, là kênh thông tin về thị trường, và một tỉ lệ đáng kể quay lại sau 2–3 năm với năng lực rộng
  hơn.
- **Sau mỗi lần mất người mà bạn tiếc, hỏi một câu**: dấu hiệu đầu tiên xuất hiện từ khi nào, và cái
  gì trong hệ thống đã ngăn bạn thấy nó? Câu trả lời gần như luôn chỉ về nhịp 1-1 hoặc về việc các đề
  xuất không được phản hồi.

### Anti-patterns

- **Chỉ nói chuyện phát triển khi người ta đã nộp đơn.** Cơ chế: cuộc trò chuyện đó bây giờ được đọc
  như một nỗ lực giữ người, nên mọi cam kết trong đó đều bị chiết khấu — kể cả những cam kết thật.
  Bạn đã tiêu mất công cụ vào đúng lúc cần nó nhất. Dấu hiệu sớm: 1-1 gần nhất của bạn với người đó
  cách đây hơn một tháng, và nội dung là về trạng thái task.
- **Counter-offer như phản xạ.** Ngoài bốn cơ chế đã nêu, nó còn một tác hại phụ: nó dạy chính bạn
  rằng vấn đề giữ người giải quyết được bằng ngân sách, nên bạn ngừng tìm nguyên nhân thật.
- **Exit interview do manager trực tiếp thực hiện.** Nếu manager là một phần của nguyên nhân — và
  trong tỉ lệ lớn các trường hợp thì có — dữ liệu thu được là vô giá trị nhưng lại được ghi vào hệ
  thống như thể nó có giá trị. Điều này tệ hơn không có dữ liệu, vì nó tạo ra kết luận sai có vẻ
  ngoài chắc chắn.
- **Coi việc nghỉ là phản bội.** Phản ứng lạnh nhạt hoặc trừng phạt trong thời gian báo trước phá
  hoại ba thứ cùng lúc: chất lượng bàn giao (người đó không còn động lực chuyển giao tri thức thật),
  quan hệ lâu dài, và tín hiệu cho team về việc điều gì xảy ra nếu họ trung thực về kế hoạch của mình.
- **Giữ người bằng cách chặn đường ra.** Không giới thiệu, gây khó khăn cho việc bàn giao, nói xấu
  với người trong ngành. Trong một thị trường mà mọi người biết nhau, chi phí danh tiếng của việc này
  lớn hơn nhiều lần giá trị giữ được.
- **Dùng tín hiệu sớm để giám sát thay vì để hỏi han.** Nếu bạn dùng những quan sát ở bảng trên để
  loại ai đó khỏi dự án quan trọng "vì họ có thể sắp đi", bạn vừa tạo ra chính điều bạn sợ, và bạn
  đã biến một công cụ chẩn đoán thành một cơ chế trừng phạt.
- **Đo turnover tổng và đặt mục tiêu giảm nó.** Mục tiêu này có thể đạt được bằng cách giữ cả những
  người nên đi, và nó khiến manager ngại xử lý underperformance. Goodhart's Law áp dụng đầy đủ.

### Khi nào KHÔNG nên áp dụng

- **Khi người rời đi thuộc nhóm non-regretted.** Chạy stay interview, counter-offer, hay quy trình
  giữ người với người mà việc họ đi là kết quả tốt là lãng phí và gửi tín hiệu sai cho team. Việc
  đúng: bàn giao đàng hoàng, tôn trọng, và nhìn ngược lên quy trình tuyển để hiểu vì sao đã tuyển sai.
- **Khi bạn không thể thay đổi gì trong ba biến.** Nếu tổ chức đang trong tình trạng mà bạn không có
  quyền với lương, không có quyền với phạm vi công việc, và không có quyền với điều kiện làm việc,
  thì stay interview sẽ tạo kỳ vọng bạn không đáp ứng được — và điều đó làm tình hình xấu nhanh hơn.
  Việc trung thực hơn: nói thẳng với người đó về ràng buộc thực tế, và giúp họ ra quyết định có thông
  tin, kể cả khi quyết định đó là rời đi.
- **Trong giai đoạn tổ chức đang cắt giảm hoặc bất ổn công khai.** Câu hỏi "điều gì làm em muốn ở
  lại" trong bối cảnh mọi người đang lo về việc làm sẽ bị đọc là một cuộc đánh giá trá hình. Hoãn
  công cụ này; thay vào đó, tập trung vào truyền thông trung thực về tình hình.
- **Với người mới dưới ba tháng.** Tín hiệu sớm không đọc được vì bạn chưa có đường cơ sở cho người
  đó, và nhiều hành vi trông như rút lui thực ra là biểu hiện bình thường của giai đoạn onboarding.
  Công cụ đúng ở giai đoạn này là check-in onboarding (xem chủ đề 4), không phải stay interview.
- **Khi lý do rời đi nằm hoàn toàn ngoài tầm tổ chức** (chuyển thành phố theo gia đình, đổi ngành,
  đi học). Cố gắng "sửa nguyên nhân" ở đây là không tôn trọng quyết định của người khác. Việc đúng:
  bàn giao tốt, giữ quan hệ, và giữ cửa mở.

---

## Tự kiểm tra

Trả lời bằng tên người, con số, hoặc ngày cụ thể. Câu nào bạn không trả lời được bằng dữ liệu là một
chỗ hệ thống con người của bạn đang mù.

1. **Vị trí gần nhất bạn mở tuyển — bạn viết scorecard trước hay viết JD trước?** Nếu viết JD trước,
   bạn đã mô tả một cái ghế chứ chưa xác định năng lực còn thiếu.
2. **Trong vòng phỏng vấn gần nhất, mỗi năng lực trong scorecard được đo bởi mấy vòng, và có rubric
   không?** Nếu một năng lực chỉ được đo một lần và không có rubric, kết luận về nó là nhiễu.
3. **Người mới nhất vào team mất bao nhiêu ngày để merge PR đầu tiên, và bao nhiêu tuần để ra quyết
   định kỹ thuật độc lập đầu tiên?** Nếu bạn không biết, bạn không đang quản lý onboarding — bạn đang
   hy vọng nó tự diễn ra.
4. **Mở career ladder của công ty bạn: bậc Senior được định nghĩa bằng gì?** Nếu trong định nghĩa có
   cụm "số năm kinh nghiệm", ladder của bạn đang đo thời gian chứ không đo phạm vi.
5. **Trong review gần nhất, có ai bị bất ngờ không?** Mỗi người bị bất ngờ là một lỗi của vòng
   feedback trong sáu tháng trước đó, không phải một lỗi của người đó.
6. **Ai trong team bạn đang hoạt động ở bậc trên mà chưa có ai viết hồ sơ promotion cho họ?** Gọi tên.
   Nếu không có tên nào, hãy kiểm tra lại — hoặc bạn đang đánh giá quá chặt, hoặc bạn chưa nhìn kỹ.
7. **Có ai trong team đang dưới kỳ vọng mà bạn đã nghĩ tới quá ba lần nhưng chưa nói câu nào không?**
   Ngày bạn nhận ra là ngày nào? Khoảng cách giữa ngày đó và hôm nay chính là chi phí bạn đang tích
   luỹ.
8. **Trong 12 tháng qua, bao nhiêu người rời đi, và bao nhiêu trong số đó là regretted?** Với mỗi
   người regretted: dấu hiệu đầu tiên xuất hiện từ khi nào, và bạn đã làm gì trong 30 ngày sau đó?

---

## Liên kết chương khác

- [`00-nen-tang-leadership.md`](/series/engineering-leedership/00-nen-tang-leadership/) — Accountability quyết định ai chịu trách
  nhiệm về một lần tuyển sai; chuyển từ Individual Output sang System Output là gốc lý giải vì sao
  phát triển người là công việc chính chứ không phải việc phụ của lead.
- [`02-communication.md`](/series/engineering-leedership/02-communication/) — Feedback, Difficult Conversations và One-on-One là
  hạ tầng cho toàn bộ chương này; không có 1-1 chất lượng thì mọi cơ chế ở đây đều chạy trên dữ liệu
  sai.
- [`03-team-leadership.md`](/series/engineering-leedership/03-team-leadership/) — Psychological Safety quyết định người ta có nói
  thật trong stay interview hay không; Mentoring, Coaching và Sponsorship là ba công cụ khác nhau
  được dùng ở ba chỗ khác nhau trong chương này.
- [`04-decision-making.md`](/series/engineering-leedership/04-decision-making/) — quyết định tuyển là quyết định irreversible với
  blast radius lớn; decision log và cơ chế chống bias áp dụng trực tiếp cho debrief và calibration.
- [`05-technical-leadership.md`](/series/engineering-leedership/05-technical-leadership/) — Code Review Culture là nơi hành vi gây
  hại hiện ra sớm nhất và đo được; engineering culture là tập hành vi được thưởng, mà promotion chính
  là cơ chế thưởng mạnh nhất.
- [`07-project-delivery.md`](/series/engineering-leedership/07-project-delivery/) — vì sao thêm người không cứu được dự án trễ;
  áp lực delivery là một trong những nguyên nhân hệ thống phổ biến nhất của underperformance và của
  việc mất người.
- [`09-to-chuc-va-scaling.md`](/series/engineering-leedership/09-to-chuc-va-scaling/) — seniority mix, cấu trúc team, và cách
  thiết kế tổ chức để giảm phụ thuộc vào một vài cá nhân xuất sắc.
- [`11-career-evolution.md`](/series/engineering-leedership/11-career-evolution/) — mô tả chi tiết từng bậc từ Junior tới Director:
  phạm vi ảnh hưởng, cách ra quyết định, và sai lầm thường gặp khi chuyển vai trò.
- [`12-anti-patterns.md`](/series/engineering-leedership/12-anti-patterns/) — Peter Principle, Hero Culture, và các anti-pattern
  về quản lý con người được tổng hợp cùng dấu hiệu sớm và cách tháo gỡ.
