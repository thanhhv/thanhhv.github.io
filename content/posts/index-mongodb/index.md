+++
title = "Những điều mình học được về Index trong MongoDB"
date = "2025-10-05T10:50:00+07:00"
draft = false
tags = ["mongodb", "nosql", "index"]
+++

Khi làm việc với MongoDB, chắc hẳn bạn đã nghe đến **index** - một khái niệm tưởng đơn giản nhưng lại có ảnh hưởng rất lớn đến hiệu năng của hệ thống.  
Nếu không hiểu rõ cách hoạt động của index, bạn có thể khiến truy vấn chậm đi đáng kể, hoặc tốn nhiều tài nguyên mà không biết lý do.

Trong bài viết này, mình sẽ chia sẻ một cách dễ hiểu nhất về **index trong MongoDB**: nó là gì, tại sao cần dùng, có những loại nào và hoạt động ra sao.

Hãy cùng mình bắt đầu nhé

![Index trong mongodb](mongodb-index-min.png)

## 1. Index là gì, tại sao lại phải sử dụng index

**Index** trong MongoDB (và cả các database khác) giống như **mục lục của một cuốn sách**.  
Thay vì phải đọc hết từng trang để tìm một từ khóa, bạn chỉ cần mở phần mục lục, thấy trang nào có nội dung cần, rồi nhảy thẳng đến đó.

Hiểu đơn giản thì:

- Mỗi **collection** trong MongoDB là một "cuốn sách" chứa hàng triệu document.
- Khi bạn query mà **không có index**, MongoDB phải **đọc toàn bộ collection** (_collection scan_), cực kỳ tốn thời gian.
- Khi bạn **tạo index** trên field mà bạn hay truy vấn (`find`, `sort`, `filter`…), MongoDB sẽ tạo ra **một cấu trúc dữ liệu riêng** (thường là **B-Tree**) để ghi nhớ vị trí của các document có giá trị đó.

### Tại sao cần dùng index

- **Tăng tốc độ truy vấn:** tìm kiếm, lọc, sắp xếp dữ liệu nhanh hơn rất nhiều.
- **Giảm tải hệ thống:** MongoDB không phải quét toàn bộ collection → ít I/O hơn, tiết kiệm CPU và RAM.
- **Bắt buộc cho unique constraint:** ví dụ `unique index` đảm bảo không có 2 user có cùng email.
- **Hỗ trợ sort hiệu quả:** khi `sort()` trên field có index, MongoDB không cần sắp xếp thủ công.

---

## 2. Các loại index trong MongoDB

MongoDB hỗ trợ nhiều loại index khác nhau tùy vào mục đích sử dụng:

### 1️⃣ Single Field Index

Đánh index trên một field duy nhất, giúp tăng tốc độ truy vấn và sắp xếp theo field đó.

```
db.users.createIndex({ name: 1 })
```

---

### 2️⃣ Compound Index

Đánh index trên nhiều field cùng lúc, thường dùng khi query lọc hoặc sort theo nhiều trường kết hợp.

```
db.users.createIndex({ lastName: 1, firstName: 1 })
```

---

### 3️⃣ Multikey Index

Dùng cho các field có kiểu dữ liệu là mảng.  
Mỗi phần tử trong mảng sẽ được index riêng, giúp việc tìm kiếm trong array nhanh hơn.

```
db.users.createIndex({ tags: 1 })
```

---

### 4️⃣ TTL (Time-To-Live) Index

Dùng cho các dữ liệu có thời hạn, giúp MongoDB tự động xóa document sau một khoảng thời gian nhất định.

```
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 })
```

---

### 5️⃣ Partial / Sparse Index

- **Partial Index:** chỉ index những document thỏa điều kiện filter.
- **Sparse Index:** chỉ index những document có field tồn tại.  
  Giúp tiết kiệm dung lượng và tối ưu truy vấn khi dữ liệu không đồng nhất.

```
db.users.createIndex({ age: 1 }, { partialFilterExpression: { age: { $gte: 18 } } })
```

---

### 6️⃣ Text Index

Hỗ trợ tìm kiếm toàn văn (full-text search) trên các chuỗi ký tự, thường dùng cho tính năng tìm kiếm theo từ khóa.

```
db.articles.createIndex({ content: "text" })
```

---

> Ngoài ra, MongoDB còn có các loại index khác như **Geospatial**, **Hashed**, hoặc **Wildcard** — phục vụ cho dữ liệu vị trí, sharding, hoặc schema động, nhưng thực tế ít khi phải dùng đến trong các ứng dụng (một phần mình cũng chưa có làm qua =)) ) .

## 3. Cơ chế hoạt động của index trong MongoDB

Khi bạn tạo một index, MongoDB sẽ xây dựng **một cấu trúc dữ liệu riêng biệt (thường là B-Tree)** để lưu các cặp `key → pointer`, trong đó:

- **Key** là giá trị của field được đánh index.
- **Pointer** là vị trí của document thật trong collection.

Cấu trúc này được lưu trong cùng database, không phải là một collection riêng, nhưng về bản chất là **một vùng dữ liệu tách biệt** được MongoDB quản lý riêng để tối ưu tốc độ truy vấn.

---

### Ví dụ minh họa

**Collection:**

```js
{ _id: 1, name: "Gavin" }
{ _id: 2, name: "Thanhhv" }
{ _id: 3, name: "Alice" }
```

**Index `{ name: 1 }` sẽ lưu:**

```
"Alice"   → pointer tới doc _id=3
"Gavin"   → pointer tới doc _id=1
"Thanhhv" → pointer tới doc _id=2
```

**Quá trình truy vấn**

Khi có index:

- MongoDB kiểm tra xem field được query có index hay không.

- Nếu có, nó tìm trong B-Tree index để xác định pointer → nhảy trực tiếp đến document tương ứng.

- Không cần đọc toàn bộ collection → tốc độ nhanh, độ phức tạp khoảng **O(log n)**.

Giả sử: `db.users.find({ name: "Gavin" })`

MongoDB sẽ tra index, tìm key `Gavin` và truy cập ngay document `_id = 1`.

Khi không có index:

- MongoDB phải quét toàn bộ collection (`COLLSCAN`), đọc từng document rồi so sánh giá trị field name.

- Cách này chậm hơn rất nhiều, đặc biệt khi collection lớn, với độ phức **O(n)**.

> Index giúp MongoDB truy cập dữ liệu nhanh hơn rất nhiều bằng cách lưu bản đồ giữa giá trị và vị trí document thật.
> Nếu không có index, MongoDB phải đọc toàn bộ collection để tìm dữ liệu phù hợp, khiến tốc độ truy vấn giảm đáng kể.

## 4. Lưu ý khi sử dụng index

- Mỗi khi bạn **insert**, **update**, hoặc **delete** document, MongoDB phải **cập nhật lại index tương ứng**, vì vậy sẽ tốn thêm chi phí ghi (write overhead).
- Do đó, **chỉ nên tạo index** cho những field **thường xuyên dùng để query, filter hoặc sort**.
- **Tạo quá nhiều index** có thể khiến:
  - Hiệu năng ghi (**insert/update**) bị chậm đi.
  - MongoDB tiêu tốn nhiều **RAM** hơn, vì hệ thống cố gắng giữ index trong bộ nhớ để truy vấn nhanh hơn.

> ⚡ _Lời khuyên:_  
> Hãy cân bằng giữa tốc độ đọc và tốc độ ghi.  
> Tạo index đủ dùng, chọn đúng key — đừng quá nhiều, cũng đừng quá ít.

---

Cảm ơn bạn đã dành thời gian đọc bài viết này. Nếu có góp ý hoặc thắc mắc, đừng ngại để lại bình luận để chúng ta cùng thảo luận thêm nhé.
