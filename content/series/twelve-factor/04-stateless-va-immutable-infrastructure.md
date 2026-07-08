+++
title = "Chương 4: Stateless và Immutable Infrastructure — Hai khái niệm nền móng"
date = "2026-07-10T05:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> **Phần 1 – Foundation** | Chương trước: [Cloud Native](/series/twelve-factor/03-cloud-native/) | Chương sau: [Vì sao 12-Factor App ra đời](/series/twelve-factor/05-vi-sao-12-factor-ra-doi/)

Nếu phải chọn hai khái niệm mà toàn bộ 12-Factor xoay quanh, đó là **Stateless** (ở tầng ứng dụng) và **Immutable Infrastructure** (ở tầng hạ tầng). Hiểu sâu hai khái niệm này thì 12 chương factor phía sau trở nên hiển nhiên.

---

## PHẦN A — STATELESS

## 1. Problem Statement

**Stateless bị hiểu sai phổ biến nhất là "ứng dụng không có state".** Điều đó vô nghĩa — mọi ứng dụng hữu ích đều có state (user, đơn hàng, session...). Định nghĩa đúng:

> **Stateless process = process không giữ bất kỳ state nào cần tồn tại qua request tiếp theo. Mọi state cần bền vững được đẩy ra backing service (database, cache, object storage).**

State vẫn tồn tại — nó chỉ **không nằm trong process**. Câu hỏi phân loại:

```
State này nếu process chết thì có gây hại không?
│
├── KHÔNG (cache tính lại được, connection pool, buffer)
│   → Được phép giữ trong process. Đây là "ephemeral state" hợp lệ.
│
└── CÓ (session, file user upload, đơn hàng đang xử lý, job queue)
    → BẮT BUỘC đẩy ra backing service.
```

### Nếu process stateful thì điều gì xảy ra?

Nhắc lại câu hỏi vàng của chương 1: *"chạy 2 bản sau load balancer thì hỏng gì?"* — nhưng giờ đi xa hơn. Process stateful phá vỡ **từng khả năng một** của nền tảng hiện đại:

- **Không load-balance tự do được** → cần sticky session → tải lệch, một máy chết là mất một mảng user.
- **Không autoscale-down được** → giết instance là mất state → chỉ dám scale lên, không dám scale xuống.
- **Không rolling update được** → deploy phiên bản mới đòi hỏi giết phiên bản cũ → mất state mỗi lần deploy → "deploy vào 2h sáng Chủ nhật".
- **Không tự phục hồi được** → nền tảng phát hiện instance treo nhưng không dám thay, vì thay là mất dữ liệu.

Để ý mẫu số chung: **stateful process biến mọi thao tác tự động của nền tảng thành thao tác nguy hiểm**, do đó vô hiệu hóa toàn bộ giá trị của tự động hóa.

## 2. Cách áp dụng với Golang

### Ví dụ refactor: từ session in-memory sang Redis

**Trước (stateful — anti-pattern):**

```go
// ❌ STATEFUL: session chết cùng process
var sessions = struct {
	sync.RWMutex
	m map[string]Session
}{m: map[string]Session{}}

func login(w http.ResponseWriter, r *http.Request) {
	sid := newSessionID()
	sessions.Lock()
	sessions.m[sid] = Session{UserID: authenticate(r)}
	sessions.Unlock()
	http.SetCookie(w, &http.Cookie{Name: "sid", Value: sid})
}
```

**Sau (stateless — state đẩy ra Redis):**

```go
// ✅ STATELESS: process nào cũng đọc được session, process chết không mất gì
package session

import (
	"context"
	"encoding/json"
	"time"

	"github.com/redis/go-redis/v9"
)

type Store struct {
	rdb *redis.Client
	ttl time.Duration
}

func NewStore(rdb *redis.Client, ttl time.Duration) *Store {
	return &Store{rdb: rdb, ttl: ttl}
}

type Session struct {
	UserID string `json:"user_id"`
	Role   string `json:"role"`
}

func (s *Store) Save(ctx context.Context, sid string, sess Session) error {
	b, err := json.Marshal(sess)
	if err != nil {
		return err
	}
	return s.rdb.Set(ctx, "sess:"+sid, b, s.ttl).Err()
}

func (s *Store) Get(ctx context.Context, sid string) (*Session, error) {
	b, err := s.rdb.Get(ctx, "sess:"+sid).Bytes()
	if err != nil {
		return nil, err // redis.Nil nếu không tồn tại
	}
	var sess Session
	if err := json.Unmarshal(b, &sess); err != nil {
		return nil, err
	}
	return &sess, nil
}
```

Một lựa chọn khác cho session là **JWT/stateless token** — state nằm ngay trong token phía client, không cần cả Redis. Trade-off: không thu hồi (revoke) được token trước khi hết hạn, kích thước cookie lớn hơn. Bài toán quyết định: cần revoke tức thì (banking) → session store; chấp nhận TTL ngắn (API nội bộ) → JWT.

### Ephemeral state hợp lệ — đừng cực đoan

```go
// ✅ HỢP LỆ trong process stateless:
var (
	// Connection pool: mất đi thì tạo lại, không phải "dữ liệu"
	dbPool *pgxpool.Pool

	// Local cache có TTL: chỉ là tối ưu, miss thì đọc lại từ nguồn.
	// Lưu ý: cache local trên N instance sẽ có N bản có thể lệch nhau
	// trong TTL — chấp nhận được với dữ liệu đọc nhiều ít đổi
	// (feature flags, config), KHÔNG chấp nhận được với số dư tài khoản.
	localCache = expirable.NewLRU[string, Product](1000, nil, 30*time.Second)
)
```

Nguyên tắc: **stateless là về tính đúng đắn (correctness), không cấm tối ưu (optimization)**. Cache local sai lệch trong 30 giây có làm hệ thống *sai* không? Nếu không — cứ dùng, nó giảm tải backing service đáng kể.

---

## PHẦN B — IMMUTABLE INFRASTRUCTURE

## 3. Problem Statement

Chương 1 đã mô tả configuration drift: server bị "vá" dần qua năm tháng thành một bông tuyết không ai tái tạo nổi. Immutable Infrastructure là câu trả lời triệt để:

> **Không bao giờ sửa đổi hạ tầng đang chạy. Muốn thay đổi bất kỳ điều gì — version app, config OS, thư viện — hãy build một image mới, tạo instance mới từ image đó, và hủy instance cũ.**

So sánh với mô hình cũ:

```
   MUTABLE (truyền thống)                IMMUTABLE (hiện đại)
  ──────────────────────────           ──────────────────────────
   Server sống lâu                       Instance sống ngắn
   SSH vào sửa/update tại chỗ            Không SSH để sửa; sửa = thay mới
   Trạng thái = tích lũy lịch sử         Trạng thái = hàm của (image, config)
   Rollback = "sửa ngược lại" (cầu may)  Rollback = deploy lại image cũ (chắc chắn)
   Drift tất yếu theo thời gian          Drift không thể xảy ra theo thiết kế
```

Điểm sâu sắc nhất: immutable infrastructure biến trạng thái hệ thống từ **kết quả của một quá trình lịch sử** (không thể suy luận) thành **kết quả của một hàm thuần túy** `state = f(image, config)` (hoàn toàn suy luận được). Đây là cùng một bước nhảy mà functional programming mang lại cho code: loại bỏ mutation để loại bỏ cả một lớp bug.

## 4. Cách áp dụng

Đơn vị immutable phổ biến nhất hiện nay là **container image**. Quy trình chuẩn:

```
  Git commit ──> CI build ──> Image (immutable, có digest sha256)
                                │
                                ├──> deploy lên staging   (cùng image)
                                ├──> deploy lên production (cùng image)
                                └──> rollback = deploy lại image trước đó
```

```dockerfile
# Dockerfile — artifact immutable
FROM golang:1.24-alpine AS build
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -trimpath -ldflags="-s -w" -o /out/app ./cmd/server

FROM gcr.io/distroless/static-debian12:nonroot
COPY --from=build /out/app /app
# Image này không có shell, không có package manager —
# về mặt vật lý KHÔNG THỂ "SSH vào sửa". Immutable by construction.
ENTRYPOINT ["/app"]
```

Hai quy tắc thực hành quan trọng đi kèm:

**Pin theo digest, không tin tag mutable.** Tag `myapp:latest` (và cả `myapp:v1.2` nếu registry cho phép ghi đè) có thể trỏ tới image khác nhau ở hai thời điểm — tức là mutable. Production nên tham chiếu `myapp@sha256:abc123...` hoặc tag bất biến có kèm digest.

**Filesystem của container là read-only.** Kubernetes cho phép ép điều này:

```yaml
# Trích deployment.yaml
securityContext:
  readOnlyRootFilesystem: true   # ghi vào filesystem → lỗi ngay, lộ anti-pattern sớm
  runAsNonRoot: true
volumes:
  - name: tmp
    emptyDir: {}                 # chỗ ghi tạm hợp lệ duy nhất (nếu thật sự cần)
```

`readOnlyRootFilesystem: true` là một "máy phát hiện vi phạm stateless" tuyệt vời: mọi đoạn code lén ghi file local sẽ fail ngay ở môi trường test thay vì âm thầm tích state trên production.

## 5. Trade-off

**Stateless:**

- *Được*: scale tự do, tự phục hồi, deploy không sợ hãi. *Mất*: mọi state đi qua network → cộng 0.5–2ms mỗi lần chạm Redis so với ~100ns đọc RAM (chênh ~4 bậc độ lớn — nhưng hãy so với tổng latency request thường 50–200ms để đánh giá đúng mức độ).
- Hệ thống **thật sự stateful về bản chất** (database, message broker, game server giữ world state, WebSocket long-lived có state phiên phức tạp) không ép stateless được — chúng cần công cụ khác (StatefulSet, consensus, session migration). 12-Factor áp dụng cho tầng *application*, không phải tầng *data*.

**Immutable Infrastructure:**

- *Được*: rollback tin cậy, môi trường đồng nhất, audit được (mọi thay đổi là một commit + một image). *Mất*: chu trình thay đổi dài hơn — sửa một dòng config cũng phải qua build-deploy (vài phút) thay vì sửa tại chỗ (vài giây). Với hotfix khẩn cấp, vài phút đó gây khó chịu thật; kỷ luật nằm ở chỗ chấp nhận nó, vì "sửa nhanh tại chỗ" chính là cánh cửa đưa drift quay lại.
- Chi phí storage/băng thông cho image mỗi lần build — thực tế nhỏ, và multi-stage build + layer caching (chương 19) giảm đáng kể.

## 6. Anti-patterns

- **`kubectl exec` vào container để sửa config/cài tool trên production** — tái tạo snowflake server bên trong container. Thay đổi biến mất ở lần restart kế tiếp, tạo ra "sự cố ma" không tái hiện được.
- **Container tự update bản thân lúc runtime** (`apt-get upgrade` trong entrypoint, auto-updater trong app) — hai instance cùng image trở nên khác nhau → mất reproducibility.
- **VM "golden image" nhưng sau đó vẫn SSH vá tay** — immutable nửa vời còn nguy hiểm hơn mutable công khai, vì nó tạo ảo giác an toàn.
- **Dùng `:latest` trên production** — không biết đang chạy gì, không rollback được về "cái trước đó" vì không ai biết "cái trước đó" là gì.

## 7. Khi nào KHÔNG áp dụng

- **Stateless**: database và hệ stateful bản chất (nêu trên); ứng dụng desktop/CLI đơn máy; batch job một bản duy nhất có checkpoint local (dù checkpoint ra object storage vẫn tốt hơn).
- **Immutable**: môi trường dev cá nhân (hot-reload để lặp nhanh là hợp lý — miễn là CI build lại image chuẩn); phần cứng đặc thù không ảo hóa được; hệ thống pet có chủ đích với đầy đủ nhận thức về chi phí (một máy Oracle license theo core chẳng hạn).

---

## Tóm tắt chương

- **Stateless ≠ không có state.** State vẫn tồn tại nhưng nằm ở backing service; process chỉ giữ ephemeral state vô hại. Câu hỏi phân loại: *"process chết thì state này mất có gây hại không?"*
- Stateful process không chỉ khó scale — nó **vô hiệu hóa mọi khả năng tự động của nền tảng** (load balancing, autoscaling, rolling update, self-healing).
- **Immutable Infrastructure**: thay đổi = thay mới, không sửa tại chỗ. Biến trạng thái hệ thống thành hàm thuần túy `f(image, config)` → rollback tin cậy, drift bất khả thi.
- Hai công cụ ép kỷ luật rẻ tiền: distroless image (không có shell để nghịch) và `readOnlyRootFilesystem: true` (ghi file là lỗi ngay).

Chương tiếp theo: ghép tất cả nền móng lại — [Vì sao 12-Factor App ra đời](/series/twelve-factor/05-vi-sao-12-factor-ra-doi/).
