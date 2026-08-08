+++
title = "Level 1 — Self Leadership: Quản trị bản thân trước khi dẫn dắt người khác"
date = "2026-08-01T09:00:00+07:00"
draft = false
tags = ["engineering-leadership", "management", "career"]
series = ["Engineering Leadership & Technical Management"]
+++

# Level 1 — Self Leadership

Thứ Ba, 16h20. Một Tech Lead mở Slack và thấy: một câu hỏi từ QA để từ 10h sáng chưa trả lời, một
Pull Request của người mới treo hai ngày, một tin nhắn của PO hỏi "estimate hôm qua anh nói bao lâu
nhỉ", và một cuộc họp với khách Mỹ lúc 22h mà chưa ai chuẩn bị tài liệu. Người này không lười, không
thiếu năng lực kỹ thuật, và làm việc mười một giờ mỗi ngày. Nhưng team của họ đang chịu bốn thứ đo
được: commitment sprint hoàn thành 60% ba sprint liền, thời gian chờ review trung bình 31 giờ, hai
quyết định kiến trúc bị treo quá hai tuần vì "chờ anh xem lại", và người mới sau sáu tuần vẫn chưa
tự deploy được lần nào.

Đây là dạng thất bại đặc trưng của Level 1. Vấn đề không phải team kém. Vấn đề là **hệ thống cá nhân
của người lead đang là điểm nghẽn của hệ thống team**. Mọi hành vi lead — ra quyết định, review, cam
kết, đặt chuẩn — đều đi qua một con người có giới hạn attention, working memory và năng lượng. Khi
con người đó không quản lý được các giới hạn ấy, đầu ra của họ không phải là zero. Nó **âm**: nó trở
thành nguồn nhiễu (noise) cho những người phụ thuộc vào họ. Commitment không giữ được làm team mất
khả năng lập kế hoạch. Quyết định trễ làm hai người ngồi chờ. Chất lượng dao động theo mức mệt làm
chuẩn review mất giá trị tín hiệu — cùng một đoạn code có thể pass hoặc bị chặn tuỳ hôm.

Chương này không nói về việc "kỷ luật bản thân". Nó xử lý sáu hệ thống con có thể thiết kế được:
vùng chịu hệ quả (Ownership), phân bổ attention (Time Management), thứ tự việc (Prioritization), bộ
nhớ ngoài (Productivity System), cơ chế ra quyết định cá nhân, cơ chế học, và ngân sách năng lượng.
Truy theo chuỗi tư duy của bộ tài liệu: Level 1 nằm ở tầng **People**, nhưng nó là input của Process
và Execution. Một lead không giữ được commitment cá nhân thì mọi process do họ dựng lên đều mất hiệu
lực, vì process chạy bằng niềm tin rằng các cam kết trong đó là thật.

Mục lục nội bộ:

1. [Ownership ở cấp cá nhân](#1-ownership-ở-cấp-cá-nhân)
2. [Time Management cho engineer](#2-time-management-cho-engineer)
3. [Prioritization](#3-prioritization)
4. [Personal Productivity System](#4-personal-productivity-system)
5. [Decision Making ở cấp cá nhân](#5-decision-making-ở-cấp-cá-nhân)
6. [Continuous Learning có chủ đích](#6-continuous-learning-có-chủ-đích)
7. [Quản lý năng lượng, sustainable pace và burnout](#7-quản-lý-năng-lượng-sustainable-pace-và-burnout)
8. [Tự kiểm tra](#tự-kiểm-tra)
9. [Liên kết chương khác](#liên-kết-chương-khác)

---

## 1. Ownership ở cấp cá nhân

### Problem Statement

Một ODC ở TP.HCM, dự án cho khách Nhật, 14 engineer. Job đối chiếu dữ liệu chạy 2h sáng mỗi ngày.
Ngày 3 tháng trước, job fail. Minh — Senior Backend, người viết job đó — thấy alert trong channel
lúc 9h sáng, đọc log, xác định là do bên đối tác đổi format file, và viết vào channel: "cái này do
partner đổi format, cần BA xác nhận với họ". Rồi Minh quay lại task sprint của mình.

Ba tuần sau, khách phát hiện dữ liệu lệch 19 ngày. Không ai chối được rằng Minh đã báo. Ticket được
tạo, gán cho BA, BA nghỉ phép một tuần, ticket rơi vào backlog. Trong ba tuần đó không ai hỏi lại,
và alert 2h sáng vẫn nổ mỗi đêm cho tới khi có người mute channel vì "nhiều noise".

Hiện tượng quan sát được khi ownership ở cấp cá nhân yếu, không phải cảm nhận mà là số đếm:

- Số việc **rơi giữa hai người** trong một quý. Đo bằng cách rà các ticket có hơn một lần đổi
  assignee mà không có comment nào trong 5 ngày. Trong dự án trên: 11 việc trong 6 sprint, trung
  bình trễ 9 ngày mỗi việc.
- Số lần một vấn đề được **báo mà không được đóng**. Đo bằng tỉ lệ alert / message dạng "cái này do
  X" mà không có follow-up trong 48 giờ.
- Khoảng cách giữa "task tôi được gán" và "kết quả khách cần". Dấu hiệu: khi hỏi "tính năng này
  hoạt động chưa", câu trả lời là "phần của em xong rồi".
- Số alert bị mute hoặc bị bỏ qua theo thói quen. Đây là chỉ số ownership của cả hệ, không của một
  người.

Ở đầu ngược lại của phổ, cũng có một dạng hỏng dễ nhận: Hà, Tech Lead cùng công ty, nhận hết. Hà tự
sửa job của Minh, tự nhắc partner, tự viết tài liệu cho BA, tự làm hotfix cuối tuần. Số đo của Hà:
72% thời gian là việc không nằm trong kế hoạch, ba PR quan trọng nhất của quý đều do Hà viết, và khi
Hà nghỉ ba ngày thì hai luồng công việc dừng. Hà đang có ownership rất rộng và **hệ thống đang phụ
thuộc vào một điểm hỏng duy nhất là Hà**.

Hai hiện tượng này trông đối lập nhưng cùng gốc: không ai trong tổ chức phân biệt được **vùng chịu
hệ quả** với **vùng thực thi**. Nếu thiếu sự phân biệt đó, tổ chức chỉ có hai chế độ — đẩy việc đi
và gánh hết — và không có chế độ nào ở giữa.

### First Principles

**Cơ chế một: externality.** Kinh tế học gọi phần chi phí mà một tác nhân tạo ra nhưng người khác
chịu là externality. Khi Minh khép vùng chịu hệ quả của mình lại ở biên "code tôi viết đúng", phần
hệ quả còn lại (dữ liệu lệch 19 ngày, uy tín với khách, ba tuần công của người khác) không biến mất
— nó tràn ra ngoài và được chi trả bởi tổ chức. Ownership, hiểu theo cơ chế, là **hành động
internalize externality**: chủ động kéo phần hệ quả bên ngoài vào trong vùng mình chịu, để chính
mình có động cơ xử lý nó. Đây là lý do ownership không thể bị áp đặt từ trên xuống bằng lời: bạn có
thể ra lệnh cho một người làm việc, nhưng không thể ra lệnh cho họ **quan tâm tới hệ quả** — điều
đó phụ thuộc vào việc hệ quả có chạm tới họ hay không.

**Cơ chế hai: locus of control.** Tâm lý học (Julian Rotter, nghiên cứu công khai từ thập niên 1960)
mô tả một chiều cá nhân: người có internal locus of control quy kết kết quả về hành động của mình;
người có external locus quy kết về hoàn cảnh, may mắn, người khác. Điểm quan trọng cho công việc kỹ
thuật không phải là "internal thì tốt hơn". Điểm quan trọng là **locus of control quyết định bạn
nhìn thấy bao nhiêu lựa chọn**. Một người tin rằng nguyên nhân nằm ngoài mình sẽ ngừng tìm kiếm sau
khi xác định được nguyên nhân ngoài. Người tin rằng mình có phần sẽ hỏi tiếp: "với thông tin này,
tôi làm gì được?" — và câu hỏi đó mở ra tập hành động mà người kia không nhìn thấy. Ownership vì thế
là một **hàm sinh lựa chọn**, không phải một đức tính.

**Cơ chế ba: learned helplessness và mặt trái của nó.** Nghiên cứu công khai về learned helplessness
cho thấy khi hành động liên tục không tạo ra thay đổi trong kết quả, sinh vật ngừng hành động — kể
cả khi điều kiện đã đổi và hành động lại có tác dụng. Trong tổ chức, đây là cơ chế tạo ra kỹ sư
"chỉ làm đúng task được gán". Họ không sinh ra như vậy. Thường họ đã đề xuất ba lần, bị bác ba lần,
và đã học đúng bài học từ dữ liệu họ có. Hệ quả cho người lead: **muốn tăng ownership của một người,
việc đầu tiên phải làm không phải là nói về ownership, mà là làm cho một hành động của họ tạo ra
thay đổi quan sát được.**

**Extreme Ownership hiểu đúng.** Khái niệm này (Jocko Willink, nguồn công khai) bị đọc sai rất phổ
biến trong ngành thành "cái gì cũng là lỗi của tôi". Đọc như vậy tạo ra hai hệ quả tệ: người ta nhận
lỗi thay để kết thúc cuộc tranh luận (làm mất dữ liệu về nguyên nhân thật), và người ta gánh việc
vượt capacity (làm mất khả năng thực thi). Đọc đúng theo cơ chế: Extreme Ownership là **mở rộng biên
của vùng mình chịu hệ quả và hành động trong vùng đó**, không phải mở rộng biên của vùng mình nhận
lỗi. Hai thứ này khác nhau ở output. Nhận lỗi cho ra một câu nói. Mở rộng vùng chịu hệ quả cho ra
một hành động cụ thể: một tin nhắn cho partner, một ticket có deadline, một alert được sửa để không
nổ vô ích, một escalation.

Kiểm tra nhanh để biết mình đang làm cái nào: sau khi bạn "nhận ownership", có tồn tại một hành động
mà người khác quan sát được không? Nếu không, bạn vừa làm một hành vi ngôn từ.

**Cơ chế bốn: ownership cần ba thành tố mới hoạt động.** Từ principal-agent problem, một vai trò chỉ
thực sự chịu trách nhiệm khi có đủ: **thông tin** (biết trạng thái thật), **quyền** (thay đổi được
gì đó), và **hệ quả** (kết quả chạm tới mình). Thiếu thông tin, ownership thành đoán. Thiếu quyền,
ownership thành lo lắng không có đầu ra — đây chính là ownership giả. Thiếu hệ quả, ownership thành
diễn.

### Mental Model

**Mô hình 1 — Ba vòng: Control / Influence / Accept.**

```
   +-------------------------------------------------+
   |  ACCEPT — không tác động được trong horizon này  |
   |  (thị trường, quyết định C-level đã chốt,        |
   |   luật, việc partner đã ký)                      |
   |   +-----------------------------------------+    |
   |   |  INFLUENCE — tác động qua người khác     |    |
   |   |  (ưu tiên của PO, chuẩn của team khác,   |    |
   |   |   quyết định của cấp trên)                |   |
   |   |    +-------------------------------+      |   |
   |   |    |  CONTROL — mình quyết một mình |     |   |
   |   |    |  (code mình viết, cách mình    |     |   |
   |   |    |   phản hồi, thời gian mình đặt)|     |   |
   |   |    +-------------------------------+      |   |
   |   +-----------------------------------------+    |
   +-------------------------------------------------+
```

Ba vòng này không phải để phân loại cho gọn. Chúng là để **chọn loại hành động khác nhau**, vì dùng
hành động của vòng trong cho vấn đề vòng ngoài là nguồn gốc của phần lớn năng lượng bị đốt vô ích.
Với vòng Control, hành động đúng là *làm*. Với vòng Influence, hành động đúng là *xây lập luận và
chọn người*. Với vòng Accept, hành động đúng là *điều chỉnh kế hoạch của mình theo thực tế đó và
ngừng chi phí cảm xúc*. Sai lệch phổ biến: mang một vấn đề Influence ra bàn cà phê và bàn suốt ba
tuần (đốt năng lượng, không có hành động); hoặc coi một vấn đề Control là Accept ("hệ thống nó thế
rồi").

**Mô hình 2 — Consequence surface (bề mặt chịu hệ quả).**

Hình dung mỗi việc bạn làm có một bề mặt: từ lúc bạn commit code tới lúc người dùng cuối nhận giá
trị. Mỗi vai có một biên mặc định:

```
commit → CI xanh → merge → deploy → chạy production → khách dùng được → khách đạt mục tiêu
  |________ Junior _______|
  |_______________ Mid ____________|
  |_________________________ Senior ______________|
  |_________________________________ Tech Lead / Staff _________________|
```

Đây không phải là bảng title. Nó là **định nghĩa hoạt động của seniority**: seniority đo bằng độ dài
của bề mặt mà bạn tự động theo dõi mà không cần ai nhắc. Nâng cấp bản thân, hiểu theo mô hình này,
là kéo biên phải của mình sang phải một đoạn — và điều đó có chi phí thật, vì mỗi đoạn thêm vào là
thêm attention phải chi.

**Mô hình 3 — Ownership = f(thông tin, quyền, hệ quả).**

Dùng mô hình này khi bạn cảm thấy "tôi chịu trách nhiệm mà không làm gì được". Chẩn đoán bằng cách
hỏi thành tố nào đang bằng không, rồi đàm phán đúng thành tố đó — chứ không cố gắng bù bằng cách nỗ
lực nhiều hơn ở hai thành tố còn lại. Thiếu quyền thì không có lượng nỗ lực nào bù được.

### Practical Framework

**Bước 1 — Ownership Sort, 20 phút mỗi tuần.**

Input: danh sách mọi thứ đang làm bạn bận tâm trong tuần (lấy từ inbox, Slack đã đọc, ticket, và cả
những thứ chỉ ở trong đầu). Output: mỗi dòng có một nhãn và một hành động kế tiếp.

| Loại | Tiêu chí nhận biết | Hành động mặc định | Output bắt buộc | Biết là xong khi |
|---|---|---|---|---|
| **Control** | Bạn làm được mà không cần ai đồng ý | Làm ngay hoặc đặt vào block thời gian cụ thể | Một task có ngày | Việc đóng, không cần thông báo ai |
| **Influence** | Cần người khác quyết hoặc làm | Xác định *một* người quyết, chuẩn bị đề xuất một trang, chốt ngày trả lời | Một tin nhắn / một meeting request có agenda | Người đó nói yes/no rõ ràng, có ngày |
| **Accept** | Không đổi được trong 1 quý tới | Ghi lại, điều chỉnh kế hoạch, ngừng bàn | Một dòng trong ghi chú "constraint" | Bạn không mang nó ra bàn nữa |

Quy tắc chống tự lừa: một việc chỉ được vào **Accept** khi bạn viết được ra *ai* có quyền đổi nó và
*vì sao* bạn không tiếp cận được người đó trong quý này. Nếu không viết được, nó là Influence và bạn
đang dùng Accept làm chỗ trốn.

Quy tắc thứ hai: một việc **Influence** phải có tên một người, không phải tên một team. "Cần
Platform team hỗ trợ" là chưa xong việc phân loại. "Cần anh Nam (Platform Lead) duyệt quota, hỏi
trong stand-up thứ Năm" là xong.

**Bước 2 — Ownership Contract khi nhận một việc lớn.**

Khi bạn được giao một việc mà hệ quả kéo dài (một module, một tích hợp, một migration), hỏi năm câu
này *trước khi* bắt đầu, và ghi câu trả lời vào ticket hoặc ADR. Đây là năm câu để phát hiện
ownership giả sớm, khi chi phí sửa còn thấp:

1. Kết quả cuối cùng mà tổ chức muốn là gì, đo bằng gì? (không phải "làm xong tính năng X")
2. Tôi được quyết một mình những gì? Những gì phải xin? Xin ai?
3. Tôi cần gì từ ai để làm được, và người đó đã biết chưa?
4. Điều gì xảy ra nếu việc này trễ hai tuần? Ai bị ảnh hưởng đầu tiên?
5. Ngưỡng nào thì tôi escalate thay vì tự xoay? (đặt bằng số: trễ N ngày, hoặc chặn quá N giờ)

Câu 2 và câu 5 là hai câu hay bị bỏ, và là hai câu tạo ra phần lớn giá trị. Câu 2 lộ ra ngay khi bạn
được giao accountability mà không có authority. Câu 5 chuyển escalation từ hành vi "mách" — điều mà
văn hoá Việt thường thấy khó — thành một điều khoản đã thoả thuận trước, do đó không mang màu sắc cá
nhân khi xảy ra.

**Bước 3 — Quy tắc "báo và không rời".**

Khi bạn phát hiện một vấn đề không thuộc vùng thực thi của mình, làm ba việc thay vì một:

- Báo cho đúng một người có quyền xử lý (không báo vào channel chung rồi coi là xong).
- Đề xuất một hành động cụ thể kèm ước lượng — kể cả khi bạn không phải người làm.
- Đặt một reminder cho chính mình. Nếu sau N ngày chưa đóng, bạn hỏi lại một lần, rồi escalate.

Đây là toàn bộ khác biệt giữa Minh trong tình huống đầu chương và một Senior thực sự. Chi phí thêm:
khoảng 4 phút và một reminder. Giá trị: bịt được lỗ rơi việc — nơi tạo ra phần lớn sự cố kiểu
"không ai biết là nó đang hỏng".

### Trade-off

| Trục | Nghiêng về mở rộng ownership khi | Nghiêng về giữ biên hẹp khi | Ai chịu phần mất |
|---|---|---|---|
| **Mở rộng vùng chịu hệ quả vs giữ focus** | Việc đang rơi giữa các vai; bạn là người duy nhất có thông tin; hệ quả nghiêm trọng và không đảo được | Bạn đang ở giới hạn capacity; có người khác có quyền và thông tin tốt hơn; việc đó có owner rõ và đang chạy | Mở rộng quá: deliverable chính của bạn trượt, và team học rằng có người sẽ hứng — nên ngừng tự xử |
| **Tự làm vs escalate** | Việc nằm trong Control, chi phí thấp, làm xong là đóng hẳn | Vấn đề là hệ thống, sẽ lặp lại; sửa lần này che mất tín hiệu cho người có quyền sửa gốc | Tự làm quá nhiều: tổ chức không bao giờ nhận được tín hiệu, root cause tồn tại vô hạn |
| **Nhận accountability không có authority vs từ chối** | Bạn đang xây uy tín, việc nhỏ, thời hạn ngắn, và bạn đàm phán được quyền dần | Hệ quả lớn, thời hạn dài, và bạn đã xin quyền hai lần không được | Nhận: bạn chịu phần mất một mình, tổ chức có ảo giác việc đã có người lo — nên không sửa cấu trúc |
| **Ownership cá nhân vs ownership hệ thống** | Giai đoạn khẩn cấp, cần một người quyết nhanh | Sau khẩn cấp: phải chuyển sang owner có tên trong tài liệu, có Runbook | Giữ ở cá nhân: bus factor = 1, và người đó không nghỉ được |

Trade-off cần nói thẳng nhất: **mở rộng ownership luôn có giá, và giá đó thường được trả bằng thứ
không ai nhìn thấy.** Khi bạn nhận thêm một vùng, phần bị hy sinh không phải là "thời gian rảnh" mà
là độ sâu của việc quan trọng nhất bạn đang làm. Với một Tech Lead, thứ bị hy sinh đầu tiên thường
là chất lượng của việc suy nghĩ về kiến trúc — thứ không có ai phàn nàn trong ba tháng đầu và tạo ra
hoá đơn trong ba năm sau.

### Real-world Scenarios

**Tình huống A — Ownership giả trong ODC: accountability không có authority.**

Bối cảnh: dự án ODC cho khách Singapore, 20 engineer. Tuấn (Senior BE, 6 năm) được EM giao "own
performance của hệ thống", sau khi khách phàn nàn API chậm. Không có thay đổi nào về quyền: schema
do kiến trúc bên khách quyết, việc thêm index phải qua DBA của khách với SLA 5 ngày, và Tuấn không
được vào backlog để đưa việc tối ưu vào sprint.

Sau một tháng: Tuấn có một tài liệu phân tích 12 trang, ba lần bị hỏi trong weekly "sao chưa cải
thiện", và p95 không đổi. Tuấn bắt đầu ở lại muộn để làm việc tối ưu ngoài sprint, chất lượng task
chính giảm, và trong performance review được ghi "chưa đạt kỳ vọng ở mục ownership".

Đây là ownership giả, và nó gây hại theo hai chiều: Tuấn chịu hệ quả của việc mình không có quyền
đổi, còn tổ chức thì tin rằng vấn đề performance "đã có người phụ trách" nên không cấp nguồn lực.

Cách xử lý đúng — script cho buổi 1-1 với EM:

> **Tuấn:** Em muốn chốt lại phạm vi việc performance, vì em nghĩ hiện tại nó đang không chạy được
> và em muốn nói rõ vì sao trước khi nó thành vấn đề lớn hơn.
>
> **EM:** Ừ, anh cũng thấy một tháng rồi chưa có gì.
>
> **Tuấn:** Đúng, em nhận phần đó. Em đã phân tích ra ba nguyên nhân chính, chiếm khoảng 80% độ trễ:
> thiếu ba index, N+1 ở luồng listing, và một call đồng bộ sang service của khách. Cả ba việc em đều
> không tự làm được: index phải qua DBA bên khách, sửa N+1 cần 5 ngày dev mà em không có quyền đưa
> vào sprint, và cái thứ ba phải khách đổi.
>
> **EM:** Vậy em cần gì?
>
> **Tuấn:** Em đề nghị hai lựa chọn. Một, anh giao em quyền đưa tối đa 20% capacity sprint vào việc
> performance trong 6 tuần, và anh mở kênh để em gửi yêu cầu index trực tiếp cho DBA bên khách. Với
> điều kiện đó em cam kết p95 giảm 40% sau 6 tuần và em chịu kết quả này. Hai, nếu không có hai
> quyền đó, em đề nghị đổi mô tả việc của em thành "phân tích và đề xuất", còn accountability về kết
> quả p95 thuộc về anh, vì anh là người đàm phán được capacity và kênh với khách. Em vẫn làm hết
> phần phân tích.
>
> **EM:** Nói thế nghe như em đẩy việc.
>
> **Tuấn:** Em hiểu vì sao nghe như vậy. Nhưng nếu giữ nguyên như một tháng qua thì kết quả sẽ giống
> một tháng qua: em bận, anh bị hỏi, và p95 không đổi. Em thà nói thẳng bây giờ còn hơn để đến review
> cuối quý. Em nghiêng về lựa chọn một — em muốn làm việc này, chỉ cần đủ quyền.

Điểm cần chú ý trong script: Tuấn không phủ nhận phần của mình, đưa dữ liệu trước khi đưa yêu cầu,
đề xuất *hai* lựa chọn thay vì một lời từ chối, và gắn cam kết cụ thể vào lựa chọn có quyền. Đây là
cách nói "tôi không có quyền" mà không kích hoạt phản ứng "nhân viên này đang từ chối việc" — điều
quan trọng trong môi trường Việt Nam nơi việc phản biện cấp trên dễ bị đọc là thái độ.

**Tình huống B — Ba góc nhìn về một alert bị mute.**

Sự việc: channel `#alert-batch` có 40 tin mỗi ngày, 90% là noise. Một người mute channel. Ba tuần
sau, một alert thật bị bỏ qua và gây sự cố dữ liệu.

*Nhìn từ IC:* "Alert đó không phải của tôi, và channel nhiễu quá nên tôi mute." Với dữ liệu mà IC
có, hành động này hợp lý. Ownership ở cấp IC không phải là đọc hết 40 tin — đó là chi phí attention
không bền. Ownership đúng ở cấp này là: mute *và* tạo một ticket "alert này nổ sai, cần sửa điều
kiện", gán cho người có quyền, kèm reminder tự nhắc sau một tuần. Đây là hành động Control (tạo
ticket, ghi lại) cộng với Influence (thúc đúng một người).

*Nhìn từ Tech Lead:* alert noise không phải vấn đề của một người — nó là **hệ thống phát tín hiệu đã
mất giá trị thông tin**. Ownership ở cấp Tech Lead là chịu hệ quả cho chất lượng tín hiệu của team:
đặt một quy tắc rằng mỗi alert phải có owner và ngưỡng có thể hành động, và mỗi alert nổ mà không cần
hành động thì bị coi là bug phải sửa trong sprint. Tech Lead xử lý sai khi họ tự đi sửa 40 alert vào
cuối tuần — giải quyết triệu chứng, không đổi cơ chế sinh ra triệu chứng.

*Nhìn từ Manager:* việc alert bị mute là một tín hiệu về **incentive**. Không ai được ghi nhận cho
việc sửa alert; mọi người được ghi nhận cho việc đóng ticket tính năng. Ownership ở cấp Manager là
chịu hệ quả cho việc hệ thống khen thưởng đang tạo ra hành vi gì: đưa "chất lượng tín hiệu vận hành"
vào tiêu chí đánh giá, cấp thời gian cho nó, và làm cho việc sửa alert được nhìn thấy. Manager xử lý
sai khi họ họp team và nói về "tinh thần trách nhiệm" — cách này không đổi được incentive, chỉ thêm
một lớp cảm giác tội lỗi lên hành vi vốn hợp lý.

Cùng một sự việc, ba vùng chịu hệ quả khác nhau, ba loại hành động khác nhau. Nếu ba tầng đều nghĩ
mình có cùng loại ownership, hoặc cả ba đều nghĩ mình không có, sự việc sẽ tái diễn.

### Best Practices

**Chuyển mọi phát hiện thành một trong hai thứ: một hành động hoặc một dòng constraint được ghi.**
Lý do: một phát hiện chưa được chuyển hoá sẽ chiếm tài nguyên nhận thức của bạn (xem mục 4 về hiệu
ứng Zeigarnik) mà không tạo ra giá trị nào cho tổ chức. Nó là dạng tệ nhất: bạn trả chi phí, tổ chức
không nhận được lợi ích.

**Khi báo vấn đề, luôn kèm một đề xuất và một ước lượng, kể cả khi bạn không phải người làm.** Lý do:
người có quyền quyết hầu như không thiếu thông tin về sự tồn tại của vấn đề — họ thiếu một phương án
đủ cụ thể để so sánh với các việc khác. Báo trần trụi ("chỗ này chậm") đẩy toàn bộ chi phí phân tích
sang họ, và việc của bạn sẽ nằm sau những việc đã có phương án.

**Đàm phán quyền tại thời điểm nhận việc, không sau khi trễ.** Lý do: tại thời điểm nhận việc, cuộc
đối thoại là về thiết kế công việc; sau khi trễ, cùng nội dung đó bị đọc là bào chữa. Đây là hiệu
ứng framing thuần, và nó rất mạnh trong tổ chức Việt Nam nơi kết quả trễ dễ bị gán về thái độ.

**Với mỗi vùng ownership bạn nhận, viết ra một tiêu chí chuyển giao.** Lý do: ownership không có
đường ra sẽ tích tụ. Sau hai năm, bạn giữ 11 vùng, mỗi vùng chiếm một ít attention, và bạn không thể
nghỉ phép. Tiêu chí có thể là: "khi có Runbook và một người khác đã xử lý được hai lần thì tôi
chuyển giao".

**Phân biệt rõ trong ngôn từ: "tôi chịu hệ quả" và "tôi sẽ làm".** Lý do: gộp hai câu này là nguồn
gốc của martyr complex. Một Tech Lead có thể nói thật lòng "tôi chịu hệ quả nếu việc này trượt" và
đồng thời "người làm là Linh, tôi review vào thứ Tư" — hai câu này không xung đột, và tách chúng ra
là điều kiện để Delegation hoạt động (xem `03-team-leadership.md`).

### Anti-patterns

**Martyr complex — nhận hết việc rồi nghẽn.**
Cơ chế phá hệ thống: bạn trở thành một hàng đợi có một server. Theo Queueing Theory, khi utilization
của một server tiến tới 100%, thời gian chờ tăng phi tuyến — không phải tăng dần mà bốc lên. Vì vậy
người nhận hết việc không "chậm hơn một chút"; họ tạo ra độ trễ không dự đoán được cho tất cả những
ai phụ thuộc vào họ. Hệ quả thứ hai nặng hơn: team học được rằng có người sẽ hứng, nên họ ngừng phát
triển năng lực hứng. Sau một năm, bạn đúng là người duy nhất làm được — không phải vì bạn giỏi hơn
mà vì bạn đã hấp thụ hết cơ hội thực hành của người khác.
Dấu hiệu sớm: bạn là người comment trong hơn 60% ticket của team; câu "để anh làm cho nhanh" xuất
hiện hơn hai lần một tuần; bạn không nghỉ phép trọn một tuần trong sáu tháng; người khác hỏi bạn
những câu mà tài liệu đã trả lời.

**Ownership giả — accountability không có authority.**
Cơ chế: tổ chức nhận được tín hiệu "việc này đã có người lo" nên không cấp nguồn lực, trong khi
người được gán không thay đổi được biến số nào. Kết quả là vấn đề tồn tại lâu hơn so với việc không
gán cho ai — vì khi không gán cho ai, nó còn nổi lên trong các cuộc họp.
Dấu hiệu sớm: bạn được hỏi về tiến độ nhiều hơn số lần bạn được quyết định; mọi phương án của bạn
đều kết thúc bằng "phải chờ X đồng ý"; bạn làm việc ngoài giờ cho việc này vì trong giờ không có
capacity chính thức.

**"Tôi đã báo rồi" như một cơ chế phòng thân.**
Cơ chế: câu này chuyển ownership thành một hành vi tạo bằng chứng. Nó tối ưu cho việc bảo vệ bản
thân trong cuộc điều tra tương lai, không tối ưu cho việc vấn đề được đóng. Tổ chức nào để câu này
hoạt động hiệu quả sẽ dần có nhiều người báo và ít người sửa.
Dấu hiệu sớm: các message dạng "cái này do team X" không có ai reply; số lượng ticket được tạo tăng
nhưng thời gian đóng trung bình cũng tăng; trong postmortem, phần lớn thời gian dành cho việc dựng
lại timeline ai biết gì lúc nào.

**Ownership bị dùng làm công cụ quy tội cá nhân cho lỗi hệ thống.**
Cơ chế: khi mỗi sự cố kết thúc bằng việc xác định một người "thiếu ownership", tổ chức có một cách
giải thích rẻ và không phải sửa cấu trúc. Chi phí trả sau: người ta giảm phạm vi hành động để giảm
rủi ro bị quy kết — chính xác là hành vi ngược với ownership.
Dấu hiệu sớm: postmortem có action item dạng "nhắc nhở team cẩn thận hơn"; người ta hỏi "ai làm cái
này" trước khi hỏi "cơ chế nào cho phép việc này xảy ra".

**Ownership theo cảm xúc: lo lắng thay cho hành động.**
Cơ chế: bạn mang một vấn đề vòng Influence hoặc Accept trong đầu suốt nhiều tuần. Nó tiêu thụ
attention như một việc đang mở, nhưng không có bước kế tiếp nên không bao giờ đóng. Đây là dạng
ownership tốn nhiều nhất và tạo ra ít nhất.
Dấu hiệu sớm: bạn nói về cùng một vấn đề trong ba buổi 1-1 liên tiếp mà không có gì thay đổi giữa
các buổi.

### Khi nào KHÔNG nên áp dụng

**Khi bạn đã ở hoặc vượt giới hạn capacity.** Mở rộng vùng ownership trong trạng thái này không tạo
ra kết quả tốt hơn — nó chỉ chuyển việc trượt từ vùng mới sang vùng cũ, và làm cả hai vùng trượt
không dự đoán được. Trong điều kiện này, hành động ownership đúng là **đóng bớt vùng**: chuyển giao
tường minh một vùng cho người khác, kèm tài liệu, thay vì âm thầm giảm chất lượng ở mọi vùng. Câu
nói tương ứng: "Em nhận việc này, nhưng em cần bỏ việc Y. Anh chọn giúp em bỏ cái nào."

**Khi bạn không có quyền và đã đàm phán quyền thất bại hai lần.** Tiếp tục "chịu trách nhiệm" trong
điều kiện này gây hại cho cả bạn và tổ chức: bạn kiệt sức, tổ chức mất tín hiệu rằng vấn đề chưa có
người xử lý được. Hành động đúng là làm cho việc thiếu quyền trở nên hữu hình — ghi vào ticket, nêu
trong weekly, đưa vào Decision Log — rồi giới hạn phần cam kết của mình ở phần bạn kiểm soát được.

**Khi vấn đề tái diễn theo hệ thống và việc bạn tự xử che mất tín hiệu.** Nếu một tích hợp fail mỗi
tuần và bạn tự sửa mỗi tuần trong 20 phút, bạn đang trả một khoản nhỏ để duy trì vô hạn một khoản
lớn. Trong trường hợp này, để nó fail *một lần có kiểm soát* (đã cảnh báo trước, đã chuẩn bị phương
án) là hành động ownership cao hơn việc âm thầm sửa. Điều kiện biên: chỉ làm vậy khi hệ quả của lần
fail đó có thể chịu được và bạn đã thông báo cho người chịu hệ quả — không áp dụng với hệ thống
fintech/thanh toán nơi một lần fail có chi phí compliance thật.

**Trong 30–60 ngày đầu ở tổ chức mới.** Vùng ownership mở rộng quá nhanh khi bạn chưa hiểu bối cảnh
tạo ra hai rủi ro: bạn giải một vấn đề đã được cân nhắc và cố tình để vậy (Chesterton's fence), và
bạn tiêu hết uy tín ban đầu vào việc không quan trọng. Trong giai đoạn này, ưu tiên là **mở rộng
thông tin trước, mở rộng ownership sau**.

**Khi tổ chức đang dùng ownership làm ngôn ngữ để đẩy rủi ro xuống cá nhân.** Dấu hiệu: mọi cuộc họp
kết thúc bằng việc gán tên người cho những việc mà nguồn lực chưa được quyết. Trong môi trường như
vậy, nhận thêm ownership không xây được uy tín — nó chỉ tăng bề mặt để bạn bị quy kết. Việc đúng cần
làm là thay đổi cơ chế (nếu bạn có đủ influence), hoặc ghi lại rõ ràng điều kiện kèm theo mỗi lần
nhận việc. Nếu cả hai đều không khả thi trong nhiều quý, đây là dữ liệu về tổ chức, không phải về
bạn — và nó thuộc phạm vi `11-career-evolution.md`.

---

## 2. Time Management cho engineer

### Problem Statement

Khoa là Tech Lead một team 7 người tại một startup fintech ở Hà Nội. Tuần vừa rồi Khoa làm 54 giờ.
Xuất từ calendar và log Slack, đây là bức tranh thật:

| Hạng mục | Số giờ | Số lần cắt |
|---|---|---|
| Họp có lịch trước | 11h | 9 buổi |
| Họp phát sinh trong ngày (không có trên calendar khi bắt đầu tuần) | 6h | 7 buổi |
| Trả lời Slack/Zalo trong giờ làm | ~5h (ước từ 240 message) | 60+ |
| Review code | 4h | 14 lần mở ra rồi bỏ dở |
| Việc kỹ thuật cần tập trung | 9h | 22 lần bị cắt |
| Việc hành chính (report, timesheet, phê duyệt) | 3h | — |
| Không xác định được đã làm gì | 16h | — |

Đầu ra thực tế của tuần: một PR 120 dòng, một tài liệu thiết kế viết được 40%, và hai quyết định
kiến trúc bị hoãn sang tuần sau. Cùng tuần đó Khoa cảm thấy "làm việc cả tuần không nghỉ".

Hiện tượng quan sát được khi Time Management ở cấp cá nhân yếu — đo được, không phải cảm giác:

- **Số block liên tục dài hơn 90 phút trong tuần.** Ở nhiều Tech Lead Việt Nam, con số này là 0–2.
- **Thời gian chờ review trung bình.** Khi lead không có block cố định cho review, phân phối thời
  gian chờ có đuôi rất dài: median 4 giờ nhưng p90 là 2 ngày.
- **Tỉ lệ deliverable của chính mình bị đẩy sang tuần sau.** Nếu quá 50% trong ba tuần liền, đây
  không phải vấn đề nỗ lực.
- **Số giờ "không xác định được đã làm gì".** Đây là chỉ số quan trọng nhất và ít ai đo. 16 giờ trên
  54 giờ nghĩa là 30% thời gian đi vào chỗ không tạo ra đầu ra nào ghi nhận được.
- **Việc quan trọng nhất trong quý được làm vào lúc nào.** Nếu nó luôn được làm sau 20h, bạn đang
  dùng thời gian có chất lượng thấp nhất cho việc có leverage cao nhất.

Hệ quả lan xuống team, và đây mới là phần đắt: khi lead không có thời gian có thể dự đoán được, team
mất khả năng lập kế hoạch dựa vào lead. Người ta bắt đầu chặn (block) chờ quyết định, hoặc tự quyết
mà không có bối cảnh. Cả hai đều tệ hơn việc lead nói thẳng "tôi trả lời loại câu hỏi này vào 14h
mỗi ngày".

### First Principles

**Nguyên lý một: hai loại thời gian có hình dạng khác nhau.** Paul Graham (bài luận công khai
*Maker's Schedule, Manager's Schedule*) mô tả hai chế độ. Manager's schedule chia ngày thành các ô
một giờ; đơn vị công việc là cuộc trò chuyện; một ô trống là một ô có thể lấp. Maker's schedule cần
đơn vị nửa ngày; một cuộc họp giữa buổi sáng không lấy đi một giờ — nó **phá vỡ cả buổi sáng thành
hai mảnh, mỗi mảnh quá ngắn để bắt đầu việc khó**.

Điểm cốt lõi không phải là "họp thì xấu". Điểm cốt lõi là **hai lịch này không cộng được với nhau
theo phép cộng số học**. Một người ở vai Tech Lead phải chạy đồng thời cả hai chế độ, và đó là nguồn
gốc thật của cảm giác kiệt sức ở vai này — không phải khối lượng, mà là việc liên tục chuyển giữa hai
hình dạng thời gian không tương thích.

**Nguyên lý hai: attention residue.** Nghiên cứu công khai của Sophie Leroy (2009) về "attention
residue" cho thấy khi chuyển từ task A sang task B mà A chưa hoàn thành hoặc chưa được đóng lại một
cách rõ ràng, một phần attention vẫn còn dính ở A và làm giảm hiệu suất ở B. Đây là cơ chế giải thích
tại sao "chỉ 5 phút thôi" không hề chỉ tốn 5 phút: chi phí thật gồm 5 phút gián đoạn cộng thời gian
tái nạp bối cảnh cộng phần suy giảm hiệu suất do residue.

**Nguyên lý ba: chi phí tái nạp bối cảnh không tuyến tính theo độ phức tạp của việc.** Việc kỹ thuật
khó đòi hỏi giữ trong working memory một mô hình gồm nhiều thành phần liên hệ với nhau: trạng thái
dữ liệu, các nhánh điều kiện, giả định về hệ thống ngoài, giả thuyết đang kiểm tra. Working memory
của người trưởng thành giữ được rất ít đơn vị đồng thời (nghiên cứu công khai từ Miller đến Cowan cho
các con số khác nhau, cỡ 4–7). Khi bị cắt, mô hình đó không được lưu ở đâu ngoài đầu bạn, và phải
dựng lại gần như từ đầu.

Từ đó suy ra kết luận thực dụng nhất của mục này: **4 giờ liền không tương đương 8 lần 30 phút.**

```
Kịch bản A — một block 4 giờ:
[ 20' nạp bối cảnh ][ ---------- 3h40 làm việc năng suất ---------- ]
Đầu ra: giải được một vấn đề cần giữ mô hình phức tạp.

Kịch bản B — 8 block 30 phút:
[15' nạp][15' làm][15' nạp][15' làm] x 4 ... + attention residue mỗi lần chuyển
Thời gian nạp bối cảnh: ~2h. Thời gian làm: ~2h, ở mức chất lượng thấp hơn.
Đầu ra: xử lý được việc nhỏ, gần như không thể giải việc cần giữ mô hình phức tạp.
```

Nói cách khác, quan hệ giữa độ dài block và giá trị đầu ra không phải đường thẳng — nó có ngưỡng.
Dưới một ngưỡng nào đó (với phần lớn việc kỹ thuật là khoảng 60–90 phút), block gần như không dùng
được cho việc khó; nó chỉ dùng được cho việc cơ học. Đây là lý do một tuần 54 giờ có thể cho ra ít
hơn một tuần 40 giờ được cấu trúc tốt.

**Nguyên lý bốn: thời gian là tài nguyên có chất lượng thay đổi theo thời điểm.** Attention không
phân bố đều trong ngày, và không phục hồi tức thời sau khi bị tiêu. Một giờ lúc 9h sáng và một giờ
lúc 21h không phải cùng một tài nguyên, dù trên timesheet chúng bằng nhau. Hệ quả: mọi cách quản lý
thời gian chỉ đếm giờ đều sai về bản chất; phải đếm giờ **theo loại chất lượng** (xem thêm mục 7 về
quản lý năng lượng).

### Mental Model

**Mô hình 1 — Calendar như một hệ thống có bốn loại tài nguyên.**

Thay vì coi calendar là danh sách cuộc hẹn, coi nó là bảng phân bổ bốn loại thời gian có tính chất
khác nhau:

| Loại | Đặc tính | Có thể gộp? | Có thể cắt? | Thời điểm tối ưu |
|---|---|---|---|---|
| **Deep Work** | Cần block ≥ 90 phút, chi phí gián đoạn rất cao | Không — 2 block 45' không bằng 1 block 90' | Không | Giờ attention cao nhất của bạn |
| **Collaboration** | Cần đồng bộ với người khác; giá trị nằm ở tương tác | Có — gộp thành cụm để bảo vệ block Deep Work | Không nên | Giữa buổi, gần nhau |
| **Reactive** | Trả lời, unblock người khác, xử lý phát sinh | Có | Có | Cuối buổi sáng và cuối buổi chiều |
| **Admin** | Report, timesheet, phê duyệt, xử lý email hành chính | Có | Có | Lúc attention thấp — đây là ưu điểm, không phải nhược điểm |

Mô hình này cho ra một nguyên tắc thiết kế ngược trực giác: **việc Admin nên được xếp vào lúc bạn
mệt nhất.** Đa số người làm ngược lại — xử lý email và timesheet vào 9h sáng khi vừa vào ca — và
tiêu tài nguyên đắt nhất vào việc chịu được tài nguyên rẻ.

**Mô hình 2 — Queueing Theory áp cho attention của một người.**

Bạn là một server. Mỗi câu hỏi, mỗi review, mỗi quyết định là một job vào hàng đợi. Hai kết luận từ
lý thuyết này áp trực tiếp:

- **Utilization càng gần 100%, thời gian chờ càng bốc lên phi tuyến.** Một lead lấp kín calendar
  không phải là lead hiệu quả — đó là hệ thống không có slack, nên mọi việc phát sinh đều tạo ra độ
  trễ lan truyền. Slack (thời gian trống có chủ đích) không phải sự xa xỉ; nó là **thành phần thiết
  kế** để giữ thời gian chờ có thể dự đoán được.
- **Batching giảm chi phí setup.** Xử lý 10 câu hỏi trong một block 45 phút rẻ hơn nhiều so với 10
  lần gián đoạn, vì chi phí setup (nạp bối cảnh) chỉ trả một lần.

**Mô hình 3 — Đường phân định: Interrupt-driven vs Schedule-driven.**

Hình dung công việc của bạn có hai chế độ vận hành, giống hai chế độ trong kernel:

```
Interrupt-driven: mọi tín hiệu bên ngoài đều có quyền chiếm CPU ngay
   → Ưu: latency thấp cho người khác. Nhược: throughput của bạn gần 0 cho việc khó.

Schedule-driven: tín hiệu vào queue, được xử lý theo lịch đã công bố
   → Ưu: throughput cao, latency dự đoán được. Nhược: latency trung bình cao hơn.
```

Không có chế độ nào đúng tuyệt đối. Câu hỏi thiết kế là: **loại tín hiệu nào xứng đáng được
interrupt?** Danh sách nên ngắn và được công bố cho team: production incident, khách hàng đang bị
chặn, và một người trong team bị block hoàn toàn không tự đi tiếp được. Mọi thứ khác vào queue.
Việc công bố danh sách này quan trọng hơn nội dung của nó, vì nó chuyển kỳ vọng của team từ "hy vọng
lead rảnh" sang "biết lead trả lời lúc nào".

### Practical Framework

**Bước 1 — Calendar Audit, làm một lần 45 phút, rồi 10 phút mỗi tuần.**

Input: calendar và lịch sử Slack của hai tuần vừa qua. Cách làm:

1. Gán nhãn mọi block đã xảy ra vào bốn loại: Deep Work / Collaboration / Reactive / Admin. Thêm
   nhãn thứ năm: **Unaccounted** cho thời gian bạn ở công ty nhưng không truy được đầu ra.
2. Tính tổng theo từng loại và tính tỉ lệ.
3. Đếm số block Deep Work liên tục ≥ 90 phút.
4. Với mỗi buổi họp, đánh dấu một trong ba: *cần tôi quyết* / *cần tôi có thông tin* / *tôi chỉ ngồi
   nghe*. Loại thứ ba là danh sách ứng viên để rút khỏi hoặc thay bằng đọc biên bản.

Output: một bảng như bảng của Khoa ở đầu mục, cộng ba số: tỉ lệ Deep Work, số block ≥ 90 phút, và số
giờ Unaccounted. Tiêu chí biết là xong: bạn viết được ra **một** thay đổi cụ thể cho tuần tới.

Vùng tham chiếu để so sánh (không phải chuẩn, là điểm khởi đầu để bạn tự hiệu chỉnh):

| Vai | Deep Work | Collaboration | Reactive | Admin |
|---|---|---|---|---|
| Senior IC | 55–65% | 15% | 15% | 5–10% |
| Tech Lead | 30–40% | 30% | 20% | 10% |
| Engineering Manager | 10–20% | 45% | 25% | 15% |

Nếu bạn là Tech Lead mà Deep Work đang ở 10%, có hai khả năng: bạn đang thực chất làm vai Manager mà
không được công nhận, hoặc bạn đang để Reactive ăn hết. Hai chẩn đoán này dẫn tới hai hành động khác
nhau — cái đầu là cuộc đối thoại về vai trò với cấp trên, cái sau là vấn đề thiết kế lịch.

**Bước 2 — Time blocking có điều kiện phòng thủ.**

Time blocking chỉ hoạt động nếu block chịu được áp lực. Bốn điều kiện làm nó chịu được:

1. **Block phải có tên nội dung cụ thể**, không phải "Deep Work". Ghi "Thiết kế lược đồ reconcile
   v2". Block có tên chung là block đầu tiên bị người khác đặt họp lên, và cũng là block bạn tự bỏ.
2. **Block phải được công bố.** Trong stand-up hoặc channel team: "10h–12h thứ Ba và thứ Năm tôi
   không nhận họp, trừ incident. Câu hỏi gửi vào channel, tôi trả lời lúc 14h."
3. **Phải có block Reactive tương ứng.** Đây là điều kiện quan trọng nhất và hay bị bỏ. Bạn không
   thể chỉ đóng cửa; bạn phải nói rõ cửa mở lúc nào. Không có phần thứ hai, việc bảo vệ thời gian bị
   đọc là khó tính.
4. **Phải có một quy tắc khi block bị vi phạm.** Ví dụ: nếu một block Deep Work bị họp chiếm, block
   đó được dời chứ không bị xoá, và bạn dời một việc khác ra khỏi tuần. Không có quy tắc này, block
   Deep Work trở thành biến điều chỉnh mặc định cho mọi phát sinh.

**Bước 3 — Thiết kế một tuần mẫu.**

Mẫu dưới đây cho vai Tech Lead ở công ty product Việt Nam có họp với khách hoặc đối tác nước ngoài
không thường xuyên. Đây là mẫu để sửa, không phải để sao chép:

```
        Thứ 2          Thứ 3          Thứ 4          Thứ 5          Thứ 6
09:00   Planning       DEEP WORK      DEEP WORK      DEEP WORK      DEEP WORK
10:00   /Alignment     (2h, đóng)     (2h, đóng)     (2h, đóng)     (2h, đóng)
11:00   Review batch   Review batch   Review batch   Review batch   Review batch
--------------------------------- nghỉ trưa ----------------------------------
13:30   1-1 (2 slot)   Collab/design  1-1 (2 slot)   Collab/design  Weekly review
15:00   Reactive       Reactive       Reactive       Reactive       Reactive
16:30   Admin          DEEP WORK      Admin          DEEP WORK      Admin / dọn
        (report)       (1.5h)                        (1.5h)         (đóng tuần)
```

Ba đặc điểm thiết kế cần chú ý: block Deep Work đặt vào buổi sáng của bốn ngày liên tiếp để tạo tính
liên tục của bối cảnh; review được batch vào một cửa sổ cố định 11h nên thời gian chờ review có
trần dự đoán được (tệ nhất là ~24 giờ, không phải "không biết bao lâu"); và thứ Hai được dành cho
alignment vì đó là ngày có nhiều thông tin thay đổi nhất, không nên đặt Deep Work vào ngày đó.

**Bước 4 — Weekly Calendar Review, 10 phút chiều thứ Sáu.**

Checklist:

- [ ] Tuần này có bao nhiêu block Deep Work bị chiếm? Bị chiếm bởi loại việc gì?
- [ ] Có buổi họp nào tôi thuộc loại "chỉ ngồi nghe"? Tuần tới tôi rút khỏi buổi nào?
- [ ] Có buổi họp nào lặp lại mà mục đích ban đầu đã hết hiệu lực?
- [ ] Số giờ Unaccounted tuần này? Nó rơi vào khoảng thời gian nào trong ngày?
- [ ] Tuần tới có họp nào rơi ngoài giờ làm việc (họp đêm với khách)? Tôi đã dịch chuyển gì để bù?
- [ ] Việc quan trọng nhất của quý có được đặt vào block Deep Work nào trong tuần tới không? Nếu
      không, tuần tới sẽ là tuần thứ mấy liên tiếp nó không được làm?

Câu cuối là câu có giá trị chẩn đoán cao nhất. Một việc quan trọng không được đặt vào lịch trong ba
tuần liên tiếp là một việc sẽ không xảy ra, bất kể nó được nhắc bao nhiêu lần trong các cuộc họp.

**Bước 5 — Xử lý bốn lực đặc thù của bối cảnh Việt Nam.**

**Lực 1 — Văn hoá "hỏi nhanh cái".** Ở phần lớn công ty Việt Nam, việc đi tới bàn đồng nghiệp và hỏi
trực tiếp là hành vi mặc định, được coi là thân thiện và hiệu quả. Nó thật sự hiệu quả — cho người
hỏi. Chi phí được chuyển hoàn toàn sang người bị hỏi và không hiển thị ở đâu.

Không thể xử lý lực này bằng cách từ chối, vì từ chối trực diện xung đột với chuẩn mực giữ hoà khí.
Cách xử lý có hiệu lực là **đổi kênh và đổi thời điểm, không đổi tính sẵn sàng**:

> **Đồng nghiệp:** Anh ơi hỏi nhanh cái, cái luồng refund này khi nào thì gọi sang service kế toán?
>
> **Bạn:** Câu này anh trả lời được nhưng cần mở lại code để nói chính xác. Em ghi vào channel team
> giúp anh nhé, 2h chiều anh trả lời một loạt. Còn nếu em đang bị chặn hoàn toàn không đi tiếp được
> thì nói luôn, anh dừng đây xử lý ngay.
>
> **Đồng nghiệp:** Không, em chưa làm đến đó, chiều cũng được.
>
> **Bạn:** Vậy em cứ ghi vào channel. Ghi vào đó có cái lợi là lần sau người khác hỏi thì tìm được.

Ba yếu tố làm script này hoạt động: bạn không nói không, bạn cho một đường thoát rõ ràng cho trường
hợp thật sự gấp (và tin vào câu trả lời của họ), và bạn nêu một lý do có lợi cho họ chứ không phải
lý do bảo vệ bạn. Sau khoảng 3–4 tuần áp dụng nhất quán, tỉ lệ câu hỏi đi vào channel tăng rõ, và
phần thưởng phụ là bạn có một tập câu hỏi lặp để viết tài liệu.

**Lực 2 — Slack/Zalo và kỳ vọng phản hồi tức thời.** Đặc thù Việt Nam: Zalo thường được dùng lẫn
giữa công việc và đời sống, không có ranh giới kênh, và có tín hiệu "đã xem" làm việc không trả lời
mang ý nghĩa xã hội. Đây là vấn đề khó hơn Slack.

Xử lý theo ba lớp: (a) **tách kênh theo mức khẩn cấp và công bố** — ví dụ Zalo chỉ dùng cho việc gấp
ngoài giờ và incident, mọi việc dự án đi qua Slack/Jira; (b) **tắt thông báo theo mặc định, bật theo
danh sách trắng** — chỉ giữ notification cho channel alert production và tin nhắn trực tiếp từ 3–4
người; (c) **trả lời có mẫu cho việc không gấp**: "Anh nhận được rồi, việc này anh xử lý trước 5h
chiều nay." Câu này đóng vòng lặp xã hội (người ta biết đã được nghe) mà không tiêu block của bạn.
Phần lớn kỳ vọng phản hồi tức thời thực chất là **kỳ vọng được xác nhận**, không phải kỳ vọng có câu
trả lời — và xác nhận rẻ hơn nhiều lần.

**Lực 3 — Họp đột xuất từ cấp trên.** Đây là lực bạn không kiểm soát được (vòng Accept hoặc
Influence, không phải Control). Bốn cách giảm thiệt hại, xếp theo mức khả thi:

- Không đặt Deep Work vào khung giờ mà theo dữ liệu lịch sử hay có họp đột xuất nhất (thường là đầu
  giờ chiều và sát cuối ngày). Nhường khung đó cho Reactive.
- Có một "block dự phòng" mỗi tuần để dời việc bị mất, và coi nó là bất khả xâm phạm.
- Đàm phán ở cấp cơ chế thay vì cấp từng lần: đề nghị cấp trên một cửa sổ cố định 30 phút mỗi ngày
  để hỏi mọi việc phát sinh. Với nhiều cấp trên, họ chấp nhận vì nó cũng làm việc của họ dễ hơn.
- Khi bị mất block, ghi lại. Sau bốn tuần, bạn có dữ liệu để nói: "Tháng này có 9 buổi họp phát sinh,
  chiếm 11 giờ, và đây là hai việc bị chậm vì thế." Dữ liệu đổi được cơ chế; lời phàn nàn thì không.

**Lực 4 — Múi giờ khách Mỹ/EU.** Với khách Mỹ bờ Tây, cửa sổ giao nhau với giờ làm việc Việt Nam gần
như chỉ có sáng sớm hoặc đêm. Với EU, khả thi hơn: chiều muộn Việt Nam. Ba nguyên tắc thực dụng:

1. **Gộp họp đêm vào ít ngày nhất có thể.** Hai đêm họp trong một tuần rẻ hơn bốn đêm mỗi đêm một
   buổi, vì mỗi đêm bị chiếm làm hỏng cả sáng hôm sau.
2. **Coi sáng sau đêm họp là tài nguyên đã bị tiêu.** Không đặt Deep Work vào sáng hôm sau; đặt Admin
   hoặc Reactive. Nếu tổ chức của bạn cho phép, dịch giờ vào muộn.
3. **Chuyển tối đa nội dung sang bất đồng bộ.** Với khách nước ngoài, một tài liệu viết tốt gửi trước
   thay được 60% thời lượng họp. Đây cũng là cách giảm áp lực tiếng Anh nói — bạn kiểm soát được chất
   lượng câu chữ khi viết. Kỹ thuật cụ thể ở `02-communication.md`.

### Trade-off

| Trục | Nghiêng về bên A khi | Nghiêng về bên B khi | Ai chịu phần mất |
|---|---|---|---|
| **A: Bảo vệ block Deep Work — B: Sẵn sàng phản hồi tức thời** | Bạn còn deliverable kỹ thuật thực chất; team đủ trưởng thành tự đi tiếp 2 giờ; việc của bạn cần giữ mô hình phức tạp | Đang có incident hoặc release; team có 2+ người mới; bạn là điểm nghẽn kiến thức duy nhất | Bảo vệ quá mức: người khác chờ, và bạn mất tín hiệu sớm về vấn đề. Mở quá mức: bạn không hoàn thành việc của mình, dần mất tư cách kỹ thuật |
| **A: Batch phản hồi — B: Phản hồi ngay** | Đa số câu hỏi không chặn ai; bạn đã công bố cửa sổ trả lời | Team làm theo luồng phụ thuộc chặt; hoặc bạn đang unblock người mới onboarding | Batch: latency trung bình tăng, người mới cảm thấy bị bỏ rơi nếu không giải thích. Ngay: throughput của bạn sụp |
| **A: Lịch dày, ít slack — B: Giữ 15–20% slack** | Giai đoạn ngắn có deadline cứng đã cam kết | Vận hành bình thường; hệ thống có nhiều phát sinh (fintech, legacy tích hợp nhiều đối tác) | Không có slack: mọi phát sinh đẩy trễ lan truyền, và bạn không có chỗ để suy nghĩ |
| **A: Tối ưu thời gian của mình — B: Tối ưu thời gian của team** | Bạn đang là điểm nghẽn về đầu ra kỹ thuật | Bạn đang là điểm nghẽn về quyết định — khi đó việc bạn "làm nhiều" khiến cả team chậm | Tối ưu sai đầu: một trong hai bên trả giá mà chỉ số của bên kia lại đẹp |

Trục thứ tư đáng dừng lại. Là IC, tối ưu thời gian của mình gần như đồng nghĩa với tối ưu đầu ra. Là
Tech Lead, hai thứ này tách ra và có thể ngược nhau: 2 giờ bạn dành để unblock ba người có giá trị
hệ thống cao hơn 2 giờ bạn viết code, dù chỉ số cá nhân của bạn xấu hơn. Nhận ra điểm chuyển này là
một trong những khác biệt lớn giữa một Senior giỏi và một Tech Lead hoạt động được. Xem thêm phần
Function Split ở `00-nen-tang-leadership.md`.

### Real-world Scenarios

**Tình huống A — Tech Lead mất hết Deep Work, và ba cách sửa với ba kết quả khác nhau.**

Bối cảnh: Linh, Tech Lead 6 người tại một công ty e-commerce, mùa chuẩn bị campaign. Trong 4 tuần,
Linh có 0 block ≥ 90 phút. Việc quan trọng nhất của quý — thiết kế lại luồng tính giá khuyến mãi để
chịu được peak — không tiến triển. Linh xử lý bằng cách làm buổi tối.

*Cách sửa thứ nhất (Linh đã thử, thất bại):* làm việc quan trọng từ 21h–23h ở nhà. Kết quả sau 3
tuần: bản thiết kế có, nhưng hai lần phải làm lại vì bỏ sót ràng buộc mà nếu tỉnh táo sẽ thấy; và
Linh bắt đầu đến muộn, làm cửa sổ Reactive buổi sáng biến mất, khiến team chờ nhiều hơn. Cơ chế thất
bại: dùng tài nguyên chất lượng thấp cho việc cần chất lượng cao, và mượn từ ngân sách năng lượng của
ngày mai.

*Cách sửa thứ hai (nửa hiệu quả):* Linh block 9h–11h mỗi ngày, không nói với ai, và từ chối họp. Kết
quả: hai tuần đầu tốt, tuần thứ ba PO phàn nàn với EM rằng "Linh khó liên lạc", và một quyết định về
tích hợp đối tác bị chậm 3 ngày vì không ai dám quyết thay. Cơ chế thất bại: đóng cửa mà không công
bố cửa mở lúc nào — thiếu điều kiện phòng thủ số 3.

*Cách sửa thứ ba (hoạt động):* Linh làm ba việc cùng lúc. (1) Công bố trong stand-up và pin lên
channel: "9h–11h thứ Ba/Tư/Năm tôi làm thiết kế giá, không nhận họp trừ incident; 11h–11h45 tôi
review PR; 15h–16h30 tôi trả lời mọi câu hỏi và tham gia mọi thảo luận." (2) Uỷ quyền tường minh:
"Nam quyết được mọi thứ liên quan tới luồng checkout dưới mức đổi contract API; nếu phải đổi contract
thì chờ tôi." (3) Nói trước với PO: "Trong 3 tuần tới tôi ưu tiên thiết kế giá vì nếu không có nó thì
campaign tháng 11 có rủi ro. Đổi lại, các câu hỏi của chị tôi trả lời trong ngày, không phải trong
giờ."

Kết quả sau 3 tuần: thiết kế hoàn thành, thời gian chờ review giảm từ p90 hai ngày xuống dưới một
ngày (nghịch lý biểu kiến: batch review làm giảm p90 vì trước đó review bị bỏ dở nhiều lần), và PO
không phàn nàn vì kỳ vọng đã được đặt lại trước.

Bài học truy được về cơ chế: bảo vệ thời gian là một hành động **giao tiếp và uỷ quyền**, không phải
một hành động về lịch. Ai chỉ sửa lịch mà không sửa hai thứ kia sẽ lặp lại cách thứ hai.

**Tình huống B — "Hỏi nhanh cái" từ một người lớn tuổi hơn.**

Bối cảnh: Trang, 29 tuổi, Tech Lead. Anh Hùng, 41 tuổi, Senior BE trong team, vào công ty trước
Trang 4 năm. Anh Hùng có thói quen sang bàn Trang hỏi trực tiếp 5–8 lần mỗi ngày, nhiều câu là bàn
luận mở không có quyết định cần ra.

Đây là tình huống mà công cụ thuần kỹ thuật (đóng lịch, đặt trạng thái busy) không giải quyết được,
vì lực đang tác động là chuẩn mực xã hội về tuổi và thâm niên. Cách xử lý sai phổ biến: Trang chịu
đựng ba tháng rồi một hôm phản ứng gay gắt — làm tổn hại quan hệ và vẫn không đổi được hành vi.

Cách xử lý có xác suất thành công cao hơn: chuyển từ đối thoại về **hành vi của anh Hùng** sang đối
thoại về **cách làm việc của cả hai**, trong một buổi 1-1 riêng, và đặt Trang vào vị trí người cần
giúp:

> **Trang:** Anh Hùng, em muốn nhờ anh một việc về cách hai anh em phối hợp. Em đang có vấn đề là em
> không giữ được mạch khi làm phần thiết kế, và em nhận là do em sắp xếp chưa tốt.
>
> **Anh Hùng:** Ừ, dạo này thấy em cũng căng.
>
> **Trang:** Em định thử cách này, muốn xin ý anh. Buổi sáng 9h–11h em tập trung làm thiết kế, em
> tắt Slack. Còn 14h em để trống hẳn một tiếng, anh em mình ngồi với nhau, có gì bàn hết trong đó —
> em nghĩ mấy chuyện về kiến trúc bàn liền một mạch còn hiệu quả hơn hỏi rời rạc. Anh thấy khung
> giờ đó có ổn với anh không, hay anh muốn giờ khác?
>
> **Anh Hùng:** 14h thì được, nhưng có việc gấp thì sao?
>
> **Trang:** Gấp thì anh cứ gọi điện, em nghe ngay. Em chỉ muốn tách cái gấp với cái bàn cho rõ ra.
> Với cả có một việc em muốn nhờ anh thật: mấy câu junior hỏi em suốt về luồng payment, anh trả lời
> chắc còn tốt hơn em vì anh làm từ đầu. Anh nhận giúp em phần đó được không?

Ba kỹ thuật ở đây: Trang đặt vấn đề như vấn đề của mình (giữ mặt cho người đối diện — quan trọng
trong bối cảnh Việt Nam), cho anh Hùng quyền chọn khung giờ (biến quyết định áp đặt thành quyết định
chung), và trao một vai có giá trị (trả lời junior về payment) để việc giảm tiếp xúc không bị đọc là
xa cách. Chi tiết hơn về kỹ thuật loại này ở `02-communication.md`, và về khoảng cách tuổi/thâm niên
trong uy tín kỹ thuật ở `03-team-leadership.md`.

### Best Practices

**Đặt block Deep Work vào cùng khung giờ, cùng ngày, hàng tuần.** Lý do: kỳ vọng của người khác hình
thành qua lặp lại. Một block xuất hiện ở chỗ khác nhau mỗi tuần không tạo được kỳ vọng, nên luôn phải
bảo vệ bằng nỗ lực ý chí từng lần — thứ sẽ hết vào tuần thứ ba.

**Luôn công bố cửa sổ Reactive cùng lúc với việc công bố block đóng.** Lý do: chi phí xã hội của việc
không sẵn sàng đến từ sự bất định, không từ độ trễ. Người ta chịu được việc chờ đến 14h; họ không
chịu được việc không biết có được trả lời không.

**Trước khi vào một block Deep Work, viết ra một câu: kết thúc block này tôi sẽ có gì.** Lý do: giảm
thời gian nạp bối cảnh ở đầu block và cho một tiêu chí để biết block có hiệu quả hay không. Không có
câu này, block dễ bị trôi thành đọc tài liệu.

**Khi bị cắt giữa việc khó, dành 60 giây ghi lại trạng thái đang giữ trong đầu.** Lý do: phần đắt
nhất của gián đoạn là dựng lại mô hình. Ghi ba dòng — đang kiểm tra giả thuyết gì, đã loại được gì,
bước kế tiếp là gì — giảm chi phí tái nạp đáng kể và giảm attention residue vì việc đã được "đóng
tạm" thay vì để mở.

**Giữ 15–20% calendar trống có chủ đích.** Lý do: xem mô hình Queueing. Lịch kín 100% không phải dấu
hiệu của người hiệu quả mà là hệ thống không có khả năng hấp thụ biến động — và trong môi trường có
phát sinh cao (fintech, legacy, nhiều đối tác), biến động là hằng số.

**Với mỗi cuộc họp định kỳ bạn tạo ra, đặt ngày hết hạn.** Lý do: họp định kỳ tích tụ theo thời gian
và không có cơ chế tự loại bỏ. Ngày hết hạn buộc phải xác nhận lại giá trị thay vì mặc định tồn tại.

### Anti-patterns

**Dùng "làm nhiều giờ" như giải pháp cho vấn đề cấu trúc lịch.**
Cơ chế: giờ thêm vào nằm ở cuối ngày, là giờ có chất lượng attention thấp nhất, và nó vay từ ngân
sách phục hồi của ngày mai. Sau 2–3 tuần, tổng đầu ra giảm dù tổng giờ tăng — và vì tổng giờ tăng,
bạn kết luận sai rằng mình cần thêm giờ nữa.
Dấu hiệu sớm: việc khó nhất trong tuần luôn được làm sau 20h; số lỗi phát hiện trong review tăng ở
các PR bạn viết buổi tối; bạn bắt đầu đến muộn và cửa sổ phản hồi buổi sáng biến mất.

**Calendar kín 100% và coi đó là bằng chứng của sự quan trọng.**
Cơ chế: không còn slack để suy nghĩ hoặc hấp thụ phát sinh, nên mọi việc mới đều đẩy một việc cũ ra
mà không qua quyết định có ý thức. Việc bị đẩy ra thường là việc quan trọng-không-gấp, vì nó không
có ai đòi.
Dấu hiệu sớm: bạn quyết định mọi thứ trong 5 phút cuối buổi họp; bạn không nhớ lần cuối mình suy nghĩ
về một vấn đề mà không có deadline.

**Interrupt-driven hoàn toàn, và gọi đó là "support team tốt".**
Cơ chế: bạn tối ưu latency cho người khác bằng cách đưa throughput của mình về gần 0 cho việc cần tập
trung. Về ngắn hạn team hài lòng; về trung hạn team mất người duy nhất có thời gian nghĩ về thiết kế,
và các quyết định kiến trúc bắt đầu được ra một cách tình cờ trong các câu trả lời 5 phút.
Dấu hiệu sớm: bạn trả lời dưới 2 phút cho gần như mọi tin nhắn; deliverable của bạn luôn là việc nhỏ;
team không có tài liệu vì mọi kiến thức đi qua bạn theo kênh khẩu ngữ.

**Time blocking không có cơ chế phòng thủ.**
Cơ chế: block bị chiếm 3 lần liên tiếp thì nó mất giá trị tín hiệu với cả bạn và người khác, và bạn
kết luận sai rằng "time blocking không phù hợp với công việc của tôi". Nguyên nhân thật là thiếu công
bố, thiếu cửa sổ Reactive tương ứng, và thiếu quy tắc khi bị vi phạm.
Dấu hiệu sớm: block trên calendar có tên chung chung; bạn tự nhận họp vào block của mình.

**Đo hiệu quả bằng số ticket đóng hoặc số message trả lời.**
Cơ chế: hai chỉ số này thiên vị mạnh cho việc nhỏ và việc phản ứng. Tối ưu theo chúng dẫn tới việc
bạn hệ thống hoá việc tránh các bài toán lớn — vì bài toán lớn không tạo ra chuyển động nhìn thấy
trong ngày.
Dấu hiệu sớm: cuối tuần bạn kể được 30 việc đã làm nhưng không kể được một vấn đề đã được giải quyết
dứt điểm.

### Khi nào KHÔNG nên áp dụng

**Trong incident production hoặc trong cửa sổ release.** Trong hai giai đoạn này, chế độ đúng là
interrupt-driven, và mọi block phải bị phá không do dự. Lý do có cơ chế: chi phí độ trễ trong incident
tăng theo thời gian và không đảo được, còn chi phí gián đoạn công việc thiết kế thì đảo được. Cố giữ
block Deep Work trong lúc production đang lỗi là áp dụng đúng công cụ vào sai điều kiện. Chi tiết cách
vận hành ở `06-incident-va-metrics.md`.

**Trong 2–4 tuần đầu của một người mới trong team, hoặc 2–4 tuần đầu của bạn ở công ty mới.** Onboarding
là giai đoạn mà giá trị của việc sẵn sàng cao hơn giá trị của việc tập trung. Với người mới, mỗi lần
bạn trả lời chậm 4 giờ có thể làm họ mất nửa ngày và học được rằng hỏi thì bất lợi. Trong giai đoạn
này, hãy giảm chỉ tiêu Deep Work của mình một cách có ý thức và ghi nó vào kế hoạch, thay vì cố giữ
cả hai rồi thất bại ở cả hai.

**Khi bạn đang ở vai Engineering Manager với 8+ report.** Ở vai này, phần lớn giá trị đầu ra thật sự
nằm ở tương tác, và việc cố giữ 40% Deep Work là chẩn đoán sai về công việc. Nếu bạn ở vai này mà
vẫn khao khát block dài để viết code, đó là dữ liệu về việc bạn thuộc nhánh nào — vấn đề của
`11-career-evolution.md`, không phải vấn đề của lịch.

**Khi tổ chức chưa có bất kỳ hình thức bất đồng bộ nào.** Nếu team chưa dùng ticket, chưa viết tài
liệu, và mọi thông tin đi bằng miệng, thì việc bạn đơn phương chuyển sang schedule-driven sẽ làm
thông tin bị mất chứ không được xếp vào hàng đợi — vì không có hàng đợi nào tồn tại. Trong điều kiện
này, việc phải làm trước là dựng kênh bất đồng bộ (channel có chuẩn, ticket có template), rồi mới bảo
vệ thời gian. Làm ngược thứ tự sẽ tạo ra một lead khó tiếp cận trong một tổ chức không có bộ nhớ.

**Với công việc mà giá trị nằm ở tần suất tiếp xúc, không ở độ sâu.** Ví dụ: giai đoạn bạn đang cố
xây Trust với một team mới nhận, hoặc giai đoạn đang có xung đột giữa hai thành viên. Trong các giai
đoạn này, việc "có mặt" là công việc chính, không phải chi phí của công việc chính.

---

## 3. Prioritization

### Problem Statement

Một công ty logistics ở TP.HCM, team 9 engineer, backlog sprint có 23 hạng mục. Nhãn ưu tiên trong
Jira: 19 hạng mục P0, 3 hạng mục P1, 1 hạng mục P2. Người tạo nhãn: 4 người khác nhau (PO, Head of
Ops, CTO, và một Account Manager). Không ai trong số họ nói dối — với mỗi người, việc họ đánh P0 thật
sự là việc quan trọng nhất trong phạm vi họ nhìn thấy.

Kết quả sau sprint: 11 hạng mục xong, trong đó 3 hạng mục không ai dùng trong tháng đó; hai việc
quan trọng thật sự (một là lỗi tính phí giao hàng làm sai hoá đơn của 200 đơn/ngày, một là chuẩn bị
capacity cho campaign) đều không xong; và trong retro, kết luận là "team cần cải thiện estimation".

Đây không phải vấn đề estimation. Đây là vấn đề **hệ thống không có hàm ưu tiên**, và khi không có
hàm, thứ tự thực thi được quyết định bởi những lực khác: ai nhắc gần nhất, ai có giọng to nhất, việc
nào dễ bắt đầu nhất, và việc nào người dev thấy thú vị nhất.

Hiện tượng quan sát được khi Prioritization yếu:

- **Phân phối nhãn ưu tiên bị dồn về một đầu.** Nếu hơn 60% hạng mục là P0, hệ thống nhãn đã mất
  hoàn toàn giá trị thông tin. Về mặt lý thuyết thông tin, một nhãn chỉ mang thông tin khi nó phân
  biệt được — nhãn mà ai cũng có thì tương đương không có nhãn.
- **Số lần đổi thứ tự trong một sprint.** Đo bằng số lần một hạng mục đang in-progress bị dừng để làm
  việc khác. Ở dự án trên: 14 lần trong 2 tuần.
- **Tỉ lệ WIP so với số người.** Khi WIP > số người, mọi người đang làm nhiều việc cùng lúc, tức là
  ưu tiên đang được giải quyết bằng cách không ưu tiên.
- **Thời gian một việc "quan trọng nhưng không ai đòi" tồn tại trong backlog.** Ví dụ điển hình: nâng
  cấp thư viện có CVE, hoặc dựng lại luồng backup. Nếu con số này vượt 2 quý, tổ chức đang chỉ phản
  ứng với áp lực, không ưu tiên theo giá trị.
- **Chi phí của việc trễ chưa từng được nói bằng con số.** Nếu không ai trong team trả lời được "trễ
  việc này một tuần thì mất gì", thì mọi cuộc tranh luận ưu tiên sẽ được quyết bằng thứ bậc, không
  bằng lập luận.

Ở cấp cá nhân, biểu hiện là: bạn kết thúc tuần với cảm giác đã làm rất nhiều nhưng không việc nào
trong ba việc quan trọng nhất tiến triển. Đây là dấu hiệu của việc bạn đang tối ưu theo **tính cấp
bách được cảm nhận**, không theo giá trị kỳ vọng.

### First Principles

**Nguyên lý một: prioritization là một hàm, và hàm đó có ba biến.** Dạng tổng quát nhất, có ích khi
không có dữ liệu tốt:

```
Ưu tiên ≈  (Giá trị kỳ vọng × Xác suất thành công)
          ---------------------------------------
             (Chi phí thực hiện × Độ trễ chấp nhận được)
```

Ba điều rút ra từ dạng này. Thứ nhất, **giá trị phải nhân với xác suất** — một việc có giá trị rất cao
nhưng xác suất thành công 20% có thể xếp sau một việc giá trị trung bình gần như chắc chắn xong. Đây
là lý do các "big bet" cần được tách nhỏ thành bước có thể kiểm chứng, để tăng xác suất thay vì tăng
giá trị hứa hẹn. Thứ hai, **chi phí ở mẫu số nghĩa là việc nhỏ được lợi thế cấu trúc** — không phải
vì việc nhỏ quan trọng hơn, mà vì nó cho phép học sớm và giải phóng năng lực. Thứ ba, **độ trễ chấp
nhận được là biến hay bị bỏ**: hai việc có cùng giá trị và cùng chi phí nhưng một việc mất giá trị
nếu làm muộn (chuẩn bị cho campaign có ngày cố định) và một việc thì không (refactor), phải xếp khác
nhau.

**Nguyên lý hai: Cost of Delay.** Khái niệm này (từ dòng lý thuyết Lean/Flow, nguồn công khai
Donald Reinertsen) đảo ngược câu hỏi: thay vì hỏi "việc này có giá trị bao nhiêu", hỏi "**mỗi tuần
chậm việc này, tổ chức mất bao nhiêu**". Câu hỏi thứ hai dễ trả lời hơn nhiều và cho ra lập luận
thuyết phục hơn với người ngoài kỹ thuật.

Bốn dạng đường Cost of Delay cần phân biệt, vì chúng dẫn tới chiến lược khác nhau:

```
1. Tuyến tính:      giá trị mất tăng đều theo thời gian
   Ví dụ: lỗi tính phí làm sai 200 hoá đơn/ngày → mất 200 đơn công việc sửa tay mỗi ngày.
   Chiến lược: làm sớm, nhưng có thể chia nhỏ.

2. Deadline cứng:   mất gần như bằng 0 cho tới ngày X, rồi mất rất lớn
   Ví dụ: chuẩn bị capacity cho campaign 11/11; tuân thủ một quy định có ngày hiệu lực.
   Chiến lược: lập kế hoạch ngược từ ngày X, có buffer, không được để trôi.

3. Mất cơ hội:      giá trị giảm dần và không quay lại
   Ví dụ: ra tính năng trước đối thủ; tận dụng một đối tác đang muốn hợp tác.
   Chiến lược: ưu tiên tốc độ hơn độ hoàn thiện.

4. Tăng theo hàm mũ: mỗi tuần chậm làm chi phí sửa tăng nhân
   Ví dụ: technical debt lan ra thêm code phụ thuộc; dữ liệu sai tiếp tục sinh ra dữ liệu sai.
   Chiến lược: chặn nguồn trước, sửa hậu quả sau.
```

Nhận diện dạng đường là bước có giá trị cao nhất, và nó thường bị bỏ qua vì mọi việc đều được nói
bằng cùng một từ: "gấp".

**Nguyên lý ba: Theory of Constraints — chỉ có một điểm nghẽn quan trọng tại một thời điểm.** Nếu hệ
thống có một constraint (Eliyahu Goldratt, nguồn công khai), thì mọi cải thiện ở nơi không phải
constraint đều **không làm tăng throughput của hệ thống**. Áp vào prioritization cá nhân: trước khi
xếp thứ tự cho 20 việc, hỏi "throughput của hệ thống tôi đang thuộc về bị chặn ở đâu?" Nếu team bị
chặn ở review, thì việc bạn viết thêm code không tăng đầu ra — nó chỉ tăng WIP. Nếu team bị chặn ở
QA, việc bạn tối ưu CI không giúp gì. Đây là lý do một danh sách ưu tiên tốt ở cấp cá nhân phải bắt
đầu từ một chẩn đoán ở cấp hệ thống.

**Nguyên lý bốn: chi phí của việc ước lượng là chi phí thật.** Mọi framework prioritization đều cần
input. RICE cần bốn số cho mỗi hạng mục. Nếu backlog có 40 hạng mục và mỗi hạng mục cần 20 phút thảo
luận để cho ra bốn số, chi phí là hơn 13 giờ người — và độ chính xác của các số đó thường không đủ để
đổi thứ tự. Đây là một bài toán tối ưu bậc hai: **chọn framework có độ chính xác vừa đủ để ra được
quyết định khác đi.** Nếu một framework tinh vi cho ra cùng thứ tự với một framework thô, framework
tinh vi đó là chi phí thuần.

**Nguyên lý năm: prioritization là quyết định về việc KHÔNG làm.** Một danh sách đã sắp thứ tự mà
không có đường kẻ "dưới đây không làm trong quý này" thì chưa phải là ưu tiên — nó là một danh sách
mong muốn đã sắp xếp. Cơ chế: khi mọi việc vẫn còn trên bàn, mỗi việc vẫn tiêu attention (xem hiệu
ứng Zeigarnik ở mục 4) và vẫn tạo ra kỳ vọng cho người đề xuất nó.

### Mental Model

**Mô hình 1 — Đường kẻ cắt (the cut line).**

Vẽ danh sách theo thứ tự, rồi kẻ một đường ngang tại chỗ capacity hết. Thay đổi nhận thức quan trọng:
việc quan trọng nhất bạn phải truyền đạt không phải là *thứ hạng*, mà là *đường kẻ nằm ở đâu*. Với
stakeholder, câu "việc của chị đang ở vị trí thứ 7" không tạo ra phản ứng gì; câu "việc của chị nằm
dưới đường kẻ, nghĩa là quý này không làm, trừ khi chị muốn đổi với việc số 3" tạo ra một cuộc đối
thoại về trade-off thật.

**Mô hình 2 — Ma trận Cost of Delay × Duration (nền tảng của WSJF).**

```
        Cost of Delay cao
               ^
               |   [ Làm ngay,      ] | [ Làm ngay,       ]
               |   [ dù dài          ] | [ ưu tiên số 1    ]
               |   [ (chia nhỏ nếu   ] | [                 ]
               |   [  có thể)        ] | [                 ]
               |---------------------+-------------------->
               |   [ Làm sau /        ] | [ Làm khi có khe  ]
               |   [ có thể bỏ        ] | [ trống — rẻ và   ]
               |   [                  ] | [ dọn được nợ     ]
        Cost of Delay thấp             Duration ngắn
                    Duration dài  <----
```

Mô hình này giải thích vì sao xếp theo "giá trị" đơn thuần cho ra thứ tự tệ: hai việc cùng giá trị,
việc nào ngắn hơn phải làm trước, vì làm việc ngắn trước giảm tổng thời gian chờ của cả hệ thống.
Đây là kết quả quen thuộc trong scheduling theory (Shortest Job First) và là gốc của WSJF.

**Mô hình 3 — Ba tầng ưu tiên phải khớp nhau.**

```
Tầng tổ chức:  Business Goal quý này (1–3 mục tiêu)
                        ↓ phải truy được
Tầng team:      Sprint goal / roadmap item
                        ↓ phải truy được
Tầng cá nhân:   Ba việc quan trọng nhất tuần này của bạn
```

Kiểm tra rất nhanh và rất tàn nhẫn: lấy ba việc bạn dành nhiều thời gian nhất tuần qua, và thử truy
mỗi việc lên tới Business Goal của quý. Việc nào không truy được, hoặc là việc duy trì bắt buộc
(vận hành, compliance), hoặc là việc bạn đang làm vì lý do khác (thú vị, dễ, hoặc có người gào to).
Tỉ lệ không truy được ở mức 30–40% là bình thường trong tổ chức thật; ở mức 70% là dấu hiệu bạn đang
làm việc cho một hệ thống ưu tiên khác với hệ thống bạn nghĩ mình đang phục vụ.

### Practical Framework

**Bước 1 — Chọn framework theo điều kiện dữ liệu, không theo mức độ tinh vi.**

| Framework | Cơ chế xếp hạng | Dữ liệu cần có | Phù hợp nhất với | Thất bại khi |
|---|---|---|---|---|
| **Eisenhower** (Urgent × Important) | Chia 4 ô, hành động khác nhau mỗi ô | Không cần số, chỉ cần phán đoán | Ưu tiên **công việc cá nhân trong ngày/tuần**; tách việc gấp giả khỏi việc quan trọng | Dùng cho backlog sản phẩm — "important" không có định nghĩa chung nên mỗi người điền khác nhau |
| **Impact / Effort** | Ma trận 2×2, chọn ô Impact cao/Effort thấp trước | Hai ước lượng thô (cao/trung/thấp) | Quyết định nhanh trong 15 phút với một nhóm nhỏ; sàng lọc sơ bộ 20+ hạng mục | Các hạng mục có impact khác loại (doanh thu vs rủi ro compliance) — không so được trên cùng trục |
| **RICE** (Reach × Impact × Confidence / Effort) | Cho điểm số, xếp theo điểm | Bốn ước lượng, trong đó Reach cần dữ liệu người dùng thật | Team product có analytics; so sánh nhiều tính năng cùng loại | Không có dữ liệu Reach → Reach bị đoán, và điểm số tạo ra vẻ chính xác giả (false precision) |
| **WSJF** (Cost of Delay / Job Duration) | Xếp theo tỉ lệ | Cost of Delay ước lượng được (có thể bằng thang tương đối), và duration | Bài toán có yếu tố thời gian mạnh: campaign, compliance có deadline, nợ đang lan | Team không phân biệt được 4 dạng đường Cost of Delay → mọi việc đều được cho CoD cao |
| **MoSCoW** (Must/Should/Could/Won't) | Phân nhóm, không xếp thứ tự trong nhóm | Chỉ cần thoả thuận về scope | **Đàm phán scope với khách hoặc stakeholder**, đặc biệt trong ODC có hợp đồng | Dùng làm công cụ ưu tiên nội bộ — vì nhóm Must luôn phình ra và không có cơ chế chống lại |

Quy tắc chọn thực dụng: **dữ liệu bạn có quyết định framework, không phải framework quyết định dữ liệu
bạn cần đi thu thập.** Nếu bạn không có analytics, đừng dùng RICE — dùng Impact/Effort và ghi rõ đó
là phán đoán. Một quyết định thô được ghi lại minh bạch tốt hơn một điểm số 8.4 mà không ai truy được
nguồn.

Kết hợp thường dùng và có hiệu lực trong thực tế Việt Nam: **MoSCoW để đàm phán biên với khách/PO →
WSJF hoặc Impact/Effort để xếp thứ tự bên trong nhóm Must và Should → Eisenhower để bạn tự sắp lịch
tuần của mình.** Ba tầng, ba công cụ, vì ba bài toán khác nhau.

**Bước 2 — Quy trình xếp ưu tiên 45 phút cho một backlog 20–40 hạng mục.**

Input: danh sách hạng mục, capacity ước lượng của kỳ, và một câu phát biểu Business Goal của quý.

1. **(5 phút) Tách nhóm bắt buộc ra trước.** Vận hành, compliance, incident follow-up, và các cam kết
   đã ký. Nhóm này không tham gia xếp hạng — nó là chi phí sàn. Tính capacity còn lại.
2. **(10 phút) Gán dạng đường Cost of Delay cho từng hạng mục** (một trong bốn dạng ở trên, hoặc
   "không mất gì nếu chậm một quý"). Đây là bước có tỉ lệ giá trị/chi phí cao nhất trong toàn quy
   trình.
3. **(10 phút) Ước lượng duration theo thang thô** — nửa ngày / 1–3 ngày / 1 tuần / hơn 1 tuần. Không
   ước lượng chi tiết ở bước này; độ chính xác thêm không đổi được thứ tự.
4. **(10 phút) Xếp theo tỉ lệ CoD/duration.** Với các hạng mục ngang nhau, phá vỡ thế cân bằng bằng
   câu hỏi: việc nào giảm được nhiều bất định nhất?
5. **(5 phút) Kẻ đường cắt tại chỗ capacity hết.** Viết ra tường minh: những hạng mục dưới đường
   này **không làm trong kỳ này**.
6. **(5 phút) Viết ba câu để truyền đạt:** ba việc đầu là gì và vì sao; đường cắt ở đâu; điều kiện
   nào sẽ làm thứ tự này thay đổi.

Tiêu chí biết là xong: bạn nói được cho một stakeholder bất kỳ *vì sao việc của họ ở vị trí đó* mà
không dùng từ "chưa có thời gian".

**Bước 3 — Ba việc quan trọng nhất tuần, và một quy tắc chống loãng.**

Ở cấp cá nhân, mỗi thứ Hai viết ra ba việc mà nếu tuần này chỉ xong ba việc đó thì tuần vẫn được coi
là thành công. Quy tắc chống loãng: **mỗi việc phải có một block cụ thể trên calendar trước khi được
coi là đã ưu tiên.** Một việc được gọi là ưu tiên nhưng không có chỗ trên lịch chưa được ưu tiên —
nó chỉ được mong muốn. Đây là mối nối trực tiếp giữa mục 2 và mục 3 của chương này: prioritization
không có time blocking là danh sách; time blocking không có prioritization là kỷ luật không có hướng.

**Bước 4 — Script từ chối và đàm phán khi mọi thứ là P0.**

Tình huống: PO đến với việc thứ tư, gọi là P0, trong khi sprint đã cam kết ba việc.

> **PO:** Anh ơi cái báo cáo cho bên Ops mình phải làm trong sprint này, P0 nhé, chị Head of Ops
> đang cần.
>
> **Tech Lead:** Em hiểu là nó gấp. Em cần chị giúp em một việc: sprint này team đang có ba việc đã
> cam kết — sửa lỗi tính phí giao hàng, chuẩn bị capacity cho campaign, và tích hợp API đối tác vận
> chuyển. Capacity đã kín. Nếu thêm báo cáo Ops, một trong ba việc kia phải ra. Chị chọn giúp em ra
> việc nào?
>
> **PO:** Không, ba việc kia cũng quan trọng mà, cái nào cũng P0.
>
> **Tech Lead:** Vậy em đề nghị mình quyết bằng cái này thay vì bằng nhãn. Em hỏi ba câu cho từng
> việc: chậm một tuần thì mất gì, mất cho ai, và có cách nào tạm thời không cần code không. Với lỗi
> tính phí: mỗi ngày 200 hoá đơn sai, Ops phải sửa tay, khách gọi complain — mất tăng đều mỗi ngày.
> Với capacity campaign: từ giờ đến 11/11 thì không mất gì, nhưng sau ngày đó nếu sập thì mất doanh
> thu cả đợt. Với báo cáo Ops: chị cho em biết chậm một tuần thì chị Head of Ops không làm được việc
> gì?
>
> **PO:** Chị ấy cần số để họp với ban giám đốc thứ Sáu tuần sau.
>
> **Tech Lead:** Vậy nhu cầu thật là có số trước thứ Sáu tuần sau, không phải có một báo cáo trong
> hệ thống. Em có thể chạy query và xuất Excel cho chị ấy trong 2 giờ, tuần này. Báo cáo tự động em
> đưa vào sprint sau. Nếu chị đồng ý, em không phải bỏ việc nào trong ba việc kia.
>
> **PO:** Được, làm thế đi.

Bốn kỹ thuật cần thấy trong script này: (1) không tranh luận về nhãn P0 — chuyển sang tranh luận về
capacity, thứ có giới hạn khách quan; (2) buộc người đề xuất tham gia vào quyết định trade-off thay
vì để mình một mình chịu; (3) hỏi Cost of Delay bằng câu hỏi cụ thể "chậm một tuần thì ai không làm
được gì" — câu này gần như luôn lộ ra rằng nhu cầu thật khác với yêu cầu được phát ra; (4) tìm giải
pháp có chi phí nhỏ hơn cho nhu cầu thật. Điểm quan trọng nhất là kỹ thuật (3): phần lớn yêu cầu
"P0" trong thực tế là một **giải pháp** được phát ra thay cho một **nhu cầu**, và khi tách được nhu
cầu ra, thường có đường rẻ hơn nhiều.

### Trade-off

| Trục | Nghiêng về bên A khi | Nghiêng về bên B khi | Ai chịu phần mất |
|---|---|---|---|
| **A: Ước lượng chính xác — B: Ước lượng thô, quyết nhanh** | Quyết định khó đảo (chọn nền tảng, thuê thêm người, cam kết với khách); chi phí sai lớn hơn nhiều chi phí đo | Quyết định đảo được trong 1 sprint; hoặc số hạng mục lớn và độ chính xác thêm không đổi thứ tự | Chính xác quá: chi phí ước lượng ăn vào capacity thực thi, và độ chính xác giả tạo ra sự tự tin sai |
| **A: Tuân thủ thứ tự đã chốt — B: Linh hoạt theo tín hiệu mới** | Tổ chức đang bị đổi ưu tiên quá nhiều; team mất khả năng hoàn thành việc gì | Có tín hiệu thị trường/khách hàng thật, kiểm chứng được, khác hẳn giả định ban đầu | Cứng quá: làm xong việc không còn cần thiết. Lỏng quá: WIP tăng, không việc nào xong |
| **A: Việc có giá trị đo được — B: Việc giảm rủi ro / dọn nợ** | Đang thiếu doanh thu hoặc cần chứng minh giá trị để có vốn (startup trước vòng gọi) | Chỉ số vận hành đang xấu dần: change failure rate tăng, lead time tăng, incident lặp | Nghiêng A dài hạn: chi phí thay đổi tăng dần cho tới khi mọi tính năng đều đắt gấp ba |
| **A: Tối ưu ưu tiên của team mình — B: Tối ưu throughput toàn hệ** | Team bạn đang là constraint của hệ thống | Constraint nằm ở team khác — khi đó việc bạn làm nhanh chỉ tạo tồn kho chờ | Nghiêng A khi không phải constraint: chỉ số team đẹp, giá trị giao tới khách không tăng |

Trục đầu tiên đáng nói kỹ, vì đây là chỗ nhiều team đốt nhiều thời gian nhất. Chi phí của việc ước
lượng không chỉ là số giờ họp — nó còn là chi phí cơ hội của attention và chi phí tâm lý của việc
tranh luận về những con số mà không ai có cơ sở. Một quy tắc thực dụng: **chỉ tăng độ chính xác của
ước lượng khi bạn xác định được rằng số chính xác hơn có khả năng đổi thứ tự.** Nếu việc A và việc B
sẽ vẫn theo thứ tự đó dù ước lượng lệch 50%, việc đo thêm là chi phí thuần.

### Real-world Scenarios

**Tình huống A — "Mọi thứ đều P0" nhìn từ ba tầng.**

Bối cảnh: công ty e-commerce, ba tuần trước campaign 11/11. Backlog có 5 việc, capacity đủ cho 3.
Các việc: (1) sửa lỗi tính phí giao hàng, (2) load test và tăng capacity cho peak, (3) tính năng
voucher mới mà Marketing đã hứa trong kế hoạch truyền thông, (4) nâng cấp thư viện có CVE mức cao,
(5) dashboard cho Ops.

*Nhìn từ IC (Nam, Senior BE):* Nam thấy backlog có 5 việc P0 và cảm thấy bất lực. Sai lầm thường
thấy ở tầng này là chờ người khác quyết, rồi làm việc dễ bắt đầu nhất. Hành động đúng ở tầng IC
không phải là tự quyết ưu tiên của cả team — Nam không có đủ thông tin về doanh thu và cam kết truyền
thông. Hành động đúng là **cung cấp input mà chỉ Nam có**: viết ba dòng cho mỗi việc về chi phí kỹ
thuật và rủi ro nếu chậm, đặc biệt là hai thứ mà người ngoài không thấy được — rằng việc (2) nếu để
sau thì không còn thời gian sửa những gì load test phát hiện, và rằng việc (4) có thể để sang tháng
12 vì CVE đó chỉ khai thác được qua một đường mà hệ thống hiện không mở. Nam cũng nên nêu rõ điều
mình cần: "em cần biết trong 5 việc này, việc nào em được phép làm dở."

*Nhìn từ Tech Lead (Hà):* Hà chịu hệ quả cho việc hệ thống chạy được qua peak, và có đủ thông tin kỹ
thuật để gán dạng Cost of Delay. Hà đọc backlog theo bốn dạng đường: (2) là deadline cứng với hậu quả
rất lớn và không có cách bù sau; (1) là tuyến tính, mất mỗi ngày, nhưng có cách giảm thiệt hại tạm
thời bằng script đối chiếu; (3) là mất cơ hội nhưng có thể thu hẹp scope; (4) là tuyến tính rất
thoải, có thể hoãn một quý mà không đổi rủi ro thực tế; (5) không mất gì nếu chậm, và nhu cầu thật
có thể đáp ứng bằng query tay.

Quyết định của Hà: (2) làm ngay và làm đủ, (1) làm bản sửa tối thiểu cộng script bù dữ liệu, (3) đàm
phán thu hẹp còn một loại voucher, (4) và (5) đưa xuống dưới đường cắt kèm lý do viết ra. Sai lầm mà
Hà cần tránh: tự quyết việc (3) mà không nói với Marketing — đây là việc có cam kết bên ngoài, và
việc thu hẹp scope phải là quyết định chung, nếu không Hà sẽ đúng về kỹ thuật và sai về tổ chức.

*Nhìn từ Manager (EM/CTO):* ở tầng này, việc backlog có 5 P0 là **triệu chứng của việc thiếu một cơ
chế**, không phải một sự cố của sprint này. Manager thấy điều hai tầng dưới không thấy: 4 người khác
nhau có quyền gán nhãn P0, không ai trong số họ chịu hệ quả của việc gán sai, và không có ai được
chỉ định làm người quyết cuối cùng khi có xung đột. Hành động đúng ở tầng này không phải là quyết
giúp Hà thứ tự của 5 việc — đó là việc Hà làm được. Hành động đúng là đổi cơ chế: một người quyết
cuối cùng cho ưu tiên sản phẩm, hạn mức số lượng P0 (ví dụ tối đa 3 hạng mục P0 tại một thời điểm,
muốn thêm phải bỏ ra), và một cửa sổ cố định hàng tuần để bàn thay đổi ưu tiên thay vì bàn bất cứ lúc
nào.

Điểm mấu chốt của ba góc nhìn: cùng một hiện tượng, ba tầng có ba đòn bẩy khác nhau, và **nếu một
tầng cố dùng đòn bẩy của tầng khác thì sẽ thất bại**. IC cố quyết ưu tiên toàn cục sẽ bị coi là vượt
quyền và thường quyết sai do thiếu thông tin kinh doanh. Tech Lead cố đổi cơ chế gán nhãn của tổ chức
mà không có sự hậu thuẫn sẽ tiêu uy tín vô ích. Manager cố quyết thứ tự chi tiết từng việc sẽ trở
thành điểm nghẽn và làm hai tầng dưới mất năng lực phán đoán.

**Tình huống B — Prioritization trong ODC khi khách quyết scope.**

Bối cảnh: ODC cho khách Nhật, 12 engineer, hợp đồng theo tháng. Khách gửi danh sách 30 hạng mục mỗi
sprint và coi tất cả là bắt buộc. Team làm được khoảng 18. Sau ba sprint, mỗi sprint đều "trễ", và
quan hệ với khách xấu đi dù đầu ra thực tế không đổi.

Chẩn đoán: vấn đề không phải năng suất mà là **không có cơ chế đàm phán biên**. Team đang nhận toàn
bộ rủi ro của việc scope vượt capacity, và đang trả bằng uy tín.

Cách xử lý: dùng MoSCoW như một công cụ giao tiếp, không như một công cụ nội bộ. Trước mỗi sprint,
team gửi lại danh sách 30 hạng mục đã được phân nhóm, kèm capacity: "Must — 14 hạng mục, chúng tôi
cam kết; Should — 6 hạng mục, sẽ làm nếu Must xong sớm; Could — 10 hạng mục, đề nghị chuyển sprint
sau." Kèm một câu hỏi duy nhất cho khách: "Nếu nhóm Must của quý vị khác với đề xuất của chúng tôi,
xin cho biết hạng mục nào cần đổi vào và hạng mục nào ra."

Kết quả sau hai sprint: khách bắt đầu tự sắp thứ tự trước khi gửi, và số lượng hạng mục mỗi sprint
giảm về khoảng 20. Cơ chế hoạt động: khi việc lựa chọn được đưa về phía người có quyền quyết scope,
họ phải đối diện với ràng buộc capacity — điều mà trước đó họ không thấy vì team đã hấp thụ nó.

Trade-off phải nói rõ: cách này làm cho một số sprint có ít hạng mục hơn trên báo cáo, và có thể bị
người đọc báo cáo hiểu là năng suất giảm. Cần chuẩn bị trước bằng cách nêu tỉ lệ cam kết/hoàn thành
như một chỉ số chất lượng, không chỉ nêu số lượng. Cách trình bày cho stakeholder nước ngoài chi tiết
ở `07-project-delivery.md`.

### Best Practices

**Gán dạng đường Cost of Delay trước khi ước lượng effort.** Lý do: nếu ước lượng effort trước, thảo
luận có xu hướng dồn vào tranh luận kỹ thuật và mất dấu câu hỏi kinh doanh. Ngoài ra, nhiều hạng mục
sẽ bị loại ngay ở bước gán dạng đường ("chậm một quý cũng không mất gì") và không cần ước lượng —
tiết kiệm chi phí đo.

**Luôn viết ra đường cắt và những gì nằm dưới nó.** Lý do: phần giá trị lớn nhất của prioritization
nằm ở việc thông tin "không làm" được truyền đi. Nếu chỉ truyền thứ tự, mọi người vẫn giữ kỳ vọng và
tiếp tục hỏi, và bạn tiếp tục trả chi phí giải thích mỗi tuần.

**Giới hạn số hạng mục ở mức ưu tiên cao nhất bằng một con số cứng.** Lý do: đây là ràng buộc cơ chế,
không phải kỷ luật. Khi số lượng P0 có trần, việc thêm một P0 buộc phải bỏ một P0, và cuộc tranh luận
về trade-off xảy ra tự nhiên thay vì phải do bạn khởi xướng mỗi lần.

**Với mỗi yêu cầu "gấp", tách yêu cầu khỏi nhu cầu bằng câu hỏi "chậm một tuần thì ai không làm được
gì".** Lý do: câu hỏi này có tỉ lệ phát hiện cao — nó thường lộ ra rằng nhu cầu thật rẻ hơn giải pháp
được đề xuất, hoặc lộ ra rằng không có nhu cầu thật nào phía sau.

**Xem lại các hạng mục nằm lâu trong backlog mỗi quý và đóng bớt.** Lý do: một hạng mục tồn tại hai
quý mà chưa bao giờ vượt đường cắt là dữ liệu — nó nói rằng việc này không đủ quan trọng theo hàm ưu
tiên hiện tại. Giữ nó chỉ tạo ra nhiễu và ảo giác cho người đề xuất. Đóng nó và nói rõ lý do là hành
động tôn trọng người đề xuất hơn là để nó nằm im mãi.

**Khi hai người bất đồng về ưu tiên, chuyển tranh luận sang tranh luận về giả định, không về thứ
hạng.** Lý do: thứ hạng là đầu ra; giả định là đầu vào. Hai người thường bất đồng vì có giả định
khác nhau về giá trị hoặc về xác suất, và khi giả định được nói ra, một trong hai bên thường đổi ý
mà không cần ai thắng. Kỹ thuật này được mở rộng ở `04-decision-making.md`.

### Anti-patterns

**Lạm phát P0 (mọi thứ đều P0).**
Cơ chế: nhãn ưu tiên là một hệ tín hiệu. Khi có nhiều người được phát tín hiệu mà không ai chịu chi
phí cho việc phát sai, hệ tín hiệu sẽ bị lạm phát cho tới khi mất hết giá trị phân biệt — hoàn toàn
tương tự lạm phát tiền tệ. Từ đó, thứ tự thực thi rơi về các lực không chính thức, và không ai chịu
trách nhiệm cho thứ tự đó.
Dấu hiệu sớm: xuất hiện nhãn mới cao hơn P0 ("P0 blocker", "critical urgent"); người ta phải nhắc lại
bằng lời nói để việc của mình được làm; dev hỏi "cái nào thật sự gấp trong mấy cái gấp này".

**Ưu tiên theo ai gào to nhất (squeaky wheel) và theo HiPPO.**
Cơ chế: hệ thống chuyển từ tối ưu giá trị sang tối ưu **âm lượng và thứ bậc**. Hai hệ quả nối tiếp:
những người có tính cách nhẹ nhàng hoặc ở xa trung tâm quyền lực sẽ không bao giờ được phục vụ, dù
nhu cầu của họ có giá trị cao hơn; và các thành viên học được rằng cách để việc của mình được làm là
leo thang cảm xúc chứ không phải xây lập luận. Trong bối cảnh Việt Nam, biến thể phổ biến là ưu tiên
theo thứ bậc — ý của người có chức vụ cao nhất trong phòng thắng, và không ai phản biện vì phản biện
cấp trên là điều khó nói.
Dấu hiệu sớm: thứ tự backlog đổi sau mỗi cuộc họp có mặt lãnh đạo; các quyết định ưu tiên không được
ghi lại ở đâu (vì nếu ghi lại sẽ thấy chúng mâu thuẫn nhau); một người trong team nói "cái này chị X
yêu cầu" như một lý do đủ.

**False precision — dùng framework có điểm số khi không có dữ liệu.**
Cơ chế: RICE cho ra 8.4 và 7.9, và con số làm cuộc tranh luận dừng lại. Nhưng nếu Reach và Impact
được đoán, sai số của điểm số lớn hơn nhiều khoảng cách giữa hai số đó. Tệ hơn: điểm số che mất giả
định, nên không ai phản biện được — và giả định sai không bị phát hiện cho tới khi tính năng ra mắt
và không ai dùng.
Dấu hiệu sớm: không ai giải thích được Reach được lấy từ đâu; các điểm số không bao giờ được đối
chiếu lại với kết quả thực tế sau khi ra mắt.

**Ưu tiên lại quá thường xuyên (thrashing).**
Cơ chế: mỗi lần đổi ưu tiên, phần việc đang dở trở thành tồn kho chưa cho ra giá trị, và chi phí tái
nạp bối cảnh phải trả lại. Nếu tần suất đổi cao hơn thời gian hoàn thành trung bình của một hạng mục,
hệ thống về mặt toán học không thể hoàn thành gì — nó chỉ tích luỹ WIP.
Dấu hiệu sớm: số hạng mục in-progress lớn hơn số người; nhiều nhánh code sống lâu không merge;
câu "cái đó tạm dừng rồi" xuất hiện thường xuyên trong stand-up.

**Chỉ ưu tiên việc có giá trị nhìn thấy được, bỏ hoàn toàn việc giảm rủi ro.**
Cơ chế: việc dọn nợ và giảm rủi ro có Cost of Delay dạng hàm mũ nhưng không có ai đòi trong ngắn hạn.
Vì hàm ưu tiên chỉ nhận input từ những người biết đòi, loại việc này luôn ở dưới đường cắt cho tới
khi nó chuyển thành incident — lúc đó nó nhảy lên P0 với chi phí lớn hơn nhiều lần.
Dấu hiệu sớm: change failure rate tăng dần; thời gian để thêm một tính năng tương tự tăng theo quý;
mọi người bắt đầu nói "chỗ đó không ai dám sửa".

**Prioritization cá nhân theo cảm giác hoàn thành.**
Cơ chế: việc nhỏ cho phản hồi dopamine nhanh và tạo cảm giác tiến triển. Nếu bạn xếp việc theo cảm
giác đó, bạn sẽ có tuần rất bận và không có tiến triển ở việc quan trọng — vì việc quan trọng gần như
luôn có tiến triển không nhìn thấy trong những ngày đầu.
Dấu hiệu sớm: bạn bắt đầu ngày bằng việc dọn inbox; bạn kể được nhiều việc đã làm nhưng không kể được
vấn đề nào đã đóng.

### Khi nào KHÔNG nên áp dụng

**Trong incident đang diễn ra.** Không xếp ưu tiên bằng framework khi production đang lỗi. Trong
incident, cơ chế đúng là một Incident Commander quyết theo thứ tự cố định: chặn thiệt hại lan rộng →
phục hồi dịch vụ → bảo toàn dữ liệu và bằng chứng → điều tra nguyên nhân. Áp RICE hay WSJF vào thời
điểm đó là hành vi nghi lễ. Xem `06-incident-va-metrics.md`.

**Khi số hạng mục dưới 5 và khác biệt giữa chúng rõ ràng.** Chi phí của việc chạy framework (họp, gán
điểm, tranh luận) vượt giá trị khi câu trả lời đã hiển nhiên với mọi người trong phòng. Dùng framework
ở đây tạo ra hai tác hại: tốn thời gian, và làm loãng giá trị của framework khi thật sự cần đến — nếu
mọi việc đều phải qua quy trình, người ta sẽ tìm cách đi vòng.

**Khi bạn không phải người có quyền quyết và cũng không được mời cho input.** Trong bối cảnh ODC nơi
scope do khách quyết hoàn toàn, việc bạn xây một bảng WSJF nội bộ rồi thất vọng vì nó không được dùng
là chi phí thuần. Việc đúng cần làm trước là **giành quyền cung cấp input** — xin một khe 15 phút
trong buổi planning với khách để trình bày rủi ro kỹ thuật. Framework chỉ có giá trị khi có đường
nối tới quyết định thật.

**Khi vấn đề thật là capacity, không phải thứ tự.** Nếu backlog cần 400 ngày người và capacity quý là
120 ngày người, việc xếp lại thứ tự tối ưu 20 lần cũng không đổi kết quả căn bản. Trong điều kiện
này, prioritization tinh vi trở thành cách để tránh cuộc đối thoại thật: cần cắt scope ở tầng chiến
lược, cần thêm người, hoặc cần lùi cam kết. Dấu hiệu bạn đang ở tình huống này: mỗi quý ba việc đầu
đều xong, và mọi việc còn lại đều được chuyển sang quý sau nguyên vẹn.

**Khi tổ chức đang trong giai đoạn thăm dò (exploration) chưa biết mình cần gì.** Với một sản phẩm
mới chưa có người dùng, giá trị kỳ vọng của mọi hạng mục đều là phỏng đoán, nên hàm ưu tiên không có
input đáng tin. Trong giai đoạn này, tiêu chí đúng để xếp thứ tự không phải giá trị mà là **lượng bất
định giảm được trên một đơn vị thời gian** — làm việc nào giúp bạn biết mình sai ở đâu nhanh nhất.
Áp RICE vào giai đoạn này cho ra các con số đẹp và vô nghĩa.

---

## 4. Personal Productivity System

### Problem Statement

Nam là Tech Lead trong bộ phận IT của một ngân hàng, team 8 người, ba luồng công việc song song: một
dự án chuyển đổi core report, một luồng hỗ trợ nghiệp vụ, và một luồng compliance. Thử nghiệm sau đây
có thể làm với bất kỳ ai trong vai này: sáng thứ Hai, yêu cầu Nam liệt kê mọi việc anh đang có cam
kết với người khác. Nam viết ra 9 việc trong 4 phút.

Rà lại Slack, email, Jira và biên bản họp của hai tuần trước, con số thật là 23 cam kết còn mở. 14
việc không tồn tại ở bất kỳ hệ thống nào ngoài đầu Nam và đầu người kia. Trong 14 việc đó: 5 việc đã
quá hạn mà Nam chưa biết, 2 việc đang chặn người khác, 1 việc là lời hứa với bộ phận nghiệp vụ trong
một cuộc họp cách đây 11 ngày mà Nam không còn ký ức nào về nó.

Đây không phải vấn đề trí nhớ kém. Đây là vấn đề **một hệ thống có 23 process đang chạy trên một bộ
nhớ chứa được 7 đơn vị**.

Hiện tượng quan sát được khi không có Personal Productivity System, đo được:

- **Số cam kết được phát ra bằng miệng trong họp mà không tồn tại ở bất kỳ hệ thống nào.** Cách đo: mở
  biên bản hoặc recording của ba buổi họp gần nhất, đếm mọi câu chứa "để anh...", "em sẽ...", "mình
  check lại nhé", rồi đối chiếu với ticket/todo. Tỉ lệ không tìm thấy thường 50–70%.
- **Tỉ lệ việc quá hạn được phát hiện bởi người khác thay vì bởi bạn.** Đây là chỉ số chất lượng cao
  nhất của hệ thống ngoài. Nếu bạn hầu như chỉ biết mình trễ khi có người nhắc, hệ thống ngoài của bạn
  đang là bộ nhớ của đồng nghiệp.
- **Số lần bị nhắc lại trong một tuần** ("anh ơi cái PR hôm trước...", "chị ơi cái file em gửi..."). Mỗi
  lần bị nhắc là một lần chi phí bảo trì bộ nhớ của bạn được chuyển sang người khác.
- **Thời gian tìm lại thông tin mình từng biết.** Đo thô: trong một ngày, bao nhiêu lần bạn phải scroll
  Slack để tìm một quyết định hoặc một link. Ở người không có hệ thống, con số này thường 8–15 lần/ngày.
- **Cảm giác "còn gì đó chưa làm" vào tối Chủ nhật.** Đây là triệu chứng chủ quan nhưng có cơ chế khách
  quan phía sau (xem First Principles), và nó là chỉ báo sớm hơn mọi con số ở trên.

Hệ quả lan xuống team, và đây là phần ít được nói: khi lead không có hệ thống, team buộc phải xây hệ
thống lưu trữ hộ. Hệ quả thứ nhất là chi phí phân tán — mỗi người phải giữ một bản sao các cam kết của
lead. Hệ quả thứ hai nghiêm trọng hơn và mang tính bất công: **việc của bạn được làm hay không phụ
thuộc vào việc bạn có dám nhắc hay không.** Trong bối cảnh Việt Nam, người ít dám nhắc nhất thường là
junior, người mới, và người ở xa trung tâm — chính những người cần được unblock nhất. Sau vài tháng,
team học được một quy tắc ngầm: muốn được phục vụ thì phải nhắc nhiều lần, và ai nhắc dai thì được ưu
tiên. Đó là một hàm ưu tiên do sự thiếu tổ chức của lead tạo ra.

### First Principles

**Cơ chế một: working memory là tài nguyên dùng chung, không phải một khoang lưu trữ riêng.** Working
memory của người trưởng thành giữ được rất ít đơn vị đồng thời (các nghiên cứu công khai từ Miller
đến Cowan cho các con số khác nhau, khoảng 4–7). Điểm quan trọng không phải con số, mà là: **cùng một
tài nguyên đó vừa dùng để giữ các việc đang mở, vừa dùng để suy nghĩ.** Mỗi cam kết bạn giữ trong đầu
không nằm ở một chỗ riêng — nó chiếm một slot mà lẽ ra dùng để giữ mô hình của bài toán bạn đang giải.
Đây là lý do một người có 20 việc mở trong đầu thực sự *kém thông minh hơn* chính họ khi có 3 việc mở.
Không phải ẩn dụ; đó là cùng một hạn mức.

**Cơ chế hai: hiệu ứng Zeigarnik và điều nghiên cứu tiếp theo phát hiện.** Zeigarnik (nguồn công khai,
thập niên 1920) quan sát rằng các nhiệm vụ chưa hoàn thành có xu hướng được ghi nhớ và xâm nhập vào ý
thức nhiều hơn nhiệm vụ đã hoàn thành. Phần quan trọng hơn cho việc thiết kế hệ thống lại đến từ dòng
nghiên cứu sau đó (Masicampo & Baumeister, 2011, nguồn công khai): việc **lập một kế hoạch cụ thể** cho
nhiệm vụ chưa xong làm giảm sự xâm nhập gần như tương đương với việc hoàn thành nó.

Suy ra kết luận thiết kế quan trọng nhất của mục này: **ghi lại (capture) một mình không đủ để giải
phóng nhận thức.** Một danh sách 40 dòng chưa được xử lý vẫn xâm nhập, vì mỗi dòng vẫn là một câu hỏi
mở. Sự giải phóng đến từ bước **clarify** — quyết định bước kế tiếp cụ thể và khi nào. Đây là lý do
những người "có ghi chép" mà vẫn thấy nặng đầu không thiếu công cụ; họ thiếu bước hai.

**Cơ chế ba: hệ thống chỉ chịu tải khi được tin, và niềm tin đến từ nhịp review.** Nếu bạn không rà lại
danh sách theo một nhịp ổn định, não bạn học được (đúng) rằng danh sách không đáng tin, và nó giữ một
bản sao song song để phòng thân. Khi đó bạn trả chi phí hai lần: chi phí ghi, và chi phí giữ trong đầu.
Đây là dạng thất bại phổ biến nhất của mọi hệ thống productivity, và nó là thất bại của nhịp, không
phải của công cụ.

**Cơ chế bốn: danh sách là một hàng đợi, và hàng đợi có luật của nó.** Nếu tốc độ nạp vào lớn hơn tốc
độ xử lý một cách bền vững, độ dài hàng đợi tăng không giới hạn. Một danh sách 300 việc không phải là
một hệ thống tổ chức tốt — nó là một nghĩa địa, và nghĩa địa phá niềm tin (cơ chế ba). Hệ quả: mọi hệ
thống bền được phải có **cơ chế xả**: một quyết định "không làm" tường minh, chứ không phải để việc tự
mục đi.

**Cơ chế năm: chi phí lớn nhất không phải chi phí lưu, mà chi phí quyết định lại.** Mỗi lần bạn đọc lại
một dòng trong danh sách và không quyết gì, bạn đã trả toàn bộ chi phí nạp bối cảnh và không nhận được
đầu ra nào. Chức năng chính của hệ thống, do đó, không phải là *nhớ hộ* — mà là **ngăn bạn phải quyết
lại cùng một điều nhiều lần**. Đây là tiêu chí để đánh giá mọi lựa chọn thiết kế trong hệ thống của
bạn: nó có làm giảm số lần quyết lại không?

### Mental Model

**Mô hình 1 — Bộ nhớ ngoài như một hệ thống phân tầng, giống cache hierarchy.**

```
L1  INBOX          — biến động, phải xả về 0 mỗi ngày
                     tốc độ ghi: < 5 giây. Không phân loại ở tầng này.
L2  NEXT ACTIONS   — việc có bước kế tiếp cụ thể, làm được trong tuần này
                     rà mỗi ngày, chọn từ đây khi có khe trống.
L3  COMMITMENTS    — việc nhiều bước, dự án, cam kết với người khác
     + WAITING-FOR   rà mỗi tuần. Mỗi dòng phải có ít nhất một next action.
L4  SOMEDAY /      — kho lạnh. Rà mỗi quý. Được phép chết ở đây.
    REFERENCE
```

Sai lầm phổ biến nhất là **để mọi thứ ở L1** — dùng inbox mail hoặc channel Slack làm danh sách việc.
Cơ chế phá hoại: L1 được sắp thứ tự bởi *thời điểm đến*, tức bởi người khác. Làm việc từ L1 nghĩa là
giao hàm ưu tiên của bạn cho tốc độ gõ của đồng nghiệp.

**Mô hình 2 — Open loop chỉ có ba đường ra.**

Mỗi việc đang mở là một process chiếm RAM. Đóng nó nghĩa là làm một trong đúng ba việc:

```
1. LÀM      → có bước kế tiếp + có chỗ trên lịch  (vào L2 hoặc Calendar)
2. GIAO     → có tên người + có ngày kiểm tra lại (vào WAITING-FOR)
3. KHÔNG LÀM→ quyết định tường minh + ghi lại lý do (vào L4 hoặc xoá)
```

Không có đường thứ tư. "Để đó xem sao" không phải một đường ra — nó là process vẫn chạy, chỉ là bạn đã
ngừng gọi tên nó. Phần lớn cảm giác quá tải không đến từ số việc phải làm, mà từ số việc chưa được đưa
vào một trong ba đường này.

**Mô hình 3 — Hệ thống là cái phanh, không phải cái động cơ.**

Kỳ vọng sai về productivity system: nó làm bạn làm được nhiều hơn. Thực tế cơ chế: nó làm cho **chi phí
của một cam kết mới trở nên hữu hình tại đúng thời điểm bạn cam kết**. Khi bạn nhìn thấy 12 việc đang
mở và 2 block trống trong tuần, câu "được, để anh làm" trở nên khó nói ra một cách vô thức. Đó là giá
trị lớn nhất — và nó là giá trị của việc từ chối, không phải của việc làm nhanh hơn.

### Practical Framework

Bộ khung dưới đây là phần chịu tải được của *Getting Things Done* (David Allen, nguồn công khai), đã
được rút gọn cho công việc kỹ thuật. Bốn bước: **capture → clarify → organize → review**.

**Bước 1 — Capture: đúng một inbox, dưới 5 giây.**

Yêu cầu thiết kế, không phải sở thích:

- **Một inbox duy nhất** (tối đa hai nếu bạn cần một trên điện thoại, và chúng phải tự đồng bộ). Lý do:
  hai inbox trở lên tạo ra bài toán "mình ghi ở đâu", và bài toán đó tiêu đúng thứ tài nguyên mà việc
  capture định giải phóng.
- **Ghi được dưới 5 giây, không cần phân loại.** Nếu việc ghi cần chọn project, chọn tag, chọn ngày,
  bạn sẽ không ghi vào lúc bận — tức là đúng lúc cần nhất.
- **Capture ngay trong cuộc họp, thành tiếng.** Kỹ thuật cụ thể: khi nhận một việc, nói ra "để em ghi
  lại: em sẽ gửi anh bản so sánh trước thứ Năm". Câu này làm ba việc cùng lúc — đưa việc vào hệ thống,
  biến cam kết mơ hồ thành cam kết có ngày, và lộ ra ngay tại chỗ nếu bạn đang quá tải (vì bạn buộc
  phải nói một ngày cụ thể).

**Bước 2 — Clarify: hai câu hỏi cho mỗi dòng inbox.**

Đây là bước tạo ra toàn bộ hiệu ứng giải phóng nhận thức, và là bước bị bỏ nhiều nhất.

1. **Việc này thực sự là gì?** Kết quả mong muốn là gì, đo bằng gì. "Xem lại kiến trúc payment" chưa
   phải một việc; "đọc ba luồng chính của payment và viết ra ba rủi ro để đưa vào buổi review thứ Sáu"
   là một việc.
2. **Bước kế tiếp *vật lý* là gì?** Không phải "suy nghĩ về X" mà "mở file Y và viết ba dòng", "gửi tin
   nhắn cho Nam hỏi Z". Lý do: một dòng không có bước vật lý sẽ bị bỏ qua mỗi lần bạn đọc danh sách, và
   mỗi lần bỏ qua đó là một lần trả chi phí đọc mà không có đầu ra.

Kèm hai quy tắc rẻ: việc dưới 2 phút thì làm ngay tại bước clarify (chi phí ghi và quản lý lớn hơn chi
phí làm); việc có thời điểm cố định thì vào Calendar, không vào danh sách.

**Bước 3 — Organize: bốn danh sách, không phải mười lăm.**

| Danh sách | Chứa gì | Nhịp rà | Vì sao cần |
|---|---|---|---|
| **Next actions** | Việc có bước vật lý, làm được tuần này. Nhóm theo bối cảnh: cần máy / cần gặp người / cần block dài | Mỗi ngày | Là nơi bạn chọn việc khi có khe trống, thay vì mở Slack |
| **Waiting-for** | Việc bạn đã giao hoặc đang chờ người khác. Mỗi dòng: tên người + ngày giao + ngày kiểm tra | Mỗi tuần | Đây là nơi ownership của việc đã giao tồn tại (xem mục 1) |
| **Calendar** | Chỉ việc có thời điểm thật + các block đã đặt (mục 2) | Mỗi ngày | Calendar không phải danh sách mong muốn; đưa việc không có thời điểm vào đây làm hỏng độ tin cậy của lịch |
| **Someday / reference** | Ý tưởng, việc có thể làm sau, tài liệu tham chiếu | Mỗi quý | Chỗ để thứ gì đó *chết một cách tường minh* thay vì mục dần trong Next actions |

**Waiting-for là danh sách bị bỏ nhiều nhất và có giá trị cao nhất cho một lead.** Cơ chế: khi bạn giao
một việc mà không có dòng waiting-for, bạn chỉ có hai chế độ — hoặc giữ nó trong đầu (tốn slot), hoặc
quên nó (rồi tự làm lại vào tuần sau, dạy cho người kia rằng không làm cũng không sao). Danh sách này
là điều kiện kỹ thuật để Delegation hoạt động; xem `03-team-leadership.md`.

**Bước 4 — Weekly review 30 phút, có checklist và có đồng hồ.**

Đặt lịch cố định (khuyến nghị: chiều thứ Sáu, sau đó không nhận họp). Thời lượng từng phần là để chống
việc buổi review biến thành buổi làm việc:

- [ ] **(3 phút) Xả inbox về 0.** Mọi dòng đi qua bước clarify. Không được để lại dòng nào "để mai".
- [ ] **(5 phút) Rà calendar hai tuần vừa qua.** Câu hỏi duy nhất: trong các buổi họp đó tôi đã hứa gì
      mà chưa vào hệ thống? Đây là bước bắt được đúng loại rò rỉ ở đầu mục (14/23 cam kết của Nam).
- [ ] **(5 phút) Rà calendar hai tuần tới.** Buổi nào cần chuẩn bị tài liệu? Việc chuẩn bị đó đã có
      block chưa? Có buổi họp đêm nào cần dịch chuyển việc khác để bù?
- [ ] **(5 phút) Rà Waiting-for.** Dòng nào quá ngày kiểm tra? Với mỗi dòng: nhắc, escalate, hoặc bỏ.
      Không được để nguyên.
- [ ] **(5 phút) Rà từng commitment ở L3.** Mỗi cái có ít nhất một next action không? Cái nào không có
      là cái đang chết — và nó thường là việc quan trọng nhất, vì việc quan trọng thường không có bước
      kế tiếp hiển nhiên.
- [ ] **(4 phút) Chọn ba việc quan trọng nhất tuần tới và đặt block cụ thể trên calendar.** Đây là mối
      nối với mục 2 và mục 3 — không có bước này, review chỉ là dọn dẹp.
- [ ] **(3 phút) Rà Someday.** Có gì cần đóng hẳn? Việc nào đã nằm đây hai quý thì viết một dòng lý do
      và xoá.

Tiêu chí biết là xong — cả ba phải đúng: inbox bằng 0; mỗi commitment có một next action; ba việc quan
trọng nhất có chỗ trên lịch. Tiêu chí chủ quan bổ sung: bạn ra khỏi buổi review mà không còn cảm giác
"còn gì đó" — nếu vẫn còn, thường là do có một cam kết bạn biết mình sẽ không làm mà chưa dám ghi ra
quyết định không làm.

**Bước 5 — Quản lý inbox và notification: ba mức, mặc định là mức thấp nhất.**

| Mức | Nghĩa | Áp cho | Cấu hình cụ thể |
|---|---|---|---|
| **Interrupt** | Được phép cắt bạn giữa việc khó | Alert production, tin nhắn trực tiếp từ 3–4 người đã liệt kê, cuộc gọi | Bật notification kể cả khi Do-not-disturb |
| **Batch** | Xử lý theo cửa sổ đã công bố | Channel dự án, email công việc, PR review | 2–3 cửa sổ/ngày, có giờ cố định |
| **Pull** | Chỉ mở khi bạn chủ động mở | Channel thông báo chung, group xã hội, newsletter | Tắt hoàn toàn notification |

Quy tắc lọc rất rẻ và rất hiệu quả: **bất kỳ notification nào bạn không hành động ngay khi nó đến thì
phải tắt.** Nó không mang thông tin bạn dùng; nó chỉ tiêu attention và tạo attention residue.

Xử lý cụ thể cho bối cảnh Việt Nam: Zalo thường lẫn công việc với đời sống và có nhiều group không thể
rời vì lý do xã hội. Cách khả thi là **mute toàn bộ theo mặc định rồi mở theo danh sách trắng**, cộng
với việc đặt một tin nhắn ghim trong group dự án nêu rõ kênh nào dùng cho việc gấp. Không cố gắng rời
group — chi phí xã hội cao hơn lợi ích, và mute đạt được đúng mục tiêu.

### Trade-off

| Trục | Nghiêng về bên A khi | Nghiêng về bên B khi | Ai chịu phần mất |
|---|---|---|---|
| **A: Hệ thống chi tiết — B: Hệ thống tối thiểu** | Bạn có 3+ luồng công việc song song, nhiều cam kết với người ngoài team, và nhiều việc chờ người khác | Công việc đơn tuyến; hoặc bạn đang trong giai đoạn biến động cao (crunch, incident, mới nhận vai) | Chi tiết quá: chi phí bảo trì vượt lợi ích, hệ thống bị bỏ đúng vào tuần bận nhất. Tối thiểu quá: cam kết rơi, người khác trả giá |
| **A: Ghi mọi thứ — B: Ghi có chọn lọc** | Bạn là nút giao của nhiều luồng; hậu quả của việc quên chạm tới người khác | Bạn làm sâu một việc duy nhất trong nhiều tuần | Ghi mọi thứ: danh sách phình thành nghĩa địa, mất niềm tin, quay về giữ trong đầu |
| **A: Một hệ thống duy nhất — B: Tách hệ cá nhân và hệ team** | Bạn muốn giảm số chỗ phải rà | Việc của team phải hiển thị cho team (Jira/board); việc riêng của bạn không nên nằm ở đó | Gộp hết vào hệ cá nhân: bus factor = 1, team không thấy trạng thái. Tách quá: bạn phải rà hai chỗ, và cái ít rà hơn sẽ rò |
| **A: Đầu tư thời gian vào công cụ — B: Dùng công cụ thô, đầu tư vào nhịp** | Bạn đã chạy được nhịp review 3 tháng liền và đang thực sự bị công cụ giới hạn | Bạn chưa duy trì được weekly review 4 tuần liên tiếp | Đầu tư công cụ trước khi có nhịp: bạn có một hệ thống rất đẹp và không dùng — đây là dạng trì hoãn có vỏ năng suất |

Trục thứ tư là trục có sai lệch phổ biến nhất trong ngành, vì việc cấu hình công cụ *giống như* làm
việc: nó có đầu ra nhìn thấy được, có cảm giác tiến triển, và không có ai phản đối. Nhưng biến quyết
định sự sống còn của hệ thống là nhịp review, không phải cấu trúc dữ liệu. Một file markdown với bốn
đề mục và weekly review đều đặn chịu tải tốt hơn một workspace Notion 12 database không được rà.

### Real-world Scenarios

**Tình huống A — Hai lần xây hệ thống, một lần sống.**

Bối cảnh: chính Nam ở đầu mục. Sau khi phát hiện 14 cam kết không được ghi ở đâu, Nam sửa hai lần.

*Lần thứ nhất (thất bại sau 3 tuần):* Nam dựng một workspace Notion với 5 database liên kết nhau —
Projects, Tasks, Meetings, People, Decisions — mỗi task có 9 property (priority, effort, area, stakeholder,
status, due, blocked-by, energy, context). Hai tuần đầu Nam rất hài lòng: mọi thứ hiển thị đẹp, có
dashboard, có view theo tuần. Tuần thứ ba có một sự cố hệ thống report kéo dài 4 ngày. Trong 4 ngày đó
Nam không nhập được gì vào hệ thống. Sau sự cố, hệ thống đã lệch với thực tế tới mức việc cập nhật lại
mất khoảng 2 giờ, và Nam không bao giờ làm 2 giờ đó. Hệ thống chết.

Cơ chế thất bại, nói rõ để không lặp lại: chi phí ghi một dòng khoảng 90 giây (vì phải điền 9 property).
Ở mức đó, việc capture không xảy ra trong lúc bận — mà lúc bận là chính xác lúc hệ thống cần hoạt động.
Đây là một lỗi thiết kế cổ điển: **tối ưu cho ngày tốt nhất thay vì cho ngày tệ nhất.**

*Lần thứ hai (sống được 14 tháng tính đến lúc viết case này):* Nam dùng một file markdown trong repo
riêng, đồng bộ qua git, cộng app note trên điện thoại làm inbox phụ. File có đúng bốn đề mục: `INBOX`,
`NEXT`, `WAITING`, `SOMEDAY`. Không có property nào. Waiting-for có định dạng cố định một dòng:
`- [Tuấn] runbook reconcile — giao 12/3 — check 19/3`. Weekly review 30 phút chiều thứ Sáu, đã đặt lịch
định kỳ và đánh dấu busy.

Kết quả sau 3 tháng, đo được: số lần bị người khác nhắc giảm từ trung bình 6 lần/tuần xuống 1; số cam
kết quá hạn mà Nam tự phát hiện trước người khác tăng từ khoảng 20% lên khoảng 80%; và một hiệu ứng
không dự đoán trước — Nam bắt đầu **từ chối nhiều hơn**, vì trong cuộc họp anh có thể mở file ra và
thấy 11 việc đang mở. Chính hiệu ứng này, không phải hiệu ứng ghi nhớ, là phần giá trị lớn nhất.

Bài học truy được về cơ chế: sức chịu tải của hệ thống nằm ở **chi phí biên của một lần dùng**, không ở
sức mạnh biểu đạt của công cụ. Mọi thứ làm tăng chi phí biên (nhiều field, nhiều tầng phân loại, nhiều
chỗ phải mở) đều làm giảm xác suất hệ thống còn sống sau cú sốc đầu tiên.

**Tình huống B — Khi bạn quên một cam kết và bị nhắc.**

Đây là tình huống xảy ra với mọi người, và cách xử lý nó có tác động đến hệ thống của cả team, không
chỉ của bạn. Cách xử lý sai phổ biến ở Việt Nam có hai biến thể: giải thích dài về việc mình bận (làm
người kia thấy có lỗi vì đã nhắc, và họ sẽ ngần ngừ lần sau), hoặc xin lỗi mơ hồ rồi không đặt cơ chế
gì (nên việc sẽ lặp lại).

> **Hà (Mid BE, 26 tuổi):** Anh ơi, cái PR về luồng notification em nhờ anh review từ thứ Ba, không
> biết anh có xem chưa ạ... em cũng không dám nhắc anh nhiều.
>
> **Bạn:** Chưa, anh chưa xem. Cái đó rơi khỏi danh sách của anh, lỗi ở anh chứ không phải em nhắc
> nhiều. Anh review trong hôm nay, trước 5h.
>
> **Hà:** Vâng, không gấp đâu ạ, anh cứ xem khi nào rảnh.
>
> **Bạn:** Không, nó gấp — em đang bị chặn ba ngày rồi. Và anh muốn sửa cái cơ chế để nó không lặp lại,
> vì đây là lần thứ hai trong tháng. Hai việc: một, từ giờ PR em đừng nhắn riêng cho anh, tag anh trong
> channel `#review` — anh có cửa sổ review 11h mỗi ngày và anh làm theo danh sách trong channel đó, cái
> gì nhắn riêng thì anh dễ trôi. Hai, nếu quá 24 giờ mà chưa thấy anh comment, em nhắc anh, và anh coi
> việc em nhắc là một phần của quy trình chứ không phải em làm phiền. Nếu anh không phản hồi sau lần
> nhắc thứ hai thì em nói với anh Khoa, không phải chờ anh.
>
> **Hà:** Nói với anh Khoa thì em thấy hơi...
>
> **Bạn:** Anh hiểu. Nhưng anh nói trước với Khoa việc này rồi, coi như thoả thuận từ đầu. Em bị chặn
> ba ngày thì thiệt hại là của dự án, không phải chuyện riêng giữa anh và em.

Bốn điểm cần thấy trong script: (1) nhận lỗi ở đúng chỗ và ngắn — không giải thích về việc mình bận, vì
lời giải thích đó chuyển chi phí cảm xúc sang người nhắc; (2) sửa lại đánh giá mức độ gấp thay cho Hà,
vì cô ấy đang tự hạ mức để giữ hoà khí; (3) đổi kênh để việc rơi vào một hàng đợi có thể rà được thay
vì vào một cuộc trò chuyện riêng; (4) **hợp pháp hoá việc nhắc và cả việc escalate** — đây là phần quan
trọng nhất trong bối cảnh Việt Nam, nơi việc nhắc cấp trên hoặc người lớn tuổi hơn có chi phí xã hội
thật. Nếu bạn không tuyên bố trước rằng việc nhắc là hợp pháp, hệ thống của bạn sẽ chỉ nhận tín hiệu từ
những người dám nhắc.

### Best Practices

**Đúng một inbox, và chi phí ghi dưới 5 giây.** Lý do: xác suất capture xảy ra trong lúc bận là biến
quyết định toàn bộ hệ thống, và nó tỉ lệ nghịch với ma sát. Mọi tính năng làm tăng ma sát đều là chi
phí trả bằng độ bao phủ của hệ thống.

**Ghi cam kết ngay trong họp và nói ra thành tiếng, kèm một ngày cụ thể.** Lý do: ba tác dụng cùng lúc
— vào hệ thống, biến cam kết mơ hồ thành cam kết đo được, và lộ ra tình trạng quá tải tại thời điểm rẻ
nhất để đàm phán (trước khi nhận, không phải sau khi trễ).

**Mỗi dòng Waiting-for phải có tên người và ngày kiểm tra.** Lý do: không có ngày kiểm tra thì dòng đó
là một mong đợi, không phải một cơ chế; và bạn sẽ phát hiện việc chậm khi có người phàn nàn.

**Weekly review đặt lịch định kỳ và làm cả trong tuần bận nhất.** Lý do: giá trị của review tăng theo
mức độ bận, vì đó là lúc số việc rơi nhiều nhất. Bỏ review vào tuần bận là bỏ đúng lúc nó đắt giá nhất
— và đó cũng là cách mọi hệ thống chết.

**Thiết kế hệ thống cho ngày tệ nhất của bạn, không cho ngày tốt nhất.** Lý do: hệ thống chỉ có giá trị
nếu nó còn hoạt động khi bạn kiệt sức, đang trong incident, hoặc vừa họp đêm. Một hệ thống chỉ chạy khi
bạn tỉnh táo và có thời gian là một hệ thống không giải quyết vấn đề nó được tạo ra để giải quyết.

**Không dùng inbox mail hoặc Slack làm danh sách việc.** Lý do: thứ tự trong đó do người khác quyết
(thời điểm gửi), và không có chỗ ghi bước kế tiếp. Làm việc từ mail nghĩa là ưu tiên theo tốc độ gõ của
đồng nghiệp — hoàn toàn không tương quan với giá trị.

**Việc gì thuộc về team thì phải ở chỗ team thấy được, kể cả khi hệ thống cá nhân của bạn tiện hơn.**
Lý do: hệ thống cá nhân là bộ nhớ riêng; khi trạng thái của việc team chỉ tồn tại trong đó, tổ chức có
bus factor bằng 1 và team mất khả năng tự phối hợp khi bạn nghỉ.

### Anti-patterns

**Productivity porn — xây hệ thống thay cho làm việc.**
Cơ chế: việc cấu hình công cụ cho phản hồi nhanh, có cảm giác kiểm soát, và không có rủi ro thất bại
công khai — tức là nó có đúng đặc tính của một hành vi trì hoãn hấp dẫn, chỉ khác là nó trông giống
việc nghiêm túc nên không ai can. Tài nguyên bị tiêu là những block attention chất lượng cao.
Dấu hiệu sớm: bạn đổi công cụ chính hơn hai lần trong một năm; bạn dành thời gian cuối tuần để cấu hình
template; bạn có thể nói chi tiết về cấu trúc hệ thống của mình nhưng không nhớ lần cuối làm weekly
review.

**Hệ thống nhiều tầng phân loại.**
Cơ chế: mỗi tầng phân loại thêm vào là một quyết định phải ra tại thời điểm capture. Vì capture xảy ra
lúc bận, các quyết định đó bị hoãn, nên việc không được ghi. Hệ thống càng biểu đạt được nhiều thì độ
bao phủ càng thấp — và độ bao phủ mới là thứ tạo ra giá trị.
Dấu hiệu sớm: bạn có những việc "chưa biết xếp vào đâu" nên chưa ghi; inbox có dòng nằm quá một tuần.

**Ghi mà không review — danh sách thành nghĩa địa.**
Cơ chế: sau khoảng 4–6 tuần không rà, danh sách chứa đủ thứ đã hết hiệu lực để việc đọc nó trở nên khó
chịu. Não học được rằng danh sách không đáng tin và bắt đầu giữ bản sao song song. Từ đó bạn trả chi
phí hai lần và nhận lợi ích của không hệ thống nào.
Dấu hiệu sớm: bạn ghi việc vào hệ thống rồi vẫn nhắc lại nó trong đầu; bạn mở danh sách ra và cảm thấy
muốn đóng lại.

**Giữ cam kết của người khác trong đầu, rồi tự làm thay khi họ quên.**
Cơ chế: đây là martyr complex (mục 1) mặc bộ đồ của sự chu đáo. Nó có hai chi phí: bạn tiêu slot working
memory cho việc của người khác, và người kia không nhận được tín hiệu nào về việc họ đã bỏ cam kết —
nên hành vi đó tiếp tục.
Dấu hiệu sớm: bạn biết chính xác ai đang nợ gì mà không cần mở ghi chú; bạn thường xuyên làm những việc
mà tuần trước bạn đã giao cho người khác.

**Dùng hệ thống cá nhân để chứa thứ lẽ ra phải công khai.**
Cơ chế: quyết định kiến trúc, trạng thái dự án, danh sách rủi ro nằm trong file riêng của bạn. Hệ thống
của bạn hoạt động rất tốt — và tổ chức không có bộ nhớ nào. Khi bạn nghỉ phép hoặc đổi việc, thông tin
đó biến mất hoàn toàn.
Dấu hiệu sớm: người khác phải hỏi bạn những câu về trạng thái mà lẽ ra board hoặc tài liệu trả lời
được; bạn là người duy nhất biết một việc đang chờ ai.

**Coi inbox 0 là mục tiêu thay vì là hệ quả.**
Cơ chế: khi inbox 0 trở thành chỉ số, hành vi tối ưu là xử lý nhanh mọi thứ đến — tức là ưu tiên theo
thời điểm đến. Bạn có một inbox rỗng và một tuần không có tiến triển ở việc quan trọng.
Dấu hiệu sớm: bạn bắt đầu ngày bằng việc dọn inbox; bạn cảm thấy bất an khi có 5 email chưa đọc nhưng
không cảm thấy bất an khi việc quan trọng nhất của quý chưa được chạm tới trong hai tuần.

### Khi nào KHÔNG nên áp dụng

**Trong incident hoặc trong giai đoạn crunch — phải rút gọn hệ thống một cách có chủ đích.** Đây là
điều kiện biên quan trọng nhất, và nó có cơ chế rõ: trong incident, tần suất thay đổi trạng thái cao
hơn tần suất bạn cập nhật hệ thống. Hệ quả là hệ thống trở nên **sai**, và một hệ thống sai tệ hơn
không có hệ thống, vì bạn ra quyết định trên dữ liệu cũ mà vẫn tin là mới.

Cấu hình rút gọn cho giai đoạn này: đúng một danh sách duy nhất, ghi ở một chỗ mà mọi người thấy (một
pinned message trong channel incident, hoặc một tờ giấy nếu bạn đang ở chế độ war room); không phân loại;
không waiting-for tách riêng; và thay weekly review bằng một lần rà 5 phút cuối mỗi ngày với đúng hai
câu hỏi: cái gì đang chặn ai, và cái gì tôi đã hứa hôm nay. Toàn bộ phần còn lại của hệ thống được coi
là đóng băng.

Điều kiện đi kèm, không được bỏ: **sau khi crunch kết thúc, phải có một buổi phục hồi hệ thống 45 phút**
— rà lại calendar và Slack của toàn bộ giai đoạn crunch để bắt các cam kết đã phát sinh, và đưa hệ thống
về trạng thái đáng tin. Nếu bỏ buổi này, nợ tổ chức tích tụ âm thầm và bạn sẽ phát hiện nó qua các cuộc
gọi phàn nàn ba tuần sau đó.

**Khi công việc thật sự đơn tuyến.** Một Mid engineer làm một luồng, 2–3 task cùng lúc, một stakeholder
không cần bộ khung bốn danh sách — chi phí bảo trì vượt lợi ích rõ ràng. Với hình thái công việc đó,
một danh sách phẳng cộng board của team là đủ. Việc áp một hệ thống đầy đủ ở đây tạo ra tác hại đặc
trưng: người đó kết luận rằng "mấy cái này không thực tế", và sẽ khó thuyết phục lại ở thời điểm họ
thực sự cần — tức là khi lên vai lead.

**Khi vấn đề thật là quá tải, không phải thiếu tổ chức.** Nếu bạn có 40 cam kết và capacity cho 15, hệ
thống chỉ giúp bạn thấy rõ hơn mình đang trượt cái gì. Nó không tạo ra capacity. Trong điều kiện này,
việc tổ chức lại hệ thống có thể trở thành một cách trì hoãn cuộc đối thoại thật (đàm phán bỏ cam kết —
xem mục 1 và mục 3). Dấu hiệu chẩn đoán: sau ba tuần chạy hệ thống nghiêm túc, số việc quá hạn không
giảm mà chỉ trở nên rõ ràng hơn. Đó là dấu hiệu vấn đề nằm ở tầng cam kết, không ở tầng tổ chức.

**Khi tổ chức chưa có bộ nhớ chung.** Nếu team chưa dùng ticket một cách nghiêm túc và chưa viết tài
liệu, việc bạn xây một hệ thống cá nhân xuất sắc sẽ làm bạn trở thành bộ nhớ duy nhất của tổ chức. Bạn
sẽ rất hữu dụng, rất khó thay thế, và không thể nghỉ phép — đó không phải thành công. Thứ tự đúng là
dựng kênh chung trước (ticket có template, một trang trạng thái dự án, Decision Log), rồi hệ thống cá
nhân chỉ chứa phần thực sự riêng của bạn.

**Trong 2–3 tuần đầu ở một tổ chức mới.** Giai đoạn này lượng thông tin vào rất lớn nhưng cấu trúc của
nó chưa rõ, nên mọi cách phân loại bạn dựng lên đều sẽ sai và phải làm lại. Chế độ đúng ở đây là
capture thô, gần như không clarify — một file ghi ngày tháng, ghi tất cả, chấp nhận lộn xộn — rồi sau
2–3 tuần dựng cấu trúc từ thứ đã tích lũy. Cấu trúc rút ra từ dữ liệu thật chịu tải tốt hơn cấu trúc
tưởng tượng trước.

---

## 5. Decision Making ở cấp cá nhân

### Problem Statement

Hai quyết định trong cùng một quý, cùng một người, tại một startup fintech Series A ở TP.HCM.

**Quyết định thứ nhất:** chọn message broker cho luồng thông báo — khối lượng thực tế khoảng 50
message/giây, đỉnh 200. Khoa (Tech Lead) dành 3 tuần: dựng PoC với Kafka, PoC với RabbitMQ, đọc về
Redis Streams, làm một bảng so sánh 14 tiêu chí, tổ chức hai buổi họp 90 phút với team. Kết luận cuối
cùng: RabbitMQ — thứ mà hai người trong team đã dùng ở công ty trước và đã được đề xuất trong 20 phút
đầu của tuần thứ nhất. Trong 3 tuần đó, hai engineer chờ để bắt đầu luồng notification.

**Quyết định thứ hai:** tối trước ngày release, nghiệp vụ cần thêm 6 trường dữ liệu cho bản ghi giao
dịch. Trong stand-up 15 phút, Khoa quyết: nhồi cả 6 trường vào một cột `extra_data` kiểu JSON của bảng
`transactions`, "để sau refactor". Thời gian ra quyết định: khoảng 4 phút.

Tám tháng sau: lựa chọn broker hoàn toàn không quan trọng — với khối lượng đó cả ba phương án đều chạy,
và đã có wrapper nên đổi được trong một sprint nếu cần. Cột `extra_data` thì có 41 triệu dòng, không có
schema, không validate, ba team đọc theo ba cách khác nhau, hai lần sự cố đối soát vì kiểu dữ liệu
không nhất quán, và một dự án migration 7 tuần đang chờ trong roadmap.

Đây là hiện tượng đặc trưng nhất của việc thiếu Decision Making ở cấp cá nhân: **chi phí ra quyết định
được phân bổ ngược với mức độ không đảo được của quyết định.** Vấn đề không phải Khoa suy nghĩ ít hay
nhiều. Vấn đề là không có bước phân loại, nên mức đầu tư được quyết định bởi *cảm giác thú vị* và *áp
lực thời gian tại thời điểm đó* thay vì bởi cấu trúc của quyết định.

Hiện tượng quan sát được, đo được:

- **Thời gian từ khi vấn đề được nêu tới khi có quyết định, tách theo loại.** Nếu quyết định đảo được
  mất trung bình lâu hơn quyết định khó đảo, phân bổ đang ngược.
- **Số quyết định đang "chờ thêm thông tin" quá 2 tuần mà không ai định nghĩa được thông tin nào, ai
  lấy, và bao giờ có.** Đây gần như luôn là trì hoãn được đóng gói bằng ngôn ngữ kỹ thuật.
- **Số người-ngày bị chặn bởi các quyết định chưa ra.** Ở dự án của Khoa: 2 người × 15 ngày = 30
  người-ngày cho một quyết định mà mọi phương án đều chấp nhận được.
- **Số quyết định khó đảo được ra trong cuộc họp không có tài liệu chuẩn bị trước.** Trong fintech và
  ngân hàng, đây là chỉ số rủi ro thật, không phải chỉ số quy trình.
- **Số lần quay lại một quyết định cũ mà không có thông tin mới.** Nếu cùng một chủ đề xuất hiện trong
  ba buổi họp trong hai tháng và không ai có dữ liệu mới, quyết định đó chưa bao giờ thực sự được ra.
- **Tỉ lệ quyết định có ghi lại lý do.** Nếu gần 0, tổ chức không thể phân biệt "quyết định sai" với
  "quyết định đúng gặp kết quả tệ" — và khi không phân biệt được, người ta học sai bài học.

### First Principles

**Cơ chế một: mỗi quyết định có hai loại chi phí, và chúng chạy ngược nhau theo thời gian.**

```
Chi phí kỳ vọng của việc sai  =  P(sai) × Chi phí sửa sai
Chi phí của việc chậm         =  Cost of Delay × thời gian suy nghĩ

Suy nghĩ thêm làm giảm P(sai) — nhưng theo đường lợi ích cận biên giảm dần.
Suy nghĩ thêm làm tăng chi phí chậm — thường theo đường thẳng hoặc dốc hơn.

Điểm tối ưu: nơi lợi ích cận biên của việc suy nghĩ thêm = chi phí cận biên của việc chậm.
```

Hai hệ quả. Thứ nhất, **không tồn tại một mức đầu tư suy nghĩ đúng cho mọi quyết định** — mọi lời khuyên
dạng "hãy quyết nhanh" hoặc "hãy cẩn thận" đều sai một nửa. Thứ hai, biến quan trọng nhất là *chi phí
sửa sai*, vì nó là biến biến động mạnh nhất giữa các quyết định: từ vài phút (đổi tên biến) tới vài
tháng (migration 41 triệu dòng dữ liệu).

**Cơ chế hai: reversibility quyết định hình dạng của chi phí sai — Type 1 và Type 2.** Cách phân loại
này (Jeff Bezos, các thư gửi cổ đông công khai) chia quyết định thành hai loại: **Type 1 — cửa một
chiều**, hệ quả khó hoặc rất đắt để đảo; **Type 2 — cửa hai chiều**, đi qua rồi quay lại được với chi
phí thấp.

Điểm sâu hơn thường bị bỏ: với Type 2, **bản thân quyết định là thí nghiệm rẻ nhất bạn có.** Suy nghĩ
thêm ba tuần để dự đoán kết quả đắt hơn nhiều so với việc thử một tuần và biết chắc. Do đó với Type 2,
việc quyết nhanh không phải là chấp nhận rủi ro — nó là chiến lược thu thập thông tin tối ưu. Ngược lại,
với Type 1, chi phí sai không đối xứng nên bạn phải trả giá cho thông tin trước, kể cả khi đang gấp.

**Cơ chế ba: satisficing vs maximizing.** Herbert Simon (bounded rationality, nguồn công khai) chỉ ra
rằng vì thông tin không đầy đủ và chi phí tìm kiếm là thật, chiến lược hợp lý cho phần lớn quyết định
là **satisficing**: đặt trước một ngưỡng "đủ tốt" và chọn phương án đầu tiên vượt ngưỡng, thay vì tìm
phương án tốt nhất (maximizing). Nghiên cứu công khai sau đó về khuynh hướng maximizing (Schwartz và
đồng nghiệp) cho thấy người maximizing đạt kết quả khách quan tốt hơn một chút nhưng trả bằng chi phí
tìm kiếm cao hơn nhiều và mức hối tiếc cao hơn.

Áp vào công việc kỹ thuật: maximizing đúng cho quyết định Type 1 có blast radius lớn; satisficing đúng
cho phần còn lại — tức là đa số. Và điều cần nhấn: **kỹ năng cần luyện không phải "suy nghĩ giỏi hơn"
mà là "phân loại đúng hơn"**, vì phân loại quyết định bạn dùng chế độ nào.

**Cơ chế bốn: số lượng quyết định là một ràng buộc, không chỉ chất lượng từng quyết định.** Mỗi quyết
định cần suy nghĩ thật tiêu một block attention chất lượng cao, và bạn có rất ít block như vậy mỗi ngày
(mục 2). Cần nói rõ về mức độ chắc chắn của khoa học ở đây: khái niệm "ego depletion" từng được dùng để
giải thích điều này đang bị tranh cãi mạnh trong các nghiên cứu tái lập, nên không nên dựa vào nó. Cơ
chế không tranh cãi thì đơn giản hơn: quyết định cần chất lượng attention, chất lượng attention giảm
trong ngày, do đó **cách khả thi để tăng chất lượng quyết định là giảm số lượng quyết định** — bằng
default và policy — chứ không phải cố nâng năng lực quyết định.

**Cơ chế năm: dưới áp lực thời gian, phán đoán của bạn xấu đi đúng theo cách nguy hiểm nhất.** Khi gấp,
con người mặc định về lối tư duy nhanh và về trường hợp tham chiếu dễ gợi lại nhất (Kahneman, nguồn công
khai). Với Type 2, điều đó chấp nhận được. Với Type 1, đó là nguồn của những khoản nợ kéo dài nhiều năm
— như cột `extra_data`. Hệ quả thiết kế quan trọng: **cổng phân loại phải là một quy tắc máy móc, không
phải một phán đoán**, bởi vì trạng thái mà bạn cần nó nhất (đang gấp) chính là trạng thái mà phán đoán
của bạn kém nhất. Một cổng 90 giây chạy được lúc 23h đêm trước release có giá trị hơn một framework 10
bước chỉ chạy được lúc tỉnh táo.

### Mental Model

**Mô hình 1 — Ma trận Reversibility × Blast radius.**

```
        Khó đảo (Type 1)
              ^
              |  [ RFC, có người phản biện    ] | [ Quyết chậm nhất,      ]
              |  [ được chỉ định, pre-mortem, ] | [ pre-mortem bắt buộc,  ]
              |  [ ADR, 1-2 tuần              ] | [ ADR + Decision Log,   ]
              |  [ VD: contract nội bộ        ] | [ VD: data model, public]
              |  [                            ] | [ API, quyết định nhân sự]
              |------------------------------+------------------------->
              |  [ Quyết một mình, ≤ 10 phút ] | [ Quyết + 1 người review,]
              |  [ VD: tên biến, thứ tự task ] | [ 1 ngày, ghi 1 dòng     ]
              |  [                            ] | [ VD: quy ước code, CI  ]
        Dễ đảo (Type 2)
                 Blast nhỏ            Blast lớn ---->
```

Giá trị của mô hình không nằm ở việc phân loại cho đẹp, mà ở việc **mỗi ô có một quy trình khác nhau và
một người quyết khác nhau**. Sai lệch phổ biến: dùng quy trình của ô trên-phải (RFC, họp, nhiều người)
cho việc ở ô dưới-trái — đây là nguồn của phần lớn sự chậm trễ trong các tổ chức "cẩn thận"; và dùng
quy trình ô dưới-trái cho việc ở ô trên-phải — nguồn của phần lớn nợ kiến trúc trong các tổ chức
"nhanh".

**Mô hình 2 — Danh sách cửa một chiều thật, và danh sách cửa trông như một chiều.**

Đây là mô hình có giá trị thực dụng cao nhất trong mục này, vì nó là thứ bạn cần nhớ khi đang gấp.

| Thực sự khó đảo trong software | Vì sao |
|---|---|
| Data model của dữ liệu đã sinh ra ở production | Đảo được nhưng phải migration, và sai dữ liệu không lấy lại được |
| Contract API có consumer bên ngoài tổ chức | Bạn không kiểm soát được lịch của consumer |
| Cấu trúc phân quyền và ranh giới dữ liệu (multi-tenant, PII) | Sửa sau đồng nghĩa với rủi ro compliance và audit trail |
| Chọn ngôn ngữ / nền tảng cho hệ lõi | Chi phí đảo đo bằng năm-người, và nó khoá cả thị trường tuyển dụng |
| Quyết định về người: tuyển, cho thôi việc, đưa vào PIP | Chi phí sai bất đối xứng và không đảo được về mặt quan hệ |
| Tên (service, endpoint, bảng, sản phẩm) | Rẻ về kỹ thuật, đắt về nhận thức — tên sai tồn tại hàng năm |

| Trông như khó đảo nhưng thường không | Điều kiện để nó dễ đảo |
|---|---|
| Chọn thư viện / framework nội bộ | Có wrapper hoặc lớp adapter, không rò kiểu dữ liệu của nó ra toàn hệ |
| Chọn message broker, cache, search engine | Truy cập qua interface của mình, không dùng tính năng độc quyền |
| Chọn nhà cung cấp hạ tầng cho một thành phần | Có Infrastructure as Code, không dùng service độc quyền cho logic lõi |
| Cấu trúc thư mục, chia module | Refactor được bằng công cụ, không có consumer bên ngoài |

Hệ quả trực tiếp và rất thực dụng: **chi phí để biến một quyết định Type 1 thành Type 2 thường thấp hơn
chi phí để quyết đúng.** Một lớp mapping, một feature flag, một cột đúng kiểu thay vì JSON, một API có
version — mỗi thứ tốn vài ngày công và chuyển toàn bộ quyết định sang ô dưới, nơi bạn được phép quyết
nhanh và sai. Đây là đòn bẩy quan trọng nhất của người ra quyết định kỹ thuật, và nó thường không được
nhìn nhận như một lựa chọn.

**Mô hình 3 — Decision budget.**

Coi mỗi ngày bạn có khoảng 3–5 "khe quyết định" cần suy nghĩ thật. Mọi quyết định vượt số đó sẽ được ra
bằng chế độ nhanh, dù bạn có muốn hay không. Từ đó, việc quản lý quyết định không phải là quyết tốt hơn
mà là **quản lý số lượng**: chuyển việc lặp lại thành default, chuyển việc thuộc vùng người khác thành
delegation, và bảo vệ các khe còn lại cho những quyết định Type 1.

### Practical Framework

**Bước 1 — Cổng phân loại 90 giây, bốn câu hỏi.**

Chạy trước mọi quyết định mà bạn cảm thấy cần suy nghĩ. Bốn câu, trả lời thô, không cần chính xác:

1. **Nếu sai thì sửa mất bao nhiêu?** Đo bằng đơn vị so sánh được: giờ / ngày / tuần / tháng-người.
2. **Cái gì và ai đã phụ thuộc vào quyết định này sau khi nó được thực hiện?** Dữ liệu đã sinh? Team
   khác? Khách hàng? Con số này là blast radius.
3. **Sau bao lâu tôi biết được là mình sai?** Nếu tín hiệu sai đến trong một tuần, rủi ro thấp hơn nhiều
   so với khi tín hiệu chỉ đến sau một năm. Đây là biến hay bị bỏ và nó rất quan trọng: một quyết định
   khó đảo *nhưng có tín hiệu sớm* nên được xử lý gần Type 2 hơn.
4. **Bao nhiêu người đang bị chặn bởi việc chưa quyết?** Đây là Cost of Delay tính bằng người-ngày.

Output: một nhãn và một timebox.

| Loại | Ví dụ | Ai quyết | Timebox | Ghi lại |
|---|---|---|---|---|
| Type 2, blast nhỏ | Tên biến, cấu trúc thư mục, thứ tự task, chọn lib có wrapper | Bạn, một mình | Ngay, tối đa 10 phút | Không |
| Type 2, blast vừa | Quy ước code của team, công cụ CI, cách chia module | Bạn + 1 người review | 1 ngày | Một dòng trong channel |
| Type 1, blast vừa | Thêm bảng/quan hệ vào schema, contract giữa hai service nội bộ | Bạn + Tech Lead / kiến trúc | 2–3 ngày, có pre-mortem | ADR ngắn |
| Type 1, blast lớn | Data model của dữ liệu đã sinh, public API, nền tảng, quyết định nhân sự | Theo RFC, có người phản biện được chỉ định | 1–2 tuần, có ngày chốt cứng | ADR + Decision Log |

Chi tiết về ADR, RFC và Decision Log ở `05-technical-leadership.md` và `04-decision-making.md`. Phần
thuộc mục này chỉ là: **bạn phải biết mình đang ở dòng nào trước khi bắt đầu suy nghĩ.**

**Bước 2 — Timebox cho quyết định reversible, và quy tắc khi hết giờ.**

Đặt ngày *và giờ*: "16h thứ Tư tôi chốt". Quy tắc khi hết giờ mà chưa đủ thông tin — đây là phần làm
timebox có hiệu lực thật:

> Khi hết timebox, chọn **phương án dễ đảo nhất** trong số các phương án đạt ngưỡng, và ghi lại điều
> kiện sẽ xem lại.

Điều này không phải tung xu. Nó là một default có nguyên tắc: trong tình trạng thiếu thông tin, thứ có
giá trị cao nhất không phải phương án tốt nhất mà là **quyền được đổi ý rẻ**. Với quyết định của Khoa
về broker, quy tắc này cho ra câu trả lời trong 2 ngày: chọn thứ team đã biết, đặt sau một interface,
ghi lại điều kiện xem lại (khi vượt 2.000 message/giây hoặc khi cần ordering đảm bảo).

Đặt tiêu chí satisficing **trước** khi xem phương án, không phải sau. Lý do có cơ chế: nếu bạn xem
phương án trước, tiêu chí sẽ bị uốn theo phương án bạn thấy hấp dẫn — đây là hiện tượng đã được mô tả
rộng rãi và nó xảy ra ngoài ý thức. Ví dụ tiêu chí cho quyết định broker: (a) chịu được 10 lần khối
lượng hiện tại, (b) có người trong team đã vận hành production, (c) có managed service ở nhà cung cấp
cloud hiện tại. Phương án đầu tiên đạt cả ba thì chọn, dừng tìm.

**Bước 3 — Pre-mortem cá nhân, 10 phút, chỉ cho Type 1.**

Ba câu hỏi, viết ra, không làm trong đầu:

1. **Sáu tháng sau, quyết định này đã trở thành một thất bại. Kể lại câu chuyện đó.** Viết dạng tường
   thuật, không dạng danh sách rủi ro. Lý do: dạng tường thuật buộc bạn nối các nguyên nhân với nhau và
   thường lộ ra chuỗi mà danh sách rủi ro bỏ sót.
2. **Dấu hiệu sớm nhất tôi sẽ nhìn thấy là gì, và tôi đo nó bằng gì?** Output phải là một chỉ số hoặc
   một sự kiện quan sát được, kèm ngày kiểm tra. Nếu không viết được, bạn đang ra một quyết định mà bạn
   sẽ không bao giờ biết là sai.
3. **Nếu phải đảo thì tôi đảo bằng cách nào, và mất bao nhiêu?**

Câu 3 có tỉ lệ giá trị/chi phí cao nhất. Trong thực tế, nó thường phát hiện ra rằng quyết định *có thể*
đảo được nếu bây giờ ta thêm một thứ nhỏ — và khi đó bạn đã chuyển bài toán sang ô dễ hơn thay vì phải
giải nó cho đúng.

**Bước 4 — Chuyển quyết định lặp lại thành default.**

Quy tắc kích hoạt: **khi bạn quyết cùng một loại việc lần thứ ba, viết nó thành một default.** Default
là một câu có dạng "trong điều kiện X thì làm Y, trừ khi Z". Ví dụ một bộ default của một Tech Lead:

- PR dưới 250 dòng, có test, không đổi contract công khai → approve nếu không có vấn đề về ranh giới
  module; không tranh luận về style (đã có linter).
- Mọi thư viện bên thứ ba mới → đi qua một lớp adapter của mình, không import trực tiếp ở tầng domain.
- Mọi câu hỏi không chặn ai → trả lời trong cửa sổ 14h–15h.
- Mọi thay đổi schema có dữ liệu production → cần một ADR ngắn và một người review, không quyết trong
  stand-up.
- Đề nghị thêm việc vào sprint đang chạy → luôn trả lời bằng câu hỏi "bỏ việc nào ra" thay vì yes/no.

Lợi ích không phải là tiết kiệm 5 phút mỗi lần. Lợi ích là **các khe quyết định được giải phóng cho việc
Type 1**, và team học được cách bạn quyết nên họ tự dự đoán được — điều này giảm số câu hỏi gửi tới bạn.

**Bước 5 — Ghi tối thiểu năm dòng cho mọi quyết định Type 1.**

```
Quyết định  : ...
Thay cho    : (các phương án đã loại và lý do một dòng)
Giả định    : (điều gì phải đúng để quyết định này đúng)
Xem lại khi : (điều kiện hoặc ngày cụ thể)
Ai ảnh hưởng: (team/hệ thống nào cần biết)
```

Lý do dòng "Giả định" là dòng quan trọng nhất: nó cho phép sau này phân biệt hai trường hợp hoàn toàn
khác nhau — quyết định sai (giả định sai ngay từ đầu, đáng lẽ phải phát hiện) và quyết định đúng gặp kết
quả tệ (giả định đúng lúc đó, hoàn cảnh đổi). Tổ chức không phân biệt được hai thứ này sẽ dạy người ta
tránh mọi quyết định có rủi ro, kể cả khi rủi ro đó là đúng để nhận.

### Trade-off

| Trục | Nghiêng về bên A khi | Nghiêng về bên B khi | Ai chịu phần mất |
|---|---|---|---|
| **A: Quyết nhanh — B: Quyết chậm và kỹ** | Type 2; có người đang bị chặn; tín hiệu sai đến trong vòng 1–2 tuần | Type 1; dữ liệu production sẽ được sinh ra; có consumer bên ngoài; tín hiệu sai chỉ đến sau nhiều tháng | Nhanh ở Type 1: tổ chức trả trong nhiều quý, và người trả thường không phải người quyết. Chậm ở Type 2: người bị chặn trả, và chi phí này vô hình vì không ai gửi hoá đơn |
| **A: Tự quyết — B: Xin đồng thuận rộng** | Bạn có thông tin và quyền; quyết định nằm trong vùng của bạn; việc đảo được | Quyết định ảnh hưởng cách làm việc của nhiều người và cần họ thực thi tự nguyện; hoặc bạn thiếu bối cảnh kinh doanh | Đồng thuận rộng: accountability bị mờ — khi sai không ai chịu; và người phản đối muộn nhất thắng. Tự quyết khi cần buy-in: quyết định đúng bị thực thi nửa vời |
| **A: Satisficing — B: Maximizing** | Nhiều phương án đều đạt ngưỡng; chi phí tìm kiếm cao; quyết định đảo được | Chi phí sai bất đối xứng (compliance, dữ liệu tài chính, an toàn); quyết định khoá nhiều năm | Satisficing ở chỗ cần maximizing: nợ kéo dài. Maximizing ở chỗ cần satisficing: 30 người-ngày cho một lựa chọn không đổi kết quả |
| **A: Quyết bây giờ với thông tin ít — B: Chờ thêm thông tin** | Bạn không nêu được thông tin nào sẽ đến, từ đâu, khi nào | Bạn nêu được cụ thể: "load test thứ Năm sẽ cho biết", "khách xác nhận volume trước 15/4" | Chờ mà không định nghĩa được thông tin: đây là trì hoãn, và nó tiêu chi phí trễ mỗi ngày mà không mua được gì |

Trục thứ tư là bài kiểm tra thẳng thắn nhất để phát hiện analysis paralysis ở chính mình. Việc chờ chỉ
có giá trị khi nó mua được thông tin — tức là **option value**. Nếu bạn không viết ra được thông tin nào
sẽ đến, ai tạo ra nó và ngày nào, thì việc chờ không mua được gì; nó chỉ hoãn cảm giác chịu trách nhiệm.
Câu hỏi tự kiểm tra một dòng: "nếu ba ngày nữa tôi vẫn có đúng thông tin như hôm nay, tôi sẽ quyết
khác không?" Nếu không, quyết bây giờ.

Trục thứ hai cần một ghi chú riêng cho bối cảnh Việt Nam. Việc đi hỏi ý kiến nhiều người có **chi phí xã
hội thấp** và trông giống một hành vi tốt: cầu thị, tôn trọng, không tự cao. Nhưng cơ chế nó tạo ra thì
không tốt: khi 8 người đã "được hỏi ý", không ai trong 9 người thực sự chịu trách nhiệm cho kết quả, và
sau khi có hậu quả thì lịch sử được kể lại theo hướng có lợi cho người kể. Cách giữ được mặt tốt của
hành vi này mà không mất accountability là **nói rõ chế độ trước khi hỏi**: "em hỏi ý anh để lấy thông
tin, quyết định cuối cùng là của em" (consult) hoàn toàn khác với "mình cùng quyết" (consensus). Một câu
nói tại đầu cuộc trò chuyện giải quyết được vấn đề này.

### Real-world Scenarios

**Tình huống A — Analysis paralysis, và chi phí không ai gửi hoá đơn.**

Bối cảnh: chính quyết định broker của Khoa. Ba tuần, 30 người-ngày bị chặn, và kết luận trùng với đề
xuất ở ngày thứ hai.

Điều làm tình huống này khó nhận ra từ bên trong: mọi hành động của Khoa đều *trông* đúng đắn. Dựng PoC
là hành vi kỹ thuật nghiêm túc. So sánh 14 tiêu chí là sự cẩn thận. Không ai trong tổ chức phê phán
Khoa, và trong buổi họp quý anh còn được ghi nhận vì "làm việc có phương pháp". Chi phí 30 người-ngày
không xuất hiện trong bất kỳ báo cáo nào, vì nó là chi phí cơ hội — dạng chi phí duy nhất mà tổ chức
không có sổ để ghi.

Xử lý đúng bằng cổng phân loại, mất 90 giây: (1) sai thì sửa mất bao nhiêu — có wrapper thì khoảng một
sprint; (2) cái gì phụ thuộc — chỉ luồng notification, không có dữ liệu bền, không có consumer ngoài;
(3) biết sai sau bao lâu — dưới một tháng, khi có số liệu vận hành thật; (4) ai đang bị chặn — hai
người. Kết luận: Type 2, blast vừa. Timebox 2 ngày, tiêu chí satisficing đặt trước, chọn phương án đầu
tiên đạt ngưỡng, ghi ba dòng.

Và điều kiện mà Khoa **đúng** khi chi 3 tuần: nếu broker này là xương sống của luồng thanh toán, có yêu
cầu về thứ tự và về đảm bảo không mất message, có dữ liệu tài chính đi qua, và các service của đối tác
sẽ tích hợp trực tiếp vào nó. Khi đó nó là Type 1 blast lớn, và 3 tuần với một RFC là đầu tư hợp lý. Cùng
một công nghệ, cùng một cách chọn, nhưng cấu trúc quyết định khác nhau hoàn toàn — đó là toàn bộ ý của
mục này.

**Tình huống B — Quyết định khó đảo theo cảm tính vì đang gấp.**

Bối cảnh: cột `extra_data` kiểu JSON, quyết trong 4 phút vào tối trước release.

Điều cần nói rõ: **quyết định nhanh không phải vấn đề. Vấn đề là quyết định nhanh mà không phân loại.**
Khoa không có 3 ngày vào tối đó — ràng buộc thời gian là thật. Nhưng anh có 90 giây, và 90 giây đó sẽ
cho ra: sửa mất bao nhiêu — migration dữ liệu production, đo bằng tuần-người; cái gì phụ thuộc — dữ
liệu giao dịch, tức là dữ liệu bền và có audit; biết sai sau bao lâu — nhiều tháng. Kết luận: Type 1,
blast lớn, trong điều kiện không có thời gian.

Trong tình huống Type 1 mà không có thời gian, hành động đúng gần như luôn là **chọn phương án bảo toàn
khả năng đảo, không phải chọn phương án tốt nhất.** Ba phương án loại đó, đều làm được trong tối hôm ấy:

- Thêm 6 cột đúng kiểu dữ liệu, nullable, không có ràng buộc phức tạp. Thêm 2–3 giờ công, không cần
  thiết kế hoàn hảo, và quan trọng nhất: dữ liệu sinh ra vẫn có schema nên tương lai đảo được rẻ.
- Ship release mà không có 6 trường đó, bù bằng bản 1.1 trong tuần sau. Đàm phán được nếu nêu rõ chi phí
  của phương án nhanh.
- Nếu buộc phải dùng JSON: kèm một validation schema ở tầng ứng dụng ngay từ commit đầu tiên, cộng một
  dòng trong Decision Log với ngày xem lại là 30 ngày. Đây là phương án tệ nhất trong ba nhưng vẫn tốt
  hơn nhiều so với những gì đã xảy ra, vì nó ngăn được phần đắt nhất: **dữ liệu không nhất quán về
  kiểu**, thứ mà không migration nào sửa lại được hoàn toàn.

**Tình huống C — Bị ép chốt ngay một quyết định Type 1 trong cuộc họp.**

Bối cảnh: startup fintech, CTO cũng là co-founder, quen ra quyết định nhanh và không thích nghe "cần
thêm thời gian". Bạn là Tech Lead, đang bàn thiết kế cho một hệ thống ví mới.

> **CTO:** Thôi chốt luôn đi, mai team bắt đầu làm. Mình không có thời gian bàn tiếp.
>
> **Bạn:** Em chốt được ngay phần lớn ạ. Kiến trúc luồng và cách chia service em chốt theo phương án B,
> em chịu trách nhiệm phần đó. Còn đúng một thứ em xin thêm thời gian: cấu trúc bảng `wallet_transaction`.
>
> **CTO:** Có gì mà phải nghĩ, làm như bên kia mình làm hồi trước.
>
> **Bạn:** Lý do em xin thêm thời gian cụ thể là thế này: bảng đó khi đã sinh dữ liệu thật thì đổi phải
> migration, và lần trước ở dự án đối soát mình mất 3 tuần cho đúng việc đó. Hai ngày nghĩ bây giờ so
> với 3 tuần sửa sau — em thấy đáng. Và hai ngày này không chặn ai: team làm phần API, phần UI và phần
> tích hợp KYC trước, đủ việc cho cả tuần.
>
> **CTO:** Hai ngày lâu đấy. Anh muốn tuần này chốt hết.
>
> **Bạn:** Vậy em rút xuống một ngày — chiều mai 4h em có câu trả lời kèm một trang lý do. Nhưng em muốn
> nói rõ một điều để anh chọn: nếu anh cần em chốt ngay bây giờ, em sẽ không chọn phương án tốt nhất, em
> sẽ chọn phương án giữ được khả năng đổi — tức là thêm một lớp mapping và một bảng phụ. Cái đó tốn thêm
> khoảng 2 ngày công của team và làm code phức tạp hơn một chút. Nên anh chọn giúp em: một ngày để em
> nghĩ, hay hai ngày công để làm lớp đệm. Em làm theo cách nào cũng được.
>
> **CTO:** Thì lấy một ngày.
>
> **Bạn:** Vâng. Chiều mai 4h em gửi anh. Em sẽ ghi lại giả định của quyết định này và điều kiện xem lại,
> để nếu sau này volume khác dự kiến thì mình biết phải xem lại chỗ nào chứ không phải đi tìm.

Bốn kỹ thuật cần thấy: (1) **tách phần chốt được ra khỏi phần không chốt được** — điều này loại bỏ hoàn
toàn ấn tượng "cấp dưới đang trì hoãn", vốn là rủi ro thật khi phản biện cấp trên trong môi trường Việt
Nam; (2) nêu chi phí bằng đơn vị so sánh được, có tiền lệ cụ thể trong chính tổ chức (3 tuần ở dự án đối
soát), thay vì lập luận nguyên tắc; (3) chứng minh việc chờ không chặn ai — đây là câu trả lời cho phản
đối thật của CTO, vốn là về tốc độ chứ không về nội dung; (4) **chuyển từ xin phép sang đưa hai lựa chọn
có giá rõ ràng**, cả hai đều là "em làm được". Kỹ thuật thứ tư là kỹ thuật quan trọng nhất: cấp trên
không phải chọn giữa đồng ý và không đồng ý với bạn, mà chọn giữa hai loại chi phí — và trong tình huống
đó họ hầu như luôn chọn phương án rẻ hơn, tức là phương án bạn muốn.

### Best Practices

**Phân loại trước khi suy nghĩ, luôn luôn theo thứ tự đó.** Lý do: mức đầu tư đúng là hàm của cấu trúc
quyết định, không của cảm giác về nó. Bỏ bước phân loại nghĩa là để mức đầu tư được quyết bởi mức độ thú
vị của bài toán và mức độ áp lực tại thời điểm đó — hai biến không tương quan với giá trị.

**Đặt tiêu chí "đủ tốt" trước khi xem phương án.** Lý do: tiêu chí đặt sau sẽ bị uốn theo phương án hấp
dẫn, ngoài ý thức. Đặt trước cũng cho bạn điều kiện dừng — thứ mà maximizing không có, và thiếu điều
kiện dừng là cơ chế thật của analysis paralysis.

**Chi tiền để biến Type 1 thành Type 2 trước khi chi tiền để quyết đúng.** Lý do: chi phí của một lớp
adapter, một feature flag, một API có version, một cột đúng kiểu — gần như luôn nhỏ hơn chi phí của việc
suy nghĩ đủ để đúng ngay lần đầu, và nó còn có giá trị ngay cả khi bạn đã đúng.

**Khi hết timebox, chọn phương án dễ đảo nhất trong nhóm đạt ngưỡng và ghi điều kiện xem lại.** Lý do:
biến "quyết định chưa xong" — thứ tiêu attention vô hạn và chặn người khác — thành "quyết định đã ra với
một cửa mở". Chi phí là có thể phải quyết lại một lần; lợi ích là hệ thống tiếp tục chạy.

**Nói rõ chế độ trước khi hỏi ý kiến: consult hay consensus.** Lý do: giữ được lợi ích của việc lấy
thông tin và của việc tôn trọng người khác, mà không làm mờ accountability. Một câu ở đầu cuộc trò
chuyện thay thế được cả một cuộc tranh luận sau khi có hậu quả.

**Với quyết định gây tranh luận, tự viết ra lập luận mạnh nhất của phía đối lập trước khi quyết.** Lý do:
bạn không thể phát hiện điểm yếu trong lập luận của mình bằng cách xem lại lập luận của mình. Viết
steelman cho phía kia là cách rẻ nhất để mô phỏng một người phản biện khi không có ai đủ tự tin phản
biện bạn — tình huống rất phổ biến khi bạn là người có thâm niên cao nhất trong phòng.

**Ghi lại giả định, không chỉ ghi quyết định.** Lý do: quyết định không có giả định thì không kiểm chứng
được, và tổ chức sẽ đánh giá nó bằng kết quả. Đánh giá bằng kết quả dạy người ta chọn việc an toàn.

### Anti-patterns

**Analysis paralysis với vỏ ngoài kỹ thuật.**
Cơ chế: chi phí trễ là chi phí cơ hội, và tổ chức không có sổ nào ghi chi phí cơ hội. Vì vậy việc trì
hoãn quyết định không bao giờ bị phản đối, trong khi một quyết định sai thì bị nhìn thấy ngay. Cấu trúc
khuyến khích này tạo ra hành vi trì hoãn ở chính những người cẩn thận nhất — và PoC là hình thức trì
hoãn hoàn hảo vì nó tạo ra artifact.
Dấu hiệu sớm: bạn đang dựng PoC thứ ba; câu "cần thêm dữ liệu" mà không nói được dữ liệu nào, ai lấy,
ngày nào; số tiêu chí so sánh vượt 8; bạn cảm thấy nhẹ nhõm khi buổi họp kết thúc mà chưa phải chốt.

**Quyết định khó đảo theo cảm tính vì đang gấp.**
Cơ chế: dưới áp lực, bạn dùng trường hợp tham chiếu dễ gợi lại nhất và tối ưu cho ràng buộc hiển thị
nhất (ngày release). Chi phí rơi vào một khoảng thời gian mà bạn không còn liên hệ nhân quả với quyết
định đó — thường 6–18 tháng sau, thường do người khác trả.
Dấu hiệu sớm: câu "để sau refactor" xuất hiện trong lúc quyết định về dữ liệu; quyết định về schema
được ra trong stand-up; không ai trong phòng nói được sửa cái này sau sẽ mất bao lâu.

**Quyết định bằng cách không quyết ("để hôm sau bàn tiếp") lặp lại nhiều lần.**
Cơ chế: việc không quyết vẫn tạo ra một kết quả — kết quả mặc định, thường là hiện trạng — nhưng không
có ai chịu trách nhiệm cho nó và không có ai ghi lại. Đây là dạng quyết định tệ nhất về mặt học hỏi: hệ
thống có hệ quả mà không có tác giả.
Dấu hiệu sớm: cùng một chủ đề xuất hiện trong ba buổi họp mà không có thông tin mới giữa các buổi;
không ai được gán vai người quyết cho chủ đề đó.

**Consensus giả — hỏi ý tám người để không phải chịu trách nhiệm một mình.**
Cơ chế: accountability bị pha loãng tới mức bằng không, và quyết định trở thành hàm của thứ bậc và của
người phản đối muộn nhất. Trong bối cảnh Việt Nam, anti-pattern này đặc biệt khó nhận ra vì hành vi bề
mặt trùng với một đức tính được tôn trọng.
Dấu hiệu sớm: không ai trả lời được "ai là người quyết việc này"; sau khi có hậu quả, mọi người đều
nhớ mình đã có ý kiến khác; các cuộc họp quyết định có hơn 7 người mà không có chủ trì.

**Đánh giá chất lượng quyết định bằng kết quả (outcome bias).**
Cơ chế: trong hệ thống có nhiễu và độ trễ, một quyết định đúng vẫn có thể cho kết quả tệ và ngược lại.
Khi tổ chức đánh giá bằng kết quả, hành vi tối ưu của cá nhân là chọn việc có phương sai thấp — tức là
tránh đúng những quyết định có giá trị kỳ vọng cao. Sau hai năm, tổ chức có một tập người cẩn thận và
không ai đề xuất gì đáng kể.
Dấu hiệu sớm: postmortem tập trung vào "ai quyết cái này" thay vì "với thông tin lúc đó thì quyết định
này hợp lý không"; không ai ghi giả định nên không có cách nào phân biệt hai loại sai.

**Quyết theo thứ vừa đọc (recency và availability).**
Cơ chế: một bài viết hay hoặc một hội thảo tạo ra tính khả dụng cao trong bộ nhớ, và bộ nhớ dễ gợi lại
bị nhầm với xác suất phù hợp cao. Ở cấp lead, tác hại được nhân lên vì bạn có quyền áp đặt.
Dấu hiệu sớm: giải pháp xuất hiện trước khi bài toán được phát biểu; lý do chọn có dạng "công ty X dùng
cái này"; không ai nói được vấn đề hiện tại nào sẽ mất đi.

### Khi nào KHÔNG nên áp dụng

**Trong incident đang diễn ra.** Không chạy cổng phân loại cho từng quyết định trong incident. Chế độ
đúng là một người chỉ huy (Incident Commander) quyết theo thứ tự cố định, và **hầu hết quyết định trong
incident được đối xử như Type 2 kể cả khi chúng không phải**, vì chi phí trễ đang tăng theo thời gian và
tính mạng của hệ thống đến trước tính đúng của thiết kế. Điều kiện đi kèm không được bỏ: mọi quyết định
Type 1 đã ra trong incident phải được ghi vào danh sách xem lại trong postmortem — nếu không, các phương
án chữa cháy sẽ trở thành kiến trúc vĩnh viễn. Xem `06-incident-va-metrics.md`.

**Với quyết định về con người.** Đừng áp timebox kiểu Type 2 cho quyết định tuyển, cho thôi việc, đưa vào
PIP, hoặc đổi vai của một người, kể cả khi bạn rất muốn xong nhanh và kể cả khi mọi dấu hiệu đã rõ. Cơ
chế: chi phí sai bất đối xứng mạnh và phần không đảo được nằm ở quan hệ và ở tín hiệu gửi tới cả team,
chứ không nằm ở giấy tờ. Thêm nữa, tín hiệu về việc quyết định đúng hay sai đến rất muộn. Loại quyết
định này thuộc `08-hiring-va-phat-trien.md`.

**Khi bạn không phải người có quyền quyết.** Chi phí lớn nhất trong tình huống này không phải là quyết
sai — mà là bạn tiêu các khe quyết định của mình để suy nghĩ hộ một việc không có đường nối tới quyết
định thật. Việc đúng cần làm: chuyển suy nghĩ đó thành **input dạng viết** (một trang: phương án, đánh
đổi, ngưỡng bạn sẽ lo lắng), gửi cho người quyết, rồi đóng vòng lặp trong đầu mình lại. Nếu bạn thấy
mình vẫn nghĩ về nó sau khi đã gửi, đó là ownership theo cảm xúc (mục 1), không phải trách nhiệm.

**Khi tổ chức chưa có chỗ ghi quyết định.** Đừng bắt đầu bằng việc áp quy trình ADR đầy đủ với template
12 mục — nó sẽ bị bỏ sau bốn tuần và sau đó rất khó khởi động lại, vì mọi người đã có trải nghiệm rằng
"cái đó không chạy được ở đây". Bắt đầu bằng một file duy nhất, năm dòng mỗi quyết định, chỉ cho quyết
định Type 1. Khi file đó được người khác đọc và dẫn lại trong tranh luận, bạn đã có bằng chứng để mở
rộng.

**Khi quyết định thực chất là quyết định chiến lược của tổ chức nhưng bị đẩy xuống cho bạn dưới vỏ kỹ
thuật.** Ví dụ điển hình: "em chọn build hay mua giải pháp đối soát" khi chưa có ai quyết về ngân sách,
về việc tổ chức có muốn sở hữu năng lực đó lâu dài hay không, và về việc ai sẽ vận hành nó trong 3 năm.
Trong tình huống này, việc bạn ra một quyết định kỹ thuật xuất sắc vẫn dẫn tới kết quả tệ, vì hàm mục
tiêu chưa được xác định. Hành động đúng là trả nó lên với hai phương án và hệ quả của mỗi phương án theo
đúng ngôn ngữ mà người quyết dùng (chi phí, rủi ro, phụ thuộc, năng lực cần có) — kỹ thuật này ở
`09-to-chuc-va-scaling.md`. Tự quyết trong trường hợp này là nhận ownership giả (mục 1).

**Khi chi phí của việc phân loại vượt chi phí của quyết định.** Với những việc rất nhỏ mà bạn ra hàng
chục lần mỗi ngày, chạy cổng 90 giây là chi phí thuần. Đó là lý do bước 4 (default) tồn tại: đích đến
đúng của một người ra quyết định thành thục không phải là chạy framework nhiều lần hơn, mà là **có ít
quyết định cần chạy framework hơn**.

---

## 6. Continuous Learning có chủ đích

### Problem Statement

Tuấn có 9 năm kinh nghiệm, CV liệt kê 6 dự án ở ba công ty ODC, title hiện tại Senior Backend Engineer,
mức lương thuộc nhóm trên của thị trường. Trong một buổi phỏng vấn cho vị trí Tech Lead ở một công ty
product, Tuấn được hỏi: hệ thống có hai service cùng ghi vào một bảng số dư, làm sao đảm bảo không âm
số dư khi có hai request đồng thời. Tuấn trả lời "dùng transaction". Người phỏng vấn hỏi tiếp về isolation
level, về việc transaction ở hai service khác nhau, về đánh đổi giữa pessimistic lock và một hàng đợi
tuần tự hoá. Tuấn không có khung tư duy nào cho loại câu hỏi đó.

Điều này không phải vì Tuấn kém. Rà lại 6 dự án: cả 6 đều là hệ thống CRUD trên Spring Boot với
PostgreSQL, cùng một tầng kiến trúc, cùng loại vấn đề, khối lượng dữ liệu trong ngưỡng mà mọi thứ đều
chạy được. Tuấn đã thực hành rất nhiều — cùng một thứ, 9 lần. Đây là hình thái đặc trưng của **"10 năm
kinh nghiệm" thực chất là 1 năm lặp 10 lần**, và nó xảy ra do cấu trúc công việc chứ không do thiếu nỗ
lực.

Hiện tượng quan sát được khi Continuous Learning không có chủ đích:

- **Số *loại* vấn đề mới thực sự gặp trong 12 tháng qua.** Chú ý: loại vấn đề, không phải số công nghệ.
  Học Next.js sau khi đã dùng React không phải một loại vấn đề mới; lần đầu phải xử lý một hệ thống có
  ràng buộc consistency giữa hai nguồn dữ liệu là một loại vấn đề mới. Nếu con số này là 0–1 trong 12
  tháng, năng lực không tăng bất kể số giờ làm.
- **Độ trễ và độ nhiễu của vòng feedback.** Câu hỏi cụ thể: bạn biết mình quyết định sai sau bao lâu, và
  bằng cách nào? Nếu câu trả lời là "thường tôi không biết", vòng học không tồn tại.
- **Tỉ lệ thời gian làm việc trong vùng đã thành thạo.** Ở mức trên 90% trong nhiều quý, bạn đang được
  trả tiền để lặp lại, và điều đó có thể hoàn toàn ổn với tổ chức nhưng là dữ liệu về bạn.
- **Tần suất bạn thấy điều mình không nghĩ tới khi đọc code người khác.** Nếu gần như không bao giờ, hai
  khả năng: bạn chỉ đọc code của người kém hơn, hoặc bạn đọc mà không so sánh với cách mình sẽ làm.
- **Số lần bạn đổi ý về một quan điểm kỹ thuật trong 12 tháng qua.** Con số 0 là dấu hiệu xấu — không
  phải vì bạn cần dao động, mà vì một người đang học sẽ gặp bằng chứng ngược với niềm tin cũ.
- **Điều bạn học được từ incident gần nhất.** Nếu bài học nằm ở dạng "cần cẩn thận hơn" hoặc "cần thêm
  test", vòng học đã dừng ở tầng bề mặt và mô hình sai trong đầu bạn vẫn nguyên vẹn.

Hệ quả ở cấp cá nhân trong bối cảnh thị trường Việt Nam có một hình dạng đặc trưng: lương tăng theo thị
trường và theo số lần đổi việc, trong khi năng lực tăng chậm hơn. Khi thị trường nóng, khoảng cách này
không hiển thị. Khi thị trường nguội hoặc khi bạn cần bước lên Staff/Principal/EM, khoảng cách hiện ra
đột ngột — vì các vai đó được đánh giá bằng năng lực phán đoán, thứ không mua được bằng số năm.

Hệ quả ở cấp lead nghiêm trọng hơn và ít được nói: một Tech Lead không giữ được năng lực học sẽ mất khả
năng **đánh giá phương án mà mình chưa từng làm**. Từ đó chỉ còn hai chế độ quyết định — chọn theo hype
hoặc chọn theo cái mình biết — và cả hai đều là quyết định không có cơ sở, chỉ khác nhau ở hướng lệch.

### First Principles

**Cơ chế một: deliberate practice cần ba điều kiện, và công việc hàng ngày thường vi phạm cả ba.** Dòng
nghiên cứu về deliberate practice (Ericsson, nguồn công khai) nêu ba điều kiện: nhiệm vụ ở mức khó vừa
vượt năng lực hiện tại, feedback cụ thể và nhanh, và lặp lại có sửa chữa. Cần trung thực về mức độ chắc
chắn: các phân tích tổng hợp sau đó (Macnamara và đồng nghiệp, nguồn công khai) cho thấy lượng thực hành
chỉ giải thích được một phần biến thiên về hiệu suất, thấp hơn nhiều so với cách khái niệm này được phổ
biến. Nhưng ba điều kiện đó vẫn là mô tả tốt về *khi nào* thực hành tạo ra tiến bộ, và điều đó đủ để
dùng.

Đối chiếu với công việc kỹ thuật thông thường: nhiệm vụ thường nằm *trong* năng lực hiện tại (vì tổ chức
giao việc theo năng lực đã chứng minh); feedback về quyết định thiết kế đến sau nhiều tháng, qua nhiều
tầng nhiễu, và thường qua miệng người khác; và gần như không có lặp lại có sửa chữa — bạn không bao giờ
thiết kế lại cùng một hệ thống lần thứ hai với thông tin của lần thứ nhất. Ba vi phạm này giải thích vì
sao thời gian ở công việc và năng lực không tương quan chặt.

**Cơ chế hai: khi vòng feedback dài hơn nhiệm kỳ của bạn, bạn không nhận được tín hiệu.** Hệ quả của một
quyết định kiến trúc thường hiện ra sau 12–36 tháng. Nếu bạn đổi dự án mỗi 12 tháng hoặc đổi công ty mỗi
18 tháng, bạn hệ thống hoá việc rời đi trước khi hoá đơn tới. Đây là một mô tả cơ chế, không phải một lời
phê phán đạo đức — và nó dẫn tới hai đường ra cụ thể: **ở lại đủ lâu để thấy hoá đơn của chính mình**,
hoặc **đi tìm hoá đơn của người khác** (đọc postmortem, đọc code legacy cùng với lịch sử git và các ADR
đã có, nói chuyện với người đã vận hành hệ thống 3 năm). Đường thứ hai rẻ hơn nhiều và bị dùng ít hơn
nhiều.

**Cơ chế ba: desirable difficulty — cảm giác dễ hiểu không phải tín hiệu của việc học được.** Dòng nghiên
cứu công khai của Robert Bjork về "desirable difficulties" cho thấy các điều kiện làm việc học *cảm thấy*
khó hơn (tự gợi lại thay vì đọc lại, giãn cách thay vì dồn, xen kẽ nhiều loại bài thay vì làm khối) cho
kết quả giữ được lâu hơn và chuyển giao tốt hơn. Hệ quả thực dụng, ngược trực giác: đọc một bài viết hay
và cảm thấy "hiểu rồi" là tín hiệu yếu; **giải thích lại cho người khác, dựng lại từ trí nhớ, hoặc dự
đoán trước khi xem đáp án** mới là học. Đây là lý do một buổi share 20 phút cho team tạo ra nhiều năng
lực hơn 10 giờ đọc tài liệu.

**Cơ chế bốn: kiến thức có chu kỳ bán rã khác nhau, nên ROI của việc học khác nhau theo tầng.**

```
Chu kỳ bán rã dài (10+ năm) — nền
   Mạng, hệ điều hành, đồng thời/song song, mô hình dữ liệu, hệ phân tán,
   xác suất và thống kê cơ bản, kinh tế học của hệ thống (chi phí, hàng đợi, độ trễ)
   → học một lần, dùng qua mọi công nghệ. Chuyển giao rất tốt.

Chu kỳ bán rã trung (5-8 năm) — mẫu và kiến trúc
   Mẫu tích hợp, mẫu dữ liệu, cách phân rã hệ thống, cách thiết kế API,
   cách đọc trade-off của một công nghệ mới
   → chuyển giao tốt trong cùng lớp bài toán.

Chu kỳ bán rã ngắn (2-4 năm) — công cụ
   Framework, thư viện, cú pháp, dịch vụ cụ thể của một nhà cung cấp cloud
   → giá trị thực dụng cao ngay, chuyển giao kém. Học nhanh khi cần.
```

Đây là cơ chế đứng sau nhận xét rằng học framework mới nhất có ROI thấp hơn nhìn bề ngoài. Không phải vì
nó vô ích — mà vì nó khấu hao nhanh, trong khi tầng nền thì không. Điểm quan trọng: người đã có tầng nền
vững học một framework mới trong vài ngày; người chỉ có tầng công cụ phải học lại gần như từ đầu mỗi lần
công nghệ đổi.

**Cơ chế năm: throughput của việc học đo bằng số vòng thực hành có feedback, không bằng lượng tiêu thụ.**
Số trang đọc, số video xem, số khoá học mua đều là chỉ số nạp vào. Đầu ra chỉ xuất hiện khi có một vòng
kín: làm → nhận feedback → sửa. Từ đó suy ra một learning backlog 30 mục là một danh sách mong muốn,
không phải một kế hoạch — vì throughput thực tế của một người đang làm việc toàn thời gian là khoảng
một vòng có ý nghĩa mỗi 2–6 tuần.

### Mental Model

**Mô hình 1 — Bản đồ T-shaped, đọc theo hai loại năng lực khác nhau.**

Sự phân biệt cần thiết mà mô hình chữ T thường bị nói mờ: chiều rộng và chiều sâu không phải hai mức của
cùng một thứ, mà là **hai loại năng lực có tiêu chí khác nhau**.

```
Chiều rộng — tiêu chí: đủ để ĐÁNH GIÁ và ĐỐI THOẠI
   Biết vấn đề gì thuộc lĩnh vực đó, đọc được trade-off, biết câu hỏi nào cần hỏi,
   biết ai là người cần gọi. KHÔNG cần làm được.

Chiều sâu — tiêu chí: đủ để LÀM và DẠY
   Debug được lúc 2h sáng khi không có tài liệu, giải thích được vì sao nó hoạt động,
   dự đoán được nó hỏng ở đâu, dạy được người khác.
```

Suy ra hệ quả về đầu tư theo vai: ở vai Tech Lead, giá trị cận biên của chiều rộng cao hơn — bạn phải
đánh giá phương án ở nhiều lĩnh vực bạn không tự làm. Ở vai Staff/Principal, cần **ít nhất một chiều sâu
đủ để tổ chức tin**, vì thẩm quyền kỹ thuật ở vai đó không đến từ title. Ở vai EM, chiều rộng kỹ thuật
phải đủ để không bị qua mặt và để tuyển đúng, còn chiều sâu chuyển sang các lĩnh vực khác (con người,
kế hoạch, ngân sách).

**Mô hình 2 — Bốn tầng feedback theo độ trễ, và chiến lược kéo feedback về tầng nhanh.**

```
compiler / test          → giây      (feedback rất chặt, rất rẻ)
code review              → giờ-ngày  (chặt vừa, phụ thuộc chất lượng reviewer)
production / incident    → tuần-tháng (rất thật, rất đắt)
hệ quả kiến trúc         → quý-năm   (thật nhất, và thường không đến tay bạn)
```

Kỹ năng học nhanh, nói bằng một câu, là: **kéo feedback từ tầng chậm về tầng nhanh bằng cách tạo dự đoán
trước.** Ví dụ cụ thể và rẻ nhất trong công việc hàng ngày: trước khi review một PR, viết ra ba vấn đề
bạn dự đoán sẽ tìm thấy; sau đó đối chiếu. Việc này biến code review — vốn là feedback về người khác —
thành feedback về mô hình trong đầu bạn, và nó có độ trễ tính bằng phút. Tương tự: trước khi chạy load
test, viết ra con số bạn dự đoán; trước khi đọc phần kết của một postmortem, viết ra nguyên nhân bạn đoán.

**Mô hình 3 — Learning backlog vận hành như một sprint, WIP = 1.**

Mỗi mục học phải có bốn thứ, giống một ticket: một câu hỏi cụ thể (không phải một chủ đề), một deliverable
quan sát được, một nguồn feedback có tên người, và một ngày. WIP = 1 vì lý do đã nêu ở mục 2 và 3: học
cần block sâu, và ba mục học song song cho ra ba mục chưa xong.

### Practical Framework

**Bước 1 — Skill map 45 phút, một lần mỗi 6 tháng.**

Input: mô tả vai trò bạn muốn có trong 12–18 tháng (không phải title — mô tả công việc thật). Output: một
bảng, và từ đó chọn 2–3 ô để đầu tư.

| Năng lực | Mức hiện tại (0–3) | Mức cần cho vai đích | Khoảng cách | Có cơ hội thực hành trong công việc hiện tại? |
|---|---|---|---|---|
| ... | | | | Có / Không / Có nếu tôi đề xuất X |

Thang mức: **0** — chưa biết; **1** — đọc hiểu và nói chuyện được; **2** — làm được khi có người review;
**3** — làm được, debug được, và dạy được.

Cột cuối là cột quyết định, và nó thường bị bỏ. Cơ chế: một năng lực không có cơ hội thực hành trong công
việc hiện tại có chi phí học cao gấp nhiều lần (vì thiếu điều kiện thứ hai và thứ ba của deliberate
practice — feedback thật và lặp lại có sửa chữa) và tỉ lệ bỏ giữa đường rất cao. Với mỗi ô như vậy, bạn
có đúng ba đường: **tạo cơ hội** (đề xuất một việc trong tổ chức tạo ra vùng thực hành), **chấp nhận học
chậm và nông** qua side project, hoặc **đổi môi trường**. Ba đường có chi phí khác nhau và đều hợp lệ;
điều không hợp lệ là ghi nó vào kế hoạch mà không chọn đường nào.

**Bước 2 — Chọn ô để đầu tư theo định hướng vai trò.**

Hai nhánh nghề nghiệp cần bộ năng lực khác nhau ở *thứ tự đầu tư*, không phải khác nhau hoàn toàn. Bảng
dưới là điểm khởi đầu để tự hiệu chỉnh, không phải chuẩn:

| Năng lực | IC track (Staff / Principal) | Manager track (EM) |
|---|---|---|
| System design ở quy mô nhiều team | 3 | 2 |
| Đọc và phản biện kiến trúc của người khác | 3 | 2 |
| Viết RFC / ADR có sức thuyết phục | 3 | 2 |
| Debug production sâu, đọc được hệ thống lạ | 3 | 1–2 |
| Hiểu kinh tế học của hệ thống (chi phí hạ tầng, độ trễ, hàng đợi) | 3 | 2 |
| Estimation và planning nhiều luồng | 2 | 3 |
| Tuyển dụng và đánh giá năng lực người khác | 1–2 | 3 |
| Difficult conversation, feedback, xử lý underperformance | 2 | 3 |
| Stakeholder management và nhận thức về ngân sách | 2 | 3 |
| Mentoring và phát triển người | 2 | 3 |
| Hiểu kinh tế học sản phẩm và ưu tiên ở cấp tổ chức | 2 | 3 |

Hai ghi chú quan trọng. Thứ nhất, trong 12 tháng bạn chỉ có tài nguyên thực cho khoảng 2–3 ô, nên bảng
này là công cụ *loại bỏ*, không phải danh sách phải hoàn thành. Thứ hai, trong nhiều công ty Việt Nam
không có tầng Staff/Principal thực sự, nên cột bên phải là con đường duy nhất được tổ chức công nhận —
điều này tạo áp lực đẩy IC giỏi vào quản lý. Nếu đó là hoàn cảnh của bạn, việc học cần đi kèm một quyết
định về môi trường, không chỉ về nội dung. Vấn đề này thuộc `11-career-evolution.md`.

**Bước 3 — Learning backlog: mục học viết đúng cách.**

| Viết sai | Viết dùng được |
|---|---|
| "Học Kubernetes" | "Trả lời được: khi một pod trong cluster của mình bị OOMKilled thì có những chuỗi nguyên nhân nào, và tôi kiểm tra từng chuỗi bằng lệnh gì. Deliverable: runbook 1 trang cho team. Feedback: Nam (SRE) review. Xong trước 30/4." |
| "Học về distributed systems" | "Giải thích được cho team vì sao luồng đối soát hiện tại có thể ghi trùng khi retry, và đề xuất hai cách chống trùng kèm đánh đổi. Deliverable: một trang gửi vào channel + 15 phút trong buổi tech share. Feedback: phản biện của Hà." |
| "Đọc sách Designing Data-Intensive Applications" | "Đọc chương về replication, rồi vẽ lại mô hình replication của chính hệ thống mình và tìm một chỗ mà giả định của mình sai. Deliverable: một đoạn ghi chú trong file 'mô hình sai đã sửa'." |

Ba đặc điểm bắt buộc: mục tiêu là **một câu hỏi bạn hiện không trả lời được** chứ không phải một chủ đề;
deliverable là thứ **người khác nhìn thấy được**; và có **một người cho feedback có tên**. Thiếu yếu tố
thứ ba là nguyên nhân phổ biến nhất khiến việc học tự phát không tạo ra năng lực — vì không có feedback
thì không có deliberate practice, chỉ có tiêu thụ.

**Bước 4 — Hai vòng học rẻ nhất, nằm sẵn trong công việc hàng ngày.**

Đây là phần có đòn bẩy cao nhất của toàn mục, vì nó không cần thêm thời gian ngoài giờ.

*(a) Học từ code của người khác — 25 phút mỗi tuần.* Chọn một PR hoặc một module **không phải của mình**,
ưu tiên của người bạn cho là giỏi hơn hoặc của một hệ thống bạn không hiểu. Quy trình bắt buộc theo đúng
thứ tự: đọc mô tả vấn đề trước, **viết ra cách mình sẽ làm** (3–5 dòng), rồi mới đọc code. Sau đó viết
đúng hai dòng: một điều mình không nghĩ tới, và một điều mình vẫn sẽ làm khác kèm lý do.

Cơ chế: bước dự đoán trước khi nhận thông tin biến việc đọc thụ động thành retrieval practice có feedback
tức thì (mô hình 2). Bỏ bước đó, bạn chỉ đang xác nhận rằng code người khác hợp lý — cảm giác hiểu mà
không có học.

*(b) Học từ incident — hai lớp bài học.* Mọi postmortem tử tế đều ghi lớp thứ nhất: nguyên nhân kỹ thuật
và action item. Lớp thứ hai gần như luôn bị bỏ, và nó là lớp tạo ra năng lực: **mô hình sai nào trong đầu
tôi khiến tôi không lường được điều này?**

Ví dụ cụ thể để thấy sự khác biệt giữa hai lớp. Lớp một: "job đối soát fail vì partner đổi format file;
action item: thêm validation và alert". Lớp hai: "mô hình sai của tôi là coi input từ đối tác là ổn định
theo hợp đồng. Mô hình đúng hơn: mọi input từ ngoài biên tổ chức là không đáng tin bất kể hợp đồng nói
gì, và biên đó cần một tầng kiểm tra riêng." Lớp hai chuyển giao sang mọi tích hợp trong tương lai; lớp
một chỉ sửa được một job.

Giữ một file riêng — đặt tên gì cũng được, ví dụ `mo-hinh-sai-da-sua.md` — mỗi mục 3–5 dòng: mô hình cũ,
bằng chứng phá nó, mô hình mới. Sau hai năm, file này thường là tài sản có giá trị nhất trong toàn bộ hệ
thống ghi chép của bạn, và nó cũng là nguồn nội dung tốt nhất cho việc mentoring.

Biến thể rẻ hơn nữa: **đọc postmortem công khai của công ty khác** và tự trả lời câu hỏi lớp hai trước
khi đọc phần kết luận. Đây là cách mượn hoá đơn của người khác — giải pháp trực tiếp cho cơ chế hai ở
phần First Principles.

**Bước 5 — Năm câu hỏi khi có một công nghệ đang được nói nhiều.**

1. **Vấn đề nào trong hệ thống của tôi mà cái này giải quyết?** Nếu không có, giá trị của việc học nó là
   giá trị thị trường lao động thuần — điều đó hợp lệ, nhưng phải gọi đúng tên, vì mục tiêu khác nhau dẫn
   tới độ sâu cần khác nhau.
2. **Nó thay thế cái gì, và chi phí chuyển đổi thật là bao nhiêu?** Bao gồm cả chi phí vận hành, chi phí
   tuyển người biết nó, và chi phí của việc team mất năng suất trong 3 tháng đầu.
3. **Kiến thức nền nào nằm dưới nó?** Ví dụ: học Kafka mà không hiểu về log, phân vùng, thứ tự và
   at-least-once thì chỉ học được cách gọi API. Học tầng nền chuyển giao được sang mọi broker khác.
4. **Có ai trong bán kính của tôi đã chạy nó ở production đủ 12 tháng?** Đây là câu hỏi lọc mạnh nhất, vì
   phần đắt nhất của một công nghệ chỉ hiện ra ở tháng thứ 6–12, và nội dung đó không có trong tài liệu
   chính thức.
5. **Nếu tôi bỏ 40 giờ vào việc này thay vì vào ô trống lớn nhất trong skill map, tôi mất gì?**

### Trade-off

| Trục | Nghiêng về bên A khi | Nghiêng về bên B khi | Ai chịu phần mất |
|---|---|---|---|
| **A: Đi sâu một lĩnh vực — B: Mở rộng nhiều lĩnh vực** | Bạn nhắm IC track; tổ chức có bài toán đủ khó trong lĩnh vực đó; bạn cần một cơ sở để tổ chức tin | Bạn nhắm lead/EM; bạn phải đánh giá phương án ở nhiều lĩnh vực; bạn đang ở tổ chức nhỏ phải làm nhiều thứ | Sâu quá hẹp: khi lĩnh vực đó hết nhu cầu, chuyển đổi đắt. Rộng mà không sâu: không ai gọi bạn khi có việc khó, và uy tín kỹ thuật không hình thành |
| **A: Học nền tảng — B: Học công cụ đang cần ngay** | Bạn có 3+ năm kinh nghiệm và bắt đầu thấy mình lặp lại; bạn muốn năng lực chuyển giao được | Có nhu cầu công việc trong 4 tuần tới; hoặc bạn đang cần một việc mới ngay | Chỉ nền tảng: không ứng dụng được nên không có feedback, và học trở thành đọc. Chỉ công cụ: khấu hao nhanh, mỗi 3 năm học lại từ đầu |
| **A: Học để đổi việc tăng lương — B: Học để tăng năng lực hệ thống** | 3–5 năm đầu nghề; mức lương đang dưới thị trường rõ rệt; môi trường hiện tại không có bài toán mới | Bạn đang tiến tới trần lương của Senior; bước tiếp theo cần bằng chứng về ảnh hưởng dài hạn | Xem phân tích bên dưới |
| **A: Học trong giờ làm — B: Học ngoài giờ** | Việc học có nối tới bài toán của tổ chức (và bạn nói được điều đó với cấp trên) | Nội dung không liên quan tới công việc hiện tại; hoặc bạn đang chuẩn bị chuyển hướng | Ngoài giờ bằng ngân sách năng lượng đã âm: tỉ lệ hoàn thành thấp, và chất lượng công việc chính giảm — trả hai lần, xem mục 7 |

Trục thứ ba cần nói thẳng, vì nó là trục thật trong thị trường Việt Nam và việc né nó là không trung
thực.

**Học để đổi việc tăng lương là một chiến lược có lợi nhuận thật.** Ở thị trường Việt Nam trong nhiều
năm, mức tăng lương khi đổi việc thường vượt xa mức tăng nội bộ, và bộ kỹ năng để đổi việc thành công
(rộng, có từ khoá đúng, luyện được các dạng câu hỏi phỏng vấn phổ biến) khác với bộ kỹ năng để làm hệ
thống tốt. Phủ nhận điều này là dạy người ta một chiến lược thua.

Trade-off thật của chiến lược đó, nói bằng cơ chế: nó tối ưu tốt cho khoảng 3–5 năm đầu và bắt đầu tính
lãi sau đó, vì ba lý do. Thứ nhất, nếu bạn không bao giờ ở lại đủ lâu để thấy hệ quả của quyết định kiến
trúc của mình, bạn không phát triển được năng lực phán đoán kiến trúc — đây là cơ chế hai ở phần First
Principles, và nó không thể bù bằng đọc sách. Thứ hai, mức lương của Senior tiến tới trần thị trường, và
bước tiếp theo (Staff, Principal, EM) được đánh giá bằng bằng chứng về ảnh hưởng dài hạn — thứ mà chu kỳ
18 tháng không tạo ra được. Thứ ba, khi thị trường nguội, nhóm có title cao mà thiếu chiều sâu là nhóm
chịu ảnh hưởng đầu tiên, vì họ đắt và khó chứng minh giá trị.

Và trade-off đối xứng, cũng phải nói: **ở lại một chỗ quá lâu có chi phí không nhỏ hơn.** Bạn học rất sâu
về một hệ thống mà không biết phần nào là nguyên lý chung và phần nào là đặc thù của chỗ đó; bạn mất khả
năng đọc thị trường và định giá bản thân; và nếu môi trường đó không có người giỏi hơn bạn, chiều sâu của
bạn dừng lại ở mức mà tổ chức đó cần.

Hai điều kiện thực dụng để chọn: **ở lại** khi bạn vẫn gặp một loại vấn đề mới mỗi khoảng 6 tháng, và
trong bán kính làm việc vẫn có người mà bạn học được từ họ. **Đi** khi hai điều đó đã hết — và lưu ý rằng
"hết" là một trạng thái có thể tự tạo ra lại được (đề xuất một bài toán mới, chuyển sang một team khác,
mời một người giỏi vào) trước khi kết luận rằng phải đi.

### Real-world Scenarios

**Tình huống A — "9 năm kinh nghiệm" trong ràng buộc ODC.**

Bối cảnh: chính Tuấn ở đầu mục. Ràng buộc thật của môi trường: kiến trúc do bên khách quyết, kỹ sư không
được tham gia buổi thiết kế, khối lượng dữ liệu luôn trong ngưỡng dễ, và utilization rate được theo dõi
nên khó xin thời gian cho việc không billable. Trong ràng buộc đó, lời khuyên "hãy tìm bài toán khó hơn"
là vô nghĩa.

Cách xử lý sai mà Tuấn đã làm trong hai năm: mua 4 khoá học online về kiến trúc, hoàn thành một, và học
vào 23h sau khi làm 10 giờ. Cơ chế thất bại có hai lớp: không có vùng thực hành nên không có feedback
(vi phạm điều kiện 2 và 3 của deliberate practice), và học bằng ngân sách năng lượng đã âm nên tỉ lệ
hoàn thành thấp — điều này còn tạo thêm một vòng lặp tệ là cảm giác mình thiếu kỷ luật, trong khi vấn đề
là thiết kế.

Cách xử lý có hiệu lực: tạo vùng thực hành **bên trong** ràng buộc, không cần xin phép ai đổi cấu trúc.
Bốn việc Tuấn làm trong 6 tháng:

1. **Xin làm người viết tài liệu thiết kế** cho các module mới của team, kể cả khi quyết định thiết kế
   thuộc bên khách. Việc viết buộc phải hiểu đủ để giải thích, và tài liệu tạo ra một vòng feedback: khách
   và team đọc và phản biện. Chi phí với tổ chức bằng không — đây là việc mà không ai muốn làm.
2. **Nhận review hai module không phải của mình mỗi tuần**, áp quy trình dự đoán-trước ở Bước 4a.
3. **Xin vào buổi họp kiến trúc với khách ở vai ghi biên bản.** Đây là kỹ thuật rất hiệu quả trong ODC:
   vai ghi biên bản gần như luôn được chấp nhận, và nó cho quyền tiếp cận đúng những cuộc thảo luận mà
   Tuấn cần nghe. Sau ba tháng, việc ghi biên bản chuyển thành việc được hỏi ý kiến, vì Tuấn là người
   nhớ toàn bộ lịch sử quyết định.
4. **Dựng lại một phần hệ thống ở quy mô nhỏ và chủ động phá nó.** Cụ thể với khoảng trống của Tuấn: dựng
   hai service cùng ghi vào một bảng số dư trên máy cá nhân, tạo tình trạng đồng thời bằng script, quan
   sát số dư âm, rồi thử lần lượt từng cách chống — isolation level cao hơn, lock bi quan, một hàng đợi
   tuần tự hoá theo tài khoản — và đo. Ba buổi tối, và nó tạo ra đúng thứ mà 9 năm dự án CRUD không tạo
   ra: một vòng feedback chặt trên một loại vấn đề mới.

Kết quả sau 6 tháng: Tuấn trả lời được loại câu hỏi ở đầu mục không phải vì đã đọc về nó mà vì đã thấy nó
hỏng trên máy mình. Bài học truy được về cơ chế: khi tổ chức không cung cấp vòng feedback, bạn dựng vòng
feedback nhân tạo — và một vòng nhân tạo có feedback chặt vẫn tốt hơn nhiều so với việc đọc về một vòng
thật.

**Tình huống B — Một Senior muốn mang công nghệ mới vào sau một hội thảo.**

Bối cảnh: startup product, team 12 người, hệ thống đang chạy ổn với một monolith và PostgreSQL. Khôi
(Senior BE, 5 năm) sau một hội thảo đề xuất chuyển luồng đơn hàng sang event sourcing với Kafka. Bạn là
Tech Lead.

Hai cách xử lý sai, cả hai đều phổ biến. Cách thứ nhất: đồng ý vì không muốn dập nhiệt tình của một người
giỏi — kết quả là 4 tháng sau team có hai hệ thống, một nửa hoàn thành, và không ai ngoài Khôi vận hành
được. Cách thứ hai: từ chối bằng lập luận "mình chưa cần" — đúng về kỹ thuật, nhưng Khôi mất động lực,
ngừng đề xuất, và 8 tháng sau nghỉ việc; bạn đã tiết kiệm được chi phí kỹ thuật và trả bằng một chi phí
lớn hơn nhiều.

> **Khôi:** Em nghĩ mình nên chuyển luồng đơn hàng sang event sourcing. Bên chia sẻ ở hội thảo họ làm và
> nói giải quyết được vấn đề audit với cả việc replay lại trạng thái.
>
> **Bạn:** Anh không nói không, và anh muốn nói trước là anh thấy hướng này đáng xem xét thật. Nhưng anh
> muốn biến nó thành một quyết định có cơ sở, chứ không phải anh gật hoặc anh lắc theo cảm tính. Em nhận
> việc này được không: viết cho anh một RFC hai trang trả lời bốn câu.
>
> **Khôi:** Bốn câu gì ạ?
>
> **Bạn:** Một, vấn đề nào của mình hiện tại nó giải quyết — nói bằng chuyện đã xảy ra, ví dụ lần
> đối soát tháng trước mình không dựng lại được trạng thái. Hai, nó thay thế cái gì và chi phí vận hành
> thêm là gì: ai on-call cho Kafka, ai nâng cấp, và nếu em nghỉ phép hai tuần thì ai xử lý. Ba, phương án
> rẻ hơn nào cho cùng vấn đề đó — anh nghĩ có ít nhất một là bảng audit log dạng append-only, em so sánh
> giúp anh. Bốn, nếu làm thì phần nhỏ nhất mình có thể làm trước để biết mình sai là gì.
>
> **Khôi:** Nghe như anh đang muốn em tự chứng minh là không nên làm.
>
> **Bạn:** Anh hiểu vì sao nghe như vậy, nên anh nói rõ hai điều. Thứ nhất, nếu bốn câu đó có câu trả
> lời tốt thì anh sẽ đưa vào roadmap quý sau và em là người dẫn — anh cam kết chỗ đó. Thứ hai, kể cả nếu
> kết luận là chưa làm, thì cái em học được từ ba ngày đó là của em và anh ghi nhận nó trong review, vì
> viết được một RFC so sánh phương án là đúng thứ mà một Staff Engineer làm. Anh cấp cho em 3 ngày trong
> sprint này, không phải làm ngoài giờ.
>
> **Khôi:** Vậy em làm.
>
> **Bạn:** Một chuyện nữa: sau khi viết xong, em trình bày 20 phút cho cả team. Nếu phương án bị bác thì
> nó phải bị bác trong phòng có nhiều người, không phải bị anh bác một mình — như thế công bằng hơn với
> em và team cũng học được cách đọc trade-off.

Cơ chế của cách xử lý này: nó chuyển động lực học của một cá nhân thành **đầu ra mà tổ chức dùng được**,
giữ được người, và tạo ra một vòng feedback thật cho Khôi (viết, bị phản biện, sửa) — đúng ba điều kiện
của deliberate practice. Nó cũng đặt chi phí đúng chỗ: 3 ngày trong sprint là chi phí thật mà bạn chấp
nhận trả để mua hai thứ, một quyết định có cơ sở và sự phát triển của một người giỏi.

### Best Practices

**Định nghĩa mục tiêu học bằng một câu hỏi bạn hiện không trả lời được, không bằng một chủ đề.** Lý do:
chủ đề không có điều kiện dừng nên không bao giờ xong, và không có tiêu chí nào để biết mình đã tiến bộ.
Câu hỏi có cả hai.

**Mỗi mục học phải có deliverable người khác nhìn thấy được và một người cho feedback có tên.** Lý do:
thiếu feedback thì không có deliberate practice, chỉ có tiêu thụ. Deliverable công khai còn tạo một ràng
buộc bên ngoài, thứ chịu tải tốt hơn động lực nội tại trong tuần thứ ba.

**Giữ WIP = 1.** Lý do: học cần block sâu và block sâu là tài nguyên khan nhất (mục 2). Ba mục học song
song cho ra ba mục dở dang, và mục dở dang không tạo ra năng lực nào — khác với công việc, ở đó phần dở
dang ít nhất còn là code.

**Mỗi năm đầu tư ít nhất một chủ đề ở tầng nền.** Lý do: chu kỳ bán rã. Đây là phần duy nhất của việc
học có tính tích lũy; phần còn lại khấu hao.

**Dự đoán trước khi nhận thông tin, ở mọi nơi có thể.** Lý do: đây là cách rẻ nhất để kéo feedback từ
tầng chậm về tầng nhanh, và nó áp dụng được vào việc bạn đã làm hàng ngày mà không cần thêm thời gian.

**Giữ một file "mô hình sai đã sửa" và cập nhật sau mỗi incident.** Lý do: nó ép việc học lên lớp mô
hình thay vì dừng ở lớp kỹ thuật, và nó là tài sản duy nhất trong hệ thống ghi chép của bạn có giá trị
tăng theo thời gian.

**Chọn nơi làm việc theo mật độ vấn đề mới và theo việc có người giỏi hơn mình trong bán kính.** Lý do:
hai biến này quyết định vòng feedback bạn được tiếp xúc hàng ngày, và chúng có ảnh hưởng lớn hơn mọi kế
hoạch học cá nhân. Không có kế hoạch tự học nào bù được việc làm ba năm trong một môi trường không có bài
toán mới.

### Anti-patterns

**Tiêu thụ thay cho thực hành.**
Cơ chế: đọc và xem cho cảm giác tiến triển ngay và không có rủi ro thất bại, trong khi thực hành cho cảm
giác kém cỏi và có rủi ro sai. Vì cảm giác dễ hiểu bị nhầm với việc học được (cơ chế ba ở First
Principles), người ta chọn đúng cái ít tạo ra năng lực nhất.
Dấu hiệu sớm: tỉ lệ khoá học đã mua trên khoá học hoàn thành; bạn nhớ tên nhiều khái niệm nhưng không kể
được lần nào bạn dùng chúng để quyết một việc thật; bạn không viết ra gì sau khi đọc.

**Chạy theo hype không có bài toán.**
Cơ chế: chi phí không nằm ở việc học sai thứ — mà ở việc bạn mang một giải pháp đi tìm vấn đề. Ở cấp IC,
tác hại giới hạn ở thời gian của bạn. Ở cấp lead, bạn có quyền áp đặt, nên tác hại nhân lên thành chi phí
vận hành, chi phí Cognitive Load của cả team, và một hệ thống có hai kiến trúc.
Dấu hiệu sớm: giải pháp được nêu trước khi vấn đề được phát biểu; lý do có dạng "công ty X dùng cái này";
không ai trả lời được vấn đề hiện tại nào sẽ mất đi.

**Rộng mãi, không có chiều sâu nào đủ để tổ chức tin.**
Cơ chế: chiều rộng có được rẻ và cho cảm giác tiến bộ nhanh, nên nó là con đường ít trở lực. Nhưng thẩm
quyền kỹ thuật hình thành từ việc người khác đã thấy bạn xử lý một việc khó mà họ không làm được — điều
chỉ có ở chiều sâu.
Dấu hiệu sớm: bạn có ý kiến về mọi thứ; không ai gọi bạn khi có sự cố khó; trong tranh luận kỹ thuật, ý
kiến của bạn không đổi được quyết định.

**"10 năm thành 1 năm lặp 10 lần".**
Cơ chế: tổ chức giao việc theo năng lực đã chứng minh, nên nếu bạn không chủ động tạo vùng thực hành mới,
hệ thống sẽ tối ưu bạn về phía lặp lại — đó là hành vi hợp lý của tổ chức, không phải sự thiếu quan tâm.
Dấu hiệu sớm: bạn không kể được một quyết định kỹ thuật nào của mình mà bạn đã thấy hệ quả đầy đủ; mọi
dự án gần đây có cùng hình dạng; bạn không nhớ lần cuối mình bị bế tắc thật.

**Chỉ học từ tài liệu chính thức, không học từ hệ thống thật.**
Cơ chế: tài liệu mô tả trường hợp lý tưởng; hệ thống thật chứa các quyết định đánh đổi, các chỗ chữa
cháy, và lịch sử của chúng. Người chỉ đọc tài liệu học được cách một thứ nên hoạt động, không học được
cách nó hỏng — và toàn bộ giá trị của Senior nằm ở phần thứ hai.
Dấu hiệu sớm: bạn chưa từng đọc git log của module mình đang sửa; bạn gọi code cũ là "code rác" mà chưa
tìm hiểu vì sao nó thành ra như vậy.

**Học ngoài giờ bằng ngân sách năng lượng đã âm.**
Cơ chế: học đòi hỏi working memory và khả năng chịu được cảm giác chưa hiểu — hai thứ suy giảm mạnh khi
thiếu ngủ và khi đã tiêu hết attention trong ngày. Kết quả: tỉ lệ hoàn thành thấp, chất lượng công việc
chính giảm, và người học kết luận sai rằng vấn đề là kỷ luật.
Dấu hiệu sớm: khung giờ học của bạn là sau 22h; bạn phải đọc lại cùng một trang nhiều lần; bạn bỏ mục
học ở tuần thứ hai và tự trách mình. Xem mục 7.

### Khi nào KHÔNG nên áp dụng

**Trong 4–8 tuần crunch hoặc giai đoạn nhiều incident — cắt learning backlog về 0 một cách tường minh.**
Cơ chế: việc học cần hai thứ mà giai đoạn này không có — block attention chất lượng cao, và dung lượng
để làm sai. Việc giữ mục tiêu học trong giai đoạn đó không tạo ra học, chỉ tạo ra nợ cảm xúc và thêm một
mục thất bại vào danh sách. Cách làm đúng: ghi ra ngày mở lại (một ngày cụ thể, không phải "khi nào hết
bận"), và trong giai đoạn đó chỉ giữ đúng một vòng học — vòng học từ incident ở Bước 4b, vì nó xảy ra
trong lúc làm việc và không cần thêm thời gian.

**Trong 3 tháng đầu ở một vai trò mới — ưu tiên học bối cảnh, không học công nghệ.** Bối cảnh gồm: hệ
thống thật hoạt động thế nào, ai quyết cái gì, những quyết định nào đã được cân nhắc và cố tình để vậy,
lịch sử của các mâu thuẫn hiện tại. Đây là dạng kiến thức có giá trị cao nhất trong giai đoạn đó và mất
giá nhanh nhất nếu không thu thập ngay (sau 6 tháng bạn sẽ mất khả năng đặt câu hỏi của người mới). Cần
nói thẳng một điều: nhiều người vừa lên lead dùng việc học kỹ thuật như một chỗ trốn khỏi phần khó của
vai trò mới — vì học kỹ thuật là vùng họ giỏi và có cảm giác kiểm soát, còn đối thoại khó thì không.

**Khi khoảng cách năng lực không phải nguyên nhân của vấn đề bạn đang gặp.** Nếu bạn không được lắng nghe
trong các quyết định kiến trúc, khả năng cao nguyên nhân nằm ở Influence, ở cách trình bày, hoặc ở việc
bạn chưa xây được uy tín — thêm một certificate không đổi được gì. Chẩn đoán trước khi học: nếu bạn đưa
ra đúng phương án và nó vẫn không được chọn, vấn đề không ở nội dung. Xem `00-nen-tang-leadership.md` và
`02-communication.md`.

**Khi tổ chức không có ứng dụng nào cho năng lực đó trong 12 tháng tới.** Việc học vẫn có giá trị thị
trường lao động và đó là lý do hợp lệ, nhưng phải gọi đúng tên và không kỳ vọng nó cải thiện tình hình
hiện tại. Nhầm hai mục tiêu này tạo ra thất vọng có hệ thống: bạn học xong, không dùng được, và kết luận
sai rằng việc học vô ích.

**Khi bạn đang là điểm nghẽn về throughput của team.** Một khoá học 40 giờ hoặc một tuần đi hội thảo trong
giai đoạn team đang chờ bạn có chi phí hệ thống thật, và chi phí đó do người khác trả. Điều này không có
nghĩa là không được học — nó có nghĩa là phải sửa việc mình là điểm nghẽn trước (bằng Delegation và tài
liệu), và việc đó bản thân nó cũng là một mục học.

**Khi bạn đang ở giai đoạn cần chiều sâu để có thẩm quyền, mà lại đi mở rộng.** Điều kiện nhận biết: bạn
đã đủ rộng để nói chuyện được về mọi lĩnh vực, nhưng chưa có một lĩnh vực nào mà team tự động tìm đến
bạn. Trong điều kiện đó, việc học thêm một lĩnh vực mới có giá trị cận biên âm — nó lấy đúng thời gian
cần cho việc đào sâu.

---

## 7. Quản lý năng lượng, sustainable pace và burnout

### Problem Statement

Linh là Tech Lead một team 7 người tại công ty e-commerce đã xuất hiện ở mục 2, bốn tháng sau khi nhận
vai. Không ai trong tổ chức nghĩ có vấn đề gì: Linh vẫn đi làm đúng giờ, vẫn giao được release, vẫn được
đánh giá là "gánh team". Nhưng dữ liệu trong công cụ của chính team nói khác:

| Chỉ số | Bốn tháng trước | Hiện tại |
|---|---|---|
| Số comment trung bình trên mỗi PR Linh review | 6 | 1 |
| Tỉ lệ comment về thiết kế / kiến trúc (so với comment về hình thức) | ~70% | ~15% |
| Số giờ làm mỗi tuần | 45 | 54 |
| Số PR Linh merge mỗi sprint | 9 | 5 |
| Hai việc khó nhất trong backlog (thiết kế lại reconcile, dựng load test) | đang làm | bị đẩy sang sprint sau ba lần liên tiếp |
| Số đề xuất kỹ thuật Linh đưa ra trong quý | 4 | 0 |
| Số ngày nghỉ phép trong 7 tháng | — | 0 |

Thêm ba quan sát định tính: trong retro, Linh bắt đầu dùng các câu tổng quát hoá ("bên product lúc nào
cũng vậy"); Linh tắt camera trong các buổi họp mà trước đây bật; và một lỗi cấu hình ở bước deploy —
loại lỗi mà Linh chưa từng mắc trong ba năm.

Đây là dạng thất bại đắt nhất của Level 1, vì nó **không hiển thị trong bất kỳ chỉ số nào mà tổ chức đang
theo dõi** cho tới khi nó thành một trong hai thứ: một sự cố, hoặc một đơn xin nghỉ việc.

Dấu hiệu sớm quan sát được trong công việc kỹ thuật — đây là danh sách quan trọng nhất của mục này, vì nó
áp dụng được cho chính bạn và cho người trong team:

- **Chất lượng review giảm, và giảm theo một hình dạng cụ thể:** số comment giảm, và nội dung comment
  chuyển từ tầng thiết kế sang tầng hình thức (đặt tên, format). Cơ chế: đọc thiết kế cần giữ mô hình
  trong working memory; đọc hình thức thì không. Khi năng lượng thấp, bạn tự động chuyển sang loại việc
  rẻ hơn về nhận thức mà vẫn trông như đang làm việc.
- **Tránh việc khó một cách có hệ thống:** việc cần block sâu bị đẩy sang sprint sau nhiều lần, trong khi
  các task cơ học được hoàn thành đúng hạn. Đây là dấu hiệu có độ đặc hiệu cao và thường xuất hiện sớm.
- **Giờ tăng, đầu ra giảm.** Sự đồng thời của hai xu hướng này gần như luôn có nghĩa là chất lượng
  attention đang giảm và bạn đang bù bằng số lượng.
- **Giảm phạm vi hành động tự nguyện:** ngừng đề xuất, ngừng phản biện, ngừng nhận việc ngoài phạm vi hẹp.
  Đây là dấu hiệu bị đọc sai thường xuyên nhất — nó hay được ghi nhận là "đã chín chắn hơn", "đã bớt bốc
  đồng".
- **Ngôn ngữ chuyển sang tổng quát hoá và giễu.** Cụ thể: chủ ngữ chuyển từ người sang bộ phận ("bên
  product", "mấy ông business"), và câu chuyển từ mô tả sự việc sang mô tả bản chất ("lúc nào cũng vậy").
- **Giảm tiếp xúc:** tắt camera, im lặng trong họp, ngừng ăn trưa với team, trả lời ngắn hơn bình thường.
- **Tăng lỗi cơ học** trong các thao tác quen: deploy, cấu hình, chạy script trên production.
- **Không nghỉ phép, hoặc nghỉ phép mà vẫn đọc Slack.**

Hệ quả lan xuống team, và đây là phần khiến vấn đề này thuộc chương về leadership chứ không thuộc phạm vi
đời sống cá nhân: khi chất lượng review của lead giảm, **chuẩn kỹ thuật của team giảm mà không có ai công
bố** — code đi qua, mọi người tưởng nó đạt. Khi lead tránh việc khó, các quyết định kiến trúc bị trì hoãn
và team phải tự quyết trong tình trạng thiếu bối cảnh. Và cynicism của lead lan nhanh hơn mọi thông điệp
chính thức, vì team đọc hành vi của lead như tín hiệu đáng tin nhất về việc tổ chức thực sự thế nào.

### First Principles

**Cơ chế một: đầu ra là hàm của giờ nhân với chất lượng attention, và chất lượng attention có tốc độ phục
hồi giới hạn.**

```
Đầu ra ≈ Số giờ × Chất lượng attention

Chất lượng attention giảm trong ngày, giảm theo tuần nếu không phục hồi,
và phục hồi có tốc độ tối đa (không thể "nghỉ bù nhanh").

→ Giờ thêm vào nằm ở cuối ngày, tức là ở vùng chất lượng thấp nhất,
  và nó vay từ ngân sách phục hồi của ngày mai.
```

Hệ quả quan trọng và ngược trực giác với cách nhiều tổ chức Việt Nam đo lường: sau vài tuần, **tổng đầu
ra có thể giảm dù tổng giờ tăng**. Và vì tổng giờ tăng, người ta rút ra kết luận sai rằng cần thêm giờ
nữa — đó là một vòng phản hồi dương đi sai hướng. Về mức độ chắc chắn của bằng chứng: có một dòng nghiên
cứu công khai lâu đời về quan hệ giữa số giờ làm việc kéo dài và năng suất (từ các nghiên cứu công nghiệp
thế kỷ 20 tới các tổng hợp trong xây dựng và quân sự), và các con số ngưỡng cụ thể khác nhau theo loại
công việc. Điều chắc chắn về cơ chế là quan hệ **không tuyến tính và có ngưỡng**; đừng dựa vào một con số
cụ thể nào như thể nó là hằng số vật lý.

**Cơ chế hai: burnout có ba trục độc lập, và chúng không đi cùng nhau.** Mô hình được dùng rộng rãi nhất
(Maslach, nguồn công khai qua Maslach Burnout Inventory) mô tả ba trục:

| Trục | Nội dung | Biểu hiện trong công việc kỹ thuật |
|---|---|---|
| **Exhaustion** | Kiệt lực về năng lượng | Thời gian nạp bối cảnh tăng; tránh việc cần tập trung dài; tăng lỗi cơ học |
| **Cynicism** (depersonalization) | Xa cách, giễu, coi công việc và người khác như đồ vật | Tổng quát hoá về bộ phận khác; humour phòng vệ; ngừng phản biện có xây dựng |
| **Inefficacy** | Cảm giác việc mình làm không tạo ra tác động | Ngừng đề xuất; giảm ownership tự nguyện; "làm cũng chẳng để làm gì" |

Điểm quan trọng nhất mà mọi cuộc thảo luận thông thường bỏ qua: **ba trục không tăng cùng nhau và không
cùng tốc độ.** Có người làm 60 giờ/tuần trong nhiều tháng với exhaustion cao mà hai trục kia thấp — và họ
không burnout theo nghĩa lâm sàng, họ chỉ mệt. Có người làm đúng 40 giờ mà burnout nặng vì inefficacy và
cynicism cao — việc bị đổi ưu tiên liên tục, làm xong không được release, ba tháng công không ai dùng.

Suy ra hai hệ quả rất thực dụng. Thứ nhất, **đo giờ không phát hiện được burnout**; phải đọc cả ba trục.
Thứ hai, trong công việc kỹ thuật, trục nguy hiểm nhất và bị bỏ nhiều nhất thường là **inefficacy** — và
nó không được sửa bằng nghỉ ngơi. Một người bị inefficacy đi nghỉ một tuần sẽ trở lại với cùng vấn đề.

**Cơ chế ba: nguồn của burnout nằm chủ yếu ở điều kiện công việc, không ở đặc điểm cá nhân.** Dòng nghiên
cứu công khai của Maslach và Leiter nêu sáu vùng có thể lệch giữa người và công việc: **workload,
control, reward, community, fairness, values**. Đây là mô hình có giá trị chẩn đoán cao nhất trong mục
này, vì nó chỉ ra rằng các "vùng lệch" khác nhau cần các can thiệp hoàn toàn khác nhau:

```
Lệch workload  → cần cắt cam kết, thêm người, hoặc giảm scope
Lệch control   → cần quyền quyết định trong phạm vi mình chịu hệ quả (xem mục 1)
Lệch reward    → cần ghi nhận đúng loại việc (không chỉ tiền)
Lệch community → cần quan hệ và Psychological Safety trong team
Lệch fairness  → cần minh bạch trong đánh giá, promotion, phân việc
Lệch values    → cần thay đổi việc mình làm, hoặc thay đổi nơi làm
```

Hệ quả rất thẳng: **"tự chăm sóc bản thân" không sửa được lệch về control, fairness hay values.** Một
team burnout vì priority thrashing (ưu tiên bị đổi liên tục — mục 3) không được cứu bằng lớp yoga hay
ngày team building. Điều đó không có nghĩa là ngủ đủ và tập thể dục vô ích — chúng tăng hạn mức chịu tải.
Nhưng nếu nguyên nhân là cơ chế mà can thiệp lại ở tầng cá nhân, kết quả là người lao động nhận được
thông điệp rằng vấn đề nằm ở khả năng chịu đựng của họ, và điều đó **làm tăng trục cynicism**.

**Cơ chế bốn: giấc ngủ là input của đúng hai năng lực mà công việc lead dùng nhiều nhất.** Không giảng
đạo, chỉ nêu cơ chế: thiếu ngủ làm giảm dung lượng working memory và giảm khả năng ức chế (inhibition) —
tức là đúng năng lực giữ mô hình phức tạp (việc kỹ thuật) và năng lực không phản ứng theo cảm xúc trong
đối thoại khó (việc lead). Đây là lý do một buổi họp đêm với khách Mỹ có chi phí kép: nó lấy buổi tối và
làm giảm chất lượng của cả ngày hôm sau.

**Cơ chế năm: phục hồi cần detachment, không chỉ cần thời gian.** Dòng nghiên cứu công khai về
psychological detachment (Sonnentag và đồng nghiệp) cho thấy thời gian nghỉ mà vẫn theo dõi công việc tạo
ra rất ít phục hồi. Hai hệ quả thiết kế: nghỉ phép mà vẫn đọc Slack là chi phí không mua được lợi ích
(bạn mất ngày phép và không phục hồi); và **on-call liên tục có chi phí cao hơn số incident thực tế**,
vì trạng thái sẵn sàng ngăn detachment kể cả trong những đêm không có gì xảy ra. Đây là lập luận mạnh
nhất cho việc rotation on-call, và nó là lập luận về chi phí chứ không về sự công bằng.

### Mental Model

**Mô hình 1 — Ngân sách năng lượng có bốn dòng.**

```
NẠP     : ngủ, thời gian không có nghĩa vụ, việc tạo cảm giác hoàn thành,
          quan hệ, hoạt động thể chất
RÚT     : giờ làm, số quyết định, đối thoại khó, họp đêm, on-call, bất định
LÃI     : nợ năng lượng không trả sẽ tính lãi bằng chất lượng quyết định
          và chất lượng quan hệ — hai thứ khó khôi phục hơn giấc ngủ
HẠN MỨC : có giới hạn cá nhân, và giới hạn đó thay đổi theo giai đoạn đời
```

Giá trị của mô hình nằm ở dòng LÃI. Người ta thường nghĩ chi phí của việc thiếu ngủ là cảm giác mệt — thứ
biến mất sau một đêm ngủ bù. Chi phí thật là các quyết định đã ra trong trạng thái đó và các câu đã nói
trong các cuộc đối thoại khó, và hai thứ đó không ngủ bù được.

**Mô hình 2 — Ba trục Maslach như ba đèn chỉ báo, mỗi trục có cách đo riêng.**

| Trục | Tín hiệu trong công việc kỹ thuật | Đo bằng gì | Can thiệp đúng |
|---|---|---|---|
| Exhaustion | Thời gian từ lúc mở task đến commit đầu tiên tăng; tránh task cần block dài; lỗi thao tác | Thời gian nạp bối cảnh; số lỗi cơ học; giờ làm | Giảm tải, phục hồi có detachment |
| Cynicism | Tổng quát hoá về bộ phận khác; giảm phản biện xây dựng; giễu | Ngôn ngữ trong retro và trong PR comment | Sửa vùng lệch fairness/community; đối thoại thật |
| Inefficacy | Ngừng đề xuất; ngừng nhận việc ngoài phạm vi; "làm cũng chẳng để làm gì" | Số đề xuất/RFC trong quý; tỉ lệ việc làm xong được release và được dùng | Nối việc với kết quả; giảm thrashing; trả lại control |

Mô hình này quan trọng vì **chẩn đoán sai trục dẫn tới can thiệp vô hiệu**. Cho một người bị inefficacy
nghỉ một tuần là tiêu một tuần phép mà không sửa gì. Cho một người bị exhaustion thêm quyền quyết định là
thêm tải.

**Mô hình 3 — Sustainable pace như một SLO, không như một giá trị đạo đức.**

Đây là mô hình có sức thuyết phục cao nhất trong môi trường công ty Việt Nam, vì nó chuyển cuộc tranh
luận từ đạo đức và ý chí ("chịu được hay không", "có tinh thần hay không") sang **ngân sách** — thứ
thương lượng được bằng dữ liệu.

```
Đặt ngưỡng, ví dụ:
  ≤ 45 giờ/tuần tính trung bình trượt 4 tuần
  ≤ 2 tuần crunch mỗi quý, và crunch phải có ngày kết thúc viết ra
  ≥ 1 kỳ nghỉ liền 5 ngày làm việc mỗi 6 tháng, có backup có tên
  ≤ 1 tuần on-call mỗi 3 tuần

Vượt ngưỡng = tiêu error budget. Mỗi lần tiêu phải có: lý do, ai quyết,
và kế hoạch bù cụ thể có ngày.
Hết error budget = dừng nhận cam kết mới, không phải "cố thêm chút".
```

Cơ chế hiệu quả của mô hình: nó làm việc vượt ngưỡng trở thành **một quyết định có tác giả** thay vì một
sự trôi dạt không ai chịu trách nhiệm. Khái niệm error budget được lấy từ thực hành SLO trong vận hành;
chi tiết ở `06-incident-va-metrics.md`.

### Practical Framework

**Bước 1 — Bảng chỉ báo cá nhân, 5 phút mỗi thứ Sáu, ghi cùng chỗ với weekly review.**

Sáu câu, ghi số hoặc thang 1–5. Mục tiêu không phải đánh giá bản thân mà là tạo một chuỗi thời gian:

1. Tuần này tôi làm bao nhiêu giờ? (số)
2. Việc khó nhất tuần này — tôi làm nó, hay tôi đẩy nó? (làm / đẩy)
3. Khi review code tuần này, tôi đọc thiết kế hay chỉ đọc hình thức? (1–5)
4. Tôi có đề xuất hoặc phản biện điều gì trong tuần không? (số)
5. Số lần tôi nghĩ "làm cũng chẳng để làm gì"? (số)
6. Cuối tuần trước tôi có thực sự rời công việc không — không đọc Slack, không nghĩ về nó? (có / không)

Cách đọc, và đây là phần quan trọng: **giá trị tuyệt đối của một tuần không có ý nghĩa; xu hướng bốn tuần
mới có.** Ai cũng có tuần tệ. Ngưỡng hành động cụ thể: hai tuần liên tiếp trả lời "đẩy" ở câu 2, hoặc câu
5 khác 0 trong hai tuần liên tiếp — đó là tín hiệu cần **hành động cơ cấu** (đổi tải, đổi cơ chế, đối
thoại với cấp trên), không phải tín hiệu cần nghỉ một ngày. Lý do phân biệt: câu 2 và câu 5 đo hai trục
mà nghỉ ngơi không sửa được.

**Bước 2 — Thiết lập giới hạn theo ba lớp, theo thứ tự sức bền giảm dần.**

| Lớp | Ví dụ | Vì sao mạnh/yếu |
|---|---|---|
| **Lớp cơ chế** (mạnh nhất) | On-call có rotation ba người; họp với khách nước ngoài giới hạn hai tối/tuần; không deploy chiều thứ Sáu; hạn mức số P0 (mục 3) | Không phụ thuộc ý chí của bạn; tổ chức duy trì nó; nó tồn tại cả khi bạn mệt |
| **Lớp thoả thuận** | Công bố cửa sổ Reactive (mục 2); thoả thuận với cấp trên về ngưỡng escalate; thoả thuận với team về việc gì được phép gọi ngoài giờ | Có người khác giữ giúp bạn, nhưng cần được nhắc lại định kỳ |
| **Lớp cá nhân** (yếu nhất) | Không mở laptop sau 21h; thứ Bảy không đọc Slack; tắt notification | Chỉ có bạn bảo vệ, nên nó sụp đầu tiên dưới áp lực |

Lý do thứ tự này quan trọng: rất nhiều người thử lớp cá nhân trước, thất bại sau ba tuần, rồi kết luận
sai rằng "giới hạn không khả thi ở công ty này". Thực tế họ đã chọn lớp yếu nhất để chống lại một lực cơ
cấu. Với mỗi giới hạn bạn muốn có, hỏi: **có cách nào đưa nó lên lớp cơ chế?**

**Bước 3 — Script nói chuyện với cấp trên khi quá tải.**

Đây là phần khó nhất của mục này trong bối cảnh Việt Nam, vì việc nói mình quá tải dễ bị đọc là thiếu
năng lực hoặc thiếu cam kết. Cấu trúc năm phần dưới đây được thiết kế để giảm đúng rủi ro đó:

1. **Dữ liệu, không cảm xúc.** Mở đầu bằng số, không bằng "em rất mệt".
2. **Hệ quả cho tổ chức, không cho bản thân.** Đây là phần khiến cuộc nói chuyện được nghe.
3. **Đề xuất cụ thể, nhiều phương án.**
4. **Mời cấp trên quyết trade-off** thay vì xin phép.
5. **Cam kết rõ ràng cho phần bạn giữ.**

> **Linh:** Anh cho em 15 phút, em muốn nói một việc bằng số liệu chứ không phải bằng cảm giác, và em muốn
> nói sớm chứ không đợi đến lúc trượt tiến độ.
>
> **EM:** Ừ, có chuyện gì?
>
> **Linh:** Bốn tuần vừa rồi em làm trung bình 54 giờ. Trong bốn tuần đó có ba thứ đang xấu đi mà anh
> chưa thấy: số comment thiết kế của em trên PR giảm từ 6 xuống 1, hai việc khó nhất của em là thiết kế
> lại luồng reconcile và dựng load test đều bị em đẩy sang sprint sau ba lần, và tuần trước em để lọt một
> lỗi ở bước cấu hình deploy — loại lỗi ba năm nay em chưa mắc.
>
> **EM:** Nhưng anh thấy team vẫn chạy tốt mà, release vẫn đúng hạn.
>
> **Linh:** Đúng, phần nhìn thấy được vẫn chạy, vì em đang bù bằng giờ. Phần đang xấu đi là phần không ai
> thấy trong hai tháng tới: chất lượng review và hai việc thiết kế. Nếu ba tháng nữa mới thấy thì lúc đó
> nó là sự cố production trong mùa campaign, không phải là một cuộc nói chuyện như bây giờ. Em muốn nói
> khi còn xử lý được.
>
> **EM:** Vậy em cần gì?
>
> **Linh:** Em đề nghị ba việc, anh chọn giúp em ít nhất hai. Một, chuyển on-call ban đêm sang rotation
> ba người thay vì mặc định là em; em viết runbook trong hai tuần để người khác trực được. Hai, họp với
> khách Mỹ gom vào tối thứ Ba và thứ Năm; các buổi khác em gửi tài liệu trước và Nam đi thay. Ba, việc
> dashboard cho Ops chuyển sang Trang, em bàn giao trong ba ngày. Phần em cam kết giữ nguyên: thiết kế
> reconcile xong trước ngày 20, và em vẫn chịu trách nhiệm cho chất lượng release.
>
> **EM:** Cái on-call thì khó. Chưa ai xử lý được như em, đêm hôm mà người khác trực thì anh không yên
> tâm.
>
> **Linh:** Em đồng ý là hiện tại chưa ai làm được như em. Và đó chính là cái em muốn sửa, vì nó là rủi
> ro của anh chứ không chỉ của em: ba tháng tới nếu em vẫn là người duy nhất trực được thì anh có một
> điểm hỏng duy nhất, và em thì không nghỉ phép được. Em đề nghị cách chuyển tiếp: hai tuần tới em vẫn
> trực chính, nhưng mọi ca em kèm Tuấn cùng vào; từ tuần thứ ba Tuấn trực chính và em backup qua điện
> thoại. Chi phí là việc reconcile chậm khoảng hai ngày vì em mất thời gian viết runbook. Anh thấy đổi
> được không?
>
> **EM:** Được, làm thế. Nhưng vụ họp khách thì để anh xem, cái đó anh phải nói với account.
>
> **Linh:** Vâng, anh cứ xem. Em sẽ gửi anh trong hôm nay một danh sách các buổi họp đó kèm ghi chú buổi
> nào thực sự cần em quyết, buổi nào em chỉ ngồi nghe — để anh có dữ liệu khi nói với account.

Năm kỹ thuật cần thấy: (1) mở bằng dữ liệu về **đầu ra**, không về cảm giác — điều này ngăn cuộc nói
chuyện trở thành cuộc tranh luận về sức chịu đựng; (2) chuyển hệ quả về phía tổ chức và về phía rủi ro của
chính cấp trên, đó là lý do câu về điểm hỏng duy nhất có hiệu lực; (3) đưa ba phương án và yêu cầu chọn
hai — cách này khác hoàn toàn với việc xin giảm tải chung; (4) **nêu chi phí của phương án của mình một
cách chủ động** (reconcile chậm hai ngày), vì người tự nêu chi phí được tin hơn nhiều; (5) giữ nguyên một
cam kết cụ thể, để cuộc nói chuyện không bị đọc là rút lui.

**Bước 4 — Buổi "đóng sổ" bắt buộc sau mỗi giai đoạn crunch, 45 phút.**

Ba việc, theo thứ tự:

1. **Xác nhận nợ đã được trả bằng cái gì cụ thể.** Nghỉ bù thật, có ngày trên lịch, không phải "khi nào
   rảnh thì nghỉ". Nếu không trả được, ghi lại là nợ chưa trả — việc ghi lại quan trọng vì nợ không được
   gọi tên sẽ được coi như đã trả.
2. **Phục hồi hệ thống cá nhân** (mục 4, Bước rút gọn hệ thống): rà lại calendar và Slack của toàn bộ
   giai đoạn crunch để bắt các cam kết đã phát sinh; đưa danh sách về trạng thái đáng tin; ghi lại các nợ
   kỹ thuật đã tạo ra trong crunch trước khi chúng bị quên.
3. **Một câu hỏi cơ cấu:** crunch này do một biến cố hay do một cơ chế? Kiểm tra bằng cách xem ba giai
   đoạn crunch gần nhất — nếu chúng có cùng nguyên nhân (ước lượng thiếu, scope đổi giữa dòng, phụ thuộc
   bên ngoài không được quản lý), thì đây là vấn đề cơ chế và phải đưa vào retro của tổ chức, không phải
   vào kế hoạch nghỉ ngơi cá nhân. Xem `07-project-delivery.md`.

### Trade-off

| Trục | Nghiêng về bên A khi | Nghiêng về bên B khi | Ai chịu phần mất |
|---|---|---|---|
| **A: Crunch có giới hạn — B: Giữ sustainable pace tuyệt đối** | Có ngày kết thúc xác định và được viết ra; Cost of Delay là deadline cứng thật (campaign có ngày, hạn compliance, cam kết đã ký); có bù cụ thể đã lên lịch; **và** đây không phải quý thứ ba liên tiếp | "Deadline" do nội bộ tự đặt và đã dịch hai lần; team đã crunch trong hai quý gần nhất; công việc đang cần chất lượng phán đoán (thiết kế, quyết định Type 1) hơn là throughput cơ học | Crunch không có bù: trả bằng turnover, và mỗi người rời đi mang theo 3–6 tháng năng suất cộng kiến thức chưa tài liệu hoá. Từ chối mọi crunch: mất uy tín ở đúng thời điểm tổ chức cần nhất |
| **A: Lead hấp thụ áp lực để bảo vệ team — B: Truyền áp lực lên đúng nơi tạo ra nó** | Giai đoạn rất ngắn, team đang có người mới, và bạn đang mua thời gian để sửa cơ chế | Áp lực đến từ cơ chế lặp lại; hoặc bạn đã hấp thụ hơn một quý | Hấp thụ hết: bạn là điểm hỏng, và tổ chức không nhận được tín hiệu nên không sửa nguyên nhân. Truyền hết không lọc: team mất tập trung và mất niềm tin vào lead |
| **A: Nói ra khi quá tải — B: Im lặng và tự xoay** | Bạn có dữ liệu về đầu ra, có phương án, và quan hệ với cấp trên đủ để cuộc nói chuyện không bị đọc lệch | Bạn chưa có dữ liệu (khi đó việc cần làm là thu thập 2–3 tuần trước khi nói) | Im lặng: tổ chức tối ưu dựa trên giả định sai rằng tải hiện tại là bền được, và bạn trả một mình. Nói mà không có dữ liệu: bị đọc là thiếu cam kết, và lần sau nói khó hơn |
| **A: Nghỉ phép trọn tuần — B: Nghỉ lẻ từng ngày** | Cần phục hồi thật; đã có backup có tên và runbook | Đang trong giai đoạn không thể vắng; hoặc bạn chỉ cần giảm tải chứ chưa cần phục hồi sâu | Nghỉ lẻ: gần như không tạo detachment nên phục hồi rất ít — bạn tiêu ngày phép và không nhận được lợi ích |

Trục thứ nhất cần nói thẳng thay vì né. **Có những giai đoạn mà tăng cường là quyết định đúng.** Một
startup ba tuần trước vòng gọi vốn, một team e-commerce ba tuần trước campaign lớn nhất năm, một hệ thống
phải tuân thủ một quy định có ngày hiệu lực — trong các tình huống đó, việc từ chối mọi mức tăng cường
với lý do sustainable pace là tối ưu sai hàm mục tiêu, và nó làm bạn mất uy tín ở thời điểm mà uy tín được
hình thành. Bốn điều kiện để crunch là quyết định đúng, phải có **đủ cả bốn**: ngày kết thúc xác định và
được viết ra; nguyên nhân là một Cost of Delay thật chứ không phải một cam kết tuỳ ý; phần bù đã được lên
lịch **trước khi** crunch bắt đầu; và scope của crunch được cắt (không phải mọi việc đều crunch — chỉ
những việc thực sự thuộc deadline đó).

Và cái giá phải trả kể cả khi làm đúng cả bốn điều kiện, phải nói ra để không ai bị bất ngờ: 2–3 tuần sau
crunch năng suất sẽ thấp, và điều đó là bình thường chứ không phải dấu hiệu của vấn đề; nợ kỹ thuật sinh
ra trong crunch phải được ghi lại ngay lúc tạo, vì nếu không nó sẽ trở thành kiến trúc vĩnh viễn.

### Real-world Scenarios

**Tình huống A — Crunch làm đúng và crunch làm sai, cùng một công ty.**

Bối cảnh: công ty e-commerce, ba tuần trước campaign 11/11 — sự kiện chiếm phần lớn doanh thu quý.

*Cách làm sai, năm trước:* thông báo "cả team tăng cường đến khi qua campaign", không có ngày kết thúc
tường minh vì campaign kéo dài và có đuôi xử lý sau. Mọi việc trong sprint được coi là gấp, kể cả những
việc không liên quan tới campaign. Lời hứa "sau campaign sẽ nghỉ" — nhưng tháng 12 có campaign 12/12, rồi
Tết. Kết quả: hai người nghỉ việc trong tháng 1, một người trong đó là người duy nhất hiểu luồng tính
giá; các bản sửa nhanh trong crunch không được ghi lại nên tháng 3 team mất bốn tuần để hiểu lại chính
code của mình; và trong quý 1, các đề xuất kỹ thuật của team giảm về gần 0.

*Cách làm đúng, năm nay:* năm việc cụ thể, và mỗi việc đều nhắm vào một cơ chế đã hỏng năm trước.

1. **Ngày kết thúc viết ra và không dịch:** crunch từ 25/10 đến 13/11, sau đó về pace bình thường bất kể
   trạng thái. Việc còn lại sau 13/11 được xử lý theo pace thường.
2. **Cắt scope của crunch:** chỉ ba luồng thuộc campaign được crunch. Mọi việc khác giữ pace thường và
   được nói rõ là sẽ chậm — có một danh sách công bố cho stakeholder.
3. **Bù được lên lịch trước khi crunch bắt đầu:** ba ngày nghỉ cho mỗi người, ngày cụ thể đã đặt trên lịch
   trong tháng 12, được EM phê duyệt trước ngày 25/10. Cơ chế quan trọng ở đây là *trước*: một lời hứa bù
   sau crunch có xác suất được thực hiện thấp hơn nhiều so với một ngày đã nằm trên lịch.
4. **Đóng băng thay đổi lớn:** không deploy thay đổi kiến trúc trong tuần crunch; chỉ sửa và tăng capacity.
   Lý do: trong crunch, chất lượng phán đoán thấp nhất, nên đây là lúc tệ nhất để ra quyết định Type 1
   (mục 5).
5. **Một người được chỉ định theo dõi chỉ báo của team** (không phải Tech Lead, vì Tech Lead cũng đang
   trong crunch): mỗi ngày kiểm tra ba thứ — có ai làm quá 12 giờ hai ngày liên tiếp, có ai làm cả hai
   ngày cuối tuần, có ai bắt đầu im lặng trong stand-up.

Kết quả: campaign chạy được, không ai nghỉ việc trong quý sau, và các bản sửa nhanh có một danh sách 14
mục được ghi trong lúc tạo — trong đó 9 mục được xử lý trong tháng 12 và 5 mục được quyết định là chấp
nhận sống với nó, có ghi lý do.

Bài học truy được về cơ chế: khác biệt giữa hai năm không nằm ở khối lượng công việc — hai năm gần như
bằng nhau. Nó nằm ở **tính xác định**: có ngày kết thúc, có phạm vi, có phần bù đã cam kết. Bất định là
thứ tiêu năng lượng nhiều nhất, không phải khối lượng.

**Tình huống B — Một Senior ngừng đề xuất, nhìn từ ba tầng.**

Sự việc: Khoa (Senior BE, 4 năm trong công ty) trong 6 tháng qua đã thay đổi: đi đúng giờ, về đúng giờ,
làm đúng task được gán, chất lượng vẫn tốt, không phàn nàn gì. Số đề xuất kỹ thuật trong 6 tháng: 0 —
trước đó là khoảng 2–3 mỗi quý. Trong retro Khoa nói ít, và khi được hỏi ý kiến thì trả lời "cái nào cũng
được, các anh quyết đi".

*Nhìn từ IC (chính Khoa):* dữ liệu quan trọng nhất là **trục nào đang cao**. Khoa không kiệt lực — anh
ngủ đủ, làm 40 giờ, không mệt. Anh đang ở inefficacy: ba đề xuất gần nhất của anh đều được nghe, được
khen, rồi không có gì xảy ra; hai lần anh làm xong việc thì ưu tiên đổi và việc không được release. Hành
động đúng ở tầng này không phải nghỉ phép — nghỉ phép không sửa được inefficacy. Hành động đúng là chẩn
đoán bằng sáu vùng lệch (đây là lệch **control** và một phần **reward**), rồi mở một cuộc đối thoại cụ
thể: "trong ba đề xuất em đưa ra, em muốn hiểu vì sao chúng không đi tới đâu, và em muốn biết loại đề xuất
nào ở tổ chức mình thì có đường đi." Cuộc đối thoại đó có thể cho ra hai kết quả, và cả hai đều tốt hơn
hiện tại: một đường đi rõ ràng, hoặc dữ liệu rằng tổ chức này không có đường đi — thuộc phạm vi
`11-career-evolution.md`.

*Nhìn từ Tech Lead:* rủi ro lớn nhất ở tầng này là **đọc sai tín hiệu**. Một người có chất lượng thực thi
cao và số đề xuất bằng 0 rất dễ được đọc là "đã ổn định, đã chín". Tín hiệu đúng để đọc: khi chất lượng
thực thi vẫn cao mà phạm vi hành động tự nguyện co lại, đó là inefficacy hoặc cynicism, không phải sự hài
lòng. Hành động đúng: một buổi 1-1 hỏi về nguyên nhân cụ thể (không hỏi "em có ổn không" — câu đó gần
như luôn nhận được câu trả lời "em ổn"), và kiểm tra một chỉ số khách quan: trong hai quý qua, tỉ lệ việc
Khoa làm xong mà được release và được dùng là bao nhiêu. Tech Lead xử lý sai khi họ động viên Khoa hãy
đề xuất nhiều hơn — điều đó đặt vấn đề lên phía Khoa trong khi cơ chế nằm ở phía tổ chức, và nó làm tăng
cynicism.

*Nhìn từ Manager:* ở tầng này, việc một người ngừng đề xuất là **tín hiệu về hệ thống, không phải về một
người**. Manager có ba dữ liệu mà hai tầng dưới không có: tổng số đề xuất/RFC của cả team theo quý; tỉ lệ
việc được làm xong nhưng không release; và số lần ưu tiên bị đổi giữa dòng trong quý. Nếu ba con số này
đang xấu, nguồn của vấn đề nằm ở vùng lệch **control** và **fairness** ở cấp tổ chức — cụ thể: đề xuất
không có đường đi tới quyết định, và công việc bị đổi ưu tiên liên tục nên không ai thấy kết quả của việc
mình làm. Hành động đúng ở tầng này là cơ chế: một đường đi công khai cho đề xuất kỹ thuật (RFC có người
quyết và có SLA trả lời), một hạn mức cho việc đổi ưu tiên giữa sprint, và làm cho kết quả của việc đã
release hiển thị cho người làm. Manager xử lý sai khi họ tổ chức một buổi team building hoặc gửi một email
về tinh thần chủ động — can thiệp ở tầng cá nhân cho một vấn đề ở tầng cơ chế, và tác dụng thực tế là tăng
cynicism vì nó ngầm nói rằng vấn đề nằm ở thái độ của người lao động.

Điểm mấu chốt của ba góc nhìn: cùng một hiện tượng — số đề xuất bằng 0 — nhưng nếu IC đọc nó là "mình cần
nghỉ", Tech Lead đọc nó là "team đã ổn định", và Manager đọc nó là "cần tăng tinh thần", thì cả ba đều can
thiệp sai và hiện tượng sẽ tiến tới bước tiếp theo là một đơn xin nghỉ việc — thời điểm mà mọi người sẽ
bất ngờ.

**Một ghi chú ngắn về mức nghiêm trọng.** Toàn bộ mục này nói về vùng còn xử lý được bằng thiết kế công
việc. Có một vùng khác: khi các dấu hiệu kéo dài nhiều tháng và đã lan ra ngoài công việc — ảnh hưởng tới
giấc ngủ, tới sức khoẻ thể chất, hoặc bạn nhận ra mình không còn quan tâm tới những việc trước đây mình
quan tâm. Đó là vấn đề sức khoẻ, và cách xử lý đúng là tìm hỗ trợ chuyên môn (bác sĩ, chuyên gia tâm lý),
giống như bạn không tự chữa một chấn thương dây chằng bằng cách đọc tài liệu. Không có framework quản trị
nào thay thế được việc đó, và việc tìm hỗ trợ không phải là dấu hiệu của sự yếu kém — nó là chẩn đoán
đúng tầng của vấn đề.

### Best Practices

**Đo xu hướng, không đo mức.** Lý do: mức tuyệt đối khác nhau rất nhiều giữa người và giữa giai đoạn đời,
nên nó không có giá trị chẩn đoán. Xu hướng bốn tuần của cùng một người thì có. Đây cũng là lý do bảng chỉ
báo phải được ghi đều đặn ngay từ giai đoạn bình thường — nếu chỉ bắt đầu ghi khi đã có vấn đề, bạn không
có đường cơ sở để so.

**Đặt giới hạn ở lớp cơ chế trước khi đặt ở lớp cá nhân.** Lý do: lớp cá nhân chỉ có bạn bảo vệ nên nó
sụp đầu tiên dưới áp lực, và khi nó sụp bạn kết luận sai rằng giới hạn là không khả thi.

**Nói khi dữ liệu còn tốt, không nói khi đã trượt.** Lý do: cùng một nội dung, nếu nói trước khi trượt thì
là một cuộc đối thoại về thiết kế công việc; nếu nói sau khi trượt thì bị đọc là bào chữa. Đây là hiệu ứng
framing thuần và nó mạnh hơn nội dung của lời nói.

**Điều kiện để nghỉ phép được là thiết kế, không phải ý chí.** Lý do: bạn chỉ tắt máy được nếu có runbook
và một người backup có tên. Vì vậy việc chuẩn bị để nghỉ phép phải bắt đầu ba tuần trước ngày nghỉ, và nó
là công việc kỹ thuật (viết runbook, chuyển giao) chứ không phải một quyết định.

**Bảo vệ giấc ngủ trước khi bảo vệ số giờ.** Lý do: working memory là input của mọi việc còn lại, nên một
giờ mất ngủ có chi phí lan rộng hơn một giờ làm thêm. Áp dụng cụ thể: sáng sau một đêm họp khách nước
ngoài phải được coi là tài nguyên đã bị tiêu — đặt Admin hoặc Reactive vào đó, không đặt Deep Work
(mục 2).

**Tách hai câu "tôi mệt" và "việc này vô nghĩa" khi tự chẩn đoán.** Lý do: chúng là hai trục khác nhau với
hai can thiệp khác nhau, và nhầm chúng dẫn tới việc tiêu ngày phép cho một vấn đề mà nghỉ ngơi không sửa
được.

**Là lead, đừng làm gương bằng số giờ.** Lý do: team đọc hành vi của lead như tín hiệu về kỳ vọng thật,
đáng tin hơn mọi lời nói. Nếu bạn gửi PR lúc 1h sáng, team học được rằng đó là mức để được công nhận, bất
kể bạn nói gì trong buổi họp. Nếu buộc phải làm muộn, dùng tính năng hẹn giờ gửi.

### Anti-patterns

**Coi số giờ làm là bằng chứng của cam kết.**
Cơ chế: hệ thống này chọn lọc theo **khả năng chịu đựng và hoàn cảnh cá nhân** thay vì theo năng lực. Hệ
quả nối tiếp: những người có ràng buộc gia đình (thường là người có con nhỏ) bị loại khỏi cơ hội một cách
hệ thống dù năng lực cao hơn; và những người giỏi việc tối ưu chứ không giỏi việc ngồi lâu sẽ rời đi. Về
lâu dài tổ chức có một tập người chịu được, và đó không phải cùng một tập với người làm tốt.
Dấu hiệu sớm: việc ở lại muộn được nhắc trong đánh giá; câu "bạn kia rất nhiệt tình, hôm nào cũng về
muộn" xuất hiện trong cuộc bàn về promotion; không ai đo đầu ra.

**Crunch không có ngày kết thúc — crunch thường trực.**
Cơ chế: bất định là thứ tiêu năng lượng nhiều hơn khối lượng. Một crunch ba tuần có ngày kết thúc rõ được
chịu đựng tốt; một crunch "đến khi nào xong" ở mức thấp hơn lại phá nhiều hơn, vì không thể lập kế hoạch
phục hồi. Thêm nữa, crunch thường trực làm mất công cụ crunch — khi thật sự cần tăng cường, không còn dự
trữ nào.
Dấu hiệu sớm: không ai nói được crunch này kết thúc ngày nào; lời hứa bù không có ngày cụ thể; ba giai
đoạn crunch gần nhất có cùng nguyên nhân.

**Lead hấp thụ toàn bộ áp lực và gọi đó là bảo vệ team.**
Cơ chế: hai tác hại đồng thời. Bạn trở thành điểm hỏng duy nhất (mục 1, martyr complex); và quan trọng
hơn, **tổ chức không nhận được tín hiệu rằng cơ chế đang sai**, nên nguyên nhân tồn tại vô hạn. Bạn đang
trả một khoản nhỏ mỗi tuần để duy trì một khoản lớn.
Dấu hiệu sớm: team không biết dự án đang gặp vấn đề gì; cấp trên tin rằng tải hiện tại là bền được; bạn
là người duy nhất làm việc cuối tuần.

**Nghỉ phép danh nghĩa.**
Cơ chế: không có detachment thì gần như không có phục hồi, nên bạn tiêu ngày phép mà không nhận được lợi
ích — và tệ hơn, bạn xác nhận với tổ chức rằng bạn vẫn xử lý được trong lúc nghỉ, nên lần sau kỳ vọng đó
được giữ nguyên.
Dấu hiệu sớm: bạn trả lời Slack trong ngày phép; bạn không có backup có tên; bạn nói "chỉ xem cho yên
tâm".

**Xử lý burnout bằng phúc lợi bề mặt khi nguyên nhân là control hoặc fairness.**
Cơ chế: can thiệp ở tầng cá nhân cho một vấn đề ở tầng cơ chế truyền đi một thông điệp ngầm rằng vấn đề
nằm ở khả năng chịu đựng của người lao động. Thông điệp đó **làm tăng trục cynicism** — tức là can thiệp
này không trung tính, nó có hại.
Dấu hiệu sớm: có ngày team building nhưng ưu tiên vẫn bị đổi ba lần mỗi sprint; có phòng nghỉ đẹp nhưng
không ai được quyết định gì về công việc của mình.

**Đọc dấu hiệu inefficacy thành sự chín chắn.**
Cơ chế: một người ngừng phản biện và ngừng đề xuất trở nên dễ quản lý hơn, nên tín hiệu này có động cơ bị
đọc theo hướng tích cực. Chi phí trả sau: bạn mất đúng nhóm người có khả năng nhìn thấy vấn đề trước khi
nó thành sự cố.
Dấu hiệu sớm: số đề xuất kỹ thuật của team giảm theo quý mà không ai coi đó là chỉ số; các buổi retro
ngày càng ngắn và "không có gì để nói".

**Dùng sustainable pace làm lý do từ chối mọi giai đoạn tăng cường.**
Cơ chế: cần nêu anti-pattern đối xứng này để mục 7 không trở thành một khẩu hiệu. Tổ chức có những thời
điểm mà Cost of Delay là thật và rất lớn; người từ chối tăng cường trong mọi điều kiện sẽ bị đọc là không
chia sẻ rủi ro, và điều đó có chi phí thật về uy tín và cơ hội — đặc biệt ở vai lead, nơi việc chia sẻ
rủi ro với tổ chức là một phần của vai.
Dấu hiệu sớm: bạn từ chối mà không phân tích được Cost of Delay của việc đó; bạn dùng cùng một lập luận
cho campaign 11/11 và cho một deadline nội bộ đã dịch hai lần.

### Khi nào KHÔNG nên áp dụng

**Crunch có giới hạn rõ, có bù, và có Cost of Delay thật.** Đây là điều kiện biên chính của toàn mục, và
nó phải được nói thẳng: trong tình huống đó, việc giữ ngưỡng sustainable pace là tối ưu sai hàm mục tiêu.
Bốn điều kiện đủ đã nêu ở phần Trade-off, và cần nhắc lại rằng phải có **đủ cả bốn** — thiếu một điều kiện
là dấu hiệu rằng đây không phải crunch mà là sự trôi dạt được gọi bằng tên khác. Cái giá phải trả kể cả
khi làm đúng: 2–3 tuần năng suất thấp sau đó, và một danh sách nợ kỹ thuật phải được ghi trong lúc tạo.

**Trong giai đoạn startup có ràng buộc runway thật.** Nếu công ty còn hai tháng tiền, việc tối ưu
sustainable pace là tối ưu cho một tương lai có thể không tồn tại. Nhưng điều kiện đi kèm là bắt buộc và
thường bị bỏ: **phải nói rõ với team đây là gì, kéo dài bao lâu, và điều gì sẽ đổi nếu tình hình đổi**, và
những người không thể chọn giai đoạn đó (vì hoàn cảnh gia đình, sức khoẻ) phải có một lựa chọn khác mà
không bị coi là thiếu cam kết. Crunch mà không nói rõ bối cảnh không phải crunch — nó là bóc tách sự tin
tưởng.

**Trong incident và trong ca on-call.** Trong 24–72 giờ của một incident nghiêm trọng, các ngưỡng bị treo
và điều đó là đúng. Nhưng phải có một luật cứng đi kèm, không thương lượng: **sau một ca đêm, người đó
không thao tác trên production và không ra quyết định Type 1 trong ngày hôm sau.** Lý do có cơ chế rõ:
thiếu ngủ làm giảm đúng hai năng lực cần cho hai việc đó, và đây là nguồn phổ biến của sự cố thứ hai —
sự cố do việc xử lý sự cố thứ nhất gây ra. Xem `06-incident-va-metrics.md`.

**Trong 2–3 tháng đầu ở một vai trò mới.** Mức nỗ lực cao hơn bình thường trong giai đoạn này là đầu tư
có lợi nhuận, vì bối cảnh học được lúc đó có giá trị kép: nó vừa là kiến thức, vừa là uy tín ban đầu —
và cả hai đều khó có được sau này. Điều kiện biên: phải có ngày kết thúc mà bạn tự đặt (ví dụ 10 tuần) và
phải kiểm tra lại bằng bảng chỉ báo, vì đây cũng chính là giai đoạn mà nhiều người mới lên lead bước vào
một pace mà họ không bao giờ ra khỏi.

**Khi vấn đề thật là công việc không tạo ra tác động (inefficacy), không phải quá tải.** Trong trường hợp
này, giảm giờ làm và nghỉ nhiều hơn gần như không có tác dụng, và việc áp dụng các biện pháp về năng lượng
sẽ trì hoãn cuộc đối thoại đúng. Can thiệp đúng nằm ở chỗ khác: nối việc với kết quả (làm cho người làm
thấy được việc mình làm được dùng), giảm priority thrashing (mục 3), trả lại quyền quyết định trong phạm
vi họ chịu hệ quả (mục 1), hoặc — nếu tổ chức không đổi được sau hai lần đối thoại có dữ liệu — đổi việc.
Dấu hiệu chẩn đoán: sau một kỳ nghỉ thật sự có detachment, cảm giác đó quay lại trong vòng một tuần.

**Khi bạn đang áp mô hình này cho người khác mà không có quyền đổi điều kiện công việc của họ.** Nếu bạn
là Tech Lead và phát hiện một người trong team có ba trục đang xấu, việc nói chuyện với họ về quản lý năng
lượng cá nhân trong khi bạn không đổi được tải, không đổi được ưu tiên và không đổi được cơ chế on-call sẽ
làm tăng cynicism — vì nó đặt vấn đề lên phía họ. Trong điều kiện đó, việc đúng cần làm trước là mang dữ
liệu lên tầng có quyền đổi cơ chế, và nói với người đó rằng bạn đang làm việc đó cùng với thời hạn cụ thể.

---

## Tự kiểm tra

Tám câu hỏi dưới đây không phải câu hỏi lý thuyết. Mỗi câu yêu cầu bạn mở một công cụ thật (calendar,
Jira, Slack, git) và tìm một con số. Nếu bạn không tìm được con số, đó cũng là một câu trả lời.

1. **Mở calendar bốn tuần vừa qua và đếm:** bạn có bao nhiêu block liên tục dài hơn 90 phút không bị
   cắt? Bao nhiêu giờ bạn không truy được đầu ra? Việc quan trọng nhất của quý này được làm vào những
   khung giờ nào — và đó là khung giờ có chất lượng attention cao nhất hay thấp nhất của bạn?

2. **Lấy ba việc bạn dành nhiều thời gian nhất tuần qua và thử truy mỗi việc lên tới một Business Goal
   của quý.** Việc nào không truy được? Nó thuộc loại nào: việc duy trì bắt buộc, việc bạn làm vì thú vị,
   hay việc bạn làm vì có người nhắc to hơn những người khác?

3. **Mở biên bản hoặc lịch sử chat của ba buổi họp gần nhất, đếm số câu bạn đã nói dạng "để anh/em sẽ...",
   rồi đối chiếu với hệ thống ghi chú của bạn.** Bao nhiêu phần trăm không tồn tại ở đâu ngoài đầu bạn?
   Trong số đó, có việc nào đang chặn một người khác mà bạn chưa biết?

4. **Chọn quyết định kỹ thuật gần nhất mà bạn dành nhiều thời gian nhất, và quyết định khó đảo nhất bạn
   đã ra trong sáu tháng qua.** Hai quyết định đó có phải cùng một quyết định không? Nếu không, bạn đã
   chi bao nhiêu cho mỗi cái, và tỉ lệ đó có hợp lý theo ma trận Reversibility × Blast radius?

5. **Kể ra một loại vấn đề kỹ thuật mà bạn lần đầu gặp trong 12 tháng qua** — loại vấn đề, không phải công
   nghệ. Nếu không kể được, hãy trả lời câu tiếp: trong công việc hiện tại của bạn có tồn tại một vùng
   thực hành nào cho khoảng trống lớn nhất trong skill map của bạn không, và nếu không thì bạn chọn đường
   nào trong ba đường (tạo cơ hội / học chậm bên ngoài / đổi môi trường)?

6. **Mở lịch sử review code của bạn ba tháng trước và so với tháng này:** số comment trung bình mỗi PR có
   giảm không, và nội dung comment có chuyển từ tầng thiết kế xuống tầng hình thức không? Trong hai sprint
   gần nhất, việc khó nhất trong backlog của bạn — bạn làm nó hay đẩy nó?

7. **Kiểm tra ba giai đoạn tăng cường gần nhất của bạn hoặc của team:** mỗi giai đoạn có ngày kết thúc
   được viết ra không? Phần bù có được lên lịch trước khi bắt đầu không? Ba giai đoạn đó có cùng nguyên
   nhân không? Nếu có cùng nguyên nhân, đó là vấn đề cơ chế — và bạn đã đưa nó vào retro nào của tổ chức?

8. **Với vùng ownership rộng nhất bạn đang giữ:** bạn có đủ ba thành tố thông tin, quyền và hệ quả không?
   Nếu thiếu quyền, bạn đã đàm phán mấy lần và bằng dữ liệu gì? Và bạn có viết ra được tiêu chí chuyển
   giao cho vùng đó chưa — hay nó sẽ ở với bạn vô hạn?

## Liên kết chương khác

- [`00-nen-tang-leadership.md`](/series/engineering-leedership/00-nen-tang-leadership/) — Nền tảng của Ownership, Accountability vs
  Responsibility, Influence và Trust. Chương này giả định các khái niệm đó; nếu phần phân biệt "tôi chịu
  hệ quả" và "tôi sẽ làm" ở mục 1 gây thắc mắc, đọc phần Accountability vs Responsibility ở `00`.
- [`02-communication.md`](/series/engineering-leedership/02-communication/) — Mọi script trong chương này là ứng dụng của các kỹ thuật
  ở `02`: Difficult Conversations (nói với cấp trên khi quá tải, khi thiếu quyền), Stakeholder Management
  (đàm phán ưu tiên với PO), và Technical Writing (chuyển nội dung họp sang bất đồng bộ để giảm họp đêm và
  giảm áp lực tiếng Anh nói).
- [`03-team-leadership.md`](/series/engineering-leedership/03-team-leadership/) — Delegation là cơ chế duy nhất để thoát khỏi trạng thái
  martyr complex ở mục 1 và trạng thái quá tải ở mục 7; danh sách Waiting-for ở mục 4 là điều kiện kỹ
  thuật để Delegation hoạt động. Chương này cũng xử lý khoảng cách tuổi và thâm niên trong uy tín kỹ
  thuật, xuất hiện ở tình huống của Trang trong mục 2.
- [`04-decision-making.md`](/series/engineering-leedership/04-decision-making/) — Mở rộng mục 5 lên cấp team và tổ chức: Decision
  Matrix, Risk Assessment, Decision Log, và các framework prioritization ở mục 3 khi chúng được dùng
  trong một nhóm thay vì một người.
- [`07-project-delivery.md`](/series/engineering-leedership/07-project-delivery/) — Estimation, Roadmap và Risk Management. Khi cùng
  một nguyên nhân xuất hiện ở ba giai đoạn crunch (mục 7, Bước 4) hoặc khi vấn đề thật là capacity chứ
  không phải thứ tự (mục 3), lời giải nằm ở chương này, không ở Level 1.
- [`11-career-evolution.md`](/series/engineering-leedership/11-career-evolution/) — Khi các dấu hiệu ở chương này không phải dữ liệu về
  bạn mà là dữ liệu về tổ chức: ownership giả không sửa được sau nhiều lần đàm phán, không còn loại vấn đề
  mới trong nhiều quý, hoặc inefficacy có nguồn ở cơ chế mà tổ chức không đổi. Chương này cũng xử lý lựa
  chọn IC track vs Manager track làm cơ sở cho bảng năng lực ở mục 6.
- [`06-incident-va-metrics.md`](/series/engineering-leedership/06-incident-va-metrics/) — Điều kiện biên của gần như mọi thực hành
  trong chương này là "trừ khi đang có incident". Chương đó nói cách vận hành trong chế độ ngoại lệ, và
  cách chuyển các quyết định chữa cháy về lại chế độ bình thường.
