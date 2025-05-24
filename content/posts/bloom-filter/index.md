+++
title = "Bloom filter là gì và trường hợp sử dụng"
date = "2025-05-24T20:00:00+07:00"
draft = false
tags = ["backend", "nodejs"]
+++

Chào mọi người, hôm nay trong lúc mình ôn tập kiến thức thì nhớ đến Bloom Filter, thế là mình lại phải viết thêm một bài nói về cái này nữa rồi =))

**Bloom Filter** là một cấu trúc dữ liệu xác suất được thiết kế để kiểm tra nhanh chóng xem một phần tử có thuộc tập hợp hay không.

Công dụng của nó là kiểm tra nhanh xem *"Cái này có chưa nhỉ?"*

* Nếu nó bảo là **chưa có** → chắc chắn là chưa.
* Nếu nó bảo là **có rồi** → có thể có, hoặc là nhầm \:D

## Ví dụ về cách hoạt động của Bloom Filter

1. Khởi tạo một mảng bit (0,0,0,0,...).
2. Khởi tạo một vài hàm băm (h1, h2, h3,...).
3. Khi thêm một phần tử, dùng các hàm băm để tính ra các vị trí trong mảng bit → set các vị trí đó thành 1.
4. Khi kiểm tra phần tử khác, băm ra các vị trí tương ứng và kiểm tra xem các bit đó đã là 1 chưa.

## Minh họa ví dụ

Giả sử bài toán là: Nhận một đoạn text, nếu đoạn text đã tồn tại thì báo lỗi, nếu chưa thì thêm vào.

### 1. Khởi tạo mảng bit

Giả sử 8 bit: `[0,0,0,0,0,0,0,0]`
(Chọn 8 cho dễ minh họa, thực tế có thể hàng ngàn bit. Nếu quá nhỏ → dễ xung đột bit, tăng false positive. Nếu quá lớn → tốn bộ nhớ.)

### 2. Định nghĩa hàm băm:

```js
h1(x) = (sum of ASCII letters) % 8
h2(x) = (length of string * 3) % 8
```

![Ascii](ascii.png)

### 3. Thêm phần tử `cat`

* h1("cat") = (99 + 97 + 116) % 8 = 312 % 8 = 0 → `bit[0] = 1`
* h2("cat") = 3 \* 3 = 9 → 9 % 8 = 1 → `bit[1] = 1`

Mảng bit: `[1,1,0,0,0,0,0,0]`

### 4. Kiểm tra phần tử `dog`

* h1("dog") = (100 + 111 + 103) = 314 → 314 % 8 = 2
* h2("dog") = 3 \* 3 = 9 → 9 % 8 = 1

Kiểm tra `bit[2] = 0` → chắc chắn `dog` **chưa có** → set `bit[2] = 1`

Mảng bit: `[1,1,1,0,0,0,0,0]`

### 5. Thêm phần tử `duck`

* h1("duck") = (100 + 117 + 99 + 107) = 423 % 8 = 7
* h2("duck") = 4 \* 3 = 12 → 12 % 8 = 4

Update: `bit[7] = 1`, `bit[4] = 1`

Mảng bit: `[1,1,1,0,1,0,0,1]`

### 6. Kiểm tra lại `cat`

* h1("cat") = 0, h2("cat") = 1 → `bit[0]` và `bit[1]` đều = 1 → **có thể tồn tại** → cần kiểm tra lại DB để chắc chắn.

### 7. Ví dụ dương tính giả với `god`

* h1("god") = (103 + 111 + 100) = 314 % 8 = 2
* h2("god") = 3 \* 3 = 9 % 8 = 1

`bit[2] = 1`, `bit[1] = 1` → Bloom Filter nói: "god có thể tồn tại". Nhưng thực tế chưa từng thêm `god`.
→ Đây là **false positive**: các phần tử trước đó đã vô tình bật bit đó.

## Các trường hợp sử dụng phổ biến trong backend

### ✅ Kiểm tra nhanh username đã tồn tại chưa

* Người dùng nhập `username = gavin`
* Hash `gavin` bằng `h1`, `h2` → bật các bit tương ứng → lưu mảng bit vào Redis
* Lần sau có người nhập `gavin`, hash lại:

  * Nếu **ít nhất 1 bit = 0** → chắc chắn chưa tồn tại → cho phép tạo tài khoản
  * Nếu **tất cả bit = 1** → có thể đã tồn tại → cần truy vấn DB để xác thực

➡️ Giảm tải cho DB, phản hồi nhanh

> ⚠️ **Bloom Filter không thay thế DB**, vì có thể false positive

### ✅ Kiểm tra blacklist, spam IP/email

* Dùng Bloom Filter để lưu các IP/email bị chặn
* Trước khi xử lý request, check nhanh qua Bloom Filter

➡️ Rất nhanh và nhẹ, tiết kiệm truy vấn

---

Hy vọng bài viết giúp bạn hiểu rõ hơn về Bloom Filter và ứng dụng nó vào các hệ thống thực tế một cách hiệu quả!
