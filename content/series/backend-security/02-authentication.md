+++
title = "Backend Security — Tập 2: Authentication"
date = "2026-07-07T09:00:00+07:00"
draft = false
tags = ["backend", "security"]
series = ["Backend Security"]
+++

> **Đối tượng:** Backend Engineer, Senior Backend Engineer, Tech Lead, Solution Architect, Software Architect.
>
> **Mạch tư duy:** Asset → Threat → Attack → Vulnerability → Defense → Trade-off → Production Best Practice.
>
> Tập này đi từ First Principles. Chúng ta sẽ **không** bắt đầu bằng "JWT là một chuẩn token". Thay vào đó, chúng ta bắt đầu từ câu hỏi gốc: *nếu HTTP không nhớ bạn là ai, thì làm sao một hệ thống biết bạn đã đăng nhập?* Từ đó, mỗi cơ chế xác thực sẽ hiện ra như một câu trả lời cho một điểm đau cụ thể của cơ chế trước nó.

---

## 0. First Principles: Tại sao Authentication là một bài toán khó

### HTTP là stateless — và đó là gốc rễ của mọi thứ

Giao thức HTTP không có trí nhớ. Mỗi request là một tờ giấy trắng: server nhận được nó mà không biết gì về các request trước đó. Bạn gõ đúng mật khẩu ở request đăng nhập — nhưng ở request tiếp theo (lấy danh sách đơn hàng), server đã "quên" bạn là ai.

Đây không phải lỗi thiết kế; statelessness là lý do HTTP mở rộng (scale) được. Nhưng nó tạo ra bài toán trung tâm của Authentication:

> **Làm sao để server nhận ra "đây vẫn là người vừa đăng nhập" ở mọi request tiếp theo, mà không bắt họ nhập lại mật khẩu mỗi lần?**

Nếu không giải được bài toán này, bạn có hai lựa chọn tồi: hoặc bắt người dùng nhập mật khẩu ở *mọi* request (không thể chấp nhận), hoặc không xác thực gì cả (mở toang hệ thống). Toàn bộ lịch sử tiến hóa của Authentication — từ Basic Auth đến JWT đến OAuth — là chuỗi các câu trả lời ngày càng tinh vi cho câu hỏi này, mỗi câu trả lời sửa một điểm yếu của câu trả lời trước.

### Threat Model chung cho mọi cơ chế Authentication

Trước khi đi vào từng cơ chế, hãy đóng đinh: attacker muốn gì và tấn công thế nào ở tầng xác thực?

- **Đánh cắp credential:** lấy mật khẩu qua phishing, keylogger, database bị lộ, hay dò (brute-force, credential stuffing dùng mật khẩu rò rỉ từ site khác).
- **Đánh cắp "vé thông hành":** không cần mật khẩu, chỉ cần đánh cắp cái token/cookie/session id mà server dùng để nhận diện bạn (qua XSS, sniff mạng, đọc log, CSRF lợi dụng).
- **Giả mạo (forge):** tự chế ra một token hợp lệ mà không cần đăng nhập (nếu cơ chế ký/kiểm tra yếu).
- **Replay:** bắt lại một request/token hợp lệ và gửi lại.
- **Không thu hồi được:** kể cả khi bạn biết một token đã bị lộ, làm sao vô hiệu hóa nó?

Mỗi cơ chế dưới đây sẽ được đánh giá qua đúng bộ mối đe dọa này.

```mermaid
flowchart LR
    A["HTTP stateless:<br/>server không nhớ bạn"] --> B["Cần một cách nhận diện<br/>xuyên nhiều request"]
    B --> C["Basic Auth<br/>gửi lại credential mỗi lần"]
    C -->|"credential phơi bày<br/>liên tục"| D["Session + Cookie<br/>server nhớ, client cầm session id"]
    D -->|"server phải lưu state,<br/>khó scale ngang"| E["Token / JWT<br/>đẩy state sang client"]
    E -->|"cần cấp quyền cho<br/>bên thứ ba an toàn"| F["OAuth2<br/>ủy quyền, không lộ mật khẩu"]
    F -->|"OAuth chỉ ủy quyền,<br/>không nói bạn LÀ AI"| G["OpenID Connect<br/>lớp danh tính trên OAuth2"]
```

---

## 1. Basic Authentication — Điểm khởi đầu và vì sao nó không đủ

### 1.1. Problem Statement

Cách ngây thơ nhất để giải bài toán statelessness: **đính kèm username và password vào mọi request.** Đó chính là HTTP Basic Authentication — client gửi header `Authorization: Basic <base64(username:password)>` ở mỗi request. Server giải mã, kiểm tra, cho qua nếu đúng.

Nó giải quyết được bài toán "server nhận ra bạn ở mọi request" một cách trực tiếp nhất. Nếu không có nó (hoặc thứ tương đương), không có cách nào để một request ẩn danh chứng minh danh tính.

### 1.2. Threat Model & Attack

Vấn đề chí mạng: **credential thật (mật khẩu) bị gửi đi lặp đi lặp lại trong mọi request.**

- `base64` **không phải mã hóa** — nó là mã hóa chuyển đổi có thể giải ngược tức thì. Bất kỳ ai bắt được request đều đọc được mật khẩu.
- Nếu không có HTTPS, mật khẩu bay qua mạng dưới dạng gần như plaintext → sniff là lấy được.
- Mật khẩu dễ lọt vào **log, proxy, browser history, cache**.
- Không có cơ chế **hết hạn** hay **thu hồi**: chừng nào mật khẩu còn đúng, "vé" còn hiệu lực. Muốn thu hồi phải đổi mật khẩu.
- Rất khó gắn MFA vào luồng Basic Auth thuần.

### 1.3. Tại sao nó tồn tại

Basic Auth là một phần của chuẩn HTTP từ rất sớm (RFC 1945/2617). Nó tồn tại vì **cực kỳ đơn giản** — không cần state phía server, không cần cơ sở hạ tầng gì thêm. Trong bối cảnh gọi API server-to-server nội bộ, script tự động, hay bảo vệ nhanh một endpoint admin sau HTTPS, nó vẫn còn đất dùng.

### 1.4. Cách hoạt động

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server
    C->>S: GET /resource (không có credential)
    S-->>C: 401 Unauthorized<br/>WWW-Authenticate: Basic
    C->>S: GET /resource<br/>Authorization: Basic base64(user:pass)
    Note over S: Giải base64, so khớp<br/>với store (đã hash)
    S-->>C: 200 OK
    Note over C,S: MỖI request tiếp theo<br/>đều đính kèm lại credential
```

### 1.5. Trade-off

Đơn giản tối đa (DX tốt cho việc test nhanh, không state → scale dễ) đổi lấy an toàn tối thiểu (credential phơi bày liên tục, không hết hạn, không thu hồi, không MFA). Về Performance nó rẻ, về Security nó đắt.

### 1.6–1.7. Best Practice & Anti-pattern

**Chỉ dùng Basic Auth trên HTTPS, và tốt nhất là chỉ cho server-to-server nội bộ hoặc công cụ.** Không bao giờ dùng cho đăng nhập người dùng cuối trên web/mobile. Anti-pattern: Basic Auth qua HTTP thuần; nhúng credential Basic vào URL (`https://user:pass@host` — lọt vào log/history); dùng nó làm cơ chế đăng nhập chính của một ứng dụng người dùng.

### 1.8–1.10. Case study, Troubleshooting & Khi nào KHÔNG dùng

Rất nhiều rò rỉ credential đến từ các endpoint nội bộ dùng Basic Auth mà vô tình để lộ ra Internet, hoặc để credential trong URL bị log lại. **Khi nào KHÔNG dùng:** gần như luôn luôn cho ứng dụng người dùng cuối — hãy dùng Session hoặc Token. Basic Auth là công cụ cho máy nói chuyện với máy trong môi trường kiểm soát, không phải cho con người đăng nhập.

---

## 2. Session Authentication — Server bắt đầu "nhớ"

### 2.1. Vì sao Session xuất hiện (Problem Statement)

Điểm đau lớn nhất của Basic Auth: mật khẩu bị gửi lặp lại mãi mãi. Ý tưởng đột phá của Session: **chỉ gửi mật khẩu đúng một lần lúc đăng nhập.** Sau khi xác thực thành công, server tạo ra một bản ghi phiên (session) trong bộ nhớ/DB của nó, và cấp cho client một **session id** — một chuỗi ngẫu nhiên vô nghĩa. Từ đó, client chỉ cần trình session id này; server tra nó trong kho session để biết "à, đây là user X đã đăng nhập".

Điểm mấu chốt: session id **không mang thông tin gì cả** — nó chỉ là một cái vé giữ chỗ. Toàn bộ sự thật ("vé này thuộc về ai, đăng nhập lúc nào, quyền gì") nằm ở **phía server**. Đây gọi là **stateful authentication**: server giữ state.

### 2.2. Threat Model & Attack

Khi credential thật không còn bay qua mạng liên tục, attacker chuyển mục tiêu sang **session id**:

- **Session hijacking:** đánh cắp session id (qua sniff nếu không HTTPS, qua XSS đọc cookie, qua log). Có id là "là" user đó, không cần mật khẩu.
- **Session fixation:** attacker ép nạn nhân dùng một session id do attacker biết trước, rồi chiếm phiên sau khi nạn nhân đăng nhập.
- **CSRF:** vì session id thường nằm trong cookie tự động gửi kèm, một site độc hại có thể khiến trình duyệt nạn nhân gửi request kèm cookie hợp lệ (chi tiết ở Tập Browser Security).

### 2.3. Cách hoạt động bên trong

```mermaid
sequenceDiagram
    participant C as Browser
    participant S as Server
    participant St as Session Store<br/>(Redis/DB)
    C->>S: POST /login (user + pass) — MỘT lần duy nhất
    S->>S: Xác thực (so hash mật khẩu)
    S->>St: Tạo session {id, userId, createdAt, ...}
    S-->>C: Set-Cookie: session_id=abc... (HttpOnly)
    Note over C: Trình duyệt tự lưu cookie
    C->>S: GET /orders — Cookie: session_id=abc...
    S->>St: Tra session_id=abc → userId=42
    St-->>S: {userId: 42, role: customer}
    S-->>C: 200 OK (dữ liệu của user 42)
```

### 2.4. Điểm mạnh cốt lõi: Revocation dễ dàng

Vì server nắm state, muốn **đăng xuất / thu hồi** một phiên chỉ cần xóa bản ghi session khỏi store. Ngay lập tức, session id đó vô giá trị. Đây là **ưu điểm lớn nhất của session so với token tự chứa** (sẽ thấy ở phần JWT): kiểm soát tập trung, thu hồi tức thì. "Đăng xuất khỏi mọi thiết bị" = xóa mọi session của user đó — chuyện đơn giản.

### 2.5. Trade-off

- **Scalability:** Đây là gánh nặng chính. Server phải tra session store ở *mỗi* request. Với nhiều instance backend (scale ngang), tất cả phải chia sẻ một session store chung (thường là Redis) → thêm một thành phần hạ tầng, thêm một điểm phụ thuộc và điểm lỗi. "Sticky session" (ghim user vào một instance) là giải pháp tệ vì phá vỡ cân bằng tải và khả năng chịu lỗi.
- **Performance:** Mỗi request tốn một lượt tra store (thường rất nhanh với Redis, nhưng vẫn là round-trip mạng).
- **Security:** Tốt về revocation; rủi ro nằm ở việc bảo vệ session id (cookie) và chống CSRF.
- **DX:** Đơn giản, chín muồi, được mọi framework hỗ trợ sẵn.

### 2.6. Best Practice

Dùng session id **ngẫu nhiên đủ dài, không đoán được** (CSPRNG). **Tái tạo session id sau khi đăng nhập** (chống session fixation). Đặt **thời hạn tuyệt đối và thời hạn nghỉ (idle timeout)**. Lưu session id trong cookie với đầy đủ cờ bảo vệ: `HttpOnly` (JS không đọc được → chống trộm qua XSS), `Secure` (chỉ gửi qua HTTPS), `SameSite` (chống CSRF). Dùng session store tập trung (Redis) khi scale ngang. Với thao tác nhạy cảm, yêu cầu **re-authentication** dù phiên còn hiệu lực.

### 2.7. Anti-pattern

Lưu session id trong `localStorage` (JS đọc được → mồi ngon cho XSS) thay vì cookie `HttpOnly`; session id ngắn/đoán được; không đặt hết hạn (phiên sống vĩnh viễn); không tái tạo id sau login; in-memory session không chia sẻ được khi có nhiều instance (user "bị đăng xuất ngẫu nhiên" khi load balancer đổi instance).

### 2.8. Production Case Study & 2.9. Troubleshooting

Vấn đề vận hành kinh điển: một ứng dụng chạy tốt trên một server, khi scale lên nhiều instance thì người dùng "bị đăng xuất ngẫu nhiên" — nguyên nhân là session lưu in-memory từng instance, load balancer route request sang instance không có session đó. Giải pháp là chuyển session sang store dùng chung. Khi troubleshoot session, hãy log vòng đời session (tạo, tra cứu, hết hạn, thu hồi) và giám sát tỷ lệ "session miss".

### 2.10. Khi nào KHÔNG nên dùng Session

Khi bạn cần xác thực **cross-domain / xuyên nhiều dịch vụ độc lập** (nhiều API, nhiều domain, mobile + web + third-party), mô hình session-cookie trở nên vướng víu, và nhu cầu về một credential **tự chứa, không cần tra store tập trung** xuất hiện — đó là lúc Token/JWT vào cuộc. Session vẫn là lựa chọn tuyệt vời cho ứng dụng web truyền thống một domain, server-rendered.

---

## 3. Cookie-Based Authentication — Bản chất là cơ chế vận chuyển

### 3.1. Làm rõ một nhầm lẫn phổ biến

Nhiều người đối lập "Cookie" với "Token" như hai cơ chế xác thực cạnh tranh. Điều này gây hiểu nhầm. **Cookie không phải một phương pháp xác thực; cookie là một cơ chế lưu trữ và vận chuyển ở phía trình duyệt.** Bạn có thể để *bất cứ thứ gì* trong cookie — một session id (session auth), hoặc thậm chí một JWT (token auth qua cookie).

Câu hỏi đúng không phải "Cookie hay Token?" mà là hai câu hỏi độc lập:
1. **Credential mang thông tin gì?** — một id tham chiếu (stateful, cần tra store) hay một token tự chứa (stateless)?
2. **Credential được vận chuyển và lưu ở đâu?** — trong Cookie (trình duyệt tự quản) hay trong header `Authorization` (JS tự quản, thường lưu ở bộ nhớ/localStorage)?

### 3.2. Bản chất Cookie vs Token khác nhau ở đâu

Điểm khác biệt bản chất nằm ở **ai điều khiển việc gửi credential**:

- **Cookie:** *trình duyệt tự động* đính kèm cookie vào mọi request tới đúng domain — kể cả request do một site khác kích hoạt. Đây là nguồn gốc của **CSRF**: sự tiện lợi "tự động gửi" bị lợi dụng. Bù lại, cookie có thể được đánh dấu `HttpOnly` để JavaScript *không đọc được*, giúp **chống đánh cắp qua XSS** rất hiệu quả.
- **Token trong `Authorization` header:** *JavaScript của ứng dụng phải chủ động đính kèm* token vào mỗi request. Không tự động gửi → **miễn nhiễm CSRF một cách tự nhiên**. Nhưng để JS đính kèm được thì JS phải *đọc được* token → nếu lưu ở nơi JS truy cập được (localStorage), nó **phơi bày trước XSS**.

```mermaid
flowchart TD
    subgraph Cookie["Lưu trong Cookie (HttpOnly)"]
        C1["✅ JS không đọc được → chống XSS-trộm-token"]
        C2["❌ Trình duyệt tự gửi → cần chống CSRF (SameSite, CSRF token)"]
    end
    subgraph Header["Lưu cho Authorization header (vd localStorage)"]
        H1["✅ Không tự gửi → miễn nhiễm CSRF"]
        H2["❌ JS đọc được → XSS đọc được token"]
    end
```

### 3.3. Kết luận bản chất

Đây là một **trade-off đối xứng, không có lựa chọn thắng tuyệt đối**:

> **Cookie `HttpOnly`:** an toàn trước XSS-trộm-token, nhưng phải xử lý CSRF.
> **Token trong header/localStorage:** an toàn trước CSRF, nhưng phơi bày trước XSS.

Vì **XSS là lỗ hổng phổ biến và nguy hiểm hơn CSRF** trong thực tế (và CSRF đã có `SameSite` cookie làm lá chắn mạnh), xu hướng best practice hiện đại nghiêng về: **lưu token nhạy cảm (đặc biệt refresh token) trong cookie `HttpOnly` + `Secure` + `SameSite`**, thay vì localStorage. Chi tiết cơ chế cookie an toàn (HttpOnly, Secure, SameSite) và CSRF thuộc về Tập Browser Security; ở đây chỉ cần nắm bản chất trade-off.

---

## 4. Token Authentication & Stateless Authentication — Đẩy state sang client

### 4.1. Vì sao Token xuất hiện (Problem Statement)

Điểm đau của session là **statefulness**: server phải nhớ mọi phiên, phải tra store ở mọi request, và mọi instance phải chia sẻ store đó. Trong thế giới microservices, mobile app, và API dùng bởi nhiều bên, điều này thành gánh nặng: mỗi service phải gọi tới session store trung tâm chỉ để biết "request này là của ai".

Ý tưởng của **stateless / token authentication**: thay vì cấp một *id tham chiếu* (phải tra store để hiểu), hãy cấp một **token tự chứa (self-contained)** — bản thân token *chứa sẵn* thông tin danh tính ("user 42, role customer, hết hạn lúc X") và được **ký bằng chữ ký số**. Server nhận token chỉ cần **xác minh chữ ký** để tin rằng nội dung không bị giả mạo, rồi đọc thẳng thông tin bên trong — **không cần tra bất kỳ store nào.**

Đây là sự đảo chiều triết học: thay vì "server nhớ, client cầm vé giữ chỗ" (session), giờ là "**client cầm toàn bộ sự thật đã được server ký tên, server không cần nhớ gì**" (stateless token).

### 4.2. Threat Model

Sự đảo chiều này chuyển dịch các mối đe dọa:

- **Forgery (giả mạo):** nếu client cầm cả nội dung, điều gì ngăn họ tự sửa "role customer" thành "role admin"? → Câu trả lời là **chữ ký số**: sửa nội dung thì chữ ký sai. Toàn bộ an toàn dồn vào sự đúng đắn của thuật toán ký và bảo vệ khóa ký.
- **Token theft:** vẫn như session — ai cầm token là "là" chủ nhân. Nhưng nghiêm trọng hơn ở điểm sau.
- **Revocation cực khó:** đây là cái giá lớn nhất của statelessness. Vì server *không lưu* token, nó *không có danh sách* token đang hiệu lực để mà xóa. Một token đã ký, một khi phát ra, **hợp lệ cho tới khi hết hạn** — kể cả khi bạn biết nó đã bị đánh cắp. Với session, thu hồi = xóa một dòng. Với token stateless, "thu hồi" là một bài toán khó cần giải pháp riêng (xem 4.4 và phần Refresh/Revocation).

### 4.3. Cách hoạt động

```mermaid
sequenceDiagram
    participant C as Client
    participant A as Auth Service
    participant S1 as Service A
    participant S2 as Service B
    C->>A: Đăng nhập (user + pass)
    A->>A: Xác thực, rồi KÝ token chứa {sub:42, role, exp}
    A-->>C: access_token (đã ký)
    Note over C: Client tự lưu token
    C->>S1: request + token
    S1->>S1: Xác minh CHỮ KÝ bằng khóa công khai<br/>(KHÔNG tra store)
    S1-->>C: 200 OK
    C->>S2: request + token
    S2->>S2: Cũng chỉ cần verify chữ ký
    S2-->>C: 200 OK
    Note over S1,S2: Không service nào cần<br/>session store chung
```

### 4.4. Trade-off cốt lõi: Stateless đánh đổi Revocation lấy Scalability

| Tiêu chí | Session (stateful) | Token stateless |
|----------|--------------------|-----------------|
| Server lưu state | Có (session store) | Không |
| Mỗi request | Tra store | Chỉ verify chữ ký |
| Scale ngang / microservices | Cần store chung | Rất hợp — không phụ thuộc chung |
| **Revocation** | **Tức thì (xóa bản ghi)** | **Khó — phải chờ hết hạn** |
| Đăng xuất mọi thiết bị | Dễ | Khó |
| Kích thước credential | Nhỏ (id) | Lớn hơn (chứa claims) |

Đây là **trung tâm của mọi tranh luận session-vs-JWT**: bạn không "nâng cấp" từ session lên token; bạn **đổi** khả năng thu hồi tức thì lấy khả năng mở rộng không state. Best practice production thường là **kết hợp cả hai** (access token stateless ngắn hạn + refresh token có thể thu hồi), sẽ trình bày ở phần Refresh Token.

---

## 5. JWT (JSON Web Token) — Hiện thực phổ biến nhất của token, và những cạm bẫy của nó

### 5.1. Vì sao JWT xuất hiện (Problem Statement)

Ý tưởng "token tự chứa, có ký" cần một **định dạng chuẩn**: cấu trúc thế nào, ký ra sao, các bên khác nhau đọc/verify thống nhất kiểu gì. JWT (RFC 7519) là câu trả lời được chuẩn hóa và phổ biến nhất. Nó định nghĩa một token gồm ba phần, mã hóa base64url, phân tách bởi dấu chấm: **`header.payload.signature`**.

JWT giải quyết: một định dạng **liên vận hành (interoperable)** để mang claims đã ký, mà bất kỳ bên nào có khóa xác minh đều đọc và tin được — nền tảng cho stateless auth, SSO, và trao đổi danh tính giữa các hệ thống.

### 5.2. Cách hoạt động bên trong

Ba phần của JWT:

- **Header:** metadata, quan trọng nhất là `alg` (thuật toán ký, vd `HS256`, `RS256`) và `typ`.
- **Payload:** các **claims** — dữ liệu. Gồm claim chuẩn: `sub` (subject/chủ thể), `iss` (issuer), `aud` (audience), `exp` (hết hạn), `iat` (phát hành lúc), `nbf` (không dùng trước), `jti` (id token duy nhất) — và claim tùy biến (role, tenant...).
- **Signature:** ký lên `base64url(header) + "." + base64url(payload)` bằng khóa bí mật (HMAC với HS256) hoặc khóa riêng (chữ ký bất đối xứng với RS256/ES256).

```mermaid
flowchart LR
    subgraph JWT["JWT = header.payload.signature"]
        H["Header<br/>{alg: RS256, typ: JWT}"]
        P["Payload / Claims<br/>{sub:42, role, exp, iss, aud}"]
        S["Signature<br/>ký(header.payload, khóa)"]
    end
    H -. base64url .-> T["eyJ...  .  eyJ...  .  SflK..."]
    P -. base64url .-> T
    S -. base64url .-> T
```

**Điểm cực kỳ quan trọng — hai hiểu lầm chết người:**

> **(1) JWT được *ký*, KHÔNG được *mã hóa*.** Payload chỉ là base64url — **bất kỳ ai cũng đọc được** nội dung (dán vào jwt.io là thấy hết). Chữ ký chỉ đảm bảo *toàn vẹn* (không sửa được mà không bị phát hiện), **không đảm bảo *bí mật***. → **Tuyệt đối không để dữ liệu nhạy cảm** (mật khẩu, PII, số thẻ) trong payload JWT.
>
> **(2) "Verify JWT" chỉ chứng minh token *hợp lệ và chưa hết hạn*, KHÔNG chứng minh nó *chưa bị thu hồi*.** Một JWT của nhân viên vừa bị sa thải vẫn "verify pass" cho tới lúc `exp`.

### 5.3. Threat Model & Attack — JWT bị lạm dụng như thế nào

JWT nổi tiếng vì các lỗ hổng do triển khai sai. Đây là những cái kinh điển:

- **`alg: none` attack:** JWT cho phép giá trị `alg` là `none` (không ký). Thư viện cấu hình lỏng sẽ *chấp nhận* token không chữ ký nếu attacker set `alg:none` và bỏ phần signature → giả mạo bất kỳ claim nào. **Phòng:** luôn ép danh sách thuật toán cho phép (allowlist), từ chối `none`.
- **Nhầm lẫn thuật toán RS256 → HS256 (algorithm confusion / key confusion):** hệ thống dùng RS256 (khóa công khai để verify). Attacker đổi header thành `HS256` và **dùng chính khóa công khai (vốn ai cũng biết) làm khóa bí mật HMAC** để ký. Thư viện ngây thơ dùng "khóa verify" (khóa công khai) để verify HMAC → token giả pass. **Phòng:** cố định thuật toán mong đợi ở phía verify, không để header token quyết định.
- **Không kiểm `exp` / `aud` / `iss`:** chấp nhận token hết hạn, hoặc token cấp cho service khác (thiếu kiểm `aud`), hoặc token do issuer khác phát.
- **Khóa yếu với HS256:** secret HMAC ngắn/đoán được → brute-force offline ra khóa rồi tự ký token.
- **Không revoke được:** token bị lộ vẫn sống tới `exp` (vấn đề bản chất của stateless, mục 4.2).
- **Lưu sai chỗ:** để access token/refresh token trong localStorage → XSS đọc mất.

### 5.4. Trade-off

- **Security:** Toàn bộ dồn vào (a) quản lý khóa ký đúng, (b) cấu hình verify đúng (alg allowlist, kiểm exp/aud/iss), (c) không nhét dữ liệu nhạy cảm. Sai một trong ba → thảm họa.
- **Performance:** Verify HMAC/chữ ký rất nhanh, không round-trip store → tuyệt vời cho hệ thống phân tán. RS256 verify chậm hơn HS256 chút nhưng tách được quyền ký/verify.
- **Scalability:** Điểm mạnh lớn nhất — stateless, mọi service verify độc lập.
- **DX & Complexity:** Dễ bắt đầu, nhưng *rất dễ dùng sai*; kích thước token lớn hơn cookie session, gửi kèm mọi request tốn băng thông; revocation cần cơ chế bổ sung → phức tạp thật sự nằm ở vòng đời token.

### 5.5. Best Practice

**Access token JWT phải ngắn hạn** (vài phút đến ~15 phút) — đây là cách chính để bù cho việc khó thu hồi: token có lộ cũng sớm hết hạn. Luôn kiểm **`exp`, `iss`, `aud`** khi verify. **Ép thuật toán** (allowlist, ưu tiên RS256/ES256 để tách khóa ký khỏi khóa verify; nếu HS256 thì secret phải dài và ngẫu nhiên mạnh). Không để PII/secret trong payload. Đặt **`kid`** (key id) trong header để hỗ trợ **xoay khóa (key rotation)**. Kết hợp với refresh token (mục 7) để có đường thu hồi. Với nhu cầu thu hồi ngay, duy trì một **denylist (blocklist) token theo `jti`** cho tới khi hết hạn — chấp nhận thêm chút state để lấy khả năng revoke.

### 5.6. Anti-pattern

Access token JWT sống nhiều ngày/tháng; để dữ liệu nhạy cảm trong payload; tin `alg` từ token; dùng cùng một secret HMAC yếu khắp nơi; không có kế hoạch xoay khóa; lưu token trong localStorage; dùng JWT cho session web truyền thống một-domain (nơi session cookie đơn giản và an toàn hơn) chỉ vì "JWT hiện đại".

### 5.7. Production Case Study

Nhiều sự cố nghiêm trọng bắt nguồn từ chính **`alg:none`** và **RS256→HS256 confusion** trong các thư viện JWT đời đầu và cấu hình lỏng lẻo. Mô-típ: một API tin tưởng vào trường `alg` do client gửi trong header token. Attacker chỉ cần chỉnh header, hoặc bỏ chữ ký (`none`), hoặc ký HMAC bằng khóa công khai — và tự tạo ra token "admin". Bài học: **phía verify phải là bên quyết định thuật toán và khóa, không bao giờ để token tự khai báo cách nó nên được kiểm.** Một case study khác lặp đi lặp lại là logout "giả": ứng dụng dùng JWT stateless, người dùng bấm "đăng xuất" nhưng token cũ vẫn hợp lệ cho tới khi hết hạn — nếu nó đã bị đánh cắp, kẻ trộm vẫn dùng được. Đây là lý do refresh token và token sống ngắn là bắt buộc.

### 5.8. Troubleshooting

Log các lý do verify thất bại một cách phân loại: chữ ký sai (nghi giả mạo/nhầm khóa), hết hạn (`exp`), sai audience (`aud` — token gửi nhầm service), sai issuer. Khi xoay khóa, dùng `kid` để phân biệt token ký bằng khóa cũ/mới và hỗ trợ giai đoạn chuyển tiếp (chấp nhận cả hai khóa trong thời gian ngắn). Giám sát tỷ lệ token hết hạn để tinh chỉnh TTL và luồng refresh.

### 5.9. Khi nào KHÔNG nên dùng JWT

- **Ứng dụng web một domain, server-rendered:** session cookie đơn giản hơn, thu hồi dễ hơn, an toàn hơn. JWT ở đây là dùng dao mổ trâu.
- **Khi bạn cần thu hồi tức thì làm yêu cầu bắt buộc** (vd banking, khóa tài khoản ngay): stateful session hoặc reference token (token mờ + tra store) phù hợp hơn JWT thuần.
- **Khi payload cần chứa nhiều dữ liệu:** token phình to, gửi kèm mọi request tốn kém.

Giải pháp thay thế đáng cân nhắc: **opaque reference token** (token ngẫu nhiên vô nghĩa + introspection tra store) — kết hợp ưu điểm token (dùng cho API, mobile) với khả năng thu hồi của stateful.

---

## 6. OAuth2 — Ủy quyền, KHÔNG phải xác thực

### 6.1. Problem Statement — bài toán OAuth thật sự giải

Hãy tách bạch ngay từ đầu, vì đây là hiểu lầm phổ biến nhất trong toàn bộ chủ đề Authentication:

> **OAuth2 là một framework *ủy quyền (authorization)*, không phải giao thức *xác thực (authentication)*.**

Bài toán OAuth2 sinh ra để giải: *"Làm sao để ứng dụng A được phép truy cập một phần dữ liệu của tôi ở dịch vụ B, mà tôi KHÔNG phải đưa mật khẩu B cho A?"*

Ví dụ kinh điển: một app chỉnh sửa ảnh muốn truy cập ảnh của bạn trên một dịch vụ lưu trữ. Cách cũ (chống pattern): bạn đưa username/password của dịch vụ lưu trữ cho app đó. Thảm họa: app giờ có *toàn quyền* tài khoản bạn, mật khẩu bị chia sẻ, không thu hồi được trừ khi đổi mật khẩu, không giới hạn phạm vi. OAuth2 giải quyết bằng cách cấp cho app một **access token có phạm vi hẹp (scope) và có hạn**, thay cho mật khẩu — app chỉ đọc được đúng phần được cho phép, và bạn thu hồi quyền bất cứ lúc nào mà không đụng tới mật khẩu.

### 6.2. Vì sao OAuth KHÔNG phải Authentication

Điều OAuth2 chứng minh là: *"chủ tài nguyên đã đồng ý cấp cho ứng dụng này quyền truy cập phạm vi X"*. Nó **không** được thiết kế để chứng minh *"người đang đăng nhập là ai"*. Access token OAuth nói **"cái gì được phép"**, không nói **"ai đang hiện diện"**.

Sự nhầm lẫn "đăng nhập bằng OAuth" đến từ pattern "Login with [nhà cung cấp]". Nhưng dùng OAuth2 thuần để xác thực danh tính là **không an toàn** vì một lý do tinh tế: một access token được cấp cho *app này* có thể bị *app khác* đem dùng lại (**confused deputy / token substitution**) — vì access token không ràng buộc "ai là người dùng cuối đang đăng nhập vào ứng dụng của tôi". Đây chính là lỗ hổng lịch sử đã dẫn tới sự ra đời của OpenID Connect (mục 7... thực ra là mục kế). Nói ngắn gọn: **OAuth trả lời "app được làm gì", không trả lời "user là ai" — dùng nó cho việc thứ hai là sai mục đích và có rủi ro bảo mật.**

### 6.3. Cách hoạt động — Các vai và luồng Authorization Code

OAuth2 định nghĩa bốn vai:

- **Resource Owner:** người dùng, chủ của dữ liệu.
- **Client:** ứng dụng muốn truy cập dữ liệu.
- **Authorization Server:** cấp token sau khi người dùng đồng ý.
- **Resource Server:** nơi giữ dữ liệu, chấp nhận access token.

Luồng chuẩn và an toàn nhất cho web/mobile là **Authorization Code Flow** (kèm **PKCE** cho client công khai như SPA/mobile):

```mermaid
sequenceDiagram
    participant U as Resource Owner (User)
    participant C as Client App
    participant AS as Authorization Server
    participant RS as Resource Server

    U->>C: "Kết nối tài khoản B"
    C->>AS: Redirect: /authorize?client_id&scope&redirect_uri<br/>&code_challenge (PKCE)
    AS->>U: Đăng nhập ở B + màn hình đồng ý (consent)
    U->>AS: Đồng ý cấp scope
    AS-->>C: Redirect về redirect_uri?code=AUTH_CODE
    Note over C: Client đổi code lấy token<br/>(server-side, kèm code_verifier)
    C->>AS: POST /token (code + code_verifier + client_secret)
    AS-->>C: access_token (+ refresh_token)
    C->>RS: Gọi API + access_token
    RS->>RS: Verify token + kiểm scope
    RS-->>C: Dữ liệu (trong phạm vi scope)
```

Điểm thiết kế then chốt: **authorization code** là mã dùng-một-lần, đổi lấy token ở kênh sau (back channel) an toàn — access token không bao giờ lộ ra URL/trình duyệt. **PKCE** chống việc code bị chặn và dùng lại bởi app độc hại, đặc biệt quan trọng cho mobile/SPA không giữ được `client_secret`.

### 6.4. Các grant type khác & khi nào dùng

- **Authorization Code + PKCE:** mặc định cho hầu hết web/mobile/SPA hiện đại.
- **Client Credentials:** cho **machine-to-machine** (không có người dùng) — service này gọi service kia.
- **Device Code:** cho thiết bị không có trình duyệt/bàn phím tốt (TV, CLI).
- **Implicit Flow** và **Resource Owner Password Credentials (ROPC):** **đã bị khuyến nghị loại bỏ** — implicit để lộ token trong URL; ROPC bắt app cầm mật khẩu người dùng (phản bội chính mục đích của OAuth).

### 6.5. Trade-off

- **Security:** Rất mạnh khi làm đúng (không chia sẻ mật khẩu, scope hẹp, thu hồi được, token ngắn hạn). Nhưng **phức tạp**, nhiều biến thể, nhiều chỗ sai (redirect_uri không validate → chiếm token; thiếu PKCE; dùng grant type lỗi thời).
- **Complexity & DX:** Đường cong học dốc; dễ hiểu sai. Nên dùng thư viện/nhà cung cấp đã kiểm chứng, không tự cuộn (roll-your-own).
- **Scalability:** Tốt; token có thể là JWT stateless hoặc reference token.

### 6.6. Best Practice

Dùng **Authorization Code + PKCE**; loại bỏ Implicit và ROPC. **Validate `redirect_uri` chặt chẽ** theo allowlist chính xác (đây là bề mặt tấn công lớn — hở là bị chuyển hướng token cho attacker). Yêu cầu `state` để chống CSRF trong luồng redirect. Cấp **scope tối thiểu** cần thiết (Least Privilege). Access token ngắn hạn + refresh token. **Không tự triển khai OAuth server** trừ khi bắt buộc — dùng nhà cung cấp/thư viện chín muồi. Với M2M dùng Client Credentials, không nhét người dùng vào đó.

### 6.7. Anti-pattern

Dùng OAuth2 thuần làm cơ chế **đăng nhập/xác thực danh tính** (phải dùng OIDC — mục kế); `redirect_uri` dùng wildcard hoặc không validate; bỏ `state`/PKCE; xin scope quá rộng; dùng Implicit/ROPC; nhầm access token là bằng chứng danh tính người dùng; tự viết authorization server.

### 6.8. Production Case Study

Một lớp lỗ hổng OAuth kinh điển là **open redirect / redirect_uri lỏng lẻo**: nếu Authorization Server chấp nhận `redirect_uri` không khớp chính xác allowlist, attacker dựng một redirect_uri trỏ về máy họ, dụ nạn nhân bấm link "đăng nhập", và authorization code (hoặc token) bị gửi tới attacker. Một lớp khác là **token nhầm app** khi lập trình viên dùng access token như bằng chứng danh tính: app độc hại lấy access token mà người dùng cấp cho *nó* rồi đem trình cho app nạn nhân như thể "tôi là người dùng đó" — và app nạn nhân, vì xác thực bằng OAuth thay vì OIDC, đã tin. Cách phòng: validate redirect_uri tuyệt đối, dùng OIDC (ID Token) cho xác thực danh tính, và kiểm `aud` để đảm bảo token đúng là cấp cho mình.

### 6.9. Troubleshooting

Các lỗi hay gặp: `redirect_uri mismatch` (khai báo không khớp), `invalid_grant` (code hết hạn/đã dùng), scope bị từ chối, lệch đồng hồ gây token "chưa hợp lệ" (`nbf`) hay "hết hạn" (`exp`). Log rõ từng bước của luồng (authorize → callback → token exchange) và **không log token/code** ra nơi lưu trữ lâu dài.

### 6.10. Khi nào KHÔNG nên dùng OAuth2

Nếu bạn chỉ cần đăng nhập người dùng cho **chính ứng dụng của mình** (first-party), không có bên thứ ba nào truy cập dữ liệu thay người dùng, thì OAuth2 là **thừa và phức tạp không cần thiết** — một session cookie hoặc token first-party đơn giản là đủ và ít bề mặt lỗi hơn. OAuth tỏa sáng khi có **delegation** (ủy quyền cho bên thứ ba) hoặc bạn cần SSO/liên kết danh tính. Đừng kéo cả bộ máy OAuth vào chỉ vì "đăng nhập".

---

## 7. OpenID Connect (OIDC) — Lớp danh tính đặt trên OAuth2

### 7.1. Problem Statement & OAuth vs OIDC khác nhau thế nào

Vì OAuth2 *không* trả lời "người dùng là ai", các hệ thống lại *cần* câu trả lời đó để "Đăng nhập bằng [nhà cung cấp]", ngành công nghiệp đã chuẩn hóa một lớp mỏng đặt **trên** OAuth2: **OpenID Connect (OIDC)**. Quan hệ của chúng:

> **OAuth2 = Authorization (ủy quyền: "app được làm gì").**
> **OIDC = Authentication (xác thực danh tính: "user là ai"), xây trên OAuth2.**

OIDC bổ sung ba thứ then chốt vào OAuth2:

1. **ID Token** — một **JWT** chứa thông tin danh tính người dùng (`sub` = định danh người dùng, `iss`, `aud`, `exp`, `email`, `name`...), được ký bởi Authorization Server (giờ gọi là **OpenID Provider**). ID Token là thứ *thực sự* trả lời "user là ai" một cách an toàn — khác với access token.
2. **UserInfo endpoint** — nơi lấy thêm thông tin hồ sơ người dùng.
3. **Chuẩn hóa scope/claims** (`openid`, `profile`, `email`...).

Sự khác biệt bản chất so với việc "dùng access token để đăng nhập": **ID Token có `aud` = client của bạn** và ràng buộc vào phiên đăng nhập của *người dùng cuối vào ứng dụng của bạn*, nên không bị tấn công token-substitution như access token. Đây chính xác là lỗ hổng mà OIDC sinh ra để bịt.

### 7.2. Cách hoạt động (rút gọn)

Luồng OIDC giống Authorization Code Flow của OAuth2, chỉ thêm scope `openid` và nhận về **ID Token** cùng access token:

```mermaid
sequenceDiagram
    participant U as User
    participant C as Client (RP)
    participant OP as OpenID Provider
    U->>C: "Đăng nhập bằng nhà cung cấp"
    C->>OP: /authorize?scope=openid profile email ...
    OP->>U: Xác thực + đồng ý
    OP-->>C: code
    C->>OP: đổi code → ID Token (JWT) + access_token
    Note over C: Verify ID Token:<br/>chữ ký, iss, aud=chính mình, exp, nonce
    C-->>U: Đăng nhập thành công (biết "user là ai" qua sub)
```

### 7.3. Best Practice & Anti-pattern (ngắn gọn)

**Xác thực danh tính thì verify ID Token, không dùng access token cho mục đích này.** Kiểm đầy đủ `iss`, `aud` (phải bằng client id của bạn), `exp`, và `nonce` (chống replay). Dùng thư viện OIDC chín muồi. **Anti-pattern:** coi access token là bằng chứng danh tính; bỏ kiểm `aud`/`nonce`; tự parse ID Token mà không verify chữ ký. OIDC được nêu ở đây để bạn thấy **bức tranh tổng thể**: OAuth cấp quyền, OIDC xác thực danh tính — hai mảnh ghép bổ sung, không thay thế nhau.

---

## 8. Refresh Token, Rotation & Revocation — Giải quyết nghịch lý "ngắn hạn nhưng tiện dụng"

### 8.1. Problem Statement — nghịch lý cần giải

Chúng ta đã kết luận: access token phải **ngắn hạn** (để hạn chế thiệt hại khi bị đánh cắp, vì stateless khó thu hồi). Nhưng nếu access token chỉ sống 15 phút, chẳng lẽ bắt người dùng đăng nhập lại mỗi 15 phút? Đó là trải nghiệm không thể chấp nhận.

**Refresh token** giải nghịch lý này. Ý tưởng: cấp hai token với vai trò khác nhau:

- **Access token:** ngắn hạn (phút), stateless, gửi kèm *mọi* request tới API. Bị lộ cũng sớm hết hạn.
- **Refresh token:** dài hạn (ngày/tuần), **chỉ gửi tới Authorization Server** khi cần lấy access token mới, **không** gửi tới các API thông thường → bề mặt phơi bày nhỏ hơn nhiều. Và quan trọng: refresh token **có thể là stateful / thu hồi được** (lưu ở server).

Kết quả: hưởng lợi thế stateless tốc độ cao của access token cho mọi lời gọi API, đồng thời có một điểm kiểm soát (refresh token) để **thu hồi** khi cần — lấy lại thứ mà JWT thuần đã đánh mất.

```mermaid
sequenceDiagram
    participant C as Client
    participant AS as Auth Server
    participant API as API (Resource Server)
    C->>AS: Đăng nhập
    AS-->>C: access_token (15') + refresh_token (dài hạn)
    loop Trong 15 phút
        C->>API: request + access_token
        API-->>C: 200 (chỉ verify chữ ký, stateless)
    end
    C->>API: request + access_token (đã hết hạn)
    API-->>C: 401 (token expired)
    C->>AS: POST /token (refresh_token)
    Note over AS: Kiểm refresh_token trong store<br/>(có thể thu hồi tại đây)
    AS-->>C: access_token MỚI + refresh_token MỚI (rotation)
    C->>API: retry request + access_token mới
    API-->>C: 200
```

### 8.2. Rotation — vì sao xoay refresh token

Refresh token dài hạn là mục tiêu giá trị cao. **Refresh Token Rotation**: mỗi lần dùng refresh token để lấy access token mới, Authorization Server **cấp một refresh token mới và vô hiệu hóa cái cũ**. Lợi ích an ninh nằm ở khả năng **phát hiện đánh cắp**:

- Refresh token là **one-time-use**. Nếu một refresh token đã bị "tiêu" (đã đổi) mà *lại bị dùng lần nữa*, hệ thống biết chắc có chuyện bất thường — hoặc client hợp lệ hoặc attacker đang dùng một bản sao. **Reuse detection**: khi phát hiện một refresh token cũ (đã xoay) được tái sử dụng, Authorization Server **thu hồi toàn bộ chuỗi (token family)** của phiên đó, buộc đăng nhập lại. Điều này biến việc đánh cắp refresh token thành hành động **tự bại lộ**.

### 8.3. Revocation — các chiến lược thu hồi

Vì access token stateless khó thu hồi trực tiếp, các chiến lược thực tế:

- **Token sống ngắn + thu hồi ở tầng refresh:** thu hồi refresh token → chậm nhất sau khi access token hiện tại hết hạn (vài phút), attacker mất quyền vì không refresh được. Đây là mô hình phổ biến nhất, chấp nhận "cửa sổ" ngắn.
- **Denylist theo `jti`:** lưu id các token bị thu hồi cho tới khi chúng hết hạn; API kiểm denylist. Thêm chút state, đổi lấy thu hồi gần-tức-thì.
- **Opaque reference token + introspection:** token là chuỗi vô nghĩa, API hỏi Authorization Server (introspection) mỗi lần hoặc có cache ngắn → thu hồi tức thì nhưng mất tính stateless.
- **Sự kiện toàn cục:** "đăng xuất mọi thiết bị" / đổi mật khẩu → tăng một `token_version` trong hồ sơ user; token mang version cũ bị từ chối. Cần một lượt tra nhẹ nhưng rất hiệu quả.

### 8.4. Trade-off & Best Practice

Đây thuần túy là bài toán cân bằng **độ trễ thu hồi ↔ tính stateless ↔ độ phức tạp**. Best practice production điển hình: **access token JWT ngắn hạn (5–15') + refresh token stateful có rotation + reuse detection**, refresh token lưu trong **cookie `HttpOnly` + `Secure` + `SameSite`** (không để localStorage). Với hệ thống yêu cầu thu hồi tức thì (banking), bổ sung denylist `jti` hoặc reference token. Luôn thu hồi token family khi phát hiện reuse, khi đổi mật khẩu, khi người dùng báo mất thiết bị.

### 8.5. Anti-pattern

Refresh token vĩnh viễn không hết hạn/không xoay; lưu refresh token ở localStorage; không có reuse detection (mất hết giá trị an ninh của rotation); cấp access token dài hạn để "khỏi phiền refresh" (phá bỏ toàn bộ lợi ích ngắn-hạn); coi "đăng xuất" là chỉ xóa token phía client mà không thu hồi phía server (token bị đánh cắp vẫn sống).

### 8.6. Production Case Study

Mô-típ lặp lại: một ứng dụng phát access token JWT sống 30 ngày "cho tiện", không refresh token, không denylist. Một token bị lộ qua XSS. Vì không có đường thu hồi và token sống cả tháng, attacker duy trì quyền truy cập suốt 30 ngày bất kể người dùng đổi mật khẩu hay đăng xuất. Khắc phục sau sự cố luôn là cùng một công thức: rút ngắn access token, thêm refresh token có rotation + reuse detection, và một cơ chế thu hồi (token_version hoặc denylist). Bài học: **"tiện" bằng token dài hạn là vay nợ an ninh với lãi suất rất cao.**

---

## Tổng kết Tập 2 — Từ statelessness của HTTP đến kiến trúc token hiện đại

Toàn bộ hành trình Authentication là một chuỗi câu trả lời, mỗi cái sửa điểm đau của cái trước:

- **HTTP không nhớ bạn** → cần một cách nhận diện xuyên request.
- **Basic Auth** gửi lại mật khẩu mỗi lần → phơi bày credential liên tục, không hết hạn/thu hồi.
- **Session** chỉ gửi mật khẩu một lần, server nhớ, client cầm session id → thu hồi tức thì, nhưng **stateful, khó scale ngang**.
- **Cookie vs Token** không phải hai cơ chế xác thực cạnh tranh mà là **hai trục độc lập**: nội dung credential (tham chiếu vs tự chứa) và cách vận chuyển (cookie tự-gửi → lo CSRF; header JS-gửi → lo XSS).
- **Token/JWT stateless** đẩy state sang client, verify bằng chữ ký, tuyệt vời cho microservices → nhưng **đánh đổi khả năng thu hồi tức thì**, và *rất dễ triển khai sai* (`alg:none`, RS256↔HS256, quên kiểm exp/aud, lưu sai chỗ, nhét dữ liệu nhạy cảm).
- **OAuth2** giải bài toán **ủy quyền cho bên thứ ba không lộ mật khẩu** — và **không phải** cơ chế xác thực danh tính.
- **OIDC** đặt lớp **danh tính** (ID Token) lên OAuth2 để trả lời an toàn câu hỏi "user là ai".
- **Refresh token + rotation + revocation** hòa giải nghịch lý "ngắn hạn để an toàn" với "tiện dụng để dùng được", và trả lại khả năng thu hồi mà JWT thuần đã mất.

Không có cơ chế nào là "tốt nhất" tuyệt đối. Session vẫn xuất sắc cho web một-domain. JWT xuất sắc cho API/microservices/mobile nhưng đòi hỏi kỷ luật triển khai. OAuth/OIDC là bắt buộc khi có bên thứ ba và SSO. **Nhiệm vụ của architect không phải chọn "công nghệ hot nhất", mà là chọn đúng công cụ cho đúng mô hình mối đe dọa và đúng ràng buộc vận hành** — và hiểu rõ mỗi lựa chọn đang đánh đổi Security, Performance, Scalability, và khả năng thu hồi ra sao.

Các tập tiếp theo sẽ áp cùng tư duy này cho **Password Security** (vì sao phải hash chậm), **Transport Security** (TLS thực sự bảo vệ điều gì), **Browser Security** (CORS/CSRF/CSP/cookie), **API Security**, **OWASP Top 10**, **Server Security**, và **các kiến trúc thực tế** (e-commerce, banking, fintech, SaaS, microservices).
