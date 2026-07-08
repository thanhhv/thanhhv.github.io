+++
title = "Chương 15 — Factor 10: Dev/Prod Parity"
date = "2026-07-10T16:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> *"Keep development, staging, and production as similar as possible"* — Giữ các môi trường giống nhau nhất có thể.
>
> **Phần 2 – The Twelve Factors** | Nhóm: **Reproducible** | Chương trước: [Disposability](/series/twelve-factor/14-factor-09-disposability/) | Chương sau: [Logs](/series/twelve-factor/16-factor-11-logs/)

---

## 1. Problem Statement

Câu nói ám ảnh nhất ngành vận hành: **"Trên staging chạy mà!"**

Mọi tầng kiểm thử (unit test, integration test, staging, QA) đều dựa trên một giả định ngầm: **môi trường kiểm thử dự đoán được môi trường thật**. Mỗi khác biệt giữa hai môi trường đục một lỗ vào giả định đó — và bug sẽ chui đúng qua những lỗ ấy, xuất hiện *chỉ* trên production, nơi chi phí sửa cao nhất và công cụ debug nghèo nhất.

12-Factor gốc chỉ ra ba khoảng cách (gap):

- **Time gap**: code viết hôm nay, vài tuần sau mới deploy → thứ deploy là một cục thay đổi khổng lồ, không ai còn nhớ ngữ cảnh.
- **Personnel gap**: dev viết code, ops deploy → người deploy không hiểu code, người viết không thấy hậu quả vận hành.
- **Tools gap**: dev dùng SQLite/macOS, production dùng Postgres/Linux → hành vi khác nhau ở đúng chỗ không ngờ tới.

Ví dụ tools gap kinh điển làm sập hệ thống thật: SQLite chấp nhận kiểu dữ liệu lỏng lẻo (`"abc"` vào cột INTEGER — OK), Postgres từ chối → crash chỉ có trên production. Hoặc: macOS filesystem không phân biệt hoa thường, Linux có → `import "MyPackage"` chạy trên máy dev, chết trên container.

## 2. Tại sao nguyên lý này tồn tại

- **Deployment**: mục tiêu của cả hệ 12-Factor là deploy liên tục, rủi ro thấp. Deploy chỉ "nhàm chán" khi những gì xảy ra trên production đã xảy ra y hệt ở các tầng trước.
- **Operational**: sự cố production cần tái hiện được ở môi trường khác để debug; parity thấp → "không tái hiện được" → sửa mò trên production.
- **Business**: time gap lớn = vòng phản hồi chậm = chi phí sửa bug tăng theo cấp số (bug bắt ở dev rẻ hơn hàng chục lần bug bắt ở production — mọi nghiên cứu chất lượng phần mềm đồng thuận điểm này).
- **Văn hóa**: personnel gap chính là bức tường dev–ops mà phong trào DevOps sinh ra để phá; "you build it, you run it" (Werner Vogels, Amazon) là câu trả lời triệt để.

## 3. Bản chất

Ba gap là ba mặt của cùng một tệ nạn: **vòng phản hồi bị kéo dài và bị nhiễu**. Thu hẹp gap = làm cho tín hiệu "thay đổi này có an toàn không?" quay về nhanh nhất và trung thực nhất:

```
  Time gap      → thu hẹp bằng CI/CD: deploy sau MỖI merge (giờ, không phải tuần)
  Personnel gap → thu hẹp bằng văn hóa + nền tảng: dev tự deploy được an toàn
  Tools gap     → thu hẹp bằng container + backing service đồng nhất
```

Container là công cụ đắc lực nhất cho tools gap kể từ khi 12-Factor ra đời: **cùng một image** chạy từ laptop đến production (F5 — build once), OS + dependency trong image là một, khác biệt chỉ còn ở config (F3) và *cỡ* của backing service.

Nguyên tắc chi tiết đáng nhớ của tài liệu gốc: **cùng loại và cùng version backing service ở mọi môi trường**. "Dev dùng SQLite cho nhẹ, prod dùng Postgres" là cám dỗ có tên — adapter/ORM che syntax nhưng không che hành vi (locking, transaction isolation, kiểu dữ liệu, giới hạn connection). Docker đã xóa lý do tồn tại của cám dỗ này: `docker compose up` cho bạn Postgres **đúng version production** trong 5 giây.

**Parity không có nghĩa là bằng nhau tuyệt đối.** Khác biệt hợp lệ: *cỡ* (staging 1 replica DB nhỏ, prod cluster HA), *dữ liệu* (staging không có PII thật), *config* (endpoint, credential — chính là F3). Khác biệt không hợp lệ: *loại* công nghệ, *version*, *cấu trúc* (staging không có LB mà prod có → bug LB không bao giờ lộ trước).

**Điều gì xảy ra nếu vi phạm?** Một lớp bug trở nên **về nguyên tắc không thể bắt trước production**. Không phải test kém — mà là môi trường test không chứa điều kiện sinh bug. Đầu tư thêm test trên môi trường lệch chuẩn là đổ nước vào xô thủng.

## 4. Cách áp dụng

### 4.1. Tools gap: một image, backing service thật, đúng version

```yaml
# docker-compose.yml — môi trường dev PHẢN CHIẾU production
# Nguyên tắc: mọi version ở đây khớp version production
services:
  app:
    build: .                      # cùng Dockerfile với CI → cùng image prod
    ports: ["8080:8080"]
    environment:
      DATABASE_URL: postgres://app:dev@db:5432/app?sslmode=disable
      REDIS_URL: redis://redis:6379/0
      KAFKA_BROKERS: kafka:9092
      LOG_LEVEL: debug            # khác biệt hợp lệ: config
    depends_on:
      db: { condition: service_healthy }
  db:
    image: postgres:17.4-alpine    # ĐÚNG version RDS production — không phải "latest"
    environment: { POSTGRES_USER: app, POSTGRES_PASSWORD: dev, POSTGRES_DB: app }
    healthcheck: { test: ["CMD-SHELL", "pg_isready -U app"], interval: 2s, retries: 15 }
  redis:
    image: redis:7.2-alpine        # đúng version ElastiCache
  kafka:
    image: apache/kafka:3.9.0      # đúng version MSK
```

### 4.2. Integration test với backing service thật — testcontainers-go

Tầng test cũng phải parity: unit test dùng mock (nhanh, cô lập logic), nhưng integration test phải chạy trên **Postgres thật**, không phải SQLite hay mock:

```go
// internal/repository/postgres/order_test.go
package postgres_test

import (
	"context"
	"testing"

	"github.com/testcontainers/testcontainers-go/modules/postgres"
	"github.com/testcontainers/testcontainers-go"
)

func TestOrderRepo_Postgres(t *testing.T) {
	if testing.Short() {
		t.Skip("integration test")
	}
	ctx := context.Background()

	// Postgres THẬT, ĐÚNG version production, sống trong đời test rồi biến mất
	pgc, err := postgres.Run(ctx, "postgres:17.4-alpine",
		postgres.WithDatabase("app"),
		postgres.WithUsername("test"),
		postgres.WithPassword("test"),
		testcontainers.WithWaitStrategyAndDeadline(30*time.Second,
			wait.ForListeningPort("5432/tcp")),
	)
	if err != nil {
		t.Fatal(err)
	}
	t.Cleanup(func() { pgc.Terminate(ctx) })

	dsn, _ := pgc.ConnectionString(ctx, "sslmode=disable")
	pool := mustMigrateAndConnect(t, dsn) // chạy CHÍNH migration của app (F12)

	repo := NewOrderRepo(pool)
	// ... test hành vi thật: transaction, constraint, kiểu dữ liệu —
	// những thứ mock không bao giờ bắt được
}
```

### 4.3. Time gap: pipeline biến "merge" thành "đang chạy production" trong vòng giờ

```
  PR → CI (test + build image) → auto-deploy staging → smoke test
     → deploy production (tự động hoặc một nút bấm) — cùng image, khác config
```

Kèm kỷ luật: thay đổi nhỏ, merge thường xuyên; feature chưa xong giấu sau feature flag thay vì nằm ở branch lâu ngày (branch sống lâu là time gap trá hình).

### 4.4. Structure parity trên Kubernetes: cùng manifest, khác giá trị

```
deploy/k8s/
├── base/                        # CẤU TRÚC dùng chung — một nguồn sự thật
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
└── overlays/
    ├── staging/                 # chỉ GHI ĐÈ giá trị: replicas, resources, hostname
    │   └── kustomization.yaml   #   (2 replica, DB nhỏ)
    └── production/
        └── kustomization.yaml   #   (10 replica, HPA, PDB)
```

Kustomize/Helm đảm bảo staging và production **cùng cấu trúc**, khác duy nhất phần giá trị được khai báo tường minh trong overlay — diff hai môi trường là diff hai file ngắn, không phải so hai đống YAML chép tay.

## 5. Trade-off

**Chi phí của parity vs xác suất × chi phí của bug.** Parity đầy đủ có giá thật: staging "giống prod" tốn tiền (cụm K8s riêng, DB riêng, license); Kafka trên laptop dev nặng nề. Nguyên tắc đầu tư: parity ở những chiều **hay sinh bug nhất** — loại + version backing service (rẻ, nhờ Docker), cấu trúc deploy (rẻ, nhờ overlay), OS/kiến trúc runtime (miễn phí, nhờ container). Chấp nhận lệch ở chiều đắt: quy mô, dữ liệu, traffic.

**Staging có bao giờ "đủ giống" không?** Không — traffic thật, dữ liệu thật, quy mô thật là thứ staging không bao giờ có. Kết luận trưởng thành của ngành: thu hẹp gap *đến mức kinh tế*, phần còn lại xử lý bằng **giảm bán kính vụ nổ trên production**: canary release, feature flags, progressive delivery (chương 23), observability tốt (chương 22). "Testing in production" một cách *có kiểm soát* không phải trò đùa — nó là sự thừa nhận rằng production là môi trường duy nhất trung thực 100%.

**Dev laptop: giả lập đến đâu?** Chạy đủ 15 service + Kafka trên laptop là trải nghiệm khổ sở. Các đáp án theo mức đầu tư: docker-compose chỉ những backing service của service đang sửa (đa số team); môi trường dev trên cloud (Gitpod/Codespaces, telepresence); preview environment per-PR (đỉnh của trò chơi — mỗi PR một môi trường thật, tự hủy khi merge).

## 6. Best Practices

- Một Dockerfile, một image cho mọi môi trường (F5); khác biệt chỉ ở env/overlay.
- Version backing service ghim và đồng bộ: nâng RDS thì nâng docker-compose + testcontainers cùng một PR.
- Integration test trên engine thật (testcontainers); CI chạy trên Linux cùng kiến trúc production (chú ý dev Mac ARM vs prod x86 — build multi-arch hoặc test trên CI là chốt chặn).
- Deploy staging tự động sau mỗi merge; giữ thời gian merge→production dưới một ngày.
- Kustomize/Helm: base chung + overlay mỏng; cấm chép tay manifest giữa môi trường.
- Dữ liệu staging: anonymized/synthetic có hình dạng giống thật (cả *độ lớn* — query nhanh trên 1k dòng có thể chết trên 10M dòng).
- "You build it, you run it": team viết service trực on-call service đó — đóng personnel gap bằng cơ chế, không bằng khẩu hiệu.

## 7. Anti-patterns

- **SQLite ở dev, Postgres ở prod** — poster child của tools gap; khác biệt hành vi ở transaction, type, concurrency chỉ lộ trên production.
- **Mock backing service trong integration test** ("Postgres giả bằng map") — test pass chứng minh mock hoạt động, không chứng minh app hoạt động.
- **Staging lệch cấu trúc** (không có ingress/LB/CDN mà prod có; 1 replica trong khi prod 10) — cả lớp bug về routing, header, race giữa các replica trở nên vô hình.
- **Staging làm bãi rác** — config trôi dạt, dữ liệu hỏng, service version cũ ba tháng; kết quả test trên đó là nhiễu, mọi người học cách bỏ qua staging fail → staging chết chức năng.
- **Branch release sống hàng tháng** — time gap khổng lồ; ngày merge là ngày hội tích hợp đau đớn.
- **"Ops sẽ deploy giúp"** — personnel gap được thể chế hóa; dev không bao giờ học được hậu quả vận hành của code mình.
- **Nâng version DB trên prod trước, môi trường khác "nâng sau"** — đảo ngược chiều an toàn; mọi nâng cấp phải đi dev → staging → prod.

## 8. Khi nào KHÔNG cần áp dụng đầy đủ

- **Chi phí license/hạ tầng chặn parity**: không ai dựng Oracle RAC hay mainframe trên laptop — dùng container bản gần nhất có thể (Oracle Free), dồn test lệch-môi-trường lên một staging chung và chấp nhận rủi ro có ghi nhận.
- **Dịch vụ bên thứ ba không có sandbox tốt** (một số payment gateway): mock/contract test ở dev, test thật ở staging với tài khoản test — gap có ý thức, có giám sát.
- **Hardware-dependent** (GPU đặc thù, thiết bị nhúng): dev trên simulator, chấp nhận tầng test cuối phải chạy trên phần cứng thật.
- **Solo dev, hệ nhỏ**: có thể bỏ staging, deploy thẳng — nhưng đừng bỏ *tools parity* (docker-compose đúng version DB gần như miễn phí, và là thứ cứu bạn nhiều nhất).

---

## Tóm tắt

- Ba gap: **time** (viết → deploy lâu), **personnel** (người viết ≠ người vận hành), **tools** (công nghệ dev ≠ prod). Bug production-only chui qua đúng các gap này.
- Container + compose + testcontainers đã làm tools parity gần như miễn phí: **cùng image, cùng loại, cùng version backing service** — không còn lý do cho SQLite-ở-dev.
- Parity ≠ bằng nhau: khác *cỡ* và *dữ liệu* là hợp lệ; khác *loại*, *version*, *cấu trúc* là không.
- Staging không bao giờ giống prod 100% — phần dư xử lý bằng canary, feature flags, observability: giảm bán kính vụ nổ thay vì mơ môi trường hoàn hảo.
