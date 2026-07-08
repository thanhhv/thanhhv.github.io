+++
title = "Chương 21 — CI/CD: tự động hóa Build → Release → Deploy"
date = "2026-07-10T22:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> **Phần 3 – Engineering** | Chương trước: [Kubernetes](/series/twelve-factor/20-kubernetes/) | Chương sau: [Observability](/series/twelve-factor/22-observability/)

Factor 5 định nghĩa ba giai đoạn build–release–run; chương này xây **cỗ máy** thi hành chúng: pipeline CI/CD hoàn chỉnh với GitHub Actions, và mô hình GitOps cho phần deploy.

---

## 1. Bản chất: CI/CD là gì — tách bạch hai khái niệm

- **CI (Continuous Integration)**: mọi thay đổi được tích hợp vào trunk thường xuyên, và mỗi lần tích hợp được **kiểm chứng tự động** (build + test). CI là về *niềm tin vào trạng thái của trunk*: trunk luôn ở trạng thái ship được.
- **CD** có hai nghĩa cần phân biệt: **Continuous Delivery** — mọi commit qua CI đều *sẵn sàng* deploy (artifact + quy trình tự động, người bấm nút); **Continuous Deployment** — bỏ luôn nút bấm, mọi commit xanh tự lên production. Đa số tổ chức trưởng thành dừng ở Delivery cho production và Deployment cho staging — đó là lựa chọn quản trị rủi ro, không phải mức độ "tiến hóa".

Điểm nối với 12-Factor: pipeline chính là **ranh giới cưỡng chế** của các factor — codebase (chỉ build từ Git), dependencies (verify trong CI), build/release/run (mỗi stage một job), parity (cùng artifact đi xuyên môi trường). Kỷ luật con người thất bại theo thời gian; kỷ luật pipeline thì không.

## 2. Pipeline hoàn chỉnh — GitHub Actions

```yaml
# .github/workflows/ci.yml
name: ci
on:
  pull_request:
  push:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE: ghcr.io/${{ github.repository }}

jobs:
  # ---------- STAGE 1: VERIFY — chạy cho cả PR lẫn main ----------
  verify:
    runs-on: ubuntu-latest
    services:                # backing service THẬT cho integration test (F10)
      postgres:
        image: postgres:17.4-alpine
        env: { POSTGRES_USER: test, POSTGRES_PASSWORD: test, POSTGRES_DB: test }
        options: >-
          --health-cmd pg_isready --health-interval 2s --health-retries 15
        ports: ["5432:5432"]
      redis:
        image: redis:7.2-alpine
        ports: ["6379:6379"]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with: { go-version-file: go.mod, cache: true }

      - name: Dependencies hygiene (F2)
        run: |
          go mod tidy && git diff --exit-code go.mod go.sum
          go mod verify

      - name: Lint
        uses: golangci/golangci-lint-action@v6

      - name: Unit + integration tests
        env:
          DATABASE_URL: postgres://test:test@localhost:5432/test?sslmode=disable
          REDIS_URL: redis://localhost:6379/0
        run: go test -race -count=1 -coverprofile=cover.out ./...

      - name: Vulnerability scan (F2)
        run: go run golang.org/x/vuln/cmd/govulncheck@latest ./...

  # ---------- STAGE 2: BUILD ARTIFACT — chỉ trên main (F5) ----------
  build:
    needs: verify
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    permissions: { contents: read, packages: write, id-token: write }
    outputs:
      digest: ${{ steps.push.outputs.digest }}
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build & push (tag = commit SHA — F1+F5)
        id: push
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ${{ env.IMAGE }}:${{ github.sha }}
          build-args: |
            COMMIT_SHA=${{ github.sha }}
            VERSION=${{ github.ref_name }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Scan image
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.IMAGE }}:${{ github.sha }}
          exit-code: "1"
          severity: CRITICAL,HIGH

      # Ký image + SBOM: chuỗi cung ứng kiểm chứng được (cosign keyless)
      - name: Sign image
        run: cosign sign --yes ${{ env.IMAGE }}@${{ steps.push.outputs.digest }}

  # ---------- STAGE 3: RELEASE — cập nhật repo GitOps (xem mục 3) ----------
  release-staging:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Update GitOps repo (staging tự động — Continuous Deployment)
        run: |
          # clone repo gitops, cập nhật digest trong overlays/staging, commit, push
          # (dùng deploy key/app token có quyền hẹp)
          yq -i '.images[0].digest = "${{ needs.build.outputs.digest }}"' \
             overlays/staging/kustomization.yaml
          git commit -am "staging: myapp ${{ github.sha }}" && git push

  release-production:
    needs: release-staging
    environment: production        # GitHub Environments: bắt buộc phê duyệt —
    runs-on: ubuntu-latest         # Continuous DELIVERY: người bấm nút
    steps:
      - name: Update GitOps repo (production)
        run: |
          yq -i '.images[0].digest = "${{ needs.build.outputs.digest }}"' \
             overlays/production/kustomization.yaml
          git commit -am "production: myapp ${{ github.sha }}" && git push
```

Những đường nét quan trọng: **PR chỉ verify** (không build artifact — tiết kiệm, và artifact chỉ sinh từ trunk); **một artifact duy nhất** đi từ staging đến production (chỉ digest được sao chép giữa hai overlay — F5/F10 bằng cơ chế, không bằng lời hứa); **quyền hẹp** (job build không có quyền deploy; token GitOps chỉ ghi được repo config).

## 3. GitOps — deploy bằng pull, không phải push

Mô hình truyền thống (push): CI có credential của cluster, chạy `kubectl apply`. Vấn đề: CI trở thành chỗ chứa chìa khóa vạn năng của production; trạng thái cluster trôi dạt khỏi Git theo thời gian (ai đó `kubectl edit`); khôi phục cluster = khảo cổ học.

GitOps (pull) đảo chiều:

```
  Repo GitOps (mô tả TOÀN BỘ trạng thái mong muốn)      Cluster
  ├── apps/myapp/base/                                  ┌────────────────────┐
  ├── apps/myapp/overlays/{staging,production}/    ◀────│ Argo CD / Flux     │
  └── infrastructure/ (ingress, observability...)  sync │ agent TRONG cluster│
                                                        │ liên tục đối chiếu │
  CI chỉ được COMMIT vào repo này ──────────────────▶   │ Git ↔ thực tế      │
  (không bao giờ chạm cluster)                          └────────────────────┘
```

- **Git là nguồn sự thật duy nhất** của trạng thái vận hành — mở rộng Factor 1 từ code sang cả *hạ tầng khai báo*. Mọi thay đổi production là một commit: có tác giả, review, lịch sử, revert.
- **Rollback = `git revert`** — hiện thân sạch nhất từng có của "re-release bản cũ" (F5).
- **Drift tự chữa**: ai `kubectl edit` tay, agent phát hiện lệch và đưa về đúng Git (hoặc cảnh báo) — immutable infrastructure có máy tuần tra.
- **Bảo mật đảo chiều**: không credential cluster nào rời khỏi cluster; bề mặt tấn công CI giảm hẳn.

Chi phí thật của GitOps: thêm một repo + một hệ thống (Argo CD/Flux) phải vận hành; vòng phản hồi gián tiếp hơn (commit → chờ sync → xem trạng thái ở UI Argo thay vì ngay trong log CI); secret cần lời giải riêng (External Secrets Operator kéo từ secret manager là lời giải phổ biến — đừng SealedSecrets nếu tổ chức đã có Vault/ASM). Với 1–2 service và một cluster, `kubectl apply` từ CI với credential hẹp vẫn là lựa chọn hợp lý — GitOps tỏa sáng từ khi số service × số môi trường đủ lớn.

## 4. Chiến lược test trong pipeline — hình chóp thực dụng

```
        ┌────────┐  E2E/smoke sau deploy staging: ÍT (đắt, chậm, mong manh)
      ┌─┴────────┴─┐  integration (testcontainers/service thật): VỪA —
    ┌─┴────────────┴─┐   đây là tầng bắt bug thật nhiều nhất với backend
    │   unit (-race) │  NHIỀU: nhanh, rẻ, chạy mọi PR
    └────────────────┘
```

Ba nguyên tắc: **`-race` luôn bật** (bug concurrency Go rẻ nhất khi bắt ở CI); test phụ thuộc *hành vi* không phụ thuộc *thứ tự/dữ liệu chung* (mỗi test tự tạo dữ liệu của mình — chạy song song được); pipeline **phải nhanh** — mục tiêu <10 phút từ push đến staging, vì tốc độ pipeline quyết định mọi kỷ luật khác (chương 10 đã lập luận).

## 5. Anti-patterns

- **Deploy từ máy cá nhân** bằng kubeconfig admin — mọi kỷ luật pipeline thành trang trí; thu hồi bằng RBAC, không bằng lời nhắc.
- **PR test một đằng, main build một nẻo** không chung bước verify — artifact sinh ra từ code chưa qua đúng bộ test.
- **Secret production trong CI vars dùng chung** cho mọi workflow/branch — PR từ fork chạy code tùy ý với secret thật; dùng environment-scoped secrets + OIDC (id-token) thay long-lived key.
- **Test quarantine vĩnh viễn** (`skip flaky`) — flaky test là bug thật của test hoặc của app; skip là tắt chuông báo cháy.
- **Pipeline 45 phút** — dev gộp commit to, né test, deploy dồn cục; tốc độ pipeline là đòn bẩy văn hóa số một.
- **`kubectl apply -f` thẳng từ CI khi đã tuyên bố GitOps** — hai nguồn sự thật đánh nhau; chọn một mô hình và đóng đường kia bằng quyền.
- **Đè tag image thay vì digest trong manifest** — Argo "synced" mà cluster chạy image khác; đã nhắc từ F5, xuất hiện lại ở đây vì GitOps làm hậu quả kín đáo hơn.

## 6. Khi nào KHÔNG cần đầy đủ

- **Side project / MVP một người**: CI tối thiểu (test + build) vẫn đáng có (30 phút setup), CD có thể là script `deploy.sh` gọi Cloud Run — miễn build từ Git, artifact có version.
- **Phần mềm ship theo quý cho khách on-prem**: CD liên tục không áp; nhưng CI + artifact ký + reproducible build lại càng quan trọng.
- **GitOps**: như mục 3 — dưới ngưỡng phức tạp nhất định, push-based đơn giản hơn là đúng.

---

## Tóm tắt

- CI = trunk luôn ship được (verify tự động mỗi lần tích hợp); CD = delivery (sẵn sàng, người bấm) hoặc deployment (tự động hoàn toàn) — chọn theo khẩu vị rủi ro từng môi trường.
- Pipeline là **cơ chế cưỡng chế 12-Factor**: một artifact từ trunk, đi nguyên vẹn qua các môi trường, chỉ config đổi; quyền hẹp từng stage; ký + scan + SBOM cho chuỗi cung ứng.
- GitOps: Git là nguồn sự thật của trạng thái vận hành, agent pull và tự chữa drift; rollback = git revert — Factor 1 + 5 + immutable infrastructure hội tụ.
- Giữ pipeline <10 phút — tốc độ pipeline là nền của mọi kỷ luật còn lại.
