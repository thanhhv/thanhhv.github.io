+++
title = "2.0 — SOLID: lịch sử ra đời và vấn đề nó thực sự giải quyết"
date = "2026-07-17T08:00:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## 1. Vấn đề lịch sử — SOLID sinh ra từ nỗi đau nào?

SOLID không rơi từ trên trời xuống. Nó là sản phẩm của một thời điểm lịch sử rất cụ thể — và hiểu bối cảnh đó quyết định việc bạn áp dụng nó đúng hay mù quáng.

**Thập niên 1990, thế giới C++/OOP doanh nghiệp.** Các hệ thống lớn viết bằng C++ thời đó có đặc điểm: biên dịch một lần mất hàng giờ; sửa một header file kéo theo recompile cả dây chuyền phụ thuộc; deploy là chép binary khổng lồ; và cây kế thừa sâu là "best practice" được dạy chính thống. Robert C. Martin (Uncle Bob) quan sát các codebase như vậy và mô tả bốn triệu chứng của "thiết kế mục nát" (design rot):

- **Rigidity (cứng)**: sửa một chỗ buộc phải sửa dây chuyền chỗ khác — thay đổi nhỏ tốn tuần.
- **Fragility (giòn)**: sửa chỗ này, vỡ chỗ kia — ở nơi *tưởng như không liên quan*.
- **Immobility (bất động)**: muốn tái sử dụng một phần nhưng không gỡ ra được vì nó dính chùm với mọi thứ.
- **Viscosity (nhớt)**: làm đúng thiết kế thì khó, hack thì dễ — nên ai cũng hack.

Nhìn bằng ngôn ngữ Level 1: cả bốn đều là triệu chứng của **coupling cao + cohesion thấp + mũi tên phụ thuộc trỏ sai chiều**. Các nguyên tắc mà Martin tổng hợp (từ công trình của nhiều người: Liskov 1987, Meyer 1988, Parnas 1972...) là **các chiến thuật cụ thể chống lại bốn triệu chứng đó**. Michael Feathers sau này xếp lại thứ tự thành từ viết tắt "SOLID" (~2004).

Điểm cần khắc cốt: **SOLID là phương tiện, không phải mục đích.** Mục đích là code chịu được thay đổi. Một codebase "vi phạm" SOLID mà chi phí thay đổi thấp là codebase tốt. Một codebase tuân thủ SOLID hình thức mà mỗi tính năng phải mở 15 file là codebase tồi.

## 2. Bản đồ 5 nguyên tắc — dưới góc nhìn thống nhất

Cả 5 nguyên tắc là 5 câu trả lời cho cùng một câu hỏi: **"Khi yêu cầu thay đổi, thay đổi đó lan đến đâu?"**

| | Tên | Một câu bản chất | Chống lại | Gốc Level 1 |
|---|---|---|---|---|
| **S** | Single Responsibility | Một module chỉ nên có một lý do để thay đổi | Thay đổi A làm vỡ tính năng B trong cùng module | Cohesion |
| **O** | Open/Closed | Thêm hành vi mới bằng cách *thêm code*, không *sửa code cũ* | Mỗi tính năng mới phải mổ lại code đang chạy ổn | Abstraction |
| **L** | Liskov Substitution | Subtype phải thay thế được base type mà không phá kỳ vọng | Polymorphism trở thành bãi mìn `if type == ...` | Polymorphism |
| **I** | Interface Segregation | Không ép client phụ thuộc method nó không dùng | Recompile/redeploy/re-mock dây chuyền vì method không liên quan | Coupling |
| **D** | Dependency Inversion | Phụ thuộc vào abstraction, không vào chi tiết; chi tiết phụ thuộc abstraction | Domain quý giá bị đóng đinh vào hạ tầng thay thế được | Dependency direction |

Và một quan sát ít được nói ra: **5 nguyên tắc không độc lập** — chúng chồng lấn và củng cố nhau. OCP thường được *thi hành* bằng DIP. ISP là SRP áp cho interface. LSP là điều kiện để OCP hoạt động (mở rộng bằng subtype chỉ an toàn khi subtype thay thế được). Học chúng như 5 điều rời rạc là học kiểu từ điển; học chúng như một hệ là hiểu.

## 3. SOLID còn phù hợp với Golang/Node.js hiện đại không?

Câu hỏi xứng đáng được trả lời thẳng, vì bối cảnh 2020s khác 1990s căn bản: ngôn ngữ có structural typing và first-class function; deploy là `git push` chứ không phải ép đĩa; microservices thay đổi đơn vị deployment; và cộng đồng Go vốn dị ứng với "enterprise pattern".

Đánh giá từng cái một cách sòng phẳng:

- **SRP**: còn nguyên giá trị, thậm chí tăng — vì tần suất thay đổi và kích thước team đều tăng. Nhưng đơn vị áp dụng trong Go là *package* nhiều hơn là *struct*.
- **OCP**: giá trị nhất khi bạn viết **thư viện/framework/plugin system** — nơi người khác mở rộng code của bạn mà không sửa được nó. Trong code ứng dụng nội bộ mà bạn sở hữu toàn bộ, "mở file ra sửa + chạy test" nhiều khi rẻ hơn xây điểm mở rộng. Bớt giáo điều hơn thời C++ là đúng.
- **LSP**: Go không có inheritance nên hình thức cổ điển ít gặp — nhưng LSP thực chất nói về **hợp đồng hành vi của interface**, và điều đó còn nguyên: mọi `io.Reader` phải hành xử như tài liệu của `io.Reader` hứa, nếu không cả hệ sinh thái vỡ.
- **ISP**: Go *nội hóa* nguyên tắc này vào văn hóa ngôn ngữ — interface 1-2 method, consumer-side. Trong Go, vi phạm ISP trông lạc lõng ngay; trong TS/NestJS vẫn phải tự kỷ luật.
- **DIP**: còn nguyên và là nền của mọi kiến trúc hiện đại (Clean/Hexagonal). Nhưng *chi phí* áp dụng trong Go thấp hơn Java (implicit interface) nên áp dụng *chọn lọc* dễ hơn — chỉ đảo chiều ở ranh giới hạ tầng, không đảo mọi thứ.

Kết luận trung thực: **tinh thần SOLID trường tồn, hình thức SOLID phải phiên dịch lại theo ngôn ngữ và thời đại.** Các chương sau làm đúng việc phiên dịch đó — mỗi nguyên tắc một chương, đi từ bài toán thật, không tụng kinh.

## 4. Cách đọc 5 chương tiếp theo

Mỗi chương theo đúng template: bài toán thật → code smell → nguyên lý → refactor từng bước → trade-off → production → anti-pattern → khi nào bỏ qua. Đặc biệt, mỗi chương đều có mục **"Khi nguyên tắc này trở thành bệnh"** — vì với SOLID, over-apply phổ biến và tốn kém không kém under-apply.

---

*Tiếp theo: [2.1 — Single Responsibility Principle](/series/software-design/level-2-principles/01-srp/)*
