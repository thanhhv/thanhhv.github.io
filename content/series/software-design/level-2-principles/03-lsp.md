+++
title = "2.3 — Liskov Substitution Principle: hợp đồng hành vi"
date = "2026-07-17T08:30:00+07:00"
draft = false
tags = ["backend", "software-design", "design-patterns"]
series = ["Software Design & Design Patterns"]
+++

## 1. Problem Statement

LSP nổi tiếng qua ví dụ Hình vuông/Hình chữ nhật, nhưng ví dụ đó khiến nhiều người nghĩ nó là chuyện hàn lâm về kế thừa. Thực tế, LSP là nguyên tắc **thực chiến nhất** với dev Go/TS hiện đại — vì nó trả lời câu hỏi: *"khi nào một implementation của interface là ĐÚNG?"*

Phát biểu của Barbara Liskov (1987, diễn giải): **nếu S là subtype của T, thì mọi chỗ dùng T phải hoạt động đúng khi thay bằng S — mà không cần biết đó là S.**

Chìa khóa nằm ở vế sau: *"mà không cần biết"*. Polymorphism chỉ có giá trị khi caller **tin** được mọi implementation hành xử theo cùng hợp đồng. Mất niềm tin đó, caller bắt đầu viết `if _, ok := x.(*S3Storage); ok { ... }` — và toàn bộ lợi ích của abstraction sụp đổ: bạn có chi phí của interface *cộng* chi phí của switch-theo-loại.

## 2. Code Smell — vi phạm LSP trong Go trông như thế nào

Go không có inheritance, nhưng vi phạm LSP đầy rẫy — dưới dạng **implementation phản bội hợp đồng ngầm của interface**. Bài toán thật: hệ thống lưu file, có local disk và S3.

```go
type FileStore interface {
    Save(ctx context.Context, path string, data []byte) error
    Load(ctx context.Context, path string) ([]byte, error)
    Delete(ctx context.Context, path string) error
}
```

```go
// Implementation 1 — LocalStore: Delete file không tồn tại → error
func (s *LocalStore) Delete(ctx context.Context, path string) error {
    return os.Remove(s.root + path)   // file không có → *PathError
}

// Implementation 2 — S3Store: Delete object không tồn tại → S3 trả 204, KHÔNG error
func (s *S3Store) Delete(ctx context.Context, path string) error {
    _, err := s.client.DeleteObject(ctx, /*...*/)
    return err                         // không tồn tại → nil!
}
```

Caller viết code dựa trên hành vi của LocalStore (môi trường dev):

```go
// ❌ logic nghiệp vụ dựa vào "Delete file thiếu sẽ báo lỗi"
if err := store.Delete(ctx, path); err != nil {
    metrics.Inc("delete_missing_file")   // dev: chạy đúng. prod (S3): không bao giờ tăng
}
```

Test pass ở local, hành vi *lệch* ở production — loại bug khó nhất: không crash, chỉ *sai lặng lẽ*. Các biến thể cùng họ, gặp hằng ngày:

- Method này trả `nil` slice, method kia trả slice rỗng, caller check `== nil`.
- Implementation A thread-safe, B thì không — swap vào là data race.
- A trả error khi not-found, B trả `(nil, nil)`.
- Fake trong test hành xử khác thật (fake cho phép Save trùng key, thật thì không) → test xanh, prod đỏ. **Fake cũng là một subtype — LSP áp dụng cho cả test double.**
- Mock "chồng chất kỳ vọng" đến mức chỉ mô phỏng đúng một kịch bản — thay đổi caller một chút là mock sai kiểu khác.

## 3. First Principles — hợp đồng gồm những gì

Chữ ký hàm (types) chỉ là phần **cú pháp** của hợp đồng — compiler check được. LSP nói về phần **ngữ nghĩa** — compiler KHÔNG check được:

1. **Precondition** — subtype không được *đòi hỏi nhiều hơn* (base nhận mọi path, subtype panic với path có dấu `/` → caller vỡ).
2. **Postcondition** — subtype không được *hứa ít hơn* (base bảo đảm "sau Save thì Load thấy ngay", subtype eventual-consistent → caller vỡ).
3. **Invariant & hiệu ứng phụ** — error semantics, nil vs empty, thread-safety, idempotency, blocking hay không.

Quy tắc nhớ nhanh: **subtype được phép rộng lượng hơn với input và nghiêm ngặt hơn với output — không bao giờ ngược lại.**

Trong Go, hợp đồng ngữ nghĩa sống ở **doc comment của interface**. Đọc `io.Reader` sẽ thấy nó dày đặc điều khoản: *"Read may return n < len(p)"*, *"may return either err == EOF or err == nil"* ở cuối stream, *"Callers should always process the n > 0 bytes returned before considering the error err"*... Mỗi câu là một điều khoản mà **mọi** implementation phải giữ và **mọi** caller được phép dựa vào. Viết interface mà không viết hợp đồng = mời mỗi implementation tự phát minh ngữ nghĩa riêng.

## 4. Refactoring Journey

**Bước 1 — viết hợp đồng thành lời:**

```go
// FileStore lưu trữ blob theo path.
//
// Hợp đồng cho MỌI implementation:
//   - Save ghi đè nếu path đã tồn tại.
//   - Load trả ErrNotFound nếu path không tồn tại (không bao giờ (nil, nil)).
//   - Delete là idempotent: xóa path không tồn tại trả nil.
//   - An toàn khi gọi đồng thời từ nhiều goroutine.
var ErrNotFound = errors.New("filestore: not found")

type FileStore interface { /* ... như cũ ... */ }
```

Lưu ý hai quyết định: chuẩn hóa error bằng **sentinel error** để caller check `errors.Is(err, ErrNotFound)` — không caller nào phải biết `*PathError` hay S3 error code (đó là chi tiết rò rỉ); và **chọn** Delete idempotent (theo S3) rồi buộc LocalStore tuân theo — chọn chuẩn nào ít quan trọng bằng việc *có* chuẩn.

**Bước 2 — ép mọi implementation về hợp đồng:**

```go
func (s *LocalStore) Delete(ctx context.Context, path string) error {
    err := os.Remove(filepath.Join(s.root, path))
    if errors.Is(err, fs.ErrNotExist) {
        return nil                    // tuân thủ hợp đồng idempotent
    }
    return err
}

func (s *LocalStore) Load(ctx context.Context, path string) ([]byte, error) {
    b, err := os.ReadFile(filepath.Join(s.root, path))
    if errors.Is(err, fs.ErrNotExist) {
        return nil, ErrNotFound       // dịch error hạ tầng → error hợp đồng
    }
    return b, err
}
```

**Bước 3 — hợp đồng phải được TEST: contract test chạy chung cho mọi implementation.** Đây là kỹ thuật ăn tiền nhất chương này:

```go
// filestore_contract_test.go — một bộ test, N implementation
func RunContractTests(t *testing.T, newStore func(t *testing.T) FileStore) {
    t.Run("load missing returns ErrNotFound", func(t *testing.T) {
        s := newStore(t)
        _, err := s.Load(context.Background(), "nope")
        if !errors.Is(err, ErrNotFound) {
            t.Fatalf("want ErrNotFound, got %v", err)
        }
    })
    t.Run("delete is idempotent", func(t *testing.T) {
        s := newStore(t)
        if err := s.Delete(context.Background(), "nope"); err != nil {
            t.Fatalf("delete missing must be nil, got %v", err)
        }
    })
    t.Run("save then load roundtrip", func(t *testing.T) { /* ... */ })
}

func TestLocalStore(t *testing.T) { RunContractTests(t, newTempLocalStore) }
func TestS3Store(t *testing.T)    { RunContractTests(t, newS3TestStore) }   // testcontainers/minio
func TestFakeStore(t *testing.T)  { RunContractTests(t, newFakeStore) }     // fake CŨNG phải qua!
```

Dòng cuối là điểm sâu nhất: **fake dùng trong unit test phải pass cùng contract test với implementation thật** — đó là cách duy nhất bảo đảm "test xanh với fake" suy ra được điều gì đó về production. Chuỗi niềm tin: caller tin interface → interface được định nghĩa bằng contract test → mọi implementation (kể cả fake) pass contract test.

## 5. Trade-off

- **Hợp đồng chặt vs quyền tự do của implementation**: quy định "Delete idempotent" giúp caller đơn giản nhưng bắt mọi backend tương lai phải mô phỏng được điều đó (một backend không hỗ trợ sẽ phải thêm một round-trip check — chậm hơn). Hợp đồng càng hứa nhiều, caller càng sướng, implementation càng khổ — cân theo tỉ lệ caller/implementation.
- **Hợp đồng lỏng vs gánh nặng caller**: `io.Reader` hứa rất ít (được phép trả 0 byte + nil error!) → cực dễ implement, và caller phải viết loop cẩn thận (nên mới có `io.ReadFull`, `io.ReadAll` làm helper). Stdlib chọn lỏng vì số implementation là vô hạn. App nội bộ thường nên chọn chặt vì caller nhiều hơn implementation.
- **Chi phí contract test**: thêm hạ tầng test (testcontainers cho S3/Postgres thật) — nhưng đây là chỗ ROI cao nhất của integration test: một bộ test khóa chặt ngữ nghĩa cho mọi backend hiện tại và tương lai.

## 6. Production Examples

- **`io.Reader`**: hợp đồng viết kỹ đến từng trường hợp biên. Vi phạm nổi tiếng nhất: implementation trả `(n > 0, EOF)` cùng lúc là *hợp lệ* nhưng nhiều caller xử lý sai — bug xuất hiện cả trong code các công ty lớn. Bài học: hợp đồng tinh tế cần helper chính chủ (`io.ReadAll`) để caller khỏi tự viết sai.
- **`context.Context`**: hợp đồng "Done() có thể được gọi nhiều lần, từ nhiều goroutine, phải trả cùng channel" — mọi custom context phải giữ, nếu không mọi thư viện dựa trên context vỡ.
- **`sql.DB` drivers**: một bài học LSP dài một thập kỷ — các driver khác nhau về hỗ trợ `LastInsertId`, named parameter, isolation level. `database/sql` xử lý bằng cách để hợp đồng lõi tối thiểu + interface tùy chọn, và app dùng ORM/query builder để che khác biệt còn lại.
- **Ecosystem TS**: Promise A+ spec chính là một contract spec cho "thenable" — có cả bộ test chuẩn (promises-aplus-tests) mà mọi thư viện promise phải pass: contract test ở quy mô cộng đồng, đúng mô hình Bước 3.

## 7. Anti-pattern

- **Type-switch trong caller**: `if s3, ok := store.(*S3Store); ok { /* đường riêng */ }` — lời thú nhận interface đã chết. Sửa tận gốc: đưa khác biệt vào hợp đồng (thêm method, thêm capability interface) hoặc hợp nhất ngữ nghĩa.
- **Method "không hỗ trợ"**: `func (s *ReadOnlyStore) Save(...) error { return errors.New("not supported") }` — thỏa cú pháp, phản bội ngữ nghĩa; caller nhận interface đầy đủ nhưng ăn lỗi runtime. Sửa: tách interface nhỏ hơn (`Loader` riêng khỏi `Saver`) — chính là ISP, chương kế tiếp. *LSP violation thường là triệu chứng; ISP là thuốc.*
- **Fake dễ dãi**: fake chấp nhận mọi thứ, không mô phỏng ràng buộc (unique key, not-found...) → unit test kiểm chứng một thế giới không tồn tại.

## 8. Khi nào KHÔNG cần nghiêm ngặt

Interface một implementation duy nhất + fake test đơn giản: hợp đồng ngầm đủ dùng, đừng viết văn bản pháp lý cho package 200 dòng. Mức đầu tư theo số lượng và độ "xa lạ" của implementation: 1 impl → doc ngắn; 2-3 impl trong repo → doc kỹ + contract test; interface public cho bên thứ ba implement → hợp đồng chi tiết + bộ conformance test là bắt buộc (như Kubernetes CSI/CNI, Terraform provider SDK đều có).

---

*Tiếp theo: [2.4 — Interface Segregation Principle](/series/software-design/level-2-principles/04-isp/)*
