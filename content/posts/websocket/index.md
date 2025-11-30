+++
title = "Hướng dẫn implement 1 hệ thống realtime sử dụng Socket IO bằng GoLang"
date = "2025-09-10T15:00:00+07:00"
draft = false
tags = ["socket-io", "websocket"]
+++

Hôm nay nhân dịp cuối tuần cùng nhìn lại những gì đã làm trong 2 tuần làm việc vừa qua của mình sau khi join vào 1 công ty mới, mình ngồi viết những dòng này để hệ thống lại những gì đã làm trong tuần qua, cũng như summerize lại kiến thức để tránh việc chảy máu chất xám. Hôm nay mình sẽ nói về việc xây dựng 1 hệ thống realtime backend sử dụng socket.io với ngôn ngữ lập trình Golang. Mình mới chuyển từ nodejs sang Go lang =))

Nào hãy cùng bắt đầu cùng mình nhé.

### 1. Websocket là gì? So sánh websocket với socketIO.
WebSocket là giao thức giúp giữ kết nối 2 chiều liên tục giữa client và server. Khác với HTTP (request-response), WebSocket cho phép:

Server gửi dữ liệu chủ động về client mà không cần client hỏi lại.

Trao đổi dữ liệu real-time, độ trễ thấp.

Kết nối được giữ mở đến khi đóng.


Vì WebSocket tạo ra kết nối 2 chiều persistent (giữ mở liên tục). Nên Sau khi handshake thành công, client ↔ server có thể tự do gửi dữ liệu bất kỳ lúc nào không cần request lặp lại.


Websocket dùng HTTP/1.1 để handshake (Upgrade header)

Sau handshake → chuyển sang WebSocket protocol riêng

ví dụ dễ thấy khi bạn mở 1 kết nối websocket thì bấm f12 ở trên chrome bạn sẽ thấy 1 phần inspect tương tự như thế này:
GET /ws HTTP/1.1
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: xxxxx
Sec-WebSocket-Version: 13

Sau khi thiết lập kết nối từ client đến server, nó sẽ giữ lại kết nối và Gửi ping/pong frame định kỳ → phát hiện disconnect. Không mất thời gian mở/đóng kết nối nhiều lần như HTTP, tuy nhiên chính vì giữ kết nối như vậy nó sẽ làm tiêu tốn resource của server nhiều hơn so với RestAPI truyền thống.

#### So sánh với socketIO


Dùng WebSocket giống như là bạn code thuần vậy còn nếu dùng socketIO thì bạn sẽ được cung cấp các implement sẵn ví dụ như room, namespace, auto reconnect,... Hãy tưởng tượng giống như khi code 1 HTTP bằng code thuần so với sử dụng framework thì cái nào sẽ tiện ích hơn. Dĩ nhiên ai chả thích nhanh gọn phải không, mình cũng vậy hihi.

Tuy nhiên SocketIO không phải là websocket thuần, nó hỗ trợ nhiều transport trong đó có websocket và http longpolling. Flow: client try WebSocket → nếu fail → fallback HTTP polling.

### 2. Các khái niệm cơ bản trong Socket IO.

#### 2.1 Event-based Communication
Socket.IO không truyền raw message, nó dùng event model:
ví dụ:
```
socket.emit("chat_message", { text: "hello" })
socket.on("chat_message", handler)
```
Ưu điểm: clean interface cho logic realtime.

#### 2.1 Namespace là gì?

Chia logic theo endpoint

Kiểu microservice cho realtime

#### 2.2 Room là gì?

#### 2.3 Cơ chế pubsub.

### 2. Implement 1 service đơn giản với Golang.


### 3. Tổng kết, nhận xét.

Có 1 thực tế là khi làm các ứng dụng với socket.IO thì việc sử dụng Golang sẽ không thể tốt hơn Nodejs được. Nodejs sử dụng event loop hoạt động rất tốt với các kiểu xử lý dạng IObound như này, trong khi golang thiên về CPU-bound nhiều hơn. Tuy nhiên trong bài viết này mình không đề cập đến việc sử dụng nodejs tại vì nó quá là quen thuộc rồi.

Thư viện SocketIO mà mình đang sử dụng hiện tại chỉ implement đến socketIO version 2 thôi, có nghĩa là nó sẽ bị miss đi 1 số chức năng so với phiên bản socketIO implement bằng Nodejs trên trang chủ chính thức của socket.io, việc này dẫn đến phía client cũng phải downgrade kết nối để không bị xảy ra vấn đề. 1 Điểm đáng chú ý là ở phiên bản v2 này bạn không thể truyền token thông qua header như v4 được mà phải truyền nó như 1 params. Thường thì 1 số tài liệu sẽ khuyến khích việc xác thực qua header hơn (JWT bảo mật tốt hơn qua header). Tuy nhiên bạn cũng có thể implement bàng cách gửi qua cac event /authenticate và nhận về kết quả thành công tại /authenticated hoặc thất bại tại /authenticate_failed.

Trong bài viết kế tiếp mình sẽ thử implement sang socketIO version 4 để có thể khắc phục những thiếu sót này.

Cám ơn bạn đã đọc đến đây, hẹn gặp lại.