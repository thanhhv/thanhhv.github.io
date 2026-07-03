+++
title = "Bài 7 — Security & Authentication"
date = "2026-07-03T14:00:00+07:00"
draft = false
tags = ["backend", "interview"]
series = ["Backend Interview"]
+++

# Security & Authentication

---

## Câu 1 — [Fundamental → Senior] JWT vs Session: khác nhau bản chất ở đâu, và tại sao thu hồi JWT lại khó?

### 1. Câu hỏi
"So sánh session-based auth và JWT. JWT có ưu điểm stateless — nhưng làm sao anh thu hồi (revoke) một JWT trước khi nó hết hạn?"

### 2. Interviewer muốn kiểm tra điều gì?
- Hiểu bản chất stateful vs stateless của hai mô hình, không chỉ "JWT hiện đại hơn".
- Nhận ra ưu điểm lớn nhất của JWT (stateless) chính là nhược điểm lớn nhất (không revoke được) — câu bẫy kinh điển.
- Kinh nghiệm thật với refresh token, token rotation, và các đánh đổi.

### 3. Câu trả lời ngắn gọn (30 giây)
"Session: server lưu trạng thái đăng nhập (session store — Redis/DB), client chỉ giữ session ID; muốn logout/revoke thì xóa ở server — tức thời, nhưng mỗi request phải tra store (stateful, cần chia sẻ store khi scale). JWT: token tự chứa thông tin + chữ ký, server chỉ verify chữ ký không cần tra store — stateless, scale ngang dễ, nhưng **không revoke được** vì server không giữ trạng thái gì để xóa. Token đã phát là hợp lệ tới khi hết hạn. Giải pháp thực dụng: access token sống ngắn (5–15 phút) + refresh token sống dài lưu ở server (revoke được); muốn revoke ngay thì thêm blocklist — nhưng blocklist khiến JWT không còn stateless nữa, đó chính là nghịch lý."

### 4. Câu trả lời Senior Level (3–5 phút)
**Problem:** HTTP stateless, nhưng "đã đăng nhập" là trạng thái. Câu hỏi: giữ trạng thái đó ở đâu — server hay client?

**Session (trạng thái ở server):** client giữ opaque ID trong cookie; server map ID → dữ liệu session trong store. Ưu: revoke tức thì (xóa key), đổi quyền có hiệu lực ngay, ID không lộ thông tin. Nhược: mỗi request tra store (thêm latency + store là dependency phải HA); scale ngang cần shared store hoặc sticky session; cookie kèm CSRF concern.

**JWT (trạng thái ở client):** token gồm header.payload.signature; server verify chữ ký (HMAC với secret, hoặc RSA/ECDSA với public key) → tin payload mà không tra DB. Ưu: stateless thật (bất kỳ node nào verify được — hợp microservices, public key phân phát cho nhiều service), giảm tải store. Nhược cốt lõi: **không có nút tắt**. Sa thải nhân viên, đổi mật khẩu, phát hiện token lộ — token cũ vẫn chạy tới khi exp.

**Cách hành nghề đúng:**
- Access token ngắn hạn (5–15 phút) + refresh token dài hạn. Access hết hạn → dùng refresh xin cái mới. Cửa sổ rủi ro khi revoke = tuổi thọ access token.
- Refresh token **lưu server** (DB/Redis) → revoke được thật; logout = xóa refresh + chờ access hết hạn.
- **Refresh token rotation:** mỗi lần refresh phát cặp mới, vô hiệu cái cũ; nếu một refresh token cũ được dùng lại → dấu hiệu bị đánh cắp → thu hồi cả họ token (reuse detection).
- Muốn revoke access token tức thì: blocklist theo `jti` trong Redis với TTL = thời gian còn lại của token. Chấp nhận đánh đổi: giờ mỗi request tra Redis — về lại mô hình gần-stateful. Trung thực mà nói: nếu bạn cần revoke tức thì cho mọi request, session có khi là lựa chọn đơn giản hơn.

**Production:** lưu token ở đâu phía client cũng là câu hỏi bảo mật: localStorage dễ bị XSS đọc; httpOnly cookie chống XSS đọc token nhưng mở CSRF (cần SameSite + CSRF token). Không có chỗ lưu hoàn hảo — chọn theo mô hình đe dọa.

### 5. Giải thích bản chất
Đây là một thể hiện của trade-off nền tảng đã gặp khắp bộ tài liệu: **trạng thái tập trung (dễ kiểm soát, khó scale) vs trạng thái phân tán (dễ scale, khó kiểm soát)**. Session đặt sự thật ở một nơi → thay đổi có hiệu lực tức thì nhưng nơi đó phải luôn sống và luôn được hỏi. JWT phát tán sự thật vào tay client → không cần hỏi ai nhưng cũng không thu hồi được điều đã trao. Chữ ký số giải quyết được "làm sao tin client" (không giả mạo được), nhưng **không** giải quyết được "làm sao rút lại điều đã tin" — đó là giới hạn bản chất của mọi credential mang-theo-được (bearer token), giống hệt việc phát hành hộ chiếu giấy: cấp thì dễ, thu hồi tờ đã in thì phải có danh sách đen ở cửa khẩu. Mọi kiến trúc token đều dao động quanh nghịch lý này.

### 6. Trade-off
- **Session:** revoke tức thì, đổi quyền ngay, token nhỏ ↔ tra store mỗi request, store là SPOF phải HA, scale cần shared state.
- **JWT thuần:** stateless, scale đẹp, hợp đa service ↔ không revoke, payload lộ (chỉ ký chứ không mã hóa — đừng để dữ liệu nhạy cảm trong payload), token lớn hơn đi kèm mọi request.
- **JWT + refresh + rotation:** cân bằng thực dụng ↔ độ phức tạp cao hơn hẳn (rotation, reuse detection, refresh store).
- **JWT + blocklist:** revoke được ↔ mất tính stateless — nửa nạc nửa mỡ, cân nhắc quay về session.

### 7. Ví dụ Production
Sự cố kinh điển: công ty dùng JWT access token TTL 24 giờ cho tiện (đỡ phải refresh thường xuyên). Một nhân viên bị sa thải lúc 9h sáng, admin "khóa tài khoản" trong hệ thống — nhưng JWT của người đó vẫn hợp lệ tới 9h sáng hôm sau, và anh ta vẫn tải được dữ liệu khách hàng suốt buổi chiều. Root cause: TTL dài + không có blocklist = "khóa tài khoản" chỉ là ảo giác UI. Fix: access token về 15 phút, thêm blocklist theo jti cho hành động khóa khẩn cấp. Bài học tôi luôn nhấn: **TTL của access token chính là cửa sổ mà "revoke" của bạn vô nghĩa** — chọn nó với ý thức đó, không phải chọn theo sự tiện lợi.

### 8. Những câu trả lời chưa đủ tốt
- "JWT tốt hơn session vì stateless." → Stateless là đánh đổi, không phải ưu điểm thuần. Hỏi ngược: revoke thế nào? Câu này rớt ngay.
- "JWT mã hóa thông tin nên an toàn." → JWT mặc định chỉ **ký** (signed), không **mã hóa** — ai cũng đọc được payload bằng base64 decode. Nhầm sign với encrypt là lỗi hiểu bản chất.

### 9. Sai lầm phổ biến của ứng viên
- Nghĩ JWT được mã hóa, để thông tin nhạy cảm (password hash, PII) trong payload.
- Không biết `alg: none` attack và JWT library confusion (RS256 bị ép thành HS256 dùng public key làm HMAC secret) — lỗ hổng JWT nổi tiếng.
- TTL access token quá dài → revoke vô nghĩa.
- Lưu refresh token ở client mà không rotation → lộ một lần là dùng mãi.
- Không phân biệt authentication (bạn là ai) và authorization (bạn được làm gì) — JWT giải cái đầu, không tự động giải cái sau.

### 10. Follow-up Questions
- Refresh token rotation với reuse detection hoạt động chính xác thế nào? Xử lý race khi client gọi refresh hai lần đồng thời (mạng chập)?
- Access token nên chứa gì, không nên chứa gì? Role/permission để trong token hay tra mỗi request? (đánh đổi: token cồng kềnh + quyền cũ vs tra DB mất tính stateless.)
- `alg: none` và key confusion attack phòng thế nào? (allowlist thuật toán ở phía verify.)
- Single logout xuyên nhiều service/thiết bị làm sao với JWT?
- Nếu thiết kế lại cho hệ 50 microservice — mỗi service tự verify JWT hay có auth gateway tập trung? Trade-off?

### 11. Liên hệ với Production
Đa số hệ lớn dùng mô hình lai: JWT access ngắn hạn cho tốc độ + refresh token có trạng thái cho khả năng thu hồi; auth thường tập trung ở API gateway (verify một chỗ) rồi truyền identity xuống dưới. Vấn đề nghiêm trọng khi: cần tuân thủ (nhân viên nghỉ phải mất quyền ngay), hoặc phát hiện token lộ hàng loạt. Dấu hiệu cần rà soát: TTL access token tính bằng giờ/ngày, không có cơ chế revoke khẩn cấp, refresh token không rotation, và câu hỏi kiểm toán đơn giản: "nếu một token bị lộ ngay bây giờ, mất bao lâu để nó vô hại?" — nếu câu trả lời là "tới khi hết hạn", bạn có vấn đề.

---

## Câu 2 — [Senior] OAuth2 và OpenID Connect: giải quyết bài toán gì, và các flow khác nhau thế nào?

### 1. Câu hỏi
"Giải thích OAuth2. Authorization Code flow hoạt động ra sao và tại sao cần PKCE? OAuth2 khác OpenID Connect thế nào?"

### 2. Interviewer muốn kiểm tra điều gì?
- Phân biệt authorization delegation (OAuth2) và authentication (OIDC) — nhầm lẫn cực phổ biến.
- Hiểu tại sao có nhiều flow và mỗi flow cho loại client nào.
- Nhận thức bảo mật: tại sao implicit flow bị khai tử, PKCE giải quyết gì.

### 3. Câu trả lời ngắn gọn (30 giây)
"OAuth2 là giao thức **ủy quyền** (delegation): cho phép app A truy cập tài nguyên của user ở dịch vụ B mà không cần user đưa mật khẩu B cho A — thay vào đó user cấp một access token có phạm vi (scope) giới hạn. Nó KHÔNG phải giao thức đăng nhập. OpenID Connect là lớp mỏng **xây trên** OAuth2 để làm authentication: thêm ID token (JWT chứa danh tính user) và endpoint chuẩn — 'Login with Google' thực chất là OIDC. Authorization Code flow: user login ở provider → provider trả code ngắn hạn về app → app đổi code lấy token qua kênh server-to-server (bí mật không lộ ra browser). PKCE thêm một secret động mỗi lần để chống việc code bị chặn giữa đường — bắt buộc cho mobile/SPA vì chúng không giữ được client secret."

### 4. Câu trả lời Senior Level (3–5 phút)
**Problem gốc:** thời trước OAuth, muốn app đọc danh bạ Gmail của bạn, bạn phải đưa nó username/password Gmail — thảm họa: app giữ mật khẩu, truy cập mọi thứ, không thu hồi được. OAuth2 giải: user xác thực trực tiếp với provider, app chỉ nhận token phạm vi hẹp (chỉ đọc danh bạ), thu hồi được, không bao giờ thấy mật khẩu.

**Các vai:** Resource Owner (user), Client (app), Authorization Server (cấp token), Resource Server (giữ dữ liệu, chấp nhận token).

**Authorization Code flow (chuẩn cho web app có backend):**
1. App chuyển user tới Authorization Server kèm client_id, scope, redirect_uri.
2. User đăng nhập + đồng ý → server redirect về app kèm **authorization code** (ngắn hạn, dùng một lần).
3. App (backend) đổi code + client_secret lấy access token qua kênh sau lưng (back channel) — token không bao giờ đi qua browser/URL.
4. App dùng access token gọi Resource Server.

**Tại sao PKCE (Proof Key for Code Exchange):** mobile app và SPA không giữ được client_secret an toàn (decompile/xem source là ra). Kẻ tấn công chặn được authorization code (qua redirect độc, app giả đăng ký cùng URL scheme) có thể đổi lấy token. PKCE: client tạo `code_verifier` ngẫu nhiên mỗi phiên, gửi `code_challenge = hash(verifier)` lúc xin code; lúc đổi code phải kèm `verifier` gốc → server hash lại và so. Code bị chặn cũng vô dụng vì thiếu verifier. Nay PKCE được khuyến nghị cho **mọi** client, kể cả web có backend.

**Các flow khác:**
- **Implicit flow (đã khai tử):** trả token thẳng trong URL fragment — token lộ trong history/log/referer; thay hoàn toàn bằng Authorization Code + PKCE.
- **Client Credentials:** không có user — service gọi service (machine-to-machine).
- **Resource Owner Password (chống chỉ định):** app nhận trực tiếp username/password — phá vỡ toàn bộ mục đích OAuth, chỉ dùng cho legacy first-party.

**OAuth2 vs OIDC:** OAuth2 access token dành cho **Resource Server** (đừng dùng nó để xác định danh tính — nó chỉ nói "được phép làm X", không nói "đây là ai"). OIDC thêm **ID token** — JWT có `sub`, `email`, `iss`, `aud` — dành cho **Client** biết user là ai. Nhầm lẫn kinh điển: dùng OAuth access token làm bằng chứng đăng nhập → lỗ hổng "confused deputy" (token cấp cho app khác bị dùng lại).

### 5. Giải thích bản chất
OAuth2 về bản chất là bài toán **ủy quyền có kiểm soát giữa ba bên không hoàn toàn tin nhau**: làm sao để bên A hành động thay user tại bên B, mà B chắc chắn user đồng ý, A không lạm quyền, và user rút lại được — tất cả không để lộ credential gốc. Lời giải là **thay thế credential vạn năng (mật khẩu) bằng token hạn chế (scope + expiry + revocable)** và **tách kênh xác thực (user↔provider) khỏi kênh sử dụng (app↔resource)**. PKCE là một lớp phòng thủ thêm dựa trên nguyên lý mật mã cơ bản: chứng minh "tôi là bên khởi tạo request" bằng cách giữ bí mật một tiền ảnh (preimage) mà chỉ tiết lộ ở bước cuối — kẻ đứng giữa thấy hash nhưng không đảo ngược được. Toàn bộ độ phức tạp của OAuth là cái giá của việc **không tin bất kỳ ai hoàn toàn** trong một hệ ba bên — bớt đi bên nào tin tưởng thì flow đơn giản hơn (client credentials không có user nên gọn hơn hẳn).

### 6. Trade-off
- **Authorization Code + PKCE:** an toàn nhất, token không lộ browser ↔ nhiều bước, cần back channel/redirect handling.
- **Delegation qua provider (Login with Google):** user không tạo mật khẩu mới, bạn không giữ mật khẩu ↔ phụ thuộc provider (họ down thì user không login được), và lock-in danh tính.
- **Access token ngắn + scope hẹp:** rủi ro lộ thấp ↔ refresh thường xuyên hơn, quản lý scope phức tạp.
- **Tự dựng OAuth server vs mua (Auth0/Keycloak/Cognito):** kiểm soát toàn bộ ↔ auth là mảng cực dễ sai và hậu quả nặng — thường nên mua/dùng thư viện đã kiểm chứng thay vì tự viết.

### 7. Ví dụ Production
Lỗ hổng thật rất phổ biến: một startup làm "Login with Google" nhưng dùng **access token** của Google làm bằng chứng đăng nhập, verify bằng cách gọi Google API xem token có hợp lệ không. Vấn đề: access token đó có thể được cấp cho một app **khác** (cùng gọi Google), kẻ tấn công lấy token từ app mình rồi gửi sang startup → được đăng nhập dưới danh nghĩa nạn nhân. Đây là confused deputy. Fix đúng: dùng **ID token** của OIDC và verify `aud` (audience) khớp client_id của mình — access token không bao giờ là bằng chứng danh tính. Bài học: 90% lỗ hổng OAuth đến từ hiểu sai "token này dành cho ai và chứng minh điều gì".

### 8. Những câu trả lời chưa đủ tốt
- "OAuth2 để đăng nhập bằng Google/Facebook." → Đó là OIDC (xây trên OAuth2). OAuth2 thuần là ủy quyền, không phải đăng nhập. Nhầm này rất phổ biến và bị trừ điểm.
- "Có nhiều flow, dùng cái nào cũng được." → Mỗi flow cho loại client cụ thể; chọn sai (implicit cho SPA, password grant cho third-party) là lỗ hổng.

### 9. Sai lầm phổ biến của ứng viên
- Nhầm OAuth2 (authz) với OIDC (authn); dùng access token làm bằng chứng danh tính.
- Không biết implicit flow đã bị khai tử và tại sao (token lộ trong URL).
- Không hiểu PKCE giải quyết gì, nghĩ nó chỉ là "bảo mật thêm".
- Không verify `aud`, `iss`, `exp` của ID token → chấp nhận token của app khác/provider giả.
- Nghĩ tự viết OAuth server là chuyện đơn giản.

### 10. Follow-up Questions
- Token bị lộ trong `redirect_uri` mở (open redirect) — tấn công thế nào, chống ra sao? (allowlist redirect_uri chính xác, không wildcard.)
- Scope design: chia scope mịn tới đâu? Chuyện gì khi user cấp scope rồi bạn cần thêm scope mới?
- Machine-to-machine giữa 50 service dùng client credentials — quản lý client_secret thế nào ở quy mô đó? (mTLS thay secret? workload identity?)
- ID token vs access token: cái nào gửi tới Resource Server, cái nào app tự đọc? Tại sao không lẫn lộn?
- Nếu Authorization Server (provider) bị down, hệ thống của anh degrade thế nào?

### 11. Liên hệ với Production
Mọi hệ có "đăng nhập bằng mạng xã hội" hoặc tích hợp bên thứ ba đều chạy OAuth2/OIDC; các công ty lớn thường có Identity Provider tập trung (Okta/Keycloak/Auth0 hoặc tự dựng) làm nguồn danh tính duy nhất. Vấn đề nghiêm trọng khi: tích hợp nhiều provider, cần SSO xuyên nhiều sản phẩm, hoặc compliance đòi audit truy cập chi tiết. Dấu hiệu cần rà soát: dùng access token làm danh tính, không verify audience, redirect_uri dùng wildcard, hoặc tự viết logic OAuth thay vì dùng thư viện chuẩn — auth là nơi "tự làm cho hiểu" biến thành lỗ hổng production.

---

## Câu 3 — [Senior → Staff] OWASP & các lỗ hổng web: SQL Injection, XSS, CSRF — cơ chế và phòng thủ tầng nào?

### 1. Câu hỏi
"Giải thích SQL Injection, XSS, CSRF: cơ chế tấn công và phòng thủ. Tại sao 'validate input' không phải là câu trả lời đầy đủ cho cả ba?"

### 2. Interviewer muốn kiểm tra điều gì?
- Hiểu cơ chế từng lỗ hổng, không đọc thuộc tên.
- Biết phòng thủ đúng tầng (parameterized query, output encoding, token/SameSite) — mỗi lỗ hổng một lớp khác nhau.
- Tư duy defense in depth thay vì một liều thuốc.

### 3. Câu trả lời ngắn gọn (30 giây)
"Cả ba là lỗi **trộn lẫn dữ liệu với lệnh/ngữ cảnh tin cậy**, nhưng ở ba nơi khác nhau nên phòng thủ khác nhau. SQL Injection: dữ liệu user bị diễn giải thành SQL → chống bằng **parameterized query/prepared statement** (tách lệnh khỏi dữ liệu ở tầng driver), không phải escape thủ công. XSS: dữ liệu user bị diễn giải thành HTML/JS trong browser nạn nhân → chống bằng **output encoding theo ngữ cảnh** + Content Security Policy, không phải chỉ lọc input. CSRF: browser tự động gửi cookie khiến request giả mạo được thực thi với quyền nạn nhân → chống bằng **CSRF token + SameSite cookie**, đây là lỗi về xác thực nguồn gốc request chứ không phải về nội dung. 'Validate input' giúp giảm bề mặt nhưng không giải quyết gốc — gốc nằm ở tách biệt dữ liệu/lệnh và xác thực ngữ cảnh."

### 4. Câu trả lời Senior Level (3–5 phút)
**SQL Injection:** `"SELECT * FROM users WHERE name = '" + input + "'"` với input `' OR '1'='1` → truy vấn đổi nghĩa. Gốc rễ: chuỗi lệnh và dữ liệu bị nối thành một. Phòng thủ đúng: **prepared statement** — driver gửi câu lệnh có placeholder (`WHERE name = $1`) và dữ liệu **riêng biệt** tới DB; DB không bao giờ parse dữ liệu thành SQL. Escape thủ công là chống chỉ định (sót ca, sai charset). Bổ sung: least privilege (app user không có DROP), ORM đúng cách (nhưng cẩn thận raw query trong ORM), và lưu ý injection không chỉ ở SQL — có NoSQL injection (Mongo `$where`, operator injection), command injection, LDAP injection — cùng một gốc.

**XSS (Cross-Site Scripting):** dữ liệu user (comment, tên) chứa `<script>` được render vào trang mà không encode → chạy trong browser nạn nhân, đánh cắp cookie/token, thao tác thay user. Ba loại: stored (lưu DB, phát tán), reflected (phản chiếu từ URL), DOM-based (JS client xử lý sai). Phòng thủ đúng: **output encoding theo ngữ cảnh** — encode khác nhau cho HTML body, HTML attribute, JS, URL, CSS (framework hiện đại như React tự escape phần lớn, nhưng `dangerouslySetInnerHTML`/`v-html` mở lại cửa). Lớp hai: **CSP** giới hạn script nào được chạy (chặn inline script, chỉ cho phép nguồn tin cậy) — phòng tuyến cuối khi encoding sót. Đây là lý do "lọc input" không đủ: cùng một chuỗi an toàn trong ngữ cảnh này, nguy hiểm trong ngữ cảnh khác — phải xử lý ở **output**, đúng ngữ cảnh render.

**CSRF (Cross-Site Request Forgery):** khác hẳn hai cái trên — không inject gì cả. Kẻ tấn công lừa browser nạn nhân (đang đăng nhập ngân hàng) gửi request `POST /transfer` từ một trang độc; browser **tự động đính kèm cookie** → server tưởng là request hợp lệ. Gốc rễ: cookie gửi tự động theo domain bất kể ai khởi tạo request. Phòng thủ: **CSRF token** (giá trị bí mật per-session/per-request mà trang độc không đọc được vì same-origin policy) + **SameSite cookie** (`Lax`/`Strict` — browser không gửi cookie cho request cross-site). API dùng token trong header (không phải cookie) miễn nhiễm CSRF tự nhiên — vì trang độc không set được header tùy ý cross-origin.

**Tư duy tổng:** defense in depth — không lỗ hổng nào chỉ có một lớp phòng thủ. Và WAF là lớp bọc ngoài, không thay được sửa gốc.

### 5. Giải thích bản chất
Ba lỗ hổng khác bề mặt nhưng cùng một **định lý an ninh nền tảng: mọi lỗ hổng injection là hệ quả của việc một trình thông dịch (interpreter) nhận đầu vào trộn lẫn dữ liệu không tin cậy với lệnh tin cậy, và không phân biệt được đâu là đâu**. SQL: interpreter là DB engine. XSS: interpreter là browser (HTML/JS parser). Command injection: interpreter là shell. Lời giải phổ quát vì thế cũng chung một hình dạng: **giữ dữ liệu và lệnh ở hai kênh tách biệt để interpreter không bao giờ nhầm** — prepared statement tách kênh ở protocol DB; output encoding "trung hòa" dữ liệu để browser không diễn giải nó như mã. CSRF hơi khác họ — nó không phải injection mà là lỗi **nhầm lẫn thẩm quyền (authority confusion)**: server suy ra thẩm quyền từ một tín hiệu (cookie) mà nó không kiểm soát được ai kích hoạt. Nhận ra "đây là bài toán tách dữ liệu/lệnh" vs "đây là bài toán xác thực nguồn gốc" giải thích tại sao dùng nhầm thuốc (validate input cho CSRF) là vô ích.

### 6. Trade-off
- **Prepared statement:** an toàn tuyệt đối với injection + được DB cache plan ↔ khó với truy vấn động (tên bảng/cột động cần allowlist, không parameterize được identifier).
- **CSP nghiêm ngặt:** chặn XSS mạnh ↔ tốn công cấu hình, dễ vỡ tính năng (inline script, third-party widget), cần nonce/hash.
- **SameSite=Strict:** chống CSRF tốt nhất ↔ phá luồng đăng nhập qua link ngoài (user click link email vào site đã login lại thấy như chưa login); `Lax` là cân bằng phổ biến.
- **WAF:** chặn được lớp tấn công tự động, mua thời gian ↔ false positive chặn user thật, tạo cảm giác an toàn giả khiến lơ là sửa gốc.

### 7. Ví dụ Production
Sự cố thật kiểu mẫu: đội dùng ORM nên tin "ORM chống SQL injection rồi", nhưng một chỗ dùng raw query cho báo cáo động — `ORDER BY ` + tên cột từ query param → injection qua clause ORDER BY (không parameterize được identifier). Kẻ tấn công dùng subquery trong ORDER BY để exfiltrate dữ liệu từng bit (blind injection). Fix: allowlist tên cột được sort, không bao giờ nối identifier từ input. Bài học: **ORM giảm rủi ро nhưng không miễn nhiễm** — mọi raw query và mọi phần không parameterize được (identifier, LIMIT động) là điểm mù. Với XSS, sự cố phổ biến nhất tôi thấy là `dangerouslySetInnerHTML` render markdown/HTML do user nhập mà không sanitize (thiếu DOMPurify) — một comment chứa `<img onerror=...>` là đủ.

### 8. Những câu trả lời chưa đủ tốt
- "Chống SQL injection bằng cách escape ký tự đặc biệt." → Chống chỉ định. Prepared statement mới là gốc; escape thủ công luôn sót ca.
- "Validate input là chống được XSS/injection." → Validate là một lớp, không phải gốc. XSS phải xử lý ở output theo ngữ cảnh; injection ở tầng driver. Câu này lộ hiểu hời hợt.

### 9. Sai lầm phổ biến của ứng viên
- Gộp cả ba thành "lọc input là xong" — không hiểu mỗi cái phòng thủ ở tầng khác nhau.
- Tin ORM/framework miễn nhiễm hoàn toàn — quên raw query, `v-html`, identifier động.
- Nhầm CSRF với XSS — hai cơ chế hoàn toàn khác, thuốc khác.
- Không biết CSP, hoặc nghĩ nó thay được output encoding (nó là lớp bổ sung).
- Không biết stored XSS nguy hiểm hơn reflected (phát tán tới mọi người xem), hoặc không biết NoSQL/command injection cùng họ.

### 10. Follow-up Questions
- Blind SQL injection là gì? Time-based blind injection hoạt động thế nào khi không có output trực tiếp?
- API-only backend (không render HTML) có cần lo XSS không? (có — nếu client render dữ liệu từ API; XSS là lỗ hổng render, không phải lỗ hổng server render.)
- SameSite=Lax không bảo vệ trường hợp nào? (top-level GET có side effect — nên GET đừng bao giờ có side effect.)
- CSP với nonce hoạt động thế nào cho inline script cần thiết?
- Nếu phát hiện đang bị injection tấn công trên production ngay lúc này, anh làm gì theo thứ tự? (WAF rule tạm, patch, xoay credential DB, kiểm tra dữ liệu đã lộ, audit log.)

### 11. Liên hệ với Production
OWASP Top 10 là chuẩn ngành; các công ty nghiêm túc có SAST/DAST trong CI, security review cho code chạm auth/query, và bug bounty. Vấn đề nghiêm trọng khi: hệ thống nhận input từ nhiều nguồn (form, API, file upload, webhook), hoặc render nội dung do user tạo (UGC — comment, profile, markdown). Dấu hiệu cần rà soát: có raw SQL nối chuỗi trong codebase, dùng `dangerouslySetInnerHTML`/`innerHTML` với dữ liệu user, không có CSP header, cookie session thiếu `SameSite`/`HttpOnly`/`Secure`, và không có ai review khía cạnh security cho các PR chạm vùng nhạy cảm.

---

## Câu 4 — [Senior → Staff] Lưu trữ mật khẩu, quản lý secrets, và bảo mật giữa các service (mTLS)

### 1. Câu hỏi
"Lưu password thế nào cho đúng, và tại sao? Secrets (API key, DB password) quản lý ra sao ở quy mô nhiều service? Các service nội bộ xác thực lẫn nhau bằng gì?"

### 2. Interviewer muốn kiểm tra điều gì?
- Kiến thức nền tảng crypto ứng dụng: hash vs encrypt, salt, tại sao bcrypt/argon2 chứ không SHA256.
- Tư duy vận hành: secrets không nằm trong code/env plaintext, rotation, blast radius.
- Bảo mật zero-trust nội bộ: không tin mạng nội bộ mặc định.

### 3. Câu trả lời ngắn gọn (30 giây)
"Password: **không bao giờ mã hóa** (encrypt là reversible), mà **hash một chiều** bằng thuật toán **cố tình chậm** — bcrypt, scrypt, hoặc argon2 (khuyến nghị nhất hiện nay) — với salt ngẫu nhiên per-user (chống rainbow table) và work factor điều chỉnh được (chống brute-force khi phần cứng mạnh lên). Không dùng SHA256/MD5 vì chúng nhanh — nhanh là điểm yếu khi hash password, GPU thử hàng tỷ lần/giây. Secrets: không để trong code/git/env plaintext; dùng secret manager (Vault, AWS Secrets Manager, KMS) với truy cập theo IAM, rotation tự động, audit log. Service-to-service: đừng tin mạng nội bộ — dùng mTLS (cả hai phía trình chứng chỉ) hoặc workload identity (SPIFFE) để mỗi service chứng minh danh tính mật mã, thay cho 'nó gọi từ IP nội bộ nên tin'."

### 4. Câu trả lời Senior Level (3–5 phút)
**Password hashing:**
- **Hash, không encrypt:** encrypt reversible → ai có key đọc được mọi password; hash một chiều → kể cả DB bị lộ, password không khôi phục trực tiếp. Server không cần biết password gốc, chỉ cần verify (hash lại input, so sánh).
- **Salt per-user:** salt ngẫu nhiên trộn vào trước khi hash → hai người cùng password có hash khác nhau; vô hiệu rainbow table (bảng hash tính sẵn) và lộ "ai trùng password với ai".
- **Cố tình chậm (key stretching):** bcrypt/scrypt/argon2 có tham số work factor/cost. SHA256 tính ~tỷ lần/giây trên GPU → brute-force password yếu trong giây. bcrypt cost 12 → mỗi lần hash ~250ms → brute-force bất khả thi kinh tế. **Argon2id** còn chống cả tấn công GPU/ASIC (memory-hard — tốn RAM, khó song song hóa phần cứng rẻ). Điều chỉnh cost theo thời gian khi phần cứng mạnh lên.
- **Pepper (tùy chọn):** secret toàn cục lưu tách khỏi DB (trong HSM/secret manager) trộn thêm → DB lộ mà không lộ pepper thì hash vẫn khó phá.

**Secrets management:**
- Nguyên tắc: secret không nằm trong source code, không commit git (dù git private — lịch sử git là mãi mãi), không hardcode. Env var plaintext tốt hơn hardcode nhưng vẫn lộ qua process listing/crash dump/log.
- Đúng: secret manager tập trung (Vault/AWS SM/GCP SM) — app lấy secret lúc runtime qua danh tính (IAM role/workload identity), không có secret tĩnh trên đĩa. Kèm: **rotation tự động** (đổi định kỳ + đổi khẩn khi nghi lộ), **audit** (ai lấy secret nào khi nào), **least privilege** (mỗi service chỉ đọc secret của nó — giới hạn blast radius khi một service bị hack).
- Encryption: dữ liệu nhạy cảm mã hóa at-rest (KMS quản key, envelope encryption) và in-transit (TLS mọi nơi, kể cả nội bộ).

**Service-to-service (zero trust):**
- Sai lầm truyền thống: "trong VPC nên tin nhau" — một service bị xâm nhập là kẻ tấn công đi ngang toàn hệ thống (lateral movement).
- Đúng: mỗi service có danh tính mật mã. **mTLS** (mutual TLS): cả client và server trình chứng chỉ, xác thực hai chiều → chỉ service có cert hợp lệ mới gọi được. Service mesh (Istio/Linkerd) tự động hóa cấp/xoay cert. **SPIFFE/SPIRE** chuẩn hóa workload identity. Cộng với authz per-call (service A được gọi endpoint nào của B).

### 5. Giải thích bản chất
Password hashing là ứng dụng của một nguyên lý bất đối xứng đẹp: **làm cho việc kiểm tra rẻ nhưng việc đoán đắt**. Server verify một password = một lần hash (chấp nhận được, ~vài trăm ms cho một lần login). Kẻ tấn công đoán = hàng tỷ lần hash (bất khả thi). "Cố tình chậm" đảo ngược trực giác kỹ sư (thường tối ưu cho nhanh) vì ở đây **chậm là tính năng bảo mật**, không phải bug. Argon2 memory-hard đẩy xa hơn: khai thác chỗ phần cứng tấn công (GPU/ASIC) mạnh về tính toán nhưng yếu về băng thông bộ nhớ. Về secrets, nguyên lý nền là **giảm blast radius**: giả định mọi thứ rồi sẽ bị lộ, thiết kế sao cho một lần lộ gây thiệt hại nhỏ nhất và thu hồi nhanh nhất — rotation, least privilege, tách secret khỏi nơi nó được dùng. Zero-trust nội bộ là cùng triết lý áp cho mạng: **không có "vùng an toàn"** — chu vi (perimeter) không tồn tại trong thế giới cloud/microservices, nên xác thực phải ở mọi biên, không chỉ ở cổng ngoài.

### 6. Trade-off
- **Work factor cao (bcrypt cost/argon2 params):** khó brute-force ↔ login chậm hơn + tốn CPU/RAM server (login storm có thể thành DoS tự gây — cân nhắc rate limit login).
- **Secret manager tập trung:** an toàn, audit, rotation ↔ thêm dependency (nó down thì app không lấy được secret — cần cache + fallback), độ phức tạp vận hành.
- **mTLS mọi nơi:** zero-trust thật ↔ chi phí quản lý cert (cấp, xoay, thu hồi — không có service mesh thì cực nhọc), latency handshake, debug khó hơn.
- **Rotation thường xuyên:** cửa sổ lộ nhỏ ↔ rủi ro outage khi rotation lỗi (secret cũ hết hạn trước khi mới lan tới mọi nơi) — rotation phải grace period (chấp nhận cả cũ lẫn mới trong thời gian chuyển).

### 7. Ví dụ Production
Vụ lộ dữ liệu kinh điển của ngành: các công ty bị hack DB và lộ password hash bằng **SHA1/MD5 không salt** (hoặc plaintext) → hàng trăm triệu password bị crack trong ngày qua rainbow table, và vì user dùng chung password khắp nơi, thiệt hại lan sang mọi dịch vụ khác. Đây là lý do argon2/bcrypt + salt là bắt buộc tuyệt đối. Sự cố secrets phổ biến nhất tôi thấy: AWS key commit nhầm lên GitHub public → bot quét git tìm thấy trong **vài phút** → dựng EC2 đào coin, hóa đơn chục nghìn đô qua đêm. Fix và phòng: secret scanning trong CI (git-secrets, GitHub secret scanning), rotation ngay khi lộ, và không bao giờ để secret rời khỏi secret manager. Bài học kép: giả định mọi secret sẽ lộ, và thiết kế để lộ không phải là tận thế.

### 8. Những câu trả lời chưa đủ tốt
- "Hash password bằng SHA256 cho an toàn." → SHA256 nhanh = tệ cho password. Cần thuật toán cố tình chậm + salt. Câu này lộ thiếu kiến thức crypto ứng dụng.
- "Để secret trong biến môi trường là đủ." → Tốt hơn hardcode nhưng vẫn lộ qua nhiều đường; ở quy mô nhiều service cần secret manager + rotation. Không nhắc rotation/least privilege là chưa đủ tầm Senior.

### 9. Sai lầm phổ biến của ứng viên
- Nhầm hash và encrypt; hoặc dùng hash nhanh (SHA/MD5) cho password.
- Quên salt, hoặc dùng salt cố định toàn hệ thống.
- Tin mạng nội bộ an toàn ("sau firewall rồi") — không có zero-trust.
- Commit secret vào git rồi nghĩ "xóa commit là xong" (lịch sử git giữ mãi, phải rotate secret).
- Không có kế hoạch rotation; secret sống vĩnh viễn.
- Tự nghĩ ra thuật toán mã hóa/hash "cho chắc" — vi phạm nguyên tắc "đừng tự chế crypto".

### 10. Follow-up Questions
- Argon2 có ba biến thể (i, d, id) — khác nhau và chọn cái nào? Tại sao id?
- Rotation một DB password đang được 20 service dùng, không downtime — quy trình? (dual credential, grace period.)
- User quên password — reset flow an toàn thế nào? (token một lần, hết hạn ngắn, không tiết lộ email có tồn tại hay không.)
- mTLS vs JWT cho service-to-service — khi nào dùng cái nào, kết hợp được không?
- Nếu phát hiện secret bị lộ, checklist ứng phó theo thứ tự? (rotate ngay, đánh giá phạm vi truy cập bằng audit log, kiểm tra dữ liệu bị chạm, thông báo.)

### 11. Liên hệ với Production
Các công ty trưởng thành về bảo mật đều có: password argon2id, secret manager + rotation tự động, mTLS/service mesh cho traffic nội bộ, secret scanning trong CI, và mã hóa at-rest qua KMS. Vấn đề nghiêm trọng khi: hệ thống lưu dữ liệu nhạy cảm (PII, tài chính, y tế) chịu compliance (PCI-DSS, GDPR, HIPAA), hoặc khi số service tăng khiến quản lý secret/cert thủ công vỡ trận. Dấu hiệu cần hành động: password hash bằng thuật toán nhanh, secret trong env/code/git, traffic nội bộ plaintext, không có rotation, không audit được "ai truy cập secret gì" — và câu hỏi kiểm toán: "nếu DB production bị lộ nguyên vẹn ngay bây giờ, kẻ tấn công làm được gì?" — câu trả lời càng dài, nợ bảo mật càng lớn.
