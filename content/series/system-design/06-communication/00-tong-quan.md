+++
title = "Phần 6 — Communication"
date = "2026-07-13T09:30:00+07:00"
draft = false
tags = ["backend", "system-design"]
series = ["System Design — Tư Duy Thiết Kế Hệ Thống"]
+++

> Chủ đề: REST, GraphQL, gRPC, RabbitMQ, Kafka, Event-driven, Saga, Outbox Pattern.

## Luận điểm trung tâm của phần này

Chọn cơ chế giao tiếp là chọn **hợp đồng về thời gian và lỗi** — trước khi là chọn công nghệ. Ba câu hỏi định hình mọi lựa chọn trong phần này:

1. **Bên gọi có chờ không?** Sync (chờ kết quả để đi tiếp) vs async (ghi nhận ý định, kết quả đến sau). Sync dễ suy luận, giam tài nguyên theo độ trễ của bên kia; async chịu tải và cô lập lỗi, đổi bằng eventual consistency với chính nghiệp vụ của mình ([12.3](/series/system-design/12-evolution/03-background-worker/)).
2. **Lỗi hiện ở đâu?** Sync: ngay trước mặt, trong response. Async: âm thầm phía sau — backlog, DLQ, lag ([Phần 13.3](/series/system-design/13-production-failure-cases/03-messaging-failures/)). Hệ async đòi giám sát chủ động gấp đôi.
3. **Ai phải biết ai tồn tại?** Gọi trực tiếp: caller biết callee. Work queue: producer biết *việc*, không biết worker. Event log: producer không biết ai nghe ([12.7](/series/system-design/12-evolution/07-kafka-event-driven/)). Hướng của sự phụ thuộc quyết định hệ thống tiến hóa dễ hay khó.

Quy tắc rút gọn xuyên suốt: **command cần kết quả → sync (REST/gRPC); việc cần làm → work queue (RabbitMQ); sự thật đã xảy ra → log (Kafka).** Ba công cụ, ba việc — dùng chéo là nguồn anti-pattern.

## Mục lục

**Nhóm sync — request/response:**

- [6.1. REST — hợp đồng chung của web](/series/system-design/06-communication/01-rest/)
- [6.2. GraphQL — client tự khai hình dữ liệu](/series/system-design/06-communication/02-graphql/)
- [6.3. gRPC — RPC có kỷ luật cho nội bộ](/series/system-design/06-communication/03-grpc/)

**Nhóm async — messaging:**

- [6.4. RabbitMQ — smart broker cho work queue](/series/system-design/06-communication/04-rabbitmq/)
- [6.5. Kafka — distributed log cho sự kiện](/series/system-design/06-communication/05-kafka/)
- [6.6. Event-driven Architecture — nghĩ bằng sự kiện](/series/system-design/06-communication/06-event-driven/)

**Nhóm pattern — tin cậy xuyên ranh giới:**

- [6.7. Saga — transaction khi không còn transaction](/series/system-design/06-communication/07-saga/)
- [6.8. Outbox Pattern — móng của mọi event đáng tin](/series/system-design/06-communication/08-outbox/)

## Bốn kỷ luật áp cho MỌI cạnh giao tiếp

Bất kể chọn cơ chế nào, mỗi cạnh gọi ra ngoài process phải có đủ bốn thứ — chúng là thuộc tính của *cạnh*, không phải của công nghệ:

1. **Timeout tường minh** — không nhận default của thư viện ([13.5 — 3rd party](/series/system-design/13-production-failure-cases/05-infrastructure-failures/)).
2. **Retry có kỷ luật** — backoff + jitter + budget, chỉ retry lỗi đáng retry ([13.3 — Retry Storm](/series/system-design/13-production-failure-cases/03-messaging-failures/)).
3. **Circuit breaker / fail-fast** — cắt vòng khuếch đại ([13.4 — Cascading](/series/system-design/13-production-failure-cases/04-distributed-failures/)).
4. **Idempotency ở bên nhận** — vì retry + at-least-once nghĩa là mọi thứ có thể đến hai lần ([13.3 — Duplication](/series/system-design/13-production-failure-cases/03-messaging-failures/)).

Bốn thứ này nên nằm trong service template/SDK nội bộ ([12.6](/series/system-design/12-evolution/06-microservices/)) — kỷ luật bằng công cụ, không bằng trí nhớ của từng dev.
