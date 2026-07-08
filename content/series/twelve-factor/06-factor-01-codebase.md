+++
title = "Chương 6 — Factor 1: Codebase"
date = "2026-07-10T07:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> *"One codebase tracked in revision control, many deploys"* — Một codebase trong version control, nhiều deploy.
>
> **Phần 2 – The Twelve Factors** | Nhóm: **Reproducible** | Chương sau: [Factor 2: Dependencies](/series/twelve-factor/07-factor-02-dependencies/)

---

## 1. Problem Statement

Câu hỏi mà factor này trả lời nghe có vẻ tầm thường: **"Code nào đang chạy trên production?"** — cho đến khi bạn gặp các tình huống sau (tất cả đều là chuyện thật ở các doanh nghiệp):

- Bản production hiện tại được build từ folder `app_final_v2_fix_REAL/` trên máy một dev đã nghỉ việc.
- Có 3 bản fork của cùng một hệ thống cho 3 khách hàng, mỗi bản được sửa riêng, giờ muốn fix một bug bảo mật phải fix 3 nơi — và không nơi nào giống nơi nào nữa.
- Hotfix được sửa trực tiếp trên server production, không commit; lần deploy sau **ghi đè mất hotfix**, sự cố cũ quay lại lúc nửa đêm.
- Staging chạy từ branch `develop`, production chạy từ `master`, hai branch lệch nhau 4 tháng — mọi kết quả test trên staging vô nghĩa.

Nếu không có nguyên tắc codebase: **mất khả năng truy vết** (không trả lời được "đang chạy gì"), **mất khả năng tái tạo** (không dựng lại được bản đang chạy), và **phân mảnh sự thật** (nhiều biến thể code cùng tồn tại không kiểm soát).

## 2. Tại sao nguyên lý này tồn tại

- **Business**: audit và compliance đòi hỏi trả lời "phiên bản nào chạy lúc xảy ra sự cố X" trong vài phút, không phải vài ngày khảo cổ.
- **Operational**: mọi quy trình tự động (CI/CD, rollback, diff giữa hai bản deploy) đều cần một nguồn sự thật duy nhất để làm gốc tọa độ.
- **Deployment**: "deploy" chỉ có thể tự động hóa khi nó là hàm của một tham chiếu bất biến (commit SHA), không phải của "thư mục trên máy anh Nam".
- **Scalability (về mặt tổ chức)**: nhiều team làm chung một hệ thống chỉ khả thi khi có một lịch sử chung để hợp nhất thay đổi.

## 3. Bản chất

Phát biểu đầy đủ của factor gồm hai vế, mỗi vế chặn một loại bệnh:

**Vế 1 — Một app ↔ một codebase.** Quan hệ 1-1:

- Nhiều codebase cho một app → không phải app, đó là distributed system chưa được thừa nhận; hoặc tệ hơn: các bản fork-per-customer, nơi mọi bugfix phải nhân bản thủ công.
- Nhiều app dùng chung một codebase theo kiểu copy-paste hoặc cùng repo không ranh giới → thay đổi cho app A làm gãy app B. Code dùng chung phải được tách thành **library có version riêng** và tiêu thụ qua dependency manager (Factor 2). *(Monorepo được bàn ở mục Trade-off — nó không vi phạm nguyên tắc này nếu làm đúng.)*

**Vế 2 — Một codebase → nhiều deploy.** Cùng một codebase chạy ở dev, staging, production, có thể ở các commit khác nhau tại một thời điểm, nhưng luôn là **các điểm trên cùng một dòng lịch sử**. Điều khác nhau giữa các deploy chỉ được phép là **config** (Factor 3) — không bao giờ là code.

```
        MỘT CODEBASE (Git)
   ────●────●────●────●────●───▶ thời gian (commit)
        \         \         \
         \         \         └── production  (commit e, config prod)
          \         └──────────── staging     (commit e, config staging)
           └───────────────────── dev Hùng    (commit c + WIP, config local)

   Hợp lệ:   các deploy ở commit khác nhau trên CÙNG lịch sử
   Vi phạm:  deploy chạy code KHÔNG tồn tại trong lịch sử nào
             (sửa tay trên server, fork riêng cho khách...)
```

**Điều gì xảy ra nếu vi phạm?** Mọi tầng phía trên sụp theo: CI/CD không có gì để build (vi phạm F5), Dev/Prod Parity không định nghĩa được vì không có "cùng code" để so (F10), rollback thành trò may rủi. Codebase là factor rẻ nhất và nền tảng nhất — nó gần như là *tiên đề* để các factor khác tồn tại.

## 4. Cách áp dụng

### 4.1. Cấu trúc một repo Go chuẩn mực

```
myapp/
├── go.mod                  # định danh module + dependencies (F2)
├── go.sum
├── cmd/
│   ├── server/main.go      # entrypoint API server
│   └── worker/main.go      # entrypoint background worker
│                           # → NHIỀU process type, MỘT codebase (hợp lệ,
│                           #   liên quan F8; khác với nhiều APP một repo)
├── internal/               # code riêng của app — compiler Go CHẶN import từ ngoài
│   ├── config/
│   ├── handler/
│   ├── service/
│   └── repository/
├── migrations/             # schema migration sống cùng code (F12)
├── deploy/
│   ├── Dockerfile
│   └── k8s/
└── .github/workflows/ci.yml
```

Điểm đáng chú ý của Go: thư mục `internal/` được **compiler cưỡng chế** — package bên ngoài module không thể import. Đây là công cụ giữ ranh giới codebase tốt hơn mọi quy ước.

### 4.2. Nhúng định danh build vào binary — nối runtime về codebase

Nguyên tắc codebase chỉ trọn vẹn khi từ một process đang chạy, bạn truy ngược được về commit sinh ra nó:

```go
// internal/buildinfo/buildinfo.go
package buildinfo

// Được gán giá trị lúc build qua -ldflags (xem Dockerfile bên dưới)
var (
	Version   = "dev"
	CommitSHA = "unknown"
	BuildTime = "unknown"
)
```

```go
// cmd/server/main.go (trích)
http.HandleFunc("/version", func(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")
	fmt.Fprintf(w, `{"version":%q,"commit":%q,"build_time":%q}`,
		buildinfo.Version, buildinfo.CommitSHA, buildinfo.BuildTime)
})
```

```dockerfile
# deploy/Dockerfile (trích) — truyền định danh Git vào binary
FROM golang:1.24-alpine AS build
ARG COMMIT_SHA=unknown
ARG VERSION=dev
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -trimpath \
    -ldflags="-s -w \
      -X myapp/internal/buildinfo.CommitSHA=${COMMIT_SHA} \
      -X myapp/internal/buildinfo.Version=${VERSION}" \
    -o /out/server ./cmd/server

FROM gcr.io/distroless/static-debian12:nonroot
COPY --from=build /out/server /server
ENTRYPOINT ["/server"]
```

```yaml
# .github/workflows/ci.yml (trích) — Git SHA chảy xuyên suốt pipeline
- name: Build & push image
  run: |
    docker build \
      --build-arg COMMIT_SHA=${{ github.sha }} \
      --build-arg VERSION=${{ github.ref_name }} \
      -t registry.example.com/myapp:${{ github.sha }} .
    docker push registry.example.com/myapp:${{ github.sha }}
```

Kết quả: chuỗi truy vết khép kín `pod đang chạy → image tag = commit SHA → git show <SHA>`. Câu hỏi "production đang chạy code nào?" được trả lời bằng một lệnh `curl /version`.

### 4.3. Kubernetes — deploy tham chiếu commit

```yaml
# deploy/k8s/deployment.yaml (trích)
spec:
  template:
    metadata:
      labels:
        app: myapp
      annotations:
        app.kubernetes.io/version: "1.7.3"
        myapp/commit: "9f8e7d6"     # ai kubectl describe cũng thấy nguồn gốc
    spec:
      containers:
        - name: server
          image: registry.example.com/myapp:9f8e7d6c...  # tag = commit SHA
```

## 5. Trade-off

**Monorepo vs polyrepo.** Câu hỏi hay gặp: "nhiều service trong một repo có vi phạm Factor 1?" Trả lời: **không**, nếu mỗi service vẫn có ranh giới rõ (thư mục riêng, pipeline build riêng, deploy độc lập) — Google, Meta vận hành monorepo khổng lồ mà vẫn đúng tinh thần "một nguồn sự thật". Trade-off thực sự nằm ở tooling: monorepo cần CI thông minh (chỉ build phần thay đổi — Bazel, Turborepo, Go workspace) và kỷ luật ownership; polyrepo cần quản lý version chéo repo và cơn đau "thay đổi 1 API chạm 5 repo". Nguyên tắc bất biến của Factor 1 chỉ là: *mọi deploy truy về được một commit trong một lịch sử được kiểm soát* — cả hai mô hình đều thỏa được.

**Fork-per-customer vs multi-tenant.** Bán phần mềm cho nhiều khách hàng có nhu cầu khác nhau, fork mỗi khách một bản là con đường ngắn hạn dễ đi và dài hạn tự sát (n bản code = n lần chi phí bảo trì, tăng mãi). Giải pháp đúng: một codebase + feature flags + config theo tenant (chương 23 bàn multi-tenant kỹ hơn). Chấp nhận fork chỉ khi hợp đồng thực sự đòi hỏi phân kỳ vĩnh viễn — và khi đó hãy coi nó là hai sản phẩm.

## 6. Best Practices

- Trunk-based development hoặc GitHub Flow với branch sống ngắn; branch sống hàng tháng là mầm của "nhiều codebase trá hình".
- Mọi artifact (image) gắn tag bằng commit SHA; cấm build từ working directory chưa commit trên CI.
- Endpoint `/version` (hoặc in ra log lúc khởi động) trong mọi service.
- Code dùng chung giữa các app → tách library có semver, tiêu thụ qua `go.mod`; cấm copy-paste chéo repo.
- Bảo vệ branch chính: mọi thay đổi qua PR + CI xanh; production chỉ deploy từ branch chính (hoặc tag từ nó).

## 7. Anti-patterns

- **Sửa code trực tiếp trên server/container production** — code đang chạy không tồn tại trong bất kỳ lịch sử nào; lần deploy sau âm thầm hoàn tác nó. Đây là anti-pattern nguy hiểm nhất của factor này.
- **"Deploy branch"**: branch `production` chứa những commit không bao giờ merge về trunk → hai dòng sự thật phân kỳ vĩnh viễn.
- **Build từ máy cá nhân** đẩy thẳng lên production — artifact phụ thuộc trạng thái máy dev (dependency lệch, code chưa commit lẫn vào).
- **Fork-per-customer không kiểm soát** (đã phân tích ở Trade-off).
- **Nhét library dùng chung vào từng repo bằng copy-paste** — mỗi bản copy trở thành một codebase con không ai cập nhật.

## 8. Khi nào KHÔNG cần áp dụng (đầy đủ)

Nói thẳng: **gần như không có ngoại lệ.** Đây là factor có chi phí thấp nhất (Git là miễn phí và phổ cập) và giá trị nền tảng nhất. Ngay cả script cá nhân, internal tool, MVP 2 tuần — một `git init` không làm bạn chậm đi phút nào. Các "ngoại lệ" hiếm hoi mang tính kỹ thuật: notebook thí nghiệm dùng một lần, cấu hình vendor không cho phép version control (hãy vẫn export và commit nếu được). Nếu có một factor để tuân thủ vô điều kiện, đó là factor này.

---

## Tóm tắt

- Một app ↔ một codebase trong version control; nhiều deploy chỉ khác nhau ở **config và commit**, không bao giờ khác nhau ở "dòng lịch sử".
- Giá trị cốt lõi: **truy vết** (đang chạy gì) và **tái tạo** (dựng lại được). Khép kín chuỗi bằng `-ldflags` nhúng commit SHA vào binary + image tag theo SHA + endpoint `/version`.
- Code dùng chung → library có version, không copy-paste; nhiều khách hàng → flags + config, không fork.
- Factor rẻ nhất, nền tảng nhất, ít ngoại lệ nhất trong cả 12.
