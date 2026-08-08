+++
title = "Engineering Career Evolution — Từ Junior đến Principal, Engineering Manager và Director"
date = "2026-08-01T19:00:00+07:00"
draft = false
tags = ["engineering-leadership", "management", "career"]
series = ["Engineering Leadership & Technical Management"]
+++

# Engineering Career Evolution

Một kỹ sư ở công ty fintech Việt được promote lên Senior sau ba năm vì viết code sạch, review kỹ, không bao giờ để bug lọt production. Hai năm sau anh ta được đề nghị làm Tech Lead. Sáu tháng sau đó, team của anh trễ hai release liên tiếp, hai thành viên xin chuyển team, và trong buổi review với Engineering Manager anh nói một câu rất thật: "Em không hiểu mình làm sai ở đâu. Em vẫn làm đúng những thứ trước giờ em làm — code em vẫn tốt nhất team, em vẫn review kỹ nhất, em vẫn là người sửa mọi sự cố."

Đó chính là chỗ sai. Anh ta leo thang bằng cách làm tốt hơn thứ mình đã giỏi. Nhưng ở bậc mới, thứ anh giỏi không còn là thứ được đo. Cái thay đổi khi lên bậc không phải **độ khó** của công việc — mà là **hàm mục tiêu**: định nghĩa của "thành công" bị viết lại. Ở bậc Senior, thành công là "tôi giao được thứ khó". Ở bậc Tech Lead, thành công là "team giao được thứ khó, kể cả khi tôi không viết dòng nào". Hai câu đó không phải hai mức độ của cùng một thang — chúng là hai hàm khác nhau, và tối ưu hàm cũ mạnh hơn sẽ làm bạn thua ở hàm mới.

Chương này không phải bảng lương. Không phải bảng title. Không phải danh sách yêu cầu để bạn mang đi đàm phán promotion — mặc dù nó có thể dùng cho việc đó. Nó là bản đồ về việc **cái gì thay đổi** khi phạm vi ảnh hưởng của bạn mở rộng: hàm mục tiêu nào được dùng để đo bạn, loại quyết định nào rơi vào tay bạn, ai là người có quyền phản biện bạn, thời gian một ngày của bạn bị tiêu vào đâu, và những gì bạn phải bỏ lại. Mỗi bậc ở đây được mô tả bằng cùng bảy câu hỏi, để bạn đọc theo chiều dọc (bậc này gồm những gì) hoặc chiều ngang (hàm mục tiêu biến đổi thế nào qua tám bậc).

Ba nguyên tắc đọc chương này:

1. **Bậc không phải title.** Title là cái công ty in trên name card. Bậc là tổ hợp phạm vi ảnh hưởng, độ mơ hồ của bài toán, và cách bạn tạo ra kết quả. Ở Việt Nam hai thứ này lệch nhau rất xa (xem mục Bối cảnh Việt Nam).
2. **Không phải bậc nào cũng tồn tại ở mọi công ty.** Staff Engineer ở một ODC 40 người thường không có chỗ đứng thật. Director ở một startup 25 người là một cái tên, không phải một vai trò.
3. **Đi lên không phải hướng duy nhất.** Mở rộng phạm vi ở cùng một bậc, hoặc chuyển nhánh ngang, là những nước đi hợp lý và trong nhiều trường hợp tốt hơn việc leo lên một bậc mà bạn chưa muốn hàm mục tiêu của nó.

**Mục lục nội bộ**

- [Bảng tổng quan tám bậc](#bảng-tổng-quan-tám-bậc)
- [Ba trục thật của sự thăng tiến](#ba-trục-thật-của-sự-thăng-tiến)
- [1. Junior Engineer](#1-junior-engineer)
- [2. Mid-level Engineer](#2-mid-level-engineer)
- [3. Senior Engineer](#3-senior-engineer)
- [4. Tech Lead](#4-tech-lead)
- [5. Staff Engineer](#5-staff-engineer)
- [6. Principal Engineer](#6-principal-engineer)
- [7. Engineering Manager](#7-engineering-manager)
- [8. Director of Engineering](#8-director-of-engineering)
- [Bốn chuyển dịch khó nhất](#bốn-chuyển-dịch-khó-nhất)
- [Chọn nhánh: IC track vs Manager track](#chọn-nhánh-ic-track-vs-manager-track)
- [Bối cảnh Việt Nam](#bối-cảnh-việt-nam)
- [Bản đồ tự đánh giá](#bản-đồ-tự-đánh-giá)
- [Tự kiểm tra](#tự-kiểm-tra)
- [Liên kết chương khác](#liên-kết-chương-khác)

---

## Bảng tổng quan tám bậc

Đọc bảng này theo chiều ngang trước. Chú ý rằng cột "Đầu ra chính" đổi bản chất hai lần: từ *code* sang *hệ thống và người*, rồi sang *cơ chế*. Đó là hai điểm rẽ thật của cả sự nghiệp.

| Bậc | Phạm vi ảnh hưởng | Đầu ra chính | Loại bài toán | Cách ra quyết định | Chân trời thời gian | Nguồn quyền lực |
|---|---|---|---|---|---|---|
| **1. Junior** | Task được giao trong một module | Code đúng, chạy được, có test | Bài toán đã được người khác định nghĩa và chia nhỏ | Hỏi và làm theo; quyết định trong phạm vi implementation | 1 ngày – 1 sprint | Không có; ảnh hưởng đến từ chất lượng công việc được kiểm chứng |
| **2. Mid-level** | Một feature hoặc một module | Feature hoàn chỉnh, tự chịu trách nhiệm end-to-end | Bài toán được định nghĩa ở mức yêu cầu, cách làm tự chọn | Tự quyết trong module; escalate khi vượt ranh giới | 1 sprint – 1 quý | Độ tin cậy (reliability): giao đúng cái đã hứa |
| **3. Senior** | Một hệ thống hoặc một domain; 3–6 người quanh mình | Hệ thống chạy được lâu dài + năng lực của người quanh mình tăng lên | Bài toán khó nhưng đã được nhận diện là bài toán | Trade-off kỹ thuật có tài liệu (ADR/RFC); tự chịu hậu quả | 1 quý – 1 năm | Uy tín kỹ thuật đã được kiểm chứng qua các quyết định cũ |
| **4. Tech Lead** (role) | Một team 4–8 người, trong một chu kỳ delivery | Delivery của team + chất lượng kỹ thuật của team | Bài toán tổ chức-kỹ thuật lai: scope, thứ tự, ai làm gì | Quyết định phân bổ và ưu tiên kỹ thuật; đàm phán scope với PO | 1 sprint – 2 quý | Vai trò được giao + uy tín kỹ thuật; không có quyền nhân sự |
| **5. Staff** | Một lĩnh vực (domain) cắt qua 2–5 team | Quyết định kiến trúc, chuẩn, và bài toán được định nghĩa lại | Bài toán khó, đa team, không ai sở hữu trọn vẹn | Bằng bằng chứng và văn bản; thuyết phục qua peer, không qua quyền | 2 quý – 2 năm | Ảnh hưởng (influence) không cần thẩm quyền; chất lượng phán đoán |
| **6. Principal** | Toàn tổ chức engineering; đôi khi cả thị trường/ngành | Định hướng kỹ thuật dài hạn; bài toán được phát hiện trước khi thành khủng hoảng | Bài toán **chưa ai biết là bài toán** | Định khung (framing) vấn đề cho người khác quyết; ra quyết định ít nhưng đắt | 1 – 5 năm | Track record dài + khả năng nhìn trước; được executive tin |
| **7. Engineering Manager** | 5–10 người báo cáo trực tiếp | Năng lực và kết quả của người khác; sức khoẻ của team | Bài toán về người, hệ thống động lực, và throughput của team | Quyết định về người, ưu tiên, và ranh giới; chịu trách nhiệm cả khi không đồng ý | 1 quý – 1 năm | Thẩm quyền chính thức + niềm tin của từng người |
| **8. Director** | 3–6 manager, 30–80 người, nhiều team | Một tổ chức chạy được khi mình không có mặt | Bài toán về cấu trúc, cơ chế, và phân bổ nguồn lực | Quyết định qua thiết kế tổ chức và cơ chế, ít khi qua can thiệp trực tiếp | 2 – 3 năm | Vị trí trong hệ thống + độ tin cậy của các dự báo đã đưa ra |

Ba quan sát từ bảng trên, quan trọng hơn bản thân bảng:

- **Cột "Nguồn quyền lực" đổi ba lần.** Junior/Mid dựa vào chất lượng công việc. Senior/Staff/Principal dựa vào uy tín và ảnh hưởng — thứ không ai cấp cho bạn, và cũng không ai lấy được. Tech Lead/EM/Director có thêm thẩm quyền chính thức — thứ công ty cấp và công ty rút lại được. Người nhầm hai loại này với nhau sẽ dùng sai công cụ: một Staff Engineer cố ép người khác bằng "vì tôi là Staff" sẽ mất ảnh hưởng, và một EM chỉ dựa vào uy tín kỹ thuật mà không dùng thẩm quyền sẽ để những vấn đề về người mưng mủ.
- **Cột "Loại bài toán" là trục khó nhìn nhất.** Từ "bài toán được giao" → "bài toán khó" → "bài toán chưa ai biết là bài toán". Hầu hết các cuộc tranh luận về "sao tôi chưa được lên Staff" thực chất là tranh luận về cột này: bạn đang giải rất giỏi những bài toán người khác đưa cho bạn.
- **Chân trời thời gian giãn ra theo hàm số nhân.** Từ 1 ngày lên 5 năm là ba bậc độ lớn. Đây là lý do một người quen với feedback loop hàng ngày (compile, test, deploy) thường rất khó chịu ở bậc EM và Director, nơi feedback loop dài hàng quý.

---

## Ba trục thật của sự thăng tiến

Career ladder thường được vẽ như một cầu thang: một trục, tám bậc, đi từ dưới lên. Cách vẽ đó tiện cho HR nhưng sai về mặt mô tả, và cái sai đó gây ra hai hệ quả cụ thể: người ta tưởng chỉ cần "làm tốt hơn" là lên bậc, và người ta không hiểu vì sao hai người cùng title lại khác nhau đến thế.

Thực tế có **ba trục tăng độc lập với nhau**. Vị trí của bạn là một điểm trong không gian ba chiều, không phải một số trên một thang.

### Trục 1 — Phạm vi (Scope)

Chuỗi mở rộng: **một task → một module → một hệ thống → nhiều hệ thống → tổ chức**.

Đây là trục dễ nhìn nhất và cũng dễ nói dối nhất. Phạm vi thật được đo bằng câu hỏi: *nếu bạn ra một quyết định tệ ở đây, bao nhiêu người và bao nhiêu hệ thống phải sống với hậu quả đó, trong bao lâu?* Một người "phụ trách kiến trúc" mà quyết định của họ không ai buộc phải tuân theo thì phạm vi thật của họ bằng không — họ chỉ đang viết tài liệu.

Chú ý: mở rộng phạm vi không đơn điệu tăng theo bậc. Một Senior Engineer sở hữu payment gateway của một ví điện tử có phạm vi *thật* lớn hơn một "Solution Architect" ở một doanh nghiệp truyền thống nơi mọi quyết định kỹ thuật đều do vendor nước ngoài chốt.

### Trục 2 — Độ mơ hồ (Ambiguity)

Chuỗi mở rộng: **bài toán được định nghĩa rõ → bài toán phải tự định nghĩa**.

Ở đầu thấp: "implement API này theo spec". Ở giữa: "làm cho checkout không timeout vào giờ cao điểm" — bài toán rõ nhưng cách làm mở. Ở đầu cao: "trong 18 tháng tới cái gì trong hệ thống của chúng ta sẽ trở thành nút thắt mà hiện tại chưa ai nhìn ra, và ta nên trả giá bao nhiêu ngay bây giờ để tránh nó?"

Trục này là trục phân biệt Senior với Staff, và Staff với Principal, rõ hơn bất kỳ trục nào khác. Nó cũng là trục mà việc "làm tốt hơn" hoàn toàn không giúp gì: giải nhanh gấp đôi một bài toán đã được định nghĩa rõ không đưa bạn tiến một milimet trên trục này.

Cơ chế vì sao trục này khó: bài toán tự định nghĩa không có tiêu chí đúng/sai ngoại sinh. Bạn phải tự tạo ra tiêu chí, rồi tự bảo vệ tiêu chí đó trước những người có tiêu chí khác. Điều này đòi hỏi chịu được trạng thái *không biết mình có đúng không trong nhiều tháng* — một trạng thái mà bộ não đã được huấn luyện bằng compiler và unit test rất khó chịu đựng.

### Trục 3 — Cách tạo ra kết quả (Mode of Impact)

Chuỗi mở rộng: **tự làm → qua người khác → qua cơ chế và cấu trúc**.

- **Tự làm**: bạn viết code, bạn fix bug, kết quả gắn trực tiếp với giờ làm việc của bạn. Đòn bẩy 1x.
- **Qua người khác**: bạn mentor, review, thiết kế trước rồi người khác implement, bạn unblock. Kết quả là hàm của n người. Đòn bẩy n-lần nhưng phụ thuộc vào chất lượng truyền đạt.
- **Qua cơ chế và cấu trúc**: bạn thay đổi cái quyết định *cách mọi người làm việc* — ranh giới team, quy trình review, chuẩn kiến trúc, cách đo lường, cách tuyển. Kết quả tiếp tục xảy ra khi bạn đi nghỉ, và tiếp tục xảy ra sau khi bạn rời công ty. Đòn bẩy lớn nhất, feedback loop chậm nhất, và khả năng gây hại cũng lớn nhất.

Đây là trục bị hiểu sai nhiều nhất, vì người ta gán nó cho nhánh Manager. Không đúng. Một Principal Engineer làm việc chủ yếu qua cơ chế (chuẩn kiến trúc, RFC process, platform mà 200 người dùng) mà không quản lý ai. Một EM tồi làm việc chủ yếu bằng tự làm (nhảy vào code hộ, tự đi họp thay team) và vì thế không tạo ra đòn bẩy nào.

### Ba trục này tăng độc lập — và đó là điều hữu dụng nhất ở đây

| Tổ hợp | Mô tả thực tế | Thường gặp ở đâu |
|---|---|---|
| Scope cao, Ambiguity thấp, Mode = tự làm | "Người biết mọi thứ về hệ thống legacy" — không ai dám chạm vào code đó ngoài anh ta | Doanh nghiệp truyền thống, e-commerce có legacy 8 năm |
| Scope thấp, Ambiguity cao, Mode = tự làm | Người nghiên cứu một bài toán khó chưa ai định nghĩa nhưng chỉ ảnh hưởng một service | R&D team, ML team giai đoạn đầu |
| Scope cao, Ambiguity thấp, Mode = qua người khác | Delivery Manager giỏi: điều phối nhiều team làm những việc đã rõ | ODC, dự án outsourcing lớn |
| Scope cao, Ambiguity cao, Mode = qua cơ chế | Principal Engineer / Director thật | Product company đã scale |

Bảng này giải thích vì sao câu hỏi "tôi đang ở bậc nào" thường không có câu trả lời tốt, còn câu hỏi "trục nào của tôi đang thấp nhất so với bậc tôi muốn" thì luôn có.

### Vì sao số năm kinh nghiệm không phải một trục

Số năm là **biến đại diện (proxy)** cho ba trục trên, và là một proxy tệ. Cơ chế:

- Ba trục trên tăng khi bạn **ra quyết định và sống với hậu quả**. Số năm chỉ đo thời gian ngồi ở đó.
- Ai ở trong môi trường mà quyết định do người khác ra (ODC nơi khách chốt kiến trúc; doanh nghiệp nơi vendor chốt) có thể tích luỹ 8 năm mà trục Ambiguity gần như không nhích.
- Việc lặp lại một chu kỳ 18 tháng năm lần không giống việc trải qua một chu kỳ 7 năm một lần. Cái thứ hai cho bạn thấy hậu quả của quyết định năm thứ hai vào năm thứ sáu — thứ không có cách nào mua được bằng số năm cộng lại.

Cách dùng số năm đúng đắn: như một *cảnh báo âm*. Nếu ai đó có 3 năm kinh nghiệm và tự nhận đang giải bài toán ở phạm vi tổ chức, khả năng cao họ chưa từng thấy hậu quả dài hạn của bất cứ quyết định nào của mình. Nhưng nếu ai đó có 12 năm, điều đó *không* nói lên trục nào của họ đã cao.

### Vì sao title inflation ở Việt Nam làm nhiễu tín hiệu này

Trong giai đoạn thị trường nóng, title là công cụ giữ người rẻ nhất mà công ty có: nâng title không tốn ngân sách như nâng lương, và có tác dụng ngay lập tức lên cảm giác tiến bộ của nhân sự. Hệ quả tích luỹ:

1. **Phân phối title dịch xuống.** "Senior" ở nhiều nơi có nghĩa là 3 năm kinh nghiệm. "Tech Lead" có nghĩa là người trong team làm lâu nhất. "Architect" có nghĩa là người vẽ sơ đồ trong đề xuất bán hàng.
2. **Tín hiệu bị nén.** Khi 70% engineer trên thị trường có title chứa chữ "Senior" (số minh hoạ, không phải số đo), title mất khả năng phân biệt. Nhà tuyển dụng buộc phải suy ra bậc từ nơi khác, và người ứng tuyển không biết phải chứng minh gì.
3. **Người trong nghề tự đo mình bằng thước sai.** Đây là hậu quả đắt nhất. Một người có title Tech Lead ba năm nhưng chưa bao giờ đàm phán scope với PO, chưa bao giờ nói với ai rằng họ đang underperform, chưa bao giờ chịu hậu quả của một quyết định kiến trúc — người đó có ba năm ở trục Scope bằng một Mid-level, và sẽ rất sốc khi đổi sang công ty có bậc thật.

Cách đọc title của người khác (và của chính mình) được xử lý cụ thể ở mục [Bối cảnh Việt Nam](#bối-cảnh-việt-nam).

---

## 1. Junior Engineer

### Hàm mục tiêu ở bậc này

Thành công ở bậc Junior là: **task được giao hoàn thành đúng, và lượng thời gian người khác phải bỏ ra để giúp bạn giảm dần theo tuần.**

Vế thứ hai quan trọng hơn vế thứ nhất và hầu như không bao giờ được nói ra. Một Junior hoàn thành mọi task nhưng mỗi task đều tốn của Senior bốn lần review và hai buổi pair programming đang tạo ra **giá trị ròng âm** cho team — nếu Senior tự làm thì nhanh hơn. Team vẫn giữ Junior đó vì đầu tư vào tương lai, nhưng khoản đầu tư đó phải cho thấy đường cong đi lên. Đường cong đó, không phải số task, là hàm mục tiêu thật.

Khác với bậc trước (không có bậc trước — đây là điểm bắt đầu): điều Junior thường mang từ trường học sang là hàm mục tiêu "làm bài đúng và được điểm cao". Hàm mục tiêu ở công việc khác ở ba chỗ. Một, không có đáp án cuối sách — có nhiều cách đúng và bạn phải chọn. Hai, "đúng" bao gồm cả "người sau đọc được và sửa được", điều mà bài tập ở trường không đo. Ba, tốc độ học của bạn là một phần của kết quả, không phải chuyện riêng của bạn.

### Phạm vi ảnh hưởng

- **Người**: 0 người phụ thuộc vào quyết định của bạn. 1–2 người (mentor, reviewer) bị tiêu thời gian bởi bạn.
- **Hệ thống**: một phần của một module. Thường là những phần đã có khuôn mẫu để copy: thêm một endpoint giống endpoint đã có, thêm một field vào form, viết thêm test cho một hàm đã tồn tại.
- **Chân trời thời gian**: 1 ngày đến 1 sprint. Nếu một task giao cho Junior có chân trời dài hơn hai tuần mà không được chia nhỏ, đó là lỗi của người giao việc, không phải của Junior.
- **Phạm vi rủi ro**: hậu quả của một sai sót được chặn lại bởi Code Review và CI. Đây là đặc điểm định nghĩa của bậc này — Junior làm việc trong một môi trường có lưới an toàn, và lưới đó là cố ý.

Một cách kiểm tra phạm vi thật: hỏi "nếu tôi biến mất hai tuần, cái gì bị chậm?" Ở bậc Junior câu trả lời đúng là "một phần backlog bị chậm, không có gì bị vỡ". Nếu câu trả lời là "không gì cả", bạn đang bị giao việc không quan trọng và cần nói ra. Nếu câu trả lời là "một hệ thống production không ai biết cách vận hành", công ty đang có vấn đề về phân bổ rủi ro, không phải bạn đang được tin tưởng.

### Cách ra quyết định

Loại quyết định thuộc về bậc này rất hẹp, và biết ranh giới đó là một kỹ năng:

| Quyết định | Thuộc về Junior? | Vì sao |
|---|---|---|
| Đặt tên biến, chia hàm, cấu trúc một file | Có | Hậu quả cục bộ, reviewer sửa được với chi phí thấp |
| Chọn dùng library đã có trong project hay viết tay 20 dòng | Có, kèm nêu lý do trong PR | Reversible, chi phí sai thấp |
| Thêm một dependency mới vào project | Không — đề xuất, để Senior/Lead quyết | Ảnh hưởng đến build, security, và mọi người sau |
| Đổi schema của một bảng đang có dữ liệu | Không | Irreversible hoặc rất đắt để đảo ngược |
| Quyết định một edge case xử lý thế nào khi spec không nói | Không — phải hỏi | Đây là quyết định về nghiệp vụ, không phải về code |

Dựa trên gì: **quy ước hiện có của codebase**, không phải sở thích cá nhân hay bài blog đọc tối qua. Ở bậc này, "làm giống những gì đang có" gần như luôn là lựa chọn đúng, kể cả khi cái đang có không phải best practice — vì tính nhất quán của codebase có giá trị cao hơn sự tối ưu cục bộ, và vì bạn chưa có đủ ngữ cảnh để biết vì sao cái đang có lại như vậy.

Ai phản biện: reviewer trong Code Review, và mentor trong buổi 1-1. Đặc điểm ở bậc này là **vòng phản biện rất chặt và rất nhanh** — mỗi PR là một lần phản biện. Đây là món quà lớn nhất của bậc Junior mà người ta chỉ nhận ra khi mất nó: ở bậc Staff, không ai review quyết định của bạn trong hai ngày, và bạn có thể sai suốt sáu tháng mà không ai nói gì.

### Kỹ năng cần phát triển

**1. Đọc code nhanh hơn viết code.** Cần ở bậc này chứ không phải bậc sau, vì đây là bậc duy nhất bạn có lý do chính đáng để dành 60% thời gian đọc mà không ai phàn nàn. Ở bậc Mid trở lên, kỳ vọng về throughput sẽ không cho bạn khoảng trống đó nữa. Kỹ năng cụ thể: theo được một request từ endpoint xuống database và ngược lại, không dùng debugger, chỉ đọc.

**2. Đặt câu hỏi có cấu trúc.** Không phải "kỹ năng giao tiếp" chung chung, mà là một định dạng: *tôi đang cố làm X; tôi đã thử A và B; A ra kết quả này, B ra kết quả này; giả thuyết của tôi là C; câu hỏi cụ thể của tôi là D.* Vì sao cần ở bậc này: chi phí một câu hỏi tồi ở bậc Junior là 20 phút của một Senior, và bạn hỏi 10 lần một tuần. Người không học định dạng này sẽ bị giảm dần số lần được giúp — không ai nói ra, nhưng phản hồi sẽ chậm lại. Người học được sẽ được giúp nhiều hơn và học nhanh hơn, tạo thành vòng tự tăng cường.

**3. Ước lượng thời gian cho task nhỏ và ghi lại độ lệch.** Vì sao ở bậc này: Estimation là kỹ năng duy nhất trong nghề này cải thiện gần như hoàn toàn bằng vòng lặp đo–so sánh, và task nhỏ cho bạn 5–10 vòng lặp mỗi sprint. Ở bậc Senior, mỗi vòng lặp Estimation dài một quý; bạn không có đủ số lần thử để học. Cách làm: trước khi bắt đầu, ghi con số; sau khi xong, ghi con số thật và một câu về nguyên nhân lệch. Sau ba tháng bạn sẽ thấy nguyên nhân lệch của bạn có mẫu — thường là một trong ba: không tính thời gian review, không tính thời gian setup môi trường, hoặc không tính những gì mình chưa biết là mình chưa biết.

**4. Viết được mô tả thay đổi của mình cho người không ngồi cạnh.** PR description, commit message, một dòng trong daily. Vì sao ở bậc này: đây là bản nháp của mọi văn bản kỹ thuật bạn sẽ viết về sau (ADR ở bậc Senior, RFC ở bậc Staff). Người không tập ở kích thước nhỏ sẽ viết RFC 12 trang không ai đọc được ở bậc Staff.

**5. Nhận feedback về code mà không coi đó là feedback về mình.** Vì sao ở bậc này chứ không phải bậc sau: mật độ feedback ở bậc Junior cao nhất trong cả sự nghiệp. Nếu mỗi comment trong PR tiêu tốn của bạn một lượng năng lượng cảm xúc, bạn sẽ hết năng lượng trong sáu tháng, và phản ứng tự nhiên là viết PR nhỏ và an toàn để tránh bị comment — nghĩa là tự khoá trục Scope của mình. Cơ chế đối phó thực tế: phân biệt trong đầu ba loại comment (sai về sự thật / khác về gu / thiếu ngữ cảnh) và trả lời ba loại đó khác nhau.

**6. Biết khi nào dừng lại và báo.** Quy tắc cụ thể có thể áp dụng ngay: nếu bạn bị chặn quá 60–90 phút mà không tiến triển, báo. Vì sao cần ở bậc này: Junior thường sợ bị đánh giá là kém nên tự vật lộn cả ngày. Chi phí thật của việc đó không phải một ngày của bạn — mà là một ngày trong lịch delivery của team mà Tech Lead không biết là đã mất.

### Tín hiệu cho thấy đã sẵn sàng lên bậc tiếp theo

Hành vi quan sát được, không phải cảm giác:

- **Số vòng review trên mỗi PR giảm và giữ ổn định** ở mức 1–2 vòng cho task cùng loại. Không phải PR không có comment — mà comment chuyển từ "chỗ này sai" sang "chỗ này có thể làm khác".
- **Bạn phát hiện ra vấn đề trong task trước khi bắt đầu code.** Ví dụ: đọc ticket và hỏi "spec nói khi user không có địa chỉ thì hiện gì?" — trước khi viết dòng nào. Đây là dấu hiệu sớm nhất và đáng tin nhất.
- **Bạn tự tìm được đường đi trong phần codebase mình chưa từng chạm.** Đo bằng: số lần hỏi "code xử lý X nằm ở đâu" giảm xuống gần 0.
- **Bạn bắt đầu review PR của người khác và phát hiện được thứ có giá trị**, không chỉ format và typo.
- **Bạn làm xong một task và tự thấy phần nào của nó sẽ gây khó cho người bảo trì sau** — và nói ra trong PR description thay vì im lặng.
- **Ước lượng của bạn lệch dưới 50%** với task cùng loại đã làm vài lần. Con số 50% là số minh hoạ để có cái neo, không phải chuẩn ngành.

Tín hiệu **không** đáng tin: cảm giác tự tin, số dòng code, số task đóng trong sprint, việc được khen trong retro. Ba thứ đầu đo throughput ở mức task; thứ tư đo mức độ dễ mến.

### Sai lầm thường gặp

**1. Tự vật lộn quá lâu vì sợ bị đánh giá.** Cơ chế: trong môi trường mà năng lực được đánh giá liên tục và người mới chưa biết ngưỡng nào là "bình thường", việc hỏi bị hiểu (bởi chính người hỏi) là bằng chứng thiếu năng lực. Nghịch lý là người hỏi ít bị đánh giá thấp hơn, vì tiến độ chậm nhìn thấy rõ hơn số lần hỏi. Ở bối cảnh Việt Nam, cơ chế này bị khuếch đại bởi việc ngại làm mất thời gian người lớn tuổi hơn hoặc cấp trên.

**2. Copy giải pháp mà không hiểu, rồi không nói ra.** Cơ chế: giải pháp copy được thường chạy được trong 90% trường hợp, nên nó không bị phát hiện trong review. 10% còn lại nổ ở production sáu tuần sau, khi không ai còn nhớ ngữ cảnh. Cách chữa không phải "đừng copy" — copy là hợp lý — mà là ghi trong PR: "đoạn này tôi lấy từ nguồn X, tôi hiểu phần A, tôi chưa hiểu phần B".

**3. Tối ưu sớm để chứng minh mình biết nhiều.** Cơ chế: Junior mới đọc về design pattern hoặc performance và muốn cho thấy mình không phải người mới. Kết quả là một cache không cần thiết, một abstraction ba lớp cho một use case. Chi phí thật rơi vào người bảo trì, không rơi vào người viết — nên tín hiệu phản hồi không đến được người gây ra.

**4. Coi ticket là biên giới của trách nhiệm.** Cơ chế: hệ thống ticket dạy rằng công việc là đóng ticket. Nên khi thấy một thứ rõ ràng sai ở cạnh task của mình, phản ứng là "cái đó không trong ticket của em". Đây là sai lầm khoá trục Scope mạnh nhất, vì phạm vi ảnh hưởng chỉ mở rộng qua việc bạn *tự nhận* thêm một chút trách nhiệm bên ngoài ranh giới được vẽ. Điều cần làm không phải tự sửa mọi thứ thấy sai — mà là **báo lại**, kèm đánh giá mức độ.

**5. Nhận feedback bằng cách giải thích thay vì bằng cách hỏi.** Cơ chế: bản năng khi bị chỉ ra sai là chứng minh mình có lý do. Việc đó không sai, nhưng khi nó là phản ứng mặc định, reviewer học được rằng góp ý cho bạn tốn công, và họ sẽ góp ý ít đi.

**6. So sánh tốc độ của mình với người có 5 năm kinh nghiệm hơn.** Cơ chế: bạn thấy được đầu ra của họ (nhanh, gọn) nhưng không thấy được 5 năm vòng lặp đã tạo ra nó. Hệ quả hành vi là hoặc tuyệt vọng, hoặc giả vờ biết. Cả hai đều làm chậm việc học.

### Một ngày làm việc điển hình

Thời gian được tiêu chủ yếu vào **một khối liên tục**, và đó là đặc quyền lớn nhất của bậc này.

9:00 — Đọc lại comment trong PR hôm qua, sửa ba chỗ, push. 9:40 — Bắt đầu task mới: đọc ticket, mở codebase tìm chỗ tương tự, mất 40 phút chỉ để hiểu luồng hiện tại. 10:30 — Daily standup 12 phút; nói một câu về task hôm qua, một câu về hôm nay, không nói gì về blocker vì chưa gặp. 10:45 — Viết code. Bị chặn ở việc mock một service bên ngoài trong test; loay hoay 50 phút; nhớ ra quy tắc 90 phút và nhắn mentor. 12:00 — Ăn trưa. 13:30 — Mentor pair 25 phút, chỉ ra rằng project đã có một test helper cho đúng việc đó. 14:00 — Viết code liên tục đến 16:30, gần như không bị gián đoạn. 16:30 — Mở PR, viết description 6 dòng nêu rõ mình đã quyết định gì và chỗ nào chưa chắc. 16:45 — Review PR của một Junior khác, để lại hai comment, một cái là câu hỏi thật. 17:15 — Ghi vào file cá nhân: task này ước lượng 1 ngày, thực tế 1,5 ngày, nguyên nhân lệch là không biết có test helper.

Tổng: khoảng 5,5 giờ làm việc sâu, 1 giờ họp và trao đổi, 0 quyết định có hậu quả ngoài phạm vi module. Số họp trong tuần: 2–3. Số lần bị ngắt: thấp. Đây là cấu trúc thời gian sẽ không bao giờ quay lại sau bậc Senior — điều đáng làm là dùng nó để xây móng, không phải sốt ruột thoát khỏi nó.

---

## 2. Mid-level Engineer

### Hàm mục tiêu ở bậc này

Thành công ở bậc Mid-level là: **một feature được giao cho bạn thì Tech Lead có thể ngừng nghĩ về nó.**

Đây là bậc bị hiểu sai nhiều nhất vì nó không có tên hay. Nó thường bị mô tả là "Junior nhưng nhanh hơn". Không đúng. Cái thay đổi là **đơn vị công việc**: từ *task* sang *feature*. Task có định nghĩa xong rõ ràng do người khác viết. Feature thì không — nó có yêu cầu, và bạn phải tự chia nó thành task, tự phát hiện những phần mà yêu cầu không nói tới, tự quyết định phần nào làm trước.

Khác bậc trước ở ba điểm cụ thể:

- **Junior**: "task này xong chưa?" → **Mid**: "feature này còn thiếu gì để có thể release?"
- **Junior**: được giúp đỡ là bình thường → **Mid**: được giúp đỡ vẫn bình thường, nhưng bạn phải tự biết mình cần giúp ở đâu và hỏi trước khi bị chặn.
- **Junior**: hoàn thành là hết trách nhiệm → **Mid**: bạn là người được gọi khi feature đó lỗi ở production ba tuần sau. Ownership bắt đầu tồn tại ở bậc này.

Vế "Tech Lead có thể ngừng nghĩ về nó" là hàm mục tiêu thật, và nó giải thích một hiện tượng: hai Mid-level có cùng throughput nhưng người được đánh giá cao hơn là người mà Tech Lead không phải kiểm tra. Cái được đo không phải sản lượng, mà là **lượng attention của người khác mà bạn giải phóng**.

### Phạm vi ảnh hưởng

- **Người**: 0–2 người phụ thuộc vào bạn (một Junior đang được bạn hướng dẫn, một QA đang chờ bạn giải thích luồng). Bạn bắt đầu là *người khác phải chờ*, chứ không chỉ là người chờ.
- **Hệ thống**: một module hoặc một feature cắt qua 2–3 module. Bạn hiểu đủ sâu một phần của hệ thống để trả lời câu hỏi "nếu đổi cái này thì cái gì vỡ" mà không cần grep.
- **Chân trời thời gian**: 1 sprint đến 1 quý. Bạn cần giữ trong đầu một feature có 8–15 phần việc trong 3–6 tuần, biết phần nào phụ thuộc phần nào.
- **Phạm vi rủi ro**: lưới an toàn mỏng đi. Code Review vẫn còn, nhưng reviewer bắt đầu tin bạn về những chi tiết họ không kiểm được. Đây là lần đầu bạn có thể gây ra sự cố production mà không ai chặn kịp.

Kiểm tra phạm vi thật: "trong ba tháng qua, có quyết định nào của tôi mà người khác phải sống với hậu quả không?" Nếu không có, bạn đang được giao việc ở mức task và cần chủ động xin việc ở mức feature.

### Cách ra quyết định

| Quyết định | Thuộc về Mid? | Cơ sở |
|---|---|---|
| Chia feature thành các bước implement theo thứ tự nào | Có | Bạn hiểu chi tiết nhất; sai thì chỉ tốn thời gian của bạn |
| Cấu trúc code nội bộ của module mình sở hữu | Có | Reversible, phạm vi cục bộ |
| Chọn có viết integration test hay chỉ unit test cho phần này | Có, kèm nêu rủi ro | Trade-off giữa thời gian và độ tin cậy, ở mức bạn nhìn thấy được |
| Cắt một phần nhỏ của feature để kịp release | Không tự quyết — đề xuất kèm phân tích, PO/Lead chốt | Đây là quyết định về giá trị nghiệp vụ |
| Đổi contract API mà team khác đang dùng | Không | Vượt ranh giới sở hữu; cần Tech Lead hoặc thoả thuận liên team |
| Chấp nhận Technical Debt có ý thức | Có, nếu ghi lại và nêu ra | Trở thành sai lầm chỉ khi không ghi lại |

Dựa trên gì: bắt đầu chuyển từ "quy ước của codebase" sang **hậu quả dự kiến**. Câu hỏi ở bậc Junior là "cái này có giống phần còn lại không?"; câu hỏi ở bậc Mid là "cái này sẽ tạo ra chi phí gì trong sáu tháng tới, và chi phí đó có xứng với thứ tôi tiết kiệm hôm nay không?"

Ai phản biện: reviewer, Tech Lead, và một nhân vật mới — **QA và người vận hành**. Ở bậc này bạn bắt đầu nhận phản biện từ người không đọc code của bạn nhưng phải sống với nó. Loại phản biện này khó nghe hơn vì nó không nói bằng ngôn ngữ kỹ thuật ("cái này dùng thấy khó hiểu"), và nhiều Mid-level bỏ qua nó vì không dịch được sang vấn đề cụ thể. Dịch được là một kỹ năng.

### Kỹ năng cần phát triển

**1. Chia bài toán trung bình thành các phần giao được độc lập.** Vì sao ở bậc này chứ không phải bậc trước: ở bậc Junior bài toán đã được chia sẵn, nên không có cơ hội tập. Vì sao không để đến bậc Senior: ở bậc Senior bạn phải chia bài toán *cho người khác làm*, và bạn không thể học chia việc cho người khác nếu chưa từng chia việc cho chính mình. Kỹ năng cụ thể: chia sao cho mỗi phần có thể merge vào main mà không làm vỡ gì, kể cả khi các phần sau chưa xong. Đây là lý do feature flag và expand-contract migration là kỹ năng của bậc Mid, không phải kỹ năng nâng cao.

**2. Ownership end-to-end: từ yêu cầu tới production tới hậu quả.** Vì sao ở bậc này: đây là bậc đầu tiên bạn còn ở đó khi hậu quả xuất hiện. Cụ thể: bạn phải biết feature của mình có log gì, có metric gì, khi nó lỗi thì ai nhận alert, và khi có người báo lỗi thì tra ở đâu. Nếu bạn không thêm được những thứ đó, bạn chưa sở hữu feature — bạn chỉ viết code cho nó.

**3. Ước lượng cho khối việc nhiều tuần, và giao tiếp độ không chắc chắn.** Vì sao ở bậc này: Estimation ở mức task là bài toán số học; ở mức feature là bài toán về những thứ chưa biết. Kỹ năng cụ thể không phải đoán đúng hơn, mà là **nói được cái gì đang tạo ra khoảng không chắc chắn**: "3 tuần nếu API của đối tác đúng như tài liệu; 5 tuần nếu phải xử lý cả trường hợp họ trả về dữ liệu thiếu, mà tôi chưa test được vì chưa có sandbox". Câu này có giá trị hơn con số "4 tuần" gấp nhiều lần vì nó cho Tech Lead một hành động (đi xin sandbox).

**4. Debug trong hệ thống mình không viết.** Vì sao ở bậc này: Junior debug code của chính mình, có ngữ cảnh đầy đủ trong đầu. Mid phải debug qua ranh giới — một request đi qua ba service, hai trong số đó do team khác viết. Kỹ năng cụ thể: đọc log phân tán, dựng lại timeline, và phân biệt "lỗi ở chỗ tôi" với "lỗi hiện ra ở chỗ tôi". Vì sao không để đến bậc sau: ở bậc Senior bạn sẽ phải dẫn dắt cuộc điều tra sự cố, và không thể dẫn dắt cái bạn chưa từng tự làm.

**5. Viết được văn bản kỹ thuật ngắn có kết luận.** Một trang, ba lựa chọn, một đề xuất, lý do. Vì sao ở bậc này: đây là bậc đầu tiên bạn có quyết định đủ lớn để cần ghi lại nhưng đủ nhỏ để không tốn quá nhiều công. Người tập ở kích thước này sẽ viết được ADR ở bậc Senior. Người bỏ qua sẽ có thói quen quyết định trong đầu và giải thích bằng miệng — thói quen không scale.

**6. Hướng dẫn được một Junior qua một task.** Không phải mentoring toàn diện — chỉ là ngồi cạnh và giúp một người đi qua một task mà không làm hộ. Vì sao ở bậc này: đây là liều thử đầu tiên cho trục "tạo ra kết quả qua người khác". Nó cực nhỏ và vì thế rẻ để thử. Người phát hiện mình không chịu được việc nhìn người khác làm chậm hơn mình sẽ có dữ liệu quan trọng cho quyết định IC vs Manager sau này — sớm hơn nhiều năm.

### Tín hiệu cho thấy đã sẵn sàng lên bậc tiếp theo

- **Bạn phát hiện ra thứ thiếu trong yêu cầu, một cách hệ thống.** Không phải tình cờ thấy một edge case — mà là có thói quen đọc yêu cầu và ra được một danh sách 5–8 câu hỏi trước khi bắt đầu, và đa số câu hỏi đó là câu hỏi PO chưa nghĩ tới.
- **Người khác bắt đầu hỏi bạn về phần hệ thống bạn sở hữu**, và họ hỏi vì bạn trả lời chính xác, không vì bạn là người duy nhất còn ở đó.
- **Bạn nói "không" hoặc "chưa" có kèm phương án.** Ví dụ: "nếu thêm cái này thì release lùi một tuần; hoặc ta bỏ phần X thì giữ được ngày, tôi nghiêng về phương án hai vì X ít user dùng". Đây là tín hiệu mạnh nhất, vì nó thể hiện bạn đã nhìn bài toán từ vị trí của người phải chọn.
- **Bạn chủ động sửa những thứ ngoài task khi chi phí thấp**, và biết dừng khi chi phí không còn thấp — nghĩa là bạn đã có một hàm chi phí trong đầu, không chỉ có nhiệt tình.
- **Sự cố production do phần của bạn giảm dần, và khi có sự cố bạn tự điều tra được** trước khi cần Senior.
- **Bạn đã ít nhất một lần đề xuất một cách làm khác với cách Tech Lead đề xuất, có dữ liệu, và cuộc tranh luận diễn ra ở mức kỹ thuật.** Kể cả khi bạn thua.

Tín hiệu không đáng tin: số năm ở bậc Mid, việc được giao nhiều việc hơn (đó có thể chỉ là dấu hiệu team thiếu người), và việc thành thạo framework mới nhất.

### Sai lầm thường gặp

**1. Nhận hết mọi thứ được đưa tới và không bao giờ nói về đánh đổi.** Cơ chế: ở bậc Mid, "đáng tin cậy" là giá trị được khen nhiều nhất, và cách rẻ nhất để trông đáng tin là đồng ý. Hậu quả xuất hiện chậm: bạn liên tục ở trạng thái quá tải, chất lượng giảm ngầm, và vì không ai nghe bạn nói về đánh đổi nên không ai điều chỉnh kỳ vọng. Trong bối cảnh Việt Nam, cơ chế này mạnh hơn vì việc từ chối yêu cầu từ người cấp trên bị coi là thiếu hợp tác. Cách chữa không phải từ chối — mà là **luôn kèm một câu về cái phải nhường**.

**2. Sở hữu bằng cách độc quyền.** Cơ chế: cách nhanh nhất để trở nên quan trọng là trở thành người duy nhất hiểu một thứ. Nó hiệu quả ngắn hạn (không ai sa thải bạn) và tự hại dài hạn (không ai promote bạn, vì không ai dám lấy bạn ra khỏi chỗ đó). Đây là một trong những cái bẫy khoá bậc phổ biến nhất, và người mắc nó thường không cố ý — họ chỉ đơn giản là không viết tài liệu và không để ai khác chạm vào.

**3. Tưởng rằng "làm nhiều hơn" là đường lên Senior.** Cơ chế: Mid-level thấy Senior làm nhiều thứ khó, nên suy ra rằng cần làm nhiều thứ khó hơn. Nhưng cái phân biệt Senior không phải khối lượng — mà là việc quyết định của họ ảnh hưởng người khác. Một Mid làm việc 55 giờ/tuần vẫn là Mid; một Mid ra một quyết định kiến trúc nhỏ đúng, có tài liệu, mà ba người sau dùng lại, đã bước một chân sang Senior.

**4. Bỏ qua phần vận hành của feature.** Cơ chế: định nghĩa "xong" mà hệ thống ticket dạy bạn là "merge và pass QA". Log, metric, alert, runbook không có trong định nghĩa đó, và không ai bắt lỗi ngay. Chi phí xuất hiện lúc 2 giờ sáng cho người on-call — thường không phải bạn. Đây là ví dụ điển hình của externality: người tạo ra chi phí không phải người trả.

**5. Tranh luận kỹ thuật bằng sở thích thay vì bằng hậu quả.** Cơ chế: ở bậc Mid bạn đã đủ kiến thức để có quan điểm mạnh nhưng chưa đủ trải nghiệm để biết quan điểm nào của mình chưa được kiểm chứng. Biểu hiện: "cái này không clean", "pattern này mới hơn". Cách chữa: mọi lập luận phải kết thúc bằng một câu dạng "nếu làm cách A thì trong tình huống X sẽ tốn Y".

**6. Không dám hỏi lại yêu cầu vì nghĩ PO đã suy nghĩ kỹ.** Cơ chế: khoảng cách vị thế. PO thường lớn tuổi hơn hoặc là founder, và giả định mặc định là họ có ngữ cảnh mà bạn không có. Thực tế PO có ngữ cảnh nghiệp vụ nhưng không có ngữ cảnh về những gì hệ thống làm được. Chi phí của việc im lặng: xây xong một thứ không ai dùng, mất 4 tuần, và không ai kết luận đó là lỗi của bạn — nên vòng lặp không được sửa.

### Một ngày làm việc điển hình

Khối thời gian sâu vẫn còn nhưng bị chia thành hai, và xuất hiện một loại công việc mới: **việc để người khác không bị chặn**.

9:00 — Kiểm tra alert đêm qua liên quan module mình sở hữu; một cảnh báo latency, xem dashboard 15 phút, kết luận là do batch job của team khác, nhắn cho họ kèm link biểu đồ. 9:30 — Standup; nói rõ mình đang chờ sandbox của đối tác, đề nghị Tech Lead push. 9:45 — Viết code phần chính của feature, khối liên tục 2 giờ. 11:45 — Một Junior nhắn hỏi cách test một luồng; ngồi 20 phút, không làm hộ, chỉ vẽ luồng lên giấy và để họ tự viết. 12:15 — Ăn trưa. 13:30 — Họp refinement 45 phút cho sprint sau; đặt bốn câu hỏi về yêu cầu, hai câu làm PO phải mang về hỏi lại business. 14:20 — Viết một trang so sánh hai cách xử lý idempotency cho luồng thanh toán, gửi Tech Lead và một Senior, không chờ họp. 15:00 — Code tiếp, khối 1,5 giờ, bị ngắt hai lần. 16:40 — Review hai PR, một cái để lại comment về việc thiếu xử lý khi response rỗng. 17:10 — Cập nhật ticket với trạng thái thật (không phải "đang làm" mà "đã xong 3/5 phần, phần 4 chờ sandbox").

Tổng: khoảng 4 giờ làm việc sâu, 1,5 giờ họp, 1 giờ giúp người khác và trao đổi bất đồng bộ. Số họp trong tuần: 4–6. Điểm khác biệt so với bậc Junior không phải số họp — mà là bạn bắt đầu **tiêu thời gian để giảm sự không chắc chắn của người khác**, và bắt đầu phải chọn giữa việc code thêm một giờ hay trả lời một câu hỏi giúp người khác tiến được nửa ngày.

---

## 3. Senior Engineer

### Hàm mục tiêu ở bậc này

Thành công ở bậc Senior là: **hệ thống bạn phụ trách vẫn đúng khi bạn không nghĩ về nó nữa, và những người quanh bạn ra quyết định tốt hơn vì có bạn ở đó.**

Hai vế, và vế thứ hai là vế mà đa số Senior ở Việt Nam thiếu. Bậc Senior thường được hiểu là "người giỏi kỹ thuật nhất team" — một định nghĩa đo trục Ambiguity nhưng bỏ hoàn toàn trục Mode of Impact. Một người giải được mọi bài toán khó nhưng team không khá hơn sau hai năm có mặt anh ta là một Senior chưa hoàn chỉnh, và người đó sẽ bị chặn ở bậc này rất lâu mà không hiểu vì sao.

Khác bậc Mid ở ba điểm:

| | Mid-level | Senior |
|---|---|---|
| Đơn vị công việc | Một feature | Một hệ thống hoặc một domain, tồn tại qua nhiều feature |
| Thứ được đo | Feature giao đúng | Hệ thống còn đúng sau 12 tháng và 5 người khác đã sửa nó |
| Quan hệ với sự không chắc chắn | Nêu ra để người khác giải | Tự giảm nó xuống: làm spike, dựng prototype, đo, rồi kết luận |
| Quan hệ với người khác | Nhận giúp đỡ | Là nguồn giúp đỡ, và biết cách giúp mà không làm hộ |

Điểm chuyển quan trọng nhất: ở bậc Mid, bạn được đánh giá bởi những gì bạn làm ra. Ở bậc Senior, bạn bắt đầu được đánh giá bởi **chất lượng phán đoán của bạn** — bao gồm những việc bạn quyết định *không làm*. Một Senior thuyết phục team không xây một hệ thống mà đáng lẽ mất bốn tháng đã tạo ra bốn tháng giá trị, nhưng không có artifact nào để chỉ vào. Đây là lý do bậc Senior đòi hỏi viết: nếu bạn không ghi lại lập luận, đóng góp lớn nhất của bạn sẽ vô hình.

### Phạm vi ảnh hưởng

- **Người**: 3–6 người bị ảnh hưởng bởi quyết định của bạn. Bao gồm những người sẽ vào team sau khi bạn đã quyết — họ chịu hậu quả mà không có tiếng nói.
- **Hệ thống**: một hệ thống hoàn chỉnh (ví dụ: toàn bộ luồng thanh toán của một ví điện tử) hoặc một domain (ví dụ: toàn bộ phần định giá và khuyến mãi của một sàn e-commerce). Bạn biết cả phần code, phần dữ liệu, phần vận hành, và các tích hợp bên ngoài.
- **Chân trời thời gian**: 1 quý đến 1 năm. Bạn phải trả lời được "hệ thống này sẽ chịu được lượng traffic của campaign cuối năm không" vào tháng Sáu.
- **Phạm vi rủi ro**: lưới an toàn gần như hết. Không còn ai review được quyết định kiến trúc của bạn với đủ ngữ cảnh trong team. Đây là điểm chuyển tâm lý lớn nhất của bậc này — bạn phải tự trở thành reviewer của chính mình, và phải tự đi tìm người phản biện thay vì chờ họ đến.

Một kiểm tra cụ thể cho phạm vi thật: **có bao nhiêu quyết định của bạn trong 12 tháng qua mà đến giờ vẫn còn hiệu lực và không thể đảo ngược trong một sprint?** Nếu con số là 0, bạn đang có title Senior nhưng phạm vi Mid.

### Cách ra quyết định

Loại quyết định thuộc về bậc này:

| Loại quyết định | Ví dụ cụ thể | Cơ sở ra quyết định |
|---|---|---|
| Kiến trúc trong một hệ thống | Tách một service hay giữ monolith; dùng event hay gọi trực tiếp | Đo được: throughput cần, độ chịu lỗi cần, số team sẽ chạm vào |
| Mô hình dữ liệu | Schema cho luồng giao dịch, chiến lược migration | Chi phí đảo ngược; dữ liệu là thứ khó sửa nhất |
| Ranh giới với hệ thống khác | Contract API, ai chịu retry, ai chịu idempotency | Đàm phán với Senior của team kia; ghi thành văn bản |
| Đánh đổi chất lượng/thời gian trong phạm vi hệ thống | Phần nào cần test đầy đủ, phần nào chấp nhận rủi ro | Xác suất lỗi × chi phí lỗi, không phải cảm giác |
| Technical Debt: nhận, trả, hay khoanh vùng | Có refactor module này trước khi thêm feature không | Chi phí biên của mỗi thay đổi tương lai |
| Không làm gì | Không xây cache vì bottleneck thật ở chỗ khác | Dữ liệu đo được, không phải suy đoán |

Cơ sở: **bằng chứng, và tài liệu hoá được**. Cụ thể là ADR (Architecture Decision Record) hoặc một RFC ngắn. Sự khác biệt với bậc Mid không phải độ dài văn bản mà là bắt buộc phải có **các phương án đã bị loại và lý do loại**. Vì sao: người đọc ba năm sau cần biết bạn đã cân nhắc gì, để họ biết khi nào điều kiện đã đổi và quyết định nên được xem lại.

Ai phản biện — và đây là phần khó ở bậc này: **bạn phải tự tạo ra người phản biện mình.** Không còn cơ chế nào tự động đưa phản biện đến. Ba nguồn thực tế:

1. **Senior của một team khác** — người có đủ trình độ nhưng không có lợi ích trong quyết định của bạn. Nguồn tốt nhất.
2. **Người phải vận hành hệ thống** — SRE, người on-call. Họ phản biện từ góc bạn ít nghĩ tới.
3. **Chính bạn, ở dạng viết.** Viết ra lập luận và để một ngày rồi đọc lại là hình thức phản biện rẻ nhất và hiệu quả bất ngờ. Cơ chế: khi lập luận còn trong đầu, nó được lưu ở dạng cảm giác về sự hợp lý; khi viết ra, nó buộc phải có bước.

Chống lại một sai lầm phổ biến: ở bậc Senior, việc **hỏi ý kiến người ít kinh nghiệm hơn** không làm giảm uy tín của bạn mà tăng nó, vì nó cho phép họ luyện tập phản biện — thứ họ cần để thành Senior. Nhưng chỉ đúng nếu bạn thực sự đổi quyết định khi họ đúng. Nếu không, đó là nghi lễ, và họ sẽ nhận ra trong vòng hai lần.

### Kỹ năng cần phát triển

**1. Đọc và viết được văn bản kiến trúc có kết luận (ADR / RFC).** Vì sao ở bậc này chứ không bậc trước: ở bậc Mid, quyết định của bạn đủ nhỏ để giải thích bằng miệng; ở bậc Senior, quyết định của bạn sẽ được người chưa gặp bạn đọc lại sau khi bạn rời công ty. Vì sao không để đến bậc Staff: ở bậc Staff bạn phải viết văn bản thuyết phục *người không báo cáo bạn và không đồng ý với bạn* — kỹ năng đó cần một nền tảng đã thành thục về cấu trúc lập luận, và nền tảng đó chỉ có được sau khoảng 10–15 văn bản đã viết và bị phản biện.

**2. Mentoring có mục tiêu, không phải giúp đỡ tuỳ hứng.** Cụ thể: chọn một người, xác định một khoảng cách năng lực cụ thể (ví dụ: "Hà chưa tự thiết kế được schema"), và tạo ra một chuỗi công việc có độ khó tăng dần để lấp khoảng cách đó, kèm review sau mỗi bước. Vì sao ở bậc này: đây là bậc đầu tiên bạn có đủ uy tín để người khác nhận hướng dẫn, và đủ ngữ cảnh để biết khoảng cách năng lực nào là quan trọng. Vì sao đây là kỹ năng bắt buộc chứ không phải tùy chọn: nó là điều kiện cần cho cả hai nhánh phía trước — không có nó, bạn không thể lên Staff (Staff phải nâng năng lực nhiều team) và càng không thể lên EM.

**3. Nhìn thấy chi phí vận hành như một phần của thiết kế.** Vì sao ở bậc này: Mid được đánh giá khi feature release; Senior được đánh giá sau 12 tháng, tức là qua toàn bộ phần đời vận hành của hệ thống. Cụ thể: khi thiết kế, trả lời trước ba câu — hệ thống này sẽ lỗi kiểu gì, ai sẽ biết khi nó lỗi, và người đó cần thông tin gì để sửa trong 15 phút.

**4. Đàm phán ranh giới với team khác.** Vì sao ở bậc này: đây là lần đầu bạn phải đạt được điều gì đó từ người không có nghĩa vụ giúp bạn và không cùng cấp trên với bạn. Kỹ năng cụ thể: đến với một đề xuất đã tính đến lợi ích của họ, không phải với một yêu cầu. Vì sao không để đến sau: toàn bộ bậc Staff được xây trên kỹ năng này ở quy mô lớn hơn.

**5. Điều tra sự cố có phương pháp và dẫn dắt Postmortem.** Vì sao ở bậc này: bạn là người có đủ ngữ cảnh về hệ thống để dựng lại chuỗi nhân quả, và đủ uy tín để giữ cuộc họp ở trạng thái Blameless. Cụ thể: phân biệt trigger với root cause, và luôn hỏi thêm một tầng — "vì sao cơ chế phát hiện của chúng ta không bắt được cái này trước?"

**6. Nói "cái này không nên làm" và bảo vệ được câu đó.** Vì sao ở bậc này: đây là kỹ năng có đòn bẩy cao nhất mà một Senior có, và cũng khó nhất về mặt xã hội. Cơ chế khó: bạn phải bảo vệ một chi phí không xảy ra (bốn tháng tiết kiệm được) chống lại một lợi ích tưởng tượng (feature ai cũng thấy hấp dẫn). Vì sao ở bối cảnh Việt Nam khó hơn: khi đề xuất đến từ founder hoặc từ khách hàng của một ODC, việc phản biện trực diện bị coi là thiếu tôn trọng. Cách thực tế: không phản biện *ý tưởng*, mà đưa ra chi phí và hỏi họ chọn — "cái này làm được, mất khoảng 4 tuần và phải hoãn phần thanh toán định kỳ; anh muốn ưu tiên cái nào trước?"

### Tín hiệu cho thấy đã sẵn sàng lên bậc tiếp theo

Ở bậc này tín hiệu chia làm hai hướng vì phía trước có hai nhánh. Tín hiệu chung trước:

- **Quyết định của bạn được người khác trích dẫn khi bạn không có mặt.** "Theo ADR mà Minh viết thì mình không dùng cách đó." Đây là bằng chứng của ảnh hưởng lan truyền — điều kiện cần cho mọi bậc phía trên.
- **Bạn được hỏi ý kiến về những thứ ngoài hệ thống bạn sở hữu**, bởi người không thuộc team bạn.
- **Có ít nhất một người trong team rõ ràng giỏi hơn nhờ bạn**, và bạn nói được cụ thể họ giỏi hơn ở đâu và bạn đã làm gì.
- **Bạn phát hiện ra một bài toán chưa ai nêu ra, và thuyết phục được người khác rằng đó là bài toán.** Đây là tín hiệu quan trọng nhất cho nhánh IC, vì nó là bài tập nhỏ của trục Ambiguity.
- **Bạn có thể vắng mặt hai tuần và hệ thống của bạn không có sự cố nào cần bạn.**

Tín hiệu nghiêng về nhánh Tech Lead / Manager:

- Bạn thấy mình tự động nghĩ về việc *ai* nên làm việc gì, không chỉ việc gì nên làm.
- Bạn thấy khó chịu khi thấy team làm sai thứ tự việc, hơn là khi thấy code không sạch.
- Khi có xung đột trong team, bạn có xu hướng đi nói chuyện với cả hai bên thay vì tránh.

Tín hiệu nghiêng về nhánh Staff:

- Bạn thấy hứng thú với bài toán cắt qua nhiều team hơn là bài toán về người.
- Bạn viết văn bản và thấy nó thay đổi hành vi của người ở team khác.
- Bạn thấy bức bối vì phạm vi kỹ thuật của mình bị giới hạn ở một team, không vì mình không có quyền.

### Sai lầm thường gặp

**1. Trở thành người không thể thiếu, và coi đó là thành tựu.** Cơ chế: bậc Senior có đủ năng lực để làm mọi thứ nhanh hơn người khác, nên cân bằng ngắn hạn tối ưu luôn là "để tôi làm". Mỗi lần chọn như vậy, khoảng cách năng lực giữa bạn và team giãn thêm, làm lần sau càng hợp lý hơn để bạn tự làm. Đây là một vòng phản hồi dương và nó kết thúc ở chỗ: team không thể vận hành khi bạn nghỉ, và vì thế bạn không thể lên bậc. Dấu hiệu sớm: bạn là người duy nhất trong on-call rotation thực sự xử lý được, hoặc mọi PR quan trọng đều phải bạn approve.

**2. Nghĩ rằng uy tín kỹ thuật tự động chuyển thành ảnh hưởng.** Cơ chế: ở bậc Mid, đúng thì thắng, vì phạm vi tranh luận nhỏ và mọi người cùng đọc code. Ở bậc Senior, bạn phải thuyết phục người không đọc code của bạn — PO, khách hàng, Senior team khác. Người mang nguyên vũ khí "tôi đúng" sang chiến trường đó sẽ đúng và thua. Chi phí: quyết định tệ được thông qua, và bạn kết luận sai rằng "công ty này không lắng nghe kỹ thuật".

**3. Giải quyết vấn đề bằng cách viết thêm code khi vấn đề là vấn đề tổ chức.** Cơ chế: bạn có một cây búa rất tốt. Ví dụ điển hình: hai team liên tục làm vỡ contract của nhau; giải pháp kỹ thuật là thêm một lớp adapter và contract test; giải pháp thật là hai team cần một cuộc trao đổi về ai sở hữu cái gì. Lớp adapter được xây, vấn đề quay lại sau bốn tháng ở hình dạng khác.

**4. Review code như một cổng kiểm soát chất lượng thay vì như một kênh truyền năng lực.** Cơ chế: cách nhanh nhất để đảm bảo chất lượng là chỉ ra mọi lỗi. Nhưng người nhận 20 comment sẽ sửa 20 chỗ và không học được nguyên tắc nào; lần sau họ mắc 20 lỗi khác. Cách tốn công hơn nhưng có đòn bẩy: chọn 3 comment quan trọng nhất, nêu nguyên tắc phía sau, và bỏ qua phần còn lại. Đánh đổi thật: chất lượng của PR này thấp hơn, chất lượng của 20 PR sau cao hơn.

**5. Từ chối làm những việc "không kỹ thuật" và gọi đó là giữ tiêu chuẩn.** Cơ chế: đầu tư vào kỹ năng kỹ thuật có phản hồi nhanh và dễ chịu; đầu tư vào việc viết tài liệu, đi họp với business, làm rõ yêu cầu thì phản hồi chậm và không thoả mãn. Nên người ta hợp lý hoá bằng cách gọi loại việc thứ hai là "không phải việc của engineer". Hậu quả: bạn bị chặn ở bậc Senior vĩnh viễn, vì cả hai nhánh phía trên (Staff và Manager) đều đòi hỏi loại việc thứ hai ở mức nặng hơn.

**6. Đo mình bằng độ khó của bài toán mình giải, không bằng giá trị nó tạo ra.** Cơ chế: cộng đồng kỹ thuật thưởng cho việc giải bài toán khó bằng sự tôn trọng, và điều đó có ích cho việc học. Nhưng ở bậc Senior, việc chọn giải một bài toán khó mà không quan trọng là một dạng thất bại về phán đoán, và nó khó nhìn thấy vì kết quả vẫn ấn tượng. Câu hỏi tự kiểm: trong ba việc khó nhất tôi làm năm nay, việc nào business sẽ nhắc lại sau hai năm?

### Một ngày làm việc điển hình

Khối thời gian sâu bị co lại còn khoảng 3 giờ, và thời gian được tiêu vào một loại việc mới: **giảm sự không chắc chắn cho quyết định của người khác**.

8:45 — Đọc kênh incident đêm qua; không phải hệ thống mình nhưng có liên quan; ghi một câu hỏi để mang vào Postmortem chiều nay. 9:00 — Viết tiếp ADR về việc chuyển luồng đối soát sang xử lý bất đồng bộ; phần khó là mô tả ba phương án đã loại. 10:00 — Standup, rồi ở lại 10 phút với hai Mid để giải quyết một bất đồng về contract giữa hai module — không quyết hộ, chỉ đặt ba câu hỏi để họ tự thấy phương án nào rẻ hơn khi sai. 10:30 — Khối code liên tục: implement phần khó nhất của luồng retry, phần mà không giao được cho ai khác vì rủi ro cao. 12:15 — Ăn trưa. 13:15 — Họp với Senior của team Merchant về ai chịu trách nhiệm idempotency key; kết thúc bằng một đoạn ghi chú hai bên cùng xác nhận, dán vào cả hai kênh. 14:00 — Review 3 PR; trên PR của một Junior chỉ để 3 comment nhưng mỗi comment nêu một nguyên tắc, và nhắn riêng rằng phần còn lại mình để cho lần sau. 14:45 — Postmortem 1 giờ; dẫn phần dựng timeline; đẩy cuộc họp từ "ai deploy" sang "vì sao alert không nổ trước 40 phút". 16:00 — Spike đo thử throughput của một hàng đợi để trả lời câu hỏi capacity cho campaign tháng 11; kết quả là một biểu đồ và ba dòng kết luận gửi Tech Lead. 17:00 — Ngồi 20 phút với một Mid đang chuẩn bị nhận feature lớn đầu tiên, xem cách họ chia việc, chỉnh thứ tự hai phần.

Tổng: khoảng 3 giờ làm việc sâu, 2,5 giờ họp và trao đổi có cấu trúc, 1,5 giờ nâng năng lực người khác và viết. Số họp trong tuần: 6–8. Điểm khác biệt cốt lõi so với bậc Mid: **bạn tiêu thời gian vào những việc mà kết quả không xuất hiện hôm nay** — ADR sẽ có tác dụng sau ba tháng, comment trên PR sẽ có tác dụng sau ba PR, buổi Postmortem sẽ có tác dụng sau sự cố tiếp theo. Đây là lần đầu bạn phải chịu độ trễ giữa hành động và kết quả, và là bài tập chuẩn bị cho mọi bậc phía trên.

---

## 4. Tech Lead

> **Tech Lead là một vai trò (role), không phải một bậc (level).** Đây là điều cần nói rõ trước khi nói bất cứ điều gì khác. Một người có thể là Senior Engineer đang giữ vai Tech Lead cho một team, và sáu tháng sau không còn giữ vai đó nữa mà bậc của họ không đổi. Trong các career ladder công khai của nhiều công ty công nghệ, Tech Lead xuất hiện như một *assignment* gắn với một team và một chu kỳ, không phải một ô trong bảng bậc lương. Hệ quả thực tế rất quan trọng: (1) thôi làm Tech Lead không phải bị giáng cấp; (2) bạn có thể là Staff Engineer mà không bao giờ làm Tech Lead, và ngược lại; (3) khi đàm phán, đừng đổi bậc lấy vai — hãy hỏi vai này gắn với bậc nào.

### Hàm mục tiêu ở bậc này

Thành công ở vai Tech Lead là: **team giao được thứ đã cam kết, với chất lượng kỹ thuật không xấu đi, và không phụ thuộc vào việc bạn tự viết phần khó nhất.**

Đây là lần đầu hàm mục tiêu **không chứa đầu ra cá nhân của bạn**. Ở bậc Senior, "hệ thống của tôi đúng" vẫn là một câu về thứ bạn làm ra. Ở vai Tech Lead, nếu team trễ mà code bạn viết là hoàn hảo, bạn thất bại. Nếu team giao đúng mà bạn không viết dòng nào trong sprint đó, bạn thành công.

Chuyển dịch này khó vì nó đảo ngược một quy tắc bạn đã dùng 5–8 năm: *khi lo lắng, hãy tự làm.* Ở vai Tech Lead, phản ứng "tự làm" là cách nhanh nhất phá hoại chính hàm mục tiêu — vì mỗi giờ bạn code là một giờ bạn không nhìn được cái đang chặn bốn người khác, và cái đang chặn bốn người khác lớn hơn cái bạn viết được trong một giờ.

Ba vế của hàm mục tiêu, theo thứ tự khi phải chọn:

1. **Delivery** — team giao được, và cam kết được đưa ra có cơ sở (chứ không phải "kịp deadline bằng mọi giá").
2. **Chất lượng kỹ thuật** — không tích luỹ Technical Debt ngoài ý thức; những món nợ nhận là nhận có ghi.
3. **Năng lực team** — sau hai quý, số người trong team làm được việc khó tăng lên.

Vế 3 luôn bị hy sinh đầu tiên dưới áp lực, và đó là sai lầm định nghĩa của vai này (xem mục Sai lầm).

### Phạm vi ảnh hưởng

- **Người**: 4–8 người. Đây là con số quan trọng: dưới 4, vai Tech Lead không cần thiết và bạn nên chỉ là Senior; trên 8, bạn không thể vừa nắm kỹ thuật vừa nắm điều phối, và tổ chức đang thiếu một EM.
- **Hệ thống**: toàn bộ phạm vi kỹ thuật của team — thường 2–5 service hoặc một sản phẩm.
- **Chân trời thời gian**: 1 sprint đến 2 quý. Bạn phải giữ trong đầu đồng thời: sprint này ai đang chặn ai, quý này roadmap có khả thi không, và quý sau cần chuẩn bị gì.
- **Thẩm quyền**: đây là đặc điểm định nghĩa và là nguồn của mọi khó khăn — **bạn có trách nhiệm về delivery nhưng không có quyền nhân sự.** Bạn không quyết lương, không quyết promotion, không sa thải được ai, và thường không quyết được ai vào/ra team. Bạn phải đạt được kết quả bằng ảnh hưởng và bằng vai được giao.

Điểm này khiến Tech Lead là vai trò có **tỷ lệ trách nhiệm/quyền hạn lệch nhất** trong toàn bộ tám bậc. Người không nhận ra điều đó sẽ hoặc cố dùng quyền không có (và mất uy tín), hoặc từ chối chịu trách nhiệm (và mất vai).

### Cách ra quyết định

| Loại quyết định | Thuộc Tech Lead? | Ghi chú |
|---|---|---|
| Ai làm phần nào trong sprint | Có | Cân giữa "ai làm nhanh nhất" và "ai cần học phần này" |
| Thứ tự làm các phần của một epic | Có | Dựa trên phụ thuộc kỹ thuật và rủi ro, không dựa trên độ hấp dẫn |
| Chuẩn kỹ thuật nội bộ team (test coverage, branch strategy) | Có, nên quyết cùng team | Áp đặt được nhưng chi phí là sự tuân thủ hình thức |
| Cam kết scope của sprint/release với PO | Có — đây là quyết định trung tâm của vai này | Phải dựa trên capacity thật, gồm cả nợ vận hành |
| Có cắt scope hay lùi deadline khi trễ | Đề xuất và đàm phán; PO/EM chốt | Vai của bạn là làm cho lựa chọn hiện ra rõ, không phải tự chốt |
| Đánh giá hiệu suất, lương, promotion | Không | Nhưng bạn là nguồn input quan trọng nhất, và phải cung cấp input trung thực |
| Kiến trúc xuyên team | Không một mình | Cần phối hợp với Staff/Principal hoặc Tech Lead team khác |

Cơ sở ra quyết định đổi bản chất: từ **bằng chứng kỹ thuật** sang **bằng chứng kỹ thuật cộng với thực tế về người và thời gian**. Ví dụ cụ thể: phương án A tốt hơn về kỹ thuật nhưng chỉ có một người trong team làm được, và người đó đang là điểm nghẽn ở hai việc khác. Phương án B kém hơn 20% nhưng ba người làm được song song. Ở bậc Senior bạn chọn A. Ở vai Tech Lead, B thường đúng, và việc *biết vì sao B đúng* là nội dung chính của vai này.

Ai phản biện bạn: PO (về scope), EM (về cách bạn dùng người), Senior trong team (về kỹ thuật), và — khó nhất — **thực tế delivery của quý trước**. Tech Lead nên có một cơ chế đối chiếu: cuối mỗi sprint, so cam kết với thực tế và ghi nguyên nhân lệch. Không phải để tự trách, mà vì đây là dữ liệu duy nhất giúp cam kết sprint sau chính xác hơn.

### Kỹ năng cần phát triển

**1. Delegation có tính toán — giao việc theo mục tiêu phát triển, không chỉ theo năng lực hiện tại.** Vì sao ở vai này chứ không trước: ở bậc Senior bạn giao việc cho người khác chỉ khi mình quá tải, và tiêu chí là "ai làm được". Ở vai Tech Lead, giao việc là công cụ chính, và tiêu chí có hai chiều: việc này xong đúng hạn, *và* ai lớn lên qua nó. Kỹ năng cụ thể: với mỗi việc, chọn giữa ba mức — giao và tự do hoàn toàn, giao kèm checkpoint giữa, giao kèm thiết kế đã chốt sẵn. Chọn sai mức về phía lỏng thì mất thời gian; chọn sai về phía chặt thì người đó không học được gì và bạn tạo ra sự phụ thuộc.

**2. Đàm phán scope với PO bằng ngôn ngữ chi phí và rủi ro.** Vì sao ở vai này: đây là lần đầu bạn là *người ký* vào cam kết. Cụ thể: không bao giờ trả lời "được" hay "không được" cho một câu hỏi về deadline. Trả lời bằng một tập lựa chọn có giá. Kịch bản ba lựa chọn (đủ scope + lùi 2 tuần / đúng ngày + bỏ phần X / đúng ngày + đủ scope nhưng nhận nợ Y và trả trong quý sau) là công cụ dùng được ngay. Vì sao quan trọng ở bối cảnh Việt Nam: trong ODC, người đưa deadline là khách hàng nước ngoài, và việc đàm phán phải đi qua Account Manager. Tech Lead không luyện được kỹ năng biến ràng buộc kỹ thuật thành ngôn ngữ chi phí sẽ bị biến thành người truyền tin, và team sẽ nhận scope không khả thi mỗi quý.

**3. Nhìn thấy điểm nghẽn của team, không phải điểm nghẽn của mình.** Vì sao ở vai này: đầu ra của team là hàm của luồng công việc, không phải tổng năng suất cá nhân. Mental model dùng được là Theory of Constraints: throughput của một hệ thống bị quyết định bởi ràng buộc chật nhất, nên tối ưu mọi chỗ khác là vô nghĩa. Cụ thể ở một team: nếu mọi PR đều chờ bạn review 8 tiếng, ràng buộc là bạn, và việc bạn viết code nhanh hơn làm tình hình xấu hơn. Kỹ năng: mỗi tuần trả lời một câu — "tuần này cái gì làm chậm nhất luồng việc của team, và tôi có phải nó không?"

**4. Chạy các cuộc trao đổi kỹ thuật đến được kết luận.** Vì sao ở vai này: bạn là người chịu chi phí khi một cuộc tranh luận kiến trúc kéo ba tuần không chốt. Kỹ năng cụ thể: trước mỗi cuộc trao đổi, tuyên bố rõ nó thuộc loại nào — thu thập thông tin, tạo phương án, hay chốt. Và khi chốt: nếu không đạt đồng thuận sau một khung thời gian đã định, bạn quyết, ghi lại lý do, và nói rõ điều kiện nào sẽ khiến quyết định được xem lại. Không chốt là lựa chọn tệ nhất, vì nó chuyển chi phí sang người đang chờ.

**5. Cung cấp Feedback về hiệu suất mà không có thẩm quyền.** Vì sao ở vai này: bạn thấy rõ nhất ai đang chững lại, nhưng bạn không phải người đánh giá họ. Cụ thể: feedback phải mô tả hành vi và hậu quả, không mô tả tính cách, và phải được nói trực tiếp với người đó *trước khi* nói với EM. Vì sao ở bối cảnh Việt Nam khó: văn hoá tránh xung đột trực diện làm feedback bị pha loãng đến mức người nhận không hiểu là feedback. Kiểm tra: sau buổi nói chuyện, người đó có nói lại được cụ thể cần thay đổi gì không?

**6. Giữ được đủ độ sâu kỹ thuật để phán đoán còn giá trị.** Vì sao đây là kỹ năng chứ không tự nhiên có: thời gian code giảm xuống 20–30%, và độ sâu sẽ tự rơi nếu không có chủ ý. Cụ thể, cách giữ hiệu quả nhất không phải viết feature (bạn sẽ thành điểm nghẽn) mà là: đọc mọi thay đổi quan trọng, tự làm những việc nhỏ có tính khám phá (spike, prototype, đọc log sự cố), và làm những phần không nằm trên đường tới hạn của sprint.

### Tín hiệu cho thấy đã sẵn sàng lên bậc tiếp theo

Từ vai Tech Lead, "bậc tiếp theo" có hai hướng. Tín hiệu chung:

- **Team giao được hai quý liên tiếp với cam kết do chính team đưa ra**, và độ lệch giữa cam kết và thực tế nằm trong khoảng dự đoán được.
- **Bạn nghỉ một tuần và sprint không lệch.** Đây là tín hiệu mạnh nhất và cũng khó chịu nhất, vì nó chứng minh bạn không cần thiết trong ngắn hạn — điều đó là mục đích.
- **Có người trong team giờ có thể giữ vai Tech Lead**, và bạn nói được tên.
- **Bạn bắt đầu thấy các vấn đề *không giải được ở trong team*** — ví dụ: team liên tục trễ vì chờ team Platform, và cấu trúc phụ thuộc đó không ai sở hữu. Việc bạn nhìn ra loại vấn đề này là cửa vào bậc Staff.

Nghiêng về nhánh **Staff**: bạn thấy mình hứng thú với việc gỡ những phụ thuộc kiến trúc xuyên team hơn là với việc điều phối; bạn viết một tài liệu và team khác đổi cách làm; bạn thấy phần "quản lý người" là thuế phải trả, không phải phần có ý nghĩa.

Nghiêng về nhánh **Engineering Manager**: bạn thấy phần buổi 1-1 là phần có ý nghĩa nhất trong tuần; bạn đã thực sự giúp một người kém đi lên hoặc đã dẫn một cuộc nói chuyện khó về hiệu suất; bạn thấy bức bối vì không có quyền để giải quyết những vấn đề về người mà bạn nhìn rõ.

### Sai lầm thường gặp

**1. Tự nhận phần khó nhất của mỗi sprint.** Cơ chế: hợp lý ở mọi mức cục bộ — bạn làm nhanh nhất, rủi ro thấp nhất, và cảm giác hoàn thành rất tốt. Nhưng nó tạo ba hậu quả tích luỹ: bạn thành đường tới hạn của mỗi sprint; không ai trong team lớn lên vào phần khó; và bạn không còn thời gian nhìn tổng thể nên phát hiện vấn đề chậm. Dấu hiệu sớm: bạn code vào buổi tối để bù cho việc ban ngày phải họp.

**2. Trở thành người truyền tin giữa PO và team.** Cơ chế: cách nhanh nhất giảm gián đoạn cho team là bạn nhận hết trao đổi với PO. Ngắn hạn team tập trung hơn. Dài hạn: team mất ngữ cảnh nghiệp vụ nên ra quyết định kỹ thuật kém, mọi thông tin phải đi qua bạn nên bạn thành điểm nghẽn, và không ai trong team phát triển được kỹ năng làm việc với stakeholder. Cách chữa: đưa hai người trong team vào các buổi refinement, và để họ trả lời trực tiếp những câu hỏi thuộc phần họ sở hữu.

**3. Nhận cam kết vượt capacity vì không muốn nói không.** Cơ chế: chi phí của việc nói "không kịp" đến ngay lập tức và rơi vào bạn (mặt PO, sự thất vọng, cảm giác kém cỏi). Chi phí của việc nhận rồi trễ đến sau ba tuần và bị phân tán (team overtime, chất lượng giảm, ai cũng chịu một phần). Con người chọn chi phí bị trì hoãn và phân tán — một thiên lệch có tên là temporal discounting. Cách chữa cơ chế, không chữa ý chí: đưa capacity thành số công khai trước mỗi sprint, để việc từ chối là kết luận số học chứ không phải quan điểm cá nhân.

**4. Áp chuẩn kỹ thuật bằng thẩm quyền vai trò.** Cơ chế: bạn được giao vai nên bạn tưởng mình có quyền quyết chuẩn. Về hình thức thì có. Nhưng chuẩn được tuân thủ bằng sự đồng ý, không bằng lệnh — người không đồng ý sẽ tuân thủ theo chữ và phá theo tinh thần (viết test cho đủ coverage nhưng test không assert gì). Dấu hiệu sớm: coverage đạt chuẩn mà bug vẫn nhiều như cũ.

**5. Hy sinh vế "năng lực team" mỗi khi có áp lực.** Cơ chế: trong ba vế của hàm mục tiêu, chỉ vế delivery được đo hàng sprint. Nên khi áp lực tăng, việc giao việc khó cho người cần học là thứ đầu tiên bị bỏ. Vấn đề: áp lực không bao giờ hết, nên "tạm thời" thành vĩnh viễn, và sau bốn quý team có cùng phân bố năng lực như bốn quý trước — nghĩa là bạn đã tiêu bốn quý mà không tạo ra tài sản nào.

**6. Coi vai Tech Lead là bậc thang bắt buộc và nhận nó khi chưa muốn.** Cơ chế: ở nhiều công ty Việt, Tech Lead là dấu hiệu duy nhất của sự tiến bộ sau Senior, vì không có bậc Staff thật. Nên người ta nhận vai để không bị coi là chững lại. Hậu quả: một Senior xuất sắc trở thành một Tech Lead trung bình, mất một năm, và cả team chịu phần thiệt. Cách phòng: hỏi thẳng công ty rằng nếu sau hai quý bạn muốn quay lại làm IC thì có đường về không — và nghe cả cách họ phản ứng với câu hỏi, không chỉ nội dung trả lời.

### Một ngày làm việc điển hình

Thời gian bị **phân mảnh** — đây là đặc điểm định nghĩa. Bạn hầu như không còn khối 2 giờ liên tục nào trong ngày làm việc, và cố tạo ra khối đó bằng cách bỏ qua tín hiệu từ team là cách phổ biến để thất bại ở vai này.

8:40 — Xem board trước standup; thấy hai ticket đứng nguyên hai ngày; ghi lại để hỏi. 9:00 — Standup 15 phút; sau đó giữ lại 10 phút với người đang tắc ở tích hợp SDK đối tác, quyết định cho họ 4 giờ nữa rồi đổi hướng sang mock. 9:30 — Nhắn team Platform xin ưu tiên cho một ticket đang chặn team mình; kèm ngữ cảnh chi phí, không kèm áp lực. 9:45 — Buổi 1-1 với một Mid: nghe họ nói về việc muốn làm phần kiến trúc nhiều hơn; thống nhất giao họ thiết kế module thông báo trong sprint sau, có review giữa. 10:30 — Review 4 PR, ưu tiên cái đang chặn người khác merge. 11:15 — Họp với PO về scope release tháng sau; mang theo ba phương án có giá; PO chọn phương án cắt phần báo cáo. 12:00 — Ăn trưa. 13:00 — Khối kỹ thuật duy nhất trong ngày: 90 phút làm một spike đo thử độ trễ của luồng mới, không nằm trên đường tới hạn của sprint. 14:30 — Hai người bất đồng về việc có nên dùng thêm một hàng đợi; ngồi 30 phút, đặt câu hỏi "nếu sai thì cái nào sửa rẻ hơn", họ tự chốt. 15:00 — Viết cập nhật tuần cho EM và PO: tiến độ, hai rủi ro, một quyết định cần họ. Khoảng 12 dòng. 15:30 — Thấy một Junior đang đi sai hướng ba ngày liền không báo; ngồi lại, không sửa hộ, chỉ định nghĩa lại bài toán nhỏ hơn. 16:15 — Cập nhật kế hoạch capacity cho sprint sau, tính cả một người sắp nghỉ phép và nợ vận hành. 17:00 — Đọc lại các thay đổi quan trọng đã merge trong ngày, không comment, chỉ để giữ ngữ cảnh.

Tổng: khoảng 1,5 giờ làm việc kỹ thuật sâu, 3 giờ trao đổi và họp, 2 giờ unblock và phát triển người, 0,5 giờ viết. Số họp trong tuần: 10–14. Điểm khác biệt cốt lõi so với bậc Senior: **giá trị của một giờ của bạn giờ đây phụ thuộc vào việc giờ đó có tháo được nút thắt nào không** — và những nút thắt lớn nhất thường xuất hiện dưới dạng một tin nhắn ngắn từ người đang bí, không phải dưới dạng một task trên board.

---

## 5. Staff Engineer

> **Staff là một bậc (level), Tech Lead là một vai (role).** Tech Lead gắn với delivery của một team cụ thể trong một chu kỳ cụ thể; hết chu kỳ, hết team, hết vai. Staff gắn với **phạm vi kỹ thuật** bạn có thẩm quyền phán đoán trên đó, và phạm vi đó không biến mất khi tổ chức đổi ranh giới team. Hệ quả thực tế: một Staff Engineer có thể đang giữ vai Tech Lead cho một team, hoặc không giữ vai nào, mà bậc không đổi. Và câu hỏi đúng khi ai đó nói "tôi muốn lên Staff" không phải "bạn muốn dẫn team nào" mà là **"bài toán nào bạn muốn được coi là người chịu trách nhiệm phán đoán về nó"**.

### Hàm mục tiêu ở bậc này

Thành công ở bậc Staff là: **chất lượng của các quyết định kỹ thuật trong lĩnh vực bạn phụ trách tăng lên, kể cả những quyết định do người khác ra và bạn không có mặt.**

Đây là lần đầu hàm mục tiêu nói về **quyết định của người khác**. Ở vai Tech Lead, bạn còn đo được bằng delivery của một team trong một sprint. Ở bậc Staff, thứ bạn tạo ra là: một ràng buộc được viết ra làm ba team ngừng đi vào một hướng sai; một chuẩn mà năm team tự nguyện dùng vì nó tiết kiệm việc cho họ; một bài toán được định nghĩa lại nên thay vì làm sáu tháng thì chỉ cần ba tuần.

Ba đầu ra cụ thể, xếp theo độ khó tăng dần:

1. **Quyết định kiến trúc có tài liệu** — ADR/RFC mà người đọc sáu tháng sau hiểu được vì sao chọn thế, và điều kiện nào làm quyết định đó hết hiệu lực.
2. **Chuẩn được các team tự nguyện dùng** — thư viện, template, contract, quy ước. Tiêu chí kiểm tra tàn nhẫn: có team nào dùng nó mà bạn không phải đi thuyết phục không?
3. **Nâng chất lượng quyết định của người khác** — sau khi bạn tham gia một cuộc thiết kế, những người trong đó lần sau tự đặt được câu hỏi mà trước đây bạn phải đặt.

Vế 3 là vế phân biệt Staff thật với "Senior lâu năm". Nó cũng là vế duy nhất không ai đo được bằng công cụ nào, nên nó là vế bị bỏ đầu tiên.

### Phạm vi ảnh hưởng

- **Bài toán, không phải team.** Đây là đặc điểm định nghĩa. Bạn gắn với một lĩnh vực: hệ thống thanh toán và đối soát, tầng dữ liệu, độ tin cậy của luồng checkout, nền tảng tích hợp đối tác. Lĩnh vực đó cắt qua 2–5 team và không team nào sở hữu trọn vẹn — đó chính là lý do bậc này tồn tại.
- **Không có quyền nhân sự, và cũng không có cả thẩm quyền vai trò như Tech Lead.** Tech Lead ít nhất được tổ chức tuyên bố là người chịu trách nhiệm delivery của một team. Staff thường không được tuyên bố gì. Công cụ của bạn là ba thứ: **văn bản, chuẩn, và công cụ** — thứ hoạt động khi bạn không có mặt và không cần ai phải đồng ý với bạn trước mặt bạn.
- **Chân trời thời gian**: 2 quý đến 2 năm. Bạn phải chịu được việc một quyết định hôm nay chỉ chứng minh được đúng hay sai vào năm sau.
- **Hậu quả khi sai**: 2–5 team phải sống với nó trong nhiều quý. Đây là phép thử phạm vi thật: nếu một quyết định sai của bạn chỉ ảnh hưởng một service của một team, bạn đang làm việc ở phạm vi Senior với title Staff.

### Bốn archetype của Staff+

Trong tài liệu công khai của Will Larson (sách *Staff Engineer* và trang staffeng.com — **nguồn công khai**, không phải khung nội bộ của công ty nào), bậc Staff+ được mô tả bằng bốn archetype. Việc biết mình thuộc archetype nào không phải để dán nhãn, mà để trả lời ba câu rất thực dụng: *đầu ra của tôi trông như thế nào, tôi nên dành thời gian vào đâu, và tôi nên được đánh giá bằng gì.*

| Archetype | Cách tạo ra kết quả | Đầu ra trông như thế nào | Rủi ro riêng của archetype này |
|---|---|---|---|
| **Tech Lead** | Dẫn một team qua bài toán kỹ thuật khó, thường gắn với delivery | Team giao được thứ trước đó chưa ai biết cách làm | Bị kéo hết thời gian vào một team, mất phạm vi xuyên team |
| **Architect** | Sở hữu định hướng kỹ thuật của một lĩnh vực trong nhiều năm | Chuẩn, ràng buộc, lộ trình migration được thực thi | Thành kiến trúc trên giấy, không chịu hậu quả vận hành |
| **Solver** | Được điều vào bài toán khó nhất đang cháy, giải, rồi đi | Một lớp vấn đề bị dập tắt, không tái phát | Đi mãi mà không để lại năng lực; tổ chức phụ thuộc bạn |
| **Right Hand** | Làm việc sát một leader, mở rộng khả năng ra quyết định của tầng đó | Quyết định của tổ chức tốt hơn, nhanh hơn | Mất tiếp xúc kỹ thuật; uy tín gắn với một người |

Điều nguy hiểm nhất là **tổ chức kỳ vọng bạn ở archetype A trong khi bạn đang tối ưu cho archetype B**. Ví dụ: Linh được lên Staff với kỳ vọng Architect cho tầng dữ liệu, nhưng cô làm việc như Solver — quý nào cũng có ba sự cố hay để cô nhảy vào, và cô giải rất giỏi. Cuối năm, mọi sự cố đều đã được dập, nhưng không có một quyết định nào về tầng dữ liệu được viết ra, và ba team vẫn tự thiết kế schema theo cách riêng. Đánh giá của cô kém, cô không hiểu vì sao. Cách phòng: hỏi thẳng manager và một Principal câu này mỗi hai quý — *"trong sáu tháng tới, đầu ra nào của tôi mà nếu không có thì các anh coi là tôi đã thất bại?"*

### Cách ra quyết định

Cơ sở ra quyết định đổi từ **bằng chứng kỹ thuật** sang **bằng chứng kỹ thuật viết ra được cho người không cùng ngữ cảnh**. Ở bậc Senior, bạn thuyết phục bốn người ngồi cùng phòng, cùng biết hệ thống. Ở bậc Staff, bạn phải thuyết phục một Tech Lead ở team khác, người có backlog riêng, KPI riêng, và không có lý do cấu trúc nào để nghe bạn.

| Loại quyết định | Thuộc Staff? | Ghi chú |
|---|---|---|
| Chuẩn kỹ thuật xuyên team (contract, versioning, quy ước dữ liệu) | Có, là đầu ra trung tâm | Chỉ có giá trị nếu có ít nhất một team dùng mà không bị ép |
| Kiến trúc của một lĩnh vực cắt qua nhiều team | Có, và phải viết lại | Kèm điều kiện xem lại, kèm chi phí migration |
| Chọn giữa hai hướng khi hai team đang bất đồng | Thường là bạn định khung; EM/Director chốt nếu liên quan nguồn lực | Nếu bạn tự chốt bằng uy tín, bạn thắng lần này và mất lần sau |
| Kiến trúc nội bộ một team | Không | Đây là của Tech Lead team đó. Can thiệp vào đây là cách nhanh nhất mất đồng thuận |
| Đánh đổi scope/deadline của một team | Không | Nhưng bạn phải nói rõ chi phí kỹ thuật để người quyết biết mình đang trả gì |
| Thứ tự ưu tiên trong roadmap | Không, nhưng bạn là nguồn input về rủi ro kỹ thuật | Input phải kèm định lượng, không kèm cảm giác |

Ai phản biện bạn: các Tech Lead của những team chịu ảnh hưởng, Principal (nếu tổ chức có), và **thực tế vận hành 6–18 tháng sau**. Cơ chế đối chiếu bắt buộc ở bậc này: mỗi quyết định kiến trúc lớn phải ghi kèm một dự đoán kiểm chứng được ("sau khi migration, độ trễ p99 của luồng đối soát giảm dưới 400ms và số ca lệch số cuối ngày về 0 — *số minh hoạ*"), rồi quay lại đọc nó. Không có cơ chế này, bạn sẽ tích luỹ uy tín thay vì tích luỹ phán đoán, và hai thứ đó tách nhau ra rất nhanh.

### Kỹ năng cần phát triển

**1. Viết để thuyết phục người không có động lực đọc bạn.** Vì sao ở bậc này chứ không trước: ở các bậc dưới, ảnh hưởng của bạn đi qua sự hiện diện — bạn ngồi cạnh, bạn giải thích, bạn review. Ở bậc Staff, phạm vi vượt quá số cuộc họp bạn dự được, nên **văn bản là phương tiện mở rộng duy nhất**. Cụ thể: một RFC hiệu quả bắt đầu bằng vấn đề mà người đọc đang chịu, không bắt đầu bằng giải pháp bạn thích; có mục "phương án đã cân nhắc và vì sao loại"; có mục "cái này làm bạn mất gì". Kiểm tra: đưa tài liệu cho một người không dự cuộc họp nào, họ tóm lại được quyết định và lý do không?

**2. Xây đồng thuận không qua thẩm quyền.** Cụ thể là một quy trình, không phải một phẩm chất: (a) nói riêng với 3–4 người có quyền phá quyết định *trước khi* đưa ra bàn chung, hỏi họ mất gì; (b) sửa phương án theo phản hồi thật, để họ thấy dấu vết của mình trong đó; (c) khi ra bàn chung, đã có người đồng ý nói trước bạn. Vì sao ở bối cảnh Việt Nam khâu (a) đặc biệt quan trọng: văn hoá tránh xung đột trực diện làm người ta không phản đối trong cuộc họp — họ đồng ý ở đó rồi không thực thi. Sự im lặng trong họp không phải đồng thuận; nó thường là chi phí bị hoãn.

**3. Chọn bài toán.** Đây là kỹ năng đặc trưng nhất của bậc này và cũng khó nhất, vì nó đòi hỏi từ chối những bài toán thú vị. Ở các bậc dưới, bài toán được đưa cho bạn. Ở bậc Staff, bạn có 60–70% thời gian tự định hướng, và **cách bạn tiêu phần đó chính là toàn bộ đóng góp của bạn**. Công cụ dùng được: mỗi quý viết ra ba bài toán trong lĩnh vực của mình, kèm ba con số ước lượng — chi phí nếu không làm, số team được lợi, và ai khác có thể làm được nó. Bài toán mà *chỉ bạn* làm được và *không ai khác* đang nhìn là ứng viên tốt nhất. Bài toán mà một Senior giỏi cũng làm được thì đưa cho họ.

**4. Chịu hậu quả vận hành của kiến trúc mình đặt ra.** Vì sao đây là kỹ năng: nó phải được thiết kế vào cách bạn làm việc, không tự nhiên xảy ra. Cụ thể: ở trong ca on-call của hệ thống mình thiết kế, đọc postmortem của các sự cố liên quan, tự làm phần khó nhất của bước migration đầu tiên chứ không giao. Cơ chế đằng sau: kiến trúc là một tập giả định về chi phí vận hành, và cách duy nhất kiểm chứng giả định đó là trả chi phí.

**5. Nâng chất lượng phán đoán của người khác một cách có chủ ý.** Cụ thể, ba thao tác lặp lại được: trong design review, đặt câu hỏi thay vì đưa kết luận ("nếu lưu lượng gấp mười thì chỗ nào vỡ trước?"); sau đó nói rõ *vì sao mình đặt câu hỏi đó*, để lần sau họ tự đặt; và giao cho một Senior viết RFC mà bạn chỉ review. Kiểm tra sau hai quý: có tài liệu kiến trúc nào trong lĩnh vực của bạn mà tác giả không phải bạn?

### Tín hiệu cho thấy đã sẵn sàng lên bậc tiếp theo

- **Một chuẩn hoặc công cụ do bạn đưa ra đang được dùng ở team mà bạn chưa từng làm việc cùng**, và họ dùng vì nó tiện, không vì ai yêu cầu.
- **Bạn được gọi vào các cuộc thảo luận trước khi quyết định được đưa ra**, không phải sau — kể cả bởi những người không thuộc lĩnh vực của bạn. Đây là tín hiệu tổ chức đã coi phán đoán của bạn là đầu vào, không phải là kiểm duyệt.
- **Bạn bắt đầu phát hiện những thứ chưa ai coi là bài toán.** Không phải "cái này khó", mà "trong 18 tháng, cách chúng ta đang gắn dữ liệu khách hàng vào từng service sẽ làm mọi thay đổi compliance tốn gấp năm lần, và hiện chưa ai gọi đó là vấn đề". Đây là cửa vào bậc Principal.
- **Những quyết định của bạn 12–18 tháng trước đã được thực tế phán xử**, và bạn nói được cái nào sai, sai vì giả định nào.
- **Người khác trong lĩnh vực của bạn đã tự ra được những quyết định mà trước đây phải chờ bạn.**

### Sai lầm thường gặp

**1. Trở thành người viết code nhiều nhất.** Cơ chế: đây là thứ bạn giỏi nhất, phản hồi tức thì, và không ai phản đối. Nhưng phạm vi của code bạn viết bằng đúng một người, còn phạm vi của một quyết định kiến trúc bằng năm team trong hai năm. Bạn đang tiêu tài sản đắt nhất của mình vào việc có đòn bẩy 1x. Dấu hiệu sớm: quý này bạn là người merge nhiều commit nhất trong lĩnh vực của mình, và không có tài liệu quyết định nào được viết.

**2. Làm architect trên giấy, không chịu hậu quả vận hành.** Cơ chế: bạn vẽ sơ đồ, viết chuẩn, rồi chuyển sang bài toán khác trước khi ai đó phải triển khai nó. Không có vòng phản hồi nào chạm tới bạn, nên các giả định sai của bạn không bao giờ bị sửa — và uy tín của bạn tiếp tục tăng vì không ai truy được hậu quả về nguồn. Dấu hiệu sớm: các team nói "làm theo đúng thiết kế thì không chạy được, nhưng thôi tự xử". Cách chữa mang tính cơ chế: bạn phải ở trong nhóm on-call của thứ mình thiết kế trong ít nhất một quý sau khi nó lên production.

**3. Đi giải mọi bài toán thú vị thay vì bài toán quan trọng nhất.** Cơ chế: bậc Staff cho bạn quyền tự chọn việc, và bộ não chọn theo độ hấp dẫn kỹ thuật chứ không theo chi phí tổ chức. Kết quả sau một năm: mười thứ hay được sửa, không thứ nào là nút thắt. Đây là dạng thất bại êm ái nhất ở bậc này, vì mỗi việc riêng lẻ đều có ích và không ai chỉ ra được lỗi. Cách chữa: mỗi quý chỉ có **một** bài toán được gọi là bài toán chính, viết ra, và cho manager cùng một Tech Lead xem — người ngoài phát hiện sự lệch hướng nhanh hơn bạn.

**4. Dùng bậc như thẩm quyền.** Câu "tôi là Staff nên tôi quyết" phá đúng thứ tạo nên bậc này. Ảnh hưởng không có thẩm quyền hoạt động vì người khác tin phán đoán của bạn; khi bạn viện dẫn bậc, bạn đang nói rằng phán đoán không đủ.

**5. Can thiệp vào kiến trúc nội bộ của một team.** Cơ chế: bạn thấy rõ chỗ họ làm dở, và bạn đúng. Nhưng bạn vừa lấy đi quyền quyết định của Tech Lead team đó trước mặt team họ. Lần sau họ sẽ không mời bạn. Ranh giới dùng được: bạn có tiếng nói ở những gì **vượt ra khỏi ranh giới team** (contract, dữ liệu chia sẻ, rủi ro lan sang team khác); những gì nằm hoàn toàn trong team là của họ, kể cả khi họ chọn dở.

**6. Không có ai kế nhiệm trong lĩnh vực của mình.** Nếu sau hai năm bạn vẫn là người duy nhất hiểu tầng đối soát, bạn đã tạo ra một điểm chết cho tổ chức và đồng thời tự khoá mình lại — bạn không thể chuyển sang bài toán mới vì không ai giữ được cái cũ.

### Một ngày làm việc điển hình

Điểm khác biệt lớn nhất so với vai Tech Lead: **thời gian không bị phân mảnh bởi nhịp sprint, mà bị phân mảnh bởi các cuộc trao đổi bạn tự chọn**. Bạn có khối liên tục — và chính vì có, việc tiêu nó vào đâu trở thành phép thử.

8:45 — Đọc kênh sự cố đêm qua: một lỗi lệch số ở đối soát, không nghiêm trọng, nhưng là lần thứ ba trong quý. Ghi lại như một tín hiệu, không nhảy vào sửa. 9:00 — Khối viết 90 phút: RFC về việc chuyển đối soát sang mô hình event-sourced, mục "chi phí migration cho từng team" — mục khó nhất và là mục quyết định RFC có được thực thi hay không. 10:30 — Nói riêng với Tuấn (Tech Lead team Payment) 30 phút: đưa bản nháp, hỏi thẳng "cái này làm team em mất gì trong quý sau"; nhận ra chi phí lớn hơn ước lượng của mình 40% (*số minh hoạ*), sửa lại phần lộ trình. 11:15 — Design review của team khác, tham gia với tư cách phản biện: đặt hai câu hỏi về chuyện gì xảy ra khi đối tác trả lời chậm 30 giây, không đưa giải pháp. 12:00 — Ăn trưa với Khoa, một Senior đang muốn lên Staff: nghe cậu ấy trình bày ba bài toán, chỉ ra rằng cả ba đều là bài toán đã được người khác định nghĩa. 13:30 — Ca on-call: đọc trace của lỗi đối soát sáng nay, xác nhận nó là hệ quả của một giả định trong thiết kế cũ *của mình* hai năm trước; ghi vào RFC như bằng chứng. 15:00 — 45 phút với Hà (EM) và Trang (Head of Product): giải thích bằng ngôn ngữ chi phí vì sao nếu không làm đối soát trong hai quý tới thì mỗi đối tác mới sẽ tốn thêm ba tuần tích hợp (*số minh hoạ*). 16:00 — Review RFC do một Senior team Platform viết; comment sáu chỗ, năm chỗ là câu hỏi, một chỗ là ràng buộc cứng. 17:00 — Cập nhật một trang tổng quan về trạng thái lĩnh vực của mình: quyết định đang mở, rủi ro, ai đang sở hữu cái gì.

Tổng: khoảng 2,5 giờ viết, 3 giờ trao đổi 1-1 và họp phản biện, 1 giờ vận hành thật, 1 giờ phát triển người. Số dòng code viết trong ngày: có thể bằng không, và điều đó không phải vấn đề. Phép thử của một tuần ở bậc Staff không phải "tôi đã làm xong gì" mà **"quyết định nào của người khác đã tốt hơn vì tôi có mặt trong tuần này"**.

---

## 6. Principal Engineer

> Ở bậc này, câu hỏi "tuần này anh làm gì" trở nên khó trả lời một cách trung thực. Không phải vì công việc mơ hồ, mà vì đầu ra của một tuần thường là **một câu hỏi được đặt ra đúng lúc** hoặc **một hướng đi bị chặn lại** — hai thứ không xuất hiện trên bất kỳ board nào. Nhiều tổ chức Việt không có bậc này, và điều đó không hẳn là thiếu sót: một công ty 40 engineer với một dòng sản phẩm thường không có đủ khối lượng bài toán ở phạm vi này để một người sống bằng nó.

### Hàm mục tiêu ở bậc này

Thành công ở bậc Principal là: **hàng trăm quyết định kỹ thuật độc lập của người khác, trong nhiều năm, hội tụ về một hướng còn dùng được — thay vì phân kỳ thành một tập hệ thống không ai gộp lại được.**

Loại bài toán ở bậc này khác về bản chất, không khác về độ khó. Staff giải **bài toán khó đã được nhận diện là bài toán**. Principal làm việc với **thứ chưa ai biết là một bài toán**. Sự khác biệt này rất cụ thể:

- Staff: "Đối soát đang sai ba lần một quý, phải thiết kế lại." — Vấn đề đã có người phàn nàn, đã có triệu chứng.
- Principal: "Chúng ta đang gắn logic định giá vào bốn service khác nhau. Hiện tại không ai đau. Nhưng khi mở thị trường thứ hai vào năm sau, mọi thay đổi giá sẽ cần bốn team phối hợp, và tốc độ ra thị trường của công ty sẽ bị quyết định bởi một quyết định kỹ thuật không ai còn nhớ đã ra." — Chưa có triệu chứng. Chưa ai phàn nàn. Người duy nhất nhìn thấy nó phải tự tạo ra bằng chứng để thuyết phục người khác rằng vấn đề tồn tại.

Đặc điểm này tạo ra một khó khăn tâm lý đặc trưng: **bạn đúng vào thời điểm không ai kiểm chứng được, và được xác nhận vào thời điểm không còn ai nhớ.** Nếu bạn ngăn được một khủng hoảng, không có khủng hoảng nào để chỉ vào. Đây là lý do bậc này đòi hỏi viết lại: một dự đoán đã ghi ngày tháng là bằng chứng duy nhất bạn có.

Ba đầu ra cụ thể:

1. **Định hình ràng buộc và hướng đi** để quyết định của người khác hội tụ — chuẩn dữ liệu, ranh giới hệ thống, danh mục công nghệ được phép, nguyên tắc chia service.
2. **Phát hiện bài toán trước khi nó thành khủng hoảng**, kèm bằng chứng đủ để tổ chức chịu trả giá ngay bây giờ.
3. **Nói "không" với những hướng đi tốn kém mà chưa ai chất vấn** — đầu ra này bị đánh giá thấp nhất và có giá trị kinh tế cao nhất.

### Vế thứ ba: nói "không" như một đầu ra

Trong một tổ chức đã scale, những đề xuất tốn kém nhất thường không đến từ sự thiếu năng lực — chúng đến từ những hướng đi **có vẻ hiển nhiên đúng và không ai có vị trí để chất vấn**: "chuyển toàn bộ sang microservices", "tự xây nền tảng dữ liệu thay vì mua", "viết lại hệ thống cũ", "chuẩn hoá tất cả về một framework". Mỗi đề xuất này có người bảo vệ nhiệt tình, có luận điểm nghe được, và có chi phí thật rơi vào 18 tháng sau.

Principal là người có đủ ba thứ để chặn lại: dữ liệu lịch sử về những lần tương tự, uy tín để phát biểu ngược lại đa số, và vị trí không phụ thuộc vào việc đề xuất đó được duyệt. Nhưng "nói không" ở bậc này không phải phủ định — nó là **thay thế bằng một phương án nhỏ hơn có thể kiểm chứng được**:

| Đề xuất | Cách nói "không" tệ | Cách nói "không" dùng được |
|---|---|---|
| Viết lại hệ thống cũ trong 12 tháng | "Không khả thi, tôi đã thấy nhiều lần thất bại" | "Chọn một luồng chiếm 30% lưu lượng, viết lại trong 8 tuần, đo bằng ba chỉ số đã định trước. Nếu đúng như kỳ vọng thì làm tiếp; nếu không, ta mất 8 tuần thay vì 12 tháng" |
| Tách 60 microservice | "Chúng ta chưa đủ trưởng thành" | "Tách theo ranh giới có đau thật: chỉ tách phần nào đang buộc hai team phải deploy cùng nhau. Số service là kết quả, không phải mục tiêu" |
| Tự xây nền tảng dữ liệu | "Mua rẻ hơn" | "Chi phí tự xây gồm cả hai người vận hành nó trong ba năm. Ước lượng của tôi cao hơn đề xuất 2,5 lần (*số minh hoạ*). Nếu con số đó sai, hãy chỉ ra chỗ sai" |

Cột thứ ba có một điểm chung: nó **chuyển cuộc tranh luận từ thẩm quyền sang cơ chế kiểm chứng**. Đây là kỹ năng trung tâm của bậc này.

### Phạm vi ảnh hưởng

- **Toàn bộ tổ chức engineering**, và ở một số trường hợp là quan hệ với đối tác, cơ quan quản lý, hoặc cộng đồng kỹ thuật bên ngoài.
- **Chân trời 1–5 năm.** Đây là biên độ mà không có công cụ nào kiểm chứng được ngoài lịch sử. Nó đòi hỏi khả năng sống nhiều tháng trong trạng thái không biết mình đúng hay sai — và không chuyển trạng thái đó thành sự do dự khi phát biểu.
- **Số quyết định ít, chi phí mỗi quyết định lớn.** Một Principal có thể chỉ ra 5–10 quyết định thực sự trong một năm. Điều này khó chịu với người quen với nhịp phản hồi hàng ngày.
- **Không có quyền nhân sự.** Nguồn quyền lực: track record dài đã được thực tế phán xử, cộng với niềm tin của tầng executive — và niềm tin đó được xây bằng những lần dự đoán đúng có ghi ngày, không bằng số năm.

### Cách ra quyết định

Thay đổi cốt lõi: từ **ra quyết định** sang **định khung (framing) để người khác ra quyết định đúng**. Cơ chế đằng sau: ở phạm vi tổ chức, số quyết định vượt xa khả năng một người tham gia; nên đòn bẩy nằm ở việc thay đổi *tập lựa chọn* và *tiêu chí* mà người khác dùng, chứ không nằm ở việc tự chọn.

Một framing dùng được có bốn phần: (1) nêu quyết định thực sự đang được đặt ra, thường khác với câu hỏi ban đầu; (2) hai đến ba lựa chọn thật, mỗi lựa chọn kèm cái mất; (3) tiêu chí phân biệt — điều gì làm ta nghiêng bên nào; (4) thứ sẽ không thay đổi bất kể chọn gì. Phần (4) thường bị bỏ và là phần làm người quyết an tâm nhất.

| Loại quyết định | Vai của Principal |
|---|---|
| Định hướng kỹ thuật nhiều năm (nền tảng, ranh giới hệ thống, danh mục công nghệ) | Sở hữu; viết ra; chịu trách nhiệm nếu hướng sai |
| Đầu tư kỹ thuật lớn (build vs buy, viết lại, migration nhiều quý) | Định khung và ước lượng chi phí; CTO/Director chốt vì liên quan ngân sách |
| Chuẩn xuyên tổ chức | Sở hữu, nhưng thực thi qua Staff và Tech Lead của từng lĩnh vực |
| Thiết kế tổ chức | Không sở hữu, nhưng phải nói khi cấu trúc tổ chức đang sinh ra kiến trúc sai |
| Kiến trúc trong một lĩnh vực đã có Staff | Không; can thiệp vào đây là làm suy yếu tầng Staff |
| Tuyển và đánh giá | Tham gia phỏng vấn tầng cao, làm calibration; không quyết |

Ai phản biện bạn: CTO, các Staff (đây là nhóm quan trọng nhất — nếu không Staff nào dám phản biện bạn, tổ chức đã mất một tầng kiểm soát), Head of Product về giả định thị trường, và Finance về chi phí. Cơ chế đối chiếu bắt buộc: một tài liệu ghi các dự đoán kèm ngày, đọc lại mỗi sáu tháng. Đây là thứ duy nhất phân biệt phán đoán với uy tín tích luỹ.

### Kỹ năng cần phát triển

**1. Đọc bối cảnh kinh doanh đủ sâu để suy ra ràng buộc kỹ thuật.** Vì sao ở bậc này: mọi quyết định chân trời nhiều năm đều là một cược vào hướng đi của doanh nghiệp. Cụ thể, cần trả lời được: doanh thu đến từ đâu và phần nào đang tăng; đơn vị kinh tế của một giao dịch; công ty sống được bao lâu với tiền hiện có; ràng buộc compliance nào sắp đổi; thị trường tiếp theo có gì khác về mặt kỹ thuật. Một kiến trúc tối ưu cho quy mô gấp mười ở một công ty còn 14 tháng runway là một quyết định sai, bất kể nó đúng về kỹ thuật.

**2. Viết cho hai loại người đọc rất khác nhau.** Với engineer: cơ chế, ràng buộc, con số kỹ thuật, các phương án đã loại. Với executive: một trang, kết luận ở dòng đầu, chi phí và rủi ro bằng tiền và thời gian, và câu "tôi cần anh quyết điều gì". Sai lầm phổ biến là mang bản dành cho engineer đi họp executive — kết quả là tài liệu không được đọc và kết luận là "kỹ thuật không giải thích được cho ai". Ở bối cảnh Việt Nam, thêm một tầng: khi executive hoặc đối tác dùng tiếng Anh, khả năng viết ngắn gọn và chính xác bằng tiếng Anh trở thành ràng buộc thật lên phạm vi ảnh hưởng của bạn, không phải một kỹ năng phụ.

**3. Sức bền chính trị.** Không phải mưu lược — mà là khả năng **giữ một quan điểm đúng qua nhiều quý mà không ai ủng hộ, không biến nó thành xung đột cá nhân, và vẫn giữ được quan hệ để thực thi khi thời điểm đến**. Cụ thể: nêu lại một lần mỗi quý với bằng chứng mới thay vì nêu mười lần trong một tháng; tách "tôi không đồng ý" khỏi "tôi không hợp tác"; và khi tổ chức chọn hướng khác, cam kết thực thi hướng đó đồng thời ghi lại điều kiện sẽ khiến ta xem lại. Người không có kỹ năng này rơi vào một trong hai cực: bỏ cuộc sau lần đầu bị từ chối, hoặc trở thành người phản đối thường trực mà không ai nghe nữa.

**4. Nhận diện tín hiệu sớm một cách có phương pháp.** Vì sao là kỹ năng chứ không phải trực giác: "nhìn trước" có thể phân rã thành các thao tác lặp lại được. Cụ thể: đọc postmortem của toàn tổ chức để tìm **lớp nguyên nhân lặp lại**, không phải nguyên nhân từng ca; theo dõi những chỉ số đi trước khủng hoảng (thời gian onboard một đối tác mới, số team phải phối hợp cho một thay đổi thường gặp, thời gian từ commit đến production); và một lần mỗi quý viết ba trang trả lời câu "nếu quy mô gấp năm và có thêm một thị trường, chỗ nào vỡ trước".

**5. Xây tầng dưới mình.** Đầu ra dài hạn của một Principal được đo bằng số Staff mà tổ chức có. Cụ thể: giao lại các lĩnh vực đã ổn định, review RFC của người khác thay vì tự viết, và đưa Staff vào các cuộc họp mà bạn đang là người duy nhất từ engineering.

### Tín hiệu cho thấy đã sẵn sàng lên bậc tiếp theo

Từ bậc Principal, "bậc tiếp theo" không phải một ô cao hơn trong bảng — thường là mở rộng phạm vi (CTO, VP Engineering, hoặc Principal ở một tổ chức lớn hơn nhiều). Tín hiệu:

- **Các dự đoán bạn ghi 2–3 năm trước đã được thực tế phán xử**, và tỷ lệ đúng cao hơn ngẫu nhiên một cách rõ ràng — kèm việc bạn nói được cái nào sai và vì sao.
- **Tổ chức tự chạy được các cơ chế bạn dựng** (RFC process, calibration kỹ thuật, chuẩn dữ liệu) mà không cần bạn có mặt.
- **Executive mang cho bạn những câu hỏi chưa thành hình** — "chúng ta có nên vào thị trường này không" — thay vì những câu hỏi kỹ thuật đã đóng gói.
- **Có ít nhất hai Staff trong tổ chức mà bạn đã góp phần tạo ra**, và họ phản biện bạn được.

### Sai lầm thường gặp

**1. Mất kết nối với hiện trường nên đưa ra chiến lược không thi hành được.** Cơ chế: bậc này kéo bạn về phía họp, tài liệu, và trao đổi với executive. Sau 12 tháng, mô hình trong đầu bạn về hệ thống là mô hình của 12 tháng trước, nhưng bạn vẫn phát biểu với cùng độ tự tin. Hậu quả: một chiến lược mà các team không phản đối trong họp rồi im lặng không thực thi. Dấu hiệu sớm: bạn không nhớ lần cuối đọc log production, và ước lượng thời gian của bạn cho một việc lệch quá hai lần so với người làm. Cách chữa: mỗi quý tự làm trọn một việc nhỏ nhưng thật — sửa một bug thật, làm một prototype, ngồi một ca on-call.

**2. Dùng uy tín cũ để áp quyết định.** Cơ chế: uy tín tích luỹ làm chi phí thuyết phục giảm dần, đến mức bạn không cần lập luận nữa — người ta đồng ý vì bạn nói. Điều này rất tiện và rất nguy hiểm: nó tắt vòng phản hồi vốn là thứ giữ cho phán đoán của bạn còn đúng. Dấu hiệu sớm: đã nhiều tháng không ai phản biện bạn trong một cuộc thiết kế; hoặc bạn nhận ra mình đang trả lời "tôi đã thấy chuyện này nhiều lần rồi" thay vì trình bày cơ chế. Cách chữa mang tính cơ chế: yêu cầu một Staff cụ thể đóng vai phản biện trong mỗi tài liệu lớn, và công khai sửa theo phản biện đó ít nhất một lần để mọi người thấy việc phản biện có tác dụng.

**3. Làm việc đơn độc nên không ai thực thi kết luận của mình.** Cơ chế: bài toán ở bậc này đòi hỏi suy nghĩ dài và sâu, việc đó làm một mình hiệu quả hơn. Nhưng một kết luận đúng do một người nghĩ ra một mình không có ai sở hữu ngoài người đó. Hậu quả điển hình: một tài liệu chiến lược xuất sắc, được khen, và không thay đổi hành vi của ai. Cách chữa: đưa 3–4 người vào từ giai đoạn *đặt câu hỏi*, không phải giai đoạn review — người tham gia định nghĩa vấn đề sẽ thực thi kết luận; người chỉ được xem bản cuối thì không.

**4. Giải bài toán ở phạm vi Staff vì nó dễ chịu hơn.** Cơ chế: bài toán Principal mơ hồ, phản hồi chậm, có thể sai trong hai năm. Bài toán Staff rõ ràng, giải được trong một quý, và có người cảm ơn. Người ta trôi về phía dễ chịu mà không nhận ra. Dấu hiệu sớm: năm nay bạn không viết được một tài liệu nào có chân trời trên một năm.

**5. Không nói khi cấu trúc tổ chức đang sinh ra kiến trúc sai.** Ranh giới team quyết định ranh giới hệ thống — đây là quan sát công khai lâu đời (định luật Conway). Principal thường tự coi thiết kế tổ chức là việc của Director nên im lặng, rồi mất hai năm chữa hậu quả kiến trúc của một quyết định tổ chức mà lúc đó nói ra thì rẻ.

### Một ngày làm việc điển hình

Đặc điểm định nghĩa: **một ngày trông rất giống nhau về hình thức nhưng khác nhau hoàn toàn về giá trị**, và bạn thường không biết ngày nào là ngày quan trọng cho đến nhiều tháng sau.

8:30 — Đọc: postmortem tuần trước của ba team, một báo cáo doanh thu theo dòng sản phẩm, và một tài liệu thị trường mới từ Trang (Head of Product). Không phản hồi gì, chỉ tìm mẫu lặp lại. 9:30 — Khối viết 2 giờ: bản một trang cho ban lãnh đạo về việc logic định giá đang phân tán trong bốn service và chi phí của nó khi mở thị trường thứ hai; kết luận ở dòng đầu, ba con số, một yêu cầu quyết định. 11:30 — 45 phút với Minh (CTO): không xin duyệt, mà kiểm tra xem giả định của mình về kế hoạch 18 tháng có còn đúng; phát hiện thị trường thứ hai có thể lùi hai quý, sửa mức độ khẩn của đề xuất. 13:00 — Design review với hai Staff về ranh giới của nền tảng tích hợp đối tác; vai của bạn là chỉ ra rằng câu hỏi họ đang tranh luận không phải câu hỏi thật, câu hỏi thật là ai chịu trách nhiệm khi đối tác đổi contract. 14:00 — 1-1 với Linh (Staff): giao lại toàn bộ lĩnh vực dữ liệu, thống nhất bạn chỉ còn review các quyết định vượt ranh giới. 14:45 — Một Director đề xuất chuẩn hoá toàn bộ về một framework; bạn không phản đối trong họp, hẹn gửi ước lượng chi phí migration trong hai ngày — cách chặn một hướng đi bằng con số thay vì bằng vị thế. 15:30 — Ca on-call quan sát: đọc trace một sự cố ở team mình không quản, để giữ mô hình hệ thống trong đầu còn đúng. 16:30 — Cập nhật tài liệu dự đoán: ghi lại hai dự đoán mới kèm ngày và tiêu chí kiểm chứng; đọc lại hai dự đoán từ 18 tháng trước, một đúng một sai, ghi lý do sai. 17:15 — Trả lời một Senior ở team khác hỏi về lựa chọn hàng đợi; trả lời bằng ba câu hỏi thay vì một khuyến nghị.

Tổng: 2–3 giờ viết, 3 giờ trao đổi có mục đích, 1 giờ đọc để giữ mô hình còn đúng, 1 giờ phát triển tầng Staff. Điểm khác biệt cốt lõi so với bậc Staff: **bạn không còn được đo bằng bài toán bạn giải, mà bằng những bài toán tổ chức tránh được và không bao giờ biết là đã tránh** — nên nếu bạn không tự ghi lại dự đoán và kết quả, sẽ không có ai làm việc đó thay bạn.

---

## 7. Engineering Manager

> Đây là điểm rẽ duy nhất trong tám bậc mà hàm mục tiêu **đổi hoàn toàn**, không mở rộng. Từ Junior đến Principal, mọi bậc đều là biến thể của một câu: "phán đoán kỹ thuật của tôi tạo ra kết quả ở phạm vi ngày càng lớn". Ở bậc EM, câu đó bị thay bằng: "năng lực và kết quả của người khác là đầu ra của tôi". Vì đây là một hàm khác chứ không phải một mức cao hơn, **việc bạn giỏi ở bậc trước không nói gì nhiều về việc bạn sẽ giỏi ở bậc này** — và ngược lại, quay lại nhánh IC sau hai năm làm EM không phải thất bại, mà là một quan sát về hàm mục tiêu nào phù hợp với bạn.

### Hàm mục tiêu ở bậc này

Thành công ở bậc EM là: **team của bạn liên tục cho ra kết quả kinh doanh, và năng lực của từng người trong đó cao hơn sau bốn quý so với khi họ vào.**

Hai vế, và cả hai đều là đầu ra của người khác. Không có vế nào chứa thứ bạn tự làm. Đây là lý do chuyển dịch này khó theo một cách mà các chuyển dịch khác không khó: bạn phải **thay đổi định nghĩa của một ngày làm việc tốt**. Một ngày mà bạn không tạo ra vật gì hữu hình nhưng đã tháo được một xung đột đang làm chậm hai người, đã sửa một mô tả công việc để tuyển đúng hơn, và đã nói với một người rằng họ đang không đạt kỳ vọng — đó là một ngày tốt ở bậc này, và bộ não bạn sẽ báo ngược lại.

Năm đầu ra cụ thể mà tổ chức thực sự mua khi thuê một EM:

1. **Tuyển đúng.** Đây là đòn bẩy dài hạn lớn nhất và cũng là quyết định khó đảo ngược nhất. Một lần tuyển sai trong team 6 người tiêu của bạn một đến hai quý — bao gồm cả thời gian của những người khác.
2. **Phát triển người.** Ai đang chững lại, ai sắp sẵn sàng cho việc lớn hơn, ai đang bị dùng sai chỗ. Đầu ra kiểm chứng được: sau bốn quý, có người trong team làm được việc mà bốn quý trước phải chờ người khác.
3. **Cấu trúc team** — ai làm gì, ranh giới sở hữu, ai giữ vai Tech Lead, khi nào tách team. Đây là công cụ mạnh hơn mọi lời khuyên cá nhân, vì nó thay đổi điều kiện chứ không chỉ thay đổi hành vi.
4. **Loại bỏ vật cản.** Từ vật cản nhỏ (thiếu quyền truy cập, môi trường test hỏng) đến vật cản cấu trúc (team phụ thuộc một team khác không có cam kết nào).
5. **Quyết định phân bổ nguồn lực.** Ai vào việc gì, phần trăm capacity cho công việc kỹ thuật không tạo ra feature, khi nào nói không với một yêu cầu từ bên ngoài.

### Phần mất và phần được — nói thẳng

Đây là mục hay bị viết cho có, nên nó cần viết thật. Ai đang cân nhắc nhánh này cần biết chính xác cái giá, không phải một câu "quản lý là một sự đánh đổi".

| Bạn mất gì | Cơ chế cụ thể |
|---|---|
| **Tiếp xúc kỹ thuật** | Không phải mất khả năng, mà mất *tính hiện thời*. Sau 12–18 tháng, mô hình của bạn về codebase lạc hậu; bạn vẫn hiểu nguyên lý nhưng không còn quyền phát biểu về chi tiết. Điều này đau nhất với người có uy tín được xây bằng độ sâu kỹ thuật. |
| **Cảm giác hoàn thành hàng ngày** | Bạn đã có 5–10 năm sống với vòng phản hồi vài phút: compile, test xanh, deploy. Vòng phản hồi đó biến mất gần như hoàn toàn. Nhiều người mô tả năm đầu làm EM là "cảm giác không làm được gì cả", dù về mặt khách quan họ tạo ra nhiều giá trị hơn trước. |
| **Feedback loop kéo dài hàng quý** | Một quyết định về cấu trúc team hay về một người chỉ cho biết kết quả sau 2–4 quý. Trong khoảng đó bạn phải hành động tiếp mà không biết mình đúng hay sai. |
| **Phải mang tin xấu** | Nói với một người rằng họ không được promote, rằng họ đang underperform, rằng team bị cắt headcount, rằng dự án họ làm sáu tháng bị dừng. Bạn thường không phải người ra quyết định đó nhưng luôn là người truyền đạt nó, và bạn không được phép nói "tôi cũng không đồng ý" nếu điều đó phá vỡ khả năng thực thi. |
| **Không còn được đánh giá bởi thứ mình kiểm soát trực tiếp** | Kết quả của bạn là hàm của 6–8 người có ý chí riêng, cộng với những yếu tố ngoài tầm (thị trường, quyết định của tầng trên, một người giỏi nghỉ việc). Bạn chịu trách nhiệm về những thứ bạn chỉ ảnh hưởng được một phần. |
| **Sự riêng tư của cảm xúc** | Sự lo lắng của bạn lan xuống team với hệ số phóng đại. Bạn mất quyền thể hiện bất an một cách tự nhiên trong giờ làm việc. |

| Bạn được gì | Cơ chế cụ thể |
|---|---|
| **Đòn bẩy lớn hơn** | Một quyết định cấu trúc tốt tạo ra nhiều giá trị hơn toàn bộ code bạn có thể viết trong một quý. Đây là điều kiện thực, không phải lời động viên: tổ chức đúng cấu trúc làm cho những việc đúng trở thành việc dễ nhất để làm. |
| **Ảnh hưởng thật lên đời sống nghề nghiệp của người khác** | Bạn là biến số lớn nhất quyết định một người phát triển hay chững lại trong hai năm tới. Đây vừa là phần có ý nghĩa nhất, vừa là gánh nặng, và ai không cảm thấy nó là gánh nặng thường đang làm sai. |
| **Nhìn thấy hệ thống rõ hơn** | Bạn thấy đồng thời: lý do thật của một quyết định roadmap, ràng buộc ngân sách, động lực của từng người, và cách một quyết định ở tầng trên biến thành hành vi ở tầng dưới. Góc nhìn này không có cách nào lấy được từ vị trí IC. |
| **Kỹ năng chuyển được ra ngoài nghề** | Tuyển người, xử lý xung đột, đàm phán ưu tiên, dẫn một cuộc trò chuyện khó — những kỹ năng này không mất giá khi công nghệ đổi. |

Một lưu ý về bối cảnh Việt Nam: kỳ vọng gia đình và xã hội thường coi "làm quản lý" là dấu hiệu thành công, và nhiều công ty không có bậc Staff thật nên nhánh EM là con đường duy nhất để tăng lương đáng kể. Hai áp lực này cộng lại đẩy rất nhiều IC giỏi vào bậc này vì lý do bên ngoài. Câu hỏi tự kiểm tra không phải "tôi có làm được không" (phần lớn người giỏi làm được) mà là: **nếu lương và title của hai nhánh bằng nhau, tôi chọn nhánh nào?**

### Phạm vi ảnh hưởng

- **Người**: 5–10 người báo cáo trực tiếp. Dưới 4, bạn không đủ khối lượng việc quản lý nên sẽ tự lấp bằng việc kỹ thuật (và làm hỏng vai). Trên 10, thời gian 1-1 bị nén xuống mức không phát hiện được vấn đề trước khi nó nổ.
- **Chân trời thời gian**: 1 quý đến 1 năm. Bạn phải giữ đồng thời: quý này team có giao được không, và trong một năm ai trong team sẽ ở đâu.
- **Thẩm quyền chính thức**: lần đầu bạn có nó — đánh giá hiệu suất, đề xuất lương và promotion, quyết định ai vào/ra team, tham gia quyết định sa thải. Đây là nguồn quyền lực khác về bản chất với uy tín: nó được cấp, nó rút lại được, và **nó chỉ nên dùng khi ảnh hưởng đã không đủ**.
- **Hậu quả khi sai**: rơi vào đời sống nghề nghiệp của những con người cụ thể, và không phải lúc nào cũng sửa được. Đây là loại hậu quả không có ở bất kỳ bậc IC nào.

### Cách ra quyết định

Cơ sở ra quyết định đổi từ **bằng chứng kỹ thuật** sang **bằng chứng về hành vi qua thời gian**. Đặc điểm khó chịu của loại bằng chứng này: nó không lặp lại được, không đo lại được, và luôn có ít nhất hai cách giải thích. Một người giao chậm ba sprint liền có thể là do năng lực, do bài toán bị định nghĩa sai, do một vấn đề cá nhân, hoặc do bạn đã giao sai người. Bốn cách chữa hoàn toàn khác nhau, và chọn sai thì tốn một quý.

| Loại quyết định | Thuộc EM? | Ghi chú |
|---|---|---|
| Đánh giá hiệu suất, đề xuất lương, promotion | Có, đây là quyết định trung tâm | Input từ Tech Lead và peer là bắt buộc, nhưng kết luận là của bạn |
| Ai vào team, ai ra khỏi team | Có | Bao gồm cả việc chủ động chuyển một người sang team phù hợp hơn |
| Ai giữ vai Tech Lead | Có | Và phải nói rõ đây là vai, không phải bậc |
| Phân bổ capacity giữa feature và công việc kỹ thuật | Có, cùng PO | Nếu bạn không bảo vệ được phần này thì không ai bảo vệ |
| Kiến trúc và giải pháp kỹ thuật | Không | Thuộc Tech Lead / Staff. Bạn có quyền hỏi "chi phí và rủi ro là gì", không có quyền chọn phương án |
| Ưu tiên sản phẩm | Không | Thuộc PO. Bạn cung cấp ràng buộc thật về capacity |
| Kết thúc hợp đồng với một người | Đề xuất và thực thi; HR và Director cùng quyết | Nhưng bạn là người phải nói, và phải nói được lý do rõ |

Ai phản biện bạn: từng thành viên trong team (qua 1-1 và qua việc họ ở lại hay đi), Director của bạn, PO, và **tỷ lệ rời việc cùng độ chính xác của các dự báo bạn đã đưa ra**. Cơ chế đối chiếu dùng được ở bậc này: mỗi quý viết ra ba dự đoán về người và team ("Vy sẽ sẵn sàng dẫn một epic vào quý sau"; "sự phụ thuộc vào team Platform sẽ làm trễ ít nhất hai tuần"), rồi đọc lại. Không có cơ chế này, bạn sẽ nhớ chọn lọc và tưởng phán đoán của mình về người tốt hơn thực tế.

### Kỹ năng cần phát triển

**1. Dẫn một cuộc trò chuyện khó đến kết luận.** Vì sao ở bậc này: đây là kỹ năng có tỷ lệ "khó/không thể tránh" cao nhất, và là kỹ năng phân biệt EM thật với người giữ chức EM. Cấu trúc dùng được: nêu hành vi quan sát được (không nêu tính cách) → nêu hậu quả cụ thể lên ai → hỏi cách người đó nhìn việc này → thống nhất thay đổi cụ thể và mốc kiểm tra → viết lại một đoạn ngắn sau buổi nói. Vì sao khó ở bối cảnh Việt Nam: văn hoá "giữ mặt" và tránh xung đột trực diện làm phần "nêu hậu quả" bị làm mềm đến mức mất nghĩa. Kiểm tra kết quả: hôm sau người đó có nói lại được chính xác điều cần thay đổi và mốc thời gian không? Nếu không, buổi nói chuyện đó chưa xảy ra.

**2. Tuyển người như một quy trình có thể lặp lại, không như một chuỗi phỏng vấn theo cảm giác.** Cụ thể: xác định trước 3–4 tín hiệu cần kiểm tra và ai kiểm tra tín hiệu nào; câu hỏi dựa trên hành vi trong quá khứ chứ không dựa trên giả định ("kể một lần anh phải đổi hướng kỹ thuật giữa dự án — điều gì làm anh đổi?"); và quyết định dựa trên bằng chứng đã ghi, không dựa trên trí nhớ về ấn tượng. Cơ chế đằng sau: phỏng vấn không cấu trúc có giá trị dự báo thấp — đây là kết luận đã được lặp lại trong nhiều nghiên cứu công khai về tuyển dụng — vì nó chủ yếu đo mức độ giống người phỏng vấn.

**3. Chẩn đoán vấn đề hiệu suất trước khi can thiệp.** Vì sao là kỹ năng riêng: bản năng của người từ nhánh kỹ thuật là chữa ngay. Bốn nguyên nhân cần loại trừ theo thứ tự: kỳ vọng chưa bao giờ được nói rõ (thường gặp nhất và rẻ nhất để chữa) → thiếu kỹ năng cụ thể → thiếu động lực hoặc đang bị dùng sai chỗ → không phù hợp với vai. Ba nguyên nhân đầu là việc của bạn; chỉ nguyên nhân thứ tư dẫn đến việc thay đổi vai hoặc chia tay. Chi phí của việc bỏ qua bước chẩn đoán: bạn đưa một người vào performance plan trong khi vấn đề thật là bạn chưa từng nói rõ kỳ vọng.

**4. Chuyển ngữ giữa hai hướng.** Xuống dưới: biến một quyết định mơ hồ từ tầng trên thành việc cụ thể có ý nghĩa, giữ được phần *vì sao*. Lên trên: biến ràng buộc kỹ thuật và tình trạng team thành ngôn ngữ chi phí, rủi ro, và thời gian. Kỹ năng cụ thể ở chiều lên: không bao giờ báo cáo "team đang cố gắng"; báo cáo "với capacity hiện tại, phần báo cáo sẽ xong sau release, hoặc ta bỏ phần import — tôi đề xuất bỏ, vì...".

**5. Xây cơ chế thay vì tự làm chốt kiểm soát.** Vì sao ở bậc này: một EM tự mình là điểm kiểm soát cho mọi thứ sẽ chặn ở 6 người. Cụ thể: thay vì tự duyệt mọi quyết định kỹ thuật, dựng ngưỡng rõ ràng cái gì Tech Lead quyết; thay vì tự phát hiện mọi vấn đề, dựng nhịp mà vấn đề tự nổi lên (1-1 hai tuần một lần có nội dung thật, retro có hành động được theo dõi, một kênh để báo vật cản); thay vì tự nhớ ai cần gì, có một tài liệu về mục tiêu phát triển của từng người.

**6. Chịu được sự mơ hồ về giá trị của chính mình.** Đây là kỹ năng, không phải tính cách, và nó phải được xây bằng cơ chế: viết một bản ghi cuối tuần về những gì đã thay đổi vì bạn (một xung đột được tháo, một người được unblock, một cam kết được đàm phán lại) — vì nếu không viết, cuối tuần bạn sẽ chỉ nhớ rằng mình đã họp cả tuần.

### Tín hiệu cho thấy đã sẵn sàng lên bậc tiếp theo

- **Team giao được ổn định qua 3–4 quý và không phụ thuộc vào sự có mặt hàng ngày của bạn.** Bạn nghỉ hai tuần, không có gì đổ.
- **Bạn đã dẫn trọn vòng một trường hợp hiệu suất khó** — chẩn đoán, can thiệp, và một kết cục rõ ràng dù là đi lên hay chia tay — và làm được điều đó mà quan hệ với những người còn lại trong team không xấu đi.
- **Có người trong team giờ đang giữ vai Tech Lead tốt**, và bạn không phải là người duy nhất phát hiện vấn đề trong team.
- **Bạn bắt đầu thấy những vấn đề không giải được trong một team**: hai team liên tục va nhau vì ranh giới sở hữu sai; quy trình tuyển của cả bộ phận cho ra kết quả kém; tầng Tech Lead của tổ chức không được đào tạo. Nhìn ra loại vấn đề này là cửa vào bậc Director.
- **Bạn đã phát triển ít nhất một người từ Mid lên Senior một cách có chủ ý**, và nói được mình đã làm gì cụ thể.

### Sai lầm thường gặp

**1. Vẫn nhận task trên critical path.** Cơ chế: đây là chỗ bạn tự tin nhất, và trong ngắn hạn nó thực sự giúp team. Nhưng lịch của một EM bị gián đoạn bởi những việc không dời được (1-1, xử lý một xung đột, một cuộc họp khủng hoảng), nên một task nằm trên đường tới hạn sẽ bị bạn làm trễ — và người phụ thuộc vào nó không dám nhắc bạn vì bạn là manager của họ. Dấu hiệu sớm: có ticket của bạn đứng nguyên bốn ngày mà không ai hỏi. Cách chữa: bạn chỉ nhận việc thoả hai điều kiện — không nằm trên đường tới hạn, và nếu bạn không làm được thì không ai bị chặn.

**2. Micromanage vì lo lắng.** Cơ chế quan trọng ở đây là **nguyên nhân**: micromanagement gần như không bao giờ đến từ mong muốn kiểm soát, nó đến từ việc bạn mất khả năng nhìn thấy tiến độ và bộ não bù đắp bằng cách hỏi nhiều hơn. Hậu quả: người bị hỏi liên tục sẽ chuyển sang chế độ báo cáo phòng thủ, thông tin bạn nhận được xấu đi, bạn lo hơn, và vòng lặp tự củng cố. Cách chữa đúng là chữa nguyên nhân, không chữa hành vi: tạo ra tín hiệu tiến độ mà bạn không phải đi hỏi (một bản cập nhật viết ngắn giữa tuần, một board phản ánh thực tế, checkpoint đã hẹn trước). Dấu hiệu sớm: bạn hỏi "xong chưa" nhiều hơn một lần mỗi ngày cho cùng một việc.

**3. Trở thành người truyền tin thay vì người quyết định.** Cơ chế: chuyển nguyên vẹn yêu cầu từ trên xuống và than phiền từ dưới lên là con đường ít xung đột nhất — không ai giận bạn vì bạn không phải người quyết gì. Hậu quả sau hai quý: team thấy bạn không bảo vệ được họ nên ngừng nói thật, và tầng trên thấy bạn không lọc được gì nên ngừng hỏi ý bạn. Bạn mất giá trị ở cả hai chiều đồng thời. Phép thử: quý này bạn đã nói "không" với ai ở tầng trên, về việc gì, và với lập luận nào?

**4. Tránh cuộc trò chuyện khó.** Cơ chế chi phí giống như trường hợp Tech Lead nhận cam kết vượt capacity: chi phí của việc nói đến ngay và rơi vào bạn; chi phí của việc không nói đến sau và rơi vào người khác — team chịu đựng người underperform, những người giỏi thấy tiêu chuẩn không được thực thi và mất niềm tin vào bạn, còn chính người đó mất thêm một năm mà không ai nói cho họ biết. Hình thức nguy hiểm nhất của sai lầm này là feedback được nói nhẹ đến mức người nhận nghĩ đó là lời khen có kèm góp ý nhỏ.

**5. Đối xử với mọi người theo cùng một cách vì "công bằng".** Cơ chế: công bằng ở bậc này là **cùng một tiêu chuẩn, khác cách hỗ trợ**. Áp dụng cùng một mức độ tự do cho một Junior mới vào và một Senior sáu năm là làm hỏng cả hai: người thứ nhất chìm, người thứ hai bị xúc phạm.

**6. Giữ lại vai Tech Lead của mình.** Rất thường gặp khi một Tech Lead được lên EM cho chính team cũ. Hậu quả: người được giao vai Tech Lead mới không có thẩm quyền thật vì mọi người vẫn hỏi bạn, và bạn tiêu thời gian vào việc mà tổ chức đã trả tiền cho người khác làm. Cách chữa mang tính cơ chế: chuyển hướng công khai — khi có người hỏi bạn một câu kỹ thuật thuộc phạm vi Tech Lead, trả lời "hỏi Tuấn, đây là quyết định của cậu ấy" trong kênh chung, không nhắn riêng.

### Một ngày làm việc điển hình

Đặc điểm định nghĩa: **lịch của bạn phần lớn do người khác quyết**, và phần bạn tự quyết được là phần quyết định bạn có phải một EM tốt hay không.

8:30 — Đọc board và kênh team, không nhắn gì; ghi ba câu cần hỏi trong ngày. 9:00 — Standup, dự với vai người nghe; sau đó nhắn riêng cho một người có vẻ mệt hai ngày liền. 9:30 — 1-1 với Vy (Mid, 1,5 năm): 40 phút, phần lớn là cô ấy nói; nội dung thật là cô ấy đang muốn đổi sang phần backend nhưng chưa dám đề nghị; thống nhất một việc thử trong sprint sau. 10:15 — 1-1 với Duy (Senior đang chững lại): buổi thứ ba nói về cùng một chủ đề; lần này nói thẳng khoảng cách giữa kỳ vọng ở bậc Senior và những gì quý vừa rồi cho thấy, kèm hai ví dụ cụ thể và một mốc kiểm tra sau sáu tuần. Buổi này mất 50 phút và làm bạn kiệt sức. 11:15 — Họp với Trang (Head of Product) về roadmap quý: mang theo con số capacity thật gồm 20% cho công việc kỹ thuật, và từ chối một trong ba yêu cầu bằng cách đưa lựa chọn có giá. 12:00 — Ăn trưa, cố ý ăn với hai người trong team không phải để nói về công việc. 13:15 — Phỏng vấn ứng viên Senior BE, 60 phút, với hai tín hiệu đã định trước; viết đánh giá ngay sau đó 10 phút trong khi còn nhớ. 14:30 — Tuấn (Tech Lead) trình bày một quyết định kiến trúc; bạn không chọn phương án, chỉ hỏi ba câu về chi phí vận hành và rủi ro migration, rồi xác nhận đây là quyết định của cậu ấy. 15:00 — Việc bảo vệ ranh giới: một Director bộ phận khác muốn team bạn nhận một tích hợp gấp; bạn nói không, và đề xuất một cách khác dùng nguồn lực của họ. 15:45 — Ba mươi phút cho việc bị bỏ mãi: viết dự thảo mục tiêu phát triển quý cho từng người, dùng cho calibration cuối quý. 16:30 — Một xung đột nhỏ: hai người bất đồng về việc ai sở hữu một module; bạn không xử nội dung kỹ thuật, chỉ ra quyết định về ranh giới sở hữu và ghi lại. 17:00 — Bản cập nhật tuần cho Director: tiến độ, hai rủi ro, một quyết định cần họ, một tín hiệu về người.

Tổng: khoảng 3 giờ 1-1 và trao đổi về người, 2 giờ họp về ưu tiên và ranh giới, 1 giờ tuyển dụng, 1 giờ viết, 0 giờ code. Điểm khác biệt cốt lõi so với vai Tech Lead: **Tech Lead tối ưu throughput của một chu kỳ delivery; EM tối ưu năng lực của team trong bốn chu kỳ tới** — nên nhiều việc đúng nhất bạn làm trong tuần này sẽ không có tác dụng nào nhìn thấy được trong tuần này.

---

## 8. Director of Engineering

> Ở bậc EM, công cụ chính của bạn là cuộc trò chuyện. Ở bậc Director, công cụ chính của bạn là **cấu trúc** — ai báo cáo cho ai, team nào sở hữu cái gì, tiền và headcount đi đâu, quyết định nào được phép ra ở tầng nào. Đây là bậc đầu tiên mà bạn gần như không còn tác động vào kết quả bằng hành động trực tiếp, và mọi bản năng "vào tay cho nhanh" đã trở thành nguồn gây hại chính.

Hiện tượng nhận ra bậc này nhanh nhất: một Director mới lên nghe ba manager báo cáo rằng mọi thứ "về cơ bản ổn", rồi cuối quý phát hiện hai trong ba team trễ, một người giỏi đã nghỉ, và một quyết định kiến trúc sai đã được nhân bản sang ba service. Không ai nói dối. Tín hiệu chỉ đơn giản là đã bị làm mượt qua hai tầng người, mỗi tầng đều có động lực hợp lý để làm mượt nó.

### Hàm mục tiêu ở bậc này

Thành công ở bậc Director là: **tổ chức của bạn tiếp tục cho ra kết quả đúng khi bạn không có mặt, và tầng manager dưới bạn ra được phần lớn những quyết định mà bạn sẽ ra nếu ở đó.**

Hai vế này đo cùng một thứ từ hai phía. Vế đầu là kiểm chứng ngắn hạn: bạn nghỉ ba tuần, có gì đổ không. Vế sau là kiểm chứng dài hạn và quan trọng hơn: khi một manager của bạn ra quyết định mà bạn không biết trước, tỷ lệ bạn thấy "đúng cái tôi sẽ làm" là bao nhiêu. Nếu tỷ lệ đó thấp, bạn chưa truyền được khung phán đoán — và cách chữa không phải kiểm soát nhiều hơn, mà là làm cho khung phán đoán trở nên rõ và học được.

Đầu ra của bạn không phải một hệ thống, không phải một team, mà là **một cỗ máy tạo ra team**. Cụ thể, năm thứ tổ chức thực sự mua khi thuê một Director:

| Đầu ra | Kiểm chứng được bằng gì |
|---|---|
| **Một cấu trúc tổ chức khớp với kiến trúc và với roadmap** | Số lần một tính năng bình thường cần ba team phối hợp mới xong. Nếu số này cao, cấu trúc sai, không phải người kém. |
| **Một tầng manager có năng lực và có thể thay thế nhau** | Một manager nghỉ đột ngột hai tháng: team đó có tụt không, và ai đỡ được. |
| **Cơ chế thay vì can thiệp** | Với một loại vấn đề đã xảy ra ba lần, lần thứ tư nó được xử lý mà không cần bạn ở trong phòng. |
| **Phân bổ nguồn lực khớp với ưu tiên đã tuyên bố** | Đối chiếu phần trăm engineer thực tế đang làm mỗi hướng với thứ tự ưu tiên nói ra ở đầu quý. Lệch lớn là dấu hiệu bạn đang tuyên bố một chiến lược và tài trợ một chiến lược khác. |
| **Dự báo đủ chính xác để tầng trên lập kế hoạch dựa trên nó** | Những gì bạn nói ở đầu quý về việc gì xong, việc gì rủi ro, có khớp với thực tế cuối quý ở mức nào. |

Chú ý cái không nằm trong danh sách: không có bất kỳ đầu ra nào là một sản phẩm cụ thể, một quyết định kỹ thuật cụ thể, hay một người cụ thể được cứu. Đây là điểm khiến bậc này khó chịu với người vừa lên: bạn làm việc cả tuần và không chỉ ra được một vật gì mình đã tạo ra.

### Phạm vi ảnh hưởng

- **Người**: 3–6 manager báo cáo trực tiếp, tổng 30–80 người (số minh hoạ, thay đổi theo tổ chức). Dưới 3 manager, bạn không đủ khối lượng việc ở tầng cấu trúc và sẽ tự lấp bằng cách làm EM của các team — vai bị trùng, manager của bạn bị rút quyền. Trên 6 manager, thời gian bạn dành cho mỗi người xuống dưới ngưỡng phát hiện được một manager đang chìm.
- **Ngân sách và headcount**: đây là quyền lực thật đầu tiên bạn có mà không cần thuyết phục ai từng lần. Quyết định "team Platform tăng hai người, team Growth giữ nguyên" định hình những gì có thể xảy ra trong bốn quý tới mạnh hơn mọi bài nói về ưu tiên.
- **Chân trời thời gian**: 1–3 năm. Bạn phải giữ đồng thời ba tầng: quý này có giao được không, năm nay tổ chức có đủ năng lực cho roadmap không, và trong hai năm cấu trúc hiện tại sẽ vỡ ở đâu.
- **Hậu quả khi sai**: chậm và khuếch đại. Một quyết định cấu trúc sai không gây lỗi ngay; nó làm cho một loại công việc trở nên đắt hơn 20–30% (số minh hoạ) trong mọi quý tiếp theo, và chi phí đó bị mọi người diễn giải thành "team này làm chậm". Đây là loại hậu quả khó truy về nguyên nhân nhất trong tám bậc — và vì thế cũng là loại hậu quả bạn dễ nhất trong việc không bao giờ bị buộc phải nhận.

Một hệ quả cơ chế đáng nêu riêng: **hầu hết vấn đề đến tay bạn đều đã bị lọc bởi những người có động lực lọc nó** — manager không muốn trông như không kiểm soát được team, engineer không muốn manager của mình bị đánh giá xấu. Không ai hành xử sai; họ đang hành xử hợp lý trong hệ thống bạn thiết kế. Muốn tín hiệu ít bị lọc hơn, bạn phải thay đổi cái làm cho việc lọc trở nên hợp lý, không phải yêu cầu mọi người trung thực hơn.

### Cách ra quyết định

Cơ sở ra quyết định đổi từ **bằng chứng về hành vi của người mình biết** sang **bằng chứng tổng hợp về mẫu hành vi của những người mình không trực tiếp làm việc cùng**. Điều này có ba đặc điểm khó:

1. Bạn nhận dữ liệu qua trung gian, và trung gian đó là người bạn đang đánh giá.
2. Số quan sát của bạn về mỗi người rất thấp — có thể bốn lần một quý — nên bạn dễ khái quát từ một mẫu nhỏ.
3. Số liệu tổng hợp (velocity, incident count, DORA metrics) rất dễ nhìn và rất dễ bị tối ưu ngược. Khi bạn quản lý bằng một chỉ số, chỉ số đó ngừng đo thứ nó từng đo.

Nguyên tắc dùng được ở bậc này: **quyết định bằng số liệu tổng hợp, nhưng hiệu chỉnh mô hình bằng tiếp xúc trực tiếp**. Số liệu cho bạn biết chỗ nào cần nhìn; nó không bao giờ cho bạn biết vì sao. Cơ chế cụ thể: mỗi quý dự phần vào một buổi họp thật của mỗi team với vai người quan sát (không phát biểu, không chốt gì), đọc một số postmortem và một số RFC nguyên bản chứ không đọc bản tóm tắt, và giữ vài cuộc trò chuyện định kỳ với người không báo cáo cho bạn — với ranh giới rõ là bạn nghe để hiệu chỉnh mô hình, không phải để ra quyết định thay manager của họ.

| Loại quyết định | Thuộc Director? | Ghi chú |
|---|---|---|
| Thiết kế tổ chức: bao nhiêu team, ranh giới sở hữu, ai báo cáo cho ai | Có, đây là quyết định trung tâm | Cùng bàn với Staff/Principal về việc ranh giới team có khớp ranh giới hệ thống không |
| Phân bổ ngân sách và headcount giữa các hướng | Có | Đây là nơi chiến lược thật của bạn hiện ra, khác với chiến lược bạn nói |
| Chọn, phát triển, và thay manager | Có | Quyết định đắt nhất và chậm hồi phục nhất ở bậc này |
| Đặt cơ chế: ngưỡng quyết định, nhịp review, chuẩn promotion, quy trình tuyển | Có | Đây là hình thức "làm việc" chính của bậc này |
| Cam kết với tầng executive về phạm vi và thời gian ở mức quý/năm | Có | Kèm nghĩa vụ nói ra mức không chắc chắn |
| Đánh giá hiệu suất của một IC trong team | Không | Thuộc EM. Bạn hiệu chỉnh tiêu chuẩn giữa các manager, không đánh giá thay họ |
| Kiến trúc của một hệ thống cụ thể | Không | Thuộc Staff/Tech Lead. Bạn có quyền hỏi về chi phí, rủi ro, và tính đảo ngược |
| Sprint scope, ai làm ticket nào | Không | Nếu bạn đang ở đây, một tầng nào đó đang không làm việc của nó — và việc bạn cần làm là chữa tầng đó |
| Ưu tiên sản phẩm | Không | Thuộc Product. Bạn cung cấp ràng buộc thật về năng lực và chi phí kỹ thuật |

Ai phản biện bạn: các manager của bạn (qua việc họ có mang vấn đề thật đến hay không), peer ở Product và Data, Principal Engineer (người duy nhất phản biện được bạn về việc cấu trúc tổ chức đang chống lại kiến trúc), CTO, và — chậm nhưng chính xác nhất — **tỷ lệ giữ người ở tầng manager cùng độ khớp của các dự báo bạn đã đưa ra**. Cơ chế đối chiếu dùng được: đầu mỗi quý ghi ra năm dự đoán có thể sai ("việc tách team Payment sẽ làm chậm hai tuần rồi hết"; "Hà sẽ sẵn sàng nhận thêm một team vào quý sau"; "sự phụ thuộc vào đối tác X sẽ trượt"), cuối quý đọc lại và đếm. Không có cơ chế này, bậc Director là bậc dễ nhất trong tám bậc để tự tin mà không có căn cứ, vì feedback loop đủ dài để bạn kịp có một câu chuyện giải thích mọi thứ đã xảy ra.

### Kỹ năng cần phát triển

**1. Đọc tín hiệu đã bị nén qua nhiều tầng.** Vì sao ở bậc này: mọi thứ đến tay bạn đều là bản tóm tắt của một bản tóm tắt, và phép nén luôn bỏ đúng phần khó xử. Cụ thể: học đọc phần bị thiếu chứ không chỉ phần được nói. Bốn dấu hiệu đáng đào sâu — một team đột nhiên ngừng báo rủi ro (rủi ro không mất đi, nó chỉ ngừng được báo); một manager mô tả vấn đề của team bằng tính cách của người khác; một estimate không thay đổi qua ba lần cập nhật liên tiếp; và một người bạn từng nghe tên thường xuyên bỗng không xuất hiện trong bất kỳ báo cáo nào. Cơ chế cần hiểu: đây là một dạng Principal–Agent Problem chuẩn — người báo cáo và bạn không có cùng hàm mục tiêu về tin xấu, nên bạn cần **kênh thông tin không đi qua chuỗi báo cáo** (postmortem nguyên bản, dữ liệu hệ thống, khảo sát ẩn danh, tiếp xúc trực tiếp có ranh giới) để hiệu chỉnh.

**2. Thiết kế cơ chế.** Đây là kỹ năng định nghĩa bậc này, và nó khác hoàn toàn với việc "thêm quy trình". Phân biệt: một quy trình bắt mọi người làm thêm việc để bạn yên tâm; một cơ chế làm cho việc đúng trở thành việc dễ nhất để làm và tự chạy khi bạn không nhìn. Bốn tiêu chí của một cơ chế dùng được: (a) có người sở hữu rõ, không phải "cả tổ chức"; (b) có tín hiệu tự nổi lên khi nó hỏng; (c) không cần bạn trong vòng lặp; (d) có ngày hết hạn hoặc điểm review, vì mọi cơ chế đều thoái hoá thành nghi lễ nếu không ai đóng nó lại. Ví dụ cụ thể: thay vì họp duyệt mọi thay đổi hạ tầng, đặt ngưỡng "thay đổi dưới mức X do team tự quyết và ghi ADR; trên mức X cần một reviewer từ team khác" — và định nghĩa X bằng số, không bằng cảm giác.

**3. Executive communication.** Vì sao là kỹ năng riêng: tầng executive không mua giải pháp kỹ thuật, họ mua sự giảm bất định về tiền, thời gian, và rủi ro. Cấu trúc dùng được cho mọi lần lên tiếng: kết luận hoặc đề nghị trước → hai đến ba phương án kèm giá và hệ quả → mức không chắc chắn và cái gì sẽ làm bạn đổi ý → thứ bạn cần từ họ. Ba lỗi hay gặp nhất ở người từ nhánh kỹ thuật: kể quá trình suy nghĩ trước khi nói kết luận; trình bày rủi ro kỹ thuật bằng ngôn ngữ kỹ thuật ("nợ kỹ thuật ở module thanh toán đang cao") thay vì bằng hệ quả kinh doanh ("với tốc độ hiện tại, đến quý sau mỗi tính năng thanh toán sẽ tốn gấp đôi thời gian, và một sự cố mức nghiêm trọng có khả năng xảy ra trong ba tháng"); và tô hồng để giữ hình ảnh — sai lầm đắt nhất, vì tài sản duy nhất bạn có ở tầng này là **độ tin cậy của các dự báo trước đây**. Ở bối cảnh Việt Nam có thêm một lớp: khi executive là người nước ngoài hoặc là khách hàng nước ngoài, rào cản ngôn ngữ khiến người ta rút ngắn phần "mức không chắc chắn" — đúng phần quan trọng nhất. Cách chữa: viết trước, gửi trước, để phần khó nhất nằm trên văn bản chứ không nằm trong khả năng nói ứng khẩu.

**4. Chịu được việc không biết chi tiết.** Đây là kỹ năng, không phải tính cách, và nó cần cơ chế thay thế chứ không cần ý chí. Vấn đề thật: bạn đã dựa vào việc biết chi tiết để cảm thấy an toàn trong mười năm, và giờ bạn phải chịu trách nhiệm về những thứ bạn chỉ biết ở mức tóm tắt. Cách thay thế cảm giác an toàn đó bằng cái khác: (a) kiểm tra chất lượng phán đoán của người dưới thay vì kiểm tra chi tiết công việc — đọc cách họ lập luận trong một ADR, không đọc code; (b) chọn hai đến ba chỗ để đi sâu có chủ ý mỗi quý và công khai rằng phần còn lại bạn tin tầng dưới; (c) chấp nhận trước một mức sai số — ví dụ mỗi quý có một quyết định ở tầng dưới bạn thấy mình sẽ làm khác nhưng không can thiệp, và ghi lại xem sau đó ai đúng. Bài kiểm tra: khi có người hỏi bạn một câu chi tiết mà bạn không biết, câu trả lời "tôi không biết, Hà biết" phát ra tự nhiên hay phát ra kèm cảm giác mất mặt.

**5. Chọn và phát triển manager.** Vì sao đây là quyết định đắt nhất: một EM tồi làm hỏng năng lực của 6–8 người trong hai đến bốn quý, và bạn nhận tín hiệu chậm vì người bị ảnh hưởng báo cáo cho chính người đó. Ba tín hiệu đáng tin khi chọn manager mới, theo thứ tự: đã tự nguyện làm việc phát triển người khi chưa có chức (mentor thật, không phải onboarding hộ); đã dẫn được một cuộc trò chuyện khó và giữ được quan hệ; và chịu được việc kết quả của mình phụ thuộc vào người khác mà không giật lại việc. Tín hiệu không đáng tin: kỹ thuật giỏi nhất team, thâm niên cao nhất, và "muốn làm quản lý". Về phát triển: một EM mới cần được dạy bằng ca cụ thể chứ không bằng nguyên tắc — cùng chuẩn bị một buổi nói chuyện khó trước khi họ vào, cùng đọc lại sau, và trong sáu tháng đầu chấp nhận rằng bạn đang trả giá bằng tốc độ để mua năng lực.

**6. Thiết kế tổ chức như một bài toán có ràng buộc.** Nguyên lý cần dùng: cấu trúc giao tiếp của tổ chức sẽ in vào cấu trúc hệ thống (quan sát của Conway, một tham chiếu công khai đã được lặp lại rộng rãi trong ngành) — nên khi bạn muốn hai hệ thống ghép chặt bớt lại, công cụ hiệu quả thường là đổi ranh giới sở hữu chứ không phải một dự án refactor. Bốn ràng buộc luôn xung đột với nhau: khớp domain nghiệp vụ, giữ Cognitive Load của mỗi team ở mức chịu được, giảm phụ thuộc chéo, và số người thật bạn đang có. Không phương án nào thoả cả bốn; việc của bạn là chọn phương án mà **phần bị hy sinh là phần bạn sẵn sàng trả giá trong bốn quý tới**, và nói rõ điều đó ra.

### Tín hiệu cho thấy đã sẵn sàng lên bậc tiếp theo

- **Bạn nghỉ ba tuần và không có quyết định nào bị treo chờ bạn.** Đây là bài kiểm tra trực tiếp cho hàm mục tiêu của bậc này.
- **Các manager của bạn ra quyết định bạn không biết trước, và phần lớn khớp với cái bạn sẽ làm** — nghĩa là khung phán đoán đã được truyền, không chỉ được nói.
- **Bạn đã phát triển ít nhất một người từ Senior/Tech Lead thành EM đủ năng lực**, và nói được cụ thể mình đã làm gì.
- **Bạn đã thiết kế lại một phần tổ chức và sống qua hậu quả của nó** — bao gồm cả phần hậu quả xấu mà bạn đã dự đoán và phần bạn không dự đoán.
- **Bạn bắt đầu thấy những vấn đề không giải được trong phạm vi một Director**: chuẩn bậc và lương của toàn công ty tạo ra hành vi sai; ranh giới giữa Engineering và Product làm cho mọi quyết định đều tốn hai lần bàn; chiến lược sản phẩm không có phần công nghệ nên các đầu tư hạ tầng luôn phải xin. Nhìn ra loại vấn đề này là cửa vào tầng VP/CTO.
- **Tầng trên bắt đầu dùng dự báo của bạn để lập kế hoạch của họ** mà không hỏi lại — dấu hiệu độ tin cậy đã tích luỹ đủ.

### Sai lầm thường gặp

**1. Bỏ qua tầng manager để nói trực tiếp với IC.** Cơ chế: bạn từng là người giỏi nhất trong việc gỡ vấn đề kỹ thuật, bạn thấy một vấn đề, và đường ngắn nhất là nhắn thẳng cho người đang làm. Hậu quả kép và cả hai đều nặng: (a) IC nhận hai nguồn ưu tiên và sẽ luôn chọn nguồn có chức cao hơn, nên EM mất khả năng điều phối team của mình; (b) EM học được rằng những vấn đề khó cuối cùng cũng sẽ được bạn xử, nên họ ngừng nhận trách nhiệm về nó. Sau hai quý bạn có một tầng manager làm điều phối viên hành chính, và bạn tự hỏi vì sao không tìm được ai đủ chín để lên thay mình. Dấu hiệu sớm: IC nhắn trực tiếp cho bạn để xin quyết định, và họ thấy điều đó bình thường. Cách chữa mang tính cơ chế, không phải ý chí: khi cần thông tin từ IC, vẫn hỏi, nhưng tuyên bố rõ "tôi đang tìm hiểu, không ra quyết định gì ở đây"; và mọi quyết định phát sinh đều đi ngược qua EM trước khi thành hành động. Ngoại lệ hợp lý: khủng hoảng đang xảy ra, vấn đề an toàn hoặc đạo đức, và trường hợp chính EM là vấn đề — cả ba đều cần nói rõ đây là ngoại lệ.

**2. Quản lý bằng chỉ số mà không bao giờ xuống hiện trường.** Cơ chế: dashboard cho cảm giác kiểm soát với chi phí thời gian gần bằng không, nên nó cạnh tranh thắng mọi hình thức tiếp xúc trực tiếp. Nhưng mọi chỉ số tổng hợp đều là hàm mất mát dữ liệu, và một khi nó được dùng để đánh giá, nó bị tối ưu trực tiếp: velocity tăng bằng cách chia nhỏ ticket, incident giảm bằng cách hạ mức phân loại, cycle time giảm bằng cách mở PR muộn hơn. Hậu quả: bạn có một bảng số đẹp và một tổ chức đang xấu đi, và bạn phát hiện ra qua đơn xin nghỉ việc. Dấu hiệu sớm: bạn không kể được tên một khó khăn cụ thể mà một team đang gặp trong tuần này. Cách chữa: giữ hai đến ba kênh tiếp xúc trực tiếp có nhịp cố định như một mục trong lịch chứ không phải việc làm khi rảnh, và với mỗi chỉ số bạn dùng, viết ra trước một câu "chỉ số này sẽ bị lách bằng cách nào".

**3. Thêm quy trình để giải quyết vấn đề năng lực quản lý.** Cơ chế: một manager không phát hiện được rủi ro sớm; bạn phản ứng bằng cách yêu cầu tất cả các team viết báo cáo rủi ro hàng tuần. Chi phí rơi lên toàn bộ tổ chức, trong đó phần lớn là những người không có vấn đề đó; còn manager kia vẫn không biết cách nhìn ra rủi ro, chỉ biết cách điền một biểu mẫu. Đây là hình thức phổ biến nhất của việc dùng cơ chế sai chỗ: **cơ chế giải quyết vấn đề hệ thống, huấn luyện và thay người giải quyết vấn đề năng lực cá nhân**. Phép thử trước khi thêm bất cứ quy trình nào: vấn đề này xảy ra ở bao nhiêu phần trên tổng số team? Nếu ở một chỗ, đó là việc phải nói với một người. Dấu hiệu sớm: số cuộc họp định kỳ trong tổ chức tăng đều mỗi quý và không có cuộc nào bị đóng lại.

**4. Giữ lại những quyết định nên giao cho manager.** Cơ chế: những quyết định này thường là chỗ bạn tự tin nhất và làm nhanh nhất, nên giao đi cảm giác như làm chậm mọi thứ đi — và trong ngắn hạn thì đúng như vậy. Hậu quả: manager của bạn không tích luỹ được phán đoán vì họ chưa bao giờ phải sống với hậu quả của một quyết định mình ra; đồng thời bạn trở thành nút thắt, và mọi quyết định trong tổ chức bị giới hạn bởi lịch của bạn. Dấu hiệu sớm: có một hàng chờ những việc chỉ bạn quyết được, và nó không ngắn lại. Cách chữa cụ thể hơn "hãy delegate": phân loại quyết định theo tính đảo ngược. Quyết định đảo ngược được thì giao hẳn kể cả khi bạn nghĩ mình sẽ chọn khác — chi phí sai là học phí, và học phí đó rẻ hơn việc có một tầng manager không tự quyết được gì. Quyết định khó đảo ngược (thay đổi cấu trúc, chấm dứt hợp đồng, cam kết ra ngoài) thì giữ, nhưng nói rõ **vì sao** bạn giữ, để nó không bị đọc thành thiếu tin tưởng.

### Một ngày làm việc điển hình

Đặc điểm định nghĩa: **phần lớn thời gian của bạn dùng để làm một việc mà cuối ngày không nhìn thấy kết quả**, và ba thứ có ích nhất bạn làm trong ngày đều là những thứ không ai yêu cầu.

8:30 — Đọc: bản cập nhật tuần của ba EM, một postmortem nguyên bản (không đọc bản tóm tắt), dashboard của tổ chức trong 15 phút. Ghi lại hai chỗ lệch giữa số liệu và những gì được báo cáo. 9:00 — 1-1 với Hà (EM team Payment): 45 phút, phần lớn là cô ấy nói; nội dung thật là cô ấy đang tránh một cuộc trò chuyện với một Senior trong team; bạn không xử ca đó thay cô ấy, bạn cùng cô ấy chuẩn bị ba câu mở đầu và hẹn đọc lại sau buổi nói. 9:45 — 1-1 với một EM khác về một quyết định phân bổ; bạn thấy mình sẽ chọn khác, nhưng quyết định này đảo ngược được nên bạn nêu rủi ro rồi để cậu ấy quyết, và ghi vào sổ để đối chiếu sau. 10:30 — Họp với Trang (Head of Product) và Minh (CTO) về kế hoạch quý sau: mang theo con số năng lực thật theo hướng, không theo đầu người; nói rõ hai trong năm mục tiêu sẽ cần thêm bốn người hoặc phải bỏ, và đề nghị bỏ một cái. 11:30 — Bốn mươi phút cho việc quan trọng nhất trong tuần: viết dự thảo tách team Platform thành hai, gồm cả phần "phương án này hy sinh cái gì" và mốc ba tháng để xem có sai. 12:15 — Ăn trưa với Linh (Staff Engineer) — người duy nhất nói thẳng được với bạn rằng ranh giới team hiện tại đang chống lại kiến trúc. 13:30 — Dự standup của một team với vai người quan sát, không phát biểu; sau đó không nhắn gì cho ai trong team đó, chỉ ghi một câu để hỏi EM của họ vào tuần sau. 14:15 — Calibration giữa ba EM về chuẩn Senior: mục tiêu không phải chốt từng người mà là làm cho ba người dùng cùng một thước; bạn phát hiện một EM đang dùng thước dễ hơn hai người kia và xử lý điều đó ở đây thay vì ở kết quả cuối quý. 15:30 — Phỏng vấn ứng viên EM, vòng cuối; hai tín hiệu định trước là "đã dẫn cuộc trò chuyện khó" và "chịu được việc không giật lại việc". 16:30 — Một EM xin bạn quyết một việc thuộc thẩm quyền của họ; bạn trả lại kèm câu hỏi "cậu nghiêng về phương án nào và vì sao", và chỉ nói ý mình sau khi cậu ấy nói xong. 17:00 — Viết: bản cập nhật cho CTO gồm tiến độ theo hướng, hai rủi ro kèm mức không chắc chắn, một quyết định cần ở tầng trên, một tín hiệu về tầng manager. 17:30 — Ghi ba dòng vào sổ dự đoán của quý.

Tổng: khoảng 2 giờ với các manager, 2 giờ họp về ưu tiên và nguồn lực, 1 giờ tuyển dụng, 1,5 giờ đọc và viết, 1 giờ tiếp xúc trực tiếp không qua chuỗi báo cáo, 0 giờ code, 0 quyết định kỹ thuật. Điểm khác biệt cốt lõi so với bậc EM: **EM tối ưu năng lực của một team trong bốn quý tới; Director tối ưu khả năng của tổ chức tự tạo ra team tốt** — nên việc giá trị nhất trong ngày hôm nay (bốn mươi phút viết dự thảo tách team) là việc không ai hỏi bạn về nó, không có deadline, và sẽ chỉ cho biết kết quả sau ba quý.

---

## Bốn chuyển dịch khó nhất

Tám bậc ở trên không cách nhau đều. Giữa Junior và Mid-level, giữa Mid-level và Senior, cái thay đổi là *mức độ*: phạm vi rộng hơn, bài toán khó hơn, nhưng hàm mục tiêu vẫn cùng một dạng. Bạn tiến bằng cách làm tốt hơn thứ bạn đang làm.

Có bốn điểm trong chuỗi đó mà **hàm mục tiêu bị viết lại**, và ở đúng bốn chỗ này, "làm tốt hơn thứ mình đang làm" là chiến lược sai — trong nhiều trường hợp là chiến lược gây hại, vì nó lấy đi thời gian khỏi thứ thật sự cần thay đổi. Phần này mổ xẻ bốn điểm rẽ đó theo cùng bốn câu hỏi: cái gì thay đổi, vì sao khó về mặt cơ chế, sai lầm điển hình, và cụ thể phải làm gì trong sáu tháng trước khi bước qua.

Lý do phải chuẩn bị trước sáu tháng, chứ không phải học sau khi nhận vai: ở cả bốn điểm rẽ, chi phí của việc học tại chỗ không do bạn trả. Một Tech Lead mới học cách đàm phán scope bằng cách để team làm quá tải hai sprint; một EM mới học cách nói chuyện khó bằng cách để một người mất thêm hai quý. Sáu tháng là khoảng đủ để tích luỹ vài lần thử ở quy mô nhỏ, nơi hậu quả còn rẻ.

### (a) Senior Engineer → Tech Lead: từ đầu ra cá nhân sang đầu ra hệ thống

**Cái gì thay đổi.** Trước đây bạn được đo bằng thứ bạn tạo ra; giờ bạn được đo bằng thứ *team* tạo ra, kể cả trong những tuần bạn không viết dòng nào. Ba hệ quả cụ thể:

- **Đầu ra đổi đơn vị.** Từ "code và quyết định kỹ thuật của tôi" sang "throughput của một chu kỳ delivery": scope có đúng không, thứ tự có đúng không, ai làm gì, ai đang bị chặn, chất lượng có ở mức chịu được lâu dài không.
- **Cách bạn tạo giá trị lớn nhất đổi chỗ.** Ở bậc Senior, giá trị lớn nhất của bạn nằm trong hai giờ tập trung sâu. Ở vai Tech Lead, nó thường nằm trong mười lăm phút bạn gỡ một người đang bế tắc, hoặc trong một cuộc nói chuyện với PO làm cho hai tuần công việc vô nghĩa không bao giờ được bắt đầu.
- **Bạn có trách nhiệm mà không có thẩm quyền.** Tech Lead là một vai, không phải một bậc: bạn chịu trách nhiệm về delivery nhưng không đánh giá hiệu suất, không quyết lương, không quyết ai vào ai ra. Nguồn sức mạnh duy nhất của bạn là uy tín kỹ thuật cộng phán đoán được kiểm chứng.

**Vì sao khó — cơ chế.** Ba cơ chế cộng lại, và chúng độc lập nên chữa một cái không giúp gì cho hai cái còn lại.

Thứ nhất, **vòng phản hồi bị kéo dài và làm nhiễu**. Bạn đã sống năm đến mười năm với vòng phản hồi vài phút: viết, chạy test, biết mình đúng hay sai. Đầu ra mới của bạn có vòng phản hồi hàng sprint, và tín hiệu luôn lẫn — team giao đúng hạn có thể vì bạn chia việc tốt, hoặc vì scope vốn dễ, hoặc vì hai người làm thêm cuối tuần mà không nói. Bộ não bạn không có cách phân biệt, nên nó quay về đo cái nó đo được: số dòng code bạn viết. Đây là lý do cơ học vì sao Tech Lead mới thường tự nhận thêm task — không phải vì họ không hiểu vai, mà vì họ đang thiếu tín hiệu và tự cấp cho mình một nguồn tín hiệu giả.

Thứ hai, **chi phí ngắn hạn của việc làm đúng là dương và hiện ra ngay**. Để Vy tự làm một việc bạn làm nhanh hơn ba lần khiến sprint này chậm hơn thật. Lợi ích (Vy làm được việc đó mà không cần bạn từ quý sau) xuất hiện sau, ở một chỗ không ai gán công cho bạn. Mọi lựa chọn đúng ở vai này đều có hình dạng như vậy: chi phí tức thì, rõ ràng, thuộc về bạn; lợi ích trễ, khuếch tán, không truy được. Người không chuẩn bị trước cho cấu trúc chi phí này sẽ tự nhiên chọn sai.

Thứ ba, **quan hệ đồng nghiệp thay đổi không đối xứng**. Bạn vẫn nghĩ mình là một trong nhóm; những người kia đã bắt đầu đọc câu nói của bạn như một quyết định. Một câu bạn nói lúc uống cà phê — "cách này chắc không ổn" — trước đây là một ý kiến ngang hàng, giờ có thể làm một người bỏ hai ngày công việc. Bạn không được thông báo về sự thay đổi này; bạn phát hiện ra nó qua hậu quả.

**Sai lầm điển hình.** Giữ nguyên lượng công việc kỹ thuật cũ rồi làm thêm phần lead vào buổi tối và cuối tuần. Điều này hoạt động được khoảng tám đến mười tuần, và vì nó hoạt động được, nó thuyết phục bạn rằng đó là cách đúng. Sau đó nó vỡ theo cùng một trình tự mỗi lần: task của bạn trên critical path bị trễ (vì lịch của bạn bị gián đoạn bởi những việc không dời được), không ai dám nhắc, những người trong team dần ngừng mang vấn đề đến vì thấy bạn đã quá tải, và bạn mất luôn nguồn thông tin cần để làm vai của mình. Hình thức nhẹ hơn nhưng phổ biến hơn: vẫn nhận review mọi PR như một chốt kiểm soát, biến mình thành nút thắt của team và gọi đó là bảo vệ chất lượng.

Sai lầm thứ hai, ít được nói tới: **tưởng rằng vai này là phần thưởng cho việc giỏi kỹ thuật, nên coi việc mất thời gian code là bất công**. Người ở trạng thái đó thường làm phần lead ở mức tối thiểu, đủ để không bị phê, và ở lại vai đó nhiều năm với kết quả là một team không bao giờ khoẻ hơn.

**Cách chuẩn bị trước sáu tháng.** Toàn bộ danh sách này có thể làm khi bạn vẫn là Senior, không cần ai bổ nhiệm.

| Việc làm trong 6 tháng | Nó luyện đúng cơ chế nào |
|---|---|
| Nhận dẫn một epic nhỏ 4–6 tuần có 2–3 người tham gia, và cố ý không tự làm phần khó nhất | Luyện chịu chi phí ngắn hạn của việc để người khác làm |
| Mỗi tuần dành 2 giờ cho việc unblock người khác và ghi lại nó vào một danh sách riêng | Tạo nguồn tín hiệu thay thế cho tín hiệu "code đã viết" |
| Viết 3 ADR cho quyết định của người khác trong team, chỉ đóng vai người ghi và người đặt câu hỏi | Luyện việc tạo giá trị mà không phải người quyết |
| Dự một buổi refinement với vai người đàm phán scope, chuẩn bị trước ba lựa chọn có giá thay vì một lời từ chối | Luyện phần khó nhất của vai: nói không bằng cách đưa lựa chọn |
| Xin phản hồi từ 3 người trong team bằng một câu cụ thể: "khi tôi nói về giải pháp, cậu thấy khó phản biện ở điểm nào?" | Đo mức không đối xứng đã hình thành, trước khi nó gây hại |
| Đọc lại một dự án đã hỏng trong quá khứ và viết ra: phần nào hỏng vì kỹ thuật, phần nào hỏng vì scope, thứ tự, hoặc thông tin không chảy | Luyện phân biệt bài toán kỹ thuật với bài toán tổ chức-kỹ thuật |

Tiêu chí biết là đã sẵn sàng: có một tuần trong sáu tháng đó mà bạn không đóng góp gì đáng kể bằng code nhưng chỉ ra được ba việc cụ thể đã xảy ra vì bạn, và bạn không cảm thấy tuần đó là tuần lãng phí.

### (b) Tech Lead → chọn nhánh: IC hay Manager

**Cái gì thay đổi.** Đây là điểm rẽ duy nhất trong bốn điểm mà thứ thay đổi không phải công việc, mà là **tập hợp các lựa chọn còn lại trong 5–10 năm tới**. Vai Tech Lead là một trạng thái lai: bạn vừa làm kỹ thuật sâu, vừa làm việc về người và về điều phối. Trạng thái lai đó dễ chịu và có thể duy trì vài năm, nhưng nó không mở rộng được — vì phần kỹ thuật sâu và phần tổ chức đều đòi hỏi thời gian liên tục để đi xa hơn, và bạn chỉ có một quỹ thời gian.

Cụ thể, hai nhánh yêu cầu hai loại tích luỹ khác nhau, và cả hai đều cần nhiều năm liền mạch:

- **Nhánh IC (Staff → Principal)** tích luỹ độ sâu kỹ thuật hiện thời, số lượng quyết định kiến trúc mà bạn đã sống qua hậu quả, và uy tín kỹ thuật với peer. Loại tài sản này mất giá nếu ngừng dùng: hai năm không chạm vào hệ thống thật là đủ để bạn mất quyền phát biểu về chi tiết.
- **Nhánh Manager (EM → Director)** tích luỹ số ca về người bạn đã xử lý trọn vòng, số lần tuyển đúng và tuyển sai mà bạn đã thấy kết quả, và khả năng đọc động lực của tổ chức. Loại tài sản này bền hơn nhưng cần thời gian thật, không thể tích luỹ trong 20% thời gian.

**Vì sao khó — cơ chế.** Cái làm điểm rẽ này khó không phải thiếu thông tin về hai nhánh, mà là bốn cơ chế bóp méo quyết định:

1. **Tín hiệu từ tổ chức lệch một chiều.** Ở phần lớn công ty Việt Nam, nhánh Manager có bậc rõ ràng hơn, mức lương cao hơn ở cùng số năm, và được nhìn thấy nhiều hơn. Nhánh IC ở nhiều nơi dừng ở Senior và không có gì phía trên. Bạn không đang chọn giữa hai lựa chọn ngang giá — bạn đang chọn giữa một lựa chọn được tài trợ và một lựa chọn có thể không tồn tại ở công ty hiện tại.
2. **Bạn phải quyết trước khi có dữ liệu.** Cách duy nhất biết mình có phù hợp với nhánh Manager là làm thử một hai năm. Nhưng chi phí của việc thử không chỉ do bạn trả: nếu không phù hợp, 6–8 người đã sống qua hai năm dưới một manager không phù hợp.
3. **Áp lực khan hiếm ở phía tổ chức.** Khi một team cần manager và bạn là người sẵn có nhất, câu hỏi được đặt ra không phải "cậu muốn nhánh nào" mà "cậu nhận team này nhé". Đó là một câu hỏi khác, và trả lời câu hỏi đó bằng cách nghĩ mình đang trả lời câu đầu là cách người ta rẽ nhánh mà không nhận ra mình đã rẽ.
4. **Sự mơ hồ của trạng thái lai làm bạn trì hoãn.** Vì Tech Lead làm được cả hai phần, không có ngày nào bạn *buộc* phải chọn. Kết quả thường gặp: bốn năm ở trạng thái lai, và cả hai loại tài sản đều tích luỹ ở mức nửa vời.

**Sai lầm điển hình.** Rẽ nhánh Manager vì đó là con đường duy nhất được tổ chức mở, rồi diễn giải quyết định đó thành một sở thích của bản thân. Cách nhận ra mình đang mắc sai lầm này: bài kiểm tra ở cuối mục Engineering Manager — nếu lương và title của hai nhánh bằng nhau, bạn chọn nhánh nào. Nếu câu trả lời đổi khi bỏ yếu tố lương và title ra, thì cái bạn muốn không phải nhánh Manager, mà là thứ mà chỉ nhánh Manager ở công ty này cấp cho bạn — và có hai cách khác để lấy thứ đó: đàm phán một bậc IC thật ở công ty hiện tại, hoặc đổi sang công ty có bậc đó.

Sai lầm thứ hai: coi đây là quyết định một lần và không thể đảo. Nó không phải, nhưng chi phí đảo tăng theo thời gian — điều đó được xử lý ở mục (c).

**Cách chuẩn bị trước sáu tháng.** Mục tiêu của sáu tháng này không phải chọn, mà là **tạo ra dữ liệu về chính mình ở quy mô rẻ**.

- Xin nhận hai việc thuộc nhánh Manager mà không đổi vai: tham gia ba vòng phỏng vấn với vai người ra kết luận (không phải người hỏi kỹ thuật), và dẫn onboarding trọn vẹn cho một người mới trong sáu tuần.
- Xin nhận một việc thuộc nhánh IC mà không đổi vai: chọn một bài toán cắt qua hai team, tự định nghĩa nó, viết một RFC, và mang nó đi thuyết phục người ở team khác — nơi bạn không có bất kỳ thẩm quyền nào.
- Ghi lại năng lượng, không ghi kết quả. Cuối mỗi tuần trong sáu tháng, viết một dòng: việc nào tuần này làm bạn kiệt sức theo kiểu không hồi phục, việc nào làm bạn mệt nhưng muốn làm lại. Đây là dữ liệu duy nhất về sự phù hợp mà không ai có thể cấp cho bạn từ bên ngoài, và nó chỉ đáng tin khi được ghi tại thời điểm, không phải khi nhớ lại.
- Nói chuyện với hai người đã rẽ mỗi nhánh được ít nhất ba năm, và hỏi đúng một câu: "phần nào của công việc này anh không lường trước được?"

Bộ câu hỏi chẩn đoán chi tiết cho quyết định này nằm ở mục [Chọn nhánh: IC track vs Manager track](#chọn-nhánh-ic-track-vs-manager-track).

### (c) IC → Engineering Manager

**Cái gì thay đổi.** Đây là chuyển dịch có độ gián đoạn lớn nhất trong tám bậc, vì nó là chỗ duy nhất mà hàm mục tiêu không chứa bất kỳ thành phần nào là đầu ra của chính bạn. Chi tiết đã được mổ xẻ ở mục 7; ở đây chỉ nêu ba thứ thay đổi mà người chuẩn bị hay bỏ sót:

- **Đơn vị công việc đổi từ vấn đề sang con người.** Một bug được sửa xong và biến mất. Một người thì không: cùng một chủ đề về động lực có thể quay lại trong ba quý liên tiếp với ba hình dạng khác nhau, và mỗi lần bạn phải xử lý lại từ trạng thái mới.
- **Bạn mất quyền có tâm trạng.** Sự lo lắng của bạn lan xuống team với hệ số phóng đại, nên phần lớn cảm xúc của bạn phải được xử lý ở nơi khác — với Director của bạn, với peer EM, hoặc bên ngoài công việc.
- **Thông tin bạn nhận được bắt đầu bị lọc vì bạn.** Ngày trước đồng nghiệp nói với bạn mọi thứ. Từ tuần bạn nhận vai, họ tính đến việc bạn là người viết đánh giá hiệu suất của họ. Đây không phải sự thiếu tin tưởng cá nhân; đây là hành vi hợp lý trước một thay đổi về cấu trúc động lực.

**Vì sao khó — cơ chế.** Ngoài những cơ chế đã nêu ở mục 7, có một cơ chế riêng cho *quá trình chuyển đổi* mà ít được nói: **bạn bị đo bằng thước cũ trong khi đang học kỹ năng mới**. Trong 6–12 tháng đầu, người xung quanh — và chính bạn — vẫn còn mô hình về bạn như một người kỹ thuật giỏi. Khi bạn không còn giải được vấn đề kỹ thuật nhanh như trước, tín hiệu bạn nhận là "mình đang tệ đi", trong khi thứ bạn thật sự đang xây (khả năng chẩn đoán vấn đề về người, khả năng dẫn cuộc trò chuyện khó) chưa có bất kỳ thang đo nào để hiện ra. Đây là lý do cơ học vì sao năm đầu làm EM được nhiều người mô tả là năm mất tự tin nhất trong nghề, kể cả những người sau đó làm rất tốt.

Cơ chế thứ hai: **những sai lầm ở bậc này có nạn nhân cụ thể**. Một quyết định kiến trúc sai làm hệ thống chậm. Một quyết định sai về người làm một con người mất một năm trong đời làm việc của họ. Loại trách nhiệm này không có ở bất kỳ bậc IC nào, và không ai chuẩn bị cho nó bằng cách đọc.

**Sai lầm điển hình.** Ba cái, theo thứ tự phổ biến: giữ lại phần kỹ thuật trên critical path (mục 7, sai lầm 1); micromanage vì mất tín hiệu tiến độ (mục 7, sai lầm 2); và — cái ít được nhận diện nhất — **coi việc nhận vai EM là quyết định không thể xét lại**, nên khi phát hiện không phù hợp thì chịu đựng thêm hai năm thay vì đổi hướng. Sai lầm cuối này đắt cho cả hai phía và nó xứng đáng được xử lý riêng.

#### Quay lại làm IC không phải thất bại — và đây là cơ chế

Đây là mục mà phần lớn tài liệu về career ladder viết một câu cho có, nên nó cần viết thật. Có bốn lập luận, và không lập luận nào trong đó là lời an ủi.

**Thứ nhất, vì EM là một hàm mục tiêu khác, không phải một mức cao hơn.** Toàn bộ chương này được xây trên quan sát đó. Nếu bậc EM là bậc trên của bậc Senior/Staff, thì rời nó là đi xuống. Nhưng nó không phải: đầu ra khác, kỹ năng khác, loại năng lượng bị tiêu hao khác, và một người có thể ở rất cao trên nhánh IC mà không bao giờ quản lý ai. Rời một hàm mục tiêu để về một hàm mục tiêu khác là **đổi trục**, không phải tụt bậc. Sự nhầm lẫn ở đây gần như hoàn toàn là sản phẩm của việc vẽ career ladder thành một cầu thang một chiều.

**Thứ hai, vì chi phí của việc ở lại lớn hơn nhiều so với chi phí đổi hướng, và phần lớn chi phí đó rơi lên người khác.** Một EM không phù hợp với vai không phải một người kém — thường là ngược lại, họ đủ giỏi để giữ mọi thứ không sụp. Nhưng cái giá được trả bằng: 6–8 người có một manager tránh cuộc trò chuyện khó nên không ai được nói thật về hiệu suất của mình; những quyết định về người bị hoãn; và một tầng Tech Lead không được phát triển. So sánh trực tiếp: một người rời vai EM sau 18 tháng gây một quý xáo trộn; một người ở lại vai đó năm năm vì sợ bị coi là thất bại gây ra một tổ chức không phát triển được ai trong năm năm.

**Thứ ba, vì người đã làm EM rồi quay lại IC là một tài sản đặc biệt của tổ chức, không phải một người bị hạ cấp.** Họ mang lại ba thứ mà một Staff Engineer chưa từng quản lý ai thường không có: hiểu vì sao một quyết định roadmap trông vô lý từ dưới lại hợp lý từ trên, nên họ tranh luận với business bằng ngôn ngữ đúng; biết cách nói chuyện với một người đang có vấn đề mà không làm họ phòng thủ, nên họ mentor tốt hơn; và biết ràng buộc thật của manager, nên đề xuất kỹ thuật của họ thường kèm phương án khả thi về nguồn lực. Trong nhiều tổ chức, chính những người này trở thành Staff/Principal tốt nhất.

**Thứ tư, vì tín hiệu bạn tạo ra khi việc quay lại được xử lý đúng có giá trị cơ chế lớn hơn bản thân ca đó.** Đây là điểm quan trọng nhất với một Director. Nếu tổ chức của bạn xử lý một lần quay lại như một thất bại, bạn vừa dạy cho tất cả những người còn lại rằng thử vai quản lý là một cửa một chiều — và hệ quả trực tiếp là những người có tố chất nhất, tức những người biết cân nhắc nghiêm túc về việc mình có phù hợp không, sẽ là những người từ chối thử. Bạn còn lại một tập ứng viên nghiêng về những người ít cân nhắc hơn. Ngược lại, nếu việc quay lại được xử lý như một quan sát về sự phù hợp, chi phí thử vai quản lý giảm, và số người dám thử tăng.

Một lưu ý bối cảnh Việt Nam, nói thẳng: ở nơi không có bậc IC thật phía trên Senior, việc quay lại làm IC đi kèm mất lương thật, không chỉ mất mặt. Trong trường hợp đó, câu "đây không phải thất bại" là đúng về bản chất nhưng không đủ để giải quyết tình huống. Việc của Director là làm hai thứ trước khi cuộc trò chuyện này xảy ra ở tổ chức của mình: tạo ra ít nhất một bậc IC thật có mức lương chồng lấn với bậc EM, và nói ra điều đó *trước khi* bổ nhiệm ai làm EM, không phải sau.

#### Script hội thoại: một EM nói với Director rằng muốn quay lại làm IC

Bối cảnh: Hà làm EM một team 7 người ở một product company Việt được 16 tháng. Trước đó là Senior BE 6 năm. Team đang giao ổn, không có sự cố lớn. Sơn là Director của Hà.

**Hà mở đầu:**

> "Em muốn nói với anh một việc em đã nghĩ khoảng ba tháng. Em nghĩ em muốn quay lại làm IC. Không phải vì team hay vì anh — team đang ổn. Em chỉ thấy phần công việc làm em kiệt nhất là phần chiếm phần lớn thời gian, và phần làm em muốn đi làm thì gần như không còn. Em không giỏi lên ở nó theo cách em từng giỏi lên ở kỹ thuật, và em đã thử đủ lâu để không nghĩ đây là chuyện thích nghi."

**Phản ứng sai:**

> Sơn: "Ơ, sao lại thế. Hà đang làm tốt mà, anh vừa nói với Minh là Hà là một trong hai EM chắc tay nhất. Em đừng quyết vội. Ai làm EM năm đầu cũng thấy thế, anh cũng từng thế, rồi qua. Mà em nghĩ kỹ chưa — bậc IC của mình cao nhất là Senior, quay lại thì lương em sẽ phải điều chỉnh, rồi mọi người sẽ hỏi vì sao. Em cứ giữ đến hết quý đi, xong quý này team qua release lớn rồi mình nói lại. Với lại giờ anh mà mất em thì không có ai nhận team này cả, Tuấn thì chưa đủ chín."

Bốn thứ đã xảy ra trong đoạn đó: chẩn đoán được thay bằng một giả định ("ai cũng thế rồi qua"), thông tin về lương được dùng như một lực cản chứ như một thông tin, quyết định bị hoãn vô hạn định bằng một mốc mềm, và ràng buộc của tổ chức được đặt lên vai Hà như một nghĩa vụ đạo đức. Hà sẽ rút lại đề nghị, ở thêm chín tháng, và rời công ty — vì con đường duy nhất còn lại để đổi hàm mục tiêu là đổi công ty.

**Phản ứng đúng:**

> Sơn: "Cảm ơn Hà đã nói, và nói sớm chứ không phải nói lúc đã có offer chỗ khác — anh coi đây là việc Hà làm đúng vai của mình. Anh sẽ không trả lời hôm nay, và anh cũng không muốn Hà quyết hôm nay. Nhưng anh muốn tách hai câu hỏi khác nhau ra, vì trả lời gộp thì dễ chọn sai.
>
> Câu thứ nhất: cái làm Hà kiệt là *bản chất của vai*, hay là *điều kiện hiện tại của vai này ở đây*? Ví dụ cụ thể — nếu bảy người xuống còn năm, nếu Hà không phải làm phần báo cáo và điều phối chéo team đang chiếm chừng một ngày mỗi tuần, nếu có Tuấn giữ vai Tech Lead thật để Hà không phải là người duy nhất trả lời câu hỏi kỹ thuật — thì bức tranh có đổi không? Anh hỏi không phải để giữ Hà lại. Anh hỏi vì nếu vấn đề là điều kiện thì Hà sẽ mang nó theo sang chỗ khác, và anh cũng cần biết để không đặt người tiếp theo vào đúng cái bẫy đó.
>
> Câu thứ hai, và đây là câu quan trọng: nếu lương và bậc của hai nhánh bằng nhau, Hà chọn nhánh nào? Anh cần câu trả lời của Hà cho câu này ở dạng đã bỏ hết yếu tố tiền ra, vì phần tiền là việc của anh, không phải của Hà.
>
> Về việc quay lại: anh nói rõ ngay để nó không lơ lửng. Đây không phải hạ cấp và anh sẽ không để nó được kể như hạ cấp — mười sáu tháng vừa rồi Hà tích luỹ được thứ mà không Staff Engineer nào chưa từng quản lý ai có: Hà hiểu vì sao roadmap trông vô lý từ dưới, và Hà biết cách nói chuyện với người đang bế tắc. Ở nhánh IC, hai thứ đó là đòn bẩy, không phải hành lý.
>
> Còn phần khó thật, anh nói thẳng: bậc IC cao nhất hiện tại của mình là Senior, nên nếu chỉ chuyển vai thì có vấn đề về lương và anh không muốn giải quyết nó bằng cách bảo Hà chấp nhận. Anh có hai tuần để làm việc với Minh về việc mở một bậc Staff thật — đây là việc anh cần làm dù Hà đi hay ở, và ca của Hà chỉ làm nó thành cấp thiết hơn. Anh sẽ cho Hà biết kết quả, kể cả khi kết quả là không được.
>
> Đề nghị cụ thể: hai tuần nữa mình nói lại. Trong hai tuần đó, Hà ghi lại theo ngày việc nào làm Hà kiệt và việc nào không — anh không cần bản phân tích, anh cần dữ liệu thô. Và bất kể kết luận là gì, anh cần Hà làm một việc: viết ra ba thứ Hà đã học mà người nhận team sau Hà nên biết. Nếu Hà chuyển vai, mình sẽ có 8–10 tuần chuyển giao, và mình sẽ cùng thống nhất trước cách nói việc này với team, bằng ngôn ngữ đúng, do Hà chọn."

**Mổ xẻ khác biệt:**

| Trục | Phản ứng sai | Phản ứng đúng |
|---|---|---|
| **Giả định về nguyên nhân** | Giả định sẵn là "chưa thích nghi", nên không cần chẩn đoán | Chưa giả định gì; tách "bản chất vai" và "điều kiện hiện tại" thành hai câu hỏi kiểm tra được |
| **Vị thế trong cuộc trò chuyện** | Thuyết phục — mục tiêu là làm Hà đổi ý | Cùng chẩn đoán — mục tiêu là ra kết luận đúng, kể cả khi kết luận đó là Hà chuyển vai |
| **Thứ được bảo vệ** | Sự thuận tiện ngắn hạn của Sơn (không phải tìm EM mới) | Chất lượng của quyết định, và tín hiệu mà tổ chức nhận được từ cách ca này được xử lý |
| **Cách dùng thông tin về lương** | Dùng như lực cản để Hà tự rút lại | Nêu như một ràng buộc thật của tổ chức, và nhận nó là việc của Director phải giải |
| **Xử lý mốc thời gian** | Mốc mềm, dời được vô hạn ("hết quý này rồi nói lại") | Mốc cứng hai tuần, có việc cụ thể cần làm trong hai tuần đó, có cam kết phản hồi kể cả khi kết quả xấu |
| **Thông tin thu được cho tổ chức** | Không có. Nguyên nhân thật không bao giờ được biết, nên nó lặp lại với người tiếp theo | Biết được phần nào của vai đang bị thiết kế sai (điều phối chéo team, thiếu Tech Lead thật, team quá lớn) — dùng được cho mọi EM khác |
| **Hệ quả với chính Hà** | Ở thêm 9 tháng, mất niềm tin, rời công ty; đơn xin nghỉ được đọc là "bị burnout" | Chuyển vai có chuyển giao, hoặc ở lại vì điều kiện đã đổi thật; cả hai kết cục đều dùng được |
| **Hệ quả với những người khác** | Cả tổ chức học được rằng thử vai quản lý là cửa một chiều → người cân nhắc nghiêm túc nhất sẽ không thử | Chi phí thử vai quản lý giảm → tập ứng viên EM rộng hơn và tự chọn tốt hơn |
| **Ai chịu phần mất** | Hà chịu toàn bộ; Sơn không mất gì trong quý này | Sơn chịu phần khó (tìm EM mới, đàm phán bậc IC với CTO) — đúng vai của Director |

Một điều kiện biên cần nói rõ để mục này không bị đọc thành khẩu hiệu: **không phải mọi lần muốn quay lại đều nên được chấp thuận ngay**. Ba trường hợp cần xử lý khác: nếu người đó mới nhận vai dưới sáu tháng, dữ liệu chưa đủ và câu trả lời đúng thường là "hãy làm đủ hai chu kỳ hiệu suất trước khi kết luận"; nếu lý do thật là đang tránh một cuộc trò chuyện khó cụ thể, thì việc cần làm là dẫn họ qua cuộc trò chuyện đó trước, vì chuyển vai để tránh nó sẽ mang cùng vấn đề sang nhánh IC dưới hình dạng khác; và nếu đang ở giữa một khủng hoảng mà việc chuyển vai lập tức gây hại thật cho nhiều người, thì thoả thuận một mốc cứng có ngày cụ thể là hợp lý — nhưng phải là ngày, không phải "sau khi ổn".

**Cách chuẩn bị trước sáu tháng (cho chuyển dịch IC → EM).**

- Dẫn trọn một trường hợp phản hồi khó ở quy mô nhỏ: nói với một người trong team về một hành vi cụ thể đang gây hậu quả cụ thể, và kiểm tra bằng câu hỏi ở mục 7 — hôm sau người đó có nói lại được chính xác điều cần thay đổi không.
- Tham gia 4–5 vòng phỏng vấn với vai người viết kết luận, và sau ba tháng đọc lại đánh giá của mình về những người đã vào làm.
- Nhận mentor chính thức cho một người trong hai quý, có mục tiêu viết ra và có mốc kiểm tra — để đo xem việc người khác tiến bộ có thật sự làm bạn thoả mãn hay không.
- Dành một quý cố ý không nhận việc kỹ thuật khó nhất trong team, và ghi lại cảm giác. Đây là bài kiểm tra rẻ nhất cho cái giá lớn nhất của nhánh này.
- Thoả thuận trước với Director của bạn hai điều, bằng văn bản: đường quay lại nhánh IC trông như thế nào, và bậc IC nào tồn tại thật ở công ty này. Nếu câu trả lời cho câu thứ hai là "cao nhất là Senior", bạn đã biết một thông tin quan trọng trước khi quyết định thay vì sau.

### (d) Engineering Manager → Director: từ quản lý người sang quản lý manager và cơ chế

**Cái gì thay đổi.** Ở bậc EM, công cụ chính là cuộc trò chuyện, và bạn biết từng người trong phạm vi của mình đủ rõ để chẩn đoán. Ở bậc Director, phạm vi vượt ngưỡng mà một người có thể biết trực tiếp, nên công cụ chính đổi sang cấu trúc và cơ chế. Ba thứ đổi cụ thể:

- **Từ can thiệp sang thiết kế điều kiện.** Ở bậc EM bạn sửa vấn đề bằng cách nói với người đang gặp nó. Ở bậc Director, cùng một loại vấn đề xuất hiện ở ba team, và cách xử lý đúng thường là đổi cái sinh ra nó — ranh giới sở hữu, ngưỡng quyết định, chuẩn promotion — chứ không phải nói với ba người.
- **Từ đầu ra là năng lực của IC sang đầu ra là năng lực của manager.** Đây là một tầng gián tiếp nữa, và mỗi tầng gián tiếp làm vòng phản hồi dài thêm và tín hiệu nhiễu thêm.
- **Từ biết người sang biết mẫu.** Bạn không còn nhớ được ai đang gặp gì. Thứ bạn phải nhìn thấy là mẫu: loại vấn đề nào lặp lại, ở đâu, và cơ chế nào đang sinh ra nó.

**Vì sao khó — cơ chế.** Ba cơ chế, và chúng khác về bản chất với những cơ chế ở ba chuyển dịch trên.

Thứ nhất, **mất tiếp xúc trực tiếp với hiện thực mà vẫn chịu trách nhiệm về nó**. Mọi thông tin đến qua trung gian, và trung gian là người bạn đang đánh giá — một Principal–Agent Problem chuẩn. Ở bậc EM bạn có thể tự kiểm tra bằng cách đi hỏi. Ở bậc Director, việc đi hỏi trực tiếp lại chính là hành vi phá thẩm quyền của manager, nên công cụ tự nhiên nhất của bạn bị vô hiệu hoá bởi vai của bạn.

Thứ hai, **vòng phản hồi của cơ chế dài hơn vòng phản hồi của can thiệp một bậc độ lớn**. Một cuộc trò chuyện cho tín hiệu trong một tuần. Một thay đổi cấu trúc cho tín hiệu trong hai đến bốn quý — và trong khoảng đó có quá nhiều thứ khác thay đổi để bạn quy kết chắc chắn. Hệ quả hành vi: Director mới thường thay cấu trúc quá nhanh, chưa kịp thấy kết quả của lần trước đã thay lần sau, và tổ chức chịu chi phí xáo trộn liên tục mà không ai học được gì.

Thứ ba, **kỹ năng cũ vẫn dùng được vừa đủ để bạn không bị buộc phải học kỹ năng mới**. Đây là cơ chế âm thầm nhất. Một Director mới vẫn có thể xử lý ca về người rất tốt, và mỗi lần làm thế đều thấy có ích thật. Nhưng cứ mỗi lần bạn tự xử một ca thuộc phạm vi EM, bạn vừa lấy đi một lần học của manager đó, vừa tiêu mất thời gian đáng lẽ dùng cho việc chỉ bạn làm được — nhìn ra mẫu và đổi cơ chế. Vì phần lỗ không hiện ra ngay, hành vi này tự củng cố.

**Sai lầm điển hình.** Trở thành một "EM của các EM": tổ chức 1-1 với các manager mà nội dung chủ yếu là cùng giải các ca cụ thể trong team của họ. Hình dạng của nó gần giống việc coaching nên rất khó tự phát hiện. Phép phân biệt dùng được: trong buổi 1-1, ai là người ra kết luận? Nếu là bạn, đó là can thiệp. Nếu là họ và bạn chỉ hỏi để chất lượng lập luận hiện ra, đó là phát triển manager.

Sai lầm thứ hai là mặt đối xứng của nó: **rút hẳn lên tầng số liệu và mất hoàn toàn tiếp xúc với hiện trường** (mục 8, sai lầm 2). Hai sai lầm này trông đối nghịch nhưng có cùng gốc — chưa xây được cơ chế để biết chuyện gì đang xảy ra mà không phải tự đi xem.

Sai lầm thứ ba: **giữ vai EM của một team cũ trong khi đã là Director**, thường xảy ra khi một EM được lên Director cho chính bộ phận của mình và một team chưa có người thay. Trạng thái này được biện minh bằng "tạm thời", và nó thường kéo dài ba đến bốn quý, trong đó team đó nhận được một manager làm việc ở 30% công suất và các team khác nhận được một Director làm việc ở 60%.

**Cách chuẩn bị trước sáu tháng.**

| Việc làm trong 6 tháng | Nó luyện đúng cơ chế nào |
|---|---|
| Nhận một vấn đề cắt qua ba team (quy trình tuyển, chuẩn onboarding, quy trình incident) và giải nó bằng một cơ chế có người sở hữu và có ngày review | Luyện việc tạo kết quả qua cơ chế thay vì qua can thiệp |
| Mentor một Tech Lead đang chuẩn bị lên EM, trọn hai quý, và ghi lại từng lần bạn muốn nhảy vào xử thay | Luyện phần khó nhất của bậc Director: để người khác học bằng cách sống với hậu quả |
| Mỗi tháng viết một trang cho tầng executive theo cấu trúc kết luận trước, phương án kèm giá, mức không chắc chắn — và xin phản hồi về nó | Luyện executive communication ở nơi rủi ro còn thấp |
| Chọn một chỗ trong tổ chức bạn không có quyền gì, và cố ý không can thiệp trong một quý dù thấy sai; ghi dự đoán trước, đối chiếu sau | Luyện việc chịu được không biết chi tiết, và đo xem phán đoán của mình có thật sự tốt hơn không |
| Xin dự các buổi bàn về ngân sách và headcount với vai người nghe | Luyện đọc bài toán phân bổ — thứ không nhìn thấy được từ bậc EM |
| Viết ra 5 dự đoán mỗi quý về team và người, đối chiếu cuối quý | Xây cơ chế đối chiếu trước khi vào bậc mà feedback loop dài đủ để tự lừa mình |

Tiêu chí biết là đã sẵn sàng: bạn chỉ ra được một vấn đề mà bạn đã xử lý bằng cách đổi điều kiện thay vì đổi hành vi của một người, và nói được cả phần cơ chế đó đã gây ra hệ quả ngoài ý muốn ở đâu.

---

## Chọn nhánh: IC track vs Manager track

Cuộc trò chuyện này thường xảy ra sai cách. Một Senior được gọi vào phòng và nghe: "team cần một người dẫn, cậu nhận nhé". Người đó nói có, vì nói không nghe như từ chối trách nhiệm, và vì trong đầu họ câu đó có nghĩa là "cậu đã được ghi nhận". Mười tám tháng sau, cả hai bên đều đang trả giá cho một quyết định mà chưa ai từng phát biểu thành một lựa chọn.

Mục này không nói nhánh nào tốt hơn. Nó cung cấp hai thứ: một bảng so sánh theo các trục thật (không phải theo lương và title), và một bộ câu hỏi chẩn đoán mà bạn tự trả lời — với điều kiện trả lời thật, vì không ai đọc câu trả lời của bạn nên không có lý do gì để trả lời cho hay.

### Bảng so sánh theo bảy trục

| Trục | IC track (Senior → Staff → Principal) | Manager track (EM → Director) |
|---|---|---|
| **Đầu ra** | Phán đoán kỹ thuật được kết tinh thành hệ thống, chuẩn, và bài toán được định nghĩa lại. Đầu ra có tính vật chất: một kiến trúc, một platform, một RFC làm ba team đổi hướng. | Năng lực và kết quả của người khác, và cấu trúc sinh ra năng lực đó. Đầu ra không có hình dạng: không tuần nào bạn chỉ ra được vật gì mình đã tạo. |
| **Độ dài feedback loop** | Trung bình. Một quyết định kiến trúc cho tín hiệu sau 1–4 quý; nhưng bạn vẫn giữ được vòng phản hồi ngắn hàng ngày qua công việc kỹ thuật thật, nên hệ thần kinh có chỗ tựa. | Dài và nhiễu. Một quyết định về người hoặc về cấu trúc cho tín hiệu sau 2–4 quý, và tín hiệu luôn có ít nhất hai cách giải thích. Không có vòng phản hồi ngắn nào để tựa vào. |
| **Loại năng lượng bị tiêu hao** | Năng lượng nhận thức: giữ một mô hình phức tạp trong đầu trong nhiều giờ liền. Hồi phục bằng nghỉ và ngủ. Bị phá bởi gián đoạn — hai cuộc họp đặt lệch giờ có thể xoá một ngày. | Năng lượng cảm xúc: hấp thụ lo lắng của người khác, mang tin xấu, ngồi trong xung đột mà không rút. Không hồi phục bằng ngủ. Bị phá bởi việc phải giả vờ bình tĩnh khi không bình tĩnh. |
| **Khả năng quay lại nhánh kia** | Cao trong 1–2 năm đầu; sau đó vẫn được nhưng bạn thiếu số ca về người đã xử lý trọn vòng, và điều đó chỉ tích luỹ được bằng cách làm thật. | Giảm theo thời gian và theo tốc độ đổi công nghệ. Sau 18 tháng bạn mất tính hiện thời; sau 4–5 năm việc quay lại thường có nghĩa là chấp nhận bắt đầu lại ở độ sâu kỹ thuật, dù phán đoán hệ thống vẫn còn giá trị. |
| **Mức độ phụ thuộc vào tổ chức cụ thể** | Thấp về mặt kỹ năng (kiến thức hệ thống mang đi được), **cao về mặt bậc tồn tại**: nếu công ty không có bậc Staff/Principal thật, nhánh này đơn giản không có đường đi ở đó. | Cao về mặt kỹ năng có tác dụng (bạn phải học lại chính trị nội bộ, ai quyết gì, ràng buộc ngân sách ở mỗi tổ chức mới — thường mất 2–3 quý), **thấp về mặt bậc tồn tại**: mọi công ty đủ người đều cần manager. |
| **Thứ bạn sẽ mất** | Ảnh hưởng trực tiếp lên đời sống nghề nghiệp của người khác; quyền quyết định về nguồn lực; và ở nhiều công ty Việt Nam là trần lương thấp hơn. Thêm một mất mát ít được nói: bạn phải thuyết phục lại từ đầu ở mỗi bài toán, vì bạn không có thẩm quyền nào để rút ngắn. | Tính hiện thời kỹ thuật; cảm giác hoàn thành hàng ngày; quyền có tâm trạng; và quyền được đánh giá bằng thứ mình kiểm soát trực tiếp. |
| **Cách thất bại điển hình** | Trở thành người viết tài liệu không ai đọc và không ai buộc phải tuân theo — phạm vi thật bằng không. Hoặc trở thành người duy nhất hiểu một hệ thống, tưởng đó là quyền lực trong khi đó là một cái bẫy: bạn không thể rời khỏi nó và cũng không thể phát triển lên. | Trở thành người truyền tin: chuyển yêu cầu từ trên xuống, than phiền từ dưới lên, không quyết gì, mất giá ở cả hai chiều đồng thời. Hoặc trở thành EM vẫn giữ việc kỹ thuật trên critical path và chặn cả team. |

Ba quan sát từ bảng này quan trọng hơn bảng:

- **Trục "loại năng lượng bị tiêu hao" là trục dự báo tốt nhất về sự bền vững**, và là trục ít được cân nhắc nhất. Người ta chọn nhánh dựa trên "tôi có làm được không" (phần lớn người giỏi làm được cả hai) thay vì "loại kiệt sức nào tôi hồi phục được". Kiệt sức nhận thức có cách chữa đã biết: chặn lịch, giảm gián đoạn, ngủ. Kiệt sức cảm xúc thì không, và nó tích luỹ âm thầm qua nhiều quý.
- **Hai nhánh không đối xứng về khả năng quay lại**, và đó là một thông tin thực hành: chi phí thử nhánh Manager tăng theo thời gian nhanh hơn chi phí thử nhánh IC. Hệ quả: nếu bạn định thử, thử sớm thì rẻ hơn, và thoả thuận trước về đường quay lại thì rẻ hơn nữa.
- **Trục "mức độ phụ thuộc vào tổ chức cụ thể" giải thích phần lớn các quyết định sai ở Việt Nam.** Nhiều người tưởng mình đang chọn giữa hai nhánh, trong khi thực tế họ đang chọn giữa một nhánh có đường đi và một nhánh không tồn tại ở công ty của họ. Đó là một quyết định hoàn toàn khác, và nó nên được phát biểu đúng: "ở đây tôi chỉ có một đường; tôi chọn đi đường đó, hoặc tôi đổi chỗ".

### Nói thẳng: làm quản lý không phải cấp trên của làm IC

Hai khẳng định cần nói rõ, vì phần lớn tổ chức hành xử như thể chúng sai.

**Thứ nhất: không phải mọi kỹ sư giỏi đều nên làm quản lý.** Không phải "chưa sẵn sàng", không phải "cần đào tạo thêm" — mà là không nên, kể cả sau khi được đào tạo tốt. Cơ chế: hai nhánh đòi hỏi hai loại chịu đựng khác nhau, và loại chịu đựng là đặc tính khá bền của một người. Một người tạo ra giá trị lớn nhất khi giữ một mô hình phức tạp trong đầu bốn giờ liền không tự nhiên trở nên giỏi hơn ở việc bị cắt lịch thành hai mươi mảnh; và ngược lại. Hệ quả kinh tế rất rõ: chuyển một Staff Engineer xuất sắc thành một EM trung bình là một giao dịch lỗ hai lần — tổ chức mất một đầu ra khó thay thế và nhận về một đầu ra ai cũng làm được ở mức đó.

**Thứ hai: nhánh Manager không nằm phía trên nhánh IC.** Chúng là hai trục, không phải hai đoạn của một trục. Một Principal Engineer và một Director là hai vai có phạm vi tương đương với hai loại đòn bẩy khác nhau; không ai trong hai người là cấp trên của người kia về giá trị tạo ra.

Vì sao ngành hay nhầm điều này? Năm cơ chế, và không cơ chế nào trong đó là một sai sót ngẫu nhiên:

1. **Cấu trúc lương và thẩm quyền được gắn vào sơ đồ tổ chức, vì đó là thứ dễ đo.** Ngân sách và số người báo cáo là con số; phạm vi ảnh hưởng của một IC thì không. Khi phải phân bổ lương một cách có thể bảo vệ được trước HR và trước tài chính, tổ chức chọn biến đại diện dễ đo nhất — và biến đó là vị trí trên sơ đồ báo cáo.
2. **Công việc quản lý hiển thị lên trên, công việc IC thì không.** Manager ngồi trong các phòng họp có executive; đầu ra của họ được nghe trực tiếp bởi người quyết định lương. Một Staff Engineer làm một việc mà hệ quả lớn hơn nhưng chỉ hiện ra dưới dạng "hệ thống không có sự cố" — thứ không ai nhìn thấy.
3. **Chuỗi báo cáo bị đọc thành thứ tự giá trị.** Ngôn ngữ củng cố điều này: "cấp trên", "được lên làm quản lý", "dưới quyền". Quan hệ báo cáo là một cơ chế điều phối thông tin và trách nhiệm; nó không phải một xếp hạng về giá trị đóng góp. Nhưng khi từ ngữ đã mã hoá sẵn thứ bậc, người dùng từ ngữ đó khó nghĩ khác được.
4. **Bậc IC thật đắt hơn bậc Manager để duy trì.** Một tổ chức muốn có bậc Staff/Principal thật phải làm được ba việc khó: định nghĩa tiêu chí không dựa trên số người báo cáo, có người đủ tầm để đánh giá tiêu chí đó, và bảo vệ được mức lương đó trước câu hỏi "anh ta quản lý ai". Nhiều tổ chức công bố bậc IC trên giấy nhưng không làm được ba việc này, nên bậc đó tồn tại như một cái tên.
5. **Vòng lặp tự củng cố.** Vì bậc IC yếu, IC giỏi rẽ sang Manager. Vì IC giỏi rẽ đi, không còn ai đủ tầm để làm chuẩn cho bậc IC. Vì không có chuẩn, bậc IC tiếp tục yếu. Ở bối cảnh Việt Nam, vòng lặp này cộng thêm kỳ vọng gia đình và xã hội về "làm quản lý" như dấu hiệu thành công, nên nó quay nhanh hơn.

Điều này có nghĩa gì về mặt thực hành với người đang đọc: **áp lực bạn cảm thấy về việc phải rẽ sang nhánh Manager phần lớn không phải tín hiệu về sự phù hợp của bạn — nó là tín hiệu về cách tổ chức của bạn đo lường và trả tiền.** Hai thứ đó cần được tách ra trước khi quyết định, vì chúng dẫn đến hai hành động khác nhau: một cái là đổi nhánh, cái kia là đàm phán bậc hoặc đổi tổ chức.

### Tám câu hỏi chẩn đoán

Cách dùng: trả lời bằng một sự việc cụ thể đã xảy ra, không bằng một ý định. Câu trả lời "tôi nghĩ mình sẽ..." không có giá trị chẩn đoán nào. Đây không phải bài trắc nghiệm có điểm; mỗi câu chỉ ra một trục, và bạn đọc mẫu chung của tám câu trả lời.

1. **Bạn thấy hài lòng hơn khi tự giải xong một bài toán khó, hay khi thấy người bạn kèm giải xong nó?** Trả lời bằng hai lần cụ thể đã xảy ra trong sáu tháng qua và so sánh cảm giác. Nếu câu trả lời là "cả hai như nhau", khả năng cao bạn chưa từng thật sự kèm ai đến điểm họ tự giải được — vì hai cảm giác đó rất khác nhau khi đã trải qua cả hai.

2. **Lần gần nhất bạn dành hai giờ giúp người khác gỡ một bug mà không ai biết, bạn thấy đó là thời gian mất đi hay thời gian đáng giá?** Điều kiện "không ai biết" là phần quan trọng nhất của câu hỏi: nó loại bỏ động lực được ghi nhận và chỉ để lại phản ứng thật. Nếu cảm giác chủ đạo là bực vì mất hai giờ của mình, đó là một dữ liệu mạnh về nhánh IC — và là dữ liệu tốt, không phải khuyết điểm.

3. **Bạn có nói được tên một người đã tiến bộ rõ rệt vì bạn trong hai năm qua, và nói được cụ thể bạn đã làm gì?** "Tôi hay giúp mọi người" không tính. Cần: người đó trước làm được gì, sau làm được gì, và ba việc bạn đã làm. Nếu bạn không dẫn được một ca như vậy dù đã ở bậc Senior nhiều năm, thì không phải bạn không làm được — mà là bạn chưa bao giờ tự nguyện chọn làm việc đó khi không ai yêu cầu, và đó chính là dữ liệu.

4. **Bạn đã bao giờ nói một câu làm người đối diện im lặng năm giây rồi mặt xấu đi — và một tháng sau nói lại lần thứ hai với cùng người đó về cùng vấn đề?** Đây là câu hỏi có tỷ lệ trả lời "chưa" cao nhất, và nó lọc mạnh nhất. Kỹ năng phân biệt EM thật với người giữ chức EM không phải khả năng nói câu khó một lần, mà là khả năng nói nó lần thứ hai và thứ ba khi lần đầu chưa tạo ra thay đổi. Nếu bạn chưa từng làm điều này, bạn không có dữ liệu về việc mình chịu được nó hay không.

5. **Có một tuần nào trong năm qua bạn không tạo ra gì hữu hình nhưng cuối tuần vẫn thấy đó là tuần tốt — và bạn kể được ba việc đã xảy ra vì bạn?** Nếu phải nghĩ lâu mới ra, đây là bài kiểm tra trực tiếp cho cái giá lớn nhất của nhánh Manager: sống nhiều năm mà không có cảm giác hoàn thành hàng ngày.

6. **Khi bạn thấy một người trong team làm sai cách, phản xạ đầu tiên của bạn là gì: tự sửa, chỉ cho họ cách đúng, hay tìm hiểu vì sao họ chọn cách đó?** Ba phản xạ này tương ứng khá chính xác với ba mode of impact (tự làm / qua người khác / qua việc hiểu hệ thống sinh ra hành vi). Phản xạ không nói bạn nên chọn nhánh nào, nhưng nó nói bạn phải học lại cái gì.

7. **Nghĩ đến hai năm nữa: bạn sợ điều nào hơn — không còn hiểu nổi codebase mà mình từng viết, hay không còn ai trong tổ chức muốn hỏi ý kiến bạn?** Hai nỗi sợ này tương ứng với hai mất mát thật của hai nhánh. Câu trả lời của bạn cho biết bạn định nghĩa bản thân bằng cái gì, và cái đó khó đổi hơn kỹ năng.

8. **Nếu lương và title của hai nhánh bằng nhau, bạn chọn nhánh nào?** Câu này đặt cuối vì nó chỉ có nghĩa sau bảy câu trên. Nếu câu trả lời của bạn đổi khi bỏ yếu tố lương và title ra, thì thứ bạn muốn không phải nhánh Manager mà là thứ chỉ nhánh Manager ở công ty này cấp cho bạn — và việc cần làm là đàm phán về bậc IC hoặc đổi tổ chức, không phải đổi nhánh.

Một bẫy cần nêu riêng, vì nó không hiện ra trong tám câu trên: **muốn làm quản lý vì muốn có quyền quyết định** là một động lực dẫn đến thất vọng gần như chắc chắn. Cơ chế: thẩm quyền tăng thì số ràng buộc cũng tăng nhanh hơn. Một EM có quyền quyết ít việc hơn một Staff Engineer tưởng, và phần lớn thời gian của EM dùng để thực thi những quyết định do người khác ra. Nếu điều bạn thật sự muốn là quyền quyết định về kỹ thuật, nhánh IC cho bạn nhiều hơn.

### Script hội thoại: một Senior nói muốn thử vai lead

Bối cảnh: Quân, Senior BE 5 năm, nói với Hà (EM của cậu) trong buổi 1-1. Team đang có Tuấn giữ vai Tech Lead.

**Quân:**

> "Chị ơi, em muốn nói một việc. Em nghĩ em muốn thử vai lead. Em thấy mình làm kỹ thuật thì ổn rồi, mà cứ ở đây thì em không biết mình đi đâu tiếp. Có cơ hội nào không chị?"

Hai phản ứng dễ nhất đều sai, và sai theo hai hướng đối nghịch. **Hứa** ("được, để chị xem quý sau có team nào tách ra không") tạo một kỳ vọng mà Hà không kiểm soát được, và khi nó không xảy ra, Hà mất niềm tin của Quân đúng vào lúc Quân đang cân nhắc đi. **Từ chối** ("hiện chưa có chỗ, Tuấn đang làm Tech Lead rồi") trả lời một câu hỏi khác với câu Quân hỏi — Quân không xin chức của Tuấn, cậu đang nói cậu không thấy đường đi tiếp, và câu trả lời đó xác nhận rằng thật sự không có đường.

**Cách nên trả lời — tạo cơ hội có kiểm soát:**

> Hà: "Cảm ơn Quân nói ra. Chị muốn tách câu hỏi của Quân thành hai câu, vì chị nghĩ trong đó có hai việc khác nhau.
>
> Việc thứ nhất: Quân nói không biết đi đâu tiếp. Cái đó chị nhận là việc của chị chưa làm — chị chưa bao giờ nói rõ với Quân bậc tiếp theo ở nhánh kỹ thuật ở đây trông như thế nào và Quân đang thiếu gì. Chị sẽ trả lời việc đó bằng văn bản trong hai tuần, và nó không phụ thuộc vào chuyện lead hay không lead.
>
> Việc thứ hai: Quân muốn thử vai lead. Chị nói thẳng phần chị không làm được trước: chị không hứa được một vai Tech Lead, vì nó phụ thuộc vào việc team có tách hay không, và chị không quyết được điều đó một mình. Chị sẽ không hứa thứ chị không kiểm soát.
>
> Nhưng phần chị làm được ngay thì có, và nó thực chất là phần quan trọng hơn: mình tạo một lần thử thật để cả Quân và chị có dữ liệu. Cụ thể — sprint sau có epic tích hợp đối tác thanh toán, khoảng năm tuần, có Vy và Khoa tham gia. Chị đề nghị Quân dẫn epic đó, với ba điều kiện rõ:
>
> Một: Quân dẫn nghĩa là Quân chịu trách nhiệm về việc epic đó xong và xong đúng cách, không phải là người viết phần khó nhất. Nếu cuối epic chị thấy Quân đã tự làm 70% code trong đó, mình sẽ coi đó là một kết quả — và là kết quả cho biết điều gì đó.
>
> Hai: Quân đàm phán scope trực tiếp với Trang, chị đi cùng buổi đầu và ngồi im, từ buổi thứ hai Quân đi một mình. Đây là phần khó nhất và cũng là phần cho biết nhiều nhất.
>
> Ba: mình gặp nhau mười lăm phút mỗi tuần, chỉ nói về ba câu: tuần này Quân đã làm gì mà không phải viết code, chỗ nào Quân muốn nhảy vào làm hộ mà đã kìm lại, và chỗ nào Quân đã kìm lại nhưng đáng lẽ nên nhảy vào.
>
> Và chị nói rõ về việc điều này *không* là gì, để nó không bị hiểu sai ở hai phía. Nó không phải một lời hứa về vai Tech Lead — nếu Quân làm rất tốt mà quý sau team không tách thì vẫn không có vai, và chị sẽ nói thẳng lúc đó. Nó cũng không phải một bài kiểm tra mà trượt thì mất điểm: nếu sau năm tuần kết luận là Quân không thích công việc này, chị coi đó là năm tuần đáng giá nhất trong quý, vì mình vừa tránh được một quyết định sai kéo dài hai năm. Chị cũng sẽ nói với Tuấn và với team rằng Quân dẫn epic này, để Quân có thẩm quyền thật trong năm tuần đó chứ không phải đi nhờ từng người.
>
> Một câu hỏi chị muốn Quân trả lời trước khi nhận: trong năm tuần đó, nếu Vy làm một phần theo cách Quân biết là chậm hơn cách của Quân khoảng ba ngày, nhưng cô ấy sẽ học được, Quân sẽ làm gì? Quân không cần trả lời chị ngay bây giờ. Nhưng đó là câu mà cả nhánh này rút gọn về."

Bốn thuộc tính làm cho phản ứng này khác với hứa và với từ chối: nó **tách nhu cầu thật (không thấy đường đi) khỏi giải pháp mà người đó tự đề xuất (làm lead)** và trả lời cả hai; nó **nói rõ ranh giới của thẩm quyền của Hà** thay vì hứa vượt ranh giới đó; nó **tạo một lần thử có phạm vi, có thời hạn, có thẩm quyền thật và có tiêu chí quan sát được** thay vì một lời động viên; và nó **định nghĩa trước cả hai kết cục là hợp lệ**, nên Quân không có động lực diễn cho ra kết quả "thành công" — điều sẽ làm hỏng toàn bộ giá trị chẩn đoán của năm tuần đó.

---

## Bối cảnh Việt Nam

Tám bậc ở trên được mô tả như thể chúng tồn tại. Ở phần lớn công ty Việt Nam, một nửa trong số đó không tồn tại như một vai có phạm vi thật, và một số bậc tồn tại dưới dạng title mà không có nội dung. Mục này không phán xét điều đó — nó là hệ quả hợp lý của tuổi thị trường, quy mô tổ chức, và cấu trúc doanh thu. Nhưng nếu bạn lập kế hoạch nghề nghiệp dựa trên một cái thang không có ở nơi bạn đang đứng, bạn sẽ tối ưu sai trong nhiều năm. Năm đặc thù dưới đây được xử lý theo cùng một cách: cơ chế sinh ra nó, hệ quả quan sát được, và bạn làm gì với nó.

### Thiếu tầng Staff/Principal thật — và hệ quả kép

**Cơ chế.** Một bậc IC thật đòi hỏi ba thứ mà tổ chức nhỏ và trẻ khó có đồng thời: (1) đủ độ phức tạp hệ thống để có những bài toán cắt qua nhiều team — dưới khoảng 40–50 engineer, loại bài toán đó ít khi xuất hiện; (2) có người đủ tầm để đánh giá công việc của một Staff Engineer, mà chính người đó lại là người bạn đang thiếu; (3) khả năng bảo vệ mức lương cao trước câu hỏi "anh ta quản lý bao nhiêu người" — câu hỏi này đến từ tài chính và HR, không đến từ engineering, và nó rất khó trả lời bằng dữ liệu.

Vì ba điều kiện đó khó thoả, con đường tăng lương đáng kể duy nhất ở nhiều công ty đi qua nhánh quản lý. Đây không phải một quyết định ai đó cố tình đưa ra; nó là điểm cân bằng mặc định.

**Hệ quả kép.** Cả hai vế đều tốn kém, và chúng xảy ra đồng thời với cùng một con người:

- **Vế thứ nhất — mất một IC khó thay thế.** Người đã tích luỹ 8 năm về một domain, biết vì sao mỗi quyết định trong hệ thống được ra như vậy, được chuyển sang một vai mà 70% thời gian là họp và nói chuyện. Kiến thức đó không được truyền lại, nó chỉ mất tính hiện thời dần trong 12–18 tháng. Đây là mất mát ít nhìn thấy nhất vì nó không tạo ra sự cố nào ngay; nó chỉ làm mọi quyết định kỹ thuật về sau đắt hơn một chút.
- **Vế thứ hai — nhận về một manager trung bình, và 6–8 người trả giá cho điều đó.** Vì người này rẽ nhánh vì lương chứ không vì phù hợp, xác suất họ tránh phần khó nhất của vai (cuộc trò chuyện khó, quyết định về người) cao hơn. Team dưới họ không ai được nói thật về hiệu suất trong hai năm.

Vế thứ ba, ít được nêu nhưng đắt nhất về dài hạn: **tổ chức đó vĩnh viễn không xây được chuẩn cho bậc IC**, vì mỗi người đủ tầm để làm chuẩn đều đã rẽ đi. Vòng lặp tự khoá.

**Bạn làm gì với nó.** Nếu bạn là người bị đẩy: trước khi nhận vai quản lý, hỏi một câu cụ thể và yêu cầu câu trả lời cụ thể — "ở công ty này, bậc IC cao nhất là gì, đã có ai ở bậc đó chưa, và dải lương của nó chồng lấn với dải lương EM ở mức nào?" Nếu câu trả lời là "chúng ta chưa có nhưng đang xây", đó là một thông tin hợp lệ và bạn nên hỏi thêm ai đang sở hữu việc xây đó và mốc nào. Nếu không có ai sở hữu, nó chưa tồn tại. Nếu bạn là Director hoặc CTO: cách rẻ nhất để mở bậc IC không phải viết một career ladder mười trang, mà là chọn một người, đặt họ vào một bài toán cắt qua ba team với thẩm quyền thật, trả họ ở mức của một EM, và để cả tổ chức nhìn thấy điều đó xảy ra một lần. Một tiền lệ có thật mạnh hơn một tài liệu.

### Title inflation — cách đọc title bằng cách hỏi về phạm vi

Cơ chế đã nêu ở mục [Ba trục thật của sự thăng tiến](#ba-trục-thật-của-sự-thăng-tiến): trong thị trường nóng, nâng title là công cụ giữ người có chi phí ngân sách gần bằng không và có tác dụng tức thì. Ở đây chỉ xử lý phần thực hành: khi bạn phỏng vấn một ứng viên có title "Tech Lead 3 năm" hoặc "Solution Architect", làm sao biết bậc thật.

Nguyên tắc: **đừng hỏi về title, hỏi về phạm vi, độ mơ hồ, và cách họ tạo ra kết quả** — đúng ba trục của chương này. Title là thứ công ty cũ cấp; ba trục kia là thứ họ đã sống qua.

| Trục cần đo | Câu hỏi dùng được | Câu trả lời nông trông như thế nào | Câu trả lời có bậc thật trông như thế nào |
|---|---|---|---|
| **Phạm vi** | "Quyết định lớn nhất anh ra ở vai đó là gì, và bao nhiêu người phải sống với hậu quả của nó, trong bao lâu?" | Mô tả một quyết định trong module của chính mình; hoặc mô tả một quyết định mà thực chất do người khác chốt | Nêu được quyết định, ai bị ảnh hưởng, và hậu quả đã hiện ra thế nào sau 6–12 tháng |
| **Độ mơ hồ** | "Có lần nào anh phải tự định nghĩa bài toán chứ không nhận bài toán đã định nghĩa? Lúc đó anh dùng gì để biết mình định nghĩa đúng?" | "Em tự thiết kế giải pháp cho yêu cầu của khách" — đây là bài toán đã được định nghĩa, chỉ mở về cách làm | Nêu được cách họ tự tạo tiêu chí đúng/sai, và cách họ bảo vệ tiêu chí đó trước người có tiêu chí khác |
| **Cách tạo kết quả** | "Trong sáu tháng cuối ở vai đó, phần trăm thời gian anh viết code là bao nhiêu? Phần còn lại vào việc gì?" | 80–90% code, phần còn lại là họp — tức vai Tech Lead trên giấy | 30–50% code, phần còn lại có tên cụ thể: đàm phán scope, unblock, viết RFC, kèm người |
| **Thẩm quyền thật** | "Ai có thể phủ quyết quyết định kỹ thuật của anh, và có lần nào điều đó xảy ra?" | "Không ai" (thường nghĩa là không ai buộc phải tuân theo, tức phạm vi bằng không) hoặc "khách hàng quyết hết" | Nêu được ranh giới rõ: cái gì tôi quyết, cái gì phải qua ai, và một ca cụ thể bị phản biện thành công |
| **Đã sống qua hậu quả** | "Một quyết định kiến trúc anh ra mà sau đó hoá ra sai — anh phát hiện lúc nào, bằng dấu hiệu gì, và đã sửa thế nào?" | Không nêu được ca nào, hoặc nêu ca mà hậu quả do người khác gánh sau khi họ đã đi | Nêu được thời điểm phát hiện, chi phí sửa, và điều họ làm khác đi từ đó |
| **Việc về người** | "Anh đã bao giờ nói với ai rằng họ đang không đạt kỳ vọng? Kể lại buổi đó — anh mở đầu bằng câu gì?" | "Em có nhắc nhở nhẹ" | Nhớ được câu mở đầu, phản ứng của người kia, và điều gì đã đổi sau đó |

Hai lưu ý để không dùng bảng này sai. Thứ nhất, **title thấp cũng nhiễu theo hướng ngược lại**: một người mang title "Senior" ở một công ty có chuẩn khắt khe có thể ở bậc thật cao hơn một "Tech Lead" ở nơi khác, và nếu bạn lọc CV theo title bạn sẽ bỏ qua chính những người này. Thứ hai, **đừng phạt ứng viên vì title được nâng cho họ** — họ không phải người đã in nó. Việc của bạn là đo bậc thật để đặt đúng chỗ và đặt đúng kỳ vọng, không phải để bắt lỗi. Một người có title Tech Lead ba năm nhưng bậc thật là Mid-level cao thường là một ứng viên tốt nếu được đặt đúng và được nói rõ ngay từ đầu; điều làm họ thất bại là được nhận vào ở kỳ vọng của title.

### Job hopping và việc tích luỹ năng lực phạm vi rộng

**Phần mất, nói bằng cơ chế.** Nếu bạn ở mỗi công ty 18 tháng, có bốn loại kinh nghiệm bạn không thể có, và không có cách nào bù bằng số lượng công ty:

1. **Bạn không bao giờ thấy hậu quả của quyết định kiến trúc của mình.** Một quyết định về ranh giới service, về việc chọn đồng bộ hay bất đồng bộ, về mức độ chuẩn hoá dữ liệu — hậu quả thật của nó xuất hiện ở năm thứ hai đến năm thứ tư, khi lưu lượng đổi, khi có ba team khác build lên nó, khi phải migrate. Ở 18 tháng bạn ra quyết định rồi rời đi ngay trước lúc hoá đơn đến. Điều này tạo ra một loại kỹ sư rất tự tin về kiến trúc mà chưa từng trả giá cho bất kỳ lựa chọn nào của mình — và cơ chế học duy nhất ở trục Ambiguity là trả giá.
2. **Bạn không thấy một hệ thống đi qua một lần đổi bậc quy mô.** Kinh nghiệm giá trị nhất về scaling không phải việc thiết kế cho quy mô lớn, mà việc sống qua đoạn hệ thống cũ vỡ dần và phải vừa vá vừa chuyển.
3. **Bạn không thấy một người mình tuyển hay mình kèm phát triển qua ba năm.** Vòng phản hồi cho phán đoán về người dài 2–4 quý cho tín hiệu đầu và 2–3 năm cho tín hiệu thật. Đây là lý do trục "phán đoán về người" gần như không tích luỹ được với nhiệm kỳ ngắn.
4. **Bạn tiêu phần lớn nhiệm kỳ vào giai đoạn chưa được giao việc rộng.** Ở một tổ chức mới, để được giao một bài toán cắt qua nhiều team, bạn cần 9–15 tháng (số minh hoạ) xây uy tín và hiểu bối cảnh. Nếu nhiệm kỳ là 18 tháng, bạn dành phần lớn thời gian ở giai đoạn xây tiền đề rồi rời đi trước khi dùng nó. Về mặt tích luỹ trục Scope, năm lần 18 tháng không bằng một lần 7 năm — và điều đó không liên quan gì đến lòng trung thành, nó là số học.

**Phần hợp lý, cũng nói bằng cơ chế.** Job hopping ở thị trường Việt Nam không phải một khuyết điểm về tính cách, và bất kỳ ai khuyên "hãy ở lại lâu" mà không nói phần này đang đưa ra lời khuyên tốn tiền cho người khác:

- **Dải lương nội bộ điều chỉnh chậm hơn thị trường.** Trong giai đoạn thị trường nóng, mức tăng nội bộ hàng năm ở nhiều công ty nằm trong khoảng thấp hơn hẳn mức chênh mà một lần đổi việc mang lại (số minh hoạ, nhưng khoảng cách này được quan sát rộng rãi). Ở lại là một lựa chọn có chi phí thật, và chi phí đó rơi vào người ở lại chứ không phải vào công ty.
- **Nếu bậc tiếp theo không tồn tại ở nơi bạn đang làm, ở lại không sinh ra nó.** Đây là trường hợp job hopping hoàn toàn hợp lý: bạn đã tối đa hoá phạm vi có thể có ở tổ chức hiện tại, và bậc trên chỉ là một tên gọi hoặc không có. Kiên nhẫn ở đây không tích luỹ gì ngoài thời gian.
- **Giai đoạn đầu nghề cần độ rộng.** Với 0–4 năm kinh nghiệm, việc thấy ba loại tổ chức khác nhau (một startup, một ODC, một công ty có hệ thống lớn) cho bạn dữ liệu để biết mình muốn gì — dữ liệu mà ở lại một nơi tám năm không cho.
- **Rủi ro tổ chức là thật.** Nhiều công ty Việt không tồn tại đủ lâu để bạn hoàn thành một chu kỳ bảy năm ở đó, và việc thoát khỏi một tổ chức đang xấu đi là một quyết định đúng chứ không phải sự thiếu bền bỉ.
- **Ở lại một nơi mà quyết định do vendor hoặc khách hàng chốt không tích luỹ trục Ambiguity.** Tám năm trong điều kiện đó có thể kém giá trị hơn ba năm ở nơi bạn được quyết và phải chịu hậu quả.

**Cách dung hoà, cụ thể.** Không phải "hãy ở lại lâu hơn", mà là ba nguyên tắc dùng được:

1. **Mỗi lần đổi phải tăng ít nhất một trong ba trục**, và bạn phải nói được trục nào. Đổi để tăng lương mà phạm vi, độ mơ hồ, và cách tạo kết quả đều giữ nguyên là một lần đổi tốt về tài chính và bằng không về năng lực — hoàn toàn hợp lý nếu bạn biết mình đang chọn điều đó.
2. **Ít nhất một lần trong nghề, ở đủ lâu để nhận hoá đơn của chính mình** — nghĩa là 3–4 năm liền ở một chỗ nơi bạn đã ra quyết định kiến trúc thật. Đây là lần duy nhất trục Ambiguity nhích được một bậc thật, và nó không thay thế được bằng bất cứ gì.
3. **Trước khi đổi, kiểm tra xem vấn đề có đi theo bạn không.** Nếu lý do đổi là "ở đây không ai nghe tôi", có hai khả năng: tổ chức không có chỗ cho ảnh hưởng, hoặc bạn chưa học cách tạo ảnh hưởng khi không có thẩm quyền. Khả năng thứ hai đi theo bạn sang mọi công ty.

### Kỳ vọng gia đình về "làm quản lý"

Đây là một biến thật trong quyết định nghề nghiệp ở Việt Nam, và bỏ qua nó làm cho toàn bộ phần phân tích ở trên trở nên không dùng được với nhiều người.

**Cơ chế.** Nghề software engineering còn quá mới để có một mô hình thành công được xã hội hiểu. Với thế hệ trước, thang bậc dễ hiểu duy nhất trong mọi ngành là quản lý: có bao nhiêu người dưới quyền. Một câu như "con làm Staff Engineer, con không quản lý ai nhưng quyết định kiến trúc của cả hệ thống" không dịch được sang mô hình đó, trong khi "con lên trưởng phòng" thì dịch được ngay. Đây không phải sự thiếu hiểu biết — đó là việc dùng một biến đại diện đã đúng trong hầu hết các ngành khác suốt nhiều thập kỷ.

**Hệ quả quan sát được.** Áp lực này cộng với việc thiếu bậc IC tạo ra một lực đẩy cùng hướng, và người ở giữa hai lực đó thường không phân biệt được đâu là mong muốn của mình. Dấu hiệu nhận ra: khi trả lời câu "nếu lương và title bằng nhau bạn chọn nhánh nào", bạn thấy mình đang nghĩ đến việc giải thích với ai đó chứ không đang nghĩ về công việc.

**Cách xử lý.** Không phải thuyết phục gia đình đọc về career ladder. Ba việc dùng được: (1) tách rõ trong đầu ba thứ đang bị trộn — công việc bạn muốn làm, mức thu nhập bạn cần, và cái nhãn dễ giải thích; ba thứ này có thể giải quyết bằng ba cách khác nhau chứ không phải bằng một quyết định; (2) nếu cái cần là một nhãn dễ giải thích, nhiều tổ chức hoàn toàn có thể cấp một title bên ngoài dễ hiểu ("Kiến trúc trưởng", "Chuyên gia trưởng") cho một vai IC — đây là một yêu cầu hợp lệ và rẻ, và bạn nên nêu ra thay vì rẽ nhánh; (3) nếu cái cần là thu nhập, thì đó là bài toán đàm phán về dải lương bậc IC, không phải bài toán chọn nhánh — và trộn hai bài toán này là cách đắt nhất để giải cả hai.

### Bậc nào tồn tại thật ở ba loại tổ chức

Bảng dưới đây là mô tả khái quát, không phải quy luật; mỗi loại đều có ngoại lệ, và một công ty có thể ở giữa hai loại.

| | **Product company Việt** (đã qua 40–50 engineer) | **Outsourcing / ODC** | **Doanh nghiệp truyền thống (bộ phận IT)** |
|---|---|---|---|
| **Bậc tồn tại thật** | Junior → Senior đầy đủ; Tech Lead thật; EM thật; Staff bắt đầu có nghĩa khi vượt ~50 engineer | Junior → Senior đầy đủ; Tech Lead thật (nhưng phạm vi hẹp hơn); vai quản lý nghiêng về delivery và utilization | Các vai vận hành và điều phối vendor; vai quản lý theo cấp bậc doanh nghiệp |
| **Bậc thường chỉ có trên giấy** | Principal (ở dưới 100 engineer thường chưa có bài toán cho nó) | Staff/Principal — vì quyết định kiến trúc do khách hàng chốt nên không có chỗ cho phán đoán độc lập ở phạm vi rộng | Gần như toàn bộ nhánh IC; "Solution Architect" thường là vai vẽ sơ đồ trong đề xuất, không có thẩm quyền thi hành |
| **Ai ra quyết định kiến trúc** | Bên trong: Tech Lead / Staff / CTO | Khách hàng, hoặc kiến trúc sư phía khách | Vendor, hoặc nhà cung cấp giải pháp |
| **Trục nào tăng nhanh** | Ambiguity và Scope, nếu bạn ở lại đủ lâu để nhận hậu quả | Chất lượng thực thi, kỷ luật quy trình, giao tiếp với stakeholder nước ngoài (kể cả tiếng Anh) — những trục thật và mang đi được | Hiểu nghiệp vụ sâu, quản lý thay đổi trong tổ chức kháng cự, làm việc với ràng buộc compliance |
| **Trục nào bị kẹt** | Ít khi bị kẹt về bản chất, nhưng runway hữu hạn có thể làm mọi thứ dồn về ngắn hạn | Ambiguity — bài toán luôn đến ở dạng đã được định nghĩa; và Mode of Impact khó lên tới "qua cơ chế" vì cơ chế do khách quyết | Ambiguity kỹ thuật, và tốc độ: một chu kỳ quyết định có thể dài hàng quý |
| **Cái được, ít được ghi nhận** | Thấy toàn bộ chuỗi từ business goal xuống execution | Thấy nhiều domain và nhiều chuẩn kỹ thuật khác nhau trong thời gian ngắn; học được cách làm việc dưới ràng buộc và cách viết tài liệu tử tế — hai thứ mà nhiều product engineer thiếu | Hiểu vì sao tổ chức lớn hành xử như vậy — kiến thức cực kỳ hữu dụng nếu sau này bạn làm Director |
| **Chiến lược phát triển hợp lý ở đó** | Ở đủ lâu để nhận hoá đơn của một quyết định lớn của mình | Chủ động tìm phần được quyết: xin phụ trách phần khách không có ý kiến, xây internal tooling, làm phần pre-sales kỹ thuật nơi bạn phải tự định nghĩa bài toán. Nếu mục tiêu là nhánh Staff/Principal, tính trước một lần đổi loại tổ chức | Nhắm vào các vai chuyển đổi số nơi bạn được quyết thật, hoặc dùng nơi này để tích luỹ hiểu biết nghiệp vụ rồi mang sang product company |

Cách dùng bảng này: nó không nói loại tổ chức nào tốt hơn. Nó nói rằng **câu hỏi "làm gì để lên Staff" chỉ có câu trả lời sau khi biết bậc đó có tồn tại ở nơi bạn đang làm hay không.** Nếu không tồn tại, mọi kế hoạch phát triển cá nhân đều sẽ va vào một trần mà không ai nói ra, và bạn sẽ diễn giải cái trần đó thành thiếu sót của bản thân.

---

## Bản đồ tự đánh giá

Phần này để dùng, không để đọc. Nó làm hai việc: định vị bạn trên ba trục bằng hành vi quan sát được thay vì bằng cảm giác, rồi biến vị trí đó thành một việc cụ thể cho hai quý tới.

Một điều kiện để nó có giá trị: **mỗi mức bạn tự chọn phải kèm được một sự việc đã xảy ra trong 12 tháng qua**. Nếu không dẫn được sự việc, bạn chưa ở mức đó — bạn đang ở mức bạn tin mình có thể làm được, và hai thứ này lệch nhau đủ xa để làm sai toàn bộ kế hoạch. Cách kiểm tra rẻ nhất: đưa bảng cho hai người từng làm việc gần bạn và nhờ họ chọn giúp, độc lập. Chỗ họ chọn thấp hơn bạn là chỗ đáng nhìn.

### Trục 1 — Phạm vi (Scope)

Câu hỏi định nghĩa của trục này: *nếu bạn ra một quyết định tệ, bao nhiêu người và bao nhiêu hệ thống phải sống với hậu quả, trong bao lâu?*

| Mức | Hành vi quan sát được | Bằng chứng kiểm chứng được |
|---|---|---|
| **S1** | Bạn hoàn thành task trong một module do người khác chia; ranh giới công việc của bạn do người khác vẽ | Ticket được giao, không có quyết định nào của bạn ảnh hưởng ra ngoài module |
| **S2** | Bạn sở hữu một feature hoặc một module end-to-end; bạn quyết cách làm, không quyết có làm hay không | Có một phần hệ thống mà khi nó lỗi, người ta tìm bạn trước |
| **S3** | Bạn sở hữu một hệ thống hoặc một domain; 3–6 người thay đổi cách làm vì quyết định của bạn | Có ADR/RFC của bạn mà người khác phải tuân theo; có ít nhất một người nói được rằng họ làm khác đi vì bạn |
| **S4** | Quyết định của bạn ràng buộc 2–5 team mà bạn không có thẩm quyền với họ; hoặc bạn chịu trách nhiệm về kết quả của 5–10 người | Có một chuẩn, một platform, hoặc một ranh giới sở hữu do bạn đề xuất và nay đang được thi hành |
| **S5** | Bạn quyết cấu trúc, ngân sách, hoặc định hướng kỹ thuật ảnh hưởng 30+ người trong 1–3 năm | Có một quyết định của bạn mà hậu quả vẫn đang hiện ra sau 12 tháng, và bạn nói được nó đã đúng/sai ở đâu |

Bẫy hay gặp ở trục này: nhầm *số người bạn tiếp xúc* với *số người phải tuân theo quyết định của bạn*. Nếu không ai buộc phải làm theo, phạm vi thật là S2 kể cả khi bạn nói chuyện với cả công ty.

### Trục 2 — Độ mơ hồ (Ambiguity)

| Mức | Hành vi quan sát được | Bằng chứng kiểm chứng được |
|---|---|---|
| **A1** | Bạn nhận bài toán đã định nghĩa và đã chia nhỏ; tiêu chí đúng/sai do người khác đặt | Acceptance criteria được viết bởi người khác |
| **A2** | Bạn nhận yêu cầu ở mức mục tiêu, tự chọn cách làm; vẫn có người xác nhận bạn đúng | "Làm cho API này chịu được 3x tải" — mục tiêu rõ, cách mở |
| **A3** | Bạn nhận một vấn đề đã được nhận diện nhưng chưa được định nghĩa thành bài toán, và bạn định nghĩa nó | Bạn đã viết một tài liệu biến một lời phàn nàn ("hệ thống chậm") thành một bài toán có tiêu chí đo được |
| **A4** | Bạn tìm ra bài toán chưa ai nêu, thuyết phục người khác rằng nó là bài toán, và chịu được nhiều tháng không biết mình đúng | Có một việc bạn đã bắt đầu khi chưa ai yêu cầu, và sau đó tổ chức thừa nhận nó là ưu tiên |
| **A5** | Bạn định khung những bài toán mà kết luận chỉ kiểm chứng được sau 2–5 năm, và bạn nói được mức không chắc chắn của mình | Có một dự báo bạn đưa ra 2 năm trước, ghi lại được, và bạn đã đối chiếu nó |

Bẫy hay gặp: nhầm "bài toán khó" với "bài toán mơ hồ". Một bài toán thuật toán rất khó vẫn là A1–A2 nếu tiêu chí đúng/sai đã có sẵn.

### Trục 3 — Cách tạo ra kết quả (Mode of Impact)

| Mức | Hành vi quan sát được | Bằng chứng kiểm chứng được |
|---|---|---|
| **M1** | Kết quả tỷ lệ trực tiếp với giờ bạn làm; bạn nghỉ một tuần thì phần việc đó dừng | Toàn bộ đầu ra của bạn là code và tài liệu do bạn viết |
| **M2** | Bạn tăng đầu ra của người bên cạnh qua review, mentor, thiết kế trước rồi họ implement | Có người nói được cụ thể họ làm được việc gì nhờ bạn mà trước đó không làm được |
| **M3** | Bạn thiết kế cách làm việc của một nhóm nhỏ: ai làm gì, thứ tự, ranh giới, chuẩn review | Nhóm giao được ổn định trong 2 quý; bạn nghỉ một tuần không có gì đổ |
| **M4** | Bạn đổi điều kiện thay vì đổi hành vi từng người: ranh giới sở hữu, ngưỡng quyết định, quy trình tuyển, chuẩn bậc | Có một cơ chế do bạn đặt, có người sở hữu, và nó vẫn chạy đúng khi bạn không nhìn trong một quý |
| **M5** | Cơ chế bạn đặt vẫn tạo ra kết quả sau khi bạn rời vai đó, và người khác vận hành được nó mà không cần bạn giải thích | Một cơ chế của bạn còn sống sau khi bạn chuyển vai hoặc chuyển công ty |

Bẫy hay gặp: gán M4–M5 cho nhánh Manager. Một Principal Engineer có thể ở M5 mà không quản lý ai; một EM có thể ở M1 nếu chủ yếu nhảy vào làm hộ.

### Đối chiếu tổ hợp với bậc

| Tổ hợp điển hình | Bậc tương ứng | Chú ý |
|---|---|---|
| S2 / A2 / M1 | Mid-level | Trục cần nhìn tiếp thường là A |
| S3 / A2–A3 / M2 | Senior | Nếu M vẫn ở M1, bạn đang là một Mid-level rất nhanh |
| S3–S4 / A3 / M3 | Tech Lead (vai) | Nếu M ở M1–M2, bạn đang giữ title mà chưa làm vai |
| S4 / A4 / M2–M4 | Staff | Nếu S ở S3, bạn là một Senior xuất sắc, chưa phải Staff |
| S5 / A5 / M4–M5 | Principal | Trục A là trục quyết định, không phải S |
| S4 / A3 / M3–M4 | Engineering Manager | Nếu A ở A4–A5 mà bạn thích nó, cân nhắc lại nhánh |
| S5 / A4 / M4–M5 | Director | Nếu M ở M3, bạn đang làm EM của các EM |

Tổ hợp lệch là bình thường và là thông tin hữu ích nhất ở đây. Ví dụ S4 / A2 / M3 là mô tả khá chính xác một Delivery Manager giỏi: phạm vi rộng, việc đã rõ, tạo kết quả qua điều phối — một vai hoàn toàn hợp lệ, nhưng nếu đích của bạn là Staff thì trục cần làm là A, không phải S.

### Quy trình bốn bước cho hai quý tới

**Bước 1 — Định vị bằng bằng chứng.** Chọn một mức trên mỗi trục, mỗi mức kèm một sự việc trong 12 tháng qua. Nhờ hai người đối chiếu độc lập. Nếu có chênh lệch, lấy mức thấp hơn — không phải vì khiêm tốn, mà vì mức thấp hơn là mức người khác quan sát được, và phạm vi chỉ tồn tại khi người khác thấy nó.

**Bước 2 — Xác định đích và chọn đúng một trục.** Viết ra vai bạn muốn ở 2–4 quý tới (một vai cụ thể ở tổ chức cụ thể, không phải một title chung), tra bảng đối chiếu để biết cần mức nào trên ba trục, rồi tính khoảng cách. **Chọn đúng một trục — trục có khoảng cách lớn nhất.** Lý do chỉ một: hai quý chỉ đủ cho một lần thay đổi hành vi thật, và người chọn ba trục cùng lúc sẽ mặc định quay về trục mình đã mạnh nhất.

Một kiểm tra ở bước này: nếu đích bạn viết ra là một vai không tồn tại thật ở tổ chức hiện tại (xem [Bối cảnh Việt Nam](#bối-cảnh-việt-nam)), thì việc cần làm trong hai quý tới không phải phát triển năng lực — mà là đàm phán để bậc đó tồn tại, hoặc chuẩn bị đổi tổ chức. Đó là một kế hoạch khác hẳn.

**Bước 3 — Chọn một việc thật, không chọn một khoá học.** Việc đó phải thoả bốn điều kiện, và nếu thiếu một thì nó không luyện được trục nào: (a) có hậu quả thật nếu làm dở; (b) có người khác phụ thuộc vào nó; (c) kết thúc được trong một quý để bạn nhận được tín hiệu; (d) nó ép bạn vào đúng trục đã chọn ở bước 2, chứ không chỉ liên quan đến trục đó. Ví dụ về (d): nếu trục cần lên là A, thì "dẫn một epic đã có spec rõ" không tính — nó luyện S và M. Việc tính là "chọn một vấn đề chưa ai định nghĩa, tự viết định nghĩa và tiêu chí, mang đi thuyết phục hai team khác".

**Bước 4 — Đặt cơ chế đối chiếu trước khi bắt đầu.** Viết ra ngay từ đầu: tín hiệu quan sát được nếu thành công, tín hiệu nếu thất bại, ngày review, người đối chiếu, và điều kiện dừng. Bước này hay bị bỏ, và bỏ nó khiến toàn bộ ba bước trên trở thành một bản dự định — vì sáu tháng sau bạn sẽ nhớ chọn lọc và kết luận rằng mình đã tiến bộ.

Mẫu ghi lại, dùng được nguyên dạng:

```
=== ĐỊNH VỊ & KẾ HOẠCH HAI QUÝ ===
Người viết:            Ngày:            Review lại vào:

[1] ĐỊNH VỊ HIỆN TẠI (mỗi mức phải kèm một sự việc trong 12 tháng qua)
  Scope      = S__   bằng chứng: ..........................................
  Ambiguity  = A__   bằng chứng: ..........................................
  Mode       = M__   bằng chứng: ..........................................
  Hai người đối chiếu: ......... chọn S_/A_/M_ | ......... chọn S_/A_/M_
  Chỗ họ chọn thấp hơn tôi: ............................................

[2] ĐÍCH & TRỤC ĐƯỢC CHỌN
  Vai đích (cụ thể, ở tổ chức nào): ....................................
  Vai này có tồn tại thật ở đây không?  [ ] Có  [ ] Không  [ ] Không rõ
     -> nếu Không / Không rõ: việc của hai quý này là đàm phán hoặc
        chuẩn bị đổi tổ chức, không phải phát triển năng lực.
  Mức cần: S__ / A__ / M__
  Khoảng cách lớn nhất ở trục:  [ ] Scope  [ ] Ambiguity  [ ] Mode
  Trục được chọn (chỉ một): ............................................
  Trục tôi sẽ KHÔNG làm gì thêm trong hai quý này: .....................

[3] MỘT VIỆC THẬT
  Việc: ................................................................
  (a) Hậu quả thật nếu làm dở là gì:  ..................................
  (b) Ai phụ thuộc vào nó:            ..................................
  (c) Ngày kết thúc:                  ..................................
  (d) Nó ép tôi vào trục đã chọn ở chỗ nào (nói cụ thể):
      ................................................................
  Việc tôi sẽ BỎ để có thời gian cho nó: ...............................

[4] CƠ CHẾ ĐỐI CHIẾU
  Tín hiệu thành công (quan sát được, không phải cảm giác):
      ................................................................
  Tín hiệu thất bại:
      ................................................................
  Ba dự đoán tôi ghi lại hôm nay để đối chiếu sau:
      1. ..............................................................
      2. ..............................................................
      3. ..............................................................
  Người đối chiếu cùng tôi:  ...........  Ngày:  ......................
  Điều kiện dừng (khi nào tôi kết luận việc này không luyện được trục đó):
      ................................................................
```

Một lưu ý về cách dùng mẫu này: **phần khó nhất không phải mục [3] mà là hai dòng "việc tôi sẽ BỎ" và "trục tôi sẽ KHÔNG làm gì thêm".** Một kế hoạch phát triển chỉ có phần thêm vào là một kế hoạch chưa được ra quyết định, và nó sẽ bị đè bởi công việc thường ngày trong vòng ba tuần.

---

## Tự kiểm tra

Trả lời bằng sự việc đã xảy ra ở tổ chức của bạn, không bằng nguyên tắc chung.

1. Ba trục của bạn hiện ở mức nào, và với mỗi mức bạn dẫn được sự việc nào trong 12 tháng qua? Nếu có trục không dẫn được sự việc, đó là trục bạn đang tự đánh giá cao hơn thực tế.
2. Ở công ty bạn đang làm, bậc IC cao nhất *tồn tại thật* là bậc nào — nghĩa là đã có người ở đó, với dải lương chồng lấn dải lương EM? Nếu không có, kế hoạch nghề nghiệp hiện tại của bạn đang giả định điều gì?
3. Lần gần nhất bạn ra một quyết định kỹ thuật mà hậu quả của nó hiện ra sau ít nhất 12 tháng, và bạn còn ở đó để nhận nó — là khi nào? Nếu chưa có lần nào, trục Ambiguity của bạn chưa được kiểm chứng ở bất kỳ mức nào trên A3.
4. Nếu lương và title của nhánh IC và nhánh Manager ở công ty bạn bằng nhau, bạn chọn nhánh nào — và câu trả lời đó có khác với nhánh bạn đang đi?
5. Trong tổ chức của bạn, một người muốn quay lại nhánh IC sau khi làm EM sẽ được xử lý như thế nào? Đã có tiền lệ chưa, và tiền lệ đó dạy cho những người còn lại điều gì về chi phí của việc thử vai quản lý?
6. Với mỗi người báo cáo cho bạn (hoặc mỗi người bạn đang mentor), bạn nói được trục nào của họ đang thấp nhất so với bậc họ muốn không? Nếu không, các buổi 1-1 về phát triển của bạn đang bàn về cái gì?
7. Trong ba tháng qua, bạn đã dùng thời gian của mình theo cách phù hợp với bậc bạn *đang giữ*, hay theo cách phù hợp với bậc bạn *đã rời*? Dẫn ra tỷ lệ thời gian thật, không phải ý định.
8. Nếu bạn nghỉ ba tuần từ mai, có bao nhiêu quyết định bị treo chờ bạn? Con số đó nói gì về Mode of Impact thật của bạn, chứ không phải về mức độ bạn quan trọng?

---

## Liên kết chương khác

- [00-nen-tang-leadership.md](/series/engineering-leadership/00-nen-tang-leadership/) — Cung cấp phần khung: sự khác biệt giữa Leadership và Management, giữa Authority và Influence. Chương này áp dụng khung đó vào từng bậc, nhất là ở cột "Nguồn quyền lực" của bảng tổng quan.
- [01-self-leadership.md](/series/engineering-leadership/01-self-leadership/) — Xử lý phần mà chương này chỉ nêu ra: quản lý năng lượng và sự chú ý khi chân trời thời gian giãn ra và vòng phản hồi biến mất. Đọc cùng mục "loại năng lượng bị tiêu hao" ở phần chọn nhánh.
- [02-communication.md](/series/engineering-leadership/02-communication/) — Chứa chi tiết hai kỹ năng mà chương này chỉ liệt kê: dẫn cuộc trò chuyện khó (bậc EM) và executive communication (bậc Director).
- [03-team-leadership.md](/series/engineering-leadership/03-team-leadership/) — Đi sâu vào công việc hàng ngày của bậc Tech Lead và EM: Delegation, Psychological Safety, xử lý xung đột. Đây là chương thực hành cho hai bậc mà chương này chỉ mô tả hàm mục tiêu.
- [04-decision-making.md](/series/engineering-leadership/04-decision-making/) — Cột "Cách ra quyết định" trong bảng tám bậc được triển khai ở đó: phân loại quyết định theo tính đảo ngược, ngưỡng quyết định, và ai được phản biện. Cần thiết cho sai lầm "giữ lại những quyết định nên giao" ở bậc Director.
- [05-technical-leadership.md](/series/engineering-leadership/05-technical-leadership/) — Nội dung công việc của nhánh IC ở bậc Staff và Principal: ADR/RFC, chuẩn kiến trúc, cách tạo ảnh hưởng kỹ thuật khi không có thẩm quyền.
- [07-project-delivery.md](/series/engineering-leadership/07-project-delivery/) — Phần đàm phán scope và Estimation mà một Tech Lead phải làm được; đọc cùng phần chuẩn bị sáu tháng cho chuyển dịch Senior → Tech Lead.
- [08-hiring-va-phat-trien.md](/series/engineering-leadership/08-hiring-va-phat-trien/) — Triển khai hai đầu ra trung tâm của bậc EM và Director: tuyển như một quy trình lặp lại được, và phát triển người có chủ ý. Cũng chứa phần đọc bậc thật của ứng viên mà mục Bối cảnh Việt Nam chỉ nêu bảng câu hỏi.
- [09-to-chuc-va-scaling.md](/series/engineering-leadership/09-to-chuc-va-scaling/) — Công cụ chính của bậc Director: thiết kế tổ chức, Team Topologies, Cognitive Load, và cách ranh giới team tương tác với kiến trúc hệ thống.
- [10-case-studies.md](/series/engineering-leadership/10-case-studies/) — Các tình huống dài chạy trọn chuỗi bối cảnh → lựa chọn → trade-off → hậu quả, trong đó có những ca về chuyển dịch bậc mà chương này mô tả ở dạng khái quát.
- [12-anti-patterns.md](/series/engineering-leadership/12-anti-patterns/) — Tập hợp và mổ xẻ sâu hơn các mẫu sai đã nêu rải rác ở mục "Sai lầm thường gặp" của tám bậc, kèm dấu hiệu sớm và cách tháo.
