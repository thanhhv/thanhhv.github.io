+++
title = "Chương 14 — Security: từ \"root hoặc không\" đến quyền chi li từng syscall"
date = "2026-02-21T21:00:00+07:00"
draft = false
tags = ["backend", "linux", "operating-system"]
series = ["Operating Systems cho Backend Engineer"]
+++

## 1. Problem Statement

Mô hình Unix nguyên thủy chỉ có hai hạng người: **root (uid 0) — làm được tất cả**, và còn lại — bị giới hạn bởi quyền file. Hai vấn đề chết người: (1) nhiều tác vụ chỉ cần *một mẩu* quyền root (bind port 80, đổi giờ) nhưng phải trao *cả* root — ping từng phải setuid-root chỉ để mở raw socket; (2) quyền file không nói gì về *hành vi*: một process bị khai thác (RCE trong thư viện parse ảnh) được làm mọi thứ *user đó* làm được — đọc DB credential, mở kết nối ra ngoài, fork shell.

Ba lớp cơ chế của chương này thu hẹp dần: **Capability** (cắt root thành mảnh), **seccomp** (cắt bề mặt syscall), **LSM — SELinux/AppArmor** (chính sách bắt buộc toàn hệ thống, vượt trên ý muốn của chủ file). Cùng nhau, chúng hiện thực nguyên tắc **least privilege** — giả định *sẽ* bị xâm nhập và tối thiểu hóa thiệt hại (defense in depth).

## 2. Capability — cắt root thành ~40 mảnh

Kernel tách đặc quyền root thành các capability độc lập (`man 7 capabilities`): `CAP_NET_BIND_SERVICE` (bind port <1024), `CAP_NET_RAW`, `CAP_SYS_ADMIN` (cái "túi rác" quá rộng — gần bằng root, tránh cấp), `CAP_SYS_PTRACE`, `CAP_CHOWN`, `CAP_SETUID`... Mỗi process có các tập (effective/permitted/inheritable/bounding/ambient); file thực thi cũng gán được capability thay cho setuid-root:

```bash
# nginx bind port 80 mà KHÔNG chạy root:
setcap 'cap_net_bind_service=+ep' /usr/sbin/nginx
getpcaps <pid>            # xem process đang cầm gì
# Container: docker mặc định giữ ~14 capability; siết tiếp:
docker run --cap-drop ALL --cap-add NET_BIND_SERVICE myapp
```

Trade-off: mô hình nhiều tập + quy tắc thừa kế qua execve phức tạp đến mức chính nó thành nguồn bug cấu hình; và `CAP_SYS_ADMIN` phình to cho thấy việc "cắt mảnh" khó đến đâu khi API kernel không được thiết kế từ đầu cho nó. Nhưng với backend, quy tắc thực dụng đơn giản: **service không bao giờ chạy root; cấp đúng capability lẻ nếu thiếu; trong K8s drop ALL rồi add lại từng cái.**

## 3. seccomp — thu hẹp cánh cửa syscall

Ý tưởng: bề mặt tấn công kernel = ~450 syscall. Đa số service chỉ dùng 40-100. seccomp-BPF gắn một **filter (chương trình BPF cổ điển) vào process, chạy trước MỌI syscall**: cho qua / trả lỗi / giết process / báo user-space supervisor (`SECCOMP_RET_USER_NOTIF`). Một khi gắn (và bật `no_new_privs`), **không gỡ được, thừa kế qua fork/exec** — kẻ xâm nhập không tắt được.

```
syscall từ process ──► seccomp filter (trong kernel, ~vài chục ns)
   ├─ ALLOW  → chạy bình thường
   ├─ ERRNO  → trả -EPERM, không thực thi
   └─ KILL   → SIGSYS, chết ngay
```

Docker default profile chặn ~40 syscall nguy hiểm (`mount`, `reboot`, `init_module`, `kexec_load`, `ptrace` một phần, và — như đã gặp ở chương 10 — có giai đoạn cả `io_uring`); đây là lớp đã vô hiệu hóa nhiều CVE kernel *trước khi chúng được phát hiện*, đơn giản vì syscall lỗi nằm ngoài danh sách cho phép. Go minh họa mức khái niệm:

```go
// Hạ tầng thật dùng libseccomp/landlock; ý tưởng:
// 1. Khởi động, mở sẵn mọi file/socket cần thiết
// 2. prctl(PR_SET_NO_NEW_PRIVS, 1)
// 3. Nạp filter: chỉ cho {read, write, epoll_*, futex, mmap, close, exit...}
// → RCE sau điểm này không thể open file mới, không connect ra ngoài,
//   không execve /bin/sh — payload phổ biến chết từ trong trứng
```

Trade-off: profile quá chặt vỡ theo cách khó lường (libc đổi `open`→`openat2`, runtime mới thêm syscall lạ — app chết bí ẩn sau khi nâng cấp image); quá lỏng thì vô nghĩa. Vì vậy mặc định của Docker/K8s (`RuntimeDefault`) là điểm cân bằng tốt: **luôn bật nó** (K8s từng để trống mặc định — kiểm tra cluster của bạn), profile tùy chỉnh chỉ cho service giá trị cao.

## 4. LSM: SELinux và AppArmor — chính sách đứng trên cả chủ sở hữu

DAC (quyền file truyền thống) là "tùy nghi": chủ file cấp quyền được cho ai tùy ý — process bị chiếm thì "ý của chủ" cũng bị chiếm. **MAC** (Mandatory Access Control) thêm một tầng *bắt buộc*: chính sách toàn hệ thống mà root thường cũng không vượt qua. Kernel cài sẵn các **hook LSM** tại mọi điểm truy cập tài nguyên (mở file, connect socket, đọc /proc...); module LSM quyết định cho phép hay không — *sau khi* DAC đã cho phép (cả hai phải gật đầu).

- **SELinux** (RHEL/Fedora/Android): mọi process và object mang **label** (`httpd_t`, `postgresql_db_t`); chính sách là tập luật "domain nào được động verb gì lên type nào". Cực mạnh, mịn — và nổi tiếng khó: skill riêng, debug bằng `ausearch`/`audit2why`. Android là minh chứng giá trị: toàn bộ hệ sinh thái app bị nhốt bằng SELinux.
- **AppArmor** (Ubuntu/Debian/SUSE): profile theo **đường dẫn** cho từng chương trình — dễ đọc, dễ viết (`/var/lib/myapp/** rw`, `network tcp`, `deny /etc/shadow r`), kém mịn hơn. Có chế độ `complain` (chỉ log) để soạn profile từ hành vi thật.

Với container, LSM là lớp chặn **đường thoát ra host**: process thoát namespace vẫn đối mặt label/profile chặn nó đọc file host. Thực tế vận hành đau lòng nhất: sự cố "Permission denied bí ẩn" (volume K8s, log path lạ) → tra `dmesg`/`/var/log/audit/audit.log` **trước khi** `setenforce 0`. Tắt SELinux vì một lỗi permission là anti-pattern kinh điển — `audit2allow` sinh luật thiếu chính xác hơn nhưng vẫn tốt hơn tắt hẳn.

## 5. Trade-off & bức tranh xếp lớp

```
Kẻ tấn công RCE vào app trong container phải xuyên:
  seccomp (syscall bị chặn) → capability (không quyền đặc biệt)
  → user namespace (root giả) → LSM (label/profile chặn file host)
  → và vẫn còn: kernel phải có bug thì mới thoát hẳn
Mỗi lớp: rẻ (ns-µs mỗi lần kiểm tra), độc lập, hỏng lớp này còn lớp kia.
```

| Cơ chế | Trả lời câu hỏi | Chi phí runtime | Chi phí vận hành |
|---|---|---|---|
| Capability | "mảnh root nào?" | ~0 | thấp |
| seccomp | "syscall nào?" | vài chục ns/syscall | vừa (profile vỡ khi upgrade) |
| AppArmor | "file/net nào, theo path?" | thấp | vừa |
| SELinux | "ai chạm gì, theo label?" | thấp | cao |

## 6. Production & Anti-pattern

- Baseline hợp lý cho một backend đi production: non-root user trong image (`USER app`), `--cap-drop ALL` + add lẻ, seccomp RuntimeDefault, `readOnlyRootFilesystem: true`, `allowPrivilegeEscalation: false`, LSM của distro để nguyên chế độ enforce. Toàn bộ gói này gần như **miễn phí hiệu năng**.
- Anti-pattern hàng đầu: `setenforce 0`/`--privileged`/chạy root "để fix nhanh rồi quên"; mount docker.sock; cấp `CAP_SYS_ADMIN` thay vì tìm capability đúng; copy profile seccomp từ blog không kiểm chứng với runtime của mình.
- Audit định kỳ: `docker inspect` xem CapAdd/Privileged; K8s dùng Pod Security Standards (baseline/restricted) enforce ở namespace.

## 7. Failure Analysis — case mẫu: service chết ngay khi khởi động sau nâng cấp base image

- **Triệu chứng**: pod CrashLoopBackOff, log chỉ có một dòng "Bad system call".
- **Điều tra**: "Bad system call" = SIGSYS = seccomp giết. `strace` bản local tái hiện: glibc mới trong base image dùng syscall `clone3` — profile seccomp tùy chỉnh (viết 3 năm trước, whitelist) không có `clone3`.
- **Root cause**: whitelist tĩnh + userland tiến hóa. Kernel/libc thêm syscall mới là chuyện thường kỳ.
- **Fix**: thêm `clone3` (và rà các syscall mới: `openat2`, `faccessat2`, `rseq`...) vào profile; hoặc quay về RuntimeDefault (được duy trì bởi cộng đồng đúng cho việc này). **Prevention**: CI test image mới với đúng profile production; theo dõi release note của runtime/libc; ưu tiên profile mặc định được bảo trì thay vì tự chế trừ khi có lý do cụ thể.

## 8. Khi nào không nên tối ưu (siết)

Bảo mật cũng có premature optimization: viết SELinux policy tùy chỉnh cho service nội bộ ít giá trị, seccomp whitelist tự chế cho mọi pod — chi phí bảo trì vượt xa rủi ro giảm được, và profile vỡ âm thầm làm team *mất niềm tin* rồi tắt hết (kết cục tệ hơn điểm xuất phát). Ưu tiên theo thứ tự giá trị/chi phí: non-root → drop capabilities → seccomp mặc định → LSM mặc định của distro → (chỉ khi threat model đòi hỏi) profile tùy chỉnh và sandbox mạnh (gVisor/Kata) cho code không tin cậy.

---

**Phần tiếp theo — Level 5**: toàn bộ kiến thức 14 chương được đưa vào thực chiến: [Chương 15](/series/linux-os-for-backend/15-failure-cases-cpu-memory/) & [Chương 16](/series/linux-os-for-backend/16-failure-cases-io-network-concurrency/) — 22 production failure case, và [Chương 17](/series/linux-os-for-backend/17-kien-truc-thuc-te/) — Linux dưới các hệ thống bạn chạy hằng ngày.
