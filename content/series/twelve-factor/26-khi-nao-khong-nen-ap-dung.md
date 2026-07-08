+++
title = "Chương 26 — Khi nào KHÔNG nên áp dụng 12-Factor (và lời kết)"
date = "2026-07-11T03:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> **Phần 4** | Chương trước: [So sánh khách quan](/series/twelve-factor/25-so-sanh-khach-quan/) | [Về mục lục](/series/twelve-factor/00-muc-luc/)

Mỗi chương factor đã có mục "khi nào không áp dụng" riêng. Chương cuối này tổng hợp chúng thành một khung quyết định — và khép lại tài liệu bằng những điều đáng mang theo nhất.

---

## 1. Nguyên tắc gốc để suy luận ngoại lệ

Toàn bộ 12-Factor suy ra từ một tiền đề (chương 2): **hạ tầng tạm bợ, được tự động hóa, app chạy nhiều bản sao**. Vậy quy tắc suy ngoại lệ rất gọn:

> **Ở đâu tiền đề không đúng, factor tương ứng mất tính bắt buộc.** App không bao giờ có bản sao thứ hai → sức ép stateless giảm. Không có "môi trường thứ hai" → dev/prod parity vô nghĩa. Không có nền tảng tự động → hợp đồng port/probe không có bên thi hành.

Nhưng có một nhóm factor **không phụ thuộc tiền đề đó** — chúng đúng vì lý do kỹ thuật phần mềm phổ quát (truy vết được, tái tạo được, không lộ bí mật):

**Nhóm bất khả xâm phạm (chi phí ~0, giá trị mọi nơi):** F1 Codebase (Git), F2 Dependencies (go.mod/lockfile), F3 phần *secret không vào code*, F11 phần *log có cấu trúc tối thiểu*. Vi phạm nhóm này không bao giờ là "trade-off hợp lý" — nó chỉ là cẩu thả.

**Nhóm theo tiền đề (cân nhắc theo hoàn cảnh):** F6 stateless, F8 scale-out, F9 disposability đầy đủ, F10 parity đầy đủ, F4 mức interface-hóa, F5 mức nghi thức release.

## 2. Bản đồ các hoàn cảnh — mức áp dụng khuyến nghị

| Hoàn cảnh | Áp dụng | Bỏ qua được | Giải pháp thay thế phù hợp hơn |
|---|---|---|---|
| **Internal tool** (vài chục user, 1 instance) | Nhóm bất khả xâm phạm + graceful shutdown 10 dòng | Stateless đầy đủ, K8s, autoscale, staging | Một VM/PaaS nhỏ + SQLite/Postgres nhỏ + systemd; **ghi lại điều kiện đảo chiều** ("cần instance 2 → Redis + Postgres") |
| **Desktop application** | F1, F2; config theo chuẩn OS | F3-env, F6, F7, F8, F9, F10, F11-stdout | Config file `~/.config`, SQLite local, log file + crash reporter — mô hình khác, không phải mô hình "kém" |
| **Embedded / IoT** | F1, F2 (reproducible build càng quan trọng hơn!) | Gần như toàn bộ nhóm runtime | Cross-compile static binary, OTA update có A/B slot (chính là immutable infrastructure phiên bản firmware!), log ra serial/telemetry riêng |
| **Batch job đơn giản** (một script chạy đêm) | F1, F2, F3-secret, exit code chuẩn | F6 đầy đủ, F7, F8 | Cron + script trong Git; checkpoint idempotent nếu chạy lại; nâng lên K8s CronJob *khi đã có cluster*, đừng dựng cluster vì một script |
| **MVP / prototype 2 tuần** | Nhóm bất khả xâm phạm + PaaS lo phần còn lại | Tự dựng hạ tầng, observability đầy đủ, worker tách | PaaS (Cloud Run/Fly/Railway) — trớ trêu là PaaS *ép* bạn theo phần lớn 12-Factor một cách miễn phí; đó là lối tắt đúng |
| **Hệ stateful lõi** (tự vận hành DB/broker) | F1, F2, F5, F11, immutable image | F6 (state LÀ nghiệp vụ) | StatefulSet + operator trưởng thành (CloudNativePG, Strimzi) hoặc managed service |
| **HPC / ML training** | F1, F2, F3, F11 | F6-per-request, F9-nhanh | Checkpoint định kỳ ra object storage (disposability qua checkpoint), job scheduler chuyên dụng (Slurm, Kueue) |
| **Hệ legacy đang chạy ổn** | Đừng đập; áp nhóm bất khả xâm phạm khi chạm vào | Rewrite toàn bộ "cho chuẩn" | Strangler fig: phần mới viết đúng chuẩn, phần cũ bọc và thay dần theo giá trị business — như case study chương 24 |

## 3. Ba kiểu sai lầm khi "áp dụng" — nguy hiểm ngang nhau

**Sai lầm 1 — Giáo điều (over-application).** Dựng K8s + Kafka + service mesh cho tool nội bộ 20 user; ép stateless một game server; "chuẩn hóa" một hệ legacy đang chạy ổn để rồi tạo sự cố đầu tiên sau 3 năm. Chi phí phức tạp là thật và trả trước; lợi ích tính lựa chọn chỉ đến nếu tương lai cần. *Câu hỏi giải độc: "Bài toán nào của TÔI mà factor này giải? Nếu không chỉ ra được — đừng làm."*

**Sai lầm 2 — Hình thức (cargo cult).** Có Dockerfile nhưng shell-form ENTRYPOINT; có `/healthz` trả 200 vô điều kiện; có "config từ env" nhưng default trỏ thẳng production; có GitOps nhưng vẫn `kubectl edit` khi vội. Tick đủ 12 ô mà không giữ bản chất nào — hình thức của chuẩn mực cộng rủi ro của hỗn loạn. *Câu hỏi giải độc: "Nếu tôi giết một pod ngẫu nhiên ngay bây giờ / deploy 10 lần hôm nay, có ai đau không?" — kiểm chứng bằng hành động, không bằng checklist.*

**Sai lầm 3 — Vi phạm vô thức (under-awareness).** Không phải quyết định "chúng ta chấp nhận session in-memory vì X" mà là *không ai biết* session đang in-memory — cho đến ngày scale. Khác biệt giữa kỹ sư giỏi và kỹ sư cẩu thả không nằm ở chỗ có vi phạm hay không, mà ở chỗ **vi phạm có được ghi nhận, có điều kiện đảo chiều, có nằm trong sổ nợ kỹ thuật không**. *Thực hành cụ thể: một file `ARCHITECTURE.md` liệt kê các khoản "nợ 12-Factor" có chủ đích + điều kiện trả.*

## 4. Lời kết — những điều đáng mang theo

Nếu quên hết chi tiết, hãy giữ lại năm điều:

1. **12-Factor là hệ quả của một tiền đề, không phải 12 điều răn.** Hạ tầng tạm bợ + tự động hóa → mọi factor suy ra được. Nắm tiền đề, bạn tự suy lại được các factor — và biết khi nào chúng không áp.

2. **Nó là hợp đồng hai bên.** App giữ phần mình (stateless, config ngoài, SIGTERM êm, stdout) thì nền tảng — bất kể Heroku 2011 hay K8s 2026 — mới thi hành được phần của nó (scale, heal, deploy, observe). Mọi "K8s không thấy sướng" đều nên bắt đầu điều tra từ phía hợp đồng của app.

3. **Các factor không bình đẳng về giá.** Bốn thứ gần miễn phí (Git, dependency manifest, secret ngoài code, log stdout) — làm ở mọi dự án, kể cả script cuối tuần. Phần còn lại — mua theo nhu cầu, như mua bảo hiểm: đúng rủi ro, đúng mức phí.

4. **Câu hỏi vàng dùng cho mọi review kiến trúc:** *"Chạy 2 bản sau load balancer thì hỏng gì?"* và *"Giết một instance ngẫu nhiên ngay bây giờ thì mất gì?"* — hai câu này phát hiện 80% vi phạm nghiêm trọng nhanh hơn mọi audit checklist.

5. **Vi phạm có ý thức là kỹ thuật; vi phạm vô thức là nợ xấu.** Tài liệu này thành công không phải khi bạn tuân thủ 12/12, mà khi mọi con số dưới 12 trong hệ thống của bạn đều là *quyết định* — có lý do, có ghi chép, có đường quay lại.

Chúc bạn xây những hệ thống nhàm chán khi vận hành — vì mọi sự thú vị đã được chuyển hết vào sản phẩm.

---

*Đọc tiếp / tham khảo: [12factor.net](https://12factor.net) (nguyên bản, có bản dịch nhiều thứ tiếng), "Beyond the Twelve-Factor App" (Kevin Hoffman, O'Reilly), tài liệu Kubernetes chính thức, Google SRE Book (chương SLO), và CNCF landscape cho hệ sinh thái công cụ.*
