+++
title = "Chương 07 — Memory Management: ảo ảnh lớn nhất và đắt giá nhất"
date = "2026-02-21T14:00:00+07:00"
draft = false
tags = ["backend", "linux", "operating-system"]
series = ["Operating Systems cho Backend Engineer"]
+++

## 1. Problem Statement

RAM là một mảng byte vật lý hữu hạn, phẳng, ai cũng thấy như nhau. Ta cần: (1) mỗi process một không gian địa chỉ **riêng, cô lập**; (2) tổng memory các process *tưởng* mình có được phép **vượt** RAM thật; (3) cấp phát/thu hồi linh hoạt không phân mảnh chết; (4) tận dụng RAM thừa làm **cache** cho disk. Virtual memory giải cả bốn — với cái giá là mọi truy cập memory đều phải qua một tầng dịch địa chỉ.

Nếu không có virtual memory: mọi chương trình phải biên dịch theo địa chỉ vật lý cố định (không thể chạy 2 bản), một con trỏ lỗi phá cả máy, không có COW, không có mmap, không có container density như ngày nay.

## 2. Tại sao nó tồn tại

- **Security/Isolation**: cô lập bằng phần cứng (MMU) — nền của mọi thứ khác.
- **Hardware limitation**: RAM đắt và hữu hạn; địa chỉ ảo cho phép "hứa trước, cấp sau" (lazy allocation) và "gom sau" (swap, reclaim).
- **Business**: density — nhét 50 container vào một máy được vì mỗi cái chỉ *thật sự* dùng phần nó chạm tới.
- **Performance**: RAM thừa tự động thành Page Cache — lý do "Linux ăn hết RAM" là feature, không phải bug.

## 3. Internal Architecture

### 3.1. Page và Page Table

Memory chia thành **page 4KB**. Địa chỉ ảo → địa chỉ vật lý dịch qua **page table** — trên x86-64 là cây 4 tầng (PGD→PUD→PMD→PTE; 5 tầng với địa chỉ 57-bit):

```
Địa chỉ ảo 48-bit: [9 bit PGD][9 bit PUD][9 bit PMD][9 bit PTE][12 bit offset]
                        │          │          │         │
CR3 ──► bảng PGD ──► bảng PUD ──► bảng PMD ──► bảng PTE ──► physical page + offset

Mỗi PTE (8 byte) chứa: số hiệu physical frame + bit cờ:
  P (present) | R/W | U/S (user/supervisor) | A (accessed) | D (dirty) | NX (no-execute)
```

Vì sao cây nhiều tầng thay vì bảng phẳng? Bảng phẳng cho 48-bit địa chỉ cần 2^36 entry × 8B = nửa TB *mỗi process*. Cây thì thưa: vùng nào không dùng, cả nhánh không tồn tại. Trade-off: mỗi lần dịch địa chỉ cần **4 lần đọc memory** — không thể chấp nhận → sinh ra TLB.

### 3.2. MMU và TLB

**MMU** là phần cứng trong CPU thực hiện việc dịch trên *mọi* truy cập memory. **TLB** là cache của kết quả dịch (~1500-2000 entry L2 TLB):

```
CPU truy cập địa chỉ ảo V
  ├── TLB hit (99%+): +0-1 cycle, coi như miễn phí
  └── TLB miss: page walk 4 tầng (~20-100+ cycle)
        ├── PTE present: nạp vào TLB, đi tiếp
        └── PTE không present/không quyền: PAGE FAULT → trap vào kernel
```

Sức chứa TLB: ~2000 entry × 4KB = **~8MB** — trong khi heap service của bạn là hàng GB. Workload truy cập ngẫu nhiên trên working set lớn sẽ TLB miss liên tục — đây chính là chỗ **Huge Page** vào cuộc: page 2MB → một entry TLB phủ 512 lần nhiều hơn (~4GB với 2000 entry), và cây page table bớt một tầng.

- **Explicit huge page** (`hugetlbfs`, cấp trước, không swap được): PostgreSQL, DPDK, JVM lớn dùng — kiểm soát được, hiệu quả đo được (giảm 20-50% TLB miss với database buffer pool lớn).
- **Transparent Huge Page (THP)**: kernel tự gom 4KB→2MB sau lưng. Với workload latency-sensitive từng gây tai tiếng: `khugepaged` compact memory gây khựng, COW 2MB khuếch đại (Redis fork!), phân mảnh. Khuyến nghị phổ biến cho database: `madvise` hoặc `never` (chương 17).

### 3.3. Bốn loại Page Fault — phải phân biệt được

| Loại | Nguyên nhân | Chi phí | Bản chất |
|---|---|---|---|
| **Minor fault** | Page hợp lệ nhưng chưa map (lazy alloc, COW, page đã có trong cache) | ~0.5-2µs | Vận hành bình thường |
| **Major fault** | Phải đọc **disk** (swap in, mmap file chưa cache) | ~0.1-10ms | Tín hiệu cần chú ý |
| SIGSEGV | Không có VMA / sai quyền | — | Bug |
| SIGBUS | mmap file bị truncate... | — | Bug/race |

`ps -o min_flt,maj_flt`, `sar -B`, `perf stat -e page-faults,major-faults`. **Major fault rate cao = memory đang thiếu hoặc mmap IO** — một trong các chỉ số cảnh báo sớm tốt nhất cho swap thrashing.

### 3.4. malloc không phải syscall — hai tầng cấp phát

```
Ứng dụng: malloc(64B) / new / go runtime alloc
   ↓ (99.9% trường hợp: HOÀN TOÀN trong user space, lấy từ pool có sẵn — ns)
Allocator (glibc ptmalloc / jemalloc / tcmalloc / Go runtime)
   ↓ (chỉ khi pool cạn)
Syscall: brk (heap cổ điển) hoặc mmap(MAP_ANONYMOUS) (vùng lớn)
   ↓ kernel chỉ tạo/mở rộng VMA — CHƯA CẤP PAGE NÀO (lazy!)
Chạm vào page lần đầu → minor fault → lúc này mới cấp physical page
```

Hệ quả quan trọng:

- **VSZ vs RSS**: VSZ = tổng VMA (lời hứa), RSS = số page thật đang map. VSZ 30GB + RSS 2GB là bình thường (Go, JVM reserve lớn). **Theo dõi RSS (và cgroup usage), đừng hoảng vì VSZ.**
- **Overcommit**: mặc định (`vm.overcommit_memory=0`) kernel hứa nhiều hơn RAM+swap theo heuristic. Cái giá: khi mọi người cùng đòi nợ → không đủ → **OOM Killer**. Chọn thay thế (`=2`, không overcommit): malloc fail sớm, an toàn nhưng lãng phí và nhiều app không xử lý nổi malloc fail. Redis khuyên `=1` để fork/BGSAVE không bị từ chối oan.
- Allocator trả memory lại kernel dè dặt (`MADV_FREE`/`MADV_DONTNEED`) → RSS không giảm ngay sau khi app free — không phải leak.

### 3.5. Page Cache, Reclaim, Swap, OOM — chuỗi phòng thủ khi thiếu memory

```
RAM đầy dần
  │
  ▼ (còn "free" thấp hơn watermark low)
kswapd thức dậy, reclaim NỀN theo LRU (2 danh sách active/inactive, xấp xỉ LRU):
  1. Page Cache sạch  → vứt (đọc lại từ disk khi cần) ← rẻ nhất
  2. Page Cache bẩn   → ghi xuống disk rồi vứt
  3. Anonymous page (heap) → chỉ vứt được nếu có SWAP (ghi ra swap)
  │
  ▼ (cấp phát nhanh hơn reclaim; chạm watermark min)
DIRECT RECLAIM: chính process xin memory phải TỰ ĐI DỌN — latency spike
  │            ← đây là thủ phạm số 1 của "khựng" khi memory căng
  ▼ (vẫn không đủ)
OOM KILLER: chọn nạn nhân theo oom_score (≈ %memory dùng, hiệu chỉnh bởi
  oom_score_adj −1000..+1000) → SIGKILL. Log trong dmesg.
```

`vm.swappiness` (0-200, mặc định 60) điều chỉnh cán cân bước 2 vs 3 — thiên về vứt cache hay swap heap. **Swap không phải kẻ xấu**: một ít swap cho anonymous page nguội (init code, buffer không dùng) giải phóng RAM cho cache hữu ích. Kẻ xấu là **thrashing**: working set thật > RAM → swap in/out liên tục → hệ thống tê liệt (case chương 15). Không swap cũng có mặt tối: mất memory đệm đàn hồi, kernel buộc vứt page cache *đang cần* (kể cả text của chính executable — "swap thrashing không có swap"), rồi OOM đột ngột. PSI memory là công cụ hiện đại nhất để thấy sức ép: `/proc/pressure/memory`.

## 4. Cách hoạt động — theo dấu một lần ghi vào page COW (sau fork)

```
Redis (đã fork BGSAVE), parent SET key... → ghi vào page P (đang read-only vì COW)
  ↓ MMU: PTE cấm ghi → PAGE FAULT (exception, chương 03)
kernel: handle_mm_fault (mm/memory.c)
  1. Tìm VMA chứa địa chỉ → hợp lệ, quyền ghi OK → là COW fault
  2. Cấp physical page mới, copy 4KB (hoặc 2MB nếu THP!)
  3. Sửa PTE của parent trỏ page mới, bật quyền ghi
  4. Flush entry TLB tương ứng (nếu nhiều CPU từng chạm: TLB SHOOTDOWN —
     gửi IPI tới các core, mỗi lần ~vài µs — chi phí ẩn của multi-thread + COW)
  5. Quay lại user space, lệnh ghi CHẠY LẠI, lần này thành công
Ứng dụng không biết gì — chỉ thấy lệnh ghi "hơi chậm" (µs thay vì ns)
```

Nhân câu chuyện này lên hàng trăm nghìn page khi Redis nhận write dồn dập trong lúc BGSAVE → hiểu ngay vì sao memory phình và latency gợn sóng. Đây là ví dụ hoàn hảo về **ảo ảnh rò rỉ**: "ghi một biến" tưởng là thao tác ns, có thể là chuỗi fault–copy–shootdown µs.

## 5. Trade-off

| Cơ chế | Được | Mất |
|---|---|---|
| Virtual memory + MMU | Cô lập, COW, mmap, overcommit | Mọi truy cập qua dịch địa chỉ; TLB miss; page table tốn RAM |
| Lazy allocation | Cấp nhanh, density cao | Chi phí dời về lúc chạm (fault storm khi khởi động); OOM thay vì malloc fail |
| Page 4KB | Linh hoạt, ít lãng phí trong page | TLB phủ ít; cây page table sâu |
| Huge page | TLB phủ nhiều, ít fault | Phân mảnh; THP gây khựng; COW khuếch đại |
| Page Cache tham lam | IO nhanh "miễn phí" | Con số "free" gây hiểu lầm; reclaim gây latency khi căng |
| Overcommit | Fork/reserve lớn không fail oan | OOM Killer — chết đột tử thay vì lỗi rõ ràng |
| Swap | Đệm đàn hồi, đuổi page nguội | Thrashing khi working set vượt RAM; latency khó đoán |

## 6. Production

- **Đọc `free -m` cho đúng**: cột `available` (ước lượng cấp được không gây swap) mới là "còn bao nhiêu", không phải `free`. `buff/cache` là RAM hữu ích, thu hồi được (phần lớn).
- Dashboard tối thiểu: RSS per service, cgroup `memory.current` vs `memory.max`, major fault rate, swap in/out (`vmstat` cột si/so), PSI memory (some/full), OOM kill count (dmesg/`node_vmstat_oom_kill`).
- `smem`/`/proc/<pid>/smaps_rollup`: **PSS** (chia đều phần share) là con số cộng-được duy nhất khi nhiều process share memory — RSS cộng dồn sẽ đếm trùng thư viện và Page Cache mmap.
- Tuning điển hình theo workload (chi tiết chương 17): database tự quản cache → hạ swappiness (1-10), tắt/madvise THP, cân nhắc explicit huge page cho buffer pool; hộp trộn nhiều service → để mặc định, kiểm soát bằng cgroup `memory.max` + `memory.low`.
- `oom_score_adj`: bảo vệ process chính (−500), hy sinh sidecar (+500) — điều khiển được OOM Killer chọn ai thay vì cầu nguyện. systemd: `OOMScoreAdjust=`; K8s đặt theo QoS class.

## 7. Anti-pattern

- Thấy `free` thấp → mua RAM / restart service. (Đọc nhầm Page Cache.)
- `echo 3 > /proc/sys/vm/drop_caches` định kỳ "cho nhẹ máy" — vứt cache hữu ích, IO tăng vọt. Chỉ dùng khi benchmark.
- Tắt swap mọi nơi theo giáo điều rồi ngạc nhiên vì OOM sớm và IO đọc lại executable liên tục khi memory căng.
- Để THP `always` cho Redis/Mongo/hệ latency-sensitive.
- Đo "memory dùng" bằng VSZ; hoặc alert RSS mà quên rằng cgroup tính cả page cache của container vào `memory.current` (K8s: nạn nhân OOMKilled dù "heap còn nhỏ" — thủ phạm là ghi file dồn dập tạo dirty page cache trong cgroup).

## 8. Failure Analysis — case mẫu: OOM Killer bắn database lúc 3 giờ sáng

- **Triệu chứng**: postgres chết không log gì; `dmesg`: `Out of memory: Killed process 812 (postgres) total-vm:..., anon-rss:18GB`.
- **Điều tra**: đêm đó có job phân tích chạy cùng máy, RSS 20GB; máy 64GB không swap; postgres shared_buffers 16GB + work_mem × connections spike do batch query.
- **Kernel view**: cấp phát mới không thỏa được ở mọi mức reclaim → `out_of_memory()` → tính `oom_badness()` cho mọi task → postgres RSS lớn nhất → SIGKILL. Kernel *đúng logic* nhưng *sai nạn nhân* theo góc nhìn kinh doanh.
- **Fix ngắn hạn**: `oom_score_adj=-800` cho postgres; tách job phân tích sang cgroup `memory.max=24G`. **Dài hạn**: không đặt workload không kiểm soát cạnh database; alert PSI memory + `available` < 10%; cân nhắc swap nhỏ + `memory.low` bảo vệ postgres.
- **Prevention**: quy hoạch memory từng service bằng cgroup (systemd slice / K8s requests-limits) — đừng để OOM Killer là người quy hoạch.

## 9. Khi nào không nên tối ưu

Khi `available` dồi dào, không major fault, PSI memory ~0 — mọi tinh chỉnh swappiness/THP/hugepage đều không tạo khác biệt đo được; hãy để mặc định cho dễ vận hành. Huge page chỉ đáng khi working set hàng chục GB truy cập ngẫu nhiên và bạn *đo* được TLB miss (perf `dTLB-load-misses`) chiếm phần đáng kể. Tối ưu allocator (đổi jemalloc/tcmalloc) chỉ sau khi profiling chỉ ra allocator là hotspot — với đa số service, GC tuning của runtime quan trọng hơn nhiều.

---

**Chương tiếp theo**: nhiều thread chung một address space thì phải giải quyết tranh chấp — [Chương 08: Synchronization](/series/linux-os-for-backend/08-synchronization/).
