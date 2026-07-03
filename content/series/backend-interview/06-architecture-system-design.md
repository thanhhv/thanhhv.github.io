+++
title = "Bài 6 — Architecture & System Design"
date = "2026-07-03T13:00:00+07:00"
draft = false
tags = ["backend", "interview"]
series = ["Backend Interview"]
+++

# Architecture & System Design

---

## Câu 1 — [Intermediate → Senior] Clean Architecture / Hexagonal Architecture: giải quyết vấn đề gì và khi nào là over-engineering?

### 1. Câu hỏi
"Trình bày Clean/Hexagonal Architecture. Nó giải quyết vấn đề gì thật sự, và khi nào áp dụng nó là sai lầm?"

### 2. Interviewer muốn kiểm tra điều gì?
- Hiểu nguyên lý gốc (dependency rule) thay vì thuộc tên các vòng tròn.
- Đủ trưởng thành để nói về mặt trái — ứng viên chỉ ca ngợi pattern là red flag.
- Kinh nghiệm thật: đã thấy cả codebase được cứu lẫn codebase bị giết bởi nó.

### 3. Câu trả lời ngắn gọn (30 giây)
"Cả hai chung một nguyên lý: **business logic không được phụ thuộc vào chi tiết hạ tầng** — dependency chỉ theo một chiều, từ ngoài (DB, HTTP, framework) vào trong (domain), thông qua interface (port) do domain định nghĩa và adapter hiện thực ở ngoài. Lợi ích thật: test business logic không cần DB, đổi hạ tầng không đụng domain, và quan trọng nhất — buộc dev phát biểu nghiệp vụ tách khỏi kỹ thuật. Là over-engineering khi: service CRUD mỏng không có nghiệp vụ (interface + 4 layer để bọc một câu SELECT), hoặc team áp dụng máy móc đủ vòng tròn mà không hiểu mục đích — abstraction có giá, chỉ trả khi domain đủ phức tạp để hoàn vốn."

### 4. Câu trả lời Senior Level (3–5 phút)
**Problem:** codebase điển hình sau 3 năm: logic tính giá nằm trong HTTP handler, trộn query SQL, gọi thẳng SDK của vendor. Hệ quả: không test được nếu thiếu DB thật; đổi vendor là mổ toàn thân; đọc nghiệp vụ phải lọc qua rừng kỹ thuật. Gốc rễ: **phụ thuộc trỏ loạn hướng** — domain biết về hạ tầng.

**How — một nguyên lý, nhiều tên gọi:**
- **Dependency Rule:** source code dependency chỉ trỏ vào trong. Domain định nghĩa `OrderRepository` (port — interface theo ngôn ngữ nghiệp vụ); tầng infra hiện thực bằng Postgres (adapter). Đảo chiều nhờ dependency inversion: domain không import infra, infra import domain.
- Hexagonal (Ports & Adapters — Cockburn), Onion, Clean (Uncle Bob) khác nhau ở cách vẽ, giống nhau ở lõi. Nói được điều này thể hiện hiểu bản chất thay vì học từng "trường phái".
- Hệ quả thực dụng: unit test domain thuần (nhanh, ổn định — hàng nghìn test chạy trong giây); adapter test riêng bằng integration test ít hơn; thay Postgres→DynamoDB, REST→gRPC chỉ chạm adapter.

**Khi nào KHÔNG (phần ăn điểm):**
- Service CRUD thuần: 4 layer + mapping DTO↔entity↔model qua từng tầng cho logic bằng không — chi phí đọc/viết tăng, giá trị bằng 0. Transaction script thẳng tay là lựa chọn đúng.
- Trừu tượng hóa những thứ không bao giờ đổi và không cản test: bọc interface cho logger, cho chính ngôn ngữ, "phòng khi đổi DB" mà 10 năm không đổi — YAGNI. (Lưu ý tinh tế: repository interface vẫn đáng giá **vì test**, kể cả khi không bao giờ đổi DB.)
- Chi phí thật của abstraction: navigation trong code (nhảy 5 file để tìm một câu SQL), người mới khó vào, và nguy cơ "abstraction rò rỉ" — interface repository nhưng ngữ nghĩa transaction/lock của Postgres rò qua khắp nơi.

**Production:** quy tắc tôi áp cho team: bắt đầu đơn giản (handler → service → repo, 3 tầng phẳng); chỉ nâng cấp độ trừu tượng khi **đau thật** (test khó, logic trùng, đổi hạ tầng đến gần). Kiến trúc là thứ tiến hóa theo độ phức tạp của domain, không phải template ngày đầu.

### 5. Giải thích bản chất
First principles: phần mềm có hai loại code với **tốc độ thay đổi và lý do thay đổi khác nhau** — nghiệp vụ (đổi theo business) và hạ tầng (đổi theo công nghệ). Nguyên lý tổng quát: **những thứ thay đổi vì lý do khác nhau nên tách khỏi nhau; phụ thuộc nên trỏ về phía ổn định hơn**. Domain ổn định hơn framework (nghiệp vụ tính lãi sống lâu hơn Express/Gin) → domain ở tâm. Nếu đảo ngược — domain phụ thuộc framework — mỗi lần công nghệ đổi, nghiệp vụ phải viết lại: đó chính là "legacy system" theo nghĩa đau đớn nhất. Mọi pattern kiến trúc chỉ là cách hiện thực nguyên lý này; hiểu nguyên lý thì không cần thuộc pattern.

### 6. Trade-off
- **Testability + independence ↔ indirection:** thêm mỗi interface là thêm một bước nhảy khi đọc code, thêm mapping, thêm file.
- **Bảo vệ domain ↔ tốc độ ship ban đầu:** dự án 2 tuần proof-of-concept không cần hexagonal; hệ core banking 10 năm thì rất cần.
- **Kỷ luật đồng đều ↔ thực tế team:** kiến trúc chỉ sống khi cả team hiểu tại sao; áp đặt bằng template mà không hiểu → "hexagonal về hình thức, mì spaghetti về nội dung" (logic nghiệp vụ lén nằm trong adapter).

### 7. Ví dụ Production
Hai câu chuyện đối xứng tôi chứng kiến: (1) Hệ tính cước viễn thông — domain phức tạp khủng khiếp, được tách sạch theo hexagonal: khi migrate Oracle→Postgres, đội chỉ viết lại adapter trong 6 tuần, 4.000 unit test domain không đổi một dòng — khoản đầu tư 5 năm trước hoàn vốn một cú. (2) Startup 8 người áp Clean Architecture đầy đủ cho app CRUD: mỗi endpoint mới cần chạm 7 file (controller, usecase, port, adapter, 3 DTO); tốc độ ship giảm một nửa, và vì không có nghiệp vụ thật, các "usecase" chỉ là hàm chuyển tiếp. Sáu tháng sau họ đập đi làm lại phẳng hơn. Kiến trúc đúng ở chỗ này là sai ở chỗ khác — **độ phức tạp của giải pháp phải khớp độ phức tạp của bài toán**.

### 8. Những câu trả lời chưa đủ tốt
- Vẽ lại 4 vòng tròn và kể tên layer. → Thuộc bài. Câu hỏi thật là: dependency rule để làm gì, và cái giá?
- "Giúp code sạch, dễ maintain." → Sáo rỗng. Cụ thể: test không cần DB? đổi adapter? tách tốc độ thay đổi? Phải nói được cơ chế.

### 9. Sai lầm phổ biến của ứng viên
- Nghĩ hexagonal = cấu trúc thư mục. Nó là hướng của dependency, không phải tên folder.
- Domain "thuần" nhưng entity lại là struct của ORM (annotation dính đầy) — phụ thuộc ngầm chưa đảo.
- Bọc interface mọi thứ kể cả thứ không cần test/đổi — hiểu máy móc dependency inversion.
- Không phân biệt DTO tầng API và entity domain — hoặc ngược lại, mapping cuồng tín 5 lớp cho cùng một hình dạng dữ liệu.
- Không nói được khi nào KHÔNG dùng — dấu hiệu chưa từng trả giá cho abstraction thừa.

### 10. Follow-up Questions
- Transaction xuyên nhiều repository xử lý ở tầng nào? (unit of work — và tại sao để usecase mở transaction là rò rỉ hạ tầng vào domain? Các cách dung hòa?)
- Domain event từ tầng domain phát ra ngoài thế nào mà không phụ thuộc message broker?
- Nếu ORM entity ≠ domain entity, mapping đặt đâu và chi phí gì? Khi nào chấp nhận dùng chung?
- Hexagonal trong microservices: mỗi service một hexagon? Chia sẻ domain model giữa service được không? (không — bounded context.)
- Anh từng gỡ một kiến trúc quá phức tạp chưa? Quy trình đơn giản hóa mà không dừng ship tính năng?

### 11. Liên hệ với Production
Các hệ core (banking, billing, pricing) sống hàng thập kỷ đều hội tụ về dạng này — vì hạ tầng đã đổi 3 thế hệ còn nghiệp vụ thì không. Vấn đề nghiêm trọng khi: codebase không test được nếu thiếu môi trường đầy đủ (dấu hiệu domain dính hạ tầng), hoặc ngược lại — velocity giảm vì boilerplate (dấu hiệu abstraction thừa). Dấu hiệu cần rà soát: thời gian viết test > thời gian viết code cho logic đơn giản; import graph có domain trỏ ra infra; sự cố do logic nghiệp vụ trùng lặp ở nhiều adapter.

---

## Câu 2 — [Senior → Staff] DDD: Bounded Context và Aggregate — hai khái niệm đáng giá nhất

### 1. Câu hỏi
"Trong DDD, Bounded Context và Aggregate là gì? Tại sao tôi nói đây là hai khái niệm quyết định thành bại của microservices?"

### 2. Interviewer muốn kiểm tra điều gì?
- DDD chiến lược (bounded context) — thứ đắt giá — thay vì DDD chiến thuật hình thức (repository, value object).
- Hiểu aggregate như ranh giới nhất quán — nối thẳng tới transaction, lock, và thiết kế phân tán.
- Khả năng nói chuyện với business — ubiquitous language không phải khẩu hiệu.

### 3. Câu trả lời ngắn gọn (30 giây)
"**Bounded Context**: ranh giới trong đó một mô hình và một ngôn ngữ có nghĩa nhất quán — 'Sản phẩm' của team Catalog (mô tả, ảnh, SEO) khác hẳn 'Sản phẩm' của team Kho (tồn, vị trí kệ) và của team Pricing; ép một model chung cho tất cả tạo ra god-object mà mọi thay đổi đều đụng mọi team. Microservices chia đúng = chia theo bounded context. **Aggregate**: cụm object thay đổi cùng nhau như một đơn vị nhất quán, có một root kiểm soát mọi truy cập và bất biến (Order chứa OrderLines: 'tổng ≤ hạn mức' phải đúng nguyên tử) — aggregate là đơn vị của transaction và lock. Ranh giới aggregate sai → hoặc transaction phình to (contention), hoặc bất biến bị xé (bug tiền bạc)."

### 4. Câu trả lời Senior Level (3–5 phút)
**Bounded Context — DDD chiến lược:**
- Sai lầm chết người của mô hình hóa: tìm "một model đúng cho toàn công ty". Thực tế mỗi bộ phận nhìn cùng khái niệm với thuộc tính, vòng đời, bất biến khác nhau. Model chung = hợp nhất mọi mối quan tâm = bảng `products` 200 cột mà 5 team cùng migrate — coupling tổ chức trá hình dưới dạng schema.
- Bounded context hợp pháp hóa sự đa nghĩa: mỗi context một model tối ưu cho mối quan tâm của nó; giữa các context là **context map** với quan hệ tường minh: Anti-corruption Layer (dịch model kẻ khác thành model của mình — chặn khái niệm ngoại lai xâm thực), Customer-Supplier, Published Language (event schema chung).
- Đây chính là câu trả lời cho "chia microservice thế nào": ranh giới service = ranh giới ngôn ngữ nghiệp vụ. Chia theo entity kỹ thuật (user-service, product-service kiểu CRUD) tạo ra distributed monolith — mọi tính năng chạm 4 service.

**Aggregate — DDD chiến thuật phần đáng tiền:**
- Định nghĩa qua bất biến: những dữ liệu nào phải đúng **cùng nhau, ngay lập tức**? Chúng thuộc một aggregate, sửa qua root, lưu trong một transaction. Những gì nhất quán được **sau một lúc** (eventual) → aggregate khác, đồng bộ qua domain event.
- Quy tắc thực dụng: aggregate nhỏ nhất có thể; tham chiếu aggregate khác bằng ID, không object; một transaction chạm một aggregate — vi phạm quy tắc cuối là dấu hiệu ranh giới sai.
- Nối với thế giới thật: aggregate = đơn vị lock (contention trên aggregate to = throughput chết — Order chứa toàn bộ lịch sử 10 năm của khách là aggregate sai); = document trong MongoDB; = partition key tự nhiên trong Kafka/DynamoDB; = entity trong event sourcing. **Một khái niệm DDD trả lời bốn câu hỏi hạ tầng.**

**Trade-off & Production:** DDD đắt: workshop với domain expert (event storming), kỷ luật ngôn ngữ, ACL code thêm. Đáng cho domain phức tạp nghiệp vụ (logistics, bảo hiểm, ngân hàng); thừa cho CRUD/CMS. DDD-lite (chỉ bounded context + aggregate + domain event, bỏ qua nghi lễ) là điểm hiệu quả/chi phí tốt nhất theo kinh nghiệm của tôi.

### 5. Giải thích bản chất
Bounded context là **thừa nhận giới hạn nhận thức**: không tồn tại mô hình toàn cục nhất quán cho một tổ chức đủ lớn — cũng như không tồn tại schema database toàn công ty. Mọi mô hình là sự đơn giản hóa **cho một mục đích**; mục đích khác nhau → mô hình khác nhau là tất yếu, không phải thất bại. Aggregate là **thừa nhận giới hạn của tính nhất quán**: nhất quán tức thời chỉ rẻ trong phạm vi nhỏ (một máy, một lock, một transaction); kéo dài phạm vi là kéo dài lock là giết concurrency. Cả hai khái niệm cùng một triết lý: **vẽ ranh giới tường minh ở nơi chi phí vượt lợi ích** — và đó là lý do chúng ánh xạ hoàn hảo sang hệ phân tán, nơi mọi chi phí đều bị khuếch đại.

### 6. Trade-off
- **Model riêng per context:** tối ưu từng nơi, team tự chủ ↔ dữ liệu trùng lặp có chủ đích, dịch thuật giữa context (ACL) là code phải nuôi.
- **Aggregate nhỏ:** concurrency cao, transaction gọn ↔ nhiều bất biến thành eventual — nghiệp vụ phải chấp nhận và xử lý cửa sổ không nhất quán.
- **Ubiquitous language:** giảm lỗi dịch thuật dev–business ↔ đòi kỷ luật và thời gian với domain expert — thứ nhiều tổ chức không chịu trả.

### 7. Ví dụ Production
Hệ thương mại điện tử tôi tham gia refactor: bảng `orders` là god-model — checkout, kho, kế toán, CSKH cùng đọc ghi. Mỗi migration là một cuộc đàm phán 4 team, mỗi thay đổi cột là nguy cơ sự cố chéo. Tách theo context: Checkout giữ `Order` (giỏ, giá, thanh toán), Fulfillment có `Shipment` riêng (nhận OrderPlaced event, dựng model của mình), Accounting có `Invoice`. Dữ liệu "trùng" (địa chỉ xuất hiện 3 nơi) — nhưng mỗi bản phục vụ bất biến riêng và **các team ngừng chờ nhau**: velocity đo được tăng, sự cố chéo team giảm hẳn. Chi phí thật: 6 tháng, một ACL cho hệ legacy, và các cuộc tranh luận "Order nghĩa là gì" — chính các cuộc tranh luận đó là giá trị: chúng phơi bày sự mơ hồ đã gây bug hàng năm trời.

### 8. Những câu trả lời chưa đủ tốt
- Liệt kê entity/value object/repository/service. → DDD chiến thuật hình thức — phần ít giá trị nhất. Bounded context mới là 80% giá trị.
- "DDD là thiết kế theo domain." → Tautology. Cụ thể: ranh giới ngôn ngữ ở đâu, bất biến nào cần nguyên tử, quan hệ giữa các context là gì?

### 9. Sai lầm phổ biến của ứng viên
- Nghĩ cả công ty phải thống nhất một định nghĩa "Customer" — ngược tinh thần bounded context.
- Aggregate to bằng cả object graph (Order → Customer → mọi Order khác của Customer) — lock cả thế giới.
- Tham chiếu object trực tiếp giữa aggregate thay vì ID → ORM lazy-load kéo nửa database.
- Áp DDD cho mọi service kể cả CRUD thuần — nghi lễ không có nghiệp vụ.
- Bỏ qua context map — có ranh giới nhưng quan hệ giữa chúng mù mờ, ACL không ai viết, model ngoại lai xâm thực dần.

### 10. Follow-up Questions
- Event storming là gì, chạy một buổi thế nào, output là gì?
- Bất biến "email duy nhất toàn hệ thống" — thuộc aggregate nào? (không aggregate nào — cần cơ chế khác: unique index, saga kiểm tra, hoặc chấp nhận và xử lý trùng — thảo luận này rất lộ trình độ.)
- Hai bounded context cần "cùng một dữ liệu" real-time — các lựa chọn và trade-off? (event + local copy, API call đồng bộ, shared DB — tại sao cái cuối nguy hiểm nhất.)
- Aggregate trong event sourcing đóng vai gì? (đơn vị của stream + replay.)
- Làm sao thuyết phục business bỏ thời gian làm ubiquitous language? (câu Principal — bán bằng chi phí của bug do hiểu nhầm.)

### 11. Liên hệ với Production
Các sách kiến trúc microservices thành công (Amazon "two-pizza team sở hữu service", cấu trúc team của Spotify) về bản chất là bounded context + Conway's Law vận dụng có chủ đích. Vấn đề nghiêm trọng khi: một thay đổi nghiệp vụ nhỏ chạm ≥3 service (ranh giới sai), các team tranh cãi bất tận về schema chung, hoặc deadlock/contention tập trung trên vài row nóng (aggregate quá to). Dấu hiệu cần hành động: "shared library domain model" dùng chung nhiều service (coupling trá hình), meeting liên team cho mọi thay đổi model, và từ vựng trong code lệch xa từ vựng của business.

---

## Câu 3 — [Senior → Staff] Microservices vs Monolith: chi phí thật của phân tán

### 1. Câu hỏi
"Khi nào nên chuyển từ monolith sang microservices? Chi phí thật của microservices mà người ta hay đánh giá thấp là gì?"

### 2. Interviewer muốn kiểm tra điều gì?
- Đã trả giá thật với hệ phân tán chưa, hay chỉ đọc blog.
- Hiểu động lực đúng (tổ chức, scale độc lập) vs động lực sai (trend, CV).
- Tư duy Staff: chi phí vận hành, con người, và con đường di trú an toàn.

### 3. Câu trả lời ngắn gọn (30 giây)
"Microservices giải bài toán **tổ chức** trước bài toán kỹ thuật: cho phép nhiều team deploy độc lập không giẫm chân nhau, scale và chọn công nghệ theo từng phần. Chi phí thật bị đánh giá thấp: mọi function call thành network call (thêm latency, thêm failure mode, mất transaction), debugging thành điều tra phân tán (bắt buộc tracing), mỗi service kéo theo CI/CD + monitoring + on-call, và dữ liệu bị chia — join thành API composition, nhất quán thành eventual. Quy tắc của tôi: monolith có cấu trúc tốt (modular monolith) cho đến khi có **bằng chứng** đau: team > 20–30 dev giẫm chân nhau khi deploy, hoặc một phần hệ thống cần scale/độ tin cậy khác hẳn phần còn lại. Tách vì đau thật, không tách vì sợ đau tương lai."

### 4. Câu trả lời Senior Level (3–5 phút)
**Động lực đúng:**
1. **Tổ chức (mạnh nhất):** 50 dev trên một codebase — merge conflict, release train, một bug chặn cả đoàn tàu. Deploy độc lập là giá trị số một của microservices, và nó chỉ đến khi ranh giới đúng (bounded context — câu trước).
2. **Scale không đồng nhất:** search cần 50 node CPU, upload cần I/O, còn admin panel cần 2 pod — monolith buộc scale cả khối theo phần háu đói nhất.
3. **Độ tin cậy khác nhau:** payment cần 99.99%, recommendation chết cũng không sao — cô lập blast radius.

**Chi phí thật (phần lớn ứng viên nói được 2 cái đầu, Senior nói được cả danh sách):**
- **Network thay function call:** latency (µs→ms), partial failure (một hàm không thể "timeout", một RPC thì có), phải có retry + timeout budget + circuit breaker cho **từng** cạnh gọi — độ phức tạp nhân theo số cạnh, không số node.
- **Mất transaction:** monolith bọc 3 bảng trong một ACID transaction; microservices thành saga/outbox/eventual — mỗi luồng ghi xuyên service là một bài toán thiết kế riêng (xem câu Saga).
- **Dữ liệu chia cắt:** "JOIN 2 bảng" thành gọi 2 API và ghép tay (API composition) hoặc nhân bản dữ liệu qua event — cả hai đắt hơn JOIN hàng trăm lần về công sức.
- **Chi phí nền tảng:** service mesh/discovery, tracing, log tập trung, CI/CD per service, contract test, secret management — cần platform team thật sự; dưới một quy mô nhất định, chi phí này lớn hơn toàn bộ lợi ích.
- **Chi phí nhận thức:** không ai còn hiểu toàn hệ thống; onboarding lâu hơn; "sửa bug này ở service nào?" thành câu hỏi triết học.
- **Distributed monolith** — kết cục tồi tệ nhất: chia service nhưng ranh giới sai → mọi tính năng chạm 4 service, deploy phải xếp lịch cùng nhau, chậm hơn monolith mà đắt hơn monolith. Tệ hơn cả hai thế giới.

**Con đường đúng:** modular monolith trước (module ranh giới rõ, giao tiếp qua interface nội bộ, DB schema tách theo module — cấm join xuyên module); khi cần tách, ranh giới đã có sẵn, tách bằng strangler fig (định tuyến dần từng luồng sang service mới, không big-bang rewrite).

### 5. Giải thích bản chất
Không có phép màu nào ở đây — chỉ là **định luật bảo toàn độ phức tạp**: độ phức tạp của bài toán không giảm khi chia hệ thống; nó chỉ **di chuyển** từ trong code (module, function call) ra **giữa** các hệ thống (network, contract, vận hành). Sự di chuyển này đáng giá khi nó đổi loại phức tạp mà tổ chức của bạn xử lý giỏi lấy loại xử lý kém: 50 dev quản code chung rất kém (phức tạp trong code đắt) nhưng một platform team giỏi có thể chuẩn hóa vận hành (phức tạp giữa hệ thống rẻ đi). Ngược lại với team 8 người, phức tạp trong code rẻ (họp một bàn là xong) còn phức tạp phân tán đắt vô cùng. **Cùng một kiến trúc, chi phí–lợi ích đảo dấu theo quy mô tổ chức** — vì thế câu hỏi "monolith hay microservices" không có đáp án, chỉ có ngữ cảnh. Kèm Conway's Law: kiến trúc rồi sẽ phản chiếu cấu trúc giao tiếp của tổ chức — nên thiết kế tổ chức và kiến trúc phải đi cùng nhau (Inverse Conway Maneuver).

### 6. Trade-off
- **Deploy độc lập ↔ contract management:** không còn release train, nhưng breaking change giờ là sự cố runtime giữa các team (cần consumer-driven contract test, versioning kỷ luật).
- **Scale/tech độc lập ↔ chuẩn hóa:** mỗi service một stack nghe hay — 5 năm sau là bảo tàng công nghệ không ai maintain; cần golden path.
- **Cô lập lỗi ↔ failure mode mới:** service A chết không sập B — nhưng xuất hiện cascading failure, retry storm, thundering herd — các loại chết mà monolith không có.
- **Autonomy ↔ nhất quán dữ liệu:** mỗi team một DB ↔ báo cáo xuyên domain cần data platform riêng (CDC, warehouse).

### 7. Ví dụ Production
Câu chuyện thật ngành này lặp đi lặp lại: công ty ~30 dev tách monolith thành 40 microservices theo entity (user, product, order, notification...). Một năm sau: tạo đơn hàng đi qua 7 service đồng bộ; p99 từ 80ms lên 900ms; sự cố "đơn treo" cần 3 team điều tra 2 ngày (thiếu tracing); một dev nghỉ việc mang theo toàn bộ hiểu biết về 6 service. Họ merge ngược về 8 service theo bounded context — con số không quan trọng, **ranh giới quan trọng**. Ngược lại: sàn TMĐT lớn tách riêng mỗi search + checkout khỏi monolith vì hai phần này cần scale và SLA khác hẳn — 3 service tổng cộng, lợi ích rõ, chi phí kiểm soát được. Bài học: đếm số service của một công ty thành công rồi bắt chước con số là học vẹt; thứ cần bắt chước là **kỷ luật ranh giới**.

### 8. Những câu trả lời chưa đủ tốt
- "Microservices giúp scale tốt hơn." → Scale cái gì? Monolith chạy 200 pod sau LB scale ngon lành. Phần lớn lợi ích scale là scale **team**, không phải scale máy.
- "Monolith là legacy, microservices là hiện đại." → Shopify, Stack Overflow, Basecamp chạy monolith/modular monolith ở quy mô khổng lồ. Câu này là red flag nặng.

### 9. Sai lầm phổ biến của ứng viên
- Chia service theo entity kỹ thuật thay vì bounded context → distributed monolith.
- Không nhắc chi phí dữ liệu (mất join, mất transaction) — phần đắt nhất.
- Big-bang rewrite thay vì strangler fig.
- Shared database giữa các service ("tách code nhưng chung DB") — coupling nguyên vẹn, chi phí network cộng thêm.
- Không đặt câu hỏi về quy mô team — kiến trúc tách rời khỏi tổ chức.

### 10. Follow-up Questions
- Strangler fig cụ thể: định tuyến ở đâu, dữ liệu đồng bộ hai chiều thế nào trong giai đoạn chuyển tiếp?
- Hai service cần dữ liệu của nhau liên tục — dấu hiệu gì và sửa thế nào? (ranh giới sai → merge; hoặc data duplication qua event.)
- Contract testing (Pact) hoạt động thế nào, đặt ở đâu trong CI?
- Chi phí một service mới ở công ty anh là bao nhiêu (thời gian từ ý tưởng tới production)? — câu này lộ ngay mức đầu tư platform.
- Nếu được làm lại từ đầu ở công ty gần nhất, anh giữ mấy service? (câu tự phản tư — chấm sự trung thực.)

### 11. Liên hệ với Production
Amazon và Netflix thành công với microservices ở quy mô nghìn dev + platform đầu tư khổng lồ; Shopify thành công với modular monolith + đầu tư kỷ luật module; Segment công khai bài "chúng tôi merge microservices về monolith". Vấn đề nghiêm trọng khi: thời gian debug sự cố tăng theo số service, tính năng trung bình cần sửa ≥3 repo, hạ tầng chiếm >30% năng lực kỹ sư ở công ty nhỏ. Dấu hiệu cần hành động: deploy các service phải xếp lịch cùng nhau (coupling lộ diện), số cuộc gọi đồng bộ trong một request tăng đều theo quý, và chi phí cloud tăng nhanh hơn traffic.

---

## Câu 4 — [Senior → Staff] Event-Driven Architecture, CQRS và Event Sourcing: sức mạnh và cái giá

### 1. Câu hỏi
"Event-driven architecture giải quyết gì? CQRS là gì và khi nào cần? Event sourcing khác gì và tại sao tôi khuyên phần lớn team KHÔNG dùng nó?"

### 2. Interviewer muốn kiểm tra điều gì?
- Phân biệt rạch ròi ba khái niệm hay bị trộn lẫn: EDA ≠ CQRS ≠ Event Sourcing.
- Hiểu eventual consistency như hệ quả tất yếu và cách sống chung.
- Trưởng thành để nói "không dùng" — event sourcing là nơi over-engineering tụ tập.

### 3. Câu trả lời ngắn gọn (30 giây)
"**EDA**: các service giao tiếp bằng event bất đồng bộ thay vì gọi nhau đồng bộ — producer không biết consumer → decoupling thời gian lẫn cấu trúc, chịu lỗi tốt hơn (consumer chết, event chờ trong log), thêm consumer mới không sửa producer. Giá: eventual consistency, luồng nghiệp vụ vô hình (không còn call stack), debugging khó. **CQRS**: tách model ghi và model đọc — ghi vào model chuẩn hóa giữ bất biến, đọc từ projection denormalize tối ưu cho từng màn hình; cần khi read/write pattern lệch nhau nghiêm trọng. **Event sourcing**: không lưu state hiện tại mà lưu chuỗi event, state = replay — audit hoàn hảo, time-travel, nhưng chi phí nhận thức và vận hành rất cao (schema evolution của event vĩnh viễn, snapshot, GDPR xóa dữ liệu trong log bất biến). Ba thứ độc lập: dùng được EDA + CQRS mà không event sourcing — và đa số nên dừng ở đó."

### 4. Câu trả lời Senior Level (3–5 phút)
**EDA:**
- Chuyển từ "orchestration đồng bộ" (A gọi B gọi C — A chờ tất cả, availability = tích các availability) sang "phát sự kiện" (A ghi + phát OrderPlaced; B, C, D tự nghe). A nhanh và sống độc lập; thêm consumer thứ 5 không ai phải biết.
- Kỹ thuật bắt buộc đi kèm (thiếu là vỡ): **outbox pattern** (ghi DB + event nguyên tử — ghi event vào bảng outbox cùng transaction, relay/CDC đẩy sang broker; không có nó là dual-write, nguồn lệch dữ liệu số một), **idempotent consumer** (at-least-once là mặc định thực tế), schema registry + compatibility.
- Cái giá ít người nói: **luồng nghiệp vụ tan vào không khí** — "điều gì xảy ra sau khi đặt hàng?" không còn trả lời được bằng đọc code một chỗ; cần tracing xuyên event, documentation luồng, và những kỹ sư giữ bức tranh tổng trong đầu.

**CQRS:**
- Động lực: model ghi tốt (chuẩn hóa, bất biến) thường là model đọc tồi (màn hình cần join 8 bảng), và tỷ lệ đọc/ghi thường 100:1 — tối ưu hai chiều trên một model là kéo co.
- Phổ áp dụng từ nhẹ tới nặng: (1) chỉ tách code path đọc/ghi trên cùng DB; (2) read replica cho query; (3) projection denormalize riêng (Elasticsearch cho search, Redis cho feed, bảng phẳng cho dashboard) cập nhật qua event. Mức 3 mới là CQRS đầy đủ — và kèm eventual consistency giữa ghi và đọc: UI phải xử lý "vừa đặt hàng xong chưa thấy trong danh sách" (optimistic UI, read-your-writes routing).
- Không cần event sourcing để làm CQRS — projection dựng từ event **phát ra** từ hệ CRUD bình thường (hoặc CDC) là đủ.

**Event Sourcing:**
- Lưu `OrderCreated, ItemAdded, ItemRemoved, OrderConfirmed` — state hiện tại = fold(events). Được: audit trail tuyệt đối (là dữ liệu chính, không phải log phụ), truy vấn quá khứ ("giỏ hàng lúc 3giờ chiều hôm qua"), dựng projection mới từ lịch sử đầy đủ, debug bằng replay.
- Giá thật (lý do tôi khuyên phần lớn team tránh): event schema là **hợp đồng vĩnh viễn** — nghiệp vụ đổi, event cũ vẫn phải replay được (upcasting, versioning — mãi mãi); mọi query cần projection (không SELECT nổi thứ chưa dựng); snapshot cho stream dài; xóa dữ liệu cá nhân (GDPR) trong store bất biến là bài toán mở (crypto-shredding); và chi phí đào tạo — mọi dev mới phải học lại cách nghĩ. Đáng ở domain mà **bản thân lịch sử là nghiệp vụ** (sổ cái ngân hàng, trading, kế toán — vốn dĩ là event sourcing từ thời giấy bút); thừa ở nơi chỉ cần audit log (bảng audit riêng rẻ hơn 10 lần).

### 5. Giải thích bản chất
Cả ba xoay quanh một câu hỏi: **sự thật của hệ thống nằm ở đâu?** CRUD truyền thống: sự thật = state hiện tại, quá khứ bị ghi đè mất. Event sourcing: sự thật = lịch sử thay đổi, state chỉ là cache dẫn xuất — đúng bản chất của kế toán, của WAL, của Git. EDA: sự thật được **loan báo** thay vì **hỏi thăm** — đổi mô hình kéo (query) sang đẩy (publish), và mọi hệ quả (decoupling, eventual, khó debug) đều từ đó. CQRS thừa nhận: "sự thật để giữ đúng" và "sự thật để trình bày" có hình dạng tối ưu khác nhau — cùng insight với ClickHouse vs Postgres, index vs bảng, cache vs DB. Nhìn thấy các pattern này là **một ý tưởng lặp lại ở mọi tầng** (derived data từ log) chính là trình độ Staff: Kafka, WAL, materialized view, CQRS projection, cache — tất cả là một họ.

### 6. Trade-off
- **EDA:** decoupling + resilience ↔ eventual consistency, mất call stack, ordering phải thiết kế (per-key), chi phí broker + governance.
- **CQRS:** đọc và ghi cùng tối ưu ↔ hai model phải nuôi, sync lag, code nhân đôi cho luồng đơn giản.
- **Event sourcing:** lịch sử là tài sản ↔ chi phí nhận thức vĩnh viễn, schema evolution vĩnh viễn, tooling tự xây nhiều.
- **Đồng bộ vs bất đồng bộ tổng quát:** đồng bộ dễ reason, fail nhanh và rõ ↔ bất đồng bộ chịu lỗi tốt, scale tốt, fail chậm và mờ. Chọn per-luồng: luồng cần câu trả lời ngay (thanh toán authorize) đồng bộ; luồng hệ quả (email, tích điểm, đồng bộ kho) bất đồng bộ.

### 7. Ví dụ Production
Case chuyển đổi thật: luồng đặt hàng gọi đồng bộ 6 service (kho, giá, khuyến mãi, điểm, email, phân tích) — availability tích lũy 97%, p99 1.8s, mỗi service mới thêm vào là sửa checkout. Chuyển sang outbox + Kafka: checkout chỉ ghi DB + phát event (p99 120ms, availability chỉ phụ thuộc DB); 5 consumer tự xử lý. Sự cố sau đó mới dạy bài học thật: consumer tích điểm bị bug xử lý trùng (thiếu idempotency) → cộng điểm gấp đôi cho 8.000 đơn — **at-least-once không phải chi tiết hiện thực, nó là ngữ nghĩa nghiệp vụ phải thiết kế**. Và một bài học nữa: khi PM hỏi "tại sao khách đặt xong 5 giây mới có email?", câu trả lời "eventual consistency" phải được nói **trước khi ký thiết kế**, không phải sau sự cố kỳ vọng.

### 8. Những câu trả lời chưa đủ tốt
- Trộn ba khái niệm thành một ("EDA là khi dùng event sourcing với CQRS...") → rớt ngay phần định nghĩa.
- "Event sourcing cho audit tốt." → Bảng audit log thường đủ cho 90% nhu cầu audit với 10% chi phí. Phải so sánh với phương án rẻ trước khi chọn phương án đắt.

### 9. Sai lầm phổ biến của ứng viên
- Dual-write (ghi DB xong publish Kafka trong hai thao tác rời) — không biết outbox; đây là câu sàng lọc kinh nghiệm EDA thật.
- Giả định event đến đúng thứ tự và đúng một lần — thiết kế consumer không idempotent.
- CQRS mức nặng cho app 100 user — hai model, một nỗi đau, không lợi ích.
- Event mang cả state thế giới (fat event 2MB) hoặc quá gầy (chỉ ID — consumer dội ngược API, thành coupling đồng bộ trá hình). Không biết trade-off event notification vs event-carried state transfer.
- Không có chiến lược replay/backfill khi projection hỏng hoặc cần dựng mới.

### 10. Follow-up Questions
- Outbox pattern chi tiết: relay polling vs CDC? Bảng outbox dọn thế nào? Ordering đảm bảo sao?
- Projection bị bug 3 ngày mới phát hiện — quy trình rebuild? (replay từ đâu, downtime cho read model, versioning projection.)
- Event schema evolution: thêm field? đổi nghĩa field? consumer cũ mới chạy song song?
- Chuỗi event xuyên service cần "hoàn tác" khi bước cuối fail — dẫn sang saga (câu sau).
- GDPR yêu cầu xóa user khỏi event store bất biến — các phương án? (crypto-shredding: mã hóa PII per-user, xóa key.)

### 11. Liên hệ với Production
LinkedIn/Uber xây cả nền tảng quanh log trung tâm (Kafka là xương sống dữ liệu); các ngân hàng lõi vốn dĩ event-sourced từ trước khi có tên gọi (sổ cái bút toán). Vấn đề nghiêm trọng khi: consumer lag làm projection cũ hàng giờ (UX vỡ), event không governance (nghìn schema không ai own), hoặc team nhỏ ngập trong hạ tầng event cho bài toán CRUD. Dấu hiệu cần hành động: tỷ lệ sự cố "dữ liệu lệch giữa hai hệ" tăng (thiếu outbox/idempotency), thời gian truy vết một luồng nghiệp vụ vượt giờ, và câu hỏi "event này ai consume?" không ai trả lời được.

---

## Câu 5 — [Staff] Saga Pattern: transaction phân tán mà không có 2PC

### 1. Câu hỏi
"Đơn hàng cần: trừ kho, trừ tiền, tạo vận đơn — ba service khác nhau. Không có distributed transaction, anh đảm bảo tính đúng đắn thế nào? Choreography hay orchestration?"

### 2. Interviewer muốn kiểm tra điều gì?
- Bài toán trung tâm của microservices: nhất quán xuyên service. Ai chưa từng giải nó thì chưa vận hành microservices thật.
- Hiểu tại sao không dùng 2PC, compensating action, và các trạng thái dở dang.
- Tư duy về semantic của nghiệp vụ: không phải mọi bước đều hoàn tác được.

### 3. Câu trả lời ngắn gọn (30 giây)
"Saga: chuỗi transaction cục bộ, mỗi bước commit thật ở service của nó; bước sau fail thì chạy **compensating action** hoàn tác các bước trước theo nghĩa nghiệp vụ (hoàn tiền, cộng lại kho) — không phải rollback kỹ thuật. Không dùng 2PC vì nó giữ lock qua network round-trip nhiều pha, coordinator chết là tất cả treo — availability không chấp nhận được. Hai kiểu điều phối: **choreography** — các service nghe event của nhau, không não trung tâm, decoupled nhưng luồng vô hình, dễ thành mạng nhện; **orchestration** — một orchestrator giữ state machine gọi từng bước, luồng tường minh, dễ vận hành/timeout/retry, đổi lại thêm một thành phần trung tâm. Kinh nghiệm của tôi: saga ≥3 bước hoặc có compensation phức tạp → orchestration; 2 bước đơn giản → choreography đủ."

### 4. Câu trả lời Senior Level (3–5 phút)
**Tại sao không 2PC:** prepare phase giữ lock/resource ở **mọi** participant chờ pha commit; coordinator chết giữa chừng → participant kẹt in-doubt (giữ lock không dám nhả); availability = tích của mọi participant + coordinator. Trên mạng WAN/service không đồng nhất, đây là công thức treo hệ thống — vì thế cả ngành từ bỏ nó cho luồng nghiệp vụ (XA còn sót lại ở một số hệ nội bộ đồng nhất, nơi nó vẫn hợp lý).

**Saga đúng cách:**
- Mỗi bước là local ACID transaction + phát event/lệnh cho bước sau (qua outbox!). Fail ở bước k → chạy compensation cho k-1..1.
- **Compensation là nghiệp vụ, không phải kỹ thuật:** "trừ tiền" hoàn tác bằng "hoàn tiền" (một giao dịch mới, có phí, có thông báo khách) — không phải xóa bản ghi. Có bước **không hoàn tác được** (đã gửi email, đã bắn hàng lên xe) → thứ tự bước là quyết định thiết kế: xếp bước khó hoàn tác **cuối cùng** (pivot transaction); trước pivot là các bước compensable, sau pivot chỉ được phép là các bước retry-được-đến-thành-công (retriable).
- **Trạng thái trung gian lộ ra ngoài:** giữa bước 1 và 3, khách thấy "tiền bị trừ, đơn chưa xác nhận". Saga không có isolation của ACID — phải thiết kế semantic lock (đơn ở trạng thái PENDING, kho ở trạng thái RESERVED thay vì trừ thẳng — reserve/confirm/release là bộ ba kinh điển).
- **Orchestration thực dụng:** state machine bền vững (lưu DB hoặc dùng temporal/cadence-style engine), mỗi bước có timeout + retry policy + max attempts → dead letter + cảnh báo con người. Orchestrator chết → tỉnh dậy đọc state đi tiếp (recovery tự nhiên). Câu hỏi phỏng vấn ngược hay gặp: "orchestrator có phải SPOF không?" — không, nó stateless-recoverable nếu state trong DB; SPOF thật là **thiếu** state bền vững.
- **Idempotency mọi bước** (retry là mặc định) + **correlation ID** xuyên saga để trace.

### 5. Giải thích bản chất
ACID transaction là một ảo giác đắt tiền: "mọi thứ xảy ra tức thời hoặc không gì xảy ra". Trong một máy, lock rẻ nên ảo giác rẻ. Xuyên network, duy trì ảo giác đòi giữ lock qua độ trễ mạng và sự cố — giá tăng phi mã (2PC). Saga là **thừa nhận sự thật**: trong hệ phân tán, mọi thứ xảy ra **dần dần**, và thay vì che giấu điều đó, ta mô hình hóa nó tường minh — trạng thái trung gian có tên (PENDING, RESERVED), sự cố có kịch bản (compensation), thời gian có ngân sách (timeout). Thú vị nhất: **thế giới thật vận hành bằng saga từ nghìn năm** — đặt cọc, giữ chỗ, hoàn tiền, phạt cọc — nghiệp vụ vốn hiểu eventual consistency giỏi hơn kỹ sư; nhiệm vụ của ta là hỏi business "khi bước 3 hỏng thì công ty muốn làm gì?" thay vì giấu họ sau chữ "transaction".

### 6. Trade-off
- **Choreography:** decoupled tối đa, thêm bước không sửa não trung tâm ↔ luồng vô hình (hiểu saga = đọc N codebase), khó trả lời "saga này đang ở đâu", vòng lặp event vô tình.
- **Orchestration:** luồng tường minh một chỗ, vận hành/giám sát/timeout dễ ↔ thêm thành phần, nguy cơ orchestrator phình thành god-service chứa nghiệp vụ của người khác.
- **Saga vs 2PC:** availability + không lock xuyên mạng ↔ mất isolation, phải thiết kế compensation cho từng bước — chi phí thiết kế trả trước.
- **Reserve/confirm vs trừ thẳng:** trạng thái trung gian sạch ↔ mọi resource cần TTL cho reservation (giữ chỗ 15 phút) và job dọn.

### 7. Ví dụ Production
Sự cố dạy tôi nhiều nhất: saga đặt vé (giữ ghế → trừ tiền → xuất vé), compensation đầy đủ. Một ngày, bước trừ tiền **timeout** — không fail, không success, không biết. Code coi timeout là fail → chạy compensation nhả ghế → nhưng lệnh trừ tiền thật ra đã thành công ở phía cổng thanh toán (response lạc trên đường về) → khách mất tiền không có vé. Bài học khắc cốt: **timeout không phải failure — nó là unknown**. Xử lý đúng: bước có tiền phải có **truy vấn trạng thái** (query cổng thanh toán trước khi quyết định compensation) hoặc thiết kế reconciliation định kỳ đối soát mọi giao dịch dở dang. Từ đó tôi luôn yêu cầu mọi saga có bảng trạng thái + job đối soát + alert cho saga kẹt quá X phút — saga không có đối soát là saga chưa xong.

### 8. Những câu trả lời chưa đủ tốt
- "Dùng saga để rollback qua các service." → Chữ 'rollback' lộ hiểu sai: compensation là giao dịch nghiệp vụ mới, và có bước không hoàn tác được.
- "Choreography tốt hơn vì decoupled." → Decoupling không miễn phí — ai trả lời được saga đang kẹt ở đâu lúc 3h sáng?

### 9. Sai lầm phổ biến của ứng viên
- Không biết vấn đề tồn tại — thiết kế microservices với giả định ngầm có transaction xuyên service.
- Coi timeout là failure (như câu chuyện trên) — thiếu bước truy vấn trạng thái/đối soát.
- Quên isolation: không thiết kế trạng thái trung gian, khách thấy dữ liệu nửa vời, hoặc hai saga đan xen phá bất biến của nhau.
- Compensation không idempotent hoặc tự nó có thể fail — không có kế hoạch cho "compensation của compensation" (câu trả lời đúng: retry đến thành công + human escalation, vì thế compensation phải retriable).
- Không sắp thứ tự bước theo khả năng hoàn tác (pivot transaction).

### 10. Follow-up Questions
- Thiết kế saga cụ thể cho chuyển tiền liên ngân hàng — bước nào pivot? Đối soát thế nào?
- Hai saga đồng thời trên cùng đơn hàng (khách hủy đúng lúc hệ thống đang xử lý) — chống đan xen thế nào? (semantic lock, version, saga phải xử lý event hủy như một bước.)
- Temporal/Cadence giải quyết phần nào của bài toán? Durable execution là gì?
- Saga 10 bước có phải dấu hiệu xấu? (thường là ranh giới service sai — quá nhiều service cho một hành động nghiệp vụ.)
- So sánh saga với TCC (try-confirm-cancel) — khác gì, khi nào TCC hợp?

### 11. Liên hệ với Production
Mọi hệ thanh toán/đặt chỗ lớn (Grab, các ví điện tử, OTA) sống bằng saga + đối soát — bộ phận reconciliation là "compensation bằng người" cho những gì code không lường. Vấn đề nghiêm trọng khi: số saga kẹt tăng theo traffic (thiếu timeout/retry policy chuẩn), tiền/hàng lệch giữa các hệ cần đối soát tay hàng ngày, hoặc luồng nghiệp vụ mới không ai dám sửa vì sợ vỡ chuỗi event. Dấu hiệu cần hành động: dashboard không trả lời được "bao nhiêu saga đang dở dang, dở ở bước nào", và support ticket kiểu "bị trừ tiền nhưng không có đơn" xuất hiện — mỗi ticket như vậy là một saga hỏng đang được khách hàng debug hộ.

---

## Câu 6 — [Senior → Staff] Scalability & High Availability: từ 1 server tới hệ thống chịu lỗi đa tầng

### 1. Câu hỏi
"Hệ thống của anh cần scale từ 1.000 lên 1.000.000 user và đạt 99.99% availability. Trình bày cách tiếp cận có hệ thống: scale ở đâu, HA bằng gì, và những failure mode mới nào xuất hiện khi hệ thống lớn lên?"

### 2. Interviewer muốn kiểm tra điều gì?
- Phương pháp luận: tìm bottleneck bằng số liệu thay vì rải công nghệ theo trend.
- Hiểu HA là số học (compound availability) và kiến trúc (loại bỏ SPOF), không phải khẩu hiệu.
- Nhận thức các failure mode chỉ xuất hiện ở quy mô: cascading failure, retry storm, metastable state.

### 3. Câu trả lời ngắn gọn (30 giây)
"Scale có thứ tự: đo để tìm bottleneck thật → stateless hóa app layer để scale ngang sau LB → cache các tầng (CDN, app cache, DB cache) → tách đọc/ghi DB (replica) → async hóa những gì không cần đồng bộ (queue) → cuối cùng mới shard. HA là bài toán khác scale: loại bỏ SPOF từng tầng (LB đôi, multi-AZ, DB failover tự động), và quan trọng hơn — **giới hạn blast radius**: timeout + retry có ngân sách + circuit breaker + bulkhead để lỗi một phần không thành lỗi toàn phần. 99.99% = 52 phút downtime/năm — nghĩa là failover phải tự động (con người không kịp), deploy phải không downtime, và mọi dependency phải có kịch bản chết."

### 4. Câu trả lời Senior Level (3–5 phút)
**Scale — nguyên tắc trước công cụ:**
1. **Đo trước:** bottleneck ở đâu — CPU app? DB IOPS? Connection? Lock? Scale thứ không nghẽn là đốt tiền. USE/RED dashboard trước khi vẽ kiến trúc.
2. **Stateless app:** session ra Redis/token, file ra object storage → app layer thành cattle, autoscale tự do. Đây là bước rẻ nhất, giá trị nhất.
3. **Cache theo tầng:** CDN cho static + edge cache API đọc nhiều; cache app (câu Redis đã bàn); mục tiêu: chặn request trước khi nó chạm tầng đắt.
4. **DB:** replica cho đọc; connection pooling; index/query trước khi thêm máy (một query tồi ăn bằng 50 query tốt). Ghi quá tải → partition/shard (bước cuối, đắt nhất — chương Postgres).
5. **Async hóa:** những gì không cần trả lời ngay đẩy vào queue — san phẳng peak (buffer), cô lập tốc độ các thành phần. Peak 10x trung bình là bình thường; queue biến bài toán "chịu peak" thành "chịu trung bình".
6. **Định luật chi phối:** Amdahl (phần tuần tự giới hạn tổng speedup — DB ghi thường là phần tuần tự đó); Little's Law (concurrency = throughput × latency — dùng để tính pool size, số worker); tail latency (p99 của hệ gọi 10 service = gần như chắc chắn chạm p99 của một trong số đó — "tail at scale": hedged request, giảm fan-out đồng bộ).

**HA — số học và kiến trúc:**
- Chuỗi phụ thuộc nhân xác suất: 5 thành phần 99.9% nối tiếp = 99.5% (43 giờ/năm). Muốn 99.99% tổng thể: từng tầng redundant (song song thay nối tiếp) hoặc giảm số dependency bắt buộc (degrade được thì không tính vào chuỗi).
- Loại SPOF từng tầng: LB (đôi + health check), app (N+2 across AZ), DB (auto failover — nhưng nhớ: failover cũng có thời gian, và async replication có RPO), queue (cluster), **và cả những SPOF mềm**: DNS, secret manager, CI/CD (không deploy được bản fix cũng là downtime), một region cloud, một con người duy nhất hiểu hệ thống.
- **Failure mode của quy mô (phần phân biệt Staff):** cascading failure (một dependency chậm → thread/connection pool cạn → service khỏe cũng nghẽn — chống bằng timeout ngắn + bulkhead cô lập pool per-dependency); retry storm (lỗi thoáng qua × retry 3 lần × N tầng = tải nhân 3^N — chống bằng retry budget, chỉ retry ở một tầng, exponential backoff + jitter); thundering herd sau khi hồi phục (mọi client quay lại cùng lúc đánh gục lần hai — chống bằng jitter + slow start + load shedding); metastable failure (hệ kẹt ở trạng thái xấu tự duy trì ngay cả khi nguyên nhân gốc đã hết — thường phải chủ động shed load để thoát).
- **Graceful degradation:** xếp hạng tính năng theo mức sống còn — recommendation chết thì trả danh sách phổ biến; search chết thì trả cache cũ; nhưng checkout phải sống. Thiết kế "chết từng phần" là khác biệt giữa sự cố 5 phút và outage toàn trang.

### 5. Giải thích bản chất
Hai định luật nền: (1) **Scale = loại bỏ điểm tuần tự hóa** — mọi thứ buộc request đi qua một chỗ duy nhất (một lock, một DB ghi, một LB, một cache nóng) sớm muộn thành trần; lịch sử scale của một hệ thống là lịch sử tuần tự các điểm serialization bị phát hiện và tháo gỡ. (2) **HA = không có gì được phép là duy nhất, kể cả sự hiểu biết** — redundancy máy móc dễ, redundancy dữ liệu khó hơn (replication có lag, failover có RPO), redundancy tri thức khó nhất (runbook, on-call, không bus-factor-1). Và một nghịch lý đáng nói ở mức Principal: **thêm thành phần để tăng HA cũng thêm thứ để hỏng** — cluster failover tự động từng gây nhiều outage hơn nó ngăn được ở các hệ cấu hình kém (split-brain, failover flapping); độ tin cậy đến từ sự đơn giản được kiểm chứng, không từ chồng công nghệ.

### 6. Trade-off
- **Redundancy ↔ chi phí + phức tạp:** N+2 mọi tầng, multi-AZ, multi-region — tiền và độ phức tạp vận hành tăng theo; 99.9%→99.99% thường đắt gấp nhiều lần 99%→99.9%; và câu hỏi đúng là "business cần bao nhiêu số 9, cho luồng nào?" — không phải mọi endpoint cần như nhau.
- **Autoscale ↔ ổn định:** phản ứng nhanh với tải ↔ flapping, cold start, và autoscale không cứu được spike nhanh hơn thời gian khởi động pod (cần headroom + queue).
- **Timeout ngắn ↔ false positive:** cắt nhanh cứu pool ↔ giết request lẽ ra sẽ xong; timeout là phân phối, đặt theo p99 của dependency + budget tổng.
- **Load shedding ↔ trải nghiệm:** chủ động từ chối x% để 100-x% sống tốt — quyết định phải cài sẵn từ thời bình, giữa sự cố không ai dám bật.

### 7. Ví dụ Production
Outage đáng học nhất tôi từng mổ xẻ: một service phụ (gợi ý sản phẩm) chậm dần do GC — không chết, chỉ p99 từ 50ms lên 2s. Trang chủ gọi nó **đồng bộ không timeout riêng** → thread pool của trang chủ cạn dần → trang chủ nghẽn → user bấm refresh (retry của loài người) → tải tăng → mọi service sau trang chủ nghẽn theo → outage toàn trang 40 phút. Nguyên nhân gốc: một tính năng **không quan trọng** thiếu 3 dòng cấu hình (timeout 100ms, circuit breaker, fallback trả rỗng). Từ đó quy tắc bất di bất dịch của tôi: **mọi cuộc gọi xuyên service phải khai báo timeout, retry policy, và fallback — như một phần của code review checklist**; dependency không thiết yếu mặc định phải fail-open. Sự cố lớn hiếm khi do thành phần quan trọng chết — thường do thành phần không quan trọng được đối xử như thể nó không thể gây hại.

### 8. Những câu trả lời chưa đủ tốt
- "Thêm server và load balancer." → Scale được app stateless — còn DB? state? Và HA của chính LB?
- "Dùng Kubernetes để HA." → K8s restart pod chết, không cứu: DB failover, cascading failure, retry storm, region chết. HA là thuộc tính kiến trúc, không phải tính năng của orchestrator.

### 9. Sai lầm phổ biến của ứng viên
- Scale trước khi đo — rải cache/shard/microservices lên hệ chưa biết nghẽn đâu.
- Quên rằng availability nhân theo chuỗi dependency — thêm dependency đồng bộ là giảm availability tổng.
- Retry không backoff không jitter không budget — tự chế máy khuếch đại sự cố.
- Không phân biệt HA (chịu lỗi thành phần) và DR (chịu thảm họa vùng) — hai bài, hai giá.
- Thiết kế cho steady state, quên transient: deploy, failover, cold cache, reconnect — đa số outage xảy ra lúc **chuyển trạng thái**.

### 10. Follow-up Questions
- Tính pool size cho service 2.000 RPS, downstream p99 50ms? (Little's Law — và tại sao pool theo p99 chứ không mean.)
- Circuit breaker: ngưỡng mở, half-open thế nào, khác gì retry? Đặt ở client hay mesh?
- Load shedding ưu tiên theo gì — user tier, endpoint, độ tươi cache? Ai quyết định bảng ưu tiên?
- Graceful shutdown đúng cách khi autoscale thu pod: drain thế nào với request đang bay, connection dài, job dở?
- 99.99% cam kết SLA nhưng dependency bên thứ ba chỉ SLA 99.9% — các lựa chọn? (degrade được? cache được? dual vendor? đàm phán lại SLA?)

### 11. Liên hệ với Production
Google SRE book chuẩn hóa cả lĩnh vực này (error budget: 99.99% nghĩa là được phép hỏng 52 phút/năm — dùng ngân sách đó cho velocity thay vì sợ hãi); AWS well-architected framework là checklist thực dụng. Vấn đề nghiêm trọng khi: hệ thống lớn nhanh hơn sự trưởng thành vận hành (traffic ×10 nhưng chưa có on-call, runbook, load test), hoặc availability cam kết trong hợp đồng vượt khả năng kiến trúc thật. Dấu hiệu cần hành động: sự cố lặp cùng dạng (thiếu postmortem culture), MTTR không giảm theo thời gian, p99 tăng dần theo traffic (điểm serialization đang bão hòa), và không ai trả lời được "nếu Redis/AZ/region X chết thì chuyện gì xảy ra" — chưa có câu trả lời nghĩa là câu trả lời là outage.

---

## Câu 7 — [Staff → Principal] Multi-region, Disaster Recovery và Observability: vận hành hệ thống toàn cầu

### 1. Câu hỏi
"Thiết kế hệ thống phục vụ user toàn cầu: multi-region thế nào, DR với RPO/RTO ra sao, và observability cần gì để vận hành nổi? CAP theorem áp vào đây thế nào?"

### 2. Interviewer muốn kiểm tra điều gì?
- Mức Staff/Principal: quyết định có giá hàng triệu đô và ràng buộc vật lý (tốc độ ánh sáng) — không thể học vẹt.
- Hiểu CAP/PACELC như công cụ phân loại quyết định thực tế, không phải trivia.
- Observability như năng lực vận hành: ba trụ, SLO, và văn hóa postmortem.

### 3. Câu trả lời ngắn gọn (30 giây)
"Ba mức theo nhu cầu thật: (1) **một region + DR region** — backup liên tục + replica warm ở region khác, RPO phút, RTO giờ, đủ cho đa số; (2) **active-passive** — region phụ nhận traffic đọc hoặc standby nóng, failover có kịch bản; (3) **active-active** — user ghi ở region gần nhất, đối mặt trực diện bài toán ghi đồng thời hai nơi: hoặc trả latency xuyên đại dương cho consensus đồng bộ (Spanner-style), hoặc chấp nhận conflict và giải bằng CRDT/last-write-wins/phân vùng dữ liệu theo user-home-region — cách cuối thực dụng nhất. RPO/RTO là quyết định **kinh doanh** trả bằng tiền: RPO=0 nghĩa là sync replication xuyên region trên mọi ghi. Observability: metrics (SLO + alert trên symptom), tracing phân tán (bắt buộc với microservices), structured logs có correlation — và DR chỉ tồn tại nếu được **diễn tập định kỳ**: backup chưa từng restore là backup chưa tồn tại."

### 4. Câu trả lời Senior Level (3–5 phút)
**CAP/PACELC làm khung:** partition xảy ra (không tránh được giữa region) → chọn C (từ chối ghi khi không liên lạc được đa số — hệ tiền bạc) hay A (nhận ghi cả hai phía, hòa giải sau — giỏ hàng, social). PACELC bổ sung phần quan trọng hơn cho thiết kế thường nhật: **ngay cả khi không có partition**, vẫn phải chọn Latency vs Consistency — ghi sync xuyên region = +100–300ms mọi giao dịch (vật lý: Singapore–Virginia ~220ms RTT, không công nghệ nào mua được tốc độ ánh sáng).

**Chiến lược thực dụng theo dữ liệu (điểm Staff — không chọn một mô hình cho cả hệ):**
- Phân loại: dữ liệu user-scoped (profile, đơn hàng) → **home region per user** (ghi luôn về region nhà — không bao giờ conflict, đọc local cache ở region khác); dữ liệu toàn cục ít ghi (catalog, config) → ghi một nơi, replicate đọc mọi nơi; dữ liệu đếm/hợp nhất được (like, view) → CRDT/gộp async; dữ liệu tiền bạc toàn cục → chấp nhận latency consensus hoặc thiết kế để không cần (sổ cái per-region, đối soát).
- Failover region: DNS/anycast routing, nhưng cạm bẫy thật nằm ở **dữ liệu**: async replication có lag → failover là mất RPO giây-phút dữ liệu + nguy cơ split-brain nếu region cũ còn nhận ghi (cần fencing ở tầng routing). Failback còn khó hơn failover — dữ liệu ghi ở region phụ trong sự cố phải hòa về.

**DR:** RPO (mất tối đa bao nhiêu dữ liệu) và RTO (đứng dậy trong bao lâu) do business ký, kỹ thuật báo giá: RPO giờ = backup định kỳ (rẻ); RPO phút = replication liên tục; RPO ~0 = sync xuyên region (đắt + chậm mọi ghi). Kim tự tháp thường thấy: DB chính RPO 5 phút, object storage versioned, config/infra as code (region dựng lại được từ code). **Diễn tập là phần lớn giá trị:** game day mỗi quý, restore backup thật vào môi trường thật, đo RTO thật — con số trên giấy luôn lạc quan 5–10 lần. Và đừng quên DR cho: secret, DNS, CI/CD, monitoring (sự cố mà mù thì không sửa được), và **con người** (runbook, phân quyền khi nửa đêm).

**Observability — ba trụ có chủ đích:**
- **Metrics:** RED per service (rate, errors, duration percentiles) + USE per resource; alert trên **symptom user-facing** (SLO burn rate) không phải cause (CPU cao mà user không đau thì để ban ngày xem) — giảm alert fatigue, thứ giết on-call.
- **Tracing:** với microservices là bắt buộc — không trace, một request lỗi xuyên 8 service là 8 giờ điều tra; trace ID phát ở edge, propagate mọi hop (HTTP header/Kafka header), sample thông minh (giữ 100% lỗi + chậm, 1% bình thường).
- **Logs:** structured (JSON), có trace/correlation ID để nhảy metrics→trace→log trong một cú click. 
- **SLO + error budget:** đặt mục tiêu theo user journey (checkout success rate 99.95% trong 30 ngày), budget còn → ship nhanh; cháy budget → freeze feature, trả nợ ổn định. Đây là hợp đồng hòa bình giữa product và reliability.

### 5. Giải thích bản chất
Tầng sâu nhất của multi-region là **vật lý và toán**: thông tin không đi nhanh hơn ánh sáng → hai nơi cách 15.000km không thể cùng biết một sự thật trong <100ms → mọi "nhất quán toàn cầu tức thời" là ảo giác được mua bằng latency (chờ đủ round-trip) hoặc bằng nói dối có kiểm soát (eventual + hòa giải). Consensus (Paxos/Raft) không xóa giới hạn này — nó chỉ đảm bảo *một* thứ tự được thống nhất, với giá ≥1 RTT tới đa số. Vì thế nghệ thuật thiết kế toàn cầu = **giảm phạm vi của những gì cần thống nhất toàn cầu**: dữ liệu của user Việt Nam không cần consensus với Virginia — trừ khi bạn thiết kế để nó cần. Observability có bản chất tương tự: hệ phân tán không quan sát trực tiếp được — nó chỉ kể chuyện qua telemetry; đầu tư observability là đầu tư vào **khả năng đặt câu hỏi chưa nghĩ ra trước** (khác monitoring: trả lời câu hỏi định sẵn). Và ở mức Principal, tất cả quy về một câu: bạn không thể vận hành thứ bạn không nhìn thấy, và không thể tin thứ bạn chưa từng diễn tập hỏng.

### 6. Trade-off
- **Active-active ↔ độ phức tạp dữ liệu:** latency đọc/ghi local tuyệt vời ↔ conflict resolution là bài toán vĩnh viễn; đa số công ty nên dừng ở active-passive + home-region.
- **RPO/RTO thấp ↔ tiền:** mỗi bậc giảm RPO/RTO tăng chi phí hạ tầng và độ phức tạp bậc thang; region standby full-size đắt gấp đôi mọi thứ để dùng vài giờ mỗi năm (pilot light/warm standby là điểm giữa).
- **Cardinality của metrics ↔ chi phí:** label per-user/per-endpoint nổ chi phí lưu trữ; sampling trace ↔ mất dấu request hiếm; log mọi thứ ↔ hóa đơn log vượt hóa đơn compute (chuyện thật, rất thường gặp).
- **Alert nhạy ↔ alert fatigue:** trượt về phía nào cũng chết — nhạy quá thì on-call điếc dần và bỏ qua alert thật; trễ quá thì user phát hiện trước monitoring — SLO burn rate đa cửa sổ là lời giải kỹ thuật, văn hóa dọn alert định kỳ là lời giải con người.

### 7. Ví dụ Production
Bài kiểm tra DR đắt giá nhất tôi chứng kiến: công ty tự tin có backup đầy đủ + region dự phòng, giấy tờ RTO 4 giờ. Game day đầu tiên (diễn tập thật, region chính bị "tắt"): restore DB 2TB mất 9 giờ (băng thông restore chưa từng đo); app dựng lên được nhưng secret manager chỉ tồn tại ở region chết; DNS TTL 24h nghĩa là một nửa traffic vẫn đổ về region chết cả ngày; và runbook viết 2 năm trước chỉ dẫn tới dashboard đã bị xóa. RTO thật: ~2 ngày, tức 12 lần con số cam kết. Không có sự cố nào xảy ra — nhưng game day đó thay đổi cả lộ trình hạ tầng năm sau: restore drill hàng quý tự động, secret/DNS/monitoring nhân bản trước cả DB, runbook được test như code. Bài học Principal: **DR không phải kiến trúc, DR là bài tập thể dục — bỏ tập là mất**.

### 8. Những câu trả lời chưa đủ tốt
- "Dùng multi-region active-active cho HA tối đa." → Cho toàn bộ dữ liệệu? Conflict giải thế nào? Chi phí? Câu này không phải câu trả lời, nó là đầu bài của mười câu hỏi khó.
- "CAP: chọn 2 trong 3." → Cách hiểu sai kinh điển — P không phải lựa chọn (partition là thực tế), và trade-off C/L tồn tại cả khi mạng khỏe (PACELC). Nói "chọn 2 trong 3" là red flag ở mức Staff.

### 9. Sai lầm phổ biến của ứng viên
- Thiết kế multi-region cho compute mà quên dữ liệu — app chạy hai region nhưng cùng gọi về một DB (latency + SPOF nguyên vẹn).
- RPO/RTO tự nghĩ trong đầu kỹ sư thay vì lấy từ business — dẫn tới hoặc thừa (đốt tiền) hoặc thiếu (vỡ cam kết).
- Backup có, restore chưa bao giờ thử; DR plan có, phụ thuộc (DNS, secret, monitoring, con người) chưa kiểm kê.
- Observability = cài đủ Prometheus/Grafana/Jaeger — công cụ có, câu hỏi không: không SLO, alert theo cause, trace không propagate qua queue.
- Nói về consistency toàn cầu như thể một cấu hình bật lên được — không nhận thức chi phí vật lý.

### 10. Follow-up Questions
- Spanner đạt external consistency bằng gì? (TrueTime — GPS + đồng hồ nguyên tử, chờ hết uncertainty window; cái giá và bài học cho người không phải Google?)
- Thiết kế home-region cho user hay di chuyển (du lịch, chuyển quốc gia) — migrate dữ liệu nhà thế nào?
- SLO burn rate alert đa cửa sổ (5m+1h, 6h+3d) hoạt động thế nào, tại sao hơn threshold tĩnh?
- Chaos engineering: bắt đầu từ đâu một cách an toàn? Tiêu chí một experiment tốt?
- Nếu ngân sách chỉ đủ một trong hai: nâng observability hay dựng region DR — anh chọn gì cho công ty X? (câu Principal thuần — chấm khung phân tích: tần suất × thiệt hại của từng loại rủi ro với business cụ thể.)

### 11. Liên hệ với Production
Netflix (chaos monkey, region evacuation drill định kỳ — họ **thường xuyên** sơ tán region thật), Google (SLO/error budget là chuẩn ngành từ SRE book), Cloudflare/AWS các outage postmortem công khai đều quy về: thứ chưa diễn tập thì không hoạt động lúc cần. Vấn đề nghiêm trọng khi: công ty ký SLA/compliance đòi DR mà hạ tầng chỉ có backup; sự cố region cloud thật xảy ra (mỗi năm vài lần, không phải giả thuyết); hoặc MTTR bị chi phối bởi "tìm xem chuyện gì đang xảy ra" thay vì "sửa" — dấu hiệu observability nợ nần. Dấu hiệu cần hành động: chưa từng restore drill trong 12 tháng, alert channel bị mute, không có postmortem blameless sau sự cố lớn, và câu "region chết thì sao" chỉ có câu trả lời bằng miệng chưa có bằng chứng bằng diễn tập.
