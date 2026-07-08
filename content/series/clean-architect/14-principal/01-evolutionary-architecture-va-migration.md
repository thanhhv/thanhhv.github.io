+++
title = "Chương 14 — Level 4 Principal: Evolutionary Architecture, Refactor Legacy, Tổ chức & Hiệu năng"
date = "2026-07-08T15:00:00+07:00"
draft = false
tags = ["backend", "golang", "clean-architecture"]
series = ["Clean Architecture với Golang"]
+++

> **Level 4 – Principal** · Ở cấp này, kiến trúc không còn là bài toán code — nó là bài toán *thời gian* (hệ thống tiến hóa), *con người* (team, ownership) và *kinh tế* (chi phí, rủi ro, thứ tự đầu tư).

---

## 1. Evolutionary Architecture — thiết kế cho sự thay đổi của chính kiến trúc

Sai lầm cấp principal phổ biến nhất: thiết kế "kiến trúc đích" 5 năm rồi ép hệ thống về nó. Thực tế: nghiệp vụ, tổ chức và công nghệ đều thay đổi nhanh hơn kế hoạch 5 năm. Kiến trúc tiến hóa (Ford/Parsons/Kua) đảo cách đặt vấn đề: **thay vì đoán đích đúng, tối ưu chi phí đổi hướng.**

Hai công cụ vận hành được ngay:

**a) Fitness functions — biến nguyên tắc thành test chạy trong CI.** Mọi quy tắc kiến trúc trong tài liệu này đều đã có dạng thực thi: depguard chặn import sai hướng (chương 2.2); test domain purity; contract test giữa module; ngưỡng độ trễ p99 trong load test; giới hạn kích thước module (`gocloc` per package). Nguyên tắc: **quy tắc nào không có fitness function sẽ mục nát trong ~2 quý** — review của con người không thắng được deadline.

**b) ADR (Architecture Decision Records)** — mỗi quyết định một file ngắn: bối cảnh, lựa chọn đã cân nhắc, quyết định, hệ quả chấp nhận. Giá trị thật nằm ở 2 năm sau: người mới hiểu *vì sao* thay vì phán "kiến trúc này ngu" rồi lặp lại sai lầm cũ; và khi bối cảnh đổi, ADR cho biết quyết định nào hết hạn. Các "vết nứt chấp nhận có chủ đích" của chương 12.1 chính là ADR dạng mầm.

Chỉ dấu đo sức khỏe tiến hóa của codebase: (1) thời gian từ ý tưởng feature → production (lead time) có tăng theo tuổi codebase không; (2) tỷ lệ PR chạm ≥ 3 module; (3) số vi phạm fitness function bị suppress. Ba số này xấu đi cùng lúc = kiến trúc đang thua cuộc đua với entropy.

## 2. Large-scale Refactoring: đưa Legacy về Clean Architecture

Bối cảnh điển hình: monolith Go 300k dòng, handler 500 dòng gọi thẳng GORM, global `db`, test coverage 8%, business vẫn cần feature mỗi tuần. **Nguyên tắc số một: không big-bang.** Rewrite toàn phần thất bại vì hai lý do cấu trúc: hệ cũ chứa nghiệp vụ không văn bản hóa (chỉ tồn tại trong code + bug đã thành đặc tả), và trong 18 tháng viết lại, hệ cũ vẫn phải tiến hóa — bạn đuổi theo mục tiêu di động.

Chiến lược đã kiểm chứng — **tạo vùng sạch, mở rộng dần**:

```
Bước 1. Đóng băng hướng xấu đi (tuần đầu tiên):
   - Thêm depguard: CODE MỚI không được import gorm/http trong package nghiệp vụ mới
   - Mọi feature mới viết theo kiến trúc mới trong internal/<feature>/
   → Nợ ngừng tăng trước khi bắt đầu trả.

Bước 2. Characterization tests cho vùng sắp sửa:
   - Chưa refactor gì — viết test ghi lại HÀNH VI HIỆN TẠI (kể cả hành vi kỳ quặc)
     qua API/HTTP mức cao (golden tests, snapshot response).
   → Lưới an toàn khi chưa hiểu hết logic.

Bước 3. Bóc từng nghiệp vụ một, theo giá trị × rủi ro:
   a. Chọn module đau nhất-đổi-nhiều-nhất (định luật: 20% code nhận 80% thay đổi)
   b. Trích business rule từ handler/model callback vào package domain mới (pure)
   c. Handler cũ trở thành adapter gọi use case mới — GIỮ NGUYÊN hành vi
   d. Characterization tests xanh → thay dần bằng unit tests thật → xóa golden
   e. Lặp lại với module kế tiếp

Bước 4. Bóp chết đường cũ:
   - Route mới + redirect, metric đếm lượt gọi đường cũ, xóa khi về 0.
```

Kỹ thuật "Seam" (Michael Feathers) trong Go — tìm điểm chèn ranh giới rẻ nhất: hàm global `CalcDiscount(o *gorm.Order)` → bước 1 tách phần thuần (`calcDiscount(total int64, tier string) int64` — test được ngay), bước 2 caller cũ thành wrapper. Mỗi seam là một PR nhỏ, ship liên tục, không nhánh refactor sống 3 tháng.

Kinh tế học của việc này — cách trình bày với lãnh đạo: không xin "6 tháng dừng feature để refactor" (sẽ bị từ chối, và đáng bị từ chối). Thay vào đó: **thuế 15–20% mỗi sprint gắn vào feature** — refactor đúng vùng đang sửa ("boy scout at scale"). Lộ trình đo được bằng lead time của module đã sạch so với module chưa sạch — dùng chính số đó xin ngân sách tiếp.

## 3. Organization Boundary & Team Ownership — Conway là trọng lực

Định luật Conway: kiến trúc hệ thống sao chép cấu trúc giao tiếp của tổ chức — **dù bạn muốn hay không**. Hệ quả thao tác được (Inverse Conway Maneuver): muốn kiến trúc mục tiêu, tổ chức team theo nó *trước*; ngược lại, vẽ ranh giới module cắt ngang một team đang ngồi chung là vẽ ranh giới sẽ mục nát.

Quy tắc thực hành:

- **Ranh giới module = ranh giới ownership**: mỗi module trong modular monolith có đúng một team chủ (CODEOWNERS enforce review). Module 2 chủ = module không chủ.
- **Contract giữa module = contract giữa team**: đổi `payment.Facade` cần thương lượng như đổi API công khai — vì nó *là* API công khai, về mặt xã hội.
- **Kích thước bounded context ≤ kích thước nhận thức của một team** (mô hình Team Topologies: cognitive load là ràng buộc cứng). Context to hơn một team hiểu nổi sẽ tự phân mảnh vô tổ chức.
- **Platform team** sở hữu `platform/` (config, observability, tooling) như một sản phẩm nội bộ có "khách hàng" — không phải bãi rác chung.
- Số đo cảnh báo tổ chức-kiến trúc lệch nhau: tỷ lệ PR cần review chéo team; thời gian chờ review chéo; module có > 3 team cùng commit trong quý.

## 4. Performance Impact — trả lời bằng số

Câu chống lại Clean Architecture thường gặp: "nhiều tầng thế này chậm". Phân rã chi phí:

| Nguồn chi phí | Độ lớn thực tế | Đánh giá |
|---|---|---|
| Gọi qua interface (dynamic dispatch, mất inline) | ~1–5 ns/call | Nhiễu nền. Một query DB = 1–10 ms = 10⁶ lần lớn hơn |
| Copy struct DTO ↔ domain tại ranh giới | ~50–500 ns tùy kích thước | Đáng kể **chỉ khi** trong vòng lặp nóng hàng triệu items |
| Allocation thêm (entity + DTO thay vì 1 struct) | áp lực GC nhẹ | Đo bằng pprof trước khi kết luận |
| Kiến trúc khuyến khích N+1 (load aggregate trong loop) | **ms → giây** | ĐÂY mới là hiểm họa thật |

Kết luận trung thực: chi phí trực tiếp của các tầng là **không đáng kể với I/O-bound service** (99% backend). Chi phí thật nằm ở *pattern sử dụng sai*: query qua aggregate cho danh sách (giải bằng CQRS — chương 10), N+1 do repository-per-entity, mapping trong hot loop. Kỷ luật đúng: (a) benchmark hot path bằng `testing.B` + pprof, không tranh luận chay; (b) khi một hot path thật sự cần, **hạ tầng số nghi thức cục bộ cho path đó** (SQL thẳng, struct dùng chung, mất interface) và ghi ADR — ngoại lệ có hồ sơ, không phải tiền lệ tự do.

```go
// Benchmark ranh giới — trước khi tin ai đó nói "interface chậm lắm"
func BenchmarkFeeDirect(b *testing.B)    { for range b.N { _ = Fee(o) } }
func BenchmarkFeeViaPort(b *testing.B)   { var p Pricer = svc; for range b.N { _, _ = p.Quote(o) } }
// Kết quả điển hình: chênh vài ns — rồi nhìn sang query 3ms bên cạnh mà cười.
```

## 5. Migration Strategy — khung quyết định tổng hợp

Khi đứng trước "hệ thống hiện tại không ổn", tuần tự câu hỏi của một principal:

1. **Đau ở đâu, đo bằng gì?** (lead time? incident? onboarding? scale?) — không có số thì chưa được kê đơn.
2. **Vấn đề là code, dữ liệu, hay tổ chức?** — 3 bệnh, 3 thuốc: refactor (mục 2), tách ownership dữ liệu (chương 12.2), reorg trước re-architect (mục 3).
3. **Đường đi tăng dần tồn tại không?** — hầu như luôn có (strangler, seam, module-by-module). Đề xuất big-bang phải chứng minh *vì sao* đường tăng dần không khả thi, không phải ngược lại.
4. **Mỗi bước có điểm dừng an toàn không?** — kế hoạch tốt là kế hoạch có thể *dừng giữa chừng* mà hệ thống vẫn tốt hơn lúc bắt đầu (giai đoạn 2 e-commerce là điểm dừng hợp lệ vô thời hạn).
5. **Ai trả tiền và thấy gì sau mỗi quý?** — mỗi quý phải ship một lợi ích nhìn thấy được (lead time module X giảm, incident loại Y về 0), nếu không dự án sẽ bị cắt giữa chừng — và đáng bị cắt.

## 6. Lời kết của toàn tài liệu

Clean Architecture không phải đích đến, càng không phải tôn giáo. Nó là một bộ quy tắc đặt ranh giới đã kiểm chứng, đứng trên bốn chân lý cũ hơn nó nhiều: ranh giới theo trục thay đổi, phụ thuộc về phía ổn định, chính sách tách khỏi cơ chế, và cưỡng chế bằng máy chứ không bằng lời hứa. Người kỹ sư trưởng thành dùng nó như dùng mọi công cụ: đúng liều, đúng chỗ, đo được, và sẵn sàng bỏ nghi thức khi bối cảnh không cần — vì mục tiêu cuối cùng chưa bao giờ là kiến trúc đẹp, mà là **phần mềm giữ được khả năng thay đổi với chi phí dự đoán được, trong suốt vòng đời mà doanh nghiệp cần nó.**
