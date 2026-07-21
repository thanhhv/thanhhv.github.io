+++
title = "2.4 — Interface Segregation Principle: interface nhỏ, phía consumer"
date = "2026-07-17T08:40:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## 1. Problem Statement

Câu chuyện gốc của ISP đáng kể lại vì nó rất "đời": Robert Martin tư vấn cho Xerox, hệ thống điều khiển máy in đa năng. Mọi tác vụ (in, scan, staple, fax) đều đi qua một class `Job` khổng lồ. Task chỉ cần in cũng phụ thuộc `Job` — nên mỗi khi ai đó sửa phần staple, **toàn bộ** hệ thống recompile và redeploy (C++ thập niên 90: recompile tính bằng giờ). Thuốc của Martin: tách `Job` thành các interface theo *từng client* — `PrintJob`, `StapleJob` — mỗi client chỉ thấy phần nó dùng.

> **"Clients should not be forced to depend on methods they do not use."**

Ngày nay không ai đau vì recompile C++ nữa — nhưng nỗi đau đổi hình chứ không biến mất: **mock 28 method để test 2 method**, **đọc godoc interface 30 method để tìm 2 method liên quan**, **không thể biết hàm này thực sự đụng gì vào dependency** — đều là "bị ép phụ thuộc thứ không dùng".

## 2. Code Smell

Code thật, gặp ở mọi codebase Go trưởng thành nửa vời:

```go
// ❌ interface "cho đủ" — producer-side, phản chiếu mọi thứ struct làm được
type UserRepository interface {
    Create(ctx context.Context, u *User) error
    GetByID(ctx context.Context, id string) (*User, error)
    GetByEmail(ctx context.Context, email string) (*User, error)
    Update(ctx context.Context, u *User) error
    Delete(ctx context.Context, id string) error
    List(ctx context.Context, f Filter) ([]*User, error)
    Count(ctx context.Context, f Filter) (int, error)
    SetPassword(ctx context.Context, id string, hash []byte) error
    // ... 20 method nữa, mọc dần theo năm tháng
}
```

Hàm gửi email chào mừng chỉ cần **một** method:

```go
func SendWelcomeEmail(ctx context.Context, repo UserRepository, mailer Mailer, userID string) error {
    u, err := repo.GetByID(ctx, userID)
    /* ... */
}
```

Hậu quả cụ thể:

- **Test**: fake `UserRepository` phải có đủ 28 method (dù stub rỗng) — mỗi lần interface thêm method, MỌI fake trong hàng chục file test phải sửa. Đây là Shotgun Surgery do interface béo gây ra.
- **Chữ ký hàm nói dối**: `SendWelcomeEmail` khai nhận "toàn quyền user repository" trong khi chỉ đọc một user. Người review/maintain phải đọc thân hàm mới biết nó *thực sự* đụng gì — và không dám tin nó *không* Delete.
- **Blast radius**: method mới cho tính năng billing thêm vào interface → mọi consumer không liên quan billing đều "thấy" thay đổi, mọi mock vỡ.

## 3. First Principles

ISP là **SRP áp cho interface** (interface béo có nhiều lý do thay đổi) và là dạng đặc biệt của giảm coupling: **diện tích tiếp xúc (surface area) giữa hai module nên nhỏ nhất đủ dùng**. Mỗi method trong interface là một điều khoản ràng buộc ba bên: consumer được gọi, implementation phải cung cấp, maintainer phải giữ ổn định. Interface 28 method = hợp đồng 28 điều khoản mà đa số các bên không cần — thuần chi phí, không lợi ích.

Nhắc lại từ chương 1.5 vì đây là chỗ nó tỏa sáng: **implicit interface của Go làm ISP gần như miễn phí** — consumer tự khai interface nhỏ, `*PostgresUserRepo` tự động khớp, không sửa gì ở producer. Trong Java/C# (nominal), tách interface phải sửa class thêm `implements` → ISP đắt → người ta lười → interface béo tồn tại dai dẳng. Một tính năng ngôn ngữ thay đổi cả kinh tế học của một nguyên tắc thiết kế.

## 4. Refactoring Journey

**Bước 1 — mỗi consumer tự khai đúng nhu cầu:**

```go
// ✅ trong package email — interface 1 method, đặt cạnh người dùng nó
type UserGetter interface {
    GetByID(ctx context.Context, id string) (*User, error)
}

func SendWelcomeEmail(ctx context.Context, users UserGetter, mailer Mailer, userID string) error {
    // chữ ký giờ là TÀI LIỆU CHÍNH XÁC: hàm này chỉ đọc user theo ID, chấm hết
}
```

Fake test giờ là 4 dòng. Interface 28-method dần **không còn ai import** → xóa. Codebase hội tụ về: **concrete struct ở producer, nhiều interface nhỏ rải ở các consumer**. Trông "ngược" với Java nhưng đây chính là hình dạng tự nhiên của Go code tốt.

**Bước 2 — khi nhiều consumer trùng nhu cầu, compose từ interface nguyên tử:**

```go
// Theo mẫu io.Reader/io.Writer/io.ReadWriter của stdlib
type UserReader interface {
    GetByID(ctx context.Context, id string) (*User, error)
    GetByEmail(ctx context.Context, email string) (*User, error)
}
type UserWriter interface {
    Create(ctx context.Context, u *User) error
    Update(ctx context.Context, u *User) error
}
type UserReadWriter interface {
    UserReader
    UserWriter
}
```

**Bước 3 — mở rộng khả năng không phá hợp đồng cũ: optional interface + type assertion.** Kỹ thuật ISP nâng cao mà stdlib dùng khắp nơi:

```go
// http.ResponseWriter chỉ có 3 method. Nhưng flush là khả năng TÙY CHỌN:
if f, ok := w.(http.Flusher); ok {
    f.Flush()   // dùng nếu có, bỏ qua nếu không — hợp đồng lõi không phình
}
```

Đây là cách thêm capability mà không vi phạm ISP lẫn không phá LSP (không ép mọi implementation hỗ trợ). Trade-off của nó: khả năng trở nên "tàng hình" trong chữ ký — và có bẫy thật: middleware bọc `ResponseWriter` bằng struct riêng làm *mất* các optional interface của writer gốc (bug có thật ở nhiều thư viện middleware Go; `http.NewResponseController` sinh ra để vá vấn đề này). Bài học: optional interface mạnh nhưng compose kém — dùng cho capability hiếm, đừng dùng làm cơ chế chính.

**So sánh TypeScript**: cùng đích đến bằng `Pick`/utility types (`Pick<UserRepository, "getById">`) hoặc khai interface nhỏ độc lập — structural typing của TS cho phép "kiểu Go" hoàn toàn; rào cản ở TS là *văn hóa DI framework* (NestJS inject theo class token, kéo về interface/class to producer-side).

## 5. Trade-off

- **Nhiều interface nhỏ = nhiều khai báo rải rác**: `UserGetter` xuất hiện ở 4 package với tên hơi khác nhau (`UserGetter`, `UserFetcher`, `UserByIDLoader`) — người mới hỏi "sao không dùng chung một cái?". Trả lời: trùng lặp *khai báo* này rẻ (không phải trùng lặp logic) và mua được sự độc lập giữa các consumer; nhưng nếu thấy khó chịu, hội tụ về bộ interface chuẩn trong package domain (Bước 2) là thỏa hiệp tốt.
- **Interface nhỏ quá mức cũng có điểm gãy**: hàm cần 5 khả năng nhận 5 tham số interface riêng → chữ ký cồng kềnh; lúc đó gom thành interface tổ hợp hoặc struct tham số hợp lý hơn.
- **Đo lường**: heuristic thô nhưng hữu ích — interface >5 method là đáng nghi vấn trong Go; stdlib hầu như không có interface nào quá 4 (trừ những ca lịch sử đặc biệt).

## 6. Production Examples

- **`io` package**: bộ sưu tập ISP mẫu mực — `Reader`, `Writer`, `Closer`, `Seeker` nguyên tử; `ReadWriteCloser`... là tổ hợp. `os.File` thỏa mãn cả chục interface mà không biết chúng tồn tại; `io.Copy` chỉ đòi `Reader` + `Writer` nên copy được giữa *bất kỳ* nguồn/đích nào — và còn tăng tốc bằng cách sniff optional interface (`io.ReaderFrom`, `io.WriterTo`) đúng kỹ thuật Bước 3.
- **`fs.FS` (Go 1.16)**: hợp đồng lõi đúng MỘT method `Open`; mọi khả năng khác (`ReadDirFS`, `GlobFS`, `SubFS`...) là optional interface + helper function fallback. Một case study thiết kế API bằng ISP đáng đọc nguyên bản.
- **Kubernetes client-go**: `Lister`/`Getter`/`Watcher` tách bạch — controller khai đúng cái nó cần, test không phải fake cả clientset khổng lồ.
- **Đối chứng NestJS/TypeORM**: inject `Repository<User>` full-featured vào mọi service — tiện lúc viết, nhưng test phải mock repository lớn và không ai biết service *thực sự* dùng gì. Các team TS kỷ luật tự khai port interface nhỏ ở domain (kiểu hexagonal) — tức là tự tay làm điều Go khuyến khích sẵn.

## 7. Anti-pattern

- **Header interface**: interface = copy nguyên danh sách method của struct, đặt cạnh struct, một implementation — nghi lễ vô nghĩa phổ biến nhất trong Go code viết bởi người quen Java/C#. Không phục vụ consumer nào cụ thể, chỉ thêm một file phải sửa mỗi khi thêm method.
- **Interface phình dần**: interface 3 method, mỗi quý ai đó "tiện tay" thêm một method → 3 năm sau 15 method. Kỷ luật: method mới phục vụ consumer nào? Nếu chỉ một consumer — khai interface riêng ở chỗ nó.
- **Fake 28 method với `panic("not implemented")`**: triệu chứng đo được của vi phạm ISP — đếm số method panic trong fake của codebase bạn là có ngay chỉ số sức khỏe interface.

## 8. Khi nào KHÔNG cần

Interface nội bộ một package, 4-5 method, hai consumer cùng dùng cả — để nguyên. CRUD repository mà *mọi* consumer thật sự dùng gần hết method — tách là hình thức. ISP mua lợi khi consumer đa dạng và nhu cầu lệch nhau; một consumer duy nhất dùng hết → không có gì để segregate. Và đừng tách interface *đón đầu* — như mọi nguyên tắc trong tài liệu này: chờ bằng chứng (một consumer mới chỉ cần một góc nhỏ) rồi tách, trong Go việc đó tốn năm phút.

---

*Tiếp theo: [2.5 — Dependency Inversion Principle](/series/software-design/level-2-principles/05-dip/)*
