+++
title = "Level 2A — Communication: Lắng nghe, Feedback, One-on-One và Stakeholder Management"
date = "2026-08-01T10:00:00+07:00"
draft = false
tags = ["engineering-leadership", "management", "career"]
series = ["Engineering Leadership & Technical Management"]
+++

# Level 2A — Communication

Một ví điện tử ở Hà Nội, 40 engineer. Sprint review thứ Sáu, PO nói: "tính năng hoàn tiền phải xử lý
được trường hợp giao dịch một phần". Tech Lead ghi vào notebook. Thứ Hai, Tech Lead nói lại với hai
engineer: "hoàn tiền cần hỗ trợ partial". Engineer hiểu "partial" là hoàn một phần **số tiền**. PO
nói "một phần" là hoàn một phần **danh sách item trong đơn**. Ba tuần sau, feature lên staging, PO
mở ra và nói "cái này không phải cái anh nói". Rework 9 ngày công. Không ai nói dối, không ai lười,
không ai kém năng lực. Một chuỗi bốn lần mã hoá — giải mã đã làm mất phần thông tin quan trọng nhất
của thông điệp, và không có bước nào trong chuỗi đó kiểm tra xem phần bị mất là gì.

Đây là hình dạng thật của cái mà nhiều tổ chức gọi là "vấn đề con người". Giao tiếp không phải một
phẩm chất tính cách. Nó là **cơ chế truyền tín hiệu trong một hệ thống có nhiễu và có mất mát thông
tin**: mỗi lần một ý định đi từ đầu người này sang đầu người khác, nó phải được nén thành ngôn ngữ,
đi qua một kênh có băng thông giới hạn, rồi được giải nén bằng một bộ mã khác — bộ mã của người nhận,
gồm kinh nghiệm, giả định và ngữ cảnh khác với người gửi. Mất mát ở mỗi bước là mặc định, không phải
ngoại lệ.

Hệ quả thực dụng: phần lớn "xung đột", "thái độ", "không hợp tác", "team không chịu nghe" trong dự án
kỹ thuật là **lỗi truyền tin có thể truy vết** — truy được ai gửi gì, qua kênh nào, người nhận giải
mã thành gì, và ở bước nào không có ai kiểm tra. Truy vết được thì sửa được bằng thiết kế: đổi kênh,
đổi thứ tự thông tin, thêm bước xác nhận, ghi lại thành văn bản. Không truy vết được thì tổ chức quy
về tính cách, và cái gì quy về tính cách thì không sửa được — chỉ còn cách thay người.

Chương này nằm ở tầng **People** của chuỗi tư duy, nhưng nó là điều kiện tồn tại của tầng Process và
Feedback. Một process chỉ hoạt động nếu các thông điệp trong nó (cam kết, tiêu chí, trạng thái, rủi
ro) truyền được nguyên vẹn tới người cần biết, đúng lúc còn kịp làm gì đó. Tám chủ đề dưới đây là tám
kênh khác nhau của cùng một hệ thống truyền tin, xếp theo thứ tự từ mô hình nền tới các kênh có băng
thông cao nhất và bền nhất.

Mục lục nội bộ:

1. [Vì sao giao tiếp thất bại — mô hình nền](#1-vì-sao-giao-tiếp-thất-bại--mô-hình-nền)
2. [Active Listening](#2-active-listening)
3. [Feedback](#3-feedback)
4. [Difficult Conversations](#4-difficult-conversations)
5. [One-on-One](#5-one-on-one)
6. [Stakeholder Management](#6-stakeholder-management)
7. [Presentation và trình bày kỹ thuật cho người không kỹ thuật](#7-presentation-và-trình-bày-kỹ-thuật-cho-người-không-kỹ-thuật)
8. [Technical Writing](#8-technical-writing)
9. [Tự kiểm tra](#tự-kiểm-tra)
10. [Liên kết chương khác](#liên-kết-chương-khác)

---

## 1. Vì sao giao tiếp thất bại — mô hình nền

### Problem Statement

Trong dự án hoàn tiền ở đầu chương, nếu chỉ dừng ở kết luận "lần sau nói rõ hơn" thì ba tháng sau sự
việc tương tự sẽ xảy ra ở một feature khác. Muốn sửa, phải đo. Dưới đây là các hiện tượng quan sát
được của một hệ thống truyền tin đang mất mát cao — tất cả đều đếm được từ Jira, Slack và Git, không
cần khảo sát cảm nhận:

- **Rework do hiểu sai yêu cầu.** Đếm số ticket bị reopen hoặc có commit "sửa lại theo yêu cầu mới"
  mà requirement không thay đổi trong tài liệu. Ở ví điện tử trên: 6 sprint gần nhất có 14 ticket
  dạng này, tổng 41 ngày công, tương đương 11% capacity.
- **Số lần một quyết định phải quyết lại.** Một quyết định đã chốt trong họp nhưng hai tuần sau lại
  bị mở ra tranh luận là dấu hiệu quyết định đó chưa từng được truyền thành công tới người thực thi
  hoặc chưa được ghi lại.
- **Signal latency của tin xấu.** Đo khoảng thời gian từ lúc người đầu tiên trong team biết một rủi
  ro (tìm trong Slack cá nhân, commit message, comment) tới lúc người có quyền xử lý biết. Con số
  này là chỉ số sức khoẻ giao tiếp mạnh nhất trong một tổ chức kỹ thuật. Ở nhiều ODC, nó là 2–3
  tuần, và điểm cuối luôn là buổi họp với khách.
- **Độ dài thread trước khi ra được một quyết định.** Một thread Slack 60 message không kết luận là
  bằng chứng của việc chọn sai kênh: vấn đề cần băng thông cao đang bị nhồi vào kênh băng thông thấp.
- **Tần suất câu "tôi tưởng là..."** trong retrospective và postmortem. Mỗi lần câu này xuất hiện,
  có một giả định đã tồn tại trong đầu một người mà không tồn tại trong đầu người khác, và không có
  bước nào phát hiện ra.
- **Tỉ lệ người trong họp không nói gì.** Nếu một buổi họp 9 người mà 3 người phát biểu 90% thời
  lượng, 6 người còn lại đang là kênh chỉ có một chiều. Thông tin họ có không vào được hệ thống.

Nếu không có mô hình về cơ chế thất bại, tổ chức sẽ dùng ba biện pháp phản xạ và cả ba đều làm tệ
hơn: họp nhiều hơn (tăng chi phí, giảm thời gian làm việc sâu), viết dài hơn (không ai đọc), và thêm
người trung gian (thêm một tầng nén thông tin nữa).

### First Principles

**Cơ chế một: kênh có nhiễu và bộ mã không giống nhau.** Mô hình Shannon–Weaver (lý thuyết thông tin,
công bố công khai từ 1948) mô tả truyền tin gồm: nguồn → mã hoá → kênh (có nhiễu) → giải mã → đích.
Hai điểm quan trọng cho công việc lead. Thứ nhất, **thông điệp nhận được không bao giờ đồng nhất với
thông điệp gửi đi**; kỳ vọng ngược lại là kỳ vọng sai về mặt vật lý. Thứ hai, cách duy nhất để chống
nhiễu là **redundancy và error detection** — gửi lại bằng dạng khác, và có cơ chế kiểm tra phía nhận.
Trong hàng không và y tế, cơ chế đó có tên: read-back. Người nhận đọc lại lệnh bằng lời của mình.
Trong phần mềm, chúng ta hầu như không làm, rồi ngạc nhiên khi mất tin.

Nhưng phần khó hơn nhiễu là **bộ mã**. Mã hoá và giải mã dùng một từ điển gồm kinh nghiệm, ngữ cảnh
và giả định. "Partial refund", "gần xong", "cái này đơn giản", "làm nhanh thôi" — mỗi cụm này giải
nén ra kết quả khác nhau tuỳ từ điển của người nghe. Càng khác biệt về nền tảng (Backend vs Business,
engineer vs founder, Việt vs Nhật), từ điển càng lệch, và mất mát càng lớn dù kênh sạch.

**Cơ chế hai: curse of knowledge.** Khi một người đã biết một thứ, họ mất khả năng mô phỏng trạng
thái không biết. Đây là giới hạn nhận thức đã được chứng minh trong nhiều nghiên cứu công khai (thí
nghiệm cổ điển: người gõ nhịp một bài hát quen luôn đánh giá cao mức độ người nghe nhận ra được).
Với một Tech Lead, đây là nguồn thất bại lớn nhất khi giao việc: bạn nói "làm cái sync service như
service bên payment" và tin rằng mình đã nói đủ, vì trong đầu bạn cụm đó kéo theo mười quyết định
kiến trúc mà bạn đã từng ra. Người nhận không có mười quyết định đó. Họ nghe một câu.

**Cơ chế ba: illusion of transparency.** Người gửi tin đánh giá quá cao mức độ trạng thái nội tâm và
ý định của mình được người khác nhìn thấy (nghiên cứu công khai của Gilovich, Savitsky và cộng sự).
Hệ quả cụ thể: bạn nói "chỗ này anh thấy hơi lo" và nghĩ rằng mình đã phát tín hiệu ngăn chặn rõ
ràng; người nghe ghi nhận đó là một nhận xét nhẹ. Bạn phê bình nhẹ để giữ mặt và tin rằng thông điệp
"đây là vấn đề nghiêm trọng" đã đến; nó không đến. Đây là cơ chế giải thích tại sao đa số Tech Lead
nghĩ họ đã feedback rồi, trong khi người nhận nghĩ chưa từng bị feedback.

**Cơ chế bốn: inferential distance.** Để đi từ dữ liệu tới kết luận của bạn cần một chuỗi suy luận có
n bước. Nếu người nghe không có bước 1 đến n-1, kết luận của bạn nghe như một sở thích cá nhân, hoặc
tệ hơn, như thái độ. Khi một Senior nói "không dùng microservice cho cái này", chuỗi suy luận đằng
sau có thể gồm 8 bước về team size, operational maturity, network failure mode, chi phí distributed
transaction. Người nghe thiếu 8 bước đó chỉ nhận được: "anh ấy không thích microservice". Xung đột
kiến trúc trong team phần lớn là xung đột về inferential distance chưa được thu hẹp, không phải xung
đột về kỹ thuật.

**Cơ chế năm: kênh có băng thông và độ trễ khác nhau, và không thể thay thế lẫn nhau.** Nói mặt đối
mặt là kênh băng thông rất cao (ngôn từ, giọng, mặt, khả năng hỏi lại ngay) nhưng độ bền bằng không
và chi phí đồng bộ cao (n người phải rảnh cùng lúc). Văn bản có băng thông thấp hơn cho phần cảm xúc
và ngữ cảnh, nhưng bền, tra cứu được, và chi phí mở rộng gần bằng không (một lần viết, n người đọc,
đọc được sau ba năm). Sai lầm hệ thống hay gặp nhất trong công ty phần mềm là **dùng kênh sai cho
loại thông điệp**: quyết định kiến trúc chỉ tồn tại trong một cuộc gọi; tin nhắn nhạy cảm về hiệu
suất làm việc gửi qua chat; cập nhật trạng thái định kỳ tổ chức thành họp 45 phút cho 9 người.

**Cơ chế sáu: nén thông tin qua tầng tổ chức.** Mỗi lần thông điệp đi qua một tầng, người ở tầng đó
phải nén để phù hợp thời lượng và mối quan tâm của tầng trên. Việc nén không xoá đều: nó giữ lại kết
luận, xoá **điều kiện và mức bất định**. "Chúng ta có thể xong trước 30/9 nếu môi trường của khách
sẵn sàng trong tuần này và không có thay đổi scope" đi qua ba tầng sẽ đến CEO dưới dạng "xong trước
30/9". Không ai nói dối. Đây là mất mát cấu trúc: điều kiện là phần đắt nhất để truyền và là phần
đầu tiên bị bỏ. Với một tổ chức 4 tầng và mỗi tầng mất 20% ngữ cảnh, thông tin còn lại ở đỉnh là
0.8⁴ ≈ 41%.

### Mental Model

**Model 1 — Protocol stack: chọn giữa UDP và TCP cho mỗi thông điệp.** Một tin nhắn Slack broadcast
là UDP: gửi ra, không đảm bảo tới, không đảm bảo hiểu, không có ack. Điều đó ổn với thông tin có giá
trị thấp và mất được (thông báo quán ăn trưa). Với thông điệp mà chi phí mất mát cao — thay đổi
requirement, cam kết ngày, tiêu chí chấp nhận — bạn cần TCP: có handshake (xác nhận đã hiểu), có
sequence (ghi lại thứ tự quyết định), có retransmit (nhắc lại ở kênh khác). Câu hỏi thực dụng trước
mỗi thông điệp quan trọng: **"nếu thông điệp này mất mà tôi không biết là nó mất, chi phí là bao
nhiêu?"** Chi phí cao thì phải có ack tường minh.

**Model 2 — Lossy compression có kiểm soát.** Bạn không thể truyền toàn bộ ngữ cảnh; nén là bắt buộc.
Câu hỏi không phải "nén hay không" mà là "nén bằng codec nào, và giữ lại gì". Nguyên tắc: khi nén cho
tầng trên, **giữ lại điều kiện và mức bất định, hy sinh chi tiết kỹ thuật**. Cụ thể: thay "xong ngày
30/9" bằng "70% khả năng xong 30/9, phụ thuộc môi trường khách trước 5/9; nếu trượt mốc đó thì
15/10". Đây là dạng nén giữ được thông tin dùng để ra quyết định.

**Model 3 — Common ground như một cache cần warm-up.** Hai người làm cùng nhau lâu có một cache ngữ
cảnh chung lớn; nói ba từ là hiểu. Người mới, người ở bộ phận khác, khách nước ngoài có cache lạnh.
Chi phí giao tiếp tỉ lệ nghịch với độ ấm của cache. Sai lầm phổ biến: dùng cùng một cách nói với
người có cache ấm và người có cache lạnh. Với cache lạnh, phải trả chi phí warm-up (nêu bối cảnh,
định nghĩa thuật ngữ, nói cả chuỗi suy luận), và chi phí này là đầu tư một lần chứ không phải phí.

### Practical Framework

Năm bước dưới đây áp cho mọi thông điệp có chi phí mất mát cao. Với thông điệp thường ngày thì không
cần — dùng nó cho tất cả là lãng phí và làm người xung quanh mệt.

**Bước 1 — Xác định audience thật (không phải người có mặt).** Viết ra: ai nghe, họ đã biết gì, họ
đang tối ưu điều gì, cache của họ ấm hay lạnh về chủ đề này. Với thông điệp gửi nhiều nhóm, xác định
nhóm nào là primary. Output: một dòng cho mỗi nhóm.

**Bước 2 — Mục tiêu hành vi, không phải mục tiêu thông tin.** Câu hỏi bắt buộc: *sau thông điệp này,
tôi muốn ai làm gì khác đi trước ngày nào?* "Để họ biết" không phải mục tiêu — nó không kiểm tra
được. Ba dạng mục tiêu hành vi hợp lệ: ra một quyết định (cần gì để quyết), thay đổi một hành vi (bắt
đầu/ngừng làm gì), hoặc điều chỉnh kỳ vọng (chấp nhận mốc mới). Output: một câu dạng "X làm Y trước
Z".

**Bước 3 — Chọn kênh theo bảng.** Không ứng biến, dùng bảng dưới. Sai kênh làm hỏng thông điệp đúng.

| Loại thông điệp | Kênh nên dùng | Vì sao |
|---|---|---|
| Thay đổi requirement, tiêu chí chấp nhận | Văn bản + xác nhận lại trong họp | Cần bền và cần ack; sai thì đắt |
| Quyết định kiến trúc | RFC/ADR viết trước, họp để phản biện, ghi lại kết luận | Cần chuỗi suy luận, cần người sau 2 năm đọc được |
| Tin xấu về tiến độ | Nói trực tiếp trước, văn bản ngay sau | Kênh nói xử lý phản ứng; văn bản chốt cam kết |
| Feedback về hành vi | 1-1 mặt đối mặt | Cần băng thông cảm xúc và khả năng đối thoại |
| Cập nhật trạng thái định kỳ | Văn bản bất đồng bộ | Không cần đồng bộ; họp cho việc này là thuế |
| Brainstorm, thu hẹp inferential distance | Đồng bộ, có bảng vẽ | Cần vòng hỏi–đáp nhanh, nhiều lượt |
| Thông tin gây lo lắng cho nhiều người (layoff, đổi cơ cấu) | Nói trước với nhóm bị ảnh hưởng, sau đó broadcast | Tin đi trước qua tin hành lang thì mất kiểm soát |

**Bước 4 — Mã hoá: cấu trúc trước, chi tiết sau.** Ba quy tắc có tác động lớn nhất:

- Đặt kết luận và điều người nghe cần làm ở câu đầu (chi tiết ở chủ đề 7 — BLUF).
- Định lượng thay cho tính từ: không "sắp xong", mà "còn 2 API và 1 vòng test, dự kiến 3 ngày".
- Nói cả mức bất định và điều kiện: "với điều kiện A, B; nếu A trượt thì hệ quả là C".

**Bước 5 — Kiểm tra đã nhận đúng (bước hay bị bỏ).** Không hỏi "em hiểu chưa" — câu này chỉ kiểm tra
được sự lịch sự. Ba kỹ thuật dùng được:

- **Read-back:** "Anh muốn chắc là anh nói rõ chưa. Em nói lại giúp anh phần em sẽ làm và phần em
  thấy chưa rõ." Đặt gánh nặng ở phía người nói, không ở phía người nghe.
- **Hỏi hệ quả:** "Nếu partner trả về status PENDING quá 24h thì theo cách này mình xử lý sao?" Câu
  trả lời sai chứng tỏ mô hình trong đầu người nghe khác mô hình của bạn.
- **Yêu cầu output nhỏ nhanh:** đề nghị viết lại 5 dòng trong ticket hoặc vẽ nhanh flow. Trạng thái
  hiểu chỉ kiểm tra được qua một artifact, không qua một cái gật đầu.

Tiêu chí biết là xong: có artifact ghi lại (ticket, doc, message tóm kết), người nhận đã phát lại
được nội dung bằng lời của họ, và có ngày cùng người chịu trách nhiệm.

### Trade-off

| Trục | Nghiêng về bên trái khi | Nghiêng về bên phải khi | Cái giá phải trả |
|---|---|---|---|
| **Đồng bộ (họp/gọi) vs Bất đồng bộ (văn bản)** | Vấn đề nhiều bất định, cần nhiều lượt hỏi–đáp; có cảm xúc; cần thu hẹp inferential distance lớn | Thông tin một chiều; nhiều người nhận; múi giờ lệch; cần lưu vết | Đồng bộ: đắt theo n người, không để lại dấu, người vắng mất tin. Bất đồng bộ: độ trễ cao, dễ bị hiểu lệch mà không ai phát hiện |
| **Nói ngắn vs nói đủ điều kiện** | Người nghe cache ấm, chỉ cần kết luận để hành động | Người nghe sẽ dùng thông tin để cam kết với bên thứ ba | Ngắn: mất điều kiện → người khác cam kết sai. Đủ: dài, người nghe lọc mất phần quan trọng |
| **Broadcast rộng vs Gửi đúng người** | Cần tính minh bạch, chống tin hành lang, nhiều người bị ảnh hưởng | Thông tin gây lo lắng, chưa chốt, hoặc chỉ 2 người cần hành động | Broadcast: tạo nhiễu, làm loãng tín hiệu, người ta học cách bỏ qua channel. Hẹp: người ngoài vòng biết muộn và coi đó là bị che |
| **Chuẩn hoá template vs Nói tự nhiên** | Tổ chức lớn, nhiều người mới, cần so sánh được giữa các nhóm | Team nhỏ, cache ấm, template thành nghi lễ | Chuẩn hoá quá mức: người ta điền cho đủ, chất lượng thông tin giảm còn thấp hơn nói tự nhiên |

Một trade-off riêng của bối cảnh Việt Nam: **nói thẳng vs giữ hoà khí trong nhóm**. Nói thẳng trước
mặt nhiều người làm tăng độ chính xác thông tin nhưng có thể làm người bị nói mất mặt, và cái giá là
lần sau họ không mang thông tin xấu ra nữa — nghĩa là bạn đổi một lần thông tin chính xác lấy một
kênh bị đóng lâu dài. Cách xử lý không phải là nói nhẹ đi, mà là **đổi kênh**: giữ nội dung, chuyển
sang 1-1.

### Real-world Scenarios

**Tình huống 1 — "Dạ vâng em hiểu rồi" (ODC, khách Nhật, 18 engineer).**

Tuấn (Tech Lead) giao cho Khoa (Junior, 8 tháng kinh nghiệm) làm module import file từ đối tác. Cuộc
hội thoại thực tế:

Phiên bản sai:

> **Tuấn:** Em làm import file như bên module cũ nhé, nhớ handle case file lỗi, với validate mã đơn
> theo quy tắc của khách. Cuối tuần xong được không?
> **Khoa:** Dạ vâng ạ, em hiểu rồi.
> **Tuấn:** Ừ, có gì không hiểu thì hỏi anh.

Hai lỗi cơ chế trong 30 giây. Thứ nhất, curse of knowledge: "như bên module cũ" trong đầu Tuấn là 6
quyết định (retry policy, nơi lưu file lỗi, cách log, cách báo BA, transaction boundary, giới hạn
dung lượng); trong đầu Khoa là "đọc file, parse, insert". Thứ hai, câu "có gì không hiểu thì hỏi anh"
đặt toàn bộ gánh nặng lên người có ít thông tin nhất và ít vị thế nhất — trong bối cảnh Việt Nam,
việc một junior nói "em không hiểu" với người trên mình còn mang thêm rủi ro bị đánh giá là kém, nên
xác suất họ hỏi thấp hơn xác suất họ tự đoán. "Dạ vâng" ở đây là tín hiệu về **quan hệ** (em tôn
trọng anh, em sẽ cố), không phải tín hiệu về **nội dung** (em đã giải mã đúng).

Phiên bản đúng:

> **Tuấn:** Anh giao em module import file từ partner. Anh nói trước phần bối cảnh vì nó ảnh hưởng
> cách làm: file này chạy vào 2h sáng, nếu lỗi mà mình không biết thì hôm sau khách thấy thiếu dữ
> liệu và đó là thứ họ đã complain một lần rồi.
> **Khoa:** Dạ.
> **Tuấn:** Anh không mong em nắm hết ngay. Anh làm thế này: anh nói phần anh biết, rồi em nói lại
> cho anh nghe em định làm gì — chỗ nào anh nói thiếu thì lúc đó anh mới biết mà bổ sung. Không phải
> kiểm tra em, là kiểm tra anh nói đủ chưa.
> **Khoa:** Dạ vâng.
> **Tuấn:** (nói 5 phút về 6 quyết định) Giờ em nói lại giúp anh: file lỗi thì đi đâu, ai được báo,
> và mình retry mấy lần?
> **Khoa:** Em... phần retry em chưa rõ ạ. File lỗi thì em nghĩ là log ra rồi bỏ qua dòng đó.
> **Tuấn:** Đó, chỗ đó anh nói chưa rõ. Bỏ qua dòng lỗi là sai với khách này vì họ đối chiếu tổng
> số. Em ghi vào ticket ba dòng: hành vi khi file lỗi, khi một dòng lỗi, và khi partner không trả
> file. Chiều nay anh xem, mất 5 phút. Xong bước đó mình mới nói tới deadline.

Điểm khác biệt về cơ chế: câu "là kiểm tra anh nói đủ chưa" chuyển việc thừa nhận không hiểu từ hành
vi rủi ro thành hành vi được yêu cầu. Đây là cách thực dụng nhất để vượt rào cản văn hoá mà không cần
"đổi văn hoá" — chỉ cần đổi ai chịu trách nhiệm cho việc thông điệp không tới.

**Tình huống 2 — Ba góc nhìn về cùng một buổi họp (product startup, 30 engineer, Series A).**

Bối cảnh: họp planning 90 phút. PO (đồng thời là founder) trình bày một tính năng mới cho campaign
tháng sau. Hà (Senior BE) nêu rằng thiết kế hiện tại sẽ làm query trên bảng orders 200 triệu dòng
không có index phù hợp, và đề xuất lùi một sprint để làm partition. PO trả lời "campaign đã book
media rồi, không lùi được". Cuộc họp kết thúc với kết luận "làm theo scope hiện tại, tối ưu sau". Hai
tuần sau, campaign chạy, p99 latency lên 8 giây trong 40 phút, checkout fail 6%.

*Nhìn từ IC (Hà):* "Tôi đã nói rồi, không ai nghe." Tín hiệu Hà nhận được là ý kiến kỹ thuật của mình
không có giá trị trong phòng đó. Hệ quả hành vi dễ đoán: lần sau Hà nói một lần rồi thôi, hoặc ghi
một dòng trong ticket để sau này chỉ vào. Đây là điểm bắt đầu của defensive documentation — viết để
tự bảo vệ thay vì để hệ thống ra quyết định tốt hơn. Nhưng đọc lại lỗi truyền tin: Hà đã gửi kết luận
("cần partition, cần một sprint") mà không gửi **chuỗi suy luận và hệ quả bằng đơn vị của PO**. PO
nhận được một yêu cầu lùi lịch, không nhận được một dự báo rủi ro doanh thu.

*Nhìn từ Tech Lead (Nam):* Nam ở trong phòng và im lặng vì thấy PO đã quyết. Đây là chỗ Nam bỏ mất
việc của mình. Việc của Tech Lead trong khoảnh khắc đó không phải là chọn bên, mà là **dịch**: chuyển
thông điệp của Hà sang trục giá trị của PO và biến một tranh luận thành một quyết định có điều kiện.
Câu Nam nên nói: "Em tóm lại rủi ro theo cách khác. Nếu giữ scope, xác suất checkout chậm trong giờ
cao điểm campaign là cao — theo số hiện tại thì bảng orders sẽ scan trên 200 triệu dòng. Nếu chuyện
đó xảy ra đúng ngày chạy media thì mình mất phần lớn traffic đã trả tiền. Có ba lựa chọn: lùi một
sprint, giữ lịch nhưng giới hạn tính năng cho 10% user, hoặc giữ nguyên và chuẩn bị kill-switch cùng
người trực. Anh chọn phương án nào thì em triển khai, nhưng em cần anh chọn có biết rủi ro."

*Nhìn từ Manager (Linh, EM):* Linh không có mặt trong họp, và biết chuyện sau incident. Với Linh, đây
không phải sự cố kỹ thuật mà là **sự cố hệ thống truyền tin có thể lặp lại**: một rủi ro có người
biết trước hai tuần đã không tới được nơi ra quyết định dưới dạng dùng được. Việc của Linh không phải
tìm ai sai. Ba can thiệp cấu trúc: (1) yêu cầu mọi quyết định "làm nhanh, tối ưu sau" phải được ghi
kèm hệ quả định lượng và một điều kiện kích hoạt kế hoạch B; (2) dạy team cách phát biểu rủi ro bằng
đơn vị business, coi đó là kỹ năng cần đánh giá trong Performance Review của Senior trở lên; (3) tạo
kênh có độ trễ thấp cho rủi ro loại này (một mục 5 phút cố định trong weekly với PO, hoặc quyền
escalate trực tiếp). Nếu Linh chỉ nói "lần sau các em nói mạnh hơn", cùng một incident sẽ quay lại
với một tên feature khác.

Ba góc nhìn cùng đọc một sự việc: IC thấy vấn đề về sự tôn trọng, Tech Lead thấy vấn đề về dịch ngôn
ngữ, Manager thấy vấn đề về thiết kế kênh. Cả ba đều đúng, nhưng chỉ hai góc sau tạo ra hành động
thay đổi được tần suất lặp lại.

**Tình huống 3 — Tiếng Anh không phải tiếng mẹ đẻ (ODC, khách Mỹ và khách Nhật).**

Cùng một team, hai khách, hai dạng thất bại ngược nhau.

Với khách Mỹ: engineer Việt viết email vòng vo, đặt kết luận ở cuối, dùng nhiều "maybe", "I think
we might", "if possible". Người Mỹ đọc và không thấy có yêu cầu nào, nên không hành động. Ba tuần
sau team Việt nói "bọn em đã báo rồi mà khách không phản hồi". Lỗi ở đây không phải trình độ tiếng
Anh mà là **mức trực tiếp của thông điệp**: trong bối cảnh đó, một câu điều kiện lịch sự bị giải mã
là tín hiệu yếu, gần với "không quan trọng".

Với khách Nhật: cùng một team viết một email rất thẳng theo lời khuyên trên, gửi trực tiếp cho
engineer phía khách mà bỏ qua PM của họ, và nhận về sự lạnh nhạt kéo dài. Ở đây thông điệp đúng
nhưng **đường đi sai** so với kỳ vọng về quy trình của phía đối tác.

Nguyên tắc rút ra không phải "người Mỹ thích thẳng, người Nhật thích vòng" — đó là stereotype dùng
không được. Nguyên tắc là: khi giao tiếp bằng ngôn ngữ thứ hai với đối tác có bộ mã khác, **giảm phụ
thuộc vào sắc thái và tăng phụ thuộc vào cấu trúc**. Cụ thể:

- Dùng câu ngắn, một ý một câu. Bỏ thành ngữ, bỏ mệnh đề lồng.
- Nêu tường minh mức độ quan trọng và loại thông điệp: "This is a blocker", "This is FYI, no action
  needed", "I need a decision by Sep 5".
- Số hoá mọi thứ: ngày, giờ có múi giờ, số lượng.
- Dùng bullet và tiêu đề thay vì đoạn văn. Cấu trúc chịu nhiễu tốt hơn văn xuôi.
- Xác nhận lại bằng văn bản sau mọi cuộc gọi. Trong cuộc gọi tiếng Anh, cả hai bên đều đánh giá quá
  cao mức mình hiểu nhau.

### Best Practices

- **Trước mỗi thông điệp quan trọng, viết ra một câu "ai làm gì trước ngày nào".** Lý do: mục tiêu
  hành vi buộc bạn chọn thông tin theo tiêu chí "cần cho quyết định" thay vì "tôi biết nên tôi nói".
- **Chuyển gánh nặng xác nhận sang người gửi.** Thay "em hiểu chưa" bằng "em nói lại giúp anh xem anh
  nói đủ chưa". Lý do: bất đối xứng vị thế làm người nhận không dám báo lỗi truyền tin; đổi cách hỏi
  là cách rẻ nhất để phá bất đối xứng đó.
- **Luôn kèm điều kiện khi báo tiến độ lên trên.** Lý do: điều kiện là phần bị nén mất đầu tiên qua
  các tầng; nếu bạn không nói nó thành một mệnh đề riêng biệt, nó sẽ không đi qua được tầng thứ hai.
- **Sau mọi cuộc họp có quyết định, gửi 5–10 dòng tóm kết trong 24 giờ.** Lý do: biến kênh không bền
  thành kênh bền, và tạo cơ hội cuối để người hiểu khác lên tiếng khi chi phí sửa còn thấp.
- **Với người có cache lạnh, trả chi phí warm-up trước, đừng tiết kiệm.** Lý do: 5 phút bối cảnh rẻ
  hơn 9 ngày rework; và cache một khi đã ấm thì dùng lại được nhiều lần.
- **Định lượng thay tính từ trong mọi phát biểu về trạng thái và rủi ro.** Lý do: tính từ ("gần
  xong", "khá rủi ro") không giải nén được thành hành động, và mỗi người giải nén thành một mức khác
  nhau.

### Anti-patterns

- **Giao tiếp bằng "chắc là hiểu rồi".** Cơ chế phá hệ thống: bỏ hẳn tầng error detection, nên lỗi
  chỉ lộ ra ở cuối chuỗi khi chi phí sửa đã nhân lên. Dấu hiệu sớm: trong họp không ai đặt câu hỏi
  nào; ticket không có tiêu chí chấp nhận; câu "tôi tưởng là" xuất hiện trong retro.
- **Nhồi mọi thứ vào kênh đồng bộ.** Cơ chế: chi phí O(n) người × thời lượng, đồng thời phá vỡ thời
  gian làm việc sâu của engineer, và không để lại vết nên người vắng mặt bị loại khỏi ngữ cảnh. Dấu
  hiệu sớm: calendar kín, nhưng khi hỏi "quyết định X ghi ở đâu" thì không ai chỉ được.
- **Ngược lại: nhồi mọi thứ vào kênh bất đồng bộ.** Cơ chế: một tranh luận có bất định cao và có cảm
  xúc chạy trên văn bản sẽ leo thang, vì thiếu tín hiệu phi ngôn từ và mỗi lượt cách nhau nhiều giờ.
  Dấu hiệu sớm: thread trên 30 message, xuất hiện câu "như em đã nói ở trên".
- **Broadcast để tránh trách nhiệm.** Gửi vào channel 50 người rồi coi như đã thông báo. Cơ chế:
  trách nhiệm loãng ra, không ai là người nhận cụ thể; đồng thời làm giảm tỉ lệ đọc của cả channel.
  Dấu hiệu sớm: nhiều message quan trọng không có ai reply.
- **Dùng độ dài làm bằng chứng của sự rõ ràng.** Doc 20 trang không có phần tóm tắt. Cơ chế: người
  quyết không đọc, nên quyết định vẫn được ra bằng thông tin hành lang. Dấu hiệu sớm: người viết nói
  "mọi thứ có trong doc rồi" khi bị hỏi lại.
- **Thay đổi requirement bằng lời trong daily.** Cơ chế: mất vết, nên khi tranh chấp không có nguồn
  đúng, và người vắng mặt làm theo bản cũ. Dấu hiệu sớm: hai engineer trong cùng team mô tả cùng một
  yêu cầu theo hai cách khác nhau.

### Khi nào KHÔNG nên áp dụng

Quy trình năm bước và các cơ chế xác nhận ở trên có chi phí thật, và có những vùng nó gây hại:

- **Khi thông điệp có chi phí mất mát thấp.** Áp read-back cho mọi việc sẽ biến bạn thành người khó
  làm việc cùng và làm loãng tín hiệu "cái này quan trọng". Nếu mọi thứ đều được xác nhận ba lần, sự
  xác nhận không còn mang thông tin. Giữ nghi thức đầy đủ cho nhóm thông điệp có hệ quả tài chính,
  compliance, hoặc ảnh hưởng nhiều người.
- **Trong incident đang chạy.** Lúc đó cần command-and-control: câu ngắn, một người điều phối, không
  tranh luận về từ ngữ. Việc dừng lại để "thu hẹp inferential distance" giữa lúc hệ thống down là
  sai; phần đó thuộc postmortem. Xem chi tiết ở [06-incident-va-metrics.md](/series/engineering-leedership/06-incident-va-metrics/).
- **Khi team đã có common ground rất ấm và tốc độ là ràng buộc chính.** Hai người pair với nhau hai
  năm, cùng một codebase, đang chạy hotfix: bắt họ viết doc xác nhận là thuế thuần. Chi phí giao tiếp
  nên tỉ lệ với độ lạnh của cache và chi phí sai.
- **Khi vấn đề thật là thiếu Psychological Safety, không phải thiếu kỹ thuật truyền tin.** Nếu một
  người im lặng vì lần trước nói thật thì bị nhắc trong Performance Review, thì mọi kỹ thuật hỏi mở
  đều vô dụng; câu trả lời sẽ vẫn là "dạ em ổn". Sửa nguyên nhân ở
  [03-team-leadership.md](/series/engineering-leedership/03-team-leadership/) trước, đừng sửa kỹ thuật hỏi.
- **Khi vấn đề thật là cấu trúc tổ chức.** Nếu thông tin phải đi qua 5 tầng và mỗi tầng có động cơ
  làm đẹp báo cáo, không có kỹ thuật giao tiếp cá nhân nào bù được mất mát cấu trúc đó. Đó là bài
  toán ở [09-to-chuc-va-scaling.md](/series/engineering-leedership/09-to-chuc-va-scaling/): giảm số hop, hoặc tạo kênh bỏ qua
  tầng (skip-level).

---

## 2. Active Listening

### Problem Statement

Một công ty logistics, 55 engineer. Trang (Senior Frontend, 4 năm ở công ty, người duy nhất hiểu hết
màn hình điều phối) nộp đơn xin nghỉ vào một ngày thứ Ba. Trong exit interview, Trang nói: "Em nói
chuyện này ba lần rồi." Manager mở lại notes: lần thứ nhất, sáu tháng trước, Trang nói "dạo này em
làm mấy cái màn hình config hơi nhiều" — manager trả lời "ừ giai đoạn này thế, sau campaign sẽ khác".
Lần thứ hai, bốn tháng trước, Trang hỏi "công ty mình có định làm design system không anh" — manager
trả lời "hay đấy, em viết proposal đi". Lần thứ ba, hai tháng trước, Trang nói "em thấy hơi khó thấy
mình đang tiến bộ" — manager trả lời bằng 10 phút kể về roadmap quý sau.

Ba lần đó là ba lần một tín hiệu về ý định nghỉ việc được phát ở dạng đã nói giảm, và ba lần người
nghe giải mã nó thành một câu phàn nàn nhỏ rồi trả lời bằng giải pháp. Không ai nói dối, và manager
này thậm chí ghi notes. Vấn đề là notes ghi lại **câu chữ** mà bỏ mất **thứ câu chữ đó đang che**.

Hiện tượng quan sát được của một tổ chức có năng lực nghe thấp:

- **Tin xấu đến muộn và đến từ nguồn ngoài.** Bạn biết một người không hài lòng từ đơn xin nghỉ, biết
  một dự án trễ từ khách hàng, biết hai người xung đột từ người thứ ba.
- **Tỉ lệ 1-1 mà người kia nói dưới 50% thời lượng.** Đo được: bật đồng hồ trong 5 buổi 1-1 liên tiếp
  và ước lượng. Nhiều lead thấy con số thật của mình là 20–30%.
- **Số lần lead đưa giải pháp trước khi người kia mô tả xong vấn đề.** Đếm trong một tuần. Con số
  thường gây bất ngờ cho chính người đếm.
- **Người trong team ngừng mang vấn đề đến.** Dấu hiệu định lượng: số message chủ động báo rủi ro
  giảm dần theo quý, trong khi số incident không giảm.
- **Trong retro, các vấn đề nêu ra đều là vấn đề an toàn** (CI chậm, thiếu môi trường test) và không
  bao giờ là vấn đề về người, quyết định hay ưu tiên.

Cái mất ở đây không phải "sự gắn kết". Cái mất là **năng lực phát hiện sớm của tổ chức**. Một lead
nghe kém vẫn có đủ dữ liệu từ dashboard, nhưng dashboard chỉ hiển thị thứ đã xảy ra. Tín hiệu về thứ
sắp xảy ra — người sắp nghỉ, kiến trúc sắp vỡ, khách sắp mất kiên nhẫn — hầu như chỉ tồn tại trong
đầu người và chỉ ra được qua kênh nói, ở dạng yếu, một lần, và thường vào lúc bạn đang bận.

### First Principles

**Cơ chế một: nghe là hoạt động tốn tài nguyên và nó cạnh tranh trực tiếp với việc soạn câu trả
lời.** Working memory con người có sức chứa nhỏ (dòng nghiên cứu công khai về giới hạn 4–7 chunk).
Nghe để hiểu đòi hỏi giữ trong bộ nhớ làm việc: nội dung vừa nói, mô hình về ngữ cảnh của người nói,
và các giả thuyết về điều họ chưa nói. Soạn câu trả lời cũng dùng chính nguồn tài nguyên đó. Vì vậy
**không thể vừa nghe kỹ vừa nghĩ câu trả lời** — đó không phải vấn đề ý chí, đó là ràng buộc phần
cứng. Người nghe giỏi không phải người tập trung hơn; họ là người **hoãn việc soạn câu trả lời**,
thường bằng cách viết ra để giải phóng bộ nhớ, hoặc bằng cách chấp nhận vài giây im lặng trước khi
đáp.

**Cơ chế hai: người nói nén thông tin theo mức rủi ro cảm nhận.** Khi chi phí của việc nói thẳng cao
(mất mặt, bị đánh giá, làm hỏng quan hệ), người ta không im lặng hoàn toàn — họ phát tín hiệu ở mức
công suất thấp, thăm dò. "Hơi nhiều", "cũng hơi khó", "em chỉ hỏi thôi ạ", "không có gì đâu anh, em
nói vui" đều là dạng tín hiệu đã giảm biên độ. Trong bối cảnh Việt Nam, nơi giữ mặt và tránh xung đột
trực diện là chuẩn mặc định, biên độ giảm còn mạnh hơn: một mức bất mãn 8/10 có thể được phát ra ở
mức 2/10. Người nghe áp thang tuyến tính vào tín hiệu đã bị nén sẽ luôn đánh giá thấp. Nguyên tắc
hiệu chỉnh: **với người ở vị thế thấp hơn hoặc trong văn hoá tránh xung đột, hãy nhân biên độ tín
hiệu lên vài lần trước khi kết luận về mức nghiêm trọng** — và đừng kết luận, hãy hỏi thêm.

**Cơ chế ba: vì sao lead nghe kém dần khi lên cao.** Bốn lực đồng thời, tất cả đều mang tính cấu
trúc, không phải tính cách:

1. *Đổi tỉ lệ ai nói với ai.* Càng lên cao, càng nhiều người có động cơ nói với bạn, và bạn càng có
   ít thời gian mỗi người. Mỗi cuộc trò chuyện bị nén xuống, và cách nén nhanh nhất là cắt phần mô tả
   vấn đề để nhảy tới giải pháp.
2. *Lọc thông tin xấu theo động cơ.* Người báo tin xấu chịu rủi ro; người báo tin tốt được thưởng.
   Không cần ai cố ý, sau vài quý dòng thông tin đến bạn tự động lệch về phía tích cực. Đây là bài
   toán principal–agent áp vào thông tin.
3. *Chi phí của việc nói ra tăng theo khoảng cách quyền lực.* Cùng một câu "em thấy quyết định này
   sai", nói với đồng nghiệp là rẻ, nói với người quyết lương của mình là đắt. Bạn càng lên cao,
   người khác càng phải trả giá cao hơn để cho bạn thông tin chính xác.
4. *Kỹ năng của bạn tăng khả năng đoán.* Vì bạn đã gặp 50 tình huống giống thế, sau 20 giây bạn đã có
   một giả thuyết. Giả thuyết đó đúng 70% và đó chính là cái bẫy: 30% còn lại là những tình huống bạn
   sẽ hiểu sai một cách tự tin, và chúng thường là những tình huống quan trọng nhất.

**Cơ chế bốn: nghe là hành vi tạo tín hiệu ngược.** Khi một người nói ra một điều khó và được nghe
đến hết, họ nhận được thông tin: "kênh này còn hoạt động". Khi bị cắt lời hoặc bị đáp bằng giải pháp
tức thì, họ nhận thông tin: "kênh này chỉ nhận báo cáo, không nhận vấn đề". Đây là feedback loop có
độ lợi rất cao: hai hoặc ba lần là đủ để một người ngừng dùng kênh, và việc ngừng đó không có tín
hiệu báo — bạn không bao giờ nhận được thông báo "tôi đã ngừng nói với bạn".

### Mental Model

**Model 1 — Bốn mức nghe.** Không phải thang đạo đức, mà là bốn chế độ có chi phí và ứng dụng khác
nhau:

| Mức | Chế độ | Bạn đang làm gì | Dùng khi |
|---|---|---|---|
| 1 | Nghe để trả lời | Chờ khe trống để nói ý mình | Tranh luận kỹ thuật đã rõ vấn đề, cần chốt nhanh |
| 2 | Nghe để lấy dữ liệu | Trích thông tin cần cho quyết định của bạn | Status update, thu thập fact cho incident |
| 3 | Nghe để hiểu mô hình của người kia | Dựng lại cách họ nhìn vấn đề, kể cả phần bạn cho là sai | 1-1, xung đột kiến trúc, feedback đi vào |
| 4 | Nghe để họ tự nghe được mình | Đặt câu hỏi giúp họ làm rõ điều họ chưa gọi tên | Coaching, quyết định nghề nghiệp, người đang bế tắc |

Sai lầm phổ biến của Tech Lead mới: dùng mức 2 cho tình huống cần mức 3. Bạn trích được fact và bỏ
mất lý do người ta mang fact đó tới.

**Model 2 — Iceberg: nội dung / cảm xúc / nhu cầu.** Mỗi phát biểu có ba tầng. "Sao mình lại phải làm
lại module này lần thứ ba?" — nội dung là một câu hỏi về quyết định, cảm xúc là sự bực và mệt, nhu cầu
có thể là "tôi cần biết công việc của tôi có giá trị lâu dài". Trả lời chỉ ở tầng nội dung ("vì
requirement đổi") là kỹ thuật đúng và giao tiếp thất bại, vì tầng tạo ra hành vi tiếp theo của người
đó là tầng ba.

**Model 3 — Nghe như giảm bất định (Bayesian).** Vào cuộc trò chuyện với một prior (giả thuyết) và
theo dõi xem prior của bạn có dịch chuyển không. Nếu sau 20 phút giả thuyết ban đầu của bạn không thay
đổi gì, khả năng cao bạn đã không nghe mà chỉ thu thập bằng chứng ủng hộ. Một câu hỏi tự kiểm tra sau
mỗi 1-1: **"tôi vừa biết thêm điều gì mà trước đó tôi không biết hoặc tưởng khác?"** Không trả lời
được là dấu hiệu buổi đó chạy ở mức 1 hoặc 2.

### Practical Framework

**Chuẩn bị (2 phút, trước cuộc trò chuyện):**

- Viết ra giả thuyết hiện tại của bạn về vấn đề. Viết ra để không phải giữ trong bộ nhớ làm việc, và
  để sau đó kiểm tra được nó có dịch chuyển hay không.
- Đặt mục tiêu số lượt: trong 30 phút, bạn nói không quá 30% thời lượng.
- Đóng laptop hoặc lật úp. Đây không phải phép lịch sự mà là quản lý tài nguyên attention của chính
  bạn.

**Vòng lặp bốn bước trong cuộc trò chuyện:**

1. **Hỏi mở, hỏi đúng chỗ.** Câu hỏi mở tốt có ba tính chất: không gợi ý câu trả lời, hỏi về cụ thể
   thay vì tổng quát, và hỏi về quá trình thay vì kết luận. So sánh: "Em có ổn không?" (đóng, gợi ý
   câu trả lời "ổn") so với "Tuần vừa rồi việc gì làm em mất nhiều thời gian nhất mà em thấy không
   đáng?" (mở, cụ thể, về quá trình).
2. **Phản chiếu (reflect) nội dung và cả cảm xúc.** "Nếu anh hiểu đúng thì: em làm ba màn hình config
   liên tiếp, em không thấy nó liên quan tới hướng em muốn đi, và em đã nói chuyện này trước đây mà
   thấy không có gì thay đổi. Phần cuối có đúng không?" Phản chiếu làm ba việc: kiểm tra giải mã, cho
   người nói cơ hội sửa, và phát tín hiệu rằng bạn đang ở mức 3.
3. **Đào sâu một tầng trước khi kết luận.** Ba câu hỏi có tỉ lệ thu hoạch cao nhất:
   - "Em kể cho anh một ví dụ cụ thể gần nhất."  (biến tính từ thành sự kiện)
   - "Nếu chuyện này tiếp tục ba tháng nữa thì sao?"  (hiệu chỉnh biên độ tín hiệu bị nén)
   - "Còn gì nữa không?" — hỏi hai lần. Thông tin quan trọng thường nằm ở lượt thứ hai hoặc thứ ba,
     sau khi người nói đã kiểm tra rằng nói ra là an toàn.
4. **Xác nhận lại và chốt ai làm gì.** Kết thúc bằng một câu tóm kết do bạn nói và một câu hỏi "anh
   ghi lại thế đúng chưa". Nếu có hành động, nói rõ ai làm và khi nào bạn quay lại chủ đề này.

**Kỹ thuật phát hiện thông tin đã bị nói giảm** — bốn dấu hiệu, mỗi dấu hiệu kèm một câu để mở:

| Dấu hiệu trong lời nói | Điều nó thường che | Câu nên dùng |
|---|---|---|
| Trạng từ giảm nhẹ: "hơi", "cũng", "một chút" | Mức thật cao hơn nhiều | "Nếu cho điểm từ 1 tới 10, chỗ này em ở mức nào?" |
| Chủ ngữ chung: "mọi người thấy", "team đang nói là" | Ý kiến của chính người nói, chưa dám nhận | "Còn riêng em thì em thấy thế nào?" |
| Rào trước: "em chỉ hỏi thôi ạ", "không có gì đâu" | Đây là chủ đề chính, không phải phụ | "Em cứ nói, chỗ này quan trọng. Anh nghe." |
| Hỏi về quy trình/chính sách thay vì về mình: "công ty mình có làm X không" | Một nhu cầu cá nhân đang được hỏi gián tiếp | "Anh trả lời câu đó, nhưng trước đó anh hỏi lại: nếu có X thì nó giúp gì cho em?" |
| Im lặng sau khi bạn nói xong | Không đồng ý nhưng không phản đối | Đợi 5 giây. Rồi: "Anh thấy em đang nghĩ gì đó. Em nói thật giúp anh." |

Tiêu chí biết là xong: bạn nói được một điều mới học được từ buổi đó; người kia đã sửa lại ít nhất
một lần cách bạn tóm tắt; và có một artifact ghi lại (notes riêng của bạn) kèm ngày quay lại.

### Trade-off

| Trục | Nghiêng về nghe nhiều hơn khi | Nghiêng về chốt nhanh hơn khi | Cái giá |
|---|---|---|---|
| **Nghe đủ vs Tốc độ quyết định** | Vấn đề có yếu tố người, bất định cao, hệ quả dài hạn | Đang trong incident; deadline giờ tính bằng giờ; vấn đề đã rõ và chỉ cần chọn | Nghe quá lâu: quyết định trễ, người nói cảm thấy "nói mãi không thấy gì thay đổi". Chốt quá nhanh: quyết bằng thông tin sai |
| **Empathy (đứng cùng phía) vs Objectivity (giữ khoảng cách)** | Người đang trong khủng hoảng; cần khôi phục kênh trước khi lấy dữ liệu | Đang điều tra một tranh chấp giữa hai người; đánh giá hiệu suất | Empathy quá mức: bạn nhận hết lời giải thích và mất khả năng đánh giá. Khách quan quá mức: người kia thấy mình bị điều tra và đóng kênh |
| **Nghe cá nhân vs Nghe theo hệ thống** | Cần tín hiệu sớm, cần niềm tin | Vấn đề lặp lại ở nhiều người → cần đo và sửa cấu trúc, không sửa từng ca | Chỉ nghe cá nhân: bạn thành nơi hấp thụ mọi bất mãn và không có thay đổi cấu trúc nào; số người tới nói tăng, số vấn đề được sửa không tăng |

Một trade-off dễ bị bỏ qua: **nghe nhiều làm tăng kỳ vọng**. Khi bạn nghe rất kỹ mà không có hành
động nào theo sau, bạn tạo ra một dạng nợ. Người ta chấp nhận "anh nghe rồi và anh không làm được vì
lý do X" tốt hơn nhiều so với "anh nghe rồi" rồi im. Vì vậy kỹ năng nghe phải đi kèm kỷ luật đóng
vòng: mỗi chủ đề nghe được, hoặc có hành động, hoặc có một câu trả lời tường minh rằng sẽ không có
hành động và vì sao.

### Real-world Scenarios

**Tình huống 1 — Nhảy vào giải pháp (product startup, 25 engineer).**

Nam (Senior BE, 6 năm kinh nghiệm) vào 1-1 với Hà (Tech Lead).

Phiên bản sai:

> **Nam:** Em thấy cái service notification đang khó bảo trì quá. Mỗi lần thêm loại thông báo mới là
> phải sửa bốn chỗ, mà—
> **Hà:** Ừ anh biết. Cái đó là do hồi đầu làm nhanh. Em cứ tách cái template engine ra, làm một
> interface chung, xong define config bằng YAML là gọn. Anh nghĩ mất tầm một sprint. Em viết cái RFC
> ngắn rồi anh review nhé.
> **Nam:** Dạ... vâng.
> **Hà:** Còn gì nữa không?
> **Nam:** Dạ hết rồi ạ.

Hà vừa giải quyết đúng một vấn đề kỹ thuật và bỏ mất thông tin thật. Điều Nam định nói tiếp là: bốn
chỗ phải sửa nằm ở bốn service do bốn người khác nhau own, và hai người trong đó không phản hồi PR
của Nam trong hai tuần. Vấn đề của Nam không phải thiết kế, mà là **quyền và sự phối hợp**. Sau buổi
này Nam nhận thêm một việc refactor mà không giải quyết được gốc, và học được rằng mang vấn đề đến Hà
thì sẽ ra thêm việc.

Phiên bản đúng:

> **Nam:** Em thấy cái service notification đang khó bảo trì quá. Mỗi lần thêm loại thông báo mới là
> phải sửa bốn chỗ, mà—
> **Hà:** Em nói tiếp đi, anh đang nghe. (im lặng)
> **Nam:** ...mà bốn chỗ đó thuộc bốn service khác team. Lần vừa rồi em mở PR sang service của team
> Payment, treo mười ngày. Em ping hai lần rồi thôi vì thấy hơi khó xử.
> **Hà:** Anh tóm lại xem đúng chưa: vấn đề kỹ thuật là coupling bốn chỗ, nhưng thứ đang chặn em thực
> ra là PR chéo team không ai review, và em không thoải mái khi phải đi nhắc người ở team khác. Phần
> sau đúng không?
> **Nam:** Đúng ạ. Em không biết mình có quyền đòi hỏi cái đó không.
> **Hà:** Còn gì nữa không?
> **Nam:** Còn... thật ra tháng trước em có nói với anh Khoa về việc này, nhưng em thấy mình cứ nói
> mà không có gì đổi thì cũng ngại.
> **Hà:** Cảm ơn em nói câu đó, nó quan trọng hơn phần kỹ thuật. Anh làm hai việc: một, anh nói với
> Tech Lead bên Payment về SLA review PR chéo team, trong tuần này, và anh sẽ nói lại kết quả cho em
> dù kết quả là không. Hai, việc đi nhắc PR chéo team là việc của vai Senior, em có quyền làm và nếu
> có ai phản ứng thì đó là việc của anh. Còn thiết kế lại notification thì mình bàn sau, nó là hệ quả
> chứ không phải nguyên nhân.

Khác biệt: hai giây im lặng ở lượt đầu đã mua được thông tin quan trọng nhất. Câu "còn gì nữa không"
hỏi lần thứ hai mua được thông tin đắt hơn nữa — rằng Nam đã bắt đầu ngừng dùng kênh.

**Tình huống 2 — Nghe khi bạn biết người kia sai (ODC, khách EU).**

Minh (Senior BE) khăng khăng rằng nguyên nhân chậm là do database, trong khi Tuấn (Tech Lead) đã xem
trace và biết rõ nguyên nhân là N+1 query trong tầng application của chính Minh. Bản năng: đưa bằng
chứng ra ngay, chốt trong 2 phút. Cái mất khi làm thế: Minh giữ mô hình sai trong đầu (chỉ ngừng nói
ra chứ không đổi), và lần sau khi Minh có một chẩn đoán đúng nhưng ngược ý lead, Minh sẽ không nói.

Cách chi phí thấp hơn về dài hạn: "Anh nghe cách em đọc vấn đề đã. Em dựa vào tín hiệu nào để kết
luận là database?" Nghe hết, rồi: "Anh có một dữ liệu khác, mình xem cùng." Ở mức 3, mục tiêu không
phải thắng chẩn đoán mà là **sửa mô hình chẩn đoán của người kia**, vì mô hình đó sẽ được dùng lại
trong 50 lần tiếp theo mà bạn không có mặt. Ranh giới: nếu hệ thống đang down, bỏ hết phần trên, chốt
ngay bằng dữ liệu, và để phần sửa mô hình cho postmortem.

**Tình huống 3 — Nghe trong họp có 9 người.** Nghe cá nhân không mở rộng được sang họp nhóm: người ở
vị thế thấp sẽ không nói khi có 8 người nghe. Ba can thiệp cụ thể: (1) hỏi vòng tròn có tên — "Trang,
em làm phần này lâu nhất, em thấy chỗ nào rủi ro?" (chỉ định giúp người ngại không phải tự giành
lượt); (2) yêu cầu viết trước — mỗi người viết 3 dòng ý kiến vào doc trước họp, giảm hiệu ứng người
nói đầu neo cả phòng; (3) lead nói sau cùng — nếu lead nêu quan điểm trước, phần lớn ý kiến sau đó sẽ
hội tụ về quan điểm đó và bạn mất dữ liệu thật.

### Best Practices

- **Viết giả thuyết của bạn ra giấy trước cuộc trò chuyện quan trọng.** Lý do: giải phóng working
  memory và biến việc "nghe kỹ" thành việc kiểm chứng có tiêu chí, thay vì một ý định tốt.
- **Hỏi "còn gì nữa không" hai lần.** Lý do: người nói thường thăm dò bằng vấn đề an toàn trước; vấn
  đề thật xuất hiện sau khi họ xác nhận rằng nói ra không bị trừng phạt.
- **Phản chiếu trước khi phản biện, và phản chiếu cả phần bạn không đồng ý.** Lý do: người ta chỉ
  nghe được phản biện sau khi tin rằng mình đã được hiểu; phản chiếu là chi phí thấp nhất để đạt điều
  đó.
- **Hiệu chỉnh biên độ theo khoảng cách vị thế.** Lý do: cùng một câu, người junior nói "hơi mệt" và
  người ngang cấp nói "hơi mệt" mang hai mức thông tin khác nhau; áp cùng thang đo là sai hệ thống.
- **Đóng vòng công khai mọi thứ bạn nghe được, kể cả khi câu trả lời là không.** Lý do: kênh chỉ tồn
  tại nếu người dùng nó thấy nó có tác dụng; một lần "nghe rồi im" đắt hơn một lần từ chối rõ ràng.
- **Trong họp, phát biểu sau.** Lý do: anchoring của người có quyền cao nhất làm sai lệch toàn bộ dữ
  liệu thu được sau đó.

### Anti-patterns

- **Nghe để phản biện.** Bạn nghe với mục tiêu tìm điểm sai để bác. Cơ chế phá hệ thống: người nói
  nhận ra sau vài lần và bắt đầu chỉ nói những gì đã chuẩn bị chống bác, nên bạn mất luôn phần thông
  tin thô, phần có giá trị nhất. Dấu hiệu sớm: bạn thường bắt đầu câu trả lời bằng "nhưng"; bạn biết
  mình sẽ nói gì từ giây thứ 15.
- **Nhảy vào giải pháp trước khi người kia nói xong vấn đề.** Cơ chế: bạn giải bài toán được phát
  biểu, mà phát biểu đầu tiên hầu như không bao giờ là bài toán thật; đồng thời bạn lấy mất quyền sở
  hữu vấn đề của người kia. Dấu hiệu sớm: người ta ra khỏi 1-1 với thêm việc chứ không với sự rõ ràng
  hơn; câu "dạ vâng" xuất hiện ngay sau giải pháp của bạn.
- **Nghe kiểu trị liệu, không đóng vòng.** Nghe rất tốt, đồng cảm rất nhiều, không làm gì. Cơ chế:
  tạo kỳ vọng rồi phá kỳ vọng, tệ hơn không nghe. Dấu hiệu sớm: cùng một chủ đề quay lại ở buổi 1-1
  thứ ba với cùng nội dung.
- **Ghi notes nguyên văn câu chữ mà bỏ ngữ cảnh.** Cơ chế: bạn lưu vỏ và mất tầng nhu cầu, nên khi
  đọc lại ba tháng sau không tái tạo được tín hiệu. Dấu hiệu sớm: notes của bạn toàn danh từ và không
  có dòng nào dạng "giả thuyết của tôi về điều đang thật sự xảy ra".
- **Dùng 1-1 để nghe nhưng đồng thời trả lời Slack.** Cơ chế: chia attention làm giảm chất lượng giải
  mã, và quan trọng hơn, nó phát tín hiệu về mức ưu tiên rõ hơn mọi lời nói. Dấu hiệu sớm: bạn không
  nhớ nổi ba điều người kia nói buổi trước.
- **Săn tín hiệu quá mức, đọc ý mọi câu nói.** Cơ chế ngược: khi mọi phát biểu đều bị coi là ẩn ý,
  người ta phải rào trước nhiều hơn và giao tiếp trở nên đắt hơn. Dấu hiệu sớm: nhân viên bắt đầu nói
  "không, ý em đúng là như em nói thôi ạ".

### Khi nào KHÔNG nên áp dụng

- **Trong incident đang chạy.** Mức nghe cần thiết là mức 2: lấy fact, ngắn, nhanh. Phản chiếu cảm
  xúc lúc hệ thống đang down làm chậm quá trình khôi phục và gây khó hiểu cho người đang xử lý.
- **Khi vấn đề đã được nghe ba lần và cần một quyết định.** Đến lượt thứ tư, việc tiếp tục nghe trở
  thành cách trì hoãn. Người kia không cần được hiểu thêm, họ cần một câu trả lời — kể cả câu trả lời
  là "không, và đây là lý do".
- **Khi đang xử lý một vi phạm rõ ràng về quy tắc** (rò rỉ dữ liệu khách, gian lận, hành vi gây mất
  an toàn cho người khác). Ở đây cần nghe để lấy fact và cần hành động; kỹ thuật nghe kiểu coaching
  gửi tín hiệu sai rằng ranh giới còn thương lượng được.
- **Khi người nói đang dùng kênh nghe của bạn để tránh việc.** Có dạng nhân viên mang vấn đề đến liên
  tục và mỗi lần được nghe rất kỹ, không lần nào tự xử lý. Ở đây can thiệp đúng là chuyển sang mức
  coaching có ràng buộc: "em mang cho anh hai lựa chọn và khuyến nghị của em, mình bàn 10 phút" (xem
  [08-hiring-va-phat-trien.md](/series/engineering-leedership/08-hiring-va-phat-trien/)).
- **Khi bạn không có năng lượng để nghe thật.** Nghe nửa vời tệ hơn hoãn: nó tiêu một lần vốn tin cậy
  mà không mua được thông tin. Hoãn 24 giờ và nói rõ lý do là lựa chọn tốt hơn.

---

## 3. Feedback

### Problem Statement

Một fintech ở TP.HCM, 70 engineer, chu kỳ Performance Review nửa năm. Tháng 6, Khoa (Middle BE, 3
năm) nhận rating "cần cải thiện" với nhận xét: giao tiếp kém, không chủ động, chất lượng code không
ổn định. Khoa sốc thật, không diễn: trong sáu tháng trước đó Khoa chưa từng nghe điều gì tương tự.
Manager của Khoa thì tin rằng mình đã nói nhiều lần — bằng những câu như "chỗ này em xem lại nhé", "cố
lên", "cái PR vừa rồi hơi lâu đấy". Illusion of transparency ở dạng thuần khiết nhất: người gửi tin
tưởng rằng mức độ nghiêm trọng đã được truyền, người nhận ghi nhận một nhận xét thoáng qua.

Hai tuần sau, Khoa nghỉ. Trong exit interview: "Nếu biết sớm em đã sửa."

Hiện tượng quan sát được của một tổ chức có hệ feedback hỏng:

- **Performance Review có nội dung mới.** Đây là chỉ số quan trọng nhất. Nếu trong buổi review có bất
  kỳ điều gì người nhận chưa từng nghe, hệ feedback liên tục đã thất bại và buổi review đang đóng vai
  kênh truyền tin chính — với độ trễ sáu tháng.
- **Khoảng cách giữa lần hành vi xảy ra và lần nó được nói ra.** Đo bằng ngày. Trên 2 tuần thì ngữ
  cảnh đã mất, và cuộc trò chuyện chuyển từ "sự việc" sang "tính cách".
- **Hành vi sai lặp lại trong Code Review.** Cùng một loại comment (thiếu test, tên biến, không handle
  error) xuất hiện trên PR của cùng một người qua 10 PR liên tiếp. Điều này nghĩa là feedback đang
  được phát ở tầng cú pháp, không tới được tầng nguyên tắc.
- **Junior sợ mở PR.** Đo bằng kích thước PR trung bình (tăng dần vì người ta gom lại để tránh nhiều
  vòng review) và thời gian từ commit đầu tới lúc mở PR.
- **Feedback chỉ đi một chiều.** Nếu lead chưa từng nhận được một feedback cụ thể nào từ team trong
  hai quý, không có nghĩa lead không có vấn đề — nghĩa là kênh ngược không tồn tại.
- **Tỉ lệ người rời đi nói "em không được nói trước".** Con số này đo trực tiếp độ trễ của hệ feedback.

### First Principles

**Cơ chế một: không có feedback thì không có learning loop, và không có learning loop thì hành vi
hiện tại được coi là chuẩn.** Đây là điểm bị hiểu nhầm nhiều nhất. Khi một hành vi xảy ra và không có
tín hiệu điều chỉnh nào, hệ thống không ở trạng thái trung tính — nó **phát ra tín hiệu chấp thuận**.
Im lặng không phải là không nói gì; im lặng là nói "cái này ổn". Sau ba tháng, hành vi đó đã được
củng cố và thành một phần bản dạng nghề nghiệp của người đó ("tôi là kiểu engineer làm nhanh, không
viết test"). Chi phí sửa một hành vi tăng theo thời gian nó được ngầm chấp thuận, vì bạn không còn
sửa một hành vi mà đang thách thức một bản dạng.

**Cơ chế hai: feedback thất bại vì nó bị giải mã như một đe doạ bản dạng.** Con người xử lý tín hiệu
về vị thế xã hội và về giá trị bản thân bằng cơ chế phản ứng nhanh, có trước phần suy luận. Khi một
phát biểu bị giải mã là "tôi là người không đủ tốt", năng lực nghe và xử lý logic giảm mạnh — người
nhận chuyển sang chế độ phòng vệ: giải thích, phản bác, hoặc đóng lại. Điều này giải thích tại sao
feedback đúng về nội dung vẫn thất bại: bạn nói về một hành vi, người kia nghe về con người họ. Hệ
quả thực dụng: **mọi kỹ thuật feedback tốt đều là kỹ thuật tách hành vi khỏi bản dạng**. "PR này thiếu
test cho nhánh lỗi" và "em không cẩn thận" mô tả cùng một sự việc, nhưng cái thứ hai không sửa được
vì nó không nêu hành động nào, và nó tấn công vào phần người ta phải bảo vệ.

**Cơ chế ba: độ trễ phá giá trị của tín hiệu.** Trong bất kỳ control system nào, feedback càng trễ so
với hành động thì càng khó gán nguyên nhân và càng dễ gây dao động. Với con người, độ trễ hai tuần đã
làm mất ngữ cảnh (không ai nhớ chính xác mình đã nghĩ gì lúc đó), và độ trễ sáu tháng biến feedback
thành một phán quyết về con người thay vì một chỉnh sửa về hành vi. Ngoài ra độ trễ tạo hiệu ứng cộng
dồn: người đưa feedback tích lũy nhiều sự việc rồi nói một lần, nên cường độ đột ngột cao và người
nhận nghe như một cuộc tấn công.

**Cơ chế bốn: bất đối xứng chi phí trong bối cảnh Việt Nam.** Chi phí ngắn hạn của việc nói thẳng là
hiện hữu và cá nhân (không khí căng, người kia mất mặt, quan hệ khó xử, có thể bị coi là "gay gắt");
lợi ích thì trễ, phân tán và không quy được cho ai. Với một lead trẻ nói với một người lớn tuổi hơn
hoặc nhiều thâm niên hơn, chi phí đó còn cao hơn. Bất đối xứng này làm cân bằng tự nhiên của tổ chức
nghiêng về **không nói** — và điều đó không phải lỗi tính cách của ai, mà là kết quả tất yếu của cấu
trúc chi phí. Muốn đổi kết quả phải đổi chi phí: làm cho việc đưa feedback trở thành thứ được kỳ vọng
và có nghi thức (một mục cố định trong 1-1, một vòng feedback định kỳ trong team), để nói không còn là
hành vi đặc biệt cần dũng cảm.

**Cơ chế năm: feedback là hàng hoá có giá trị nếu người nhận tin vào ý định của người gửi.** Cùng một
câu, đến từ người đã chứng minh muốn bạn tốt lên thì được xử lý như dữ liệu; đến từ người chưa có nền
tin cậy thì được xử lý như một cuộc tấn công cần phòng thủ. Vì vậy nền tin cậy là **điều kiện tiên
quyết**, không phải phần trang trí. Đây là lý do cơ học của việc "care personally" đứng trước
"challenge directly".

### Mental Model

**Model 1 — Radical Candor (Kim Scott, mô hình công khai): hai trục độc lập.** Trục dọc: mức bạn quan
tâm tới con người đó một cách cá nhân (care personally). Trục ngang: mức bạn dám nói thẳng vấn đề
(challenge directly). Bốn ô:

```
                 challenge directly cao
                          │
  Obnoxious Aggression    │   Radical Candor
  (thẳng mà không quan    │   (thẳng và có quan tâm)
   tâm — "nói cho biết")  │
 ─────────────────────────┼─────────────────────────  care personally cao
  Manipulative Insincerity│   Ruinous Empathy
  (không thẳng, không     │   (quan tâm mà không dám
   quan tâm — nói sau lưng)│   nói — dạng phổ biến nhất
                          │    ở Việt Nam)
```

Điểm quan trọng khi áp vào bối cảnh Việt Nam: ô nguy hiểm nhất không phải Obnoxious Aggression mà là
**Ruinous Empathy**. Nó dễ chịu cho cả hai bên trong ngắn hạn, được xã hội khen là "tinh tế", và nó
là cơ chế trực tiếp tạo ra tình huống của Khoa ở trên. Cái giá của Ruinous Empathy do người nhận trả,
trả trễ, và trả một lần rất lớn.

**Model 2 — Feedback như control loop có gain và delay.** Ba tham số: độ trễ (delay), độ lợi (gain,
tức cường độ), và tần suất (sampling rate). Hệ hoạt động tốt khi độ trễ nhỏ, tần suất cao, độ lợi
vừa. Hầu hết tổ chức chạy sai cả ba: độ trễ 6 tháng, tần suất 2 lần/năm, độ lợi cực cao trong buổi
review. Đó là công thức chuẩn để tạo dao động (người nhận sốc, phản ứng quá mức, hoặc rời đi).

**Model 3 — Feedback là dữ liệu, không phải phán quyết.** Với người nhận: mọi feedback đều là một
điểm dữ liệu về **cách bạn được cảm nhận bởi một người trong một ngữ cảnh**, không phải sự thật về
bạn. Điều này không làm nó bớt giá trị — nếu ba người độc lập nói cùng một điều thì đó là tín hiệu
mạnh về hệ quả hành vi của bạn, bất kể ý định của bạn là gì. Khoảng cách giữa ý định và hệ quả chính
là nội dung mà feedback mang lại, và nó là thông tin bạn không thể tự tạo ra.

### Practical Framework

**Framework 1 — SBI (Situation – Behavior – Impact).** Ba thành phần, thiếu thành phần nào thì hỏng
theo cách riêng:

| Thành phần | Nội dung | Nếu thiếu |
|---|---|---|
| **Situation** | Thời điểm, bối cảnh cụ thể: "trong buổi review PR #412 chiều thứ Ba" | Trở thành nhận xét về tính cách; người nhận không kiểm chứng được nên phòng vệ |
| **Behavior** | Điều quan sát được, không suy diễn: "em viết 'đoạn này viết sai hoàn toàn, đọc lại docs đi' trong ba comment" | Thành cảm nhận không có căn cứ; tranh luận chuyển sang "anh nghĩ em thế à" |
| **Impact** | Hệ quả cụ thể lên người khác, hệ thống, hoặc kết quả: "Linh sau đó gom PR to hơn để ít bị review, PR gần nhất 900 dòng và mất 4 ngày để merge" | Thành sở thích cá nhân của người nói; người nhận có thể hợp lý từ chối |

Ba câu kiểm tra trước khi nói: (1) Tôi có mô tả được hành vi bằng thứ một camera ghi lại được không?
(2) Tôi có nêu được hệ quả bằng thứ đếm được hoặc quan sát được không? (3) Tôi có đang nói về hành vi
hay đang nói về con người?

**Framework 2 — Feedforward: hướng về lần sau, không về lần trước.** Sau khi nêu SBI, chuyển sang một
yêu cầu cụ thể cho hành vi tương lai. Lý do cơ chế: quá khứ không sửa được nên tranh luận về nó là
tranh luận zero-sum về việc ai đúng; tương lai thì thương lượng được. So sánh: "lần trước em không
nên nói thế" (mời gọi tranh luận) so với "lần sau, khi thấy thiết kế sai hướng, em nói ở dạng câu hỏi
và đề nghị gọi 15 phút thay vì viết ba comment" (một hành vi có thể thực hiện).

**Framework 3 — Quy trình 6 bước cho một buổi feedback có chuẩn bị:**

1. **Kiểm tra động cơ của chính mình.** Câu hỏi: tôi nói cái này để người kia tốt hơn, hay để tôi bớt
   bực? Nếu là cái thứ hai, hoãn 24 giờ. Feedback phát ra từ sự bực sẽ mang cường độ sai và người
   nhận đọc được điều đó.
2. **Thu thập đủ 2–3 sự việc cụ thể, có ngày.** Một sự việc thì dễ bị coi là ngoại lệ; năm sự việc
   thì thành cáo trạng. Hai đến ba là vùng tối ưu.
3. **Chọn kênh và thời điểm.** 1-1, riêng, không ngay trước một buổi họp quan trọng của người đó,
   không vào chiều thứ Sáu (họ phải mang nó qua cuối tuần mà không nói được với ai).
4. **Nói: bối cảnh → SBI → im lặng.** Im lặng sau phần Impact là phần khó nhất và quan trọng nhất.
   Đây là chỗ bạn nhường lượt và biết được người kia đã giải mã ra gì.
5. **Nghe phản hồi ở mức 3, kể cả khi bạn không đồng ý.** Có thể bạn thiếu dữ liệu. Nếu người kia đưa
   ngữ cảnh mới, ghi nhận thật, không "vâng nhưng dù sao thì".
6. **Chốt một hành vi tương lai, một mốc thời gian, một cách kiểm tra — và ghi lại.** Không ghi lại
   thì ba tháng sau bạn không có cơ sở nói rằng đã có tiến bộ hay không, và người kia cũng không có.

**Framework 4 — Nhận feedback (phần thường bị bỏ, nhưng quyết định việc bạn có nhận được feedback lần
thứ hai hay không).** Bốn bước:

- **Nghe hết, không giải thích ngay.** Phản xạ giải thích ngữ cảnh ("tại vì hôm đó...") được người
  gửi giải mã là phòng vệ, và họ sẽ nhẹ đi hoặc dừng lại.
- **Hỏi thêm một ví dụ.** "Anh cho em thêm một ví dụ nữa được không, em muốn nhận ra pattern." Câu
  này làm hai việc: bạn có dữ liệu dùng được, và bạn phát tín hiệu rằng kênh này an toàn.
- **Tách ý định khỏi hệ quả.** Bạn có quyền giữ rằng ý định của mình tốt, và đồng thời chấp nhận rằng
  hệ quả đã khác ý định. Đây là chỗ duy nhất để học.
- **Đóng vòng sau 2–4 tuần.** "Tháng trước anh nói với em về cách em comment trong review. Em đã đổi
  cách X. Anh thấy có khác không?" Người từng cho bạn feedback mà thấy nó dẫn tới thay đổi sẽ cho
  tiếp; người không thấy gì sẽ ngừng.

**Template ghi lại (dùng cho cả hai phía, lưu trong notes 1-1):**

```
[FEEDBACK LOG]
Ngày:              2026-03-12
Người nhận:        Khoa (Middle BE)
Người đưa:         Tuấn (Tech Lead)
Kênh:              1-1, 20 phút

Situation:  PR #412 (2026-03-09) và PR #418 (2026-03-11), review cho Linh (Junior).
Behavior:   3 comment dạng "viết sai hoàn toàn", "đọc lại docs", không nêu lý do,
            không đề xuất cách sửa. Không có comment nào ghi nhận phần làm đúng.
Impact:     Linh nhắn riêng hỏi "em có nên nghỉ việc không". Kích thước PR của Linh
            tăng từ ~150 lên ~900 dòng trong 3 tuần (gom lại để ít bị review).
            Lead time merge của Linh tăng từ 1.2 ngày lên 4 ngày.

Phản hồi của người nhận:  "Em không nghĩ nó tới mức đó. Em chỉ muốn nhanh."
                          Bối cảnh mới: Khoa đang review 9 PR/tuần, quá tải.

Hành vi thống nhất cho lần sau:
  1. Mỗi comment chặn (blocking) phải có: lý do + link chuẩn/ví dụ + một cách sửa gợi ý.
  2. Vấn đề trên 3 comment cùng chủ đề → chuyển sang gọi 15 phút, không viết tiếp.
  3. Khoa được giảm còn 5 PR/tuần; Tuấn nhận phần còn lại trong 3 tuần.

Cách kiểm tra:   Xem lại comment của Khoa trên 5 PR kế tiếp; hỏi Linh trong 1-1 ngày 2026-04-09.
Mốc quay lại:    2026-04-09
Người chịu trách nhiệm theo dõi:  Tuấn
```

### Trade-off

| Trục | Nghiêng về nói thẳng khi | Nghiêng về nói nhẹ / giữ mặt khi | Cái giá của mỗi bên |
|---|---|---|---|
| **Trực tiếp vs Giữ mặt** | Hành vi đang gây hại cho người khác hoặc cho hệ thống; đã nói nhẹ mà không đổi; người nhận là Senior trở lên và ngữ cảnh nghề nghiệp đòi hỏi | Nền tin cậy chưa có; người nhận đang trong giai đoạn khủng hoảng cá nhân; vấn đề nhỏ và không lặp lại; có mặt người thứ ba | Thẳng: người nhận có thể mất mặt, quan hệ căng vài tuần, rủi ro bị coi là gay gắt trong văn hoá tránh xung đột. Nhẹ: thông điệp không tới, hành vi tiếp tục, người nhận bị đánh giá thấp sau 6 tháng mà không hiểu vì sao — cái giá này do họ trả và nó lớn hơn nhiều |
| **Feedback tức thì vs Chờ thời điểm phù hợp** | Sự việc còn nóng, ngữ cảnh còn tươi, hành vi có thể tái diễn trong ngày | Người kia đang trong cảm xúc mạnh; có khán giả; bạn đang bực | Tức thì: cường độ có thể sai, dễ nói trước mặt người khác. Chờ: mất ngữ cảnh, dễ trở thành "để dồn nói một lần" |
| **Riêng tư vs Công khai** | Hầu hết mọi feedback điều chỉnh (correcting) | Feedback ghi nhận (reinforcing); hoặc khi cần xác lập chuẩn cho cả team (nêu hành vi, không nêu tên) | Công khai điều chỉnh: người nhận mất mặt và mọi người khác học được rằng nêu vấn đề ra là rủi ro. Riêng tư ghi nhận: mất cơ hội truyền chuẩn |
| **Đưa feedback vs Đổi hệ thống** | Vấn đề nằm ở lựa chọn hành vi của một người | Cùng một hành vi xuất hiện ở 4/8 người trong team | Feedback từng người khi nguyên nhân là hệ thống: bạn đang phạt người cho một lỗi thiết kế, và bạn phải làm lại việc đó với từng người mới |

Về trục "giữ mặt", cần nói thẳng phần thường bị né: **quá nhẹ nhàng không phải là tử tế, nó là chuyển
chi phí sang người khác và sang tương lai.** Người lead chọn nói nhẹ đang mua sự dễ chịu cho chính
mình trong 20 phút, và bán đi cơ hội sửa của người kia trong sáu tháng. Trong đa số trường hợp ở Việt
Nam mà tôi quan sát được, người bị nói nhẹ không cảm ơn khi biết sự thật muộn — họ hỏi "sao anh không
nói sớm". Cách xử lý ranh giới văn hoá không phải giảm nội dung, mà là **giữ nguyên nội dung và đầu tư
vào ba thứ**: kênh riêng tư, nền tin cậy trước đó, và cách mở đầu không đe doạ bản dạng.

### Real-world Scenarios

**Tình huống 1 (bắt buộc theo yêu cầu) — Feedback cho Senior có code chất lượng cao nhưng review làm
junior sợ.**

Bối cảnh: product startup, 28 engineer. Khoa (Senior BE, 7 năm, chất lượng kỹ thuật cao nhất team, là
người phát hiện ba bug nghiêm trọng trước production trong quý), review rất khắt khe. Hai junior mới
vào ba tháng: một người đã hỏi HR về việc chuyển team, một người gom PR to dần. Tuấn (Tech Lead) phải
nói chuyện này. Ràng buộc thật: Khoa đang đúng về mặt kỹ thuật, và nếu Tuấn xử lý sai thì mất người
giỏi nhất về chất lượng.

Phiên bản sai số 1 — vòng vo (Ruinous Empathy):

> **Tuấn:** Ê Khoa, dạo này ổn không? À mà... anh thấy mấy bạn junior hơi ngợp. Em review thoáng
> thoáng thôi cho các bạn đỡ sợ nhé, các bạn còn mới mà.
> **Khoa:** Ơ, nhưng code sai thì em phải nói. Anh muốn em cho pass code sai à?
> **Tuấn:** Không không, ý anh là... em nói nhẹ hơn ấy.
> **Khoa:** Dạ. (không đổi gì, vì không biết đổi cái gì)

Hỏng ở ba chỗ: yêu cầu là một tính từ ("thoáng", "nhẹ hơn") nên không thực hiện được; nó nghe như đề
nghị hạ chuẩn kỹ thuật, tức đe doạ đúng phần bản dạng mà Khoa tự hào nhất; và không có dữ liệu nào nên
Khoa hợp lý cho rằng Tuấn đang nghe lời phàn nàn của người yếu hơn.

Phiên bản sai số 2 — nói trước mặt cả team trong retro:

> **Tuấn:** Về việc review, anh muốn cả team lưu ý là mình nên nói năng nhẹ nhàng hơn với các bạn mới.
> Anh thấy mấy comment gần đây hơi nặng.

Hỏng: tất cả mọi người biết đang nói về Khoa. Khoa mất mặt trước team, và bài học mà cả team học được
không phải "review có tính xây dựng" mà là "nêu vấn đề chất lượng có thể bị nhắc trước mặt mọi người".
Số comment review của toàn team giảm 40% trong ba tuần sau đó — chất lượng tín hiệu code review sụp,
đúng thứ Tuấn không muốn.

Phiên bản đúng (1-1, 25 phút, có chuẩn bị dữ liệu):

> **Tuấn:** Anh muốn nói với em một việc về review, và anh nói trước hai điều cho rõ ràng. Một: chuẩn
> kỹ thuật của em là thứ anh không muốn hạ, một chút cũng không. Quý này em chặn ba bug nghiêm trọng
> trước production, anh biết và anh cần em tiếp tục. Hai: anh muốn nói về **cách** comment, không về
> **chuẩn**. Hai thứ đó tách nhau được.
> **Khoa:** Dạ, anh nói.
> **Tuấn:** PR #412 của Linh, ngày 9. Có ba comment: "viết sai hoàn toàn", "đọc lại docs đi", và "cái
> này ai dạy em thế". Anh đọc lại thì cả ba đều đúng về kỹ thuật — chỗ đó lock sai thật.
> **Khoa:** Vâng, em có kiểm tra rồi mới nói.
> **Tuấn:** Anh tin. Giờ phần hệ quả. Sau PR đó Linh nhắn riêng cho anh hỏi bạn ấy có nên nghỉ không.
> Và có số này: PR của Linh ba tuần trước trung bình 150 dòng, ba tuần gần đây 900 dòng, thời gian
> merge từ 1.2 ngày lên 4 ngày. Bạn ấy đang gom việc lại để ít phải qua review. Nghĩa là chuẩn của em
> đang tạo ra tác dụng ngược với chính mục tiêu của em: PR to hơn thì chất lượng review thấp hơn.
> **Khoa:** (im lặng) Em không nghĩ tới chỗ đó. Nhưng anh nói thật là em cũng đang review 9 PR một
> tuần, em không có thời gian viết dài dòng.
> **Tuấn:** Đó là dữ liệu mới với anh và nó là vấn đề của anh, không phải của em. Anh sẽ nói phần đó ở
> dưới. Trước hết mình chốt phần cách viết. Anh đề nghị ba thứ cụ thể, em xem có làm được không: một,
> comment nào chặn merge thì phải có lý do và một hướng sửa — không cần dài, một dòng đủ: "lock ở đây
> sai vì hai request song song sẽ cùng đọc, dùng SELECT FOR UPDATE hoặc đổi sang optimistic lock".
> Hai, nếu cùng một chủ đề mà em phải viết quá ba comment thì dừng viết, gọi 15 phút. Ba, mỗi PR ghi
> một dòng về phần làm đúng — không phải để khen, mà để người ta biết cái gì nên giữ.
> **Khoa:** Cái thứ ba em thấy hơi hình thức.
> **Tuấn:** Anh nghe. Đổi lại thế này: không bắt buộc ghi phần làm đúng, nhưng nếu em chặn trên năm
> chỗ trong một PR thì em phải nói được một câu về hướng tổng thể — kiểu "hướng đi đúng, vấn đề tập
> trung ở phần transaction". Vì năm comment chặn liên tiếp mà không có câu định hướng thì người nhận
> không đọc ra là "code này sai vài chỗ", họ đọc ra là "mình không làm được việc này".
> **Khoa:** Cái đó em làm được.
> **Tuấn:** Còn phần 9 PR một tuần: từ tuần sau em còn 5, anh và Nam chia phần còn lại. Anh cũng sẽ
> nói với Linh rằng anh đã nói chuyện với em về cách review, và rằng chuẩn kỹ thuật thì không đổi —
> anh không để em thành người xấu trong chuyện này. Bốn tuần nữa mình xem lại: anh đọc comment trên
> 5 PR kế tiếp, và anh hỏi Linh.
> **Khoa:** Được ạ. Anh nói thẳng thế này em dễ làm hơn.

Vì sao phiên bản này hoạt động: nội dung không hề nhẹ hơn (Tuấn nói thẳng "tác dụng ngược với mục tiêu
của em" và đưa số), nhưng bản dạng của Khoa được bảo vệ tường minh ngay từ đầu; yêu cầu là hành vi cụ
thể chứ không phải tính từ; phản hồi của Khoa được nghe thật và làm thay đổi một phần thoả thuận (mục
ba), nên Khoa có sở hữu trong kết quả; và Tuấn nhận phần lỗi hệ thống thuộc về mình (phân bổ review).

**Tình huống 2 — Feedback lên trên.** Hà (Senior BE) thấy Tech Lead của mình thường chốt quyết định
kiến trúc trong DM riêng rồi thông báo trong daily, làm cả team mất ngữ cảnh. Nói thế nào?

Phiên bản sai: "Anh quyết một mình không hỏi ai." (nói về con người, không kiểm chứng được, mời gọi
phòng vệ).

Phiên bản đúng: "Anh cho em nói một việc về cách ra quyết định, em thấy nó ảnh hưởng tới việc của em.
Hai lần vừa rồi — chuyển sang Kafka và đổi cách partition — em biết kết quả trong daily. Hệ quả với
em là em không hiểu vì sao loại các lựa chọn khác, nên khi Junior hỏi em, em không giải thích được, và
em cũng không phát hiện sớm được chỗ nó đụng vào module của em. Em đề nghị: quyết định ảnh hưởng trên
một service thì viết 10 dòng vào channel trước 24 giờ, ai không phản đối thì chốt. Em không cần họp,
em cần thấy các lựa chọn đã bị loại."

Cấu trúc giống hệt SBI, cộng thêm một đề xuất chi phí thấp cho người nhận. Feedback lên trên có tỉ lệ
thành công cao hơn nhiều khi kèm một giải pháp không tốn thời gian của người nhận.

**Tình huống 3 — Khi feedback nhận được là sai.** Trang (Tech Lead) bị PO nói trong họp: "team em
chậm". Phản xạ sai là phòng vệ tại chỗ hoặc im lặng rồi bực. Cách xử lý: "Anh cho em một ví dụ cụ thể
gần nhất mà anh thấy chậm hơn kỳ vọng." Sau đó chuyển về dữ liệu: lead time, số lần đổi scope giữa
sprint, thời gian chờ môi trường của khách. Có thể kết luận cuối cùng là team không chậm mà là hàng
đợi phía ngoài dài — nhưng bạn chỉ tới được đó nếu bước đầu là hỏi ví dụ chứ không phải phản bác. Và
lưu ý cơ chế: nếu PO cảm nhận là chậm thì cảm nhận đó là một sự thật về **kênh báo tiến độ**, kể cả
khi nó sai về tốc độ thật.

### Best Practices

- **Không để Performance Review chứa nội dung mới.** Lý do: review là kênh có độ trễ cao và cường độ
  cao; dùng nó làm kênh chính là thiết kế sai control loop. Nguyên tắc kiểm tra: trước buổi review,
  liệt kê mọi điểm sẽ nói và đánh dấu điểm nào chưa từng nói — nếu có, đó là lỗi của bạn trong sáu
  tháng qua.
- **Nói trong vòng 72 giờ kể từ khi sự việc xảy ra.** Lý do: đủ nguội để không nói bằng cảm xúc, đủ
  tươi để cả hai còn ngữ cảnh và còn kiểm chứng được.
- **Mô tả hành vi bằng thứ camera ghi được.** Lý do: cắt đường phòng vệ về suy diễn động cơ; tranh
  luận chuyển từ "anh nghĩ em thế à" sang "sự việc đó có xảy ra không".
- **Bảo vệ bản dạng tường minh khi nội dung nặng.** Lý do: người nhận cần biết cái gì đang bị thách
  thức và cái gì không; nói rõ "anh không nói về chuẩn kỹ thuật của em" giải phóng năng lực nghe cho
  phần còn lại.
- **Kết thúc bằng một hành vi cụ thể, một mốc, một cách kiểm tra.** Lý do: không có cách kiểm tra thì
  ba tháng sau hai bên có hai phiên bản khác nhau về việc đã có tiến bộ hay chưa.
- **Xin feedback về mình bằng câu hỏi hẹp.** Lý do: "em thấy anh làm gì chưa tốt?" quá rủi ro để trả
  lời thật; "trong buổi họp hôm qua, có chỗ nào anh nói làm em thấy khó phản biện không?" thì hẹp, cụ
  thể, và trả lời được mà không phải phán xét con người bạn.
- **Đưa feedback ghi nhận (reinforcing) một cách cụ thể, không chung chung.** Lý do: "em làm tốt lắm"
  không nói cho ai biết cần giữ hành vi nào; "cái cách em viết lại phần mô tả bug trong ticket #530
  làm QA không phải hỏi lại lần nào — giữ cách đó" mới truyền được chuẩn.

### Anti-patterns

- **Feedback sandwich (khen – phê – khen).** Cơ chế phá hệ thống: người nhận học rất nhanh rằng lời
  khen là bao bì, nên từ đó mọi lời khen của bạn mất giá trị tín hiệu, và họ chờ phần giữa. Ngoài ra
  nó làm loãng mức nghiêm trọng nên thông điệp chính không tới. Dấu hiệu sớm: người nhận căng lên
  ngay khi bạn bắt đầu khen.
- **Dồn nhiều tháng rồi nói một lần.** Cơ chế: cường độ đột ngột cao, mất ngữ cảnh từng sự việc, và
  người nhận phải phòng vệ trên nhiều mặt trận cùng lúc nên không sửa được gì. Dấu hiệu sớm: bạn có
  một danh sách trong đầu về một người và bạn chưa nói bất kỳ mục nào.
- **Feedback qua kênh sai.** Nói việc hiệu suất trong channel công khai, hoặc trong comment PR bằng
  câu ám chỉ. Cơ chế: thêm yếu tố khán giả nên người nhận bắt buộc phải bảo vệ vị thế thay vì xử lý
  nội dung. Dấu hiệu sớm: bạn thấy mình viết comment kỹ thuật mà thật ra đang nói về thái độ.
- **Feedback nêu suy diễn động cơ.** "Em không quan tâm tới chất lượng", "em không muốn học". Cơ chế:
  bạn khẳng định một điều không thể quan sát được, nên người nhận buộc phải phủ định con người mình
  thay vì bàn về hành vi. Dấu hiệu sớm: câu feedback của bạn có động từ chỉ nội tâm (muốn, nghĩ, quan
  tâm, cố ý).
- **Feedback không có Impact.** "Em nên viết test nhiều hơn." Cơ chế: thành sở thích cá nhân của lead,
  nên trong xung đột ưu tiên nó luôn thua deadline. Dấu hiệu sớm: người nhận trả lời "vâng, khi nào
  rảnh em làm".
- **Chỉ có feedback đi xuống.** Cơ chế: lead trở thành điểm mù duy nhất trong hệ thống learning của
  team, và điểm mù đó ở vị trí có tác động lớn nhất. Dấu hiệu sớm: hai quý không ai nói cho bạn một
  điều cụ thể nào cần đổi.
- **Đưa feedback rồi không bao giờ quay lại.** Cơ chế: người nhận đổi hành vi mà không được ghi nhận
  sẽ kết luận rằng chuyện đó không thật quan trọng, và quay về hành vi cũ. Dấu hiệu sớm: cùng một
  feedback được đưa lần thứ hai sau bốn tháng.

### Khi nào KHÔNG nên áp dụng

- **Khi nguyên nhân là hệ thống, không phải cá nhân.** Nếu 4/8 người trong team đều không viết test,
  đưa feedback từng người là sai địa chỉ và tạo cảm giác bất công. Nguyên nhân có thể là CI chậm 40
  phút, thiếu test data, hoặc chính bạn đã chấp nhận nhiều lần "lần này gấp, bỏ test". Sửa hệ thống
  trước, xem [05-technical-leadership.md](/series/engineering-leedership/05-technical-leadership/).
- **Khi nền tin cậy chưa tồn tại.** Feedback điều chỉnh vào tuần đầu bạn làm lead với một người bạn
  chưa từng làm việc cùng sẽ được giải mã là "người mới đến muốn dằn mặt". Ngoại lệ: hành vi gây hại
  ngay (xúc phạm người khác, rủi ro bảo mật) — cái đó nói ngay, và nói như một ranh giới, không như
  một feedback phát triển.
- **Khi người nhận đang trong khủng hoảng cá nhân.** Feedback cần năng lực xử lý mà lúc đó họ không
  có. Hoãn, và nói rõ là sẽ quay lại. Nhưng đừng hoãn vô hạn — "đang có việc gia đình" kéo dài chín
  tháng thì đó là một cuộc trò chuyện khác (chủ đề 4).
- **Khi bạn không có dữ liệu, chỉ có cảm nhận qua trung gian.** Feedback dựa trên "anh nghe mấy bạn
  nói" mất hết sức mạnh và tạo ra một vấn đề mới: người nhận sẽ đi tìm xem ai nói. Nếu chỉ có thông
  tin gián tiếp, việc của bạn là quan sát trực tiếp trước, hoặc tạo điều kiện để người có thông tin
  nói trực tiếp.
- **Khi vấn đề thật là quyết định về vai trò, không phải điều chỉnh hành vi.** Nếu bạn đã kết luận
  rằng người này không phù hợp với vai trò, tiếp tục đưa feedback phát triển là không trung thực và
  làm mất thời gian của cả hai. Đó là chủ đề 4.
- **Khi tần suất đã quá cao.** Một người nhận feedback về năm thứ khác nhau trong hai tuần sẽ không
  sửa được thứ nào và sẽ kết luận rằng mình đang bị nhắm vào. Chọn một hoặc hai thứ quan trọng nhất,
  bỏ phần còn lại — đây là quyết định về độ lợi của control loop.

---

## 4. Difficult Conversations

### Problem Statement

Ba tình huống có cùng một cấu trúc, xảy ra trong cùng một quý ở một công ty e-commerce 45 engineer:

*Một.* Nam (Middle BE, 4 năm) đã được nói từ đầu năm rằng "quý này em có cơ hội lên Senior". Đến kỳ
review, hội đồng không thông qua. Manager của Nam lùi buổi nói chuyện ba lần, rồi thông báo qua Slack
lúc 18h thứ Sáu: "Kết quả promotion quý này chưa được em nhé, quý sau mình cố gắng." Thứ Hai Nam vẫn
đi làm, nhưng trong sáu tuần sau đó số PR của Nam giảm một nửa và Nam từ chối nhận việc dẫn dắt phần
tích hợp đối tác. Ba tháng sau Nam nghỉ.

*Hai.* Hà đang làm Tech Lead một team 6 người và không phù hợp với vai trò đó: ba sprint liên tiếp
trượt commitment, hai người trong team nói riêng rằng không rõ ưu tiên, và Hà tự nhận đang kiệt sức
nhưng không muốn thừa nhận. EM cần đưa Hà về vai trò IC. Không có ai trong công ty từng làm việc này
trước đó, và mọi người đều mặc định rằng "xuống làm IC" nghĩa là bị giáng chức.

*Ba.* Dự án ODC cho khách Nhật, mốc release 30/9 đã cam kết trong hợp đồng phase. Ngày 5/9, Tech Lead
biết chắc rằng scope hiện tại không kịp — thiếu khoảng 3 tuần công. Nếu nói với khách ngay, công ty
chịu một cuộc họp rất khó và có thể ảnh hưởng phase sau. Nếu không nói, khả năng cao đến 25/9 sẽ phải
nói, lúc đó khách không còn lựa chọn nào ngoài chấp nhận trễ.

Trong cả ba trường hợp, cái làm hỏng kết quả không phải nội dung — nội dung là cố định và khó. Cái làm
hỏng là **việc trì hoãn và cách mở đầu**. Hiện tượng quan sát được của một tổ chức không có năng lực
xử lý cuộc trò chuyện khó:

- **Độ trễ giữa lúc lead biết và lúc người liên quan biết.** Với promotion: nhiều nơi là 2–4 tuần.
  Với tin trễ dự án: thường tới khi không còn phương án nào.
- **Tin xấu được truyền qua kênh bậc thấp** (Slack, email một chiều, người thứ ba) thay vì kênh băng
  thông cao. Đây là dấu hiệu người gửi đang tối ưu sự dễ chịu của chính mình.
- **Số người rời đi trong 90 ngày sau một tin xấu được truyền tệ.** Đây là chi phí đo được.
- **Không có văn bản follow-up**, nên hai bên có hai phiên bản về những gì đã được thống nhất, và ba
  tháng sau xuất hiện tranh chấp "anh có nói là..."
- **Vấn đề hiệu suất kéo dài trên 6 tháng mà chưa từng có một cuộc trò chuyện tường minh.** Chi phí do
  cả team trả, và team biết rất rõ ai đang không làm được việc.

### First Principles

**Cơ chế một: mỗi cuộc trò chuyện khó thực chất là ba cuộc trò chuyện chồng lên nhau** (khung phân
tích từ *Difficult Conversations*, Stone – Patton – Heen, Harvard Negotiation Project, tài liệu công
khai):

1. **The What Happened conversation** — tranh chấp về sự thật, về ai đúng, ai gây ra. Đặc điểm: hai
   bên đều tin mình có sự thật, và cả hai đều thiếu dữ liệu của bên kia.
2. **The Feelings conversation** — phần cảm xúc không được nói ra nhưng chi phối toàn bộ. Nếu không
   được thừa nhận, nó sẽ rò rỉ ra qua giọng, qua sự im lặng, qua các câu đá sang chuyện khác.
3. **The Identity conversation** — cuộc trò chuyện diễn ra trong đầu mỗi người: "điều này nói gì về
   việc tôi là ai?" Với engineer, ba bản dạng hay bị chạm nhất là: tôi là người có năng lực, tôi là
   người tử tế/hợp tác, và tôi là người xứng đáng được tôn trọng.

Tầng ba là tầng quyết định phản ứng, và nó là tầng không ai nói ra. Đây là lý do đưa thêm dữ liệu
không giải quyết được cuộc trò chuyện khó: người ta không phản ứng với dữ liệu, họ phản ứng với **hàm
ý của dữ liệu về giá trị bản thân**. Nam ở tình huống một không sốc vì thiếu một mức lương; Nam sốc vì
thông điệp "tôi không đủ tốt" đến bất ngờ và đến qua một kênh cho thấy người gửi không muốn đối diện.

**Cơ chế hai: identity threat làm giảm năng lực xử lý thông tin.** Khi bản dạng bị đe doạ, phản ứng
sinh lý (nhịp tim, adrenaline) đi trước phần suy luận. Trong 30–60 giây đầu sau một thông tin đe doạ,
người nhận gần như không xử lý được nội dung chi tiết. Hệ quả thực dụng cực kỳ quan trọng: **mọi thông
tin bạn nói trong một phút đầu sau tin xấu sẽ bị mất**. Vì vậy đừng nhồi lý do, kế hoạch phát triển,
và lời an ủi vào phút đó. Nói tin, dừng lại, để họ xử lý, rồi mới nói phần còn lại — hoặc hẹn một buổi
thứ hai cho phần kế hoạch.

**Cơ chế ba: trì hoãn có lợi tức âm.** Chi phí của cuộc trò chuyện khó không giữ nguyên theo thời
gian, nó tăng theo ba hướng: (a) người kia tiếp tục hành động dựa trên giả định sai (Nam từ chối cơ
hội khác vì tin mình sắp lên Senior); (b) số người biết trước người liên quan tăng lên, nên khi họ
biết, họ biết thêm rằng mình là người biết cuối cùng — đây là phần gây tổn hại nhiều hơn cả nội dung;
(c) không gian lựa chọn thu hẹp (khách Nhật ngày 5/9 còn có thể cắt scope, ngày 25/9 thì chỉ còn chấp
nhận trễ). Trong ba tình huống trên, phần lớn thiệt hại đến từ độ trễ, không từ nội dung.

**Cơ chế bốn: quan sát và suy diễn bị trộn trong ngôn ngữ tự nhiên.** "Em không chủ động" là một suy
diễn được phát biểu như một quan sát. Người nghe không thể phản biện một suy diễn bằng dữ liệu, nên
họ chỉ còn cách phủ nhận, và cuộc trò chuyện chuyển thành tranh chấp về con người. Tách hai lớp này ra
là kỹ thuật đơn giản nhất và có tác động lớn nhất: "trong sáu tháng qua, có bốn lần một vấn đề trong
module của em được phát hiện bởi người khác, và không có lần nào em nêu trước" (quan sát) — "anh đọc
điều đó là em đang giới hạn phạm vi của mình ở task được gán; anh có thể đọc sai, em nói anh nghe"
(suy diễn, được nêu tường minh là suy diễn và mở cho phản biện).

**Cơ chế năm: trong bối cảnh Việt Nam, cuộc trò chuyện khó có thêm một biến số — khán giả vô hình.**
Người nhận không chỉ nghĩ về nội dung mà còn nghĩ: ai biết chuyện này, mình sẽ giải thích với team thế
nào, bố mẹ đã kể với họ hàng rằng con sắp lên quản lý. Đặc biệt với tình huống rút khỏi vai trò lead,
phần "mất mặt" có chi phí thật và người lead phải xử lý nó như một phần của bài toán, không bỏ qua như
chuyện phụ. Cách xử lý cụ thể: thống nhất trước với người đó về **câu chuyện công khai** (public
narrative) — nói gì với team, ai nói, nói khi nào.

### Mental Model

**Model 1 — BATNA và kết quả tối thiểu chấp nhận được.** Vay từ lý thuyết thương lượng: trước khi vào
cuộc, xác định (a) kết quả mong muốn, (b) kết quả tối thiểu bạn chấp nhận rời khỏi phòng với nó, (c)
điều gì xảy ra nếu không đạt thoả thuận nào. Với cuộc trò chuyện về hiệu suất, kết quả tối thiểu có
thể chỉ là: "người đó rời khỏi phòng với hiểu biết chính xác về vị trí hiện tại của họ và một mốc thời
gian". Không phải "người đó đồng ý và cảm thấy tốt". Xác định trước điều này ngăn bạn nhượng bộ nội
dung để đổi lấy không khí dễ chịu.

**Model 2 — Tách quan sát / suy diễn / phán xét thành ba cột.** Trước cuộc trò chuyện, kẻ ba cột và
điền. Chỉ cột một được nói ở dạng khẳng định. Cột hai được nói ở dạng "anh đang đọc thế này, có thể
sai". Cột ba không nói.

| Quan sát (camera ghi được) | Suy diễn của tôi | Phán xét (không nói) |
|---|---|---|
| 3 sprint liên tiếp trượt commitment; 2/6 người nói không rõ ưu tiên | Hà đang giữ quá nhiều việc kỹ thuật và không có thời gian cho phần điều phối | "Hà không có tố chất làm lead" |
| Hà commit code 6 buổi tối/tuần trong 3 tuần gần nhất | Hà đang bù đắp bằng cách tự làm | "Hà không biết delegate" |

**Model 3 — Vòng lặp mở đầu trung tính.** Ba câu đầu quyết định phần còn lại. Một mở đầu trung tính có
ba tính chất: nêu rõ chủ đề (không để người kia phải đoán), nêu rõ đây là cuộc trò chuyện loại gì
(thông báo một quyết định đã chốt, hay thảo luận để cùng quyết), và không mang phán xét. Sai lầm phổ
biến nhất: mở đầu bằng small talk vui vẻ rồi rẽ ngoặt — điều này dạy người kia rằng những lần small
talk sau đều là dấu hiệu của tin xấu, và bạn phá một kênh dùng lâu dài.

### Practical Framework

**Giai đoạn 1 — Chuẩn bị (30–60 phút, viết ra, không làm trong đầu):**

1. **Mục tiêu:** một câu về kết quả bạn muốn có sau 48 giờ, mô tả bằng hành vi.
2. **Loại cuộc trò chuyện:** (a) thông báo quyết định đã chốt, (b) cùng ra quyết định, (c) nêu vấn đề
   để người kia tự đề xuất. Nói rõ loại nào ở câu mở đầu. Trộn ba loại này là nguồn gây tổn hại lớn
   nhất: nếu quyết định đã chốt mà bạn mở đầu như đang thảo luận, người kia sẽ tranh luận, rồi nhận ra
   mình bị lừa vào một vở kịch.
3. **Dữ liệu:** 2–4 quan sát cụ thể, có ngày, kiểm chứng được.
4. **Kết quả tối thiểu chấp nhận được** và **điều bạn sẽ không nhượng bộ**.
5. **Dự đoán ba phản ứng có thể** (phủ nhận, tấn công ngược, sụp xuống im lặng) và câu bạn sẽ dùng cho
   mỗi phản ứng.
6. **Câu chuyện công khai:** sau buổi này, team sẽ được nói gì, bởi ai, khi nào.
7. **Logistics:** phòng riêng, đủ thời gian (đừng để 20 phút rồi có họp tiếp), không phải chiều thứ
   Sáu, không phải ngay trước một demo quan trọng của người đó.

**Giai đoạn 2 — Trong cuộc trò chuyện (cấu trúc 6 bước):**

1. **Mở đầu trung tính, nêu chủ đề và loại, trong 2 câu.** "Anh gọi em vào để nói về kết quả promotion
   quý này. Đây là quyết định đã có kết quả, anh nói cho em biết và nói rõ lý do; sau đó mình bàn phần
   tiếp theo."
2. **Nói tin chính trong câu thứ ba, không vòng.** Chôn tin xấu ở giữa một đoạn dài là cách chắc chắn
   để nó bị hiểu sai, và người kia sẽ hiểu ra ở giữa câu, lúc đó họ không nghe được phần còn lại nữa.
3. **Im lặng 5–15 giây.** Đây là bước bị bỏ nhiều nhất. Bạn đang cho phần sinh lý của họ thời gian đi
   qua đỉnh. Đừng lấp bằng lời.
4. **Nêu quan sát, tách khỏi suy diễn.** Cột một ở dạng khẳng định, cột hai ở dạng "anh đọc thế này,
   có thể sai".
5. **Nghe. Không phòng vệ, không sửa lại phán quyết vì áp lực cảm xúc.** Nếu có dữ liệu mới thật, ghi
   nhận và nói rõ nó có đổi được quyết định hay không. Nếu không đổi được thì nói thẳng: "dữ liệu đó
   anh chưa biết, nhưng nó không đổi kết quả quý này; nó đổi cách mình làm quý sau."
6. **Chốt hành động, mốc thời gian, và câu chuyện công khai.** Kết bằng: "Anh gửi em bản ghi lại trong
   hôm nay để mình cùng một phiên bản."

**Giai đoạn 3 — Follow-up bằng văn bản trong 24 giờ.** Bắt buộc, vì trong trạng thái cảm xúc cao,
người nhận giữ lại rất ít nội dung. Văn bản không phải để tự bảo vệ pháp lý mà để cả hai có cùng một
phiên bản sự thật.

```
[Chủ đề] Tóm kết buổi trao đổi ngày 2026-07-14 — Nam & Tuấn
Người viết: Tuấn (Tech Lead) | Gửi: Nam | CC: Linh (EM)
Ngày: 2026-07-14

1. Nội dung đã trao đổi
   - Kết quả promotion lên Senior kỳ này: không thông qua. Quyết định của hội đồng
     ngày 2026-07-10, không thay đổi trong kỳ này.
   - Lý do, theo đúng thứ tự trọng số hội đồng đã dùng:
     a) Phạm vi ảnh hưởng: các đóng góp trong 2 quý tập trung trong module Order,
        chưa có hạng mục nào ảnh hưởng ngoài team (tiêu chí Senior mục 3.2).
     b) Dẫn dắt kỹ thuật: chưa có RFC/ADR nào do Nam chủ trì; 0/4 quyết định
        kiến trúc quý II do Nam đề xuất.
     c) Chất lượng và tốc độ thực thi: đạt (không phải điểm trừ).

2. Điều tôi (Tuấn) làm chưa đúng
   - Tháng 1 tôi nói "quý này em có cơ hội" mà không nêu tiêu chí cụ thể và không
     kiểm tra tiến độ giữa kỳ. Đó là lỗi của tôi, không phải của Nam.

3. Thống nhất cho kỳ tới (mục tiêu: hội đồng tháng 1/2027)
   - Nam chủ trì RFC cho phần tích hợp đối tác giao vận (bắt đầu: tuần 30/7).
   - Nam là người đại diện team trong nhóm làm việc chung với team Payment.
   - Review giữa kỳ: 2026-10-15, đối chiếu trực tiếp với 4 tiêu chí Senior.

4. Về việc nói với team
   - Không thông báo kết quả promotion cho team. Nếu có ai hỏi, Nam trả lời theo
     cách Nam muốn; tôi sẽ không đề cập.
   - Việc Nam chủ trì RFC sẽ được thông báo trong planning ngày 30/7 như một
     phân công bình thường.

5. Nam bổ sung / phản đối điểm nào thì reply vào email này trước 18/7.
   Nếu Nam thấy cách xử lý kỳ này không công bằng, Nam có thể trao đổi trực tiếp
   với Linh (EM) mà không cần qua tôi.
```

### Trade-off

| Trục | Nghiêng bên trái khi | Nghiêng bên phải khi | Cái giá |
|---|---|---|---|
| **Nói ngay vs Chuẩn bị kỹ** | Không gian lựa chọn đang thu hẹp theo ngày (tin trễ dự án); người kia đang ra quyết định dựa trên giả định sai | Nội dung phức tạp, có yếu tố lương/vai trò/pháp lý; bạn đang bực | Nói ngay chưa chuẩn bị: nói sai lý do, phải sửa lại lần sau và mất uy tín. Chuẩn bị quá lâu: chi phí trì hoãn (mục First Principles, cơ chế ba) |
| **Nói hết mọi lý do vs Nói 2–3 lý do chính** | Người nhận là Senior và cần dữ liệu đầy đủ để tự sửa | Người nhận đang trong trạng thái cảm xúc cao; danh sách dài sẽ thành cáo trạng | Nói hết: quá tải, người nhận nghe thành "mọi thứ đều tệ", không sửa được gì. Nói ít: người nhận sau này phát hiện có lý do khác chưa nói và mất tin cậy vào bạn |
| **Bảo vệ cảm xúc người nhận vs Nói rõ vị trí thật** | Tin không có tính quyết định về tương lai của họ | Có hệ quả về việc làm, vai trò, lương | Bảo vệ quá mức: người nhận rời phòng với hiểu biết sai về mức nghiêm trọng, và đó là sự bất công thật với họ. Nói quá thô: mất kênh, người nhận đóng lại và không sửa gì |
| **Minh bạch với team vs Bảo mật cho cá nhân** | Thay đổi cấu trúc mà cả team bị ảnh hưởng (đổi lead) | Kết quả promotion, đánh giá hiệu suất, lý do cá nhân | Minh bạch quá: người liên quan mất mặt, và mọi người học rằng chuyện của họ cũng có thể bị nói ra. Bảo mật quá: tin hành lang lấp chỗ trống và phiên bản trong tin hành lang luôn tệ hơn sự thật |

### Real-world Scenarios

**Tình huống 1 — Nói với một người rằng họ không được promote.**

Phiên bản sai (mở đầu vui vẻ rồi rẽ ngoặt, lý do mơ hồ, an ủi thay cho dữ liệu):

> **Tuấn:** Nam ơi, dạo này ổn không? Nghe nói con em mới vào lớp một à? À mà... cái vụ promotion,
> anh nói thật là hội đồng thấy em cũng tốt rồi, nhưng năm nay quota hơi ít, với lại còn nhiều anh em
> khác nữa. Em cứ cố gắng, quý sau chắc là được. Em đừng buồn nhé, anh vẫn đánh giá em cao.
> **Nam:** ...dạ.
> **Tuấn:** Ừ, có gì cứ nói anh nhé.

Bốn lỗi: small talk trước tin xấu (phá kênh small talk lâu dài); lý do quy về "quota" — một lý do
ngoài tầm kiểm soát của Nam, nên Nam không có gì để sửa và kết luận hợp lý là "kỳ sau cũng thế"; "chắc
là được" tạo một cam kết mơ hồ mà Tuấn không kiểm soát được; và "anh vẫn đánh giá em cao" mâu thuẫn
trực tiếp với kết quả, nên Nam phải chọn tin một trong hai, và Nam sẽ tin cái có hệ quả tài chính.

Phiên bản đúng:

> **Tuấn:** Nam vào đây anh nói một việc. Về promotion kỳ này: kết quả là không thông qua. Hội đồng
> họp ngày 10, quyết định đã chốt và anh không thay đổi được kỳ này. Anh nói lý do cụ thể, và anh cũng
> có phần anh làm sai trong chuyện này.
> **Nam:** (im lặng)
> **Tuấn:** (đợi) ...
> **Nam:** Em... em hơi bất ngờ. Tháng 1 anh nói là em có cơ hội.
> **Tuấn:** Đúng, và đó là phần anh sai. Anh nói "có cơ hội" mà không đưa cho em bốn tiêu chí và
> không kiểm tra giữa kỳ. Nếu tháng 4 anh ngồi với em một tiếng thì hôm nay em không phải bất ngờ.
> Anh xin lỗi về chỗ đó, và anh sẽ không để chuyện đó lặp lại.
> **Nam:** Vậy lý do là gì ạ?
> **Tuấn:** Hai lý do, theo đúng thứ tự hội đồng nói. Một: phạm vi ảnh hưởng. Hai quý vừa rồi mọi
> đóng góp của em nằm trong module Order. Tiêu chí Senior yêu cầu ít nhất một hạng mục có ảnh hưởng ra
> ngoài team. Hai: dẫn dắt kỹ thuật — quý II có bốn quyết định kiến trúc, không có cái nào do em đề
> xuất hay chủ trì viết RFC. Còn chất lượng và tốc độ thực thi thì đạt, cái đó không phải điểm trừ.
> **Nam:** Nhưng module Order là phần khó nhất, và em là người duy nhất làm được phần reconcile.
> **Tuấn:** Cái đó đúng, và anh đã nói trong hội đồng. Nó là lý do em ở mức "đạt" chứ không phải "dưới
> mức" ở tiêu chí ba. Nhưng nó không thay được tiêu chí một — và anh nói thẳng một chuyện: chính việc
> em là người duy nhất làm được phần đó đang là một phần của vấn đề, không phải bằng chứng ủng hộ.
> Senior được đo bằng việc nâng năng lực của người khác, không chỉ bằng việc mình làm được.
> **Nam:** ...Em hiểu.
> **Tuấn:** Giờ phần thực tế. Anh không nói "quý sau chắc được" vì anh không quyết được điều đó. Anh
> nói cái anh quyết được: từ tuần 30/7 em chủ trì RFC tích hợp đối tác giao vận, và em là người đại
> diện team làm việc với team Payment. Đó là hai hạng mục đúng vào tiêu chí một và hai. Ngày 15/10
> mình ngồi lại đối chiếu, không đợi tới kỳ review. Nếu tới đó em làm được hai việc đó thì anh sẽ đưa
> vào hội đồng tháng 1 với dữ liệu, và anh nói được là anh tin em đạt. Nếu em thấy hướng này không phải
> thứ em muốn — ví dụ em muốn đi sâu kỹ thuật hơn là dẫn dắt — thì đó cũng là một cuộc trò chuyện hợp
> lệ, mình nói tuần sau. Chiều nay anh gửi em bản ghi lại để mình cùng một phiên bản.

**Tình huống 2 — Nói với một người rằng họ sẽ không tiếp tục làm Tech Lead.**

Đây là cuộc trò chuyện chạm mạnh nhất vào tầng identity, và ở Việt Nam nó còn chạm vào kỳ vọng gia
đình về "làm quản lý". Nguyên tắc cốt lõi: **tách vai trò khỏi giá trị con người, và tách tường minh
bằng lời, đừng để họ tự suy ra.**

> **Linh (EM):** Hà, anh gọi em để nói về vai trò Tech Lead của team Order. Đây là quyết định của anh,
> anh đã cân nhắc ba tuần và anh nói cho em biết hôm nay: từ sprint sau, Nam sẽ nhận vai Tech Lead,
> em quay về vai Senior Engineer, giữ nguyên lương và giữ nguyên band.
> **Hà:** (im lặng dài)
> **Linh:** (đợi)
> **Hà:** Em làm sai chỗ nào ạ?
> **Linh:** Anh nói ba quan sát, rồi nói cách anh đọc nó, rồi em phản biện. Quan sát: ba sprint liên
> tiếp commitment hoàn thành dưới 60%. Hai người trong team nói với anh rằng họ không rõ ưu tiên trong
> tuần. Và trong ba tuần gần nhất em commit code vào buổi tối sáu ngày một tuần. Ba cái đó là sự việc,
> em kiểm tra được.
> **Hà:** Vâng, đúng ạ.
> **Linh:** Cách anh đọc — có thể anh đọc sai: em đang gánh phần kỹ thuật nặng nhất của team và đồng
> thời phải làm phần điều phối, và em chọn phần kỹ thuật vì đó là phần em làm giỏi và em thấy an toàn.
> Kết quả là phần điều phối không có ai làm, và em thì đang cạn. Anh có đọc sai chỗ nào không?
> **Hà:** Không... đúng ạ. Em biết em không kịp làm phần kia nhưng em không biết bỏ cái gì.
> **Linh:** Đó là chỗ anh làm sai. Anh giao em vai lead mà không giảm phần kỹ thuật của em, không dạy
> em phần điều phối, và không kiểm tra trong ba tháng đầu. Đây là lỗi thiết kế của anh. Nhưng anh
> không thể để team tiếp tục ba sprint nữa như thế, nên anh đổi.
> **Hà:** Em thấy như bị giáng chức. Em nói với team thế nào?
> **Linh:** Đó là câu quan trọng nhất, mình bàn phần đó lâu hơn. Ba việc anh sẽ làm. Một: về mặt thực
> tế, đây không phải giáng chức — lương và band giữ nguyên, và Tech Lead ở công ty mình là một vai
> trò được phân công, không phải một cấp bậc; anh sẽ nói tường minh điều đó với team, bằng chính câu
> này. Hai: câu chuyện công khai anh đề nghị, em xem có chấp nhận được không — "team Order cần một
> người tập trung vào phần điều phối và tích hợp; Hà quay lại tập trung vào phần lõi reconcile và
> kiến trúc dữ liệu, là phần khó nhất và không ai thay được". Cái đó là sự thật, không phải cách nói
> tránh. Ba: anh sẽ nói câu đó trong planning, không phải em; em không phải tự đi giải thích với từng
> người. Nếu em muốn sửa cách diễn đạt thì mình sửa bây giờ.
> **Hà:** Em muốn bỏ chữ "cần một người tập trung", nó nghe như em không tập trung.
> **Linh:** Được. "Hà chuyển sang chủ trì phần kiến trúc dữ liệu và reconcile cho cả hai team; Nam
> nhận phần điều phối team Order." Thế được không?
> **Hà:** Được ạ.
> **Linh:** Còn một việc nữa anh nói thẳng: nếu sau này em muốn quay lại nhánh quản lý, đường đó không
> đóng. Nhưng lần đó anh sẽ làm khác — có người mentor, giảm 50% phần code, và ba tháng kiểm tra một
> lần. Còn nếu em thấy nhánh Staff phù hợp hơn, thì đó là nhánh anh nghĩ em có lợi thế hơn thật, không
> phải anh nói cho dễ nghe. Tuần sau mình ngồi một tiếng riêng về chuyện đó.

Điểm kỹ thuật quan trọng: Linh nhận phần lỗi hệ thống thuộc về mình một cách cụ thể (không phải "anh
cũng có lỗi" chung chung), và dành thời lượng đáng kể cho câu chuyện công khai — vì trong bối cảnh
Việt Nam, phần "mất mặt" là phần chi phí lớn nhất và nếu không xử lý thì người đó sẽ nghỉ dù các điều
kiện vật chất không đổi.

**Tình huống 3 — Nói với khách hàng rằng scope không kịp (ODC, khách Nhật).**

Nguyên tắc: khi mang tin trễ tới một bên có quyền quyết scope, **mang kèm nguyên nhân đã phân tích và
2–3 phương án có chi phí** — nếu chưa có hai thứ đó thì bạn đang chuyển phần việc khó nhất (quyết cắt
gì) sang cho họ. Ngoại lệ duy nhất là khi mức bất định còn quá cao để có phương án: lúc đó vẫn báo,
nhưng nói rõ đây là báo sớm ở mức "có rủi ro, chưa có phương án", kèm ngày bạn sẽ có phương án. Và mang
càng sớm càng tốt, vì giá trị của tin trễ nằm ở số lựa chọn nó còn để lại cho khách.

Phiên bản sai (đợi tới sát hạn, không có phương án, đổ lỗi):

> "Chúng tôi rất tiếc phải thông báo rằng do requirement thay đổi nhiều lần từ phía khách hàng và môi
> trường test được cấp muộn, chúng tôi không thể hoàn thành đúng ngày 30/9. Chúng tôi sẽ cố gắng hết
> sức."

Vấn đề: gửi ngày 25/9 (khách không còn lựa chọn), nguyên nhân được phát biểu như một lời buộc tội
(dù đúng về mặt sự việc), và không có phương án nào — nên khách phải tự làm phần việc lẽ ra là của
bạn, tức là quyết định cắt gì.

Phiên bản đúng, gửi ngày 5/9, sau khi đã gọi điện nói trước với PM phía khách (thứ tự bắt buộc: nói
trước, viết sau):

> **Tech Lead (trong call, tiếng Anh đơn giản):** Tanaka-san, I have a schedule risk to report, and I
> want to report it now while we still have options. Based on the current remaining work, we need
> about three more weeks than the September 30 date. I will explain how I calculated it, then I have
> three options for you to choose from. I am not asking for a decision today, but I need one by
> September 12 — after that, option A is no longer possible.

Sau call, gửi văn bản trong cùng ngày (template ở chủ đề 6).

### Best Practices

- **Nói tin chính trong ba câu đầu.** Lý do: người nhận sẽ hiểu ra ở giữa câu bất kể bạn vòng thế
  nào, và khi đó họ mất khả năng nghe phần còn lại; vòng vo chỉ mua thêm sự nghi ngờ về động cơ của
  bạn.
- **Nêu rõ loại cuộc trò chuyện ngay từ đầu.** Lý do: nếu quyết định đã chốt mà bạn để người kia
  tưởng đang thảo luận, họ sẽ tiêu năng lượng thuyết phục rồi phát hiện mình bị đưa vào một vở kịch —
  tổn hại lớn hơn nội dung tin xấu.
- **Im lặng sau tin chính, đủ dài để thấy khó chịu.** Lý do: bạn đang trả cho phần sinh lý của họ
  thời gian nó cần; mọi câu bạn nói trong khoảng đó là lãng phí.
- **Nhận phần lỗi hệ thống của mình, cụ thể, có sự việc.** Lý do: nó khôi phục tính công bằng của cuộc
  trò chuyện và làm giảm nhu cầu phòng vệ; nhưng phải cụ thể, vì "anh cũng có lỗi" chung chung bị đọc
  là kỹ thuật xoa dịu.
- **Thoả thuận câu chuyện công khai trước khi rời phòng.** Lý do: phần lo lắng lớn nhất của người
  nhận thường không phải nội dung mà là "người khác sẽ nghĩ gì"; để họ tự xoay xử phần đó là bỏ họ
  giữa đường.
- **Gửi văn bản trong 24 giờ, có mục "phần tôi làm sai" và mục "phản đối thì reply".** Lý do: cả hai
  cần cùng một phiên bản; và một kênh phản đối tường minh giảm khả năng bất đồng chuyển thành sự bất
  mãn âm thầm.
- **Với tin xấu ra ngoài (khách hàng): gọi trước, viết sau, và trong cùng ngày.** Lý do: kênh nói xử
  lý được phản ứng và cho phép hỏi lại; kênh viết chốt được cam kết và phương án. Đảo thứ tự thì phía
  khách đọc văn bản trong trạng thái không có ai để hỏi, và phản ứng đầu tiên của họ sẽ được ghi lại
  bằng văn bản, khó rút lại.

### Anti-patterns

- **Trì hoãn tới khi không còn lựa chọn.** Cơ chế: giá trị của tin xấu nằm ở số phương án nó còn để
  lại; trì hoãn biến một quyết định thành một thông báo, và người nhận nhận ra rằng họ bị lấy mất
  quyền chọn. Dấu hiệu sớm: bạn đã lùi buổi nói chuyện này hơn một lần; bạn thấy nhẹ người khi hôm đó
  người kia nghỉ phép.
- **Truyền tin xấu qua kênh bậc thấp.** Slack, email một chiều, hoặc để người khác nói. Cơ chế: người
  nhận đọc kênh như một thông điệp về mức tôn trọng, và thông điệp đó tồn tại lâu hơn nội dung. Dấu
  hiệu sớm: bạn đang soạn một tin nhắn dài cho một việc đáng ra phải nói mặt đối mặt.
- **Dùng lý do ngoài tầm kiểm soát của người nhận làm lý do chính** ("quota ít", "công ty đang khó").
  Cơ chế: người nhận không còn gì để sửa, nên kết luận hợp lý là kỳ sau cũng thế, và họ bắt đầu tìm
  chỗ khác. Nếu lý do thật là quota thì phải nói thẳng đó là quota và nói rõ vị trí của họ trong hàng
  đợi, đừng trộn nó với lý do năng lực.
- **Nhượng bộ nội dung dưới áp lực cảm xúc.** Người kia khóc hoặc phản ứng mạnh và bạn nói "thôi để
  anh xem lại". Cơ chế: bạn dạy cả tổ chức rằng phản ứng cảm xúc là công cụ đàm phán hiệu quả, và bạn
  sẽ phải làm lại cuộc trò chuyện này trong điều kiện tệ hơn. Cách xử lý đúng: giữ nội dung, thay đổi
  nhịp — "anh giữ nguyên kết quả; mình dừng ở đây, chiều mai nói tiếp phần kế hoạch".
- **Nói bằng bị động và danh từ hoá để tránh chủ thể.** "Có một quyết định là em sẽ không tiếp tục vai
  lead." Cơ chế: người nhận không biết ai quyết nên không biết nói chuyện với ai, và họ đọc ra rằng
  bạn không dám nhận. Dùng "anh quyết định".
- **Không có follow-up văn bản.** Cơ chế: trong trạng thái cảm xúc cao, người nhận lưu lại rất ít; ba
  tháng sau hai bên có hai phiên bản, và tranh chấp về phiên bản làm hỏng cả phần đã thống nhất. Dấu
  hiệu sớm: xuất hiện câu "hồi đó anh có nói là...".
- **Biến cuộc trò chuyện khó thành một buổi khám phá cảm xúc kéo dài 90 phút.** Cơ chế: người nhận
  kiệt sức, và phần nội dung cần nhớ bị chôn trong khối lượng. Dấu hiệu sớm: buổi nói chuyện đi quá 45
  phút mà chưa chốt được một hành động nào.

### Khi nào KHÔNG nên áp dụng

- **Khi bạn chưa có quyền quyết định điều mình đang nói.** Nếu bạn nói với một người rằng họ sẽ đổi
  vai trò trong khi việc đó còn phụ thuộc phê duyệt của người khác, bạn tạo ra một tin xấu có thể bị
  đảo — và một tin xấu bị đảo phá uy tín của bạn nhiều hơn cả tin xấu thật. Chờ tới khi quyết định
  chắc, hoặc nói rõ trạng thái: "đây là đề xuất của anh, chưa chốt, anh nói trước để em không bị bất
  ngờ".
- **Khi vấn đề nằm ở phạm vi HR/pháp lý** (tố cáo quấy rối, vi phạm hợp đồng, nghi vấn gian lận).
  Đừng dùng framework hội thoại phát triển; chuyển cho đúng bộ phận và làm đúng quy trình, vì ở đó
  việc ghi nhận và bảo mật có ràng buộc khác.
- **Khi người kia đang trong trạng thái không thể xử lý thông tin** (vừa nhận tin gia đình, vừa qua
  một incident 20 giờ). Hoãn 24–48 giờ là quyết định đúng, miễn là bạn nói rõ có việc cần nói và hẹn
  thời điểm — treo lơ lửng "anh cần nói với em một chuyện" rồi để ba ngày là hình thức trừng phạt
  không cố ý.
- **Khi bạn đang bực.** Cuộc trò chuyện khó phát ra từ sự bực sẽ mang cường độ sai, và cường độ sai
  không rút lại được. Hoãn tới khi bạn viết được ba quan sát mà không có tính từ nào.
- **Khi bạn đang dùng "cuộc trò chuyện khó" để giải quyết một vấn đề vốn là của hệ thống.** Nếu ba
  Tech Lead liên tiếp đều thất bại ở cùng một team, cuộc trò chuyện khó thứ ba là sai địa chỉ: vấn đề
  ở thiết kế vai trò, ở phạm vi công việc, hoặc ở người phân công. Xem
  [09-to-chuc-va-scaling.md](/series/engineering-leedership/09-to-chuc-va-scaling/).
- **Khi mục tiêu thật của bạn là để mình cảm thấy đã làm đủ.** Có dạng cuộc trò chuyện được tổ chức để
  người lead có thể nói "tôi đã nói rồi" — với nội dung mờ đủ để không gây khó chịu. Cái đó tệ hơn
  không nói, vì nó tiêu một lần cơ hội và tạo bằng chứng giả rằng vấn đề đã được xử lý.

---

## 5. One-on-One

### Problem Statement

Một product startup 35 engineer, chuỗi sự việc trong bốn tháng:

- Tháng 3: Tech Lead của team Growth huỷ 1-1 với ba người trong team vì "tuần này chạy campaign". Lịch
  1-1 được đặt lại vào tuần sau, rồi lại huỷ.
- Tháng 4: một Senior nghỉ. Trong exit interview, câu chính: "Em không biết mình đang đi đâu ở đây."
- Tháng 5: hai engineer trong cùng team không nói chuyện với nhau ba tuần vì một tranh chấp về ranh
  giới module. Tech Lead biết vào tuần thứ tư, qua QA.
- Tháng 6: một Junior sau bốn tháng onboarding vẫn chưa deploy độc lập lần nào. Không ai theo dõi việc
  đó vì "bạn ấy không nói gì".
- Tháng 7: CTO hỏi Tech Lead "trong team ai đang có nguy cơ nghỉ", câu trả lời là "em nghĩ là không
  ai".

Bốn sự việc, một nguyên nhân: **kênh có băng thông cao nhất và độ tin cậy cao nhất của tổ chức đang
không hoạt động**, nên mọi tín hiệu sớm phải đi đường vòng — qua người thứ ba, qua QA, hoặc qua đơn
xin nghỉ. Hiện tượng đo được:

- **Tỉ lệ 1-1 thực hiện / 1-1 đã lên lịch trong 3 tháng.** Dưới 70% là tín hiệu đỏ. Con số này quan
  trọng hơn nội dung buổi 1-1, vì nó là chỉ số ưu tiên mà người khác đọc được.
- **Số ngày trung bình giữa hai buổi 1-1 của mỗi người.** Không đo trung bình toàn team — đo từng
  người, vì phân bố thường rất lệch: người lead thích nói chuyện thì 2 tuần/lần, người khó nói chuyện
  thì 3 tháng/lần, và người thứ hai chính là người có rủi ro cao.
- **Số tín hiệu nghỉ việc mà lead phát hiện trước đơn.** Nếu bằng 0 trong một năm ở team 8 người, kênh
  đang không hoạt động.
- **Tỉ lệ thời lượng người kia nói.** Dưới 50% nghĩa là buổi đó đang là kênh của bạn, không của họ.
- **Trong 1-1 có bàn tới chủ đề nào ngoài trạng thái công việc không.** Nếu 1-1 chỉ nói ticket, nó
  đang trùng lặp với daily standup và bạn đang trả hai lần cho cùng một thông tin.

### First Principles

**Cơ chế một: 1-1 là kênh có băng thông cao nhất và độ tin cậy cao nhất, và không có kênh nào thay
được.** Băng thông cao vì có ngôn từ, giọng, biểu cảm, và khả năng hỏi lại nhiều lượt trong thời gian
ngắn. Độ tin cậy cao vì tính riêng tư làm giảm chi phí của việc nói thật: không có khán giả, nên người
nói không phải bảo vệ vị thế trước người khác. Mọi kênh khác đều thiếu một trong hai: standup có băng
thông thấp và có khán giả; retro có khán giả; survey ẩn danh có độ tin cậy về nội dung nhưng không cho
phép đào sâu; Slack không có tầng cảm xúc. Vì vậy có một lớp thông tin **chỉ tồn tại trong 1-1**: mức
hài lòng thật, ý định nghỉ, xung đột giữa người với người, lo lắng về năng lực, và mâu thuẫn giữa mục
tiêu cá nhân với công việc đang làm. Không có 1-1 thì lớp thông tin này không biến mất — nó chỉ đi vào
kênh khác, và kênh mặc định là đơn xin nghỉ.

**Cơ chế hai: bất đối xứng thông tin và tần suất lấy mẫu.** Bạn có dashboard cho hệ thống với tần suất
lấy mẫu mỗi giây. Với con người, tần suất lấy mẫu của bạn bằng đúng nhịp 1-1. Nếu nhịp là một tháng,
bạn đang phát hiện vấn đề với độ trễ tối đa một tháng — và các quá trình quan trọng ở con người (mất
động lực, bắt đầu tìm việc, xung đột leo thang) có hằng số thời gian ngắn hơn thế. Nếu nhịp là ba
tháng, bạn thực chất không có cảm biến.

**Cơ chế ba: 1-1 là nơi trả nợ trước cho các cuộc trò chuyện khó.** Nền tin cậy không tạo ra được vào
đúng lúc cần. Khi bạn phải đưa feedback nặng hoặc thông báo tin xấu, mức độ nó được xử lý như dữ liệu
thay vì như một cuộc tấn công phụ thuộc vào vốn tin cậy đã tích trước đó — và 1-1 định kỳ là cơ chế
tích vốn đó với chi phí thấp. Đây là lý do 1-1 phải diễn ra cả khi không có vấn đề gì: bạn đang trả
tiền bảo hiểm.

**Cơ chế bốn: việc huỷ 1-1 là một thông điệp, không phải một sự vắng mặt.** Trong lý thuyết tín hiệu,
hành động tốn kém mang thông tin đáng tin hơn lời nói. Giữ một buổi 1-1 trong tuần bận nhất là một tín
hiệu tốn kém, nên nó được đọc là "người này thật sự ưu tiên tôi". Huỷ nó vì việc gấp cũng là một tín
hiệu tốn kém theo chiều ngược, và nó được đọc là "tôi ở dưới mức ưu tiên của một campaign". Mọi lời
giải thích sau đó không đảo được tín hiệu, vì hành động rẻ (lời nói) không đảo được kết luận từ hành
động đắt.

**Cơ chế năm: trong bối cảnh Việt Nam, 1-1 phải vượt qua hai rào cản đặc thù.** Một là kỳ vọng thứ bậc:
nhiều engineer chưa từng có trải nghiệm được người trên hỏi ý kiến thật, nên phản xạ đầu là báo cáo
tình hình và trả lời "em ổn ạ". Hai là việc nói về bản thân và về mong muốn cá nhân bị coi là không
khiêm tốn. Hệ quả: **1-1 hiệu quả ở Việt Nam cần nhiều buổi warm-up hơn**, và cần người lead nói ra
tường minh mục đích của buổi ("đây không phải báo cáo") kèm việc tự nói trước về những thứ mình cũng
đang khó — vì tính đối xứng làm giảm chi phí nói thật.

### Mental Model

**Model 1 — Ba loại 1-1, không được trộn.** Đây là mô hình có tác động lớn nhất trong chủ đề này.

| Loại | Mục tiêu | Tần suất tham khảo | Ai làm chủ agenda | Dấu hiệu đang bị trộn |
|---|---|---|---|---|
| **Status / unblock** | Loại bỏ vật cản trong tuần, đồng bộ ưu tiên | Hàng tuần, 15–20 phút, hoặc gộp vào kênh async | Lead có thể đề xuất | Nói hết 30 phút về ticket, hết giờ trước khi tới phần khác |
| **Growth / career** | Hướng phát triển, kỹ năng, phạm vi ảnh hưởng, con đường Ladder | Mỗi 4–6 tuần, 45–60 phút, có chuẩn bị hai phía | Người kia làm chủ | Câu trả lời là "em cũng chưa nghĩ" vì không được báo trước |
| **Trust / signal** | Lấy tín hiệu sớm, xây nền tin cậy, nói về cái khó | Hàng tuần hoặc hai tuần, 30 phút | Người kia làm chủ hoàn toàn | Lead nói 70% thời lượng; không có chủ đề nào ngoài công việc |

Vì sao không trộn: ba loại này cần ba trạng thái tâm lý khác nhau. Status cần chế độ vận hành, nhanh,
liệt kê. Growth cần chế độ phản tư, cần thời gian và không bị cắt. Trust cần cảm giác an toàn và không
có áp lực kết quả. Trộn chúng thì phần status luôn thắng, vì nó cấp bách và cụ thể — đây là ứng dụng
của quy luật cấp bách lấn quan trọng. Cách xử lý thực dụng: đẩy phần status sang kênh bất đồng bộ (một
message ngắn trước buổi), và bảo vệ thời lượng 1-1 cho hai loại còn lại.

**Model 2 — 1-1 như cảm biến trong control loop.** Bạn đang thiết kế một hệ thống quan sát: nhịp lấy
mẫu (tần suất), độ nhiễu (mức người kia nói thật), và độ trễ tới hành động (bạn làm gì với tín hiệu).
Cải thiện một tham số mà bỏ hai tham số kia là vô ích: 1-1 hàng tuần mà người ta chỉ nói "em ổn" thì
nhịp cao mà nhiễu 100%; 1-1 rất thật mà bạn không làm gì với thông tin thì độ trễ tới hành động là vô
hạn, và sau ba lần người ta ngừng cung cấp tín hiệu.

**Model 3 — Vốn tin cậy như một tài khoản.** Mỗi 1-1 được giữ đúng hẹn, mỗi lần bạn làm điều đã hứa,
mỗi lần bạn nói một điều thật về chính mình là một khoản nạp nhỏ. Mỗi cuộc trò chuyện khó, mỗi quyết
định người kia không thích, mỗi lần bạn không giữ lời là một khoản rút. Bạn không kiểm soát được thời
điểm phải rút — nó đến từ business. Bạn chỉ kiểm soát được việc nạp trước. Team có số dư âm thì mọi
quyết định khó đều gây thiệt hại nhân sự.

### Practical Framework

**Thiết lập (làm một lần):**

1. Đặt lịch định kỳ cho từng người, cùng khung giờ mỗi tuần, trên calendar, không phải "khi nào cần".
2. Buổi đầu tiên với mỗi người: nói rõ mục đích và luật của kênh này. Script mở kênh:

> "Anh muốn nói rõ buổi này dùng để làm gì, vì mỗi công ty làm một kiểu. Đây không phải chỗ báo cáo
> tiến độ — phần đó mình đã có standup và anh đọc board rồi. Buổi này là của em: em muốn nói gì thì
> mình nói cái đó. Ba loại việc anh thấy hữu ích nhất khi nói ở đây: thứ nhất, cái gì đang làm em mất
> thời gian mà em thấy không đáng; thứ hai, cái gì em muốn làm mà hiện tại chưa được làm; thứ ba,
> chuyện gì anh đang làm hoặc quyết định mà em thấy không ổn — chỗ này quan trọng nhất và cũng khó nói
> nhất, nên anh sẽ hỏi lại nhiều lần. Anh cũng nói rõ hai điều: những gì em nói ở đây anh không mang
> ra team, trừ khi em đồng ý hoặc trừ khi nó thuộc loại anh buộc phải báo (an toàn, pháp lý) — trường
> hợp đó anh sẽ nói cho em biết trước. Và buổi này anh không huỷ; nếu bận thật thì anh dịch, không
> bỏ."

3. Tạo một tài liệu chia sẻ cho mỗi người (một doc, hai người cùng ghi), có phần agenda và phần lịch
   sử.

**Chuẩn bị (10 phút trước mỗi buổi):**

- Đọc lại notes buổi trước; đánh dấu mọi việc bạn đã hứa và trạng thái của nó.
- Chuẩn bị một câu ghi nhận cụ thể (một hành vi thật, có sự việc — không phải lời khen chung).
- Chuẩn bị một câu hỏi mà bạn thật sự không biết câu trả lời.

**Trong buổi (30 phút, cấu trúc gợi ý):**

| Phút | Nội dung | Ghi chú |
|---|---|---|
| 0–3 | Đóng vòng các việc đã hứa buổi trước | Đặt trước mọi thứ khác. Đây là thứ xây tin cậy nhanh nhất |
| 3–15 | Agenda của người kia | Không cắt. Nếu họ không có gì, dùng bộ câu hỏi dưới |
| 15–22 | Agenda của bạn: feedback, ngữ cảnh tổ chức, điều chỉnh kỳ vọng | Một chủ đề, không ba |
| 22–28 | Một câu hỏi đào sâu / chủ đề growth | Xoay vòng theo tuần |
| 28–30 | Chốt: ai làm gì, khi nào; xác nhận notes | Nói ra bằng lời, không ngầm hiểu |

**Bộ câu hỏi thực dụng** (chọn 1–2 mỗi buổi, không hỏi cả danh sách):

*Lấy tín hiệu sớm:*

- "Tuần rồi việc gì làm em mất nhiều thời gian nhất mà em thấy không đáng?"
- "Có việc gì em đang chờ người khác mà chờ quá ba ngày không?"
- "Nếu em là anh, tuần sau em sẽ đổi một thứ gì trong cách team đang làm?"
- "Có chuyện gì em biết mà em nghĩ là anh chưa biết không?"
- "Trong hai tuần vừa rồi, có lúc nào em thấy khó nói thật trong họp không? Lúc nào?"

*Về động lực và rủi ro rời đi (hỏi định kỳ, không đợi có dấu hiệu):*

- "Phần việc nào trong tháng vừa rồi em làm mà thấy thời gian đi nhanh?" (và phần nào ngược lại)
- "Nếu sáu tháng nữa em vẫn làm đúng những việc như bây giờ, em thấy thế nào?"
- "Có ai liên hệ với em về cơ hội khác không? Anh hỏi thật, và anh không phản ứng gì cả." (câu này chỉ
  dùng được khi đã có nền tin cậy; dùng sớm thì gây sợ)
- "Điều gì đang làm em nghĩ tới việc rời khỏi đây, dù chỉ ở mức thoáng qua?"

*Về growth (báo trước một tuần để họ chuẩn bị):*

- "Trong 12 tháng tới, em muốn làm được việc gì mà hiện tại em chưa làm được?"
- "Kỹ năng nào em thấy đang chặn em nhiều nhất?"
- "Trong team ai làm được thứ mà em muốn học? Mình sắp xếp để em làm cùng người đó được không?"
- "Em muốn đi nhánh sâu kỹ thuật hay nhánh dẫn dắt người? Nếu chưa rõ thì mình thiết kế một thử nghiệm
  3 tháng để em có dữ liệu."

*Về chính bạn (bắt buộc, vì đây là kênh feedback ngược duy nhất bạn có):*

- "Trong hai tuần vừa rồi có quyết định nào của anh mà em thấy sai hoặc không hiểu?"
- "Có chỗ nào anh đang làm chậm việc của em không?"
- "Nếu anh phải bỏ một cuộc họp mà anh đang bắt em tham gia thì nên bỏ cái nào?"

**Ghi chép và theo dõi.** Một doc cho mỗi người, hai phần: phần chung (cả hai đọc và ghi) và phần
riêng của bạn (giả thuyết, quan sát về hiệu suất, tín hiệu rủi ro).

```
[1-1 NOTES] Trang (Senior FE) — chủ trì: Hà (Tech Lead)
Nhịp: thứ Tư 14:00, 30 phút, tuần/lần. Loại: trust + growth (status đi qua Slack thứ Ba).

=== 2026-07-22 ===
Việc đã hứa buổi trước:
  [x] Hà nói với PO về việc dừng 2 màn hình config -> đã nói, PO đồng ý dừng 1, giữ 1. Đã nói lại lý do.
  [ ] Hà tìm mentor design system -> chưa xong, hạn mới 29/7. (Đã nói rõ là chưa xong, không im.)

Trang nêu:
  - Ba sprint liên tiếp làm màn hình config; Trang tự đánh giá mức "thấy mình tiến bộ" là 3/10.
  - Muốn làm phần design system hoặc phần performance; đã tự đọc về nó buổi tối.
  - Nói (lần đầu): "em có nghĩ tới việc đổi chỗ, nhưng chưa làm gì cả."

Hà nêu:
  - Ghi nhận cụ thể: cách Trang viết lại mô tả bug ticket #530 -> QA không phải hỏi lại lần nào.
  - Ngữ cảnh: quý sau có 1 slot cho hạng mục nền tảng FE, chưa chốt ai.

Quan sát riêng (không share):
  - Tín hiệu rời đi: mức 3/5. Câu "đã nghĩ tới việc đổi chỗ" nói ở dạng nhẹ -> hiệu chỉnh lên.
  - Giả thuyết: vấn đề không phải lương, là phạm vi công việc và cảm giác không tiến bộ.
  - Rủi ro hệ thống: Trang là single point of failure cho màn hình điều phối.

Hành động (có chủ, có hạn):
  1. Hà: đề xuất Trang chủ trì hạng mục nền tảng FE quý sau, nói với EM trước 29/7.  -> Hà
  2. Hà: trong 2 sprint tới, phân bổ tối đa 1 màn hình config cho Trang.             -> Hà
  3. Trang: viết 1 trang đề xuất phạm vi design system, hạn 5/8.                     -> Trang
  4. Bắt đầu chuyển giao màn hình điều phối cho 1 người thứ hai (giảm SPOF).         -> Hà, hạn 15/8

Chủ đề quay lại buổi sau: kết quả nói với EM; mức "thấy mình tiến bộ" đo lại sau 4 tuần.
```

**Tiêu chí biết là buổi 1-1 đó có giá trị:** bạn học được ít nhất một điều bạn chưa biết; có ít nhất
một hành động có chủ và có hạn (hoặc một câu trả lời rõ ràng rằng sẽ không có hành động và vì sao);
và người kia nói trên 50% thời lượng.

### Trade-off

| Trục | Nghiêng bên trái khi | Nghiêng bên phải khi | Cái giá |
|---|---|---|---|
| **Tần suất cao (tuần) vs Tần suất thấp (2–4 tuần)** | Người mới dưới 6 tháng; người đang có vấn đề; team dưới 8 người; giai đoạn thay đổi lớn | Người Senior tự chủ cao, đã làm cùng lâu; bạn có trên 10 báo cáo trực tiếp | Tần suất cao với người tự chủ: thành nghi lễ, cả hai không có gì để nói, giá trị tín hiệu giảm. Tần suất thấp: độ trễ phát hiện vấn đề vượt hằng số thời gian của vấn đề |
| **Người kia làm chủ agenda vs Lead làm chủ** | Mục tiêu là lấy tín hiệu và xây tin cậy | Có việc cần truyền đạt bắt buộc; người kia chưa quen và không biết nói gì | Người kia làm chủ hoàn toàn với người chưa quen: buổi trống, họ căng thẳng. Lead làm chủ: biến thành status report, mất chức năng cảm biến |
| **Ghi chép chi tiết vs Không ghi** | Cần theo dõi cam kết, cần dữ liệu cho Performance Review, nhiều người | Cuộc trò chuyện rất nhạy cảm, người kia thấy bị lập hồ sơ | Ghi hết: người kia nói dè dặt hơn khi thấy bạn gõ. Không ghi: bạn quên cam kết của mình, và đó là cách nhanh nhất phá vốn tin cậy |
| **Bảo mật tuyệt đối vs Hành động dựa trên thông tin** | Thông tin thuộc về cá nhân, chưa chín | Thông tin cho thấy rủi ro hệ thống hoặc rủi ro cho người khác | Bảo mật tuyệt đối: bạn biết mà không sửa được gì, và cùng vấn đề tái diễn với người khác. Hành động không xin phép: nguồn tin bị lộ, kênh đóng vĩnh viễn. Cách đúng: xin phép tường minh và thoả thuận cách diễn đạt |

### Real-world Scenarios

**Tình huống 1 — 1-1 biến thành status report (ODC, khách Singapore, 22 engineer).**

Phiên bản sai:

> **Tuấn:** Ok Khoa, tuần này thế nào, mấy task kia xong chưa?
> **Khoa:** Dạ task API xong rồi, còn phần integration đang chờ môi trường của khách. Task migration
> em làm được 60%.
> **Tuấn:** Ừ, cái migration cố cuối tuần nhé. Còn cái API nhớ viết test. À, thứ Năm có họp với
> khách, em chuẩn bị slide phần của em.
> **Khoa:** Dạ vâng.
> **Tuấn:** Ok, có gì không?
> **Khoa:** Dạ không ạ.
> **Tuấn:** Ừ, tuần sau gặp lại.

Bốn phút, và Tuấn vừa trả một slot 30 phút để lấy thông tin đã có trên board. Không có một bit thông
tin mới nào. Câu "có gì không?" ở cuối, sau khi Tuấn đã giao thêm ba việc, gần như không thể được trả
lời bằng một vấn đề thật.

Phiên bản đúng:

> **Tuấn:** Phần trạng thái task anh đã đọc trên board và trong message em gửi sáng nay rồi, mình
> không nói lại. Trước hết, việc anh hứa tuần trước: anh nói với khách về việc cấp môi trường test —
> anh đã nói, họ trả lời là cuối tuần này. Anh đánh giá là họ sẽ trễ, nên anh đã chuẩn bị phương án
> mock để em không bị chặn. Việc thứ hai anh hứa là tìm người review phần security cho em, cái đó anh
> chưa làm, anh xin lỗi, hạn mới là thứ Sáu.
> **Khoa:** Dạ, không sao ạ.
> **Tuấn:** Giờ tới phần của em. Tuần rồi việc gì làm em mất nhiều thời gian nhất mà em thấy không
> đáng?
> **Khoa:** ...Thật ra là phần viết báo cáo hàng tuần cho khách. Em mất khoảng ba tiếng, mà em không
> biết ai đọc nó. Lần trước em viết sai một số liệu, hai tuần không ai nói gì.
> **Tuấn:** Ba tiếng một tuần là gần 8% thời gian của em. Anh sẽ kiểm tra xem ai đọc thật và họ dùng
> nó làm gì; nếu không ai dùng thì mình cắt, nếu có người dùng thì mình đổi sang template ngắn hơn.
> Anh trả lời em trước thứ Tư tuần sau. Còn gì nữa không?
> **Khoa:** ...Còn một việc, em không biết có nên nói không.
> **Tuấn:** Em nói.
> **Khoa:** Trong họp với khách, khi họ nói tiếng Anh nhanh, em thường không nghe hết. Em vẫn "yes,
> yes" rồi sau đó đoán. Có lần em đoán sai và làm lại hai ngày. Em không dám nói vì sợ anh nghĩ em
> không đủ trình.
> **Tuấn:** Cảm ơn em, đây là thứ quan trọng nhất trong buổi hôm nay và nó không phải vấn đề của
> riêng em — anh cũng mất khoảng 20% nội dung trong các call đó. Mình làm ba việc: một, từ tuần sau
> mọi call với khách đều bật recording và anh sẽ yêu cầu họ gửi minutes; hai, em có quyền nói câu
> "could you repeat that, I want to make sure I got it right" bất cứ lúc nào — nếu có ai phản ứng thì
> đó là việc của anh; ba, anh sẽ dạy em ba câu cụ thể để dùng trong call, mình tập 10 phút cuối buổi
> 1-1 trong ba tuần tới.

**Tình huống 2 — Người nói "em ổn" trong sáu buổi liên tiếp.**

Đây là tình huống phổ biến nhất ở Việt Nam và nó không giải quyết được bằng cách hỏi kỹ hơn. Ba can
thiệp theo thứ tự chi phí tăng dần: (1) *Bạn nói thật trước.* "Tuần rồi anh ra một quyết định anh nghĩ
là sai — anh đồng ý bỏ test cho phần thanh toán để kịp demo, và giờ anh đang phải trả giá." Tính đối
xứng giảm chi phí nói thật của người kia. (2) *Đổi từ câu hỏi mở sang câu hỏi hẹp và cụ thể.* "Em ổn
không" không trả lời được; "hôm thứ Ba khi anh nói trong họp là sẽ làm theo phương án của anh Nam, em
nghĩ gì lúc đó?" thì trả lời được. (3) *Đổi định dạng.* Đi ăn trưa, đi bộ, hoặc chuyển sang hỏi qua
văn bản trước — một số người viết thật hơn nhiều so với khi nói. Nếu sau ba tháng vẫn không có tín
hiệu nào, đó là dữ liệu về Psychological Safety của cả team, không phải về tính cách một người.

**Tình huống 3 — Ba góc nhìn về việc huỷ 1-1.** Tech Lead huỷ 1-1 với cả team trong tuần chạy campaign.

*Nhìn từ IC:* "Chuyện của tôi không quan trọng bằng campaign." Với người đang cân nhắc nghỉ, đây là
một điểm dữ liệu ủng hộ quyết định. Với người đang có vấn đề chưa nói, đây là lý do hợp lý để không
nói nữa.

*Nhìn từ Tech Lead:* một quyết định hợp lý về phân bổ thời gian trong tuần cao điểm — tiết kiệm được
4 giờ để làm việc gấp. Điểm mù: 4 giờ đó là chi phí hiện hữu và dễ thấy, còn cái mất (độ trễ phát hiện
vấn đề tăng, một khoản rút khỏi vốn tin cậy của 8 người) là chi phí trễ và không quy được cho ai, nên
không vào phép tính.

*Nhìn từ Manager:* dấu hiệu của một vấn đề khác — Tech Lead đang không có đủ slack để làm cả phần điều
phối và phần kỹ thuật, và giải pháp mà họ tự tìm ra là hy sinh phần ít cấp bách nhất. Can thiệp đúng
không phải nhắc "phải giữ 1-1", mà là giảm tải kỹ thuật của Tech Lead trong tuần cao điểm, hoặc cho
phép rút ngắn 1-1 xuống 15 phút thay vì huỷ (giữ tín hiệu, giảm chi phí), và làm rõ rằng 1-1 nằm trong
phần công việc được đánh giá của vai trò lead, không phải phần tuỳ chọn.

### Best Practices

- **Không huỷ; dịch hoặc rút ngắn.** Lý do: giá trị tín hiệu của việc giữ hẹn cao hơn giá trị nội dung
  của một buổi cụ thể; 15 phút giữ được tín hiệu, huỷ thì phát tín hiệu ngược.
- **Bắt đầu bằng việc đóng vòng các cam kết của bạn, kể cả cam kết chưa làm được.** Lý do: đây là bằng
  chứng rẻ nhất và mạnh nhất rằng kênh này dẫn tới hành động; nói thẳng "anh chưa làm, hạn mới là X"
  xây tin cậy tốt hơn im lặng.
- **Tách status ra khỏi 1-1 bằng một message async trước buổi.** Lý do: nếu không tách, phần cấp bách
  sẽ ăn hết thời lượng của phần quan trọng, mỗi tuần, không có ngoại lệ.
- **Báo trước một tuần khi buổi tới là buổi growth.** Lý do: câu hỏi về định hướng nghề nghiệp không
  trả lời được ứng khẩu; hỏi không báo trước thì nhận được "em cũng chưa nghĩ" và bạn kết luận sai rằng
  họ không có mong muốn gì.
- **Hỏi về chính bạn mỗi 3–4 buổi, bằng câu hỏi hẹp.** Lý do: đây là kênh feedback ngược duy nhất bạn
  có ở vị trí lead; và câu hỏi hẹp ("quyết định nào của anh tuần rồi em thấy không hiểu") an toàn hơn
  câu hỏi rộng nên có xác suất được trả lời thật cao hơn.
- **Ghi lại và cho người kia thấy phần chung của notes.** Lý do: minh bạch về cái được ghi làm giảm lo
  ngại bị lập hồ sơ, và giúp cả hai theo dõi được cam kết qua nhiều tháng.
- **Xin phép tường minh trước khi hành động trên thông tin nghe được.** Lý do: một lần thông tin bị
  truy về nguồn là đủ để đóng kênh với cả team, vì mọi người sẽ biết.
- **Với người mới, dùng tần suất cao trong 8 tuần đầu rồi giảm.** Lý do: giai đoạn onboarding có mật
  độ vấn đề cao nhất và người mới có ít kênh khác nhất; đây là đầu tư có lợi tức cao nhất trong toàn
  bộ vòng đời làm việc của họ.

### Anti-patterns

- **1-1 biến thành status report.** Cơ chế: bạn tiêu tài nguyên đắt nhất (thời gian đồng bộ, kênh
  riêng tư) để lấy thông tin đã có ở kênh rẻ hơn, và mất luôn lớp thông tin chỉ có ở kênh này. Dấu
  hiệu sớm: buổi 1-1 kết thúc sớm 15 phút; bạn không học được gì mới; nội dung trùng board.
- **Huỷ 1-1 khi bận.** Cơ chế: phát tín hiệu ưu tiên rõ hơn mọi lời nói, và tín hiệu này không đảo
  được bằng giải thích. Dấu hiệu sớm: bạn đã huỷ hai lần liên tiếp với cùng một người — thường là
  người khó nói chuyện nhất, tức người có rủi ro cao nhất.
- **Chỉ 1-1 khi có vấn đề.** Cơ chế: buổi 1-1 trở thành tín hiệu của tin xấu, nên lời mời họp riêng
  gây lo lắng, và người ta chuẩn bị phòng vệ trước khi vào. Bạn cũng mất khả năng tích vốn tin cậy
  trước khi cần. Dấu hiệu sớm: khi bạn nhắn "em có 30 phút không", câu trả lời đầu tiên là "có việc gì
  hả anh".
- **Lead nói 80% thời lượng.** Cơ chế: buổi đó thành kênh phát của bạn, và cảm biến bị tắt. Dấu hiệu
  sớm: bạn thấy buổi 1-1 hiệu quả và người kia thì im lặng nhiều.
- **Hứa trong 1-1 rồi không làm và không nói lại.** Cơ chế: mỗi lần như thế là một khoản rút lớn; sau
  hai lần, thông tin đắt sẽ không được đưa vào kênh nữa vì người ta kết luận kênh không dẫn tới đâu.
  Dấu hiệu sớm: notes của bạn có nhiều dòng chưa đánh dấu xong từ ba buổi trước.
- **Dùng 1-1 để bàn về người khác.** Cơ chế: người kia hiểu rằng bạn cũng bàn về họ trong 1-1 với
  người khác, nên kênh mất tính riêng tư trong nhận thức của họ. Dấu hiệu sớm: bạn đang xin ý kiến về
  hiệu suất của đồng nghiệp trong 1-1 thường lệ.
- **1-1 nhóm.** Gộp ba người để tiết kiệm thời gian. Cơ chế: có khán giả, nên đúng lớp thông tin bạn
  cần lấy sẽ không xuất hiện. Đó là một buổi họp team, gọi đúng tên nó.

### Khi nào KHÔNG nên áp dụng

- **Khi bạn không có quyền hành động trên thông tin thu được.** Nếu bạn là Tech Lead không có ảnh
  hưởng nào tới lương, vai trò, hay phân bổ công việc, việc mở kênh 1-1 kiểu growth sẽ tạo kỳ vọng mà
  bạn không đáp ứng được. Xử lý: nói rõ ngay từ buổi đầu bạn quyết được gì và không quyết được gì, và
  chỉ mở phần bạn kiểm soát.
- **Khi số người trực tiếp quá lớn.** Với 15 người trực tiếp, 1-1 hàng tuần chất lượng là bất khả thi
  (7.5 giờ mỗi tuần chỉ riêng thời lượng, chưa tính chuẩn bị và follow-up). Đó là vấn đề cấu trúc, và
  giải pháp là đổi cấu trúc, không phải rút 1-1 xuống 10 phút; xem
  [09-to-chuc-va-scaling.md](/series/engineering-leedership/09-to-chuc-va-scaling/).
- **Trong tuần có incident lớn hoặc release nặng.** Rút xuống 10–15 phút và chỉ hỏi một câu ("có gì
  đang chặn em, có gì gấp không"), rồi quay lại nhịp bình thường tuần sau. Cố giữ đủ 30 phút cho phần
  growth trong tuần đó là sai ưu tiên và người kia cũng không ở trạng thái nói được.
- **Với contractor hoặc người sắp rời đi trong hai tuần.** Phần growth không còn ý nghĩa; đổi mục tiêu
  sang chuyển giao tri thức và thu thập thông tin về hệ thống — đó là một loại cuộc trò chuyện khác và
  nên nói rõ.
- **Khi vấn đề là thiếu Psychological Safety ở cấp team.** Nếu người ta không dám nói trong 1-1 vì lần
  trước có người bị trừng phạt vì nói thật, tăng tần suất 1-1 sẽ chỉ tăng số lần nghe câu "em ổn".
  Sửa nguyên nhân ở [03-team-leadership.md](/series/engineering-leedership/03-team-leadership/).
- **Khi bạn dùng 1-1 như cách duy nhất để quản lý.** Có dạng lead xử lý mọi thứ trong kênh riêng: giao
  việc, chốt kiến trúc, giải quyết xung đột. Kết quả là team không có ngữ cảnh chung, mọi thông tin
  phải đi qua lead, và lead trở thành điểm nghẽn — đúng lỗi mô tả ở
  [12-anti-patterns.md](/series/engineering-leedership/12-anti-patterns/). 1-1 là kênh bổ sung, không thay được kênh công khai.

---

## 6. Stakeholder Management

### Problem Statement

Ba mảnh của cùng một hệ thống hỏng, ở ba dạng tổ chức khác nhau:

*Một — product startup, PO là founder.* Tech Lead đề xuất hai sprint để trả technical debt ở tầng
thanh toán. Founder trả lời "cái đó để sau, giờ mình phải ra tính năng". Ba tháng sau, một sự cố thanh
toán kéo dài 40 phút vào ngày campaign. Founder hỏi "sao không ai cảnh báo tôi". Tech Lead có bằng
chứng đã cảnh báo — trong một message Slack, dùng từ "coupling cao", "khó test", "nợ kỹ thuật". Founder
đã đọc và không hiểu đó là một rủi ro doanh thu.

*Hai — ODC, khách Nhật quyết scope.* Trong 6 tháng, khách thêm 23 yêu cầu nhỏ, mỗi cái "chỉ một chút".
Team Việt nhận hết vì "khách là khách". Đến mốc phase, trễ 4 tuần. Khách kết luận rằng năng lực team
yếu. Không có bất kỳ tài liệu nào ghi lại rằng scope đã tăng 31%.

*Ba — bộ phận IT của một doanh nghiệp truyền thống.* Dự án thay hệ thống quản lý kho, đã chạy 8 tháng.
Phòng nghiệp vụ (vận hành kho) không dùng hệ thống mới, vẫn dùng Excel song song. Trong mọi buổi họp
họ nói "chúng tôi ủng hộ chuyển đổi số". Không ai trong team IT từng hỏi: nếu hệ thống mới chạy thật
thì công việc và vị thế của họ thay đổi thế nào.

Hiện tượng đo được của việc quản lý stakeholder yếu:

- **Số quyết định kỹ thuật bị đảo sau khi đã bắt đầu triển khai.** Mỗi lần đảo là bằng chứng rằng người
  có quyền quyết không được thuyết phục bằng ngôn ngữ của họ ngay từ đầu.
- **Scope creep không được ghi nhận.** Đếm số yêu cầu phát sinh có ghi lại kèm ước lượng so với tổng
  số yêu cầu phát sinh. Tỉ lệ dưới 50% nghĩa là bạn đang tích luỹ trách nhiệm mà không tích luỹ bằng
  chứng.
- **Tần suất stakeholder biết tin xấu từ nguồn khác trước khi biết từ bạn.** Mỗi lần xảy ra, mức tin
  cậy vào toàn bộ báo cáo của bạn giảm — kể cả các phần đúng.
- **Tỉ lệ chấp nhận thực tế của hệ thống mới** (số người dùng thật / số người lẽ ra phải dùng). Với dự
  án chuyển đổi số, đây là chỉ số duy nhất đáng tin.
- **Số đề xuất kỹ thuật được duyệt so với số đề xuất đã đưa.** Tỉ lệ rất thấp trong nhiều quý thường
  không có nghĩa là đề xuất sai; nó có nghĩa là chúng được viết bằng ngôn ngữ mà người quyết không
  dùng để quyết.

### First Principles

**Cơ chế một: mỗi stakeholder tối ưu một hàm mục tiêu khác nhau, và các hàm này không đồng nhất.** Đây
là điểm cốt lõi. Founder tối ưu tốc độ tới product-market fit dưới ràng buộc runway. Sales tối ưu số
hợp đồng chốt được quý này. CFO tối ưu chi phí và dự đoán được của chi phí. Trưởng phòng nghiệp vụ tối
ưu việc công việc hàng ngày của phòng không bị gián đoạn và vị thế của phòng không bị suy giảm. Khách
ODC tối ưu việc mình không bị trách bởi cấp trên của họ. Engineer tối ưu tính đúng đắn, khả năng bảo
trì, và trải nghiệm làm việc của chính mình.

Hệ quả: **xung đột giữa các bên phần lớn là xung đột hàm mục tiêu, không phải xung đột tính cách.** Đây
không phải cách nói cho dễ nghe, nó là một khẳng định có tính dự báo: nếu bạn thay founder đó bằng một
người dễ tính hơn nhưng cùng vị trí và cùng ràng buộc runway, bạn sẽ nhận được cùng một hành vi về ưu
tiên. Vì vậy chiến lược đúng không phải là làm cho họ dễ tính hơn, mà là **tìm giao của các hàm mục
tiêu** hoặc **dịch đề xuất của mình sang biến số nằm trong hàm mục tiêu của họ**.

**Cơ chế hai: principal–agent và bất đối xứng thông tin.** Bạn biết về hệ thống nhiều hơn stakeholder
rất nhiều lần. Điều này cho bạn một quyền lực thật: bạn chọn thông tin nào được truyền. Và nó cho bạn
một nghĩa vụ tương ứng: stakeholder không thể tự phát hiện điều bạn không nói. Khi một Tech Lead nói
"cái này làm được trong hai tuần" mà không nói "với điều kiện không có gì phát sinh, và xác suất điều
đó khoảng 40%", họ đang khai thác bất đối xứng thông tin, dù không cố ý. Về dài hạn, cơ chế này tự
phá: sau vài lần ước lượng trượt, stakeholder ngừng tin mọi con số của bạn và bắt đầu tự nhân hệ số —
lúc đó bạn mất khả năng truyền tin chính xác cả khi bạn nói đúng.

**Cơ chế ba: no-surprise principle có nền tảng ở lý thuyết ra quyết định.** Một stakeholder biết tin
xấu sớm còn có thể hành động (cắt scope, thêm người, đổi cam kết với bên thứ ba, chuẩn bị truyền
thông). Biết muộn thì họ chỉ còn chịu hậu quả, và điều họ mất không chỉ là kết quả mà là **quyền được
lựa chọn**. Con người phản ứng với việc bị lấy mất quyền lựa chọn mạnh hơn nhiều so với phản ứng với
kết quả xấu. Đây là lý do cơ học tại sao "trễ mà báo sớm" được xử lý khác hoàn toàn "trễ mà báo muộn",
dù kết quả cuối giống nhau. Với khách Nhật, cơ chế này còn mạnh hơn: một sự cố được báo sớm và có
phương án thường được coi là dấu hiệu của sự chuyên nghiệp, còn một sự cố được báo muộn phá vỡ niềm
tin vào toàn bộ quy trình.

**Cơ chế bốn: giá trị kỹ thuật không tự dịch sang giá trị business.** "Coupling cao", "thiếu test",
"nợ kỹ thuật" là các phát biểu về thuộc tính nội tại của hệ thống. Chúng chỉ có nghĩa với người quyết
khi được chuyển thành một trong bốn trục: **doanh thu, rủi ro, chi phí, thời gian ra thị trường.** Việc
dịch này không phải kỹ năng giao tiếp trang trí — nó là kỹ năng phân tích, vì bạn phải thật sự tính
được hệ quả. "Nợ kỹ thuật ở tầng thanh toán" dịch thành: "hiện tại mỗi thay đổi ở tầng thanh toán mất
trung bình 6 ngày thay vì 2 ngày, tức chúng ta ra tính năng thanh toán chậm gấp ba lần đối thủ; và xác
suất sự cố trong giờ cao điểm campaign là đáng kể vì không có test cho nhánh timeout của cổng thanh
toán — một sự cố 40 phút vào ngày campaign, theo doanh thu giờ cao điểm tháng trước, tương đương
khoảng X. Hai sprint trả nợ đưa thời gian thay đổi về 2–3 ngày và bỏ được nhánh không test."

**Cơ chế năm: kháng cự thay đổi thường là hành vi hợp lý dưới một hàm mục tiêu bạn chưa nhìn thấy.**
Phòng nghiệp vụ ở tình huống ba không "kháng cự chuyển đổi số vì lạc hậu". Họ đang bảo vệ ba thứ có
thật: quy trình mà họ đã tối ưu qua nhiều năm và biết rõ các trường hợp ngoại lệ, số lượng nhân sự của
phòng, và việc họ không bị trách khi có sự cố. Hệ thống mới đe doạ cả ba, và không ai giải thích cho họ
điều gì sẽ thay thế. Nếu bạn coi đó là sự lạc hậu, bạn sẽ chọn giải pháp truyền thông và đào tạo — và
nó sẽ thất bại, vì bạn đang giải một bài toán khác với bài toán thật.

### Mental Model

**Model 1 — Stakeholder map theo power/interest.** Bốn ô, mỗi ô một chiến lược khác nhau và một nhịp
liên lạc khác nhau:

```
  Quyền lực cao │  KEEP SATISFIED          │  MANAGE CLOSELY
                │  (CFO, giám đốc khối)    │  (PO/founder, khách chính,
                │  Nhịp: tóm tắt định kỳ,  │   trưởng phòng nghiệp vụ)
                │  chỉ escalate khi cần    │  Nhịp: tuần, chủ động,
                │                          │  có kênh riêng cho tin xấu
                ├──────────────────────────┼──────────────────────────
  Quyền lực thấp│  MONITOR                 │  KEEP INFORMED
                │  (team lân cận không     │  (QA, Support, Data,
                │   liên quan)             │   team phụ thuộc API của bạn)
                │  Nhịp: broadcast         │  Nhịp: 2 tuần, hoặc khi có
                │                          │  thay đổi ảnh hưởng họ
                └──────────────────────────┴──────────────────────────
                   Mức quan tâm thấp           Mức quan tâm cao
```

Sai lầm phổ biến nhất: dồn hết năng lượng vào ô "Manage closely" và bỏ ô "Keep informed" — rồi bị chặn
ở phút cuối bởi một team mà bạn không nghĩ là stakeholder (Support không được đào tạo nên không xử lý
được ticket; Data phát hiện schema đổi làm vỡ pipeline của họ vào ngày release).

**Model 2 — Bảng dịch bốn trục.** Với mỗi đề xuất kỹ thuật, điền bảng này trước khi trình bày. Không
điền được ô nào thì đó là chỗ lập luận của bạn còn yếu, không phải chỗ người nghe không hiểu.

| Đề xuất kỹ thuật | Doanh thu | Rủi ro | Chi phí | Time to market |
|---|---|---|---|---|
| Tách service thanh toán | Cho phép thêm cổng thanh toán mới trong 1 tuần thay vì 1 tháng → mở được kênh thanh toán mới trong Q4 | Giảm bán kính ảnh hưởng: sự cố thanh toán không kéo theo toàn bộ đơn hàng | +2 instance (khoảng X/tháng); 2 sprint công | Chậm 2 sprint cho tính năng hiện tại, nhanh hơn cho mọi tính năng thanh toán sau đó |
| Thêm test cho nhánh timeout cổng thanh toán | Không trực tiếp | Bỏ được nhánh không test đang xử lý tiền thật; hiện là nguyên nhân của 2/3 incident P1 quý trước | 4 ngày công | Không ảnh hưởng |
| Nâng version framework | Không trực tiếp | Version hiện tại hết hỗ trợ bảo mật từ tháng 12 → rủi ro compliance khi audit | 6 ngày công | Chậm 1 sprint nếu làm trong quý này; đắt gấp 3 nếu để sang năm |

**Model 3 — Trust như một tài khoản có lãi kép, và tin xấu là khoản nạp lớn nhất.** Nghịch lý mà nhiều
người không tin tới khi tự trải nghiệm: **báo tin xấu sớm, có phân tích và có phương án, là cách xây
tin cậy nhanh nhất với stakeholder.** Cơ chế: nó cung cấp bằng chứng rằng báo cáo của bạn có tính chọn
lọc thấp — bạn nói cả cái bất lợi cho mình. Sau vài lần như thế, khi bạn nói "cái này ổn", câu đó mang
thông tin thật. Ngược lại, người chỉ báo tin tốt tạo ra một hệ quả tàn khốc: mọi báo cáo tốt của họ
đều bị chiết khấu.

### Practical Framework

**Bước 1 — Lập stakeholder map (làm một lần mỗi quý, 30 phút).** Với mỗi bên, điền năm trường:

```
Tên / vai:            Trưởng phòng Vận hành kho (chị Trang)
Hàm mục tiêu thật:    Kho không bị gián đoạn; không bị trách khi có sự cố;
                      giữ được số nhân sự của phòng.
Họ bị đo bằng gì:     Tỉ lệ giao hàng đúng hạn; số lần kiểm kê lệch.
Quyền lực / quan tâm: Cao / Cao  -> Manage closely
Kênh và nhịp:         Gặp trực tiếp 2 tuần/lần tại kho (không phải họp online);
                      không dùng email vì không đọc.
Điều họ sợ nhất:      Ngày đầu chạy hệ thống mới trùng ngày cao điểm; mất Excel
                      mà chưa tin hệ thống mới.
```

Trường "điều họ sợ nhất" là trường có giá trị cao nhất và hay bị bỏ. Nó cho bạn biết phải đưa cái gì
vào thiết kế để đổi được sự hợp tác — ở tình huống trên: cho phép chạy song song Excel trong 6 tuần và
cung cấp báo cáo đối chiếu tự động, tức là bạn mua sự hợp tác bằng cách giảm rủi ro cho họ, không bằng
cách thuyết phục.

**Bước 2 — Thiết lập nhịp cập nhật chủ động.** Nguyên tắc: **bạn định nghĩa nhịp, không để stakeholder
phải đi hỏi.** Khi họ phải hỏi, hai điều xảy ra: bạn mất vị thế người kiểm soát thông tin, và họ đã ở
trong trạng thái lo lắng trước khi đọc thông tin của bạn.

Template status update hàng tuần (dùng cho PO/founder hoặc khách ODC; giữ dưới một trang, cùng thứ tự
mỗi tuần để người đọc học được cách quét):

```
[WEEKLY STATUS] Project Kestrel — Week 30 (2026-07-20 → 2026-07-24)
Owner: Tuấn (Tech Lead)  |  Ngày phát hành: 2026-07-24  |  Bản: #18

1. TÌNH TRẠNG TỔNG THỂ:  AMBER  (tuần trước: GREEN)
   Lý do đổi màu: môi trường UAT của khách chậm 6 ngày so với kế hoạch;
   ảnh hưởng trực tiếp tới mốc UAT 05/08.

2. CẦN QUYẾT ĐỊNH TỪ PHÍA ANH/CHỊ (deadline quyết định)
   - [ ] Chọn 1 trong 2 phương án cho phần đối soát: A (giữ scope, UAT lùi 1 tuần)
         hoặc B (bỏ báo cáo tổng hợp khỏi phase này, giữ mốc UAT).
         Cần quyết trước: 29/07. Sau ngày này, phương án B không còn khả thi.
   - [ ] Xác nhận ai là người ký duyệt UAT phía nghiệp vụ. Cần trước: 31/07.

3. TIẾN ĐỘ SO VỚI KẾ HOẠCH
   - Hoàn thành tuần này: API đối soát (7/7 endpoint), migration dữ liệu lịch sử (100%).
   - Kế hoạch tuần sau: màn hình đối soát, chuẩn bị bộ test UAT.
   - Tổng thể phase: 68% hạng mục đã xong (kế hoạch: 74%).

4. RỦI RO (chỉ liệt kê cái đang mở, kèm chủ và hạn)
   | # | Rủi ro | Ảnh hưởng nếu xảy ra | Xác suất | Hành động | Chủ | Hạn |
   |---|--------|----------------------|----------|-----------|-----|-----|
   | R1| UAT env chậm tiếp | Lùi mốc UAT 1-2 tuần | Cao | Dùng mock để không chặn dev; cần khách cấp env trước 29/07 | Khách | 29/07 |
   | R2| Chưa rõ người ký UAT | UAT không kết thúc được | Trung bình | Cần xác nhận | Khách | 31/07 |
   | R3| Dữ liệu lịch sử có 1.2% bản ghi thiếu mã kho | Sai số báo cáo đối soát | Thấp | Đã có quy tắc xử lý, chờ nghiệp vụ xác nhận | Ta | 30/07 |

5. THAY ĐỔI SCOPE ĐÃ GHI NHẬN TRONG TUẦN
   - CR-018: thêm bộ lọc theo nhà cung cấp. Ước lượng: 3 ngày công.
     Trạng thái: đã đưa vào phase này, bù bằng việc lùi CR-011 sang phase sau (đã xác nhận 22/07).
   - Tổng scope phase so với baseline: +14% (baseline 2026-05-02).

6. VIỆC TÔI ĐÃ HỨA TUẦN TRƯỚC VÀ TRẠNG THÁI
   - [x] Gửi bản mô tả quy tắc đối soát -> đã gửi 21/07.
   - [ ] Ước lượng cho CR-016 -> chưa xong, hạn mới 28/07. Lý do: chờ xác nhận
         quy tắc thuế từ nghiệp vụ.

7. KHÔNG CÓ GÌ KHÁC CẦN ANH/CHỊ CHÚ Ý TUẦN NÀY.
```

Ba đặc điểm thiết kế của template này, mỗi cái giải quyết một cơ chế đã nêu: mục 2 đặt lên trên cùng
(người đọc chỉ có 90 giây; đặt phần cần họ hành động trước phần tiến độ); mục 5 tồn tại để chống dạng
thất bại "scope tăng 31% mà không có tài liệu"; mục 6 công khai cả việc bạn chưa làm được, và đây là
mục xây tin cậy mạnh nhất trong toàn bộ báo cáo.

**Bước 3 — Dịch mọi đề xuất kỹ thuật sang trục giá trị của người quyết** (dùng Model 2 ở trên). Kiểm
tra: nếu bỏ hết thuật ngữ kỹ thuật khỏi đề xuất mà nó vẫn còn nội dung, bạn đã dịch xong. Nếu bỏ hết
thuật ngữ mà chỉ còn "chúng ta cần làm cho code tốt hơn", bạn chưa dịch.

**Bước 4 — Escalation có cấu trúc.** Escalation không phải hành vi tố cáo; nó là cơ chế đưa một quyết
định lên đúng tầng có quyền và có thông tin để quyết. Bốn điều kiện phải đủ trước khi escalate: (a)
bạn đã cố giải quyết ở tầng dưới và có bằng chứng; (b) vấn đề vượt quyền quyết của bạn; (c) bạn mang
theo phương án, không chỉ vấn đề; (d) bạn đã nói trước với người mà bạn escalate qua đầu — escalate mà
không nói trước là hành vi phá quan hệ, và nó làm mất khả năng hợp tác về sau.

Template escalation email cho khách ODC (tiếng Anh, gửi sau khi đã gọi điện thông báo trước):

```
Subject: [Kestrel] Schedule risk on UAT milestone (05 Aug) — decision needed by 29 Jul

Hi Tanaka-san,

Following our call this morning, I am writing to confirm the situation in writing
so that we have a shared record.

1. SITUATION
   The UAT environment on your side was planned for 14 Jul and is not available as
   of today (24 Jul), 6 working days behind plan. Our remaining work needs 9
   working days of testing against a real environment before the 05 Aug UAT gate.

2. IMPACT IF NOTHING CHANGES
   We will not be able to start UAT on 05 Aug. Estimated slip: 7 to 10 working days.
   The Phase 3 kickoff (01 Sep) would also move by the same amount.

3. WHAT WE HAVE ALREADY DONE ON OUR SIDE
   - Built a mock of the partner API so that development is not blocked (done 18 Jul).
   - Reordered the remaining work to put environment-independent items first.
   - Prepared the full UAT test set in advance (ready 29 Jul instead of 04 Aug).
   These actions recovered 4 of the 6 days. We cannot recover the remaining 2 days
   without a decision from your side.

4. OPTIONS (we recommend Option B)
   Option A — Keep the full scope, move the UAT gate to 14 Aug.
     Cost: Phase 3 kickoff moves to 08 Sep. No additional cost.
   Option B — Keep the UAT gate on 05 Aug, move the "consolidated report" feature
     to Phase 3.
     Cost: the consolidated report is available 3 weeks later. No schedule impact
     on Phase 3. Our recommendation, because the report is not used by the
     warehouse team during the first weeks of operation.
   Option C — Keep both scope and date by adding 2 engineers for 3 weeks.
     Cost: additional 30 man-days, billed as a change request. We do not recommend
     this: onboarding cost means the actual gain is about 1 week, not 2.

5. WHAT WE NEED FROM YOU
   - A decision between A, B and C by 29 Jul. After 29 Jul, Option B is no longer
     possible because we will have started the report work.
   - The UAT environment, or a firm date for it, by 29 Jul.
   - Confirmation of who signs off UAT on the business side, by 31 Jul.

6. WHAT I WILL DO
   I will send a short update every Tuesday and Friday until the UAT gate is
   closed, whether or not the situation changes.

I am available for a call at your convenience, including outside our normal hours.

Best regards,
Tuan — Tech Lead, Project Kestrel
CC: Linh (Delivery Manager), Ken-san (PMO)
```

**Bước 5 — Đóng vòng.** Sau mỗi quyết định của stakeholder, ghi lại quyết định, ngày, người quyết, và
các lựa chọn đã bị loại. Đây không phải để đổ lỗi về sau; nó là để khi kết quả xấu xảy ra, cuộc trò
chuyện là "quyết định này dựa trên thông tin gì, thông tin nào đã thiếu" thay vì "ai gây ra".

### Trade-off

| Trục | Nghiêng bên trái khi | Nghiêng bên phải khi | Cái giá |
|---|---|---|---|
| **Minh bạch hoàn toàn vs Lọc thông tin cho phù hợp tầng** | Thông tin ảnh hưởng tới quyết định của họ; rủi ro có thể hiện thực hoá | Chi tiết kỹ thuật không đổi được quyết định nào; thông tin còn quá sớm và sẽ gây hoảng loạn không cần thiết | Minh bạch mọi chi tiết: stakeholder quá tải, học cách bỏ qua báo cáo của bạn, và can thiệp vào những thứ họ không nên can thiệp. Lọc quá nhiều: khi có sự cố, họ phát hiện bạn đã biết trước, và mất tin cậy vào toàn bộ báo cáo |
| **Nói "không" với yêu cầu mới vs Nhận và ghi nhận cái giá** | Bạn có vị thế và quan hệ đủ; yêu cầu rõ ràng phá vỡ cam kết đã có | Bối cảnh ODC nơi khách quyết scope; quan hệ còn mới | Nói "không" trong bối cảnh ODC: rủi ro quan hệ thật, có thể ảnh hưởng phase sau. Nhận hết mà không ghi cái giá: đến mốc thì bạn là người chịu trách nhiệm cho một scope bạn không kiểm soát — dạng thất bại của tình huống hai |
| **Chủ động cập nhật dày vs Chỉ báo khi có việc** | Stakeholder ở ô Manage closely; giai đoạn có rủi ro cao; quan hệ chưa có nền tin cậy | Quan hệ đã lâu và có tin cậy; giai đoạn ổn định | Cập nhật dày: tốn thời gian thật (2–4 giờ/tuần), và nếu toàn tin không quan trọng thì bị bỏ qua. Báo thưa: mỗi lần bạn xuất hiện là có tin xấu, và bạn mất khả năng xây tin cậy trong giai đoạn tốt |
| **Bảo vệ team khỏi áp lực stakeholder vs Cho team tiếp xúc trực tiếp** | Áp lực đang ở mức phá vỡ khả năng làm việc; stakeholder có hành vi không phù hợp | Team cần hiểu ngữ cảnh business; muốn phát triển Senior lên vai trò rộng hơn | Bảo vệ hoàn toàn: team mất ngữ cảnh, ra quyết định kỹ thuật không tính tới business, và bạn thành điểm nghẽn duy nhất. Cho tiếp xúc không chuẩn bị: một câu nói không cân nhắc của engineer có thể tạo cam kết mà công ty phải trả |

### Real-world Scenarios

**Tình huống 1 — PO là founder và mọi thứ đều gấp (product startup Series A, 30 engineer).**

Bối cảnh thật: founder không phải người vô lý; họ đang chịu một ràng buộc mà team không thấy — runway
9 tháng và một vòng gọi vốn cần số liệu tăng trưởng. Trong hàm mục tiêu đó, "hai sprint trả nợ kỹ
thuật" là một đề xuất tiêu 15% thời gian còn lại cho một thứ không xuất hiện trong pitch deck.

Phiên bản sai:

> **Tech Lead:** Anh ơi, mình cần hai sprint để refactor tầng thanh toán. Code hiện tại coupling cao,
> không có test, mỗi lần sửa là rủi ro. Nợ kỹ thuật đang chồng lên nhiều rồi.
> **Founder:** Anh hiểu, nhưng giờ mình đang cần tính năng để lên số. Cái đó để sau nhé, khi nào rảnh
> hơn.

Không có gì để founder cân nhắc: không có con số, không có rủi ro định lượng, không có phương án nhỏ
hơn. Và cụm "khi nào rảnh hơn" là dấu hiệu chắc chắn rằng đề xuất chưa vào được hàm mục tiêu của họ.

Phiên bản đúng:

> **Tech Lead:** Em cần 15 phút về một rủi ro doanh thu, không phải về refactor. Em nói ba con số. Một:
> mỗi thay đổi ở luồng thanh toán hiện mất trung bình 6 ngày, trong khi cùng loại thay đổi ở luồng đơn
> hàng mất 2 ngày. Quý sau roadmap có bốn hạng mục thanh toán, nghĩa là mình sẽ mất thêm khoảng 16
> ngày công chỉ vì phần này khó sửa. Hai: nhánh timeout của cổng thanh toán hiện không có test nào, và
> đó là nhánh xử lý tiền thật; hai trong ba incident P1 quý trước xuất phát từ đó. Ba: campaign 20/9
> dự kiến gấp bốn lần lưu lượng bình thường; theo doanh thu giờ cao điểm tháng trước, một sự cố 40
> phút trong khung đó tương đương khoảng [số cụ thể], cộng với việc đó là ngày có mặt nhà đầu tư trong
> data room.
> **Founder:** Vậy em cần bao lâu?
> **Tech Lead:** Em không xin hai sprint. Em đưa ba mức để anh chọn. Mức tối thiểu: 4 ngày công, chỉ
> làm test cho nhánh timeout và thêm circuit breaker — cái này xử lý được rủi ro campaign, không xử lý
> được tốc độ. Mức trung: 1 sprint, thêm phần tách module thanh toán ra khỏi service đơn hàng, đưa
> thời gian thay đổi về khoảng 3 ngày. Mức đầy đủ: 2 sprint, tách hẳn service. Em khuyến nghị mức
> trung, làm trong sprint trước campaign, và em chấp nhận lùi tính năng gợi ý sản phẩm sang sau
> campaign vì tính năng đó không ảnh hưởng số liệu vòng gọi vốn.
> **Founder:** Làm mức trung. Nhưng phải xong trước 10/9.
> **Tech Lead:** Được. Em ghi lại quyết định này kèm việc mình đã loại tính năng gợi ý sản phẩm, và em
> báo tiến độ mỗi thứ Ba. Nếu tới 3/9 mà em thấy không kịp, em nói với anh ngày đó, không đợi tới 10/9.

Vì sao hoạt động: đề xuất được phát biểu bằng doanh thu và rủi ro; có nhiều mức để founder thực hiện
quyền chọn (người ta chấp nhận một lựa chọn mình tự chọn dễ hơn nhiều so với một yêu cầu); và Tech Lead
tự đề xuất phần đánh đổi thay vì bắt founder tìm ra chỗ cắt.

**Tình huống 2 — Khách ODC quyết scope, 23 yêu cầu nhỏ (khách Nhật).**

Sai lầm gốc không phải nhận yêu cầu — trong mô hình ODC, khách có quyền quyết scope và điều đó nằm
trong hợp đồng. Sai lầm là **nhận mà không định giá và không ghi nhận**, khiến rủi ro tiến độ chuyển
âm thầm từ khách sang team.

Cơ chế xử lý, dùng được từ ngày đầu và không cần vị thế đàm phán mạnh: **luôn nói "được" kèm cái giá,
và luôn ghi lại.**

> **Khách:** Chỗ này thêm một bộ lọc theo nhà cung cấp nữa nhé, cái này nhỏ thôi.
> **Sai:** "Dạ vâng, để em thêm ạ."
> **Đúng:** "Được ạ, chúng tôi làm được. Việc này chúng tôi ước lượng 3 ngày công. Hiện phase đang
> đầy, nên có hai cách: một, đưa vào phase này và lùi CR-011 sang phase sau; hai, giữ CR-011 và đưa
> việc này sang phase sau. Anh chọn cách nào chúng tôi làm theo, và chiều nay tôi gửi email ghi lại
> lựa chọn đó."

Không có chữ "không" nào trong câu trả lời đúng, và vẫn bảo vệ được tiến độ. Điểm then chốt là biến
mọi yêu cầu thành một **lựa chọn có chi phí hiển thị**, và đưa quyền chọn về đúng người có quyền. Sau
sáu tháng, tài liệu cho thấy scope tăng 31% với 23 lựa chọn có ký nhận — và cuộc trò chuyện ở mốc phase
trở thành "chúng ta đã cùng chọn thế này", không phải "team các anh yếu".

**Tình huống 3 — Phòng nghiệp vụ không dùng hệ thống mới (doanh nghiệp truyền thống).**

Chẩn đoán sai: "họ kháng cự thay đổi, cần truyền thông và đào tạo thêm". Kết quả của chẩn đoán sai:
thêm ba buổi đào tạo, tỉ lệ dùng thật không đổi.

Chẩn đoán đúng bắt đầu bằng một câu hỏi: **nếu hệ thống mới chạy đúng, ai mất gì?** Sau ba buổi ngồi
tại kho quan sát (không họp), team IT tìm ra ba điều: (1) hệ thống mới không hỗ trợ trường hợp hàng về
thiếu chứng từ — chiếm khoảng 15% lượt nhập và là trường hợp phòng nghiệp vụ xử lý bằng một quy ước
không có trong tài liệu nào; (2) báo cáo cuối ngày mà phòng phải gửi lên ban giám đốc chưa có trong hệ
thống mới, nên họ vẫn phải làm bằng Excel dù đã nhập vào hệ thống — tức hệ thống mới **cộng thêm việc**
chứ không thay việc; (3) trưởng phòng lo rằng nếu hệ thống hiển thị mọi sai lệch kiểm kê theo thời gian
thực thì mọi sai sót vận hành sẽ thành số liệu công khai trước khi họ có cơ hội xử lý.

Ba can thiệp tương ứng, và không cái nào là truyền thông: hỗ trợ trường hợp thiếu chứng từ; đưa báo cáo
cuối ngày vào hệ thống để bỏ Excel; và thoả thuận rằng báo cáo sai lệch có một cửa sổ 24 giờ để phòng
xử lý trước khi vào dashboard của ban giám đốc — đây là quyết định về quy trình, không phải kỹ thuật,
và nó là điều kiện đủ để đổi được sự hợp tác. Tỉ lệ dùng thật là chỉ số duy nhất chứng minh chẩn đoán
đúng.

### Best Practices

- **Viết ra hàm mục tiêu và cách bị đo của từng stakeholder chính.** Lý do: nó chuyển "người này khó
  tính" thành "người này bị đo bằng X nên hành vi của họ dự đoán được", và biến vấn đề quan hệ thành
  vấn đề thiết kế thông điệp.
- **Bạn định nghĩa nhịp cập nhật, đừng để họ đi hỏi.** Lý do: khi stakeholder phải hỏi, họ đã lo lắng
  trước khi nghe, và bạn mất vị thế người kiểm soát dòng thông tin.
- **Báo tin xấu sớm nhất có thể, kèm nguyên nhân, phần bạn đã tự làm, và 2–3 phương án có chi phí.**
  Lý do: giá trị của tin xấu tỉ lệ với số phương án nó còn để lại; và việc nêu phần bạn đã tự làm ngăn
  cuộc trò chuyện trượt sang tìm lỗi.
- **Mọi yêu cầu phát sinh được trả lời bằng "được, và đây là cái giá / lựa chọn".** Lý do: giữ được
  quan hệ mà không âm thầm nhận rủi ro tiến độ; đồng thời tạo tài liệu bảo vệ bạn ở mốc kết thúc.
- **Đưa nhiều phương án thay vì một yêu cầu.** Lý do: người có quyền quyết chấp nhận một lựa chọn mình
  tự chọn dễ hơn nhiều so với một yêu cầu bị đưa ra, và bạn cũng học được hàm mục tiêu của họ từ cách
  họ chọn.
- **Ghi lại quyết định kèm ngày, người quyết, và các lựa chọn đã bị loại.** Lý do: khi kết quả xấu xảy
  ra, có tài liệu thì cuộc trò chuyện là về thông tin, không về con người.
- **Chuẩn bị cho Senior trước khi cho họ tiếp xúc trực tiếp với stakeholder.** Lý do: đây là cách phát
  triển người tốt nhất, nhưng một câu nói không cân nhắc ("cái đó đơn giản mà, hai ngày") có thể tạo
  cam kết mà cả team phải trả; chuẩn bị 15 phút giải quyết được điều này.
- **Với stakeholder không đọc email, đổi kênh theo họ, không theo bạn.** Lý do: kênh phải chọn theo
  hành vi thật của người nhận; gửi vào kênh họ không dùng rồi coi như đã thông báo là hình thức tự đánh
  lừa.

### Anti-patterns

- **Nói tiếng kỹ thuật với người không kỹ thuật rồi kết luận rằng họ không quan tâm chất lượng.** Cơ
  chế: bạn phát ở tần số họ không nhận được, rồi quy kết cho động cơ của họ; điều này còn làm bạn ngừng
  cố gắng. Dấu hiệu sớm: đề xuất của bạn nhận được câu "để sau khi nào rảnh".
- **Nhận mọi yêu cầu để giữ hoà khí.** Cơ chế: rủi ro tiến độ chuyển âm thầm từ người có quyền quyết
  sang người thực thi, và đến mốc thì bạn không có bằng chứng gì. Dấu hiệu sớm: bạn không nhớ tổng số
  yêu cầu phát sinh trong quý; không có tài liệu nào ghi baseline scope.
- **Che tin xấu để chờ xem có tự tốt lên không.** Cơ chế: không gian phương án co lại theo ngày, và khi
  buộc phải nói, bạn đồng thời phải giải thích tại sao đã biết mà không nói — mất hai lần. Dấu hiệu
  sớm: bạn đang dùng từ "chúng tôi đang theo dõi sát" cho một việc đã lệch 6 ngày.
- **Escalate không nói trước với người liên quan.** Cơ chế: người bị vượt qua đầu mất mặt và trở thành
  đối thủ trong mọi việc sau đó; bạn thắng một lần và mất một kênh lâu dài. Dấu hiệu sớm: bạn CC người
  cấp cao vào một email mà nội dung chưa từng được nói với người ngang cấp.
- **Coi kháng cự là vấn đề nhận thức của người kháng cự.** Cơ chế: dẫn tới giải pháp truyền thông/đào
  tạo cho một bài toán về lợi ích và rủi ro, nên can thiệp không chạm nguyên nhân. Dấu hiệu sớm: bạn đã
  tổ chức buổi đào tạo thứ ba mà tỉ lệ sử dụng không đổi.
- **Chỉ báo cáo màu xanh.** Cơ chế: mọi báo cáo của bạn bị chiết khấu, kể cả khi đúng; và khi chuyển
  sang đỏ thì stakeholder không tin mức độ mà tin rằng mọi thứ đã tệ từ lâu. Dấu hiệu sớm: dự án của
  bạn chưa từng ở trạng thái amber trong sáu tháng.
- **Làm stakeholder management thay cho làm việc.** Có dạng lead dành 60% thời gian cho báo cáo và
  quan hệ, và team thì không có ai giải quyết vấn đề kỹ thuật. Dấu hiệu sớm: bạn biết mọi ý muốn của
  khách và không biết trạng thái thật của hệ thống.

### Khi nào KHÔNG nên áp dụng

- **Khi mức formality vượt quá nhu cầu của tổ chức.** Một startup 12 người không cần stakeholder map và
  weekly status 7 mục cho một PO ngồi cách bạn hai mét. Ở quy mô đó, nhịp cập nhật là một cuộc nói
  chuyện 10 phút mỗi ngày, và việc dựng nghi thức chỉ tạo ra khoảng cách. Áp dụng bộ khung đầy đủ khi
  có từ hai stakeholder trở lên với hàm mục tiêu xung đột, hoặc khi có hợp đồng và mốc bên ngoài.
- **Khi bạn đang dùng khung này để tránh làm điều đúng về kỹ thuật.** Nếu một quyết định tạo rủi ro
  nghiêm trọng về dữ liệu khách hàng hoặc compliance, việc "dịch sang ngôn ngữ business và đưa ba lựa
  chọn" có thể trở thành cách chuyển trách nhiệm. Có một lớp vấn đề mà bạn phải nói "cái này chúng ta
  không làm" và chấp nhận hệ quả — xem [00-nen-tang-leadership.md](/series/engineering-leedership/00-nen-tang-leadership/) về ranh
  giới của Accountability.
- **Trong incident đang chạy.** Lúc đó không làm stakeholder management theo nhịp; chạy giao thức
  communication của incident: một người phát ngôn, nhịp cố định, chỉ nói fact đã xác nhận. Xem
  [06-incident-va-metrics.md](/series/engineering-leedership/06-incident-va-metrics/).
- **Khi stakeholder đó không thật sự có quyền quyết.** Đầu tư nhiều tháng vào một người được giới thiệu
  là người quyết nhưng thực tế mọi thứ do người khác chốt là dạng lãng phí phổ biến trong bán hàng nội
  bộ. Kiểm tra bằng câu hỏi: "để việc này được duyệt thì cần ai đồng ý?"
- **Khi quan hệ đã ở trạng thái không còn tin cậy tối thiểu.** Nếu phía đối tác đã quyết định rằng team
  của bạn không đủ năng lực, thêm báo cáo sẽ không đảo được kết luận đó. Lúc đó cần một hành động khác
  loại: một kết quả có thể kiểm chứng trong thời gian ngắn, hoặc một thay đổi về người đại diện.

---

## 7. Presentation và trình bày kỹ thuật cho người không kỹ thuật

### Problem Statement

Một fintech, buổi họp ban điều hành hàng tháng, slot 20 phút. Tech Lead trình bày đề xuất thay hệ thống
message queue hiện tại. Cấu trúc bài trình bày: 4 slide về kiến trúc hiện tại, 3 slide so sánh Kafka và
RabbitMQ theo throughput và độ bền, 2 slide về mô hình triển khai, 1 slide "kết luận: đề xuất chuyển
sang Kafka". Ở phút thứ 9, CFO mở laptop. Ở phút 14, CEO cắt ngang: "cái này tốn bao nhiêu và không làm
thì sao?" Tech Lead trả lời "để em xem lại rồi báo anh sau". Buổi họp kết thúc không có quyết định. Ba
tháng sau đề xuất vẫn treo, và người trình bày kết luận rằng "ban điều hành không hiểu kỹ thuật".

Đọc lại bằng mô hình truyền tin: nội dung đúng, thứ tự sai, và đơn vị đo sai. Thông tin mà người quyết
cần (chi phí, rủi ro nếu không làm, hệ quả nếu làm sai) nằm ở slide không tồn tại; thông tin họ không
dùng để quyết (so sánh throughput) chiếm 70% thời lượng. Và vì thứ tự sai, họ hết kiên nhẫn trước khi
tới phần cần nghe.

Hiện tượng đo được:

- **Tỉ lệ đề xuất kỹ thuật được duyệt.** Nếu dưới 30% trong nhiều quý ở một tổ chức bình thường, vấn đề
  nằm ở cách trình bày, không ở chất lượng đề xuất.
- **Thời điểm người nghe mở laptop.** Đo bằng mắt. Nếu nó xảy ra trước phút thứ ba, câu mở đầu của bạn
  đã không nêu được điều họ cần.
- **Câu hỏi đầu tiên của người nghe.** Nếu câu đầu tiên là "vậy cuối cùng anh cần gì" hoặc "cái này tốn
  bao nhiêu", nghĩa là thông tin đó lẽ ra phải ở slide đầu.
- **Số lần bạn phải nói "phần đó ở slide sau".** Mỗi lần là một dấu hiệu thứ tự thông tin ngược với thứ
  tự nhu cầu của người nghe.
- **Số quyết định ra được trong buổi.** Đây là chỉ số duy nhất đáng kể. Một bài trình bày rất hay mà
  không ra quyết định nào là một bài trình bày thất bại.

### First Principles

**Cơ chế một: người nghe không kỹ thuật ra quyết định trên bốn biến, và kiến trúc không nằm trong đó.**
Bốn biến: chi phí (tiền và người), rủi ro (xác suất × mức thiệt hại, gồm cả rủi ro compliance và uy
tín), thời gian (khi nào có kết quả, ảnh hưởng tới cam kết nào), và tính đảo ngược (nếu sai thì sửa
được không, tốn bao nhiêu). Kiến trúc là **phương tiện** dẫn tới bốn biến đó. Trình bày phương tiện mà
không trình bày bốn biến là yêu cầu người nghe tự làm phép dịch — mà họ không có dữ liệu để làm, nên họ
sẽ dịch bằng cách đoán, hoặc bỏ qua.

**Cơ chế hai: attention là tài nguyên suy giảm, và thứ tự thông tin quyết định phần nào được nghe.**
Người nghe cấp điều hành phân bổ attention theo mức liên quan cảm nhận được, và đánh giá đó được hình
thành trong 60–90 giây đầu. Nếu trong khoảng đó họ không xác định được "việc này liên quan gì tới tôi
và tôi cần làm gì", họ chuyển sang chế độ lắng nghe thụ động — và từ đó mọi thông tin bạn nói, kể cả
phần quan trọng nhất, đều bị mất. Đây là cơ sở của BLUF: không phải "nói ngắn cho tiện", mà là **đặt
phần quyết định vào cửa sổ mà attention còn ở mức cao nhất**.

**Cơ chế ba: thông tin không có cấu trúc thì không lưu được.** Người nghe phải giữ thông tin trong bộ
nhớ làm việc trước khi hợp nhất được thành hiểu biết. Sức chứa nhỏ, nên một chuỗi 15 dữ kiện không có
cây phân cấp sẽ mất phần lớn. Cấu trúc kim tự tháp (Minto) hoạt động vì nó cho mỗi dữ kiện một chỗ để
gắn vào: kết luận ở đỉnh, 3 lập luận đỡ kết luận, mỗi lập luận có bằng chứng. Người nghe chỉ cần giữ 3
nút, và có thể bỏ chi tiết mà vẫn giữ được cấu trúc.

**Cơ chế bốn: uy tín của người trình bày ảnh hưởng tới việc thông điệp có được xử lý hay không, và uy
tín được đánh giá bằng cách bạn xử lý phần bất định.** Điểm phản trực giác: nói "tôi không biết, tôi
sẽ trả lời trước thứ Năm" làm tăng uy tín, còn trả lời vòng vo một câu mình không biết làm giảm uy tín
nhiều hơn cả việc không biết. Cơ chế: người nghe không kỹ thuật không kiểm tra được nội dung kỹ thuật
của bạn, nên họ dùng tín hiệu thay thế — mức nhất quán, mức bạn phân biệt được cái mình biết và cái
mình đoán. Một lần bị bắt gặp nói chắc về thứ mình không chắc là đủ để mọi con số sau đó bị nghi ngờ.

**Cơ chế năm: đơn giản hoá là một phép chiếu, và mọi phép chiếu đều mất thông tin.** Khi bạn nói "hệ
thống sẽ nhanh hơn ba lần", bạn đang chiếu một không gian nhiều chiều (nhanh hơn ở đâu, dưới tải nào,
với đánh đổi gì) xuống một số. Điều này là cần thiết và không tránh được. Ranh giới nằm ở chỗ: **đơn
giản hoá được phép mất chi tiết, không được phép đảo dấu kết luận hoặc xoá điều kiện mà người nghe cần
để ra quyết định.** "Nhanh hơn ba lần ở luồng đọc, không đổi ở luồng ghi, và cần thêm một thành phần
phải vận hành" vẫn đơn giản, nhưng không lừa. Còn "cái này chỉ cần một tuần" khi bạn biết rằng một tuần
là kịch bản tốt nhất với xác suất 30% thì đó không phải đơn giản hoá, đó là chuyển rủi ro sang người
nghe mà không cho họ biết.

### Mental Model

**Model 1 — BLUF (Bottom Line Up Front).** Ba câu đầu phải chứa: kết luận/đề xuất, điều bạn cần từ
người nghe, và hạn quyết định. Mọi thứ khác là phần đỡ. Kiểm tra: nếu buổi họp bị cắt còn 90 giây, bạn
nói được đủ không?

**Model 2 — SCQA (Situation – Complication – Question – Answer).** Dùng khi cần dựng ngữ cảnh trước
cho người nghe có cache lạnh:

- *Situation:* điều cả hai bên đã đồng ý là đúng. "Hiện mỗi ngày hệ thống xử lý 400 nghìn giao dịch,
  và luồng thanh toán chạy trên hàng đợi tự viết từ 2021."
- *Complication:* điều đã thay đổi và làm trạng thái cũ không còn dùng được. "Từ tháng 4, lượng giao
  dịch giờ cao điểm tăng gấp ba; hai lần trong quý vừa rồi hàng đợi bị đầy và mất 1.100 giao dịch phải
  đối soát tay."
- *Question:* câu hỏi mà người nghe sẽ tự đặt ra. "Làm gì để không mất giao dịch trong campaign 20/9?"
- *Answer:* đề xuất của bạn, là câu trả lời trực tiếp cho Question.

Sức mạnh của SCQA là nó làm người nghe tự sinh ra câu hỏi mà bạn sắp trả lời — nên câu trả lời của bạn
được nhận như thông tin họ đang cần, không như một đề xuất bị áp.

**Model 3 — Minto Pyramid cho thân bài.** Một kết luận ở đỉnh, đúng ba lập luận đỡ (ba là con số hoạt
động tốt với bộ nhớ làm việc), mỗi lập luận có một bằng chứng định lượng. Quy tắc kiểm tra: đọc riêng
ba câu lập luận, chúng phải đủ để dẫn tới kết luận mà không cần chi tiết.

**Model 4 — Ba lớp người nghe trong cùng một phòng.** Hầu như mọi buổi trình bày có ba loại: người
quyết (cần bốn biến ở First Principles), người bị ảnh hưởng (cần biết việc của họ đổi thế nào), và
người kiểm tra tính đúng đắn kỹ thuật (cần chi tiết). Không thể phục vụ cả ba trong 20 phút. Cấu trúc
đúng: **thân bài phục vụ người quyết; chi tiết kỹ thuật đưa vào appendix và vào doc gửi trước; phần ảnh
hưởng tới công việc của từng bên gói lại một slide.** Cố nhồi cả ba vào thân bài là nguyên nhân phổ
biến nhất của một bài trình bày thất bại.

### Practical Framework

**Giai đoạn chuẩn bị (tỉ lệ thời gian: 60% cho cấu trúc, 30% cho dữ liệu, 10% cho slide):**

1. **Viết một câu duy nhất: "sau buổi này, [ai] cần [làm gì] trước [khi nào]".** Không viết được câu
   này thì đừng đặt lịch họp — bạn đang cần gửi một văn bản, không cần một buổi trình bày.
2. **Xác định người quyết thật và bốn biến của họ.** Điền số vào cả bốn: chi phí, rủi ro nếu không làm,
   thời gian, tính đảo ngược. Ô nào không điền được là ô bạn sẽ bị hỏi và không trả lời được.
3. **Viết ba câu lập luận.** Mỗi câu là một phát biểu có thể đúng hoặc sai, kèm một con số.
4. **Viết trước phần "nếu không làm gì thì sao".** Đây là slide có tác động cao nhất và hay thiếu nhất.
   Người quyết luôn so sánh đề xuất của bạn với phương án mặc định là không làm gì; nếu bạn không mô tả
   phương án đó, họ sẽ tự mô tả nó theo cách rẻ hơn thực tế.
5. **Chuẩn bị danh sách câu hỏi khó và câu trả lời.** Cách làm: nhờ một người ngoài team đọc và tấn
   công. Bảng dưới là bộ câu hỏi xuất hiện nhiều nhất trong bối cảnh Việt Nam.

| Câu hỏi | Điều họ thật sự đang hỏi | Cách trả lời |
|---|---|---|
| "Cái này tốn bao nhiêu?" | Tôi có phải xin thêm ngân sách không, và tôi giải thích với ai | Nêu cả ba loại: người-ngày, chi phí hạ tầng tăng thêm/tháng, và chi phí cơ hội (việc gì bị lùi) |
| "Không làm thì sao?" | Đây có phải việc bắt buộc hay là mong muốn của kỹ thuật | Mô tả phương án không làm gì bằng số: xác suất sự cố, chi phí mỗi lần, tốc độ ra tính năng |
| "Sao đến giờ mới nói?" | Có phải các anh đã che thông tin | Trả lời bằng mốc thời gian thật: biết từ khi nào, đã làm gì, cái gì đổi làm nó thành cấp bách bây giờ. Nếu thật là đã biết lâu mà chưa nói thì nhận thẳng |
| "Đối thủ họ làm thế nào?" | Tôi cần một điểm neo bên ngoài | Nếu biết thì nói; nếu không thì nói không biết và nêu điểm neo khác (chuẩn ngành, tài liệu công khai) |
| "Sao không thuê giải pháp có sẵn?" | Tôi muốn giảm rủi ro thực thi | Đã đánh giá phương án mua chưa, tiêu chí gì, vì sao loại. Nếu chưa đánh giá thì đây là lỗ hổng thật, nhận và hẹn |
| "Em bảo đảm được không?" | Tôi cần một người chịu trách nhiệm | Không hứa điều không kiểm soát được. "Em bảo đảm phần trong tầm em: X, Y. Phần phụ thuộc bên ngoài là Z, em không bảo đảm được và đây là cách em giảm rủi ro cho nó" |

6. **Chuẩn bị một câu trả lời "không biết" và tập nói nó.** Cấu trúc ba phần: thừa nhận không biết,
   nói cái mình biết ở lân cận, cam kết một mốc cụ thể. "Em chưa có số đó. Em biết số cho luồng đọc là
   X; luồng ghi em chưa đo. Em đo và gửi anh trước 10h thứ Năm." Điều gây thiệt hại lớn nhất trong
   tình huống này là đoán một con số rồi trình bày như đã đo — vì khi nó lệch, mọi con số khác trong
   bài cũng bị nghi ngờ, và bạn không có mặt để giải thích lúc họ phát hiện.

**Quy tắc làm slide (nếu có slide):**

- Một thông điệp một slide, và thông điệp đó là **tiêu đề của slide** ở dạng câu hoàn chỉnh. "Chi phí
  hạ tầng tăng 12% nhưng chi phí sự cố giảm nhiều hơn" là một tiêu đề tốt; "Chi phí" là một tiêu đề vô
  dụng. Kiểm tra: đọc riêng các tiêu đề slide theo thứ tự, chúng phải tạo thành một lập luận đầy đủ.
- Không sơ đồ kiến trúc trong thân bài trước người không kỹ thuật, trừ khi nó trực tiếp giải thích một
  trong bốn biến. Sơ đồ kiến trúc đưa vào appendix.
- Mỗi con số phải có đơn vị, mốc so sánh, và nguồn. "p99 8 giây" không nói gì; "p99 8 giây, so với 400
  ms ở ngày thường, đo từ APM ngày 20/6" thì nói được.
- Không đọc slide. Slide là phần đỡ cho lời nói, không phải bản ghi của lời nói.

**Trong buổi (cấu trúc 20 phút):**

| Phút | Nội dung |
|---|---|
| 0–2 | BLUF: đề xuất, điều cần từ họ, hạn quyết định |
| 2–5 | SCQA: bối cảnh và cái gì đã thay đổi (bằng số) |
| 5–12 | Ba lập luận, mỗi cái một bằng chứng |
| 12–15 | Phương án không làm gì, và 1–2 phương án khác đã bị loại kèm lý do |
| 15–20 | Câu hỏi. Nếu không có câu hỏi, tự đặt câu hỏi khó nhất và trả lời |

Tiêu chí biết là xong: có một quyết định, hoặc có một mốc quyết định kèm danh sách cụ thể những gì
người quyết cần thêm để quyết. Kết thúc bằng câu tóm kết do bạn nói và gửi lại bằng văn bản trong ngày.

### Trade-off

| Trục | Nghiêng bên trái khi | Nghiêng bên phải khi | Cái giá |
|---|---|---|---|
| **Đơn giản hoá vs Chính xác kỹ thuật** | Người nghe không dùng chi tiết để quyết; thời lượng ngắn; mục tiêu là một quyết định go/no-go | Có người kiểm tra tính đúng đắn trong phòng; quyết định phụ thuộc vào điều kiện biên kỹ thuật; lĩnh vực có ràng buộc compliance | Đơn giản quá: người nghe quyết dựa trên mô hình sai, và khi thực tế khác thì bạn mất uy tín — họ sẽ nói "anh bảo là đơn giản mà". Chính xác quá: mất attention, không ra được quyết định, và những người có thể hỗ trợ bạn không hiểu để hỗ trợ |
| **Trình bày trực tiếp vs Gửi văn bản đọc trước** | Cần đọc phản ứng, có tranh luận, cần thuyết phục | Nội dung phức tạp, nhiều số, nhiều người nghe ở múi giờ khác | Trực tiếp: người nghe không có thời gian tiêu hoá, quyết định theo phản xạ hoặc trì hoãn. Văn bản: không ai đọc, hoặc đọc và hiểu sai mà không có ai để hỏi |
| **Nêu mức bất định vs Nói dứt khoát** | Người nghe sẽ dùng thông tin để cam kết với bên thứ ba; văn hoá tổ chức chấp nhận nói về xác suất | Người nghe cần một hướng để ra quyết định và đã quá tải bởi các phương án | Nêu bất định quá nhiều: bị đọc là không tự tin, không có chủ kiến, và người nghe sẽ tìm người khác quyết. Nói dứt khoát: khi lệch, bạn mất uy tín cho mọi con số sau đó. Cách cân bằng: khuyến nghị dứt khoát một phương án, nêu mức bất định của con số |
| **Trả lời hết mọi câu hỏi trong phòng vs Hẹn trả lời sau** | Câu hỏi ảnh hưởng trực tiếp tới quyết định đang cần | Câu hỏi đi vào chi tiết chỉ 1–2 người quan tâm; bạn không có số | Trả lời hết: buổi kéo dài, những người khác mất attention, và bạn dễ đoán số. Hẹn quá nhiều: bị đọc là chưa chuẩn bị |

### Real-world Scenarios

**Tình huống 1 — Cùng một đề xuất, hai cách mở đầu (fintech, họp ban điều hành, 20 phút).**

Phiên bản sai:

> "Em xin trình bày về hệ thống message queue hiện tại. Như mọi người biết, hiện chúng ta đang dùng một
> hàng đợi tự viết dựa trên bảng database, được xây từ 2021. Kiến trúc hiện tại gồm ba thành phần
> chính: producer ở tầng API, một bảng queue trong Postgres, và các worker chạy poll mỗi 500 ms. Vấn đề
> của cách này là khi số lượng message tăng, việc poll gây lock contention trên bảng, và..."

Đến câu thứ tư, chưa có ai biết họ đang cần quyết gì, và từ "lock contention" đã đẩy nửa phòng ra ngoài
cuộc.

Phiên bản đúng:

> "Em cần một quyết định hôm nay về việc thay hàng đợi xử lý thanh toán, với 1 sprint công trước ngày
> 10/9. Lý do là ngày 20/9 có campaign lớn, và với hệ thống hiện tại em đánh giá khả năng cao là sẽ mất
> giao dịch trong giờ cao điểm.
>
> Bối cảnh: hệ thống hiện xử lý 400 nghìn giao dịch mỗi ngày, hàng đợi thanh toán là phần chúng ta tự
> viết năm 2021 khi lượng giao dịch bằng một phần mười hiện nay. Điều đã thay đổi: từ tháng 4, lưu
> lượng giờ cao điểm tăng gấp ba, và hai lần trong quý vừa rồi hàng đợi đầy — lần gần nhất ngày 20/6,
> mất 1.100 giao dịch, bộ phận vận hành phải đối soát tay trong hai ngày, và có 43 khiếu nại khách
> hàng.
>
> Câu hỏi: làm gì để campaign 20/9 không lặp lại chuyện đó, ở mức chi phí thấp nhất.
>
> Đề xuất của em: thay hàng đợi tự viết bằng một hàng đợi có sẵn, 1 sprint công, chi phí hạ tầng tăng
> khoảng [số]/tháng.
>
> Ba lý do. Một, về rủi ro: hiện không có cơ chế nào đảm bảo giao dịch không mất khi hàng đợi đầy; sau
> khi thay thì có. Hai, về chi phí sự cố: mỗi lần mất giao dịch tốn khoảng hai ngày công vận hành và
> tạo khiếu nại, ngoài phần doanh thu mất trong 40 phút gián đoạn. Ba, về tốc độ ra tính năng: ba tính
> năng trong roadmap quý sau đều cần xử lý bất đồng bộ, và trên nền hiện tại mỗi cái mất thêm khoảng ba
> ngày.
>
> Nếu không làm gì: campaign 20/9 dự kiến gấp bốn lần lưu lượng bình thường, tức khoảng gấp 1.3 lần
> mức đã làm hệ thống hỏng ngày 20/6. Em đánh giá xác suất sự cố ở mức cao, và ngày đó cũng là ngày
> chúng ta đang mở data room.
>
> Em cũng đã xem hai phương án khác và loại: tăng cấu hình database (chi phí thấp hơn nhưng chỉ mua
> được khoảng 1.5 lần dung lượng, không đủ cho gấp bốn); và viết lại phần poll cho tối ưu hơn (4 ngày
> công, giảm rủi ro nhưng không bỏ được nguyên nhân gốc — em để phương án này làm dự phòng nếu ban điều
> hành muốn giữ nguyên hệ thống).
>
> Em cần quyết định trước 27/8 vì sau đó không còn đủ thời gian trước campaign. Phần chi tiết kỹ thuật
> và so sánh công nghệ em để trong tài liệu đã gửi hôm qua, phần appendix."

Cùng nội dung, khác thứ tự và khác đơn vị đo. Câu hỏi "tốn bao nhiêu" và "không làm thì sao" đã được
trả lời trước khi ai kịp hỏi.

**Tình huống 2 — Bị hỏi một câu không biết trước ban điều hành.**

> **CFO:** Nếu thay hàng đợi thì chi phí vận hành năm sau tăng bao nhiêu phần trăm trên tổng chi phí hạ
> tầng?

Ba cách trả lời:

*Sai kiểu một (đoán và nói như đã đo):* "Khoảng 5% ạ." Nếu con số thật là 14%, bạn vừa mất uy tín cho
mọi con số trong bài, và mất nó vào lúc bạn không có mặt để giải thích.

*Sai kiểu hai (vòng vo):* "Cái đó thì tuỳ, còn phụ thuộc nhiều yếu tố, ví dụ như lượng message, cách
mình cấu hình cluster, rồi còn tuỳ vào việc mình chọn managed service hay tự vận hành nữa ạ..." Người
nghe nhận được: người này không biết và không dám nói.

*Đúng:* "Em chưa có con số đó, em không muốn đoán trong phòng này. Em biết phần cụ thể: chi phí hạ
tầng thêm cho cluster là khoảng [số]/tháng theo báo giá em đã lấy. Phần em chưa tính là chi phí vận
hành và giám sát. Em làm việc với anh Nam bên hạ tầng và gửi anh con số đầy đủ trước 10h thứ Năm. Nếu
con số đó vượt [ngưỡng] thì em nghĩ ta nên cân nhắc lại phương án, và em sẽ nói rõ điều đó."

Câu cuối là phần làm tăng uy tín nhiều nhất: bạn nêu trước điều kiện có thể làm đề xuất của chính bạn
bị loại. Người nghe đọc đó là bằng chứng rằng bạn đang phân tích, không đang bán.

**Tình huống 3 — Trình bày cho phòng nghiệp vụ, không cho ban điều hành.** Cùng một hệ thống, người
nghe khác thì bốn biến khác. Trưởng phòng vận hành kho không quan tâm chi phí hạ tầng; họ quan tâm ba
thứ: công việc hàng ngày của phòng đổi thế nào, ngày chuyển đổi có rơi vào cao điểm không, và nếu hệ
thống lỗi thì họ làm gì. Trình bày đúng cho họ không có slide kiến trúc, không có con số throughput, mà
có: một bảng "trước / sau" cho từng công việc hàng ngày, lịch chuyển đổi tránh cao điểm, phương án dự
phòng khi hệ thống lỗi, và tên người họ gọi khi có vấn đề kèm số điện thoại. Sai lầm phổ biến là dùng
lại bộ slide đã trình bày cho ban điều hành — cùng nội dung, sai audience, và nó bị đọc là "IT đang nói
chuyện của IT".

### Best Practices

- **Đặt kết luận, điều cần từ người nghe, và hạn quyết định vào 90 giây đầu.** Lý do: đó là cửa sổ
  attention cao nhất; phần đặt sau đó có xác suất bị mất cao.
- **Chuẩn bị và trình bày phương án "không làm gì" bằng số.** Lý do: đây là phương án mặc định mà người
  quyết luôn so sánh với; nếu bạn không mô tả nó, họ sẽ tự mô tả nó rẻ hơn thực tế.
- **Nêu 1–2 phương án bạn đã loại và lý do loại.** Lý do: nó chứng minh bạn đã phân tích chứ không chỉ
  đang bảo vệ một sở thích, và nó chặn trước câu hỏi "sao không làm cách X".
- **Tiêu đề slide là câu hoàn chỉnh chứa thông điệp.** Lý do: người nghe quét tiêu đề trước; và khi đọc
  riêng chuỗi tiêu đề, nó phải tự thành một lập luận đầy đủ cho người đọc lại slide sau ba tháng.
- **Nói "tôi không biết" theo cấu trúc ba phần: không biết – biết cái lân cận – mốc trả lời.** Lý do:
  người nghe không kiểm tra được kỹ thuật của bạn nên họ dùng sự nhất quán làm tín hiệu; phân biệt rõ
  biết và đoán là tín hiệu mạnh nhất bạn có thể phát.
- **Nêu trước điều kiện có thể làm đề xuất của bạn bị loại.** Lý do: chuyển bạn từ vị thế người bán
  sang vị thế người phân tích, và đó là vị thế được tin hơn.
- **Gửi tài liệu chi tiết trước, giữ buổi họp cho tranh luận và quyết định.** Lý do: hai kênh làm hai
  việc khác nhau; nhồi chi tiết vào kênh đồng bộ là dùng sai tài nguyên đắt nhất trong phòng.
- **Nếu không có câu hỏi nào, tự đặt câu hỏi khó nhất và trả lời.** Lý do: im lặng thường là dấu hiệu
  người nghe chưa hiểu đủ để hỏi, không phải đã đồng ý; và việc bạn tự nêu điểm yếu làm giảm khả năng
  đề xuất bị đảo sau buổi họp bởi một người không nói gì trong phòng.

### Anti-patterns

- **Cấu trúc kể chuyện theo trình tự bạn đã tìm hiểu.** "Đầu tiên em xem hệ thống hiện tại, rồi em thấy
  vấn đề, rồi em so sánh các giải pháp, cuối cùng em đề xuất..." Cơ chế: thứ tự này là thứ tự khám phá
  của bạn, không phải thứ tự nhu cầu của người nghe; nó đặt kết luận ở chỗ attention thấp nhất. Dấu
  hiệu sớm: slide cuối cùng có chữ "Kết luận" hoặc "Đề xuất".
- **Dùng thuật ngữ kỹ thuật làm bằng chứng cho mức nghiêm trọng.** "Coupling cao", "không idempotent",
  "N+1 query". Cơ chế: người nghe không giải nén được thành hệ quả, nên mức nghiêm trọng được giải mã
  bằng giọng nói của bạn, và giọng nói thì luôn bị chiết khấu. Dấu hiệu sớm: bạn thấy mình phải giải
  thích một thuật ngữ giữa bài, và sau khi giải thích thì mất mạch.
- **Slide dày chữ và đọc slide.** Cơ chế: người nghe không thể vừa đọc vừa nghe (hai kênh cạnh tranh),
  nên họ chọn một trong hai và bạn mất kiểm soát nội dung nào được tiếp nhận. Dấu hiệu sớm: bạn cần
  slide để nhớ mình định nói gì.
- **Đoán một con số dưới áp lực.** Cơ chế: một lần con số sai bị phát hiện làm toàn bộ định lượng của
  bạn bị nghi ngờ, kể cả phần đúng; và bạn không có mặt để giải thích khi họ phát hiện. Dấu hiệu sớm:
  bạn đang nói "khoảng chừng..." với một con số bạn chưa hề đo.
- **Trình bày một đề xuất khi người quyết không có trong phòng.** Cơ chế: bạn tiêu một lần cơ hội và
  thông tin sẽ tới người quyết qua một tầng nén nữa. Dấu hiệu sớm: bạn không trả lời được câu "để việc
  này được duyệt thì cần ai đồng ý".
- **Xin quá nhiều thứ trong một buổi.** Cơ chế: người quyết có ngân sách attention và ngân sách quyết
  định cho mỗi buổi; ba đề xuất trong 20 phút thường dẫn tới không quyết cái nào. Dấu hiệu sớm: câu
  BLUF của bạn có hai chữ "và".
- **Đơn giản hoá tới mức đảo dấu.** Nói "việc này không có rủi ro gì" khi ý bạn là "rủi ro thấp và đã
  có phương án". Cơ chế: khi rủi ro hiện thực hoá, bạn không mất uy tín vì rủi ro, bạn mất vì đã nói
  không có. Dấu hiệu sớm: bạn dùng từ tuyệt đối ("không có", "chắc chắn", "luôn luôn") về một hệ thống
  phân tán.

### Khi nào KHÔNG nên áp dụng

- **Khi người nghe là engineer.** Toàn bộ khung này thiết kế cho người ra quyết định không kỹ thuật. Với
  một buổi design review, bỏ BLUF kiểu điều hành và dùng cấu trúc RFC: vấn đề, ràng buộc, các phương án,
  đánh đổi, khuyến nghị — vì ở đó chi tiết kỹ thuật **là** nội dung ra quyết định, không phải phần đỡ.
- **Khi vấn đề cần thảo luận, không cần thuyết phục.** Nếu bạn chưa có kết luận và mục tiêu là thu ý
  kiến, mở đầu bằng BLUF là sai — nó neo cả phòng vào một phương án. Nói rõ đây là buổi thăm dò, đưa
  không gian phương án, và giữ ý kiến của bạn tới cuối.
- **Khi bạn không có dữ liệu.** Một bài trình bày có cấu trúc tốt và số liệu bịa là tình huống tệ nhất
  trong danh sách này: nó thuyết phục được, và hậu quả là một quyết định sai được ra với sự tự tin cao.
  Nếu chưa có dữ liệu, việc đúng là xin hai tuần đo trước, và trình bày đúng cái đó.
- **Trong buổi họp mà quyết định thật đã được ra trước đó ở nơi khác.** Đây là tình huống thường gặp
  trong doanh nghiệp lớn. Trình bày rất tốt vào một quyết định đã chốt là lãng phí; việc cần làm là tìm
  ra kênh mà quyết định thật diễn ra và tham gia kênh đó (xem chủ đề 6).
- **Khi văn hoá tổ chức phạt việc nói về rủi ro.** Nếu ở nơi bạn làm, người nêu rủi ro bị coi là người
  gây khó, thì việc trình bày phương án "không làm gì" bằng số có thể gây hại cho bạn về ngắn hạn. Điều
  này không có nghĩa là không nói — nghĩa là chọn kênh khác (1-1 với người quyết trước buổi họp) và
  chọn cách phát biểu khác. Vấn đề gốc là vấn đề tổ chức, xem
  [12-anti-patterns.md](/series/engineering-leedership/12-anti-patterns/).

---

## 8. Technical Writing

### Problem Statement

Một e-commerce, 60 engineer, hai sự việc cách nhau 14 tháng.

*Sự việc một.* Tháng 3/2025, team quyết định lưu trạng thái đơn hàng bằng một bảng event thay vì cập
nhật trực tiếp. Quyết định được ra trong một buổi họp 90 phút, có 5 người. Không có tài liệu nào ngoài
một dòng trong biên bản: "chốt dùng event sourcing cho order state". Tháng 5/2026, hai trong năm người
đó đã rời công ty. Một engineer mới đề xuất "đơn giản hoá bằng cách bỏ bảng event, cập nhật trực tiếp
cho nhanh". Không ai trong phòng biết ba lý do gốc: yêu cầu audit của đối tác thanh toán, việc phải tái
tạo trạng thái đơn để xử lý khiếu nại, và một sự cố mất dữ liệu năm 2024. Đề xuất được duyệt. Bốn tháng
sau, đối tác thanh toán yêu cầu audit trail và team phải làm lại — 6 tuần công, cộng với một lần đàm
phán khó với đối tác.

*Sự việc hai.* Cùng công ty, số họp mỗi tuần của một Senior Engineer: 14 buổi, 11 giờ. Khi rà lại nội
dung, 6 buổi trong đó có mục tiêu là "đồng bộ thông tin" — tức là truyền một trạng thái mà mọi người
cần biết. Không có tài liệu nào để đọc, nên cách duy nhất để biết là ngồi trong phòng. Chi phí: 11 giờ
× số người × 52 tuần, và toàn bộ thông tin đó bay đi sau buổi họp.

Hai sự việc, một nguyên nhân: tổ chức không có năng lực chuyển quyết định thành văn bản. Hiện tượng đo
được:

- **Số quyết định kiến trúc quan trọng không có tài liệu.** Kiểm tra bằng cách chọn năm quyết định lớn
  nhất 12 tháng qua và tìm tài liệu. Nếu tìm được dưới ba, tổ chức đang chạy bằng ký ức của một vài
  người.
- **Số giờ họp có mục tiêu "đồng bộ thông tin" mỗi tuần.** Đây là chỉ số trực tiếp của việc viết kém.
  Cơ chế: khi không có văn bản, kênh duy nhất còn lại là kênh đồng bộ, và nó có chi phí O(n).
- **Thời gian để một engineer mới hiểu được một hệ thống.** Nếu con số này là "phải hỏi anh Minh", bạn
  có một single point of failure về tri thức và một nút thắt về onboarding.
- **Số lần cùng một câu hỏi được hỏi trong Slack.** Mỗi lần lặp là một tài liệu đáng ra phải tồn tại.
- **Tỉ lệ design doc có mục "các phương án đã loại".** Nếu thấp, các doc đang mô tả giải pháp mà không
  ghi lại lý do — và lý do là phần duy nhất còn giá trị sau hai năm.
- **Số incident report không có mục hành động có chủ và có hạn.** Nếu cao, postmortem đang là nghi lễ.

### First Principles

**Cơ chế một: văn bản là cơ chế mở rộng quy mô của quyết định.** Một cuộc họp có chi phí O(n) người ×
thời lượng và tạo ra kết quả không bền. Một tài liệu có chi phí cố định cho người viết và chi phí gần
bằng không cho mỗi người đọc thêm, và nó còn được đọc sau khi người viết đã rời tổ chức. Về mặt kinh
tế, viết là hình thức đầu tư có lợi tức tăng theo số người và theo thời gian: cùng một nội dung, ở
team 5 người trong 3 tháng thì viết có thể không đáng; ở tổ chức 60 người trong 3 năm thì không viết là
tốn kém nghiêm trọng. Đây là lý do các tổ chức lớn hội tụ về văn hoá viết (thực hành công khai được biết
rộng rãi: Amazon dùng narrative memo 6 trang thay cho slide trong các buổi ra quyết định) — không vì họ
thích viết, mà vì ở quy mô đó kênh nói không mở rộng được.

**Cơ chế hai: viết là công cụ buộc tư duy rõ ràng, và đây là giá trị lớn hơn cả giá trị lưu trữ.** Nói
cho phép nhập nhằng: bạn có thể nói "chỗ này mình xử lý bằng cách retry" và cả phòng gật đầu, trong khi
mỗi người hình dung một cơ chế retry khác. Viết không cho phép điều đó: khi phải viết ra "retry 3 lần
với backoff 1/2/4 giây, sau 3 lần thì đưa vào dead letter queue và tạo alert", bạn buộc phải quyết định
những thứ mình đang chưa quyết. Vì vậy tỉ lệ lỗi thiết kế được phát hiện trong lúc viết doc cao hơn
nhiều so với trong lúc họp. Hệ quả thực dụng: **nếu bạn viết một design doc và không phát hiện ra điều
gì mới về thiết kế của mình, khả năng cao bạn đang viết một bản mô tả thay vì một bản thiết kế.**

**Cơ chế ba: quyết định là một hàm của thông tin và ràng buộc tại thời điểm ra quyết định, và cả hai
thứ đó bốc hơi rất nhanh.** Điều còn lại sau hai năm không phải giải pháp — giải pháp đọc được từ code.
Điều mất đi là **tập hợp các ràng buộc và các phương án đã bị loại**. Đây là thông tin không thể tái tạo
từ code, và không có nó thì người sau sẽ lặp lại một phương án đã bị loại vì một lý do vẫn còn đúng.
Sự việc một ở trên là dạng thất bại chuẩn của cơ chế này. Vì vậy phần giá trị nhất của một design doc
không phải phần "giải pháp", mà là hai phần: **ràng buộc** và **các phương án đã loại kèm lý do**.

**Cơ chế bốn: tổ chức viết kém buộc phải họp nhiều, và điều này tự củng cố.** Chuỗi nhân quả: không có
văn bản → cách duy nhất để lấy ngữ cảnh là ngồi trong phòng → mọi người bị mời vào mọi buổi họp → không
ai có thời gian sâu để viết → không có văn bản. Vòng lặp này có độ lợi dương và không tự phá; nó chỉ bị
phá bằng một can thiệp có chủ đích, thường là một quy tắc dạng "quyết định loại X phải có doc trước, và
buổi họp bắt đầu bằng 10 phút đọc im lặng".

**Cơ chế năm: văn bản không có chủ và không có ngày là văn bản không dùng được.** Người đọc cần trả lời
hai câu trước khi tin nội dung: cái này còn đúng không, và hỏi ai nếu không rõ. Không có ngày thì không
đánh giá được độ tươi; không có người chịu trách nhiệm thì không có ai cập nhật, và một tài liệu sai còn
tệ hơn không có tài liệu — vì nó tạo ra sự tự tin sai. Đây là lý do mọi wiki công ty sau ba năm đều có
một tầng tài liệu chết mà không ai dám xoá.

### Mental Model

**Model 1 — Ba loại người đọc, ba nhu cầu, một tài liệu.** Mỗi tài liệu kỹ thuật có ba loại người đọc,
và cấu trúc phải phục vụ cả ba mà không hy sinh nhóm nào:

| Người đọc | Họ đọc trong bao lâu | Họ cần gì | Phần tài liệu phục vụ họ |
|---|---|---|---|
| **Người quyết** (EM, PO, Staff+, khách) | 2–5 phút, thường chỉ đọc phần đầu | Vấn đề, khuyến nghị, chi phí, rủi ro, cái gì cần họ duyệt | Summary + Decision needed ở đầu (nửa trang) |
| **Người triển khai** (engineer làm việc này, người review) | 20–40 phút, đọc kỹ | Ràng buộc, thiết kế chi tiết, các trường hợp biên, kế hoạch triển khai và rollback | Phần thân |
| **Người đến sau 2 năm** (người debug lúc 2h sáng, người muốn thay đổi thiết kế) | 5 phút, tìm kiếm bằng từ khoá | Vì sao lại thế này, đã loại cái gì và vì sao, giả định nào có thể đã hết đúng | Ràng buộc + Phương án đã loại + Giả định + Ngày và chủ |

Kiểm tra một doc: xoá phần thân, phần còn lại có đủ cho người quyết không? Xoá phần giải pháp, phần còn
lại có đủ cho người đến sau hai năm không? Đa số doc trong thực tế chỉ phục vụ nhóm hai.

**Model 2 — Viết như thiết kế API cho bộ não người đọc.** Người đọc quét trước, đọc sau. Vì vậy cấu
trúc phải cho phép quét: tiêu đề mang thông tin, đoạn đầu của mỗi mục chứa kết luận của mục đó, danh
sách và bảng thay cho văn xuôi dài, và không có thông tin quan trọng nào bị chôn ở giữa một đoạn dài.
Một quy tắc cụ thể có tác động lớn: **mỗi mục bắt đầu bằng câu kết luận của mục đó**, phần lập luận đặt
sau.

**Model 3 — Doc như một hợp đồng có phiên bản.** Tài liệu tốt nêu rõ trạng thái của mình: đang đề xuất
(draft), đang chờ phản hồi (in review), đã chốt (accepted), đã bị thay thế (superseded by ADR-018). Điều
này giải quyết vấn đề lớn nhất của wiki: người đọc không biết tài liệu này còn hiệu lực không. Cơ chế
ADR (Architecture Decision Record) hoạt động chính vì nó bất biến — bạn không sửa một ADR đã chốt, bạn
viết một ADR mới thay thế nó, nên lịch sử lý do được bảo toàn.

### Practical Framework

**Cấu trúc design doc / RFC (dùng cho quyết định có ảnh hưởng trên một service hoặc trên một sprint
công):**

```
# RFC-024: Thay hàng đợi thanh toán tự viết bằng hàng đợi có sẵn

Trạng thái:   In review  (Draft / In review / Accepted / Rejected / Superseded by ___)
Người viết:   Tuấn (Tech Lead, team Payment)
Người duyệt:  Linh (EM), Nam (Staff Engineer)
Ngày tạo:     2026-08-14      Cập nhật gần nhất: 2026-08-19
Hạn phản hồi: 2026-08-22 (sau ngày này, không phản hồi = không phản đối)

## 1. Tóm tắt (dành cho người quyết — đọc mục này là đủ để duyệt)
Đề xuất thay hàng đợi thanh toán tự viết (bảng Postgres + poll) bằng [hàng đợi X].
Chi phí: 1 sprint (10 người-ngày) + khoảng [số]/tháng hạ tầng.
Rủi ro nếu không làm: mất giao dịch trong campaign 20/9 (đã xảy ra 2 lần trong Q2,
lần gần nhất mất 1.100 giao dịch).
Cần duyệt: ngân sách hạ tầng và việc lùi tính năng gợi ý sản phẩm sang sau campaign.

## 2. Vấn đề
Mô tả bằng hiện tượng đo được, không bằng tính từ. Kèm dữ liệu và ngày.
- 2026-06-20: hàng đợi đầy trong 41 phút, 1.100 giao dịch không được xử lý.
- Lưu lượng giờ cao điểm tăng 3x từ tháng 4/2026.
- Mỗi thay đổi ở luồng này mất trung bình 6 ngày (so với 2 ngày ở luồng đơn hàng).

## 3. Ràng buộc  (mục quan trọng nhất cho người đọc sau 2 năm)
- Không được mất giao dịch: yêu cầu của đối tác thanh toán, có trong hợp đồng mục 4.2.
- Phải giữ audit trail 24 tháng (yêu cầu compliance).
- Không được downtime quá 5 phút trong giờ hành chính.
- Team hiện có 4 BE, không có ai từng vận hành [hàng đợi X] trong production.
- Phải xong trước 10/9 (campaign 20/9).

## 4. Giả định  (ghi ra để người sau biết cái gì có thể đã hết đúng)
- Lưu lượng campaign 20/9 khoảng 4x ngày thường (dựa trên campaign 3/2026).
- Đối tác thanh toán không đổi giao thức trong 12 tháng tới.

## 5. Phương án được đề xuất
Thiết kế, sơ đồ, các trường hợp biên, cơ chế retry và dead letter, cách migrate
dữ liệu đang trong hàng đợi, kế hoạch rollback.

## 6. Các phương án đã xem xét và loại  (không được bỏ mục này)
| Phương án | Ưu | Nhược | Lý do loại |
|---|---|---|---|
| Tăng cấu hình DB | Rẻ, 0 ngày công | Chỉ mua thêm ~1.5x dung lượng | Không đủ cho 4x |
| Tối ưu vòng poll | 4 ngày công, ít rủi ro | Không bỏ được nguyên nhân gốc | Giữ làm dự phòng nếu RFC bị từ chối |
| Dùng managed service của cloud | Không phải vận hành | Chi phí cao hơn 2.3x; ràng buộc vendor | Chi phí vượt ngưỡng đã thống nhất với CFO |

## 7. Kế hoạch triển khai và rollback
Các mốc, ai làm, cách bật dần (feature flag / dual write), tiêu chí rollback,
ai được quyền quyết định rollback.

## 8. Ảnh hưởng tới các bên khác
Team nào cần đổi gì; Support cần biết gì; có cần đào tạo không.

## 9. Câu hỏi mở
Liệt kê thật, không để trống cho đẹp. Ghi rõ ai sẽ trả lời và khi nào.
```

**Cấu trúc incident report** (chi tiết về quy trình ở
[06-incident-va-metrics.md](/series/engineering-leedership/06-incident-va-metrics/); ở đây là phần viết):

```
# INC-2026-0620: Hàng đợi thanh toán đầy, 1.100 giao dịch không được xử lý

Mức độ:        P1        Thời lượng: 41 phút (20/06 20:14 – 20:55 GMT+7)
Chủ report:    Tuấn      Ngày phát hành: 2026-06-23  (trong 72h kể từ sự cố)
Trạng thái:    Đã đóng phần khôi phục; 2/4 hành động phòng ngừa còn mở

## Ảnh hưởng (viết bằng đơn vị của người dùng và business, không bằng đơn vị kỹ thuật)
1.100 giao dịch thanh toán không được xử lý trong 41 phút; 43 khiếu nại khách hàng;
2 ngày công đối soát tay của bộ phận vận hành.

## Dòng thời gian (fact, có mốc giờ, không suy diễn)
20:14  Lưu lượng tăng 3.2x so với trung bình giờ đó (bắt đầu flash sale).
20:19  Độ trễ hàng đợi vượt 60s. Không có alert nào cấu hình cho chỉ số này.
20:31  Nhân viên Support báo trong channel #support rằng khách phản ánh thanh toán treo.
20:34  On-call bắt đầu điều tra.  (20 phút từ khi vấn đề xuất hiện tới khi có người xem)
20:47  Xác định nguyên nhân: lock contention trên bảng queue.
20:52  Tăng số worker và giảm tần suất poll.  Hàng đợi bắt đầu tiêu thoát.
20:55  Độ trễ về mức bình thường.
21:40  Hoàn tất xử lý lại 1.100 giao dịch bằng script.

## Nguyên nhân (phân biệt rõ ba tầng)
- Nguyên nhân trực tiếp: lock contention khi số message vượt ~8.000.
- Nguyên nhân góp phần: không có alert cho độ trễ hàng đợi; không có kiểm thử tải
  cho luồng này.
- Nguyên nhân hệ thống: hàng đợi tự viết năm 2021 cho quy mô nhỏ hơn 10 lần,
  đã có 2 lần suýt xảy ra (2026-04-11, 2026-05-30) được ghi nhận nhưng không escalate.

## Điều đã diễn ra tốt
Script xử lý lại có sẵn từ sự cố 2024 và hoạt động đúng; không mất dữ liệu.

## Hành động  (mỗi hành động có chủ, có hạn, có link ticket — không có dòng nào là "cần chú ý hơn")
| # | Hành động | Loại | Chủ | Hạn | Ticket |
|---|---|---|---|---|---|
| 1 | Alert cho độ trễ hàng đợi > 30s | Phát hiện | Khoa | 2026-06-27 | OPS-812 |
| 2 | Kiểm thử tải luồng thanh toán ở 5x | Phòng ngừa | Nam | 2026-07-10 | PAY-455 |
| 3 | RFC thay hàng đợi | Loại bỏ nguyên nhân gốc | Tuấn | 2026-08-22 | RFC-024 |
| 4 | Quy tắc: 2 lần suýt xảy ra cùng loại → bắt buộc escalate | Quy trình | Linh | 2026-07-05 | ENG-233 |

## Ghi chú
Report này viết theo nguyên tắc Blameless: không nêu tên người gây ra thay đổi,
nêu tên người chịu trách nhiệm hành động tiếp theo.
```

**Status update** — template đã có ở chủ đề 6. Nguyên tắc viết chung cho status update: cùng một cấu
trúc mỗi lần (để người đọc học được cách quét), phần cần người đọc hành động ở trên cùng, và luôn có
mục "việc tôi đã hứa lần trước và trạng thái".

**Tiêu chí đủ tốt để merge / để chốt** — checklist dùng khi review một doc:

- [ ] Có ngày tạo, ngày cập nhật gần nhất, người viết, người duyệt, trạng thái.
- [ ] Mục vấn đề mô tả bằng hiện tượng đo được, có số và có ngày, không phải bằng tính từ.
- [ ] Có mục ràng buộc, và ràng buộc được nêu là ràng buộc thật (có nguồn: hợp đồng, compliance, năng
      lực team), không phải sở thích.
- [ ] Có ít nhất hai phương án đã bị loại kèm lý do loại.
- [ ] Có mục giả định, để người đọc sau biết cái gì cần kiểm tra lại.
- [ ] Người quyết đọc riêng nửa trang đầu là đủ để duyệt hoặc để biết cần hỏi gì.
- [ ] Có kế hoạch rollback hoặc tiêu chí dừng, nếu đây là thay đổi có rủi ro.
- [ ] Mọi hành động có chủ và có hạn. Không có dòng nào dạng "team cần lưu ý".
- [ ] Đọc riêng các tiêu đề mục theo thứ tự, chúng tạo thành một lập luận hoàn chỉnh.
- [ ] Có hạn phản hồi và quy tắc mặc định khi không ai phản hồi.

**Nhịp thực hành để tổ chức chuyển từ họp sang viết** (can thiệp cụ thể, làm được trong một quý):

1. Xác định một loại quyết định bắt buộc phải có doc: quyết định ảnh hưởng trên một service, hoặc tốn
   trên năm ngày công.
2. Đổi định dạng họp: gửi doc trước 24 giờ, buổi họp bắt đầu bằng 10 phút đọc im lặng. Cách này giải
   quyết vấn đề "không ai đọc trước" mà không cần thuyết phục ai, và nó làm phần tranh luận trong họp
   có chất lượng cao hơn nhiều.
3. Tạo quy tắc mặc định cho phản hồi: có hạn, và không phản hồi trong hạn nghĩa là không phản đối. Nếu
   không có quy tắc này, doc sẽ treo vô hạn vì không ai chịu là người chốt.
4. Đo: số quyết định lớn có doc, và số giờ họp loại "đồng bộ thông tin". Nếu số thứ hai không giảm sau
   một quý, can thiệp chưa hiệu quả.

### Trade-off

| Trục | Nghiêng bên trái khi | Nghiêng bên phải khi | Cái giá |
|---|---|---|---|
| **Viết đầy đủ vs Viết vừa đủ** | Quyết định khó đảo ngược; ảnh hưởng nhiều team; ràng buộc compliance; tổ chức lớn hoặc luân chuyển người nhiều | Quyết định đảo ngược được trong một ngày; team nhỏ với ngữ cảnh chung ấm; đang ở giai đoạn thử nghiệm | Viết đầy đủ cho việc nhỏ: chi phí thật (một RFC tốt tốn 4–8 giờ), và nếu bắt buộc cho mọi thứ thì người ta viết cho đủ thủ tục và chất lượng thông tin sụt. Viết vừa đủ cho việc lớn: lặp lại sự việc một ở đầu mục |
| **Viết trước (doc rồi mới code) vs Viết sau (code rồi mới doc)** | Bất định về thiết kế cao; nhiều bên liên quan; cần thống nhất trước khi tiêu công | Cần học bằng cách thử; vấn đề nhỏ và rõ; prototype để lấy dữ liệu | Viết trước: có thể tiêu một tuần viết về một thiết kế mà lẽ ra hai ngày prototype đã loại. Viết sau: doc trở thành bản mô tả những gì đã làm, mất chức năng buộc tư duy và mất phần "phương án đã loại" |
| **Chuẩn hoá template vs Tự do định dạng** | Nhiều người viết, cần so sánh được, nhiều người mới | Team nhỏ, người viết có kinh nghiệm, nội dung không giống nhau | Chuẩn hoá quá: người ta điền cho có, xuất hiện mục "N/A" ở những chỗ quan trọng nhất. Tự do quá: mỗi doc một kiểu, người đọc phải học lại cấu trúc mỗi lần và bỏ qua |
| **Doc chi tiết vs Doc còn được cập nhật** | Hệ thống ổn định, ít thay đổi (giao thức tích hợp, quy tắc nghiệp vụ) | Hệ thống đang đổi nhanh | Doc quá chi tiết ở phần đổi nhanh sẽ sai trong hai tháng, và một doc sai gây hại hơn không có doc. Nguyên tắc: chi tiết ở phần "vì sao" (bền), tối giản ở phần "thế nào" (dễ lệch với code) |

### Real-world Scenarios

**Tình huống 1 — Doc mô tả giải pháp mà không nêu vấn đề (ODC, khách EU).**

Một Senior gửi doc 12 trang tiêu đề "Thiết kế module đồng bộ dữ liệu". Nội dung: sơ đồ thành phần,
schema, danh sách API, thư viện sẽ dùng. Reviewer đọc và không duyệt được — không phải vì thiết kế sai,
mà vì không có cách nào đánh giá nó: không biết vấn đề là gì, không biết ràng buộc nào phải thoả, không
biết đã loại phương án nào. Câu hỏi của reviewer: "sao không dùng luôn cơ chế replication của
database?" Người viết trả lời "cái đó em có nghĩ rồi, không được vì khách không cho truy cập trực tiếp
DB". Đó là một ràng buộc quyết định toàn bộ thiết kế, và nó không có trong 12 trang.

Sửa: thêm ba mục ở đầu (Vấn đề, Ràng buộc, Phương án đã loại), tổng khoảng một trang. Sau đó doc được
duyệt trong một vòng. Bài học có tính hệ thống: **phần đắt nhất khi viết không phải phần dài nhất.** Ba
mục ngắn nhất chứa gần hết giá trị lâu dài của tài liệu.

**Tình huống 2 — Ba góc nhìn về việc "không có thời gian viết doc" (product startup, 30 engineer).**

*Nhìn từ IC:* viết doc là thời gian không sinh ra code, không hiện trong sprint, và không được tính khi
đánh giá. Nếu tuần này viết doc 6 giờ thì velocity giảm và có người sẽ hỏi. Trong hệ khuyến khích hiện
tại, quyết định không viết là quyết định hợp lý của cá nhân.

*Nhìn từ Tech Lead:* nhìn thấy chi phí ở phía khác — mỗi tuần trả 11 giờ họp để đồng bộ những thứ lẽ ra
đọc được, và mỗi lần có người nghỉ việc thì mất một tháng để tái tạo ngữ cảnh. Nhưng Tech Lead thường
chọn can thiệp sai: nhắc nhở, hoặc đưa "viết doc" vào Definition of Done và hy vọng nó tự chạy. Can
thiệp đúng: (a) đưa việc viết doc vào sprint như một hạng mục có estimate, để nó không phải là việc làm
thêm ngoài giờ; (b) giới hạn phạm vi bắt buộc — chỉ quyết định trên năm ngày công mới cần RFC, phần còn
lại một trang là đủ; (c) tự viết một hoặc hai doc mẫu, vì chuẩn được truyền bằng ví dụ nhanh hơn bằng
quy định.

*Nhìn từ Manager:* đây là vấn đề về hệ khuyến khích và về rủi ro tổ chức, không phải về kỷ luật cá
nhân. Hai can thiệp thuộc tầm của Manager: đưa năng lực viết vào Career Ladder — từ mức Senior trở lên,
"có thể viết một design doc mà người khác ra được quyết định từ đó" là tiêu chí đánh giá, không phải
điểm cộng; và đo rủi ro tri thức tập trung (bao nhiêu hệ thống chỉ một người hiểu) và coi việc giảm nó
là một mục tiêu có chủ. Nếu Manager chỉ nói "team mình cần viết nhiều hơn" mà không đổi hệ khuyến khích,
kết quả sẽ là vài doc được viết trong hai tuần rồi trở lại như cũ.

Ba góc nhìn hội tụ vào một điểm: viết doc là hành vi có chi phí cá nhân hiện hữu và lợi ích tập thể
trễ. Mọi hành vi có cấu trúc chi phí như thế chỉ tồn tại được nếu có một cơ chế bù — hoặc đưa vào định
nghĩa công việc, hoặc đưa vào tiêu chí đánh giá.

**Tình huống 3 — Doc không có ngày và không có chủ.** Một wiki nội bộ có 340 trang. Một engineer mới
làm theo hướng dẫn triển khai và làm sập môi trường staging, vì hướng dẫn đó viết cho phiên bản hạ tầng
đã bị thay 18 tháng trước. Không có ngày trên trang, không có tên ai. Sau sự cố, can thiệp không phải
"viết lại toàn bộ wiki" (bất khả thi và sẽ không xong), mà là ba quy tắc chi phí thấp: mọi trang phải
có chủ và ngày cập nhật; trang không có chủ sau một tháng rà soát thì được đánh dấu "không được bảo
trì, dùng ở mức tự chịu rủi ro" thay vì bị xoá; và các runbook được kiểm tra bằng việc dùng thật (mỗi
lần on-call dùng một runbook thì cập nhật hoặc xác nhận nó còn đúng). Quy tắc thứ ba là quy tắc duy
nhất tự duy trì được, vì nó gắn việc cập nhật vào một hành vi vốn đã xảy ra.

### Best Practices

- **Bắt đầu mọi tài liệu bằng vấn đề, không bằng giải pháp.** Lý do: người đọc không đánh giá được một
  giải pháp khi chưa biết ràng buộc, nên doc kiểu đó không dẫn tới quyết định mà dẫn tới một vòng hỏi
  đáp thừa.
- **Viết mục "các phương án đã loại và lý do" ngay cả khi bạn chỉ xem xét chúng trong 10 phút.** Lý do:
  đây là phần không tái tạo được từ code và là phần ngăn tổ chức lặp lại một sai lầm đã trả giá.
- **Ghi giả định thành một mục riêng.** Lý do: giả định là thứ hết đúng theo thời gian; nếu chúng nằm
  rải rác trong văn xuôi thì hai năm sau không ai kiểm tra lại được cái nào đã đổi.
- **Mọi doc có ngày, người viết, người duyệt, trạng thái.** Lý do: người đọc cần đánh giá độ tươi và
  cần biết hỏi ai; không có hai thứ đó thì doc không dùng được kể cả khi nội dung đúng.
- **Đặt phần cần người quyết hành động trong nửa trang đầu.** Lý do: người quyết đọc 2–5 phút; nếu phần
  cần họ duyệt nằm ở trang 8, quyết định sẽ bị trì hoãn hoặc ra bằng thông tin hành lang.
- **Có hạn phản hồi và quy tắc "không phản hồi = không phản đối".** Lý do: doc không có cơ chế chốt sẽ
  treo, và một doc treo tạo ra tình trạng tệ nhất — người triển khai không biết mình được phép làm hay
  không.
- **Mỗi mục bắt đầu bằng câu kết luận của mục đó.** Lý do: người đọc quét; nếu kết luận nằm ở câu cuối
  đoạn, phần lớn người đọc sẽ không tới đó.
- **Bắt đầu buổi họp quyết định bằng 10 phút đọc im lặng.** Lý do: giải quyết vấn đề "không ai đọc
  trước" bằng thiết kế thay vì bằng nhắc nhở, và nó làm cho việc viết doc có tác dụng thấy được ngay —
  đây là cách nhanh nhất để tạo động lực viết trong một tổ chức chưa có văn hoá viết.
- **Chi tiết ở phần "vì sao", tối giản ở phần "thế nào".** Lý do: phần "thế nào" sẽ lệch với code trong
  vài tháng và có thể đọc từ code; phần "vì sao" không có nguồn nào khác và bền theo năm.

### Anti-patterns

- **Doc mô tả giải pháp mà không nêu vấn đề và các lựa chọn đã loại.** Cơ chế: người review không có
  tiêu chí để đánh giá nên hoặc duyệt cho qua, hoặc tranh luận về sở thích; và người đến sau hai năm sẽ
  lặp lại phương án đã bị loại. Dấu hiệu sớm: câu hỏi đầu tiên của reviewer là "sao không dùng cách X".
- **Doc không có ngày và không có người chịu trách nhiệm.** Cơ chế: người đọc không đánh giá được độ
  tươi, nên hoặc bỏ qua toàn bộ wiki, hoặc làm theo một hướng dẫn đã hết đúng. Dấu hiệu sớm: có người
  hỏi trong Slack "trang này còn đúng không?".
- **Viết doc sau khi đã code xong, để hoàn thành thủ tục.** Cơ chế: mất toàn bộ chức năng buộc tư duy
  rõ ràng và mất phần phương án đã loại (vì lúc đó chỉ còn một phương án — cái đã làm). Dấu hiệu sớm:
  doc và code được commit trong cùng một PR, và doc không có mục nào về phương án khác.
- **Doc kết thúc bằng danh sách hành động không có chủ.** "Cần bổ sung test", "team nên chú ý phần
  này". Cơ chế: hành động không có chủ và không có hạn thì không xảy ra, và sự tồn tại của danh sách tạo
  cảm giác sai rằng vấn đề đã được xử lý. Dấu hiệu sớm: đọc lại incident report ba tháng trước, các hành
  động vẫn ở nguyên trạng thái.
- **Viết dài để chứng minh mức độ nghiêm túc.** Cơ chế: người quyết không đọc, nên quyết định vẫn ra
  bằng kênh khác; và người viết có cảm giác đã truyền tin. Dấu hiệu sớm: doc trên 10 trang không có tóm
  tắt nửa trang; bạn trả lời câu hỏi bằng "cái đó có trong doc rồi".
- **Dùng doc như công cụ tự bảo vệ.** Viết để sau này chỉ vào và nói "tôi đã cảnh báo". Cơ chế: mục
  tiêu chuyển từ ra quyết định tốt sang phân bổ trách nhiệm, nên nội dung được tối ưu cho việc chứng
  minh chứ không cho việc hiểu; và người đọc nhận ra điều đó. Dấu hiệu sớm: doc có nhiều câu dạng "như
  đã nêu ở trên" và nhiều lời cảnh báo không kèm phương án.
- **Copy template mà bỏ trống các mục khó.** Mục "Ràng buộc: N/A", "Phương án đã loại: chưa xem xét".
  Cơ chế: template trở thành nghi lễ, và tệ hơn, nó tạo bằng chứng giả rằng đã có quá trình phân tích.
  Dấu hiệu sớm: nhiều doc trong tổ chức có cùng các mục bị bỏ trống.

### Khi nào KHÔNG nên áp dụng

- **Khi quyết định đảo ngược được với chi phí thấp.** Chọn tên biến, chọn thư viện cho một script nội
  bộ, thử một cách tổ chức code trong một module — viết RFC cho những việc này là thuế thuần. Tiêu chí
  phân loại: chi phí đảo ngược, số người bị ảnh hưởng, và độ dài thời gian quyết định còn hiệu lực.
- **Khi bất định còn quá cao để viết có ý nghĩa.** Nếu bạn không biết đủ để nêu ràng buộc, một tuần
  viết doc sẽ là một tuần viết phỏng đoán. Việc đúng là dành hai ngày làm prototype để lấy dữ liệu, rồi
  viết. Doc viết trên dữ liệu prototype ngắn hơn và đúng hơn nhiều.
- **Trong lúc incident đang chạy.** Ghi timeline thô (một dòng mỗi hành động, có giờ) là bắt buộc, còn
  viết report thì làm sau khi đã khôi phục. Dừng lại để viết cho đẹp trong lúc hệ thống down là sai ưu
  tiên.
- **Khi tổ chức không có cơ chế đọc và cơ chế chốt.** Viết doc trong một tổ chức mà quyết định luôn được
  ra trong các cuộc gọi không ghi lại sẽ dẫn tới một thư mục doc chết và một người viết mất động lực.
  Trước khi thúc đẩy viết, phải tạo được một điểm mà doc thật sự chặn quyết định — ví dụ quy tắc "không
  có RFC thì không lên lịch buổi review".
- **Khi bạn đang dùng việc viết để trì hoãn một cuộc trò chuyện.** Có dạng lead viết một tài liệu dài về
  vấn đề hiệu suất của một người thay vì nói trực tiếp với họ. Văn bản không thay được kênh có băng
  thông cảm xúc; xem chủ đề 3 và 4.
- **Khi chi phí viết vượt giá trị vòng đời của thông tin.** Một tài liệu chi tiết về một hệ thống sẽ
  được thay trong hai tháng là chi phí không thu hồi được. Trong trường hợp đó, viết một trang về "vì
  sao" và bỏ phần "thế nào".

---

## Tự kiểm tra

Áp trực tiếp vào tổ chức bạn đang làm, trả lời bằng số hoặc bằng sự việc có ngày, không bằng cảm nhận.

1. Trong ba tháng gần nhất, có bao nhiêu ticket bị làm lại vì hiểu sai yêu cầu, và tổng bao nhiêu ngày
   công? Ở mỗi ca đó, thông điệp gốc đã đi qua mấy người và có bước xác nhận nào không?
2. Lấy rủi ro lớn nhất mà tổ chức bạn gặp trong sáu tháng qua: người đầu tiên biết nó là ai, ngày nào,
   và người có quyền xử lý biết ngày nào? Khoảng cách đó là bao nhiêu ngày, và nó dừng ở tầng nào?
3. Mở calendar ba tháng qua: tỉ lệ 1-1 đã thực hiện so với đã lên lịch là bao nhiêu? Người bạn 1-1 ít
   nhất là ai, và người đó có phải là người bạn thấy khó nói chuyện nhất không?
4. Trong buổi Performance Review gần nhất bạn thực hiện hoặc nhận, có điểm nào mà người nhận chưa từng
   nghe trước đó? Nếu có, điểm đó lẽ ra phải được nói vào ngày nào?
5. Chọn năm quyết định kiến trúc lớn nhất 12 tháng qua: bao nhiêu cái có tài liệu ghi lại ràng buộc và
   các phương án đã bị loại? Với những cái không có, nếu hôm nay có người đề xuất làm ngược lại, ai
   trong công ty còn có thể giải thích lý do gốc?
6. Đề xuất kỹ thuật gần nhất của bạn bị từ chối hoặc bị treo: bạn đã điền được cả bốn ô doanh thu, rủi
   ro, chi phí, time to market chưa? Ô nào bạn không điền được, và đó là vì thiếu dữ liệu hay vì đề
   xuất chưa đủ vững?
7. Với stakeholder khó nhất của bạn: bạn viết được ra hàm mục tiêu của họ, cách họ bị đo, và điều họ sợ
   nhất không? Nếu không, bạn đang gọi họ là "khó tính" thay vì đang hiểu họ.
8. Đếm số giờ họp mỗi tuần của bạn có mục tiêu là "đồng bộ thông tin". Với mỗi buổi đó, cái gì đang
   ngăn nó trở thành một văn bản ngắn được đọc bất đồng bộ?

## Liên kết chương khác

- [00-nen-tang-leadership.md](/series/engineering-leedership/00-nen-tang-leadership/) — Ownership, Accountability và chuỗi Business
  Goal → Execution. Mọi thông điệp trong chương này chỉ có nghĩa khi truy vết được về một mắt trong
  chuỗi đó; phần ranh giới của Accountability quyết định khi nào bạn phải nói "việc này chúng ta không
  làm" thay vì đưa ba phương án.
- [01-self-leadership.md](/series/engineering-leedership/01-self-leadership/) — Attention và năng lượng là tài nguyên mà Active
  Listening và 1-1 tiêu thụ trực tiếp. Một lead không quản được lịch của mình sẽ huỷ 1-1 trước tiên, vì
  đó là việc quan trọng mà không cấp bách.
- [03-team-leadership.md](/series/engineering-leedership/03-team-leadership/) — Psychological Safety là điều kiện tiên quyết của mọi
  kỹ thuật trong chương này. Nếu người ta không an toàn khi nói thật, kỹ thuật hỏi mở chỉ tạo ra thêm
  câu "em ổn". Phần xử lý xung đột trong team cũng nằm ở đó.
- [05-technical-leadership.md](/series/engineering-leedership/05-technical-leadership/) — RFC, ADR, Code Review và cách dựng chuẩn
  kỹ thuật. Chương này nói về cách viết và cách nói; chương đó nói về việc dùng chúng để ra quyết định
  kỹ thuật và quản lý Technical Debt.
- [06-incident-va-metrics.md](/series/engineering-leedership/06-incident-va-metrics/) — Giao thức communication trong lúc incident
  (một người phát ngôn, nhịp cố định, chỉ nói fact đã xác nhận) là ngoại lệ của phần lớn nguyên tắc
  trong chương này. Postmortem Blameless và cách viết incident report chi tiết cũng ở đó.
- [08-hiring-va-phat-trien.md](/series/engineering-leedership/08-hiring-va-phat-trien/) — Feedback và 1-1 là đầu vào của Coaching,
  Career Ladder và Performance Review. Cuộc trò chuyện về promotion ở chủ đề 4 chỉ đúng nếu tiêu chí
  Ladder đã tồn tại và đã được nói từ đầu kỳ.
- [09-to-chuc-va-scaling.md](/series/engineering-leedership/09-to-chuc-va-scaling/) — Mất mát thông tin qua tầng tổ chức, số hop, và
  giới hạn số người trực tiếp. Khi vấn đề giao tiếp là vấn đề cấu trúc, không kỹ thuật cá nhân nào bù
  được; phải giảm số hop hoặc tạo kênh skip-level.
- [10-case-studies.md](/series/engineering-leedership/10-case-studies/) — Các case study dài có chuỗi bối cảnh → lựa chọn →
  trade-off → quyết định → hậu quả, trong đó nhiều case có nguyên nhân gốc là lỗi truyền tin mô tả ở
  chủ đề 1.
- [12-anti-patterns.md](/series/engineering-leedership/12-anti-patterns/) — Tập hợp các anti-pattern ở cấp tổ chức: lead thành điểm
  nghẽn thông tin, văn hoá phạt người báo tin xấu, họp thay cho viết, và báo cáo chỉ có màu xanh.

