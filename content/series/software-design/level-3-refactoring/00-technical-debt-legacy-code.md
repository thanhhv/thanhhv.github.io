+++
title = "3.0 — Technical Debt & Legacy Code: kinh tế học của việc refactor"
date = "2026-07-17T09:10:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## 1. Problem Statement

Level 1–2 cho bạn *lý thuyết* về thiết kế tốt. Nhưng bạn không làm việc trên trang giấy trắng — bạn làm việc trên một codebase 4 năm tuổi, 200 nghìn dòng, viết bởi 15 người trong đó 10 người đã nghỉ, không có test, và đang chạy production nuôi sống công ty. Câu hỏi thực tế không phải "thiết kế đẹp trông thế nào" mà là:

- Khi nào **đáng** refactor, khi nào nên để yên?
- Refactor thế nào mà **không làm vỡ** thứ đang chạy?
- Thuyết phục manager cấp thời gian refactor bằng lập luận gì?

Level 3 trả lời các câu hỏi đó. Chương này dựng khung kinh tế và khung an toàn; hai chương sau là danh mục **Code Smell** (triệu chứng); hai chương cuối là danh mục **kỹ thuật refactoring** (thuốc).

## 2. Technical Debt — dùng đúng phép ẩn dụ

Ward Cunningham (1992) tạo ra phép ẩn dụ "nợ kỹ thuật" với nghĩa chính xác: **đi nhanh hôm nay bằng giải pháp chưa hoàn thiện = vay một khoản nợ; lãi suất là chi phí phụ trội mỗi lần chạm vào vùng code đó**. Không trả gốc (refactor) thì chỉ è cổ trả lãi mãi mãi.

Điểm hay bị quên: Cunningham nói vay nợ **có thể là quyết định đúng** — startup vay nợ kỹ thuật để ra thị trường trước đối thủ giống doanh nghiệp vay vốn để mở rộng. Vấn đề không phải nợ, mà là **nợ không tự giác**: không biết mình đang vay, không ghi sổ, không có kế hoạch trả.

Phân loại thực dụng (theo hai trục của Martin Fowler):

| | **Cố ý** | **Vô ý** |
|---|---|---|
| **Thận trọng** | "Ship trước Black Friday, tuần sau trả" — nợ chiến lược, có sổ sách | "Giờ mới hiểu ra domain, thiết kế cũ sai" — nợ học phí, không tránh được |
| **Liều lĩnh** | "Không có thời gian cho thiết kế" — nợ xấu | "Design pattern là gì?" — nợ do thiếu kỹ năng |

Hai ô trên là bình thường và lành mạnh. Hai ô dưới mới là thứ giết codebase. Hàm ý quản trị: nợ chiến lược cần **ghi sổ** (TODO có ticket, ADR ghi lại trade-off đã chọn) — nợ có sổ sách trả được, nợ tàng hình thì không.

**Lập luận với manager — dùng ngôn ngữ chi phí, đừng dùng ngôn ngữ thẩm mỹ.** "Code xấu lắm" không thuyết phục được ai. Thứ thuyết phục: *"Tính năng loại này năm ngoái tốn 3 ngày, giờ tốn 2 tuần vì mọi thay đổi phải đi qua module X. Đầu tư 1 sprint tách module X, các tính năng sau trở về 3-4 ngày"* — đo được, kiểm chứng được, có hoàn vốn. Nếu bạn không quy được ra chi phí thay đổi, có thể chính bạn cũng chưa chắc nó đáng refactor.

## 3. Refactoring là gì — định nghĩa chặt

Martin Fowler: **Refactoring là thay đổi cấu trúc bên trong mà KHÔNG thay đổi hành vi quan sát được từ bên ngoài.** Định nghĩa này chặt hơn cách dùng thường ngày rất nhiều, và sự chặt chẽ đó là nguồn sức mạnh:

- "Refactor + tiện tay sửa bug" → **không phải refactoring**. Sửa bug là đổi hành vi.
- "Refactor + thêm tính năng nhỏ" → không phải refactoring.
- "Đập đi viết lại (rewrite)" → càng không phải refactoring — đó là canh bạc khác hẳn về rủi ro.

Kỷ luật **hai chiếc mũ** (Kent Beck): tại mỗi thời điểm bạn đội đúng một mũ — mũ *thêm hành vi* (viết test mới, thêm tính năng) hoặc mũ *refactor* (đổi cấu trúc, test giữ nguyên xanh). Đổi mũ thì commit trước đã. Lợi ích cụ thể: khi test đỏ giữa chừng, bạn biết ngay lỗi thuộc loại nào; khi review, PR "pure refactor, no behavior change" đọc nhanh và merge tự tin hơn hẳn PR trộn lẫn.

Nhịp làm việc chuẩn: **chuỗi bước nhỏ, mỗi bước giữ hệ thống chạy được, commit dày đặc**. Refactor lớn = nhiều refactor bé nối nhau, không phải một cú nhảy. Nếu bạn đang ở trạng thái "code không compile 2 tiếng rồi" — đó không phải refactoring, đó là phẫu thuật không gây mê.

## 4. Legacy Code — làm việc với code không có test

Định nghĩa nổi tiếng của Michael Feathers (*Working Effectively with Legacy Code*): **legacy code là code không có test** — bất kể tuổi đời. Vì không có test, mọi thay đổi đều là đức tin. Từ đó sinh ra nghịch lý trung tâm của legacy:

> Muốn refactor an toàn phải có test. Muốn viết test được phải refactor (code đang dính chùm, không cách ly được). **Gà và trứng.**

Feathers cho lối thoát gồm hai công cụ:

### Công cụ 1 — Characterization Test: chụp X-quang hành vi hiện tại

Với code không hiểu nổi và không có đặc tả, **đừng test theo "điều đúng phải là"** — test theo **"điều nó ĐANG làm"**:

```go
// Hàm legacy 300 dòng tính giá cước, không ai hiểu hết, không dám sửa
func TestLegacyPricing_Characterization(t *testing.T) {
    // Bước 1: quăng input đa dạng vào, GHI LẠI output hiện tại làm chuẩn
    cases := []struct {
        name  string
        input PricingInput
        want  int64 // giá trị này lấy bằng cách CHẠY code cũ, không phải suy luận
    }{
        {"đơn thường HN", PricingInput{Weight: 2, Zone: "HN"}, 35_000},
        {"đơn 0kg", PricingInput{Weight: 0, Zone: "HN"}, 15_000},        // ơ, 0kg vẫn tính 15k?
        {"zone rỗng", PricingInput{Weight: 2, Zone: ""}, 35_000},        // zone rỗng = HN?! ghi lại đã
        /* ... 30-50 case phủ các nhánh ... */
    }
    for _, c := range cases {
        if got := LegacyPrice(c.input); got != c.want {
            t.Errorf("%s: got %d want %d", c.name, got, c.want)
        }
    }
}
```

Những case "ơ kìa" (0kg vẫn tính tiền, zone rỗng ngầm hiểu là HN) chính là giá trị lớn nhất: có thể là bug, có thể là hành vi mà khách hàng nào đó đang **dựa vào**. Characterization test đóng băng chúng lại — refactor xong hành vi y nguyên, còn việc "có nên sửa hành vi đó không" trở thành quyết định nghiệp vụ *riêng biệt*, có sổ sách. Mẹo thực chiến: dùng coverage tool (`go test -cover`) đo xem bộ characterization đã phủ nhánh nào, quăng thêm input tới khi phủ đủ vùng sắp sửa; với output phức tạp, golden file test (`-update` flag ghi lại snapshot) tiết kiệm rất nhiều công.

### Công cụ 2 — Seam: khe hở để chen test vào

Seam = điểm có thể **thay đổi hành vi của code mà không sửa code đó**. Legacy code khó test vì thiếu seam — hàm gọi thẳng DB, thời gian thật, API thật. Nghệ thuật là mở seam bằng thay đổi **nhỏ và an toàn nhất có thể**, chưa cần đẹp:

```go
// Legacy: gọi thẳng — không seam
func SettleDaily() error {
    now := time.Now()                          // thời gian thật — test không điều khiển được
    rows := queryOrdersFromProdDB(now)         // DB thật
    /* 200 dòng logic đối soát */
    sendReportEmail(result)                    // email thật
}
```

```go
// Bước mở seam TỐI THIỂU: Extract & Parameterize — logic thuần tách ra, I/O đẩy ra mép
func SettleDaily() error {                     // vỏ cũ giữ nguyên cho caller
    now := time.Now()
    rows := queryOrdersFromProdDB(now)
    result := settle(rows, now)                // ← hàm MỚI, thuần, TEST ĐƯỢC
    return sendReportEmail(result)
}

func settle(rows []OrderRow, now time.Time) SettleResult {
    /* 200 dòng logic — giờ nằm sau một seam */
}
```

Đây chính là **Functional Core, Imperative Shell** (chương 2.1) áp vào legacy: chưa cần interface, chưa cần DI — chỉ cần *tách phần tính toán ra khỏi phần I/O* là 80% giá trị test đã có. Interface/DIP đến sau, khi cần thay cả tầng I/O.

Vòng lặp chuẩn của Feathers, dùng cho mọi thay đổi trên legacy:

```
1. Xác định điểm cần sửa
2. Tìm điểm đặt test gần nhất
3. Mở seam TỐI THIỂU để test được (đôi khi phải chấp nhận xấu tạm)
4. Viết characterization test — lưới an toàn
5. Giờ mới refactor / thêm tính năng, dưới lưới
```

## 5. Chiến lược theo quy mô — và khi nào KHÔNG refactor

**Ưu tiên theo nhiệt độ**: dữ liệu git không nói dối — `git log --format= --name-only | sort | uniq -c | sort -rg | head -20` cho ngay danh sách file bị sửa nhiều nhất. **Smell × tần suất sửa = độ ưu tiên.** Code xấu không ai chạm 3 năm → để yên tuyệt đối; code tạm ổn nhưng cả team sửa mỗi tuần → mỏ vàng refactor.

**Ba nhịp đầu tư**:

- **Nhịp thở (mỗi PR)**: Boy Scout Rule — sạch hơn một chút mỗi lần đi qua: đổi tên biến, extract một hàm, thêm một test. Miễn phí, không cần xin phép.
- **Nhịp tính năng (chuẩn bị mặt bằng)**: *"make the change easy, then make the easy change"* (Kent Beck) — refactor mở đường TRƯỚC khi thêm tính năng, tính vào chi phí tính năng. Đây là cách trả nợ bền vững nhất vì nó tự động nhắm đúng vùng nóng.
- **Nhịp chiến dịch (có ticket, có thời hạn)**: tách module lớn, thay hạ tầng — cần kỹ thuật riêng như **Strangler Fig**: dựng đường mới cạnh đường cũ, điều hướng dần traffic (feature flag/router), đường cũ teo dần rồi cắt. Không big-bang.

**Khi nào KHÔNG refactor** — trung thực với chính mình:

- Code sắp bị thay thế/khai tử → dọn dẹp là lãng phí thuần túy.
- Không hiểu domain đủ sâu → refactor bây giờ = đóng khung sự hiểu sai vào cấu trúc "đẹp"; đợi hiểu thêm đã (nợ học phí chưa đáo hạn).
- Đang cháy deadline và vùng code không nằm trên đường đi → ghi sổ, đi tiếp.
- Rewrite toàn phần: hầu như luôn là cái bẫy (case study kinh điển: Netscape 5 — đập đi viết lại, 3 năm không ship được gì, mất thị trường). Điều kiện hiếm hoi để rewrite thắng: hệ nhỏ, đặc tả rõ (có characterization test làm đặc tả!), công nghệ cũ thực sự chết. Còn lại: strangler.

## 6. Kết nối với phần còn lại của Level 3

Bốn chương tiếp theo là danh mục tra cứu, tổ chức theo cặp *triệu chứng → thuốc*:

- **3.1 — Smells phình to** (Long Method, Large Class/God Object, Primitive Obsession, Data Clumps): bệnh của *một đơn vị code ôm quá nhiều*.
- **3.2 — Smells coupling** (Duplicate Code, Shotgun Surgery, Feature Envy, Message Chains, Middle Man): bệnh của *tri thức đặt sai chỗ*.
- **3.3 — Kỹ thuật cơ bản** (Extract Method, Move Method, Extract Class, Introduce Parameter Object): thao tác dao kéo hằng ngày.
- **3.4 — Kỹ thuật cấu trúc** (Replace Conditional with Polymorphism, Replace Inheritance with Composition, Replace Loop with Strategy): các phép biến hình đưa code *đến ngưỡng cửa của pattern* — cây cầu sang Level 4.

Nguyên tắc dùng danh mục: smell là **triệu chứng cần chẩn đoán, không phải bản án tự động**. Mỗi smell đều có ngoại lệ hợp lý — chương nào cũng sẽ nói rõ khi nào "smell" thực ra là quyết định đúng.

---

*Tiếp theo: [3.1 — Code Smells: nhóm phình to](/series/software-design/level-3-refactoring/01-smells-phinh-to/)*
