+++
title = "Chương 7 — Factor 2: Dependencies"
date = "2026-07-10T08:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> *"Explicitly declare and isolate dependencies"* — Khai báo tường minh và cô lập dependency.
>
> **Phần 2 – The Twelve Factors** | Nhóm: **Reproducible** | Chương trước: [Codebase](/series/twelve-factor/06-factor-01-codebase/) | Chương sau: [Config](/series/twelve-factor/08-factor-03-config/)

---

## 1. Problem Statement

Câu hỏi factor này trả lời: **"Ứng dụng cần chính xác những gì để chạy?"** — và câu trả lời phải nằm trong codebase, không nằm trong trí nhớ của ai đó hay trạng thái của một máy nào đó.

Các sự cố kinh điển khi vi phạm:

- App chạy trên máy dev vì máy dev "tình cờ" có sẵn ImageMagick / một version OpenSSL cụ thể / `git` trong PATH. Máy production không có → crash lúc runtime, đúng tính năng đó, đúng lúc khách dùng.
- Hai dev build cùng một commit ra hai binary hành vi khác nhau, vì mỗi máy có version thư viện khác nhau trong GOPATH (thời tiền Go modules — chính là lý do Go modules ra đời).
- Library bị tác giả xóa khỏi internet (sự kiện `left-pad` 2016 làm sập một góc hệ sinh thái npm) → không build lại được app của chính mình.
- Nâng OS của server → version Python hệ thống đổi từ 3.8 lên 3.11 → app khác (không liên quan gì đến việc nâng OS) chết theo.

Bản chất chung: **dependency ngầm (implicit dependency) là environment coupling** — đúng căn bệnh gốc đã chẩn đoán ở chương 1, thể hiện ở tầng thư viện và công cụ hệ thống.

## 2. Tại sao nguyên lý này tồn tại

- **Deployment**: máy mới (autoscaling sinh ra, dev mới onboard) phải dựng được app **chỉ từ codebase** — không có anh Tuấn nào SSH vào cài tay từng thứ.
- **Operational**: vá lỗ hổng bảo mật đòi hỏi biết *chính xác* mình đang dùng thư viện nào version nào (câu hỏi "chúng ta có dính Log4Shell không?" phải trả lời được trong 5 phút, không phải 5 ngày).
- **Business**: supply chain attack là rủi ro thật (SolarWinds, xz-utils 2024); kiểm soát dependency là kiểm soát bề mặt tấn công.
- **Scalability**: 100 instance phải chạy **đúng cùng một bộ** dependency — điều chỉ đảm bảo được khi bộ đó được khai báo và đóng gói, không phải "cài trên máy".

## 3. Bản chất

Factor này có hai vế độc lập và đều bắt buộc:

**Declare (khai báo tường minh)**: toàn bộ dependency được liệt kê trong một manifest sống trong codebase, **kèm version chính xác** — với Go là `go.mod` + `go.sum`. Không dependency nào được phép "mặc định là có".

**Isolate (cô lập)**: lúc chạy, app không được lôi dependency từ môi trường xung quanh (system packages, GOPATH global, tool trong PATH). Ranh giới cô lập hiện đại là **container image**: mọi thứ app cần nằm trong image, những gì ngoài image coi như không tồn tại.

Vì sao phải cả hai? Chỉ declare không isolate: app khai báo cần `libfoo 1.2` nhưng runtime vẫn nhặt `libfoo 1.5` cài sẵn trên máy → khai báo thành trang trí. Chỉ isolate không declare: container "hoạt động" nhưng được dựng thủ công không ai biết trong đó có gì → không tái tạo được, không audit được.

**Điều gì xảy ra nếu vi phạm?** Reproducibility gãy ngay tại gốc: cùng commit không còn sinh ra cùng hành vi → Dev/Prod Parity (F10) trở nên bất khả thi về nguyên tắc, CI/CD (F5) mất ý nghĩa vì "build hôm nay" khác "build hôm qua" dù code không đổi.

## 4. Cách áp dụng với Go

### 4.1. Go modules — declare đúng mức

Go có lợi thế lớn ở factor này: modules là bắt buộc, lockfile (`go.sum`) là mặc định, và binary tĩnh loại bỏ phần lớn dependency hệ thống.

```
go.mod  → khai báo module cần gì (version tối thiểu, theo semver)
go.sum  → checksum MỌI module trong dependency graph
          → ai đó thay nội dung một version đã phát hành (supply chain attack)
            là build FAIL ngay. Luôn commit go.sum.
```

```bash
go mod tidy                        # đồng bộ go.mod với import thực tế
go mod verify                      # kiểm tra module cache khớp go.sum
go list -m all                     # liệt kê toàn bộ dependency graph
govulncheck ./...                  # quét lỗ hổng — chỉ báo lỗ hổng ở code path
                                   # bạn THỰC SỰ gọi tới (ít false positive)
```

Điểm hay ít người để ý: Go toolchain còn khai báo được **chính version compiler**:

```go
// go.mod
module github.com/acme/myapp

go 1.24.0          // version ngôn ngữ tối thiểu
toolchain go1.24.1 // ép đúng toolchain — cùng go.mod, mọi máy build bằng cùng compiler
```

### 4.2. Dependency không phải thư viện: tool và binary hệ thống

Lỗ hổng phổ biến nhất ở team Go không nằm ở `go.mod` mà ở **dependency ngoài Go**: app gọi `exec.Command("ffmpeg", ...)`, migration cần `psql`, build cần `protoc`. Chúng cũng phải được declare + isolate:

```go
// ❌ Dependency ngầm: giả định ffmpeg có trên máy — không khai báo ở đâu cả
cmd := exec.Command("ffmpeg", "-i", in, out)

// ✅ Lựa chọn 1 (tốt nhất): dùng thư viện Go thuần nếu có, biến nó thành
//    dependency được go.mod quản lý.
// ✅ Lựa chọn 2: nếu buộc phải dùng binary ngoài — cài nó TRONG Dockerfile
//    với version ghim cụ thể. Image trở thành manifest của nó.
```

Tool phục vụ development (codegen, lint) khai báo bằng cơ chế `tool` directive (Go ≥1.24) để cả team dùng đúng một version:

```bash
go get -tool github.com/sqlc-dev/sqlc/cmd/sqlc@v1.27.0
go tool sqlc generate   # mọi máy chạy đúng v1.27.0, ghi trong go.mod
```

### 4.3. Dockerfile — isolate trọn vẹn

```dockerfile
# ---- Stage 1: build — môi trường build cũng được ghim version ----
FROM golang:1.24.1-alpine3.21 AS build
#           ^^^^^^^^^^^^^^^^^ ghim cụ thể, không dùng golang:latest
WORKDIR /src

# Copy manifest trước, tải deps trước → Docker cache layer này,
# đổi code không phải tải lại toàn bộ dependency
COPY go.mod go.sum ./
RUN go mod download && go mod verify

COPY . .
RUN CGO_ENABLED=0 go build -trimpath -ldflags="-s -w" -o /out/app ./cmd/server

# ---- Stage 2: runtime — chỉ chứa đúng những gì cần chạy ----
FROM gcr.io/distroless/static-debian12:nonroot
# distroless static: không shell, không libc, không package manager.
# Binary Go tĩnh (CGO_ENABLED=0) không cần gì từ OS → bề mặt tấn công tối thiểu,
# và mọi "dependency lén" sẽ lộ ngay vì không có gì để dựa vào.
COPY --from=build /out/app /app
USER nonroot:nonroot
ENTRYPOINT ["/app"]
```

Nếu app buộc cần binary hệ thống (ví dụ ffmpeg), stage runtime chuyển sang base image có package manager và ghim version:

```dockerfile
FROM alpine:3.21
RUN apk add --no-cache ffmpeg=6.1.2-r1   # declare version ngay trong image
```

### 4.4. CI — cưỡng chế kỷ luật

```yaml
# .github/workflows/ci.yml (trích)
- run: go mod tidy && git diff --exit-code go.mod go.sum
  # go.mod/go.sum lệch với code → fail: cấm dependency "chưa khai báo"
- run: go mod verify
- run: govulncheck ./...
```

## 5. Trade-off

**Ghim chặt version (reproducibility) vs cập nhật nhanh (security patch).** Ghim chặt mọi thứ nghĩa là bạn *không tự động* nhận bản vá bảo mật. Đáp án của ngành không phải chọn một trong hai mà là: **ghim chặt + tự động hóa việc nâng** — Dependabot/Renovate mở PR nâng version, CI chạy test, con người (hoặc auto-merge với minor patch) quyết định. Được cả hai: mọi thay đổi dependency đều là một commit có kiểm chứng.

**Vendor (`go mod vendor`) vs module proxy.** Vendoring copy toàn bộ source dependency vào repo: build không cần mạng, miễn nhiễm chuyện package bị xóa; giá là repo phình to và diff PR ồn ào. Với Go, `GOPROXY=proxy.golang.org` (mặc định) đã cache bất biến các module công khai — đủ cho đa số. Vendor xứng đáng khi: môi trường build air-gapped, hoặc yêu cầu compliance giữ toàn bộ source.

**Ít dependency vs tự viết lại.** Mỗi dependency là code của người lạ chạy trong process của bạn với đầy đủ quyền. Văn hóa Go ("a little copying is better than a little dependency") có lý do bảo mật lẫn bảo trì. Ngưỡng cân nhắc: kéo cả một framework để dùng một hàm là mất cân đối; tự viết lại crypto/parser phức tạp là liều lĩnh theo chiều ngược lại.

## 6. Best Practices

- Commit `go.mod` + `go.sum`; CI chặn khi `go mod tidy` tạo diff.
- Ghim version cụ thể cho base image; tốt hơn nữa: pin theo digest `sha256`.
- `govulncheck` trong CI (nhẹ, ít false positive); cân nhắc SBOM (`syft`, `docker sbom`) cho yêu cầu compliance — trả lời "có dính CVE X không" trong một câu lệnh.
- Multi-stage build; runtime image tối thiểu (distroless/alpine); `CGO_ENABLED=0` trừ khi thật sự cần cgo.
- Dependabot/Renovate bật cho cả Go modules, Dockerfile base image và GitHub Actions.
- Binary hệ thống mà app gọi (`exec.Command`) phải được cài trong Dockerfile với version ghim — coi Dockerfile là manifest thứ hai.

## 7. Anti-patterns

- **"Cài trên server là xong"** — dependency sống ngoài codebase, máy mới không tự dựng được; đây chính là snowflake server phiên bản thư viện.
- **Không commit `go.sum`** (hoặc xóa cho "đỡ conflict") — mất kiểm chứng checksum, mở cửa supply chain attack và build không tái tạo.
- **`FROM golang:latest` / `FROM alpine:latest`** — image build hôm nay khác hôm qua; "build lại đúng bản cũ" trở thành bất khả thi.
- **Giả định tool tồn tại trong PATH lúc runtime** — crash chỉ xuất hiện ở môi trường thiếu tool, thường là production.
- **Copy source thư viện vào repo rồi sửa vài dòng** (fork ngầm) — không nhận được update nữa, và không ai biết bản này đã bị sửa. Nếu cần sửa: fork công khai + `replace` directive, có dấu vết.
- **Dùng chung một base image "cty tự build" chứa sẵn 50 tool** cho mọi service — quay lại mô hình "môi trường cài sẵn", chỉ là trong container.

## 8. Khi nào KHÔNG cần áp dụng đầy đủ

Cũng như Factor 1, chi phí tuân thủ gần bằng không với Go (modules là mặc định), nên hầu như không có lý do bỏ qua vế *declare*. Vế *isolate* có ngoại lệ hợp lý: script vận hành nhỏ chạy trên máy đã được quản lý bằng config management (Ansible ghim version tool — bản thân nó là một dạng declare); môi trường embedded nơi container không khả thi (khi đó Nix hoặc static binary là dạng isolate thay thế); CLI tool phân phối cho user cuối — Go static binary tự nó đã là "isolate hoàn hảo" không cần container.

---

## Tóm tắt

- Trả lời câu hỏi "app cần gì để chạy" **bằng code, không bằng trí nhớ**: `go.mod`/`go.sum` cho thư viện, Dockerfile cho tool hệ thống và runtime.
- Hai vế bắt buộc: **declare** (manifest + version + checksum) và **isolate** (container là ranh giới; những gì ngoài image không tồn tại).
- Dependency nguy hiểm nhất là dependency **ngầm** — binary trong PATH, package cài sẵn trên OS, "máy tôi có sẵn".
- Ghim chặt version, rồi tự động hóa việc nâng (Renovate + CI) — reproducibility và security không phải chọn một.
