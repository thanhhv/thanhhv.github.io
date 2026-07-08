+++
title = "Chương 17 — Factor 12: Admin Processes"
date = "2026-07-10T18:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> *"Run admin/management tasks as one-off processes"* — Chạy tác vụ quản trị như các process một lần.
>
> **Phần 2 – The Twelve Factors** | Nhóm: **Operable** | Chương trước: [Logs](/series/twelve-factor/16-factor-11-logs/) | Chương sau: [Golang Patterns](/series/twelve-factor/18-golang-patterns/)

---

## 1. Problem Statement

Ngoài dòng traffic chính, hệ thống nào cũng có **tác vụ quản trị**: database migration, backfill dữ liệu, sửa dữ liệu hỏng một lần, chạy lại job fail, script thống kê, xóa dữ liệu hết hạn theo lịch. Câu hỏi của factor này: **các tác vụ đó chạy ở đâu, bằng code nào, với môi trường nào?**

Các cách làm sai kinh điển — và hậu quả:

- **Script "cứu hỏa" viết vội trên máy dev, chạy thẳng vào DB production** bằng Python trong khi app là Go: script dùng logic *khác* app (validate khác, encoding khác, timezone khác) → "sửa dữ liệu" tạo ra dữ liệu hỏng kiểu mới. Script không nằm trong Git, không review, không chạy lại được.
- **SSH vào server production, mở psql gõ UPDATE tay** — không transaction, không backup trước, gõ thiếu WHERE. Mọi DBA lâu năm đều có một câu chuyện kinh hoàng bắt đầu như thế.
- **Cron đặt tay trên "server số 1"** — tri thức nằm ngoài codebase (vi phạm F1), chết cùng server, và nhân bản khi scale (đã bàn ở F6).
- **Migration chạy tự động trong `main()` của app** — N pod đua nhau migrate (đã bàn ở F5).

Mẫu số chung: tác vụ quản trị chạy với **code khác, môi trường khác, kỷ luật khác** so với app — trong khi nó đụng vào *cùng một dữ liệu* với quyền còn cao hơn.

## 2. Tại sao nguyên lý này tồn tại

- **Correctness**: tác vụ quản trị thao tác dữ liệu thật với quyền lớn; nó cần *cùng logic* với app (cùng model, cùng validation, cùng serialization) — khác một ly là dữ liệu đi một dặm.
- **Operational**: sự cố cần được xử lý bằng công cụ *lặp lại được, có dấu vết* — "đã chạy script X phiên bản Y lúc Z, output đây" thay vì "hình như anh A có gõ gì đó tối qua".
- **Deployment**: schema migration là một phần của release (F5) — nó phải đi qua cùng pipeline, cùng review, cùng rollback plan với code.
- **Compliance/Security**: truy cập production tùy tiện (SSH + psql) là thứ audit SOC2/ISO không bao giờ cho qua; one-off process qua nền tảng có RBAC + log là câu trả lời có kiểm soát.

## 3. Bản chất

Nguyên tắc gói trong một câu:

> **Tác vụ quản trị là công dân hạng nhất của codebase: cùng code, cùng release, cùng config, cùng môi trường thực thi với app — chỉ khác ở chỗ chạy một lần rồi thoát.**

Cụ thể hóa "cùng" theo từng factor đã học:

```
  Cùng CODEBASE (F1):  lệnh admin nằm trong repo, được review, có lịch sử
  Cùng RELEASE  (F5):  chạy từ CHÍNH image đã build — không phải script rời
  Cùng CONFIG   (F3):  đọc DATABASE_URL từ env như app — không hardcode DSN nào khác
  Cùng DEPENDENCIES (F2): image chứa mọi thứ cần — không "máy tôi có psql"
  Log ra stdout (F11): output được thu gom như mọi process khác
  Chạy như PROCESS (F6): sinh ra, làm việc, thoát — không sống lâu, không giữ state
```

Hình dung đúng nhất: app của bạn có nhiều *cách chạy* — `server` (dài hạn, phục vụ), `worker` (dài hạn, tiêu thụ queue), và các *one-off*: `migrate`, `backfill`, `cleanup`. Tất cả là **một binary từ một image**, khác nhau ở subcommand.

**Điều gì xảy ra nếu vi phạm?** Hai hệ sinh thái logic song song hình thành: app một đằng, "đống script vận hành" một nẻo — trôi dạt dần theo thời gian, và một ngày script cũ 8 tháng chạy vào schema mới. Cộng thêm: không audit được ai đã làm gì với dữ liệu production.

## 4. Cách áp dụng với Go

### 4.1. Một binary, nhiều subcommand

```go
// cmd/app/main.go — một entrypoint cho mọi vai
package main

import (
	"context"
	"fmt"
	"os"
)

func main() {
	if len(os.Args) < 2 {
		fmt.Fprintln(os.Stderr, "usage: app <server|worker|migrate|backfill-emails|cleanup-expired>")
		os.Exit(2)
	}
	cfg := config.MustLoad()          // CÙNG config (F3)
	logger := logging.New(cfg.LogLevel) // CÙNG logging (F11)
	ctx := context.Background()

	var err error
	switch os.Args[1] {
	case "server":
		err = runServer(ctx, cfg, logger)      // process dài hạn
	case "worker":
		err = runWorker(ctx, cfg, logger)      // process dài hạn
	case "migrate":
		err = runMigrate(ctx, cfg, logger)     // one-off (F12)
	case "backfill-emails":
		err = runBackfillEmails(ctx, cfg, logger) // one-off (F12)
	case "cleanup-expired":
		err = runCleanup(ctx, cfg, logger)     // định kỳ — CronJob gọi (F12)
	default:
		fmt.Fprintf(os.Stderr, "unknown command %q\n", os.Args[1])
		os.Exit(2)
	}
	if err != nil {
		logger.Error("command failed", "cmd", os.Args[1], "err", err)
		os.Exit(1) // exit code chuẩn — nền tảng biết job fail để retry/alert
	}
}
```

### 4.2. Migration — embed vào binary, dùng chung mọi thứ với app

```go
// internal/migrate/migrate.go — golang-migrate với file SQL embed trong binary:
// image TỰ CHỨA schema của chính nó (F2), không cần mount gì thêm
package migrate

import (
	"embed"

	"github.com/golang-migrate/migrate/v4"
	_ "github.com/golang-migrate/migrate/v4/database/postgres"
	"github.com/golang-migrate/migrate/v4/source/iofs"
)

//go:embed sql/*.sql
var migrations embed.FS

func Up(databaseURL string) error {
	src, err := iofs.New(migrations, "sql")
	if err != nil {
		return err
	}
	m, err := migrate.NewWithSourceInstance("iofs", src, databaseURL)
	if err != nil {
		return err
	}
	defer m.Close()
	if err := m.Up(); err != nil && err != migrate.ErrNoChange {
		return err
	}
	return nil
}
```

### 4.3. Backfill — mẫu one-off an toàn cho production

Backfill (điền dữ liệu cho cột mới, sửa hàng loạt) là one-off nguy hiểm nhất. Mẫu an toàn tối thiểu:

```go
func runBackfillEmails(ctx context.Context, cfg *config.Config, log *slog.Logger) error {
	pool := mustConnect(ctx, cfg.DatabaseURL)
	defer pool.Close()

	dryRun := os.Getenv("DRY_RUN") != "false" // ✅ mặc định DRY RUN — phá hoại phải opt-in
	const batchSize = 500                      // ✅ theo lô nhỏ — không khóa bảng, không ăn hết IO

	var total int
	for {
		// ✅ idempotent: điều kiện WHERE tự loại hàng đã xử lý —
		// job chết giữa chừng (F9!) chạy lại là tiếp tục, không hỏng
		rows, err := selectBatch(ctx, pool, batchSize) // WHERE email_normalized IS NULL
		if err != nil {
			return err
		}
		if len(rows) == 0 {
			break
		}
		if dryRun {
			log.Info("DRY RUN: would update", "count", len(rows), "sample", rows[0].ID)
		} else {
			if err := updateBatch(ctx, pool, rows); err != nil {
				return err
			}
		}
		total += len(rows)
		log.Info("progress", "processed", total) // ✅ tiến độ ra stdout (F11)
		time.Sleep(100 * time.Millisecond)       // ✅ nhường IO cho traffic thật
	}
	log.Info("backfill done", "total", total, "dry_run", dryRun)
	return nil
}
```

### 4.4. Kubernetes — Job cho one-off, CronJob thay crontab

```yaml
# Job: chạy backfill MỘT lần — cùng image, cùng secret với app
apiVersion: batch/v1
kind: Job
metadata:
  name: backfill-emails-20260708        # tên có ngày: dấu vết tự nhiên
spec:
  backoffLimit: 0                       # backfill fail → người xem xét, không tự retry mù
  ttlSecondsAfterFinished: 86400
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: backfill
          image: registry.example.com/myapp@sha256:...   # CHÍNH image đang chạy prod
          args: ["backfill-emails"]
          env: [{ name: DRY_RUN, value: "false" }]
          envFrom: [{ secretRef: { name: myapp-secrets } }]  # CÙNG config
          resources:
            requests: { cpu: 100m, memory: 128Mi }
            limits: { memory: 256Mi }
---
# CronJob: thay thế crontab-trên-server-số-1 (giải bài toán F6 để lại)
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cleanup-expired
spec:
  schedule: "0 3 * * *"
  concurrencyPolicy: Forbid             # lần trước chưa xong thì không chạy đè
  startingDeadlineSeconds: 3600
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: cleanup
              image: registry.example.com/myapp@sha256:...
              args: ["cleanup-expired"]
              envFrom: [{ secretRef: { name: myapp-secrets } }]
```

Còn nhu cầu "gõ lệnh thăm dò" (REPL của thế giới Rails/Django)? Go không có REPL, và thay thế đúng là: viết lệnh thăm dò thành subcommand read-only (`app inspect-user <id>`), hoặc truy vấn qua công cụ DB có kiểm soát quyền (read-only replica + SQL client có audit). `kubectl exec` vào pod app chỉ dành cho chẩn đoán khẩn cấp — và distroless image (chương 4) cố tình làm nó bất khả thi; dùng `kubectl debug` với ephemeral container khi thật sự cần.

## 5. Trade-off

**Kỷ luật vs tốc độ lúc cứu hỏa.** Đang sự cố, viết subcommand + build + deploy Job chậm hơn mở psql gõ UPDATE. Trung thực mà nói: đôi khi bạn *sẽ* gõ SQL tay lúc cháy nhà. Kỷ luật thực tế của tổ chức trưởng thành: (1) đường nóng có kiểm soát — bastion có audit log, transaction bắt buộc, hai người một màn hình; (2) *sau* sự cố, thao tác tay được chuyển thành subcommand có test — lần cháy sau đã có sẵn bình cứu hỏa; (3) pipeline đủ nhanh (build 5 phút) để con đường kỷ luật không quá đắt.

**Job trong release pipeline vs Job thủ công.** Migration nên tự động trong pipeline (mọi release đều chạy, idempotent, `ErrNoChange` là bình thường). Backfill/sửa dữ liệu nên **thủ công có chủ đích** (người bấm nút, có dry-run trước) — tự động hóa thứ chạy đúng một lần không đáng rủi ro chạy nhầm.

**Một binary đa vai vs binary riêng cho tool.** Một binary: đảm bảo "cùng release" tuyệt đối, image gọn (một cái). Giá: binary to hơn chút, và cẩn thận để lệnh phá hoại không bị gọi nhầm (subcommand rõ ràng, xác nhận, DRY_RUN mặc định). Với đa số team, một binary thắng.

## 6. Best Practices

- Mọi tác vụ quản trị là subcommand trong repo — review, test (integration test cho migration/backfill trên testcontainers!), có lịch sử.
- Chạy từ **đúng image của release đang chạy** (pin digest); cùng env/secret với app qua `envFrom`.
- One-off nguy hiểm: DRY_RUN mặc định, batch nhỏ, idempotent, có tiến độ, có giới hạn tốc độ.
- Migration: tự động trong pipeline như release step (F5), forward-only + expand–contract; backfill: thủ công có chủ đích.
- CronJob: `concurrencyPolicy: Forbid`, job idempotent (K8s CronJob có thể chạy trùng/bỏ lỡ — thiết kế để chịu được).
- Exit code nghiêm túc (0/1) — hệ thống alert dựa vào Job status.
- Quyền chạy Job tách khỏi quyền sửa Deployment (RBAC) — ai được "bấm nút" là quyết định có chủ đích.

## 7. Anti-patterns

- **SSH + psql gõ UPDATE tay trên production** — không dấu vết, không lặp lại được, không lối thoát khi gõ nhầm; anti-pattern mẹ của factor này.
- **Script vận hành bằng ngôn ngữ khác app, sống ngoài repo** — logic trôi dạt khỏi app; "đúng hôm qua" không có nghĩa "đúng hôm nay".
- **Script hardcode DSN production** — vừa vi phạm F3 vừa là quả bom: ai đó chạy nhầm từ máy dev là dữ liệu thật lãnh đủ.
- **Migration trong `main()` app** — N pod đua migrate; crash-loop toàn fleet khi migration fail (F5 đã mổ xẻ).
- **Crontab trên một server cưng** — chết cùng server, nhân bản khi scale, vô hình với codebase.
- **`kubectl exec` vào pod chạy việc quản trị định kỳ** — dùng nền tảng (Job/CronJob), không dùng đường chẩn đoán làm đường vận hành.
- **Backfill một transaction ôm 10 triệu hàng** — khóa bảng, đầy WAL, replica lag; luôn theo lô.

## 8. Khi nào KHÔNG cần áp dụng đầy đủ

- **Hệ nhỏ một server**: script bash cạnh app + cron trên server chấp nhận được — miễn script **nằm trong repo** và không hardcode secret. Hai điều đó không bao giờ có lý do bỏ.
- **Thao tác thăm dò read-only**: query replica qua BI/SQL tool là bình thường, không cần nghi thức Job — ranh giới là *ghi*.
- **Tác vụ của DBA thuần hạ tầng** (VACUUM, reindex, tuning): thuộc tầng vận hành database, theo quy trình của tầng đó — không phải mọi thứ chạm DB đều là "admin process của app".

---

## Tóm tắt

- Tác vụ quản trị là **công dân hạng nhất**: cùng codebase, cùng image/release, cùng config, log ra stdout — chỉ khác là chạy một lần rồi thoát.
- Hình thái Go: **một binary, nhiều subcommand** (`server`/`worker`/`migrate`/`backfill`); migration embed trong binary, chạy như release step; one-off qua K8s Job; định kỳ qua CronJob thay crontab.
- One-off nguy hiểm cần nghi thức: DRY_RUN mặc định, batch nhỏ, idempotent, dấu vết đầy đủ.
- Thao tác tay lúc cứu hỏa là thực tế — tổ chức trưởng thành biến mỗi lần tay thành một subcommand cho lần sau.

**Hết Phần 2.** Mười hai factor đã đi qua — và bạn có thể thấy chúng đan vào nhau chặt đến mức chương nào cũng phải tham chiếu chương khác. Phần 3 chuyển sang thực chiến kỹ thuật: [Golang patterns cho ứng dụng 12-Factor](/series/twelve-factor/18-golang-patterns/).
