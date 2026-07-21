+++
title = "5.4 — Tổng kết: toàn bộ tài liệu trên một trang, và con đường tiếp theo"
date = "2026-07-17T13:00:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## 1. Một sợi chỉ xuyên suốt

Ba mươi ba chương quy về **một hàm mục tiêu duy nhất**, phát biểu từ chương 1.1 và chưa từng đổi:

> **Tối thiểu hóa tổng chi phí thay đổi (hiểu + lan truyền + kiểm chứng + phối hợp), chiết khấu theo xác suất thay đổi đó xảy ra — với chi phí hiện tại chấp nhận được.**

Mọi thứ còn lại là công cụ phục vụ hàm đó, xếp thành bốn tầng:

```
Tầng ĐO LƯỜNG   (Level 1): coupling, cohesion, chiều phụ thuộc, invariant, immutability
                → ngôn ngữ để NHÌN THẤY chi phí trong code

Tầng NGUYÊN LÝ  (Level 2): SOLID, DRY, KISS/YAGNI, LoD/TDA
                → các heuristic đã đặt tên để GIẢM chi phí — kèm phanh hãm chống lạm dụng

Tầng THAO TÁC   (Level 3): smell (chẩn đoán) + refactoring (phẫu thuật từng-bước-an-toàn)
                → cách DI CHUYỂN code từ trạng thái đắt về trạng thái rẻ, không ngừng hệ thống

Tầng CẤU TRÚC   (Level 4-5): pattern và kiến trúc
                → các ĐÍCH ĐẾN đã được đặt tên của những cuộc di chuyển lặp lại đủ nhiều
```

Và luận điểm trung tâm, giờ đã được chứng minh bằng ba con đường độc lập: **pattern là kết quả, không phải điểm xuất phát** — chứng minh bằng refactoring (mọi pattern ở Level 4 đều được *dẫn ra* từ code smell có thật), bằng lịch sử (bảng số phận 4.0/4.10: ngôn ngữ tiến hóa thì nghi thức đổi, bài toán ở lại), và bằng production (5.3: các hệ lớn *mọc ra* pattern từ bài toán quy mô của chính họ).

## 2. Bộ câu hỏi mang theo — dùng ở mọi design review

Chưng cất toàn bộ tài liệu thành sáu câu, theo đúng thứ tự nên hỏi:

1. **Cái gì sẽ thay đổi, cái gì không?** — trục thay đổi (1.1) quyết định chỗ nào đáng đầu tư flexibility. Chưa trả lời được → chưa thiết kế được, và mặc định đúng là *đơn giản nhất có thể*.
2. **Thay đổi này lan đến đâu?** — soi coupling (1.3): sửa A có phải mở B? thêm loại mới đụng mấy file? (2.1–2.5 là năm lời giải cho năm kiểu lan).
3. **Ai gác invariant này?** — mọi dữ liệu có luật phải có đúng một người gác: constructor, aggregate root, type system (1.2, 1.5, 5.0). "Mọi người tự giác" không phải câu trả lời.
4. **Test cái này cần dựng những gì?** — chi phí dựng test là *chỉ số đo coupling trung thực nhất* (xuyên suốt 2.3–2.5, 4.11). Test khó viết là code đang tự thú.
5. **Có cách đơn giản hơn không?** — hàm thường? struct thường? switch tại chỗ? copy 10 dòng? (2.6, và mục "khi nào KHÔNG dùng" của *mọi* chương Level 4). Trả lời "không" phải có bằng chứng, không phải khiếu thẩm mỹ.
6. **Mình có bài toán của họ không?** — trước mọi pattern/kiến trúc/công nghệ mượn từ bài blog của công ty lớn (5.3 quy luật 3).

## 3. Lộ trình đọc lại theo vai trò

- **Senior chuẩn bị lên Tech Lead**: đọc lại Level 2 + 4.11–4.13 — vì việc của lead là *đặt ranh giới cho người khác làm việc bên trong*: interface, hợp đồng, chiều phụ thuộc.
- **Người đang ôm legacy**: Level 3 là nhà của bạn — đặc biệt 3.0 (characterization test, seam, strangler) trước khi động dao; bảng 3.4 chỉ đường sang pattern khi cần.
- **Người thiết kế hệ mới**: 5.0–5.2 theo đúng thứ tự bảng quyết định 5.2 mục 3 — bắt đầu từ bậc *thấp nhất* chịu được, không phải bậc ấn tượng nhất.
- **Người phỏng vấn / bị phỏng vấn system design**: khung sáu câu hỏi mục 2 chính là cấu trúc một câu trả lời thiết kế tốt — thay cho việc rải tên pattern.

## 4. Đọc gì tiếp — nguồn gốc của những gì bạn vừa học

Tài liệu này đứng trên vai các tác phẩm sau — đọc bản gốc khi muốn đào sâu:

- *A Philosophy of Software Design* — John Ousterhout: deep module, chi phí nhận thức (nền của 1.2).
- *Refactoring* (bản 2) — Martin Fowler: danh mục đầy đủ mà Level 3 chưng cất.
- *Working Effectively with Legacy Code* — Michael Feathers: seam, characterization test (3.0).
- *Design Patterns* — GoF: đọc *sau* khi có Level 1–3, đọc phần Consequences kỹ nhất (4.0).
- *Patterns of Enterprise Application Architecture* — Fowler: quê gốc của 4.12.
- *Domain-Driven Design* — Eric Evans (và bản dễ vào hơn: *Learning DDD* — Vlad Khononov): toàn bộ 5.0.
- *Designing Data-Intensive Applications* — Martin Kleppmann: nền tảng phân tán cho 4.13/5.2.
- Go riêng: hiệu ứng chuẩn mực từ chính stdlib — đọc source `io`, `net/http`, `database/sql` như đọc sách (5.3A); cộng *100 Go Mistakes* — Teiva Harsanyi.

## 5. Lời kết

Nếu phải nén ba mươi ba chương thành ba câu để dán lên màn hình:

> **Code đơn giản chạy được hôm nay đánh bại kiến trúc đẹp cho tương lai chưa chắc đến.**
> **Nhưng khi tương lai đến thật — smell sẽ báo, refactoring có đường đi từng bước, và pattern đứng chờ sẵn ở cuối đường.**
> **Nghề thiết kế là nghe được tiếng báo đó sớm, và không rút dao trước khi nó vang lên.**

Chúc bạn viết những hệ thống mà người kế nhiệm sẽ thầm cảm ơn.

---

*— Hết bộ tài liệu. Quay lại [Mục lục](/series/software-design/00-muc-luc/).*
