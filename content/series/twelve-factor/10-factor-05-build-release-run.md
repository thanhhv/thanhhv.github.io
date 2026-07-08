+++
title = "Chương 10 — Factor 5: Build, Release, Run"
date = "2026-07-10T11:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> *"Strictly separate build and run stages"* — Tách bạch nghiêm ngặt các giai đoạn build và run.
>
> **Phần 2 – The Twelve Factors** | Nhóm: **Reproducible** | Chương trước: [Backing Services](/series/twelve-factor/09-factor-04-backing-services/) | Chương sau: [Processes](/series/twelve-factor/11-factor-06-processes/)

---

## 1. Problem Statement

Câu hỏi factor này trả lời: **"Từ code đến process đang chạy, chuyện gì xảy ra — và có đảo ngược được không?"**

Trong thế giới chương 1, ba việc trộn làm một: build trên máy dev, copy lên server, sửa config tại chỗ, có khi sửa cả code tại chỗ. Hệ quả:

- **Không trả lời được "đang chạy bản nào"** — thứ đang chạy là code + các bản vá tay không tồn tại ở đâu khác.
- **Không rollback được** — "bản trước" không được lưu như một khối nguyên vẹn; rollback nghĩa là cố nhớ lại và làm ngược các thao tác tay.
- **Deploy không lặp lại được** — làm lại từ đầu chưa chắc ra kết quả cũ.
- Kịch bản kinh điển: sự cố lúc 2h sáng → sửa nóng file trên server → sáng hôm sau CI deploy bản mới → **bản vá bị ghi đè, sự cố quay lại** → và giờ không ai nhớ đêm qua đã sửa gì.

## 2. Tại sao nguyên lý này tồn tại

- **Deployment**: tự động hóa đòi hỏi quy trình là các bước thuần túy, đầu vào rõ, đầu ra rõ. "Build rồi sửa tay một chút" không tự động hóa được.
- **Operational**: rollback phải là thao tác một lệnh trong một phút. Điều đó chỉ khả thi khi mỗi lần deploy tạo ra một **release bất biến, đánh số, lưu giữ được** — quay về = trỏ lại release cũ.
- **Business/Compliance**: audit đòi hỏi "bản chạy lúc 14:03 ngày 5/3 là gì, ai phê duyệt" — chỉ trả lời được khi release là thực thể có định danh.
- **Scalability**: autoscaler tạo instance mới bất kỳ lúc nào; nó cần một artifact hoàn chỉnh để chạy ngay, không cần "làm thêm vài bước tay".

## 3. Bản chất

Chia vòng đời thành ba giai đoạn với **ranh giới một chiều** — mỗi giai đoạn có đầu vào, đầu ra và tính chất riêng:

```
   BUILD                      RELEASE                      RUN
   ─────                      ───────                      ───
   vào:  code (commit SHA)    vào:  build + config         vào:  release
         + dependencies             của MỘT môi trường
   ra:   BUILD ARTIFACT       ra:   RELEASE                ra:   process đang chạy
         (image bất biến)           (bất biến, có ID,
                                     rollback về được)
   khi:  mỗi lần code đổi     khi:  artifact HOẶC config   khi:  bất kỳ lúc nào —
                                    đổi                          restart, scale,
   phức tạp: được phép cao    phức tạp: thấp                    thay node hỏng
   (compile, test, tool)      (ghép hai thứ có sẵn)        phức tạp: PHẢI tối thiểu
                                                           (không build, không cài,
                                                            không quyết định gì)

   ═══ RANH GIỚI MỘT CHIỀU: không sửa code sau build, không sửa release sau release,
       không "vá" process đang chạy. Muốn đổi → quay lại giai đoạn trước. ═══
```

Ba hệ quả logic đáng khắc cốt:

1. **`release = build ⊗ config`** — đây là điểm F3 và F5 khớp nhau: vì config tách khỏi code, một build dùng được cho mọi môi trường; release chỉ là phép ghép. Suy ra: **build một lần, deploy nhiều nơi** — image staging và production là *cùng một image*, cùng digest. Thứ bạn test chính là thứ bạn chạy.
2. **Run phải "ngu"** — giai đoạn run xảy ra không báo trước (node chết, autoscale lúc 3h sáng), nên nó phải đơn giản đến mức không thể hỏng: lấy release, chạy. Mọi thứ thông minh (compile, migration, quyết định) dồn về build/release, nơi có người trực và có thể fail an toàn.
3. **Rollback = re-release bản cũ** — vì release bất biến và được lưu, quay lại chỉ là trỏ con trỏ. Không cần build lại (rủi ro build ra thứ khác), không cần "sửa ngược".

**Điều gì xảy ra nếu vi phạm?** Mỗi kiểu vi phạm phá một hệ quả: build riêng cho từng môi trường → thứ test ≠ thứ chạy; container tự build/tự cài lúc khởi động → khởi động chậm + có thể fail lúc không ai trực (npm registry sập lúc autoscale = không scale được); sửa nóng trên server → release đang chạy không tương ứng artifact nào, rollback và audit đều mù.

## 4. Cách áp dụng

### 4.1. Build — GitHub Actions tạo artifact bất biến

```yaml
# .github/workflows/build.yml (trích)
name: build
on:
  push:
    branches: [main]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with: { go-version-file: go.mod }

      - run: go vet ./... && go test -race ./...   # test là một phần của BUILD

      - name: Build & push image (đánh tag bằng commit SHA — bất biến)
        run: |
          docker build \
            --build-arg COMMIT_SHA=${{ github.sha }} \
            -t $REGISTRY/myapp:${{ github.sha }} .
          docker push $REGISTRY/myapp:${{ github.sha }}
          # Ghi lại digest — định danh tuyệt đối của artifact
          docker inspect --format='{{index .RepoDigests 0}}' $REGISTRY/myapp:${{ github.sha }}
```

Yêu cầu với giai đoạn build: chạy trên CI (không phải máy cá nhân), từ commit sạch, kết quả **chỉ phụ thuộc commit** (reproducible), đẩy vào registry — registry chính là kho lưu build artifact.

### 4.2. Release + Run trên Kubernetes

Trên Kubernetes, ánh xạ ba giai đoạn rất rõ:

- **Build artifact** = container image (theo digest).
- **Release** = Deployment spec đã render đầy đủ: image digest + ConfigMap/Secret + tham số. Với Helm: `helm release` đúng nghĩa đen — có số revision, lưu lịch sử, rollback được. Với GitOps (chương 23): release = một commit trong repo config.
- **Run** = kubelet kéo image và chạy container theo spec. Không có bước nào khác.

```yaml
# deployment.yaml (trích) — một RELEASE cụ thể
spec:
  replicas: 4
  strategy:
    rollingUpdate: { maxUnavailable: 0, maxSurge: 1 }   # release mới thay dần release cũ
  template:
    metadata:
      labels:
        app: myapp
        version: "9f8e7d6"          # nhìn pod biết release
    spec:
      containers:
        - name: server
          # Pin theo DIGEST — release trỏ vào artifact tuyệt đối, không thể xê dịch
          image: registry.example.com/myapp@sha256:4c2a...e91b
          envFrom:
            - configMapRef: { name: myapp-config-v42 }  # config cũng có version
```

```bash
# Rollback = re-release bản cũ, một lệnh:
helm rollback myapp 41            # hoặc
kubectl rollout undo deployment/myapp
# GitOps: git revert <commit> → hệ thống tự hội tụ về release cũ
```

### 4.3. Database migration nằm ở đâu trong ba giai đoạn?

Câu hỏi thực chiến hay gặp nhất của factor này. Migration **không thuộc run** (run phải ngu) và **không thuộc build** (build không được chạm môi trường). Nó là một **release step** — chạy một lần cho mỗi lần release, trước khi phiên bản mới nhận traffic:

```yaml
# Job migration — chạy như một bước của release (hoặc Helm pre-upgrade hook / initContainer
# tùy quy mô; Job tách riêng là sạch nhất vì chạy đúng MỘT lần, không phải mỗi pod)
apiVersion: batch/v1
kind: Job
metadata:
  name: myapp-migrate-9f8e7d6
spec:
  backoffLimit: 2
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: migrate
          image: registry.example.com/myapp@sha256:4c2a...e91b  # CÙNG image với app (F12!)
          args: ["migrate", "up"]
          envFrom:
            - secretRef: { name: myapp-secrets }
```

Kèm kỷ luật **expand–contract** (vì trong rolling update, bản cũ và mới chạy song song trên cùng schema): bước 1 release schema mở rộng (thêm cột, tương thích cả hai bản) → bước 2 release code mới → bước 3 release dọn schema cũ. Chi tiết ở chương 24.

## 5. Trade-off

**Build-once-run-anywhere vs build-per-environment.** Chuẩn là build once. Nhưng có ngoại lệ thực tế: frontend SPA nhúng biến lúc build (`VITE_API_URL`) buộc build theo môi trường — cách chữa là runtime config (`config.js` bơm lúc serve). Go không có vấn đề này: binary đọc env lúc chạy. Nếu bạn thấy mình cần `--build-arg ENVIRONMENT=prod` cho backend Go — đó là dấu hiệu config đang lọt vào build, quay lại F3.

**Tốc độ hotfix vs kỷ luật quy trình.** Sửa nóng trên server nhanh hơn (phút) so với đi qua pipeline (5–15 phút). Nhưng phép so sánh đúng phải tính cả kỳ vọng rủi ro: bản vá tay bị ghi đè, không audit, không test. Đầu tư đúng là **làm pipeline nhanh** (build cache, test song song, deploy tự động) để con đường kỷ luật cũng là con đường nhanh — khi pipeline dưới 10 phút, cám dỗ sửa tay gần như biến mất.

**Lưu bao nhiêu release?** Mỗi release chiếm chỗ trong registry. Thực dụng: giữ mọi release N tháng gần nhất + mọi release từng chạy production (tag riêng), dọn image của branch/PR sau vài tuần bằng registry lifecycle policy.

## 6. Best Practices

- Build chỉ trên CI, từ commit đã push; cấm `docker push` từ máy cá nhân lên registry production (enforce bằng quyền registry).
- Tag image bằng commit SHA; production tham chiếu digest; cấm `:latest` sau ranh giới CI.
- Test chạy trong giai đoạn build — artifact đã push nghĩa là đã qua test.
- Release có định danh và lịch sử: Helm revision, hoặc Git history của repo GitOps.
- Rollback được diễn tập, không chỉ được tin là có: thử `rollout undo` trên staging định kỳ.
- Migration là release step, cùng image với app, theo expand–contract.
- Đo thời gian pipeline như một SLO của team platform — pipeline chậm là nguyên nhân số một khiến kỷ luật bị phá.

## 7. Anti-patterns

- **Sửa code/config trực tiếp trên server hoặc trong container đang chạy** — phá cả ba ranh giới một lúc; bản vá sẽ bị ghi đè bởi lần deploy sau.
- **Build lúc khởi động container** (clone repo + `go build` trong entrypoint; `npm install` khi start) — run phụ thuộc network + registry ngoài đúng lúc nhạy cảm nhất (autoscale, recovery); khởi động chậm phá luôn F9.
- **Build riêng từng môi trường** (`make build-staging`, `make build-prod`) — hai artifact khác nhau, thứ test không phải thứ chạy.
- **CI đè tag** (`myapp:staging` được push đè liên tục) — "staging đang chạy gì?" không có câu trả lời xác định; hai pod cùng tag có thể chạy hai code khác nhau sau restart.
- **Rollback bằng cách build lại code cũ** — chậm (chờ build lúc đang cháy) và rủi ro (build mới từ code cũ chưa chắc giống build cũ nếu dependency trôi). Rollback phải dùng lại artifact cũ.
- **Migration tự động chạy trong `main()` của app** — N pod cùng khởi động là N lần đua migration; lỗi migration làm crash-loop toàn bộ fleet thay vì fail một Job có kiểm soát.

## 8. Khi nào KHÔNG cần áp dụng đầy đủ

- **Ngôn ngữ thông dịch + môi trường đơn giản** (script Python nội bộ): "build" gần như rỗng — nhưng ranh giới release/run vẫn đáng giữ (deploy từ Git tag, không sửa trên server).
- **Dev local**: hot-reload (air, docker compose watch) cố tình xóa ranh giới build/run để lặp nhanh — hợp lệ, vì dev không phải một "deploy" theo nghĩa production. Ranh giới chỉ bắt buộc từ CI trở đi.
- **Hệ thống một máy, một người** (side project): kỷ luật đầy đủ có thể quá nặng; mức tối thiểu đáng giữ: build từ Git, giữ lại binary cũ để quay về.

---

## Tóm tắt

- Ba giai đoạn, ranh giới một chiều: **build** (code → artifact bất biến), **release** (artifact ⊗ config → release có ID), **run** (chạy release, không quyết định gì).
- **Build once, run anywhere**: staging và prod chạy cùng một image, cùng digest — thứ bạn test là thứ bạn chạy.
- **Rollback = re-release bản cũ** — một lệnh, một phút; không bao giờ build lại để rollback.
- Run xảy ra lúc không ai trực → run phải tối giản; mọi phức tạp dồn về build/release. Migration là release step với kỷ luật expand–contract.
