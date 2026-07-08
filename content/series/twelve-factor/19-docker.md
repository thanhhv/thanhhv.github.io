+++
title = "Chương 19 — Docker: đóng gói ứng dụng 12-Factor"
date = "2026-07-10T20:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> **Phần 3 – Engineering** | Chương trước: [Golang Patterns](/series/twelve-factor/18-golang-patterns/) | Chương sau: [Kubernetes](/series/twelve-factor/20-kubernetes/)

Container không phải một factor — nó là **công nghệ hiện thực hóa nhiều factor cùng lúc**: F2 (isolate dependencies), F5 (build artifact bất biến), F10 (cùng image mọi môi trường), và là đơn vị của F6/F8/F9 (process/scale/dispose). Chương này đi từ bản chất container đến Dockerfile production tối ưu.

---

## 1. Bản chất: container là gì (và không phải là gì)

Container **không phải VM nhẹ**. VM ảo hóa *phần cứng* (mỗi VM một kernel riêng); container ảo hóa *hệ điều hành* — mọi container trên một máy **dùng chung kernel host**, được cô lập bằng hai cơ chế Linux:

- **Namespaces** — cô lập *tầm nhìn*: PID (chỉ thấy process của mình — app của bạn là PID 1), network (interface/port riêng), mount (filesystem riêng), UTS/IPC/user.
- **Cgroups** — cô lập *tài nguyên*: giới hạn CPU, memory, IO cho nhóm process (đây chính là thứ đứng sau `resources.limits` của K8s, và là lý do phải quan tâm `GOMAXPROCS` — chương 13).

Hệ quả so sánh:

| | VM | Container |
|---|---|---|
| Khởi động | phút | **mili giây–giây** (chỉ là fork process) → F9 |
| Overhead | GB (OS riêng mỗi VM) | MB (chung kernel) |
| Cô lập | mạnh (hypervisor) | yếu hơn (chung kernel — escape khó nhưng không phải bất khả) |
| Mật độ | chục/host | trăm–nghìn/host |

**Image** = filesystem đóng băng + metadata (entrypoint, env, port), xếp thành **layer** bất biến, address theo content hash — vì thế image là hiện thân hoàn hảo của immutable artifact (chương 4): cùng digest là cùng bit, mọi nơi, mọi lúc. **Container** = một process được chạy từ image đó với namespace + cgroup bọc quanh. Nhớ đẳng thức "container ≈ process" thì mọi nguyên tắc 12-Factor về process áp thẳng vào container.

## 2. Dockerfile production cho Go — bản đầy đủ có chú giải

```dockerfile
# syntax=docker/dockerfile:1

########## Stage 1: BUILD — to, bẩn, nhiều tool; sẽ bị vứt ##########
FROM golang:1.24.1-alpine3.21 AS build
# Ghim version cụ thể (F2). Cân nhắc pin digest: golang@sha256:...

WORKDIR /src

# Layer caching: go.mod/go.sum đổi ít → tách riêng để cache bước download.
# Cache mount: giữ module cache & build cache giữa các lần build CI
COPY go.mod go.sum ./
RUN --mount=type=cache,target=/go/pkg/mod \
    go mod download && go mod verify

COPY . .
ARG VERSION=dev
ARG COMMIT_SHA=unknown
RUN --mount=type=cache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache/go-build \
    CGO_ENABLED=0 GOOS=linux go build \
      -trimpath \
      -ldflags="-s -w \
        -X myapp/internal/buildinfo.Version=${VERSION} \
        -X myapp/internal/buildinfo.CommitSHA=${COMMIT_SHA}" \
      -o /out/app ./cmd/app
# CGO_ENABLED=0 → binary tĩnh hoàn toàn: không cần libc → chạy được trên scratch/distroless
# -trimpath     → bỏ đường dẫn máy build khỏi binary (reproducible hơn)
# -s -w         → bỏ symbol table + DWARF: binary nhỏ hơn ~30%

########## Stage 2: RUNTIME — nhỏ, sạch, bất biến ##########
FROM gcr.io/distroless/static-debian12:nonroot
# distroless static: ~2MB, KHÔNG shell, KHÔNG package manager, user nonroot sẵn.
# Không có gì để attacker dùng, không có gì để "sửa tay" (chương 4).

COPY --from=build /out/app /app

USER nonroot:nonroot        # không bao giờ chạy root trong container
EXPOSE 8080
ENV PORT=8080

ENTRYPOINT ["/app"]         # EXEC FORM — bắt buộc!
CMD ["server"]              # subcommand mặc định; Job/CronJob ghi đè args (F12)
```

Kết quả thực tế: image ~15–25MB (so với ~1GB nếu FROM golang + không multi-stage), pull trong ~1 giây, bề mặt tấn công gần bằng không, scan CVE gần như luôn sạch.

### Những quyết định quan trọng và lý do

**Multi-stage build** tách đôi thế giới: môi trường *build* (cần compiler, git, cache — to và bẩn) và môi trường *run* (chỉ cần binary — nhỏ và sạch). Đây chính là F5 (build ≠ run) khắc vào tầng đóng gói. Không multi-stage = ship cả compiler và source code lên production.

**ENTRYPOINT exec form `["/app"]` — không phải shell form.** Shell form (`ENTRYPOINT ./app`) chạy `/bin/sh -c ./app`: shell là PID 1, app là con — và **shell không chuyển tiếp SIGTERM** → app không bao giờ biết mình sắp bị giết → toàn bộ graceful shutdown (F9, chương 14) chết từ trong trứng. Lỗi một dòng, hậu quả cả hệ thống.

**Chọn base image runtime:**

| Base | Cỡ | Khi nào |
|---|---|---|
| `scratch` | 0 | Tối giản tuyệt đối; nhưng thiếu CA certs, tzdata, /tmp — tự lo hết |
| `distroless/static` | ~2MB | **Mặc định đúng cho Go**: có CA certs + tzdata + user nonroot, không shell |
| `alpine` | ~8MB | Khi cần shell/apk để debug hoặc cài binary ngoài (ffmpeg...) — chấp nhận tăng bề mặt |
| `debian-slim` | ~75MB | Khi cần glibc/công cụ hệ tiêu chuẩn (cgo phức tạp) |

(Cần debug container distroless đang chạy? `kubectl debug` gắn ephemeral container có sẵn tool — không cần nhét shell vào image production.)

## 3. Một container = một process (một mối bận tâm)

Quy tắc bị vi phạm nhiều nhất bởi người mới chuyển từ VM: nhét app + nginx + cron + supervisor vào một container "cho giống server cũ". Vì sao sai — theo từng factor:

- Nền tảng quản lý vòng đời **container**, không nhìn thấy các process con: worker chết trong khi web sống → container "healthy" giả tạo, không ai restart worker (phá F9, self-healing).
- Không scale độc lập từng phần (phá F8), không đo lường/giới hạn tài nguyên từng phần.
- Log các process trộn một dòng stdout (phá F11), signal chỉ đến PID 1 — supervisor nội bộ tái phát minh (tệ hơn) orchestrator.

Mỗi process type (chương 13) → một image entry riêng (hoặc một image, nhiều command) → một Deployment riêng. Việc "ghép các container cần đứng cạnh nhau" đã có khái niệm **pod** của K8s (sidecar) — đúng tầng, đúng công cụ.

## 4. docker-compose — môi trường dev hoàn chỉnh

Đã dựng dần qua các chương; bản hợp nhất nằm trong case study (chương 24). Điểm đáng nhấn thêm: **compose watch** cho vòng lặp dev nhanh mà vẫn giữ tools parity:

```yaml
# docker-compose.yml (trích) — hot rebuild khi code đổi, vẫn chạy trong container
services:
  app:
    build: { context: ., target: build }   # dev dùng stage build (có toolchain)
    command: go run ./cmd/app server
    develop:
      watch:
        - action: sync+restart
          path: .
          target: /src
```

## 5. Bảo mật image — checklist tối thiểu

- **Không chạy root** (`USER nonroot`); trên K8s khóa thêm bằng `runAsNonRoot: true`, `allowPrivilegeEscalation: false`, drop mọi capability.
- **readOnlyRootFilesystem: true** — đã là bạn cũ từ chương 4.
- **Không secret trong image** — không `COPY .env`, không ARG chứa token (ARG lưu trong history!); secret đến từ môi trường lúc run (F3). `docker history <image>` phải sạch.
- **`.dockerignore`**: loại `.git`, `.env*`, tài liệu, test data — vừa nhanh vừa tránh rò rỉ vào build context.
- **Scan trong CI**: trivy/grype cho CVE; hadolint cho Dockerfile lint.
- **Pin digest base image** + Renovate tự mở PR khi base có bản vá (F2).

## 6. Trade-off

**Image nhỏ nhất vs khả năng vận hành.** Distroless không shell là bảo mật tuyệt vời nhưng đêm sự cố không `exec` vào xem được — phải có văn hóa `kubectl debug` + observability tốt (chương 22) *trước khi* khóa cửa. Team chưa sẵn sàng có thể dùng alpine một thời gian — đánh đổi có ý thức.

**Cache build nhanh vs reproducible tuyệt đối.** Cache mount tăng tốc CI nhiều lần nhưng đưa "trạng thái máy build" quay lại một phần. Với Go + go.sum, rủi ro thực tế rất thấp (module bất biến theo checksum); build release chính thức có thể chạy no-cache định kỳ để kiểm chứng.

**Một image đa vai (server/worker/migrate) vs image riêng từng vai.** Một image: đảm bảo cùng release tuyệt đối (F5, F12), đơn giản registry. Image riêng: nhỏ hơn chút, tách bề mặt. Đa số team: một image đa vai thắng — khác biệt cỡ vài MB không đáng đổi lấy rủi ro lệch version giữa các vai.

## 7. Anti-patterns

- **Shell form ENTRYPOINT** — giết graceful shutdown âm thầm (đã mổ xẻ).
- **Nhiều process một container** (supervisor, cron trong container).
- **`FROM golang` làm runtime** (không multi-stage) — 1GB image chứa compiler + source trên production.
- **Chạy root** + filesystem ghi được — container escape dễ hơn, vi phạm stateless dễ hơn.
- **Secret nướng vào image** — image được pull đi khắp nơi, secret đi theo.
- **`COPY . .` trước `go mod download`** — mọi thay đổi code làm hỏng cache tải dependency; đảo thứ tự là quy tắc cache số một.
- **`:latest` và tag mutable** — đã thuộc lòng từ F5.
- **VOLUME cho dữ liệu app "cho tiện"** — cửa sau đưa state quay lại container (phá F6); volume chỉ cho cache có chủ đích hoặc hệ stateful thật.

## 8. Khi nào KHÔNG dùng container

- **CLI tool phân phối end-user**: Go static binary tự nó đã là "container một file" — ship binary, đừng bắt user cài Docker.
- **Môi trường cấm/không có runtime** (một số edge, embedded): static binary + systemd unit (cũng khai báo được restart, limit — một orchestrator mini).
- **Workload cực nhạy hiệu năng syscall/network** hiếm hoi: overhead container rất nhỏ nhưng khác không; đo trước khi kết luận.
- Serverless managed (Lambda zip) — nền tảng đã lo isolate; container là *một* lựa chọn đóng gói, không phải bắt buộc.

---

## Tóm tắt

- Container = **process + namespace (tầm nhìn) + cgroup (tài nguyên)**, không phải VM nhẹ; image = filesystem bất biến theo content hash — hiện thân của immutable artifact.
- Dockerfile Go chuẩn: multi-stage, ghim version, cache mount, `CGO_ENABLED=0`, distroless nonroot, **ENTRYPOINT exec form** — mỗi quyết định gắn với một factor cụ thể.
- Một container một mối bận tâm; ghép cạnh nhau là việc của pod, không phải của supervisor trong container.
- Bảo mật tối thiểu: nonroot, read-only FS, không secret trong image, scan CI, pin digest.
