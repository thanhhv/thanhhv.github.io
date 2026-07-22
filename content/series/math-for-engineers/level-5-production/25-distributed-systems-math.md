+++
title = "Chương 25 — Distributed Systems Mathematics: Toán của sự không chắc chắn"
date = "2026-07-20T11:10:00+07:00"
draft = false
tags = ["math", "computer-science", "algorithms"]
series = ["Math for Engineers"]
+++

> Level 5 – Production
> Yêu cầu trước: Chương 02 (Set Theory, Functions & Relations), Chương 05 (Counting & Combinatorics), Chương 11 (Probability), Chương 12 (Hashing), Chương 19 (Number Theory)

---

## 1. Problem Statement

Lúc 14:03:07.421 theo đồng hồ của server tại Singapore, một user bấm nút "Mua". Lúc 14:03:07.418 — *sớm hơn 3 millisecond* — theo đồng hồ của server tại Virginia, chính user đó bấm "Hủy đơn". Câu hỏi tưởng như tầm thường: **hành động nào xảy ra trước?**

Câu trả lời trung thực: *không thể biết*. Hai con số 421 và 418 đến từ hai chiếc đồng hồ khác nhau, và không có gì bảo đảm chúng chỉ cùng một "thời điểm". Đồng hồ quartz trong server trôi khoảng 30–50 phần triệu — nghe nhỏ, nhưng 50ppm nghĩa là **lệch hơn 4 giây mỗi ngày** nếu không hiệu chỉnh. NTP kéo chúng về gần nhau, nhưng qua WAN sai số thường vẫn ở mức vài millisecond đến vài chục millisecond — lớn hơn nhiều khoảng cách 3ms trong câu chuyện trên.

Và ngay cả khi đồng hồ hoàn hảo, vật lý vẫn chặn đường: ánh sáng trong cáp quang chạy ~200.000 km/s (2/3 tốc độ trong chân không). Singapore → Virginia mất tối thiểu ~75ms *một chiều* theo đường chim bay, thực tế trên 100ms. Trong 100ms đó, một CPU hiện đại thực hiện hàng trăm triệu lệnh. Khi thông tin "chuyện gì vừa xảy ra ở đây" đến được máy kia, nó đã là tin cũ. **Trong hệ phân tán, không tồn tại khái niệm "bây giờ" chung** — mỗi máy sống trong hiện tại của riêng nó và chỉ nhìn thấy quá khứ của các máy khác.

Chưa hết. Mạng có thể mất gói, trễ vô hạn, giao sai thứ tự. Máy có thể chết giữa chừng rồi sống lại với dữ liệu cũ. Và điều ác nghiệt nhất: **không có cách nào phân biệt một máy đã chết với một máy chỉ đang chậm** — cả hai đều biểu hiện là "chưa thấy trả lời".

Đây là bài toán trung tâm của chương này:

**Làm sao xây dựng một hệ thống đúng đắn từ nhiều máy không đáng tin, nối với nhau bằng mạng không đáng tin, khi không máy nào biết chắc "bây giờ" là lúc nào?**

Bài toán tách thành ba câu hỏi nhỏ, và mỗi câu có một lời giải toán học đẹp đến bất ngờ:

1. **Partition** — dữ liệu nên nằm ở máy nào? (số học modular trên vòng tròn)
2. **Replication** — ghi bao nhiêu bản, đọc bao nhiêu bản thì an toàn? (nguyên lý chuồng bồ câu)
3. **Ordering & Consensus** — chuyện gì xảy ra trước, và cả cụm thống nhất bằng cách nào? (quan hệ thứ tự bộ phận, và một định lý bất khả thi)

## 2. Trực giác

### Hệ phân tán là một nhóm người viết thư cho nhau

Hãy quên máy tính đi. Tưởng tượng năm người bạn ở năm thành phố, chỉ liên lạc bằng thư — thư có thể lạc, có thể đến sau ba tuần, hai lá thư gửi cách nhau một ngày có thể đến ngược thứ tự. Đồng hồ mỗi người chạy lệch nhau vài phút. Nhóm này phải cùng quản lý một sổ cái chung.

Mọi khái niệm trong chương này đều là lời giải cho một khía cạnh của tình huống đó:

- **Chia việc (consistent hashing)**: chia sổ cái thành phần cho từng người sao cho khi một người nghỉ, không phải chép lại toàn bộ sổ.
- **Sao lưu (quorum)**: mỗi mục ghi vào sổ của vài người; muốn đọc thì hỏi vài người — chọn "vài" bao nhiêu để chắc chắn có ít nhất một người biết bản mới nhất?
- **Thứ tự (logical clock)**: đừng tin đồng hồ treo tường; hãy nhìn *quan hệ nhân quả* — nếu thư của A trích dẫn thư của B, thì thư B chắc chắn viết trước.
- **Đồng thuận (consensus)**: khi cần cả nhóm quyết một chuyện, biểu quyết theo đa số — vì hai nhóm đa số bất kỳ luôn có người chung, không thể có hai quyết định trái ngược cùng "thắng".
- **Lan truyền (gossip)**: mỗi người kể tin cho một người ngẫu nhiên mỗi tuần — tin đồn lan đến cả nhóm nhanh đáng kinh ngạc.

### Trực giác về quorum: hai ủy ban phải có người chung

Công ty 5 người có quy định: mọi quyết định ghi vào sổ phải được một "ủy ban ghi" 3 người ký; mọi lần tra cứu phải hỏi một "ủy ban đọc" 3 người. Bất kể hai ủy ban được chọn thế nào, 3 + 3 = 6 > 5 người — theo nguyên lý chuồng bồ câu (chương 05), **hai ủy ban luôn có ít nhất một thành viên chung**. Người đó biết quyết định mới nhất, nên lần đọc không bao giờ "trắng tay". Toàn bộ lý thuyết quorum chỉ là mệnh đề này viết bằng ký hiệu.

### Trực giác về thời gian: nhân quả thay cho đồng hồ

Bạn nhận được hai email không có timestamp. Email thứ hai trích dẫn nội dung email thứ nhất. Bạn biết chắc cái nào viết trước — **không cần đồng hồ**. Nhưng nếu hai email không liên quan gì đến nhau? Khi đó câu hỏi "cái nào trước" *không có ý nghĩa*, và điều đúng đắn là thừa nhận chúng **đồng thời (concurrent)**. Thời gian trong hệ phân tán không phải một đường thẳng mà là một mạng nhân quả — một *thứ tự bộ phận* (partial order, chương 02), không phải thứ tự toàn phần.

## 3. First Principles

Xuất phát từ đúng bốn sự thật vật lý, không giả định gì thêm:

1. **Thông tin lan truyền có độ trễ, và độ trễ không có chặn trên** (mô hình bất đồng bộ): message có thể đến sau 1ms hoặc 10 giây, hoặc không bao giờ.
2. **Đồng hồ vật lý trôi** với tốc độ khác nhau ở mỗi máy.
3. **Máy có thể dừng bất cứ lúc nào** (crash), kể cả giữa một thao tác.
4. **Từ (1) và (3): không thể phân biệt máy chết với máy chậm.** Đây là giới hạn nhận thức luận (epistemic limit) quan trọng nhất — mọi failure detector đều chỉ là *phỏng đoán*.

Từ bốn tiên đề này, các hệ quả được *suy ra*, không phải chọn tùy tiện:

- Muốn sống sót khi máy chết (3) → phải **nhân bản dữ liệu** → lập tức sinh ra bài toán các bản sao khác nhau → cần định nghĩa "bản nào mới hơn" → nhưng (2) cấm dùng timestamp vật lý → phải xây **thời gian logic** từ quan hệ gửi–nhận message.
- Muốn nhiều máy cùng quyết một giá trị trong khi (4) đúng → có kẻ sẽ phải quyết định khi *chưa chắc* các máy khác còn sống → đây là gốc rễ của **FLP impossibility**.
- Muốn chia dữ liệu cho n máy mà n thay đổi liên tục → cần ánh xạ key → máy có tính chất **ổn định cục bộ**: thay đổi một máy chỉ ảnh hưởng một phần nhỏ ánh xạ. Hàm `hash % n` không có tính chất đó; ta phải thiết kế một hàm khác.

Nếu không có lớp toán học này thì sao? Bạn vẫn xây được hệ phân tán — nó sẽ chạy tốt trong demo, rồi âm thầm mất dữ liệu mỗi khi mạng chập chờn, và bạn không bao giờ tái hiện được bug. Sự khác biệt giữa "chạy được" và "đúng đắn" trong hệ phân tán không thể kiểm chứng bằng test thông thường, vì kẻ thù là *thứ tự xen kẽ của các sự kiện* — không gian tổ hợp khổng lồ (chương 05) mà test chỉ chạm được một góc. Chỉ có chứng minh mới phủ hết.

## 4. Mathematical Model

### 4.1 Consistent Hashing — số học trên vòng tròn

**Vấn đề của `hash % n`.** Cách chia dữ liệu ngây thơ nhất: key k nằm ở máy `hash(k) % n`. Đúng, đều, đơn giản. Nhưng xem điều gì xảy ra khi thêm máy thứ 11 vào cụm 10 máy: key k giữ nguyên chỗ chỉ khi `hash(k) % 10 == hash(k) % 11`. Với hash phân bố đều (chương 12), xác suất đó chỉ khoảng 1/11 — nghĩa là **~91% số key phải di chuyển**. Tổng quát: đổi số máy quanh mức n làm xấp xỉ (n−1)/n số key đổi chỗ — cụm càng lớn, thảm họa càng gần 100%. Với cache cluster, đây là cache miss storm đánh thẳng vào database; với storage cluster, đây là hàng terabyte trôi qua mạng chỉ vì *một* máy thêm vào. Trong khi lượng di chuyển *tối thiểu cần thiết* chỉ là 1/(n+1) — đủ để lấp đầy máy mới. Ta trả đắt gấp n lần mức cần.

Nguyên nhân sâu xa: phép `% n` đưa **n vào định nghĩa của ánh xạ**. Đổi n là đổi toàn bộ hàm. Lời giải: loại n ra khỏi công thức.

**Ring 2⁶⁴.** Hash cả key lẫn tên máy vào cùng một không gian [0, 2⁶⁴), và coi không gian đó là một vòng tròn — vị trí 2⁶⁴ − 1 kề vị trí 0, mọi khoảng cách tính theo modulo 2⁶⁴ (số học modular, chương 19 — cùng thứ toán học của mặt đồng hồ 12 giờ). Quy tắc duy nhất: **key thuộc về máy đầu tiên theo chiều kim đồng hồ tính từ vị trí của key**.

```
                    hash(node A)
                   ╱
          ┌──────●──────┐
      key₃ ✕            ✕ key₁        key₁ → đi theo chiều kim
        │                 │            đồng hồ → gặp B trước
   hash(C) ●              │            → key₁ thuộc B
        │               ● hash(B)
        └───────✕───────┘
              key₂                    key₂ → thuộc C
```

Chú ý điều đẹp đẽ: định nghĩa ánh xạ **không chứa n**. Thêm máy D vào giữa cung C–A: chỉ những key nằm trên cung từ C đến D đổi chủ (từ A sang D); mọi key khác không hề biết D tồn tại.

**Vì sao chỉ K/n key di chuyển?** Với hash tốt, vị trí của n máy là n điểm "ngẫu nhiên đều" trên vòng tròn. Vì các máy hoàn toàn đối xứng — không máy nào đặc biệt — kỳ vọng độ dài cung mỗi máy sở hữu là 1/n vòng tròn (chương 11: tính đối xứng cho ngay kỳ vọng mà không cần tích phân). Máy mới thêm vào chiếm một cung có kỳ vọng 1/(n+1) vòng → kỳ vọng **K/(n+1) key di chuyển** trên tổng K key, và không key nào khác bị đụng. Đây chính là mức tối thiểu lý thuyết ở trên — consistent hashing đạt tối ưu về lượng di chuyển (theo kỳ vọng), và đó là nội dung của chữ "consistent".

**Virtual nodes — vũ khí chống hot spot.** Kỳ vọng là 1/n, nhưng *phương sai* thì lớn: với mỗi máy chỉ một điểm trên ring, độ dài các cung rất chênh lệch (cung dài nhất cỡ Θ(log n / n) — gấp log n lần trung bình), nghĩa là có máy gánh gấp vài lần máy khác. Lời giải: mỗi máy vật lý xuất hiện tại **v điểm ảo** trên ring (hash "node-A#0", "node-A#1", …). Tải của máy = tổng độ dài v cung nhỏ. Theo luật số lớn (chương 11), tổng của v biến ngẫu nhiên gần độc lập có độ lệch chuẩn tương đối co lại theo **1/√v**: v = 100 đưa mất cân bằng về mức ±10%, v = 200 về ~±7%. Đó là lý do các hệ thực tế chọn 100–256 vnode mỗi máy (Cassandra cổ điển: `num_tokens=256`) — tăng thêm nữa lợi ích giảm theo căn bậc hai nhưng chi phí metadata tăng tuyến tính, không đáng. Bonus quan trọng: khi một máy chết, v cung của nó rơi vào v máy kế cận *khác nhau* — tải của máy chết được rải đều thay vì đổ trọn lên một nạn nhân.

### 4.2 Quorum — nguyên lý chuồng bồ câu bảo vệ dữ liệu

Dữ liệu được nhân bản trên N máy. Ghi được xác nhận khi **W** bản sao ghi xong; đọc hỏi **R** bản sao và lấy bản có version mới nhất. Điều kiện vàng:

> **R + W > N ⟹ mọi lần đọc nhìn thấy lần ghi thành công gần nhất**

*Chứng minh* (một dòng, bằng đếm — chương 05): gọi tập máy đã ghi là 𝒲 (|𝒲| = W), tập máy được đọc là ℛ (|ℛ| = R). Theo inclusion–exclusion: |ℛ ∩ 𝒲| = |ℛ| + |𝒲| − |ℛ ∪ 𝒲| ≥ R + W − N ≥ 1. Hai tập **buộc phải giao nhau** — cùng một lý lẽ chuồng bồ câu như hai ủy ban ở mục 2. Máy nằm trong giao điểm giữ bản ghi mới nhất, và phép đọc so version sẽ chọn đúng nó. ∎

Lưu ý điều định lý *không* nói: nó không nói bản sao nào "mới nhất" — muốn so sánh version giữa các bản, ta cần thời gian logic (mục 4.3). Quorum bảo đảm *nhìn thấy*, thời gian logic bảo đảm *nhận ra*.

Với N = 3, các cấu hình thỏa R + W > N:

| Cấu hình | Ghi | Đọc | Chịu lỗi khi ghi | Chịu lỗi khi đọc | Phù hợp |
|---|---|---|---|---|---|
| W=2, R=2 | chờ 2/3 máy | hỏi 2/3 máy | 1 máy chết vẫn ghi được | 1 máy chết vẫn đọc được | cân bằng — mặc định phổ biến |
| W=3, R=1 | chờ cả 3 | hỏi 1 máy bất kỳ | **1 máy chết → không ghi được** | 2 máy chết vẫn đọc được | read-heavy, ghi hiếm |
| W=1, R=3 | chờ 1 | hỏi cả 3 | rất sẵn sàng khi ghi | 1 máy chết → không đọc được | write-heavy, đọc hiếm |

Còn W=1, R=1 (không thỏa điều kiện)? Đó không phải cấu hình "sai" — đó là lựa chọn **eventual consistency**: nhanh nhất, sẵn sàng nhất, và chấp nhận đọc phải dữ liệu cũ. Toán học không cấm; nó chỉ buộc bạn *biết mình đang đánh đổi gì*.

### 4.3 Thời gian logic — đo "trước/sau" không cần đồng hồ

**Quan hệ happened-before.** Lamport (1978) định nghĩa quan hệ **a → b** ("a xảy ra trước b") là quan hệ nhỏ nhất thỏa:

1. a và b cùng process và a đến trước b theo thứ tự chương trình ⟹ a → b
2. a là gửi message m, b là nhận message m ⟹ a → b
3. Bắc cầu: a → b và b → c ⟹ a → c

Đây là một **strict partial order** (chương 02): có những cặp sự kiện không so sánh được — không a → b cũng không b → a — và ta gọi chúng là **concurrent (a ∥ b)**. Điểm triết học quan trọng: concurrent không phải là "chưa biết cái nào trước", mà là "**không tồn tại** cái nào trước" — không có luồng thông tin nào nối hai sự kiện, nên mọi thứ tự gán cho chúng đều là quy ước, không phải sự thật.

**Lamport clock.** Mỗi process giữ một số nguyên L, với ba quy tắc:

- Trước mỗi sự kiện nội bộ: `L++`
- Khi gửi message: `L++`, đính kèm L vào message
- Khi nhận message mang giá trị t: `L = max(L, t) + 1`

Tính chất (chứng minh bằng quy nạp trên định nghĩa của →): **a → b ⟹ L(a) < L(b)**. Nhưng chiều ngược lại **sai**: L(a) < L(b) không suy ra a → b — hai sự kiện concurrent vẫn có thể mang số khác nhau. Lamport clock là "mũi tên một chiều": nó *tôn trọng* nhân quả nhưng không *phát hiện* được sự vắng mặt của nhân quả. Đủ để tạo thứ tự toàn phần nhất quán (dùng L, tie-break bằng process ID) — nền của distributed lock, total order broadcast — nhưng không đủ để trả lời "hai bản ghi này có xung đột không?".

**Vector clock.** Muốn chiều ngược lại, phải trả thêm tiền: mỗi process i giữ nguyên một **vector** VC[1..n] — "hiểu biết của tôi về tiến độ của từng process". Quy tắc: sự kiện nội bộ tăng VC[i]; gửi kèm cả vector; nhận thì lấy max từng phần tử rồi tăng VC[i]. So sánh: VC(a) < VC(b) khi mọi phần tử ≤ và ít nhất một phần tử <. Khi đó:

> **a → b ⟺ VC(a) < VC(b)**, và a ∥ b ⟺ hai vector không so sánh được

Vector clock *phát hiện được concurrency* — đây chính là cách Dynamo nhận ra hai version của giỏ hàng được sửa song song và cần merge thay vì ghi đè. Cái giá: mỗi message/record cõng O(n) số nguyên, với n là số node (hoặc số client) từng chạm vào dữ liệu — lý do các hệ thống thực phải cắt tỉa vector và đôi khi lùi về Last-Write-Wins.

**Spanner TrueTime — mua lại đồng hồ vật lý bằng phần cứng.** Google đi đường ngược: trang bị GPS + đồng hồ nguyên tử cho mỗi datacenter, và API `TT.now()` trả về **một khoảng** [earliest, latest] với cam kết thời điểm thật nằm trong đó (độ rộng ε thường 1–7ms). Muốn hai transaction có thứ tự timestamp đúng với thứ tự thật, Spanner chỉ cần **commit-wait**: giữ kết quả lại cho đến khi `latest` của thời điểm commit đã trôi qua — chờ ε mili-giây để đổi lấy external consistency trên quy mô toàn cầu. Bài học: không phá được giới hạn vật lý thì *định lượng* nó; sự không chắc chắn được đo đạc cẩn thận cũng dùng được như sự chắc chắn.

### 4.4 Các định lý giới hạn: FLP, CAP, và sức mạnh của đa số

**FLP impossibility (Fischer–Lynch–Paterson, 1985).** Phát biểu: *trong hệ bất đồng bộ (độ trễ message không chặn trên), nếu chỉ một process có thể crash, thì không tồn tại thuật toán consensus tất định nào bảo đảm luôn kết thúc.* Trực giác: vì không phân biệt được "chết" với "chậm" (First Principles số 4), luôn tồn tại một kịch bản trì hoãn message ác ý khiến hệ do dự mãi mãi. Hệ quả thực tiễn — và đây là cách đọc đúng một định lý bất khả thi: **mọi hệ consensus thật (Paxos, Raft, Zab) đều phải dùng timeout hoặc randomness**, tức chấp nhận hy sinh *liveness* (có thể tạm không tiến triển khi mạng quá tệ) để giữ *safety* (không bao giờ ra hai quyết định trái nhau). Khi cluster etcd "mất leader" và khựng lại vài giây — đó không phải bug, đó là FLP đang thu phí.

**CAP — phát biểu chính xác.** Dạng đúng (Gilbert & Lynch, 2002): *một hệ phân tán không thể đồng thời bảo đảm Consistency (linearizability), Availability (mọi request tới node còn sống đều được trả lời), và Partition tolerance (chịu được mất liên lạc giữa các nhóm node).* Vì network partition là sự kiện *sẽ xảy ra* chứ không phải lựa chọn, cách đọc đúng là: **khi partition xảy ra, phải chọn C hoặc A**. Các diễn giải sai phổ biến:

- *"Chọn 2 trong 3"* — sai, vì P không phải món để chọn; hệ "CA phân tán" không tồn tại trên mạng thật.
- *Nhầm C với Consistency trong ACID* — C của CAP là linearizability (mọi thao tác như thể xảy ra tại một thời điểm trên một bản duy nhất), khái niệm hẹp và đắt hơn nhiều.
- *Nghĩ rằng lúc nào cũng phải hy sinh* — khi không có partition, hệ có thể có cả C lẫn A.

**PACELC** vá đúng lỗ hổng cuối: *if Partition then A-vs-C, Else Latency-vs-Consistency* — ngay cả lúc mạng khỏe, muốn consistency mạnh hơn vẫn phải trả thêm độ trễ (chờ nhiều bản sao xác nhận hơn). DynamoDB là PA/EL; Spanner là PC/EC.

**Raft và con số 2f+1.** Muốn chịu được f máy chết mà vẫn hoạt động, cluster cần **N = 2f+1** máy, và mọi quyết định (bầu leader, commit log entry) cần **đa số f+1 phiếu**. Vì sao đa số? Lại chuồng bồ câu: hai tập đa số bất kỳ có (f+1) + (f+1) = 2f+2 > N thành viên → **luôn giao nhau ít nhất một máy**. Máy chung đó khiến hai leader không thể được bầu trong cùng term, và mọi leader tương lai chắc chắn *nhìn thấy* mọi entry đã commit — safety được bảo vệ bằng đúng một bất đẳng thức. Hệ quả ít ai để ý: cluster 4 máy cần đa số 3, chỉ chịu được 1 máy chết — *không tốt hơn* cluster 3 máy, lại thêm chi phí. Đó là lý do etcd/ZooKeeper luôn khuyến nghị số lẻ: 3, 5, 7.

**Gossip — toán của tin đồn.** Mỗi vòng, mỗi node đã "nhiễm" thông tin kể cho một node ngẫu nhiên. Số node nhiễm xấp xỉ *nhân đôi* mỗi vòng khi còn ít, nên phủ toàn cụm n node trong **O(log n) vòng** (phân tích kỹ: ~log₂n + ln n vòng; n = 10.000 → ~23 vòng, mỗi vòng 1 giây thì nửa phút cho vạn máy). Không cần điều phối trung tâm, chịu lỗi tự nhiên (mất vài message chỉ chậm một nhịp), tổng lưu lượng O(n log n). Đây là mô hình lan truyền dịch bệnh (epidemic model) — cùng phương trình với virus, áp dụng cho membership và failure detection trong Cassandra, Consul, và giao thức SWIM.

## 5. Thuật toán

Consistent hash ring với virtual nodes — phiên bản rút gọn nhưng đúng cấu trúc mà thư viện thực tế dùng (sorted slice + binary search, tra cứu O(log V) với V là tổng số vnode):

```go
package ring

import (
	"fmt"
	"hash/fnv"
	"sort"
)

type Ring struct {
	vnodes int               // số virtual node mỗi máy (100–200)
	keys   []uint64          // vị trí các vnode trên vòng, đã sort
	owner  map[uint64]string // vị trí vnode → tên máy vật lý
}

func New(vnodes int) *Ring {
	return &Ring{vnodes: vnodes, owner: make(map[uint64]string)}
}

func hash64(s string) uint64 {
	h := fnv.New64a()
	h.Write([]byte(s))
	return h.Sum64()
}

// AddNode đặt vnodes điểm ảo của máy lên vòng tròn.
// Chỉ các key nằm trên những cung ngay trước các điểm này đổi chủ —
// kỳ vọng K/(n+1) key, mọi key khác không bị ảnh hưởng.
func (r *Ring) AddNode(name string) {
	for i := 0; i < r.vnodes; i++ {
		h := hash64(fmt.Sprintf("%s#%d", name, i))
		r.owner[h] = name
		r.keys = append(r.keys, h)
	}
	sort.Slice(r.keys, func(a, b int) bool { return r.keys[a] < r.keys[b] })
}

// Get trả về máy sở hữu key: vnode đầu tiên theo chiều kim đồng hồ.
func (r *Ring) Get(key string) string {
	h := hash64(key)
	// binary search: vnode nhỏ nhất có vị trí >= h
	i := sort.Search(len(r.keys), func(i int) bool { return r.keys[i] >= h })
	if i == len(r.keys) {
		i = 0 // vòng qua điểm 0 của ring — đây chính là phép mod 2^64
	}
	return r.owner[r.keys[i]]
}
```

Lamport clock — toàn bộ "thuyết tương đối cho hệ phân tán" nằm gọn trong một phép `max`:

```go
type LamportClock struct {
	mu sync.Mutex
	t  uint64
}

// Tick: gọi trước mỗi sự kiện nội bộ hoặc trước khi gửi message.
func (c *LamportClock) Tick() uint64 {
	c.mu.Lock()
	defer c.mu.Unlock()
	c.t++
	return c.t
}

// Recv: gọi khi nhận message mang timestamp remote.
// max() đảm bảo đồng hồ nhận không bao giờ "đi lùi" so với nhân quả:
// sự kiện nhận luôn được đánh số lớn hơn sự kiện gửi.
func (c *LamportClock) Recv(remote uint64) uint64 {
	c.mu.Lock()
	defer c.mu.Unlock()
	if remote > c.t {
		c.t = remote
	}
	c.t++
	return c.t
}
```

Với vector clock, phép so sánh là trái tim — kết quả có **bốn** khả năng, không phải ba, và khả năng thứ tư chính là thông tin quý nhất:

```go
// Compare trả về: -1 nếu a → b, +1 nếu b → a,
// 0 nếu bằng nhau, và ErrConcurrent nếu a ∥ b (cần merge/giải xung đột).
func Compare(a, b []uint64) (int, error) {
	aLess, bLess := false, false
	for i := range a {
		if a[i] < b[i] {
			aLess = true
		}
		if a[i] > b[i] {
			bLess = true
		}
	}
	switch {
	case aLess && bLess:
		return 0, ErrConcurrent // không so sánh được: hai nhánh lịch sử song song
	case aLess:
		return -1, nil
	case bLess:
		return 1, nil
	}
	return 0, nil
}
```

## 6. Trade-off

**Consistency vs Latency — trade-off gốc.** Mọi mức consistency mạnh hơn đều mua bằng round-trip: eventual (W=1) trả lời sau 1 lần ghi local; quorum chờ máy *chậm thứ W* trong N (tail latency, chương 18 sẽ nhắc bạn P99 của max nhiều máy tệ hơn P99 từng máy); linearizability qua Raft tốn thêm một vòng leader → đa số. Không có cấu hình "tốt nhất" — chỉ có cấu hình khớp hay không khớp với nghiệp vụ: số dư ngân hàng đáng giá 2 round-trip; đếm like thì không.

**W cao hay R cao?** R + W > N là một *đường ngân sách*: tổng cố định, chia cho bên nào tùy tần suất. Hệ đọc 99% nên dồn chi phí sang ghi (W=N, R=1). Nhưng nhớ mặt tối của W=N: chỉ một replica chết là **mất khả năng ghi** — availability của phép ghi giảm khi W tăng. Quorum đối xứng W=R=⌈(N+1)/2⌉ là điểm cân bằng an toàn khi chưa biết workload.

**Số virtual node.** Nhiều vnode → cân bằng tải tốt hơn (∝1/√v) và rải đều tải khi node chết; nhưng metadata ring lớn hơn, rebalance nhiều mảnh nhỏ hơn, và xác suất "nhiều bản sao của cùng key rơi vào cùng máy vật lý" cần xử lý riêng (placement phải nhảy qua vnode cùng chủ). Cassandra hiện đại *giảm* mặc định từ 256 xuống 16 vnode kết hợp thuật toán phân bổ thông minh hơn — dấu hiệu của một trade-off thật: con số tối ưu thay đổi khi các ràng buộc khác thay đổi.

**Lamport vs Vector clock.** O(1) mỗi message so với O(n) — nhưng khác biệt thật nằm ở *câu hỏi trả lời được*: Lamport đủ nếu bạn chỉ cần một thứ tự toàn phần nhất quán nào đó; vector clock cần thiết nếu bạn phải *phát hiện* xung đột. Chọn Last-Write-Wins với timestamp vật lý là lựa chọn thứ ba: rẻ nhất, và âm thầm vứt dữ liệu khi có ghi đồng thời — Cassandra chọn nó một cách có chủ đích, và kỹ sư dùng Cassandra phải biết điều đó.

**Consensus vs Gossip.** Raft cho câu trả lời *chắc chắn* nhưng mọi thao tác đi qua leader — throughput bị chặn bởi một máy, và cụm dừng khi mất đa số. Gossip không chắc chắn tại một thời điểm (chỉ hội tụ *dần*) nhưng scale tuyến tính và không có điểm nghẽn. Vì thế kiến trúc trưởng thành dùng cả hai đúng chỗ: Raft cho metadata nhỏ-mà-thiêng (cấu hình, leader, schema), gossip cho trạng thái lớn-mà-xuề-xòa (membership, health).

## 7. Production Applications

**DynamoDB / Cassandra — cả chương này trong một hệ thống.** Kiến trúc Dynamo (paper 2007) là bản lắp ráp đúng ba lời giải: consistent hashing với virtual nodes để partition; sloppy quorum N=3, W=2, R=2 để replication; vector clock (Dynamo) hoặc LWW-timestamp (Cassandra) để xử lý version. Cassandra cho chỉnh consistency level *theo từng query* (`ONE`, `QUORUM`, `ALL`) — bất đẳng thức R + W > N được trao tận tay lập trình viên như một núm vặn.

**Kafka — quorum kiểu khác.** Kafka không dùng majority quorum cho dữ liệu mà dùng **ISR (In-Sync Replicas)**: tập replica đang bám kịp leader. Ghi với `acks=all` chỉ chờ các máy trong ISR; `min.insync.replicas=2` cùng replication factor 3 tạo bất biến kiểu R + W > N: ghi chạm ≥2 máy, nên còn ≥1 máy giữ đủ dữ liệu khi 1 máy chết. Điểm tinh tế mà nhiều team vấp: `acks=all` với ISR co về còn 1 máy (khi 2 follower chậm) *không* bảo đảm gì cả — thiếu `min.insync.replicas`, cam kết chỉ là ảo giác.

**etcd/Raft trong Kubernetes.** Toàn bộ trạng thái cluster Kubernetes — mọi Pod, Service, Secret — sống trong etcd, một Raft state machine. Ba hoặc năm máy, mọi thao tác ghi qua leader và cần đa số. Đây là lý do tài liệu vận hành khuyên cluster etcd số lẻ và đặt các thành viên ở các failure domain khác nhau: con số 2f+1 của mục 4.4 hiện nguyên hình thành quy tắc SRE.

**Redis Cluster — 16384 hash slot.** Redis chọn biến thể rời rạc của consistent hashing: `slot = CRC16(key) mod 16384`, và các *slot* (không phải key) được gán cho node. Di chuyển dữ liệu = chuyển giao slot — thô hơn ring liên tục nhưng metadata bé (16384 × node-id, đủ nhỏ để gossip trong heartbeat) và dễ vận hành. Cùng một bài toán, lời giải thô hơn có chủ đích — vì Redis ưu tiên sự đơn giản vận hành hơn độ mịn phân phối.

**CRDT — nhân quả hóa thành cấu trúc dữ liệu.** Conflict-free Replicated Data Types là bước tiến hóa sau vector clock: thay vì *phát hiện* xung đột rồi bắt ai đó giải quyết, thiết kế kiểu dữ liệu sao cho merge **giao hoán, kết hợp, lũy đẳng** — mọi thứ tự áp dụng đều hội tụ về cùng kết quả (một semilattice, họ hàng của partial order chương 02). Figma, Google Docs (họ OT/CRDT), Redis CRDB dùng chúng để nhiều người sửa cùng tài liệu offline rồi hợp nhất không cần khóa. Đổi lại là ràng buộc ngặt lên phép toán được phép — không phải logic nghiệp vụ nào cũng nhét vừa một semilattice.

## 8. Interview

Câu hỏi system design kinh điển: **"Thiết kế một distributed cache / key-value store."** Ứng viên yếu liệt kê công nghệ ("dùng Redis, thêm Kafka..."); ứng viên mạnh trả lời bằng toán, theo đúng trình tự ba bài toán con của chương này:

1. **Partition** — "Tôi dùng consistent hashing với ~150 virtual node mỗi máy. Vì sao không `hash % n`: thêm một máy làm (n−1)/n key di chuyển, còn ring chỉ K/n. Vì sao virtual node: giảm phương sai tải theo 1/√v và rải tải khi node chết."
2. **Replication** — "Mỗi key nhân bản N=3 máy kế tiếp trên ring. Chọn W=2, R=2 vì R+W>N bảo đảm đọc thấy ghi mới nhất — chứng minh bằng pigeonhole. Nếu đề bài nói cache chấp nhận stale, tôi hạ xuống W=1, R=1 và nói rõ cái giá."
3. **Consistency** — "Version bằng vector clock nếu cần phát hiện ghi đồng thời, LWW nếu chấp nhận mất; metadata cụm (membership, cấu hình) qua Raft hoặc gossip tùy quy mô."

Mỗi lựa chọn đi kèm một *lý do định lượng* — đó là điều interviewer chấm, không phải tên công nghệ.

**Lỗi tư duy phổ biến trong phỏng vấn (và production):**

- **Tin đồng hồ vật lý**: "Tôi lấy bản ghi có timestamp lớn hơn" — interviewer sẽ hỏi ngay: hai máy lệch 50ms thì sao? Câu trả lời đúng phải nhắc clock skew và thời gian logic.
- **Hiểu CAP là "chọn 2 trong 3" mọi lúc**: phát biểu đúng là *khi partition xảy ra* mới phải chọn C hoặc A; lúc bình thường trade-off là latency vs consistency (PACELC).
- **"Thêm node để tăng độ chịu lỗi" một cách máy móc**: 4 node Raft không chịu lỗi tốt hơn 3; chịu lỗi đến từ *đa số*, không từ *số lượng*.
- **Quên câu hỏi "ai phát hiện node chết, và phát hiện sai thì sao?"** — mọi thiết kế nêu "khi node chết thì failover" đều phải trả lời được: dựa vào timeout, và timeout có thể nhầm máy chậm thành máy chết (FLP không cho phép làm tốt hơn về nguyên tắc).

## 9. Anti-pattern

**Dùng timestamp vật lý để quyết định thứ tự nghiệp vụ.** `ORDER BY created_at` giữa các bản ghi sinh từ nhiều server, "lấy bản ghi mới nhất" bằng wall clock, TTL so sánh giờ giữa hai máy — tất cả đều là bug tiềm ẩn với xác suất kích hoạt tỷ lệ thuận với traffic. Đặc biệt hiểm: NTP *step* có thể kéo đồng hồ **đi lùi**, khiến `time.Now()` lần sau nhỏ hơn lần trước. (Go cung cấp monotonic clock trong `time.Time` chính vì vậy — nhưng monotonic chỉ có nghĩa trong một process.)

**Shard bằng `hash % n` rồi mới nghĩ đến chuyện scale.** Hoạt động hoàn hảo đến ngày thêm máy thứ n+1 — và hôm đó là ngày cache hit rate rơi từ 95% về ~9%, database phía sau nhận cú đấm gấp 18 lần bình thường. Quả bom nổ đúng lúc bạn scale *vì* đang tăng trưởng — thời điểm tệ nhất có thể.

**Cluster số chẵn "cho đủ đôi".** 2 node Raft: đa số là 2 — *một* máy chết là cụm tê liệt, tệ hơn chạy 1 máy. 4 node: đa số 3, chịu lỗi đúng 1 máy như cụm 3 node nhưng đắt hơn và xác suất có máy hỏng cao hơn. Độ chịu lỗi là hàm bậc thang của ⌊(N−1)/2⌋, không phải hàm tuyến tính của N.

**Coi "eventual consistency" là "consistency, chỉ hơi chậm".** Eventual consistency cho phép đọc thấy dữ liệu cũ, thấy ghi của chính mình biến mất (không read-your-writes), thấy hai lần đọc liên tiếp *lùi thời gian*. Code frontend giả định "vừa POST xong thì GET phải thấy" trên storage eventual là lớp bug kinh điển — sửa bằng session consistency/sticky routing, hoặc chấp nhận và thiết kế UI quanh nó.

**Retry không kèm idempotency.** Mạng không đáng tin nghĩa là "không nhận được ACK" ≠ "thao tác không xảy ra". Retry một lệnh trừ tiền không idempotent là trừ tiền hai lần. Mọi API ghi trong hệ phân tán cần idempotency key — hệ quả trực tiếp của việc message có thể lạc *ở chiều về*.

## 10. Best Practices

**Nên:**

- Thiết kế theo trình tự ba câu hỏi: partition thế nào (consistent hashing/slot), replicate thế nào (N, W, R — viết rõ bất đẳng thức), đồng thuận/thứ tự thế nào (Raft cho metadata, logical clock hoặc LWW *có chủ đích* cho data).
- Chọn consistency level theo *nghiệp vụ từng thao tác*, không theo hệ thống: cùng một app, số dư dùng QUORUM, presence status dùng ONE.
- Dùng số lẻ node cho mọi cụm consensus; đặt thành viên khác failure domain (rack/AZ).
- Đo clock skew thực tế giữa các máy của bạn và coi nó như một SLO vận hành — bạn không loại được skew, nhưng phải *biết* nó đang là bao nhiêu.
- Kiểm tra hệ thống bằng cách *tiêm* sự không chắc chắn: chaos testing, tăng độ trễ nhân tạo, partition giả — vì không gian interleaving quá lớn để chờ bug tự đến (và hãy đọc các báo cáo Jepsen như đọc truyện trinh thám).

**Không nên:**

- Không tự viết consensus. Paxos/Raft là những thuật toán mà cả tác giả lẫn reviewer giỏi nhất vẫn viết sai; dùng etcd, ZooKeeper, hoặc thư viện Raft đã qua kiểm chứng.
- Không cam kết "exactly-once delivery" trong tài liệu API — toán học chỉ cho phép at-least-once + idempotency (hoặc at-most-once); "exactly-once processing" là tính chất của *pipeline có transaction*, không phải của message delivery.
- Không thêm replica/node để "tăng an toàn" mà không tính lại quorum — thêm node có thể *giảm* availability của phép ghi nếu W dâng theo N.

**Câu hỏi tự kiểm tra khi rời chương này:**

1. Cụm cache 20 máy dùng `hash % 20`. Thêm máy thứ 21, bao nhiêu phần trăm key đổi chỗ? Cùng kịch bản với consistent hashing thì sao — và vì sao con số thứ hai là tối ưu?
2. Hệ N=3, W=2, R=2: chứng minh trong một dòng rằng phép đọc luôn chạm ít nhất một bản sao mới nhất. Định lý này *không* bảo đảm điều gì, và cần thêm công cụ nào để lấp?
3. Lamport clock cho L(a) = 5, L(b) = 9. Kết luận nào sau đây được phép rút ra: a → b; b ↛ a; a ∥ b? Vì sao vector clock trả lời được nhiều hơn, và cái giá là gì?

---

*Chương tiếp theo: [26 — Cryptography Mathematics](/series/math-for-engineers/level-5-production/26-cryptography/), nơi sự không chắc chắn đổi vai: thay vì là kẻ thù phải khắc chế, độ khó tính toán trở thành bức tường thành — bảo mật được xây trực tiếp trên những bài toán số học không ai giải nổi trong thời gian hợp lý.*
