+++
title = "Chương 8 — Factor 3: Config"
date = "2026-07-10T09:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> *"Store config in the environment"* — Lưu config trong môi trường (triển khai).
>
> **Phần 2 – The Twelve Factors** | Nhóm: **Configurable** | Chương trước: [Dependencies](/series/twelve-factor/07-factor-02-dependencies/) | Chương sau: [Backing Services](/series/twelve-factor/09-factor-04-backing-services/)

Đây là factor được trích dẫn nhiều nhất, bị hiểu máy móc nhiều nhất, và có nhiều cập nhật hiện đại nhất. Đọc kỹ mục Trade-off.

---

## 1. Problem Statement

**Config là gì?** Định nghĩa của 12-Factor rất chính xác: config là **mọi thứ thay đổi giữa các deploy** (dev/staging/production): URL và credential của database, cache, message broker; credential của dịch vụ ngoài (S3, payment gateway); giá trị theo môi trường (hostname, log level, port).

**Config KHÔNG phải là**: cấu hình nội tại của app không đổi theo môi trường (routing, DI wiring, hằng số nghiệp vụ như "phí ship 25k") — những thứ đó là code, cứ để trong code.

Bài toán: cùng một artifact (Factor 5) phải chạy được ở mọi môi trường. Nếu config nằm trong code:

- Đổi môi trường = sửa code + build lại → mỗi môi trường một binary khác nhau → thứ bạn test không phải thứ bạn chạy.
- **Secret nằm trong Git** — sự cố bảo mật phổ biến bậc nhất ngành. GitGuardian báo cáo ~23.8 triệu secret mới bị lộ trên GitHub công khai chỉ trong năm 2024. Một credential AWS lộ trong commit có thể bị bot quét và khai thác trong **vài phút**.
- Git history là vĩnh viễn: xóa secret ở commit sau không thu hồi được nó khỏi lịch sử — buộc phải rotate.

Bài kiểm tra gốc của 12factor.net vẫn là chuẩn vàng: **"Bạn có thể open-source codebase ngay bây giờ mà không lộ bất kỳ credential nào không?"**

## 2. Tại sao nguyên lý này tồn tại

- **Deployment**: một artifact → n môi trường chỉ khả thi khi khác biệt được bơm từ ngoài vào lúc chạy.
- **Security**: tách secret khỏi code là ranh giới bảo mật căn bản; quyền đọc code ≠ quyền truy cập production.
- **Operational**: đổi endpoint/log level phải là thao tác vận hành (nhanh, không cần build), không phải thao tác phát triển.
- **Business/Compliance**: SOC2, PCI-DSS đều yêu cầu kiểm soát và audit truy cập secret — bất khả thi nếu secret rải trong source.

## 3. Bản chất

Tách một phép nhân ra khỏi phép cộng:

```
  KHÔNG tách config:                       Tách config:
  n môi trường × m phiên bản               m artifact  +  n bộ config
  = n × m tổ hợp build                     artifact ⊗ config lúc RUN
  (mỗi tổ hợp một binary riêng,            (một binary duy nhất được test
   không cái nào được test đầy đủ)          rồi chạy khắp nơi)
```

Chú ý mệnh đề quan trọng của tài liệu gốc thường bị bỏ qua: config nên là các **biến độc lập (granular)**, không phải "profile" (`ENV=production` rồi app tự tra cứu bó config theo tên). Profile tạo ra tổ hợp bùng nổ ("staging nhưng DB của dev", "prod nhưng log debug") và các nhánh `if env == "production"` trong code — chính là config lẻn ngược vào code. App chỉ nên biết `DATABASE_URL`, không nên biết "mình đang ở staging".

**Điều gì xảy ra nếu vi phạm?** Nhẹ: mỗi lần đổi config là một lần build + deploy đầy đủ, chu kỳ vận hành chậm đi hàng chục lần. Nặng: secret vào Git → rotate toàn bộ credential, điều tra truy cập, có thể là sự cố công khai. Rất nặng: "một binary cho staging, một binary cho prod" → bug chỉ có ở binary prod, mọi tầng test mất giá trị.

## 4. Cách áp dụng với Go

### 4.1. Load config từ environment — fail fast, có kiểu, có validate

Nguyên tắc: đọc **một lần lúc khởi động**, validate ngay, crash ngay nếu thiếu — đừng để thiếu config lộ ra ở request thứ 1000 lúc 2h sáng.

```go
// internal/config/config.go
package config

import (
	"fmt"
	"time"

	"github.com/caarlos0/env/v11"
)

type Config struct {
	// Server
	Port            int           `env:"PORT" envDefault:"8080"`
	ShutdownTimeout time.Duration `env:"SHUTDOWN_TIMEOUT" envDefault:"15s"`

	// Backing services (F4): mỗi resource một URL — đổi resource không đổi code
	DatabaseURL string `env:"DATABASE_URL,required"`
	RedisURL    string `env:"REDIS_URL,required"`

	// Secret: hỗ trợ cả file (K8s mounted secret) — xem 4.3
	PaymentAPIKey string `env:"PAYMENT_API_KEY,required,file,unset"`
	// `file`  → giá trị env var là ĐƯỜNG DẪN, thư viện đọc nội dung file
	// `unset` → xóa env var sau khi đọc, giảm rủi ro rò rỉ qua /proc, child process

	// Observability
	LogLevel string `env:"LOG_LEVEL" envDefault:"info"`
}

func Load() (*Config, error) {
	cfg := &Config{}
	if err := env.Parse(cfg); err != nil {
		return nil, fmt.Errorf("parse config: %w", err)
	}
	if err := cfg.validate(); err != nil {
		return nil, fmt.Errorf("invalid config: %w", err)
	}
	return cfg, nil
}

func (c *Config) validate() error {
	if c.Port < 1 || c.Port > 65535 {
		return fmt.Errorf("PORT %d ngoài khoảng hợp lệ", c.Port)
	}
	switch c.LogLevel {
	case "debug", "info", "warn", "error":
	default:
		return fmt.Errorf("LOG_LEVEL %q không hợp lệ", c.LogLevel)
	}
	return nil
}
```

```go
// cmd/server/main.go (trích)
func main() {
	cfg, err := config.Load()
	if err != nil {
		// Fail fast: chết ngay lúc khởi động với thông điệp rõ ràng.
		// K8s sẽ thấy pod crash → không đưa traffic vào (F9 + readiness).
		log.Fatalf("config error: %v", err)
	}
	// LƯU Ý: không bao giờ log giá trị config chứa secret.
	slog.Info("config loaded", "port", cfg.Port, "log_level", cfg.LogLevel)
	// ...
}
```

Hai quy tắc code đi kèm: **(1)** chỉ đọc `os.Getenv` ở tầng config lúc khởi động — cấm rải `os.Getenv` khắp codebase (thành dependency ngầm không ai liệt kê nổi); **(2)** truyền `*Config` (hoặc các trường cần thiết) xuống qua constructor — testable, tường minh.

### 4.2. Docker & docker-compose

```bash
# Chạy local — cùng image với production, khác mỗi env
docker run --rm -p 8080:8080 \
  -e DATABASE_URL="postgres://app:dev@host.docker.internal:5432/app?sslmode=disable" \
  -e REDIS_URL="redis://host.docker.internal:6379/0" \
  -e PAYMENT_API_KEY_FILE=/run/secrets/payment_key \
  registry.example.com/myapp:9f8e7d6
```

```yaml
# docker-compose.yml — môi trường dev
services:
  app:
    build: .
    ports: ["8080:8080"]
    environment:
      DATABASE_URL: postgres://app:dev@db:5432/app?sslmode=disable
      REDIS_URL: redis://redis:6379/0
      LOG_LEVEL: debug
    env_file: .env.local        # KHÔNG commit; commit file .env.example làm mẫu
    depends_on: [db, redis]
  db:
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: dev
      POSTGRES_DB: app
  redis:
    image: redis:7-alpine
```

`.gitignore` phải có `.env*` (trừ `.env.example`). File `.env.example` liệt kê **tên** mọi biến với giá trị giả — nó chính là tài liệu sống của config.

### 4.3. Kubernetes — ConfigMap cho config thường, Secret mount file cho secret

```yaml
# configmap.yaml — config KHÔNG nhạy cảm
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
data:
  LOG_LEVEL: "info"
  SHUTDOWN_TIMEOUT: "15s"
---
# secret.yaml — thực tế nên tạo bằng External Secrets Operator / SealedSecrets,
# KHÔNG commit secret thô vào Git (xem chương 21 - GitOps)
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secrets
stringData:
  DATABASE_URL: postgres://app:S3cr3t@db.prod.internal:5432/app
  payment_api_key: pk_live_xxx
---
# deployment.yaml (trích)
spec:
  template:
    spec:
      containers:
        - name: server
          image: registry.example.com/myapp:9f8e7d6
          envFrom:
            - configMapRef: { name: myapp-config }
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef: { name: myapp-secrets, key: DATABASE_URL }
            # Secret dạng FILE cho key nhạy cảm nhất:
            - name: PAYMENT_API_KEY_FILE
              value: /var/run/secrets/app/payment_api_key
          volumeMounts:
            - name: app-secrets
              mountPath: /var/run/secrets/app
              readOnly: true
      volumes:
        - name: app-secrets
          secret:
            secretName: myapp-secrets
            items: [{ key: payment_api_key, path: payment_api_key }]
```

Vì sao secret quan trọng nên mount **file** thay vì env var? Env var dễ rò rỉ theo các đường mà bạn không kiểm soát: `kubectl describe pod` (với env từ literal), crash dump, panic handler in `os.Environ()`, process con thừa hưởng toàn bộ env, một số framework in env khi khởi động debug. File mount có quyền đọc rõ ràng, không thừa hưởng, và cập nhật được khi secret rotate (kubelet sync lại file mà không cần restart pod — nếu app hỗ trợ reload).

## 5. Trade-off — Env var vs Config file vs Secret manager

| Tiêu chí | Env var | Config file (mount) | Secret manager (Vault, ASM) |
|---|---|---|---|
| Đơn giản, phổ cập | ✅ mọi ngôn ngữ/OS | ✅ | ❌ thêm hạ tầng |
| Cấu trúc phức tạp (nested, list) | ❌ chỉ chuỗi phẳng | ✅ YAML/JSON | ✅ |
| Rủi ro rò rỉ | Trung bình (kế thừa, /proc, log) | Thấp hơn | Thấp nhất + audit log |
| Rotate không restart | ❌ env bất biến sau khi process start | ✅ (file được sync lại) | ✅ (lease, TTL) |
| Audit "ai đọc lúc nào" | ❌ | ❌ | ✅ |
| Phù hợp | Config thường, non-secret | Secret mức K8s, config phức tạp | Secret ở tổ chức có yêu cầu cao |

Khuyến nghị thực dụng (2026): **env var cho config không nhạy cảm** (đúng tinh thần F3 nguyên bản), **mounted file hoặc secret manager cho secret** (cập nhật hiện đại của F3 — nguyên tắc "tách khỏi code" giữ nguyên, cơ chế nâng cấp). Đỉnh cao là **loại bỏ hẳn secret** bằng workload identity: pod nhận IAM role (IRSA trên EKS, Workload Identity trên GKE) → gọi AWS/GCP không cần key nào tồn tại để mà lộ.

Trade-off còn lại: **fail-fast lúc start vs reload nóng**. Đọc một lần lúc start đơn giản và dễ suy luận (config của process là bất biến — cùng triết lý immutable); reload nóng (feature flags, log level) tiện vận hành nhưng thêm độ phức tạp đồng bộ. Khuyến nghị: mặc định bất biến; thứ cần đổi thường xuyên lúc runtime → tách sang hệ feature-flag chuyên dụng, đừng biến toàn bộ config thành mutable.

## 6. Best Practices

- Một bảng config duy nhất (struct `Config`), validate đủ, fail fast; `.env.example` làm tài liệu.
- Không log secret; cân nhắc kiểu `redacted string` (String() trả `"[REDACTED]"`) cho trường nhạy cảm.
- Secret không vào Git dưới mọi hình thức — kể cả repo private; bật secret scanning (gitleaks, GitHub secret scanning) trong CI.
- Rotate được là thiết kế, không phải may mắn: app đọc secret qua file/manager, hạ tầng rotate định kỳ.
- Phân biệt 3 loại giá trị: hằng số nghiệp vụ → code; thay đổi theo deploy → config; thay đổi lúc runtime theo quyết định business → feature flag.
- ConfigMap/Secret đổi tên theo nội dung (hash suffix) hoặc dùng tool như Reloader để pod tự restart khi config đổi — tránh trạng thái "ConfigMap đã đổi nhưng pod cũ vẫn chạy config cũ" âm thầm.

## 7. Anti-patterns

- **`config_prod.go`, `config_staging.go` compile theo build tag** — mỗi môi trường một binary: phá F5, thứ chạy prod không phải thứ đã test.
- **`if os.Getenv("ENV") == "production" {...}` rải trong business logic** — config profile lẻn vào code; hành vi app phân nhánh theo tên môi trường là nguồn bug "chỉ xảy ra trên prod" kinh điển.
- **Secret trong Git** — kể cả "tạm thời", kể cả base64 (không phải mã hóa!), kể cả trong `values.yaml` của Helm chart nội bộ.
- **`os.Getenv` rải rác khắp nơi, đọc lazy lúc request** — thiếu config phát nổ lúc runtime thay vì lúc start; không ai liệt kê nổi app cần biến gì.
- **Default "an toàn giả"**: `envDefault` trỏ vào DB dev cho biến bắt buộc — quên set trên prod là app *chạy được* nhưng ghi vào DB dev. Thiếu thì phải chết, không được đoán.
- **In toàn bộ env ra log lúc khởi động** để "dễ debug" — dump secret vào hệ thống log, nơi quyền đọc rộng hơn quyền đọc secret rất nhiều.

## 8. Khi nào KHÔNG cần áp dụng đầy đủ

- **CLI tool cho end-user**: config file trong `~/.config/` là UX đúng; env var chỉ là override.
- **Ứng dụng desktop/mobile**: config đi cùng bản phân phối; khái niệm "deploy environment" mờ.
- **Script một lần**: hardcode một đường dẫn có thể chấp nhận — nhưng hardcode *credential* thì không bao giờ, vì script hay bị copy/commit hơn ta tưởng.
- App nội bộ đơn giản có thể dùng config file thay vì env — miễn giữ hai nguyên tắc bất khả xâm phạm: **file config nằm ngoài Git nếu chứa secret**, và **không build riêng binary theo môi trường**.

---

## Tóm tắt

- Config = những gì **thay đổi giữa các deploy**; hằng số nghiệp vụ không phải config. Bài test: *"open-source repo ngay bây giờ có lộ credential không?"*
- Mục tiêu bất biến: một artifact ⊗ n bộ config lúc run. Cơ chế hiện đại: env var cho non-secret, mounted file / secret manager / workload identity cho secret.
- Trong Go: đọc env **một lần** lúc start vào struct có validate, fail fast; cấm `os.Getenv` rải rác; không log secret.
- Config là biến độc lập, không phải profile — app không nên biết "mình đang ở môi trường nào".
