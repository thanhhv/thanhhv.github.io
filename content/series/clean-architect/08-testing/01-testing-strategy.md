+++
title = "Chương 8 — Testing Strategy: Unit, Integration, Mock vs Fake, Testcontainers"
date = "2026-07-08T07:00:00+07:00"
draft = false
tags = ["backend", "golang", "clean-architecture"]
series = ["Clean Architecture với Golang"]
+++

> **Level 3 – Senior** · Testability không phải phần thưởng phụ của Clean Architecture — nó là *bằng chứng* kiến trúc đúng. Kiến trúc không test được là kiến trúc sai, bất kể sơ đồ đẹp đến đâu.

---

## 1. Problem Statement

Hai thái cực thất bại phổ biến:

- **Test kim tự tháp ngược**: 500 E2E test dựng cả hệ thống, chạy 40 phút, flaky 5% — CI thành xổ số, dev tắt test để merge. Nguyên nhân gốc: nghiệp vụ dính hạ tầng nên *chỉ có thể* test kiểu E2E.
- **Mock hóa toàn phần**: mock mọi thứ kể cả struct nội bộ, assert từng lời gọi hàm — test khẳng định *code viết như thế nào* thay vì *hệ thống làm gì*. Refactor giữ nguyên hành vi vẫn vỡ 200 test → test trở thành lực cản refactor, phản bội chính mục đích của nó.

Chiến lược đúng bám vào cấu trúc các vòng: **mỗi vòng một loại test, chi phí thấp nhất có thể cho độ tin cần thiết.**

## 2. Bản đồ test theo vòng

```
┌───────────────────────────────────────────────────────────────────┐
│ Vòng            │ Loại test        │ Thay thế gì     │ Tốc độ      │
├───────────────────────────────────────────────────────────────────┤
│ Domain          │ Unit thuần       │ Không cần thay  │ µs          │
│ Use case        │ Unit + fake      │ Fake các cổng   │ ms          │
│ Adapter DB      │ Integration      │ DB thật (container)│ giây     │
│ Adapter HTTP    │ httptest         │ Fake use case*  │ ms          │
│ Toàn hệ thống   │ E2E ít, critical │ Không thay gì   │ chục giây   │
└───────────────────────────────────────────────────────────────────┘
```

Phân bổ điển hình lành mạnh: ~70% unit (domain + use case), ~25% integration (adapter), ~5% E2E (vài happy path + luồng tiền bạc). Số lượng không phải mục tiêu — **mục tiêu là mỗi rule nghiệp vụ có test ở tầng rẻ nhất diễn đạt được nó.**

(*) Muốn fake được use case từ handler, handler nên phụ thuộc interface của use case — hoặc thực dụng hơn: test handler cùng use case thật + fake repo, coi "handler+usecase" là một khối.

## 3. Mock vs Fake — chọn đúng vũ khí

Định nghĩa gọn: **Fake** = implementation thật nhưng nhẹ (map trong RAM có đủ ngữ nghĩa contract). **Mock** = object lập trình sẵn kỳ vọng lời gọi và trả lời đóng hộp (mockgen/mockery). **Stub** = mock chỉ trả dữ liệu, không assert lời gọi.

**Khuyến nghị mạnh: mặc định dùng fake, dè chừng mock.** Lý do:

- Fake test *trạng thái* ("sau khi Execute, ví còn 500 điểm") — bất biến qua refactor. Mock test *tương tác* ("Save được gọi đúng 1 lần với đúng tham số") — vỡ khi đổi cách gọi dù kết quả y nguyên.
- Một fake viết một lần phục vụ mọi test của module; mock setup lại từng test, 15 dòng `EXPECT()` cho 3 dòng logic.
- Fake buộc bạn hiểu contract thật (phải giả lập ngữ nghĩa not-found, duplicate...) — chính việc hiểu đó phát hiện lỗi thiết kế contract.

```go
// FakeWalletRepo — dùng chung cho toàn bộ test của module loyalty
type FakeWalletRepo struct {
	mu      sync.Mutex
	wallets map[string]domain.Wallet
	// điểm tiêm lỗi cho test đường buồn
	FailSaveWith error
}

func NewFakeWalletRepo(seed ...domain.Wallet) *FakeWalletRepo {
	f := &FakeWalletRepo{wallets: map[string]domain.Wallet{}}
	for _, w := range seed { f.wallets[w.CustomerID()] = w }
	return f
}

func (f *FakeWalletRepo) ByCustomer(_ context.Context, id string) (domain.Wallet, error) {
	f.mu.Lock(); defer f.mu.Unlock()
	w, ok := f.wallets[id]
	if !ok { return domain.Wallet{}, domain.ErrWalletNotFound } // đúng contract!
	return w, nil
}

func (f *FakeWalletRepo) Save(_ context.Context, w domain.Wallet) error {
	if f.FailSaveWith != nil { return f.FailSaveWith }
	f.mu.Lock(); defer f.mu.Unlock()
	f.wallets[w.CustomerID()] = w
	return nil
}
```

Mock còn chỗ đứng khi *tương tác chính là điều cần khẳng định*: "email chỉ gửi một lần", "audit log ghi trước khi ghi DB". Khi đó một mock tay 10 dòng (như `fakeNotifier` chương 1.3) thường vẫn đủ — codegen mock chỉ đáng khi interface lớn và nhiều (mà interface lớn tự nó là red flag, chương ISP).

## 4. Ba tầng test qua ví dụ

### 4.1 Domain — table-driven, thuần túy

Đã minh họa chương 2.3. Đặc điểm: không context, không goroutine, không I/O — chỉ input/output. Đây là nơi test *mọi nhánh* của rule (đủ điểm/thiếu điểm/chạm limit/qua ngày mới). Nếu bạn thấy khó viết test kiểu này — entity của bạn đang phụ thuộc gì đó, đi tìm và cắt.

### 4.2 Use case — kịch bản nghiệp vụ với fake

```go
func TestRedeem_VoucherLoi_KhongLuuVi(t *testing.T) {
	repo := NewFakeWalletRepo(domain.RehydrateWallet("c1", domain.TierGold, 1000, 0, time.Time{}))
	vouchers := &FakeVoucherIssuer{FailWith: errors.New("voucher service down")}
	uc := usecase.NewRedeem(repo, vouchers, &FakeEvents{}, fixedNow)

	_, err := uc.Execute(context.Background(), usecase.RedeemInput{CustomerID: "c1", Cost: 100})

	if err == nil { t.Fatal("phai tra loi") }
	w, _ := repo.ByCustomer(context.Background(), "c1")
	if w.Balance() != 1000 {
		t.Fatalf("vi bi tru du voucher loi: %d", w.Balance()) // test TRẠNG THÁI
	}
}
```

Use case test là **đặc tả quy trình bằng code chạy được** — reviewer đọc tên test hiểu chính sách hệ thống: `TestRedeem_EventLoi_VanThanhCong`, `TestRedeem_SaveLoi_TraLoi`.

### 4.3 Adapter — integration với testcontainers

Đã minh họa chương 5 mục 5. Bổ sung các kỷ luật production:

- **Container một lần cho cả package** (`TestMain` hoặc `sync.Once`), mỗi test một schema/truncate — 30 test integration chạy trong ~10s thay vì 5 phút.
- **Contract test dùng chung** giữa fake và implementation thật — đảm bảo fake không nói dối:

```go
// Chạy CÙNG bộ test cho cả fake lẫn Postgres — LSP bằng hành động
func TestFakeWalletRepo_Contract(t *testing.T) {
	RunWalletRepoContract(t, func(t *testing.T) usecase.WalletRepo { return NewFakeWalletRepo() })
}
func TestPostgresWalletRepo_Contract(t *testing.T) {
	RunWalletRepoContract(t, newPostgresRepoWithContainer)
}
```

- Handler HTTP test bằng `httptest` — khẳng định status code, JSON shape, dịch lỗi:

```go
func TestHandler_Redeem_HetDiem_Tra422(t *testing.T) {
	h := httpapi.NewHandler(usecaseWithFakes(walletCoBalance(10)))
	req := httptest.NewRequest("POST", "/v1/customers/c1/redeem",
		strings.NewReader(`{"cost": 100}`))
	req.SetPathValue("customerID", "c1")
	rec := httptest.NewRecorder()

	h.Redeem(rec, req)

	if rec.Code != http.StatusUnprocessableEntity {
		t.Fatalf("status = %d, body = %s", rec.Code, rec.Body)
	}
}
```

## 5. Trade-off & thực tế

- **Fake là code phải bảo trì** — thêm method vào port = sửa fake. Chi phí thật, trả bằng: port hẹp (ISP) và contract test giữ fake trung thực.
- **Integration test cần Docker trong CI** — setup một lần, nhưng là ma sát thật với môi trường hạn chế; phương án phụ: dịch vụ DB của CI (Github Actions services) — mất tính "mỗi test một môi trường sạch".
- **Coverage không phải mục tiêu.** 100% coverage với test khẳng định `err == nil` là con số trang trí; 70% coverage tập trung vào rule và đường buồn giá trị hơn nhiều. Đo thứ có ý nghĩa: mọi sentinel error có test tạo ra nó; mọi nhánh quyết định của use case có kịch bản.
- **Test song song**: `t.Parallel()` mặc định cho unit; fake phải thread-safe (mutex như trên).

## 6. Anti-patterns

- **Assert chuỗi SQL / assert số lần gọi hàm nội bộ** — test cài xi măng vào implementation.
- **Test qua HTTP cho rule domain**: dựng server + DB để kiểm tra "GOLD nhân đôi" — đắt gấp 10.000 lần test hàm `EarnFromPurchase`.
- **Sleep-based test** cho code async: `time.Sleep(100ms)` rồi assert — flaky vĩnh viễn. Inject clock, dùng channel đồng bộ, hoặc polling có deadline.
- **Fixture khổng lồ dùng chung**: một `seed.sql` 500 dòng mọi test phụ thuộc — sửa một test vỡ mười test. Mỗi test tự dựng dữ liệu tối thiểu của nó (builder/helper).
- **Test private function bằng export_test.go tràn lan** — dấu hiệu unit quá to hoặc test sai tầng; test qua API công khai của package.

## Tóm tắt

- Test theo vòng: domain thuần µs, use case + fake ms, adapter + container giây, E2E nhỏ giọt.
- Fake > mock cho cổng dữ liệu; mock chỉ khi tương tác là điều cần khẳng định; contract test giữ mọi implementation trung thực.
- Test trạng thái và hành vi công khai, không test cấu trúc nội bộ — để test là lưới an toàn cho refactor, không phải xích chân.

**Chương tiếp theo:** [Clean Architecture & DDD](/series/clean-architect/09-ddd/01-clean-architecture-va-ddd/)
