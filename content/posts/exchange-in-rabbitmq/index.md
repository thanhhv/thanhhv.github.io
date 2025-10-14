+++
title = "Hiểu rõ cách hoạt động của RabbitMQ Exchange khi dùng NestJS Microservice"
date = "2025-10-14T14:00:00+07:00"
draft = false
tags = ["RabbitMQ", "NestJS"]
+++

![rabbitmq-exchange](rabbit-exchange.png)

Trong quá trình làm việc với **microservice trong NestJS**, mình chủ yếu dùng **RabbitMQ microservice transport**.  
Bản chất là các service giao tiếp qua **message broker (RabbitMQ)**, **chứ không gọi HTTP trực tiếp**.

Mỗi service có thể đóng vai trò:

- **Publisher**: gửi message
- **Consumer**: nhận message

Khi gửi message, service sẽ **publish đến exchange** — đây là **nơi định tuyến trung gian**.  
RabbitMQ dùng **binding rule giữa exchange và các queue** để quyết định message sẽ được gửi đến queue nào.  
Các service khác (**consumer**) sẽ **subscribe vào queue**, lắng nghe message được định tuyến đến đó và xử lý.

---

Hôm nay chúng ta sẽ cùng nhau tìm hiểu **các loại exchange trong RabbitMQ** nhé.  
Hôm trước đi phỏng vấn mình gặp đúng case này nên viết bài này để ghi nhớ luôn 😄

Bắt đầu nhé.

---

## 1. Direct Exchange

![direct-exchange](direct-exchange.png)

Message có một **routing key cụ thể** (ví dụ: `"order.created"`).

Exchange chỉ gửi message tới queue nào có **binding key trùng khớp 100%**.  
Giao tiếp kiểu **1-1**, rất chính xác.

**Ví dụ:**

```
Exchange: order_exchange
Queue: order_created_queue (bind với key order.created)
```

Khi publish message với key `"order.created"`, chỉ queue đó nhận.

**Dùng khi:**  
Cần định tuyến chính xác đến một consumer.

**Trong NestJS**

> NestJS mặc định dùng **Direct exchange** nếu không chỉ định loại khác.

- Producer service: (Gửi message)

```typescript
// app.module.ts
@Module({
  imports: [
    ClientsModule.register([
      {
        name: 'ORDER_SERVICE',
        transport: Transport.RMQ,
        options: {
          urls: ['amqp://guest:guest@localhost:5672'],
          queue: 'order_created_queue',
          queueOptions: { durable: true },
        },
      },
    ]),
    ...
  ],
  providers: [...],
})
export class AppModule {}
```

```typescript
// app.service.ts
...
@Injectable()
export class AppService {
  constructor(
    @Inject('ORDER_SERVICE') private readonly client: ClientProxy,
  ) {}

  async createOrder(order: any) {
    // routing key: 'order.created'
    this.client.emit('order.created', order);
  }
  ...
}
```

- Consumer service (nhận message)

```typescript
// main.ts
...
async function bootstrap() {
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(
    AppModule,
    {
      transport: Transport.RMQ,
      options: {
        urls: ['amqp://guest:guest@localhost:5672'],
        queue: 'order_created_queue',
        queueOptions: { durable: true },
      },
    },
  );
  await app.listen();
}
bootstrap();
```

```typescript
// app.service.ts
...
@Controller()
export class AppService {
  @EventPattern('order.created')
  handleOrderCreated(@Payload() data: any) {
    console.log('Received order:', data);
  }
}
```

---

## 2. Fanout Exchange

![fanout-exchange](fanout-exchange.png)

- Bỏ qua routing key.
- Gửi **broadcast** đến tất cả queue đã bind vào exchange đó.
- Mỗi consumer đều nhận cùng message.

**Ví dụ:**

```
Exchange: notification_exchange
Queue: email_queue
Queue: sms_queue
Queue: push_queue
```

Publish 1 message → cả 3 queue đều nhận.

**Dùng khi:**  
Muốn nhiều service nhận cùng một event  
(ví dụ: user đăng ký mới → gửi email, SMS, push notification).

**Trong NestJS:**

```typescript
// Producer
client.emit('user.registered', { userId: 123 });
// Consumer (các service khác nhau đều lắng nghe)
@EventPattern('user.registered')
handleUserRegistered(data) { ... }
```

> Fanout exchange trong RabbitMQ chính là cơ chế tương đương với **pub/sub pattern**.

---

## 3. Topic Exchange

![topic-exchange](topic-exchange.png)

**Cơ chế:**  
Sử dụng **pattern routing key** (dấu `*` và `#`):

- `*` = match 1 từ (ngăn cách bởi `.`)
- `#` = match 0 hoặc nhiều từ

Giúp định tuyến linh hoạt và powerful hơn.

**Ví dụ:**

```
Exchange: logs_topic
Queue A: bind với key user.* (nhận user.created, user.deleted)
Queue B: bind với key user.# (nhận tất cả message bắt đầu bằng user.)
```

**Dùng khi:**  
Muốn nhóm các sự kiện theo chủ đề (**event-based architecture**).  
Ví dụ: `order.created`, `order.cancelled`, `payment.refunded`.

**Trong NestJS:**

```typescript
@EventPattern('order.*')
handleOrderEvents(data) {
  console.log(data);
}
```

> @EventPattern('order.\*') chỉ là cách đặt tên, chứ không khiến NestJS tự tạo Topic Exchange.
> Nếu bạn thực sự muốn wildcard routing, bạn phải tự cấu hình exchange type = topic trong RabbitMQ hoặc code.

---

## 4. Headers Exchange

**Cơ chế:**  
Không dùng routing key, mà so sánh theo **header key-value** của message.

**Dùng khi:**  
Routing phức tạp, phụ thuộc metadata hoặc nhiều điều kiện.  
Thường dùng trong hệ thống lớn, nhiều chiều dữ liệu.

**Trong NestJS:**  
Headers exchange ít dùng trực tiếp — cần cấu hình ở RabbitMQ layer chứ không qua decorator.

> Phức tạp, thực tế mình **chưa gặp phải loại này**.

## 5. Hiểu sâu hơn về cách hoạt động trong NestJS

---

### 🟩 1. NestJS mặc định dùng Direct Exchange

Khi bạn tạo microservice với `transport: Transport.RMQ`,  
NestJS **mặc định dùng Direct Exchange** (tên là `amq.direct`).

Nó sẽ:

- Tự động tạo 1 **queue** (nếu chưa có).
- **Bind queue đó vào exchange `amq.direct`** với **routing key = pattern** bạn khai báo trong `@EventPattern()`.

```typescript
@EventPattern('order.created')
handleOrderCreated(data) { ... }
```

Khi chạy, NestJS sẽ bind queue với routing key `"order.created"` đến exchange `amq.direct`.  
👉 Đây **không phải Topic Exchange**, mà chỉ là **Direct binding chính xác**.

---

### 🟨 2. Khi bạn viết `@EventPattern('order.*')`, NestJS **không tạo Topic Exchange**

Dù bạn dùng ký tự `*` hoặc `#`, NestJS **không tự chuyển sang topic exchange**.  
Nó vẫn **bind với Direct Exchange**, nghĩa là:

- RabbitMQ **sẽ không hiểu** ký tự `*` hay `#` theo wildcard.
- Nó sẽ coi `'order.*'` chỉ là một **chuỗi literal (exact match)**.

Tức là:

- Nếu publisher emit `'order.created'`,
- Consumer `@EventPattern('order.*')` **sẽ KHÔNG nhận được**,  
  trừ khi bạn có một queue thực sự bind bằng key `'order.created'`.

---

### 🟦 3. Nếu muốn dùng Topic Exchange thật sự

Bạn phải **tự định nghĩa `exchangeType = 'topic'`** khi tạo connection hoặc queue trong RabbitMQ.

Cụ thể:

1. Tạo exchange thủ công (qua **RabbitMQ Management UI** hoặc **code**).
2. Bind queue vào exchange đó với key pattern `order.*`.
3. Trong NestJS, cấu hình exchange tương ứng trong `ClientProxy` hoặc `microservice` option.

**Ví dụ:**

```typescript
options: {
  urls: ['amqp://guest:guest@localhost:5672'],
  exchange: 'logs_topic',
  exchangeType: 'topic',
  routingKey: 'order.*',
  queue: 'order_topic_queue',
}
```

### 🟧 4. Tương tự với Fanout

> Fanout bỏ qua routing key

Giống như **Topic Exchange**, **NestJS mặc định không tạo Fanout Exchange**.  
Nó chỉ dùng **Direct Exchange (`amq.direct`)** trừ khi bạn chỉ định khác.

Vì vậy, nếu bạn muốn dùng **Fanout**, bạn cần:

1. **Tạo exchange kiểu fanout** trong RabbitMQ (qua UI hoặc script).
2. **Khai báo** `exchangeType: 'fanout'` trong NestJS microservice config.

**Ví dụ:**

```typescript
options: {
  urls: ['amqp://guest:guest@localhost:5672'],
  exchange: 'notification_exchange',
  exchangeType: 'fanout',
}
```

---

> 💡 **Tóm lại:**  
> NestJS mặc định chỉ hỗ trợ **Direct Exchange (`amq.direct`)**.
>
> Nếu bạn muốn dùng **Topic** hoặc **Fanout**, bạn cần **tự cấu hình `exchangeType`**  
> trong RabbitMQ hoặc trong phần `options` khi khởi tạo microservice.

---

Việc lựa chọn **loại exchange** trong RabbitMQ phụ thuộc vào **bài toán cụ thể**:

- Cần **định tuyến chính xác** → dùng _Direct_
- Cần **broadcast cho nhiều service** → dùng _Fanout_
- Cần **phân loại theo chủ đề** → dùng _Topic_
- Cần **routing phức tạp theo metadata** → dùng _Headers_

Hy vọng qua bài viết này, mình (và bạn 😄) đã nắm được sâu hơn về **exchange trong RabbitMQ**.

Cảm ơn bạn đã đọc!  
Nếu có góp ý, cứ _feel free_ để lại comment phía dưới nhé.

**Bye for now 👋**
