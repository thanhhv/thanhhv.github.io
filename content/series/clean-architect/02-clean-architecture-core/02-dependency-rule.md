+++
title = "Chương 2.2 — Dependency Rule: Luật duy nhất"
date = "2026-07-08T00:00:00+07:00"
draft = false
tags = ["backend", "golang", "clean-architecture"]
series = ["Clean Architecture với Golang"]
+++

> **Level 2 – Engineering** · Nếu chỉ được nhớ một điều từ toàn bộ tài liệu, hãy nhớ chương này.

---

## 1. Phát biểu

> **Source code dependencies must point only inward, toward higher-level policies.**
> Phụ thuộc mã nguồn chỉ được trỏ vào trong, về phía chính sách cấp cao hơn.

"Biết" ở đây nghĩa là bất kỳ dạng nào sau: câu lệnh `import`; dùng kiểu dữ liệu của vòng ngoài trong chữ ký hàm/field; tham chiếu tên hàm, hằng số, biến; **và cả những dạng không có import**: hiểu ngầm về format JSON của API, về schema bảng, về tên topic Kafka, về mã lỗi HTTP. Dependency Rule cấm tất cả — vòng trong phải có thể compile, test, và *đọc hiểu* mà không cần vòng ngoài tồn tại.

## 2. Vì sao chỉ cần một luật này là đủ

Mọi lợi ích của Clean Architecture đều suy ra được từ luật này, như định lý suy từ tiên đề:

- **Testability**: domain không biết hạ tầng → thay hạ tầng bằng fake trong RAM → unit test thuần. (suy ra trực tiếp)
- **Độc lập framework**: framework ở vòng ngoài → domain không import nó → upgrade/thay framework không chạm domain.
- **Nhiều delivery**: HTTP, gRPC, CLI đều ở ngoài trỏ vào → dùng chung use case.
- **Khoanh vùng thay đổi**: thay đổi chỉ lan theo chiều mũi tên phụ thuộc; mũi tên chụm vào trong → thay đổi vòng ngoài không lan đi đâu, thay đổi vòng trong lan ra có kiểm soát (compiler chỉ đích danh nơi phải sửa).

Và ngược lại: vi phạm luật ở **một** điểm là đủ phá chuỗi lợi ích. Một import `net/http` trong domain nghĩa là toàn bộ domain giờ dính vòng đời của HTTP layer.

## 3. Cách hoạt động: vượt ranh giới hai chiều

### Chiều vào (ngoài → trong): gọi thẳng

Handler gọi use case, dùng kiểu của use case — hợp lệ, vì mũi tên trỏ vào:

```go
// httpapi (vòng 3) import order (vòng 2): ĐÚNG chiều
func (h *Handler) Create(w http.ResponseWriter, r *http.Request) {
	var req createOrderRequest // DTO của tầng HTTP
	json.NewDecoder(r.Body).Decode(&req)

	o, err := h.svc.PlaceOrder(r.Context(), req.CustomerID, toDomainItems(req.Items))
	...
}
```

### Chiều ra (trong → ngoài): qua interface do vòng trong sở hữu

Use case cần lưu DB — control flow chạy ra ngoài, nhưng source dependency vẫn trỏ vào trong nhờ DIP (chương 1.3). Đây là điểm ảo diệu duy nhất, và nó chỉ là cơ chế đã học:

```
   control flow:      usecase ──────────────▶ postgres.Repo.Save()
   source dep:        usecase ◀────────────── postgres (import order để implement order.Repository)
```

### Dữ liệu vượt ranh giới

Quy tắc: **dữ liệu vượt ranh giới phải ở dạng thuận tiện cho vòng trong.** Use case nhận/trả kiểu domain hoặc struct input/output đơn giản do chính nó định nghĩa — không bao giờ nhận `*gin.Context`, `*sql.Rows`, protobuf message. Việc dịch (mapping) là trách nhiệm của vòng ngoài — đó chính là nghĩa của từ "adapter".

## 4. Cưỡng chế bằng công cụ — kiến trúc phải được compiler/CI bảo vệ

Kiến trúc chỉ tồn tại trong code khi có cơ chế chặn vi phạm tự động. Ba tầng phòng thủ trong Go:

**Tầng 1 — `internal/`:** package trong `internal/` chỉ import được bởi cây thư mục cha. Dùng để chặn module khác xâm nhập chi tiết của module này.

**Tầng 2 — linter theo luật import:**

```yaml
# .golangci.yml
linters-settings:
  depguard:
    rules:
      domain:
        files: ["**/internal/*/domain/**", "**/internal/*/usecase/**"]
        deny:
          - pkg: "database/sql"
          - pkg: "net/http"
          - pkg: "github.com/gin-gonic"
          - pkg: "gorm.io"
          - pkg: "github.com/segmentio/kafka-go"
            # domain/usecase không được biết bất kỳ hạ tầng nào
```

**Tầng 3 — test kiến trúc:** viết test duyệt `go list -deps` và fail nếu package domain kéo theo dependency cấm:

```go
func TestDomainPurity(t *testing.T) {
	out, err := exec.Command("go", "list", "-deps", "./internal/order/domain/...").Output()
	if err != nil { t.Fatal(err) }
	for _, banned := range []string{"database/sql", "net/http", "gorm.io"} {
		if strings.Contains(string(out), banned) {
			t.Errorf("domain package phu thuoc %s — vi pham Dependency Rule", banned)
		}
	}
}
```

## 5. Các vi phạm kinh điển và cách nhận diện

| Vi phạm | Dấu hiệu | Vì sao nguy hiểm |
|---|---|---|
| Domain import ORM | `gorm:"column:..."` tag trong entity | Schema DB định hình domain; đổi ORM = đổi domain |
| Use case nhận `*gin.Context` | chữ ký use case có kiểu framework | Use case chết dính vào Gin; gRPC/CLI phải giả lập Context |
| Domain trả lỗi HTTP | `errors.New("404 not found")` trong domain | Domain biết giao thức; thêm gRPC là dịch ngược lỗi |
| Interface ở package hạ tầng | business import `postgres` để lấy interface | Inversion giả (chương 1.3, mục 3.3) |
| Entity serialize thẳng ra API | `json:"..."` tag trên entity dùng làm response | Đổi tên field nội bộ = breaking change API công khai; format API đóng băng domain |
| "Đi tắt" qua DB | hai module đọc chung bảng | Coupling vô hình với compiler (chương 1.1, 8.3) |

Về JSON tag trên entity: đây là vi phạm *mức nhẹ* và nhiều team chấp nhận có chủ đích ở API nội bộ để đỡ mapping. Điều quan trọng là **quyết định có ý thức**: API công khai/versioned → bắt buộc DTO riêng; API nội bộ ít rủi ro → có thể dùng chung và ghi rõ trong ADR (Architecture Decision Record).

## 6. Trade-off của việc tuân thủ nghiêm

- **Mapping tax**: mỗi ranh giới nghiêm cần code dịch DTO ↔ domain. Với entity 30 field, đó là boilerplate thật. Giải pháp: chỉ đặt ranh giới nghiêm ở nơi format ngoài *thực sự* biến động độc lập với domain.
- **Sự cám dỗ của ranh giới mềm**: cho phép "tạm import cái này đã" là bắt đầu của mục nát — vi phạm đầu tiên hợp thức hóa vi phạm thứ hai. Kinh nghiệm: giữ luật *tuyệt đối* cho domain/usecase, thực dụng ở adapter.
- **Không phải mọi mũi tên đều đáng đảo**: giữa các package hạ tầng với nhau (postgres ↔ migration tool) không cần Dependency Rule — luật chỉ áp trên trục trong–ngoài.

## Tóm tắt

- Một luật duy nhất: phụ thuộc trỏ vào trong; "phụ thuộc" bao gồm cả kiến thức ngầm, không chỉ import.
- Chiều ra vượt bằng interface do vòng trong sở hữu; dữ liệu vượt ranh giới ở dạng của vòng trong.
- Luật phải được cưỡng chế bằng `internal/` + linter + test kiến trúc, không bằng lời hứa.

**Chương tiếp theo:** [Bốn vòng — Entities, Use Cases, Interface Adapters, Frameworks & Drivers trong Go](/series/clean-architect/02-clean-architecture-core/03-cac-layer/)
