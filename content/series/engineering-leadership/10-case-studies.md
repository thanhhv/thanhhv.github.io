+++
title = "Case Studies — Bảy tình huống quản trị kỹ thuật phân tích sâu"
date = "2026-08-01T18:00:00+07:00"
draft = false
tags = ["engineering-leadership", "management", "career"]
series = ["Engineering Leadership & Technical Management"]
+++

# Case Studies

Một quyết định quản trị hiếm khi cho biết nó đúng hay sai trong vòng một tuần. Bạn quyết định không tách service, bốn tháng sau team vẫn chạy bình thường, và mười tháng sau thì mọi release đều phải điều phối bốn người. Bạn quyết định promote một senior lên Tech Lead, ba tháng đầu mọi thứ tốt, tháng thứ bảy thì hai người trong team nghỉ việc. Giữa hành động và kết quả có **độ trễ** dài, và trong khoảng trễ đó có hàng chục biến khác cùng thay đổi — thị trường, nhân sự, một khách hàng mới, một đợt reorg. Đó là định nghĩa kỹ thuật của một hệ thống khó học: **tín hiệu phản hồi đến muộn và bị nhiễu**.

Trong một hệ thống như vậy, kinh nghiệm cá nhân là nguồn học rất đắt và rất chậm. Một Engineering Manager làm mười năm may ra trải qua ba lần scale team, hai lần incident nghiêm trọng, một lần migration lớn. Số mẫu quá nhỏ để rút ra quy luật, nhưng lại đủ lớn để sinh ra niềm tin sai — bởi vì mỗi lần đều có một câu chuyện mạch lạc để tự kể cho mình nghe. Case study tồn tại để giải quyết đúng vấn đề đó: nó cho phép bạn xem lại **toàn bộ chuỗi quyết định trong bối cảnh đầy đủ**, bao gồm cả những thông tin mà người trong cuộc *không* có tại thời điểm quyết định, và bao gồm cả phần hậu quả xảy ra sau khi người quyết định đã rời đi.

Một lưu ý phải nói thẳng ngay từ đầu: **mọi case trong chương này là hợp thành từ nhiều tình huống có thật đã được ẩn danh và tái dựng.** Không case nào là biên bản của một công ty cụ thể. Tên người, tên sản phẩm, mốc thời gian, cấu trúc tổ chức đều đã được thay đổi và ghép lại từ nhiều nguồn để giữ đúng *cơ chế* trong khi xoá sạch phần nhận diện. Mọi con số trong chương — số ngày, tỉ lệ phần trăm, số tiền, số incident — đều là **số minh hoạ**: chúng có độ lớn hợp lý và có quan hệ nội tại đúng với nhau, nhưng không phải số đo thật của bất kỳ tổ chức nào. Nếu bạn thấy một case giống công ty mình, đó là vì cơ chế lặp lại, không phải vì đây là công ty bạn.

Mục tiêu của chương này là **phân tích cơ chế, không phải kể chuyện**. Mỗi case được viết để trả lời: cấu trúc nào tạo ra hành vi này, tín hiệu nào đã có mặt mà không được đọc, và nếu lùi lại điểm rẽ thì lựa chọn nào thực sự khả thi. Phần "Trade-off" trong mỗi case cố tình viết cân bằng — nếu bạn đọc xong bảng trade-off mà thấy phương án được chọn hiển nhiên là đúng, thì bảng đó viết dở.

**Cách đọc chương này.** Đọc tuần tự mục 1 đến mục 5 của một case, rồi **dừng lại và che phần "Quyết định và cách thực thi"**. Tự chọn một phương án và viết ra một câu: "tôi chọn X vì tôi tin rằng Y". Sau đó mới đọc tiếp. Giá trị của case study không nằm ở việc biết tổ chức kia đã làm gì — nó nằm ở khoảng cách giữa lựa chọn của bạn và lựa chọn của họ, và ở lý do tạo ra khoảng cách đó. Nếu bạn đọc thẳng một mạch từ trên xuống, bạn sẽ chỉ thu được cảm giác "hợp lý mà", tức là hindsight bias ở dạng tinh khiết nhất, và bạn sẽ không học được gì.

Mỗi case dùng cấu trúc tám phần cố định: Bối cảnh, Triệu chứng ban đầu, Chẩn đoán sai ban đầu, Các lựa chọn thực tế, Trade-off của từng lựa chọn, Quyết định và cách thực thi, Hậu quả, Bài học.

Mục lục nội bộ:

1. [Case 1 — Dự án thất bại vì giao tiếp kém](#case-1--dự-án-thất-bại-vì-giao-tiếp-kém)
2. [Case 2 — Technical Debt tích tụ nhiều năm](#case-2--technical-debt-tích-tụ-nhiều-năm)
3. [Case 3 — Chuyển từ Monolith sang Microservices](#case-3--chuyển-từ-monolith-sang-microservices)
4. [Case 4 — Scale team từ 5 lên 50 trong 18 tháng](#case-4--scale-team-từ-5-lên-50-trong-18-tháng)
5. [Case 5 — Incident production nghiêm trọng](#case-5--incident-production-nghiêm-trọng)
6. [Case 6 — Mâu thuẫn giữa Product và Engineering](#case-6--mâu-thuẫn-giữa-product-và-engineering)
7. [Case 7 — "Ship nhanh" hay "làm đúng"](#case-7--ship-nhanh-hay-làm-đúng)
8. [Bảng tra cứu case theo triệu chứng](#bảng-tra-cứu-case-theo-triệu-chứng)
9. [Cách dùng case study để huấn luyện team](#cách-dùng-case-study-để-huấn-luyện-team)

---

## Case 1 — Dự án thất bại vì giao tiếp kém

### Bối cảnh

Công ty là một ODC (Offshore Development Center) tại Hà Nội, khoảng 180 engineer, phục vụ chủ yếu khách Nhật. Mô hình doanh thu là lab-type contract: khách trả theo tháng cho một team cố định, có cam kết phạm vi theo từng phase. Đây là mô hình lai — không thuần time-and-material (khách vẫn kỳ vọng scope được giao đủ), cũng không thuần fixed-price (không có SOW chi tiết đến mức có thể tranh luận từng dòng).

Dự án: xây lại hệ thống quản lý đơn hàng nội bộ cho một nhà phân phối thiết bị công nghiệp tại Osaka. Khách hàng đã dùng hệ thống cũ mười hai năm, viết bằng một framework nội bộ, không có tài liệu. Thời lượng dự kiến 8 tháng, chia 3 phase, UAT ở cuối phase 3. Team phía Việt Nam gồm 14 người:

| Vai | Người | Ghi chú |
|---|---|---|
| Project Manager | Minh | 8 năm, quản 2 dự án song song |
| Tech Lead | Tuấn | 6 năm, mạnh backend, tiếng Anh khá, không biết tiếng Nhật |
| BrSE | Linh | N1, 3 năm làm BrSE, không có nền kỹ thuật |
| Comtor | Trang | N2, mới 1 năm, chủ yếu dịch tài liệu |
| Backend | 5 engineer | 2 senior, 3 mid |
| Frontend | 4 engineer | 1 senior (Hà), 3 mid |
| QA | 2 engineer | Khoa (lead QA), 1 junior |

Phía khách: một Project Owner tên Sato-san (bộ phận nghiệp vụ, không phải IT), một kỹ sư hệ thống nội bộ, và ba trưởng nhóm nghiệp vụ ở ba kho hàng khác nhau — những người thực sự dùng hệ thống hằng ngày nhưng không tham gia họp định kỳ.

Ràng buộc quan trọng: hợp đồng có điều khoản UAT pass là điều kiện nghiệm thu phase 3, và có penalty nếu chậm quá 30 ngày. Utilization rate của team được tính vào KPI của Minh. Ngoài ra, khách hàng làm việc theo văn hoá "yêu cầu đúng phải được hiểu, không cần nói hết" — tài liệu spec dày 140 trang nhưng phần lớn mô tả màn hình, rất ít mô tả quy tắc nghiệp vụ.

### Triệu chứng ban đầu

**Tháng 1.** Kick-off suôn sẻ. Spec được dịch sang tiếng Việt trong ba tuần. Trang dịch, Linh review. Tuấn đọc bản dịch và nói với Minh: "spec này mô tả UI kỹ lắm nhưng logic tính giá thì chỉ có một đoạn ba dòng." Minh ghi lại, đưa vào danh sách câu hỏi gửi khách. Khách trả lời sau chín ngày: *"Về cơ bản giống hệ thống hiện tại."*

**Tháng 2.** Team hỏi lại: hệ thống hiện tại tính giá thế nào? Khách gửi một file Excel 4.000 dòng là bảng giá xuất từ database. Không có công thức. Linh giải thích lại cho team: "Khách nói là cứ theo bảng này." Tuấn cho team implement lookup theo bảng.

**Tháng 3.** Trong một buổi weekly, Sato-san hỏi bằng tiếng Nhật một câu dài về xử lý đơn hàng "特別対応" (xử lý đặc biệt). Linh dịch: "Khách hỏi là có xử lý trường hợp đặc biệt không." Tuấn trả lời: "Có, mình có flag is_special trên đơn." Linh dịch sang tiếng Nhật. Sato-san gật đầu, nói "はい、わかりました". Buổi họp kết thúc sớm 10 phút. Không ai ghi minute cho câu hỏi đó.

**Tháng 4–5.** Phase 1 và 2 demo trôi. Khách xem demo qua màn hình chia sẻ, mỗi lần 45 phút, chủ yếu bấm qua các màn hình chính. Sato-san khen "画面はきれいですね" (màn hình đẹp nhỉ). Minh báo cáo lên PMO: dự án on-track, không có risk cao.

**Tháng 6.** Hà (senior frontend) nhắn riêng cho Tuấn: "Anh ơi em thấy lạ, mình làm màn hình duyệt đơn nhưng không có luồng nào cho người duyệt từ chối một phần đơn hàng. Bên mình chỉ có approve hoặc reject cả đơn." Tuấn nói: "Spec không có thì mình không làm, làm thêm là scope creep, khách không trả tiền." Hà thôi không hỏi nữa.

**Tháng 6, tuần 3.** Một backend mid-level tên Quân nói trong daily: "Bảng giá có mấy dòng trùng mã sản phẩm nhưng giá khác nhau, em không biết lấy dòng nào." Tuấn bảo Quân lấy dòng mới nhất theo cột ngày. Quân làm theo. Không ai gửi câu hỏi này cho khách vì "đã tự giải quyết được".

**Tháng 7.** Chuẩn bị UAT. Khách gửi bộ test case của họ — 320 case, viết bằng tiếng Nhật, gửi trước UAT bốn ngày. Trang dịch được 120 case trong bốn ngày đó. Team chạy thử 120 case, pass 108.

**Tháng 7, ngày UAT thứ nhất.** Khách chạy 320 case. Pass 191. Fail 129. Trong đó 71 case fail vì cùng một nguyên nhân: logic tính giá theo hợp đồng khung của từng khách hàng (契約単価) — một khái niệm chưa từng xuất hiện trong 140 trang spec. 34 case fail vì luồng duyệt từng phần đơn hàng — đúng cái Hà đã hỏi tháng 6. 24 case còn lại là lỗi lặt vặt.

**Tháng 7, ngày UAT thứ hai.** Sato-san gửi email cho account manager phía Việt Nam, cc lãnh đạo hai bên, với một câu mà Linh dịch lại là: "Chúng tôi lo ngại về mức độ hiểu nghiệp vụ của đội phát triển."

### Chẩn đoán sai ban đầu

Trong buổi họp khẩn nội bộ hai ngày sau, tổ chức thống nhất nguyên nhân: **"spec của khách quá sơ sài, khách giấu yêu cầu."**

Cần thừa nhận là chẩn đoán này *hợp lý ở thời điểm đó*, và hợp lý vì ba lý do có thật:

1. **Nó khớp với bằng chứng bề mặt.** 140 trang spec thật sự không có chữ nào về 契約単価. Nếu bạn mở file spec ra tìm, bạn sẽ không tìm thấy. Kết luận "khách không viết ra" là kết luận đúng về mặt sự kiện.
2. **Nó khớp với mô hình tinh thần sẵn có của ngành.** Ai làm ODC Nhật đủ lâu đều đã gặp cảnh khách coi nhiều thứ là hiển nhiên. "Khách Nhật hay giả định ngầm" là một schema có sẵn, và schema có sẵn luôn được ưu tiên khi giải thích dữ liệu mới.
3. **Nó bảo vệ được danh tính nghề nghiệp của mọi người trong phòng.** Nếu nguyên nhân nằm ở khách, thì không ai trong 14 người phải xem lại phán đoán của mình. Đây không phải là sự thiếu trung thực có chủ ý — đó là cách hệ thống phòng vệ nhận thức hoạt động khi chi phí của kết luận thay thế quá cao.

Chẩn đoán này dẫn tới hành động đầu tiên: Minh yêu cầu Linh lập bảng đối chiếu để chứng minh từng case fail đều nằm ngoài spec, chuẩn bị đàm phán change request. Bảng làm xong sau ba ngày. Nó đúng về mặt pháp lý và vô dụng về mặt quan hệ — vì khách không tranh cãi rằng họ đã viết ra; khách đang nói rằng họ tưởng đội phát triển sẽ hỏi.

Chẩn đoán đúng chỉ lộ ra khi Tuấn ngồi đọc lại toàn bộ minute các buổi weekly và tìm thấy câu hỏi tháng 3 về 特別対応. Anh nhờ một người khác dịch lại câu hỏi gốc từ file ghi âm. Câu hỏi đầy đủ của Sato-san là: *"Với những khách hàng có hợp đồng khung, đơn giá áp dụng khác với bảng giá chuẩn — hệ thống mới sẽ xử lý phần đó ở đâu?"* Câu trả lời "có, mình có flag is_special" đã được dịch ngược sang tiếng Nhật thành một câu đại ý "chúng tôi có hỗ trợ xử lý đặc biệt". Sato-san nghe xong, cho rằng yêu cầu đã được nắm, và gật đầu.

Vậy nguyên nhân thật không phải là "spec sơ sài". Nguyên nhân là **yêu cầu bị suy giảm qua ba tầng dịch**, và **không tầng nào có cơ chế kiểm tra ngược**:

```
Sato-san (nghiệp vụ, tiếng Nhật)
   │  mất: ngữ cảnh nghiệp vụ, giả định "ai cũng biết"
   ▼
Linh / Trang (ngôn ngữ, không có nền kỹ thuật)
   │  mất: chi tiết kỹ thuật của câu hỏi, biến câu hỏi thành câu đóng
   ▼
Tuấn (kỹ thuật, không có nền nghiệp vụ, không biết tiếng Nhật)
   │  mất: khả năng nhận ra mình đang trả lời một câu hỏi khác
   ▼
Team implement theo hiểu biết của Tuấn
```

Bốn cơ chế cụ thể cùng hoạt động:

**Cơ chế 1 — "Dạ vâng" không phải là đồng ý.** Trong cả tiếng Việt lẫn tiếng Nhật, tín hiệu nghe-hiểu và tín hiệu đồng-ý dùng chung một từ. "はい" của Sato-san nghĩa là "tôi nghe rồi". "Vâng em hiểu rồi" của một engineer trong buổi grooming nghĩa là "tôi không muốn hỏi thêm nữa". Cả hai đều bị đọc thành cam kết. Một tổ chức phân tán về ngôn ngữ mà dùng tín hiệu này làm bằng chứng đồng thuận thì đang xây trên nền không có gì cả.

**Cơ chế 2 — Người biết tin xấu không có động lực nói.** Hà biết có vấn đề từ tháng 6. Hà đã nói một lần. Phản hồi Hà nhận được là một bài giảng về scope creep. Với Hà, nói lần hai có chi phí (bị coi là người vẽ việc, làm chậm sprint của chính mình) và không có lợi ích (nếu đúng thì cũng chỉ là "em đã bảo rồi", nếu sai thì mất uy tín). Quân thậm chí không nói — Quân gặp dữ liệu mâu thuẫn và tự giải quyết, vì trong văn hoá team, "tự giải quyết được" là dấu hiệu của người giỏi. Đây là bài toán incentive kinh điển: **tín hiệu xấu chỉ đi lên khi chi phí báo tin thấp hơn chi phí giấu tin**, và ở đây tỉ lệ đó ngược.

**Cơ chế 3 — Demo kiểm tra sai thứ.** Năm buổi demo trong sáu tháng đều kiểm tra "màn hình có chạy không", không kiểm tra "kết quả tính ra có đúng không". Khách nhìn màn hình đẹp và kết luận dự án ổn. Đây là một dạng của việc chọn sai proxy metric: demo UI là proxy rẻ và dễ, nhưng nó không tương quan với thứ mà UAT sẽ đo.

**Cơ chế 4 — Bộ test của khách dựa trên giả định chưa bao giờ được viết ra.** 320 test case của khách phản ánh chính xác mô hình nghiệp vụ trong đầu ba trưởng nhóm kho — những người chưa từng dự một buổi họp nào. Bộ test đó là bản đặc tả thật của hệ thống. Nó tồn tại từ đầu, chỉ là ở phía bên kia và không ai xin. Nhìn theo cách này, thất bại không xảy ra ở tháng 7; nó xảy ra ở tháng 1, khi không ai hỏi "các anh sẽ nghiệm thu bằng cách nào".

### Các lựa chọn thực tế

Sau UAT ngày thứ hai, Minh triệu tập Tuấn, Linh, Khoa và account manager. Trên bàn có bốn phương án, và tại thời điểm đó không phương án nào hiển nhiên sai.

**Phương án A — Đàm phán change request, giữ nguyên cách làm việc.** Dùng bảng đối chiếu đã lập, chứng minh 105/129 case fail nằm ngoài spec, đề nghị khách ký CR bổ sung 8 tuần và ngân sách tương ứng. Đây là phương án duy nhất bảo vệ được biên lợi nhuận dự án và bảo vệ được KPI utilization. Về mặt hợp đồng, phía Việt Nam có cơ sở.

**Phương án B — Nhận trách nhiệm, rework bằng chi phí của mình, không đổi quy trình.** Xin lỗi, cam kết sửa trong 6 tuần, tăng cường thêm 3 người từ dự án khác, chạy overtime. Phương án này cứu quan hệ nhanh nhất và không cần đàm phán gì. Nó cũng là phương án mà đội sales muốn, vì khách này chiếm 18% doanh thu chi nhánh (số minh hoạ).

**Phương án C — Dừng dự án 2 tuần, chạy một đợt requirement discovery lại từ đầu với người dùng thật.** Bay hai người (Tuấn và Linh) sang Osaka, ngồi ba ngày ở mỗi kho hàng với ba trưởng nhóm nghiệp vụ, dựng lại mô hình nghiệp vụ, viết ra bằng cả hai thứ tiếng, rồi mới lập lại kế hoạch sửa. Chi phí: hai tuần đứng hình của 14 người, cộng chi phí đi lại. Rủi ro: khách đang giận, có thể coi việc "dừng để tìm hiểu lại" là bằng chứng đội không đủ năng lực.

**Phương án D — Đổi cấu trúc giao tiếp trước, sửa code sau.** Giữ team chạy sửa 24 lỗi lặt vặt (phần rõ ràng là lỗi của mình), đồng thời trong hai tuần đó thiết lập lại cơ chế: chỉ định một Business Analyst nói được tiếng Nhật ngồi cùng team, yêu cầu khách chỉ định một người dùng thật làm điểm liên hệ hằng ngày, chuyển toàn bộ trao đổi yêu cầu sang dạng văn bản có ví dụ số cụ thể, và yêu cầu khách gửi bộ test case ngay khi viết chứ không chờ UAT. Sau khi cơ chế chạy, mới quyết định phạm vi rework và đàm phán CR cho phần thực sự mới.

### Trade-off của từng lựa chọn

| Phương án | Được | Mất | Ai chịu phần mất | Điều kiện nên chọn |
|---|---|---|---|---|
| **A — Đàm phán CR, giữ nguyên cách làm** | Bảo vệ biên lợi nhuận và utilization; đúng về mặt hợp đồng | Quan hệ với khách xấu đi ở đúng thời điểm nhạy cảm; nguyên nhân gốc không được chạm tới nên sẽ tái phát ở phase sau | Khách, và team ở dự án tiếp theo | Khi hợp đồng là fixed-price có SOW chi tiết, và khách có văn hoá tranh luận hợp đồng bình thường |
| **B — Rework bằng chi phí của mình** | Cứu quan hệ nhanh; không tốn thời gian đàm phán; đội sales hài lòng | Team overtime 6 tuần, burnout thật; tổ chức học sai bài học ("cứ cày là qua"); vẫn không có cơ chế mới nên phase sau lặp lại | Engineer trong team | Khi lỗi thực sự thuộc về mình rõ ràng, khối lượng nhỏ, và cơ chế gây lỗi đã được sửa ở nơi khác |
| **C — Dừng 2 tuần discovery lại từ đầu** | Chạm đúng nguyên nhân gốc; xây được mô hình nghiệp vụ dùng được cho cả phase sau | 2 tuần × 14 người đứng hình (số minh hoạ: ~ 28 man-week); rủi ro khách hiểu nhầm thành thiếu năng lực; chi phí đi lại | P&L của dự án, và Minh trước PMO | Khi còn đủ thời gian trước deadline hợp đồng và khách còn đủ tin tưởng để chấp nhận một khoảng dừng |
| **D — Đổi cấu trúc giao tiếp trước, sửa sau** | Vừa có tiến độ nhìn thấy được, vừa sửa cơ chế; giữ được đòn bẩy đàm phán CR cho phần thực sự mới | Chậm hơn B trong 2 tuần đầu; đòi hỏi khách phải thay đổi (chỉ định người dùng thật) — không chắc khách đồng ý; cần tuyển/điều được BA tiếng Nhật ngay | Timeline ngắn hạn; và Minh phải chịu một cuộc nói chuyện khó với khách | Khi nguyên nhân gốc là cơ chế truyền tin chứ không phải năng lực kỹ thuật, và quan hệ còn đủ để đề nghị khách đổi cách làm việc |

Điểm đáng chú ý: A và B trông như hai cực đối lập (cứng rắn vs mềm mỏng) nhưng chúng giống nhau ở chỗ quan trọng nhất — **cả hai đều không đụng tới cơ chế đã gây ra lỗi**. Đây là cái bẫy hay gặp: khi khủng hoảng nổ ra, không gian tranh luận bị thu về trục "nhận lỗi hay không nhận lỗi", và trục "cấu trúc nào đã tạo ra chuyện này" biến mất khỏi bàn.

### Quyết định và cách thực thi

Minh chọn **D, có pha một phần B**.

Thứ tự thực thi, theo đúng trình tự đã diễn ra:

**Ngày 1–2. Tách bạch ba loại lỗi trước khi nói bất cứ điều gì với khách.** Khoa (lead QA) phân loại 129 case fail thành ba nhóm: (i) lỗi thuần của team — 24 case; (ii) yêu cầu chưa từng được truyền đạt dưới bất kỳ hình thức nào — 71 case về 契約単価; (iii) yêu cầu đã từng được nhắc nhưng bị hiểu sai hoặc bị bỏ qua — 34 case về duyệt từng phần. Việc phân loại này quan trọng vì nó cho phép nói một câu trung thực và cụ thể thay vì một câu xin lỗi chung chung.

**Ngày 3. Cuộc gọi với Sato-san, do Minh dẫn, có Tuấn ngồi cùng.** Minh không mở đầu bằng bảng đối chiếu. Anh mở đầu bằng nhóm (iii):

> **Minh:** "Sato-san, trước khi nói về kế hoạch sửa, tôi muốn báo cáo một việc chúng tôi đã làm sai. Tháng 3 anh có hỏi về đơn giá theo hợp đồng khung. Chúng tôi đã trả lời sai câu hỏi đó — chúng tôi trả lời về một tính năng khác. Chúng tôi đã nghe lại file ghi âm buổi họp và xác nhận điều này. Đây là lỗi của chúng tôi, không phải của anh."
>
> **Sato-san:** (im lặng khoảng năm giây) "...Tôi cũng nghĩ là tôi đã nói rồi."
>
> **Minh:** "Anh đã nói. Vấn đề là chúng tôi không có cách nào để phát hiện ra mình hiểu sai. Tôi muốn đề nghị thay đổi cách làm việc để chuyện này không lặp lại, trước khi bàn tới lịch sửa."

Việc thừa nhận nhóm (iii) trước — phần rõ ràng là lỗi của mình và có bằng chứng cụ thể — đã mở được cửa để bàn nhóm (ii) như một vấn đề chung, thay vì một cuộc đổ lỗi.

**Ngày 4–10. Bốn thay đổi cơ chế, triển khai song song với việc sửa 24 lỗi nhóm (i).**

1. **Một BA nói được tiếng Nhật, có nền kỹ thuật, ngồi trong team.** Công ty điều Vy từ một dự án khác — Vy là cựu developer, N2, đã làm BA hai năm. Khác biệt then chốt so với Linh: Vy có thể nghe một câu hỏi nghiệp vụ và tự đặt câu hỏi kỹ thuật tiếp theo mà không cần chuyển qua Tuấn.
2. **Khách chỉ định một người dùng thật làm điểm liên hệ hằng ngày.** Sato-san đồng ý cho một trưởng nhóm kho tham gia một buổi 30 phút mỗi ngày trong bốn tuần. Đây là thứ khó xin nhất và có giá trị cao nhất.
3. **Mọi quy tắc nghiệp vụ phải được viết dưới dạng ví dụ số cụ thể, không phải mô tả bằng lời.** Thay vì "đơn giá theo hợp đồng khung được ưu tiên", team viết: *khách hàng A, sản phẩm SKU-1201, bảng giá chuẩn 12.000, hợp đồng khung 10.800, số lượng 500 → thành tiền 5.400.000*. Mỗi quy tắc kèm ít nhất ba ví dụ, trong đó một ví dụ biên. Ví dụ số không dịch được sai.
4. **Khách gửi test case ngay khi viết, không chờ UAT.** Điều khoản này được đưa vào biên bản thoả thuận bổ sung. Đây là thay đổi có đòn bẩy lớn nhất trong bốn thay đổi: nó biến bộ tiêu chí nghiệm thu từ thông tin ẩn thành thông tin chung.

**Tuần 3–8. Rework.** Team sửa nhóm (iii) bằng chi phí của mình (34 case, khoảng 2,5 tuần). Nhóm (ii) — 71 case về 契約単価 — được đàm phán thành một CR nhỏ: khách trả 60% khối lượng, phía Việt Nam chịu 40% như phần trách nhiệm "lẽ ra phải hỏi". Tổng thời gian rework: 6 tuần.

**Điều gì suýt hỏng.** Ở tuần thứ hai, Vy phát hiện quy tắc tính giá còn phức tạp hơn nữa: có một loại chiết khấu theo sản lượng luỹ kế theo quý, chỉ tồn tại trong đầu một trưởng nhóm kho ở Nagoya và trong một macro Excel người này tự viết. Nếu phát hiện này đến sau khi rework xong, dự án sẽ fail UAT lần hai và hợp đồng gần như chắc chắn bị chấm dứt. Nó được phát hiện kịp chỉ vì có buổi 30 phút hằng ngày với người dùng thật — tức là **cơ chế mới đã cứu dự án trước khi việc sửa code kịp hoàn thành**. Đây là lập luận mạnh nhất cho việc sửa cơ chế trước khi sửa hậu quả.

Một chuyện nữa suýt hỏng theo hướng con người: ở tuần thứ tư, trong một buổi retro, một backend senior nói thẳng "nếu tháng 6 anh Tuấn nghe Hà thì mình đã không ở đây". Câu đó đúng nhưng nói ở chỗ đông người. Tuấn phản ứng phòng vệ trong 30 giây, rồi dừng lại và nói: "Đúng. Tôi đã trả lời Hà bằng một câu về scope thay vì hỏi tiếp. Từ giờ ai thấy khoảng trống trong spec thì đó là bug, không phải scope creep." Nếu Tuấn phản ứng khác đi, kênh báo tin xấu vừa mở đã đóng lại ngay.

### Hậu quả

**Phần tốt (số minh hoạ):**

- UAT lần hai sau 6 tuần: 320 case, pass 309, fail 11 — tất cả đều là lỗi nhỏ, sửa trong 3 ngày. Nghiệm thu phase 3 muộn 41 ngày so với hợp đồng gốc, nhưng nằm trong phần đã được khách chấp thuận gia hạn nên không kích hoạt penalty.
- Số câu hỏi làm rõ yêu cầu gửi cho khách tăng từ trung bình 4 câu/tuần lên 23 câu/tuần trong 8 tuần sau khi có Vy. Đây là chỉ báo sức khoẻ, không phải chỉ báo rắc rối: tần suất hỏi thấp trong một dự án cross-language gần như luôn có nghĩa là người ta đang đoán.
- Ba tháng sau, khách ký tiếp phase 4 với phạm vi lớn hơn. Lý do nêu trong email của Sato-san, Vy dịch lại: "vì lần này chúng tôi biết đội đang hiểu gì".
- Mô hình "BA kỹ thuật ngồi trong team + ví dụ số bắt buộc + test case gửi sớm" được nhân ra bốn dự án Nhật khác của chi nhánh trong năm sau.

**Phần xấu:**

- Hợp đồng phase 4 bị siết điều khoản: thêm milestone review có sự tham gia của người dùng cuối, thêm điều khoản cho phép khách giữ lại 15% giá trị phase đến sau 30 ngày vận hành. Chi phí quản trị hợp đồng tăng thật.
- Biên lợi nhuận dự án giảm từ mức mục tiêu xuống gần bằng không sau khi tính 40% phần rework tự chịu và chi phí điều Vy.
- Hai người trong team nghỉ việc trong bốn tháng sau đó. Cả hai đều nêu lý do liên quan tới giai đoạn overtime, dù phần overtime thực tế ngắn hơn nhiều so với phương án B thuần tuý.
- Minh nhận đánh giá cuối năm thấp hơn kỳ vọng, vì hệ đo lường của tổ chức tính theo biên lợi nhuận dự án và utilization, không tính "đã ngăn được một hợp đồng bị huỷ".

**Hậu quả ngoài dự kiến:**

- Linh, BrSE, xin chuyển sang dự án khác sau hai tháng. Việc đưa Vy vào — dù không ai nói ra — được Linh đọc là tín hiệu "tôi không đủ năng lực". Tổ chức đã sửa cơ chế mà quên rằng sửa cơ chế luôn phát ra tín hiệu về con người. Lẽ ra phải có một cuộc nói chuyện rõ ràng với Linh về việc vai trò BrSE và vai trò BA kỹ thuật là hai vai khác nhau, và về con đường phát triển cho Linh.
- Một hiệu ứng ngược ở dự án khác: sau khi mô hình được nhân rộng, một PM khác áp dụng máy móc quy tắc "mọi quy tắc nghiệp vụ phải có ba ví dụ số" cho một dự án maintenance nhỏ với khách đã hợp tác sáu năm. Chi phí tài liệu tăng mà không mang lại giá trị, vì ở dự án đó khoảng cách hiểu biết vốn đã rất hẹp. Một cơ chế được thiết kế để đóng một khoảng trống cụ thể sẽ thành nghi lễ khi khoảng trống đó không tồn tại.

### Bài học

**1. Trong hệ thống nhiều tầng dịch, mọi tín hiệu đồng thuận đều phải được thay bằng tín hiệu tái tạo.** Cơ chế: xác nhận bằng "vâng/はい" chỉ chứng minh tín hiệu đã đến tai, không chứng minh mô hình trong đầu hai bên giống nhau. Cách duy nhất kiểm tra được là buộc bên nghe *tái tạo lại* yêu cầu dưới dạng khác — ví dụ số, luồng màn hình, test case — rồi để bên nói xác nhận trên bản tái tạo đó. Đây là kỹ thuật cốt lõi của giao tiếp bất đối xứng, xem [02-communication.md](/series/engineering-leadership/02-communication/).

**2. Tín hiệu xấu đi lên hay không là bài toán incentive, không phải bài toán tính cách.** Cơ chế: Hà đã nói, và cái Hà nhận lại là chi phí. Sau một lần như vậy, mọi lời kêu gọi "cứ nói thẳng nhé" đều vô hiệu, vì hành vi được điều khiển bởi hệ quả gần nhất chứ không phải bởi tuyên bố. Muốn đảo chiều, người có quyền phải trả giá công khai một lần — như câu "khoảng trống trong spec là bug, không phải scope creep" của Tuấn — và phải hành xử nhất quán trong vài tháng sau đó. Xem [03-team-leadership.md](/series/engineering-leadership/03-team-leadership/) về Psychological Safety như một thuộc tính của hệ khuyến khích, và [12-anti-patterns.md](/series/engineering-leadership/12-anti-patterns/) về mẫu "shoot the messenger".

**3. Nếu bạn không biết bên kia sẽ nghiệm thu bằng gì, bạn không biết mình đang xây gì.** Cơ chế: bộ 320 test case là bản đặc tả thật, tồn tại từ tháng 1, và nó nằm ở phía không ai hỏi. Trong mọi dự án có nghiệm thu, câu hỏi "anh sẽ kiểm tra bằng cách nào" phải được đặt ở tuần đầu tiên, và tiêu chí nghiệm thu phải là tài sản chung chứ không phải tài sản của bên mua. Xem [07-project-delivery.md](/series/engineering-leadership/07-project-delivery/).

**4. Demo kiểm tra được điều rẻ nhất, không phải điều quan trọng nhất.** Cơ chế: năm buổi demo UI liên tiếp tạo ra bằng chứng giả về sức khoẻ dự án, vì proxy được chọn (màn hình chạy được) không tương quan với biến thật (logic nghiệp vụ đúng). Trong dự án có lõi nghiệp vụ phức tạp, mỗi buổi demo phải có ít nhất một phần chạy dữ liệu thật và đối chiếu kết quả với hệ thống hiện hành. Xem [06-incident-va-metrics.md](/series/engineering-leadership/06-incident-va-metrics/) về việc chọn chỉ số phản ánh đúng biến cần đo.

**5. Khi khủng hoảng nổ ra, trục tranh luận tự động thu về "nhận lỗi hay không nhận lỗi" — và đó là trục sai.** Cơ chế: cả phương án A lẫn B đều trả lời câu hỏi "ai chịu tiền", không trả lời câu hỏi "cấu trúc nào tạo ra chuyện này". Người dẫn cuộc họp có nhiệm vụ giữ trục thứ hai trên bàn, thường bằng cách ép phân loại vấn đề trước khi cho phép bàn giải pháp. Xem [04-decision-making.md](/series/engineering-leadership/04-decision-making/).

**6. Sửa cơ chế luôn phát ra tín hiệu về con người, và tín hiệu đó phải được xử lý chủ động.** Cơ chế: đưa Vy vào là quyết định đúng về mặt hệ thống và là một thông điệp không lời gửi tới Linh. Mọi thay đổi cấu trúc đều đồng thời là một thông điệp về giá trị của những người đang giữ vai trò cũ; nếu người ra quyết định không nói rõ thông điệp đó, người bị ảnh hưởng sẽ tự viết ra phiên bản tệ nhất. Xem [08-hiring-va-phat-trien.md](/series/engineering-leadership/08-hiring-va-phat-trien/) và [09-to-chuc-va-scaling.md](/series/engineering-leadership/09-to-chuc-va-scaling/).

---

## Case 2 — Technical Debt tích tụ nhiều năm

### Bối cảnh

Một công ty e-commerce Việt Nam, mô hình marketplace kết hợp bán hàng trực tiếp (1P). Sáu năm tuổi, khoảng 40 engineer chia thành sáu team, doanh thu tăng đều nhưng biên mỏng, và như mọi e-commerce Việt, đời sống của tổ chức bị chia thành hai mùa: mùa campaign (9.9, 10.10, 11.11, 12.12, Tết) và mùa còn lại.

Hệ thống là một monolith PHP dựng từ năm đầu, cộng thêm bốn service Java được tách dần trong ba năm gần đây, cộng một cụm job chạy nền viết bằng Python mà không ai nhận là chủ. Các module chính:

| Module | Tuổi | Chủ sở hữu danh nghĩa | Ghi chú |
|---|---|---|---|
| `order` | 6 năm | Team Order | Lõi monolith, ~180k dòng, không có test tích hợp |
| `pricing-promo` | 6 năm, viết lại một phần năm 3 | Team Growth | Nơi mọi campaign đi qua |
| `inventory` | 4 năm | Team Fulfillment | Service Java, tách năm 3, tương đối sạch |
| `payment` | 3 năm | Team Payment | Service Java, có test, có runbook |
| `search` | 2 năm | Team Discovery | Service mới nhất, chất lượng tốt nhất |
| `seller-portal` | 5 năm | Không rõ | Ba team cùng sửa, không ai own |

Nhân sự chính trong case: Hà là Engineering Manager phụ trách hai team (Order và Growth), lên EM được mười tháng sau bảy năm làm backend. Tuấn là Tech Lead team Order, sáu năm ở công ty — người viết phần lớn module `order` phiên bản đầu. Linh là Staff Engineer, mới về được năm tháng, đến từ một công ty lớn hơn. Trang là PO của Growth, báo cáo lên VP Product. Sơn là VP Engineering. Quân, Nam, Vy, Duy là engineer ở các team khác nhau.

Ràng buộc quan trọng cần nhớ khi đọc case: công ty không thiếu người thông minh và không có ai lười. Mọi quyết định tạo ra nợ trong sáu năm qua đều được ra bởi những người có lý do tốt tại thời điểm đó. Đây là điểm khiến Technical Debt khó nói: không có ai để chỉ tay vào.

Ba ví dụ cụ thể về những quyết định vay nợ đó, kèm lý do tại thời điểm ra quyết định:

**Năm 1 — bảng `orders` gánh thêm cột thay vì tách bảng.** Khi thêm loại đơn hàng "đặt trước", Tuấn thêm bốn cột vào `orders` thay vì tách một bảng riêng. Lý do lúc đó: công ty còn 11 người, đang chạy đua ra tính năng trước Tết, và tách bảng đồng nghĩa phải sửa 30 chỗ đang query trực tiếp. Quyết định này **đúng** ở năm 1. Sáu năm sau, bảng `orders` có 94 cột, trong đó 23 cột chỉ có nghĩa với một loại đơn cụ thể, và không có tài liệu nào nói cột nào đi với loại nào.

**Năm 3 — logic khuyến mãi được nhân bản thay vì trừu tượng hoá.** Campaign 11.11 năm đó cần một kiểu giảm giá mới. Deadline còn 9 ngày. Copy hàm cũ và sửa mất 2 ngày; trừu tượng hoá thành rule engine mất khoảng 3 tuần theo ước lượng. Team copy. Quyết định này cũng **đúng** — 3 tuần lúc đó là bỏ lỡ campaign lớn nhất năm. Sáu năm sau có 11 bản sao của hàm tính giảm giá, khác nhau ở những chi tiết nhỏ mà không ai lập bản đồ được.

**Năm 4 — không viết test cho `pricing-promo` vì "sắp viết lại".** Có một kế hoạch viết lại module này, được nhắc trong ba lần planning liên tiếp, chưa bao giờ được cấp thời gian. Trong lúc chờ, không ai viết test cho code sắp bị vứt. Kế hoạch viết lại không xảy ra. Code không có test vẫn ở đó, dày lên mỗi quý.

Đây là cấu trúc quan trọng nhất của case: **lãi kép**. Mỗi khoản vay riêng lẻ hợp lý; tổng của chúng không hợp lý; và không có thời điểm nào mà một khoản vay đơn lẻ trông đủ tệ để bị từ chối.

### Triệu chứng ban đầu

Các triệu chứng xuất hiện dần, không có mốc nào đủ kịch tính để trở thành sự kiện. Đây chính là lý do chúng không được xử lý.

**Ước lượng bắt đầu lệch một chiều.** Trong bốn quý liên tiếp, các story chạm vào `order` hoặc `pricing-promo` bị vượt ước lượng, còn story chạm vào `search` hoặc `payment` thì không. Team Order phản ứng bằng cách tăng hệ số ước lượng — một tính năng từng ước 3 điểm giờ ước 8 điểm. Việc này khiến sprint lại "đúng kế hoạch", và do đó **xoá mất tín hiệu**: burndown chart đẹp trở lại, và không ai nhìn thấy vấn đề trên dashboard nữa. Đây là một trong những cơ chế nguy hiểm nhất trong quản trị kỹ thuật — hệ thống tự điều chỉnh để che triệu chứng của chính nó.

**Số người dám sửa một số vùng code co lại.** Đến năm thứ 5, chỉ Tuấn và một người nữa dám đụng vào phần tính giá cuối trong `order`. Khi cả hai bận, mọi thay đổi liên quan phải xếp hàng. Khi Tuấn nghỉ phép Tết mười ngày, ba story bị dời sang sprint sau, và điều đó được ghi trong retro là "do nghỉ lễ" chứ không phải "do bus factor bằng một".

**Thời gian onboarding kéo dài.** Một backend mới vào team Search commit lên production trong tuần thứ hai. Một backend mới vào team Order mất khoảng chín tuần mới dám tự merge một thay đổi không tầm thường. Không ai đo con số này, nên nó chỉ tồn tại dưới dạng câu nói vui trong bữa trưa: "vào Order thì cứ xác định ba tháng đầu ngồi đọc."

**Hotfix sau mỗi campaign.** Sau 11.11 năm thứ 5, có bảy hotfix trong 48 giờ, sáu trong số đó ở `pricing-promo`. Sau 12.12, năm hotfix, bốn ở `pricing-promo`. Trong các buổi postmortem, nguyên nhân được ghi lần lượt là "sai config", "thiếu case biên", "race condition", "sai thứ tự áp dụng khuyến mãi". Bốn nguyên nhân khác nhau, cùng một nguyên nhân gốc, nhưng vì mỗi postmortem là một tài liệu riêng nên không ai nhìn thấy hình dạng chung.

**Người rời đi nói những câu giống nhau.** Trong mười tám tháng, năm engineer nghỉ. Ba người trong exit interview nói một biến thể của: "làm ở đây không thấy mình giỏi lên." Bộ phận Nhân sự tổng hợp thành "thiếu cơ hội phát triển" và đề xuất tăng ngân sách đào tạo. Không ai kết nối câu đó với việc phần lớn thời gian của họ dùng để đọc code cũ và sửa hồi quy.

**Và triệu chứng cuối cùng, cái duy nhất mà lãnh đạo nhìn thấy:** trong buổi review quý, Trang (PO) nói: "Cùng một loại tính năng khuyến mãi, hai năm trước mình làm hai tuần, giờ làm sáu tuần." Câu này được ghi vào biên bản. Nó là lần đầu tiên vấn đề được phát biểu bằng ngôn ngữ mà người ngoài engineering hiểu được — và đáng chú ý là nó đến từ Product, không phải từ Engineering.

### Chẩn đoán sai ban đầu

Chẩn đoán chính thức trong hai năm đầu: **"team Order và Growth làm chậm, cần siết quy trình và tăng năng suất."**

Chẩn đoán này hợp lý ở thời điểm đó vì bốn lý do:

1. **Nó khớp với dữ liệu duy nhất đang có.** Thứ duy nhất được đo là velocity và số story hoàn thành. Hai team này có số thấp hơn. Nếu bạn chỉ có một cái thước, mọi vấn đề đều được đọc theo cái thước đó.
2. **Engineering không đưa ra được số phản bác.** Khi Tuấn nói "code chỗ đó khó sửa lắm", anh nói thật, nhưng anh đang đưa ra một *cảm giác* để chống lại một *con số*. Trong bất kỳ cuộc tranh luận nào ở tầng lãnh đạo, cảm giác thua số — không phải vì lãnh đạo ngu ngốc, mà vì họ phải phân bổ nguồn lực giữa nhiều đề xuất và chỉ có số mới so sánh được.
3. **Nó có giải pháp rẻ.** "Siết quy trình" tốn gần như không có gì. "Refactor sáu tháng" tốn rất nhiều. Khi một chẩn đoán có giải pháp rẻ và một chẩn đoán có giải pháp đắt cùng giải thích được hiện tượng, tổ chức gần như luôn chọn cái rẻ trước.
4. **Nó có tiền lệ ngành.** Ai cũng đã nghe câu chuyện về team dùng refactor làm cớ để tránh làm tính năng. Đây là một schema có sẵn, và schema có sẵn thắng dữ liệu mới.

Hệ quả của chẩn đoán này kéo dài hai năm: thêm daily standup thứ hai vào buổi chiều, thêm bảng theo dõi story quá hạn, thêm một buổi grooming mỗi tuần. Không cái nào làm lead time giảm. Điều đó lẽ ra là bằng chứng phản bác, nhưng nó được đọc thành "chưa siết đủ chặt".

Còn một chẩn đoán sai thứ hai, tinh vi hơn, đến từ chính phía engineering: **"vấn đề là code xấu, giải pháp là viết lại cho đẹp."** Chẩn đoán này sai không phải ở chỗ code không xấu — code xấu thật — mà ở chỗ nó đặt vấn đề vào phạm trù thẩm mỹ. Một vấn đề thẩm mỹ thì không có ai ngoài engineering quan tâm, và không thể xin ngân sách cho nó. Mỗi lần Tuấn đề xuất refactor, cuộc tranh luận diễn ra gần như y hệt:

> **Tuấn:** "Module pricing giờ không maintain nổi nữa. Mình cần ba tháng viết lại."
>
> **Trang:** "Ba tháng đó mình mất bao nhiêu tính năng?"
>
> **Tuấn:** "Khoảng... sáu, bảy tính năng vừa."
>
> **Trang:** "Sau ba tháng đó khách hàng được gì?"
>
> **Tuấn:** "Code sạch hơn, sau này làm nhanh hơn."
>
> **Trang:** "Nhanh hơn bao nhiêu?"
>
> **Tuấn:** "..."

Khoảng lặng đó là toàn bộ case này. Tuấn đúng về bản chất và thua về hình thức, và anh thua vì anh mang một mệnh đề không kiểm chứng được vào một cuộc đàm phán phân bổ nguồn lực. Trang không phản đối refactor; Trang phản đối việc đánh đổi một thứ đo được lấy một thứ không đo được. Nếu ở vị trí Trang, hầu hết chúng ta cũng làm vậy.

Chẩn đoán đúng chỉ hình thành khi Hà — mới lên EM, chưa có uy tín để "xin", nên buộc phải chứng minh — bắt đầu đo. Điểm khởi đầu là một câu Linh nói trong buổi 1-1 tháng thứ hai sau khi về:

> **Linh:** "Em thấy lạ là mọi người ở đây nói về technical debt như nói về thời tiết. Ai cũng đồng ý là trời xấu, không ai có nhiệt kế."

Nguyên nhân gốc, sau khi đo, hoá ra có thể phát biểu chính xác hơn nhiều so với "code xấu": **chi phí biên của một thay đổi trong module `order` và `pricing-promo` đã tăng đến mức mà phần lớn nguồn lực engineering đang được tiêu vào việc vượt qua chính hệ thống, chứ không phải vào việc tạo ra chức năng mới.** Phát biểu này khác phát biểu "code xấu" ở một điểm sống còn: nó đo được, và nó nói bằng ngôn ngữ chi phí cơ hội — ngôn ngữ mà mọi người ngồi trong phòng họp ngân sách đều hiểu.

### Các lựa chọn thực tế

Trước khi bàn chiến lược trả nợ, Hà mất bảy tuần chỉ để dựng bốn phép đo. Phần này quan trọng nên phải kể chi tiết, vì đây là thứ tạo ra toàn bộ khả năng lựa chọn về sau.

**Phép đo 1 — Lead time theo module.** Định nghĩa: thời gian từ commit đầu tiên của một thay đổi đến khi nó chạy trên production. Dữ liệu lấy từ Git và CI, gắn nhãn module bằng đường dẫn thư mục của các file bị đổi (thay đổi chạm nhiều module thì tính cho module chiếm nhiều file nhất). Kết quả 12 tháng gần nhất (số minh hoạ):

| Module | Lead time trung vị | Lead time P90 | So với 24 tháng trước |
|---|---|---|---|
| `search` | 1,5 ngày | 4 ngày | +10% |
| `payment` | 2 ngày | 6 ngày | +20% |
| `inventory` | 3 ngày | 9 ngày | +50% |
| `order` | 9 ngày | 31 ngày | +200% |
| `pricing-promo` | 11 ngày | 38 ngày | +260% |
| `seller-portal` | 14 ngày | 45 ngày | +180% |

**Phép đo 2 — Change failure rate theo module.** Tỉ lệ deploy phải rollback hoặc phải hotfix trong 48 giờ. Số minh hoạ: `search` 3%, `payment` 4%, `inventory` 9%, `order` 22%, `pricing-promo` 31%, `seller-portal` 26%.

**Phép đo 3 — Số incident theo module trong 12 tháng.** Gắn nhãn từ hệ thống ticket, chỉ tính Sev1–Sev3. Số minh hoạ: tổng 84 incident, trong đó `pricing-promo` 29, `order` 21, `seller-portal` 14, ba module còn lại cộng lại 20. Hai module chiếm 60% incident trong khi chiếm khoảng 35% khối lượng code.

**Phép đo 4 — Thời gian onboarding vào module.** Định nghĩa vận hành: số ngày từ ngày đầu tiên của một engineer mới trong team đến pull request thứ ba được merge mà không cần sửa lại theo review về mặt logic nghiệp vụ. Đây là phép đo Hà tự nghĩ ra và là phép đo có sức thuyết phục cao nhất với lãnh đạo, vì nó quy đổi trực tiếp ra tiền lương trả cho thời gian chưa tạo ra giá trị. Số minh hoạ, trung bình trên 9 người tuyển trong 18 tháng: `search` 11 ngày, `payment` 15 ngày, `inventory` 24 ngày, `order` 58 ngày, `pricing-promo` 63 ngày.

Bước cuối là quy đổi. Hà không trình bày bảng số thô cho Sơn và ban lãnh đạo. Cô trình bày một con số duy nhất, dựng từ bảng trên:

> Trong 12 tháng qua, chênh lệch lead time giữa các module "nợ cao" và các module "nợ thấp", nhân với số thay đổi thực tế đã đi qua từng module, tương đương khoảng **9,5 người-năm** engineering (số minh hoạ). Với 40 engineer, đó là khoảng **24% tổng năng lực kỹ thuật của công ty**, đang được tiêu vào việc chống lại chính hệ thống của mình. Con số này tăng đều trong 24 tháng và chưa có dấu hiệu bão hoà.

Đây là thời điểm cuộc trò chuyện đổi bản chất. Trước đó là "engineering muốn code đẹp". Sau đó là "công ty đang mất một phần tư năng lực sản xuất mỗi năm". Cùng một sự thật, hai khả năng huy động nguồn lực hoàn toàn khác nhau.

Với dữ liệu đó, ba chiến lược trả nợ được đưa lên bàn.

**Phương án A — Big rewrite.** Dựng một hệ thống mới song song cho toàn bộ `order` + `pricing-promo`, chạy hai hệ trong sáu tháng, cắt sang một lần. Ước lượng: 8 engineer trong 9 tháng. Người ủng hộ mạnh nhất là Tuấn — hợp lý về mặt tâm lý, vì Tuấn là người hiểu rõ nhất mức độ tồi tệ của code hiện tại, và người hiểu rõ nhất luôn là người muốn xoá sạch nhất.

**Phương án B — Strangler Fig.** Dựng một lớp facade trước `pricing-promo`, tách từng nhóm quy tắc giá ra service mới sau facade đó, di chuyển traffic theo từng phần, xoá dần code cũ. Ước lượng: 4 engineer trong 12 tháng, có giá trị giao được từng phần. Linh đề xuất.

**Phương án C — Opportunistic refactor.** Không lập dự án riêng. Ra quy tắc: mọi story chạm vào module nợ cao được cộng thêm 30% thời gian để dọn phần code liên quan (boy-scout rule có ngân sách), cộng một "debt day" mỗi sprint. Không cần xin phê duyệt lớn, không cần dừng roadmap.

**Phương án D — Không làm gì, chỉ khoanh vùng.** Chấp nhận `order` và `pricing-promo` như vùng chi phí cao, ngừng thêm tính năng vào đó, đẩy mọi thứ mới ra service mới, để module cũ chết dần theo sản phẩm. Phương án này ít được nói ra nhưng nó là phương án mặc định của rất nhiều tổ chức, và trong một số điều kiện nó đúng.

### Trade-off của từng lựa chọn

| Phương án | Được | Mất | Ai chịu phần mất | Điều kiện nên chọn |
|---|---|---|---|---|
| **A — Big rewrite** | Thoát hẳn ràng buộc thiết kế cũ; kiến trúc mới không phải thoả hiệp với 6 năm giả định cũ; tinh thần team cao trong 3 tháng đầu | Không có giá trị giao được cho tới khi cắt sang; hệ cũ vẫn phải bảo trì song song nên tổng tải tăng chứ không giảm; mọi hành vi ngầm chưa có tài liệu đều là mìn; rủi ro tập trung vào một ngày cắt | Toàn công ty nếu ngày cắt hỏng; và team phải duy trì hai hệ trong 6 tháng | Khi hệ cũ nhỏ và có đặc tả rõ, hoặc khi ràng buộc mới (compliance, nền tảng cũ hết hỗ trợ) khiến hệ cũ không thể tiếp tục dù có sửa |
| **B — Strangler Fig** | Có giá trị giao từng phần, dừng được giữa chừng mà không mất trắng; rủi ro rải đều; học được về domain trong lúc làm | Phải sống với hai hệ trong thời gian dài; lớp facade là chi phí vĩnh viễn nếu bỏ dở; chậm hơn A trong kịch bản mọi thứ thuận lợi; đòi hỏi kỷ luật cao để không dừng ở trạng thái lai | Team phải chịu độ phức tạp tạm thời tăng; người vận hành phải hiểu hai đường đi | Khi hệ cũ lớn, đang chạy tiền thật, và không có đặc tả — tức là phần lớn trường hợp thực tế |
| **C — Opportunistic** | Không cần xin phê duyệt; không dừng roadmap; cải thiện chỗ nào hay đụng nhất, tức là chỗ có giá trị biên cao nhất | Không giải quyết được nợ kiến trúc (nợ cấu trúc dữ liệu, ranh giới module sai) vì những thứ đó không sửa được trong 30% của một story; dễ bị nuốt mất khi mùa campaign đến | Không ai chịu ngay, nên nó dễ được chọn — và đó chính là rủi ro | Khi nợ chủ yếu là nợ cục bộ (hàm dài, thiếu test, trùng lặp) chứ không phải nợ ranh giới; hoặc dùng làm nền cho A/B chứ không thay thế |
| **D — Khoanh vùng, để chết dần** | Chi phí gần bằng không; hợp lý nếu sản phẩm sắp thay đổi hướng | Module cũ vẫn phải bảo trì và vẫn sinh incident; "ngừng thêm tính năng" hầu như không giữ được khi kinh doanh có nhu cầu; nợ tiếp tục sinh lãi | Người trực on-call, và người mới vào team | Khi có kế hoạch dừng sản phẩm/dòng doanh thu đó trong 12–18 tháng, và điều đó đã được cam kết ở cấp công ty |

Một điểm dễ bị bỏ qua: A và B không khác nhau ở chỗ "viết lại hay không viết lại" — cả hai đều viết lại. Chúng khác nhau ở **cách rủi ro được phân bố theo thời gian**. A dồn toàn bộ rủi ro vào một điểm; B rải mỏng rủi ro và trả giá bằng độ phức tạp kéo dài. Chọn giữa hai cái này thực chất là trả lời câu hỏi: tổ chức chịu được một cú sốc lớn hay chịu được sự khó chịu kéo dài? Với một công ty e-commerce có mùa campaign không thể dời, câu trả lời gần như đã có sẵn — không tồn tại một cửa sổ đủ rộng và đủ yên tĩnh cho một ngày cắt lớn.

### Quyết định và cách thực thi

Quyết định: **B là chính (cho `pricing-promo`), C là nền (cho toàn bộ code base), D cho `seller-portal`.** Không chọn A.

**Bước 1 — Đóng khung lại vấn đề trước khi xin nguồn lực.** Hà trình bày với ban lãnh đạo không phải một đề xuất refactor mà một đề xuất khôi phục năng lực. Slide đầu tiên không có chữ "technical debt". Nó có một biểu đồ lead time theo module trong 24 tháng và một câu: "chi phí để làm một tính năng ở hai module này đang tăng 8–10% mỗi quý, và đây là đường cong lãi kép, không phải đường thẳng." Sơn hỏi câu quan trọng nhất:

> **Sơn:** "Nếu mình không làm gì, sáu tháng nữa con số này ở đâu?"
>
> **Hà:** "Nếu ngoại suy theo tốc độ 24 tháng qua, lead time trung vị của `pricing-promo` khoảng 14–15 ngày. Nhưng em nghĩ số đó là ước lượng thấp, vì trong 24 tháng qua mình còn Tuấn và Nam biết rõ module đó. Rủi ro thật là bus factor chứ không phải đường cong."
>
> **Sơn:** "Cái gì thuyết phục anh nhất trong đống số này?"
>
> **Hà:** "58 ngày onboarding vào `order`, so với 11 ngày vào `search`. Mình tuyển 9 người 18 tháng qua. Chênh lệch đó là tiền mình đã trả rồi, không thu lại được."

Ngân sách được duyệt: 4 engineer toàn thời gian trong 12 tháng cho nhánh `pricing-promo`, cộng chính sách "debt day" cho tất cả các team. Điều đáng chú ý: ngân sách được duyệt trong một buổi họp 45 phút, sau hai năm không xin được gì. Thứ thay đổi không phải là mức độ nghiêm trọng của vấn đề — nó đã nghiêm trọng từ lâu. Thứ thay đổi là vấn đề đã được chuyển từ dạng không đo được sang dạng đo được.

**Bước 2 — Chọn người và định nghĩa "xong".** Linh dẫn nhánh strangler với Vy, Duy và một engineer từ team Discovery. Tuấn cố tình *không* được đưa vào nhánh này. Lý do Hà nói riêng với Tuấn:

> **Hà:** "Anh là người duy nhất giữ được hệ cũ chạy qua mùa campaign. Nếu anh sang nhánh mới, mình mất cả hai đầu. Với lại — em nói thẳng — anh muốn viết lại toàn bộ, mà mình không chọn viết lại toàn bộ. Em cần người dẫn nhánh đó tin vào cách làm từng phần."

Câu thứ hai quan trọng hơn câu thứ nhất và khó nói hơn nhiều. Nó cũng là mầm mống của một hậu quả sẽ nói ở phần sau.

Tiêu chí "xong" được định nghĩa bằng chính bốn phép đo ban đầu, không bằng mô tả kiến trúc: lead time trung vị của các thay đổi liên quan tới quy tắc giá xuống dưới 4 ngày; change failure rate dưới 10%; một engineer mới merge được thay đổi quy tắc giá trong vòng 20 ngày. Định nghĩa xong bằng metric chứ không bằng "code sạch" là thứ giữ cho dự án không trôi.

**Bước 3 — Thứ tự tách.** Không tách theo cấu trúc code mà theo tần suất thay đổi. Linh chạy một phân tích lịch sử Git: nhóm quy tắc nào bị sửa nhiều nhất trong 24 tháng thì tách trước. Ba nhóm đầu (khuyến mãi theo campaign, mã giảm giá, freeship) chiếm khoảng 70% số lần thay đổi nhưng chỉ khoảng 30% khối lượng code. Nguyên tắc: **trả nợ ở chỗ lãi suất cao nhất, không phải ở chỗ xấu nhất.** Code xấu mà sáu tháng không ai đụng thì lãi suất bằng không.

**Bước 4 — Đo liên tục và công khai.** Bốn phép đo được đưa lên dashboard cập nhật hằng tuần, mọi người xem được, bao gồm cả Product. Đây là quyết định có rủi ro: nếu số không cải thiện, Hà không có chỗ trốn. Nhưng nó cũng là thứ giữ được sự tin tưởng khi tiến độ chậm ở quý 2.

**Bước 5 — Bảo vệ ngân sách qua mùa campaign.** Trước 11.11, có áp lực rút 4 người sang làm tính năng campaign. Hà đàm phán được một thoả hiệp: rút 2 người trong 5 tuần, đổi lại nhánh strangler được gia hạn 6 tuần và điều đó được ghi vào biên bản. Việc ghi vào biên bản quan trọng hơn con số — nó biến một khoản vay thời gian thành một khoản nợ có chủ.

### Hậu quả

**Phần tốt (số minh hoạ, sau 14 tháng):**

- `pricing-promo`: ba nhóm quy tắc lớn nhất đã ra service mới. Lead time trung vị cho thay đổi quy tắc giá: từ 11 ngày xuống 3 ngày. Change failure rate: từ 31% xuống 8%.
- Mùa campaign đầu tiên sau khi tách: 2 hotfix, so với 6–7 của hai mùa trước.
- Onboarding: một backend mới merge được thay đổi quy tắc giá sau 17 ngày.
- Chính sách "debt day" mỗi sprint tạo ra hiệu ứng phụ tích cực ngoài dự tính: số test tự động ở `order` tăng từ gần bằng không lên mức đủ để chặn ba lỗi hồi quy nghiêm trọng trong năm đó.
- Bốn phép đo trở thành một phần của review quý. Sau đó, khi team Fulfillment xin nguồn lực trả nợ cho `inventory`, họ không phải tranh luận về phương pháp — họ chỉ cần điền số vào cái khung đã có.

**Phần xấu — và phần này quan trọng hơn phần tốt:**

- **Nhánh refactor `order` bị bỏ dở.** Sau thành công ở `pricing-promo`, quý 3 công ty mở thêm một nhánh cho `order` do Tuấn dẫn, 3 người, mục tiêu tách phần vòng đời đơn hàng. Nhánh này chạy 5 tháng rồi dừng. Ba nguyên nhân, xếp theo mức độ quan trọng: (i) không định nghĩa "xong" bằng metric — mục tiêu được viết là "tách state machine của đơn hàng ra khỏi monolith", một mục tiêu không có điểm dừng tự nhiên; (ii) `order` có phụ thuộc ngược từ chín nơi khác nhau, trong đó ba nơi là job Python không ai own — phạm vi thật lớn gấp đôi ước lượng ban đầu và điều này chỉ lộ ra ở tháng thứ ba; (iii) Tuấn vẫn là người duy nhất giữ hệ cũ chạy, nên thực tế anh làm hai việc và nhánh mới luôn là việc bị hoãn. Khi Tết đến, nhánh này bị tạm dừng "hai tuần" và không bao giờ khởi động lại. Kết quả để lại: một lớp abstraction nửa vời trong `order`, một số chỗ đi đường mới, phần lớn đi đường cũ — tức là công ty đã trả chi phí phức tạp của việc chuyển đổi mà không nhận được lợi ích.
- Chi phí cơ hội thật: 4 người trong 12 tháng là khoảng 10% năng lực engineering, và trong năm đó có hai sáng kiến sản phẩm bị hoãn.
- `seller-portal` theo phương án D tiếp tục xấu đi. Chính sách "ngừng thêm tính năng" giữ được khoảng năm tháng rồi vỡ khi bộ phận kinh doanh cần một tính năng cho đối tác lớn.

**Hậu quả ngoài dự kiến:**

- **Metric bị dùng làm vũ khí.** Sau khi dashboard công khai, một team bắt đầu chia nhỏ pull request một cách giả tạo để hạ lead time trung vị. Số đẹp lên, thực tế không đổi. Hà phát hiện sau hai tháng và phải bổ sung một phép đo đối trọng (thời gian từ khi story bắt đầu đến khi giá trị đến tay người dùng, đo ở tầng story chứ không tầng commit). Đây là Goodhart's Law ở dạng sách giáo khoa: khi một phép đo trở thành mục tiêu, nó thôi là phép đo tốt. Bài học vận hành: mọi metric dùng cho quyết định nguồn lực đều cần ít nhất một metric đối trọng.
- **Quan hệ giữa Hà và Tuấn xấu đi rõ rệt trong khoảng sáu tháng.** Việc không đưa Tuấn vào nhánh strangler là đúng về mặt hệ thống, nhưng nó được Tuấn đọc là: người viết ra module này không được tin tưởng để sửa nó. Hà nói lý do một lần, riêng tư, rồi cho rằng đã xong. Thực tế thông điệp cần được nhắc lại nhiều lần và cần đi kèm một vai trò thay thế có giá trị rõ ràng — thứ chỉ được làm khi Tuấn được giao dẫn nhánh `order`, và lúc đó thì việc giao vai trò mang màu sắc bù đắp nhiều hơn màu sắc chiến lược. Một phần nguyên nhân nhánh `order` thất bại nằm ở chỗ này.
- **Một hiệu ứng ngược ở cấp tổ chức:** sau khi bốn phép đo thành công cụ xin ngân sách, một EM khác dựng một bộ số tương tự cho một module thực tế không có vấn đề, chỉ để có nguồn lực làm một dự án anh ta thích. Bộ số không sai, nhưng nó chọn khung thời gian có lợi. Sơn duyệt, và sáu tháng sau dự án đó không tạo ra thay đổi nào đo được. Bất kỳ công cụ thuyết phục nào đủ mạnh cũng sẽ bị dùng cho mục đích khác — và tổ chức cần một bước kiểm tra độc lập với người đề xuất.

**Ba góc nhìn về cùng một sự việc — quyết định không đưa Tuấn vào nhánh strangler:**

*Nhìn từ IC (Vy, engineer trong nhánh mới).* Vy thấy đây là cơ hội lớn nhất từ khi vào công ty: được làm từ đầu, có Staff Engineer kèm, có mục tiêu rõ. Nhưng Vy cũng thấy khó chịu khi phải sang hỏi Tuấn về hành vi của hệ cũ và cảm nhận được sự lạnh nhạt. Với Vy, câu chuyện là: "có một sự căng thẳng giữa hai anh chị ở trên mà em không hiểu, và nó làm em ngại đi hỏi." Điều Vy quan tâm nhất là công việc hằng ngày của mình có bị chặn không. Chi phí chính trị của quyết định, với Vy, hiện ra dưới dạng ma sát khi cần thông tin.

*Nhìn từ Tech Lead (Tuấn).* Tuấn thấy sáu năm hiểu biết của mình bị đánh giá thấp hơn một phương pháp mà một người mới về năm tháng mang đến. Anh cũng có một lập luận kỹ thuật thật, không phải chỉ tự ái: anh biết những chỗ mà strangler sẽ gặp khó — các luồng ngầm giữa `pricing-promo` và `order` — và anh tin rằng nếu đằng nào cũng phải chạm vào cả hai thì viết lại một lần rẻ hơn. Lập luận này về sau tỏ ra đúng một phần: chính những phụ thuộc ngầm đó đã giết nhánh `order`. Vấn đề của Tuấn không phải là anh sai, mà là anh không có cách nào diễn đạt lập luận của mình thành thứ so sánh được với bảng số của Hà. Anh lại rơi vào đúng vị trí đã thua hai năm trước.

*Nhìn từ Manager (Hà).* Hà nhìn thấy hai ràng buộc mà cả Vy lẫn Tuấn đều không nhìn thấy đầy đủ: (i) nếu Tuấn rời hệ cũ, rủi ro vận hành mùa campaign tăng đến mức không chấp nhận được; (ii) một nhánh refactor do người tin vào big rewrite dẫn dắt sẽ trôi dần thành big rewrite, vì mỗi quyết định nhỏ đều nghiêng về phía "làm luôn cho gọn". Cả hai ràng buộc đều đúng. Cái Hà làm sai không nằm ở quyết định mà nằm ở việc coi giao tiếp về quyết định là một sự kiện một lần thay vì một quá trình. Và có một điều Hà chỉ nhận ra sau: cô đã không tìm cách khai thác lập luận kỹ thuật của Tuấn về các phụ thuộc ngầm — cô đọc nó như sự phản đối chứ không như dữ liệu.

Cùng một quyết định, ba cách đọc, và không cách nào sai. Đây là hình dạng bình thường của một quyết định tổ chức: nó đúng ở tầng này và tốn kém ở tầng khác, và người ra quyết định phải trả phần chi phí ở tầng mà mình không trực tiếp cảm thấy.

### Bài học

**1. "Đo trước, xin sau" là thứ tự bắt buộc, không phải thứ tự khuyến nghị.** Cơ chế: một đề xuất phân bổ nguồn lực luôn cạnh tranh với các đề xuất khác trong cùng một ngân sách, và cơ chế so sánh duy nhất mà tổ chức có là các đại lượng quy đổi được. Khi engineering mang "cảm giác code tệ" vào phòng họp đó, họ không thua vì lãnh đạo không hiểu kỹ thuật — họ thua vì đề xuất của họ không nằm trong cùng một hệ đơn vị với các đề xuất khác, nên không thể được đặt lên bàn cân. Hệ quả thực hành: nếu bạn chưa có số, đừng xin; hãy dành sáu tới tám tuần để đo, và chấp nhận rằng sáu tuần đó là phần rẻ nhất của toàn bộ dự án. Đảo ngược thứ tự — xin trước rồi hứa sẽ đo — gần như luôn thất bại, vì lần từ chối đầu tiên sẽ trở thành tiền lệ cho mọi lần xin sau. Xem [06-incident-va-metrics.md](/series/engineering-leadership/06-incident-va-metrics/) về thiết kế phép đo, và [02-communication.md](/series/engineering-leadership/02-communication/) về việc dịch vấn đề kỹ thuật sang ngôn ngữ của người nghe.

**2. Technical Debt là bài toán lãi kép, và trực giác con người xử lý lãi kép rất tệ.** Cơ chế: mỗi quyết định vay nợ được đánh giá riêng lẻ, tại thời điểm nó có lợi ích tức thời rõ ràng và chi phí phân tán vào tương lai. Không có một khoản vay nào trông đủ tệ để bị từ chối, nên tất cả đều được chấp nhận, và tổng của chúng tăng theo hàm mũ. Hệ quả thực hành: không thể quản lý nợ bằng cách xét từng quyết định — phải quản lý bằng chỉ báo tổng hợp theo dõi liên tục (lead time và change failure rate theo module), giống như theo dõi tỉ lệ nợ trên vốn chứ không xét từng khoản vay. Xem [05-technical-leadership.md](/series/engineering-leadership/05-technical-leadership/).

**3. Trả nợ ở chỗ lãi suất cao nhất, không phải ở chỗ xấu nhất.** Cơ chế: chi phí thực của technical debt bằng độ khó của code nhân với tần suất chạm vào nó. Code kinh khủng nhưng không ai đụng đến có chi phí gần bằng không; code chỉ hơi lộn xộn nhưng bị sửa hằng tuần có chi phí rất lớn. Bản năng kỹ sư kéo về phía "chỗ xấu nhất" vì đó là chỗ gây khó chịu nhất, và đó là bản năng sai về mặt kinh tế. Hệ quả thực hành: phân tích lịch sử thay đổi (change frequency) trước khi chọn phạm vi refactor. Xem [05-technical-leadership.md](/series/engineering-leadership/05-technical-leadership/) và [04-decision-making.md](/series/engineering-leadership/04-decision-making/).

**4. Một dự án trả nợ không có định nghĩa "xong" bằng metric sẽ bị bỏ dở.** Cơ chế: refactor không có điểm kết thúc tự nhiên — luôn còn chỗ để dọn tiếp. Khi mục tiêu được phát biểu bằng trạng thái kiến trúc ("tách state machine ra"), tiến độ không quan sát được từ bên ngoài, nên khi có áp lực nguồn lực, dự án đó là thứ đầu tiên bị hoãn, và một khi đã hoãn thì không có tiêu chí nào để biết khi nào phải khởi động lại. Nhánh `pricing-promo` sống sót vì nó có ba con số phải đạt; nhánh `order` chết vì nó không có. Hệ quả nghiêm trọng hơn: một refactor bỏ dở tệ hơn không refactor, vì tổ chức trả chi phí phức tạp của trạng thái lai mà không nhận được lợi ích. Xem [07-project-delivery.md](/series/engineering-leadership/07-project-delivery/).

**5. Mọi metric dùng để phân bổ nguồn lực sẽ bị tối ưu hoá, nên phải đi theo cặp.** Cơ chế: khi một con số quyết định ngân sách hoặc đánh giá, hành vi của người bị đo sẽ dịch chuyển về phía làm đẹp con số bằng con đường rẻ nhất — chia nhỏ pull request, gộp incident, đổi cách gắn nhãn. Đây không phải gian lận có ý thức trong đa số trường hợp; đó là phản ứng bình thường trước một hệ khuyến khích. Hệ quả thực hành: mỗi metric tốc độ phải đi kèm một metric chất lượng hoặc một metric ở tầng cao hơn không thể tối ưu cục bộ. Xem [06-incident-va-metrics.md](/series/engineering-leadership/06-incident-va-metrics/) và [12-anti-patterns.md](/series/engineering-leadership/12-anti-patterns/).

**6. Quyết định đúng về kỹ thuật vẫn có thể thất bại vì phần con người không được xử lý.** Cơ chế: một quyết định cấu trúc luôn đồng thời là một tuyên bố về giá trị của những người liên quan. Việc để Tuấn ở lại hệ cũ là đúng về quản trị rủi ro và là một thông điệp về vị trí của anh trong tương lai của hệ thống. Người ra quyết định thường coi việc giải thích lý do một lần là đã xong; người nhận thông điệp thì diễn giải lại nó mỗi ngày qua hàng chục tín hiệu nhỏ. Ngoài ra, người phản đối mạnh nhất thường là người nắm thông tin mà bạn cần nhất — Tuấn đúng về các phụ thuộc ngầm, và thông tin đó bị bỏ lỡ vì nó đến kèm sự phản đối. Xem [03-team-leadership.md](/series/engineering-leadership/03-team-leadership/) và [11-career-evolution.md](/series/engineering-leadership/11-career-evolution/).

---

## Case 3 — Chuyển từ Monolith sang Microservices

### Bối cảnh

Công ty là một trung gian thanh toán tại TP.HCM: cổng thanh toán cho merchant, ví điện tử cho người dùng cuối, và một mảng thu hộ/chi hộ cho các đối tác tài chính. Sáu năm tuổi, có giấy phép trung gian thanh toán, chịu báo cáo định kỳ với cơ quan quản lý và một đợt audit PCI-DSS mỗi năm. Doanh thu đến từ phí giao dịch, nên downtime không chỉ là mất tiền tức thời mà còn là rủi ro merchant chuyển sang đối thủ — hợp đồng với merchant lớn có SLA 99,9% và điều khoản phạt.

Engineering: 35 người, chia làm ba team theo mảng nghiệp vụ từ khoảng mười tháng trước:

| Team | Người | Phạm vi nghiệp vụ | Tech Lead |
|---|---|---|---|
| Payment | 13 (8 BE, 2 FE, 2 QA, 1 SRE kiêm) | Cổng thanh toán, kết nối ngân hàng và tổ chức thẻ, xử lý giao dịch | Tuấn |
| Merchant | 11 (5 BE, 4 FE, 2 QA) | Onboarding merchant, hồ sơ, biểu phí, portal, đối soát, báo cáo | Khoa |
| Risk & Ledger | 8 (6 BE, 1 QA, 1 Data) | Sổ cái, số dư, hạn mức, quy tắc chống gian lận | Quân |
| Nền tảng | 3 | CI/CD, hạ tầng, monitoring — mới lập, chưa đủ người | (Linh kiêm) |

Vai khác: Sơn là CTO. Hà là Engineering Manager phụ trách Merchant và Risk & Ledger; Minh là Engineering Manager phụ trách Payment. Linh là Staff Engineer duy nhất của công ty, mới về mười một tháng, trước đó làm ở một công ty thương mại điện tử lớn hơn. Trang là Head of Product.

Hệ thống: một monolith Java Spring Boot tên nội bộ là `core-pay`, khoảng 450 nghìn dòng, một cơ sở dữ liệu PostgreSQL với 180 bảng, chạy trên 12 instance sau load balancer. Không có microservice nào ngoài hai thứ nhỏ: một service gửi OTP và một service xuất báo cáo. Deploy theo release train: thứ Ba hằng tuần, cửa sổ 22h–00h, có một quy trình phê duyệt nội bộ vì lý do compliance.

Ràng buộc sẽ quyết định mọi thứ về sau: **ba team đã được chia theo domain, nhưng codebase và pipeline thì chưa.** Ba team làm việc trong cùng một repo, cùng một database, cùng một lịch deploy. Cấu trúc tổ chức và cấu trúc kỹ thuật lệch nhau, và mọi triệu chứng dưới đây là hệ quả trực tiếp của độ lệch đó.

### Triệu chứng ban đầu

Điều đáng chú ý là không triệu chứng nào dưới đây là triệu chứng hiệu năng. Hệ thống chạy tốt. Số minh hoạ trong quý trước thời điểm quyết định: peak khoảng 900 giao dịch/giây, CPU trung bình 35%, P99 luồng thanh toán thẻ 780ms — trong đó khoảng 600ms là chờ ngân hàng, phần hệ thống tự chịu trách nhiệm chỉ khoảng 180ms; connection pool chưa bao giờ cạn; uptime 12 tháng 99,94% với bốn incident Sev1, ba trong số đó do đối tác ngân hàng.

Triệu chứng thật nằm ở tốc độ đưa thay đổi ra ngoài:

| Chỉ báo (số minh hoạ) | 18 tháng trước | Quý gần nhất |
|---|---|---|
| Lead time trung vị (commit → production) | 4 ngày | 12 ngày |
| Số release bị hoãn sang tuần sau | 1/tháng | 2,5/tháng |
| Số lần một team phải rollback vì lỗi của team khác | hiếm | 3/tháng |
| Thời gian chạy regression trước mỗi release | 3 giờ | 9 giờ |
| Số pull request có merge conflict phải giải quyết thủ công | 8% | 26% |
| Thời gian trung bình một PR chờ được merge | 6 giờ | 31 giờ |

Một số hiện tượng cụ thể hơn, kiểu chỉ nghe được khi ngồi cạnh team:

**Release train trở thành một cuộc đàm phán chính trị hằng tuần.** Mỗi thứ Hai có buổi "go/no-go" 45 phút, ba Tech Lead tranh luận xem cái gì được lên chuyến tàu. Vì tất cả cùng một artifact, một tính năng chưa sẵn sàng của Merchant có thể chặn một hotfix của Payment. Cách xử lý phổ biến là revert commit của team kia — khoảng hai lần một tháng, mỗi lần để lại dư vị xấu.

**Coupling qua database, không phải qua code.** Bảng `transaction` có 64 cột. Payment ghi vào nó, Risk truy vấn trực tiếp để chấm điểm, Merchant đọc để dựng báo cáo đối soát. Khi Payment thêm một cột và đổi ý nghĩa của cột `status`, hai team kia hỏng — và chỉ biết khi hỏng, vì không có hợp đồng nào giữa họ ngoài schema.

**Ownership danh nghĩa, không có ranh giới thật.** File `CODEOWNERS` có tồn tại, nhưng khoảng 40% pull request chạm code của từ hai team trở lên (số minh hoạ), nên phải chờ hai vòng review từ hai team có ưu tiên khác nhau. Đây là chỗ 31 giờ chờ merge đến từ.

**On-call không xác định được ai chịu trách nhiệm.** Alert bắn ra từ một process duy nhất; người trực phải đọc stack trace để đoán đây là lỗi domain nào rồi mới gọi đúng người. 22 phút trung bình từ alert đến khi đúng người bắt đầu điều tra, phần lớn là thời gian định tuyến.

**Nỗi sợ thay đổi lan sang cả những thay đổi nhỏ.** Vì regression chạy 9 giờ và vì một lỗi chặn cả ba team, các team bắt đầu gộp nhiều thay đổi vào một release để "đằng nào cũng chạy regression một lần". Batch size tăng thì tỉ lệ lỗi trên mỗi release tăng, tỉ lệ lỗi tăng thì lại càng sợ, càng gộp. Một vòng lặp tự củng cố, và là dấu hiệu đáng lo nhất trong danh sách.

### Chẩn đoán sai ban đầu

Có hai chẩn đoán sai, xảy ra nối tiếp nhau. Chẩn đoán sai thứ hai nguy hiểm hơn nhiều và tốn của công ty gần một năm.

**Chẩn đoán sai thứ nhất: "monolith không scale được nữa."**

Đây là phiên bản được nói ra ở buổi off-site kỹ thuật quý. Lập luận nghe rất tự nhiên: hệ thống chậm lại, monolith là kiến trúc cũ, các công ty lớn đều đã chuyển sang microservices, vậy nguyên nhân là monolith. Ba trong bốn slide hôm đó có cụm từ "scale".

Linh phá vỡ chẩn đoán này bằng một câu hỏi và hai ngày làm việc:

> **Linh:** "Mình đang chậm ở đâu — chậm khi chạy, hay chậm khi thay đổi?"
>
> **Tuấn:** "Chậm khi thay đổi. Chạy thì vẫn ổn."
>
> **Linh:** "Vậy microservices giải quyết cái nào trong hai cái đó?"

Hai ngày sau, Linh mang ra bảng số hiệu năng ở phần trên. Kết luận: **hệ thống không có vấn đề scale kỹ thuật, và sẽ không có trong ít nhất hai năm nữa với tốc độ tăng trưởng hiện tại.** Tách microservices vì lý do hiệu năng nghĩa là trả một khoản chi phí lớn để giải quyết một vấn đề không tồn tại.

Đến đây thì phần lớn mọi người kết luận "vậy không tách". Đó cũng là kết luận sai, và Linh nói tiếp phần quan trọng hơn:

> **Linh:** "Vấn đề thật là ba team đang cần đi ba tốc độ khác nhau nhưng bị buộc vào một nhịp deploy. Đấy là vấn đề tổ chức, không phải vấn đề kỹ thuật. Nhưng nó vẫn là một lý do chính đáng để tách — chỉ là tách theo cách khác."

Đây là điểm tinh tế nhất của case này. Trong rất nhiều tài liệu, câu "tách microservices vì lý do tổ chức chứ không vì lý do kỹ thuật" được dùng như một lời chê. Nó không phải. **Chi phí phối hợp giữa các nhóm người là một chi phí thật, nó tăng theo bình phương số người trong cùng một ranh giới thay đổi, và ở một quy mô nhất định nó vượt xa mọi chi phí kỹ thuật.** Với 35 engineer trong một repo và một pipeline, đó chính xác là chỗ công ty này đang đứng. Lý do tổ chức là một lý do đủ tốt để tách.

Nhưng lý do tổ chức đặt ra một ràng buộc mà lý do kỹ thuật không đặt ra: **nếu bạn tách để mỗi team đi được độc lập, thì ranh giới service phải trùng với ranh giới của việc thay đổi — tức ranh giới domain — chứ không phải ranh giới của kiến trúc kỹ thuật.** Tách theo tầng (API layer riêng, business layer riêng, job riêng) hay theo loại thao tác đều tạo ra các service mà một thay đổi nghiệp vụ phải đi xuyên qua nhiều cái cùng lúc: bạn giữ nguyên chi phí phối hợp và cộng thêm chi phí mạng, chi phí vận hành, chi phí nhất quán dữ liệu. Đó là distributed monolith, kết cục phổ biến nhất của các dự án chuyển đổi kiểu này.

**Chẩn đoán sai thứ hai: "chúng ta đã hiểu domain đủ rõ để vẽ ranh giới."**

Chẩn đoán này không ai nói thành lời, nên không ai phản biện. Sau khi thống nhất tách theo domain, ba Tech Lead vẽ ranh giới trong hai buổi họp và ra một sơ đồ bốn service: Merchant, Payment, Ledger, Risk. Nó ánh xạ gọn gàng vào ba team hiện có, ai cũng thấy tên miền của mình trong đó, và không ai phản đối.

Vấn đề là ranh giới được vẽ bằng **danh từ nghiệp vụ**, không phải bằng **quan sát về cách hệ thống thực sự thay đổi**. "Risk" là một khái niệm nghiệp vụ có thật, nhưng trong hệ thống này, mọi quy tắc rủi ro đều đọc trạng thái của giao dịch ngay lúc giao dịch đang diễn ra, và mọi luật rủi ro mới đều cần một trường dữ liệu mới mà chỉ luồng thanh toán biết. Risk và Payment thay đổi cùng nhau. Dữ liệu này không nằm trong đầu ai — nó nằm trong lịch sử Git, và không ai nhìn vào đó.

Linh về sau chạy phân tích co-change trên 24 tháng lịch sử (số minh hoạ — tỉ lệ commit chạm cả hai vùng code trên tổng số commit chạm ít nhất một trong hai):

| Cặp vùng code | Tỉ lệ co-change |
|---|---|
| Merchant ↔ Payment | 7% |
| Merchant ↔ Ledger | 11% |
| Ledger ↔ Payment | 14% |
| **Risk ↔ Payment** | **58%** |

Bảng này, nếu có trước, đã đủ để chặn việc tách Risk. Nó không có trước, vì nó là loại bằng chứng mà người ta chỉ nghĩ đến khi đã trả giá.

### Các lựa chọn thực tế

Bốn phương án được đưa lên bàn, cộng một phương án thứ năm chỉ được nói ra sau khi mọi chuyện đã xảy ra.

**Phương án A — Modular monolith.** Không tách process. Chia repo thành các module có ranh giới cưỡng chế bằng công cụ (module này không được import package của module kia, vi phạm thì build fail); tách database thành các schema riêng theo domain, cấm truy vấn chéo; tách pipeline test để chỉ chạy test của module bị ảnh hưởng; chuyển từ release train sang trunk-based với feature flag để mỗi team merge và bật/tắt độc lập. Ước lượng 3 engineer trong 5 tháng. Linh ủng hộ, ban đầu.

**Phương án B — Tách theo ranh giới kỹ thuật.** Dễ nhất về thực thi và vì thế nguy hiểm nhất. Tách những thứ có ranh giới kỹ thuật rõ và ít phụ thuộc: một BFF cho portal, một service chạy toàn bộ batch job, một service notification, một service report. Ước lượng 2 engineer trong 4 tháng. Sức hấp dẫn: bắt đầu được ngay tuần sau, và khoe được "chúng ta đã có microservices" trong hai tháng.

**Phương án C — Tách theo ranh giới domain, tuần tự, mỗi service một team own trọn.** Mỗi lần một service, có tiêu chí nghiệm thu rõ ràng trước khi bắt đầu cái tiếp theo. Thứ tự dự kiến: Merchant trước (ít coupling nhất, dùng để học), rồi Ledger (khó nhất về dữ liệu nhưng ranh giới nghiệp vụ sắc nhất), rồi xét tiếp. Ước lượng 6 engineer, 12–15 tháng, cộng đầu tư nền tảng.

**Phương án D — Viết lại toàn bộ.** Hệ mới song song với 10–12 service, chạy hai hệ, cắt sang theo merchant. Ước lượng lạc quan 18 tháng. Được nêu ra chủ yếu để bị loại, nhưng lập luận ủng hộ không ngớ ngẩn: hệ hiện tại có những giả định về mô hình dữ liệu — một bảng `transaction` phẳng cho cả thẻ, ví, chuyển khoản và thu hộ — mà mọi cách sửa dần đều phải thoả hiệp với.

**Phương án E — Tách tổ chức thay vì tách hệ thống.** Gộp ba team thành hai, hoặc lập một team release engineering chịu trách nhiệm điều phối. Nghe ngược đời, nhưng đây là phương án đúng ở một số quy mô.

### Trade-off của từng lựa chọn

| Phương án | Được | Mất | Ai chịu phần mất | Điều kiện nên chọn |
|---|---|---|---|---|
| **A — Modular monolith** | Giải quyết đúng nguyên nhân với chi phí vận hành gần bằng không; không mạng, không nhất quán cuối, không transaction phân tán; đảo ngược được nếu sai | Không tách được nhịp deploy — vẫn một artifact, một lần rollback ảnh hưởng tất cả; ranh giới chỉ giữ được bằng kỷ luật và công cụ, mà kỷ luật xói mòn dưới áp lực deadline | Team chạy chậm nhất vẫn kéo lùi các team khác ở khâu release; và người phải giữ luật kiến trúc | Dưới ~25–30 engineer; ranh giới domain chưa ổn định; chưa có nền tảng vận hành |
| **B — Ranh giới kỹ thuật** | Nhanh, rủi ro từng bước thấp, tạo cảm giác tiến bộ; có vài lợi ích thật (tách batch nặng khỏi luồng online là tách đúng) | Không giảm chi phí phối hợp vì thay đổi nghiệp vụ vẫn xuyên nhiều service; cộng toàn bộ chi phí phân tán mà không có lợi ích tổ chức; tệ nhất là tạo tiền lệ về cách vẽ ranh giới | Toàn bộ engineering, đặc biệt người trực on-call trong hai năm tới | Chỉ đúng cho generic subdomain thật sự — notification, gửi file, export báo cáo, cron job nặng |
| **C — Domain, tuần tự** | Ranh giới trùng ranh giới thay đổi nên chi phí phối hợp giảm thật; mỗi team có nhịp deploy và on-call riêng; học được sau mỗi lần tách; dừng được sau service đầu | Chi phí vận hành tăng ngay còn lợi ích đến sau 2–3 quý — hố lõm mà nhiều tổ chức bỏ cuộc; đòi hỏi đầu tư nền tảng trước khi có bất kỳ lợi ích nào và năng lực vẽ ranh giới mà công ty có thể chưa có | Team nền tảng quá tải trước; ban lãnh đạo chịu 2–3 quý không thấy cải thiện | Trên ~30 engineer; domain đã ổn định; có 2–3 người dựng và vận hành được nền tảng |
| **D — Viết lại toàn bộ** | Thoát ràng buộc mô hình dữ liệu cũ | Không có giá trị giao được trong 12+ tháng; hai hệ song song trong fintech là hai nguồn sự thật về tiền, phải đối soát chéo liên tục; compliance phải chứng nhận lại | Toàn công ty, tập trung vào ngày cắt | Gần như không bao giờ với hệ đang xử lý tiền; chỉ khi nền tảng cũ hết hỗ trợ hoặc pháp lý buộc thay |
| **E — Tách tổ chức** | Chi phí gần bằng không, hiệu lực trong một sprint | Không mở rộng được, chỉ mua thêm thời gian; gộp team làm mất focus domain và tăng cognitive load | Tech Lead bị mở rộng phạm vi; người mất đường phát triển trong team cũ | Dưới ~20 engineer, hoặc khi ba team thực chất phục vụ cùng một dòng giá trị |

Một điểm hay bị bỏ qua khi so sánh A và C: **chúng không loại trừ nhau, chúng là hai điểm trên cùng một trục.** Việc khó nhất trong cả hai — vẽ đúng ranh giới và cắt được phụ thuộc dữ liệu — là giống hệt nhau; khác biệt duy nhất là trong modular monolith, ranh giới sai sửa bằng một lần refactor, còn khi đã tách process thì sửa ranh giới sai đòi hỏi migration dữ liệu, đổi hợp đồng API và thường là đổi cả cơ cấu team. **Giá của một ranh giới sai chênh nhau khoảng một bậc độ lớn giữa hai kiến trúc.** Đó là lý do A rồi C gần như luôn tốt hơn C thẳng — và trong case này, việc bỏ qua bước A là gốc rễ của sai lầm kể ở dưới.

### Quyết định và cách thực thi

Quyết định cuối cùng, sau ba tuần tranh luận: **chọn C, tách theo domain, tuần tự, ba đợt trong 15 tháng; kèm một phần nhỏ của B cho notification và export báo cáo; không làm A một cách tường minh.** Việc bỏ qua A có lý do, và lý do đó nghe được ở thời điểm ấy:

> **Sơn:** "Nếu mình bỏ năm tháng làm modular monolith rồi mới tách, thì mình mất năm tháng. Team đang đau bây giờ. Mình biết ranh giới ở đâu rồi — ba team đã chia theo domain mười tháng nay."
>
> **Linh:** "Mình biết ranh giới của tổ chức. Cái đó chưa chắc là ranh giới của code."
>
> **Sơn:** "Nếu nó lệch thì mình sẽ thấy khi tách. Anh không muốn dừng thêm một quý nữa."
>
> **Linh:** "Em đồng ý làm theo hướng này. Nhưng em muốn ghi vào biên bản một câu: nếu tách xong một service mà vẫn phải deploy nó cùng lúc với monolith, thì đó là dấu hiệu tách sai, và mình phải dừng lại xem xét chứ không tách tiếp."

Câu cuối của Linh được ghi vào biên bản. Nó là thứ duy nhất về sau cứu được công ty khỏi việc tách thêm hai service sai nữa.

**Bước 1 — Đầu tư nền tảng trước, ba tháng, trước khi tách bất cứ gì.** Ba người của team Nền tảng cộng hai người mượn từ Payment dựng: CI/CD theo service với template chung; tracing phân tán; log tập trung gắn nhãn service; service template gồm health check, metric, circuit breaker, retry có backoff, idempotency key; và quan trọng nhất — **quy trình on-call theo service với runbook bắt buộc**. Sơn hỏi tại sao mất ba tháng cho thứ không tạo ra tính năng nào. Câu trả lời của Linh trong RFC:

> Một service không có tracing, không có runbook, không có người trực rõ ràng thì không phải là service — nó là một cách mới để bị đánh thức lúc 3 giờ sáng mà không biết gọi ai. Chi phí vận hành của microservices được trả trước và trả bằng nền tảng. Nếu không trả khoản đó, ta vẫn sẽ trả, nhưng bằng incident.

**Bước 2 — `merchant-service`, đợt tách thứ nhất (tháng 4–8).** Chọn Merchant đi trước không phải vì nó quan trọng nhất mà vì nó **rủi ro thấp nhất và dạy được nhiều nhất**: co-change dưới 11%, không nằm trên đường đi đồng bộ của giao dịch, ranh giới dữ liệu rõ. Cách làm: service mới với database riêng, đồng bộ hai chiều bằng change data capture trong giai đoạn chuyển tiếp, chuyển từng nhóm chức năng của portal sang API mới sau một facade, cuối cùng cắt quyền ghi khỏi monolith. Mất 5 tháng thay vì 3 — phần chênh gần như toàn bộ nằm ở 14 chỗ trong monolith đang đọc trực tiếp bảng merchant mà không ai biết.

**Bước 3 — `ledger-service`, đợt tách thứ hai (tháng 8–14).** Khó hơn nhiều vì đây là nơi giữ tiền. Ba quyết định thiết kế tạo nên thành công của đợt này: **API dạng command thô chứ không CRUD** (`reserve`, `commit`, `reverse`, `postEntry` — mỗi lời gọi là một đơn vị nghiệp vụ trọn vẹn); **mọi lời gọi có idempotency key** để retry không sinh bút toán trùng; và **chấp nhận nhất quán cuối một cách tường minh, thiết kế quy trình đối soát quanh nó** (job đối soát mỗi 15 phút, đối soát toàn phần hằng đêm, cảnh báo khi lệch). Quân dẫn đợt này. Kết quả: 2 lời gọi trên mỗi giao dịch, không có lời gọi ngược từ ledger về payment.

**Bước 4 — `risk-service`, đợt tách thứ ba (tháng 12–17). Đây là đợt tách sai.**

Quyết định tách Risk được đưa ra trong lúc hai đợt đầu đang thành công, và điều đó có ý nghĩa: **thành công tạo ra động lượng, và động lượng làm giảm mức độ hoài nghi.** Không ai chạy lại phân tích co-change. Lập luận ủng hộ nghe rất mạnh: team đã có kinh nghiệm tách một service; quy tắc rủi ro thay đổi rất thường xuyên nên "cần deploy độc lập"; và có kế hoạch dùng một mô hình machine learning cần Python, không chạy được trong JVM monolith.

Lý do thứ ba là lý do tốt nhất và cũng là cái bẫy: nó đúng cho **một phần** của Risk (chấm điểm bằng mô hình, chạy bất đồng bộ) nhưng bị dùng để biện minh cho việc tách **toàn bộ** Risk, gồm cả phần luật cứng chạy đồng bộ trong đường đi của giao dịch. Ranh giới đã được vẽ theo danh từ tổ chức — "team Risk làm gì thì đó là service Risk" — thay vì theo tính chất của từng luồng.

Service được tách trong 5 tháng và chạy trên production 9 tháng trước khi bị gộp lại.

**Bước 5 — Nhận ra tách sai.** Bốn dấu hiệu, xuất hiện theo thứ tự này (số minh hoạ):

*Dấu hiệu 1 — Deploy khớp nhau.* Trong 9 tháng, **83% các lần deploy `risk-service` phải đi kèm một lần deploy `core-pay` trong cùng cửa sổ**, 61% có ràng buộc thứ tự. Nghĩa đơn giản: hai service này là một đơn vị triển khai. Lợi ích chính của việc tách bằng không, trong khi chi phí điều phối cao hơn trước, vì giờ phải điều phối hai pipeline thay vì một.

*Dấu hiệu 2 — Chatty API.* Một giao dịch thẻ gọi `risk-service` 4 lần. Payload lớn dần vì risk cần trạng thái mà chỉ payment có — đến tháng thứ sáu, lời gọi thứ ba có 47 trường. Mỗi luật rủi ro mới là một lần thêm trường, tức một lần đổi hợp đồng API, tức một lần deploy khớp nhau. P99 luồng thanh toán tăng 140ms, trong đó khoảng 90ms là chi phí mạng và serialize thuần tuý.

*Dấu hiệu 3 — Transaction phân tán.* Luật hạn mức đòi hỏi giữ chỗ một khoản trước khi gọi ngân hàng và nhả ra nếu bị từ chối. Trước khi tách, đây là một transaction database; sau khi tách, là một saga hai bước xuyên ba service với một job dọn reservation mồ côi. Job đó phát sinh 4 incident Sev2 trong 9 tháng — mỗi lần là một nhóm người dùng bị treo hạn mức, phải xử lý thủ công.

*Dấu hiệu 4 — Chi phí nhận thức không giảm.* Để sửa một bug về hạn mức, engineer phải mở hai repo, hiểu hai mô hình dữ liệu mô tả cùng một thứ, dựng môi trường có cả hai service. Thời gian để một engineer mới merge được thay đổi đầu tiên: 31 ngày, so với 12 ngày ở `ledger-service`.

Đến tháng thứ bảy, Hà mang bốn dấu hiệu này lên bảng và trích lại câu Linh đã ghi vào biên bản 15 tháng trước:

> **Hà:** "83% deploy phải đi kèm. Anh đọc con số này thế nào?"
>
> **Quân:** "Em đọc là mình chưa tách xong. Nếu em đẩy nốt phần state của giao dịch sang risk thì hết phụ thuộc."
>
> **Hà:** "Đẩy state giao dịch sang risk nghĩa là risk trở thành nơi giữ trạng thái giao dịch. Lúc đó cái nào là payment?"
>
> **Quân:** "..." *(sau một lúc)* "Nghĩa là em đang đề xuất gộp lại nhưng đổi tên."
>
> **Hà:** "Đúng. Và đó là câu trả lời."

Khoảnh khắc này quan trọng vì nó cho thấy hình dạng điển hình của việc nhận ra ranh giới sai: **triệu chứng không bao giờ tự nói "ranh giới sai", nó luôn nói "chưa tách đủ".** Mỗi lần bạn đẩy thêm dữ liệu qua ranh giới để cắt một phụ thuộc, bạn tạo ra phụ thuộc mới ở chiều ngược lại. Nếu sau ba vòng như vậy mà tổng số lời gọi chéo không giảm, ranh giới đó sai — không phải chưa xong.

**Bước 6 — Gộp lại, và chi phí thật của việc gộp.** Quyết định gộp `risk-service` trở lại `core-pay` được đưa ra ở tháng 24 và mất **4,5 tháng với 3 engineer** — nhiều hơn cả thời gian tách ra, tính theo người-tháng. Các khoản chi phí, xếp theo mức độ hay bị đánh giá thấp:

| Khoản | Nội dung | Vì sao bị đánh giá thấp |
|---|---|---|
| Dữ liệu | Database riêng đã chạy 9 tháng, giữ quyết định rủi ro của ~40 triệu giao dịch; compliance buộc lưu 5 năm và phải giải trình được vì sao một giao dịch bị từ chối | Không thể xoá. Chọn giữ như kho lịch sử chỉ đọc, nghĩa là mọi truy vấn tra cứu phải biết đọc hai nguồn — một khoản nợ vĩnh viễn |
| Consumer ngoài dự tính | Hệ thống chăm sóc khách hàng và một pipeline của team Data đã bắt đầu gọi API của `risk-service` | Phải giữ một facade mỏng chuyển tiếp vào monolith, tức là về hình thức service vẫn tồn tại sau khi đã gộp. **Một API công khai không bao giờ chết đúng lịch** |
| Compliance | Kiến trúc xử lý dữ liệu nhạy cảm đã nằm trong hồ sơ audit; đổi kiến trúc là cập nhật hồ sơ, mô tả lại luồng dữ liệu, giải trình ở kỳ sau | ~3 tuần công cộng một cuộc họp không dễ chịu; không ai đưa khoản này vào ước lượng |
| Tổ chức | Ba engineer đang own `risk-service` phải quay về monolith cùng team Payment, dưới một Tech Lead khác, trong một nhịp deploy khác. Một người nghỉ việc | Khoản đắt nhất. Conway's Law chạy chiều ngược: **khi cấu trúc tổ chức đã bám theo cấu trúc hệ thống, mỗi lần sửa hệ thống là một lần reorg** |
| Uy tín và tâm lý | Ba tháng đầu, cụm từ "thì lúc trước bảo tách" xuất hiện thường xuyên; Quân đã cân nhắc nghỉ việc | Điều giữ Quân lại là cách Hà đóng khung việc gộp trước toàn engineering: không phải "chúng ta đã sai" mà "chúng ta đã chạy một thí nghiệm đắt và bây giờ biết một điều về hệ thống của mình mà trước đó không tài liệu nào nói" |

**Ba góc nhìn về cùng một sự việc — quyết định gộp `risk-service` trở lại:**

*Nhìn từ IC (Vy, engineer trong team Risk & Ledger).* Vy đã bỏ chín tháng vào service này. Với Vy, việc gộp trước hết là câu hỏi về bản thân: chín tháng đó tính là gì, và kỳ Performance Review sắp tới viết gì vào phần thành tựu. Vy còn thấy một điều rất cụ thể mà cấp trên không thấy: gộp lại nghĩa là quay về chờ release train thứ Ba, quay về regression 9 giờ, quay về xin review từ team khác. Đó là mất mát rõ ràng về chất lượng công việc hằng ngày, và không bảng số nào bù được. Vy là người nghỉ việc.

*Nhìn từ Tech Lead (Quân).* Hai thứ chồng lên nhau. Thứ nhất là lập luận kỹ thuật: anh tin có một cách tách đúng — tách phần chấm điểm bất đồng bộ, giữ phần luật cứng ở lại — và anh đúng, đó chính là phương án cuối cùng. Thứ hai khó nói hơn: quyết định tách toàn bộ Risk là của anh, được duyệt dựa trên uy tín của anh, và bây giờ bị đảo. Quân mất khoảng sáu tuần để tách hai thứ này ra khỏi nhau, và trong sáu tuần đó mọi lập luận kỹ thuật anh đưa ra đều bị nghe như phòng thủ, kể cả khi đúng — lặp lại đúng mẫu đã thấy ở Case 2 với Tuấn.

*Nhìn từ Manager (Hà).* Hà thấy thứ mà cả hai người kia không có vị trí để thấy: đây là quyết định về **precedent** nhiều hơn về một service. Nếu để `risk-service` tồn tại thêm một năm với 83% deploy khớp nhau, định nghĩa nội bộ về "một service tốt" sẽ bị hạ xuống và ba lần tách tiếp theo sẽ lặp lại mẫu đó. Chi phí gộp một service là 4,5 tháng; chi phí chuẩn hoá một ranh giới sai thành chuẩn mực thì không đo được. Nhưng Hà cũng đánh giá thấp một thứ: cô coi việc gộp là quyết định kiến trúc và giao Quân thực thi, trong khi phần khó nhất của nó là quyết định tổ chức. Cô xử lý phần đó muộn sáu tuần, và Vy nghỉ việc trong sáu tuần ấy.

**Bước 7 — Tách lại đúng phần đáng tách.** Sau khi gộp, phần chấm điểm bằng mô hình được tách thành một service nhỏ chạy Python, giao tiếp bất đồng bộ qua message queue, không nằm trên đường đi đồng bộ của giao dịch. Service này có 0 lần deploy khớp bắt buộc trong 6 tháng đầu và không tham gia transaction phân tán nào. Đó là ví dụ về ranh giới đúng: **cắt theo tính chất của luồng — đồng bộ hay bất đồng bộ, có hay không giữ trạng thái giao dịch — chứ không cắt theo tên phòng ban.**

### Hậu quả

**Số liệu sau 30 tháng (số minh hoạ), so với thời điểm bắt đầu:**

| Chỉ báo | Trước | Tháng 12 | Tháng 30 |
|---|---|---|---|
| Lead time trung vị toàn công ty | 12 ngày | 14 ngày | 5 ngày |
| Lead time của team Merchant | 12 ngày | 6 ngày | 2 ngày |
| Số release bị hoãn/tháng | 2,5 | 2 | 0,5 |
| Thời gian regression trước release | 9 giờ | 7 giờ | 1,5 giờ (theo service) |
| Số incident Sev1–Sev2 / quý | 6 | 11 | 5 |
| Thời gian định tuyến alert đến đúng người | 22 phút | 17 phút | 4 phút |
| Chi phí hạ tầng/tháng | 100 (chuẩn hoá) | 152 | 168 |
| Số người trong team Nền tảng | 3 | 5 | 7 |

Bảng này chứa toàn bộ bài học về thời gian của loại dự án này. **Ở tháng 12, mọi chỉ số quan trọng đều xấu hơn hoặc bằng lúc bắt đầu.** Nếu ban lãnh đạo đánh giá dự án ở tháng 12 — mốc đánh giá tự nhiên vì trùng cuối năm tài chính — kết luận hợp lý nhất sẽ là dừng. Lợi ích chỉ rõ ràng từ khoảng tháng 18 và chỉ áp đảo ở tháng 30.

Điều cứu dự án ở tháng 12 không phải niềm tin, mà là một quyết định về cách đo đưa ra từ đầu: **đo lead time theo từng team, không chỉ trung bình toàn công ty.** Ở tháng 12, team Merchant — team duy nhất đã tách xong — có lead time 6 ngày trong khi hai team còn lại 17 ngày. Con số 6 ngày là bằng chứng cơ chế hoạt động; con số trung bình 14 ngày chỉ nói rằng phần lớn công ty chưa được hưởng nó. Trung bình toàn công ty là phép đo tồi cho một quá trình chuyển đổi tuần tự, vì nó pha loãng tín hiệu của phần đã thành công vào phần chưa bắt đầu.

**Phần tốt:** `merchant-service` và `ledger-service` đều đạt tiêu chí — dưới 5% deploy phải khớp với monolith, không tham gia transaction đồng bộ xuyên service, mỗi service một team own và trực riêng. Team Merchant deploy 9 lần/tuần ở tháng 30 so với 1 lần/tuần lúc đầu, change failure rate giảm từ 18% xuống 6%. Vòng lặp "sợ deploy → gộp lớn → nhiều lỗi → càng sợ" bị phá ở hai trong ba team. Và năng lực nền tảng trở thành tài sản: khi công ty mở một dòng sản phẩm mới ở tháng 26, service đầu tiên của dòng đó lên production trong 3 tuần thay vì 3 tháng.

**Phần xấu:**

- Chi phí trực tiếp của lần tách sai: 5 tháng × 4 người để tách, 4,5 tháng × 3 người để gộp, khoảng 22 người-tháng, chưa tính 9 tháng vận hành với P99 cao hơn 140ms và 4 incident Sev2 — khoảng 8–9% năng lực engineering của một năm.
- Một người giỏi nghỉ việc, và lý do nghỉ truy được về việc phần tổ chức của quyết định gộp bị xử lý muộn.
- Chi phí hạ tầng tăng 68% và không quay về mức cũ. Đây không phải thất bại — đó là giá đúng của kiến trúc mới — nhưng ước lượng ban đầu chỉ tính 25%.
- Team Payment, team lớn nhất, vẫn ở trong monolith ở tháng 30 với lead time 8 ngày. Nghịch lý: team đau nhất lúc đầu được hưởng lợi cuối cùng, vì domain của họ khó tách nhất. Nó tạo ra một sự bất mãn âm ỉ không có lời giải kỹ thuật.

**Hậu quả ngoài dự kiến:**

- **Ranh giới service trở thành ranh giới chính trị.** Tranh luận "tính năng này thuộc service nào" bắt đầu mang màu sắc phân bổ nguồn lực chứ không thuần tuý là thiết kế; ba lần trong hai năm, một tính năng được đặt vào service sai về thiết kế vì team kia không có người. Cách giảm nhẹ duy nhất tìm được: bắt buộc viết một ADR ngắn nêu lý do cho mỗi quyết định đặt tính năng, và đọc lại các ADR đó ở review kiến trúc hằng quý.
- **On-call theo service tạo ra bất bình đẳng mới.** Người trực `ledger-service` bị gọi nhiều gấp ba người trực `merchant-service` vì ledger nằm trên đường đi của tiền, trong khi chế độ trực ban đầu như nhau. Mất hai quý và một lần suýt mất người mới đổi sang tính tải trực theo số lần bị gọi thực tế thay vì theo số ca.
- **Nghịch lý của việc tách thành công:** khi Merchant deploy độc lập được, hai lần trong năm đó một thay đổi của họ làm hỏng giả định của Payment mà contract test không bắt được — vì contract test kiểm tra hình dạng dữ liệu, không kiểm tra ngữ nghĩa. Deploy độc lập không loại bỏ coupling; nó chuyển coupling từ chỗ nhìn thấy được (buổi họp thứ Hai) sang chỗ không nhìn thấy (runtime). Đó là đánh đổi đúng, nhưng phải biết mình đang đánh đổi cái gì.

### Bài học

Trước sáu bài học, một phần phải nói riêng, vì trong đa số trường hợp thực tế ở thị trường Việt Nam câu trả lời đúng là **không tách — hoặc chưa tách**. Sáu điều kiện dưới đây, nếu có từ hai điều trở lên, thì việc tách gần như chắc chắn tạo ra distributed monolith:

1. **Quy mô dưới ngưỡng.** Chi phí phối hợp trong một codebase tăng theo số người, nhưng chi phí vận hành của kiến trúc phân tán là khoản cố định lớn phải trả trước. Dưới khoảng 20–25 engineer, khoản cố định đó gần như luôn lớn hơn khoản tiết kiệm được. Dấu hiệu: số service chia cho số engineer lớn hơn 0,5 nghĩa là bạn có nhiều service hơn số người trực được chúng.
2. **Domain chưa ổn định.** Nếu ranh giới nghiệp vụ đã đổi từ hai lần trở lên trong 12 tháng qua, mọi ranh giới vẽ hôm nay sẽ sai trong sáu tháng. Đây là lý do mạnh nhất để một startup giai đoạn sớm giữ monolith kể cả khi đủ năng lực làm khác.
3. **Chưa có nền tảng vận hành.** Nếu chưa có tracing, log tập trung, CI/CD tự phục vụ, service template và on-call theo service, thì chi phí thật của mỗi service mới không phải chi phí viết nó, mà là chi phí không nhìn thấy được bên trong nó lúc 3 giờ sáng.
4. **Tỉ lệ co-change cao.** Kinh nghiệm thực hành: **dưới 15% thì tách được; 15–30% phải xem xét kỹ và có thể cần vẽ lại ranh giới; trên 30% thì không tách.** Ở case này, Risk ↔ Payment là 58%.
5. **Không cắt được dữ liệu.** Nếu hai vùng phải nhất quán tức thời — hai bên một bút toán kép, giữ chỗ hạn mức và trừ tiền — thì tách nghĩa là biến một transaction database thành một saga, tức một hệ thống khác hẳn với trạng thái trung gian, bù trừ, bản ghi mồ côi và một cơ chế đối soát phải xây và phải vận hành.
6. **Vấn đề thật là tổ chức và sửa được bằng tổ chức.** Câu hỏi kiểm tra: nếu ba team này ngồi chung với một backlog chung, họ có còn xung đột không? Nếu không, vấn đề nằm ở cách chia team.

Một trường hợp riêng: **tách để tuyển hoặc để giữ người.** Lý do này thật hơn người ta thừa nhận — kỹ sư muốn có microservices trong CV, và ở thị trường mà job hopping là bình thường, đó là áp lực thật lên người quản lý. Nó không phải lý do đủ. Nếu vấn đề là giữ người, giải bằng đường phát triển nghề nghiệp và chất lượng công việc hằng ngày, đừng giải bằng một quyết định kiến trúc mà công ty trả giá trong năm năm. Xem [08-hiring-va-phat-trien.md](/series/engineering-leadership/08-hiring-va-phat-trien/).

**1. Lý do tổ chức là lý do chính đáng để tách — nhưng nó buộc ranh giới phải là ranh giới domain.** Cơ chế: khi tách để giảm chi phí phối hợp giữa các nhóm người, thứ cần đạt được là một thay đổi nghiệp vụ điển hình chỉ chạm một service, do đó chỉ cần một team, một lần review, một lần deploy. Ranh giới kỹ thuật (tầng, loại thao tác, công nghệ) cắt vuông góc với hướng của thay đổi nghiệp vụ, nên mọi thay đổi đều xuyên nhiều service — bạn giữ nguyên chi phí phối hợp và cộng thêm chi phí mạng, nhất quán, vận hành. Hệ quả thực hành: trước khi vẽ ranh giới, lấy năm thay đổi nghiệp vụ gần nhất và đếm xem mỗi cái chạm bao nhiêu service trong sơ đồ đề xuất; trung bình lớn hơn 1,5 thì sơ đồ sai. Xem [09-to-chuc-va-scaling.md](/series/engineering-leadership/09-to-chuc-va-scaling/) và [05-technical-leadership.md](/series/engineering-leadership/05-technical-leadership/).

**2. Conway's Law chạy cả hai chiều, và chiều ngược lại là chiều đắt.** Cơ chế: hệ thống sao chép cấu trúc giao tiếp của tổ chức tạo ra nó — chiều thuận, và Inverse Conway Manoeuvre khai thác nó bằng cách chia team trước rồi để kiến trúc bám theo. Chiều ngược ít được nói: một khi cấu trúc hệ thống đã ổn định, tổ chức bị khoá vào nó, và mọi lần sửa ranh giới hệ thống trở thành một lần reorg với đầy đủ chi phí con người. Ở đây, gộp một service về kỹ thuật là 4,5 tháng; về con người là ba người đổi team, một người nghỉ. Hệ quả thực hành: coi mỗi quyết định vẽ ranh giới service như một quyết định nhân sự dài hạn, định giá nó theo chi phí đảo ngược chứ không theo chi phí thực hiện. Xem [09-to-chuc-va-scaling.md](/series/engineering-leadership/09-to-chuc-va-scaling/) và [00-nen-tang-leadership.md](/series/engineering-leadership/00-nen-tang-leadership/).

**3. Distributed monolith có bốn dấu hiệu đo được, cả bốn xuất hiện trong quý đầu — nếu chịu đo.** Cơ chế: khi ranh giới sai, hệ thống liên tục phát tín hiệu, nhưng tín hiệu ấy luôn được diễn giải là "chưa tách xong" chứ không phải "tách sai", vì diễn giải thứ nhất cho phép đi tiếp còn diễn giải thứ hai đòi hỏi thừa nhận sai lầm. Bốn dấu hiệu: tỉ lệ deploy phải khớp với service khác (ngưỡng báo động khoảng 20%); số lời gọi đồng bộ chéo trên một đơn vị nghiệp vụ tăng dần thay vì giảm; xuất hiện transaction phân tán trên đường đi đồng bộ; thời gian onboarding vào service mới không thấp hơn vào monolith. Hệ quả thực hành: ghi bốn ngưỡng này vào biên bản quyết định *trước* khi tách, kèm cam kết dừng lại xem xét khi vượt — vì sau khi tách, không ai còn đủ khách quan để đặt ngưỡng. Xem [06-incident-va-metrics.md](/series/engineering-leadership/06-incident-va-metrics/) và [12-anti-patterns.md](/series/engineering-leadership/12-anti-patterns/).

**4. Chi phí đến ngay, lợi ích đến sau 2–3 quý, và khoảng lõm ở giữa là chỗ dự án chết.** Cơ chế: mọi chi phí của kiến trúc phân tán — hạ tầng, nền tảng, độ phức tạp vận hành, incident do mô hình lỗi mới — phát sinh từ service đầu tiên, trong khi lợi ích chỉ hiện ra khi một team đã tách xong và nhịp deploy mới đã ổn định. Với chu kỳ đánh giá theo quý hoặc năm tài chính, dự án gần như chắc chắn bị đánh giá đúng vào lúc nó trông tệ nhất. Hệ quả thực hành: chốt trước thời điểm và tiêu chí đánh giá ngay khi xin ngân sách; đo lead time theo từng team thay vì trung bình toàn công ty; và chọn service đầu tiên theo tiêu chí "chứng minh được cơ chế sớm nhất", không theo tiêu chí "quan trọng nhất". Xem [07-project-delivery.md](/series/engineering-leadership/07-project-delivery/) và [04-decision-making.md](/series/engineering-leadership/04-decision-making/).

**5. Phân tích co-change là phép thử rẻ nhất và bị bỏ qua thường xuyên nhất.** Cơ chế: ranh giới đúng là ranh giới mà hai bên thay đổi độc lập; lịch sử Git là bản ghi khách quan duy nhất về việc thứ gì thực sự thay đổi cùng thứ gì, và nó miễn nhiễm với ý kiến, chức danh và sơ đồ đẹp. Ở đây, một phép đo nửa ngày công đã có thể tiết kiệm 22 người-tháng. Nó bị bỏ qua không phải vì khó, mà vì ở thời điểm ra quyết định mọi người đã tin là mình biết câu trả lời — và người ta không đi tìm bằng chứng cho thứ mình đã tin. Hệ quả thực hành: đưa co-change thành mục bắt buộc trong template RFC cho mọi đề xuất tách service, để nó được chạy vì quy trình chứ không vì ai đó nghi ngờ. Xem [04-decision-making.md](/series/engineering-leadership/04-decision-making/) và [05-technical-leadership.md](/series/engineering-leadership/05-technical-leadership/).

**6. Cách bạn nói về một quyết định bị đảo sẽ quyết định lần sau có ai dám đề xuất nữa không.** Cơ chế: chi phí lớn nhất của một sai lầm kỹ thuật hiếm khi là chi phí sửa nó, mà là mức độ giảm sẵn sàng chấp nhận rủi ro của mọi người sau đó. Nếu việc gộp `risk-service` được đóng khung là thất bại của Quân, thông điệp mà 35 engineer nhận được là: đề xuất thay đổi lớn có rủi ro cá nhân cao, cách an toàn là không đề xuất gì. Hệ quả thực hành: tách bạch giữa đánh giá *quyết định* (thông tin có tại thời điểm đó, chất lượng lập luận) và đánh giá *kết quả* (bị chi phối bởi may rủi); công khai điều đã học được về hệ thống; và xử lý phần tổ chức của việc đảo ngược — ai đổi team, ai đổi Tech Lead, ai mất phạm vi — sớm hơn phần kỹ thuật, vì đó mới là phần khiến người ta nghỉ việc. Xem [03-team-leadership.md](/series/engineering-leadership/03-team-leadership/), [02-communication.md](/series/engineering-leadership/02-communication/) và [11-career-evolution.md](/series/engineering-leadership/11-career-evolution/).

---

## Case 4 — Scale team từ 5 lên 50 trong 18 tháng

### Bối cảnh

Product startup tại TP.HCM, SaaS quản lý vận hành cho chuỗi bán lẻ vừa (POS, tồn kho, chương trình khách hàng thân thiết). Ba năm bootstrapping, khoảng 140 cửa hàng dùng, rồi đóng Series A. Board đặt ba mục tiêu cho 18 tháng: mở hai dòng sản phẩm mới, tăng ARR gấp bốn, và đạt được mức độ ổn định đủ để bán cho chuỗi trên 200 điểm bán. Ngân sách tuyển: từ 5 lên 50 engineer.

Đội ban đầu năm người, làm cùng nhau từ 20 đến 34 tháng: **Minh** (founder kiêm CTO, viết phần lớn core, vẫn merge khoảng 30% pull request của công ty), **Tuấn** (Tech Lead trên danh nghĩa, thực chất là senior mạnh nhất về backend), **Linh** (giỏi nhất về dữ liệu và hiệu năng), **Hà** và **Khoa** (full-stack, cùng vào cách nhau hai tuần). Không có PO — Minh làm luôn. Không có quy trình viết ra: quyết định kỹ thuật diễn ra ở bàn ăn trưa, và ai cũng biết mọi thứ vì codebase một người đọc hết trong hai tuần.

Điểm quan trọng cần giữ trong đầu suốt case này: **ở quy mô 5 người, cấu hình đó không phải là sự lộn xộn — nó là lợi thế cạnh tranh.** Không họp, không tài liệu, không phê duyệt, một người có toàn bộ context sản phẩm và kỹ thuật trong đầu. Đó chính là lý do công ty sống được đến Series A. Case này nói về việc cùng một cấu hình ấy trở thành ràng buộc chính của tổ chức khi số người nhân lên mười lần.

### Triệu chứng ban đầu

Số minh hoạ, ghi theo mốc quý:

| Mốc | Headcount eng | Số team | Lead time trung vị | Tỉ lệ nghỉ việc 12 tháng | Số việc chờ Minh duyệt (trung bình/ngày) |
|---|---|---|---|---|---|
| Tháng 0 | 5 | 1 | 3 ngày | 0% | 2 |
| Tháng 3 | 9 | 1 | 4 ngày | 0% | 6 |
| Tháng 6 | 16 | 2 | 6 ngày | 8% | 11 |
| Tháng 9 | 24 | 3 | 9 ngày | 14% | 17 |
| Tháng 12 | 33 | 4 | 13 ngày | 21% | 23 |
| Tháng 15 | 42 | 5 | 15 ngày | 27% | 19 |
| Tháng 18 | 48 | 6 | 8 ngày | 24% | 6 |

Đọc bảng theo cột dọc thì thấy một câu chuyện tăng trưởng. Đọc theo hàng ngang thì thấy điều thật sự xảy ra: **headcount nhân 6,6 lần trong khi lead time xấu đi 5 lần.** Nếu ước lượng thô rằng throughput tỉ lệ nghịch với lead time ở khối lượng công việc tương đương, năng suất trên đầu người ở tháng 15 chỉ còn khoảng một phần tám so với tháng 0. Công ty trả lương cho 42 người để nhận được sản lượng của chừng 10 người thời kỳ đầu.

Ba triệu chứng phụ đi kèm, không nằm trong bảng:

- **Thời gian một câu hỏi được trả lời** tăng từ vài phút (quay sang hỏi) lên 1–2 ngày (đăng Slack, đợi người biết rảnh). Với người mới, mỗi câu hỏi chưa trả lời là một block; ở tháng 12, một engineer mới trung bình bị block 1,8 lần/tuần, mỗi lần 0,7 ngày.
- **Tỉ lệ người vào dưới 6 tháng** vượt 50% từ tháng 9 và đạt đỉnh 61% ở tháng 13.
- **Số quyết định kỹ thuật bị đảo sau khi đã bắt đầu code** tăng từ gần 0 lên 3–4 lần/tháng ở tháng 12, gần như tất cả đều do Minh đảo.

### Chẩn đoán sai ban đầu

Chẩn đoán của Minh ở tháng 9, nói nguyên văn trong một buổi họp lãnh đạo: *"Mình chậm vì đang thiếu người ở đúng chỗ. Backend đang gánh ba dòng sản phẩm. Tuyển thêm sáu backend nữa là hết nghẽn."*

Chẩn đoán này sai theo ba tầng, và mỗi tầng sai một kiểu khác nhau.

**Tầng một — nhầm capacity với throughput.** Đây là Brooks's Law ở dạng thuần khiết: thêm người vào một dự án đang chậm làm nó chậm hơn. Cơ chế cụ thể ở đây có thể đo: mỗi người mới tiêu khoảng 25–30% thời gian của một người cũ trong 8 tuần đầu, và chỉ đóng góp khoảng 30–40% sản lượng của một người thạo việc trong quý đầu. Ở tháng 6–9 công ty có 7 người đủ khả năng mentor và nhận 11 người mới trong 3 tháng — tỉ lệ mentee trên mentor là 1,6. Với tỉ lệ đó, mỗi người cũ mất khoảng 40–48% thời gian cho việc dạy, tức 7 người cũ chỉ còn năng lực tương đương 3,8 người, cộng 11 người mới ở mức 0,35 tương đương 3,9 người: tổng 7,7 so với 7 trước khi tuyển. **Tuyển 11 người, nhận thêm 0,7 đơn vị năng lực trong quý đó.** Đó không phải nghịch lý — đó là số học của việc onboarding tiêu tài nguyên từ đúng nhóm người đang tạo ra sản lượng.

**Tầng hai — nhầm nghẽn nguồn lực với nghẽn quyết định.** Cột cuối của bảng là cột quan trọng nhất và là cột duy nhất không ai theo dõi. 23 việc chờ Minh duyệt mỗi ngày với năng lực xử lý thực tế 6–8 việc/ngày nghĩa là hàng đợi tăng đơn điệu. Theo Little's Law, thời gian chờ trung bình bằng độ dài hàng đợi chia cho tốc độ phục vụ; hàng đợi này chỉ ngừng tăng khi người ta bỏ cuộc — hoặc tự quyết mà không nói, hoặc bỏ hẳn ý tưởng. Cả hai đều tệ hơn việc chờ. Thêm sáu backend vào một hệ thống nghẽn ở khâu phê duyệt chỉ làm hàng đợi dài thêm.

**Tầng ba — nhầm sự cố văn hoá với sự cố tuyển sai.** Khi tỉ lệ nghỉ việc chạm 21%, phản xạ đầu tiên là siết tiêu chuẩn tuyển. Nhưng phần lớn người nghỉ ở tháng 10–14 là người vào ở tháng 4–8, tức nhóm được onboard bởi những người cũng vừa mới vào. Họ không "không hợp văn hoá" — họ chưa bao giờ tiếp xúc với văn hoá đó. **Văn hoá truyền qua tiếp xúc, không truyền qua tài liệu.** Một người học cách công ty thật sự hành xử bằng việc quan sát người cũ xử lý tình huống có xung đột: một incident lúc nửa đêm, một lần cãi nhau về kiến trúc, một lần cắt scope trước deadline. Cần vài chục lần quan sát như vậy để hình thành phản xạ. Khi 61% tổ chức có dưới 6 tháng thâm niên, người mới học từ người mới, và thứ được nhân bản là bản sao của bản sao.

#### Chẩn đoán đúng: bốn ngưỡng chuyển pha

Trước khi vào các lựa chọn, cần đặt tên cho cấu trúc đang tạo ra hành vi trên. Tổ chức kỹ thuật không tăng trưởng trơn — nó đi qua các ngưỡng mà tại đó cùng một cách làm đột ngột hết hiệu lực.

**Ngưỡng 1 — khoảng 8 người.** Một lead mất hai năng lực cùng lúc. Thứ nhất, 1-1 chất lượng: 8 người × 45 phút mỗi hai tuần cộng thời gian chuẩn bị và theo dõi là khoảng 9 giờ/tháng, và đó là phần dễ; phần khó là giữ được trong đầu bối cảnh nghề nghiệp, động lực, mâu thuẫn ngầm của 8 người. Thứ hai, không còn đọc hết code. Khi lead thôi biết hết code, mọi phê duyệt kỹ thuật của lead chuyển từ "biết" sang "tin", mà chưa ai kịp xây cơ sở cho việc tin.

**Ngưỡng 2 — 15 đến 20 người.** Số cặp giao tiếp là n(n−1)/2: ở 8 người là 28, ở 18 người là 153. Không ai giữ nổi 153 kênh. Bắt buộc phải chia team, và ngay khi chia thì thông tin không còn đi qua bàn ăn trưa nữa. Đây là ngưỡng bắt buộc **văn bản hoá quyết định** — không phải vì tài liệu tốt, mà vì một quyết định không viết ra sẽ được tái tranh luận mỗi khi có người mới, và chi phí tái tranh luận tăng tuyến tính theo tốc độ tuyển.

**Ngưỡng 3 — 25 đến 30 người.** Ngưỡng của người sáng lập. Người có toàn bộ context không còn đủ băng thông để dùng context đó. Ở đây quyền quyết định phải rời khỏi người có nhiều thông tin nhất và chuyển sang người ở gần vấn đề nhất — một sự đánh đổi có thật: chất lượng trung bình của từng quyết định giảm, thông lượng quyết định tăng nhiều lần.

**Ngưỡng 4 — 40 đến 50 người.** Ba thứ đồng thời tới hạn: một người không quản được quá 5–7 team lead nên cần **tầng quản lý thứ hai**; chi phí trùng lặp hạ tầng giữa 5–6 team vượt chi phí nuôi một **platform team**; và không còn cách nào trả lời câu "sao anh ta lên senior mà tôi thì không" ngoài một **career ladder** chính thức, vì số cặp so sánh đã đủ lớn để mọi bất nhất trở nên nhìn thấy được.

#### Ba góc nhìn về một sự việc

Tháng 11. Nam (backend, vào tháng 5) làm module đồng bộ tồn kho đa kho, thiết kế đã được Tuấn review và duyệt. Đến buổi demo, Minh xem 4 phút rồi nói cách này sai, phải làm lại theo event log. Ba tuần công việc bị bỏ.

> **Hà:** Anh Minh, em nói thẳng một chuyện. Ba tuần của Nam mất không phải vì thiết kế sai — anh nói đúng, event log tốt hơn. Nó mất vì anh xem thiết kế ở tuần thứ ba thay vì tuần thứ nhất.
>
> **Minh:** Thì bảo Tuấn gửi anh sớm hơn.
>
> **Hà:** Tuần trước anh có 23 thứ chờ duyệt. Gửi sớm hơn thì nó nằm trong hàng đợi lâu hơn thôi. Vấn đề không phải Tuấn gửi muộn, vấn đề là mọi thứ đều phải đi qua anh. Anh đang là điểm nghẽn của công ty này, và điều đó không sửa được bằng cách anh làm nhanh hơn.
>
> **Minh:** Nhưng nếu anh không xem thì ai chịu trách nhiệm khi nó hỏng?
>
> **Hà:** Đúng câu hỏi rồi đấy. Câu trả lời là Tuấn. Nhưng Tuấn chỉ chịu được trách nhiệm đó nếu anh không đảo quyết định của cậu ấy nữa — kể cả những lần anh đúng.

Cùng sự việc, ba cách đọc:

**Nhìn từ IC (Nam).** "Tôi làm đúng quy trình, người có thẩm quyền đã duyệt, và ba tuần của tôi vẫn bị xoá." Kết luận anh ta rút ra không phải "cần thiết kế tốt hơn" mà "quy trình duyệt ở đây vô nghĩa, thứ duy nhất có giá trị là ý kiến của Minh". Hành vi thay đổi tương ứng: từ tháng 12, Nam bắt đầu nhắn riêng cho Minh trước khi làm bất cứ việc gì lớn. Điều này hợp lý với Nam và làm hàng đợi của Minh dài thêm.

**Nhìn từ Tech Lead (Tuấn).** "Tôi vừa mất thẩm quyền trước mặt team của mình." Không phải vấn đề tự ái: sau lần đó, khi Tuấn duyệt một thiết kế, không ai coi đó là đã duyệt xong. Thẩm quyền của một Tech Lead không đến từ chức danh mà từ việc quyết định của người đó có được giữ nguyên hay không. Một lần đảo công khai xoá nhiều uy tín hơn mười lần ủng hộ riêng tư.

**Nhìn từ Manager (Minh).** "Tôi thấy một lỗi kiến trúc sẽ tốn sáu tháng để sửa, và tôi có nghĩa vụ chặn nó." Đây là phần thường bị bỏ qua: **Minh đúng về mặt kỹ thuật.** Chính vì thường xuyên đúng nên anh không có tín hiệu nào cho thấy hành vi này gây hại. Đó là cấu trúc bẫy của ngưỡng 3 — cùng một hành vi tạo ra lợi thế ở quy mô 5 người (một người có toàn bộ context, quyết nhanh, quyết đúng) trở thành ràng buộc ở quy mô 30, và feedback loop báo hại lại đến rất muộn, qua chỉ số nghỉ việc, chứ không đến qua chất lượng từng quyết định.

### Các lựa chọn thực tế

Điểm rẽ là tháng 9, khi lead time chạm 9 ngày và người thứ ba nghỉ trong một quý. Ràng buộc tại thời điểm đó: tiền đã có trong tài khoản và board đã duyệt kế hoạch 50 người; hai đối thủ cùng phân khúc đang gọi vốn và ước tính có 12 tháng trước khi họ ra thị trường; hợp đồng với hai chuỗi lớn có điều khoản tính năng phải sẵn sàng trước quý sau. Không ai trong cuộc có được kết luận "vấn đề là hàng đợi quyết định" — đó là kết quả của phân tích về sau. Bốn phương án dưới đây là bốn phương án thật sự được đặt lên bàn.

**A. Giữ nguyên kế hoạch tuyển, sửa quy trình sau.** Lập luận: cửa sổ thị trường là thứ không mua lại được, còn quy trình thì sửa lúc nào cũng được. Đây là phương án mặc định của gần như mọi startup vừa gọi vốn, và nó không ngu ngốc — nó dựa trên giả định rằng tăng trưởng che được rối loạn tổ chức đủ lâu để đến vòng tiếp theo. Giả định này đúng khi sản phẩm đã có product-market fit rõ và phần lớn công việc là nhân bản thứ đã biết cách làm; nó sai khi công ty còn phải khám phá, vì khám phá đòi hỏi vòng phản hồi ngắn mà rối loạn tổ chức thì kéo dài vòng phản hồi.

**B. Hạ nhịp tuyển xuống khoảng 2 người/tháng và chấp nhận trễ lộ trình.** Lập luận: nếu năng suất trên đầu người đang giảm nhanh hơn tốc độ tăng đầu người, thì việc tuyển thêm là chuyển tiền thành nhiễu. Phương án này đòi hỏi một hành động chính trị khó: quay lại nói với board rằng con số headcount trong bản kế hoạch gọi vốn không còn là chỉ số nên dùng. Rủi ro không nằm ở kỹ thuật tổ chức mà ở niềm tin — một founder xin giảm tốc ngay quý đầu sau Series A phải có dữ liệu rất sạch, nếu không sẽ bị đọc là mất tự tin.

**C. Giữ nhịp tuyển nhưng dựng ngay tầng quản lý thứ hai và platform team.** Lập luận: nghẽn không phải số người mà là cấu trúc; hãy sửa cấu trúc và giữ đà tuyển. Vấn đề là hai thứ cần dựng đều có thời gian hình thành riêng — một EM mới cần khoảng hai quý mới thật sự hữu ích, một platform team cần một quý mới có thứ dùng được — trong khi việc rút 2 engineer mạnh khỏi vai trò IC và 4 người khỏi việc làm tính năng có hiệu lực tức thì. Nói cách khác, phương án này làm mọi thứ tệ hơn trong ngắn hạn ngay tại thời điểm đang tệ.

**D. Đẩy một dòng sản phẩm sang một ODC bên ngoài.** Lập luận: mua năng lực thực thi cho phần biên, giữ đội trong nhà nhỏ và tinh, và đảo ngược được nếu không hiệu quả. Bối cảnh Việt Nam làm phương án này khả thi hơn nhiều nơi khác vì nguồn ODC dồi dào và rẻ tương đối. Điều kiện áp dụng lại rất hẹp: phần việc phải tách sạch, có spec ổn định, không nằm trên lõi sản phẩm, và phía trong nhà phải có người đủ rảnh để làm đầu mối kỹ thuật — điều kiện cuối cùng chính là thứ đang thiếu.

### Trade-off của từng lựa chọn

| Phương án | Được | Mất | Ai chịu phần mất | Điều kiện nghiêng về |
|---|---|---|---|---|
| **A. Tuyển đúng kế hoạch** | Giữ cam kết với board; chiếm thị phần trước đối thủ; không phải giải thích việc chậm | Năng suất/đầu người tiếp tục giảm; văn hoá loãng thêm; nghỉ việc có thể vượt 30% | Nhóm người cũ (kiệt sức vì dạy) và nhóm vào sau (không được onboard) | Cửa sổ thị trường thật sự đóng trong 12 tháng và sản phẩm đã product-market fit rõ |
| **B. Hạ nhịp tuyển** | Bảo toàn năng suất/đầu người; văn hoá còn kịp truyền; giảm nghỉ việc | Trễ ít nhất một dòng sản phẩm; rủi ro mất niềm tin của board ngay ở quý đầu sau Series A | Founder (chịu áp lực board) và doanh thu năm kế tiếp | Nghỉ việc đã trên 15% hoặc tỉ lệ người dưới 6 tháng đã trên 40% |
| **C. Dựng tầng quản lý ngay** | Giải nghẽn quyết định sớm; tạo đường phát triển cho người cũ | Mất 2 engineer giỏi nhất khỏi vai trò IC; rủi ro promote sai cao khi chưa có ladder và chưa ai được huấn luyện | Chính hai người được promote, và team của người promote hỏng | Đã qua ngưỡng 25–30 và có ít nhất hai ứng viên đã từng mentor thành công |
| **D. Đẩy sang ODC** | Tăng sản lượng không làm loãng đội trong nhà; đảo ngược được | Chi phí phối hợp xuyên tổ chức; tri thức domain rời khỏi công ty; chất lượng khó kiểm soát ở phần tích hợp | Team trong nhà (gánh phần tích hợp) và khách hàng | Phần việc tách được sạch, có spec ổn định, không nằm trên lõi sản phẩm |

Không phương án nào đủ một mình. A bỏ qua cơ chế đang gây hại; B đúng về kỹ thuật tổ chức nhưng có thể giết công ty theo đường khác; C đúng hướng nhưng làm vội thì hỏng người; D chỉ áp dụng được cho phần biên mà công ty này lại chưa có phần biên nào sạch.

### Quyết định và cách thực thi

Chọn **B kết hợp C, có thứ tự và có điều kiện dừng**. Bốn việc, làm theo trình tự:

**1. Gắn tốc độ tuyển vào năng lực onboard (tháng 9).** Công thức tự đặt, số minh hoạ: mỗi engineer đủ tiêu chuẩn mentor được nhận tối đa 1 mentee cùng lúc, mentee tính trong 8 tuần. Số người mới nhận mỗi tháng không vượt số slot mentor trống. Thực tế nhịp tuyển rơi từ 4 xuống 2,5 người/tháng trong hai quý. Đây là cam kết khó nhất vì phải nói với board rằng lộ trình lùi một quý.

**2. Chuyển quyền quyết định kỹ thuật khỏi Minh (tháng 9–12).** Ba cơ chế: (a) mọi quyết định kiến trúc trên một ngưỡng phải có ADR, người quyết là Tech Lead của team, Minh có 48 giờ để phản đối bằng văn bản, quá hạn thì mặc định thông qua; (b) Minh giữ quyền phủ quyết đúng ba loại việc — mô hình dữ liệu khách hàng, bảo mật/thanh toán, và cam kết với đối tác lớn — mọi thứ khác bỏ hẳn; (c) Linh lên Staff Engineer với vai trò rõ ràng là lấp phần "chất lượng kỹ thuật xuyên team" mà Minh đang giữ, chứ không phải làm quản lý.

Điểm khó nhất không phải viết ra ba điều đó, mà là ba tháng đầu Minh phải im lặng nhìn team đi vào hai quyết định mà anh biết là dưới tối ưu. Một trong hai quyết định đó về sau đúng là phải sửa, mất 5 tuần. Đó là **giá của việc chuyển giao quyền quyết định**, và nó phải được ngân sách hoá từ trước, nếu không lần đầu trả giá sẽ bị dùng làm lý do quay lại cách cũ.

**3. Tầng quản lý thứ hai (tháng 10).** Promote Hà và Khoa lên EM. Chi tiết cách làm nằm ở phần Hậu quả, vì đây là phần sai.

**4. Platform team (tháng 13).** 4 người, do Linh dẫn về mặt kỹ thuật, phạm vi hẹp và đo được: CI/CD tự phục vụ, môi trường dev dựng trong 30 phút, service template, log và tracing tập trung. Tiêu chí thành công đặt trước: thời gian từ lúc một engineer mới ngồi xuống đến lúc pull request đầu tiên lên production giảm từ 9 ngày xuống dưới 3 ngày.

### Hậu quả

Số minh hoạ, tháng 18 so với tháng 12:

| Chỉ báo | Tháng 12 | Tháng 18 |
|---|---|---|
| Lead time trung vị | 13 ngày | 8 ngày |
| Việc chờ Minh duyệt/ngày | 23 | 6 |
| Thời gian đến pull request production đầu tiên | 9 ngày | 2,5 ngày |
| Tỉ lệ người dưới 6 tháng | 58% | 31% |
| Nghỉ việc 12 tháng | 21% | 24% (đang giảm, đỉnh 29% ở tháng 14) |
| Lộ trình sản phẩm | đúng kế hoạch | trễ 1 quý, 1 dòng sản phẩm bị cắt |

Phần đắt nhất không nằm trong bảng: **Hà thành công trong vai EM, Khoa thất bại và nghỉ việc ở tháng 16.**

Kết luận dễ nhất — và sai nhất — là "Hà hợp làm quản lý hơn". Cả hai đều là engineer mạnh, cùng thâm niên, cùng được quý mến. Khác biệt nằm ở bốn thứ, trong đó ba thứ thuộc về bối cảnh chứ không thuộc về con người.

**Khác biệt 1 — team được giao.** Hà nhận team Tồn kho: tồn tại 8 tháng, 6 người trong đó 2 người đã ở công ty trên một năm, domain ổn định, và Tuấn ở đó làm Tech Lead. Khoa nhận team Khách hàng thân thiết: lập mới hoàn toàn, 5 người đều vào trong vòng 4 tháng, domain chưa ai từng làm, không có Tech Lead. Hà quản lý một hệ thống đang chạy; Khoa vừa phải xây hệ thống vừa phải lái nó.

**Khác biệt 2 — có hay không người lấp phần kỹ thuật.** Vì team Khoa không có ai đủ trình, Khoa vẫn là người mạnh nhất về kỹ thuật trong team mình. Anh không thể buông phần kỹ thuật — buông thì team hỏng ngay tuần đó. Kết quả là Khoa làm hai việc toàn thời gian: mỗi tuần khoảng 25 giờ code cộng 20 giờ quản lý. Sau bốn tháng, cả hai việc đều dưới chuẩn, và bản thân anh kết luận rằng mình dở ở cả hai. Hà không rơi vào bẫy đó vì có Tuấn để buông tay.

**Khác biệt 3 — phạm vi thẩm quyền được giao.** Hà được giao ngân sách tuyển của team, tiếng nói quyết định trong review lương, và quyền nói không với yêu cầu từ Trang (PO). Khoa được giao cùng chức danh nhưng mọi thứ trên vẫn phải hỏi Minh — không phải vì Minh phân biệt, mà vì team của Khoa mới, Minh chưa yên tâm. Team của Khoa nhận ra điều này trong vòng sáu tuần. Một manager mà cấp dưới biết là không quyết được gì thì chỉ còn là một tầng truyền tin, và tầng truyền tin thì làm chậm mọi thứ mà không thêm giá trị nào.

**Khác biệt 4 — cách hai người bước vào vai trò.** Hà chủ động xin: trước đó đã mentor 4 người, đã tự chạy phần onboarding, và đã nói với Minh từ tháng 6 rằng mình muốn đi nhánh quản lý. Khoa được thuyết phục, với lập luận "không còn ai khác" — một lập luận nghe như tin tưởng nhưng thực chất là thông báo rằng lựa chọn này không có phương án thay thế. Quan trọng hơn: **thời điểm promote, công ty chưa có career ladder, nên không có bậc Staff để Khoa quay về.** Khi anh nhận ra mình không muốn làm quản lý ở tháng 14, con đường duy nhất còn lại là nghỉ việc. Ladder được ban hành tháng 15, muộn một tháng.

Nói cách khác: nếu đổi chỗ hai người, khả năng cao Hà cũng gãy ở team của Khoa. Điều thực sự khác nhau không phải phẩm chất cá nhân mà là **có hay không ba điều kiện chống đỡ: một Tech Lead để buông phần kỹ thuật, một phạm vi thẩm quyền thật, và một đường lui không mất mặt.**

Một hậu quả ngoài dự kiến: khi ADR trở thành bắt buộc, số quyết định bị đảo giảm từ 3–4 xuống dưới 1 lần/tháng, nhưng thời gian ra quyết định trung bình tăng từ 1,5 lên 4 ngày. Tổ chức đổi một hàng đợi tập trung dài lấy nhiều hàng đợi ngắn cộng chi phí văn bản. Đó là đánh đổi đúng ở quy mô 40 người và sẽ là đánh đổi sai ở quy mô 8 người.

### Bài học

**1. Tổ chức đi qua ngưỡng, không đi theo đường thẳng — và cách làm hết hiệu lực trước khi có dấu hiệu.** Cơ chế: chi phí phối hợp tăng theo bình phương số người trong khi năng lực điều phối của một cá nhân là hằng số; khi hai đường cắt nhau, hệ thống không suy giảm dần mà đổi chế độ. Bốn ngưỡng đáng lập kế hoạch trước: ~8 (một lead hết khả năng vừa 1-1 chất lượng vừa biết hết code), 15–20 (bắt buộc chia team và văn bản hoá quyết định), 25–30 (quyền quyết định phải rời người có nhiều context nhất), 40–50 (tầng quản lý thứ hai, platform, career ladder). Hệ quả thực hành: gắn mỗi ngưỡng với một việc phải hoàn thành **trước** khi chạm ngưỡng, vì thời gian xây năng lực quản trị là 2–3 tháng còn thời gian vượt ngưỡng khi đang tuyển 4 người/tháng chỉ là 6 tuần. Xem [09-to-chuc-va-scaling.md](/series/engineering-leadership/09-to-chuc-va-scaling/) và [03-team-leadership.md](/series/engineering-leadership/03-team-leadership/).

**2. Tốc độ tuyển phải bị chặn bởi năng lực onboard, không bởi ngân sách.** Cơ chế: onboarding tiêu tài nguyên lấy từ đúng nhóm người đang tạo ra sản lượng; khi tỉ lệ mentee trên mentor vượt khoảng 1, phần năng lực mất đi của người cũ xấp xỉ hoặc lớn hơn phần thêm vào của người mới, nên tuyển thêm làm giảm sản lượng tuyệt đối trong 1–2 quý. Ngân sách gọi vốn tạo áp lực ngược chiều vì tiền đã có sẵn và board đo tiến độ bằng headcount. Hệ quả thực hành: công bố một công thức trần tuyển (số slot mentor trống) và báo cáo nó cho board như một chỉ số vận hành, để việc hạ nhịp tuyển là dữ liệu chứ không phải lời xin lỗi. Xem [08-hiring-va-phat-trien.md](/series/engineering-leadership/08-hiring-va-phat-trien/) và [07-project-delivery.md](/series/engineering-leadership/07-project-delivery/).

**3. Văn hoá truyền qua tiếp xúc; khi tỉ lệ người mới vượt ngưỡng, thứ được nhân bản là bản sao của bản sao.** Cơ chế: hành vi tổ chức được học qua quan sát người cũ xử lý tình huống có xung đột — incident, tranh luận kiến trúc, cắt scope — chứ không qua tài liệu giá trị cốt lõi. Số lần quan sát cần thiết là hằng số sinh học, không rút ngắn được bằng văn bản. Khi trên 50% tổ chức có dưới 6 tháng thâm niên, người mới học từ người mới và độ trung thực của bản sao giảm theo mỗi thế hệ. Hệ quả thực hành: theo dõi tỉ lệ người dưới 6 tháng như một chỉ số vận hành với ngưỡng báo động 40%; phân bổ người cũ trải đều các team thay vì dồn team quan trọng; và cố ý tạo dịp cho người mới quan sát người cũ trong tình huống thật (mời dự postmortem, dự tranh luận ADR) thay vì chỉ dạy quy trình. Xem [00-nen-tang-leadership.md](/series/engineering-leadership/00-nen-tang-leadership/) và [03-team-leadership.md](/series/engineering-leadership/03-team-leadership/).

**4. Cùng một hành vi của founder là lợi thế ở quy mô 5 và là ràng buộc ở quy mô 30 — và tín hiệu báo hại đến rất muộn.** Cơ chế: giá trị của việc một người giữ toàn bộ context là quyết định nhanh và đúng, đúng đến mức không có phản hồi tiêu cực nào ở cấp từng quyết định. Chi phí lại tích ở chỗ khác: hàng đợi phê duyệt tăng đơn điệu khi nhu cầu vượt năng lực xử lý, và mỗi lần đảo quyết định của một Tech Lead trước mặt team sẽ chuyển thẩm quyền của người đó về không, khiến mọi việc lại chảy về founder — một vòng lặp tự củng cố. Hệ quả thực hành: đo số việc chờ một người duyệt như một chỉ số hệ thống; thu hẹp quyền phủ quyết xuống một danh sách viết ra được (2–4 loại việc); dùng cơ chế phản đối có thời hạn thay cho phê duyệt; và ngân sách hoá trước cái giá của 1–2 quyết định dưới tối ưu như học phí chuyển giao. Xem [05-technical-leadership.md](/series/engineering-leadership/05-technical-leadership/) và [12-anti-patterns.md](/series/engineering-leadership/12-anti-patterns/).

**5. Promote thành công phụ thuộc vào ba điều kiện của bối cảnh nhiều hơn phẩm chất của người được promote.** Cơ chế: một EM mới chỉ chuyển được trọng tâm sang con người nếu có ai đó gánh phần kỹ thuật — nếu không, người đó làm hai việc toàn thời gian và dưới chuẩn ở cả hai; thẩm quyền hình thức không kèm quyền thật (ngân sách, tiếng nói trong lương, quyền từ chối scope) bị team nhận ra trong 4–6 tuần và biến manager thành tầng truyền tin; và khi chưa có career ladder có bậc Staff, một người phát hiện mình không hợp quản lý chỉ còn cửa nghỉ việc, biến một quyết định thử nghiệm thành quyết định một chiều. Hệ quả thực hành: trước mỗi lần promote, kiểm ba điều kiện đó như checklist chặn; ghép EM mới với team đã ổn định thay vì team mới lập; công bố ladder **trước** đợt promote đầu tiên; và nói rõ bằng lời rằng quay lại IC trong 12 tháng là kết quả chấp nhận được. Xem [11-career-evolution.md](/series/engineering-leadership/11-career-evolution/), [08-hiring-va-phat-trien.md](/series/engineering-leadership/08-hiring-va-phat-trien/) và [02-communication.md](/series/engineering-leadership/02-communication/).

---

## Case 5 — Incident production nghiêm trọng

### Bối cảnh

Công ty là một ví điện tử Việt Nam, giấy phép trung gian thanh toán, khoảng 45 engineer chia sáu team: Payment Core, Wallet, Merchant, Growth, Mobile, Platform. Mỗi ngày xử lý khoảng 1,2 triệu giao dịch; nạp tiền và thanh toán hoá đơn chiếm khoảng 38% khối lượng nhưng phần lớn doanh thu phí (số minh hoạ).

Các vai khi sự cố xảy ra:

- Minh — CTO, gia nhập 14 tháng trước từ một công ty công nghệ lớn; kỹ thuật mạnh, phản xạ nhanh, quen môi trường mà mọi thứ đo được và trả lời được trong vài phút.
- Hà — Engineering Manager phụ trách Payment Core và Wallet, 12 người. Tuấn — Tech Lead Payment Core.
- Linh — Staff Engineer, không thuộc team cố định, được kỳ vọng là người "đi ngang" giữa các hệ thống. Trang — Product Owner mảng nạp tiền và thanh toán hoá đơn.
- Quân — Senior Backend, hiểu connection pool và tầng data access sâu nhất công ty. Duy — junior, vào công ty 5 tháng, team Platform. Vy — Senior Backend team Wallet. Sơn — SRE duy nhất, kiêm on-call hạ tầng.

Hai đặc điểm của tổ chức này, cả hai là hệ quả của tăng trưởng nhanh chứ không phải của sự cẩu thả.

**Thứ nhất, hệ thống đã vượt ngưỡng mà một người giữ được toàn bộ mô hình trong đầu.** Ba năm trước Tuấn biết mọi dòng code chạy trong production; bây giờ có 23 service, 4 database cluster, hai tầng cache, ba tích hợp ngân hàng. Nhưng tổ chức vẫn vận hành theo giả định cũ: "người review sẽ hiểu thứ mình review".

**Thứ hai, các lớp phòng vệ còn nguyên trên giấy nhưng đã mục.** Code Review vẫn bắt buộc, staging vẫn tồn tại, alert vẫn có dashboard, runbook vẫn nằm trong wiki. Với auditor bên ngoài, tổ chức này có đủ mọi thứ cần có — và đó chính xác là dạng tổ chức mà sự cố nghiêm trọng xảy ra: không phải nơi thiếu quy trình, mà nơi quy trình đã rỗng ruột và không ai biết.

Ràng buộc: fintech có giấy phép, nên downtime luồng nạp tiền kéo theo nghĩa vụ báo cáo với đối tác ngân hàng, khiếu nại khách hàng có tiền treo, và rủi ro truyền thông.

Sự cố kéo dài 4 giờ 12 phút, bắt đầu 23h40 một tối thứ Ba. Lỗi kỹ thuật tầm thường: một tham số connection pool bị đổi sai. Thứ đáng phân tích là vì sao nó xuyên qua bốn lớp phòng vệ, và vì sao sáng hôm sau tổ chức suýt sa thải người ít trách nhiệm nhất trong chuỗi.

### Triệu chứng ban đầu

Cột quan trọng nhất trong timeline dưới đây không phải "ai làm gì" mà là **thông tin nào chưa ai biết tại mốc đó** — vì mọi quyết định sai trong đêm đều hợp lý với thông tin có sẵn lúc ấy.

```
23h40  Deploy định kỳ team Platform hoàn tất. Trong batch có PR #4471 — đổi
       connection pool payment-core từ max_pool_size 80 xuống 20. Tác giả: Duy.
       CHƯA AI BIẾT: thay đổi này chạm payment-core. PR nằm trong repo config
       chung, tiêu đề "chore: chuẩn hoá pool config theo template".

23h41  Deploy xanh, health check pass. Traffic đêm ~9% peak, 20 connection thừa.
       CHƯA AI BIẾT: đây là ổn định giả, chỉ ổn vì tải thấp.

23h47  Batch đối soát cuối ngày của ngân hàng đối tác khởi động, mở nhiều
       connection dài. CHƯA AI BIẾT: batch dùng chung pool với payment-core.

23h49  Pool cạn. Request nạp tiền xếp hàng. p99 từ 240ms lên 9s. Chưa lỗi.
       CHƯA AI BIẾT: không alert nào bắn — alert "pool saturation" đã tắt 3 tháng.

23h52  Timeout tầng trên cắt. Lỗi nạp tiền 34%. Alert "error rate > 5%" bắn.
       Sơn (SRE on-call) nhận page.
       CHƯA AI BIẾT: nguyên nhân. Alert chỉ nói "error rate cao ở payment-core".

23h55  Sơn xem dashboard: CPU, memory, DB CPU bình thường. Kết luận "không phải
       hạ tầng". Gọi Quân.
       CHƯA AI BIẾT: dashboard không có panel connection pool — gỡ cùng đợt
       tắt alert.

23h58  Quân online, log đầy "timeout acquiring connection". Giả thuyết đầu
       tiên: database chậm.
       CHƯA AI BIẾT: database hoàn toàn khoẻ; client không có connection để
       dùng, chứ không phải server không trả lời.

00h05  Quân mở runbook "Payment Core - DB degradation" và làm theo.
       CHƯA AI BIẾT: runbook viết cho kiến trúc 8 tháng trước; bước failover
       trỏ tới host đã xoá.

00h12  Lỗi lên 61%. Thanh toán hoá đơn fail theo vì gọi sang payment-core.
       CHƯA AI BIẾT: chưa ai tuyên bố Incident Commander. Quân mặc định vừa
       chỉ huy vừa tự tay sửa.

00h18  Hà (EM) online, hỏi "cần gì không". Quân: "đang xử lý". Hà rút.
       CHƯA AI BIẾT: câu "đang xử lý" đã thành lá chắn ngăn mọi trợ giúp.

00h24  Linh (Staff Engineer) online, không hỏi ai, tự mở nhánh riêng: nghi DB
       quá tải, scale up instance primary.
       CHƯA AI BIẾT: Quân không biết Linh làm gì; Linh không biết Quân đang
       chuẩn bị failover.

00h30  Minh (CTO) vào channel, hỏi dồn: đang bị gì, bao nhiêu user, bao giờ
       xong, báo đối tác chưa, ai chịu trách nhiệm. Tuấn trả lời từng câu.
       CHƯA AI BIẾT: không ai có số impact thật; con số đưa ra chỉ là ước
       lượng của Sơn nhìn dashboard.

00h31– Quân và Tuấn dành phần lớn 26 phút trả lời Minh thay vì điều tra.
00h57  CHƯA AI BIẾT: chi phí thật của đoạn này — không ai đo được.

01h05  Scale DB của Linh xong, không cải thiện. Linh báo channel lần đầu.
       Quân: "ơ ai scale DB?"
       BÂY GIỜ MỚI BIẾT: có hai nhánh song song — sau khi đã mất 41 phút.

01h20  Trang (PO) hỏi có nên bật banner thông báo trên app không. 15 phút
       không ai trả lời.
       CHƯA AI BIẾT: truyền thông với người dùng cũng là một phần của
       incident, và nó không có chủ.

01h35  Vy đề xuất restart payment-core. Restart. Hồi phục 3 phút rồi sập lại.
       CHƯA AI BIẾT: restart tạo lại pool, đủ dùng tới khi batch chiếm lại
       connection. Manh mối lớn nhất đêm đó, không ai đọc ra.

02h10  Tuấn đề nghị rollback deploy 23h40. Quân phản đối: "deploy đó không
       đụng payment-core". Cả hai đều tin thế.
       CHƯA AI BIẾT: PR #4471 nằm trong repo config chung, không ai map được
       nó sang payment-core.

02h48  Sơn dựng tay dashboard mới, thêm panel connection pool.
       BÂY GIỜ MỚI BIẾT: active connections = 20/20, đứng yên suốt.

02h55  Quân: "20 là số gì? Mình để 80 mà". Grep lịch sử commit repo config,
       ra PR #4471, merge 6 tiếng trước.

03h18  Tăng max_pool_size lên 120 để có biên, deploy hotfix.

03h26  Hotfix lên production. Error rate bắt đầu giảm.

03h40  Error rate dưới 2%. Nhưng Linh vẫn đang chạy script re-index ở nhánh
       riêng và Vy vừa restart thêm một node.
       CHƯA AI BIẾT: hành động nào thực sự cứu hệ thống — ba thứ xảy ra
       trong 14 phút.

03h52  Tuyên bố hồi phục. Tổng 4 giờ 12 phút.
       CHƯA AI BIẾT: và sẽ không bao giờ biết chắc, vì không ai ghi thứ tự
       hành động nên không thể quy nhân quả.
```

Thứ đập vào mắt không phải là ai dốt — Quân, Linh, Sơn đều giỏi — mà là: **từ lúc hệ thống hỏng tới lúc có người nhìn thấy con số 20/20 mất 3 giờ 8 phút; từ đó tới lúc sửa xong mất 18 phút.** Toàn bộ chi phí nằm ở giai đoạn không ai nhìn đúng chỗ — bài toán về hệ thống quan sát và cấu trúc phối hợp, không phải về năng lực kỹ thuật.

### Chẩn đoán sai ban đầu

Sáng hôm sau, tổ chức tự đưa ra ba chẩn đoán. Cả ba sai theo cùng một kiểu: dừng ở lớp phòng vệ cuối cùng bị xuyên thủng thay vì đếm tất cả các lớp.

**"Lỗi do Duy đổi config."** Nơi phần lớn tổ chức dừng lại, vì nó khép kín về tâm lý: một hành động, một người, một hậu quả. Đúng về sự kiện, sai về cơ chế. Hỏi ngược: nếu Duy không tồn tại, hệ thống này có an toàn không? Không. Một thay đổi config sai kiểu này có thể đến từ bất kỳ ai trong 45 người; xác suất để suốt một năm không ai đổi sai một tham số là gần bằng không. Thiết kế tổ chức phải giả định thay đổi sai **sẽ** xảy ra; câu hỏi thật là vì sao nó không bị chặn.

**"Review lỏng lẻo, cần thêm một tầng approval."** Được lãnh đạo ưa thích vì tạo ra hành động cụ thể. Nhưng PR #4471 **đã** qua review; vấn đề là reviewer approve thứ mình không hiểu, và thêm một tầng vào hệ thống mà tầng hiện tại đang rỗng thì được hai tầng rỗng cộng độ trễ. Anti-pattern kinh điển: **thêm quy trình mà không kiểm tra quy trình hiện có có hoạt động không.** Xem [12-anti-patterns.md](/series/engineering-leadership/12-anti-patterns/).

**"On-call chậm, cần training thêm."** Sai tinh vi nhất. On-call không chậm — họ mất 3 giờ vì không có dữ liệu để nhìn. Sơn kiểm tra CPU, memory, DB đúng quy trình, đúng runbook; cái thiếu là panel connection pool đã bị gỡ. Training người nhìn dashboard không có dữ liệu thì họ vẫn không thấy gì, chỉ nhìn nhanh hơn.

**Chẩn đoán đúng: Swiss Cheese Model.** Mô hình của James Reason (nguồn công khai, dùng rộng rãi trong hàng không và y tế): mỗi lớp phòng vệ là một lát phô mai có lỗ vì không lớp nào hoàn hảo; tai nạn xảy ra khi các lỗ tình cờ thẳng hàng. Đêm đó bốn lớp đều có lỗ đúng chỗ:

| Lớp phòng vệ | Đáng lẽ chặn được gì | Vì sao có lỗ | Tồn tại bao lâu |
|---|---|---|---|
| Code Review | Nhận ra pool 20 quá thấp cho payment-core | Reviewer thuộc team Platform, không biết payment-core dùng repo config này; approve vì "trông đúng template" | ~1 năm |
| Staging | Test tải làm cạn pool trước khi lên production | Đặt cứng max_pool_size 200, không đọc repo config; traffic bằng 2% production | Từ ngày dựng staging |
| Alert | Cảnh báo pool saturation lúc 23h49, sớm 3 phút và chỉ đúng nguyên nhân | Tắt vì "quá ồn" — bắn 40 lần/tuần, phần lớn false positive (số minh hoạ) | 3 tháng |
| Runbook | Hướng dẫn on-call kiểm tra pool | Mô tả kiến trúc cũ, bước failover trỏ tới host đã xoá | 8 tháng |

**Không lỗ nào là kết quả của sự cẩu thả.** Reviewer approve vì trong văn hoá tổ chức, từ chối một PR mình không hiểu bị xem là làm chậm người khác. Staging khác production vì đọc config từ repo chung sẽ kéo theo cả secret. Alert bị tắt vì nó ồn thật. Runbook cũ vì viết runbook không bao giờ nằm trong sprint. Bốn quyết định hợp lý cục bộ cộng lại thành 4 giờ 12 phút — và đó là lý do "tìm người có lỗi" là phương pháp kém: không ai ở vị trí nhìn thấy cả bốn lỗ.

**Góc nhìn ba tầng: chuyện alert bị tắt ba tháng trước.** Khác biệt không nằm ở ai đúng mà ở chỗ mỗi vai nhìn thấy phần nào của hệ thống.

*Nhìn từ IC (Quân, Senior BE):* "Alert đó bắn 40 lần một tuần, 38 lần vô nghĩa. Tôi bị đánh thức lúc 3h sáng vì một spike 5 giây rồi tự hết. Tôi tắt để còn ngủ mà làm việc." Hợp lệ: alert fatigue là chi phí thật, đo được bằng số lần bị đánh thức và tỷ lệ nghỉ việc của người on-call. IC là người duy nhất trả chi phí đó bằng cơ thể mình.

*Nhìn từ Tech Lead (Tuấn):* "Tắt alert là quyết định kiến trúc về khả năng quan sát, không phải thao tác vận hành. Lẽ ra phải kèm một trong ba thứ: sửa ngưỡng, đổi sang alert theo xu hướng, hoặc ghi nhận công khai rằng ta chấp nhận mù ở vùng này đến bao giờ. Lỗi của tôi là không có cơ chế nào bắt việc tắt alert để lại dấu vết." Tech Lead thấy thứ IC không thấy: một hành động cục bộ đã đổi tính chất hệ thống.

*Nhìn từ Manager (Hà, EM):* "Alert ồn 40 lần/tuần nhiều tuần liền mà không ai sửa là dấu hiệu team không có capacity cho công việc vận hành. Sprint nào cũng full feature. Tôi biết và tôi không đổi. Nếu tôi có mặt lúc Quân tắt alert, tôi cũng đồng ý tắt, vì tôi cũng không có người để sửa." Manager thấy lỗ hổng sinh ra ở tầng phân bổ nguồn lực, tức tầng Process trong chuỗi Business Goal → People → Process → Technology.

Ba góc nhìn là ba lát cắt của cùng một nguyên nhân; postmortem dừng ở lát cắt thứ nhất sẽ đưa ra hành động sai. Xem [06-incident-va-metrics.md](/series/engineering-leadership/06-incident-va-metrics/).

### Các lựa chọn thực tế

**Nhóm A — trong đêm, lúc 00h30, đã 50 phút mà chưa có giả thuyết đúng.** Người ra quyết định thực tế là Tuấn, dù không ai chính thức trao quyền đó.

- **A1. Giữ nguyên.** Quân tiếp tục vừa chỉ huy vừa sửa.
- **A2. Tuyên bố Incident Commander và tách vai:** một người chỉ huy và không chạm bàn phím, một người điều tra, một scribe, một người lo communication.
- **A3. Rollback mù toàn bộ deploy 23h40** — không cần hiểu nguyên nhân, chỉ đưa hệ thống về trạng thái đã biết là tốt.
- **A4. Escalate:** gọi thêm Vy, Linh, mở war room, huy động tối đa người.
- **A5. Degrade có kiểm soát:** chủ động tắt luồng nạp tiền, hiện thông báo bảo trì, điều tra trong trạng thái không có traffic gây nhiễu.

**Nhóm B — sáng hôm sau, khi email ban lãnh đạo nêu đích danh Duy.** Người ra quyết định thực tế là Hà.

- **B1. Im lặng,** để quy trình kỷ luật chạy. Lựa chọn mặc định ở phần lớn tổ chức.
- **B2. Bảo vệ Duy bằng lý lẽ đạo đức:** đổ lỗi cho junior là bất công.
- **B3. Bảo vệ Duy bằng lý lẽ chi phí hệ thống:** sa thải Duy sẽ tạo ra hành vi giấu thay đổi trong toàn tổ chức, chi phí đó lớn hơn nhiều lần chi phí sự cố này.
- **B4. Nhận trách nhiệm thay:** Hà tự làm bia đỡ đạn để chấm dứt cuộc truy tìm.
- **B5. Hoãn mọi quyết định nhân sự tới sau postmortem.** Không bảo vệ ai, chỉ yêu cầu thứ tự đúng: dữ liệu trước, quyết định sau.

### Trade-off của từng lựa chọn

| Lựa chọn | Được gì | Mất gì | Ai chịu phần mất | Nghiêng về khi |
|---|---|---|---|---|
| A1. Giữ nguyên | Không tốn chi phí chuyển đổi giữa lúc cháy | Người điều tra bị ngắt liên tục; không ai nắm bức tranh tổng; nhánh song song không bị phát hiện | Toàn hệ thống, và cá nhân Quân | Sự cố nhỏ, dưới 20 phút |
| A2. Tuyên bố IC, tách vai | Một điểm hội tụ thông tin; phát hiện nhánh song song ngay; có timeline để phân tích sau | Mất 5–10 phút thiết lập; IC phải chịu được việc không tự tay sửa | Cảm giác kiểm soát của người giỏi nhất | Sự cố quá 20 phút, hoặc từ 3 người trở lên đang thao tác |
| A3. Rollback mù | Nhanh nhất về kỳ vọng | Sai hướng thì mất thời gian và làm nhiễu giả thuyết | Người dùng | Có thay đổi gần đây, reversible, chi phí thử thấp |
| A4. Escalate | Nhiều giả thuyết hơn | Chi phí phối hợp tăng phi tuyến; không có IC thì thêm người là thêm nhánh song song — đúng thứ xảy ra lúc 00h24 | Chất lượng điều tra | Đã có cấu trúc chỉ huy rõ |
| A5. Degrade có kiểm soát | Dừng chảy máu; loại nhiễu để điều tra sạch | Downtime tự nguyện, phải báo đối tác; cần quyền mà kỹ sư trực đêm không có | Doanh thu ngắn hạn | Trạng thái nửa vời hại hơn dừng hẳn — với fintech, giao dịch treo tệ hơn giao dịch bị từ chối |
| B1. Im lặng | Không tốn vốn chính trị của người quản lý | Tổ chức học được rằng thay đổi rủi ro thì nên giấu; MTTR sau đó tăng | Tổ chức, trong 12–24 tháng, dưới dạng chi phí không nhìn thấy | Không bao giờ, nếu còn muốn có dữ liệu thật |
| B2. Lý lẽ đạo đức | Đúng về giá trị, dễ nói | Không thuyết phục người đang chịu áp lực từ hội đồng quản trị; dễ bị đọc là bao che | Người bảo vệ, mất uy tín để lần sau nói tiếp | Tổ chức đã tin Blameless |
| B3. Lý lẽ chi phí hệ thống | Đúng ngôn ngữ người ra quyết định; đổi tranh luận từ "công bằng" sang "cái nào đắt hơn" | Đòi hỏi định lượng được và chịu được phản biện | Người nói, nếu định lượng ẩu | Người đối diện tư duy bằng chi phí và rủi ro |
| B4. Nhận trách nhiệm thay | Chấm dứt cuộc truy tìm ngay | Giữ nguyên giả định sai rằng sự cố phải có một người có lỗi | Người nhận thay, và lần sau không còn ai để nhận | Cần mua thời gian trong một cuộc họp |
| B5. Hoãn quyết định nhân sự | Thiết lập thứ tự đúng; không đối đầu trực diện | Chỉ trì hoãn nếu áp lực đủ mạnh | Duy, sống trong bất định thêm vài ngày | Bước đầu trước khi dùng B3 |

Trade-off cốt lõi nhóm A là **Tốc độ cá nhân vs Khả năng phối hợp**: một kỹ sư giỏi tự xử lý luôn nhanh hơn trong 15 phút đầu và luôn chậm hơn sau 45 phút, vì chi phí của việc không ai khác nắm trạng thái tăng theo thời gian còn lợi thế tốc độ thì không. Nhóm B là **Chi phí hiện tại vs Chi phí ẩn tương lai**: sa thải Duy tốn 0 đồng hôm nay và rất đắt về sau, nhưng phần đắt đó không xuất hiện trên báo cáo nào — đó là lý do nó hấp dẫn.

### Quyết định và cách thực thi

**Phần kỹ thuật: mitigate trước, hiểu sau.**

Nguyên tắc mà tổ chức này thiếu: **trong incident, mục tiêu là khôi phục dịch vụ, không phải hiểu nguyên nhân.** Hai mục tiêu bị nhầm là một vì với kỹ sư, hiểu nguyên nhân là con đường tự nhiên tới khôi phục — nhưng hiểu nguyên nhân mất 3 giờ, rollback mất 6 phút.

1. **Phút 0–5: tuyên bố.** "Tôi tuyên bố incident. Tôi là Incident Commander cho tới khi bàn giao." Không cần xin phép; IC là vai điều phối, không phải vai giỏi nhất.
2. **Phút 5–10: kiểm tra thay đổi gần đây trước mọi giả thuyết khác.** Câu hỏi đầu tiên là "có gì thay đổi trong 24 giờ qua", không phải "cái gì hỏng". Có sẵn trang liệt kê thay đổi kèm cột "service bị ảnh hưởng" thì PR #4471 lộ ra ở phút thứ 8 thay vì phút thứ 190.
3. **Phút 10–20: mitigate.** Rollback nếu reversible, degrade nếu không, và nói trước tiêu chí thất bại: "nếu 10 phút nữa không cải thiện thì chuyển sang Y."
4. **Toàn thời gian: một scribe** ghi timeline theo mốc phút, không làm gì khác. Và mọi thao tác production phải đăng ký trong channel trước khi làm — nếu Linh làm thế lúc 00h24, tổ chức tiết kiệm 41 phút.

**Phần con người, đoạn 1: Minh hỏi dồn trong channel.**

Đây là dạng nhiễu từ trên xuống nguy hiểm nhất vì động cơ của nó chính đáng — CTO có nghĩa vụ trả lời hội đồng quản trị và đối tác ngân hàng. Nhưng mỗi câu hỏi là một lần ngắt tư duy người đang debug, chi phí khôi phục tập trung 10–15 phút mỗi lần.

Phiên bản Tuấn xử lý sai — chính là thứ đã xảy ra:

> **Minh (00h30):** Đang bị gì vậy mọi người? Bao nhiêu user bị ảnh hưởng? Bao giờ xong? Đã báo bên ngân hàng chưa?
> **Tuấn:** Dạ đang điều tra ạ. Hiện tại chưa rõ nguyên nhân.
> **Minh:** Chưa rõ là chưa rõ thế nào? Đã loại trừ được cái gì rồi?
> **Tuấn:** Dạ đã loại DB, đã loại hạ tầng. Đang xem log ạ.
> **Minh:** Log nói gì? Gửi tôi xem. Và bao giờ xong thì cho tôi con số, tôi phải trả lời anh Đ. bên ngân hàng.
> **Tuấn:** Dạ em gửi ngay. Em ước chừng 30 phút nữa ạ.
> **Minh:** Chắc không?
> **Tuấn:** Dạ... em cố ạ.

Ba lỗi. Một, Tuấn trả lời từng câu, biến mình thành cổng thông tin nhưng không ngăn được việc Quân cũng đọc và cũng bị ngắt. Hai, con số "30 phút" đưa ra vì bị hỏi chứ không vì biết, rồi thành cam kết Minh mang đi hứa với đối tác. Ba, Tuấn không chạm tới nhu cầu thật của Minh nên Minh phải hỏi tiếp.

Phiên bản đúng:

> **Minh (00h30):** Đang bị gì vậy mọi người? Bao nhiêu user bị ảnh hưởng? Bao giờ xong?
> **Tuấn:** Anh Minh, em là Incident Commander cho sự cố này. Em cập nhật anh 15 phút một lần trong thread riêng #inc-2411-cto, kể cả khi không có gì mới. Channel này em xin giữ cho team điều tra tập trung.
> **Tuấn:** Trạng thái 00h30: nạp tiền và thanh toán hoá đơn lỗi khoảng 60%, các luồng khác bình thường. Ước tính 4.000 giao dịch bị ảnh hưởng từ 23h52 — độ tin cậy thấp, em đang cho xác minh.
> **Tuấn:** Về "bao giờ xong": chưa có nguyên nhân nên em chưa thể cho ETA. Thứ em cam kết được: 00h45 em báo anh một trong hai điều — hoặc đã tìm ra nguyên nhân, hoặc em chuyển sang mitigate mù bằng rollback deploy gần nhất. Anh cho em hỏi: anh cần con số để trả lời ai, hạn chót mấy giờ?
> **Minh:** Anh Đ. bên ngân hàng, trước 6h sáng. Cần số giao dịch fail và số tiền treo.
> **Tuấn:** Rõ. Em giao Trang chuẩn bị song song, không lấy người khỏi việc điều tra. 6h sáng anh sẽ có.
> **Minh:** Được. Tôi rút khỏi channel này. Có gì gọi trực tiếp.

Khác biệt nằm ở cơ chế, không phải sự lễ phép. Tuấn (a) tuyên bố vai để có thẩm quyền điều phối cả cấp trên, (b) đổi từ trả lời phản ứng sang nhịp cập nhật cố định — thứ giảm lo lắng hiệu quả hơn bất kỳ câu trả lời nào, (c) từ chối đưa ETA nhưng thay bằng cam kết kiểm chứng được vào 00h45, (d) hỏi ngược để tìm nhu cầu thật và tách nó thành luồng riêng. Xem [02-communication.md](/series/engineering-leadership/02-communication/).

Rào cản lớn nhất của phiên bản đúng, ở Việt Nam, không phải kỹ năng mà là việc một Tech Lead phải nói câu đó với CTO. Điều đó cần một thoả thuận ký lúc trời yên; chưa có thì đừng thử lần đầu lúc đang cháy.

**Phần con người, đoạn 2: sáng hôm sau và chuyện suýt sa thải Duy.**

8h15, email ban điều hành gửi CTO và các trưởng bộ phận, tiêu đề "Sự cố đêm 12/11 — yêu cầu giải trình". Câu đầu tiên: "Ai deploy tối qua?" Trong phần thân, tên Duy kèm mã PR. Đến trưa, một phó tổng đề xuất "xử lý dứt điểm để làm gương".

Người can thiệp là Hà, chọn B5 trước rồi B3 sau — trình tự này quan trọng. Trong cuộc họp 14h, Hà không bảo vệ Duy, chỉ nói: "Em đề nghị hoãn mọi quyết định nhân sự đến sau postmortem. Chúng ta chưa biết chuỗi nguyên nhân, và một quyết định nhân sự sai thì không rút lại được, còn hoãn ba ngày thì rút lại được." Đây là lập luận về tính bất đối xứng của rủi ro, không phải về Duy, nên khó bị phản đối: nó không đòi ai đổi quan điểm.

Sau khi postmortem cho ra bốn lớp phòng vệ hỏng, Hà mới dùng lập luận chi phí:

"Nếu sa thải Duy, thông điệp mà 44 người còn lại nhận được không phải 'hãy cẩn thận hơn', mà là: nếu thay đổi của bạn có thể liên quan tới một sự cố, đừng để ai biết. Ba hành vi sẽ xuất hiện trong vòng một tháng. Một, người ta gộp thay đổi rủi ro vào PR lớn để nó chìm đi. Hai, trong incident không ai tự nói 'tôi vừa đổi cái này' — mà đó chính là câu tiết kiệm được 3 giờ đêm qua. Ba, người ta ngừng đề xuất cải thiện ở những chỗ mình không chắc.

Cái chúng ta cần mua là xác suất người tiếp theo lên tiếng sớm; sa thải Duy là cách chắc chắn nhất để mua điều ngược lại. Chi phí một sự cố là con số chúng ta thấy được; chi phí của việc không ai dám nói là con số chúng ta sẽ không bao giờ thấy, và nó lớn hơn."

Lập luận thuyết phục vì ba lý do. Nó nói bằng ngôn ngữ chi phí và rủi ro — ngôn ngữ ban lãnh đạo dùng — chứ không bằng ngôn ngữ công bằng. Nó nêu ba hành vi quan sát được, tức có thể sai và có thể kiểm chứng, nên đáng tin hơn một tuyên bố giá trị. Và nó neo vào con số mọi người vừa đau, rồi chỉ ra phần lớn 4 giờ 12 phút đó là chi phí của thiếu thông tin chứ không phải của thiếu kỷ luật.

Minh chốt: "Chúng ta không kỷ luật ai. Nhưng tôi muốn thấy bốn lỗ hổng kia được bịt, có tên người sở hữu và có hạn."

**Cuộc nói chuyện với Duy.** Điểm khó: không trấn an theo kiểu xoá trách nhiệm — như thế Duy không học được gì và cũng không tin — nhưng cũng không để Duy mang toàn bộ gánh nặng.

> **Hà:** Duy, em ngồi đi. Em có ngủ được không?
> **Duy:** Dạ không ạ. Em xin lỗi anh. Em không biết config đó ảnh hưởng payment-core, em thấy template ghi 20 nên chuẩn hoá theo.
> **Hà:** Anh đọc PR rồi. Anh nói ba điều. Thứ nhất: em sẽ không bị kỷ luật, không bị ghi vào Performance Review, và chuyện này đã chốt ở cấp trên anh. Đây là sự thật, không phải anh an ủi em.
> **Duy:** Dạ... nhưng rõ ràng là do em ạ.
> **Hà:** Đó là điều thứ hai. Thay đổi của em là một trong bốn thứ phải cùng hỏng mới ra sự cố này. Reviewer approve mà không hiểu hệ thống, staging không giống production, alert bị tắt ba tháng trước khi em còn chưa vào công ty, runbook sai từ tám tháng trước — không cái nào là lỗi em. Em là lát phô mai cuối cùng, không phải nguyên nhân. Không có em thì tháng sau là người khác, và anh mới là người chịu trách nhiệm vì để bốn cái lỗ đó nằm đó.
> **Duy:** Dạ.
> **Hà:** Điều thứ ba là phần của em. Khi em đổi một tham số vận hành mà không tự trả lời được câu "service nào đọc file này", em dừng lại và hỏi. Không phải vì em phải biết mọi thứ — không ai biết — mà vì chính câu hỏi đó là tín hiệu em đang đứng ngoài vùng mình nhìn thấy.
> **Duy:** Dạ. Em có nên xin lỗi cả team trong postmortem không anh?
> **Hà:** Không, postmortem không có xin lỗi. Nhưng anh muốn em trình bày phần timeline của thay đổi đó — không phải để nhận lỗi, mà vì em hiểu rõ nhất vì sao một người ở vị trí em lại thấy thay đổi đó là an toàn.

Giao Duy trình bày là quyết định có chủ đích: nó chuyển Duy từ vị trí bị cáo sang vị trí người cung cấp dữ liệu, và buộc tổ chức nghe câu quan trọng nhất — "tôi thấy thay đổi này an toàn vì..." — mô tả của lỗ hổng, từ chính người rơi qua nó.

**Postmortem Blameless,** bốn quy tắc nói ra đầu buổi: (1) không dùng tên người kèm động từ chỉ lỗi, chỉ mô tả hành động và bối cảnh sinh ra nó; (2) mọi câu "lẽ ra phải" đổi thành "điều gì làm cho X trông hợp lý lúc đó", vì hindsight bias khiến mọi quyết định quá khứ trông ngu ngốc; (3) mỗi action item có một chủ và một ngày; (4) postmortem không đưa khuyến nghị nhân sự.

### Hậu quả

Mọi con số dưới đây là số minh hoạ.

**Chi phí trực tiếp.** Khoảng 11.400 giao dịch fail, trong đó 340 giao dịch treo tiền phải đối soát tay trong 3 ngày. Mất khoảng 190 triệu doanh thu phí cộng 60 triệu chi phí đối soát và chăm sóc khách hàng. Một báo cáo giải trình gửi đối tác ngân hàng; không bị phạt nhưng có ghi nhận trong hồ sơ tuân thủ.

**Chi phí gián tiếp, lớn hơn.** 9 người mất trọn một đêm và dưới 50% năng suất hai ngày sau. Tuấn mất 3 tuần điều phối khắc phục, đẩy một milestone của Trang trễ 2 sprint.

**Điều đã không xảy ra.** Duy không bị sa thải. Giá trị của quyết định đó xuất hiện 7 tuần sau: một senior team Merchant, sau khi deploy thấy latency nhích nhẹ, tự viết vào channel "tôi vừa deploy X lúc 14h05, nếu ai thấy bất thường thì đây có thể là nguyên nhân". Hành vi chủ động khai báo thay đổi khi chưa ai hỏi là thứ không tồn tại trong tổ chức này trước tháng 11. Cơ chế đằng sau là Psychological Safety: giá trị vận hành cụ thể, không phải giá trị đạo đức trừu tượng. Xem [03-team-leadership.md](/series/engineering-leadership/03-team-leadership/).

**Thay đổi hệ thống trong 90 ngày,** mỗi lỗ một người sở hữu: (1) CODEOWNERS cho repo config, thay đổi tham số vận hành phải có approve của owner service bị ảnh hưởng, kèm script tự comment danh sách service đọc file đó — Linh; (2) staging đọc cùng nguồn config với production, cộng load test hàng đêm ở 30% tải — Sơn; (3) alert đang tắt phải có ngày tự bật lại — Quân; (4) runbook hạn 90 ngày, quá hạn dashboard hiện đỏ — Hà. Cộng thêm quy trình Incident Command chính thức (IC, scribe, comms lead) và trang "Recent Changes".

**Chỉ số quý sau.** MTTR trung vị giảm từ 71 xuống 26 phút; số lần "hai người thao tác production mà không biết nhau" về 0; tỷ lệ alert tắt vĩnh viễn giảm từ 31% xuống 4%. Số incident **không** giảm mà tăng nhẹ, vì tổ chức bắt đầu tuyên bố incident cho những thứ trước đây xử lý im lặng — dấu hiệu tốt bị đọc nhầm là xấu.

**Cái không sửa được.** Không ai biết chắc hành động nào đã cứu hệ thống lúc 03h40. Hotfix pool gần như chắc chắn là nguyên nhân chính, nhưng vì Linh chạy re-index và Vy restart node trong cùng 14 phút mà không ai ghi thứ tự, tổ chức mất vĩnh viễn khả năng xác nhận. Điều này được ghi thẳng vào mục "những gì chúng ta không biết" của postmortem.

### Bài học

**Bài 1 — Sự cố nghiêm trọng không có nguyên nhân đơn lẻ; đi tìm một nguyên nhân là tự đảm bảo sẽ tái diễn.**

Cơ chế: mỗi lớp phòng vệ đều có lỗ, sinh ra từ những quyết định hợp lý cục bộ — tắt alert vì ồn, staging khác production vì bảo mật, review nhanh vì không muốn chặn đồng đội. Sự cố xảy ra khi các lỗ thẳng hàng; dừng ở lát cắt cuối là để nguyên ba lỗ kia. Dấu hiệu mắc lỗi này: postmortem kết thúc với đúng một action item, và nó là "cẩn thận hơn khi review". Xem [06-incident-va-metrics.md](/series/engineering-leadership/06-incident-va-metrics/) và [12-anti-patterns.md](/series/engineering-leadership/12-anti-patterns/).

**Bài 2 — Không có Incident Commander thì càng nhiều kỹ sư giỏi càng hồi phục chậm.**

Cơ chế: chi phí phối hợp tăng theo bình phương số người, năng lực điều tra chỉ tăng tuyến tính. Không có điểm hội tụ thông tin thì mỗi người giỏi mở một nhánh riêng và các nhánh thao tác chồng lên nhau — Linh scale DB trong khi Quân chuẩn bị failover. IC là vai của người **không chạm bàn phím**, điểm phản trực giác và khó bán nhất cho kỹ sư giỏi. Kèm theo là scribe: không ai ghi timeline theo phút thì postmortem sẽ dựa trên trí nhớ, tức dựa trên hư cấu. Xem [05-technical-leadership.md](/series/engineering-leadership/05-technical-leadership/).

**Bài 3 — Trong incident, cấp trên là một nguồn tải cần quản lý, không phải người cần phục vụ theo phản xạ.**

Cơ chế: mỗi câu hỏi từ trên xuống tiêu 10–15 phút khôi phục tập trung của người đang debug, và nhu cầu thật đằng sau câu hỏi thường không phải câu trả lời mà là giảm bất định — nhịp cập nhật cố định thoả mãn nhu cầu đó rẻ hơn nhiều. Không đưa ETA khi chưa có nguyên nhân; thay bằng cam kết về thời điểm ra quyết định tiếp theo. Điều kiện tiên quyết là thoả thuận trước rằng IC có quyền điều phối kênh bất kể chức danh. Xem [02-communication.md](/series/engineering-leadership/02-communication/).

**Bài 4 — Blameless không phải lòng tốt; nó là khoản đầu tư vào chất lượng thông tin, và phải được biện hộ bằng ngôn ngữ chi phí.**

Cơ chế: tốc độ hồi phục phụ thuộc vào việc thông tin về thay đổi gần đây có nổi lên nhanh không. Nếu tổ chức trừng phạt người có thay đổi liên quan tới sự cố, hành vi tối ưu của cá nhân là im lặng và chờ — hợp lý với cá nhân, tàn phá với tổ chức, đúng dạng Principal–Agent Problem. Khi bảo vệ một người trước ban lãnh đạo, lập luận đạo đức thường thua và lập luận chi phí thường thắng; hoãn quyết định nhân sự trước, đưa lập luận chi phí sau khi đã có dữ liệu. Xem [00-nen-tang-leadership.md](/series/engineering-leadership/00-nen-tang-leadership/), [03-team-leadership.md](/series/engineering-leadership/03-team-leadership/) và [08-hiring-va-phat-trien.md](/series/engineering-leadership/08-hiring-va-phat-trien/).

**Bài 5 — Lớp phòng vệ mục âm thầm, nên phải có cơ chế làm cho sự mục nhìn thấy được.**

Cơ chế: alert bị tắt, runbook cũ, staging lệch production đều là trạng thái tĩnh, không phát tín hiệu nào, và không ai bị đo lường theo chúng. Công việc bảo trì phòng vệ không có chủ vì nó không nằm trong sprint và không ai được thăng chức nhờ nó. Giải pháp là gắn hạn sử dụng vào từng lớp: alert tắt phải có ngày tự bật lại, runbook quá 90 ngày thì hiện đỏ, staging chạy load test hàng đêm để lệch cấu hình tự lộ. Ở tầng Manager: nếu sprint nào cũng full feature thì bạn đã quyết định — bằng cách không quyết định — rằng các lớp phòng vệ sẽ mục. Xem [05-technical-leadership.md](/series/engineering-leadership/05-technical-leadership/) và [12-anti-patterns.md](/series/engineering-leadership/12-anti-patterns/).

---

## Case 6 — Mâu thuẫn giữa Product và Engineering

### Bối cảnh

Một product startup Việt Nam, SaaS quản lý bán hàng cho chuỗi cửa hàng vừa và nhỏ, Series A được mười
bốn tháng, runway còn khoảng mười tám tháng (số minh hoạ). Nhà đầu tư bắt đầu hỏi về tốc độ tăng khách
hàng trả tiền, và câu hỏi đó dội xuống tổ chức dưới dạng một Roadmap quý dày đặc.

Engineering có 25 engineer, ba team. **Core** giữ luồng đơn hàng, thanh toán, tồn kho — Tuấn là Tech
Lead, Linh là Senior Backend. **Growth** làm onboarding và tích hợp kênh bán — Quân là Senior Fullstack,
lead trên thực tế nhưng chưa có title. **Platform** ba người, lo CI/CD, môi trường, observability, và
là team duy nhất không có dòng nào trong Roadmap sản phẩm.

Hà là Engineering Manager của cả ba team, báo cáo Minh — CTO, co-founder. Trang là Head of Product,
vào công ty mười tháng, đến từ một công ty e-commerce lớn hơn, báo cáo trực tiếp CEO. Vy là Product
Owner của team Core, làm việc hàng ngày với Tuấn.

```
                 CEO
                /     \
             CTO      Head of Product
            Minh          Trang
              |             |
             EM Hà        PO Vy, PO ...
           /   |   \
       Core  Growth  Platform
       Tuấn   Quân   (3 người)
```

Chi tiết cần thấy ngay từ sơ đồ: **Trang và Minh gặp nhau lần đầu ở cấp CEO.** Mọi mâu thuẫn không
giải được ở tầng Tuấn–Vy sẽ leo thẳng lên một cái đỉnh mà cả hai bên đều phải chuẩn bị hồ sơ trước
khi lên.

Tình trạng khi câu chuyện bắt đầu: ba quý liên tiếp trượt cam kết Roadmap.

| Quý | Hạng mục cam kết đầu quý | Ship trong quý | Tỷ lệ | Ghi chú |
|---|---|---|---|---|
| Q1 | 14 | 8 | 57% | 3 hạng mục ship thiếu scope, vẫn tính là ship |
| Q2 | 16 | 8 | 50% | 1 Incident production 6 tiếng giữa quý |
| Q3 | 18 | 9 | 50% | 2 hạng mục bị cắt vào tuần cuối |

*(Mọi số liệu trong case này là số minh hoạ, dùng để làm rõ cơ chế.)*

Chi tiết ít ai để ý lúc đó: số hạng mục cam kết tăng đều — 14, 16, 18 — trong khi số ship được đứng
yên ở 8–9, và đội ngũ không tăng người. Đây không phải dấu hiệu một team ngày càng lười. Đây là dấu
hiệu một hệ thống đang tự bơm phồng con số cam kết mà không ai chịu trách nhiệm về việc con số đó có
nghĩa gì.

Quan hệ giữa hai bộ phận xấu tới mức đo được bằng metadata email: trong sáu tuần cuối Q3 có mười một
email giữa Product và Engineering mà một bên cc CEO hoặc CTO ngay từ email đầu tiên (số minh hoạ).
Không email nào chửi bới. Tất cả đều lịch sự. Đó mới là điều đáng sợ.

> **Product gửi Tuấn, cc Minh và CEO:** "Em xác nhận lại là đồng bộ tồn kho đa chi nhánh đã được team
> confirm ngày 12/8 là kịp cuối tháng 9, và bên em đã thông báo ba khách hàng enterprise theo mốc đó."

> **Engineering gửi Trang, cc Minh:** "Con số ngày 12/8 dựa trên giả định API đối tác ổn định và không
> đổi scope. Từ đó tới nay có bốn yêu cầu bổ sung, em đính kèm danh sách."

Cả hai email đều đúng sự thật, đều do người có thiện chí viết, và đều là hành động phòng thủ. Khi xung
đột chuyển sang dạng văn bản lịch sự có cc sếp, nó không còn là xung đột về công việc — nó đã thành
xung đột về hồ sơ. Mỗi email là một mảnh bằng chứng cất đi để dùng cho phiên xử sau.

### Triệu chứng ban đầu

Liệt kê những gì **quan sát được**, không phải những gì các bên cảm thấy. Trong xung đột liên bộ phận,
cảm giác của hai bên luôn mâu thuẫn nhau; hiện tượng thì không.

- **Estimation mất giá trị thông tin.** Sai số trung bình của team Core với hạng mục 2–4 tuần là +85%
  (số minh hoạ), và sai số này *không giảm* qua ba quý dù đã có bốn buổi retro về estimation. Hệ thống
  học được thì sai số phải giảm; sai số đứng yên sau nhiều vòng phản hồi nghĩa là tín hiệu phản hồi
  đang bị chặn ở đâu đó.
- **Đổi ưu tiên 3,2 lần mỗi Sprint hai tuần.** Mỗi lần đều có lý do chính đáng: khách lớn doạ rời, đối
  thủ ra tính năng tương tự, CEO gặp bug khi demo. Nhưng tính chính đáng của từng lần không nói gì về
  chi phí tổng của 3,2 lần — một dạng chi phí cộng dồn không ai sở hữu.
- **Kỹ sư bắt đầu nói giọng luật sư.** Câu quen của Linh chuyển từ "cái này chắc một tuần" sang "em cần
  chị xác nhận bằng văn bản là scope chỉ có ba màn hình này". Khi kỹ sư đòi bằng chứng văn bản từ đồng
  nghiệp nội bộ, chi phí giao dịch nội bộ đã vượt chi phí làm việc thật.
- **Team Platform biến mất.** Không hạng mục nào trong Roadmap, ba người dần bị mượn sang Core và
  Growth mỗi lần có deadline; thời gian cho việc nền tảng giảm từ 60% xuống 20% (số minh hoạ). Hệ quả
  trễ: build CI tăng từ 9 lên 22 phút, mỗi release tốn thêm nửa ngày thao tác tay. Không ai nối các số
  này với việc trượt Roadmap, vì chúng nằm ở bảng khác, do người khác đọc.
- **Hai kỹ sư giỏi nghỉ trong Q3**, cả hai viết trong exit interview một câu gần giống nhau: "làm ở
  đây em không thấy việc mình làm có ý nghĩa gì với sản phẩm" — dấu hiệu liên kết giữa công việc kỹ
  thuật và kết quả kinh doanh đã đứt trong nhận thức người làm.

#### Một buổi planning điển hình, trước khi sửa

Bản ghi rút gọn buổi Sprint planning team Core, tuần thứ hai Q3. Đọc với câu hỏi: *ai đang nói dối?*
Câu trả lời là không ai — và đó mới là vấn đề.

> **Trang:** Mục tiêu quý này của bên chị là ra được luồng thanh toán trả góp trước hội nghị khách hàng
> ngày 20 tháng 11. Sales đang có ba deal chờ đúng tính năng này. Tuấn ước lượng giúp chị bao lâu?
>
> **Tuấn:** Em hỏi lại chút đã. Trả góp thì mình tích hợp mấy đối tác ạ?
>
> **Vy:** Ban đầu một thôi anh. Nhưng làm sao sau này thêm đối tác thứ hai không phải viết lại.
>
> **Tuấn:** *(im ba giây)* Vâng. Một đối tác, cộng phần abstraction để sau mở rộng, cộng luồng đối
> soát… em nghĩ khoảng tám tuần.
>
> **Trang:** Tám tuần là qua mất hội nghị rồi em. Chị hỏi thật: tám tuần này là chắc chắn, hay là đã
> tính cả rủi ro trong đó?
>
> **Tuấn:** Dạ… là tám tuần theo kinh nghiệm của em với việc tương tự.
>
> **Trang:** Chị hiểu. Nhưng lần trước luồng hoàn tiền em cũng nói sáu tuần, cuối cùng xong trong bốn
> tuần rưỡi. Chị không trách gì đâu, chị chỉ muốn hiểu cách team ước lượng. Nếu bỏ phần abstraction,
> làm cứng cho một đối tác trước, thì bao lâu?
>
> **Tuấn:** *(nhìn Linh)* Linh nghĩ sao?
>
> **Linh:** Hardcode một đối tác thì integration khoảng ba tuần. Nhưng đối soát vẫn phải làm, và cái
> đó phụ thuộc bên tài chính trả lời format file.
>
> **Trang:** Vậy ba tuần cộng đối soát. Chị sẽ push tài chính trả lời trong tuần này. Nếu chị lo được
> phần đó thì mình chốt năm tuần được không? Năm tuần là còn một tuần đệm trước hội nghị.
>
> **Tuấn:** *(bốn giây im lặng)* … Năm tuần thì rất căng chị ạ.
>
> **Trang:** Chị biết là căng. Nhưng đây là ba deal, giá trị hợp đồng năm đầu khoảng bốn tỷ. Nếu em
> nói không được thì chị báo lại anh CEO là mình bỏ, chị chấp nhận. Chị chỉ cần biết chính xác là được
> hay không.
>
> **Tuấn:** … Thôi để em với Linh cố. Nhưng em nói trước, nếu đối tác trả API chậm thì em không đảm bảo.
>
> **Trang:** Được. Chị ghi nhận rủi ro đó. Cảm ơn team.

Buổi họp kết thúc trong không khí tốt. Ai cũng lịch sự. Và mọi thứ vừa xảy ra đều sai:

1. **Tám tuần không phải ước lượng.** Đó là ước lượng thật (khoảng năm tuần rưỡi trong đầu Tuấn) cộng
   một lớp đệm phòng thủ. Tuấn không nói dối theo nghĩa đạo đức; anh đang phản ứng hợp lý với môi
   trường mà con số của anh sẽ thành cam kết cứng và sẽ bị mang ra đối chất khi trượt.
2. **"Tám tuần chắc chắn hay đã tính rủi ro" không phải câu hỏi**, đó là một nước đi đàm phán. Trang
   biết con số đầu tiên của Engineering luôn có đệm. Cô không sai — cô đã học một quy luật thật từ dữ
   liệu thật.
3. **Dẫn chứng "sáu tuần xong trong bốn tuần rưỡi" là bằng chứng chính xác**, và nó củng cố đúng hành
   vi phá hệ thống: dạy Trang rằng đàm phán xuống là hợp lý, dạy Tuấn rằng lần sau phải đệm dày hơn.
4. **Bỏ abstraction để rút tám xuống ba tuần là quyết định kiến trúc có hệ quả nhiều năm, ra trong bốn
   mươi giây, bởi một cuộc mặc cả về lịch.** Không ai ghi lại rằng technical debt này tồn tại, vì sao,
   và khi nào phải trả.
5. **"Nếu em nói không được thì chị báo CEO là mình bỏ" là một câu ép rất mạnh gói trong hình thức
   trao quyền.** Trang không cố ý ép. Nhưng ở bối cảnh Việt Nam, nơi nói "không" trực diện với người
   ngang hoặc cao cấp hơn đã khó sẵn, câu này đặt Tuấn vào thế: hoặc nhận, hoặc thành người khiến công
   ty mất bốn tỷ.
6. **"Thôi để em với Linh cố" không phải cam kết**, đó là câu xã giao để thoát khỏi phòng họp. Nhưng
   nó được ghi vào biên bản như một cam kết.

Mấu chốt: hai người thông minh, có thiện chí, cùng muốn công ty thắng, vừa cùng nhau tạo ra một con số
vô nghĩa — và cả hai đều biết nó vô nghĩa — mà không ai nói ra.

#### Ba góc nhìn về cùng một sự việc: tính năng bị cắt vào phút cuối

Tuần cuối Q3. Tính năng "báo cáo doanh thu theo chi nhánh" dự kiến ship thứ Sáu. Chiều thứ Tư, Tuấn và
Hà quyết định cắt.

**Engineer (Linh):** "Em làm ba tuần. Thứ Ba em phát hiện query gộp theo chi nhánh trên dữ liệu của
khách lớn nhất chạy mười bốn giây và khoá bảng orders, làm chậm cả luồng tạo đơn. Sửa đúng thì cần
bảng tổng hợp và job chạy nền, thêm tuần rưỡi. Sửa nhanh thì cache, nhưng số liệu tài chính trễ ba
mươi phút — em không dám ship. Cắt là đúng. Cái em ức là ba tuần của em không ai nhìn thấy: trong
report gửi CEO nó chỉ là một dòng gạch đỏ 'không hoàn thành'. Em không có chỗ nào để nói rằng em vừa
ngăn một sự cố production."

Cơ chế: **công sức phòng ngừa không có đơn vị đo.** Incident bạn ngăn được không bao giờ xuất hiện
trong báo cáo nào, vì nó đã không xảy ra. Đây là bất đối xứng cấu trúc giữa việc tạo tính năng và việc
bảo vệ hệ thống.

**PO (Vy):** "Em đã hẹn hai khách là thứ Sáu có. Một trong hai vừa gia hạn hợp đồng, và em dùng chính
tính năng này làm lý do để họ gia hạn. Chiều thứ Tư anh Tuấn nhắn một dòng Slack: 'bên anh phải cắt
cái report, có vấn đề performance'. Em không hiểu 'vấn đề performance' là gì, không biết nghiêm trọng
cỡ nào, không biết nếu ship thì rủi ro thật ra sao để còn cân. Em không đòi ship bằng mọi giá. Em cần
biết trước ba ngày chứ không phải hai, và cần một câu giải thích em lặp lại được cho khách."

Cơ chế: **thông tin đến muộn và ở sai độ phân giải.** Cái Vy cần không phải quyền phủ quyết quyết định
kỹ thuật, mà là thời gian để tái đàm phán với thế giới bên ngoài. Engineering hay nghĩ báo sớm là làm
người khác lo sớm; thực ra báo sớm là chuyển rủi ro sang người có công cụ xử lý rủi ro đó.

**CTO (Minh):** "Về kỹ thuật, Tuấn quyết đúng, tôi ký cả hai tay. Nhưng tôi thấy thứ hai người kia
không thấy: đây là lần thứ năm trong quý mình phát hiện vấn đề hiệu năng ở tuần cuối. Không phải tuần
thứ hai, không phải lúc thiết kế — tuần cuối. Nghĩa là mình không có cơ chế phát hiện rủi ro hiệu năng
sớm: không load test trên dữ liệu giống production, không ai review query plan lúc design. Cắt tính
năng chỉ là chi phí nổi lên bề mặt. Và điều tôi lo hơn: Tuấn báo cho Vy bằng một dòng Slack chiều thứ
Tư vì Tuấn đang sợ. Người sợ thì báo muộn và báo ngắn."

Cơ chế: **một sự kiện đơn lẻ với người bên dưới là một mẫu lặp với người đứng trên.** Đây là khác biệt
cốt lõi giữa cấp Tech Lead và cấp Director: cùng dữ liệu, nhưng đơn vị phân tích là "sự việc" hay "phân
bố của các sự việc". Ba góc nhìn đều đúng, không góc nào bao trùm hai góc kia, và ba người chưa bao giờ
ngồi nghe nhau — vì công ty không có nghi thức nào cho việc đó.

### Chẩn đoán sai ban đầu

Cuối Q3, CEO yêu cầu Minh và Trang "giải quyết dứt điểm". Hai người ngồi hai tiếng và đi tới kết luận
nghe rất hợp lý: *"Vấn đề là hai bên chưa hiểu nhau. Product không hiểu ràng buộc kỹ thuật, Engineering
không hiểu áp lực thị trường. Cần giao tiếp tốt hơn."*

Giải pháp: một buổi sync Product–Engineering hàng tuần 90 phút; một kênh Slack chung; quy định mọi thay
đổi ưu tiên phải được thảo luận trong buổi sync; một buổi team building chung. Cả hai bên đều hoan
nghênh.

Sáu tuần sau, tình hình xấu hơn (số minh hoạ):

| Chỉ số | Trước khi thêm sync | Sau 6 tuần |
|---|---|---|
| Giờ họp/tuần của một Tech Lead | 6,5 | 9,0 |
| Số lần đổi ưu tiên giữa Sprint | 3,2 | 3,5 |
| Sai số estimation trung bình | +85% | +110% |
| Email có cc cấp trên / 6 tuần | 11 | 14 |
| Thời gian coding tập trung của Tuấn/tuần | ~14h | ~9h |

Ba cơ chế giải thích vì sao một giải pháp nghe hợp lý lại làm mọi thứ tệ hơn.

**Cơ chế 1 — Thêm kênh giao tiếp không sửa được hàm mục tiêu.**

Trang và Tuấn không xung đột vì thiếu thông tin về nhau; họ hiểu nhau khá rõ. Họ xung đột vì đang được
đo bằng hai hàm mục tiêu khác nhau, và trong nhiều tình huống hai hàm đó nghịch nhau.

| Trang (Head of Product) được đo bằng | Tuấn và Hà (Engineering) được đo bằng |
|---|---|
| Số tính năng cam kết ship đúng hạn | Uptime, số Incident P1/P2 |
| Số khách hàng trả tiền mới | Thời gian khôi phục sau sự cố |
| Tỷ lệ adoption tính năng mới | Tỷ lệ bug rò rỉ ra production |
| Doanh thu mở rộng từ khách cũ | Giữ chân kỹ sư, sức khoẻ team |

Khi phải chọn giữa "ship thứ Sáu này với một shortcut" và "lùi hai tuần để làm đúng", cột trái nghiêng
về đâu và cột phải nghiêng về đâu là điều không cần bàn.

Điểm tinh tế: **không ai cố ý thiết kế mâu thuẫn này.** Không có cuộc họp nào mà ai đó nói "hãy đặt hai
bộ phận vào thế đối đầu". Hai bảng chỉ số được viết ở hai thời điểm khác nhau, bởi hai người khác nhau,
mỗi bảng đều hợp lý khi đọc riêng. Mâu thuẫn sinh ra ở chỗ giao nhau, và chỗ giao nhau thì không ai sở
hữu. Đây là lỗi **thiết kế tổ chức**, không phải lỗi con người. Khi hàm mục tiêu nghịch nhau, thêm một
buổi họp 90 phút chỉ tạo thêm sân khấu để hai hàm đó va vào nhau, với nhiều khán giả hơn.

**Cơ chế 2 — Vòng xoáy mất niềm tin trong Estimation.**

```
   Engineering đệm ước lượng ngầm để tự vệ
                  ↓
   Một số việc xong sớm hơn con số đã nói
                  ↓
   Product học được: con số đầu tiên luôn có đệm
                  ↓
   Product đàm phán xuống một cách hệ thống
                  ↓
   Engineering thấy số thật bị cắt → đệm dày hơn
                  ↓
   Ước lượng mất hoàn toàn giá trị thông tin
                  ↓
   Hai bên ra quyết định trên con số vô nghĩa
                  ↓
   Trượt cam kết → mất niềm tin thêm → quay lại đầu
```

Hai đặc điểm làm vòng xoáy này rất khó phá.

*Thứ nhất, cả hai bên đều đang hành động hợp lý.* Nếu bạn là Tuấn và biết con số mình đưa ra sẽ bị cắt
30% rồi thành cam kết cứng, đệm 30% là chiến lược tối ưu. Nếu bạn là Trang và biết con số kia có đệm,
đàm phán xuống là chiến lược tối ưu. Đây là một **cân bằng xấu**: mỗi bên đang chơi nước tốt nhất trước
nước đi của bên kia, và điểm cân bằng chung lại tệ cho cả hai. Đặc trưng của nó là **không bên nào thoát
ra đơn phương được**. Tuấn bỏ đệm mà Trang vẫn cắt 30% thì Tuấn chết; Trang ngừng cắt mà Tuấn vẫn đệm
thì Roadmap phồng lên vô lý.

*Thứ hai, không ai nói ra được.* Để nói về vòng xoáy, Tuấn phải mở đầu bằng "thật ra em vẫn đệm ước
lượng" — thừa nhận đã đưa số không thật suốt hai năm. Trang phải mở đầu bằng "chị vẫn cắt số của em
xuống theo thói quen" — thừa nhận đã ép team. Cả hai lời thú nhận đều đắt về uy tín và về việc giữ mặt,
vốn là chi phí rất thật trong môi trường Việt Nam. Kết quả: **vòng xoáy được duy trì bởi chính chi phí
của việc gọi tên nó.** Đây là lý do buổi sync hàng tuần vô dụng — chừng nào chủ đề duy nhất không được
phép nói vẫn không được nói, thời gian thêm chỉ dùng để bàn kỹ hơn về triệu chứng.

**Cơ chế 3 — "Giao tiếp tốt hơn" chuyển gánh nặng xuống cá nhân.**

Kết luận "vấn đề là giao tiếp" hàm ý ngầm: hệ thống ổn, con người chưa đủ khéo. Nó buộc Tuấn và Trang
giải quyết bằng nỗ lực cá nhân một thứ họ không có thẩm quyền thay đổi — không ai trong hai người sửa
được bảng KPI của người kia. Và nó tạo cớ đổ lỗi mới: khi mọi thứ vẫn hỏng dù đã có "kênh giao tiếp",
kết luận tiếp theo rất tự nhiên là "vậy là do thái độ của bên kia". Chẩn đoán sai không trung tính; nó
chủ động làm hỏng thêm.

Một phép thử nhanh: **nếu giải pháp của bạn cho xung đột liên bộ phận là thêm một cuộc họp, hãy hỏi cuộc
họp đó thay đổi động cơ của ai.** Nếu câu trả lời là "không của ai cả", bạn vừa tạo ra một nghi lễ.

**Điểm phải nói thẳng:** sửa hàm mục tiêu không phải việc Trang và Tuấn tự thoả thuận được. Hai người ở
hai nhánh báo cáo, gặp nhau ở cấp CEO. Không ai có quyền viết lại bảng chỉ số của người kia, cũng không
ai cam kết được rằng cách đo mới sẽ được cấp trên của bên kia công nhận. Mọi "thoả thuận quân tử" giữa
hai người ngang cấp trong tình huống này đều sụp ở lần đầu có áp lực thật, vì khi đó mỗi người quay về
bảng chỉ số mà sếp mình đang dùng để đánh giá mình. **Chỉ người đứng trên cả hai hàm mục tiêu mới sửa
được hàm mục tiêu** — ở đây là Minh, cùng CEO. Nếu Minh né và tiếp tục yêu cầu Tuấn "làm việc khéo hơn
với Product", Minh đang uỷ thác xuống dưới một vấn đề chỉ giải được ở trên.

### Các lựa chọn thực tế

Minh viết ra cả những phương án anh biết sẽ loại, vì lý do loại chính là thứ anh cần khi giải thích với
CEO và với hai bộ phận.

**A — Siết kỷ luật cam kết.** Giữ nguyên cấu trúc, biến cam kết Roadmap thành ràng buộc cứng: trượt thì
đưa vào Performance Review, thay đổi scope phải qua phê duyệt chặt. Logic riêng của nó có thật: cam kết
không có hậu quả thì chỉ là mong muốn. Nhưng trong bối cảnh vòng xoáy đệm số, tăng hình phạt cho việc
trượt sẽ **làm tăng động cơ đệm**. Bạn đang tăng giá của việc sai trong khi nguyên nhân sai là thông tin
đầu vào đã hỏng.

**B — Product toàn quyền quyết scope, Engineering thuần thực thi.** Xoá mâu thuẫn bằng cách cho một bên
thắng: Product quyết tất, Engineering nhận việc và báo thời gian. Thực chất là mô hình ODC áp vào một
công ty product. Ưu điểm thật: chấm dứt tranh cãi, tăng tốc độ ra quyết định ngắn hạn, và đúng trong
một số bối cảnh như outsourcing nơi scope thuộc về khách. Nhưng ở product startup nó cắt đứt một vòng
phản hồi: người hiểu chi phí kỹ thuật của lựa chọn không còn tham gia vào việc chọn. Hệ quả trung hạn
quen thuộc là technical debt tích nhanh, kỹ sư giỏi rời đi, sáu tháng sau tốc độ ship giảm mạnh dù
không ai đổi cách làm.

**C — Bỏ Estimation, chuyển sang mô hình dòng chảy.** Ngừng ước lượng theo tuần, giới hạn việc đang làm
dở, đo lead time thật, dự báo bằng phân bố lịch sử. Product hỏi "cái này đứng thứ mấy trong hàng đợi"
thay vì "bao lâu". Về lý thuyết đây là phương án tấn công đúng chỗ hỏng nhất. Nhưng nó đòi hai điều
kiện công ty chưa có: dữ liệu lịch sử đủ sạch để dự báo, và mức tin cậy đủ cao để Product chấp nhận câu
trả lời "chưa có ngày, nhưng nó đang ở vị trí thứ hai". Áp một mô hình đòi hỏi niềm tin vào tổ chức
đang khủng hoảng niềm tin là đặt sai thứ tự.

**D — Tái cấu trúc thành squad có chung người quản lý.** Mỗi squad gồm PO, engineer, designer, cùng báo
cáo một người. Mâu thuẫn liên bộ phận biến thành mâu thuẫn nội bộ squad, dễ giải hơn nhiều vì có một
người duy nhất chịu trách nhiệm cho cả hai loại kết quả. Mạnh nhất về cấu trúc và đắt nhất: đổi đường
báo cáo, đổi cách đánh giá, có người mất phạm vi quản lý, mất ít nhất một quý để ổn định. Câu hỏi không
phải nó đúng hay sai, mà là công ty 25 engineer đang trượt ba quý và còn mười tám tháng runway có chịu
nổi một quý nhiễu loạn ngay lúc này không.

**E — Sửa hai tầng: hàm mục tiêu và nghi thức làm việc.** Không đổi cấu trúc báo cáo. *Tầng 1 — thiết kế
mục tiêu:* một bảng mục tiêu chung duy nhất chứa cả chỉ số sản phẩm lẫn chỉ số sức khoẻ hệ thống, và
quan trọng nhất là **được đọc bởi cùng một người, trong cùng một buổi review**; Trang và Minh cùng chịu
trách nhiệm cho toàn bảng, không ai chỉ chịu nửa của mình. *Tầng 2 — nghi thức làm việc:* ước lượng dạng
khoảng công khai, capacity chia theo bucket thoả thuận trước quý, quy tắc đổi ưu tiên giữa Sprint có giá
nói rõ. Phương án này chấp nhận rằng cấu trúc hiện tại còn dùng được ở quy mô 25 engineer, và đánh vào
chỗ có tỷ lệ hiệu lực trên chi phí cao nhất: cách đo và cách nói. Nó cũng là phương án duy nhất mà tầng
1 bắt buộc phải do người đứng trên làm — vừa là điểm mạnh, vừa là rủi ro nếu Minh và CEO không thật sự
cam kết.

### Trade-off của từng lựa chọn

| Phương án | Được gì | Mất gì | Ai chịu phần mất | Nên chọn khi |
|---|---|---|---|---|
| A — Siết cam kết | Kỷ luật hình thức, dễ giải thích với nhà đầu tư | Đệm dày hơn, che giấu rủi ro sớm, giảm tính trung thực | Engineer tuyến dưới, rồi khách hàng khi bug ra production | Đã có estimation đáng tin và vấn đề thật là kỷ luật cá nhân |
| B — Product toàn quyền | Hết tranh cãi, ra quyết định nhanh, rõ trách nhiệm | Technical debt tích nhanh, mất kỹ sư senior, tốc độ giảm sau 2–3 quý | Chính Product sau 6 tháng, dưới dạng tốc độ ship tụt | Scope thuộc bên ngoài (ODC, dự án khách hàng), vòng đời sản phẩm ngắn |
| C — Bỏ Estimation | Loại bỏ con số vô nghĩa, dự báo bằng dữ liệu thật | Đòi niềm tin và dữ liệu chưa có; Product mất công cụ hứa với khách | Sales và Product, ngay lập tức | Tin cậy hai bên đã ổn, có ≥2 quý dữ liệu lead time sạch |
| D — Reorg thành squad | Sửa tận gốc bằng cấu trúc, một người chịu cả hai kết quả | Một quý nhiễu loạn, có người mất phạm vi quản lý, rủi ro nghỉ việc | Toàn tổ chức trong ngắn hạn | Quy mô >40–50 engineer, hoặc đã thử E mà thất bại |
| E — Sửa mục tiêu + nghi thức | Đánh đúng cơ chế, chi phí cấu trúc thấp, giữ được người | Chậm thấy kết quả (1–2 quý), sụp nếu cấp trên không giữ cam kết | Minh và CEO — họ phải bỏ vốn chính trị | Quy mô 15–40 engineer, cấu trúc còn dùng được, lãnh đạo chịu tự sửa mình |

Trục trade-off thật ở đây không phải "phương án nào tốt nhất" mà là **ai trả giá và trả khi nào**. A và
B đẩy chi phí xuống tuyến dưới và về tương lai — đó là lý do chúng luôn hấp dẫn trong tuần đầu. D và E
bắt lãnh đạo trả trước. Một quy tắc dùng được: khi triệu chứng lặp ở nhiều team và nhiều quý, chi phí
gần như chắc chắn nằm ở tầng thiết kế, và phương án nào không bắt tầng thiết kế trả giá thì phương án
đó chỉ dời vấn đề.

### Quyết định và cách thực thi

Minh chọn **E ngay trong quý tới, giữ D làm phương án cho mốc 40 engineer**. Lý do viết ra thành văn:
runway không cho phép một quý nhiễu loạn, và nếu chưa sửa được hàm mục tiêu thì reorg chỉ chuyển mâu
thuẫn vào bên trong squad dưới dạng khó thấy hơn.

Việc đầu tiên không phải công bố quy trình mới, mà là một cuộc trò chuyện ba bên do Minh chủ trì. Điểm
kỹ thuật của buổi này: **Minh không phân xử ai đúng ai sai, anh đặt tên cho vòng xoáy và nhận phần lỗi
thuộc về mình trước.** Đó là cách duy nhất để hai người kia trả được chi phí thú nhận.

> **Minh:** Hôm nay tôi không hỏi ai sai. Tôi muốn mô tả một cơ chế, hai người nghe xem có đúng không.
> Tuấn, khi em đưa một con số, em có cộng thêm một lớp phòng thân không? Em không phải trả lời ngay,
> tôi nói tiếp đã. Trang, khi chị nhận một con số từ Engineering, chị có mặc định trong đầu là nó cắt
> được khoảng 20–30% không?
>
> **Trang:** *(im một lúc)* … Có. Chị nghĩ ai làm Product lâu cũng có phản xạ đó.
>
> **Minh:** Tuấn?
>
> **Tuấn:** Dạ… có ạ. Em cộng đệm. Vì em bị hỏi lại đúng ba lần trong sáu tháng qua là "sao lúc trước
> em nói thế".
>
> **Minh:** Cảm ơn hai người. Thứ vừa xảy ra là điều tôi cần. Bây giờ nhìn lại: em đệm vì chị cắt, chị
> cắt vì em đệm. Không ai bắt đầu, và không ai dừng lại một mình được — nếu Tuấn bỏ đệm mà Trang vẫn
> cắt thì Tuấn chết, nếu Trang ngừng cắt mà Tuấn vẫn đệm thì Roadmap của Trang thành vô nghĩa. Đây
> không phải vấn đề thái độ. Đây là một cân bằng xấu, và cả hai người đang chơi đúng.
>
> **Trang:** Vậy ai phá được?
>
> **Minh:** Tôi. Và tôi phải nói phần lỗi của mình trước: bảng đánh giá của Engineering có uptime và
> số Incident, không có một dòng nào về tốc độ ra tính năng. Bảng của Product có ngày ship, không có
> dòng nào về sức khoẻ hệ thống. Hai bảng đó do tôi và anh CEO viết. Chúng ta vừa yêu cầu hai người
> hợp tác trong khi trả lương cho hai người theo hai hướng ngược nhau. Trong hai tuần tới tôi và anh
> CEO sẽ viết lại thành một bảng, và tôi với Trang sẽ cùng bị hỏi về cả bảng đó, không ai chỉ trả lời
> nửa của mình.
>
> **Tuấn:** Nếu vậy thì em xin một điều: cho em được đưa ước lượng dạng khoảng, và khoảng đó không bị
> đàm phán xuống. Muốn nhanh hơn thì mình bàn cắt scope, chứ đừng bàn cắt số.
>
> **Minh:** Đó chính là điều thứ hai tôi định đề xuất. Trang thấy sao?
>
> **Trang:** Chị đồng ý, với một điều kiện ngược lại: khi team biết trượt, chị cần biết trong vòng 48
> tiếng, không phải tuần cuối. Chị không cần lời hứa, chị cần thời gian.

Sau buổi đó, hai tầng được triển khai.

**Tầng 1 — thiết kế mục tiêu (Minh và CEO làm, mất ba tuần).** Một bảng mục tiêu chung cho quý, sáu
dòng: ba dòng chỉ số sản phẩm (khách trả tiền mới, adoption tính năng mới, doanh thu mở rộng) và ba
dòng chỉ số sức khoẻ hệ thống (số Incident P1/P2, thời gian build và release, tỷ lệ thời gian dành cho
việc nền tảng). Trang và Minh cùng trình bày toàn bộ sáu dòng trong cùng một buổi review với CEO. Không
ai được phép chỉ báo cáo nửa của mình. Team Platform lần đầu có ba dòng trong bảng mà CEO đọc.

**Tầng 2 — nghi thức làm việc (Hà và Trang làm, áp dụng ngay từ Sprint kế tiếp).**

- *Ước lượng dạng khoảng công khai.* Mọi hạng mục có một khoảng (ví dụ "4–7 tuần") kèm hai đến ba giả
  định làm nó rộng như vậy. Khoảng không bị đàm phán; muốn thu hẹp thì phải xoá giả định (chốt format
  đối soát, khoá scope) hoặc cắt phạm vi. Ghi công khai để lần sau đối chiếu.
- *Capacity theo bucket thoả thuận trước quý.* 60% tính năng mới, 20% chất lượng và nền tảng, 20% việc
  không lường trước (số minh hoạ, chốt trước quý). Bucket 20% nền tảng do Engineering toàn quyền, không
  phải xin. Bucket 20% dự phòng chính là chỗ tiếp các việc gấp — nó thay thế lớp đệm ngầm bằng một lớp
  đệm công khai mà cả hai bên cùng nhìn thấy.
- *Đổi ưu tiên giữa Sprint có giá rõ ràng.* Vẫn được đổi, nhưng theo nguyên tắc một-vào-một-ra: người
  yêu cầu phải chỉ ra hạng mục nào bị đẩy ra và ký vào đó. Việc đổi vẫn nhanh, chỉ là không còn miễn phí.
- *Quy tắc 48 tiếng.* Ai thấy dấu hiệu trượt phải báo trong 48 tiếng, kèm ba phương án (cắt scope, lùi
  ngày, thêm rủi ro chấp nhận được). Báo sớm không bị hỏi tội; báo muộn thì có.

| Cơ chế | Trước | Sau |
|---|---|---|
| Đơn vị ước lượng | Một con số duy nhất, có đệm ngầm | Khoảng công khai kèm giả định |
| Cách rút ngắn lịch | Đàm phán hạ con số | Xoá giả định hoặc cắt scope |
| Chỗ chứa rủi ro | Đệm giấu trong từng ước lượng | Bucket 20% dự phòng, công khai |
| Đổi ưu tiên | Miễn phí, không ai đếm | Một-vào-một-ra, có người ký |
| Việc nền tảng | Xin khi rảnh | Bucket 20% mặc định, có dòng trong bảng mục tiêu |
| Tin xấu | Nổi lên ở tuần cuối | Bắt buộc trong 48 tiếng, kèm ba phương án |
| Ai đọc chỉ số | Product đọc bảng Product, Engineering đọc bảng Engineering | Một bảng, cùng một buổi, hai người cùng trả lời |
| Nơi giải quyết xung đột | Email cc CEO | Bàn ưu tiên tại chỗ, leo lên khi hết bucket |

### Hậu quả

Sau hai quý (số minh hoạ):

| Chỉ số | Trước | Sau Q1 | Sau Q2 |
|---|---|---|---|
| Hạng mục cam kết / ship | 18 / 9 | 12 / 10 | 13 / 12 |
| Sai số estimation trung bình | +110% | +55% | +30% |
| Đổi ưu tiên giữa Sprint | 3,5 | 2,1 | 1,4 |
| Email có cc cấp trên / 6 tuần | 14 | 5 | 2 |
| Thời gian build CI | 22 phút | 16 phút | 9 phút |
| Incident P1/P2 mỗi quý | 5 | 4 | 2 |

Điều đáng chú ý nhất không nằm ở cột cuối mà ở dòng đầu: **số hạng mục cam kết giảm từ 18 xuống 12.**
Trang chấp nhận hứa ít hơn với CEO. Đó là chi phí chính trị thật mà cô phải trả, và nó chỉ trả được vì
Minh đã đứng cùng cô trong buổi review — nếu Trang phải một mình giải thích vì sao Roadmap ngắn đi, cô
đã không làm.

Những gì không tốt lên, và cần nói thẳng:

- **Quý đầu tiên tệ hơn về cảm giác.** Khi lớp đệm ngầm bị đưa ra ánh sáng, con số cam kết tụt mạnh và
  trông như team đang chậm lại. Có hai tuần CEO nghi ngờ toàn bộ thay đổi này. Bất kỳ ai làm việc tương
  tự đều phải chuẩn bị cho khoảng trũng đó và nói trước với cấp trên rằng nó sẽ xảy ra.
- **Quân ở team Growth vẫn không có title.** Nghi thức mới không sửa được vấn đề Career Ladder; ba tháng
  sau Quân nghỉ. Sửa cơ chế phối hợp không thay thế được việc sửa đường thăng tiến.
- **Quy tắc một-vào-một-ra bị lách hai lần trong quý đầu**, cả hai lần đều do CEO yêu cầu trực tiếp.
  Không quy tắc nào sống sót nếu người cao nhất được miễn trừ; Minh phải nói riêng với CEO một lần nữa.
- **Ước lượng dạng khoảng làm một số PO khó chịu**, vì khách hàng không mua "4–7 tuần". Cách xử lý cuối
  cùng là Product cam kết ra ngoài bằng đầu trên của khoảng, và giữ phần chênh làm vùng an toàn của
  chính mình — tức là đệm vẫn tồn tại, nhưng nó nằm ở đúng người có quyền đàm phán với bên ngoài.

### Bài học

**1. Xung đột lặp lại giữa hai bộ phận hầu như luôn là lỗi thiết kế mục tiêu, không phải lỗi tính cách.**
Cơ chế: con người tối ưu theo thứ họ bị đo. Khi hai nhóm bị đo bằng hai hàm mục tiêu nghịch nhau, hành
vi xung đột là kết quả tất yếu, và thay người chỉ tạo ra một cặp xung đột mới sau vài tháng. Dấu hiệu
sớm: cùng một loại tranh cãi tái diễn với những người khác nhau. Xem
[00-nen-tang-leadership.md](/series/engineering-leadership/00-nen-tang-leadership/) và
[09-to-chuc-va-scaling.md](/series/engineering-leadership/09-to-chuc-va-scaling/).

**2. Thêm cuộc họp không sửa được động cơ.** Cơ chế: giao tiếp chỉ giải quyết bất cân xứng thông tin;
nó không giải quyết được xung đột lợi ích. Khi hai bên đã hiểu nhau nhưng vẫn bị thưởng phạt ngược
chiều, kênh giao tiếp thêm chỉ tăng tần suất và khán giả của va chạm. Phép thử trước khi lập bất kỳ
nghi thức nào: cuộc họp này thay đổi động cơ của ai. Xem
[02-communication.md](/series/engineering-leadership/02-communication/) và [12-anti-patterns.md](/series/engineering-leadership/12-anti-patterns/).

**3. Estimation là kênh thông tin, và nó hỏng theo cách tự củng cố.** Cơ chế: đệm sinh ra cắt, cắt sinh
ra đệm dày hơn, đến khi con số không còn mang thông tin. Cách chữa không phải kêu gọi trung thực mà là
đổi định dạng của tín hiệu — khoảng công khai kèm giả định, thu hẹp bằng cách xoá giả định chứ không
bằng đàm phán — và đưa lớp đệm ra thành bucket ai cũng thấy. Xem
[07-project-delivery.md](/series/engineering-leadership/07-project-delivery/).

**4. Một số vấn đề chỉ giải được bởi người đứng trên cả hai bên, và nhận ra điều đó là một kỹ năng ra
quyết định.** Cơ chế: thoả thuận giữa hai người ngang cấp không tồn tại lâu hơn áp lực đầu tiên, vì mỗi
người vẫn bị đánh giá bởi cấp trên riêng. Nếu bạn là Tech Lead, việc đúng cần làm không phải cố dàn xếp
mà là mô tả cơ chế lên trên và yêu cầu người có thẩm quyền sửa cách đo — kèm bằng chứng, không kèm lời
than. Xem [04-decision-making.md](/series/engineering-leadership/04-decision-making/) và
[00-nen-tang-leadership.md](/series/engineering-leadership/00-nen-tang-leadership/).

**5. Gọi tên vòng xoáy đắt hơn duy trì nó, nên người có nhiều vốn chính trị nhất phải nói trước.** Cơ
chế: thú nhận đã đệm hay đã ép đều tốn uy tín và mất mặt, đặc biệt trong văn hoá ngại xung đột trực
diện; vì vậy trạng thái mặc định là im lặng. Minh phá được bế tắc vì anh nhận phần lỗi của mình trước
khi hỏi hai người kia — hạ chi phí thú nhận xuống mức trả được. Với Tech Lead, phiên bản nhỏ của việc
này là tự nói ra lớp đệm của mình trước khi đòi Product ngừng cắt. Xem
[03-team-leadership.md](/series/engineering-leadership/03-team-leadership/) và [02-communication.md](/series/engineering-leadership/02-communication/).

---

## Case 7 — "Ship nhanh" hay "làm đúng"

### Bối cảnh

Một công ty fintech Việt Nam: ví điện tử kèm nhánh cho vay tiêu dùng nhỏ, khoảng 30 engineer chia thành bốn team (Payment, Lending, Platform, Data). Giữa tháng, bộ phận compliance nhận được văn bản yêu cầu hoàn thiện một nhóm nghĩa vụ báo cáo và kiểm soát dữ liệu giao dịch trước một mốc cụ thể. Mốc này khác mọi deadline team đã quen:

- Không dời được. Không có ai để thương lượng lùi hạn, không có "phase 2 vào tháng sau".
- Trượt hạn thì hậu quả không nằm ở chỉ số sản phẩm mà ở giấy phép: có khả năng bị đình chỉ một phần nghiệp vụ, kèm xử lý hành chính.
- Hậu quả rơi vào pháp nhân, không rơi vào engineering — nghĩa là người chịu trách nhiệm cuối cùng là ban lãnh đạo, không phải Tech Lead.

Còn **7 tuần**. Ước lượng khối lượng sau khi Tuấn và Linh bóc tách là **11 tuần** — đây là **số minh hoạ**, giữ tỉ lệ thiếu hụt khoảng 36% cho dễ theo dõi.

Các vai: **Minh** (CTO, có accountability kỹ thuật trước ban lãnh đạo, ký quyết định cuối), **Tuấn** (Tech Lead của phần lớn khối lượng — vị trí người đọc nên đứng vào), **Trang** (Head of Product, chủ backlog và các cam kết roadmap bị đụng), **Hà** (EM, chủ capacity), **Linh** (Staff Engineer, người bóc tách và viết bản so sánh phương án), **Quân, Vy, Duy** (engineer thực thi; Duy sẽ tự bỏ test mà không nói với ai).

**Phần bàn về nghĩa vụ pháp lý ở đây là mô tả nguyên tắc quản trị — cách phân vai và ra quyết định — không phải tư vấn pháp lý.** Trong tổ chức thật, xác định một yêu cầu bắt buộc đến đâu là việc của pháp chế và compliance.

Câu hỏi của case không phải "làm sao nhồi 11 tuần vào 7 tuần" — đó là số học và số học không giải được. Câu hỏi là: khi bốn biến của một cam kết bị bóp lại, áp lực **rò rỉ** ra biến nào, ai quyết định cho nó rò rỉ, và làm sao để việc đó thành quyết định được ghi lại thay vì một chuỗi việc xảy ra trong im lặng.

### Triệu chứng ban đầu

Tuần đầu sau khi yêu cầu vào engineering, mọi thứ nhìn bề ngoài rất tốt. Đó là triệu chứng.

**Kickoff nghe rất quyết tâm, nhưng không ai nói con số.** Minh mở buổi họp bằng "việc này bắt buộc phải kịp". Tuấn nhắc con số 11 tuần trong một câu giữa buổi, và câu đó trôi qua. Biên bản ghi "team cam kết đáp ứng hạn": khoảng cách 4 tuần biến mất khỏi văn bản trong 50 phút.

**Ước lượng tự co lại mà scope không đổi.** Ba ngày sau, slide ghi "8–9 tuần, nếu mọi thứ suôn sẻ". Không việc nào bị bỏ ra, không ai được thêm vào — chỉ có tính từ "suôn sẻ" xuất hiện. Khi ước lượng giảm mà không có thay đổi vật lý nào ở scope hoặc nguồn lực, phần giảm đó đang vay từ chất lượng.

**PR to hơn, review nhanh hơn.** Tuần 2, PR trung bình từ khoảng 200 lên khoảng 900 dòng, thời gian mở-đến-approve rút từ khoảng 6 giờ xuống dưới 40 phút (số minh hoạ). Không ai ra lệnh review nhanh hơn; ai cũng hiểu ngầm rằng chặn PR của đồng đội lúc này là không hợp thời điểm.

**Câu "cái này để sau" xuất hiện 14 lần trong tuần 2 (số minh hoạ), không lần nào vào backlog.** Đây là dạng nợ tệ nhất: không tên, không chủ, chỉ tồn tại trong ký ức của người tạo ra nó — và ký ức đó sẽ đi khỏi công ty cùng người đó.

**Team làm cuối tuần mà không ai yêu cầu.** Ba người commit vào Chủ nhật hai tuần liền, không ai báo. Trong bối cảnh Việt Nam, chi tiết này dễ bị đọc là tin tốt. Nó là tin xấu: team đang tự hấp thụ một khoảng cách kế hoạch mà lẽ ra phải được đưa lên trên.

**Câu hỏi của Product đặt sai dạng.** Trang hỏi "có cắt được gì không?" — thiện chí nhưng vô dụng, vì nó bắt người kỹ thuật vừa đề xuất vừa gánh trách nhiệm cho việc cắt.

Triệu chứng của tuần 1–2 không phải "team chậm" — team đang nhanh hơn bình thường. Triệu chứng là: **khoảng cách giữa kế hoạch và thực tế đang bị hấp thụ ở tầng thấp nhất của tổ chức, bằng những quyết định không ai ghi lại.**

### Chẩn đoán sai ban đầu

Hai tuần đầu, tổ chức chẩn đoán sai ba lần, cả ba theo cùng một cách: nhắm vào biến dễ thấy nhất thay vì biến đang thực sự bị bóp.

**"Đây là vấn đề nguồn lực."** Minh đề nghị điều hai engineer từ team Lending sang. Nhưng khối lượng compliance nằm trên phần core ledger mà chỉ Linh và Tuấn hiểu sâu; hai người mới cần khoảng 2 tuần để đủ context (số minh hoạ), và trong 2 tuần đó họ tiêu thời gian của đúng hai người đang là nút thắt. Thêm người vào nút thắt làm nút thắt hẹp hơn — Brooks's Law ở dạng thuần khiết nhất, mà ai trong phòng cũng đã đọc về nó.

**"Đây là vấn đề động lực."** Trong họp lãnh đạo tuần 2 có một câu được thả ra: "team mình chưa đủ sense of urgency". Chẩn đoán này hấp dẫn vì không đòi ai từ bỏ gì. Nó sai về cơ chế: team đang làm cuối tuần, giảm review, bỏ test — urgency không thiếu. Cái thiếu là một quyết định cắt scope mà chỉ người có thẩm quyền mới ra được. Khi tổ chức gọi vấn đề cấu trúc là vấn đề thái độ, kết quả duy nhất là người giỏi làm nhiều hơn cho đến khi họ nghỉ.

**"Chất lượng bù sau, giờ cứ ship đã."** Đây là chẩn đoán sai quan trọng nhất, và cần bóc tách cơ chế bên dưới mới thấy nó sai ở đâu.

#### Bốn biến của một cam kết, và biến duy nhất không có cổng phê duyệt

Mỗi cam kết giao hàng chỉ có bốn biến. Không có biến thứ năm.

| Biến | Ai phải phê duyệt khi thay đổi | Bằng chứng để lại | Thời gian phát hiện |
|---|---|---|---|
| Scope | Head of Product / Business owner; ở đây có thêm pháp chế | Ticket bị đóng, backlog thay đổi, biên bản họp | Ngay lập tức |
| Thời gian | Stakeholder bên ngoài; ở đây là cơ quan quản lý — tức không ai | Thông báo chính thức, roadmap sửa | Ngay lập tức |
| Nguồn lực | EM / CTO, kèm ngân sách | Đơn điều chuyển, hợp đồng, chi phí | Trong vài ngày |
| **Chất lượng** | **Không ai** | **Không có** | **Hàng tháng đến hàng năm** |

Ba biến đầu đều có **cổng phê duyệt**: muốn thay đổi phải có người đồng ý, và việc đồng ý để lại dấu vết. Biến thứ tư không có cổng — một engineer có thể bỏ test lúc 11 giờ đêm, không xin ai, và hệ thống vẫn xanh.

Hệ quả không phụ thuộc vào chất lượng con người: **khi áp lực tăng và ba biến đầu đều bị đóng, áp lực rò rỉ vào biến không có cổng** — không phải vì team thiếu kỷ luật, mà vì đó là đường có điện trở thấp nhất. Với một tổ chức mà mọi lối thoát bị chặn trừ một lối, không cần dự đoán nước chảy đi đâu.

Từ đó suy ra nhiệm vụ đầu tiên của Tech Lead, và nó không phải nhiệm vụ kỹ thuật: **dựng một cổng phê duyệt tạm thời cho biến chất lượng** — biến việc cắt chất lượng từ hành vi cá nhân im lặng thành quyết định có tên người ký. Có cổng đó rồi, áp lực buộc phải quay lại ba biến kia, và đó là lúc scope mới thực sự được đem ra bàn.

#### Chất lượng bên ngoài và chất lượng bên trong

Nói "cắt chất lượng" là một câu vô nghĩa, vì có hai loại chất lượng với hành vi kinh tế khác nhau.

**Chất lượng bên ngoài** là những gì người dùng và cơ quan quản lý quan sát được: số tính năng, giao diện, số chiều lọc của báo cáo, số giao dịch mỗi giây. Cắt loại này là **tiết kiệm** — bỏ một dashboard tuần này thì tuần sau làm lại đúng bằng chi phí ban đầu, khoản không làm không sinh lãi. Nó mất ngay và ai cũng thấy, nên nó luôn được đem ra bàn.

**Chất lượng bên trong** là khả năng tiếp tục thay đổi hệ thống: độ rõ của mô hình dữ liệu, biên giới module, độ phủ của test, khả năng deploy và rollback, khả năng quan sát. Cắt loại này là **vay nợ có lãi kép** — mỗi thay đổi sau đó phải trả thêm phí trên nền đã hỏng, và việc trả nợ cũng đắt dần theo thời gian. Hôm nay không mất gì thấy được, hậu quả rơi vào chính team sau 3–18 tháng, nên nó không bao giờ được đem ra bàn.

Vậy câu hỏi đúng không phải "ship nhanh hay làm đúng", mà gồm hai phần: **cắt chất lượng bên ngoài đến đâu thì business còn chấp nhận được**, và **vay chất lượng bên trong bao nhiêu thì còn trả được, ai ghi sổ khoản vay đó.** Hai tuần đầu đã trôi qua trong việc trả lời sai câu hỏi đó: team cắt loại đắt nhất và không ai phê duyệt, để giữ nguyên loại đáng ra phải mang đi thương lượng trước.

### Các lựa chọn thực tế

Trước khi liệt kê phương án, Tuấn và Linh làm hai việc lẽ ra phải làm từ đầu.

#### Việc thứ nhất: chốt danh sách "không bao giờ cắt"

Trong bối cảnh compliance của một tổ chức tài chính, có một nhóm thuộc tính mà cắt đi không tạo ra tiết kiệm mà tạo ra rủi ro khác về bản chất. Danh sách của Tuấn có bốn mục:

| Mục không cắt | Vì sao không thương lượng được ở fintech |
|---|---|
| Tính đúng của số tiền | Sai số tiền là thiệt hại tài chính, lan ra hàng nghìn tài khoản trước khi bị phát hiện; đối soát và hoàn tiền đắt hơn nhiều lần chi phí làm đúng từ đầu |
| Audit trail | Thường chính là đối tượng của yêu cầu compliance; không có nó thì không chứng minh được đã làm đúng, và về quản trị, không chứng minh được tương đương với không làm |
| Kiểm soát truy cập | Không sửa được hồi tố: dữ liệu đã bị đọc thì không thu về |
| Khả năng rollback | Điều kiện tiên quyết để mọi phương án rủi ro khác chấp nhận được; bỏ rollback là bỏ luôn quyền sửa sai |

Bốn mục này nằm ngoài thương lượng vì **hậu quả của việc cắt chúng không mua lại được bằng tiền hoặc thời gian**, và vì phần lớn chúng là đối tượng trực tiếp của nghĩa vụ tuân thủ. Sức ép từ ban lãnh đạo không đổi được tính chất đó; nó chỉ đổi được việc ai ký tên.

#### Việc thứ hai: tách yêu cầu bắt buộc khỏi phần tổ chức tự diễn giải

Đây là nguồn cắt scope lớn nhất và ít ai nghĩ tới. Khi Linh đối chiếu từng dòng đặc tả nội bộ team đang implement với văn bản yêu cầu gốc, một phần đáng kể khối lượng không đến từ văn bản mà từ cách tổ chức tự đọc rộng ra.

| Hạng mục trong đặc tả nội bộ | Nguồn thực tế | Khối lượng (số minh hoạ) |
|---|---|---|
| Lưu vết giao dịch theo định dạng quy định | Văn bản yêu cầu | 3,0 tuần |
| Kết xuất báo cáo định kỳ theo mẫu | Văn bản yêu cầu | 1,5 tuần |
| Kiểm soát và ghi vết truy cập dữ liệu khách hàng | Văn bản yêu cầu | 1,5 tuần |
| Dashboard nội bộ cho compliance tra cứu theo 9 chiều lọc | Tổ chức tự thêm cho tiện | 2,0 tuần |
| Tự động hoá luồng gửi báo cáo thay vì gửi thủ công | Tổ chức tự nâng chuẩn | 1,5 tuần |
| Mở rộng cơ chế sang ba luồng chưa thuộc phạm vi | Tổ chức tự mở rộng | 1,5 tuần |
| **Tổng** | | **11,0 tuần** |

Sáu tuần đầu là nghĩa vụ; năm tuần sau là cách tổ chức tự diễn giải rộng hơn mức cần. Tỉ lệ là số minh hoạ, nhưng dạng phân bố lặp lại ở nhiều dự án compliance thật: phần "cho chắc" thường không nhỏ hơn phần bắt buộc.

Ba điều kiện để việc bóc tách này an toàn:

1. **Engineering không được tự kết luận.** Bảng trên là **giả thuyết**. Mỗi dòng phải được pháp chế và compliance xác nhận bằng văn bản — email cũng được, miễn là của người có thẩm quyền — trước khi bị bỏ khỏi phạm vi. Một Tech Lead tự quyết "dòng này không bắt buộc" đang lấy rủi ro pháp lý của công ty làm vật thế chấp cho tiến độ của mình.
2. **Phần cắt phải ghi rõ là hoãn, không phải huỷ.** Dashboard 9 chiều lọc vẫn cần cho compliance, chỉ là không cần trước hạn. Cắt mà không ghi lại thì sau này nó quay về dưới dạng một yêu cầu gấp khác.
3. **Phải có phương án thủ công cho phần bị hoãn:** ai kết xuất, ai kiểm, ai gửi, mất bao lâu mỗi kỳ, và bộ phận đó đồng ý gánh việc tay trong bao lâu. Không có câu trả lời này thì "cắt scope" chỉ là đẩy việc sang phòng khác mà không hỏi họ.

#### Sức ép xuất hiện: cuộc trao đổi về audit trail

Giữa tuần 3, trong buổi họp có hai thành viên ban lãnh đạo, một đề xuất được đưa ra: phần ghi vết truy cập chiếm 1,5 tuần, "cứ log ra file trước, làm chuẩn sau khi qua hạn".

Phiên bản Tuấn nói sai:

> **Lãnh đạo:** Cái audit trail này bỏ tạm được không? Log tạm ra file, qua hạn làm lại đúng. Đang thiếu 4 tuần, chỗ nào cắt được thì phải cắt.
>
> **Tuấn:** Không được đâu ạ. Cái đó là bắt buộc, bỏ là không được.
>
> **Lãnh đạo:** Bắt buộc theo ai? Anh thấy nhiều nơi vẫn làm sau mà.
>
> **Tuấn:** Về kỹ thuật thì em thấy rủi ro cao lắm ạ. Không nên làm thế.
>
> **Lãnh đạo:** Việc gì chẳng có rủi ro. Em cứ tìm cách đi, anh tin em làm được.

Đoạn này thất bại dù Tuấn đúng về nội dung, vì ba lỗi cơ chế:

- Tuấn đặt cuộc trao đổi thành **đối đầu giữa hai cá nhân**: ý kiến một Tech Lead chống ý kiến một lãnh đạo. Trong cấu hình đó, ai có chức vụ cao hơn sẽ thắng bất kể nội dung — ở môi trường Việt Nam càng bất lợi cho người trẻ hơn.
- Tuấn nói "không được" mà không nêu **ai có quyền nói được hoặc không được**. Một lời cấm không có nguồn thẩm quyền chỉ là sở thích cá nhân.
- Cuộc họp không ra quyết định nào; Tuấn về chỗ với chỉ thị "cứ tìm cách", tức khoảng cách 4 tuần vẫn còn nguyên và giờ nằm trên vai anh dưới dạng một lời hứa mơ hồ.

Phiên bản đúng — chuyển từ đối đầu cá nhân sang câu hỏi về thẩm quyền:

> **Lãnh đạo:** Cái audit trail này bỏ tạm được không? Log tạm ra file, qua hạn làm lại đúng.
>
> **Tuấn:** Em làm được về mặt kỹ thuật, và nó tiết kiệm khoảng 1,5 tuần. Nhưng em không phải người có quyền quyết việc này. Theo đặc tả compliance gửi cho mình, phần ghi vết truy cập nằm trong nhóm nghĩa vụ, không phải nhóm nội bộ tự thêm. Nếu đổi cách làm phần đó, em cần xác nhận bằng văn bản từ chị phụ trách compliance rằng log ra file vẫn đáp ứng yêu cầu. Có xác nhận đó thì em làm ngay trong tuần này.
>
> **Lãnh đạo:** Thủ tục thế thì mất thời gian.
>
> **Tuấn:** Em nhắn chị ấy ngay bây giờ, hôm nay hoặc mai có trả lời. Chị ấy nói được thì em cắt. Chị ấy nói không thì 1,5 tuần này phải tìm ở chỗ khác, và em đã có ba chỗ khác để đề xuất — em gửi anh bản so sánh chiều nay. Chỗ em cần anh quyết là ba chỗ đó, không phải chỗ này.
>
> **Lãnh đạo:** Ba chỗ nào?
>
> **Tuấn:** Dashboard 9 chiều lọc, tự động hoá gửi báo cáo, và mở rộng sang ba luồng chưa thuộc phạm vi — cộng lại 5 tuần, cả ba đều là phần mình tự thêm. Nhưng cắt cái nào thì ai đó sẽ phải làm tay, nên cần anh và chị Trang chọn.

Khác biệt ở cơ chế, không ở thái độ:

- Tuấn **không từ chối**; anh xác nhận việc khả thi rồi nêu giá, và gắn thời hạn cho việc xác nhận nên không bị đọc là trì hoãn.
- Anh chuyển câu hỏi từ "Tuấn có đồng ý không" sang **"ai có thẩm quyền đồng ý, và bằng văn bản nào"** — cấu hình này không so thâm niên, nên không thua vì thâm niên.
- Anh **mang theo chỗ khác để cắt**. Người nói "không" mà không mang giải pháp thì bị xem là vật cản; người nói "chỗ này không cắt được, nhưng đây là ba chỗ cắt được, mời anh chọn" đang làm việc của một Tech Lead.

#### Ba phương án được đặt lên bàn

Sau khi có bảng phân loại phạm vi và xác nhận sơ bộ từ compliance, Linh viết ba phương án. Chỉ ba — hai phương án là ép chọn, năm phương án là đẩy việc phân tích sang người đọc.

- **A — Giữ nguyên scope, dồn giờ.** Bù 4 tuần bằng làm thêm giờ và cuối tuần, giảm review, giảm test, hoãn mọi việc khác của ba team.
- **B — Cắt scope theo phân loại nghĩa vụ, giữ tiêu chuẩn kỹ thuật.** Bỏ khỏi phạm vi trước hạn ba hạng mục tổ chức tự thêm (5 tuần), giữ 6 tuần nghĩa vụ, 1 tuần còn lại làm đệm; phần bỏ được xử lý thủ công tạm thời và ghi vào nợ có hạn trả.
- **C — Giữ scope, hạ tiêu chuẩn có chủ ý và có ghi sổ.** Làm cả 11 tuần khối lượng nhưng chấp nhận nợ ở phần ngoài danh sách không cắt: bỏ test tích hợp luồng phụ, hard-code cấu hình, dashboard truy vấn trực tiếp.

### Trade-off của từng lựa chọn

| Tiêu chí | A — Dồn giờ | B — Cắt scope | C — Hạ tiêu chuẩn có ghi sổ |
|---|---|---|---|
| Xác suất đúng hạn (số minh hoạ) | 40% | 85% | 60% |
| Rủi ro pháp lý nếu trượt | Cao — trượt là trượt toàn bộ | Thấp — nghĩa vụ xong trước, có đệm 1 tuần | Trung bình — nhiều mặt trận cùng mỏng |
| Nợ kỹ thuật phát sinh | Cao và **vô hình** | Gần bằng không | Cao nhưng **có tên** |
| Nợ vận hành | Không | Có — compliance làm tay một số việc | Ít |
| Ai chịu phần mất | Engineer, ngay và về sau | Compliance và Product, ngay | Engineer, quý sau |
| Chi phí ẩn lớn nhất | Mất người hiểu ledger | Quan hệ với phòng ban khác | Nợ bị quên nếu không có cơ chế thu |
| Ai phải phê duyệt | Không ai — nên nó là mặc định | Head of Product + compliance | CTO |
| Điều kiện nên chọn | Khoảng cách dưới 10%, một lần duy nhất | Có phần scope tự diễn giải để cắt | Scope đã tối giản mà vẫn thiếu |

Ba nhận xét về bảng này:

**A là phương án tệ nhất và luôn là phương án mặc định** — chính vì nó không cần ai phê duyệt. Không ai họp, không ai ký, nó xảy ra bằng cách không quyết định gì. Đây là lý do một tổ chức trôi vào A trong khi không ai trong tổ chức thực sự chọn A.

**B rẻ nhất về tổng chi phí nhưng đắt nhất về chi phí xã hội:** cắt scope nghĩa là có người phải nghe "việc anh cần sẽ chậm" hoặc "chị phải làm tay ba kỳ báo cáo". Phần lớn team chọn A không vì tính sai, mà vì tránh những cuộc trò chuyện đó.

**C chỉ chấp nhận được khi kèm cơ chế thu nợ** — C không có ghi sổ thì bản chất là A với vỏ bọc từ ngữ.

### Quyết định và cách thực thi

Quyết định cuối: **B là phần chính, cộng một phần C nhỏ có ghi sổ** cho hai chỗ mà làm chuẩn không thể xong trước hạn. Người ký là Minh, sau khi Trang xác nhận thứ tự ưu tiên và compliance xác nhận bằng email rằng ba hạng mục bị hoãn không thuộc nghĩa vụ theo văn bản.

Về quy trình, bản so sánh được **gửi trước cuộc họp 24 giờ**, dạng văn bản, không phải slide trình bày trực tiếp. Cơ chế: khi phương án được đọc lần đầu ngay trong họp, người quyết định phải vừa hiểu vấn đề vừa quyết dưới áp lực thời gian, và trong trạng thái đó người ta chọn phương án nghe êm nhất — thường là A. Đọc trước tách hai việc đó ra.

#### Template "ba phương án"

```
QUYẾT ĐỊNH CẦN: <một câu, dạng câu hỏi có thể trả lời bằng chọn A/B/C>
NGƯỜI QUYẾT: <tên, vai — người có accountability, không phải người đề xuất>
HẠN QUYẾT: <ngày> — sau ngày này chi phí đổi ý tăng vì <lý do cụ thể>

TÌNH TRẠNG
- Khối lượng còn: <X tuần, ước lượng>   Thời gian còn: <Y tuần>
- Khoảng cách <X-Y>; thêm người KHÔNG giảm khoảng cách vì <lý do>

KHÔNG CẮT (ngoài phạm vi thương lượng của tài liệu này)
- Tính đúng của số tiền | Audit trail | Kiểm soát truy cập | Khả năng rollback
Cơ sở: <xác nhận của compliance/pháp chế, kèm ngày và người xác nhận>
Muốn đổi bất kỳ mục nào ở đây cần: xác nhận bằng văn bản của <vai có thẩm quyền>

PHÂN LOẠI SCOPE
- Bắt buộc theo văn bản (compliance xác nhận ngày <...>): <danh sách, tổng tuần>
- Tổ chức tự diễn giải thêm: <danh sách, tổng tuần>   ← nguồn cắt chính

PHƯƠNG ÁN A / B / C — mỗi phương án đủ 6 dòng:
1. Làm gì (một câu)
2. Xác suất đúng hạn (ước lượng)
3. Được gì
4. Mất gì — và AI chịu phần mất (tên bộ phận, không viết "team")
5. NỢ PHÁT SINH: <mục nợ> | chủ nợ | hạn trả | chi phí trả | rủi ro nếu không trả
6. Điều kiện để phương án này là lựa chọn đúng

ĐỀ XUẤT CỦA ENGINEERING: <A/B/C> vì <một câu>
CÁI TÔI KHÔNG BIẾT: <liệt kê thật — chỗ để người quyết bổ sung thông tin>
NẾU KHÔNG QUYẾT: mặc định sẽ là <thường là A>, do <cơ chế>, và không ai ký
```

Dòng cuối làm việc nhiều nhất: nêu rõ hậu quả của việc không quyết biến sự im lặng thành một lựa chọn có tên, và ít ai muốn để tên mình cạnh nó.

#### Minh trình bày với ban lãnh đạo

> **Minh:** Em chốt phương án B: mình vẫn kịp hạn, đủ nghĩa vụ, nhưng ba việc sẽ chậm và có hai chỗ mình cố tình làm chưa chuẩn.
>
> **Lãnh đạo:** Ba việc nào chậm?
>
> **Minh:** Dashboard tra cứu 9 chiều cho compliance, tự động hoá gửi báo cáo, và mở rộng sang ba luồng chưa thuộc phạm vi. Cả ba là phần mình tự thêm — compliance đã xác nhận bằng email hôm thứ Ba rằng chúng không thuộc nghĩa vụ phải có trước hạn; em để email đó ở trang 2.
>
> **Lãnh đạo:** Vậy ba tháng tới compliance phải làm tay?
>
> **Minh:** Đúng. Chị phụ trách đã đồng ý, giới hạn ba kỳ báo cáo, mỗi kỳ khoảng bốn giờ người (số minh hoạ). Quá ba kỳ mà mình chưa tự động hoá thì đó là lỗi của em, không phải của chị ấy.
>
> **Lãnh đạo:** Còn hai chỗ "cố tình làm chưa chuẩn"? Nghe không an tâm.
>
> **Minh:** Không nằm trong bốn thứ mình không cắt: số tiền vẫn đúng, audit trail vẫn đủ, phân quyền vẫn đủ, rollback vẫn có — bốn thứ đó em không đưa vào bàn. Hai chỗ kia là test tích hợp một luồng phụ và một lớp cấu hình đang hard-code. Em ghi thành hai mục nợ, mỗi mục có tên người chịu và hạn trả trong quý sau, tổng 3 tuần. Em xin luôn 3 tuần đó trong capacity quý sau và xin anh duyệt hôm nay cùng phương án — để quý sau mới xin thì nó sẽ phải cạnh tranh với roadmap và sẽ thua.
>
> **Lãnh đạo:** Được. Anh muốn thấy tiến độ trả nợ.
>
> **Minh:** Ba dòng trong báo cáo hai tuần một lần, cùng chỗ với các chỉ số khác. Trượt hạn hai kỳ liên tiếp thì em báo anh, không đợi anh hỏi.

Hai chi tiết đáng học: Minh **xin capacity trả nợ cùng lúc với quyết định vay**, và **nhận trách nhiệm về phần việc thủ công** thay cho compliance — bảo đảm chi phí của việc cắt scope không rơi im lặng vào phòng khác.

#### Template ghi nợ

```
DEBT-<số>: <tên ngắn, dạng danh từ — phải đọc là biết sửa cái gì>
Nguồn: quyết định <mã>, ngày <...>, người ký <...>
Vay để: <việc gì đã kịp nhờ khoản vay này>

Hiện trạng    : <hệ thống đang thế nào>
Đích          : <thế nào là trả xong — tiêu chí kiểm được, không viết "cải thiện">
Chủ nợ        : <MỘT tên người, không phải tên team>
Người bảo lãnh: <Tech Lead/EM — người báo cáo nếu chủ nợ nghỉ/chuyển việc>
Hạn trả       : <quý, sprint cụ thể>   Chi phí trả: <X tuần người, ước lượng>
Chỗ trong kế hoạch: <sprint đã cam kết bằng văn bản — trống thì đây là mong ước, không phải nợ>

Lãi nếu không trả: <hiện tượng quan sát được, ví dụ: mỗi thay đổi ở luồng
  thanh toán mất thêm ~1 ngày kiểm thủ công>
ĐIỀU KIỆN LEO THANG: nếu <trượt hạn 2 kỳ liên tiếp, hoặc lãi vượt
  3 ngày/tháng> thì tự động báo <CTO>, không đợi được hỏi
Ai đọc tiến độ: <tên người ở tầng trên cùng> — mỗi <2 tuần>, trong <báo cáo nào>
```

#### Cách thực thi trong 7 tuần

| Cơ chế | Nội dung | Vấn đề nó chặn |
|---|---|---|
| Trạng thái hai màu | Mỗi hạng mục chỉ "xong theo tiêu chuẩn" hoặc "chưa xong" | "90% xong" là cách khoảng cách bị ẩn tới tuần cuối |
| Cổng chất lượng tạm | Cắt chất lượng bên trong thì phải mở mục nợ trước khi merge | Việc cắt chất lượng diễn ra không dấu vết |
| Freeze phạm vi | Thêm gì vào phạm vi phải bỏ ra một thứ tương đương, do Trang ký | Scope creep dưới danh nghĩa "cho chắc" |
| Chống người hùng | Hai tuần liền commit Chủ nhật là chủ đề trong 1-1 với Hà | Team tự hấp thụ khoảng cách trong im lặng |

### Hậu quả

**Đúng hạn, phạm vi đã cắt.** Phần nghĩa vụ hoàn thành giữa tuần 6 (số minh hoạ), dùng bốn ngày của tuần đệm để xử lý hai lỗi đối soát phát hiện muộn; không có sự cố production sau khi ship. **Chi phí thật đã trả:** compliance mất khoảng 12 giờ người làm tay (số minh hoạ), một cam kết roadmap lùi khoảng năm tuần, và hai mục nợ kỹ thuật với chi phí trả 3 tuần người.

**Một việc ngoài kế hoạch: Duy tự bỏ test.** Tuần 5, để kịp một hạng mục, Duy tắt một nhóm test tích hợp đang đỏ ở luồng phụ, merge, không nói với ai, không mở mục nợ nào. Việc này bị phát hiện tuần 8 khi một lỗi nhỏ lọt xuống staging.

Cùng một sự việc, ba tầng đọc ra ba vấn đề khác nhau — và cả ba đều đúng ở tầng của mình:

| Góc nhìn | Sự việc là gì | Việc cần làm |
|---|---|---|
| **IC (Duy)** | "Test đó vốn hay đỏ vì môi trường. Tôi kịp hạn, không gây lỗi cho khách. Nếu báo, tôi sẽ bị hỏi tại sao chậm, mà giải thích thì mất nửa ngày." | Tối ưu cục bộ hợp lý ở tầng của Duy. Vấn đề không phải đạo đức mà là **thiếu một đường báo giá rẻ**: báo thì tốn nửa ngày, không báo thì miễn phí. |
| **Tech Lead (Tuấn)** | "Tôi mất khả năng biết trạng thái thật của hệ thống. Từ tuần 5 đến tuần 8, mọi quyết định của tôi dựa trên một bảng xanh không đúng thực tế." | Chữa hệ thống, không chữa người: mở mục nợ tốn 2 phút thay vì nửa ngày, và cho merge kèm nợ. Nếu con đường đúng đắt hơn con đường sai, người ta chọn con đường sai — đó là lỗi thiết kế. |
| **Manager (Hà)** | "Một engineer đã tính rằng nói ra sẽ bị phạt. Đó là chỉ số về môi trường tôi tạo ra, không phải về Duy." | Truy xem tín hiệu "phạt khi báo chậm" đến từ đâu — thường từ cách quản lý phản ứng ở lần trước. Xử lý Duy như trường hợp cá biệt sẽ dạy cả team rằng bài học là "đừng để bị phát hiện". |

Cách xử lý đã dùng: Postmortem blameless, không có hậu quả cá nhân với Duy, kèm một thay đổi quy trình — thêm lệnh mở mục nợ trong CI, chạy 30 giây, tự gán chủ nợ là người merge. Bốn tuần sau có 9 mục nợ được mở tự nguyện (số minh hoạ); chín mục đó vốn đã tồn tại, chỉ là trước đây không có tên.

**Nợ được trả trong quý kế tiếp.** Hai mục nợ ghi sổ được trả xong trong quý sau, trượt hai tuần so với hạn ban đầu và việc trượt đó được báo trước theo điều kiện leo thang. Ba hạng mục scope bị hoãn cũng được làm trong cùng quý.

### Bài học

**1. Chất lượng là biến duy nhất không có cổng phê duyệt, nên việc của Tech Lead là dựng cổng đó.** Cơ chế: áp lực chảy theo đường điện trở thấp nhất; khi scope, thời gian và nguồn lực đều cần người ký mà chất lượng thì không, áp lực rò rỉ vào chất lượng bất kể team giỏi đến đâu. Cổng không cần nặng — quy tắc "cắt chất lượng bên trong thì phải mở mục nợ trước khi merge" là đủ, vì nó biến hành vi vô hình thành hành vi có dấu vết. Xem [05-technical-leadership.md](/series/engineering-leadership/05-technical-leadership/), [12-anti-patterns.md](/series/engineering-leadership/12-anti-patterns/).

**2. Nguồn cắt scope lớn nhất nằm trong cách tổ chức tự diễn giải yêu cầu, không nằm trong yêu cầu.** Cơ chế: mỗi lớp tổ chức khi đọc yêu cầu đều thêm một biên an toàn riêng, và các biên cộng dồn mà không ai thấy tổng — ở đây là 5 trên 11 tuần (số minh hoạ). Điều kiện để dùng an toàn: engineering chỉ lập giả thuyết, phạm vi pháp lý phải do pháp chế và compliance xác nhận bằng văn bản. Đây là nguyên tắc quản trị, không phải tư vấn pháp lý. Xem [07-project-delivery.md](/series/engineering-leadership/07-project-delivery/), [04-decision-making.md](/series/engineering-leadership/04-decision-making/).

**3. Khi bị ép cắt một thứ không cắt được, chuyển từ đối đầu cá nhân sang câu hỏi về thẩm quyền.** Cơ chế: cấu hình "ý kiến tôi chống ý kiến anh" được phân xử bằng chức vụ và Tech Lead luôn thua, nhất là trong môi trường coi trọng thâm niên; cấu hình "việc này cần ai đồng ý bằng văn bản" được phân xử bằng thẩm quyền. Điều kiện kèm theo: phải mang theo chỗ khác để cắt, nếu không thì đây chỉ là thủ tục để nói không. Xem [02-communication.md](/series/engineering-leadership/02-communication/), [00-nen-tang-leadership.md](/series/engineering-leadership/00-nen-tang-leadership/).

**4. Ba phương án bằng văn bản gửi trước là cách trả quyết định về đúng người có accountability.** Cơ chế: hậu quả pháp lý rơi vào pháp nhân nên quyền chọn thuộc ban lãnh đạo; nhưng nếu engineering không viết ra phương án thì họ không có gì để chọn ngoài "phải kịp", và câu đó tự thành phương án A. Khi nào không dùng: sự cố đang diễn ra — lúc đó cần một người quyết ngay rồi giải thích sau. Xem [04-decision-making.md](/series/engineering-leadership/04-decision-making/), [06-incident-va-metrics.md](/series/engineering-leadership/06-incident-va-metrics/).

**5. Vì sao nợ lần này được trả, còn nợ ở Case 2 thì tích tụ nhiều năm.** Cùng một loại quyết định vay, hai kết cục trái ngược; khác biệt không ở chất lượng con người mà ở bốn cơ chế:

| Cơ chế | Case 7 — nợ được trả | Case 2 (e-commerce) — nợ tích tụ nhiều năm |
|---|---|---|
| Tên và chủ | Hai mục có mã, có tiêu chí "trả xong", mỗi mục một tên người kèm người bảo lãnh | Nợ tồn tại dạng hiểu biết chung "code phần đó tệ lắm"; không mục nào có chủ, nên không ai trượt hạn |
| Chỗ trong capacity | 3 tuần người được duyệt **cùng lúc với quyết định vay**, ghi vào kế hoạch quý sau bằng văn bản | Trả nợ được hứa "khi nào rảnh"; mỗi quý phải cạnh tranh với roadmap và luôn thua |
| Điều kiện leo thang | Trượt 2 kỳ liên tiếp thì tự động báo CTO | Không có ngưỡng nào; nợ chỉ được nhắc sau sự cố rồi lại quên |
| Người tầng trên đọc tiến độ | Ba dòng trong báo cáo hai tuần một lần, gửi người đã ký quyết định vay | Không ai ngoài engineering biết khoản nợ tồn tại, nên không có áp lực từ trên |

Rút ra: **nợ kỹ thuật không được trả bằng ý chí của team mà bằng bốn thứ — tên, chủ, chỗ trong kế hoạch, và một người ở tầng trên đọc tiến độ.** Thiếu bất kỳ thứ nào, khoản vay chuyển thành nợ vĩnh viễn. Quy tắc thực hành: **xin capacity trả nợ tại đúng thời điểm xin phê duyệt vay**, vì sau khi đã ship, khoản nợ mất hết đòn bẩy. Xem [09-to-chuc-va-scaling.md](/series/engineering-leadership/09-to-chuc-va-scaling/), [05-technical-leadership.md](/series/engineering-leadership/05-technical-leadership/), [12-anti-patterns.md](/series/engineering-leadership/12-anti-patterns/).

---

## Bảng tra cứu case theo triệu chứng

Bảng này dùng để đi từ **hiện tượng bạn đang quan sát được** tới case gần nhất, rồi tới chương lý
thuyết. Thứ tự đọc được khuyến nghị là: triệu chứng → case → chương, chứ không phải ngược lại — vì
đọc lý thuyết trước khi có một tình huống cụ thể trong đầu thường tạo ra cảm giác hiểu mà không tạo
ra khả năng áp dụng.

| Triệu chứng bạn đang thấy | Case nên đọc | Chương lý thuyết |
|---|---|---|
| Khách hàng nghiệm thu và phát hiện hệ thống hiểu sai yêu cầu | Case 1 | [`02-communication.md`](/series/engineering-leadership/02-communication/) |
| Ai cũng "dạ vâng" trong họp nhưng triển khai ra kết quả khác | Case 1, Case 6 | [`02-communication.md`](/series/engineering-leadership/02-communication/), [`03-team-leadership.md`](/series/engineering-leadership/03-team-leadership/) |
| Tin xấu chỉ đến tai bạn khi đã quá muộn để ứng phó | Case 1, Case 5 | [`03-team-leadership.md`](/series/engineering-leadership/03-team-leadership/), [`07-project-delivery.md`](/series/engineering-leadership/07-project-delivery/) |
| Mọi thay đổi nhỏ trong một module đều mất gấp ba lần dự kiến | Case 2 | [`05-technical-leadership.md`](/series/engineering-leadership/05-technical-leadership/) |
| Đề xuất refactor luôn thua trong tranh luận vì "không có số" | Case 2, Case 7 | [`04-decision-making.md`](/series/engineering-leadership/04-decision-making/), [`05-technical-leadership.md`](/series/engineering-leadership/05-technical-leadership/) |
| Tách service rồi mà vẫn phải deploy đồng thời hai service | Case 3 | [`09-to-chuc-va-scaling.md`](/series/engineering-leadership/09-to-chuc-va-scaling/) |
| Ba team giẫm chân nhau trong cùng một codebase | Case 3 | [`09-to-chuc-va-scaling.md`](/series/engineering-leadership/09-to-chuc-va-scaling/) |
| Chi phí vận hành tăng nhưng chưa thấy lợi ích của việc tách | Case 3 | [`04-decision-making.md`](/series/engineering-leadership/04-decision-making/), [`06-incident-va-metrics.md`](/series/engineering-leadership/06-incident-va-metrics/) |
| Headcount tăng gấp ba mà throughput không đổi | Case 4 | [`09-to-chuc-va-scaling.md`](/series/engineering-leadership/09-to-chuc-va-scaling/) |
| Mọi quyết định kỹ thuật vẫn phải chờ một người duyệt | Case 4 | [`00-nen-tang-leadership.md`](/series/engineering-leadership/00-nen-tang-leadership/), [`03-team-leadership.md`](/series/engineering-leadership/03-team-leadership/) |
| Vừa promote hai người lên lead, một người ổn một người không | Case 4 | [`08-hiring-va-phat-trien.md`](/series/engineering-leadership/08-hiring-va-phat-trien/), [`11-career-evolution.md`](/series/engineering-leadership/11-career-evolution/) |
| Người mới vào nhiều hơn người cũ, văn hoá loãng dần | Case 4 | [`09-to-chuc-va-scaling.md`](/series/engineering-leadership/09-to-chuc-va-scaling/), [`08-hiring-va-phat-trien.md`](/series/engineering-leadership/08-hiring-va-phat-trien/) |
| Incident mà hai người sửa song song không biết về nhau | Case 5 | [`06-incident-va-metrics.md`](/series/engineering-leadership/06-incident-va-metrics/) |
| Câu hỏi đầu tiên sau sự cố là "ai deploy" | Case 5 | [`06-incident-va-metrics.md`](/series/engineering-leadership/06-incident-va-metrics/), [`12-anti-patterns.md`](/series/engineering-leadership/12-anti-patterns/) |
| Alert đã bị tắt vì ồn và không ai bật lại | Case 5 | [`06-incident-va-metrics.md`](/series/engineering-leadership/06-incident-va-metrics/) |
| Cấp trên nhảy vào channel lúc đang xử lý sự cố | Case 5 | [`02-communication.md`](/series/engineering-leadership/02-communication/), [`06-incident-va-metrics.md`](/series/engineering-leadership/06-incident-va-metrics/) |
| Ước lượng của team không còn ai tin, kể cả chính team | Case 6 | [`07-project-delivery.md`](/series/engineering-leadership/07-project-delivery/) |
| Product và Engineering gửi email cc sếp cho nhau | Case 6 | [`02-communication.md`](/series/engineering-leadership/02-communication/), [`09-to-chuc-va-scaling.md`](/series/engineering-leadership/09-to-chuc-va-scaling/) |
| Thêm họp sync mà tình hình phối hợp xấu hơn | Case 6 | [`09-to-chuc-va-scaling.md`](/series/engineering-leadership/09-to-chuc-va-scaling/) |
| Ba quý liên tiếp trượt cam kết roadmap | Case 6, Case 7 | [`07-project-delivery.md`](/series/engineering-leadership/07-project-delivery/) |
| Deadline cứng, khối lượng vượt xa thời gian còn lại | Case 7 | [`07-project-delivery.md`](/series/engineering-leadership/07-project-delivery/), [`04-decision-making.md`](/series/engineering-leadership/04-decision-making/) |
| Bị ép cắt thứ mà bạn tin là không được cắt | Case 7 | [`00-nen-tang-leadership.md`](/series/engineering-leadership/00-nen-tang-leadership/), [`02-communication.md`](/series/engineering-leadership/02-communication/) |
| Có người tự cắt test và không nói với ai | Case 5, Case 7 | [`03-team-leadership.md`](/series/engineering-leadership/03-team-leadership/), [`07-project-delivery.md`](/series/engineering-leadership/07-project-delivery/) |
| Nợ kỹ thuật hứa "dọn sau" nhưng chưa bao giờ dọn | Case 2 vs Case 7 | [`05-technical-leadership.md`](/series/engineering-leadership/05-technical-leadership/), [`07-project-delivery.md`](/series/engineering-leadership/07-project-delivery/) |

**Ba cặp case nên đọc cạnh nhau**, vì phần học nằm ở chỗ so sánh:

- **Case 2 và Case 7** — cùng là vay nợ kỹ thuật dưới áp lực. Ở Case 7 nợ được trả, ở Case 2 nợ tích
  tụ sáu năm. Khác biệt không nằm ở kỷ luật cá nhân mà ở bốn cơ chế cụ thể (nợ có tên và có chủ, có
  chỗ trong capacity đã cam kết bằng văn bản, có điều kiện leo thang, có người ở tầng trên đọc tiến
  độ trả nợ). Đây là ví dụ rõ nhất trong cả chương về việc *cùng một hành động kỹ thuật cho hai kết
  cục khác nhau tuỳ vào cơ chế quản trị bao quanh nó*.
- **Case 3 và Case 4** — cùng là hệ quả của Conway's Law, nhìn từ hai phía. Case 3 đổi hệ thống để
  khớp với tổ chức; Case 4 đổi tổ chức mà không kịp đổi hệ thống. Đọc cạnh nhau để thấy rằng hai
  quyết định này thực chất là một quyết định.
- **Case 1 và Case 6** — cùng là thất bại truyền tin, nhưng ở hai loại ranh giới khác nhau: Case 1
  qua ranh giới tổ chức (khách hàng – nhà cung cấp, kèm rào cản ngôn ngữ), Case 6 qua ranh giới chức
  năng bên trong cùng công ty. Cùng một cơ chế gốc: người biết sự thật không có động lực nói ra nó.

---

## Cách dùng case study để huấn luyện team

Case study đọc một mình có giá trị; chạy thành một buổi với team có giá trị cao hơn nhiều, vì phần
học chính nằm ở **sự khác biệt giữa các lựa chọn của những người khác nhau trong cùng một phòng**.
Dưới đây là quy trình 60 phút đã dùng được với nhóm 4–10 người.

**Chuẩn bị (người dẫn, 20 phút trước buổi).**

- Chọn một case gần với vấn đề team đang gặp, nhưng **không phải chính vấn đề đó** — nếu quá gần,
  mọi người sẽ phòng vệ và bảo vệ lựa chọn hiện tại của mình thay vì suy nghĩ.
- Cắt file thành hai phần: phần A gồm mục 1 đến mục 5 (Bối cảnh → Trade-off), phần B gồm mục 6 đến
  mục 8. Gửi trước **chỉ phần A**, kèm yêu cầu đọc trước buổi.
- Chuẩn bị một câu hỏi mở duy nhất để mở màn, không phải một bài giảng.

**Chạy buổi (60 phút).**

| Thời lượng | Hoạt động | Quy tắc |
|---|---|---|
| 5 phút | Người dẫn nhắc lại bối cảnh và ràng buộc, không nhắc phương án | Không tiết lộ mình nghĩ gì |
| 10 phút | **Mỗi người viết độc lập**: chọn một phương án + một câu "tôi chọn X vì tin rằng Y" + một điều mình sẽ làm trong 48 giờ đầu | Viết trước khi nói — đây là quy tắc quan trọng nhất của cả buổi |
| 10 phút | Lần lượt đọc to lựa chọn của mình, không tranh luận, không bình luận | Người ít thâm niên nhất nói trước |
| 15 phút | Thảo luận: chỗ nào các lựa chọn khác nhau, và **giả định nào tạo ra khác biệt đó** | Câu hỏi dẫn: "bạn đang tin điều gì mà tôi không tin?" |
| 10 phút | Đọc phần B (quyết định thật + hậu quả) | Đọc, chưa đánh giá |
| 10 phút | Rút cơ chế: điều gì trong case này đã có mặt trong tổ chức chúng ta? Chọn **một** việc để làm | Một việc, có chủ, có ngày |

**Bốn quy tắc quyết định chất lượng buổi.**

1. **Viết trước khi nói.** Nếu người có vị thế cao nhất nói đầu tiên, phòng họp bị neo và bạn mất
   toàn bộ giá trị của việc có nhiều góc nhìn (cơ chế information cascade, xem
   [`04-decision-making.md`](/series/engineering-leadership/04-decision-making/)).
2. **Người dẫn không được là người có câu trả lời.** Nếu bạn dẫn buổi để mọi người đi tới kết luận
   bạn đã có, họ sẽ nhận ra trong mười phút và buổi học biến thành một buổi truyền đạt có vỏ bọc.
   Nếu bạn thực sự đã có kết luận, hãy nói ngay từ đầu và biến nó thành một buổi trình bày trung thực.
3. **Không kết luận rằng quyết định trong case là đúng.** Case cho biết một chuỗi quyết định và hậu
   quả của nó trong một bối cảnh; nó không chứng minh phương án đó tối ưu. Nhắc lại phân biệt
   *process quality* và *outcome quality*: mục tiêu của buổi là cải thiện chất lượng quá trình suy
   nghĩ, không phải học thuộc kết quả.
4. **Kết thúc bằng một hành động, không bằng một danh sách.** Một việc có chủ và có ngày thay đổi tổ
   chức; năm việc không ai sở hữu thì không.

**Ba biến thể hữu ích.**

- *Cho người mới lên lead*: yêu cầu họ dẫn buổi thay vì tham gia. Việc phải giữ cho phòng họp không
  bị neo, phải nghe hết trước khi tổng hợp, và phải chốt được một hành động — đó chính là bài tập
  của vai trò lead, ở môi trường rủi ro thấp.
- *Cho hai team đang có xung đột phối hợp*: chạy Case 6 với người của cả hai team trong cùng phòng,
  và yêu cầu mỗi người chọn phương án **thay cho phía bên kia**. Bài tập này thường tạo ra khoảnh
  khắc nhận ra rằng phía bên kia đang tối ưu một hàm mục tiêu hợp lý.
- *Cho buổi onboarding*: dùng Case 5 để giới thiệu quy trình incident thật của tổ chức, so sánh cái
  case làm sai với cái tổ chức mình đang có. Người mới nhớ quy trình qua một câu chuyện tốt hơn nhiều
  so với qua một trang wiki.

Cuối cùng: **hãy viết case study của chính tổ chức bạn.** Sau mỗi sự việc đáng học — một dự án trễ,
một incident lớn, một lần tái cấu trúc — dành 90 phút viết theo đúng tám phần của chương này, đặc
biệt là mục 3 (Chẩn đoán sai ban đầu) và mục 4 (Các lựa chọn thực tế). Đây là tài liệu có giá trị
đào tạo cao nhất mà một tổ chức có thể tự tạo ra, vì nó nói bằng đúng ngôn ngữ, đúng hệ thống và đúng
ràng buộc mà người đọc đang sống trong đó. Cơ chế duy trì nó ở quy mô lớn được nói ở
[`09-to-chuc-va-scaling.md`](/series/engineering-leadership/09-to-chuc-va-scaling/).
