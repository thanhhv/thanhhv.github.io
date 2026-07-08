+++
title = "Chương 1.4 — Separation of Concerns & Composition over Inheritance"
date = "2026-07-07T22:00:00+07:00"
draft = false
tags = ["backend", "golang", "clean-architecture"]
series = ["Clean Architecture với Golang"]
+++

> **Level 1 – Foundation** · Hai nguyên lý cuối của phần nền tảng.

---

## Phần A — Separation of Concerns (SoC)

### 1. Problem Statement

Một hàm handler điển hình trong dự án "chạy được là được":

```go
func CreateOrder(w http.ResponseWriter, r *http.Request) {
	// 1. Parse JSON (concern: giao thức)
	// 2. Validate input (concern: hợp lệ dữ liệu vào)
	// 3. Check quyền user (concern: bảo mật)
	// 4. Tính giá, áp khuyến mãi (concern: NGHIỆP VỤ)
	// 5. Mở transaction, ghi 3 bảng (concern: lưu trữ, nhất quán)
	// 6. Ghi log, đẩy metric (concern: vận hành)
	// 7. Render JSON response (concern: giao thức)
}
```

Bảy mối quan tâm (concern), bảy tốc độ thay đổi khác nhau, bảy loại chuyên môn khác nhau — trộn trong một hàm. Muốn sửa **một** concern phải đọc và hiểu **cả bảy**; muốn test concern 4 phải dựng đủ 1–7. Đây là dạng tổng quát của mọi vấn đề đã nêu ở các chương trước.

Nếu không tách: mỗi thay đổi có bán kính ảnh hưởng bằng cả hàm; kiến thức chuyên môn không tách được theo team (security team phải đọc code khuyến mãi để review một dòng authorization); mọi test đều là integration test.

Cách cũ — comment phân đoạn (`// --- validation ---`) — chỉ tách *thị giác*, không tách *phụ thuộc*: các đoạn vẫn chung biến cục bộ, chung transaction, chung vòng đời.

### 2. Bản chất

SoC (Dijkstra, 1974) là nguyên lý mẹ: **mỗi đơn vị code chỉ nên buộc người đọc suy nghĩ về một khía cạnh của hệ thống**. Coupling/cohesion là thước đo; SRP là SoC áp cho một module; layer là SoC áp cho toàn hệ thống theo trục kỹ thuật; bounded context (DDD) là SoC theo trục nghiệp vụ.

Câu hỏi thiết kế trung tâm không phải "có tách không" mà là **"tách theo trục nào"**. Hai trục chính:

- **Trục kỹ thuật** (giao thức / chính sách / lưu trữ): cho phép chuyên môn hóa và thay công nghệ độc lập → sinh ra *layer*.
- **Trục nghiệp vụ** (order / payment / shipping): cho phép team ownership và thay đổi tính năng độc lập → sinh ra *module/bounded context*.

Hệ thống trưởng thành cần **cả hai**: chia module theo nghiệp vụ trước, trong mỗi module chia layer theo kỹ thuật. Chia một trục mà bỏ trục kia đều tạo ra ma sát (chương 3 sẽ so sánh cụ thể các cách tổ chức package).

### 3. Hình dạng trong Go — cross-cutting concerns bằng middleware/decorator

Các concern "ngang" (log, metric, auth, retry) cắt qua mọi tính năng. Tách chúng bằng decorator để nghiệp vụ không biết đến chúng:

```go
// Concern vận hành tách khỏi nghiệp vụ bằng decorator quanh interface domain
type instrumentedRepo struct {
	next    order.Repository // decorate contract của DOMAIN
	metrics *prometheus.HistogramVec
}

func (r *instrumentedRepo) Save(ctx context.Context, o order.Order) error {
	start := time.Now()
	err := r.next.Save(ctx, o)
	r.metrics.WithLabelValues("save", status(err)).Observe(time.Since(start).Seconds())
	return err
}

// main.go: svc := order.NewService(instrument(postgres.NewOrderRepo(db)), ...)
```

Package `order` không đổi một dòng khi bạn thêm/bỏ metric, retry, cache — tất cả là lớp vỏ compose ở composition root. HTTP middleware (`func(http.Handler) http.Handler`) là cùng một pattern ở tầng delivery.

### 4. Anti-patterns

- **Tách theo trục sai**: chia project thành `controllers/`, `services/`, `models/` toàn cục (chỉ trục kỹ thuật) → một feature trải qua 3 thư mục, mọi team đụng mọi thư mục. Ngược lại chia theo feature nhưng bên trong trộn SQL với business rule → không test được. Cần cả hai trục, đúng thứ tự.
- **Tách quá hạt mịn**: 12 package cho một feature 400 dòng — chi phí điều hướng vượt lợi ích cô lập.
- **Concern "vô chủ"**: transaction, cache, authorization không được xếp tầng rõ ràng nên mỗi nơi tự xử một kiểu (chương 5 và 11 giải quyết cụ thể).

---

## Phần B — Composition over Inheritance

### 1. Problem Statement

Trong các hệ thống OOP cổ điển, tái sử dụng bằng kế thừa tạo ra:

- **Fragile base class**: sửa class cha, vỡ ngầm class con ở nơi khác — coupling mạnh nhất có thể (content coupling xuyên vòng đời).
- **Cây phân loại cứng**: `Animal → Dog → RobotDog?` — thực thể đa chiều không xếp được vào cây một chiều.
- **Kế thừa để lấy code, trả giá bằng contract**: con thừa hưởng cả những method vô nghĩa với nó (vi phạm LSP kinh điển).

### 2. Go đã quyết định thay bạn

Go **không có inheritance** — quyết định thiết kế có chủ đích, không phải thiếu sót. Go cung cấp hai cơ chế compose:

**Embedding (compose struct/behavior):**

```go
// Không phải "BaseRepository" để kế thừa — mà là các mảnh ghép độc lập
type retryPolicy struct{ maxAttempts int }
func (r retryPolicy) do(ctx context.Context, fn func() error) error { ... }

type OrderRepo struct {
	retryPolicy          // embed: có method do() — quan hệ HAS-A, không phải IS-A
	db *sql.DB
}
```

Điểm khác biệt then chốt với inheritance: **không có override ảo**. Nếu `retryPolicy.do` gọi hàm khác, nó gọi phiên bản *của chính nó*, không bao giờ bị struct ngoài "đè" — loại bỏ fragile base class ngay ở mức ngôn ngữ.

**Interface composition (compose contract):**

```go
type Reader interface{ Read(p []byte) (int, error) }
type Writer interface{ Write(p []byte) (int, error) }
type ReadWriter interface { Reader; Writer } // contract lớn = tổng các contract nhỏ
```

### 3. Ý nghĩa kiến trúc

Composition là cơ chế thực thi của mọi nguyên lý đã học: decorator (SoC), adapter (DIP), strategy qua interface (OCP) — tất cả đều là "ghép các mảnh nhỏ qua contract hẹp" thay vì "thừa hưởng khối lớn". Trong Clean Architecture bằng Go, bạn sẽ không bao giờ cần `BaseService`, `BaseController`, `AbstractRepository`. Nếu thấy mình muốn viết chúng — đó là tín hiệu đang bê tư duy Java vào Go; câu trả lời đúng luôn là một mảnh compose được: middleware, decorator, hàm helper, hoặc embedded struct nhỏ.

### 4. Anti-patterns

- **Embedding để giả lập inheritance**: embed struct lớn chỉ để "thừa hưởng" 10 method rồi override vài cái bằng cách định nghĩa lại — người đọc không còn biết method nào chạy. Embed các mảnh *nhỏ, đơn nhiệm*.
- **Embed mutex/kiểu unexported lộ API**: `type Cache struct { sync.Mutex }` làm `cache.Lock()` thành public API. Dùng field thường: `mu sync.Mutex`.
- **God helper được embed khắp nơi**: `type common struct{ log, cfg, db, cache }` embed vào mọi service — biến global đội lốt composition.

---

## Tổng kết Level 1 — Foundation

Năm chương nền tảng quy về một chuỗi nhân quả:

```
Phần mềm phải thay đổi liên tục (business)
  → chi phí thay đổi do coupling & cohesion quyết định (1.1)
  → SOLID: quy tắc hành động để quản trị chúng (1.2)
  → DIP: cơ chế kỹ thuật đảo mũi tên về phía ổn định (1.3)
  → SoC: chọn trục tách; Composition: cách ghép lại (1.4)
```

Bạn đã có đủ nền để trả lời câu hỏi mà Clean Architecture sinh ra để trả lời: *"Sắp xếp các mũi tên phụ thuộc của cả hệ thống như thế nào để business logic sống lâu hơn mọi công nghệ quanh nó?"*

**Tiếp theo:** [Level 2 — Clean Architecture Core](/series/clean-architect/02-clean-architecture-core/01-vi-sao-clean-architecture-ra-doi/)
