+++
title = "Chương 09 — Filesystem: biến mảng block thô thành dữ liệu bền vững"
date = "2026-02-21T16:00:00+07:00"
draft = false
tags = ["backend", "linux", "operating-system"]
series = ["Operating Systems cho Backend Engineer"]
+++

## 1. Problem Statement

Disk (NVMe/SSD/HDD) là một mảng block 512B/4KB đánh số từ 0 đến N. Không tên file, không thư mục, không quyền, không đảm bảo gì khi mất điện giữa chừng. Filesystem phải dựng trên đó: **không gian tên phân cấp, quyền sở hữu, cấp phát không phân mảnh, và quan trọng nhất — tính toàn vẹn khi crash** (crash consistency). Cộng thêm yêu cầu hiệu năng: disk chậm hơn RAM 100-1000 lần → phải cache quyết liệt — và cache chính là nơi sinh ra khoảng cách giữa "đã ghi" và "đã bền vững", khoảng cách gây ra nhiều sự cố mất dữ liệu nhất trong nghề backend.

## 2. Tại sao nó tồn tại

- **Hardware limitation**: block device không có khái niệm gì ngoài "đọc/ghi block i". Mất điện giữa một thao tác nhiều block → trạng thái nửa vời.
- **Business**: dữ liệu là tài sản; "database bị corrupt sau sự cố điện" là sự kiện đe dọa doanh nghiệp.
- **Performance**: độ trễ và thông lượng disk chỉ chấp nhận được nhờ cache + ghi gộp + đọc trước.
- **Đa dạng**: 60+ filesystem (ext4, XFS, Btrfs, NFS, tmpfs, overlayfs, /proc...) phải hiện ra với ứng dụng qua *một* API — sinh ra VFS.

## 3. Internal Architecture

### 3.1. VFS — lớp trừu tượng thống nhất

```
                    open/read/write/fsync/stat (syscall)
                                 │
                    ┌────────────▼────────────┐
                    │           VFS           │  các object chung:
                    │  superblock  inode      │  - inode: "file là gì"
                    │  dentry      file       │  - dentry: "tên → inode" (+cache)
                    └──┬──────┬──────┬────────┘  - file: "phiên mở" (offset, flags)
                       │      │      │
                    ext4    XFS    tmpfs   nfs   overlayfs   /proc
                       │      │
                  Page Cache (chung, chương 07)
                       │
                  Block layer + IO scheduler (mq-deadline/none/bfq)
                       │
                  NVMe/SSD driver → DMA → thiết bị
```

VFS là polymorphism trong C: mỗi FS cung cấp bảng function pointer (`inode_operations`, `file_operations`...). Nhờ nó, `cat` đọc được từ ext4, NFS lẫn `/proc/cpuinfo` (file "ảo" — nội dung sinh ra khi đọc) mà không biết khác biệt. **Dentry cache** giữ kết quả phân giải đường dẫn trong RAM — path lookup là thao tác nóng bậc nhất (mỗi `open` phải walk từng thành phần đường dẫn, check quyền từng cấp).

### 3.2. Inode — danh tính thật của file

```
inode #5234 (ext4):
  type, mode (rwxr-x---), uid, gid, size, timestamps (atime/mtime/ctime)
  link count (số tên trỏ tới)
  CON TRỎ TỚI DATA BLOCK:
    ext4: extent tree — "block logic 0-32767 nằm ở block vật lý 100000-132767"
          (thay vì map từng block như ext2/3 — file lớn gọn hơn hàng nghìn lần)
  KHÔNG CHỨA: tên file!
```

Tên file sống trong **directory** — thực chất là file đặc biệt chứa các cặp (tên → số inode). Suy ra chuỗi hệ quả mà backend engineer dùng hằng ngày:

- **Hard link** = nhiều tên trỏ một inode. Xóa file (`unlink`) = xóa *tên*, giảm link count; **dữ liệu chỉ mất khi count=0 VÀ không process nào đang mở**. → Xóa file log 50GB mà disk không giảm: process vẫn mở FD → "deleted but open" — tìm bằng `lsof +L1`. Fix: reload/restart process giữ FD, hoặc `truncate` thay vì xóa.
- **Rename trong cùng FS là atomic** (đổi entry trong directory) → pattern chuẩn "ghi file mới → fsync → rename đè" cho config/checkpoint an toàn crash.
- **inode là tài nguyên hữu hạn** (ext4 cấp cố định lúc format): hàng chục triệu file nhỏ → cạn inode khi disk còn trống 40% — `df -i` (case chương 16).

### 3.3. Page Cache và writeback — khoảng cách "ghi rồi" vs "bền rồi"

```
write(fd, buf, 4KB)
  → copy vào Page Cache, đánh dấu DIRTY → TRẢ VỀ NGAY (~vài µs)
     "thành công" ở đây nghĩa là: DỮ LIỆU ĐANG Ở RAM. Mất điện = mất.
  → writeback flusher ghi xuống disk SAU (trong vòng ~30s: dirty_expire_centisecs,
     hoặc khi dirty vượt dirty_background_ratio ~10% RAM)
  → dirty vượt dirty_ratio (~20% RAM): process GHI BỊ CHẶN để throttle
     ← nguồn của "app khựng khi copy file lớn"

fsync(fd) = ép ghi mọi dirty page của file + journal commit + FLUSH CACHE
            của chính thiết bị (disk cũng có RAM cache!) → chờ xong mới về
            → ĐẮT: 0.1-1ms (NVMe) đến ~10ms (HDD)
```

Đây là trade-off trung tâm của mọi hệ lưu trữ: **an toàn (fsync mỗi lần ghi) vs nhanh (để writeback lo)**. Mọi database đặt mình trên thang này: PostgreSQL WAL fsync mỗi commit (`synchronous_commit` cho phép nới); Redis AOF `everysec` (mất tối đa 1 giây); Kafka mặc định *không* fsync mỗi message — dựa vào replication (an toàn bằng nhiều máy thay vì nhiều lần fsync). Hiểu tầng này mới hiểu config độ bền của từng hệ (chương 17).

Còn một bẫy tinh vi: fsync file mới tạo chưa đủ — phải **fsync cả thư mục cha** để entry tên file bền vững. Và bài học "fsyncgate" của PostgreSQL: khi fsync trả lỗi (EIO), dirty page có thể đã bị vứt — retry fsync thấy "OK" nhưng dữ liệu ĐÃ MẤT; phản ứng đúng là coi như hỏng và phục hồi từ WAL (Postgres giờ PANIC khi fsync lỗi).

### 3.4. Journal — sống sót qua mất điện

Một thao tác "tạo file" chạm nhiều block (inode bitmap, inode, directory, data). Mất điện giữa chừng → FS mâu thuẫn. **Journaling** (ext4/XFS): ghi *ý định* vào vùng log tuần tự trước, commit, rồi mới sửa vị trí thật; crash → replay journal. Ba mức của ext4:

- `journal`: cả metadata + data vào journal — an toàn nhất, ghi mọi thứ 2 lần, chậm.
- **`ordered` (mặc định)**: chỉ metadata vào journal, nhưng data ghi *trước* khi commit metadata — không bao giờ thấy file trỏ vào rác; không bảo vệ nội dung update giữa chừng.
- `writeback`: metadata journal, data tùy ý — nhanh nhất, sau crash file có thể chứa rác cũ.

XFS (chỉ metadata journal, thiết kế song song hóa cao — nhiều allocation group hoạt động độc lập) là mặc định RHEL, mạnh với file lớn, nhiều luồng ghi song song, ít khựng hơn ở tail — lý do nhiều hướng dẫn Kafka/PostgreSQL nghiêng về XFS. ext4 phổ biến, hồi phục dễ, hành vi được hiểu rõ. Khác biệt thật giữa hai bên với backend hiện đại: **nhỏ hơn lời đồn** — cấu hình fsync của *ứng dụng* quan trọng hơn chọn FS.

## 4. Cách hoạt động — theo dấu `open` + `write` + `fsync`

```
fd = open("/data/wal/000042.log", O_WRONLY|O_APPEND)
  → path walk qua dentry cache: "/" → "data" → "wal" → "000042.log"
    (miss cache cấp nào → đọc directory block từ disk cấp đó)
  → check quyền từng cấp → lấy inode → tạo struct file (offset, flags)
  → gắn vào bảng FD của process → trả số fd

n = write(fd, buf, 8192)
  → VFS → ext4: xác định block logic → extent tree → copy vào 2 page
    trong Page Cache (radix tree của inode) → mark dirty → return

fsync(fd)
  → ext4_sync_file: ghi các dirty page (block layer, DMA xuống NVMe)
  → chờ journal commit transaction chứa metadata liên quan
  → gửi lệnh FLUSH xuống thiết bị (đẩy cache nội bộ của disk ra vùng bền)
  → hoàn tất — DỮ LIỆU BỀN VỮNG từ điểm này, kể cả mất điện
```

Minh họa Go — pattern ghi an toàn:

```go
f, _ := os.CreateTemp(dir, "cfg-*")         // ghi file tạm cùng thư mục
f.Write(data)
f.Sync()                                     // fsync file
f.Close()
os.Rename(f.Name(), path)                    // rename atomic đè file cũ
d, _ := os.Open(dir); d.Sync(); d.Close()   // fsync thư mục — bước hay bị quên
```

## 5. Trade-off

| Quyết định | Được | Mất |
|---|---|---|
| Page Cache write-back | write ~µs, gộp IO, hấp thụ burst | "Đã ghi" ≠ "đã bền"; mất điện mất ~30s dữ liệu nếu không fsync |
| Journal (ordered) | Crash không nát FS | Ghi metadata 2 lần; fsync phải chờ journal |
| VFS trừu tượng | Một API cho 60 FS | Che mất đặc tính riêng (atomicity, giới hạn) — dev tưởng mọi FS như nhau |
| Tên tách khỏi inode | Hard link, rename atomic, xóa an toàn khi đang mở | "Deleted but open" gây đầy disk khó hiểu |
| inode cấp trước (ext4) | Đơn giản, nhanh | Cạn inode với hàng chục triệu file nhỏ |
| Delayed allocation | Chọn chỗ đặt block tốt hơn, ít phân mảnh | Kéo dài cửa sổ mất dữ liệu nếu app không fsync |

## 6. Production

- Độ trễ IO thật: `iostat -x 1` — `r_await/w_await` (ms), `aqu-sz` (hàng đợi), `%util` (lưu ý: với NVMe song song, %util=100 **không** có nghĩa bão hòa — nhìn await + aqu-sz). Phân bố độ trễ: eBPF `biolatency` (histogram — thấy tail mà iostat trung bình che mất).
- Ai gây IO: `iotop`, eBPF `biosnoop` (từng IO: process, LBA, size, latency), `filetop` (mức file).
- Page cache hiệu quả không: `cachestat` (hit ratio), `free` (buff/cache), PSI io.
- Dirty page đang dồn: `grep -E 'Dirty|Writeback' /proc/meminfo` — Dirty hàng GB kèm app khựng từng đợt = writeback storm; cân nhắc hạ `vm.dirty_background_bytes` (bắt đầu ghi sớm, đều) — đặc biệt máy RAM lớn (10% của 512GB = 51GB dirty là thảm họa latency).
- Mount option thực dụng: `noatime` (đỡ ghi metadata mỗi lần *đọc*); với DB xem khuyến nghị vendor. Scheduler: NVMe → `none`; SSD SATA → `mq-deadline` (mặc định thường đã đúng).

## 7. Anti-pattern

- Tin `write()` thành công = dữ liệu an toàn (không fsync, không hiểu tầng nào đang cache).
- Ghi trực tiếp đè file config — crash giữa chừng = file nửa vời; luôn dùng temp+rename.
- Hàng triệu file nhỏ trong một thư mục phẳng (path lookup + readdir chậm, cạn inode) — chia bucket 2 cấp theo hash, hoặc dùng database/object store đúng nghĩa.
- `O_DIRECT` "cho nhanh" mà không hiểu: mất page cache, phải tự align, tự quản cache — chỉ database engine tự quản buffer mới hưởng lợi.
- Log không rotate + process giữ FD → deleted-but-open chiếm nửa disk.
- Chạy database trên NFS/overlay layer của container (fsync semantics khác, độ trễ khó lường) — volume thật cho dữ liệu thật.

## 8. Failure Analysis — case mẫu: checkout chậm mỗi 30 giây một nhịp

- **Triệu chứng**: API latency bình thường 20ms, cứ ~30s có cụm request 2-4s. Máy chạy cả app lẫn job xuất báo cáo ghi file lớn.
- **Điều tra**: `biolatency` thấy tail w_await nhảy theo chu kỳ; `/proc/meminfo` Dirty dồn ~6GB rồi tụt sạch — đúng nhịp 30s (dirty_expire). `biosnoop`: chuỗi write khổng lồ từ flusher + fsync WAL của DB xếp sau.
- **Root cause**: job báo cáo xả 6GB vào page cache; writeback định kỳ chiếm băng thông disk; fsync của DB (cần bền vững *ngay*) kẹt sau hàng đợi ghi nền.
- **Fix**: job báo cáo ghi qua `O_DIRECT` hoặc cgroup io.max giới hạn băng thông; hạ `vm.dirty_background_bytes` còn 256MB cho writeback chảy đều; tách disk cho WAL. p99 ổn định lại.
- **Prevention**: mọi batch ghi lớn phải bị giới hạn io (cgroup v2 io.max/io.latency); dashboard Dirty bytes + biolatency tail; không đặt WAL chung disk với workload xả lũ.

## 9. Khi nào không nên tối ưu

Nếu `iostat` await thấp, hit ratio page cache cao, PSI io ~0 — chuyển FS, đổi mount option hay mua NVMe nhanh hơn đều không cải thiện gì đo được. Tối ưu tầng file khi và chỉ khi: fsync nằm trên đường latency của request (→ xem lại kiến trúc: batch commit, group commit, WAL riêng disk), hoặc IO ngẫu nhiên nhỏ chiếm ưu thế (→ xem lại access pattern trước khi đổ tiền phần cứng). Đa số service web: bottleneck là network + DB ở máy khác, filesystem local chỉ chứa log.

---

**Chương tiếp theo**: đọc/ghi đã hiểu — giờ đến câu hỏi *chờ* dữ liệu thế nào cho 10.000 connection: [Chương 10: IO Models](/series/linux-os-for-backend/10-io-models/).
