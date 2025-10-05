+++
title = "Index trong mongodb"
date = "2025-10-05T10:50:00+07:00"
draft = true
tags = ["mongodb", "nosql", "index"]
+++

## 1. Index là gì, tại sao lại phải sử dụng index


## 2. Có các loại index nào trong mongodb




## 3. Cơ chế hoạt động của index trong mongodb

- Khi bạn tạo index (ví dụ db.users.createIndex({ name: 1 })), MongoDB sẽ:

Tạo một cấu trúc dữ liệu riêng biệt (thường là B-Tree) để lưu cặp key → pointer (trỏ đến vị trí document trong collection thật).

Index này được lưu cùng trong cùng database, chứ không hẳn là một collection riêng, nhưng bản chất là một vùng lưu riêng biệt.

ví dụ:
Collection: users
{
  _id: 1, name: "Gavin"
}
{
  _id: 2, name: "Thanhhv"
}
{
  _id: 3, name: "Alice"
}

Index { name: 1 } sẽ lưu:

"Gavin"  → pointer tới doc _id=1
"Thanhhv"    → pointer tới doc _id=2
"Alice"→ pointer tới doc _id=3

- Khi bạn query:
Query: db.users.find({ name: "Gavin" })

MongoDB check xem field "name" có index không.

Nếu có → nó truy cập B-Tree index, tìm key "Gavin", lấy pointer → nhảy trực tiếp tới vị trí document trong data file.

Không cần scan toàn bộ collection → nhanh hơn rất nhiều (O(log n)).

Nếu không có index:

MongoDB phải quét toàn bộ collection (COLLSCAN), đọc từng document để so sánh field name.

→ Cực chậm khi collection lớn.

## 4. Lưu ý khi sử dụng index
Index cập nhật song song mỗi khi insert/update/delete document → tốn thêm chi phí ghi.

Vì vậy: chỉ nên tạo index cho các field thường xuyên query/filter/sort.

Quá nhiều index → insert/update chậm.

