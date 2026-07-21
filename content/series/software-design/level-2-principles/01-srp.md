+++
title = "2.1 — Single Responsibility Principle: trục của sự thay đổi"
date = "2026-07-17T08:10:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## 1. Problem Statement

SRP là nguyên tắc bị hiểu sai nhiều nhất trong SOLID. Cách hiểu sai phổ biến: *"mỗi hàm/class chỉ làm một việc"* → dẫn đến trò chẻ code thành trăm hàm 3 dòng, mỗi lần đọc luồng nghiệp vụ phải nhảy 15 file. Đó **không phải** SRP.

Định nghĩa gốc của Martin:

> **"A module should have one, and only one, reason to change."** — một module chỉ nên có MỘT LÝ DO ĐỂ THAY ĐỔI.

Và bản diễn giải sau này của chính ông, rõ hơn nhiều: *"Gather together the things that change for the same reasons. Separate those things that change for different reasons."* — **lý do thay đổi** gắn với **con người/bộ phận yêu cầu thay đổi** (actor). SRP là nguyên tắc về *tổ chức con người phản chiếu vào code*, không phải về đếm số việc của một hàm.

## 2. Code Smell — một class, ba ông chủ

Bài toán thật: báo cáo lương. V1 rất tự nhiên:

```go
// V1 — ❌ ba "ông chủ" chung một class
type PayrollService struct{ db *sql.DB }

// (1) Cách TÍNH lương — chủ sở hữu: phòng Kế toán/HR policy
func (s *PayrollService) CalculatePay(e Employee) (Pay, error) {
    base := e.BaseSalary
    ot := e.OvertimeHours * e.HourlyRate * 15 / 10   // OT hệ số 1.5
    /* thuế, bảo hiểm... */
    return Pay{Base: base, Overtime: ot /*...*/}, nil
}

// (2) Cách TRÌNH BÀY báo cáo — chủ sở hữu: phòng Vận hành (đọc report)
func (s *PayrollService) FormatReport(pays []Pay) string {
    var b strings.Builder
    b.WriteString("=== BẢNG LƯƠNG ===\n")
    /* căn cột, format tiền ... */
    return b.String()
}

// (3) Cách LƯU trữ — chủ sở hữu: team Platform/DBA
func (s *PayrollService) Save(ctx context.Context, p Pay) error {
    _, err := s.db.ExecContext(ctx, "INSERT INTO payroll ...", /*...*/)
    return err
}
```

Ba tháng sau, ba yêu cầu đến từ ba hướng **độc lập**: kế toán đổi hệ số OT; vận hành muốn xuất Excel thay vì text; platform chuyển sang Postgres schema mới. Ba dev sửa cùng một file:

- **Merge conflict** thành chuyện hằng tuần — chi phí phối hợp (loại 4, chương 1.1).
- Nguy hiểm hơn: dev sửa format **vô tình** đụng hàm tính lương (đổi shared helper, đổi struct chung) → **đổi format làm sai tiền lương** — fragility đúng nghĩa. Người approve thay đổi format không đủ thẩm quyền/kiến thức để nhận ra rủi ro về tiền.
- Test: muốn test tính lương phải dựng `sql.DB`; muốn test format phải mock DB — mọi test đều đắt vì mọi thứ dính nhau.

Đây là smell mà Level 3 gọi tên **Divergent Change**: một module bị sửa vì nhiều lý do không liên quan. (Smell đối ngẫu — *Shotgun Surgery*, một lý do phải sửa nhiều module — là dấu hiệu chẻ QUÁ tay: SRP có hai hướng hỏng.)

## 3. First Principles

Vì sao "lý do thay đổi" chứ không phải "số việc"? Quay về chi phí (1.1): chi phí kiểm chứng và phối hợp bùng lên khi **hai luồng thay đổi độc lập chạm cùng một vùng code** — mỗi thay đổi phải regression-test cả những thứ nó không chạm về logic, và những người không liên quan phải điều phối với nhau. Tách theo lý do thay đổi = mỗi luồng thay đổi có sân riêng → thay đổi song song không giẫm chân.

Hệ quả thực dụng: **muốn áp dụng SRP phải trả lời được "ai sẽ yêu cầu sửa cái này?"** — một câu hỏi về business, không phải về code. Đây là lý do SRP không thể áp máy móc: cùng một đoạn code, ở công ty này là một trách nhiệm, ở công ty khác là ba.

## 4. Refactoring Journey

**Bước 1 — tách phần tính toán thuần ra khỏi I/O.** Ranh giới rẻ nhất, chắc nhất:

```go
// V2 — package payroll: TÍNH lương, thuần túy, không I/O
package payroll

func Calculate(e Employee, policy Policy) (Pay, error) {
    // policy chứa hệ số OT, bậc thuế... — kế toán đổi policy, không đổi code cấu trúc
}
```

Hàm thuần (pure function): input → output, không side effect. Test không cần mock gì — bảng test case là đặc tả nghiệp vụ luôn:

```go
func TestCalculate(t *testing.T) {
    cases := []struct{ name string; emp Employee; want Pay }{
        {"OT hệ số 1.5", Employee{OvertimeHours: 10, HourlyRate: 100_000}, Pay{Overtime: 1_500_000}},
        /* ... mỗi rule nghiệp vụ một dòng ... */
    }
    // ...
}
```

**Bước 2 — tách trình bày và lưu trữ về lãnh địa riêng:**

```go
// V3 — mỗi lý do thay đổi một package
package payrollreport   // vận hành sở hữu — đổi Excel/PDF/CSV không đụng tiền
func RenderText(pays []payroll.Pay) string { /* ... */ }
func RenderXLSX(pays []payroll.Pay) ([]byte, error) { /* ... */ }

package payrollstore    // platform sở hữu — đổi schema không đụng tiền
type Store struct{ db *sql.DB }
func (s *Store) Save(ctx context.Context, p payroll.Pay) error { /* ... */ }
```

```go
// Tầng điều phối — mỏng, gần như không có logic để sai
func RunPayroll(ctx context.Context, emps []Employee, pol payroll.Policy,
    store *payrollstore.Store) (string, error) {
    var pays []payroll.Pay
    for _, e := range emps {
        p, err := payroll.Calculate(e, pol)
        if err != nil { return "", fmt.Errorf("calc %s: %w", e.ID, err) }
        if err := store.Save(ctx, p); err != nil { return "", err }
        pays = append(pays, p)
    }
    return payrollreport.RenderText(pays), nil
}
```

Kiểm chứng lại bằng thước đo: kế toán đổi hệ số → chỉ `payroll` đổi; vận hành đổi format → chỉ `payrollreport`; platform đổi schema → chỉ `payrollstore`. Ba PR song song, không conflict, mỗi PR được đúng người có chuyên môn review. **Đó** là SRP — không phải "hàm 5 dòng".

Kiến trúc thu được có tên: **Functional Core, Imperative Shell** — lõi tính toán thuần (dễ test, chứa nghiệp vụ đắt giá) bọc bởi vỏ I/O mỏng (khó test nhưng gần như không có logic). Đây là dạng SRP mạnh và thực dụng nhất cho backend, và là bước đệm tự nhiên đến Hexagonal Architecture (Level 5).

## 5. Trade-off

- **Chẻ nhỏ có giá**: 3 package thay vì 1 file — người mới phải học bản đồ; luồng chạy trải trên nhiều nơi. Với tool nội bộ một người bảo trì, V1 có thể là quyết định đúng *cho đến khi* các luồng thay đổi thực sự phân hóa.
- **Đừng tách theo phỏng đoán**: SRP dựa trên *lý do thay đổi thực tế*. Chưa biết ai sẽ đòi sửa gì → giữ đơn giản, để vết nứt tự lộ ra (2 lần sửa vì 2 lý do khác nhau = đủ bằng chứng để tách). Tách sai chỗ còn tệ hơn không tách — bạn ăn chi phí gián tiếp mà không được lợi ích cách ly, và thường sinh Shotgun Surgery.
- **Granularity theo tầng**: SRP áp ở mức package/module trước, struct sau, hàm cuối. Go nghiêng về "package có một chủ đề rõ" hơn là "class nhỏ li ti" kiểu Java.

## 6. Production Examples

- **Go stdlib**: `net/http` (giao thức HTTP) tách khỏi `encoding/json` (mã hóa) tách khỏi `database/sql` (truy cập DB) — mỗi package một lý do thay đổi, phiên bản hóa và phát triển độc lập hàng chục năm.
- **kube-scheduler vs kubelet (Kubernetes)**: quyết định "pod chạy ở node nào" (scheduling policy — thay đổi nhiều, nhiều thuật toán) tách hẳn khỏi "chạy pod thế nào" (container runtime). Hai nhóm maintainer, hai nhịp release — SRP ở mức process.
- **Node.js/Express điển hình**: route handler 200 dòng chứa validate + business + SQL + format response là vi phạm SRP phổ biến nhất hành tinh. Các codebase NestJS tách controller (HTTP) / service (nghiệp vụ) / repository (dữ liệu) chính là SRP được framework hóa — hoạt động tốt *khi ba tầng đó thực sự đổi vì lý do khác nhau*, và thành nghi lễ rỗng khi service chỉ forward call xuống repository.

## 7. Anti-pattern — khi SRP trở thành bệnh

- **Nanoservices trong code**: `OrderValidator`, `OrderValidatorHelper`, `OrderValidationRuleProvider`... mỗi class 10 dòng — "một lý do thay đổi" bị hiểu thành "một dòng logic". Luồng nghiệp vụ vỡ vụn, chi phí hiểu tăng vọt. Cohesion (1.3) là lực đối trọng: **những thứ đổi cùng nhau phải Ở cùng nhau.**
- **Anemic Domain Model**: tách "data" (struct trần) khỏi "behavior" (service làm mọi thứ) nhân danh SRP — thực chất phá cohesion giữa dữ liệu và hành vi của chính nó, mất luôn encapsulation (1.2). Tính lương *thuộc về* khái niệm lương.
- **Utility class hút việc**: `PayrollUtils` chứa mọi thứ "tiện" — coincidental cohesion đội lốt tách bạch.

## 8. Khi nào KHÔNG cần

Script, cron job nhỏ, tool một người dùng; MVP chưa rõ luồng thay đổi; module 100 dòng ổn định hai năm không ai sửa. SRP mua "thay đổi song song rẻ" — khi không có thay đổi song song, đừng trả tiền cho nó. Quy tắc thô: **để thứ gì đó bị sửa vì hai lý do khác nhau HAI lần trước khi tách** — bằng chứng thật hơn phỏng đoán.

---

*Tiếp theo: [2.2 — Open/Closed Principle](/series/software-design/level-2-principles/02-ocp/)*
