+++
title = "Bài 6 — CI/CD & Deployment Strategy"
date = "2026-02-01T12:00:00+07:00"
draft = false
tags = ["backend", "infrastructure", "networking"]
series = ["Infrastructure & Networking"]
+++

# Module 06 — CI/CD và Deployment Strategy

> CI/CD không phải chuyện công cụ (Jenkins vs GitHub Actions). Nó là bài toán **quản lý rủi ro thay đổi**: thay đổi là nguồn sự cố số 1 của mọi hệ thống production, và cũng là thứ duy nhất tạo ra giá trị. Mục tiêu: tăng tần suất thay đổi VÀ giảm blast radius của mỗi thay đổi — hai thứ tưởng mâu thuẫn nhưng thực ra cộng hưởng.

---

## 1. Tại sao CI/CD tồn tại

### Problem Statement
Deploy thủ công có vòng lặp tử thần: deploy đáng sợ → deploy ít → mỗi lần deploy dồn nhiều thay đổi → rủi ro mỗi lần cao hơn → đáng sợ hơn. Batch càng lớn, việc tìm "commit nào gây lỗi" càng khó (bisect 100 commit vs 3 commit). Nghiên cứu DORA chỉ ra nhất quán: team deploy **thường xuyên hơn** có tỷ lệ sự cố **thấp hơn** — vì mỗi thay đổi nhỏ, dễ hiểu, dễ rollback.

CI/CD đảo ngược vòng lặp: tự động hóa làm deploy rẻ → deploy nhỏ và thường xuyên → rủi ro mỗi lần thấp → tự tin hơn → nhanh hơn nữa.

---

## 2. Build Pipeline và Artifact

### Nguyên tắc cốt lõi: build một lần, deploy nhiều lần
**Artifact bất biến** (container image với tag theo git SHA) được build ĐÚNG MỘT LẦN, rồi chính binary đó đi qua staging → production. Build lại cho từng môi trường = mỗi môi trường chạy binary khác nhau = "pass staging" không chứng minh gì. Mọi khác biệt giữa môi trường phải nằm ở **config tiêm lúc runtime** (env, ConfigMap, secret store), không nằm trong artifact.

Pipeline chuẩn:
```
commit → lint + unit test (nhanh, <5 phút — fail sớm)
       → build image (multi-stage, cache — module 04)
       → scan (CVE, secret leak) + ký (cosign)
       → push registry (tag = git SHA)
       → integration test trên môi trường ephemeral
       → deploy staging → smoke test
       → deploy production (chiến lược ở phần 3, có cổng tự động)
```

Nguyên tắc vận hành pipeline:
- **Tốc độ là tính năng**: pipeline 45 phút → dev gộp thay đổi, né test → phá mục tiêu ban đầu. Đầu tư cache, song song hóa, test phân tầng (unit nhanh chạy mọi commit; e2e đắt chạy trước deploy).
- **Flaky test là nợ độc**: test lúc đỏ lúc xanh dạy con người bấm "retry" cho đến khi... một ngày lỗi thật bị retry cho qua. Quarantine flaky test ngay, sửa hoặc xóa.
- Registry là hạ tầng production: registry chết = không deploy được = không **rollback** được. HA registry, replication, và pull-through cache tại cluster.

---

## 3. Deployment Strategies — bảng quyết định trung tâm

Mọi chiến lược đều trả lời một câu: **bao nhiêu % user gặp phiên bản mới trước khi bạn biết nó hỏng?**

| Chiến lược | Cách chạy | Blast radius | Chi phí | Rollback | Khi dùng |
|---|---|---|---|---|---|
| Recreate | Tắt hết cũ → bật mới | 100% + downtime | Thấp nhất | Deploy lại bản cũ | Dev, batch job |
| Rolling update | Thay dần từng batch | Tăng dần, trộn 2 version | Thấp | Chậm (roll ngược từng batch) | Mặc định k8s, đa số service |
| Blue-Green | Dựng full môi trường mới, chuyển traffic 1 phát | 100% nhưng chuyển về tức thời | 2× hạ tầng lúc deploy | **Gần tức thời** (đổi lại con trỏ) | Cần rollback nhanh, release có nghi lễ |
| Canary | 1% → 5% → 25% → 100%, đo metric mỗi bước | Nhỏ nhất, kiểm soát | Cần L7 traffic split + metric tốt | Tự động khi metric xấu | Service quan trọng, traffic đủ lớn |

### Những sự thật ít được nói
- **Rolling update nghĩa là 2 phiên bản chạy đồng thời** — code và schema PHẢI tương thích ngược một bậc (N và N-1). Đây là nguồn của quy tắc migration: **expand → migrate → contract** (thêm cột mới, deploy code đọc cả hai, backfill, rồi mới xóa cột cũ — mỗi bước một deploy riêng). Đổi schema kiểu "rename column" trong một deploy = outage giữa lúc rolling.
- **Blue-green không cứu được database**: 2 môi trường app nhưng chung 1 DB — schema vẫn phải tương thích cả 2 chiều. "Rollback tức thời" chỉ tức thời với stateless.
- **Canary cần toán**: 1% traffic × lỗi hiếm 0.1% → cần lượng request đủ lớn mới có ý nghĩa thống kê. Traffic thấp thì canary 1% là kịch — hãy dùng thời gian dài hơn hoặc so sánh với baseline đối chứng (như Argo Rollouts/Kayenta làm).
- **Session/connection dài** (WebSocket — module 02): mọi chiến lược đều phải cộng thêm bài toán drain connection cũ.

### Rollback — thiết kế trước, không ứng biến
- Định nghĩa trước **điều kiện rollback tự động** (error rate, p99, saturation vượt ngưỡng trong X phút) — con người dưới áp lực sự cố quyết định chậm và dở.
- Rollback phải là **một lệnh, không cần suy nghĩ, đã được diễn tập**. Nếu rollback là "wiki 12 bước", bạn không có rollback.
- Thứ không rollback được bằng deploy: data đã ghi bởi bản lỗi, message đã publish, cache đã độc. Đó là lý do canary (chặn trước khi lan) giá trị hơn rollback (dọn sau khi lan).

---

## 4. Feature Flag — tách deploy khỏi release

### Ý tưởng đổi đời
**Deploy** (đưa code lên máy) và **release** (cho user thấy hành vi mới) là hai việc khác nhau. Feature flag tách chúng: code lên production ở trạng thái tắt, bật dần theo %, theo segment, tắt tức thì khi hỏng — **kill switch nhanh hơn mọi rollback deploy** (mili giây vs phút).

Hệ quả tổ chức: merge nhỏ liên tục vào main (trunk-based) mà không chờ tính năng hoàn chỉnh — flag che phần dở dang. Đây là cách các công ty deploy trăm lần/ngày.

### Cái giá và kỷ luật
- Mỗi flag = một nhánh runtime = tổ hợp trạng thái phải test. 50 flag sống lâu = 2^50 tổ hợp lý thuyết = không ai hiểu hệ thống nữa. **Flag phải có ngày hết hạn và người chịu trách nhiệm xóa**; xóa flag là một phần của definition-of-done.
- Hệ thống flag trở thành dependency critical: flag service chết thì app hành xử thế nào? Bắt buộc: default an toàn hard-code + cache local. Đã có outage lớn trong ngành vì flag system đẩy config rác xuống toàn fleet — pipeline config/flag cũng cần canary như code.
- Flag check trong hot path → latency: evaluate local từ config đã sync, không gọi mạng per-request.

---

## 5. GitOps và CD cho Kubernetes

Mô hình: git repo chứa manifest = nguồn sự thật; agent trong cluster (ArgoCD/Flux) liên tục **reconcile** cluster về đúng repo (đúng triết lý Kubernetes — module 05). Lợi: audit trail = git log, rollback = git revert, drift tự phát hiện, không ai cần quyền kubectl vào prod. Giá: thêm một hệ phải vận hành, và vòng sync tạo trễ — hiểu rõ khi nào cần "sync ngay" (rollback khẩn).

---

## 6. Anti-patterns tổng hợp

- Deploy thứ Sáu chiều không phải anti-pattern; **deploy khi không ai nhìn metric** mới là anti-pattern. (Team đủ trưởng thành với canary tự động deploy được mọi lúc.)
- Môi trường staging lệch xa production (kích cỡ, data, config) → staging pass vô nghĩa → "test trên production một cách vô tình". Hoặc đầu tư staging giống thật, hoặc thừa nhận và đầu tư vào canary + observability.
- Pipeline có bước manual approve nhưng người approve không có thông tin gì ngoài nút bấm → "ritual approval", chỉ thêm trễ không thêm an toàn. Thay bằng cổng metric tự động.
- Ai đó `kubectl edit` trực tiếp trên prod (hotfix tay) → drift khỏi git → deploy sau **ghi đè mất hotfix** → sự cố tái phát lúc 3h sáng. GitOps + chặn quyền ghi trực tiếp giải quyết tận gốc.
- Secret trong git/CI log. Secret đi qua secret manager (module 10), CI chỉ giữ quyền truy cập ngắn hạn (OIDC federation, không static key).

## Checklist trưởng thành CD
- [ ] Artifact bất biến, build 1 lần, tag theo SHA, ký + scan
- [ ] Pipeline < 15 phút tới staging; test phân tầng; flaky = quarantine
- [ ] Schema migration expand/contract, tương thích N-1
- [ ] Canary hoặc blue-green cho service quan trọng, cổng metric tự động
- [ ] Rollback 1 lệnh, đã diễn tập; kill switch bằng flag cho tính năng lớn
- [ ] GitOps: cluster = git, không sửa tay
- [ ] Đo 4 chỉ số DORA: deploy frequency, lead time, change failure rate, MTTR
