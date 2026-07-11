+++
title = "Bài 0 — Giới Thiệu Series"
date = "2026-07-03T07:00:00+07:00"
draft = false
tags = ["backend", "interview"]
series = ["Backend Interview"]
+++

> Viết bởi góc nhìn của một Principal Software Architect đã phỏng vấn hàng trăm ứng viên Senior/Staff/Principal. Mục tiêu: không phải học thuộc đáp án, mà là **học cách tư duy**.

## Danh sách chương

| Chủ đề | Trọng tâm |
|--------|-----------|
| Golang Internals | GPM, Channel, GC, Escape Analysis, Concurrency Patterns |
| Node.js Internals | Event Loop, Stream, Worker Thread, Memory Leak |
| PostgreSQL | MVCC, WAL, Vacuum, Isolation, Planner, Replication |
| MongoDB, Redis, ClickHouse | Replica Set, Sharding, Distributed Lock, MergeTree |
| Message Queue & API | Kafka, RabbitMQ, SQS, NATS, REST/gRPC/GraphQL/WebSocket |
| Architecture & System Design | DDD, Microservices, CQRS, Saga, HA, Multi-region, Observability |
| Security & Authentication | JWT/Session, OAuth2/OIDC, OWASP, Password/Secrets, mTLS |
| Networking, Linux/OS & K8s | TCP/TIME_WAIT, HTTP/2/3, TLS, epoll/OOM, Container, Probes, Deploy |
| System Design Case Studies | Framework 45', Rate Limiter, URL Shortener, Notification, Payment |

## Cấu trúc mỗi câu hỏi (11 mục)

1. **Câu hỏi** — kèm nhãn độ khó: `Fundamental → Intermediate → Senior → Staff → Principal`
2. **Interviewer muốn kiểm tra điều gì**
3. **Câu trả lời ngắn gọn (30 giây)** — elevator answer
4. **Câu trả lời Senior Level (3–5 phút)** — Problem → Why → How → Trade-off → Production
5. **Giải thích bản chất** — first principles
6. **Trade-off**
7. **Ví dụ Production**
8. **Những câu trả lời chưa đủ tốt**
9. **Sai lầm phổ biến của ứng viên**
10. **Follow-up Questions**
11. **Liên hệ với Production**

## Cách interviewer thực sự tư duy — đọc kỹ trước khi ôn

### 1. Interviewer không tìm người biết đáp án. Họ tìm người hiểu hệ quả.

Khi tôi hỏi "Tại sao PostgreSQL cần MVCC?", tôi không quan tâm bạn định nghĩa MVCC đúng hay không — Google làm được việc đó. Tôi quan tâm:

- Bạn có hiểu **vấn đề mà MVCC sinh ra để giải quyết** không? (reader chặn writer)
- Bạn có biết **cái giá phải trả** không? (bloat, vacuum)
- Bạn có từng **gặp hệ quả đó trên production** không? (transaction ID wraparound lúc 3 giờ sáng)

Một câu trả lời Senior luôn có cấu trúc: **vấn đề gốc → giải pháp → cái giá → khi nào cái giá trở thành vấn đề mới**.

### 2. Mỗi câu hỏi là một cái cây, không phải một cái flashcard

Câu hỏi đầu tiên chỉ là gốc cây. Interviewer sẽ leo theo nhánh dựa trên câu trả lời của bạn. Nếu bạn nói "tôi dùng Redis để cache", nhánh tiếp theo chắc chắn là: cache invalidation thế nào, stampede thì sao, Redis chết thì sao. **Đừng bao giờ nhắc đến một công nghệ mà bạn không sẵn sàng bị đào sâu.**

### 3. Từ khoá phân biệt cấp bậc

- **Junior**: mô tả "cái gì" — "goroutine là lightweight thread".
- **Mid**: mô tả "thế nào" — "goroutine được multiplex lên OS thread bởi scheduler".
- **Senior**: giải thích "tại sao" và trade-off — "Go chọn user-space scheduling vì context switch của OS thread tốn ~1–10µs và 1MB stack mặc định, không thể mở 1 triệu thread; đổi lại phải tự xử lý blocking syscall bằng cách tách M khỏi P".
- **Staff**: đặt vào bối cảnh hệ thống — "vì vậy khi service của tôi có 500k goroutine chờ I/O, điều tôi thực sự phải lo là file descriptor limit và memory của stack, không phải scheduler".
- **Principal**: đặt vào bối cảnh tổ chức và chi phí — "chúng tôi chọn Go cho gateway vì chi phí vận hành thấp hơn, nhưng giữ Node cho BFF vì team frontend own nó — quyết định công nghệ là quyết định con người".

### 4. Cách trả lời khi không biết

Nói thẳng: "Tôi chưa vận hành cái này ở production, nhưng dựa trên nguyên lý X, tôi suy luận là Y". Interviewer đánh giá cao khả năng suy luận từ first principles hơn là chém gió. Chém gió một lần, mọi câu trả lời trước đó của bạn đều bị nghi ngờ lại.

### 5. Cấu trúc trả lời 30 giây → 3 phút

Luôn trả lời phiên bản 30 giây trước, rồi hỏi "anh/chị muốn em đi sâu vào phần nào?". Điều này thể hiện kỹ năng giao tiếp của Senior: tôn trọng thời gian, để interviewer điều hướng, và không lan man.

> Ba chương 07–09 mở rộng ra ngoài "core backend" để phủ những mảng phỏng vấn Senior thực tế hay hỏi: bảo mật/auth, nền tảng mạng–OS–container, và kỹ năng dẫn dắt một buổi system design end-to-end (khác với Q&A: chương 09 dạy cách *dẫn dắt 45 phút*, có framework ở Phần 0 nên đọc trước).

## Tỷ lệ nội dung của bộ tài liệu

- ~20% Fundamentals (rải trong mọi chương — câu mở đầu mỗi chủ đề)
- ~30% Database & Distributed Systems (chương 03, 04, và phần distributed của 06)
- ~20% Golang & Node.js Internals (chương 01, 02)
- ~20% System Design (chương 06)
- ~10% Production Engineering (mục 7 và 11 của mọi câu hỏi + phần Observability/DR)

## Cách ôn hiệu quả

Đừng đọc như đọc sách. Với mỗi câu hỏi: che phần đáp án, tự trả lời thành tiếng trong 30 giây, rồi 3 phút. So sánh với tài liệu. Chỗ nào bạn chỉ nói được "cái gì" mà không nói được "tại sao" — đó là lỗ hổng thật sự. Sau đó tự trả lời toàn bộ Follow-up Questions mà không nhìn tài liệu.