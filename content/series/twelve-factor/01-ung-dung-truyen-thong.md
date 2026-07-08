+++
title = "Chương 1: Ứng dụng truyền thống và những giới hạn của nó"
date = "2026-07-10T02:00:00+07:00"
draft = false
tags = ["backend", "twelve-factor", "cloud-native"]
series = ["The Twelve-Factor App"]
+++

> **Phần 1 – Foundation** | Chương trước: [Mục lục](/series/twelve-factor/00-muc-luc/) | Chương sau: [Cloud Computing](/series/twelve-factor/02-cloud-computing/)

Trước khi nói về 12-Factor App, chúng ta phải hiểu **thế giới mà nó sinh ra để giải quyết**. Nếu bạn chưa từng trải qua nỗi đau của việc vận hành một ứng dụng truyền thống, mọi nguyên lý của 12-Factor sẽ chỉ là một checklist vô hồn. Chương này tái hiện lại thế giới đó.

---

## 1. Problem Statement

### Một ứng dụng truyền thống trông như thế nào?

Hãy hình dung một hệ thống rất phổ biến ở các doanh nghiệp Việt Nam giai đoạn 2005–2015 (và thực tế vẫn còn tồn tại rất nhiều đến hôm nay):

```
┌─────────────────────────────────────────────────────┐
│                  Server vật lý / VM                  │
│  ┌───────────────────────────────────────────────┐  │
│  │  App (WAR file / binary / PHP source)          │  │
│  │  - Config hardcode trong source               │  │
│  │  - Session lưu trong memory                   │  │
│  │  - File upload ghi vào /var/www/uploads       │  │
│  │  - Log ghi vào /var/log/app/app.log           │  │
│  └───────────────────────────────────────────────┘  │
│  ┌───────────────┐  ┌───────────────┐               │
│  │  MySQL         │  │  Cron jobs    │               │
│  │  (cùng máy)    │  │  (cùng máy)   │               │
│  └───────────────┘  └───────────────┘               │
│                                                      │
│  Được "chăm sóc" bởi anh SysAdmin tên Tuấn,          │
│  người duy nhất biết server này cấu hình ra sao.     │
└─────────────────────────────────────────────────────┘
```

Quy trình deploy điển hình:

1. Developer build trên máy cá nhân, tạo ra file `app.war` hoặc binary.
2. Copy file qua FTP/SCP lên server.
3. SSH vào server, dừng service, thay file, sửa tay file config, khởi động lại.
4. "Test nhanh" trên production bằng cách bấm F5.
5. Nếu lỗi, sửa trực tiếp trên server, hoặc copy lại bản cũ (nếu còn giữ).

Cách làm này **hoạt động được**. Đó là điều quan trọng cần thừa nhận: hàng nghìn doanh nghiệp đã vận hành như vậy trong nhiều năm. Vấn đề không phải là nó không chạy — vấn đề là **nó không chịu được sự thay đổi**: thay đổi về quy mô người dùng, tần suất release, quy mô team, và hạ tầng.

### Nếu giữ nguyên cách làm này thì điều gì xảy ra?

Từng vấn đề sẽ được mổ xẻ ở mục 2, nhưng tóm tắt trước bức tranh:

| Sự thay đổi | Điều xảy ra với ứng dụng truyền thống |
|---|---|
| Traffic tăng 10 lần | Chỉ có một cách: mua server to hơn (scale vertical). Đến một lúc không còn server nào to hơn để mua. |
| Release từ 1 lần/quý → 10 lần/ngày | Deploy thủ công không theo kịp. Mỗi lần deploy là một lần "nín thở". |
| Team từ 3 → 30 engineer | Không ai dám đụng vào server vì "chỉ anh Tuấn biết nó cấu hình thế nào". |
| Server chết lúc 2h sáng | Dựng lại server mất nhiều giờ (hoặc nhiều ngày) vì cấu hình nằm trong đầu người, không nằm trong code. |

---

## 2. Tại sao cách làm truyền thống trở thành vấn đề

### 2.1. Business Problem — Tốc độ release là lợi thế cạnh tranh

Khoảng 2010 trở về trước, phần mềm release theo quý hoặc theo năm. Deploy chậm, thủ công, rủi ro — nhưng chấp nhận được, vì đối thủ của bạn cũng vậy.

Điều thay đổi cuộc chơi: các công ty Internet (Amazon, Google, Facebook, sau này là Shopee, Grab, MoMo ở thị trường Việt Nam) chứng minh rằng **release nhanh = học từ thị trường nhanh = thắng**. Amazon từ 2011 đã deploy trung bình mỗi 11.6 giây một lần. Khi đối thủ của bạn ship tính năng mỗi ngày còn bạn ship mỗi quý, đó không còn là vấn đề kỹ thuật — đó là vấn đề sống còn của business.

Ứng dụng truyền thống **không thể** release nhanh vì mỗi lần deploy là một chuỗi thao tác thủ công, phụ thuộc vào con người, và không thể rollback một cách tin cậy.

### 2.2. Operational Problem — Snowflake Server và Configuration Drift

Server truyền thống là một **snowflake server** — "bông tuyết" vì không có hai server nào giống hệt nhau:

- Anh Tuấn cài JDK 8u151 trên server A năm 2017. Năm 2019, một bạn khác cài JDK 8u231 trên server B. Không ai ghi lại.
- Ai đó SSH vào sửa `sysctl.conf` để fix sự cố lúc nửa đêm, không commit vào đâu cả.
- Sau 5 năm, server production là kết quả tích lũy của hàng trăm thao tác tay không được ghi chép. Hiện tượng này gọi là **configuration drift**.

Hệ quả trực tiếp:

- **Không thể dựng lại server**. Nếu server chết, không ai biết chính xác cần cài gì để có lại môi trường cũ. Disaster recovery trở thành canh bạc.
- **"Works on my machine"**. Môi trường dev, staging, production khác nhau về OS, version thư viện, config. Bug chỉ xuất hiện trên production và không tái hiện được ở nơi khác.
- **Fear-driven operations**: không ai dám nâng cấp, không ai dám restart, vì không ai chắc điều gì sẽ xảy ra.

### 2.3. Deployment Problem — Deploy là một sự kiện, không phải một thói quen

Trong mô hình truyền thống, deploy có các đặc điểm:

- **Không lặp lại được (non-reproducible)**: build trên máy dev, kết quả phụ thuộc vào máy dev đó có gì.
- **Không rollback được một cách đáng tin**: "bản cũ" là một file ai đó *có thể* còn giữ đâu đó.
- **Downtime là mặc định**: dừng service → thay file → khởi động. Người dùng nhìn thấy trang lỗi.
- **Con người là một phần của quy trình**: và con người thì mệt mỏi, quên bước, gõ nhầm — đặc biệt lúc 2 giờ sáng.

Theo các báo cáo DORA (DevOps Research and Assessment) qua nhiều năm, nhóm "low performer" — thường gắn với quy trình deploy thủ công — có **change failure rate cao gấp nhiều lần** và **thời gian phục hồi sự cố tính bằng ngày/tuần** thay vì phút/giờ so với nhóm elite.

### 2.4. Scalability Problem — Kiến trúc chống lại việc scale

Đây là vấn đề nghiêm trọng nhất, và là gốc rễ của nhiều nguyên lý 12-Factor. Ứng dụng truyền thống thường **stateful theo cách vô thức**:

**Session trong memory.** User đăng nhập, session lưu trong RAM của server. Muốn thêm server thứ hai? User đăng nhập trên server 1, request tiếp theo rơi vào server 2 → văng ra ngoài. Giải pháp chắp vá là *sticky session* (load balancer ghim user vào một server) — nhưng khi server đó chết, toàn bộ user trên nó mất session, và tải không bao giờ phân bố đều.

**File upload vào local disk.** User upload avatar, file ghi vào `/var/www/uploads` của server 1. Request đọc avatar rơi vào server 2 → 404. Giải pháp chắp vá là NFS mount — thêm một single point of failure mới.

**Log ghi vào file local.** Có 5 server nghĩa là muốn tra một request lỗi, bạn SSH vào 5 máy và grep 5 file.

**Cron job chạy trên "server số 1".** Scale ra 5 server thì job chạy 5 lần? Hay phải nhớ chỉ bật cron trên một máy — và máy đó chết thì sao?

Nhìn chung: mỗi chi tiết tưởng như vô hại (một biến global, một file ghi ra disk, một session trong RAM) đều là một **sợi dây trói ứng dụng vào một máy cụ thể**. Scale horizontal đòi hỏi cắt hết những sợi dây đó — và đó chính xác là điều 12-Factor App hệ thống hóa.

---

## 3. Bản chất: State và Environment Coupling

Nếu phải tóm gọn mọi vấn đề của ứng dụng truyền thống vào hai khái niệm, đó là:

### 3.1. Implicit State — Trạng thái ngầm

Ứng dụng truyền thống rải trạng thái ra khắp nơi mà không khai báo: RAM (session, cache, biến global), local disk (file upload, log, file tạm), và cả "trạng thái của môi trường" (crontab, config sửa tay, package cài thêm). Trạng thái ngầm khiến ứng dụng:

- **Không thể nhân bản** — bản sao thứ hai không có trạng thái của bản thứ nhất.
- **Không thể thay thế** — giết process đi là mất dữ liệu.
- **Không thể suy luận** — hành vi của app phụ thuộc vào lịch sử của máy, không chỉ vào code.

### 3.2. Environment Coupling — Gắn chặt vào môi trường

Ứng dụng biết quá nhiều về nơi nó chạy: IP database hardcode, đường dẫn file tuyệt đối, hostname, port cố định, thư viện cài sẵn trên OS. Mỗi mẩu tri thức đó là một giả định — và mỗi giả định là một điểm gãy khi môi trường thay đổi.

```
Ứng dụng truyền thống:              Ứng dụng lý tưởng:

  App ──biết──> IP của DB             App ──hỏi──> "DB của tôi ở đâu?"
  App ──biết──> đường dẫn /var/www    App ──dùng──> backing service
  App ──giữ──> session trong RAM      App ──đẩy──> session ra Redis
  App ──ghi──> log ra file            App ──in──> log ra stdout

  → App chỉ chạy đúng trên            → App chạy đúng ở bất kỳ đâu
    MỘT máy cụ thể                      cung cấp đủ "câu trả lời"
```

**Điều gì xảy ra nếu vi phạm?** Không có gì xảy ra... cho đến khi bạn cần thay đổi. Đây là điểm nguy hiểm: environment coupling và implicit state là **nợ kỹ thuật không có lãi suất cho đến ngày đáo hạn** — và ngày đáo hạn là ngày bạn cần scale, cần deploy nhanh, hoặc cần dựng lại hệ thống sau sự cố.

---

## 4. Ví dụ minh họa: một ứng dụng Go "truyền thống"

Đoạn code dưới đây là một web app Go viết theo phong cách truyền thống. Nó **chạy được, thậm chí chạy tốt** trên một máy. Hãy đọc và đếm số "sợi dây" trói nó vào máy đó.

```go
// main.go — ANTI-PATTERN: ứng dụng "truyền thống"
// Chạy được trên 1 máy. KHÔNG THỂ scale ra 2 máy.
package main

import (
	"database/sql"
	"fmt"
	"io"
	"log"
	"net/http"
	"os"
	"sync"

	_ "github.com/go-sql-driver/mysql"
)

// ❌ Sợi dây 1: Config hardcode — đổi môi trường là phải sửa code, build lại
const (
	dbDSN      = "appuser:S3cr3tP@ss@tcp(192.168.1.50:3306)/proddb"
	uploadDir  = "/var/www/uploads"          // ❌ Sợi dây 2: local disk
	logFile    = "/var/log/app/app.log"      // ❌ Sợi dây 3: log ra file local
	listenAddr = ":80"                        // ❌ Sợi dây 4: port cố định, cần root
)

// ❌ Sợi dây 5: session lưu trong memory — server thứ 2 không nhìn thấy
var (
	sessions   = map[string]string{} // sessionID -> username
	sessionsMu sync.Mutex
)

func main() {
	// ❌ Log ghi vào file trên disk của máy này
	f, err := os.OpenFile(logFile, os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
	if err != nil {
		log.Fatal(err)
	}
	log.SetOutput(f)

	db, err := sql.Open("mysql", dbDSN)
	if err != nil {
		log.Fatal(err)
	}

	http.HandleFunc("/login", func(w http.ResponseWriter, r *http.Request) {
		user := r.FormValue("user")
		sessionsMu.Lock()
		sessions["sess-"+user] = user // ❌ chỉ tồn tại trong RAM máy này
		sessionsMu.Unlock()
		http.SetCookie(w, &http.Cookie{Name: "sid", Value: "sess-" + user})
		fmt.Fprintln(w, "logged in")
	})

	http.HandleFunc("/upload", func(w http.ResponseWriter, r *http.Request) {
		file, hdr, _ := r.FormFile("f")
		defer file.Close()
		// ❌ File chỉ tồn tại trên disk máy này
		dst, _ := os.Create(uploadDir + "/" + hdr.Filename)
		defer dst.Close()
		io.Copy(dst, file)
		fmt.Fprintln(w, "uploaded")
	})

	_ = db
	log.Fatal(http.ListenAndServe(listenAddr, nil))
}
```

Bài toán kiểm tra nhanh mà bạn nên tự hỏi với mọi ứng dụng: **"Nếu tôi chạy 2 bản của app này sau một load balancer, điều gì hỏng?"**

Với app trên: login hỏng (session), upload hỏng (local disk), debug hỏng (log nằm trên 2 máy), deploy hỏng (config trỏ vào một DB cụ thể được nướng vào binary). Đây chính là bộ đề bài mà 12 chương Factor phía sau sẽ lần lượt giải.

---

## 5. Trade-off: Công bằng với mô hình truyền thống

Một tài liệu tốt không được biến mô hình truyền thống thành hình nộm để đấm. Sự thật là:

**Mô hình truyền thống thắng ở simplicity.** Một binary + một server + một file config là mô hình mà một người có thể hiểu toàn bộ trong đầu. Không cần orchestrator, không cần registry, không cần CI/CD. Với hệ thống nhỏ, tải thấp, ít thay đổi — nó **rẻ hơn và ít rủi ro hơn** so với việc dựng cả một hệ sinh thái cloud-native.

**Stateful trên một máy thắng ở performance.** Session trong RAM nhanh hơn session trong Redis (nano giây so với sub-millisecond cộng network round-trip). Ghi file local nhanh hơn ghi object storage. Nếu bạn *chắc chắn* không bao giờ cần máy thứ hai, các "anti-pattern" trên là những tối ưu hợp lý.

**Vertical scaling đi xa hơn bạn nghĩ.** Một server hiện đại có thể có hàng TB RAM và hàng trăm core. Stack Overflow trong nhiều năm phục vụ lượng traffic khổng lồ với số lượng server vật lý đếm trên đầu ngón tay. "Phải scale horizontal" không phải chân lý phổ quát.

Cái giá thật sự của mô hình truyền thống không nằm ở hiệu năng hay chi phí ban đầu — nó nằm ở **tính lựa chọn (optionality)**: bạn đánh mất khả năng thay đổi nhanh trong tương lai. Kỹ sư giỏi không hỏi "mô hình nào tốt hơn?" mà hỏi "hệ thống này cần giữ những lựa chọn nào cho tương lai, và tôi sẵn sàng trả bao nhiêu cho việc đó?"

---

## 6. Khi nào mô hình truyền thống vẫn là lựa chọn đúng

- **Internal tool cho vài chục người dùng**: một VM, một binary, systemd để restart khi crash. Dựng Kubernetes cho tool chấm công nội bộ là over-engineering.
- **Ứng dụng desktop / embedded**: khái niệm "scale horizontal" không tồn tại; state local là bản chất của bài toán.
- **Batch job đơn giản chạy định kỳ**: một cron + một script có version control có thể là đủ.
- **MVP cần ship trong 2 tuần**: hãy ship trước. Nhưng — và đây là điểm quan trọng — hãy vi phạm các nguyên lý **một cách có ý thức**, biết rõ mình đang vay nợ gì, thay vì vô thức.

Điều tối thiểu nên giữ ngay cả khi làm "kiểu truyền thống": config tách khỏi code, log ra stdout, và mọi thứ nằm trong version control. Ba điều này gần như miễn phí và trả lãi ngay lập tức.

---

## Tóm tắt chương

- Ứng dụng truyền thống không "sai" — nó chỉ **không chịu được thay đổi** về quy mô, tốc độ release và hạ tầng.
- Hai căn bệnh gốc: **implicit state** (trạng thái ngầm trong RAM/disk/môi trường) và **environment coupling** (app biết quá nhiều về nơi nó chạy).
- Câu hỏi kiểm tra vàng: *"Chạy 2 bản app sau load balancer, điều gì hỏng?"*
- Mô hình truyền thống vẫn hợp lý cho hệ thống nhỏ, ít thay đổi — nhưng hãy vay nợ có ý thức.

Chương tiếp theo: hạ tầng thay đổi như thế nào — [Cloud Computing](/series/twelve-factor/02-cloud-computing/) — và vì sao sự thay đổi đó buộc ứng dụng phải thay đổi theo.
