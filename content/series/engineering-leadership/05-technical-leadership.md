+++
title = "Level 3B — Technical Leadership: Technical Vision, ADR, RFC và Technical Debt"
date = "2026-08-01T13:00:00+07:00"
draft = false
tags = ["engineering-leadership", "management", "career"]
series = ["Engineering Leadership & Technical Management"]
+++

# Level 3B — Technical Leadership

Một công ty e-commerce ở TP.HCM, 42 engineer chia thành sáu team. Tháng trước, khi cần thêm một tính
năng thông báo cho người mua, team Order dựng một worker đọc Kafka; team Promotion gọi HTTP trực tiếp
sang service notification và retry bằng vòng `for`; team Logistics đẩy job vào Redis queue vì "trước
đây làm thế đã chạy tốt". Ba cách gửi thông báo, ba mô hình retry, ba chỗ để rơi message, và không ai
vi phạm quy tắc nào — vì không có quy tắc nào. Khi có một incident mất thông báo giao hàng, on-call
phải mở ba runbook khác nhau, và người viết cả ba đều đã rời công ty. Cuối quý, một câu hỏi tưởng đơn
giản của CTO — "muốn thêm kênh Zalo thì mất bao lâu" — nhận ba câu trả lời cách nhau từ 3 ngày đến 6
tuần.

Không team nào làm sai. Mỗi team ra một quyết định hợp lý cục bộ với thông tin họ có. Tổng của các
quyết định hợp lý cục bộ lại là một hệ thống không ai thiết kế, và không ai có thể suy luận về nó.
Đây chính là vấn đề mà Technical Leadership giải quyết: **định hình các ràng buộc kỹ thuật để nhiều
người có thể ra quyết định độc lập mà hệ thống vẫn hội tụ.** Nếu thiếu, mỗi team tự chọn hướng và
tổng hệ thống phân kỳ — không phải vì thiếu năng lực mà vì thiếu một trường lực hướng các quyết định
về cùng một phía.

Phân biệt với chương trước cho rõ: chương 04 nói về **cách sản xuất một quyết định**. Chương này nói
về cách làm cho **hàng nghìn quyết định bạn không có mặt để tham gia** vẫn đi cùng hướng. Đó là một
bài toán khác về bản chất: bạn không scale bằng cách quyết nhiều hơn, bạn scale bằng cách làm cho
quyết định của người khác dễ đúng hơn. Mọi công cụ trong chương này — vision, strategy, ADR, review
culture, RFC, standard, golden path, culture — đều là biến thể của một cơ chế duy nhất: **nén thông
tin về ưu tiên và ràng buộc vào một dạng có thể mang theo khi bạn không ở trong phòng.**

Trong chuỗi Business Goal → People → Process → Technology → Execution → Feedback → Improvement →
Scaling Team → Organization, chương này bắc cầu giữa Process và Technology, và nó là nơi duy nhất
trong bộ tài liệu mà nội dung kỹ thuật của quyết định được coi là đối tượng quản trị. Không có tầng
này, tầng Technology trở thành tổng các sở thích cá nhân, và tầng Scaling Team sẽ nhân bản sự phân kỳ
đó theo số team.

Mục lục nội bộ:

1. [Technical Vision và Technical Strategy](#1-technical-vision-và-technical-strategy)
2. [Architecture Decision và ADR](#2-architecture-decision-và-adr)
3. [Code Review Culture](#3-code-review-culture)
4. [RFC Process và Architecture Review](#4-rfc-process-và-architecture-review)
5. [Technical Debt Management](#5-technical-debt-management)
6. [Engineering Standards và Golden Path](#6-engineering-standards-và-golden-path)
7. [Engineering Culture — thiết kế thay vì hô hào](#7-engineering-culture--thiết-kế-thay-vì-hô-hào)
8. [Giữ technical credibility khi không còn viết code nhiều](#8-giữ-technical-credibility-khi-không-còn-viết-code-nhiều)
9. [Tự kiểm tra](#tự-kiểm-tra)
10. [Liên kết chương khác](#liên-kết-chương-khác)

> Mọi con số trong chương này là **số minh hoạ** để bạn thấy cách tính và cách đặt ngưỡng, không phải
> dữ liệu nội bộ của bất kỳ công ty nào. Khi áp dụng, thay bằng số đo được từ hệ thống của bạn.

---

## 1. Technical Vision và Technical Strategy

### Problem Statement

Ba hiện tượng dưới đây đếm được, và mỗi cái là một dấu hiệu tổ chức đang không có technical strategy —
kể cả khi trên wiki có một trang tên là "Technical Strategy 2026".

**Hiện tượng một: cùng một bài toán, nhiều lời giải song song tồn tại lâu dài.** Đếm số cách hệ thống
của bạn làm cùng một việc: bao nhiêu cách gọi service khác, bao nhiêu cách gửi email, bao nhiêu cách
cấu hình, bao nhiêu cách xác thực nội bộ. Ở công ty e-commerce đầu chương, con số cho "gửi thông báo"
là ba, cho "gọi service nội bộ" là bốn (HTTP trực tiếp, gRPC, Kafka, và một service gọi thẳng vào
database của service khác). Với mỗi cách bổ sung, chi phí không cộng thêm mà nhân lên: on-call phải
biết cả n cách, tài liệu phải mô tả cả n, và một thay đổi xuyên suốt phải làm n lần.

**Hiện tượng hai: câu hỏi "để làm X thì cần bao lâu" nhận các câu trả lời lệch nhau hàng bậc.** Đây
không phải vấn đề Estimation. Đây là dấu hiệu **không tồn tại architectural runway**: mỗi khi có nhu
cầu mới, người ta phải dựng lại hạ tầng cho nhu cầu đó từ đầu, và thời gian phụ thuộc vào việc team
nào đang trả lời.

**Hiện tượng ba: mọi cuộc tranh luận kỹ thuật đều phải giải lại từ tiên đề.** Khi một Senior đề xuất
dùng GraphQL cho một API mới, cuộc thảo luận không bắt đầu từ "cái này có khớp với hướng đã chọn
không" mà từ "GraphQL tốt hay REST tốt". Cùng một cuộc tranh luận đó sẽ diễn ra lại ở service tiếp
theo, với những người tiếp theo. Đếm số giờ họp kỹ thuật trong quý mà kết quả là một lựa chọn cục bộ
không có tính bắt buộc với ai khác — ở nhiều tổ chức 30–50 engineer, con số minh hoạ này rơi vào
40–80 giờ mỗi quý.

Hậu quả dài hạn có một hình dạng khá cố định. Hệ thống phân kỳ → cognitive load tăng → onboarding
chậm → chỉ người viết ban đầu hiểu module của mình → khi người đó nghỉ, module thành vùng không ai
dám sửa → mọi thay đổi lớn phải đi qua vài người → những người đó thành bottleneck và burnout → tổ
chức mất cả tốc độ lẫn người. Chú ý bước cuối: **thiếu technical strategy cuối cùng biểu hiện thành
vấn đề nhân sự, không phải vấn đề kỹ thuật** — và đó là lý do CTO thường nhận ra nó qua tỉ lệ nghỉ
việc chứ không qua kiến trúc.

### First Principles

**Cơ chế một: strategy tồn tại vì tài nguyên hữu hạn, nên bản chất của nó là hy sinh có chủ đích.**
Nếu bạn có vô hạn engineer và vô hạn thời gian, bạn không cần chiến lược — bạn làm mọi thứ. Chiến
lược chỉ có ý nghĩa khi tổng nhu cầu vượt tổng năng lực, và khi đó nội dung thật của chiến lược là
**danh sách những gì sẽ không được làm dù nó hợp lý**. Suy ra một phép thử sắc: đọc tài liệu strategy
của tổ chức bạn và tìm phần nói "chúng ta cố tình không làm X". Nếu không có phần đó, tài liệu đó
không phải strategy — nó là một danh sách ước muốn, và danh sách ước muốn không định hướng được quyết
định vì nó không loại bỏ được lựa chọn nào.

**Cơ chế hai: vision là một cơ chế nén, và nó tồn tại vì băng thông phối hợp của con người có giới
hạn.** Trong một tổ chức 40 engineer, mỗi ngày có hàng trăm quyết định nhỏ được ra mà bạn không thể
tham gia. Bạn có hai lựa chọn: (a) tăng số lần đồng bộ — họp nhiều hơn, duyệt nhiều hơn, chi phí phối
hợp tăng theo n²; hoặc (b) nén ưu tiên và ràng buộc vào một dạng mà mỗi người có thể tự áp dụng — chi
phí phối hợp gần như tuyến tính. Vision là lựa chọn (b). Nó không nói người ta phải làm gì; nó nói
**trạng thái đích trông như thế nào**, để khi đứng trước hai lựa chọn, engineer có thể tự hỏi "cái nào
đưa hệ thống về gần đích hơn" mà không cần hỏi ai. Đây là cùng một nguyên lý mà Netflix gọi công khai
là "context over control": bạn phát context để người ta tự quyết, thay vì kiểm soát từng quyết định.

**Cơ chế ba: quyết định phân tán chỉ hội tụ nếu tồn tại một hàm mục tiêu chung.** Đây là kết quả có
thể suy ra từ lý thuyết hệ thống: n tác tử tối ưu cục bộ theo n hàm mục tiêu khác nhau sẽ cho một
trạng thái tổng không tối ưu theo bất kỳ hàm nào. Team Order tối ưu cho throughput, team Promotion tối
ưu cho time-to-market, team Logistics tối ưu cho tính ổn định — cả ba đều đúng theo hàm của mình, và
kết quả tổng là ba mô hình retry. Vision + strategy là cách áp một hàm mục tiêu chung mà không cần
tước quyền quyết định cục bộ.

**Cơ chế bốn: chi phí thay đổi kiến trúc tăng theo lượng thứ đã bám vào nó.** Điều này khiến vision
phải có tầm 12–24 tháng, không phải 3 tháng và không phải 5 năm. Ngắn hơn 12 tháng thì không đủ dài để
một thay đổi kiến trúc có ý nghĩa hoàn thành, nên vision biến thành roadmap. Dài hơn 24 tháng thì độ
bất định về thị trường và công nghệ vượt giá trị của việc lên kế hoạch, nên vision biến thành khoa
học viễn tưởng.

### Mental Model

**Mô hình một: Strategy Kernel (diagnosis → guiding policy → coherent actions).** Richard Rumelt mô tả
công khai trong *Good Strategy Bad Strategy* rằng một chiến lược có ba phần bắt buộc và thường bị
thiếu phần đầu:

| Thành phần | Câu hỏi nó trả lời | Dạng đúng | Dạng sai thường thấy |
|---|---|---|---|
| **Diagnosis** | Vấn đề thật là gì, và bằng chứng nào? | "Lead time trung vị cho thay đổi ở module thanh toán là 11 ngày, gấp 4 lần các module khác; 60% thời gian là chờ QA thủ công vì không có test tự động." | "Hệ thống của chúng ta cần hiện đại hoá." |
| **Guiding policy** | Cách tiếp cận tổng thể để xử lý vấn đề đó | "Ưu tiên tách các luồng có yêu cầu compliance khác nhau; mọi service mới phải chạy được trên golden path hiện có, không dựng hạ tầng riêng." | "Chúng ta sẽ dùng microservices và Kubernetes." |
| **Coherent actions** | Các hành động cụ thể, không xung đột nhau, cùng phục vụ policy | "Q1: tách payment; Q2: golden path v2 có template service; Q3: dừng nhận service mới ngoài golden path." | Danh sách 14 initiative của 6 team, không ai kiểm tra chúng có xung đột hay không. |

Từ "coherent" là từ nặng nhất trong ba dòng. Rất nhiều tổ chức có diagnosis mờ, policy chung chung, và
một tập action **xung đột nhau**: cùng lúc đẩy "tách microservice" và "giảm chi phí hạ tầng 30%", cùng
lúc yêu cầu "tăng test coverage" và "giảm 20% thời gian mỗi sprint dành cho việc không phải feature".
Khi action xung đột, engineer phải chọn, và họ chọn theo cái được thưởng — xem chủ đề 7.

**Mô hình hai: architectural runway.** Vay từ SAFe nhưng dùng theo nghĩa hẹp và hữu ích: runway là
**lượng năng lực kỹ thuật đã tồn tại đủ để hỗ trợ các tính năng sắp tới mà không cần redesign**. Máy
bay cần đường băng đủ dài trước khi cất cánh; nếu bạn xây đường băng trong lúc đang chạy, bạn hoặc
dừng lại hoặc rơi. Chỉ số quan sát được của runway: khi PO đề xuất một tính năng thuộc loại đã biết,
tỉ lệ công việc là "dựng hạ tầng mới" so với "viết business logic". Nếu tỉ lệ đó thường xuyên trên
50%, runway của bạn đã hết, và mọi cam kết roadmap sẽ trễ theo hệ thống.

**Mô hình ba: north star kỹ thuật.** Một câu duy nhất, đo được, mà mọi quyết định kiến trúc có thể
được kiểm tra ngược lại. Đặc điểm của một north star dùng được: nó có thể **sai** — tức là có những
lựa chọn kỹ thuật rõ ràng vi phạm nó.

| North star kỹ thuật (minh hoạ) | Nó loại bỏ điều gì | Có dùng được không |
|---|---|---|
| "Bất kỳ engineer nào cũng deploy được một thay đổi lên production trong ngày đầu tiên vào team." | Loại release theo lịch, loại quy trình duyệt thủ công nhiều tầng, loại môi trường mà chỉ vài người dựng được. | Có — loại bỏ rõ ràng. |
| "Mọi giao dịch tiền phải đối soát được trong 15 phút bằng một truy vấn duy nhất." | Loại kiến trúc phân tán dữ liệu tiền qua nhiều store không có nguồn sự thật; loại eventual consistency ở luồng bút toán. | Có. |
| "Chúng ta sẽ trở thành một tổ chức engineering đẳng cấp thế giới." | Không loại gì. | Không. |

### Practical Framework

**Quy trình viết một technical strategy hai trang — 5 bước, khoảng 2 tuần lịch, không quá 12 giờ làm
việc thực của bạn.**

**Bước 1 — Thu thập diagnosis bằng số, 4–6 giờ.** Không phỏng vấn cảm nhận trước khi có số. Bốn nhóm
dữ liệu tối thiểu, tất cả đều lấy được từ hệ thống bạn đang có:

| Nhóm dữ liệu | Lấy từ đâu | Câu hỏi nó trả lời |
|---|---|---|
| Lead time cho thay đổi, theo module | Git + issue tracker: thời gian từ commit đầu tiên tới production | Chỗ nào chậm, và chậm hơn phần còn lại bao nhiêu lần |
| Change failure rate và số incident, theo module | Hệ thống incident, tag theo service | Chỗ nào rủi ro tập trung |
| Phân bổ effort thực tế | Sampling 3–4 sprint gần nhất: % thời gian cho feature / bug / hạ tầng / vận hành thủ công | Tổ chức đang thật sự đầu tư vào đâu, khác gì với điều nó nói |
| Số cách làm cùng một việc | Đếm thủ công 30 phút với 2 senior | Mức độ phân kỳ hiện tại |

**Bước 2 — Xác định ràng buộc thật (constraint), 2 giờ.** Theo Theory of Constraints, cải thiện chỗ
không phải constraint không tạo ra thay đổi ở đầu ra. Viết ra một câu: "hiện tại thứ giới hạn tốc độ
đưa giá trị ra thị trường của chúng ta là ___". Ứng viên thường gặp: thời gian chờ QA thủ công; một
người duy nhất hiểu module lõi; thời gian dựng môi trường; quy trình duyệt của khách hàng (bối cảnh
ODC); chi phí hạ tầng chạm ngưỡng ngân sách. Nếu bạn viết được ba constraint, bạn chưa tìm được
constraint — hãy hỏi cái nào mà nếu tháo đi thì hai cái kia tự nhẹ đi.

**Bước 3 — Viết 3–5 nguyên tắc dẫn hướng, 2 giờ.** Tiêu chuẩn cho một nguyên tắc dùng được: (a) nó
loại bỏ được ít nhất một lựa chọn phổ biến; (b) một engineer mid-level có thể tự áp dụng mà không hỏi
bạn; (c) nó không phải phát biểu ai cũng đồng ý. Phép thử: viết câu phủ định của nguyên tắc — nếu câu
phủ định nghe vô lý với mọi người, nguyên tắc đó là filler. "Chúng ta coi trọng chất lượng code" có
câu phủ định là "chúng ta không coi trọng chất lượng code" — vô lý, nên nguyên tắc vô dụng. "Chúng ta
chấp nhận trùng lặp code giữa các service để giữ ranh giới độc lập, thay vì tạo shared library dùng
chung" có câu phủ định hợp lý — nên đây là nguyên tắc thật.

**Bước 4 — Viết danh sách "cố tình không làm", 1 giờ.** Đây là mục quyết định chất lượng của cả tài
liệu. Mỗi dòng gồm: việc không làm + lý do + điều kiện xét lại. Nếu bạn không viết được ít nhất 4
dòng, bạn chưa có chiến lược.

**Bước 5 — Định nghĩa tín hiệu để biết chiến lược sai, 1 giờ.** Mọi chiến lược đều dựa trên giả định.
Viết ra 2–3 giả định lớn nhất và tín hiệu quan sát được cho biết giả định đã sai. Đây là điểm phân
biệt một lead cấp senior với một lead cấp staff: người thứ hai viết sẵn điều kiện để mình bị chứng
minh là sai.

**TEMPLATE — Technical Strategy 2 trang**

```markdown
# Technical Strategy — [Tên tổ chức/đơn vị] — [Kỳ: ví dụ H2/2026 → H2/2027]
Owner: [một tên người]        Cập nhật lần cuối: [ngày]
Xét lại: [ngày cụ thể, tối đa 6 tháng nữa]

## 1. Technical Vision (trạng thái đích, 12–24 tháng)
Trong 18 tháng, hệ thống của chúng ta ở trạng thái:
- [Đặc tính 1 — mô tả trạng thái, đo được. VD: "một engineer mới deploy được thay đổi
  lên production trong tuần đầu, không cần ai dựng môi trường hộ"]
- [Đặc tính 2]
- [Đặc tính 3]
North star: [một câu, đo được, và có thể bị vi phạm]

## 2. Diagnosis (hiện trạng, bằng số — không tính từ)
| Chỉ số | Hiện tại | So sánh / ngưỡng đau |
|---|---|---|
| Lead time trung vị (toàn hệ thống) | [x ngày] | [module xấu nhất: y ngày] |
| Change failure rate | [x%] | |
| % effort cho vận hành thủ công | [x%] | |
| Số cách làm [việc phổ biến] | [n] | |
Ràng buộc thật hiện tại: [một câu]

## 3. Guiding Principles (3–5, mỗi cái loại bỏ được lựa chọn)
1. [Nguyên tắc] — vì [lý do]. Hệ quả: [cái gì trở thành không được phép].
2. ...

## 4. Coherent Actions (theo thứ tự, có phụ thuộc)
| # | Hành động | Owner | Kỳ | Xong nghĩa là | Phụ thuộc |
|---|---|---|---|---|---|
| 1 | | | | | |

## 5. CỐ TÌNH KHÔNG LÀM trong kỳ này
| Việc không làm | Vì sao (chi phí cơ hội) | Điều kiện xét lại |
|---|---|---|
| [VD: không migrate sang Kubernetes] | [Cần ~2 người-quý; constraint hiện tại không nằm ở orchestration] | [Khi số service > 25 hoặc chi phí VM vượt X/tháng] |
| [VD: không viết lại module báo cáo] | | |

## 6. Giả định và tín hiệu chiến lược sai
| Giả định | Nếu sai thì sao | Tín hiệu quan sát được | Ai theo dõi |
|---|---|---|---|
| [VD: lượng giao dịch tăng ≤ 3x trong 18 tháng] | Kiến trúc single-DB hết đường | [p99 latency DB > 400ms 3 tuần liên tiếp] | [tên] |

## 7. Cái này KHÔNG phải là gì
- Không phải roadmap (roadmap ở [link]) — tài liệu này không cam kết thời điểm cho tính năng.
- Không phải danh sách công nghệ được phê duyệt (xem Engineering Standards ở [link]).
```

**Phân biệt ba tài liệu — bảng này nên dán đầu mỗi tài liệu để không ai trộn chúng:**

| | Technical Vision | Technical Strategy | Roadmap |
|---|---|---|---|
| Trả lời câu hỏi | Đích trông như thế nào | Đi tới đó bằng chuỗi lựa chọn nào, hy sinh gì | Cái gì xong vào lúc nào |
| Tầm | 12–24 tháng | 6–18 tháng | 1–2 quý |
| Đơn vị nội dung | Trạng thái, đặc tính hệ thống | Nguyên tắc + lựa chọn + việc không làm | Hạng mục + mốc thời gian |
| Ai sở hữu | CTO / Head of Engineering | Principal / Staff + Head of Engineering | EM + PO |
| Thay đổi khi nào | Khi mô hình kinh doanh đổi | Khi constraint đổi hoặc giả định sai | Mỗi sprint/quý |
| Dấu hiệu viết sai | Là danh sách công nghệ | Không có phần hy sinh | Là danh sách ước muốn không có capacity |

### Trade-off

**Trade-off 1 — Vision cụ thể vs vision nguyên tắc.**

| | Vision cụ thể ("18 tháng nữa chúng ta có 8 service theo domain, event bus dùng Kafka, mọi service có SLO") | Vision nguyên tắc ("hệ thống phải cho phép mỗi domain thay đổi độc lập với thời gian chờ dưới 1 ngày") |
|---|---|---|
| Hội tụ quyết định | Nhanh — engineer biết chính xác cần gì | Chậm hơn — cần phán đoán để áp dụng |
| Độ bền | Kém — một thay đổi thị trường hoặc một công nghệ mới làm nó lỗi thời | Cao — nguyên tắc sống qua thay đổi công nghệ |
| Rủi ro | Trở thành mục tiêu tự thân: team hoàn thành "8 service" mà không giải quyết vấn đề gốc | Bị hiểu mỗi người một kiểu, biến thành khẩu hiệu |
| Phù hợp khi | Team dưới 20 người, ít senior, cần hướng dẫn cụ thể; hoặc đang trong một chương trình chuyển đổi có thời hạn | Nhiều senior có phán đoán tốt; nhiều team; miền nghiệp vụ biến động |
| Cách bù nhược điểm | Ghi rõ "8 service là phương tiện, không phải mục tiêu; mục tiêu là X" | Kèm 3–5 ví dụ đã quyết theo nguyên tắc đó, để neo cách hiểu |

Lựa chọn thực tế thường là kết hợp có phân tầng: nguyên tắc ở tầng vision (bền), cụ thể ở tầng
coherent actions của kỳ hiện tại (hội tụ nhanh), và ghi rõ phần cụ thể có ngày hết hạn.

**Trade-off 2 — Strategy tập trung vs strategy nổi lên từ team (top-down vs emergent).** Tập trung cho
tính nhất quán và khả năng đầu tư vào platform dùng chung, nhưng có rủi ro sai vì người viết xa hiện
trường. Emergent cho lựa chọn khớp bài toán cục bộ, nhưng cho ra đúng tình trạng ba mô hình retry ở
đầu chương. Điều kiện nghiêng: dưới 3 team và các team làm cùng một sản phẩm → emergent đủ, chi phí
phối hợp còn thấp. Trên 5 team hoặc có ràng buộc compliance/vận hành chung → cần một tầng tập trung,
nhưng chỉ ở những điểm **giao diện giữa các team** (cách gọi nhau, cách phát event, cách log, cách xác
thực), để nguyên quyền tự quyết bên trong ranh giới team.

**Trade-off 3 — Đầu tư vào runway vs giao tính năng.** Mỗi giờ dựng runway là một giờ không giao tính
năng, và runway chỉ có giá trị nếu các tính năng sắp tới thật sự cần nó. Đây là chỗ chiến lược dễ sai
nhất theo cả hai hướng: dựng runway cho một tương lai không tới (over-engineering), hoặc không dựng gì
rồi mỗi tính năng đều là một dự án hạ tầng. Điều kiện nghiêng về đầu tư runway: cùng một loại hạ tầng
bị dựng lại lần thứ ba; hoặc roadmap 2 quý tới có ≥ 3 hạng mục cùng cần một năng lực chưa có. Điều
kiện nghiêng về hoãn: sản phẩm chưa có product-market fit, nơi xác suất roadmap đó bị đổi hoàn toàn là
đáng kể.

### Real-world Scenarios

**Tình huống A — "Technical strategy" là danh sách công nghệ.** Một startup Series A, 24 engineer, sản
phẩm SaaS B2B. Khoa (CTO) yêu cầu Minh (Tech Lead nền tảng) viết technical strategy cho 2026. Minh
nộp một trang gồm: chuyển sang Kubernetes, áp dụng gRPC nội bộ, đưa event-driven vào các luồng chính,
dựng service mesh, chuẩn hoá observability bằng OpenTelemetry.

Khoa hỏi ba câu và tài liệu sụp: "Vấn đề nào trong số này giải?" — Minh trả lời "khả năng scale", nhưng
số liệu cho thấy traffic hiện tại dùng 18% capacity và không có incident nào do capacity trong 12
tháng. "Nếu làm hết thì tốn bao nhiêu?" — cộng lại khoảng 3,5 người-năm, tương đương 30% năng lực
engineering cả năm. "Chúng ta sẽ không làm gì để có 3,5 người-năm đó?" — không có câu trả lời.

Cái Minh viết là một **wish list công nghệ**, và nó có một đặc điểm nhận dạng: mỗi hạng mục đều là thứ
engineer muốn học. Điều này không xấu về động cơ — nó là tín hiệu team muốn phát triển — nhưng nó
không phải chiến lược, vì nó không xuất phát từ một diagnosis và không loại bỏ gì.

Minh viết lại theo quy trình 5 bước. Diagnosis lộ ra thứ khác hẳn: 34% effort của team đi vào xử lý
thủ công các yêu cầu onboarding khách hàng mới (tạo tenant, cấu hình, import dữ liệu), lead time cho
một khách hàng mới là 9 ngày, và đó chính là ràng buộc chặn tốc độ tăng trưởng doanh thu. Strategy mới
có một guiding principle duy nhất đáng nhớ: "mọi thao tác onboarding khách hàng phải làm được bằng
self-service hoặc bằng một lệnh; không được có bước cần engineer". Danh sách cố tình không làm gồm
Kubernetes, service mesh và gRPC — với điều kiện xét lại rõ ràng.

Kết quả sau 2 quý (số minh hoạ): lead time onboarding từ 9 ngày xuống 1 ngày, effort thủ công từ 34%
xuống 11%, và team giải phóng được khoảng 20% năng lực. Điểm đáng chú ý về mặt leadership: chính phần
"cố tình không làm" là phần Khoa dùng để bảo vệ team trước áp lực từ sales muốn thêm tính năng — vì đã
có văn bản nói rõ đang hy sinh cái gì để lấy cái gì.

**Tình huống B — Ba góc nhìn về cùng một quyết định trong bối cảnh ODC.** Một ODC 60 người ở Đà Nẵng
làm cho khách Nhật. Khách quyết scope và kiến trúc tổng thể. Đội có một hệ thống nội bộ đã tích tụ 4
năm. Linh (EM) đề xuất viết một technical strategy nội bộ, nhưng gặp phản đối: "kiến trúc do khách
quyết, mình viết strategy để làm gì".

*Nhìn từ IC (Nam, mid-level BE):* "Tôi được giao ticket, tôi làm ticket. Tôi không thấy strategy giúp
gì cho việc hôm nay của tôi. Điều tôi thật sự cần là biết khi gặp hai cách làm thì chọn cách nào mà
không bị reviewer bắt sửa lại lần thứ ba." — Nam đang mô tả đúng giá trị của strategy nhưng bằng ngôn
ngữ khác: anh ta cần một cơ chế nén để tự quyết. Với Nam, strategy không phải tài liệu, nó phải hiện
ra dưới dạng nguyên tắc ngắn và template có sẵn.

*Nhìn từ Tech Lead (Minh):* "Phần khách quyết là kiến trúc hệ thống của khách. Phần khách không quyết,
và cũng không quan tâm, lại chính là phần chi phối chi phí của chúng tôi: cách chúng tôi test, cách
dựng môi trường, cách viết tài liệu, cách phát hiện lỗi trước khi khách phát hiện, cách chuyển giao
khi rotate người." Minh nhận ra không gian chiến lược của mình là **không gian giữa yêu cầu của khách
và cách đội thực hiện** — và không gian đó rộng hơn nhiều người tưởng.

*Nhìn từ Manager (Linh):* "Chỉ số tôi bị đo là utilization, số ticket đóng, và số defect khách phát
hiện. Nếu tôi xin 15% capacity cho việc không nằm trong ticket, tôi phải giải thích bằng ngôn ngữ hợp
đồng." Linh viết strategy theo hướng nối trực tiếp với chỉ số của khách: giảm số defect khách phát
hiện, giảm thời gian ramp-up cho người mới rotate vào (khách trả tiền cho thời gian đó), giảm số lần
môi trường lỗi làm chậm nghiệm thu.

Cùng một tài liệu, ba lý do khác nhau để tồn tại. Bài học: strategy chỉ được chấp nhận nếu nó nối
được vào hàm mục tiêu của từng tầng. Với IC, đó là "tôi tự quyết được và không bị làm lại". Với Tech
Lead, đó là "tôi không phải giải lại cùng một tranh luận". Với Manager, đó là "tôi giải thích được chi
phí bằng ngôn ngữ của người trả tiền".

### Best Practices

- **Viết diagnosis trước, và không cho phép mình viết giải pháp trong cùng buổi.** Lý do: khi giải
  pháp đã có trong đầu, diagnosis sẽ bị viết ngược lại từ giải pháp, và bạn mất khả năng phát hiện
  rằng vấn đề thật nằm ở chỗ khác. Tình huống A là ví dụ điển hình.
- **Giới hạn cứng hai trang.** Không phải vì thẩm mỹ. Giới hạn độ dài buộc phải xếp thứ tự ưu tiên, và
  một tài liệu 20 trang thì không ai mang nó theo khi ra quyết định — nên nó không thực hiện được chức
  năng nén.
- **Mỗi nguyên tắc phải kèm hệ quả cấm.** "Nguyên tắc X — hệ quả: từ nay không được làm Y." Nếu không
  viết được hệ quả cấm, nguyên tắc đó chưa đủ sắc để dùng.
- **Gắn tên người vào mọi coherent action, và tên đó là một người, không phải một team.** Chương 04
  giải thích cơ chế: accountability chia cho nhiều người bằng không có accountability.
- **Đặt ngày xét lại trong tài liệu, tối đa 6 tháng.** Chiến lược không có ngày hết hạn sẽ được tuân
  thủ máy móc sau khi đã sai, hoặc bị âm thầm bỏ mà không ai học được gì.
- **Kiểm tra tính coherent bằng bài tập xung đột:** đặt mọi action lên một bảng và với mỗi cặp, hỏi
  "hai cái này có tranh cùng một nguồn lực hay yêu cầu hai hướng trái nhau không". Trong tổ chức nhiều
  team, cặp xung đột phổ biến nhất là "tăng tốc độ giao hàng" và "tăng độ phủ test" khi không ai bỏ
  thứ gì.
- **Công bố strategy ở nơi engineer đang làm việc, không chỉ ở wiki.** Dạng hiệu quả nhất theo kinh
  nghiệm: một file `ARCHITECTURE.md` hoặc `STRATEGY.md` trong repo chính, review qua PR như code. Nó
  được đọc vì nó nằm trên đường đi, và nó được cập nhật vì thay đổi nó là một PR.
- **Kể lại strategy bằng ba câu trong mỗi lần review kiến trúc.** Lặp lại là cơ chế duy nhất làm một
  nguyên tắc trở thành phản xạ. Nếu bạn thấy mình phát mệt vì nhắc lại, đó là dấu hiệu bạn mới bắt đầu
  đủ số lần.

### Anti-patterns

| Anti-pattern | Cơ chế phá hệ thống | Dấu hiệu sớm |
|---|---|---|
| **Vision là danh sách công nghệ muốn dùng** | Công nghệ trở thành mục tiêu tự thân; team hoàn thành hạng mục mà vấn đề gốc còn nguyên; ngân sách bị tiêu vào chỗ không phải constraint | Tài liệu strategy có tên sản phẩm/framework trong tiêu đề của các mục; không ai nêu được vấn đề mà từng hạng mục giải |
| **Strategy không có phần hy sinh** | Không loại bỏ được lựa chọn nào → không định hướng được quyết định phân tán → mỗi team vẫn tự chọn → phân kỳ tiếp tục, nhưng bây giờ có thêm một tài liệu để trích dẫn khi biện minh | Tìm từ "không làm" trong tài liệu, không có; hoặc mọi initiative của mọi team đều "khớp strategy" |
| **Strategy do một người viết trong phòng kín rồi công bố** | Người viết thiếu dữ liệu hiện trường → diagnosis sai; và người thực thi không có sense of ownership → tuân thủ hình thức | Không ai phản biện trong vòng comment; hoặc phản biện chỉ về từ ngữ |
| **Vision đổi mỗi quý** | Chi phí thay đổi kiến trúc lớn hơn một quý, nên không thay đổi nào hoàn thành; team học được rằng cứ chờ là vision sẽ đổi → không ai đầu tư dài hạn nữa | Slide "technical direction" của quý này không nhắc gì tới quý trước |
| **Strategy không gắn được với business** | Bị cắt ngân sách ngay lần đầu công ty gặp áp lực doanh thu, vì không ai bảo vệ được nó bằng ngôn ngữ tiền | Khi trình bày, câu hỏi đầu tiên của CEO là "cái này giúp gì cho doanh thu" và bạn phải suy nghĩ |
| **Copy strategy của Big Tech** | Chiến lược là hàm của constraint; constraint của một công ty 5.000 engineer khác hoàn toàn constraint của một công ty 30 engineer. Mô hình Spotify là ví dụ được sao chép nhiều nhất và bị chính người trong Spotify công khai nói rằng nó chưa từng hoạt động như tài liệu mô tả | Trong tài liệu có tên một công ty nước ngoài nhiều hơn số lần có số liệu nội bộ |

### Khi nào KHÔNG nên áp dụng

**Khi tổ chức dưới khoảng 8–10 engineer và làm một sản phẩm duy nhất.** Ở quy mô này, chi phí phối
hợp còn rất thấp: mọi người nghe được nhau, một cuộc trò chuyện 20 phút thay được một tài liệu. Viết
technical strategy hai trang lúc này tốn 12 giờ để giải quyết một vấn đề chưa xuất hiện. Thứ đáng làm
thay thế: một trang `ARCHITECTURE.md` mô tả hệ thống hiện tại và 3 nguyên tắc, cập nhật khi có gì đổi.

**Khi sản phẩm chưa có product-market fit.** Nếu xác suất mô hình kinh doanh đổi hoàn toàn trong 6
tháng là đáng kể, một vision 18 tháng là ảo tưởng — và tệ hơn, nó sẽ được dùng để biện minh cho việc
đầu tư kiến trúc vào một tương lai không tới. Ở giai đoạn này, "strategy" đúng của bạn gần như chỉ có
một dòng: giữ mọi quyết định ở dạng khả đảo, tối ưu cho tốc độ học, và chấp nhận nợ có ý thức (xem
chủ đề 5, phần "vay debt là quyết định đúng").

**Khi bạn không có quyền phân bổ tài nguyên và cũng không có người bảo trợ.** Chiến lược mà không đi
kèm quyền nói "không" với việc gì là một tài liệu mô tả nguyện vọng. Nếu bạn là Tech Lead không quyết
được capacity, việc đáng làm trước không phải viết strategy toàn tổ chức mà là (a) viết strategy trong
phạm vi bạn kiểm soát thật — cách team bạn test, deploy, review, đặt ranh giới module; và (b) dùng
chương 02 để mua được sự bảo trợ trước khi mở rộng phạm vi.

**Khi tổ chức đang trong một sự cố hoặc một cuộc khủng hoảng chưa xử lý xong.** Trong 6 tuần sau một
incident lớn hoặc trong đợt cắt giảm nhân sự, băng thông chú ý của tổ chức bằng không cho mọi thứ có
tầm dài hơn một tháng. Viết chiến lược lúc này cho ra một tài liệu không ai đọc, và tệ hơn là đốt uy
tín của chính khái niệm "technical strategy" trong tổ chức. Đợi hệ thống ổn định, dùng chính dữ liệu
từ incident làm diagnosis — nó là dữ liệu tốt nhất bạn sẽ có.

**Khi đã có một strategy đúng và chưa hết hạn.** Nghe hiển nhiên nhưng lỗi này rất phổ biến khi có
lead mới: viết lại strategy là cách tự nhiên để đánh dấu lãnh thổ. Chi phí thật là tổ chức phải học
lại một hệ khái niệm mới và mất tất cả tiến độ đang có. Nếu diagnosis cũ vẫn đúng, hãy sửa vài dòng và
để tên người viết cũ ở đó.

---

## 2. Architecture Decision và ADR

### Problem Statement

Một ví điện tử ở Hà Nội, 30 engineer. Tuấn (Senior BE) được giao việc thêm một phương thức thanh toán
mới. Anh mở code và thấy luồng thanh toán đi qua một bảng trung gian `payment_pending` với một cron
chạy mỗi 30 giây để chuyển trạng thái. Thiết kế này rõ ràng kém hơn việc xử lý đồng bộ: nó thêm độ trễ
tới 30 giây, thêm một điểm hỏng, và làm việc debug khó hơn. Tuấn đề xuất bỏ nó. Trong buổi review,
Quân — người làm ở đây 5 năm — nói "đừng chạm vào, cái đó có lý do". Được hỏi lý do gì, Quân trả lời
"tôi không nhớ chính xác, nhưng có liên quan tới đối tác cổng thanh toán".

Ba tuần sau, sau khi đã bỏ cron và deploy, hệ thống mất khả năng xử lý một trường hợp: một cổng thanh
toán cụ thể trả callback hai lần với hai trạng thái khác nhau trong vòng vài giây, và bảng trung gian
chính là cơ chế idempotency thô sơ được dựng để chịu đúng chuyện đó. Sự cố kéo dài 4 giờ, khoảng 1.800
giao dịch cần đối soát thủ công (số minh hoạ).

Chi phí thật ở đây không phải sự cố. Chi phí là **tổ chức đã trả tiền một lần để học một bài học về
đối tác đó, rồi trả lại lần thứ hai vì bài học không được lưu ở đâu.** Đây là hiện tượng trung tâm mà
ADR (Architecture Decision Record) xử lý. Các hiện tượng đếm được khác của cùng một vấn đề:

- **Câu "cái đó có lý do nhưng tôi không nhớ" xuất hiện trong review.** Đếm số lần trong một quý. Mỗi
  lần là một bài học đã mất.
- **Người mới hỏi "vì sao ở đây làm thế này" và không ai trả lời được trong 5 phút.** Đo bằng cách hỏi
  người vào gần nhất: có bao nhiêu chỗ trong hệ thống mà họ được trả lời "lịch sử để lại".
- **Một quyết định kiến trúc bị đảo, rồi bị đảo lại.** Ping-pong kiến trúc là dấu hiệu rõ nhất của việc
  rationale không được bảo tồn: mỗi thế hệ engineer thấy trạng thái hiện tại là vô lý theo tiêu chí của
  mình, và không biết tiêu chí lần trước là gì.
- **Không xác định được ai đã quyết và với giả định nào.** Kiểm tra: chọn ba quyết định kiến trúc quan
  trọng nhất trong hệ thống của bạn, thử tìm ai quyết, khi nào, và các phương án đã loại. Nếu cả ba đều
  không tìm được, bạn đang vận hành một hệ thống mà lý do tồn tại của nó là kiến thức khẩu truyền.

### First Principles

**Cơ chế một: kiến trúc là tập các quyết định khó đảo.** Đây là định nghĩa dùng được nhất, vì nó cho
một phép thử để biết cái gì là "kiến trúc" và cái gì không: nếu đổi được trong một sprint bởi một
người mà không cần ai đồng ý, đó không phải kiến trúc — đó là chi tiết triển khai, và đừng tốn quy
trình cho nó. Hệ quả trực tiếp: **số lượng quyết định kiến trúc trong một hệ thống là hữu hạn và nhỏ**
— thường vài chục cho một hệ thống trung bình, không phải vài trăm. Điều này quan trọng vì nó làm cho
việc ghi lại tất cả trở thành khả thi. Nếu team bạn có 200 ADR trong một năm, phần lớn trong đó không
phải quyết định kiến trúc.

**Cơ chế hai: code thể hiện được cái gì, nhưng không thể hiện được vì sao, và tuyệt đối không thể hiện
được những gì đã bị loại.** Đây là một giới hạn thông tin, không phải vấn đề kỷ luật viết code. Đọc
code, bạn thấy trạng thái cuối. Bạn không thấy: các phương án đã cân nhắc, tiêu chí so sánh, ràng buộc
tại thời điểm quyết (deadline, người có mặt, giới hạn ngân sách hạ tầng, yêu cầu của một đối tác cụ
thể), và những giả định đã tin. Comment trong code không giải quyết được vì nó nằm ở tầng dòng lệnh,
còn rationale nằm ở tầng hệ thống. Suy ra giá trị chính của ADR **không phải là tài liệu hoá kiến
trúc** — có nhiều công cụ sinh diagram tự động tốt hơn — mà là **bảo tồn rationale và không gian
phương án đã bị loại**. Một ADR chỉ ghi "chúng ta dùng PostgreSQL" là gần như vô giá trị; một ADR ghi
"chúng ta dùng PostgreSQL thay vì MongoDB vì luồng bút toán cần transaction đa bảng và team có 2 người
đã vận hành PostgreSQL ở tải tương tự, chấp nhận rằng schema thay đổi sẽ đắt hơn" thì có giá trị trong
5 năm.

**Cơ chế ba: không tồn tại kiến trúc tối ưu, chỉ tồn tại kiến trúc phù hợp với một xếp hạng quality
attribute cụ thể.** Consistency, availability, latency, throughput, chi phí hạ tầng, chi phí vận hành,
time-to-market, khả năng thay đổi, khả năng tuyển người — không thể tối ưu đồng thời. CAP theorem chỉ
là một trường hợp riêng, nổi tiếng vì nó chứng minh được, nhưng nguyên lý tổng quát rộng hơn và quan
trọng hơn với công việc hàng ngày: **mọi lựa chọn kiến trúc là một phép xếp hạng các thuộc tính chất
lượng, và phép xếp hạng đó đến từ business chứ không từ kỹ thuật.** Do đó câu hỏi đầu tiên của một
architecture decision không phải "cách nào tốt hơn" mà là "**thuộc tính nào chúng ta ưu tiên, theo thứ
tự nào, và ai có quyền quyết thứ tự đó**". Rất nhiều tranh luận kiến trúc kéo dài hàng tuần vì hai
người tranh luận đang tối ưu hai thứ tự khác nhau mà không ai nói ra thứ tự của mình.

**Cơ chế bốn: last responsible moment.** Với một quyết định khó đảo, giá trị của việc chờ là thông tin
bổ sung; chi phí của việc chờ là công việc bị chặn và các quyết định khác bị lùi theo. Tồn tại một thời
điểm sau đó việc chờ chỉ còn chi phí mà không còn giá trị — vì thông tin mới không còn tới, hoặc vì đã
có người phải tự quyết trong im lặng để đi tiếp. Đó là last responsible moment. Điểm hay bị hiểu sai:
nó không phải "quyết càng muộn càng tốt". Nó là "**quyết ở thời điểm muộn nhất mà bạn còn giữ được
toàn bộ các phương án**" — sau thời điểm đó, một số phương án đã bị loại bởi thực tế thay vì bởi lựa
chọn của bạn, và đó là mất mát thuần.

### Mental Model

**Fitness function.** Vay từ evolutionary architecture (Neal Ford, Rebecca Parsons, Patrick Kua công
bố khái niệm này rộng rãi): thay vì mô tả kiến trúc đích, bạn định nghĩa các **hàm kiểm tra tự động**
cho các đặc tính kiến trúc mà bạn muốn giữ, rồi để hệ thống tiến hoá miễn là không vi phạm chúng. Ví
dụ cụ thể, đều chạy được trong CI:

| Đặc tính kiến trúc muốn giữ | Fitness function chạy trong CI |
|---|---|
| Module domain không phụ thuộc vào framework web | Test kiểm tra không có import từ package web trong package domain (ArchUnit hoặc tương đương) |
| Không service nào truy cập database của service khác | Kiểm tra connection string trong config; alert khi có kết nối chéo trong môi trường staging |
| p99 của endpoint thanh toán dưới 300ms | Test tải trên pipeline, fail build nếu vượt |
| Không có thư viện nào có CVE mức High | Scan dependency, chặn merge |
| Mọi service phát event theo schema chung | Contract test với schema registry |

Giá trị của mô hình này: nó chuyển ràng buộc kiến trúc từ **văn bản người ta phải nhớ** thành **cơ chế
người ta không thể vi phạm trong im lặng**. Một ADR không có fitness function tương ứng sẽ bị vi phạm
trong vòng 6–12 tháng, và bạn sẽ chỉ biết khi có sự cố.

**Quality attribute trade-off matrix.** Trước khi so sánh phương án, xếp hạng thuộc tính. Bảng minh
hoạ cho quyết định tách service thanh toán:

| Thuộc tính | Thứ tự ưu tiên | Ai quyết thứ tự này | Ngưỡng chấp nhận |
|---|---|---|---|
| Tính đúng của số tiền (correctness) | 1 | CFO + Head of Engineering | Không thoả hiệp; sai lệch đối soát = 0 |
| Availability của luồng thanh toán | 2 | Head of Product | ≥ 99,95% trong giờ cao điểm |
| Khả năng thay đổi độc lập (deployability) | 3 | Head of Engineering | Deploy payment không cần deploy monolith |
| Latency | 4 | Head of Product | p99 ≤ 400ms |
| Chi phí hạ tầng | 5 | CTO | Tăng ≤ 15% |
| Time-to-market của tính năng thanh toán mới | 6 | PO | Không xấu hơn hiện tại |

Khi thứ tự này được viết ra và có tên người phía sau, phần lớn tranh luận kiến trúc rút ngắn được một
nửa — vì tranh luận chuyển từ "cách nào hay hơn" thành "cách nào thoả thứ tự này".

**Last responsible moment như một cửa sổ, không một điểm.** Với mỗi quyết định lớn, viết ra hai ngày:
ngày sớm nhất bạn có đủ thông tin tối thiểu, và ngày mà nếu chưa quyết thì có người bị chặn hoặc một
phương án tự biến mất. Quyết trong cửa sổ đó. Nếu cửa sổ đã đóng mà chưa quyết, bạn không còn ra quyết
định — bạn đang phê chuẩn thứ đã xảy ra.

### Practical Framework

**Quy trình một architecture decision — từ trigger tới ADR merged.**

| Bước | Việc làm | Ai | Thời lượng (minh hoạ) | Output |
|---|---|---|---|---|
| 0. Trigger | Nhận diện đây là quyết định kiến trúc: khó đảo, ảnh hưởng nhiều hơn một module hoặc một team, hoặc chạm dữ liệu/contract | Bất kỳ ai | — | Một dòng trong kênh kỹ thuật: "có thể cần ADR cho X" |
| 1. Phân loại | Áp bảng 4 mức ở chương 04. Mức 1–2 dừng ở đây, ghi ngắn trong PR | Tech Lead | 5 phút | Mức, và ai là người accountable (một tên) |
| 2. Xếp hạng thuộc tính | Viết thứ tự quality attribute và ngưỡng, xác nhận với người có quyền quyết thứ tự | Người accountable + stakeholder business | 30–60 phút | Bảng thứ tự ưu tiên |
| 3. Sinh phương án | Bắt buộc ≥ 3, trong đó một phương án luôn là "không làm gì / giữ hiện trạng" | Người accountable + 1–2 senior | 2–4 giờ | Danh sách phương án, mỗi cái mô tả 5–10 dòng |
| 4. Giảm bất định | Spike/benchmark/prototype chỉ cho các câu hỏi mà kết quả có thể thay đổi lựa chọn | 1 người | Hộp thời gian cứng: 1–5 ngày | Số đo, không phải ý kiến |
| 5. Viết draft ADR | Điền template, gồm cả phương án bị loại và lý do | Người accountable | 1–2 giờ | ADR status: Proposed, mở PR |
| 6. Phản biện bất đồng bộ | Comment trên PR. Thời hạn cứng theo mức: mức 3 là 3 ngày làm việc, mức 4 là 5–10 ngày | Reviewer bắt buộc theo mức | 3–10 ngày | Comment, và các phản biện đã được trả lời trong văn bản |
| 7. Chốt | Người accountable quyết. Nếu không hội tụ, escalate theo đường đã định trước, không mở thêm vòng thảo luận | Người accountable | 1 buổi ≤ 60 phút | ADR status: Accepted |
| 8. Gắn cơ chế thực thi | Thêm fitness function, template, hoặc lint rule tương ứng | Owner platform hoặc chính team | 0,5–2 ngày | Kiểm tra tự động trong CI |
| 9. Merge và phát | Merge ADR vào repo, đăng 3 câu tóm tắt vào kênh chung | Người accountable | 15 phút | ADR có số, có link |

**Tiêu chuẩn một ADR đủ tốt — checklist 8 điểm.** Dùng làm review checklist khi review ADR:

- [ ] **Context có số, không có tính từ.** "Latency hiện tại p99 = 820ms" thay vì "hệ thống chậm".
- [ ] **Có ≥ 2 phương án bị loại, mỗi cái có lý do loại cụ thể**, và lý do đó chiếu vào bảng thứ tự
      thuộc tính, không phải "phức tạp quá".
- [ ] **Nêu rõ ràng buộc tại thời điểm quyết** — deadline, người có sẵn, ngân sách, yêu cầu đối tác.
      Đây là phần cứu người đọc sau 2 năm khỏi kết luận "người trước dốt".
- [ ] **Nêu hệ quả cả hai chiều**, gồm cả cái xấu mà ta chấp nhận. ADR chỉ có hệ quả tốt là ADR bán
      hàng, không phải ADR.
- [ ] **Có điều kiện xét lại**, dạng "khi X xảy ra thì mở lại quyết định này", với X quan sát được.
- [ ] **Có một tên người accountable**, không phải tên team.
- [ ] **Đủ ngắn để đọc trong 10 phút.** Trên 3 trang là dấu hiệu đang trộn ADR với RFC.
- [ ] **Có cơ chế thực thi tương ứng** hoặc ghi rõ "chưa có cơ chế thực thi, dựa vào review thủ công" —
      để người sau biết mức độ đảm bảo.

**TEMPLATE ADR đầy đủ, với ví dụ thật**

```markdown
# ADR-014: Tách service thanh toán ra khỏi monolith

- Status: Accepted
- Ngày: 2026-03-12
- Người accountable: Minh (Tech Lead Payment)
- Reviewer: Khoa (CTO), Duy (Platform), Vy (QA lead), Hà (đại diện team Order)
- Thay thế / bị thay thế bởi: —
- Điều kiện xét lại: xem mục 7

## 1. Context (bằng số)
- Monolith hiện tại: ~310k dòng, 6 team cùng deploy, tần suất deploy 2 lần/tuần theo lịch chung.
- Luồng thanh toán chiếm 9% code nhưng gây 41% incident P1/P2 trong 12 tháng (23/56 incident).
- Mỗi hotfix thanh toán phải deploy toàn bộ monolith: lead time trung vị 6 giờ, 3 người tham gia.
- Yêu cầu audit mới từ đối tác ngân hàng: log truy cập dữ liệu thẻ phải tách khỏi log chung,
  và số người có quyền truy cập production của luồng thanh toán phải ≤ 4 (hiện tại là 19).
- Roadmap 2 quý tới có 3 tính năng thanh toán mới (ví trả sau, QR liên ngân hàng, hoàn tiền tự động).
(Các số trên là số minh hoạ.)

## 2. Thứ tự ưu tiên quality attribute (đã xác nhận với business)
1. Tính đúng của số tiền — không thoả hiệp (Khoa + kế toán trưởng)
2. Giới hạn quyền truy cập dữ liệu thẻ ≤ 4 người — ràng buộc compliance, không thoả hiệp
3. Deploy độc lập luồng thanh toán
4. Availability ≥ 99,95% giờ cao điểm
5. Chi phí hạ tầng tăng ≤ 15%
6. Time-to-market: không xấu hơn hiện tại sau 1 quý

## 3. Quyết định
Tách một service `payment` riêng, sở hữu database riêng, giao tiếp với monolith qua:
- API đồng bộ cho khởi tạo và truy vấn trạng thái giao dịch (idempotency key bắt buộc);
- event `payment.settled` / `payment.failed` phát qua bus hiện có cho các bên tiêu thụ (Order,
  Notification, Report).
Chuyển đổi theo strangler: monolith giữ một adapter mỏng gọi sang service mới; chạy dual-write và
đối soát song song trong 6 tuần trước khi tắt đường cũ. Không migrate dữ liệu lịch sử trong
phase 1; báo cáo cũ tiếp tục đọc từ monolith qua một view read-only.

## 4. Phương án đã cân nhắc và bị loại
### PA1 — Giữ trong monolith, chỉ tách module và siết quyền bằng row-level security
- Ưu: rẻ nhất (~3 người-tuần), không thêm hạ tầng, không thêm điểm hỏng.
- Loại vì: không thoả ràng buộc #2. Ai có quyền deploy monolith thì trên thực tế có đường
  chạm dữ liệu thẻ; đối tác ngân hàng đã nói rõ tách logic trong cùng process không được chấp nhận.
- Điều kiện làm PA1 trở lại hợp lệ: nếu ràng buộc audit được diễn giải lại theo hướng chấp nhận
  tách logic (đã hỏi, câu trả lời bằng văn bản là không).

### PA2 — Tách payment thành 3 service (authorization, settlement, refund) ngay từ đầu
- Ưu: ranh giới sạch hơn theo dài hạn, phù hợp nếu ba luồng có nhịp thay đổi khác nhau.
- Loại vì: team payment hiện có 4 người; theo thứ tự ưu tiên #6, chia 3 service làm chi phí vận
  hành và phối hợp tăng ngay trong khi lợi ích chỉ hiện ra khi team > 8 người. Vi phạm nguyên tắc
  "ranh giới service không nhỏ hơn ranh giới team".
- Điều kiện xét lại: khi team payment ≥ 8 người hoặc khi luồng refund có nhịp release riêng.

### PA3 — Dùng nhà cung cấp payment orchestration bên thứ ba
- Ưu: rút ngắn 2–3 tháng, họ chịu phần compliance.
- Loại vì: phí giao dịch bổ sung ~0,15% trên tổng GMV, vượt ngưỡng #5 khi tính theo khối lượng
  hiện tại; và đưa một phụ thuộc không đảo được vào luồng doanh thu chính (one-way door).
- Điều kiện xét lại: nếu mở thị trường mới cần tích hợp > 8 cổng nội địa của nước đó.

### PA4 — Không làm gì trong kỳ này
- Loại vì: ràng buộc audit có thời hạn (30/06), không đáp ứng thì mất kênh thanh toán qua đối tác
  ngân hàng, tương đương ~22% GMV.

## 5. Hệ quả
Được:
- Deploy độc lập: dự kiến lead time hotfix thanh toán từ 6 giờ xuống dưới 1 giờ.
- Số người truy cập production dữ liệu thẻ giảm về 4, đáp ứng audit.
- Blast radius incident thanh toán không còn kéo theo toàn monolith.
Mất / chấp nhận:
- Thêm một hệ phân tán: cần idempotency, retry, và đối soát giữa hai store. Chi phí vận hành
  tăng, ước lượng +1 on-call rotation.
- Chi phí hạ tầng +11% (trong ngưỡng, nhưng sát).
- Báo cáo xuyên hai nguồn dữ liệu trong ít nhất 2 quý: chấp nhận có một view tạm.
- Trong 6 tuần dual-write, mọi thay đổi luồng thanh toán phải làm ở hai chỗ. Đây là giai đoạn rủi
  ro nhất; đã đóng băng feature thanh toán mới trong 6 tuần đó (đã thống nhất với PO Trang).

## 6. Cơ chế thực thi
- Fitness function: test CI fail nếu monolith còn import package payment nội bộ.
- Kiểm tra tự động: alert nếu có kết nối từ credential monolith tới database payment.
- Contract test cho schema event `payment.*`.

## 7. Điều kiện xét lại quyết định này
Mở lại ADR nếu bất kỳ điều sau xảy ra:
- Sai lệch đối soát giữa hai store > 0 bút toán trong bất kỳ ngày nào của giai đoạn dual-write.
- Chi phí hạ tầng tăng > 20% so với trước khi tách.
- Sau 2 quý, lead time hotfix thanh toán không giảm được xuống dưới 2 giờ (giả định về lợi ích
  chính đã sai).
- Team payment giảm dưới 3 người (không đủ để vận hành một service riêng có on-call).
```

**Xử lý ADR bị vi phạm sau này.** Đây là phần hầu như không tài liệu nào viết, và là nơi ADR mất hiệu
lực trong thực tế. Quy trình bốn bước:

| Bước | Nội dung | Vì sao |
|---|---|---|
| 1. Phân loại vi phạm | (a) Vô tình vì không biết ADR tồn tại; (b) Cố ý vì ADR không còn phù hợp; (c) Cố ý vì tiện hơn trong ngắn hạn | Ba loại cần ba phản ứng khác nhau; xử lý chung là sai |
| 2. Với (a) | Đây là lỗi hệ thống, không phải lỗi người: ADR không ở nơi người ta gặp nó. Sửa bằng cơ chế (lint rule, template, checklist trong PR), không bằng nhắc nhở | Nếu phải nhớ mới tuân thủ, tỉ lệ tuân thủ sẽ giảm theo thời gian và theo số người mới |
| 3. Với (b) | Mở ADR mới thay thế ADR cũ, ghi `Supersedes ADR-0xx`. Không sửa ADR cũ — nó là bản ghi lịch sử | Bảo tồn rationale đòi hỏi bản ghi immutable; sửa tại chỗ là xoá lịch sử |
| 4. Với (c) | Ghi thành một exception có thời hạn và có tên người, vào debt register (chủ đề 5). Nếu không đặt thời hạn, ngoại lệ trở thành tiêu chuẩn mới trong khoảng 2 quý | Ngoại lệ không thời hạn là cơ chế chính làm standard tan rã |

### Trade-off

**Trade-off 1 — Quyết sớm vs quyết muộn.**

| | Quyết sớm | Quyết muộn (tới last responsible moment) |
|---|---|---|
| Được | Mọi việc phía sau không bị chặn; team có một hướng để tối ưu; chi phí phối hợp thấp; nếu đúng thì rất rẻ | Nhiều thông tin hơn: đã thấy tải thật, đã thấy yêu cầu thật, đã có prototype |
| Mất | Quyết bằng giả định; nếu sai thì lượng thứ đã bám vào quyết định (code, dữ liệu, tích hợp) làm chi phí đảo cao | Công việc phía sau bị chặn hoặc phải làm với abstraction tạm; chi phí quyết định trễ tích lũy tuyến tính; có người sẽ tự quyết trong im lặng |
| Nghiêng về khi | Chi phí đảo thấp (có abstraction, ít dữ liệu); hoặc quyết định chặn nhiều người; hoặc thông tin bổ sung không thể có được bằng cách chờ | Chi phí đảo cao và không thể giảm; thông tin quyết định sẽ tới trong thời gian ngắn xác định; có cách đi tiếp mà không cần quyết |
| Kỹ thuật giảm thiệt hại | Viết điều kiện xét lại vào ADR; giữ một lớp abstraction mỏng ở ranh giới | Đặt ngày cứng cho cửa sổ quyết định và người accountable; ghi rõ "quyết định tạm thời" cho các lựa chọn phải làm trong lúc chờ |

Một điểm hay bị bỏ: **chi phí chờ không đối xứng theo vai.** Người có quyền quyết thường không cảm
nhận chi phí chờ, vì họ không bị chặn — người bị chặn là engineer đang có ba PR treo. Đây là một dạng
principal-agent problem nhỏ, và cách xử lý là làm chi phí chờ hiện ra: mỗi quyết định đang chờ có một
dòng ghi "đang chặn: ai, bao nhiêu ngày".

**Trade-off 2 — ADR nhẹ (10 dòng, nhiều cái) vs ADR nặng (3 trang, ít cái).** ADR nhẹ được viết nhiều
hơn nên bắt được nhiều rationale hơn, nhưng thường thiếu phần phương án bị loại — mà đó lại là phần có
giá trị nhất. ADR nặng có chất lượng cao nhưng chi phí viết làm người ta tránh, và kết quả là các quyết
định mức trung bình không được ghi gì cả. Cách xử lý hiệu quả: hai định dạng chính thức — "ADR-lite"
cho mức 2 (10–15 dòng, bắt buộc có một dòng "đã loại gì") và ADR đầy đủ cho mức 3–4.

**Trade-off 3 — Chuẩn hoá quyết định trong ADR vs để code là nguồn sự thật duy nhất.** Lập luận "code
là tài liệu duy nhất không lỗi thời" đúng về tính chính xác của trạng thái, và sai về rationale. Trong
thực tế, ADR sẽ lệch với hệ thống sau 12–18 tháng nếu không ai bảo trì. Điều kiện nghiêng: ADR chỉ cho
các quyết định khó đảo (số lượng nhỏ, ít lỗi thời); diagram và mô tả trạng thái nên sinh tự động từ
code hoặc hạ tầng.

### Real-world Scenarios

**Tình huống A — Architecture astronaut trong một startup 18 người.** Tuấn (Senior BE, mới vào từ một
công ty lớn) đề xuất một ADR: đưa CQRS + event sourcing vào toàn bộ luồng đơn hàng. Lập luận: audit
trail đầy đủ, khả năng replay, tách read/write để scale. Tài liệu 11 trang, có diagram đẹp.

Minh (Tech Lead) không tranh luận về CQRS. Anh hỏi bốn câu theo đúng framework: (1) "Thứ tự quality
attribute của chúng ta cho luồng đơn hàng là gì, và ai xác nhận?" — chưa ai xác nhận. (2) "Vấn đề đo
được nào đang xảy ra?" — không có incident nào liên quan audit trail; số đơn hàng 4.000/ngày, database
hiện dùng 12% capacity. (3) "Ba phương án còn lại là gì?" — tài liệu không có phương án nào khác. (4)
"Ai vận hành nó khi anh nghỉ phép?" — trong 18 người, một người từng làm event sourcing.

Xử lý: Minh không reject. Anh yêu cầu Tuấn viết lại ADR với một thay đổi duy nhất — thêm mục "phương
án đã loại" và mục "hệ quả xấu chấp nhận". Trong quá trình viết, Tuấn tự phát hiện chi phí vận hành và
chi phí onboarding vượt lợi ích ở quy mô hiện tại. Quyết định cuối: giữ mô hình hiện tại, nhưng thêm
một audit log append-only cho các thay đổi trạng thái đơn hàng — giải quyết 80% nhu cầu thật với khoảng
5% chi phí, và ADR ghi rõ điều kiện xét lại event sourcing (khi số đơn > 100k/ngày hoặc khi có yêu cầu
replay từ nghiệp vụ).

Bài học về mặt leadership: cách rẻ nhất để chặn một đề xuất over-engineering không phải là phản biện
kỹ thuật — nó dẫn tới một cuộc tranh luận về sở thích mà người thâm niên hơn thắng. Cách rẻ nhất là
**buộc tài liệu phải đủ 9 mục**; template làm công việc phản biện thay cho bạn, và tác giả tự đi tới
kết luận. Điều này cũng giữ được quan hệ, quan trọng trong bối cảnh Việt Nam khi phản biện trực diện
người mới vào dễ bị đọc là "đánh phủ đầu".

**Tình huống B — Kiến trúc quyết bởi người không chịu hậu quả vận hành.** Một doanh nghiệp truyền
thống chuyển đổi số, phòng IT 25 người. Một Solution Architect thuộc bộ phận khác (không tham gia
on-call, không trực sự cố) quyết mọi kiến trúc và ký duyệt thiết kế. Quyết định gần nhất: mọi tích hợp
nội bộ đi qua một ESB tập trung do một nhà cung cấp bên ngoài vận hành, vì "chuẩn hoá tích hợp".

Sau 8 tháng, hiện tượng đo được: mọi thay đổi tích hợp phải chờ nhà cung cấp, lead time trung vị 12
ngày; 7/11 sự cố P2 trong quý có nguyên nhân trên ESB, mà team không có quyền truy cập log của ESB để
điều tra; MTTR trung vị 5,5 giờ, trong đó 4 giờ là chờ phản hồi từ nhà cung cấp (số minh hoạ).

Cơ chế hỏng ở đây rất rõ và rất phổ biến: **người quyết tối ưu cho thuộc tính mà họ bị đo (tính chuẩn
hoá, tính tuân thủ kiến trúc doanh nghiệp), và không nội tại hoá chi phí vận hành vì họ không trả chi
phí đó.** Đây là externality, đúng nghĩa kinh tế học. Không phải người đó thiếu năng lực; hàm mục tiêu
của người đó không chứa MTTR.

Cách Linh (EM của phòng IT) xử lý, mất 2 quý: không tấn công quyết định ESB. Cô làm ba việc. Một, đo
và công bố hàng tháng một bảng gồm lead time tích hợp, số sự cố theo thành phần, và thời gian chờ nhà
cung cấp — dữ liệu, không ý kiến. Hai, đề nghị một thay đổi quy trình duy nhất: mọi ADR ảnh hưởng tới
luồng production phải có chữ ký của người trong rotation on-call, và người đó có quyền viết một mục
"rủi ro vận hành" không bị xoá. Ba, đề xuất một ngoại lệ có thời hạn cho hai tích hợp nội bộ đi trực
tiếp, kèm cam kết đo và báo cáo so sánh sau 3 tháng.

Sau 3 tháng, so sánh cho thấy hai tích hợp trực tiếp có lead time 2 ngày và không sự cố. Quyết định
kiến trúc được sửa không bằng tranh luận mà bằng một thí nghiệm có đối chứng. Nguyên tắc rút ra và
đáng viết vào standard của tổ chức: **quyền quyết định kiến trúc phải đi kèm việc chịu hậu quả vận
hành; nếu không thể ghép hai thứ đó vào cùng một người, thì tối thiểu người chịu hậu quả phải có quyền
veto có ghi chép.**

### Best Practices

- **Đặt ADR trong repo, review qua PR, đánh số tăng dần, không bao giờ sửa nội dung ADR đã Accepted.**
  Cơ chế: ADR là bản ghi lịch sử; giá trị của nó nằm ở việc phản ánh trạng thái hiểu biết tại thời
  điểm quyết. Thay đổi thì viết ADR mới với `Supersedes`.
- **Bắt buộc phương án "không làm gì" trong mọi ADR.** Lý do: nó là phương án duy nhất luôn có sẵn và
  hay bị bỏ qua, và việc phải viết ra chi phí của việc không làm gì làm lộ ra liệu vấn đề có thật hay
  không.
- **Viết ràng buộc thời điểm vào Context.** Câu "chúng tôi có 5 tuần vì hạn compliance 30/06, và team
  chỉ có 4 người trong đó 1 người mới 2 tháng" là câu cứu người đọc năm 2029 khỏi kết luận sai về
  người quyết năm 2026.
- **Với mỗi ADR Accepted, hỏi ngay "cơ chế nào làm cho việc vi phạm cái này khó?"** Nếu câu trả lời là
  "review thủ công", ghi điều đó vào ADR như một rủi ro đã biết.
- **Giới hạn số ADR mở đồng thời.** Trên 3 ADR đang trong vòng phản biện cùng lúc ở một team, chất
  lượng comment giảm rõ — đây là hiện tượng WIP quá cao, cùng cơ chế với code review (chủ đề 3).
- **Đọc lại 5 ADR gần nhất mỗi quý, kiểm tra hệ thống có còn khớp không.** 30 phút. Kết quả thường là
  phát hiện 1–2 ADR đã bị vi phạm mà không ai biết — và mỗi phát hiện như vậy rẻ hơn nhiều lần so với
  phát hiện nó trong một incident.
- **Với quyết định mức 4, viết pre-mortem trước khi viết ADR.** Chương 04 có quy trình; kết quả của
  pre-mortem trở thành mục "hệ quả xấu chấp nhận" và mục "điều kiện xét lại", nên nó không phải công
  việc thêm.

### Anti-patterns

| Anti-pattern | Cơ chế phá hệ thống | Dấu hiệu sớm |
|---|---|---|
| **Architecture astronaut** | Thiết kế cho một quy mô và một tương lai chưa tồn tại; chi phí trả ngay, lợi ích có thể không bao giờ tới; cognitive load tăng làm team chậm ở mọi thay đổi nhỏ | Tài liệu thiết kế không có số về tải hiện tại; đề xuất chứa từ "sau này khi scale"; không có mục hệ quả xấu |
| **Kiến trúc quyết bởi người không chịu hậu quả vận hành** | Hàm mục tiêu của người quyết không chứa MTTR và chi phí on-call → externality → chi phí dồn sang người trực | Người ký duyệt thiết kế không có tên trong rotation on-call; không ai hỏi "ai debug cái này lúc 2 giờ sáng" |
| **ADR viết sau khi đã code xong** | Trở thành nghi lễ hợp thức hoá; mất hoàn toàn giá trị so sánh phương án vì phương án đã bị loại bởi công việc đã làm | Mục "phương án đã cân nhắc" chỉ có một cái và nó trùng với cái đã triển khai |
| **ADR không có phương án bị loại** | Người đọc sau không biết không gian lựa chọn, nên mở lại toàn bộ tranh luận từ đầu; chính là vòng lặp ở đầu chương 04 | Đếm tỉ lệ ADR có ≥ 2 phương án bị loại; dưới 50% là báo động |
| **ADR không có điều kiện xét lại** | Quyết định đúng-lúc-đó biến thành giáo lý; hoặc bị vi phạm âm thầm vì không ai biết cách thay đổi nó chính thức | Trong tổ chức có câu "cái đó là chuẩn của công ty rồi" mà không ai chỉ được văn bản |
| **Mọi thứ đều cần ADR** | Chi phí quy trình vượt lợi ích; người ta bắt đầu viết ADR hình thức để đi tiếp; chất lượng review sụp vì quá tải | Trên 30 ADR/quý cho một team 8 người; ADR cho các quyết định sửa được trong một ngày |
| **ADR bị chôn trong Confluence không ai tìm được** | Bảo tồn rationale thất bại ở khâu truy xuất; hiệu ứng giống như không có ADR, nhưng tổ chức tưởng mình có | Người mới không tìm được ADR liên quan tới module họ đang sửa trong 2 phút |

### Khi nào KHÔNG nên áp dụng

**Khi quyết định thật sự dễ đảo.** Nếu đổi được trong một sprint bởi một người, viết ADR là thêm 1–2
giờ và một vòng review cho một thứ không cần. Đúng hơn: một dòng trong PR description. Ngưỡng thực tế
dùng được: cần ADR khi chi phí đảo > 1 người-tuần hoặc khi có bên ngoài team phụ thuộc.

**Khi tổ chức chưa có thói quen đọc bất cứ tài liệu nào.** Áp ADR vào một tổ chức mà PR description
còn trống, commit message là "fix", và không ai đọc RFC — sẽ tạo ra một nghi lễ mới bị bỏ sau 2 tháng,
và làm mất uy tín của công cụ. Thứ tự đúng: bắt đầu bằng thứ có chi phí thấp nhất và lợi ích thấy ngay
(PR description có phần "vì sao", hoặc một dòng "đã cân nhắc gì" trong PR), rồi mới nâng lên ADR khi
đã có một trường hợp mà bản ghi đó cứu được ai.

**Trong 1–3 tháng đầu của một prototype sẽ bị bỏ.** Nếu đã quyết trước rằng code này sẽ bị xoá sau khi
kiểm chứng giả thuyết, thì rationale không cần bảo tồn — không có ai trong tương lai để bảo tồn cho.
Điều kiện biên quan trọng: phải **viết ra** rằng đây là prototype sẽ bị bỏ, kèm ngày. Prototype không
được tuyên bố sẽ trở thành production, và khi đó việc không có ADR là chi phí thật.

**Khi bạn đang dùng ADR để né trách nhiệm quyết định.** Có một dạng lạm dụng khó thấy: mở ADR, để mở
vòng comment vô thời hạn, và dùng "đang chờ đồng thuận trên ADR" như lý do để không chốt. ADR là công
cụ ghi lại quyết định, không phải cơ chế sản xuất đồng thuận. Nếu ADR của bạn ở trạng thái Proposed
quá 3 tuần, vấn đề không phải tài liệu — vấn đề là chưa xác định được ai có quyền quyết.

**Khi quyết định thuộc miền mà bạn không có quyền quyết và cũng không có quyền phản biện** — ví dụ ODC
nơi khách chốt kiến trúc bằng văn bản hợp đồng. Ở đây ADR không nên viết ở dạng "chúng tôi quyết"; nó
nên viết ở dạng **decision log ghi nhận ràng buộc bên ngoài**: khách quyết X, chúng tôi đã nêu rủi ro
Y bằng văn bản ngày Z, phương án chúng tôi đề xuất là W và bị loại vì lý do gì. Dạng này có giá trị
khác: nó bảo vệ đội khi hậu quả xảy ra, và nó là dữ liệu để thương lượng ở kỳ hợp đồng sau.

---

## 3. Code Review Culture

### Problem Statement

Một team e-commerce 11 engineer. Đo trong 4 sprint gần nhất (số minh hoạ, nhưng cách đo thì thật và
bạn lấy được từ API của GitHub/GitLab trong một buổi chiều):

| Chỉ số | Giá trị | Hệ quả quan sát được |
|---|---|---|
| Thời gian từ mở PR tới comment đầu tiên (trung vị) | 19 giờ | Tác giả đã chuyển sang việc khác; quay lại phải nạp lại context |
| Thời gian từ mở PR tới merge (trung vị) | 3,2 ngày | Lead time toàn trình 3,2 ngày chỉ để chờ, chưa tính thời gian làm |
| Số dòng thay đổi trung vị mỗi PR | 640 | Vượt xa ngưỡng mà review còn phát hiện được lỗi |
| Tỉ lệ PR có ≥ 1 comment thực chất (không phải style/typo) | 22% | 78% PR được approve mà không ai thật sự đọc logic |
| Tỉ lệ comment về style/format | 61% | Băng thông review bị tiêu vào việc máy làm được |
| Số PR mà Quân (senior thâm niên nhất) là reviewer | 71% | Một người là bottleneck của gần như mọi thay đổi |
| Số lần "approve" trong dưới 2 phút sau khi mở PR | 14 lần/sprint | Rubber-stamp |

Bốn hậu quả có thể truy được từ bảng trên. Một, **lead time bị chi phối bởi thời gian chờ, không phải
thời gian làm** — nghĩa là mọi nỗ lực làm engineer code nhanh hơn đều không thay đổi được đầu ra. Hai,
**review không còn chức năng phát hiện lỗi**, nên change failure rate không giảm dù có quy trình review
đầy đủ. Ba, **kiến thức không lan** — 78% PR không có thảo luận nghĩa là không có truyền đạt gì; team
có 11 người nhưng chỉ có 2–3 người hiểu hệ thống. Bốn, và tệ nhất, **review trở thành nghi lễ**, và khi
một quy trình mất chức năng nhưng vẫn được thực hiện, nó dạy cho cả team rằng quy trình ở đây là hình
thức — hiệu ứng đó lan sang mọi quy trình khác.

### First Principles

**Cơ chế một: code review có bốn mục tiêu khác nhau, chúng cạnh tranh nhau, và phần lớn team không nói
rõ mình đang làm cái nào.** Đây là nguồn gốc của đa số xung đột về review.

| Mục tiêu | Nó tối ưu cho gì | Dạng review phù hợp | Xung đột với |
|---|---|---|---|
| **Phát hiện lỗi** | Giảm change failure rate | PR nhỏ, reviewer hiểu sâu miền đó, tập trung vào failure mode và edge case | Chia sẻ kiến thức (người không biết miền đó review sẽ bắt được ít lỗi) |
| **Chia sẻ kiến thức** | Giảm bus factor, tăng số người hiểu hệ thống | Reviewer là người **chưa** biết vùng code đó; comment dạng câu hỏi; pairing | Tốc độ (chậm hơn) và phát hiện lỗi |
| **Giữ tính nhất quán** | Giảm cognitive load khi đọc code toàn hệ thống | Phần lớn nên tự động hoá: formatter, linter, architecture test | Không xung đột nếu tự động; xung đột nặng với tốc độ nếu làm thủ công |
| **Tạo accountability** | Có người thứ hai biết và đồng ý; giảm quyết định đơn phương ở vùng rủi ro | Reviewer bắt buộc là owner vùng đó; ghi lại quyết định | Tốc độ; và dễ biến thành gatekeeping |

Hệ quả thực hành: **một PR nên nói rõ nó cần loại review nào.** Một hotfix production lúc 2 giờ sáng
cần mục tiêu 1 và 4, không cần 2. Một thay đổi ở module mà chỉ một người hiểu thì mục tiêu 2 quan trọng
hơn mục tiêu 1. Trộn cả bốn vào một quy trình duy nhất rồi mong nó tối ưu tất cả là lý do vì sao review
vừa chậm vừa không bắt được lỗi.

**Cơ chế hai: khả năng phát hiện lỗi của con người giảm theo kích thước batch, và giảm nhanh.** Đây là
kết quả được công bố rộng rãi trong tài liệu về code inspection từ thời Fagan và được các nhà cung cấp
công cụ review (SmartBear công bố một nghiên cứu thường được dẫn trên codebase của Cisco) mô tả nhất
quán: hiệu quả phát hiện lỗi rơi rõ rệt khi thay đổi vượt khoảng **200–400 dòng**, và thời gian review
hiệu quả của một người trong một lượt nằm quanh 60–90 phút. Cơ chế nằm ở giới hạn nhận thức: để tìm
được một failure mode, người review phải giữ đồng thời trong đầu trạng thái, luồng dữ liệu và các nhánh
lỗi — dung lượng working memory hữu hạn, nên khi diff lớn, người review chuyển từ đọc logic sang **quét
bề mặt**, và quét bề mặt chỉ bắt được lỗi bề mặt: đặt tên, format, thiếu null check. Điều này giải thích
tại sao PR lớn lại nhận nhiều comment về style: không phải reviewer thích style, mà đó là tất cả những
gì còn khả thi ở kích thước đó.

**Cơ chế ba: review là một hệ thống hàng đợi, và mọi tính chất khó chịu của review đều là tính chất của
hàng đợi.** Từ queueing theory: thời gian chờ tăng phi tuyến khi mức sử dụng của server (reviewer) tiến
gần 100%. Một team nơi mọi senior đều "full" 95% thời gian sẽ có thời gian chờ review dài không phải vì
họ chậm mà vì hệ thống không có slack. Ba biến điều khiển được: (a) **batch size** — PR nhỏ hơn thì
thời gian phục vụ ngắn hơn và có thể chen vào khe trống; (b) **WIP** — số PR mở đồng thời của mỗi
người, vì mỗi PR mở là một context phải giữ; (c) **số server** — số người có thể review một vùng code,
tức là bus factor của việc review.

**Cơ chế bốn: comment về code bị hệ thần kinh xử lý như phản hồi về bản thân, và mức độ này khác nhau
theo văn hoá.** Không phải vì engineer mong manh. Với người viết code, đoạn code là sản phẩm của phán
đoán của họ trong nhiều giờ; phê phán nó, trong cảm nhận tức thời, khó phân biệt với phê phán năng lực
của họ. Trong bối cảnh Việt Nam, cơ chế này bị cộng thêm hai yếu tố: review công khai trước cả team
chạm vào việc "giữ mặt", và thứ bậc tuổi/thâm niên làm một comment từ người ít tuổi hơn bị đọc là vượt
vai. Đây không phải chuyện nên hay không nên; nó là ràng buộc thực tế mà thiết kế quy trình phải tính
tới, giống như latency mạng.

### Mental Model

**Review như hệ thống hàng đợi.** Ba đại lượng cần đo, và ba đòn điều khiển tương ứng:

| Đại lượng | Cách đo | Đòn điều khiển | Tác dụng dự kiến |
|---|---|---|---|
| Thời gian chờ (queue time) | Trung vị thời gian tới comment đầu tiên | SLA phản hồi + hai khe review cố định mỗi ngày | Giảm thời gian chờ, không cần thêm người |
| Thời gian phục vụ (service time) | Thời gian reviewer thực sự đọc | Giới hạn kích thước PR; PR có mô tả tốt | Giảm 2–3 lần với cùng nội dung |
| WIP | Số PR mở đồng thời / người | Quy tắc "không mở PR mới khi đã có 2 PR đang chờ review của mình" | Giảm context switching, tăng tốc toàn trình |

Điểm phản trực giác nhưng quan trọng nhất: **giảm kích thước PR làm giảm tổng thời gian review dù tổng
số PR tăng.** Vì thời gian review không tỉ lệ tuyến tính với số dòng — nó tăng nhanh hơn, do phải giữ
nhiều context hơn — nên chia một PR 600 dòng thành ba PR 200 dòng thường tốn ít tổng thời gian hơn, và
bắt được nhiều lỗi hơn.

**Review latency → lead time.** Nếu trung vị thời gian chờ comment đầu tiên là 19 giờ và mỗi PR trải
qua 2 vòng, thì chỉ riêng chờ đã là ~38 giờ, tức khoảng 1,5 ngày làm việc cho mỗi thay đổi, bất kể
thay đổi đó lớn hay nhỏ. Với một team làm 40 PR mỗi sprint, đó là một khối thời gian không tạo giá trị
nào. Đây là lý do review latency thuộc nhóm chỉ số mà một Tech Lead nên theo dõi hàng tuần, ngang với
DORA metrics ở chương 06.

**Comment cost model.** Mỗi comment có một chi phí cho tác giả: đọc, hiểu, quyết định, sửa, giải thích.
Nếu chi phí đó không tương xứng với giá trị, tác giả học được rằng review là thuế. Từ đó suy ra nguyên
tắc phân tầng comment: **comment phải tự khai báo mức bắt buộc của nó**, để tác giả biết cái gì phải
sửa và cái gì được quyền bỏ qua mà không mất lòng ai.

### Practical Framework

**TEMPLATE — Code Review Guideline của team** (đặt trong repo, review qua PR như code)

```markdown
# Code Review Guideline — Team [tên]
Phiên bản: [x.y] — Cập nhật: [ngày] — Owner: [tên]

## 1. SLA thời gian phản hồi
| Loại PR | Nhãn | Comment đầu tiên trong | Ghi chú |
|---|---|---|---|
| Hotfix production | `hotfix` | 30 phút (giờ làm việc) | Được phép review-after-merge nếu đang có incident |
| Chặn người khác | `blocking` | 4 giờ làm việc | Tác giả phải ghi rõ đang chặn ai |
| Thường | — | 24 giờ làm việc | |
| Lớn / kiến trúc | `deep-review` | 48 giờ, và hẹn 1 buổi 45 phút | Cần trao đổi trước khi viết code |
Hai khe review cố định mỗi ngày: 09:30–10:15 và 15:30–16:15. Ngoài hai khe này không ai bị
kỳ vọng phải trả lời ngay (bảo vệ deep work — xem 01-self-leadership.md).

## 2. Kích thước PR
- Mục tiêu: ≤ 250 dòng thay đổi (không tính file sinh tự động, lock file, migration đơn giản).
- Cứng: > 600 dòng thì reviewer có quyền yêu cầu chia nhỏ, trừ 3 ngoại lệ: đổi tên/di chuyển
  file cơ học, cập nhật dependency, code sinh tự động. Ghi ngoại lệ vào PR description.
- Nếu không chia nhỏ được vì lý do kỹ thuật: chuyển sang pairing hoặc walkthrough 30 phút,
  không review bất đồng bộ.

## 3. Phân tầng comment — BẮT BUỘC gắn tiền tố
| Tiền tố | Nghĩa | Tác giả phải làm gì |
|---|---|---|
| `[blocking]` | Không merge được nếu chưa xử lý: bug, lỗi bảo mật, mất dữ liệu, vi phạm ADR, thiếu test cho luồng tiền | Phải sửa hoặc phải thuyết phục được reviewer |
| `[nên sửa]` | Cải thiện thật nhưng không chặn: cấu trúc, đặt tên khó hiểu, thiếu log | Sửa trong PR này hoặc tạo issue có link, ghi rõ chọn cái nào |
| `[nit]` | Sở thích cá nhân, chi tiết nhỏ | Được quyền bỏ qua, không cần trả lời |
| `[hỏi]` | Reviewer muốn hiểu, không phê phán | Trả lời một dòng là đủ; nếu câu hỏi này lặp lại → thiếu comment/tài liệu |
| `[praise]` | Ghi nhận một cách làm tốt để người khác học | Không cần làm gì |
Comment không có tiền tố được coi là `[nit]`.

## 4. Ai review cái gì
| Loại thay đổi | Reviewer bắt buộc | Reviewer khuyến khích thêm |
|---|---|---|
| Trong một module thông thường | 1 người bất kỳ trong team | — |
| Luồng tiền / bút toán / đối soát | 1 owner module payment + 1 người khác | Vy (QA) nếu đổi hành vi |
| Schema database, migration | 1 owner service + Duy (Platform) | |
| Thay đổi contract API mà team khác dùng | 1 người của team tiêu thụ | |
| Cấu hình hạ tầng, IAM, secret | Duy (Platform), bắt buộc | |
| Vùng code chỉ 1 người hiểu (bus factor = 1) | Người hiểu + 1 người CHƯA hiểu (mục tiêu chia sẻ kiến thức) | |

## 5. Khi nào dùng pairing thay vì review bất đồng bộ
- Thay đổi > 600 dòng không chia nhỏ được.
- Người mới trong 4 tuần đầu, cho PR đầu tiên vào mỗi module mới.
- Đã qua 2 vòng review mà vẫn còn bất đồng: dừng comment, gọi 20 phút.
- Hotfix trong lúc có incident: pairing luôn, review sau.
- Thay đổi mà tác giả tự nói "tôi không chắc cách này đúng": pairing rẻ hơn nhiều lần.

## 6. Nguyên tắc cho reviewer
- Không comment những gì công cụ làm được. Nếu bạn định comment về format → sửa cấu hình
  formatter, đó là PR khác.
- Comment về code, dùng chủ ngữ là code: "hàm này" thay vì "anh".
- Với `[blocking]`, phải nêu failure mode cụ thể: "nếu callback tới 2 lần thì bút toán ghi 2 lần"
  thay vì "cái này không an toàn".
- Đề xuất, không ra lệnh: kèm một lựa chọn cụ thể hoặc một link.
- Nếu bạn không hiểu ý định, dùng `[hỏi]` trước khi dùng `[blocking]`.
- Approve khi PR làm hệ thống tốt hơn trạng thái hiện tại, không phải khi nó hoàn hảo.

## 7. Nguyên tắc cho tác giả
- PR description có 4 phần: làm gì / vì sao / đã cân nhắc gì khác / test thế nào.
- Tự review PR của mình trước khi giao, và để lại comment ở những chỗ mình biết là đáng ngờ.
- Không tranh luận quá 2 lượt trong comment. Lượt 3 chuyển sang gọi.
- Không mở PR mới khi đã có 2 PR của mình đang chờ review.

## 8. Đo và xét lại guideline này
Theo dõi hàng tuần: trung vị thời gian tới comment đầu tiên; % PR > 600 dòng; % PR không có
comment thực chất; phân bố reviewer (không ai > 40% tổng số PR). Xét lại guideline mỗi quý.
```

**Quy trình đưa guideline vào một team đang có văn hoá review kém — 4 tuần:**

| Tuần | Việc làm | Vì sao theo thứ tự này |
|---|---|---|
| 0 | Đo và công bố bảng số liệu hiện trạng (như bảng ở Problem Statement). Không kèm đánh giá ai | Dữ liệu tạo sự đồng thuận về vấn đề trước khi thảo luận giải pháp; và nó chuyển vấn đề từ "ai kém" thành "hệ thống của chúng ta thế nào" |
| 1 | Tự động hoá toàn bộ style: formatter chạy trong pre-commit, linter chặn merge. Xoá mọi comment style khỏi phạm vi review | Đây là bước rẻ nhất, không ai phản đối, và nó giải phóng 61% băng thông review ngay lập tức |
| 2 | Đưa phân tầng comment (`[blocking]`/`[nên sửa]`/`[nit]`/`[hỏi]`) và SLA. Lead làm mẫu 20 comment đầu | Phân tầng là thay đổi có tỉ lệ lợi ích/chi phí cao nhất về mặt quan hệ: nó giải quyết phần lớn cảm giác bị tấn công |
| 3 | Đưa giới hạn kích thước PR. Bắt đầu bằng mục tiêu, không bằng chặn cứng | Đây là thay đổi khó nhất vì nó buộc đổi cách chia việc, không chỉ đổi cách review |
| 4 | Đổi phân bố reviewer: gán chủ đích người chưa biết vùng code vào review, đặt trần 40% cho một người | Chỉ làm được sau khi PR đã nhỏ và có phân tầng, nếu không thì người mới review sẽ vô hiệu |

### Trade-off

**Trade-off 1 — Review kỹ vs tốc độ giao hàng.**

| | Review kỹ (2 reviewer, đọc sâu, blocking nhiều) | Review nhanh (1 reviewer, ngưỡng approve thấp) |
|---|---|---|
| Được | Change failure rate thấp hơn; kiến thức lan; chuẩn mực được giữ | Lead time ngắn; WIP thấp; tác giả giữ được context |
| Mất | Lead time dài; senior thành bottleneck; tác giả mất context giữa các vòng | Lỗi lọt ra production; nợ tích tụ; kiến thức không lan |
| Nghiêng về khi | Luồng tiền, dữ liệu không tạo lại được, contract public, compliance; hoặc team có nhiều người mới | Thay đổi khả đảo, có feature flag, có test tự động tốt, blast radius nhỏ |
| Cách có cả hai | Phân tầng theo blast radius chứ không theo team: cùng một team dùng hai chế độ review khác nhau cho hai loại thay đổi. Đầu tư vào test tự động và feature flag để **hạ chi phí sai**, thay vì đầu tư vào review để hạ xác suất sai |

Điểm cần nói thẳng: ở phần lớn team Việt Nam mà tôi thấy có vấn đề review, nút thắt không phải "review
chưa đủ kỹ" mà là **PR quá lớn cộng thời gian chờ quá dài**. Sửa hai biến đó cho kết quả tốt hơn mọi
nỗ lực yêu cầu người ta "review cẩn thận hơn", vì nó thay đổi điều kiện thay vì thay đổi ý chí.

**Trade-off 2 — Reviewer là người hiểu sâu vs người chưa hiểu.** Người hiểu sâu bắt được nhiều lỗi hơn
nhưng không học được gì và bus factor không đổi; người chưa hiểu bắt ít lỗi nhưng làm bus factor tăng
và thường đặt được những câu hỏi mà người quen tay không còn thấy. Điều kiện nghiêng: vùng code có bus
factor = 1 và không đang trong giai đoạn khẩn cấp → chọn người chưa hiểu, chấp nhận chậm hơn, coi đó là
đầu tư. Vùng luồng tiền hoặc đang chuẩn bị campaign lớn → chọn người hiểu sâu.

**Trade-off 3 — Đồng thuận về style vs tự do cá nhân.** Đây là trade-off giả ở phần lớn trường hợp, và
đáng nêu vì nó tiêu rất nhiều năng lượng: chọn một formatter, chấp nhận mặc định của nó, đưa vào
pre-commit, và không bao giờ bàn lại. Chi phí của việc thống nhất thấp; chi phí của việc tranh luận
hàng tuần cao. Điều kiện duy nhất nghiêng về tự do: các codebase độc lập hoàn toàn, không ai đọc chéo.

### Real-world Scenarios

**Tình huống A — Reviewer 26 tuổi review code của người 41 tuổi.** Bối cảnh: một công ty logistics,
team backend 9 người. Hà (Senior FE chuyển sang BE, 26 tuổi, vào công ty 2 năm) là owner module định
tuyến. Quân (41 tuổi, 12 năm kinh nghiệm, mới vào 3 tháng) mở một PR 480 dòng vào module đó, trong đó
có một vòng lặp gọi API bên ngoài không có timeout và không có giới hạn số lần retry.

Hà gửi comment (phiên bản nói sai):

> "Đoạn này viết sai rồi. Gọi API trong for loop mà không có timeout là kiến thức cơ bản. Anh xem lại
> phần retry, hình như anh chưa hiểu cách service này hoạt động. Em reject PR nhé."

Bốn lỗi trong bốn câu, và mỗi lỗi có một cơ chế riêng. "Viết sai rồi" là phán xét về người viết chứ
không mô tả failure mode. "Kiến thức cơ bản" là một câu đánh vào năng lực — nó không thêm thông tin
nào và trong bối cảnh chênh lệch tuổi thì nó gần như đảm bảo một phản ứng phòng vệ. "Hình như anh chưa
hiểu" là suy diễn về trạng thái nhận thức của người khác từ một đoạn code. "Em reject PR nhé" đặt hành
động ở cuối như một tuyên bố quyền lực mà không có đường đi tiếp.

Kết quả thực tế: Quân không trả lời trên PR. Anh nhắn riêng cho Linh (EM) rằng "team ở đây không tôn
trọng kinh nghiệm". Ba tuần sau, Quân ngừng review PR của người khác và chỉ làm những task được giao.
Chi phí thật của bốn câu đó không phải một cuộc tranh cãi — nó là mất một người có 12 năm kinh nghiệm
khỏi cuộc đối thoại kỹ thuật của team.

Cùng nội dung kỹ thuật, phiên bản nói đúng:

> `[blocking]` Chỗ này em thấy một failure mode cụ thể: nếu API `partner-geo` treo (đã xảy ra 2 lần
> trong tháng 4, xem INC-231), vòng lặp ở dòng 88 sẽ giữ connection cho tới khi hết timeout mặc định
> của client là 60 giây, và với 200 điểm giao thì worker này chặn cả queue. Em đề xuất hai cách: (a)
> đặt timeout 3 giây ở client và retry tối đa 2 lần với backoff, (b) đổi sang gọi batch qua endpoint
> `/geo/batch` mà bên họ mới mở tháng trước. Em nghiêng về (b) vì mình giảm được 200 request xuống 4,
> nhưng (a) làm nhanh hơn nếu anh đang cần merge trước thứ Năm.
>
> `[hỏi]` Phần cache ở dòng 130 — anh chọn TTL 15 phút, em đoán là để khớp với chu kỳ cập nhật của
> đối tác, đúng không ạ? Nếu đúng thì em muốn ghi một comment ở đó, vì lần trước có người sửa xuống 1
> phút và làm tăng chi phí API.
>
> `[nit]` Tên biến `tmpList` ở dòng 45 — không chặn, anh bỏ qua được.

Bốn thay đổi cơ chế, và đây là phần đáng học chứ không phải cách nói cho êm:

| Cơ chế | Phiên bản sai | Phiên bản đúng | Tác dụng |
|---|---|---|---|
| Chủ ngữ | "anh" (người) | "chỗ này", "vòng lặp ở dòng 88" (code) | Tách phê phán khỏi con người, giảm phản ứng phòng vệ |
| Bằng chứng | "kiến thức cơ bản" | failure mode cụ thể + số incident thật | Chuyển từ tranh luận thẩm quyền sang tranh luận dữ liệu — và trong tranh luận dữ liệu, tuổi tác không có trọng số |
| Đường đi tiếp | "reject nhé" | hai phương án, có nêu điều kiện chọn từng cái | Reviewer nhận một phần chi phí sửa; tác giả giữ quyền quyết |
| Mức bắt buộc | không rõ, tất cả như nhau | `[blocking]` / `[hỏi]` / `[nit]` tách bạch | Tác giả biết chính xác phải làm gì; comment nhỏ không còn cảm giác như phán xét |

Một chi tiết về bối cảnh Việt Nam đáng nói thẳng: phiên bản đúng vẫn dùng "anh"/"em" và vẫn giữ tôn
trọng thứ bậc tuổi, nhưng **không nhượng bộ về nội dung kỹ thuật**. Hai thứ đó không xung đột. Cái làm
comment hiệu quả không phải là bớt thẳng thắn — mà là dồn toàn bộ sự thẳng thắn vào failure mode và
toàn bộ sự mềm mại vào cách nói về người. Nhiều team Việt làm ngược: mềm về failure mode ("cái này có
thể chưa ổn lắm...") và vô tình cứng về người.

**Tình huống B — Văn hoá approve cho nhanh, và ba góc nhìn.** Bối cảnh: một startup fintech 16
engineer, cuối quý, sức ép giao hàng cao. Nam (mid BE) mở một PR 520 dòng lúc 17:30. Trong 4 phút,
Quân approve với comment "LGTM". Hai ngày sau, một bug trong PR đó gây sai số dư hiển thị cho khoảng
900 user trong 3 giờ.

*Nhìn từ IC (Nam):* "Tôi mở PR, có người approve, tôi merge. Nếu quy trình nói một approve là đủ thì
tôi đã làm đúng quy trình. Điều tôi thật sự cần là biết PR này có rủi ro và ở đâu — nhưng chính tôi
cũng không biết, đó là lý do tôi giao đi review." Nam không sai theo quy trình, và đây là điểm quan
trọng: nếu quy trình cho phép một kết quả xấu, thì kết quả xấu là thuộc tính của quy trình.

*Nhìn từ Tech Lead (Minh):* "Tôi nhìn thấy ba thứ hỏng cùng lúc. Một, PR 520 dòng vào luồng số dư —
đây là loại thay đổi đáng lẽ phải có hai reviewer trong đó một người là owner module tiền. Hai, approve
trong 4 phút cho 520 dòng là một tín hiệu đo được, và tôi đã có tín hiệu đó trong dashboard từ hai
tháng trước nhưng chưa hành động. Ba, Quân approve nhanh vì đang bị đo bằng số ticket đóng, và review
không nằm trong cái được đo. Nếu tôi chỉ nói với Quân 'anh review cẩn thận hơn', tôi đang yêu cầu anh
ấy hy sinh chỉ số của mình cho một thứ không được ghi nhận. Việc của tôi là sửa cái được đo, không phải
sửa thái độ."

*Nhìn từ Manager (Linh):* "Chi phí của incident này khoảng 6 giờ người xử lý, 40 ticket support, và
một mức mất tin cậy không đo được. Nhưng câu hỏi tôi phải trả lời với CTO không phải 'ai sai' — mà là
'cơ chế nào đã cho phép nó xảy ra, và chúng ta đổi cơ chế nào'. Tôi cũng phải thấy phần của mình: quý
này tôi đã nói với team rằng ưu tiên số một là giao đúng hạn, ba lần, trong ba buổi khác nhau. Team
nghe đúng thứ tôi nói. Nếu tôi muốn review được coi trọng thì review phải xuất hiện trong định nghĩa
'xong' và trong tiêu chí promotion, không chỉ trong lời nhắc."

Ba góc nhìn cho ba hành động khác nhau và tất cả đều cần: Nam cần một checklist rủi ro để tự đánh dấu
PR; Minh cần đổi luật reviewer bắt buộc theo vùng code và đặt cảnh báo cho approve-dưới-5-phút trên PR
lớn; Linh cần đổi cái được đo và được thưởng. Nếu chỉ làm một trong ba, hiện tượng sẽ trở lại trong
một quý.

### Best Practices

- **Tự động hoá mọi thứ về style trước khi làm bất cứ gì khác về review.** Cơ chế: nó giải phóng băng
  thông nhận thức của reviewer cho việc mà chỉ con người làm được — failure mode, ý định, ranh giới
  module. Đây cũng là thay đổi duy nhất trong chương này không ai phản đối.
- **Phân tầng comment bằng tiền tố, và lead phải là người dùng nó đầu tiên và nhiều nhất.** Cơ chế:
  quy tắc được học qua bắt chước, không qua đọc. 20 comment mẫu hiệu quả hơn một buổi họp phổ biến.
- **Đặt SLA phản hồi và bảo vệ nó bằng khe thời gian cố định, không bằng "hãy review sớm".** Cơ chế:
  không có khe thời gian thì review luôn là việc ưu tiên thấp nhất của mọi người, vì nó là việc của
  người khác.
- **Với `[blocking]`, buộc phải nêu được failure mode.** Cơ chế: nó lọc các blocking dựa trên sở thích,
  và nó dạy cả team cách suy nghĩ về hệ thống. Nếu reviewer không nêu được failure mode, comment đó
  xuống mức `[nên sửa]`.
- **Đặt trần cho tỉ lệ PR mà một người review (ví dụ 40%).** Cơ chế: chống bottleneck và chống việc một
  người trở thành nguồn phán xét duy nhất — điều làm những người khác ngừng phát triển phán đoán kỹ
  thuật của họ.
- **Chuyển sang gọi sau vòng comment thứ hai.** Cơ chế: bất đồng kéo dài trên văn bản gần như luôn là
  bất đồng về giả định hoặc về mục tiêu, và văn bản bất đồng bộ là phương tiện tệ nhất để phát hiện
  điều đó. 20 phút gọi thay được 40 comment.
- **Ghi nhận công khai review tốt, không chỉ code tốt.** Cơ chế: hành vi được thưởng là hành vi được
  lặp lại (chủ đề 7). Một comment review bắt được failure mode nghiêm trọng đáng được nhắc trong
  retrospective ngang với một tính năng được giao.
- **Với người mới, PR đầu tiên vào mỗi module nên là pairing.** Cơ chế: nó truyền được các quy tắc
  không viết ra được, và nó rẻ hơn ba vòng review bất đồng bộ.

### Anti-patterns

| Anti-pattern | Cơ chế phá hệ thống | Dấu hiệu sớm |
|---|---|---|
| **Rubber-stamp approval ("LGTM" trong 2 phút cho 500 dòng)** | Review mất chức năng phát hiện lỗi nhưng vẫn tiêu thời gian và vẫn tạo cảm giác an toàn giả; change failure rate không giảm dù quy trình đủ | Đo thời gian từ mở PR tới approve chia cho số dòng; % approve không có comment nào |
| **Nitpick về style trong khi bỏ qua failure mode** | Băng thông review bị tiêu vào việc máy làm được; tác giả học rằng review là thuế hình thức nên ngừng đọc kỹ comment | % comment về format > 40%; không có comment nào nhắc tới edge case hoặc trạng thái lỗi |
| **Một người là bottleneck của mọi PR** | Thời gian chờ tăng phi tuyến khi người đó tiến gần 100% tải; kiến thức tập trung thay vì lan ra; người đó burnout và trở thành điểm hỏng của cả tổ chức | Một người review > 50% PR; khi người đó nghỉ phép, số PR merge trong tuần giảm rõ |
| **Comment nhắm vào người thay vì code** | Kích hoạt phòng vệ; trong bối cảnh trọng thứ bậc, dẫn tới rút khỏi thảo luận thay vì tranh luận; mất người giỏi khỏi đối thoại kỹ thuật | Comment có chủ ngữ là "anh/em/bạn"; xuất hiện các từ "cơ bản", "hiển nhiên", "sao lại"; tác giả trả lời một dòng rồi im |
| **PR khổng lồ như thói quen** | Vượt ngưỡng nhận thức nên review chuyển sang quét bề mặt; đồng thời làm rollback khó và bisect vô dụng khi có sự cố | Trung vị số dòng/PR > 400; có PR mở quá 5 ngày |
| **Review là cửa xin phép (gatekeeping)** | Reviewer dùng approve như đòn bẩy để áp sở thích thiết kế của mình; tác giả học cách viết code làm hài lòng reviewer thay vì làm đúng | Cùng một reviewer yêu cầu sửa lại cùng một kiểu ở nhiều PR khác nhau mà không có tài liệu nào nói thế là chuẩn |
| **Approve để giữ hoà khí** | Tránh xung đột ngắn hạn tạo ra chi phí dài hạn trên hệ thống; và nó phá luôn cả tín hiệu approve — approve không còn nghĩa gì | Hỏi riêng một người: "có PR nào anh approve mà thật ra thấy chưa ổn không?" Nếu câu trả lời là có và nhiều, vấn đề là Psychological Safety, không phải kỹ năng review |

### Khi nào KHÔNG nên áp dụng

**Trong lúc đang xử lý một incident P1.** Review bất đồng bộ trong incident là chi phí thuần: mỗi phút
chờ là mỗi phút hệ thống còn hỏng. Chế độ đúng: pairing trực tiếp (hai người cùng nhìn màn hình, một
người gõ), merge, và **review sau** trong vòng 24 giờ với một PR follow-up nếu cần. Điều kiện để chế độ
này an toàn: nó phải được viết ra trước trong guideline, có tiêu chí rõ "khi nào được dùng", và luôn có
một bước review sau — nếu không, "đang gấp" sẽ trở thành trạng thái thường trực.

**Với prototype và spike sẽ bị xoá.** Nếu code sinh ra để trả lời một câu hỏi rồi bị bỏ, review nó là
tối ưu một thứ không tồn tại. Điều kiện biên: phải có cơ chế đảm bảo nó thật sự bị xoá — nhánh riêng,
không merge vào main, hoặc một ngày hết hạn. Prototype được merge lặng lẽ vào production là một trong
những nguồn technical debt reckless-inadvertent phổ biến nhất (chủ đề 5).

**Khi thay đổi là cơ học và có công cụ đảm bảo.** Đổi tên bằng refactoring của IDE trên 3.000 dòng,
cập nhật copyright header, chạy formatter lần đầu trên toàn repo — review từng dòng ở đây tạo ra một
diff không đọc được và không bắt được gì. Cách đúng: review **lệnh đã chạy và cách kiểm chứng**, không
review diff; và tách các thay đổi cơ học ra khỏi các thay đổi logic thành hai PR riêng.

**Khi team chỉ có 2 người và cả hai làm cùng một phần.** Ở quy mô này, pairing hoặc trao đổi trực tiếp
có chi phí thấp hơn và băng thông cao hơn review bất đồng bộ. Áp một quy trình review đầy đủ với SLA và
phân tầng comment ở đây là nghi lễ. Cái đáng giữ dù chỉ có 2 người: PR description có phần "vì sao", và
một lượt đọc nhanh của người kia trước khi merge cho các thay đổi vào vùng rủi ro.

**Khi mục tiêu thật là đánh giá năng lực cá nhân.** Đây là một dạng lạm dụng cần nói rõ: dùng code
review làm dữ liệu cho Performance Review sẽ phá cả hai. Review trở thành sân khấu — tác giả che khuyết
điểm, reviewer nhẹ tay để giữ quan hệ hoặc nặng tay để ghi điểm — và cả hai chức năng phát hiện lỗi và
chia sẻ kiến thức đều chết. Nếu bạn cần dữ liệu về năng lực, lấy nó từ những nguồn được thiết kế cho
việc đó (chương 08), và tuyên bố rõ rằng review không phục vụ mục đích đánh giá.

---

## 4. RFC Process và Architecture Review

### Problem Statement

Một công ty SaaS 55 engineer, 7 team. Đếm trong một quý (số minh hoạ, cách đếm thì lấy được từ
calendar): 34 buổi họp có từ "architecture", "design" hoặc "technical discussion" trong tiêu đề, trung
bình 6,2 người tham dự, trung bình 74 phút. Tổng khoảng 260 người-giờ. Trong 34 buổi đó, 9 buổi bàn lại
một chủ đề đã bàn trước đó trong cùng quý.

Bảng dưới đây là các hiện tượng cụ thể của một tổ chức "quyết định bằng họp" thay vì "quyết định bằng
văn bản":

| Hiện tượng | Cách đo | Cơ chế phía sau |
|---|---|---|
| Cùng chủ đề mở lại nhiều lần | Đếm chủ đề xuất hiện ở ≥ 2 buổi cách nhau > 2 tuần | Không có bản ghi lập luận, nên mỗi người mới hoặc mỗi tình huống mới đều mở lại từ đầu |
| Người quyết định là người nói nhiều nhất | Ghi âm hoặc quan sát: phân bố thời lượng nói | Trong họp, băng thông là tài nguyên khan hiếm và nó phân bổ theo mức tự tin và địa vị, không theo chất lượng lập luận |
| Người ở múi giờ khác hoặc không nói tốt tiếng Anh không tham gia được | Đếm số lần các thành viên đó phát biểu trong họp so với trong văn bản | Họp đồng bộ loại bỏ những người có băng thông nói thấp nhưng băng thông viết cao — trong bối cảnh ODC, đây thường là những người hiểu hệ thống nhất |
| Quyết định "chốt trong họp" bị triển khai khác nhau | So sánh biên bản với code sau 1 tháng | Ngôn ngữ nói mơ hồ hơn ngôn ngữ viết; ba người nghe cùng một câu ra ba mô hình khác nhau |
| Người phản đối im lặng trong họp rồi phản đối sau | Quan sát: có ai nêu vấn đề sau khi đã "chốt" | Chi phí phản đối trong họp cao (phải phản đối trước mặt nhiều người, ngay lập tức, không có thời gian chuẩn bị) — đặc biệt trong văn hoá tránh xung đột trực diện |

Hậu quả kép: tổ chức vừa tiêu rất nhiều thời gian đồng bộ, vừa không tích lũy được gì. Và có một hậu
quả thứ ba ít được nhắc: **tổ chức không viết thì tổ chức không suy nghĩ được các ý phức tạp.** Có một
giới hạn về độ phức tạp của lập luận mà con người theo được bằng lời nói trong một cuộc họp — khoảng
2–3 lớp điều kiện. Một trade-off kiến trúc thật thường có 5–6 lớp. Không viết ra, tổ chức tự giới hạn ở
những quyết định đơn giản, và với các quyết định phức tạp thì nó chọn theo trực giác của người có địa
vị cao nhất.

### First Principles

**Cơ chế một: văn bản làm cho bất đồng trở nên khả thi về mặt chi phí.** So sánh chi phí phản biện
trong hai phương tiện:

| | Trong họp | Trên văn bản |
|---|---|---|
| Thời gian để hình thành phản biện | Vài giây | Vài giờ tới vài ngày |
| Khán giả tức thời | 5–10 người, có cả cấp trên | Không ai xem lúc bạn viết |
| Chi phí xã hội nếu phản biện sai | Cao, ngay lập tức, trước mặt mọi người | Thấp; sửa comment được |
| Yêu cầu về kỹ năng nói / ngoại ngữ | Cao | Thấp hơn nhiều |
| Khả năng phản biện các lớp lập luận sâu | Thấp | Cao — trích dẫn được từng đoạn |

Suy ra: **RFC không phải là công cụ tài liệu, nó là công cụ hạ chi phí của việc không đồng ý.** Đây là
lý do RFC có tác động lớn hơn hẳn trong các tổ chức có văn hoá tránh xung đột trực diện — nó cho phép
phản biện mà không ai phải mất mặt trước mặt người khác, vì phản biện là về một đoạn văn bản, ở một
thời điểm khác, và có thể sửa.

**Cơ chế hai: viết cưỡng bức tư duy.** Amazon công khai mô tả việc bỏ PowerPoint để dùng narrative 6
trang với lập luận rằng slide cho phép che các lỗ hổng logic bằng bullet point, còn câu văn hoàn chỉnh
thì không. Cơ chế thật: khi viết một câu có chủ ngữ, động từ và mệnh đề điều kiện, bạn buộc phải xác
định ai làm gì trong điều kiện nào — và chính lúc đó bạn phát hiện mình chưa biết. Tỉ lệ đáng kể các
RFC bị tác giả tự rút lại trong quá trình viết, trước khi có ai comment. Đó không phải thất bại của quy
trình; đó là lợi ích lớn nhất của nó, và nó rẻ.

**Cơ chế ba: chi phí phối hợp đồng bộ tăng theo n², chi phí phối hợp bất đồng bộ tăng gần tuyến tính.**
Một buổi họp 8 người tốn 8 đơn vị thời gian cho 1 đơn vị thảo luận, và số cặp cần đồng bộ hiểu là 28.
Một RFC 8 người đọc tốn 8 đơn vị đọc nhưng mỗi người đọc lúc họ rảnh, và mọi người đồng bộ với **văn
bản** thay vì với nhau — nên số cặp cần đồng bộ là 8, không phải 28. Đây là lý do vì sao mọi tổ chức
engineering vượt qua khoảng 30–40 người đều tự nhiên phát triển một dạng RFC, hoặc chết ngạt trong họp.

**Cơ chế bốn: quyết định được mở lại khi lập luận không được bảo tồn, và điều đó độc lập với chất lượng
quyết định.** Một quyết định tốt không có bản ghi sẽ bị mở lại; một quyết định trung bình có bản ghi
đầy đủ thường không bị. Điều này nghe bất công nhưng hợp lý về mặt thông tin: khi không có bản ghi, chi
phí để một người mới kiểm tra xem quyết định có đúng không cao hơn chi phí để họ tự nghĩ lại từ đầu —
nên họ nghĩ lại từ đầu.

### Mental Model

**RFC như một thị trường phản biện có thời hạn.** Ba tham số điều khiển: ai được đưa hàng ra (ai được
mở RFC), thời gian mở cửa (deadline comment), và ai đóng phiên (người chốt). Nếu thiếu tham số thứ ba,
thị trường không đóng — đó là trạng thái phổ biến nhất của RFC thất bại: 40 comment, không quyết định.

**Reversible/irreversible như bộ lọc đầu vào.** Nối trực tiếp với bảng 4 mức ở chương 04: RFC là công
cụ đắt (10–20 giờ tổng của tổ chức cho một RFC mức 3), nên nó chỉ đáng dùng cho các quyết định mà chi
phí sai vượt chi phí quy trình. Dùng RFC cho quyết định mức 1–2 là cách nhanh nhất để giết quy trình
RFC, vì nó tạo ra một khối lượng văn bản không ai đọc và dạy cả tổ chức rằng RFC là thủ tục.

**Architecture Review Board như một server trong hàng đợi.** Cùng mô hình queueing như code review: nếu
mọi quyết định kiến trúc phải qua một board họp hai tuần một lần, thì thời gian chờ trung bình là một
tuần, và với 7 team, hàng đợi sẽ dài. Board có ích khi nó là **nơi phản biện các quyết định ở ranh giới
giữa các team**; nó trở thành bottleneck khi nó là **cửa phê duyệt cho mọi thứ**. Phân biệt hai chế độ
này là quyết định thiết kế tổ chức quan trọng nhất trong chủ đề này.

### Practical Framework

**Quy trình RFC — sáu tham số phải quyết trước khi mở quy trình.**

| Tham số | Lựa chọn khuyến nghị (tổ chức 30–80 engineer) | Vì sao |
|---|---|---|
| **Ai được mở RFC** | Bất kỳ engineer nào, không cần xin phép; nhưng phải có một sponsor ở mức Tech Lead trở lên đồng ý rằng vấn đề đáng bàn | Mở tự do giữ được ý tưởng từ dưới lên; yêu cầu sponsor lọc các RFC không có vấn đề thật, và bảo vệ tác giả khỏi việc viết 3 ngày rồi bị bỏ qua |
| **Độ dài tối đa** | 6 trang (khoảng 2.500 từ), gồm cả bảng. Cứng | Trên ngưỡng này, tỉ lệ người đọc hết giảm mạnh, và RFC dài thường là dấu hiệu chưa quyết được phạm vi |
| **Thời hạn comment** | Mức 3: 3–5 ngày làm việc. Mức 4: 7–10 ngày. Ghi ngày cụ thể ngay đầu tài liệu | Không có deadline thì RFC treo vô hạn; deadline quá ngắn thì loại người đang bận |
| **Ai chốt** | Một người, tên cụ thể, ghi ở đầu tài liệu. Thường là owner của vùng bị ảnh hưởng nhiều nhất; với quyết định xuyên nhiều team, là Head of Engineering hoặc người được uỷ quyền rõ ràng | Đây là tham số bị bỏ nhiều nhất và là nguyên nhân số một của RFC không kết thúc |
| **Làm gì khi không hội tụ** | Bước 1: người chốt viết một mục "các bất đồng còn lại và lý do tôi chọn hướng này". Bước 2: nếu người phản đối vẫn không đồng ý, họ có quyền yêu cầu một buổi 45 phút với người escalate. Bước 3: người escalate quyết trong 48 giờ. Không có bước 4 | Hạn định số vòng là cách duy nhất tránh analysis paralysis; và "disagree and commit" chỉ hoạt động nếu bất đồng được ghi lại, không bị xoá |
| **Ai bắt buộc phải đọc** | Danh sách tên cụ thể, tối đa 5–7 người, ghi ở đầu. Những người khác được mời nhưng không bắt buộc | "Mọi người nên đọc" nghĩa là không ai chịu trách nhiệm đọc |

**Việc gì cần RFC và việc gì không — bảng quyết định.** Nối trực tiếp với phân loại reversible/
irreversible ở chương 04:

| Loại việc | Mức (chương 04) | Cần gì | Không cần gì |
|---|---|---|---|
| Đổi cấu trúc thư mục trong một service | 1 | PR description | RFC, ADR |
| Chọn thư viện mới chỉ dùng trong một service, đã có trên danh sách được phép | 2 | Một dòng trong PR | RFC |
| Thêm một thư viện ngoài danh sách được phép | 2–3 | ADR-lite + xác nhận của Platform về security/license | RFC đầy đủ |
| Thay đổi contract API mà team khác tiêu thụ | 3 | RFC ngắn (2 trang) + chữ ký team tiêu thụ | Board |
| Thêm một loại hạ tầng mới vào tổ chức (broker, cache layer, database engine) | 3–4 | RFC đầy đủ + ai vận hành + chi phí + ADR | |
| Tách/gộp service, đổi ranh giới team | 4 | RFC + pre-mortem + quyết của Head of Engineering | |
| Schema dữ liệu tài chính, migration không đảo được | 4 | RFC + ADR + kế hoạch rollback đã diễn thử + chữ ký business | |
| Chuẩn xuyên tổ chức (log format, cách phát event, cách xác thực nội bộ) | 3–4 | RFC + Architecture Review + fitness function | |
| Hotfix trong incident | — | Không gì; ghi vào postmortem | RFC, ADR, board |

Nguyên tắc chọn ngưỡng: **cần RFC khi quyết định ảnh hưởng tới người không có mặt trong team bạn, hoặc
khi chi phí đảo vượt 2 người-tuần.** Hai tiêu chí, không cần thêm.

**Architecture Review — khi nào cần và khi nào nó thành bottleneck.**

| Quy mô tổ chức | Cơ chế phù hợp | Vì sao |
|---|---|---|
| < 15 engineer, 1–2 team | Không cần board. Tech Lead + một senior đọc RFC là đủ | Số quyết định xuyên ranh giới rất ít; chi phí lập board vượt lợi ích |
| 15–40 engineer, 3–5 team | Không lập board thường trực. Dùng **reviewer bắt buộc theo miền**: mỗi RFC phải có chữ ký của owner các vùng bị ảnh hưởng | Giữ được phản biện chéo mà không tạo hàng đợi tập trung |
| 40–120 engineer, 6–12 team | Một forum định kỳ 60 phút/tuần, **không phải cửa phê duyệt**: tác giả trình bày 10 phút, nhận phản biện, nhưng quyền quyết vẫn ở người accountable của RFC | Ở quy mô này số quyết định xuyên ranh giới đủ nhiều để cần một nơi thấy được toàn cảnh; nhưng nếu forum có quyền veto, nó sẽ thành bottleneck |
| > 120 engineer | Cần một tầng chuẩn hoá thật (nhóm Platform/Architecture có ngân sách và có on-call), cộng với nguyên tắc "chuẩn được thực thi bằng công cụ, không bằng duyệt" | Chi phí phân kỳ ở quy mô này lớn hơn chi phí chuẩn hoá; nhưng duyệt thủ công không scale được |

**Bốn dấu hiệu board đã thành bottleneck** — kiểm tra mỗi quý:

| Dấu hiệu | Ngưỡng minh hoạ | Phản ứng |
|---|---|---|
| Thời gian chờ từ khi RFC sẵn sàng tới khi được xét | > 10 ngày làm việc | Tăng tần suất, hoặc hạ ngưỡng việc phải qua board |
| Tỉ lệ RFC bị yêu cầu quay lại vòng sau | > 30% | Vấn đề nằm ở tiêu chí không rõ, không ở chất lượng RFC. Viết tiêu chí ra |
| Số quyết định đi đường tắt (làm rồi mới báo) | Tăng theo quý | Tổ chức đang bỏ qua quy trình vì quy trình đắt hơn lợi ích |
| Tỉ lệ thành viên board có tên trong rotation on-call | < 50% | Board đang quyết mà không chịu hậu quả — xem anti-pattern |

### Trade-off

**Trade-off chính — Chuẩn hoá quyết định kiến trúc toàn công ty vs tự chủ theo team.**

| | Chuẩn hoá tập trung | Tự chủ theo team |
|---|---|---|
| Được | Cognitive load thấp khi đi giữa các service; engineer chuyển team không phải học lại; on-call chung khả thi; đầu tư platform có đòn bẩy trên mọi team; tuyển nội bộ dễ vì kỹ năng chuyển được | Quyết định khớp bài toán cục bộ; nhanh, không chờ; team có ownership thật nên chất lượng vận hành cao hơn; công nghệ mới vào được tổ chức |
| Mất | Chậm; một số lựa chọn không tối ưu cho một số bài toán; rủi ro tầng trung tâm xa hiện trường; dễ trở thành cửa phê duyệt | Phân kỳ: n cách làm cùng một việc; on-call chéo bất khả; chi phí vận hành tăng theo số biến thể; kiến thức không chuyển được giữa team |
| Nghiêng về khi | > 5 team; có ràng buộc compliance/audit chung; có nhu cầu on-call chéo; tổ chức có tỉ lệ luân chuyển nội bộ cao; hệ thống có nhiều điểm tích hợp giữa các team | ≤ 3 team; các team làm các sản phẩm độc lập, ít tích hợp; team có senior đủ mạnh để tự chịu hậu quả vận hành; đang trong giai đoạn khám phá công nghệ |
| Điểm cân bằng thực tế | Chuẩn hoá **giao diện**, tự chủ **bên trong**: quy định cách các team gọi nhau, cách phát event, cách log/trace, cách xác thực, cách deploy; để nguyên quyền chọn framework, cấu trúc code, thư viện nội bộ bên trong ranh giới team | |

Cách phát biểu điểm cân bằng này dưới dạng một nguyên tắc dùng được: **chuẩn hoá những gì đi qua ranh
giới giữa các team, và những gì on-call phải chạm vào lúc 2 giờ sáng. Mọi thứ khác để team quyết.** Hai
tiêu chí này có cơ sở: cái đi qua ranh giới tạo chi phí phối hợp cho người khác; cái on-call chạm vào
tạo chi phí nhận thức trong tình huống áp lực cao nhất.

**Trade-off thứ hai — RFC bắt buộc vs RFC tuỳ chọn.** Bắt buộc đảm bảo mọi quyết định lớn có bản ghi,
nhưng sinh ra RFC hình thức viết sau khi đã quyết. Tuỳ chọn giữ được chất lượng của những RFC được viết
nhưng bỏ sót đúng các quyết định do người ít kinh nghiệm nhất ra. Điều kiện nghiêng: bắt buộc theo
**tiêu chí khách quan** (chạm dữ liệu tiền, đổi contract liên team, thêm hạ tầng mới) và tuỳ chọn cho
phần còn lại. Tiêu chí khách quan là chìa khoá — "khi bạn thấy cần" là tiêu chí không dùng được, vì
người cần RFC nhất là người không thấy cần.

**Trade-off thứ ba — Đồng thuận rộng vs quyết nhanh.** Mỗi người đọc thêm làm tăng chất lượng phản biện
với lợi ích giảm dần, và tăng thời gian chờ với chi phí tăng đều. Với RFC mức 3, số reviewer bắt buộc
tối ưu theo kinh nghiệm là 3–5; trên 7 người, chất lượng comment giảm vì hiệu ứng khuếch tán trách
nhiệm — mỗi người cho rằng người khác sẽ đọc kỹ.

### Real-world Scenarios

**Tình huống A — RFC thành nghi lễ xin phép.** Một công ty fintech 70 engineer đưa ra quy định: mọi
thay đổi kiến trúc phải có RFC được Architecture Review Board (5 người, họp thứ Ba hàng tuần) phê
duyệt. Sau 6 tháng, các hiện tượng đo được: thời gian chờ trung vị từ RFC sẵn sàng tới được xét là 12
ngày; 44% RFC bị yêu cầu quay lại vòng sau; và điều đáng lo nhất — số RFC mở mỗi tháng giảm từ 9 xuống
3, trong khi số service mới vẫn tăng. Nghĩa là các quyết định vẫn được ra, chỉ không đi qua quy trình
nữa.

Điều tra bằng cách hỏi riêng 6 Tech Lead cho ra hai lý do lặp lại. Một: "viết RFC mất 2 ngày, chờ 2
tuần, rồi bị hỏi những câu mà nếu biết trước tôi đã trả lời — tôi không biết họ đánh giá theo tiêu chí
gì." Hai: "họ hỏi những câu về scale mà không ai trong board đang vận hành hệ thống nào cả."

Cơ chế hỏng: board đã trở thành cửa phê duyệt (mode "gate") thay vì nơi phản biện (mode "forum"), và
tiêu chí phê duyệt không được viết ra. Khi tiêu chí ẩn, chi phí kỳ vọng của việc đi qua quy trình trở
nên không dự đoán được, và người ta chọn đường tắt. Đây là hành vi hợp lý, không phải thiếu kỷ luật.

Cách sửa, làm trong một quý: (1) Viết ra tiêu chí — một checklist 9 điểm công khai mà board sẽ hỏi, để
tác giả tự kiểm trước. (2) Bỏ quyền veto của board; board cho phản biện, người accountable của RFC
quyết, và nếu board phản đối mạnh thì escalate lên CTO trong 48 giờ. (3) Yêu cầu ≥ 3/5 thành viên board
có tên trong rotation on-call. (4) Hạ ngưỡng: chỉ quyết định thuộc mức 4 và các chuẩn xuyên tổ chức mới
cần qua board; mức 3 chỉ cần reviewer bắt buộc theo miền. Sau một quý (số minh hoạ): thời gian chờ
trung vị 3 ngày, số RFC/tháng lên 11, và số quyết định đi đường tắt giảm rõ vì đường chính đã rẻ hơn
đường tắt.

**Tình huống B — RFC 40 trang không ai đọc.** Tuấn viết một RFC 41 trang về nền tảng dữ liệu, có 14
diagram, 6 bảng so sánh công cụ. Hai tuần, 3 comment, tất cả về lỗi chính tả. Tuấn kết luận "team không
quan tâm". Đọc lại tài liệu cho thấy vấn đề khác: nó trộn bốn quyết định độc lập (chọn công cụ ingest,
chọn định dạng lưu, chọn engine truy vấn, chọn mô hình quản trị dữ liệu), và mỗi quyết định có tập
stakeholder khác nhau. Không ai là người phù hợp để đọc cả bốn, nên không ai đọc gì.

Xử lý: Minh giúp Tuấn chia thành một RFC "định hướng" 3 trang (vấn đề, nguyên tắc, thứ tự quyết định,
và những gì cố tình chưa quyết) cộng ba RFC 2 trang cho ba quyết định đầu, mỗi cái có danh sách 3–4
người bắt buộc đọc, tên cụ thể. Kết quả: RFC định hướng nhận 21 comment trong 4 ngày. Bài học có thể
tổng quát: **số comment trên một RFC gần như tỉ lệ nghịch với độ dài, và tỉ lệ thuận với việc danh sách
người đọc có tên cụ thể hay không.** Nếu RFC của bạn không nhận comment, hai giả thuyết cần kiểm tra
trước mọi giả thuyết về văn hoá: nó quá dài, hoặc nó không nói rõ ai phải đọc.

**Script hội thoại — đóng một RFC không hội tụ.** Bối cảnh: RFC về việc chuẩn hoá cách phát event. Sau
9 ngày, 32 comment, hai luồng đối lập: Tuấn muốn schema registry với Avro (chặt, có kiểm tra tương
thích tự động), Hà muốn JSON có versioning nhẹ (nhanh, đội không phải học thêm công cụ). Minh là người
chốt.

Phiên bản nói sai:

> Minh: "Thôi mình dừng tranh luận ở đây nhé, mình thấy hai bên đều có lý. Anh nghiêng về JSON vì nó
> đơn giản hơn. Mọi người ok chứ? Không ai phản đối thì mình chốt vậy đi."

Ba lỗi. "Hai bên đều có lý" xoá lập luận thay vì phán xử nó — người thua không biết mình thua ở đâu nên
sẽ mở lại. "Anh nghiêng về" là phát biểu sở thích, không phải áp tiêu chí, nên quyết định này không có
giá trị tiền lệ cho lần sau. "Không ai phản đối thì chốt" trong một tổ chức tránh xung đột trực diện là
cách thu về sự im lặng và gọi nó là đồng thuận — Tuấn sẽ không phản đối trong họp và sẽ dùng Avro trong
service của mình ba tháng sau.

Phiên bản nói đúng:

> Minh: "Mình đóng RFC này hôm nay. Trước khi tôi nói quyết định, tôi nhắc lại tiêu chí mà chúng ta đã
> chốt ở mục 2 của RFC, theo thứ tự: (1) không được mất event trong luồng thanh toán, (2) một team mới
> phải tích hợp được trong dưới 2 ngày, (3) chi phí vận hành thêm không quá một nửa người.
>
> Theo tiêu chí (1), cả hai phương án đều đạt nếu có contract test — Tuấn đã chỉ ra đúng rằng JSON
> không có kiểm tra tương thích tự động, nên tôi đưa contract test vào phần bắt buộc của quyết định,
> đó là điểm Tuấn thắng và nó thay đổi thiết kế. Theo tiêu chí (2), Avro cần đội học schema registry,
> ước lượng 3–5 ngày cho một team mới, không đạt. Theo tiêu chí (3), schema registry cần một người vận
> hành mà hiện tại chúng ta không có — Duy đã xác nhận trong comment ngày 14.
>
> Nên quyết định: JSON với versioning theo quy ước ở RFC, cộng contract test bắt buộc trong CI cho mọi
> producer. Tôi accountable cho quyết định này.
>
> Điều tôi biết là mình đang chấp nhận rủi ro: không có kiểm tra tương thích ở tầng schema, nên sai sót
> sẽ bị bắt muộn hơn. Ghi vào RFC như một rủi ro đã biết. Điều kiện xét lại: nếu có ≥ 2 incident do
> event không tương thích trong 2 quý, hoặc khi số producer vượt 15, chúng ta mở lại và lúc đó Avro là
> phương án mặc định.
>
> Tuấn, tôi biết anh không đồng ý với hướng này. Tôi đã ghi lập luận của anh vào mục 'bất đồng còn
> lại', nguyên văn, có tên anh. Nếu anh muốn escalate lên Khoa, anh có quyền và tôi sẽ hỗ trợ đặt
> lịch trong tuần này. Nếu không escalate, tôi cần anh commit vào hướng này — nghĩa là service của anh
> cũng dùng JSON, không có ngoại lệ. Anh nói thẳng giúp tôi được không?"

Năm cơ chế trong đoạn này: (1) áp tiêu chí đã chốt trước, nên quyết định không phải sở thích và có giá
trị tiền lệ; (2) nêu rõ điểm mà người thua đã thắng, để lập luận của họ không bị xoá; (3) tuyên bố một
tên accountable; (4) ghi rủi ro đã biết và điều kiện xét lại, nên quyết định này có đường mở lại chính
thức — điều làm giảm mạnh nhu cầu mở lại không chính thức; (5) tách "không đồng ý" khỏi "không commit"
và yêu cầu một câu trả lời thẳng, thay vì thu về sự im lặng.

### Best Practices

- **Viết mục tiêu chí đánh giá trước khi viết phần phương án, và chốt tiêu chí với người sẽ quyết.**
  Cơ chế: nếu tiêu chí được chốt sau khi đã biết phương án nào thắng, nó sẽ được viết để hợp thức hoá
  lựa chọn có sẵn. Đây là cùng nguyên tắc với decision matrix ở chương 04.
- **Ghi ngày hết hạn comment và tên người chốt ở dòng đầu tiên của RFC.** Cơ chế: hai thông tin này
  quyết định liệu người đọc có bỏ thời gian không — họ cần biết comment của mình còn kịp và có ai xử
  lý.
- **Danh sách người bắt buộc đọc là tên người, tối đa 5–7.** Cơ chế: trách nhiệm phân tán bằng không
  trách nhiệm; và một danh sách 20 người cho tín hiệu rằng không ai thật sự cần đọc.
- **Giữ mục "bất đồng còn lại" trong RFC đã chốt, có tên người, nguyên văn.** Cơ chế: nó là điều kiện
  để "disagree and commit" khả thi về mặt tâm lý — người phản đối chấp nhận thua dễ hơn nhiều khi lập
  luận của họ được lưu lại thay vì bị xoá. Nó cũng là dữ liệu quý khi mở lại quyết định sau 18 tháng.
- **Chia RFC theo số quyết định, không theo chủ đề.** Một RFC = một quyết định + một tập stakeholder.
  Cơ chế: nó làm cho danh sách người đọc xác định được, và làm cho RFC ngắn tự nhiên.
- **Với RFC quan trọng, mở một buổi 30 phút ở giữa thời hạn comment, không ở đầu và không ở cuối.** Cơ
  chế: ở đầu thì chưa ai đọc nên họp thành buổi thuyết trình; ở cuối thì đã quá muộn để đổi thiết kế.
  Giữa thời hạn là lúc phản biện đã hình thành nhưng còn sửa được.
- **Đo và công bố ba chỉ số của quy trình RFC mỗi quý:** thời gian trung vị từ mở tới chốt, số RFC/
  tháng, và tỉ lệ quyết định lớn có RFC. Cơ chế: quy trình không được đo sẽ trượt dần thành nghi lễ mà
  không ai thấy.
- **Cho phép RFC bị rút, và ghi nhận việc rút như một kết quả tốt.** Cơ chế: nếu rút RFC bị coi là thất
  bại, tác giả sẽ bảo vệ đề xuất của mình thay vì tìm câu trả lời đúng — và giá trị "viết cưỡng bức tư
  duy" bị mất.

### Anti-patterns

| Anti-pattern | Cơ chế phá hệ thống | Dấu hiệu sớm |
|---|---|---|
| **RFC thành nghi lễ xin phép** | Chi phí đi qua quy trình không dự đoán được → engineer chọn đường tắt → quyết định vẫn được ra nhưng bây giờ không có bản ghi nào, tệ hơn cả trước khi có quy trình | Số RFC/tháng giảm trong khi số thay đổi kiến trúc không giảm; tác giả nói "không biết họ muốn gì" |
| **Review board không có ai chịu hậu quả vận hành** | Externality: board tối ưu tính chuẩn hoá và tính an toàn trên giấy, chi phí MTTR và on-call dồn sang người khác; đồng thời board mất uy tín kỹ thuật nên quyết định của nó bị lách | Đếm tỉ lệ thành viên board có tên trong rotation on-call; nghe câu "họ không hiểu hệ thống thật chạy thế nào" |
| **RFC 40 trang không ai đọc** | Vượt ngưỡng đọc và trộn nhiều tập stakeholder → không ai là người phù hợp để đọc hết → không phản biện → RFC được "thông qua" mà thật ra chưa được xét | Tỉ lệ comment thực chất/độ dài; comment chỉ về chính tả và format |
| **RFC không có người chốt** | Thị trường phản biện không đóng; tài liệu treo ở Proposed; công việc phía sau bị chặn hoặc người ta tự làm không theo RFC | Đếm số RFC ở trạng thái Proposed quá 3 tuần |
| **RFC viết sau khi code đã xong** | Nghi lễ hợp thức hoá; giá trị "viết cưỡng bức tư duy" bằng không vì tư duy đã kết thúc; và phản biện trở thành công kích vì người ta biết phản biện vô nghĩa | Ngày mở RFC muộn hơn commit đầu tiên của nhánh liên quan |
| **Board họp để phê duyệt mọi thứ** | Hàng đợi tập trung, thời gian chờ tăng phi tuyến; và board không có băng thông để đọc sâu nên chất lượng phản biện thấp cho cả những việc thật sự cần | Agenda board có > 4 hạng mục mỗi buổi; thời gian mỗi hạng mục < 15 phút |
| **Dùng RFC để phân tán trách nhiệm** | "Đã có RFC, mọi người đã comment" trở thành lá chắn khi kết quả xấu; không ai học được gì vì không ai chịu | Trong RFC không có tên một người accountable, chỉ có tên team; sau sự cố, câu trả lời là "quy trình đã được tuân thủ" |

### Khi nào KHÔNG nên áp dụng

**Khi tổ chức dưới 15 engineer và mọi người ngồi cùng một chỗ.** Chi phí phối hợp đồng bộ ở quy mô này
còn thấp hơn chi phí viết. Một cuộc trò chuyện 30 phút với ba người, kèm 10 dòng ADR-lite ghi lại kết
quả, làm được đúng việc mà một quy trình RFC đầy đủ làm — với 5% chi phí. Ngưỡng đáng bắt đầu nghĩ tới
RFC: khi bạn thấy mình phải giải thích cùng một quyết định cho lần thứ ba cho ba nhóm người khác nhau,
hoặc khi có team thứ ba.

**Khi quyết định phải ra trong vòng 24 giờ.** Có những tình huống có cửa sổ hẹp thật: một đối tác yêu
cầu tích hợp trước ngày X, một lỗ hổng bảo mật đang bị khai thác, một chi phí cloud tăng vọt cần chặn.
Ở đây quy trình RFC 5 ngày là chi phí thuần. Cách đúng: quyết ngay với người accountable rõ ràng, đánh
dấu là **quyết định tạm thời có thời hạn**, và viết RFC hồi tố trong 2 tuần với mục đích khác — không
phải để xin phép mà để kiểm tra xem quyết định gấp có cần điều chỉnh, và để tổ chức học. Điều kiện để
lối này không bị lạm dụng: tiêu chí "gấp" phải được định nghĩa trước, và số lần dùng phải được đếm.

**Khi tác giả không có quyền thực hiện quyết định đó.** RFC đề xuất một hướng mà người viết không có
capacity, không có ngân sách và không có sponsor sẽ chết ở khâu thực thi, và trải nghiệm đó dạy cả tổ
chức rằng viết RFC là vô ích. Thứ tự đúng: đảm bảo có sponsor và có capacity trên nguyên tắc trước, rồi
mới viết RFC để quyết **cách làm**. Đây là lý do vì sao yêu cầu "phải có sponsor mức Tech Lead" trong
bảng tham số không phải quan liêu — nó bảo vệ tác giả.

**Khi vấn đề thật là bất đồng về mục tiêu, không phải về giải pháp kỹ thuật.** Nếu hai team đang tranh
luận về kiến trúc nhưng gốc là một team bị đo bằng tốc độ giao hàng và team kia bị đo bằng độ ổn định,
thì không RFC nào giải quyết được — mọi vòng phản biện sẽ đi vòng quanh vì hai bên đang tối ưu hai hàm
khác nhau. Việc cần làm trước là đưa xung đột về mục tiêu lên tầng có quyền quyết mục tiêu (thường là
Head of Engineering hoặc cấp trên nữa), chốt thứ tự ưu tiên, rồi mới mở RFC. Dấu hiệu nhận biết: RFC
đã qua ba vòng comment mà không phương án nào tiến lên, và các comment lặp lại cùng một cặp lập luận.

**Khi bạn là ODC và khách hàng đã chốt kiến trúc bằng hợp đồng.** Mở một RFC nội bộ để "quyết" một thứ
đã bị quyết là tạo ra kỳ vọng sai và làm engineer thất vọng khi kết quả không thay đổi được gì. Dạng
đúng ở đây là một **technical position paper**: nêu rủi ro bằng số, nêu phương án thay thế, gửi cho
khách qua đúng kênh, và lưu lại như bản ghi. Nó có chức năng khác RFC — không phải để quyết, mà để dịch
chuyển một quyết định của bên khác và để bảo vệ đội khi hậu quả xảy ra.

---

## 5. Technical Debt Management

### Problem Statement

Một công ty logistics, hệ thống chạy 6 năm, 28 engineer. Bốn hiện tượng được ghi lại trong một quý (số
minh hoạ, cách đo thì lấy được từ git và issue tracker của bạn):

| Hiện tượng | Số đo | Đọc ra điều gì |
|---|---|---|
| Lead time trung vị cho một thay đổi nhỏ ở module `pricing` | 9 ngày, so với 1,5 ngày ở các module khác | Mỗi thay đổi ở đây tốn thêm ~7,5 ngày — đó là **tiền lãi** đang trả |
| Change failure rate của module `pricing` | 31%, so với 8% toàn hệ thống | Gần một phần ba thay đổi phải sửa lại; chi phí lãi còn cao hơn con số trên |
| Thời gian để một engineer mới đóng được ticket đầu tiên vào module này | 5 tuần, so với 1 tuần ở module khác | Debt biểu hiện thành chi phí nhân sự, và nó là lý do chỉ 2/28 người dám sửa module này |
| Số lần câu "phải hỏi anh Quân" xuất hiện trong Slack liên quan module này | 47 lần/quý | Bus factor = 1, và Quân là bottleneck của một phần doanh thu |

Khi Minh trình bày với ban lãnh đạo, câu hỏi đầu tiên là "cái này ảnh hưởng doanh thu thế nào". Đây là
câu hỏi hợp lý và là chỗ phần lớn đề xuất trả debt thất bại. Cách trả lời không dùng được: "code rất
xấu, chúng ta cần refactor". Cách trả lời dùng được: "module này chặn 3 hạng mục trong roadmap quý sau;
mỗi hạng mục hiện tốn thêm ~7 ngày lead time và có 31% xác suất phải làm lại, tương đương khoảng 34
người-ngày lãi trả trong một quý; nếu bỏ 20 người-ngày để tách phần tính giá ra khỏi phần điều phối,
lãi giảm còn khoảng 8 người-ngày/quý và chúng ta hoàn vốn trong hơn một quý."

Hậu quả của việc không quản lý debt không phải "code xấu". Nó là: mỗi ước lượng đều trượt và không ai
biết vì sao → PO mất tin vào Estimation của team → PO bắt đầu ép deadline → team cắt góc để kịp → debt
tăng → lãi tăng → ước lượng trượt nặng hơn. Đây là một vòng phản hồi dương, và đặc điểm của vòng phản
hồi dương là nó không tự dừng.

### First Principles

**Cơ chế một: technical debt là khoảng cách giữa thiết kế hiện tại và thiết kế cần có cho yêu cầu hiện
tại.** Ba hệ quả quan trọng từ định nghĩa này. Một, debt là **tương đối với yêu cầu**, không phải thuộc
tính nội tại của code: một script 200 dòng không có test là thiết kế hoàn hảo cho một công cụ nội bộ
chạy một lần, và là debt nghiêm trọng cho một luồng tính tiền. Hai, **debt có thể tăng mà không ai sửa
gì** — nếu yêu cầu đổi, khoảng cách rộng ra. Ba, "code cũ" không phải debt; code cũ mà không cần đổi
và không ai phải đọc thì lãi bằng không, và trả nó là tiêu tiền vô ích.

**Cơ chế hai: ma trận Fowler — bốn loại debt cần bốn phản ứng khác nhau.** Martin Fowler công bố ma
trận hai chiều này (deliberate/inadvertent × prudent/reckless), và giá trị của nó nằm ở chỗ nó chỉ ra
rằng chỉ có một ô là vấn đề về năng lực, còn ba ô kia là vấn đề về quản trị.

| | **Prudent (thận trọng)** | **Reckless (bất cẩn)** |
|---|---|---|
| **Deliberate (có ý thức)** | "Chúng ta biết cách làm đúng, nhưng phải ship trước 30/06 nên làm tắt, đã ghi lại và sẽ trả trong Q3." → **Đây là quản trị tốt.** Cần: bản ghi, ước lượng lãi, thời hạn trả, tên người. | "Không có thời gian cho thiết kế." → Vấn đề về ưu tiên và về áp lực từ trên. Cần: đổi cách đặt deadline, không phải đào tạo kỹ thuật. |
| **Inadvertent (vô ý)** | "Bây giờ chúng ta mới hiểu ranh giới đúng nên nằm ở đâu." → **Không tránh được và không nên tránh.** Đây là chi phí của việc học. Cần: cơ chế phát hiện và chỉnh sửa dần, không phải cố thiết kế đúng từ đầu. | "Layering là gì?" → Vấn đề về năng lực. Cần: mentoring, review, pairing, tuyển. |

Sai lầm quản trị phổ biến nhất là xử lý cả bốn ô bằng một biện pháp duy nhất — thường là "tăng test
coverage" hoặc "làm sprint dọn dẹp". Ô deliberate-reckless không giảm bằng test; nó chỉ giảm khi cách
đặt deadline đổi.

**Cơ chế ba: lãi kép — debt làm chậm mọi thay đổi sau đó, nên chi phí không tuyến tính.** Hình dung một
module có hệ số ma sát: mỗi thay đổi tốn thêm 30% thời gian. Với 10 thay đổi, bạn mất 3 đơn vị. Nhưng
mỗi thay đổi làm trong điều kiện thiết kế xấu có xác suất cao tự thêm debt (vì cách rẻ nhất để thêm
tính năng vào một thiết kế xấu thường là làm nó xấu hơn), nên hệ số ma sát tăng dần. Đây là cơ chế lãi
kép, và nó có ba hệ quả thực hành:

- **Chi phí trả debt tăng theo thời gian, còn lợi ích của việc trả giảm theo thời gian** (vì càng gần
  cuối đời của một module, càng ít thay đổi còn lại để được lợi). Suy ra tồn tại một cửa sổ tối ưu để
  trả, và trả quá muộn là tiêu tiền không thu hồi được.
- **Lãi tỉ lệ với tần suất thay đổi của module.** Một module tệ nhưng không ai sửa có lãi gần bằng
  không. Đây là tiêu chí xếp ưu tiên quan trọng nhất và hay bị bỏ: xếp ưu tiên theo mức độ tệ là sai;
  xếp theo **tệ × tần suất thay đổi × giá trị kinh doanh của các thay đổi đó** là đúng.
- **Lãi được trả bởi người khác với người vay.** Người vay ship kịp deadline và được ghi nhận; người
  trả lãi là người sửa module đó 8 tháng sau, thường là người khác. Đây là một externality trong nội
  bộ tổ chức, và nó là lý do debt tích tụ dù mọi người đều thông minh và có thiện chí.

**Cơ chế bốn: broken windows và entropy.** Hai cơ chế bổ sung nhau. Broken windows (lý thuyết được
Kent Beck và cộng đồng XP dùng rộng rãi cho phần mềm): một chỗ xấu tồn tại lâu làm hạ ngưỡng chấp nhận
của những người tiếp theo — nếu ở đây đã có ba cách xử lý lỗi khác nhau, thêm cách thứ tư không tạo
cảm giác sai. Entropy: một hệ thống có nhiều người chạm vào, nếu không có lực hướng, sẽ tăng độ mất trật
tự đơn thuần do số lượng thay đổi độc lập. Cả hai đều nói cùng một điều về mặt quản trị: **giữ trạng
thái hiện tại đã cần công; muốn cải thiện thì cần công nhiều hơn thế.**

### Mental Model

**Debt như lãi vay — bốn đại lượng phải phân biệt.**

| Đại lượng | Nghĩa trong phần mềm | Cách đo |
|---|---|---|
| **Principal (gốc)** | Chi phí để sửa về thiết kế đúng | Ước lượng người-ngày cho việc refactor |
| **Interest (lãi)** | Thời gian mất thêm mỗi lần thay đổi vùng đó, cộng chi phí do failure rate cao | (lead time module − lead time baseline) × số thay đổi/quý + chi phí làm lại |
| **Interest rate (lãi suất)** | Tần suất thay đổi × mức ma sát | Số PR chạm module đó mỗi quý |
| **Term (kỳ hạn)** | Còn bao lâu nữa module này còn được dùng | Roadmap: module này còn trong hệ thống 6 tháng hay 5 năm |

Quyết định trả hay không trả là một phép tính đơn giản khi có bốn số này: **trả nếu lãi tích lũy trong
kỳ hạn còn lại vượt gốc, và nếu bạn có capacity trong cửa sổ đó.** Sức mạnh của mô hình không phải ở độ
chính xác của phép tính — mà ở chỗ nó buộc cuộc thảo luận chuyển từ "code xấu/không xấu" (không giải
được) sang "lãi bao nhiêu, gốc bao nhiêu, kỳ hạn bao lâu" (giải được, và nói được với người không kỹ
thuật).

**Broken windows như một ngưỡng.** Có một hiện tượng ngưỡng: dưới một mức lộn xộn nhất định, người mới
vào tự tuân theo chuẩn hiện có vì chuẩn dễ nhận ra; trên mức đó, người mới không nhận ra chuẩn nào là
chuẩn nên họ chọn bất kỳ. Hệ quả thực hành quan trọng: **trong một codebase đã lộn xộn, viết thêm tài
liệu về chuẩn gần như vô tác dụng** — vì tín hiệu từ code mạnh hơn tín hiệu từ tài liệu. Cách vào duy
nhất hiệu quả là làm sạch một vùng nhỏ và hoàn toàn, rồi dùng nó làm tham chiếu ("làm như module
`notification`"), thay vì làm sạch 20% ở khắp nơi.

**Ba loại thứ hay bị gọi sai là debt.** Phân biệt quan trọng vì mỗi loại có cách xử lý khác:

| Bị gọi là debt | Thật ra là gì | Xử lý đúng |
|---|---|---|
| "Code này không theo style tôi thích" | Khác biệt sở thích | Formatter, hoặc bỏ qua |
| "Chúng ta chưa dùng công nghệ mới nhất" | Không phải debt nếu công nghệ hiện tại còn đáp ứng yêu cầu | Chỉ trả nếu có lãi thật: hết support, không tuyển được người, có lỗ hổng bảo mật |
| "Thiếu tính năng X" | Đây là backlog sản phẩm | Đưa vào backlog, đừng gọi là debt để tránh cạnh tranh sai chỗ |

### Practical Framework

**Bước 1 — Đo debt bằng tín hiệu quan sát được, không bằng cảm giác.** Bốn tín hiệu, tất cả lấy được
từ hệ thống bạn đang có, và chúng nên được tính theo **module** chứ không theo toàn hệ thống:

| Tín hiệu | Nguồn dữ liệu | Ngưỡng đáng chú ý (minh hoạ) |
|---|---|---|
| Lead time trung vị của thay đổi trong module | Git + issue tracker, tag PR theo path | > 2,5 lần trung vị toàn hệ thống |
| Change failure rate theo module | Hệ thống incident + số hotfix/rollback theo module | > 2 lần trung vị toàn hệ thống |
| Số incident theo module trong 12 tháng | Postmortem có tag component | Module chiếm > 25% incident trong khi < 10% code |
| Thời gian onboarding vào module | Thời gian từ khi được gán tới PR đầu tiên được merge, của 3 người gần nhất | > 3 lần trung vị |

Ba lý do dùng bốn tín hiệu này thay vì các chỉ số tĩnh (cyclomatic complexity, coverage, số code smell):
(a) chúng đo **hậu quả** thay vì đo **hình dạng**, và hậu quả là cái business quan tâm; (b) chúng không
thao tác được dễ dàng bằng cách viết test rỗng; (c) chúng dịch trực tiếp được sang tiền, vì đơn vị của
chúng là thời gian người.

**Bước 2 — Lập debt register có chủ và có ước lượng lãi.** Đây là hiện vật trung tâm. Nó không phải
backlog thứ hai; nó là một bảng có tối đa 15–20 dòng, xét lại mỗi quý, và mọi dòng đều có tên người.

**TEMPLATE — Debt Register (dạng bảng, đặt trong repo hoặc wiki có version)**

| ID | Mô tả debt | Loại (Fowler) | Module | Lãi/quý (người-ngày) | Cách tính lãi | Gốc (người-ngày) | Kỳ hạn còn lại | Ưu tiên | Chủ | Chiến lược trả | Trạng thái |
|---|---|---|---|---|---|---|---|---|---|---|---|
| TD-07 | Logic tính giá trộn trong service điều phối; không tách được để test riêng | Deliberate-prudent (vay để kịp campaign 11/2025, có ghi lại) | pricing | 34 | (9 − 1,5 ngày) × 12 PR/quý × 31% rework | 20 | ≥ 3 năm (lõi nghiệp vụ) | 1 | Minh | Tách module + test đặc tả, chia 3 PR | Đang làm, 40% |
| TD-11 | Không có idempotency ở webhook đối tác; xử lý trùng bằng check thủ công | Inadvertent-prudent (khi làm chưa biết đối tác gửi trùng) | integration | 12 (chủ yếu là thời gian đối soát thủ công) | 4 giờ/tuần × 13 tuần / 8 | 8 | ≥ 2 năm | 2 | Hà | Opportunistic: làm cùng lần tích hợp đối tác tiếp theo | Chờ trigger |
| TD-14 | 3 cách gửi thông báo song song | Inadvertent-reckless | notification | 9 | 3 runbook, MTTR +45 phút × 6 incident/quý | 15 | ≥ 3 năm | 2 | Duy | Strangler: mọi caller mới dùng đường chuẩn; chuyển dần caller cũ, 2 caller/quý | Đang làm, 30% |
| TD-19 | Thư viện auth hết support từ 03/2026, không còn bản vá bảo mật | Deliberate-prudent | toàn hệ thống | 0 hiện tại, nhưng rủi ro compliance | Không phải lãi thời gian; là rủi ro | 12 | — | 1 (do rủi ro, không do lãi) | Duy | Dedicated: block 2 tuần trong Q3 | Chưa bắt đầu |
| TD-22 | Module báo cáo cũ khó đọc, không có test | Inadvertent-reckless | reporting | ~1 | 2 PR/năm chạm vào | 25 | 9 tháng (sẽ thay bằng BI tool) | 5 — **không trả** | — | Cố tình không trả, ghi lý do | Đóng, "won't fix" |

Dòng TD-22 là dòng quan trọng nhất trong bảng mẫu này: một debt được **quyết định không trả**, có ghi
lý do. Một debt register mà mọi dòng đều "sẽ trả" là một danh sách ước muốn. Đúng như strategy ở chủ đề
1, giá trị của register nằm ở các dòng bị loại.

**TEMPLATE — Một entry debt ở dạng văn bản (dùng khi cần trình bày hoặc xin ngân sách)**

```markdown
## TD-07 — Logic tính giá trộn trong service điều phối

Loại: Deliberate-prudent (vay có ý thức, tháng 11/2025, để kịp campaign Black Friday;
      ADR-009 có ghi và có ngày trả dự kiến là Q1/2026 — đã trượt 1 quý)
Module: pricing | Chủ: Minh | Trạng thái: đang làm, 40%

### Lãi đang trả (số minh hoạ, tính theo dữ liệu quý gần nhất)
- Lead time thay đổi trong module: 9 ngày, baseline toàn hệ thống 1,5 ngày → +7,5 ngày/thay đổi.
- Số thay đổi chạm module: 12 PR/quý.
- Change failure rate: 31% (baseline 8%) → khoảng 3,7 lần làm lại mỗi quý, mỗi lần ~2 ngày.
- Lãi ước tính: 12 × 7,5 × 0,3 (phần quy được cho thiết kế) + 3,7 × 2 ≈ 34 người-ngày/quý.
- Chi phí phi thời gian: bus factor = 1 (chỉ Quân sửa được) → rủi ro nhân sự.

### Gốc
- 20 người-ngày: tách module tính giá, viết test đặc tả từ hành vi hiện tại, chuyển caller.
- Chia được thành 3 PR độc lập, mỗi PR ship riêng, không cần big-bang.

### Vì sao trả bây giờ
- Roadmap Q3 có 3 hạng mục chạm module này (giá theo vùng, giá theo hợp đồng B2B, khuyến mãi
  theo bậc). Nếu không trả, 3 hạng mục này chịu toàn bộ lãi và có 31% xác suất làm lại.
- Ước tính hoàn vốn: 20 người-ngày bỏ ra, tiết kiệm ~26 người-ngày/quý → hoàn vốn trong
  khoảng 1 quý, sau đó là lợi.

### Nếu không trả
- 3 hạng mục Q3 sẽ trượt ước lượng khoảng 30–40%; đây là con số tôi sẽ dùng khi commit roadmap.
- Bus factor vẫn = 1. Nếu Quân nghỉ phép 2 tuần trong Q3, 3 hạng mục đó dừng.

### Điều kiện dừng / xét lại
- Nếu sau PR đầu (7 người-ngày) lead time module không giảm xuống dưới 5 ngày, dừng và xét lại
  cách tiếp cận — giả định về nguyên nhân đã sai.
```

**Bước 3 — Chọn một trong ba chiến lược trả, theo tiêu chí.**

| Chiến lược | Cách làm | Phù hợp khi | Rủi ro | Cách kiểm soát rủi ro |
|---|---|---|---|---|
| **Opportunistic** ("dọn khi đi ngang qua") | Mỗi khi sửa một vùng, cải thiện vùng đó trong cùng PR, giới hạn ~15% kích thước PR | Debt rải rác, lãi thấp mỗi chỗ, không có vùng nào chặn roadmap | PR phình to; review khó vì trộn refactor với logic; không bao giờ xử lý được debt lớn | Quy tắc: refactor và thay đổi hành vi ở hai commit riêng, hoặc hai PR riêng |
| **Dedicated capacity** (bucket %) | Dành cố định 15–20% capacity mỗi sprint cho debt trong register, có chủ và có định nghĩa xong | Debt tập trung ở vài module, có lãi đo được, tổ chức chấp nhận được một tỉ lệ ổn định | Trở thành "sprint dọn dẹp" không đổi nguyên nhân; bucket bị mượn khi gấp và không bao giờ trả lại | Bucket phải gắn với entry cụ thể trong register, không phải "việc kỹ thuật chung"; nếu bị mượn thì ghi lại số lần |
| **Strangler** | Dựng đường mới bên cạnh, chuyển caller dần, tắt đường cũ khi hết caller | Debt lớn, module còn kỳ hạn dài, không thể dừng phát triển tính năng | Giai đoạn hai đường song song rất tốn; nếu không tắt được đường cũ thì tệ hơn trạng thái ban đầu | Bắt buộc có: điều kiện tắt đường cũ, ngày dự kiến tắt, và số caller còn lại là một chỉ số được theo dõi công khai |

**Bước 4 — Xin ngân sách bằng ngôn ngữ tiền.** Nối trực tiếp với cost-benefit ở chương 04. Cấu trúc
bốn câu, dùng được trong 90 giây với một người không kỹ thuật:

1. **Việc gì đang bị chặn** (nói bằng hạng mục roadmap, không bằng tên module): "ba hạng mục Q3 đều
   chạm vùng này."
2. **Lãi đang trả, bằng người-ngày hoặc bằng tiền**: "khoảng 34 người-ngày mỗi quý, tương đương một
   người làm cả quý."
3. **Gốc và thời gian hoàn vốn**: "20 người-ngày, hoàn vốn trong khoảng một quý."
4. **Cái gì sẽ không được làm nếu chi 20 người-ngày này** — và đây là câu làm đề xuất của bạn đáng tin
   cậy: "để làm việc này, hạng mục X sẽ lùi 2 tuần. Tôi đề nghị lùi X vì nó không có ràng buộc thời
   điểm bên ngoài."

Câu thứ tư là câu phân biệt một Tech Lead với một người xin xỏ. Người ra quyết định luôn biết rằng mọi
thứ đều có chi phí cơ hội; nếu bạn không nêu nó, họ sẽ giả định bạn chưa nghĩ tới và giảm mức tin cậy
cho cả ba câu đầu.

### Trade-off

**Trade-off chính — Short-term Delivery vs Long-term Quality.** Điểm quan trọng nhất của mục này: **vay
debt là quyết định đúng trong một số điều kiện xác định được**, và một lead không nhận ra điều đó sẽ
làm chậm tổ chức nhân danh chất lượng.

| Điều kiện | Vay debt là ĐÚNG | Vì sao |
|---|---|---|
| Có cửa sổ thị trường thật với ngày cụ thể | Có | Giá trị của việc vào thị trường đúng cửa sổ có thể lớn hơn toàn bộ chi phí lãi trong 2 năm. Ví dụ: kịp mùa campaign, kịp yêu cầu compliance có hạn, kịp trước khi đối thủ ra mắt |
| Sản phẩm chưa có product-market fit | Có, mạnh | Xác suất đáng kể là toàn bộ code này bị bỏ. Đầu tư thiết kế cho code sẽ bị xoá là chi phí thuần. Ở đây, tối ưu cho tốc độ học, không cho khả năng bảo trì |
| Prototype đã tuyên bố sẽ bị bỏ, có ngày | Có | Không có tương lai để trả lãi |
| Cần dữ liệu để quyết định thiết kế đúng | Có | Thiết kế "đúng" chưa biết được; xây tạm để học ranh giới thật rồi làm lại thường rẻ hơn thiết kế trước cho một mô hình sai (đây là ô inadvertent-prudent của Fowler) |
| Cạn runway, cần một mốc để gọi vốn | Có, nhưng ghi lại | Sống sót là điều kiện tiên quyết cho mọi chất lượng dài hạn |
| "Chúng ta luôn gấp" | Không | Đây không phải cửa sổ thị trường; đây là thất bại của việc ưu tiên. Vay ở đây là ô deliberate-reckless và nó không hoàn được |
| Vùng dữ liệu tiền, dữ liệu không tạo lại được, hoặc ràng buộc compliance | Không | Chi phí sai không phải thời gian mà là mất tiền, mất giấy phép, mất dữ liệu. Ở đây "nợ" không phải nợ mà là rủi ro không giới hạn |
| Vay lần thứ ba vào cùng một vùng | Không | Lãi kép đã vào vùng phi tuyến; và ba lần vay vào cùng một chỗ là bằng chứng thiết kế hiện tại không chịu được loại thay đổi này |

Điều kiện để việc vay là prudent thay vì reckless — bốn thứ, thiếu một là reckless:

1. Có **bản ghi** (một dòng trong debt register hoặc một mục trong ADR), không phải chỉ trong đầu ai đó.
2. Có **ước lượng lãi** — biết mình đang trả bao nhiêu mỗi quý.
3. Có **điều kiện trả** — không nhất thiết là ngày, có thể là trigger ("khi hạng mục tiếp theo chạm vùng
   này").
4. Có **tên người** biết về nó và có trách nhiệm nhắc lại.

**Trade-off phụ — Trả debt bằng dedicated capacity vs trả bằng cách gắn vào tính năng.** Dedicated cho
tiến độ đo được và bảo vệ được khỏi áp lực giao hàng, nhưng dễ mất kết nối với giá trị kinh doanh và bị
cắt đầu tiên khi khó khăn. Gắn vào tính năng thì luôn có business case và không bao giờ bị cắt, nhưng
không xử lý được debt ở những vùng mà roadmap không chạm tới trong 12 tháng — và những vùng đó chính là
nơi rủi ro tích tụ âm thầm (thư viện hết support, chứng chỉ hết hạn, phiên bản database không còn bản
vá). Điều kiện nghiêng: dùng dedicated cho debt loại rủi ro (bảo mật, hết support, compliance) và gắn
vào tính năng cho debt loại ma sát.

### Real-world Scenarios

**Tình huống A — Đề xuất big rewrite và cách nó bị thay thế bằng một phương án tốt hơn.** Bối cảnh: một
sàn thương mại điện tử, hệ thống 7 năm, monolith PHP 400k dòng, 32 engineer. Tuấn (Senior BE) trình bày
một đề xuất viết lại toàn bộ backend sang một stack mới, ước lượng 8 tháng với 6 người.

Minh không phản đối bằng cách tranh luận về stack. Anh đề nghị Tuấn trả lời bốn câu trong một tuần:

1. **Trong 8 tháng đó, hệ thống cũ có ngừng thay đổi không?** Câu trả lời thật: không — roadmap có 14
   hạng mục và business không dừng. Nghĩa là phải làm cả hai hệ thống song song trong 8 tháng, hoặc
   đóng băng roadmap 8 tháng. Không phương án nào được chấp nhận.
2. **Số 8 tháng dựa trên gì?** Dựa trên ước lượng phần chức năng đã biết. Nhưng phần đắt nhất của việc
   viết lại là các hành vi không ai biết là có: các trường hợp đặc biệt cho 6 đối tác, các quy tắc thuế
   theo từng thời kỳ, các bản vá lỗi tích tụ 7 năm mà mỗi bản vá là một bài học đã trả tiền. Số này
   không ước lượng được từ code, chỉ ước lượng được từ số incident lịch sử.
3. **Ai vận hành hai hệ thống trong giai đoạn chuyển?** Team on-call hiện có 6 người; hai hệ thống nghĩa
   là hai bộ runbook và gấp đôi diện tích lỗi trong 8+ tháng.
4. **Nếu tháng thứ 6 chúng ta phải dừng vì lý do kinh doanh, chúng ta có gì?** Với big rewrite: gần như
   không có gì dùng được. Đây là câu hỏi quyết định — nó cho thấy big rewrite là một quyết định one-way
   door không có điểm thoát trung gian.

Phương án được chọn: strangler theo domain, với thứ tự dựa trên debt register. Bắt đầu bằng module có
lãi cao nhất và ranh giới rõ nhất (thanh toán, đã có ADR-014 ở chủ đề 2), rồi tới tìm kiếm, rồi tới giỏ
hàng. Mỗi module tách ra là một đơn vị giao được trong 6–10 tuần, có giá trị độc lập, và nếu phải dừng
ở bất kỳ thời điểm nào thì phần đã làm vẫn dùng được. Sau 14 tháng (số minh hoạ): 4 domain đã tách,
khoảng 45% lưu lượng trên hệ thống mới, monolith vẫn còn nhưng lãi đã giảm ở các vùng đắt nhất, và
roadmap không bị đóng băng ngày nào.

Bài học tổng quát hơn cả câu chuyện: **cách đánh giá một đề xuất trả debt lớn không phải là hỏi "kiến
trúc đích có tốt hơn không" — nó gần như luôn tốt hơn. Câu hỏi đúng là "con đường tới đó có giá trị ở
từng đoạn không, và có điểm thoát ở đâu".**

**Tình huống B — "Sprint dọn dẹp" mỗi quý mà debt vẫn tăng, và ba góc nhìn.** Một công ty SaaS 20
engineer áp dụng "cleanup sprint": mỗi quý dành trọn một sprint 2 tuần cho technical debt. Sau bốn quý,
các tín hiệu debt (lead time, change failure rate) không cải thiện.

*Nhìn từ IC (Nam):* "Cleanup sprint là hai tuần dễ chịu. Nhưng tôi chọn việc theo cái tôi thấy khó
chịu nhất, thường là những chỗ tôi vừa chạm, không phải chỗ đắt nhất. Và trong 12 tuần còn lại, mỗi
lần tôi định làm cẩn thận thì có người hỏi 'cái này bao giờ xong', nên tôi làm nhanh. Tôi biết mình
đang tạo debt, và tôi cũng biết là sẽ có cleanup sprint."

Câu cuối là câu chẩn đoán. Cleanup sprint đã trở thành một cơ chế **cho phép** tạo debt — nó chuyển
debt từ một thứ phải cân nhắc thành một thứ có chỗ xử lý sau, và theo đúng lý thuyết khuyến khích, cái
gì có chỗ xử lý sau thì được tạo nhiều hơn.

*Nhìn từ Tech Lead (Minh):* "Tôi đã đo. Trong 4 cleanup sprint, 60% công việc là ở các module có lãi
thấp — người ta chọn việc mình thích, không phải việc đắt nhất, vì không có register nào để xếp thứ tự.
Và nguyên nhân tạo debt thì tôi biết rõ: định nghĩa 'xong' của chúng ta không có test cho luồng lỗi, và
mọi cam kết sprint đều được đặt ở mức 100% capacity nên không có chỗ cho việc làm đúng. Hai tuần mỗi
quý không sửa được hai điều đó."

*Nhìn từ Manager (Linh):* "Cleanup sprint là thứ tôi bán được cho ban lãnh đạo, vì nó có hình dạng dễ
hiểu và có thời hạn. Nếu tôi xin 15% capacity liên tục, tôi phải giải thích mỗi quý và nó sẽ bị cắt
trong quý nào doanh thu xấu. Nhưng bây giờ tôi thấy vấn đề: tôi đã tối ưu cho việc dễ bán, không cho
việc có tác dụng. Cái tôi cần làm là đổi định nghĩa 'xong' — đó là thay đổi không cần ngân sách và
không ai phải phê duyệt — và đổi cách chúng ta cam kết sprint từ 100% xuống 80% capacity."

Ba thay đổi được làm: (1) lập debt register có ước lượng lãi, và cleanup chỉ được làm việc trong
register theo thứ tự ưu tiên; (2) đưa test cho luồng lỗi vào definition of done cho các module trong
register; (3) cam kết sprint ở 80% capacity. Bỏ cleanup sprint, đổi sang 15% bucket mỗi sprint. Sau hai
quý (số minh hoạ): lead time ở hai module đắt nhất giảm khoảng 40%, và điều đáng chú ý là tổng số điểm
giao mỗi sprint không giảm — vì phần lãi tiết kiệm được bù lại phần capacity bỏ ra.

### Best Practices

- **Đo debt theo module, không theo hệ thống.** Cơ chế: chỉ số toàn hệ thống bị trung bình hoá và ẩn
  đúng chỗ cần xử lý; và mọi hành động trả debt đều xảy ra ở cấp module.
- **Xếp ưu tiên theo lãi × tần suất thay đổi × kỳ hạn, không theo mức độ tệ.** Cơ chế: module tệ nhất
  mà không ai sửa có lãi bằng không; trả nó là tiêu tiền để có cảm giác sạch sẽ.
- **Mỗi lần vay debt có ý thức, viết một dòng ngay lúc vay, không để sau.** Cơ chế: thông tin về lý do
  vay và mức lãi ước tính chỉ có đầy đủ tại thời điểm vay; sau hai tuần nó đã mất một nửa.
- **Giữ debt register ở 15–20 dòng, và mỗi quý bắt buộc đóng một số dòng ở trạng thái "won't fix".** Cơ
  chế: một register 200 dòng là một danh sách không ai đọc; và việc chủ động đóng dòng là cách duy nhất
  giữ nó có tín hiệu.
- **Trả debt bằng các PR nhỏ ship được độc lập, không bằng một nhánh refactor dài.** Cơ chế: nhánh
  refactor dài có xác suất bị bỏ cao và tạo conflict tăng theo thời gian; PR nhỏ có tiến độ quan sát
  được và có thể dừng bất kỳ lúc nào mà phần đã làm vẫn có giá trị.
- **Viết test đặc tả hành vi hiện tại trước khi refactor vùng không có test.** Cơ chế: bạn không thể
  refactor an toàn thứ mà bạn không biết hành vi đúng của nó; và các test này thường phát hiện ra rằng
  "hành vi đúng" hiện tại chứa vài bug mà nghiệp vụ đã phụ thuộc vào.
- **Nối mọi đề xuất trả debt với một hạng mục roadmap cụ thể.** Cơ chế: nó cho business case sẵn có, và
  nó tự động xếp thứ tự ưu tiên theo giá trị kinh doanh thay vì theo cảm nhận kỹ thuật.
- **Sau khi trả, đo lại và công bố chênh lệch.** Cơ chế: đây là dữ liệu duy nhất làm cho lần xin ngân
  sách tiếp theo dễ hơn; và nếu số không cải thiện, bạn cần biết để không tiếp tục theo hướng sai.

### Anti-patterns

| Anti-pattern | Cơ chế phá hệ thống | Dấu hiệu sớm |
|---|---|---|
| **Refactor lớn không có business case** | Tiêu capacity mà không ai bảo vệ được khi có áp lực → bị cắt giữa đường → để lại hệ thống ở trạng thái nửa vời, tệ hơn cả hai đầu | Đề xuất không nêu được hạng mục roadmap nào bị chặn; không có ước lượng lãi bằng số |
| **"Sprint dọn dẹp" mỗi quý mà không đổi nguyên nhân** | Trở thành cơ chế cho phép tạo debt (có chỗ xử lý sau → tạo nhiều hơn); và công việc trong sprint đó được chọn theo sở thích chứ theo lãi | Debt register không tồn tại; tín hiệu debt không cải thiện sau 2–3 chu kỳ; definition of done không đổi |
| **Big rewrite** | Không có điểm thoát trung gian; hệ thống cũ vẫn phải thay đổi song song; các hành vi ẩn tích tụ nhiều năm không ước lượng được nên ước lượng luôn sai theo hướng lạc quan | Ước lượng dạng "khoảng 6–9 tháng"; không có kế hoạch cho việc chạy song song; không trả lời được "nếu dừng ở tháng thứ 6 thì có gì" |
| **Debt register là backlog thứ hai với 200 dòng** | Không có tín hiệu; không ai đọc; và sự tồn tại của nó tạo cảm giác đang quản lý debt trong khi không | Số dòng tăng đều mỗi quý; không có dòng nào bị đóng ở trạng thái "won't fix" |
| **Gọi mọi thứ mình không thích là technical debt** | Làm loãng khái niệm nên khi có debt thật, business không phân biệt được và không ưu tiên | Trong register có các dòng về style, về "công nghệ cũ" mà không có ước lượng lãi |
| **Trả debt trong im lặng, không ghi và không đo** | Không có dữ liệu cho lần xin ngân sách sau; và khi ai đó hỏi "hai tuần qua team làm gì", không có câu trả lời — điều này làm mất tin cậy nhanh hơn cả việc không trả debt | Không ai ngoài team biết công việc đó đã làm |
| **Ép ngưỡng coverage làm mục tiêu** | Goodhart's law: chỉ số bị thao tác — test rỗng, test khẳng định điều hiển nhiên — và tổ chức có coverage cao với chất lượng không đổi, cộng thêm một khối test phải bảo trì | Coverage tăng nhanh trong khi change failure rate không giảm |

### Khi nào KHÔNG nên áp dụng

**Khi module sắp bị thay thế hoặc ngừng sử dụng.** Nếu kỳ hạn còn lại ngắn hơn thời gian hoàn vốn, trả
debt là tiêu tiền không thu hồi được. Dòng TD-22 trong register mẫu là ví dụ: 25 người-ngày gốc cho một
module còn 9 tháng và chỉ nhận 2 PR mỗi năm. Việc đúng ở đây là **quyết định không trả, viết lý do, và
đóng dòng đó** — để không ai phải cân nhắc lại mỗi quý.

**Khi công ty đang trong tình trạng sống còn.** Runway 4 tháng, cần một mốc để gọi vốn hoặc để đạt hợp
đồng cứu tử. Ở đây tối ưu đúng là tốc độ, và vay debt có ý thức là quyết định của một lead giỏi, không
phải sự thoả hiệp. Điều kiện duy nhất: ghi lại. Bốn tháng sau, nếu công ty sống, bản ghi đó là thứ giúp
bạn trả nợ theo thứ tự đúng thay vì theo tiếng ồn.

**Khi bạn chưa có dữ liệu về lãi.** Bắt đầu một chương trình trả debt dựa trên cảm nhận sẽ dẫn tới việc
trả ở những chỗ gây khó chịu nhất chứ không đắt nhất, và sau hai quý bạn sẽ không chứng minh được kết
quả — làm mất khả năng xin ngân sách lần sau. Thứ đáng làm trước: bỏ một tuần để có bốn tín hiệu ở Bước
1 theo module. Chi phí thấp, và nó thường đảo ngược thứ tự ưu tiên mà bạn tưởng mình biết.

**Khi debt thật ra là vấn đề về năng lực hoặc về ưu tiên.** Nếu debt được sinh ra ở ô
inadvertent-reckless (không ai trong team biết cách thiết kế đúng), thì trả debt xong nó sẽ được tạo
lại trong hai quý. Việc phải làm trước là mentoring, pairing, tuyển một người có kinh nghiệm hơn, hoặc
đưa chuẩn vào công cụ (chủ đề 6). Tương tự với ô deliberate-reckless: nếu nguyên nhân là mọi sprint đều
cam kết ở 110% capacity, thì trả debt là múc nước trong khi vòi vẫn mở.

**Khi việc trả debt đòi hỏi đóng băng roadmap.** Bất kỳ đề xuất nào có dạng "cho chúng tôi 2 tháng
không làm tính năng gì" gần như luôn bị từ chối, và nếu được chấp nhận thì gần như luôn bị cắt giữa
đường khi có một yêu cầu gấp. Cách đúng không phải thuyết phục mạnh hơn — mà là **chia lại công việc**
thành các đơn vị 1–2 tuần ship được độc lập. Nếu bạn không chia được, đó là thông tin quan trọng về bản
thân debt đó: nó có thể chưa được hiểu đủ rõ để trả.

---

## 6. Engineering Standards và Golden Path

### Problem Statement

Cùng công ty e-commerce ở đầu chương. Một engineer mới, Vy, vào team Promotion. Nhiệm vụ tuần đầu: dựng
một service nhỏ để tính điểm tích luỹ. Ghi lại thời gian thực tế của cô ấy (số minh hoạ, nhưng phân bố
này thì rất phổ biến):

| Việc | Thời gian | Ghi chú |
|---|---|---|
| Viết business logic của service | 1,5 ngày | Phần duy nhất tạo giá trị |
| Tìm hiểu nên đặt repo ở đâu, cấu trúc thế nào | 0,5 ngày | Ba service tham khảo có ba cấu trúc khác nhau |
| Dựng CI/CD | 2 ngày | Copy từ service khác, sửa 40 dòng YAML, hỏi Duy 6 lần |
| Cấu hình logging, metrics, tracing | 1,5 ngày | Không biết dashboard nào là dashboard chuẩn |
| Xin secret, cấu hình quyền truy cập database | 1 ngày | Chờ Duy, Duy đang trong sprint khác |
| Cấu hình health check, deploy lên staging | 1 ngày | Staging có hai cách deploy, chọn sai một lần |
| Tổng | 7,5 ngày | Trong đó 6 ngày là chi phí "không phải bài toán" |

Tám mươi phần trăm thời gian của một service mới đi vào những việc mà mọi service đều phải làm và đều
đã có ai đó làm trước. Đó là chi phí thuần của việc không có golden path. Và có một chi phí thứ hai
đắt hơn: mỗi lần một người tự dựng, họ tạo ra một biến thể mới — nên số cách làm cùng một việc tăng
đều theo số service, và cognitive load của cả tổ chức tăng theo.

Ba hiện tượng khác của cùng vấn đề, đều đếm được:

- **Trang "Engineering Standards" trên wiki có ngày cập nhật cuối cách đây 14 tháng, và trong 30 ngày
  qua có 3 lượt xem.** Kiểm tra được trong 1 phút với analytics của wiki.
- **Tỉ lệ service tuân thủ chuẩn giảm theo tuổi của chuẩn.** Đo bằng cách chọn 5 quy tắc trong tài liệu
  chuẩn và đếm số service vi phạm. Nếu chuẩn được thực thi bằng văn bản thay vì bằng công cụ, tỉ lệ vi
  phạm tăng đều theo thời gian và theo số người mới.
- **Mọi công nghệ mới đều bị chặn, và engineer giỏi bắt đầu học ở nhà thay vì ở công ty.** Dấu hiệu:
  không có công nghệ nào mới được đưa vào tổ chức trong 18 tháng, đồng thời có 2–3 người nghỉ với lý do
  "muốn làm với stack hiện đại hơn".

### First Principles

**Cơ chế một: standard tồn tại để giảm cognitive load và chi phí phối hợp, không để thể hiện quyền
lực.** Cognitive load — theo cách Team Topologies dùng thuật ngữ này — là tổng lượng thứ một engineer
phải giữ trong đầu để làm việc. Nó hữu hạn, và mỗi biến thể trong hệ thống chiếm một phần. Khi có ba
cách gửi thông báo, một engineer on-call phải giữ ba mô hình; khi có một cách, họ dùng phần dung lượng
còn lại để suy nghĩ về bài toán nghiệp vụ. Suy ra một phép thử cho mọi standard được đề xuất: **nó
giảm cognitive load của ai, và tăng của ai?** Một standard buộc mọi team dùng một công cụ mà chỉ team
platform hiểu thì giảm load cho platform và tăng cho tất cả — đó là standard sai, và nó thường được
biện minh bằng từ "chuẩn hoá".

**Cơ chế hai: mọi standard là một đánh đổi giữa tự do cục bộ và hiệu quả toàn cục, và tỉ lệ đánh đổi
phụ thuộc vào số lượng đơn vị phải phối hợp.** Toán học của việc này đơn giản: lợi ích của chuẩn hoá
tăng theo số team (vì số cặp phải hiểu nhau tăng theo n²), còn chi phí của chuẩn hoá — mất tối ưu cục
bộ — tăng gần tuyến tính theo số team. Nên tồn tại một ngưỡng n mà trên đó chuẩn hoá thắng. Với các
quyết định ở giao diện giữa các team, ngưỡng đó rất thấp (2–3 team đã đủ). Với các quyết định bên trong
một team, ngưỡng đó rất cao, có thể không bao giờ đạt được. Đây là cơ sở của nguyên tắc "chuẩn hoá
giao diện, tự chủ bên trong" ở chủ đề 4.

**Cơ chế ba: con người tuân thủ theo đường ít lực cản nhất, không theo quy định.** Đây là kết quả vững
nhất trong thiết kế hành vi và nó có một hệ quả rất cụ thể: **nếu con đường tuân thủ chuẩn dài hơn con
đường tự làm, chuẩn sẽ bị vi phạm, và tỉ lệ vi phạm không phụ thuộc vào việc bạn nhắc bao nhiêu lần.**
Suy ra chiến lược duy nhất bền: đừng làm con đường sai khó đi, hãy làm con đường đúng dễ đi hơn. Đó
chính là định nghĩa của golden path. Nói cách khác: **standard được thực thi bằng công cụ thì tồn tại;
standard được thực thi bằng quy trình duyệt thì phân rã.**

**Cơ chế bốn: chi phí của một quy tắc gồm cả chi phí của các ngoại lệ hợp lý mà nó chặn.** Mọi quy tắc
được viết cho trường hợp phổ biến, và mọi hệ thống thật có trường hợp biên. Một quy tắc không có cơ chế
ngoại lệ sẽ gặp một trong hai kết cục: hoặc nó bị vi phạm âm thầm (và tổ chức mất khả năng biết trạng
thái thật của mình), hoặc nó buộc người ta chọn giải pháp tệ hơn để tuân thủ. Do đó **cơ chế xin ngoại
lệ không phải sự thoả hiệp của standard — nó là bộ phận làm standard hoạt động được.** Điều kiện: ngoại
lệ phải có thời hạn, vì ngoại lệ vô hạn định trở thành tiêu chuẩn thứ hai trong khoảng hai quý.

### Mental Model

**Ba mức ràng buộc — must / should / may.** Vay từ cách RFC của IETF dùng các từ này, nhưng điểm quan
trọng là gắn mỗi mức với **ai được quyết** và **cơ chế thực thi nào**:

| Mức | Nghĩa | Ai được quyết mức này | Cơ chế thực thi | Ngoại lệ |
|---|---|---|---|---|
| **MUST** | Vi phạm là chặn merge/deploy | Head of Engineering hoặc CTO, sau RFC. Số lượng phải rất ít — 10–15 quy tắc cho cả tổ chức | Tự động: CI fail, policy-as-code, lint chặn, kiểm tra hạ tầng | Chỉ qua exception có thời hạn, có tên người phê duyệt và ngày hết hạn |
| **SHOULD** | Mặc định đúng; lệch được nhưng phải ghi lý do một dòng | Tech Lead của vùng đó, hoặc nhóm Platform | Cảnh báo trong CI (không chặn), checklist trong PR template | Ghi lý do trong PR là đủ |
| **MAY** | Khuyến nghị, có ví dụ, không ràng buộc | Bất kỳ ai; thường là kết quả của một cuộc thảo luận trong team | Không có; chỉ là tài liệu và template có sẵn | Không cần |

Lỗi phổ biến nhất trong thực tế: viết mọi thứ ở mức MUST vì nó nghe nghiêm túc hơn, rồi không thực thi
được cái nào — và khi một MUST không được thực thi, nó dạy cả tổ chức rằng mọi MUST khác cũng chỉ là
gợi ý. **Số lượng MUST phải nhỏ đến mức bạn thực thi được tất cả bằng công cụ.** Nếu bạn có 40 quy tắc
MUST, bạn thực tế có 0.

**Golden path như một sản phẩm nội bộ.** Golden path là con đường mặc định đã được lắp sẵn: một lệnh
tạo service mới, và service đó ra đời với repo đúng cấu trúc, CI/CD chạy được, logging/metrics/tracing
đã nối vào dashboard chung, health check, secret management, security baseline, và một PR template.
Cách đúng để nghĩ về nó: **golden path là một sản phẩm có khách hàng (engineer nội bộ), có đối thủ cạnh
tranh (việc tự làm), và nó chỉ thắng khi nó tiện hơn đối thủ.** Suy ra ba hệ quả: nó cần một owner, nó
cần được đo bằng tỉ lệ sử dụng (adoption), và khi tỉ lệ sử dụng thấp thì kết luận đúng là "sản phẩm
chưa đủ tốt", không phải "engineer không tuân thủ".

**Paved road vs guard rail.** Hai cơ chế bổ sung: paved road (đường trải nhựa) làm con đường đúng nhanh
hơn; guard rail (rào chắn) làm con đường sai khó đi hoặc phát hiện được. Cần cả hai, nhưng theo thứ tự:
**paved road trước, guard rail sau.** Nếu bạn dựng guard rail trước khi có đường trải nhựa, bạn chỉ làm
mọi việc chậm đi và tạo ra một mối quan hệ đối kháng giữa platform và các team sản phẩm — điều rất khó
sửa sau này.

### Practical Framework

**Bước 1 — Quyết cái gì cần chuẩn hoá, bằng hai câu hỏi.** Chỉ chuẩn hoá thứ thoả ít nhất một trong hai:
(a) nó đi qua ranh giới giữa các team; (b) người on-call phải chạm vào nó lúc 2 giờ sáng. Bảng ứng dụng:

| Hạng mục | Đi qua ranh giới? | On-call chạm? | Mức đề xuất |
|---|---|---|---|
| Format log, trace ID, correlation ID | Có | Có | MUST |
| Cách phát và tiêu thụ event (schema, versioning) | Có | Có | MUST |
| Cách xác thực giữa các service nội bộ | Có | Có | MUST |
| Quản lý secret, không hardcode credential | Không | Có | MUST |
| Health check endpoint, cách deploy, cách rollback | Không | Có | MUST |
| Baseline bảo mật: scan dependency, không dùng lib có CVE High | Có (rủi ro chung) | Đôi khi | MUST |
| Cách đặt tên và versioning API public | Có | Có | MUST |
| Ngôn ngữ backend | Có (tuyển dụng, luân chuyển) | Có (kỹ năng debug) | SHOULD, có danh sách được phép |
| Framework web trong một service | Không | Không (nếu log/metric đã chuẩn) | MAY |
| Cấu trúc thư mục trong service | Không | Không | MAY (có template làm mặc định) |
| Thư viện test, cách viết assert | Không | Không | MAY |
| ORM, query builder | Không | Không | MAY |
| Cách chia commit, quy ước commit message | Không | Đôi khi (khi bisect) | SHOULD |

Bảng này là bảng đáng tranh luận trong team bạn hơn là đáng sao chép. Nhưng chú ý một quy luật: **số
hạng mục MUST rất ít, và tất cả đều nằm ở giao diện hoặc ở đường vận hành.** Không có hạng mục nào ở
mức MUST liên quan tới cách viết code bên trong.

**Bước 2 — Xây golden path theo thứ tự lợi ích/chi phí.** Đừng làm một nền tảng nội bộ hoàn chỉnh; làm
theo thứ tự này, mỗi bước ship được và có giá trị ngay:

| Thứ tự | Hạng mục | Chi phí (minh hoạ) | Lợi ích ngay |
|---|---|---|---|
| 1 | Service template (một lệnh tạo repo mới có cấu trúc, Dockerfile, health check, PR template) | 5–8 người-ngày | Cắt 1–1,5 ngày mỗi service mới; và nó là nơi neo mọi chuẩn về sau |
| 2 | CI/CD pipeline dùng chung, tham số hoá, không copy-paste | 8–12 người-ngày | Cắt 2 ngày mỗi service; và mọi thay đổi pipeline làm một lần cho tất cả |
| 3 | Observability mặc định: log format, metrics chuẩn, tracing, dashboard tự sinh | 10–15 người-ngày | Cắt 1,5 ngày mỗi service; và giảm MTTR cho mọi service |
| 4 | Secret/config management và quyền truy cập tự phục vụ | 8–12 người-ngày | Cắt 1 ngày và bỏ được sự phụ thuộc vào Duy — đây thường là điểm nghẽn lớn nhất |
| 5 | Security baseline tự động: scan dependency, scan secret trong commit, policy-as-code | 5–10 người-ngày | Biến một nhóm MUST từ văn bản thành cơ chế |
| 6 | Tài liệu golden path: một trang, dạng "làm X thì chạy lệnh Y" | 2 người-ngày | Chỉ có giá trị **sau khi** 1–5 tồn tại |

Chú ý thứ tự của hạng mục 6. Viết tài liệu chuẩn trước khi có công cụ là cách phổ biến nhất để tạo ra
một trang wiki 14 tháng không ai đọc.

**Bước 3 — Cơ chế xin ngoại lệ có thời hạn.** Đây là phần làm standard tồn tại được thay vì bị lách.

```markdown
# Exception Request — [tên hạng mục]

- ID: EXC-023 | Ngày: 2026-07-02 | Hết hạn: 2026-10-02 (tối đa 2 quý cho một lần xin)
- Người xin: Hà (Tech Lead team Promotion)
- Quy tắc xin ngoại lệ: MUST-04 "mọi service phát event qua bus chung theo schema đã đăng ký"
- Người phê duyệt: Khoa (CTO)   | Người theo dõi hết hạn: Duy (Platform)

## 1. Xin ngoại lệ để làm gì
Service `loyalty` cần gọi trực tiếp HTTP tới `promotion` thay vì qua event, cho luồng
tính điểm thời gian thực.

## 2. Vì sao quy tắc không áp dụng được ở đây
Yêu cầu nghiệp vụ: người dùng phải thấy điểm cập nhật trong dưới 2 giây tại màn hình thanh toán.
Event bus hiện tại có độ trễ p99 khoảng 4,5 giây do batch consumer (đã đo, xem dashboard [link]).

## 3. Phương án đã thử để tuân thủ
- Giảm batch size của consumer: đã thử, p99 xuống 2,8s nhưng chi phí xử lý tăng ~30% và ảnh
  hưởng 6 consumer khác. Duy không đồng ý.
- Dùng một topic riêng độ trễ thấp: khả thi nhưng cần 6 người-ngày của Platform, không có
  capacity trong Q3.

## 4. Rủi ro của ngoại lệ và cách giảm
- Rủi ro: coupling đồng bộ giữa hai service; nếu `promotion` chậm thì màn hình thanh toán chậm.
- Giảm: timeout 800ms, circuit breaker, và fallback hiển thị điểm từ cache (chấp nhận cũ 5 phút).

## 5. Điều kiện để ngoại lệ này kết thúc
- Khi topic độ trễ thấp có sẵn (Platform dự kiến Q4), chuyển trong vòng 2 sprint; hoặc
- Khi hết hạn 02/10 mà chưa chuyển: Hà phải xin gia hạn với dữ liệu mới, KHÔNG tự động gia hạn.

## 6. Ghi vào debt register
TD-27, lãi ước tính 2 người-ngày/quý (chi phí vận hành thêm của circuit breaker + rủi ro).
```

**Bước 4 — Đo và xét lại standard mỗi quý.** Bốn chỉ số, mỗi cái dẫn tới một hành động khác nhau:

| Chỉ số | Cách đo | Nếu xấu thì làm gì |
|---|---|---|
| Adoption của golden path | % service mới trong quý dùng template | < 70%: golden path chưa đủ tiện, không phải engineer chưa tuân thủ. Đi phỏng vấn 3 người đã không dùng |
| Thời gian từ "cần một service mới" tới "deploy được lên staging" | Đo với người mới nhất | > 1 ngày: còn hạng mục trong Bước 2 chưa làm |
| Số exception đang mở và số exception quá hạn | Đếm | Nhiều exception cho cùng một quy tắc → quy tắc đó sai, không phải người ta lách. Xét lại quy tắc |
| Số MUST không có cơ chế thực thi tự động | Đếm | > 3: các MUST đó đang phân rã. Hoặc tự động hoá, hoặc hạ xuống SHOULD cho trung thực |

### Trade-off

**Trade-off chính — Standardization vs Flexibility, theo số lượng team.**

| Số team | Nghiêng về | Cơ chế phù hợp | Vì sao |
|---|---|---|---|
| 1 team (≤ 8 người) | Flexibility gần như hoàn toàn | Quy ước miệng + formatter. Không viết standard | Mọi người biết mọi thứ; chi phí phối hợp gần bằng không; viết standard là chi phí không có lợi ích |
| 2–3 team | Chuẩn hoá tối thiểu ở giao diện | Chuẩn cho: log format, cách gọi nhau, cách deploy. Còn lại tự do. Có service template nếu hay tạo service mới | Số cặp phối hợp là 1–3, còn quản được bằng trao đổi; nhưng giao diện đã bắt đầu tạo chi phí |
| 4–8 team | Golden path thật, MUST rõ ràng nhưng ít | Đủ 6 hạng mục ở Bước 2; 10–15 MUST được thực thi tự động; cơ chế exception chính thức | Số cặp phối hợp 6–28; on-call chéo bắt đầu cần thiết; không chuẩn hoá thì cognitive load vượt ngưỡng |
| > 8 team | Chuẩn hoá mạnh ở giao diện, nhưng bắt buộc phải có nhóm Platform có ngân sách và on-call | Platform as a product: golden path có owner, có roadmap, có SLA, có khảo sát người dùng nội bộ | Ở quy mô này, chuẩn hoá bằng văn bản và duyệt hoàn toàn không scale; chỉ chuẩn hoá bằng công cụ mới tồn tại |

**Trade-off phụ — Chuẩn hoá công nghệ (danh sách được phép) vs cho phép thử nghiệm.** Danh sách được
phép cho lợi ích rõ: tuyển được người, luân chuyển nội bộ dễ, on-call chéo khả thi, và giảm số thứ phải
vận hành. Chi phí thường bị đánh giá thấp: nó dừng dòng công nghệ mới vào tổ chức, và trong một thị
trường lao động mà engineer giỏi chọn nơi làm việc theo cơ hội học tập, đó là chi phí nhân sự thật. Cách
xử lý hiệu quả trong thực tế là một **cơ chế thử nghiệm có ràng buộc**, không phải một cánh cửa đóng:

| Vòng | Điều kiện | Ràng buộc |
|---|---|---|
| Vòng 1 — thử nghiệm nội bộ | Bất kỳ ai, cho công cụ nội bộ hoặc service không nằm trên đường doanh thu | Không được đưa lên production luồng chính; có ngày kết thúc; viết lại 1 trang kết quả |
| Vòng 2 — thí điểm production | Cần một sponsor mức Tech Lead + xác nhận có ai vận hành được ngoài người đề xuất | Một service duy nhất; có kế hoạch rút; theo dõi 1 quý với chỉ số so sánh |
| Vòng 3 — vào danh sách được phép | Cần RFC: ai vận hành, chi phí, đường onboarding, và **cái gì bị loại khỏi danh sách để đổi lại** | Vào danh sách kèm golden path support; nếu không có golden path thì chỉ ở mức MAY |

Ràng buộc "cái gì bị loại khỏi danh sách để đổi lại" là ràng buộc quan trọng nhất và ít tổ chức nào áp
dụng. Không có nó, danh sách công nghệ được phép chỉ tăng, và tổng chi phí vận hành tăng theo.

**Trade-off thứ ba — Standard áp bằng công cụ vs áp bằng quy trình duyệt.** Công cụ thực thi nhất quán,
không mệt, không thiên vị, và cho phản hồi ngay tại thời điểm engineer đang làm — đó là thời điểm sửa
rẻ nhất. Chi phí: phải xây, và một luật viết dở trong công cụ sẽ chặn cả những trường hợp hợp lý. Quy
trình duyệt linh hoạt hơn và bắt được những thứ công cụ không thấy (ý định, ranh giới miền, chất lượng
thiết kế), nhưng nó tạo hàng đợi, phụ thuộc vào người, và phản hồi tới muộn. Điều kiện nghiêng: dùng
công cụ cho mọi thứ kiểm tra được máy móc; giữ duyệt cho phán đoán thiết kế, và chỉ ở các quyết định
mức 3–4.

### Real-world Scenarios

**Tình huống A — Standard bị áp bằng quy trình duyệt và hệ quả sau 9 tháng.** Một ngân hàng số, phòng
engineering 80 người, 9 team. Sau một sự cố do một service dùng thư viện có lỗ hổng, CTO ra quy định:
mọi thư viện mới phải được Security Review phê duyệt trước khi dùng, qua một form và một buổi họp
tuần.

Sau 9 tháng, ba hiện tượng (số minh hoạ): thời gian chờ phê duyệt trung vị 11 ngày; 23 yêu cầu bị rút
vì "không kịp"; và điều tra sau một sự cố khác phát hiện 14 thư viện đang chạy trên production không
qua phê duyệt — engineer đã thêm chúng như dependency gián tiếp hoặc vào lúc gấp rồi không báo lại.

Cơ chế hỏng đúng như cơ chế ba ở phần First Principles: con đường tuân thủ (11 ngày chờ) dài hơn con
đường lách (thêm dependency và không nói gì), nên tỉ lệ lách tăng. Tệ hơn: tổ chức bây giờ **tin rằng**
mình kiểm soát được dependency, nên nó không đầu tư vào cơ chế thật. Quy trình duyệt không chỉ thất bại
— nó tạo ra một ảo giác an toàn, và ảo giác an toàn đắt hơn việc biết mình không an toàn.

Cách sửa, 6 tuần, do Duy (Platform) và một người từ Security làm cùng: (1) Đưa scan dependency vào CI,
chặn merge nếu có CVE mức High hoặc license không được phép — phản hồi trong 3 phút thay vì 11 ngày.
(2) Tạo một danh sách được phép dạng máy đọc, và các thư viện trong danh sách đi qua không cần hỏi ai.
(3) Với thư viện ngoài danh sách: tự động mở một issue có checklist, người đề xuất tự trả lời 5 câu, và
một người trong Security có SLA 2 ngày làm việc để xem — không cần họp. (4) Chạy scan trên toàn bộ
production hiện có, phát hiện và xử lý 14 thư viện chưa qua kiểm tra.

Kết quả: thời gian chờ trung vị từ 11 ngày xuống 2 ngày; số yêu cầu bị rút xuống gần 0; và số thư viện
không qua kiểm tra trên production về 0 và giữ được, vì bây giờ nó là một cơ chế chạy liên tục thay vì
một cuộc kiểm tra định kỳ. Nguyên tắc đáng ghi vào standard của bạn: **nếu một quy tắc chỉ được kiểm tra
bởi người, thì tỉ lệ tuân thủ thật của nó là một con số bạn không biết.**

**Tình huống B — Golden path thất bại vì làm ngược thứ tự.** Một startup 30 engineer, 5 team. Duy dành
một quý xây một CLI nội bộ khá đầy đủ để tạo service mới. Sau ba tháng, chỉ 2/7 service mới dùng nó.

Duy kết luận "team không hợp tác" và đề nghị CTO ra quy định bắt buộc. Minh đề nghị làm một việc khác
trước: phỏng vấn 5 người đã không dùng, mỗi người 20 phút. Kết quả gom lại thành ba lý do, và không lý
do nào là "không hợp tác":

| Lý do | Tần suất | Bản chất vấn đề |
|---|---|---|
| "Nó tạo ra cấu trúc không giống service của team tôi, tôi phải sửa lại nhiều hơn là tự làm" | 3/5 | Template được thiết kế theo ý platform, không theo cách các team thật đang làm |
| "Nó thiếu bước xin quyền truy cập database, mà đó là chỗ tôi mất một ngày" | 4/5 | Đã tự động hoá phần rẻ, bỏ qua phần nghẽn thật |
| "Tôi không biết nó có" | 2/5 | Không có kênh phát; và không có ai trong team dùng nó để làm tham chiếu |

Ba thay đổi được làm trong 5 tuần: (1) Duy ngồi cùng hai team, dựng lại template từ service thật của
họ thay vì từ ý tưởng của mình. (2) Ưu tiên tự động hoá phần xin quyền và secret — chính là phần nghẽn
— trước các phần đẹp hơn về kỹ thuật. (3) Đo adoption hàng tháng và coi đó là chỉ số của Platform, và
với mỗi trường hợp không dùng thì hỏi một câu duy nhất: "cái gì làm anh không dùng". Sau một quý:
6/7 service mới dùng golden path, không cần quy định bắt buộc nào.

Bài học tổng quát: **khi adoption thấp, giả thuyết đầu tiên phải là "sản phẩm chưa đủ tốt", không phải
"người dùng chưa tuân thủ".** Nếu bạn chuyển sang cưỡng chế trước khi kiểm tra giả thuyết đó, bạn sẽ có
tuân thủ hình thức và một mối quan hệ đối kháng giữa Platform và các team sản phẩm — mối quan hệ mà
theo kinh nghiệm mất 2–3 quý để sửa và làm mọi việc chuẩn hoá sau đó đắt hơn.

### Best Practices

- **Đưa mỗi MUST vào một cơ chế tự động trong cùng sprint mà nó được ban hành.** Cơ chế: một MUST không
  có cơ chế thực thi sẽ có tỉ lệ tuân thủ giảm dần, và khi nó bị vi phạm mà không có hậu quả, nó làm
  giảm hiệu lực của mọi MUST khác.
- **Giới hạn số MUST ở mức bạn thực thi được hết bằng công cụ — thực tế là 10–15.** Cơ chế: hiệu lực của
  một hệ quy tắc bằng hiệu lực của quy tắc yếu nhất, vì nó xác lập kỳ vọng về mức độ nghiêm túc.
- **Viết standard bằng cách mô tả cách làm hiện tại tốt nhất trong tổ chức, không bằng cách mô tả lý
  tưởng.** Cơ chế: standard mô tả một cách làm mà không service nào đang theo thì không có tham chiếu
  nào để học, và nó bắt đầu với 100% vi phạm.
- **Mỗi quy tắc kèm một câu "vì sao" và một ví dụ về failure mode nó ngăn.** Cơ chế: engineer tuân thủ
  quy tắc họ hiểu và lách quy tắc họ không hiểu; và câu "vì sao" là thứ cho phép họ xử lý đúng các
  trường hợp mà quy tắc không lường tới.
- **Cơ chế exception phải rẻ hơn việc lách.** Cơ chế: nếu xin ngoại lệ mất 2 tuần và lách mất 0 phút,
  bạn sẽ không biết trạng thái thật của hệ thống mình. Mục tiêu thực tế: xin ngoại lệ trong dưới 2 ngày.
- **Mọi exception có ngày hết hạn và một người theo dõi hết hạn khác với người xin.** Cơ chế: người xin
  không có động cơ nhắc lại; nếu không có người theo dõi độc lập, ngoại lệ trở thành vĩnh viễn.
- **Đo adoption của golden path và coi đó là chỉ số của nhóm Platform, không phải của các team sản
  phẩm.** Cơ chế: nó đặt trách nhiệm ở phía có khả năng thay đổi sản phẩm, và nó biến quan hệ từ cưỡng
  chế thành phục vụ.
- **Xét lại danh sách MUST mỗi 6 tháng và chủ động bỏ những cái đã hết lý do.** Cơ chế: quy tắc tích tụ
  và không ai có động cơ bỏ; một hệ standard chỉ tăng sẽ đạt tới mức mà tuân thủ trọn vẹn là bất khả.

### Anti-patterns

| Anti-pattern | Cơ chế phá hệ thống | Dấu hiệu sớm |
|---|---|---|
| **Standard viết trên wiki không ai đọc** | Không nằm trên đường đi của engineer nên không được đọc; tổ chức tin mình có chuẩn trong khi thực tế mỗi người làm một kiểu → ảo giác kiểm soát | Ngày cập nhật cuối > 6 tháng; số lượt xem 30 ngày < số engineer; người mới không biết trang đó tồn tại |
| **Standard áp bằng quy trình duyệt thay vì bằng công cụ** | Con đường tuân thủ dài hơn con đường lách → tỉ lệ lách tăng theo áp lực thời gian; hàng đợi phê duyệt tạo bottleneck | Thời gian chờ phê duyệt > 3 ngày; có trường hợp "làm rồi mới xin"; số yêu cầu bị rút vì không kịp |
| **Cấm mọi công nghệ mới nên đội ngũ ngừng học** | Không có dòng công nghệ vào tổ chức → hệ thống lạc hậu dần; và engineer giỏi rời đi vì mất cơ hội học, đây là chi phí nhân sự không hiện trên báo cáo nào | 18 tháng không có công nghệ mới nào vào; trong exit interview có lý do về stack; engineer học công nghệ mới ngoài giờ và không dùng được ở công ty |
| **40 quy tắc đều ở mức MUST** | Không thực thi được hết → có MUST bị vi phạm không hậu quả → toàn bộ hệ quy tắc mất hiệu lực; và engineer không phân biệt được cái gì thật sự quan trọng | Đếm số MUST; đếm số MUST có kiểm tra tự động. Tỉ lệ dưới 50% là báo động |
| **Chuẩn hoá cả những gì bên trong ranh giới team** | Giảm tối ưu cục bộ mà không đổi lại được lợi ích phối hợp nào; tạo cảm giác bị kiểm soát và làm mất ownership | Standard quy định cấu trúc thư mục, tên biến, framework test; các team phàn nàn về "vi quản lý kỹ thuật" |
| **Golden path không có owner** | Phân rã trong 2–3 quý: template lỗi thời, pipeline không cập nhật, và người dùng quay lại tự làm | Không trả lời được "ai sở hữu template này"; issue trên repo template không được xử lý |
| **Ngoại lệ không có thời hạn** | Ngoại lệ trở thành tiêu chuẩn thứ hai; sau vài quý có hai standard song song và không ai biết cái nào áp dụng ở đâu | Có exception được cấp hơn 2 quý trước, không ai theo dõi; có nhiều exception cho cùng một quy tắc |

### Khi nào KHÔNG nên áp dụng

**Khi chỉ có một team và một hệ thống.** Với 6–8 người ngồi cùng nhau, chuẩn hoá bằng văn bản là chi phí
thuần: mọi người đã biết cách làm, và khi có gì đổi thì một câu nói là đủ. Thứ duy nhất đáng làm ở quy
mô này là formatter và linter với cấu hình mặc định. Ngưỡng để bắt đầu nghĩ tới standard viết ra: có
team thứ hai, hoặc có người mới vào mà không ai có thời gian kèm.

**Khi bạn chưa có golden path.** Ban hành MUST trước khi có con đường dễ đi để tuân thủ MUST đó là cách
tạo ra một tổ chức vừa không tuân thủ vừa oán giận. Thứ tự bắt buộc: làm con đường đúng dễ đi trước
(paved road), rồi mới dựng rào chắn (guard rail). Nếu bạn không có capacity để làm golden path, hãy hạ
tất cả xuống mức SHOULD và trung thực với chính mình về mức đảm bảo thật.

**Với các đội thực nghiệm, R&D, hoặc team đang khám phá một hướng sản phẩm mới.** Mục tiêu của các đội
này là học nhanh và tạo ra thông tin, không phải sản xuất hệ thống bền. Áp toàn bộ standard vào đây làm
chậm việc học mà không giảm được rủi ro nào có ý nghĩa — vì phần lớn code sẽ bị bỏ. Điều kiện biên phải
rõ và phải được viết ra: các service của đội này không được nằm trên đường doanh thu, không được xử lý
dữ liệu người dùng thật ở mức có rủi ro pháp lý, và có một cửa rõ ràng (một checklist) để đi từ trạng
thái thực nghiệm sang trạng thái production.

**Trong bối cảnh ODC nơi khách hàng đã có standard riêng.** Áp thêm một lớp standard nội bộ lên trên
standard của khách tạo ra hai bộ quy tắc xung đột và engineer phải chọn — họ sẽ chọn bộ của khách vì đó
là bộ có hậu quả hợp đồng. Cách đúng: nhận standard của khách làm MUST, và chỉ thêm chuẩn nội bộ ở
những chỗ khách không quy định — thường là cách team test nội bộ, cách dựng môi trường local, cách viết
tài liệu chuyển giao, cách chuẩn bị cho việc rotate người. Đó là không gian thật của bạn, và nó đủ rộng.

**Khi vấn đề thật là năng lực hoặc thiếu người vận hành, không phải thiếu chuẩn.** Nếu hệ thống lộn xộn
vì team có 8 người mà 6 người dưới 2 năm kinh nghiệm và không có ai mentoring, thì một bộ standard sẽ
không được hiểu, không được áp dụng đúng, và sẽ tạo cảm giác thất bại thường trực. Việc phải làm trước
là tuyển hoặc điều một người có kinh nghiệm vào, thiết lập pairing và review có chất lượng (chủ đề 3),
rồi chuẩn sẽ tự nổi lên từ cách làm chung — và lúc đó việc viết nó ra là ghi lại một thực tế, không
phải áp một lý tưởng.

---

## 7. Engineering Culture — thiết kế thay vì hô hào

### Problem Statement

Một công ty fintech, 45 engineer. Trên tường phòng làm việc có ba giá trị: "Chất lượng là ưu tiên số
một", "Chúng tôi học từ thất bại", "Minh bạch". Bảng dưới đây so sánh tuyên bố với các hiện tượng đếm
được trong 12 tháng (số minh hoạ, cách đo thì bạn làm được trong hai buổi chiều):

| Tuyên bố | Hiện tượng đếm được | Kết luận thật |
|---|---|---|
| "Chất lượng là ưu tiên số một" | 3 người được promote lên Senior trong năm; cả ba là người có số tính năng ship nhiều nhất. Người viết bộ test tích hợp làm change failure rate giảm từ 19% xuống 8% không được nhắc trong bất kỳ buổi review nào | Tốc độ ship là ưu tiên số một |
| "Chúng tôi học từ thất bại" | 8 postmortem trong năm; 5 trong đó có mục "nguyên nhân: lỗi con người, đã nhắc nhở"; 2 người liên quan tới các incident lớn nhất đã rời công ty trong 6 tháng sau đó | Thất bại là thứ phải tìm người chịu |
| "Minh bạch" | Quyết định về việc dừng một sản phẩm được biết qua tin hành lang 3 tuần trước khi công bố; 4 engineer nói trong 1-1 rằng họ "không dám hỏi về roadmap" | Thông tin là tài nguyên của cấp trên |

Đây không phải sự đạo đức giả. Trong hầu hết trường hợp, ban lãnh đạo tin vào các giá trị đã viết. Vấn
đề là **các giá trị viết trên tường không có cơ chế thực thi, trong khi hệ thống khuyến khích thì
có** — và khi hai thứ này lệch nhau, hệ thống khuyến khích thắng, mỗi lần, không có ngoại lệ.

Hậu quả có thể truy được: engineer học được đâu là hành vi thật sự được thưởng trong khoảng 3–6 tháng
đầu (nhanh hơn bất kỳ chương trình onboarding nào), rồi họ tối ưu theo đó. Những người không muốn tối
ưu theo đó — thường là những người coi trọng chất lượng thật — sẽ đi. Sau 18–24 tháng, tổ chức có một
đội ngũ được chọn lọc theo hệ khuyến khích thật, và lúc đó việc thay đổi văn hoá không còn là đổi cơ chế
mà là đổi người.

### First Principles

**Cơ chế một: culture là tập hợp hành vi được thưởng và bị phạt trên thực tế.** Định nghĩa này có ba
tính chất làm nó dùng được. Nó **quan sát được** (bạn đếm được ai được promote, cái gì bị bỏ khi gấp).
Nó **thay đổi được** (bạn đổi được cơ chế thưởng/phạt; bạn không đổi được "niềm tin" của 45 người). Nó
**dự đoán được** (biết cái gì được thưởng, bạn dự đoán được hành vi trong 6 tháng tới). Ngược lại, định
nghĩa culture là "tập hợp giá trị và niềm tin chung" không có tính chất nào trong ba tính chất đó, nên
nó không dùng được cho việc quản trị.

**Cơ chế hai: nếu tuyên bố và hệ thống khuyến khích lệch nhau thì hệ thống khuyến khích thắng — và
tuyên bố còn gây hại thêm.** Phần thứ hai của câu này quan trọng và ít được nói. Khi một tổ chức tuyên
bố "chất lượng là số một" nhưng thưởng tốc độ, nó không chỉ có văn hoá tốc độ — nó còn có thêm một hiệu
ứng: engineer học được rằng **lời của lãnh đạo không phải dữ liệu đáng tin**. Từ đó, mọi thông điệp sau
đó, kể cả những thông điệp thật, đều bị chiết khấu. Đây là lý do vì sao "không nói gì" đôi khi tốt hơn
"nói một giá trị mình không thực thi".

**Cơ chế ba: tín hiệu mạnh nhất về culture không phải cái được nói, mà là cái được làm khi có xung đột
giữa hai ưu tiên.** Trong điều kiện bình thường, mọi tổ chức đều có thể vừa nhanh vừa chất lượng. Culture
chỉ lộ ra ở điểm xung đột: tuần cuối trước deadline, cái gì bị bỏ? Khi một người phát hiện rủi ro vào
ngày ship, chuyện gì xảy ra với người đó? Khi có incident, câu hỏi đầu tiên trong phòng là "hệ thống hỏng
ở đâu" hay "ai làm"? **Mỗi lần một xung đột như vậy được giải quyết, tổ chức học một quy tắc — và nó học
từ hành động, không từ lời giải thích kèm theo hành động.**

**Cơ chế bốn: culture lan qua kể chuyện, và câu chuyện được kể là câu chuyện có kết cục rõ ràng.** Con
người lưu trữ chuẩn mực xã hội dưới dạng chuyện kể chứ không dưới dạng danh sách quy tắc. Trong một tổ
chức engineering, các câu chuyện được kể lại nhiều nhất thường thuộc ba loại: ai đó cứu một sự cố lớn
trong đêm (tạo hero culture); ai đó bị đổ lỗi cho một sự cố (tạo sợ hãi); ai đó nói không với một yêu
cầu và không có hậu quả xấu (tạo Psychological Safety). Suy ra một đòn điều khiển rẻ và mạnh: **chọn
câu chuyện nào được kể lại.** Nếu bạn muốn văn hoá coi trọng phòng ngừa, hãy kể lại lần một engineer
dừng một release vào phút cuối và điều đó là quyết định đúng — kể ở retrospective, ở all-hands, trong
1-1 với người mới.

### Mental Model

**Đọc culture hiện tại qua sáu tín hiệu quan sát được.** Đây là công cụ chẩn đoán, làm được trong một
tuần, và nó cho bức tranh chính xác hơn mọi bản khảo sát:

| # | Tín hiệu | Cách đo cụ thể | Nó tiết lộ điều gì |
|---|---|---|---|
| 1 | **Ai được promote** | Lấy 5 lần promotion gần nhất, viết ra lý do thật (không phải lý do trên giấy) | Định nghĩa thật của "giỏi" trong tổ chức này. Đây là tín hiệu mạnh nhất trong sáu tín hiệu |
| 2 | **Chuyện gì được kể lại** | Hỏi 5 người: "chuyện đáng nhớ nhất về team này mà anh/chị hay kể cho người mới là gì?" | Chuẩn mực nào đang được truyền; và loại anh hùng nào được tôn vinh |
| 3 | **Phản ứng khi có incident** | Đọc 5 postmortem gần nhất: đếm số lần xuất hiện tên người so với số lần xuất hiện tên cơ chế/hệ thống | Blameless là thật hay là từ ngữ. Nếu "root cause" là tên một người, tổ chức đang tối ưu cho việc tìm người chịu |
| 4 | **PR bị reject vì lý do gì** | Lấy 20 PR có nhiều tranh luận nhất: phân loại lý do (bug / thiết kế / style / sở thích reviewer) | Tiêu chuẩn thật về chất lượng, và mức độ quyền lực cá nhân trong quy trình kỹ thuật |
| 5 | **Ai dám nói không** | Trong 3 tháng qua, ai đã từ chối một yêu cầu và không có hậu quả xấu? Ai đã từ chối và có hậu quả? | Psychological Safety, và nó phân bố không đều: thường chỉ vài người có "quyền" nói không |
| 6 | **Cái gì bị bỏ khi gấp** | Tuần cuối trước một release quan trọng gần nhất: cái gì bị cắt đầu tiên? | Thứ tự ưu tiên thật. Nếu test và code review bị bỏ đầu tiên, chúng là thứ hạng cuối bất kể tuyên bố |

Cách dùng bảng này: làm cho tổ chức của bạn, viết câu trả lời ra giấy, rồi đặt cạnh danh sách giá trị
tuyên bố. Khoảng cách giữa hai cột là chương trình làm việc của bạn trong hai quý tới — và nó cụ thể
hơn bất kỳ initiative văn hoá nào.

**Culture = f(cơ chế), không f(thông báo).** Ba loại cơ chế có đòn bẩy cao nhất, xếp theo thứ tự sức
mạnh:

| Cơ chế | Nó điều khiển hành vi nào | Ai đổi được | Chi phí đổi |
|---|---|---|---|
| **Định nghĩa promotion và career ladder** | Cái gì được coi là "giỏi"; định hướng dài hạn của hành vi | EM + Head of Engineering + HR | Cao về mặt phối hợp, gần bằng 0 về tiền |
| **Definition of done** | Hành vi hàng ngày ở tầng Execution | Tech Lead + team, không cần ai phê duyệt | Rất thấp — đây là đòn dễ nhất và bị dùng ít nhất |
| **Cách chạy postmortem** | Hành vi khi có sự cố; và mức độ người ta báo cáo vấn đề sớm | Tech Lead hoặc EM chủ trì | Thấp; chủ yếu là kỷ luật của người chủ trì |

Ba cơ chế phụ nhưng đáng dùng: cái gì được nói trong all-hands (chọn câu chuyện), ai được mời vào phòng
ra quyết định kỹ thuật (tín hiệu về ai được coi là có thẩm quyền), và cách xử lý người nói ra tin xấu
(quyết định liệu bạn còn nhận được tin xấu nữa hay không).

### Practical Framework

**Quy trình thay đổi một mảnh culture — 6 bước, một quý.** Không làm "chương trình văn hoá"; chọn một
hành vi cụ thể và đổi cơ chế sinh ra nó.

| Bước | Việc làm | Output | Ví dụ (muốn team coi trọng chất lượng) |
|---|---|---|---|
| 1 | Viết hành vi mong muốn ở dạng quan sát được, không phải giá trị | Một câu có động từ và có thể đếm | "Khi một engineer thấy rủi ro trước release, họ nói ra và release được hoãn nếu rủi ro thật" (không phải "team coi trọng chất lượng") |
| 2 | Chẩn đoán bằng 6 tín hiệu: hành vi hiện tại là gì, cơ chế nào đang sinh ra nó | Bảng 6 tín hiệu đã điền | Tín hiệu 1: promote theo số tính năng. Tín hiệu 6: test bị bỏ đầu tiên. Tín hiệu 5: chỉ Quân dám nói không |
| 3 | Tìm cơ chế mâu thuẫn và bỏ nó trước khi thêm cơ chế mới | Danh sách cơ chế phải bỏ | Bỏ việc công bố bảng xếp hạng số story point mỗi sprint |
| 4 | Đổi một cơ chế có đòn bẩy cao, chỉ một | Một thay đổi cụ thể, có ngày hiệu lực | Đưa vào definition of done: "có test cho luồng lỗi ở các module trong debt register"; và đưa vào tiêu chí Senior: "đã cải thiện được một chỉ số chất lượng đo được của hệ thống" |
| 5 | Làm mẫu và kể lại, tối thiểu 3 lần trong quý | 3 câu chuyện được kể công khai | Lần Nam hoãn release 1 ngày vì thấy thiếu rollback plan: kể ở retro, và Linh nhắc lại ở all-hands, nói rõ "đây là quyết định đúng" |
| 6 | Đo lại tín hiệu sau một quý, và công bố | Bảng so sánh trước/sau | Số lần release bị hoãn vì rủi ro kỹ thuật; change failure rate; số người khác Quân đã nói không |

Bước 3 là bước bị bỏ nhiều nhất và là nguyên nhân chính của thất bại. Thêm một cơ chế mới trong khi cơ
chế cũ vẫn thưởng hành vi cũ chỉ tạo ra một thông điệp kép, và engineer sẽ theo cơ chế nào có hậu quả
rõ hơn — thường là cơ chế cũ, vì nó đã được kiểm chứng.

**Bốn cơ chế cụ thể và cách sửa từng cái:**

| Cơ chế | Dạng phổ biến tạo ra hành vi sai | Dạng sửa | Cơ chế phía sau |
|---|---|---|---|
| Definition of done | "Code xong, QA pass" | Thêm: có test cho luồng lỗi; có log/metric đủ để debug trên production; có mục rollback; ADR nếu là quyết định mức 3 | Đây là nơi hành vi hàng ngày được định nghĩa, và nó đổi được không cần ai phê duyệt |
| Tiêu chí promotion | "Ship được nhiều, giải quyết được vấn đề khó" | Thêm một tiêu chí bắt buộc ở mức Senior trở lên: "đã tạo ra một cải thiện đo được cho hệ thống hoặc cho người khác" — có thể là giảm lead time, giảm failure rate, tăng số người hiểu một module | Định nghĩa "giỏi" là tín hiệu văn hoá mạnh nhất; đổi nó là đòn bẩy dài nhất |
| Postmortem | Mục "root cause: lỗi con người, đã nhắc nhở" | Cấm tên người trong mục nguyên nhân; buộc mọi nguyên nhân phải ở dạng "cơ chế nào cho phép việc này xảy ra"; và mọi action item có tên chủ và ngày | Nếu báo cáo vấn đề dẫn tới hậu quả cho người báo, tổ chức mất tín hiệu sớm — mất mát lớn nhất có thể có |
| Cách ghi nhận công khai | Chỉ nhắc người ship tính năng và người cứu sự cố | Thêm việc nhắc: comment review bắt được failure mode; người viết tài liệu làm giảm thời gian onboarding; người từ chối một yêu cầu vô lý có lý do đúng | Hành vi được nhắc công khai là hành vi được lặp lại; và những hành vi phòng ngừa vốn không có "khoảnh khắc anh hùng" nên cần được nhắc chủ đích |

### Trade-off

**Trade-off 1 — Culture mạnh, nhất quán vs đa dạng cách làm.** Culture mạnh cho phối hợp nhanh, tiêu
chuẩn rõ, tuyển và sa thải dễ vì tiêu chí rõ. Chi phí: nó lọc bớt sự đa dạng, và một culture mạnh sai
hướng thì tự sửa rất chậm — vì cơ chế đã lọc ra những người không thấy vấn đề. Điều kiện nghiêng về
culture mạnh: giai đoạn cần phối hợp cao và ra quyết định nhanh (scale-up, khủng hoảng). Nghiêng về đa
dạng: giai đoạn cần khám phá hướng mới, hoặc khi tổ chức vừa nhận ra mình đang sai hướng.

**Trade-off 2 — Chất lượng vs tốc độ ở tầng cơ chế.** Nói rõ để không rơi vào khẩu hiệu: một tổ chức
không thể tối đa hoá cả hai, và việc chọn tốc độ ở một giai đoạn là hợp lý. Cái không hợp lý là **chọn
tốc độ trong cơ chế nhưng tuyên bố chất lượng trong ngôn từ** — vì nó vừa không có chất lượng, vừa mất
lòng tin. Nếu ưu tiên thật là tốc độ, cách trung thực là nói: "trong hai quý tới ưu tiên là tốc độ; đây
là danh sách những gì chúng ta chấp nhận nợ, đây là mức lãi, và đây là điều kiện chúng ta sẽ trả." Điều
này giữ được cả tốc độ lẫn lòng tin, và nó biến một sự thoả hiệp thành một quyết định.

**Trade-off 3 — Ghi nhận cá nhân vs ghi nhận nhóm.** Ghi nhận cá nhân tạo động lực mạnh và rõ, nhưng
trong công việc kỹ thuật nơi phần lớn kết quả là sản phẩm của phối hợp, nó tạo ra tối ưu cục bộ: người
ta chọn việc dễ thấy thay vì việc đắt giá. Ghi nhận nhóm giảm hiệu ứng đó nhưng làm mờ đóng góp và làm
người xuất sắc thấy bất công. Điều kiện nghiêng: ghi nhận cá nhân cho các hành vi **có tính lan** (viết
tài liệu, mentoring, cải thiện golden path, review chất lượng cao) và ghi nhận nhóm cho kết quả giao
hàng. Cách đặt này đảo ngược trực giác thông thường, và nó có lý do: kết quả giao hàng vốn đã dễ thấy,
còn hành vi lan tỏa thì không có ai tự nhiên nhắc tới.

### Real-world Scenarios

**Tình huống A — "Chúng tôi coi trọng chất lượng" nhưng promote người ship nhanh nhất.** Bối cảnh: công
ty fintech ở đầu chủ đề. Minh (Tech Lead) mang ba dữ kiện tới Linh (EM): change failure rate 19%; 5/8
postmortem có "nguyên nhân là lỗi con người"; và ba lần promotion gần nhất đều là người ship nhiều nhất,
trong đó một người có change failure rate cá nhân cao nhất team.

Phân tích cơ chế, ba lớp:

*Lớp 1 — hệ khuyến khích.* Promotion là tín hiệu có giá trị cao nhất trong tổ chức (tiền, địa vị, và sự
xác nhận công khai về định nghĩa "giỏi"). Khi tín hiệu đó gắn với số tính năng ship, mọi engineer hợp lý
sẽ tối đa hoá số tính năng ship, và họ sẽ làm điều đó bằng cách cắt những thứ không bị đo: test luồng
lỗi, log đủ để debug, tài liệu, review kỹ. Không ai vi phạm quy định nào.

*Lớp 2 — vòng phản hồi.* Change failure rate cao làm tăng số incident; incident làm tăng công việc gián
đoạn; công việc gián đoạn làm giảm capacity cho tính năng; giảm capacity làm tăng áp lực ship; áp lực
tăng làm cắt góc nhiều hơn. Vòng phản hồi dương, không tự dừng.

*Lớp 3 — chọn lọc.* Sau 18 tháng, những người coi trọng chất lượng đã hoặc rời đi, hoặc học cách không
nêu vấn đề. Đây là lớp đắt nhất, vì nó không đảo được bằng cách đổi cơ chế — bạn phải tuyển lại.

Cách sửa, ba việc, theo thứ tự và trong hai quý:

1. **Đổi definition of done (tuần 1, không cần ai phê duyệt).** Thêm ba điều kiện: test cho luồng lỗi ở
   các module trong debt register; đủ log/metric để debug trên production mà không cần thêm code; một
   dòng về rollback. Đây là việc Minh làm được ngay, và nó tác động vào hành vi hàng ngày.
2. **Đổi tiêu chí promotion (quý 1, cần Linh và HR).** Thêm một tiêu chí bắt buộc ở mức Senior: "đã tạo
   ra một cải thiện đo được cho hệ thống hoặc cho người khác", với ví dụ cụ thể về những gì tính. Và
   quan trọng không kém: **lần promotion tiếp theo phải là một người thoả tiêu chí này**, vì lời nói về
   tiêu chí mới chỉ có hiệu lực khi có một trường hợp thật.
3. **Đổi cách chạy postmortem (tuần 2).** Cấm tên người trong mục nguyên nhân; buộc câu hỏi "cơ chế nào
   cho phép việc này xảy ra". Chi tiết quan trọng: Linh phải là người đầu tiên áp dụng nó cho một
   incident mà **cô ấy** có phần trách nhiệm — vì cách nhanh nhất để chứng minh Blameless là thật là để
   người có quyền lực nhất trong phòng nhận phần của mình trước.

Điều Minh cố tình **không** làm: không tổ chức một buổi họp về "văn hoá chất lượng", không viết một tài
liệu về giá trị, không gửi email nhắc nhở. Lý do: các phương tiện đó tác động vào tuyên bố, và tuyên bố
là bên đang thua trong cơ chế hai ở phần First Principles.

Kết quả sau hai quý (số minh hoạ): change failure rate từ 19% xuống 11%; số postmortem có "nguyên nhân
là lỗi con người" về 0; một người được promote với lý do chính là xây bộ test tích hợp — và đó là câu
chuyện được kể lại nhiều nhất trong quý đó, có tác dụng hơn mọi thông báo.

**Tình huống B — Culture bị định hình bởi metric của khách (ODC), và ba góc nhìn.** Bối cảnh: ODC 60
người, khách Nhật. Khách đo hai chỉ số: utilization rate (mục tiêu ≥ 90% billable) và số ticket đóng
mỗi sprint. Hợp đồng tính theo giờ.

Cơ chế hỏng rất trực tiếp: nếu utilization phải ≥ 90% billable, thì mọi việc không billable — viết tài
liệu, cải thiện môi trường test, review kỹ, mentoring người mới, viết tooling — đều là chi phí mà đội
phải tự chịu. Và nếu ticket đóng là đơn vị đo, thì một ticket "sửa hạ tầng test" không có giá trị đo
được, còn một ticket "sửa bug UI" thì có. Hệ quả sau hai năm: đội có utilization 92%, số ticket cao, và
đồng thời có 5 tuần ramp-up cho mỗi người mới rotate vào, môi trường test hỏng hai lần mỗi tháng, và
một người duy nhất hiểu luồng tích hợp.

*Nhìn từ IC (Nam):* "Tôi biết môi trường test hỏng làm mất thời gian, nhưng tôi không log giờ vào việc
sửa nó được — nó không có ticket. Nên tôi sửa tạm mỗi lần, mất 30 phút, và log vào ticket đang làm. Tôi
làm thế 20 lần trong quý."

*Nhìn từ Tech Lead (Minh):* "Tôi thấy được tổng: 20 lần × 30 phút × 6 người là khoảng 60 giờ mỗi quý,
tất cả đều đang được tính là billable nhưng không tạo giá trị nào cho khách. Nếu tôi bỏ 24 giờ để sửa
dứt điểm, tôi tiết kiệm 60 giờ mỗi quý. Vấn đề là 24 giờ đó phải nằm ở đâu đó trong hệ thống ghi giờ.
Đây không phải bài toán kỹ thuật, đây là bài toán trình bày."

*Nhìn từ Manager (Linh):* "Tôi không đổi được metric của khách — nó nằm trong hợp đồng và nó là cách
họ kiểm soát nhà cung cấp. Nhưng tôi đổi được ba thứ. Một, tôi tạo ra một hạng mục hợp lệ trong hệ ghi
giờ tên là 'productivity improvement', giới hạn 8% và có báo cáo kết quả bằng số cho khách mỗi quý —
khách Nhật chấp nhận điều này khi được trình bày bằng dữ liệu về giảm defect, vì đó là chỉ số họ cũng
quan tâm. Hai, tôi đưa thời gian ramp-up của người rotate vào thành một chỉ số trong báo cáo hàng
tháng, vì khách trả tiền cho thời gian đó nên nó là ngôn ngữ họ hiểu. Ba, ở tầng nội bộ, tôi ghi nhận
trong performance review những việc không billable nhưng có giá trị — vì đó là phần tôi hoàn toàn kiểm
soát."

Điều đáng học từ tình huống này: **một lead vẫn tạo được không gian chất lượng trong một hệ khuyến khích
mình không kiểm soát, bằng cách (a) dịch giá trị kỹ thuật sang chỉ số mà bên trả tiền đã quan tâm, (b)
tạo một hạng mục hợp lệ có giới hạn thay vì xin ngoại lệ từng lần, và (c) dùng đầy đủ các cơ chế nội bộ
mà mình kiểm soát — ghi nhận, definition of done, cách chạy postmortem, cách chọn câu chuyện để kể.**
Không gian đó nhỏ hơn ở một product company, nhưng nó khác không, và phần lớn lead ODC dùng chưa hết.

### Best Practices

- **Bắt đầu bằng chẩn đoán 6 tín hiệu, không bằng tuyên bố giá trị.** Cơ chế: bạn không đổi được thứ bạn
  chưa mô tả được ở dạng quan sát được; và bảng 6 tín hiệu thường lộ ra rằng vấn đề khác với điều bạn
  tưởng.
- **Đổi một cơ chế mỗi lần, và bỏ cơ chế mâu thuẫn trước khi thêm cơ chế mới.** Cơ chế: thông điệp kép
  làm engineer chọn cơ chế có hậu quả rõ hơn, và cơ chế cũ luôn rõ hơn vì đã được kiểm chứng.
- **Dùng definition of done như đòn bẩy đầu tiên.** Cơ chế: nó có chi phí thay đổi thấp nhất trong ba
  cơ chế chính, không cần ai phê duyệt, và nó tác động vào hành vi hàng ngày thay vì hành vi hàng năm.
- **Với mỗi giá trị bạn muốn có, tìm một câu chuyện thật trong tổ chức và kể lại ba lần trong quý.** Cơ
  chế: chuẩn mực lan qua chuyện kể; và một câu chuyện thật từ chính tổ chức có sức thuyết phục cao hơn
  bất kỳ ví dụ từ Big Tech.
- **Người có quyền lực nhất phải là người đầu tiên chịu áp dụng quy tắc mới cho chính mình.** Cơ chế:
  nó là bằng chứng duy nhất mà tổ chức chấp nhận rằng quy tắc là thật. Blameless postmortem đầu tiên
  nên là về một incident mà lead có phần.
- **Đo hành vi, không đo cảm nhận.** Cơ chế: khảo sát culture đo được cảm nhận và bị ảnh hưởng bởi tâm
  lý gần đây; số lần release bị hoãn vì rủi ro kỹ thuật, tỉ lệ postmortem có nguyên nhân là cơ chế, phân
  bố reviewer — đều là hành vi và đều đếm được.
- **Khi ưu tiên thật là tốc độ, hãy nói ra là tốc độ.** Cơ chế: chi phí của một tuyên bố không được thực
  thi cao hơn chi phí của việc thừa nhận một trade-off — vì nó phá giá trị của mọi tuyên bố sau đó.
- **Bảo vệ người nói ra tin xấu, công khai, ngay lần đầu tiên.** Cơ chế: tổ chức quyết định trong 1–2
  trường hợp đầu tiên xem nói ra tin xấu có an toàn không, và quyết định đó rất khó đảo sau đó.

### Anti-patterns

| Anti-pattern | Cơ chế phá hệ thống | Dấu hiệu sớm |
|---|---|---|
| **Culture deck / giá trị trên tường không có cơ chế** | Không đổi hành vi vì không đổi thưởng/phạt; và tạo thêm khoảng cách giữa lời nói và thực tế, làm mọi thông điệp sau bị chiết khấu | Hỏi 5 engineer "giá trị số hai của công ty là gì" — nếu không ai nhớ, tài liệu đó không tồn tại về mặt chức năng |
| **Hackathon để chữa vấn đề cấu trúc** | Hackathon tạo năng lượng ngắn hạn và một vài prototype; nó không đổi definition of done, không đổi promotion, không đổi cách chạy postmortem. Sau 3 tuần mọi thứ trở lại, và tổ chức học rằng "sáng kiến" là hoạt động theo mùa | Sau hackathon không có cơ chế nào đổi; các prototype không có owner; cùng vấn đề xuất hiện ở hackathon năm sau |
| **Hero culture được thưởng công khai** | Thưởng người cứu sự cố lúc 3 giờ sáng tạo động cơ để hệ thống tiếp tục có sự cố lúc 3 giờ sáng; đồng thời việc phòng ngừa — vốn không có khoảnh khắc anh hùng — trở thành vô hình; và nó dẫn tới burnout của đúng những người giỏi nhất | Trong all-hands, các câu chuyện được nhắc đều là chuyện cứu sự cố; một người xuất hiện trong > 40% incident với vai người cứu |
| **Blameless trên giấy, tìm người trên thực tế** | Người ta ngừng báo cáo vấn đề sớm; tổ chức mất tín hiệu sớm — mất mát lớn nhất có thể có, vì mọi hệ thống an toàn đều dựa vào việc vấn đề được nêu khi còn nhỏ | Postmortem có "root cause: lỗi con người"; sau incident có người bị chuyển việc hoặc bị nhắc trong review |
| **Đo culture bằng khảo sát rồi không làm gì** | Khảo sát tạo kỳ vọng; không hành động sau khảo sát dạy tổ chức rằng việc nói ra là vô ích, và tỉ lệ trả lời khảo sát sau sẽ giảm — kèm theo chất lượng câu trả lời | Tỉ lệ trả lời khảo sát giảm qua các kỳ; không có action item nào từ kỳ trước được nhắc lại |
| **Tuyển "culture fit" mà thực chất là "giống chúng ta"** | Lọc mất sự đa dạng về cách tiếp cận, làm tổ chức mất khả năng tự phát hiện mình sai hướng | Trong phỏng vấn, tiêu chí "culture fit" không có định nghĩa hành vi cụ thể; đội ngũ đồng nhất về xuất phát điểm |
| **Lead nói một đằng, hành xử một nẻo ở điểm xung đột** | Tín hiệu từ hành động của lead ở điểm xung đột mạnh hơn hàng chục lần lời nói; một lần lead bỏ test khi gấp xoá bỏ ba tháng nói về chất lượng | Tuần cuối trước release, chính lead là người đề nghị bỏ bước review hoặc test |

### Khi nào KHÔNG nên áp dụng

**Khi bạn không kiểm soát bất kỳ cơ chế nào.** Nếu bạn là Tech Lead không tham gia quyết định promotion,
không đổi được definition of done (khách quyết), và không chủ trì postmortem, thì một chương trình thay
đổi culture sẽ thất bại và đốt uy tín của bạn. Việc đúng: xác định chính xác cơ chế nào bạn kiểm soát —
thường vẫn còn ít nhất ba thứ: bạn ghi nhận ai và ghi nhận vì việc gì trong team mình, bạn phản ứng thế
nào khi có người báo tin xấu, và câu chuyện nào bạn kể cho người mới. Bắt đầu từ đó, và mở rộng khi có
kết quả để trình bày.

**Trong 6–8 tuần sau một sự kiện gây bất an lớn** — cắt giảm nhân sự, một người quan trọng rời đi, một
sự cố nghiêm trọng ra tới khách hàng. Trong giai đoạn này, mọi thông điệp về giá trị và văn hoá đều bị
đọc qua lăng kính của sự kiện đó, và thường bị đọc là mị dân. Việc đúng trong giai đoạn này là những
hành động cụ thể có kết quả thấy được trong 1–2 tuần, và trả lời thẳng các câu hỏi thực tế mà mọi người
đang có.

**Khi vấn đề thật là vấn đề về người cụ thể.** Nếu một người trong team liên tục có hành vi phá vỡ hợp
tác — hạ thấp người khác trong review, giữ thông tin làm quyền lực, không nhận feedback — thì đó là
việc cần xử lý ở tầng cá nhân (chương 03 và 08), không phải bằng một chương trình văn hoá. Dùng
initiative văn hoá để né một cuộc trò chuyện khó với một người là mẫu hành vi rất phổ biến, nó không
giải quyết vấn đề, và nó gửi cho cả team một tín hiệu tệ hơn: rằng hành vi đó được chấp nhận.

**Khi bạn định đo và công bố culture bằng khảo sát mà chưa có ý định hành động.** Chi phí của một khảo
sát không có hành động theo sau không phải bằng không — nó âm. Nếu bạn chưa có capacity hoặc chưa có
quyền để thay đổi gì trong quý này, đừng khảo sát; hãy làm một việc nhỏ mà bạn kiểm soát được, rồi
khảo sát sau khi đã có một thay đổi để chỉ ra.

**Khi tổ chức đang chuyển đổi cấu trúc lớn.** Trong lúc ranh giới team đang được vẽ lại, ai báo cáo cho
ai đang đổi, và một số vai chưa có người — cơ chế thưởng/phạt chưa xác định, nên mọi nỗ lực thiết kế
culture đều đang xây trên nền chưa đông. Thứ tự đúng: chốt cấu trúc và các vai accountable trước (chương
09), rồi mới thiết kế cơ chế trên cấu trúc đó.

---

## 8. Giữ technical credibility khi không còn viết code nhiều

### Problem Statement

Minh làm Tech Lead 14 tháng. Anh không còn viết code trên critical path từ tháng thứ tư. Bốn hiện tượng
xuất hiện dần, và mỗi cái là một tín hiệu đo được:

| Hiện tượng | Cách đo | Nó nói gì |
|---|---|---|
| Trong design review, Minh nêu một ý và không ai phản biện, nhưng thiết kế cuối không theo ý đó | Đối chiếu ý kiến trong biên bản với code sau 1 tháng | Uy tín đã chuyển thành sự lịch sự. Đây là tín hiệu sớm nhất và dễ bỏ qua nhất |
| Minh ước lượng một việc là "2 ngày", team làm 9 ngày | So sánh ước lượng của lead với thực tế, 5 lần gần nhất | Mô hình về hệ thống trong đầu Minh đã lệch với hệ thống thật |
| Khi có incident, Minh không hiểu được dashboard mới và phải chờ người khác giải thích | Quan sát trong 2 incident gần nhất | Mất kết nối với tầng vận hành — tầng mà mọi quyết định kiến trúc cuối cùng được kiểm nghiệm |
| Minh quyết một hướng kỹ thuật dựa trên trạng thái hệ thống 8 tháng trước | Kiểm tra: giả định trong quyết định gần nhất có còn đúng không | Quyết định dựa trên dữ liệu đã hết hạn |

Bốn hiện tượng này gộp lại thành một hậu quả cụ thể: **Minh vẫn có thẩm quyền theo chức danh, nhưng
không còn ảnh hưởng theo lập luận.** Team vẫn nghe anh nói vì anh là lead, nhưng khi anh nói xong, họ
làm theo phán đoán của người có dữ liệu hiện trường. Trong bối cảnh Việt Nam, hiện tượng này bị che
thêm một lớp: văn hoá giữ mặt làm không ai nói "anh không còn nắm hệ thống nữa", nên tín hiệu tới rất
muộn — thường là khi một quyết định sai đã có hậu quả.

Cần nói rõ chiều ngược lại để không rơi vào kết luận sai: giải pháp không phải quay lại viết code nhiều.
Một lead viết code trên critical path sẽ trở thành bottleneck, và chi phí đó thường lớn hơn chi phí của
việc mất một phần chi tiết kỹ thuật. Bài toán thật là: **nạp lại uy tín kỹ thuật với chi phí thời gian
thấp nhất và không tạo phụ thuộc vào bản thân.**

### First Principles

**Cơ chế một: uy tín kỹ thuật là điều kiện để influence trong một tổ chức kỹ thuật, và nó khác thẩm
quyền.** Thẩm quyền (authority) đến từ vị trí và cho bạn quyền yêu cầu; uy tín (credibility) đến từ việc
người khác tin phán đoán của bạn và cho bạn quyền được lắng nghe. Trong công việc kỹ thuật, thẩm quyền
đủ để bắt người ta làm, nhưng không đủ để họ làm tốt — vì phần lớn công việc kỹ thuật đòi hỏi hàng trăm
phán đoán nhỏ mà bạn không kiểm soát được. Suy ra: **một lead có thẩm quyền nhưng không có uy tín sẽ
nhận được tuân thủ hình thức**, và tuân thủ hình thức trong công việc kỹ thuật cho ra kết quả tệ hơn cả
việc không yêu cầu gì.

**Cơ chế hai: uy tín kỹ thuật phân rã theo thời gian, và tốc độ phân rã tỉ lệ với tốc độ thay đổi của
hệ thống.** Uy tín không phải một tài sản tích lũy vĩnh viễn; nó là niềm tin rằng **phán đoán hiện tại**
của bạn đáng tin. Niềm tin đó dựa trên một mô hình trong đầu bạn về hệ thống, và hệ thống thay đổi mỗi
tuần: code mới, hạ tầng mới, đối tác mới, tải mới, người mới. Nếu bạn không nạp lại, mô hình trong đầu
bạn lệch dần với thực tế, và có một thời điểm mà phán đoán của bạn trở nên tệ hơn phán đoán của một
engineer mid-level đang làm việc đó hàng ngày. Ước lượng thô từ kinh nghiệm (không phải số liệu nghiên
cứu): với một hệ thống đang phát triển tích cực, khoảng 6 tháng không tiếp xúc là đủ để mô hình lệch tới
mức các ước lượng của bạn sai hệ thống theo hướng lạc quan.

**Cơ chế ba: thứ phân rã không phải kiến thức về công nghệ, mà là kiến thức về hệ thống cụ thể này.**
Đây là phân biệt quan trọng và hay bị lẫn. Kiến thức nền — cách nghĩ về consistency, về hàng đợi, về
failure mode, về trade-off — phân rã rất chậm, và nó là phần lớn giá trị bạn mang lại. Kiến thức về hệ
thống cụ thể — module nào mong manh, đối tác nào trả callback trùng, dashboard nào đáng tin, deploy mất
bao lâu — phân rã nhanh. Suy ra chiến lược nạp lại đúng: **không cần học lại framework mới; cần tiếp xúc
với hiện trường của chính hệ thống mình.** Đây là lý do đọc PR và tham gia postmortem có hiệu quả cao
hơn nhiều so với làm một khoá học công nghệ.

**Cơ chế bốn: uy tín được xây bằng phán đoán được kiểm chứng công khai, không bằng việc kể lại thành
tích cũ.** Cơ chế xã hội: người ta cập nhật niềm tin về phán đoán của bạn dựa trên các lần bạn đưa ra
một dự đoán và nó được kiểm chứng. Một lead nói "tôi nghĩ chỗ này sẽ vỡ khi tải gấp ba, cụ thể ở connection
pool" và sáu tuần sau nó vỡ đúng chỗ đó — uy tín tăng mạnh hơn bất kỳ bài trình bày nào. Ngược lại, dùng
thành tích cũ ("hồi tôi làm hệ thống X...") không tăng uy tín vì nó không kiểm chứng được, và trong tổ
chức Việt Nam nơi thâm niên vốn đã được trọng, nó còn dễ bị đọc là dùng địa vị thay cho lập luận.

### Mental Model

**Uy tín như một tài khoản có lãi âm.** Mỗi tháng không tiếp xúc, số dư giảm. Các khoản nạp có chi phí
và hiệu quả khác nhau; các khoản rút thì thường không nhìn thấy tới khi đã âm:

| Hoạt động | Loại | Chi phí thời gian/tuần | Hiệu quả nạp | Rủi ro |
|---|---|---|---|---|
| Đọc PR (không comment hết, chỉ đọc) | Nạp | 2–3 giờ | Cao: thấy được cả code, cả cách team suy nghĩ | Thấp |
| Tham gia design review với vai người đặt câu hỏi | Nạp | 1–2 giờ | Cao: thấy được các trade-off đang thực sự tồn tại | Thấp, nếu không dùng để áp quyết định |
| Nằm trong rotation on-call (kể cả ở vai thứ hai) | Nạp | ~1 tuần mỗi 6–8 tuần | Rất cao: đây là tầng mà mọi quyết định kiến trúc bị kiểm nghiệm | Trung bình: tốn năng lượng, ảnh hưởng việc khác |
| Đọc log và dashboard 20 phút mỗi sáng | Nạp | 1,5 giờ | Trung bình-cao: giữ mô hình về tải và về failure mode | Thấp |
| Đọc mọi postmortem, tham gia ít nhất một buổi/tháng | Nạp | 1 giờ | Rất cao: postmortem là nơi khoảng cách giữa thiết kế và thực tế lộ ra rõ nhất | Thấp |
| Làm task nhỏ ngoài critical path (tooling nội bộ, script, cải thiện golden path) | Nạp | 3–4 giờ | Cao: giữ được cảm giác về ma sát thật của việc phát triển | Thấp nếu không ai chờ |
| Nhận task trên critical path | Nạp mạnh nhưng — | Không kiểm soát được | Cao | **Rất cao: bạn thành bottleneck; đây là lỗi phổ biến nhất** |
| Kể lại kinh nghiệm ở công ty cũ | Rút | 0 | Âm | Bị đọc là dùng địa vị thay lập luận |
| Quyết định dựa trên trạng thái hệ thống cũ | Rút mạnh | 0 | Âm | Một quyết định sai vì thiếu dữ liệu hiện trường mất nhiều uy tín hơn ba tháng nạp |

**Ngưỡng cảnh báo mất kết nối kỹ thuật — checklist tự kiểm mỗi tháng, 5 phút.** Nếu bạn trả lời "không"
cho từ ba câu trở lên, bạn đang trong vùng mất kết nối:

- [ ] Tôi biết tên và mục đích của ba PR lớn nhất được merge trong tuần qua.
- [ ] Tôi tự dựng được môi trường development của service chính từ máy sạch, không cần hỏi ai.
- [ ] Tôi đọc được dashboard chính và biết giá trị bình thường của 3 chỉ số quan trọng nhất.
- [ ] Tôi biết incident gần nhất là gì, nguyên nhân cơ chế của nó, và action item còn lại.
- [ ] Tôi biết deploy một thay đổi nhỏ lên production hiện nay mất bao lâu và qua những bước nào.
- [ ] Ước lượng gần nhất của tôi cho một việc kỹ thuật lệch không quá 2 lần so với thực tế.
- [ ] Tôi biết ba thứ trong hệ thống mà team đang sợ chạm vào, và vì sao.
- [ ] Trong tháng qua có ít nhất một lần một engineer thay đổi ý kiến sau khi nghe lập luận kỹ thuật của
      tôi (không phải sau khi nghe quyết định của tôi).

Câu cuối là câu quan trọng nhất, vì nó đo trực tiếp cái ta đang nói tới: influence bằng lập luận, không
bằng vị trí.

### Practical Framework

**Gói nạp lại hàng tuần — 6–8 giờ, thiết kế theo nguyên tắc "tiếp xúc, không chiếm chỗ".**

| Nhịp | Hoạt động | Thời lượng | Quy tắc để không tạo phụ thuộc |
|---|---|---|---|
| Hàng ngày | Đọc dashboard và log lỗi buổi sáng | 15–20 phút | Không mở ticket từ đây trừ khi nghiêm trọng; ghi vào một danh sách quan sát |
| Hàng ngày | Đọc PR, comment có chọn lọc | 30 phút, trong khe review cố định | Không bao giờ là reviewer bắt buộc; comment ở mức `[hỏi]` và `[nên sửa]`, hạn chế `[blocking]` |
| Hàng tuần | Tham gia một design review với vai đặt câu hỏi | 1 giờ | Nói sau cùng; đặt câu hỏi thay vì đưa kết luận (xem script dưới) |
| Hàng tuần | Một task kỹ thuật nhỏ ngoài critical path | 3–4 giờ, một khối liền | Điều kiện bắt buộc: không ai chờ nó; nếu bị gián đoạn 2 tuần thì không ai bị ảnh hưởng |
| Hàng tuần | Đọc một postmortem hoặc một ADR mới | 30 phút | — |
| 6–8 tuần | Một ca on-call, ở vai thứ hai (shadow hoặc backup) | 1 tuần | Ở vai thứ hai để không chặn việc điều phối; nhưng phải thật sự nhận cuộc gọi |
| Hàng quý | Tự dựng lại môi phát triển từ máy sạch | 2–3 giờ | Đây là phép đo trực tiếp ma sát mà mọi người mới phải trải qua; kết quả đưa vào backlog của golden path |

Tổng khoảng 6–8 giờ/tuần, tức 15–20% thời gian. Đây là mức mà theo kinh nghiệm giữ được uy tín kỹ thuật
ở một team 8–15 người mà không trở thành bottleneck.

**Chọn loại task đúng — bảng lọc.** Không phải mọi task nhỏ đều an toàn cho một lead:

| Loại task | Có nên nhận | Vì sao |
|---|---|---|
| Tooling nội bộ, script tự động hoá việc thủ công | Nên, tốt nhất | Không ai chờ; tiếp xúc thật với codebase và với ma sát; kết quả có ích ngay và tạo uy tín |
| Cải thiện golden path, template, CI | Nên | Cùng lý do; và nó cho bạn dữ liệu về trải nghiệm của engineer |
| Sửa một bug không gấp ở vùng bạn muốn hiểu | Nên | Cách rẻ nhất để nạp lại kiến thức về một module cụ thể |
| Viết test đặc tả cho một module không có test | Nên | Học được hành vi thật của hệ thống; và tạo giá trị lâu dài |
| Một phần của tính năng đang trong sprint | Không | Bạn sẽ bị gián đoạn bởi họp và bởi việc lead, và bạn sẽ chặn người khác |
| Refactor một module lõi | Không | Thời gian của bạn không dự đoán được; nhánh dài sẽ conflict |
| Phần khó nhất của một dự án quan trọng | Không, dù bạn làm nhanh nhất | Nó lấy đi cơ hội phát triển của người khác, tạo bus factor = 1 ở chính bạn, và bạn là người ít có thời gian liền mạch nhất trong team |
| Hotfix trong incident | Có, nếu bạn thật sự là người phù hợp nhất và có mặt | Đây là ngoại lệ hợp lý; nhưng sau đó phải có người khác học lại vùng đó |

**Script hội thoại — vào một design review sau ba tháng không chạm code.** Bối cảnh: team đang thiết kế
lại luồng đồng bộ tồn kho. Minh đã ba tháng không đọc vùng này.

Phiên bản nói sai:

> Minh: "Cái này anh thấy nên dùng event-driven, mình publish một event khi tồn kho đổi rồi các bên tự
> subscribe. Hồi anh làm ở công ty cũ mình làm thế, chạy rất ổn với vài triệu bản ghi mỗi ngày. Cứ làm
> theo hướng đó đi, có gì vướng thì nói anh."

Bốn vấn đề, mỗi cái tốn một phần uy tín. Đưa ra kết luận trước khi có dữ liệu về ràng buộc hiện tại. Dùng
kinh nghiệm ở nơi khác làm luận cứ chính — không kiểm chứng được và làm mờ ranh giới giữa "hệ thống đó"
và "hệ thống này". "Cứ làm theo hướng đó" đóng không gian thảo luận bằng thẩm quyền. Và "có gì vướng thì
nói anh" đặt chi phí phản biện lên vai người khác trong khi chính anh là người thiếu dữ liệu.

Nếu team im lặng và làm theo, có hai kết cục và cả hai đều xấu: hoặc hướng đó sai và uy tín Minh mất
nhiều hơn số anh vừa dùng, hoặc hướng đó đúng nhưng team học được rằng thảo luận kỹ thuật ở đây kết thúc
bằng ý của lead.

Phiên bản nói đúng:

> Minh: "Tôi đã ba tháng không đọc vùng này nên tôi nói trước là mô hình của tôi có thể cũ — mọi người
> sửa tôi khi tôi nói sai. Tôi muốn hỏi bốn câu trước khi có ý kiến.
>
> Thứ nhất, hiện nay số lần tồn kho thay đổi mỗi phút ở giờ cao điểm là bao nhiêu, và bên nào đang đọc
> nó? Tôi cần biết số này vì nó quyết định phần lớn thiết kế.
>
> Thứ hai, cái gì đang thực sự hỏng? Tôi thấy hai khả năng khác nhau: hoặc là dữ liệu không nhất quán
> giữa các bên đọc, hoặc là độ trễ cập nhật quá lớn. Hai vấn đề này có lời giải khác nhau, và nếu chúng
> ta chưa xác định là cái nào thì mọi tranh luận về event-driven đều sớm.
>
> Thứ ba, nếu tồn kho hiển thị sai trong 30 giây thì hậu quả nghiệp vụ là gì — có bán quá số không, và
> nếu có thì bao nhiêu đơn mỗi ngày? Tôi hỏi vì đây là thứ quyết định mình cần consistency ở mức nào,
> và đó là câu hỏi cho Trang chứ không phải cho chúng ta.
>
> Thứ tư, ai sẽ vận hành cái này lúc 2 giờ sáng, và họ debug nó thế nào?
>
> Còn về hướng event-driven: tôi có làm ở một hệ thống khác nên tôi biết hai chỗ nó thường đau — thứ tự
> event khi có nhiều nguồn cập nhật, và việc reconcile khi consumer bị lag. Tôi không biết hai chỗ đó
> có là vấn đề ở đây hay không; nếu Hà đã nghĩ tới rồi thì tôi không thêm gì. Còn nếu chưa, đó là hai
> câu tôi đề nghị trả lời trong RFC.
>
> Về quyết định: Hà là owner vùng này nên Hà chốt, không phải tôi. Tôi có một yêu cầu duy nhất — thứ tự
> quality attribute phải được xác nhận với Trang trước khi so sánh phương án, vì đó là chỗ chúng ta hay
> đi vòng."

Năm cơ chế trong đoạn này: (1) tuyên bố trước giới hạn dữ liệu của mình — điều này tăng uy tín thay vì
giảm, vì nó cho thấy bạn biết mình biết gì; (2) hỏi số trước khi có ý kiến, và các câu hỏi đó tự cho
thấy chất lượng mô hình tư duy — đây là cách một lead thể hiện năng lực kỹ thuật hiệu quả nhất khi không
còn ở trong chi tiết; (3) tách vấn đề thành hai khả năng có lời giải khác nhau, tức đóng góp bằng cách
làm rõ bài toán chứ không bằng cách đề xuất lời giải; (4) dùng kinh nghiệm cũ ở dạng **chỉ ra failure
mode cần kiểm tra**, không ở dạng kết luận; (5) trả quyền quyết định cho owner và chỉ giữ một yêu cầu về
quy trình — đó chính là công việc thật của một lead ở tầng này.

### Trade-off

**Trade-off chính — Viết code vs không viết code.**

| | Lead viết code đáng kể (> 40% thời gian) | Lead gần như không viết code (< 10%) |
|---|---|---|
| Được | Uy tín kỹ thuật cao; hiểu chi tiết; ước lượng chính xác; thấy ma sát thật ngay lập tức | Leverage cao: thời gian dùng cho việc chỉ mình làm được — dỡ vật cản, định hình ràng buộc, phát triển người, đàm phán với stakeholder |
| Mất | Trở thành bottleneck; lấy cơ hội phát triển của người khác; bị gián đoạn liên tục nên chất lượng code thấp và không dự đoán được; các việc lead bị làm dở | Mất chi tiết; ước lượng lệch; quyết định dựa trên dữ liệu cũ; dễ mất kết nối với đội |
| Nghiêng về khi | Team ≤ 4 người; giai đoạn đầu của một sản phẩm; bạn là người duy nhất biết một miền quan trọng (nhưng phải có kế hoạch thoát) | Team ≥ 8 người; nhiều stakeholder; nhiều người mới cần kèm; nhiều quyết định xuyên team |
| Ngưỡng theo quy mô | 2–4 người: 50–70% code. 5–7 người: 25–40%. 8–12 người: 10–20%, và chỉ ngoài critical path. > 12 người hoặc nhiều team: dưới 10%, chủ yếu là tooling và đọc | | |

Điểm quan trọng nhất trong bảng này: **con số không quan trọng bằng loại công việc.** Một lead dành 20%
thời gian cho tooling nội bộ, đọc PR và một ca on-call giữ được uy tín tốt hơn một lead dành 40% thời
gian cho một phần tính năng trên critical path — vì người thứ hai vừa chặn team vừa liên tục xin lỗi vì
chậm.

**Trade-off phụ — Nhận on-call vs không.** On-call là cách nạp uy tín hiệu quả nhất trên một đơn vị thời
gian, và nó cũng gửi một tín hiệu văn hoá mạnh (người quyết kiến trúc cũng là người trực hậu quả — nối
với chủ đề 2 và 4). Chi phí thật: nó tiêu năng lượng và làm giảm chất lượng các quyết định khác trong
tuần đó, và một lead không được ngủ là một rủi ro cho tổ chức. Điều kiện nghiêng về nhận: ở vai thứ hai,
tần suất 6–8 tuần một lần, và với điều kiện tuần đó bạn không có việc lead nào quan trọng. Nghiêng về
không nhận: khi bạn là người duy nhất xử lý được một số leo thang tổ chức, hoặc khi rotation đã quá mỏng
và việc bạn tham gia chỉ làm mọi người dựa vào bạn.

**Trade-off thứ ba — Nói ra sự không chắc chắn vs giữ hình ảnh chắc chắn.** Trong nhiều tổ chức Việt
Nam, có kỳ vọng ngầm rằng người lead phải luôn có câu trả lời, và việc nói "tôi không biết" bị lo là mất
uy tín. Thực tế theo kinh nghiệm là ngược lại, với một điều kiện: nói "tôi không biết" phải luôn kèm
"đây là cách tôi sẽ tìm ra, và trước ngày này". "Tôi không biết" đơn thuần thì đúng là làm giảm niềm
tin; "tôi không biết, tôi sẽ đọc số liệu tải và trả lời trước thứ Năm" thì làm tăng, vì nó thể hiện một
phương pháp. Điều kiện nghiêng về giữ hình ảnh chắc chắn: rất ít — chủ yếu là trong lúc đang xử lý một
incident, khi tổ chức cần một người ra quyết định dứt khoát và sự bàn luận về mức độ chắc chắn làm chậm
việc.

### Real-world Scenarios

**Tình huống A — Lead nhận task trên critical path và làm chậm cả team.** Bối cảnh: startup 22 engineer.
Minh nhận phần tích hợp cổng thanh toán mới, phần khó nhất của sprint, vì "tôi làm nhanh nhất và đây là
việc quan trọng nhất". Ước lượng 4 ngày.

Thực tế: tuần đó có hai buổi họp với đối tác, một buổi review roadmap quý, một cuộc 1-1 khủng hoảng với
một người định nghỉ, và một incident. Minh làm xong sau 11 ngày. Trong 11 ngày đó, hai người khác bị
chặn (phần UI và phần đối soát đều chờ contract của anh), và Minh phải nói "sắp xong" ba lần trong
standup.

Kế toán chi phí đầy đủ, và đây là phần đáng làm với mọi lần lead định nhận task quan trọng: 11 ngày của
Minh, cộng khoảng 8 ngày chờ của hai người khác, cộng chi phí uy tín của ba lần "sắp xong" — trong đó
khoản thứ ba là đắt nhất và không hiện trong bất kỳ báo cáo nào. Đổi lại: Minh nạp được kiến thức về một
miền, và tiết kiệm được có lẽ 2–3 ngày so với việc Nam làm.

Điều đáng nói là **lý do Minh nhận task không phải là lý do anh nói ra.** Lý do thật, anh thừa nhận
trong một cuộc 1-1 với Khoa: anh cảm thấy không thoải mái khi cả tuần không viết dòng code nào, và anh
lo team nghĩ mình không còn làm kỹ thuật. Đây là động cơ rất phổ biến ở lead mới trong 12–18 tháng đầu,
và nó cần được nhận diện đúng — vì nếu bạn không giải quyết nó bằng một kênh an toàn (tooling nội bộ,
on-call, task ngoài critical path), nó sẽ tự giải quyết bằng cách bạn nhận task trên critical path.

Cách sửa: Minh chuyển sang một nguyên tắc cứng — task kỹ thuật của anh phải thoả điều kiện "nếu tôi
không làm gì trong hai tuần thì không ai bị ảnh hưởng". Trong hai quý sau, anh làm: một công cụ sinh
dữ liệu test cho môi trường staging (tiết kiệm khoảng 3 giờ/tuần cho cả team), viết lại phần dựng môi
trường local (giảm onboarding từ 2 ngày xuống 3 giờ), và nhận một ca on-call mỗi 8 tuần. Kết quả về uy
tín tốt hơn hẳn: hai công cụ đó được cả team dùng hàng ngày, và mỗi lần dùng là một lần nhắc rằng lead
vẫn làm kỹ thuật.

**Tình huống B — Ba góc nhìn về việc lead có nên nằm trong rotation on-call.** Bối cảnh: một ví điện tử,
team 12 người, rotation on-call 5 người. Đề xuất: Minh (Tech Lead) tham gia rotation ở vai thứ hai.

*Nhìn từ IC (Nam, đang trong rotation):* "Tôi ủng hộ, nhưng vì một lý do có thể anh không nghĩ tới. Tôi
không cần thêm một người trực — tôi cần người quyết định kiến trúc hiểu rằng khi service payment timeout
lúc 2 giờ sáng thì tôi không có log để biết request nào bị ảnh hưởng. Tôi đã nói việc này trong hai
retro và nó không được ưu tiên. Nếu anh trực một lần, việc đó sẽ được ưu tiên."

*Nhìn từ Tech Lead (Minh):* "Tôi thấy ba lợi ích và một chi phí. Lợi ích: tôi nạp lại kiến thức vận
hành, thứ đang phân rã nhanh nhất; tôi có dữ liệu trực tiếp để ưu tiên các việc về observability mà
hiện tôi chỉ nghe qua báo cáo; và nó là một tín hiệu văn hoá — người quyết kiến trúc cũng trực hậu quả.
Chi phí: tuần on-call tôi sẽ ra quyết định kém hơn vì thiếu ngủ, nên tôi phải tránh xếp các việc quan
trọng vào tuần đó. Tôi chọn vai thứ hai để không chặn việc điều phối, nhưng tôi phải thật sự nhận cuộc
gọi — nếu chỉ shadow trên giấy thì không có tác dụng gì và còn tệ hơn vì nó là hình thức."

*Nhìn từ Manager (Linh):* "Tôi phải cân nhắc thứ mà cả hai không thấy: nếu Minh trong rotation và có
một incident lớn trùng với tuần anh ấy cần xử lý một vấn đề nhân sự hoặc một cuộc đàm phán quan trọng,
tôi mất cả hai. Nên tôi đồng ý với hai điều kiện. Một, tần suất 8 tuần một lần, và có quyền đổi ca khi
trùng việc quan trọng — nhưng phải đổi trước, không phải bỏ giữa ca. Hai, sau mỗi ca của Minh, tôi muốn
một danh sách 3 việc anh ấy thấy cần sửa từ hiện trường, và ba việc đó vào backlog có ưu tiên. Nếu
không có đầu ra đó thì việc anh ấy trực chỉ là nạp uy tín cá nhân, và tôi cần nó tạo giá trị cho hệ
thống."

Yêu cầu của Linh ở cuối là yêu cầu đúng và nó tổng quát được: **mọi hoạt động nạp uy tín kỹ thuật nên
có một đầu ra cho tổ chức, không chỉ cho bản thân bạn.** Đọc PR thì cho ra một quan sát về chỗ nào hay
tranh luận. On-call thì cho ra danh sách việc cần sửa. Tự dựng môi trường thì cho ra backlog cho golden
path. Đây cũng là cách bạn bảo vệ 15–20% thời gian đó khi ai đó hỏi nó dùng để làm gì.

### Best Practices

- **Đặt một khối 3–4 giờ liền mạch mỗi tuần cho việc kỹ thuật, trong calendar, và bảo vệ nó như một
  cuộc họp với CEO.** Cơ chế: công việc kỹ thuật cần thời gian liền mạch; 8 lần 30 phút không tương
  đương một lần 4 giờ. Nếu không đặt trước, nó sẽ bị các việc khác chiếm hết, mỗi tuần.
- **Chọn task theo tiêu chí "không ai chờ", không theo tiêu chí "quan trọng nhất".** Cơ chế: việc quan
  trọng nhất là việc có người phụ thuộc, và bạn là người có thời gian ít dự đoán được nhất trong team.
- **Nằm trong rotation on-call ở vai thứ hai, tần suất 6–8 tuần.** Cơ chế: đây là điểm tiếp xúc có mật
  độ thông tin cao nhất về khoảng cách giữa thiết kế và thực tế; và nó xoá được externality "người quyết
  không chịu hậu quả" ở chủ đề 2.
- **Vào design review với câu hỏi, ra khỏi design review không kèm quyết định của mình** (trừ khi bạn là
  owner). Cơ chế: chất lượng câu hỏi là cách thể hiện năng lực kỹ thuật hiệu quả nhất ở tầng này, và nó
  không lấy đi quyền quyết định của owner.
- **Nói ra giới hạn dữ liệu của mình trước khi nêu ý kiến.** Cơ chế: nó cho phép người khác sửa bạn mà
  không phải mạo hiểm về mặt quan hệ — điều đặc biệt quan trọng trong văn hoá tránh xung đột trực diện;
  và nó làm ý kiến của bạn được cân đúng trọng số thay vì được cân theo chức danh.
- **Đưa ra dự đoán cụ thể có thể kiểm chứng, và ghi lại.** Cơ chế: uy tín được xây bằng phán đoán được
  kiểm chứng công khai. Nối với chương 04: viết dự đoán kèm mức tin cậy và ngày kiểm chứng.
- **Mỗi hoạt động nạp uy tín phải có một đầu ra cho tổ chức.** Cơ chế: nó biến 15–20% thời gian từ một
  đặc quyền cần biện minh thành một hạng mục có giá trị đo được; và nó là cách nó tồn tại được qua các
  kỳ áp lực.
- **Chạy checklist 8 câu ở mục Mental Model mỗi tháng, và ghi kết quả.** Cơ chế: mất kết nối kỹ thuật
  diễn ra chậm và không có khoảnh khắc báo động; chỉ một phép đo định kỳ mới bắt được nó khi còn rẻ.

### Anti-patterns

| Anti-pattern | Cơ chế phá hệ thống | Dấu hiệu sớm |
|---|---|---|
| **Lead nhận task trên critical path rồi làm chậm cả team** | Thời gian của lead bị gián đoạn không dự đoán được → task trượt → người khác bị chặn; và lead phải chọn giữa việc lead và việc code, thường làm dở cả hai | Lead nói "sắp xong" ≥ 2 lần trong standup cho cùng một việc; có PR của lead mở quá 5 ngày |
| **Lead phủ nhận hoàn toàn việc kỹ thuật rồi ra quyết định sai vì không có dữ liệu hiện trường** | Mô hình trong đầu lệch với hệ thống thật → ước lượng sai theo hướng lạc quan, ưu tiên sai, và các quyết định kiến trúc dựa trên giả định đã hết hạn | Ước lượng của lead lệch > 3 lần; lead không đọc được dashboard hiện tại; trong incident lead phải hỏi mọi thứ |
| **Lead dùng uy tín cũ để áp quyết định mới** | Uy tín là niềm tin về phán đoán hiện tại; dùng thành tích cũ làm luận cứ vừa không kiểm chứng được vừa dạy team rằng tranh luận kỹ thuật ở đây được quyết bằng thâm niên → engineer giỏi ít thâm niên rút khỏi thảo luận | Câu "hồi anh làm ở..." xuất hiện như luận cứ chính; các quyết định kỹ thuật kết thúc mà không có ai nêu số liệu |
| **Lead review mọi PR và thành reviewer bắt buộc** | Bottleneck hàng đợi (chủ đề 3); và nó chặn việc các engineer khác phát triển phán đoán kỹ thuật vì luôn có người phán xử cuối | Lead là reviewer trong > 40% PR; PR bị treo khi lead nghỉ phép |
| **Lead viết code trong bí mật vào buổi đêm rồi push** | Không ai review được; tạo ra một vùng code chỉ lead hiểu; và nó gửi tín hiệu rằng quy trình chung không áp dụng cho lead | Commit của lead vào lúc 23h–2h; commit trực tiếp vào main; PR của lead không ai comment |
| **Lead nói "để tôi làm cho nhanh" khi thấy người khác chậm** | Lấy đi cơ hội học của người đó; và tạo kỳ vọng rằng khi khó thì lead sẽ nhận, làm giảm ownership của cả team | Nghe câu này ≥ 1 lần/sprint; các engineer mid-level không tiến bộ ở các miền khó |
| **Học công nghệ mới thay vì tiếp xúc hệ thống hiện tại** | Cái phân rã là kiến thức về hệ thống cụ thể, không phải kiến thức công nghệ; nên đọc sách về một framework mới không nạp lại được cái đang mất | Lead nói nhiều về công nghệ chưa dùng trong tổ chức, không nói được số liệu về hệ thống đang chạy |

### Khi nào KHÔNG nên áp dụng

**Khi team dưới 4–5 người.** Ở quy mô này, lead vẫn nên là một người đóng góp kỹ thuật chính — 50–70%
thời gian viết code là hợp lý, và việc "giữ uy tín" không phải là bài toán vì bạn đang ở trong hiện
trường mỗi ngày. Bài toán thật của bạn là ngược lại: dành đủ thời gian cho việc lead, vì với 4 người
thì các việc lead vẫn tồn tại (làm việc với stakeholder, phát triển người, ưu tiên) và chúng thường bị
bỏ trước.

**Trong 4–8 tuần đầu khi bạn mới nhận vai lead.** Trong giai đoạn này, việc đúng không phải giữ uy tín
kỹ thuật — bạn vừa mới có nó — mà là hiểu bối cảnh mới: ai là stakeholder, hệ khuyến khích thật là gì,
những cam kết nào đang treo, ai đang có vấn đề. Cắt gần hết việc kỹ thuật trong 4–8 tuần đầu là quyết
định đúng, và uy tín kỹ thuật của bạn không phân rã đáng kể trong thời gian đó. Sai lầm phổ biến là dùng
việc kỹ thuật làm nơi trú ẩn khỏi sự khó chịu của vai mới.

**Khi bạn đang trong một khủng hoảng tổ chức.** Nếu đang có ba người định nghỉ, một cuộc tái cấu trúc,
hoặc một khách hàng lớn định rời đi — thì 15–20% thời gian cho việc kỹ thuật là phân bổ sai. Uy tín kỹ
thuật phân rã chậm (tính theo tháng); các vấn đề tổ chức trên thì phân rã theo ngày. Quay lại nạp khi
tình hình ổn định, và chấp nhận rằng bạn sẽ phải nạp bù nhiều hơn.

**Khi việc bạn tham gia kỹ thuật thực sự lấn quyền của người khác.** Nếu bạn có một Staff Engineer hoặc
một Tech Lead cấp dưới đang sở hữu miền kỹ thuật đó, thì việc bạn đọc PR và bình luận ở đó không phải là
nạp uy tín — nó là làm mờ ranh giới ownership, và người đó sẽ dần ngừng ra quyết định. Ở tình huống này,
kênh nạp đúng là các miền **không có ai sở hữu**: tooling nội bộ, golden path, observability xuyên hệ
thống, và các phân tích dữ liệu về hệ thống mà không ai có thời gian làm.

**Khi bạn đã chuyển hẳn sang nhánh Manager và tổ chức đã có tầng technical leadership riêng.** Nếu bạn
là EM của 3 team và có 3 Tech Lead cộng một Staff Engineer, thì cố giữ uy tín kỹ thuật ở mức chi tiết là
phân bổ thời gian sai và có nguy cơ vi quản lý. Cái bạn cần giữ ở tầng này khác về loại: khả năng đọc
được một tài liệu thiết kế và đặt được ba câu hỏi đúng, khả năng phân biệt một lập luận kỹ thuật vững
với một lập luận nghe hay, và đủ kiến thức để biết khi nào cần gọi ai. Nó nạp bằng đọc RFC và postmortem,
không bằng viết code. Chương 11 nói về sự chuyển dịch này.

---

## Tự kiểm tra

Bảy câu hỏi dưới đây không trả lời được bằng cách suy nghĩ chung chung. Mỗi câu yêu cầu bạn mở một hệ
thống thật — repo, dashboard, calendar, wiki — và viết ra một câu trả lời cụ thể.

1. **Mở tài liệu technical strategy của tổ chức bạn (hoặc thừa nhận là không có) và tìm phần "cố tình
   không làm".** Nếu không có phần đó, hãy viết ngay bây giờ bốn dòng: bốn việc kỹ thuật hợp lý mà tổ
   chức bạn sẽ không làm trong hai quý tới, mỗi dòng kèm lý do và điều kiện xét lại. Nếu bạn không viết
   được bốn dòng, bạn không có chiến lược — bạn có một danh sách ước muốn.

2. **Chọn ba quyết định kiến trúc quan trọng nhất trong hệ thống bạn đang làm. Với mỗi cái, tìm xem: ai
   quyết, khi nào, những phương án nào đã bị loại và vì lý do gì.** Đếm bạn tìm được mấy trên chín thông
   tin đó. Với mỗi thông tin không tìm được, hỏi: nếu người biết nó rời công ty tháng sau thì tổ chức
   mất gì?

3. **Lấy số liệu review của team bạn trong 4 sprint gần nhất: trung vị thời gian tới comment đầu tiên,
   trung vị số dòng mỗi PR, tỉ lệ PR không có comment thực chất, và phân bố reviewer.** Bốn số đó nằm ở
   đâu so với các ngưỡng ở chủ đề 3? Số nào bạn sửa được trong hai tuần tới mà không cần ai phê duyệt?

4. **Đếm số MUST trong bộ standard của tổ chức bạn, và đếm bao nhiêu trong số đó có kiểm tra tự động.**
   Nếu tỉ lệ dưới 50%, chọn ba MUST không có cơ chế: bạn sẽ tự động hoá cái nào, và cái nào bạn sẽ trung
   thực hạ xuống SHOULD trong tháng này?

5. **Tính lãi của module tệ nhất trong hệ thống của bạn bằng số:** lead time trung vị của thay đổi trong
   module đó, trừ đi baseline, nhân với số PR mỗi quý, cộng chi phí làm lại. Con số đó là bao nhiêu
   người-ngày mỗi quý? Gốc để sửa là bao nhiêu? Bây giờ viết bốn câu xin ngân sách theo cấu trúc ở Bước
   4 của chủ đề 5 — và câu thứ tư phải nêu rõ việc gì bạn đề nghị lùi lại.

6. **Điền bảng sáu tín hiệu culture ở chủ đề 7 cho tổ chức bạn, đặc biệt là tín hiệu 1 (ai được promote
   và vì lý do thật gì) và tín hiệu 6 (cái gì bị bỏ khi gấp).** Đặt kết quả cạnh danh sách giá trị tuyên
   bố của công ty. Khoảng cách lớn nhất nằm ở đâu, và cơ chế nào đang sinh ra khoảng cách đó? Cơ chế đó
   có nằm trong phạm vi bạn kiểm soát không — nếu không, ai kiểm soát nó?

7. **Chạy checklist 8 câu về mất kết nối kỹ thuật ở chủ đề 8, ngay bây giờ, không tra cứu.** Bạn trả lời
   "không" mấy câu? Với câu cuối — lần gần nhất một engineer thay đổi ý kiến sau khi nghe **lập luận**
   của bạn chứ không phải sau khi nghe **quyết định** của bạn là khi nào? Nếu bạn không nhớ được trong
   một tháng qua, hãy đặt vào calendar tuần tới một khối 3 giờ cho một task kỹ thuật thoả điều kiện
   "nếu tôi không làm gì trong hai tuần thì không ai bị ảnh hưởng".

---

## Liên kết chương khác

- [`00-nen-tang-leadership.md`](/series/engineering-leadership/00-nen-tang-leadership/) — Ownership và Accountability là điều kiện
  tiên quyết: mọi hiện vật trong chương này (ADR, RFC, debt register, exception) đều có một trường "một
  tên người", và trường đó vô nghĩa nếu tổ chức chưa phân biệt được Responsibility với Accountability.
- [`02-communication.md`](/series/engineering-leadership/02-communication/) — Technical Leadership hoạt động qua văn bản và qua các
  cuộc trao đổi khó: chương đó là phần kỹ thuật truyền đạt (viết cho stakeholder không kỹ thuật, dịch
  giá trị kỹ thuật sang ngôn ngữ tiền), chương này là phần nội dung được truyền đạt.
- [`03-team-leadership.md`](/series/engineering-leadership/03-team-leadership/) — Psychological Safety là điều kiện để code review và
  RFC nhận được phản biện thật thay vì sự im lặng lịch sự; và Delegation là cơ chế để bạn không trở
  thành người quyết mọi thứ khi đã có standard và golden path.
- [`04-decision-making.md`](/series/engineering-leadership/04-decision-making/) — Chương đó là quá trình sản xuất một quyết định;
  chương này là cách làm cho hàng nghìn quyết định bạn không tham gia vẫn hội tụ. Bảng 4 mức
  reversible/irreversible ở chương 04 là bộ lọc đầu vào cho ADR (chủ đề 2) và RFC (chủ đề 4).
- [`06-incident-va-metrics.md`](/series/engineering-leadership/06-incident-va-metrics/) — Postmortem là nơi khoảng cách giữa kiến
  trúc thiết kế và hệ thống thật lộ ra, và là một trong sáu tín hiệu đọc culture; DORA metrics là dữ
  liệu đầu vào để đo technical debt (chủ đề 5) và để đánh giá review latency (chủ đề 3).
- [`07-project-delivery.md`](/series/engineering-leadership/07-project-delivery/) — Architectural runway (chủ đề 1) là điều kiện để
  Estimation không sai hệ thống; và lãi của technical debt là biến giải thích phần lớn hiện tượng ước
  lượng luôn trượt theo cùng một hướng.
- [`08-hiring-va-phat-trien.md`](/series/engineering-leadership/08-hiring-va-phat-trien/) — Tiêu chí promotion là cơ chế văn hoá có
  đòn bẩy dài nhất (chủ đề 7), và golden path quyết định phần lớn thời gian onboarding của người mới —
  hai chỗ mà technical leadership tác động trực tiếp vào tầng People.
- [`09-to-chuc-va-scaling.md`](/series/engineering-leadership/09-to-chuc-va-scaling/) — Ranh giới team quyết định ranh giới service
  hợp lý ("ranh giới service không nhỏ hơn ranh giới team", ADR-014); và ngưỡng số team là biến chính
  trong mọi trade-off Standardization vs Flexibility ở chương này.
- [`10-case-studies.md`](/series/engineering-leadership/10-case-studies/) — Các tình huống dài đi trọn chuỗi bối cảnh → phương án →
  trade-off → quyết định → hậu quả → bài học, dùng chính các template ADR, RFC và debt register ở đây.
- [`11-career-evolution.md`](/series/engineering-leadership/11-career-evolution/) — Chủ đề 8 là một mảnh của bài toán lớn hơn: nhánh
  Staff/Principal giữ uy tín kỹ thuật bằng chiều sâu kỹ thuật, nhánh Manager giữ nó bằng khả năng đọc và
  phản biện thiết kế; chương đó nói về cách chọn giữa hai nhánh.
- [`12-anti-patterns.md`](/series/engineering-leadership/12-anti-patterns/) — Tổng hợp xuyên chương các mẫu architecture astronaut,
  rubber-stamp review, big rewrite, culture deck, hero culture và standard-không-thực-thi cùng biểu hiện
  của chúng ở tầng tổ chức.
